用户要求制作一个弓箭射击模拟游戏。
要求：
1. 四个靶子。
2. 玩家命中所有目标所用的时间越短，在排行榜上的名次就越高。
3. 不要查看其他文件，直接创建。
4. 输出要求：直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。
5. 编程语言/形式：通常这种要求最好用HTML/JS/CSS做一个浏览器里能跑的游戏，因为不需要安装额外环境，直接浏览器打开即可运行。或者做一个Python的终端游戏或Pygame游戏？但用户说“可运行产物”，并且“直接创建”，用HTML单文件最方便，跨平台且可以直接双击运行。

让我们构思一个HTML+JS的单文件弓箭射击模拟游戏。
游戏机制：
- 画面上有弓箭（可以用鼠标瞄准和射击，或者点击屏幕发射箭矢）。
- 四个靶子分布在画面上。
- 玩家点击靶子发射箭（模拟射击），命中后靶子消失或标记。
- 计时器从第一次射击或游戏开始计时，直到四个靶子全部命中。
- 记录时间，并有一个简单的排行榜（可以存localStorage或者只是内存中，或者简单硬编码一些虚拟成绩用于演示，或者纯粹记录本次和历史最好成绩）。
- 为了模拟“弓箭射击”，可以加上一些物理抛物线或者简单的飞行动画，或者就是点击即命中（鼠标瞄准）。为了有趣，可以用鼠标移动控制角度和力度（拉弓），松开射击，箭沿抛物线飞出，命中检测。
- 但简单一点：鼠标点击屏幕某处，弓箭从固定位置（比如左下角）射向点击位置（或者直接点击靶子就算命中，但加上飞行动画更有“射击模拟”的感觉）。

需求细化：
- 4个靶子：随机位置或固定位置。
- 射击：鼠标点击控制方向和力度（拉弓），或者简单点：鼠标移动瞄准，点击发射，箭头飞向鼠标点击点。
- 命中检测：箭矢飞行过程中如果碰到靶子，就判定命中。
- 计时：从第一次发射开始计时，到所有靶子被击中结束。
- 排行榜：根据所用时间排序。可以用localStorage保存历史记录，并展示在页面上。
- 单文件HTML，包含CSS和JS。

设计游戏：
1. 使用HTML5 Canvas绘制。
2. 玩家在底部中央（弓箭手位置）。
3. 靶子分布在上方不同位置。
4. 鼠标控制瞄准，按住鼠标左键蓄力（力度条），松开发射箭矢。
5. 箭矢受重力影响（抛物线），或者直线飞行？模拟弓箭通常有重力。我们就加一点重力，增加趣味性。
6. 命中靶子：检测箭矢坐标和靶子圆心距离是否小于靶子半径。
7. 四个靶子都命中后，停止计时，提示成绩，并询问名字保存到排行榜。
8. 排行榜存localStorage，显示在侧边或下方。

具体实现步骤：
- HTML结构：canvas元素，侧边栏/覆盖层显示排行榜、计时、说明。
- JS逻辑：
  - 游戏状态：ready, playing, finished.
  - 靶子数组：[{x, y, r, hit}]
  - 箭矢数组：[{x, y, vx, vy}]
  - 弓箭手位置：固定，比如 {x: canvas.width/2, y: canvas.height - 20}
  - 鼠标事件：mousedown记录起点（或者开始蓄力），mouseup发射。
  - 物理更新：requestAnimationFrame更新箭位置，应用重力。
  - 碰撞检测：遍历箭和靶子。
  - 计时器：performance.now()。
  - 排行榜：localStorage.getItem('leaderboard')，JSON.parse，排序，取前10名展示。

让我们写代码。使用单文件 `archery.html`。

需要注意用户要求：“不要查看其他文件，直接创建。请直接完成任务并交付可运行产物”。所以我应该直接输出完整的HTML代码，并在回答中提供运行方式和测试说明。

代码结构：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>弓箭射击模拟游戏</title>
<style>
  body { margin: 0; font-family: sans-serif; background: #222; color: #fff; display: flex; flex-direction: column; align-items: center; }
  canvas { background: #87CEEB; display: block; margin-top: 10px; cursor: crosshair; }
  #ui { width: 800px; display: flex; justify-content: space-between; margin-top: 10px;}
  .panel { background: #333; padding: 10px; border-radius: 5px; }
  button { padding: 5px 10px; margin-top: 10px; cursor: pointer; }
  table { border-collapse: collapse; width: 100%; }
  th, td { border: 1px solid #555; padding: 4px 8px; text-align: left; }
  #message { color: #ff0; margin-top: 5px; }
</style>
</head>
<body>
<h2>弓箭射击模拟游戏</h2>
<canvas id="gameCanvas" width="800" height="500"></canvas>
<div id="ui">
  <div class="panel" style="flex:1; margin-right:10px;">
    <h3>游戏状态</h3>
    <p>时间: <span id="timer">0.00</span> 秒</p>
    <p>剩余靶子: <span id="targetsLeft">4</span></p>
    <p>操作: 在画布上按住鼠标左键蓄力，松开射箭。瞄准上方四个靶子！</p>
    <div id="message"></div>
    <button id="restartBtn">重新开始</button>
  </div>
  <div class="panel" style="flex:1;">
    <h3>排行榜 (用时越短越好)</h3>
    <table id="leaderboard">
      <thead><tr><th>排名</th><th>玩家</th><th>时间(秒)</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>
</div>

<script>
// JS代码
</script>
</body>
</html>
```

JS逻辑详细设计：
- 变量：
  `canvas`, `ctx`
  `targets = []`
  `arrows = []`
  `archer = {x: 400, y: 480}`
  `isDrawing = false`
  `drawStart = {x, y}` (可以用鼠标按下的位置来计算力度和方向，或者更简单：弓箭手朝鼠标位置发射，蓄力时间决定速度)
  为了简单且好操作：按下鼠标开始蓄力（显示力度条），松开时根据蓄力大小和鼠标方向计算速度。
  或者：鼠标位置就是瞄准方向，按下时间决定力度。
  我们就做：鼠标在canvas上移动，显示一条瞄准线（从弓箭手到鼠标）。按住鼠标左键，力度增加（最大限制），松开时发射。箭初始速度方向为弓箭手到鼠标的方向，速度大小基于蓄力。
  重力 `g = 0.2` (每帧加速度)。
  箭每帧更新： `vy += g; x += vx; y += vy;`
  碰撞：如果箭和未命中靶子距离 < 靶子半径，标记靶子命中，移除箭（或者让箭插在靶子上）。
  计时：游戏开始时（第一次发射箭时）开始计时，四个靶子全中时停止。
  排行榜：提示输入名字，存入localStorage。

让我们实现蓄力机制：
`mousedown` -> `isCharging = true; chargeStart = performance.now();`
`mouseup` -> `isCharging = false; let charge = Math.min((performance.now() - chargeStart)/1000, 1.0);` (charge 0~1)
方向： `angle = Math.atan2(mouse.y - archer.y, mouse.x - archer.x);`
速度： `speed = 5 + charge * 15;` (5到20)
`vx = Math.cos(angle)*speed; vy = Math.sin(angle)*speed;`
发射箭： `arrows.push({x: archer.x, y: archer.y, vx, vy, stuck: false})`

如果箭飞出边界，移除。
如果箭击中靶子，箭变成stuck，靶子hit=true。
所有靶子hit后，如果游戏没结束，则结束，记录时间。

为了展示瞄准，可以在 `mousemove` 里记录 `mouse` 坐标，并在 `draw` 里如果 `isCharging`，画一个力度条和预测线（简单点画线到鼠标）。

靶子初始化：
4个靶子，分布在上方，比如：
`[{x: 150, y: 100, r: 30}, {x: 350, y: 180, r: 30}, {x: 550, y: 120, r: 30}, {x: 700, y: 250, r: 30}]`
（加点随机偏移也行，但固定也不错，或者每次重开随机高度和x）。

游戏循环：
```js
function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}
```

更新逻辑：
```js
if (gameState === 'playing' && startTime) {
  elapsed = (performance.now() - startTime) / 1000;
  timerEl.textContent = elapsed.toFixed(2);
}
// update arrows
for (let i = arrows.length - 1; i >= 0; i--) {
  let a = arrows[i];
  if (!a.stuck) {
    a.vy += 0.2; // gravity
    a.x += a.vx;
    a.y += a.vy;
    // check out of bounds
    if (a.x < 0 || a.x > canvas.width || a.y > canvas.height) {
      arrows.splice(i, 1);
      continue;
    }
    // check collision with targets
    for (let t of targets) {
      if (!t.hit) {
        let dx = a.x - t.x;
        let dy = a.y - t.y;
        if (dx*dx + dy*dy <= t.r*t.r) {
          t.hit = true;
          a.stuck = true; // arrow stays
          targetsLeft--;
          targetsLeftEl.textContent = targetsLeft;
          if (targetsLeft === 0) {
            endGame();
          }
          break;
        }
      }
    }
  }
}
```

绘图逻辑：
- 背景天空（CSS已经做了，canvas可以清屏为透明或天空蓝，用 `clearRect` 即可，因为CSS背景是天蓝）。
- 画弓箭手（简单的线条或圆）。
- 画靶子（同心圆，命中后变灰或画个箭）。
- 画箭（线条）。
- 画蓄力条和瞄准线。

结束游戏：
```js
function endGame() {
  gameState = 'finished';
  let finalTime = (performance.now() - startTime) / 1000;
  let name = prompt(`恭喜！你用了 ${finalTime.toFixed(2)} 秒击中所有靶子。\n请输入你的名字记录到排行榜：`, '玩家');
  if (name) {
    saveScore(name, finalTime);
  }
  renderLeaderboard();
  document.getElementById('message').textContent = '游戏结束！点击“重新开始”再玩一次。';
}
```

排行榜存储：
```js
function saveScore(name, time) {
  let lb = JSON.parse(localStorage.getItem('archery_lb') || '[]');
  lb.push({name, time});
  lb.sort((a,b) => a.time - b.time);
  lb = lb.slice(0, 10); // top 10
  localStorage.setItem('archery_lb', JSON.stringify(lb));
}
function renderLeaderboard() {
  let lb = JSON.parse(localStorage.getItem('archery_lb') || '[]');
  let tbody = document.querySelector('#leaderboard tbody');
  tbody.innerHTML = '';
  lb.forEach((entry, i) => {
    let tr = document.createElement('tr');
    tr.innerHTML = `<td>${i+1}</td><td>${entry.name}</td><td>${entry.time.toFixed(2)}</td>`;
    tbody.appendChild(tr);
  });
}
```

开始游戏/重置：
```js
function resetGame() {
  targets = [
    {x: 100+Math.random()*100, y: 80+Math.random()*100, r: 25, hit: false},
    {x: 300+Math.random()*100, y: 80+Math.random()*100, r: 25, hit: false},
    {x: 500+Math.random()*100, y: 80+Math.random()*100, r: 25, hit: false},
    {x: 650, y: 80+Math.random()*150, r: 25, hit: false}
  ];
  arrows = [];
  targetsLeft = 4;
  targetsLeftEl.textContent = targetsLeft;
  startTime = null;
  gameState = 'ready';
  document.getElementById('message').textContent = '准备就绪，按住鼠标左键蓄力，松开射箭！';
  timerEl.textContent = '0.00';
}
```
第一次射箭时启动计时：
```js
if (gameState === 'ready') {
  gameState = 'playing';
  startTime = performance.now();
}
```

测试说明：
- 描述如何打开HTML文件。
- 描述操作方式（蓄力、发射）。
- 描述排行榜功能（localStorage持久化）。

完善细节：
- 防止页面滚动：`e.preventDefault()` 在mousedown里。
- 考虑鼠标坐标获取相对于canvas：
```js
function getMousePos(e) {
  let rect = canvas.getBoundingClientRect();
  return { x: e.clientX - rect.left, y: e.clientY - rect.top };
}
```

蓄力机制：为了视觉效果，可以用鼠标距离按下点的距离作为力度？还是时间？用时间更简单稳健。
但是用“拉弓”感觉：鼠标按下后，移动鼠标拉弓（向后），松开射出。这会比较复杂。
最简单的“拉开弩”隐喻：按住鼠标，力度条增长，松开发射。方向是瞄准线方向（从弓箭手指向当前鼠标位置）。这很直观。

我们加个力度条：在弓箭手旁边画个竖条，或者直接在UI显示。画在canvas上更好看。

代码拼装检查：
- HTML结构完整，CSS简洁，JS逻辑清晰。
- 没有外部依赖，纯原生。
- 代码没有引用其他文件。
- 直接交付一个完整的HTML文件内容。

用户要求：“直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。”
因此我的回答应该是：
1. 说明交付的是一个单文件HTML游戏。
2. 提供完整的代码（放在代码块里，或者说明保存为某文件）。
3. 提供运行方式（双击打开，或者浏览器打开）。
4. 提供测试说明（如何玩，怎么验证排行榜等）。

稍微优化游戏体验：
- 给靶子画成同心圆（红白相间），像真实箭靶。
- 弓箭手画成一个小人和弓。
- 箭画成一条线带箭头。
- 如果箭没击中靶子出界了，就消失。
- 在游戏结束时，若排行榜更新，则显示。
- 可以在命中时加个简单的文字提示或者音效？音效需要Web Audio，不必了，视觉上足够。

让我们把代码仔细写出来，确保没有语法错误。
另外，注意 `requestAnimationFrame` 循环开始时 `ctx.clearRect`。

靶子画法：
```js
function drawTarget(t) {
  ctx.beginPath();
  ctx.arc(t.x, t.y, t.r, 0, Math.PI*2);
  ctx.fillStyle = t.hit ? '#888' : '#fff';
  ctx.fill();
  ctx.lineWidth = 2;
  ctx.strokeStyle = '#000';
  ctx.stroke();

  ctx.beginPath();
  ctx.arc(t.x, t.y, t.r*0.66, 0, Math.PI*2);
  ctx.fillStyle = t.hit ? '#666' : '#0a0';
  ctx.fill();

  ctx.beginPath();
  ctx.arc(t.x, t.y, t.r*0.33, 0, Math.PI*2);
  ctx.fillStyle = t.hit ? '#444' : '#f00';
  ctx.fill();
}
```

弓箭手和瞄准：
```js
function drawArcher() {
  ctx.fillStyle = '#333';
  ctx.fillRect(archer.x - 10, archer.y - 20, 20, 40); // body
  // bow
  ctx.beginPath();
  ctx.arc(archer.x, archer.y - 10, 15, -Math.PI/3, Math.PI/3);
  ctx.strokeStyle = '#8B4513';
  ctx.lineWidth = 3;
  ctx.stroke();
}
```

瞄准线：
```js
if (mousePos) {
  ctx.beginPath();
  ctx.moveTo(archer.x, archer.y - 10);
  ctx.lineTo(mousePos.x, mousePos.y);
  ctx.strokeStyle = 'rgba(255,255,255,0.5)';
  ctx.setLineDash([5,5]);
  ctx.stroke();
  ctx.setLineDash([]);
}
```

蓄力条：
```js
if (isCharging) {
  let charge = Math.min((performance.now() - chargeStart) / 1000, 1.0);
  ctx.fillStyle = 'rgba(0,0,0,0.5)';
  ctx.fillRect(archer.x - 20, archer.y + 30, 40, 10);
  ctx.fillStyle = '#f00';
  ctx.fillRect(archer.x - 20, archer.y + 30, 40 * charge, 10);
}
```

发射逻辑：
```js
canvas.addEventListener('mousedown', e => {
  if (gameState === 'finished') return;
  isCharging = true;
  chargeStart = performance.now();
});
canvas.addEventListener('mouseup', e => {
  if (!isCharging) return;
  isCharging = false;
  let charge = Math.min((performance.now() - chargeStart) / 1000, 1.0);
  let angle = Math.atan2(mousePos.y - (archer.y-10), mousePos.x - archer.x);
  let speed = 5 + charge * 15;
  let vx = Math.cos(angle) * speed;
  let vy = Math.sin(angle) * speed;
  arrows.push({x: archer.x, y: archer.y-10, vx, vy, stuck: false});
  if (gameState === 'ready') {
    gameState = 'playing';
    startTime = performance.now();
  }
});
canvas.addEventListener('mousemove', e => {
  mousePos = getMousePos(e);
});
```
（注意，如果mousePos还没初始化，mouseup时可能会出问题，所以初始时mousePos设为 {x: archer.x, y: archer.y-100} 向上的位置）。

检查箭的绘制：
```js
for (let a of arrows) {
  ctx.beginPath();
  ctx.moveTo(a.x, a.y);
  // 箭尾在反方向
  let angle = Math.atan2(a.vy, a.vx);
  let tailX = a.x - Math.cos(angle)*15;
  let tailY = a.y - Math.sin(angle)*15;
  ctx.lineTo(tailX, tailY);
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 2;
  ctx.stroke();
  // arrow head
  ctx.beginPath();
  ctx.arc(a.x, a.y, 3, 0, Math.PI*2);
  ctx.fillStyle = '#000';
  ctx.fill();
}
```

检查命中和stuck箭：
如果箭头stuck，就不更新它的位置了。它就停留在靶子上。
所以draw里，stuck的箭也画出来。

游戏结束条件：
四个靶子都hit，并且当前没有飞行中的箭？或者一命中最后一个就结束？通常命中最后一个瞬间就结束。
```js
if (targetsLeft === 0 && gameState === 'playing') {
  endGame();
}
```
但如果还有箭在飞，可能无所谓，不过最好就是命中即结束。

排行榜初始时可以加几条假数据吗？或者留空让用户自己创造。留空也可以，或者加个“AI玩家”示例。为了演示“名次越高”，可以预置几个。不过localStorage里如果没有就不显示。或者代码里写个初始默认排行榜？其实不用，真实记录就好。

让游戏支持多次重开：
`restartBtn.addEventListener('click', resetGame);`

确保所有元素都在一个文件里，不引用外部图片或库。

再检查一下重力：0.2可能太大或太小？
画布 800x500，射手在 y=480，靶子在 y=80~200。垂直距离约 300px。
如果speed=20，角度-60度，vy=-17，vx=10。
飞行时间到顶点：17/0.2 = 85帧，约1.4秒，上升距离 17*85/2 = 722px（太高了）。所以重力0.2稍微大一点或者speed改小一点。
为了好打中，我们降低重力，比如 `g = 0.1`，speed范围 `8 + charge*12` (8到20)。
其实不用太严格物理，只要能击中就行。甚至可以不要重力（直线射击），但“弓箭”有抛物线更真实。我们把重力设小一点，比如0.15。

另外，允许直线瞄准：如果玩家直接朝靶子瞄，由于重力箭会往下掉，需要稍微往上瞄。这就是游戏的技巧点。

测试说明里可以提到这点。

最后检查一下需求：
- 四个靶子：有了。
- 玩家命中所有目标所用的时间越短，在排行榜上的名次就越高：计时+排序，有了。
- 不要查看其他文件，直接创建：单HTML文件，没有外部依赖。
- 提供可运行产物、运行方式、测试说明：会包含在回复中。

是否有任何违反安全或政策的地方？没有，这是一个简单的游戏。

输出格式：
先给出一段简短说明，然后给出代码，再给出运行方式和测试说明。代码放在 ```html 代码块里，方便复制。也可以直接说“将以下代码保存为 archery.html 并用浏览器打开”。

我在输出时不要说太多废话，按照用户要求直接交付。但用户说“请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。”所以结构清晰即可。

让我们把代码整合一遍，保证可以直接运行。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>弓箭射击模拟游戏</title>
<style>
  body { margin: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #2c3e50; color: #ecf0f1; display: flex; flex-direction: column; align-items: center; }
  h2 { margin: 15px 0; }
  canvas { background: linear-gradient(#87CEEB, #e0f6ff); display: block; border: 2px solid #34495e; border-radius: 4px; cursor: crosshair; }
  #ui { width: 804px; display: flex; justify-content: space-between; margin-top: 15px; }
  .panel { background: #34495e; padding: 15px; border-radius: 6px; box-shadow: 0 2px 5px rgba(0,0,0,0.3); }
  .panel.left { flex: 1; margin-right: 10px; }
  .panel.right { flex: 1; margin-left: 10px; }
  p { margin: 8px 0; font-size: 14px; }
  button { padding: 8px 16px; margin-top: 15px; cursor: pointer; background: #e67e22; border: none; color: white; border-radius: 4px; font-weight: bold; }
  button:hover { background: #d35400; }
  table { border-collapse: collapse; width: 100%; font-size: 14px; }
  th, td { border: 1px solid #2c3e50; padding: 6px 8px; text-align: left; }
  th { background: #2c3e50; }
  #message { color: #f1c40f; margin-top: 10px; font-weight: bold; min-height: 20px; }
</style>
</head>
<body>
...
```

JS部分注意事项：
在 `mouseup` 事件里，如果 `mousePos` 存在且 `isCharging` 为 true 才发射。
由于 `mousePos` 是相对于 canvas，当鼠标在 canvas 外松开时 `mouseup` 可能不会触发在 canvas 上。可以给 `window` 加 `mouseup` 监听，更稳。

```js
window.addEventListener('mouseup', e => {
  if (!isCharging) return;
  isCharging = false;
  let charge = Math.min((performance.now() - chargeStart) / 1000, 1.0);
  let angle = Math.atan2(mousePos.y - (archer.y-10), mousePos.x - archer.x);
  let speed = 8 + charge * 12;
  let vx = Math.cos(angle) * speed;
  let vy = Math.sin(angle) * speed;
  arrows.push({x: archer.x, y: archer.y-10, vx, vy, stuck: false});
  if (gameState === 'ready') {
    gameState = 'playing';
    startTime = performance.now();
  }
});
```

重置靶子时，确保重置状态。
`renderLeaderboard` 在一开始也要调用一次，显示已有记录。

测试一下蓄力满是多少：按住1秒就是1.0，speed最大20。重力0.15，每帧0.15，60fps的话，1秒速度变化9。从-10到+10需要大概1.3秒。射程足够。

好了，逻辑严密，游戏可玩，满足所有需求。准备输出。

输出格式：
- 简短开场，说明文件。
- 完整HTML代码（可保存为 `archery_game.html`）。
- 运行方式。
- 测试说明（包括玩法、排行榜等）。

最后检查：用户要求“不要查看其他文件，直接创建”，这意味着不需要去读取或假设其他文件的存在，我创建的单文件本身自包含，符合。并且我不需要写“我已经查看了...”之类的话，直接产出结果。

代码细节再查一遍：
- 靶子数组 `targets` 有4个元素。
- 计时器显示更新在 `loop` 中。
- `requestAnimationFrame` 正确调用。
- 排行榜排序是升序（时间越短越好），正确。
- 坐标计算正确，没有使用可能导致NaN的计算。
- 一切正常。

（在最终的回答里，我会把代码放在一个代码块里，然后说明运行方式和测试说明。注意不要使用markdown嵌套错误。）