```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>战术小队：浏览器战术FPS</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; user-select:none; font-family:'Segoe UI', Arial, sans-serif; }
  body { background:#000; overflow:hidden; width:100vw; height:100vh; }
  canvas#game { display:block; width:100%; height:100%; }

  /* 全屏覆盖层 */
  .overlay { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(10,12,18,0.88); color:#ddd; display:flex; flex-direction:column; align-items:center; justify-content:center; z-index:50; }
  .hidden { display:none !important; }
  .panel { background:#161b26; border:1px solid #2b3448; border-radius:12px; padding:28px 34px; width:min(680px, 94vw); box-shadow:0 0 40px rgba(0,0,0,.7); }
  h1 { font-size:30px; letter-spacing:2px; color:#e2e8f0; margin-bottom:6px; }
  h2 { font-size:22px; color:#cbd5e1; margin-bottom:18px; }
  h3 { color:#8fa3bf; margin-top:12px; margin-bottom:6px; font-size:16px; border-bottom:1px solid #2b3448; padding-bottom:4px; }
  .row { display:flex; gap:12px; flex-wrap:wrap; justify-content:center; margin-top:12px; }
  .btn { background:#263045; color:#e2e8f0; border:1px solid #3b4c6b; padding:9px 16px; border-radius:8px; cursor:pointer; font-size:14px; transition:.15s; }
  .btn:hover { background:#31415e; }
  .btn.selected { background:#31588f; border-color:#6b9cff; color:#fff; }
  .btn.small { padding:5px 10px; font-size:13px; }
  .btn.buy { display:block; width:100%; text-align:left; margin:5px 0; font-size:14px; }
  .grid2 { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
  .money { color:#ffd166; }

  /* HUD */
  #hud { position:fixed; inset:0; pointer-events:none; z-index:10; }
  #topbar { position:absolute; top:8px; left:50%; transform:translateX(-50%); background:rgba(0,0,0,0.45); border-radius:8px; padding:4px 18px; color:#fff; text-align:center; font-size:15px; line-height:1.4; }
  #scoreT { color:#f4a261; font-weight:bold; }
  #scoreCT { color:#5aa9ff; font-weight:bold; }
  #bottomleft { position:absolute; left:16px; bottom:16px; width:220px; background:rgba(0,0,0,0.55); border-radius:10px; padding:10px 14px; color:#fff; font-size:14px; line-height:1.5; }
  #ammo { font-size:22px; font-weight:bold; }
  #weaponName { color:#a7c0e8; font-size:13px; }
  #armor { color:#7fd1ff; }
  #crosshair { position:absolute; left:50%; top:50%; transform:translate(-50%, -50%); pointer-events:none; }
  #crosshair .dot { width:3px; height:3px; background:#0f0; position:absolute; left:-1px; top:-1px; box-shadow:0 0 3px #000; }
  #crosshair .bar { background:rgba(0,255,0,0.8); position:absolute; box-shadow:0 0 3px #000; }
  #crosshair .tl { width:8px; height:1px; left:-10px; top:-1px; }
  #crosshair .tr { width:8px; height:1px; left:2px; top:-1px; }
  #crosshair .bl { width:8px; height:1px; left:-10px; top:0; }
  #crosshair .br { width:8px; height:1px; left:2px; top:0; }
  #minimap { position:absolute; left:10px; top:8px; border:2px solid rgba(255,255,255,0.25); border-radius:6px; background:rgba(0,0,0,0.45); pointer-events:none; }
  #killfeed { position:absolute; right:16px; top:10px; width:280px; font-size:13px; color:#fff; text-align:right; }
  .kill-item { background:rgba(0,0,0,0.45); border-radius:4px; padding:2px 8px; margin-bottom:3px; }
  #centerBanner { position:absolute; left:50%; top:36%; transform:translate(-50%,-50%); font-size:34px; font-weight:bold; letter-spacing:4px; color:#fff; text-shadow:0 0 12px #000; opacity:0; transition:opacity .2s; text-align:center; }
  #interact { position:absolute; left:50%; bottom:26%; transform:translateX(-50%); color:#ffd166; background:rgba(0,0,0,0.5); padding:6px 16px; border-radius:30px; font-size:15px; text-align:center; pointer-events:none; }
  #interact .bar { width:180px; height:6px; background:#333; margin-top:5px; border-radius:3px; overflow:hidden; margin-left:auto; margin-right:auto; }
  #interact .fill { height:100%; background:#ffd166; width:0%; }
  #scope { position:fixed; inset:0; pointer-events:none; z-index:20; background:radial-gradient(circle, transparent 24%, rgba(0,0,0,0.98) 25%); }
  #scope::before, #scope::after { content:''; position:absolute; background:rgba(0,0,0,0.9); }
  #scope::before { left:50%; top:0; width:1px; height:100%; transform:translateX(-50%); }
  #scope::after { top:50%; left:0; height:1px; width:100%; transform:translateY(-50%); }

  #buyMenu { z-index:60; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(5,8,12,0.78); display:flex; align-items:center; justify-content:center; }
  #buyMenu .panel { max-height:90vh; overflow:auto; }
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="hud">
  <div id="topbar">
    <div style="display:flex; gap:16px; align-items:center;">
      <span>回合 <span id="roundNum">1</span></span>
      <span id="scoreDisplay"><span id="scoreT">0</span> : <span id="scoreCT">0</span></span>
      <span id="timer">5.0</span>
    </div>
  </div>
  <canvas id="minimap" width="170" height="170"></canvas>
  <div id="bottomleft">
    <div><span id="hp">100</span> HP</div>
    <div id="armor">无护甲</div>
    <div id="ammo">--</div>
    <div id="weaponName">--</div>
  </div>
  <div id="crosshair">
    <div class="dot"></div>
    <div class="bar tl"></div><div class="bar tr"></div>
    <div class="bar bl"></div><div class="bar br"></div>
  </div>
  <div id="killfeed"></div>
  <div id="centerBanner"></div>
  <div id="interact"></div>
  <div id="scope" class="hidden"></div>
</div>

<div id="startMenu" class="overlay">
  <div class="panel" style="text-align:center;">
    <h1>战术小队</h1>
    <h2>浏览器战术FPS</h2>
    <h3>地图</h3>
    <div class="row">
      <button class="btn map-btn selected" data-map="dust2">Dust2 复刻</button>
      <button class="btn map-btn" data-map="station">冰雪枢纽</button>
      <button class="btn map-btn" data-map="bazaar">集市暗巷</button>
    </div>
    <h3>阵营</h3>
    <div class="row">
      <button class="btn team-btn selected" data-team="T">恐怖分子</button>
      <button class="btn team-btn" data-team="CT">反恐精英</button>
    </div>
    <h3>Bot 设置</h3>
    <div class="row">
      <span>每队Bot数</span>
      <select id="botCount" class="btn">
        <option value="1">1</option>
        <option value="2" selected>2</option>
        <option value="3">3</option>
        <option value="4">4</option>
      </select>
      <span>难度</span>
      <select id="difficulty" class="btn">
        <option value="easy">简单</option>
        <option value="normal" selected>普通</option>
        <option value="hard">困难</option>
      </select>
    </div>
    <br>
    <button class="btn" id="startBtn" style="font-size:18px; padding:12px 36px; margin-top:10px;">开始对局</button>
    <div style="color:#7d8aa0; font-size:12px; margin-top:14px;">拖拽鼠标 / WASD移动 / Shift冲刺 / Ctrl下蹲 / 空格跳跃 / E互动 / R换弹 / B购买 / 1-8切换武器</div>
  </div>
</div>

<div id="pauseMenu" class="overlay hidden">
  <div class="panel" style="text-align:center;">
    <h2>已暂停</h2>
    <div class="row">
      <button class="btn" id="resumeBtn">继续</button>
      <button class="btn" id="restartBtn">重新开始</button>
      <button class="btn" id="mainMenuBtn">主菜单</button>
    </div>
  </div>
</div>

<div id="buyMenu" class="hidden">
  <div class="panel">
    <h2>购买菜单 <span id="buyMoney" class="money" style="float:right;"></span></h2>
    <div id="buyList"></div>
    <div class="row"><button class="btn" id="closeBuyBtn">关闭购买菜单</button></div>
  </div>
</div>

<script type="importmap">
{ "imports": { "three": "https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.module.js" } }
</script>
<script type="module">
import * as THREE from 'three';

// ----------------------------------------------------------------
// 工具函数
// ----------------------------------------------------------------
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const rand = (a, b) => a + Math.random() * (b - a);
const dist = (a, b) => a ? a.distanceTo(b) : 999;
const dist2D = (ax, az, bx, bz) => Math.hypot(ax - bx, az - bz);

// ----------------------------------------------------------------
// 程序化贴图
// ----------------------------------------------------------------
function makeTexture(base, kind = 'wall') {
  const c = document.createElement('canvas');
  c.width = 256; c.height = 256;
  const g = c.getContext('2d');
  g.fillStyle = base; g.fillRect(0, 0, 256, 256);
  for (let i = 0; i < 5000; i++) {
    g.fillStyle = `rgba(${Math.random()*60-30|0},${Math.random()*50-25|0},${Math.random()*45-20|0},0.15)`;
    g.fillRect(Math.random()*256, Math.random()*256, 2, 2);
  }
  if (kind === 'wall') {
    const yCount = 8, h = 32;
    for (let y = 0; y < yCount; y++) {
      const yy = y * h;
      g.strokeStyle = 'rgba(0,0,0,0.25)';
      g.lineWidth = 2;
      g.beginPath(); g.moveTo(0, yy); g.lineTo(256, yy); g.stroke();
      for (let x = (y % 2) * 40; x < 256; x += 80) {
        g.beginPath(); g.moveTo(x, yy); g.lineTo(x, yy + h); g.stroke();
      }
    }
  }
  if (kind === 'crate') {
    g.fillStyle = 'rgba(0,0,0,0.2)';
    g.fillRect(20, 20, 216, 216);
    for (let i = 0; i < 6; i++) {
      g.strokeStyle = 'rgba(0,0,0,0.35)';
      g.beginPath(); g.moveTo(40, 30 + i * 30); g.lineTo(216, 10 + i * 30); g.stroke();
    }
  }
  const tex = new THREE.CanvasTexture(c);
  tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
  tex.repeat.set(2, 2);
  return tex;
}

// ----------------------------------------------------------------
// 地图网格
// ----------------------------------------------------------------
const DUST2_GRID = [
"####################",
"#..................#",
"#..............A...#",
"#..............A...#",
"#..#####.######...#",
"#..................#",
"#..................#",
"#..................#",
"#..................#",
"#.T..............C.#",
"#.T..............C.#",
"#..##############..#",
"#..##############..#",
"#..##############..#",
"#..##############..#",
"#..##############..#",
"#..##############..#",
"#................B.#",
"#..................#",
"####################"
];

const STATION_GRID = [
"##################",
"#................#",
"#..A..........B..#",
"#................#",
"#................#",
"#....##....##....#",
"#....##....##....#",
"#................#",
"#................#",
"#................#",
"#................#",
"#....##....##....#",
"#....##....##....#",
"#................#",
"#..T..........C..#",
"#................#",
"#................#",
"##################"
];

const BAZAAR_GRID = [
"##################",
"#................#",
"#..T..........B..#",
"#..##..####..##..#",
"#................#",
"#................#",
"#..##..####..##..#",
"#................#",
"#....########....#",
"#................#",
"#....########....#",
"#................#",
"#..##..####..##..#",
"#................#",
"#..A..........C..#",
"#................#",
"#................#",
"##################"
];

const MAPS = {
  dust2: {
    id: 'dust2', name: 'Dust2 复刻', grid: DUST2_GRID, cellSize: 6, wallHeight: 5,
    sky: 0xc9b28c, fog: 0xc0a985, ground: '#b59a6b', wall: '#a98f5f', crate: '#8a704c',
    coverCells: [
      { c:15, r:3, dx:0.8, dz:0.6, sx:2.6, sz:1.7, h:1.2 },
      { c:16, r:3, dx:-0.6, dz:0.2, sx:2.0, sz:2.4, h:0.9 },
      { c:15, r:2, dx:0.0, dz:-0.6, sx:2.2, sz:1.5, h:1.4 },
      { c:17, r:2, dx:0.2, dz:0.0, sx:1.4, sz:2.2, h:0.8 },
      { c:7, r:2, dx:0.0, dz:0.0, sx:2.0, sz:1.2, h:1.0 },
      { c:10, r:3, dx:0.2, dz:-0.2, sx:1.6, sz:1.6, h:0.7 },
      { c:9, r:9, dx:0.0, dz:0.0, sx:1.5, sz:1.5, h:1.0 },
      { c:17, r:17, dx:0.0, dz:0.6, sx:1.8, sz:2.0, h:1.1 },
      { c:16, r:17, dx:0.5, dz:-0.6, sx:2.2, sz:1.6, h:0.8 },
      { c:16, r:18, dx:0.0, dz:0.0, sx:2.0, sz:1.0, h:1.0 }
    ]
  },
  station: {
    id: 'station', name: '冰雪枢纽', grid: STATION_GRID, cellSize: 7, wallHeight: 5,
    sky: 0x7a8a9a, fog: 0x6d7c8c, ground: '#6f7d8a', wall: '#5c6875', crate: '#4b5460',
    coverCells: [
      { c:5, r:2, dx:0, dz:0, sx:2.4, sz:1.6, h:1.0 },
      { c:13, r:2, dx:0, dz:0, sx:2.4, sz:1.6, h:1.0 },
      { c:7, r:7, dx:0, dz:0, sx:2.0, sz:2.0, h:1.2 },
      { c:10, r:10, dx:0, dz:0, sx:2.0, sz:2.0, h:1.2 }
    ]
  },
  bazaar: {
    id: 'bazaar', name: '集市暗巷', grid: BAZAAR_GRID, cellSize: 7, wallHeight: 5,
    sky: 0x3a4658, fog: 0x2c3442, ground: '#4a4540', wall: '#49423b', crate: '#5d4f3a',
    coverCells: [
      { c:4, r:2, dx:0, dz:0, sx:1.8, sz:1.4, h:1.1 },
      { c:13, r:2, dx:0, dz:0, sx:1.8, sz:1.4, h:1.1 },
      { c:5, r:8, dx:0, dz:0, sx:2.2, sz:1.2, h:1.0 },
      { c:12, r:8, dx:0, dz:0, sx:2.2, sz:1.2, h:1.0 }
    ]
  }
};

// ----------------------------------------------------------------
// 武器与装备定义
// ----------------------------------------------------------------
const WEAPONS = {
  knife: { name:'战术刀', kind:'melee', damage:50, rate:500, price:0 },
  glock: { name:'Glock-18', kind:'pistol', damage:28, rate:400, mag:20, reserve:120, price:0, reload:2.3, auto:false, baseSpread:0.015, spreadInc:0.006, recoil:0.012, killReward:300, pen:1 },
  usp: { name:'USP-S', kind:'pistol', damage:31, rate:350, mag:12, reserve:72, price:0, reload:2.4, auto:false, baseSpread:0.012, spreadInc:0.004, recoil:0.010, killReward:300, pen:1 },
  p250: { name:'P250', kind:'pistol', damage:34, rate:400, mag:13, reserve:78, price:300, reload:2.4, auto:false, baseSpread:0.014, spreadInc:0.006, recoil:0.013, killReward:300, pen:1 },
  deagle: { name:'Desert Eagle', kind:'pistol', damage:55, rate:267, mag:7, reserve:35, price:700, reload:2.6, auto:false, baseSpread:0.02, spreadInc:0.02, recoil:0.05, killReward:300, pen:2 },
  mac10: { name:'MAC-10', kind:'smg', damage:24, rate:720, mag:30, reserve:90, price:1050, reload:2.1, auto:true, baseSpread:0.025, spreadInc:0.012, recoil:0.02, killReward:600, pen:1 },
  mp9: { name:'MP9', kind:'smg', damage:26, rate:750, mag:30, reserve:90, price:1250, reload:2.1, auto:true, baseSpread:0.024, spreadInc:0.012, recoil:0.019, killReward:600, pen:1 },
  nova: { name:'Nova', kind:'shotgun', damage:12, pellets:9, rate:68, mag:8, reserve:32, price:1050, reload:3.6, auto:false, baseSpread:0.03, spreadInc:0.03, recoil:0.05, killReward:900, pen:1 },
  galil: { name:'Galil AR', kind:'rifle', damage:30, rate:600, mag:35, reserve:90, price:2000, reload:2.6, auto:true, baseSpread:0.016, spreadInc:0.01, recoil:0.022, killReward:300, pen:2 },
  famas: { name:'FAMAS', kind:'rifle', damage:30, rate:660, mag:25, reserve:90, price:2250, reload:2.8, auto:true, baseSpread:0.015, spreadInc:0.01, recoil:0.021, killReward:300, pen:2 },
  ak47: { name:'AK-47', kind:'rifle', damage:36, rate:600, mag:30, reserve:90, price:2700, reload:2.5, auto:true, baseSpread:0.014, spreadInc:0.011, recoil:0.03, killReward:300, pen:2 },
  m4a4: { name:'M4A4', kind:'rifle', damage:33, rate:666, mag:30, reserve:90, price:3100, reload:2.8, auto:true, baseSpread:0.014, spreadInc:0.01, recoil:0.026, killReward:300, pen:2 },
  awp: { name:'AWP', kind:'sniper', damage:110, rate:40, mag:5, reserve:30, price:4750, reload:3.8, auto:false, baseSpread:0.0, spreadInc:0.0, recoil:0.09, killReward:100, pen:3, scope:true }
};

const GRENADES = {
  he: { name:'高爆手雷', price:300, kind:'he', fuse:1.8 },
  flash: { name:'闪光弹', price:200, kind:'flash', fuse:1.5 },
  smoke: { name:'烟雾弹', price:300, kind:'smoke', fuse:2.0 },
  molotov: { name:'燃烧瓶', price:400, kind:'molotov', fuse:2.0 }
};

const BUY_ITEMS = [
  { cat:'手枪', items:['glock','usp','p250','deagle'] },
  { cat:'主武器', items:['nova','mac10','mp9','galil','famas','ak47','m4a4','awp'] },
  { cat:'装备', items:['armor','helmet','defuseKit'] },
  { cat:'投掷物', items:['he','flash','smoke','molotov'] }
];

function getPrice(itemId, team) {
  if (itemId === 'glock') return team === 'T' ? 0 : 200;
  if (itemId === 'usp') return team === 'CT' ? 0 : 200;
  if (itemId === 'molotov') return team === 'T' ? 400 : 600;
  if (WEAPONS[itemId]) return WEAPONS[itemId].price || 0;
  if (GRENADES[itemId]) return GRENADES[itemId].price || 0;
  if (itemId === 'armor') return 650;
  if (itemId === 'helmet') return 350;
  if (itemId === 'defuseKit') return 400;
  return 0;
}

function isTeamWeapon(itemId, team) {
  if (itemId === 'ak47' || itemId === 'mac10' || itemId === 'galil') return team === 'T';
  if (itemId === 'm4a4' || itemId === 'famas' || itemId === 'mp9') return team === 'CT';
  if (itemId === 'glock') return team === 'T';
  if (itemId === 'usp') return team === 'CT';
  return true;
}

// ----------------------------------------------------------------
// 音频系统
// ----------------------------------------------------------------
class AudioSystem {
  constructor() { this.ctx = null; }
  resume() {
    if (!this.ctx) this.ctx = new (window.AudioContext || window.webkitAudioContext)();
    if (this.ctx.state === 'suspended') this.ctx.resume();
  }
  noise(duration, type = 'lowpass', freq = 1000, gain = 0.4) {
    if (!this.ctx) return;
    const n = this.ctx.sampleRate * duration;
    const buf = this.ctx.createBuffer(1, n, this.ctx.sampleRate);
    const ch = buf.getChannelData(0);
    for (let i = 0; i < n; i++) ch[i] = (Math.random()*2-1) * (1 - i / n);
    const src = this.ctx.createBufferSource(); src.buffer = buf;
    const filter = this.ctx.createBiquadFilter(); filter.type = type; filter.frequency.value = freq;
    const g = this.ctx.createGain(); g.gain.value = gain;
    src.connect(filter); filter.connect(g); g.connect(this.ctx.destination);
    src.start();
  }
  tone(freq, duration, gain = 0.2, type = 'square') {
    if (!this.ctx) return;
    const o = this.ctx.createOscillator(); o.type = type; o.frequency.value = freq;
    const g = this.ctx.createGain(); g.gain.value = gain;
    o.connect(g); g.connect(this.ctx.destination);
    o.start(); o.stop(this.ctx.currentTime + duration);
    g.gain.setValueAtTime(gain, this.ctx.currentTime);
    g.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + duration);
  }
  shoot(w) {
    const t = w ? (w.kind === 'pistol' ? 0.09 : w.kind === 'sniper' ? 0.15 : 0.11) : 0.1;
    this.noise(t, 'bandpass', w && w.kind === 'sniper' ? 400 : 1200, 0.45);
    this.tone(180, 0.07, 0.1, 'sawtooth');
  }
  reload() { this.noise(0.08, 'highpass', 2500, 0.15); setTimeout(()=>this.noise(0.1,'highpass',2000,0.18), 120); setTimeout(()=>this.noise(0.08,'lowpass',900,0.2), 450); }
  footstep() { this.noise(0.04, 'lowpass', 600, 0.05); }
  explosion() { this.noise(0.8, 'lowpass', 120, 0.7); this.tone(60, 0.6, 0.3, 'sawtooth'); }
  flashbang() { this.tone(1200, 0.5, 0.15, 'sine'); }
  smoke() { this.noise(0.7, 'lowpass', 300, 0.2); }
  plant() { this.tone(900, 0.1, 0.1); setTimeout(()=>this.tone(700,0.1,0.1),250); setTimeout(()=>this.tone(500,0.2,0.1),500); }
  defuse() { this.tone(500, 0.15, 0.08); setTimeout(()=>this.tone(700,0.15,0.08),250); setTimeout(()=>this.tone(900,0.3,0.1),500); }
  buy() { this.tone(660, 0.12, 0.12); setTimeout(()=>this.tone(990, 0.18, 0.12), 100); }
  roundWin() { this.tone(523, 0.18, 0.15); setTimeout(()=>this.tone(659,0.18,0.15),150); setTimeout(()=>this.tone(784,0.3,0.15),300); }
  roundLose() { this.tone(392, 0.3, 0.12, 'triangle'); setTimeout(()=>this.tone(311,0.3,0.1,'triangle'),200); }
  hit() { this.noise(0.08, 'bandpass', 2000, 0.35); }
  headshot() { this.noise(0.12, 'bandpass', 3000, 0.4); }
}
const audio = new AudioSystem();

// ----------------------------------------------------------------
// 主游戏类
// ----------------------------------------------------------------
class Game {
  constructor() {
    this.renderer = new THREE.WebGLRenderer({ canvas: document.getElementById('game'), antialias: true });
    this.renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.shadowMap.enabled = true;
    this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 500);
    this.camera.rotation.order = 'YXZ';
    this.scene.add(this.camera);

    this.keys = new Set();
    this.mouseDown = false;
    this.ads = false;
    this.isLocked = false;
    this.menuOpen = false;
    this.running = false;

    this.colliders = [];
    this.mapMeshes = [];
    this.shootableMeshes = [];
    this.projectiles = [];
    this.shells = [];
    this.explosions = [];
    this.smokes = [];
    this.fires = [];
    this.bots = [];
    this.mapDef = null;
    this.grid = null;
    this.waypoints = [];
    this.nodeMap = new Map();
    this.bombSites = [];

    this.player = this.createPlayerEntity();
    this.playerBodyGroup = new THREE.Group();
    this.scene.add(this.playerBodyGroup);
    const invMat = new THREE.MeshBasicMaterial({ transparent:true, opacity:0, depthWrite:false });
    const makeBox = (sx, sy, sz, y) => { const m = new THREE.Mesh(new THREE.BoxGeometry(sx, sy, sz), invMat); m.position.y = y; this.playerBodyGroup.add(m); };
    makeBox(0.32, 0.30, 0.30, 1.6);
    makeBox(0.52, 0.62, 0.32, 1.2);
    makeBox(0.32, 0.5, 0.32, 0.65);
    this.playerBodyGroup.traverse(o => { if (o.isMesh) o.userData.entity = this.player; });

    this.bombMesh = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.18, 0.28), new THREE.MeshLambertMaterial({ color: 0x111111 }));
    this.bombMesh.visible = false;
    this.scene.add(this.bombMesh);

    // 枪模
    this.gunGroup = new THREE.Group();
    this.camera.add(this.gunGroup);

    this.lastTime = performance.now();
    this.initInput();
    this.initUI();
    this.loadMap(MAPS.dust2);
    this.animate = this.animate.bind(this);
    requestAnimationFrame(this.animate);
  }

  // ---------- 玩家/实体 ----------
  createPlayerEntity() {
    return {
      name: 'You', team: 'T', pos: new THREE.Vector3(0, 0, 0), vel: new THREE.Vector3(),
      yaw: 0, pitch: 0, health: 100, armor: 0, helmet: false, defuseKit: false,
      money: 800, alive: true, survived: false, hasBomb: false, primary: null, secondary: null,
      activeWeapon: null, reloading: false, reloadTimer: 0, lastFire: 0,
      grenades: { he:0, flash:0, smoke:0, molotov:0 },
      crouching:false, sprinting:false, walking:false
    };
  }

  setPlayerWeapons() {
    const p = this.player;
    p.secondary = JSON.parse(JSON.stringify(WEAPONS[p.team === 'T' ? 'glock' : 'usp']));
    p.activeWeapon = p.secondary;
    p.grenades = { he:0, flash:0, smoke:0, molotov:0 };
    this.updateGunModel();
  }

  // ---------- 输入 ----------
  initInput() {
    window.addEventListener('resize', () => {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    });

    document.addEventListener('keydown', e => {
      if (e.code === 'KeyB') { this.openBuyMenu(); return; }
      this.keys.add(e.code);
      if (e.code === 'KeyR') this.startReload();
      if (e.code === 'Digit1') this.selectSlot('primary');
      if (e.code === 'Digit2') this.selectSlot('secondary');
      if (e.code === 'Digit3') this.selectSlot('knife');
      if (e.code === 'Digit4') this.selectGrenade('he');
      if (e.code === 'Digit5') this.selectGrenade('flash');
      if (e.code === 'Digit6') this.selectGrenade('smoke');
      if (e.code === 'Digit7') this.selectGrenade('molotov');
      if (e.code === 'KeyQ') { this.ads = false; }
    });
    document.addEventListener('keyup', e => this.keys.delete(e.code));

    document.addEventListener('mousemove', e => {
      if (!this.isLocked || this.menuOpen || !this.running) return;
      const p = this.player;
      if (p.dead) return;
      p.yaw -= e.movementX * 0.002;
      p.pitch -= e.movementY * 0.002;
      p.pitch = clamp(p.pitch, -1.55, 1.55);
    });

    document.addEventListener('mousedown', e => {
      if (!this.isLocked || this.menuOpen || !this.running) return;
      if (e.button === 0) this.mouseDown = true;
      if (e.button === 2) { this.ads = true; }
      audio.resume();
    });
    document.addEventListener('mouseup', e => {
      if (e.button === 0) this.mouseDown = false;
      if (e.button === 2) this.ads = false;
    });
    this.renderer.domElement.addEventListener('click', () => {
      if (!this.isLocked && !this.menuOpen && this.running) {
        this.renderer.domElement.requestPointerLock();
      }
    });

    document.addEventListener('pointerlockchange', () => {
      this.isLocked = document.pointerLockElement === this.renderer.domElement;
      if (!this.isLocked && !this.menuOpen && this.running && this.phase !== 'ended') {
        this.showPause();
      }
    });

    document.addEventListener('contextmenu', e => e.preventDefault());
  }

  // ---------- UI绑定 ----------
  initUI() {
    document.querySelectorAll('.map-btn').forEach(b => b.onclick = () => {
      document.querySelectorAll('.map-btn').forEach(x => x.classList.remove('selected'));
      b.classList.add('selected');
      this.pendingMap = b.dataset.map;
    });
    this.pendingMap = 'dust2';
    document.querySelectorAll('.team-btn').forEach(b => b.onclick = () => {
      document.querySelectorAll('.team-btn').forEach(x => x.classList.remove('selected'));
      b.classList.add('selected');
      this.pendingTeam = b.dataset.team;
    });
    this.pendingTeam = 'T';
    document.getElementById('startBtn').onclick = () => {
      const botCount = parseInt(document.getElementById('botCount').value);
      const difficulty = document.getElementById('difficulty').value;
      this.startMatch({ mapId: this.pendingMap, team: this.pendingTeam, botCount, difficulty });
    };
    document.getElementById('resumeBtn').onclick = () => this.closeMenu();
    document.getElementById('restartBtn').onclick = () => {
      const cfg = this.lastConfig; if (cfg) this.startMatch(cfg);
    };
    document.getElementById('mainMenuBtn').onclick = () => this.toMainMenu();
    document.getElementById('closeBuyBtn').onclick = () => this.closeBuyMenu();
    this.buildBuyMenu();
  }

  buildBuyMenu() {
    const list = document.getElementById('buyList');
    list.innerHTML = '';
    for (const cat of BUY_ITEMS) {
      const h = document.createElement('h3'); h.textContent = cat.cat; list.appendChild(h);
      for (const id of cat.items) {
        const btn = document.createElement('button');
        btn.className = 'btn buy';
        btn.id = 'buy-' + id;
        btn.innerHTML = `<span>${this.itemTitle(id)}</span> <span style="float:right" class="money">${this.itemPrice(id)}</span>`;
        btn.onclick = () => this.buyItem(id);
        list.appendChild(btn);
      }
    }
  }

  itemTitle(id) {
    if (WEAPONS[id]) return WEAPONS[id].name;
    if (GRENADES[id]) return GRENADES[id].name;
    if (id === 'armor') return '护甲';
    if (id === 'helmet') return '防弹头盔';
    if (id === 'defuseKit') return '拆弹器';
    return id;
  }
  itemPrice(id) { return getPrice(id, this.player.team); }

  openBuyMenu() {
    if (!this.running || this.phase !== 'freeze' || this.player.dead) return;
    this.menuOpen = true;
    document.getElementById('buyMenu').classList.remove('hidden');
    document.getElementById('buyMoney').textContent = '$' + this.player.money;
    this.updateBuyMenuButtons();
    document.exitPointerLock();
  }

  closeBuyMenu() {
    this.menuOpen = false;
    document.getElementById('buyMenu').classList.add('hidden');
    if (this.running) this.renderer.domElement.requestPointerLock();
  }

  updateBuyMenuButtons() {
    const money = this.player.money;
    for (const cat of BUY_ITEMS) {
      for (const id of cat.items) {
        const btn = document.getElementById('buy-' + id);
        if (!btn) continue;
        const teamOk = isTeamWeapon(id, this.player.team);
        const price = getPrice(id, this.player.team);
        btn.style.opacity = (teamOk && money >= price) ? '1' : '0.4';
      }
    }
  }

  buyItem(id) {
    if (!this.running || this.phase !== 'freeze') return;
    if (!isTeamWeapon(id, this.player.team)) return;
    const price = getPrice(id, this.player.team);
    const p = this.player;
    if (p.money < price) return;
    if (WEAPONS[id]) {
      const w = JSON.parse(JSON.stringify(WEAPONS[id]));
      if (w.kind === 'primary' || w.kind === 'rifle' || w.kind === 'smg' || w.kind === 'shotgun' || w.kind === 'sniper') {
        p.primary = { ...w, mag: w.mag, reserve: w.reserve };
        p.activeWeapon = p.primary;
      } else {
        p.secondary = { ...w, mag: w.mag, reserve: w.reserve };
        p.activeWeapon = p.secondary;
      }
      this.updateGunModel();
    } else if (GRENADES[id]) {
      p.grenades[id] = (p.grenades[id] || 0) + 1;
      this.selectGrenade(id);
    } else if (id === 'armor') {
      p.armor = 100; // 有护甲
    } else if (id === 'helmet') {
      p.helmet = true; p.armor = 100;
    } else if (id === 'defuseKit') {
      p.defuseKit = true;
    }
    p.money -= price;
    audio.buy();
    this.updateBuyMenuButtons();
    this.updateHUD();
  }

  // ---------- 地图 ----------
  loadMap(def) {
    this.mapDef = def;
    this.grid = def.grid;
    const cols = Math.max(...this.grid.map(r => r.length));
    const rows = this.grid.length;
    const cell = def.cellSize;
    this.cellSize = cell;
    this.mapWidth = cols * cell;
    this.mapHeight = rows * cell;

    // 删除旧世界
    const toRemove = [];
    for (const child of this.scene.children) {
      if (child !== this.camera && child !== this.playerBodyGroup && child !== this.bombMesh) toRemove.push(child);
    }
    toRemove.forEach(c => this.scene.remove(c));
    this.colliders = [];
    this.mapMeshes = [];
    this.waypoints = [];
    this.nodeMap.clear();
    this.bombSites = [];
    this.projectiles = [];
    this.shells = [];
    this.explosions = [];
    this.smokes = [];
    this.fires = [];

    this.scene.background = new THREE.Color(def.sky);
    this.scene.fog = new THREE.Fog(def.fog, 40, 180);

    const light = new THREE.DirectionalLight(0xfff5e0, 1.2);
    light.position.set(40, 60, 20);
    light.castShadow = true;
    light.shadow.mapSize.set(1024, 1024);
    light.shadow.camera.left = -80; light.shadow.camera.right = 80;
    light.shadow.camera.top = 80; light.shadow.camera.bottom = -80;
    this.scene.add(light);
    this.scene.add(new THREE.AmbientLight(0x88aaff, 0.45));
    const hemi = new THREE.HemisphereLight(0xddeeff, 0x504030, 0.55);
    this.scene.add(hemi);

    // 地面
    const floorMat = new THREE.MeshLambertMaterial({ map: makeTexture(def.ground, 'wall') });
    const floor = new THREE.Mesh(new THREE.PlaneGeometry(this.mapWidth, this.mapHeight), floorMat);
    floor.rotation.x = -Math.PI / 2;
    floor.receiveShadow = true;
    this.scene.add(floor);
    this.mapMeshes.push(floor);

    const wallMat = new THREE.MeshLambertMaterial({ map: makeTexture(def.wall, 'wall') });
    const crateMat = new THREE.MeshLambertMaterial({ map: makeTexture(def.crate, 'crate') });

    const cellPos = (c, r) => {
      const x = (c + 0.5) * cell - this.mapWidth / 2;
      const z = (r + 0.5) * cell - this.mapHeight / 2;
      return new THREE.Vector3(x, 0, z);
    };

    for (let r = 0; r < rows; r++) {
      const line = this.grid[r];
      if (!line) continue;
      for (let c = 0; c < line.length; c++) {
        const ch = line[c];
        if (ch === '#') {
          const pos = cellPos(c, r);
          const h = def.wallHeight;
          const mesh = new THREE.Mesh(new THREE.BoxGeometry(cell, h, cell), wallMat);
          mesh.position.set(pos.x, h / 2, pos.z);
          mesh.castShadow = true; mesh.receiveShadow = true;
          this.scene.add(mesh);
          this.mapMeshes.push(mesh);
          this.colliders.push({
            min: new THREE.Vector3(pos.x - cell/2, 0, pos.z - cell/2),
            max: new THREE.Vector3(pos.x + cell/2, h, pos.z + cell/2)
          });
        }
        if (ch === 'T') this.mapDef._tSpawns ? this.mapDef._tSpawns.push(cellPos(c, r)) : (this.mapDef._tSpawns = [cellPos(c, r)]);
        if (ch === 'C') this.mapDef._cSpawns ? this.mapDef._cSpawns.push(cellPos(c, r)) : (this.mapDef._cSpawns = [cellPos(c, r)]);
        if (ch === 'A' || ch === 'B') this.bombSites.push({ id: ch, center: cellPos(c, r) });
        if (ch !== '#') {
          const node = { col:c, row:r, pos: cellPos(c, r), conns: [] };
          const key = c + ',' + r;
          if (!this.nodeMap.has(key)) { this.nodeMap.set(key, node); this.waypoints.push(node); }
        }
      }
    }

    // 连接路点
    for (const node of this.waypoints) {
      const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
      for (const [dc, dr] of dirs) {
        const nc = node.col + dc, nr = node.row + dr;
        if (nc < 0 || nr < 0 || nr >= rows || nc >= (this.grid[nr] ? this.grid[nr].length : 0)) continue;
        if (this.grid[nr][nc] !== '#') {
          const key = nc + ',' + nr;
          const other = this.nodeMap.get(key);
          if (other) node.conns.push(other);
        }
      }
    }

    // 防弹箱/掩体
    if (def.coverCells) {
      for (const cv of def.coverCells) {
        const base = cellPos(cv.c, cv.r);
        const x = base.x + (cv.dx || 0);
        const z = base.z + (cv.dz || 0);
        const h = cv.h;
        const sx = cv.sx, sz = cv.sz;
        const mesh = new THREE.Mesh(new THREE.BoxGeometry(sx, h, sz), crateMat);
        mesh.position.set(x, h/2, z);
        mesh.castShadow = true; mesh.receiveShadow = true;
        this.scene.add(mesh);
        this.mapMeshes.push(mesh);
        this.colliders.push({
          min: new THREE.Vector3(x - sx/2, 0, z - sz/2),
          max: new THREE.Vector3(x + sx/2, h, z + sz/2)
        });
      }
    }

    // 合并站点中心
    if (this.bombSites.length) {
      const centers = {};
      for (const site of this.bombSites) {
        if (!centers[site.id]) centers[site.id] = { x:0, z:0, n:0 };
        centers[site.id].x += site.center.x;
        centers[site.id].z += site.center.z;
        centers[site.id].n++;
      }
      this.bombSites = Object.keys(centers).map(id => ({
        id, center: new THREE.Vector3(centers[id].x / centers[id].n, 0, centers[id].z / centers[id].n), radius: 8
      }));
    }

    if (!this.mapDef._tSpawns || !this.mapDef._cSpawns) {
      this.waypoints.sort((a, b) => a.pos.x - b.pos.x);
      this.mapDef._tSpawns = [this.waypoints[0]?.pos || new THREE.Vector3()];
      this.mapDef._cSpawns = [this.waypoints[this.waypoints.length-1]?.pos || new THREE.Vector3()];
    }

    this.shootableMeshes = [...this.mapMeshes, this.bombMesh];
  }

  nearestNode(pos) {
    let best = null, bestD = 1e9;
    for (const n of this.waypoints) {
      const d = pos.distanceToSquared(n.pos);
      if (d < bestD) { bestD = d; best = n; }
    }
    return best;
  }

  findPath(startPos, targetNode) {
    if (!targetNode) return [];
    const start = this.nearestNode(startPos);
    if (!start) return [];
    const queue = [start];
    const came = new Map([[start, null]]);
    const seen = new Set([start]);
    while (queue.length) {
      const q = queue.shift();
      if (q === targetNode) {
        const path = [];
        let cur = q;
        while (cur) { path.unshift(cur); cur = came.get(cur); }
        return path;
      }
      for (const nb of q.conns) {
        if (!seen.has(nb)) {
          seen.add(nb);
          came.set(nb, q);
          queue.push(nb);
        }
      }
    }
    return [];
  }

  // ---------- 对局流程 ----------
  startMatch(cfg) {
    this.lastConfig = cfg;
    this.running = true;
    this.menuOpen = false;
    this.player.team = cfg.team;
    this.player.survived = false;
    this.player.money = 800;
    this.player.primary = null;
    this.player.grenades = { he:0, flash:0, smoke:0, molotov:0 };
    this.scores = { T:0, CT:0 };
    this.lossBonus = { T:1400, CT:1400 };
    this.roundNumber = 0;
    this.matchOver = false;
    this.difficulty = cfg.difficulty;
    this.difficultySettings = {
      easy: { reaction: [0.8, 1.3], accuracy: 0.18, spread: 0.06, throwChance: 0.02 },
      normal: { reaction: [0.4, 0.8], accuracy: 0.45, spread: 0.03, throwChance: 0.05 },
      hard: { reaction: [0.15, 0.35], accuracy: 0.75, spread: 0.01, throwChance: 0.09 }
    }[cfg.difficulty];

    // 创建bot
    for (const b of this.bots) { this.scene.remove(b.group); }
    this.bots = [];
    const botCount = cfg.botCount;
    for (let i = 0; i < botCount; i++) { this.createBot('T', i); this.createBot('CT', i); }
    for (const b of this.bots) { this.scene.add(b.group); this.shootableMeshes.push(b.group); }

    this.playerBodyGroup.traverse(o => { if (o.isMesh) o.userData.entity = this.player; });
    this.shootableMeshes.push(this.playerBodyGroup);

    this.loadMap(MAPS[cfg.mapId] || MAPS.dust2);
    this.player.team = cfg.team;
    this.setPlayerWeapons();
    this.startRound();

    document.getElementById('startMenu').classList.add('hidden');
    document.getElementById('pauseMenu').classList.add('hidden');
    document.getElementById('buyMenu').classList.add('hidden');
    this.renderer.domElement.requestPointerLock();
  }

  createBot(team, index) {
    const bot = {
      name: (team === 'T' ? 'T' : 'CT') + 'Bot' + (index+1),
      team, index,
      pos: new THREE.Vector3(), vel: new THREE.Vector3(),
      yaw: 0, pitch: 0,
      health: 100, armor: 0, helmet: false, defuseKit: team === 'CT',
      money: 800, alive: true, survived: false, hasBomb: false,
      primary: null, secondary: null, activeWeapon: null,
      reloading: false, reloadTimer: 0, lastFire: 0,
      grenades: { he:0, flash:0, smoke:0, molotov:0 },
      state: 'attack', targetNode: null, path: [], pathI: 0,
      seesEnemy: false, enemy: null, reaction: 0, flashTime: 0, fireTick: 0,
      lastFoot: 0, throwCd: 0, lastDamageFire: 0
    };
    bot.secondary = JSON.parse(JSON.stringify(WEAPONS[team === 'T' ? 'glock' : 'usp']));
    bot.activeWeapon = bot.secondary;
    const mat = new THREE.MeshLambertMaterial({ color: team === 'T' ? 0x7a5c3e : 0x39475b });
    const skin = new THREE.MeshLambertMaterial({ color: 0xc89b7a });
    const dark = new THREE.MeshLambertMaterial({ color: 0x222222 });
    const group = new THREE.Group();
    const torso = new THREE.Mesh(new THREE.BoxGeometry(0.52, 0.6, 0.32), mat); torso.position.y = 1.2;
    const head = new THREE.Mesh(new THREE.BoxGeometry(0.34, 0.3, 0.3), skin); head.name = 'head'; head.position.y = 1.65;
    const legL = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.7, 0.22), dark); legL.position.set(-0.14, 0.35, 0);
    const legR = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.7, 0.22), dark); legR.position.set(0.14, 0.35, 0);
    const armL = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.6, 0.16), dark); armL.position.set(-0.38, 1.2, 0);
    const armR = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.6, 0.16), dark); armR.position.set(0.38, 1.2, 0);
    group.add(torso, head, legL, legR, armL, armR);
    group.traverse(o => { if (o.isMesh) o.userData.entity = bot; });
    bot.group = group;
    bot.legs = [legL, legR];
    this.bots.push(bot);
    return bot;
  }

  startRound() {
    this.roundNumber++;
    this.phase = 'freeze';
    this.phaseTime = 5;
    this.roundTimer = 115;
    this.bombPlanted = false;
    this.bombTimer = 0;
    this.plantProgress = 0;
    this.defuseProgress = 0;
    this.bombMesh.visible = false;
    this.winner = null;

    for (const b of this.bots) this.resetEntity(b);
    this.resetEntity(this.player);

    // 分配炸弹
    this.bombMesh.visible = false;
    if (this.player.team === 'T') {
      this.player.hasBomb = true;
      for (const b of this.bots) b.hasBomb = false;
    } else {
      const tBots = this.bots.filter(b => b.team === 'T');
      const carrier = tBots.length ? tBots[Math.floor(Math.random()*tBots.length)] : null;
      if (carrier) carrier.hasBomb = true;
    }

    // bot自动购买
    for (const b of this.bots) this.botAutoBuy(b);

    // bot目标
    for (const b of this.bots) this.setBotObjective(b);

    document.getElementById('roundNum').textContent = this.roundNumber;
    this.showBanner(`第 ${this.roundNumber} 回合 · 购买阶段`, 2);
    this.updateHUD();
  }

  resetEntity(e) {
    const spawns = e.team === 'T' ? this.mapDef._tSpawns : this.mapDef._cSpawns;
    const base = spawns && spawns.length ? spawns[Math.floor(Math.random()*spawns.length)] : new THREE.Vector3();
    e.pos.copy(base);
    e.pos.x += rand(-1.2, 1.2);
    e.pos.z += rand(-1.2, 1.2);
    e.pos.y = 0;
    e.vel.set(0, 0, 0);
    e.alive = true;
    e.crouching = false;
    e.activeWeapon = e.secondary;
    e.reloading = false;
    e.reloadTimer = 0;
    e.lastFire = 0;
    e.seesEnemy = false;
    e.enemy = null;
    e.flashTime = 0;
    e.throwCd = 0;
    e.yaw = e.team === 'T' ? 0.7 : Math.PI - 0.7;
    e.pitch = 0;
    e.hasBomb = false;
    if (!e.survived) {
      e.health = 100;
      e.armor = 0;
      e.helmet = false;
      if (e.team === 'CT') e.defuseKit = true; else e.defuseKit = false;
      e.primary = null;
      e.grenades = { he:0, flash:0, smoke:0, molotov:0 };
      e.secondary = JSON.parse(JSON.stringify(WEAPONS[e.team === 'T' ? 'glock' : 'usp']));
      e.activeWeapon = e.secondary;
    } else {
      e.health = 100;
      e.armor = e.armor || 0;
    }
    if (!e.primary && e.activeWeapon && e.activeWeapon.kind !== 'pistol') e.activeWeapon = e.secondary;
    if (!e.activeWeapon) e.activeWeapon = e.secondary;
    this.updateGunModel();
  }

  botAutoBuy(b) {
    const allowed = ['nova','mac10','mp9','galil','famas','ak47','m4a4','awp'];
    const buyable = allowed.filter(id => isTeamWeapon(id, b.team) && b.money >= getPrice(id, b.team));
    if (buyable.length && !b.primary) {
      const pick = buyable.includes(b.team === 'T' ? 'ak47' : 'm4a4') ? (b.team === 'T' ? 'ak47' : 'm4a4')
        : buyable[buyable.length - 1];
      const w = JSON.parse(JSON.stringify(WEAPONS[pick]));
      b.primary = { ...w, mag: w.mag, reserve: w.reserve };
      b.money -= getPrice(pick, b.team);
      b.activeWeapon = b.primary;
    }
    if (b.money >= 1000 && b.armor < 100) { b.armor = 100; b.helmet = true; b.money -= 1000; }
    if (b.team === 'CT' && !b.defuseKit && b.money >= 400) { b.defuseKit = true; b.money -= 400; }
    if (b.money >= 300) {
      b.grenades.he = (b.grenades.he || 0) + Math.min(1, Math.floor(b.money / 300));
      b.money -= 300 * Math.min(1, Math.floor(b.money / 300));
    }
    if (b.money >= 300 && b.grenades.smoke === 0) { b.grenades.smoke = 1; b.money -= 300; }
  }

  setBotObjective(bot) {
    const nearest = this.nearestNode(bot.pos);
    let target = null;
    if (bot.team === 'T') {
      if (bot.hasBomb) {
        const site = this.bombSites[Math.floor(Math.random() * this.bombSites.length)];
        target = this.nearestNode(site.center);
        bot.state = 'plant';
      } else {
        const site = this.bombSites[Math.floor(Math.random() * this.bombSites.length)];
        target = this.nearestNode(site.center);
        bot.state = 'attack';
      }
    } else {
      if (this.bombPlanted) {
        target = this.nearestNode(this.bombMesh.position);
        bot.state = 'defuse';
      } else {
        const site = this.bombSites[Math.floor(Math.random() * this.bombSites.length)];
        target = this.nearest