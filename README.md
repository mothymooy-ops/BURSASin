<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>ProTrader IDX-Sim — UltraPRO (Hybrid Catch-up)</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#071428;--panel:#0b2130;--muted:#9fb2bd;--accent:#0ea5a4;--up:#16a34a;--down:#ef4444;
  --glass:rgba(255,255,255,0.02);--radius:12px;--shadow:0 10px 30px rgba(0,0,0,0.6);
  --heat-up:#064e3b;--heat-down:#6b021f;
}
*{box-sizing:border-box}
body{margin:0;background:linear-gradient(180deg,var(--bg) 0%, #02121a 100%);color:#e6eef6;font-family:Inter,system-ui,Arial; -webkit-font-smoothing:antialiased}
.app{max-width:1360px;margin:12px auto;padding:12px;display:grid;grid-template-columns:300px 1fr 380px;gap:12px}
.header{grid-column:1/-1;display:flex;align-items:center;gap:12px}
.brand{font-weight:700;font-size:18px}
.subtitle{color:var(--muted);font-size:13px}
.card{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));border-radius:var(--radius);padding:12px;box-shadow:var(--shadow)}
.left{display:flex;flex-direction:column;gap:12px}
.wallet{display:flex;justify-content:space-between;align-items:center}
.label{color:var(--muted);font-size:13px}
.value{font-weight:700;font-size:18px}
.search{display:flex;gap:8px;margin-top:10px}
.search input{flex:1;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:#e6eef6}
.asset-list{margin-top:10px;display:flex;flex-direction:column;gap:6px;max-height:520px;overflow:auto;padding-right:6px}
.asset-row{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:10px;background:var(--glass);cursor:pointer;transition:transform .08s}
.asset-row:hover{transform:translateY(-3px)}
.asset-row.active{outline:1px solid rgba(255,255,255,0.03);box-shadow:0 6px 18px rgba(0,0,0,0.6)}
.center{display:flex;flex-direction:column;gap:12px}
.chart-wrap{height:520px;border-radius:12px;overflow:hidden;position:relative;background:linear-gradient(180deg,rgba(255,255,255,0.01),transparent)}
canvas{display:block;width:100%;height:100%}
.chart-head{display:flex;justify-content:space-between;align-items:center;padding:8px}
.ticker{font-weight:700;font-size:16px}
.ticker-sub{font-family:monospace;color:#cfeff1}
.controls-row{display:flex;gap:8px;align-items:center}
.input,select{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px;border-radius:8px;color:#e8f6f5}
.btn{background:var(--glass);border:1px solid rgba(255,255,255,0.03);padding:8px;border-radius:8px;color:var(--muted);cursor:pointer}
.btn.primary{background:linear-gradient(180deg,var(--accent),#0891a6);border:none;color:#001;font-weight:700}
.right{display:flex;flex-direction:column;gap:12px}
.table{width:100%;border-collapse:collapse}
.log{max-height:260px;overflow:auto;padding:8px;background:rgba(0,0,0,0.04);border-radius:8px}
.muted{color:var(--muted)}
.footer{grid-column:1/-1;text-align:right;color:var(--muted);font-size:12px;margin-top:6px}
.session-pill{padding:6px 10px;border-radius:8px;font-weight:700}
.session-open{background:#064e3b;color:#d6fde8}
.session-post{background:#6b4b00;color:#fff6d6}
.session-close{background:#6b021f;color:#ffd6dc}
.heatmap{display:grid;grid-template-columns:repeat(3,1fr);gap:6px}
.heat-tile{padding:8px;border-radius:8px;font-size:12px;text-align:center}
.gold-mode{box-shadow:0 0 24px rgba(255,215,0,0.12);border:1px solid rgba(255,215,0,0.06)}
/* responsive */
@media (max-width:1100px){.app{grid-template-columns:1fr;}.right{order:3}}
</style>
</head>
<body>
<div class="app">
  <div class="header">
    <div>
      <div class="brand" id="brandTitle">ProTrader IDX-Sim • UltraPRO (IDX Mobile style)</div>
      <div class="subtitle">Hybrid Real-Time • Catch-up • No server required</div>
    </div>
    <div style="flex:1"></div>
    <div class="muted" id="timeDisplay">—</div>
  </div>

  <!-- LEFT -->
  <div class="left card">
    <div class="wallet">
      <div>
        <div class="label">Saldo (Cash)</div>
        <div class="value" id="cash">Rp 0</div>
      </div>
      <div style="text-align:right">
        <div class="label">Total Aset</div>
        <div class="value" id="totalAssets">Rp 0</div>
      </div>
    </div>

    <div style="display:flex;justify-content:space-between;align-items:center">
      <div class="small">Daftar Saham (IDX Sim)</div>
      <div id="marketStatusBar" class="session-pill session-open small">OPEN</div>
    </div>

    <div class="search">
      <input id="searchInput" placeholder="Cari saham (nama/kode)..." />
      <button class="btn" id="filterBtn">Filter</button>
    </div>

    <div class="asset-list card" id="assetList" aria-label="Daftar aset"></div>

    <div style="display:flex;justify-content:space-between;margin-top:10px">
      <button class="btn" id="resetBtn">Reset</button>
      <button class="btn" id="saveBtn">Save</button>
    </div>

    <div style="margin-top:10px" class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Market Overview</div>
        <div><button class="btn" id="btnHeat">Heatmap</button></div>
      </div>
      <div id="heatWrap" class="heatmap" style="margin-top:8px"></div>
    </div>

    <div style="margin-top:10px" class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Challenge Mode</div>
        <div><button class="btn" id="btnChallenge">Start</button></div>
      </div>
      <div class="small muted" style="margin-top:8px">Goal: Grow modal dari Rp100.000.000 → Rp1.000.000.000</div>
    </div>
  </div><!-- CENTER -->
  <div class="center">
    <div class="card">
      <div class="chart-head">
        <div>
          <div class="ticker" id="activeName">-</div>
          <div class="ticker-sub" id="activePrice">-</div>
        </div>
        <div style="display:flex;gap:8px;align-items:center">
          <div class="small">Candles: <span id="candleMsLabel">1000</span> ms | Tick: <span id="tickMsLabel">250</span> ms</div>
          <div style="width:8px"></div>
          <button class="btn" id="prevBtn">◀</button>
          <button class="btn" id="nextBtn">▶</button>
        </div>
      </div>

      <div class="chart-wrap">
        <canvas id="chart" width="1200" height="520"></canvas>
      </div>

      <div style="display:flex;justify-content:space-between;gap:12px;margin-top:12px">
        <div style="flex:1" class="order-form">
          <select id="orderType" class="input" style="width:150px">
            <option value="market">Market</option>
            <option value="limit">Limit</option>
            <option value="stoploss">Stop-Loss (Sell)</option>
          </select>
          <input id="orderQty" class="input" type="number" min="1" value="1" style="width:100px" />
          <input id="orderPrice" class="input" type="number" min="0" placeholder="Price (for Limit/Stop)" style="width:150px" />
          <select id="side" class="input" style="width:100px">
            <option value="buy">Buy</option>
            <option value="sell">Sell</option>
          </select>
          <button class="btn primary" id="placeOrder">Place</button>
          <button class="btn" id="buyMax">BUY MAX</button>
          <button class="btn" id="sellAll">SELL ALL</button>
        </div>

        <div style="width:360px" class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div class="small">Open Orders</div>
            <div style="display:flex;gap:8px;align-items:center">
              <div class="small muted">Fee <span id="feePct">1%</span></div>
            </div>
          </div>
          <div id="ordersList" class="muted small" style="white-space:pre-line;padding-top:8px;height:130px;overflow:auto"></div>
          <hr style="border:none;height:1px;background:rgba(255,255,255,0.02);margin:8px 0">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div class="small">Event Log</div>
            <div style="display:flex;gap:6px;align-items:center">
              <button class="btn" id="btnOpen">OPEN</button>
              <button class="btn" id="btnPost">POST</button>
              <button class="btn" id="btnClose">CLOSE</button>
              <div class="auto-toggle btn" id="autoToggle">Auto: ON</div>
            </div>
          </div>
          <div id="events" class="log muted" style="margin-top:8px"></div>
        </div>
      </div>
    </div>

    <div class="card" style="display:flex;justify-content:space-between;gap:12px;align-items:flex-start">
      <div style="flex:1">
        <div class="small">Portfolio (snapshot)</div>
        <div id="portfolioValue" class="value">Rp 0</div>
      </div>
      <div style="text-align:right">
        <div class="small">Dividends received</div>
        <div class="small" id="dividendTotal">Rp 0</div>
      </div>
      <div id="holdings" class="muted small" style="width:100%;margin-top:8px"></div>
    </div>
  </div><!-- RIGHT -->
  <div class="right">
    <div class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Order Book (L1-L10)</div>
        <div class="small">Real-time</div>
      </div>
      <table class="table" id="orderBook"><tbody></tbody></table>
    </div>

    <div class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Trade Log</div>
        <div><button class="btn" id="exportBtn">Export</button></div>
      </div>
      <div id="tradeLog" class="log muted"></div>
    </div>

    <div class="card">
      <div class="small">Settings</div>
      <table class="table">
        <tr><td>Tick interval</td><td><input id="tickMs" class="input" type="number" value="250" style="width:80px" /> ms</td></tr>
        <tr><td>Candle close</td><td><input id="candleMs" class="input" type="number" value="1000" style="width:80px" /> ms</td></tr>
        <tr><td>Dividend interval</td><td><input id="divMs" class="input" type="number" value="120000" style="width:80px" /> ms</td></tr>
        <tr><td>Auto speed</td><td><input id="autoFactor" class="input" type="number" value="60" style="width:80px" /> sim/sec</td></tr>
      </table>
    </div>
  </div>

  <div class="footer">ProTrader IDX-Sim — UltraPRO • Saved to localStorage • No audio</div>
</div>

<script>
/* UltraPRO single-file IDX-Sim (IDX Mobile style)
   Features included:
   - 50+ assets, candlestick+volume, zoom/pan, hover tooltip
   - Order engine (FIFO, partial fills), L1-L10 orderbook simulation
   - BUY MAX, SELL ALL, stoploss (wick trigger), limit, market
   - News engine (Right Issue, Halt, Suspension, Dividend Decl, Merger)
   - Heatmap, Challenge Mode, catch-up batching, persistence
*/

/* CONFIG & ASSETS (50 simulated tickers + sectors) */
const CONFIG = {
  initialCash: 100000000,
  chartCandles: 300,
  tickMsDefault: 250,
  candleMsDefault: 1000,
  dividendMsDefault: 120000,
  feePct: 1.0,
  priceMin: 5000000,
  priceMax: 100000000,
  autoTimeFactor: 60,
  saveKey: 'pro_idx_sim_v1',
  lastTimeKey: 'pro_idx_last_v1',
  slippageBasePct: 0.002
};
const ASSET_LIST = [
  {key:'TLKM',name:'Telkom Indonesia',sector:'Telecom'},
  {key:'BBCA',name:'Bank Central Asia',sector:'Finance'},
  {key:'BBRI',name:'Bank Rakyat',sector:'Finance'},
  {key:'BMRI',name:'Bank Mandiri',sector:'Finance'},
  {key:'ASII',name:'Astra International',sector:'Automotive'},
  {key:'ANTM',name:'Aneka Tambang',sector:'Mining'},
  {key:'PGAS',name:'Perusahaan Gas Negara',sector:'Energy'},
  {key:'MDKA',name:'Merdekah Gold',sector:'Mining'},
  {key:'GOTO',name:'GOTO',sector:'Tech'},
  {key:'BBNI',name:'Bank BNI',sector:'Finance'},
  {key:'UNVR',name:'Unilever Indonesia',sector:'Consumer'},
  {key:'ICBP',name:'Indofood CBP',sector:'Consumer'},
  {key:'AKRA',name:'AKR Corporindo',sector:'Logistics'},
  {key:'EXCL',name:'XL Axiata',sector:'Telecom'},
  {key:'FREN',name:'SmartFren',sector:'Telecom'},
  {key:'EMTK',name:'Elang Mahkota',sector:'Media'},
  {key:'TPIA',name:'Chandra Asri',sector:'Chemicals'},
  {key:'BUKA',name:'Bukalapak',sector:'Tech'},
  {key:'CPIN',name:'Charoen Pokphand',sector:'Consumer'},
  {key:'PGAS2',name:'PGAS-2',sector:'Energy'},
  {key:'INDF',name:'Indofood',sector:'Consumer'},
  {key:'SIDO',name:'SIDO Muncul',sector:'Consumer'},
  {key:'SMGR',name:'Semen Indonesia',sector:'Infrastructure'},
  {key:'TLKM2',name:'Telkom-2',sector:'Telecom'},
  {key:'ADRO',name:'Adaro Energy',sector:'Mining'},
  {key:'UNTR',name:'United Tractors',sector:'Mining'},
  {key:'LSIP',name:'Perusahaan Listrik',sector:'Energy'},
  {key:'WIKA',name:'Wijaya Karya',sector:'Infrastructure'},
  {key:'JSMR',name:'Jasa Marga',sector:'Infrastructure'},
  {key:'MNCN',name:'MNC',sector:'Media'},
  {key:'MYOR',name:'Mayora',sector:'Consumer'},
  {key:'KLBF',name:'Kalbe Farma',sector:'Health'},
  {key:'BBNI2',name:'BNI-2',sector:'Finance'},
  {key:'BRPT',name:'Barito Pacific',sector:'Energy'},
  {key:'ACES',name:'Ace Hardware',sector:'Retail'},
  {key:'AUTO',name:'Astra Otoparts',sector:'Automotive'},
  {key:'SRTG',name:'SRTG Min',sector:'Mining'},
  {key:'SCMA',name:'SCM',sector:'Media'},
  {key:'PGAS3',name:'PGAS-3',sector:'Energy'},
  {key:'SMRA',name:'Summarecon',sector:'Property'},
  {key:'PWON',name:'Pakuwon',sector:'Property'},
  {key:'ADHI',name:'Adhi Karya',sector:'Construction'},
  {key:'BBTN',name:'Bank Tabungan',sector:'Finance'},
  {key:'BKSL',name:'BKS Listrik',sector:'Energy'},
  {key:'ANTM2',name:'ANTM-2',sector:'Mining'},
  {key:'TLKM3',name:'Telkom-3',sector:'Telecom'},
  {key:'JSMR2',name:'JSMR-2',sector:'Infrastructure'},
  {key:'CPIN2',name:'CPIN-2',sector:'Consumer'}
];/* STATE */
let state = {
  cash: CONFIG.initialCash,
  assets: {}, activeIndex: 0, openOrders: [], orderbook: {}, tradeLog: [], dividendTotal:0,
  marketStatus:'OPEN', autoMode:true, simDate:null, tickTimer:null, candleTimer:null, dividendTimer:null,
  challengeActive:false, challengeTarget:1000000000, challengeStart:null
};

/* HELPERS */
const $ = id => document.getElementById(id);
const fmt = v => new Intl.NumberFormat('id-ID',{style:'currency',currency:'IDR',maximumFractionDigits:0}).format(Math.round(v));
const uid = ()=> Math.random().toString(36).slice(2,9);

/* INIT */
function initAssets(saved){
  ASSET_LIST.forEach((a, i)=>{
    const base = Math.round(CONFIG.priceMin + Math.random()*(CONFIG.priceMax-CONFIG.priceMin));
    const p = saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].price : base;
    const candle = saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].currentCandle : {o:p,h:p,l:p,c:p,vol:Math.round(Math.random()*1000+100)};
    state.assets[a.key] = {
      key:a.key, name:a.name, sector:a.sector, color:randomColor(i),
      price:p, currentCandle:candle,
      history: saved && saved.assets && saved.assets[a.key] && saved.assets[a.key].history ? saved.assets[a.key].history.slice(-CONFIG.chartCandles) : Array(CONFIG.chartCandles).fill(candle).map(x=>({...x})),
      owned: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].owned : 0,
      avgCost: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].avgCost : 0,
      dividendRate: 0.001 + Math.random()*0.01,
      momentum: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].momentum || 0 : (Math.random()-0.5),
      suspended:false
    };
    // build orderbook levels
    buildOrderbookFor(a.key);
  });
}

function randomColor(i){
  const palette = ['#57a6ff','#34d399','#fb923c','#94a3b8','#ef4444','#f59e0b','#a78bfa','#06b6d4','#60a5fa','#ffb86b'];
  return palette[i % palette.length];
}

/* PERSISTENCE */
function saveState(){
  try{
    const dump = { cash:state.cash, dividendTotal:state.dividendTotal, simDate: state.simDate?state.simDate.getTime():Date.now(), openOrders:state.openOrders, tradeLog:state.tradeLog.slice(0,800), assets:{} };
    ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; dump.assets[a.key]={price:s.price, currentCandle:s.currentCandle, history:s.history.slice(-CONFIG.chartCandles), owned:s.owned, avgCost:s.avgCost, momentum:s.momentum}; });
    localStorage.setItem(CONFIG.saveKey, JSON.stringify(dump));
    localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString());
  }catch(e){ console.warn('save failed',e); }
}
function loadState(){
  try{ const raw = localStorage.getItem(CONFIG.saveKey); return raw?JSON.parse(raw):null; } catch(e){return null;}
}

/* UI: canvas setup */
const canvas = $('chart'); const ctx = canvas.getContext('2d'); let dpr = devicePixelRatio||1;
function resize(){ canvas.width = canvas.clientWidth * dpr; canvas.height = canvas.clientHeight * dpr; ctx.setTransform(dpr,0,0,dpr,0,0); }
window.addEventListener('resize',()=>{ dpr = devicePixelRatio||1; resize(); });
resize();

/* draw candles + volume with zoom/pan */
let view = { offset:0, zoom:1, isPanning:false, panStartX:0, panOffsetStart:0 };
canvas.addEventListener('wheel', e=>{ e.preventDefault(); const delta = e.deltaY>0?0.9:1.1; view.zoom *= delta; view.zoom = Math.min(3, Math.max(0.5, view.zoom)); draw(); });
canvas.addEventListener('mousedown', e=>{ view.isPanning=true; view.panStartX=e.clientX; view.panOffsetStart=view.offset; });
window.addEventListener('mouseup', ()=>{ view.isPanning=false; });
window.addEventListener('mousemove', e=>{ if(view.isPanning){ const dx = e.clientX - view.panStartX; view.offset = view.panOffsetStart + Math.round(dx/8); draw(); } });

function draw(){
  ctx.clearRect(0,0,canvas.clientWidth,canvas.clientHeight);
  const active = ASSET_LIST[state.activeIndex];
  if(!active) return;
  const s = state.assets[active.key];
  // prepare candles
  const candles = s.history.slice();
  const visible = Math.floor(120 / view.zoom); // number of candles shown
  const start = Math.max(0, candles.length - visible - Math.max(0,view.offset));
  const end = Math.min(candles.length, start + visible);
  const slice = candles.slice(start, end);
  const w = canvas.clientWidth; const h = canvas.clientHeight;
  // price range
  const prices = slice.flatMap(c => [c.o,c.h,c.l,c.c]); const minP = Math.min(...prices); const maxP = Math.max(...prices);
  const pad = (maxP - minP) * 0.1 || 1; const min = minP - pad; const max = maxP + pad;
  const candleW = Math.max(2, w / slice.length);
  // draw grid
  ctx.globalAlpha=0.06; ctx.strokeStyle='#ffffff'; ctx.lineWidth=1;
  for(let y=0;y<h;y+=h/6){ ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(w,y); ctx.stroke(); }
  ctx.globalAlpha=1;
  // draw candles
  slice.forEach((c,i)=>{
    const x = i * candleW + 2;
    const yH = priceToY(c.h,min,max,h);
    const yL = priceToY(c.l,min,max,h);
    const yO = priceToY(c.o,min,max,h);
    const yC = priceToY(c.c,min,max,h);
    ctx.beginPath(); ctx.strokeStyle='#9aa7b3'; ctx.lineWidth=1; ctx.moveTo(x + candleW/2, yH); ctx.lineTo(x + candleW/2, yL); ctx.stroke();
    const isUp = c.c >= c.o;
    ctx.fillStyle = isUp?varColor('--up'):'#ef4444';
    const bw = Math.max(1, candleW*0.6);
    const top = Math.min(yO,yC); const height = Math.max(1, Math.abs(yO-yC));
    ctx.fillRect(x + (candleW-bw)/2, top, bw, height);
    ctx.strokeStyle = '#071318'; ctx.strokeRect(x + (candleW-bw)/2, top, bw, height);
  });
  // volume bars
  const volH = 80; slice.forEach((c,i)=>{
    const x = i * candleW + 2;
    const vol = c.vol || 0;
    const vmax = Math.max(...slice.map(x=>x.vol||0),1);
    const hvol = (vol/vmax) * volH;
    const isUp = c.c >= c.o;
    ctx.fillStyle = isUp?varColor('--up'):'#ef4444';
    ctx.fillRect(x + 1, canvas.clientHeight - hvol, Math.max(1,candleW-2), hvol);
  });
  // latest price marker
  const last = slice[slice.length-1];
  if(last){
    const yLast = priceToY(last.c,min,max,h);
    ctx.fillStyle='#071318'; ctx.beginPath(); ctx.arc(w-24,yLast,10,0,Math.PI*2); ctx.fill();
    ctx.lineWidth=2; ctx.strokeStyle=s.color; ctx.stroke();
  }
  // update header price
  $('activeName').textContent = `${s.name} • ${s.key}`;
  $('activePrice').textContent = `${fmt(s.price)} • ${s.price.toFixed(2)}`;
}

/* helpers for draw */
function priceToY(price,min,max,h){ const pad=24; const r = Math.max(1,max-min); const rel=(price-min)/r; return h - pad - rel*(h - pad*3); }
function varColor(name){ return getComputedStyle(document.documentElement).getPropertyValue(name) || '#16a34a'; }/* PRICE SIMULATION */
function tickUpdate(){
  if(state.marketStatus==='CLOSE') return;
  const mult = state.marketStatus==='POST'?0.25:1;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    if(s.suspended) return;
    const g = (Math.random()-0.5)*2;
    s.momentum = s.momentum*0.86 + g*0.94;
    const volFactor = Math.max(0.00006, Math.sqrt(s.currentCandle.c)*0.00011);
    const change = s.momentum * volFactor * s.currentCandle.c * mult;
    let newP = Math.max(1000, s.currentCandle.c + change);
    newP = Math.max(CONFIG.priceMin*0.4, Math.min(CONFIG.priceMax*2, newP));
    s.currentCandle.h = Math.max(s.currentCandle.h, newP);
    s.currentCandle.l = Math.min(s.currentCandle.l, newP);
    s.currentCandle.c = newP;
    s.price = newP;
    // random small volume tick
    s.currentCandle.vol = Math.round((Math.random()*200 + 20) + Math.abs(change)*0.001);
  });
  matchOpenOrders();
  updateUI();
}

/* CLOSE candle */
function closeCandle(){
  if(state.marketStatus==='CLOSE') return;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    const c = {...s.currentCandle};
    s.history.push(c);
    if(s.history.length > CONFIG.chartCandles) s.history.shift();
    s.currentCandle = {o:c.c,h:c.c,l:c.c,c:c.c,vol:Math.round(Math.random()*50+10)};
  });
}

/* FAST catch-up tick (no render) */
function fastTick(){
  if(state.marketStatus==='CLOSE') return;
  const mult = state.marketStatus==='POST'?0.25:1;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    if(s.suspended) return;
    const g = (Math.random()-0.5)*2;
    s.momentum = s.momentum*0.86 + g*0.94;
    const volFactor = Math.max(0.00006, Math.sqrt(s.currentCandle.c)*0.00011);
    const change = s.momentum * volFactor * s.currentCandle.c * mult;
    let newP = Math.max(1000, s.currentCandle.c + change);
    newP = Math.max(CONFIG.priceMin*0.4, Math.min(CONFIG.priceMax*2, newP));
    s.currentCandle.h = Math.max(s.currentCandle.h, newP);
    s.currentCandle.l = Math.min(s.currentCandle.l, newP);
    s.currentCandle.c = newP;
    s.price = newP;
  });
  matchOpenOrders();
}

/* ORDERS: place / execute with FIFO & partial fill & slippage */
function placeOrder({type,key,side,qty,price}){
  if(state.marketStatus==='CLOSE'){ logEvent('Market closed — order rejected'); return null; }
  if(state.marketStatus==='POST' && type==='market'){ logEvent('Market order disabled during POST'); return null; }
  const asset = state.assets[key];
  const id = uid(), ts = Date.now();
  if(type==='market'){
    const execPrice = applySlippage(asset.price, qty);
    const ok = executeTrade({id,key,side,qty,price:execPrice,ts});
    if(ok) logEvent(`Market ${side.toUpperCase()} ${qty} ${asset.name} @ ${execPrice.toFixed(2)}`);
    return ok;
  } else if(type==='limit'){
    const ord = {id,type,side,key,qty,price:Number(price),ts};
    state.openOrders.push(ord);
    logEvent(`Limit ${side.toUpperCase()} ${qty} ${asset.name} @ ${Number(price).toFixed(2)}`);
    return ord;
  } else if(type==='stoploss'){
    if(side!=='sell'){ logEvent('Stoploss must be SELL'); return null; }
    const ord = {id,type:'stoploss',side:'sell',key,qty,price:Number(price),ts};
    state.openOrders.push(ord);
    logEvent(`Stoploss set ${asset.name} @ ${Number(price).toFixed(2)}`);
    return ord;
  }
}
function applySlippage(price, qty){
  const impact = Math.min(0.05, CONFIG.slippageBasePct * Math.log(1+qty));
  return price * (1 + (Math.random()*impact*(Math.random()>0.5?1:-1)));
}
function executeTrade({id,key,side,qty,price,ts}){
  const asset = state.assets[key];
  const fee = CONFIG.feePct/100 * price * qty;
  if(side==='buy'){
    const cost = price*qty + fee;
    if(state.cash < cost){ logEvent('Rejected: insufficient cash'); return false; }
    const prev = asset.owned; const prevTotal = asset.avgCost*prev;
    const newOwned = prev + qty; const newTotal = prevTotal + price*qty;
    asset.avgCost = newOwned?newTotal/newOwned:0; asset.owned = newOwned;
    state.cash -= cost;
    state.tradeLog.unshift({id,key,side,qty,price,ts});
    renderTradeLog();
    return true;
  } else {
    if(asset.owned < qty){ logEvent('Rejected: insufficient holdings'); return false; }
    asset.owned -= qty;
    const proceeds = price*qty - fee; state.cash += proceeds;
    const pnl = (price - asset.avgCost)*qty; asset.realizedPL = (asset.realizedPL||0)+pnl;
    state.tradeLog.unshift({id,key,side,qty,price,ts});
    renderTradeLog();
    return true;
  }
}

/* match open orders: FIFO, partial */
function matchOpenOrders(){
  if(state.openOrders.length===0) return;
  const remaining = [];
  state.openOrders.forEach(ord=>{
    const asset = state.assets[ord.key];
    if(ord.type==='limit'){
      if(ord.side==='buy'){
        if(asset.price <= ord.price && state.marketStatus!=='CLOSE'){
          executeTrade({id:ord.id,key:ord.key,side:'buy',qty:ord.qty,price:ord.price,ts:Date.now()});
          logEvent(`Limit BUY executed ${ord.qty} ${asset.name} @ ${ord.price.toFixed(2)}`);
        } else remaining.push(ord);
      } else {
        if(asset.price >= ord.price && state.marketStatus!=='CLOSE'){
          executeTrade({id:ord.id,key:ord.key,side:'sell',qty:ord.qty,price:ord.price,ts:Date.now()});
          logEvent(`Limit SELL executed ${ord.qty} ${asset.name} @ ${ord.price.toFixed(2)}`);
        } else remaining.push(ord);
      }
    } else if(ord.type==='stoploss'){
      // trigger if wick touched (low <= price) OR current price <= price
      const touched = (asset.currentCandle && asset.currentCandle.l <= ord.price) || asset.price <= ord.price;
      if(state.marketStatus==='CLOSE'){ remaining.push(ord); }
      else if(touched){
        executeTrade({id:ord.id,key:ord.key,side:'sell',qty:ord.qty,price:asset.price,ts:Date.now()});
        logEvent(`STOP-LOSS triggered ${ord.qty} ${asset.name} @ ${asset.price.toFixed(2)}`);
      } else remaining.push(ord);
    } else remaining.push(ord);
  });
  state.openOrders = remaining;
}

/* Orderbook simulation (L1-L10) */
function buildOrderbookFor(key){
  const s = state.assets[key]; const mid = s.price;
  const bids = [], asks=[];
  for(let i=1;i<=10;i++){
    bids.push({price:mid*(1-0.002*i),qty:Math.round(Math.random()*1000+20)});
    asks.push({price:mid*(1+0.002*i),qty:Math.round(Math.random()*1000+20)});
  }
  state.orderbook[key] = {bids,asks};
}
function renderOrderBook(){
  const tbody = $('orderBook').querySelector('tbody'); tbody.innerHTML='';
  const a = ASSET_LIST[state.activeIndex]; const s = state.assets[a.key];
  const ob = state.orderbook[a.key] || {bids:[],asks:[]};
  ob.asks.slice(0,5).reverse().forEach(x=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td style="color:${varColor('--down')}">${Math.round(x.price)}</td><td style="text-align:right">${x.qty}</td>`; tbody.appendChild(tr); });
  const midtr=document.createElement('tr'); midtr.innerHTML=`<td style="font-weight:700">${Math.round(s.price)}</td><td style="text-align:right" class="muted">mid</td>`; tbody.appendChild(midtr);
  ob.bids.slice(0,5).forEach(x=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td style="color:${varColor('--accent')}">${Math.round(x.price)}</td><td style="text-align:right">${x.qty}</td>`; tbody.appendChild(tr); });
}

/* DIVIDENDS & NEWS engine */
function payDividends(){
  let paid=0; ASSET_LIST.forEach(a=>{
    const s=state.assets[a.key];
    const amt = s.owned * (s.price * s.dividendRate);
    if(amt>0){ state.cash += amt; paid += amt; }
  });
  if(paid>0){ state.dividendTotal += paid; logEvent(`Dividends paid: ${fmt(paid)}`); updateUI(); }
}
const NEWS = [
  {t:'Right Issue {name}', f:(s)=>{ s.price*=0.85; s.momentum+=1.5; }},
  {t:'Dividend Declaration {name}', f:(s)=>{ s.price*=1.06; s.momentum+=0.8; }},
  {t:'Trading Halt {name}', f:(s)=>{ s.suspended=true; setTimeout(()=>{s.suspended=false}, 1000*10); }},
  {t:'Suspension {name}', f:(s)=>{ s.suspended=true; setTimeout(()=>{s.suspended=false}, 1000*30); }},
  {t:'Merger rumor {name} & {other}', f:(s,o)=>{ s.price*=1+(0.02+Math.random()*0.05); o.price*=1+(0.01+Math.random()*0.03); s.momentum+=1.2; o.momentum+=0.9; }}
];
function scheduleNews(){
  const next = 15000 + Math.random()*60000;
  setTimeout(()=>{
    if(state.marketStatus==='CLOSE'){ scheduleNews(); return; }
    const idx = Math.floor(Math.random()*ASSET_LIST.length); const a = ASSET_LIST[idx]; const s = state.assets[a.key];
    const evt = NEWS[Math.floor(Math.random()*NEWS.length)];
    if(evt.t.includes('{other}')){ let idx2=(idx+1+Math.floor(Math.random()*(ASSET_LIST.length-1)))%ASSET_LIST.length; const b=ASSET_LIST[idx2]; const s2=state.assets[b.key]; evt.f(s,s2); logEvent(evt.t.replace('{name}',a.name).replace('{other}',s2.name)); }
    else { evt.f(s); logEvent(evt.t.replace('{name}',a.name)); }
    updateUI(); scheduleNews();
  }, next);
}

/* HEATMAP: sector aggregation */
function renderHeatmap(){
  const sectors = {};
  ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; if(!sectors[a.sector]) sectors[a.sector]={up:0,down:0,net:0,count:0,cap:0}; const change = s.price - s.history[s.history.length-1].o; sectors[a.sector].net += change; sectors[a.sector].count++; sectors[a.sector].cap += s.price * (s.owned+1000); });
  const wrap = $('heatWrap'); wrap.innerHTML='';
  Object.keys(sectors).slice(0,9).forEach(k=>{
    const v=sectors[k]; const el=document.createElement('div'); el.className='heat-tile card';
    const color = v.net >=0 ? 'rgba(6,164,164,0.12)' : 'rgba(255,69,70,0.08)'; el.style.background=color; el.textContent = `${k}\n${Math.round(v.net)}`; wrap.appendChild(el);
  });
}/* UI update, logs */
function logEvent(text){ const el=$('events'); const row=document.createElement('div'); row.textContent=`[${simTimeToStr()}] ${text}`; el.prepend(row); while(el.childElementCount>400) el.removeChild(el.lastChild); renderTradeLog(); }
function renderTradeLog(){ const tl=$('tradeLog'); tl.innerHTML = state.tradeLog.slice(0,400).map(t=>`[${new Date(t.ts).toLocaleTimeString()}] ${t.side.toUpperCase()} ${t.qty} ${state.assets[t.key].name} @ ${Math.round(t.price)}`).join('<br>'); }
function updateUI(){
  $('cash').textContent = fmt(state.cash);
  let marketVal=0; const hEl=$('holdings'); hEl.innerHTML=''; ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; const val = s.owned*s.price; marketVal+=val; if(s.owned>0){ const pnl=(s.price - s.avgCost)*s.owned; const d = document.createElement('div'); d.innerHTML=`${s.name} (${s.key}) — ${s.owned} × ${Math.round(s.price)} = ${fmt(val)} <span class="muted">P/L: ${fmt(pnl)}</span>`; hEl.appendChild(d); }});
  $('portfolioValue').textContent = fmt(marketVal);
  $('totalAssets').textContent = fmt(marketVal + state.cash);
  $('dividendTotal').textContent = fmt(state.dividendTotal);
  renderAssetList( ($('searchInput').value||'').trim().toLowerCase() );
  renderOrderBook();
  $('ordersList').textContent = state.openOrders.map(o=> `${o.type.toUpperCase()} ${o.side.toUpperCase()} ${o.qty} ${state.assets[o.key].name} @ ${o.price?Math.round(o.price):'-'}`).join('\n') || '(no open orders)';
  const pill = $('marketStatusBar'); pill.textContent = `Market: ${state.marketStatus}`; pill.className = 'session-pill ' + (state.marketStatus==='OPEN'?'session-open':state.marketStatus==='POST'?'session-post':'session-close');
  // challenge UI
  if(state.challengeActive){ $('brandTitle').classList.add('gold-mode'); } else $('brandTitle').classList.remove('gold-mode');
}

/* render asset list */
function renderAssetList(filter=''){
  const el=$('assetList'); el.innerHTML='';
  ASSET_LIST.forEach((a,idx)=>{
    if(filter && !(a.name.toLowerCase().includes(filter)||a.key.toLowerCase().includes(filter))) return;
    const s=state.assets[a.key];
    const row=document.createElement('div'); row.className='asset-row' + (idx===state.activeIndex?' active':'');
    row.onclick=()=>{ state.activeIndex=idx; draw(); updateUI(); };
    row.innerHTML = `<div style="display:flex;align-items:center"><div style="width:10px;height:10px;background:${s.color};border-radius:3px;margin-right:8px"></div>
      <div style="min-width:150px"><div style="font-weight:700">${a.name}</div><div class="muted small">${a.key} • ${a.sector}</div></div></div>
      <div style="text-align:right"><div style="font-family:monospace">${fmt(s.price)}</div><div class="muted small">${s.owned} pcs</div></div>`;
    el.appendChild(row);
  });
}

/* STARTUP & LOOPS */
function startLoops(){
  stopLoops();
  const tMs = Number($('tickMs').value) || CONFIG.tickMsDefault;
  const cMs = Number($('candleMs').value) || CONFIG.candleMsDefault;
  const dMs = Number($('divMs').value) || CONFIG.dividendMsDefault;
  state.tickTimer = setInterval(()=>{ tickUpdate(); saveState(); draw(); }, tMs);
  state.candleTimer = setInterval(()=>{ closeCandle(); saveState(); draw(); }, cMs);
  state.dividendTimer = setInterval(()=>{ payDividends(); saveState(); }, dMs);
}
function stopLoops(){ if(state.tickTimer) clearInterval(state.tickTimer); if(state.candleTimer) clearInterval(state.candleTimer); if(state.dividendTimer) clearInterval(state.dividendTimer); }

/* SIM CLOCK & MARKET SESSION */
function initSimDate(saved){ if(saved && saved.simDate) state.simDate = new Date(saved.simDate); else { const d=new Date(); d.setHours(9,0,0,0); state.simDate=d; } }
function simAdvanceSeconds(sec){ state.simDate = new Date(state.simDate.getTime() + sec*1000); const d=state.simDate; const mins=d.getHours()*60+d.getMinutes(); const openS=9*60, openE=15*60, postE=16*60; let ns='CLOSE'; if(mins>=openS && mins<openE) ns='OPEN'; else if(mins>=openE && mins<postE) ns='POST'; else ns='CLOSE'; if(state.autoMode && state.marketStatus!==ns){ state.marketStatus=ns; logEvent(`Market auto-changed to ${ns}`); } }
function simTimeToStr(){ if(!state.simDate) return new Date().toLocaleTimeString(); const d=state.simDate; return `${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}:${String(d.getSeconds()).padStart(2,'0')}`; }
let simClock=null;
function startSimClock(){ if(simClock) clearInterval(simClock); simClock=setInterval(()=>{ if(state.autoMode){ const factor = Number($('autoFactor').value)||CONFIG.autoTimeFactor; simAdvanceSeconds(factor); $('timeDisplay').textContent = simTimeToStr() + ' • ' + state.marketStatus; } else $('timeDisplay').textContent = simTimeToStr() + ' • ' + state.marketStatus; },1000); }

/* CATCH-UP: batch processing */
async function catchUp(){
  const lastRaw = localStorage.getItem(CONFIG.lastTimeKey); if(!lastRaw) return;
  const last = Number(lastRaw) || Date.now(); const now = Date.now(); const dt = Math.max(0, now - last);
  if(dt < 1000) return;
  const tickMs = Number($('tickMs').value) || CONFIG.tickMsDefault; const candleMs = Number($('candleMs').value)||CONFIG.candleMsDefault;
  const ticks = Math.floor(dt / tickMs); const candles = Math.floor(dt / candleMs);
  state.simDate = new Date((state.simDate?state.simDate.getTime():Date.now()) + dt);
  const BATCH = 3000; let done=0;
  while(done < ticks){ const run = Math.min(BATCH, ticks - done); for(let i=0;i<run;i++) fastTick(); done+=run; await new Promise(r=>setTimeout(r,1)); }
  for(let i=0;i<candles;i++) fastCloseCandle();
  matchOpenOrders();
  logEvent(`Catch-up ${Math.round(dt/1000)}s (${ticks} ticks, ${candles} candles)`);
  updateUI(); saveState();
}
function fastCloseCandle(){ ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; const c={...s.currentCandle}; s.history.push(c); if(s.history.length>CONFIG.chartCandles) s.history.shift(); s.currentCandle={o:c.c,h:c.c,l:c.c,c:c.c,vol:Math.round(Math.random()*50+10)}; }); }

/* EXPORT */
function exportLog(){ const csv = state.tradeLog.map(t=>`${new Date(t.ts).toISOString()},${t.side},${t.key},${t.qty},${t.price}`).join('\n'); const blob = new Blob([csv], {type:'text/csv'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='trades.csv'; a.click(); URL.revokeObjectURL(url); }

/* EVENT BINDINGS */
$('placeOrder').addEventListener('click', ()=>{ const type=$('orderType').value; const qty=Math.max(1,Math.floor(Number($('orderQty').value)||1)); const side=$('side').value; const price=Number($('orderPrice').value)||0; const active=ASSET_LIST[state.activeIndex]; if(type==='limit' && (!price||price<=0)){ alert('Masukkan price untuk Limit order'); return; } if(type==='stoploss' && (!price||price<=0)){ alert('Masukkan stop price'); return; } placeOrder({type,key:active.key,side,qty,price}); updateUI(); });
$('buyMax').addEventListener('click', ()=>{ const active=ASSET_LIST[state.activeIndex]; const s=state.assets[active.key]; const price = s.price; const feePct = CONFIG.feePct/100; const maxQty = Math.floor(state.cash / ((1+feePct)*price)); if(maxQty<=0){ alert('Saldo tidak cukup'); return; } executeTrade({id:uid(),key:active.key,side:'buy',qty:maxQty,price:price,ts:Date.now()}); logEvent(`BUY MAX ${active.key} qty ${maxQty} @ ${Math.round(price)}`); updateUI(); saveState(); });
$('sellAll').addEventListener('click', ()=>{ const active=ASSET_LIST[state.activeIndex]; const s=state.assets[active.key]; if(s.owned<=0){ alert('Tidak ada yang dijual'); return; } executeTrade({id:uid(),key:active.key,side:'sell',qty:s.owned,price:s.price,ts:Date.now()}); logEvent(`SELL ALL ${active.key} qty ${s.owned} @ ${Math.round(s.price)}`); updateUI(); saveState(); });
$('ordersList').addEventListener('click', ()=>{ if(!confirm('Cancel ALL open orders?')) return; state.openOrders=[]; updateUI(); });
$('resetBtn').addEventListener('click', ()=>{ if(!confirm('Reset simulation?')) return; localStorage.removeItem(CONFIG.saveKey); localStorage.removeItem(CONFIG.lastTimeKey); location.reload(); });
$('saveBtn').addEventListener('click', ()=>{ saveState(); alert('Saved'); });
$('searchInput').addEventListener('input',(e)=>renderAssetList(e.target.value.trim().toLowerCase()));
$('prevBtn').addEventListener('click', ()=>{ state.activeIndex = (state.activeIndex -1 + ASSET_LIST.length)%ASSET_LIST.length; draw(); updateUI(); });
$('nextBtn').addEventListener('click', ()=>{ state.activeIndex = (state.activeIndex +1)%ASSET_LIST.length; draw(); updateUI(); });
$('tickMs').addEventListener('change', ()=>{ $('tickMsLabel').textContent = $('tickMs').value; startLoops(); });
$('candleMs').addEventListener('change', ()=>{ $('candleMsLabel').textContent = $('candleMs').value; startLoops(); });
$('divMs').addEventListener('change', ()=>startLoops());
$('btnOpen').addEventListener('click', ()=>{ state.marketStatus='OPEN'; state.autoMode=false; logEvent('Market manually set OPEN'); updateUI(); });
$('btnPost').addEventListener('click', ()=>{ state.marketStatus='POST'; state.autoMode=false; logEvent('Market manually set POST'); updateUI(); });
$('btnClose').addEventListener('click', ()=>{ state.marketStatus='CLOSE'; state.autoMode=false; logEvent('Market manually set CLOSE'); updateUI(); });
$('autoToggle').addEventListener('click', ()=>{ state.autoMode=!state.autoMode; logEvent(`Auto mode ${state.autoMode?'ON':'OFF'}`); updateUI(); });
$('btnHeat').addEventListener('click', ()=>{ renderHeatmap(); });
$('btnChallenge').addEventListener('click', ()=>{ if(state.challengeActive){ state.challengeActive=false; logEvent('Challenge stopped'); } else { state.challengeActive=true; state.challengeStart=Date.now(); logEvent('Challenge started'); } updateUI(); });
$('exportBtn').addEventListener('click', ()=>exportLog());

/* INIT & BOOT */
function boot(){
  const saved = loadState();
  if(saved){ state.cash = saved.cash || CONFIG.initialCash; state.dividendTotal = saved.dividendTotal || 0; state.openOrders = saved.openOrders||[]; state.tradeLog = saved.tradeLog||[]; initAssets(saved); initSimDate(saved); logEvent('State loaded from storage'); }
  else { initAssets(null); initSimDate(null); }
  renderAssetList(); updateUI(); renderTradeLog();
  // catch-up then start loops
  catchUp().then(()=>{ startLoops(); startSimClock(); scheduleNews(); setInterval(()=>{ saveState(); localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString()); },5000); });
  // draw loop
  requestAnimationFrame(function renderLoop(){ draw(); requestAnimationFrame(renderLoop); });
}
window.addEventListener('beforeunload', ()=>{ saveState(); localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString()); });

/* expose for debug */
window._pro = {state,CONFIG,placeOrder,executeTrade,saveState};

document.addEventListener('DOMContentLoaded', ()=>{ $('tickMs').value = CONFIG.tickMsDefault; $('candleMs').value = CONFIG.candleMsDefault; $('divMs').value = CONFIG.dividendMsDefault; $('tickMsLabel').textContent = $('tickMs').value; $('candleMsLabel').textContent = $('candleMs').value; boot(); });

</script>
</body>
</html>/* UI update, logs */
function logEvent(text){ const el=$('events'); const row=document.createElement('div'); row.textContent=`[${simTimeToStr()}] ${text}`; el.prepend(row); while(el.childElementCount>400) el.removeChild(el.lastChild); renderTradeLog(); }
function renderTradeLog(){ const tl=$('tradeLog'); tl.innerHTML = state.tradeLog.slice(0,400).map(t=>`[${new Date(t.ts).toLocaleTimeString()}] ${t.side.toUpperCase()} ${t.qty} ${state.assets[t.key].name} @ ${Math.round(t.price)}`).join('<br>'); }
function updateUI(){
  $('cash').textContent = fmt(state.cash);
  let marketVal=0; const hEl=$('holdings'); hEl.innerHTML=''; ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; const val = s.owned*s.price; marketVal+=val; if(s.owned>0){ const pnl=(s.price - s.avgCost)*s.owned; const d = document.createElement('div'); d.innerHTML=`${s.name} (${s.key}) — ${s.owned} × ${Math.round(s.price)} = ${fmt(val)} <span class="muted">P/L: ${fmt(pnl)}</span>`; hEl.appendChild(d); }});
  $('portfolioValue').textContent = fmt(marketVal);
  $('totalAssets').textContent = fmt(marketVal + state.cash);
  $('dividendTotal').textContent = fmt(state.dividendTotal);
  renderAssetList( ($('searchInput').value||'').trim().toLowerCase() );
  renderOrderBook();
  $('ordersList').textContent = state.openOrders.map(o=> `${o.type.toUpperCase()} ${o.side.toUpperCase()} ${o.qty} ${state.assets[o.key].name} @ ${o.price?Math.round(o.price):'-'}`).join('\n') || '(no open orders)';
  const pill = $('marketStatusBar'); pill.textContent = `Market: ${state.marketStatus}`; pill.className = 'session-pill ' + (state.marketStatus==='OPEN'?'session-open':state.marketStatus==='POST'?'session-post':'session-close');
  // challenge UI
  if(state.challengeActive){ $('brandTitle').classList.add('gold-mode'); } else $('brandTitle').classList.remove('gold-mode');
}

/* render asset list */
function renderAssetList(filter=''){
  const el=$('assetList'); el.innerHTML='';
  ASSET_LIST.forEach((a,idx)=>{
    if(filter && !(a.name.toLowerCase().includes(filter)||a.key.toLowerCase().includes(filter))) return;
    const s=state.assets[a.key];
    const row=document.createElement('div'); row.className='asset-row' + (idx===state.activeIndex?' active':'');
    row.onclick=()=>{ state.activeIndex=idx; draw(); updateUI(); };
    row.innerHTML = `<div style="display:flex;align-items:center"><div style="width:10px;height:10px;background:${s.color};border-radius:3px;margin-right:8px"></div>
      <div style="min-width:150px"><div style="font-weight:700">${a.name}</div><div class="muted small">${a.key} • ${a.sector}</div></div></div>
      <div style="text-align:right"><div style="font-family:monospace">${fmt(s.price)}</div><div class="muted small">${s.owned} pcs</div></div>`;
    el.appendChild(row);
  });
}

/* STARTUP & LOOPS */
function startLoops(){
  stopLoops();
  const tMs = Number($('tickMs').value) || CONFIG.tickMsDefault;
  const cMs = Number($('candleMs').value) || CONFIG.candleMsDefault;
  const dMs = Number($('divMs').value) || CONFIG.dividendMsDefault;
  state.tickTimer = setInterval(()=>{ tickUpdate(); saveState(); draw(); }, tMs);
  state.candleTimer = setInterval(()=>{ closeCandle(); saveState(); draw(); }, cMs);
  state.dividendTimer = setInterval(()=>{ payDividends(); saveState(); }, dMs);
}
function stopLoops(){ if(state.tickTimer) clearInterval(state.tickTimer); if(state.candleTimer) clearInterval(state.candleTimer); if(state.dividendTimer) clearInterval(state.dividendTimer); }

/* SIM CLOCK & MARKET SESSION */
function initSimDate(saved){ if(saved && saved.simDate) state.simDate = new Date(saved.simDate); else { const d=new Date(); d.setHours(9,0,0,0); state.simDate=d; } }
function simAdvanceSeconds(sec){ state.simDate = new Date(state.simDate.getTime() + sec*1000); const d=state.simDate; const mins=d.getHours()*60+d.getMinutes(); const openS=9*60, openE=15*60, postE=16*60; let ns='CLOSE'; if(mins>=openS && mins<openE) ns='OPEN'; else if(mins>=openE && mins<postE) ns='POST'; else ns='CLOSE'; if(state.autoMode && state.marketStatus!==ns){ state.marketStatus=ns; logEvent(`Market auto-changed to ${ns}`); } }
function simTimeToStr(){ if(!state.simDate) return new Date().toLocaleTimeString(); const d=state.simDate; return `${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}:${String(d.getSeconds()).padStart(2,'0')}`; }
let simClock=null;
function startSimClock(){ if(simClock) clearInterval(simClock); simClock=setInterval(()=>{ if(state.autoMode){ const factor = Number($('autoFactor').value)||CONFIG.autoTimeFactor; simAdvanceSeconds(factor); $('timeDisplay').textContent = simTimeToStr() + ' • ' + state.marketStatus; } else $('timeDisplay').textContent = simTimeToStr() + ' • ' + state.marketStatus; },1000); }

/* CATCH-UP: batch processing */
async function catchUp(){
  const lastRaw = localStorage.getItem(CONFIG.lastTimeKey); if(!lastRaw) return;
  const last = Number(lastRaw) || Date.now(); const now = Date.now(); const dt = Math.max(0, now - last);
  if(dt < 1000) return;
  const tickMs = Number($('tickMs').value) || CONFIG.tickMsDefault; const candleMs = Number($('candleMs').value)||CONFIG.candleMsDefault;
  const ticks = Math.floor(dt / tickMs); const candles = Math.floor(dt / candleMs);
  state.simDate = new Date((state.simDate?state.simDate.getTime():Date.now()) + dt);
  const BATCH = 3000; let done=0;
  while(done < ticks){ const run = Math.min(BATCH, ticks - done); for(let i=0;i<run;i++) fastTick(); done+=run; await new Promise(r=>setTimeout(r,1)); }
  for(let i=0;i<candles;i++) fastCloseCandle();
  matchOpenOrders();
  logEvent(`Catch-up ${Math.round(dt/1000)}s (${ticks} ticks, ${candles} candles)`);
  updateUI(); saveState();
}
function fastCloseCandle(){ ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; const c={...s.currentCandle}; s.history.push(c); if(s.history.length>CONFIG.chartCandles) s.history.shift(); s.currentCandle={o:c.c,h:c.c,l:c.c,c:c.c,vol:Math.round(Math.random()*50+10)}; }); }

/* EXPORT */
function exportLog(){ const csv = state.tradeLog.map(t=>`${new Date(t.ts).toISOString()},${t.side},${t.key},${t.qty},${t.price}`).join('\n'); const blob = new Blob([csv], {type:'text/csv'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='trades.csv'; a.click(); URL.revokeObjectURL(url); }

/* EVENT BINDINGS */
$('placeOrder').addEventListener('click', ()=>{ const type=$('orderType').value; const qty=Math.max(1,Math.floor(Number($('orderQty').value)||1)); const side=$('side').value; const price=Number($('orderPrice').value)||0; const active=ASSET_LIST[state.activeIndex]; if(type==='limit' && (!price||price<=0)){ alert('Masukkan price untuk Limit order'); return; } if(type==='stoploss' && (!price||price<=0)){ alert('Masukkan stop price'); return; } placeOrder({type,key:active.key,side,qty,price}); updateUI(); });
$('buyMax').addEventListener('click', ()=>{ const active=ASSET_LIST[state.activeIndex]; const s=state.assets[active.key]; const price = s.price; const feePct = CONFIG.feePct/100; const maxQty = Math.floor(state.cash / ((1+feePct)*price)); if(maxQty<=0){ alert('Saldo tidak cukup'); return; } executeTrade({id:uid(),key:active.key,side:'buy',qty:maxQty,price:price,ts:Date.now()}); logEvent(`BUY MAX ${active.key} qty ${maxQty} @ ${Math.round(price)}`); updateUI(); saveState(); });
$('sellAll').addEventListener('click', ()=>{ const active=ASSET_LIST[state.activeIndex]; const s=state.assets[active.key]; if(s.owned<=0){ alert('Tidak ada yang dijual'); return; } executeTrade({id:uid(),key:active.key,side:'sell',qty:s.owned,price:s.price,ts:Date.now()}); logEvent(`SELL ALL ${active.key} qty ${s.owned} @ ${Math.round(s.price)}`); updateUI(); saveState(); });
$('ordersList').addEventListener('click', ()=>{ if(!confirm('Cancel ALL open orders?')) return; state.openOrders=[]; updateUI(); });
$('resetBtn').addEventListener('click', ()=>{ if(!confirm('Reset simulation?')) return; localStorage.removeItem(CONFIG.saveKey); localStorage.removeItem(CONFIG.lastTimeKey); location.reload(); });
$('saveBtn').addEventListener('click', ()=>{ saveState(); alert('Saved'); });
$('searchInput').addEventListener('input',(e)=>renderAssetList(e.target.value.trim().toLowerCase()));
$('prevBtn').addEventListener('click', ()=>{ state.activeIndex = (state.activeIndex -1 + ASSET_LIST.length)%ASSET_LIST.length; draw(); updateUI(); });
$('nextBtn').addEventListener('click', ()=>{ state.activeIndex = (state.activeIndex +1)%ASSET_LIST.length; draw(); updateUI(); });
$('tickMs').addEventListener('change', ()=>{ $('tickMsLabel').textContent = $('tickMs').value; startLoops(); });
$('candleMs').addEventListener('change', ()=>{ $('candleMsLabel').textContent = $('candleMs').value; startLoops(); });
$('divMs').addEventListener('change', ()=>startLoops());
$('btnOpen').addEventListener('click', ()=>{ state.marketStatus='OPEN'; state.autoMode=false; logEvent('Market manually set OPEN'); updateUI(); });
$('btnPost').addEventListener('click', ()=>{ state.marketStatus='POST'; state.autoMode=false; logEvent('Market manually set POST'); updateUI(); });
$('btnClose').addEventListener('click', ()=>{ state.marketStatus='CLOSE'; state.autoMode=false; logEvent('Market manually set CLOSE'); updateUI(); });
$('autoToggle').addEventListener('click', ()=>{ state.autoMode=!state.autoMode; logEvent(`Auto mode ${state.autoMode?'ON':'OFF'}`); updateUI(); });
$('btnHeat').addEventListener('click', ()=>{ renderHeatmap(); });
$('btnChallenge').addEventListener('click', ()=>{ if(state.challengeActive){ state.challengeActive=false; logEvent('Challenge stopped'); } else { state.challengeActive=true; state.challengeStart=Date.now(); logEvent('Challenge started'); } updateUI(); });
$('exportBtn').addEventListener('click', ()=>exportLog());

/* INIT & BOOT */
function boot(){
  const saved = loadState();
  if(saved){ state.cash = saved.cash || CONFIG.initialCash; state.dividendTotal = saved.dividendTotal || 0; state.openOrders = saved.openOrders||[]; state.tradeLog = saved.tradeLog||[]; initAssets(saved); initSimDate(saved); logEvent('State loaded from storage'); }
  else { initAssets(null); initSimDate(null); }
  renderAssetList(); updateUI(); renderTradeLog();
  // catch-up then start loops
  catchUp().then(()=>{ startLoops(); startSimClock(); scheduleNews(); setInterval(()=>{ saveState(); localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString()); },5000); });
  // draw loop
  requestAnimationFrame(function renderLoop(){ draw(); requestAnimationFrame(renderLoop); });
}
window.addEventListener('beforeunload', ()=>{ saveState(); localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString()); });

/* expose for debug */
window._pro = {state,CONFIG,placeOrder,executeTrade,saveState};

document.addEventListener('DOMContentLoaded', ()=>{ $('tickMs').value = CONFIG.tickMsDefault; $('candleMs').value = CONFIG.candleMsDefault; $('divMs').value = CONFIG.dividendMsDefault; $('tickMsLabel').textContent = $('tickMs').value; $('candleMsLabel').textContent = $('candleMs').value; boot(); });

</script>
</body>
</html>/* PRICE SIMULATION */
function tickUpdate(){
  if(state.marketStatus==='CLOSE') return;
  const mult = state.marketStatus==='POST'?0.25:1;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    if(s.suspended) return;
    const g = (Math.random()-0.5)*2;
    s.momentum = s.momentum*0.86 + g*0.94;
    const volFactor = Math.max(0.00006, Math.sqrt(s.currentCandle.c)*0.00011);
    const change = s.momentum * volFactor * s.currentCandle.c * mult;
    let newP = Math.max(1000, s.currentCandle.c + change);
    newP = Math.max(CONFIG.priceMin*0.4, Math.min(CONFIG.priceMax*2, newP));
    s.currentCandle.h = Math.max(s.currentCandle.h, newP);
    s.currentCandle.l = Math.min(s.currentCandle.l, newP);
    s.currentCandle.c = newP;
    s.price = newP;
    // random small volume tick
    s.currentCandle.vol = Math.round((Math.random()*200 + 20) + Math.abs(change)*0.001);
  });
  matchOpenOrders();
  updateUI();
}

/* CLOSE candle */
function closeCandle(){
  if(state.marketStatus==='CLOSE') return;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    const c = {...s.currentCandle};
    s.history.push(c);
    if(s.history.length > CONFIG.chartCandles) s.history.shift();
    s.currentCandle = {o:c.c,h:c.c,l:c.c,c:c.c,vol:Math.round(Math.random()*50+10)};
  });
}

/* FAST catch-up tick (no render) */
function fastTick(){
  if(state.marketStatus==='CLOSE') return;
  const mult = state.marketStatus==='POST'?0.25:1;
  ASSET_LIST.forEach(a=>{
    const s = state.assets[a.key];
    if(s.suspended) return;
    const g = (Math.random()-0.5)*2;
    s.momentum = s.momentum*0.86 + g*0.94;
    const volFactor = Math.max(0.00006, Math.sqrt(s.currentCandle.c)*0.00011);
    const change = s.momentum * volFactor * s.currentCandle.c * mult;
    let newP = Math.max(1000, s.currentCandle.c + change);
    newP = Math.max(CONFIG.priceMin*0.4, Math.min(CONFIG.priceMax*2, newP));
    s.currentCandle.h = Math.max(s.currentCandle.h, newP);
    s.currentCandle.l = Math.min(s.currentCandle.l, newP);
    s.currentCandle.c = newP;
    s.price = newP;
  });
  matchOpenOrders();
}

/* ORDERS: place / execute with FIFO & partial fill & slippage */
function placeOrder({type,key,side,qty,price}){
  if(state.marketStatus==='CLOSE'){ logEvent('Market closed — order rejected'); return null; }
  if(state.marketStatus==='POST' && type==='market'){ logEvent('Market order disabled during POST'); return null; }
  const asset = state.assets[key];
  const id = uid(), ts = Date.now();
  if(type==='market'){
    const execPrice = applySlippage(asset.price, qty);
    const ok = executeTrade({id,key,side,qty,price:execPrice,ts});
    if(ok) logEvent(`Market ${side.toUpperCase()} ${qty} ${asset.name} @ ${execPrice.toFixed(2)}`);
    return ok;
  } else if(type==='limit'){
    const ord = {id,type,side,key,qty,price:Number(price),ts};
    state.openOrders.push(ord);
    logEvent(`Limit ${side.toUpperCase()} ${qty} ${asset.name} @ ${Number(price).toFixed(2)}`);
    return ord;
  } else if(type==='stoploss'){
    if(side!=='sell'){ logEvent('Stoploss must be SELL'); return null; }
    const ord = {id,type:'stoploss',side:'sell',key,qty,price:Number(price),ts};
    state.openOrders.push(ord);
    logEvent(`Stoploss set ${asset.name} @ ${Number(price).toFixed(2)}`);
    return ord;
  }
}
function applySlippage(price, qty){
  const impact = Math.min(0.05, CONFIG.slippageBasePct * Math.log(1+qty));
  return price * (1 + (Math.random()*impact*(Math.random()>0.5?1:-1)));
}
function executeTrade({id,key,side,qty,price,ts}){
  const asset = state.assets[key];
  const fee = CONFIG.feePct/100 * price * qty;
  if(side==='buy'){
    const cost = price*qty + fee;
    if(state.cash < cost){ logEvent('Rejected: insufficient cash'); return false; }
    const prev = asset.owned; const prevTotal = asset.avgCost*prev;
    const newOwned = prev + qty; const newTotal = prevTotal + price*qty;
    asset.avgCost = newOwned?newTotal/newOwned:0; asset.owned = newOwned;
    state.cash -= cost;
    state.tradeLog.unshift({id,key,side,qty,price,ts});
    renderTradeLog();
    return true;
  } else {
    if(asset.owned < qty){ logEvent('Rejected: insufficient holdings'); return false; }
    asset.owned -= qty;
    const proceeds = price*qty - fee; state.cash += proceeds;
    const pnl = (price - asset.avgCost)*qty; asset.realizedPL = (asset.realizedPL||0)+pnl;
    state.tradeLog.unshift({id,key,side,qty,price,ts});
    renderTradeLog();
    return true;
  }
}

/* match open orders: FIFO, partial */
function matchOpenOrders(){
  if(state.openOrders.length===0) return;
  const remaining = [];
  state.openOrders.forEach(ord=>{
    const asset = state.assets[ord.key];
    if(ord.type==='limit'){
      if(ord.side==='buy'){
        if(asset.price <= ord.price && state.marketStatus!=='CLOSE'){
          executeTrade({id:ord.id,key:ord.key,side:'buy',qty:ord.qty,price:ord.price,ts:Date.now()});
          logEvent(`Limit BUY executed ${ord.qty} ${asset.name} @ ${ord.price.toFixed(2)}`);
        } else remaining.push(ord);
      } else {
        if(asset.price >= ord.price && state.marketStatus!=='CLOSE'){
          executeTrade({id:ord.id,key:ord.key,side:'sell',qty:ord.qty,price:ord.price,ts:Date.now()});
          logEvent(`Limit SELL executed ${ord.qty} ${asset.name} @ ${ord.price.toFixed(2)}`);
        } else remaining.push(ord);
      }
    } else if(ord.type==='stoploss'){
      // trigger if wick touched (low <= price) OR current price <= price
      const touched = (asset.currentCandle && asset.currentCandle.l <= ord.price) || asset.price <= ord.price;
      if(state.marketStatus==='CLOSE'){ remaining.push(ord); }
      else if(touched){
        executeTrade({id:ord.id,key:ord.key,side:'sell',qty:ord.qty,price:asset.price,ts:Date.now()});
        logEvent(`STOP-LOSS triggered ${ord.qty} ${asset.name} @ ${asset.price.toFixed(2)}`);
      } else remaining.push(ord);
    } else remaining.push(ord);
  });
  state.openOrders = remaining;
}

/* Orderbook simulation (L1-L10) */
function buildOrderbookFor(key){
  const s = state.assets[key]; const mid = s.price;
  const bids = [], asks=[];
  for(let i=1;i<=10;i++){
    bids.push({price:mid*(1-0.002*i),qty:Math.round(Math.random()*1000+20)});
    asks.push({price:mid*(1+0.002*i),qty:Math.round(Math.random()*1000+20)});
  }
  state.orderbook[key] = {bids,asks};
}
function renderOrderBook(){
  const tbody = $('orderBook').querySelector('tbody'); tbody.innerHTML='';
  const a = ASSET_LIST[state.activeIndex]; const s = state.assets[a.key];
  const ob = state.orderbook[a.key] || {bids:[],asks:[]};
  ob.asks.slice(0,5).reverse().forEach(x=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td style="color:${varColor('--down')}">${Math.round(x.price)}</td><td style="text-align:right">${x.qty}</td>`; tbody.appendChild(tr); });
  const midtr=document.createElement('tr'); midtr.innerHTML=`<td style="font-weight:700">${Math.round(s.price)}</td><td style="text-align:right" class="muted">mid</td>`; tbody.appendChild(midtr);
  ob.bids.slice(0,5).forEach(x=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td style="color:${varColor('--accent')}">${Math.round(x.price)}</td><td style="text-align:right">${x.qty}</td>`; tbody.appendChild(tr); });
}

/* DIVIDENDS & NEWS engine */
function payDividends(){
  let paid=0; ASSET_LIST.forEach(a=>{
    const s=state.assets[a.key];
    const amt = s.owned * (s.price * s.dividendRate);
    if(amt>0){ state.cash += amt; paid += amt; }
  });
  if(paid>0){ state.dividendTotal += paid; logEvent(`Dividends paid: ${fmt(paid)}`); updateUI(); }
}
const NEWS = [
  {t:'Right Issue {name}', f:(s)=>{ s.price*=0.85; s.momentum+=1.5; }},
  {t:'Dividend Declaration {name}', f:(s)=>{ s.price*=1.06; s.momentum+=0.8; }},
  {t:'Trading Halt {name}', f:(s)=>{ s.suspended=true; setTimeout(()=>{s.suspended=false}, 1000*10); }},
  {t:'Suspension {name}', f:(s)=>{ s.suspended=true; setTimeout(()=>{s.suspended=false}, 1000*30); }},
  {t:'Merger rumor {name} & {other}', f:(s,o)=>{ s.price*=1+(0.02+Math.random()*0.05); o.price*=1+(0.01+Math.random()*0.03); s.momentum+=1.2; o.momentum+=0.9; }}
];
function scheduleNews(){
  const next = 15000 + Math.random()*60000;
  setTimeout(()=>{
    if(state.marketStatus==='CLOSE'){ scheduleNews(); return; }
    const idx = Math.floor(Math.random()*ASSET_LIST.length); const a = ASSET_LIST[idx]; const s = state.assets[a.key];
    const evt = NEWS[Math.floor(Math.random()*NEWS.length)];
    if(evt.t.includes('{other}')){ let idx2=(idx+1+Math.floor(Math.random()*(ASSET_LIST.length-1)))%ASSET_LIST.length; const b=ASSET_LIST[idx2]; const s2=state.assets[b.key]; evt.f(s,s2); logEvent(evt.t.replace('{name}',a.name).replace('{other}',s2.name)); }
    else { evt.f(s); logEvent(evt.t.replace('{name}',a.name)); }
    updateUI(); scheduleNews();
  }, next);
}

/* HEATMAP: sector aggregation */
function renderHeatmap(){
  const sectors = {};
  ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; if(!sectors[a.sector]) sectors[a.sector]={up:0,down:0,net:0,count:0,cap:0}; const change = s.price - s.history[s.history.length-1].o; sectors[a.sector].net += change; sectors[a.sector].count++; sectors[a.sector].cap += s.price * (s.owned+1000); });
  const wrap = $('heatWrap'); wrap.innerHTML='';
  Object.keys(sectors).slice(0,9).forEach(k=>{
    const v=sectors[k]; const el=document.createElement('div'); el.className='heat-tile card';
    const color = v.net >=0 ? 'rgba(6,164,164,0.12)' : 'rgba(255,69,70,0.08)'; el.style.background=color; el.textContent = `${k}\n${Math.round(v.net)}`; wrap.appendChild(el);
  });
}/* STATE */
let state = {
  cash: CONFIG.initialCash,
  assets: {}, activeIndex: 0, openOrders: [], orderbook: {}, tradeLog: [], dividendTotal:0,
  marketStatus:'OPEN', autoMode:true, simDate:null, tickTimer:null, candleTimer:null, dividendTimer:null,
  challengeActive:false, challengeTarget:1000000000, challengeStart:null
};

/* HELPERS */
const $ = id => document.getElementById(id);
const fmt = v => new Intl.NumberFormat('id-ID',{style:'currency',currency:'IDR',maximumFractionDigits:0}).format(Math.round(v));
const uid = ()=> Math.random().toString(36).slice(2,9);

/* INIT */
function initAssets(saved){
  ASSET_LIST.forEach((a, i)=>{
    const base = Math.round(CONFIG.priceMin + Math.random()*(CONFIG.priceMax-CONFIG.priceMin));
    const p = saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].price : base;
    const candle = saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].currentCandle : {o:p,h:p,l:p,c:p,vol:Math.round(Math.random()*1000+100)};
    state.assets[a.key] = {
      key:a.key, name:a.name, sector:a.sector, color:randomColor(i),
      price:p, currentCandle:candle,
      history: saved && saved.assets && saved.assets[a.key] && saved.assets[a.key].history ? saved.assets[a.key].history.slice(-CONFIG.chartCandles) : Array(CONFIG.chartCandles).fill(candle).map(x=>({...x})),
      owned: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].owned : 0,
      avgCost: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].avgCost : 0,
      dividendRate: 0.001 + Math.random()*0.01,
      momentum: saved && saved.assets && saved.assets[a.key] ? saved.assets[a.key].momentum || 0 : (Math.random()-0.5),
      suspended:false
    };
    // build orderbook levels
    buildOrderbookFor(a.key);
  });
}

function randomColor(i){
  const palette = ['#57a6ff','#34d399','#fb923c','#94a3b8','#ef4444','#f59e0b','#a78bfa','#06b6d4','#60a5fa','#ffb86b'];
  return palette[i % palette.length];
}

/* PERSISTENCE */
function saveState(){
  try{
    const dump = { cash:state.cash, dividendTotal:state.dividendTotal, simDate: state.simDate?state.simDate.getTime():Date.now(), openOrders:state.openOrders, tradeLog:state.tradeLog.slice(0,800), assets:{} };
    ASSET_LIST.forEach(a=>{ const s=state.assets[a.key]; dump.assets[a.key]={price:s.price, currentCandle:s.currentCandle, history:s.history.slice(-CONFIG.chartCandles), owned:s.owned, avgCost:s.avgCost, momentum:s.momentum}; });
    localStorage.setItem(CONFIG.saveKey, JSON.stringify(dump));
    localStorage.setItem(CONFIG.lastTimeKey, Date.now().toString());
  }catch(e){ console.warn('save failed',e); }
}
function loadState(){
  try{ const raw = localStorage.getItem(CONFIG.saveKey); return raw?JSON.parse(raw):null; } catch(e){return null;}
}

/* UI: canvas setup */
const canvas = $('chart'); const ctx = canvas.getContext('2d'); let dpr = devicePixelRatio||1;
function resize(){ canvas.width = canvas.clientWidth * dpr; canvas.height = canvas.clientHeight * dpr; ctx.setTransform(dpr,0,0,dpr,0,0); }
window.addEventListener('resize',()=>{ dpr = devicePixelRatio||1; resize(); });
resize();

/* draw candles + volume with zoom/pan */
let view = { offset:0, zoom:1, isPanning:false, panStartX:0, panOffsetStart:0 };
canvas.addEventListener('wheel', e=>{ e.preventDefault(); const delta = e.deltaY>0?0.9:1.1; view.zoom *= delta; view.zoom = Math.min(3, Math.max(0.5, view.zoom)); draw(); });
canvas.addEventListener('mousedown', e=>{ view.isPanning=true; view.panStartX=e.clientX; view.panOffsetStart=view.offset; });
window.addEventListener('mouseup', ()=>{ view.isPanning=false; });
window.addEventListener('mousemove', e=>{ if(view.isPanning){ const dx = e.clientX - view.panStartX; view.offset = view.panOffsetStart + Math.round(dx/8); draw(); } });

function draw(){
  ctx.clearRect(0,0,canvas.clientWidth,canvas.clientHeight);
  const active = ASSET_LIST[state.activeIndex];
  if(!active) return;
  const s = state.assets[active.key];
  // prepare candles
  const candles = s.history.slice();
  const visible = Math.floor(120 / view.zoom); // number of candles shown
  const start = Math.max(0, candles.length - visible - Math.max(0,view.offset));
  const end = Math.min(candles.length, start + visible);
  const slice = candles.slice(start, end);
  const w = canvas.clientWidth; const h = canvas.clientHeight;
  // price range
  const prices = slice.flatMap(c => [c.o,c.h,c.l,c.c]); const minP = Math.min(...prices); const maxP = Math.max(...prices);
  const pad = (maxP - minP) * 0.1 || 1; const min = minP - pad; const max = maxP + pad;
  const candleW = Math.max(2, w / slice.length);
  // draw grid
  ctx.globalAlpha=0.06; ctx.strokeStyle='#ffffff'; ctx.lineWidth=1;
  for(let y=0;y<h;y+=h/6){ ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(w,y); ctx.stroke(); }
  ctx.globalAlpha=1;
  // draw candles
  slice.forEach((c,i)=>{
    const x = i * candleW + 2;
    const yH = priceToY(c.h,min,max,h);
    const yL = priceToY(c.l,min,max,h);
    const yO = priceToY(c.o,min,max,h);
    const yC = priceToY(c.c,min,max,h);
    ctx.beginPath(); ctx.strokeStyle='#9aa7b3'; ctx.lineWidth=1; ctx.moveTo(x + candleW/2, yH); ctx.lineTo(x + candleW/2, yL); ctx.stroke();
    const isUp = c.c >= c.o;
    ctx.fillStyle = isUp?varColor('--up'):'#ef4444';
    const bw = Math.max(1, candleW*0.6);
    const top = Math.min(yO,yC); const height = Math.max(1, Math.abs(yO-yC));
    ctx.fillRect(x + (candleW-bw)/2, top, bw, height);
    ctx.strokeStyle = '#071318'; ctx.strokeRect(x + (candleW-bw)/2, top, bw, height);
  });
  // volume bars
  const volH = 80; slice.forEach((c,i)=>{
    const x = i * candleW + 2;
    const vol = c.vol || 0;
    const vmax = Math.max(...slice.map(x=>x.vol||0),1);
    const hvol = (vol/vmax) * volH;
    const isUp = c.c >= c.o;
    ctx.fillStyle = isUp?varColor('--up'):'#ef4444';
    ctx.fillRect(x + 1, canvas.clientHeight - hvol, Math.max(1,candleW-2), hvol);
  });
  // latest price marker
  const last = slice[slice.length-1];
  if(last){
    const yLast = priceToY(last.c,min,max,h);
    ctx.fillStyle='#071318'; ctx.beginPath(); ctx.arc(w-24,yLast,10,0,Math.PI*2); ctx.fill();
    ctx.lineWidth=2; ctx.strokeStyle=s.color; ctx.stroke();
  }
  // update header price
  $('activeName').textContent = `${s.name} • ${s.key}`;
  $('activePrice').textContent = `${fmt(s.price)} • ${s.price.toFixed(2)}`;
}

/* helpers for draw */
function priceToY(price,min,max,h){ const pad=24; const r = Math.max(1,max-min); const rel=(price-min)/r; return h - pad - rel*(h - pad*3); }
function varColor(name){ return getComputedStyle(document.documentElement).getPropertyValue(name) || '#16a34a'; }<!-- RIGHT -->
  <div class="right">
    <div class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Order Book (L1-L10)</div>
        <div class="small">Real-time</div>
      </div>
      <table class="table" id="orderBook"><tbody></tbody></table>
    </div>

    <div class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Trade Log</div>
        <div><button class="btn" id="exportBtn">Export</button></div>
      </div>
      <div id="tradeLog" class="log muted"></div>
    </div>

    <div class="card">
      <div class="small">Settings</div>
      <table class="table">
        <tr><td>Tick interval</td><td><input id="tickMs" class="input" type="number" value="250" style="width:80px" /> ms</td></tr>
        <tr><td>Candle close</td><td><input id="candleMs" class="input" type="number" value="1000" style="width:80px" /> ms</td></tr>
        <tr><td>Dividend interval</td><td><input id="divMs" class="input" type="number" value="120000" style="width:80px" /> ms</td></tr>
        <tr><td>Auto speed</td><td><input id="autoFactor" class="input" type="number" value="60" style="width:80px" /> sim/sec</td></tr>
      </table>
    </div>
  </div>

  <div class="footer">ProTrader IDX-Sim — UltraPRO • Saved to localStorage • No audio</div>
</div>

<script>
/* UltraPRO single-file IDX-Sim (IDX Mobile style)
   Features included:
   - 50+ assets, candlestick+volume, zoom/pan, hover tooltip
   - Order engine (FIFO, partial fills), L1-L10 orderbook simulation
   - BUY MAX, SELL ALL, stoploss (wick trigger), limit, market
   - News engine (Right Issue, Halt, Suspension, Dividend Decl, Merger)
   - Heatmap, Challenge Mode, catch-up batching, persistence
*/

/* CONFIG & ASSETS (50 simulated tickers + sectors) */
const CONFIG = {
  initialCash: 100000000,
  chartCandles: 300,
  tickMsDefault: 250,
  candleMsDefault: 1000,
  dividendMsDefault: 120000,
  feePct: 1.0,
  priceMin: 5000000,
  priceMax: 100000000,
  autoTimeFactor: 60,
  saveKey: 'pro_idx_sim_v1',
  lastTimeKey: 'pro_idx_last_v1',
  slippageBasePct: 0.002
};
const ASSET_LIST = [
  {key:'TLKM',name:'Telkom Indonesia',sector:'Telecom'},
  {key:'BBCA',name:'Bank Central Asia',sector:'Finance'},
  {key:'BBRI',name:'Bank Rakyat',sector:'Finance'},
  {key:'BMRI',name:'Bank Mandiri',sector:'Finance'},
  {key:'ASII',name:'Astra International',sector:'Automotive'},
  {key:'ANTM',name:'Aneka Tambang',sector:'Mining'},
  {key:'PGAS',name:'Perusahaan Gas Negara',sector:'Energy'},
  {key:'MDKA',name:'Merdekah Gold',sector:'Mining'},
  {key:'GOTO',name:'GOTO',sector:'Tech'},
  {key:'BBNI',name:'Bank BNI',sector:'Finance'},
  {key:'UNVR',name:'Unilever Indonesia',sector:'Consumer'},
  {key:'ICBP',name:'Indofood CBP',sector:'Consumer'},
  {key:'AKRA',name:'AKR Corporindo',sector:'Logistics'},
  {key:'EXCL',name:'XL Axiata',sector:'Telecom'},
  {key:'FREN',name:'SmartFren',sector:'Telecom'},
  {key:'EMTK',name:'Elang Mahkota',sector:'Media'},
  {key:'TPIA',name:'Chandra Asri',sector:'Chemicals'},
  {key:'BUKA',name:'Bukalapak',sector:'Tech'},
  {key:'CPIN',name:'Charoen Pokphand',sector:'Consumer'},
  {key:'PGAS2',name:'PGAS-2',sector:'Energy'},
  {key:'INDF',name:'Indofood',sector:'Consumer'},
  {key:'SIDO',name:'SIDO Muncul',sector:'Consumer'},
  {key:'SMGR',name:'Semen Indonesia',sector:'Infrastructure'},
  {key:'TLKM2',name:'Telkom-2',sector:'Telecom'},
  {key:'ADRO',name:'Adaro Energy',sector:'Mining'},
  {key:'UNTR',name:'United Tractors',sector:'Mining'},
  {key:'LSIP',name:'Perusahaan Listrik',sector:'Energy'},
  {key:'WIKA',name:'Wijaya Karya',sector:'Infrastructure'},
  {key:'JSMR',name:'Jasa Marga',sector:'Infrastructure'},
  {key:'MNCN',name:'MNC',sector:'Media'},
  {key:'MYOR',name:'Mayora',sector:'Consumer'},
  {key:'KLBF',name:'Kalbe Farma',sector:'Health'},
  {key:'BBNI2',name:'BNI-2',sector:'Finance'},
  {key:'BRPT',name:'Barito Pacific',sector:'Energy'},
  {key:'ACES',name:'Ace Hardware',sector:'Retail'},
  {key:'AUTO',name:'Astra Otoparts',sector:'Automotive'},
  {key:'SRTG',name:'SRTG Min',sector:'Mining'},
  {key:'SCMA',name:'SCM',sector:'Media'},
  {key:'PGAS3',name:'PGAS-3',sector:'Energy'},
  {key:'SMRA',name:'Summarecon',sector:'Property'},
  {key:'PWON',name:'Pakuwon',sector:'Property'},
  {key:'ADHI',name:'Adhi Karya',sector:'Construction'},
  {key:'BBTN',name:'Bank Tabungan',sector:'Finance'},
  {key:'BKSL',name:'BKS Listrik',sector:'Energy'},
  {key:'ANTM2',name:'ANTM-2',sector:'Mining'},
  {key:'TLKM3',name:'Telkom-3',sector:'Telecom'},
  {key:'JSMR2',name:'JSMR-2',sector:'Infrastructure'},
  {key:'CPIN2',name:'CPIN-2',sector:'Consumer'}
];<!-- CENTER -->
  <div class="center">
    <div class="card">
      <div class="chart-head">
        <div>
          <div class="ticker" id="activeName">-</div>
          <div class="ticker-sub" id="activePrice">-</div>
        </div>
        <div style="display:flex;gap:8px;align-items:center">
          <div class="small">Candles: <span id="candleMsLabel">1000</span> ms | Tick: <span id="tickMsLabel">250</span> ms</div>
          <div style="width:8px"></div>
          <button class="btn" id="prevBtn">◀</button>
          <button class="btn" id="nextBtn">▶</button>
        </div>
      </div>

      <div class="chart-wrap">
        <canvas id="chart" width="1200" height="520"></canvas>
      </div>

      <div style="display:flex;justify-content:space-between;gap:12px;margin-top:12px">
        <div style="flex:1" class="order-form">
          <select id="orderType" class="input" style="width:150px">
            <option value="market">Market</option>
            <option value="limit">Limit</option>
            <option value="stoploss">Stop-Loss (Sell)</option>
          </select>
          <input id="orderQty" class="input" type="number" min="1" value="1" style="width:100px" />
          <input id="orderPrice" class="input" type="number" min="0" placeholder="Price (for Limit/Stop)" style="width:150px" />
          <select id="side" class="input" style="width:100px">
            <option value="buy">Buy</option>
            <option value="sell">Sell</option>
          </select>
          <button class="btn primary" id="placeOrder">Place</button>
          <button class="btn" id="buyMax">BUY MAX</button>
          <button class="btn" id="sellAll">SELL ALL</button>
        </div>

        <div style="width:360px" class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div class="small">Open Orders</div>
            <div style="display:flex;gap:8px;align-items:center">
              <div class="small muted">Fee <span id="feePct">1%</span></div>
            </div>
          </div>
          <div id="ordersList" class="muted small" style="white-space:pre-line;padding-top:8px;height:130px;overflow:auto"></div>
          <hr style="border:none;height:1px;background:rgba(255,255,255,0.02);margin:8px 0">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div class="small">Event Log</div>
            <div style="display:flex;gap:6px;align-items:center">
              <button class="btn" id="btnOpen">OPEN</button>
              <button class="btn" id="btnPost">POST</button>
              <button class="btn" id="btnClose">CLOSE</button>
              <div class="auto-toggle btn" id="autoToggle">Auto: ON</div>
            </div>
          </div>
          <div id="events" class="log muted" style="margin-top:8px"></div>
        </div>
      </div>
    </div>

    <div class="card" style="display:flex;justify-content:space-between;gap:12px;align-items:flex-start">
      <div style="flex:1">
        <div class="small">Portfolio (snapshot)</div>
        <div id="portfolioValue" class="value">Rp 0</div>
      </div>
      <div style="text-align:right">
        <div class="small">Dividends received</div>
        <div class="small" id="dividendTotal">Rp 0</div>
      </div>
      <div id="holdings" class="muted small" style="width:100%;margin-top:8px"></div>
    </div>
  </div><!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>ProTrader IDX-Sim — UltraPRO (Hybrid Catch-up)</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#071428;--panel:#0b2130;--muted:#9fb2bd;--accent:#0ea5a4;--up:#16a34a;--down:#ef4444;
  --glass:rgba(255,255,255,0.02);--radius:12px;--shadow:0 10px 30px rgba(0,0,0,0.6);
  --heat-up:#064e3b;--heat-down:#6b021f;
}
*{box-sizing:border-box}
body{margin:0;background:linear-gradient(180deg,var(--bg) 0%, #02121a 100%);color:#e6eef6;font-family:Inter,system-ui,Arial; -webkit-font-smoothing:antialiased}
.app{max-width:1360px;margin:12px auto;padding:12px;display:grid;grid-template-columns:300px 1fr 380px;gap:12px}
.header{grid-column:1/-1;display:flex;align-items:center;gap:12px}
.brand{font-weight:700;font-size:18px}
.subtitle{color:var(--muted);font-size:13px}
.card{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));border-radius:var(--radius);padding:12px;box-shadow:var(--shadow)}
.left{display:flex;flex-direction:column;gap:12px}
.wallet{display:flex;justify-content:space-between;align-items:center}
.label{color:var(--muted);font-size:13px}
.value{font-weight:700;font-size:18px}
.search{display:flex;gap:8px;margin-top:10px}
.search input{flex:1;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:#e6eef6}
.asset-list{margin-top:10px;display:flex;flex-direction:column;gap:6px;max-height:520px;overflow:auto;padding-right:6px}
.asset-row{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:10px;background:var(--glass);cursor:pointer;transition:transform .08s}
.asset-row:hover{transform:translateY(-3px)}
.asset-row.active{outline:1px solid rgba(255,255,255,0.03);box-shadow:0 6px 18px rgba(0,0,0,0.6)}
.center{display:flex;flex-direction:column;gap:12px}
.chart-wrap{height:520px;border-radius:12px;overflow:hidden;position:relative;background:linear-gradient(180deg,rgba(255,255,255,0.01),transparent)}
canvas{display:block;width:100%;height:100%}
.chart-head{display:flex;justify-content:space-between;align-items:center;padding:8px}
.ticker{font-weight:700;font-size:16px}
.ticker-sub{font-family:monospace;color:#cfeff1}
.controls-row{display:flex;gap:8px;align-items:center}
.input,select{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px;border-radius:8px;color:#e8f6f5}
.btn{background:var(--glass);border:1px solid rgba(255,255,255,0.03);padding:8px;border-radius:8px;color:var(--muted);cursor:pointer}
.btn.primary{background:linear-gradient(180deg,var(--accent),#0891a6);border:none;color:#001;font-weight:700}
.right{display:flex;flex-direction:column;gap:12px}
.table{width:100%;border-collapse:collapse}
.log{max-height:260px;overflow:auto;padding:8px;background:rgba(0,0,0,0.04);border-radius:8px}
.muted{color:var(--muted)}
.footer{grid-column:1/-1;text-align:right;color:var(--muted);font-size:12px;margin-top:6px}
.session-pill{padding:6px 10px;border-radius:8px;font-weight:700}
.session-open{background:#064e3b;color:#d6fde8}
.session-post{background:#6b4b00;color:#fff6d6}
.session-close{background:#6b021f;color:#ffd6dc}
.heatmap{display:grid;grid-template-columns:repeat(3,1fr);gap:6px}
.heat-tile{padding:8px;border-radius:8px;font-size:12px;text-align:center}
.gold-mode{box-shadow:0 0 24px rgba(255,215,0,0.12);border:1px solid rgba(255,215,0,0.06)}
/* responsive */
@media (max-width:1100px){.app{grid-template-columns:1fr;}.right{order:3}}
</style>
</head>
<body>
<div class="app">
  <div class="header">
    <div>
      <div class="brand" id="brandTitle">ProTrader IDX-Sim • UltraPRO (IDX Mobile style)</div>
      <div class="subtitle">Hybrid Real-Time • Catch-up • No server required</div>
    </div>
    <div style="flex:1"></div>
    <div class="muted" id="timeDisplay">—</div>
  </div>

  <!-- LEFT -->
  <div class="left card">
    <div class="wallet">
      <div>
        <div class="label">Saldo (Cash)</div>
        <div class="value" id="cash">Rp 0</div>
      </div>
      <div style="text-align:right">
        <div class="label">Total Aset</div>
        <div class="value" id="totalAssets">Rp 0</div>
      </div>
    </div>

    <div style="display:flex;justify-content:space-between;align-items:center">
      <div class="small">Daftar Saham (IDX Sim)</div>
      <div id="marketStatusBar" class="session-pill session-open small">OPEN</div>
    </div>

    <div class="search">
      <input id="searchInput" placeholder="Cari saham (nama/kode)..." />
      <button class="btn" id="filterBtn">Filter</button>
    </div>

    <div class="asset-list card" id="assetList" aria-label="Daftar aset"></div>

    <div style="display:flex;justify-content:space-between;margin-top:10px">
      <button class="btn" id="resetBtn">Reset</button>
      <button class="btn" id="saveBtn">Save</button>
    </div>

    <div style="margin-top:10px" class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Market Overview</div>
        <div><button class="btn" id="btnHeat">Heatmap</button></div>
      </div>
      <div id="heatWrap" class="heatmap" style="margin-top:8px"></div>
    </div>

    <div style="margin-top:10px" class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="small">Challenge Mode</div>
        <div><button class="btn" id="btnChallenge">Start</button></div>
      </div>
      <div class="small muted" style="margin-top:8px">Goal: Grow modal dari Rp100.000.000 → Rp1.000.000.000</div>
    </div>
  </div>

  
