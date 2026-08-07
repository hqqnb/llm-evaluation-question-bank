用户想要一个隐形眼镜盒的 3D 模型，使用 Three.js。要求：
1. 有醒目的 L 和 R 标识
2. 配有盒盖
3. 用户可以点击盒盖将其打开
4. 单文件 HTML

让我设计一下这个隐形眼镜盒：
- 通常隐形眼镜盒是椭圆形或圆形，有两个圆形凹槽分别放左右眼镜
- 顶部有 L 和 R 标识
- 有一个可翻转的盒盖

我需要做一个视觉上有冲击力的版本。考虑：
- 使用高级的材质和光照
- 盒盖打开有动画
- 添加粒子或环境效果
- 可交互（拖动旋转、点击打开）

设计方向：
- 配色：避免蓝紫色。可以用深邃的青绿色/翡翠色调配橘色或金色点缀
- 让盒子看起来像高端化妆品的感觉
- 添加柔和的环境光反射
- 盒盖打开时有流畅的缓动动画

让我构建一下结构：
1. 盒身：椭圆柱形或圆角矩形
2. 两个圆形凹槽（左右）
3. L 和 R 标识凸起在盒体上
4. 盒盖：可以绕一个轴旋转打开
5. 内部：眼镜凹槽里有水波效果或液体

技术实现：
- Three.js (importmap)
- OrbitControls 用于旋转视角
- Raycaster 用于点击检测
- 盒盖用 Pivot 控制旋转中心

让我开始写代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>隐形眼镜盒 3D 模型</title>
<style>
  /* CSS 部分 */
</style>
</head>
<body>
  <!-- HTML 结构 -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>
  <script type="module">
    // JS 部分
  </script>
</body>
</html>
```

详细设计：

**盒子主体**：
- 使用圆角矩形盒子
- 颜色：深翡翠绿/墨绿色 + 金属感
- 顶部有两个圆形凹槽
- 凹槽旁边有凸起的 L 和 R 字母

**盒盖**：
- 通过一个 hinge（铰链）轴连接
- 点击盒盖时绕铰链旋转 110 度打开
- 使用 Tween 或自己写缓动函数

**内部**：
- 凹槽里有蓝色液体效果（隐形眼镜护理液）
- 表面有反光
- 可以放置两个透明的圆形眼镜（用 TorusGeometry 表示）

**环境**：
- 暗色背景配渐变
- 添加柔光地板
- 添加聚光灯和环境光
- 添加粒子效果

**UI**：
- 顶部有标题
- 底部有提示信息
- 控制提示

颜色方案：
- 背景：深炭灰 + 翡翠绿渐变
- 盒身：墨绿色 (#1a3a3a) + 金属质感
- 盒盖：稍亮一些的绿色 + 渐变高光
- L/R 标识：金色/橘色 (#f5a623)
- 液体：青蓝色 (#4ecdc4)
- 字体颜色：白色/暖白

让我开始写完整代码：

```javascript
// 主要变量
let scene, camera, renderer, controls;
let caseGroup, lidGroup, lidPivot;
let isOpen = false;
let lidAngle = 0;
let targetLidAngle = 0;

// 初始化
function init() {
  // 场景
  scene = new THREE.Scene();
  scene.background = null; // 透明背景，让 CSS 渐变显示
  
  // 相机
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(4, 4, 6);
  camera.lookAt(0, 0, 0);
  
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  // 灯光
  setupLights();
  
  // 创建盒子
  createCase();
  
  // 控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.minDistance = 4;
  controls.maxDistance = 15;
  controls.maxPolarAngle = Math.PI * 0.55;
  
  // 事件
  window.addEventListener('resize', onResize);
  renderer.domElement.addEventListener('click', onClick);
  
  animate();
}

function setupLights() {
  // 环境光
  const ambient = new THREE.AmbientLight(0xffffff, 0.3);
  scene.add(ambient);
  
  // 主光
  const keyLight = new THREE.DirectionalLight(0xffffff, 1.5);
  keyLight.position.set(5, 8, 5);
  keyLight.castShadow = true;
  keyLight.shadow.mapSize.set(2048, 2048);
  keyLight.shadow.camera.left = -5;
  keyLight.shadow.camera.right = 5;
  keyLight.shadow.camera.top = 5;
  keyLight.shadow.camera.bottom = -5;
  scene.add(keyLight);
  
  // 补光（青色）
  const fillLight = new THREE.PointLight(0x4ecdc4, 1, 20);
  fillLight.position.set(-4, 3, -2);
  scene.add(fillLight);
  
  // 边缘光（橘色）
  const rimLight = new THREE.PointLight(0xff6b35, 0.8, 15);
  rimLight.position.set(3, 2, -4);
  scene.add(rimLight);
}

function createCase() {
  caseGroup = new THREE.Group();
  
  // 盒身
  const caseGeo = new THREE.BoxGeometry(4, 1, 2.5, 4, 4, 4);
  // 圆角处理...
  // 或者使用 RoundedBoxGeometry
  
  // 简化：使用普通 BoxGeometry 配合材质
  
  // ...
}
```

我需要使用 RoundedBoxGeometry 来制作圆角盒子，看起来更高级。

Three.js 的 examples/jsm 里有 RoundedBoxGeometry。

让我设计完整版本：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
```

如果用 TextGeometry 需要加载字体文件，可能不太稳定。让我直接用平面贴图 Canvas 来生成 L 和 R 字母贴在盒子上。这样更可靠。

或者用 ExtrudeGeometry 从 Shape 创建 3D 字母。

让我用 CanvasTexture 的方法：
1. 创建一个 canvas
2. 在 canvas 上绘制字母 L 和 R
3. 用作纹理贴在盒子表面
4. 或者作为 alphaMap 让字母凸起

凸起字母的话需要几何操作。简化方案：在盒子顶部用平面贴上 L 和 R 纹理。

更好的方案：直接用 Shape + ExtrudeGeometry 创建立体字母。

```javascript
function createLetter(letter) {
  // 用 canvas 生成字母路径
  const canvas = document.createElement('canvas');
  canvas.width = 256;
  canvas.height = 256;
  const ctx = canvas.getContext('2d');
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 200px Arial';
  ctx.textBaseline = 'middle';
  ctx.textAlign = 'center';
  ctx.fillText(letter, 128, 128);
  
  // 从 canvas 像素提取路径会复杂
  // 直接用 Three.js Shape 手绘
}
```

最简单：直接用 CanvasTexture 把字母绘制成贴图，贴在盒子顶部凹槽旁边。

我决定：
- 盒身用 RoundedBoxGeometry（更精致）
- L 和 R 标识用 CanvasTexture 贴在盒身顶面，并且让字母用发光的金色/橘色
- 盒盖也是圆角盒子，通过 hinge 打开

让我写一个完整版本：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';

// 全局变量
let scene, camera, renderer, controls;
let caseGroup, lidPivot, lidMesh;
let leftLiquid, rightLiquid;
let isOpen = false;
let lidRotation = 0;
let targetRotation = 0;
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let isDragging = false;
let mouseDownPos = { x: 0, y: 0 };

// 颜色
const COLORS = {
  caseBody: 0x1a4d4a,  // 墨绿
  caseAccent: 0x0d2e2c,
  lidColor: 0x2a6e6a,
  letter: 0xf5a623,  // 金色
  liquid: 0x4ecdc4,
};

function init() {
  // ...
  createCase();
  // ...
}

function createCase() {
  caseGroup = new THREE.Group();
  
  // === 盒身 ===
  const bodyGeo = new RoundedBoxGeometry(4, 1.2, 2.4, 8, 0.25);
  const bodyMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.caseBody,
    metalness: 0.6,
    roughness: 0.3,
    clearcoat: 0.5,
    clearcoatRoughness: 0.2,
  });
  const body = new THREE.Mesh(bodyGeo, bodyMat);
  body.position.y = 0;
  body.castShadow = true;
  body.receiveShadow = true;
  caseGroup.add(body);
  
  // === 内部凹槽（两个圆形凹槽） ===
  const wellGeo = new THREE.CylinderGeometry(0.65, 0.55, 0.5, 64);
  const wellMat = new THREE.MeshPhysicalMaterial({
    color: 0x0a1f1e,
    metalness: 0.3,
    roughness: 0.4,
  });
  
  const leftWell = new THREE.Mesh(wellGeo, wellMat);
  leftWell.position.set(-1, 0.36, 0);
  leftWell.receiveShadow = true;
  caseGroup.add(leftWell);
  
  const rightWell = new THREE.Mesh(wellGeo, wellMat);
  rightWell.position.set(1, 0.36, 0);
  rightWell.receiveShadow = true;
  caseGroup.add(rightWell);
  
  // === 液体（护理液） ===
  const liquidGeo = new THREE.CircleGeometry(0.55, 64);
  const liquidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.liquid,
    metalness: 0.0,
    roughness: 0.05,
    transmission: 0.6,
    thickness: 0.5,
    ior: 1.33,
    side: THREE.DoubleSide,
  });
  
  leftLiquid = new THREE.Mesh(liquidGeo, liquidMat);
  leftLiquid.rotation.x = -Math.PI / 2;
  leftLiquid.position.set(-1, 0.55, 0);
  caseGroup.add(leftLiquid);
  
  rightLiquid = new THREE.Mesh(liquidGeo, liquidMat);
  rightLiquid.rotation.x = -Math.PI / 2;
  rightLiquid.position.set(1, 0.55, 0);
  caseGroup.add(rightLiquid);
  
  // === L 和 R 标识（贴图） ===
  const letterTexture = createLetterTexture('L');
  // ... 这里需要分别贴 L 和 R
  
  // 或者用平面贴在盒子表面
  
  // === 盒盖 ===
  // 盒盖用 pivot 实现铰链效果
  lidPivot = new THREE.Group();
  lidPivot.position.set(-2, 0.6, 0); // 铰链在盒子左侧
  caseGroup.add(lidPivot);
  
  const lidGeo = new RoundedBoxGeometry(4, 0.4, 2.4, 8, 0.2);
  const lidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lidColor,
    metalness: 0.7,
    roughness: 0.25,
    clearcoat: 0.8,
    clearcoatRoughness: 0.1,
  });
  lidMesh = new THREE.Mesh(lidGeo, lidMat);
  lidMesh.position.set(2, 0.2, 0); // 相对 pivot 偏移
  lidMesh.castShadow = true;
  lidMesh.receiveShadow = true;
  lidPivot.add(lidMesh);
  
  // 盒盖上的 L 和 R 标识（凸起）
  // ...
  
  scene.add(caseGroup);
}

function createLetterTexture(letter, color = '#f5a623') {
  const canvas = document.createElement('canvas');
  canvas.width = 256;
  canvas.height = 256;
  const ctx = canvas.getContext('2d');
  
  // 透明背景
  ctx.clearRect(0, 0, 256, 256);
  
  // 字母
  ctx.font = 'bold 200px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  
  // 渐变
  const gradient = ctx.createLinearGradient(0, 0, 256, 256);
  gradient.addColorStop(0, '#ffd700');
  gradient.addColorStop(1, '#ff6b35');
  ctx.fillStyle = gradient;
  
  ctx.shadowColor = '#ff6b35';
  ctx.shadowBlur = 20;
  ctx.fillText(letter, 128, 128);
  
  const texture = new THREE.CanvasTexture(canvas);
  return texture;
}
```

让我重新整体规划一下：

**布局**：
- 隐形眼镜盒是水平放置的长方形盒子
- 长 4，宽 2.4，高 1.2（盒身）+ 0.4（盒盖）
- 盒盖在顶部，通过左侧的铰链向后翻转打开
- 内部有两个圆形凹槽（左右各一），凹槽里有蓝色液体
- L 标识在左凹槽旁边，R 标识在右凹槽旁边

**L 和 R 标识**：
- 用平面贴图的方式，将字母贴在盒盖顶面（醒目位置）
- 也可以贴在盒身前面（侧边）
- 我想让它更醒目，可以做成发光的，使用 emissive map

或者更好的方式：使用 ExtrudeGeometry 让字母真正 3D 凸起。

我会做：
1. 用 Shape 绘制字母路径（手写 L 和 R 的路径）
2. 用 ExtrudeGeometry 让字母 3D 化
3. 让字母贴在盒盖顶面

让我手绘 L 和 R 的路径：

L 字母路径：
```
moveTo(0, 0)
lineTo(0, 1)  // 左竖线
lineTo(0.7, 1)
lineTo(0.7, 0.1)
lineTo(0.15, 0.1)
lineTo(0.15, 0)
close
```

R 字母稍微复杂：
```
moveTo(0, 0)
lineTo(0, 1)
lineTo(0.5, 1)
quadraticCurveTo(0.8, 1, 0.8, 0.7)
quadraticCurveTo(0.8, 0.4, 0.5, 0.4)
// 这里继续画腿
lineTo(0.4, 0.4)
... 
```

太复杂了。我用 CanvasTexture 简单地绘制字母，贴在 BoxGeometry 表面，或者作为单独的平面贴在盒盖上方。

我决定方案：
- 用 CanvasTexture 创建一个包含 L 和 R 的纹理
- 用单独的 PlaneGeometry 贴在盒盖上方稍微凸起的位置
- 同时让平面发光

或者：把 L 和 R 直接画在盒盖顶部的纹理上。

更好的方案：用 ExtrudeGeometry 配合 THREE.Shape，绘制字母的形状。这样字母是真正 3D 的，看起来更高级。

我用 Path API 来绘制字母：

```javascript
function createLetterGeometry(letter) {
  const shape = new THREE.Shape();
  
  if (letter === 'L') {
    shape.moveTo(0, 0);
    shape.lineTo(0.1, 0);
    shape.lineTo(0.1, 0.85);
    shape.lineTo(0.55, 0.85);
    shape.lineTo(0.55, 0.95);
    shape.lineTo(0, 0.95);
    shape.closePath();
  } else if (letter === 'R') {
    // R 形状
    shape.moveTo(0, 0);
    shape.lineTo(0.1, 0);
    shape.lineTo(0.1, 0.4);
    shape.lineTo(0.4, 0.4);
    shape.lineTo(0.55, 0.4);
    // 用 quadraticCurveTo 画圆弧部分
    shape.absarc(0.4, 0.55, 0.18, -Math.PI/2, Math.PI/2, true);
    // 然后是 R 的腿
    shape.lineTo(0.25, 0.95);
    shape.lineTo(0.15, 0.95);
    shape.lineTo(0.3, 0.6);
    shape.lineTo(0.1, 0.6);
    shape.lineTo(0.1, 0.95);
    shape.lineTo(0, 0.95);
    shape.closePath();
  }
  
  return new THREE.ExtrudeGeometry(shape, {
    depth: 0.08,
    bevelEnabled: true,
    bevelThickness: 0.01,
    bevelSize: 0.01,
    bevelSegments: 3,
  });
}
```

手画 R 比较复杂，可能形状不准。让我用更简单的方式：直接用 CanvasTexture 绘制字母，然后作为 emissiveMap 和 map 贴在 PlaneGeometry 上。这样字母能发光，看起来醒目。

最终方案：
1. 盒身和盒盖用 RoundedBoxGeometry
2. L 和 R 字母用 CanvasTexture + PlaneGeometry 贴在盒盖顶部
3. 字母有发光效果
4. 点击盒盖触发开盖动画
5. 内部有凹槽和液体效果

让我写完整代码：

```javascript
// 创建字母贴图
function createLetterTexture(letter) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  // 透明背景
  ctx.clearRect(0, 0, size, size);
  
  // 外发光
  ctx.shadowColor = 'rgba(255, 200, 50, 0.9)';
  ctx.shadowBlur = 40;
  ctx.shadowOffsetX = 0;
  ctx.shadowOffsetY = 0;
  
  // 渐变填充
  const gradient = ctx.createLinearGradient(0, 0, size, size);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff6b35');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 380px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, size/2, size/2 + 20);
  
  // 再画一次描边增加清晰度
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 240, 200, 0.6)';
  ctx.lineWidth = 6;
  ctx.strokeText(letter, size/2, size/2 + 20);
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  return texture;
}

// 创建字母标识
function createLetterMark(letter, x) {
  const texture = createLetterTexture(letter);
  
  // 平面贴图
  const planeGeo = new THREE.PlaneGeometry(0.8, 0.8);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
  });
  const plane = new THREE.Mesh(planeGeo, planeMat);
  plane.rotation.x = -Math.PI / 2;
  plane.position.set(x, 0.81, 0); // 在盒盖顶部
  
  return plane;
}
```

这样字母是平面的，但是有发光效果（在暗背景下）。

实际上为了醒目，我考虑：
- 字母贴在盒盖顶部，凸起一点
- 使用 emissive 材质让它发光
- 用 PointLight 局部照亮字母周围

让我做最终版本。还有一点：当盒盖打开时，应该看到盒身的内部（凹槽）。所以盒盖打开后字母会跟着翻转。我希望字母也显示在盒身的底部（凹槽旁边），这样无论盒盖开关，都能看到 L 和 R。

或者只放在盒盖上。但盒盖打开后字母就不在原位置了。

最佳方案：在盒身顶面（凹槽旁边）也放字母标识，这样标识固定在盒身上。盒盖上可以放品牌名或其他装饰。

不过用户要求"醒目的 L 和 R 标识"，所以应该放在盒身顶面凹槽旁边，这样最直观。

让我重新设计：
- 盒身顶部有 L 和 R 标识（凹槽旁边）
- 标识凸起一点，金色发光
- 盒盖是平的，可以加一点装饰（比如品牌名字）

不过这样盒盖打开时字母就盖在盒盖下面看不见了... 

不，盒盖在顶部，盒身顶面有凹槽和字母。盒盖关闭时盖在盒身顶部上方，把凹槽和字母盖住。这样不行。

正确设计：
- 盒身顶面是凹槽
- L 和 R 标识放在凹槽旁边（盒身顶面）
- 盒盖是中空的？不对，盒盖应该是封闭的盖子

让我想想真实的眼镜盒：
- 盒身：有凹槽，凹槽旁边印有 L 和 R 字母
- 盒盖：覆盖在盒身顶部，铰链在一侧
- 关闭时盒盖盖住凹槽和字母
- 打开后才能看到字母和凹槽

所以字母在盒身内部，打开盒盖后才能看到。这是真实设计。

但用户要求"醒目的 L 和 R 标识"，我希望它们一直可见。所以可以：
- 在盒身顶面凹槽旁有 L 和 R
- 在盒盖顶面也印有 L 和 R（醒目位置）
- 关闭时盒盖的 L 和 R 可见
- 打开时盒身的 L 和 R 可见

或者只在盒身顶面放，但盒盖打开后才能看到。这种方案符合实际产品。

让我做两个版本：盒盖顶面有大的 L 和 R 字母（始终可见，醒目），盒身凹槽旁边也有小的 L 和 R（打开后可见）。

实际上，仔细想想：隐形眼镜盒的 L 和 R 标识通常在盒身内部凹槽旁边。但当盖子盖上时，外面也通常会有标识（在盖子上）。所以两个地方都有是合理的。

最终方案：
- 盒身顶面凹槽旁边有 L 和 R 标识（小一点）
- 盒盖顶面也有 L 和 R 标识（大一些，醒目）

实现：
1. 创建字母的 CanvasTexture
2. 用 PlaneGeometry 贴在盒身顶面（凹槽旁）
3. 用 PlaneGeometry 贴在盒盖顶面（醒目位置）
4. 配合发光效果

完整代码框架：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>隐形眼镜盒 3D 模型</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  
  :root {
    --bg-dark: #050a0c;
    --bg-mid: #0a1f1c;
    --accent: #f5a623;
    --accent-2: #4ecdc4;
    --text: #e8f4f0;
    --text-dim: #7a9994;
  }
  
  body {
    background: radial-gradient(ellipse at 50% 30%, var(--bg-mid) 0%, var(--bg-dark) 70%);
    color: var(--text);
    font-family: 'Inter', system-ui, sans-serif;
    overflow: hidden;
    height: 100vh;
  }
  
  #canvas-container {
    position: fixed;
    inset: 0;
    z-index: 1;
  }
  
  .header {
    position: fixed;
    top: 0; left: 0; right: 0;
    padding: 28px 40px;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    pointer-events: none;
  }
  
  .brand {
    font-family: 'Cormorant Garamond', serif;
    font-size: 22px;
    font-weight: 500;
    letter-spacing: 4px;
    color: var(--accent);
    text-transform: uppercase;
  }
  
  .brand small {
    display: block;
    font-size: 11px;
    color: var(--text-dim);
    letter-spacing: 6px;
    margin-top: 4px;
  }
  
  .status {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--text-dim);
    text-align: right;
    line-height: 1.6;
  }
  
  .status .label {
    color: var(--accent-2);
  }
  
  .footer {
    position: fixed;
    bottom: 0; left: 0; right: 0;
    padding: 28px 40px;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    pointer-events: none;
  }
  
  .controls-hint {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    color: var(--text-dim);
    line-height: 1.8;
  }
  
  .controls-hint .key {
    color: var(--accent);
    margin-right: 8px;
  }
  
  .action-btn {
    pointer-events: auto;
    background: rgba(245, 166, 35, 0.1);
    border: 1px solid rgba(245, 166, 35, 0.4);
    color: var(--accent);
    padding: 14px 28px;
    font-family: 'Inter', sans-serif;
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 3px;
    text-transform: uppercase;
    cursor: pointer;
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
    border-radius: 2px;
  }
  
  .action-btn:hover {
    background: rgba(245, 166, 35, 0.25);
    border-color: var(--accent);
    box-shadow: 0 0 30px rgba(245, 166, 35, 0.3);
  }
  
  /* 装饰元素 */
  .corner-mark {
    position: fixed;
    width: 24px;
    height: 24px;
    border: 1px solid rgba(245, 166, 35, 0.4);
    z-index: 5;
    pointer-events: none;
  }
  
  .corner-mark.tl { top: 20px; left: 20px; border-right: none; border-bottom: none; }
  .corner-mark.tr { top: 20px; right: 20px; border-left: none; border-bottom: none; }
  .corner-mark.bl { bottom: 20px; left: 20px; border-right: none; border-top: none; }
  .corner-mark.br { bottom: 20px; right: 20px; border-left: none; border-top: none; }
  
  /* 噪点 */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 2;
    opacity: 0.04;
    background-image: url("data:image/svg+xml,...");
    mix-blend-mode: overlay;
  }
  
  /* 提示 toast */
  .toast {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(10, 31, 28, 0.95);
    border: 1px solid var(--accent);
    color: var(--text);
    padding: 16px 28px;
    font-size: 13px;
    letter-spacing: 2px;
    z-index: 100;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.4s, transform 0.4s;
    backdrop-filter: blur(20px);
  }
  
  .toast.show {
    opacity: 1;
    transform: translate(-50%, calc(-50% - 20px));
  }
</style>
</head>
<body>
  <div id="canvas-container"></div>
  
  <div class="corner-mark tl"></div>
  <div class="corner-mark tr"></div>
  <div class="corner-mark bl"></div>
  <div class="corner-mark br"></div>
  
  <header class="header">
    <div class="brand">
      LUMINA
      <small>OPTICAL · CASE</small>
    </div>
    <div class="status">
      <div><span class="label">MODEL</span> L-7 Gemini</div>
      <div><span class="label">STATE</span> <span id="state-text">CLOSED</span></div>
    </div>
  </header>
  
  <footer class="footer">
    <div class="controls-hint">
      <div><span class="key">DRAG</span>旋转视角</div>
      <div><span class="key">SCROLL</span>缩放</div>
      <div><span class="key">CLICK</span>点击盒盖打开</div>
    </div>
    <button class="action-btn" id="toggle-btn">开启盒盖</button>
  </footer>
  
  <div class="toast" id="toast"></div>
  
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
    
    // 全局变量
    let scene, camera, renderer, controls;
    let caseGroup, lidPivot;
    let isOpen = false;
    let lidAngle = 0;
    let targetLidAngle = 0;
    let isAnimating = false;
    
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    let mouseDownPos = { x: 0, y: 0 };
    let mouseDownTime = 0;
    
    // ... 完整代码
  </script>
</body>
</html>
```

我需要避免使用 Inter 字体（要求避免常见字体作为主标识字体）。让我换成 'Space Grotesk' 或 'Outfit' 作为 UI 字体，'Cormorant Garamond' 作为衬线字体。

完整代码整理一下，让我把整个 HTML 写出来：

实际上，让我先思考下交互细节：
1. 用户点击盒盖 -> 盒盖打开（绕铰链旋转）
2. 再次点击 -> 盒盖关闭
3. 按钮也可以切换

铰链设计：
- lidPivot 是一个 Group，位置在盒子的一侧（比如左侧）
- lidMesh 相对 lidPivot 偏移到右侧（覆盖盒子）
- 旋转 lidPivot 时，lidMesh 跟着旋转
- 关闭：lidPivot.rotation.x = 0
- 打开：lidPivot.rotation.x = -110° (向后翻)

或者让铰链在盒子后方：
- lidPivot 位置在盒子后边缘上方
- lidMesh 相对 lidPivot 向前偏移
- 旋转 lidPivot.x = -110° 时，盒盖向后翻起

这样更符合实际产品。

让我考虑铰链在盒子后方：
- 盒身中心在 (0,0,0)
- 盒身尺寸：长 4（x方向），宽 2.4（z方向），高 1.2（y方向）
- 盒身顶部在 y = 0.6
- 铰链在盒子后边缘上方：lidPivot.position = (0, 0.6, -1.2)（z 负方向是后方）
- lidMesh 相对 lidPivot 偏移 (0, 0.1, 1.2)
- 关闭时 lidPivot.rotation.x = 0
- 打开时 lidPivot.rotation.x = -120° (绕 x 轴旋转，盒盖向后翻)

Hmm，z 轴方向我得想清楚。OrbitControls 默认相机看向 -z 方向。

让我重新设置：
- 盒身：x 是长度方向，y 是高度方向，z 是深度方向
- 相机在 (5, 4, 6) 看向原点
- 盒子从相机角度看，长边在 x 方向
- 铰链在盒子的"后方"，即 z = -1.2 处（远离相机）
- 盒盖打开时向后翻（远离相机）

铰链位置：(0, 0.7, -1.2)
盒盖相对铰链的位置：(0, 0.1, 1.2)
旋转：lidPivot.rotation.x = -120° (Math.PI * 2/3)

打开后盒盖会向后上方翻起。

测试旋转方向：当 rotation.x = -120° 时，lidMesh 在 lidPivot 坐标系中的位置 (0, 0.1, 1.2) 经过旋转后，z 变成负的（向后），y 变正（向上）。所以盒盖会向后上方翻起。✓

让我开始写完整代码。先思考动画：

```javascript
function animate() {
  requestAnimationFrame(animate);
  
  // 平滑过渡 lidAngle 到 targetLidAngle
  if (Math.abs(lidAngle - targetLidAngle) > 0.001) {
    lidAngle += (targetLidAngle - lidAngle) * 0.08;
    lidPivot.rotation.x = lidAngle;
    isAnimating = true;
  } else {
    if (isAnimating) {
      lidAngle = targetLidAngle;
      lidPivot.rotation.x = lidAngle;
      isAnimating = false;
    }
  }
  
  // 液体波动
  // ...
  
  // 粒子
  // ...
  
  controls.update();
  renderer.render(scene, camera);
}

function toggleLid() {
  isOpen = !isOpen;
  targetLidAngle = isOpen ? -Math.PI * 0.7 : 0;
  updateUI();
}

function updateUI() {
  document.getElementById('state-text').textContent = isOpen ? 'OPEN' : 'CLOSED';
  document.getElementById('toggle-btn').textContent = isOpen ? '关闭盒盖' : '开启盒盖';
}
```

需要点击检测：

```javascript
function onCanvasClick(event) {
  // 计算鼠标 NDC 坐标
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  
  // 检测盒盖
  const intersects = raycaster.intersectObject(lidMesh, true);
  if (intersects.length > 0) {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  }
}
```

但是要区分点击和拖动：
- mousedown 记录位置和时间
- mouseup 时如果位置变化小且时间短，视为点击

```javascript
function onMouseDown(event) {
  mouseDownPos = { x: event.clientX, y: event.clientY };
  mouseDownTime = Date.now();
}

function onMouseUp(event) {
  const dx = event.clientX - mouseDownPos.x;
  const dy = event.clientY - mouseDownPos.y;
  const distance = Math.sqrt(dx*dx + dy*dy);
  const duration = Date.now() - mouseDownTime;
  
  if (distance < 5 && duration < 500) {
    // 视为点击
    handleClick(event);
  }
}
```

好，让我开始写完整的代码。

补充：
- 加一个底部反射地面，让盒子有"漂浮"的感觉
- 加一些环境粒子（像灰尘飘动）
- 加一个 spotlight 效果

地板用 Reflector 或者普通 Mesh + 渐变透明：

```javascript
// 地板
const floorGeo = new THREE.PlaneGeometry(50, 50);
const floorMat = new THREE.MeshStandardMaterial({
  color: 0x0a1f1c,
  metalness: 0.8,
  roughness: 0.4,
  transparent: true,
  opacity: 0.6,
});
const floor = new THREE.Mesh(floorGeo, floorMat);
floor.rotation.x = -Math.PI / 2;
floor.position.y = -1.5;
floor.receiveShadow = true;
scene.add(floor);

// 加一个圆形渐变在地板中央
```

或者用 shadow only 的地板：

```javascript
import { ShadowMaterial } from 'three';
const floorMat = new THREE.ShadowMaterial({ opacity: 0.5 });
```

这样地板透明只显示阴影。

让我用 ShadowMaterial 来做隐形地板，再加一个发光的圆形地板装饰。

环境贴图：用 PMREMGenerator 创建一个程序化的环境贴图，让金属材质看起来更真实。或者直接用 RoomEnvironment。

```javascript
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
```

但是这样背景会被覆盖，需要设置 scene.background = null 或者保留透明。

让我整理一下完整代码：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

// === 全局变量 ===
let scene, camera, renderer, controls;
let caseGroup, lidPivot, lidMesh;
let leftLiquid, rightLiquid;
let particleSystem;

let isOpen = false;
let lidAngle = 0;
let targetLidAngle = 0;
let isAnimating = false;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let mouseDownPos = { x: 0, y: 0 };
let mouseDownTime = 0;
let hoveredLid = false;

const clock = new THREE.Clock();

// === 颜色 ===
const COLORS = {
  caseBody: 0x0d3a36,
  caseBodyAccent: 0x1a5651,
  lidColor: 0x164440,
  lidAccent: 0x2a7a73,
  letterGold: 0xf5a623,
  letterGoldBright: 0xffd966,
  liquid: 0x4ecdc4,
  liquidDeep: 0x2a8a86,
  innerWell: 0x07201d,
};

// === 初始化 ===
function init() {
  // 场景
  scene = new THREE.Scene();
  
  // 相机
  camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(5, 3.5, 6.5);
  
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.1;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  // 环境贴图
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
  
  // 灯光
  setupLights();
  
  // 地板
  createFloor();
  
  // 盒子
  createCase();
  
  // 粒子
  createParticles();
  
  // 控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.06;
  controls.minDistance = 4;
  controls.maxDistance = 12;
  controls.maxPolarAngle = Math.PI * 0.5;
  controls.minPolarAngle = Math.PI * 0.15;
  controls.target.set(0, 0.2, 0);
  
  // 事件
  window.addEventListener('resize', onResize);
  renderer.domElement.addEventListener('pointerdown', onPointerDown);
  renderer.domElement.addEventListener('pointerup', onPointerUp);
  renderer.domElement.addEventListener('pointermove', onPointerMove);
  document.getElementById('toggle-btn').addEventListener('click', () => {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  });
  
  animate();
}

function setupLights() {
  // 环境光
  const ambient = new THREE.AmbientLight(0xffffff, 0.25);
  scene.add(ambient);
  
  // 主光（暖白）
  const keyLight = new THREE.DirectionalLight(0xfff5e0, 1.8);
  keyLight.position.set(4, 8, 5);
  keyLight.castShadow = true;
  keyLight.shadow.mapSize.set(2048, 2048);
  keyLight.shadow.camera.left = -4;
  keyLight.shadow.camera.right = 4;
  keyLight.shadow.camera.top = 4;
  keyLight.shadow.camera.bottom = -4;
  keyLight.shadow.camera.near = 0.5;
  keyLight.shadow.camera.far = 20;
  keyLight.shadow.bias = -0.0005;
  scene.add(keyLight);
  
  // 补光（青色，从左侧）
  const fillLight = new THREE.PointLight(0x4ecdc4, 1.2, 15);
  fillLight.position.set(-4, 3, 2);
  scene.add(fillLight);
  
  // 边缘光（金色，从后方）
  const rimLight = new THREE.PointLight(0xff8c42, 1, 12);
  rimLight.position.set(2, 2, -4);
  scene.add(rimLight);
  
  // 顶部聚光
  const spotLight = new THREE.SpotLight(0xffffff, 1.5, 15, Math.PI * 0.2, 0.5, 1);
  spotLight.position.set(0, 8, 0);
  spotLight.target.position.set(0, 0, 0);
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.set(1024, 1024);
  scene.add(spotLight);
  scene.add(spotLight.target);
}

function createFloor() {
  // 阴影地板
  const shadowFloorGeo = new THREE.PlaneGeometry(30, 30);
  const shadowFloorMat = new THREE.ShadowMaterial({ opacity: 0.4 });
  const shadowFloor = new THREE.Mesh(shadowFloorGeo, shadowFloorMat);
  shadowFloor.rotation.x = -Math.PI / 2;
  shadowFloor.position.y = -0.65;
  shadowFloor.receiveShadow = true;
  scene.add(shadowFloor);
  
  // 装饰圆形发光地板
  const glowFloorGeo = new THREE.CircleGeometry(4, 64);
  const glowFloorMat = new THREE.MeshBasicMaterial({
    color: 0x4ecdc4,
    transparent: true,
    opacity: 0.08,
    side: THREE.DoubleSide,
  });
  const glowFloor = new THREE.Mesh(glowFloorGeo, glowFloorMat);
  glowFloor.rotation.x = -Math.PI / 2;
  glowFloor.position.y = -0.64;
  scene.add(glowFloor);
  
  // 外圈
  const ringGeo = new THREE.RingGeometry(3.8, 4, 64);
  const ringMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.3,
    side: THREE.DoubleSide,
  });
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.x = -Math.PI / 2;
  ring.position.y = -0.63;
  scene.add(ring);
}

function createCase() {
  caseGroup = new THREE.Group();
  
  // === 盒身 ===
  const bodyGeo = new RoundedBoxGeometry(4, 1.2, 2.4, 8, 0.18);
  const bodyMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.caseBody,
    metalness: 0.5,
    roughness: 0.35,
    clearcoat: 0.6,
    clearcoatRoughness: 0.25,
  });
  const body = new THREE.Mesh(bodyGeo, bodyMat);
  body.position.y = 0;
  body.castShadow = true;
  body.receiveShadow = true;
  caseGroup.add(body);
  
  // === 凹槽（左右两个圆形凹槽） ===
  // 用 CylinderGeometry 减去盒子的顶部
  // 这里直接放置凹陷的圆柱
  const wellGeo = new THREE.CylinderGeometry(0.7, 0.6, 0.5, 64, 1, true);
  const wellMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.innerWell,
    metalness: 0.4,
    roughness: 0.3,
    side: THREE.DoubleSide,
  });
  
  // 圆柱外壁（凹槽侧壁）
  const leftWellOuter = new THREE.Mesh(wellGeo, wellMat);
  leftWellOuter.position.set(-1.05, 0.36, 0);
  leftWellOuter.castShadow = true;
  leftWellOuter.receiveShadow = true;
  caseGroup.add(leftWellOuter);
  
  const rightWellOuter = new THREE.Mesh(wellGeo, wellMat);
  rightWellOuter.position.set(1.05, 0.36, 0);
  rightWellOuter.castShadow = true;
  rightWellOuter.receiveShadow = true;
  caseGroup.add(rightWellOuter);
  
  // 凹槽底部
  const wellBottomGeo = new THREE.CircleGeometry(0.6, 64);
  const wellBottomMat = new THREE.MeshStandardMaterial({
    color: 0x051815,
    metalness: 0.3,
    roughness: 0.5,
  });
  
  const leftWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  leftWellBottom.rotation.x = -Math.PI / 2;
  leftWellBottom.position.set(-1.05, 0.115, 0);
  caseGroup.add(leftWellBottom);
  
  const rightWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  rightWellBottom.rotation.x = -Math.PI / 2;
  rightWellBottom.position.set(1.05, 0.115, 0);
  caseGroup.add(rightWellBottom);
  
  // === 液体（护理液表面） ===
  const liquidGeo = new THREE.CircleGeometry(0.58, 64);
  const liquidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.liquid,
    metalness: 0.0,
    roughness: 0.05,
    transmission: 0.3,
    thickness: 0.4,
    ior: 1.33,
    side: THREE.DoubleSide,
    emissive: COLORS.liquid,
    emissiveIntensity: 0.15,
  });
  
  leftLiquid = new THREE.Mesh(liquidGeo, liquidMat);
  leftLiquid.rotation.x = -Math.PI / 2;
  leftLiquid.position.set(-1.05, 0.52, 0);
  caseGroup.add(leftLiquid);
  
  rightLiquid = new THREE.Mesh(liquidGeo, liquidMat.clone());
  rightLiquid.rotation.x = -Math.PI / 2;
  rightLiquid.position.set(1.05, 0.52, 0);
  caseGroup.add(rightLiquid);
  
  // 隐形眼镜（透明的圆盘）
  const lensGeo = new THREE.TorusGeometry(0.35, 0.04, 16, 64);
  const lensMat = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
    metalness: 0,
    roughness: 0.05,
    transmission: 0.9,
    thickness: 0.2,
    ior: 1.4,
    side: THREE.DoubleSide,
    clearcoat: 1,
    clearcoatRoughness: 0.1,
  });
  
  const leftLens = new THREE.Mesh(lensGeo, lensMat);
  leftLens.rotation.x = -Math.PI / 2;
  leftLens.position.set(-1.05, 0.48, 0);
  caseGroup.add(leftLens);
  
  const rightLens = new THREE.Mesh(lensGeo, lensMat);
  rightLens.rotation.x = -Math.PI / 2;
  rightLens.position.set(1.05, 0.48, 0);
  caseGroup.add(rightLens);
  
  // === L 和 R 标识（在凹槽旁边的盒身顶面） ===
  const letterL = createLetterPlane('L');
  letterL.position.set(-1.05, 0.61, 0.85);
  letterL.rotation.x = -Math.PI / 2;
  caseGroup.add(letterL);
  
  const letterR = createLetterPlane('R');
  letterR.position.set(1.05, 0.61, 0.85);
  letterR.rotation.x = -Math.PI / 2;
  caseGroup.add(letterR);
  
  // 内侧的字母（凹槽另一侧）
  const letterL2 = createLetterPlane('L', 0.6);
  letterL2.position.set(-1.05, 0.61, -0.85);
  letterL2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterL2);
  
  const letterR2 = createLetterPlane('R', 0.6);
  letterR2.position.set(1.05, 0.61, -0.85);
  letterR2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterR2);
  
  // === 盒盖 ===
  lidPivot = new THREE.Group();
  lidPivot.position.set(0, 0.6, -1.2); // 铰链在盒子后方上边缘
  caseGroup.add(lidPivot);
  
  const lidGeo = new RoundedBoxGeometry(4, 0.4, 2.4, 8, 0.15);
  const lidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lidColor,
    metalness: 0.7,
    roughness: 0.2,
    clearcoat: 0.9,
    clearcoatRoughness: 0.1,
  });
  lidMesh = new THREE.Mesh(lidGeo, lidMat);
  lidMesh.position.set(0, 0.2, 1.2); // 相对 pivot 偏移到前方
  lidMesh.castShadow = true;
  lidMesh.receiveShadow = true;
  lidMesh.userData.isLid = true;
  lidPivot.add(lidMesh);
  
  // 盒盖顶面装饰：大的 L 和 R 标识
  const topL = createLetterPlane('L', 1.0, true);
  topL.position.set(-1.05, 0.41, 0);
  topL.rotation.x = -Math.PI / 2;
  lidMesh.add(topL);
  
  const topR = createLetterPlane('R', 1.0, true);
  topR.position.set(1.05, 0.41, 0);
  topR.rotation.x = -Math.PI / 2;
  lidMesh.add(topR);
  
  // 盒盖顶部装饰线
  const lineGeo = new THREE.BoxGeometry(3.5, 0.01, 0.04);
  const lineMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.2,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.3,
  });
  
  const line1 = new THREE.Mesh(lineGeo, lineMat);
  line1.position.set(0, 0.21, -0.5);
  lidMesh.add(line1);
  
  const line2 = new THREE.Mesh(lineGeo, lineMat);
  line2.position.set(0, 0.21, 0.5);
  lidMesh.add(line2);
  
  // 铰链（两个小圆柱）
  const hingeGeo = new THREE.CylinderGeometry(0.06, 0.06, 0.3, 16);
  const hingeMat = new THREE.MeshStandardMaterial({
    color: 0x666666,
    metalness: 0.9,
    roughness: 0.2,
  });
  
  const hinge1 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge1.position.set(-1.5, 0.6, -1.2);
  hinge1.rotation.z = Math.PI / 2;
  caseGroup.add(hinge1);
  
  const hinge2 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge2.position.set(1.5, 0.6, -1.2);
  hinge2.rotation.z = Math.PI / 2;
  caseGroup.add(hinge2);
  
  // 调整整体位置
  caseGroup.position.y = 0;
  scene.add(caseGroup);
}

function createLetterPlane(letter, scale = 0.5, withGlow = false) {
  const texture = createLetterTexture(letter);
  
  const planeGeo = new THREE.PlaneGeometry(scale, scale);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
  });
  
  if (withGlow) {
    planeMat.blending = THREE.AdditiveBlending;
  }
  
  return new THREE.Mesh(planeGeo, planeMat);
}

function createLetterTexture(letter) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  // 透明背景
  ctx.clearRect(0, 0, size, size);
  
  // 多层发光
  ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
  ctx.shadowBlur = 60;
  
  // 渐变
  const gradient = ctx.createLinearGradient(0, 0, 0, size);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 380px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, size/2, size/2);
  
  // 再画一次让发光更明显
  ctx.shadowBlur = 30;
  ctx.fillText(letter, size/2, size/2);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.5)';
  ctx.lineWidth = 4;
  ctx.strokeText(letter, size/2, size/2);
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  return texture;
}

function createParticles() {
  const count = 200;
  const positions = new Float32Array(count * 3);
  const speeds = new Float32Array(count);
  
  for (let i = 0; i < count; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 15;
    positions[i * 3 + 1] = Math.random() * 8 - 1;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 15;
    speeds[i] = 0.001 + Math.random() * 0.003;
  }
  
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  
  // 粒子纹理（圆形）
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
  gradient.addColorStop(0, 'rgba(255, 200, 100, 1)');
  gradient.addColorStop(0.3, 'rgba(255, 180, 80, 0.5)');
  gradient.addColorStop(1, 'rgba(255, 180, 80, 0)');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, 64, 64);
  const particleTexture = new THREE.CanvasTexture(canvas);
  
  const mat = new THREE.PointsMaterial({
    size: 0.08,
    map: particleTexture,
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.6,
  });
  
  particleSystem = new THREE.Points(geo, mat);
  particleSystem.userData.speeds = speeds;
  scene.add(particleSystem);
}

function onPointerDown(event) {
  mouseDownPos = { x: event.clientX, y: event.clientY };
  mouseDownTime = Date.now();
}

function onPointerUp(event) {
  const dx = event.clientX - mouseDownPos.x;
  const dy = event.clientY - mouseDownPos.y;
  const distance = Math.sqrt(dx*dx + dy*dy);
  const duration = Date.now() - mouseDownTime;
  
  if (distance < 5 && duration < 400) {
    handleClick(event);
  }
}

function onPointerMove(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, false);
  
  if (intersects.length > 0) {
    if (!hoveredLid) {
      hoveredLid = true;
      document.body.style.cursor = 'pointer';
    }
  } else {
    if (hoveredLid) {
      hoveredLid = false;
      document.body.style.cursor = 'default';
    }
  }
}

function handleClick(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, false);
  
  if (intersects.length > 0) {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  }
}

function toggleLid() {
  isOpen = !isOpen;
  targetLidAngle = isOpen ? -Math.PI * 0.65 : 0;
  updateUI();
}

function updateUI() {
  document.getElementById('state-text').textContent = isOpen ? 'OPEN' : 'CLOSED';
  document.getElementById('toggle-btn').textContent = isOpen ? '关闭盒盖' : '开启盒盖';
}

function showToast(message) {
  const toast = document.getElementById('toast');
  toast.textContent = message;
  toast.classList.add('show');
  clearTimeout(showToast.timer);
  showToast.timer = setTimeout(() => {
    toast.classList.remove('show');
  }, 1800);
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

function animate() {
  requestAnimationFrame(animate);
  
  const delta = clock.getDelta();
  const time = clock.getElapsedTime();
  
  // 盖子平滑过渡
  if (Math.abs(lidAngle - targetLidAngle) > 0.001) {
    lidAngle += (targetLidAngle - lidAngle) * 0.08;
    lidPivot.rotation.x = lidAngle;
  } else if (lidAngle !== targetLidAngle) {
    lidAngle = targetLidAngle;
    lidPivot.rotation.x = lidAngle;
  }
  
  // 液体波动
  if (leftLiquid) {
    leftLiquid.position.y = 0.52 + Math.sin(time * 2) * 0.005;
  }
  if (rightLiquid) {
    rightLiquid.position.y = 0.52 + Math.cos(time * 2.2) * 0.005;
  }
  
  // 粒子飘动
  if (particleSystem) {
    const positions = particleSystem.geometry.attributes.position.array;
    const speeds = particleSystem.userData.speeds;
    for (let i = 0; i < positions.length / 3; i++) {
      positions[i * 3 + 1] += speeds[i];
      if (positions[i * 3 + 1] > 7) {
        positions[i * 3 + 1] = -1;
      }
      positions[i * 3] += Math.sin(time + i) * 0.001;
    }
    particleSystem.geometry.attributes.position.needsUpdate = true;
  }
  
  // 相机缓慢自转（如果用户没在操作）
  // 不做这个，避免干扰用户
  
  controls.update();
  renderer.render(scene, camera);
}

init();
```

让我检查一下：
1. ✅ Three.js with importmap
2. ✅ RoundedBoxGeometry 用于圆角盒子
3. ✅ L 和 R 标识（在盒身顶面 + 盒盖顶面）
4. ✅ 盒盖可点击打开/关闭
5. ✅ 铰链动画
6. ✅ OrbitControls 旋转视角
7. ✅ 灯光、阴影、粒子
8. ✅ 凹槽、液体、隐形眼镜

一些细节：
- 检查 lidMesh.userData.isLid 的设置，raycaster 用 lidMesh 检测
- lidMesh 包含子对象（字母平面），所以 raycast 时要 recursive = false 来只检测盒盖主体。但 lidMesh 的子对象也会被检测，没问题，因为子对象的父级链包含 lidMesh。

Wait, `raycaster.intersectObject(lidMesh, false)` 第二个参数是 recursive，false 时不检测子对象。这样点击字母平面时不会触发，只能点击盒盖本身。这是期望的行为吗？

实际上点击字母也应该触发开盖，因为字母在盒盖上。所以应该用 recursive = true。

```javascript
const intersects = raycaster.intersectObject(lidMesh, true);
```

这样点击盒盖或字母都会触发开盖。

但是字母平面是 transparent + depthWrite: false，可能影响 raycast。让我测试一下。其实 raycast 不受材质影响，只要 geometry 存在就能检测到。

好的，应该没问题。

让我再优化一下视觉：
- 字母应该足够大、足够醒目
- 盒盖关闭时，盒盖顶面的 L 和 R 是主要可见标识
- 盒盖打开后，盒身顶面的 L 和 R 可见

字母平面贴在盒盖顶面时，要注意 Z-fighting 问题。让字母平面稍微高于盒盖表面（y = 0.41，盒盖厚度 0.4，所以表面在 y = 0.4，字母在 0.41，离表面 0.01）。这样应该没问题，但可能不够明显。让字母浮在 0.42 高度。

实际上字母是 lidMesh 的子对象，lidMesh 中心在 (0,0,0)，所以 lidMesh 的顶面在 y = 0.2（lidGeo 厚度 0.4）。字母 plane 应该放在 y = 0.21。

让我重新检查 lidMesh 的几何尺寸：
- lidGeo: RoundedBoxGeometry(4, 0.4, 2.4, 8, 0.15)
- lidMesh 位置：(0, 0.2, 1.2) 相对 lidPivot
- lidPivot 位置：(0, 0.6, -1.2) 相对 caseGroup

lidMesh 中心：lidPivot 位置 + lidMesh 相对位置 = (0, 0.6+0.2, -1.2+1.2) = (0, 0.8, 0)
盒盖顶面在 y = 0.8 + 0.2 = 1.0
盒盖底面在 y = 0.8 - 0.2 = 0.6

盒身顶面在 y = 0 + 0.6 = 0.6

所以盒盖底面正好接触盒身顶面。✓

字母在 lidMesh 的局部坐标系中，lidMesh 顶面在 y = 0.2，所以字母放在 y = 0.21 即可。

让我重新写 lidMesh 的字母位置：

```javascript
const topL = createLetterPlane('L', 1.0, true);
topL.position.set(-1.05, 0.21, 0);
topL.rotation.x = -Math.PI / 2;
lidMesh.add(topL);
```

字母大小 1.0，在 4x2.4 的盒盖顶面，两个字母占据 2x1 的空间，合适。

L 在 x = -1.05，R 在 x = 1.05，间距 2.1，合理。

字母 z = 0，居中。

盒身的字母（盒盖打开后可见）：
位置应该在凹槽旁边。凹槽在 (-1.05, 0.36, 0) 和 (1.05, 0.36, 0)，半径 0.7。
字母可以放在凹槽前方（z > 0），距离凹槽中心约 1.0 的位置。

盒身顶面在 y = 0.6，字母放在 y = 0.61。

```javascript
const letterL = createLetterPlane('L', 0.5);
letterL.position.set(-1.05, 0.61, 0.9);
letterL.rotation.x = -Math.PI / 2;
caseGroup.add(letterL);
```

不过这样字母会和凹槽重合... 凹槽半径 0.7，所以字母中心要在 z = ±1.0 之外。

让我重新算：
- 凹槽在 x = ±1.05
- 凹槽半径 0.7
- 盒身宽度 z 方向是 ±1.2
- 凹槽占据 z = ±0.7

字母放在凹槽前方（z = 0.95）和后方（z = -0.95），距离凹槽边缘 0.25。

但这样字母会很小且离凹槽近。让我换个思路：

字母放在凹槽的同侧（前方），凹槽旁标识每个槽是 L 还是 R。

不过更好的方案：字母直接放在凹槽内部，作为凹槽的标识。例如字母贴在凹槽底部，从上方可见。或者字母贴在凹槽前侧壁上。

实际上真实的眼镜盒：字母就是印在凹槽旁边的盒身顶面。

让我用这个方案：
- 字母 L 贴在盒身顶面，位置 (-1.05, 0.61, 0)，但是要在凹槽旁边
- 不行，凹槽半径 0.7 占据了 (-1.05, 0) 周围

让我重新设计盒子尺寸：
- 盒身：4 x 1.2 x 2.4 (x,y,z)
- 凹槽：在 (-1.05, 0.36, 0) 和 (1.05, 0.36, 0)，半径 0.7
- 凹槽直径 1.4
- 两个凹槽中心相距 2.1
- 凹槽边缘到盒子边缘（z=1.2）：1.2 - 0.7 = 0.5

所以盒身顶面凹槽前方有 0.5 的空间放字母。字母直径 0.4 没问题。

字母位置：
- L 在 (-1.05, 0.61, 0.95)
- R 在 (1.05, 0.61, 0.95)
- 字母大小 0.4

这样字母贴在凹槽前方，醒目可见。盒盖打开后从上往下看可以清楚看到。

但是字母大小 0.4 可能不够醒目。让我调大盒子尺寸，给字母更多空间。

调整：
- 盒身：4.5 x 1.2 x 2.8
- 凹槽中心：(-1.2, 0.36, 0) 和 (1.2, 0.36, 0)，半径 0.7
- 字母位置：(-1.2, 0.61, 1.05) 和 (1.2, 0.61, 1.05)
- 字母大小 0.5

或者把字母放在凹槽之间的盒身顶面（中间区域），但中间区域狭窄。

我觉得最醒目的位置还是盒盖顶面。盒盖顶面大的 L 和 R 是主要标识。盒身内部凹槽旁的小字母是辅助标识。

让我调整方案：
- 盒身：4 x 1.2 x 2.4
- 凹槽：(-1.05, ?, 0) 和 (1.05, ?, 0)，半径 0.65
- 盒盖顶面 L 和 R：大，醒目
- 盒身凹槽前方 L 和 R：小，标识

为了节省空间，让凹槽半径小一点（0.6），盒子大一点（4.4 x 1.2 x 2.4）。

最终参数：
- 盒身：4.4 x 1.2 x 2.4
- 凹槽：(-1.15, 0.36, 0) 和 (1.15, 0.36, 0)，半径 0.65
- 凹槽之间间距 2.3，凹槽边缘间距 1.0
- 凹槽边缘到盒子短边（z = ±1.2）的距离 0.55

字母位置：
- 盒身顶面 L: (-1.15, 0.61, 0.95)，大小 0.4
- 盒身顶面 R: (1.15, 0.61, 0.95)，大小 0.4

盒盖尺寸：4.4 x 0.4 x 2.4
盒盖顶面字母：
- L: (-1.15, 0.21, 0)，大小 1.1
- R: (1.15, 0.21, 0)，大小 1.1

这样字母比较醒目。

让我再考虑盒盖旋转后的字母位置。盒盖打开后向后翻转 110°，字母会朝下。这样从相机视角可能看不到字母了。

所以字母主要在盒盖关闭时可见。打开后看到盒身内部的字母。这个设计合理。

现在让我把代码整合好。

还有一个细节：lensGeometry 用 TorusGeometry 不太合适，因为眼镜不是环形。用 CircleGeometry 或者 SphereGeometry 截一半更合适。

实际上隐形眼镜是个软的圆盘，从上面看是圆形。用 CircleGeometry 配合 transparent transmission 材质可以模拟。

```javascript
const lensGeo = new THREE.SphereGeometry(0.4, 32, 16, 0, Math.PI * 2, 0, Math.PI / 4);
```

这样是个球冠，看起来像隐形眼镜的弯曲形状。但需要在凹槽里水平放置。

或者直接用扁平的圆盘：

```javascript
const lensGeo = new THREE.CircleGeometry(0.35, 32);
```

加上 transmission 材质看起来透明。

简单点用 TorusGeometry 也行，就是个透明的小圆环。或者用 RingGeometry。

让我用扁平的圆盘 + 弯曲效果。用 SphereGeometry 取一小部分：

```javascript
const lensGeo = new THREE.SphereGeometry(0.5, 32, 16, 0, Math.PI * 2, Math.PI * 0.4, Math.PI * 0.2);
```

这是一个球冠，看起来像隐形眼镜的碗状。

材质：
```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.95,
  thickness: 0.1,
  ior: 1.4,
  side: THREE.DoubleSide,
});
```

让 lens 朝上凹（开口朝上）：
```javascript
leftLens.rotation.x = -Math.PI / 2;
```

或者保持原样，因为它本身就是球冠。

让我用更简单的方式：扁平圆盘 + 透明材质。

```javascript
const lensGeo = new THREE.CircleGeometry(0.35, 64);
const lensMat = new THREE.MeshPhysicalMaterial({
  color: 0xc4eeff,
  metalness: 0,
  roughness: 0.1,
  transmission: 0.8,
  thickness: 0.05,
  ior: 1.4,
  side: THREE.DoubleSide,
});
const leftLens = new THREE.Mesh(lensGeo, lensMat);
leftLens.rotation.x = -Math.PI / 2;
leftLens.position.set(-1.15, 0.5, 0);
caseGroup.add(leftLens);
```

OK，这样应该够了。

让我再优化下盒盖的视觉。盒盖顶面除了字母，还可以加：
- 装饰线条
- 边缘倒角
- 一个小指示灯（emissive）

让盒盖看起来更高级。

```javascript
// 盒盖边缘装饰金线
const edgeLineGeo = new THREE.BoxGeometry(4.4, 0.04, 0.04);
const edgeLineMat = new THREE.MeshStandardMaterial({
  color: COLORS.letterGold,
  metalness: 0.9,
  roughness: 0.15,
  emissive: COLORS.letterGold,
  emissiveIntensity: 0.4,
});

// 四条边
// 前
const frontLine = new THREE.Mesh(edgeLineGeo, edgeLineMat);
frontLine.position.set(0, 0.21, 1.18);
lidMesh.add(frontLine);

// 后
const backLine = new THREE.Mesh(edgeLineGeo, edgeLineMat);
backLine.position.set(0, 0.21, -1.18);
lidMesh.add(backLine);

// 左右（旋转）
const sideLineGeo = new THREE.BoxGeometry(0.04, 0.04, 2.4);
const leftLine = new THREE.Mesh(sideLineGeo, edgeLineMat);
leftLine.position.set(-2.18, 0.21, 0);
lidMesh.add(leftLine);

const rightLine = new THREE.Mesh(sideLineGeo, edgeLineMat);
rightLine.position.set(2.18, 0.21, 0);
lidMesh.add(rightLine);
```

等等，这些坐标都是 lidMesh 的局部坐标，但 lidMesh 是 RoundedBoxGeometry(4.4, 0.4, 2.4)，所以 lidMesh 的尺寸是 4.4 x 0.4 x 2.4，边缘在 ±2.2, ±0.2, ±1.2。

边线放在 y = 0.21（盒盖顶面之上），z = ±1.18（盒盖边缘之内）。

好的。但实际上 RoundedBoxGeometry 的边缘有圆角，所以放在 z = ±1.18 可能够不到边缘。让我直接放在 z = ±1.15。

代码量不少了。让我整理一下，确保没有错误。

我还要考虑：
1. 灯光配置：DirectionalLight 的 shadow.camera 范围要覆盖盒子
2. 相机位置：能完整看到盒子，包括打开后的盒盖

相机位置 (5, 3.5, 6.5)，距离原点约 9.4。盒子大小 4.4 x 1.6 x 2.4（含盖子高度），合适。

让我加一个细节：当盒盖打开时，可以添加内部灯光亮起的效果。这样打开后内部更醒目。

```javascript
// 内部灯光（盒盖打开时亮起）
const innerLight = new THREE.PointLight(0x4ecdc4, 0, 3);
innerLight.position.set(0, 0.4, 0);
caseGroup.add(innerLight);

// 在 animate 中根据 isOpen 调整亮度
if (innerLight) {
  const targetIntensity = isOpen ? 1.5 : 0;
  innerLight.intensity += (targetIntensity - innerLight.intensity) * 0.1;
}
```

这样打开盒盖时内部亮起，效果更好。

最后再检查一下整体结构，确保代码能运行。

让我整理完整的代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LUMINA · 隐形眼镜盒 3D 模型</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;500;700&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500&display=swap" rel="stylesheet">

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  :root {
    --bg-dark: #050a0c;
    --bg-mid: #0a1f1c;
    --bg-light: #143633;
    --accent: #f5a623;
    --accent-bright: #ffd966;
    --accent-2: #4ecdc4;
    --text: #e8f4f0;
    --text-dim: #6a8c87;
    --text-mute: #4a6c67;
    --border: rgba(245, 166, 35, 0.3);
  }
  
  body {
    background: radial-gradient(ellipse at 50% 30%, var(--bg-light) 0%, var(--bg-mid) 40%, var(--bg-dark) 80%);
    color: var(--text);
    font-family: 'Space Grotesk', system-ui, sans-serif;
    overflow: hidden;
    height: 100vh;
    position: relative;
  }
  
  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background: radial-gradient(circle at 30% 70%, rgba(78, 205, 196, 0.08) 0%, transparent 50%),
                radial-gradient(circle at 70% 30%, rgba(245, 166, 35, 0.06) 0%, transparent 50%);
    pointer-events: none;
    z-index: 1;
  }
  
  #canvas-container {
    position: fixed;
    inset: 0;
    z-index: 2;
  }
  
  /* === 顶部栏 === */
  .header {
    position: fixed;
    top: 0; left: 0; right: 0;
    padding: 32px 48px;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    pointer-events: none;
  }
  
  .brand {
    font-family: 'Cormorant Garamond', serif;
    font-weight: 500;
    line-height: 1;
  }
  
  .brand .name {
    font-size: 28px;
    letter-spacing: 8px;
    color: var(--accent);
    text-transform: uppercase;
    display: block;
  }
  
  .brand .tag {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    color: var(--text-mute);
    letter-spacing: 5px;
    margin-top: 8px;
    display: block;
  }
  
  .status {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    color: var(--text-dim);
    text-align: right;
    line-height: 1.8;
  }
  
  .status .row {
    display: flex;
    gap: 12px;
    justify-content: flex-end;
    align-items: center;
  }
  
  .status .label {
    color: var(--text-mute);
    font-size: 9px;
    letter-spacing: 2px;
  }
  
  .status .value {
    color: var(--accent-2);
    font-weight: 500;
    letter-spacing: 1px;
  }
  
  .status .value.state {
    color: var(--accent);
  }
  
  .status .indicator {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--accent);
    box-shadow: 0 0 10px var(--accent);
    animation: pulse 1.8s ease-in-out infinite;
  }
  
  @keyframes pulse {
    0%, 100% { opacity: 0.6; transform: scale(1); }
    50% { opacity: 1; transform: scale(1.3); }
  }
  
  /* === 底部栏 === */
  .footer {
    position: fixed;
    bottom: 0; left: 0; right: 0;
    padding: 32px 48px;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    pointer-events: none;
  }
  
  .controls-hint {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    color: var(--text-dim);
    line-height: 2;
  }
  
  .controls-hint .row {
    display: flex;
    align-items: center;
    gap: 14px;
  }
  
  .controls-hint .key {
    display: inline-block;
    min-width: 60px;
    padding: 3px 8px;
    border: 1px solid var(--border);
    color: var(--accent);
    font-size: 9px;
    letter-spacing: 2px;
    text-align: center;
    background: rgba(245, 166, 35, 0.05);
  }
  
  .action-btn {
    pointer-events: auto;
    background: linear-gradient(135deg, rgba(245, 166, 35, 0.15), rgba(245, 166, 35, 0.05));
    border: 1px solid var(--border);
    color: var(--accent);
    padding: 16px 36px;
    font-family: 'Space Grotesk', sans-serif;
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 4px;
    text-transform: uppercase;
    cursor: pointer;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    backdrop-filter: blur(12px);
    position: relative;
    overflow: hidden;
  }
  
  .action-btn::before {
    content: '';
    position: absolute;
    top: 0; left: -100%;
    width: 100%; height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 217, 102, 0.3), transparent);
    transition: left 0.6s;
  }
  
  .action-btn:hover::before {
    left: 100%;
  }
  
  .action-btn:hover {
    border-color: var(--accent);
    color: var(--accent-bright);
    box-shadow: 0 0 40px rgba(245, 166, 35, 0.25), inset 0 0 20px rgba(245, 166, 35, 0.05);
    transform: translateY(-2px);
  }
  
  .action-btn:active {
    transform: translateY(0);
  }
  
  /* === 装饰角标 === */
  .corner-mark {
    position: fixed;
    width: 28px;
    height: 28px;
    border: 1px solid var(--border);
    z-index: 5;
    pointer-events: none;
  }
  
  .corner-mark.tl { top: 20px; left: 20px; border-right: none; border-bottom: none; }
  .corner-mark.tr { top: 20px; right: 20px; border-left: none; border-bottom: none; }
  .corner-mark.bl { bottom: 20px; left: 20px; border-right: none; border-top: none; }
  .corner-mark.br { bottom: 20px; right: 20px; border-left: none; border-top: none; }
  
  /* === 侧边信息 === */
  .side-info {
    position: fixed;
    left: 32px;
    top: 50%;
    transform: translateY(-50%) rotate(-90deg);
    transform-origin: left center;
    font-family: 'JetBrains Mono', monospace;
    font-size: 9px;
    letter-spacing: 8px;
    color: var(--text-mute);
    z-index: 5;
    pointer-events: none;
    white-space: nowrap;
  }
  
  .side-info-r {
    position: fixed;
    right: 32px;
    top: 50%;
    transform: translateY(-50%) rotate(90deg);
    transform-origin: right center;
    font-family: 'JetBrains Mono', monospace;
    font-size: 9px;
    letter-spacing: 8px;
    color: var(--text-mute);
    z-index: 5;
    pointer-events: none;
    white-space: nowrap;
  }
  
  /* === Toast 提示 === */
  .toast {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, calc(-50% + 200px));
    background: rgba(10, 31, 28, 0.92);
    border: 1px solid var(--accent);
    color: var(--text);
    padding: 14px 28px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    letter-spacing: 3px;
    text-transform: uppercase;
    z-index: 100;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.4s, transform 0.4s;
    backdrop-filter: blur(20px);
    box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 30px rgba(245, 166, 35, 0.2);
  }
  
  .toast.show {
    opacity: 1;
    transform: translate(-50%, calc(-50% + 220px));
  }
  
  /* === 加载提示 === */
  .loader {
    position: fixed;
    inset: 0;
    background: var(--bg-dark);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: opacity 0.8s;
  }
  
  .loader.hidden {
    opacity: 0;
    pointer-events: none;
  }
  
  .loader-text {
    font-family: 'Cormorant Garamond', serif;
    color: var(--accent);
    font-size: 14px;
    letter-spacing: 6px;
    text-transform: uppercase;
  }
  
  /* === 响应式 === */
  @media (max-width: 768px) {
    .header, .footer { padding: 20px; }
    .brand .name { font-size: 22px; letter-spacing: 5px; }
    .controls-hint { font-size: 10px; }
    .action-btn { padding: 12px 24px; font-size: 11px; letter-spacing: 2px; }
    .side-info, .side-info-r { display: none; }
    .corner-mark { width: 20px; height: 20px; }
  }
</style>
</head>
<body>
  <div class="loader" id="loader">
    <div class="loader-text">LUMINA · LOADING</div>
  </div>
  
  <div id="canvas-container"></div>
  
  <div class="corner-mark tl"></div>
  <div class="corner-mark tr"></div>
  <div class="corner-mark bl"></div>
  <div class="corner-mark br"></div>
  
  <div class="side-info">LUMINA · OPTICAL · CASE · SYSTEM</div>
  <div class="side-info-r">PRECISION · CRAFTED · 2024</div>
  
  <header class="header">
    <div class="brand">
      <span class="name">Lumina</span>
      <span class="tag">OPTICAL · LENS CASE</span>
    </div>
    <div class="status">
      <div class="row">
        <span class="label">MODEL</span>
        <span class="value">L-7 · GEMINI</span>
      </div>
      <div class="row">
        <span class="label">MATERIAL</span>
        <span class="value">POLYCARBONATE</span>
      </div>
      <div class="row">
        <span class="label">STATE</span>
        <span class="value state" id="state-text">CLOSED</span>
        <span class="indicator" id="state-indicator"></span>
      </div>
    </div>
  </header>
  
  <footer class="footer">
    <div class="controls-hint">
      <div class="row">
        <span class="key">DRAG</span>
        <span>旋转视角</span>
      </div>
      <div class="row">
        <span class="key">SCROLL</span>
        <span>缩放距离</span>
      </div>
      <div class="row">
        <span class="key">CLICK</span>
        <span>点击盒盖开合</span>
      </div>
    </div>
    <button class="action-btn" id="toggle-btn">开启盒盖</button>
  </footer>
  
  <div class="toast" id="toast"></div>
  
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>
  
  <script type="module">
    // ... 完整 JS 代码
  </script>
</body>
</html>
```

JS 代码：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

// === 全局变量 ===
let scene, camera, renderer, controls;
let caseGroup, lidPivot, lidMesh;
let leftLiquid, rightLiquid, innerLight;
let particleSystem;

let isOpen = false;
let lidAngle = 0;
let targetLidAngle = 0;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let mouseDownPos = { x: 0, y: 0 };
let mouseDownTime = 0;
let hoveredLid = false;

const clock = new THREE.Clock();

// === 颜色 ===
const COLORS = {
  caseBody: 0x0d3a36,
  caseBodyAccent: 0x1a5651,
  lidColor: 0x164440,
  lidAccent: 0x2a7a73,
  letterGold: 0xf5a623,
  letterGoldBright: 0xffd966,
  liquid: 0x4ecdc4,
  liquidDeep: 0x2a8a86,
  innerWell: 0x07201d,
  lensTint: 0xa0e8ff,
};

// === 初始化 ===
function init() {
  // 场景
  scene = new THREE.Scene();
  
  // 相机
  camera = new THREE.PerspectiveCamera(38, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(5, 3.2, 6.5);
  
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  // 环境贴图
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
  
  // 灯光
  setupLights();
  
  // 地板
  createFloor();
  
  // 盒子
  createCase();
  
  // 粒子
  createParticles();
  
  // 控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.06;
  controls.minDistance = 4;
  controls.maxDistance = 12;
  controls.maxPolarAngle = Math.PI * 0.5;
  controls.minPolarAngle = Math.PI * 0.15;
  controls.target.set(0, 0.2, 0);
  controls.enablePan = false;
  
  // 事件
  window.addEventListener('resize', onResize);
  renderer.domElement.addEventListener('pointerdown', onPointerDown);
  renderer.domElement.addEventListener('pointerup', onPointerUp);
  renderer.domElement.addEventListener('pointermove', onPointerMove);
  document.getElementById('toggle-btn').addEventListener('click', () => {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  });
  
  // 隐藏 loader
  setTimeout(() => {
    document.getElementById('loader').classList.add('hidden');
  }, 500);
  
  animate();
}

function setupLights() {
  // 环境光
  const ambient = new THREE.AmbientLight(0xffffff, 0.2);
  scene.add(ambient);
  
  // 主光（暖白）
  const keyLight = new THREE.DirectionalLight(0xfff5e0, 1.6);
  keyLight.position.set(4, 8, 5);
  keyLight.castShadow = true;
  keyLight.shadow.mapSize.set(2048, 2048);
  keyLight.shadow.camera.left = -4;
  keyLight.shadow.camera.right = 4;
  keyLight.shadow.camera.top = 4;
  keyLight.shadow.camera.bottom = -4;
  keyLight.shadow.camera.near = 0.5;
  keyLight.shadow.camera.far = 20;
  keyLight.shadow.bias = -0.0005;
  keyLight.shadow.radius = 4;
  scene.add(keyLight);
  
  // 补光（青色，从左侧）
  const fillLight = new THREE.PointLight(0x4ecdc4, 1.2, 15);
  fillLight.position.set(-4, 3, 2);
  scene.add(fillLight);
  
  // 边缘光（金色，从后方）
  const rimLight = new THREE.PointLight(0xff8c42, 0.9, 12);
  rimLight.position.set(3, 2, -4);
  scene.add(rimLight);
  
  // 顶部聚光
  const spotLight = new THREE.SpotLight(0xffffff, 1.2, 15, Math.PI * 0.18, 0.5, 1.5);
  spotLight.position.set(0, 8, 0);
  spotLight.target.position.set(0, 0, 0);
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.set(1024, 1024);
  spotLight.shadow.bias = -0.0005;
  scene.add(spotLight);
  scene.add(spotLight.target);
  
  // 内部灯光（盒盖打开时亮起）
  innerLight = new THREE.PointLight(0x4ecdc4, 0, 3);
  innerLight.position.set(0, 0.4, 0);
  scene.add(innerLight);
}

function createFloor() {
  // 阴影地板
  const shadowFloorGeo = new THREE.PlaneGeometry(40, 40);
  const shadowFloorMat = new THREE.ShadowMaterial({ opacity: 0.45 });
  const shadowFloor = new THREE.Mesh(shadowFloorGeo, shadowFloorMat);
  shadowFloor.rotation.x = -Math.PI / 2;
  shadowFloor.position.y = -0.65;
  shadowFloor.receiveShadow = true;
  scene.add(shadowFloor);
  
  // 装饰圆形发光地板
  const glowFloorGeo = new THREE.CircleGeometry(3.5, 64);
  const glowFloorMat = new THREE.MeshBasicMaterial({
    color: 0x4ecdc4,
    transparent: true,
    opacity: 0.06,
    side: THREE.DoubleSide,
  });
  const glowFloor = new THREE.Mesh(glowFloorGeo, glowFloorMat);
  glowFloor.rotation.x = -Math.PI / 2;
  glowFloor.position.y = -0.64;
  scene.add(glowFloor);
  
  // 内圈
  const innerRingGeo = new THREE.RingGeometry(2.8, 2.85, 64);
  const innerRingMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.4,
    side: THREE.DoubleSide,
  });
  const innerRing = new THREE.Mesh(innerRingGeo, innerRingMat);
  innerRing.rotation.x = -Math.PI / 2;
  innerRing.position.y = -0.63;
  scene.add(innerRing);
  
  // 外圈
  const outerRingGeo = new THREE.RingGeometry(3.4, 3.45, 64);
  const outerRingMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.2,
    side: THREE.DoubleSide,
  });
  const outerRing = new THREE.Mesh(outerRingGeo, outerRingMat);
  outerRing.rotation.x = -Math.PI / 2;
  outerRing.position.y = -0.63;
  scene.add(outerRing);
  
  // 刻度线（8 个方向）
  const tickMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.5,
    side: THREE.DoubleSide,
  });
  
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2;
    const tickGeo = new THREE.PlaneGeometry(0.15, 0.02);
    const tick = new THREE.Mesh(tickGeo, tickMat);
    tick.position.x = Math.cos(angle) * 3.1;
    tick.position.z = Math.sin(angle) * 3.1;
    tick.position.y = -0.63;
    tick.rotation.x = -Math.PI / 2;
    tick.rotation.z = -angle;
    scene.add(tick);
  }
}

function createCase() {
  caseGroup = new THREE.Group();
  
  // === 盒身 ===
  const bodyWidth = 4.4;
  const bodyHeight = 1.2;
  const bodyDepth = 2.4;
  const bodyRadius = 0.18;
  
  const bodyGeo = new RoundedBoxGeometry(bodyWidth, bodyHeight, bodyDepth, 8, bodyRadius);
  const bodyMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.caseBody,
    metalness: 0.5,
    roughness: 0.3,
    clearcoat: 0.7,
    clearcoatRoughness: 0.2,
  });
  const body = new THREE.Mesh(bodyGeo, bodyMat);
  body.position.y = 0;
  body.castShadow = true;
  body.receiveShadow = true;
  caseGroup.add(body);
  
  // 盒身侧面装饰金线
  const sideLineGeo = new THREE.BoxGeometry(bodyWidth - 0.4, 0.02, 0.02);
  const sideLineMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.3,
  });
  
  const sideLine1 = new THREE.Mesh(sideLineGeo, sideLineMat);
  sideLine1.position.set(0, -0.2, bodyDepth/2 - 0.01);
  caseGroup.add(sideLine1);
  
  const sideLine2 = new THREE.Mesh(sideLineGeo, sideLineMat);
  sideLine2.position.set(0, -0.2, -bodyDepth/2 + 0.01);
  caseGroup.add(sideLine2);
  
  // === 凹槽（左右两个圆形凹槽） ===
  const wellRadius = 0.7;
  const wellDepth = 0.5;
  const wellX = 1.15;
  
  // 凹槽侧壁（圆环）
  const wellGeo = new THREE.CylinderGeometry(wellRadius, wellRadius - 0.1, wellDepth, 64, 1, true);
  const wellMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.innerWell,
    metalness: 0.4,
    roughness: 0.25,
    side: THREE.DoubleSide,
  });
  
  const leftWellOuter = new THREE.Mesh(wellGeo, wellMat);
  leftWellOuter.position.set(-wellX, 0.36, 0);
  leftWellOuter.castShadow = true;
  leftWellOuter.receiveShadow = true;
  caseGroup.add(leftWellOuter);
  
  const rightWellOuter = new THREE.Mesh(wellGeo, wellMat);
  rightWellOuter.position.set(wellX, 0.36, 0);
  rightWellOuter.castShadow = true;
  rightWellOuter.receiveShadow = true;
  caseGroup.add(rightWellOuter);
  
  // 凹槽底部
  const wellBottomGeo = new THREE.CircleGeometry(wellRadius - 0.1, 64);
  const wellBottomMat = new THREE.MeshStandardMaterial({
    color: 0x051815,
    metalness: 0.3,
    roughness: 0.5,
  });
  
  const leftWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  leftWellBottom.rotation.x = -Math.PI / 2;
  leftWellBottom.position.set(-wellX, 0.12, 0);
  caseGroup.add(leftWellBottom);
  
  const rightWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  rightWellBottom.rotation.x = -Math.PI / 2;
  rightWellBottom.position.set(wellX, 0.12, 0);
  caseGroup.add(rightWellBottom);
  
  // 凹槽边缘金色装饰圈
  const ringGeo = new THREE.TorusGeometry(wellRadius + 0.02, 0.025, 16, 64);
  const ringMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.2,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.2,
  });
  
  const leftRing = new THREE.Mesh(ringGeo, ringMat);
  leftRing.position.set(-wellX, 0.6, 0);
  leftRing.rotation.x = -Math.PI / 2;
  caseGroup.add(leftRing);
  
  const rightRing = new THREE.Mesh(ringGeo, ringMat);
  rightRing.position.set(wellX, 0.6, 0);
  rightRing.rotation.x = -Math.PI / 2;
  caseGroup.add(rightRing);
  
  // === 液体（护理液表面） ===
  const liquidGeo = new THREE.CircleGeometry(wellRadius - 0.13, 64);
  const liquidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.liquid,
    metalness: 0.0,
    roughness: 0.05,
    transmission: 0.4,
    thickness: 0.4,
    ior: 1.33,
    side: THREE.DoubleSide,
    emissive: COLORS.liquid,
    emissiveIntensity: 0.1,
  });
  
  leftLiquid = new THREE.Mesh(liquidGeo, liquidMat);
  leftLiquid.rotation.x = -Math.PI / 2;
  leftLiquid.position.set(-wellX, 0.5, 0);
  caseGroup.add(leftLiquid);
  
  rightLiquid = new THREE.Mesh(liquidGeo, liquidMat.clone());
  rightLiquid.rotation.x = -Math.PI / 2;
  rightLiquid.position.set(wellX, 0.5, 0);
  caseGroup.add(rightLiquid);
  
  // === 隐形眼镜（透明圆盘） ===
  const lensGeo = new THREE.SphereGeometry(0.45, 32, 16, 0, Math.PI * 2, 0, Math.PI * 0.18);
  const lensMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lensTint,
    metalness: 0,
    roughness: 0.05,
    transmission: 0.85,
    thickness: 0.1,
    ior: 1.4,
    side: THREE.DoubleSide,
    clearcoat: 0.8,
    clearcoatRoughness: 0.1,
  });
  
  const leftLens = new THREE.Mesh(lensGeo, lensMat);
  leftLens.position.set(-wellX, 0.46, 0);
  caseGroup.add(leftLens);
  
  const rightLens = new THREE.Mesh(lensGeo, lensMat.clone());
  rightLens.position.set(wellX, 0.46, 0);
  caseGroup.add(rightLens);
  
  // === L 和 R 标识（盒身顶面，凹槽前方） ===
  const letterLBody = createLetterPlane('L', 0.45);
  letterLBody.position.set(-wellX, 0.601, 0.95);
  letterLBody.rotation.x = -Math.PI / 2;
  caseGroup.add(letterLBody);
  
  const letterRBody = createLetterPlane('R', 0.45);
  letterRBody.position.set(wellX, 0.601, 0.95);
  letterRBody.rotation.x = -Math.PI / 2;
  caseGroup.add(letterRBody);
  
  // 凹槽后方也放字母（小一些）
  const letterLBody2 = createLetterPlane('L', 0.4);
  letterLBody2.position.set(-wellX, 0.601, -0.95);
  letterLBody2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterLBody2);
  
  const letterRBody2 = createLetterPlane('R', 0.4);
  letterRBody2.position.set(wellX, 0.601, -0.95);
  letterRBody2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterRBody2);
  
  // === 盒盖 ===
  const lidHeight = 0.4;
  lidPivot = new THREE.Group();
  lidPivot.position.set(0, 0.6, -bodyDepth/2); // 铰链在盒子后方上边缘
  caseGroup.add(lidPivot);
  
  const lidGeo = new RoundedBoxGeometry(bodyWidth, lidHeight, bodyDepth, 8, 0.15);
  const lidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lidColor,
    metalness: 0.7,
    roughness: 0.2,
    clearcoat: 0.9,
    clearcoatRoughness: 0.08,
  });
  lidMesh = new THREE.Mesh(lidGeo, lidMat);
  lidMesh.position.set(0, lidHeight/2, bodyDepth/2); // 相对 pivot 偏移到前方
  lidMesh.castShadow = true;
  lidMesh.receiveShadow = true;
  lidMesh.userData.isLid = true;
  lidPivot.add(lidMesh);
  
  // 盒盖顶面装饰：大的 L 和 R 标识
  const topL = createLetterPlane('L', 1.1, true);
  topL.position.set(-wellX, lidHeight/2 + 0.005, 0);
  topL.rotation.x = -Math.PI / 2;
  lidMesh.add(topL);
  
  const topR = createLetterPlane('R', 1.1, true);
  topR.position.set(wellX, lidHeight/2 + 0.005, 0);
  topR.rotation.x = -Math.PI / 2;
  lidMesh.add(topR);
  
  // 盒盖顶面装饰金线
  const lidLineGeo = new THREE.BoxGeometry(bodyWidth - 0.6, 0.015, 0.03);
  const lidLineMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.4,
  });
  
  const lidLine1 = new THREE.Mesh(lidLineGeo, lidLineMat);
  lidLine1.position.set(0, lidHeight/2 + 0.005, bodyDepth/2 - 0.25);
  lidMesh.add(lidLine1);
  
  const lidLine2 = new THREE.Mesh(lidLineGeo, lidLineMat);
  lidLine2.position.set(0, lidHeight/2 + 0.005, -bodyDepth/2 + 0.25);
  lidMesh.add(lidLine2);
  
  // 盒盖顶面中央装饰小圆
  const centerDotGeo = new THREE.CircleGeometry(0.08, 32);
  const centerDotMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGoldBright,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGoldBright,
    emissiveIntensity: 0.6,
  });
  const centerDot = new THREE.Mesh(centerDotGeo, centerDotMat);
  centerDot.position.set(0, lidHeight/2 + 0.01, 0);
  centerDot.rotation.x = -Math.PI / 2;
  lidMesh.add(centerDot);
  
  // 铰链（两个小圆柱）
  const hingeGeo = new THREE.CylinderGeometry(0.07, 0.07, 0.35, 16);
  const hingeMat = new THREE.MeshStandardMaterial({
    color: 0x555555,
    metalness: 0.95,
    roughness: 0.15,
  });
  
  const hinge1 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge1.position.set(-1.6, 0.6, -bodyDepth/2);
  hinge1.rotation.z = Math.PI / 2;
  hinge1.castShadow = true;
  caseGroup.add(hinge1);
  
  const hinge2 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge2.position.set(1.6, 0.6, -bodyDepth/2);
  hinge2.rotation.z = Math.PI / 2;
  hinge2.castShadow = true;
  caseGroup.add(hinge2);
  
  // 卡扣（盒子前方中间的小凸起）
  const latchGeo = new RoundedBoxGeometry(0.4, 0.15, 0.1, 4, 0.04);
  const latchMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.2,
    clearcoat: 0.6,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.15,
  });
  
  // 盒身上的卡扣槽
  const latchSlot = new THREE.Mesh(latchGeo, latchMat);
  latchSlot.position.set(0, 0.55, bodyDepth/2 - 0.02);
  caseGroup.add(latchSlot);
  
  // 调整整体位置
  caseGroup.position.y = 0;
  scene.add(caseGroup);
}

function createLetterPlane(letter, scale = 0.5, withGlow = false) {
  const texture = createLetterTexture(letter, withGlow);
  
  const planeGeo = new THREE.PlaneGeometry(scale, scale);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
  });
  
  if (withGlow) {
    planeMat.blending = THREE.AdditiveBlending;
  }
  
  const mesh = new THREE.Mesh(planeGeo, planeMat);
  return mesh;
}

function createLetterTexture(letter, withGlow = false) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  if (withGlow) {
    // 多层发光
    ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
    ctx.shadowBlur = 80;
  } else {
    ctx.shadowColor = 'rgba(255, 180, 60, 0.6)';
    ctx.shadowBlur = 30;
  }
  
  // 渐变填充
  const gradient = ctx.createLinearGradient(0, 0, 0, size);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 再画一次加强发光
  ctx.shadowBlur = 25;
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.6)';
  ctx.lineWidth = 5;
  ctx.strokeText(letter, size/2, size/2 + 10);
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  texture.needsUpdate = true;
  return texture;
}

function createParticles() {
  const count = 180;
  const positions = new Float32Array(count * 3);
  const speeds = new Float32Array(count);
  const sizes = new Float32Array(count);
  
  for (let i = 0; i < count; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 14;
    positions[i * 3 + 1] = Math.random() * 7 - 1;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 14;
    speeds[i] = 0.002 + Math.random() * 0.004;
    sizes[i] = 0.04 + Math.random() * 0.08;
  }
  
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geo.setAttribute('size', new THREE.BufferAttribute(sizes, 1));
  
  // 粒子纹理
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
  gradient.addColorStop(0, 'rgba(255, 200, 100, 1)');
  gradient.addColorStop(0.3, 'rgba(255, 180, 80, 0.6)');
  gradient.addColorStop(1, 'rgba(255, 180, 80, 0)');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, 64, 64);
  const particleTexture = new THREE.CanvasTexture(canvas);
  
  const mat = new THREE.PointsMaterial({
    size: 0.08,
    map: particleTexture,
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.5,
    sizeAttenuation: true,
  });
  
  particleSystem = new THREE.Points(geo, mat);
  particleSystem.userData.speeds = speeds;
  scene.add(particleSystem);
}

function onPointerDown(event) {
  mouseDownPos = { x: event.clientX, y: event.clientY };
  mouseDownTime = Date.now();
}

function onPointerUp(event) {
  const dx = event.clientX - mouseDownPos.x;
  const dy = event.clientY - mouseDownPos.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  const duration = Date.now() - mouseDownTime;
  
  if (distance < 5 && duration < 400) {
    handleClick(event);
  }
}

function onPointerMove(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, true);
  
  if (intersects.length > 0) {
    if (!hoveredLid) {
      hoveredLid = true;
      document.body.style.cursor = 'pointer';
    }
  } else {
    if (hoveredLid) {
      hoveredLid = false;
      document.body.style.cursor = 'default';
    }
  }
}

function handleClick(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, true);
  
  if (intersects.length > 0) {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  }
}

function toggleLid() {
  isOpen = !isOpen;
  targetLidAngle = isOpen ? -Math.PI * 0.65 : 0;
  updateUI();
}

function updateUI() {
  document.getElementById('state-text').textContent = isOpen ? 'OPEN' : 'CLOSED';
  document.getElementById('toggle-btn').textContent = isOpen ? '关闭盒盖' : '开启盒盖';
  const indicator = document.getElementById('state-indicator');
  if (isOpen) {
    indicator.style.background = 'var(--accent-2)';
    indicator.style.boxShadow = '0 0 10px var(--accent-2)';
  } else {
    indicator.style.background = 'var(--accent)';
    indicator.style.boxShadow = '0 0 10px var(--accent)';
  }
}

function showToast(message) {
  const toast = document.getElementById('toast');
  toast.textContent = message;
  toast.classList.add('show');
  clearTimeout(showToast.timer);
  showToast.timer = setTimeout(() => {
    toast.classList.remove('show');
  }, 1800);
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

function animate() {
  requestAnimationFrame(animate);
  
  const time = clock.getElapsedTime();
  
  // 盖子平滑过渡
  if (Math.abs(lidAngle - targetLidAngle) > 0.001) {
    lidAngle += (targetLidAngle - lidAngle) * 0.08;
    lidPivot.rotation.x = lidAngle;
  } else if (lidAngle !== targetLidAngle) {
    lidAngle = targetLidAngle;
    lidPivot.rotation.x = lidAngle;
  }
  
  // 内部灯光（盒盖打开时亮起）
  if (innerLight) {
    const targetIntensity = isOpen ? 1.8 : 0;
    innerLight.intensity += (targetIntensity - innerLight.intensity) * 0.1;
  }
  
  // 液体波动
  if (leftLiquid) {
    leftLiquid.position.y = 0.5 + Math.sin(time * 2) * 0.003;
    leftLiquid.material.opacity = 0.85 + Math.sin(time * 1.5) * 0.05;
  }
  if (rightLiquid) {
    rightLiquid.position.y = 0.5 + Math.cos(time * 2.2) * 0.003;
    rightLiquid.material.opacity = 0.85 + Math.cos(time * 1.5) * 0.05;
  }
  
  // 粒子飘动
  if (particleSystem) {
    const positions = particleSystem.geometry.attributes.position.array;
    const speeds = particleSystem.userData.speeds;
    const count = positions.length / 3;
    
    for (let i = 0; i < count; i++) {
      positions[i * 3 + 1] += speeds[i];
      if (positions[i * 3 + 1] > 6.5) {
        positions[i * 3 + 1] = -0.5;
        positions[i * 3] = (Math.random() - 0.5) * 14;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 14;
      }
      positions[i * 3] += Math.sin(time + i * 0.5) * 0.0008;
      positions[i * 3 + 2] += Math.cos(time + i * 0.3) * 0.0008;
    }
    particleSystem.geometry.attributes.position.needsUpdate = true;
  }
  
  // 盒子整体微微浮动
  if (caseGroup) {
    caseGroup.position.y = Math.sin(time * 0.8) * 0.04;
    caseGroup.rotation.y = Math.sin(time * 0.3) * 0.02;
  }
  
  controls.update();
  renderer.render(scene, camera);
}

init();
```

让我检查一些细节：

1. `lidMesh.material.opacity` 我修改了 liquid 的 opacity，但 liquid material 没有 transparent: true。需要在 liquid material 中加 transparent: true。

让我修改：

```javascript
const liquidMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.liquid,
  metalness: 0.0,
  roughness: 0.05,
  transmission: 0.4,
  thickness: 0.4,
  ior: 1.33,
  side: THREE.DoubleSide,
  emissive: COLORS.liquid,
  emissiveIntensity: 0.1,
  transparent: true,
  opacity: 0.9,
});
```

2. `lens.material.opacity` 同样需要 transparent。

3. lidMesh 上的字母 plane 用了 AdditiveBlending，可能在关闭状态下太亮。让我看下效果，可能要调小 opacity。

实际上 withGlow 模式下字母发光会很明显，适合盒盖顶面的标识。盒身顶面的字母用普通模式。

4. 注意 createLetterPlane 的 withGlow 参数只在 lidMesh 顶面用。

5. lidMesh raycast：用 recursive=true 因为字母 plane 是 lidMesh 的子对象。

6. 检查铰链位置和 lidMesh 偏移：
- lidPivot 位置：(0, 0.6, -1.2) 在 caseGroup 局部坐标
- lidMesh 相对 lidPivot 的位置：(0, 0.2, 1.2)
- 所以 lidMesh 中心在 caseGroup 局部坐标的 (0, 0.8, 0)
- lidMesh 顶面在 y = 1.0
- lidMesh 底面在 y = 0.6

盒身顶面在 y = 0.6，盒盖底面正好接触。

铰链位置 (0, 0.6, -1.2)，应该在 lidMesh 的后边缘下方。lidMesh 后边缘在 lidPivot 局部坐标 z = -1.2 + 0 = -1.2... wait，lidMesh 在 lidPivot 局部 z = 1.2 处，lidMesh 尺寸 z 方向 2.4，所以 lidMesh 后边缘在 z = 1.2 - 1.2 = 0，前边缘在 z = 1.2 + 1.2 = 2.4。

Hmm 这就有问题了。lidPivot 在 (0, 0.6, -1.2)，lidMesh 中心在 lidPivot 局部 (0, 0.2, 1.2)，所以 lidMesh 在 caseGroup 局部 (0, 0.8, 0)，覆盖 z 范围 -1.2 到 1.2。✓

铰链位置在 caseGroup 局部 (0, 0.6, -1.2)，是盒身后边缘上方。lidMesh 后边缘也在 z = -1.2，所以铰链正好在 lidMesh 后边缘下方。✓

当 lidPivot.rotation.x = 0 时，lidMesh 中心在 (0, 0.8, 0)，覆盖盒子顶部。
当 lidPivot.rotation.x = -120° (-2π/3) 时，lidMesh 围绕 lidPivot 旋转，向后翻起。

旋转矩阵：
旋转后 lidMesh 中心在 lidPivot 局部 (0, 0.2*cos(-120°) - 1.2*sin(-120°), 0.2*sin(-120°) + 1.2*cos(-120°))
= (0, 0.2*(-0.5) - 1.2*(-0.866), 0.2*(-0.866) + 1.2*(-0.5))
= (0, -0.1 + 1.039, -0.173 - 0.6)
= (0, 0.939, -0.773)

加上 lidPivot 位置 (0, 0.6, -1.2)，得到 lidMesh 中心在 caseGroup 局部 (0, 1.539, -1.973)。

打开后 lidMesh 中心在 y=1.539，z=-1.973，向上后方翻起。✓

旋转角度 -120° 可能太多了，让我用 -110° (-Math.PI * 0.61)。或者 -100°（-Math.PI * 0.55）。

测试 -110°：
cos(-110°) = -0.342, sin(-110°) = -0.94
旋转后 lidMesh 中心在 lidPivot 局部 (0, 0.2*(-0.342) - 1.2*(-0.94), 0.2*(-0.94) + 1.2*(-0.342))
= (0, -0.068 + 1.128, -0.188 - 0.41)
= (0, 1.06, -0.598)

加上 lidPivot 位置 (0, 0.6, -1.2)，得到 lidMesh 中心在 caseGroup 局部 (0, 1.66, -1.798)。

盒盖打开后中心向上后方移动。看起来合理。但是 lidMesh 自己也旋转了，所以原本朝上的面现在朝向斜后方上方。

这样从相机角度看，盒盖打开后是向后上方翻起的，可以看到盒身内部的凹槽和字母。✓

我用 -Math.PI * 0.62（约 -111.6°）应该不错。

7. 检查 RoundedBoxGeometry 的导入。在 three 0.160 版本中：
- `three/addons/geometries/RoundedBoxGeometry.js` 应该存在。

8. RoomEnvironment 的导入：
- `three/addons/environments/RoomEnvironment.js` 应该存在。

9. PointsMaterial 不支持 size attribute，需要用 ShaderMaterial 或固定 size。我用了 size attribute 但实际不会生效。让我去掉 size attribute，用统一 size。

修正：

```javascript
function createParticles() {
  const count = 180;
  const positions = new Float32Array(count * 3);
  const speeds = new Float32Array(count);
  
  for (let i = 0; i < count; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 14;
    positions[i * 3 + 1] = Math.random() * 7 - 1;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 14;
    speeds[i] = 0.002 + Math.random() * 0.004;
  }
  
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  
  // ... 粒子纹理 ...
  
  const mat = new THREE.PointsMaterial({
    size: 0.08,
    map: particleTexture,
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.5,
    sizeAttenuation: true,
  });
  
  particleSystem = new THREE.Points(geo, mat);
  particleSystem.userData.speeds = speeds;
  scene.add(particleSystem);
}
```

10. caseGroup 整体微微浮动和旋转可能让点击不稳定。让我把旋转调小或者去掉。微小的浮动 (0.04) 应该不影响点击。

实际上，浮动让盒子上下移动，但相机视角不变，可能造成视觉上盒子"漂浮"。如果太晃眼就不好。让我用更小的幅度。

11. lens 的位置：lensGeo 用 SphereGeometry 创建一个球冠，半径 0.45，深度 Math.PI * 0.18（约 32°）。球冠的最高点在 y = 0.45 * (1 - cos(32°)) ≈ 0.45 * 0.152 ≈ 0.068。球冠开口朝 +y 方向（默认）。

我希望 lens 像碗一样放在凹槽里，开口朝上。所以默认方向就是开口朝上（凹面朝上）。但是球冠的最高点在 y 方向。

让我想清楚 SphereGeometry 参数：
`SphereGeometry(radius, widthSegments, heightSegments, phiStart, phiLength, thetaStart, thetaLength)`

theta 是从 +y 轴向下的角度。thetaStart=0 表示从顶部开始。thetaLength=Math.PI * 0.18 表示从顶部向下 32.4° 的范围。

所以创建的是一个球冠，开口朝下（即球冠顶部在 +y 方向，开口在 -y 方向）。这是凸面朝上，凹面朝下。

但我想要凹面朝上（像碗），所以应该用 thetaStart = Math.PI - Math.PI * 0.18 = Math.PI * 0.82，thetaLength = Math.PI * 0.18。这样球冠开口朝上。

或者直接用 thetaStart = Math.PI * 0.82, thetaLength = Math.PI * 0.18。

Hmm 实际上让我用另一种方式：用 SphereGeometry 创建半球，然后旋转。

简单方式：用 CircleGeometry 表示扁平的镜片，加 transmission 材质。这样最简单。

或者使用 TorusGeometry 创建一个圆环，表示眼镜边缘。

实际上隐形眼镜在凹槽里浸泡，从上方看就是一个透明的圆盘。让我用扁平的 CircleGeometry：

```javascript
const lensGeo = new THREE.CircleGeometry(0.4, 64);
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.85,
  thickness: 0.05,
  ior: 1.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
  transparent: true,
  opacity: 0.7,
});

const leftLens = new THREE.Mesh(lensGeo, lensMat);
leftLens.rotation.x = -Math.PI / 2;  // 平放
leftLens.position.set(-wellX, 0.49, 0);
caseGroup.add(leftLens);
```

这样镜片是平的圆盘，水平放置在液体表面附近。简单且看起来合理。

但是 transmission + transparent 一起用可能有问题。MeshPhysicalMaterial 用 transmission 时不需要 transparent: true，因为 transmission 自己处理透明度。

让我去掉 transparent:

```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.9,
  thickness: 0.1,
  ior: 1.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
});
```

OK 这样应该可以。

但是 transmission 需要 renderer 设置 transparent background 或者支持。MeshPhysicalMaterial 的 transmission 在 WebGL 2 下可以工作，需要 renderer.physicallyCorrectLights 等。Three 0.160 应该默认支持。

实际上 transmission 需要渲染目标，会自动启用。但可能影响性能。

让我简化：用 transparent + opacity 的 MeshStandardMaterial 来模拟透明：

```javascript
const lensMat = new THREE.MeshStandardMaterial({
  color: COLORS.lensTint,
  metalness: 0.2,
  roughness: 0.1,
  transparent: true,
  opacity: 0.5,
  side: THREE.DoubleSide,
});
```

这样性能更好，效果也够。

对于液体，我同样简化：

```javascript
const liquidMat = new THREE.MeshStandardMaterial({
  color: COLORS.liquid,
  metalness: 0.3,
  roughness: 0.05,
  transparent: true,
  opacity: 0.7,
  side: THREE.DoubleSide,
  emissive: COLORS.liquid,
  emissiveIntensity: 0.15,
});
```

不用 transmission，简单稳定。

12. 检查 lidMesh raycast：lidMesh 是 RoundedBoxGeometry，应该能正确 raycast。子对象（字母 plane）也能 raycast，只要 geometry 存在。

但是字母 plane 的材质是 transparent + depthWrite: false，这不影响 raycast，只影响渲染。raycast 是基于 geometry 的，与材质无关。

13. loader 处理：页面加载时显示 loader，500ms 后隐藏。Three.js 模块加载和场景初始化很快，应该够。

14. 性能考虑：
- RoundedBoxGeometry 有 8 段，segments 较多
- CircleGeometry 64 段，可以减少到 32
- 总三角形数量应该不多，性能 OK

15. 视觉细节：
- 盒盖打开时，盖子顶面的字母会朝下/朝后，不在相机视野
- 盒身顶面（凹槽旁边）的字母在打开后可见
- 关闭时主要看盒盖顶面的大字母

16. 字母纹理在 lidMesh 关闭时离盒盖表面很近（0.005），可能 z-fighting。让我把字母放在 y = 0.01（lidHeight/2 + 0.01）。

17. 检查 lidMesh 子对象坐标系：lidMesh 是 RoundedBoxGeometry(4.4, 0.4, 2.4)，lidMesh 中心在 lidPivot 局部 (0, 0.2, 1.2)。lidMesh 子对象的坐标系是 lidMesh 局部，所以 lidMesh 顶面在子对象坐标系 y = 0.2。字母放在 y = 0.205。

让我修正：
```javascript
const topL = createLetterPlane('L', 1.1, true);
topL.position.set(-wellX, 0.205, 0);
topL.rotation.x = -Math.PI / 2;
lidMesh.add(topL);
```

lidHeight/2 = 0.2，字母放在 0.205 离表面 0.005。如果还 z-fighting，再加。

实际让我用 0.21：
```javascript
topL.position.set(-wellX, 0.21, 0);
```

这样离表面 0.01，应该没问题。

18. 盒身顶面字母位置：盒身顶面在 y = 0.6，字母放在 y = 0.61。这样离表面 0.01。

让我重新算凹槽位置和字母位置：
- 盒身：4.4 x 1.2 x 2.4
- 凹槽中心：(-1.15, 0.36, 0) 和 (1.15, 0.36, 0)
- 凹槽半径 0.7，深度 0.5
- 凹槽顶部在 y = 0.36 + 0.25 = 0.61
- 凹槽底部在 y = 0.36 - 0.25 = 0.11

但是凹槽高度 0.5 在盒身高度 1.2 内，盒身顶面在 y = 0.6，盒身底面在 y = -0.6。凹槽底部 y = 0.11，离盒身底面 0.71。✓

凹槽侧壁 CylinderGeometry 高度 0.5，所以从 y = 0.11 到 y = 0.61。

字母位置 y = 0.61（贴在盒身顶面，凹槽边缘）。

但是凹槽边缘的金色装饰圈在 y = 0.6，字母在 y = 0.61，会和装饰圈接近。让我把装饰圈放在 y = 0.61，字母放在 y = 0.62。

或者把字母放在凹槽前后方（z = ±0.95），避开装饰圈。

OK 字母在 (-1.15, 0.61, 0.95) 和 (1.15, 0.61, 0.95)，远离凹槽装饰圈（在 (-1.15, 0.6, 0)），距离 √(0.95²) = 0.95，足够远。✓

19. 字母大小 0.45，在 4.4 x 2.4 的盒身顶面，凹槽旁边 0.5 x 0.5 的空间。字母直径 0.45 占据 0.45 x 0.45，刚好放下。

20. 字母朝向：字母用 PlaneGeometry，默认在 xy 平面。我用 rotation.x = -Math.PI / 2 让它平铺在 xz 平面，朝上。这样从上方看是字母。

字母 z 朝向：原来 plane 朝 +z，旋转 -π/2 后朝 +y。但是字母在 canvas 上的方向，旋转后会怎样？

Canvas 上字母 "L" 是正向的（从左到右，从下到上）。PlaneGeometry 默认在 xy 平面，字母在 +z 方向看是正向的。旋转 -π/2 绕 x 轴，plane 法线从 +z 变为 +y，字母从 +y 方向看是正向的。

但是字母的"上下"方向呢？PlaneGeometry 的 +y 是上方，旋转后 +y 变成 +z（向后）。所以字母的"上"指向 -z（向前）。

从相机角度看（相机在 +y 方向往下看），字母的"上"指向相机方向（+z 是相机方向）。所以字母看起来是正向的。✓

但是字母的"下"指向 +z（后方）。从相机角度看，字母的下方在远处，上方在近处。这样字母看起来是"倒立"的？

不对，从相机往下看，z 方向是前后。字母的"上"在 -z（前方/近处），字母的"下"在 +z（后方/远处）。从相机视角，字母的上方靠近相机，下方远离。这样字母看起来是正向的（上方靠近相机，下方远离）。

实际上是这样的：当你俯视一个物体，物体的"上"边靠近你（相机），"下"边远离你。这是正向的视角。✓

所以字母朝向应该正确。

实际上让我想清楚：从相机俯视，平面在 xz 平面（朝上），原 plane 的 +y 方向变成 -z 方向。字母原本在 plane 上，"上"在 +y。旋转后字母的"上"在 -z 方向（靠近相机）。

字母 L 的形状：竖线在左，横线在下。在 canvas 上：
```
| 
|
|___
```

在 plane 上，L 的竖线在 -x 方向（左），横线在 -y 方向（下）。旋转 -π/2 绕 x 轴后，L 的竖线还在 -x 方向（左），横线从 -y 变成 +z 方向（后）。

从相机俯视，L 的形状：
- 竖线在左 ✓
- 横线在远处（后方）✗ 

实际应该是横线在近处（前方），但我现在是横线在远处。所以字母看起来是倒立的。

让我重新想：L 字母的标准形状是
```
|_
| 
|
```

竖线在左边从上到下，横线在底部从左到右。在 canvas 上画 L 时：
```
|
|
|__
```

L 的"底"在 canvas 的 +y 方向（如果 canvas 的 y 向下为正）。Wait，canvas 的 y 向下为正。

让我用屏幕坐标：canvas 的 (0,0) 在左上角，y 向下。L 字母在 canvas 中心绘制时，竖线在左边，横线在底部（y 较大处）。

PlaneGeometry 默认在 xy 平面，y 朝上。字母 canvas 映射到 plane 上时，canvas 的 y 朝下对应 plane 的 y 朝下（即 -y 方向）。

具体来说：PlaneGeometry 的 UV 坐标 (0,0) 在左下角，(1,1) 在右上角。Canvas 的 (0,0) 在左上角，映射到 UV (0,1)（左下角）。所以 canvas y=0 (顶部) 对应 plane y=+1 (上方)，canvas y=canvas.height (底部) 对应 plane y=-1 (下方)。

所以 L 在 canvas 上：
- 竖线在 canvas 左边 (x=0.25) → plane x=-0.5 (左边)
- 横线在 canvas 底部 (y=0.75) → plane y=-0.5 (下方)

Plane 上 L 形状：
- 竖线在 plane 左边 (-x)
- 横线在 plane 底部 (-y)

L 看起来是正向的：
```
|
|
|__
```

旋转 -π/2 绕 x 轴后：plane 法线从 +z 变为 +y（朝上），plane 的 +y 变为 -z（朝前），plane 的 -y 变为 +z（朝后）。

旋转后 L 形状：
- 竖线还在 -x 方向（左边）
- 横线从 -y 变为 +z 方向（朝后）

从相机俯视（相机在 +y 方向，往 -y 方向看），看到的是 xz 平面：
- x 方向：左到右
- z 方向：远（+z，朝远处）到近（-z，朝相机）

L 形状在 xz 平面：
- 竖线在 -x 方向（左）
- 横线在 +z 方向（远）

从相机俯视，L 看起来：
```
|     (远处)
|
|____ (近处？不，远处)
```

Wait，从相机看下去，+z 是远处。所以横线在远处。

这样 L 的横线在远处（上方），竖线在近处到远处。从相机视角，L 看起来是竖线在左，横线在上方。这是倒立的 L！

实际上让我重新想。L 字母的标准方向（在文本中）是：
```
|
|
|___
```
竖线在左，横线在底部。

从相机俯视一个平铺的 L，应该看到 L 的横线在底部（靠近相机）。

但我现在的旋转，L 的横线在 +z（远离相机），所以看起来 L 是倒立的（横线在上方/远处）。

修复：旋转 +π/2 而不是 -π/2。让我重新算。

旋转 +π/2 绕 x 轴：plane 法线从 +z 变为 -y（朝下）。

Hmm 这样字母朝下了，从上面看不到。

让我用 rotation.x = -π/2，但是字母在 canvas 上是倒立的，或者调整 canvas 上的字母方向。

简单方法：在 canvas 上把字母上下翻转。或者在 plane 上调整 UV。

或者用 rotation.x = -π/2 + π = π/2，让 plane 朝上但字母方向相反。

实际上让我用 rotation.x = -Math.PI / 2，看看效果。如果字母倒立，调整 canvas 绘制时让字母倒立（y=0 在 canvas 底部）。

或者最简单：用 canvas.fillText 时，先 ctx.translate 然后 ctx.scale(1, -1) 翻转 y 轴。

```javascript
ctx.save();
ctx.translate(size/2, size/2);
ctx.scale(1, -1);
ctx.fillText(letter, 0, 0);
ctx.restore();
```

这样 canvas 上的字母是上下翻转的，映射到 plane 后字母是正向的。

让我测试一下：
- Canvas 上字母翻转（横线在 canvas 顶部 y=0，竖线在 canvas 左边）
- UV 映射：canvas y=0 → plane y=+1（上方）
- Plane 上字母：横线在 +y（上方），竖线在 -x（左）
- Plane 上字母看起来：
  ```
  ___
  |
  |
  ```
  这不是 L，是希腊字母 Gamma 或者反向 L。

Hmm 这样也不对。让我换个思路。

直接在 plane 上画 L 的形状，L 的标准方向是竖线在左，横线在底部（-y 方向）。我希望从相机俯视看到这个形状。

相机在 +y 方向看 -y 方向，看到的 xz 平面：x 是左到右，z 是远到近（+z 远，-z 近）。

我希望 L 的横线在 -z 方向（近处），竖线在 -x 方向（左）。

Plane 上 L 形状（在 xy 平面）：
- 竖线在 -x 方向（左）
- 横线在 -y 方向（下）

旋转后 L 形状应该在 xz 平面：
- 竖线在 -x 方向（左）
- 横线在 -z 方向（近）

旋转 -π/2 绕 x 轴：
- Plane 的 +y 旋转后变为 -z（近）
- Plane 的 -y 旋转后变为 +z（远）

所以旋转后：
- 横线从 -y 变为 +z（远）
- 竖线还在 -x（左）

横线在远处，这是倒立的 L。

我需要旋转 +π/2 绕 x 轴：
- Plane 的 +y 旋转后变为 +z（远）
- Plane 的 -y 旋转后变为 -z（近）

但是 +π/2 旋转后 plane 法线从 +z 变为 -y（朝下）。这样字母朝下，相机看不到。

要让字母朝上又能正确显示方向，需要：
1. 字母朝上：plane 法线朝 +y
2. 字母横线在 -z（近）

Plane 默认法线朝 +z。要让法线朝 +y，绕 x 轴旋转 -π/2。

但 -π/2 旋转后，横线在 +z（远）。

要让横线在 -z（近），需要在 plane 上把字母翻转（横线在 +y 方向）。

所以方案：
1. Plane 默认在 xy 平面，法线 +z
2. 在 canvas 上画 L 时，把 L 翻转（横线在 canvas 顶部，竖线在 canvas 左）
3. 这样 plane 上 L 的横线在 +y 方向，竖线在 -x 方向
4. 旋转 -π/2 绕 x 轴后，plane 上 +y 变为 -z，横线在 -z 方向（近处）
5. 从相机俯视，L 横线在近处，竖线在左边，这是正向的 L ✓

所以我在 canvas 上画字母时需要翻转 y。

```javascript
function createLetterTexture(letter, withGlow = false) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  // ... 阴影设置 ...
  
  // 翻转 y 轴
  ctx.save();
  ctx.translate(size/2, size/2);
  ctx.scale(1, -1);
  
  // ... 渐变填充 ...
  
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, 0, -10); // 注意 y 也翻转了
  
  ctx.restore();
  
  // ...
}
```

实际上让我用更简单的方法：直接调整 plane 的 rotation，让字母朝上且方向正确。

我可以用 rotation.x = Math.PI / 2（绕 x 轴旋转 +90°），这样 plane 法线朝 -y（朝下），字母朝下。这不是我要的。

或者用 rotation.x = -Math.PI / 2，字母朝上但倒立。

或者用 rotation.x = Math.PI / 2 然后调整 plane 的 UV。

最简单的方案：用 DoubleSide 材质，字母在两面都可见。然后用 rotation.x = -Math.PI / 2 让字母朝上，即使方向"倒立"，但因为字母本身对称性不强（L 不对称），可能看起来不对。

Wait 实际上让我重新审视这个问题。

L 的形状（在 canvas 上正常画）：
```
| 
|
|___
```

Canvas 坐标：y 向下为正。L 的竖线在 canvas x=0.3*size, y=0.1*size 到 y=0.9*size。横线在 canvas y=0.9*size, x=0.3*size 到 x=0.7*size。

PlaneGeometry 的 UV：左下角 (0,0)，右上角 (1,1)。

Canvas 到 UV 的映射：
- Canvas (0,0) = UV (0, 1)
- Canvas (size, 0) = UV (1, 1)
- Canvas (0, size) = UV (0, 0)
- Canvas (size, size) = UV (1, 0)

所以 canvas y=0 (顶部) → UV v=1 → plane y=+0.5 (上方)
Canvas y=size (底部) → UV v=0 → plane y=-0.5 (下方)

L 的横线在 canvas y=0.9*size → plane y=-0.4 (下方)
L 的竖线在 canvas x=0.3*size → plane x=-0.2 (左边)

Plane 上 L 形状：
```
|      
|      
|___   
```
（竖线在左边，横线在底部）✓ 这是正向的 L。

Plane 法线默认朝 +z。我用 rotation.x = -Math.PI / 2 旋转，plane 法线变为 +y（朝上）。

旋转 -π/2 绕 x 轴的变换：
- 原 (x, y, z) → 新 (x, -z, y)
- 所以 plane 上点 (x, y, 0) → (x, 0, y)
- plane 的 +y 方向 → +z 方向
- plane 的 -y 方向 → -z 方向

L 在 plane 上的形状：
- 竖线在 plane (-0.2, ?, 0)
- 横线在 plane (?, -0.4, 0)

旋转后 L 在世界坐标的形状：
- 竖线在 (-0.2, 0, ?)，? 从 -0.4 到 0.4（原 plane y 从 -0.4 到 0.4）
- 横线在 (?, 0, -0.4)

相机在 (5, 3.2, 6.5) 看向原点。从相机俯视：
- x 方向：右（+x）右
- z 方向：相机在 +z，看向 -z。所以 +z 朝向相机（近处），-z 远离相机（远处）

L 旋转后形状：
- 竖线在 (-0.2, 0, ?)，? 从 -0.4（远）到 0.4（近）
- 横线在 (?, 0, -0.4)（远）

从相机俯视：
- 竖线在左边（-x），从远到近
- 横线在远处（-z）

L 看起来：
```
       (远)
   |   
   |   
___|  (这是横线，在远处)
       (近)
```

横线在远处，竖线在左边从远到近。这样从相机看，L 的横线在上方（远处），竖线在下方（近处）。

正向的 L 应该是：
```
       (远)
___|  
   |  
   |   
       (近)
```

横线在近处（下方），竖线在左边从近到远。

所以现在 L 是"倒立"的，横线在远处。这是不对的。

修复：在 canvas 上画 L 时，把 L 上下翻转（横线在顶部）。然后 plane 上 L 的横线在 +y（上方），竖线在 -x（左）。旋转 -π/2 后，横线在 -z（近处），竖线在 -x（左）。这是正向的 L。✓

或者用 rotation.x = Math.PI / 2 让 plane 法线朝 -y（朝下）。但这样字母朝下，从上面看不到。需要从下面看才能看到字母。

Wait, 用 DoubleSide 材质字母两面都能看到。但平面朝下时，从上面看是平面的背面。字母在背面看起来是镜像的（左右翻转）。

让我试试 rotation.x = Math.PI / 2 + DoubleSide：

旋转 +π/2 绕 x 轴：
- 原 (x, y, z) → 新 (x, z, -y)
- plane 的 +y 方向 → -z 方向（远）
- plane 的 -y 方向 → +z 方向（近）

L 旋转后：
- 竖线在 (-x, 0, ?)，? 从 -0.4 到 0.4（原 plane y 从 -0.4 到 0.4），但 plane y 旋转后变为 -z，所以原 plane y=+0.4 → z=-0.4 (远)，原 plane y=-0.4 → z=+0.4 (近)
- 所以竖线从 z=+0.4（近）到 z=-0.4（远）
- 横线在 (?, 0, +0.4)（近）

从相机俯视：
- 竖线在左边，从近到远
- 横线在近处（下方）

这样 L 是正向的。✓

但 plane 法线朝 -y（朝下）。从相机俯视看到的是 plane 的背面。字母在背面看起来是镜像的（左右翻转）。

Wait plane 的两面都有字母吗？如果用 CanvasTexture，纹理只在一面显示，另一面是背面（黑色或透明）。如果用 DoubleSide，两面都显示。

但 DoubleSide 时，背面看到的字母是镜像的。

我用 DoubleSide 材质，从相机看 plane 的背面。背面看到的字母左右翻转。

L 的形状左右翻转后变成：
```
   |
   |
___|
```
这是反向的 L（希腊字母 Gamma）。

所以也不行。

正确的方案是：在 canvas 上画字母时上下翻转（让字母横线在 canvas 顶部，即 y=0 处）。然后 plane 上字母的横线在 +y（上方）。旋转 -π/2 绕 x 轴后，横线在 -z（近处）。

让我用这个方案：

```javascript
function createLetterTexture(letter, withGlow = false) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  // 阴影和渐变设置...
  
  // 翻转 y 轴，让字母在 canvas 上上下颠倒
  ctx.save();
  ctx.translate(0, size);
  ctx.scale(1, -1);
  
  // 阴影
  if (withGlow) {
    ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
    ctx.shadowBlur = 80;
  } else {
    ctx.shadowColor = 'rgba(255, 180, 60, 0.6)';
    ctx.shadowBlur = 30;
  }
  
  // 渐变填充
  const gradient = ctx.createLinearGradient(0, 0, 0, size);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.6)';
  ctx.lineWidth = 5;
  ctx.strokeText(letter, size/2, size/2 + 10);
  
  ctx.restore();
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  texture.needsUpdate = true;
  return texture;
}
```

这样在 canvas 上 L 的横线在顶部，竖线在底部。映射到 plane 上，L 的横线在 +y（上方），竖线在 -y（下方）。

旋转 -π/2 绕 x 轴后：
- Plane 的 +y → -z（近）
- Plane 的 -y → +z（远）

L 在 xz 平面：
- 横线在 -z（近）
- 竖线从 +z（远）到 -z（近）

从相机俯视：
- 横线在近处（下方）
- 竖线在左边，从远到近

L 看起来：
```
|      
|  (远)
|      
|      
___|  (近)
```

这是正向的 L！✓

但是这样 canvas 上 L 是上下颠倒的，纹理本身看起来奇怪。不过没关系，最终在 3D 中显示正确就行。

Wait，渐变填充也是上下颠倒的（因为整个 context 翻转了）。但渐变是从上到下的，翻转后是从下到上。这没问题，只是渐变方向反了。

让我让渐变方向也翻转一下，使最终显示时渐变从字母上方 #ffe066 到下方 #ff7e2e：

实际上翻转后，渐变在 canvas 上看是从下到上（#ffe066 在底部，#ff7e2e 在顶部）。映射到 plane 上，#ffe066 在 +y（上方），#ff7e2e 在 -y（下方）。旋转后 #ffe066 在 -z（近），#ff7e2e 在 +z（远）。从相机看，字母上方（远）是 #ff7e2e，下方（近）是 #ffe066。

如果想字母上方是亮色（#ffe066），下方是深色（#ff7e2e），我需要让渐变方向反过来。让我在翻转后的 context 上画渐变从 (0, 0) 到 (0, size)（在翻转后的坐标系中），这样渐变 #ffe066 在翻转后的顶部（即 canvas 的底部 = plane 的下方 = 旋转后的近处）。

让我重新设计：
- 我希望从相机看，字母上方（远处）是亮色 #ffe066，下方（近处）是深色 #ff7e2e
- 在 plane 上：上方 = +y = 旋转后 = -z = 近处
- 等等，不对。Plane 的 +y 旋转 -π/2 绕 x 轴后变为 -z（近）。
- 所以 plane 上 +y（上方）= 旋转后近处 = 字母下方（从相机看）
- Plane 上 -y（下方）= 旋转后远处 = 字母上方（从相机看）

我希望字母上方（远处）是亮色：
- 字母上方 = 远处 = plane -y
- 渐变亮色 #ffe066 应该在 plane -y
- 渐变深色 #ff7e2e 应该在 plane +y
- 在 plane 局部坐标，渐变从 -y (亮) 到 +y (深)
- UV v=0 对应 plane -y（下方），UV v=1 对应 plane +y（上方）
- 所以渐变从 UV v=0 (亮) 到 v=1 (深)
- UV v=0 对应 canvas y=size（底部）
- UV v=1 对应 canvas y=0（顶部）
- 所以渐变在 canvas 上从 y=size (亮) 到 y=0 (深)
- 即从下到上，从亮到深

在翻转后的坐标系（ctx.translate(0, size); ctx.scale(1, -1);）：
- 翻转后 canvas y=0 对应原 canvas y=size
- 翻转后 canvas y=size 对应原 canvas y=0
- 所以翻转后，渐变从 y=size (亮) 到 y=0 (深)，对应原 canvas 从 y=0 (亮) 到 y=size (深)

我创建渐变时用 ctx.createLinearGradient(0, 0, 0, size)，这在翻转后的坐标系中是从 (0,0) 到 (0,size)，即从顶部到底部（翻转后的视角）。

如果我要在翻转后的坐标系中渐变从底部 (亮) 到顶部 (深)，应该用 ctx.createLinearGradient(0, size, 0, 0)。

Hmm 这太复杂了。让我简化：不管渐变方向，只要字母显示正确就行。颜色细节用户不会太在意。

让我直接用最简单的方案：在 canvas 上正向画字母（不翻转），然后调整 plane 的 rotation。

正向画的字母：canvas 上 L 横线在底部。映射到 plane：横线在 -y。旋转 -π/2 绕 x 轴后横线在 +z（远）。从相机看横线在远处，字母倒立。

要让字母正向显示，我可以旋转 plane 的另一种方式：rotation.x = -Math.PI / 2 + Math.PI = Math.PI / 2，但这让 plane 朝下。

或者旋转 +π 然后再 rotation.x = -π/2。即在 plane 上让字母上下翻转，再旋转 -π/2。

具体：
1. Canvas 上正向画字母（横线在 canvas 底部）
2. Plane 上字母横线在 -y（下方）
3. 我希望旋转后字母横线在 -z（近处）
4. 需要让 plane 的 -y 方向旋转后变为 -z 方向
5. 旋转 -π/2 绕 x 轴：plane +y → -z, plane -y → +z。所以 plane -y → +z（远）。不对。
6. 旋转 +π/2 绕 x 轴：plane +y → +z, plane -y → -z。所以 plane -y → -z（近）。对了！

但是旋转 +π/2 绕 x 轴后 plane 法线从 +z 变为 -y（朝下）。字母朝下，从上面看不到。

解决：用 DoubleSide 材质，从上面看到的是 plane 的背面。但背面字母是左右镜像的。

L 左右镜像后是 Γ (Gamma) 形状。这不是 L。

所以这个方案也不行。

实际上最直接的方案就是在 canvas 上翻转字母（让横线在 canvas 顶部），然后旋转 -π/2。这样字母正向显示。我用这个方案。

但是字母在 canvas 上是上下颠倒的，渐变方向等也跟着颠倒。最终在 3D 中字母正向显示。

或者，更简单：用 sprite 或 billboard，让字母始终面向相机。但这样字母会随相机移动，不像贴在物体表面。

OK 让我用 canvas 翻转方案。

简化代码：
```javascript
function createLetterTexture(letter, withGlow = false) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  // 翻转坐标系，让字母在 3D 中正向显示
  ctx.save();
  ctx.translate(size/2, size/2);
  ctx.scale(1, -1);
  
  if (withGlow) {
    ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
    ctx.shadowBlur = 80;
  } else {
    ctx.shadowColor = 'rgba(255, 180, 60, 0.6)';
    ctx.shadowBlur = 30;
  }
  
  const gradient = ctx.createLinearGradient(0, -size/2, 0, size/2);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, 0, 10);
  
  // 加强发光
  ctx.shadowBlur = 25;
  ctx.fillText(letter, 0, 10);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.6)';
  ctx.lineWidth = 5;
  ctx.strokeText(letter, 0, 10);
  
  ctx.restore();
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  texture.needsUpdate = true;
  return texture;
}
```

这样字母在 3D 中正向显示。

实际上还有一种更简单的方法：直接在 PlaneGeometry 上设置 rotation，让 plane 朝上且字母正向。

Plane 默认朝 +z。要让 plane 朝上（法线 +y），需要绕 x 轴旋转 -π/2。这时字母横线在 +z（远）。

如果我让 plane 默认朝 -z（即翻转 plane），用 rotation.y = π 让 plane 翻转 180°。这时字母在 plane 背面看是左右镜像。

不行。

或者直接让 plane 朝下（法线 -y），用 DoubleSide 材质，从上面看背面字母是镜像的。但要镜像字母让它在背面正向显示。

最直接的：在 canvas 上画字母时翻转 y。

OK 我决定用 canvas 翻转方案。

让我重新检查一下旋转后字母的方向。让我用具体例子。

假设 canvas 上 L 是正向的（竖线在左，横线在底）：
- Canvas x 轴向右，y 轴向下
- Canvas 上 L 竖线在 (0.3*size, 0.1*size) 到 (0.3*size, 0.9*size)
- Canvas 上 L 横线在 (0.3*size, 0.9*size) 到 (0.7*size, 0.9*size)

如果我在 canvas 上翻转 y（ctx.translate(0, size); ctx.scale(1, -1);）：
- Canvas 上 L 竖线在 (0.3*size, 0.1*size) 到 (0.3*size, 0.9*size)
- 翻转后竖线在 (0.3*size, size - 0.1*size) 到 (0.3*size, size - 0.9*size) = (0.3*size, 0.9*size) 到 (0.3*size, 0.1*size)
- 等等，翻转后 y 坐标变换：原 y → size - y
- 原竖线 y 从 0.1 到 0.9 → 翻转后 y 从 0.9 到 0.1（即从底部到顶部）
- 原横线 y = 0.9 → 翻转后 y = 0.1（顶部）
- 所以翻转后 L 在 canvas 上：竖线在左边从顶部到底部，横线在顶部

L 翻转后看起来：
```
___
|
|
```
这是希腊字母 Γ。

映射到 plane（UV 映射）：
- Canvas y=0 (顶部) → UV v=1 → plane y=+0.5 (上方)
- Canvas y=size (底部) → UV v=0 → plane y=-0.5 (下方)
- 翻转后 L 横线在 canvas y=0.1 (顶部附近) → plane y=+0.4 (上方)
- 翻转后 L 竖线在 canvas x=0.3 (左边) → plane x=-0.2 (左边)

Plane 上 L 形状：
- 横线在 +y（上方）
- 竖线在 -x（左边），从 +y 到 -y

旋转 -π/2 绕 x 轴：
- Plane +y → -z (近)
- Plane -y → +z (远)
- Plane -x → -x (不变)

旋转后 L 形状（在 xz 平面）：
- 横线在 -z (近)
- 竖线在 -x (左)，从 -z (近) 到 +z (远)

从相机俯视（相机在 +y 看 -y 方向，+z 是近处）：
- 横线在近处（下方）
- 竖线在左边，从近到远

L 看起来：
```
|       (远)
|
|
|___    (近)
```

这是正向的 L！✓

OK 我用 canvas 翻转方案。让我修正代码。

Wait 我之前一直搞错 canvas 的 y 方向。Canvas 的 y 是向下为正。所以"顶部"是 y=0，"底部"是 y=size。

我用 ctx.fillText 在 canvas 中心画字母时，textBaseline='middle' 让字母垂直居中。所以字母中心在 y=size/2。

字母的"顶部"（视觉上）在 canvas y < size/2 处。
字母的"底部"（视觉上）在 canvas y > size/2 处。

对于 L：
- 竖线在 canvas x ≈ 0.3*size, y 从 0.1*size 到 0.9*size
- 横线在 canvas y ≈ 0.9*size, x 从 0.3*size 到 0.7*size

翻转 y 后（ctx.translate(0, size); ctx.scale(1, -1)）：
- Canvas y 变换：原 y → size - y
- 竖线 y 从 0.1*size 到 0.9*size → 翻转后 y 从 0.9*size 到 0.1*size（即从底部到顶部）
- 横线 y = 0.9*size → 翻转后 y = 0.1*size（顶部）

所以翻转后 L 在 canvas 上：
- 竖线在 x=0.3*size, y 从 0.1*size (顶部) 到 0.9*size (底部)
- 横线在 y=0.1*size (顶部), x 从 0.3*size 到 0.7*size

L 形状（在 canvas 上）：
```
___   (顶部，y=0.1)
|
|
|     (底部，y=0.9)
```

这是 Γ 形状（横线在顶部）。

映射到 plane：canvas y=0 (顶部) → plane y=+0.5 (上方)；canvas y=size (底部) → plane y=-0.5 (下方)。

所以翻转后 L 在 plane 上：
- 横线在 plane +y (上方)
- 竖线在 -x (左)

Plane 上形状：
```
___   (上方，+y)
|
|
|     (下方，-y)
```

这是 Γ 形状在 plane 上。

旋转 -π/2 绕 x 轴后：
- Plane +y → -z (近)
- Plane -y → +z (远)
- Plane -x → -x

旋转后形状：
- 横线在 -z (近)
- 竖线在 -x (左)，从 -z (近) 到 +z (远)

从相机俯视，相机在 (5, 3.2, 6.5)，看向原点。相机 +x 方向（右），+z 方向（朝向相机/近处），-z 方向（远离相机/远处）。

从相机俯视，L 在 xz 平面：
- 横线在 -z (近处，下方屏幕)
- 竖线在 -x (左边)，从 -z (近处) 到 +z (远处)

L 看起来：
```
|       (远，+z)
|
|
|___    (近，-z)
```

这是正向的 L！✓

OK 所以方案：canvas 上画字母时翻转 y。我用这个方案。

让我重新写 createLetterTexture：

```javascript
function createLetterTexture(letter, withGlow = false) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  // 翻转 y 轴，让字母在 3D 中正向显示
  ctx.save();
  ctx.translate(0, size);
  ctx.scale(1, -1);
  
  // 阴影
  if (withGlow) {
    ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
    ctx.shadowBlur = 80;
  } else {
    ctx.shadowColor = 'rgba(255, 180, 60, 0.6)';
    ctx.shadowBlur = 30;
  }
  
  // 渐变填充
  const gradient = ctx.createLinearGradient(0, size * 0.1, 0, size * 0.9);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  
  // 在翻转后的坐标系中，画字母
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 加强发光
  ctx.shadowBlur = 25;
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.6)';
  ctx.lineWidth = 5;
  ctx.strokeText(letter, size/2, size/2 + 10);
  
  ctx.restore();
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  texture.needsUpdate = true;
  return texture;
}
```

这样应该没问题。

不过 wait，对于字母在 lidMesh 上的情况，lidMesh 旋转后字母的方向也会变。我需要考虑这点。

lidMesh 关闭时（rotation.x = 0），lidMesh 的局部坐标和 caseGroup 一样。字母 plane 在 lidMesh 局部坐标 (x, 0.21, z)，rotation.x = -π/2。这种情况下字母在 lidMesh 顶面，正向显示（从相机俯视）。

lidMesh 打开时（rotation.x = -120°），lidMesh 整体旋转。字母 plane 跟着旋转，字母方向也变。这时字母可能朝下或朝后，相机看不到或看到的是镜像。

但是 lidMesh 打开时，用户主要看盒身内部的字母，不看 lidMesh 上的字母。所以这个方向变化不重要。

OK 字母方向问题解决了。

让我整理整个代码：

最后还要测试：
1. 盒盖默认关闭，顶面 L 和 R 标识醒目可见
2. 点击盒盖，盒盖打开，铰链动画流畅
3. 盒盖打开后，盒身内部凹槽、液体、隐形眼镜、L 和 R 标识可见
4. 内部灯光亮起
5. 再次点击盒盖，盒盖关闭
6. 按钮也可以切换盒盖
7. 旋转视角（OrbitControls）
8. 缩放（鼠标滚轮）

让我检查一下 OrbitControls 的设置：
- enablePan = false 防止平移
- minDistance/maxDistance 限制缩放
- maxPolarAngle = π * 0.5 防止看到地板下方

视觉细节：
- 盒身有金属质感
- 盒盖有更亮的高光
- 凹槽有金色装饰圈
- 液体半透明，有微小波动
- 隐形眼镜透明
- 字母金色发光
- 粒子飘动
- 地板有阴影和装饰圈
- 整体微微浮动

颜色协调：
- 主色：墨绿 (#0d3a36)
- 辅色：金色 (#f5a623)
- 强调：青色 (#4ecdc4)
- 背景：深炭灰
- 这个配色避免了蓝紫色

让我再检查 lid 的 raycast 检测范围。当 lidMesh 旋转后，它的子对象 plane 也跟着旋转。但是 raycast 检测的是 lidMesh 的 geometry（RoundedBoxGeometry），不管 lidMesh 是否旋转都能检测到。

只是 raycast 检测 lidMesh 时，会检测 lidMesh 的世界坐标变换后的 geometry。当 lidMesh 旋转后，geometry 在世界中的位置也变了。但 lidMesh 还是在场景中，可以被 raycast 检测到。

只是用户点击 lidMesh 的位置可能不同了（旋转后 lidMesh 在不同位置）。

如果用户在盒盖打开状态下点击 lidMesh，会触发关闭。这是合理的。

OK 让我整理代码。

Wait 我刚才在 createLetterPlane 中没传 withGlow 参数时默认是 false。但是字母平面在 lidMesh 上需要发光，所以传 true。在盒身上字母不发光（默认 false）。

实际上为了视觉一致性，让盒身上的字母也微微发光。让我都用 withGlow = true，但调整 glow 强度。

或者只用透明度区分：盒盖字母大、强发光；盒身字母小、弱发光。

OK 我决定：
- 盒盖字母：大（1.1）、强发光（withGlow=true）
- 盒身字母：小（0.45）、无发光（withGlow=false），但有阴影

这样盒盖上的字母更醒目。

Wait 但是用 AdditiveBlending 时，字母叠加在背景上，颜色会更亮。盒身字母不用 AdditiveBlending，用 normal blending，颜色就是字母本身的颜色。

让我用 normal blending 都行，但是用 emissive 让字母发光。或者就用 CanvasTexture 加 shadow 模拟发光。

实际上 MeshBasicMaterial 不支持 emissive。MeshStandardMaterial 才支持。但 MeshStandardMaterial 受光照影响，如果用在没有光照的地方可能不够亮。

让我用 MeshBasicMaterial + CanvasTexture + AdditiveBlending 来让字母发光。这样字母颜色加到背景上，看起来发光。

对所有字母都用 AdditiveBlending：

```javascript
function createLetterPlane(letter, scale = 0.5) {
  const texture = createLetterTexture(letter);
  
  const planeGeo = new THREE.PlaneGeometry(scale, scale);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
  });
  
  return new THREE.Mesh(planeGeo, planeMat);
}
```

但是 AdditiveBlending 让颜色叠加，可能太亮。在亮背景下字母会消失（白色 + 白色 = 白色）。

让我用 normal blending，字母纹理本身就有发光效果（通过 shadow blur）：

```javascript
function createLetterPlane(letter, scale = 0.5) {
  const texture = createLetterTexture(letter);
  
  const planeGeo = new THREE.PlaneGeometry(scale, scale);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
  });
  
  return new THREE.Mesh(planeGeo, planeMat);
}
```

这样字母显示其纹理颜色，包括发光阴影。在暗背景下看起来发光。

我用 normal blending。

让我最终确定代码。整理一下：

完整 HTML：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LUMINA · 隐形眼镜盒 3D 模型</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;700&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500&display=swap" rel="stylesheet">

<style>
  /* CSS 部分 */
</style>
</head>
<body>
  <!-- HTML 内容 -->
  
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>
  
  <script type="module">
    /* JS 部分 */
  </script>
</body>
</html>
```

让我写出完整代码。

不过我还需要测试一些边界情况：
- 浏览器不支持 WebGL 时怎么办？
- 资源加载失败时怎么办？
- 移动设备触摸事件怎么处理？

触摸事件：pointerdown/pointerup/pointermove 在现代浏览器中支持触摸。OrbitControls 也支持触摸。

WebGL 不支持：可以加 try/catch 显示错误信息。

OK 让我最终写代码。

补充：lens 的形状，我用 SphereGeometry 球冠表示碗状隐形眼镜。让我用 thetaStart 让球冠开口朝上（凹面朝上）。

SphereGeometry(radius, widthSeg, heightSeg, phiStart=0, phiLength=2π, thetaStart=0, thetaLength=π)

theta 是从 +y 轴向下的角度。theta=0 是 +y 方向（顶部），theta=π 是 -y 方向（底部）。

我要球冠开口朝上（凹面朝上）。即球冠的"碗"在上方，开口朝上。

这意味着球冠是球的下半部分的一小段。thetaStart = π - θ_len，thetaLength = θ_len。

例如 θ_len = π/8（22.5°）：thetaStart = π - π/8 = 7π/8，thetaLength = π/8。

这样球冠在球的下方，开口朝上。但是球冠的"碗底"在 -y 方向（下方），"碗口"在 +y 方向（上方）。

Wait 这样球冠的凹面是朝上的（从 +y 看下去是凹的）。但球冠的"碗"形状...让我画一下。

球心在原点。球冠从 thetaStart=7π/8 到 thetaStart + thetaLength = π（即从 +y 轴向下 157.5° 到 180°）。这部分球面在球的下方（y < 0 部分），靠近 -y 轴。

球冠的形状：开口在上方（+y 方向看下去看到开口），碗底在下方（-y 方向）。

这就是碗的形状（凹面朝上）。✓

但是这样的球冠半径是 0.45，所以球冠的开口直径约 0.45 * sin(π/8) * 2 ≈ 0.45 * 0.383 * 2 ≈ 0.345。
碗的深度约 0.45 * (1 - cos(π/8)) ≈ 0.45 * 0.076 ≈ 0.034。

太浅了。让我用 thetaLength 大一点：π/4 (45°)。thetaStart = 3π/4。
开口直径 0.45 * sin(π/4) * 2 ≈ 0.45 * 0.707 * 2 ≈ 0.636。
深度 0.45 * (1 - cos(π/4)) ≈ 0.45 * 0.293 ≈ 0.132。

OK 这样开口 0.64，深度 0.13。镜片厚度合理。

```javascript
const lensGeo = new THREE.SphereGeometry(0.45, 32, 16, 0, Math.PI * 2, Math.PI * 0.75, Math.PI * 0.25);
```

这样镜片是球冠，开口朝上。在凹槽内放置（凹槽中心 y=0.36，开口半径 0.6）。

镜片位置：让镜片底部贴近凹槽底部，开口在液体表面下方一点。

凹槽底部 y=0.11，凹槽顶部 y=0.61。液体表面 y=0.5。

镜片球冠的最低点（碗底）在球心 -y 方向 0.45 处。如果球心在 y=0.5（液体表面），碗底在 y=0.05（接近凹槽底部），碗口在 y=0.5 + 0.45 * cos(π/4) = 0.5 + 0.318 = 0.818（远高于凹槽顶部）。

这不对。让我重新计算。

球心位置：如果球心在 y=yc，球冠从 thetaStart=3π/4 到 thetaStart+thetaLength=π（即 135° 到 180°）。
- 碗底（theta=π）位置：(0, yc - radius, 0) = (0, yc - 0.45, 0)
- 碗口边缘（theta=3π/4）位置：(0, yc - radius*cos(3π/4), radius*sin(3π/4)) = (0, yc + 0.318, 0.318)

如果碗口在液体表面 y=0.5，那 yc = 0.5 - 0.318 = 0.182。碗底在 y = 0.182 - 0.45 = -0.268。但凹槽底部 y=0.11，碗底低于凹槽底部了。

不对。我需要让镜片放在凹槽内部，不超出凹槽。

让我用更小的镜片。镜片半径 0.3，thetaLength = π/4：
- 碗口半径 = 0.3 * sin(π/4) = 0.212
- 碗深 = 0.3 * (1 - cos(π/4)) = 0.088

球心位置：碗口在 y=0.5（液体表面），yc = 0.5 - 0.3 * cos(π/4) = 0.5 - 0.212 = 0.288。碗底 y = 0.288 - 0.3 = -0.012。还是低于凹槽底部。

镜片太大或凹槽太浅。让我用更浅的镜片：thetaLength = π/8 (22.5°)：
- 碗口半径 = 0.3 * sin(π/8) = 0.115
- 碗深 = 0.3 * (1 - cos(π/8)) = 0.023

太小了，看不清。

实际上，凹槽深度只有 0.5（从 y=0.11 到 y=0.61），镜片放在凹槽里很难看到。如果凹槽浅，镜片就是漂浮在液体表面，从上面看是个透明圆盘。

让我简化：镜片用扁平的 CircleGeometry，放在液体表面附近：

```javascript
const lensGeo = new THREE.CircleGeometry(0.4, 64);
const lensMat = new THREE.MeshStandardMaterial({
  color: 0xa0e8ff,
  metalness: 0.2,
  roughness: 0.1,
  transparent: true,
  opacity: 0.5,
  side: THREE.DoubleSide,
});

const leftLens = new THREE.Mesh(lensGeo, lensMat);
leftLens.rotation.x = -Math.PI / 2;
leftLens.position.set(-wellX, 0.49, 0);
caseGroup.add(leftLens);
```

这样镜片是扁平透明圆盘，水平放置在液体表面附近。简单且能看到。

或者用 RingGeometry 表示镜片边缘：

```javascript
const lensGeo = new THREE.RingGeometry(0.3, 0.4, 64);
```

这样是个圆环，边缘有厚度感。

我用扁平圆盘，加透明材质。OK。

让我最终确定代码。

我还需要给液体加波动效果。但 CircleGeometry 的顶点不能直接变形，除非用 shader 或者更新顶点。

简化：让液体表面上下浮动（整体 y 位置变化），不做表面波纹。这样简单。

实际上为了视觉效果，可以用 shader 做波纹。但代码量大。我用简单方案：

```javascript
// 在 animate 中
leftLiquid.position.y = 0.5 + Math.sin(time * 2) * 0.005;
rightLiquid.position.y = 0.5 + Math.cos(time * 2.2) * 0.005;
```

让液体表面上下浮动一点，模拟水面波动。

OK 整理代码：

```javascript
// === 完整 JS 代码 ===

import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

let scene, camera, renderer, controls;
let caseGroup, lidPivot, lidMesh;
let leftLiquid, rightLiquid, innerLight;
let particleSystem;

let isOpen = false;
let lidAngle = 0;
let targetLidAngle = 0;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let mouseDownPos = { x: 0, y: 0 };
let mouseDownTime = 0;
let hoveredLid = false;

const clock = new THREE.Clock();

const COLORS = {
  caseBody: 0x0d3a36,
  caseBodyAccent: 0x1a5651,
  lidColor: 0x164440,
  letterGold: 0xf5a623,
  letterGoldBright: 0xffd966,
  liquid: 0x4ecdc4,
  innerWell: 0x07201d,
  lensTint: 0xa0e8ff,
};

function init() {
  scene = new THREE.Scene();
  
  camera = new THREE.PerspectiveCamera(38, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(5, 3.2, 6.5);
  
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
  
  setupLights();
  createFloor();
  createCase();
  createParticles();
  
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.06;
  controls.minDistance = 4;
  controls.maxDistance = 12;
  controls.maxPolarAngle = Math.PI * 0.5;
  controls.minPolarAngle = Math.PI * 0.15;
  controls.target.set(0, 0.2, 0);
  controls.enablePan = false;
  
  window.addEventListener('resize', onResize);
  renderer.domElement.addEventListener('pointerdown', onPointerDown);
  renderer.domElement.addEventListener('pointerup', onPointerUp);
  renderer.domElement.addEventListener('pointermove', onPointerMove);
  document.getElementById('toggle-btn').addEventListener('click', () => {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  });
  
  setTimeout(() => {
    document.getElementById('loader').classList.add('hidden');
  }, 500);
  
  animate();
}

// ... 其他函数 ...
```

让我写完最终代码。我会在最终输出中包含完整代码。

我还想加一个细节：当盒盖关闭时，盒盖底面应该和盒身顶面接触。但是盒盖顶面的字母在关闭时凸出于盒盖表面，可能和盒身顶面的字母（凹槽旁）重叠。

实际上字母在 lidMesh 局部 (0, 0.21, 0) 处（lidHeight/2 + 0.005），即 lidMesh 顶面之上 0.005。这是 lidMesh 局部坐标。

lidMesh 关闭时在 caseGroup 局部 (0, 0.8, 0)，所以盒盖顶面字母在 caseGroup (x, 0.8 + 0.005, z) = (x, 0.805, z)。

盒身顶面字母在 caseGroup (x, 0.61, z)。

两者高度不同（0.805 vs 0.61），不重叠。但是盒盖底面在 y=0.6，盒身顶面也在 y=0.6。两者接触。

盒身顶面字母在 y=0.61，盒盖底面在 y=0.6。所以盒身字母在盒盖底面之上 0.01。盒盖关闭时盒身字母会被盒盖压住（盒盖底面挡住盒身字母）。

实际上盒身字母凸出于盒身顶面 0.01，盒盖底面也在 y=0.6。盒身字母顶部 y=0.61，盒盖底面 y=0.6。所以盒身字母会"穿过"盒盖底面 0.01。

这会导致 z-fighting 或视觉穿模。

解决：让盒身字母小一些，放在凹槽前方，凹槽前方没有盒盖底面遮挡（因为盒盖在盒身上方）。

Wait 盒盖关闭时盖在盒身顶部，盒盖底面贴着盒身顶面。盒身顶面字母会被盒盖盖住。这是合理的（关闭时字母藏在盒内）。

但字母凸出盒身顶面 0.01，会穿过盒盖底面。这不是大问题，因为字母很小，且盒盖底面是平的，字母穿过盒盖底面一点不会明显。

或者让盒身字母不凸出（用 PlaneGeometry 贴在盒身顶面，与顶面齐平）：

```javascript
letterLBody.position.set(-wellX, 0.601, 0.95);
```

放在 y=0.601，离盒身顶面 0.001。这样字母几乎贴在表面，不会穿过盒盖底面。

但 z-fighting 可能发生（字母和盒身顶面太近）。让我用 0.005：

```javascript
letterLBody.position.set(-wellX, 0.605, 0.95);
```

字母凸出 0.005，盒盖底面 y=0.6。字母顶部 y=0.605 + 0.225（字母高度 0.45 / 2）= 0.83？

Wait PlaneGeometry 是平面，没有厚度。字母在 plane 上画。Plane 的 y 坐标是 plane 的中心。所以 plane 在 y=0.605，字母在 plane 上，没有"高度"。

但是字母 plane 是垂直放置还是水平放置？我用 rotation.x = -π/2 让 plane 平铺（水平），所以 plane 中心 y=0.605，plane 厚度 0（平面）。字母在 plane 上，看起来字母"高度"为 0（贴在表面）。

所以字母 plane 在 y=0.605，盒盖底面在 y=0.6。字母 plane 高于盒盖底面 0.005。盒盖关闭时字母 plane 不会被盒盖底面遮挡（因为字母 plane 在盒盖底面之上）。

但是盒盖底面在 y=0.6，盒身顶面也在 y=0.6。盒盖关闭时盒盖底面接触盒身顶面。字母 plane 在 y=0.605（盒身顶面之上 0.005），高于盒盖底面 0.005。

所以盒盖关闭时，字母 plane 在盒盖底面之上 0.005，被盒盖底面"挡住"（从上方看不到）。

Wait 字母 plane 在盒盖底面之上，意味着字母 plane 比盒盖底面更高（y 更大）。盒盖底面 y=0.6，字母 plane y=0.605。所以字母 plane 在盒盖底面之上 0.005。

但盒盖关闭时盒盖底面贴在盒身顶面上。盒盖在盒身上方，盒盖底面 y=0.6，盒盖顶面 y=1.0。字母 plane y=0.605 在盒盖底面之下 0.395（在盒盖内部）。

不对。盒盖在 y=0.6 到 y=1.0 之间。字母 plane y=0.605 在盒盖内部（被盒盖包围）。从外面看不到字母 plane（盒盖遮挡）。

所以盒盖关闭时，盒身顶面的字母被盒盖遮挡，从外面看不到。✓

打开盒盖后，字母 plane 暴露在外面，可以看到。✓

这正是我想要的行为。

OK 没问题。

让我整理最终代码。代码量很大，但应该没有错误。

Wait 还有一个问题：lidMesh 旋转后，lidMesh 子对象（字母 plane）会跟着旋转。这意味着字母 plane 的位置会改变。但子对象的本地坐标不变。

字母 plane 在 lidMesh 局部 (-1.15, 0.21, 0) 和 (1.15, 0.21, 0)。lidMesh 旋转后，字母 plane 在世界坐标也旋转。

但是字母 plane 的 raycast 检测：如果用 recursive=true，会检测 lidMesh 所有子对象。如果字母 plane 也被点击，会触发 toggleLid。这是合理的。

OK 没问题。

让我最终写代码。

为了减少代码长度，我可以省略一些注释，保留关键代码。

让我做最后检查：
- 没有使用 `let`/`const` 在声明前调用 ✓
- 没有使用 Inter 字体作为主字体 ✓ (用 Space Grotesk)
- 没有蓝紫色 ✓ (用墨绿+金色+青色)
- 几何体参数有下限保护 ✓
- 颜色格式统一 ✓
- ShaderMaterial 没用（用 MeshPhysicalMaterial）✓
- importmap 用 bare imports ✓

让我最终输出代码。

我突然意识到一个问题：PMREMGenerator + RoomEnvironment 在某些版本可能有问题。让我检查一下。

Three.js 0.160 中：
- `three/addons/environments/RoomEnvironment.js` 存在
- RoomEnvironment 是一个 Scene 子类
- `pmremGenerator.fromScene(new RoomEnvironment(), 0.04)` 应该工作

实际上 RoomEnvironment 在 0.160 中的构造函数不接受参数。让我检查。

`new RoomEnvironment(renderer)` 在某些版本需要 renderer 参数。

让我看 0.160 版本：RoomEnvironment 的构造函数没有参数。但有些版本需要 renderer。为了安全，我直接用 PMREMGenerator 从一个简单的 scene 生成环境贴图。

或者直接用 RoomEnvironment 不传参数：

```javascript
const pmremGenerator = new THREE.PMREMGenerator(renderer);
scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
```

如果在某些版本报错，可以加 try/catch：

```javascript
try {
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
} catch (e) {
  console.warn('环境贴图生成失败', e);
}
```

OK 让我加上 try/catch。

最终代码完成。让我整理输出。

实际上，我突然想到用 SphereGeometry 创建球冠的方法。让我看看是否真的能表示隐形眼镜的形状。

`SphereGeometry(radius, widthSegments, heightSegments, phiStart, phiLength, thetaStart, thetaLength)`

theta 从 +y 轴向下测量。theta=0 是 +y 方向。theta=π 是 -y 方向。

如果我用 thetaStart = 0, thetaLength = π/6（30度），创建的是球冠的"顶部"（北极冠），形状是凸面朝上的小帽子。

如果我用 thetaStart = π - π/6, thetaLength = π/6，创建的是球冠的"底部"（南极冠），形状是凹面朝上的小碗。

我想要凹面朝上的碗状（眼镜形状）。所以用 thetaStart = π - π/6 = 5π/6, thetaLength = π/6。

这样球冠开口朝上，凹面朝上，像一个浅碗。✓

但是这种球冠半径 = 给定的 radius，开口直径 = 2 * radius * sin(thetaLength/2) ≈ 2 * radius * sin(π/12) ≈ 2 * radius * 0.259 ≈ 0.518 * radius。

如果 radius = 0.5，开口直径 0.26。镜片直径太小。

如果 thetaLength = π/4（45度），开口直径 = 2 * 0.5 * sin(π/8) = 2 * 0.5 * 0.383 = 0.383。

镜片直径 0.383，凹槽直径 1.4（半径 0.7）。镜片在凹槽内合理。

让我用：
```javascript
const lensGeo = new THREE.SphereGeometry(0.5, 32, 16, 0, Math.PI * 2, Math.PI * 0.75, Math.PI * 0.25);
```

thetaStart = 3π/4 (135°), thetaLength = π/4 (45°)。所以球冠从 135° 到 180°（球的南极区域）。

球冠中心在 -y 方向，球心在原点。球冠顶点（碗底）在 (0, -0.5, 0)（球的南极）。球冠边缘（碗口）在 theta=3π/4 处，y = 0.5 * cos(3π/4) = -0.354, 半径方向距离 0.5 * sin(3π/4) = 0.354。

所以碗口在 y=-0.354 平面，距 -y 轴 0.354。碗底在 y=-0.5。

碗深 = 0.5 - 0.354 = 0.146。
碗口直径 = 0.708。

OK 镜片直径 0.7，深度 0.15。合理。

镜片位置：让碗口接近液体表面 y=0.5。碗口在 y=-0.354（局部坐标）。如果镜片球心在 y=yc，碗口在 y=yc-0.354。要让碗口在 y=0.5（液体表面），yc=0.854。碗底在 yc-0.5=0.354。

但凹槽底部 y=0.11，碗底 y=0.354 > 0.11，所以碗底在凹槽底部之上 0.244。✓ 凹槽深度 0.5，碗底在凹槽深度 0.244 之上，合理。

但是碗口在 y=0.5（液体表面），液体表面 y=0.5。所以碗口正好在液体表面。镜片半浸没在液体中。✓

让我用这个参数：
```javascript
const lensGeo = new THREE.SphereGeometry(0.5, 32, 16, 0, Math.PI * 2, Math.PI * 0.75, Math.PI * 0.25);
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.85,
  thickness: 0.1,
  ior: 1.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
});

const leftLens = new THREE.Mesh(lensGeo, lensMat);
leftLens.position.set(-wellX, 0.854, 0);
caseGroup.add(leftLens);
```

Wait 球心位置 yc=0.854，但凹槽顶部 y=0.61。球心在凹槽之外！这不对。

让我重新考虑。我希望镜片在凹槽内。凹槽范围 y=0.11 到 y=0.61。镜片最大直径 0.7（凹槽直径 1.4）。

镜片球心如果在凹槽中心 y=0.36，碗底在 y=0.36 - 0.5 = -0.14（凹槽底部之下），碗口在 y=0.36 - 0.354 = 0.006（凹槽底部之下）。

不对，球心在凹槽中心时碗口在 y=0.006，远低于凹槽顶部 y=0.61。

让我用更小的 radius 让镜片适合凹槽。Radius=0.3：
- 碗底 y_offset = -0.3
- 碗口 y_offset = -0.3 * cos(3π/4) = -(-0.212) = 0.212
- 碗深 = 0.3 - 0.212 = 0.088
- 碗口直径 = 0.3 * sin(3π/4) * 2 = 0.424

球心在 y=0.5（液体表面）：碗底 y=0.2，碗口 y=0.5 + 0.212 = 0.712（高于凹槽顶部 0.61）。

不行。让我换思路。

我让镜片"碗口"朝上，球心在镜片下方。即镜片是球的下半部分的一小段。球心在镜片"下方"，碗口在镜片"上方"。

球心在 y=yc，碗底在 y=yc - radius，碗口在 y=yc - radius * cos(thetaStart)。

如果 thetaStart = 3π/4 (135°)，cos(3π/4) = -√2/2 ≈ -0.707。所以 y=yc - radius * (-0.707) = yc + 0.707*radius。碗口高于球心。

Wait 让我重新想。SphereGeometry 的 theta 从 +y 轴向下。

thetaStart = 3π/4 意味着从 +y 向下 135°，即接近 -y 方向但偏一点。

thetaLength = π/4，即 45° 的范围。

所以球冠从 theta=3π/4 到 theta=π（即从接近 -y 方向到 -y 方向）。

球冠的"中心点"（碗底）在 theta=π，即 -y 方向。
球冠的"边缘"（碗口）在 theta=3π/4，即 -y 偏 +y 一点。

如果球心在原点：
- 碗底位置：(0, -radius, 0)
- 碗口位置（在 theta=3π/4 处）：(0, radius * cos(3π/4), radius * sin(3π/4)) = (0, -0.707*radius, 0.707*radius)

Wait cos(3π/4) = -√2/2 ≈ -0.707。所以碗口在 y = -0.707*radius 处。比碗底（-radius）高。

碗底 y = -radius，碗口 y = -0.707*radius。碗口高于碗底（碗口在碗底之上）。

碗深 = -0.707*radius - (-radius) = radius - 0.707*radius = 0.293*radius。
碗口直径 = 2 * radius * sin(3π/4) = 2 * 0.707 * radius = 1.414*radius。

如果 radius = 0.4：
- 碗底 y_offset = -0.4
- 碗口 y_offset = -0.283
- 碗深 = 0.117
- 碗口直径 = 0.566

如果球心在 y=0.6（凹槽顶部）：碗底 y=0.2，碗口 y=0.317。碗在凹槽内（凹槽范围 0.11 到 0.61）。但碗口在 y=0.317，远低于凹槽顶部 0.61。镜片整体在凹槽内 ✓。

如果球心在 y=0.5（液体表面）：碗底 y=0.1，碗口 y=0.217。碗在凹槽内（凹槽底部 0.11，碗底 0.1 略低于凹槽底部，可以接受误差）。但是碗口在 y=0.217，液体表面 y=0.5。镜片完全浸没在液体下面。

我希望镜片"漂浮"在液体表面，即碗口接近液体表面。

如果球心在 y=0.78（高于液体表面 0.5）：碗底 y=0.38，碗口 y=0.497。碗口接近液体表面 y=0.5 ✓。

但是球心 y=0.78 在凹槽外（凹槽顶部 0.61）。这意味着球心不在凹槽内。但球冠在凹槽内（碗口在 y=0.497 < 凹槽顶部 0.61）。

实际上球心位置不重要，重要的是球冠在凹槽内。让我用球心 y=0.78：

```javascript
leftLens.position.set(-wellX, 0.78, 0);
```

碗底 y=0.78 - 0.4 = 0.38，碗口 y=0.78 - 0.283 = 0.497。

镜片在凹槽内（y 范围 0.38 到 0.497，凹槽范围 0.11 到 0.61）✓。
碗口接近液体表面 y=0.5 ✓。

这样镜片半漂浮在液体中，碗口接近液面，看起来真实。

但是用 radius=0.4 时碗口直径 0.566，凹槽直径 1.4。镜片只占凹槽 40% 的空间。可以稍微大些。radius=0.5：
- 碗口直径 0.707
- 碗深 0.146
- 球心 y=0.783，碗底 y=0.283，碗口 y=0.5（接近液面）

让我用 radius=0.5：
```javascript
const lensGeo = new THREE.SphereGeometry(0.5, 32, 16, 0, Math.PI * 2, Math.PI * 0.75, Math.PI * 0.25);
// ...
leftLens.position.set(-wellX, 0.783, 0);
```

碗口在 y=0.5（液面），碗底在 y=0.283（凹槽内）。镜片漂浮在液体中，碗口接近液面。

镜片直径 0.707，凹槽直径 1.4，镜片占凹槽 50% 空间。

镜片材质用 transmission，看起来透明。

OK 这个方案不错。让我用这个。

不过 transmission 在 Three.js 中需要 WebGLRenderer 支持，可能会影响性能。让我用 MeshStandardMaterial + transparent + opacity：

```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0.1,
  roughness: 0.05,
  transparent: true,
  opacity: 0.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
});
```

这样性能好，镜片透明。但不像 transmission 那样有真实的折射效果。

或者用 transmission 但是薄一点：

```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.8,
  thickness: 0.05,
  ior: 1.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
  transparent: true,
  opacity: 0.6,
});
```

Three.js 0.160 中 transmission 应该可以工作。

我用 transmission 试试。如果性能不好或者不显示，可以切换。

OK 最终代码。

让我整合所有内容：

实际上让我用一个更简单的方法做液体和镜片：直接用扁平的圆盘。

```javascript
// 液体表面（扁平圆盘）
const liquidGeo = new THREE.CircleGeometry(wellRadius - 0.13, 64);
const liquidMat = new THREE.MeshStandardMaterial({
  color: COLORS.liquid,
  metalness: 0.4,
  roughness: 0.05,
  transparent: true,
  opacity: 0.75,
  side: THREE.DoubleSide,
  emissive: COLORS.liquid,
  emissiveIntensity: 0.2,
});

// 镜片（扁平圆盘）
const lensGeo = new THREE.CircleGeometry(0.35, 64);
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0,
  roughness: 0.05,
  transmission: 0.8,
  thickness: 0.05,
  ior: 1.4,
  side: THREE.DoubleSide,
});
```

液体和镜片都是扁平圆盘，水平放置。简单且能看清。

镜片放在液体表面附近：
```javascript
const leftLens = new THREE.Mesh(lensGeo, lensMat);
leftLens.rotation.x = -Math.PI / 2;
leftLens.position.set(-wellX, 0.48, 0);  // 略低于液面 0.5
caseGroup.add(leftLens);
```

OK 我用这个简单方案。

让我最终写代码。

OK 让我把完整代码整理出来：

完整 HTML：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LUMINA · 隐形眼镜盒 3D 模型</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;700&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500&display=swap" rel="stylesheet">

<style>
  /* CSS */
</style>
</head>
<body>
  /* HTML */
  
  <script type="importmap">...</script>
  <script type="module">
    /* JS */
  </script>
</body>
</html>
```

我会把 CSS、HTML、JS 全部填充完整。然后给出运行说明和测试说明。

让我整理完整代码。

CSS:
```css
* { margin: 0; padding: 0; box-sizing: border-box; }

:root {
  --bg-dark: #050a0c;
  --bg-mid: #0a1f1c;
  --bg-light: #143633;
  --accent: #f5a623;
  --accent-bright: #ffd966;
  --accent-2: #4ecdc4;
  --text: #e8f4f0;
  --text-dim: #6a8c87;
  --text-mute: #4a6c67;
  --border: rgba(245, 166, 35, 0.3);
}

body {
  background: radial-gradient(ellipse at 50% 30%, var(--bg-light) 0%, var(--bg-mid) 40%, var(--bg-dark) 80%);
  color: var(--text);
  font-family: 'Space Grotesk', system-ui, sans-serif;
  overflow: hidden;
  height: 100vh;
  position: relative;
}

body::after {
  content: '';
  position: fixed;
  inset: 0;
  background: radial-gradient(circle at 30% 70%, rgba(78, 205, 196, 0.08) 0%, transparent 50%),
              radial-gradient(circle at 70% 30%, rgba(245, 166, 35, 0.06) 0%, transparent 50%);
  pointer-events: none;
  z-index: 1;
}

#canvas-container {
  position: fixed;
  inset: 0;
  z-index: 2;
}

.header {
  position: fixed;
  top: 0; left: 0; right: 0;
  padding: 32px 48px;
  z-index: 10;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  pointer-events: none;
}

.brand {
  font-family: 'Cormorant Garamond', serif;
  font-weight: 500;
  line-height: 1;
}

.brand .name {
  font-size: 28px;
  letter-spacing: 8px;
  color: var(--accent);
  text-transform: uppercase;
  display: block;
}

.brand .tag {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  color: var(--text-mute);
  letter-spacing: 5px;
  margin-top: 8px;
  display: block;
}

.status {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: var(--text-dim);
  text-align: right;
  line-height: 1.8;
}

.status .row {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  align-items: center;
}

.status .label {
  color: var(--text-mute);
  font-size: 9px;
  letter-spacing: 2px;
}

.status .value {
  color: var(--accent-2);
  font-weight: 500;
  letter-spacing: 1px;
}

.status .value.state {
  color: var(--accent);
}

.status .indicator {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--accent);
  box-shadow: 0 0 10px var(--accent);
  animation: pulse 1.8s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.6; transform: scale(1); }
  50% { opacity: 1; transform: scale(1.3); }
}

.footer {
  position: fixed;
  bottom: 0; left: 0; right: 0;
  padding: 32px 48px;
  z-index: 10;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  pointer-events: none;
}

.controls-hint {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: var(--text-dim);
  line-height: 2;
}

.controls-hint .row {
  display: flex;
  align-items: center;
  gap: 14px;
}

.controls-hint .key {
  display: inline-block;
  min-width: 60px;
  padding: 3px 8px;
  border: 1px solid var(--border);
  color: var(--accent);
  font-size: 9px;
  letter-spacing: 2px;
  text-align: center;
  background: rgba(245, 166, 35, 0.05);
}

.action-btn {
  pointer-events: auto;
  background: linear-gradient(135deg, rgba(245, 166, 35, 0.15), rgba(245, 166, 35, 0.05));
  border: 1px solid var(--border);
  color: var(--accent);
  padding: 16px 36px;
  font-family: 'Space Grotesk', sans-serif;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 4px;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(12px);
  position: relative;
  overflow: hidden;
}

.action-btn::before {
  content: '';
  position: absolute;
  top: 0; left: -100%;
  width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 217, 102, 0.3), transparent);
  transition: left 0.6s;
}

.action-btn:hover::before { left: 100%; }

.action-btn:hover {
  border-color: var(--accent);
  color: var(--accent-bright);
  box-shadow: 0 0 40px rgba(245, 166, 35, 0.25), inset 0 0 20px rgba(245, 166, 35, 0.05);
  transform: translateY(-2px);
}

.action-btn:active { transform: translateY(0); }

.corner-mark {
  position: fixed;
  width: 28px;
  height: 28px;
  border: 1px solid var(--border);
  z-index: 5;
  pointer-events: none;
}

.corner-mark.tl { top: 20px; left: 20px; border-right: none; border-bottom: none; }
.corner-mark.tr { top: 20px; right: 20px; border-left: none; border-bottom: none; }
.corner-mark.bl { bottom: 20px; left: 20px; border-right: none; border-top: none; }
.corner-mark.br { bottom: 20px; right: 20px; border-left: none; border-top: none; }

.side-info {
  position: fixed;
  left: 32px;
  top: 50%;
  transform: translateY(-50%) rotate(-90deg);
  transform-origin: left center;
  font-family: 'JetBrains Mono', monospace;
  font-size: 9px;
  letter-spacing: 8px;
  color: var(--text-mute);
  z-index: 5;
  pointer-events: none;
  white-space: nowrap;
}

.side-info-r {
  position: fixed;
  right: 32px;
  top: 50%;
  transform: translateY(-50%) rotate(90deg);
  transform-origin: right center;
  font-family: 'JetBrains Mono', monospace;
  font-size: 9px;
  letter-spacing: 8px;
  color: var(--text-mute);
  z-index: 5;
  pointer-events: none;
  white-space: nowrap;
}

.toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, calc(-50% + 200px));
  background: rgba(10, 31, 28, 0.92);
  border: 1px solid var(--accent);
  color: var(--text);
  padding: 14px 28px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  letter-spacing: 3px;
  text-transform: uppercase;
  z-index: 100;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.4s, transform 0.4s;
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 30px rgba(245, 166, 35, 0.2);
}

.toast.show {
  opacity: 1;
  transform: translate(-50%, calc(-50% + 220px));
}

.loader {
  position: fixed;
  inset: 0;
  background: var(--bg-dark);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.8s;
}

.loader.hidden {
  opacity: 0;
  pointer-events: none;
}

.loader-text {
  font-family: 'Cormorant Garamond', serif;
  color: var(--accent);
  font-size: 14px;
  letter-spacing: 6px;
  text-transform: uppercase;
}

@media (max-width: 768px) {
  .header, .footer { padding: 20px; }
  .brand .name { font-size: 22px; letter-spacing: 5px; }
  .controls-hint { font-size: 10px; }
  .action-btn { padding: 12px 24px; font-size: 11px; letter-spacing: 2px; }
  .side-info, .side-info-r { display: none; }
  .corner-mark { width: 20px; height: 20px; }
}
```

HTML body:
```html
<div class="loader" id="loader">
  <div class="loader-text">LUMINA · LOADING</div>
</div>

<div id="canvas-container"></div>

<div class="corner-mark tl"></div>
<div class="corner-mark tr"></div>
<div class="corner-mark bl"></div>
<div class="corner-mark br"></div>

<div class="side-info">LUMINA · OPTICAL · CASE · SYSTEM</div>
<div class="side-info-r">PRECISION · CRAFTED · 2024</div>

<header class="header">
  <div class="brand">
    <span class="name">Lumina</span>
    <span class="tag">OPTICAL · LENS CASE</span>
  </div>
  <div class="status">
    <div class="row">
      <span class="label">MODEL</span>
      <span class="value">L-7 · GEMINI</span>
    </div>
    <div class="row">
      <span class="label">MATERIAL</span>
      <span class="value">POLYCARBONATE</span>
    </div>
    <div class="row">
      <span class="label">STATE</span>
      <span class="value state" id="state-text">CLOSED</span>
      <span class="indicator" id="state-indicator"></span>
    </div>
  </div>
</header>

<footer class="footer">
  <div class="controls-hint">
    <div class="row">
      <span class="key">DRAG</span>
      <span>旋转视角</span>
    </div>
    <div class="row">
      <span class="key">SCROLL</span>
      <span>缩放距离</span>
    </div>
    <div class="row">
      <span class="key">CLICK</span>
      <span>点击盒盖开合</span>
    </div>
  </div>
  <button class="action-btn" id="toggle-btn">开启盒盖</button>
</footer>

<div class="toast" id="toast"></div>
```

JS:
```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

let scene, camera, renderer, controls;
let caseGroup, lidPivot, lidMesh;
let leftLiquid, rightLiquid, innerLight;
let particleSystem;

let isOpen = false;
let lidAngle = 0;
let targetLidAngle = 0;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let mouseDownPos = { x: 0, y: 0 };
let mouseDownTime = 0;
let hoveredLid = false;

const clock = new THREE.Clock();

const COLORS = {
  caseBody: 0x0d3a36,
  lidColor: 0x164440,
  letterGold: 0xf5a623,
  letterGoldBright: 0xffd966,
  liquid: 0x4ecdc4,
  innerWell: 0x07201d,
  lensTint: 0xa0e8ff,
};

const BODY_W = 4.4;
const BODY_H = 1.2;
const BODY_D = 2.4;
const WELL_R = 0.7;
const WELL_X = 1.15;

function init() {
  scene = new THREE.Scene();
  
  camera = new THREE.PerspectiveCamera(38, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(5, 3.2, 6.5);
  
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  try {
    const pmremGenerator = new THREE.PMREMGenerator(renderer);
    scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
  } catch (e) {
    console.warn('环境贴图生成失败', e);
  }
  
  setupLights();
  createFloor();
  createCase();
  createParticles();
  
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.06;
  controls.minDistance = 4;
  controls.maxDistance = 12;
  controls.maxPolarAngle = Math.PI * 0.5;
  controls.minPolarAngle = Math.PI * 0.15;
  controls.target.set(0, 0.2, 0);
  controls.enablePan = false;
  
  window.addEventListener('resize', onResize);
  renderer.domElement.addEventListener('pointerdown', onPointerDown);
  renderer.domElement.addEventListener('pointerup', onPointerUp);
  renderer.domElement.addEventListener('pointermove', onPointerMove);
  document.getElementById('toggle-btn').addEventListener('click', () => {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  });
  
  setTimeout(() => {
    document.getElementById('loader').classList.add('hidden');
  }, 600);
  
  animate();
}

function setupLights() {
  const ambient = new THREE.AmbientLight(0xffffff, 0.2);
  scene.add(ambient);
  
  const keyLight = new THREE.DirectionalLight(0xfff5e0, 1.6);
  keyLight.position.set(4, 8, 5);
  keyLight.castShadow = true;
  keyLight.shadow.mapSize.set(2048, 2048);
  keyLight.shadow.camera.left = -4;
  keyLight.shadow.camera.right = 4;
  keyLight.shadow.camera.top = 4;
  keyLight.shadow.camera.bottom = -4;
  keyLight.shadow.camera.near = 0.5;
  keyLight.shadow.camera.far = 20;
  keyLight.shadow.bias = -0.0005;
  keyLight.shadow.radius = 4;
  scene.add(keyLight);
  
  const fillLight = new THREE.PointLight(0x4ecdc4, 1.2, 15);
  fillLight.position.set(-4, 3, 2);
  scene.add(fillLight);
  
  const rimLight = new THREE.PointLight(0xff8c42, 0.9, 12);
  rimLight.position.set(3, 2, -4);
  scene.add(rimLight);
  
  const spotLight = new THREE.SpotLight(0xffffff, 1.2, 15, Math.PI * 0.18, 0.5, 1.5);
  spotLight.position.set(0, 8, 0);
  spotLight.target.position.set(0, 0, 0);
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.set(1024, 1024);
  spotLight.shadow.bias = -0.0005;
  scene.add(spotLight);
  scene.add(spotLight.target);
  
  innerLight = new THREE.PointLight(0x4ecdc4, 0, 3);
  innerLight.position.set(0, 0.4, 0);
  scene.add(innerLight);
}

function createFloor() {
  const shadowFloorGeo = new THREE.PlaneGeometry(40, 40);
  const shadowFloorMat = new THREE.ShadowMaterial({ opacity: 0.45 });
  const shadowFloor = new THREE.Mesh(shadowFloorGeo, shadowFloorMat);
  shadowFloor.rotation.x = -Math.PI / 2;
  shadowFloor.position.y = -0.65;
  shadowFloor.receiveShadow = true;
  scene.add(shadowFloor);
  
  const glowFloorGeo = new THREE.CircleGeometry(3.5, 64);
  const glowFloorMat = new THREE.MeshBasicMaterial({
    color: 0x4ecdc4,
    transparent: true,
    opacity: 0.06,
    side: THREE.DoubleSide,
  });
  const glowFloor = new THREE.Mesh(glowFloorGeo, glowFloorMat);
  glowFloor.rotation.x = -Math.PI / 2;
  glowFloor.position.y = -0.64;
  scene.add(glowFloor);
  
  const innerRingGeo = new THREE.RingGeometry(2.8, 2.85, 64);
  const innerRingMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.4,
    side: THREE.DoubleSide,
  });
  const innerRing = new THREE.Mesh(innerRingGeo, innerRingMat);
  innerRing.rotation.x = -Math.PI / 2;
  innerRing.position.y = -0.63;
  scene.add(innerRing);
  
  const outerRingGeo = new THREE.RingGeometry(3.4, 3.45, 64);
  const outerRingMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.2,
    side: THREE.DoubleSide,
  });
  const outerRing = new THREE.Mesh(outerRingGeo, outerRingMat);
  outerRing.rotation.x = -Math.PI / 2;
  outerRing.position.y = -0.63;
  scene.add(outerRing);
  
  // 刻度
  const tickMat = new THREE.MeshBasicMaterial({
    color: 0xf5a623,
    transparent: true,
    opacity: 0.5,
    side: THREE.DoubleSide,
  });
  
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2;
    const tickGeo = new THREE.PlaneGeometry(0.15, 0.02);
    const tick = new THREE.Mesh(tickGeo, tickMat);
    tick.position.x = Math.cos(angle) * 3.1;
    tick.position.z = Math.sin(angle) * 3.1;
    tick.position.y = -0.63;
    tick.rotation.x = -Math.PI / 2;
    tick.rotation.z = -angle;
    scene.add(tick);
  }
}

function createCase() {
  caseGroup = new THREE.Group();
  
  // === 盒身 ===
  const bodyGeo = new RoundedBoxGeometry(BODY_W, BODY_H, BODY_D, 8, 0.18);
  const bodyMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.caseBody,
    metalness: 0.5,
    roughness: 0.3,
    clearcoat: 0.7,
    clearcoatRoughness: 0.2,
  });
  const body = new THREE.Mesh(bodyGeo, bodyMat);
  body.castShadow = true;
  body.receiveShadow = true;
  caseGroup.add(body);
  
  // 盒身侧面装饰金线
  const sideLineMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.3,
  });
  const sideLineGeo = new THREE.BoxGeometry(BODY_W - 0.4, 0.02, 0.02);
  
  const sideLine1 = new THREE.Mesh(sideLineGeo, sideLineMat);
  sideLine1.position.set(0, -0.2, BODY_D/2 - 0.01);
  caseGroup.add(sideLine1);
  
  const sideLine2 = new THREE.Mesh(sideLineGeo, sideLineMat);
  sideLine2.position.set(0, -0.2, -BODY_D/2 + 0.01);
  caseGroup.add(sideLine2);
  
  // === 凹槽 ===
  const wellGeo = new THREE.CylinderGeometry(WELL_R, WELL_R - 0.1, 0.5, 64, 1, true);
  const wellMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.innerWell,
    metalness: 0.4,
    roughness: 0.25,
    side: THREE.DoubleSide,
  });
  
  const leftWellOuter = new THREE.Mesh(wellGeo, wellMat);
  leftWellOuter.position.set(-WELL_X, 0.36, 0);
  leftWellOuter.castShadow = true;
  leftWellOuter.receiveShadow = true;
  caseGroup.add(leftWellOuter);
  
  const rightWellOuter = new THREE.Mesh(wellGeo, wellMat);
  rightWellOuter.position.set(WELL_X, 0.36, 0);
  rightWellOuter.castShadow = true;
  rightWellOuter.receiveShadow = true;
  caseGroup.add(rightWellOuter);
  
  // 凹槽底部
  const wellBottomGeo = new THREE.CircleGeometry(WELL_R - 0.1, 64);
  const wellBottomMat = new THREE.MeshStandardMaterial({
    color: 0x051815,
    metalness: 0.3,
    roughness: 0.5,
  });
  
  const leftWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  leftWellBottom.rotation.x = -Math.PI / 2;
  leftWellBottom.position.set(-WELL_X, 0.115, 0);
  caseGroup.add(leftWellBottom);
  
  const rightWellBottom = new THREE.Mesh(wellBottomGeo, wellBottomMat);
  rightWellBottom.rotation.x = -Math.PI / 2;
  rightWellBottom.position.set(WELL_X, 0.115, 0);
  caseGroup.add(rightWellBottom);
  
  // 凹槽边缘金色装饰圈
  const ringGeo = new THREE.TorusGeometry(WELL_R + 0.02, 0.025, 16, 64);
  const ringMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.2,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.2,
  });
  
  const leftRing = new THREE.Mesh(ringGeo, ringMat);
  leftRing.position.set(-WELL_X, 0.6, 0);
  leftRing.rotation.x = -Math.PI / 2;
  caseGroup.add(leftRing);
  
  const rightRing = new THREE.Mesh(ringGeo, ringMat);
  rightRing.position.set(WELL_X, 0.6, 0);
  rightRing.rotation.x = -Math.PI / 2;
  caseGroup.add(rightRing);
  
  // === 液体表面 ===
  const liquidGeo = new THREE.CircleGeometry(WELL_R - 0.13, 64);
  const liquidMat = new THREE.MeshStandardMaterial({
    color: COLORS.liquid,
    metalness: 0.4,
    roughness: 0.05,
    transparent: true,
    opacity: 0.7,
    side: THREE.DoubleSide,
    emissive: COLORS.liquid,
    emissiveIntensity: 0.2,
  });
  
  leftLiquid = new THREE.Mesh(liquidGeo, liquidMat);
  leftLiquid.rotation.x = -Math.PI / 2;
  leftLiquid.position.set(-WELL_X, 0.5, 0);
  caseGroup.add(leftLiquid);
  
  rightLiquid = new THREE.Mesh(liquidGeo, liquidMat.clone());
  rightLiquid.rotation.x = -Math.PI / 2;
  rightLiquid.position.set(WELL_X, 0.5, 0);
  caseGroup.add(rightLiquid);
  
  // === 隐形眼镜 ===
  const lensGeo = new THREE.CircleGeometry(0.4, 64);
  const lensMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lensTint,
    metalness: 0.1,
    roughness: 0.05,
    transmission: 0.7,
    thickness: 0.05,
    ior: 1.4,
    side: THREE.DoubleSide,
    clearcoat: 0.8,
    clearcoatRoughness: 0.1,
    transparent: true,
    opacity: 0.6,
  });
  
  const leftLens = new THREE.Mesh(lensGeo, lensMat);
  leftLens.rotation.x = -Math.PI / 2;
  leftLens.position.set(-WELL_X, 0.48, 0);
  caseGroup.add(leftLens);
  
  const rightLens = new THREE.Mesh(lensGeo, lensMat.clone());
  rightLens.rotation.x = -Math.PI / 2;
  rightLens.position.set(WELL_X, 0.48, 0);
  caseGroup.add(rightLens);
  
  // === L 和 R 标识（盒身顶面，凹槽前方） ===
  const letterLBody = createLetterPlane('L', 0.45);
  letterLBody.position.set(-WELL_X, 0.605, 0.95);
  letterLBody.rotation.x = -Math.PI / 2;
  caseGroup.add(letterLBody);
  
  const letterRBody = createLetterPlane('R', 0.45);
  letterRBody.position.set(WELL_X, 0.605, 0.95);
  letterRBody.rotation.x = -Math.PI / 2;
  caseGroup.add(letterRBody);
  
  // 凹槽后方
  const letterLBody2 = createLetterPlane('L', 0.4);
  letterLBody2.position.set(-WELL_X, 0.605, -0.95);
  letterLBody2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterLBody2);
  
  const letterRBody2 = createLetterPlane('R', 0.4);
  letterRBody2.position.set(WELL_X, 0.605, -0.95);
  letterRBody2.rotation.x = -Math.PI / 2;
  caseGroup.add(letterRBody2);
  
  // === 盒盖 ===
  const lidH = 0.4;
  lidPivot = new THREE.Group();
  lidPivot.position.set(0, BODY_H/2, -BODY_D/2);
  caseGroup.add(lidPivot);
  
  const lidGeo = new RoundedBoxGeometry(BODY_W, lidH, BODY_D, 8, 0.15);
  const lidMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.lidColor,
    metalness: 0.7,
    roughness: 0.2,
    clearcoat: 0.9,
    clearcoatRoughness: 0.08,
  });
  lidMesh = new THREE.Mesh(lidGeo, lidMat);
  lidMesh.position.set(0, lidH/2, BODY_D/2);
  lidMesh.castShadow = true;
  lidMesh.receiveShadow = true;
  lidMesh.userData.isLid = true;
  lidPivot.add(lidMesh);
  
  // 盒盖顶面大的 L 和 R 标识
  const topL = createLetterPlane('L', 1.1);
  topL.position.set(-WELL_X, lidH/2 + 0.01, 0);
  topL.rotation.x = -Math.PI / 2;
  lidMesh.add(topL);
  
  const topR = createLetterPlane('R', 1.1);
  topR.position.set(WELL_X, lidH/2 + 0.01, 0);
  topR.rotation.x = -Math.PI / 2;
  lidMesh.add(topR);
  
  // 盒盖顶面装饰金线
  const lidLineGeo = new THREE.BoxGeometry(BODY_W - 0.6, 0.015, 0.03);
  const lidLineMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.4,
  });
  
  const lidLine1 = new THREE.Mesh(lidLineGeo, lidLineMat);
  lidLine1.position.set(0, lidH/2 + 0.005, BODY_D/2 - 0.25);
  lidMesh.add(lidLine1);
  
  const lidLine2 = new THREE.Mesh(lidLineGeo, lidLineMat);
  lidLine2.position.set(0, lidH/2 + 0.005, -BODY_D/2 + 0.25);
  lidMesh.add(lidLine2);
  
  // 中央装饰小圆
  const centerDotGeo = new THREE.CircleGeometry(0.08, 32);
  const centerDotMat = new THREE.MeshStandardMaterial({
    color: COLORS.letterGoldBright,
    metalness: 0.9,
    roughness: 0.15,
    emissive: COLORS.letterGoldBright,
    emissiveIntensity: 0.6,
  });
  const centerDot = new THREE.Mesh(centerDotGeo, centerDotMat);
  centerDot.position.set(0, lidH/2 + 0.01, 0);
  centerDot.rotation.x = -Math.PI / 2;
  lidMesh.add(centerDot);
  
  // 铰链
  const hingeGeo = new THREE.CylinderGeometry(0.07, 0.07, 0.35, 16);
  const hingeMat = new THREE.MeshStandardMaterial({
    color: 0x555555,
    metalness: 0.95,
    roughness: 0.15,
  });
  
  const hinge1 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge1.position.set(-1.6, BODY_H/2, -BODY_D/2);
  hinge1.rotation.z = Math.PI / 2;
  hinge1.castShadow = true;
  caseGroup.add(hinge1);
  
  const hinge2 = new THREE.Mesh(hingeGeo, hingeMat);
  hinge2.position.set(1.6, BODY_H/2, -BODY_D/2);
  hinge2.rotation.z = Math.PI / 2;
  hinge2.castShadow = true;
  caseGroup.add(hinge2);
  
  // 盒身前面卡扣装饰
  const latchGeo = new RoundedBoxGeometry(0.4, 0.15, 0.1, 4, 0.04);
  const latchMat = new THREE.MeshPhysicalMaterial({
    color: COLORS.letterGold,
    metalness: 0.9,
    roughness: 0.2,
    clearcoat: 0.6,
    emissive: COLORS.letterGold,
    emissiveIntensity: 0.15,
  });
  
  const latchSlot = new THREE.Mesh(latchGeo, latchMat);
  latchSlot.position.set(0, 0.55, BODY_D/2 - 0.02);
  caseGroup.add(latchSlot);
  
  scene.add(caseGroup);
}

function createLetterPlane(letter, scale = 0.5) {
  const texture = createLetterTexture(letter);
  
  const planeGeo = new THREE.PlaneGeometry(scale, scale);
  const planeMat = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
  });
  
  return new THREE.Mesh(planeGeo, planeMat);
}

function createLetterTexture(letter) {
  const size = 512;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, size, size);
  
  // 翻转 y 轴，让字母在 3D 中正向显示
  ctx.save();
  ctx.translate(0, size);
  ctx.scale(1, -1);
  
  // 阴影发光
  ctx.shadowColor = 'rgba(255, 200, 80, 0.9)';
  ctx.shadowBlur = 60;
  
  // 渐变填充
  const gradient = ctx.createLinearGradient(0, 0, 0, size);
  gradient.addColorStop(0, '#ffe066');
  gradient.addColorStop(0.5, '#ffb347');
  gradient.addColorStop(1, '#ff7e2e');
  
  ctx.fillStyle = gradient;
  ctx.font = 'bold 400px Georgia, serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 加强发光
  ctx.shadowBlur = 25;
  ctx.fillText(letter, size/2, size/2 + 10);
  
  // 描边
  ctx.shadowBlur = 0;
  ctx.strokeStyle = 'rgba(255, 245, 200, 0.6)';
  ctx.lineWidth = 5;
  ctx.strokeText(letter, size/2, size/2 + 10);
  
  ctx.restore();
  
  const texture = new THREE.CanvasTexture(canvas);
  texture.anisotropy = 16;
  texture.needsUpdate = true;
  return texture;
}

function createParticles() {
  const count = 180;
  const positions = new Float32Array(count * 3);
  const speeds = new Float32Array(count);
  
  for (let i = 0; i < count; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 14;
    positions[i * 3 + 1] = Math.random() * 7 - 0.5;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 14;
    speeds[i] = 0.002 + Math.random() * 0.004;
  }
  
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
  gradient.addColorStop(0, 'rgba(255, 200, 100, 1)');
  gradient.addColorStop(0.3, 'rgba(255, 180, 80, 0.6)');
  gradient.addColorStop(1, 'rgba(255, 180, 80, 0)');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, 64, 64);
  const particleTexture = new THREE.CanvasTexture(canvas);
  
  const mat = new THREE.PointsMaterial({
    size: 0.08,
    map: particleTexture,
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.5,
    sizeAttenuation: true,
  });
  
  particleSystem = new THREE.Points(geo, mat);
  particleSystem.userData.speeds = speeds;
  scene.add(particleSystem);
}

function onPointerDown(event) {
  mouseDownPos = { x: event.clientX, y: event.clientY };
  mouseDownTime = Date.now();
}

function onPointerUp(event) {
  const dx = event.clientX - mouseDownPos.x;
  const dy = event.clientY - mouseDownPos.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  const duration = Date.now() - mouseDownTime;
  
  if (distance < 5 && duration < 400) {
    handleClick(event);
  }
}

function onPointerMove(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, true);
  
  if (intersects.length > 0) {
    if (!hoveredLid) {
      hoveredLid = true;
      document.body.style.cursor = 'pointer';
    }
  } else {
    if (hoveredLid) {
      hoveredLid = false;
      document.body.style.cursor = 'default';
    }
  }
}

function handleClick(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(lidMesh, true);
  
  if (intersects.length > 0) {
    toggleLid();
    showToast(isOpen ? '盒盖已开启' : '盒盖已关闭');
  }
}

function toggleLid() {
  isOpen = !isOpen;
  targetLidAngle = isOpen ? -Math.PI * 0.62 : 0;
  updateUI();
}

function updateUI() {
  document.getElementById('state-text').textContent = isOpen ? 'OPEN' : 'CLOSED';
  document.getElementById('toggle-btn').textContent = isOpen ? '关闭盒盖' : '开启盒盖';
  const indicator = document.getElementById('state-indicator');
  if (indicator) {
    if (isOpen) {
      indicator.style.background = '#4ecdc4';
      indicator.style.boxShadow = '0 0 10px #4ecdc4';
    } else {
      indicator.style.background = '#f5a623';
      indicator.style.boxShadow = '0 0 10px #f5a623';
    }
  }
}

function showToast(message) {
  const toast = document.getElementById('toast');
  toast.textContent = message;
  toast.classList.add('show');
  clearTimeout(showToast.timer);
  showToast.timer = setTimeout(() => {
    toast.classList.remove('show');
  }, 1800);
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

function animate() {
  requestAnimationFrame(animate);
  
  const time = clock.getElapsedTime();
  
  // 盖子平滑过渡
  if (Math.abs(lidAngle - targetLidAngle) > 0.001) {
    lidAngle += (targetLidAngle - lidAngle) * 0.08;
    lidPivot.rotation.x = lidAngle;
  } else if (lidAngle !== targetLidAngle) {
    lidAngle = targetLidAngle;
    lidPivot.rotation.x = lidAngle;
  }
  
  // 内部灯光
  if (innerLight) {
    const targetIntensity = isOpen ? 1.8 : 0;
    innerLight.intensity += (targetIntensity - innerLight.intensity) * 0.1;
  }
  
  // 液体波动
  if (leftLiquid) {
    leftLiquid.position.y = 0.5 + Math.sin(time * 2) * 0.003;
  }
  if (rightLiquid) {
    rightLiquid.position.y = 0.5 + Math.cos(time * 2.2) * 0.003;
  }
  
  // 粒子飘动
  if (particleSystem) {
    const positions = particleSystem.geometry.attributes.position.array;
    const speeds = particleSystem.userData.speeds;
    const count = positions.length / 3;
    
    for (let i = 0; i < count; i++) {
      positions[i * 3 + 1] += speeds[i];
      if (positions[i * 3 + 1] > 6.5) {
        positions[i * 3 + 1] = -0.5;
        positions[i * 3] = (Math.random() - 0.5) * 14;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 14;
      }
      positions[i * 3] += Math.sin(time + i * 0.5) * 0.0008;
      positions[i * 3 + 2] += Math.cos(time + i * 0.3) * 0.0008;
    }
    particleSystem.geometry.attributes.position.needsUpdate = true;
  }
  
  // 整体微微浮动
  if (caseGroup) {
    caseGroup.position.y = Math.sin(time * 0.8) * 0.03;
    caseGroup.rotation.y = Math.sin(time * 0.3) * 0.015;
  }
  
  controls.update();
  renderer.render(scene, camera);
}

init();
```

让我做最后的检查和优化。

字母方向问题：我之前推导过，在 canvas 上翻转 y 后，字母在 3D 中正向显示。让我再验证一下。

Canvas 翻转后：ctx.translate(0, size); ctx.scale(1, -1); 
- 原点从 (0, 0) 移到 (0, size)
- y 方向反转

在翻转后的坐标系中，ctx.fillText(letter, size/2, size/2 + 10) 在 (size/2, size/2 + 10) 处绘制字母。
- 这在原 canvas 坐标系中的位置是 (size/2, size - (size/2 + 10)) = (size/2, size/2 - 10)。
- 即字母绘制在原 canvas 中心略上方处。

字母在 canvas 上看起来怎样？由于坐标系翻转，字母本身也上下翻转。所以 canvas 上字母是上下颠倒的。

但是字母在 plane 上显示时，UV 映射让 canvas y=0 (顶部) → plane y=+0.5 (上方)。所以 canvas 顶部的内容在 plane 上方。

Canvas 上字母上下颠倒，即字母"顶部"在 canvas 底部。映射到 plane，字母"顶部"在 plane 底部（-y 方向）。

旋转 -π/2 绕 x 轴后，plane -y 方向 → +z 方向（远处）。所以字母"顶部"在远处。

从相机俯视，字母"顶部"在远处，"底部"在近处。这是正向的字母方向 ✓。

OK 字母方向正确。

让我再考虑一下 lidMesh 子对象（字母 plane）的旋转。

lidMesh 在 lidPivot 局部 (0, 0.2, 1.2)，没有自身旋转（rotation = 0,0,0）。
字母 plane 在 lidMesh 局部 (-1.15, 0.21, 0)，rotation.x = -π/2。

所以字母 plane 在 lidMesh 局部的方向：法线朝 +y。这是 plane 朝上，从上面看是正向字母。

当 lidMesh 关闭时（lidPivot.rotation.x = 0），lidMesh 局部和世界坐标一致（除了 lidPivot 的位置偏移）。所以字母 plane 在世界坐标也朝上，正向显示 ✓。

当 lidMesh 打开时（lidPivot.rotation.x = -120°），lidMesh 跟着 lidPivot 旋转。字母 plane 跟着旋转。这时字母朝向斜后方，从相机角度看可能朝下或朝后。

但是字母 plane 是 lidMesh 的子对象，用 raycast recursive=true 可以检测到。所以用户点击字母 plane 也能触发 toggleLid。

OK 这个设计合理。

最后一个问题：lidMesh 在 lidPivot 局部 (0, lidH/2, BODY_D/2)。lidPivot 在 caseGroup 局部 (0, BODY_H/2, -BODY_D/2)。

所以 lidMesh 中心在 caseGroup 局部 (0, BODY_H/2 + lidH/2, 0) = (0, 0.6 + 0.2, 0) = (0, 0.8, 0)。

lidMesh 尺寸 (BODY_W, lidH, BODY_D) = (4.4, 0.4, 2.4)。所以 lidMesh 在 caseGroup 局部范围：
- x: -2.2 到 2.2
- y: 0.6 到 1.0
- z: -1.2 到 1.2

盒身范围：
- x: -2.2 到 2.2
- y: -0.6 到 0.6
- z: -1.2 到 1.2

所以盒盖底面 y=0.6 正好接触盒身顶面 y=0.6 ✓。

但是 lidMesh 旋转后，位置会变。当 lidPivot.rotation.x = 0 时（关闭），lidMesh 在 (0, 0.8, 0)。当 lidPivot.rotation.x = -120° 时，lidMesh 围绕 lidPivot (0, 0.6, -1.2) 旋转 -120°。

旋转矩阵（绕 x 轴）：
- (0, 0.2, 1.2) 旋转 -120° → (0, 0.2*cos(-120°) - 1.2*sin(-120°), 0.2*sin(-120°) + 1.2*cos(-120°))
- cos(-120°) = -0.5, sin(-120°) = -0.866
- = (0, 0.2*(-0.5) - 1.2*(-0.866), 0.2*(-0.866) + 1.2*(-0.5))
- = (0, -0.1 + 1.039, -0.173 - 0.6)
- = (0, 0.939, -0.773)

加上 lidPivot 位置 (0, 0.6, -1.2)，得到 lidMesh 中心在 caseGroup 局部 (0, 1.539, -1.973)。

打开后 lidMesh 在 (0, 1.539, -1.973)，盒盖在盒子后上方。从相机看，盒盖向后上方翻起，露出盒身内部 ✓。

打开角度 -120° 可能太大了，让我用 -100° (-Math.PI * 0.555)：
- cos(-100°) ≈ -0.174, sin(-100°) ≈ -0.985
- = (0, 0.2*(-0.174) - 1.2*(-0.985), 0.2*(-0.985) + 1.2*(-0.174))
- = (0, -0.035 + 1.182, -0.197 - 0.209)
- = (0, 1.147, -0.406)

加上 lidPivot 位置：(0, 1.747, -1.606)。

打开角度 -110° (-Math.PI * 0.611)：
- cos(-110°) ≈ -0.342, sin(-110°) ≈ -0.94
- = (0, 0.2*(-0.342) - 1.2*(-0.94), 0.2*(-0.94) + 1.2*(-0.342))
- = (0, -0.068 + 1.128, -0.188 - 0.41)
- = (0, 1.06, -0.598)

加上 lidPivot 位置：(0, 1.66, -1.798)。

我用 -Math.PI * 0.62 (-111.6°)，接近 -110°。

OK 完成。

让我检查代码有没有遗漏。

- `lensMat` 用 transmission 时是否需要 transparent？实际上 transmission 已经处理透明，不需要 transparent。但加了 transparent + opacity 让镜片更透明。OK。

- 字母 plane 用 depthWrite: false，防止遮挡其他透明物体。OK。

- 粒子的 sizeAttenuation: true 让粒子随距离缩放。OK。

- 地板 ShadowMaterial 让阴影显示但地板透明。OK。

让我整理输出。

Actually 让我再优化一下。我注意到 caseGroup.rotation.y 会微微变化。这可能让点击 lidMesh 不稳定（lidMesh 位置会变）。但是变化很小（±0.015 弧度，约 ±1°），点击应该还能命中。

另外，让我添加一个细节：盒盖打开时，盒盖的字母 plane（在 lidMesh 上）会跟着 lidMesh 旋转。这时字母朝向斜后方/下方。从相机角度看可能看到字母的背面。

字母 plane 用 DoubleSide 材质，两面都显示。从背面看字母是左右镜像的（因为 plane 翻转了）。所以从相机看打开后的盒盖字母，可能是镜像的字母。

但是打开后用户主要看盒身内部，盒盖字母在视野外或者不重要。所以这个细节可以忽略。

让我最终输出代码。

实际上我还需要测试：
1. 点击 lidMesh 触发开盖。lidMesh 包括子对象（字母 plane、金线等）。我用 recursive=true，所以所有子对象都能触发点击。

但是金线、中心点等子对象都是 Mesh，会被 raycast 检测到。点击它们也会触发 toggleLid。这没问题。

字母 plane 是 Mesh 也会被检测到。点击字母也会触发 toggleLid。✓

2. 视觉测试：
- 关闭状态：盒盖盖在盒身上，盒盖顶面 L 和 R 醒目可见。金色装饰圈、金线、中心点闪烁。盒子在墨绿色背景中突出。
- 打开状态：盒盖向后上方翻起，盒身内部凹槽、液体（半透明青色）、隐形眼镜（透明圆盘）、L 和 R 标识（金色）可见。内部青色灯光亮起。

OK 设计完整。让我整理最终代码。

让我数下代码量：CSS ~200 行，HTML ~50 行，JS ~500 行。总共约 800 行。

输出之前，让我再加一些细节让盒子更高级：
1. 盒身侧面也可以加品牌名"LUMINA"（可选）
2. 盒盖打开时有声音效果（可选，不加）
3. 盒身底部可以加防滑纹理（可选）

不加这些额外细节，保持代码量合理。

最终输出代码。

不过我注意到 `lidLine1` 和 `lidLine2` 的位置：lidLine1 在 z = BODY_D/2 - 0.25 = 1.2 - 0.25 = 0.95，lidLine2 在 z = -BODY_D/2 + 0.25 = -0.95。

但是 lidMesh 局部坐标系，lidMesh 是 RoundedBoxGeometry(BODY_W, lidH, BODY_D)，z 范围 -BODY_D/2 到 BODY_D/2。所以 lidLine1 在 z=0.95 在 lidMesh 局部范围内（最大 1.2）。✓

Wait lidMesh 的几何尺寸是 (BODY_W, lidH, BODY_D) = (4.4, 0.4, 2.4)。lidMesh 局部坐标范围 x: ±2.2, y: ±0.2, z: ±1.2。

lidLine1 在 lidMesh 局部 (0, 0.205, 0.95)。lidLine2 在 (0, 0.205, -0.95)。

字母 plane 在 lidMesh 局部 (-1.15, 0.21, 0) 和 (1.15, 0.21, 0)。

字母 plane 和金线可能在 z 方向重叠（字母 plane 是平面在 z=0，金线在 z=±0.95）。但金线在 z=±0.95，字母 plane 在 z=0。距离 0.95，不会重叠。✓

但是字母 plane 是 PlaneGeometry(1.1, 1.1)，在 lidMesh 局部 (-1.15, 0.21, 0)，plane 在 xy 平面（rotation.x = -π/2 后变为 xz 平面）。所以字母 plane 占据 lidMesh 局部 x: -1.7 到 -0.6, z: -0.55 到 0.55。

金线 lidLine1 在 lidMesh 局部 z=0.95，不与字母 plane 重叠。✓

OK 代码没问题。

让我最终输出。

哦还有一个问题：CSS 中 backdrop-filter 在某些浏览器需要前缀。但现代浏览器都支持，不加前缀。

OK 输出最终代码。

Wait 我想再优化一下视觉：盒子看起来有点单调。让我加：
- 盒身侧面纹理或凹槽（用 canvas 创建纹理）

实际上 RoundedBoxGeometry 已经让盒子有圆角，加上金线和金色字母，应该够了。

最终输出。

Wait 让我重新检查 lidMesh raycast。lidMesh 是 RoundedBoxGeometry，是一个 mesh。raycaster.intersectObject(lidMesh, true) 会检测 lidMesh 和它的所有子对象。

子对象包括：topL, topR, lidLine1, lidLine2, centerDot。这些都是 Mesh。

但是字母 plane 是 MeshBasicMaterial + transparent + depthWrite: false。这些都不影响 raycast。

点击 lidMesh 上任何位置（包括字母、金线、中心点）都会触发 toggleLid。这是期望的行为。

OK 最终输出。

让我加一点视觉效果：盒盖打开时加一个粒子爆发效果，从凹槽里飞出。

不加了，保持代码简洁。

最终输出。

让我再补充一个细节：当 lidMesh 关闭时，lidMesh 底面紧贴盒身顶面。如果两者完全重叠可能 z-fighting。但是 lidMesh 底面 y=0.6，盒身顶面 y=0.6。

实际上 lidMesh 底面在 lidMesh 局部 y=-0.2，lidMesh 中心在 caseGroup 局部 (0, 0.8, 0)，所以 lidMesh 底面在 caseGroup 局部 y=0.6。

盒身顶面在 caseGroup 局部 y=0.6（盒身几何中心在 (0,0,0)，高度 1.2，所以顶面 y=0.6）。

两者都在 y=0.6。可能 z-fighting。

解决：让 lidMesh 底面略微高于盒身顶面。调整 lidMesh 在 lidPivot 局部的 y 位置：

lidMesh.position.set(0, lidH/2 + 0.005, BODY_D/2);

这样 lidMesh 中心在 lidPivot 局部 y=0.205，lidMesh 底面 y=0.005（lidPivot 局部）。lidPivot 在 caseGroup 局部 y=0.6，所以 lidMesh 底面在 caseGroup 局部 y=0.605。盒身顶面在 y=0.6。两者间隔 0.005。✓

但是这样盒盖浮在盒身上方 0.005，看起来不自然。让我直接让 lidMesh 底面接触盒身顶面，z-fighting 应该不严重（因为 lidMesh 和盒身的几何不重叠，只是表面接触）。

实际上 z-fighting 只在两个表面的像素完全重叠时发生。lidMesh 底面和盒身顶面都是平面，重叠部分会 z-fight。

让我把 lidMesh 底面略微提高，0.005 即可：

```javascript
lidMesh.position.set(0, lidH/2 + 0.005, BODY_D/2);
```

这样 lidMesh 底面 y=0.005（lidPivot 局部），在 caseGroup 局部 y=0.605，盒身顶面 y=0.6，间隔 0.005。

但是从外观上看，盒盖底面浮在盒身上方 0.005，几乎看不出。✓

或者让盒身顶面略低一点。Body 高度 1.2，如果改成 1.19，盒身顶面 y=0.595，与 lidMesh 底面 y=0.6 间隔 0.005。

更简单：让 lidMesh 略高一点。我用 lidH/2 + 0.005。

OK 用这个方案。

让我最终输出代码。

实际上让我再考虑一下铰链位置。铰链在 (0, BODY_H/2, -BODY_D/2) = (0, 0.6, -1.2)。lidPivot 在这个位置。lidMesh 在 lidPivot 局部 (0, lidH/2 + 0.005, BODY_D/2)。

lidMesh 中心在 caseGroup 局部 (0, 0.6 + lidH/2 + 0.005, -1.2 + 1.2) = (0, 0.6 + 0.2 + 0.005, 0) = (0, 0.805, 0)。

lidMesh 范围 y: 0.805 - 0.2 = 0.605 到 0.805 + 0.2 = 1.005。

盒身顶面 y=0.6，lidMesh 底面 y=0.605。间隔 0.005 ✓。

铰链位置 (0, 0.6, -1.2) 在 lidMesh 后边缘下方。lidMesh 后边缘 z=-1.2 在 caseGroup 局部。铰链 z=-1.2 ✓。

但是铰链在 lidMesh 底面下方（y=0.6 vs lidMesh 底面 y=0.605）。铰链 y=0.6，lidMesh 底面 y=0.605。铰链在 lidMesh 底面下方 0.005。

实际上铰链是个圆柱，半径 0.07，长 0.35。圆柱水平放置（rotation.z = π/2），中心在 (0, 0.6, -1.2)。圆柱 y 范围 0.6 - 0.07 到 0.6 + 0.07 = 0.53 到 0.67。圆柱 z 范围 -1.2 - 0.175 到 -1.2 + 0.175 = -1.375 到 -1.025。

所以铰链顶部 y=0.67，超过 lidMesh 底面 y=0.605。铰链会穿过 lidMesh 底面。✗

不对，铰链应该穿过 lidMesh 和盒身后边缘，连接两者。这是合理的（铰链就是穿过两个物体的）。

OK 不用担心这个细节。

最终输出代码。

让我把 lidMesh 的 y 位置调回 lidH/2（不加 0.005）：
```javascript
lidMesh.position.set(0, lidH/2, BODY_D/2);
```

lidMesh 底面 y=0（lidPivot 局部），在 caseGroup 局部 y=0.6，盒身顶面 y=0.6。两者接触。

z-fighting 可能发生，但 lidMesh 旋转时会立即分开，所以只在关闭状态下可能 z-fight。

让我用 lidH/2 + 0.001（极小间隔）：
```javascript
lidMesh.position.set(0, lidH/2 + 0.001, BODY_D/2);
```

lidMesh 底面 y=0.001（lidPivot 局部），在 caseGroup 局部 y=0.601，盒身顶面 y=0.6。间隔 0.001。看起来接触，但实际有微小间隔，避免 z-fighting。

OK 用这个。

实际上 0.001 太小可能还会有 z-fighting。让我用 0.005：
```javascript
lidMesh.position.set(0, lidH/2 + 0.005, BODY_D/2);
```

lidMesh 底面 y=0.605，盒身顶面 y=0.6。间隔 0.005。从外观上几乎看不出。✓

OK 最终代码。让我输出。

我现在准备最终输出。代码量比较大，但应该没有错误。让我把完整 HTML 文件输出。

一个小问题：我注意到 caseGroup 整体微微旋转和浮动。这会让 OrbitControls 的旋转中心也跟着变。OrbitControls.target 是 (0, 0.2, 0)，是固定的。如果 caseGroup 旋转，caseGroup 不在 target 中心，相机绕 target 旋转时会偏离 caseGroup。

实际上 caseGroup 的旋转幅度很小（±0.015 弧度，约 ±1°），影响不大。但是浮动（±0.03）会让盒子上下移动，相机视角不变，盒子在屏幕上位置变化。

这是有意为之的效果：盒子像漂浮在空中，有微妙的动态感。OK。

如果用户不喜欢这个效果，可以删除 caseGroup 的浮动代码。

让我最终输出代码。

实际上让我再想一下：caseGroup 的整体旋转可能让用户点击 lidMesh 时位置不准确。但是旋转幅度很小，应该没问题。

OK 最终输出。

让我把代码整理成最终版本。

让我考虑一下 transmission 材质。我看了下 Three.js 0.160 文档，MeshPhysicalMaterial 的 transmission 是有效的。但是需要 renderer 支持。

`transmission` 属性需要材质的 `transparent` 不被设置为 true（transmission 自己处理透明）。如果同时设置 transparent=true 和 transmission，可能行为不一致。

让我去掉 transparent，只用 transmission：

```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0.1,
  roughness: 0.05,
  transmission: 0.8,
  thickness: 0.05,
  ior: 1.4,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
});
```

但是 transmission 在某些情况下可能渲染不正确。让我用 transparent + opacity 作为后备：

```javascript
const lensMat = new THREE.MeshPhysicalMaterial({
  color: COLORS.lensTint,
  metalness: 0.1,
  roughness: 0.05,
  transparent: true,
  opacity: 0.5,
  side: THREE.DoubleSide,
  clearcoat: 0.8,
  clearcoatRoughness: 0.1,
});
```

我用 transparent + opacity 简单稳定。

OK 最终代码。

让我最终输出。