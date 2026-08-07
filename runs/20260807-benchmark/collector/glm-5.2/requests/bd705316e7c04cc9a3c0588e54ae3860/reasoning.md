用户想要一个隐形眼镜盒的3D模型，使用Three.js。需要：
1. L 和 R 醒目标识
2. 配有盒盖
3. 点击盒盖可以打开
4. 单文件HTML

让我设计一个出色的方案：

**视觉设计：**
- 隐形眼镜盒通常是小型塑料盒，圆形或椭圆形
- 两个独立的圆形容器，分别标记 L 和 R
- 一个整体的盒盖，可以掀开
- 颜色方面，我选择一个高级感的配色——不要默认蓝紫色
- 想用深青绿/薄荷绿 + 珊瑚橙的对比色，或者深色金属质感+亮色标识

让我选择：暗墨绿底色 + 珊瑚橙 L/R 标识 + 内部蓝色镜片液体感

**结构：**
- 底座：椭圆/圆角矩形底座，包含两个圆形凹槽
- 两个圆形容器：透明感，内部有蓝绿色液体模拟隐形眼镜浸泡液
- 盒盖：弧形顶盖，覆盖整个底座，通过铰链旋转打开
- L 和 R 标识：在底座和盒盖上都要清晰可见

**交互：**
- 点击盒盖打开/关闭，使用缓动动画
- 鼠标拖拽旋转视角
- 滚轮缩放
- 加载完成后有一个开盖的演示动画

**技术实现：**
- Three.js + OrbitControls
- 使用 importmap
- Raycaster 检测点击
- TWEEN 或自定义缓动实现开盖动画
- 添加柔和光照、环境反射
- 添加底部阴影/地面
- 粒子背景增加氛围

**配色方案：**
- 背景：深炭灰 #1a1d23 带渐变
- 眼镜盒主体：磨砂奶白 / 暖米色
- L/R 标识：珊瑚红 #ff5a4e 和青蓝 #2ec4b6
- 液体：淡青色
- 强调色：暖金色光源

让我开始编码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>隐形眼镜盒 · 3D 互动</title>
  ...
</head>
```

**详细几何构建：**

1. 底座：使用 RoundedBoxGeometry 或自定义形状，约 4x2.5x1.2 单位
2. 两个圆形凹槽：使用 CylinderGeometry，开口朝上
3. 镜片容器内部：小的 CylinderGeometry 带液体
4. 盒盖：使用一个椭圆柱或圆角矩形体，通过 Group 旋转
5. 铰链：细圆柱体连接

让我具体规划坐标：
- 底座尺寸：长 4，宽 2.4，高 1.0
- 两个凹槽中心：x = -1.0 和 x = 1.0，y = 顶部
- 凹槽半径：0.7，深度 0.6
- 盒盖：与底座同尺寸，但中空，覆盖在上面
- 铰链：在 z = -1.2（背面）

**L/R 标识实现：**
- 在底座正面顶部浮雕或贴字
- 在盒盖顶部也贴上 L 和 R
- 使用 CanvasTexture 生成文字纹理，或者用 TextGeometry

CanvasTexture 更简单且效果好，我用它。

**动画：**
- 点击盒盖时，盖子绕铰链轴旋转 0 → 110度
- 使用 easeOutCubic 缓动
- 同时镜片容器内的液体可以有微小的波纹效果

**额外细节：**
- 镜片本身：小的透明半球体，浮在液体表面
- 底座底部添加柔和阴影
- 一个微弱的环境光晕
- 鼠标悬停盒盖时有高亮提示

让我写代码：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
```

整个场景：
- 摄像机初始位置 (4, 3, 5)
- 看向 (0, 0.3, 0)
- OrbitControls，限制极角避免穿地

**光照：**
- 环境光 0.4
- 主方向光从右上方
- 暖色填充光
- 桌面反射光（hemisphere light）

**地面：**
- 一个大圆盘或平面，使用 ShadowMaterial 接收阴影
- 或者用一个带渐变的圆盘做底

**HUD/UI：**
- 顶部标题
- 底部说明文字"点击盒盖开合 · 拖拽旋转 · 滚轮缩放"
- 状态指示：开/关
- 右上角小信息卡

让我开始写完整代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LensCase · 隐形眼镜盒 3D 互动</title>
<style>
  /* 字体 */
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;500;700&family=DM+Mono:wght@400;500&display=swap');
  
  :root {
    --bg-0: #0e1116;
    --bg-1: #161b22;
    --fg: #e8e6e1;
    --muted: #8b8f97;
    --accent: #ff6b5b;
    --accent-2: #2dd4bf;
    --border: rgba(255,255,255,0.08);
    --card: rgba(22,27,34,0.7);
  }
  
  * { box-sizing: border-box; margin: 0; padding: 0; }
  
  html, body {
    height: 100%;
    background: radial-gradient(ellipse at 30% 20%, #1a1f29 0%, #0a0d12 70%);
    color: var(--fg);
    font-family: 'Space Grotesk', sans-serif;
    overflow: hidden;
  }
  
  #app {
    position: relative;
    width: 100vw;
    height: 100vh;
  }
  
  canvas { display: block; }
  
  /* 顶部标题栏 */
  .header {
    position: absolute;
    top: 0; left: 0; right: 0;
    padding: 24px 32px;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    z-index: 10;
    pointer-events: none;
  }
  
  .brand {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  
  .brand .tag {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.3em;
    color: var(--muted);
    text-transform: uppercase;
  }
  
  .brand h1 {
    font-size: 28px;
    font-weight: 700;
    letter-spacing: -0.02em;
    background: linear-gradient(120deg, #fff 0%, #ff6b5b 60%, #ffb088 100%);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
  }
  
  .brand .sub {
    font-size: 13px;
    color: var(--muted);
    font-weight: 300;
  }
  
  /* 右侧状态卡 */
  .status-card {
    background: var(--card);
    border: 1px solid var(--border);
    backdrop-filter: blur(20px);
    border-radius: 14px;
    padding: 14px 18px;
    pointer-events: auto;
    min-width: 200px;
  }
  
  .status-card .label {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.25em;
    color: var(--muted);
    text-transform: uppercase;
    margin-bottom: 6px;
  }
  
  .status-card .value {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 16px;
    font-weight: 500;
  }
  
  .status-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: var(--accent-2);
    box-shadow: 0 0 12px var(--accent-2);
    transition: all 0.3s;
  }
  
  .status-dot.open {
    background: var(--accent);
    box-shadow: 0 0 12px var(--accent);
  }
  
  /* 底部控制提示 */
  .footer {
    position: absolute;
    bottom: 0; left: 0; right: 0;
    padding: 24px 32px;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    z-index: 10;
    pointer-events: none;
  }
  
  .hint {
    background: var(--card);
    border: 1px solid var(--border);
    backdrop-filter: blur(20px);
    border-radius: 12px;
    padding: 12px 16px;
    pointer-events: auto;
    display: flex;
    gap: 18px;
    flex-wrap: wrap;
  }
  
  .hint-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    color: var(--muted);
  }
  
  .hint-item .key {
    font-family: 'DM Mono', monospace;
    background: rgba(255,255,255,0.06);
    border: 1px solid var(--border);
    border-radius: 5px;
    padding: 2px 7px;
    font-size: 11px;
    color: var(--fg);
  }
  
  .reset-btn {
    pointer-events: auto;
    background: var(--card);
    border: 1px solid var(--border);
    backdrop-filter: blur(20px);
    color: var(--fg);
    padding: 10px 18px;
    border-radius: 10px;
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    letter-spacing: 0.1em;
    cursor: pointer;
    transition: all 0.25s;
    text-transform: uppercase;
  }
  
  .reset-btn:hover {
    background: rgba(255,107,91,0.15);
    border-color: var(--accent);
    color: var(--accent);
  }
  
  /* 加载 */
  .loader {
    position: absolute;
    inset: 0;
    background: var(--bg-0);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    transition: opacity 0.6s;
  }
  
  .loader.hidden {
    opacity: 0;
    pointer-events: none;
  }
  
  .loader-content {
    text-align: center;
  }
  
  .loader-ring {
    width: 48px;
    height: 48px;
    border: 2px solid rgba(255,255,255,0.08);
    border-top-color: var(--accent);
    border-radius: 50%;
    margin: 0 auto 16px;
    animation: spin 1s linear infinite;
  }
  
  @keyframes spin {
    to { transform: rotate(360deg); }
  }
  
  .loader-text {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.3em;
    color: var(--muted);
    text-transform: uppercase;
  }
  
  /* 装饰角标 */
  .corner-deco {
    position: absolute;
    width: 60px; height: 60px;
    border: 1px solid var(--border);
    pointer-events: none;
    opacity: 0.5;
  }
  .corner-deco.tl { top: 12px; left: 12px; border-right: none; border-bottom: none; }
  .corner-deco.tr { top: 12px; right: 12px; border-left: none; border-bottom: none; }
  .corner-deco.bl { bottom: 12px; left: 12px; border-right: none; border-top: none; }
  .corner-deco.br { bottom: 12px; right: 12px; border-left: none; border-top: none; }
  
  /* 响应式 */
  @media (max-width: 640px) {
    .header { padding: 16px; flex-direction: column; gap: 12px; }
    .footer { padding: 16px; flex-direction: column-reverse; gap: 10px; align-items: stretch; }
    .brand h1 { font-size: 22px; }
    .hint { gap: 10px; }
  }
</style>
</head>
<body>
<div id="app">
  <!-- 角标 -->
  <div class="corner-deco tl"></div>
  <div class="corner-deco tr"></div>
  <div class="corner-deco bl"></div>
  <div class="corner-deco br"></div>
  
  <!-- 顶部 -->
  <header class="header">
    <div class="brand">
      <span class="tag">3D Interactive</span>
      <h1>LENS CASE</h1>
      <span class="sub">隐形眼镜盒 · 左右分明</span>
    </div>
    <div class="status-card">
      <div class="label">Case Status</div>
      <div class="value">
        <span class="status-dot" id="statusDot"></span>
        <span id="statusText">已合盖</span>
      </div>
    </div>
  </header>
  
  <!-- 底部 -->
  <footer class="footer">
    <div class="hint">
      <div class="hint-item"><span class="key">CLICK</span>点击盒盖开合</div>
      <div class="hint-item"><span class="key">DRAG</span>旋转视角</div>
      <div class="hint-item"><span class="key">SCROLL</span>缩放</div>
    </div>
    <button class="reset-btn" id="resetBtn">重置视角</button>
  </footer>
  
  <!-- 加载 -->
  <div class="loader" id="loader">
    <div class="loader-content">
      <div class="loader-ring"></div>
      <div class="loader-text">Loading Scene</div>
    </div>
  </div>
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
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';

// ============== 场景基础 ==============
const scene = new THREE.Scene();
scene.background = null;

const camera = new THREE.PerspectiveCamera(40, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(4.5, 3.8, 5.5);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.1;
renderer.outputColorSpace = THREE.SRGBColorSpace;
document.getElementById('app').appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.06;
controls.minDistance = 3;
controls.maxDistance = 12;
controls.maxPolarAngle = Math.PI * 0.48;
controls.target.set(0, 0.3, 0);

// ============== 光照 ==============
const hemi = new THREE.HemisphereLight(0xfff5e8, 0x1a2030, 0.55);
scene.add(hemi);

const keyLight = new THREE.DirectionalLight(0xfff0d8, 1.6);
keyLight.position.set(4, 7, 4);
keyLight.castShadow = true;
keyLight.shadow.mapSize.set(2048, 2048);
keyLight.shadow.camera.left = -5;
keyLight.shadow.camera.right = 5;
keyLight.shadow.camera.top = 5;
keyLight.shadow.camera.bottom = -5;
keyLight.shadow.camera.near = 0.5;
keyLight.shadow.camera.far = 20;
keyLight.shadow.bias = -0.0005;
keyLight.shadow.radius = 6;
scene.add(keyLight);

const fillLight = new THREE.DirectionalLight(0x6088ff, 0.4);
fillLight.position.set(-5, 3, -3);
scene.add(fillLight);

const rimLight = new THREE.PointLight(0xff7755, 0.8, 10, 2);
rimLight.position.set(-2, 2, -3);
scene.add(rimLight);

// ============== 桌面 ==============
const tableGeo = new THREE.CircleGeometry(8, 64);
const tableMat = new THREE.MeshStandardMaterial({
  color: 0x12161d,
  roughness: 0.85,
  metalness: 0.2
});
const table = new THREE.Mesh(tableGeo, tableMat);
table.rotation.x = -Math.PI/2;
table.position.y = -0.5;
table.receiveShadow = true;
scene.add(table);

// 桌面上有一圈微光
const glowGeo = new THREE.RingGeometry(1.8, 2.5, 64);
const glowMat = new THREE.MeshBasicMaterial({
  color: 0xff6b5b,
  transparent: true,
  opacity: 0.08,
  side: THREE.DoubleSide
});
const glow = new THREE.Mesh(glowGeo, glowMat);
glow.rotation.x = -Math.PI/2;
glow.position.y = -0.49;
scene.add(glow);

// ============== 隐形眼镜盒 ==============
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// 材质
const bodyMat = new THREE.MeshPhysicalMaterial({
  color: 0xf2ece2,
  roughness: 0.35,
  metalness: 0.05,
  clearcoat: 0.5,
  clearcoatRoughness: 0.2
});

const accentMatL = new THREE.MeshPhysicalMaterial({
  color: 0xff6b5b,
  roughness: 0.3,
  metalness: 0.1,
  clearcoat: 0.6
});

const accentMatR = new THREE.MeshPhysicalMaterial({
  color: 0x2dd4bf,
  roughness: 0.3,
  metalness: 0.1,
  clearcoat: 0.6
});

const liquidMat = new THREE.MeshPhysicalMaterial({
  color: 0x9bd5ff,
  roughness: 0.05,
  metalness: 0,
  transmission: 0.7,
  transparent: true,
  opacity: 0.55,
  thickness: 0.5,
  ior: 1.33
});

const lensMat = new THREE.MeshPhysicalMaterial({
  color: 0xaaccff,
  roughness: 0.05,
  metalness: 0,
  transmission: 0.95,
  transparent: true,
  opacity: 0.4,
  thickness: 0.2,
  ior: 1.4,
  side: THREE.DoubleSide
});

// 底座
const baseGeo = new RoundedBoxGeometry(4, 0.9, 2.4, 8, 0.25);
const base = new THREE.Mesh(baseGeo, bodyMat);
base.position.y = 0;
base.castShadow = true;
base.receiveShadow = true;
caseGroup.add(base);

// 底座凹陷：两个圆形凹槽
function makeWell(x, color) {
  const wellGroup = new THREE.Group();
  wellGroup.position.set(x, 0.45, 0);
  
  // 凹槽内壁
  const innerRadius = 0.7;
  const innerDepth = 0.55;
  const innerWall = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius, innerRadius, innerDepth, 48, 1, true),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.4, metalness: 0.1, side: THREE.BackSide })
  );
  innerWall.position.y = -0.05;
  wellGroup.add(innerWall);
  
  // 内底（带颜色的圆盘）
  const innerBottom = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius, innerRadius, 0.05, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.5, metalness: 0.1 })
  );
  innerBottom.position.y = -0.275;
  wellGroup.add(innerBottom);
  
  // 液体
  const liquid = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius - 0.02, innerRadius - 0.02, 0.35, 48),
    liquidMat
  );
  liquid.position.y = -0.15;
  wellGroup.add(liquid);
  
  // 镜片（薄圆盘）
  const lens = new THREE.Mesh(
    new THREE.CylinderGeometry(0.55, 0.55, 0.04, 48),
    lensMat
  );
  lens.position.y = -0.05;
  lens.rotation.x = 0.1;
  wellGroup.add(lens);
  
  // 顶部边缘环（加强感）
  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(innerRadius, 0.04, 16, 48),
    new THREE.MeshStandardMaterial({ color: 0xe0d8c8, roughness: 0.3, metalness: 0.2 })
  );
  ring.rotation.x = Math.PI/2;
  wellGroup.add(ring);
  
  return wellGroup;
}

const wellL = makeWell(-1.0, 0xff6b5b);
const wellR = makeWell(1.0, 0x2dd4bf);
caseGroup.add(wellL, wellR);

// L R 文字纹理
function makeLetterTexture(letter, color) {
  const c = document.createElement('canvas');
  c.width = 256; c.height = 256;
  const ctx = c.getContext('2d');
  // 透明背景
  ctx.clearRect(0,0,256,256);
  ctx.fillStyle = color;
  ctx.font = 'bold 200px "Space Grotesk", Arial, sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, 128, 138);
  const tex = new THREE.CanvasTexture(c);
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}

const texL = makeLetterTexture('L', '#ff6b5b');
const texR = makeLetterTexture('R', '#2dd4bf');

// 在底座顶部贴 L R 标签（小贴片）
function makeLabel(x, texture, color) {
  const labelGroup = new THREE.Group();
  // 底层圆盘
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.45, 0.45, 0.02, 48),
    new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.6, metalness: 0.3 })
  );
  disc.position.set(x, 0.461, 0.85);
  labelGroup.add(disc);
  
  // 文字面（朝上的平面）
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.7, 0.7),
    new THREE.MeshBasicMaterial({ map: texture, transparent: true })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.472, 0.85);
  labelGroup.add(plane);
  
  return labelGroup;
}

caseGroup.add(makeLabel(-1.0, texL, 0xff6b5b));
caseGroup.add(makeLabel(1.0, texR, 0x2dd4bf));

// ============== 盒盖 ==============
// 盒盖通过铰链旋转打开
const lidPivot = new THREE.Group();
lidPivot.position.set(0, 0.45, -1.2); // 铰链轴位置
caseGroup.add(lidPivot);

const lidGroup = new THREE.Group();
lidPivot.add(lidGroup);

// 盒盖主体（向上偏移，使其覆盖底座）
const lidGeo = new RoundedBoxGeometry(4, 0.5, 2.4, 8, 0.22);
const lid = new THREE.Mesh(lidGeo, bodyMat);
lid.position.set(0, 0.25, 1.2);
lid.castShadow = true;
lidGroup.add(lid);

// 盒盖顶部装饰条
const stripeGeo = new RoundedBoxGeometry(3.6, 0.08, 0.2, 4, 0.04);
const stripe = new THREE.Mesh(
  stripeGeo,
  new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.4, metalness: 0.4 })
);
stripe.position.set(0, 0.52, 1.2);
lidGroup.add(stripe);

// 盒盖顶部 L R 文字
function makeLidLabel(x, texture, color) {
  const labelGroup = new THREE.Group();
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.4, 0.4, 0.02, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.4, metalness: 0.2 })
  );
  disc.position.set(x, 0.515, 1.2);
  labelGroup.add(disc);
  
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.6, 0.6),
    new THREE.MeshBasicMaterial({ map: texture, transparent: true })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.527, 1.2);
  labelGroup.add(plane);
  
  return labelGroup;
}

// 重新生成文字纹理（白色版本，在彩色圆盘上）
function makeLetterTextureWhite(letter) {
  const c = document.createElement('canvas');
  c.width = 256; c.height = 256;
  const ctx = c.getContext('2d');
  ctx.clearRect(0,0,256,256);
  ctx.fillStyle = '#ffffff';
  ctx.font = 'bold 200px "Space Grotesk", Arial, sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, 128, 138);
  const tex = new THREE.CanvasTexture(c);
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}

const texLwhite = makeLetterTextureWhite('L');
const texRwhite = makeLetterTextureWhite('R');

lidGroup.add(makeLidLabel(-1.0, texLwhite, 0xff6b5b));
lidGroup.add(makeLidLabel(1.0, texRwhite, 0x2dd4bf));

// 铰链（可见的圆柱）
const hingeGeo = new THREE.CylinderGeometry(0.08, 0.08, 2.6, 16);
const hingeMat = new THREE.MeshStandardMaterial({ color: 0x2a2e36, roughness: 0.4, metalness: 0.7 });
const hinge = new THREE.Mesh(hingeGeo, hingeMat);
hinge.rotation.z = Math.PI/2;
hinge.position.set(0, 0, -1.2);
caseGroup.add(hinge);

// 让 lid 接收 raycast，但需要在 lidGroup 上做检测
// 给 lid 设置 userData 标记
lid.userData.clickable = true;

// ============== 点击交互 ==============
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let isOpen = false;
let animating = false;
let lidAngle = 0; // 当前角度（弧度）
let targetAngle = 0;

const statusDot = document.getElementById('statusDot');
const statusText = document.getElementById('statusText');

function updateStatus(open) {
  if (open) {
    statusDot.classList.add('open');
    statusText.textContent = '已开盖';
  } else {
    statusDot.classList.remove('open');
    statusText.textContent = '已合盖';
  }
}

function toggleLid() {
  if (animating) return;
  isOpen = !isOpen;
  targetAngle = isOpen ? Math.PI * 0.62 : 0; // ~112度
  animating = true;
  updateStatus(isOpen);
}

renderer.domElement.addEventListener('click', (e) => {
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  // 检测 lid 以及 lidGroup 下的所有子物体
  const intersects = raycaster.intersectObjects([lid, ...lidGroup.children], true);
  if (intersects.length > 0) {
    toggleLid();
  }
});

// 鼠标悬停效果
let hovered = false;
renderer.domElement.addEventListener('mousemove', (e) => {
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects([lid, ...lidGroup.children], true);
  const nowHovered = intersects.length > 0;
  if (nowHovered !== hovered) {
    hovered = nowHovered;
    renderer.domElement.style.cursor = hovered ? 'pointer' : 'grab';
  }
});

renderer.domElement.style.cursor = 'grab';
renderer.domElement.addEventListener('mousedown', () => {
  renderer.domElement.style.cursor = 'grabbing';
});
renderer.domElement.addEventListener('mouseup', () => {
  renderer.domElement.style.cursor = hovered ? 'pointer' : 'grab';
});

// ============== 重置视角 ==============
document.getElementById('resetBtn').addEventListener('click', () => {
  // 用动画方式回到默认视角
  const startPos = camera.position.clone();
  const startTarget = controls.target.clone();
  const endPos = new THREE.Vector3(4.5, 3.8, 5.5);
  const endTarget = new THREE.Vector3(0, 0.3, 0);
  let t = 0;
  const dur = 800;
  const t0 = performance.now();
  function step() {
    const now = performance.now();
    t = Math.min(1, (now - t0)/dur);
    const e = 1 - Math.pow(1-t, 3);
    camera.position.lerpVectors(startPos, endPos, e);
    controls.target.lerpVectors(startTarget, endTarget, e);
    controls.update();
    if (t < 1) requestAnimationFrame(step);
  }
  step();
});

// ============== 渲染循环 ==============
function easeOutCubic(t) {
  return 1 - Math.pow(1 - t, 3);
}

let lastTime = performance.now();
function animate() {
  requestAnimationFrame(animate);
  const now = performance.now();
  const dt = (now - lastTime) / 1000;
  lastTime = now;
  
  // 盖子动画
  if (animating) {
    const diff = targetAngle - lidAngle;
    if (Math.abs(diff) < 0.001) {
      lidAngle = targetAngle;
      animating = false;
    } else {
      // 简单缓动
      lidAngle += diff * Math.min(1, dt * 6);
    }
    lidPivot.rotation.x = -lidAngle; // 负号让它向后翻开
  }
  
  // 桌面光晕呼吸
  glowMat.opacity = 0.06 + Math.sin(now * 0.001) * 0.02;
  
  // 微光点光源呼吸
  rimLight.intensity = 0.7 + Math.sin(now * 0.0012) * 0.15;
  
  controls.update();
  renderer.render(scene, camera);
}

// ============== 启动 ==============
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// 隐藏加载器
setTimeout(() => {
  document.getElementById('loader').classList.add('hidden');
}, 600);

// 进场动画：盒子从下方升起
caseGroup.position.y = -1;
caseGroup.scale.setScalar(0.8);
const introT0 = performance.now();
function intro() {
  const t = Math.min(1, (performance.now() - introT0) / 1200);
  const e = easeOutCubic(t);
  caseGroup.position.y = -1 + e * 1;
  caseGroup.scale.setScalar(0.8 + e * 0.2);
  if (t < 1) requestAnimationFrame(intro);
}
intro();

animate();
</script>
</body>
</html>
```

让我审视一下设计：

1. ✅ 隐形眼镜盒 3D 模型 - 有
2. ✅ L 和 R 醒目标识 - 在底座和盒盖上都有
3. ✅ 盒盖 - 有
4. ✅ 点击盒盖打开 - 有
5. ✅ 单文件 - 是

让我再检查几个潜在问题：

1. `RoundedBoxGeometry` - 这是 addons 中的，需要确认路径正确
   - 路径 `three/addons/geometries/RoundedBoxGeometry.js` 是对的

2. 盒盖旋转方向 - lidPivot 在 (0, 0.45, -1.2)，绕 X 轴旋转
   - 负角度向后翻开 - 应该对
   - 但等等，lid 自身在 lidGroup 内位置 z=1.2，相对于 pivot
   - 当 pivot 绕 X 轴旋转时，lid 会以 pivot 为圆心摆动
   - 这就是铰链效果，正确

3. raycaster 检测 lid 时，由于 lid 是 lidGroup 的子，而 lidGroup 又是 lidPivot 的子，所以 lid 的世界变换会随着 pivot 旋转变化。这没问题，raycaster 会处理。

4. 点击区域 - 当盒盖关闭时，点击 lid 顶部很合理。当打开时，盒盖翻到背面，可能就不太能点到了，但用户可以再点击底座上的内容... 嗯，打开后用户还想关闭，这时盒盖在背面，怎么办？

让我处理：当盒盖打开时，应该让用户也能通过其他方式关闭。或者在打开状态下，盒盖仍然在视野内（因为绕铰链翻开后，盒盖会在背面朝上）。

实际上，盒盖打开 112度后，它会朝上后方，从摄像机角度看仍然可见（特别是从上方看）。用户应该能点中。

但更稳妥的做法是：扩大点击检测范围。让 base 也参与检测，但 base 的点击在盒盖打开时关闭它，盒盖关闭时打开它。

或者更简单：用户点击 lid 即可切换状态。打开状态下，lid 在 z 负方向上方，从摄像机 (4.5, 3.8, 5.5) 看向 (0, 0.3, 0)，lid 应该是可见的，只是角度不同。

让我再思考一下：lidPivot 在 (0, 0.45, -1.2)，旋转 112度，lid 中心初始在 (0, 0.45+0.25, -1.2+1.2) = (0, 0.7, 0)。
旋转后 lid 中心位置：
- 相对 pivot: (0, 0.25, 1.2)
- 绕 X 轴旋转 -112度：
  - y' = y*cos(θ) - z*sin(θ) = 0.25*cos(-112°) - 1.2*sin(-112°) = 0.25*(-0.375) - 1.2*(-0.927) = -0.094 + 1.112 = 1.018
  - z' = y*sin(θ) + z*cos(θ) = 0.25*sin(-112°) + 1.2*cos(-112°) = 0.25*(-0.927) + 1.2*(-0.375) = -0.232 - 0.45 = -0.682
- 加上 pivot 位置: (0, 0.45+1.018, -1.2-0.682) = (0, 1.47, -1.88)

所以 lid 在打开后会升到 y=1.47，z=-1.88，仍然在视野中。从摄像机 (4.5, 3.8, 5.5) 看，lid 是可见的，可以点击。

好，逻辑没问题。

5. 加载器隐藏时机 - 用 setTimeout 600ms，不够严谨。但因为我没有真正的资源加载，所以可以接受。其实可以更早隐藏。

6. 字体 - 我用了 Space Grotesk 和 DM Mono，这些不算太"过度使用"。

7. 色彩 - 珊瑚红 #ff6b5b + 青蓝 #2dd4bf，奶白色主体，深色桌面 - 比默认蓝紫好。

8. 装饰元素 - 角标、桌面光晕呼吸、rim light 呼吸、进场动画 - 有"signature moment"。

让我再加一些细节：
- 镜片表面的高光
- 液体表面波纹（用着色器或简单的时间偏移）
- 鼠标悬停 lid 时有轻微高亮

实际上不需要太多，目前已经够丰富了。我主要担心：
- 镜片是 CylinderGeometry，不太像真实镜片。可以改成扁球体或部分球面。
- 但小尺寸下看不太清，可以接受。

让我把镜片改成一个低高度的圆盘 + 半球，更像真实隐形眼镜：
- 镜片：半球体扁圆，类似 (SphereGeometry, phiStart/thetaLength控制) 或 (CircleGeometry)
- 实际上简化版用很扁的 CylinderGeometry 也行

考虑到代码长度，我就保持扁圆柱了。

还有一个问题：raycaster intersect 检测时，我传入了 `[lid, ...lidGroup.children]`，但 lid 本身就是 lidGroup.children 之一，所以重复了。改成 `[lid]` 或 `lidGroup.children` 都行。我改用更稳的方式：

```js
const clickableObjects = [];
lidGroup.traverse(obj => {
  if (obj.isMesh) clickableObjects.push(obj);
});
```

或者直接检测整个 caseGroup 的子树，但这样底座也会被点中。我应该让 lid 及其装饰都是可点击的，而底座点击不应触发开盖。

让我重新思考：
- 我希望点击 lid 任何部分都能切换开关
- 底座点击不触发
- 这意味着只有 lidGroup 的所有子 mesh 是 clickable

修改：
```js
const clickableMeshes = [];
lidGroup.traverse(obj => {
  if (obj.isMesh) clickableMeshes.push(obj);
});

// 在点击事件中
const intersects = raycaster.intersectObjects(clickableMeshes, false);
```

但 `intersectObjects` 第二个参数 `recursive` 默认是 true，所以传 `clickableMeshes` 数组就够了。但其实直接传 `[lidGroup]` 然后让 recursive=true 也可以：

```js
const intersects = raycaster.intersectObject(lidGroup, true);
```

这样最简洁。让我用这个。

但 lidGroup 是 lidPivot 的子，lidPivot 旋转会带动 lidGroup，从而 lid 的世界位置变化。raycast 应该能正确处理。

让我再考虑细节：

实际上还有一个问题，我让 base.position.y = 0，但 base 几何高度是 0.9，所以 base 顶部在 y=0.45。well 的位置 (x, 0.45, 0)，但 well 内的物体位置是相对 wellGroup 的，所以 well 内的内底在 y=0.45-0.275=0.175，这是世界 y 坐标。但 base 顶部是 0.45，所以内底在 base 内部。OK。

液体在 y=0.45-0.15=0.3，这是液体中心，液体高度 0.35，所以液体顶部在 0.3+0.175=0.475，超过了 base 顶部 0.45。这是合理的，因为液体应该几乎填满凹槽。

镜片在 y=0.45-0.05=0.4，刚好在液体表面附近。

但等等，内壁高度 0.55，y=-0.05，所以内壁顶部在 y=0.45-0.05+0.275=0.675，底部在 y=0.45-0.05-0.275=0.125。base 顶部是 0.45，所以内壁顶部超出 base 0.225。这意味着凹槽的"墙"伸出了 base 表面。

这不对！凹槽应该内嵌在 base 中，开口在 base 顶部表面。

让我重新计算：
- base 高度 0.9，中心 y=0，顶部 y=0.45，底部 y=-0.45
- 凹槽开口应该在 y=0.45（base 顶部）
- 凹槽深度比如 0.6，所以凹槽底部在 y=0.45-0.6=-0.15

我想让凹槽看起来是从 base 顶部凹陷进去：
- 内壁圆柱，开口在 y=0.45，向下延伸到 y=-0.15
- 内壁中心 y = (0.45 + -0.15)/2 = 0.15，高度 0.6
- 内底在 y=-0.15
- 液体在内壁中，比如高度 0.4，中心 y = 0.45-0.2 = 0.25，底部 y=-0.15+0.05 = -0.1（离底部0.05）

我重新设计 well：

```js
function makeWell(x, color) {
  const wellGroup = new THREE.Group();
  wellGroup.position.set(x, 0, 0);  // 中心放在 base 中心高度
  
  const innerRadius = 0.7;
  const wellTopY = 0.45;   // 凹槽开口
  const wellDepth = 0.6;
  const wellBottomY = wellTopY - wellDepth; // -0.15
  const wellCenterY = (wellTopY + wellBottomY) / 2; // 0.15
  
  // 内壁
  const innerWall = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius, innerRadius, wellDepth, 48, 1, true),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.4, metalness: 0.1, side: THREE.BackSide })
  );
  innerWall.position.y = wellCenterY;
  wellGroup.add(innerWall);
  
  // 内底
  const innerBottom = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius - 0.01, innerRadius - 0.01, 0.02, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.5, metalness: 0.1 })
  );
  innerBottom.position.y = wellBottomY + 0.01;
  wellGroup.add(innerBottom);
  
  // 液体
  const liquidHeight = 0.4;
  const liquid = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius - 0.025, innerRadius - 0.025, liquidHeight, 48),
    liquidMat
  );
  liquid.position.y = wellBottomY + liquidHeight/2 + 0.02;
  wellGroup.add(liquid);
  
  // 镜片（扁圆盘）
  const lens = new THREE.Mesh(
    new THREE.CylinderGeometry(0.55, 0.55, 0.04, 48),
    lensMat
  );
  lens.position.y = wellBottomY + 0.45; // 漂在液体表面附近
  wellGroup.add(lens);
  
  // 顶部边缘环
  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(innerRadius, 0.04, 16, 48),
    new THREE.MeshStandardMaterial({ color: 0xe0d8c8, roughness: 0.3, metalness: 0.2 })
  );
  ring.rotation.x = Math.PI/2;
  ring.position.y = wellTopY;
  wellGroup.add(ring);
  
  return wellGroup;
}
```

更新标签位置：标签在底座顶部，紧贴表面。
```js
function makeLabel(x, texture, color) {
  // 在两个 well 之间，base 顶部表面
  // 我让标签放在 base 顶部前侧（z=0.85）
  const labelGroup = new THREE.Group();
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.32, 0.32, 0.015, 48),
    new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.6, metalness: 0.3 })
  );
  disc.position.set(x, 0.46, 0.85); // x=-1 或 1
  // 等等，标签应该和 well 对齐，因为 well 在 x=-1 和 1
  // 但 well 是圆形凹槽，如果标签也在 x=-1 和 1，会和凹槽重叠！
}
```

哦，这是个问题。标签 L 和 R 应该和凹槽对应，但凹槽占据 x=-1 和 1。所以标签必须放在凹槽旁边或者凹槽内。

让我重新规划布局：
- base 长度 4，宽度 2.4
- 两个凹槽中心在 x=-1.0 和 x=1.0，每个半径 0.7
- 标签 L 应该紧贴 R 凹槽左边或左边凹槽上方？

更好的设计：标签放在 base 顶部前缘，比如 z=0.8（base 前缘是 z=1.2），距离凹槽中心 x=-1 和 1 不变。

也就是 L 标签在 (x=-1, z=0.8) 但这位置在凹槽正前方。
看俯视图：凹槽中心 (x=-1, z=0)，半径 0.7，所以凹槽覆盖 z 从 -0.7 到 0.7。标签在 z=0.8 就在凹槽正前方了，没问题！

调整：
- base 宽度（z方向）2.4，z 从 -1.2 到 1.2
- 凹槽半径 0.7，z 范围 -0.7 到 0.7
- 标签半径 0.32，放在 z=0.95，没问题，离凹槽边缘 0.25 距离

让我让标签更靠前一点，z=0.95。

实际上更好的设计是让 L R 标签直接印在凹槽的前侧弧形外缘，紧贴凹槽，就像真实眼镜盒那样。我可以做成弧形或简单的圆盘+文字。

简单方案：用两个小圆盘贴在 base 顶部，分别在 L 凹槽和 R 凹槽的前方：
- L 标签 (x=-1, y=0.46, z=0.95)
- R 标签 (x=1, y=0.46, z=0.95)

但 L 在左侧凹槽前方，R 在右侧凹槽前方，从俯视图看 L 在左下，R 在右下。这样在 lid 上也对应位置贴上 L 和 R。

我觉得这样就 OK。让我把代码完善。

继续盒盖部分：
盒盖覆盖整个底座，所以盒盖尺寸也是 4 x 0.5 x 2.4。盒盖中心在 (0, base顶+0.25, 0) = (0, 0.7, 0) 但要通过 pivot 偏移。

我之前设置：
- lidPivot 在 (0, 0.45, -1.2) — 这是 base 后缘顶部
- lid 在 lidGroup 内，lidGroup 是 lidPivot 的子
- lid 位置 (0, 0.25, 1.2) — 相对 pivot，即在 (0, 0.7, 0) 世界坐标
- lid 几何尺寸 4 x 0.5 x 2.4，所以 lid 顶部 y=0.95，底部 y=0.45
- 当 lid 关闭时，lid 底部刚好接触 base 顶部

好，让我把所有这些整合到最终代码：

实际上我重新审视一下，我让 lid 在 lidGroup 内，但 lidGroup 是 lidPivot 的子。lidGroup 的 position 默认 (0,0,0)，rotation 默认 0。所以 lid 的局部坐标 (0, 0.25, 1.2) 经过 lidGroup (0,0,0) 再经过 lidPivot (0, 0.45, -1.2)，世界坐标 (0, 0.7, 0)。

旋转 lidPivot.rotation.x = -θ 时，lid 绕 pivot 旋转。

看起来 OK。

让我重新审视代码并写出最终版本。

我还想到一个改进：增加镜片表面的小高光斑点，让镜片看起来更真实。但这会增加复杂度，先不做。

另一个改进：底座侧面加一些纹理装饰（品牌名等）。可以在 base 侧面贴上文字。但 CanvasTexture + Plane 已够。

最终再加一个细节：让 base 上有一些防滑的橡胶垫脚（在底部），但用户从上方看不到，没必要。

好，让我开始写最终版本。

让我看看代码总长度，应该不会太长。最终版本：

我注意到一个问题：我用了 `lidGroup.children` 来传给 raycaster，但应该用 lidGroup 本身让 recursive=true：

```js
const intersects = raycaster.intersectObject(lidGroup, true);
```

这是更简洁的方法。我会用这个。

让我也考虑：当 lid 打开后，用户能否再点击 lid 关闭？lid 打开后位置在 (0, ~1.47, ~-1.88) — 仍然在摄像机视野中，从 (4.5, 3.8, 5.5) 看向 (0, 0.3, 0)，应该能看到 lid 的内表面（朝向摄像机/上方）。raycast 仍然能命中。所以可以点击关闭。

OK，开始写最终代码。

让我再思考一下视觉表现：
- 主体颜色：奶白色 #f2ece2
- L 标签：珊瑚红 #ff6b5b
- R 标签：青蓝 #2dd4bf
- 液体：淡青色，半透明
- 镜片：透明带蓝色调
- 桌面：深炭灰 #12161d
- 背景径向渐变

光照：
- HemisphereLight 暖色顶光 + 冷色底光
- KeyLight 暖白主光，从右上方
- FillLight 冷蓝填充
- RimLight 暖红边缘光，背面

这些组合应该产生一个有质感的画面。

字体：
- Space Grotesk 显示字体
- DM Mono 用于小标签

颜色变量 CSS 中用珊瑚红和青蓝作为强调色。

OK 我现在写最终的完整代码。

还有一点：我应该让盒盖打开时，里面的镜片和液体有一个高光增强效果（因为光线进入）。这可以通过增加一个 PointLight 在凹槽上方实现，或者打开时增加 lid 的 emissive。简单点处理：直接让液体材质在打开时有更亮的高光，因为光源没有被遮挡。

事实上我的 keyLight 在 (4, 7, 4) 上方，所以 lid 关闭时会挡住光线进入凹槽，打开时光线进入，自然就会变亮。Three.js 的光照模型会自动处理。

好，开始最终代码：

最后再思考下：用户问的是"隐形眼镜盒"，所以是装隐形眼镜的小盒子。我的设计是合理的：一个长方形扁盒，两个圆形凹槽，标识 L 和 R，配盒盖。

一些最后的调整：
- 让 lid 厚度从 0.5 减到 0.35，更薄更精致
- lid 中心 y 调整为 0.45+0.175 = 0.625，相对 pivot y=0.175
- 顶部装饰条 y 调整

实际上 0.5 厚度还可以，看起来有质感。我保持。

我打算把 lid 的位置从相对 pivot 的 (0, 0.25, 1.2) 改为 (0, 0.25, 1.2)，让 lid 底部刚好在 base 顶部 (y=0.45)，lid 顶部 y=0.7。这样合盖时 lid 覆盖 base 顶部。

实际查看 lid 的几何：lidGeo = RoundedBoxGeometry(4, 0.5, 2.4)，几何中心在原点。lid mesh 在 lidGroup 内 position (0, 0.25, 1.2)，意味着 lid 的几何中心相对 pivot 在 y=0.25, z=1.2，lid 顶部 y=0.5, 底部 y=0。pivot 在 (0, 0.45, -1.2)，所以 lid 顶部世界 y=0.95，底部 y=0.45。

所以 lid 底部刚好在 base 顶部 y=0.45。完美。

Lid 上的装饰条 position (0, 0.52, 1.2)，相对 pivot 是 y=0.52，世界 y=0.97。这位置在 lid 顶部上方 0.02，刚好露出来。OK。

lid 上的标签 disc position (x, 0.515, 1.2)，相对 pivot y=0.515，世界 y=0.965。在 lid 顶部上方 0.015。OK。

让我再次检查标签位置：
- 底座标签 disc.position = (x, 0.46, 0.85)，x=-1 或 1，所以世界坐标 (-1, 0.46, 0.85) 和 (1, 0.46, 0.85)
- 底座顶部 y=0.45，标签 disc 中心 y=0.46，disc 高度 0.015，所以 disc 底部 y=0.4525，刚好在 base 表面之上 0.0025。OK。
- 文字 plane 在 y=0.472，刚好在 disc 上方。

但 L 标签在 x=-1 时，与 L 凹槽（中心 x=-1）共线。从俯视图看，L 标签在 L 凹槽的前方 z=0.85，距离凹槽中心 0.85，凹槽半径 0.7，所以标签距离凹槽边缘 0.15。空间紧凑但不重叠。

实际上 z=0.85 太近了。base 前缘在 z=1.2，标签半径 0.32，所以标签边缘在 z=0.85+0.32=1.17，离 base 前缘 0.03。太靠前了。

我把标签半径减小到 0.28，z 改为 0.9。标签边缘 z=1.18，仍接近 base 前缘 1.2。可以接受。

或者把标签放在 z=0.92 处，半径 0.26，那么标签边缘 z=1.18。OK。

但标签还需要避开凹槽：凹槽半径 0.7，凹槽前缘 z=0.7。标签后缘 z=0.92-0.26=0.66。所以标签和凹槽之间有 0.04 间距。这太紧了。

让我重新设计 base 尺寸和凹槽位置：
- base 长度 4.4，宽度 2.6（更宽一点）
- 凹槽中心 x=-1.1 和 1.1
- 凹槽半径 0.65
- 标签 z=0.95，半径 0.3
- 凹槽前缘 z=0.65，标签后缘 z=0.95-0.3=0.65。刚好接触。

或者让标签和凹槽都更小一点，留出更多空间：
- base 4 x 2.6
- 凹槽中心 x=-1.05 和 1.05，半径 0.6
- 标签 z=0.9，半径 0.28
- 凹槽前缘 z=0.6，标签后缘 z=0.9-0.28=0.62

留出 0.02 间距。仍紧。

实际上真实的眼镜盒，L/R 标签就是直接和凹槽相邻甚至围绕凹槽。我让标签直接放在凹槽旁边或者甚至和凹槽重叠的"前侧"。

简化方案：去掉单独的标签 disc，直接在凹槽内壁底部印上 L/R 文字！这样最简洁。

或者：在凹槽前方的 base 顶部表面，直接用 Plane 贴上文字纹理，不要 disc 装饰。文字是 L 或 R 单字符。

让我用这种方案：
- 凹槽前方的 base 表面（z=0.95 处），放一个 Plane，贴上字母纹理
- 字母颜色：L 是珊瑚红，R 是青蓝
- Plane 大小 0.5x0.5，刚好显眼

```js
function makeLetterPlane(x, letter, color) {
  const tex = makeLetterTexture(letter, color);
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.5, 0.5),
    new THREE.MeshBasicMaterial({ map: tex, transparent: true })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.451, 0.95); // 略高于 base 顶部
  return plane;
}
```

但这样 L 和 R 颜色直接显示在 base 上，没有底色衬托，可能不够醒目。我加上一个圆形底色 disc：

```js
function makeLetterLabel(x, letter, letterColor) {
  const group = new THREE.Group();
  // 底色 disc
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.28, 0.28, 0.01, 48),
    new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.6, metalness: 0.3 })
  );
  disc.position.set(x, 0.455, 0.95);
  group.add(disc);
  
  // 字母 plane
  const tex = makeLetterTexture(letter, letterColor);
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.4, 0.4),
    new THREE.MeshBasicMaterial({ map: tex, transparent: true })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.461, 0.95);
  group.add(plane);
  
  return group;
}
```

这样 L 标签是黑底珊瑚红字，R 标签是黑底青蓝字，醒目且高级。

base 尺寸保持 4 x 0.9 x 2.4，凹槽半径 0.7 中心 x=-1 和 1。
- 凹槽前缘 z=0.7
- 标签 z=0.95，半径 0.28，后缘 z=0.67

差 0.03，紧但不重叠。

让 base 宽度增加到 2.6 让间距更舒服。但 RoundedBoxGeometry 2.4 也行，我保留。

让我让 base 长度 4.2 给两端更多余量：
- 凹槽中心 x=-1.05 和 1.05，半径 0.7
- 凹槽外缘 x=-1.75 和 1.75
- base 长度 4.2，两端从 -2.1 到 2.1
- base 两端到凹槽外缘 0.35

OK，更舒服。

让 lid 也变成 4.2 x 0.5 x 2.4。

最终参数：
- base: 4.2 x 0.9 x 2.4，圆角 0.25
- 凹槽: 半径 0.7，深 0.6，中心 (±1.05, 0.45, 0)
- 标签: 圆盘半径 0.28，z=0.95
- lid: 4.2 x 0.5 x 2.4，圆角 0.22
- lidPivot: (0, 0.45, -1.2)
- lid 局部: (0, 0.25, 1.2)
- 铰链: (0, 0.45, -1.2)，长度 2.6，半径 0.08

实际上铰链长度应该比 lid 宽度短一点，刚好嵌在两端。lid 宽度 2.4，铰链长度 2.6 — 略长。改 2.4。

或者让铰链更明显，长度 2.6，超出 lid 两端 0.1。视觉上像个金属杆穿过。我保留 2.6。

最后让我加一个细节：lid 顶部不仅有装饰条，还可以加上一些图案，比如一个小的品牌 logo 圆形。

实际上目前已经够丰富，不要再加了。开始写代码。

我想再做一个改进：让 case 整体可以微微悬浮（轻微上下浮动），增加生动感。但这可能干扰交互，因为用户需要点击 lid。

不加悬浮，但保持桌面光晕和 rim light 的呼吸效果。

OK 最终代码！

代码细节修正：

1. 在 lid 上点击时，需要避免与 OrbitControls 拖拽冲突。OrbitControls 默认在 mousedown 后开始拖拽，但 click 事件在 mouseup 时触发。如果用户在 lid 上 mousedown 然后立刻 mouseup（没有移动），就会触发 click。如果用户拖拽视角，click 不会触发（因为鼠标移动了）。

但 Three.js 的 click 事件实际上是基于浏览器的，只要 mousedown 和 mouseup 在同一元素上，且移动距离很小，就会触发。我可以用一个阈值判断：

```js
let mouseDownPos = null;
renderer.domElement.addEventListener('mousedown', (e) => {
  mouseDownPos = { x: e.clientX, y: e.clientY };
});
renderer.domElement.addEventListener('mouseup', (e) => {
  if (!mouseDownPos) return;
  const dx = e.clientX - mouseDownPos.x;
  const dy = e.clientY - mouseDownPos.y;
  const dist = Math.sqrt(dx*dx + dy*dy);
  if (dist < 5) {
    // 这是个点击，不是拖拽
    handleClick(e);
  }
  mouseDownPos = null;
});
```

这样更精确。我用这种方式。

让我整合到代码中：

```js
let mouseDownPos = null;
renderer.domElement.addEventListener('mousedown', (e) => {
  mouseDownPos = { x: e.clientX, y: e.clientY };
});
renderer.domElement.addEventListener('mouseup', (e) => {
  if (!mouseDownPos) return;
  const dx = e.clientX - mouseDownPos.x;
  const dy = e.clientY - mouseDownPos.y;
  const dist = Math.sqrt(dx*dx + dy*dy);
  if (dist < 5) {
    // 是个点击
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObject(lidGroup, true);
    if (intersects.length > 0) {
      toggleLid();
    }
  }
  mouseDownPos = null;
});
```

这样更可靠。让我用这种方式。

实际上更好的做法是直接监听 'click' 事件，因为浏览器已经做了区分。如果用户在拖拽后释放，浏览器不会触发 click（除非拖拽距离很小）。但为了保险，用我上面的方式。

OK 让我整合最终代码。

让我再仔细审视一下 lid 旋转方向：
- lidPivot 在 (0, 0.45, -1.2)
- lid 在 lidPivot 局部坐标 (0, 0.25, 1.2)
- 当 lidPivot.rotation.x = 0 时，lid 在 (0, 0.7, 0) 世界坐标（关闭状态）
- 当 lidPivot.rotation.x = -π/2 时，lid 旋转 -90度（绕 X 轴）
  - 局部 (0, 0.25, 1.2) → 世界 (0, 0.45 + 1.2, -1.2 - 0.25) = (0, 1.65, -1.45)
  - lid 在 pivot 后方上方，这就是开盖状态

所以负角度让 lid 向后翻开，这是正确的。

实际上一般的眼镜盒是向前翻开（朝向用户）。如果用户在 +z 方向，那么 lid 应该向 +z 方向翻开。

让我改成 lidPivot 在 (0, 0.45, +1.2)（base 前缘），lid 局部 (0, 0.25, -1.2)（在 pivot 后方）。
- 关闭时 lid 在 (0, 0.7, 0)
- 当 rotation.x = +π/2 时，lid 旋转 +90度
  - 局部 (0, 0.25, -1.2) → 世界 (0, 0.45 + (-1.2), 1.2 - 0.25) = (0, -0.75, 0.95)
  - lid 在 pivot 前下方？这不对，lid 应该向前翻开

让我重新想：lid 应该绕铰链旋转，铰链在 lid 一侧。比如铰链在 lid 后缘（-z 方向），lid 向后翻开（朝-z）就露出顶部。或者铰链在前缘（+z 方向），lid 向前翻开（朝+z）。

通常隐形眼镜盒是向后翻开的（远离用户），因为这样不影响用户操作。但也可以向前翻开。

我选择向后翻开（朝-z 方向），lid 在打开后位于 base 后上方。

铰链位置 (0, 0.45, -1.2)，lid 局部 (0, 0.25, 1.2) — lid 在铰链的 +z 方向。
当 lidPivot.rotation.x = -θ 时，lid 向 -z 方向翻开（即 lid 的 +z 部分向上向后移动）。

具体计算：
- 初始 lid 中心局部 (0, 0.25, 1.2)
- 旋转 -θ 后：(0, 0.25*cos(θ) + 1.2*sin(θ), -0.25*sin(θ) + 1.2*cos(θ))
- 加 pivot: (0, 0.45 + 0.25*cos(θ) + 1.2*sin(θ), -1.2 - 0.25*sin(θ) + 1.2*cos(θ))

θ=π/2 (90度)：
- y = 0.45 + 0 + 1.2 = 1.65
- z = -1.2 - 0.25 + 0 = -1.45
- lid 在 (0, 1.65, -1.45) — 后上方

θ=π*0.6 (~108度)：
- cos = -0.309, sin = 0.951
- y = 0.45 + 0.25*(-0.309) + 1.2*0.951 = 0.45 - 0.077 + 1.141 = 1.514
- z = -1.2 - 0.25*0.951 + 1.2*(-0.309) = -1.2 - 0.238 - 0.371 = -1.809
- lid 在 (0, 1.514, -1.809)

OK 这样 lid 向后翻开了。从摄像机 (4.5, 3.8, 5.5) 看，lid 在 base 后上方，可见。

让 θ = π * 0.6 ≈ 108度，刚好让 lid 大致水平向后。

或者让 lid 翻开更多，θ = π * 0.65 ≈ 117度，lid 略微下垂。

我选 θ = π * 0.6。

OK 最终代码！

让我写出完整版本：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LENS CASE · 隐形眼镜盒 3D 互动</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;500;700&family=DM+Mono:wght@400;500&display=swap');

:root {
  --bg-0: #0a0d12;
  --bg-1: #161b22;
  --fg: #e8e6e1;
  --muted: #8b8f97;
  --accent: #ff6b5b;
  --accent-2: #2dd4bf;
  --border: rgba(255,255,255,0.08);
  --card: rgba(20,25,32,0.65);
}

* { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  height: 100%;
  background: 
    radial-gradient(ellipse 80% 60% at 30% 15%, #1c2230 0%, transparent 70%),
    radial-gradient(ellipse 60% 50% at 75% 80%, #1f1612 0%, transparent 70%),
    #06080c;
  color: var(--fg);
  font-family: 'Space Grotesk', sans-serif;
  overflow: hidden;
  user-select: none;
}

#app {
  position: relative;
  width: 100vw;
  height: 100vh;
}

canvas { display: block; outline: none; }

/* 装饰角标 */
.corner {
  position: absolute;
  width: 48px; height: 48px;
  pointer-events: none;
  z-index: 5;
  opacity: 0.6;
}
.corner::before, .corner::after {
  content: '';
  position: absolute;
  background: var(--border);
}
.corner.tl { top: 16px; left: 16px; }
.corner.tl::before { top: 0; left: 0; width: 20px; height: 1px; }
.corner.tl::after { top: 0; left: 0; width: 1px; height: 20px; }
.corner.tr { top: 16px; right: 16px; }
.corner.tr::before { top: 0; right: 0; width: 20px; height: 1px; }
.corner.tr::after { top: 0; right: 0; width: 1px; height: 20px; }
.corner.bl { bottom: 16px; left: 16px; }
.corner.bl::before { bottom: 0; left: 0; width: 20px; height: 1px; }
.corner.bl::after { bottom: 0; left: 0; width: 1px; height: 20px; }
.corner.br { bottom: 16px; right: 16px; }
.corner.br::before { bottom: 0; right: 0; width: 20px; height: 1px; }
.corner.br::after { bottom: 0; right: 0; width: 1px; height: 20px; }

/* 顶部 */
.header {
  position: absolute;
  top: 0; left: 0; right: 0;
  padding: 28px 36px;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  z-index: 10;
  pointer-events: none;
}

.brand {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.brand .tag {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  letter-spacing: 0.32em;
  color: var(--muted);
  text-transform: uppercase;
}

.brand .tag::before {
  content: '';
  width: 18px;
  height: 1px;
  background: var(--accent);
}

.brand h1 {
  font-size: 32px;
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1;
  background: linear-gradient(110deg, #ffffff 0%, #ffd4cc 50%, #ff6b5b 100%);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

.brand .sub {
  font-size: 13px;
  color: var(--muted);
  font-weight: 300;
  letter-spacing: 0.04em;
}

/* 状态卡 */
.status-card {
  background: var(--card);
  border: 1px solid var(--border);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 14px;
  padding: 14px 20px;
  pointer-events: auto;
  min-width: 220px;
}

.status-card .label {
  font-family: 'DM Mono', monospace;
  font-size: 10px;
  letter-spacing: 0.25em;
  color: var(--muted);
  text-transform: uppercase;
  margin-bottom: 8px;
}

.status-card .value {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 17px;
  font-weight: 500;
}

.status-dot {
  position: relative;
  width: 10px; height: 10px;
  border-radius: 50%;
  background: var(--accent-2);
  box-shadow: 0 0 14px var(--accent-2);
  transition: all 0.35s;
}

.status-dot::after {
  content: '';
  position: absolute;
  inset: -4px;
  border-radius: 50%;
  border: 1px solid currentColor;
  opacity: 0.3;
  animation: pulse 2s ease-out infinite;
}

.status-dot.open {
  background: var(--accent);
  box-shadow: 0 0 14px var(--accent);
  color: var(--accent);
}

.status-dot.closed {
  color: var(--accent-2);
}

@keyframes pulse {
  0% { transform: scale(1); opacity: 0.4; }
  100% { transform: scale(1.6); opacity: 0; }
}

.status-meta {
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px solid var(--border);
  display: flex;
  justify-content: space-between;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  color: var(--muted);
}

.status-meta .lr {
  display: flex;
  gap: 14px;
}

.status-meta .lr span {
  display: flex;
  align-items: center;
  gap: 6px;
}

.status-meta .lr .l-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--accent);
}

.status-meta .lr .r-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--accent-2);
}

/* 底部 */
.footer {
  position: absolute;
  bottom: 0; left: 0; right: 0;
  padding: 24px 36px;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  z-index: 10;
  pointer-events: none;
}

.hint {
  background: var(--card);
  border: 1px solid var(--border);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 12px;
  padding: 12px 16px;
  pointer-events: auto;
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.hint-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: var(--muted);
  padding: 0 12px;
  position: relative;
}

.hint-item:not(:last-child)::after {
  content: '';
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 1px;
  height: 14px;
  background: var(--border);
}

.hint-item .key {
  font-family: 'DM Mono', monospace;
  background: rgba(255,255,255,0.06);
  border: 1px solid var(--border);
  border-radius: 5px;
  padding: 2px 8px;
  font-size: 10px;
  letter-spacing: 0.1em;
  color: var(--fg);
}

.action-group {
  display: flex;
  gap: 10px;
  pointer-events: auto;
}

.btn {
  background: var(--card);
  border: 1px solid var(--border);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  color: var(--fg);
  padding: 11px 20px;
  border-radius: 10px;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  letter-spacing: 0.18em;
  cursor: pointer;
  transition: all 0.25s;
  text-transform: uppercase;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn:hover {
  background: rgba(255,107,91,0.12);
  border-color: var(--accent);
  color: var(--accent);
  transform: translateY(-1px);
}

.btn.primary {
  background: linear-gradient(135deg, rgba(255,107,91,0.18), rgba(255,107,91,0.08));
  border-color: rgba(255,107,91,0.4);
  color: #ffd4cc;
}

.btn.primary:hover {
  background: linear-gradient(135deg, rgba(255,107,91,0.3), rgba(255,107,91,0.15));
  border-color: var(--accent);
  color: #fff;
}

/* 加载 */
.loader {
  position: absolute;
  inset: 0;
  background: #06080c;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  transition: opacity 0.7s;
}

.loader.hidden {
  opacity: 0;
  pointer-events: none;
}

.loader-content {
  text-align: center;
}

.loader-ring {
  width: 52px;
  height: 52px;
  border: 1px solid rgba(255,255,255,0.08);
  border-top-color: var(--accent);
  border-right-color: var(--accent-2);
  border-radius: 50%;
  margin: 0 auto 18px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loader-text {
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  letter-spacing: 0.35em;
  color: var(--muted);
  text-transform: uppercase;
}

/* 通知 */
.toast {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0.9);
  background: var(--card);
  border: 1px solid var(--border);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  padding: 14px 22px;
  border-radius: 12px;
  font-size: 14px;
  color: var(--fg);
  pointer-events: none;
  opacity: 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 20;
  display: flex;
  align-items: center;
  gap: 10px;
}

.toast.show {
  opacity: 1;
  transform: translate(-50%, -50%) scale(1);
}

.toast .icon {
  width: 8px; height: 8px;
  border-radius: 50%;
  background: var(--accent);
}

.toast.closed .icon { background: var(--accent-2); }

@media (max-width: 720px) {
  .header { padding: 16px; flex-direction: column; gap: 14px; }
  .footer { padding: 14px; flex-direction: column-reverse; gap: 10px; align-items: stretch; }
  .brand h1 { font-size: 24px; }
  .hint { gap: 0; }
  .hint-item { padding: 4px 10px; }
  .status-card { min-width: 0; }
}
</style>
</head>
<body>
<div id="app">
  <div class="corner tl"></div>
  <div class="corner tr"></div>
  <div class="corner bl"></div>
  <div class="corner br"></div>
  
  <header class="header">
    <div class="brand">
      <span class="tag">3D Interactive</span>
      <h1>LENS CASE</h1>
      <span class="sub">隐形眼镜盒 · 左右分明</span>
    </div>
    <div class="status-card">
      <div class="label">Case Status</div>
      <div class="value">
        <span class="status-dot closed" id="statusDot"></span>
        <span id="statusText">已合盖</span>
      </div>
      <div class="status-meta">
        <div class="lr">
          <span><span class="l-dot"></span>L</span>
          <span><span class="r-dot"></span>R</span>
        </div>
        <div id="counterText">CLICKS · 0</div>
      </div>
    </div>
  </header>
  
  <footer class="footer">
    <div class="hint">
      <div class="hint-item"><span class="key">CLICK</span>点击盒盖开合</div>
      <div class="hint-item"><span class="key">DRAG</span>旋转视角</div>
      <div class="hint-item"><span class="key">SCROLL</span>缩放</div>
    </div>
    <div class="action-group">
      <button class="btn" id="resetBtn">重置视角</button>
      <button class="btn primary" id="toggleBtn">开 / 合盒盖</button>
    </div>
  </footer>
  
  <div class="toast" id="toast">
    <span class="icon"></span>
    <span id="toastText">盒盖已打开</span>
  </div>
  
  <div class="loader" id="loader">
    <div class="loader-content">
      <div class="loader-ring"></div>
      <div class="loader-text">Initializing Scene</div>
    </div>
  </div>
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
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';

// ============== 场景 ==============
const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(38, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(4.8, 4.2, 6.0);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.05;
renderer.outputColorSpace = THREE.SRGBColorSpace;
document.getElementById('app').appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.07;
controls.minDistance = 3;
controls.maxDistance = 14;
controls.maxPolarAngle = Math.PI * 0.48;
controls.minPolarAngle = Math.PI * 0.12;
controls.target.set(0, 0.4, 0);
controls.enablePan = false;

// ============== 光照 ==============
const hemi = new THREE.HemisphereLight(0xfff0d8, 0x141821, 0.5);
scene.add(hemi);

const keyLight = new THREE.DirectionalLight(0xfff2dc, 1.6);
keyLight.position.set(5, 8, 4);
keyLight.castShadow = true;
keyLight.shadow.mapSize.set(2048, 2048);
keyLight.shadow.camera.left = -5;
keyLight.shadow.camera.right = 5;
keyLight.shadow.camera.top = 5;
keyLight.shadow.camera.bottom = -5;
keyLight.shadow.camera.near = 0.5;
keyLight.shadow.camera.far = 25;
keyLight.shadow.bias = -0.0005;
keyLight.shadow.radius = 5;
scene.add(keyLight);

const fillLight = new THREE.DirectionalLight(0x6a8aff, 0.35);
fillLight.position.set(-5, 3, -2);
scene.add(fillLight);

const rimLight = new THREE.PointLight(0xff5a3a, 1.2, 8, 2);
rimLight.position.set(-2, 1.5, -3.5);
scene.add(rimLight);

const topGlow = new THREE.PointLight(0x88ddff, 0.4, 6, 2);
topGlow.position.set(0, 5, 0);
scene.add(topGlow);

// ============== 桌面 ==============
const tableGeo = new THREE.CircleGeometry(9, 64);
const tableMat = new THREE.MeshStandardMaterial({
  color: 0x0e1116,
  roughness: 0.92,
  metalness: 0.15
});
const table = new THREE.Mesh(tableGeo, tableMat);
table.rotation.x = -Math.PI/2;
table.position.y = -0.5;
table.receiveShadow = true;
scene.add(table);

// 桌面光晕
const glowGeo = new THREE.RingGeometry(1.6, 2.6, 64);
const glowMat = new THREE.MeshBasicMaterial({
  color: 0xff6b5b,
  transparent: true,
  opacity: 0.08,
  side: THREE.DoubleSide,
  depthWrite: false
});
const glow = new THREE.Mesh(glowGeo, glowMat);
glow.rotation.x = -Math.PI/2;
glow.position.y = -0.49;
scene.add(glow);

// 第二圈微光
const glow2 = new THREE.Mesh(
  new THREE.RingGeometry(2.8, 4.5, 64),
  new THREE.MeshBasicMaterial({ color: 0x2dd4bf, transparent: true, opacity: 0.04, side: THREE.DoubleSide, depthWrite: false })
);
glow2.rotation.x = -Math.PI/2;
glow2.position.y = -0.488;
scene.add(glow2);

// ============== 隐形眼镜盒 ==============
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// 材质
const bodyMat = new THREE.MeshPhysicalMaterial({
  color: 0xf2ece2,
  roughness: 0.4,
  metalness: 0.05,
  clearcoat: 0.4,
  clearcoatRoughness: 0.25
});

const lidMat = new THREE.MeshPhysicalMaterial({
  color: 0xf5efe4,
  roughness: 0.35,
  metalness: 0.08,
  clearcoat: 0.6,
  clearcoatRoughness: 0.18
});

const liquidMat = new THREE.MeshPhysicalMaterial({
  color: 0x9bd5ff,
  roughness: 0.05,
  metalness: 0,
  transmission: 0.85,
  transparent: true,
  opacity: 0.5,
  thickness: 0.5,
  ior: 1.33,
  depthWrite: false
});

const lensMat = new THREE.MeshPhysicalMaterial({
  color: 0xb5d4ff,
  roughness: 0.05,
  metalness: 0,
  transmission: 0.95,
  transparent: true,
  opacity: 0.35,
  thickness: 0.2,
  ior: 1.42,
  side: THREE.DoubleSide,
  depthWrite: false
});

const metalMat = new THREE.MeshStandardMaterial({
  color: 0x2a2e36,
  roughness: 0.35,
  metalness: 0.85
});

// === 底座 ===
const base = new THREE.Mesh(
  new RoundedBoxGeometry(4.2, 0.9, 2.4, 8, 0.22),
  bodyMat
);
base.position.y = 0;
base.castShadow = true;
base.receiveShadow = true;
caseGroup.add(base);

// 底座底部装饰条（侧面凹槽）
const sideGroove = new THREE.Mesh(
  new RoundedBoxGeometry(3.6, 0.08, 0.04, 3, 0.02),
  new THREE.MeshStandardMaterial({ color: 0xd4c8b0, roughness: 0.5, metalness: 0.3 })
);
sideGroove.position.set(0, 0, 1.21);
caseGroup.add(sideGroove);

const sideGrooveBack = sideGroove.clone();
sideGrooveBack.position.z = -1.21;
caseGroup.add(sideGrooveBack);

// === 凹槽 ===
function makeWell(x, color) {
  const g = new THREE.Group();
  g.position.set(x, 0, 0);
  
  const innerRadius = 0.7;
  const wellTopY = 0.45;
  const wellDepth = 0.62;
  const wellBottomY = wellTopY - wellDepth;
  const wellCenterY = (wellTopY + wellBottomY) / 2;
  
  // 内壁
  const innerWall = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius, innerRadius, wellDepth, 48, 1, true),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.4, metalness: 0.15, side: THREE.BackSide })
  );
  innerWall.position.y = wellCenterY;
  g.add(innerWall);
  
  // 内底
  const innerBottom = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius - 0.01, innerRadius - 0.01, 0.02, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.5, metalness: 0.15 })
  );
  innerBottom.position.y = wellBottomY + 0.015;
  g.add(innerBottom);
  
  // 液体
  const liquidH = 0.42;
  const liquid = new THREE.Mesh(
    new THREE.CylinderGeometry(innerRadius - 0.025, innerRadius - 0.025, liquidH, 48),
    liquidMat
  );
  liquid.position.y = wellBottomY + liquidH/2 + 0.025;
  g.add(liquid);
  
  // 液体表面高光圆
  const surfaceHighlight = new THREE.Mesh(
    new THREE.CircleGeometry(innerRadius - 0.04, 48),
    new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.15, depthWrite: false })
  );
  surfaceHighlight.rotation.x = -Math.PI/2;
  surfaceHighlight.position.y = wellBottomY + liquidH + 0.026;
  g.add(surfaceHighlight);
  
  // 镜片
  const lens = new THREE.Mesh(
    new THREE.CylinderGeometry(0.55, 0.55, 0.05, 48),
    lensMat
  );
  lens.position.y = wellBottomY + liquidH - 0.02;
  lens.rotation.x = 0.12;
  lens.rotation.z = 0.08;
  g.add(lens);
  
  // 顶部边缘环
  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(innerRadius, 0.035, 12, 48),
    new THREE.MeshStandardMaterial({ color: 0xd8cdb6, roughness: 0.3, metalness: 0.3 })
  );
  ring.rotation.x = Math.PI/2;
  ring.position.y = wellTopY;
  g.add(ring);
  
  return g;
}

caseGroup.add(makeWell(-1.05, 0xff6b5b));
caseGroup.add(makeWell(1.05, 0x2dd4bf));

// === L R 文字纹理 ===
function makeLetterTexture(letter, color) {
  const c = document.createElement('canvas');
  c.width = 256; c.height = 256;
  const ctx = c.getContext('2d');
  ctx.clearRect(0,0,256,256);
  ctx.fillStyle = color;
  ctx.font = 'bold 180px "Space Grotesk", Arial, sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, 128, 140);
  const tex = new THREE.CanvasTexture(c);
  tex.colorSpace = THREE.SRGBColorSpace;
  tex.anisotropy = 8;
  return tex;
}

const texL = makeLetterTexture('L', '#ff6b5b');
const texR = makeLetterTexture('R', '#2dd4bf');
const texLwhite = makeLetterTexture('L', '#ffffff');
const texRwhite = makeLetterTexture('R', '#ffffff');

// 底座上的 L R 标签
function makeBaseLabel(x, texture) {
  const group = new THREE.Group();
  // 黑色底圆盘
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.3, 0.3, 0.018, 48),
    new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.5, metalness: 0.4 })
  );
  disc.position.set(x, 0.46, 0.95);
  group.add(disc);
  
  // 字母
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.42, 0.42),
    new THREE.MeshBasicMaterial({ map: texture, transparent: true, depthWrite: false })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.471, 0.95);
  group.add(plane);
  
  return group;
}

caseGroup.add(makeBaseLabel(-1.05, texL));
caseGroup.add(makeBaseLabel(1.05, texR));

// === 盒盖 ===
const lidPivot = new THREE.Group();
lidPivot.position.set(0, 0.45, -1.21);
caseGroup.add(lidPivot);

const lidGroup = new THREE.Group();
lidPivot.add(lidGroup);

// 盒盖主体
const lid = new THREE.Mesh(
  new RoundedBoxGeometry(4.2, 0.45, 2.4, 8, 0.2),
  lidMat
);
lid.position.set(0, 0.225, 1.21);
lid.castShadow = true;
lid.receiveShadow = true;
lidGroup.add(lid);

// 盒盖内衬（底面深色）
const lidInner = new THREE.Mesh(
  new RoundedBoxGeometry(4.0, 0.05, 2.2, 6, 0.15),
  new THREE.MeshStandardMaterial({ color: 0x2a2e36, roughness: 0.6, metalness: 0.2 })
);
lidInner.position.set(0, 0.0, 1.21);
lidGroup.add(lidInner);

// 盒盖顶部装饰条
const stripe = new THREE.Mesh(
  new RoundedBoxGeometry(3.8, 0.06, 0.18, 4, 0.03),
  new THREE.MeshStandardMaterial({ color: 0x1a1d23, roughness: 0.4, metalness: 0.5 })
);
stripe.position.set(0, 0.255, 1.21);
lidGroup.add(stripe);

// 盒盖上的 L R 标签
function makeLidLabel(x, texture, color) {
  const group = new THREE.Group();
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.32, 0.32, 0.02, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.35, metalness: 0.25 })
  );
  disc.position.set(x, 0.248, 1.21);
  group.add(disc);
  
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.5, 0.5),
    new THREE.MeshBasicMaterial({ map: texture, transparent: true, depthWrite: false })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.259, 1.21);
  group.add(plane);
  
  return group;
}

lidGroup.add(makeLidLabel(-1.05, texLwhite, 0xff6b5b));
lidGroup.add(makeLidLabel(1.05, texRwhite, 0x2dd4bf));

// 铰链
const hinge = new THREE.Mesh(
  new THREE.CylinderGeometry(0.07, 0.07, 2.6, 16),
  metalMat
);
hinge.rotation.z = Math.PI/2;
hinge.position.set(0, 0.45, -1.21);
caseGroup.add(hinge);

// 铰链两端装饰
const hingeCap1 = new THREE.Mesh(
  new THREE.SphereGeometry(0.08, 16, 16),
  metalMat
);
hingeCap1.position.set(-1.3, 0.45, -1.21);
caseGroup.add(hingeCap1);

const hingeCap2 = hingeCap1.clone();
hingeCap2.position.set(1.3, 0.45, -1.21);
caseGroup.add(hingeCap2);

// ============== 交互 ==============
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let isOpen = false;
let animating = false;
let lidAngle = 0;
let targetAngle = 0;
let clickCount = 0;

const statusDot = document.getElementById('statusDot');
const statusText = document.getElementById('statusText');
const counterText = document.getElementById('counterText');
const toast = document.getElementById('toast');
const toastText = document.getElementById('toastText');

function updateStatus(open) {
  if (open) {
    statusDot.classList.add('open');
    statusDot.classList.remove('closed');
    statusText.textContent = '已开盖';
  } else {
    statusDot.classList.remove('open');
    statusDot.classList.add('closed');
    statusText.textContent = '已合盖';
  }
}

function showToast(open) {
  toastText.textContent = open ? '盒盖已打开 · 镜片露出' : '盒盖已合上 · 镜片保存中';
  toast.classList.toggle('closed', !open);
  toast.classList.add('show');
  clearTimeout(showToast._t);
  showToast._t = setTimeout(() => toast.classList.remove('show'), 1600);
}

function toggleLid() {
  if (animating) return;
  isOpen = !isOpen;
  targetAngle = isOpen ? Math.PI * 0.62 : 0;
  animating = true;
  clickCount++;
  counterText.textContent = `CLICKS · ${clickCount}`;
  updateStatus(isOpen);
  showToast(isOpen);
}

// 点击检测（区分拖拽和点击）
let mouseDownPos = null;
let mouseDownTime = 0;

renderer.domElement.addEventListener('mousedown', (e) => {
  mouseDownPos = { x: e.clientX, y: e.clientY };
  mouseDownTime = performance.now();
});

renderer.domElement.addEventListener('mouseup', (e) => {
  if (!mouseDownPos) return;
  const dx = e.clientX - mouseDownPos.x;
  const dy = e.clientY - mouseDownPos.y;
  const dist = Math.sqrt(dx*dx + dy*dy);
  const dt = performance.now() - mouseDownTime;
  mouseDownPos = null;
  
  if (dist > 5 || dt > 500) return; // 拖拽或长按，不算点击
  
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidGroup, true);
  if (intersects.length > 0) {
    toggleLid();
  }
});

// 触摸支持
renderer.domElement.addEventListener('touchend', (e) => {
  if (e.changedTouches.length === 0) return;
  const t = e.changedTouches[0];
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((t.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((t.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidGroup, true);
  if (intersects.length > 0) {
    toggleLid();
  }
});

// 鼠标悬停效果
let hovered = false;
renderer.domElement.addEventListener('mousemove', (e) => {
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidGroup, true);
  const nowHovered = intersects.length > 0 && !animating;
  if (nowHovered !== hovered) {
    hovered = nowHovered;
    renderer.domElement.style.cursor = hovered ? 'pointer' : 'grab';
  }
});

renderer.domElement.style.cursor = 'grab';
renderer.domElement.addEventListener('mousedown', () => {
  if (!hovered) renderer.domElement.style.cursor = 'grabbing';
});
window.addEventListener('mouseup', () => {
  renderer.domElement.style.cursor = hovered ? 'pointer' : 'grab';
});

// 按钮
document.getElementById('resetBtn').addEventListener('click', () => {
  const startPos = camera.position.clone();
  const startTarget = controls.target.clone();
  const endPos = new THREE.Vector3(4.8, 4.2, 6.0);
  const endTarget = new THREE.Vector3(0, 0.4, 0);
  const t0 = performance.now();
  const dur = 700;
  function step() {
    const t = Math.min(1, (performance.now() - t0)/dur);
    const e = 1 - Math.pow(1-t, 3);
    camera.position.lerpVectors(startPos, endPos, e);
    controls.target.lerpVectors(startTarget, endTarget, e);
    controls.update();
    if (t < 1) requestAnimationFrame(step);
  }
  step();
});

document.getElementById('toggleBtn').addEventListener('click', toggleLid);

// ============== 渲染循环 ==============
function easeOutCubic(t) { return 1 - Math.pow(1 - t, 3); }
function easeInOutCubic(t) {
  return t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
}

let lastTime = performance.now();
function animate() {
  requestAnimationFrame(animate);
  const now = performance.now();
  const dt = (now - lastTime) / 1000;
  lastTime = now;
  
  // 盖子动画 - 使用插值缓动
  if (animating) {
    const diff = targetAngle - lidAngle;
    if (Math.abs(diff) < 0.0005) {
      lidAngle = targetAngle;
      animating = false;
    } else {
      lidAngle += diff * Math.min(1, dt * 5.5);
    }
    lidPivot.rotation.x = -lidAngle;
  }
  
  // 呼吸效果
  glowMat.opacity = 0.07 + Math.sin(now * 0.0011) * 0.025;
  rimLight.intensity = 1.0 + Math.sin(now * 0.0013) * 0.2;
  topGlow.intensity = 0.35 + Math.sin(now * 0.0008 + 1) * 0.1;
  
  // 桌面光晕2呼吸（不同节奏）
  glow2.material.opacity = 0.035 + Math.sin(now * 0.0007 + 2) * 0.015;
  
  controls.update();
  renderer.render(scene, camera);
}

// ============== 启动 ==============
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// 进场动画
caseGroup.position.y = -1.5;
caseGroup.scale.setScalar(0.85);
const introT0 = performance.now();
function intro() {
  const t = Math.min(1, (performance.now() - introT0) / 1400);
  const e = easeOutCubic(t);
  caseGroup.position.y = -1.5 + e * 1.5;
  caseGroup.scale.setScalar(0.85 + e * 0.15);
  if (t < 1) requestAnimationFrame(intro);
}
intro();

animate();

// 隐藏加载器
setTimeout(() => {
  document.getElementById('loader').classList.add('hidden');
}, 500);
</script>
</body>
</html>
```

让我再检查一下：

1. ✅ Three.js 通过 importmap 导入
2. ✅ 没有使用默认蓝紫色，用了珊瑚红+青蓝+奶白+深炭灰
3. ✅ 字体用 Space Grotesk + DM Mono，避免 Inter/Roboto
4. ✅ 没用 alert/prompt，用 toast 提示
5. ✅ 标题没有 emoji
6. ✅ 使用语义化 HTML
7. ✅ 响应式
8. ✅ 完整交互：点击盒盖、拖拽旋转、滚轮缩放、按钮
9. ✅ 多个动态元素：呼吸光晕、rim light 呼吸、进场动画、状态点脉冲
10. ✅ 3D 可交互

视觉细节：
- 进场动画：盒子从下方升起
- 桌面有光晕（珊瑚色 + 青蓝色，呼吸节奏不同）
- 状态卡显示开关状态、点击次数、L/R 标识
- 按钮：重置视角 + 切换盒盖
- 加载动画
- Toast 提示

我觉得这版应该足够出色。让我最后整理一下代码并确保没有错误。

一些可能的细节问题：
1. lidInner 在 lid 底部，当 lid 关闭时面向 base。其 y 坐标相对 lid 中心是 -0.225，但 lid 底部在 y=0（相对 pivot），所以 lidInner 应该在 y=-0.025（相对 lidGroup，刚好在 lid 底部下方一点）。

让我重新审视：
- lid 几何是 4.2 x 0.45 x 2.4，所以 lid 半高 0.225
- lid.position.y = 0.225（在 lidGroup 内）
- lid 顶部在 0.225 + 0.225 = 0.45（lidGroup 局部）
- lid 底部在 0.225 - 0.225 = 0

- lidInner 几何 4.0 x 0.05 x 2.2，半高 0.025
- lidInner.position.y = 0 → lidInner 顶部 0.025，底部 -0.025
- lidInner 顶部刚好在 lid 底部（0）的下方一点 (-0.025 到 0.025)

实际上 lidInner 应该贴在 lid 底部，所以 lidInner.position.y = -0.025 + 0.001（防止 z-fighting）

让我修正：
```js
lidInner.position.set(0, -0.024, 1.21);  // 略低于 lid 底部
```

但 lidInner 是 0.05 高度，所以中心在 -0.025 + 一点点 = -0.024，顶部 -0.024 + 0.025 = 0.001 — 在 lid 底部稍微下方。

但 lid 底部 y=0（在 lidGroup 局部），lidInner 顶部 y=0.001 还在 lid 底部之上 0.001。这会重叠。

让我重新设：
- lidInner 中心 y = -0.025 - 0.001 = -0.026
- lidInner 顶部 y = -0.026 + 0.025 = -0.001（在 lid 底部下方 0.001）

```js
lidInner.position.set(0, -0.026, 1.21);
```

或者简化，让 lidInner 中心 y = 0，但 lidInner 半高 0.025，所以 lidInner 顶部 0.025 在 lid 底部 0 之上 0.025。这会让 lidInner 突出 lid 底部。

实际上我想让 lidInner 看起来像 lid 内表面（从 lid 底部下凸的盖板）。当 lid 关闭时，lidInner 在 lid 内部上方（朝向 base）。当 lid 打开后翻转，lidInner 朝上。

让 lidInner.position.y = -0.22 (即 lid 底部下方一点，朝向 base)，那么 lidInner 顶部 y=-0.22+0.025=-0.195，lidInner 底部 y=-0.225。

实际意思是 lidInner 是 lid 内部的一个小盖板，在 lid 底部内侧。这样从下面看 lid（lid 打开后从内部看）会看到 lidInner。但实际上 lidInner 在 lid 几何内部，被 lid 的几何挡住。

我想要的实际效果是：当 lid 打开后，用户从上方看 lid 内部，能看到一个深色的内衬。这是通过把 lidInner 贴在 lid 底部内侧实现。

让 lidInner.position.y = 0.001（紧贴 lid 底部内侧上方），半高 0.025，所以 lidInner 底部 0.001 - 0.025 = -0.024（伸到 lid 底部下方 0.024）。lidInner 顶部 0.001 + 0.025 = 0.026。

这意味着 lidInner 有一半在 lid 内部，一半伸出 lid 底部。伸出部分会挡住 base 上的凹槽！

让 lidInner 是个薄盖板，仅贴在 lid 底部内侧：
- lidInner 几何改成 4.0 x 0.02 x 2.2，半高 0.01
- lidInner.position.y = 0.011（lid 底部上方 0.011）
- lidInner 底部 0.001，顶部 0.021

实际上当 lid 关闭时，从 lid 内部（朝下看）应该看到 lid 底部的内侧。lid 几何本身是 RoundedBoxGeometry，所以 lid 已经有底面。我不需要 lidInner。

去掉 lidInner 简化代码。

或者保留 lidInner 作为更深的盖板：
- lidInner 几何 4.0 x 0.02 x 2.2，半高 0.01
- lidInner.position.y = -0.215（lid 底部下方 0.215，在 lid 几何内）— 但 lid 底部 y=0，lidInner 中心 y=-0.215 在 lid 内部下方
- lidInner 顶部 y=-0.205，底部 y=-0.225

这样 lidInner 在 lid 几何内部下方，从 lid 底部下方 0.215 处。当 lid 关闭时 lidInner 在 lid 内部，从 lid 底部内侧露出。当 lid 翻开后 lidInner 朝上。

但 lidInner 在 lid 几何内部会被 lid 几何包围，看不到。所以这个设计没用。

简化：去掉 lidInner，让 lid 内表面就是 lid 几何本身。Lid 用 lidMat 即可，已经是浅色。但 lid 关闭时 lid 内表面朝下，用户看不到（朝向 base）。lid 打开后 lid 内表面朝上，用户能看到。

我想给 lid 内表面一个深色，区分内外。可以用 multi-material：lid 6 个面用不同材质。但 RoundedBoxGeometry 的材质组分配可能复杂。

简化：保持 lid 单材质 lidMat。当 lid 打开后，用户看到 lid 内表面（lidMat 颜色）。可以接受。

去掉 lidInner，简化代码。

让我整理最终代码并删掉 lidInner：

实际上有 lidInner 也无伤大雅，让我保留但简化：
- lidInner 是一个紧贴 lid 内顶部的薄板
- 当 lid 翻开后，从上方看到 lid 内部，lidInner 是深色内衬
- lidInner.position.y 应该在 lid 底部内侧紧贴的位置

lid 底部内侧 = lid.position.y - lidHeight/2 + epsilon = 0.225 - 0.225 + 0.001 = 0.001
lidInner 应该贴在 lid 底部内侧，所以 lidInner.position.y = 0.001 + lidInnerHeight/2 = 0.001 + 0.025 = 0.026

不对！lidInner 应该贴在 lid 底部内侧，朝向 base。lid 底部 y=0，lidInner 在 lid 底部内侧上方一点。lidInner.position.y = 0 + lidInnerHeight/2 + 0.001 = 0.026。

但这样 lidInner 在 lid 内部，从外面看不到。

或者，让 lidInner 是从 lid 底部向下伸出的小盖板：
- lidInner.position.y = 0 - lidInnerHeight/2 - 0.001 = -0.026
- lidInner 顶部 -0.001（紧贴 lid 底部下方），底部 -0.051

这样 lidInner 从 lid 底部向下伸出 0.05 单位，像个内部盖板嵌入 base 凹槽区域。当 lid 关闭时，lidInner 在 base 上方 0.05 处，悬空在凹槽上方。当 lid 打开后，lidInner 朝上。

这个设计可能更好——lidInner 像是 lid 的延伸盖板，盖住凹槽。但 lid 已经盖住 base 顶部了，再加 lidInner 显得冗余。

我决定去掉 lidInner，简化代码。

最终代码我把 lidInner 删掉。

让我整理代码最终版：

实际上让我重新考虑：当 lid 关闭时，base 顶部 y=0.45，lid 底部 y=0（lidGroup 局部），即世界坐标 y=0.45（pivot y=0.45，lid 底部相对 pivot y=-0.225，世界 0.45-0.225+0.225=0.45）。

等等我重新算：lid 在 lidGroup 局部坐标 (0, 0.225, 1.21)。lidGroup 是 lidPivot 的子，lidPivot 在 (0, 0.45, -1.21)。lidGroup 在 lidPivot 局部 (0,0,0)。

所以 lid 中心的世界坐标 = lidPivot 位置 + lid 在 lidGroup 局部 = (0, 0.45+0.225, -1.21+1.21) = (0, 0.675, 0)。

lid 半高 0.225，所以 lid 顶部世界 y = 0.675+0.225 = 0.9，lid 底部世界 y = 0.675-0.225 = 0.45。

base 顶部世界 y = 0+0.45 = 0.45（base 中心 y=0，半高 0.45）。

所以 lid 底部 y=0.45 刚好等于 base 顶部 y=0.45。完美贴合！

但 lid 边缘和 base 边缘在 y=0.45 这条线上接触。从视觉上，lid 直接坐在 base 顶部，没有缝隙。这看起来不像真实眼镜盒（通常 lid 和 base 之间有一点边缘重叠）。

让 lid 稍微大于 base 或 lid 中心略低，制造"盖在 base 上"的视觉效果：
- 把 lid 的 z 尺寸 2.4 改为 2.5（比 base 大 0.1），这样 lid 边缘超出 base 边缘 0.05
- 但 lid 中心位置不变，lid 底部仍然 y=0.45 = base 顶部 y=0.45

或者把 lid 中心略低：lid.position.y = 0.2，那么 lid 底部 y=0.2-0.225 = -0.025，lid 顶部 y=0.2+0.225=0.425。但这会让 lid 底部低于 base 顶部，重叠 0.025。视觉上 lid 罩在 base 上方一点点。

不对，我希望 lid 在 base 上方，不应该重叠。让 lid 中心略高：lid.position.y = 0.24，lid 底部 y=0.015，lid 顶部 y=0.465。lid 底部 y=0.015 在 lidGroup 局部，世界 y = 0.45+0.015 = 0.465。base 顶部 y=0.45，所以 lid 底部世界 y=0.465 高于 base 顶部 0.015。视觉上有 0.015 的缝隙。

或者保持贴合 y=0.45 = 0.45。视觉上看起来 lid 直接坐在 base 上，没有缝隙。这是合理的。

OK 保持原样。

最终代码我会做以下决定：
1. 去掉 lidInner（简化）
2. 让 base 顶部和 lid 底部贴合
3. 增加一些边缘高光让 lid 看起来不与 base 完全粘连

或者更简单：lid 稍微大于 base（lid 高度 0.5，base 高度 0.9）让 lid 视觉上像盖子。

让我让 lid z=2.5（比 base 大 0.1）：
- lid RoundedBoxGeometry(4.3, 0.45, 2.5, 8, 0.2)
- 边缘超出 base 0.05

这样 lid 看起来像盖在 base 上的盖子。

调整 lidGroup 内的位置和装饰条尺寸。

实际上让 lid 大于 base 在视觉上更好。我把 lid 设为 4.3 x 0.45 x 2.5，装饰条 3.8 x 0.06 x 0.18，圆盘半径 0.32 等不变。

而 lidPivot 在 (0, 0.45, -1.21) 不变（铰链在 base 后缘）。
lid 在 lidGroup 局部 (0, 0.225, 1.21)，所以 lid 中心 z=0（世界），lid 后缘 z=-1.25（世界），lid 前缘 z=1.25（世界）。base 前缘 z=1.2，lid 前缘 z=1.25，超出 0.05。lid 后缘 z=-1.25，铰链在 z=-1.21，lid 后缘超出铰链 0.04，这样 lid 后部能"包"住铰链。

实际上让 lid 后缘在铰链处或略前一点，避免 lid 翻开时与铰链碰撞。

简化：让 lid z 尺寸 2.4 = base z 尺寸 2.4，对齐。lid 翻开时不会有问题。

OK 保持 lid 4.2 x 0.45 x 2.4，与 base 同尺寸。视觉上 lid 和 base 紧密贴合像整体，但用户可以点击打开。

实际效果：用户看到一个完整的眼镜盒，点击盒盖后盒盖向后翻开，露出内部凹槽和镜片。

OK 让我整理最终代码：

让我再看看代码长度，应该约 500-600 行 HTML+CSS+JS。可以接受。

最后再考虑：是否要加更多视觉细节，比如：
- base 侧面的品牌名"LENS CASE"小字
- 镜片表面的小高光
- 液体表面的微涟漪

这些会增加代码量但提升质感。我选择加：
- 镜片高光：在镜片表面加一个小的 MeshBasicMaterial 白色透明圆盘
- 液体涟漪：跳过，太复杂

OK 整理最终代码：

最后我注意到代码中有一个错误：我在 `lidGroup.add(makeLidLabel(...))` 中传入的是 `1.05`，但 lid 在 lidGroup 局部坐标 (0, 0.225, 1.21)。lidLabel 在 lidGroup 局部应该和 lid 在同一位置，所以 z=1.21。但 lid 在 lidGroup 局部 z=1.21，所以 lidLabel 也应该在 z=1.21（lid 顶部）。

等等，lid 是 lidGroup 的子，position (0, 0.225, 1.21)。lidLabel 也是 lidGroup 的子，position (x, 0.248, 1.21)。lidLabel y=0.248，刚好在 lid 顶部 y=0.45（lid 中心 0.225 + 半高 0.225）下方...

等等 lid 中心 y=0.225，半高 0.225，所以 lid 顶部 y=0.45，lid 底部 y=0。lidLabel y=0.248 在 lid 顶部 y=0.45 下方 0.202，这意味着 lidLabel 在 lid 内部下方。

错了！lidLabel 应该在 lid 顶部外侧（朝上的圆盘）。

修正：lidLabel.position.y 应该是 0.45 + epsilon（lid 顶部之上）。
- disc position y = 0.46（lid 顶部上方 0.01）
- plane position y = 0.471（disc 上方 0.011）

让我修正：
```js
function makeLidLabel(x, texture, color) {
  const group = new THREE.Group();
  const disc = new THREE.Mesh(
    new THREE.CylinderGeometry(0.32, 0.32, 0.015, 48),
    new THREE.MeshStandardMaterial({ color: color, roughness: 0.35, metalness: 0.25 })
  );
  disc.position.set(x, 0.46, 1.21);  // lid 顶部上方
  group.add(disc);
  
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(0.5, 0.5),
    new THREE.MeshBasicMaterial({ map: texture, transparent: true, depthWrite: false })
  );
  plane.rotation.x = -Math.PI/2;
  plane.position.set(x, 0.468, 1.21);
  group.add(plane);
  
  return group;
}
```

但 lid 中心 y=0.225，半高 0.225，lid 顶部 y=0.45。lidLabel disc y=0.46，刚好在 lid 顶部上方 0.01。OK。

stripe (装饰条) position y = 0.255 — 错了！应该在 lid 顶部上方。修正为 0.46。

等等，stripe 是 lid 顶部上的装饰条，应该在 lid 顶部外侧。lid 顶部 y=0.45，所以 stripe y=0.45+stripe半高=0.45+0.03=0.48（如果 stripe 高 0.06，半高 0.03）。让 stripe y=0.48 略高于 lid 顶部 0.03。

但 stripe 中心 y=0.48，半高 0.03，所以 stripe 底部 0.45 刚好贴 lid 顶部，stripe 顶部 0.51 在 lid 上方。

让 stripe y=0.485 给一点余量：stripe 底部 0.455 (略高于 lid 顶部 0.45)。

实际上我希望 stripe 是嵌入 lid 顶部的，所以 stripe 中心 y=0.46，半高 0.03，stripe 底部 0.43 (在 lid 内部)，stripe 顶部 0.49 (在 lid 上方 0.04)。

这样 stripe 像装饰条嵌入 lid 顶部表面。我用 stripe y=0.46。

修正代码：
- stripe.position.set(0, 0.46, 1.21)
- disc.position.set(x, 0.475, 1.21) (disc 在 stripe 上方)
- plane.position.set(x, 0.486, 1.21)

等等 disc 是 0.015 高度，半高 0.0075。disc 中心 y=0.475，disc 底部 0.4675 (在 stripe 上方)，disc 顶部 0.4825。OK。

但 stripe 是细长条，disc 在 stripe 上方。disc 是圆形。从俯视图看，stripe 横在 lid 中线，disc 在 stripe 上方（z=1.21）— 等等 disc 在 z=1.21，stripe 也在 z=1.21。它们重叠了。

哦不对，stripe 是装饰条，长度 3.8（沿 x），高度 0.06（沿 y），宽度 0.18（沿 z）。stripe 在 lid 顶部沿 x 方向横着。

disc 是圆盘，在 lid 顶部，位置 (x, y, z) = (±1.05, 0.475, 1.21)。

stripe z=1.21，宽度 0.18，所以 stripe z 范围 1.12 到 1.30。
disc z=1.21，半径 0.32，所以 disc z 范围 0.89 到 1.53。

它们在 z=1.21 附近重叠！stripe 横穿 disc 中心。

但 stripe 高度 0.06 < disc 高度 0.015，所以 stripe 比 disc 厚。stripe 中心 y=0.46，disc 中心 y=0.475。stripe y 范围 0.43 到 0.49，disc y 范围 0.4675 到 0.4825。

它们在 3D 空间中重叠。视觉上会冲突。

解决：把 stripe 改成沿 z 方向短一些，不穿过 disc：
- stripe 长度 1.0（在 disc 之间，沿 x 方向）
- stripe x 范围 -0.5 到 0.5（在两个 disc 之间）
- disc 在 x=±1.05

让 stripe 是 1.0 x 0.06 x 0.18，position (0, 0.46, 1.21)。
这样 stripe 在两个 disc 之间。

或者去掉 stripe，仅靠 disc 装饰。

我决定去掉 stripe，简化。lid 顶部仅靠 L R 圆盘装饰。

或者把 stripe 做成沿 x 方向贯穿 lid 顶部边缘的装饰，但不在 disc 中间。

简化版：去掉 stripe。lid 顶部就是两个 L R 圆盘。

让我去掉 stripe：

```js
// (去掉 stripe 相关代码)
```

OK 最终决定，去掉 stripe 和 lidInner，简化代码。

让我把代码最终整理：

整理后的最终代码大致结构：
1. HTML 头部 + CSS
2. body 内的 UI 元素
3. importmap
4. module script:
   - 场景、相机、渲染器
   - 光照
   - 桌面 + 光晕
   - case group
   - 材质
   - base + 凹槽
   - L R 标签
   - lid + 铰链
   - 交互
   - 渲染循环

我已经规划得差不多了，让我现在写出最终干净版本。

最后一点：考虑到性能和加载速度，我可以省去 OrbitControls 的某些功能，但保留主要的旋转和缩放。controls.enablePan = false 已经禁用平移。

OK 最终输出！

我想再加一个改进：让相机有微小的自动旋转，在用户不交互时。但 controls.autoRotate 可以实现，让用户体验更动态。

controls.autoRotate = true;
controls.autoRotateSpeed = 0.3;

但当用户交互时停止：当 mousedown 时设置 autoRotate = false，过一段时间没有交互后恢复。

简单实现：
```js
let lastInteraction = performance.now();
controls.addEventListener('start', () => {
  controls.autoRotate = false;
  lastInteraction = performance.now();
});
controls.addEventListener('end', () => {
  lastInteraction = performance.now();
});

// 在 animate 中
if (!controls.autoRotate && performance.now() - lastInteraction > 4000) {
  controls.autoRotate = true;
}
```

但这可能干扰用户。我让 autoRotate 默认开启，用户交互时暂停 4 秒，然后恢复。

或者直接关闭 autoRotate，让用户主动控制。我选择关闭，避免干扰。

OK 我决定关闭 autoRotate，但保持进场动画 + 呼吸光 + 状态点脉冲作为动态元素。这样动态元素已经有：
1. 进场动画（盒子从下方升起）
2. 桌面光晕呼吸
3. rim light 呼吸
4. 状态点脉冲
5. 盒盖开合动画
6. Toast 提示

够多了。

OK 最终输出！让我写完整版本：

实际上让我再想一下背景。我用了 body 的径向渐变，但渲染器的 alpha 是 true，所以背景是透明的，会显示 body 的渐变。

但 Three.js scene 没有 background，所以渲染结果是透明的，叠加在 body 渐变上。这应该可以工作。

或者让 scene.background = new THREE.Color(0x06080c) 直接设置背景色。

或者用 body 渐变 + 透明渲染器。我选这个，因为可以让背景与 UI 协调。

OK 最终输出。

让我最后再看一下代码完整性，确保没有遗漏：

1. importmap - 有
2. 场景设置 - 有
3. 光照 - 有
4. 桌面 - 有
5. 眼镜盒 - 有
6. 交互 - 有
7. UI - 有
8. 加载器 - 有
9. 响应式 - 有

OK 开始最终输出！