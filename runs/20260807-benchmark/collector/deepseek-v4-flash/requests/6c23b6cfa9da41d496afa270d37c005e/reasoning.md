用户想要一个3D数据可视化地球仪，具有丰富的功能和视觉效果。我需要创建一个完整的HTML文件，使用Three.js（通过importmap + ESM方式），包含：

1. 3D地球：真实纹理、旋转交互、大气光晕
2. 数据可视化：城市发光点、航线弧线动画、国家热力着色
3. 交互：拖拽旋转、滚轮缩放、点击城市弹窗、悬停航线显示信息
4. 控制面板：切换指标（GDP、人口、航班量）、时间轴播放
5. 视觉效果：自转动画、日夜分界线、城市灯光

这是一个非常复杂的项目，我需要仔细规划：

- 使用Three.js库（通过CDN + importmap）
- 地球纹理：使用NASA或类似的免费地球纹理URL
- 实现日夜分界线：可以用自定义shader或叠加暗面
- 城市灯光：在夜晚侧显示光点
- 数据可视化：需要准备一些示例数据

让我一步步实现：

**基本设置：**
- importmap指向Three.js CDN
- 创建场景、相机、渲染器
- 添加轨道控制器（OrbitControls）

**地球实现：**
- 使用SphereGeometry加载地球纹理
- 使用MeshPhongMaterial或MeshStandardMaterial
- 大气光晕：使用自定义shader或简单的透明球体

**数据可视化：**
- 城市点：使用Points或小精灵（Sprite）
- 航线：使用QuadraticBezierCurve3创建弧线
- 热力着色：需要自定义shader或使用纹理叠加

**交互：**
- 鼠标拖拽：OrbitControls自带
- 滚动缩放：OrbitControls自带
- 点击城市：Raycaster检测
- 悬停航线：Raycaster检测

**控制面板：**
- HTML/CSS控制面板，切换指标和时间轴

**视觉效果：**
- 自转：持续旋转地球
- 日夜分界线：使用自定义shader或光源方向计算
- 城市灯光：在夜晚侧显示

由于这个项目非常复杂，我需要精简实现但保持功能完整。让我开始编码：

考虑到代码量巨大，我会尽量在核心功能上做完整实现。

让我设计数据结构：
- 城市：名称、经纬度、GDP、人口、航班量、所属国家
- 航线：起点城市、终点城市、航班量

我要选择一个合适的地球纹理URL。可以使用：
- https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg
- https://unpkg.com/three-globe/example/img/earth-night.jpg  
- https://unpkg.com/three-globe/example/img/earth-topology.png

对于日夜分界线，我可以在着色器中实现，或者用光照和自定义shader。

让我考虑使用Three.js的`ShaderMaterial`来实现日夜效果和热力着色。

实际上，为了让代码可控，我会：
1. 使用标准材质加载地球纹理
2. 添加一个半透明的大气层网格
3. 用点精灵（Points）或球面几何体渲染城市
4. 用贝塞尔曲线渲染航线
5. 使用Raycaster进行点击检测

对于热力着色，我可以在地球表面叠加颜色，但实现复杂。我可以简化：根据选中指标，将国家区域着色。这需要国家边界数据，非常复杂。让我用另一种方式：在每个城市位置根据指标值绘制不同大小的圆点，并且可以用城市颜色代表指标值。

对于日夜分界线，我可以使用自定义shader，在顶点/片段着色器中处理太阳方向。

让我简化实现：
- 日夜分界线：用一个自定义shader处理地球材质，根据太阳光方向计算暗面
- 城市灯光：在暗面显示光点

实际上，让我重新思考架构：

地球使用`ShaderMaterial`自定义材质，实现：
- 基础纹理
- 日夜分界线（基于光照方向）
- 热力着色（叠加颜色）

但这太复杂了。让我采用更实用的方法：
- 白天使用地球纹理 + 光照明暗
- 夜晚使用夜景纹理（或黑色+城市灯光点）
- 用`EffectComposer`或简单混合实现

实际上，最简单的方案：
- 用`MeshPhongMaterial`加载纹理，用方向光模拟太阳
- 地球自转，光源固定，这样就能产生日夜效果
- 城市灯光用Sprite/Points显示，并控制其只在暗面可见

让我这样做：

1. 地球使用`MeshPhongMaterial` + 地球纹理，一个方向光作为太阳
2. 大气光晕：一个稍微大一点的半透明球体，用`ShaderMaterial`做边缘发光
3. 城市点：用`Points`放置在表面，每个点根据数据着色，通过动画/材质调整可见性
4. 航线：用`TubeGeometry`或`Line`的贝塞尔曲线

对于热力着色，我可以加载一个世界国家的GeoJSON并在地球上绘制多边形... 这太复杂了。让我换一种方法：不使用真实国家边界，而是使用`CanvasTexture`在地球纹理上绘制热力色块。这需要在画布上绘制经纬度网格并着色... 

让我再简化：使用城市数据点的大小和颜色代表不同指标值，这已经满足了"可视化"的要求。而"国家/地区热力着色"可以用城市点的颜色渐变来近似。

好的，让我确定最终方案：

**最终架构：**

```
场景
├── 地球 (SphereGeometry + MeshPhongMaterial 加载纹理)
├── 大气层 (SphereGeometry + ShaderMaterial 边缘发光)
├── 城市点 (Points 系统，位于地球表面)
├── 航线 (自定义弧线，使用 Line)
├── 日夜分界线 (方向光实现光照)
└── 星星背景 (Points 系统)
```

**数据准备：**
- 内置一些示例城市数据（约20-30个主要城市）
- 航线数据（一些连接关系）
- 每个城市有GDP、人口、航班量等数据

**控制面板：**
- 指标切换按钮
- 时间轴滑块（模拟年份变化）

让我开始写代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D数据可视化地球仪</title>
<style>
  /* CSS */
</style>
</head>
<body>
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
</script>
<script type="module">
// 代码
</script>
</body>
</html>
```

让我思考数据驱动可视化的具体实现：

对于每个城市：
```javascript
{
  name: '北京',
  lat: 39.9,
  lng: 116.4,
  gdp: 4280, // 亿美元
  population: 2154, // 万
  flights: 667, // 每日航班
  country: 'CN'
}
```

对于航线：
```javascript
{
  from: '北京',
  to: '上海',
  flights: 120
}
```

**城市点渲染：**

将经纬度转为3D坐标：
```javascript
function latLngToVector3(lat, lng, radius) {
  const phi = (90 - lat) * Math.PI / 180;
  const theta = (lng + 180) * Math.PI / 180;
  return new THREE.Vector3(
    -radius * Math.sin(phi) * Math.cos(theta),
    radius * Math.cos(phi),
    radius * Math.sin(phi) * Math.sin(theta)
  );
}
```

使用`Points`系统，自定义`PointsMaterial`：
- 大小根据指标值
- 颜色根据指标值（渐变）

或者每个城市用一个`Sprite`，更灵活（可以加标签、hover效果）。

考虑到点击交互，用`Sprite`或独立的`Mesh`更方便。让我用`Sprites`。

**航线旋转动画：**

航线弧线使用`QuadraticBezierCurve3`，中点向外偏移。流动动画可以通过修改`Line`的`dashOffset`或自定义shader实现。

用`LineDashedMaterial` + `dashOffset`动画实现流动效果。

**热力着色：**

我没有国家边界数据，但可以使用Canvas在地球纹理上绘制热力图。具体做法：

1. 在国际上，每个国家用一个颜色值
2. 在Canvas上绘制，然后作为纹理应用到地球

但这需要国家边界GeoJSON。让我使用一种替代方案：

加载一个包含国家边界的图片（如世界地图的轮廓），或者使用`THREE.CanvasTexture`动态绘制热力图。

实际上，可以这样：先不加载真实的Texture，而是用Canvas创建一个地球纹理，在上面画出彩色区域，然后和真实的纹理混合... 这太复杂。

让我换一种方式：将热力数据显示为从城市点向外扩展的圆形渐变，在3D球面上呈现。这样虽然不精确，但能看到"热点区域"。

或者，我可以在地球上方加一层粒子系统，粒子位置对应各国中心，颜色和强度代表指标值。

让我用更简单的方式：在控制面板切换指标时，城市点的颜色和大小会变化，形成一个直观的分布视图。

对于"国家/地区热力着色"，我可以使用一个半透明图层：在地球表面上叠加一个`Mesh`，使用`ShaderMaterial`，在片元着色器中根据经纬度查找预定义的颜色表（每个国家一个颜色），实现热力效果。

这个太复杂了... 让我简化：我将城市数据点的大小和颜色进行可视化，并在城市点周围添加模糊光晕（用Sprite），模拟热力效果。

**最终决定：**

由于这是一个演示项目，我会：
1. 使用地球纹理作为基础
2. 城市点使用带光晕的Sprite，大小和颜色随选中的指标变化
3. 航线使用有流动动画的弧线
4. 城市灯光效果通过Sprite实现，仅在地球暗面显示
5. 日夜分界线通过方向光自然实现（固定光源方向）

让我开始写代码吧。

对了，关于热力着色，我可以尝试用`ShaderMaterial`为地球添加一个热力覆盖层。基本原理：
- 准备一些国家中心点及其数值
- 在地球的片元着色器中，计算每个片元到国家中心点的距离，混合颜色
- 这样形成热力渐变

这实际上是可行的，而且不需要geoJSON边界数据。

但为了简化，我可能会采用基于城市点的热力效果。

让我现在构建代码。由于代码量很大，我需要精心组织。

**城市数据（模拟）：**

我准备一些主要城市：

```javascript
const CITY_DATA = [
  { name: '北京', lat: 39.9, lng: 116.4, gdp: 4280, population: 2154, flights: 667, country: '中国', countryCode: 'CN' },
  { name: '上海', lat: 31.2, lng: 121.5, gdp: 5320, population: 2487, flights: 721, country: '中国', countryCode: 'CN' },
  { name: '东京', lat: 35.7, lng: 139.7, gdp: 19320, population: 3740, flights: 890, country: '日本', countryCode: 'JP' },
  { name: '纽约', lat: 40.7, lng: -74.0, gdp: 18600, population: 841, flights: 1200, country: '美国', countryCode: 'US' },
  { name: '伦敦', lat: 51.5, lng: -0.1, gdp: 9030, population: 890, flights: 750, country: '英国', countryCode: 'GB' },
  { name: '巴黎', lat: 48.9, lng: 2.35, gdp: 8450, population: 1100, flights: 690, country: '法国', countryCode: 'FR' },
  { name: '悉尼', lat: -33.9, lng: 151.2, gdp: 4610, population: 530, flights: 320, country: '澳大利亚', countryCode: 'AU' },
  { name: '莫斯科', lat: 55.8, lng: 37.6, gdp: 3120, population: 1250, flights: 540, country: '俄罗斯', countryCode: 'RU' },
  { name: '迪拜', lat: 25.2, lng: 55.3, gdp: 3980, population: 340, flights: 780, country: '阿联酋', countryCode: 'AE' },
  { name: '新加坡', lat: 1.35, lng: 103.8, gdp: 4890, population: 560, flights: 680, country: '新加坡', countryCode: 'SG' },
  { name: '洛杉矶', lat: 34.05, lng: -118.2, gdp: 11200, population: 390, flights: 860, country: '美国', countryCode: 'US' },
  { name: '芝加哥', lat: 41.9, lng: -87.6, gdp: 7600, population: 270, flights: 640, country: '美国', countryCode: 'US' },
  { name: '多伦多', lat: 43.7, lng: -79.4, gdp: 5720, population: 290, flights: 450, country: '加拿大', countryCode: 'CA' },
  { name: '圣保罗', lat: -23.55, lng: -46.6, gdp: 4920, population: 2200, flights: 520, country: '巴西', countryCode: 'BR' },
  { name: '墨西哥城', lat: 19.4, lng: -99.1, gdp: 4230, population: 2100, flights: 480, country: '墨西哥', countryCode: 'MX' },
  { name: '开罗', lat: 30.0, lng: 31.2, gdp: 1820, population: 1980, flights: 310, country: '埃及', countryCode: 'EG' },
  { name: '内罗毕', lat: -1.29, lng: 36.8, gdp: 640, population: 440, flights: 180, country: '肯尼亚', countryCode: 'KE' },
  { name: '孟买', lat: 19.1, lng: 72.9, gdp: 3450, population: 2040, flights: 520, country: '印度', countryCode: 'IN' },
  { name: '首尔', lat: 37.6, lng: 127.0, gdp: 8450, population: 980, flights: 620, country: '韩国', countryCode: 'KR' },
  { name: '香港', lat: 22.3, lng: 114.2, gdp: 4960, population: 750, flights: 580, country: '中国', countryCode: 'CN' },
];
```

**航线数据：**
```
北京-上海、北京-东京、北京-伦敦、上海-纽约、上海-新加坡、东京-洛杉矶、伦敦-纽约、伦敦-巴黎、巴黎-迪拜、迪拜-新加坡、新加坡-悉尼、纽约-伦敦、纽约-洛杉矶、洛杉矶-东京、莫斯科-北京、迪拜-伦敦、悉尼-新加坡...
```

**时间轴数据：**
模拟2015-2024年的数据变化。对于每个城市，给出不同年份的GDP/人口等数据。

由于数据量问题，我可以在代码中生成模拟的时间序列数据。

让我完整规划代码结构：

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

// 1. 场景、相机、渲染器、控制器
// 2. 创建地球
// 3. 创建大气层
// 4. 创建星星背景
// 5. 添加城市数据点
// 6. 添加航线
// 7. 添加交互（点击、悬停）
// 8. 添加控制面板
// 9. 动画循环
```

让我进一步思考每个部分的实现细节。

**地球纹理：**
使用`https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg`作为白天纹理。
使用`https://unpkg.com/three-globe/example/img/earth-night.jpg`作为夜晚纹理。

用`MeshPhongMaterial`同时设置`map`和`lightMap`？不行，`lightMap`需要UV2。

更好的方式：使用自定义ShaderMaterial，混合白天和夜晚纹理，基于光照方向。

但更简单的方式：用两个Mesh，一个白天纹理，一个夜晚纹理，夜晚纹理由一个不透明度和光照方向控制。

让我用一个`ShaderMaterial`实现：

```javascript
const earthMaterial = new THREE.ShaderMaterial({
  uniforms: {
    dayTexture: { value: dayTexture },
    nightTexture: { value: nightTexture },
    sunDirection: { value: new THREE.Vector3(1, 0, 0) }
  },
  vertexShader: `
    varying vec2 vUv;
    varying vec3 vNormal;
    varying vec3 vPosition;
    void main() {
      vUv = uv;
      vNormal = normalize(normalMatrix * normal);
      vPosition = (modelViewMatrix * vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform sampler2D dayTexture;
    uniform sampler2D nightTexture;
    uniform vec3 sunDirection;
    varying vec2 vUv;
    varying vec3 vNormal;
    varying vec3 vPosition;
    
    void main() {
      vec3 n = normalize(vNormal);
      vec3 sunDir = normalize(sunDirection);
      float diff = dot(n, sunDir);
      
      vec4 dayColor = texture2D(dayTexture, vUv);
      vec4 nightColor = texture2D(nightTexture, vUv);
      
      // 白天/夜晚混合
      float mixFactor = smoothstep(-0.1, 0.1, diff);
      vec3 color = mix(nightColor.rgb, dayColor.rgb, mixFactor);
      
      // 简单Lambert光照增强
      float light = max(diff, 0.0);
      color *= 0.5 + 0.5 * light;
      
      gl_FragColor = vec4(color, 1.0);
    }
  `
});
```

但注意：shader中使用的太阳方向是固定的，当地球自转时，同一位置的明暗会变化，实现日夜效果。

太阳方向应该在世界空间中固定，比如`new THREE.Vector3(1, 0, 0)`。

**大气光晕：**

使用一个稍大的球体，用`ShaderMaterial`实现边缘发光：

```javascript
const atmosphereMaterial = new THREE.ShaderMaterial({
  uniforms: {
    sunDirection: { value: new THREE.Vector3(1, 0, 0) }
  },
  vertexShader: `
    varying vec3 vNormal;
    varying vec3 vPosition;
    void main() {
      vNormal = normalize(normalMatrix * normal);
      vPosition = (modelMatrix * vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    varying vec3 vNormal;
    varying vec3 vPosition;
    uniform vec3 sunDirection;
    void main() {
      vec3 viewDir = normalize(cameraPosition - vPosition);
      vec3 normal = normalize(vNormal);
      
      float rim = 1.0 - abs(dot(viewDir, normal));
      rim = pow(rim, 3.0);
      
      // 朝阳一侧更亮
      float sunFace = dot(normal, normalize(sunDirection)) * 0.5 + 0.5;
      
      vec3 atmColor = mix(vec3(0.3, 0.6, 1.0), vec3(0.8, 0.4, 0.2), sunFace);
      gl_FragColor = vec4(atmColor, rim * 0.8);
    }
  `,
  blending: THREE.AdditiveBlending,
  side: THREE.BackSide,
  transparent: true
});
```

**城市点：**

用`Sprites`更灵活。每个Sprite是一个小发光圆点。

加载纹理：
```javascript
function createGlowTexture() {
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
  gradient.addColorStop(0, 'rgba(255,255,255,1)');
  gradient.addColorStop(0.3, 'rgba(255,255,255,0.8)');
  gradient.addColorStop(1, 'rgba(255,255,255,0)');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, 64, 64);
  return new THREE.CanvasTexture(canvas);
}
```

每个城市一个Sprite：
```javascript
const spriteMaterial = new THREE.SpriteMaterial({
  map: glowTexture,
  color: 0xff6633,
  blending: THREE.AdditiveBlending,
  transparent: true,
  depthWrite: false
});
const sprite = new THREE.Sprite(spriteMaterial);
sprite.position.copy(latLngToVector3(lat, lng, radius * 1.01));
sprite.scale.set(size, size, 1);
```

**航线弧线：**

创建贝塞尔曲线：
```javascript
function createFlightPath(fromPos, toPos) {
  const mid = fromPos.clone().add(toPos).multiplyScalar(0.5);
  const dist = fromPos.distanceTo(toPos);
  mid.normalize().multiplyScalar(fromPos.length() + dist * 0.15); // 向外偏移
  const curve = new THREE.QuadraticBezierCurve3(fromPos, mid, toPos);
  return curve;
}
```

画线：
```javascript
const points = curve.getPoints(50);
const lineGeometry = new THREE.BufferGeometry().setFromPoints(points);
const lineMaterial = new THREE.LineBasicMaterial({ color: 0x00ccff, transparent: true, opacity: 0.6 });
const line = new THREE.Line(lineGeometry, lineMaterial);
```

**流动动画：**

可以使用`LineDashedMaterial`的`dashOffset`实现流动，或者使用自定义shader。

让我用`ShaderMaterial`实现流动效果：

```javascript
const flowMaterial = new THREE.ShaderMaterial({
  uniforms: {
    time: { value: 0 },
    color: { value: new THREE.Color(0x00ccff) }
  },
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
      // 沿曲线方向的流动效果
      float flow = fract(vUv.x - time);
      float alpha = smoothstep(0.0, 0.2, flow) * (1.0 - smoothstep(0.7, 1.0, flow));
      gl_FragColor = vec4(color, alpha * 0.8);
    }
  `,
  transparent: true,
  depthWrite: false,
  blending: THREE.AdditiveBlending
});
```

但是给`Line`设置UV不是直接的... 对于`BufferGeometry`的管线，UV需要手动计算。

让我用另一种方式：用`TubeGeometry`生成管道，然后通过UV做流动。

或者，更简单的方法：用多个小点（Sphere/Points）沿曲线移动：

```javascript
// 每段航线上有一些移动的点
class FlightAnimation {
  constructor(curve, count, color) {
    this.curve = curve;
    this.count = count;
    this.points = [];
    for (let i = 0; i < count; i++) {
      const sprite = new THREE.Sprite(glowSpriteMaterial);
      sprite.color = color;
      this.points.push(sprite);
      scene.add(sprite);
    }
    this.t = 0;
  }
  
  update(delta) {
    this.t += delta;
    for (let i = 0; i < this.count; i++) {
      const p = (this.t * 0.1 + i / this.count) % 1.0;
      const pos = this.curve.getPoint(p);
      this.points[i].position.copy(pos);
    }
  }
}
```

这种方式效果很好，但开销较大。

让我混合使用：画一条半透明的基线弧线，另外加几个流动的光点。

实际上，用`Line` + 自定义shader更优雅。我可以手动设置UV。

对于`Line`的`BufferGeometry`，我可以添加`uv`属性：

```javascript
const points = curve.getPoints(50);
const geometry = new THREE.BufferGeometry();
geometry.setAttribute('position', new THREE.Float32BufferAttribute(points.flatMap(p => [p.x, p.y, p.z]), 3));
// 创建UV属性：沿曲线的t值
const uvs = points.map((_, i) => [i / (points.length - 1), 0]);
geometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs.flat(), 2));
```

然后shader中`vUv.x`就是沿曲线的进度。

这样可以实现流动效果。

**热力着色（简化版）：**

对于每个城市，根据指标值在地球表面生成一个圆形渐变纹理（Canvas），然后将所有热力图合并为一个Canvas纹理，叠加到地球上。每个指标对应不同的热力尺寸和颜色。

但更简单的方案：使用Points系统渲染热力点。

让我用一个`Points`系统：
- 在球面上随机生成大量点（或在地球表面网格上）
- 每个点根据其到各城市的指标值加权计算颜色
- 形成热力图

这需要在地球表面生成采样点。我可以在球面上均匀分布一些点，比如用Fibonacci球面分布。

然后为每个采样点计算热力值：
```javascript
for (let i = 0; i < samplePoints.length; i++) {
  let value = 0;
  for (const city of cities) {
    const dist = samplePoint.distanceTo(cityPos);
    value += city.value / (dist * dist + epsilon);
  }
  // 根据value设置颜色
}
```

这计算量较大，但如果采样点不多（几千个），还是可行的。

让我实现这个简化版的热力效果。实际上，直接用Canvas在纹理上绘制会更快。

让我再想一种更简单的方式：创建一张热力纹理Canvas，在上面以每个城市为中心绘制径向渐变，然后作为覆盖层。

```javascript
function createHeatmapTexture(cities, metric) {
  const canvas = document.createElement('canvas');
  canvas.width = 1024;
  canvas.height = 512;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, 1024, 512);
  
  // 归一化数据
  const values = cities.map(c => c[metric]);
  const max = Math.max(...values);
  
  let heatmap = document.createElement('canvas');
  // ... 在1024x512的画布上，将经纬度映射为x/y坐标，绘制径向渐变
  for (const city of cities) {
    const x = (city.lng + 180) / 360 * 1024;
    const y = (90 - city.lat) / 180 * 512;
    const value = city[metric] / max;
    const radius = 40 * value + 10;
    const gradient = ctx.createRadialGradient(x, y, 0, x, y, radius);
    gradient.addColorStop(0, `rgba(255, 0, 0, ${0.5 * value})`);
    gradient.addColorStop(1, 'rgba(255, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(x - radius, y - radius, x + radius, y + radius);
  }
  
  return new THREE.CanvasTexture(canvas);
}
```

然后将这个热力纹理叠加到地球材质上。这可以在ShaderMaterial中实现：

```javascript
uniform sampler2D heatmapTexture;
varying vec2 vUv;
// ...
vec4 heat = texture2D(heatmapTexture, vUv);
color = mix(color, vec3(1.0, 0.3, 0.3), heat.a);
```

这是可行的！让我使用这种方法实现热力着色。

**日夜分界线：**

通过固定方向光位置实现。使用`ShaderMaterial`的地球会根据这个光照方向计算明暗。

实际上，我决定在Fragment Shader中计算密度：

```javascript
float diff = dot(normalize(vNormal), sunDirection);
float mixFactor = smoothstep(-0.15, 0.15, diff);
```

`mixFactor`接近0为夜晚，接近1为白天。晚上显示夜景纹理+城市灯光。

城市灯光效果：在暗面显示发光点。可以在Sprite上控制透明度：

```javascript
// 在动画循环中
sprite.material.opacity = 1.0 - mixFactor; // 暗面时更亮
```

但计算每个sprite的mixFactor需要知道其法线方向（与球心方向相同）。可以在创建时存储法线方向，然后在动画中计算。

**星星背景：**

使用`Points`系统，随机生成星星：

```javascript
const starsGeometry = new THREE.BufferGeometry();
const starsVertices = [];
for (let i = 0; i < 5000; i++) {
  const r = 200;
  const theta = Math.random() * Math.PI * 2;
  const phi = Math.acos(2 * Math.random() - 1);
  starsVertices.push(r * Math.sin(phi) * Math.cos(theta));
  starsVertices.push(r * Math.sin(phi) * Math.sin(theta));
  starsVertices.push(r * Math.cos(phi));
}
```

**控制面板：**

简单的HTML/CSS控制面板，放在右下角或左侧。

包含：
- 指标切换按钮：GDP、人口、航班量
- 时间轴（滑块）和播放按钮
- 可能的数据年份？

让我设计时间轴为年份2015-2024。

对于城市数据的年份变化，我可以生成模拟数据：

```javascript
// 每个城市有基准值和增长率，按年份计算
function getCityValue(city, metric, year) {
  const base = city[metric];
  const growth = 0.02; // 每年2%增长
  const years = year - 2015;
  return base * Math.pow(1 + growth, years);
}
```

**点击City显示信息弹窗：**

用CSS2DRenderer或简单的HTML overlay。

让我使用CSS2DRenderer来显示城市标签，用HTML overlay显示详细信息弹窗。

或者用HTML的div作为弹窗，定位在屏幕坐标。

用Raycaster检测点击：

```javascript
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

renderer.domElement.addEventListener('click', (event) => {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  
  // 检测城市sprites
  const intersects = raycaster.intersectObjects(citySprites);
  if (intersects.length > 0) {
    showInfoPopup(intersects[0].object);
  }
});
```

Raycaster对Sprite也有效，但要先检测sprite。

**悬停航线显示航班信息：**

用Raycaster检测航线（Line），需要将Line的`frustumCulled`设为false。

对Line做Raycaster检测：`raycaster.intersectObject(line)` 对Line是支持的，但需要在Line的`material`中设置`linewidth`（虽然大多数浏览器不支持）。

实际上，Line的raycast使用较粗糙，可能不够准确。可以改用`THREE.TubeGeometry` + `Mesh`来替代Line。

但为了简化，我仍然使用Line，并在悬停时显示一个提示框。如果检测不准确，可以设置`raycaster.params.Line.threshold = 1`。

**CSS2D标签：**

用CSS2DRenderer显示不需要3D定位的HTML标签。

我之前说过用CSS2DRenderer来显示城市名称。这样城市上一直有名字显示，但当数据太多时会拥挤。

让我改为：默认不显示标签，悬停城市时显示标签。

**总结最终实现方案：**

1. 地球：`ShaderMaterial`，白天+夜晚纹理，热力覆盖层
2. 大气层：`ShaderMaterial`，边缘发光
3. 城市点：`Sprite`对象，带发光纹理
4. 航线：`TubeGeometry`或`Line`+shader的流动效果
5. 交互：`OrbitControls` + Raycaster
6. 控制面板：HTML/CSS
7. 动画：地球自转 + 航线流动 + 时间轴

现在让我开始编写完整的代码。这将是相当长的一段代码。

让我写代码：

注意：由于Code length限制，我需要确保代码高效组织。但这是一个大型项目，需要完整实现。

让我仔细地一步一步写代码，确保一切功能正常。

首先，我考虑纹理加载。由于需要加载远程纹理，可能会跨域。需要确保CORS没问题。

Three.js使用ImageLoader加载纹理，在JavaScript中加载远程图片通常会遇到跨域问题。但unpkg CDN是允许CORS的。

但是，对于一个纯HTML文件，通过`<script type="importmap">`导入Three.js，然后加载外部纹理，通常是允许的（只要服务器允许CORS）。

让我用`https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg`作为纹理。

实际上，为了更可靠，我可以使用`TextureLoader`，并在加载失败时使用备用方案（纯色球体）。

让我准备代码。我从基础开始，逐层添加功能。

好，让我列举所有需要的纹理URL：
- 地球白天纹理: https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg
- 地球夜晚纹理: https://unpkg.com/three-globe/example/img/earth-night.jpg

其实三globle的纹理可能很好。

城市数据，我使用上面准备的数据。

航线数据，我准备约15条航线：

```javascript
const ROUTES = [
  { from: '北京', to: '上海', flights: 120 },
  { from: '北京', to: '东京', flights: 80 },
  { from: '北京', to: '伦敦', flights: 45 },
  { from: '上海', to: '纽约', flights: 30 },
  { from: '上海', to: '新加坡', flights: 90 },
  { from: '东京', to: '洛杉矶', flights: 65 },
  { from: '伦敦', to: '纽约', flights: 120 },
  { from: '伦敦', to: '巴黎', flights: 180 },
  { from: '巴黎', to: '迪拜', flights: 50 },
  { from: '迪拜', to: '新加坡', flights: 70 },
  { from: '新加坡', to: '悉尼', flights: 55 },
  { from: '纽约', to: '洛杉矶', flights: 150 },
  { from: '洛杉矶', to: '东京', flights: 45 },
  { from: '莫斯科', to: '北京', flights: 60 },
  { from: '悉尼', to: '新加坡', flights: 40 },
  { from: '圣保罗', to: '纽约', flights: 35 },
  { from: '开罗', to: '伦敦', flights: 40 },
  { from: '孟买', to: '迪拜', flights: 85 },
  { from: '首尔', to: '北京', flights: 70 },
  { from: '香港', to: '上海', flights: 100 },
];
```

好，让我开始编码。

这段代码会非常长，但我尽量实现全部功能。

考虑到复杂度，让我按模块组织代码。

让我再想想航线流动动画的实现。使用`Line` + 自定义shader：

```javascript
// 创建弧线
function createArc(fromPos, toPos) {
  const mid = fromPos.clone().add(toPos).multiplyScalar(0.5);
  const dist = fromPos.distanceTo(toPos);
  mid.normalize().multiplyScalar(1.0 + dist * 0.12); // 地球半径+偏移
  const curve = new THREE.QuadraticBezierCurve3(fromPos, mid, toPos);
  return curve;
}

// 为每条航线创建一个Line
const curve = createArc(fromPos, toPos);
const points = curve.getPoints(64);
const geometry = new THREE.BufferGeometry();
geometry.setAttribute('position', new THREE.Float32BufferAttribute(points.flatMap(p => [p.x, p.y, p.z]), 3));
const uvs = points.map((_, i) => [i / (points.length - 1), 0]);
geometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs.flat(), 2));

const material = new THREE.ShaderMaterial({
  uniforms: {
    time: { value: 0 },
    color: { value: new THREE.Color(0x00ccff) },
  },
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
      // 流动效果
      float flow = fract(vUv.x * 3.0 - time * 0.5);
      float alpha = smoothstep(0.0, 0.1, flow) * (1.0 - smoothstep(0.5, 0.7, flow));
      
      // 弧线渐隐
      float fade = sin(vUv.x * 3.14159);
      
      gl_FragColor = vec4(color, alpha * fade * 0.8);
    }
  `,
  transparent: true,
  depthWrite: false,
  blending: THREE.AdditiveBlending
});
```

这个效果好。航线还有底部基线：

```javascript
// 基线（在地球表面以下，更暗）
```

好，还有城市点。让我用Sprite实现。

**城市灯光效果：** 在暗面显示亮光。

对于每个城市，Sprite的透明度根据光照方向决定：

```javascript
// 每个sprite存储其位置的法线方向（即位置方向）
// 在动画循环中：
const sunDir = sunDirection.clone().normalize();
citySprite.material.opacity = cityValue * (1.0 - smoothstep(-0.1, 0.2, citySpritePositionNorm.dot(sunDir)));
```

但是`SpriteMaterial`的`opacity`是全局的，如果多个Sprite共享材质，会互相影响。所以每个城市要有独立的`SpriteMaterial`（或少量共享材质）。

让我为每个城市创建独立的SpriteMaterial实例（因为每个城市的颜色、大小可能不同）。

**点击弹窗：**

用HTML div，绝对定位。

```html
<div id="popup" class="city-popup">
  <div class="popup-title"></div>
  <div class="popup-content"></div>
  <button class="popup-close">×</button>
</div>
```

**控制面板：**

```html
<div class="control-panel">
  <div class="panel-title">🌍 数据地球</div>
  <div class="panel-section">
    <div class="section-label">数据指标</div>
    <div class="metric-buttons">
      <button class="metric-btn active" data-metric="gdp">GDP</button>
      <button class="metric-btn" data-metric="population">人口</button>
      <button class="metric-btn" data-metric="flights">航班量</button>
    </div>
  </div>
  <div class="panel-section">
    <div class="section-label">时间轴</div>
    <input type="range" id="timeSlider" min="2015" max="2024" step="1" value="2020">
    <div class="time-label">2020年</div>
    <button id="playBtn">▶ 播放</button>
  </div>
</div>
```

**热力着色实现细节：**

在`ShaderMaterial`中关联热力纹理。

```javascript
// 热力纹理
function createHeatmapTexture(cities, metric, year) {
  const canvas = document.createElement('canvas');
  canvas.width = 2048;
  canvas.height = 1024;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // 归一化
  const values = cities.map(c => getCityValue(c, metric, year));
  const max = Math.max(...values);
  
  // 对每个城市绘制径向渐变
  for (let i = 0; i < cities.length; i++) {
    const city = cities[i];
    const x = (city.lng + 180) / 360 * canvas.width;
    const y = (90 - city.lat) / 180 * canvas.height;
    const value = values[i] / max;
    const radius = 60 * value + 20;
    
    // 使用HSL渐变：从蓝色到红色
    const hue = (1 - value) * 0.7; // 蓝→绿→红
    const color = `hsla(${hue * 360}, 80%, 50%, ${0.6 * value})`;
    
    const gradient = ctx.createRadialGradient(x, y, 0, x, y, radius);
    gradient.addColorStop(0, color);
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(x - radius, y - radius, radius * 2, radius * 2);
  }
  
  return new THREE.CanvasTexture(canvas);
}
```

在地球ShaderMaterial中合并热力纹理：

```glsl
uniform sampler2D heatmapTexture;
varying vec2 vUv;
// ...
vec4 heat = texture2D(heatmapTexture, vUv);
color = color * (1.0 - heat.a) + heat.rgb * heat.a;
```

这样，热力区域会覆盖在地球表面。

**时间轴播放：**

播放按钮切换播放/暂停。播放时，年份从2015到2024循环。每次年份变化，更新：
- 城市点大小/颜色
- 热力纹理
- 航线透明度/速度

**地球自转：**

在动画循环中，地球绕Y轴缓慢旋转：
```javascript
earth.rotation.y += 0.001; // 每帧约0.001弧度
```

但同时用户交互（OrbitControls）会旋转相机。为避免冲突，自转和相机旋转是独立的。用户拖拽旋转的是相机，地球自转是地球模型在旋转。

好，这样设计没有问题。

**日夜分界线的太阳方向：**

设置世界空间中的一个固定方向光方向，比如 `new THREE.Vector3(1, 0, 0)`。地球自转时，不同经度会经历昼夜变化。

在ShaderMaterial中，太阳方向是世界空间中的方向。但由于地球自身在旋转，地球的顶点在旋转。我需要确保太阳方向是在世界空间，而不是地球局部空间。

让我把太阳方向作为uniform，在Shader中与变换后的法线（世界空间）计算点积：

```glsl
vWorldNormal = normalize(mat3(modelMatrix) * normal);
// 然后 dot(vWorldNormal, sunDir)
```

由于`modelMatrix`包含旋转，所以法线会随着地球旋转而旋转。这是正确的——地球旋转时，表面的法线在世界空间中旋转，与固定太阳方向（世界空间）的点积会变化，产生昼夜交替。

但注意：我刚看到Shader material中`modelMatrix`是在vertex shader中可用的。需要把世界法线传给fragment shader。

好，让我把法线变换到世界空间。

```glsl
varying vec3 vWorldNormal;
void main() {
  vWorldNormal = normalize(mat3(modelMatrix) * normal);
  // ...
}
```

然后：
```glsl
float diff = dot(normalize(vWorldNormal), sunDir);
```

**大气光晕的效果：**

大气光晕球体可以也考虑太阳方向，让朝阳侧更亮。

**城市灯光的可见性：**

每个城市的Sprite需要在夜晚侧更亮，白天侧几乎透明。

计算方式：
```javascript
// 在城市sprite位置判断昼夜
const cityWorldPos = sprite.getWorldPosition(new THREE.Vector3());
const normal = cityWorldPos.clone().normalize();
const sunDot = normal.dot(sunDirection);
const nightFactor = smoothstep(0.1, -0.1, sunDot); // 暗时0->1
sprite.material.opacity = 0.2 + 0.8 * nightFactor * normalizedValue;
```

但如果地球旋转，`sprite`作为地球的子对象，它的世界位置会变化。所以需要在每一帧重新计算。

我把sprite作为地球的子对象（`earth.add(sprite)`），这样sprite会随地球一起旋转。这没问题。

但计算sprite的世界位置时，需要调用`updateMatrixWorld`或使用`getWorldPosition`，这会自动考虑父级矩阵。

**航线也作为地球的子对象。**

好，让我把城市Sprties和航线都添加到earth对象上。

**交互细节：**

点击城市时，如果有弹窗打开，先关闭再打开新的。

悬停航线时，显示航线的航班信息。

**Raycaster对Sprite的检测：**

```javascript
raycaster.params.Points.threshold = 0.5;
```

对于Sprite，raycaster可以检测到，但需要传入`sprite`对象。

**CSS2DRenderer用于城市标签：**

是否使用CSS2DRenderer？如果需要城市标签，可以加载它。

```javascript
import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';
```

然后：
```javascript
const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight);
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = '0';
labelRenderer.domElement.style.pointerEvents = 'none';
document.body.appendChild(labelRenderer.domElement);
```

每个城市标签：
```javascript
const div = document.createElement('div');
div.className = 'city-label';
div.textContent = city.name;
const label = new CSS2DObject(div);
label.position.copy(cityPos);
earth.add(label);
```

但标签可能太多。让我仅在悬停或点击时显示标签。

或者默认不显示标签，点击城市后显示标签。

让我默认显示小标签，当指标切换时更新。实际上，这么多城市的标签会很拥挤，让我默认不显示标签，hover时显示。

好，让我简化：点击城市显示信息弹窗，同时hover某个城市时显示该城市名称的小标签。

**悬停航线显示航班信息：**

当鼠标悬停在航线上时，显示一个tooltip。

**现在让我开始写完整的代码。这是个大工程，让我逐步构建。**

由于代码量巨大，我会分部分写，但最终提供一个完整的HTML文件。

让我组织好代码结构。

首先，确定HTML结构：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>3D 数据可视化地球仪</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { overflow: hidden; background: #000; font-family: 'Segoe UI', sans-serif; }
  #canvas { display: block; }
  
  /* 控制面板 */
  .control-panel { position: absolute; right: 20px; top: 20px; background: rgba(15, 23, 42, 0.85); ... }
  
  /* 城市弹窗 */
  .city-popup { position: absolute; display: none; background: rgba(0,0,0,0.85); ... }
  
  /* 航空信息提示 */
  .route-tooltip { position: absolute; ... }
  
  /* 城市标签 */
  .city-label { color: #fff; font-size: 12px; ... }
</style>
</head>
<body>
<!-- UI元素 -->
<script type="importmap">...</script>
<script type="module">...</script>
</body>
</html>
```

我先实现主要部分，然后补充细节。

好的，我现在开始写代码。我会尽量在单次回复中完成全部代码。

让我认真写完。由于代码量可能超过正常的回复限制，我需要合理安排。但我会尽我所能。

让我尝试在合理范围内完成。核心功能都会实现，但可能有些视觉效果会简化。

最终我决定使用以下方案：

**城市数据：** 使用20个城市的模拟数据。

**航线：** 使用20条航线。

**热力：** 基于城市点的热力纹理叠加。

**控制面板：** 简单的CSS面板。

**时间轴：** 2015-2024。

让我开始写吧…

由于需要控制代码量，我会把重点放在核心功能的实现上。

首先让我理清楚所有代码的层次：

1. 场景设置
2. 地球（ShaderMaterial）
3. 大气层
4. 星星背景
5. 城市点（Sprites）
6. 航线（Lines+shader）
7. 太阳光（方向光）
8. 交互系统
9. 控制面板
10. 动画循环

接下来开始写代码。

有一点需要注意：城市点和管理系统。

每个城市对象：
```javascript
{
  name, lat, lng, gdp, population, flights,
  country, countryCode,
  sprite: THREE.Sprite,
  label: CSS2DObject  // 可选
}
```

在动画中，根据当前指标和时间：
- 更新sprite大小：`sprite.scale.set(v, v, 1)`，`v`基于归一化指标值
- 更新sprite颜色：基于值，从蓝到红渐变
- 更新sprite透明度：考虑昼夜

对于昼夜效果：
```javascript
const normal = cityPos.clone().normalize();
const sunDot = normal.dot(sunDirection);
const nightFactor = 1.0 - smoothstep(-0.1, 0.2, sunDot);
sprite.material.opacity = nightFactor * (0.3 + 0.7 * normalizedValue);
```

这样，夜晚一侧的城市灯光效果明显。

但白天也需要看到城市点（作为数据可视化），所以：
```javascript
sprite.material.opacity = 0.3 + 0.7 * normalizedValue; // 基础透明度
// 然后让夜晚更亮
sprite.material.opacity *= (0.3 + 0.7 * nightFactor);
```

**航线的昼夜效果：**
航线也可以根据起终点城市的昼夜来调整透明度。

**热力着色：**
在地球ShaderMaterial中叠加热力纹理。

**关于地球材质**：我决定使用`ShaderMaterial`，因为需要同时处理日夜纹理、热力。

好，开始编码。

我需要非常小心地处理ShaderMaterial，确保没有bug。

让我先定义一个最终的EarthShader：

```glsl
// Vertex Shader
varying vec2 vUv;
varying vec3 vWorldNormal;
varying vec3 vWorldPosition;

void main() {
  vUv = uv;
  vWorldNormal = normalize(mat3(modelMatrix) * normal);
  vWorldPosition = (modelMatrix * vec4(position, 1.0)).xyz;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}

// Fragment Shader
uniform sampler2D dayTexture;
uniform sampler2D nightTexture;
uniform sampler2D heatmapTexture;
uniform vec3 sunDirection;

varying vec2 vUv;
varying vec3 vWorldNormal;
varying vec3 vWorldPosition;

void main() {
  vec3 n = normalize(vWorldNormal);
  vec3 sunDir = normalize(sunDirection);
  float diff = dot(n, sunDir);
  
  vec4 dayColor = texture2D(dayTexture, vUv);
  vec4 nightColor = texture2D(nightTexture, vUv);
  
  // 昼夜混合
  float mixFactor = smoothstep(-0.1, 0.2, diff);
  vec3 baseColor = mix(nightColor.rgb, dayColor.rgb, mixFactor);
  
  // 光照
  float light = max(diff, 0.0);
  baseColor *= 0.4 + 0.6 * light;
  
  // 热力叠加
  vec4 heat = texture2D(heatmapTexture, vUv);
  vec3 color = mix(baseColor, heat.rgb, heat.a * 0.8);
  
  gl_FragColor = vec4(color, 1.0);
}
```

关于`normalMatrix`和`mat3(modelMatrix)`：在顶点着色器中，变换法线应该用`normalMatrix`（模型视图矩阵的逆转置矩阵），但我需要的是世界空间的法线，`mat3(modelMatrix)`可以近似（在没有非均匀缩放时）。由于地球是一个标准球体，没有非均匀缩放，所以这是OK的。

**创建热力纹理的函数：**

```javascript
function generateHeatmapTexture(cityData, metric, year) {
  const canvas = document.createElement('canvas');
  canvas.width = 2048;
  canvas.height = 1024;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // 计算数据范围
  const values = cityData.map(c => getValue(c, metric, year));
  const max = Math.max(...values);
  const min = Math.min(...values);
  const range = max - min || 1;
  
  cityData.forEach((city, i) => {
    const x = (city.lng + 180) / 360 * canvas.width;
    const y = (90 - city.lat) / 180 * canvas.height;
    const normValue = (values[i] - min) / range;
    const radius = 30 + normValue * 80; // 10~90像素
    
    // 颜色：蓝→绿→黄→红
    const hue = (1 - normValue) * 0.7; // 红=0, 蓝=0.7
    const color = `hsla(${hue * 360}, 70%, 55%, ${0.5 * normValue})`;
    
    for (let dx = -radius; dx <= radius; dx += 2) {
      for (let dy = -radius; dy <= radius; dy += 2) {
        const d = Math.sqrt(dx*dx + dy*dy);
        if (d <= radius) {
          const alpha = 0.5 * normValue * (1 - d/radius);
          ctx.fillStyle = `hsl(${hue * 360}, 70%, 55%)`;
          ctx.globalAlpha = alpha;
          ctx.fillRect(x + dx, y + dy, 2, 2);
        }
      }
    }
    ctx.globalAlpha = 1;
  });
  
  return new THREE.CanvasTexture(canvas);
}
```

这个O(radius^2 * cities)的操作，如果城市多、radius大，性能可能有问题（20个城市，平均radius 50，每个约10000像素点，循环约10000*20=200000次，还好）。

更高效的方式：使用canvas的径向渐变。

```javascript
cityData.forEach((city, i) => {
  const x = (city.lng + 180) / 360 * canvas.width;
  const y = (90 - city.lat) / 180 * canvas.height;
  const normValue = (values[i] - min) / range;
  const radius = 30 + normValue * 80;
  
  const hue = (1 - normValue) * 0.7;
  const gradient = ctx.createRadialGradient(x, y, 0, x, y, radius);
  gradient.addColorStop(0, `hsla(${hue * 360}, 70%, 55%, ${0.6 * normValue})`);
  gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
  ctx.fillStyle = gradient;
  ctx.beginPath();
  ctx.arc(x, y, radius, 0, Math.PI * 2);
  ctx.fill();
});
```

这更高效。好。

但使用热力纹理时，注意纹理映射到球面上可能变形。在低纬度（赤道附近）没问题，在高纬度（极地附近）会有拉伸。

**控制面板：**

让我用CSS实现一个美观的控制面板。

**弹窗：**

```html
<div id="cityPopup" style="display:none;">
  <div id="cityPopupTitle"></div>
  <div id="cityPopupContent"></div>
</div>
```

**Tooltip：**

```html
<div id="routeTooltip" style="display:none;"></div>
```

**运行方式：**

由于使用了`importmap`和ES模块，需要通过HTTP服务器访问，不能直接打开file://。

可以使用：
```
npx serve
python -m http.server
或者 VS Code Live Server
```

**测试说明：**

1. 拖拽旋转地球
2. 滚轮缩放
3. 点击城市查看信息
4. 悬停航线查看航班信息
5. 切换数据指标（GDP/人口/航班量）
6. 播放时间轴动画

好，我现在开始写最终代码。

让我注意几个方面：

1. 太阳方向向量：`new THREE.Vector3(1, 0, 0).normalize()`
2. 地球自转：`earth.rotation.y += 0.001`
3. 星空背景：一个大球体，内侧贴星星纹理，或简单的Points

让我把太阳方向设置为`(1, 0, 0)`。

**关于地球自转和太阳方向：**

本地球在自转，太阳方向固定在世界空间中。当地面某点转到朝向太阳时，该点为白天；转到背向太阳时，为夜晚。

在Vertex Shader中，`mat3(modelMatrix) * normal`给出了世界法线。太阳方向是世界空间中的固定方向。所以`dot(worldNormal, sunDir)`给出了该点的昼夜状态。

这是正确的。

**特殊情况：** 当用户拖拽旋转地球时，OrbitControls控制的是相机，不是地球。所以地球模型自身的旋转（自转）保持不变，当用户旋转视角时，看到的地球方向变化，但昼夜线始终固定在太阳方向的一侧。

好的。

让我们开始写完整的代码吧。

考虑到要加入航线信息弹窗，需要确保航线对象有`userData`存储from/to和flights数据。

**航线动画（流动）：**

对于`ShaderMaterial`，我需要传递`time` uniform。在animation loop中更新。

```javascript
routeLines.forEach(route => {
  route.line.material.uniforms.time.value = elapsedTime;
});
```

好，现在我来构建最终的HTML文件。

由于代码量太大，我需要先列出重点，然后完善。

让我试着开始书写。

注意：对于热力纹理的更新，每次切换指标或时间轴变化时都需要重新生成CanvasTexture。这可能导致卡顿。优化：生成纹理的尺寸适当（如1024x512），并且在时间轴播放时，每帧只更新一次。

还需要注意内存：旧的纹理应该`dispose()`。

让我把热力纹理作为地球材质（ShaderMaterial）的uniform。

好，所有的设计已经就绪。我现在开始写代码。

为了代码可读性，我加入注释。

开始写吧... 这将会是一个大段代码。

让我留点时间整理City/Route数据的组织。

城市数据使用 `const CITIES = [...]`。

对于时间序列，我需要一个函数：
```javascript
function getValue(city, metric, year) {
  const base = city[metric];
  const growthRates = {
    gdp: 0.03,
    population: 0.01,
    flights: 0.02
  };
  const growth = growthRates[metric];
  return base * Math.pow(1 + growth, year - 2015);
}
```

然后年份由timer控制，从2015到2024。

**城市sprite大小和颜色：**

```javascript
function updateCityVisuals(metric, year) {
  const values = CITIES.map(c => getValue(c, metric, year));
  const max = Math.max(...values);
  const min = Math.min(...values);
  
  CITIES.forEach((city, i) => {
    const norm = (values[i] - min) / (max - min);
    const size = 0.5 + norm * 3.0;
    city.sprite.scale.set(size, size, 1);
    
    const hue = (1 - norm) * 0.7; // 红到蓝
    city.sprite.material.color.setHSL(hue, 0.8, 0.6);
  });
  
  // 更新热力纹理
  const heatmapTexture = generateHeatmapTexture(CITIES, metric, year);
  earthMaterial.uniforms.heatmapTexture.value = heatmapTexture;
  // Dispose old texture
  if (currentHeatmapTexture) currentHeatmapTexture.dispose();
  currentHeatmapTexture = heatmapTexture;
}
```

**航线宽度/颜色：**

```javascript
function updateRouteVisuals(metric, year) {
  ROUTES.forEach(route => {
    const fromCity = CITIES.find(c => c.name === route.from);
    const toCity = CITIES.find(c => c.name === route.to);
    const v = (getValue(fromCity, metric, year) + getValue(toCity, metric, year)) / 2;
    const norm = v / maxVal;
    route.line.material.uniforms.color.value.setHSL(0.5 - norm * 0.5, 1, 0.5);
  });
}
```

好，我意识到这里要处理很多细节。让我集中精力写完整代码。

由于代码长度限制，我会把代码压缩到尽可能简洁但功能完整。

Let me start coding.

实际上，关于航线悬停检测：`THREE.Raycaster`对`THREE.Line`支持有限。要检测Line，可以使用：
```javascript
raycaster.params.Line.threshold = 0.5;
```

这应该可以在一定距离内检测到线。

但为了更准确的悬停，我可以用`THREE.Mesh` + `TubeGeometry`：

```javascript
const tubeGeometry = new THREE.TubeGeometry(curve, 64, 0.01, 4, false);
const tubeMesh = new THREE.Mesh(tubeGeometry, new THREE.MeshBasicMaterial({ color: 0x00ccff }));
```

但TubeGeometry的UV不好设置，流动效果需要自定义shader。

让我在航线的tube mesh上使用自定义shader，处理UV流动。

实际上，为了让UV在管线上分布，TubeGeometry会自动生成UV，但可能不是沿曲线的。让我在自定义shader中重新计算。

或者，我可以用TubeGeometry的`uv[0]`表示沿轨迹的位置（因为TubeGeometry会为每个环分配相同的uv.x）。

让我测试一下。根据Three.js文档，TubeGeometry的UV：
- U沿管线方向（0到1）
- V围绕管线（0到1）

所以`vUv.x`就是沿曲线的进度。太好了。

所以我可以使用TubeGeometry来替代Line，这样拉线检测更准确。

但需要更新航线外观：透明、发光。

好，让我使用TubeGeometry + ShaderMaterial。

对流动画：在shader中使用`vUv.x`。

```glsl
float flow = fract(vUv.x * 5.0 - time * 0.8); // 5条流动光带
float alpha = smoothstep(0.0, 0.15, flow) * (1.0 - smoothstep(0.4, 0.6, flow));
```

这样可以看到光带沿着管线流动。

同时，整个航线的透明度由`vUv.x`控制（在两端渐隐）：
```glsl
float fade = sin(vUv.x * 3.14159);
```

最终：
```glsl
gl_FragColor = vec4(color, alpha * fade * 0.8);
```

好，方案定了。

但使用TubeGeometry还有一个好处：可以与Raycaster精确检测。

```javascript
const ray = new THREE.Raycaster();
ray.params.Mesh.threshold = 0.05;
const intersects = ray.intersectObjects(routeMeshes);
```

好的，让我把航线实现为TubeGeometry + Mesh。

现在开始详细代码... 这会是一个很长的文件。

让我估算代码行数：
- CSS: ~100行
- HTML: ~30行  
- JS核心逻辑: ~600-800行

总计约 900-1000 行。

好，开始写吧！

让我想一想城市Sprites的Raycaster检测。Sprite的raycast在Three.js中是支持的。检测结果是先返回sprite对象。

但由于城市之间有重叠，可能不太好点准确。我可以在raycast时，只检测城市sprites，按距离排序取最近的。

```javascript
const cityIntersects = raycaster.intersectObjects(citySprites);
if (cityIntersects.length > 0) {
  const city = cityIntersects[0].object;
  showCityPopup(city);
}
```

好。

让我把`cityIntersects`跟`routeIntersects`分开检测。

**关于CSS2DRenderer：**
为了显示城市标签，我用CSS2DRenderer。但这样需要额外的渲染器，增加复杂度。让我用HTML div + 投影到屏幕坐标的方式：

在动画循环中，对于hover的城市，计算其屏幕位置，设置div的位置。

```javascript
function screenPosition(worldPos) {
  const vec = worldPos.clone().project(camera);
  return {
    x: (vec.x * 0.5 + 0.5) * window.innerWidth,
    y: (-vec.y * 0.5 + 0.5) * window.innerHeight,
  };
}
```

这更简单，不需要CSS2DRenderer。

好，方案更新：不使用CSS2DRenderer，用纯DOM + 投影。

现在让我想想完整的城市数据结构。

由于每个城市需要：
- 基础数据：lat, lng, gdp, population, flights
- 3D对象：sprite, labelDOM
- 动态数据：当前指标值

让我定义一个类或普通对象。

另外，点击城市弹窗需要显示：
- 城市名
- 国家
- GDP（当前年份）
- 人口（当前年份）
- 航班量（当前年份）

好。现在开始写代码吧。我会尽量一次性完成。

让我用以下技术选型：
- Three.js 0.160.0
- OrbitControls
- 无其他依赖

开始吧。

等一下，让我再考虑一下星空背景。简单的Points方式：

```javascript
function createStars() {
  const geometry = new THREE.BufferGeometry();
  const vertices = [];
  for (let i = 0; i < 5000; i++) {
    const r = 500;
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    vertices.push(r * Math.sin(phi) * Math.cos(theta));
    vertices.push(r * Math.sin(phi) * Math.sin(theta));
    vertices.push(r * Math.cos(phi));
  }
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
  const material = new THREE.PointsMaterial({ color: 0xffffff, size: 0.5, transparent: true });
  const stars = new THREE.Points(geometry, material);
  scene.add(stars);
}
```

好。

**关于canvas尺寸和响应：**

```javascript
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
window.addEventListener('resize', onWindowResize);
```

好，开始写代码。

等待，关于城市Sprite的scale。由于地球半径是2，一个城市的sprite大小需要很小。让我设置scale为0.05到0.2之间。

地球半径：我设置为2。

城市的3D位置：
```javascript
const R = 2;
function latLngToVector3(lat, lng, radius) {
  const phi = (90 - lat) * Math.PI / 180;
  const theta = (lng + 180) * Math.PI / 180;
  return new THREE.Vector3(
    -radius * Math.sin(phi) * Math.cos(theta),
    radius * Math.cos(phi),
    radius * Math.sin(phi) * Math.sin(theta)
  );
}
```

城市sprite的位置在半径约2.01处（稍微高出地球表面，避免z-fighting）。

航线弧线的控制点偏移：
```javascript
function createArc(fromPos, toPos, earthRadius = 2) {
  const mid = fromPos.clone().add(toPos).multiplyScalar(0.5);
  const dist = fromPos.distanceTo(toPos);
  const elevation = earthRadius + dist * 0.1; // 弧线高度
  mid.normalize().multiplyScalar(elevation);
  return new THREE.QuadraticBezierCurve3(fromPos, mid, toPos);
}
```

我可以加上`const elevation = 2 + dist * 0.15;`。

好。

让我开始写吧。最终代码会很长，但我会尽量完整。

由于代码量限制在正常回复范围内（约2000行），应该可以完成。

Let me write it now.

我还有一点需要说明：航线数据中的航班量是每日航班数，可以用来控制航线的显示粗细/颜色。

关于航线hover显示的信息：
```
航班信息：
X→Y
航班量：120班/日
飞行距离：XXXX km
```

距离可以估算：
```javascript
function haversineDistance(lat1, lng1, lat2, lng2) {
  const R = 6371;
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLng = (lng2 - lng1) * Math.PI / 180;
  const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180) * Math.cos(lat2*Math.PI/180) * Math.sin(dLng/2)**2;
  return 2 * R * Math.asin(Math.sqrt(a));
}
```

好。现在开始写。

在动画循环中，需要更新很多内容：
- 地球自转
- 航线流动
- 城市灯光（昼夜判断）
- 时间轴播放（如果正在播放）
- 星空旋转（可选）

让我把逻辑都放在`animate()`函数中。

写吧。

关于控制面板的时间轴UI，让我设计：

```html
<div class="panel-section">
  <div class="section-label">时间轴</div>
  <div class="time-range">
    <span>2015</span>
    <input type="range" id="timeSlider" min="2015" max="2024" value="2020" step="1">
    <span>2024</span>
  </div>
  <div class="time-current">
    <span id="currentYear">2020</span>
    <button id="playBtn">▶ 播放</button>
  </div>
</div>
```

让我CSS美化。

好的，现在开始编写完整的代码。由于非常长，我会注意排版和注释。

让我确保每个城市对象的sprite有正确的`userData`存储城市数据引用。

最后，让我再想想一些细节：

**航线流动的方向：**从起始城市到终点城市，UV.x从0→1，所以流动方向正确。

**航线Base Line：**除了弧线之外，我可能还想在地球表面上显示一条微弱的大圆线连接两个城市。但这会增加复杂度，让我省略。

**底部基线**：可以不画。弧线本身已经足够。

**航线数量：** 20条。每条航线一个TubeGeometry，64段，4个径向分段。总共 64*4*2*20 ≈ 10240个三角形，非常轻量。

**城市Sprite数量：** 20个。非常轻量。

好，开始吧。

让我整理一下HTML结构：

```html
<body>
  <div id="container"></div>
  <div class="control-panel">...</div>
  <div id="cityPopup">...</div>
  <div id="routeTooltip">...</div>
</body>
```

Three.js渲染器挂载到container上。

让我写最终代码。

代码中的关键部分:

1. 初始化场景、相机、渲染器、控制器
2. 加载纹理
3. 创建地球
4. 创建大气层
5. 创建星空
6. 创建城市
7. 创建航线
8. 交互绑定
9. 控制面板逻辑
10. 动画循环

让我注意时间轴播放逻辑：

```javascript
let isPlaying = false;
let currentYearIndex = 0;
let animationTime = 0;

function togglePlay() {
  isPlaying = !isPlaying;
  playBtn.textContent = isPlaying ? '⏸ 暂停' : '▶ 播放';
}

// 在动画循环中
if (isPlaying) {
  animationTime += delta;
  if (animationTime > 1.0) {
    animationTime -= 1.0;
    // 每年推进
    currentYear++;
    if (currentYear > 2024) currentYear = 2015;
    timeSlider.value = currentYear;
    updateDataVisualization(currentMetric, currentYear);
  }
}
```

或者更简单：
```javascript
if (isPlaying) {
  animationTime += delta;
  if (animationTime > 2) { // 每2秒前进一年
    animationTime -= 2;
    currentYear++;
    if (currentYear > 2024) currentYear = 2015;
    timeSlider.value = currentYear;
    currentYearLabel.textContent = currentYear + '年';
    updateDataVisualization(currentMetric, currentYear);
  }
}
```

好。

现在让我编写最终的完整代码。这是一个很大的文件，我尽量做到完善。

让我也考虑一下城市灯光只在夜晚显示。关于夜景的显示策略：

- 当城市处于夜晚时，Sprite的透明度增加（发光）
- 当城市处于白天时，Sprite的透明度降低（但仍可见，作为数据点）

所以Sprite的最终opacity：
```javascript
const normalizedValue = ...; // 0~1
const nightFactor = ...; // 0~1
const opacity = 0.15 + 0.35 * normalizedValue + 0.5 * nightFactor * normalizedValue;
```

这样：
- 夜间城市：opacity = 0.15 + 0.35*v + 0.5*v = 0.15 + 0.85v
- 白天城市：opacity = 0.15 + 0.35*v

很好。

**热力纹理的更新频率：** 每次指标/年份变化时更新。由于生成纹理是CPU密集操作，我缓存结果。

对于年份变化，考虑到不会有太多的热力数据变化，每次更新是可以接受的。

让我设置热力纹理尺寸为1024x512（更小更快）。

好的，全部准备好了。现在写吧。

可能我在写的时候，由于代码太长，需要分几部分完成。我会在最终输出中提供完整的一个HTML文件。

Let me code it now...

为了避免代码过长导致截断，我尽量精简代码：

- 减少CSS冗余
- 减少数据量（20城市+15航线）
- 使用简洁的JS代码

但仍保持所有功能。

好，我开始了。这是一个巨大的编码任务，我会认真完成。让我不要犹豫，直接开始写吧。
