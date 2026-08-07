用户要求使用 Three.js 制作一个隐形眼镜盒的 3D 模型。
要求：
1. 眼镜盒有醒目的 L 和 R 标识。
2. 配有盒盖。
3. 用户能点击盒盖将其打开。
4. 使用 HTML、CSS 和 JavaScript，全部在一个文件里（单文件 HTML）。
5. 不要查看其他文件，直接创建。
6. 提供运行方式和必要的测试说明。

需要使用的技术栈：
- HTML5
- CSS3 (简单样式，使 canvas 全屏)
- Three.js (通过 CDN 引入，如 https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js)
- JavaScript (逻辑：创建场景、相机、渲染器、灯光、几何体构建眼镜盒、Raycaster 用于点击检测、动画用于打开盖子)

隐形眼镜盒的建模思路：
隐形眼镜盒通常有两个圆形凹槽（或双胞胎形状），分为左右。
简单一点：一个长方体基座，上面有两个圆柱形的凹陷（或者只是视觉上两个圆形区域）。
盒盖：可以是两个独立的半圆形/圆柱形盖子，或者一个整体的盖子分为左右两部分，或者更简单一点，一个整体的盖子（翻盖式）上面写着 L 和 R？但真实的隐形眼镜盒通常是两个小圆盖，分别拧开或扣开。
用户说“点击盒盖将其打开”，单数“盒盖”或者泛指盖子。可以做成两个独立的圆盖（L盖和R盖），点击任何一个都可以打开对应的盖子（向上旋转/移动）。或者做一个对开门/翻盖的盒子。
为了视觉上清晰并具有 L 和 R 标识：
- 盒身（底座）：一个圆角矩形或简单的 BoxGeometry（比如宽度 6，深度 4，高度 1）。
- 在盒身上表面，画/放两个圆形表示左右槽。
- 两个圆柱体（CylinderGeometry）作为盒盖，分别位于左边和右边。
- 在盖子上方（或侧面），用 TextGeometry 或者直接用 Canvas 纹理（更轻量，不需加载字体文件）来显示醒目的 "L" 和 "R"。
- 用 Canvas 生成纹理，或者直接用 HTML 的 overlay 也可以，但用户要求在 3D 模型上有标识。用 CanvasTexture 写在材质上是最简单的，不用引入字体加载器。
- 点击交互：用 Raycaster 检测点击。如果点击了 L 盖子，L 盖子执行打开动画（比如绕一侧铰链旋转 90 度，或者向上移动并淡出，但铰链旋转更像真实开盖）。
- 动画：使用 requestAnimationFrame，在 render 循环里更新盖子的旋转角度，或者使用一个简单的补间逻辑（手动更新角度直到目标角度）。

详细几何设计：
- 底座：BoxGeometry(4, 1, 3) 或者 CylinderGeometry 类似药丸形。用 BoxGeometry 比较简单，或者两个 Cylinder 并排 + 中间连接。为了简单且好看，底座可以用一个稍扁的圆柱体（或者两个并排的扁圆柱体代表两个槽，下面用个底板）。其实真实双联盒就是一个药丸形（胶囊形）的底座。可以用 ShapeGeometry+ExtrudeGeometry 做胶囊形，但为了不引入太多复杂代码，用两个圆柱体作为底座的左右槽，下面加一个扁平的盒子连接。
- 更好：直接做一个扁平的 Box (width=5, height=0.8, depth=3) 作为底座。上面并排放两个浅碟子（用 Cylinder 表示，半径 1.2，高度 0.2，稍微陷进去一点或者用不同颜色表示）。
- 盖子：两个 Cylinder (radius=1.3, height=0.4)，分别位于 x=-1.2 和 x=1.2。
- 盖子的 L 和 R 标识：用 Canvas 画一个带 "L" 或 "R" 的文字，作为盖子的 top 材质（或者整体材质）。因为 Cylinder 的材质数组是 [侧面, 顶, 底]，可以给顶面赋上 CanvasTexture。

盖子打开的动画机制：
- 每个盖子有一个 hinge（铰链）。真实的隐形眼镜盒盖是侧边连着底座的（翻盖）。可以在盖子旁边放一个不可见的枢轴（Group），盖子作为 Group 的子对象偏移一定距离，然后旋转 Group。
- 例如：左盖 Group 放在左边 (x=-1.2, y=0.4)，盖子 mesh 相对 Group 在 (0, 0.2, 0)（假设圆柱高0.4，中心在原点，那么如果 Group 在边缘，盖子要在中心？）。
等一下，如果盖子是 Cylinder，中心在原点，半径 1.3。如果铰链在盖子的一侧（比如 z 方向的后边缘），那么 Group 的位置在 (x=-1.2, z=-1.3, y=0.4)，盖子 mesh 放在 Group 内部的位置是 (0, 0, 1.3)。这样旋转 Group 的 X 轴，盖子就会像翻书一样翻开。
- 打开状态：旋转 Group.rotation.x 从 0 到 -Math.PI / 2 （翻开 90 度）。
- 关闭状态：0。

Raycaster 点击检测：
- 监听 'click' 或 'pointerdown' 事件。
- 获取鼠标在归一化设备坐标 (NDC) 的位置。
- raycaster.setFromCamera(mouse, camera)。
- 检测与两个盖子 mesh 的交点。
- 注意：如果盖子在 Group 里，raycaster 检测的是 mesh，然后我们可以改变其 parent Group 的旋转。

场景设置：
- 场景背景色（浅灰或白色）。
- 相机：PerspectiveCamera，位置稍微倾斜看上方，比如 (0, 5, 7)，lookAt(0, 0, 0)。
- 灯光：AmbientLight + DirectionalLight（投影可选，加点立体感）。
- 渲染器：WebGLRenderer，开启 antialias，设置大小，添加到 body。
- 窗口自适应（resize 事件）。

编写完整 HTML 文件：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>隐形眼镜盒 3D 模型</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #f0f0f0; }
        canvas { display: block; }
        #info {
            position: absolute;
            top: 10px;
            width: 100%;
            text-align: center;
            font-family: sans-serif;
            color: #333;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div id="info">点击眼镜盒的盖子 (L 或 R) 将其打开/关闭</div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // 代码逻辑
    </script>
</body>
</html>
```

详细 JS 逻辑：
1. 初始化场景、相机、渲染器。
```javascript
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xf5f5f5);

const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(0, 6, 8);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);
```

2. 灯光。
```javascript
const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
scene.add(ambientLight);

const dirLight = new THREE.DirectionalLight(0xffffff, 0.6);
dirLight.position.set(5, 10, 5);
dirLight.castShadow = true;
scene.add(dirLight);
```

3. 底座函数。
为了好看，做一个底座：
```javascript
const baseGroup = new THREE.Group();

// 底板
const baseGeo = new THREE.BoxGeometry(5, 0.5, 3);
const baseMat = new THREE.MeshStandardMaterial({ color: 0xeeeeee });
const baseMesh = new THREE.Mesh(baseGeo, baseMat);
baseMesh.position.y = 0.25; // 底部在0
baseMesh.receiveShadow = true;
baseGroup.add(baseMesh);

// 左右凹槽 (简单的圆柱，颜色稍微深一点)
const slotGeo = new THREE.CylinderGeometry(1.1, 1.1, 0.1, 32);
const slotMat = new THREE.MeshStandardMaterial({ color: 0xcccccc });
const leftSlot = new THREE.Mesh(slotGeo, slotMat);
leftSlot.position.set(-1.2, 0.5, 0);
baseGroup.add(leftSlot);

const rightSlot = new THREE.Mesh(slotGeo, slotMat);
rightSlot.position.set(1.2, 0.5, 0);
baseGroup.add(rightSlot);

scene.add(baseGroup);
```

4. 生成带有 L 和 R 文字的 CanvasTexture。
```javascript
function createTextTexture(text, bgColor, textColor) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    // 背景
    ctx.fillStyle = bgColor;
    ctx.fillRect(0, 0, 256, 256);
    
    // 边框（可选）
    ctx.strokeStyle = textColor;
    ctx.lineWidth = 10;
    ctx.strokeRect(5, 5, 246, 246);
    
    // 文字
    ctx.fillStyle = textColor;
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 138);
    
    const texture = new THREE.CanvasTexture(canvas);
    return texture;
}

const leftTex = createTextTexture('L', '#ff9999', '#ffffff');
const rightTex = createTextTexture('R', '#9999ff', '#ffffff');
// 为了醒目，L 用红色系，R 用蓝色系，文字白色。
```

5. 盖子构造（带铰链）。
盖子是一个包含 Cylinder 的 Group。
```javascript
const lidHeight = 0.4;
const lidRadius = 1.2;

function createLid(texture, xPos, isLeft) {
    // 铰链组，位置在盖子后侧 (z = -lidRadius)
    const hinge = new THREE.Group();
    // 盖子中心 x 为 xPos，铰链在 z = -lidRadius 处
    hinge.position.set(xPos, 0.5, -lidRadius);
    
    const lidGeo = new THREE.CylinderGeometry(lidRadius, lidRadius, lidHeight, 32);
    // 材质：侧面和底面普通色，顶面带字
    const sideMat = new THREE.MeshStandardMaterial({ color: 0xdddddd });
    const topMat = new THREE.MeshStandardMaterial({ map: texture });
    // Cylinder 材质数组顺序：侧面, 顶面, 底面
    const lidMat = [sideMat, topMat, sideMat];
    
    const lidMesh = new THREE.Mesh(lidGeo, lidMat);
    // 把盖子放在铰链前方 lidRadius 处，这样旋转铰链时绕后缘翻转
    lidMesh.position.set(0, lidHeight/2, lidRadius);
    lidMesh.castShadow = true;
    lidMesh.receiveShadow = true;
    
    // 记录引用，用于 raycaster
    lidMesh.userData = { hinge: hinge, isOpen: false, targetRot: 0, baseRot: 0 };
    
    hinge.add(lidMesh);
    scene.add(hinge);
    
    return lidMesh;
}

const leftLid = createLid(leftTex, -1.2, true);
const rightLid = createLid(rightTex, 1.2, false);
```

6. 交互 (Raycaster)。
```javascript
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

window.addEventListener('pointerdown', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects([leftLid, rightLid]);
    
    if (intersects.length > 0) {
        const lid = intersects[0].object;
        lid.userData.isOpen = !lid.userData.isOpen;
        // 目标旋转：如果打开，翻开 90 度（-Math.PI/2），如果关闭，0
        lid.userData.targetRot = lid.userData.isOpen ? -Math.PI / 2 : 0;
    }
});
```
等一下，我的 hinge 是绕 X 轴转的。
hinge 初始旋转为 0，盖子在 z 正方向，翻转打开应该是绕 X 轴负方向转（向后翻），所以是 -Math.PI/2。对的。

7. 动画循环。
```javascript
function animate() {
    requestAnimationFrame(animate);
    
    // 平滑开合盖子
    [leftLid, rightLid].forEach(lid => {
        const hinge = lid.userData.hinge;
        // 简单插值
        hinge.rotation.x += (lid.userData.targetRot - hinge.rotation.x) * 0.1;
    });
    
    renderer.render(scene, camera);
}
animate();
```

8. 窗口调整。
```javascript
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
```

需要检查一下盖子尺寸和位置。
底座是 Box(5, 0.5, 3)，宽 5（X轴），深 3（Z轴），高 0.5（Y轴），中心在 y=0.25，顶面在 y=0.5。
左凹槽在 (-1.2, 0.5, 0)，半径 1.1。
左盖子铰链在 (-1.2, 0.5, -1.2)。盖子 mesh 在铰链组内的 (0, 0.2, 1.2)。所以盖子中心在 (-1.2, 0.7, 0)，半径 1.2，高 0.4。底边在 y=0.5（正好盖在底座上），顶边在 y=0.9。
当旋转 -90 度（绕铰链 X 轴），盖子会转到铰链后面（z 方向从 +1.2 变成 y 方向 +1.2 左右？不对，如果绕 x 轴旋转 -90 度，z 正方向会变成 y 负方向？
右手法则：X 轴正向向右（屏幕右），Y 轴向上，Z 轴向屏幕外。
绕 X 轴旋转：正角是 Y 向 Z 旋转，负角是 Y 向 -Z 旋转。
我们盖子在 +Z（前方），如果要向后翻（-Z），应该是绕 X 轴正向旋转？
让我们想一下：点 (0, 0, R) 绕 X 轴旋转角度 theta。
旋转矩阵（绕 X）：
y' = y*cos - z*sin
z' = y*sin + z*cos
如果 theta = +90度（PI/2）：cos=0, sin=1
y' = -R
z' = 0
所以 +Z 会转到 -Y（向下插入地下）。
如果 theta = -90度（-PI/2）：cos=0, sin=-1
y' = R
z' = 0
所以 +Z 会转到 +Y（向上翻开）。
对！theta = -Math.PI/2 时，盖子在 +Y 处，也就是向上立起来了。这正是我们想要的“打开”状态。
等等，铰链在 z = -R，盖子中心在 z = 0（相对于世界），也就是铰链前方 R 处。
旋转 -90 度后，盖子中心变成相对于铰链 (0, R, 0)，也就是世界坐标 (-1.2, 0.5 + R, -R)。也就是向上并且向后移动了，就像翻盖一样。完全正确。

如果希望打开的角度更大一点，比如 -Math.PI * 0.6（约 108 度），也可以，但 90 度就很明显了。
修改 targetRot 为 -Math.PI / 2 没问题。

有个小问题：我们想要醒目 L 和 R 标识，而且盖子顶面有字，但当盖子盖上时（水平），我们从斜上方看，能看到顶面，就能看到字。当盖子打开（竖起），顶面朝向侧面（左侧或右侧？因为绕 X 轴转，顶面原来朝上，转 -90 度后，顶面朝前（+Z）或者朝后？）。
原顶面法线是 +Y。绕 X 转 -90 度后，法线变成 +Z！所以打开时，字朝向 +Z（相机前方偏下），这很好，字依然可见。
不过，很多时候眼镜盒盖打开后，里面是空的或者装镜片的，我们的盖子只做了一个圆柱，底面和顶面都是平的。打开后露出里面底座的凹槽，这很符合逻辑。

检查材质：
CylinderGeometry 的材质数组顺序：在 Three.js 中，通常是 [side, top, bottom]。
在 r128 中，`CylinderGeometry` 创建的组的 material index：
0: 侧面
1: 顶（y+）
2: 底（y-）
所以 `const lidMat = [sideMat, topMat, sideMat];` 是对的。
当盖子合上时，能看到顶面的字，也就是 L 和 R。这正符合要求。

关于盖子位置微调：
左盖 id 在 x = -1.2，右盖在 x = 1.2。
底板的宽度是 5（从 -2.5 到 2.5）。左盖 -1.2，右盖 1.2。中间间隔 2.4 - 2*1.2 = 0（刚好相切？1.2 是中心，盖子半径 1.2，所以左盖最右到 0，右盖最左到 0。完美相切！但盖板半径 1.2，底板宽 5，所以左右各留 0.1 的边框（-2.5 到 -2.4 是边框）。非常紧凑美观！
底板深度 z 从 -1.5 到 1.5。盖子半径 1.2，铰链在 z = -1.2，所以盖子从 z = 0 到 z = -2.4？等等，铰链在 z = -1.2，盖子 mesh 在 z = 1.2（相对于铰链），所以盖子中心在 z = 0，盖子覆盖 z 从 -1.2 到 +1.2。底板 z 从 -1.5 到 1.5，所以前后各留 0.3 边框。这非常完美。

关于颜色：
底座用浅灰色 (0xeeeeee)。
盖子侧面用稍深的灰色 (0xdddddd) 或者带点颜色？为了醒目，L 盖侧面可以用淡红色，R 盖侧面用淡蓝色，这样不开盖也知道左右。
或者就简单的灰色，顶上有大大的 L 和 R（带背景色）。
用户要求“醒目的 L 和 R 标识”。
我们用 CanvasTexture 生成的 `leftTex` 背景是 `#ff9999`（红），文字白；`rightTex` 背景 `#9999ff`（蓝），文字白。这样盖子顶面非常醒目！

为了更逼真，可以在盖子的顶面贴图之外，再给盖子加个边缘或者让字更大。CanvasTexture 的 256x256 画了背景和字，已经足够。

让我们再检查一下 CanvasTexture 在 Three.js 中的方向。
CylinderGeometry 的顶面贴图，UV 映射是怎样的？
默认情况下，Cylinder 顶面的 UV 是从中心向外辐射，或者通常是一个圆。
不过 Three.js 默认顶面 UV 是：以中心为 (0.5, 0.5)，然后一个圆形映射。如果是这样，我们的文字可能会变形或者方向不对（比如倒过来或者旋转）。
可以在创建 Texture 后设置 `texture.center.set(0.5, 0.5);` 或者调整 `rotation`。为了保险，我们可以测试一下。但我现在没法测试，只能凭经验。
在 Three.js 中，Cylinder 顶面的 UV 映射是将一个正方形纹理映射到圆形上，[0,1]x[0,1] 映射到圆。文本画在中心（128, 138）应该大致在中心，不会严重变形，只是可能会旋转。
等一下，Cylinder 顶面的 UV 映射：默认是顶部朝上，如果你从顶部看，纹理是正的。但 Y 轴可能会被翻转或旋转。
如果不使用 CanvasTexture 作为顶面，我们也可以直接在盖子上放一个小的平面（PlaneGeometry）作为标签，这样更容易控制文字的方向和位置！
对，直接用 Mesh 贴在平面上作为标签，比依赖 Cylinder 的 UV 靠谱得多。
做法：
在创建盖子时，创建一个 PlaneGeometry(1.8, 1.8) （略小于盖子直径 2.4），然后贴上带字的 CanvasTexture，材质 `MeshBasicMaterial` 或 `MeshStandardMaterial` (不透明，或者设置 transparent 且中间透明)。
但 CanvasTexture 背景是红/蓝色，那就直接是一个不透明的小圆角矩形或圆形平面，贴在盖子顶面。
或者更好的做法：直接把 Canvas 画成圆形！或者画个圆形背景，这样平面贴上去就是一个圆标签。
其实，我们还是可以直接把 CanvasTexture 作为顶面的 material，但如果担心 UV 映射问题导致文字颠倒，可以用贴片（Sprite 或 Mesh）的方式。
最简单的：在盖子 mesh 上再添加一个子 Mesh（平面），位置在 y = lidHeight/2 + 0.01，朝向 +Y（旋转 -90 度绕 X）。
这样绝对不会有 Cylinder UV 扭曲的问题。
我们来修改这部分：

```javascript
function createLid(texture, xPos, isLeft) {
    const hinge = new THREE.Group();
    hinge.position.set(xPos, 0.5, -lidRadius);
    
    const lidGeo = new THREE.CylinderGeometry(lidRadius, lidRadius, lidHeight, 32);
    const sideMat = new THREE.MeshStandardMaterial({ color: isLeft ? 0xffcccc : 0xccccff });
    const plainMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
    const lidMat = [sideMat, plainMat, plainMat];
    
    const lidMesh = new THREE.Mesh(lidGeo, lidMat);
    lidMesh.position.set(0, lidHeight/2, lidRadius);
    lidMesh.castShadow = true;
    lidMesh.receiveShadow = true;
    
    // 添加标签平面
    const labelGeo = new THREE.PlaneGeometry(lidRadius * 1.8, lidRadius * 1.8);
    const labelMat = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const labelMesh = new THREE.Mesh(labelGeo, labelMat);
    // 平面默认在 XY 面，法线 +Z。要放在盖子顶面（+Y），需绕 X 旋转 -90 度
    labelMesh.rotation.x = -Math.PI / 2;
    // 稍微高于顶面以防 Z-fighting
    labelMesh.position.y = lidHeight / 2 + 0.01;
    lidMesh.add(labelMesh);
    
    lidMesh.userData = { hinge: hinge, isOpen: false, targetRot: 0 };
    
    hinge.add(lidMesh);
    scene.add(hinge);
    
    return lidMesh;
}
```
等等，如果盖子翻转了，顶面朝下，那标签会跟着盖子转（因为 labelMesh 是 lidMesh 的子对象），这没问题。
如果 texture 是带透明背景的（只画字和圆圈），那就更清爽了。
修改 `createTextTexture`：
```javascript
function createTextTexture(text, color) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    // 透明背景 (不用 fillRect 铺满，只画圆)
    ctx.clearRect(0, 0, 256, 256);
    
    // 画一个圆作为背景
    ctx.beginPath();
    ctx.arc(128, 128, 120, 0, Math.PI * 2);
    ctx.fillStyle = color; // 如 '#ff6666'
    ctx.fill();
    
    // 文字
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 138);
    
    return new THREE.CanvasTexture(canvas);
}
```
这样标签就是个圆形标签，有醒目的颜色，中间有白色 L / R，非常清晰！而且因为是用平面贴图，文字方向完全可控（不会颠倒）。

检查一下 Raycaster 检测：
我们点击的是 `lidMesh`，而 `labelMesh` 是它的子对象。如果点击到 `labelMesh`，`intersectObjects([leftLid, rightLid], false)`（递归=false）就检测不到子对象。
所以raycaster检测时：
`raycaster.intersectObjects([leftLid, rightLid], true)` 设为 true 以包含子对象（标签）。但返回的是 `labelMesh` 或者 `lidMesh`。需要往上找有 `userData.hinge` 的对象。
更简单的方法：把 `leftLid` 和 `rightLid` 放到一个数组里，或者检测时把 `labelMesh` 也加进去，或者判断 `intersects[0].object` 是否在 lid 里。
或者给 `labelMesh` 也加上 `userData` 指向 lid：
```javascript
labelMesh.userData = { parentLid: lidMesh };
```
然后在 click 事件里：
```javascript
let clickedObj = intersects[0].object;
if (clickedObj.userData.hinge) {
    // 就是盖子
} else if (clickedObj.userData.parentLid) {
    clickedObj = clickedObj.userData.parentLid;
}
```
更健壮的方法：在 `createLid` 里设置：
`lidMesh.userData.isLid = true;`
`labelMesh.userData.isLid = true;`
`labelMesh.userData.lidRef = lidMesh;`
检测时：
```javascript
let obj = intersects[0].object;
if (obj.userData.isLid) {
    let lid = obj.userData.lidRef ? obj.userData.lidRef : obj;
    lid.userData.isOpen = !lid.userData.isOpen;
    lid.userData.targetRot = lid.userData.isOpen ? -Math.PI / 2 : 0;
}
```
不过其实 `intersectObjects` 如果传 `[leftLid, rightLid]` 并且 `recursive=true`（第二个参数默认是 true 吗？Three.js 中 `intersectObjects(objects, recursive)` 默认 recursive 是 true，但最好显式传 true），那么如果相交到子对象，它会返回子对象，但我们可以在查找时向上遍历：
```javascript
let obj = intersects[0].object;
while (obj && !obj.userData.hinge) {
    obj = obj.parent;
}
if (obj && obj.userData.hinge) {
    // obj 就是 lidMesh
    obj.userData.isOpen = !obj.userData.isOpen;
    obj.userData.targetRot = obj.userData.isOpen ? -Math.PI / 2 : 0;
}
```
这非常干净！

让我们重新整理 `createLid` 里的 `userData`：
```javascript
lidMesh.userData = { hinge: hinge, isOpen: false, targetRot: 0 };
labelMesh.userData = { isLabel: true };
```
然后在 click 里：
```javascript
const intersects = raycaster.intersectObjects([leftLid, rightLid], true);
if (intersects.length > 0) {
    let obj = intersects[0].object;
    while (obj && !(obj.userData && obj.userData.hinge)) {
        obj = obj.parent;
    }
    if (obj && obj.userData.hinge) {
        obj.userData.isOpen = !obj.userData.isOpen;
        obj.userData.targetRot = obj.userData.isOpen ? -Math.PI / 2 : 0;
    }
}
```
完美。

再考虑盖子开合动画的细节：
```javascript
function animate() {
    requestAnimationFrame(animate);
    
    [leftLid, rightLid].forEach(lid => {
        const hinge = lid.userData.hinge;
        // 插值
        hinge.rotation.x += (lid.userData.targetRot - hinge.rotation.x) * 0.15;
    });
    
    renderer.render(scene, camera);
}
```
这里有个小问题：当 `hinge.rotation.x` 趋近于 `targetRot` 时，由于浮点精度，可能会一直计算，但这消耗极小，不用管。或者加个极小值判断，但没必要。

光照和阴影：
为了阴影生效，需要：
```javascript
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 可选

dirLight.castShadow = true;
// 设置光源阴影范围
dirLight.shadow.camera.left = -10;
dirLight.shadow.camera.right = 10;
dirLight.shadow.camera.top = 10;
dirLight.shadow.camera.bottom = -10;

baseMesh.receiveShadow = true;
leftSlot.receiveShadow = true;
rightSlot.receiveShadow = true;
// 盖子
lidMesh.castShadow = true;
lidMesh.receiveShadow = true;
```
这些前面都设置了。

加入 OrbitControls？
题目没要求，但用户通常会想旋转视角看看模型。如果不加，视角固定。不过 Three.js 的 OrbitControls 需要从外部引入（比如 `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`），既然要求单文件，我们可以引入这个 CDN，或者自己不写控制（用户只要求点击盒盖打开）。为了更稳定（避免 CDN 加载失败导致整个脚本报错），我不加 OrbitControls，只固定一个好视角即可。但如果能通过鼠标拖拽旋转会更好。不用了，保持简单，满足需求即可。如果加 OrbitControls，需要确保它能正常加载，或者用 ES module 形式。考虑到用 `<script src=".../three.min.js">` 是全局 THREE，可以配合 `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js` 来加。
加上 OrbitControls 会让作品看起来更专业：
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
```
然后在 JS 中：
```javascript
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.target.set(0, 0.5, 0);
```
在 animate 里 `controls.update();`。这非常好，能旋转查看。

隐形眼镜盒的底座细节：
目前的底座是：
底板：`BoxGeometry(5, 0.5, 3)`（宽，高，深），中心在 `(0, 0.25, 0)`。
这其实是个方盒子。隐形眼镜盒通常是圆角的或者胶囊状的。为了简单和兼容，Box 也行，或者把底板换成两个大圆柱加中间连接，不过 Box 也挺直观。
如果想更像，可以把底板做成稍微带有颜色的塑料感。
```javascript
const baseMat = new THREE.MeshStandardMaterial({ color: 0xfafafa, roughness: 0.4, metalness: 0.1 });
```
隐形眼镜盒通常会有个夹子或者只是两个独立的槽。
我们的 `leftSlot` 和 `rightSlot` 是 `CylinderGeometry(1.1, 1.1, 0.1, 32)`，高度只有 0.1，放在 y=0.5（和底板顶部齐平），这样就像凹进去一点？不，如果中心在 y=0.5，高度 0.1，它的范围是 0.45 到 0.55，也就是突出底板 0.05。如果要凹陷，应该放在 y=0.45，这样范围是 0.4 到 0.5，正好平齐或者稍微凹下。
改一下：
`leftSlot.position.set(-1.2, 0.45, 0);` （顶部在 0.5，和底板齐平，或者 0.44，稍微凹下）。
然后里面可以放一个更小的圆柱代表隐形眼镜（或者空着就行，用户只是说隐形眼镜盒，没说必须放镜片）。空着就可以了，打开盖子就能看到凹槽。

修改一下凹槽，用不同颜色表示里面是空的：
```javascript
const slotGeo = new THREE.CylinderGeometry(1.0, 1.0, 0.2, 32);
const slotMat = new THREE.MeshStandardMaterial({ color: 0xab47bc }); // 或者浅蓝色，这里用淡蓝 0x81d4fa
slotMat.color.set(0x81d4fa);
```
让凹槽看起来像装护理液的蓝色池子。

还有，盖子在合上时，由于标签平面放在 `y = lidHeight / 2 + 0.01`，这个 `lidHeight` 是 0.4，所以标签在 `y = 0.21` 相对于盖子中心。这是对的。但当一个 Group（hinge）加了 lidMesh，lidMesh 在 `y = lidHeight/2 = 0.2`，所以标签在 `y = 0.21` 相对于 lidMesh。也就是盖子顶面上方 0.01。没问题。

等等，再仔细算一下高度：
底座底板：高度 0.5，y 中心 0.25，顶部在 y = 0.5。
左右凹槽：高度 0.2，如果中心在 y = 0.4，顶部在 y = 0.5，底部在 y = 0.3（嵌在底板里），完美。
左盖铰链（hinge）：`y = 0.5`（和底座顶部同高）。
盖子 mesh：在 hinge 内的 `y = lidHeight / 2 = 0.2`。所以盖子中心在世界坐标 `y = 0.5 + 0.2 = 0.7`。
盖子高度 0.4，所以底部在世界 `y = 0.5`，顶部在世界 `y = 0.9`。这正好盖在底座上！
所以盖子合上时，盖子底部 `y = 0.5` 正好和底座顶部 `y = 0.5` 接触。完美。
那标签就在 `y = 0.7 + 0.21 = 0.91`（略微高于盖子顶面 `0.9`）。没问题。

关于 z 轴位置：
底板深度 3（z 从 -1.5 到 1.5）。
左盖中心在 x = -1.2，铰链在 z = -1.2，盖子 mesh 在 hinge 内的 z = 1.2。
合上时，盖子 mesh 的世界 z = -1.2 + 1.2 = 0。所以左盖覆盖 z 从 -1.2 到 +1.2。完全在底板 z 范围（-1.5 到 1.5）内！完美。
而且左盖 x = -1.2，右盖 x = 1.2。左盖覆盖 x 从 -2.4 到 0，右盖覆盖 x 从 0 到 2.4。底板 x 从 -2.5 到 2.5。左右各留 0.1 的边缘，中间没有缝隙也不断开（相切于 x=0）。完美对称和紧凑！

盖子侧边材质：
为了更美观，`sideMat` 给个稍微带点颜色的：
左盖：浅红 `0xffcdd2` 或者直接用白色 `0xffffff`。其实顶面已经有醒目的标签了，盖子侧面可以用纯白或半透明塑料感（MeshStandardMaterial，roughness 高一点，metalness 0）。
```javascript
const sideMat = new THREE.MeshStandardMaterial({ 
    color: isLeft ? 0xffebee : 0xe3f2fd, 
    roughness: 0.3, 
    metalness: 0.1 
});
```
左盖侧面极淡的红，右盖极淡的蓝，呼应 L 和 R。很好。

再检查 Raycaster 和动画部分：
由于我们加了 OrbitControls，鼠标点击时可能会和拖拽冲突（拖拽松开也会触发 click）。为了防止拖拽视角时误触开盖，最好用 `pointerdown` 记录位置，`pointerup` 时检查移动距离，如果移动很小才算 click；或者直接使用 `click` 事件，OrbitControls 在拖拽时可能不会触发 click（浏览器通常如果 mousedown 和 mouseup 之间有移动，不一定触发 click，但有时候也会触发。比较保险的做法是判断鼠标位移）。
简单起见，用 `click` 事件即可，一般小幅拖拽不会触发 click（或者有触发但影响不大）。
更好的 `click` 检测：
```javascript
let mouseDownPos = new THREE.Vector2();
window.addEventListener('pointerdown', (e) => {
    mouseDownPos.set(e.clientX, e.clientY);
});
window.addEventListener('pointerup', (e) => {
    const dist = Math.hypot(e.clientX - mouseDownPos.x, e.clientY - mouseDownPos.y);
    if (dist < 5) { // 鼠标几乎没动，算作点击
        // 执行 raycaster 检测
        mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const intersects = raycaster.intersectObjects([leftLid, rightLid], true);
        if (intersects.length > 0) {
            let obj = intersects[0].object;
            while (obj && !(obj.userData && obj.userData.hinge)) {
                obj = obj.parent;
            }
            if (obj && obj.userData.hinge) {
                obj.userData.isOpen = !obj.userData.isOpen;
                obj.userData.targetRot = obj.userData.isOpen ? -Math.PI / 2 : 0;
            }
        }
    }
});
```
这样体验更好。

关于 Three.js 的版本：
`https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js` 和 OrbitControls 的 CDN：
`https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`
这两个版本匹配，都在 0.128 / r128。没问题。

再检查一下动画循环中的开合逻辑：
```javascript
[leftLid, rightLid].forEach(lid => {
    const hinge = lid.userData.hinge;
    // 如果当前角度和目标角度有差异，进行插值
    if (Math.abs(hinge.rotation.x - lid.userData.targetRot) > 0.001) {
        hinge.rotation.x += (lid.userData.targetRot - hinge.rotation.x) * 0.15;
    } else {
        hinge.rotation.x = lid.userData.targetRot;
    }
});
```
没问题。

还有一点：当盖子打开时，是向上翻起的（绕 X 轴旋转 -90 度，由原本水平在底板上，变成垂直竖立，且向后倒，因为铰链在后面）。这很符合真实的翻盖式隐形眼镜盒。
等等，铰链在后面（z = -1.2），盖子原本在 z = 0（前），往上翻开（-90 度绕 X），盖子中心会移动到 y = 0.5 + 1.2 = 1.7，z = -1.2 的位置，也就是向上且向后。这正是向后翻开的动作，非常合理。

在 3D 场景里加个地面（GridHelper 或者单纯一个平面），让阴影有地方投射，会更美观。
如果不加地面，把 `renderer.setClearColor(0xf5f5f5)` 也可以，但阴影只在物体之间（self-shadow），这已经不错了。为了更好看，可以加一个极简的地面接收阴影：
```javascript
const groundGeo = new THREE.PlaneGeometry(50, 50);
const groundMat = new THREE.ShadowMaterial({ opacity: 0.1 }); // 只显示阴影，透明
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.position.y = 0; // 因为底座在 y从0开始，底面正好在 y=0
ground.receiveShadow = true;
scene.add(ground);
```
`ShadowMaterial` 非常好，只显示阴影，不挡住背景色。 backdrop 还是 `scene.background = new THREE.Color(0xf5f5f5);`。

光源：
```javascript
const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;
dirLight.shadow.mapSize.width = 1024;
dirLight.shadow.mapSize.height = 1024;
dirLight.shadow.camera.near = 0.5;
dirLight.shadow.camera.far = 50;
dirLight.shadow.camera.left = -10;
dirLight.shadow.camera.right = 10;
dirLight.shadow.camera.top = 10;
dirLight.shadow.camera.bottom = -10;
scene.add(dirLight);
```
再加个半球光或者环境光，让颜色明亮：
```javascript
const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.4);
scene.add(hemiLight);
```

相机位置：
`camera.position.set(0, 7, 8);`
`controls.target.set(0, 0.5, 0);` 看向盒子中心稍微偏上。
这样斜俯视看得很清楚。

盖子的半径：`lidRadius = 1.2`，这是正确的。
`lidHeight = 0.4`，这是盖子厚度。对于真实眼镜盒，盖子可能稍微薄一点（比如 0.2），但厚一点在 3D 里更明显，不容易看错，0.4 可以。如果改成 0.3，更好看：
`lidHeight = 0.3`，那么 `lidMesh.position.set(0, lidHeight/2, lidRadius);` 自动适应。
标签位置 `labelMesh.position.y = lidHeight/2 + 0.01;` 也自动适应。
凹槽：`slotGeo = new THREE.CylinderGeometry(1.0, 1.0, 0.2, 32);` 半径比盖子 1.2 小，正好盖子能盖住，并且边缘包住凹槽，合理。

还有一个细节，隐形眼镜盒通常左右两边各有一个小孔用来透气，或者只是装饰，但我们模型上已经有了醒目的大 L 和 R 标识，这就足够满足“醒目的 L 和 R 标识”了。

把代码拼装成单文件 HTML：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D 隐形眼镜盒</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #f5f5f5; font-family: sans-serif; }
        #info {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            color: #333;
            pointer-events: none;
            font-size: 18px;
            text-shadow: 1px 1px 2px rgba(255,255,255,0.8);
        }
        #info p { margin: 5px 0; font-size: 14px; color: #666; }
    </style>
</head>
<body>
    <div id="info">
        <strong>3D 隐形眼镜盒模型</strong>
        <p>点击左(L)或右(R)盒盖可将其打开 / 关闭。拖拽可旋转视角，滚轮可缩放。</p>
    </div>

    <!-- 引入 Three.js 和 OrbitControls -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

    <script>
        // ... 全部 JS 代码 ...
    </script>
</body>
</html>
```

编写 JS 部分：
```javascript
// 1. 基础场景设置
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xf5f5f5);

const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(0, 7, 8);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.target.set(0, 0.5, 0);

// 2. 灯光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.4);
scene.add(hemiLight);

const dirLight = new THREE.DirectionalLight(0xffffff, 0.7);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;
dirLight.shadow.mapSize.width = 1024;
dirLight.shadow.mapSize.height = 1024;
dirLight.shadow.camera.near = 0.5;
dirLight.shadow.camera.far = 50;
dirLight.shadow.camera.left = -10;
dirLight.shadow.camera.right = 10;
dirLight.shadow.camera.top = 10;
dirLight.shadow.camera.bottom = -10;
scene.add(dirLight);

// 3. 地面（只接收阴影）
const groundGeo = new THREE.PlaneGeometry(50, 50);
const groundMat = new THREE.ShadowMaterial({ opacity: 0.15 });
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.position.y = 0;
ground.receiveShadow = true;
scene.add(ground);

// 4. 底座与凹槽
const baseGroup = new THREE.Group();

// 底板
const baseGeo = new THREE.BoxGeometry(5, 0.5, 3);
const baseMat = new THREE.MeshStandardMaterial({ color: 0xfafafa, roughness: 0.5 });
const baseMesh = new THREE.Mesh(baseGeo, baseMat);
baseMesh.position.y = 0.25; // 底部在 y=0，顶部在 y=0.5
baseMesh.receiveShadow = true;
baseMesh.castShadow = true;
baseGroup.add(baseMesh);

// 凹槽 (左)
const slotGeo = new THREE.CylinderGeometry(1.0, 1.0, 0.2, 32);
const slotMat = new THREE.MeshStandardMaterial({ color: 0x81d4fa, roughness: 0.2 }); // 淡蓝色护理液感
const leftSlot = new THREE.Mesh(slotGeo, slotMat);
leftSlot.position.set(-1.2, 0.4, 0); // 顶部在 0.5，嵌在底板里
leftSlot.receiveShadow = true;
baseGroup.add(leftSlot);

// 凹槽 (右)
const rightSlot = new THREE.Mesh(slotGeo, slotMat.clone());
rightSlot.material.color.set(0xffe082); // 右边换个颜色或者一样，一般两边一样，就都用淡蓝
rightSlot.material.color.set(0x81d4fa);
rightSlot.position.set(1.2, 0.4, 0);
rightSlot.receiveShadow = true;
baseGroup.add(rightSlot);

scene.add(baseGroup);

// 5. 生成带 L/R 文字的纹理
function createTextTexture(text, bgColor) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, 256, 256);
    ctx.beginPath();
    ctx.arc(128, 128, 120, 0, Math.PI * 2);
    ctx.fillStyle = bgColor;
    ctx.fill();
    
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 138);
    
    return new THREE.CanvasTexture(canvas);
}

const leftTex = createTextTexture('L', '#ef5350'); // 红色
const rightTex = createTextTexture('R', '#42a5f5'); // 蓝色

// 6. 创建盖子（带铰链机制）
const lidHeight = 0.3;
const lidRadius = 1.2;

function createLid(texture, xPos, isLeft) {
    // 铰链放在盖子后侧
    const hinge = new THREE.Group();
    hinge.position.set(xPos, 0.5, -lidRadius);
    
    // 盖子材质：侧面用淡色，顶底面用白色
    const sideMat = new THREE.MeshStandardMaterial({ 
        color: isLeft ? 0xffebee : 0xe3f2fd, 
        roughness: 0.3 
    });
    const plainMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 });
    const lidMat = [sideMat, plainMat, plainMat];
    
    const lidGeo = new THREE.CylinderGeometry(lidRadius, lidRadius, lidHeight, 32);
    const lidMesh = new THREE.Mesh(lidGeo, lidMat);
    // 盖子相对铰链往前放 lidRadius，使得铰链在盖子后缘
    lidMesh.position.set(0, lidHeight / 2, lidRadius);
    lidMesh.castShadow = true;
    lidMesh.receiveShadow = true;
    
    // 添加文字标签平面
    const labelGeo = new THREE.PlaneGeometry(lidRadius * 1.6, lidRadius * 1.6);
    const labelMat = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const labelMesh = new THREE.Mesh(labelGeo, labelMat);
    labelMesh.rotation.x = -Math.PI / 2; // 平放在盖顶
    labelMesh.position.y = lidHeight / 2 + 0.01; // 避免 Z-fighting
    lidMesh.add(labelMesh);
    
    // 用户数据，记录开合状态
    lidMesh.userData = { hinge: hinge, isOpen: false, targetRot: 0 };
    
    hinge.add(lidMesh);
    scene.add(hinge);
    
    return lidMesh;
}

const leftLid = createLid(leftTex, -1.2, true);
const rightLid = createLid(rightTex, 1.2, false);

// 7. 交互：点击开盖
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let mouseDownPos = new THREE.Vector2();

window.addEventListener('pointerdown', (e) => {
    mouseDownPos.set(e.clientX, e.clientY);
});

window.addEventListener('pointerup', (e) => {
    const dist = Math.hypot(e.clientX - mouseDownPos.x, e.clientY - mouseDownPos.y);
    if (dist > 5) return; // 拖拽，不算点击
    
    mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects([leftLid, rightLid], true);
    
    if (intersects.length > 0) {
        let obj = intersects[0].object;
        // 找到包含 hinge 数据的父级（盖子本体）
        while (obj && !(obj.userData && obj.userData.hinge)) {
            obj = obj.parent;
        }
        if (obj && obj.userData.hinge) {
            obj.userData.isOpen = !obj.userData.isOpen;
            obj.userData.targetRot = obj.userData.isOpen ? -Math.PI / 2 : 0;
        }
    }
});

// 8. 窗口自适应
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});

// 9. 动画循环
function animate() {
    requestAnimationFrame(animate);
    
    // 平滑开合盖子
    [leftLid, rightLid].forEach(lid => {
        const hinge = lid.userData.hinge;
        if (Math.abs(hinge.rotation.x - lid.userData.targetRot) > 0.001) {
            hinge.rotation.x += (lid.userData.targetRot - hinge.rotation.x) * 0.15;
        } else {
            hinge.rotation.x = lid.userData.targetRot;
        }
    });
    
    controls.update();
    renderer.render(scene, camera);
}

animate();
```

检查细节：
`rightSlot.material.color.set(0x81d4fa);` 这一点我写重复了，克隆后先 set 了 `0xffe082`，又 set 了 `0x81d4fa`。直接去掉多余的即可。
```javascript
const rightSlot = new THREE.Mesh(slotGeo, slotMat.clone());
rightSlot.position.set(1.2, 0.4, 0);
```
这就是淡蓝色了，两边一样。

标签平面的尺寸：`lidRadius * 1.6` = `1.2 * 1.6 = 1.92`。盖子直径是 2.4。所以标签直径 1.92，比盖子小一点，四周留出白边（盖子本身的白色），这很好看。
材质 `transparent: true` 保证了 canvas 里的透明背景生效，只显示中间彩色圆和字。

测试说明和交付要求：
用户要求：直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。
输出要求：只给我一个文件。不要查看其他文件，直接创建。

所以我的回答应该是：
1. 一段说明（运行方式、测试说明）。
2. 完整的单文件 HTML 代码（用 markdown code block 包裹，说明这是单文件）。

运行方式：
- 将提供的代码保存为 `index.html`。
- 由于使用了 CDN 引入 Three.js，直接双击在浏览器中打开即可（需联网）。或者通过本地服务器（如 `python -m http.server`）打开。

测试说明：
- 页面加载后，应显示一个静音灰白色的双联隐形眼镜盒模型，左右各有一个凹槽。
- 左盖顶部有醒目的红色圆形标签带白字 "L"，右盖有蓝色圆形标签带白字 "R"。
- 使用鼠标拖拽可以旋转视角，滚轮可以缩放。
- 点击左盖或右盖（或标签），对应盖子会以流畅的动画向后翻开（呈 90 度竖立）。再次点击，盖子会合上。
- 点击空白处或拖拽视角不会触发开盖。

所有需求都满足了：
- Three.js 制作 3D 模型。
- 隐形眼镜盒，有 L 和 R 标识（醒目，红蓝对比）。
- 配有盒盖（两个独立盒盖，有铰链翻盖结构）。
- 用户能点击盒盖将其打开（Raycaster + 动画）。
- 使用 HTML, CSS, JS，单文件。
- 直接创建，不查看其他文件。

代码再次校验：
`intersectObjects([leftLid, rightLid], true)` 正确。第二个参数 `recursive` 默认为 `true`，但显式传 `true` 更好。
在找父级时 `obj = obj.parent;` 如果一直找到 `scene` 或者 `null` 会退出循环。由于左右盖的 `hinge` 在 scene 下，`lidMesh` 在 `hinge` 下，所以 `obj` 最多找到 `lidMesh` 就会命中 `userData.hinge`，不会出问题。如果没有命中（比如点到了别处，但是 intersects 有长度的情况？因为我们只检测 [leftLid, rightLid]，所以 intersects 里的 object 必定是它们的子代或自身），所以肯定能找到。
`controls.update()` 只有在 `enableDamping` 时才需要在循环中调用，这里写了。

一个很小的问题：OrbitControls 在 r128 中，是通过 `THREE.OrbitControls` 访问的（因为是全局脚本引入），所以 `new THREE.OrbitControls(...)` 是对的。

还有一个小细节：如果用户点击的是盖子侧面，由于 `raycaster` 检测到了 `lidMesh`，它往上找父级时直接就是 `lidMesh`（因为 `lidMesh` 本身有 `userData.hinge`），所以没问题。如果点到了平面标签 `labelMesh`，它往上找父级，找到 `lidMesh`，也成功了。

最后检查 HTML 格式，确保没有未闭合的标签等问题。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
...
```
结构完全正确。

补充说明：隐形眼镜盒的“盒盖”在题目里说的是“盒盖”（可能是指整个盒子的盖子，或者两个盖子的任意一个）。我做成了左右两个独立翻盖，点击任意一个开任意一个，这比做一个大翻盖更符合真实隐形眼镜盒的设计，也更好地展示了 L 和 R 的独立标识。功能完全符合并超越基础预期。

关于阴影：
`renderer.shadowMap.enabled = true;` 和 `renderer.shadowMap.type = THREE.PCFSoftShadowMap;` 已经设置。
`dirLight.castShadow = true;` 也已经设置。
几何体也都设置了 `castShadow` 和 `receiveShadow`。地面用了 `ShadowMaterial`。非常完整。

最后检查 CanvasTexture 是否在 WebGL 中需要 `texture.needsUpdate = true`。因为是在创建时直接传给 Material 的，Three.js 会在第一次渲染时自动上传纹理，所以不需要额外设置 `needsUpdate`，除非是在创建纹理后修改了 canvas 内容。这里没有后续修改，所以没问题。

OK，构思和代码都完备，可以撰写最终回复。