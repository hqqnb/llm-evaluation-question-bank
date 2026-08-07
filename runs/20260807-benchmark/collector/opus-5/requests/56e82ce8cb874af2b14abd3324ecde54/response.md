## 先说一件必须坦白的事

这个会话里我**没有联网检索工具**，无法真的去搜索并逐页阅读原文。所以我没有假装"读完了"——下面的飞船是我依据**金·斯坦利·罗宾逊《极光》(Kim Stanley Robinson, *Aurora*, 2015)** 已有的认知重建的。哪些是原著明确设定、哪些是我按小说逻辑补的，我在界面的"设定依据"面板里逐条标了出来，代码注释里也标了 `(推测)`。

**原著中我有把握的核心设定（模型据此建造）：**
- 世代飞船，从土星轨道出发飞往鲸鱼座τ（Tau Ceti，11.9 光年），巡航速度约 0.1c，航行约 170 年，船上约 2000 人；目的地是行星 E 的卫星，被命名为"极光 (Aurora)"。
- 结构：一根长中轴（spine）+ **两个环（Ring A / Ring B）**，每环 **12 个生物群落（biome）**，共 **24 个**，以气闸相连；环靠自转产生约 1 g 重力；群落内部由灯带照明、有昼夜循环，不靠舷窗采光。
- 群落以地球生物地理区命名（Nova Scotia、Costa Rica、Amazonia、Patagonia、Labrador、The Steppes、Olympia、Tasmania、Bengal、Sonora、Kilimanjaro、Yangtze、Iran、Mongolia、Siberia、Pampas 等）；主角 Freya 生活在 Ring B 的 Nova Scotia。
- 船首有巨大防护盾（0.1c 下星际尘埃/氢是致命的）；中轴上有燃料、制造车间（"打印机"）、枢纽与渡船坞；叙述者就是飞船自己的量子计算机。

**我推测/补齐的部分**（原著未给精确数字或我记忆不确定）：环半径 2 km、中轴长 10 km、群落管径 200 m、两环反向自转、24 个群落中 8 个名字、散热板/推进器的具体形态。这些在界面里都标注了。

**技术选型**：Three.js (WebGL) + 原生 ES Module，单文件零构建。理由：需要真 3D 网格、轨道相机、内外场景切换和几千个实例化植被——WebGL 是唯一合适的选择；不引入打包器是为了你能直接跑。

---

## 交付物：`aurora_ship.html`

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>《极光》世代飞船 · 交互式三维还原</title>
<style>
  :root{
    --bg:#05070c; --fg:#dfe8f2; --dim:#8b9bb0; --acc:#5fd0ff; --acc2:#ffb35f;
    --panel:rgba(8,14,24,.82); --line:rgba(95,208,255,.22);
  }
  *{box-sizing:border-box}
  html,body{margin:0;height:100%;background:var(--bg);color:var(--fg);
    font:13px/1.6 "Helvetica Neue",Arial,"PingFang SC","Microsoft YaHei",sans-serif;overflow:hidden}
  #app{position:fixed;inset:0}
  canvas{display:block}
  .ui{position:fixed;z-index:10}
  /* 顶部 */
  #top{top:0;left:0;right:0;display:flex;align-items:center;gap:14px;padding:10px 14px;
    background:linear-gradient(180deg,rgba(3,6,12,.9),rgba(3,6,12,0));pointer-events:none}
  #top h1{margin:0;font-size:15px;font-weight:600;letter-spacing:.5px}
  #top h1 small{display:block;font-size:11px;color:var(--dim);font-weight:400;letter-spacing:0}
  #modes{margin-left:auto;display:flex;gap:6px;pointer-events:auto}
  button,select{font:inherit;color:var(--fg);background:rgba(16,26,40,.85);border:1px solid var(--line);
    border-radius:6px;padding:6px 11px;cursor:pointer;transition:.15s}
  button:hover,select:hover{background:rgba(30,52,78,.95);border-color:var(--acc)}
  button.on{background:rgba(95,208,255,.18);border-color:var(--acc);color:#fff}
  /* 右侧信息面板 */
  #panel{top:64px;right:12px;width:330px;max-height:calc(100vh - 190px);overflow:auto;
    background:var(--panel);border:1px solid var(--line);border-radius:10px;padding:14px;
    backdrop-filter:blur(8px)}
  #panel h2{margin:0 0 8px;font-size:14px;color:var(--acc)}
  #panel h3{margin:14px 0 6px;font-size:12px;color:var(--acc2);letter-spacing:.4px}
  #panel p{margin:6px 0;color:#cbd8e6}
  #panel .kv{display:flex;justify-content:space-between;gap:10px;border-bottom:1px dotted rgba(255,255,255,.08);padding:3px 0}
  #panel .kv span:last-child{color:var(--dim);text-align:right}
  .tag{display:inline-block;font-size:10px;padding:1px 6px;border-radius:20px;margin-left:6px;
    border:1px solid var(--line);color:var(--dim)}
  .tag.canon{border-color:rgba(95,255,150,.4);color:#7fe6a5}
  .tag.guess{border-color:rgba(255,179,95,.45);color:var(--acc2)}
  /* 底部控制条 */
  #bar{bottom:0;left:0;right:0;display:flex;flex-wrap:wrap;align-items:center;gap:14px;
    padding:10px 14px;background:linear-gradient(0deg,rgba(3,6,12,.92),rgba(3,6,12,0))}
  #bar .grp{display:flex;align-items:center;gap:7px}
  #bar label{color:var(--dim);font-size:11px;white-space:nowrap}
  input[type=range]{width:96px;accent-color:var(--acc)}
  /* 标签 */
  #labels{position:fixed;inset:0;z-index:5;pointer-events:none}
  .lab{position:absolute;transform:translate(-50%,-50%);pointer-events:auto;cursor:pointer;
    background:rgba(6,12,22,.72);border:1px solid var(--line);border-left:2px solid var(--acc);
    padding:2px 8px;border-radius:4px;font-size:11px;white-space:nowrap;color:#e6f2ff;
    text-shadow:0 0 6px #000;transition:.15s}
  .lab:hover{background:rgba(95,208,255,.22);border-color:#fff}
  #tip{position:fixed;z-index:20;pointer-events:none;display:none;background:rgba(6,12,22,.92);
    border:1px solid var(--line);padding:4px 9px;border-radius:5px;font-size:11px}
  /* 遮罩 */
  #load{position:fixed;inset:0;z-index:50;display:flex;flex-direction:column;align-items:center;
    justify-content:center;gap:12px;background:#05070c;transition:opacity .6s}
  #load .ring{width:46px;height:46px;border:2px solid rgba(95,208,255,.2);border-top-color:var(--acc);
    border-radius:50%;animation:spin 1s linear infinite}
  @keyframes spin{to{transform:rotate(360deg)}}
  .modal{position:fixed;inset:0;z-index:40;display:none;align-items:center;justify-content:center;
    background:rgba(2,4,9,.78);padding:20px}
  .modal.open{display:flex}
  .modal .box{max-width:720px;max-height:80vh;overflow:auto;background:var(--panel);
    border:1px solid var(--line);border-radius:12px;padding:20px}
  .modal h2{margin-top:0;color:var(--acc)}
  .modal ul{padding-left:18px;color:#cbd8e6} .modal li{margin:5px 0}
  #hint{bottom:56px;left:14px;color:var(--dim);font-size:11px;max-width:44ch}
  @media (max-width:820px){#panel{display:none}}
</style>
<script type="importmap">
{"imports":{
  "three":"https://unpkg.com/three@0.160.0/build/three.module.js",
  "three/addons/":"https://unpkg.com/three@0.160.0/examples/jsm/"
}}
</script>
</head>
<body>
<div id="app"></div>
<div id="labels"></div>

<div class="ui" id="top">
  <h1>《极光》世代飞船<small>Aurora · Kim Stanley Robinson —— 双环 24 生物群落 · 中轴 10 km · 0.1c 巡航</small></h1>
  <div id="modes">
    <button id="btnExt" class="on">舰外视图</button>
    <button id="btnInt">进入内部</button>
    <button id="btnAbout">设定依据</button>
  </div>
</div>

<div class="ui" id="panel"></div>

<div class="ui" id="bar">
  <div class="grp"><label>环自转</label><input type="range" id="spin" min="0" max="20" step="0.5" value="6"><span id="spinTxt">×6</span></div>
  <div class="grp"><label>星尘流速</label><input type="range" id="flow" min="0" max="3" step="0.05" value="1"></div>
  <div class="grp"><label>昼夜速度</label><input type="range" id="day" min="0" max="4" step="0.1" value="1"></div>
  <div class="grp" id="grpExt">
    <button id="btnCut">剖视</button><button id="btnLab" class="on">标注</button><button id="btnDest" class="on">目的地</button>
  </div>
  <div class="grp" id="grpInt" style="display:none">
    <label>位置</label><select id="loc"></select>
    <button id="btnWalk">步行/漫游 (F)</button>
  </div>
  <div class="grp"><button id="btnReset">重置视角 (R)</button><button id="btnPerf">低画质</button></div>
</div>

<div class="ui" id="hint"></div>
<div id="tip"></div>

<div id="load"><div class="ring"></div><div>正在建造飞船…（24 个生物群落 / 数千棵植被实例）</div></div>

<div class="modal" id="about"><div class="box">
  <h2>还原度说明：哪些来自原著，哪些是重建</h2>
  <p style="color:#ffb35f">重要：本模型由 AI 依据对小说的既有认知重建，未在本次会话中检索原文。原著对多数尺寸只给了定性描述，凡涉及精确数字者请以书为准。</p>
  <h3 style="color:#7fe6a5">原著设定（高可信）</h3>
  <ul>
    <li>世代飞船由 <b>两个环</b> 组成，每环 <b>12 个生物群落</b>，共 <b>24 个</b>，彼此以气闸/锁相连。</li>
    <li>环围绕 <b>中轴（spine）</b> 布置，靠自转提供约 <b>1 g</b> 的"自旋重力"。</li>
    <li>群落以地球生物地理区命名；主角 Freya 出生在 Ring B 的 <b>Nova Scotia</b>。</li>
    <li>群落内部靠 <b>灯带照明并有昼夜循环</b>，是封闭生态，而非靠舷窗采光。</li>
    <li>船首有 <b>大型防护盾</b>，抵御 0.1c 下的星际尘埃与氢原子。</li>
    <li>中轴内含燃料、制造/打印车间、枢纽与 <b>渡船（ferry）坞</b>；渡船用于登陆。</li>
    <li>目的地：<b>鲸鱼座 τ（11.9 光年）</b>，行星 E 的卫星被命名为 <b>极光 Aurora</b>；航程约 170 年，人口约 2000。</li>
    <li>叙述者是 <b>飞船自身的人工智能</b>（量子计算机）。</li>
  </ul>
  <h3 style="color:#ffb35f">本模型的推测与取舍（原著未明确 / 记忆不确定）</h3>
  <ul>
    <li>具体尺寸：环中心半径 2 km、群落管径 200 m、中轴长 10 km、防护盾半径 2.3 km——按 1 g 自旋重力与 24 群落的可居住面积反推的自洽方案。</li>
    <li>两环 <b>反向自转</b>（抵消净角动量）——工程上合理，书中未见明说。</li>
    <li>散热板、减速磁线圈、推进喷口的数量与外形为工程化想象。</li>
    <li>24 个群落名中，标 <span class="tag guess">推测</span> 的 8 个是按"地球生物地理区"命名逻辑补齐的。</li>
    <li>群落地形、湖泊、城镇、植被密度为程序化生成，仅体现"1 km 长、200 m 径、有湖有林有农田有聚落"的量级。</li>
  </ul>
  <p style="text-align:right"><button onclick="document.getElementById('about').classList.remove('open')">关闭</button></p>
</div></div>

<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

/* ══════════════════════════════════════════════════════════════
   0. 单位与工具
   1 场景单位 = 10 米。飞船全长 10 km → 1000 单位。
   ══════════════════════════════════════════════════════════════ */
const U = 0.1, m = x => x * U;            // 米 → 单位
const clamp = (v,a,b)=>Math.min(b,Math.max(a,v));
const lerp  = (a,b,t)=>a+(b-a)*t;
const smooth= t=>t*t*(3-2*t);
const rnd   = (a,b)=>a+Math.random()*(b-a);

// —— 2D 值噪声（地形/植被密度）
function h2(i,j){const s=Math.sin(i*127.1+j*311.7)*43758.5453;return s-Math.floor(s);}
function vn2(x,y){
  const i=Math.floor(x),j=Math.floor(y),fx=x-i,fy=y-j;
  const u=fx*fx*(3-2*fx),v=fy*fy*(3-2*fy);
  const a=h2(i,j),b=h2(i+1,j),c=h2(i,j+1),d=h2(i+1,j+1);
  return (a*(1-u)+b*u)*(1-v)+(c*(1-u)+d*u)*v;
}
function fbm2(x,y,oct=4){let s=0,a=.5,f=1;for(let k=0;k<oct;k++){s+=a*vn2(x*f,y*f);f*=2.07;a*=.5;}return s;}

// —— 程序化贴图
function radialTex(size,stops){
  const c=document.createElement('canvas');c.width=c.height=size;
  const g=c.getContext('2d'),gr=g.createRadialGradient(size/2,size/2,0,size/2,size/2,size/2);
  stops.forEach(s=>gr.addColorStop(s[0],s[1]));
  g.fillStyle=gr;g.fillRect(0,0,size,size);
  const t=new THREE.CanvasTexture(c);t.colorSpace=THREE.SRGBColorSpace;return t;
}
const TEX_GLOW = radialTex(128,[[0,'rgba(255,255,255,1)'],[.22,'rgba(255,255,255,.6)'],[.55,'rgba(255,255,255,.12)'],[1,'rgba(255,255,255,0)']]);
const TEX_SOFT = radialTex(256,[[0,'rgba(255,255,255,.9)'],[.35,'rgba(255,255,255,.25)'],[1,'rgba(255,255,255,0)']]);

function textSprite(text,color='#bfe9ff'){
  const pad=16,f=44,c=document.createElement('canvas'),g=c.getContext('2d');
  g.font=`600 ${f}px "PingFang SC","Microsoft YaHei",sans-serif`;
  const w=Math.ceil(g.measureText(text).width)+pad*2;
  c.width=w;c.height=f+pad*2;
  const g2=c.getContext('2d');
  g2.fillStyle='rgba(4,10,20,.68)';g2.fillRect(0,0,c.width,c.height);
  g2.strokeStyle='rgba(95,208,255,.55)';g2.lineWidth=3;g2.strokeRect(1.5,1.5,c.width-3,c.height-3);
  g2.font=`600 ${f}px "PingFang SC","Microsoft YaHei",sans-serif`;
  g2.fillStyle=color;g2.textBaseline='middle';g2.fillText(text,pad,c.height/2+2);
  const t=new THREE.CanvasTexture(c);t.colorSpace=THREE.SRGBColorSpace;
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:t,transparent:true,depthWrite:false}));
  sp.scale.set(c.width/c.height*1.7,1.7,1);
  return sp;
}
function disposeTree(root){
  root.traverse(o=>{
    if(o.geometry) o.geometry.dispose();
    if(o.material){const a=Array.isArray(o.material)?o.material:[o.material];a.forEach(x=>x.dispose());}
  });
}

/* ══════════════════════════════════════════════════════════════
   1. 飞船主尺寸（★=推测值，见"设定依据"）
   ══════════════════════════════════════════════════════════════ */
const SHIP = {
  spineLen : m(9600),   // 中轴长 9.6 km ★
  spineR   : m(90),     // 中轴半径 90 m ★
  ringR    : m(2000),   // 环中心半径 2 km ★（1 g 时自转周期 ≈ 90 s）
  tubeR    : m(200),    // 生物群落管半径 200 m ★
  ringAz   : m(1800),   // Ring A 沿轴位置
  ringBz   : m(-1800),  // Ring B 沿轴位置
  shieldR  : m(2400),   // 防护盾半径 ★
  noseZ    : m(4800),
  tailZ    : m(-4800),
  biomeArc : Math.PI*2/12,
  gapRatio : 0.13       // 群落之间气闸占的弧长比例
};
const SPIN_OMEGA = Math.sqrt(9.8/2000);            // ≈0.07 rad/s → 1 g
const SPIN_PERIOD= 2*Math.PI/SPIN_OMEGA;           // ≈90 s

/* ══════════════════════════════════════════════════════════════
   2. 24 个生物群落
   canon:true = 原著中出现过的名字；guess:true = 按命名逻辑补齐 (推测)
   ══════════════════════════════════════════════════════════════ */
const PAL = {
  temperate:{sky:0x9fc4de,fog:0xb5cbdc,low:0x4a6b3a,mid:0x5c7d43,high:0x7f8f6a,sand:0xa79b78,water:0x2b5f78,tree:0x2f5d33},
  tropical :{sky:0xbcd9e8,fog:0xc2d9d4,low:0x2f5f34,mid:0x3d7a3a,high:0x6d8a54,sand:0xc2b183,water:0x2d7b7a,tree:0x1f6b32},
  boreal   :{sky:0xa9bccc,fog:0xc0ced8,low:0x3b4f38,mid:0x4a5f42,high:0x8d8f84,sand:0x9a9682,water:0x30536b,tree:0x24452f},
  steppe   :{sky:0xb9cbd8,fog:0xd2cfba,low:0x7d7c4a,mid:0x94904f,high:0xa89f6c,sand:0xb9ab7d,water:0x3d6f7a,tree:0x5d6b3a},
  desert   :{sky:0xd4d1bd,fog:0xdcd0b2,low:0xa9855a,mid:0xbf9a68,high:0xd0b183,sand:0xd8c191,water:0x4a7f86,tree:0x6f7a45},
  alpine   :{sky:0x9db6cc,fog:0xc6d3dc,low:0x5b6b4a,mid:0x7d8367,high:0xc9c9c4,sand:0x9c9484,water:0x3a6e86,tree:0x37543a},
  wetland  :{sky:0xb2cbd8,fog:0xc4d6d2,low:0x466b3f,mid:0x5a8046,high:0x7c8b5e,sand:0xa89c74,water:0x35707a,tree:0x2c6238},
};
// ring, 名称, 中文, 气候, 调色板, 地形起伏(单位=10m), 水位比, 树型, 数量
const BIOMES = [
  {ring:'B',en:'Nova Scotia',zh:'新斯科舍',clim:'温带海岸混交林',pal:'temperate',amp:1.6,water:.38,tree:'conifer',count:900,canon:true,note:'主角 Freya 的家乡群落'},
  {ring:'B',en:'Costa Rica',zh:'哥斯达黎加',clim:'热带雨林',pal:'tropical',amp:1.2,water:.30,tree:'palm',count:1300,canon:true},
  {ring:'B',en:'Amazonia',zh:'亚马逊',clim:'热带雨林与河道',pal:'tropical',amp:0.9,water:.46,tree:'broad',count:1500,canon:true},
  {ring:'B',en:'Patagonia',zh:'巴塔哥尼亚',clim:'寒温带草原',pal:'alpine',amp:2.2,water:.24,tree:'shrub',count:420,canon:true},
  {ring:'B',en:'Labrador',zh:'拉布拉多',clim:'北方针叶林/苔原',pal:'boreal',amp:1.8,water:.34,tree:'conifer',count:750,canon:true},
  {ring:'B',en:'The Steppes',zh:'大草原',clim:'温带干草原',pal:'steppe',amp:1.0,water:.20,tree:'shrub',count:260,canon:true},
  {ring:'B',en:'Olympia',zh:'奥林匹亚',clim:'温带雨林',pal:'temperate',amp:2.0,water:.32,tree:'conifer',count:1200,canon:true},
  {ring:'B',en:'Tasmania',zh:'塔斯马尼亚',clim:'温带雨林丘陵',pal:'temperate',amp:1.9,water:.34,tree:'broad',count:1000,canon:true},
  {ring:'B',en:'Bengal',zh:'孟加拉',clim:'季风三角洲',pal:'wetland',amp:0.7,water:.50,tree:'palm',count:900,canon:true},
  {ring:'B',en:'Sonora',zh:'索诺拉',clim:'荒漠',pal:'desert',amp:1.5,water:.10,tree:'shrub',count:220,canon:true},
  {ring:'B',en:'Kilimanjaro',zh:'乞力马扎罗',clim:'高山草甸',pal:'alpine',amp:2.6,water:.18,tree:'shrub',count:320,canon:true},
  {ring:'B',en:'Yangtze',zh:'长江',clim:'农业水田',pal:'wetland',amp:0.8,water:.42,tree:'broad',count:600,canon:true},
  {ring:'A',en:'Iran',zh:'伊朗',clim:'干旱高原',pal:'desert',amp:2.0,water:.14,tree:'shrub',count:260,canon:true},
  {ring:'A',en:'Mongolia',zh:'蒙古',clim:'干草原',pal:'steppe',amp:1.3,water:.18,tree:'shrub',count:220,canon:true},
  {ring:'A',en:'Siberia',zh:'西伯利亚',clim:'泰加林',pal:'boreal',amp:1.5,water:.30,tree:'conifer',count:1300,canon:true},
  {ring:'A',en:'Pampas',zh:'潘帕斯',clim:'湿润草原',pal:'steppe',amp:0.8,water:.28,tree:'shrub',count:300,canon:true},
  {ring:'A',en:'Yukon',zh:'育空',clim:'北方河谷',pal:'boreal',amp:2.1,water:.33,tree:'conifer',count:800,guess:true},
  {ring:'A',en:'Sahel',zh:'萨赫勒',clim:'半干旱疏林',pal:'steppe',amp:1.1,water:.16,tree:'shrub',count:340,guess:true},
  {ring:'A',en:'Namib',zh:'纳米布',clim:'极旱荒漠',pal:'desert',amp:1.7,water:.06,tree:'none',count:60,guess:true},
  {ring:'A',en:'Borneo',zh:'婆罗洲',clim:'热带雨林',pal:'tropical',amp:1.3,water:.40,tree:'palm',count:1400,guess:true},
  {ring:'A',en:'Altiplano',zh:'阿尔蒂普拉诺',clim:'高原湖沼',pal:'alpine',amp:2.4,water:.30,tree:'shrub',count:200,guess:true},
  {ring:'A',en:'Kerguelen',zh:'凯尔盖朗',clim:'亚南极草甸',pal:'boreal',amp:1.6,water:.30,tree:'shrub',count:250,guess:true},
  {ring:'A',en:'Ceylon',zh:'锡兰',clim:'热带季风',pal:'tropical',amp:1.4,water:.38,tree:'palm',count:1100,guess:true},
  {ring:'A',en:'Ushuaia',zh:'乌斯怀亚',clim:'亚极地森林',pal:'boreal',amp:2.3,water:.30,tree:'conifer',count:700,guess:true},
];
BIOMES.forEach((b,i)=>{b.idx=i;b.seed=i*7.31+1.7;});

/* 部件说明 */
const PARTS = {
  shield:{t:'前置防护盾 Shield',canon:true,d:'以 0.1c 巡航时，星际尘埃与氢原子构成持续的撞击与辐射流。船首这面直径数公里的盾负责烧蚀与屏蔽，是全船最脆弱也最关键的部件。'},
  spine:{t:'中轴 Spine',canon:true,d:'贯穿全船的主承力结构，长约 10 km。内部有零重力走廊、燃料、制造/打印车间、仓储，以及叙述本书的飞船人工智能。'},
  ringA:{t:'Ring A（12 个生物群落）',canon:true,d:'12 段封闭生态圆筒首尾相接成环，以气闸相连。每段是独立的地球生物区复制品：土壤、水体、植被、动物与人类聚落。'},
  ringB:{t:'Ring B（12 个生物群落）',canon:true,d:'与 Ring A 对称。Freya 的家乡 Nova Scotia 位于此环。24 个群落合计承载约 2000 人与全部封闭生态循环。'},
  spoke:{t:'辐条与枢纽 Spokes & Hub',canon:true,d:'连接环与中轴的通道，内有电梯。从 1 g 的环走向零重力的中轴，重力沿途递减——书中反复描写的过渡体验。'},
  tank:{t:'燃料舱 Fuel Tanks',canon:false,d:'减速与机动所需的推进剂。到达前的减速消耗了大量储备，成为剧情的硬约束。（形态为推测）'},
  rad:{t:'散热板 Radiators',canon:false,d:'封闭生态与聚变系统的废热必须辐射进太空，否则船内温度失控。（形态为推测）'},
  engine:{t:'推进与减速阵列',canon:false,d:'巡航段几乎不点火；抵达前的长期减速与返航段的加速是全船能量预算的核心。（形态为推测）'},
  dock:{t:'渡船坞 Ferry Dock',canon:true,d:'搭载登陆渡船。抵达鲸鱼座 τ 后，渡船把探索队送往行星 E 的卫星"极光"。'},
  dish:{t:'通信天线',canon:false,d:'指向太阳系的深空通信阵列。往返一次消息要走十几年，飞船长期处于"独自决策"状态。（形态为推测）'},
};

/* ══════════════════════════════════════════════════════════════
   3. 渲染器 / 场景框架
   ══════════════════════════════════════════════════════════════ */
const host = document.getElementById('app');
const renderer = new THREE.WebGLRenderer({antialias:true,logarithmicDepthBuffer:true,powerPreference:'high-performance'});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setSize(innerWidth,innerHeight);
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.05;
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
host.appendChild(renderer.domElement);

const extScene = new THREE.Scene();
const intScene = new THREE.Scene();
const extCam = new THREE.PerspectiveCamera(52,innerWidth/innerHeight,.5,300000);
const intCam = new THREE.PerspectiveCamera(62,innerWidth/innerHeight,.03,4000);
extCam.position.set(m(5200),m(2400),m(7200));
const extCtl = new OrbitControls(extCam,renderer.domElement);
extCtl.enableDamping=true; extCtl.dampingFactor=.07;
extCtl.minDistance=m(300); extCtl.maxDistance=m(80000);
extCtl.rotateSpeed=.75; extCtl.zoomSpeed=.9; extCtl.panSpeed=.6;
const intCtl = new OrbitControls(intCam,renderer.domElement);
intCtl.enableDamping=true; intCtl.dampingFactor=.08;
intCtl.minDistance=m(3); intCtl.maxDistance=m(1400);
intCtl.enabled=false;

let mode='exterior', walkMode=false, perfLow=false;
let spinMul=6, flowMul=1, dayMul=1, cutaway=false, labelsOn=true, destOn=true;

/* ══════════════════════════════════════════════════════════════
   4. 深空背景：星云 + 星场 + 星尘拉丝 + 目的地
   ══════════════════════════════════════════════════════════════ */
const NEB_VS=`varying vec3 vD; void main(){vD=normalize(position);gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.);}`;
const NEB_FS=`
varying vec3 vD;
float h(vec3 p){return fract(sin(dot(p,vec3(127.1,311.7,74.7)))*43758.5453);}
float n3(vec3 p){vec3 i=floor(p),f=fract(p);f=f*f*(3.-2.*f);
 return mix(mix(mix(h(i),h(i+vec3(1,0,0)),f.x),mix(h(i+vec3(0,1,0)),h(i+vec3(1,1,0)),f.x),f.y),
            mix(mix(h(i+vec3(0,0,1)),h(i+vec3(1,0,1)),f.x),mix(h(i+vec3(0,1,1)),h(i+vec3(1,1,1)),f.x),f.y),f.z);}
float fbm(vec3 p){float s=0.,a=.5;for(int k=0;k<5;k++){s+=a*n3(p);p*=2.03;a*=.5;}return s;}
void main(){
  vec3 d=vD;
  float band=pow(max(0.,1.-abs(d.y)*1.35),2.2);          // 银河带
  float f=fbm(d*3.1), g=fbm(d*7.3+13.0);
  vec3 c1=vec3(.035,.055,.115), c2=vec3(.10,.055,.15), c3=vec3(.02,.09,.11);
  vec3 col=mix(c1,c2,smoothstep(.35,.85,f));
  col=mix(col,c3,smoothstep(.45,.9,g)*.6);
  col*= .35+1.25*band;
  col+=vec3(.012,.016,.03);
  gl_FragColor=vec4(col,1.);
}`;
extScene.add(new THREE.Mesh(
  new THREE.SphereGeometry(140000,48,32),
  new THREE.ShaderMaterial({vertexShader:NEB_VS,fragmentShader:NEB_FS,side:THREE.BackSide,depthWrite:false})
));

function makeStars(n,radius,size,bright){
  const pos=new Float32Array(n*3),col=new Float32Array(n*3),c=new THREE.Color();
  for(let i=0;i<n;i++){
    const u=Math.random()*2-1,th=Math.random()*Math.PI*2,s=Math.sqrt(1-u*u);
    const r=radius*rnd(.8,1);
    pos[i*3]=r*s*Math.cos(th); pos[i*3+1]=r*u*rnd(.55,1); pos[i*3+2]=r*s*Math.sin(th);
    const k=Math.random();
    c.setHSL(k<.55?.58:(k<.8?.09:.02), rnd(.05,.5), rnd(.6,1)*bright);
    col[i*3]=c.r;col[i*3+1]=c.g;col[i*3+2]=c.b;
  }
  const g=new THREE.BufferGeometry();
  g.setAttribute('position',new THREE.BufferAttribute(pos,3));
  g.setAttribute('color',new THREE.BufferAttribute(col,3));
  return new THREE.Points(g,new THREE.PointsMaterial({size,map:TEX_GLOW,vertexColors:true,
    transparent:true,blending:THREE.AdditiveBlending,depthWrite:false,sizeAttenuation:true}));
}
extScene.add(makeStars(16000,90000,220,.85));
extScene.add(makeStars(1400,90000,900,1));

// 星际尘埃拉丝（体现"正在深空航行"，前进方向 = +Z）
const DUST_N=2200;
const dustGeo=new THREE.BufferGeometry();
const dustPos=new Float32Array(DUST_N*6), dustSeed=new Float32Array(DUST_N*3);
for(let i=0;i<DUST_N;i++){
  const a=Math.random()*Math.PI*2, r=m(400)+Math.pow(Math.random(),.6)*m(9000);
  dustSeed[i*3]=Math.cos(a)*r; dustSeed[i*3+1]=Math.sin(a)*r;
  dustSeed[i*3+2]=rnd(-m(30000),m(30000));
}
dustGeo.setAttribute('position',new THREE.BufferAttribute(dustPos,3));
const dust=new THREE.LineSegments(dustGeo,new THREE.LineBasicMaterial({
  color:0xbfe4ff,transparent:true,opacity:.55,blending:THREE.AdditiveBlending,depthWrite:false}));
extScene.add(dust);
function updateDust(dt){
  const speed=m(26000)*flowMul, len=m(260)+m(2600)*flowMul;
  for(let i=0;i<DUST_N;i++){
    let z=dustSeed[i*3+2]-speed*dt;
    if(z< -m(30000)) z+=m(60000);
    dustSeed[i*3+2]=z;
    const x=dustSeed[i*3],y=dustSeed[i*3+1];
    dustPos[i*6]=x;dustPos[i*6+1]=y;dustPos[i*6+2]=z;
    dustPos[i*6+3]=x;dustPos[i*6+4]=y;dustPos[i*6+5]=z+len;
  }
  dustGeo.attributes.position.needsUpdate=true;
  dust.material.opacity=.18+.5*Math.min(1,flowMul);
}

// 鲸鱼座 τ（K 型橙矮星，位于前方）+ 光照
const starDir=new THREE.Vector3(0.14,0.10,1).normalize();
const sunSprite=new THREE.Sprite(new THREE.SpriteMaterial({map:TEX_SOFT,color:0xffd9a0,
  transparent:true,blending:THREE.AdditiveBlending,depthWrite:false}));
sunSprite.position.copy(starDir).multiplyScalar(110000); sunSprite.scale.setScalar(9000);
extScene.add(sunSprite);
const sunLight=new THREE.DirectionalLight(0xffe3bd,2.6);
sunLight.position.copy(starDir).multiplyScalar(1000); extScene.add(sunLight);
extScene.add(new THREE.AmbientLight(0x223349,.55));
const backLight=new THREE.DirectionalLight(0x4f7bd4,.5); backLight.position.set(-1,-.4,-1); extScene.add(backLight);

// 目的地：行星 E 与其卫星"极光"
const destGroup=new THREE.Group();
{
  const p=new THREE.Mesh(new THREE.SphereGeometry(4200,48,32),
    new THREE.MeshStandardMaterial({color:0x6f8fb5,roughness:.95,metalness:0}));
  p.position.set(9000,2600,72000);
  const moon=new THREE.Mesh(new THREE.SphereGeometry(1150,32,24),
    new THREE.MeshStandardMaterial({color:0x9fb6c2,roughness:.9}));
  moon.position.set(15200,4200,66000);
  const halo=new THREE.Sprite(new THREE.SpriteMaterial({map:TEX_SOFT,color:0x8fd8ff,
    transparent:true,opacity:.35,blending:THREE.AdditiveBlending,depthWrite:false}));
  halo.position.copy(moon.position); halo.scale.setScalar(5200);
  destGroup.add(p,moon,halo);
  destGroup.userData.moon=moon;
}
extScene.add(destGroup);

/* ══════════════════════════════════════════════════════════════
   5. 建造飞船（外部）
   ══════════════════════════════════════════════════════════════ */
const ship=new THREE.Group(); extScene.add(ship);
const MAT={
  hull  : new THREE.MeshStandardMaterial({color:0xa8b0ba,metalness:.78,roughness:.40}),
  hull2 : new THREE.MeshStandardMaterial({color:0x6d747d,metalness:.65,roughness:.55}),
  dark  : new THREE.MeshStandardMaterial({color:0x3a3f47,metalness:.55,roughness:.7}),
  gold  : new THREE.MeshStandardMaterial({color:0xc9a15a,metalness:.9,roughness:.35}),
  shield: new THREE.MeshStandardMaterial({color:0xcfd8e0,metalness:.35,roughness:.85}),
  rad   : new THREE.MeshStandardMaterial({color:0x2b2f36,metalness:.4,roughness:.6,
            emissive:0x7a1f10,emissiveIntensity:.6,side:THREE.DoubleSide}),
};
function addGlow(parent,pos,scale,color,opacity=.9){
  const s=new THREE.Sprite(new THREE.SpriteMaterial({map:TEX_GLOW,color,transparent:true,
    opacity,blending:THREE.AdditiveBlending,depthWrite:false}));
  s.position.copy(pos); s.scale.setScalar(scale); parent.add(s); return s;
}
function tagPart(obj,key,extra){obj.traverse(o=>{o.userData.part=key;if(extra)Object.assign(o.userData,extra);});}

/* —— 5.1 中轴 —— */
const spine=new THREE.Group(); ship.add(spine);
{
  const core=new THREE.Mesh(new THREE.CylinderGeometry(SHIP.spineR,SHIP.spineR,SHIP.spineLen,32,1),MAT.hull);
  core.rotation.x=Math.PI/2; spine.add(core);
  // 加强环与桁架
  for(let i=0;i<=24;i++){
    const z=-SHIP.spineLen/2+i*(SHIP.spineLen/24);
    const rib=new THREE.Mesh(new THREE.TorusGeometry(SHIP.spineR*1.09,m(9),6,28),MAT.hull2);
    rib.position.z=z; spine.add(rib);
  }
  for(let k=0;k<4;k++){
    const a=k*Math.PI/2+Math.PI/4;
    const beam=new THREE.Mesh(new THREE.BoxGeometry(m(24),m(24),SHIP.spineLen*.96),MAT.hull2);
    beam.position.set(Math.cos(a)*SHIP.spineR*1.05,Math.sin(a)*SHIP.spineR*1.05,0); spine.add(beam);
  }
  // 中轴照明带
  for(let k=0;k<3;k++){
    const a=k*Math.PI*2/3;
    const strip=new THREE.Mesh(new THREE.BoxGeometry(m(10),m(4),SHIP.spineLen*.9),
      new THREE.MeshStandardMaterial({color:0x9fd8ff,emissive:0x54b6ff,emissiveIntensity:1.6}));
    strip.position.set(Math.cos(a)*SHIP.spineR*1.13,Math.sin(a)*SHIP.spineR*1.13,0); spine.add(strip);
  }
  tagPart(spine,'spine');
}

/* —— 5.2 前置防护盾 —— */
const shield=new THREE.Group(); ship.add(shield);
{
  const pts=[];
  for(let i=0;i<=16;i++){const t=i/16; pts.push(new THREE.Vector2(SHIP.shieldR*t, -m(420)*t*t));}
  const dish=new THREE.Mesh(new THREE.LatheGeometry(pts,64),MAT.shield);
  dish.rotation.x=-Math.PI/2; dish.position.z=SHIP.noseZ; shield.add(dish);
  const back=new THREE.Mesh(new THREE.CylinderGeometry(SHIP.shieldR,SHIP.shieldR*.8,m(260),64,1,true),MAT.hull2);
  back.rotation.x=Math.PI/2; back.position.z=SHIP.noseZ-m(200); shield.add(back);
  const neck=new THREE.Mesh(new THREE.CylinderGeometry(SHIP.spineR*1.4,SHIP.shieldR*.8,m(900),32),MAT.hull);
  neck.rotation.x=Math.PI/2; neck.position.z=SHIP.noseZ-m(760); shield.add(neck);
  // 烧蚀弓激波
  const shock=new THREE.Mesh(new THREE.RingGeometry(SHIP.shieldR*.55,SHIP.shieldR*1.16,96),
    new THREE.MeshBasicMaterial({color:0x8fd0ff,transparent:true,opacity:.28,
      blending:THREE.AdditiveBlending,depthWrite:false,side:THREE.DoubleSide}));
  shock.position.z=SHIP.noseZ+m(60); shield.add(shock);
  shield.userData.shock=shock;
  shield.userData.glow=addGlow(shield,new THREE.Vector3(0,0,SHIP.noseZ+m(150)),SHIP.shieldR*2.4,0x9ad4ff,.30);
  tagPart(shield,'shield');
}

/* —— 5.3 环（12 群落 + 气闸 + 舷窗 + 剖视生态层） —— */
function buildRing(ringId){
  const g=new THREE.Group();
  const R=SHIP.ringR, r=SHIP.tubeR, arc=SHIP.biomeArc, gap=arc*SHIP.gapRatio;
  const list=BIOMES.filter(b=>b.ring===ringId);
  const cutMats=[];
  list.forEach((b,i)=>{
    const seg=new THREE.Group(); seg.rotation.z=i*arc+gap/2;
    // 群落外壳
    const shellMat=MAT.hull.clone();
    const shell=new THREE.Mesh(new THREE.TorusGeometry(R,r,20,44,arc-gap),shellMat);
    shell.userData={part:ringId==='A'?'ringA':'ringB',biomeIndex:b.idx};
    seg.add(shell);
    // 剖视时可见的内部生态层
    const inner=new THREE.Mesh(new THREE.TorusGeometry(R,r*.82,16,40,arc-gap),
      new THREE.MeshBasicMaterial({color:new THREE.Color(PAL[b.pal].mid).lerp(new THREE.Color(0xffffff),.15),
        transparent:true,opacity:0,depthWrite:false}));
    seg.add(inner); cutMats.push({shellMat,inner});
    // 群落外壁纵向加强肋
    for(let k=1;k<6;k++){
      const u=(arc-gap)*k/6;
      const rib=new THREE.Mesh(new THREE.TorusGeometry(r*1.02,m(16),5,20),MAT.hull2);
      rib.position.set(Math.cos(u)*R,Math.sin(u)*R,0);
      rib.rotation.set(Math.PI/2,0,0); rib.rotateY(u);
      seg.add(rib);
    }
    // 舷窗/灯带（原著中群落靠内部灯带照明，这些是维护窗与外部航行灯）
    const NW=26, win=new THREE.InstancedMesh(new THREE.BoxGeometry(m(26),m(5),m(11)),
      new THREE.MeshStandardMaterial({color:0xbfe8ff,emissive:0x63c8ff,emissiveIntensity:2.2}),NW*3);
    const d=new THREE.Object3D(); let n=0;
    for(let k=0;k<N