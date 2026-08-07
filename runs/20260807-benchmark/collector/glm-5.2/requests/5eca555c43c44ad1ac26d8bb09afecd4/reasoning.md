用户要求制作一个弓箭射击模拟游戏，要有四个靶子，玩家命中所有目标的时间越短，排行榜名次越高。需要直接创建可运行的HTML文件。

让我设计一下这个游戏：

1. **核心玩法**：
   - 玩家控制弓箭，瞄准并射出箭矢
   - 场景中有4个靶子
   - 需要命中所有4个靶子
   - 计时开始 → 命中所有靶子 → 计时结束
   - 时间越短，排行榜名次越高

2. **视觉设计**：
   - 不要默认蓝紫色
   - 选择一个有特色的配色 - 我想用一种森林/野外的感觉，深绿+琥珀色/橙色
   - 或者更独特一点：深色背景+霓虹色，赛博朋克风格的弓箭游戏
   - 让我用一个温暖的黄昏/夕阳色 - 琥珀色、橙色、深红，配合深色背景
   - 主色：琥珀金 #f5a623, 深红 #c41e3a，背景深棕黑 #1a0f0a

3. **交互设计**：
   - 鼠标控制瞄准方向
   - 拉弓蓄力（按住鼠标）→ 释放射箭
   - 蓄力越久，力度越大，飞行越远
   - 弓箭受重力影响，需要预判抛物线
   - 显示瞄准辅助线（可选，或部分显示）

4. **靶子**：
   - 4个不同位置/距离的靶子
   - 同心圆环，中心得分高
   - 命中后消失或破碎动画
   - 可能有些移动的靶子增加难度

5. **物理系统**：
   - 重力下落
   - 简单的2D弹道学

6. **UI元素**：
   - 顶部：计时器、命中数/总数、当前蓄力
   - 排行榜面板
   - 开始/重置按钮
   - 玩家名字输入

7. **排行榜**：
   - 存储在localStorage
   - 显示前10名
   - 显示名字+时间+日期

让我开始写代码。我会使用Canvas 2D来制作，因为这是2D游戏。

让我详细规划一下游戏场景：
- 侧视图，玩家弓箭手在左侧
- 靶子分布在右侧不同位置
- 鼠标控制瞄准角度
- 按住鼠标蓄力，松开射箭
- 箭矢抛物线飞行
- 命中靶子后靶子消失，箭矢插在上面

视觉风格：我想做一个夕阳/黄昏的射箭场风格
- 天空：深橙红渐变
- 远山剪影
- 草地/地面
- 弓箭手在左侧（剪影或简化绘制）
- 靶子是经典的红白圆环靶

让我写代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>黄昏弓道 - Archery</title>
<style>
/* ... */
</style>
</head>
<body>
<!-- ... -->
<script>
// ...
</script>
</body>
</html>
```

游戏状态机：
- IDLE: 等待开始
- PLAYING: 游戏中（计时中）
- AIMING: 瞄准中（按住鼠标蓄力）
- WIN: 全部命中
- LOSE: （这个游戏不一定需要输的条件）

游戏流程：
1. 玩家输入名字（可选，默认"匿名射手"）
2. 点击"开始挑战"
3. 计时开始，4个靶子出现
4. 玩家瞄准、蓄力、射箭
5. 命中所有4个靶子后计时停止
6. 显示成绩，加入排行榜
7. 可以再次挑战

物理参数：
- 重力：约0.4-0.6 像素/帧²
- 初速度根据蓄力时间，0-25 像素/帧
- 角度由鼠标位置决定（相对于弓的位置）

让我开始写代码。

我会精心设计配色：
- 背景：深棕黑 #1a0f0a
- 天空渐变：从 #2a1810 到 #d4521a 到 #f5a623
- 远山：#3a2418
- 草地：#2a1f12
- 强调色：琥珀金 #f5a623, 火红 #e74c3c
- 文字：#fff5e1（暖白）

字体：用Cinzel（衬线展示字体，带古典感）+ JetBrains Mono（数字/计时器）

让我开始具体实现：

```javascript
// 游戏配置
const config = {
  gravity: 0.45,
  maxPower: 25,
  powerSpeed: 0.4, // 每帧蓄力增加
  targetCount: 4,
  arrowSpeed: 1,
};

// 游戏状态
let game = {
  state: 'idle', // idle, playing, won
  startTime: 0,
  endTime: 0,
  playerName: '匿名射手',
  hits: 0,
  arrows: [], // 已射出的箭
  currentArrow: null, // 当前飞行的箭
  isCharging: false,
  power: 0,
  angle: 0,
  targets: [],
  bow: { x: 100, y: 400 },
};
```

我需要思考一下游戏的实现细节：

1. **弓箭手绘制**：简化为剪影，弓在手中
2. **瞄准**：鼠标位置决定角度，弓朝向鼠标
3. **蓄力**：按住鼠标左键，弓弯曲，能量条增长
4. **射箭**：松开鼠标，箭沿当前角度射出
5. **飞行**：抛物线轨迹
6. **碰撞**：检测箭头位置是否在靶子范围内
7. **命中效果**：靶子颤动、破碎粒子、声效（用WebAudio生成）

让我考虑一下靶子的设计：
- 靶子是同心圆，外圈白色，向内红黄交替，中心金色
- 命中时显示击中位置（环数）
- 4个靶子在不同距离/高度
- 可以让其中一两个靶子缓慢移动增加难度

我想让游戏更丰富：
- 风（影响箭的飞行）
- 距离指示器
- 角度指示器
- 弹道预览（淡化的轨迹点）

但不要过度复杂化，先实现核心玩法。

让我现在开始正式写代码。

代码结构：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>黄昏弓道 · Twilight Archery</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;800&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<style>
:root {
  --bg-deep: #0d0705;
  --bg: #1a0f0a;
  --sky-1: #2a1810;
  --sky-2: #8b3a1a;
  --sky-3: #d4521a;
  --sky-4: #f5a623;
  --accent: #f5a623;
  --accent-hot: #e74c3c;
  --gold: #ffd700;
  --fg: #fff5e1;
  --muted: #a08868;
  --card: rgba(40, 25, 15, 0.7);
  --border: rgba(245, 166, 35, 0.2);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: var(--bg-deep);
  color: var(--fg);
  font-family: 'Cinzel', serif;
  min-height: 100vh;
  overflow: hidden;
}

/* ... */
</style>
</head>
<body>
<div id="app">
  <header>
    <h1>黄昏弓道</h1>
    <p>射穿四靶，争分夺秒</p>
  </header>
  
  <main>
    <canvas id="game"></canvas>
    
    <div id="hud">
      <div class="hud-item">
        <span class="label">时间</span>
        <span class="value" id="timer">0.00</span>
      </div>
      <div class="hud-item">
        <span class="label">命中</span>
        <span class="value" id="hits">0/4</span>
      </div>
      <div class="hud-item">
        <span class="label">蓄力</span>
        <div class="power-bar"><div id="power-fill"></div></div>
      </div>
    </div>
    
    <div id="leaderboard">
      <h2>排行榜</h2>
      <ol id="rank-list"></ol>
    </div>
    
    <div id="start-panel">
      <h2>开始挑战</h2>
      <input id="player-name" placeholder="输入射手之名">
      <button id="start-btn">开始</button>
      <p>按住鼠标蓄力，松开射箭</p>
    </div>
    
    <div id="result-panel" hidden>
      <h2>挑战完成</h2>
      <p>用时 <span id="final-time"></span></p>
      <p>排名 <span id="final-rank"></span></p>
      <button id="restart-btn">再次挑战</button>
    </div>
  </main>
</div>

<script>
// ... 游戏代码
</script>
</body>
</html>
```

现在我来写完整的游戏代码。让我仔细考虑物理和绘制：

**坐标系统**：
- Canvas: 1200x600（响应式缩放）
- 弓箭手位置: (120, 380)
- 地面: y = 480
- 靶子位置: 不同距离和高度

**靶子设计**：
4个靶子，难度递增：
1. 距离400，高度350 - 近距离，简单
2. 距离700，高度300 - 中距离
3. 距离900，高度400 - 远距离，地面附近
4. 距离1050，高度200 - 远距离，高处

或者让其中一个移动？让我让第4个靶子上下移动，增加难度。

**箭的物理**：
```
arrow.x += arrow.vx
arrow.y += arrow.vy
arrow.vy += gravity
arrow.rotation = atan2(arrow.vy, arrow.vx)
```

**蓄力**：
- 按下鼠标开始蓄力
- power 从0增长到maxPower
- 蓄力越满，箭飞得越远
- 松开鼠标射箭

**碰撞检测**：
箭头位置（前端）与靶子中心距离 < 靶子半径

**绘制箭**：
- 旋转角度跟随速度方向
- 主体木杆 + 末端羽毛 + 前端箭头

让我考虑一下UI布局。Canvas是主视觉，旁边放HUD和排行榜。

考虑响应式：使用CSS让canvas适应屏幕，逻辑坐标保持不变。

让我详细写一下：

```javascript
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

// 逻辑尺寸
const W = 1200, H = 600;
canvas.width = W;
canvas.height = H;

// 地面、天空、太阳
const GROUND_Y = 480;
const bow = { x: 120, y: 380 };

// 靶子
const targets = [
  { x: 400, y: 320, r: 35, hit: false, baseY: 320, moveAmp: 0, movePhase: 0 },
  { x: 700, y: 280, r: 30, hit: false, baseY: 280, moveAmp: 0, movePhase: 0 },
  { x: 950, y: 380, r: 28, hit: false, baseY: 380, moveAmp: 0, movePhase: 0 },
  { x: 1100, y: 220, r: 25, hit: false, baseY: 220, moveAmp: 60, movePhase: 0 },
];
```

等等，1100可能太靠右了，离canvas边缘太近。让我调整一下。

实际上canvas宽1200，那1100离右边缘100像素，应该OK。但我想让最后一个靶子有特殊效果，比如移动。

让我重新设计靶子位置：
1. (380, 360) - 近，静止
2. (620, 290) - 中距离，静止  
3. (820, 400) - 远，静止，地面附近
4. (1050, 230) - 远，移动（上下浮动）

每个靶子的r（半径）：35, 30, 28, 25（难度递增）

**视觉细节**：
- 天空渐变（夕阳）
- 太阳在远处地平线附近
- 远山剪影
- 草地纹理
- 飞鸟剪影（远处）
- 飘落的叶子/羽毛粒子

**命中效果**：
- 靶子震动
- 木屑/碎片粒子
- 屏幕轻微震动
- 命中数字弹出
- 箭插在靶子上（保持显示）

**未命中效果**：
- 箭落地
- 落地后慢慢消失或保持
- 可以限制每局箭的数量？不限制，但记录射出次数

让我加上"箭数"统计，作为辅助数据，但主排名按时间。

好，现在开始写代码。

```javascript
// === 游戏状态 ===
const STATE = {
  IDLE: 'idle',
  PLAYING: 'playing',
  WON: 'won',
};

let game = {
  state: STATE.IDLE,
  startTime: 0,
  endTime: 0,
  playerName: '匿名射手',
  arrowsShot: 0,
  hits: 0,
  arrows: [], // 所有飞行的箭
  stuckArrows: [], // 插在靶子上或地上的箭
  isCharging: false,
  chargeStart: 0,
  power: 0,
  mouseX: 0,
  mouseY: 0,
  targets: [],
  particles: [],
  floatingTexts: [],
  shake: 0,
};
```

让我开始写完整代码：

实际上，让我把整个游戏整理一下，写得更紧凑高效：

```javascript
// 配置
const CFG = {
  W: 1200, H: 600,
  GROUND_Y: 480,
  BOW: { x: 120, y: 380 },
  GRAVITY: 0.42,
  MAX_POWER: 26,
  CHARGE_RATE: 0.45,
  ARROW_LEN: 50,
};

// 靶子配置
const TARGET_CONFIG = [
  { x: 380, y: 360, r: 38, moveAmp: 0, moveSpd: 0 },
  { x: 640, y: 280, r: 32, moveAmp: 0, moveSpd: 0 },
  { x: 850, y: 410, r: 28, moveAmp: 0, moveSpd: 0 },
  { x: 1060, y: 220, r: 26, moveAmp: 50, moveSpd: 0.025 },
];
```

让我考虑一下蓄力机制：
- 鼠标按下 → 开始蓄力
- power从0线性增长到MAX_POWER
- 鼠标松开 → 射箭
- 蓄力期间显示拉弓动画和能量条
- 蓄满后保持在MAX_POWER（不会爆）

蓄力曲线可以是：
- 0-30%快速
- 30-70%中速
- 70-100%慢速
这样玩家需要时间蓄满，但又不会等太久

或者简单线性，最快2秒蓄满。

射箭初速度 = power (向上限MAX_POWER)
角度 = atan2(mouseY - bow.y, mouseX - bow.x)
箭速度 = (cos(angle) * power, sin(angle) * power)

让我考虑碰撞检测：
- 箭的前端坐标 = arrow.x + cos(arrow.rot) * ARROW_LEN/2, arrow.y + sin(arrow.rot) * ARROW_LEN/2
- 检测这个点是否在某个靶子的圆内

箭的旋转 = atan2(vy, vx)

实际上更简单：
- 箭头的位置就是 arrow.x, arrow.y（前端）
- 检测 arrow.x, arrow.y 与靶子中心距离

让我写代码：

```javascript
function update() {
  if (game.state === STATE.PLAYING) {
    // 更新计时
    // ...
  }
  
  // 蓄力
  if (game.isCharging && game.power < CFG.MAX_POWER) {
    game.power += CFG.CHARGE_RATE;
    if (game.power > CFG.MAX_POWER) game.power = CFG.MAX_POWER;
  }
  
  // 更新箭
  for (let i = game.arrows.length - 1; i >= 0; i--) {
    const a = game.arrows[i];
    a.vy += CFG.GRAVITY;
    a.x += a.vx;
    a.y += a.vy;
    a.rot = Math.atan2(a.vy, a.vx);
    
    // 检测命中靶子
    let hit = false;
    for (const t of game.targets) {
      if (t.hit) continue;
      const dx = a.x - t.x;
      const dy = a.y - t.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if (dist < t.r) {
        t.hit = true;
        game.hits++;
        hit = true;
        // 计算环数（中心10环）
        const ring = Math.max(1, Math.ceil((1 - dist / t.r) * 10));
        // 粒子效果
        spawnHitParticles(a.x, a.y, t);
        // 浮动文字
        game.floatingTexts.push({
          x: a.x, y: a.y - 20,
          text: `${ring}环!`,
          life: 60, maxLife: 60,
          color: '#ffd700'
        });
        // 震动
        game.shake = 12;
        // 箭插在靶子上
        game.stuckArrows.push({
          x: a.x, y: a.y, rot: a.rot, life: 9999, stuck: true
        });
        game.arrows.splice(i, 1);
        
        // 检查是否全部命中
        if (game.hits >= CFG.TARGET_COUNT) {
          game.state = STATE.WON;
          game.endTime = performance.now();
          showResult();
        }
        break;
      }
    }
    
    if (hit) continue;
    
    // 检测落地
    if (a.y > CFG.GROUND_Y) {
      a.y = CFG.GROUND_Y;
      game.stuckArrows.push({
        x: a.x, y: a.y, rot: a.rot, life: 600, stuck: false
      });
      spawnDustParticles(a.x, a.y);
      game.arrows.splice(i, 1);
      continue;
    }
    
    // 超出边界
    if (a.x > CFG.W + 50 || a.x < -50 || a.y < -50) {
      game.arrows.splice(i, 1);
    }
  }
  
  // 更新靶子移动
  for (const t of game.targets) {
    if (t.moveAmp > 0) {
      t.movePhase += t.moveSpd;
      t.y = t.baseY + Math.sin(t.movePhase) * t.moveAmp;
    }
  }
  
  // 更新粒子
  for (let i = game.particles.length - 1; i >= 0; i--) {
    const p = game.particles[i];
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.15;
    p.vx *= 0.98;
    p.life--;
    if (p.life <= 0) game.particles.splice(i, 1);
  }
  
  // 更新浮动文字
  for (let i = game.floatingTexts.length - 1; i >= 0; i--) {
    const t = game.floatingTexts[i];
    t.y -= 1.5;
    t.life--;
    if (t.life <= 0) game.floatingTexts.splice(i, 1);
  }
  
  // 震动衰减
  if (game.shake > 0) game.shake *= 0.85;
  
  // 更新插在地上的箭
  for (let i = game.stuckArrows.length - 1; i >= 0; i--) {
    const a = game.stuckArrows[i];
    if (!a.stuck) {
      a.life--;
      if (a.life <= 0) game.stuckArrows.splice(i, 1);
    }
  }
}
```

绘制部分：

```javascript
function draw() {
  ctx.save();
  
  // 震动
  if (game.shake > 0.1) {
    const sx = (Math.random() - 0.5) * game.shake;
    const sy = (Math.random() - 0.5) * game.shake;
    ctx.translate(sx, sy);
  }
  
  // 天空
  const skyGrad = ctx.createLinearGradient(0, 0, 0, CFG.GROUND_Y);
  skyGrad.addColorStop(0, '#2a1810');
  skyGrad.addColorStop(0.4, '#5a2a15');
  skyGrad.addColorStop(0.7, '#d4521a');
  skyGrad.addColorStop(1, '#f5a623');
  ctx.fillStyle = skyGrad;
  ctx.fillRect(0, 0, CFG.W, CFG.GROUND_Y);
  
  // 太阳
  const sunX = CFG.W * 0.7;
  const sunY = CFG.GROUND_Y - 30;
  const sunGrad = ctx.createRadialGradient(sunX, sunY, 0, sunX, sunY, 200);
  sunGrad.addColorStop(0, 'rgba(255, 220, 100, 0.9)');
  sunGrad.addColorStop(0.3, 'rgba(245, 166, 35, 0.5)');
  sunGrad.addColorStop(1, 'rgba(245, 166, 35, 0)');
  ctx.fillStyle = sunGrad;
  ctx.fillRect(0, 0, CFG.W, CFG.GROUND_Y);
  
  ctx.fillStyle = '#ffe48a';
  ctx.beginPath();
  ctx.arc(sunX, sunY, 40, 0, Math.PI * 2);
  ctx.fill();
  
  // 远山
  ctx.fillStyle = '#2a1a14';
  ctx.beginPath();
  ctx.moveTo(0, CFG.GROUND_Y);
  ctx.lineTo(0, 420);
  ctx.lineTo(150, 380);
  ctx.lineTo(300, 410);
  ctx.lineTo(450, 370);
  ctx.lineTo(600, 400);
  ctx.lineTo(800, 360);
  ctx.lineTo(1000, 390);
  ctx.lineTo(1200, 380);
  ctx.lineTo(1200, CFG.GROUND_Y);
  ctx.closePath();
  ctx.fill();
  
  // 近山
  ctx.fillStyle = '#1a0f0a';
  ctx.beginPath();
  ctx.moveTo(0, CFG.GROUND_Y);
  ctx.lineTo(0, 460);
  ctx.lineTo(200, 440);
  ctx.lineTo(400, 460);
  ctx.lineTo(700, 445);
  ctx.lineTo(1000, 470);
  ctx.lineTo(1200, 455);
  ctx.lineTo(1200, CFG.GROUND_Y);
  ctx.closePath();
  ctx.fill();
  
  // 地面
  const groundGrad = ctx.createLinearGradient(0, CFG.GROUND_Y, 0, CFG.H);
  groundGrad.addColorStop(0, '#3a2418');
  groundGrad.addColorStop(1, '#1a0a05');
  ctx.fillStyle = groundGrad;
  ctx.fillRect(0, CFG.GROUND_Y, CFG.W, CFG.H - CFG.GROUND_Y);
  
  // 草地纹理（线条）
  ctx.strokeStyle = 'rgba(245, 166, 35, 0.15)';
  ctx.lineWidth = 1;
  for (let x = 0; x < CFG.W; x += 8) {
    ctx.beginPath();
    ctx.moveTo(x, CFG.GROUND_Y);
    ctx.lineTo(x + 2, CFG.GROUND_Y - 4 - Math.random() * 3);
    ctx.stroke();
  }
  
  // 绘制靶子
  for (const t of game.targets) {
    drawTarget(t);
  }
  
  // 绘制插着的箭
  for (const a of game.stuckArrows) {
    drawArrow(a.x, a.y, a.rot);
    if (!a.stuck) {
      // 淡出
      // 这里其实可以根据life调整透明度，但简化处理
    }
  }
  
  // 绘制飞行的箭
  for (const a of game.arrows) {
    drawArrow(a.x, a.y, a.rot);
  }
  
  // 绘制粒子
  for (const p of game.particles) {
    const alpha = p.life / p.maxLife;
    ctx.fillStyle = p.color + Math.floor(alpha * 255).toString(16).padStart(2, '0');
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r * alpha, 0, Math.PI * 2);
    ctx.fill();
  }
  
  // 绘制浮动文字
  for (const t of game.floatingTexts) {
    const alpha = t.life / t.maxLife;
    ctx.fillStyle = t.color;
    ctx.globalAlpha = alpha;
    ctx.font = 'bold 24px Cinzel';
    ctx.textAlign = 'center';
    ctx.fillText(t.text, t.x, t.y);
    ctx.globalAlpha = 1;
  }
  
  // 绘制弓箭手
  drawArcher();
  
  // 绘制瞄准辅助线
  if (game.state === STATE.PLAYING && game.isCharging) {
    drawAimLine();
  } else if (game.state === STATE.PLAYING) {
    drawAimLine(true); // 淡显示
  }
  
  ctx.restore();
}

function drawTarget(t) {
  if (t.hit) {
    // 命中后绘制残破的靶子（破碎效果）
    ctx.globalAlpha = 0.4;
  }
  
  // 支架
  ctx.fillStyle = '#3a2418';
  ctx.fillRect(t.x - 3, t.y, 6, CFG.GROUND_Y - t.y);
  
  // 靶面圆环
  const rings = [
    { r: t.r, color: '#f5f5dc' },
    { r: t.r * 0.8, color: '#1a1a1a' },
    { r: t.r * 0.6, color: '#3a8ed6' },
    { r: t.r * 0.4, color: '#e74c3c' },
    { r: t.r * 0.2, color: '#ffd700' },
  ];
  
  for (const ring of rings) {
    ctx.fillStyle = ring.color;
    ctx.beginPath();
    ctx.arc(t.x, t.y, ring.r, 0, Math.PI * 2);
    ctx.fill();
  }
  
  // 颤动效果（命中后短暂）
  // ...
  
  ctx.globalAlpha = 1;
}

function drawArcher() {
  const bx = CFG.BOW.x;
  const by = CFG.BOW.y;
  
  // 身体
  ctx.fillStyle = '#1a0f0a';
  ctx.beginPath();
  ctx.ellipse(bx - 5, by + 20, 10, 25, 0, 0, Math.PI * 2);
  ctx.fill();
  
  // 头
  ctx.beginPath();
  ctx.arc(bx - 5, by - 15, 10, 0, Math.PI * 2);
  ctx.fill();
  
  // 弓
  const angle = Math.atan2(game.mouseY - by, game.mouseX - bx);
  const bowR = 35;
  // 弓的中心在弓箭手前方
  const bowCx = bx + Math.cos(angle) * 15;
  const bowCy = by + Math.sin(angle) * 15;
  
  // 弓的弧度（蓄力时变直）
  const bend = (1 - game.power / CFG.MAX_POWER * 0.7) * 20;
  
  ctx.strokeStyle = '#5a3a1a';
  ctx.lineWidth = 3;
  ctx.beginPath();
  const perpAngle = angle + Math.PI / 2;
  ctx.moveTo(bowCx + Math.cos(perpAngle) * bowR, bowCy + Math.sin(perpAngle) * bowR);
  ctx.quadraticCurveTo(
    bowCx - Math.cos(angle) * bend,
    bowCy - Math.sin(angle) * bend,
    bowCx - Math.cos(perpAngle) * bowR,
    bowCy - Math.sin(perpAngle) * bowR
  );
  ctx.stroke();
  
  // 弓弦
  ctx.strokeStyle = 'rgba(255, 245, 225, 0.8)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  const stringPull = game.isCharging ? -game.power * 1.2 : 0;
  ctx.moveTo(
    bowCx + Math.cos(perpAngle) * bowR,
    bowCy + Math.sin(perpAngle) * bowR
  );
  ctx.lineTo(
    bowCx + Math.cos(angle) * stringPull,
    bowCy + Math.sin(angle) * stringPull
  );
  ctx.lineTo(
    bowCx - Math.cos(perpAngle) * bowR,
    bowCy - Math.sin(perpAngle) * bowR
  );
  ctx.stroke();
  
  // 待发的箭（蓄力中）
  if (game.isCharging) {
    const arrowStart = {
      x: bowCx + Math.cos(angle) * stringPull,
      y: bowCy + Math.sin(angle) * stringPull,
    };
    drawArrow(arrowStart.x + Math.cos(angle) * 25, arrowStart.y + Math.sin(angle) * 25, angle);
  }
}
```

让我整合所有代码，写一个完整、可运行的版本。我需要确保：

1. 视觉精美 - 黄昏夕阳风格，氛围感强
2. 物理合理 - 抛物线弹道，蓄力机制清晰
3. 反馈丰富 - 命中粒子、震动、浮动文字、声效
4. 排行榜功能 - localStorage存储
5. 响应式 - 不同屏幕尺寸都能玩

我还要加入一些视觉亮点：
- 飘落的叶子粒子（背景动态）
- 远处飞鸟
- 草地风吹效果
- 太阳光晕
- 命中时的金色火花

让我开始整理最终代码。考虑代码长度，要保持精炼但功能完整。

让我想想还要做什么：
- 开始面板要好看
- 排行榜要好看
- 结果面板要好看
- 整体UI风格统一（古典黄昏风格）

字体：Cinzel 用于标题，JetBrains Mono 用于数字（计时器、分数）

我还要做一个简单的WebAudio音效：
- 蓄力时的弦音
- 射箭时的"嗖"声
- 命中时的"咚"声

这些可以用 oscillator + envelope 生成。

让我现在写最终代码：

```javascript
// 音效
let audioCtx = null;
function ensureAudio() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  return audioCtx;
}

function playSound(type) {
  const ctx = ensureAudio();
  if (ctx.state === 'suspended') ctx.resume();
  
  if (type === 'shoot') {
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = 'sawtooth';
    osc.frequency.setValueAtTime(800, ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(200, ctx.currentTime + 0.15);
    gain.gain.setValueAtTime(0.15, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.15);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + 0.15);
  } else if (type === 'hit') {
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = 'sine';
    osc.frequency.setValueAtTime(150, ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(80, ctx.currentTime + 0.2);
    gain.gain.setValueAtTime(0.3, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.3);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + 0.3);
    
    // 加一个高音"叮"
    const osc2 = ctx.createOscillator();
    const gain2 = ctx.createGain();
    osc2.type = 'triangle';
    osc2.frequency.setValueAtTime(1200, ctx.currentTime);
    gain2.gain.setValueAtTime(0.1, ctx.currentTime);
    gain2.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.15);
    osc2.connect(gain2);
    gain2.connect(ctx.destination);
    osc2.start();
    osc2.stop(ctx.currentTime + 0.15);
  } else if (type === 'win') {
    // 胜利音效
    const notes = [523, 659, 784, 1047];
    notes.forEach((freq, i) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(freq, ctx.currentTime + i * 0.1);
      gain.gain.setValueAtTime(0, ctx.currentTime + i * 0.1);
      gain.gain.linearRampToValueAtTime(0.15, ctx.currentTime + i * 0.1 + 0.02);
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + i * 0.1 + 0.3);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start(ctx.currentTime + i * 0.1);
      osc.stop(ctx.currentTime + i * 0.1 + 0.3);
    });
  }
}
```

好，让我把所有代码整合起来：

实际上让我想想布局。Canvas 1200x600，旁边放排行榜。我想要：
- 主区域：Canvas（左侧/中央，占大部分空间）
- 右侧栏：排行榜 + 控制面板
- 顶部：标题和HUD

让我用CSS Grid来布局。

```css
#app {
  display: grid;
  grid-template-columns: 1fr 320px;
  grid-template-rows: auto 1fr;
  gap: 16px;
  padding: 16px;
  height: 100vh;
  max-width: 1600px;
  margin: 0 auto;
}

header {
  grid-column: 1 / -1;
}

#canvas-wrap {
  position: relative;
  /* ... */
}

#sidebar {
  /* ... */
}
```

让我现在整合并写最终代码。

让我思考一下尺寸：
- Canvas逻辑尺寸：1200 x 600（2:1）
- 16:9屏幕下，1440x900，给sidebar 300px，canvas区1140 x 800
- 那么canvas实际显示 1140 x 570 左右
- 用CSS aspect-ratio保持比例

我会用CSS让canvas宽度100%，高度按比例自动。逻辑像素1200x600固定。

写代码的时候要注意：
- mousemove 事件需要把鼠标坐标转换为canvas逻辑坐标
- 这通过getBoundingClientRect和缩放比例计算

```javascript
function getMousePos(e) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  return {
    x: (e.clientX - rect.left) * scaleX,
    y: (e.clientY - rect.top) * scaleY,
  };
}
```

OK 我现在开始写最终的完整代码。

让我考虑一下细节：

1. **箭的飞行轨迹预览**：在蓄力时显示淡淡的弹道预览线（虚线点），帮助玩家瞄准
   - 但不要太完整，否则失去挑战感
   - 可以显示前1/3的轨迹

2. **风的提示**：可以加一个风向风速指示器
   - 简单的箭头+数值
   - 风会持续影响箭的飞行（小幅水平加速度）
   - 增加策略性

3. **靶子移动**：第4个靶子上下移动

4. **完整UI**：
   - 顶部HUD：时间、命中、蓄力条
   - 右侧：排行榜、玩家名输入、开始按钮
   - 结果面板（覆盖在canvas上）

让我现在开始写。

考虑到游戏复杂性，我可能要让代码精简一点，但保持功能完整。我会专注于：
1. 核心玩法（射箭+物理+命中）
2. 视觉氛围（夕阳场景）
3. 反馈系统（粒子+震动+音效+文字）
4. 排行榜（localStorage）

让我写最终代码。

完整代码框架：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>黄昏弓道 · Twilight Archery</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;800&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
<style>
/* CSS */
</style>
</head>
<body>
<!-- HTML -->
<script>
/* JS */
</script>
</body>
</html>
```

让我写CSS：

```css
:root {
  --bg-deep: #0a0604;
  --bg: #1a0f0a;
  --card: rgba(35, 22, 14, 0.85);
  --card-light: rgba(50, 32, 20, 0.9);
  --border: rgba(245, 166, 35, 0.25);
  --border-strong: rgba(245, 166, 35, 0.5);
  --fg: #fff5e1;
  --muted: #a08868;
  --accent: #f5a623;
  --accent-hot: #e74c3c;
  --gold: #ffd700;
  --green: #4ade80;
}

* { margin:0; padding:0; box-sizing:border-box; }

html, body {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: var(--bg-deep);
  color: var(--fg);
  font-family: 'Cinzel', serif;
}

body {
  background: 
    radial-gradient(ellipse at 20% 30%, rgba(196, 30, 58, 0.15) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 70%, rgba(245, 166, 35, 0.1) 0%, transparent 50%),
    var(--bg-deep);
  display: grid;
  grid-template-columns: 1fr 340px;
  grid-template-rows: auto 1fr;
  gap: 16px;
  padding: 16px;
}

/* Header */
header {
  grid-column: 1 / -1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 8px;
}

header h1 {
  font-size: 32px;
  font-weight: 800;
  letter-spacing: 4px;
  background: linear-gradient(135deg, #f5a623, #e74c3c);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

header .subtitle {
  font-size: 12px;
  color: var(--muted);
  letter-spacing: 2px;
}

/* HUD */
.hud {
  display: flex;
  gap: 24px;
  align-items: center;
}

.hud-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.hud-item .label {
  font-size: 10px;
  color: var(--muted);
  letter-spacing: 2px;
  text-transform: uppercase;
}

.hud-item .value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 24px;
  font-weight: 700;
  color: var(--accent);
}

.power-bar {
  width: 120px;
  height: 8px;
  background: rgba(245, 166, 35, 0.1);
  border: 1px solid var(--border);
  border-radius: 4px;
  overflow: hidden;
}

.power-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #f5a623, #e74c3c);
  width: 0%;
  transition: width 0.05s linear;
}

/* Canvas wrap */
#canvas-wrap {
  position: relative;
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

#game {
  width: 100%;
  height: 100%;
  display: block;
  cursor: crosshair;
}

/* Sidebar */
#sidebar {
  display: flex;
  flex-direction: column;
  gap: 16px;
  overflow-y: auto;
}

.panel {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 20px;
  backdrop-filter: blur(10px);
}

.panel h2 {
  font-size: 16px;
  letter-spacing: 3px;
  margin-bottom: 16px;
  color: var(--accent);
  border-bottom: 1px solid var(--border);
  padding-bottom: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.input-field {
  width: 100%;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 10px 12px;
  color: var(--fg);
  font-family: 'Cinzel', serif;
  font-size: 14px;
  margin-bottom: 12px;
}

.input-field:focus {
  outline: none;
  border-color: var(--accent);
}

.btn {
  width: 100%;
  background: linear-gradient(135deg, var(--accent), var(--accent-hot));
  border: none;
  border-radius: 4px;
  padding: 12px;
  color: var(--bg-deep);
  font-family: 'Cinzel', serif;
  font-weight: 700;
  font-size: 14px;
  letter-spacing: 2px;
  cursor: pointer;
  text-transform: uppercase;
  transition: transform 0.1s, box-shadow 0.2s;
}

.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(245, 166, 35, 0.4);
}

.btn:active {
  transform: translateY(0);
}

.btn.secondary {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--accent);
}

.help-text {
  font-size: 12px;
  color: var(--muted);
  line-height: 1.6;
  margin-top: 12px;
}

.help-text strong {
  color: var(--fg);
}

/* Leaderboard */
#leaderboard ol {
  list-style: none;
  padding: 0;
  margin: 0;
}

.rank-item {
  display: grid;
  grid-template-columns: 24px 1fr auto;
  gap: 12px;
  align-items: center;
  padding: 8px 4px;
  border-bottom: 1px solid rgba(245, 166, 35, 0.08);
  font-size: 13px;
}

.rank-item .num {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 700;
  color: var(--muted);
}

.rank-item.top1 .num { color: var(--gold); }
.rank-item.top2 .num { color: #c0c0c0; }
.rank-item.top3 .num { color: #cd7f32; }

.rank-item .name {
  font-family: 'Cinzel', serif;
  color: var(--fg);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.rank-item .time {
  font-family: 'JetBrains Mono', monospace;
  color: var(--accent);
  font-weight: 700;
}

.empty-state {
  text-align: center;
  padding: 20px;
  color: var(--muted);
  font-size: 13px;
}

/* Result panel */
#result-panel {
  position: absolute;
  inset: 0;
  background: rgba(10, 6, 4, 0.85);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.result-card {
  background: var(--card);
  border: 1px solid var(--border-strong);
  border-radius: 12px;
  padding: 32px 48px;
  text-align: center;
  max-width: 400px;
  animation: slideUp 0.4s ease;
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.result-card h2 {
  font-size: 24px;
  letter-spacing: 4px;
  color: var(--accent);
  margin-bottom: 16px;
}

.result-stat {
  margin: 16px 0;
}

.result-stat .label {
  font-size: 12px;
  color: var(--muted);
  letter-spacing: 2px;
  text-transform: uppercase;
}

.result-stat .value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 36px;
  font-weight: 700;
  color: var(--fg);
}

.result-stat.rank .value {
  background: linear-gradient(135deg, #f5a623, #e74c3c);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

/* Toast */
.toast {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--card-light);
  border: 1px solid var(--border-strong);
  border-radius: 4px;
  padding: 12px 24px;
  color: var(--fg);
  font-size: 13px;
  letter-spacing: 1px;
  z-index: 100;
  animation: toastIn 0.3s ease;
}

@keyframes toastIn {
  from { transform: translate(-50%, 20px); opacity: 0; }
  to { transform: translate(-50%, 0); opacity: 1; }
}

@media (max-width: 900px) {
  body {
    grid-template-columns: 1fr;
  }
  #sidebar {
    display: none;
  }
}
```

让我考虑下排行榜的设计，应该有时间、名字、日期。

```javascript
const STORAGE_KEY = 'twilight_archery_scores';

function loadScores() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
  } catch {
    return [];
  }
}

function saveScore(name, time, arrowsShot) {
  const scores = loadScores();
  scores.push({
    name: name || '匿名射手',
    time: time,
    arrows: arrowsShot,
    date: Date.now(),
  });
  scores.sort((a, b) => a.time - b.time);
  scores.splice(20); // 保留前20名
  localStorage.setItem(STORAGE_KEY, JSON.stringify(scores));
  return scores.indexOf({ name, time, arrows: arrowsShot, date: Date.now() });
}

function getRank(time) {
  const scores = loadScores();
  let rank = 1;
  for (const s of scores) {
    if (s.time < time) rank++;
  }
  return rank;
}
```

让我写完整代码了。我需要确保所有部分都连贯。

让我最后想想视觉效果，让画面更有氛围：
1. 太阳：圆形+光晕
2. 远山：2-3层渐变剪影
3. 鸟群：远处飞过的小剪影
4. 飘落的叶子粒子：缓慢从画面顶部飘下
5. 草地：底部的草纹
6. 弓箭手：剪影
7. 弓：详细绘制
8. 箭：详细绘制
9. 靶子：标准红白蓝黄靶面+木质支架

让我写最终代码：

我意识到我需要更仔细地处理一些细节：

1. **箭的旋转角度**：箭的旋转应该是它速度的方向，所以箭头朝向飞行方向。绘制时，先用translate到，再rotate(a.rot)，再绘制箭。

2. **靶子的命中检测**：判断箭头位置是否在靶子圆内。

3. **蓄力时显示的预览箭**：在弓弦中心位置，朝向瞄准方向，显示一支待发的箭。

4. **箭飞行时的尾迹**：可以加一个淡淡的拖尾，增加视觉效果。

让我最终确定代码结构，开始写：

```javascript
// 主要的物理更新和绘制函数

function shoot() {
  if (game.state !== STATE.PLAYING) return;
  if (game.power < 5) return; // 蓄力太小不射
  
  const angle = Math.atan2(game.mouseY - CFG.BOW.y, game.mouseX - CFG.BOW.x);
  const speed = game.power;
  
  game.arrows.push({
    x: CFG.BOW.x + Math.cos(angle) * 30,
    y: CFG.BOW.y + Math.sin(angle) * 30,
    vx: Math.cos(angle) * speed,
    vy: Math.sin(angle) * speed,
    rot: angle,
    trail: [],
  });
  
  game.arrowsShot++;
  game.power = 0;
  game.isCharging = false;
  playSound('shoot');
}
```

OK 让我把代码整合到一起。考虑到代码量比较大，让我精简一下，但保持视觉效果和功能完整。

实际上让我也加上一些"风"的元素，让游戏更有挑战性。风的强度可以通过UI显示，每局开始时随机生成一个风向风速。

风：
- windX：水平加速度，-0.05 到 0.05 之间
- 显示在HUD中

```javascript
function startGame() {
  game.state = STATE.PLAYING;
  game.startTime = performance.now();
  game.endTime = 0;
  game.hits = 0;
  game.arrowsShot = 0;
  game.arrows = [];
  game.stuckArrows = [];
  game.particles = [];
  game.floatingTexts = [];
  game.power = 0;
  game.isCharging = false;
  game.shake = 0;
  
  // 重置靶子
  game.targets = TARGET_CONFIG.map(t => ({
    ...t,
    hit: false,
    baseY: t.y,
    movePhase: Math.random() * Math.PI * 2,
    hitAnim: 0,
  }));
  
  // 随机风
  game.wind = (Math.random() - 0.5) * 0.08;
}
```

OK，让我开始写最终代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>黄昏弓道 · Twilight Archery</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;800;900&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
<style>
... (CSS)
</style>
</head>
<body>
<header>
  <div>
    <h1>黄昏弓道</h1>
    <p class="subtitle">TWILIGHT ARCHERY · 射穿四靶 · 争分夺秒</p>
  </div>
  <div class="hud">
    <div class="hud-item">
      <span class="label">TIME</span>
      <span class="value" id="timer">0.00</span>
    </div>
    <div class="hud-item">
      <span class="label">HITS</span>
      <span class="value" id="hits">0 / 4</span>
    </div>
    <div class="hud-item">
      <span class="label">WIND</span>
      <span class="value" id="wind">→ 0.0</span>
    </div>
    <div class="hud-item">
      <span class="label">POWER</span>
      <div class="power-bar"><div class="power-bar-fill" id="power-fill"></div></div>
    </div>
  </div>
</header>

<div id="canvas-wrap">
  <canvas id="game"></canvas>
  
  <div id="start-overlay" class="overlay">
    <div class="result-card">
      <h2>准备就绪</h2>
      <p class="desc">按住鼠标蓄力，松开射箭。<br>命中所有四个靶子，时间越短排名越高。</p>
      <input id="player-name" class="input-field" placeholder="输入射手之名" maxlength="16">
      <button id="start-btn" class="btn">开始挑战</button>
    </div>
  </div>
  
  <div id="result-overlay" class="overlay" hidden>
    <div class="result-card">
      <h2 id="result-title">挑战完成</h2>
      <div class="result-stat">
        <div class="label">用时</div>
        <div class="value" id="final-time">0.00s</div>
      </div>
      <div class="result-stat rank">
        <div class="label">排名</div>
        <div class="value" id="final-rank">#1</div>
      </div>
      <div class="result-stat">
        <div class="label">射出箭数</div>
        <div class="value" id="final-arrows">0</div>
      </div>
      <button id="restart-btn" class="btn">再次挑战</button>
    </div>
  </div>
</div>

<aside id="sidebar">
  <div class="panel">
    <h2>操作指南</h2>
    <div class="help-text">
      <p><strong>瞄准</strong> 移动鼠标</p>
      <p><strong>蓄力</strong> 按住鼠标左键</p>
      <p><strong>射箭</strong> 松开鼠标</p>
      <p><strong>规则</strong> 命中所有4个靶子，计时越短越好。最后一靶会移动，且受风力影响。</p>
    </div>
  </div>
  
  <div class="panel">
    <h2>排行榜</h2>
    <ol id="rank-list"></ol>
  </div>
</aside>

<script>
... (JS)
</script>
</body>
</html>
```

让我整理完整代码并写出最终版本。代码会很长，但每部分都有用途。

让我再思考一下一些细节：

1. **Canvas响应式**：canvas元素需要正确的逻辑尺寸。我会用1200x600逻辑尺寸，CSS控制显示尺寸。鼠标事件中转换坐标。

2. **风的影响**：水平加速度，影响箭飞行。

3. **绘制顺序**：天空 → 远山 → 太阳 → 近山 → 地面 → 草 → 飘落叶子 → 靶子 → 插着的箭 → 飞行的箭 → 粒子 → 浮动文字 → 弓箭手 → 瞄准线

4. **叶子粒子**：背景的飘落叶子，从顶部出现，缓慢飘下，左右摆动。

5. **鸟群**：远处地平线上偶尔飞过的鸟群剪影。

让我开始写最终的整合代码。

我觉得我应该再加点细节，让画面更有质感。比如：
- 太阳的光晕（多层径向渐变）
- 远处的山的薄雾
- 弓的木纹（绘制几条线）
- 箭的羽毛（绘制几条短线）
- 靶子的阴影
- 命中后的破碎效果（靶子会有破损的样子）

OK 让我直接写代码。我意识到代码量可能比较多，让我尽量保持精简但功能完整。

让我开始写：

实际上让我先确定一下游戏体验的关键点：
1. 玩家点击"开始挑战"
2. 计时开始，4个靶子出现
3. 玩家用鼠标瞄准
4. 按住鼠标蓄力（看到弓被拉开，能量条增长）
5. 松开鼠标射箭（箭沿抛物线飞行）
6. 命中或落地
7. 重复直到命中所有4个靶子
8. 显示结果，加入排行榜
9. 可以再次挑战

这是一个清晰的游戏循环。

让我开始写最终代码。我要保证：
- 代码结构清晰
- 注释中文
- 视觉效果出色
- 玩法流畅

代码可能比较长，但是必要的。

让我开始：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>黄昏弓道 · Twilight Archery</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;800;900&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
<style>
:root {
  --bg-deep: #0a0604;
  --bg: #1a0f0a;
  --card: rgba(35, 22, 14, 0.85);
  --border: rgba(245, 166, 35, 0.25);
  --border-strong: rgba(245, 166, 35, 0.5);
  --fg: #fff5e1;
  --muted: #a08868;
  --accent: #f5a623;
  --accent-hot: #e74c3c;
  --gold: #ffd700;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

html, body {
  width: 100%;
  height: 100%;
  overflow: hidden;
  font-family: 'Cinzel', serif;
  background: 
    radial-gradient(ellipse 80% 60% at 50% 100%, rgba(196, 30, 58, 0.2) 0%, transparent 60%),
    radial-gradient(ellipse 60% 40% at 80% 20%, rgba(245, 166, 35, 0.12) 0%, transparent 60%),
    var(--bg-deep);
  color: var(--fg);
}

body {
  display: grid;
  grid-template-columns: 1fr 340px;
  grid-template-rows: auto 1fr;
  gap: 16px;
  padding: 16px;
  height: 100vh;
}

header {
  grid-column: 1 / -1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 4px 12px;
}

.brand h1 {
  font-size: 28px;
  font-weight: 800;
  letter-spacing: 6px;
  background: linear-gradient(135deg, #f5a623 0%, #e74c3c 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  line-height: 1;
}

.brand .subtitle {
  font-size: 10px;
  color: var(--muted);
  letter-spacing: 3px;
  margin-top: 4px;
  text-transform: uppercase;
}

.hud {
  display: flex;
  gap: 28px;
  align-items: center;
}

.hud-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 70px;
}

.hud-item .label {
  font-size: 9px;
  color: var(--muted);
  letter-spacing: 2px;
  text-transform: uppercase;
  font-weight: 600;
}

.hud-item .value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 22px;
  font-weight: 700;
  color: var(--fg);
  line-height: 1;
}

.hud-item.timer .value { color: var(--accent); }
.hud-item.wind .value { color: #6dd5ed; font-size: 16px; }

.power-bar {
  width: 140px;
  height: 10px;
  background: rgba(245, 166, 35, 0.08);
  border: 1px solid var(--border);
  border-radius: 2px;
  overflow: hidden;
  position: relative;
}

.power-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #f5a623 0%, #e74c3c 100%);
  width: 0%;
  transition: width 0.05s linear;
  box-shadow: 0 0 8px rgba(245, 166, 35, 0.6);
}

#canvas-wrap {
  position: relative;
  background: #1a0f0a;
  border: 1px solid var(--border);
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(0,0,0,0.5), inset 0 0 60px rgba(0,0,0,0.5);
}

#game {
  width: 100%;
  height: 100%;
  display: block;
  cursor: crosshair;
}

.overlay {
  position: absolute;
  inset: 0;
  background: rgba(10, 6, 4, 0.7);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  animation: fadeIn 0.3s ease;
}

.overlay[hidden] { display: none; }

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.result-card {
  background: var(--card);
  border: 1px solid var(--border-strong);
  border-radius: 8px;
  padding: 36px 48px;
  text-align: center;
  max-width: 420px;
  width: 90%;
  animation: slideUp 0.4s ease;
  box-shadow: 0 20px 60px rgba(0,0,0,0.6), 0 0 80px rgba(245, 166, 35, 0.15);
}

@keyframes slideUp {
  from { transform: translateY(20px) scale(0.96); opacity: 0; }
  to { transform: translateY(0) scale(1); opacity: 1; }
}

.result-card h2 {
  font-size: 28px;
  letter-spacing: 4px;
  color: var(--accent);
  margin-bottom: 24px;
  font-weight: 800;
}

.result-card .desc {
  font-size: 13px;
  color: var(--muted);
  line-height: 1.8;
  margin-bottom: 24px;
  letter-spacing: 1px;
}

.result-stat {
  margin: 20px 0;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.result-stat .label {
  font-size: 10px;
  color: var(--muted);
  letter-spacing: 3px;
  text-transform: uppercase;
}

.result-stat .value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 36px;
  font-weight: 700;
  color: var(--fg);
  line-height: 1;
}

.result-stat.rank .value {
  background: linear-gradient(135deg, #f5a623, #e74c3c);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}

.input-field {
  width: 100%;
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 12px 14px;
  color: var(--fg);
  font-family: 'Cinzel', serif;
  font-size: 14px;
  margin-bottom: 16px;
  letter-spacing: 1px;
  transition: border-color 0.2s;
}

.input-field::placeholder { color: var(--muted); }

.input-field:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(245, 166, 35, 0.15);
}

.btn {
  width: 100%;
  background: linear-gradient(135deg, #f5a623, #e74c3c);
  border: none;
  border-radius: 4px;
  padding: 14px;
  color: #1a0a05;
  font-family: 'Cinzel', serif;
  font-weight: 800;
  font-size: 14px;
  letter-spacing: 3px;
  cursor: pointer;
  text-transform: uppercase;
  transition: transform 0.1s, box-shadow 0.2s;
  position: relative;
  overflow: hidden;
}

.btn::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(120deg, transparent 30%, rgba(255,255,255,0.3) 50%, transparent 70%);
  transform: translateX(-100%);
  transition: transform 0.4s;
}

.btn:hover::before { transform: translateX(100%); }

.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(245, 166, 35, 0.4);
}

.btn:active { transform: translateY(0); }

#sidebar {
  display: flex;
  flex-direction: column;
  gap: 16px;
  overflow-y: auto;
  padding-right: 4px;
}

#sidebar::-webkit-scrollbar { width: 4px; }
#sidebar::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

.panel {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 20px;
  backdrop-filter: blur(10px);
}

.panel h2 {
  font-size: 14px;
  letter-spacing: 3px;
  margin-bottom: 16px;
  color: var(--accent);
  border-bottom: 1px solid var(--border);
  padding-bottom: 10px;
  font-weight: 700;
  text-transform: uppercase;
}

.help-text {
  font-size: 12px;
  color: var(--muted);
  line-height: 1.9;
  letter-spacing: 0.5px;
}

.help-text p { margin-bottom: 4px; }

.help-text strong {
  color: var(--accent);
  font-weight: 700;
  margin-right: 8px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  letter-spacing: 1px;
}

#rank-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.rank-item {
  display: grid;
  grid-template-columns: 28px 1fr auto;
  gap: 12px;
  align-items: center;
  padding: 10px 4px;
  border-bottom: 1px solid rgba(245, 166, 35, 0.06);
  font-size: 13px;
  transition: background 0.2s;
}

.rank-item:hover { background: rgba(245, 166, 35, 0.04); }

.rank-item .num {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 700;
  color: var(--muted);
  text-align: center;
}

.rank-item.top1 .num { color: var(--gold); text-shadow: 0 0 8px rgba(255, 215, 0, 0.6); }
.rank-item.top2 .num { color: #c0c0c0; }
.rank-item.top3 .num { color: #cd7f32; }

.rank-item .name {
  font-family: 'Cinzel', serif;
  color: var(--fg);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: 1px;
}

.rank-item .time-val {
  font-family: 'JetBrains Mono', monospace;
  color: var(--accent);
  font-weight: 700;
  font-size: 12px;
}

.rank-item .arrows-count {
  font-size: 10px;
  color: var(--muted);
  margin-top: 2px;
}

.empty-state {
  text-align: center;
  padding: 24px 8px;
  color: var(--muted);
  font-size: 12px;
  letter-spacing: 1px;
  line-height: 1.8;
}

.toast {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--card);
  border: 1px solid var(--border-strong);
  border-radius: 4px;
  padding: 12px 24px;
  color: var(--fg);
  font-size: 12px;
  letter-spacing: 2px;
  z-index: 100;
  animation: toastIn 0.3s ease;
  pointer-events: none;
}

@keyframes toastIn {
  from { transform: translate(-50%, 20px); opacity: 0; }
  to { transform: translate(-50%, 0); opacity: 1; }
}

@media (max-width: 1024px) {
  body { grid-template-columns: 1fr; padding: 8px; }
  #sidebar { display: none; }
  .hud { gap: 16px; }
  .hud-item { min-width: 60px; }
  .power-bar { width: 100px; }
}
</style>
</head>
<body>
<header>
  <div class="brand">
    <h1>黄昏弓道</h1>
    <div class="subtitle">TWILIGHT ARCHERY · 射穿四靶 · 争分夺秒</div>
  </div>
  <div class="hud">
    <div class="hud-item timer">
      <span class="label">TIME</span>
      <span class="value" id="timer">0.00</span>
    </div>
    <div class="hud-item">
      <span class="label">HITS</span>
      <span class="value" id="hits">0 / 4</span>
    </div>
    <div class="hud-item wind">
      <span class="label">WIND</span>
      <span class="value" id="wind">→ 0.0</span>
    </div>
    <div class="hud-item">
      <span class="label">POWER</span>
      <div class="power-bar"><div class="power-bar-fill" id="power-fill"></div></div>
    </div>
  </div>
</header>

<div id="canvas-wrap">
  <canvas id="game" width="1200" height="600"></canvas>
  
  <div id="start-overlay" class="overlay">
    <div class="result-card">
      <h2>准备就绪</h2>
      <p class="desc">按住鼠标蓄力，松开射箭。<br>命中所有四个靶子，时间越短排名越高。<br>最后一靶会移动，且箭受风力影响。</p>
      <input id="player-name" class="input-field" placeholder="输入射手之名（可选）" maxlength="16" value="">
      <button id="start-btn" class="btn">开始挑战</button>
    </div>
  </div>
  
  <div id="result-overlay" class="overlay" hidden>
    <div class="result-card">
      <h2 id="result-title">挑战完成</h2>
      <div class="result-stat">
        <div class="label">用时</div>
        <div class="value" id="final-time">0.00s</div>
      </div>
      <div class="result-stat rank">
        <div class="label">排名</div>
        <div class="value" id="final-rank">#1</div>
      </div>
      <div class="result-stat">
        <div class="label">射出箭数</div>
        <div class="value" id="final-arrows">0</div>
      </div>
      <button id="restart-btn" class="btn">再次挑战</button>
    </div>
  </div>
</div>

<aside id="sidebar">
  <div class="panel">
    <h2>操作指南</h2>
    <div class="help-text">
      <p><strong>瞄准</strong>移动鼠标，决定射箭方向</p>
      <p><strong>蓄力</strong>按住鼠标左键，蓄力越久力度越大</p>
      <p><strong>射箭</strong>松开鼠标，箭沿抛物线飞行</p>
      <p><strong>目标</strong>命中全部4个靶子，时间越短排名越高</p>
    </div>
  </div>
  
  <div class="panel">
    <h2>排行榜</h2>
    <ol id="rank-list">
      <li class="empty-state">尚无记录<br>成为第一位射手</li>
    </ol>
  </div>
</aside>

<script>
'use strict';

// === 配置 ===
const CFG = {
  W: 1200, H: 600,
  GROUND_Y: 480,
  BOW: { x: 130, y: 380 },
  GRAVITY: 0.42,
  MAX_POWER: 24,
  CHARGE_RATE: 0.42,
  ARROW_LEN: 48,
  TARGET_COUNT: 4,
};

const TARGET_CONFIG = [
  { x: 380, y: 360, r: 40, moveAmp: 0, moveSpd: 0, label: 'I' },
  { x: 620, y: 290, r: 34, moveAmp: 0, moveSpd: 0, label: 'II' },
  { x: 850, y: 410, r: 30, moveAmp: 0, moveSpd: 0, label: 'III' },
  { x: 1060, y: 240, r: 26, moveAmp: 55, moveSpd: 0.022, label: 'IV' },
];

const STATE = { IDLE: 'idle', PLAYING: 'playing', WON: 'won' };

// === DOM ===
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const timerEl = document.getElementById('timer');
const hitsEl = document.getElementById('hits');
const windEl = document.getElementById('wind');
const powerFill = document.getElementById('power-fill');
const startOverlay = document.getElementById('start-overlay');
const resultOverlay = document.getElementById('result-overlay');
const playerNameInput = document.getElementById('player-name');
const startBtn = document.getElementById('start-btn');
const restartBtn = document.getElementById('restart-btn');
const finalTimeEl = document.getElementById('final-time');
const finalRankEl = document.getElementById('final-rank');
const finalArrowsEl = document.getElementById('final-arrows');
const rankList = document.getElementById('rank-list');

// === 游戏状态 ===
let game = {
  state: STATE.IDLE,
  startTime: 0,
  endTime: 0,
  playerName: '匿名射手',
  arrowsShot: 0,
  hits: 0,
  arrows: [],
  stuckArrows: [],
  isCharging: false,
  power: 0,
  mouseX: 600,
  mouseY: 300,
  targets: [],
  particles: [],
  floatingTexts: [],
  leaves: [],
  shake: 0,
  wind: 0,
  frame: 0,
};

// 初始化飘落的叶子
function initLeaves() {
  game.leaves = [];
  for (let i = 0; i < 25; i++) {
    game.leaves.push({
      x: Math.random() * CFG.W,
      y: Math.random() * CFG.GROUND_Y,
      vx: -0.3 - Math.random() * 0.5,
      vy: 0.3 + Math.random() * 0.6,
      rot: Math.random() * Math.PI * 2,
      vrot: (Math.random() - 0.5) * 0.05,
      size: 3 + Math.random() * 4,
      color: Math.random() < 0.5 ? '#c41e3a' : '#f5a623',
      alpha: 0.4 + Math.random() * 0.4,
    });
  }
}

// === 音频 ===
let audioCtx = null;
function ensureAudio() {
  if (!audioCtx) {
    try {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    } catch (e) { return null; }
  }
  return audioCtx;
}

function playSound(type) {
  const ac = ensureAudio();
  if (!ac) return;
  if (ac.state === 'suspended') ac.resume();
  const t = ac.currentTime;
  
  if (type === 'shoot') {
    const osc = ac.createOscillator();
    const gain = ac.createGain();
    osc.type = 'sawtooth';
    osc.frequency.setValueAtTime(900, t);
    osc.frequency.exponentialRampToValueAtTime(180, t + 0.15);
    gain.gain.setValueAtTime(0.15, t);
    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.15);
    osc.connect(gain).connect(ac.destination);
    osc.start(t);
    osc.stop(t + 0.15);
  } else if (type === 'hit') {
    // 低音"咚"
    const o1 = ac.createOscillator();
    const g1 = ac.createGain();
    o1.type = 'sine';
    o1.frequency.setValueAtTime(160, t);
    o1.frequency.exponentialRampToValueAtTime(70, t + 0.25);
    g1.gain.setValueAtTime(0.35, t);
    g1.gain.exponentialRampToValueAtTime(0.001, t + 0.3);
    o1.connect(g1).connect(ac.destination);
    o1.start(t);
    o1.stop(t + 0.3);
    // 高音"叮"
    const o2 = ac.createOscillator();
    const g2 = ac.createGain();
    o2.type = 'triangle';
    o2.frequency.setValueAtTime(1400, t);
    o2.frequency.exponentialRampToValueAtTime(800, t + 0.12);
    g2.gain.setValueAtTime(0.15, t);
    g2.gain.exponentialRampToValueAtTime(0.001, t + 0.15);
    o2.connect(g2).connect(ac.destination);
    o2.start(t);
    o2.stop(t + 0.15);
  } else if (type === 'miss') {
    const osc = ac.createOscillator();
    const gain = ac.createGain();
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(120, t);
    osc.frequency.exponentialRampToValueAtTime(60, t + 0.15);
    gain.gain.setValueAtTime(0.15, t);
    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.2);
    osc.connect(gain).connect(ac.destination);
    osc.start(t);
    osc.stop(t + 0.2);
  } else if (type === 'win') {
    const notes = [523, 659, 784, 1047];
    notes.forEach((freq, i) => {
      const osc = ac.createOscillator();
      const gain = ac.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(freq, t + i * 0.12);
      gain.gain.setValueAtTime(0, t + i * 0.12);
      gain.gain.linearRampToValueAtTime(0.18, t + i * 0.12 + 0.02);
      gain.gain.exponentialRampToValueAtTime(0.001, t + i * 0.12 + 0.4);
      osc.connect(gain).connect(ac.destination);
      osc.start(t + i * 0.12);
      osc.stop(t + i * 0.12 + 0.4);
    });
  }
}

// === 排行榜 ===
const STORAGE_KEY = 'twilight_archery_scores_v1';

function loadScores() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
  } catch { return []; }
}

function saveScore(name, time, arrows) {
  const scores = loadScores();
  const entry = { name, time, arrows, date: Date.now() };
  scores.push(entry);
  scores.sort((a, b) => a.time - b.time);
  scores.splice(20);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(scores));
  return scores.indexOf(entry) + 1;
}

function renderLeaderboard() {
  const scores = loadScores();
  if (scores.length === 0) {
    rankList.innerHTML = '<li class="empty-state">尚无记录<br>成为第一位射手</li>';
    return;
  }
  rankList.innerHTML = scores.map((s, i) => {
    const rank = i + 1;
    const cls = rank <= 3 ? `top${rank}` : '';
    const dateStr = new Date(s.date).toLocaleDateString('zh-CN', { month: '2-digit', day: '2-digit' });
    return `<li class="rank-item ${cls}">
      <span class="num">${rank}</span>
      <span class="name" title="${s.name}">${s.name}<div class="arrows-count">${s.arrows}箭 · ${dateStr}</div></span>
      <span class="time-val">${s.time.toFixed(2)}s</span>
    </li>`;
  }).join('');
}

// === 游戏控制 ===
function startGame() {
  game.playerName = playerNameInput.value.trim() || '匿名射手';
  game.state = STATE.PLAYING;
  game.startTime = performance.now();
  game.endTime = 0;
  game.hits = 0;
  game.arrowsShot = 0;
  game.arrows = [];
  game.stuckArrows = [];
  game.particles = [];
  game.floatingTexts = [];
  game.power = 0;
  game.isCharging = false;
  game.shake = 0;
  game.wind = (Math.random() - 0.5) * 0.08;
  
  game.targets = TARGET_CONFIG.map(t => ({
    ...t,
    hit: false,
    baseY: t.y,
    movePhase: Math.random() * Math.PI * 2,
    hitAnim: 0,
  }));
  
  startOverlay.hidden = true;
  resultOverlay.hidden = true;
  updateWindDisplay();
}

function shoot() {
  if (game.state !== STATE.PLAYING) return;
  if (game.power < 4) return;
  
  const angle = Math.atan2(game.mouseY - CFG.BOW.y, game.mouseX - CFG.BOW.x);
  const speed = game.power;
  
  game.arrows.push({
    x: CFG.BOW.x + Math.cos(angle) * 35,
    y: CFG.BOW.y + Math.sin(angle) * 35,
    vx: Math.cos(angle) * speed,
    vy: Math.sin(angle) * speed,
    rot: angle,
    trail: [],
  });
  
  game.arrowsShot++;
  game.power = 0;
  game.isCharging = false;
  playSound('shoot');
}

// === 更新 ===
function update() {
  game.frame++;
  
  // 计时
  if (game.state === STATE.PLAYING) {
    const t = (performance.now() - game.startTime) / 1000;
    timerEl.textContent = t.toFixed(2);
  }
  
  // 蓄力
  if (game.isCharging && game.power < CFG.MAX_POWER) {
    game.power += CFG.CHARGE_RATE;
    if (game.power > CFG.MAX_POWER) game.power = CFG.MAX_POWER;
  }
  powerFill.style.width = (game.power / CFG.MAX_POWER * 100) + '%';
  
  // 飞行的箭
  for (let i = game.arrows.length - 1; i >= 0; i--) {
    const a = game.arrows[i];
    a.vy += CFG.GRAVITY;
    a.vx += game.wind;
    a.x += a.vx;
    a.y += a.vy;
    a.rot = Math.atan2(a.vy, a.vx);
    
    // 尾迹
    a.trail.push({ x: a.x, y: a.y, life: 20 });
    if (a.trail.length > 20) a.trail.shift();
    for (const t of a.trail) t.life--;
    
    // 命中检测
    let hit = false;
    for (const t of game.targets) {
      if (t.hit) continue;
      const dx = a.x - t.x;
      const dy = a.y - t.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if (dist < t.r) {
        t.hit = true;
        t.hitAnim = 20;
        game.hits++;
        hit = true;
        
        const ring = Math.max(1, 11 - Math.ceil(dist / t.r * 10));
        spawnHitParticles(a.x, a.y, t);
        game.floatingTexts.push({
          x: a.x, y: a.y - 25,
          text: ring >= 10 ? '正中!' : `${ring}环`,
          life: 60, maxLife: 60,
          color: ring >= 10 ? '#ffd700' : '#f5a623',
        });
        game.shake = 14;
        game.stuckArrows.push({
          x: a.x, y: a.y, rot: a.rot,
          relX: dx, relY: dy, target: t, life: 99999, stuck: true,
        });
        game.arrows.splice(i, 1);
        playSound('hit');
        hitsEl.textContent = `${game.hits} / ${CFG.TARGET_COUNT}`;
        
        if (game.hits >= CFG.TARGET_COUNT) {
          game.state = STATE.WON;
          game.endTime = performance.now();
          setTimeout(showResult, 600);
        }
        break;
      }
    }
    if (hit) continue;
    
    // 落地
    if (a.y > CFG.GROUND_Y) {
      game.stuckArrows.push({
        x: a.x, y: CFG.GROUND_Y, rot: a.rot,
        life: 240, stuck: false,
      });
      spawnDustParticles(a.x, CFG.GROUND_Y);
      game.arrows.splice(i, 1);
      playSound('miss');
      continue;
    }
    
    // 出界
    if (a.x > CFG.W + 80 || a.x < -80 || a.y < -200) {
      game.arrows.splice(i, 1);
    }
  }
  
  // 靶子移动 + 命中动画
  for (const t of game.targets) {
    if (t.moveAmp > 0 && !t.hit) {
      t.movePhase += t.moveSpd;
      t.y = t.baseY + Math.sin(t.movePhase) * t.moveAmp;
    }
    if (t.hitAnim > 0) t.hitAnim--;
  }
  
  // 插着的箭（如果靶子移动，箭要跟随）
  for (let i = game.stuckArrows.length - 1; i >= 0; i--) {
    const a = game.stuckArrows[i];
    if (a.stuck && a.target) {
      // 跟随靶子位置
      a.x = a.target.x + a.relX;
      a.y = a.target.y + a.relY;
    } else if (!a.stuck) {
      a.life--;
      if (a.life <= 0) game.stuckArrows.splice(i, 1);
    }
  }
  
  // 粒子
  for (let i = game.particles.length - 1; i >= 0; i--) {
    const p = game.particles[i];
    p.x += p.vx;
    p.y += p.vy;
    p.vy += p.g || 0.15;
    p.vx *= 0.98;
    p.life--;
    if (p.life <= 0) game.particles.splice(i, 1);
  }
  
  // 浮动文字
  for (let i = game.floatingTexts.length - 1; i >= 0; i--) {
    const t = game.floatingTexts[i];
    t.y -= 1.2;
    t.life--;
    if (t.life <= 0) game.floatingTexts.splice(i, 1);
  }
  
  // 飘落的叶子
  for (const l of game.leaves) {
    l.x += l.vx + Math.sin(game.frame * 0.02 + l.rot) * 0.3 + game.wind * 5;
    l.y += l.vy;
    l.rot += l.vrot;
    if (l.y > CFG.GROUND_Y) {
      l.y = -10;
      l.x = Math.random() * CFG.W;
    }
    if (l.x < -20) l.x = CFG.W + 20;
  }
  
  // 震动衰减
  if (game.shake > 0.1) game.shake *= 0.85;
  else game.shake = 0;
}

function spawnHitParticles(x, y, target) {
  for (let i = 0; i < 20; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 5;
    game.particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 1,
      r: 1 + Math.random() * 3,
      life: 30 + Math.random() * 30,
      maxLife: 60,
      color: Math.random() < 0.5 ? '#f5a623' : '#ffd700',
      g: 0.2,
    });
  }
  // 木屑
  for (let i = 0; i < 10; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 1 + Math.random() * 3;
    game.particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 2,
      r: 1 + Math.random() * 2,
      life: 40 + Math.random() * 20,
      maxLife: 60,
      color: '#5a3a1a',
      g: 0.25,
    });
  }
}

function spawnDustParticles(x, y) {
  for (let i = 0; i < 8; i++) {
    const angle = -Math.PI + Math.random() * Math.PI;
    const speed = 1 + Math.random() * 2;
    game.particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 1,
      r: 2 + Math.random() * 3,
      life: 25 + Math.random() * 15,
      maxLife: 40,
      color: '#6a4a2a',
      g: 0.1,
    });
  }
}

function updateWindDisplay() {
  const w = game.wind;
  const arrow = w > 0.01 ? '→' : w < -0.01 ? '←' : '·';
  windEl.textContent = `${arrow} ${Math.abs(w * 100).toFixed(1)}`;
}

// === 绘制 ===
function draw() {
  ctx.save();
  
  // 震动
  if (game.shake > 0.1) {
    ctx.translate((Math.random() - 0.5) * game.shake, (Math.random() - 0.5) * game.shake);
  }
  
  // 清屏
  ctx.fillStyle = '#1a0f0a';
  ctx.fillRect(0, 0, CFG.W, CFG.H);
  
  drawSky();
  drawSun();
  drawMountains();
  drawGround();
  drawLeaves();
  drawTargets();
  drawStuckArrows();
  drawFlyingArrows();
  drawParticles();
  drawArcher();
  drawAim();
  drawFloatingTexts();
  drawWindIndicator();
  
  ctx.restore();
}

function drawSky() {
  const grad = ctx.createLinearGradient(0, 0, 0, CFG.GROUND_Y);
  grad.addColorStop(0, '#1a0f1a');
  grad.addColorStop(0.3, '#4a1a2a');
  grad.addColorStop(0.55, '#9c3a1a');
  grad.addColorStop(0.8, '#d4521a');
  grad.addColorStop(1, '#f5a623');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, CFG.W, CFG.GROUND_Y);
  
  // 云
  ctx.fillStyle = 'rgba(60, 30, 20, 0.5)';
  for (let i = 0; i < 4; i++) {
    const cx = (i * 300 + game.frame * 0.1) % (CFG.W + 200) - 100;
    const cy = 80 + i * 30;
    ctx.beginPath();
    ctx.ellipse(cx, cy, 80, 12, 0, 0, Math.PI * 2);
    ctx.ellipse(cx + 40, cy - 5, 50, 10, 0, 0, Math.PI * 2);
    ctx.fill();
  }
}

function drawSun() {
  const sunX = CFG.W * 0.72;
  const sunY = CFG.GROUND_Y - 25;
  
  // 光晕（多层）
  for (let i = 4; i >= 1; i--) {
    const r = 50 + i * 60;
    const g = ctx.createRadialGradient(sunX, sunY, 0, sunX, sunY, r);
    g.addColorStop(0, `rgba(255, 220, 100, ${0.15 / i})`);
    g.addColorStop(1, 'rgba(255, 220, 100, 0)');
    ctx.fillStyle = g;
    ctx.fillRect(sunX - r, sunY - r, r * 2, r * 2);
  }
  
  // 太阳本体
  const sunGrad = ctx.createRadialGradient(sunX, sunY, 0, sunX, sunY, 50);
  sunGrad.addColorStop(0, '#fff5d0');
  sunGrad.addColorStop(0.5, '#ffd070');
  sunGrad.addColorStop(1, '#f5a623');
  ctx.fillStyle = sunGrad;
  ctx.beginPath();
  ctx.arc(sunX, sunY, 45, 0, Math.PI * 2);
  ctx.fill();
  
  // 水面反光（地面反光）
  ctx.fillStyle = 'rgba(255, 200, 100, 0.1)';
  ctx.fillRect(0, CFG.GROUND_Y, CFG.W, 20);
}

function drawMountains() {
  // 远山
  ctx.fillStyle = 'rgba(50, 25, 35, 0.85)';
  ctx.beginPath();
  ctx.moveTo(0, CFG.GROUND_Y);
  ctx.lineTo(0, 380);
  const farPts = [[0, 380], [80, 350], [180, 370], [260, 340], [350, 365], [450, 350], [560, 375], [680, 355], [800, 370], [920, 360], [1050, 380], [1200, 365]];
  for (const p of farPts) ctx.lineTo(p[0], p[1]);
  ctx.lineTo(CFG.W, CFG.GROUND_Y);
  ctx.closePath();
  ctx.fill();
  
  // 近山
  ctx.fillStyle = '#1a0f12';
  ctx.beginPath();
  ctx.moveTo(0, CFG.GROUND_Y);
  const nearPts = [[0, 440], [120, 425], [220, 445], [350, 430], [480, 460], [620, 435], [780, 460], [920, 440], [1080, 465], [1200, 445]];
  for (const p of nearPts) ctx.lineTo(p[0], p[1]);
  ctx.lineTo(CFG.W, CFG.GROUND_Y);
  ctx.closePath();
  ctx.fill();
  
  // 雾气
  const fog = ctx.createLinearGradient(0, 380, 0, CFG.GROUND_Y);
  fog.addColorStop(0, 'rgba(245, 166, 35, 0)');
  fog.addColorStop(1, 'rgba(245, 166, 35, 0.08)');
  ctx.fillStyle = fog;
  ctx.fillRect(0, 380, CFG.W, CFG.GROUND_Y - 380);
}

function drawGround() {
  const grad = ctx.createLinearGradient(0, CFG.GROUND_Y, 0, CFG.H);
  grad.addColorStop(0, '#3a2418');
  grad.addColorStop(0.5, '#2a1810');
  grad.addColorStop(1, '#150a05');
  ctx.fillStyle = grad;
  ctx.fillRect(0, CFG.GROUND_Y, CFG.W, CFG.H - CFG.GROUND_Y);
  
  // 草纹
  ctx.strokeStyle = 'rgba(245, 166, 35, 0.15)';
  ctx.lineWidth = 1;
  for (let x = 0; x < CFG.W; x += 6) {
    const h = 3 + Math.sin(x * 0.1) * 2 + Math.random() * 3;
    ctx.beginPath();
    ctx.moveTo(x, CFG.GROUND_Y);
    ctx.lineTo(x + 1, CFG.GROUND_Y - h);
    ctx.stroke();
  }
  
  // 远处地面光
  ctx.fillStyle = 'rgba(245, 166, 35, 0.05)';
  ctx.fillRect(0, CFG.GROUND_Y, CFG.W, 10);
}

function drawLeaves() {
  for (const l of game.leaves) {
    ctx.save();
    ctx.translate(l.x, l.y);
    ctx.rotate(l.rot);
    ctx.globalAlpha = l.alpha;
    ctx.fillStyle = l.color;
    ctx.beginPath();
    ctx.ellipse(0, 0, l.size, l.size * 0.5, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();
  }
  ctx.globalAlpha = 1;
}

function drawTargets() {
  for (const t of game.targets) {
    drawTarget(t);
  }
}

function drawTarget(t) {
  // 支架
  ctx.fillStyle = '#3a2418';
  ctx.fillRect(t.x - 4, t.y, 8, CFG.GROUND_Y - t.y);
  // 支架阴影
  ctx.fillStyle = 'rgba(0, 0, 0, 0.3)';
  ctx.fillRect(t.x + 2, t.y, 4, CFG.GROUND_Y - t.y);
  
  // 靶面阴影
  ctx.fillStyle = 'rgba(0, 0, 0, 0.4)';
  ctx.beginPath();
  ctx.arc(t.x + 3, t.y + 3, t.r + 1, 0, Math.PI * 2);
  ctx.fill();
  
  if (t.hit) {
    // 命中后破碎效果
    ctx.globalAlpha = 0.6;
  }
  
  // 颤动
  const shake = t.hitAnim > 0 ? (Math.random() - 0.5) * (t.hitAnim / 20) * 4 : 0;
  const cx = t.x + shake;
  const cy = t.y;
  
  // 靶面环
  const rings = [
    { r: t.r, color: '#f5f5dc' },
    { r: t.r * 0.78, color: '#1a1a1a' },
    { r: t.r * 0.56, color: '#3a8ed6' },
    { r: t.r * 0.34, color: '#e74c3c' },
    { r: t.r * 0.16, color: '#ffd700' },
  ];
  
  for (const ring of rings) {
    ctx.fillStyle = ring.color;
    ctx.beginPath();
    ctx.arc(cx, cy, ring.r, 0, Math.PI * 2);
    ctx.fill();
  }
  
  // 中心点
  ctx.fillStyle = '#fff5e1';
  ctx.beginPath();
  ctx.arc(cx, cy, 2, 0, Math.PI * 2);
  ctx.fill();
  
  // 边框
  ctx.strokeStyle = '#3a2418';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.arc(cx, cy, t.r, 0, Math.PI * 2);
  ctx.stroke();
  
  // 标签
  ctx.fillStyle = 'rgba(255, 245, 225, 0.5)';
  ctx.font = 'bold 11px JetBrains Mono';
  ctx.textAlign = 'center';
  ctx.fillText(t.label, cx, cy + t.r + 18);
  
  ctx.globalAlpha = 1;
}

function drawArcher() {
  const bx = CFG.BOW.x;
  const by = CFG.BOW.y;
  const angle = Math.atan2(game.mouseY - by, game.mouseX - bx);
  
  // 影子
  ctx.fillStyle = 'rgba(0, 0, 0, 0.4)';
  ctx.beginPath();
  ctx.ellipse(bx, CFG.GROUND_Y - 2, 25, 4, 0, 0, Math.PI * 2);
  ctx.fill();
  
  // 身体（剪影）
  ctx.fillStyle = '#0d0605';
  // 腿
  ctx.fillRect(bx - 8, by + 20, 6, 60);
  ctx.fillRect(bx + 2, by + 20, 6, 60);
  // 躯干
  ctx.beginPath();
  ctx.ellipse(bx, by + 15, 14, 28, 0, 0, Math.PI * 2);
  ctx.fill();
  // 头
  ctx.beginPath();
  ctx.arc(bx, by - 15, 11, 0, Math.PI * 2);
  ctx.fill();
  
  // 弓
  const bowR = 38;
  const bowCx = bx + Math.cos(angle) * 18;
  const bowCy = by + Math.sin(angle) * 18;
  const perp = angle + Math.PI / 2;
  
  const bend = 22 - (game.isCharging ? (game.power / CFG.MAX_POWER) * 16 : 0);
  
  // 弓身（木质）
  ctx.strokeStyle = '#5a3a1a';
  ctx.lineWidth = 4;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(bowCx + Math.cos(perp) * bowR, bowCy + Math.sin(perp) * bowR);
  ctx.quadraticCurveTo(
    bowCx - Math.cos(angle) * bend,
    bowCy - Math.sin(angle) * bend,
    bowCx - Math.cos(perp) * bowR,
    bowCy - Math.sin(perp) * bowR
  );
  ctx.stroke();
  
  // 弓身高光
  ctx.strokeStyle = '#8a5a2a';
  ctx.lineWidth = 1.5;
  ctx.beginPath();
  ctx.moveTo(bowCx + Math.cos(perp) * (bowR - 1), bowCy + Math.sin(perp) * (bowR - 1));
  ctx.quadraticCurveTo(
    bowCx - Math.cos(angle) * bend,
    bowCy - Math.sin(angle) * bend,
    bowCx - Math.cos(perp) * (bowR - 1),
    bowCy - Math.sin(perp) * (bowR - 1)
  );
  ctx.stroke();
  
  // 弓弦
  const pull = game.isCharging ? game.power * 1.4 : 0;
  ctx.strokeStyle = 'rgba(255, 245, 225, 0.85)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.moveTo(bowCx + Math.cos(perp) * bowR, bowCy + Math.sin(perp) * bowR);
  ctx.lineTo(bowCx - Math.cos(angle) * pull, bowCy - Math.sin(angle) * pull);
  ctx.lineTo(bowCx - Math.cos(perp) * bowR, bowCy - Math.sin(perp) * bowR);
  ctx.stroke();
  
  // 待发的箭
  if (game.isCharging) {
    const ax = bowCx - Math.cos(angle) * pull;
    const ay = bowCy - Math.sin(angle) * pull;
    drawArrow(ax + Math.cos(angle) * 28, ay + Math.sin(angle) * 28, angle);
  }
}

function drawArrow(x, y, rot) {
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(rot);
  
  // 杆
  ctx.strokeStyle = '#8a5a2a';
  ctx.lineWidth = 2;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(-22, 0);
  ctx.lineTo(18, 0);
  ctx.stroke();
  
  // 箭头
  ctx.fillStyle = '#c0c0c0';
  ctx.beginPath();
  ctx.moveTo(24, 0);
  ctx.lineTo(18, -3);
  ctx.lineTo(18, 3);
  ctx.closePath();
  ctx.fill();
  ctx.strokeStyle = '#fff5e1';
  ctx.lineWidth = 0.5;
  ctx.stroke();
  
  // 羽毛
  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.moveTo(-22, 0);
  ctx.lineTo(-28, -3);
  ctx.lineTo(-22, -1);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = '#f5a623';
  ctx.beginPath();
  ctx.moveTo(-22, 0);
  ctx.lineTo(-28, 3);
  ctx.lineTo(-22, 1);
  ctx.closePath();
  ctx.fill();
  
  ctx.restore();
}

function drawFlyingArrows() {
  for (const a of game.arrows) {
    // 尾迹
    for (let i = 0; i < a.trail.length; i++) {
      const p = a.trail[i];
      const alpha = (p.life / 20) * 0.4;
      ctx.fillStyle = `rgba(245, 166, 35, ${alpha})`;
      ctx.beginPath();
      ctx.arc(p.x, p.y, 1.5 * (i / a.trail.length), 0, Math.PI * 2);
      ctx.fill();
    }
    drawArrow(a.x, a.y, a.rot);
  }
}

function drawStuckArrows() {
  for (const a of game.stuckArrows) {
    if (!a.stuck) {
      const alpha = Math.min(1, a.life / 60);
      ctx.globalAlpha = alpha;
    }
    drawArrow(a.x, a.y, a.rot);
    ctx.globalAlpha = 1;
  }
}

function drawParticles() {
  for (const p of game.particles) {
    const alpha = p.life / p.maxLife;
    ctx.globalAlpha = alpha;
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r * (alpha * 0.5 + 0.5), 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}

function drawAim() {
  if (game.state !== STATE.PLAYING) return;
  
  const bx = CFG.BOW.x;
  const by = CFG.BOW.y;
  const angle = Math.atan2(game.mouseY - by, game.mouseX - bx);
  
  // 弱辅助线（瞄准方向）
  ctx.strokeStyle = 'rgba(245, 166, 35, 0.2)';
  ctx.lineWidth = 1;
  ctx.setLineDash([3, 6]);
  ctx.beginPath();
  ctx.moveTo(bx + Math.cos(angle) * 40, by + Math.sin(angle) * 40);
  ctx.lineTo(bx + Math.cos(angle) * 120, by + Math.sin(angle) * 120);
  ctx.stroke();
  ctx.setLineDash([]);
  
  // 蓄力时显示弹道预览
  if (game.isCharging && game.power > 5) {
    const speed = game.power;
    let px = bx + Math.cos(angle) * 35;
    let py = by + Math.sin(angle) * 35;
    let pvx = Math.cos(angle) * speed;
    let pvy = Math.sin(angle) * speed;
    
    ctx.fillStyle = 'rgba(245, 166, 35, 0.35)';
    for (let step = 0; step < 30; step++) {
      pvy += CFG.GRAVITY;
      pvx += game.wind;
      px += pvx;
      py += pvy;
      if (py > CFG.GROUND_Y || px > CFG.W) break;
      const alpha = (1 - step / 30) * 0.4;
      ctx.fillStyle = `rgba(245, 166, 35, ${alpha})`;
      ctx.beginPath();
      ctx.arc(px, py, 2, 0, Math.PI * 2);
      ctx.fill();
    }
  }
}

function drawFloatingTexts() {
  for (const t of game.floatingTexts) {
    const alpha = t.life / t.maxLife;
    ctx.globalAlpha = alpha;
    ctx.fillStyle = t.color;
    ctx.font = 'bold 22px Cinzel';
    ctx.textAlign = 'center';
    ctx.shadowColor = 'rgba(0, 0, 0, 0.8)';
    ctx.shadowBlur = 4;
    ctx.fillText(t.text, t.x, t.y);
    ctx.shadowBlur = 0;
  }
  ctx.globalAlpha = 1;
}

function drawWindIndicator() {
  if (game.state !== STATE.PLAYING) return;
  // 在画面上方显示风向
  const wx = CFG.W / 2;
  const wy = 40;
  ctx.fillStyle = 'rgba(0, 0, 0, 0.3)';
  ctx.fillRect(wx - 60, wy - 15, 120, 30);
  
  ctx.fillStyle = '#a08868';
  ctx.font = '10px JetBrains Mono';
  ctx.textAlign = 'center';
  ctx.fillText('WIND', wx, wy - 4);
  
  const w = game.wind;
  const arrowLen = Math.min(40, Math.abs(w * 500));
  const dir = w > 0 ? 1 : -1;
  
  ctx.strokeStyle = '#6dd5ed';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(wx - arrowLen * dir / 2, wy + 6);
  ctx.lineTo(wx + arrowLen * dir / 2, wy + 6);
  ctx.stroke();
  
  // 箭头
  ctx.beginPath();
  ctx.moveTo(wx + arrowLen * dir / 2, wy + 6);
  ctx.lineTo(wx + arrowLen * dir / 2 - 6 * dir, wy + 2);
  ctx.lineTo(wx + arrowLen * dir / 2 - 6 * dir, wy + 10);
  ctx.closePath();
  ctx.fillStyle = '#6dd5ed';
  ctx.fill();
}

// === 结果 ===
function showResult() {
  const elapsed = (game.endTime - game.startTime) / 1000;
  const rank = saveScore(game.playerName, elapsed, game.arrowsShot);
  
  finalTimeEl.textContent = elapsed.toFixed(2) + 's';
  finalRankEl.textContent = '#' + rank;
  finalArrowsEl.textContent = game.arrowsShot;
  
  resultOverlay.hidden = false;
  renderLeaderboard();
  playSound('win');
}

// === 输入 ===
function getMousePos(e) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  return {
    x: (e.clientX - rect.left) * scaleX,
    y: (e.clientY - rect.top) * scaleY,
  };
}

canvas.addEventListener('mousemove', (e) => {
  const p = getMousePos(e);
  game.mouseX = p.x;
  game.mouseY = p.y;
});

canvas.addEventListener('mousedown', (e) => {
  if (e.button !== 0) return;
  if (game.state !== STATE.PLAYING) return;
  game.isCharging = true;
  ensureAudio();
});

canvas.addEventListener('mouseup', (e) => {
  if (e.button !== 0) return;
  if (game.state !== STATE.PLAYING) return;
  if (game.isCharging) shoot();
});

canvas.addEventListener('mouseleave', () => {
  // 鼠标离开canvas时不取消蓄力，让玩家可以重新进入
});

startBtn.addEventListener('click', () => {
  startGame();
});

restartBtn.addEventListener('click', () => {
  startOverlay.hidden = false;
  resultOverlay.hidden = true;
  game.state = STATE.IDLE;
});

// 键盘空格也可以射箭
window.addEventListener('keydown', (e) => {
  if (e.code === 'Space') {
    e.preventDefault();
    if (game.state === STATE.PLAYING && !game.isCharging) {
      game.isCharging = true;
    }
  }
});

window.addEventListener('keyup', (e) => {
  if (e.code === 'Space') {
    e.preventDefault();
    if (game.state === STATE.PLAYING && game.isCharging) {
      shoot();
    }
  }
});

// === 主循环 ===
function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

// 初始化
initLeaves();
renderLeaderboard();
loop();
</script>
</body>
</html>
```

让我审视一下我的代码：

1. **靶子位置**：380, 620, 850, 1060 - 分布在画面右侧。1060可能离边缘太近，但canvas宽度1200，所以还有140px空间，OK。

2. **第4个靶子移动**：上下浮动55像素，速度0.022，看起来合理。

3. **物理**：重力0.42，最大速度24。蓄满力后从(130,380)以角度0射出，水平速度24，重力0.42。
   - 飞到x=1060需要 (1060-130)/24 ≈ 39帧
   - 下落距离 = 0.5 * 0.42 * 39² ≈ 320像素
   - 所以y从380降到700，会先落地
   - 需要瞄准上方一些，比如角度-15度
   - 这看起来是合理的，需要预判

4. **碰撞检测**：箭头位置在 a.x, a.y，检测与靶子中心的距离 < 靶子半径。但箭头有24px长（从-22到18+24=42总长，前端在24），所以箭的视觉前端在。但是我代码里 drawArrow 把箭头放在旋转后的+24处...

让我重新检查drawArrow：
```
ctx.translate(x, y);  // (x,y)是箭的"中心"
ctx.rotate(rot);
// 杆: -22 到 18
// 箭头尖端: 24, 0
```

所以箭头尖端在距离中心点 (24, 0) 旋转rot后的位置。但是我碰撞检测用的是a.x, a.y作为箭头位置，这不对！

让我修正：箭头位置应该是 arrow 中的 a.x, a.y 直接表示箭头尖端。然后绘制时，箭头尖端在，杆向后延伸。

修正drawArrow：
```javascript
function drawArrow(tipX, tipY, rot) {
  ctx.save();
  ctx.translate(tipX, tipY);
  ctx.rotate(rot);
  // 现在原点是箭头尖端，向左是杆
  // 杆: -46 到 0
  ctx.strokeStyle = '#8a5a2a';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(0, 0);
  ctx.stroke();
  
  // 箭头
  ctx.fillStyle = '#c0c0c0';
  ctx.beginPath();
  ctx.moveTo(0, 0);
  ctx.lineTo(-6, -3);
  ctx.lineTo(-6, 3);
  ctx.closePath();
  ctx.fill();
  
  // 羽毛
  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(-52, -3);
  ctx.lineTo(-46, -1);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = '#f5a623';
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(-52, 3);
  ctx.lineTo(-46, 1);
  ctx.closePath();
  ctx.fill();
  
  ctx.restore();
}
```

这样箭的 a.x, a.y 直接就是尖端位置，与碰撞检测一致。

然后shoot时，箭的初始位置应该是弓前端：
```
x: CFG.BOW.x + Math.cos(angle) * 35,
y: CFG.BOW.y + Math.sin(angle) * 35,
```
这里35是弓到尖端的距离，OK。

待发的箭：从弓弦中心向前发射方向28像素处的位置作为尖端。
```
const ax = bowCx - Math.cos(angle) * pull;  // 弓弦中心
const ay = bowCy - Math.sin(angle) * pull;
drawArrow(ax + Math.cos(angle) * 28, ay + Math.sin(angle) * 28, angle);
```

但是28加上箭长46，箭的尾端会在弓弦中心后面18像素。让我调整为：箭的尾端在弓弦中心。
```
const tipX = ax + Math.cos(angle) * 46;  // 箭长
const tipY = ay + Math.sin(angle) * 46;
drawArrow(tipX, tipY, angle);
```

OK 这样箭尾在弓弦中心，箭尖向前。

让我也调整一下shoot时的初始位置。箭尖应该在弓弦中心+箭长距离：
```javascript
game.arrows.push({
  x: bowTipX,  // 实际上为了视觉一致，可以从弓中心+一段距离开始
  y: bowTipY,
  ...
});
```

但其实射箭时，箭从弓弦出发，沿瞄准方向。所以起点应该是弓弦中心（即弓中心+反向位移），箭尖在前面一段距离。

让我重新考虑：
- 弓中心：bowCx, bowCy（在弓箭手前方）
- 弓弦中心（拉到最大时）：bowCx - cos(angle)*pull, bowCy - sin(angle)*pull
- 箭从这里出发，向前射出

箭尖位置 = 弓弦中心 + 箭长 * (cos(angle), sin(angle))

但是弓中心是固定位置，pull是蓄力时弓弦后拉的距离。射箭时箭从弓弦位置出发。

实际上简化处理：射箭时箭尖从弓中心位置（bowCx + 箭长*cos, bowCy + 箭长*sin）开始，速度方向是瞄准方向。

让我简化代码：
```javascript
const angle = Math.atan2(game.mouseY - CFG.BOW.y, game.mouseX - CFG.BOW.x);
const startX = CFG.BOW.x + Math.cos(angle) * 18;  // 弓中心
const startY = CFG.BOW.y + Math.sin(angle) * 18;

game.arrows.push({
  x: startX + Math.cos(angle) * 46,  // 箭尖在弓前方46像素（箭长）
  y: startY + Math.sin(angle) * 46,
  vx: Math.cos(angle) * speed,
  vy: Math.sin(angle) * speed,
  ...
});
```

好，让我再审视一遍代码并修正所有问题。

实际上为了让代码更清晰，让我重新组织一下：

让我重新写代码，使其更精炼。我会保持上面的大致结构，但修正箭的位置问题。

另外我发现一个问题：drawArrow的修改后，"待发的箭"显示位置也要调整。

让我考虑：弓箭手在 (CFG.BOW.x, CFG.BOW.y) = (130, 380)
弓中心：bowCx = 130 + cos(angle)*18, bowCy = 380 + sin(angle)*18

蓄力时弓弦中心（被拉到后面）：bowCx - cos(angle)*pull, bowCy - sin(angle)*pull

箭尾在弓弦中心，箭尖在弓弦中心 + cos(angle)*ARROW_LEN, ... + sin(angle)*ARROW_LEN

ARROW_LEN 我之前定义48，让我用46。

让我整合修正：

```javascript
// 在drawArcher中
if (game.isCharging) {
  const pull = game.power * 1.4;
  const stringX = bowCx - Math.cos(angle) * pull;
  const stringY = bowCy - Math.sin(angle) * pull;
  // 箭尖
  const tipX = stringX + Math.cos(angle) * 46;
  const tipY = stringY + Math.sin(angle) * 46;
  drawArrow(tipX, tipY, angle);
}

// shoot
function shoot() {
  // ...
  const angle = Math.atan2(game.mouseY - CFG.BOW.y, game.mouseX - CFG.BOW.x);
  const bowCx = CFG.BOW.x + Math.cos(angle) * 18;
  const bowCy = CFG.BOW.y + Math.sin(angle) * 18;
  const pull = game.power * 1.4;
  const stringX = bowCx - Math.cos(angle) * pull;
  const stringY = bowCy - Math.sin(angle) * pull;
  const tipX = stringX + Math.cos(angle) * 46;
  const tipY = stringY + Math.sin(angle) * 46;
  
  game.arrows.push({
    x: tipX,
    y: tipY,
    vx: Math.cos(angle) * speed,
    vy: Math.sin(angle) * speed,
    rot: angle,
    trail: [],
  });
  // ...
}
```

OK 这样箭的视觉位置和物理位置一致。

让我现在整理最终的代码。我会简化一些地方，保持核心功能。

我还要思考下：
- "再次挑战"按钮，应该重置游戏到idle状态显示开始面板，或者直接重新开始？
- 我觉得应该直接重新开始，玩家名字保留。

```javascript
restartBtn.addEventListener('click', () => {
  startGame();  // 直接重新开始
});
```

这样更顺畅。

让我再确认一下样式，确保好看：
- 整体黄昏色调，暖色
- 琥珀金作为主强调色
- 字体 Cinzel + JetBrains Mono
- 排行榜有金属色（金银铜）的排名标识

让我整理出最终代码并完善：

实际上我还想加几个细节：
1. 太阳的光线（向外辐射的几条线）
2. 远处的飞鸟
3. 命中时的金光爆发

让我也想想响应式：
- 屏幕宽度 < 1024 时，侧边栏隐藏
- 但canvas还是要可玩

这样手机上也能玩，但体验不如桌面。可以接受。

让我写最终的代码。我需要平衡：代码量、功能完整性、视觉效果。

我还想到一点：开始面板应该让玩家可以预览场景，而不是完全遮挡。所以我用半透明背景。开始面板背景已经设置 `rgba(10, 6, 4, 0.7)` 加 `backdrop-filter: blur(8px)`，可以看到模糊的背景，OK。

但是，在idle状态下，游戏循环还是会运行（draw会执行），所以背景是动态的。✓

让我写最终代码并测试一下逻辑。

最后，我还要考虑代码长度。让我尽量精简但保持功能。

OK，让我开始整理最终代码并输出。

让我再检查一下：updateWindDisplay() 在 startGame 中调用，但是在游戏开始前wind是0，应该显示 "· 0.0"。在游戏开始后更新。✓

实际上HUD的wind元素需要在游戏开始前显示"· 0.0"作为初始值。我在HTML里写的 "→ 0.0"，让我改成 "· 0.0"。

让我整体梳理一下代码，写最终版本：

我意识到代码可能比较长，让我精简一下，主要保留：
- 核心物理和碰撞
- 基本绘制（场景、靶子、弓箭手、箭）
- 关键UI（HUD、排行榜、结果面板）
- 反馈（粒子、震动、文字、音效）
- 风的影响

一些可以省略的：
- 鸟群（视觉锦上添花，但代码量大）
- 太阳光线（保持简单）

让我写最终代码。

哦还要考虑：游戏中如果玩家不蓄力直接松开，不应该射箭（power < 4）。这样防止误操作。✓

如果箭射出去但完全没飞到靶子，会落地或飞出。✓

如果玩家想重新开始当前局（中途放弃）？我可以加一个ESC键或者一个"放弃"按钮。简单起见，可以加一个键盘ESC功能。或者在结果面板之前，玩家无法放弃，必须完成。我觉得简单起见就不加放弃功能，玩家可以刷新页面重新开始。

实际上加一个键盘R键重置当前局比较友好。让我加上：

```javascript
window.addEventListener('keydown', (e) => {
  if (e.code === 'KeyR' && game.state === STATE.PLAYING) {
    // 重置当前局
    startGame();
  }
  // ...
});
```

OK 让我整理最终代码并输出。

我还要考虑：弓箭手在画面上看起来什么样？目前是简单的剪影。让我让它更有形态：
- 双腿分开站立
- 一只手握弓，前伸
- 另一只手拉弦

但这会增加绘制复杂度。让我保持简洁，用基本几何形状构成一个"弓箭手"形象：
- 头（圆）
- 躯干（椭圆）
- 两腿（矩形）
- 一只手臂前伸握弓

OK，我觉得我的代码已经差不多够了。让我整理输出。

一个最后的优化：让靶子的标签 "I, II, III, IV" 在标签下方显示更明显，让玩家知道哪个是哪个。

我也考虑让难度递增更明显：
- 靶子1：大、近、低
- 靶子2：中、中、中
- 靶子3：小、远、低（需要远距离精准）
- 靶子4：最小、远、高、移动

OK，最终输出代码。我会让代码尽量精简但功能完整。

我意识到我有些细节没考虑：
1. update()中的"插着的箭"的逻辑：如果靶子移动，箭要跟随。我代码里用relX/relY记录箭相对靶子的位置，每帧更新箭位置为靶子位置+相对位置。但这只在靶子未命中后才有意义？不，命中后靶子还在原位（除非移动靶），但移动靶命中后y会停在命中时的位置。

实际上命中后我让靶子还继续移动？还是停止？让我让命中后靶子停止移动（破碎状态）：

```javascript
for (const t of game.targets) {
  if (t.moveAmp > 0 && !t.hit) {  // 只有未命中且会移动的靶子才动
    t.movePhase += t.moveSpd;
    t.y = t.baseY + Math.sin(t.movePhase) * t.moveAmp;
  }
}
```

但是箭插在靶子上，如果靶子停止，箭就停在原位。如果靶子还动，箭要跟随。我代码里只有 `if (a.stuck && a.target)`，会跟随。如果命中后停止移动，那t.y不变，箭位置也不变。OK ✓

2. 命中破碎效果：t.hit=true后，drawTarget用 globalAlpha=0.6 显示淡化的靶子。但箭还插在上面，这样视觉上是"靶子被命中但还在原位"。可以加一些裂纹效果，但简化处理就用透明度表示。

3. 命中后箭应该不能再命中其他靶子（已经从game.arrows中移除了），✓

OK 让我输出最终代码。让我再次完整审视一下我的代码：

actually 我要更仔细地处理"插着的箭跟随移动靶"的问题。当靶子命中后停止移动，但是relX/relY是命中瞬间的相对位置。如果之后靶子不动，箭位置=靶子位置+relX/relY，是固定的。✓

如果命中前靶子在移动，命中瞬间t.y是当时的y值。relY = a.y - t.y。之后如果停止移动，箭就在t.y + relY的位置。✓

OK，让我整理输出。

实际上，让我重新看一下命中后的逻辑：
```javascript
if (dist < t.r) {
  t.hit = true;
  t.hitAnim = 20;
  game.hits++;
  hit = true;
  // ...
  game.stuckArrows.push({
    x: a.x, y: a.y, rot: a.rot,
    relX: dx, relY: dy, target: t, life: 99999, stuck: true,
  });
  game.arrows.splice(i, 1);
  // ...
}
```

之后在update中：
```javascript
for (let i = game.stuckArrows.length - 1; i >= 0; i--) {
  const a = game.stuckArrows[i];
  if (a.stuck && a.target) {
    a.x = a.target.x + a.relX;
    a.y = a.target.y + a.relY;
  }
  // ...
}
```

所以箭跟随靶子位置。如果靶子停止移动（命中后），箭位置不变。如果靶子继续移动...但是我代码里 `if (t.moveAmp > 0 && !t.hit)` 才移动，所以命中后不移动。✓

OK，逻辑没问题。

让我输出最终代码：

我注意到一个潜在问题：在 `drawTarget` 中我用了 `t.hit` 来降低alpha，但我也想让靶子在被命中后还是显示（带箭插着）。所以alpha=0.6表示稍微淡化。这样OK。

实际上我可以做得更好：命中后靶子保持完整显示（这样能看到插着的箭），只是不再有动效。让alpha=1。或者加一个轻微的"破损"视觉，比如颜色稍微变暗。

让我简化：命中后靶子保持显示，alpha=0.85（稍微变暗），不再颤动。

```javascript
if (t.hit) ctx.globalAlpha = 0.85;
```

OK 让我整理最终代码。

最后，一个细节：游戏结束（WON状态）后，箭还会继续飞？让我看代码：
- 在update中，箭的更新没有判断game.state。所以WON后箭还会继续飞。
- 但是命中最后一个靶子时，箭已经splice掉了。
- 还在飞的箭会继续到落地或出界。
- 实际上WON后玩家不能再射新箭（shoot函数检查state）。✓

但是已射出的箭可能还在飞，会插到地上。这没问题，看起来更真实。

OK 输出最终代码。

让我精简一下drawArrow函数，并确保所有细节都正确：

```javascript
function drawArrow(tipX, tipY, rot) {
  ctx.save();
  ctx.translate(tipX, tipY);
  ctx.rotate(rot);
  
  // 杆
  ctx.strokeStyle = '#9a6a3a';
  ctx.lineWidth = 2;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(-6, 0);
  ctx.stroke();
  
  // 箭头（金属）
  ctx.fillStyle = '#d0d0d0';
  ctx.beginPath();
  ctx.moveTo(0, 0);
  ctx.lineTo(-8, -3.5);
  ctx.lineTo(-6, 0);
  ctx.lineTo(-8, 3.5);
  ctx.closePath();
  ctx.fill();
  ctx.strokeStyle = '#fff5e1';
  ctx.lineWidth = 0.5;
  ctx.stroke();
  
  // 羽毛上
  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(-52, -4);
  ctx.lineTo(-44, -1);
  ctx.closePath();
  ctx.fill();
  
  // 羽毛下
  ctx.fillStyle = '#f5a623';
  ctx.beginPath();
  ctx.moveTo(-46, 0);
  ctx.lineTo(-52, 4);
  ctx.lineTo(-44, 1);
  ctx.closePath();
  ctx.fill();
  
  ctx.restore();
}
```

这样箭看起来更精致。

OK 让我最终输出代码。

让我再考虑一下界面的标题。"黄昏弓道"听起来不错。英文副标题"TWILIGHT ARCHERY"。

测试说明：
1. 打开HTML文件即可运行
2. 输入名字（可选），点击"开始挑战"
3. 移动鼠标瞄准
4. 按住鼠标左键蓄力（看到弓被拉开，能量条增长，弹道预览显示）
5. 松开鼠标射箭
6. 命中所有4个靶子后显示结果
7. 查看排行榜

测试用例：
- 基础玩法：射中所有4个靶子，验证计时和排名
- 排行榜：多次游戏后查看排序
- 风的影响：注意第4个移动靶和风的协同效果
- 蓄力机制：测试不同蓄力时间的飞行距离
- 物理：箭的抛物线轨迹

OK，让我写最终输出。

实际上我还想到一点：我应该让玩家可以提前看到场景（在idle状态时），让开始面板有视觉吸引力。这已经实现了（overlay半透明）。

最后，关于 Tailwind CSS，我之前提到了 Tailwind CDN，但实际上我没用Tailwind类，而是用了纯CSS。这样更轻量，也避免依赖。让我不用Tailwind了。

好，输出最终代码。

让我再次审视代码长度。我估计HTML+CSS+JS 大概会在500-700行。这个长度对一个完整的游戏来说是合理的。

让我开始写最终输出。

最后再考虑一个细节：箭飞出后，弓的蓄力状态应该清零，弓恢复原状。✓ (game.power = 0; game.isCharging = false;)

还有一个：玩家在蓄力中可以移动鼠标改变角度。这样玩家可以蓄力时调整瞄准。✓ (mousemove 一直更新 game.mouseX, mouseY)

OK 输出。

我决定再让游戏更有质感，加几个细节：
1. 太阳周围有几条光线
2. 远处地平线上有飞鸟（剪影）
3. 命中靶子时有金光爆裂效果

这些不会让代码量增加太多。

实际上飞鸟我之前没加，让我加上：

```javascript
// 飞鸟
const birds = [];
for (let i = 0; i < 3; i++) {
  birds.push({
    x: Math.random() * CFG.W,
    y: 100 + Math.random() * 100,
    spd: 0.3 + Math.random() * 0.3,
    wingPhase: 0,
    size: 3 + Math.random() * 2,
  });
}

function updateBirds() {
  for (const b of birds) {
    b.x += b.spd;
    b.wingPhase += 0.15;
    if (b.x > CFG.W + 20) b.x = -20;
  }
}

function drawBirds() {
  ctx.fillStyle = 'rgba(40, 20, 15, 0.7)';
  for (const b of birds) {
    const wing = Math.sin(b.wingPhase) * b.size * 0.5;
    ctx.beginPath();
    ctx.moveTo(b.x - b.size, b.y);
    ctx.quadraticCurveTo(b.x - b.size * 0.3, b.y - wing - b.size * 0.3, b.x, b.y);
    ctx.quadraticCurveTo(b.x + b.size * 0.3, b.y - wing - b.size * 0.3, b.x + b.size, b.y);
    ctx.quadraticCurveTo(b.x + b.size * 0.3, b.y + wing * 0.5, b.x, b.y);
    ctx.quadraticCurveTo(b.x - b.size * 0.3, b.y + wing * 0.5, b.x - b.size, b.y);
    ctx.fill();
  }
}
```

OK 加上这些。

让我把所有部分整合在一起，输出最终代码：

我需要确保 `game` 对象初始化前不会在`update()`中被访问。看代码，`initLeaves()` 在 `loop()` 前调用，初始化 `game.leaves`。OK。

`birds` 我作为全局变量。让我也放进 `game` 对象，或者作为独立的全局。我作为独立全局更简单。

实际上让我也放 `birds` 在 `game` 里：
```javascript
game = {
  ...
  birds: [],
};

function initBirds() {
  game.birds = [];
  for (let i = 0; i < 3; i++) {
    game.birds.push({...});
  }
}
```

OK 整合到一起。

好，最终代码我会一次性输出。让我先在心里梳理一下完整结构：

HTML:
- header (brand + hud)
- canvas-wrap (canvas + overlays)
- sidebar (help + leaderboard)

CSS:
- 变量定义
- 全局样式
- header 样式
- hud 样式
- canvas-wrap 样式
- overlay + result-card 样式
- input/button 样式
- sidebar + panel 样式
- rank-item 样式
- toast 样式
- 响应式

JS:
- 配置常量 (CFG, TARGET_CONFIG, STATE)
- DOM 引用
- game 状态对象
- 初始化函数 (initLeaves, initBirds)
- 音频系统
- 排行榜系统
- 游戏控制
- 输入处理
- update 函数 (含物理、粒子、文字等)
- draw 函数 (含场景、靶子、弓箭手、箭等)
- 主循环

好，让我开始最终输出。

我想我会输出一个大概700行的HTML文件。代码量合理。

实际上让我再想一下，整体视觉的氛围。黄昏的射箭场：
- 暖色调（橙、红、金）
- 太阳低悬地平线
- 远山剪影
- 飘落的叶子（秋天的感觉）
- 弓箭手剪影

这个氛围很有古典美，符合"黄昏弓道"的主题。

OK 输出最终代码。