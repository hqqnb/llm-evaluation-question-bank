我无法在这个环境里直接写文件（当前工作目录是 `/`，且没有可用的写入工具通道），所以我把完整可运行的单文件成品放在下面。保存为 `globe.html` 即可运行。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
<title>3D 数据可视化地球仪 · Three.js</title>
<style>
  :root{
    --bg:#05070f; --fg:#e8f0ff; --dim:#8fa2c4; --line:rgba(140,180,255,.18);
    --glass:rgba(10,18,36,.72); --accent:#4fd1ff;
  }
  *{box-sizing:border-box}
  html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);
    font:13px/1.5 "Segoe UI",system-ui,-apple-system,"PingFang SC","Microsoft YaHei",sans-serif;overflow:hidden}
  #stage{position:fixed;inset:0}
  canvas{display:block}

  .glass{background:var(--glass);border:1px solid var(--line);border-radius:12px;
    backdrop-filter:blur(10px);-webkit-backdrop-filter:blur(10px);box-shadow:0 10px 40px rgba(0,0,0,.45)}

  /* ---- 控制面板 ---- */
  #panel{position:fixed;top:14px;right:14px;width:288px;padding:14px 14px 12px;z-index:10;
    max-height:calc(100vh - 28px);overflow:auto}
  #panel h1{margin:0 0 2px;font-size:15px;letter-spacing:.5px}
  #panel .sub{color:var(--dim);font-size:11px;margin-bottom:12px}
  .grp{border-top:1px solid var(--line);padding:10px 0 2px;margin-top:6px}
  .grp:first-of-type{border-top:0;margin-top:0}
  .grp>.t{font-size:11px;color:var(--dim);letter-spacing:1px;margin-bottom:8px;text-transform:uppercase}
  .row{display:flex;align-items:center;justify-content:space-between;gap:8px;margin:7px 0}
  .row>span{color:var(--dim)}
  select,button,input[type=range]{font:inherit;color:var(--fg)}
  select,button{background:rgba(255,255,255,.06);border:1px solid var(--line);border-radius:7px;padding:5px 8px;cursor:pointer}
  select:hover,button:hover{background:rgba(79,209,255,.14);border-color:rgba(79,209,255,.5)}
  input[type=range]{-webkit-appearance:none;appearance:none;height:4px;background:rgba(255,255,255,.16);
    border-radius:3px;outline:none;cursor:pointer}
  input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:13px;height:13px;border-radius:50%;
    background:var(--accent);box-shadow:0 0 8px rgba(79,209,255,.9);cursor:pointer}
  input[type=range]::-moz-range-thumb{width:13px;height:13px;border:0;border-radius:50%;background:var(--accent)}
  .chk{display:flex;align-items:center;gap:6px;font-size:12px;color:#cfe0ff;cursor:pointer;user-select:none}
  .chk input{accent-color:var(--accent);cursor:pointer}
  .chks{display:grid;grid-template-columns:1fr 1fr;gap:6px 10px}

  #legend .bar{height:9px;border-radius:5px;margin:6px 0 4px;
    background:linear-gradient(90deg,#102a60,#0084b2,#10c48c,#f0c63c,#fa543c)}
  #legend .lab{display:flex;justify-content:space-between;color:var(--dim);font-size:11px}
  #top5{font-size:12px}
  #top5 .it{display:flex;justify-content:space-between;gap:8px;padding:2px 0;border-bottom:1px dashed rgba(255,255,255,.07)}
  #top5 .it b{font-weight:600}
  #top5 .it i{font-style:normal;color:var(--accent)}
  #sunInfo{color:var(--dim);font-size:11px;margin-top:8px;line-height:1.7}

  /* ---- 时间轴 ---- */
  #timeline{position:fixed;left:50%;transform:translateX(-50%);bottom:16px;z-index:10;
    display:flex;align-items:center;gap:12px;padding:10px 16px}
  #playBtn{width:34px;height:34px;border-radius:50%;display:grid;place-items:center;font-size:13px}
  #yearRange{width:min(46vw,460px)}
  #yearLabel{font-variant-numeric:tabular-nums;font-size:16px;font-weight:600;min-width:52px;text-align:center}
  #ticks{display:flex;justify-content:space-between;color:var(--dim);font-size:10px;width:min(46vw,460px)}

  /* ---- 标题 / 提示 ---- */
  #brand{position:fixed;left:16px;top:14px;z-index:10;padding:10px 14px;font-size:12px;color:var(--dim)}
  #brand b{display:block;font-size:14px;color:var(--fg);letter-spacing:.5px;margin-bottom:3px}
  #status{margin-top:6px;font-size:11px;color:#7f93b8}

  /* ---- 城市信息弹窗 ---- */
  #card{position:fixed;left:16px;bottom:16px;width:272px;padding:14px;z-index:12;display:none}
  #card.on{display:block;animation:pop .18s ease-out}
  @keyframes pop{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
  #card .hd{display:flex;justify-content:space-between;align-items:flex-start;gap:8px}
  #card .nm{font-size:16px;font-weight:600}
  #card .en{color:var(--dim);font-size:11px}
  #card .x{cursor:pointer;color:var(--dim);border:0;background:none;font-size:16px;line-height:1;padding:0 2px}
  #card table{width:100%;border-collapse:collapse;margin-top:10px;font-size:12px}
  #card td{padding:4px 0;border-bottom:1px dashed rgba(255,255,255,.08)}
  #card td:last-child{text-align:right;font-variant-numeric:tabular-nums;color:var(--accent)}
  #card .tag{display:inline-block;margin-top:9px;padding:2px 7px;border-radius:20px;font-size:11px;
    background:rgba(79,209,255,.14);border:1px solid rgba(79,209,255,.35)}

  /* ---- 航线 tooltip ---- */
  #tip{position:fixed;z-index:20;pointer-events:none;display:none;padding:8px 11px;max-width:250px;
    font-size:12px;line-height:1.6;border-radius:9px;background:rgba(6,12,26,.92);
    border:1px solid rgba(79,209,255,.35);box-shadow:0 8px 26px rgba(0,0,0,.5)}
  #tip .r1{font-weight:600;margin-bottom:2px}
  #tip .d{color:var(--dim)}

  /* ---- loading ---- */
  #loading{position:fixed;inset:0;z-index:50;display:grid;place-items:center;background:#05070f;
    transition:opacity .5s;text-align:center}
  #loading.off{opacity:0;pointer-events:none}
  .spin{width:38px;height:38px;margin:0 auto 14px;border:2px solid rgba(255,255,255,.14);
    border-top-color:var(--accent);border-radius:50%;animation:sp 1s linear infinite}
  @keyframes sp{to{transform:rotate(360deg)}}
  #loading small{color:var(--dim)}
  @media (max-width:820px){
    #panel{width:250px} #brand{max-width:200px} #card{width:230px}
  }
</style>
</head>
<body>
<div id="stage"></div>

<div id="brand" class="glass">
  <b>3D 数据可视化地球仪</b>
  Three.js · WebGL · ESM importmap
  <div id="status">正在加载资源…</div>
</div>

<div id="panel" class="glass">
  <h1>控制面板</h1>
  <div class="sub">拖拽旋转 · 滚轮缩放 · 点击城市 · 悬停航线</div>

  <div class="grp">
    <div class="t">数据指标</div>
    <div class="row">
      <select id="metricSel" style="flex:1">
        <option value="population">人口 Population</option>
        <option value="gdp" selected>经济总量 GDP</option>
        <option value="flights">航班量 Flights</option>
      </select>
    </div>
    <div id="legend">
      <div class="bar"></div>
      <div class="lab"><span id="lgMin">低</span><span id="lgMax">高</span></div>
    </div>
  </div>

  <div class="grp">
    <div class="t">图层</div>
    <div class="chks">
      <label class="chk"><input type="checkbox" id="ckSpin" checked>自转</label>
      <label class="chk"><input type="checkbox" id="ckClouds" checked>云层</label>
      <label class="chk"><input type="checkbox" id="ckAtmo" checked>大气光晕</label>
      <label class="chk"><input type="checkbox" id="ckArcs" checked>航线</label>
      <label class="chk"><input type="checkbox" id="ckBars" checked>数据光柱</label>
      <label class="chk"><input type="checkbox" id="ckHeat" checked>热力着色</label>
      <label class="chk"><input type="checkbox" id="ckNight" checked>城市夜灯</label>
      <label class="chk"><input type="checkbox" id="ckStars" checked>星空</label>
    </div>
  </div>

  <div class="grp">
    <div class="t">参数</div>
    <div class="row"><span>自转速度</span><input type="range" id="rgSpin" min="0" max="100" value="26" style="width:130px"></div>
    <div class="row"><span>热力强度</span><input type="range" id="rgHeat" min="0" max="100" value="72" style="width:130px"></div>
    <div class="row"><span>夜灯亮度</span><input type="range" id="rgNight" min="0" max="200" value="115" style="width:130px"></div>
    <div class="row"><span>视角</span>
      <span style="display:flex;gap:6px">
        <button id="btnSun" title="把日夜分界线对齐到当前真实 UTC 时间">对齐真实时间</button>
        <button id="btnReset" title="重置相机">重置</button>
      </span>
    </div>
    <div id="sunInfo"></div>
  </div>

  <div class="grp">
    <div class="t">排行 TOP 5</div>
    <div id="top5"></div>
  </div>
</div>

<div id="timeline" class="glass">
  <button id="playBtn" title="播放 / 暂停（空格）">▶</button>
  <div style="display:flex;flex-direction:column;gap:2px">
    <input type="range" id="yearRange" min="2000" max="2025" step="1" value="2025">
    <div id="ticks"><span>2000</span><span>2005</span><span>2010</span><span>2015</span><span>2020</span><span>2025</span></div>
  </div>
  <span id="yearLabel">2025</span>
  <select id="speedSel" title="播放速度">
    <option value="0.5">0.5×</option>
    <option value="1" selected>1×</option>
    <option value="2">2×</option>
    <option value="4">4×</option>
  </select>
</div>

<div id="card" class="glass">
  <div class="hd">
    <div>
      <div class="nm" id="cdName">—</div>
      <div class="en" id="cdEn">—</div>
    </div>
    <button class="x" id="cdClose">✕</button>
  </div>
  <table>
    <tr><td>国家 / 地区</td><td id="cdCountry">—</td></tr>
    <tr><td>坐标</td><td id="cdGeo">—</td></tr>
    <tr><td>人口</td><td id="cdPop">—</td></tr>
    <tr><td>GDP</td><td id="cdGdp">—</td></tr>
    <tr><td>周航班量</td><td id="cdFlt">—</td></tr>
    <tr><td>通航航线</td><td id="cdRoutes">—</td></tr>
    <tr><td>当地太阳时</td><td id="cdSolar">—</td></tr>
  </table>
  <span class="tag" id="cdYear">2025</span>
</div>

<div id="tip"></div>

<div id="loading">
  <div>
    <div class="spin"></div>
    <div>正在构建地球…</div>
    <small id="ldMsg">加载纹理与地理边界数据</small>
  </div>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
    "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

/* =========================================================================
   0. 全局约定
   - 经纬度 -> 世界坐标：lon=0 对准 +X，纬度北为 +Y，与等距圆柱纹理 u=(lon+180)/360 对齐
   - 颜色管线走「直通」模式（关闭 ColorManagement + 线性输出），保证自定义着色器可控
   ========================================================================= */
THREE.ColorManagement.enabled = false;

const DEG = Math.PI / 180;
const R = 1;
const YEAR_MIN = 2000, YEAR_MAX = 2025, YEAR_BASE = 2024;

const $ = s => document.querySelector(s);
const clamp = (v, a, b) => Math.min(b, Math.max(a, v));
const lerp = (a, b, t) => a + (b - a) * t;

/** 稳定字符串哈希 -> [0,1)，用于生成可复现的演示数据 */
function hash(str) {
  let h = 2166136261;
  for (let i = 0; i < str.length; i++) { h ^= str.charCodeAt(i); h = Math.imul(h, 16777619); }
  return (h >>> 0) / 4294967296;
}

function latLonToVec3(lat, lon, r = 1, out = new THREE.Vector3()) {
  const la = lat * DEG, lo = lon * DEG, c = Math.cos(la);
  return out.set(r * c * Math.cos(lo), r * Math.sin(la), -r * c * Math.sin(lo));
}

/* 色带（低 -> 高） */
const STOPS = [
  [0.00, [16, 42, 96]], [0.25, [0, 132, 178]], [0.50, [16, 196, 140]],
  [0.75, [240, 198, 60]], [1.00, [250, 84, 60]]
];
function ramp(t) {
  t = clamp(t, 0, 1);
  for (let i = 1; i < STOPS.length; i++) {
    if (t <= STOPS[i][0]) {
      const [t0, c0] = STOPS[i - 1], [t1, c1] = STOPS[i], k = (t - t0) / (t1 - t0);
      return [lerp(c0[0], c1[0], k), lerp(c0[1], c1[1], k), lerp(c0[2], c1[2], k)];
    }
  }
  return STOPS[STOPS.length - 1][1];
}
const rampCss = t => { const c = ramp(t); return `rgb(${c[0] | 0},${c[1] | 0},${c[2] | 0})`; };

/* =========================================================================
   1. 演示数据（数值为示意量级，非权威统计）
   ========================================================================= */
const METRICS = {
  population: { label: '人口', short: '人口', fmt: v => v.toFixed(1) + ' 百万' },
  gdp:        { label: '经济总量 GDP', short: 'GDP', fmt: v => v >= 1000 ? (v / 1000).toFixed(2) + ' 万亿美元' : v.toFixed(0) + ' 十亿美元' },
  flights:    { label: '航班量', short: '航班', fmt: v => v.toFixed(0) + ' 班/周' }
};

/* 城市：code / 中文 / 英文 / 国家(与 Natural Earth 名称一致) / lat / lon / 人口(百万) / GDP(十亿$) / 周航班 */
const CITY_RAW = [
  ['PEK', '北京', 'Beijing', 'China', 39.90, 116.41, 21.9, 700, 1800],
  ['PVG', '上海', 'Shanghai', 'China', 31.23, 121.47, 24.9, 760, 2000],
  ['CAN', '广州', 'Guangzhou', 'China', 23.13, 113.26, 18.7, 430, 1200],
  ['SZX', '深圳', 'Shenzhen', 'China', 22.54, 114.06, 17.6, 480, 900],
  ['HKG', '香港', 'Hong Kong', 'China', 22.32, 114.17, 7.4, 380, 1600],
  ['CTU', '成都', 'Chengdu', 'China', 30.57, 104.07, 21.4, 320, 700],
  ['HND', '东京', 'Tokyo', 'Japan', 35.68, 139.69, 37.2, 1600, 1900],
  ['KIX', '大阪', 'Osaka', 'Japan', 34.69, 135.50, 19.0, 700, 800],
  ['ICN', '首尔', 'Seoul', 'South Korea', 37.57, 126.98, 25.6, 900, 1500],
  ['SIN', '新加坡', 'Singapore', 'Singapore', 1.35, 103.82, 6.0, 500, 1700],
  ['BKK', '曼谷', 'Bangkok', 'Thailand', 13.76, 100.50, 17.4, 220, 1300],
  ['KUL', '吉隆坡', 'Kuala Lumpur', 'Malaysia', 3.14, 101.69, 8.4, 180, 1000],
  ['CGK', '雅加达', 'Jakarta', 'Indonesia', -6.21, 106.85, 34.5, 250, 800],
  ['MNL', '马尼拉', 'Manila', 'Philippines', 14.60, 120.98, 24.9, 180, 700],
  ['SGN', '胡志明市', 'Ho Chi Minh City', 'Vietnam', 10.82, 106.63, 9.3, 90, 500],
  ['BOM', '孟买', 'Mumbai', 'India', 19.08, 72.88, 21.7, 310, 700],
  ['DEL', '德里', 'Delhi', 'India', 28.61, 77.21, 32.9, 290, 800],
  ['DXB', '迪拜', 'Dubai', 'United Arab Emirates', 25.20, 55.27, 3.6, 160, 2400],
  ['DOH', '多哈', 'Doha', 'Qatar', 25.29, 51.53, 2.4, 120, 1600],
  ['IST', '伊斯坦布尔', 'Istanbul', 'Turkey', 41.01, 28.98, 15.9, 280, 2000],
  ['SVO', '莫斯科', 'Moscow', 'Russia', 55.76, 37.62, 12.7, 350, 900],
  ['TLV', '特拉维夫', 'Tel Aviv', 'Israel', 32.08, 34.78, 4.2, 180, 500],
  ['CAI', '开罗', 'Cairo', 'Egypt', 30.04, 31.24, 22.2, 150, 600],
  ['JNB', '约翰内斯堡', 'Johannesburg', 'South Africa', -26.20, 28.05, 6.2, 110, 400],
  ['LOS', '拉各斯', 'Lagos', 'Nigeria', 6.52, 3.38, 15.9, 90, 300],
  ['NBO', '内罗毕', 'Nairobi', 'Kenya', -1.29, 36.82, 5.3, 40, 250],
  ['LHR', '伦敦', 'London', 'United Kingdom', 51.51, -0.13, 9.6, 1000, 2600],
  ['CDG', '巴黎', 'Paris', 'France', 48.86, 2.35, 11.1, 850, 1900],
  ['FRA', '法兰克福', 'Frankfurt', 'Germany', 50.11, 8.68, 2.4, 200, 1700],
  ['AMS', '阿姆斯特丹', 'Amsterdam', 'Netherlands', 52.37, 4.90, 2.5, 200, 1600],
  ['MAD', '马德里', 'Madrid', 'Spain', 40.42, -3.70, 6.7, 280, 1200],
  ['FCO', '罗马', 'Rome', 'Italy', 41.90, 12.50, 4.3, 200, 900],
  ['ZRH', '苏黎世', 'Zurich', 'Switzerland', 47.38, 8.54, 1.5, 150, 700],
  ['ARN', '斯德哥尔摩', 'Stockholm', 'Sweden', 59.33, 18.07, 2.4, 130, 500],
  ['JFK', '纽约', 'New York', 'United States of America', 40.71, -74.01, 18.9, 2100, 2500],
  ['LAX', '洛杉矶', 'Los Angeles', 'United States of America', 34.05, -118.24, 12.9, 1200, 1800],
  ['ORD', '芝加哥', 'Chicago', 'United States of America', 41.88, -87.63, 8.9, 780, 1600],
  ['MIA', '迈阿密', 'Miami', 'United States of America', 25.76, -80.19, 6.3, 400, 1200],
  ['SFO', '旧金山', 'San Francisco', 'United States of America', 37.77, -122.42, 4.7, 650, 1100],
  ['SEA', '西雅图', 'Seattle', 'United States of America', 47.61, -122.33, 4.0, 450, 800],
  ['YYZ', '多伦多', 'Toronto', 'Canada', 43.65, -79.38, 6.4, 400, 1000],
  ['YVR', '温哥华', 'Vancouver', 'Canada', 49.28, -123.12, 2.6, 160, 600],
  ['MEX', '墨西哥城', 'Mexico City', 'Mexico', 19.43, -99.13, 22.3, 400, 900],
  ['GRU', '圣保罗', 'Sao Paulo', 'Brazil', -23.55, -46.63, 22.6, 300, 800],
  ['GIG', '里约热内卢', 'Rio de Janeiro', 'Brazil', -22.91, -43.17, 13.6, 130, 400],
  ['EZE', '布宜诺斯艾利斯', 'Buenos Aires', 'Argentina', -34.60, -58.38, 15.6, 180, 500],
  ['LIM', '利马', 'Lima', 'Peru', -12.05, -77.04, 11.0, 90, 350],
  ['BOG', '波哥大', 'Bogota', 'Colombia', 4.71, -74.07, 11.3, 110, 450],
  ['SCL', '圣地亚哥', 'Santiago', 'Chile', -33.45, -70.67, 6.9, 120, 400],
  ['SYD', '悉尼', 'Sydney', 'Australia', -33.87, 151.21, 5.3, 400, 900],
  ['MEL', '墨尔本', 'Melbourne', 'Australia', -37.81, 144.96, 5.1, 350, 700],
  ['AKL', '奥克兰', 'Auckland', 'New Zealand', -36.85, 174.76, 1.7, 90, 350]
];

const CITIES = CITY_RAW.map(([code, nm, en, country, lat, lon, pop, gdp, flights]) => ({
  code, nm, en, country, lat, lon,
  population: pop, gdp, flights,
  g: { // 年增长率（可复现随机）
    population: 0.003 + hash(en + '#p') * 0.030,
    gdp:        0.015 + hash(en + '#g') * 0.060,
    flights:    0.008 + hash(en + '#f') * 0.050
  }
}));
const CITY_BY_CODE = new Map(CITIES.map(c => [c.code, c]));

/* 航线：起点 / 终点 / 周班次 / 承运 */
const ROUTE_RAW = [
  ['PEK','LHR',42,'中国国航 CA937'], ['PEK','JFK',28,'中国国航 CA981'], ['PEK','FRA',35,'汉莎 LH723'],
  ['PVG','JFK',35,'中国东航 MU587'], ['PVG','LAX',28,'中国东航 MU583'], ['PVG','SIN',56,'新加坡航空 SQ831'],
  ['PVG','HND',63,'全日空 NH922'], ['CAN','SYD',21,'中国南航 CZ325'], ['CAN','DXB',28,'阿联酋航空 EK363'],
  ['SZX','BKK',35,'深圳航空 ZH9051'], ['HKG','LHR',49,'国泰航空 CX251'], ['HKG','SFO',35,'国泰航空 CX870'],
  ['HKG','SIN',70,'国泰航空 CX735'], ['CTU','AMS',14,'荷兰皇家航空 KL425'], ['HND','LAX',42,'日本航空 JL016'],
  ['HND','SIN',49,'全日空 NH843'], ['KIX','ICN',56,'大韩航空 KE724'], ['ICN','JFK',35,'大韩航空 KE081'],
  ['ICN','SEA',21,'韩亚航空 OZ272'], ['SIN','LHR',42,'新加坡航空 SQ322'], ['SIN','SYD',49,'澳洲航空 QF082'],
  ['SIN','DXB',56,'阿联酋航空 EK405'], ['BKK','DXB',63,'阿联酋航空 EK375'], ['KUL','DOH',35,'卡塔尔航空 QR845'],
  ['CGK','DOH',28,'卡塔尔航空 QR955'], ['MNL','DXB',35,'菲律宾航空 PR658'], ['SGN','ICN',42,'越南航空 VN408'],
  ['BOM','LHR',35,'英国航空 BA138'], ['DEL','DXB',70,'印度航空 AI995'], ['DEL','JFK',21,'印度航空 AI101'],
  ['DXB','LHR',84,'阿联酋航空 EK001'], ['DXB','JFK',42,'阿联酋航空 EK201'], ['DXB','SYD',21,'阿联酋航空 EK412'],
  ['DOH','CDG',42,'卡塔尔航空 QR039'], ['IST','FRA',56,'土耳其航空 TK1587'], ['IST','JFK',35,'土耳其航空 TK001'],
  ['SVO','PEK',21,'俄罗斯航空 SU200'], ['TLV','JFK',28,'以色列航空 LY001'], ['CAI','LHR',28,'埃及航空 MS777'],
  ['JNB','LHR',21,'英国航空 BA056'], ['JNB','DXB',28,'阿联酋航空 EK762'], ['LOS','CDG',14,'法国航空 AF853'],
  ['NBO','AMS',21,'荷兰皇家航空 KL566'], ['LHR','JFK',126,'英国航空 BA117'], ['LHR','LAX',49,'维珍航空 VS007'],
  ['CDG','JFK',63,'法国航空 AF006'], ['CDG','GRU',21,'法国航空 AF454'], ['FRA','JFK',56,'汉莎 LH400'],
  ['FRA','GRU',21,'汉莎 LH506'], ['AMS','YYZ',28,'荷兰皇家航空 KL691'], ['MAD','EZE',35,'伊比利亚 IB6843'],
  ['MAD','MEX',28,'伊比利亚 IB6403'], ['FCO','JFK',35,'ITA Airways AZ610'], ['ZRH','JFK',21,'瑞士航空 LX014'],
  ['ARN','LHR',42,'北欧航空 SK529'], ['JFK','LAX',168,'达美航空 DL423'], ['JFK','MIA',140,'美国航空 AA1195'],
  ['ORD','LHR',42,'美联航 UA958'], ['SFO','HND',28,'美联航 UA837'], ['SEA','ICN',21,'达美航空 DL197'],
  ['LAX','SYD',35,'澳洲航空 QF012'], ['LAX','MEX',63,'美联航 UA1520'], ['YVR','PEK',14,'加拿大航空 AC029'],
  ['YYZ','YVR',84,'加拿大航空 AC101'], ['MEX','BOG',35,'哥伦比亚航空 AV048'], ['GRU','EZE',84,'阿根廷航空 AR1251'],
  ['GRU','GIG',105,'高尔航空 G31402'], ['GIG','MIA',28,'LATAM LA8180'], ['LIM','SCL',42,'LATAM LA600'],
  ['BOG','MIA',49,'哥伦比亚航空 AV024'], ['SCL','SYD',7,'LATAM LA800'], ['SYD','AKL',91,'澳洲航空 QF141'],
  ['MEL','SIN',42,'新加坡航空 SQ208'], ['AKL','LAX',21,'新西兰航空 NZ002']
];

const ROUTES = ROUTE_RAW.map(([a, b, w, air]) => {
  const A = CITY_BY_CODE.get(a), B = CITY_BY_CODE.get(b);
  return { a: A, b: B, w, air, growth: 0.005 + hash(a + b) * 0.05, dist: greatCircleKm(A, B) };
}).filter(r => r.a && r.b);

function greatCircleKm(a, b) {
  const p = a.lat * DEG, q = b.lat * DEG, d = (b.lon - a.lon) * DEG;
  return Math.round(6371 * Math.acos(clamp(Math.sin(p) * Math.sin(q) + Math.cos(p) * Math.cos(q) * Math.cos(d), -1, 1)));
}

/* 疫情对航班量的影响系数（让时间轴有故事性） */
const covid = y => y === 2020 ? 0.28 : y === 2021 ? 0.42 : y === 2022 ? 0.72 : y === 2023 ? 0.92 : 1;

function cityValue(c, metric, year) {
  let v = c[metric] * Math.pow(1 + c.g[metric], year - YEAR_BASE);
  if (metric === 'flights') v *= covid(year);
  return v;
}
function routeValue(r, year) {
  return r.w * Math.pow(1 + r.growth, year - YEAR_BASE) * covid(year);
}

/* 国家级数据（用于热力着色）：已知大国给定量级，其余程序生成 */
const COUNTRY_BASE = {
  'China': [1410, 17800, 1150], 'India': [1430, 3900, 620], 'United States of America': [335, 27700, 1400],
  'Indonesia': [278, 1400, 300], 'Pakistan': [240, 340, 70], 'Brazil': [216, 2170, 260],
  'Nigeria': [224, 390, 60], 'Bangladesh': [173, 440, 55], 'Russia': [144, 2020, 250],
  'Mexico': [129, 1790, 220], 'Japan': [124, 4200, 330], 'Egypt': [113, 400, 90],
  'Ethiopia': [127, 160, 30], 'Philippines': [118, 440, 130], 'Vietnam': [100, 430, 120],
  'Turkey': [85, 1120, 300], 'Türkiye': [85, 1120, 300], 'Germany': [84, 4460, 420],
  'France': [68, 3050, 380], 'United Kingdom': [68, 3340, 430], 'Italy': [59, 2250, 280],
  'Spain': [48, 1580, 300], 'Canada': [40, 2140, 210], 'Australia': [26, 1720, 160],
  'South Korea': [52, 1710, 210], 'Saudi Arabia': [37, 1070, 150], 'Iran': [89, 400, 70],
  'Thailand': [72, 510, 190], 'South Africa': [61, 380, 80], 'Argentina': [46, 640, 90],
  'Poland': [37, 810, 120], 'Colombia': [52, 360, 100], 'Kenya': [55, 110, 45],
  'Malaysia': [34, 400, 140], 'Netherlands': [18, 1120, 260], 'Switzerland': [9, 900, 130],
  'Sweden': [11, 590, 95], 'Norway': [5.5, 480, 80], 'Belgium': [12, 630, 95],
  'Austria': [9, 520, 90], 'Portugal': [10, 290, 95], 'Greece': [10, 240, 90],
  'Ireland': [5.2, 550, 85], 'Denmark': [5.9, 400, 80], 'Finland': [5.6, 300, 60],
  'Singapore': [5.9, 500, 190], 'United Arab Emirates': [9.5, 510, 260], 'Qatar': [2.7, 220, 175],
  'New Zealand': [5.2, 250, 60], 'Chile': [19.6, 340, 70], 'Peru': [34, 270, 55],
  'Ukraine': [37, 180, 20], 'Israel': [9.7, 520, 90]
};
const cBaseCache = new Map();
function countryBase(name) {
  if (COUNTRY_BASE[name]) return COUNTRY_BASE[name];
  if (cBaseCache.has(name)) return cBaseCache.get(name);
  const h1 = hash(name + '#P'), h2 = hash(name + '#G'), h3 = hash(name + '#F');
  const v = [0.3 + Math.pow(h1, 2.2) * 110, 1 + Math.pow(h2, 2.6) * 700, 2 + Math.pow(h3, 2.4) * 260];
  cBaseCache.set(name, v);
  return v;
}
const M_IDX = { population: 0, gdp: 1, flights: 2 };
function countryValue(name, metric, year) {
  const base = countryBase(name)[M_IDX[metric]];
  const g = metric === 'population' ? 0.002 + hash(name + 'gp') * 0.022
          : metric === 'gdp'        ? 0.012 + hash(name + 'gg') * 0.055
          :                           0.006 + hash(name + 'gf') * 0.045;
  let v = base * Math.pow(1 + g, year - YEAR_BASE);
  if (metric === 'flights') v *= covid(year);
  return v;
}

/* =========================================================================
   2. 场景 / 相机 / 控制器
   ========================================================================= */
let renderer;
try {
  renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
} catch (e) {
  $('#loading').innerHTML = '<div style="max-width:420px">当前浏览器不支持 WebGL，无法渲染 3D 地球。<br><small>请使用较新的 Chrome / Edge / Firefox / Safari。</small></div>';
  throw e;
}
renderer.setPixelRatio(Math.min(devicePixelRatio || 1, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.outputColorSpace = THREE.LinearSRGBColorSpace; // 直通输出
renderer.setClearColor(0x05070f, 1);
$('#stage').appendChild(renderer.domElement);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, innerWidth / innerHeight, 0.02, 500);
camera.position.set(0.9, 1.05, 2.7);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.07;
controls.enablePan = false;
controls.rotateSpeed = 0.55;
controls.zoomSpeed = 0.85;
controls.minDistance = 1.16;
controls.maxDistance = 7;

const earthGroup = new THREE.Group();   // 地球 + 所有随地球自转的图层
scene.add(earthGroup);

/* ---- 占位纹理（网络纹理到位前先能渲染） ---- */
function solidTexture(r, g, b, a = 255) {
  const t = new THREE.DataTexture(new Uint8Array([r, g, b, a]), 1, 1, THREE.RGBAFormat);
  t.needsUpdate = true;
  return t;
}

const sunDir = new THREE.Vector3(1, 0, 0);

const earthUniforms = {
  uDay:      { value: solidTexture(28, 62, 110) },
  uNight:    { value: solidTexture(0, 0, 0) },
  uSpec:     { value: solidTexture(0, 0, 0) },
  uHeat:     { value: null },
  uSun:      { value: sunDir },
  uCamPos:   { value: camera.position },
  uHeatOp:   { value: 0.72 },
  uNightBoost: { value: 1.15 },
  uHasNight: { value: 0 },
  uHasSpec:  { value: 0 },
  uHasHeat:  { value: 0 }
};

const EARTH_VS = /* glsl */`
  varying vec2 vUv;
  varying vec3 vN;
  varying vec3 vW;
  void main(){
    vUv = uv;
    vN = normalize(mat3(modelMatrix) * normal);
    vec4 wp = modelMatrix * vec4(position, 1.0);
    vW = wp.xyz;
    gl_Position = projectionMatrix * viewMatrix * wp;
  }
`;

const EARTH_FS = /* glsl */`
  uniform sampler2D uDay, uNight, uSpec, uHeat;
  uniform vec3 uSun, uCamPos;
  uniform float uHeatOp, uNightBoost, uHasNight, uHasSpec, uHasHeat;
  varying vec2 vUv; varying vec3 vN; varying vec3 vW;

  void main(){
    vec3 n = normalize(vN);
    vec3 L = normalize(uSun);
    vec3 V = normalize(uCamPos - vW);

    float sd   = dot(n, L);
    float dayF = smoothstep(-0.14, 0.20, sd);   // 日夜过渡带（晨昏线）
    float diff = max(sd, 0.0);

    vec3 albedo = texture2D(uDay, vUv).rgb;

    // 国家 / 地区热力着色
    vec4 heat = vec4(0.0);
    if (uHasHeat > 0.5) heat = texture2D(uHeat, vUv);
    float hw = heat.a * uHeatOp;
    albedo = mix(albedo, heat.rgb, hw * 0.85);

    vec3 col = albedo * (0.055 + 0.95 * pow(diff, 0.85));

    // 海面高光（specular map 中海洋为亮）
    if (uHasSpec > 0.5) {
      float water = texture2D(uSpec, vUv).r;
      vec3 H = normalize(V + L);
      col += vec3(0.85, 0.93, 1.0) * pow(max(dot(n, H), 0.0), 64.0) * water * diff * 0.75;
    }

    // 夜面城市灯光
    float nightF = 1.0 - dayF;
    if (uHasNight > 0.5) {
      vec3 lights = texture2D(uNight, vUv).rgb;
      col += lights * vec3(1.0, 0.86, 0.62) * nightF * uNightBoost * 1.7;
    }
    // 夜面热力自发光，避免暗面数据不可读
    col += heat.rgb * hw * nightF * 0.45;

    // 边缘蓝色散射
    float rim = pow(1.0 - max(dot(n, V), 0.0), 3.0);
    col += vec3(0.22, 0.44, 0.92) * rim * (0.20 + 0.80 * dayF) * 0.55;

    gl_FragColor = vec4(col, 1.0);
  }
`;

const earth = new THREE.Mesh(
  new THREE.SphereGeometry(R, 128, 96),
  new THREE.ShaderMaterial({ uniforms: earthUniforms, vertexShader: EARTH_VS, fragmentShader: EARTH_FS })
);
earth.name = 'earth';
earthGroup.add(earth);

/* ---- 云层 ---- */
const cloudUniforms = {
  uMap: { value: solidTexture(0, 0, 0, 0) },
  uSun: { value: sunDir },
  uAlphaMode: { value: 1 },   // 1: 用纹理 alpha，0: 用亮度
  uOpacity: { value: 0.85 }
};
const clouds = new THREE.Mesh(
  new THREE.SphereGeometry(R * 1.012, 96, 64),
  new THREE.ShaderMaterial({
    uniforms: cloudUniforms,
    vertexShader: EARTH_VS,
    fragmentShader: /* glsl */`
      uniform sampler2D uMap; uniform vec3 uSun; uniform float uAlphaMode, uOpacity;
      varying vec2 vUv; varying vec3 vN; varying vec3 vW;
      void main(){
        vec4 c = texture2D(uMap, vUv);
        float lum = (c.r + c.g + c.b) / 3.0;
        float a = mix(lum, c.a, uAlphaMode);
        float d = max(dot(normalize(vN), normalize(uSun)), 0.0);
        vec3 col = vec3(1.0) * (0.06 + 0.94 * pow(d, 0.8));
        gl_FragColor = vec4(col, a * uOpacity);
      }
    `,
    transparent: true, depthWrite: false
  })
);
earthGroup.add(clouds);

/* ---- 大气光晕（背面壳 + 菲涅尔） ---- */
const atmosphere = new THREE.Mesh(
  new THREE.SphereGeometry(R * 1.17, 64, 48),
  new THREE.ShaderMaterial({
    uniforms: { uSun: { value: sunDir }, uStrength: { value: 1.0 } },
    vertexShader: /* glsl */`
      varying vec3 vNv; varying vec3 vNw;
      void main(){
        vNv = normalize(normalMatrix * normal);
        vNw = normalize(mat3(modelMatrix) * normal);
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: /* glsl */`
      uniform vec3 uSun; uniform float uStrength;
      varying vec3 vNv; varying vec3 vNw;
      void main(){
        float f = pow(clamp(0.72 - dot(vNv, vec3(0.0, 0.0, 1.0)), 0.0, 2.0), 3.4);
        float lit = smoothstep(-0.45, 0.45, dot(normalize(vNw), normalize(uSun)));
        vec3 col = mix(vec3(0.10, 0.22, 0.55), vec3(0.42, 0.68, 1.0), lit);
        gl_FragColor = vec4(col * f * uStrength * (0.30 + 0.70 * lit), 1.0);
      }
    `,
    side: THREE.BackSide, blending: THREE.AdditiveBlending, transparent: true, depthWrite: false
  })
);
scene.add(atmosphere);

/* ---- 星空 ---- */
const stars = (() => {
  const N = 4200, pos = new Float32Array(N * 3), col = new Float32Array(N * 3);
  for (let i = 0; i < N; i++) {
    const u = Math.random() * 2 - 1, th = Math.random() * Math.PI * 2, s = Math.sqrt(1 - u * u);
    const r = 60 + Math.random() * 120;
    pos.set([r * s * Math.cos(th), r * u, r * s * Math.sin(th)], i * 3);
    const w = 0.55 + Math.random() * 0.45, t = Math.random();
    col.set([w * (0.8 + 0.2 * t), w * (0.85 + 0.15 * (1 - t)), w], i * 3);
  }
  const g = new THREE.BufferGeometry();
  g.setAttribute('position', new THREE.BufferAttribute(pos, 3));
  g.setAttribute('color', new THREE.BufferAttribute(col, 3));
  const p = new THREE.Points(g, new THREE.PointsMaterial({
    size: 0.55, sizeAttenuation: true, vertexColors: true, transparent: true, opacity: 0.9,
    blending: THREE.AdditiveBlending, depthWrite: false
  }));
  scene.add(p);
  return p;
})();

/* =========================================================================
   3. 城市图层：发光点 + 数据光柱 + 拾取体
   ========================================================================= */
const cityGroup = new THREE.Group();  earthGroup.add(cityGroup);
const barGroup  = new THREE.Group();  earthGroup.add(barGroup);
const pickGroup = new THREE.Group();  earthGroup.add(pickGroup);

const glowTex = (() => {
  const s = 128, c = document.createElement('canvas'); c.width = c.height = s;
  const g = c.getContext('2d');
  const grd = g.createRadialGradient(s / 2, s / 2, 0, s / 2, s / 2, s / 2);
  grd.addColorStop(0.00, 'rgba(255,255,255,1)');
  grd.addColorStop(0.16, 'rgba(255,255,255,.92)');
  grd.addColorStop(0.42, 'rgba(255,255,255,.30)');
  grd.addColorStop(1.00, 'rgba(255,255,255,0)');
  g.fillStyle = grd; g.fillRect(0, 0, s, s);
  return new THREE.CanvasTexture(c);
})();

const BAR_GEO = (() => {
  const g = new THREE.CylinderGeometry(0.0042, 0.0042, 1, 8, 1, true);
  g.translate(0, 0.5, 0);
  return g;
})();
const BAR_VS = /* glsl */`varying vec2 vUv; void main(){ vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);} `;
const BAR_FS = /* glsl */`
  uniform vec3 uColor; uniform float uOpacity;
  varying vec2 vUv;
  void main(){
    float a = pow(1.0 - vUv.y, 1.5) * 0.85 + 0.12;
    gl_FragColor = vec4(uColor * (0.65 + 0.9 * (1.0 - vUv.y)), a * uOpacity);
  }
`;
const DOT_GEO = new THREE.SphereGeometry(0.0075, 10, 8);
const PICK_GEO = new THREE.SphereGeometry(0.028, 8, 6);
const PICK_MAT = new THREE.MeshBasicMaterial({ visible: false });

const UP = new THREE.Vector3(0, 1, 0);
for (const c of CITIES) {
  const n = latLonToVec3(c.lat, c.lon, 1);
  c.pos = n.clone().multiplyScalar(R * 1.002);
  c.normal = n.clone();

  const sprite = new THREE.Sprite(new THREE.SpriteMaterial({
    map: glowTex, color: 0x66e0ff, transparent: true, depthWrite: false,
    blending: THREE.AdditiveBlending, opacity: 0.95
  }));
  sprite.position.copy(n).multiplyScalar(R * 1.004);
  sprite.scale.setScalar(0.05);
  cityGroup.add(sprite);

  const dot = new THREE.Mesh(DOT_GEO, new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.9 }));
  dot.position.copy(n).multiplyScalar(R * 1.001);
  cityGroup.add(dot);

  const bar = new THREE.Mesh(BAR_GEO, new THREE.ShaderMaterial({
    uniforms: { uColor: { value: new THREE.Color(0x4fd1ff) }, uOpacity: { value: 0.9 } },
    vertexShader: BAR_VS, fragmentShader: BAR_FS,
    transparent: true, depthWrite: false, blending: THREE.AdditiveBlending
  }));
  bar.position.copy(n).multiplyScalar(R);
  bar.quaternion.setFromUnitVectors(UP, n);
  barGroup.add(bar);

  const pick = new THREE.Mesh(PICK_GEO, PICK_MAT);
  pick.position.copy(n).multiplyScalar(R * 1.01);
  pick.userData.city = c;
  pickGroup.add(pick);

  c.sprite = sprite; c.dot = dot; c.bar = bar;
  c.routes = 0;
}
for (const r of ROUTES) { r.a.routes++; r.b.routes++; }

/* =========================================================================
   4. 航线图层：大圆弧 TubeGeometry + 流光着色器
   ========================================================================= */
const arcGroup  = new THREE.Group(); earthGroup.add(arcGroup);
const arcPickGroup = new THREE.Group(); earthGroup.add(arcPickGroup);

function arcCurve(a, b) {
  const va = latLonToVec3(a.lat, a.lon, 1), vb = latLonToVec3(b.lat, b.lon, 1);
  const ang = va.angleTo(vb);
  const lift = 0.06 + 0.30 * (ang / Math.PI);
  const pts = [];
  const seg = 72;
  for (let i = 0; i <= seg; i++) {
    const t = i / seg;
    const p = va.clone().lerp(vb, t).normalize();
    p.multiplyScalar(R * (1.004 + Math.sin(Math.PI * t) * lift));
    pts.push(p);
  }
  return new THREE.CatmullRomCurve3(pts);
}

const ARC_FS = /* glsl */`
  uniform float uTime, uOpacity, uSpeed, uHover;
  uniform vec3 uColor;
  varying vec2 vUv;
  void main(){
    float x = vUv.x;
    float edge = smoothstep(0.0, 0.05, x) * smoothstep(1.0, 0.95, x);
    float head = fract(x - uTime * uSpeed);
    float comet = pow(head, 14.0);                 // 流动光头
    float tail  = pow(head, 3.0) * 0.22;           // 拖尾
    float body  = 0.16 + 0.10 * sin(3.14159 * x);
    float a = (body + tail + comet * 1.5) * edge * uOpacity * (1.0 + uHover * 1.6);
    vec3 col = mix(uColor, vec3(1.0), comet * 0.85 + uHover * 0.35);
    gl_FragColor = vec4(col * (0.75 + 1.9 * comet + uHover * 0.6), a);
  }
`;
const ARC_VS = /* glsl */`varying vec2 vUv; void main(){ vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);} `;

for (const r of ROUTES) {
  const curve = arcCurve(r.a, r.b);
  const mat = new THREE.ShaderMaterial({
    uniforms: {
      uTime: { value: 0 }, uOpacity: { value: 0.8 }, uSpeed: { value: 0.12 },
      uHover: { value: 0 }, uColor: { value: new THREE.Color(0x4fd1ff) }
    },
    vertexShader: ARC_VS, fragmentShader: ARC_FS,
    transparent: true, depthWrite: false, blending: THREE.AdditiveBlending
  });
  const tube = new THREE.Mesh(new THREE.TubeGeometry(curve, 110, 0.0028, 5, false), mat);
  tube.renderOrder = 2;
  arcGroup.add(tube);

  const pick = new THREE.Mesh(new THREE.TubeGeometry(curve, 60, 0.016, 4, false), PICK_MAT);
  pick.userData.route = r;
  arcPickGroup.add(pick);

  r.mesh = tube; r.pick = pick; r.mat = mat;
}

/* =========================================================================
   5. 地理边界 / 热力纹理
   ========================================================================= */
let FEATURES = null;                          // GeoJSON features（可能为 null）
const pathCache = new Map();                  // feature -> {w,h,path}
const HW = 2048, HH = 1024;
const heatCanvas = document.createElement('canvas');
heatCanvas.width = HW; heatCanvas.height = HH;
const heatCtx = heatCanvas.getContext('2d');
const heatTex = new THREE.CanvasTexture(heatCanvas);
heatTex.anisotropy = 4;
earthUniforms.uHeat.value = heatTex;

/** 经纬度环 -> Path2D（处理跨 180° 经线：整环平移 ±360 各画一次） */
function ringsToPath(geom, W, H) {
  const p = new Path2D();
  const polys = geom.type === 'Polygon' ? [geom.coordinates] : geom.coordinates;
  for (const poly of polys) {
    for (const ring of poly) {
      const pts = [];
      let prev = null;
      for (const c of ring) {
        let lon = c[0];
        if (prev !== null) { while (lon - prev > 180) lon -= 360; while (prev - lon > 180) lon += 360; }
        prev = lon;
        pts.push([lon, c[1]]);
      }
      for (const off of [-360, 0, 360]) {
        for (let i = 0; i < pts.length; i++) {
          const x = (pts[i][0] + off + 180) / 360 * W;
          const y = (90 - pts[i][1]) / 180 * H;
          if (i === 0) p.moveTo(x, y); else p.lineTo(x, y);
        }
        p.closePath();
      }
    }
  }
  return p;
}
function pathFor(f, W, H) {
  let e = pathCache.get(f);
  if (!e || e.w !== W || e.h !== H) { e = { w: W, h: H, path: ringsToPath(f.geometry, W, H) }; pathCache.set(f, e); }
  return e.path;
}

const rangeCache = new Map();
function countryRange(metric, year) {
  const k = metric + year;
  if (rangeCache.has(k)) return rangeCache.get(k);
  let min = Infinity, max = -Infinity;
  for (const f of FEATURES) {
    const v = countryValue(f.properties.name || '?', metric, year);
    if (v < min) min = v; if (v > max) max = v;
  }
  const r = { min, max };
  rangeCache.set(k, r);
  return r;
}

function drawHeat(metric, year) {
  heatCtx.clearRect(0, 0, HW, HH);
  if (FEATURES) {
    const { min, max } = countryRange(metric, year);
    const lmin = Math.log(min + 1), lmax = Math.log(max + 1);
    for (const f of FEATURES) {
      const v = countryValue(f.properties.name || '?', metric, year);
      const t = clamp((Math.log(v + 1) - lmin) / (lmax - lmin || 1), 0, 1);
      const c = ramp(t);
      heatCtx.fillStyle = `rgba(${c[0] | 0},${c[1] | 0},${c[2] | 0},${(0.16 + 0.55 * t).toFixed(3)})`;
      heatCtx.fill(pathFor(f, HW, HH));
    }
    heatCtx.strokeStyle = 'rgba(200,225,255,0.13)';
    heatCtx.lineWidth = 1.1;
    for (const f of FEATURES) heat