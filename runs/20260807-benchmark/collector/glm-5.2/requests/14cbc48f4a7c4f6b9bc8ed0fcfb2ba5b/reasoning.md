用户想要一个科幻小说《极光》中的宇宙飞船的3D可视化。让我思考一下：

1. 《极光》是金·斯坦利·罗宾逊2015年的科幻小说，讲述了一艘世代飞船"极光号"(Aurora)前往Tau Ceti系统的旅程。飞船是一个巨大的旋转结构，分为多环，包含岛屿生态系统、海洋等。

2. 我需要用Three.js来构建这个飞船，让它具备：
   - 360度旋转查看
   - 缩放功能
   - 切换到内部视图
   - 深空背景
   - 动态效果（旋转的环、灯光、星空）
   - 飞船在宇宙深空航行的感觉

3. 技术选择：
   - Three.js with ES Modules
   - importmap
   - OrbitControls用于360度查看和缩放
   - 自定义场景切换
   - 粒子星空背景
   - 飞船的多层结构：双环系统（一个环用于居住，另一个用于其他）、中央轴、连接辐条

让我设计极光号的具体结构：

根据小说：
- 飞船由两个同心的环组成，称为"环"(Rings)
- 环通过辐条连接到中央轴
- 环旋转产生人造重力
- 飞船长度约10-15公里
- 包含岛屿、海洋、森林、城镇
- 有"星星"(stars)系统提供照明
- 中央轴是零重力区域，包含飞船系统
- 前部有防护盾(protective shield)防御星际尘埃
- 后部有推进器

我的3D模型设计：
1. 外环- 大型居住区
2. 内环- 较小的居住区
3. 辐条(6-8条)连接环到中央轴
4. 中央轴- 长圆柱形结构
5. 前部防护盾- 凸圆形
6. 后部推进器- 喷射蓝色火焰
7. 内部视图：可以看到环内的"地面"、建筑、灯光

我会创建一个完整的产品级页面，包括：
- 加载界面
- 控制面板
- 信息HUD
- 内部/外部切换
- 高质量的视觉效果

颜色方案：避免蓝紫色。使用暖色调与冷色调对比，比如橙色和青绿色，呼应"极光"主题。极光是绿色和粉红色调的，所以可以用绿色/青色作为主色，配以暖色辅助。

让我开始写代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>极光号 - 世代飞船</title>
<style>
  /* CSS Variables */
  :root {
    --bg: #050811;
    --bg-2: #0a1020;
    --fg: #e8f4ff;
    --muted: #8aa0c0;
    --accent: #5feec5;  /* aurora green */
    --accent-2: #ff7a8a; /* aurora pink */
    --warning: #ffb86b;
    --card: rgba(15, 25, 40, 0.7);
    --border: rgba(95, 238, 197, 0.25);
  }
  
  /* 全局样式 */
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { height: 100%; overflow: hidden; }
  body {
    background: var(--bg);
    color: var(--fg);
    font-family: 'Inter', -apple-system, sans-serif;
  }
  
  /* Canvas 容器 */
  #canvas-container {
    position: fixed;
    inset: 0;
    z-index: 1;
  }
  
  /* 加载界面 */
  #loader {
    position: fixed;
    inset: 0;
    background: radial-gradient(ellipse at center, var(--bg-2) 0%, var(--bg) 100%);
    z-index: 100;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: opacity 1s ease;
  }
  
  ...
</style>
</head>
<body>
  <div id="loader">...</div>
  <div id="canvas-container"></div>
  <div id="ui">
    <header>...</header>
    <aside class="info-panel">...</aside>
    <div class="controls">...</div>
    <div class="hud">...</div>
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
    
    // 构建飞船
    function buildAuroraShip() {
      const ship = new THREE.Group();
      
      // 外环
      // 内环
      // 辐条
      // 中央轴
      // 前部防护盾
      // 推进器
      
      return ship;
    }
    
    // 星空背景
    function buildStarfield() {...}
    
    // 内部场景
    function buildInterior() {...}
    
    // 主循环
    function animate() {...}
    
    animate();
  </script>
</body>
</html>
```

让我详细规划飞船的几何结构：

**外环**：
- 大半径：60
- 管半径：8（粗）
- 这是一个环面，但是我会用更复杂的方式 - 用CylinderGeometry组成的多段环

实际上，更准确地说，世代飞船的环是巨大的圆环，截面是矩形（不是圆管）。我会用：
- TorusGeometry作为基础形状
- 或者用ExtrudeGeometry沿圆形路径

我用Torus + 修改会更简单。

让我设计具体参数：
- 外环：半径60，截面8x8（粗的环管）
- 内环：半径40，截面6x6
- 中央轴：半径4，长度100（沿X轴）
- 辐条：8条，连接外环到中央轴
- 前部防护盾：半径20的凸面，位于X=+60
- 推进器：位于X=-60

为了视觉效果，我会：
- 让环慢慢旋转（产生人造重力感）
- 推进器持续喷射粒子
- 飞船整体在星空背景中缓慢移动（视觉上是星空在动）
- 环上有窗户灯光（点光源）
- 中央轴有指示灯

内部视图设计：
当用户点击"进入飞船"，相机移动到环内部，可以看到：
- 环形走廊
- 上方是远端的"天空"（环对面）
- 两侧有建筑（程序化生成）
- 中央有河流/海洋
- 灯光从辐条方向照射

让我开始写完整代码。我会精心设计每个细节。

代码结构：

1. HTML结构 - 加载界面、UI面板、Canvas容器
2. CSS样式 - 现代深空主题，使用CSS变量
3. importmap - Three.js模块
4. 主script:
   - 场景设置（场景、相机、渲染器、光照）
   - 星空背景（粒子系统）
   - 飞船构建（多个函数构建不同部分）
   - 内部场景构建
   - 控制（OrbitControls + 自定义切换逻辑）
   - UI交互
   - 动画循环

让我把代码写得详尽且高质量：

```javascript
// 创建飞船
function createAuroraShip() {
  const ship = new THREE.Group();
  ship.name = 'Aurora';
  
  // === 外环（主居住环）===
  const outerRing = createLivingRing(60, 12, 0x4a8b7a);
  ship.add(outerRing);
  
  // === 内环（次居住环）===
  const innerRing = createLivingRing(35, 8, 0x5fa5c4);
  ship.add(innerRing);
  
  // === 中央轴 ===
  const axisLength = 140;
  const axisGeometry = new THREE.CylinderGeometry(5, 5, axisLength, 32);
  const axisMaterial = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.7,
    roughness: 0.4
  });
  const axis = new THREE.Mesh(axisGeometry, axisMaterial);
  axis.rotation.z = Math.PI / 2;
  ship.add(axis);
  
  // 中央轴细节 - 加环
  for (let i = -3; i <= 3; i++) {
    const ringGeo = new THREE.TorusGeometry(6, 0.5, 8, 32);
    const ringMat = new THREE.MeshStandardMaterial({
      color: 0x5feec5,
      emissive: 0x5feec5,
      emissiveIntensity: 0.5
    });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.rotation.y = Math.PI / 2;
    ring.position.x = i * 15;
    ship.add(ring);
  }
  
  // === 辐条（连接环到轴）===
  const spokeCount = 8;
  for (let i = 0; i < spokeCount; i++) {
    const angle = (i / spokeCount) * Math.PI * 2;
    
    // 外环辐条
    const outerSpoke = createSpoke(60, 5, angle);
    ship.add(outerSpoke);
    
    // 内环辐条
    const innerSpoke = createSpoke(35, 5, angle);
    ship.add(innerSpoke);
  }
  
  // === 前部防护盾 ===
  const shieldGeo = new THREE.SphereGeometry(20, 32, 16, 0, Math.PI * 2, 0, Math.PI / 2);
  const shieldMat = new THREE.MeshStandardMaterial({
    color: 0x8a9aa5,
    metalness: 0.8,
    roughness: 0.3,
    side: THREE.DoubleSide
  });
  const shield = new THREE.Mesh(shieldGeo, shieldMat);
  shield.rotation.z = -Math.PI / 2;
  shield.position.x = 80;
  ship.add(shield);
  
  // 防护盾支撑
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2;
    const supportGeo = new THREE.CylinderGeometry(0.5, 0.5, 25, 8);
    const supportMat = new THREE.MeshStandardMaterial({ color: 0x4a5560, metalness: 0.7 });
    const support = new THREE.Mesh(supportGeo, supportMat);
    support.rotation.z = Math.PI / 2;
    support.position.set(70, Math.cos(angle) * 8, Math.sin(angle) * 8);
    ship.add(support);
  }
  
  // === 推进器 ===
  // 主推进器（多个）
  for (let i = 0; i < 4; i++) {
    const angle = (i / 4) * Math.PI * 2;
    const thruster = createThruster();
    thruster.position.set(-75, Math.cos(angle) * 6, Math.sin(angle) * 6);
    ship.add(thruster);
  }
  
  return ship;
}

function createLivingRing(radius, thickness, baseColor) {
  const ringGroup = new THREE.Group();
  
  // 主环体
  const ringGeo = new THREE.TorusGeometry(radius, thickness, 16, 64);
  const ringMat = new THREE.MeshStandardMaterial({
    color: baseColor,
    metalness: 0.5,
    roughness: 0.6
  });
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ringGroup.add(ring);
  
  // 窗户灯光
  const windowCount = 80;
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const windowGeo = new THREE.BoxGeometry(0.5, 0.3, 0.3);
    const windowMat = new THREE.MeshBasicMaterial({
      color: 0xffd88a,
      transparent: true,
      opacity: 0.9
    });
    const window = new THREE.Mesh(windowGeo, windowMat);
    const x = Math.cos(angle) * (radius + thickness);
    const y = Math.sin(angle) * (radius + thickness);
    window.position.set(x, y, 0);
    window.rotation.z = angle;
    ringGroup.add(window);
  }
  
  return ringGroup;
}
```

我想让环面朝向让飞船看起来更壮观。让我重新思考飞船的轴向：

如果飞船是水平的（沿X轴），那么：
- 中央轴沿X轴方向
- 两个环在YZ平面（垂直于X轴）旋转

但是这样从侧面看不到环的旋转效果，从正面（X方向）才能看到。

更好的方案是让飞船的环在XY平面上（飞船"平躺"），这样从默认视角可以清楚看到环结构。但小说里世代飞船的环通常都是绕着飞船轴向旋转的，所以更合理的是绕X轴旋转。

让我让飞船整体绕Y轴有一个轻微的倾斜，这样从默认视角（Z+方向）可以同时看到环的正面和侧面。

或者更好的方案：飞船沿X轴方向，但相机初始位置不是正前方，而是斜上方/斜侧面，这样可以看到环的椭圆视角。

我决定：
- 飞船主轴沿X方向
- 环在YZ平面（垂直于X）
- 相机初始位置在(80, 40, 80)，看向(0,0,0)

这样可以看到飞船的3/4视角，环呈椭圆形。

让我完善代码：

实际上，让我再思考一下世代飞船的视觉特征。极光号的描述：
- 巨大的双环结构
- 环内有"biomes"（生物群落）
- 中央通道连接两个环
- 前部有大磁盾防护星际尘埃
- 后部有离子推进器

我的设计：
1. 双环系统：外环和内环（不同半径）
2. 环之间有连接桥
3. 辐条从环连接到中央轴
4. 中央长轴
5. 前部磁盾（凹面盘）
6. 后部推进器集群

让我开始写最终代码。我会做得非常详细，确保视觉效果出色。

一些视觉技巧：
- 使用半透明发光材质表现防护盾
- 使用粒子表现推进器尾焰
- 使用程序化纹理给环表面增加细节
- 使用instanced mesh提高窗户性能
- 添加体积光感觉（用平面+渐变贴图模拟）

让我也加入：
- 后处理：bloom发光效果
- 但这需要EffectComposer，会增加复杂度。让我用简单的发光材质替代。

实际上，添加bloom会大幅提升视觉效果。让我加上：
```js
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
```

OK，让我开始写完整的最终代码。我会让它看起来非常专业。

我需要谨慎处理：
1. 环的旋转方向：让环绕飞船主轴（X轴）旋转
2. 但是环上的物体（窗户、装饰）应该跟着环一起转
3. 所以我会把所有环上的元素放在一个Group里，然后让Group绕X轴旋转

让我组织代码：

```javascript
// === 飞船结构 ===
const ship = new THREE.Group();

// 外环组（包含所有外环元素）
const outerRingGroup = new THREE.Group();
// 添加环、窗户、装饰等到outerRingGroup
ship.add(outerRingGroup);

// 内环组
const innerRingGroup = new THREE.Group();
// 添加内环元素
ship.add(innerRingGroup);

// 中央轴（不旋转）
ship.add(centralAxis);

// 辐条（不旋转，因为它们连接到轴）
ship.add(spokes);

// 防护盾和推进器
ship.add(shield);
ship.add(thrusters);

// 动画时
function animate() {
  outerRingGroup.rotation.x += 0.001;
  innerRingGroup.rotation.x -= 0.0015; // 反向旋转，更有视觉趣味
}
```

好，让我开始写最终的代码。

对于内部视图，我会创建一个完全不同的场景或者切换相机位置和视角。当用户点击"进入飞船"按钮时：
- 相机淡出
- 移动到环内部位置
- 显示内部场景（程序化生成的走廊、窗户、远处的小镇）
- 淡入

内部场景设计：
- 一个环形走廊（巨大的CylinderGeometry，从内部看）
- 上方是"天空"（环对面），可以看到对面的城镇和灯光
- 两侧有建筑物（程序化生成的盒子组合）
- 中央有道路、河流
- 灯光从特定方向照射（模拟环上的"太阳"）

让我开始最终的实现：

我需要决定内部视图的实现方式。最简单的方法是：
1. 创建一个独立的内部场景（除了飞船）
2. 切换时改变主场景和相机

或者：
1. 在同一场景中，将相机移动到环内部
2. 显示环内部的细节（建筑、河流等）

第二种方法更优雅，因为可以在环转动时看到外部世界。但需要环内部有可看的内容。

让我用方法2，在创建环时就同时创建内部内容。但要注意，从内部看时，环对面看起来应该是"天空"。

具体实现：
- 环面用透明的（或半透明的）材质
- 在环内部添加：地面（环内表面）、建筑、灯光
- 当切换到内部视图时，相机进入环内

或者，我可以用方法1，创建独立的内部场景，这样自由度更高。让我用方法1，但保留一些视觉连续性。

实际上，让我做一个混合方案：
- 外部视图：完整飞船，可以从任何角度看
- 内部视图：进入环内部，相机位于环内表面附近，可以看到环对面（远处）和环内的建筑、灯光

为了让内部视图更精彩，我会单独构建一个内部场景，使用更细致的几何体。

好，让我开始写代码。我会保证：
1. 视觉冲击力强
2. 交互流畅
3. 信息丰富
4. 代码质量高

开始：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>极光号 · 世代飞船 Aurora</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;700&family=JetBrains+Mono:wght@300;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #030610;
    --bg-2: #0a1226;
    --fg: #e8f4ff;
    --muted: #6e8aa8;
    --accent: #5feec5;
    --accent-2: #ff8c69;
    --accent-3: #c4a3ff;
    --warning: #ffb86b;
    --card: rgba(8, 16, 32, 0.7);
    --border: rgba(95, 238, 197, 0.2);
    --glow: 0 0 30px rgba(95, 238, 197, 0.3);
  }
  
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { height: 100%; overflow: hidden; }
  body {
    background: #000;
    color: var(--fg);
    font-family: 'Space Grotesk', -apple-system, sans-serif;
    user-select: none;
  }
  
  #canvas-container {
    position: fixed;
    inset: 0;
    z-index: 1;
  }
  
  canvas { display: block; }
  
  /* === 加载界面 === */
  #loader {
    position: fixed;
    inset: 0;
    background: radial-gradient(ellipse at center, var(--bg-2) 0%, var(--bg) 70%, #000 100%);
    z-index: 100;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: opacity 1.2s ease;
  }
  
  #loader.hidden { opacity: 0; pointer-events: none; }
  
  .loader-logo {
    width: 80px;
    height: 80px;
    border: 2px solid var(--accent);
    border-radius: 50%;
    border-right-color: transparent;
    animation: spin 1.5s linear infinite;
    margin-bottom: 30px;
    box-shadow: var(--glow);
  }
  
  @keyframes spin { to { transform: rotate(360deg); } }
  
  .loader-title {
    font-size: 14px;
    letter-spacing: 8px;
    color: var(--accent);
    margin-bottom: 8px;
    text-transform: uppercase;
  }
  
  .loader-subtitle {
    font-size: 11px;
    color: var(--muted);
    letter-spacing: 2px;
    font-family: 'JetBrains Mono', monospace;
  }
  
  .loader-progress {
    width: 200px;
    height: 1px;
    background: rgba(95, 238, 197, 0.2);
    margin-top: 40px;
    overflow: hidden;
  }
  
  .loader-progress::after {
    content: '';
    display: block;
    width: 30%;
    height: 100%;
    background: var(--accent);
    animation: progress 2s ease infinite;
  }
  
  @keyframes progress {
    0% { transform: translateX(-100%); }
    100% { transform: translateX(400%); }
  }
  
  /* === UI 顶部 === */
  header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 10;
    padding: 20px 30px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: linear-gradient(180deg, rgba(0,0,0,0.6) 0%, transparent 100%);
    pointer-events: none;
  }
  
  .brand {
    pointer-events: auto;
  }
  
  .brand-title {
    font-size: 22px;
    font-weight: 700;
    letter-spacing: 4px;
    color: var(--fg);
  }
  
  .brand-title span {
    color: var(--accent);
  }
  
  .brand-subtitle {
    font-size: 10px;
    letter-spacing: 3px;
    color: var(--muted);
    font-family: 'JetBrains Mono', monospace;
    margin-top: 2px;
  }
  
  .mission-info {
    pointer-events: auto;
    text-align: right;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    color: var(--muted);
    line-height: 1.8;
  }
  
  .mission-info span { color: var(--accent); }
  
  /* === 左侧信息面板 === */
  .info-panel {
    position: fixed;
    left: 30px;
    top: 50%;
    transform: translateY(-50%);
    z-index: 10;
    width: 280px;
    background: var(--card);
    backdrop-filter: blur(20px);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 24px;
    pointer-events: auto;
    transition: opacity 0.4s, transform 0.4s;
  }
  
  .info-panel h3 {
    font-size: 11px;
    letter-spacing: 3px;
    color: var(--accent);
    margin-bottom: 16px;
    text-transform: uppercase;
    font-weight: 500;
  }
  
  .info-row {
    display: flex;
    justify-content: space-between;
    padding: 8px 0;
    border-bottom: 1px solid rgba(95, 238, 197, 0.08);
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
  }
  
  .info-row:last-child { border-bottom: none; }
  
  .info-label { color: var(--muted); }
  .info-value { color: var(--fg); }
  
  /* === 底部控制栏 === */
  .controls {
    position: fixed;
    bottom: 30px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    display: flex;
    gap: 12px;
    background: var(--card);
    backdrop-filter: blur(20px);
    border: 1px solid var(--border);
    border-radius: 50px;
    padding: 8px;
    pointer-events: auto;
  }
  
  .ctrl-btn {
    background: transparent;
    border: 1px solid transparent;
    color: var(--muted);
    padding: 10px 18px;
    border-radius: 50px;
    cursor: pointer;
    font-family: 'Space Grotesk', sans-serif;
    font-size: 12px;
    letter-spacing: 1.5px;
    transition: all 0.3s;
    text-transform: uppercase;
  }
  
  .ctrl-btn:hover {
    color: var(--fg);
    background: rgba(95, 238, 197, 0.08);
  }
  
  .ctrl-btn.active {
    background: var(--accent);
    color: var(--bg);
    border-color: var(--accent);
    box-shadow: 0 0 20px rgba(95, 238, 197, 0.4);
  }
  
  /* === 右侧 HUD === */
  .hud {
    position: fixed;
    right: 30px;
    top: 50%;
    transform: translateY(-50%);
    z-index: 10;
    width: 240px;
    pointer-events: auto;
  }
  
  .hud-block {
    background: var(--card);
    backdrop-filter: blur(20px);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 18px;
    margin-bottom: 12px;
  }
  
  .hud-title {
    font-size: 10px;
    letter-spacing: 3px;
    color: var(--accent);
    margin-bottom: 12px;
    text-transform: uppercase;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .hud-title::before {
    content: '';
    width: 6px;
    height: 6px;
    background: var(--accent);
    border-radius: 50%;
    box-shadow: 0 0 10px var(--accent);
  }
  
  .bar-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
  }
  
  .bar-label { 
    color: var(--muted); 
    width: 50px;
    flex-shrink: 0;
  }
  
  .bar-track {
    flex: 1;
    height: 3px;
    background: rgba(95, 238, 197, 0.1);
    overflow: hidden;
    border-radius: 2px;
  }
  
  .bar-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--accent), var(--accent-2));
    transition: width 0.5s ease;
  }
  
  .bar-value {
    color: var(--fg);
    width: 40px;
    text-align: right;
    flex-shrink: 0;
  }
  
  /* === 视图模式标识 === */
  .view-mode {
    position: fixed;
    top: 80px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    letter-spacing: 4px;
    color: var(--muted);
    text-transform: uppercase;
    opacity: 0;
    transition: opacity 0.3s;
  }
  
  .view-mode.show { opacity: 1; }
  
  .view-mode span { color: var(--accent); }
  
  /* === 提示 === */
  .hint {
    position: fixed;
    bottom: 100px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    color: var(--muted);
    font-size: 11px;
    letter-spacing: 2px;
    font-family: 'JetBrains Mono', monospace;
    opacity: 0.6;
    text-align: center;
  }
  
  .hint kbd {
    background: rgba(95, 238, 197, 0.1);
    border: 1px solid var(--border);
    padding: 2px 8px;
    border-radius: 3px;
    margin: 0 4px;
    color: var(--accent);
    font-family: inherit;
    font-size: 10px;
  }
  
  /* === Toast === */
  .toast {
    position: fixed;
    top: 80px;
    right: 30px;
    z-index: 100;
    background: var(--card);
    backdrop-filter: blur(20px);
    border: 1px solid var(--accent);
    border-radius: 4px;
    padding: 12px 20px;
    color: var(--fg);
    font-size: 12px;
    letter-spacing: 1px;
    box-shadow: 0 0 30px rgba(95, 238, 197, 0.3);
    transform: translateX(120%);
    transition: transform 0.4s ease;
  }
  
  .toast.show { transform: translateX(0); }
  
  /* === 内部场景特殊UI === */
  body.interior .info-panel { opacity: 0; transform: translateY(-50%) translateX(-120%); }
  body.interior .hud { opacity: 0; transform: translateY(-50%) translateX(120%); }
  body.interior .hint { display: none; }
  
  .room-list {
    position: fixed;
    right: 30px;
    top: 100px;
    z-index: 10;
    width: 220px;
    background: var(--card);
    backdrop-filter: blur(20px);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 16px;
    opacity: 0;
    transform: translateX(120%);
    transition: all 0.4s;
    pointer-events: none;
  }
  
  body.interior .room-list {
    opacity: 1;
    transform: translateX(0);
    pointer-events: auto;
  }
  
  .room-list h3 {
    font-size: 10px;
    letter-spacing: 3px;
    color: var(--accent);
    margin-bottom: 12px;
    text-transform: uppercase;
  }
  
  .room-item {
    display: block;
    width: 100%;
    text-align: left;
    background: transparent;
    border: none;
    color: var(--muted);
    padding: 8px 10px;
    cursor: pointer;
    font-family: 'Space Grotesk', sans-serif;
    font-size: 12px;
    border-radius: 3px;
    transition: all 0.2s;
    border-left: 2px solid transparent;
  }
  
  .room-item:hover {
    color: var(--fg);
    background: rgba(95, 238, 197, 0.05);
  }
  
  .room-item.active {
    color: var(--accent);
    border-left-color: var(--accent);
    background: rgba(95, 238, 197, 0.08);
  }
  
  /* === 响应式 === */
  @media (max-width: 768px) {
    .info-panel, .hud { display: none; }
    .controls { padding: 6px; gap: 6px; }
    .ctrl-btn { padding: 8px 12px; font-size: 10px; }
    header { padding: 15px; }
    .brand-title { font-size: 16px; letter-spacing: 2px; }
    .mission-info { font-size: 9px; }
  }
  
  /* === 角标 === */
  .corner-deco {
    position: fixed;
    width: 30px;
    height: 30px;
    z-index: 5;
    pointer-events: none;
  }
  
  .corner-deco.tl { top: 70px; left: 20px; border-top: 1px solid var(--accent); border-left: 1px solid var(--accent); }
  .corner-deco.tr { top: 70px; right: 20px; border-top: 1px solid var(--accent); border-right: 1px solid var(--accent); }
  .corner-deco.bl { bottom: 90px; left: 20px; border-bottom: 1px solid var(--accent); border-left: 1px solid var(--accent); }
  .corner-deco.br { bottom: 90px; right: 20px; border-bottom: 1px solid var(--accent); border-right: 1px solid var(--accent); }
</style>
</head>
<body>

<!-- 加载界面 -->
<div id="loader">
  <div class="loader-logo"></div>
  <div class="loader-title">AURORA</div>
  <div class="loader-subtitle">INITIALIZING GENERATION SHIP</div>
  <div class="loader-progress"></div>
</div>

<!-- Canvas 容器 -->
<div id="canvas-container"></div>

<!-- 顶部 -->
<header>
  <div class="brand">
    <div class="brand-title">极光<span>·AURORA</span></div>
    <div class="brand-subtitle">GENERATION SHIP · TAU CETI MISSION</div>
  </div>
  <div class="mission-info">
    航行时间 <span id="mtime">00:00:00</span><br>
    距离地球 <span id="mdist">0.000 AU</span>
  </div>
</header>

<!-- 视图模式 -->
<div class="view-mode" id="viewMode">
  当前视图: <span id="viewModeText">外观全景</span>
</div>

<!-- 左侧信息面板 -->
<aside class="info-panel" id="infoPanel">
  <h3>飞船规格</h3>
  <div class="info-row"><span class="info-label">舰级</span><span class="info-value">Lancer-7</span></div>
  <div class="info-row"><span class="info-label">总长度</span><span class="info-value">12.4 km</span></div>
  <div class="info-row"><span class="info-label">外环直径</span><span class="info-value">4.8 km</span></div>
  <div class="info-row"><span class="info-label">内环直径</span><span class="info-value">2.8 km</span></div>
  <div class="info-row"><span class="info-label">船员</span><span class="info-value">2,144</span></div>
  <div class="info-row"><span class="info-label">世代</span><span class="info-value">第 6 代</span></div>
  <div class="info-row"><span class="info-label">航行年数</span><span class="info-value">158 yr</span></div>
  <div class="info-row"><span class="info-label">推进系统</span><span class="info-value">聚变-反物质</span></div>
</aside>

<!-- 右侧 HUD -->
<div class="hud">
  <div class="hud-block">
    <div class="hud-title">环舱状态</div>
    <div class="bar-row">
      <span class="bar-label">外环</span>
      <div class="bar-track"><div class="bar-fill" style="width: 92%"></div></div>
      <span class="bar-value">92%</span>
    </div>
    <div class="bar-row">
      <span class="bar-label">内环</span>
      <div class="bar-track"><div class="bar-fill" style="width: 88%"></div></div>
      <span class="bar-value">88%</span>
    </div>
    <div class="bar-row">
      <span class="bar-label">生态</span>
      <div class="bar-track"><div class="bar-fill" style="width: 76%"></div></div>
      <span class="bar-value">76%</span>
    </div>
    <div class="bar-row">
      <span class="bar-label">生命</span>
      <div class="bar-track"><div class="bar-fill" style="width: 84%"></div></div>
      <span class="bar-value">84%</span>
    </div>
  </div>
  
  <div class="hud-block">
    <div class="hud-title">航向数据</div>
    <div class="info-row"><span class="info-label">航速</span><span class="info-value">0.087 c</span></div>
    <div class="info-row"><span class="info-label">目标</span><span class="info-value">Tau Ceti e</span></div>
    <div class="info-row"><span class="info-label">距离</span><span class="info-value">11.9 ly</span></div>
    <div class="info-row"><span class="info-label">预计抵达</span><span class="info-value">90 yr</span></div>
  </div>
</div>

<!-- 内部房间导航 -->
<div class="room-list" id="roomList">
  <h3>内部区域</h3>
  <button class="room-item active" data-room="bridge">指挥中枢</button>
  <button class="room-item" data-room="axis">中央通道</button>
  <button class="room-item" data-room="outerRing">外环居住区</button>
  <button class="room-item" data-room="innerRing">内环农业区</button>
  <button class="room-item" data-room="engine">引擎舱</button>
  <button class="room-item" data-room="shield">防护盾舱</button>
</div>

<!-- 底部控制 -->
<div class="controls">
  <button class="ctrl-btn active" id="btnExterior">外观</button>
  <button class="ctrl-btn" id="btnInterior">内部</button>
  <button class="ctrl-btn" id="btnRotate">自转</button>
  <button class="ctrl-btn" id="btnReset">复位</button>
</div>

<!-- 操作提示 -->
<div class="hint">
  <kbd>拖拽</kbd> 旋转视角 · <kbd>滚轮</kbd> 缩放 · <kbd>右键</kbd> 平移
</div>

<!-- 角标装饰 -->
<div class="corner-deco tl"></div>
<div class="corner-deco tr"></div>
<div class="corner-deco bl"></div>
<div class="corner-deco br"></div>

<!-- Toast 提示 -->
<div class="toast" id="toast"></div>

<!-- Three.js -->
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
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';

// === 全局状态 ===
let scene, camera, renderer, composer, controls;
let ship, starField, nebula;
let interiorScene, interiorCamera;
let currentMode = 'exterior';
let autoRotate = true;
let clock = new THREE.Clock();
let startTime = Date.now();

// === 初始化 ===
function init() {
  // 场景
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2(0x000511, 0.0008);
  
  // 相机
  camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 5000);
  camera.position.set(120, 50, 120);
  
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;
  document.getElementById('canvas-container').appendChild(renderer.domElement);
  
  // 后期处理
  composer = new EffectComposer(renderer);
  composer.addPass(new RenderPass(scene, camera));
  const bloom = new UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 0.6, 0.5, 0.85);
  composer.addPass(bloom);
  
  // 控制
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.minDistance = 30;
  controls.maxDistance = 600;
  controls.autoRotate = false;
  controls.autoRotateSpeed = 0.3;
  
  // 光照
  const ambient = new THREE.AmbientLight(0x223344, 0.4);
  scene.add(ambient);
  
  const sunLight = new THREE.DirectionalLight(0xfff4e0, 1.2);
  sunLight.position.set(200, 100, 100);
  scene.add(sunLight);
  
  const rimLight = new THREE.DirectionalLight(0x5feec5, 0.5);
  rimLight.position.set(-100, -50, -200);
  scene.add(rimLight);
  
  // 星空
  starField = createStarField(5000);
  scene.add(starField);
  
  // 星云
  nebula = createNebula();
  scene.add(nebula);
  
  // 飞船
  ship = createAuroraShip();
  scene.add(ship);
  
  // 内部场景
  interiorScene = createInteriorScene();
  
  // 事件
  window.addEventListener('resize', onResize);
  setupUI();
  
  // 隐藏加载
  setTimeout(() => {
    document.getElementById('loader').classList.add('hidden');
    showToast('极光号已就绪 · 拖拽视角探索');
  }, 2000);
}

// === 飞船构建 ===
function createAuroraShip() {
  const ship = new THREE.Group();
  ship.name = 'Aurora';
  
  // === 外环（主居住环）===
  const outerRing = createLivingRing({
    radius: 60,
    tubeRadius: 10,
    tubeSegments: 24,
    radialSegments: 96,
    baseColor: 0x3a5868,
    accentColor: 0x5feec5,
    windowCount: 120,
    windowColor: 0xffd88a
  });
  outerRing.name = 'outerRing';
  ship.add(outerRing);
  
  // === 内环（次居住环）===
  const innerRing = createLivingRing({
    radius: 35,
    tubeRadius: 7,
    tubeSegments: 20,
    radialSegments: 80,
    baseColor: 0x3a4a68,
    accentColor: 0xff8c69,
    windowCount: 80,
    windowColor: 0xff9a6a
  });
  innerRing.name = 'innerRing';
  ship.add(innerRing);
  
  // === 中央轴 ===
  const axisGroup = createCentralAxis();
  ship.add(axisGroup);
  
  // === 辐条 ===
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    // 外环辐条
    const outerSpoke = createSpoke(60, 35, 5, angle, 0x4a6a85);
    ship.add(outerSpoke);
  }
  
  // 内环到轴的辐条
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2 + Math.PI / 6;
    const innerSpoke = createSpoke(35, 5, 4, angle, 0x4a6a85);
    ship.add(innerSpoke);
  }
  
  // === 前部防护盾 ===
  const shield = createForwardShield();
  shield.position.x = 90;
  ship.add(shield);
  
  // === 推进器集群 ===
  const thrusterCluster = createThrusters();
  thrusterCluster.position.x = -75;
  ship.add(thrusterCluster);
  
  // === 装饰细节 ===
  // 标识灯
  for (let i = 0; i < 4; i++) {
    const angle = i * Math.PI / 2 + Math.PI / 4;
    const lightGeo = new THREE.SphereGeometry(0.5, 8, 8);
    const lightMat = new THREE.MeshBasicMaterial({ color: 0xff4466 });
    const light = new THREE.Mesh(lightGeo, lightMat);
    light.position.set(0, Math.cos(angle) * 60, Math.sin(angle) * 60);
    light.userData.isBlinker = true;
    light.userData.blinkPhase = i;
    ship.add(light);
  }
  
  // 飞船编号
  const numberGeo = new THREE.PlaneGeometry(20, 4);
  const canvas = document.createElement('canvas');
  canvas.width = 256;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  ctx.fillStyle = '#5feec5';
  ctx.font = 'bold 40px monospace';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText('AURORA-7', 128, 32);
  const tex = new THREE.CanvasTexture(canvas);
  const numberMat = new THREE.MeshBasicMaterial({ map: tex, transparent: true, opacity: 0.8 });
  const number = new THREE.Mesh(numberGeo, numberMat);
  number.position.set(0, 0, 70);
  number.rotation.y = Math.PI;
  ship.add(number);
  
  ship.position.set(0, 0, 0);
  return ship;
}

function createLivingRing(opts) {
  const group = new THREE.Group();
  const { radius, tubeRadius, tubeSegments, radialSegments, baseColor, accentColor, windowCount, windowColor } = opts;
  
  // 主环体（用TorusGeometry）
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, tubeSegments, radialSegments);
  
  // 给环加顶点着色变化
  const colors = new Float32Array(ringGeo.attributes.position.count * 3);
  const positions = ringGeo.attributes.position;
  for (let i = 0; i < positions.count; i++) {
    const y = positions.getY(i);
    const factor = (y + tubeRadius) / (tubeRadius * 2);
    // 顶部颜色浅，底部深
    colors[i*3] = 0.2 + factor * 0.15;
    colors[i*3+1] = 0.3 + factor * 0.2;
    colors[i*3+2] = 0.4 + factor * 0.25;
  }
  ringGeo.setAttribute('color', new THREE.BufferAttribute(colors, 3));
  
  const ringMat = new THREE.MeshStandardMaterial({
    vertexColors: true,
    color: baseColor,
    metalness: 0.6,
    roughness: 0.5,
    flatShading: false
  });
  const ring = new THREE.Mesh(ringGeo, ringMat);
  group.add(ring);
  
  // 内层结构（让环看起来更复杂）
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    const panelGeo = new THREE.BoxGeometry(0.5, 2, tubeRadius * 1.6);
    const panelMat = new THREE.MeshStandardMaterial({
      color: 0x2a3540,
      metalness: 0.8,
      roughness: 0.4
    });
    const panel = new THREE.Mesh(panelGeo, panelMat);
    panel.position.set(
      Math.cos(angle) * (radius + tubeRadius + 1),
      Math.sin(angle) * (radius + tubeRadius + 1),
      0
    );
    panel.rotation.z = angle;
    panel.lookAt(0, 0, 0);
    group.add(panel);
  }
  
  // 窗户灯光
  const windowGeo = new THREE.PlaneGeometry(0.4, 0.2);
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const winMat = new THREE.MeshBasicMaterial({
      color: windowColor,
      transparent: true,
      opacity: 0.7 + Math.random() * 0.3
    });
    const win = new THREE.Mesh(windowGeo, winMat);
    const offset = tubeRadius * 0.95;
    win.position.set(
      Math.cos(angle) * (radius + offset),
      Math.sin(angle) * (radius + offset),
      (Math.random() - 0.5) * tubeRadius
    );
    win.rotation.z = angle - Math.PI / 2;
    win.rotation.y = Math.PI / 2;
    // 朝向轴心外侧
    win.lookAt(0, 0, win.position.z);
    group.add(win);
  }
  
  // 内侧发光带
  const innerGlowGeo = new THREE.TorusGeometry(radius - tubeRadius * 0.7, 0.3, 8, radialSegments);
  const innerGlowMat = new THREE.MeshBasicMaterial({
    color: accentColor,
    transparent: true,
    opacity: 0.6
  });
  const innerGlow = new THREE.Mesh(innerGlowGeo, innerGlowMat);
  group.add(innerGlow);
  
  // 外侧发光带
  const outerGlowGeo = new THREE.TorusGeometry(radius + tubeRadius * 0.9, 0.15, 8, radialSegments);
  const outerGlowMat = new THREE.MeshBasicMaterial({
    color: accentColor,
    transparent: true,
    opacity: 0.4
  });
  const outerGlow = new THREE.Mesh(outerGlowGeo, outerGlowMat);
  group.add(outerGlow);
  
  return group;
}

function createCentralAxis() {
  const group = new THREE.Group();
  
  // 主轴
  const axisGeo = new THREE.CylinderGeometry(4, 4, 160, 32);
  const axisMat = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.8,
    roughness: 0.3
  });
  const axis = new THREE.Mesh(axisGeo, axisMat);
  axis.rotation.z = Math.PI / 2;
  group.add(axis);
  
  // 轴上的能量环
  for (let i = -4; i <= 4; i++) {
    if (i === 0) continue;
    const ringGeo = new THREE.TorusGeometry(5, 0.3, 8, 32);
    const ringMat = new THREE.MeshBasicMaterial({
      color: 0x5feec5,
      transparent: true,
      opacity: 0.8
    });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.rotation.y = Math.PI / 2;
    ring.position.x = i * 18;
    group.add(ring);
  }
  
  // 轴向管道
  for (let i = 0; i < 4; i++) {
    const angle = i * Math.PI / 2 + Math.PI / 4;
    const pipeGeo = new THREE.CylinderGeometry(0.4, 0.4, 160, 8);
    const pipeMat = new THREE.MeshStandardMaterial({
      color: 0x4a5a68,
      metalness: 0.7,
      roughness: 0.4
    });
    const pipe = new THREE.Mesh(pipeGeo, pipeMat);
    pipe.rotation.z = Math.PI / 2;
    pipe.position.set(0, Math.cos(angle) * 4.5, Math.sin(angle) * 4.5);
    group.add(pipe);
  }
  
  // 中央指挥舱
  const bridgeGeo = new THREE.SphereGeometry(8, 32, 16);
  const bridgeMat = new THREE.MeshStandardMaterial({
    color: 0x3a4858,
    metalness: 0.7,
    roughness: 0.3,
    emissive: 0x5feec5,
    emissiveIntensity: 0.1
  });
  const bridge = new THREE.Mesh(bridgeGeo, bridgeMat);
  group.add(bridge);
  
  // 指挥舱窗户
  for (let i = 0; i < 16; i++) {
    const angle = (i / 16) * Math.PI * 2;
    const winGeo = new THREE.PlaneGeometry(0.8, 0.4);
    const winMat = new THREE.MeshBasicMaterial({
      color: 0xffd88a,
      transparent: true,
      opacity: 0.9
    });
    const win = new THREE.Mesh(winGeo, winMat);
    const r = 8.1;
    win.position.set(0, Math.cos(angle) * r, Math.sin(angle) * r);
    win.lookAt(0, win.position.y * 2, win.position.z * 2);
    win.rotateY(Math.PI);
    group.add(win);
  }
  
  return group;
}

function createSpoke(outerR, innerR, thickness, angle, color) {
  const group = new THREE.Group();
  const length = outerR - innerR;
  
  const spokeGeo = new THREE.BoxGeometry(length, thickness, thickness);
  const spokeMat = new THREE.MeshStandardMaterial({
    color: color,
    metalness: 0.7,
    roughness: 0.4
  });
  const spoke = new THREE.Mesh(spokeGeo, spokeMat);
  spoke.position.set(
    0,
    Math.cos(angle) * (innerR + length/2),
    Math.sin(angle) * (innerR + length/2)
  );
  spoke.rotation.z = angle - Math.PI / 2;
  // 调整方向使其垂直于轴
  // 注意：默认box沿X方向，我们希望它沿径向（从轴向外）
  // 让我们重新计算
  spoke.position.set(0, 0, 0);
  spoke.rotation.set(0, 0, 0);
  
  // 使用group来定位
  group.rotation.x = angle;
  
  spoke.position.set(0, innerR + length/2, 0);
  spoke.rotation.z = 0;
  
  group.add(spoke);
  
  // 辐条上的小灯
  for (let i = 1; i < 4; i++) {
    const lightGeo = new THREE.SphereGeometry(0.3, 8, 8);
    const lightMat = new THREE.MeshBasicMaterial({ color: 0x5feec5 });
    const light = new THREE.Mesh(lightGeo, lightMat);
    light.position.set(0, innerR + (length * i / 4), 0);
    group.add(light);
  }
  
  return group;
}

function createForwardShield() {
  const group = new THREE.Group();
  
  // 主防护盾（凹面）
  const shieldGeo = new THREE.SphereGeometry(25, 48, 24, 0, Math.PI * 2, 0, Math.PI / 2);
  const shieldMat = new THREE.MeshStandardMaterial({
    color: 0x6a7a88,
    metalness: 0.85,
    roughness: 0.25,
    side: THREE.DoubleSide,
    emissive: 0x5feec5,
    emissiveIntensity: 0.05
  });
  const shield = new THREE.Mesh(shieldGeo, shieldMat);
  shield.rotation.z = Math.PI / 2;
  group.add(shield);
  
  // 防护盾磁场光晕
  const fieldGeo = new THREE.SphereGeometry(28, 32, 16, 0, Math.PI * 2, 0, Math.PI / 2);
  const fieldMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.15,
    side: THREE.DoubleSide
  });
  const field = new THREE.Mesh(fieldGeo, fieldMat);
  field.rotation.z = Math.PI / 2;
  group.add(field);
  
  // 防护盾支撑
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    const supportGeo = new THREE.CylinderGeometry(0.6, 0.6, 18, 8);
    const supportMat = new THREE.MeshStandardMaterial({
      color: 0x4a5560,
      metalness: 0.8
    });
    const support = new THREE.Mesh(supportGeo, supportMat);
    support.rotation.z = Math.PI / 2;
    support.position.set(-8, Math.cos(angle) * 6, Math.sin(angle) * 6);
    group.add(support);
  }
  
  // 中央传感器
  const sensorGeo = new THREE.SphereGeometry(3, 16, 16);
  const sensorMat = new THREE.MeshBasicMaterial({
    color: 0xff4466,
    transparent: true,
    opacity: 0.9
  });
  const sensor = new THREE.Mesh(sensorGeo, sensorMat);
  sensor.position.x = -2;
  group.add(sensor);
  
  // 传感器光环
  const haloGeo = new THREE.RingGeometry(4, 5, 32);
  const haloMat = new THREE.MeshBasicMaterial({
    color: 0xff4466,
    transparent: true,
    opacity: 0.5,
    side: THREE.DoubleSide
  });
  const halo = new THREE.Mesh(haloGeo, haloMat);
  halo.rotation.y = Math.PI / 2;
  halo.position.x = -2;
  group.add(halo);
  
  return group;
}

function createThrusters() {
  const group = new THREE.Group();
  
  // 4个主推进器
  for (let i = 0; i < 4; i++) {
    const angle = (i / 4) * Math.PI * 2 + Math.PI / 4;
    const thruster = createSingleThruster();
    thruster.position.set(0, Math.cos(angle) * 8, Math.sin(angle) * 8);
    group.add(thruster);
  }
  
  // 中央主推进器
  const mainThrustGeo = new THREE.CylinderGeometry(5, 6, 12, 32);
  const mainThrustMat = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.85,
    roughness: 0.3
  });
  const mainThrust = new THREE.Mesh(mainThrustGeo, mainThrustMat);
  mainThrust.rotation.z = Math.PI / 2;
  mainThrust.position.x = -6;
  group.add(mainThrust);
  
  // 推进器尾焰
  const flameGeo = new THREE.ConeGeometry(4, 30, 32, 1, true);
  const flameMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.6,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending
  });
  const flame = new THREE.Mesh(flameGeo, flameMat);
  flame.rotation.z = -Math.PI / 2;
  flame.position.x = -28;
  flame.userData.isFlame = true;
  group.add(flame);
  
  // 内层火焰
  const innerFlameGeo = new THREE.ConeGeometry(2, 20, 32, 1, true);
  const innerFlameMat = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    transparent: true,
    opacity: 0.8,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending
  });
  const innerFlame = new THREE.Mesh(innerFlameGeo, innerFlameMat);
  innerFlame.rotation.z = -Math.PI / 2;
  innerFlame.position.x = -22;
  innerFlame.userData.isFlame = true;
  innerFlame.userData.inner = true;
  group.add(innerFlame);
  
  return group;
}

function createSingleThruster() {
  const group = new THREE.Group();
  
  const bodyGeo = new THREE.CylinderGeometry(2.5, 3, 8, 16);
  const bodyMat = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.85,
    roughness: 0.3
  });
  const body = new THREE.Mesh(bodyGeo, bodyMat);
  body.rotation.z = Math.PI / 2;
  body.position.x = -4;
  group.add(body);
  
  // 喷射口
  const nozzleGeo = new THREE.CylinderGeometry(2, 2.5, 3, 16);
  const nozzleMat = new THREE.MeshStandardMaterial({
    color: 0x4a5560,
    metalness: 0.9,
    roughness: 0.2
  });
  const nozzle = new THREE.Mesh(nozzleGeo, nozzleMat);
  nozzle.rotation.z = Math.PI / 2;
  nozzle.position.x = -9.5;
  group.add(nozzle);
  
  // 火焰
  const flameGeo = new THREE.ConeGeometry(1.8, 14, 16, 1, true);
  const flameMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.5,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending
  });
  const flame = new THREE.Mesh(flameGeo, flameMat);
  flame.rotation.z = -Math.PI / 2;
  flame.position.x = -18;
  flame.userData.isFlame = true;
  group.add(flame);
  
  return group;
}

// === 星空 ===
function createStarField(count) {
  const geometry = new THREE.BufferGeometry();
  const positions = new Float32Array(count * 3);
  const colors = new Float32Array(count * 3);
  const sizes = new Float32Array(count);
  
  for (let i = 0; i < count; i++) {
    // 球面分布
    const r = 800 + Math.random() * 1500;
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    
    positions[i*3] = r * Math.sin(phi) * Math.cos(theta);
    positions[i*3+1] = r * Math.sin(phi) * Math.sin(theta);
    positions[i*3+2] = r * Math.cos(phi);
    
    // 颜色变化
    const colorChoice = Math.random();
    if (colorChoice < 0.7) {
      colors[i*3] = 1; colors[i*3+1] = 1; colors[i*3+2] = 1;
    } else if (colorChoice < 0.85) {
      colors[i*3] = 0.7; colors[i*3+1] = 0.85; colors[i*3+2] = 1;
    } else {
      colors[i*3] = 1; colors[i*3+1] = 0.85; colors[i*3+2] = 0.7;
    }
    
    sizes[i] = Math.random() * 2 + 0.5;
  }
  
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1));
  
  const material = new THREE.PointsMaterial({
    size: 1.5,
    sizeAttenuation: true,
    vertexColors: true,
    transparent: true,
    opacity: 0.9,
    blending: THREE.AdditiveBlending
  });
  
  return new THREE.Points(geometry, material);
}

function createNebula() {
  const group = new THREE.Group();
  
  // 远处的星云背景（用大球面+渐变）
  const canvas = document.createElement('canvas');
  canvas.width = 512;
  canvas.height = 512;
  const ctx = canvas.getContext('2d');
  
  // 渐变星云
  const grad = ctx.createRadialGradient(256, 256, 50, 256, 256, 250);
  grad.addColorStop(0, 'rgba(95, 238, 197, 0.4)');
  grad.addColorStop(0.3, 'rgba(196, 163, 255, 0.2)');
  grad.addColorStop(0.6, 'rgba(255, 140, 105, 0.1)');
  grad.addColorStop(1, 'rgba(0, 0, 0, 0)');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, 512, 512);
  
  // 加点纹理
  for (let i = 0; i < 200; i++) {
    const x = Math.random() * 512;
    const y = Math.random() * 512;
    const r = Math.random() * 3;
    ctx.fillStyle = `rgba(255, 255, 255, ${Math.random() * 0.3})`;
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2);
    ctx.fill();
  }
  
  const tex = new THREE.CanvasTexture(canvas);
  const mat = new THREE.MeshBasicMaterial({
    map: tex,
    side: THREE.BackSide,
    transparent: true,
    opacity: 0.6,
    depthWrite: false
  });
  const geo = new THREE.SphereGeometry(1800, 32, 16);
  const nebula = new THREE.Mesh(geo, mat);
  group.add(nebula);
  
  // 第二个星云
  const canvas2 = document.createElement('canvas');
  canvas2.width = 512;
  canvas2.height = 512;
  const ctx2 = canvas2.getContext('2d');
  const grad2 = ctx2.createRadialGradient(256, 256, 30, 256, 256, 220);
  grad2.addColorStop(0, 'rgba(255, 140, 105, 0.5)');
  grad2.addColorStop(0.4, 'rgba(196, 100, 100, 0.15)');
  grad2.addColorStop(1, 'rgba(0, 0, 0, 0)');
  ctx2.fillStyle = grad2;
  ctx2.fillRect(0, 0, 512, 512);
  
  const tex2 = new THREE.CanvasTexture(canvas2);
  const mat2 = new THREE.MeshBasicMaterial({
    map: tex2,
    side: THREE.BackSide,
    transparent: true,
    opacity: 0.5,
    depthWrite: false
  });
  const geo2 = new THREE.SphereGeometry(1200, 32, 16);
  const nebula2 = new THREE.Mesh(geo2, mat2);
  nebula2.position.set(-500, 200, -800);
  group.add(nebula2);
  
  return group;
}

// === 内部场景 ===
function createInteriorScene() {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x050810);
  scene.fog = new THREE.FogExp2(0x050810, 0.015);
  
  // 光照
  const ambient = new THREE.AmbientLight(0x335577, 0.6);
  scene.add(ambient);
  
  // 主光（模拟"太阳"）
  const sunLight = new THREE.DirectionalLight(0xfff4d0, 0.8);
  sunLight.position.set(0, 50, 0);
  scene.add(sunLight);
  
  // 蓝色辅助光
  const blueLight = new THREE.DirectionalLight(0x5feec5, 0.4);
  blueLight.position.set(20, 0, 30);
  scene.add(blueLight);
  
  // 创建内部环境
  const interior = new THREE.Group();
  scene.add(interior);
  
  return { scene, interior };
}

// 等等，让我重新设计内部场景。我会让内部场景根据用户选择的房间来动态生成。
// 每个房间有独特的视觉特征

let interiorObjects = []; // 当前房间对象

function loadInteriorRoom(roomName) {
  // 清除当前
  interiorScene.interior.clear();
  interiorObjects = [];
  
  switch(roomName) {
    case 'bridge':
      buildBridgeInterior(interiorScene.interior);
      break;
    case 'axis':
      buildAxisInterior(interiorScene.interior);
      break;
    case 'outerRing':
      buildRingInterior(interiorScene.interior, 60, 10, true);
      break;
    case 'innerRing':
      buildRingInterior(interiorScene.interior, 35, 7, false);
      break;
    case 'engine':
      buildEngineInterior(interiorScene.interior);
      break;
    case 'shield':
      buildShieldInterior(interiorScene.interior);
      break;
  }
}

function buildBridgeInterior(group) {
  // 中央指挥室 - 球形空间，有控制台和窗户
  const roomGeo = new THREE.SphereGeometry(20, 32, 24);
  const roomMat = new THREE.MeshStandardMaterial({
    color: 0x1a2530,
    side: THREE.BackSide,
    metalness: 0.5,
    roughness: 0.6
  });
  const room = new THREE.Mesh(roomGeo, roomMat);
  group.add(room);
  
  // 窗户（绕一圈）
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2;
    const winGeo = new THREE.PlaneGeometry(3, 4);
    const winMat = new THREE.MeshBasicMaterial({
      color: 0x000511,
      emissive: 0x5feec5,
      emissiveIntensity: 0.2,
      side: THREE.DoubleSide
    });
    const win = new THREE.Mesh(winGeo, winMat);
    win.position.set(Math.cos(angle) * 19, 0, Math.sin(angle) * 19);
    win.lookAt(0, 0, 0);
    group.add(win);
    
    // 窗框
    const frameGeo = new THREE.BoxGeometry(3.4, 0.2, 0.1);
    const frameMat = new THREE.MeshStandardMaterial({ color: 0x4a5560, metalness: 0.8 });
    const frame = new THREE.Mesh(frameGeo, frameMat);
    frame.position.copy(win.position);
    frame.lookAt(0, 0, 0);
    group.add(frame);
  }
  
  // 中央控制台
  const consoleGeo = new THREE.CylinderGeometry(3, 4, 1.5, 16);
  const consoleMat = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.7,
    roughness: 0.3,
    emissive: 0x5feec5,
    emissiveIntensity: 0.1
  });
  const console = new THREE.Mesh(consoleGeo, consoleMat);
  group.add(console);
  
  // 全息投影
  const holoGeo = new THREE.CylinderGeometry(0.1, 2, 6, 32, 1, true);
  const holoMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.3,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending
  });
  const holo = new THREE.Mesh(holoGeo, holoMat);
  holo.position.y = 3;
  holo.userData.isHolo = true;
  group.add(holo);
  
  // 工作站
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2;
    const stationGeo = new THREE.BoxGeometry(2, 1, 1.2);
    const stationMat = new THREE.MeshStandardMaterial({
      color: 0x1a2530,
      metalness: 0.6,
      roughness: 0.4
    });
    const station = new THREE.Mesh(stationGeo, stationMat);
    station.position.set(Math.cos(angle) * 8, -2, Math.sin(angle) * 8);
    station.lookAt(0, -2, 0);
    group.add(station);
    
    // 屏幕
    const screenGeo = new THREE.PlaneGeometry(1.5, 0.6);
    const screenMat = new THREE.MeshBasicMaterial({
      color: 0x5feec5,
      transparent: true,
      opacity: 0.7
    });
    const screen = new THREE.Mesh(screenGeo, screenMat);
    screen.position.set(Math.cos(angle) * 8, -1.2, Math.sin(angle) * 8);
    screen.lookAt(0, -1.2, 0);
    group.add(screen);
  }
  
  // 地板
  const floorGeo = new THREE.CircleGeometry(18, 32);
  const floorMat = new THREE.MeshStandardMaterial({
    color: 0x0a1520,
    metalness: 0.5,
    roughness: 0.7
  });
  const floor = new THREE.Mesh(floorGeo, floorMat);
  floor.rotation.x = -Math.PI / 2;
  floor.position.y = -3;
  group.add(floor);
  
  // 地板网格线
  const gridHelper = new THREE.GridHelper(36, 18, 0x5feec5, 0x1a3a40);
  gridHelper.position.y = -2.99;
  gridHelper.material.transparent = true;
  gridHelper.material.opacity = 0.3;
  group.add(gridHelper);
  
  // 设置相机初始位置
  if (interiorCamera) {
    interiorCamera.position.set(0, 2, 12);
    interiorCamera.lookAt(0, 0, 0);
  }
}

function buildRingInterior(group, radius, tubeRadius, isOuter) {
  // 创建一个环内部的空间
  // 使用一个TorusGeometry的内表面
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, 32, 64);
  const ringMat = new THREE.MeshStandardMaterial({
    color: isOuter ? 0x2a4a3a : 0x3a4a68,
    side: THREE.BackSide,
    metalness: 0.3,
    roughness: 0.7
  });
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.x = Math.PI / 2; // 让环轴朝向Z
  group.add(ring);
  
  // 内部装饰 - 建筑
  const buildingCount = isOuter ? 30 : 20;
  for (let i = 0; i < buildingCount; i++) {
    const angle = (i / buildingCount) * Math.PI * 2;
    const r = radius - tubeRadius * 0.5 - Math.random() * tubeRadius * 0.4;
    
    const h = 1 + Math.random() * 3;
    const w = 1 + Math.random() * 1.5;
    const buildingGeo = new THREE.BoxGeometry(w, h, w);
    const buildingMat = new THREE.MeshStandardMaterial({
      color: 0x3a4858,
      metalness: 0.5,
      roughness: 0.6
    });
    const building = new THREE.Mesh(buildingGeo, buildingMat);
    building.position.set(
      Math.cos(angle) * r,
      -tubeRadius * 0.7,
      Math.sin(angle) * r
    );
    building.rotation.y = -angle;
    group.add(building);
    
    // 建筑窗户
    const winCount = Math.floor(h);
    for (let j = 0; j < winCount; j++) {
      const winGeo = new THREE.PlaneGeometry(0.2, 0.2);
      const winMat = new THREE.MeshBasicMaterial({
        color: 0xffd88a,
        transparent: true,
        opacity: 0.7 + Math.random() * 0.3
      });
      const win = new THREE.Mesh(winGeo, winMat);
      win.position.copy(building.position);
      win.position.y = -tubeRadius * 0.7 + j + 0.3;
      win.position.x += Math.cos(angle) * 0.51;
      win.position.z += Math.sin(angle) * 0.51;
      win.rotation.y = -angle + Math.PI / 2;
      group.add(win);
    }
  }
  
  // 路径
  const pathGeo = new THREE.TorusGeometry(radius - tubeRadius * 0.8, 0.5, 8, 64);
  const pathMat = new THREE.MeshStandardMaterial({
    color: 0x1a2030,
    metalness: 0.3,
    roughness: 0.8
  });
  const path = new THREE.Mesh(pathGeo, pathMat);
  path.rotation.x = Math.PI / 2;
  path.position.y = -tubeRadius * 0.85;
  group.add(path);
  
  // 灯柱
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2;
    const lampGeo = new THREE.SphereGeometry(0.3, 8, 8);
    const lampMat = new THREE.MeshBasicMaterial({ color: 0xffd88a });
    const lamp = new THREE.Mesh(lampGeo, lampMat);
    lamp.position.set(
      Math.cos(angle) * (radius - tubeRadius * 0.8),
      -tubeRadius * 0.7 + 3,
      Math.sin(angle) * (radius - tubeRadius * 0.8)
    );
    group.add(lamp);
    
    // 光源
    const light = new THREE.PointLight(0xffd88a, 0.3, 8);
    light.position.copy(lamp.position);
    group.add(light);
  }
  
  // 远端的"天空"效果 - 在环对面放发光物体
  const skyGeo = new THREE.TorusGeometry(radius, tubeRadius * 0.7, 16, 64);
  const skyMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.05,
    side: THREE.BackSide
  });
  const sky = new THREE.Mesh(skyGeo, skyMat);
  sky.rotation.x = Math.PI / 2;
  group.add(sky);
  
  // 设置相机
  if (interiorCamera) {
    interiorCamera.position.set(radius - tubeRadius * 0.7, 0, 0);
    interiorCamera.lookAt(0, 5, 0);
  }
}

function buildEngineInterior(group) {
  // 引擎室 - 长长的走廊，尽头是发光的核心
  const corridorGeo = new THREE.CylinderGeometry(8, 8, 50, 32, 1, true);
  const corridorMat = new THREE.MeshStandardMaterial({
    color: 0x1a2030,
    side: THREE.BackSide,
    metalness: 0.6,
    roughness: 0.4
  });
  const corridor = new THREE.Mesh(corridorGeo, corridorMat);
  corridor.rotation.z = Math.PI / 2;
  group.add(corridor);
  
  // 引擎核心
  const coreGeo = new THREE.SphereGeometry(4, 32, 32);
  const coreMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.9
  });
  const core = new THREE.Mesh(coreGeo, coreMat);
  core.position.x = -20;
  core.userData.isCore = true;
  group.add(core);
  
  // 核心光晕
  const haloGeo = new THREE.SphereGeometry(6, 32, 32);
  const haloMat = new THREE.MeshBasicMaterial({
    color: 0x5feec5,
    transparent: true,
    opacity: 0.2
  });
  const halo = new THREE.Mesh(haloGeo, haloMat);
  halo.position.x = -20;
  halo.userData.isHalo = true;
  group.add(halo);
  
  // 核心光源
  const coreLight = new THREE.PointLight(0x5feec5, 2, 30);
  coreLight.position.x = -20;
  group.add(coreLight);
  
  // 走廊环带
  for (let i = -2; i <= 2; i++) {
    const ringGeo = new THREE.TorusGeometry(7.5, 0.2, 8, 32);
    const ringMat = new THREE.MeshBasicMaterial({
      color: 0x5feec5,
      transparent: true,
      opacity: 0.6
    });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.rotation.y = Math.PI / 2;
    ring.position.x = i * 8;
    group.add(ring);
  }
  
  // 管道
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2;
    const pipeGeo = new THREE.CylinderGeometry(0.3, 0.3, 50, 8);
    const pipeMat = new THREE.MeshStandardMaterial({
      color: 0x3a4858,
      metalness: 0.7,
      roughness: 0.5
    });
    const pipe = new THREE.Mesh(pipeGeo, pipeMat);
    pipe.rotation.z = Math.PI / 2;
    pipe.position.set(0, Math.cos(angle) * 6, Math.sin(angle) * 6);
    group.add(pipe);
  }
  
  // 地板网格
  const grid = new THREE.GridHelper(40, 20, 0x5feec5, 0x1a3a40);
  grid.rotation.y = Math.PI / 2;
  grid.position.y = -7.5;
  grid.material.transparent = true;
  grid.material.opacity = 0.3;
  group.add(grid);
  
  if (interiorCamera) {
    interiorCamera.position.set(15, 2, 5);
    interiorCamera.lookAt(-20, 0, 0);
  }
}

function buildAxisInterior(group) {
  // 中央通道 - 长长的零重力通道
  const tubeGeo = new THREE.CylinderGeometry(6, 6, 60, 32, 1, true);
  const tubeMat = new THREE.MeshStandardMaterial({
    color: 0x1a2030,
    side: THREE.BackSide,
    metalness: 0.5,
    roughness: 0.5
  });
  const tube = new THREE.Mesh(tubeGeo, tubeMat);
  tube.rotation.z = Math.PI / 2;
  group.add(tube);
  
  // 通道环
  for (let i = -5; i <= 5; i++) {
    const ringGeo = new THREE.TorusGeometry(5.5, 0.15, 8, 32);
    const ringMat = new THREE.MeshBasicMaterial({
      color: 0x5feec5,
      transparent: true,
      opacity: 0.5
    });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.rotation.y = Math.PI / 2;
    ring.position.x = i * 5;
    group.add(ring);
  }
  
  // 管道
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    const pipeGeo = new THREE.CylinderGeometry(0.2, 0.2, 60, 8);
    const pipeMat = new THREE.MeshStandardMaterial({
      color: 0x3a4858,
      metalness: 0.7,
      roughness: 0.5
    });
    const pipe = new THREE.Mesh(pipeGeo, pipeMat);
    pipe.rotation.z = Math.PI / 2;
    pipe.position.set(0, Math.cos(angle) * 5, Math.sin(angle) * 5);
    group.add(pipe);
  }
  
  // 漂浮的物体
  for (let i = 0; i < 20; i++) {
    const objGeo = new THREE.BoxGeometry(0.4, 0.4, 0.4);
    const objMat = new THREE.MeshStandardMaterial({
      color: 0x4a5560,
      metalness: 0.6,
      roughness: 0.4
    });
    const obj = new THREE.Mesh(objGeo, objMat);
    obj.position.set(
      (Math.random() - 0.5) * 50,
      (Math.random() - 0.5) * 4,
      (Math.random() - 0.5) * 4
    );
    obj.userData.isFloating = true;
    obj.userData.floatSpeed = (Math.random() - 0.5) * 0.5;
    group.add(obj);
  }
  
  if (interiorCamera) {
    interiorCamera.position.set(20, 0, 0);
    interiorCamera.lookAt(-10, 0, 0);
  }
}

function buildShieldInterior(group) {
  // 防护盾控制室 - 半球形
  const domeGeo = new THREE.SphereGeometry(15, 32, 16, 0, Math.PI * 2, 0, Math.PI / 2);
  const domeMat = new THREE.MeshStandardMaterial({
    color: 0x1a2530,
    side: THREE.BackSide,
    metalness: 0.5,
    roughness: 0.5
  });
  const dome = new THREE.Mesh(domeGeo, domeMat);
  group.add(dome);
  
  // 地板
  const floorGeo = new THREE.CircleGeometry(15, 32);
  const floorMat = new THREE.MeshStandardMaterial({
    color: 0x0a1520,
    metalness: 0.5,
    roughness: 0.7
  });
  const floor = new THREE.Mesh(floorGeo, floorMat);
  floor.rotation.x = -Math.PI / 2;
  floor.position.y = -0.01;
  group.add(floor);
  
  // 中央控制台
  const consoleGeo = new THREE.CylinderGeometry(1, 1.5, 1, 16);
  const consoleMat = new THREE.MeshStandardMaterial({
    color: 0x2a3540,
    metalness: 0.7,
    roughness: 0.3,
    emissive: 0xff4466,
    emissiveIntensity: 0.15
  });
  const console = new THREE.Mesh(consoleGeo, consoleMat);
  group.add(console);
  
  // 防护盾显示器（半透明球）
  const displayGeo = new THREE.SphereGeometry(5, 32, 16);
  const displayMat = new THREE.MeshBasicMaterial({
    color: 0xff4466,
    transparent: true,
    opacity: 0.15,
    blending: THREE.AdditiveBlending
  });
  const display = new THREE.Mesh(displayGeo, displayMat);
  display.position.y = 4;
  display.userData.isShieldDisplay = true;
  group.add(display);
  
  // 周围工作站
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2;
    const stationGeo = new THREE.BoxGeometry(2, 1, 1);
    const stationMat = new THREE.MeshStandardMaterial({
      color: 0x1a2530,
      metalness: 0.6,
      roughness: 0.4
    });
    const station = new THREE.Mesh(stationGeo, stationMat);
    station.position.set(Math.cos(angle) * 8, 0, Math.sin(angle) * 8);
    station.lookAt(0, 0, 0);
    group.add(station);
  }
  
  // 网格
  const grid = new THREE.GridHelper(30, 15, 0xff4466, 0x3a2030);
  grid.material.transparent = true;
  grid.material.opacity = 0.3;
  group.add(grid);
  
  if (interiorCamera) {
    interiorCamera.position.set(0, 3, 10);
    interiorCamera.lookAt(0, 1, 0);
  }
}

// === UI 设置 ===
function setupUI() {
  document.getElementById('btnExterior').addEventListener('click', () => switchMode('exterior'));
  document.getElementById('btnInterior').addEventListener('click', () => switchMode('interior'));
  document.getElementById('btnRotate').addEventListener('click', toggleAutoRotate);
  document.getElementById('btnReset').addEventListener('click', resetView);
  
  document.querySelectorAll('.room-item').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.room-item').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      const room = btn.dataset.room;
      loadInteriorRoom(room);
      const names = {
        bridge: '指挥中枢',
        axis: '中央通道',
        outerRing: '外环居住区',
        innerRing: '内环农业区',
        engine: '引擎舱',
        shield: '防护盾舱'
      };
      showToast(`进入 ${names[room]}`);
    });
  });
}

function switchMode(mode) {
  if (currentMode === mode) return;
  currentMode = mode;
  
  document.getElementById('btnExterior').classList.toggle('active', mode === 'exterior');
  document.getElementById('btnInterior').classList.toggle('active', mode === 'interior');
  
  document.body.classList.toggle('interior', mode === 'interior');
  
  const vm = document.getElementById('viewMode');
  vm.classList.add('show');
  
  if (mode === 'exterior') {
    document.getElementById('viewModeText').textContent = '外观全景';
    showToast('已切换至外观视图');
    
    // 切换控件
    controls.object = camera;
    controls.target.set(0, 0, 0);
    camera.position.set(120, 50, 120);
    controls.enableRotate = true;
    controls.enablePan = true;
    controls.minDistance = 30;
    controls.maxDistance = 600;
  } else {
    document.getElementById('viewModeText').textContent = '内部探索';
    
    // 创建内部相机
    if (!interiorCamera) {
      interiorCamera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
    }
    interiorCamera.position.set(0, 2, 12);
    interiorCamera.lookAt(0, 0, 0);
    
    // 默认加载指挥中枢
    loadInteriorRoom('bridge');
    
    // 切换控件到内部相机
    controls.object = interiorCamera;
    controls.target.set(0, 0, 0);
    controls.minDistance = 1;
    controls.maxDistance = 80;
    
    showToast('进入飞船内部 · 选择区域探索');
  }
  
  setTimeout(() => vm.classList.remove('show'), 2000);
}

function toggleAutoRotate() {
  autoRotate = !autoRotate;
  document.getElementById('btnRotate').classList.toggle('active', autoRotate);
  showToast(autoRotate ? '自动旋转已开启' : '自动旋转已关闭');
}

function resetView() {
  if (currentMode === 'exterior') {
    camera.position.set(120, 50, 120);
    controls.target.set(0, 0, 0);
  } else {
    interiorCamera.position.set(0, 2, 12);
    controls.target.set(0, 0, 0);
  }
  showToast('视角已复位');
}

function showToast(msg) {
  const toast = document.getElementById('toast');
  toast.textContent = msg;
  toast.classList.add('show');
  clearTimeout(toast._timer);
  toast._timer = setTimeout(() => toast.classList.remove('show'), 2500);
}

// === 动画循环 ===
function animate() {
  requestAnimationFrame(animate);
  
  const delta = clock.getDelta();
  const elapsed = clock.getElapsedTime();
  
  // 飞船旋转
  if (ship && currentMode === 'exterior') {
    if (autoRotate) {
      // 让飞船整体在Y轴上慢慢转
      ship.rotation.y += delta * 0.05;
    }
    
    // 环旋转（绕X轴，模拟人造重力）
    const outerRing = ship.getObjectByName('outerRing');
    const innerRing = ship.getObjectByName('innerRing');
    if (outerRing) outerRing.rotation.x += delta * 0.15;
    if (innerRing) innerRing.rotation.x -= delta * 0.22;
    
    // 推进器尾焰闪烁
    ship.traverse(obj => {
      if (obj.userData.isFlame) {
        const scale = 0.8 + Math.sin(elapsed * 8 + (obj.userData.inner ? 1 : 0)) * 0.2;
        obj.scale.set(scale, scale, 1);
        obj.material.opacity = 0.5 + Math.sin(elapsed * 5) * 0.1;
      }
      if (obj.userData.isBlinker) {
        const phase = obj.userData.blinkPhase || 0;
        const visible = Math.sin(elapsed * 2 + phase) > 0;
        obj.visible = visible;
      }
    });
  }
  
  // 内部场景动画
  if (currentMode === 'interior' && interiorScene) {
    interiorScene.interior.traverse(obj => {
      if (obj.userData.isHolo) {
        obj.rotation.y += delta;
        obj.material.opacity = 0.25 + Math.sin(elapsed * 2) * 0.05;
      }
      if (obj.userData.isCore) {
        const scale = 1 + Math.sin(elapsed * 3) * 0.05;
        obj.scale.set(scale, scale, scale);
      }
      if (obj.userData.isHalo) {
        const scale = 1 + Math.sin(elapsed * 2) * 0.1;
        obj.scale.set(scale, scale, scale);
        obj.material.opacity = 0.15 + Math.sin(elapsed * 2) * 0.05;
      }
      if (obj.userData.isFloating) {
        obj.position.x += Math.sin(elapsed + obj.position.y) * delta * obj.userData.floatSpeed;
        obj.rotation.x += delta * 0.2;
        obj.rotation.y += delta * 0.15;
      }
      if (obj.userData.isShieldDisplay) {
        obj.rotation.y += delta * 0.5;
        obj.material.opacity = 0.1 + Math.sin(elapsed * 1.5) * 0.05;
      }
    });
  }
  
  // 星空缓慢移动（飞船航行感觉）
  if (starField) {
    starField.rotation.y += delta * 0.005;
  }
  
  controls.update();
  
  // 渲染
  if (currentMode === 'exterior') {
    composer.render();
  } else {
    renderer.render(interiorScene.scene, interiorCamera);
  }
  
  // 更新HUD
  updateHUD();
}

function updateHUD() {
  // 任务时间
  const elapsed = (Date.now() - startTime) / 1000;
  const h = Math.floor(elapsed / 3600);
  const m = Math.floor((elapsed % 3600) / 60);
  const s = Math.floor(elapsed % 60);
  document.getElementById('mtime').textContent = 
    `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
  
  // 距离
  const dist = (elapsed * 0.0001).toFixed(3);
  document.getElementById('mdist').textContent = `${dist} AU`;
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  if (interiorCamera) {
    interiorCamera.aspect = window.innerWidth / window.innerHeight;
    interiorCamera.updateProjectionMatrix();
  }
  renderer.setSize(window.innerWidth, window.innerHeight);
  composer.setSize(window.innerWidth, window.innerHeight);
}

// === 启动 ===
init();
animate();
</script>
</body>
</html>
```

让我审视一下这段代码，确保它能正常运行：

1. importmap正确
2. Three.js模块导入正确
3. 飞船构建函数：需要注意辐条的几何构造

让我特别检查辐条的几何：

辐条应该是从外环（半径60）连接到内环或中央轴（半径5）。每个辐条应该：
- 在某个角度上
- 沿径向方向（从中心向外）

我的实现：
```javascript
function createSpoke(outerR, innerR, thickness, angle, color) {
  const group = new THREE.Group();
  const length = outerR - innerR;
  
  const spokeGeo = new THREE.BoxGeometry(length, thickness, thickness);
  // ...
  group.rotation.x = angle;
  spoke.position.set(0, innerR + length/2, 0);
  
  group.add(spoke);
  return group;
}
```

这是正确的。Group绕X轴旋转angle角度，spoke沿Y轴正方向放置，length从innerR到outerR。这样辐条就在YZ平面（垂直于X轴）上从中心向外辐射。

但是这里有个问题：飞船主轴沿X方向。环也应该垂直于X轴，即环在YZ平面。

我的TorusGeometry默认在XY平面，需要旋转。

让我看看createLivingRing:
```javascript
const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, ...);
const ring = new THREE.Mesh(ringGeo, ringMat);
group.add(ring);
```

TorusGeometry默认是在XY平面（环面朝向Z方向）。我需要旋转它，让环面朝向X方向（即环在YZ平面）：

```javascript
ring.rotation.y = Math.PI / 2;
```

或者：
```javascript
ring.rotation.x = Math.PI / 2;  // 这样环在XZ平面
```

让我想想：
- 默认TorusGeometry：环在XY平面，环轴沿Z方向
- 我希望环在YZ平面，环轴沿X方向

所以需要把环绕Y轴旋转90度：
```javascript
ring.rotation.y = Math.PI / 2;
```

这样环就从XY平面转到YZ平面了。

但是这样旋转会让环上的窗户位置也变化。让我检查窗户的位置计算：
```javascript
win.position.set(
  Math.cos(angle) * (radius + offset),
  Math.sin(angle) * (radius + offset),
  (Math.random() - 0.5) * tubeRadius
);
```

这里窗户在XY平面分布。如果环旋转了，窗户也应该跟着旋转，所以应该把窗户也加到旋转后的Group里。

我的实现是把所有元素加到group，然后group作为整体。但是group本身没有旋转。我需要对group应用旋转，或者直接对每个元素应用相同的旋转。

让我重新设计：把所有元素加到group中，然后让group旋转ring.rotation.y = Math.PI/2：

实际上，我应该让整个group在创建时就旋转：

```javascript
function createLivingRing(opts) {
  const group = new THREE.Group();
  // ... 添加元素 ...
  group.rotation.y = Math.PI / 2;  // 让环在YZ平面
  return group;
}
```

但是这样的话，环本身的rotation.x（用于旋转动画）会出问题。

更好的做法是嵌套：
- 外层Group（不旋转，包含整个环和它的位置/方向）
- 内层Group（旋转动画）

让我重新设计：

```javascript
function createLivingRing(opts) {
  const outerGroup = new THREE.Group(); // 静态，定位环
  const spinGroup = new THREE.Group();  // 旋转动画
  outerGroup.add(spinGroup);
  
  // 添加所有元素到spinGroup
  // ...
  
  // 让环面朝向X方向（环在YZ平面）
  outerGroup.rotation.y = Math.PI / 2;
  
  outerGroup.name = ... // 标记
  return outerGroup;
}
```

但是这样在animate里访问的就不是group本身了。

让我简化：
- 把所有元素都设计成在YZ平面（即X是环的轴向）
- 直接放置元素在YZ平面，不需要旋转Group

让我重新设计窗户位置：
```javascript
// 窗户在YZ平面（垂直于X轴）
win.position.set(
  (Math.random() - 0.5) * tubeRadius * 0.5,  // X方向偏移
  Math.cos(angle) * (radius + offset),       // Y方向
  Math.sin(angle) * (radius + offset)        // Z方向
);
```

但这样环本身也需要重新设计...

让我用最简单的方法：
- 创建TorusGeometry
- 旋转它（绕Y轴90度）让它面朝X方向
- 然后所有元素都基于这个旋转后的环来放置

实际上，更简单的方法是：让Group作为整体旋转，然后在动画时通过group.rotation.x来旋转。但要注意，外层Group的rotation.y = Math.PI/2 已经设置了，旋转动画应该作用在内层。

最简单的方法：在创建函数里直接旋转所有几何体或者使用嵌套Group。

让我用一个不同的策略：

让所有几何体在创建时就考虑正确的方向。具体来说：
- TorusGeometry默认在XY平面，环轴沿Z
- 我希望环轴沿X，所以让Torus绕Y旋转90度
- 但这会让坐标系混淆

更直接的方法：直接修改TorusGeometry的旋转，并把所有元素的位置都基于"环在YZ平面"来计算：

```javascript
function createLivingRing(opts) {
  const group = new THREE.Group();
  const { radius, tubeRadius, ... } = opts;
  
  // 环面在YZ平面，轴向沿X
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, tubeSegments, radialSegments);
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.y = Math.PI / 2;  // 旋转环，使其在YZ平面
  group.add(ring);
  
  // 窗户在YZ平面
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const win = new THREE.Mesh(windowGeo, winMat);
    // 位置在YZ平面
    const y = Math.cos(angle) * (radius + offset);
    const z = Math.sin(angle) * (radius + offset);
    win.position.set(0, y, z);
    // 让窗户朝向外面（径向）
    win.lookAt(0, y * 2, z * 2);
    // 或者更简单：让窗户的"上"方向是X
    win.up.set(1, 0, 0);
    win.lookAt(0, y * 100, z * 100);
    group.add(win);
  }
  
  // ...
  return group;
}
```

实际上，最简单的方法：所有元素都添加到一个spinGroup里，然后spinGroup绕X轴旋转动画。所有的位置都基于"环在YZ平面"的假设。

让我重写代码，更清晰地组织：

```javascript
function createLivingRing(opts) {
  const ringGroup = new THREE.Group(); // 整个环
  const spinGroup = new THREE.Group(); // 旋转部分
  ringGroup.add(spinGroup);
  ringGroup.userData.spinGroup = spinGroup;
  
  // === 主环体 ===
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, tubeSegments, radialSegments);
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.y = Math.PI / 2;  // 让环面朝向X方向（环在YZ平面）
  spinGroup.add(ring);
  
  // === 窗户 ===
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const win = new THREE.Mesh(windowGeo, winMat);
    // YZ平面分布
    win.position.set(
      0,
      Math.cos(angle) * (radius + tubeRadius * 0.9),
      Math.sin(angle) * (radius + tubeRadius * 0.9)
    );
    // 朝向外面
    win.lookAt(0, win.position.y * 2, win.position.z * 2);
    // 但Plane默认是面朝Z的，所以lookAt之后会面向那个方向
    // 我们希望窗户面朝外（径向向外）
    // lookAt的中心点是原点的反方向，即窗户位置乘以2
    // 这样窗户就面朝外
    spinGroup.add(win);
  }
  
  // ...其他元素同理...
  
  return ringGroup;
}
```

然后动画时：
```javascript
const outerRing = ship.getObjectByName('outerRing');
if (outerRing && outerRing.userData.spinGroup) {
  outerRing.userData.spinGroup.rotation.x += delta * 0.15;
}
```

这样设计是正确的。但是PlaneGeometry的lookAt会有点tricky。让我用另一种方法：直接设置旋转。

对于Plane，我希望它的法向量（默认是Z方向）指向径向（即从原点指向窗户位置的方向）。

```javascript
win.position.set(0, y, z);
// 计算窗户应该面向的方向（径向向外）
const dir = new THREE.Vector3(0, y, z).normalize();
// 让窗户的Z方向（默认法向）对准dir
// 这可以用quaternion
const quat = new THREE.Quaternion();
quat.setFromUnitVectors(new THREE.Vector3(0, 0, 1), dir);
win.quaternion.copy(quat);
```

但是这样窗户的"上"方向不确定。让我用lookAt：

```javascript
win.lookAt(0, 0, 0);  // 看向中心，但这是反方向
// 应该是看向外侧
win.lookAt(win.position.x * 2, win.position.y * 2, win.position.z * 2);
```

不对，lookAt是让物体看向那个点。如果窗户在(0, y, z)，看向(0, 2y, 2z)就是看向外侧（远离原点），窗户的+Z方向（法向）就指向外侧。

但是这样窗户的"上"方向（+Y）会被设置为世界的+Y方向，可能导致窗户旋转不正确。

让我直接试试看，如果效果不好再调整。

实际上，对于环上的窗户，我们希望它沿着环的切线方向"宽"，沿着径向方向"高"。或者更简单：让窗户就像贴在环表面一样。

我用一个更简单的方法：把窗户作为一个小Box（或者其他3D几何），这样它没有"朝向"问题，反正从外面都能看到。

让我用Box来代替Plane作为窗户：

```javascript
const windowGeo = new THREE.BoxGeometry(0.4, 0.2, 0.1);
```

然后放置时让它面朝外：

```javascript
// 窗户位置（径向外侧）
const x = 0;
const y = Math.cos(angle) * (radius + tubeRadius * 0.95);
const z = Math.sin(angle) * (radius + tubeRadius * 0.95);
win.position.set(x, y, z);

// 让窗户的Z方向（厚度方向）指向径向
// 默认Box的Z方向是厚度方向（0.1）
// 所以我们让窗户的+Z方向指向径向外
// 这等价于让窗户"看向"远离原点的方向
const lookTarget = new THREE.Vector3(0, y * 2, z * 2);
win.lookAt(lookTarget);
```

实际上，Box不需要朝向，因为从所有方向都能看到。但是为了窗户看起来正确（面朝外），我还是设置一下。

OK，让我专注于完成代码。让我用一个更简洁的设计：

```javascript
function createLivingRing(opts) {
  const { radius, tubeRadius, ... } = opts;
  
  const ringGroup = new THREE.Group();  // 整体
  const spinGroup = new THREE.Group();  // 旋转部分
  ringGroup.add(spinGroup);
  ringGroup.userData.spinGroup = spinGroup;
  
  // === 主环体 ===
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, 24, 96);
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.y = Math.PI / 2;  // 环面朝向X方向
  spinGroup.add(ring);
  
  // === 外侧装饰条 ===
  // 让我们用TorusGeometry作为装饰，但是更细
  const outerBandGeo = new THREE.TorusGeometry(radius + tubeRadius, 0.3, 8, 96);
  const outerBand = new THREE.Mesh(outerBandGeo, outerBandMat);
  outerBand.rotation.y = Math.PI / 2;
  spinGroup.add(outerBand);
  
  // === 内侧发光带 ===
  const innerBandGeo = new THREE.TorusGeometry(radius - tubeRadius, 0.4, 8, 96);
  const innerBand = new THREE.Mesh(innerBandGeo, innerBandMat);
  innerBand.rotation.y = Math.PI / 2;
  spinGroup.add(innerBand);
  
  // === 窗户 ===
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const y = Math.cos(angle) * (radius + tubeRadius * 0.9);
    const z = Math.sin(angle) * (radius + tubeRadius * 0.9);
    
    const winGeo = new THREE.BoxGeometry(0.6, 0.3, 0.1);
    const win = new THREE.Mesh(winGeo, winMat);
    win.position.set(0, y, z);
    // 让窗户的Z方向（默认）指向径向外
    win.lookAt(0, y * 2, z * 2);
    // 现在窗户的+Z方向指向外侧
    // 但是窗户是一个Box，从外面看时Z方向是0.1厚度
    // 实际上我希望窗户从径向外侧看时是一个矩形
    // BoxGeometry(0.6, 0.3, 0.1) 的X是0.6（宽），Y是0.3（高），Z是0.1（厚）
    // lookAt让+Z指向远处，所以从外面看时，我们看到的是0.6 x 0.3的面
    // 这是正确的！
    spinGroup.add(win);
  }
  
  return ringGroup;
}
```

OK，这样设计是正确的。但是有一个问题：如果环和窗户在spinGroup里，spinGroup旋转动画时，整个环（包括窗户）一起旋转。但环本身是TorusGeometry绕Y轴旋转Math.PI/2，所以环的法向已经是X方向了，环本身就在YZ平面。spinGroup绕X轴旋转，就是让环绕自己的中心轴旋转，这是正确的。

OK让我开始写最终代码。我会精简一些，但保证视觉效果。

让我也重新考虑辐条。辐条从环连接到中央轴，方向应该是径向的。在我的坐标系里：
- 飞船主轴沿X
- 环在YZ平面
- 径向方向是YZ平面上从原点向外的方向

辐条位置：
- 在某个角度angle上
- 从中央轴（半径4-5）连接到环（半径60）
- 长度约55-56
- 沿径向方向（YZ平面）

```javascript
function createSpoke(outerR, innerR, thickness, angle) {
  const group = new THREE.Group();
  const length = outerR - innerR;
  
  // 辐条主体（沿Y方向，长度length）
  const spokeGeo = new THREE.BoxGeometry(thickness, length, thickness);
  const spoke = new THREE.Mesh(spokeGeo, spokeMat);
  spoke.position.set(0, innerR + length/2, 0);  // 在Y方向上从innerR到outerR
  
  // 旋转到正确角度（绕X轴）
  group.rotation.x = angle;
  group.add(spoke);
  
  // 辐条上的灯
  for (let i = 1; i < 4; i++) {
    const lightGeo = new THREE.SphereGeometry(0.4, 8, 8);
    const lightMat = new THREE.MeshBasicMaterial({ color: 0x5feec5 });
    const light = new THREE.Mesh(lightGeo, lightMat);
    light.position.set(0, innerR + (length * i / 4), 0);
    group.add(light);
  }
  
  return group;
}
```

辐条不需要随环旋转（它连接到轴和环两个固定点）。所以辐条不放在spinGroup里。这是合理的。

OK，让我写最终的代码。我也需要确保几个细节：
1. 飞船的rotation.y在autoRotate时增加
2. 但是环的spinGroup.rotation.x也增加，所以环上的窗户会跟着转
3. 但是辐条不转，所以辐条看起来是固定的，环在辐条之间转动 - 这是合理的，因为环就像车轮，辐条连接到轴

等等，这其实有问题。如果环真的在转，那辐条和环不能固定连接啊。在现实里：
- 环（圆环管子）旋转产生重力
- 但辐条是从轴到环的"通道"
- 辐条要么也旋转（与环同步），要么是分离的结构

小说里极光号的描述：辐条是旋转的，与环一起旋转，所以它们实际上是一起转的。但是从外部观察者的角度看，整体看起来在转。

为了视觉效果，我让外环+辐条都旋转（同步），但是辐条只在特定角度出现，这样看起来像有6-8根辐条一起转。

或者更简单：辐条不旋转，只有环本身（窗户等）旋转。这在物理上不准确，但视觉上没问题。

为了简化，我让所有元素都加到spinGroup里，包括辐条。但是辐条是从外环到内环/轴，所以它们在spinGroup里就会随环一起转。这是合理的。

让我重新设计：把辐条也放在spinGroup里。但这样辐条会穿过环（如果环也有装饰）。让我把辐条作为单独的连接结构，与环同步旋转：

实际上，最简单的方案：把所有旋转的部分都放在ship对象的旋转动画里（即让整个ship绕X轴旋转）。但这会让防护盾和推进器也跟着转，这不合理（防护盾和推进器应该是固定的）。

最终方案：
- 飞船本身不绕X轴转
- 但环结构（外环+辐条+内环）作为一个group绕X轴转
- 防护盾、推进器、中央轴不转

让我重新组织ship的结构：

```javascript
function createAuroraShip() {
  const ship = new THREE.Group();
  
  // === 旋转部分 ===
  const rotatingPart = new THREE.Group();
  rotatingPart.name = 'rotatingPart';
  ship.add(rotatingPart);
  
  // 外环（包括辐条）
  rotatingPart.add(createLivingRing(outerOpts));
  rotatingPart.add(createLivingRing(innerOpts));
  
  // 辐条
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    rotatingPart.add(createSpoke(60, 5, 5, angle));
  }
  
  // === 固定部分 ===
  // 中央轴
  ship.add(createCentralAxis());
  
  // 前部防护盾
  ship.add(createForwardShield());
  
  // 推进器
  ship.add(createThrusters());
  
  return ship;
}
```

然后在动画里：
```javascript
const rotatingPart = ship.getObjectByName('rotatingPart');
if (rotatingPart) {
  rotatingPart.rotation.x += delta * 0.05;  // 慢慢转
}
```

OK，这样设计更好。但是注意：如果环本身有自己的rotation.y = Math.PI/2（用于让环在YZ平面），那么环旋转动画不能直接覆盖这个值。

让我重新设计：环本身不需要旋转Math.PI/2，因为我会直接构造几何体使其在YZ平面。或者，把环放在一个group里，group本身没有旋转，但group里的所有元素的位置都基于"环在YZ平面"来计算。

具体来说：
- TorusGeometry默认在XY平面
- 我把它旋转Math.PI/2绕Y轴，让它在YZ平面
- 这个旋转是应用于单个mesh的，不影响group的rotation
- 所以group的rotation.x仍然是0，可以用于旋转动画

这样就没问题了。我会把所有元素都放在rotatingPart这个group里，让group整体绕X轴旋转。

让我开始最终代码。

```javascript
function createLivingRing(opts) {
  const group = new THREE.Group();
  const { radius, tubeRadius, baseColor, accentColor, windowCount, windowColor } = opts;
  
  // === 主环体 ===
  const ringGeo = new THREE.TorusGeometry(radius, tubeRadius, 24, 96);
  const ringMat = new THREE.MeshStandardMaterial({
    color: baseColor,
    metalness: 0.6,
    roughness: 0.5
  });
  const ring = new THREE.Mesh(ringGeo, ringMat);
  ring.rotation.y = Math.PI / 2; // 让环面朝向X方向（环在YZ平面）
  group.add(ring);
  
  // === 外侧发光带 ===
  const outerBandGeo = new THREE.TorusGeometry(radius + tubeRadius * 0.95, 0.2, 8, 96);
  const outerBandMat = new THREE.MeshBasicMaterial({
    color: accentColor,
    transparent: true,
    opacity: 0.6
  });
  const outerBand = new THREE.Mesh(outerBandGeo, outerBandMat);
  outerBand.rotation.y = Math.PI / 2;
  group.add(outerBand);
  
  // === 内侧发光带 ===
  const innerBandGeo = new THREE.TorusGeometry(radius - tubeRadius * 0.95, 0.3, 8, 96);
  const innerBand = new THREE.Mesh(innerBandGeo, outerBandMat.clone());
  innerBand.rotation.y = Math.PI / 2;
  group.add(innerBand);
  
  // === 窗户 ===
  const windowGeo = new THREE.BoxGeometry(0.6, 0.25, 0.1);
  for (let i = 0; i < windowCount; i++) {
    const angle = (i / windowCount) * Math.PI * 2;
    const r = radius + tubeRadius * 0.9;
    const y = Math.cos(angle) * r;
    const z = Math.sin(angle) * r;
    
    const winMat = new THREE.MeshBasicMaterial({
      color: windowColor,
      transparent: true,
      opacity: 0.7 + Math.random() * 0.3
    });
    const win = new THREE.Mesh(windowGeo, winMat);
    win.position.set(0, y, z);
    // 让窗户朝向径向外
    win.lookAt(0, y * 2, z * 2);
    group.add(win);
  }
  
  // === 外侧结构条 ===
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    const r = radius + tubeRadius + 0.5;
    const y = Math.cos(angle) * r;
    const z = Math.sin(angle) * r;
    
    const strutGeo = new THREE.BoxGeometry(0.8, 0.4, tubeRadius * 1.8);
    const strutMat = new THREE.MeshStandardMaterial({
      color: 0x2a3540,
      metalness: 0.8,
      roughness: 0.4
    });
    const strut = new THREE.Mesh(strutGeo, strutMat);
    strut.position.set(0, y, z);
    strut.rotation.x = angle; // 让strut沿着环的切线方向延伸
    // 等等，strut是BoxGeometry(0.8, 0.4, tubeRadius * 1.8)，X是0.8宽，Y是0.4高，Z是tubeRadius*1.8长
    // 我们希望strut沿着环径向方向（即从内到外）延伸，长度是tubeRadius*1.8
    // 即strut的Z方向应该指向径向
    // 但是strut的位置在(0, y, z)，径向方向是(0, y, z)的方向
    // 所以让strut.lookAt(0, y * 2, z * 2)，让+Z指向径向外
    strut.lookAt(0, y * 2, z * 2);
    group.add(strut);
  }
  
  return group;
}
```

OK，这看起来不错。让我继续完成代码。

我注意到，BoxGeometry的lookAt会让它的+Z方向指向目标。所以如果我希望strut沿着径向（即strut的长度方向是径向），我应该让strut的Z方向（长度tubeRadius*1.8）指向径向。

```javascript
const strutGeo = new THREE.BoxGeometry(0.4, 0.4, tubeRadius * 1.8);  // X短，Y短，Z长
// ...
strut.lookAt(0, y * 2, z * 2);  // +Z指向径向外
```

但是lookAt会让up方向变化，可能不是我想要的。如果up方向不确定，strut可能旋转到奇怪的角度。

让我直接计算strut的quaternion：
```javascript
const dir = new THREE.Vector3(0, y, z).normalize();
const quat = new THREE.Quaternion();
quat.setFromUnitVectors(new THREE.Vector3(0, 0, 1), dir);
strut.quaternion.copy(quat);
```

这样strut的+Z方向就是径向方向了。但是strut的+Y方向（默认世界Y）可能不正确，因为这取决于lookAt的实现。

实际上，对于BoxGeometry来说，从外面看是一个矩形，朝向不重要，因为从任何方向看都是一个矩形。让我简化，不要担心朝向，只要位置对就行。

让我用一个更简单的方法：直接把strut设置在YZ平面的位置，不加旋转，让它就是一个简单的盒子：

```javascript
const strut = new THREE.Mesh(strutGeo, strutMat);
strut.position.set(0, y, z);
// 不需要旋转
```

这样strut就是世界坐标系下的盒子。如果我希望它沿着环径向方向延伸，那需要旋转。如果只是装饰，不旋转也可以。

让我简化设计，去掉strut，只用窗户和发光带来装饰环。这样代码更简单。

让我精简一下代码，确保它运行：

OK，让我写最终版本。我会保持代码精炼但视觉效果好。

我也需要确认：所有PlanarGeometry或PlaneGeometry的元素都有正确的朝向。PlaneGeometry默认在XY平面，法向沿+Z方向。

让我也想想内部场景：
- 内部场景使用一个独立的Scene对象
- 当切换到内部模式时，使用interiorCamera渲染
- OrbitControls的object需要切换到interiorCamera

但是OrbitControls的object在创建时设置，切换可能需要重新创建controls。

让我看看是否可以直接修改controls.object：
- 在Three.js最新版本里，OrbitControls的object属性可以修改
- 但更安全的方法是重新创建controls

让我用一个简单的方法：始终使用同一个camera，只是切换它所在的位置和看向的目标。当切换到内部模式时，相机移动到内部场景的位置。

但是这样需要把内部场景和外部场景合并，或者动态加载/卸载。

更简单：在主场景里，当切换到内部时，隐藏飞船，显示内部场景（作为另一个group添加到主场景）。

让我用这种方法：
- 主场景始终包含所有内容
- 外部模式：显示ship，隐藏interiorGroup
- 内部模式：隐藏ship，显示interiorGroup，相机移到interiorGroup内部

这样OrbitControls始终绑定到一个相机，只需要切换controls.target和camera.position。

让我重写：

```javascript
let ship, interiorGroup;  // 都在主scene里
let isInterior = false;

function switchMode(mode) {
  if (mode === 'exterior') {
    ship.visible = true;
    interiorGroup.visible = false;
    // 重置相机
    camera.position.set(120, 50, 120);
    controls.target.set(0, 0, 0);
    controls.minDistance = 30;
    controls.maxDistance = 600;
  } else {
    ship.visible = false;
    interiorGroup.visible = true;
    // 移动相机到内部位置
    loadInteriorRoom('bridge');
    // 相机位置和target由loadInteriorRoom设置
    controls.minDistance = 1;
    controls.maxDistance = 80;
  }
  isInterior = (mode === 'interior');
}
```

但是loadInteriorRoom里设置相机位置时，需要保证controls也更新。OrbitControls的target需要调用controls.target.set，然后controls.update()。

让我整合：

```javascript
function loadInteriorRoom(roomName) {
  // 清除interiorGroup的子对象
  while (interiorGroup.children.length > 0) {
    const obj = interiorGroup.children[0];
    interiorGroup.remove(obj);
    // dispose materials and geometries
  }
  
  // 根据roomName添加新内容
  // 同时设置相机位置
  switch(roomName) {
    case 'bridge':
      buildBridgeInterior(interiorGroup);
      camera.position.set(0, 2, 12);
      controls.target.set(0, 0, 0);
      break;
    // ...
  }
  controls.update();
}
```

这样设计应该可以工作。让我开始写最终代码。

我会让代码尽量精炼，但保证视觉效果出色。我会用以下结构：

1. HTML + CSS（精美UI）
2. importmap
3. Module script:
   - init()
   - createAuroraShip() - 外部飞船
   - createLivingRing() - 创建居住环
   - createCentralAxis() - 创建中央轴
   - createSpoke() - 创建辐条
   - createForwardShield() - 创建防护盾
   - createThrusters() - 创建推进器
   - createStarField() - 创建星空
   - createNebula() - 创建星云
   - 内部场景构建函数们
   - setupUI()
   - switchMode()
   - loadInteriorRoom()
   - animate()
   - 启动

让我现在写最终的代码。我会让它紧凑但功能完整。

最后，我需要注意：
1. 当切换到内部场景时，bloom效果可能太强，需要调整或者禁用
2. 内部场景的光照要充足
3. 内部场景的相机位置要合理，让用户能看到内容
4. 房间切换时要有过渡效果（淡入淡出）

让我也加一个淡入淡出效果：用一个黑色overlay div，切换时fade to black, change scene, fade from black。

OK，开始写代码。我会尽量精炼，但保证所有功能都实现。

我注意到我前面的代码已经基本完成了。让我整合一下，做最后的调整：

最后检查：
1. 资源管理：dispose几何体和材质在loadInteriorRoom清除时
2. 性能：用BufferGeometry，避免过多drawcalls
3. 视觉：bloom、tone mapping、星云背景

让我开始写最终代码：

我看到我的代码已经接近完成了。让我做最后的精炼和整合，然后输出最终结果。

我注意到，我需要让"内部视图"看起来也很壮观。让我特别精心设计内部视图：
- bridge: 圆形指挥室，全息投影，工作站
- axis: 长长的零重力走廊，漂浮物体
- outerRing: 环内部广阔空间，远处能看到对面的城镇
- innerRing: 类似但小一些
- engine: 引擎室，发光的核心
- shield: 半球形防护盾控制室

每个房间都有独特的视觉风格。

让我写最终代码：

实际上，再思考一下"内部视图"的实现。我之前的设计是：
- 主场景里有ship和interiorGroup
- 切换时显示/隐藏

但是interiorGroup的"位置"在哪里？如果在原点附近，会和ship重叠（虽然ship被隐藏了，但...）。

让我把interiorGroup放在远离ship的地方，比如(1000, 0, 0)，这样不会重叠。然后切换时把相机移到interiorGroup的位置。

但是这样星云、星空都会看起来一样，可能OK。

或者：interiorGroup放在原点附近，因为ship被隐藏了，所以没冲突。这是更简单的方法。

OK，最终方案：interiorGroup放在原点附近，切换时隐藏ship，显示interiorGroup，相机移到内部位置。

好，让我写最终代码。

一个细节：在内部场景里，星空和星云作为背景可能看起来怪异（从飞船内部看星空？）。让我让内部场景有自己的环境（比如用fog遮蔽远处）。

实际上，如果用户在环内部，他们应该能看到远处的"天空"（环对面）和环上的灯光。我的设计中已经考虑了这个。

但是，如果相机在环内，背景仍然是外部场景的星空，这看起来会有点怪。

解决方法：在内部场景里，用一个大的天空盒包裹相机，模拟飞船内部的天空。或者用fog遮蔽远处，让远处的星空看不清。

我用fog的方法：scene.fog = new THREE.FogExp2(color, density)。这会让远处变暗，遮蔽星空。

但是这样整个scene（包括外部ship和星空）都会有fog。我希望fog只在内部模式生效。

解决：动态设置scene.fog。在切换到内部时启用，切换到外部时禁用。

```javascript
function switchMode(mode) {
  // ...
  if (mode === 'interior') {
    scene.fog = new THREE.FogExp2(0x050810, 0.015);
  } else {
    scene.fog = null;  // 或原来的fog
  }
}
```

OK，让我写最终代码。让我组织一下，去掉一些不必要的复杂性：

最终决定：
- 单一scene，包含ship、interiorGroup、星空、星云
- 切换时显示/隐藏
- 动态调整fog
- 相机移动到合适位置
- 切换时淡入淡出

我会用一个overlay div做fade效果。

让我开始写最终代码。我会确保：
1. 视觉效果好
2. 交互流畅
3. 代码质量高

我还需要注意：
- 防止webGL context lost
- 防止内存泄漏（dispose）
- 性能优化

让我开始最终的代码。我会让它简洁但功能完整：

我的代码会比较长（大约1500-2000行），但所有都是必要的。

让我开始：

最终我决定不使用bloom（虽然好看但增加复杂度），用发光材质+additive blending来模拟发光效果。这样可以减少代码量，提高稳定性。

实际上bloom对视觉效果非常重要，特别是对于推进器尾焰、发光带等。让我还是用bloom。

OK，开始最终代码：

让我再思考一下"飞船在宇宙深空航行"的效果。这可以通过：
- 星空缓慢向后移动（相对飞船向后，即朝相机方向，因为飞船是朝X+方向"飞行"的，所以星空应该向X-方向移动）
- 推进器尾焰持续闪烁
- 飞船本身有轻微的浮动效果（上下浮动）

但是要让飞船看起来在航行，最好让相机和飞船一起向前移动，星空向后退。

但OrbitControls会让用户控制相机，所以固定相机更好。让我用以下效果：
- 星空缓慢旋转（模拟飞船在转向，或者宇宙在动）
- 推进器尾焰持续闪烁
- 飞船环旋转（人造重力）

这样足够了。

让我写最终代码：

```javascript
// 星空缓慢移动
starField.rotation.y += delta * 0.005;
starField.rotation.x += delta * 0.002;
```

OK，让我开始写最终代码。