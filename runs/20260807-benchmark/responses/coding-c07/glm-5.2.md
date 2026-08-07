我目前处于 **Ask 模式**，无法直接创建或修改文件（包括写入 HTML 文件、运行命令等）。但我可以为你提供完整的实现方案和代码，你可以将其保存为单个 HTML 文件直接运行。

下面是完整可运行的代码，包含你要求的所有功能。

## 完整代码

将以下内容保存为 `globe.html`，然后用浏览器打开即可：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<title>3D 数据可视化地球仪</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body { width: 100%; height: 100%; overflow: hidden; background: #000; font-family: -apple-system, "Segoe UI", sans-serif; color: #fff; }
  #app { position: relative; width: 100vw; height: 100vh; }
  canvas { display: block; }

  /* 控制面板 */
  #panel {
    position: absolute; top: 16px; left: 16px; z-index: 10;
    background: rgba(10, 18, 32, 0.78); backdrop-filter: blur(10px);
    border: 1px solid rgba(80,160,255,0.25); border-radius: 12px;
    padding: 16px; width: 260px; box-shadow: 0 8px 32px rgba(0,0,0,0.5);
  }
  #panel h2 { font-size: 14px; margin-bottom: 12px; color: #6cf; letter-spacing: 1px; }
  .group { margin-bottom: 14px; }
  .group label { display:block; font-size: 11px; color: #9bd; margin-bottom: 6px; }
  .metric-btns { display:flex; flex-wrap:wrap; gap:6px; }
  .metric-btns button {
    flex:1; min-width: 60px; padding: 6px 8px; font-size: 11px;
    background: rgba(60,120,200,0.15); color: #cde; border:1px solid rgba(120,180,255,0.3);
    border-radius: 6px; cursor: pointer; transition: all .2s;
  }
  .metric-btns button.active { background: rgba(80,160,255,0.5); color:#fff; border-color:#6cf; }
  .metric-btns button:hover { background: rgba(80,160,255,0.35); }

  #timeSlider { width: 100%; accent-color: #6cf; }
  #timeLabel { font-size: 11px; color:#9bd; margin-top:4px; }
  #playBtn {
    margin-top: 8px; padding: 6px 12px; font-size: 12px; cursor:pointer;
    background: rgba(80,160,255,0.3); color:#fff; border:1px solid rgba(120,180,255,0.4);
    border-radius: 6px;
  }

  /* 弹窗 */
  #popup {
    position: absolute; z-index: 20; display:none; pointer-events:none;
    background: rgba(8,16,28,0.92); border:1px solid rgba(120,180,255,0.5);
    border-radius: 8px; padding: 10px 12px; font-size: 12px; min-width: 160px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.6);
  }
  #popup h3 { color:#6cf; font-size: 13px; margin-bottom: 6px; }
  #popup .row { color:#bcd; margin: 2px 0; }
  #flightTip {
    position: absolute; z-index: 20; display:none; pointer-events:none;
    background: rgba(8,16,28,0.92); border:1px solid rgba(255,200,100,0.5);
    border-radius: 6px; padding: 6px 10px; font-size: 11px; color:#ffd;
  }

  #hint {
    position:absolute; bottom:12px; left:50%; transform:translateX(-50%);
    font-size:11px; color:rgba(180,210,255,0.6); z-index:5;
  }
</style>
</head>
<body>
<div id="app">
  <div id="panel">
    <h2>🌍 数据可视化地球</h2>
    <div class="group">
      <label>数据指标</label>
      <div class="metric-btns" id="metricBtns">
        <button data-metric="gdp" class="active">GDP</button>
        <button data-metric="population">人口</button>
        <button data-metric="flights">航班量</button>
      </div>
    </div>
    <div class="group">
      <label>时间轴 <span id="timeLabel">2020</span></label>
      <input type="range" id="timeSlider" min="0" max="4" value="0" step="1" />
      <button id="playBtn">▶ 播放</button>
    </div>
  </div>

  <div id="popup"></div>
  <div id="flightTip"></div>
  <div id="hint">拖拽旋转 · 滚轮缩放 · 点击城市 · 悬停航线</div>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// ---------- 数据 ----------
const CITIES = [
  { name: '北京',     lat: 39.90, lon: 116.40, gdp: [530,580,640,720,780], population: [21.5,21.8,22.0,22.2,22.4], flights: [980,1020,1080,1100,1180] },
  { name: '上海',     lat: 31.23, lon: 121.47, gdp: [470,510,560,610,660], population: [24.2,24.4,24.5,24.6,24.7], flights: [1020,1080,1120,1180,1240] },
  { name: '东京',     lat: 35.68, lon: 139.69, gdp: [1600,1650,1700,1750,1800], population: [37.0,37.2,37.4,37.5,37.6], flights: [880,910,940,970,1000] },
  { name: '纽约',     lat: 40.71, lon: -74.00, gdp: [1700,1750,1820,1880,1950], population: [8.4,8.5,8.6,8.7,8.8], flights: [1200,1240,1280,1320,1360] },
  { name: '伦敦',     lat: 51.51, lon: -0.13,  gdp: [650,680,710,740,770], population: [9.0,9.1,9.2,9.3,9.4], flights: [1050,1080,1110,1140,1170] },
  { name: '巴黎',     lat: 48.86, lon: 2.35,   gdp: [700,730,760,790,820], population: [2.2,2.2,2.3,2.3,2.3], flights: [820,850,880,910,940] },
  { name: '新加坡',   lat: 1.35,  lon: 103.82, gdp: [360,380,400,430,460], population: [5.7,5.8,5.9,6.0,6.1], flights: [680,720,760,800,840] },
  { name: '悉尼',     lat: -33.87,lon: 151.21, gdp: [330,350,370,390,410], population: [5.3,5.4,5.5,5.6,5.7], flights: [420,440,460,480,500] },
  { name: '迪拜',     lat: 25.20, lon: 55.27,  gdp: [360,380,400,420,440], population: [3.4,3.5,3.6,3.7,3.8], flights: [720,760,800,840,880] },
  { name: '莫斯科',   lat: 55.75, lon: 37.62,  gdp: [420,440,460,480,500], population: [12.5,12.6,12.7,12.8,12.9], flights: [560,580,600,620,640] },
  { name: '圣保罗',   lat: -23.55,lon: -46.63, gdp: [430,450,470,490,510], population: [12.3,12.4,12.5,12.6,12.7], flights: [380,400,420,440,460] },
  { name: '孟买',     lat: 19.08, lon: 72.88,  gdp: [260,290,320,350,380], population: [20.4,20.8,21.2,21.6,22.0], flights: [520,560,600,640,680] },
];

const ROUTES = [
  [0,3],[0,2],[1,4],[2,5],[6,7],[8,2],[9,3],[10,11],[3,4],[5,6],[0,11],[7,3]
];

const YEARS = [2000,2010,2018,2020,2024];
let currentMetric = 'gdp';
let currentYearIdx = 0;

// ---------- 场景 ----------
const app = document.getElementById('app');
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, innerWidth/innerHeight, 0.1, 1000);
camera.position.set(0, 0, 8);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(devicePixelRatio);
renderer.setSize(innerWidth, innerHeight);
app.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.08;
controls.minDistance = 3;
controls.maxDistance = 20;
controls.rotateSpeed = 0.5;

scene.add(new THREE.AmbientLight(0x404060, 0.6));
const sunLight = new THREE.DirectionalLight(0xffffff, 1.4);
sunLight.position.set(5, 3, 5);
scene.add(sunLight);

// ---------- 地球 ----------
const R = 2;
const earthGroup = new THREE.Group();
scene.add(earthGroup);

const loader = new THREE.TextureLoader();
loader.setCrossOrigin('anonymous');

// 使用公开地球纹理（昼夜各一）
const dayTex = loader.load('https://unpkg.com/three-globe@2.31.0/example/img/earth-blue-marble.jpg');
const nightTex = loader.load('https://unpkg.com/three-globe@2.31.0/example/img/earth-night.jpg');
const bumpTex = loader.load('https://unpkg.com/three-globe@2.31.0/example/img/earth-topology.png');
const specTex = loader.load('https://unpkg.com/three-globe@2.31.0/example/img/earth-water.png');

// 自定义 ShaderMaterial：日夜分界 + 城市灯光
const earthMat = new THREE.ShaderMaterial({
  uniforms: {
    dayTexture:   { value: dayTex },
    nightTexture: { value: nightTex },
    bumpTexture:  { value: bumpTex },
    specTexture:  { value: specTex },
    sunDirection: { value: new THREE.Vector3(5,3,5).normalize() },
  },
  vertexShader: `
    varying vec2 vUv;
    varying vec3 vNormal;
    void main() {
      vUv = uv;
      vNormal = normalize(normalMatrix * normal);
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
    }
  `,
  fragmentShader: `
    uniform sampler2D dayTexture;
    uniform sampler2D nightTexture;
    uniform sampler2D specTexture;
    uniform vec3 sunDirection;
    varying vec2 vUv;
    varying vec3 vNormal;
    void main() {
      float intensity = dot(normalize(vNormal), normalize(sunDirection));
      float dayMix = smoothstep(-0.15, 0.25, intensity);
      vec3 day = texture2D(dayTexture, vUv).rgb;
      vec3 night = texture2D(nightTexture, vUv).rgb;
      // 夜晚增强城市灯光
      vec3 nightCol = night * 2.2;
      vec3 color = mix(nightCol, day, dayMix);
      // 终止线柔光
      float rim = 1.0 - abs(intensity);
      rim = pow(rim, 6.0) * 0.3;
      color += vec3(0.3,0.5,0.8) * rim;
      gl_FragColor = vec4(color, 1.0);
    }
  `,
});

const earth = new THREE.Mesh(new THREE.SphereGeometry(R, 64, 64), earthMat);
earthGroup.add(earth);

// 大气光晕（外层 Fresnel）
const atmosphereMat = new THREE.ShaderMaterial({
  uniforms: { sunDirection: { value: new THREE.Vector3(5,3,5).normalize() } },
  vertexShader: `
    varying vec3 vNormal;
    varying vec3 vWorldPos;
    void main() {
      vNormal = normalize(normalMatrix * normal);
      vWorldPos = (modelMatrix * vec4(position,1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
    }
  `,
  fragmentShader: `
    uniform vec3 sunDirection;
    varying vec3 vNormal;
    varying vec3 vWorldPos;
    void main() {
      vec3 viewDir = normalize(cameraPosition - vWorldPos);
      float fres = pow(1.0 - max(dot(viewDir, vNormal), 0.0), 2.5);
      float sun = max(dot(normalize(vNormal), normalize(sunDirection)), 0.0);
      vec3 col = mix(vec3(0.1,0.2,0.5), vec3(0.4,0.7,1.0), sun);
      gl_FragColor = vec4(col, fres * 0.9);
    }
  `,
  side: THREE.BackSide,
  transparent: true,
  blending: THREE.AdditiveBlending,
  depthWrite: false,
});
const atmosphere = new THREE.Mesh(new THREE.SphereGeometry(R*1.15, 64, 64), atmosphereMat);
earthGroup.add(atmosphere);

// ---------- 经纬度转坐标 ----------
function latLonToVec3(lat, lon, radius=R) {
  const phi = (90 - lat) * Math.PI/180;
  const theta = (lon + 180) * Math.PI/180;
  return new THREE.Vector3(
    -radius * Math.sin(phi) * Math.cos(theta),
     radius * Math.cos(phi),
     radius * Math.sin(phi) * Math.sin(theta)
  );
}

// ---------- 城市标记 ----------
const cityGroup = new THREE.Group();
earthGroup.add(cityGroup);
const cityMeshes = [];

CITIES.forEach((city, i) => {
  const pos = latLonToVec3(city.lat, city.lon, R*1.01);

  // 发光点
  const dotGeo = new THREE.SphereGeometry(0.025, 12, 12);
  const dotMat = new THREE.MeshBasicMaterial({ color: 0x66ddff });
  const dot = new THREE.Mesh(dotGeo, dotMat);
  dot.position.copy(pos);
  dot.userData = { cityIndex: i };
  cityGroup.add(dot);
  cityMeshes.push(dot);

  // 光晕 sprite
  const haloMat = new THREE.SpriteMaterial({
    map: makeGlowTexture(), color: 0x66ddff, transparent:true,
    blending: THREE.AdditiveBlending, depthWrite:false
  });
  const halo = new THREE.Sprite(haloMat);
  halo.scale.set(0.25,0.25,1);
  halo.position.copy(pos);
  cityGroup.add(halo);
  dot.userData.halo = halo;
});

function makeGlowTexture() {
  const c = document.createElement('canvas');
  c.width = c.height = 64;
  const ctx = c.getContext('2d');
  const g = ctx.createRadialGradient(32,32,0,32,32,32);
  g.addColorStop(0, 'rgba(120,220,255,1)');
  g.addColorStop(0.4, 'rgba(80,180,255,0.5)');
  g.addColorStop(1, 'rgba(0,80,200,0)');
  ctx.fillStyle = g;
  ctx.fillRect(0,0,64,64继续上一段代码（从 `makeGlowTexture` 函数返回处接续）：

```javascript
  const tex = new THREE.CanvasTexture(c);
  return tex;
}

// ---------- 航线弧线 ----------
const routeGroup = new THREE.Group();
earthGroup.add(routeGroup);
const routeMeshes = [];

function createArc(a, b) {
  const start = latLonToVec3(a.lat, a.lon, R*1.01);
  const end   = latLonToVec3(b.lat, b.lon, R*1.01);
  const mid = start.clone().add(end).multiplyScalar(0.5);
  const dist = start.distanceTo(end);
  mid.normalize().multiplyScalar(R + dist*0.35 + 0.1);

  const curve = new THREE.QuadraticBezierCurve3(start, mid, end);
  const points = curve.getPoints(60);
  const geo = new THREE.BufferGeometry().setFromPoints(points);

  // 流动渐变色（用顶点颜色）
  const colors = [];
  for (let i=0;i<points.length;i++){
    const t = i/(points.length-1);
    const c = new THREE.Color().setHSL(0.55, 0.8, 0.4 + 0.4*Math.sin(t*Math.PI));
    colors.push(c.r,c.g,c.b);
  }
  geo.setAttribute('color', new THREE.Float32BufferAttribute(colors,3));

  const mat = new THREE.LineBasicMaterial({ vertexColors:true, transparent:true, opacity:0.7 });
  const line = new THREE.Line(geo, mat);
  return { line, curve, points };
}

ROUTES.forEach(([i,j]) => {
  const arc = createArc(CITIES[i], CITIES[j]);
  arc.line.userData = { from: CITIES[i].name, to: CITIES[j].name, route: [i,j] };
  routeGroup.add(arc.line);
  routeMeshes.push(arc);
});

// ---------- 国家热力着色（简化：用城市点辐射光斑叠加） ----------
const heatGroup = new THREE.Group();
earthGroup.add(heatGroup);
const heatSprites = [];

CITIES.forEach((city, i) => {
  const pos = latLonToVec3(city.lat, city.lon, R*1.005);
  const mat = new THREE.SpriteMaterial({
    map: makeHeatTexture(), transparent:true, depthWrite:false,
    blending: THREE.AdditiveBlending, opacity:0.0
  });
  const sp = new THREE.Sprite(mat);
  sp.position.copy(pos);
  sp.scale.set(0.6,0.6,1);
  heatGroup.add(sp);
  heatSprites.push(sp);
});

function makeHeatTexture() {
  const c = document.createElement('canvas');
  c.width = c.height = 128;
  const ctx = c.getContext('2d');
  const g = ctx.createRadialGradient(64,64,0,64,64,64);
  g.addColorStop(0, 'rgba(255,80,40,0.9)');
  g.addColorStop(0.5, 'rgba(255,160,40,0.4)');
  g.addColorStop(1, 'rgba(255,200,80,0)');
  ctx.fillStyle = g; ctx.fillRect(0,0,128,128);
  return new THREE.CanvasTexture(c);
}

// ---------- 更新数据可视化 ----------
function getValue(city, metric, yearIdx) {
  return city[metric][yearIdx];
}
function maxValue(metric) {
  return Math.max(...CITIES.map(c => Math.max(...c[metric])));
}

function updateViz() {
  const maxV = maxValue(currentMetric);
  // 城市点大小 + 热力
  CITIES.forEach((city, i) => {
    const v = getValue(city, currentMetric, currentYearIdx);
    const ratio = v / maxV;
    const scale = 0.5 + ratio*2.0;
    cityMeshes[i].userData.halo.scale.set(0.15*scale, 0.15*scale, 1);
    heatSprites[i].material.opacity = 0.15 + ratio*0.6;
    heatSprites[i].scale.set(0.4 + ratio*0.9, 0.4 + ratio*0.9, 1);
    // 颜色按指标
    let col;
    if (currentMetric==='gdp') col = new THREE.Color(0xffaa44);
    else if (currentMetric==='population') col = new THREE.Color(0x44ff88);
    else col = new THREE.Color(0x44aaff);
    heatSprites[i].material.color = col;
  });
  // 航线流动动画偏移由 uniform 控制，这里仅改透明度
  const flightMax = maxValue('flights');
  routeMeshes.forEach((arc, k) => {
    const [i,j] = arc.line.userData.route;
    const f = (CITIES[i].flights[currentYearIdx] + CITIES[j].flights[currentYearIdx]) / (2*flightMax);
    arc.line.material.opacity = 0.3 + f*0.6;
  });
  document.getElementById('timeLabel').textContent = YEARS[currentYearIdx];
}
updateViz();

// ---------- 交互：射线 ----------
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const popup = document.getElementById('popup');
const flightTip = document.getElementById('flightTip');

function onMove(e) {
  mouse.x = (e.clientX/innerWidth)*2 - 1;
  mouse.y = -(e.clientY/innerHeight)*2 + 1;

  // 城市
  raycaster.setFromCamera(mouse, camera);
  const hits = raycaster.intersectObjects(cityMeshes, false);
  if (hits.length) {
    const idx = hits[0].object.userData.cityIndex;
    const c = CITIES[idx];
    popup.style.display = 'block';
    popup.style.left = (e.clientX+14)+'px';
    popup.style.top = (e.clientY+14)+'px';
    popup.innerHTML = `<h3>${c.name}</h3>
      <div class="row">GDP: ${c.gdp[currentYearIdx]} 十亿$</div>
      <div class="row">人口: ${c.population[currentYearIdx]} 百万</div>
      <div class="row">航班: ${c.flights[currentYearIdx]} /日</div>`;
  } else {
    popup.style.display = 'none';
  }

  // 航线（线段近似检测：用包围球）
  const lineHits = raycaster.intersectObjects(routeGroup.children, false);
  if (lineHits.length) {
    const ud = lineHits[0].object.userData;
    flightTip.style.display='block';
    flightTip.style.left = (e.clientX+14)+'px';
    flightTip.style.top = (e.clientY+14)+'px';
    flightTip.innerHTML = `${ud.from} ✈ ${ud.to}`;
  } else {
    flightTip.style.display='none';
  }
}

let clickPos = null;
renderer.domElement.addEventListener('pointerdown', e => clickPos = {x:e.clientX,y:e.clientY});
renderer.domElement.addEventListener('pointerup', e => {
  if (!clickPos) return;
  const moved = Math.hypot(e.clientX-clickPos.x, e.clientY-clickPos.y);
  clickPos = null;
  if (moved > 5) return; // 拖拽不算点击
  raycaster.setFromCamera(mouse, camera);
  const hits = raycaster.intersectObjects(cityMeshes, false);
  if (hits.length) {
    const idx = hits[0].object.userData.cityIndex;
    const c = CITIES[idx];
    popup.style.display='block';
    popup.style.left = (e.clientX+14)+'px';
    popup.style.top = (e.clientY+14)+'px';
    popup.innerHTML = `<h3>${c.name}</h3>
      <div class="row">GDP: ${c.gdp[currentYearIdx]} 十亿$</div>
      <div class="row">人口: ${c.population[currentYearIdx]} 百万</div>
      <div class="row">航班: ${c.flights[currentYearIdx]} /日</div>`;
  }
});
renderer.domElement.addEventListener('pointermove', onMove);

// ---------- 控制面板 ----------
document.getElementById('metricBtns').addEventListener('click', e => {
  if (e.target.tagName !== 'BUTTON') return;
  document.querySelectorAll('#metricBtns button').forEach(b=>b.classList.remove('active'));
  e.target.classList.add('active');
  currentMetric = e.target.dataset.metric;
  updateViz();
});

const slider = document.getElementById('timeSlider');
slider.addEventListener('input', e => {
  currentYearIdx = +e.target.value;
  updateViz();
});

let playing = false, playTimer = null;
document.getElementById('playBtn').addEventListener('click', () => {
  playing = !playing;
  const btn = document.getElementById('playBtn');
  btn.textContent = playing ? '⏸ 暂停' : '▶ 播放';
  if (playing) {
    playTimer = setInterval(() => {
      currentYearIdx = (currentYearIdx+1) % YEARS.length;
      slider.value = currentYearIdx;
      updateViz();
    }, 1200);
  } else clearInterval(playTimer);
});

// ---------- 动画循环 ----------
let flowOffset = 0;
function animate() {
  requestAnimationFrame(animate);
  // 地球自转
  earthGroup.rotation.y += 0.0008;
  // 太阳方向缓慢移动（模拟日夜）
  const t = performance.now()*0.00005;
  const sunDir = new THREE.Vector3(Math.cos(t)*5, 2, Math.sin(t)*5).normalize();
  earthMat.uniforms.sunDirection.value.copy(sunDir);
  atmosphereMat.uniforms.sunDirection.value.copy(sunDir);
  // 航线流动：通过修改材质 opacity 周期变化近似
  flowOffset = (flowOffset + 0.01) % 1;
  routeMeshes.forEach((arc, k) => {
    arc.line.material.opacity = 0.4 + 0.3*Math.sin(flowOffset*Math.PI*2 + k);
  });
  controls.update();
  renderer.render(scene, camera);
}
animate();

addEventListener('resize', () => {
  camera.aspect = innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
```

## 运行方式

1. 将上面两段代码合并保存为单个文件 `globe.html`（第二段紧接第一段 `makeGlowTexture` 函数体之后，注意第一段最后那行 `ctx.fillRect(0,0,64,64` 未闭合，需补上 `);` 然后接第二段开头）。
2. 由于使用了 ESM importmap 和跨域纹理，**必须通过本地 HTTP 服务器**打开，不能直接双击 `file://`：
   ```bash
   # 任选其一
   python3 -m http.server 8000
   # 或
   npx serve .
   ```
3. 浏览器访问 `http://localhost:8000/globe.html`。

## 测试说明

| 功能 | 测试方法 |
|------|----------|
| 地球纹理 | 打开页面应看到蓝色弹珠地球纹理 |
| 拖拽旋转 | 鼠标按住拖动，地球旋转 |
| 滚轮缩放 | 滚轮上下，相机远近变化 |
| 大气光晕 | 地球边缘有蓝色 Fresnel 光晕 |
| 日夜分界 | 地球上有明暗交界线，缓慢移动 |
| 城市灯光 | 夜晚区域可见城市灯光纹理增强 |
| 城市标记 | 12 个发光点 + 光晕 |
| 点击弹窗 | 点击城市点显示 GDP/人口/航班 |
| 悬停航线 | 鼠标移到弧线显示 "A ✈ B" |
| 指标切换 | 点击 GDP/人口/航班量，热力颜色和大小变化 |
| 时间轴 | 拖动滑块或点播放，年份 2000→2024 切换，数据更新 |
| 自转 | 不操作时地球缓慢自转 |
| 航线流动 | 弧线透明度周期性变化（流动效果） |

## 关于 Ask 模式

我当前处于 **Ask 模式**，无法直接创建文件。如果你想让我直接生成 `globe.html` 文件到工作区，请切换到 **Agent 模式**后再次发送指令，我会立即创建并验证可运行产物。

## 注意事项

- 地球纹理来自 unpkg 上的 `three-globe` 示例资源，需要联网加载；若网络不通可替换为本地纹理。
- 国家热力着色这里用"城市点辐射光斑"近似实现（真实国家级热力需要 TopoJSON + 边界贴图，体积较大）。如需严格按国家边界着色，可加载 `world.geo.json` 并用 `ShapeGeometry` 投影到球面，告诉我可进一步给出方案。
- 航线"流动动画"通过透明度周期变化实现；如需更精细的"光点沿弧线移动"效果，可在每条弧线上加一个沿 `curve.getPointAt(t)` 移动的小 sprite。