用户要求制作一个能够正常运行的 3D 腕表动画。具体要求：
1. 3D 腕表动画，能够正常运行。
2. 根据当前时间运行，平滑转动的秒针、分针、时针。
3. 提供日期和星期显示功能。
4. 支持同时追踪两个时区。
5. 以完整的 3D 腕表形式呈现。
6. 输出要求：直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。

技术选型：
对于网页端的 3D 动画，最合适的工具是 Three.js。它可以很容易地创建 3D 场景、模型、动画，并且可以直接在浏览器中运行。
或者，可以用 HTML/CSS/JS（纯 CSS 3D 也可以，但 Three.js 更强大且符合“完整 3D”的要求）。由于需要“直接交付可运行产物”，最好提供一个单 HTML 文件，里面包含所有的 HTML, CSS 和 JS (通过 CDN 引入 Three.js)，这样用户只需要用浏览器打开这个 HTML 文件即可运行。

需求拆解：
- 3D 腕表：表盘、表壳、表针（时针、分针、秒针）、日期窗口、星期窗口。
- 两个时区：可以同时追踪两个时区。例如，主时区（本地时间）和副时区（比如 UTC 或用户指定）。可以通过两个表盘（小表盘或双时区指针），或者在表盘上显示两个时区的指针。最简单直观的方法是做一个大表盘显示本地时区，再加一个小表盘（或者在表盘边缘有第二个 GMT 指针）显示另一个时区。由于是“完整的 3D 腕表”，可以做一个主表盘（包含时针、分针、秒针、日期、星期），以及在下方或侧边有一个副表盘显示另一个时区的小时和分钟，或者就用一个额外的 GMT 指针（24小时制）指向主表盘外圈。为了视觉清晰，我决定做两个独立但相连的 3D 表盘：左边一个主表盘（本地时间，带日期和星期），右边一个小表盘（另一个时区，例如纽约时间）。

等等，用户说“以完整的 3D 腕表形式呈现”，且“同时追踪两个时区”。传统双时区手表（如 GMT 手表）通常是同一个表盘上有两根时针（一根本地，一根 GMT），或者有两个小表盘。为了表现 3D，我可以用 Three.js 构建：
- 一个表壳（圆柱或圆角矩形），
- 主表盘（带刻度、数字、日期窗口、星期窗口、时针、分针、秒针），
- 副表盘（显示另一个时区，可以是小表盘，或者主表盘上的第二根时针，为了清晰，加一个小表盘显示第二个时区的时分，或者第二时区的完整指针）。

为了简单且好看，我可以在同一个大表壳内，上方是大表盘（本地时间），下方是小表盘（目标时区，例如 UTC 时间），或者并排两个表盘。并排两个表盘可能更像是两个腕表，但用户说“一个 3D 腕表”，所以最好是一个表壳内有两个表盘（类似某些复杂功能腕表），或者一个主表盘+一个副时区窗口/小表盘。
我们设计一个表壳，里面左半边（或主区域）是主表盘，右半边（或小区域）是副时区表盘。

Three.js 场景设置：
- 场景 (Scene)
- 相机 (PerspectiveCamera)
- 渲染器 (WebGLRenderer)
- 灯光 (AmbientLight, DirectionalLight)
- 几何体：
  - 表壳：TorusGeometry 或 CylinderGeometry
  - 表盘：CylinderGeometry (非常薄的圆柱)
  - 刻度：使用小的 BoxGeometry 排列成圆形
  - 指针：BoxGeometry (厚度不同，长度不同，颜色不同)
  - 日期/星期：由于 Three.js 直接渲染文字比较麻烦（需要加载字体或使用 CanvasTexture），我们可以用 CanvasTexture 把文字（日期、星期、刻度）画在表盘材质的贴图上！这是一个非常聪明且高效的方法。通过 Canvas API 绘制 2D 的表盘（包含数字、星期、日期框），然后把它当作纹理贴到 3D 的圆柱体表面上。指针依然用 3D 的几何体，并且根据时间旋转。
  - 双时区：我们可以用两个 CanvasTexture，分别贴在“主表盘”和“副时区小表盘”上。

具体实现细节：
使用 CDN 引入 Three.js: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`

构建表盘纹理 (CanvasTexture):
1. 主表盘画布 (512x512):
   - 背景色（例如深色或白色）
   - 画上 12 个刻度，以及数字 1-12。
   - 画上星期显示的窗口（在某个位置，比如 9 点钟方向，留一个矩形框，里面动态更新文字？由于 CanvasTexture 是静态的，如果要动态更新日期和星期，我们可以每一帧（或者每一秒）重绘 Canvas，然后设置 `texture.needsUpdate = true`。这样做性能稍微有一点消耗，但 512x512 的重绘对于现代浏览器完全没问题，而且避免了加载字体文件的麻烦。
   - 所以：主表盘的 Canvas 每一秒重绘一次（更新日期和星期文本），或者星期和日期可以用单独的材质/网格（比如一个小的平面 PlaneGeometry）浮动在表盘上方，用单独的 CanvasTexture 来显示日期和星期数字，这样主表盘纹理不需要每一秒重绘。这更好：
     - 主表盘底图（静态）：画刻度和 1-12。
     - 日期窗口（3D 平面或带 CanvasTexture 的平面）：显示 "23"。
     - 星期窗口：显示 "MON" 或中文 "周一"。
2. 副时区表盘画布 (256x256):
   - 画刻度 1-12。
   - 显示该时区的时、分、秒针（或者只显示时和分）。

指针动画：
- 平滑转动：使用 `requestAnimationFrame`。
- 计算当前时间（带毫秒），使得秒针平滑转动（而不是一秒一跳）。
  - 本地时间：`now = new Date()`
  - 本地小时：`h = now.getHours() + m/60 + s/3600`
  - 本地分钟：`m = now.getMinutes() + s/60`
  - 本地秒：`s = now.getSeconds() + ms/1000`
  - 本地指针的角度：`sec_angle = s * (Math.PI / 30)` (因为 60 秒转 2PI，即每秒 PI/30；注意 three.js 中旋转，Y轴向上，默认指针指向 Y 轴正向，我们把它放在 XY 平面，绕 Z 轴旋转？或者放在 XZ 平面，绕 Y 轴旋转。一般表盘面对相机，所以是 XY 平面，绕 Z 轴旋转。不过在 3D 里，通常表盘平放或倾斜。为了方便，我们将表盘放在 XY 平面，相机看向 -Z 轴。这样指针在 XY 平面，绕 Z 轴旋转)。
  - 旋转角度：三点钟方向是 0 度，顺时针。在 Three.js 中，Z 轴指向屏幕外，X 轴向右，Y 轴向上。绕 Z 轴正旋转是逆时针。所以指针角度应该是负的，例如：`rotation.z = -angle`。
  - 秒针：`s / 60 * 2 * Math.PI`
  - 分针：`m / 60 * 2 * Math.PI`
  - 时针：`h / 12 * 2 * Math.PI`

副时区计算：
- 如何获取另一个时区的时间？例如 "America/New_York"。
- JS 的 `Intl.DateTimeFormat` 可以获取特定时区的时间。
- 写一个函数 `getZonedTime(timeZone)` 返回该时区的小时、分钟、秒、毫秒。
```javascript
function getZonedTime(timeZone, date = new Date()) {
    const formatter = new Intl.DateTimeFormat('en-US', {
        timeZone,
        hour: 'numeric',
        minute: 'numeric',
        second: 'numeric',
        hour12: false
    });
    const parts = formatter.formatToParts(date);
    let h, m, s;
    parts.forEach(part => {
        if (part.type === 'hour') h = parseInt(part.value) % 24;
        if (part.type === 'minute') m = parseInt(part.value);
        if (part.type === 'second') s = parseInt(part.value);
    });
    // 毫秒需要自己加，因为 Intl 没有毫秒。可以用本地时间的毫秒，假设时间差不考虑跨毫秒，基本上同步。
    const ms = date.getMilliseconds();
    return { h, m, s, ms };
}
```
等等，`hour12: false` 返回的小时可能是 24（在午夜时），或者在 `en-US` 里可能是 0-23 或者 24？最好取模 24。
副时区我们可以使用 "America/New_York"（纽约）作为示例，并可以在代码里容易地修改。

表盘和表壳的 3D 建模：
1. `watchGroup` = new THREE.Group() 包含所有的部件。
2. 表壳：`new THREE.CylinderGeometry(radius, radius, thickness, 64)`。注意，如果表盘在 XY 平面，圆柱默认是沿 Y 轴的，所以需要旋转 90 度（绕 X 轴）让圆柱平放在 XY 平面，然后它的顶面对着 +Z。
或者，用 `TorusGeometry` 做表圈，`CylinderGeometry` 做底壳。
假设让表盘对着 +Z 轴，那么：
表壳底盘：CylinderGeometry(3, 3, 0.5, 64)，旋转 X 轴 90 度 (`rotation.x = Math.PI / 2`)，这样它的圆柱体轴线沿着 Z 轴。顶面在 Z=0.25，底面在 Z=-0.25。
表盘表面：CircleGeometry(2.8, 64)，放在 Z=0.26 的位置，材质用 CanvasTexture。
或者更好的是用 CylinderGeometry(2.8, 2.8, 0.02, 64)，旋转 X 轴 90 度，放在 Z=0.25。
指针：BoxGeometry(0.1, 1.5, 0.05) （宽，长，厚）。为了让指针围绕表盘中心旋转，把指针的几何中心移开：在创建 BoxGeometry 后，用 `geometry.translate(0, length/2 - offset, 0)` 把中心点移到一端。然后把它放在 Z=0.3（秒针），Z=0.35（分针），Z=0.4（时针），这样它们不互相穿插。
为了好看，指针颜色材质可以用 `MeshStandardMaterial`，例如金属色或红色（秒针）。

双时区布局：
由于用户要求“以完整的 3D 腕表形式呈现”，我们可以把腕表做成两边各有一个表盘的“双表盘腕表”（类似双时区复杂功能表，比如积家 Reverso 或者某些飞行员表），或者同一个大表盘里带一个小表盘。
采用一个表壳包含两个圆形表盘的设计：左表盘（主），右表盘（副）。
或者简单点，一个大表盘（半径 3），右侧附加一个小表盘（半径 1.2）显示副时区。这样就像一个带有小表盘（Subdial）的腕表，非常经典。
主表盘中心在 (0, 0, 0)，副时区小表盘中心在 (2.2, -1.5, 0)（右下角）。
日期窗口：在主表盘的 3 点钟方向，即 (1.5, 0, 0.01) 放一个白色矩形平面，显示数字。
星期窗口：在主表盘的 9 点钟方向，即 (-1.5, 0, 0.01) 放一个白色矩形平面，显示星期。

让我详细设计一下坐标：
表壳：由于有两个表盘，可以做一个圆角矩形表壳，或者两个重叠的圆形表壳。
做两个重叠的圆形表壳最简单且 3D 效果好。
主表壳：Radius 3, Thickness 0.6, 中心 (0, 0, 0)
副表壳：Radius 1.5, Thickness 0.6, 中心 (2.0, -1.8, 0)
等等，如果要在 Z 轴方向看，主表壳和副表壳会有重叠部分。为了 3D 效果好，并且像一个完整的手表，我可以用一个稍微大一点的椭圆或者方形表壳把它们包起来。或者用 `ExtrudeGeometry` 画一个带有圆角的矩形/葫芦形底壳。
其实，我们可以只做一个主表盘（较大的，占据主要视觉），然后在它的 6 点钟方向（下半部分）挖空或者放一个小表盘显示第二个时区。小表盘中心在 (0, -1.8, 0.26)，半径 1.0。
主表盘半径 3，那么 6 点钟方向边缘在 Y=-3。小表盘中心在 Y=-1.7，半径 1.0，可以放在主表盘内部（类似腕表的 9 点钟、3 点钟、6 点钟小表盘）。
这样整体仍然是一个圆形表壳（半径 3.2），非常统一！

主表盘设计：
- 半径 3.0。
- 日期窗口：放在 3 点钟方向 (1.6, 0)，大小 0.8 x 0.6。
- 星期窗口：放在 9 点钟方向 (-1.6, 0)，大小 1.2 x 0.6。
- 小表盘（副时区）：放在 6 点钟方向 (0, -1.6)，半径 1.1。
这样布局很完美，不会互相遮挡。而且正好主指针在中心旋转（时针、分针长 1.5 和 2.0，不会伸到 6 点钟的小表盘里，因为 1.5 < 1.6+1.1 的边缘？等等，分针长 2.0，如果指向 6 点钟，它会延伸到 Y=-2.0，而小表盘中心在 -1.6，半径 1.1，覆盖 Y=-0.5 到 Y=-2.7。所以分针会穿到小表盘上方！这没关系，因为分针在 Z 轴更高（比如 Z=0.35），小表盘在更低（Z=0.26），而且 3D 手表通常小表盘会有指针，或者主指针悬浮在上方。如果分针穿过副时区小表盘，可能会挡住它，但这在真实手表中也常见（比如大日历或偏心小表盘，或者我们可以用短一点的分针？不，时针分针必须长。我们可以把小表盘放在 6 点稍微偏下或者缩小，或者把副时区放在右侧挖个洞？）。
或者，把副时区直接做成一个**额外的表盘**，在视觉上属于同一个手表（就像一个双面表，或者表盘上直接有两个时区指针）。例如，劳力士 GMT 就是在中心有两根时针！一根短而粗（本地），一根细而长（GMT）。我们可以用这种方法，极其优雅：
同一个表盘，中心有：
- 秒针（细长，红色）
- 分针（中等长度，白色）
- 主时针（短粗，白色）
- 副时区时针（稍长，橙色，带一个三角箭头）指向 24 小时外圈刻度。
不过用户要求“同时追踪两个时区”，可能显示两个时区的完整时间（时分秒）更清晰。使用偏心小表盘（Subdial）显示副时区是标准做法，只要小表盘在 6 点钟位置，主指针（长 2.0）确实会指到小表盘上面。为了不遮挡，主表盘刻度 1-12 可以小一点（半径 2.4），主分针长 1.8，时针长 1.2。小表盘中心在 (0, -1.5)，半径 0.9，这样分针最长 1.8，指向 -Y 时到达 -1.8，刚好到达小表盘底部边缘，基本不遮挡小表盘中心。这很可行！

副时区小表盘：
- 中心 (0, -1.5, 0.26)
- 半径 0.9
- 显示副时区的时、分、秒（或者只有时分，秒针太小没必要，但用户要求追踪两个时区，最好也平滑转动秒针或者只显示时分，我们就加个副时区分针和时针，甚至秒针如果做得细一点也行）。
- 副时区小表盘上有自己的刻度（简单的 12 个细线）。

日期和星期显示功能：
为了动态显示日期和星期，我们可以创建两个小的 `PlaneGeometry`，用 CanvasTexture 动态生成文字（或者简单点，用 HTML 元素叠加在 3D 画面上？不行，用户要求“完整的 3D 腕表形式呈现”，在 3D 场景里用 Plane + Canvas 最合适，这样 3D 旋转时它们也会跟着转）。
创建 `DateDisplay` 和 `DayDisplay` 的 Mesh。
在 `render` 循环里，如果日期或星期变了，更新这两个 Canvas 并重新生成纹理。或者为了简单，每帧都更新（反正很轻量，也可以只在秒变化时更新）。
CanvasTexture 画日期：比如 "23"
CanvasTexture 画星期：比如 "FRI" 或者 "周五"（用中文更好，考虑到可能是国内用户，不过用英文 "MON, TUE..." 也行。用中文 "星期一" 可能需要字体支持，Canvas 默认支持系统中文字体，如 "Microsoft YaHei" 或 "sans-serif"，没问题）。

让我们梳理一下 3D 元素：
1. `scene`, `camera`, `renderer`
2. `watchGroup` (整个手表)
3. `caseMesh` (表壳)
   - 用 `CylinderGeometry(3.2, 3.2, 0.5, 64)` 旋转 `Math.PI/2` 绕 X。
   - 材质 `MeshStandardMaterial({color: 0x333333, metalness: 0.8, roughness: 0.2})`
4. `bezelMesh` (表圈，让手表更好看)
   - 用 `TorusGeometry(3.1, 0.2, 16, 64)` 放在 Z=0.25。
5. `mainDialMesh` (主表盘)
   - `CircleGeometry(3.0, 64)` 放在 Z=0.26。
   - 材质：使用 `CanvasTexture`，画上深色背景、白色刻度、1-12 数字。
6. `subDialMesh` (副时区小表盘)
   - `CircleGeometry(0.9, 32)` 放在 `(0, -1.5, 0.27)`。
   - 材质：使用 `CanvasTexture`，画上浅色背景、刻度。
7. 指针组 (`handsGroup`)
   - 主时针：`BoxGeometry(0.15, 1.4, 0.05)`，偏移 `(0, 0.7, 0)` (通过改变 geometry 的 translate)，位置 `Z=0.35`。
   - 主分针：`BoxGeometry(0.1, 2.2, 0.05)`，偏移 `(0, 1.1, 0)`，位置 `Z=0.40`。
   - 主秒针：`BoxGeometry(0.04, 2.5, 0.03)`，偏移 `(0, 1.25, 0)`，位置 `Z=0.45`，颜色红色。
   - 中心轴帽：`CylinderGeometry(0.1, 0.1, 0.5, 16)`，位置 `Z=0.45`，金色。
8. 副时区指针组 (`subHandsGroup`)
   - 放在 `(0, -1.5, 0.30)`。
   - 副时针：`BoxGeometry(0.06, 0.4, 0.02)`，偏移 `(0, 0.2, 0)`，Z=0.30。
   - 副分针：`BoxGeometry(0.04, 0.7, 0.02)`，偏移 `(0, 0.35, 0)`，Z=0.32。
   - 副秒针：可选，或者直接不加，用副分针转就行。为了符合要求“同时追踪两个时区”，只要显示副时区的时分即可，或者加个副秒针也行：`BoxGeometry(0.02, 0.8, 0.01)` 偏移 `(0, 0.4, 0)` Z=0.34 红色。
9. 日期窗口 (`dateMesh`)
   - `PlaneGeometry(0.6, 0.4)` 放在 `(1.8, 0, 0.28)`。
   - `CanvasTexture` 动态更新。背景白，文字黑。
10. 星期窗口 (`dayMesh`)
    - `PlaneGeometry(1.0, 0.4)` 放在 `(-1.8, 0, 0.28)`。
    - `CanvasTexture` 动态更新。

材质与灯光：
- `AmbientLight(0xffffff, 0.6)`
- `DirectionalLight(0xffffff, 0.8)` 放在 `(5, 5, 10)`。
- `DirectionalLight(0xffffff, 0.3)` 放在 `(-5, -5, 5)` 作为补光。

CanvasTexture 绘制细节：
为了清晰，我们需要绘制高分辨率的 Canvas，然后贴图。
主表盘 Canvas 尺寸：1024 x 1024。
- 填充深蓝色背景（如 `#0f172a`）。
- 圆心在 (512, 512)。
- 绘制 60 个秒刻度（细线），12 个小时刻度（粗线）。
- 绘制 1-12 数字（白色，字体 `bold 80px sans-serif`）。
- 日期和星期窗口不需要在 Canvas 里画，因为我们用独立的 `PlaneGeometry` 盖在上面，这样方便动态更新而不用重绘整个表盘 Canvas。

副表盘 Canvas 尺寸：512 x 512。
- 填充黑色背景（`#1e293b`）。
- 圆心在 (256, 256)。
- 绘制 12 个刻度。

动态更新逻辑（在 `requestAnimationFrame` 里）：
```javascript
const now = new Date();
const ms = now.getMilliseconds();

// 本地时间
const h = now.getHours();
const m = now.getMinutes();
const s = now.getSeconds();

// 平滑角度计算
const secAngle = (s + ms / 1000) * (Math.PI / 30); // 每秒走 2PI/60 = PI/30
const minAngle = (m + s / 60) * (Math.PI / 30);
const hourAngle = ((h % 12) + m / 60) * (Math.PI / 6);

// 应用到主指针 (注意 three.js 绕 Z 轴旋转，正角度是逆时针，所以取负)
mainSecHand.rotation.z = -secAngle;
mainMinHand.rotation.z = -minAngle;
mainHourHand.rotation.z = -hourAngle;

// 更新日期和星期（如果变化）
const todayStr = now.getDate().toString();
const dayIndex = now.getDay();
const dayNames = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];
const dayStr = dayNames[dayIndex];
if (currentDateStr !== todayStr) {
    updateDateTexture(todayStr);
    currentDateStr = todayStr;
}
if (currentDayStr !== dayStr) {
    updateDayTexture(dayStr);
    currentDayStr = dayStr;
}

// 副时区时间 (例如纽约)
const tz = "America/New_York";
const zonedTime = getZonedTime(tz, now);
const subSecAngle = (zonedTime.s + ms / 1000) * (Math.PI / 30);
const subMinAngle = (zonedTime.m + zonedTime.s / 60) * (Math.PI / 30);
const subHourAngle = ((zonedTime.h % 12) + zonedTime.m / 60) * (Math.PI / 6);

subSecHand.rotation.z = -subSecAngle;
subMinHand.rotation.z = -subMinAngle;
subHourHand.rotation.z = -subHourAngle;

renderer.render(scene, camera);
```

关于 `getZonedTime` 函数：
考虑到某些旧浏览器可能不支持 `Intl.DateTimeFormat` 的 `timeZone` 选项，但现代浏览器（Chrome, Firefox, Safari, Edge）都完全支持。为了保险，可以提供 fallback：如果报错，则计算 UTC 偏移。不过既然是运行产物，假定在现代浏览器运行即可。我们可以直接让副时区依赖于 `Intl`。
更通用的方法，避免由于 `formatToParts` 的兼容性或解析问题：
```javascript
function getZonedTime(timeZone, date) {
    try {
        const formatter = new Intl.DateTimeFormat('en-US', {
            timeZone,
            hour: 'numeric',
            minute: 'numeric',
            second: 'numeric',
            hour12: false
        });
        const parts = formatter.formatToParts(date);
        let h = 0, m = 0, s = 0;
        parts.forEach(part => {
            if (part.type === 'hour') h = parseInt(part.value) % 24;
            if (part.type === 'minute') m = parseInt(part.value);
            if (part.type === 'second') s = parseInt(part.value);
        });
        return { h, m, s };
    } catch (e) {
        // fallback to UTC
        const str = date.toISOString().substr(11, 8).split(':');
        return { h: parseInt(str[0]), m: parseInt(str[1]), s: parseInt(str[2]) };
    }
}
```
如果我们在国内测试，本地时区是 `Asia/Shanghai` (UTC+8)，副时区设为 `America/New_York` (UTC-4 或 -5)，能清晰看到时差。或者副时区可以设为 `Europe/London`。由于用户可能需要知道哪个是哪个，可以在表盘上用 Canvas 画上 "LOCAL" 和 "NY" 之类的文字。

让表盘更美观：
我们可以在 Canvas 绘制时，加上 "LOCAL" 和 "NEW YORK" 文字。
主表盘 Canvas (1024x1024):
- 在 Y=200 (即 12 点偏下) 画 "LOCAL TIME" 或 "本地时间"。
- 在副表盘 Canvas (512x512) 中心下方画 "NY" 或者 "UTC-4"。

由于要求是“可运行产物”，最好将全部代码写在一个 HTML 文件里。Three.js 从 CDN 加载。

整理 HTML 结构：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>3D 双时区腕表</title>
    <style>
        body { margin: 0; overflow: hidden; background: #111; }
        canvas { display: block; }
        #info {
            position: absolute;
            top: 10px;
            width: 100%;
            text-align: center;
            color: #fff;
            font-family: sans-serif;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div id="info">3D 双时区腕表 (本地时间 & 纽约时间)</div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // 我们的代码
    </script>
</body>
</html>
```

接下来是 Three.js 的详细代码构建。

1. **场景初始化**：
```javascript
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x111111);

const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(0, 0, 12); // 相机放在 Z 轴正方向

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// 灯光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
scene.add(ambientLight);

const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
dirLight.position.set(5, 5, 10);
scene.add(dirLight);

const dirLight2 = new THREE.DirectionalLight(0xffffff, 0.4);
dirLight2.position.set(-5, -5, 5);
scene.add(dirLight2);
```

2. **绘制表盘 Canvas 函数**：
```javascript
function createMainDialTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = 1024;
    canvas.height = 1024;
    const ctx = canvas.getContext('2d');

    // 背景
    ctx.fillStyle = '#0f172a';
    ctx.beginPath();
    ctx.arc(512, 512, 512, 0, Math.PI * 2);
    ctx.fill();

    // 刻度
    for (let i = 0; i < 60; i++) {
        const angle = (i / 60) * Math.PI * 2;
        ctx.save();
        ctx.translate(512, 512);
        ctx.rotate(angle);
        if (i % 5 === 0) {
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-4, -480, 8, 30); // 小时刻度
        } else {
            ctx.fillStyle = '#94a3b8';
            ctx.fillRect(-2, -480, 4, 15); // 分钟刻度
        }
        ctx.restore();
    }

    // 数字
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 70px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    for (let i = 1; i <= 12; i++) {
        const angle = (i / 12) * Math.PI * 2;
        const r = 410;
        const x = 512 + Math.sin(angle) * r;
        const y = 512 - Math.cos(angle) * r;
        ctx.fillText(i.toString(), x, y);
    }

    // 文字标识
    ctx.fillStyle = '#cbd5e1';
    ctx.font = 'bold 40px sans-serif';
    ctx.fillText('LOCAL', 512, 250);
    ctx.fillText('TIME', 512, 300);

    const texture = new THREE.CanvasTexture(canvas);
    return texture;
}

function createSubDialTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = 512;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');

    ctx.fillStyle = '#1e293b';
    ctx.beginPath();
    ctx.arc(256, 256, 256, 0, Math.PI * 2);
    ctx.fill();

    for (let i = 0; i < 60; i++) {
        const angle = (i / 60) * Math.PI * 2;
        ctx.save();
        ctx.translate(256, 256);
        ctx.rotate(angle);
        if (i % 5 === 0) {
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-3, -230, 6, 20);
        } else {
            ctx.fillStyle = '#94a3b8';
            ctx.fillRect(-1, -230, 2, 10);
        }
        ctx.restore();
    }
    
    ctx.fillStyle = '#cbd5e1';
    ctx.font = 'bold 24px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText('NYC', 256, 340);

    return new THREE.CanvasTexture(canvas);
}
```

3. **创建 3D 物体**：
```javascript
const watchGroup = new THREE.Group();
scene.add(watchGroup);

// 表壳
const caseGeo = new THREE.CylinderGeometry(3.2, 3.2, 0.6, 64);
const caseMat = new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.8, roughness: 0.3 });
const caseMesh = new THREE.Mesh(caseGeo, caseMat);
caseMesh.rotation.x = Math.PI / 2; // 让圆柱平躺在 XY 平面
watchGroup.add(caseMesh);

// 表圈 (Bezel)
const bezelGeo = new THREE.TorusGeometry(3.1, 0.2, 16, 64);
const bezelMat = new THREE.MeshStandardMaterial({ color: 0x444444, metalness: 0.9, roughness: 0.2 });
const bezelMesh = new THREE.Mesh(bezelGeo, bezelMat);
bezelMesh.position.z = 0.25;
watchGroup.add(bezelMesh);

// 主表盘
const mainDialGeo = new THREE.CircleGeometry(3.0, 64);
const mainDialMat = new THREE.MeshBasicMaterial({ map: createMainDialTexture() });
const mainDialMesh = new THREE.Mesh(mainDialGeo, mainDialMat);
mainDialMesh.position.z = 0.26;
watchGroup.add(mainDialMesh);

// 副表盘 (小表盘，6点钟方向)
const subDialGeo = new THREE.CircleGeometry(0.9, 32);
const subDialMat = new THREE.MeshBasicMaterial({ map: createSubDialTexture() });
const subDialMesh = new THREE.Mesh(subDialGeo, subDialMat);
subDialMesh.position.set(0, -1.6, 0.27); // 稍微往下，避免主指针过度遮挡
watchGroup.add(subDialMesh);
```
等等，主表盘中心是 0,0。小表盘半径 0.9，放在 Y=-1.6，那么它占据 Y 从 -0.7 到 -2.5。主分针长 2.2，指向 6 点钟方向会到达 Y=-2.2，这会穿过小表盘。没关系，主分针在 Z=0.40，小表盘在 Z=0.27，所以分针悬浮在正上方，视觉上很有层次感。而且真实手表（如浪琴名匠月相）也是大指针扫过小表盘上方，这是完全合理的。

4. **创建指针**：
创建指针时，注意原点：我们需要指针绕一端旋转。Three.js 中 `BoxGeometry` 默认中心在几何体中心。所以我们要偏移几何体顶点：
```javascript
function createHand(width, length, depth, color, zPos, yOffset) {
    const geo = new THREE.BoxGeometry(width, length, depth);
    // 将几何体向上移动 length/2 - yOffset，使旋转轴位于 y = -yOffset 处
    geo.translate(0, length / 2 - yOffset, 0);
    const mat = new THREE.MeshStandardMaterial({ color: color, metalness: 0.5, roughness: 0.5 });
    const mesh = new THREE.Mesh(geo, mat);
    mesh.position.z = zPos;
    return mesh;
}

// 主时针
const mainHourHand = createHand(0.16, 1.4, 0.04, 0xffffff, 0.35, 0.2);
watchGroup.add(mainHourHand);

// 主分针
const mainMinHand = createHand(0.1, 2.2, 0.04, 0xffffff, 0.40, 0.2);
watchGroup.add(mainMinHand);

// 主秒针 (红色)
const mainSecHand = createHand(0.04, 2.5, 0.02, 0xff3333, 0.45, 0.3); // yOffset 0.3 让尾部露出一点
watchGroup.add(mainSecHand);

// 副时针
const subHourHand = createHand(0.06, 0.4, 0.02, 0xffffff, 0.30, 0.05);
subHourHand.position.set(0, -1.6, 0);
watchGroup.add(subHourHand);

// 副分针
const subMinHand = createHand(0.04, 0.7, 0.02, 0xffffff, 0.32, 0.05);
subMinHand.position.set(0, -1.6, 0);
watchGroup.add(subMinHand);

// 副秒针 (红色)
const subSecHand = createHand(0.02, 0.8, 0.01, 0xff3333, 0.34, 0.1);
subSecHand.position.set(0, -1.6, 0);
watchGroup.add(subSecHand);
```

中心轴帽（让指针有个轴）
```javascript
const capGeo = new THREE.CylinderGeometry(0.12, 0.12, 0.5, 16);
const capMat = new THREE.MeshStandardMaterial({ color: 0xffd700, metalness: 0.8, roughness: 0.2 });
const capMesh = new THREE.Mesh(capGeo, capMat);
capMesh.rotation.x = Math.PI / 2;
capMesh.position.z = 0.45;
watchGroup.add(capMesh);

const subCapGeo = new THREE.CylinderGeometry(0.05, 0.05, 0.1, 16);
const subCapMesh = new THREE.Mesh(subCapGeo, capMat);
subCapMesh.rotation.x = Math.PI / 2;
subCapMesh.position.set(0, -1.6, 0.34);
watchGroup.add(subCapMesh);
```

5. **日期和星期显示**：
我们用独立的 Canvas 动态更新。
```javascript
let dateCanvas, dateCtx, dateTexture, dateMesh;
let dayCanvas, dayCtx, dayTexture, dayMesh;

function initDateDayDisplays() {
    // 日期
    dateCanvas = document.createElement('canvas');
    dateCanvas.width = 256;
    dateCanvas.height = 128;
    dateCtx = dateCanvas.getContext('2d');
    dateTexture = new THREE.CanvasTexture(dateCanvas);
    const dateGeo = new THREE.PlaneGeometry(0.6, 0.3);
    const dateMat = new THREE.MeshBasicMaterial({ map: dateTexture });
    dateMesh = new THREE.Mesh(dateGeo, dateMat);
    dateMesh.position.set(2.2, 0, 0.28); // 3点钟方向
    watchGroup.add(dateMesh);

    // 星期
    dayCanvas = document.createElement('canvas');
    dayCanvas.width = 256;
    dayCanvas.height = 128;
    dayCtx = dayCanvas.getContext('2d');
    dayTexture = new THREE.CanvasTexture(dayCanvas);
    const dayGeo = new THREE.PlaneGeometry(1.0, 0.3);
    const dayMat = new THREE.MeshBasicMaterial({ map: dayTexture });
    dayMesh = new THREE.Mesh(dayGeo, dayMat);
    dayMesh.position.set(-2.2, 0, 0.28); // 9点钟方向
    watchGroup.add(dayMesh);
}

function updateDateDay(dateStr, dayStr) {
    // 日期更新
    dateCtx.fillStyle = '#ffffff';
    dateCtx.fillRect(0, 0, 256, 128);
    dateCtx.fillStyle = '#000000';
    dateCtx.font = 'bold 80px sans-serif';
    dateCtx.textAlign = 'center';
    dateCtx.textBaseline = 'middle';
    dateCtx.fillText(dateStr, 128, 64);
    dateTexture.needsUpdate = true;

    // 星期更新
    dayCtx.fillStyle = '#ffffff';
    dayCtx.fillRect(0, 0, 256, 128);
    dayCtx.fillStyle = '#000000';
    dayCtx.font = 'bold 60px sans-serif';
    dayCtx.textAlign = 'center';
    dayCtx.textBaseline = 'middle';
    dayCtx.fillText(dayStr, 128, 64);
    dayTexture.needsUpdate = true;
}
initDateDayDisplays();
```
稍微调整一下星期和日期的位置：主表盘半径 3.0，3 点方向 (2.2, 0) 比较合适，但可能离边缘有点近（3.0的边缘），实际上 2.2, 0 是离中心 2.2，边缘是 3.0，所以完全在表盘内部，很合适。为了不挡住数字 '3' 和 '9'，可以稍微缩小或偏移？真实手表日期窗口通常正好在 3 点钟数字的位置。数字 '3' 在 X=410/512*3.0 ≈ 2.4。如果我们把日期框放在 X=2.4，正好压在数字 3 上。我们可以在画主表盘的时候把数字 3 和 9 省略掉，或者把日期框稍微往外放。
在 `createMainDialTexture` 中，画数字 1-12 时，跳过 3 和 9：
```javascript
    for (let i = 1; i <= 12; i++) {
        if (i === 3 || i === 9) continue; // 留给日期和星期窗口
        // ...
    }
```
这样不仅避免了重叠，还更真实！

6. **动画循环**：
```javascript
let currentDateStr = "", currentDayStr = "";
const dayNames = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];

function getZonedTime(timeZone, date) {
    try {
        const formatter = new Intl.DateTimeFormat('en-US', {
            timeZone,
            hour: 'numeric',
            minute: 'numeric',
            second: 'numeric',
            hour12: false
        });
        const parts = formatter.formatToParts(date);
        let h = 0, m = 0, s = 0;
        parts.forEach(part => {
            if (part.type === 'hour') h = parseInt(part.value) % 24;
            if (part.type === 'minute') m = parseInt(part.value);
            if (part.type === 'second') s = parseInt(part.value);
        });
        return { h, m, s };
    } catch (e) {
        const str = date.toISOString().substr(11, 8).split(':');
        return { h: parseInt(str[0]), m: parseInt(str[1]), s: parseInt(str[2]) };
    }
}

function animate() {
    requestAnimationFrame(animate);

    const now = new Date();
    const ms = now.getMilliseconds();

    const h = now.getHours();
    const m = now.getMinutes();
    const s = now.getSeconds();

    const secAngle = (s + ms / 1000) * (Math.PI / 30);
    const minAngle = (m + s / 60) * (Math.PI / 30);
    const hourAngle = ((h % 12) + m / 60) * (Math.PI / 6);

    mainSecHand.rotation.z = -secAngle;
    mainMinHand.rotation.z = -minAngle;
    mainHourHand.rotation.z = -hourAngle;

    // 日期和星期
    const dateStr = now.getDate().toString().padStart(2, '0');
    const dayStr = dayNames[now.getDay()];
    if (dateStr !== currentDateStr || dayStr !== currentDayStr) {
        updateDateDay(dateStr, dayStr);
        currentDateStr = dateStr;
        currentDayStr = dayStr;
    }

    // 副时区 (纽约)
    const tz = "America/New_York";
    const zt = getZonedTime(tz, now);
    const subSecAngle = (zt.s + ms / 1000) * (Math.PI / 30);
    const subMinAngle = (zt.m + zt.s / 60) * (Math.PI / 30);
    const subHourAngle = ((zt.h % 12) + zt.m / 60) * (Math.PI / 6);

    subSecHand.rotation.z = -subSecAngle;
    subMinHand.rotation.z = -subMinAngle;
    subHourHand.rotation.z = -subHourAngle;

    // 稍微旋转一下手表，增加 3D 立体感 (可选，或者让相机缓慢移动)
    // watchGroup.rotation.y = Math.sin(Date.now() * 0.0005) * 0.3;
    // watchGroup.rotation.x = Math.cos(Date.now() * 0.0003) * 0.2;

    renderer.render(scene, camera);
}
animate();
```

让手表具有 3D 感觉：
如果手表完全正对屏幕，就像 2D。我们可以通过给 `watchGroup` 加一个默认倾斜来实现立体感，或者让相机带一点点角度。
例如：`camera.position.set(0, -2, 10); camera.lookAt(0, 0, 0);` 这样从上往下看，能看到厚度。或者 `watchGroup.rotation.x = -0.2; watchGroup.rotation.y = 0.2;` 这样手表稍微倾斜，能看见侧面的表壳和表圈。
我们在初始化后加上：
```javascript
watchGroup.rotation.x = -0.15;
watchGroup.rotation.y = 0.25;
```
然后在 `animate` 里不覆盖它，或者加一点非常缓慢的自动旋转（`Math.sin` 等），让 3D 效果更好。加上缓慢自转很酷：
```javascript
const time = Date.now() * 0.0005;
watchGroup.rotation.y = Math.sin(time) * 0.3;
watchGroup.rotation.x = -0.15 + Math.cos(time) * 0.1;
```
这样不仅能看出是 3D，而且能欣赏到表壳厚度。

7. **窗口大小调整**：
```javascript
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
```

8. **美化与细节调整**：
表带（Strap）？用户说“完整的 3D 腕表形式呈现”，加上表带会更像手表，而不是一个圆盘。我们可以简单加两段表带，向上和向下延伸。用 `BoxGeometry` 做弯曲的表带可能有点复杂（需要多段或 `ExtrudeGeometry`），可以简单用几个连接在一起的盒子表示表带，或者用两个简单的拱形 `TorusGeometry` 或者粗的 `BoxGeometry` 加在上下。
比如，上方表带：
```javascript
const strapMat = new THREE.MeshStandardMaterial({ color: 0x3d2314, metalness: 0.1, roughness: 0.8 }); // 棕色皮带
const topStrapGeo = new THREE.BoxGeometry(3.0, 3.0, 0.4);
const topStrap = new THREE.Mesh(topStrapGeo, strapMat);
topStrap.position.set(0, 4.0, -0.1); // 连在表壳上方
// 为了有些弧度，可以旋转一点
topStrap.rotation.x = 0.1;
watchGroup.add(topStrap);

const bottomStrapGeo = new THREE.BoxGeometry(3.0, 3.0, 0.4);
const bottomStrap = new THREE.Mesh(bottomStrapGeo, strapMat);
bottomStrap.position.set(0, -4.0, -0.1);
bottomStrap.rotation.x = -0.1;
watchGroup.add(bottomStrap);
```
由于手表整体在 XY 平面，上方是 +Y，下方是 -Y。主表盘半径 3.2，所以表带如果从 Y=2.5 开始，放在 Y=4 是合理的。为了让表带看起来连接到表壳，可以把它放在 Y=3.5，并且 Z 稍微靠后。或者不用太复杂，直接加两个简单的块。如果觉得 `BoxGeometry` 太生硬，就做几节：
```javascript
function createStrapSegment(y, z, rotX) {
    const geo = new THREE.BoxGeometry(2.8, 1.2, 0.3);
    const mesh = new THREE.Mesh(geo, strapMat);
    mesh.position.set(0, y, z);
    mesh.rotation.x = rotX;
    return mesh;
}
// 上表带
watchGroup.add(createStrapSegment(3.8, -0.1, 0.15));
watchGroup.add(createStrapSegment(4.8, -0.4, 0.35));
watchGroup.add(createStrapSegment(5.7, -0.8, 0.55));
// 下表带
watchGroup.add(createStrapSegment(-3.8, -0.1, -0.15));
watchGroup.add(createStrapSegment(-4.8, -0.4, -0.35));
watchGroup.add(createStrapSegment(-5.7, -0.8, -0.55));
```
这样不仅有了表带，而且由于 `rotation.x` 和 `z` 的变化，表带呈现出向屏幕后方弯曲的 3D 弧度，非常逼真！这大大增加了“完整腕表”的观感。
让我们调整一下参数：表壳厚度是 0.6（从 Z=-0.3 到 Z=0.3）。表带 z 从 -0.1 往后退，很好。
为了避免表带和表圈（Z=0.25）冲突，表带在 Z 轴中心应该在 -0.1 左右，厚度 0.3，所以占据 Z=-0.25 到 Z=0.05。刚好接在表壳背面（Z=-0.3 是背面）。完美！

再美化一下材质：
表壳用金属色：
```javascript
const caseMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, metalness: 0.9, roughness: 0.2 });
const bezelMat = new THREE.MeshStandardMaterial({ color: 0xeeeeee, metalness: 1.0, roughness: 0.1 });
```
这样看起来像精钢手表（如劳力士或欧米茄）。

指针颜色：
主指针白色（或者银色金属感）：
`color: 0xffffff` 配上 `MeshStandardMaterial` 的 metalness。
秒针红色：
`color: 0xff2222`

副表盘指针也可以白色和红色。
为了区分，主时区指针在中心，副时区在小表盘。

让我们重新检查一遍颜色和尺寸：
- 主表盘半径 3.0。
- 小表盘中心 `(0, -1.6, 0.27)`，半径 0.9。
- 主分针长度 2.2，如果指向正下方 (-Y)，它的尖端在 Y = -2.2。小表盘的边缘在 Y = -1.6 - 0.9 = -2.5。所以主分针的尖端完全跨过了小表盘，停留在小表盘内 0.3 的位置。由于主分针在 Z=0.40，小表盘在 Z=0.27，所以分针会从小表盘“上方”扫过，这是 3D 层次感！真实腕表中也很常见（如大日历或复杂功能表）。如果想要分针完全不遮挡小表盘，分针只能做到长 1.5，那就太短了。跨过反而是好设计。

关于副时区位置：
由于副时区在 6 点钟，如果主指针在 6 点方向，主分针正好压在副时区表盘上，用户可能觉得乱。没关系，大部分时间分针不指向 6 点，而且 3D 悬浮感解决了视觉冲突。

修改表盘文字，以免重叠：
主表盘 Canvas，跳过 3 和 9：
```javascript
    for (let i = 1; i <= 12; i++) {
        if (i === 3 || i === 9) continue; // 留给日期窗口和星期窗口
        const angle = (i / 12) * Math.PI * 2;
        const r = 410;
        const x = 512 + Math.sin(angle) * r;
        const y = 512 - Math.cos(angle) * r;
        ctx.fillText(i.toString(), x, y);
    }
```
然后在 3 点钟位置放日期窗口，在 9 点钟位置放星期窗口。
但是，3 点钟的 X 坐标是 `512 + sin(PI/2)*410 = 922`，即偏右边缘。如果日期窗口 Mesh 放在 X=2.2（相对坐标），换算成表盘上就是 `2.2 / 3.0 * 512 = 375` 像素，偏右。
等等，主表盘相当于直径 1024（半径 512）。中心是 0,0，半径 3.0。
所以 3 点钟方向在 X=3.0。如果我们在 3D 里把日期窗口放在 `X = 2.2`（距离中心 2.2），对应的 Canvas 像素是 `512 + 2.2/3.0 * 512 = 512 + 375 = 887`。而数字 '3' 原本在 `512 + 410 = 922`。因此日期窗口在 X=2.2 会完美占据 '3' 原本的位置，并且稍微往中心靠一点点，不会掉出表盘（边缘是 3.0，对应 1024）。
星期窗口在 X=-2.2，对应像素 `512 - 375 = 137`，数字 '9' 原本在 `512 - 410 = 102`。完美契合！

日期窗口的大小：`PlaneGeometry(0.7, 0.35)`，放在 `X=2.25, Y=0, Z=0.28`。
星期窗口大小：`PlaneGeometry(1.2, 0.35)`，放在 `X=-2.25, Y=0, Z=0.28`。
我们把日期和星期窗口稍微框一下：在 Canvas 里画个边框？或者在 3D 里加个小黑框？
其实白色背景加黑字，已经很像一个窗口了。可以画个细黑框。在 `MeshBasicMaterial` 里不好直接加边框，但可以在 CanvasTexture 里画边框，或者用稍大一点的黑色平面垫在下面。
给日期垫一个黑色底：
```javascript
const dateBgGeo = new THREE.PlaneGeometry(0.75, 0.4);
const dateBgMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
const dateBg = new THREE.Mesh(dateBgGeo, dateBgMat);
dateBg.position.set(2.25, 0, 0.275);
watchGroup.add(dateBg);

const dayBgGeo = new THREE.PlaneGeometry(1.25, 0.4);
const dayBg = new THREE.Mesh(dayBgGeo, dateBgMat);
dayBg.position.set(-2.25, 0, 0.275);
watchGroup.add(dayBg);
```
然后在上面放白色文字平面，这样就有黑框效果了！很棒。

调整一下 Canvas 更新逻辑：
```javascript
function updateDateDay(dateStr, dayStr) {
    // 日期更新
    dateCtx.fillStyle = '#ffffff';
    dateCtx.fillRect(0, 0, 256, 128);
    dateCtx.fillStyle = '#000000';
    dateCtx.font = 'bold 80px sans-serif';
    dateCtx.textAlign = 'center';
    dateCtx.textBaseline = 'middle';
    dateCtx.fillText(dateStr, 128, 64);
    dateTexture.needsUpdate = true;

    // 星期更新
    dayCtx.fillStyle = '#ffffff';
    dayCtx.fillRect(0, 0, 256, 128);
    dayCtx.fillStyle = '#000000';
    dayCtx.font = 'bold 60px sans-serif';
    dayCtx.textAlign = 'center';
    dayCtx.textBaseline = 'middle';
    dayCtx.fillText(dayStr, 128, 64);
    dayTexture.needsUpdate = true;
}
```

关于副时区的时间显示，用户没指定，我就用纽约时间 `"America/New_York"`。为了让用户知道这是纽约时间，主表盘上的 "LOCAL" 和副表盘上的 "NYC" 字样非常清晰。

测试说明部分：
用户要求：“同时提供运行方式和必要的测试说明”。
所以在输出时，我应该：
1. 提供完整的 HTML 代码（可直接保存为 `.html` 并在浏览器中双击运行）。
2. 给出运行方式说明（保存为 html，用现代浏览器打开，需联网加载 Three.js CDN）。
3. 给出测试说明（观察秒针是否平滑转动，检查日期和星期是否正确，检查副时区时间是否与纽约当前时间一致，观察 3D 倾斜效果等）。

关于“可运行产物”的交付：
通常直接将整个代码块（用 markdown 的 html 代码块包裹）提供即可。这样用户就能复制粘贴。

检查 Three.js 的导入和兼容性：
使用 `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`，这是比较经典且稳定的版本。

还有一个细节：手表是戴在手腕上的，默认 `watchGroup` 是面对着 +Z。相机的初始位置 `camera.position.set(0, 0, 12);`，看向原点，表盘正面正好朝向相机。加上了 `watchGroup.rotation` 的缓慢浮动，效果会很好。

再检查一下指针旋转：
`createHand` 函数中：
```javascript
function createHand(width, length, depth, color, zPos, yOffset) {
    const geo = new THREE.BoxGeometry(width, length, depth);
    geo.translate(0, length / 2 - yOffset, 0); // 平移后，旋转轴在 y = -yOffset 处
    const mat = new THREE.MeshStandardMaterial({ color: color, metalness: 0.5, roughness: 0.5 });
    const mesh = new THREE.Mesh(geo, mat);
    mesh.position.z = zPos;
    return mesh;
}
```
举个例子，主时针：长 1.4，yOffset 0.2。
`length/2 - yOffset` = 0.7 - 0.2 = 0.5。
几何体中心点最初在 0，平移 0.5 后，几何体的 y 范围从 `-0.2` 到 `1.2`。
这意味着当 mesh 放在原点 (0,0) 时，它的旋转轴在 `y = -0.2` 处？等等，如果 mesh 的位置是 (0,0)，几何体的本地坐标是从 -0.2 到 1.2。它绕本地原点 (0,0,0) 旋转！本地原点恰好是指针的尾部偏移 0.2 的位置，也就是指针“轴”的位置！这完全正确，因为旋转是绕 mesh 的本地原点 `(0,0,0)` 进行的。本地原点在 y=-0.2 处是指针的尾部往里 0.2 的位置，所以指针从轴往里伸出 0.2，往外伸出 1.2。完美！这就是真实手表的指针（有个小尾巴平衡）。

对于副表盘指针：
副时针：长 0.4，yOffset 0.05。本地原点在 `-0.05`，伸出长度 0.35，尾部 0.05。
然后把它 `position.set(0, -1.6, 0)`。它的旋转也是绕它自己的本地原点！所以它会准确定位在小表盘中心！因为小表盘 Mesh 的 position 也是 `(0, -1.6, 0.27)`。非常精确！

再来看一下时间计算是否准确：
```javascript
const hourAngle = ((h % 12) + m / 60) * (Math.PI / 6);
```
一圈 12 小时，`2*Math.PI / 12 = Math.PI / 6`。正确。
```javascript
const minAngle = (m + s / 60) * (Math.PI / 30);
```
一圈 60 分钟，`2*Math.PI / 60 = Math.PI / 30`。正确。
```javascript
const secAngle = (s + ms / 1000) * (Math.PI / 30);
```
正确。

而且 `rotation.z = -angle`。因为：
- 当 angle = 0 时，指针在 12 点钟方向（Y正半轴）。
- 当时间过去 3 小时，hourAngle = 3 * PI/6 = PI/2。
- 如果 `rotation.z = -PI/2`，在 Three.js 中，绕 Z 轴负旋转是顺时针（从 +Z 看向原点），所以指针从 12 点方向转到了 3 点钟方向。正确！

副时区计算：
`getZonedTime("America/New_York", now)` 使用了 `Intl.DateTimeFormat`。
纽约时间可能是夏令时或冬令时，Intl 会自动处理，返回当前的时、分、秒。完美。

一些防错处理：
如果 `Intl` 在某些情况下不可用（比如部分旧 Node 环境或极老浏览器），我已经加了 try-catch 回退到 UTC，但这是浏览器端通常不需要担心。现代浏览器完全支持。

视觉优化：
给场景加一点背景元素？不需要，纯色深灰背景足够突出 3D 手表。
```javascript
scene.background = new THREE.Color(0x1a1a1a);
```

让表带更好看：
表带分段可以更圆润一点，但 BoxGeometry 分段已经能体现弧度了：
```javascript
function createStrapSegment(y, z, rotX) {
    const geo = new THREE.BoxGeometry(2.8, 1.2, 0.3);
    const mesh = new THREE.Mesh(geo, strapMat);
    mesh.position.set(0, y, z);
    mesh.rotation.x = rotX;
    return mesh;
}
```
这里有个问题：如果分段之间存在缝隙怎么办？
段与段之间的连接：
段1: Y=3.8, Z=-0.1, rotX=0.15。中心在 Y=3.8。它的厚度为 1.2（在 Y方向，但旋转了，所以实际 Y 跨度大约 1.2*cos(0.15) ≈ 1.18，Z 跨度 1.2*sin(0.15) ≈ 0.18）。
段2: Y=4.8, Z=-0.4, rotX=0.35。
段之间的 Y 差是 1.0，段本身高度 1.2，即使旋转了，Y 方向投影也有 ~1.1，所以段之间会稍微重叠或紧密连接，不会有明显缝隙。这是一个很取巧且有效的做法。

表带的颜色可以用类似于黑色的橡胶带或者深棕色皮带：
```javascript
const strapMat = new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.2, roughness: 0.8 });
```
深黑色橡胶带（比如潜水表）看起来很酷，且和金属表壳形成对比。

再检查一个潜在的遮挡问题：
表带是在 Z=-0.1, -0.4, -0.8 等位置。而表壳背面在 Z=-0.3。所以第一段表带 (Z=-0.1, 厚度0.3 => Z从-0.25到0.05) 会露在表壳正面一点（Z=0.05）。这没关系，因为通常表耳（Lugs）在表壳正面也有延伸。如果想完全在背面，可以让第一段 Z=-0.2（厚度0.3 => Z从-0.35到-0.05），刚好藏在表壳后面（表壳 Z从 -0.3 到 0.3）。
所以修改一下：
```javascript
watchGroup.add(createStrapSegment(3.8, -0.3, 0.15));
watchGroup.add(createStrapSegment(4.8, -0.6, 0.35));
watchGroup.add(createStrapSegment(5.7, -1.0, 0.55));

watchGroup.add(createStrapSegment(-3.8, -0.3, -0.15));
watchGroup.add(createStrapSegment(-4.8, -0.6, -0.35));
watchGroup.add(createStrapSegment(-5.7, -1.0, -0.55));
```
厚度改为 0.4，这样段之间重叠更多：
```javascript
function createStrapSegment(y, z, rotX) {
    const geo = new THREE.BoxGeometry(2.8, 1.2, 0.4);
    // ...
```
这样表带会更连贯。

最后，加入 OrbitControls 可以让用户自己拖动查看 3D 效果，但这就需要额外引入 `OrbitControls.js`。由于要求“直接完成任务并交付可运行产物”，不依赖太多额外文件更稳妥。如果不引入 OrbitControls，我们就在 `animate` 里面做缓慢自动旋转，用户也能看到 3D 效果，而且代码更紧凑（只有一个 `<script>` 引入 Three.js 核心）。
不过，加上 `OrbitControls` 会大大提升体验。CDN 引入也很简单：
`<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>`
然后：
```javascript
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
```
如果用了 OrbitControls，我就可以去掉自动旋转（或者保留一点点），让用户可以自由查看。
考虑到“完整 3D 腕表形式呈现”，交互查看是加分项。我就加上 OrbitControls 吧，这样用户可以拖动旋转、缩放，更好地观察 3D 细节。
必须使用匹配的版本：`three@0.128.0`。
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
```
在代码里：
```javascript
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
// 移除之前的自动旋转代码，或者在 OrbitControls 更新之后加
```
在 `animate` 中：
```javascript
controls.update();
renderer.render(scene, camera);
```
这样用户可以用鼠标全方位查看腕表，包括表带和侧面金属光泽。这绝对满足并超越了“完整 3D”的要求。

总结一下全部功能点是否满足：
- [x] 能够正常运行的 3D 腕表动画（Three.js，HTML单文件）。
- [x] 根据当前时间运行，平滑转动的秒针、分针和时针（requestAnimationFrame + 毫秒计算）。
- [x] 日期和星期显示功能（动态 CanvasTexture 更新）。
- [x] 支持同时追踪两个时区（主表盘为本地，副表盘为纽约，带独立时分秒针）。
- [x] 完整的 3D 腕表形式呈现（表壳、表圈、表盘、指针、表带、3D厚度、OrbitControls自由查看）。
- [x] 输出要求：直接交付可运行产物，提供运行方式和测试说明。

再检查一下星期和日期的显示位置：
日期位于 3 点钟 (X=2.25)，星期位于 9 点钟 (X=-2.25)。
主表盘半径 3.0，边缘在 X=3.0 (对应 1024 像素的 512)。X=2.25 位置在像素 `512 + 2.25/3.0*512 = 512 + 384 = 896`。这是完全合理的，不会掉出表盘。
Canvas 绘制中，我跳过了 `i===3` 和 `i===9`，所以数字 3 和 9 的位置空出来了，正好给日期和星期窗口留白，极其和谐。

代码整合：
将所有 JavaScript 代码放在一个 `<script>` 标签中，结构清晰：
1. 初始化场景、相机、渲染器、控制器。
2. 添加灯光。
3. 创建表盘纹理函数（主、副）。
4. 创建表壳、表圈、表带、主表盘、副表盘。
5. 创建指针函数及实例（主指针、副指针、中心轴）。
6. 初始化日期和星期显示（背景板和动态纹理）。
7. 编写时区获取函数 `getZonedTime`。
8. 编写动画循环 `animate`（包含时间计算、指针旋转、纹理更新、控制器更新、渲染）。
9. 窗口缩放监听。

关于性能：
`dateTexture.needsUpdate = true` 只有在日期或星期改变时才触发（因为我加了 `if (dateStr !== currentDateStr ...)`）。所以大部分时间不会重绘 Canvas，性能极高。
指针的旋转每帧更新，由于只是修改 `rotation.z` 属性，极其轻量。
3D 渲染本身在现代设备上 60fps 毫无压力（不到 50 个 Mesh）。

关于颜色的美化：
表盘面用 `#0f172a`（深蓝），刻度白色，指针白色，秒针红色。这模仿了经典潜水表（如劳力士水鬼）的配色，非常美观。
副表盘用 `#1e293b`（稍浅的深蓝），也是同色系。
表壳用 `#cccccc`（精钢），表带用 `#222222`（黑橡胶）。
在深色背景 `0x111111` 下，会非常有高级感。

确保 `OrbitControls` 在 r128 下可用：
`THREE.OrbitControls` 是全局变量，如果通过 `examples/js/controls/OrbitControls.js` 引入的话。这个版本没问题。

最后审视一下代码有没有遗漏的大括号或语法错误：
- `getZonedTime`：使用 `Intl.DateTimeFormat` 的 `formatToParts`。部分极度老旧浏览器可能不支持 `formatToParts`，但我用了 try-catch，异常时回退到 UTC，保证不崩溃。
- 时区字符串 `"America/New_York"` 在任何支持 Intl 的环境中都是有效的 IANA 时区名。
- `BoxGeometry` 在 r128 中是 `THREE.BoxGeometry`（以前叫 `BoxBufferGeometry`，r128 中两者等价或 `BoxGeometry` 已是 BufferGeometry）。
- `CircleGeometry`、`CylinderGeometry`、`TorusGeometry`、`PlaneGeometry` 都没问题。

一个小修正：
`camera.position.set(0, 0, 12);` 对于包含表带（Y从 -6 到 +6）的手表来说，视野可能有点窄。
表带底部在 Y = -5.7 - 0.6 = -6.3，顶部在 Y = +6.3。整个手表高度大约 12.6。
相机在 Z=12，FOV=45度。在 Z=0 平面，可见高度是 `2 * 12 * tan(22.5度) ≈ 2 * 12 * 0.414 = 9.9`。这意味表带上下可能会被切掉！
所以，把相机拉远一点：`camera.position.set(0, 0, 16);` 或者把 `fov` 调大到 50。
或者将相机稍微往上移一点，并且让控制器目标在原点。
设为 `camera.position.set(0, 0, 18);` 比较保险，用户也可以用滚轮缩放。

还有，`watchGroup` 默认是平的（XY 平面）。我们加了 OrbitControls 后，用户可以自己旋转。为了初始就有 3D 感，可以默认把 `watchGroup.rotation.x = -0.2` 或者不给它旋转，而是直接让相机在一个倾斜角度：
`camera.position.set(0, -5, 15);` 然后 `camera.lookAt(0, 0, 0);` 这样从上往下看，很有立体感。
但 OrbitControls 初始化时会根据 camera 的位置自动设定视角。所以 `camera.position.set(2, -4, 16);` 是一个不错的斜视角，既有正面观感又有 3D 角度。
我们就用 `camera.position.set(3, -2, 15);` 吧。

一切检查就绪，可以输出结果了。需要提供：
1. 完整 HTML 代码（包含内联 CSS 和 JS）。
2. 运行方式（双击/浏览器打开，联网）。
3. 测试说明（各项功能如何验证）。

写回答时注意语气，直接给出产物和说明。使用 Markdown 语法包裹代码。
由于用户要求是“请直接完成任务并交付可运行产物”，我的回答应以交付物为主，代码必须完整无缺。

最后复查：`Intl.DateTimeFormat` 的 `hour12: false` 可能在某些环境下返回 "24" 而不是 "00"，所以我用 `% 24` 处理了，没问题。
`getZonedTime` 里提取 `hour` 为 `parseInt(part.value) % 24`，如果是 24 会变成 0，完美。

一个小 Bug 预防：
`new Intl.DateTimeFormat('en-US', { hour12: false })` 返回的 `hour` part 的值可能是 `"24"`。在 `parseInt` 后变为 `24`，`24 % 24 = 0`。正确。
秒针 `subSecHand` 是红色的，长度 0.8，放在 `subHourHand` 等旁边。
副时区小表盘中心在 (0, -1.6, 0.27)，主表盘中心在 (0,0,0.26)。指针 Z 高度：小表盘 0.30, 0.32, 0.34，比小表盘 0.27 高，没问题。
大表盘指针 Z: 0.35, 0.40, 0.45，比大表盘 0.26 高，没问题。
日期星期窗口 Z: 0.275 (黑底), 0.28 (白字)。大表盘 0.26。高一点，没问题。

好了，准备生成回复。