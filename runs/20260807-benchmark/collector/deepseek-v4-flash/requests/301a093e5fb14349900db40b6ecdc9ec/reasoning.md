用户想要一个完整的3D数据可视化地球仪，使用Three.js实现。让我创建一个单一HTML文件，包含所有功能：

1. 3D地球 - 真实纹理、旋转交互、大气光晕
2. 数据可视化层 - 城市标记、航线弧线、热力着色
3. 交互 - 拖拽旋转、滚轮缩放、点击城市弹窗、悬停航线显示信息
4. 控制面板 - 数据指标切换、时间轴
5. 视觉效果 - 自转动画、日夜分界线、城市灯光

我需要使用Three.js + importmap + ESM，所有代码在一个HTML文件中。

让我设计这个应用：

**纹理：**
- 使用在线免费的地球纹理（如NASA或类似来源）
- 昼夜纹理切换（日间纹理和夜间灯光纹理）

**城市数据：**
- 包含一些主要城市及其经纬度、GDP、人口、航班量等数据

**航线数据：**
- 城市之间的航线连接

**热力着色：**
- 根据指标对国家/地区进行着色

让我开始构建：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>3D数据可视化地球仪</title>
    <style>
        /* CSS */
    </style>
</head>
<body>
    <!-- 控制面板 -->
    <!-- Three.js 应用 -->
    <script type="importmap">
    {
        "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
            "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
        }
    }
    </script>
    <script type="module">
        // 应用代码
    </script>
</body>
</html>
```

让我思考下如何处理各个部分：

1. **地球** - 使用SphereGeometry + MeshPhongMaterial，加载地球纹理
2. **大气光晕** - 使用自定义shader或简单的半透明球体
3. **日夜分界线** - 可以通过shader实现，或使用自定义材质
4. **城市标记** - 将经纬度转换为3D坐标，在球面上放置发光点
5. **航线弧线** - 使用QuadraticBezierCurve3或类似曲线，动画流动效果
6. **热力着色** - 根据数据修改球面材质颜色（可以使用第二层球体）

让我考虑使用一个简化版但功能完整的实现。

对于热力着色，我可以创建一个覆盖在地球表面的半透明球体，根据国家数据改变颜色。

对于日夜分界线，我可以使用shader材质，根据太阳方向计算光照/黑暗。

让我使用更简单的方式实现日夜效果：使用两个球体，一个用白天地图纹理，一个用夜晚灯光纹理，通过光照/透明度混合。

实际上，让我用更直接的方式：
- 地球用MeshPhongMaterial，加一个平行光模拟太阳
- 夜晚灯光可以作为一个单独的球体，在背对太阳的一侧显示

或者使用一个自定义shader来同时处理白天/夜晚纹理。

让我尽量简化但保持效果：

- 使用MeshPhongMaterial + 地球纹理（白天）
- 夜晚灯光：单独一个球体，使用夜晚纹理，在背光面显示

让我用Three.js的渐变纹理或简单的方案：

对于日夜分界线，可以用一个自定义shader：
- 根据世界位置与太阳方向的夹角决定使用白天还是夜晚纹理

这样比较优雅。让我实现这个。

**纹理URL：**
- 白天地图：https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg
- 夜晚灯光：https://unpkg.com/three-globe/example/img/earth-night.jpg
- 或者使用其他免费源

让我使用 three-globe 示例中的纹理：

```js
const dayTexture = new THREE.TextureLoader().load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg');
const nightTexture = new THREE.TextureLoader().load('https://unpkg.com/three-globe/example/img/earth-night.jpg');
```

**城市数据：**
我可以硬编码一些城市：

```js
const cities = [
    { name: '北京', country: '中国', lat: 39.9, lon: 116.4, gdp: 4830, population: 2189, flights: 2800 },
    { name: '上海', country: '中国', lat: 31.2, lon: 121.5, gdp: 4460, population: 2487, flights: 2600 },
    // ...
];
```

**航线数据：**
城市之间的航线连接。

**时间轴：**
模拟数据随时间变化，可以用随机生成的数据或简单的插值。

让我开始写代码吧。这会是一个很大的文件，但我尽量组织清晰。

让我想想热力着色的实现：

方案1：在球面上绘制纹理（需要Canvas）
方案2：使用一个半透明有色球体，基于每个国家的面积着色

最简单的方式：使用Canvas创建一个纹理，在地球上划分国家区域并着色。但这需要GeoJSON数据。

更简单的方式：不实现真正的国家热力，而是用一个粒子系统或点云来代表热力分布。

让我采用更实际的方法：使用一个半透明的第二球体，其纹理是一个基于国家边界的热力图。但这需要GeoJSON数据。

另一种方案：把热力着色简化为在球面上用点/粒子表示不同值，用颜色编码。或者在城市位置放置带颜色的光晕。

让我重新考虑：要求是"国家/地区热力着色"，需要能看到国家级别的颜色变化。

实现方式：
1. 加载世界各国的GeoJSON数据（从CDN获取）
2. 将国家边界绘制到Canvas上，生成纹理
3. 将纹理应用到地球表面

但这比较复杂。让我看看有没有现成的方案。

或者，我可以使用一种简化的热力着色：
- 在球面上叠加一个半透明层
- 用Canvas绘制一个简化的世界地图，根据每个国家/地区的数据着色

让我使用一个简化的世界地图数据。我可以使用一个已有的世界地图图片，然后在地球上叠加。

实际上，让我换一种思路。我可以使用一个较小的、简化的GeoJSON来生成热力纹理。

让我使用 world-atlas 或者类似的TopoJSON数据，在Canvas上绘制国家并着色。

为了简单，让我用这个方案：
1. 加载世界地图GeoJSON（例如从 https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson）
2. 用Canvas绘制地图并填充颜色
3. 作为纹理贴到地球材质上

但这需要异步加载和处理。

让我简化一下：我可以使用一个预生成的包含热力信息的图片，或者直接用代码绘制。

让我用一个更简单但视觉效果不错的方法：
- 在球面上添加一些有颜色的点/区域来代表热力
- 或者使用一个半透明球体，用噪声/简单图案表示热力分布

实际上，最真实的方式是加载GeoJSON并绘制到Canvas。让我试试。

我考虑以下简化方案：

创建一个Canvas，在上面绘制世界地图的轮廓，然后根据每个国家/地区的数据填充颜色。

我需要：
1. 获取世界地图的GeoJSON或TopoJSON
2. 将经纬度投影到Canvas平面坐标
3. 填充颜色

但地球是3D的，这个2D Canvas如何映射到3D球体上？

答：将Canvas作为纹理贴到SphereGeometry上即可。Three.js会自动将2D纹理映射到3D球体。

所以步骤如下：
1. 加载GeoJSON数据
2. 创建Canvas，在上面绘制世界地图，用数据着色的颜色填充每个国家
3. 将Canvas作为纹理贴到地球表面

这样就可以了！

但考虑到代码复杂度和文件大小，让我看看是否能用更简单的方式。

实际上，让我用另一种更简单直观的方式：在地球上叠加一层半透明的、根据国家数据着色的球面层。使用顶点着色或贴图。

让我直接使用Canvas绘制热力图，作为地球纹理的叠加层。

让我用一个简化的世界地图数据源，或者甚至用一个离线生成的简单图案来模拟国家热力。

好吧，让我尝试更实用的方法：用一个包含国家数据的简化GeoJSON。

这里的关键是简化，因为完整的世界GeoJSON很大。让我找一个比较小的。

可以用这个：https://raw.githubusercontent.com/nvkelso/natural-earth-vector/master/geojson/ne_110m_admin_0_countries.geojson

这个文件大约几百KB，但可以通过网络加载。

让我试试加载它并在Canvas上绘制。

但问题是，对于热力着色，我需要每个国家的数据值。我可以为每个国家分配一个随机或模拟的值。

让我先不考虑加载外部GeoJSON文件，而是使用一个更简单的方法：

使用一个已经绘制好国家边界的地图图片（PNG），在上面做热力覆盖。但我需要知道每个国家的像素位置来着色。

让我换一种简化方式：直接在地球纹理的Canvas上，用国家边界数据绘制并着色。

好吧，让我采用一种折中方案：

**我将在运行时动态生成一个包含国家边界的地图Canvas，然后根据模拟数据着色。**

为此，我需要一个GeoJSON数据源。

让我使用：
```
https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json
```

这是一个相对较小的文件，包含各国边界。

让我整合这个方案。

**实现步骤：**

1. 加载地球纹理（白天和夜晚）
2. 加载GeoJSON数据
3. 创建一个Canvas，绘制世界地图，根据指标数据着色
4. 将Canvas作为第二层纹理，与地球纹理混合

等等，这太复杂了。让我简化：

**简化方案：**

1. 使用MeshPhongMaterial加载地球纹理（白天地图）
2. 夜晚灯光：使用自定义shader或第二层球体
3. 热力着色：使用一个叠加在球面上的半透明纹理球体，颜色根据数据

让我直接用一个Canvas来绘制热力纹理：

```js
// 创建热力纹理
function createHeatmapTexture(data) {
    const canvas = document.createElement('canvas');
    canvas.width = 1024;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');
    
    // 绘制背景
    ctx.fillStyle = 'rgba(0,0,0,0)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // 绘制每个国家的热力颜色
    data.countries.forEach(country => {
        // 找到国家的路径并填充颜色
    });
    
    return new THREE.CanvasTexture(canvas);
}
```

然后把这个纹理贴到一个半透明的球体上，覆盖在地球上。

但为了绘制国家，我需要GeoJSON路径。

让我直接加载GeoJSON并绘制。

好吧，让我正式构建。这个任务很复杂，但我尽可能做好。

让我梳理一下完整的实现：

**核心架构：**

```
- 场景、相机、渲染器
- 地球组（包含地球球体、大气光晕、热力层、城市标记、航线弧线）
- 光照（太阳光）
- 控制（拖拽旋转、滚轮缩放）
- 交互（射线检测点击城市、悬停航线）
- 控制面板（指标切换、时间轴）
- 动画循环
```

**数据模型：**

```js
const countryData = {
    'China': { gdp: 17.7, population: 1412, flights: 5800, ... },
    'US': { gdp: 25.5, population: 333, flights: 7200 },
    // ...
};

const cities = [
    { name: 'Beijing', country: 'China', lat: 39.9, lon: 116.4, ... },
    // ...
];
```

**航线数据：**

```js
const routes = [
    { from: 0, to: 1, flights: 1200 }, // 城市索引
    // ...
];
```

让我开始编码。

由于代码量很大，我需要仔细组织。

让我分步实现：

**第一步：基础Three.js场景**

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

// 简化：使用CSS2DRenderer做标签，或者用sprite
```

实际上，对于城市标签，我可以使用Sprite或CSS2DRenderer。但为了保持简单，让我使用Sprite。

或者，点击城市显示HTML弹窗，不需要3D标签。

让我用HTML弹窗。

**第二步：地球**

```js
// 地球纹理
const dayTexture = new THREE.TextureLoader().load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg');
const nightTexture = new THREE.TextureLoader().load('https://unpkg.com/three-globe/example/img/earth-night.jpg');

// 地球材质
const earthMaterial = new THREE.MeshPhongMaterial({
    map: dayTexture,
    bumpScale: 0.5,
    specular: new THREE.Color('grey'),
    shininess: 5,
});

const earth = new THREE.Mesh(
    new THREE.SphereGeometry(2, 64, 64),
    earthMaterial
);
```

**日夜分界线实现：**

我可以使用自定义ShaderMaterial：

```glsl
// vertex shader
varying vec2 vUv;
varying vec3 vNormal;
varying vec3 vPosition;
void main() {
    vUv = uv;
    vNormal = normalize(normalMatrix * normal);
    vPosition = (modelMatrix * vec4(position, 1.0)).xyz;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}

// fragment shader
uniform sampler2D dayTexture;
uniform sampler2D nightTexture;
uniform vec3 sunDirection;
varying vec2 vUv;
varying vec3 vNormal;
varying vec3 vPosition;
void main() {
    float intensity = dot(normalize(vNormal), sunDirection);
    intensity = smoothstep(-0.2, 0.2, intensity);
    vec3 dayColor = texture2D(dayTexture, vUv).rgb;
    vec3 nightColor = texture2D(nightTexture, vUv).rgb;
    vec3 color = mix(nightColor * 1.5, dayColor, intensity);
    gl_FragColor = vec4(color, 1.0);
}
```

这样，面向太阳的区域显示白天纹理，背向显示夜晚灯光纹理。

但`sunDirection`需要从世界空间转换到模型空间计算。因为这个shader是在世界空间计算的，我直接传入世界空间的太阳方向。

不过，地球会自转。为了让太阳相对地球移动，我需要固定太阳方向在世界空间，而地球旋转。

**太阳光效：**
- 平行光从太阳方向照射
- 地球自转时，日夜分界线会移动

让我把太阳方向固定在世界空间，与平行光一致。

**大气光晕：**

使用一个稍大的半透明球体，用shader或者简单的材质：

```js
const atmosphereMaterial = new THREE.ShaderMaterial({
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
            float intensity = pow(0.6 - dot(vNormal, vec3(0, 0, 1.0)), 2.0);
            gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
        }
    `,
    blending: THREE.AdditiveBlending,
    side: THREE.BackSide,
    transparent: true,
});
```

这个shader有点问题，因为normalMatrix是视图空间的。让我修正。

更简单的方式：使用一个外侧发光的球体，通过绘制背面实现：

```js
const atmosphere = new THREE.Mesh(
    new THREE.SphereGeometry(2.1, 64, 64),
    atmosphereMaterial
);
```

**第三步：控制（OrbitControls）**

```js
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.minDistance = 3;
controls.maxDistance = 8;
controls.autoRotate = true;
controls.autoRotateSpeed = 0.5;
```

**第四步：城市标记**

将经纬度转为3D坐标：

```js
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * Math.PI / 180;
    const theta = (lon + 180) * Math.PI / 180;
    return new THREE.Vector3(
        -radius * Math.sin(phi) * Math.cos(theta),
        radius * Math.cos(phi),
        radius * Math.sin(phi) * Math.sin(theta)
    );
}
```

城市标记用小的发光球体或Sprite：

```js
const cityMarker = new THREE.Mesh(
    new THREE.SphereGeometry(0.02, 16, 16),
    new THREE.MeshBasicMaterial({ color: 0x00ffaa })
);
cityMarker.position.copy(latLonToVector3(city.lat, city.lon, 2.01));
```

再加一个光晕效果：

```js
const glowSprite = new THREE.Sprite(
    new THREE.SpriteMaterial({
        map: glowTexture,
        color: 0x00ffaa,
        transparent: true,
        blending: THREE.AdditiveBlending,
    })
);
cityMarker.add(glowSprite);
```

**第五步：航线弧线**

```js
function createFlightArc(fromLat, fromLon, toLat, toLon) {
    const start = latLonToVector3(fromLat, fromLon, 2.01);
    const end = latLonToVector3(toLat, toLon, 2.01);
    const mid = start.clone().add(end).multiplyScalar(0.5);
    const distance = start.distanceTo(end);
    mid.normalize().multiplyScalar(2.01 + distance * 0.15);
    
    const curve = new THREE.QuadraticBezierCurve3(start, mid, end);
    const points = curve.getPoints(50);
    const geometry = new THREE.BufferGeometry().setFromPoints(points);
    const material = new THREE.LineBasicMaterial({
        color: 0x00aaff,
        transparent: true,
        opacity: 0.6,
    });
    return new THREE.Line(geometry, material);
}
```

**流动动画：**

可以在弧线上添加移动的小点。使用`getPointAt`方法：

```js
const flowPoints = [];
for (let i = 0; i < 10; i++) {
    const flowMaterial = new THREE.MeshBasicMaterial({ color: 0x00ffff });
    const flowPoint = new THREE.Mesh(new THREE.SphereGeometry(0.01, 8, 8), flowMaterial);
    flowPoints.push(flowPoint);
}
// 在动画循环中更新位置
flowPoints.forEach((point, index) => {
    const t = (time * speed + index / 10) % 1;
    point.position.copy(curve.getPointAt(t));
});
```

**第六步：热力着色**

让我加载GeoJSON并创建一个Canvas纹理。

但这需要很多代码。让我用一个简化的方法：用一个Canvas创建一个热力图纹理，使用GeoJSON数据绘制国家并着色。

让我用一个简化的国家边界数据，从网络加载。

实际上，让我使用一个预先定义好的简化世界地图图片作为热力图层的基础，或者直接创建一个纯色图层（不根据国家边界），而是用平滑的颜色过渡表示热力。

更好的方案：直接用GeoJSON数据画在Canvas上，每个国家填充对应的颜色。

让我加载GeoJSON：

```js
const geoJson = await fetch('/path/to/countries.geojson').then(res => res.json());
```

但为了在单个HTML中工作，我需要从外部URL加载。

让我从CDN加载：

```js
const geoJson = await fetch('https://raw.githubusercontent.com/nvkelso/natural-earth-vector/master/geojson/ne_110m_admin_0_countries.geojson').then(res => res.json());
```

这个文件大约几百KB。

考虑到实际加载时间和文件大小，让我使用一个更简单的方式。

**替代方案：使用已知的国家数据点作为热力**

我不需要真正的国家边界。我可以在地球上放置一些半透明有颜色的点，代表各地的热力值。

这种方法的优点：
- 不需要GeoJSON
- 简单，效果好
- 可以表示连续的热力分布

让我在城市位置和额外的一些位置放置热力点，用颜色表示不同的值。

但这不能很好地展示"国家/地区热力着色"。

**再想一个方案：**

我可以创建一个Canvas，在上面用简化的矩形或圆形表示每个国家的位置，然后用颜色填充。但这不够准确。

**最终方案：加载GeoJSON并绘制**

好吧，让我用GeoJSON。为了减小体积，我使用110m分辨率的Natural Earth数据。

但为了保持单文件HTML，我需要从外部CDN加载。

让我使用：
`https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson`

这是一个相对较小的文件，包含所有国家。

然后我可以在Canvas上绘制：

```js
function createHeatmapTexture(geoJson, data, indicator) {
    const canvas = document.createElement('canvas');
    canvas.width = 1024;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');
    
    // 绘制每个国家
    geoJson.features.forEach(feature => {
        const countryName = feature.properties.name;
        const value = data[countryName]?.[indicator] || 0;
        const color = getColorForValue(value, indicator);
        
        ctx.fillStyle = color;
        ctx.beginPath();
        // 绘制多边形
        if (feature.geometry.type === 'Polygon') {
            drawPolygon(ctx, feature.geometry.coordinates[0]);
        } else if (feature.geometry.type === 'MultiPolygon') {
            feature.geometry.coordinates.forEach(poly => {
                drawPolygon(ctx, poly[0]);
            });
        }
        ctx.fill();
        ctx.strokeStyle = 'rgba(255,255,255,0.3)';
        ctx.lineWidth = 0.5;
        ctx.stroke();
    });
    
    return new THREE.CanvasTexture(canvas);
}

function drawPolygon(ctx, coordinates) {
    ctx.beginPath();
    coordinates.forEach((point, i) => {
        const x = (point[0] + 180) / 360 * 1024;
        const y = (90 - point[1]) / 180 * 512;
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
    });
    ctx.closePath();
}
```

然后将这个纹理贴到一个半透明球体上，叠加在地球表面。

但这里的问题是：GeoJSON数据的坐标顺序是[lon, lat]，而我在Canvas上使用等距柱状投影，这应该能和地球纹理匹配（如果地球纹理也是等距柱状投影）。

**步骤整合：**

1. 加载地球纹理（等距柱状投影）
2. 加载GeoJSON
3. 创建Canvas热力图纹理
4. 在地球外叠加一个半透明球体，贴热力纹理，使用`Transparent`材质

这样就实现了热力着色。

**第七步：城市点击弹窗**

使用射线检测：

```js
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

renderer.domElement.addEventListener('click', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(cityMarkers);
    
    if (intersects.length > 0) {
        const city = intersects[0].object.userData;
        showCityInfo(city);
    }
});
```

**第八步：悬停航线显示信息**

类似地，用射线检测检查Line对象：

```js
renderer.domElement.addEventListener('mousemove', (event) => {
    // ...
});
```

但Line的射线检测不太准确。为了简单，让我用Line的包围盒或用户数据来判断。

或者，我可以使用一些近似的判断方式：在航线附近创建透明的圆柱体或管道，用来捕捉鼠标悬停。

让我使用TubeGeometry：

```js
const tube = new THREE.Mesh(
    new THREE.TubeGeometry(curve, 64, 0.01, 8, false),
    new THREE.MeshBasicMaterial({ color: 0x00aaff, transparent: true, opacity: 0.8 })
);
```

这样更容易进行射线检测。

**第九步：控制面板**

HTML元素：
- 指标切换按钮：GDP、人口、航班量
- 时间轴：一个滑块 + 播放/暂停按钮

**第十步：时间轴**

模拟数据随时间变化。为每个城市存储多个时间点的数据：

```js
const cities = [
    { name: '北京', lat: 39.9, lon: 116.4, data: { 'GDP': [4800, 5000, 5200, ...], '人口': [...], '航班量': [...] } },
];
```

实际上，让我为每个城市生成随时间变化的模拟数据。

让我使用一个简单的方式：基础数据 + 随时间的小幅波动。

```js
function getCityValue(city, indicator, time) {
    const base = city.baseData[indicator];
    return base * (1 + 0.1 * Math.sin(time / 10 + city.seed));
}
```

这样，当时间轴变化时，所有数据会平滑变化。

好吧，让我开始写完整的代码。这会很多。

让我重新审视一下结构和代码组织。

**代码结构：**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>3D数据可视化地球仪</title>
    <style>
        /* 样式 */
    </style>
</head>
<body>
    <!-- 控制面板HTML -->
    <div id="controls">...</div>
    <!-- 城市信息弹窗 -->
    <div id="cityInfo">...</div>
    <!-- 航线信息弹窗 -->
    <div id="routeInfo">...</div>
    <!-- 加载动画 -->
    <div id="loading">...</div>
    
    <script type="importmap">
    {
        "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
            "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
        }
    }
    </script>
    
    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        
        // 所有应用代码
    </script>
</body>
</html>
```

让我思考一下数据：

**城市数据：**

我选择一些主要城市：

```js
const cityData = [
    { name: '北京', country: '中国', lat: 39.9, lon: 116.4, gdp: 4830, population: 2189, flights: 2800 },
    { name: '上海', country: '中国', lat: 31.2, lon: 121.5, gdp: 4460, population: 2487, flights: 2600 },
    { name: '东京', country: '日本', lat: 35.7, lon: 139.7, gdp: 6320, population: 3740, flights: 3200 },
    { name: '纽约', country: '美国', lat: 40.7, lon: -74.0, gdp: 7130, population: 1880, flights: 3500 },
    { name: '洛杉矶', country: '美国', lat: 34.1, lon: -118.2, gdp: 3270, population: 1300, flights: 2400 },
    { name: '伦敦', country: '英国', lat: 51.5, lon: -0.1, gdp: 3980, population: 900, flights: 3000 },
    { name: '巴黎', country: '法国', lat: 48.9, lon: 2.3, gdp: 3420, population: 1100, flights: 2800 },
    { name: '迪拜', country: '阿联酋', lat: 25.2, lon: 55.3, gdp: 1930, population: 330, flights: 2200 },
    { name: '新加坡', country: '新加坡', lat: 1.4, lon: 103.8, gdp: 2230, population: 590, flights: 2100 },
    { name: '悉尼', country: '澳大利亚', lat: -33.9, lon: 151.2, gdp: 1780, population: 530, flights: 1800 },
    { name: '莫斯科', country: '俄罗斯', lat: 55.8, lon: 37.6, gdp: 1180, population: 1260, flights: 2000 },
    { name: '孟买', country: '印度', lat: 19.1, lon: 72.9, gdp: 1240, population: 2060, flights: 1700 },
    { name: '圣保罗', country: '巴西', lat: -23.6, lon: -46.6, gdp: 1050, population: 1230, flights: 1500 },
    { name: '约翰内斯堡', country: '南非', lat: -26.2, lon: 28.0, gdp: 560, population: 580, flights: 900 },
    // 更多...
];
```

**航线数据：**

```js
const routes = [
    ['北京', '上海'], ['北京', '东京'], ['北京', '纽约'], ['上海', '新加坡'],
    ['东京', '纽约'], ['伦敦', '巴黎'], ['伦敦', '纽约'], ['巴黎', '新加坡'],
    ['迪拜', '伦敦'], ['迪拜', '新加坡'], ['新加坡', '悉尼'], ['纽约', '洛杉矶'],
    // ...
];
```

**国家数据：**

为了热力着色，我需要国家数据。

让我简化：我可以只对有限的国家进行热力着色，对于没有数据的国家使用默认色。

```js
const countryData = {
    '中国': { gdp: 17.7, population: 1412, flights: 5800 },
    '美国': { gdp: 25.5, population: 333, flights: 7200 },
    '日本': { gdp: 4.9, population: 125, flights: 3200 },
    '英国': { gdp: 3.1, population: 67, flights: 2800 },
    // ...
};
```

但GeoJSON中的国家名称可能是英文的，让我用英文名称：

```js
const countryData = {
    'China': { gdp: 17.7, population: 1412, flights: 5800 },
    'United States of America': { gdp: 25.5, population: 333, flights: 7200 },
    'Japan': { gdp: 4.9, population: 125, flights: 3200 },
    // ...
};
```

不同GeoJSON源的国家名称可能不同，所以可能需要映射。

为了简单，让我在代码中处理这个映射。

**简化方案再思考：**

考虑到代码量，让我对某些功能做简化处理，但要确保核心功能正常。

让我采用以下简化：

1. **热力着色**：使用GeoJSON + Canvas绘制，但只对主要国家着色，其他国家用默认灰色。
2. **日夜分界线**：使用自定义ShaderMaterial。
3. **城市灯光**：在夜晚纹理中自带，同时我添加的城市标记点自带发光效果。

让我完整地写代码。

让我考虑时间轴和数据更新：

当时间轴变化时，需要更新：
- 城市标记的大小/颜色（基于当前指标）
- 航线弧线的颜色/粗细
- 热力着色

我会存储每个城市的多个时间点数据，或者用基础数据+波动模拟。

让我用更简单的方式：每个指标有一个基础值，时间变化时乘以一个波动因子。

```js
function getValue(city, indicator, time) {
    const base = city.values[indicator];
    const wave = Math.sin(time * 0.1 + city.phase) * 0.1 + 1;
    return base * wave;
}
```

对于国家数据也类似。

好的，让我开始写代码。这会很长。

让我先确保各部分的实现方式：

**1. 地球（ShaderMaterial）**

```js
const earthGeometry = new THREE.SphereGeometry(2, 64, 64);
const earthMaterial = new THREE.ShaderMaterial({
    uniforms: {
        dayTexture: { value: dayTexture },
        nightTexture: { value: nightTexture },
        sunDirection: { value: new THREE.Vector3(1, 0, 0) },
    },
    vertexShader: `...`,
    fragmentShader: `...`,
});
```

Shader中的暗夜处理：

```glsl
float intensity = dot(normalize(vNormal), normalize(sunDirection));
intensity = smoothstep(-0.1, 0.3, intensity);
vec3 dayColor = texture2D(dayTexture, vUv).rgb;
vec3 nightColor = texture2D(nightTexture, vUv).rgb;
vec3 color = mix(nightColor * 2.0, dayColor, intensity);
```

**2. 大气光晕**

风格化处理：后向渲染的半透明蓝光球体。

**3. 城市标记**

用Mesh + Sprite的组合。

**4. 航线弧线**

用QuadraticBezierCurve3 + TubeGeometry，加流动的点。

**5. 交互**

OrbitControls处理旋转和缩放。
点击检测城市。
悬停检测航线。

**6. 控制面板**

HTML + CSS + JS。

这些都需要整合在一个文件里。

让我现在开始写代码。由于极其复杂，我会尽量简洁但保留所有核心功能。

让我考虑一个重要问题：`OrbitControls`与点击/悬停事件的冲突。

OrbitControls会处理鼠标事件，所以需要区分拖拽和点击：
- 拖拽时发生旋转，不应触发点击
- 点击是短暂的，没有拖拽

解决方案：在mousedown和mouseup之间检查鼠标移动距离，如果小于阈值则视为点击。

```js
let mouseDownPos = null;
renderer.domElement.addEventListener('mousedown', (e) => {
    mouseDownPos = { x: e.clientX, y: e.clientY };
});
renderer.domElement.addEventListener('mouseup', (e) => {
    const dx = e.clientX - mouseDownPos.x;
    const dy = e.clientY - mouseDownPos.y;
    if (dx * dx + dy * dy < 25) {
        handleClick(e);
    }
});
```

好，让我编码。

关于热力层，让我简化实现。

为了减少代码量和加载时间，我可以用以下方案：

在Canvas上绘制国家边界，使用预定义的数据。如果GeoJSON加载失败，则显示默认地球。

我需要一个GeoJSON URL。让我用：
`https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson`

但我不确定这个数据是否包含了国家名称。让我用一个已知可用的：

`https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json`

这个包含`properties.name`，是英文国家名。

我会尝试加载，如果失败则跳过热力层。

让我先写一个大概的代码框架：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<title>3D数据可视化地球仪</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { overflow: hidden; background: #000; font-family: 'Microsoft YaHei', sans-serif; }
#container { width: 100vw; height: 100vh; }

/* 控制面板 */
#panel {
    position: absolute;
    top: 20px;
    right: 20px;
    background: rgba(10, 20, 40, 0.85);
    backdrop-filter: blur(10px);
    border-radius: 12px;
    padding: 16px;
    color: #fff;
    min-width: 220px;
    border: 1px solid rgba(100, 200, 255, 0.2);
    z-index: 100;
}
#panel h3 { margin-bottom: 12px; font-size: 14px; }
.indicator-btns { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 12px; }
.indicator-btn {
    padding: 4px 10px;
    border-radius: 10px;
    border: 1px solid rgba(100, 200, 255, 0.3);
    background: rgba(0, 40, 80, 0.5);
    color: #aac;
    cursor: pointer;
    font-size: 12px;
    transition: all 0.3s;
}
.indicator-btn:hover { background: rgba(0, 80, 120, 0.7); }
.indicator-btn.active { background: rgba(0, 150, 255, 0.6); color: #fff; box-shadow: 0 0 10px rgba(0, 150, 255, 0.4); }

#timePanel { display: flex; align-items: center; gap: 8px; }
#timeRange { flex: 1; }
#timeLabel { font-size: 12px; color: #aac; min-width: 50px; text-align: center; }

/* 信息弹窗 */
.info-popup {
    position: absolute;
    background: rgba(10, 20, 40, 0.9);
    border: 1px solid rgba(100, 200, 255, 0.4);
    border-radius: 8px;
    padding: 12px 16px;
    color: #fff;
    font-size: 13px;
    pointer-events: none;
    z-index: 200;
    display: none;
    box-shadow: 0 4px 30px rgba(0, 100, 200, 0.3);
    backdrop-filter: blur(5px);
}
.popup-title { font-weight: bold; font-size: 16px; margin-bottom: 4px; color: #0af; }
.popup-row { margin: 2px 0; color: #bbc; }

/* 加载动画 */
#loader {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: #0a0a1a;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    color: #aac;
    font-size: 14px;
    transition: opacity 0.8s;
}
.loader-spinner {
    width: 50px; height: 50px;
    border: 3px solid rgba(0, 150, 255, 0.2);
    border-top: 3px solid #0af;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 20px;
}
@keyframes spin { to { transform: rotate(360deg); } }
</style>
</head>
<body>
<div id="container"></div>

<!-- 控制面板 -->
<div id="panel">
    <h3>🌍 数据指标</h3>
    <div class="indicator-btns">
        <button class="indicator-btn active" data-indicator="gdp">GDP</button>
        <button class="indicator-btn" data-indicator="population">人口</button>
        <button class="indicator-btn" data-indicator="flights">航班量</button>
    </div>
    <h3>📅 时间轴</h3>
    <div id="timePanel">
        <button id="playBtn" style="...">▶</button>
        <input type="range" id="timeRange" min="0" max="10" value="5" style="width: 120px;">
        <span id="timeLabel">T=5</span>
    </div>
</div>

<!-- 城市信息弹窗 -->
<div id="cityPopup" class="info-popup"></div>
<!-- 航线信息弹窗 -->
<div id="routePopup" class="info-popup"></div>

<!-- 加载动画 -->
<div id="loader">
    <div class="loader-spinner"></div>
    <div>加载中...</div>
</div>

<script type="importmap">
{
    "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
        "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
    }
}
</script>

<script type="module">
// 所有代码
</script>
</body>
</html>
```

好，现在让我写主要的JavaScript代码。

由于我需要实时根据时间轴更新数据，让我设计好数据结构和更新函数。

**数据结构：**

```js
// 城市数据
const CITIES = [
    { name: '北京', country: '中国', countryEn: 'China', lat: 39.9, lon: 116.4, values: { gdp: [4320, 4450, 4580, 4720, 4850, 4980, 5120, 5250, 5380, 5520, 5650], population: [2060, 2080, 2100, 2120, 2140, 2160, 2180, 2200, 2220, 2240, 2260], flights: [1800, 1950, 2100, 2250, 2400, 2550, 2700, 2850, 3000, 3150, 3300] } },
    // ...
];
```

但这太复杂了。让我简化：为每个城市存储一个基础值（baseValues）和一个变化系数（variation），在时间轴上产生变化。

```js
const CITIES = [
    { name: '北京', country: '中国', lat: 39.9, lon: 116.4, 
      baseValues: { gdp: 5000, population: 2150, flights: 2800 },
      seed: Math.random() * Math.PI * 2 },
    // ...
];
```

在更新时间时：
```js
function getActiveValue(city, indicator, time) {
    const base = city.baseValues[indicator];
    return base * (1 + 0.15 * Math.sin(time * 0.5 + city.seed));
}
```

这样，时间变化时，数值在基础值上下波动。

对于国家数据，类似处理。

**热力着色：**

为了让国家热力着色随时间变化，我需要重新生成纹理。这很昂贵，所以让我用较低频率更新（例如每帧或每0.5秒）。

或者更简单：只用城市标记的大小/颜色来表示当前指标，而热力层仅显示静态的基础值。

但既然要求"切换不同数据指标"，热力层也应该响应指标切换。

让我在切换指标时重新生成热力纹理，而不是每帧更新。

时间轴播放时，热力层用基础颜色（不变），只有城市和航线变化。

这样，热力层只在指标切换时更新。

好吧，让我让热力图在时间轴变化时也更新，但降低频率（每0.3秒更新一次）。

这需要在Canvas上重新绘制。

让我设计一个更高效的方案：将热力图绘制到Canvas上，每次更新时用新的数据重新绘制。

考虑到性能，我可以在`window.requestAnimationFrame`中每秒更新几次。

好吧，让我先完成代码。

让我完整写出核心JS代码：

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// === 配置 ===
const EARTH_RADIUS = 2;
const ISLAND = 1;

// === 场景设置 ===
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(3, 1.5, 5);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = false;
document.getElementById('container').appendChild(renderer.domElement);

// === 控制器 ===
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.minDistance = 2.5;
controls.maxDistance = 10;
controls.autoRotate = true;
controls.autoRotateSpeed = 0.5;
controls.enablePan = false;

// === 光照 ===
const ambientLight = new THREE.AmbientLight(0x333333, 0.5);
scene.add(ambientLight);

const sunLight = new THREE.DirectionalLight(0xffffff, 1.5);
sunLight.position.set(5, 3, 5);
scene.add(sunLight);

// === 纹理加载 ===
function loadTexture(url) {
    return new Promise((resolve, reject) => {
        new THREE.TextureLoader().load(url, resolve, undefined, reject);
    });
}

// === 辅助函数：经纬度转3D坐标 ===
function latLonToVector3(lat, lon, radius) {
    const phi = (90 - lat) * Math.PI / 180;
    const theta = (lon + 180) * Math.PI / 180;
    return new THREE.Vector3(
        -radius * Math.sin(phi) * Math.cos(theta),
        radius * Math.cos(phi),
        radius * Math.sin(phi) * Math.sin(theta)
    );
}

// === 数据 ===
const cityData = [ ... ];
const routeData = [ ... ];
const countryData = { ... };

// === 创建地球 ===
let earth, atmosphere;
// ... 

// === 创建城市标记 ===
const cityGroup = new THREE.Group();
// ...

// === 创建航线 ===
const routeGroup = new THREE.Group();
// ...

// === 创建热力层 ===
let heatLayer;
// ...

// === 主初始化函数 ===
async function init() {
    // 加载纹理
    // 创建地球
    // 创建大气
    // 创建城市
    // 创建航线
    // 添加交互
    // 启动动画
}
```

好，这个代码量太大了。让我直接写完整代码。

让我整理一下我的实现：

对于热力层，让我使用GeoJSON绘制。但由于GeoJSON加载可能需要时间并且可能失败，我需要处理错误。

让我用一个备用方案：如果GeoJSON加载失败，热力层就使用简单的渐变纹理。

让我开始写完整的代码吧。

我发现这个任务太庞大了，让我专注于核心功能，确保一切可运行。

让我用更可控的方式来实现：

1. 使用现有three-globe风格的纹理
2. 使用简单的城市和航线数据
3. 热力层使用GeoJSON绘制，如果不成功则跳过
4. 日夜效果使用shader

让我直接写代码吧。

我会特别注意：
- 纹理URL的正确性
- 经纬度转换的准确性
- OrbitControls与点击事件的协调
- 动画的流畅性

让我开始。

让我考虑另一个简化：热力层加载GeoJSON需要网络请求，可能会慢或失败。让我使用另一种方法：预先在Canvas中绘制一个简单的热力网格。

实际上，让我使用一个更可靠的方式：加载一个世界各国内的采样点集，然后在这些点上绘制大小不一的圆，用颜色表示热力值。

不，这看起来不像是"国家/地区热力着色"。

好吧，让我用GeoJSON。我将从网上下载一个世界地图GeoJSON，然后在Canvas上绘制国家并填充颜色。

**Key decision**: 让我使用这个URL：
`https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json`

这个文件中的`properties.name`是英文国家名。

现在让我写完整的代码。

让我分块写。这将是一个非常大的文件。

让我考虑一下性能问题。热力图层每帧更新会很昂贵。让我只在地理指标切换时更新，时间轴播放时更新城市标记和航线。

嗯，但时间轴播放时，热力数据也应该变化（因为GDP、人口等随时间变化）。让我在时间轴变化时，降低热力层更新频率。

让我使用一个简单的方案：热力层每0.5秒更新一次，或当指标切换时更新。

最终决定：更新热力层时，重新绘制Canvas并更新纹理。这可以在指标切换时完成，时间轴播放时也定期更新（用throttle）。

让我用`setInterval`或`requestAnimationFrame`的节流来实现。

好吧，让我直接写吧。即使代码多，也要写完。

我将在代码中尽量加入注释。

让我考虑一天的最终方案：为了避免GeoJSON加载失败的风险，我用一个**极简方案**：把热力着色简化为一层基于世界地图的Canvas图像叠加，使用画布梯度。

不，不，让我还是用GeoJSON。这是最正确的做法。

让我写一个完整的代码。

让我先列出完整的代码计划：

```
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// 1. 场景/相机/渲染器设置
// 2. 控件设置
// 3. 光照设置
// 4. 数据定义（城市、航线、国家）
// 5. 地球创建（着色器实现日夜）
// 6. 大气光晕
// 7. 城市标记（发光点）
// 8. 航线弧线（流动动画）
// 9. 热力层（GeoJSON Canvas）
// 10. 交互（点击城市、悬停航线）
// 11. 控制面板（指标切换、时间轴）
// 12. 动画循环
```

让我写代码吧。这会很长，但我会尽力保证正确。

让我决定使用简单的方式处理一些功能：
- 城市点击弹窗：显示HTML弹窗
- 悬停航线显示信息：使用mousemove的raycast检测

对于航线，我使用TubeGeometry而不是Line，这样更容易拾取。

让我设计航线数据：

```js
const ROUTES = [
    { from: 0, to: 2 },  // 北京 -> 东京
    { from: 0, to: 3 },  // 北京 -> 纽约
    { from: 1, to: 4 },  // 上海 -> 洛杉矶
    // ...
];
```

每条航线有航班量数据（随时间变化）。

好的，让我直接写最终代码。这将是一个非常大的单文件HTML。

让我把城市数据简化一些，选15个主要城市：

北京、上海、东京、新加坡、迪拜、伦敦、巴黎、纽约、洛杉矶、悉尼、莫斯科、孟买、圣保罗、约翰内斯堡、首尔

航线选择主要国际航线。

国家数据：中国、美国、日本、韩国、新加坡、阿联酋、英国、法国、澳大利亚、俄罗斯、印度、巴西、南非等。

好，让我开始。

让我想想怎么写ShaderMaterial的顶点和片元着色器。

**日夜分界线的Shader：**

```js
const earthMaterial = new THREE.ShaderMaterial({
    uniforms: {
        dayTexture: { value: await loadTexture(DAY_TEXTURE_URL) },
        nightTexture: { value: await loadTexture(NIGHT_TEXTURE_URL) },
        sunDirection: { value: new THREE.Vector3(5, 3, 5).normalize() },
    },
    vertexShader: `
        varying vec2 vUv;
        varying vec3 vNormal;
        varying vec3 vWorldPosition;
        void main() {
            vUv = uv;
            vNormal = normalize(normalMatrix * normal);
            vec4 worldPos = modelMatrix * vec4(position, 1.0);
            vWorldPosition = worldPos.xyz;
            gl_Position = projectionMatrix * viewMatrix * worldPos;
        }
    `,
    fragmentShader: `
        uniform sampler2D dayTexture;
        uniform sampler2D nightTexture;
        uniform vec3 sunDirection;
        varying vec2 vUv;
        varying vec3 vNormal;
        varying vec3 vWorldPosition;
        void main() {
            vec3 dayColor = texture2D(dayTexture, vUv).rgb;
            vec3 nightColor = texture2D(nightTexture, vUv).rgb;
            
            // 计算光照强度
            float intensity = dot(normalize(vNormal), normalize(sunDirection));
            intensity = smoothstep(-0.15, 0.25, intensity);
            
            // 混合日夜颜色
            vec3 color = mix(nightColor * 2.5, dayColor, intensity);
            
            gl_FragColor = vec4(color, 1.0);
        }
    `,
});
```

注意：`normalMatrix`是视图矩阵的逆转置矩阵，因此与视图方向有关。但因为我们在地球上使用模型矩阵旋转，normalMatrix会把法线变换到视图空间。而`sunDirection`是世界空间方向。所以这两个不同空间的值不匹配。

让我修正：在顶点着色器中，将法线变换到世界空间：

```glsl
varying vec3 vNormalWorld;
vWorldNormal = normalize((modelMatrix * vec4(normal, 0.0)).xyz);
```

这样得到世界空间法线。

或者更简单：直接在模型空间中计算。

由于地球在自转（通过旋转模型矩阵），太阳方向需要在地球旋转时保持世界空间不变。

更好的方式：将`sunDirection`传入uniform，在片元中用世界空间法线计算。

```glsl
// vertex
varying vec3 vNormal;
vNormal = normalize((modelMatrix * vec4(normal, 0.0)).xyz);

// fragment
float intensity = dot(normalize(vNormal), normalize(sunDirection));
```

这样是正确的。因为地球旋转后，`modelMatrix`包含了旋转，所以法线会正确变换到世界空间。

很好。

**关于地球自转：**

地球组旋转：
```js
const earthGroup = new THREE.Group(); // 包含地球、城市、航线
earthGroup.rotation.y += 0.001; // 每帧自转
controls.autoRotate = true; // 相机自动旋转（可选）
```

但是`controls.autoRotate`会旋转相机，而地球自转是旋转物体。如果同时启用，会有叠加效果。

让我让地球自转，`controls.autoRotate`关闭。

在地球自转时，城市和航线需要跟着地球一起旋转，所以它们应该是地球的子对象。

热力层也应该是地球的子对象。

大气光晕可以是一个独立的世界空间对象，或者也是地球的子对象（跟随地球旋转？）。

大气光晕通常不跟随地球旋转，因为它是一个均匀的光圈。让我让大气光晕独立于地球旋转。

但为了简单，也可以让大气光晕作为地球的子对象（它看起来是均匀的，旋转后视觉上没有差别）。

让我把大气光晕设为地球的子对象，但这样太阳方向对它不敏感（反正大气光晕是各向同性的）。

实际上，大气光晕只跟相机方向有关，所以旋转不影响。放在地球子对象中也无妨。

**城市灯光效果：**

夜晚纹理自带城市灯光。同时，我添加的城市标记点会有发光效果，在背光面（夜晚区域）更明显。

为了让城市标记在夜晚更亮，我可以根据日夜强度调整标记的颜色：

```js
// 在动画循环中
const sunDir = sunLight.position.normalize();
city.material.color.setHSL(0.5, 1, 0.5 + intensity * 0.3);
```

但这需要对每个城市单独处理材质。

更简单的方式：让城市标记始终发光，使用AdditiveBlending，在夜晚区域看起来更亮（因为背景更暗）。

好，让我就这么做。

**航线弧线的流动动画实现：**

对于每条航线，我创建一个弧线（QuadraticBezierCurve3），然后创建多个流动点（小球），在弧线上移动。

流动点数量：每条航线2-5个，根据航线长度。

```js
// 航线弧线
function createRoute(curve, routeData) {
    const tubeGeometry = new THREE.TubeGeometry(curve, 32, 0.002, 8, false);
    const tubeMaterial = new THREE.MeshBasicMaterial({
        color: 0x00aaff,
        transparent: true,
        opacity: 0.4,
    });
    const tube = new THREE.Mesh(tubeGeometry, tubeMaterial);
    
    // 流动点
    const flowPoints = [];
    const numPoints = 3;
    for (let i = 0; i < numPoints; i++) {
        const point = new THREE.Mesh(
            new THREE.SphereGeometry(0.015, 8, 8),
            new THREE.MeshBasicMaterial({ color: 0x00ffff })
        );
        flowPoints.push(point);
    }
    
    return { tube, flowPoints, curve };
}
```

在动画循环中：
```js
route.flowPoints.forEach((point, i) => {
    const t = (time * route.speed + i / route.flowPoints.length) % 1;
    point.position.copy(route.curve.getPointAt(t));
});
```

**热力层：**

加载GeoJSON后，创建Canvas并绘制：

```js
async function createHeatmapLayer(geoJson, indicator, time) {
    const canvas = document.createElement('canvas');
    canvas.width = 1024;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');
    
    // 透明背景
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // 为每个国家绘制着色区域
    geoJson.features.forEach(feature => {
        const name = feature.properties.name;
        const value = getCountryValue(name, indicator, time);
        const color = getColorForValue(value, indicator);
        
        ctx.fillStyle = color;
        ctx.strokeStyle = 'rgba(100, 200, 255, 0.3)';
        ctx.lineWidth = 0.5;
        
        const geom = feature.geometry;
        if (geom.type === 'Polygon') {
            drawPolygon(ctx, geom.coordinates);
        } else if (geom.type === 'MultiPolygon') {
            geom.coordinates.forEach(poly => drawPolygon(ctx, poly));
        }
    });
    
    const texture = new THREE.CanvasTexture(canvas);
    if (heatLayer) {
        heatLayer.material.map.dispose();
        heatLayer.material.map = texture;
        heatLayer.material.needsUpdate = true;
    } else {
        const material = new THREE.MeshBasicMaterial({
            map: texture,
            transparent: true,
            opacity: 0.5,
            depthWrite: false,
        });
        heatLayer = new THREE.Mesh(
            new THREE.SphereGeometry(EARTH_RADIUS + 0.005, 64, 64),
            material
        );
        earthGroup.add(heatLayer);
    }
}
```

但是`heatLayer`的网格是与地球同半径的球体，贴了Canvas纹理。由于是MeshBasicMaterial且depthWrite:false，会叠加在地球颜色上，产生热力效果。

但由于地球有自转，而热力层也在地球组内，所以热力层会跟着地球旋转。这是正确的，因为热力层反映的是国家/地区的真实位置。

**获取热力值和颜色：**

```js
function getCountryValue(countryName, indicator, time) {
    const data = countryData[countryName];
    if (!data) return 0;
    const base = data[indicator] || 0;
    const wave = 1 + 0.1 * Math.sin(time * 0.3 + data.seed);
    return base * wave;
}

function getColorForValue(value, indicator) {
    // 归一化
    const max = maxValues[indicator];
    const ratio = Math.min(value / max, 1);
    // 蓝 -> 绿 -> 黄 -> 红
    const color = new THREE.Color();
    color.setHSL(0.65 - ratio * 0.65, 1.0, 0.4 + ratio * 0.3);
    return `hsl(${Math.round((0.65 - ratio * 0.65) * 360)}, 100%, ${Math.round((0.4 + ratio * 0.3) * 100)}%)`;
}
```

**交互实现：**

点击城市：
```js
let clickTimer = null;
let isDragging = false;

renderer.domElement.addEventListener('pointerdown', () => {
    isDragging = false;
});
renderer.domElement.addEventListener('pointermove', (e) => {
    if (e.pointerType === 'mouse' && e.buttons > 0 || e.pointerType === 'touch') {
        isDragging = true;
    }
});
renderer.domElement.addEventListener('pointerup', (e) => {
    if (isDragging) return;
    handleClick(e);
});

function handleClick(event) {
    const rect = renderer.domElement.getBoundingClientRect();
    const mouse = new THREE.Vector2(
        ((event.clientX - rect.left) / rect.width) * 2 - 1,
        -((event.clientY - rect.top) / rect.height) * 2 + 1
    );
    raycaster.setFromCamera(mouse, camera);
    
    // 检测城市标记
    const cityMeshes = cityGroup.children.map(child => child.userData.mesh);
    const intersects = raycaster.intersectObjects(cityGroup.children, true);
    
    if (intersects.length > 0) {
        let obj = intersects[0].object;
        while (obj && !obj.userData.city) {
            obj = obj.parent;
        }
        if (obj && obj.userData.city) {
            showCityInfo(obj.userData.city, event.clientX, event.clientY);
        }
    }
}
```

等等，我需要让城市标记的射线检测更简单。让我给每个城市标记一个组：

```js
const cityGroup = new THREE.Group();
CITIES.forEach((city, index) => {
    const marker = createCityMarker(city);
    marker.position.copy(latLonToVector3(city.lat, city.lon, EARTH_RADIUS + 0.02));
    marker.userData.city = city;
    cityGroup.add(marker);
});
```

每个marker是一个Group，包含发光球体和Sprite。射线检测时检测这个Group或者其他子对象。

但raycaster检测的是Mesh，且需要递归检查。让我直接在Mesh上设置userData.city。

好，让我整理一下：marker是一个Group，里面有一个SphereMesh（用于射线检测和视觉）。SphereMesh上设置userData.city。

OK，让我直接写完整代码吧。这太复杂了，我直接在代码中处理。

让我写一个简化版的射线检测：

```js
// 创建城市标记时
const marker = new THREE.Mesh(
    new THREE.SphereGeometry(0.03, 16, 16),
    new THREE.MeshBasicMaterial({ color: 0x00ffaa })
);
marker.userData.city = city;
marker.position.copy(latLonToVector3(city.lat, city.lon, EARTH_RADIUS + 0.02));
cityGroup.add(marker);

// 点击处理
function handleClick(event) {
    const mouse = new THREE.Vector2(
        ((event.clientX - rect.left) / rect.width) * 2 - 1,
        -((event.clientY - rect.top) / rect.height) * 2 + 1
    );
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(cityGroup.children);
    if (intersects.length > 0) {
        const city = intersects[0].object.userData.city;
        showCityInfo(city, event.clientX, event.clientY);
    }
}
```

这样简单多了。

**悬停航线：**

类似方式，但使用mousemove：

```js
let hoveredRoute = null;
renderer.domElement.addEventListener('mousemove', (e) => {
    if (isDragging) return;
    const mouse = new THREE.Vector2(
        ((e.clientX - rect.left) / rect.width) * 2 - 1,
        -((e.clientY - rect.top) / rect.height) * 2 + 1
    );
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(routeMeshes);
    
    if (intersects.length > 0) {
        const routeData = intersects[0].object.userData.route;
        showRouteInfo(routeData, e.clientX, e.clientY);
    } else {
        hideRouteInfo();
    }
});
```

**控制面板逻辑：**

切换指标：
```js
document.querySelectorAll('.indicator-btn').forEach(btn => {
    btn.addEventListener('click', () => {
        document.querySelectorAll('.indicator-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        currentIndicator = btn.dataset.indicator;
        updateVisualizations();
    });
});
```

时间轴：
```js
let isPlaying = false;
let currentTime = 5;

document.getElementById('playBtn').addEventListener('click', () => {
    isPlaying = !isPlaying;
    document.getElementById('playBtn').textContent = isPlaying ? '⏸' : '▶';
});

document.getElementById('timeRange').addEventListener('input', (e) => {
    currentTime = parseFloat(e.target.value);
    document.getElementById('timeLabel').textContent = `T=${currentTime.toFixed(1)}`;
    updateVisualizations();
});

// 在动画循环中
if (isPlaying) {
    currentTime += 0.01;
    if (currentTime > 10) currentTime = 0;
    document.getElementById('timeRange').value = currentTime;
    document.getElementById('timeLabel').textContent = `T=${currentTime.toFixed(1)}`;
}
```

**updateVisualizations函数：**

更新城市标记大小/颜色、航线颜色、热力层。

```js
function updateVisualizations() {
    updateCityMarkers();
    updateRouteColors();
    updateHeatmap();
}
```

注意：频繁更新热力层会很昂贵，让我在指标切换时才更新热力层，时间轴变化时不更新热力层（或节流更新）。

好的，让我做最终的决定：
- 热力层在指标切换时更新
- 时间轴变化时只更新城市标记的大小和颜色，以及航线的颜色和流动速度
- 这样性能和效果都比较好

好吧，让我把热力层也随时间的更新进行节流（每0.5秒一次）：

```js
let lastHeatmapUpdate = 0;
function updateVisualizations(time) {
    updateCityMarkers();
    updateRoutes();
    if (time - lastHeatmapUpdate > 0.5) {
        updateHeatmap();
        lastHeatmapUpdate = time;
    }
}
```

但重建Canvas纹理需要重新绘制整个GeoJSON，这很昂贵（GeoJSON有很多点）。

让我用另一种方式：预先绘制GeoJSON到一个离屏Canvas上，然后在热力层上叠加一个颜色层... 不对，那样就没法按国家着色了。

好吧，让我保持简单：热力层只在指标切换时更新，不随时间变化。

因此，时间轴播放时，热力层显示当前指标的基础值（不变），城市和航线会响应时间变化。

**坐标转换：**

让我确认经纬度转3D坐标的公式是否正确。使用标准球面坐标：

```js
function latLonToVector3(lat, lon, radius) {
    const phi = THREE.MathUtils.degToRad(90 - lat);
    const theta = THREE.MathUtils.degToRad(lon);
    return new THREE.Vector3(
        radius * Math.sin(phi) * Math.cos(theta),
        radius * Math.cos(phi),
        radius * Math.sin(phi) * Math.sin(theta)
    );
}
```

但地球纹理的UV映射可能不同。Three.js的SphereGeometry的UV映射是：UV的x对应经度（0到1映射到360度，从-180到180），y对应纬度（0到1对应北极到南极）。

纹理的等距柱状投影：纹理图片的x宽度对应经度-180（左边缘）到180（右边缘），y高度对应纬度90（上）到-90（下）。

所以，如果我有一个普通的等距柱状投影纹理，Three.js默认的UV映射应该能正确地对上。

关键在于经纬度转3D坐标：经度0度在纹理中心，对应3D坐标的什么位置？

在Three.js中，SphereGeometry的顶点UV是这样的：u从0到1，对应经度从-π到π（或者从0到2π，具体取决于纹理）。默认情况下，球体的起始经度是-π（即-180度），所以λ=0（0度）在球体的x正方向吗？

让我查一下：Three.js SphereGeometry的默认UV映射中，对于球面上一点，u = asin(sqrt(x^2+y^2), z) / π + 0.5 ... 不，让我用常用的公式。

标准的等距柱状纹理映射到球体上，通常：
- 纹理中心对应经度0度
- 纹理左边缘对应经度-180度
- 纹理右边缘对应经度180度

Three.js的SphereGeometry内部使用：
```
u = 0.5 + atan2(z, x) / (2π)
v = 0.5 - asin(y) / π
```

这意味着当x>0, z=0时，u=0.5（对应经度0度）。
当z>0, x=0时，u=0.75（对应经度90度？）

让我更精确地看一下。Three.js的SphereGeometry的生成代码中，顶点位置为：
```
x = -r * cos(phiStart + u * phiLength) * sin(thetaStart + v * thetaLength)
y = r * cos(thetaStart + v * thetaLength)
z = r * sin(phiStart + u * phiLength) * sin(thetaStart + v * thetaLength)
```

其中，phiStart=0，phiLength=2π，thetaStart=0，thetaLength=π。

所以：
```
x = -r * sin(θ) * cos(φ)
y = r * cos(θ)
z = r * sin(θ) * sin(φ)
```

其中φ = u * 2π（φ从0到2π），θ = v * π（θ从0到π）。

这对应于点：
- u=0: φ=0, x=-r*sin(θ), z=0 → 在x负方向
- u=0.25: φ=π/2, x=0, z=r*sin(θ) → 在z正方向
- u=0.5: φ=π, x=r*sin(θ), z=0 → 在x正方向
- u=0.75: φ=3π/2, x=0, z=-r*sin(θ) → 在z负方向

对于标准等距柱状投影纹理（经度从-180到180，左到右），u=0对应-180度，u=0.5对应0度，u=1对应180度。

所以：经度0度（u=0.5）对应3D坐标中x正方向（因为u=0.5时φ=π，x=r*sin(θ)）。

这意味着：当我在3D空间中放置一个城市标记时，需要将经度转换为φ。

设lon为经度，则 u = (lon + 180) / 360。
φ = u * 2π = (lon + 180) / 360 * 2π = (lon + 180) * π / 180。

所以：
```
x = -r * sin(θ) * cos(φ) = -r * sin(θ) * cos((lon+180)*π/180)
y = r * cos(θ)
z = r * sin(θ) * sin(φ)
```

而θ = (90 - lat) * π / 180。

让我们用纬度和经度来计算：

```
φ = (lon + 180) * π / 180
θ = (90 - lat) * π / 180

x = -r * sin(θ) * cos(φ)
y = r * cos(θ)
z = r * sin(θ) * sin(φ)
```

但cos((lon+180)*π/180) = -cos(lon*π/180)
所以 x = -r * sin(θ) * (-cos(lon*π/180)) = r * sin(θ) * cos(lon*π/180)

sin((lon+180)*π/180) = -sin(lon*π/180)
所以 z = r * sin(θ) * (-sin(lon*π/180)) = -r * sin(θ) * sin(lon*π/180)

因此：
```
x = r * sin(θ) * cos(lon_rad)
y = r * cos(θ)
z = -r * sin(θ) * sin(lon_rad)
```

其中 lon_rad = lon * π / 180，θ = (90-lat) * π / 180。

简化后：
```js
function latLonToVector3(lat, lon, radius) {
    const latRad = THREE.MathUtils.degToRad(lat);
    const lonRad = THREE.MathUtils.degToRad(lon);
    return new THREE.Vector3(
        radius * Math.cos(latRad) * Math.cos(lonRad),
        radius * Math.sin(latRad),
        -radius * Math.cos(latRad) * Math.sin(lonRad)
    );
}
```

等等，让我验证一下：
- 北纬90度（北极）：latRad=π/2, x=0, y=r, z=0 → 北极在y轴正方向 ✓
- 经度0度，纬度0度：latRad=0, lonRad=0, x=r, y=0, z=0 → 在x正方向 ✓ (与上面推导一致)
- 经度90度，纬度0度：latRad=0, lonRad=π/2, x=0, y=0, z=-r → 在z负方向

但这与地球仪上通常的显示（经度0度朝向相机，经度90度在右边或左边）可能不同。不过这没关系，只要城市位置和航线弧线相对于地球模型正确即可。

不过等一等，让我用另一种方式验证。如果我在Three.js中直接看SphereGeometry的顶点，对于经度0度、纬度0度的点，它在Three.js的世界坐标中在哪里？

在Three.js中，SphereGeometry生成时，u=0.5对应经度0度（因为u从0到1对应-180到180度，0.5对应0度）。

u=0.5时，φ = u * 2π = π。
根据顶点公式：x = -r * sin(θ) * cos(π) = r * sin(θ), y = r * cos(θ), z = r * sin(θ) * sin(π) = 0。

对于纬度0度：θ = π/2, 所以 x = r, y = 0, z = 0。

所以经度0度、纬度0度的顶点在Three.js中位于 (r, 0, 0)，即x正方向。这与我的公式一致。

所以使用我的 `latLonToVector3` 函数是正确的。

但有一个问题：纹理在球面上的方向。当我用上面的公式在地球上放置城市标记时，城市的位置应该与纹理上的位置一致。

因为纹理是按照Three.js的UV映射贴到球体上的，所以只要是等距柱状投影纹理，城市就应该出现在正确的位置。

让我用坐标转换来检查：
- 北京：lat=39.9, lon=116.4
- 计算：latRad=0.696, lonRad=2.032
- x = r * cos(0.696) * cos(2.032) = r * 0.767 * (-0.441) = -0.338r
- y = r * sin(0.696) = r * 0.642
- z = -r * cos(0.696) * sin(2.032) = -r * 0.767 * 0.898 = -0.689r

在东方亚洲的位置应该是这样的。大概在北京的位置。

好的，这应该没问题。

现在让我写完整的代码。

考虑到代码量，我有点担心篇幅。但我会尽量提供完整可运行的文件。

让我使用更简洁的方式来实现代码，同时确保所有功能正常工作。

让我把城市数量减少到12个，航线减少到15条左右，但保留所有功能。

让我们开始写吧。这会是一个很长的HTML文件。

我注意到一个问题：城市标记是地球的子对象，在地球自转时跟着转。但geoJSON热力层也是地球的子对象。如果地球自转，热力层也跟着转。这是正确的，因为热力层代表地球表面的国家。

但是热力层的纹理是基于Canvas的等距柱状投影，当它贴到球体上并旋转时，会跟着旋转。这是合理的。

地球自转：`earthGroup.rotation.y += 0.001` 每帧。

好，让我直接写完整的代码。尽量在合理范围内缩减但保持功能。

还有一点：为了让城市灯光在夜晚更明显，我需要在地球旋转时，城市标记的发光效果变化。

好的，我会在动画循环中计算每个城市是否处于夜晚区域，并调整其颜色亮度。

但这需要对每个城市单独处理，成本较高。让我简化：使用AdditiveBlending的城市标记始终发光，在夜晚背景上自然显得更亮。

城市灯光的视觉效果主要靠纹理本身（nightTexture）来实现，我添加的城市标记提供额外的发光点。

好，就这样。

现在的完整代码将会非常长。让我专注写。

关于热力层：我使用`MeshBasicMaterial` + Canvas纹理，透明度设为0.5。当切换到热力模式时，地球白天的纹理可能被遮挡，所以热力层的透明度不能太高。

或者，我可以让热力层在需要时可见，不需要时隐藏。默认显示，切换指标时更新。

好吧让我让热力层始终显示，但透明度不高。

为了让用户看到热力效果和地球底层纹理的结合，使用透明度0.4-0.5。

好的，写吧。

让我考虑另一个问题：由于代码在同一个HTML文件中，所有JS代码都在`<script type="module">`中。导入的Three.js从CDN加载。

让我检查importmap中的URL是否正确：
- three: `https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js` ✓
- OrbitControls: `https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/controls/OrbitControls.js` ✓

使用`OrbitControls`的导入路径应该正确。

好，开始写代码。

让我注意：在HTML中，`renderer.domElement`需要添加到body中，但因为我们有`#container` div，所以添加到里面。

`OrbitControls`需要传入`renderer.domElement`。

对于射线检测，需要处理CSS偏移，但我直接用`event.clientX`和`event.clientY`，因为renderer是全屏的。

好的，让我开始写完整代码。

让我在写代码时注意以下几点：
1. 确保代码可以运行（没有语法错误）
2. 使用正确的纹理URL
3. 处理好异步加载（纹理、GeoJSON）
4. 交互事件不要冲突

最终的代码将包含：
- 一个HTML文件
- 所有CSS、JS内联

让我写吧。

让我决定纹理URL：

白天纹理：`https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg`
夜晚纹理：`https://unpkg.com/three-globe/example/img/earth-night.jpg`

这些是可靠的CDN链接。

让我用这些。

好，最终开始。我会写很长的代码，让我慢慢来。

注意：我还需要一个发光点纹理（Sprite用），我可以使用Canvas生成，或者用简单的圆点。

```js
function createGlowTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = 64;
    canvas.height = 64;
    const ctx = canvas.getContext('2d');
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.5)');
    gradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 64, 64);
    return new THREE.CanvasTexture(canvas);
}
```

好的，让我开始。

让我把代码分为多个块来写，以免混乱。

由于不能直接运行代码来测试，我会尽量仔细检查逻辑。

写吧...

嗯这个任务确实复杂。让我试着精简一些功能，但保持核心完整。

让我特别注意性能问题：
- 每帧更新所有航线的流动点位置
- 每帧更新城市标记的缩放/颜色
- 热力层只在指标切换时更新

对了，关于热力层的时间变化：在时间轴播放时，如果指标是GDP，那么热力层也应该响应时间的变化。我可以设置一个固定的更新间隔（比如1秒）来更新热力层。

但这意味着在时间轴播放时，每1秒重新绘制一次热力Canvas，这会消耗CPU。

让我用更好的方案：将热力层数据预计算，然后利用Canvas的缓存... 但这很复杂。

让我最简单：**热力层在时间轴变化时每0.5秒更新一次**，如果数据变化不显著，可以接受。

这样性能问题不会太大，因为GeoJSON的绘制在一个中等规模的数据集上应该不会太慢。

好的，就这么办。

让我再次整理代码结构：

```
```
看起来我无法避免写一个超长HTML文件的事实。让我直接写吧。

我将使用以下简化：
1. 城市只保留8个
2. 航线只保留10条
3. 国家热力数据只对主要国家有值，其他国家为0
4. 热力层Canvas的大小为1024x512，减少绘制负担

好，开始。

我先写一个基础框架，然后逐步填充。

让我直接给完整的代码吧。这会非常长，但我会尽量完整。

注：由于`three-globe`的纹理是等距柱状投影，我需要确保城市的位置映射正确。

让我验证一下：`https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg` 这是一个标准的等距柱状投影地球纹理，左边是-180度经度，右边是180度。

Three.js的SphereGeometry默认将u=0放在经度-180度，u=0.5放在经度0度。但当使用`latLonToVector3`转换时，我使用的公式将经度0度放在x轴正方向。这与纹理映射一致吗？

让我再仔细验证一下。

Three.js的SphereGeometry的纹理映射：
```
u = 0   -> 经度 -180° ->  φ = 0   -> x = -sinθ, z = 0        (x负方向)
u = 0.25 -> 经度 -90° ->  φ = π/2 -> x = 0, z = sinθ         (z正方向)
u = 0.5 -> 经度 0°   ->  φ = π   -> x = sinθ, z = 0          (x正方向)
u = 0.75 -> 经度 90°  ->  φ = 3π/2 -> x = 0, z = -sinθ       (z负方向)
```

而我用`latLonToVector3(lat, lon, r)`得到：
```
lon = -180° -> x = cos(lat) * cos(π) = -cos(lat), y = sin(lat), z = -cos(lat) * sin(π) = 0
              → x负方向, 经度-180°位置 ✓
lon = 0° -> x = cos(lat) * cos(0) = cos(lat), y = sin(lat), z = 0 → x正方向 ✓
lon = 90° -> x = cos(lat) * cos(π/2) = 0, y = sin(lat), z = -cos(lat) → z负方向 ✓
```

所以我的公式与纹理映射一致。

好，现在开始写完整代码。

这次我直接输出所有的代码，不会再去想太多。让我们写吧。

由于这是个大项目，代码会很长。我会尽力确保可用性。这里需要考虑一下：

实际上，城市标记的发光效果可以使用`three`的`Sprite`或`Points`系统。简单起见，我使用`Mesh`+`CanvasTexture`。

让我现在直接输出完整代码。虽然很长，但这是应得的。

让我考虑几个技术细节：

1. 航线TubeGeometry的射线检测：TubeGeometry的Mesh可以被raycaster检测到。但需要设置正确的`userData`。

2. 城市点击弹窗：需要在HTML中定位弹窗位置。

3. 悬停航线显示信息：可以设置cursor样式为pointer。

4. 控制面板的样式需要好看。

5. 需要处理窗口resize。

好，最终开始。

让我用一个简单的方法生成数据：

对于时间轴，让每个城市的数据变化为：base * (1 + 0.1 * sin(time * 0.5 + seed))

对于国家数据，同样方式。

对于航线数据，使用from和to城市之间的航班量，也随时间变化。

好。

让我把控制面板中的时间轴滑块范围设为0-10，初始5。

指标切换：GDP（单位：亿美元）、人口（单位：万人）、航班量（单位：班次/天）。

国家热力值：
- GDP：国家GDP总值（万亿美元）
- 人口：国家人口（百万人）
- 航班量：国家航班总量（每天航班数）

城市数值：
- GDP：城市GDP（亿美元）
- 人口：城市人口（万人）
- 航班量：城市每天航班数

好的，让我定义数据。

让我只选择8个城市：
1. 北京 (中国) lat: 39.9, lon: 116.4
2. 上海 (中国) lat: 31.2, lon: 121.5
3. 东京 (日本) lat: 35.7, lon: 139.7
4. 新加坡 (新加坡) lat: 1.4, lon: 103.8
5. 迪拜 (阿联酋) lat: 25.2, lon: 55.3
6. 伦敦 (英国) lat: 51.5, lon: -0.1
7. 纽约 (美国) lat: 40.7, lon: -74.0
8. 悉尼 (澳大利亚) lat: -33.9, lon: 151.2

10条航线：
1. 北京 -> 东京
2. 北京 -> 纽约
3. 上海 -> 新加坡
4. 上海 -> 悉尼
5. 东京 -> 纽约
6. 新加坡 -> 迪拜
7. 迪拜 -> 伦敦
8. 伦敦 -> 纽约
9. 伦敦 -> 悉尼
10. 纽约 -> 悉尼

国家数据：
- 中国: gdp=17.7, population=1412, flights=5800
- 日本: gdp=4.9, population=125, flights=3200
- 新加坡: gdp=0.4, population=5.9, flights=2100
- 阿联酋: gdp=0.5, population=9.4, flights=2200
- 英国: gdp=3.1, population=67, flights=2800
- 美国: gdp=25.5, population=333, flights=7200
- 澳大利亚: gdp=1.7, population=26, flights=1800
- 其他国家默认低值

好的，现在开始写。

对了，这是非常重要的——由于地球纹理是逐像素的，而热力层也是以纹理方式叠加，如果热力层的Canvas绘制了所有国家（包括值为0的国家），那些值为0的国家会显示为默认颜色（透明或灰色）。

对于没有数据的国家，我应该让它们透明，这样地下的地球纹理能看到。

好的，让我在热力Canvas中，对没有数据的国家不绘制（保持透明），对于有数据的国家，用半透明颜色填充。

顺便说一下，透明的Canvas纹理需要用`transparent: true`材质。

好，开始代码吧。

我发现自己一直在思考细节。现在应该直接写了。我会尽量精简代码但保持完整功能。

让我以一个完整代码的方式输出。

让我就这样吧，给出完整代码。我估计代码会很长，可能有500-800行。别担心，我能行。

让我再想one more thing: 地球自转时，太阳方向保持不变。所以日夜分界线会随着地球自转而在地球表面移动。这看起来是正常的。

但我设置的`sunLight`是DirectionalLight，位置固定。它在场景空间中位置固定，方向指向原点。

对于ShaderMaterial中的`sunDirection` uniform，我需要传入与DirectionalLight一致的方向。

```js
const sunDirection = new THREE.Vector3(5, 3, 5).normalize();
sunLight.position.copy(sunDirection.clone().multiplyScalar(10));
```

这样，光照和shader中的太阳方向一致。

好的，我可以直接写代码了。

让我考虑：如果使用`MeshStandardMaterial`，我可以直接用`map`设置日间纹理，但ShaderMaterial给了更多的控制（日夜混合）。

所以，最终地球使用ShaderMaterial，实现日夜效果。

需要注意的是：ShaderMaterial需要正确处理光照。在这里，我手动在shader中计算光照（太阳方向），所以不需要Three.js的光照系统。

但是，其他物体（如城市标记）使用MeshBasicMaterial，不受光照影响，会一直发光。这是好的，因为城市标记应该是发光的。

航线也是MeshBasicMaterial，同样不受光照影响，一直可见。

热力层也是MeshBasicMaterial（带纹理），始终显示。

这样就不需要实际的光源了。去掉DirectionalLight，只保留AmbientLight... 或者完全不需要。

好，去除所有光源，因为地球用shader，其他物体用BasicMaterial。

好的，最后还有一个问题：OrbitControls的阻尼和自动旋转。

我让OrbitControls的`autoRotate`为false，让地球自转（rotation.y += 0.001每帧）。

但地球自转时，如果用户拖拽旋转相机，会怎样？用户拖拽是旋转相机，地球自转是旋转物体。两者独立，互不影响。这没问题。

好，开始写完整代码。

让我使用一个方式生成发光点纹理：

```js
function createGlowTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = 64;
    canvas.height = 64;
    const ctx = canvas.getContext('2d');
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
    gradient.addColorStop(0, 'rgba(255,255,255,1)');
    gradient.addColorStop(0.3, 'rgba(255,255,255,0.8)');
    gradient.addColorStop(0.7, 'rgba(255,255,255,0.2)');
    gradient.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 64, 64);
    return new THREE.CanvasTexture(canvas);
}
```

好的让我写完整的代码。

让我写一个能直接运行的完整的HTML文件。这将是长代码。

我已经准备好了。开始写。

实际上,让我想一下如何更好地组织代码。为了避免太过混乱,让我以功能模块为组织方式,用注释清晰分隔。

模块1: HTML/CSS
模块2: 场景设置
模块3: 数据定义
模块4: 地球创建
模块5: 城市标记
模块6: 航线弧线
模块7: 热力层
模块8: 交互
模块9: 控制面板
模块10: 动画循环

好,开始。

让我把所有代码一次性写在HTML里。我会尽我所能。