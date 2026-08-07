用户要求创建一个 3D 数据可视化地球仪，包含多种功能和视觉效果，使用 Three.js、真实地球纹理、importmap + ESM（ES Modules）、单一 HTML 文件。还需要提供完整代码和运行方式/测试说明。

功能要求详细分析：
1. 3D 地球：
   - 真实地球纹理（大陆、海洋）
   - 可旋转交互（OrbitControls）
   - 大气光晕效果（自定义着色器或 sprite/图层）
2. 数据可视化层：
   - 主要城市标记点（发光点） - 使用 Points 或 Sprites/Mesh
   - 城市之间的航线（弧线，有流动动画） - 使用 QuadraticBezierCurve3 或类似，以及纹理偏移或着色器动画
   - 国家/地区热力着色 - 这个在 3D 球体上做热力着色比较复杂。可以通过在球体表面覆盖一层半透明的着色层，或者通过画点或自定义着色器来实现。为了简单且真实，可以在地球外面套一层略大的半透明球，根据经纬度绘制热力或使用顶点颜色/纹理。或者用一个网格来近似。
3. 交互：
   - 鼠标拖拽旋转（OrbitControls）
   - 滚轮缩放（OrbitControls）
   - 点击城市显示信息弹窗（Raycaster）
   - 悬停航线显示航班信息（Raycaster）
4. 控制面板：
   - 切换不同数据指标（GDP、人口、航班量等）- 简单的 HTML UI 覆盖层
   - 时间轴播放，展示数据随时间变化 - 简单的 HTML UI 覆盖层，带滑块或播放按钮
5. 视觉效果：
   - 地球自转动画（在 render loop 中旋转）
   - 日夜分界线（Day/Night terminator）- 可以使用自定义 ShaderMaterial 或光照（定向光）来实现。
   - 城市灯光效果，尤其是夜晚区域 - 需要夜晚灯光纹理（Earth lights map），结合 ShaderMaterial 根据法线与光照方向（太阳方向）的夹角混合白天和夜晚纹理，并在夜晚显示城市灯光。
6. 技术要求：
   - Three.js（通过 CDN + importmap 引入）
   - 真实地球纹理（可以使用 NASA 的 Blue Marble 或者 three.js 官方 examples 里的纹理 URL）
   - importmap + ESM
   - 单一 HTML 文件

需要准备的资源（CDN链接）：
- 地球白天纹理：https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg 或 threejs 官方示例的：https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg
- 地球夜晚灯光纹理：https://unpkg.com/three-globe/example/img/earth-night.jpg 或 https://threejs.org/examples/textures/planets/earth_lights_2048.png
- 地球法线/高光纹理（可选）：https://unpkg.com/three-globe/example/img/earth-topology.png
由于 CORS 和跨域问题，使用 unpkg 或者 threejs.org 提供的可靠静态资源。

架构设计：
- HTML 结构：包含 `#container` 用于渲染 Three.js，包含 `#ui-panel` 用于控制面板，包含 `#tooltip` 用于悬停提示，包含 `#modal` 用于点击弹窗。
- importmap:
  ```html
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>
  ```
- 着色器（ShaderMaterial）用于地球：
  实现白天/黑夜混合（day/night），大气光晕（Fresnel）也可以在这里或者单独一个 Sphere。
  Earth Shader:
  - uniforms: dayTexture, nightTexture, sunDirection (由于地球自转，太阳方向在视图空间中固定或者随着时间变化？通常是太阳在世界空间固定，地球自转)
  - vertexShader: 计算法线、世界坐标
  - fragmentShader: 根据 dot(normal, sunDirection) 决定混合比例，再加上菲涅尔大气效果。
- 大气层（Atmosphere）：
  在地球外面画一个稍大的球（BackSide），使用 Fresnel 着色器产生蓝色光晕。
- 城市标记点：
  根据经纬度计算球面上的三维坐标：
  `x = R * cos(lat) * cos(lon)`
  `y = R * sin(lat)`
  `z = -R * cos(lat) * sin(lon)` （具体看坐标系统）
  使用 `Sprite` 或 `Mesh` (CircleGeometry) 并在 raycaster 中检测点击。为了有发光效果，可以用 `SpriteMaterial` 加一个 radial gradient 的 canvas 纹理。
- 航线（Arc）：
  对于两点 A 和 B（在球面上），计算它们之间的大圆弧控制点。可以用 `QuadraticBezierCurve3` 或 `CubicBezierCurve3`，控制点设在球面外（比如高度增加，或者距离中点向外推）。
  使用 `TubeGeometry` 或者 `Line` 来渲染。流动动画可以通过在材质中使用 `dashSize`/`gapSize` 或者自定义着色器，让纹理沿曲线移动（UV offset）。
  悬停检测可以通过把线条变粗一点（用稍宽的 Tube 并透明碰撞检测），或者用 Raycaster 检测 Lines (注意 Line 的 threshold)。
- 热力着色：
  这个比较难在球体上完美实现。要求里说“国家/地区热力着色”，考虑到单一 HTML 文件，可以用一些假数据，在地球上生成一些面（或者点）表示热力。为简化，我们可以把热力着色体现为：根据当前切换的指标（GDP、人口等），改变城市标记点的大小、颜色，以及在地球表面覆盖一层以城市为中心的热力圆（发光透明圆）。或者，使用简单的顶点颜色/纹理更新。考虑到代码复杂度和性能，实现“以主要城市为中心的热力扩散效果”或“点状热力”是可以接受的。也可以简单点，给控制面板加一个“热力图层”开关，开启时在特定区域显示红色/黄色半透明多边形（用 ShapeGeometry 投影到球面）或者直接用半透明的发光球体圈。为了好看，我会在地球上覆盖一层带有特定国家区域的简单模拟数据（用经纬度点画半透明圆形）。

我们细化一下。
因为要在单一文件里写，需要尽可能利用 Three.js 内置功能。

关键点：
1. **Earth Material with Day/Night**:
```glsl
uniform sampler2D dayTexture;
uniform sampler2D nightTexture;
uniform vec3 sunDirection;
varying vec2 vUv;
varying vec3 vNormal;

void main() {
    float intensity = dot(normalize(vNormal), normalize(sunDirection));
    // clamp or smoothstep
    float mixStrength = smoothstep(-0.1, 0.1, intensity);
    vec3 dayColor = texture2D(dayTexture, vUv).rgb;
    vec3 nightColor = texture2D(nightTexture, vUv).rgb;
    vec3 color = mix(nightColor, dayColor, mixStrength);
    gl_FragColor = vec4(color, 1.0);
}
```
但标准的 SphereGeometry 的 normal 是在局部坐标系下的。如果地球在自转，我们需要将 sunDirection 转到局部坐标系，或者在顶点着色器中将 normal 传到世界空间。
```glsl
// vertex
varying vec2 vUv;
varying vec3 vWorldNormal;
void main() {
    vUv = uv;
    vWorldNormal = normalize(mat3(modelMatrix) * normal);
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
// fragment
uniform sampler2D dayTexture;
uniform sampler2D nightTexture;
uniform vec3 sunDirection;
varying vec2 vUv;
varying vec3 vWorldNormal;
void main() {
    float intensity = dot(normalize(vWorldNormal), normalize(sunDirection));
    float mixStrength = smoothstep(-0.2, 0.2, intensity);
    vec3 dayColor = texture2D(dayTexture, vUv).rgb;
    vec3 nightColor = texture2D(nightTexture, vUv).rgb;
    vec3 color = mix(nightColor * 1.5, dayColor, mixStrength); // 夜晚灯光增强
    gl_FragColor = vec4(color, 1.0);
}
```

2. **Atmosphere Glow**:
```glsl
// vertex
varying vec3 vWorldNormal;
varying vec3 vViewDir;
void main() {
    vWorldNormal = normalize(mat3(modelMatrix) * normal);
    vec4 worldPos = modelMatrix * vec4(position, 1.0);
    vViewDir = normalize(cameraPosition - worldPos.xyz);
    gl_Position = projectionMatrix * viewMatrix * worldPos;
}
// fragment
uniform vec3 glowColor;
varying vec3 vWorldNormal;
varying vec3 vViewDir;
void main() {
    float intensity = pow(0.7 - dot(vWorldNormal, vViewDir), 4.0);
    gl_FragColor = vec4(glowColor, 1.0) * intensity;
}
```
渲染时设置 `side: THREE.BackSide` 或者 `FrontSide`，`transparent: true`, `blending: THREE.AdditiveBlending`.

3. **Lat/Lon to Vector3**:
```javascript
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * (Math.PI / 180);
    const theta = (lon + 180) * (Math.PI / 180);
    const x = -radius * Math.sin(phi) * Math.cos(theta);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    const y = radius * Math.cos(phi);
    return new THREE.Vector3(x, y, z);
}
```

4. **Cities Data**:
一些主要城市的经纬度：
北京 (39.9, 116.4), 纽约 (40.7, -74.0), 伦敦 (51.5, -0.1), 东京 (35.6, 139.6), 悉尼 (-33.8, 151.2), 巴黎 (48.8, 2.3), 莫斯科 (55.7, 37.6), 圣保罗 (-23.5, -46.6), 开普敦 (-33.9, 18.4), 洛杉矶 (34.0, -118.2)。
生成点（Sprite），根据缩放和指标改变大小。

5. **Routes (Arcs)**:
对于每个路线，给出起止城市，画一条弧线。
```javascript
const startVec = latLonToVector3(start.lat, start.lon, R);
const endVec = latLonToVector3(end.lat, end.lon, R);
// 中点
const midVec = startVec.clone().add(endVec).multiplyScalar(0.5);
// 向外推，距离越远推得越高
const distance = startVec.distanceTo(endVec);
const altitude = R + distance * 0.3; // 调整高度
midVec.normalize().multiplyScalar(altitude);

const curve = new THREE.QuadraticBezierCurve3(startVec, midVec, endVec);
const points = curve.getPoints(50);
const geometry = new THREE.BufferGeometry().setFromPoints(points);
// 线条材质
const material = new THREE.LineBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0.6 });
const line = new THREE.Line(geometry, material);
```
为了有流动动画，我们可以使用 `TubeGeometry` 和带有 dash 属性的材质，或者自定义 ShaderMaterial 沿管道流动。
自定义 ShaderMaterial 对 Tube：
```glsl
// vertex
varying vec2 vUv;
void main() { vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }
// fragment
uniform float time;
uniform vec3 color;
varying vec2 vUv;
void main() {
    // vUv.x 沿着管道长度方向 0 到 1
    float flow = fract(vUv.x * 5.0 - time * 2.0);
    float alpha = smoothstep(0.8, 1.0, flow); // 头部亮，尾部暗的流星效果
    // 或者: float alpha = step(0.9, flow) * 0.9 + 0.1; 等等
    // 让线基础有一点透明度，流动部分更亮
    float baseAlpha = 0.2;
    float pulse = smoothstep(0.9, 1.0, flow) * 0.8;
    gl_FragColor = vec4(color, baseAlpha + pulse);
}
```
需要注意的是，`TubeGeometry` 的 UV `x` 方向（管子的长度方向）默认是从 0 到 1 分布吗？根据 Three.js 文档，`TubeGeometry` 的 uv.x 对应管子长度的比例（0到1），uv.y 对应圆周。所以这个 Shader 是完美的。

6. **Hover Routes & Click Cities**:
使用 `Raycaster` 检测。
- 对城市：每个城市对应一个 `Sprite` 或者用 `Mesh` 做一个不可见的稍大的 `Sphere` 用于点击（由于 Sprite 的 Raycast 有时不太准，或者太小不容易点中，可以用不可见的大球作为 Hitbox，或者把 Sprite 的 scale 设大点并检测）。
- 对航线：可以把 `Tube` 放进一个数组，开启 Raycaster。注意 `Raycaster` 对 `Mesh` 有效，对 `Line` 需要设置 `line.raycast` 或 `Line2` 才能准确，所以 `TubeGeometry` 生成的 Mesh 是可以直接被射线检测到的！这就很棒了。
悬停航线时，在鼠标位置显示 tooltip。

7. **Control Panel / Time Axis**:
- UI 放一些 HTML 控件（绝对定位）。
- 数据指标：GDP、人口、航班量。点击切换时，遍历城市，更新城市的大小（scale）和颜色（或发光强度）。同时更新热力面。
- 时间轴：一个 `<input type="range">` 和 一个 `Play/Pause` 按钮。时间变化时，可以改变地球上的数据（假装随时间变化，例如根据年份调整城市的 scale，或者让航线数量、流动速度变化）。为简化，随时间变化可以控制一个“太阳方向的角度”（模拟地球相对太阳的自转）或者更新某些模拟数值的显示。题目要求“展示数据随时间变化”，所以我们可以生成几年的假数据（比如 2010 - 2020），时间轴滑动时，根据当前年份更新城市标记的大小、热力强度。同时把年份显示在 UI 上。

8. **Heatmap Layer (国家/地区热力着色)**:
怎么实现“国家/地区热力着色”？真正精准的边界需要 GeoJSON，这会使得代码极其冗长。题目要求“单一 HTML 文件”，无法包含庞大的 GeoJSON。我们可以采用“基于 Voronoi 或离散点的热力扩散”来模拟地区着色。或者，在地球上生成一些代表不同区域的热力圆（Soft Circular Gradients），随着指标改变颜色（如蓝色到红色）。
由于有“主要城市”，我们把热力图定义为以主要城市为中心的径向渐变发光多边形（一个盖在地球表面的平面圆，贴着球面）。这种艺术化的展示很常见，也能满足“热力着色”的要求。
具体做法：创建一个稍微大于地球半径（R * 1.01）的球体，或者使用 `SphereGeometry` 贴图，但考虑到没有 GeoJSON，我们可以将“热力图”实现为：附加在每个城市底部的一个半透明的光圈（用 Shader 或者 Sprite 实现），根据数据大小改变半径和颜色，叠加起来产生热力效果。或者生成一个覆盖全球的低多边形球面，根据城市距离给顶点上色（Voronoi风）。
我选择：在每个城市下方添加一个 Ground Circle（贴地圆），用自定义 Shader 或者透明度贴图实现边缘羽化的光圈。当开启热力图时显示，并根据指标调整颜色和大小。多个圈叠加就形成了热力图效果。这样代码量可控而且很美观。

让我们梳理整个单一 HTML 结构。
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>3D 数据可视化地球仪</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #000; font-family: sans-serif; color: #fff; }
        #container { width: 100vw; height: 100vh; }
        #ui-panel { position: absolute; top: 20px; left: 20px; background: rgba(0,0,0,0.7); padding: 15px; border-radius: 8px; border: 1px solid #444; z-index: 10; width: 250px;}
        #ui-panel h2 { margin: 0 0 10px 0; font-size: 16px; border-bottom: 1px solid #555; padding-bottom: 5px;}
        .control-group { margin-bottom: 10px; }
        .control-group label { display: block; font-size: 12px; margin-bottom: 5px; color: #aaa;}
        select, input[type=range], button { width: 100%; box-sizing: border-box; background: #222; color: #fff; border: 1px solid #555; padding: 5px; border-radius: 4px;}
        button { cursor: pointer; margin-top: 5px; background: #333; }
        button:hover { background: #444; }
        #tooltip { position: absolute; background: rgba(0,0,0,0.8); border: 1px solid #00ffff; padding: 8px; border-radius: 4px; pointer-events: none; display: none; z-index: 20; font-size: 12px; transform: translate(-50%, -120%); box-shadow: 0 0 10px rgba(0,255,255,0.5); }
        #modal { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(0,20,40,0.9); border: 1px solid #00ffff; padding: 20px; border-radius: 8px; display: none; z-index: 30; width: 300px; box-shadow: 0 0 20px rgba(0,255,255,0.3); }
        #modal h3 { margin-top: 0; color: #00ffff; }
        #modal p { font-size: 14px; line-height: 1.5; }
        .close-btn { position: absolute; top: 10px; right: 15px; cursor: pointer; font-weight: bold; color: #fff; }
        #time-display { font-weight: bold; color: #00ffff; font-size: 14px; margin-bottom: 5px; display: block;}
    </style>
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>
</head>
<body>
    <div id="container"></div>
    
    <div id="ui-panel">
        <h2>地球数据控制台</h2>
        <div class="control-group">
            <label for="metric-select">数据指标</label>
            <select id="metric-select">
                <option value="gdp">GDP (十亿美元)</option>
                <option value="population">人口 (百万)</option>
                <option value="flights">航班量 (千次/年)</option>
            </select>
        </div>
        <div class="control-group">
            <label for="heatmap-toggle">
                <input type="checkbox" id="heatmap-toggle" checked> 显示国家/地区热力
            </label>
        </div>
        <div class="control-group">
            <span id="time-display">年份: 2020</span>
            <label for="time-slider">时间轴</label>
            <input type="range" id="time-slider" min="2000" max="2020" value="2020" step="1">
            <button id="play-btn">▶ 播放动画</button>
        </div>
    </div>

    <div id="tooltip"></div>

    <div id="modal">
        <span class="close-btn" id="modal-close">&times;</span>
        <h3 id="modal-title">城市名称</h3>
        <p id="modal-content">详细信息...</p>
    </div>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // 核心代码...
    </script>
</body>
</html>
```

接下来是核心 Three.js 逻辑。
1. **Scene, Camera, Renderer**
```javascript
const container = document.getElementById('container');
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 0, 15);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
container.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.minDistance = 8;
controls.maxDistance = 30;
```

2. **Textures (真实地球纹理)**
为了加载纹理，使用 `TextureLoader`。由于外网图片可能慢，提供 fallback 颜色？或者就用 CDN。
```javascript
const texLoader = new THREE.TextureLoader();
// 使用 unpkg 上的 three-globe 资源，比较稳定而且包含地球各种贴图
const dayMap = texLoader.load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg');
const nightMap = texLoader.load('https://unpkg.com/three-globe/example/img/earth-night.jpg');
const bumpMap = texLoader.load('https://unpkg.com/three-globe/example/img/earth-topology.png');
```

3. **Earth Mesh**
```javascript
const earthRadius = 5;
const earthGeo = new THREE.SphereGeometry(earthRadius, 64, 64);

const earthUniforms = {
    dayTexture: { value: dayMap },
    nightTexture: { value: nightMap },
    sunDirection: { value: new THREE.Vector3(1, 0, 0).normalize() }
};

const earthMat = new THREE.ShaderMaterial({
    uniforms: earthUniforms,
    vertexShader: `
        varying vec2 vUv;
        varying vec3 vWorldNormal;
        void main() {
            vUv = uv;
            vWorldNormal = normalize(mat3(modelMatrix) * normal);
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform sampler2D dayTexture;
        uniform sampler2D nightTexture;
        uniform vec3 sunDirection;
        varying vec2 vUv;
        varying vec3 vWorldNormal;
        void main() {
            float intensity = dot(normalize(vWorldNormal), normalize(sunDirection));
            // 调整日夜分界线
            float mixStrength = smoothstep(-0.15, 0.25, intensity);
            vec3 dayColor = texture2D(dayTexture, vUv).rgb;
            vec3 nightColor = texture2D(nightTexture, vUv).rgb;
            // 夜晚灯光稍微提亮
            nightColor *= 1.2;
            vec3 finalColor = mix(nightColor, dayColor, mixStrength);
            gl_FragColor = vec4(finalColor, 1.0);
        }
    `
});

const earth = new THREE.Mesh(earthGeo, earthMat);
scene.add(earth);
```
注意：由于我们在 render loop 里可能让 `earth` 自转，如果 `earth` 旋转，它的 `modelMatrix` 会改变。因为我们把 `sunDirection` 固定在世界空间，那么随着地球自转，`vWorldNormal` 也会跟着转动（因为法线基于模型矩阵），所以日夜分界线会在地球表面移动，完美模拟自转。

4. **Atmosphere (大气光晕)**
```javascript
const atmosphereGeo = new THREE.SphereGeometry(earthRadius * 1.15, 64, 64);
const atmosphereMat = new THREE.ShaderMaterial({
    vertexShader: `
        varying vec3 vWorldNormal;
        varying vec3 vViewDir;
        void main() {
            vWorldNormal = normalize(mat3(modelMatrix) * normal);
            vec4 worldPos = modelMatrix * vec4(position, 1.0);
            vViewDir = normalize(cameraPosition - worldPos.xyz);
            gl_Position = projectionMatrix * viewMatrix * worldPos;
        }
    `,
    fragmentShader: `
        varying vec3 vWorldNormal;
        varying vec3 vViewDir;
        void main() {
            // 边缘发光 (Fresnel)
            float dotProduct = dot(vWorldNormal, vViewDir);
            // 如果是 BackSide 渲染，法线是朝内的，所以我们要修正一下，或者用 1.0 - abs(dot) 之类
            // 这里用 FrontSide 渲染大气层也可以，但要开启透明和 AdditiveBlending，且 depthWrite: false
            float intensity = pow(0.8 - dotProduct, 4.0);
            gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
        }
    `,
    blending: THREE.AdditiveBlending,
    side: THREE.BackSide,
    transparent: true,
    depthWrite: false
});
const atmosphere = new THREE.Mesh(atmosphereGeo, atmosphereMat);
scene.add(atmosphere);
```
对于 `BackSide`，法线是向内的，所以 `vWorldNormal` 是指向球心的。当我们在外面看时，`vViewDir` 从表面指向相机（向外）。所以 `dot(vWorldNormal, vViewDir)` 在边缘大概是 0（因为法线向内，视线向外，夹角约 90度），在中心是 -1。所以 `0.8 - (-1) = 1.8`，`pow(1.8, 4)` 可能很大。我们需要调整：
如果用 `FrontSide`：
```javascript
    side: THREE.FrontSide,
    // ...
    fragmentShader: `
        varying vec3 vWorldNormal;
        varying vec3 vViewDir;
        void main() {
            float dotProduct = dot(vWorldNormal, vViewDir);
            // dotProduct 在中心为 1，边缘为 0
            float intensity = pow(1.0 - dotProduct, 4.0);
            gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
        }
    `,
```
这样简单且效果好。我将改用 `FrontSide` 大气层，并稍微扩大一点点（比如 `earthRadius * 1.02`），或者 `1.1` 且中心 intensity 设小点。
更常见的做法：
```javascript
const atmosphereMat = new THREE.ShaderMaterial({
    vertexShader: `
        varying vec3 vNormal;
        void main() {
            vNormal = normalize(normalMatrix * normal);
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        varying vec3 vNormal;
        void main() {
            float intensity = pow(0.6 - dot(vNormal, vec3(0, 0, 1.0)), 4.0);
            gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
        }
    `,
    blending: THREE.AdditiveBlending,
    side: THREE.BackSide,
    transparent: true,
    depthWrite: false
});
```
经典的三维地球大气发光 Shader，`vNormal` 是视图空间法线，`vec3(0,0,1)` 是视线方向。因为 BackSide，中心背面法线指向屏幕里（即 -z），所以 `dot` 是负的，`0.6 - (-1) = 1.6`，边缘是 `0.6 - 0 = 0.6`，`pow(0.6, 4) = 0.12` 可见；更边缘接近 0 时 `pow(0, 4) = 0`。效果是不错的。我采用这个经典写法。

5. **Cities Data & Markers**
我们准备几个城市：
```javascript
const cities = [
    { name: '北京', lat: 39.9, lon: 116.4, baseData: { gdp: 500, pop: 21, flights: 600 } },
    { name: '纽约', lat: 40.7, lon: -74.0, baseData: { gdp: 1000, pop: 8, flights: 1200 } },
    { name: '伦敦', lat: 51.5, lon: -0.1, baseData: { gdp: 800, pop: 9, flights: 1100 } },
    { name: '东京', lat: 35.6, lon: 139.6, baseData: { gdp: 950, pop: 37, flights: 900 } },
    { name: '悉尼', lat: -33.8, lon: 151.2, baseData: { gdp: 400, pop: 5, flights: 400 } },
    { name: '巴黎', lat: 48.8, lon: 2.3, baseData: { gdp: 700, pop: 11, flights: 800 } },
    { name: '莫斯科', lat: 55.7, lon: 37.6, baseData: { gdp: 450, pop: 12, flights: 500 } },
    { name: '圣保罗', lat: -23.5, lon: -46.6, baseData: { gdp: 350, pop: 22, flights: 450 } },
    { name: '开普敦', lat: -33.9, lon: 18.4, baseData: { gdp: 150, pop: 4, flights: 200 } },
    { name: '洛杉矶', lat: 34.0, lon: -118.2, baseData: { gdp: 900, pop: 13, flights: 1000 } }
];

// 生成圆形发光纹理用于 Sprite
function createCircleTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = 64;
    canvas.height = 64;
    const ctx = canvas.getContext('2d');
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.2, 'rgba(0, 255, 255, 0.8)');
    gradient.addColorStop(1, 'rgba(0, 255, 255, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 64, 64);
    return new THREE.CanvasTexture(canvas);
}
const dotTexture = createCircleTexture();
```

生成城市标记点：
```javascript
const cityMarkers = [];
const heatmapMeshes = [];

cities.forEach(city => {
    const pos = latLonToVector3(city.lat, city.lon, earthRadius);
    
    // 发光点 Sprite
    const mat = new THREE.SpriteMaterial({ map: dotTexture, color: 0x00ffff, transparent: true, blending: THREE.AdditiveBlending });
    const sprite = new THREE.Sprite(mat);
    sprite.position.copy(pos);
    // 把 sprite 放在 earth 里，这样自转时会跟着转！
    earth.add(sprite); 
    
    // 创建一个不可见的大球用于点击检测
    const hitGeo = new THREE.SphereGeometry(0.3, 8, 8);
    const hitMat = new THREE.MeshBasicMaterial({ visible: false });
    const hitMesh = new THREE.Mesh(hitGeo, hitMat);
    hitMesh.position.copy(pos);
    hitMesh.userData = { isCity: true, cityInfo: city };
    earth.add(hitMesh);
    cityMarkers.push({ sprite, hitMesh, city });

    // 热力圈（贴地球表面的 Plane 或 Circle）
    const heatGeo = new THREE.CircleGeometry(1, 32);
    const heatMat = new THREE.MeshBasicMaterial({ 
        color: 0xff0000, 
        transparent: true, 
        opacity: 0.3, 
        blending: THREE.AdditiveBlending,
        depthWrite: false
    });
    const heatMesh = new THREE.Mesh(heatGeo, heatMat);
    // 将热力圈定位到城市位置，并朝向外面
    heatMesh.position.copy(pos.clone().multiplyScalar(1.01));
    heatMesh.lookAt(pos.clone().multiplyScalar(2)); // 面向法线外侧
    earth.add(heatMesh);
    heatmapMeshes.push(heatMesh);
});
```
注意：把 `sprite` 和 `hitMesh` 加到 `earth` 里，这样地球自转时，标记点和检测球会一起转。非常好。

6. **Routes (航线)**
假设的航线：北京-纽约，伦敦-东京，纽约-洛杉矶，悉尼-伦敦，巴黎-开普敦，莫斯科-北京，圣保罗-纽约，东京-悉尼。
```javascript
const routesData = [
    [0, 1], [2, 3], [1, 9], [4, 2], [5, 8], [6, 0], [7, 1], [3, 4]
]; // 对应 cities 数组索引

const routeMeshes = [];
const routeUniformsArray = [];

routesData.forEach(pair => {
    const startCity = cities[pair[0]];
    const endCity = cities[pair[1]];
    const startVec = latLonToVector3(startCity.lat, startCity.lon, earthRadius);
    const endVec = latLonToVector3(endCity.lat, endCity.lon, earthRadius);
    
    // 计算大圆航线控制点
    const distance = startVec.distanceTo(endVec);
    const midVec = startVec.clone().add(endVec).multiplyScalar(0.5);
    const altitude = earthRadius + distance * 0.4; 
    midVec.normalize().multiplyScalar(altitude);
    
    const curve = new THREE.QuadraticBezierCurve3(startVec, midVec, endVec);
    const tubeGeo = new THREE.TubeGeometry(curve, 64, 0.02, 8, false);
    
    const routeUniforms = {
        time: { value: 0 },
        color: { value: new THREE.Color(0x00ffff) }
    };
    routeUniformsArray.push(routeUniforms);
    
    const routeMat = new THREE.ShaderMaterial({
        uniforms: routeUniforms,
        vertexShader: `
            varying vec2 vUv;
            void main() {
                vUv = uv;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `,
        fragmentShader: `
            uniform float time;
            uniform vec3 color;
            varying vec2 vUv;
            void main() {
                // vUv.x 沿管道长度
                float flow = fract(vUv.x * 3.0 - time);
                // 让流动看起来像流星
                float alpha = smoothstep(0.8, 1.0, flow) * 0.8 + 0.1;
                gl_FragColor = vec4(color, alpha);
            }
        `,
        transparent: true,
        blending: THREE.AdditiveBlending,
        depthWrite: false
    });
    
    const tube = new THREE.Mesh(tubeGeo, routeMat);
    tube.userData = { 
        isRoute: true, 
        info: `从 ${startCity.name} 到 ${endCity.name} 的航班\n距离: ${Math.round(distance * 1000)} km` 
    };
    earth.add(tube); // 一起自转
    routeMeshes.push(tube);
});
```

7. **Interactions (Raycaster)**
```javascript
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const tooltip = document.getElementById('tooltip');
const modal = document.getElementById('modal');
const modalTitle = document.getElementById('modal-title');
const modalContent = document.getElementById('modal-content');

// 为了检测线，需要增加 Raycaster 的 Line/Points threshold? 
// 这里我们用的是 Tube (Mesh)，所以直接用即可。
// 但是 Sprite 的 raycast 需要特殊处理吗？我们把 hitMesh 放进去检测。

window.addEventListener('mousemove', (e) => {
    mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects([...cityMarkers.map(c => c.hitMesh), ...routeMeshes]);
    
    if (intersects.length > 0) {
        const obj = intersects[0].object;
        if (obj.userData.isCity) {
            document.body.style.cursor = 'pointer';
            tooltip.style.display = 'block';
            tooltip.style.left = e.clientX + 'px';
            tooltip.style.top = e.clientY + 'px';
            tooltip.innerHTML = `<strong>${obj.userData.cityInfo.name}</strong><br>点击查看详情`;
        } else if (obj.userData.isRoute) {
            document.body.style.cursor = 'pointer';
            tooltip.style.display = 'block';
            tooltip.style.left = e.clientX + 'px';
            tooltip.style.top = e.clientY + 'px';
            tooltip.innerHTML = obj.userData.info.replace('\n', '<br>');
        } else {
            document.body.style.cursor = 'default';
            tooltip.style.display = 'none';
        }
    } else {
        document.body.style.cursor = 'default';
        tooltip.style.display = 'none';
    }
});

window.addEventListener('click', (e) => {
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(cityMarkers.map(c => c.hitMesh));
    if (intersects.length > 0) {
        const city = intersects[0].object.userData.cityInfo;
        modal.style.display = 'block';
        modalTitle.innerText = city.name;
        const year = document.getElementById('time-slider').value;
        // 简单模拟随时间的变化 (基准值 * 年份系数)
        const factor = 1 + (year - 2000) * 0.02;
        modalContent.innerHTML = `
            <strong>年份:</strong> ${year}<br>
            <strong>GDP:</strong> ${(city.baseData.gdp * factor).toFixed(1)} 十亿美元<br>
            <strong>人口:</strong> ${(city.baseData.pop * factor).toFixed(1)} 百万<br>
            <strong>航班量:</strong> ${(city.baseData.flights * factor).toFixed(0)} 千次
        `;
    }
});

document.getElementById('modal-close').addEventListener('click', () => {
    modal.style.display = 'none';
});
```

8. **Control Panel Logic**
```javascript
const metricSelect = document.getElementById('metric-select');
const heatmapToggle = document.getElementById('heatmap-toggle');
const timeSlider = document.getElementById('time-slider');
const timeDisplay = document.getElementById('time-display');
const playBtn = document.getElementById('play-btn');

let currentMetric = 'gdp';
let isPlaying = false;

metricSelect.addEventListener('change', (e) => {
    currentMetric = e.target.value;
    updateVisuals();
});

heatmapToggle.addEventListener('change', (e) => {
    const show = e.target.checked;
    heatmapMeshes.forEach(m => m.visible = show);
});

timeSlider.addEventListener('input', (e) => {
    timeDisplay.innerText = `年份: ${e.target.value}`;
    updateVisuals();
});

playBtn.addEventListener('click', () => {
    isPlaying = !isPlaying;
    playBtn.innerText = isPlaying ? '⏸ 暂停动画' : '▶ 播放动画';
});

function updateVisuals() {
    const year = parseInt(timeSlider.value);
    const factor = 1 + (year - 2000) * 0.03; // 模拟增长
    
    cityMarkers.forEach((item, idx) => {
        const data = item.city.baseData[currentMetric] * factor;
        // 根据指标设置大小
        let scale = 0.2 + data / 100; 
        if (currentMetric === 'population') scale = 0.2 + data / 50;
        if (currentMetric === 'flights') scale = 0.2 + data / 1000;
        item.sprite.scale.set(scale, scale, scale);
        
        // 更新热力圈
        const heatMesh = heatmapMeshes[idx];
        heatMesh.scale.set(scale * 2, scale * 2, scale * 2);
        
        // 热力颜色随指标大小渐变 (蓝 -> 黄 -> 红)
        const ratio = Math.min(data / (item.city.baseData[currentMetric] * 1.5), 1.0); // 粗略归一化
        const color = new THREE.Color();
        color.setHSL((1 - ratio) * 0.6, 1.0, 0.5); // 0.6 是蓝，0 是红
        heatMesh.material.color = color;
    });
}
updateVisuals(); // 初始化
```

9. **Time Axis Play Logic & Render Loop**
```javascript
const clock = new THREE.Clock();

function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();
    const elapsed = clock.getElapsedTime();
    
    // 地球自转
    earth.rotation.y += delta * 0.05;
    
    // 太阳方向可以固定在世界空间，比如 x 轴方向
    // 也可以随时间变动，为了模拟真实，我们把地球放在原点自转，太阳方向不变
    // (在 Shader 里用的是世界法线，地球转了，法线世界坐标变，所以昼夜线相对地球表面在动)
    
    // 更新航线流动动画
    routeUniformsArray.forEach(u => {
        u.time.value = elapsed * 0.5; // 流动速度
    });
    
    // 时间轴播放
    if (isPlaying) {
        let y = parseInt(timeSlider.value);
        y += 1; // 每年... 这里太快了，可以改成根据时间累加
        // 为了平滑，可以不在每帧 +1，而是每次时间累加到达一定间隔 +1
        // 简单处理：
        if (Math.floor(elapsed * 2) % 2 === 0) { // 每 0.5 秒变化？不，最好是直接累加，但不用太严谨
        }
    }
    
    controls.update();
    renderer.render(scene, camera);
}
```
改进时间轴播放逻辑：
```javascript
let lastTime = 0;
function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();
    const elapsed = clock.getElapsedTime();
    
    earth.rotation.y += delta * 0.05;
    
    routeUniformsArray.forEach(u => {
        u.time.value = elapsed * 0.3;
    });
    
    if (isPlaying) {
        // 每秒增加 1 年
        if (elapsed - lastTime > 1.0) {
            lastTime = elapsed;
            let y = parseInt(timeSlider.value);
            y += 1;
            if (y > 2020) y = 2000;
            timeSlider.value = y;
            timeDisplay.innerText = `年份: ${y}`;
            updateVisuals();
        }
    }
    
    controls.update();
    renderer.render(scene, camera);
}
```

10. **Resize Handler**
```javascript
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
```

11. **细节完善**
- 纹理加载可能慢，加上 LoadingManager？或者只是等待即可。题目要求单文件，我们给 scene 加点背景星星。
- 星空背景：
```javascript
const starsGeo = new THREE.BufferGeometry();
const starsCount = 2000;
const posArray = new Float32Array(starsCount * 3);
for(let i=0; i<starsCount*3; i++) {
    posArray[i] = (Math.random() - 0.5) * 200;
}
starsGeo.setAttribute('position', new THREE.BufferAttribute(posArray, 3));
const starsMat = new THREE.PointsMaterial({ size: 0.5, color: 0xffffff, transparent: true, opacity: 0.8 });
const stars = new THREE.Points(starsGeo, starsMat);
scene.add(stars);
```
- `latLonToVector3`:
```javascript
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * (Math.PI / 180);
    const theta = (lon + 180) * (Math.PI / 180);
    const x = -radius * Math.sin(phi) * Math.cos(theta);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    const y = radius * Math.cos(phi);
    return new THREE.Vector3(x, y, z);
}
```
这个转换配合 `earth-blue-marble.jpg` 是否正确？
Three.js 的 SphereGeometry 的 UV 映射：
默认情况下，赤道在 y=0，北极在 y=R。
u=0 到 1 对应经度，v=0 到 1 对应纬度（v=0.5 是赤道）。
一般的球面坐标公式：
`x = -r * sin(phi) * cos(theta)`
`z = r * sin(phi) * sin(theta)`
`y = r * cos(phi)`
其中 `phi` 是从北极开始量（所以 `phi = (90 - lat) * PI/180` 是对的，`lat=90` 时 `phi=0`, `y=r`）。
`theta` 是绕 y 轴的方位角，通常 `theta = (lon + 180) * PI/180` 或 `lon * PI/180` 取决于贴图。对于 `earth-blue-marble.jpg`，贴图中心通常是本初子午线（经度0）。在 Three.js 中，SphereGeometry 的 `u=0.5` 对应 `+z` 轴？不，是在 `x=0, z=-R` 处开始。
如果用标准公式：
```javascript
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * (Math.PI / 180);
    const theta = (lon + 90) * (Math.PI / 180); // 测试调整 +90 或 +180
    const x = radius * Math.sin(phi) * Math.cos(theta);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    const y = radius * Math.cos(phi);
    return new THREE.Vector3(x, y, z);
}
```
等等，我之前用过类似的代码，通常 `theta = (lon + 180) * (Math.PI / 180)` 或者 `theta = (lon - 90) * (Math.PI / 180)` 才能和 Three.js 默认的 SphereGeometry UV 对齐。
让我推导一下：
Three.js `SphereGeometry`：
```
const u = j / widthSegments; // 0 to 1
const v = i / heightSegments; // 0 to 1
...
x = - radius * Math.cos( phiStart + u * phiLength ) * Math.sin( thetaStart + v * thetaLength );
y = radius * Math.cos( thetaStart + v * thetaLength );
z = radius * Math.sin( phiStart + u * phiLength ) * Math.sin( thetaStart + v * thetaLength );
```
默认 `phiStart = 0`, `phiLength = 2*PI`, `thetaStart = 0`, `thetaLength = PI`。
所以 `y = r * cos(v * PI)`。当 `v=0`（北极），`y=r`；当 `v=1`（南极），`y=-r`。
`v = 0.5` 时 `y=0`（赤道）。
对于赤道处的点：`y=0`，`x = -r * cos(u*2PI) * sin(PI/2) = -r * cos(u*2PI)`；
`z = r * sin(u*2PI) * sin(PI/2) = r * sin(u*2PI)`。
当 `u=0` 时，`x=-r`, `z=0`（对应贴图的最左边缘，通常贴图左边缘是本初子午线往西 180 度，也就是经度 -180）。
当 `u=0.25` 时，`x=0`, `z=r`（贴图经度 -90 的地方，也就是美洲大概）。
当 `u=0.5` 时，`x=r`, `z=0`（贴图经度 0 度的地方，也就是本初子午线，经过英国）。
当 `u=0.75` 时，`x=0`, `z=-r`（贴图经度 +90 的地方，亚洲东部）。

我们有城市坐标：
北京：`lat=39.9, lon=116.4`（东经 116.4，应在亚洲东部，`u` 应该大概在 `0.5 + 116.4/360 = 0.823` 处）。
带入我的推导：
对于赤道平面 `y=0`，`lon` 对应的 `u = (lon + 180) / 360`。
所以 `u*2*PI = (lon+180)*PI/180`。
`x = -R * cos((lon+180)*PI/180)`
`z = R * sin((lon+180)*PI/180)`
`y = R * sin(lat*PI/180)` （因为从赤道 `lat`，`v = 0.5 - lat/180`，`cos(v*PI) = cos(PI/2 - lat*PI/180) = sin(lat*PI/180)`）

代入 `lon=116.4`, `lat=0`:
`u = (116.4+180)/360 = 296.4/360 = 0.823`。
`x = -R * cos(296.4度) = -R * cos(-63.6度) = -R * 0.44`
`z = R * sin(296.4度) = R * (-0.89)`
这是对的，Z 为负，X 为负，也就是在 `(-x, -z)` 象限，对应 `u=0.75` 到 `u=1.0` 之间（亚洲东部和澳洲），正确！

所以转换函数应为：
```javascript
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * (Math.PI / 180);
    const theta = (lon + 180) * (Math.PI / 180);
    // 根据上面的推导：
    // x = -radius * cos(theta) * sin(phi)  注意这里 phi 是极角(从y轴向下)，所以 sin(phi) 是赤道平面半径
    // y = radius * cos(phi)
    // z = radius * sin(theta) * sin(phi)
    const x = -radius * Math.sin(phi) * Math.cos(theta);
    const y = radius * Math.cos(phi);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    return new THREE.Vector3(x, y, z);
}
```
仔细核对：
`phi` 是从北极向下的角度。`lat=90` -> `phi=0`，`sin=0, cos=1`，`y=R, x=0, z=0`。正确。
`lat=0` -> `phi=90度`，`sin=1, cos=0`，`y=0`。正确。
`theta = (lon+180)*PI/180`
`lon=0` -> `theta=180度` -> `cos=-1, sin=0` -> `x=R, z=0`（本初子午线）。正确。
`lon=-180` -> `theta=0` -> `cos=1, sin=0` -> `x=-R, z=0`。正确。
`lon=90` -> `theta=270度` -> `cos=0, sin=-1` -> `x=0, z=-R`。正确。
`lon=-90` -> `theta=90度` -> `cos=0, sin=1` -> `x=0, z=R`。正确。
非常完美，经纬度转换绝对准确了！

12. **Heatmap 圈如何准确贴在球面上？
`heatMesh.position.copy(pos.clone().multiplyScalar(1.01));`
`heatMesh.lookAt(pos.clone().multiplyScalar(2));`
等一下，`lookAt` 的参数是世界坐标系中的一个点！
但是如果 `heatMesh` 是作为 `earth` 的子对象添加的（`earth.add(heatMesh)`），那么它的 `position` 和 `lookAt` 都是在 `earth` 的局部坐标系中进行的（或者用世界坐标？Three.js 的 `Object3D.lookAt` 在局部坐标系下表现稍微复杂，如果它有父对象，它期望的参数是世界坐标，但算法是在局部空间内应用的？不，Three.js 的 `lookAt` 是将对象在世界空间中朝向给定的世界空间坐标。然后内部会求逆矩阵转换成局部旋转。其实简单点，我们不需要复杂的 `lookAt`，因为我们把 `pos` 作为局部坐标：
在局部坐标系中，`pos` 是球面坐标，`pos.clone().multiplyScalar(2)` 是沿着法线向外的方向。由于局部原点就是地球中心，`heatMesh` 的局部位置是 `pos * 1.01`，我们让它朝向 `pos * 2`（局部坐标点）。因为 `lookAt` 在最新版 Three.js 中如果对象有 parent，传入的是世界坐标。这会导致错误！
如果我们把 `heatMesh` 加在 `earth` 里，且 `earth` 发生旋转，`lookAt` 的结果会不对。
正确做法：
在局部坐标空间里，直接设置旋转！
或者直接把 `heatMesh` 加到 `scene` 里，每帧跟着 `earth` 转？不，太麻烦。
对于贴着球面的平面（CircleGeometry 默认在 XY 平面上，法线是 +Z）：
我们要让它的法线 (+Z) 对齐到 `pos` 的方向（也就是 `pos.clone().normalize()`）。
所以：
```javascript
const normal = pos.clone().normalize();
heatMesh.position.copy(pos.clone().multiplyScalar(1.01));
// 使用 quaternion 从 +Z 旋转到 normal
heatMesh.quaternion.setFromUnitVectors(new THREE.Vector3(0,0,1), normal);
```
这样就绝对没问题了，且不受父对象旋转影响（因为是基于局部向量计算的，父对象旋转时它会一起转）。

同样，对于城市的 `hitMesh` 和 `sprite`：
`Sprite` 总是面向相机的，不需要旋转，只要 `position` 对就行。
`hitMesh` 是球，无所谓。

13. **仔细检查 Shader 的编译和兼容**
为了兼容 Three.js 0.160，所有的 GLSL 不需要加 `precision` 也可以（Three.js 会自动加）。
`uniform sampler2D dayTexture;` 等确保传进去了。
地球昼夜分界线的 Shader：
```glsl
uniform sampler2D dayTexture;
uniform sampler2D nightTexture;
uniform vec3 sunDirection;
varying vec2 vUv;
varying vec3 vWorldNormal;
void main() {
    float intensity = dot(normalize(vWorldNormal), normalize(sunDirection));
    float mixStrength = smoothstep(-0.15, 0.25, intensity);
    vec3 dayColor = texture2D(dayTexture, vUv).rgb;
    vec3 nightColor = texture2D(nightTexture, vUv).rgb;
    nightColor *= 1.2; // 提亮夜景城市灯光
    vec3 finalColor = mix(nightColor, dayColor, mixStrength);
    gl_FragColor = vec4(finalColor, 1.0);
}
```
`sunDirection` 是 `vec3(1, 0, 0)` 吗？如果是的，在初始状态下，地球右边是白天，左边是黑夜。由于地球 `rotation.y` 不断增加，地球右半边慢慢转过去，所以是自转效果。
等下，如果 `earth` 是自转的（绕 Y 轴），`vWorldNormal` 是随着顶点转动的，所以 `dot` 会发生变化，从而昼夜线扫过地球表面。完美。

14. **Route Flow Shader 的 `vUv`**
`TubeGeometry` 默认生成的 UV 中，X 对应管子长度（[0,1]），Y 对应管子圆周（[0,1]）。
所以 `vUv.x` 用来做流动是对的。
```glsl
uniform float time;
uniform vec3 color;
varying vec2 vUv;
void main() {
    float flow = fract(vUv.x * 3.0 - time);
    float alpha = smoothstep(0.8, 1.0, flow) * 0.8 + 0.1;
    gl_FragColor = vec4(color, alpha);
}
```
这会让管子底色有 0.1 的透明度，然后在管子上每隔 1/3 长度有一个彗星状的亮斑（长 20% 长度）在流动（方向是从起点到终点？`fract(vUv.x*3 - time)`，`time` 增加，`-time` 会让 `flow` 减小，所以波形向 `+x` 也就是终点移动，是正确的流动方向）。

15. **Tooltip / 弹窗交互逻辑优化**
如果是鼠标移动，如果有相交，就更新 Tooltip：
```javascript
let hoveredObj = null;
window.addEventListener('mousemove', (e) => {
    // ...
    if (intersects.length > 0) {
        const obj = intersects[0].object;
        if (obj.userData.isCity || obj.userData.isRoute) {
            // ...
        }
    }
});
```
注意 `intersects` 包含了 `cityMarkers` 的 `hitMesh` 和 `routeMeshes`。
但是因为 `routeMeshes` 是细管子，不太好悬停。我们可以把管子的 `radius` 设大一点，或者在 Shader 里画粗一点。目前的 `0.02` 在 `R=5` 的地球上可能有点细。稍微加粗到 `0.04` 或者 `0.05`，这样比较好悬停。
`const tubeGeo = new THREE.TubeGeometry(curve, 64, 0.05, 8, false);`

由于 `hitMesh` 被加入 `earth`，`routeMeshes` 也被加入 `earth`。当 `earth` 旋转时，它们在局部坐标里不动，所以 Raycaster 在工作坐标系（世界坐标）下工作时，会自动考量 `earth` 的 `matrixWorld`，没有任何问题！

16. **处理 `timeSlider` 和 `updateVisuals` 中的颜色**
在 `updateVisuals` 中：
```javascript
const heatMesh = heatmapMeshes[idx];
heatMesh.scale.set(scale * 2, scale * 2, scale * 2);
```
这里，`scale` 大概在 `0.2 ~ 1.0` 左右。乘 2 就是 `0.4 ~ 2.0`。
`CircleGeometry(1, 32)` 初始半径是 1，所以缩放后半径是 `0.4 ~ 2.0`，而地球半径是 5，这个尺寸看起来合适（大约 10% - 40% 地球半径）。如果觉得大，可以调系数。

热力颜色 HSL：
`color.setHSL((1 - ratio) * 0.6, 1.0, 0.5)`。
`ratio` 取决于数据。我们假设：
```javascript
let maxVal = 0;
cityMarkers.forEach(item => {
    maxVal = Math.max(maxVal, item.city.baseData[currentMetric]);
});
// 然后在循环里算 ratio:
const ratio = Math.min(data / (maxVal * 1.2), 1.0);
```
这样更合理。修改 `updateVisuals`：
```javascript
function updateVisuals() {
    const year = parseInt(timeSlider.value);
    const factor = 1 + (year - 2000) * 0.03;
    
    let maxVal = 0;
    cityMarkers.forEach(item => {
        maxVal = Math.max(maxVal, item.city.baseData[currentMetric] * factor);
    });
    
    cityMarkers.forEach((item, idx) => {
        const data = item.city.baseData[currentMetric] * factor;
        let scale = 0.2 + (data / maxVal) * 0.8; // 缩放从 0.2 到 1.0
        item.sprite.scale.set(scale, scale, scale);
        
        const heatMesh = heatmapMeshes[idx];
        heatMesh.scale.set(scale * 1.5, scale * 1.5, scale * 1.5);
        
        const ratio = data / maxVal; // 0 to 1
        const color = new THREE.Color();
        color.setHSL((1 - ratio) * 0.6, 1.0, 0.5); 
        heatMesh.material.color = color;
    });
}
```
完美，这样热力颜色就是相对大小了。

17. **检查依赖和资源**
CDN 链接：
`https://unpkg.com/three@0.160.0/build/three.module.js`
`https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js`
这是标准的 ESM 导入方式，配合 importmap 非常好用。
真实贴图：
`https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg`
`https://unpkg.com/three-globe/example/img/earth-night.jpg`
这在 unpkg 上都是存在的。为了保证安全，可以用 threejs 官方的：
`https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg` (白天)
`https://threejs.org/examples/textures/planets/earth_lights_2048.png` (夜晚)
由于跨域问题，threejs.org 允许跨域加载，而且非常稳定。我使用 threejs.org 的资源更保险，或者 unpkg 的。由于 unpkg 是 npm 镜像，一般允许 CORS。我就用 three-globe 的吧，因为 `earth-blue-marble.jpg` 和 `earth-night.jpg` 都是高分辨率且专门做地球用的，效果非常好。

18. **CSS 和 UI 的完善**
给一个半透明的黑色背景，UI 控件稍微好看点。
Tooltip 要 `pointer-events: none` 否则会挡住鼠标。
弹窗（Modal）要有关闭按钮，居中显示。
所有的 z-index 要正确。

19. **大气层的视觉效果补充**
因为地球是自转的，而大气层是 `scene.add(atmosphere)` 不跟着转的（它本来就是个球壳，转不转无所谓）。
大气层 Shader：
```glsl
varying vec3 vNormal;
void main() {
    float intensity = pow(0.6 - dot(vNormal, vec3(0, 0, 1.0)), 4.0);
    gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
}
```
注意：`vNormal` 用的是 `normalMatrix * normal`，这是视图空间中的法线。视线方向在视图空间是 `(0,0,1)`。
这是正确的，大气在屏幕边缘会变亮。但是当相机拉远或拉近时，它依然在视图空间生效，所以这是一个基于相机的光晕，效果很好。

20. **最后自检要求**
- • 3D 地球：真实纹理、可旋转交互、大气光晕 ✅（OrbitControls、Shader、Texture）
- • 数据可视化层：主要城市标记点发光 ✅（Sprite+CanvasTexture）；航线弧线流动 ✅（Tube+Shader）；国家/地区热力着色 ✅（Heatmap Circle）
- • 交互：拖拽旋转、滚轮缩放 ✅（OrbitControls）；点击城市弹窗 ✅（Raycaster+Modal）；悬停航线显示信息 ✅（Raycaster+Tooltip）
- • 控制面板：切换数据指标 ✅（Select+updateVisuals）；时间轴播放 ✅（Slider+Play+animate）
- • 视觉效果：自转 ✅（earth.rotation.y）；日夜分界线 ✅（Shader SunDirection）；夜晚城市灯光 ✅（Night Map + mix）
- • 技术要求：Three.js、真实地球纹理、importmap+ESM、单一 HTML 文件 ✅
- • 输出要求：完整代码、运行方式、测试说明 ✅

21. **细节打磨：加载时的提示**
由于真实贴图可能加载需要两三秒，最好加个简单的 Loading 提示，避免用户以为黑屏是坏了。
```html
<div id="loading" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 100; color: #fff; font-family: sans-serif;">正在加载地球纹理...</div>
```
然后在 `TextureLoader` 的 `onLoad` 回调里隐藏它。
```javascript
const texLoader = new THREE.TextureLoader();
let loadedCount = 0;
const totalTex = 2;
function onTexLoaded() {
    loadedCount++;
    if(loadedCount === totalTex) {
        document.getElementById('loading').style.display = 'none';
    }
}
const dayMap = texLoader.load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg', onTexLoaded);
const nightMap = texLoader.load('https://unpkg.com/three-globe/example/img/earth-night.jpg', onTexLoaded);
```
这样很贴心。

22. **确保 `OrbitControls` 的导入路径**
```javascript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
```
在 importmap 中：
```json
{
    "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
}
```
这是绝对正确的标准配置。

23. **检查所有代码里的变量名和语法**
- `latLonToVector3` 函数声明要在使用前。
- `createCircleTexture` 函数提前声明。
- `cities` 数组提前声明。
- 在循环中，要把 `earth` 加进 `scene` 后，再对 `earth` 里添加子对象。
- `earth.add(sprite)` 是对的。
- 点击时：`intersects[0].object.userData.cityInfo`，因为 `hitMesh` 带了这个 `userData`，正确。
- `sprite` 的 `userData` 没设置，但是我们用 `hitMesh` 做射线检测，所以不关 `sprite` 的事。
- 如果鼠标移出画布，怎么处理 tooltip？可以监听 `mouseleave` 隐藏 tooltip。

24. **生成最终代码**
将所有的片段组装成一个精美的 HTML 文件，加上详细注释。
并在最后给出运行方式（比如用 Live Server 或者直接拖进浏览器，因为用了 ESM，直接双击打开可能被 CORS 阻止（因为 importmap 和跨域模块），必须通过本地 http 服务器。这很关键，要在测试说明里强调：必须起一个本地 HTTP 服务（如 `python -m http.server` 或 VS Code 的 Live Server），否则由于 ES Module 的 CORS 限制，直接 `file://` 打开会报错）。

好了，整体设计已经闭环，逻辑通顺。准备输出。