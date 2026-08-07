下面给你一份**单文件可运行**的浏览器第一人称战术射击游戏项目。  
保存为 `index.html`，用本地 HTTP 服务打开即可运行。

运行方式：

1. 保存下面代码为 `index.html`
2. 在该目录启动本地服务器，例如：
   - `python -m http.server 8080`
   - 或 `npx serve`
3. 浏览器打开 `http://localhost:8080`

操作：
- 鼠标：视角
- WASD：移动
- Shift：冲刺
- Ctrl：下蹲
- Space：跳跃
- 鼠标左键：射击 / 投掷物
- 鼠标右键：狙击开镜
- R：换弹
- B：打开购买菜单
- E：使用 / 下包 / 拆包 / 捡枪
- 1-7：切换武器与投掷物
- Esc：暂停

> 说明：这是一个**完整可玩的、纯浏览器单文件版本**，包含 3 张地图、爆破模式、经济系统、Bot AI、投掷物、雷达、UI、购买、下包拆包、重生、暂停与重开。  
> Dust2_Test 地图已按经典 Dust2 的战术结构做了高度近似复刻与可玩连接。

```html
<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<title>Browser Tactical FPS</title>
<style>
  html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#050607;font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif}
  #game{position:fixed;inset:0;background:#050607}
  canvas{display:block}
  .ui{position:absolute;left:0;top:0;right:0;bottom:0;pointer-events:none}
  #hud{position:absolute;left:0;top:0;right:0;bottom:0;color:#e9ecef}
  #topbar{position:absolute;left:50%;top:12px;transform:translateX(-50%);display:flex;gap:16px;align-items:center;background:rgba(10,14,16,.45);backdrop-filter:blur(8px);border:1px solid rgba(255,255,255,.09);border-radius:12px;padding:8px 14px;font-weight:700;letter-spacing:.2px}
  #score{font-size:18px;color:#f3f4f6}
  #round{font-size:14px;color:#bfc7ce}
  #timer{font-size:18px;color:#ffd166;min-width:60px;text-align:center}
  #stats{position:absolute;left:14px;bottom:14px;width:280px;background:rgba(8,10,12,.5);backdrop-filter:blur(8px);border:1px solid rgba(255,255,255,.09);border-radius:12px;padding:12px 14px}
  .row{display:flex;justify-content:space-between;align-items:center;margin:5px 0;font-size:14px}
  .label{color:#aeb7bf}
  .value{font-weight:700}
  #ammo{font-size:22px;font-weight:800;color:#f8fafc}
  #weaponName{font-size:14px;color:#bfc7ce}
  #crosshair{position:absolute;left:50%;top:50%;width:0;height:0;transform:translate(-50%,-50%);filter:drop-shadow(0 0 2px rgba(0,0,0,.7))}
  .c{position:absolute;background:#dbe2e8;border-radius:1px;opacity:.95}
  #ch1,#ch2,#ch3,#ch4{background:#e8edf2}
  #ch1{left:-1px;top:-12px;width:2px;height:8px}
  #ch2{left:-1px;top:4px;width:2px;height:8px}
  #ch3{top:-1px;left:-12px;width:8px;height:2px}
  #ch4{top:-1px;left:4px;width:8px;height:2px}
  #radarWrap{position:absolute;right:14px;bottom:14px;width:210px;height:210px;background:rgba(8,10,12,.55);backdrop-filter:blur(8px);border:1px solid rgba(255,255,255,.09);border-radius:14px;overflow:hidden}
  #radar{width:100%;height:100%;display:block}
  #killfeed{position:absolute;right:16px;top:62px;display:flex;flex-direction:column;gap:6px;align-items:flex-end}
  .kill{padding:6px 10px;background:rgba(8,10,12,.45);border:1px solid rgba(255,255,255,.08);border-radius:10px;font-size:13px;color:#e8edf2;opacity:.96}
  #hint{position:absolute;left:50%;bottom:92px;transform:translateX(-50%);padding:8px 14px;border-radius:999px;background:rgba(8,10,12,.52);border:1px solid rgba(255,255,255,.1);color:#e8edf2;font-size:13px;opacity:0;transition:opacity .15s}
  #flash{position:absolute;inset:0;background:#fff;opacity:0;pointer-events:none}
  #barWrap{position:absolute;left:50%;bottom:52px;transform:translateX(-50%);width:280px;height:12px;background:rgba(8,10,12,.55);border:1px solid rgba(255,255,255,.08);border-radius:999px;overflow:hidden;opacity:0}
  #bar{height:100%;width:0;background:linear-gradient(90deg,#ffbb00,#2dd4bf)}
  #barText{position:absolute;left:50%;bottom:68px;transform:translateX(-50%);font-size:13px;color:#f8fafc;opacity:0}
  #message{position:absolute;left:50%;top:18%;transform:translateX(-50%);background:rgba(8,10,12,.65);border:1px solid rgba(255,255,255,.09);padding:10px 16px;border-radius:12px;color:#fff;font-size:15px;opacity:0;transition:opacity .2s}
  #lobby,#pause,#gameover,#buy{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:auto}
  .panel{width:min(760px,92vw);max-height:90vh;overflow:auto;background:rgba(9,12,14,.88);backdrop-filter:blur(12px);border:1px solid rgba(255,255,255,.12);border-radius:18px;color:#e9ecef;box-shadow:0 18px 50px rgba(0,0,0,.45);padding:20px 24px}
  .panel h1,.panel h2{margin:0 0 10px 0}
  .panel h1{font-size:28px}
  .panel h2{font-size:18px;color:#dbe3ea}
  .grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:12px}
  .field{display:flex;flex-direction:column;gap:6px}
  label{font-size:13px;color:#aeb7bf}
  select,input,button{font:inherit}
  select,input{background:#11161a;color:#f3f4f6;border:1px solid rgba(255,255,255,.12);border-radius:10px;padding:10px 12px}
  button{background:#1f2933;color:#fff;border:1px solid rgba(255,255,255,.12);border-radius:12px;padding:10px 14px;cursor:pointer}
  button:hover{background:#28333f}
  .row2{display:flex;gap:10px;flex-wrap:wrap}
  .small{font-size:13px;color:#aeb7bf;line-height:1.6}
  .sep{height:1px;background:rgba(255,255,255,.08);margin:16px 0}
  #buyItems{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px}
  .item{background:#11161a;border:1px solid rgba(255,255,255,.10);border-radius:12px;padding:10px;display:flex;flex-direction:column;gap:8px}
  .item .name{font-weight:800}
  .item .meta{font-size:12px;color:#aeb7bf;min-height:36px}
  .item .cost{font-size:13px;color:#ffd166}
  .hidden{display:none !important}
  #pauseButtons,#goButtons{display:flex;gap:10px;flex-wrap:wrap}
  .tip{font-size:13px;color:#aeb7bf;margin-top:6px}
  .kbd{display:inline-block;padding:1px 6px;border-radius:6px;border:1px solid rgba(255,255,255,.15);background:#11161a;color:#f3f4f6;font-size:12px}
</style>
</head>
<body>
<div id="game">
  <div id="lobby" class="ui">
    <div class="panel">
      <h1>Browser Tactical FPS</h1>
      <div class="small">
        经典战术射击风格：爆破模式、购买、Bot 对战、雷达、经济、投掷物、下包拆包、3 张地图。
        <br>Dust2_Test 已按 Dust2 的战术结构、路线关系与交火节奏做了高还原玩法设计。
      </div>
      <div class="sep"></div>
      <div class="grid">
        <div class="field"><label>地图</label>
          <select id="mapSel">
            <option value="dust2">Dust2_Test</option>
            <option value="facility">Facility_9</option>
            <option value="harbor">Harbor_Freight</option>
          </select>
        </div>
        <div class="field"><label>阵营</label>
          <select id="sideSel"><option value="T">恐怖分子 T</option><option value="CT">反恐精英 CT</option></select>
        </div>
        <div class="field"><label>每队人数（含你）</label>
          <select id="teamSizeSel">
            <option>2</option><option>3</option><option>4</option><option selected>5</option>
          </select>
        </div>
        <div class="field"><label>Bot 难度</label>
          <select id="difficultySel"><option value="easy">简单</option><option value="normal" selected>普通</option><option value="hard">困难</option></select>
        </div>
      </div>
      <div class="sep"></div>
      <div class="row2">
        <button id="startBtn">开始对局</button>
        <button id="showTipsBtn">显示操作说明</button>
      </div>
      <div id="tips" class="tip hidden">
        <div>移动：<span class="kbd">WASD</span>，冲刺：<span class="kbd">Shift</span>，下蹲：<span class="kbd">Ctrl</span>，跳跃：<span class="kbd">Space</span></div>
        <div>射击：<span class="kbd">鼠标左键</span>，开镜：<span class="kbd">鼠标右键</span>，换弹：<span class="kbd">R</span></div>
        <div>购买：<span class="kbd">B</span>，使用 / 下包 / 拆包 / 捡枪：<span class="kbd">E</span></div>
        <div>切枪：<span class="kbd">1-7</span>，暂停：<span class="kbd">Esc</span></div>
      </div>
    </div>
  </div>

  <div id="hud" class="ui hidden">
    <div id="topbar">
      <div id="score">T 0 : 0 CT</div>
      <div id="timer">1:55</div>
      <div id="round">Round 1</div>
    </div>
    <div id="stats">
      <div class="row"><span class="label">HP / Armor</span><span id="hp" class="value">100 / 0</span></div>
      <div class="row"><span class="label">Money</span><span id="money" class="value">$800</span></div>
      <div class="row"><span class="label">Weapon</span><span id="weaponName" class="value">USP</span></div>
      <div class="row"><span class="label">Ammo</span><span id="ammo" class="value">12 / 24</span></div>
      <div class="row"><span class="label">Team</span><span id="teamTxt" class="value">CT</span></div>
    </div>
    <div id="crosshair"><div id="ch1" class="c"></div><div id="ch2" class="c"></div><div id="ch3" class="c"></div><div id="ch4" class="c"></div></div>
    <div id="killfeed"></div>
    <div id="radarWrap"><canvas id="radar" width="210" height="210"></canvas></div>
    <div id="hint"></div>
    <div id="barText"></div>
    <div id="barWrap"><div id="bar"></div></div>
    <div id="message"></div>
    <div id="flash"></div>
  </div>

  <div id="buy" class="ui hidden">
    <div class="panel">
      <h2>购买菜单</h2>
      <div class="small">仅在购枪阶段可购买。键盘 <span class="kbd">B</span> 打开 / 关闭。</div>
      <div class="sep"></div>
      <div id="buyItems"></div>
      <div class="sep"></div>
      <div class="row2">
        <button id="closeBuyBtn">关闭</button>
      </div>
    </div>
  </div>

  <div id="pause" class="ui hidden">
    <div class="panel">
      <h2>已暂停</h2>
      <div class="small">暂停时可以重新开始、返回大厅或继续。</div>
      <div class="sep"></div>
      <div id="pauseButtons">
        <button id="resumeBtn">继续</button>
        <button id="restartBtn">重新开始</button>
        <button id="backLobbyBtn">返回大厅</button>
      </div>
    </div>
  </div>

  <div id="gameover" class="ui hidden">
    <div class="panel">
      <h2 id="endTitle">对局结束</h2>
      <div id="endText" class="small"></div>
      <div class="sep"></div>
      <div id="goButtons">
        <button id="againBtn">再来一局</button>
        <button id="toLobbyBtn">返回大厅</button>
      </div>
    </div>
  </div>
</div>

<script type="module">
import * as THREE from 'https://unpkg.com/three@0.160.0/build/three.module.js';

const $=id=>document.getElementById(id), clamp=(v,a,b)=>Math.max(a,Math.min(b,v)), rand=(a,b)=>a+Math.random()*(b-a), TAU=Math.PI*2;
const lobby=$('lobby'), hud=$('hud'), buyUI=$('buy'), pauseUI=$('pause'), overUI=$('gameover'), radar=$('radar'), rctx=radar.getContext('2d');
const flash=$('flash'), hpEl=$('hp'), moneyEl=$('money'), ammoEl=$('ammo'), weaponEl=$('weaponName'), teamEl=$('teamTxt');
const scoreEl=$('score'), timerEl=$('timer'), roundEl=$('round'), msgEl=$('message'), hintEl=$('hint'), barText=$('barText'), barWrap=$('barWrap'), bar=$('bar'), feed=$('killfeed');
const buyItems=$('buyItems'), endTitle=$('endTitle'), endText=$('endText');
const mapSel=$('mapSel'), sideSel=$('sideSel'), teamSizeSel=$('teamSizeSel'), difficultySel=$('difficultySel');
const tips=$('tips');

let scene,camera,renderer,clock,rayDir=new THREE.Vector3(), tempV3=new THREE.Vector3(), tempV2=new THREE.Vector2();
let currentMap=null, currentMapData=null, gameStarted=false, paused=false, inBuy=false, matchEnded=false;
let keys={}, mouse={x:0,y:0,left:false,right:false}, pointerLocked=false;
let audioCtx=null;

const WEAPONS={
  knife:{name:'Knife',slot:3,type:'melee',price:0,clip:1,reserve:0,damage:45,range:2.0,rate:1,spread:0,recoil:0,move:1.0,draw:0.08,auto:false},
  glock:{name:'Glock-18',slot:2,type:'pistol',price:200,clip:20,reserve:120,damage:24,hs:3.6,range:60,rate:6.5,spread:0.018,recoil:0.045,move:1.0,draw:0.12,auto:true},
  usp:{name:'USP-S',slot:2,type:'pistol',price:200,clip:12,reserve:24,damage:28,hs:4.0,range:65,rate:4.5,spread:0.014,recoil:0.05,move:1.0,draw:0.12,auto:false},
  deagle:{name:'Desert Eagle',slot:2,type:'pistol',price:700,clip:7,reserve:35,damage:53,hs:4.2,range:70,rate:2.2,spread:0.015,recoil:0.09,move:0.95,draw:0.12,auto:false},
  mp5:{name:'MP5',slot:1,type:'smg',price:1500,clip:30,reserve:120,damage:21,hs:2.7,range:50,rate:12,spread:0.03,recoil:0.03,move:1.06,draw:0.16,auto:true},
  ak47:{name:'AK-47',slot:1,type:'rifle',price:2700,clip:30,reserve:90,damage:36,hs:4.0,range:80,rate:10,spread:0.015,recoil:0.055,move:0.93,draw:0.18,auto:true},
  m4a4:{name:'M4A4',slot:1,type:'rifle',price:2900,clip:30,reserve:90,damage:33,hs:3.6,range:80,rate:11,spread:0.013,recoil:0.05,move:0.94,draw:0.18,auto:true},
  awp:{name:'AWP',slot:1,type:'sniper',price:4750,clip:10,reserve:30,damage:115,hs:1.0,range:140,rate:1.1,spread:0.0002,recoil:0.12,move:0.80,draw:0.22,auto:false,scope:true},
  nova:{name:'Nova',slot:1,type:'shotgun',price:1200,clip:8,reserve:32,damage:10,hs:1.2,range:24,rate:1.3,spread:0.12,recoil:0.07,move:0.88,draw:0.16,auto:false,pellets:8},
  he:{name:'HE Grenade',slot:4,type:'nade',price:300},
  flash:{name:'Flashbang',slot:5,type:'flash',price:200},
  smoke:{name:'Smoke Grenade',slot:6,type:'smoke',price:300},
  molotov:{name:'Molotov',slot:7,type:'molotov',price:400}
};
const TEAMCOL={T:0xc2a76c,CT:0x7fb1da};
const DIFF={easy:{react:0.48,aim:0.11,shoot:0.42,peek:0.38,gren:0.10},normal:{react:0.22,aim:0.065,shoot:0.66,peek:0.56,gren:0.22},hard:{react:0.08,aim:0.028,shoot:0.85,peek:0.76,gren:0.38}};
const ROUNDS_TO_WIN=8;

function makeTex(kind){
  const c=document.createElement('canvas'); c.width=c.height=128; const g=c.getContext('2d');
  if(kind==='sand'){g.fillStyle='#b89152';g.fillRect(0,0,128,128);for(let i=0;i<1000;i++){const x=Math.random()*128,y=Math.random()*128,a=Math.random()*0.15+0.05;g.fillStyle=`rgba(${150+Math.random()*50|0},${110+Math.random()*40|0},${50+Math.random()*30|0},${a})`;g.fillRect(x,y,1,1)}g.strokeStyle='rgba(90,70,30,.25)';for(let i=0;i<12;i++){g.beginPath();g.moveTo(rand(0,128),rand(0,128));g.lineTo(rand(0,128),rand(0,128));g.stroke()}}
  else if(kind==='concrete'){g.fillStyle='#7d8489';g.fillRect(0,0,128,128);g.fillStyle='rgba(30,35,40,.10)';for(let i=0;i<40;i++)g.fillRect(rand(0,128),rand(0,128),rand(8,28),1);for(let i=0;i<800;i++)g.fillStyle=`rgba(${120+Math.random()*50|0},${120+Math.random()*50|0},${120+Math.random()*50|0},${Math.random()*0.06})`,g.fillRect(rand(0,128),rand(0,128),1,1)}
  else if(kind==='metal'){g.fillStyle='#67717c';g.fillRect(0,0,128,128);for(let y=0;y<128;y+=16){g.fillStyle=`rgba(255,255,255,${0.04})`;g.fillRect(0,y,128,1)}g.fillStyle='rgba(0,0,0,.12)';for(let x=0;x<128;x+=16)g.fillRect(x,0,1,128)}
  else if(kind==='wood'){g.fillStyle='#8a623a';g.fillRect(0,0,128,128);for(let y=0;y<128;y+=10){g.fillStyle=`rgba(80,45,18,${0.12+Math.random()*0.12})`;g.fillRect(0,y,128,2)}}
  else if(kind==='steel'){g.fillStyle='#8b9298';g.fillRect(0,0,128,128);g.fillStyle='rgba(255,255,255,.05)';for(let i=0;i<18;i++)g.fillRect(0,i*7,128,1);g.fillStyle='rgba(0,0,0,.12)';for(let i=0;i<18;i++)g.fillRect(i*7,0,1,128)}
  else if(kind==='water'){g.fillStyle='#395f79';g.fillRect(0,0,128,128);g.fillStyle='rgba(255,255,255,.05)';for(let i=0;i<40;i++){g.beginPath();g.arc(rand(0,128),rand(0,128),rand(6,18),0,TAU);g.strokeStyle=`rgba(255,255,255,${Math.random()*0.08})`;g.stroke()}}
  else if(kind==='crate'){g.fillStyle='#8a633d';g.fillRect(0,0,128,128);g.fillStyle='#6e4d2b';for(let i=0;i<8;i++)g.fillRect(i*16,0,2,128),g.fillRect(0,i*16,128,2);g.fillStyle='rgba(255,255,255,.05)';for(let i=0;i<600;i++)g.fillRect(rand(0,128),rand(0,128),1,1)}
  const t=new THREE.CanvasTexture(c); t.wrapS=t.wrapT=THREE.RepeatWrapping; t.colorSpace=THREE.SRGBColorSpace; return t;
}
const TEX={sand:makeTex('sand'),concrete:makeTex('concrete'),metal:makeTex('metal'),wood:makeTex('wood'),steel:makeTex('steel'),water:makeTex('water'),crate:makeTex('crate')};

function shotAudio(kind='rifle'){
  if(!audioCtx) return;
  const t=audioCtx.currentTime;
  const g=audioCtx.createGain(); g.connect(audioCtx.destination);
  const n=audioCtx.createBufferSource(); const len=Math.max(.08,.18); const b=audioCtx.createBuffer(1,audioCtx.sampleRate*len,audioCtx.sampleRate);
  const d=b.getChannelData(0); for(let i=0;i<d.length;i++) d[i]=(Math.random()*2-1)*(1-i/d.length);
  n.buffer=b;
  const f=audioCtx.createBiquadFilter(); f.type='highpass'; f.frequency.value=kind==='sniper'?220:350;
  const g2=audioCtx.createGain(); g2.gain.setValueAtTime(.001,t); g2.gain.exponentialRampToValueAtTime(kind==='sniper'?.8:.45,t+.01); g2.gain.exponentialRampToValueAtTime(.001,t+len);
  const o=audioCtx.createOscillator(); o.type='triangle'; o.frequency.setValueAtTime(kind==='sniper'?90:170,t); o.frequency.exponentialRampToValueAtTime(50,t+.09);
  const og=audioCtx.createGain(); og.gain.setValueAtTime(.001,t); og.gain.exponentialRampToValueAtTime(.2,t+.01); og.gain.exponentialRampToValueAtTime(.001,t+.12);
  n.connect(f); f.connect(g2); g2.connect(g);
  o.connect(og); og.connect(g);
  n.start(t); n.stop(t+len); o.start(t); o.stop(t+.13);
}
function fxAudio(kind){
  if(!audioCtx) return;
  const t=audioCtx.currentTime;
  const o=audioCtx.createOscillator(), g=audioCtx.createGain();
  o.connect(g); g.connect(audioCtx.destination);
  if(kind==='reload'){o.type='square'; o.frequency.value=1500; g.gain.setValueAtTime(.06,t); g.gain.exponentialRampToValueAtTime(.001,t+.05); o.start(t); o.stop(t+.06)}
  else if(kind==='step'){o.type='triangle'; o.frequency.value=250; g.gain.setValueAtTime(.035,t); g.gain.exponentialRampToValueAtTime(.001,t+.025); o.start(t); o.stop(t+.03)}
  else if(kind==='plant'){o.type='sawtooth'; o.frequency.value=420; g.gain.setValueAtTime(.07,t); g.gain.exponentialRampToValueAtTime(.001,t+.18); o.start(t); o.stop(t+.2)}
  else if(kind==='defuse'){o.type='sine'; o.frequency.value=700; g.gain.setValueAtTime(.05,t); g.gain.linearRampToValueAtTime(.02,t+.2); g.gain.linearRampToValueAtTime(.001,t+.4); o.start(t); o.stop(t+.42)}
  else if(kind==='explosion'){o.type='sine'; o.frequency.value=50; g.gain.setValueAtTime(.25,t); g.gain.exponentialRampToValueAtTime(.001,t+.4); o.start(t); o.stop(t+.45)}
  else if(kind==='flash'){o.type='square'; o.frequency.value=1100; g.gain.setValueAtTime(.08,t); g.gain.exponentialRampToValueAtTime(.001,t+.11); o.start(t); o.stop(t+.12)}
  else if(kind==='buy'){o.type='triangle'; o.frequency.value=920; g.gain.setValueAtTime(.05,t); g.gain.exponentialRampToValueAtTime(.001,t+.08); o.start(t); o.stop(t+.09)}
  else if(kind==='round'){o.type='sine'; o.frequency.value=480; g.gain.setValueAtTime(.05,t); g.gain.exponentialRampToValueAtTime(.001,t+.3); o.start(t); o.stop(t+.32)}
}
function ensureAudio(){if(!audioCtx)audioCtx=new (window.AudioContext||window.webkitAudioContext)(); if(audioCtx.state==='suspended')audioCtx.resume()}

function makeRect(x,z,w,d,h=6,color=0x999999,mat=null,kind='wall'){return {x,z,w,d,h,color,mat,kind}}
function makeMapDust2(){
  const W=120,H=120, walls=[], props=[], spawns={T:[],CT:[]}, sites={A:{x:18,z:20,w:18,d:18},B:{x:95,z:24,w:18,d:18}};
  const add=(x,z,w,d,h=8,kind='wall',mat='sand')=>walls.push(makeRect(x,z,w,d,h,0,mat,kind));
  const box=(x,z,w,d,h=2,mat='crate')=>props.push(makeRect(x,z,w,d,h,0,mat,'prop'));
  // borders
  add(0,0,W,3,10,'wall','sand'); add(0,H-3,W,3,10,'wall','sand'); add(0,0,3,H,10,'wall','sand'); add(W-3,0,3,H,10,'wall','sand');
  // long a lane structure
  add(10,28,18,3,8,'wall','sand'); add(10,28,3,45,8,'wall','sand'); add(25,28,3,22,8,'wall','sand');
  add(8,64,20,3,8,'wall','sand');
  // mid / doors / xbox
  add(50,44,7,3,8,'wall','sand'); add(63,44,7,3,8,'wall','sand'); add(56,36,3,10,8,'wall','sand');
  add(44,57,4,4,6,'wall','sand'); add(49,57,4,4,6,'wall','sand'); add(54,57,4,4,6,'wall','sand');
  // cat / short route
  add(42,30,3,18,8,'wall','sand'); add(42,30,12,3,8,'wall','sand'); add(39,21,3,12,8,'wall','sand');
  // tunnels / b route
  add(80,64,20,3,8,'wall','sand'); add(95,44,3,20,8,'wall','sand'); add(84,50,12,3,8,'wall','sand');
  add(83,31,3,20,8,'wall','sand');
  // ct connectors
  add(60,12,18,3,8,'wall','sand'); add(73,12,3,18,8,'wall','sand'); add(60,16,3,14,8,'wall','sand');
  // site structures
  box(14,22,4,4,3); box(18,16,4,4,3); box(21,25,4,4,3);
  box(27,24,4,4,3); box(23,14,4,4,3);
  box(22,11,4,4,3); box(28,18,4,4,3);
  box(88,22,4,4,3); box(95,20,5,5,3); box(100,31,4,4,3);
  box(90,30,4,4,3); box(93,28,4,4,3);
  box(57,55,4,4,3); box(61,52,4,4,3);
  box(48,53,4,4,3); box(65,49,4,4,3);
  // long cover / pit / platform / ct / b plat
  box(13,36,5,5,3); box(16,42,4,4,3); box(20,39,4,4,3);
  box(20,30,6,3,2); box(18,31,5,2,2);
  box(35,78,4,4,3); box(38,71,4,4,3); box(31,67,4,4,3);
  box(75,33,4,4,3); box(71,29,4,4,3); box(80,24,4,4,3);
  // explicit spawns
  spawns.T=[[60,104],[50,100],[70,100],[40,95],[80,95]];
  spawns.CT=[[60,12],[50,14],[70,14],[55,18],[65,18]];
  return {
    id:'dust2', name:'Dust2_Test', size:{w:W,h:H}, theme:{floor:'sand',fog:0xdbc08b,sky:0x8ea1b3,accent:0xe8c987},
    walls, props, spawns, sites,
    labels:[
      {name:'T 出生点',x:60,z:104},{name:'CT 出生点',x:60,z:12},{name:'A 包点',x:18,z:20},{name:'B 包点',x:95,z:24},
      {name:'中路',x:57,z:52},{name:'A 大',x:16,z:58},{name:'A 小',x:46,z:31},{name:'A 坑',x:18,z:36},{name:'A 斜坡',x:43,z:26},
      {name:'中门',x:56,z:44},{name:'双门',x:60,z:44},{name:'Xbox',x:58,z:56},{name:'B 洞',x:89,z:72},{name:'B 门',x:88,z:44},{name:'B 平台',x:94,z:30},
      {name:'警家',x:60,z:16},{name:'长门',x:24,z:76},{name:'箱体掩体',x:31,z:68}
    ],
    plans:{
      T:[[{x:24,z:74},{x:19,z:50},{x:16,z:27},{x:18,z:20}], [{x:57,z:74},{x:58,z:55},{x:46,z:31},{x:26,z:24}], [{x:85,z:74},{x:90,z:56},{x:95,z:39},{x:95,z:24}]],
      CT:[[{x:18,z:20}], [{x:57,z:52}], [{x:95,z:24}], [{x:60,z:16}]]
    },
    bombZones:[sites.A,sites.B]
  };
}
function makeMapFacility(){
  const W=110,H=110, walls=[], props=[], spawns={T:[],CT:[]}, sites={A:{x:22,z:22,w:18,d:18},B:{x:88,z:88,w:18,d:18}};
  const add=(x,z,w,d,h=8,kind='wall',mat='concrete')=>walls.push(makeRect(x,z,w,d,h,0,mat,kind));
  const box=(x,z,w,d,h=2,mat='metal')=>props.push(makeRect(x,z,w,d,h,0,mat,'prop'));
  add(0,0,W,3,10,'wall','concrete'); add(0,H-3,W,3,10,'wall','concrete'); add(0,0,3,H,10,'wall','concrete'); add(W-3,0,3,H,10,'wall','concrete');
  add(10,24,26,3,8,'wall','concrete'); add(10,24,3,26,8,'wall','concrete'); add(32,35,10,3,8,'wall','concrete');
  add(52,18,3,28,8,'wall','concrete'); add(52,51,3,26,8,'wall','concrete'); add(40,51,15,3,8,'wall','concrete');
  add(72,22,3,30,8,'wall','concrete'); add(72,66,3,20,8,'wall','concrete');
  add(22,64,26,3,8,'wall','concrete'); add(22,64,3,24,8,'wall','concrete'); add(10,82,18,3,8,'wall','concrete');
  add(60,86,18,3,8,'wall','concrete'); add(60,86,3,14,8,'wall','concrete');
  box(20,20,4,4,3); box(26,24,4,4,3); box(19,28,4,4,3);
  box(82,84,4,4,3); box(87,89,4,4,3); box(92,84,4,4,3);
  box(56,50,4,4,3); box(60,56,4,4,3); box(48,58,4,4,3);
  box(36,74,4,4,3); box(40,69,4,4,3); box(31,69,4,4,3);
  box(70,48,4,4,3); box(75,45,4,4,3);
  spawns.T=[[54,100],[44,98],[64,98],[48,94],[70,92]];
  spawns.CT=[[56,12],[46,14],[66,14],[52,18],[60,18]];
  return {
    id:'facility', name:'Facility_9', size:{w:W,h:H}, theme:{floor:'concrete',fog:0xa6b7c4,sky:0x778899,accent:0xbfd1dc},
    walls, props, spawns, sites,
    labels:[
      {name:'T 出生点',x:54,z:100},{name:'CT 出生点',x:56,z:12},{name:'A 包点',x:22,z:22},{name:'B 包点',x:88,z:88},
      {name:'中路',x:54,z:54},{name:'反应堆房',x:55,z:56},{name:'A 通道',x:30,z:36},{name:'B 长廊',x:78,z:66},{name:'旋转走廊',x:60,z:86}
    ],
    plans:{
      T:[[{x:28,z:74},{x:24,z:50},{x:22,z:22}], [{x:56,z:74},{x:56,z:55},{x:22,z:22}], [{x:82,z:74},{x:86,z:88}]],
      CT:[[{x:22,z:22}], [{x:54,z:54}], [{x:88,z:88}]]
    },
    bombZones:[sites.A,sites.B]
  };
}
function makeMapHarbor(){
  const W=120,H=120, walls=[], props=[], spawns={T:[],CT:[]}, sites={A:{x:28,z:26,w:18,d:18},B:{x:92,z:90,w:18,d:18}};
  const add=(x,z,w,d,h=8,kind='wall',mat='steel')=>walls.push(makeRect(x,z,w,d,h,0,mat,kind));
  const box=(x,z,w,d,h=2,mat='crate')=>props.push(makeRect(x,z,w,d,h,0,mat,'prop'));
  add(0,0,W,3,10,'wall','steel'); add(0,H-3,W,3,10,'wall','steel'); add(0,0,3,H,10,'wall','steel'); add(W-3,0,3,H,10,'wall','steel');
  add(14,18,18,3,8,'wall','steel'); add(14,18,3,34,8,'wall','steel'); add(30,36,4,18,8,'wall','steel');
  add(48,20,3,30,8,'wall','steel'); add(48,60,3,24,8,'wall','steel'); add(42,60,12,3,8,'wall','steel');
  add(70,40,3,30,8,'wall','steel'); add(70,74,3,18,8,'wall','steel'); add(58,74,15,3,8,'wall','steel');
  add(86,62,18,3,8,'wall','steel'); add(86,62,3,26,8,'wall','steel');
  box(24,22,4,4,3); box(18,28,4,4,3); box(31,26,4,4,3); box(25,16,5,3,2);
  box(92,84,4,4,3); box(98,90,4,4,3); box(88,96,4,4,3); box(97,98,5,3,2);
  box(56,58,4,4,3); box(63,56,4,4,3); box(59,66,4,4,3);
  box(74,76,4,4,3); box(79,82,4,4,3); box(83,88,4,4,3);
  spawns.T=[[58,104],[48,98],[68,98],[40,96],[80,94]];
  spawns.CT=[[60,12],[50,14],[70,14],[55,18],[65,18]];
  return {
    id:'harbor', name:'Harbor_Freight', size:{w:W,h:H}, theme:{floor:'water',fog:0x6f8ea4,sky:0x5e7484,accent:0xd0e3ee},
    walls, props, spawns, sites,
    labels:[
      {name:'T 出生点',x:58,z:104},{name:'CT 出生点',x:60,z:12},{name:'A 包点',x:28,z:26},{name:'B 包点',x:92,z:90},
      {name:'码头长线',x:20,z:50},{name:'货柜中路',x:58,z:58},{name:'仓库回防路',x:75,z:76}
    ],
    plans:{
      T:[[{x:24,z:76},{x:28,z:50},{x:28,z:26}], [{x:58,z:74},{x:58,z:58},{x:92,z:90}], [{x:84,z:74},{x:92,z:90}]],
      CT:[[{x:28,z:26}], [{x:58,z:58}], [{x:92,z:90}]]
    },
    bombZones:[sites.A,sites.B]
  };
}
const MAPS={dust2:makeMapDust2(),facility:makeMapFacility(),harbor:makeMapHarbor()};

function segRectHit(ax,az,bx,bz,r){
  let t0=0,t1=1,dx=bx-ax,dz=bz-az;
  const checks=[[-dx,ax-r],[dx,r-ax],[-dz,az-r],[dz,r-az]];
  for(const [p,q] of checks){if(p===0){if(q<0)return false}else{const t=q/p;if(p<0){if(t>t1)return false;if(t>t0)t0=t}else{if(t<t0)return false;if(t<t1)t1=t}}}
  return true;
}
function segRect(ax,az,bx,bz,rec){return segRectHit(ax,az,bx,bz,{x:rec.x,z:rec.z,w:rec.w,d:rec.d})}
function rectOverlapCircle(rec,x,z,r){const cx=clamp(x,rec.x,rec.x+rec.w), cz=clamp(z,rec.z,rec.z+rec.d); const dx=x-cx,dz=z-cz; return dx*dx+dz*dz<r*r}
function pointInRect(x,z,r){return x>=r.x&&x<=r.x+r.w&&z>=r.z&&z<=r.z+r.d}
function segCircle(ax,az,bx,bz,cx,cz,r){const dx=bx-ax,dz=bz-az,l2=dx*dx+dz*dz;if(!l2)return ((ax-cx)**2+(az-cz)**2)<=r*r;let t=((cx-ax)*dx+(cz-az)*dz)/l2;t=clamp(t,0,1);const px=ax+dx*t,pz=az+dz*t;return (px-cx)**2+(pz-cz)**2<=r*r}
function dist2(x1,z1,x2,z2){const dx=x2-x1,dz=z2-z1;return dx*dx+dz*dz}
function norm2(x,z){const l=Math.hypot(x,z)||1; return [x/l,z/l]}
function fmtTime(s){s=Math.max(0,s); const m=(s/60)|0, ss=(s%60)|0; return m+':'+String(ss).padStart(2,'0')}
function addFeed(txt,col){const d=document.createElement('div'); d.className='kill'; d.textContent=txt; d.style.color=col||'#e8edf2'; feed.prepend(d); setTimeout(()=>d.style.opacity='0',4200); setTimeout(()=>d.remove(),5200)}
function showMsg(t,d=1800){msgEl.textContent=t; msgEl.style.opacity=1; clearTimeout(showMsg.t); showMsg.t=setTimeout(()=>msgEl.style.opacity=0,d)}
function showHint(t,d=1400){hintEl.textContent=t; hintEl.style.opacity=1; clearTimeout(showHint.t); showHint.t=setTimeout(()=>hintEl.style.opacity=0,d)}
function setBar(p,txt,show=true){bar.style.width=`${clamp(p,0,1)*100}%`; barText.textContent=txt||''; barWrap.style.opacity=show?1:0; barText.style.opacity=show?1:0}
function killMarker(){const d=document.createElement('div'); d.style.position='absolute'; d.style.left='50%'; d.style.top='50%'; d.style.transform='translate(-50%,-50%)'; d.style.width='8px'; d.style.height='8px'; d.style.border='2px solid #fff'; d.style.borderRadius='50%'; d.style.opacity=.9; hud.appendChild(d); setTimeout(()=>d.remove(),110)}
function randomColorHex(c){return '#'+c.toString(16).padStart(6,'0')}

function createSoldier(team){
  const g=new THREE.Group();
  const c=TEAMCOL[team];
  const mats={
    base:new THREE.MeshStandardMaterial({color:team==='T'?0x7a5d33:0x325879,roughness:.9,metalness:.08}),
    vest:new THREE.MeshStandardMaterial({color:0x2d3338,roughness:.95,metalness:.05}),
    acc:new THREE.MeshStandardMaterial({color:c,roughness:.65,metalness:.1}),
    skin:new THREE.MeshStandardMaterial({color:0xb78e73,roughness:.9}),
    gun:new THREE.MeshStandardMaterial({color:0x202428,roughness:.7,metalness:.2})
  };
  const torso=new THREE.Mesh(new THREE.CapsuleGeometry(.33,.55,3,8),mats.base); torso.position.y=1.0; g.add(torso);
  const vest=new THREE.Mesh(new THREE.BoxGeometry(.52,.44,.28),mats.vest); vest.position.y=1.02; g.add(vest);
  const head=new THREE.Mesh(new THREE.SphereGeometry(.18,12,10),mats.skin); head.position.y=1.58; g.add(head);
  const helmet=new THREE.Mesh(new THREE.BoxGeometry(.28,.1,.28),mats.acc); helmet.position.y=1.72; g.add(helmet);
  const armL=new THREE.Mesh(new THREE.CapsuleGeometry(.08,.36,2,6),mats.base); armL.position.set(-.28,.98,0); armL.rotation.z=1.55; g.add(armL);
  const armR=new THREE.Mesh(new THREE.CapsuleGeometry(.08,.36,2,6),mats.base); armR.position.set(.28,.98,0); armR.rotation.z=-1.55; g.add(armR);
  const legL=new THREE.Mesh(new THREE.CapsuleGeometry(.09,.46,2,6),mats.vest); legL.position.set(-.12,.34,0); g.add(legL);
  const legR=new THREE.Mesh(new THREE.CapsuleGeometry(.09,.46,2,6),mats.vest); legR.position.set(.12,.34,0); g.add(legR);
  const gun=new THREE.Mesh(new THREE.BoxGeometry(.38,.08,.08),mats.gun); gun.position.set(.34,1.01,-.08); gun.rotation.z=-.15; g.add(gun);
  const strap=new THREE.Mesh(new THREE.BoxGeometry(.08,.55,.03),mats.acc); strap.position.set(.03,.82,.18); strap.rotation.z=-.23; g.add(strap);
  g.userData={torso,head,helmet,gun};
  return g;
}
function createViewModel(){
  const g=new THREE.Group();
  const base=new THREE.MeshStandardMaterial({color:0x1e2328,roughness:.7,metalness:.2});
  const metal=new THREE.MeshStandardMaterial({color:0x6f777d,roughness:.5,metalness:.45});
  const accent=new THREE.MeshStandardMaterial({color:0x20262d,roughness:.8,metalness:.15});
  const body=new THREE.Mesh(new THREE.BoxGeometry(.36,.18,.75),base); body.position.set(.3,-.28,-.8); g.add(body);
  const barrel=new THREE.Mesh(new THREE.BoxGeometry(.12,.1,.46),metal); barrel.position.set(.3,-.28,-1.16); g.add(barrel);
  const grip=new THREE.Mesh(new THREE.BoxGeometry(.13,.28,.12),accent); grip.position.set(.16,-.44,-.74); grip.rotation.z=.15; g.add(grip);
  const mag=new THREE.Mesh(new THREE.BoxGeometry(.09,.28,.11),accent); mag.position.set(.34,-.43,-.8); mag.rotation.z=-.2; g.add(mag);
  const scope=new THREE.Mesh(new THREE.CylinderGeometry(.06,.06,.32,10),metal); scope.rotation.x=Math.PI/2; scope.position.set(.35,-.18,-.96); scope.visible=false; g.add(scope);
  const flash=new THREE.PointLight(0xffe7b0,0,4,.5); flash.position.set(.3,-.28,-1.2); g.add(flash);
  g.userData={body,barrel,grip,mag,scope,flash};
  return g;
}
function setViewModel(vm,w){
  if(!vm) return;
  const u=vm.userData, t=WEAPONS[w];
  if(!t)return;
  const type=t.type;
  u.scope.visible=!!t.scope;
  if(type==='pistol'){u.body.scale.set(.72,.72,.75);u.barrel.scale.set(.72,.72,.72);u.mag.scale.set(.65,.65,.65);u.body.position.set(.25,-.29,-.76);u.barrel.position.set(.25,-.29,-1.05);u.mag.position.set(.18,-.43,-.72)}
  if(type==='smg'){u.body.scale.set(.96,.9,1.08);u.barrel.scale.set(1.05,1.0,1.0);u.mag.scale.set(1.0,1.0,1.0);u.body.position.set(.33,-.29,-.8);u.barrel.position.set(.34,-.29,-1.18);u.mag.position.set(.35,-.42,-.79)}
  if(type==='rifle'){u.body.scale.set(1.15,1.0,1.4);u.barrel.scale.set(1.22,1.0,1.3);u.mag.scale.set(1.12,1.1,1.2);u.body.position.set(.35,-.3,-.86);u.barrel.position.set(.38,-.29,-1.35);u.mag.position.set(.37,-.44,-.84)}
  if(type==='sniper'){u.body.scale.set(1.3,1.0,1.85);u.barrel.scale.set(1.6,1.0,1.9);u.mag.scale.set(1.0,1.0,1.0);u.body.position.set(.36,-.31,-.95);u.barrel.position.set(.42,-.29,-1.65);u.scope.position.set(.38,-.16,-1.2)}
  if(type==='shotgun'){u.body.scale.set(1.28,1.04,1.14);u.barrel.scale.set(1.34,1.0,1.16);u.mag.scale.set(1.05,1.0,1.0);u.body.position.set(.35,-.32,-.86);u.barrel.position.set(.39,-.29,-1.12)}
  u.flash.position.set(.38,-.28,t.type==='sniper'?-1.9:-1.2);
}

class Agent{
  constructor(team,isHuman=false,name='Bot'){
    this.team=team; this.isHuman=isHuman; this.name=name; this.mesh=createSoldier(team); this.mesh.visible=true;
    this.pos=new THREE.Vector3(); this.yaw=0; this.pitch=0; this.vel=new THREE.Vector3(); this.vy=0; this.jump=0; this.onGround=true;
    this.crouch=false; this.sprint=false; this.health=100; this.armor=0; this.helmet=false; this.money=800; this.alive=true; this.score=0;
    this.weapon='usp'; this.secondary='usp'; this.primary=null; this.slot=2; this.reserve={}; this.ammo={}; this.reloadT=0; this.fireCD=0; this.recoil=0; this.recoilYaw=0;
    this.kit=false; this.grenades={he:0,flash:0,smoke:0,molotov:0}; this.bomb=false; this.carryingBomb=false; this.survived=false; this.flash=0; this.aimHeat=0;
    this.viewModel=isHuman?createViewModel():null; if(this.viewModel)camera.add(this.viewModel);
    this.path=[]; this.pathIndex=0; this.pathGoal=null; this.state='idle'; this.target=null; this.plan=null; this.planIndex=0; this.reaction=0; this.lastSeen=0;
    this.nextDecision=0; this.think=0; this.blind=0; this.soundMem=0; this.turnTarget=0; this.assignedRole=0;
    this.mesh.userData.agent=this;
  }
  eye(){return this.pos.clone().add(new THREE.Vector3(0,this.crouch?1.15:1.55+this.jump,0))}
  aliveAndVisible(){return this.alive}
}

class Grenade{
  constructor(type,owner,pos,vel){this.type=type; this.owner=owner; this.pos=pos.clone(); this.vel=vel.clone(); this.timer=type==='molotov'?2.0:type==='smoke'?1.4:type==='flash'?1.2:1.6; this.mesh=this.makeMesh(); this.dead=false; scene.add(this.mesh)}
  makeMesh(){
    const m=new THREE.MeshStandardMaterial({color:this.type==='flash'?0xfff3b0:this.type==='smoke'?0x8e96a0:this.type==='molotov'?0x7d3823:0x2a3035,roughness:.8,metalness:.1});
    const mesh=new THREE.Mesh(new THREE.SphereGeometry(.11,14,10),m); mesh.position.copy(this.pos); return mesh;
  }
  update(dt){
    if(this.dead)return;
    this.timer-=dt; this.vel.y-=9.8*dt;
    let np=this.pos.clone().addScaledVector(this.vel,dt);
    if(np.x<1||np.x>currentMapData.size.w-1){this.vel.x*=-.48; np.x=clamp(np.x,1,currentMapData.size.w-1)}
    if(np.z<1||np.z>currentMapData.size.h-1){this.vel.z*=-.48; np.z=clamp(np.z,1,currentMapData.size.h-1)}
    for(const w of currentMapData.walls){ if(segRect(this.pos.x,this.pos.z,np.x,np.z,w)){ if(Math.abs(np.x-this.pos.x)>Math.abs(np.z-this.pos.z)) this.vel.x*=-.55; else this.vel.z*=-.55; } }
    if(np.y<=0){np.y=0; if(Math.abs(this.vel.y)>2)this.vel.y*=-.34; else this.vel.y=0; this.vel.x*=.72; this.vel.z*=.72}
    this.pos.copy(np); this.mesh.position.copy(this.pos); this.mesh.position.y+=.12; this.mesh.rotation.x+=dt*8; this.mesh.rotation.y+=dt*11;
    if(this.timer<=0){this.explode()}
  }
  explode(){
    if(this.dead)return; this.dead=true; scene.remove(this.mesh);
    if(this.type==='he'){
      fxAudio('explosion');
      for(const a of agents) if(a.alive&&a.team!==this.owner.team){const d=Math.sqrt(dist2(a.pos.x,a.pos.z,this.pos.x,this.pos.z)); if(d<7.5){const dmg=Math.round((1-d/7.5)*92); hurt(a,dmg,this.owner,'he',false)}}
      boomFX(this.pos,0xff9350,2.4,0.7);
      showMsg('HE 爆炸');
    }else if(this.type==='flash'){
      fxAudio('flash'); boomFX(this.pos,0xfff1a6,2.1,0.9);
      for(const a of agents){ if(a.alive&&a.team!==this.owner.team){const d=Math.sqrt(dist2(a.pos.x,a.pos.z,this.pos.x,this.pos.z)); if(d<14.5&&lineClear(this.owner.pos,a.pos,true)){a.blind=Math.max(a.blind,clamp((14.5-d)/14.5*3.8,0.8,3.8)); if(a.isHuman) flashHum(a.blind)} } }
      showMsg('闪光弹');
    }else if(this.type==='smoke'){
      smokeClouds.push({pos:this.pos.clone(),r:0.6,t:18,mesh:makeSmoke(this.pos)}); fxAudio('flash'); showMsg('烟雾弹');
    }else if(this.type==='molotov'){
      fireZones.push({pos:this.pos.clone(),r:4.5,t:8,mesh:makeFire(this.pos)}); fxAudio('explosion'); showMsg('燃烧瓶');
    }
  }
}
class WeaponDrop{
  constructor(name,pos,ammo,reserve,team){this.name=name; this.pos=pos.clone(); this.ammo=ammo; this.reserve=reserve; this.team=team; this.mesh=makeDropMesh(name); this.dead=false; scene.add(this.mesh)}
  update(){this.mesh.position.copy(this.pos); this.mesh.position.y=.16; this.mesh.rotation.y+=.02}
}
function makeDropMesh(name){
  const m=new THREE.MeshStandardMaterial({color:0x1d2329,roughness:.7,metalness:.2});
  const g=new THREE.Group();
  const s=WEAPONS[name];
  const body=new THREE.Mesh(new THREE.BoxGeometry(s.type==='sniper'?1.1:s.type==='rifle'?0.8:.5,.12,s.type==='sniper'?0.18:.1),m); body.rotation.y=Math.PI/2; g.add(body);
  const barrel=new THREE.Mesh(new THREE.BoxGeometry(.35,.06,.06),m); barrel.position.x=.36; barrel.rotation.y=Math.PI/2; g.add(barrel);
  g.scale.set(1,1,1); return g;
}
function makeSmoke(pos){
  const tex=new THREE.CanvasTexture((()=>{const c=document.createElement('canvas');c.width=c.height=128;const g=c.getContext('2d');const gr=g.createRadialGradient(64,64,10,64,64,64);gr.addColorStop(0,'rgba(255,255,255,.9)');gr.addColorStop(.45,'rgba(200,205,210,.55)');gr.addColorStop(1,'rgba(255,255,255,0)');g.fillStyle=gr;g.fillRect(0,0,128,128);return c})());
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:tex,color:0xaeb7bf,transparent:true,opacity:0.68,depthWrite:false}));
  sp.position.copy(pos); sp.scale.set(0.1,0.1,1); scene.add(sp); return sp;
}
function makeFire(pos){
  const tex=new THREE.CanvasTexture((()=>{const c=document.createElement('canvas');c.width=c.height=128;const g=c.getContext('2d');const gr=g.createRadialGradient(64,64,2,64,64,64);gr.addColorStop(0,'rgba(255,220,120,1)');gr.addColorStop(.35,'rgba(255,120,20,.8)');gr.addColorStop(.7,'rgba(160,28,0,.36)');gr.addColorStop(1,'rgba(0,0,0,0)');g.fillStyle=gr;g.fillRect(0,0,128,128);return c})());
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:tex,color:0xffaa55,transparent:true,opacity:0.9,depthWrite:false,blending:THREE.AdditiveBlending}));
  sp.position.copy(pos); sp.position.y=.04; sp.scale.set(0.1,0.1,1); scene.add(sp); return sp;
}
function boomFX(pos,color,size,opacity){
  const geo=new THREE.SphereGeometry(.12,10,8), mat=new THREE.MeshBasicMaterial({color,transparent:true,opacity});
  for(let i=0;i<12;i++){const s=new THREE.Mesh(geo,mat.clone()); s.position.copy(pos); s.userData.v=new THREE.Vector3(rand(-1,1),rand(-.2,1),rand(-1,1)).multiplyScalar(rand(1.5,4)); s.scale.setScalar(rand(.8,1.8)); scene.add(s); effects.push({mesh:s,t:0.7})}
}
function flashHum(d){flash.style.opacity='1'; clearInterval(flashHum.i); let start=performance.now(); flashHum.i=setInterval(()=>{const p=1-(performance.now()-start)/(d*1000); flash.style.opacity=String(clamp(p,0,1)*.95); if(p<=0){clearInterval(flashHum.i); flash.style.opacity='0'}},16)}

function makeWeaponForTeam(team,type){
  if(type==='primary') return team==='T'?'ak47':'m4a4';
  if(type==='smg') return 'mp5';
  if(type==='sniper') return 'awp';
  if(type==='shotgun') return 'nova';
  return team==='T'?'glock':'usp';
}
function buyCost(item){return WEAPONS[item].price||0}

function createGame(){
  scene=new THREE.Scene();
  camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.05,500);
  camera.rotation.order='YXZ';
  renderer=new THREE.WebGLRenderer({antialias:true,powerPreference:'high-performance'});
  renderer.setSize(innerWidth,innerHeight); renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
  renderer.outputColorSpace=THREE.SRGBColorSpace; renderer.toneMapping=THREE.ACESFilmicToneMapping; renderer.toneMappingExposure=1.05;
  document.getElementById('game').prepend(renderer.domElement);
  const amb=new THREE.HemisphereLight(0xffffff,0x263238,1.25); scene.add(amb);
  const sun=new THREE.DirectionalLight(0xfff1d0,1.8); sun.position.set(60,90,20); scene.add(sun);
  const fill=new THREE.DirectionalLight(0xa2c7ff,.6); fill.position.set(-50,50,-60); scene.add(fill);
  clock=new THREE.Clock();

  const floorGeo=new THREE.PlaneGeometry(1,1); floorGeo.rotateX(-Math.PI/2);
  window.addEventListener('resize',()=>{camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth,innerHeight)});
}
createGame();

function buildMap(map){
  while(scene.children.length>0){
    const ch=scene.children[0];
    if(ch.isCamera||ch.type==='AudioListener') { scene.remove(ch); continue; }
    scene.remove(ch);
  }
  const amb=new THREE.HemisphereLight(0xffffff,0x263238,1.15); scene.add(amb);
  const sun=new THREE.DirectionalLight(0xfff1d0,1.8); sun.position.set(60,90,20); scene.add(sun);
  const fill=new THREE.DirectionalLight(map.theme.sky,0.6); fill.position.set(-50,50,-60); scene.add(fill);
  scene.fog=new THREE.FogExp2(map.theme.fog,0.008);
  scene.background=new THREE.Color(map.theme.sky);
  const floorMat=new THREE.MeshStandardMaterial({map:TEX[map.theme.floor],roughness:1,metalness:0});
  floorMat.map.repeat.set(map.size.w/8,map.size.h/8);
  const floor=new THREE.Mesh(new THREE.PlaneGeometry(map.size.w,map.size.h),floorMat); floor.rotation.x=-Math.PI/2; floor.position.set(map.size.w/2,0,map.size.h/2); scene.add(floor);

  const baseMat={
    sand:new THREE.MeshStandardMaterial({map:TEX.sand,roughness:1,metalness:0}),
    concrete:new THREE.MeshStandardMaterial({map:TEX.concrete,roughness:1,metalness:0}),
    steel:new THREE.MeshStandardMaterial({map:TEX.steel,roughness:.65,metalness:.15}),
    metal:new THREE.MeshStandardMaterial({map:TEX.metal,roughness:.65,metalness:.12}),
    wood:new THREE.MeshStandardMaterial({map:TEX.wood,roughness:.95,metalness:0}),
    crate:new THREE.MeshStandardMaterial({map:TEX.crate,roughness:.95,metalness:0}),
    water:new THREE.MeshStandardMaterial({map:TEX.water,roughness:.65,metalness:.05})
  };
  const addBox=(r,h,mat)=>{
    const m=baseMat[mat]||baseMat.sand;
    const mesh=new THREE.Mesh(new THREE.BoxGeometry(r.w,h,r.d),m); mesh.position.set(r.x+r.w/2,h/2,r.z+r.d/2); mesh.castShadow=mesh.receiveShadow=false; scene.add(mesh); return mesh;
  };
  for(const w of map.walls) addBox(w,w.h,w.mat);
  for(const p of map.props) addBox(p,p.h,p.mat);
  const borderColor=new THREE.MeshStandardMaterial({color:0x1a1f22,roughness:1});
  const skybox=new THREE.Mesh(new THREE.BoxGeometry(map.size.w+60,100,map.size.h+60),new THREE.MeshBasicMaterial({color:map.theme.sky,side:THREE.BackSide}));
  skybox.position.set(map.size.w/2,25,map.size.h/2); scene.add(skybox);
  // objective markers
  const mk=(x,z,c,t)=>{const m=new THREE.Mesh(new THREE.CylinderGeometry(1.2,1.2,.15,20),new THREE.MeshStandardMaterial({color:c,emissive:c,emissiveIntensity:.25})); m.position.set(x,.08,z); scene.add(m); return m};
  map.aMarker=mk(map.sites.A.x+map.sites.A.w/2,map.sites.A.z+map.sites.A.d/2,0xff5b5b); map.bMarker=mk(map.sites.B.x+map.sites.B.w/2,map.sites.B.z+map.sites.B.d/2,0x5b9dff);
  map.labelsMeshes=[];
  map.labels.forEach(L=>{const s=new THREE.Sprite(new THREE.SpriteMaterial({color:0xffffff,transparent:true,opacity:.72})); s.position.set(L.x,2.6,L.z); s.scale.set(3,1,1); scene.add(s); map.labelsMeshes.push(s)});
  scene.add(camera);
  camera.position.set(map.size.w/2,1.55,map.size.h/2);
}

function buildGrid(map){
  const cell=4; const gw=Math.ceil(map.size.w/cell), gh=Math.ceil(map.size.h/cell), g=Array.from({length:gw},()=>Array(gh).fill(0));
  for(let i=0;i<gw;i++) for(let j=0;j<gh;j++){
    const x=i*cell+.5*cell, z=j*cell+.5*cell;
    for(const o of [...map.walls,...map.props]) if(rectOverlapCircle(o,x,z,1.2)){ g[i][j]=1; break; }
  }
  return {cell,gw,gh,g};
}
function toCell(v,cell){return {x:clamp(Math.floor(v.x/cell),0,9999),y:clamp(Math.floor(v.z/cell),0,9999)}}
function fromCell(c,cell){return new THREE.Vector2((c.x+.5)*cell,(c.y+.5)*cell)}
function pathFind(map,start,end){
  const grid=map.nav; const s=toCell(start,grid.cell), e=toCell(end,grid.cell);
  if(s.x<0||s.y<0||e.x<0||e.y<0)return [];
  const key=(x,y)=>x+','+y, open=[[s.x,s.y,0,0]], came=new Map(), gScore=new Map([[key(s.x,s.y),0]]), inO=new Set([key(s.x,s.y)]);
  const h=(x,y)=>Math.abs(x-e.x)+Math.abs(y-e.y);
  while(open.length){
    open.sort((a,b)=>a[2]-b[2]); const cur=open.shift(); const [x,y,g]=cur; const ck=key(x,y); inO.delete(ck);
    if(x===e.x&&y===e.y){ const path=[e]; let k=ck; while(came.has(k)){const p=came.get(k); path.push({x:p[0],y:p[1]}); k=key(p[0],p[1])} path.reverse(); return path.map(c=>fromCell(c,grid.cell)) }
    for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){
      const nx=x+dx, ny=y+dy; if(nx<0||ny<0||nx>=grid.gw||ny>=grid.gh||grid.g[nx][ny]) continue;
      const nk=key(nx,ny); const ng=g+1;
      if(!gScore.has(nk)||ng<gScore.get(nk)){gScore.set(nk,ng); came.set(nk,[x,y]); const f=ng+h(nx,ny); if(!inO.has(nk)){open.push([nx,ny,f]); inO.add(nk)}}
    }
  }
  return [];
}

function lineClear(from,to,ignoreSmoke=false){
  for(const w of currentMapData.walls) if(segRect(from.x,from.z,to.x,to.z,w)) return false;
  if(!ignoreSmoke) for(const s of smokeClouds) if(segCircle(from.x,from.z,to.x,to.z,s.pos.x,s.pos.z,Math.max(1.8,s.r))) return false;
  return true;
}
function rayPlayerHit(origin,dir,p,maxRange=120){
  const hs=[{c:p.pos.clone().add(new THREE.Vector3(0,1.55,0)),r:.22,m:4.0},{c:p.pos.clone().add(new THREE.Vector3(0,1.0,0)),r:.38,m:1.0}];
  let best=null;
  for(const h of hs){
    const oc=origin.clone().sub(h.c), b=2*oc.dot(dir), c=oc.dot(oc)-h.r*h.r, disc=b*b-4*c; if(disc<0) continue;
    const t=(-b-Math.sqrt(disc))/2; if(t>0&&t<maxRange&&(best===null||t<best.t)) best={t,head:h.m>1};
  }
  return best;
}
function wallRay(origin,dir,maxRange=120){
  let min=maxRange;
  const end=origin.clone().addScaledVector(dir,maxRange);
  for(const w of currentMapData.walls){const r={x:w.x,z:w.z,w:w.w,d:w.d}; if(segRect(origin.x,origin.z,end.x,end.z,r)){ 
    const t=0.5; min=Math.min(min,dist2(origin.x,origin.z,end.x,end.z)); break;
  }}
  return min;
}
function dirFromYawPitch(yaw,pitch){
  const d=new THREE.Vector3(Math.sin(yaw)*Math.cos(pitch),Math.sin(-pitch),-Math.cos(yaw)*Math.cos(pitch)); return d.normalize();
}
function makeTracer(a,b,col=0xfff4dc){
  const geo=new THREE.BufferGeometry().setFromPoints([a,b]); const line=new THREE.Line(geo,new THREE.LineBasicMaterial({color:col,transparent:true,opacity:.9}));
  scene.add(line); effects.push({mesh:line,t:.06,kind:'line'})
}

function emitSound(pos,radius,type='shot',strength=1){soundEvents.push({pos:pos.clone(),radius,type,t:strength})}

function hurt(victim,dmg,attacker,weapon,headshot=false){
  if(!victim.alive) return;
  let final=dmg;
  if(victim.armor>0){const absorb=headshot?0.15:0.45; const a=Math.min(victim.armor,Math.ceil(final*absorb)); victim.armor-=a; final-=a;}
  victim.health-=Math.round(final);
  if(victim.health<=0){kill(victim,attacker,weapon,headshot); return true}
  victim.aimHeat=Math.max(victim.aimHeat,0.1);
  return false;
}
function kill(victim,killer,weapon,headshot=false){
  victim.alive=false; victim.survived=false; victim.mesh.visible=false; if(victim.isHuman&&victim===human){showMsg('你阵亡了');}
  if(victim.primary){drops.push(new WeaponDrop(victim.primary,victim.pos.clone().add(new THREE.Vector3(0,0.1,0)),victim.ammo[victim.primary]||0,victim.reserve[victim.primary]||0,victim.team))}
  if(victim.bomb&&bomb.carrier===victim){bomb.carrier=null; bomb.dropped=victim.pos.clone(); bomb.mesh.visible=true; bomb.mesh.position.copy(victim.pos).add(new THREE.Vector3(0,.12,0));}
  let name='World'; let col='#cfd6dc'; if(killer){name=killer.name; col=randomColorHex(TEAMCOL[killer.team]); if(killer.isHuman) killer.money+=victim.team===killer.team?0:300}
  addFeed(`${name} >> ${victim.name}${headshot?' [HS]':''}${weapon?` (${weapon})`:''}`,col);
  fxAudio('reload');
  checkRoundEnd();
}

function buyAuto(a){
  if(a.money<500) return;
  const d=a.team, diff=botDiff;
  const useSniper=(diff==='hard'&&Math.random()>.7);
  const primary=(d==='CT'?'m4a4':'ak47');
  const pistol=(d==='CT'?'usp':'glock');
  if(a.money>=WEAPONS[useSniper?'awp':primary].price+1000&&useSniper){buyWeapon(a,'awp');}
  else if(a.money>=WEAPONS[primary].price+700){buyWeapon(a,primary);}
  else if(a.money>=WEAPONS['mp5'].price+350){buyWeapon(a,'mp5');}
  if(a.money>=WEAPONS[pistol].price&&a.secondary===null) buyWeapon(a,pistol,true);
  if(a.money>=1000){buyArmor(a,true);}
  if(a.money>=300){buyGrenade(a,'he');}
  if(a.money>=200){buyGrenade(a,'flash');}
  if(a.money>=300&&Math.random()>.5) buyGrenade(a,'smoke');
  if(a.team==='CT'&&a.money>=400) buyKit(a);
}
function buyArmor(a,helmet=true){const cost=helmet?1000:650; if(a.money>=cost){a.money-=cost; a.armor=helmet?100:100; a.helmet=helmet; fxAudio('buy'); return true} return false}
function buyKit(a){if(a.money>=400&&!a.kit){a.money-=400; a.kit=true; fxAudio('buy'); return true} return false}
function buyGrenade(a,t){if(a.money>=WEAPONS[t].price){a.money-=WEAPONS[t].price; a.grenades[t]++; fxAudio('buy'); return true} return false}
function buyWeapon(a,name,secondary=false){
  const w=WEAPONS[name]; if(a.money<w.price) return false; a.money-=w.price;
  if(w.type==='pistol'){a.secondary=name; if(!a.primary||secondary){a.weapon=name; a.slot=2; a.ammo[name]=w.clip; a.reserve[name]=w.reserve; setViewModel(a.viewModel,name);}}
  else{a.primary=name; a.weapon=name; a.slot=1; a.ammo[name]=w.clip; a.reserve[name]=w.reserve; setViewModel(a.viewModel,name);}
  fxAudio('buy'); return true;
}
function equipDefault(a,keep=false){
  const d=a.team; a.weapon=d==='T'?'glock':'usp'; a.secondary=d==='T'?'glock':'usp'; a.primary=null; a.slot=2;
  a.ammo[a.weapon]=WEAPONS[a.weapon].clip; a.reserve[a.weapon]=WEAPONS[a.weapon].reserve;
  if(!keep){a.armor=0;a.helmet=false;a.kit=false;a.grenades={he:0,flash:0,smoke:0,molotov:0}}
  setViewModel(a.viewModel,a.weapon);
}
function ensureLoadout(a){
  if(!a.primary&&a.weapon!==a.secondary&&a.weapon!==a.primary) { if(a.team==='T'){a.weapon='glock'} else {a.weapon='usp'} }
  if(!a.ammo[a.weapon]) a.ammo[a.weapon]=WEAPONS[a.weapon].clip;
  if(!a.reserve[a.weapon]) a.reserve[a.weapon]=WEAPONS[a.weapon].reserve;
  setViewModel(a.viewModel,a.weapon);
}
function switchWeapon(a,slot){
  if(slot===3){a.weapon='knife'; a.slot=3}
  else if(slot===2&&a.secondary){a.weapon=a.secondary; a.slot=2}
  else if(slot===1&&a.primary){a.weapon=a.primary; a.slot=1}
  else if(slot>=4&&slot<=7){a.weapon=['he','flash','smoke','molotov'][slot-4]; a.slot=slot}
  ensureLoadout(a);
}
function getCurrentWeapon(a){return WEAPONS[a.weapon]}
function canThrow(a,t){return a.grenades[t]>0}
function setRoundMsg(t){showMsg(t,1600); if(t.includes('开始')) fxAudio('round')}

const smokeClouds=[], fireZones=[], drops=[], bullets=[], effects=[], soundEvents=[];
const botNames=['Raven','Crow','Lynx','Ghost','Moss','Nova','Viper','Echo','Pulse','Sable','Vector','Rook','Ash','Hex','Mule'];
let humans=[], teams={T:[],CT:[]}, agents=[], human=null, bomb={carrier:null,planted:false,dropped:null,pos:null,mesh:null,site:null,timer:0,defuse:0,planting:0,defusingBy:null};
let round=1, score={T:0,CT:0}, freeze=5, buyTime=20, roundTime=115, roundClock=0, bombClock=0, plantTime=0, defuseTime=0, roundState='freeze', botDiff='normal', teamSize=5, playerSide='CT', buyOpen=false;
let matchInfo='', lastFrame=0, spectatorIndex=0, currentObjective=null;

function spawnBombModel(){
  const m=new THREE.MeshStandardMaterial({color:0x20262d,roughness:.8,metalness:.15});
  bomb.mesh=new THREE.Mesh(new THREE.BoxGeometry(.18,.12,.14),m); bomb.mesh.visible=false; scene.add(bomb.mesh);
  const light=new THREE.PointLight(0xff5533,0,2,.4); light.position.set(0,.18,0); bomb.mesh.add(light); bomb.mesh.userData.light=light;
}
function clearWorld(){
  for(const a of agents){if(a.viewModel&&a.viewModel.parent) a.viewModel.parent.remove(a.viewModel)}
  for(const o of [...drops,...effects]) if(o.mesh&&o.mesh.parent) o.mesh.parent.remove(o.mesh);
  while(scene.children.length>0) scene.remove(scene.children[0]);
  agents=[]; teams={T:[],CT:[]}; humans=[]; drops.length=0; smokeClouds.length=0; fireZones.length=0; bullets.length=0; effects.length=0; soundEvents.length=0;
}

function initMatch(){
  ensureAudio(); hud.classList.remove('hidden'); lobby.classList.add('hidden'); pauseUI.classList.add('hidden'); overUI.classList.add('hidden'); buyUI.classList.add('hidden');
  matchEnded=false; paused=false; inBuy=false; buyOpen=false; round=1; score={T:0,CT:0}; freeze=5; buyTime=20; roundTime=115; roundClock=0; roundState='freeze'; botDiff=difficultySel.value; teamSize=+teamSizeSel.value; playerSide=sideSel.value;
  currentMap=MAPS[mapSel.value]; clearWorld(); buildMap(currentMap); currentMap.nav=buildGrid(currentMap); spawnBombModel(); setupTeams(); startRound(true);
  document.body.requestPointerLock(); gameStarted=true; updateBuyMenu(); loop(); showMsg('进入对局');
}
function setupTeams(){
  const humanName='You'; human=new Agent(playerSide,true,humanName); agents.push(human); teams[playerSide].push(human); humans=[human];
  const other=playerSide==='T'?'CT':'T'; const humanBots=teamSize-1, enemyBots=teamSize;
  const addBot=(team,idx)=>{const a=new Agent(team,false,botNames[(idx+Math.random()*botNames.length)|0]+'_'+(idx+1)); a.money=800; agents.push(a); teams[team].push(a); return a};
  for(let i=0;i<humanBots;i++) addBot(playerSide,i);
  for(let i=0;i<enemyBots;i++) addBot(other,i+9);
  for(const a of agents){a.health=100; a.armor=0; a.helmet=false; a.kit=false; a.alive=true; a.mesh.visible=true; scene.add(a.mesh); if(a.isHuman&&a.viewModel) camera.add(a.viewModel)}
  for(const a of agents) equipDefault(a,false);
}
function startRound(first=false){
  roundState='freeze'; freeze=5; buyTime=20; roundClock=currentMapData?roundTime:115; if(!first){round++}
  bomb.planted=false; bomb.dropped=null; bomb.timer=0; bomb.defuse=0; bomb.carrier=null; bomb.pos=null; if(bomb.mesh) bomb.mesh.visible=false; if(bomb.mesh&&bomb.mesh.userData.light) bomb.mesh.userData.light.intensity=0;
  const map=currentMapData=currentMap;
  for(const a of agents){
    a.alive=true; a.health=100; a.recoil=0; a.recoilYaw=0; a.blind=0; a.crouch=false; a.vy=0; a.jump=0; a.onGround=true; a.path=[]; a.pathIndex=0; a.state='idle'; a.target=null; a.turnTarget=a.yaw;
    a.mesh.visible=true; a.survived=false; a.reloadT=0; a.fireCD=0; a.aimHeat=0; a.primary=a.survived&&a.primary?a.primary:a.primary; // keep if survived
    if(!a.survived) { equipDefault(a,false); if(a.team==='T'&&!a.isHuman&&Math.random()>.1) {a.primary='ak47'; a.weapon=a.primary; a.ammo[a.weapon]=WEAPONS[a.weapon].clip; a.reserve[a.weapon]=WEAPONS[a.weapon].reserve} }
    if(a.survived && a.primary){a.weapon=a.primary; a.slot=1}
    if(a.isHuman && a.primary===null && a.weapon!=='knife'){}
    if(a.team==='T'&&a.bomb) a.carryingBomb=true;
    const spawnArr=map.spawns[a.team]; const s=spawnArr[(Math.random()*spawnArr.length)|0]; a.pos.set(s[0],0,s[1]);
    if(a.team==='CT'){if(Math.random()>.65) buyKit(a)} else {a.carryingBomb=false}
    if(!a.isHuman) autoBuyRound(a);
  }
  const tplayers=teams.T.filter(a=>a.alive), carrier=tplayers[(Math.random()*tplayers.length)|0]; if(carrier){bomb.carrier=carrier; carrier.bomb=true; carrier.carryingBomb=true}
  if(human.team==='T'&&Math.random()>.45 && !human.primary) human.primary='ak47';
  if(bomb.carrier) showMsg(`${bomb.carrier.name} 持有 C4`,1500);
  setRoundMsg('回合开始');
  updateBuyMenu();
  roundEl.textContent=`Round ${round}`;
}
function autoBuyRound(a){
  if(a.alive){
    if(Math.random()>.3) buyArmor(a,true); else if(a.money>=650) buyArmor(a,false);
    if(a.team==='CT'&&Math.random()>.5) buyKit(a);
    if(a.team==='T'&&Math.random()>.65) buyGrenade(a,'smoke');
    if(a.team==='T'&&Math.random()>.35) buyGrenade(a,'flash');
    if(a.team==='CT'&&Math.random()>.5) buyGrenade(a,'flash');
    if(a.money>=WEAPONS[(a.team==='T'?'ak47':'m4a4')].price && Math.random()>.2) buyWeapon(a,a.team==='T'?'ak47':'m4a4');
    else if(a.money>=WEAPONS.mp5.price && Math.random()>.35) buyWeapon(a,'mp5');
  }
}
function refreshCashOnRoundEnd(winner){
  const losers=winner==='T'?'CT':'T';
  for(const a of teams[winner]){
    if(a.alive) a.money+=3500; else a.money+=1400;
    if(a===bomb.carrier&&winner==='T'&&bomb.planted) a.money+=800;
  }
  for(const a of teams[losers]){
    a.money+=Math.min(3400,1400+Math.max(0,Math.min(4,lossStreak[losers]))*500);
  }
}
const lossStreak={T:0,CT:0};

function endRound(winner,reason){
  if(matchEnded) return;
  roundState='ended'; score[winner]++; lossStreak[winner]=0; lossStreak[winner==='T'?'CT':'T']++;
  refreshCashOnRoundEnd(winner);
  for(const a of agents) a.survived=a.alive;
  addFeed(`${winner} 获胜 - ${reason}`,winner==='T'?'#f3c97a':'#92c8ff'); showMsg(`${winner} 胜利：${reason}`,2000); fxAudio('round');
  updateScore();
  if(score[winner]>=ROUNDS_TO_WIN){matchEnded=true; endTitle.textContent=`${winner} 获胜`; endText.textContent=`最终比分：T ${score.T} : ${score.CT} CT`; overUI.classList.remove('hidden'); return}
  setTimeout(()=>{startRound(false)},2200);
}
function updateScore(){scoreEl.textContent=`T ${score.T} : ${score.CT} CT`; updateRoundHud()}
function updateRoundHud(){timerEl.textContent=roundState==='freeze'?`Freeze ${fmtTime(freeze)}`:bomb.planted?fmtTime(bomb.timer):fmtTime(roundClock)}
function checkRoundEnd(){
  if(matchEnded||roundState==='ended') return;
  const Tlive=teams.T.some(a=>a.alive), CTlive=teams.CT.some(a=>a.alive);
  if(bomb.planted){ if(bomb.timer<=0) endRound('T','炸弹爆炸'); else if(!Tlive && !CTlive){} }
  else {
    if(!Tlive){endRound('CT','全歼 Terrorists')}
    else if(!CTlive){endRound('T','全歼 Counter-Terrorists')}
  }
}

function updateBuyMenu(){
  buyItems.innerHTML='';
  const items=[
    ['ak47','AK-47'],['m4a4','M4A4'],['awp','AWP'],
    ['mp5','MP5'],['nova','Nova'],['deagle','Deagle'],
    ['usp','USP-S'],['glock','Glock-18'],['armor','Kevlar+Helmet'],['kit','Defuse Kit'],
    ['he','HE Grenade'],['flash','Flashbang'],['smoke','Smoke Grenade'],['molotov','Molotov']
  ];
  for(const [key,name] of items){
    const t=WEAPONS[key]; const d=document.createElement('div'); d.className='item';
    d.innerHTML=`<div class="name">${name}</div><div class="meta">${t?.type==='rifle'?'主武器':t?.type==='smg'?'冲锋枪':t?.type==='sniper'?'狙击枪':t?.type==='shotgun'?'霰弹枪':t?.type==='pistol'?'手枪':key==='armor'?'护甲头盔':key==='kit'?'拆弹器':'投掷物'}</div><div class="cost">${key==='armor'?'$1000':key==='kit'?'$400':`$${t.price}`}</div><button>购买</button>`;
    d.querySelector('button').onclick=()=>{
      if(!human||!human.alive) return;
      if(key==='armor') buyArmor(human,true);
      else if(key==='kit') buyKit(human);
      else if(['he','flash','smoke','molotov'].includes(key)) buyGrenade(human,key);
      else buyWeapon(human,key);
      syncUI(); updateBuyMenu(); buyOpen=true;
    };
    buyItems.appendChild(d);
  }
}

function plantBomb(a){
  if(bomb.planted||a.team!=='T'||!a.alive) return false;
  const sites=currentMapData.bombZones; let inSite=false, site=null;
  for(const s of sites) if(pointInRect(a.pos.x,a.pos.z,s)){inSite=true; site=s; break}
  if(!inSite) return false;
  bomb.planting=3.0; bomb.defusingBy=null; bomb.site=site; bomb.carrier=a; bomb.pos=a.pos.clone();
  setBar(0,'正在安放 C4',true); fxAudio('plant'); showMsg('正在下包',1000); return true;
}
function defuseBomb(a){
  if(!bomb.planted||a.team!=='CT'||!a.alive) return false;
  const bpos=bomb.pos||bomb.mesh.position; if(Math.sqrt(dist2(a.pos.x,a.pos.z,bpos.x,bpos.z))>2.4) return false;
  bomb.defusingBy=a; bomb.defuse=a.kit?5:10; setBar(0,'正在拆包',true); fxAudio('defuse'); return true;
}
function completePlant(a){
  bomb.planted=true; bomb.planting=0; bomb.timer=40; bomb.dropped=null; bomb.carrier=a; bomb.pos=a.pos.clone(); bomb.mesh.visible=true; bomb.mesh.position.set(bomb.pos.x,.08,bomb.pos.z); bomb.mesh.userData.light.intensity=1.7; setBar(0,'',false); showMsg('炸弹已安放',1200)
}
function completeDefuse(a){
  bomb.planted=false; bomb.defuse=0; bomb.defusingBy=null; bomb.mesh.visible=false; bomb.mesh.userData.light.intensity=0; setBar(0,'',false); endRound('CT','成功拆包')
}
function pickupNearby(a){
  if(!a.alive) return;
  if(a.team==='T' && bomb.dropped){ const d=Math.sqrt(dist2(a.pos.x,a.pos.z,bomb.dropped.x,bomb.dropped.z)); if(d<1.5){bomb.carrier=a; a.bomb=true; a.carryingBomb=true; bomb.dropped=null; showMsg('已拾取 C4'); return} }
  for(const d of drops){ if(d.dead) continue; const dist=Math.sqrt(dist2(a.pos.x,a.pos.z,d.pos.x,d.pos.z)); if(dist<1.2){ if(!a.primary || WEAPONS[d.name].type!=='pistol'){ 
      if(WEAPONS[d.name].type==='pistol'){a.secondary=d.name; a.weapon=d.name; a.slot=2}else{a.primary=d.name; a.weapon=d.name; a.slot=1}
      a.ammo[d.name]=d.ammo; a.reserve[d.name]=d.reserve; setViewModel(a.viewModel,d.name); d.dead=true; scene.remove(d.mesh); showMsg(`拾取 ${WEAPONS[d.name].name}`); return;
    }}}
  if(a.team==='CT' && bomb.planted && Math.sqrt(dist2(a.pos.x,a.pos.z,(bomb.pos||bomb.mesh.position).x,(bomb.pos||bomb.mesh.position).z))<2.4) defuseBomb(a);
}
function useAction(a){
  if(!a.alive) return;
  if(bomb.planted){ if(a.team==='CT') defuseBomb(a); return; }
  if(a.team==='T' && currentMapData.bombZones.some(z=>pointInRect(a.pos.x,a.pos.z,z))) { if(plantBomb(a)) return; }
  pickupNearby(a);
}

function selectClosestEnemy(a,maxDist=80){
  let best=null, bestD=maxDist*maxDist;
  for(const b of agents){ if(!b.alive||b.team===a.team) continue; const d=dist2(a.pos.x,a.pos.z,b.pos.x,b.pos.z); if(d<bestD){ if(lineClear(a.eye(),b.eye())){best=b; bestD=d}}}
  return best;
}
function aimAt(a,target,dt,skill=0.5){
  const eye=a.eye(), tgt=target.eye?target.eye():target.pos.clone().add(new THREE.Vector3(0,1.2,0));
  const dx=tgt.x-eye.x, dz=tgt.z-eye.z, dy=tgt.y-eye.y;
  const yaw=Math.atan2(dx,-dz), pitch=-Math.atan2(dy,Math.hypot(dx,dz));
  const turn=clamp(dt*(2.5+skill*5),0,1); a.yaw+=angleDelta(a.yaw,yaw)*turn; a.pitch+=angleDelta(a.pitch,pitch)*turn;
}
function angleDelta(a,b){let d=b-a; while(d>Math.PI)d-=TAU; while(d<-Math.PI)d+=TAU; return d}

function fireWeapon(a,dt=0){
  if(!a.alive||a.reloadT>0||a.fireCD>0) return false;
  let w=WEAPONS[a.weapon]; if(!w) return false;
  if(a.weapon==='knife'){
    const enemy=selectClosestEnemy(a,2.1); if(enemy){hurt(enemy,48,a,'knife',false); killMarker(); emitSound(a.pos,6,'melee',0.3); fxAudio('step'); return true}
    return false;
  }
  if(a.weapon==='he'||a.weapon==='flash'||a.weapon==='smoke'||a.weapon==='molotov'){ throwGrenade(a,a.weapon); return true; }
  if((a.ammo[a.weapon]||0)<=0){reload(a); return false}
  if(roundState==='freeze' || paused) return false;
  if(gameTime<a.fireCD) return false;
  if(w.type==='shotgun'){
    a.ammo[a.weapon]--; for(let i=0;i<w.pellets;i++) shootRay(a,w,0.035+Math.max(0,a.recoil)*0.7,true); a.fireCD=gameTime+1/w.rate; shotAudio('shotgun'); a.recoil+=w.recoil; emitSound(a.pos,12,'shot',0.6); return true;
  }
  a.ammo[a.weapon]--; shootRay(a,w,w.spread); a.fireCD=gameTime+1/w.rate; shotAudio(w.type==='sniper'?'sniper':w.type==='rifle'?'rifle':'pistol'); a.recoil+=w.recoil; emitSound(a.pos,18,'shot',0.9); return true;
}
function shootRay(a,w,baseSpread,shotgun=false){
  const origin=a.eye(); const spread=baseSpread + (a.crouch?0.003:0) + (a.sprint?.018:0) + a.recoil*0.01 + (a.aimHeat||0);
  let yaw=a.yaw+(Math.random()*2-1)*spread*3 + a.recoilYaw; let pitch=a.pitch+(Math.random()*2-1)*spread*3 + a.recoil*0.02;
  let dir=dirFromYawPitch(yaw,pitch);
  const maxR=w.range||80; let wallT=maxR, wallHit=origin.clone().addScaledVector(dir,maxR);
  for(const ob of currentMapData.walls){const end=origin.clone().addScaledVector(dir,maxR); if(segRect(origin.x,origin.z,end.x,end.z,ob)){const approx=Math.sqrt(dist2(origin.x,origin.z,ob.x+ob.w/2,ob.z+ob.d/2)); if(approx<wallT) wallT=approx}}
  let hit=null;
  for(const e of agents){ if(e===a||!e.alive||e.team===a.team) continue; const r=rayPlayerHit(origin,dir,e,maxR); if(r&&r.t<wallT){ if(!hit||r.t<hit.t) hit={entity:e,t:r.t,head:r.head} } }
  const hitPoint=origin.clone().addScaledVector(dir,hit?hit.t:Math.min(wallT,maxR));
  makeTracer(origin,hitPoint,hit?0xffe7a2:0xcfd8e3);
  if(hit){ const dmg=w.damage*(hit.head?w.hs:1); const killed=hurt(hit.entity,dmg,a,a.weapon,hit.head); killMarker(); if(killed&&a.isHuman) a.money+=300; if(hit.entity.isHuman&&hit.entity===human&&a.isHuman){} }
  if(w.scope && a.isHuman && a.viewModel) a.viewModel.userData.flash.intensity=.7;
}
function reload(a){
  const w=WEAPONS[a.weapon]; if(!w||a.weapon==='knife'||a.weapon==='he'||a.weapon==='flash'||a.weapon==='smoke'||a.weapon==='molotov') return;
  if((a.reserve[a.weapon]||0)<=0 || (a.ammo[a.weapon]||0)>=w.clip) return;
  a.reloadT=w.type==='sniper'?2.3:w.type==='shotgun'?1.9:1.5; fxAudio('reload'); showMsg('换弹',500);
}
function finishReload(a){
  const w=WEAPONS[a.weapon]; if(!w) return; const need=w.clip-(a.ammo[a.weapon]||0), take=Math.min(need,a.reserve[a.weapon]||0); a.ammo[a.weapon]=(a.ammo[a.weapon]||0)+take; a.reserve[a.weapon]-=take
}
function throwGrenade(a,type){
  if(!a.grenades[type]||a.grenades[type]<=0) return false;
  a.grenades[type]--; const origin=a.eye(); const dir=dirFromYawPitch(a.yaw, a.pitch-.07);
  const speed=type==='smoke'?14:type==='flash'?17:type==='molotov'?14:18;
  const vel=dir.multiplyScalar(speed).add(new THREE.Vector3(0,4.7,0));
  grenades.push(new Grenade(type,a,origin,vel)); emitSound(a.pos,14,'throw',.4); return true;
}

const grenades=[]; let gameTime=0;
function updateMovement(a,dt){
  if(!a.alive){ if(a.isHuman&&human===a) spectateUpdate(dt); return; }
  const base=3.55, sprint=5.15, crouch=2.05, mul=getCurrentWeapon(a)?.move||1, speed=(a.crouch?crouch:a.sprint?sprint:base)*mul*(a.blind>0?.68:1);
  const forward=new THREE.Vector2(Math.sin(a.yaw),-Math.cos(a.yaw)), right=new THREE.Vector2(Math.cos(a.yaw),Math.sin(a.yaw));
  let mx=0,mz=0;
  if(a.isHuman && roundState!=='freeze' && !paused){ if(keys['KeyW']){mx+=forward.x;mz+=forward.y} if(keys['KeyS']){mx-=forward.x;mz-=forward.y} if(keys['KeyD']){mx+=right.x;mz+=right.y} if(keys['KeyA']){mx-=right.x;mz-=right.y} }
  else if(!a.isHuman && a.path.length){ const p=a.path[a.pathIndex]; if(p){const dx=p.x-a.pos.x,dz=p.y-a.pos.z,dist=Math.hypot(dx,dz); if(dist<1.25) a.pathIndex++; else {const n=norm2(dx,dz); mx+=n[0]; mz+=n[1];}}}
  if(mx||mz){const n=norm2(mx,mz); mx=n[0]*speed; mz=n[1]*speed}
  a.sprint=a.isHuman&&keys['ShiftLeft']&&!a.crouch;
  if(a.isHuman) a.crouch=keys['ControlLeft']||keys['ControlRight'];
  a.vel.x=mx; a.vel.z=mz;
  a.jump+=a.vy*dt; a.vy-=11*dt; if(a.jump<=0){a.jump=0;a.vy=0;a.onGround=true}
  if(a.isHuman && keys['Space'] && a.onGround && roundState!=='freeze'){ a.vy=4.8; a.onGround=false; keys['Space']=false }
  const np=a.pos.clone().addScaledVector(a.vel,dt);
  const r=a.crouch?.26:.34;
  let px=np.x, pz=np.z;
  // bounds
  px=clamp(px,r,currentMapData.size.w-r); pz=clamp(pz,r,currentMapData.size.h-r);
  // collision push
  for(const o of [...currentMapData.walls,...currentMapData.props]){
    if(rectOverlapCircle(o,px,pz,r)){
      const left=Math.abs(px-o.x), rightv=Math.abs((o.x+o.w)-px), top=Math.abs(pz-o.z), bottom=Math.abs((o.z+o.d)-pz);
      const m=Math.min(left,rightv,top,bottom);
      if(m===left) px=o.x-r; else if(m===rightv) px=o.x+o.w+r; else if(m===top) pz=o.z-r; else pz=o.z+o.d+r;
    }
  }
  a.pos.set(px,0,pz);
  if(a.isHuman||a===human) camera.position.set(a.pos.x,1.55+a.jump-(a.crouch?.45:0),a.pos.z);
}
function spectateUpdate(dt){
  const list=teams[human.team].filter(a=>a.alive);
  if(!list.length){camera.position.y=16; return}
  const target=list[spectatorIndex%list.length]; camera.position.lerp(new THREE.Vector3(target.pos.x,1.55,target.pos.z),0.08); camera.rotation.y=target.yaw; camera.rotation.x=target.pitch;
}
function updateBot(a,dt){
  if(!a.alive) return;
  const diff=DIFF[botDiff];
  if(a.blind>0){a.blind=Math.max(0,a.blind-dt); a.state='blind'}
  const visible=selectClosestEnemy(a,30);
  const bombPos=bomb.planted?(bomb.pos||bomb.mesh.position):null;
  if(bomb.planted && a.team==='CT'){ a.state='retake'; a.pathGoal=bombPos||new THREE.Vector3(); if(!a.path.length||a.pathIndex>=a.path.length||Math.random()<.02) a.path=pathFind(currentMapData,a.pos,bombPos); if(a.path.length&&a.pathIndex<a.path.length) moveAI(a,dt); if(bombPos && Math.sqrt(dist2(a.pos.x,a.pos.z,bombPos.x,bombPos.z))<2.5) defuseBomb(a); if(visible){a.state='combat'; aimAt(a,visible,dt,1-diff.aim*4); if(gameTime>a.fireCD) fireWeapon(a,dt)} return }
  if(visible && a.blind<=0){
    a.lastSeen=gameTime; a.state='combat'; a.target=visible;
    if(gameTime>a.reaction){ aimAt(a,visible,dt,1-diff.aim*3); if(gameTime>a.fireCD) fireWeapon(a,dt) }
    if(a.grenades.flash>0 && Math.random()<diff.gren && Math.sqrt(dist2(a.pos.x,a.pos.z,visible.pos.x,visible.pos.z))>8){ throwGrenade(a,'flash') }
    return;
  }
  if(a.team==='T'){
    if(bomb.carrier===a && !bomb.planted){ const plans=currentMapData.plans.T; if(!a.plan){a.plan=plans[(Math.random()*plans.length)|0]; a.planIndex=0} const goal=a.plan[a.planIndex]||a.plan[a.plan.length-1]; if(goal){a.path=pathFind(currentMapData,a.pos,new THREE.Vector3(goal.x,0,goal.z)); moveAI(a,dt); if(Math.sqrt(dist2(a.pos.x,a.pos.z,goal.x,goal.z))<2.2) a.planIndex++}
      if(currentMapData.bombZones.some(z=>pointInRect(a.pos.x,a.pos.z,z))){ plantBomb(a) }
    } else {
      if(!a.plan){a.plan=currentMapData.plans.T[(Math.random()*currentMapData.plans.T.length)|0]; a.planIndex=0}
