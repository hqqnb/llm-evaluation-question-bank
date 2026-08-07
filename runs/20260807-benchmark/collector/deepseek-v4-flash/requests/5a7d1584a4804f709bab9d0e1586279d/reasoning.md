We need answer in Chinese likely. Need create complete game. Need not view other files. Need produce runnable artifact. We need decide tech. "不要查看其他文件，直接创建" means build from scratch. We need provide game. We have browser? Since as API, can deliver HTML file with embedded JS? Need "制作一个弓箭射击模拟游戏。游戏中应有四个靶子。玩家命中所有目标所用的时间越短，在排行榜上的名次就越高。不要查看其他文件，直接创建。 输出要求：请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。"

We need create self-contained HTML game. Need include four targets, archery shooting simulation, timer, leaderboard sorted by time. Need maybe canvas, mouse aiming, arrow trajectory, physics? Need "shooting simulation" can be simple with bow aiming. Need make runnable. Need provide run instructions and tests.

Need think details. Need implement within HTML/CSS/JS. Need ensure playable. Need four targets maybe at different distances. Player clicks/drag? Need "弓箭射击模拟游戏" likely 2D side view: bow at left, targets at right, player sets angle/power or aims with mouse. Need time starts when first shot? Need hit all four targets. Leaderboard stored in localStorage. Rank by time. Need perhaps include "pull bow" mechanics: drag from bow to adjust power/angle, release to shoot arrow. Targets at varying positions. Need realistic arrow trajectory with gravity, wind? We can implement simple projectile.

Need consider UI: Show bow at left, four targets at different distances/heights. Need targets with rings. Player controls by moving mouse? A common archery game: bow follows mouse aim, click to shoot. But "simulation" maybe good: hold mouse to draw, release to shoot. Need ensure all targets can be hit. Need maybe target at vertical/horiz distances.

Need create robust game:

Canvas dimensions e.g. 1000x600. Ground. Bow positioned at (120, 500) maybe. Four targets (circular target with concentric rings) at positions:
- T1: x=300, centerY=400? Maybe near.
- T2: x=500, y=350
- T3: x=700, y=420
- T4: x=880, y=260? Hmm need arrow can reach. Need physics with adjustable velocity.

Need define target radius maybe 40 (scoring rings), center x,y. Need hit if arrow within target radius. Need target countdown; timer starts when "begin" or first arrow? Need all targets. Need perhaps target "hit" changes/removes or turns green.

Need game states: "ready", "playing", "finished". Button start/restart.

Need leaderboard: table with time. Store entries in localStorage key "archery_records". On finish, prompt name? Could auto use "玩家" or input. Need "名次" ranking. Could show name and time. Need allow input name after finish. After hitting all targets, show elapsed time; prompt? Better modal with input for name and save. But tests maybe require no prompt? We can implement input in finished panel. Need "排行榜上的名次越高" with shorter time. Need display sorted ascending time.

Need design.

Need "不要查看其他文件" so create one HTML file with all CSS/JS. Need provide run instructions.

Potential issues:
- Need not use external libs.
- Need arrow physics. Use projectile from bow center. Need account for bow draw "power" maybe arrow initial speed = fraction of max speed based on drag distance. But if player aims mouse, can set angle. Need decide:
  - Bow at left, arrow starts at bow tip. When mouse moves, show dotted trajectory? To help aim.
  - Hold mouse button: draw? But target can be click. Simpler: move mouse to aim, click to shoot. But arrow simulation: compute velocity from mouse direction and fixed speed? Need "simulation" could be less interactive. Better to include "bow draw" mechanic to make it feel like archery.
  
Alternative: Drag from bow to set angle/power, release to shoot. While dragging, "bow string" pulled backward and arrow trajectory preview. On release, arrow launched with velocity vector from bow tip toward release point? Let's think.

We can implement aiming:
- Bow fixed at left (120, 520). Mouse position determines aim point. Arrow velocity = normalized direction from bow to mouse * power. Power = clamp(distance / 200, 0.3, 1.0) * initialSpeed. But if click at mouse, distance small => low power. To shoot far, need click far from bow? Hmm.

Better use "dragging to draw": Bow at left, player presses mouse somewhere and drags back/away to set power and angle? Release to shoot. But more cumbersome.

Need targets at different positions; need player can adjust both angle and power. Mouse aim with fixed max speed can always hit? Let's compute if arrow speed is fixed and gravity; target positions may or may not be reachable. Need allow variable speed. We can use power slider or drag distance.

Maybe use "drag from bow": Bow at (120, 500), arrow nocked. Mouse down and drag sets vector from bow to mouse? Actually in games, to shoot arrow, drag from bow backwards to aim/charge, release. But we can make simpler: move mouse to target, click. The arrow is shot in direction of mouse with speed depending on distance from bow to mouse, scaled. If user wants high power, move mouse far away from bow in desired direction. The arrow direction is from bow to mouse. Since all targets to right, player can click far right to shoot with high power. But if click exact target, direction line passes through target. To hit target, you can set mouse directly on target and distance determines speed; due to gravity, arrow will drop, so perhaps velocity vector direction should include upward compensation. Need trajectory preview can help.

Maybe use "hold to set velocity vector": On mouse down at current mouse pos, start drag; velocity vector is from bow to current mouse, arrow follows exact line? But if no gravity, too trivial. Need gravity, so on release maybe aim above target.

Need target distances; we can implement "drag and release" to set speed/angle: Pull back from target? Another approach:
- Player sees arrow at bow, moving mouse controls a small "瞄准器" (crosshair). Click and drag from bow to set velocity vector. Use "initial velocity = (mouse - bow) * k" and arrow has gravity. Need display projected trajectory so players can calibrate. We can show trajectory preview as dashed line using current velocity. Then player clicks to shoot velocity vector. This is good.

Need ensure all targets can be hit with velocity vector from bow in visible canvas. We can choose k and max velocity such that targets within range. Need trajectory preview can show path. But if click shoot immediately, maybe no hold. Let's plan:

- Bow position B = (140, 520) perhaps. Arrow launch position B + maybe 10.
- Mouse position M. Velocity vector V = (M - B) * powerFactor. Direction from bow to mouse. Since mouse is right of bow, V_x positive. If mouse high, V_y negative. Need magnitude maybe scaled by distance: speed = distance * factor (e.g., 0.18). If M=800,500, distance=660, speed=119 px/s? Too low. Need use units per second. If canvas coordinate px, gravity maybe 300 px/s^2; initial speed 600-1200. distance from bow to mouse maybe 800; factor 1.2 => 960. Okay. So V = (M - B) * 1.2? That means if mouse at (800,300), dx=680, dy=-220, V=(816,-264), speed=857. Gravity 300. Good.

Need define max speed to keep within canvas. Need maybe clamp vector magnitude between min and max. Since M in canvas, max distance from bow to bottom-right corner ~1000, factor 1.0 gives speed 1000. Good. Use factor 1.0, gravity=300. At speed 600, 45° range = v^2/g=1200 px. Enough. But targets maybe at x 300-900, y 200-500. Need account. We can use factor 1.0. But if M near bow, speed low. Need maybe min speed 200. We can define:
- dx = mouse.x - bow.x, dy = mouse.y - bow.y
- vector = (dx, dy). But if mouse is left/behind, not allowed? We need require dx > 0? Could allow full 360, but bow faces right. Let's constrain to dx > 0 maybe. If dx <= 0, vector = (1,0).
- Speed factor = 1.0. But if mouse at (500,300) => dx=360, dy=-220, speed=422, maybe too low. Range ~594? okay. But target far 800 high might need >800. To hit T4 at (850,250), vector from bow to target dx=710, dy=-270, speed=760 with angle -20.8°. Without gravity, reaches 0.93s, drop 0.5*g*t^2=130, so hit low. Need aim high or use speed >. Good. If click just above target, V direction upward, speed maybe 760; might hit. So okay.

Need perhaps allow "click to shoot" but we should have a "draw and release" mechanic? For output requirement, no strict. We can state "移动鼠标瞄准，点击发射". But "弓箭射击" maybe enough. Need include bow drawing animation? We can show arrow at bow and bowstring.

Need render:
- background: sky gradient, grass. Targets on stands.
- Bow: arc and string, arrow when idle? We'll create drawn function drawBow. Bow at left, vertical curved bow. If aim, arrow is along vector with angle? Need draw bow as simple arc centered left; arrow nocked on string? Could be okay.

Need target drawing:
- target = circle with rings colors: white, black, blue, red, yellow center. Need positions. With rings. Hit if arrow tip (or arrow point) distance to target center < target.radius maybe. Need "hit all targets" not necessarily score rings. Maybe show ring score? But only all targets matter. Could compute score but not needed. Need perhaps target disappears/becomes "hit" state and shows check. Timer stops when all hit.

Need arrow simulation:
- An arrow object {x,y,vx,vy,angle,active,start?}.
- On shoot, create arrow at bow.x, bow.y? Need initial position maybe bow.x + 10, bow.y - 5? If using vector = (mouse-bow)*factor, if start at bow, travel.
- Each frame: update x += vx*dt; y += vy*dt; vy += gravity*dt; angle = atan2(vy, vx) maybe point arrow along velocity. Use rotate.
- Check collisions with targets each frame or after update. Need check if arrow enters target. If arrow is within target.radius and target not hit, mark hit. For multiple targets, maybe all.
- If arrow goes out of bounds or hits ground? Need it can stick in ground. For game, arrow disappears after hitting ground? But if it hits target high then falls; we can continue. Need stop when y > groundY or x > canvas width or x < 0. Remove arrow and allow shoot again. Could show arrow stuck on ground but not necessary.

Need "max arrows"? No, unlimited. Need timer starts on first shot? Could start when user clicks start. "玩家命中所有目标所用的时间越短" Usually time from start round to final hit. We can have "开始游戏" button. Timer starts when beginning after first shot? If button starts, time includes aiming. Better: on first click "开始" resets and timer starts. We can have overlay with "开始游戏"; after click timer starts. Need if arrow shot only, okay. Timer starts at first arrow shot maybe more fair. But simpler: click start, timer counts. We can implement start button restart, timer counts while playing. Need in output mention timer starts after clicking start. Need on finish show time.

Need prevent shooting after finish? After all hit, state finished, timer stops, show results and leaderboard.

Need leaderboard:
- localStorage array of {name, time, date}
- Add new entry after finish; prompt name. But if no prompt, can default "无名射手". Need to rank by time ascending. To be "排行榜上的名次越高" we show entries sorted. Need maybe add current result to leaderboard automatically after entering name. We can have a modal with input and "保存成绩". Use HTML details maybe.

Need if file opened via file:// localStorage may work in modern browsers; but if sandbox? Should work. Need "不可查看其他文件" no.

Need design and code.

Need consider "simulate" and "targets" with varied positions. Need maybe four target "has 靶子" at different distances. Need ensure all can be hit given mechanics. Need test by computing trajectories. Let's verify with code maybe not actually running, but reason.

Setup:
Canvas W=1100, H=650. Ground y=580. Bow at (130, 520). Target positions:
- T1: x=320, y=460 (near, slightly high)
- T2: x=520, y=380 (mid, high)
- T3: x=740, y=470 (far low)
- T4: x=920, y=300 (far high)
Need shots.

Question: with vector factor 1.0 and gravity 300, if mouse at (200,300) => dx=70, dy=-220, speed=231, angle -72°. It shoots upward left? Actually dx positive 70, dy -220. It will go left little high, range 178; okay. But target T4 far high maybe need high speed, distance from bow to target 801. To hit, need aim maybe high and speed around 800. If mouse at (900, 180), dx=770, dy=-340 => speed=842, angle -23.8°. Drop for 0.95 sec (to x=860) = 135; so arrow drops 135 from straight line; if initial direction at target? Starting from bow to point above target maybe. Let's solve for hitting T4 maybe.

Target center T=(920,300). Launch from B=(130,520). Need find V from mouse with factor 1.0? Actually V = (M-B) *1.0, so launch angle and speed depend on mouse M. We need choose M such that trajectory hits T. Let's derive. Let P = (u, v) = M - B. V = (u,v). Arrow position at time t: x = 130 + u t, y = 520 + v t + 0.5*300*t^2. Need x=920 => u t =790. y=300 => v t +150 t^2 = -220.

Let s = u t = horizontal displacement. For T4 s=790. Need v t = -220 -150 t^2. Ratio v/u = (-220 -150 t^2)/790. Also t = 790/u. Could solve for u maybe. We can pick t. Need V magnitude = sqrt(u²+v²). t = s/u. With u around 700-800, t~1.0-1.1. v negative. Example u=750, t=1.0533, v = (-220 - 150*1.109)/790? Wait v = (-220 -150 t²)/t? Let's solve:
v t = -220 -150 t². v = (-220 -150t²)/t. If t=1.05, -220 -165.4 = -385.4, v=-367.0. So V=(750,-367), M=B+V=(880,153). If using V=(M-B), mouse should be at (880,153). Is that on canvas? yes. So T4 can be hit by aiming at (880,153) (above-left of target). Good. Need trajectory preview to guide.

T2 at (520,380), B=(130,520), s=390, target dy=-140. Need choose t maybe 0.7? v t = -140 -150 t². t=0.7 => -140 -73.5=-213.5, v=-305, u=557, M=(687,215). good. T1 at (320,460), s=190, dy=-60. t=0.45? v = (-60 -30.4)/0.45=-200, u=422, M=(552,320). okay. T3 at (740,470), s=610, dy=-50. t=0.95? v=(-50-135.4)/0.95=-195, u=642, M=(772,325). okay. So feasible.

Need maybe show trajectory preview to help. We can simulate current aiming vector trajectory with small dashes. Need if mouse position gives target path. Could compute preview points for a few seconds until ground. Use same vector. Then player can adjust. Need "current aim" from mouse to bow. But if player moves mouse away, speed changes and trajectory. Need maybe show "拉力/速度" label. Need maybe not too cluttered.

Need maybe allow "arrows remaining" unlimited. Need "hit all targets" no score. Need if arrow hits a target, target changes color and "hit" count increments. Need if arrow hits same target multiple times no double count. Need if arrow passes through target? We check distance; if within radius, hit. But if target is "hit", arrow passes through? Could still show arrow stuck? We can ignore. Need if arrow hits already hit target, no count but target remains maybe overlay. Perhaps if target hit, mark with green X/check and maybe no longer counts. Arrow might stick in target. Need draw arrows stuck? Could keep arrow in array and freeze at hit position. But if target already hit, arrow should stick. This adds nice effect. Need implement:
- arrows array active arrows and stuck arrows. When arrow hits any target (distance <= radius) at point, set arrow.stuck = true, store x/y, angle, parent target? It doesn't need to collide further. If target not hit, mark target hit. If target already hit, still stick but no count.
- Also if arrow hits ground, set stuck true at ground (y=groundY), no count. Draw arrow on ground.
- If arrow goes out of canvas, remove.
- If arrow crosses target, with dt large, maybe miss; use substeps. Use dt capped at 0.02, check collision along segment. Better implement segment-target intersection? We can check distance from segment to target center. With substeps 4 per frame. Need robust. We'll use time step fixed: 60fps with dt = min((now-last)/1000, 0.033). Update with substeps = 4 for collision.

Need arrow drawing:
- draw arrow as line from tail to head maybe length 30. Since arrow angle from velocity. Draw line tail at (x - 20*cos(angle), y - 20*sin(angle)) to head at (x + 10*cos(angle), y + 10*sin(angle)); Add nock/fletch? Use simple:
   - Save, translate(x,y), rotate(angle).
   - line from (-24,0) to (8,0) brown; head triangle from (8,-3) to (18,0) to (8,3); fletch red at -24.
Need include arrow shadow? maybe not.

Need target drawing:
Function drawTarget(t):
  - if t.hit: maybe draw green ring around target and check.
  - rings radius: outer = 45, white outer, then black 36, blue 27, red 18, yellow 9? Actually scoring target: white outer, black, blue, red, yellow. Need target face maybe mounted on stand.
  - Draw shadow / stand: line from target center downward to ground? Since target y is center; target stand beneath. For each target, draw a wooden pole from t.y + t.radius? to ground? But target is floating? Need target center y maybe at target center; if target radius 45, top/bottom. The stand under target from y+40 to groundY with width. Draw pole.
  - Draw target face: circles from outer radius downward? Need colors:
      outer white radius 48; black radius 38; blue radius 28; red radius 18; yellow radius 8. Center small maybe.
  - Add rings with stroke.
  - If hit, draw "✔" and maybe target changes to green? Need maybe target hit should be visually obvious. We'll draw a translucent green overlay circle or check.

Need "four targets" distinct positions. Need maybe label T1-T4.

Need game flow:
State:
- gameState = 'idle' (start screen), 'playing', 'finished'
- startTime, elapsed.
- hitTargets Set.
- arrowCount maybe.
- records loaded.

HTML layout:
- Title "🏹 弓箭射击模拟游戏"
- Canvas with mouse events.
- Panel: Instructions, Info (timer, targets hit, arrows).
- Leaderboard table.
- Overlay buttons/modal.

Need start screen overlay over canvas? Could have button in HTML above canvas. Let's make simple:
- Container centered.
- Header.
- .game-card containing canvas.
- .info-bar: Timer, Hits, Arrows, Status
- .controls: buttons "开始游戏", "重置"
- .leaderboard: table with rank, name, time, date.
- .modal hidden for saving score after game.

Need "开始游戏" behavior:
```
function startGame() {
  resetGame();
  state='playing';
  startTime=performance.now();
  updateInfo();
}
```
Reset: arrows=[], hitTargets.clear(), timer display 0.00, elapsed=0, etc. Draw. Maybe remove modal.

Need if gameState finished? Could have "再来一局" button. Start button always resets.

Need after final target:
```
function allHit() {
  state='finished';
  elapsed = (performance.now() - startTime)/1000;
  showResult(elapsed);
}
```
Need show modal with input. If user doesn't save, leaderboard not updated. But maybe requirement "排行榜" means need save automatically. To avoid dependence on user, can after finish automatically save with default name "匿名射手" and also let user rename and update? Better:
- On finish, show modal "成绩：xx秒，请输入昵称" with input and "保存成绩" button. Leaderboard only updates after click. But if user can ignore, rank not updated. The game still leaderboard with previous records. Requirement likely okay.
- Could also auto save if no input? Maybe in testing, user can click save. But to ensure leaderboard gets result, we can auto-saving with default name and allow editing? Let's consider.

Need maybe "排行榜上的名次越高" means a leaderboard. We can use "ranking" computed after entry. Need save button. Let's implement modal with input:
  - On finish, state paused, show modal with "命中所有目标！用时 X 秒", input default "玩家".
  - Click "保存成绩" -> saveRecord(name, elapsed) -> hide modal -> update leaderboard -> show finish overlay "排名: X".
  - Click "再玩一次" -> startGame.
Could also "保存并再来" button. Keep simple.

Need if user closes modal without saving? Maybe no save. Need but "交付" not test all.

Need "不要查看其他文件，直接创建" not relevant.

Need "运行方式和必要的测试说明" in final. Need include "将代码保存为 index.html，双击打开或用浏览器打开". Need testing instructions: "点击开始游戏，拖动鼠标移动瞄准，观察虚线弹道，点击发射；依次命中四靶；结束后输入昵称查看排名；刷新后排行榜仍在（localStorage）". But if we implement click to shoot, no drag. Need decide.

Need perhaps "按住鼠标拖拽拉弓，松开射击" could be more fun. Need implement robustly. Let's think if click-to-shoot is less simulation. Could implement both: move mouse to aim; press and hold to draw (the longer/harder?), release to shoot. But not necessarily. Maybe "点击发射" is okay. The output can state "移动鼠标瞄准，点击射击". The title simulation.

Need maybe target positions not all visible? We can make target positions. Need "four targets" maybe can be placed near/far; we can define target radius 48. Need ensure hit detection at center. The user might need to aim using trajectory. Need perhaps no "wind".

Need perhaps "排行榜上的名次越高" with time ascending. We'll store rank after adding sorted. Need display "第 N 名" maybe.

Need maybe "名次" if multiple entries with same time, use insertion order. Sort by time ascending, then date ascending. Need rank.

Need need create complete code. Let's draft code carefully.

Potential challenge: mouse coordinate and vector factor. Need define BOW_POS = {x:130, y:520}. But if canvas scaled by CSS? Need use canvas.getBoundingClientRect() to compute mouse coords. Need if canvas width=1100, height=650.

Need preview trajectory:
- current aim vector = target from mouse to bow? Let's define crosshair? We can set aim= from bow to mouse. But if mouse cursor at (140, 400), vector (10,-120), speed 120, too low. Preview shows arch small. Need player can move mouse far away. The cursor itself maybe on canvas; if cursor exactly at bow, no move. Good.
- Need draw a "瞄准点" at mouse position and maybe "速度条". The trajectory preview should start from bow and follow vector. Need if vector length gets super high, speed high; trajectory maybe goes off screen. Fine.

But issue: if player positions mouse near target and clicks, vector direction from bow to target is down/right, speed relatively low, gravity drops arrow, so it will hit below target. They need click high/left of target. Trajectory preview will show where arrow goes. Need maybe show endpoint at ground or target? We can stop preview when hitting target? We can draw 30 points along path. Need account with gravity:
```
function getTrajectory(vx, vy, steps=60, dt=1/30) {
  points = [];
  x=BOW.x+10; y=BOW.y-10; // arrow start
  for i=0..steps:
    points.push({x,y});
    x += vx*dt; y += vy*dt; vy += GRAVITY*dt;
    if (y > groundY || x > canvas.width || x < 0) break;
  return points;
}
```
Draw dashed line.

Need maybe arrow launch point should be bow.x+? For target line, start from bow. We can define LAUNCH = {x: BOW.x + 8, y: BOW.y - 8}? But if aim vector from BOW to mouse, arrow starts at BOW not exactly. Need for visual bow arrow starts at bow. Use SHOT_ORIGIN = {x: BOW.x + 5, y: BOW.y - 5}. But trajectory calculations should use same. If V = (mouse - BOW) * factor, and trajectory starts at SHOT_ORIGIN, small offset negligible. Maybe bow tip point at (BOW.x+30, BOW.y-20)? Hmm.

Let's define bow center B = {x: 140, y: 520}. The bow arc is vertical; arrow launch origin at tip? In side view, arrow should be at bow center horizontally. Let's use BOW = {x:120, y:510}; SHOT_ORIGIN = {x: BOW.x + 20, y: BOW.y}. If V=(mouse - BOW)*1.0, start at SHOT_ORIGIN. Need if mouse near sign. Good.

Maybe draw bow as vertical bow at x=120. Arrow drawn from nock at x=120 to tip? Simpler.

Need compute V:
```
const dx = mouse.x - BOW.x;
const dy = mouse.y - BOW.y;
let vx = dx;
let vy = dy;
const speed = Math.hypot(vx,vy);
const maxSpeed = 1600;
const minSpeed = 120;
if (speed < minSpeed) { scale = minSpeed/speed; vx*=scale; vy*=scale; }
if (speed > maxSpeed) { scale=maxSpeed/speed; vx*=scale; vy*=scale; }
if (vx < 30 && Math.abs(dy) < ?? )? 
```
Should allow upward/left? If mouse is left of bow, vx negative. That would shoot left. Bow should face right maybe. We can if vx < 20, set vx = 20? Better not; but if click left maybe weird. Could clamp vx to min 40. But if mouse above/left with strong upward, arrow could go left and maybe come back due to gravity? Not useful. Constrain vx >= 0? Let's do: if (vx < 0) vx = 0; if (vx < 40) vx = 40? Actually if mouse is above but far left, vx negative; no need. We can restrict valid aim to right of bow: if (dx < 0) { vx=0; vy = dy; } But not big. Better: if dx < 20, set dx=20? But if mouse directly above bow, arrow will shoot nearly vertical with vx=0 and fall on bow. Not good. Let's set min vx = 60 by scaling? But then direction changes. Need maintain vector. Could define if dx <= 30, use dx=30; dy unchanged. This shifts vertical. But not too bad. Since all targets to right, player naturally aims right. For click near bow, speed low.
Need maybe not use factor 1.0? We can set:
```
const aim = { x: mouse.x - BOW.x, y: mouse.y - BOW.y };
let magnitude = Math.hypot(aim.x, aim.y);
if (magnitude < 30) { aim.x=30; aim.y=-50; }
const speed = Math.max(120, Math.min(1500, magnitude));
const angle = Math.atan2(aim.y, aim.x);
vx = speed * Math.cos(angle);
vy = speed * Math.sin(angle);
```
This means if distance 30, speed=120 (not magnitude? Actually if magnitude 30 but min speed 120 => scale 4). If magnitude 200, speed=200. If magnitude 900, speed=900. If magnitude >1500 impossible because canvas diagonal? max from bow to bottom-right ~1037, so under. Good. Need if aiming left, angle >90, vx negative; we can if cos<0, set angle=Math.atan2(aim.y, Math.max(aim.x, 30))? Easier:
```
if (aim.x < 30) aim.x = 30;
speed = clamp(hypot(aim), 120, 1500);
vx = speed * aim.x / hypot(aim);
vy = speed * aim.y / hypot(aim);
```
But if aim.x clamped, direction changes. Fine.

Need maybe set speed factor? Actually if speed = distance, speed range 120..1200. Good. We can use no factor. Gravity=300. Need check if speed = distance from bow to mouse. For T4, required V=(750,-367), speed=835. Mouse at B+V=(870,143). Distance from B to mouse=835. So speed=835. Good. For T1 required speed maybe 467? But if mouse at (552,320), distance from B (120,510): dx=432,dy=-190, speed=472. Required V from above u=422,v=-200 speed=467. Good. So speed=distance works. Great.

Need draw "current speed" maybe `Math.round(speed)` in UI.

Need "click to shoot" maybe pass current target vector. If mouse not over canvas? Use last known. Need `mousePos` starts at {x:500,y:300}. On mousemove update. On click shoot using mousePos. Need if click Start button, canvas click not triggered? We'll attach canvas click. Need if click modal? okay.

Need perhaps "right click" no.

Need "draw bow and arrow before shooting": draw bow; if no active arrows? We can draw nocked arrow pointing current aim? But if arrow already in flight, still draw bow without arrow. Need maybe draw a "ghost arrow" at bow? But if no arrow, okay. During play, a new arrow is always nocked after previous arrow disappears? Since arrows unlimited but only one in flight? We can allow multiple arrows on screen? Could shoot rapidly. But if arrow doesn't disappear until landing, could have many. Need maybe not constrain. But for gameplay, can have multiple arrows. However if user clicks many times rapidly, many arrows. It's okay. Need maybe set "arrows" count increments each click. Should we limit? No. Need "remaining arrows" no.

Need maybe arrow cooldown? To avoid accidental, maybe no. But in simulation, arrow can be shot every click. We can define `canShoot = true` but no cooldown; if click while active arrow still in flight, shoot another. That's okay.

Need "timer" starts at clicking start; but if start and not shoot, time. Need perhaps "time from first shot" more fair. We can have timer start on first shot? Let's think. If player needs time to aim, timer should include aiming? Usually speedrun starts when round starts and player can begin, includes aiming. Start button starts timer; reset. We'll do that. But if user waits after start, time counts. That's okay.

Need update timer every frame:
```
if (state==='playing') {
  elapsed = (performance.now() - startTime)/1000;
  timerDisplay.textContent = elapsed.toFixed(2);
}
```
Need final elapsed.

Need save record:
```
function saveRecord(name, time) {
  let records = JSON.parse(localStorage.getItem('archery_records')||'[]');
  records.push({name: name || '无名射手', time: Math.round(time*100)/100, date: new Date().toLocaleString()});
  records.sort((a,b)=>a.time-b.time);
  records = records.slice(0, 20); // top 20
  localStorage.setItem('archery_records', JSON.stringify(records));
  renderLeaderboard();
}
```
Need if localStorage unavailable? try/catch.

Need rank:
```
function getRank(time) {
  let records = loadRecords();
  // rank among including current if not saved? Could after save records contains current; before save, count how many records <= time + 1.
}
```
Could use Array findIndex after push.

Need render leaderboard:
- rows. Rank icon maybe 1 🥇, 2 🥈, 3 🥉.

Need modal:
HTML:
```
<div id="modal" class="modal hidden">
  <div class="modal-content">
    <h2>🎯 命中所有目标！</h2>
    <p>用时：<span id="finalTime"></span> 秒</p>
    <input id="playerName" maxlength="12" placeholder="输入昵称">
    <button id="saveBtn">保存成绩</button>
    <button id="againBtn2">再来一局</button>
  </div>
</div>
```
Need "保存成绩" triggers save and maybe show rank text; hide modal? But if hide modal, user can see leaderboard. Good. After save, maybe button text changes. We can:
```
saveBtn.onclick = () => {
  const name = nameInput.value.trim() || "匿名射手";
  saveRecord(name, finalTimeValue);
  showRank = true;
  modal.querySelector('.rank-hint').textContent = `当前排名：${rank}`;
  saveBtn.disabled = true;
  // maybe update info
}
```
Need `againBtn2` resets.
Need if playing, modal hidden. Need if user clicks "保存成绩" multiple times, prevent duplicate. Set saveBtn.disabled = true after save. But if startGame resets modal state.

Need maybe "tabindex" no.

Need "测试说明": Could include "可打开浏览器控制台，检查无报错；连点开始后，移动鼠标到目标上方，点击发射，四靶会依次变绿；命中全部后弹出成绩；刷新页面排行榜保留。" Good.

Need maybe "不要查看其他文件" from prompt: "do not view other files, directly create." We will not.

Need maybe "交付可运行产物" as code file. Since we can't attach file, provide code block. Need "直接完成任务并交付" means maybe include code in final. We should include entire HTML code in one block. Need mention "复制保存为 index.html". Could also include "无需依赖". Good.

Need verify code syntax.

Let's design code in detail.

HTML skeleton:
```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>弓箭射击模拟游戏</title>
<style>
...
</style>
</head>
<body>
<div class="container">
  <h1>🏹 弓箭射击模拟游戏</h1>
  <p class="subtitle">移动鼠标瞄准，点击发射；命中全部 4 个靶子，用时越短排名越高！</p>
  <div class="game-area">
    <canvas id="gameCanvas" width="1100" height="650"></canvas>
    <div id="overlay" class="overlay hidden">
      <h2>🏹 准备开始？</h2>
      <button id="startBtn" class="btn">开始游戏</button>
    </div>
  </div>
  <div class="info-bar">
    <div>⏱ 用时：<span id="timerDisplay">0.00</span>s</div>
    <div>🎯 命中：<span id="hitCount">0</span>/4</div>
    <div>🏹 已射：<span id="arrowCount">0</span> 箭</div>
    <div id="speedDisplay">速度：0</div>
    <button id="resetBtn" class="btn small">重置</button>
  </div>
  <div class="leaderboard">
    <h2>🏆 排行榜</h2>
    <div id="leaderboardBody"><table>...</table></div>
  </div>
</div>
<div id="modal" class="modal hidden">...</div>
<script>
... 
</script>
</body>
</html>
```
Need overlay start button maybe over canvas. If overlay visible, canvas clicks? We'll hide overlay when start. If no overlay, start button in info? We can have start button overlay and reset button. Simpler:
- Use overlay on canvas with start button and instructions. When click start, overlay hidden. Reset button resets and shows overlay? Maybe not. We can have startBtn in overlay. resetBtn in info. If reset clicked, state idle and overlay appears. But if user wants start, click overlay start. Good.
- `overlay` positioned absolute over canvas center, contains start button. Need game-area position relative.

Need if not started, canvas maybe draws targets and bow. Good.
- overlay maybe with translucent background. Hidden class removed.

Need reset:
```
function resetGame() {
  state='idle';
  arrows=[];
  hitTargets.clear();
  arrowCount=0;
  elapsed=0;
  timerDisplay.textContent='0.00';
  hitCountDisplay.textContent='0';
  arrowCountDisplay.textContent='0';
  hideModal();
  overlay.classList.remove('hidden');
  updateLeaderboard? not needed
  draw();
}
```
Need if click reset while playing, state idle and overlay show; no save. Good. Need ensure "startGame" removes overlay and starts timer. If reset button clicked after finish, modal? If modal open, hide. Reset button in modal does same.

Need if modal open and reset clicked? Not accessible due modal maybe. okay.

Need `startGame`:
```
function startGame() {
  resetGameState();
  overlay.classList.add('hidden');
  hideModal();
  state='playing';
  startTime = performance.now();
  lastTime = performance.now();
  requestAnimationFrame(loop);
}
```
But resetGame sets state idle and overlay hidden? Need avoid double. We can implement:
```
function resetGame({showOverlay=true} = {}) { ... if(showOverlay) overlay.classList.remove('hidden'); }
function startGame() { resetGame(false); state='playing'; startTime=performance.now(); lastTime=performance.now(); }
```
Need if resetGame(false) no overlay. Good.

Need draw loop:
- `requestAnimationFrame(loop)` always running after start? Could start on load too. We can have `function loop(now) { ... update(); draw(); requestAnimationFrame(loop); }` always running. Simpler. Then no need manage. Even on idle, draws. On load, requestAnimationFrame(loop). `lastTime` updated. If state playing, update timer and arrows. Good.
- Need `draw` uses current mouse. If mouse not set, default maybe.
- Need if state idle, draw no active arrows? reset arrows. good.

Need arrow update:
```
function update(dt) {
  if (state==='playing') {
    elapsed = (performance.now()-startTime)/1000;
    timerDisplay.textContent = elapsed.toFixed(2);
  }
  updateArrows(dt);
}
```
Need `dt` clamp. `lastTime` from RAF. If first frame, dt maybe 0.
```
function loop(now) {
  const dt = Math.min((now - lastTime)/1000, 0.05);
  lastTime = now;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}
```

Need `updateArrows`:
```
function updateArrows(dt) {
  for (let i=arrows.length-1; i>=0; i--) {
    const a = arrows[i];
    if (a.stuck) continue;
    const steps = 6;
    const sub = dt/steps;
    for (let step=0; step<steps; step++) {
      if (a.stuck) break;
      a.x += a.vx * sub;
      a.y += a.vy * sub;
      a.vy += GRAVITY * sub;
      a.angle = Math.atan2(a.vy, a.vx);
      // collision targets
      for (const t of targets) {
        if (a.stuck) break;
        const dx = a.x - t.x;
        const dy = a.y - t.y;
        if (Math.hypot(dx,dy) <= t.radius) {
           a.stuck = true;
           // clamp to surface? not necessary
           if (!t.hit) {
              t.hit = true;
              hitTargets.add(t.id);
              hitCountDisplay.textContent = hitTargets.size;
              if (hitTargets.size === targets.length) finishGame();
           }
        }
      }
      // ground/out
      if (a.y >= groundY) { a.stuck=true; a.y=groundY; a.angle = Math.atan2(a.vy,a.vx); }
      if (a.x < -50 || a.x > canvas.width + 50 || a.y < -50) { arrows.splice(i,1); break; }
    }
  }
}
```
Need after finishing, maybe arrow stuck at target but no more.

Need if arrow stuck at target, we don't remove. If arrow stuck on ground, draw. Need if arrow hit target and target already hit, arrow stuck at center, okay. If target hit, target remains with arrow.

Need "hitTargets" Set maybe use target object. Use `hitTargets = new Set()`.

Need finish:
```
function finishGame() {
  state = 'finished';
  elapsed = (performance.now() - startTime)/1000;
  timerDisplay.textContent = elapsed.toFixed(2);
  finalTimeValue = elapsed;
  document.getElementById('finalTime').textContent = elapsed.toFixed(2);
  // rank preview
  const rank = computeRank(elapsed);
  document.getElementById('rankPreview').textContent = `当前成绩可按第 ${rank} 名保存`;
  nameInput.value = '射手' + (loadRecords().length + 1)? maybe player? 
  saveBtn.disabled = false;
  modal.classList.remove('hidden');
}
```
Need "rankPreview" in modal. If no records, rank 1.

Need `computeRank(time)`:
```
function computeRank(time) {
 const records=loadRecords();
 let rank=1;
 for (const r of records) { if (r.time < time) rank++; else break; } 
 // Actually rank = count(records.time < time) + 1; ties same? if same time and existing earlier, rank maybe count < time +1, tie same rank? Leaderboards often rank with ties. But final stored with same time inserted after; if same as previous, rank after save could be later. Simpler use sorted all records including current: after save rank index+1. Before save preview can compute `records.filter(r => r.time <= time).length + 1`? Hmm.
 return rank;
}
```
Let's implement after save:
```
function saveRecord() {
 const name = ...;
 const records = loadRecords();
 records.push({name, time: Math.round(finalTimeValue*100)/100, date: new Date().toLocaleString()});
 records.sort((a,b)=>a.time-b.time || a.date.localeCompare(b.date));
 const idx = records.findIndex(r => r.name===name && r.time===Math.round(finalTimeValue*100)/100); // need unique? Could duplicate; find index of last inserted? 
 localStorage.setItem('archery_records', JSON.stringify(records.slice(0,20)));
 saveBtn.disabled = true;
 rank = idx+1; // if sliced after, okay
 rankPreview.textContent = `已保存，当前排名：第 ${rank} 名`;
 renderLeaderboard();
}
```
But findIndex may find first record with same name/time. Use date? Better after push, `idx = records.indexOf(record)`? Since we have object reference but after sort same object. We can do:
```
const record = {name, time, date};
records.push(record);
records.sort(...);
const rank = records.indexOf(record) + 1;
```
Works. Need if records sorted and sliced before rank? If slice, record may not be included if >20. But records max? Could be >20. If current not in top20, rank >20 but not in table. Need still display rank. We can rank before slice limit all records. Or keep max 50. Use `const allRecords = records`; sort; rank index; if rank <= 20 visible. Save slice(0,20) to storage. If rank >20, not visible. okay.
Need if `saveBtn.disabled = true` after save; if start new game, enable. `resetGameState` set `saveBtn.disabled=false`.

Need "排行榜" table:
```
function renderLeaderboard() {
 const records=loadRecords();
 const tbody=...
 if (records.length===0) { tbody.innerHTML='<tr><td colspan="4" class="empty">暂无记录，快来创造记录吧！</td></tr>'; return; }
 records.slice(0,10).forEach((r,i)=> { ... });
}
```
Need include top 10 maybe. Need "名次越高" perhaps show top20. Use top 15.
Need rank icon:
```
const icons = ['🥇','🥈','🥉'];
const rankText = i<3 ? icons[i] : `#${i+1}`;
```
Need time format `r.time.toFixed(2)`. Need date optional.

Need `loadRecords` try/catch:
```
function loadRecords() {
  try { return JSON.parse(localStorage.getItem('archery_records')) || []; }
  catch(e) { return []; }
}
```
Need `saveStorage`.

Need target positions:
```
const targets = [
 { id:1, x:330, y:470, radius:50, hit:false },
 { id:2, x:540, y:390, radius:50, hit:false },
 { id:3, x:760, y:460, radius:50, hit:false },
 { id:4, x:940, y:310, radius:50, hit:false },
];
```
Need maybe y and groundY. groundY=580, so target 1 center 470 radius 50 => bottom 520, pole to ground 60. Target 2 center 390 => bottom 440, pole length 140. target 4 center 310 => bottom 360, pole length 220. Good. Draw poles.

Need target x and radius. Maybe T4 too close to edge? Canvas width 1100, x=940, radius50, okay. Need hit with arrow from B=(120,510) if mouse at (850,150) etc. Good.

Need "random? no".

Need "hit all targets" target count display.

Need "drawTarget":
```
function drawTarget(t) {
  const ctx=...
  // pole
  ctx.strokeStyle = '#7a5a3a';
  ctx.lineWidth = 8;
  ctx.beginPath();
  ctx.moveTo(t.x, t.y + t.radius - 4);
  ctx.lineTo(t.x, groundY);
  ctx.stroke();
  // base
  ctx.fillStyle = '#5a3a1a';
  ctx.fillRect(t.x-30, groundY-6, 60, 10);
  // target face
  const cx=t.x, cy=t.y, r=t.radius;
  // shadow?
  ctx.beginPath(); ctx.arc(cx+3, cy+3, r, 0, Math.PI*2); ctx.fillStyle='rgba(0,0,0,0.2)'; ctx.fill();
  // outer white
  ctx.beginPath(); ctx.arc(cx,cy,r,0,2PI); ctx.fillStyle='#f5f0e8'; ctx.fill(); ctx.strokeStyle='#999'; ctx.lineWidth=2; ctx.stroke();
  ctx.beginPath(); ctx.arc(cx,cy,r*0.78,0,2PI); ctx.fillStyle='#333'; ctx.fill();
  ctx.beginPath(); ctx.arc(cx,cy,r*0.56,0,2PI); ctx.fillStyle='#1e6fd9'; ctx.fill();
  ctx.beginPath(); ctx.arc(cx,cy,r*0.36,0,2PI); ctx.fillStyle='#e03a3a'; ctx.fill();
  ctx.beginPath(); ctx.arc(cx,cy,r*0.18,0,2PI); ctx.fillStyle='#f7d51e'; ctx.fill();
  // ring strokes
  for radii [r, .78r, .56r, .36r, .18r] stroke
  // label above
  ctx.fillStyle='#fff'; ctx.font='bold 14px sans-serif'; ctx.textAlign='center'; ctx.fillText('T'+t.id, cx, cy-70);
  // if hit overlay
  if (t.hit) {
    ctx.beginPath(); ctx.arc(cx,cy,r+4,0,2PI); ctx.strokeStyle='#2ecc71'; ctx.lineWidth=6; ctx.stroke();
    ctx.fillStyle='rgba(46,204,113,0.25)'; ctx.fill()? Need not fill over target? Maybe fill with green translucent. Use ctx.fill after arc.
    ctx.font='bold 48px sans-serif'; ctx.fillStyle='#2ecc71'; ctx.fillText('✓', cx-10, cy+15);
  }
}
```
Need if hit fill green overlay then check. Fine.

Need drawGround:
- gradient sky, clouds maybe. Draw grass:
```
ctx.fillStyle = '#3a7d2c'; ctx.fillRect(0, groundY, W, H-groundY);
ctx.fillStyle='#57a54e'; maybe top grass.
```
Need maybe clouds simple circles. Good.

Need drawBow:
- draw bow at BOW_POS. Bow is arc with curve. Need coordinates:
```
const bx=BOW.x, by=BOW.y;
ctx.strokeStyle='#8B5A2B'; ctx.lineWidth=8;
ctx.beginPath();
ctx.arc(bx - 20, by, 80, -Math.PI/2.5, Math.PI/2.5); // maybe half circle to right?
```
Wait bow should curve to right? In side view, hunter bow is vertical curved; string left. Let's draw:
  - Bow arc: `ctx.arc(bx + 40, by, 90, -Math.PI*0.7, -Math.PI*0.3)`? Hmm.
Need simple. Maybe implement bow as a vertical arc:
```
ctx.beginPath();
ctx.arc(bx - 10, by, 70, -Math.PI/2 - 0.9, -Math.PI/2 + 0.9);
```
This is an arc around center (bx-10, by) from angle -2.47 to -0.67, i.e. open to right? Let's think canvas angles: 0 right, PI/2 down, -PI/2 up. The arc from -PI/2 -0.9 to -PI/2 +0.9 covers left/down/up? It starts upper-left, goes through left? Actually angle -2.47 = -141° (left-up), -0.67=-38° (right-up). It sweeps through -PI/2 (-90°, up). So arc is mostly up, open at top and bottom? Hmm. We can draw bow as a vertical curve with center at (bx, by), facing right:
```
ctx.beginPath();
ctx.arc(bx, by, 70, -Math.PI/2 - 0.7, -Math.PI/2 + 0.7, false);
```
Angle -PI/2-0.7 (=-160°) is up-left; angle -PI/2+0.7 (=-20°) is up-right. That's an arc across top? Actually it's a small arc near top of circle, not vertical. Need not be perfect. Maybe use `ctx.quadraticCurveTo` to draw bow shape:
- Bow start at top (bx, by-80), curve to right at middle (bx+35, by), then to bottom (bx, by+80). Use quadratic curves:
```
ctx.beginPath();
ctx.moveTo(bx, by-80);
ctx.quadraticCurveTo(bx+45, by-10, bx+45, by); // maybe
ctx.quadraticCurveTo(bx+45, by+10, bx, by+80);
ctx.stroke();
```
This is a bow curving right. Need string line from top to bottom:
```
ctx.beginPath();
ctx.moveTo(bx, by-80);
ctx.lineTo(bx-5, by); // bowstring center maybe
ctx.lineTo(bx, by+80);
```
But if arrow nocked, maybe string at center. We'll draw string as line top to bottom with slight bend? Simple line from top to bottom or via nock at (bx-15,by). Actually bowstring should be straight between tips; the arrow rests at string and bow. We can draw string line from top to bottom at x=bx? When idle/no arrow, string straight vertical. During aim, we can show pulled? But no drag. We'll just draw string straight. Need arrow at bow if not shooting? Since multiple arrows, no persistent. Maybe draw "loaded arrow" line from bow to mouse? But draw actual arrow at bow? Could be nice: if state playing, draw a ghost arrow at bow pointing to current aim. But when arrow is in flight, you still have infinite arrows; a loaded arrow is always available. Let's draw a loaded arrow at bow on top of bow:
```
drawNockedArrow(mousePos)
```
Use bow string center to current aim direction. But if there are active arrows, would duplicate. It represents next arrow; okay.

Draw bow:
```
function drawBow() {
  const bx=BOW.x, by=BOW.y;
  ctx.strokeStyle='#8b5a2b';
  ctx.lineWidth=7; ctx.lineCap='round';
  ctx.beginPath(); ctx.moveTo(bx, by-75); ctx.quadraticCurveTo(bx+45, by-5, bx+45, by+5); ctx.quadraticCurveTo(bx+45, by+5, bx, by+75); ctx.stroke();
  // string
  ctx.strokeStyle='#ddd'; ctx.lineWidth=2;
  ctx.beginPath(); ctx.moveTo(bx, by-75); ctx.lineTo(bx-15, by); ctx.lineTo(bx, by+75); ctx.stroke();
}
```
Maybe bow curve:
```
ctx.beginPath();
ctx.moveTo(bx, by-80);
ctx.quadraticCurveTo(bx+50, by-20, bx+45, by);
ctx.quadraticCurveTo(bx+50, by+20, bx, by+80);
ctx.stroke();
```
Need not exact.

Need drawNocked arrow:
```
function drawAimedArrow() {
  if (state==='idle') maybe still? Use current aim vector.
  if (state==='finished') maybe no.
  const vx,vy from getAimVector(mousePos);
  const angle = Math.atan2(vy,vx);
  ctx.save();
  ctx.translate(BOW.x, BOW.y);
  ctx.rotate(angle);
  // draw arrow body, but since bow string etc no matter.
  ctx.restore();
}
```
Need avoid drawing if state idle? Could draw when playing or idle. Fine.

Need `getAimVector(mouse)`:
```
function getAimVector() {
  let dx = mouse.x - BOW.x;
  let dy = mouse.y - BOW.y;
  if (dx < 30) dx = 30;
  const dist = Math.hypot(dx, dy);
  const speed = Math.max(120, Math.min(1500, dist));
  const nx=dx/dist, ny=dy/dist;
  return { vx: nx*speed, vy: ny*speed, speed };
}
```
If dist becomes 0? dx=30, dy=0? dy maybe 0, dist=30, speed=120. okay.
Need if mouse not set, default.

Need in `drawTrajectory`:
```
function drawTrajectory() {
  const {vx, vy} = getAimVector();
  let x = SHOT_ORIGIN.x, y = SHOT_ORIGIN.y, vyy = vy;
  ctx.strokeStyle='rgba(255,255,255,0.7)'; ctx.lineWidth=2; ctx.setLineDash([4,8]);
  ctx.beginPath();
  ctx.moveTo(x,y);
  const dt = 1/30;
  for (let i=0;i<80;i++) {
    x += vx*dt;
    y += vyy*dt;
    vyy += GRAVITY*dt;
    if (y > groundY || x > canvas.width || x < 0) break;
    ctx.lineTo(x,y);
  }
  ctx.stroke();
  ctx.setLineDash([]);
}
```
Need if state idle/playing? If state finished maybe no. If state idle but overlay, maybe not? Could draw anyway, but overlay covers. Fine.

Need maybe draw crosshair at mouse:
```
ctx.beginPath(); ctx.arc(mouse.x,mouse.y,8,0,2PI); ctx.strokeStyle='#fff'; ... cross lines
```
Need if mouse outside canvas? Still draw? Mouse maybe over canvas only. We can clamp.

Need update mouse pos on mousemove:
```
canvas.addEventListener('mousemove', (e) => {
 const rect=canvas.getBoundingClientRect();
 mouse.x = e.clientX - rect.left;
 mouse.y = e.clientY - rect.top;
});
```
Need if canvas CSS size maybe same as width? Use style max-width 100%; width auto. Need getBoundingClientRect with scaling. If CSS scales down, need map:
```
const scaleX = canvas.width / rect.width;
mouse.x = (e.clientX - rect.left) * scaleX;
mouse.y = (e.clientY - rect.top) * scaleY;
```
Do that.

Need shoot:
```
canvas.addEventListener('click', (e) => {
  if (state !== 'playing') return;
  const {vx,vy} = getAimVector();
  arrows.push({x: SHOT_ORIGIN.x, y: SHOT_ORIGIN.y, vx, vy, angle: Math.atan2(vy,vx), stuck:false});
  arrowCount++;
  arrowCountDisplay.textContent = arrowCount;
});
```
Need if click overlay start button, click event might bubble to canvas and shoot immediately after start? Because overlay button is over canvas; if user clicks start, event might propagate to canvas? Since overlay is a div on top; canvas is behind. The click on button target won't trigger canvas listener unless propagation? It doesn't bubble to canvas because canvas is not ancestor; overlay and canvas siblings in game-area. So no. Need attach click to canvas itself; if overlay target, not on canvas. Good.
Need if click on canvas while state idle? no.

Need "开始游戏" handles overlay button click. Could use `startBtn.addEventListener('click', startGame)`.
Need "resetBtn" resets. If state finished with modal, reset should hide modal and show overlay. Good.

Need "modal" has "againBtn" and "saveBtn". Need if modal hidden, no issue.

Need `resetGameState()`:
```
function resetGameState() {
  state='idle';
  arrows = [];
  targets.forEach(t => t.hit=false);
  hitTargets.clear();
  arrowCount=0; elapsed=0; finalTimeValue=null;
  timerDisplay.textContent='0.00';
  hitCountDisplay.textContent='0';
  arrowCountDisplay.textContent='0';
  nameInput.value = '';
  saveBtn.disabled = false;
  modal.classList.add('hidden');
  overlay? We'll manage separately
}
```
Need "overlay" show/hide. Could use resetGame() to call resetGameState(); overlay.classList.remove('hidden'). startGame calls resetGameState(); overlay.classList.add('hidden'); state='playing'; startTime=performance.now(); lastTime=performance.now();.

Need if `performance.now()` after reset? Fine.

Need perhaps draw "hitTargets.size" based on target hit states after reset. In update hit count uses Set. If target reset, Set clear.

Need `targets` as const; when reset we mutate.

Need "already has records" loaded on page load. renderLeaderboard().

Need "target hit detection" if arrow speed low and dt not too large. Need substeps. Implement:
```
const SIM_SUBSTEPS = 8;
```
With max dt 0.05, substep 0.00625. Good. But if arrow speed 1500, substep distance 9.4 px, less than target radius 50, okay. Need update vy per substep.

Need "arrow stuck at target" check should happen after moving; if arrow overshoots target between substeps? Since substep small enough. okay.

Need if arrow hits target and `hitTargets.size === targets.length`, finishGame inside update after arrow hit. Need if finishGame sets state='finished'; no more arrow updates? `update` still calls updateArrows; arrows may continue? Could stop new shots but in-flight arrows continue. When state finished, updateArrows can still move existing arrows; but timer no. That's fine. If finish triggered, `hitTargets.size===4`; `state='finished'`. In updateArrows, after a hit, if state==='finished' maybe can still process stuck? Need updateArrows runs each frame regardless; okay. But if state finished and we call finishGame inside loop, modal appears; arrows can continue to move behind modal. Fine.

Need maybe stop drawing trajectory when state finished. Use `if (state !== 'finished') drawTrajectory();`
Need if state idle, draw trajectory? Could be okay. But modal/overlay covers. Might as well draw.

Need "speedDisplay" update each frame or mousemove:
```
function updateInfo() {
 const {speed}=getAimVector();
 speedDisplay.textContent='速度：'+Math.round(speed);
}
```
Call in loop. But if state finished, still maybe.

Need "已射箭数" increments every click; no limit.

Need "show timer in start overlay" maybe no.

Need "rank" if not saved yet, "rankPreview" maybe:
```
const rank = computeRankAfterAdd(finalTimeValue);
rankPreview.textContent = `当前成绩将排在：第 ${rank} 名`;
```
But computeRankAfterAdd can compute rank if inserted among existing records:
```
function computeRankAfterAdd(time) {
 const records = loadRecords();
 const recordTime = Math.round(time*100)/100;
 let rank = 1;
 for (const r of records) {
   if (r.time < recordTime) rank++;
   else if (r.time === recordTime) rank++; // if same, later rank
   else break;
 }
 return rank;
}
```
Actually if duplicates same time, new entry ranks after existing. So rank = count(records with time <= current time)+1? If r.time <= recordTime, rank++. If time sorted. Good.
Need after save, `currentRank = records.indexOf(record)+1`, same.

Need "rankPreview" in modal:
```
<div id="rankPreview"></div>
```
Need "saveBtn" disabled initially false. If save clicked, button disabled true. If reset, false.

Need "playerNameInput" default maybe "射手". Need maybe set focus in modal. Could call `nameInput.focus()` in finishGame; but `modal.classList.remove('hidden')` then focus. Need if browser blocks? fine.

Need "modal" CSS. Need if save button disabled after save, "已保存" maybe; update text? Could set `saveBtn.textContent = '已保存 ✓'`, then on reset restore `保存成绩`. Good. But not necessary. Need maybe to avoid duplicate saves. We'll implement:
```
if (saveBtn.disabled) return;
...
saveBtn.disabled = true;
saveBtn.textContent='已保存 ✓';
```
On resetGameState: `saveBtn.disabled=false; saveBtn.textContent='保存成绩';`.

Need "renderLeaderboard" after save. Need if no records, render empty.

Need "startGame" if modal is open? hide.

Need "overlay start" with instructions:
```
<div class="overlay hidden" id="overlay">
  <div class="overlay-content">
    <h2>🎯 命中 4 个靶子</h2>
    <p>移动鼠标调整瞄准，白色虚线为预计弹道</p>
    <p>点击鼠标发射弓箭</p>
    <button id="startBtn">开始游戏</button>
  </div>
</div>
```
Maybe "速度由鼠标到射手距离决定：移动得越远，射得越快" to help.

Need "canvas" focused? not.

Need "target hit count" if a target hit, set target.hit=true and `hitTargets.add(t.id)`. If in updateArrows, after finishGame state changed, but for loop continues. Need avoid calling finishGame multiple? Since hitTargets.size equals 4 only once. But if another target hit after finishGame? All already hit. okay.

Need "arrowStuck" on target; if arrow hit target center, draw arrow. Since arrow angle from velocity; in drawArrow, use `a.angle`. If arrow stuck at target, arrow appears perhaps perpendicular to surface? okay.

Need "drawArrow":
```
function drawArrow(a) {
 ctx.save();
 ctx.translate(a.x,a.y);
 ctx.rotate(a.angle);
 ctx.strokeStyle='#8B5A2B'; ctx.lineWidth=3;
 ctx.beginPath(); ctx.moveTo(-24,0); ctx.lineTo(10,0); ctx.stroke();
 ctx.fillStyle='#c0392b';
 ctx.beginPath(); ctx.moveTo(16,0); ctx.lineTo(6,-4); ctx.lineTo(6,4); ctx.closePath(); ctx.fill();
 ctx.fillStyle='#ddd';
 ctx.beginPath(); ctx.moveTo(-24,-3); ctx.lineTo(-18,0); ctx.lineTo(-24,3); ctx.fill();
 ctx.restore();
}
```
Need arrow head tip at x=16. For collision, arrow point maybe x+? but we use a.x. okay.
Need if arrow stuck in target center, head at x+16; collision at tip? We used a.x = head? Actually a.x is arrow center/head? Let's set a.x SHOT_ORIGIN.x. In draw, arrow head at x+16, body behind. Collision uses a.x, not head. Need if target edge, arrow center at distance 50; arrow head extends 16 beyond, maybe should hit before. Could set a.x as head point. Better in draw, arrow head at x, body from x-30 to x. Then collision uses a.x as tip. Let's modify drawArrow:
```
line from (x-32,0) to (x,0), head triangle from (x,0) to (x-10,-4) etc.
```
Then translation at a.x = tip. In update, a.x is tip. SHOT_ORIGIN at bow. Good. Draw:
```
ctx.beginPath(); ctx.moveTo(-32,0); ctx.lineTo(0,0); // shaft
head: moveTo(4,0); lineTo(-6,-4); lineTo(-6,4)
fletch at -32...
```
But arrow head tip at +4? Let's set tip at 4:
- shaft from -30 to 4
- head triangle points 10, -2,-4, -2,4. Hmm.
Need simpler:
```
ctx.moveTo(-30,0); ctx.lineTo(6,0); // shaft
// head
ctx.beginPath();
ctx.moveTo(14,0); ctx.lineTo(2,-5); ctx.lineTo(2,5); ctx.fill();
// fletch at -30
```
Then tip at 14. Collision point should be a.x + 14? Hard. We can set a.x = head tip by drawing shaft from -44 to 0, head at 0. So:
```
ctx.moveTo(-44,0); lineTo(0,0);
head moveTo(4,0); lineTo(-6,-4); lineTo(-6,4); fill
```
Tip = +4. Fine, negligible. Let's set tip at x=4? Actually if translation at a.x, head extends to 4. Collision uses a.x. okay.

Need "target hit if distance(a.x, a.y) <= t.radius" with a.x near center; good.

Need "arrow out-of-bounds" after collision. If arrow passes target quickly, hit. Fine.

Need "draw current trajectory and nocked arrow" perhaps "loaded arrow" drawn behind preview. It might conflict with drawBow string. okay.

Need "drawNockedArrow":
```
function drawNockedArrow() {
 if (state==='finished' && hitTargets.size===targets.length) return? maybe no.
 const {vx,vy}=getAimVector();
 const angle=Math.atan2(vy,vx);
 ctx.save();
 ctx.translate(BOW.x, BOW.y);
 ctx.rotate(angle);
 ctx.strokeStyle='#B5651D'; ctx.lineWidth=3;
 ctx.beginPath(); ctx.moveTo(-30,0); ctx.lineTo(6,0); ctx.stroke();
 ctx.fillStyle='#c0392b'; ctx.beginPath(); ctx.moveTo(14,0); ctx.lineTo(4,-4); ctx.lineTo(4,4); ctx.fill();
 ctx.restore();
}
```
Need if angle changes, arrow rotates. Maybe draw arrow not too long.

Need "targets y" maybe target 4 at x=940, y=310; target pole goes to groundY. Need draw pole behind target; target face at center. If target y=310, bottom 360, pole length 220. okay.

Need "draw scene before targets? Pole behind target; target face over. Bow at left. Good.

Need "groundY = 580" and target4 pole tall. Need canvas height 650, okay.

Need maybe "The arrow origin" at BOW.x, BOW.y, but if bow at (120,510) and arrow target at (940,310), line from BOW to mouse. Need maybe mouse near (870,143) to hit T4. This is above canvas top? y=143 within. good.

Need "speed = dist" min 120. At mouse near target T4? If cursor at target center (940,310), v=(820,-200), speed=844, trajectory not hit target because gravity. The dashed preview will show arrow landing below target. User will adjust. Good.

Need perhaps "hit all targets" maybe with trajectory preview too easy, but okay.

Need "simulation" maybe includes "gravity" and "arrow flight". Could mention in instructions.

Need "rank" if final time 0? impossible.

Need "localStorage" on file:// maybe works but some browsers block for security? Usually localStorage works on file://; if not, try/catch. Could also fallback memory.

Need "Error: localStorage undefined"? Use:
```
function loadRecords() {
  try {
    const data = localStorage.getItem(STORAGE_KEY);
    return data ? JSON.parse(data) : [];
  } catch { return []; }
}
function saveRecords(records) {
  try { localStorage.setItem(STORAGE_KEY, JSON.stringify(records)); }
  catch(e) { /* fallback ignore */ }
}
```
Good.

Need "test instructions" maybe include:
1. 保存为 `index.html`，双击打开。
2. 点击“开始游戏”。
3. 在画布中移动鼠标，观察虚线弹道；点击发射。
4. 命中所有四个靶子后，输入昵称保存成绩。
5. 排行榜会自动刷新；刷新页面后记录仍保留。

Need perhaps "测试说明" includes "若想快速测试命中判定，可在浏览器控制台执行 `targets.forEach(t=>{t.hit=true; hitTargets.add(t.id);}); hitCountDisplay.textContent=4; finishGame();`" Hmm maybe unnecessary. But output requires "必要的测试说明" maybe not too much. We can mention "自动测试建议：页面加载无报错，点击开始后计时器增长；每击中一个靶子命中数+1；全部命中弹窗出现；保存后排行榜更新且刷新不丢失。" Good.

Need think about "不要查看其他文件，直接创建" maybe they don't want consulting other files. Our final can say "不需要其他文件". Good.

Need perhaps "output requirement" "直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" We should include final answer with code and instructions. Need maybe include code block. Ensure no hidden extra files.

Need perhaps "not view other files" use "本方案为单个 HTML 文件，不依赖外部资源". Good.

Need maybe "game should have four targets" yes.

Need "排行榜 name" maybe if no input, default "匿名射手". Good.

Need "ranking by shortest time" yes sorted ascending.

Potential issue: "命中所有目标所用的时间越短" Maybe if user shoots multiple arrows, time from start to all hit. Yes.

Potential issue: "排行榜上的名次就越高" "higher rank" but time shorter -> rank 1. Good.

Need maybe "target hit count" if arrow hits target with no target score but target turns green. Good.

Need maybe "arrows" not limited. But "已射箭数" could encourage fewer arrows? However ranking only time. But need "玩家命中所有目标所用的时间越短" no mention arrows. Fine. Could include arrows as info.

Need maybe "if player shoots before start?" canvas listener only if playing. Good.

Need "overlay hidden" after start. Need if user clicks reset during playing, overlay appears. Need if user clicks start after reset, works. Need event listeners.

Need "game loop always running" on page load. Need requestAnimationFrame. Good.

Need "lastTime" initialization:
```
let lastTime = performance.now();
function loop(now) {
  requestAnimationFrame(loop);
  const dt = Math.min((now-lastTime)/1000, 0.05);
  lastTime = now;
  update(dt);
  draw();
}
requestAnimationFrame(loop);
```
Need if tab inactive, dt clamp 0.05. okay.

Need "dt" if performance.now has high precision. okay.

Need "draw" should also update "speedDisplay" maybe in update or draw. In `update`:
```
const {speed} = getAimVector();
speedDisplay.textContent = '速度：' + Math.round(speed);
```
Need if `getAimVector` uses mouse pos; no issue.

Need "mouse" coordinates maybe if not moved, mouse at {700, 350}? Let's set default `mouse = { x: 700, y: 300 }`. At load, start overlay covers; draw shows trajectory to mouse. Fine.

Need "canvas click" if click on canvas and state playing. Need if user clicks start overlay, click not on canvas. Good. But after start, overlay hidden. Need if user clicks reset while state playing, overlay shows; state idle. Good.

Need "saveRecord" if `finalTimeValue` null? Button hidden if no finish.

Need "computeRankAfterAdd" maybe:
```
function computeRankAfterAdd(time) {
  const records = loadRecords();
  const t = Math.round(time*100)/100;
  let rank=1;
  for (const r of records) {
    if (r.time <= t) rank++;
    else break;
  }
  return rank;
}
```
Need records sorted ascending. Good.
After save:
```
const record = { name, time: t, date: new Date().toLocaleString() };
records.push(record);
records.sort((a,b)=>a.time-b.time || a.date.localeCompare(b.date));
const rank = records.indexOf(record)+1;
const top = records.slice(0,20);
saveRecords(top);
saveBtn.disabled=true;
saveBtn.textContent='已保存 ✓';
rankPreview.textContent = `已保存！当前排名：第 ${rank} 名`;
renderLeaderboard();
```
Need if rank >20, top doesn't include record but rank display ok. But if rank >20, leaderboard no current. Fine.
Need if saveRecords(top) after ranking; localStorage only top20. Good.
Need issue: if rank >20, rank computed before slicing. Good.
Need `date.localeCompare` if equal date? no.

Need "renderLeaderboard" after reset no changes.

Need "resetGameState" should not erase records. no.

Need "hitCountDisplay" uses `hitTargets.size` not "targets.filter". okay.

Need "draw" order:
```
function draw() {
  ctx.clearRect(0,0,W,H);
  drawSky();
  drawGround();
  targets.forEach(drawPole? Actually drawPole before target);
  drawTargets();
  drawBow();
  drawNockedArrow();
  arrows.forEach(drawArrow);
  if (state!=='finished') drawTrajectory();
  drawMouseHint();
}
```
Need draw order:
- Sky/ground
- Target poles behind target face. But if target face drawn after pole, fine. Could draw all targets with pole+face.
- Bow and arrow maybe should be in front? If bow at left, no issue.
- Nocked arrow should be under/above bow? maybe after bow.
- Active arrows after targets, they can overlap.
- Trajectory dashed maybe should be above scene but under arrows? Draw before arrows maybe.
- Mouse crosshair.

Need drawTargets with pole. But if target face drawn after arrow? no.

Need "drawPole" if target hit and check, okay.

Need "drawSky":
```
const grd = ctx.createLinearGradient(0,0,0,groundY);
grd.addColorStop(0,'#87CEEB'); grd.addColorStop(1,'#dceff7');
ctx.fillStyle = grd; ctx.fillRect(0,0,W,H);
// sun
ctx.fillStyle='#fff3b0'; ctx.beginPath(); ctx.arc(960,80,40,0,2PI); ctx.fill();
ctx.fillStyle='#ffe066'; ctx.beginPath(); ctx.arc(960,80,32,0,2PI); ctx.fill();
// clouds
...
```
Need drawGround:
```
ctx.fillStyle='#6ab04c'; ctx.fillRect(0,groundY,W,H-groundY);
ctx.fillStyle='#8bc34a'; ctx.fillRect(0,groundY, W, 12);
```
Need perhaps draw a target stand base.

Need "drawTarget" pole line:
```
ctx.strokeStyle='#6d4c2f';
ctx.lineWidth=10;
ctx.beginPath();
ctx.moveTo(t.x, t.y + t.radius - 6);
ctx.lineTo(t.x, groundY);
ctx.stroke();
ctx.fillStyle='#4a2f1a';
ctx.fillRect(t.x-35, groundY-10, 70, 16);
```
Need if target face bottom below pole? okay.

Need "drawTarget" label above target:
```
ctx.font='bold 18px "Segoe UI", sans-serif';
ctx.fillStyle='#ffffff';
ctx.strokeStyle='rgba(0,0,0,0.5)';
ctx.lineWidth=3;
ctx.strokeText('T'+t.id, t.x, t.y - t.radius - 12);
ctx.fillText('T'+t.id, t.x, t.y - t.radius - 12);
```
Need maybe label can overlap pole? okay.
Need "hit" overlay:
```
if(t.hit){
  ctx.beginPath();
  ctx.arc(t.x, t.y, t.radius+5,0,2PI);
  ctx.fillStyle='rgba(46,204,113,0.2)';
  ctx.fill();
  ctx.strokeStyle='#2ecc71';
  ctx.lineWidth=5;
  ctx.stroke();
  ctx.font='bold 52px sans-serif';
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillStyle='#2ecc71';
  ctx.fillText('✓', t.x, t.y+2);
}
```
Need if text baseline maybe.

Need "drawTrajectory" uses `ctx.setLineDash([5,10])`. Need reset after. If `ctx.lineDashOffset` maybe no.

Need "drawMouseHint":
```
ctx.beginPath();
ctx.arc(mouse.x, mouse.y, 10, 0, 2PI);
ctx.strokeStyle='rgba(255,255,255,0.9)';
ctx.lineWidth=2;
ctx.stroke();
ctx.beginPath();
ctx.moveTo(mouse.x-16, mouse.y); ctx.lineTo(mouse.x+16, mouse.y);
ctx.moveTo(mouse.x, mouse.y-16); ctx.lineTo(mouse.x, mouse.y+16);
ctx.stroke();
```
Need if mouse outside canvas? draw at mouse maybe outside; could be okay. Maybe clamp? If mouse leaves canvas, draw last coordinate maybe outside? Actually can be outside, arrow may draw outside; but no issue. We can clamp mouse position to canvas bounds along with mousemove; but if pointer leaves, no events. Fine.

Need "getAimVector" with `dx < 30` if mouse behind; if mouse.x huge etc. Good.

Need "startBtn" overlay maybe "开始游戏" plus "重置" not. Good.

Need "resetBtn" with `resetGame()`:
```
function resetGame() {
  resetGameState();
  overlay.classList.remove('hidden');
}
```
Need if startGame calls resetGameState not resetGame:
```
function startGame() {
  resetGameState();
  overlay.classList.add('hidden');
  state='playing';
  startTime=performance.now();
  lastTime=performance.now();
}
```
Need "resetGameState" hides modal and maybe overlay? It should not touch overlay. We'll separate.

Need "againBtn2" in modal:
```
againBtn2.addEventListener('click', startGame); // but modal is hidden by resetGameState, startGame resets and starts immediately.
```
If user wants "再来一局", start new game right away. Good.
Need "startBtn" if already playing? overlay hidden so no.

Need "resetBtn" while state finished and modal open? Reset button behind modal not clickable. But modal has again button.

Need "modal" close? Not necessary. But if user clicks save, modal remains until clicking "再来一局" or "重置". That's okay. But maybe should allow close after save to view leaderboard. Add "关闭" button? Or "再玩一次" enough. Let's add third button "关闭" to hide modal. But if close after save, can view leaderboard. Need no matter.
Modal content:
```
<button id="saveBtn">保存成绩</button>
<button id="againBtn">再来一局</button>
<button id="closeModalBtn">关闭</button>
```
Need close button hides modal and if state finished, no shooting. If user later clicks reset, start. Add to code.

Need "closeModalBtn" event hideModal. But if state finished and modal hidden, user can't shoot; can reset. okay.

Need "only top 20 records". Good.

Need "game area" canvas responsive. CSS:
```
body {
  margin:0;
  font-family: 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  background: linear-gradient(135deg,#1b3b4f,#2d5f6e);
  min-height:100vh;
  display:flex; justify-content:center; align-items:center;
}
.container { max-width: 1200px; width: 100%; padding:20px; color:white; }
h1 { text-align:center; color:#ffd966; text-shadow: 0 2px 8px rgba(0,0,0,0.4); }
.subtitle { text-align:center; color:#dce9ef; margin-bottom: 16px; }
.game-area { position:relative; display:flex; justify-content:center; }
canvas { width:100%; height:auto; max-width:1100px; border-radius:16px; box-shadow:0 8px 32px rgba(0,0,0,0.45); background:#000; cursor:crosshair; }
.overlay { position:absolute; top:0; left:50%; transform:translateX(-50%); width:100%; max-width:1100px; height:100%; background:rgba(0,0,0,0.45); border-radius:16px; display:flex; align-items:center; justify-content:center; }
.overlay-content { background:rgba(20,40,60,0.9); border:2px solid #ffd966; border-radius:16px; padding:30px 40px; text-align:center; }
...
```
But `.game-area` needs width 100%; canvas width 100%; overlay max-width? Need overlay align with canvas. Since game-area max width? We can set `.game-area { position:relative; width:100%; max-width:1100px; margin:0 auto; }`. Canvas width:100%; overlay width:100%; top:0; bottom:0. Good.
Info bar: flex with cards. CSS:
```
.info-bar {
  display:flex; gap:16px; justify-content:center; flex-wrap:wrap; margin:16px 0; background:rgba(0,0,0,0.25); border-radius:12px; padding:12px;
}
.info-bar div { background:...; padding:8px 16px; border-radius:8px; font-weight:600; }
```
Buttons:
```
.btn { background:#ffd966; color:#1b3b4f; border:none; padding:12px 24px; border-radius:8px; font-size:18px; font-weight:700; cursor:pointer; }
.btn:hover { background:#ffed9e; }
.btn.small { padding:8px 16px; font-size:14px; }
```
Leaderboard:
```
.leaderboard { background:rgba(0,0,0,0.25); border-radius:16px; padding:16px; max-width:800px; margin:20px auto 0; }
table { width:100%; border-collapse:collapse; color:white; }
th,td { padding:8px; border-bottom:1px solid rgba(255,255,255,0.15); text-align:center; }
th { background:rgba(255,215,0,0.15); }
.empty { color:#aaa; padding:20px; }
```
Modal:
```
.modal { position:fixed; inset:0; background:rgba(0,0,0,0.7); display:flex; align-items:center; justify-content:center; z-index:50; }
.modal.hidden { display:none; }
.modal-content { background:linear-gradient(135deg,#2b5a6a,#1b3b4f); border:2px solid #ffd966; border-radius:16px; padding:30px 40px; text-align:center; min-width:320px; }
input { ... }
```
Need hidden class for overlay:
```
.overlay.hidden { display:none; }
```
Need `.info-bar #speedDisplay` maybe.

Need "canvas" maybe has `display:block` to avoid white space. Add.

Need "overlay" center with `left:50%; transform:translateX(-50%)`; if `width:100%`, okay. But game-area max-width 1100; overlay left 50% of game-area? position absolute relative to game-area. Good.
Need "overlay-content" not hide.

Need "modal" hidden if not used.

Need "Start game" instructions maybe "开始后计时立即开始". Good.

Need "speedDisplay" value "速度：0" initial.

Need "localStorage key" maybe `'archery_leaderboard_v1'`.

Need "rank preview" in modal:
```
<p class="rank-preview" id="rankPreview"></p>
```
Need if load records empty, rank 1. Good.

Need "Name input" with `maxlength=10`. Need maybe "匿名射手".

Need "saveBtn" disabled after save. In startGame reset, `saveBtn.disabled=false`; `saveBtn.textContent='保存成绩'`.

Need "closeModalBtn" if user closes without saving, records not saved. Maybe if they close, they can still reset. okay.

Need "finishGame" should maybe update `hitCountDisplay` already. Good.

Potential problem: In `updateArrows`, if arrow hits a target and `t.hit` false, we mark hit. But if `t.hit` true and arrow passes, we set stuck. If arrow hits target center, arrow remains stuck inside target. Later drawTargets maybe draws target after arrows? If arrows drawn after targets, arrow visible. Good.
Need if target hit and arrow stuck at (t.x, t.y), draw arrow from t.x maybe; okay.
Need if target hit, `hitTargets` add. If arrow hits target and state finished, `finishGame` opens modal. `updateArrows` continues but all arrows might be stuck? Fine.

Need "drawTargets" if hit green overlay over arrow? Since arrows drawn after targets, arrow visible. Good.

Need "drawTrajectory" after active arrows? If draw trajectory after arrows, dashed line overlays arrows. Maybe draw trajectory before arrows. We'll set:
```
drawBow();
drawNockedArrow();
if (state==='playing') drawTrajectory();
arrows.forEach(drawArrow);
drawMouseHint();
```
Trajectory before arrows so arrows visible. But current aim line drawn on top of bow; okay.

Need "drawNockedArrow" draws arrow on bow even if active arrows. It might be weird: when an arrow is in flight, a second arrow appears at bow, representing next shot. That's okay.

Need "update speedDisplay" in update:
```
function update(dt) {
  if (state==='playing') { ... }
  const speed = getAimVector().speed;
  speedDisplay.textContent = '速度：' + Math.round(speed);
  updateArrows(dt);
}
```
Need if state finished modal open, speed display still updated? okay.

Need "updateArrows" should only update if arrows.length. Good.

Need "targets" need mutable `hit` but if `targets` used as constant and reset. Good.

Need "arrowCountDisplay" in startGame reset and shoot. Good.

Need "hitCountDisplay" in update maybe after target hit. okay.

Need "timerDisplay" if state idle after reset, "0.00".

Need "lastTime" in startGame reset may be from previous; if not reset, dt after overlay maybe large. We set lastTime=performance.now() in startGame. Good.

Need "mousemove" coordinate if canvas CSS scales. Need:
```
canvas.addEventListener('mousemove', (e) => {
 const rect = canvas.getBoundingClientRect();
 mouse.x = (e.clientX - rect.left) * (canvas.width / rect.width);
 mouse.y = (e.clientY - rect.top) * (canvas.height / rect.height);
});
```
Need if rect.width 0? no.

Need "click" event uses current mouse; if click outside? canvas only. Good.

Need "targets" maybe "T4" at x=940, y=310; its pole at x=940. But arrow trajectory maybe with x max 960; okay.

Need "drawing target labels" with textAlign changed. Need reset textBaseline maybe:
```
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';
```
In drawTarget and drawArrow independent due save.

Need "drawSky" text? no.

Need "leaderboard table" maybe `<table>` inside div: 
```
<div id="leaderboardBody"></div>
```
Could create `<table><thead>...<tbody id="leaderboardBody"></tbody></table>`. In render, set tbody.innerHTML. Good.

Need "current rank" if not save? maybe no.

Need "rank by shortest time" table sorted ascending. Good.

Need "游戏玩法" maybe in subtitle with "移动鼠标调整方向：鼠标离射手越远，力度越大；白色虚线显示飞行轨迹". Good.

Need "HTML comments" maybe not needed.

Potential issue with `Math.round(time*100)/100` and records sort. If time=1.234 and other=1.23, rounds to 1.23; okay.

Need "finalTimeValue" maybe float; if display uses `toFixed(2)`, save uses rounded. Good.

Need "if user enters name with spaces" trim. Good.

Need "if localStorage stores old version" no.

Need "all four targets hit" if arrow passes through target at high speed and substep too large? substep ~0.00625, speed 1500 => 9.375 px, target radius 50, fine. Wait max dt 0.05, substeps 8 => 0.00625; speed 1500 => 9.375 px. Good.

Need "arrow after hitting target, `a.stuck = true`; if multiple targets overlap? no.

Need "shoot multiple arrows in same frame" no.

Need "Since arrow is a point, can pass through target if target radius 50 and substep 9; okay. Need if arrow enters target but target center distance <= radius. Good.
Need maybe check collision after update, not before. If arrow enters target between substeps, okay approx.

Need "trajectory preview" with arrow origin SHOT_ORIGIN. Define:
```
const SHOT_ORIGIN = { x: 128, y: 510 };
```
But BOW.x=120, BOW.y=510. If SHOT_ORIGIN is BOW.x + 8. Let's set:
```
const BOW = { x: 125, y: 510 };
const SHOT_ORIGIN = { x: BOW.x + 5, y: BOW.y - 5 };
```
But getAimVector uses BOW; if arrow starts at SHOT_ORIGIN, trajectory preview starts slightly high. Need if launching from BOW, okay. Use SHOT_ORIGIN = BOW to avoid mismatch. Let bow limb maybe at BOW. Use `SHOT_ORIGIN = {x: BOW.x + 5, y: BOW.y}`. Trajectory with origin slightly. If targets require exact, not huge. But if using preview, we need actual origin same. Use `SHOT_ORIGIN = BOW`? Bow string line at BOW.x? Let's set BOW.x = 125, BOW.y = 510; SHOT_ORIGIN = BOW. Arrow drawn at bow. Fine.
Need no "launch at string" exact. Use same.

Need "if aim vector with dx < 30" and mouse y same as bow, arrow horizontal at speed 120? Actually if mouse at (155,510), dx=30, dist=30, speed=120, arrow horizontal. If click near, speed min 120. Fine.
Need "arrow count" increments on click. Need if state playing. Good.

Need "start overlay" appears initially; `state='idle'`; draw still draws bow/targets and trajectory maybe. If user moves mouse behind overlay, can see dashed lines? Overlay translucent, maybe. Fine.

Need "resetGame" if overlay hidden? Shows overlay. But "startGame" resets state with no overlay. Good.

Need "closeModal" if final time not saved and then state finished; user can reset. Good.

Need perhaps "saveBtn.disabled" reset in resetGameState. Good.

Need "renderLeaderboard" use `records.slice(0,10)` or 20. Use top 10 to fit. But requirement "排行榜" maybe top 10 enough. Use top 10. Need "名次" if top 10. Good.

Need "record date" maybe "2025-01-01 10:00". Use `new Date().toLocaleString('zh-CN')`. Good.

Need "empty" row set `colspan=4` if table has 4 columns. We'll have table columns: 名次, 玩家, 用时, 日期. Good.

Need "action buttons" maybe no.

Need "modal rankPreview" if no records "当前成绩将排在第 1 名". Good.

Need "input default" maybe "射手" + random? Could use "无名射手". Better set value `'射手' + (records.length + 1)`? But if user doesn't want, can edit. Let's set on finish:
```
const records = loadRecords();
nameInput.value = `射手${records.length + 1}`;
```
Need if reset, nameInput.value remains but overwritten. Good.

Need "saveBtn" event if user presses Enter in input? Add:
```
nameInput.addEventListener('keydown', e => { if(e.key==='Enter') saveBtn.click(); });
```
Good.

Need "prevent context menu" no.

Need "canvas width/height" if CSS max-width changes, coordinate mapping handles. Need `draw` uses internal coordinate W/H. Good.

Need "text align" in drawTarget, drawScene. We'll set before each fillText.

Need "drawGround" and target poles maybe.

Need "bow" drawing if arrow active: Need draw bow before nocked arrow; the string line maybe vertical, arrow passes through. okay.

Need "lineCap" reset? no.

Potential issue: `getAimVector` uses `mouse` which may be in CSS pixel if not mapped? We map. Good.

Need "if mouse moves outside, last position remains; if click outside no event. okay.

Need "overlay" hidden? At page load, state='idle', overlay not hidden. Need `resetGameState()` on load? We can just initialize and call `renderLeaderboard()`. Need state variable and arrays initialized before loop. Use let declarations. Need not call reset? We'll define:
```
let state = 'idle';
let arrows = [];
const hitTargets = new Set();
let arrowCount = 0;
let startTime = 0;
let elapsed = 0;
let finalTimeValue = null;
let lastTime = performance.now();
const mouse = { x: 700, y: 300 };
```
Need targets defined with hit false.

Need "startTime" if overlay not started, no.

Need "loop" starts; update(dt) with state idle still updates speed and arrows (none). draw.

Need "drawNockedArrow" should work even state idle. Fine.

Need "if state finished and modal hidden, still state finished; click canvas no shoot; user can reset. Good.

Need "if state finished and then save, modal remains; user closes. Speed display still. Canvas behind. okay.

Need "if user clicks reset after game, overlay show and state idle; modal hide. Good.

Need "if user clicks start after game, startGame reset and hides overlay. Good.

Need "renderLeaderboard" after localStorage may not exist; catch. Good.

Need "loadRecords" returns array. Need avoid mutating records before sort? okay.

Let's think about target positions more carefully in relation to trajectory mechanics. Need all four target locations hit from BOW=(125,510). Let's simulate approximate with getAimVector speed=dist, gravity=300. Need maybe some target too hard due low min speed? Let's calculate target 1 at (330,470). Need hit with arrow start (125,510), target diff (205,-40). Need choose parameters. If mouse at (550,330): dx=425, dy=-180, dist=462, v=(425,-180), speed 462. Time to x=330: dx=205/425 =0.482s. y = 510 + (-180*0.482) + 150*0.232 =510 -86.8 +34.8 =458. Close above target (470). So target hit maybe. Good. T1 easy.

Target 2 at (540,390), diff (415,-120). If mouse at (700,250): dx=575, dy=-260, dist=631, t=415/575=0.722, y=510 -187.7 +78.2=400.5. Close. Good.
Target3 at (760,460), diff (635,-50). If mouse at (800,300): dx=675, dy=-210, dist=707, t=635/675=0.941, y=510 -197.6 +132.8=445.2. close (needs 460). Adjust.
Target4 at (930,320?) Wait earlier set T4 (940,310). diff (815,-200). If mouse at (850,150): dx=725, dy=-360, dist=809, t=815/725=1.124, y=510 -404.6 +189.5=294.9, close. Good. So feasible.

Need perhaps target4 y maybe 320 to make easier? Actually target4 far high; if y=320, diff -190. hit maybe. Use y=315. Let's choose:
```
{id:4, x:930, y:320, radius:50}
```
Then diff (805,-190). If aim mouse (850,160), dx=725, dy=-350, t=1.11, y=510-388.5+184.8=306.3; close (target center 320, arrow y 306, within radius? delta 13) yes. Good. Need target radius 50 enough. Let's set T4 x=930,y=320. Canvas width 1100, okay.
Target3 x=760,y=460.
Target2 x=540,y=400? Let's choose y=400, diff (415,-110). With aim (700,260): dx=575, dy=-250, t=0.722, y=510-180.5+78.2=407.7 close. Good. Use y=400.
Target1 x=330,y=470. Good. Need four targets not too close vertically/horiz.

Need perhaps target "T1" at x=330, y=470; T2 at x=540,y=400; T3 x=760,y=460; T4 x=930,y=320. Good.

Need "target radius" maybe 48. Use 50.

Need "draw target labels" if target4 y=320, label at y-radius-12=262, okay.

Need "pole" from y+radius-6 = 364 to ground 580, okay.

Need "SHOT_ORIGIN" at (125,510), groundY 580, target center y 320-470. good.

Need "trajectory preview" uses 80 steps of dt=1/30 => 2.67 seconds. At speed 800, arrow travels maybe 2100 px horizontally before hitting ground; okay. For T4, time ~1.1 sec, within. For low speed 120, time to ground from y=510 (drop 70) ~0.68 sec, x=82; within. Good.
Need if arrow with high speed and small upward angle; preview stops when y>ground. Good.

Need "aim vector" speed = dist from bow to mouse (without factor), not multiplied by 1. If mouse at (800,150): dx=675,dy=-360, speed=765. Good. If mouse at (1000,500): dx=875,dy=-10, speed=875. okay.
Need "if dx<30" uses clamp and then speed computed; if mouse is behind and high, dx=30 but dy negative, speed high, direction mostly upward/right. Fine.

Need maybe "power label" "速度" but should be "力度". Since speed = distance, okay.

Need "game physics" maybe "重力" GRAVITY=300 px/s². Need if arrow speed too high, range with 45° = v²/g = 2440 px at 855, enough. Good.

Need "canvas background" maybe sky.

Need "drawTrajectory" if vx = 30, vertical; okay.

Need "arrow count display" if multiple arrows stuck and later reset? okay.

Need "modal" and "saveBtn" disabled if save clicked; if close and reset, resetGameState restores. Good.

Need "startBtn" repeated? If overlay hidden, no. But in code, startBtn remains.

Need "overlay" hidden if reset? `resetGame()`:
```
function resetGame() {
  resetGameState();
  overlay.classList.remove('hidden');
}
```
If state already idle and overlay visible, clicking reset does nothing. Good.
Need "startGame" calls `resetGameState(); overlay.classList.add('hidden'); state='playing'; startTime=performance.now(); lastTime=performance.now();`
Need "resetGameState" sets state='idle', but startGame sets after. Good.

Need "startGame" maybe if called while game running, reset and restart. okay.

Need "speedDisplay" in info bar maybe `id="speedDisplay"`.

Need "leaderboardBody" id. Good.

Need "closeModalBtn" if user closes modal and state finished; no shooting. Need if user wants start again, click reset in info or "再来一局" not available if modal closed. Reset button visible. okay.

Need perhaps "modal close button" label maybe "关闭". Good.

Need "saveRecord" after save if close, leaderboard shows. Good.

Need "date" maybe `new Date().toLocaleString('zh-CN', { hour12: false })`. Good.

Need "ranking after save" if `saveRecords(top)` where top excludes record but rank computed from full records. If rank >20, the storage no current; but maybe okay. Need maybe keep top 50 to avoid rank >20 prominent but still doesn't matter. Use top 50 for all. Need "leaderboard table top 10" but storage top 50. Good.
```
const top = records.slice(0,50); saveRecords(top);
```
Need if record outside top50, not saved; but if rank >50, no. likely not.
Need "computeRankAfterAdd" before save:
```
let previewRank = 1;
for (const r of records) {
  if (r.time <= t) previewRank++;
  else break;
}
```
Need if records unsorted? loadRecords may from old sorted. We ensure sorted. If storage corrupt, sort? Could `records.sort((a,b)=>a.time-b.time)` in loadRecords. Do:
```
function loadRecords() {
  const arr = JSON.parse...
  arr.sort((a,b)=>a.time-b.time);
  return arr;
}
```
But `loadRecords` used before save and mutates storage? no. Good.

Need "saveRecords" sorts? In save, sort before slice. Good.

Need "renderLeaderboard" after load sorted by time; maybe records from storage older unsorted? no.

Need "resetGameState" uses target.hit false; Set clear. Need if `targets` constant with hit property; okay.

Need "state" variable use `let state = 'idle';`. Need if target hit and finishGame called, state set 'finished'. But in updateArrows there may be `if (hitTargets.size === targets.length) finishGame();` after marking hit; finishGame uses `elapsed = (performance.now() - startTime)/1000`, but update loop has now available? We can compute from `performance.now()` directly. Good.
Need `elapsed` in finishGame final. Good.

Need "time display" during playing with `performance.now() - startTime`. If startTime set from performance.now. Good.
Need "if tab inactive, dt clamped but timer uses actual performance.now - startTime; could count large time if tab inactive. For time ranking, maybe should use actual time. Fine. If tab inactive, arrows freeze but timer continues; could be unfair. Could use accumulated `elapsed += dt` but then timer counts only active frames? Which is better? A speedrun should use real elapsed time; if tab inactive, real time continues, but not relevant. In game loop, dt clamp may hide inactive time but timer uses wall clock; if tab inactive, timer jumps. That's okay but maybe weird. Could use `elapsed += dt` for active frames and start time only? But if tab inactive, dt clamped, time less. Use real time for ranking. Fine.
Need maybe if browser throttles RAF, dt clamped but performance time larger; okay.

Need "shoot arrow if state playing" even if modal? modal hidden during playing. good.

Need "speedDisplay updates" in idle too; if overlay covers, no.

Need "leaderboard table no horizontal overflow" CSS. okay.

Need "Accessibility" no.

Need "test instructions" in final. Could include "手动测试" and "自动化冒烟测试". Since prompt "必要的测试说明" perhaps enough.

Let's now craft final code carefully.

Potential code issues:
- `ctx.arc(t.x + 3, t.y + 3, t.radius, 0, Math.PI*2)` okay.
- `ctx.fillStyle = ...` before `ctx.fill()`.
- Use `Math.PI * 2`.
- In drawTarget if hit overlay fill after stroke? Need path closed. Do:
```
ctx.beginPath();
ctx.arc(t.x, t.y, t.radius + 6, 0, 2 * Math.PI);
ctx.fillStyle = 'rgba(46,204,113,0.25)';
ctx.fill();
ctx.strokeStyle = '#2ecc71';
ctx.lineWidth = 5;
ctx.stroke();
```
Need `ctx.textBaseline='middle'` for check. But in drawTarget, textAlign currently 'center'; okay. Set baseline.

- `drawBow` string line from BOW.y-75 to BOW.y+75. Bow limbs at (bx, by-80) etc. okay.

Need "Nocked arrow" if angle = `Math.atan2(vy,vx)`; if vx from getAimVector maybe small. Draw arrow rotated; body line from -30 to 6 and head. Need shaft line:
```
ctx.beginPath();
ctx.moveTo(-28,0);
ctx.lineTo(4,0);
ctx.strokeStyle='#7a4a2b';
ctx.lineWidth=3;
ctx.stroke();
ctx.fillStyle='#c0392b';
ctx.beginPath();
ctx.moveTo(10,0);
ctx.lineTo(0,-4);
ctx.lineTo(0,4);
ctx.closePath();
ctx.fill();
```
But if translation at BOW, arrow head extends to x=10 right of bow. good.

Need "drawArrow" active:
```
function drawArrow(a) {
  ctx.save();
  ctx.translate(a.x, a.y);
  ctx.rotate(a.angle);
  // shaft
  ctx.strokeStyle='#9c6b3c';
  ctx.lineWidth=3;
  ctx.beginPath();
  ctx.moveTo(-30,0);
  ctx.lineTo(4,0);
  ctx.stroke();
  // head
  ctx.fillStyle='#d35400';
  ctx.beginPath();
  ctx.moveTo(10,0);
  ctx.lineTo(-1,-5);
  ctx.lineTo(-1,5);
  ctx.closePath();
  ctx.fill();
  // fletching
  ctx.fillStyle='#e74c3c';
  ctx.beginPath();
  ctx.moveTo(-30,-4);
  ctx.lineTo(-22,0);
  ctx.lineTo(-30,4);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```
Need if a.x is collision point, arrow head extends to 10, but okay. The collision point effectively at a.x (just before head tip). Accept.

Need if target hit and arrow stuck at center, draw arrow centered at target, head to right along velocity, okay.

Need "drawTarget" if target hit and arrow stuck, check overlay green maybe behind arrow. Good.

Need "drawTrajectory" if `state==='playing'` only. But if state idle and overlay hidden? At page load overlay visible, no need. If state idle and reset overlay visible, no. But drawNockedArrow and arrows no. good. We can condition:
```
if (state === 'playing') drawTrajectory();
```
During finished no trajectory.

Need maybe if state idle, nocked arrow not visible because overlay but okay. `drawNockedArrow` still draws. Could condition `if (state !== 'finished')`. fine.

Need "drawMouseHint" if state playing? We can condition:
```
if (state === 'playing') drawMouseHint();
```
But if idle, overlay. okay.

Need "speedDisplay" update on mousemove maybe better. But loop okay.

Need "state" after resetGameState is 'idle'. In startGame, set 'playing'. Good.

Need "startTime" if no.

Need "modal" hidden class. In CSS `.hidden { display: none !important; }` maybe.

Need "leaderboardBody" `tbody.innerHTML = ...`. Need prevent XSS from name? User can input name; when render using textContent to avoid. Build DOM with `createElement`? Simpler use `innerHTML` template but need escape HTML. Since name user-controlled, could contain HTML? Input maxLength but could include `<img>`, so escape. Need implement `escapeHtml`.
```
function escapeHtml(str) { return str.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }
```
Use in template. Good.

Need "leaderboard rank" if records top 10. Good.

Need "saveRecord" if name input length 0 default. Need no XSS.

Need "renderLeaderboard" date maybe `r.date || ''`. Use escapeHtml.

Need "targets" object unaffected by records.

Need "drawTargets" maybe target4's label "T4" at 262, okay with sky. Good.

Need "overlay content" maybe h2 start. Good.

Need "total targets" hardcoded 4. In `hitCountDisplay`, `targets.length`.

Need "resetGameState" uses `hitTargets.clear(); targets.forEach(t => t.hit = false);` Good.

Need "finalTimeValue" in finishGame:
```
finalTimeValue = Math.round(elapsed * 100) / 100;
```
Need display `elapsed.toFixed(2)` maybe. Use finalTimeValue for save.
Need "rankPreview" compute with finalTimeValue.
Need "saveBtn.disabled = false;" in finishGame.
Need "saveBtn.textContent = '保存成绩';" just in case from previous. Good.
Need "nameInput.value = '射手' + (loadRecords().length + 1);" But if records length 0 -> "射手1". Good. Maybe "玩家1" better.
Need "nameInput.focus()" maybe; but if modal hidden? after class remove. Good.

Need "closeModalBtn" hideModal:
```
function hideModal() { modal.classList.add('hidden'); }
```
Need "modal" variable at top. Good.

Need "againBtn" event startGame; startGame hides modal. Good.

Need "overlay" `startBtn` event:
```
startBtn.addEventListener('click', startGame);
```
Need "resetBtn" event:
```
resetBtn.addEventListener('click', resetGame);
```
Need "closeModalBtn" event:
```
closeModalBtn.addEventListener('click', hideModal);
```
Need "saveBtn" event:
```
saveBtn.addEventListener('click', saveRecord);
```
Need "nameInput keydown".

Need "renderLeaderboard" on DOMContentLoaded? Script at end after elements, so can call directly. Good.

Need "loop" start after render:
```
requestAnimationFrame(loop);
```
Need "lastTime" initialized before loop. okay.

Need "canvas" width/height attributes. If CSS `width:100%; height:auto`, internal dimensions maintain. Good.

Need "target poles" maybe should be drawn before target faces but after ground. In `drawTargets` function loops and draws pole+face. Good.

Need "drawSky" clouds random? We can draw fixed.
```
function drawSky() {
 const grd = ctx.createLinearGradient(0,0,0,groundY);
 grd.addColorStop(0,'#4a90d9');
 grd.addColorStop(1,'#b5dff7');
 ctx.fillStyle=grd; ctx.fillRect(0,0,W,H);
 // sun
 ...
 // clouds
 ctx.fillStyle='rgba(255,255,255,0.7)';
 ctx.beginPath(); ctx.arc(180,90,30,0,2PI); ...
}
```
Need not perfect.

Need "drawGround":
```
ctx.fillStyle='#5d9c4e';
ctx.fillRect(0, groundY, W, H-groundY);
ctx.fillStyle='#7bbf68';
ctx.fillRect(0, groundY, W, 10);
// small grass strokes
```
No.

Need "drawTarget" pole line width maybe 10. Target face center at t.y. If target bottom near ground? For T1 y=470 radius50 bottom=520, pole from 514 to580. okay.

Need "targets" stand base maybe if target4 pole length, okay.

Need "drawTarget" ring colors:
- white outer #f0ede8
- black #333333
- blue #1f6fd9
- red #e03a3a
- inner gold #f6c945
Need "ring stroke" each radius:
```
ctx.lineWidth = 2;
ctx.strokeStyle = 'rgba(0,0,0,0.3)';
for (const r of [t.radius, t.radius*0.78, t.radius*0.56, t.radius*0.36]) {
  ctx.beginPath(); ctx.arc(...); ctx.stroke();
}
```
Need inner gold maybe stroke too. okay.

Need "target hit overlay" if t.hit, maybe target face remains colored. Good.

Need "current score" if no.

Need "Leaderboard table" "用时(s)" column. Good.

Need "Maybe rank should be higher if shorter time" sorted ascending, rank 1 first. Good.

Need "How to run" final code. We'll include code and then instructions. "将以上代码保存为 index.html，用浏览器打开即可。" and "测试说明" bullets.

Need maybe "不要查看其他文件" maybe in final "这是一个独立文件，不读取/依赖任何外部文件。" Good.

Potential issue: "do not view other files" If we mention "不要查看其他文件" no.

Let's now write final code carefully.

Before writing, let's double-check all JS variable names and events.

Full JS code:

```
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const GROUND_Y = 580;
const GRAVITY = 300;
const BOW = { x: 125, y: 510 };
const SHOT_ORIGIN = { x: BOW.x, y: BOW.y };
const STORAGE_KEY = 'archery_leaderboard_v1';

const targets = [
 { id: 1, x: 330, y: 470, radius: 50, hit: false },
 { id: 2, x: 540, y: 400, radius: 50, hit: false },
 { id: 3, x: 760, y: 460, radius: 50, hit: false },
 { id: 4, x: 930, y: 320, radius: 50, hit: false },
];

let state = 'idle';
let arrows = [];
const hitTargets = new Set();
let arrowCount = 0;
let startTime = 0;
let elapsed = 0;
let finalTimeValue = null;
let lastTime = performance.now();
const mouse = { x: 700, y: 300 };

const overlay = document.getElementById('overlay');
const startBtn = document.getElementById('startBtn');
const resetBtn = document.getElementById('resetBtn');
const timerDisplay = document.getElementById('timerDisplay');
const hitCountDisplay = document.getElementById('hitCount');
const arrowCountDisplay = document.getElementById('arrowCount');
const speedDisplay = document.getElementById('speedDisplay');
const leaderboardBody = document.getElementById('leaderboardBody');
const modal = document.getElementById('modal');
const finalTimeDisplay = document.getElementById('finalTime');
const rankPreview = document.getElementById('rankPreview');
const playerNameInput = document.getElementById('playerName');
const saveBtn = document.getElementById('saveBtn');
const againBtn = document.getElementById('againBtn');
const closeModalBtn = document.getElementById('closeModalBtn');
```

Need "againBtn" id in HTML. okay.

`getAimVector`:
```
function getAimVector() {
  let dx = mouse.x - BOW.x;
  let dy = mouse.y - BOW.y;
  if (dx < 30) dx = 30;
  const dist = Math.hypot(dx, dy);
  const speed = Math.max(120, Math.min(1500, dist));
  const nx = dx / dist;
  const ny = dy / dist;
  return { vx: nx * speed, vy: ny * speed, speed };
}
```
If dist=0? dx=30, dy=0; dist=30, no division by zero. If dy huge and dx=30, dist >0. okay.

Need if dist <120, direction normalized from original dist; if dx=30 dy=5 dist=30.4 speed=120, ny=19.7, vx=118.4. okay. If dx=30 dy=0, vx=120.

Potential issue: If mouse far left but high, dx=30, direction mostly upward. fine.

`resetGameState`:
```
function resetGameState() {
  state = 'idle';
  arrows = [];
  hitTargets.clear();
  targets.forEach(t => t.hit = false);
  arrowCount = 0;
  elapsed = 0;
  finalTimeValue = null;
  timerDisplay.textContent = '0.00';
  hitCountDisplay.textContent = '0';
  arrowCountDisplay.textContent = '0';
  saveBtn.disabled = false;
  saveBtn.textContent = '保存成绩';
  modal.classList.add('hidden');
}
```
Need `resetGame`:
```
function resetGame() {
  resetGameState();
  overlay.classList.remove('hidden');
}
```
Need if overlay currently hidden? okay.

`startGame`:
```
function startGame() {
  resetGameState();
  overlay.classList.add('hidden');
  state = 'playing';
  startTime = performance.now();
  lastTime = performance.now();
}
```
Need if startBtn click while state playing? overlay hidden, not possible. Good.

`update`:
```
function update(dt) {
  if (state === 'playing') {
    elapsed = (performance.now() - startTime) / 1000;
    timerDisplay.textContent = elapsed.toFixed(2);
  }
  const { speed } = getAimVector();
  speedDisplay.textContent = '速度：' + Math.round(speed);
  updateArrows(dt);
}
```
Need if state finished, modal open, speedDisplay updates behind. okay.

`updateArrows`:
```
function updateArrows(dt) {
  const STEPS = 8;
  for (let i = arrows.length - 1; i >= 0; i--) {
    const a = arrows[i];
    if (a.stuck) continue;
    const subDt = dt / STEPS;
    for (let step = 0; step < STEPS; step++) {
      if (a.stuck) break;
      a.x += a.vx * subDt;
      a.y += a.vy * subDt;
      a.vy += GRAVITY * subDt;
      a.angle = Math.atan2(a.vy, a.vx);
      // target collisions
      for (const t of targets) {
        if (a.stuck) break;
        const d = Math.hypot(a.x - t.x, a.y - t.y);
        if (d <= t.radius) {
          a.stuck = true;
          if (!t.hit) {
            t.hit = true;
            hitTargets.add(t.id);
            hitCountDisplay.textContent = hitTargets.size;
            if (hitTargets.size === targets.length) {
              finishGame();
            }
          }
          break;
        }
      }
      // bounds
      if (!a.stuck) {
        if (a.y >= GROUND_Y) {
          a.y = GROUND_Y;
          a.stuck = true;
          a.angle = Math.atan2(a.vy, a.vx);
        } else if (a.x < -50 || a.x > W + 50 || a.y < -100) {
          arrows.splice(i, 1);
          break;
        }
      }
    }
  }
}
```
Potential issue: If finishGame called, state finished, but if target hit and t.hit true, no finish. Good.
If arrow goes out after hitting? no.
If arrow stuck at ground, it remains. Could accumulate many arrows on ground, maybe okay. But if too many, performance? no.

Need if arrow is stuck at target, draw; no more update. Good.

Need `finishGame`:
```
function finishGame() {
  state = 'finished';
  elapsed = (performance.now() - startTime) / 1000;
  timerDisplay.textContent = elapsed.toFixed(2);
  finalTimeValue = Math.round(elapsed * 100) / 100;
  finalTimeDisplay.textContent = finalTimeValue.toFixed(2);
  const records = loadRecords();
  const t = finalTimeValue;
  let previewRank = 1;
  for (const r of records) {
    if (r.time <= t) previewRank++;
    else break;
  }
  rankPreview.textContent = `当前成绩可进入第 ${previewRank} 名（保存后更新）`;
  playerNameInput.value = '射手' + (records.length + 1);
  saveBtn.disabled = false;
  saveBtn.textContent = '保存成绩';
  modal.classList.remove('hidden');
  playerNameInput.focus();
}
```
Need if records not sorted? loadRecords sorts. Good.

Need `saveRecord`:
```
function saveRecord() {
  if (finalTimeValue === null || saveBtn.disabled) return;
  const name = playerNameInput.value.trim() || '匿名射手';
  const records = loadRecords();
  const record = {
    name: name,
    time: finalTimeValue,
    date: new Date().toLocaleString('zh-CN', { hour12: false })
  };
  records.push(record);
  records.sort((a,b) => a.time - b.time || String(a.date).localeCompare(String(b.date)));
  const rank = records.indexOf(record) + 1;
  saveRecords(records.slice(0, 50));
  saveBtn.disabled = true;
  saveBtn.textContent = '已保存 ✓';
  rankPreview.textContent = `已保存！当前排名：第 ${rank} 名`;
  renderLeaderboard();
}
```
Need if rank >50 but record sliced out? rank from full. okay.
Need if `String(a.date).localeCompare` if date undefined. okay.
Need if existing record same date and time, indexOf record? records.indexOf is reference same. Good.

Need `loadRecords`:
```
function loadRecords() {
  try {
    const arr = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
    if (!Array.isArray(arr)) return [];
    arr.sort((a,b) => a.time - b.time);
    return arr;
  } catch {
    return [];
  }
}
```
Need `saveRecords`:
```
function saveRecords(records) {
  try { localStorage.setItem(STORAGE_KEY, JSON.stringify(records)); } catch(e) {}
}
```

Need `renderLeaderboard`:
```
function renderLeaderboard() {
  const records = loadRecords();
  const top = records.slice(0, 10);
  if (top.length === 0) {
    leaderboardBody.innerHTML = '<tr><td colspan="4" class="empty">暂无记录，快来创造纪录吧！</td></tr>';
    return;
  }
  const medals = ['🥇','🥈','🥉'];
  leaderboardBody.innerHTML = top.map((r, i) => {
    const rankText = i < 3 ? medals[i] : '#' + (i+1);
    return `<tr>
      <td>${rankText}</td>
      <td>${escapeHtml(r.name)}</td>
      <td>${Number(r.time).toFixed(2)}s</td>
      <td>${escapeHtml(r.date || '')}</td>
    </tr>`;
  }).join('');
}
```
Need `escapeHtml` before. Good.

Need `draw`:
```
function draw() {
  ctx.clearRect(0,0,W,H);
  drawSky();
  drawGround();
  targets.forEach(drawTarget);
  drawBow();
  drawNockedArrow();
  if (state === 'playing') {
    drawTrajectory();
    drawMouseHint();
  }
  arrows.forEach(drawArrow);
}
```
Need if state idle, arrows array empty. okay.
Need *active arrows after trajectory* so they appear on top. Good.
Need drawTarget before bows; arrows after. Good.

Need `drawSky`:
```
function drawSky() {
  const grd = ctx.createLinearGradient(0,0,0,GROUND_Y);
  grd.addColorStop(0,'#3a7bd5');
  grd.addColorStop(1,'#b8e1f9');
  ctx.fillStyle = grd;
  ctx.fillRect(0,0,W,H);
  ctx.fillStyle = '#ffe680';
  ctx.beginPath();
  ctx.arc(960, 80, 38, 0, Math.PI*2);
  ctx.fill();
  ctx.fillStyle = 'rgba(255,255,255,0.75)';
  ctx.beginPath();
  ctx.arc(170, 110, 26, 0, Math.PI*2);
  ctx.arc(205, 100, 32, 0, Math.PI*2);
  ctx.arc(235, 112, 24, 0, Math.PI*2);
  ctx.fill();
  ctx.beginPath();
  ctx.arc(700, 150, 22, 0, Math.PI*2);
  ctx.arc(730, 140, 28, 0, Math.PI*2);
  ctx.arc(758, 152, 20, 0, Math.PI*2);
  ctx.fill();
}
```
Need if multiple arcs in same path, fill will connect? The arcs in same path can create weird connecting lines. Better use separate `ctx.fill()` per cloud or use `ctx.moveTo` to avoid connecting? Simpler:
```
function cloud(x,y,size) {
  ctx.beginPath();
  ctx.arc(x,y,size*0.8,0,2PI);
  ctx.arc(x+size*0.7,y-size*0.1,size,0,2PI);
  ctx.arc(x+size*1.3,y+size*0.1,size*0.8,0,2PI);
  ctx.fill();
}
```
Still arcs in same path can connect but okay. Could use separate `fill` per arc:
```
ctx.beginPath(); ctx.arc(170,110,26,0,2PI); ctx.fill();
ctx.beginPath(); ctx.arc(205,100,32,0,2PI); ctx.fill();
...
```
Do that.

Need `drawGround`:
```
function drawGround() {
  ctx.fillStyle = '#6baa4e';
  ctx.fillRect(0, GROUND_Y, W, H - GROUND_Y);
  ctx.fillStyle = '#87c96f';
  ctx.fillRect(0, GROUND_Y, W, 12);
  ctx.fillStyle = '#4d8a39';
  for (let i=0; i<W; i+=24) {
    ctx.beginPath();
    ctx.moveTo(i, GROUND_Y+8);
    ctx.lineTo(i+6, GROUND_Y-6);
    ctx.lineTo(i+12, GROUND_Y+8);
    ctx.fill();
  }
}
```
Need small grass triangles okay. But loop over 1100/24 ~46, fine. Could draw before target poles. DrawGround after sky.

Need `drawTarget`:
```
function drawTarget(t) {
  const cx=t.x, cy=t.y, r=t.radius;
  // pole
  ctx.strokeStyle = '#6d4c2f';
  ctx.lineWidth = 10;
  ctx.beginPath();
  ctx.moveTo(cx, cy + r - 6);
  ctx.lineTo(cx, GROUND_Y);
  ctx.stroke();
  ctx.fillStyle = '#4a2f1a';
  ctx.fillRect(cx - 30, GROUND_Y - 8, 60, 14);
  // target face shadow
  ctx.beginPath();
  ctx.arc(cx+3, cy+3, r, 0, Math.PI*2);
  ctx.fillStyle = 'rgba(0,0,0,0.18)';
  ctx.fill();
  // rings
  const colors = ['#f7f2ea','#333333','#1f6fd9','#e03a3a','#f6c945'];
  const radii = [r, r*0.78, r*0.56, r*0.36, r*0.18];
  for (let i=0;i<colors.length;i++) {
    ctx.beginPath();
    ctx.arc(cx,cy,radii[i],0,Math.PI*2);
    ctx.fillStyle = colors[i];
    ctx.fill();
    ctx.strokeStyle = 'rgba(0,0,0,0.25)';
    ctx.lineWidth = 1.5;
    ctx.stroke();
  }
  // label
  ctx.font = 'bold 16px sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.lineWidth = 4;
  ctx.strokeStyle = 'rgba(0,0,0,0.6)';
  ctx.strokeText('T' + t.id, cx, cy - r - 16);
  ctx.fillStyle = '#ffffff';
  ctx.fillText('T' + t.id, cx, cy - r - 16);
  // hit overlay
  if (t.hit) {
    ctx.beginPath();
    ctx.arc(cx, cy, r + 6, 0, Math.PI*2);
    ctx.fillStyle = 'rgba(46,204,113,0.25)';
    ctx.fill();
    ctx.strokeStyle = '#2ecc71';
    ctx.lineWidth = 5;
    ctx.stroke();
    ctx.font = 'bold 52px sans-serif';
    ctx.fillStyle = '#2ecc71';
    ctx.fillText('✓', cx, cy + 2);
  }
}
```
Need if label T4 at y-r-16=254; okay. If hit green overlay, check at center. Good.
Need `ctx.strokeText` with lineWidth changed; okay.

Need `drawBow`:
```
function drawBow() {
  const bx=BOW.x, by=BOW.y;
  ctx.lineCap = 'round';
  // limbs
  ctx.strokeStyle = '#7a4a2b';
  ctx.lineWidth = 7;
  ctx.beginPath();
  ctx.moveTo(bx, by - 80);
  ctx.quadraticCurveTo(bx + 52, by - 25, bx + 45, by);
  ctx.quadraticCurveTo(bx + 52, by + 25, bx, by + 80);
  ctx.stroke();
  // string
  ctx.strokeStyle = '#e8e8e8';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(bx, by - 80);
  ctx.lineTo(bx - 8, by);
  ctx.lineTo(bx, by + 80);
  ctx.stroke();
  // grip
  ctx.fillStyle = '#4a2f1a';
  ctx.fillRect(bx - 8, by - 6, 18, 12);
}
```
Need "quadraticCurveTo" second from mid to bottom maybe okay. Bow curve to right; string behind left. Good.
Need after lineCap, if no reset not issue. But if target drawing lineCap default maybe; not.

Need `drawNockedArrow`:
```
function drawNockedArrow() {
  if (state === 'finished') return;
  const {vx, vy} = getAimVector();
  const angle = Math.atan2(vy, vx);
  ctx.save();
  ctx.translate(BOW.x, BOW.y);
  ctx.rotate(angle);
  ctx.lineCap = 'butt';
  ctx.strokeStyle = '#a4713b';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(-32, 0);
  ctx.lineTo(4, 0);
  ctx.stroke();
  ctx.fillStyle = '#d35400';
  ctx.beginPath();
  ctx.moveTo(10, 0);
  ctx.lineTo(0, -4);
  ctx.lineTo(0, 4);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```
Need if state idle, drawNockedArrow draws; okay. But if state finished, no loaded arrow. Fine.

Need `drawTrajectory`:
```
function drawTrajectory() {
  const {vx, vy} = getAimVector();
  let x = SHOT_ORIGIN.x;
  let y = SHOT_ORIGIN.y;
  let vv = vy;
  const dt = 1/30;
  ctx.save();
  ctx.strokeStyle = 'rgba(255,255,255,0.85)';
  ctx.lineWidth = 2.5;
  ctx.setLineDash([6, 10]);
  ctx.beginPath();
  ctx.moveTo(x, y);
  for (let i=0; i<80; i++) {
    x += vx * dt;
    y += vv * dt;
    vv += GRAVITY * dt;
    if (y >= GROUND_Y || x < -20 || x > W + 20) break;
    ctx.lineTo(x, y);
  }
  ctx.stroke();
  ctx.restore();
}
```
Need if vv/vy changes. Good.
Need trajectory preview should account for SHOT_ORIGIN if same as BOW. okay.
Need if arrow hits target, dashed line goes through target, okay.

Need `drawMouseHint`:
```
function drawMouseHint() {
  ctx.save();
  ctx.strokeStyle = 'rgba(255,255,255,0.9)';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.arc(mouse.x, mouse.y, 9, 0, Math.PI*2);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(mouse.x - 15, mouse.y);
  ctx.lineTo(mouse.x + 15, mouse.y);
  ctx.moveTo(mouse.x, mouse.y - 15);
  ctx.lineTo(mouse.x, mouse.y + 15);
  ctx.stroke();
  ctx.restore();
}
```
Need if mouse outside canvas, crosshair may be partly outside but canvas clips. okay.

Need "drawArrow" function:
```
function drawArrow(a) {
  ctx.save();
  ctx.translate(a.x, a.y);
  ctx.rotate(a.angle);
  ctx.lineCap='butt';
  ctx.strokeStyle = '#9c6b3c';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(-30,0);
  ctx.lineTo(4,0);
  ctx.stroke();
  ctx.fillStyle = '#d35400';
  ctx.beginPath();
  ctx.moveTo(10,0);
  ctx.lineTo(-1,-5);
  ctx.lineTo(-1,5);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.moveTo(-30,-4);
  ctx.lineTo(-21,0);
  ctx.lineTo(-30,4);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```
Need arrow tip at 10, collision at a.x. okay.
Need maybe line from -30 to4 then head from -1 to10; shaft overlaps head base; okay.

Need "event listeners":
```
canvas.addEventListener('mousemove', ...);
canvas.addEventListener('click', ...);
startBtn.addEventListener('click', startGame);
resetBtn.addEventListener('click', resetGame);
saveBtn.addEventListener('click', saveRecord);
againBtn.addEventListener('click', startGame);
closeModalBtn.addEventListener('click', hideModal);
playerNameInput.addEventListener('keydown', e => { if (e.key === 'Enter') saveBtn.click(); });
renderLeaderboard();
requestAnimationFrame(loop);
```
Need if `againBtn` exists. yes.

Need "mouse click" function:
```
canvas.addEventListener('click', (e) => {
  if (state !== 'playing') return;
  const {vx, vy} = getAimVector();
  arrows.push({
    x: SHOT_ORIGIN.x,
    y: SHOT_ORIGIN.y,
    vx,
    vy,
    angle: Math.atan2(vy, vx),
    stuck: false
  });
  arrowCount++;
  arrowCountDisplay.textContent = arrowCount;
});
```
Need if click after shooting but state finished? no. Good.
Need if click on canvas while state idle? no.

Need "mousemove" mapping:
```
canvas.addEventListener('mousemove', (e) => {
  const rect = canvas.getBoundingClientRect();
  mouse.x = (e.clientX - rect.left) * (W / rect.width);
  mouse.y = (e.clientY - rect.top) * (H / rect.height);
});
```
Need if rect.width 0? no. Good.
Need if CSS width 100% and rect width not equal W, mapping works.

Need "startBtn" maybe if overlay has start button; startGame hides overlay and starts. good.

Need "modal.close button" if user closes and later click reset, modal hidden? resetGameState hides. Good.

Need "resetBtn" if state idle and overlay visible, timer resets and overlay remains. okay.

Need "hideModal" function:
```
function hideModal() {
  modal.classList.add('hidden');
}
```
Need `modal` defined. Good.

Need "overlay" hidden class CSS. Good.

Need "finalTimeDisplay" id. Good.

Need "rankPreview" id. Good.

Need "再次一局" button id "againBtn". Good.

Need "closeModalBtn" button.

Need maybe "saveBtn.disabled" if finalTimeValue null and user click save via Enter? finalTimeValue null, but modal only after finish. okay.

Need "on load state idle, overlay visible, draw current trajectory? Actually if state idle, drawTrajectory not drawn, so no dashed line until start. That's fine. But nocked arrow drawn. Once start, visible.

Need "overlay start instructions" says "白色虚线为预计弹道"—only after start. okay.

Need "maybe if user clicks reset, overlay visible and state idle; no trajectory. okay."

Need "test instructions" perhaps shouldn't include "open console" maybe okay.

Let's think about "targets hit" if arrow hits target but doesn't stop? We set a.stuck=true, so arrow stops at first detected point within radius. But arrow might stop at edge of target; draw arrow at edge with head outwards. If distance <= radius, and a.x is near target edge, arrow will be drawn partly outside? It's okay. For visual, maybe arrow stops at edge. But if target not hit and arrow stuck, target mark. Good.

Need "arrow hit target center" if arrow passes through center, a.stuck at center? Actually if first substep within radius maybe at edge, not center. After update, if edge, stop. It may not penetrate center. okay.

Need "target hit detection only checks arrow point at a.x; since a.x maybe body not tip. But tip at x+10; distance less, so if body within radius, tip may be further inside/outside depending travel direction. The arrowhead tip is at a.x+10; if a.x is at target edge, tip is 10 px further in? Actually arrow angle from velocity, so tip is ahead. If a.x at distance 50 from center (body), tip at distance 40 maybe inside. The target should have been hit earlier. But stopping body at edge is okay. 

Need maybe "a.angle" calculation after adding vy before collision. good.

Need "if arrow hits ground, `a.angle = Math.atan2(a.vy, a.vx)`; but `a.vy` already includes gravity. good.
Need "if arrow goes out x > W+50, splice; if loop continues maybe index issue? We use reverse loop and splice; after splice, `break` from inner? If out-of-bounds occurs inside substep, we do:
```
} else if (a.x < -50 || a.x > W + 50 || a.y < -100) {
  arrows.splice(i, 1);
  break;
}
```
Break inner loop, then after inner loop, loop continues with next arrow. Since `i` after splice is already removed; reverse loop continues, no issue. But if a.stuck? We skip. Good.
Need if arrow hit target and finishGame, target hit maybe. Good.

Need "if arrow hits target and hitTargets size reaches 4, finishGame opens modal. But updateArrows continues; maybe another arrow in loop could hit another target? But all targets are hit. no.

Need "drawTarget" if t.hit, green check; if arrow stuck, arrow drawn after. Good.

Need "leaderboard empty state" colspan=4. Good.

Need "maybe `rankPreview` should be in modal; if no save and close, no leaderboard update. good.

Need "test instructions" "在浏览器地址栏打开 index.html" maybe.

Need "输出要求" perhaps wants "代码" not explanation. But final needs instructions. We'll present code then run/test. The final answer can be long. Desired oververbosity 5, okay.

Need perhaps "do not view other files" perhaps "没有引用外部文件、图片、框架" in final. Good.

Need "Must not use analysis" actually final only. Good.

Let's consider if there is any issue with "start time" after `startGame`: `resetGameState()` sets state idle, hides modal, clears arrows. `lastTime = performance.now()` may be slightly after loop's previous lastTime. Then `overlay.classList.add('hidden')`, `state='playing'`, `startTime=performance.now()`. The first update dt = (now - lastTime)/1000. Since lastTime set before startTime, dt maybe 0. If first RAF after a few ms, dt includes starting time; okay. Timer starts at startTime. Good.

Need "if user clicks start overlay and immediately click canvas? start button event, overlay hidden, state playing; next click shoots. okay.

Need "canvas cursor crosshair". CSS.

Need "maybe use `e.button` no; click any button. okay.

Need "file name" maybe not.

Need "leaderboard date" maybe table with long date; can be wide. Use `font-size:14px`. Fine.

Need "responsive leaderboard" maybe table width.

Need "possible issue with `toLocaleString('zh-CN', { hour12: false })` on some browsers returns e.g. "2025/4/1 12:34:56". good.

Need "escapeHtml" function:
```
function escapeHtml(value) {
  return String(value).replace(/[&<>"']/g, function(c) {
    return { '&':'&amp;', '<':'&lt;', '>':'&gt;', '"':'&quot;', "'":'&#39;' }[c];
  });
}
```
Need if value not string, String.

Need "renderLeaderboard" uses `Number(r.time).toFixed(2)`; if r.time undefined? loadRecords from old maybe; okay. Could sanitize:
```
const time = Number(r.time);
if (isNaN(time)) return '';
```
But not necessary. Could include for robust:
```
const t = Number(r.time);
...
<td>${isFinite(t) ? t.toFixed(2) : '--'}s</td>
```
Need in template with escape. Could implement. Let's do robust:
```
const time = Number(r.time);
...
`<td>${Number.isFinite(time) ? time.toFixed(2) : '--'}s</td>`
```
Need `Number.isFinite` works. Good.

Need "date" maybe undefined.

Need "computeRankAfterAdd" if records with invalid times maybe. loadRecords sort could error if r.time undefined. Probably no. Could robust:
```
arr.sort((a,b)=> (Number(a.time)||0) - (Number(b.time)||0));
```
Use that.
Need in `saveRecord`, `record.time=finalTimeValue`; sort. Good.

Need "rank" if records sorted and record inserted; `records.indexOf(record)` works even if duplicate object? yes.

Need "finalTimeValue" rounded; preview rank uses rounded. Save uses same rounded. Good.

Need "time display" `elapsed` may be 12.3456; finalTimeValue rounded 12.35. Good.

Need "if state playing and target hit all, finishGame called within updateArrows; after finish, modal opens. But `update` continues after updateArrows; update already set timer before updateArrows, okay. `finishGame` sets elapsed using performance.now() maybe a few ms later than timer; fine.
Need if updateArrows processes multiple arrows in same frame and target hit all, state finished. The outer loop still has arrows maybe; no issue.

Need "maybe if user save after finish, state remains 'finished'. If user closes modal, state finished. If user clicks reset, resetGame sets state idle. Good.

Need "againBtn" startGame: It hides overlay and starts a new round immediately. Good.
Need "closeModalBtn" doesn't reset; if closed and user clicks reset, okay.

Need "if user clicks save, leaderboard updated; modal remains; if user clicks close, sees leaderboard. Good.

Need "ranking before save preview" if records sorted ascending, count <= current time +1. If there are two records with same time, new rank after them. okay.
Need "after save, rank updated" if current not in top50? records slice top50 but rank computed before slice. If rank >50, not stored. But impossible for typical. okay.

Need "maybe localStorage key with Chinese? not. okay.

Need "targets" const but `hit` property reset. Good.

Let's maybe craft "run instructions" in final:
```
运行方式：
1. 将上面的完整代码复制保存为 index.html。
2. 用现代浏览器（Chrome/Edge/Firefox）打开该文件。
3. 点击“开始游戏”，移动鼠标瞄准，点击画布发箭；白色虚线为弹道预览，鼠标离弓箭手越远力度越大。
4. 命中全部4个靶子后，输入昵称保存成绩；排行榜按用时升序排列。

测试说明：
- 手动冒烟：开始后计时器应增加；每命中一个靶子“命中”数 +1；靶子变绿；4靶全中后弹出成绩框。
- 排行榜：保存成绩后表格应出现该记录，并用时最短排最前；刷新页面后记录仍保留。
- 边界测试：点击“重置”可停止计时并回到准备界面；重复开始/重置不会报错。
```
Need maybe "不要查看其他文件" in final "本游戏为单文件，无外部依赖". Good.

Need now write final code. Let's double-check HTML/CSS for IDs.

HTML:
```
<div class="game-area">
  <canvas id="gameCanvas" width="1100" height="650"></canvas>
  <div id="overlay" class="overlay">
    <div class="overlay-content">
      <h2>🏹 准备射击</h2>
      <p>移动鼠标瞄准，点击发射</p>
      <p>鼠标离弓越远，箭速越快</p>
      <button id="startBtn" class="btn">开始游戏</button>
    </div>
  </div>
</div>
```
No `hidden` initially. Overlay visible. In CSS `.overlay` visible by default; `.overlay.hidden {display:none}`. okay.

Info bar:
```
<div class="info-bar">
  <div>⏱ 用时：<span id="timerDisplay">0.00</span>s</div>
  <div>🎯 命中：<span id="hitCount">0</span>/4</div>
  <div>🏹 已射：<span id="arrowCount">0</span>箭</div>
  <div id="speedDisplay">速度：0</div>
  <button id="resetBtn" class="btn small">重置</button>
</div>
```
Need "hitCountDisplay" id hitCount. But id hitCount also used? no.

Leaderboard:
```
<div class="leaderboard">
 <h2>🏆 排行榜</h2>
 <table>
  <thead><tr><th>名次</th><th>玩家</th><th>用时</th><th>日期</th></tr></thead>
  <tbody id="leaderboardBody"></tbody>
 </table>
</div>
```
Modal:
```
<div id="modal" class="modal hidden">
 <div class="modal-content">
  <h2>🎯 全部命中！</h2>
  <p>用时：<span id="finalTime">0.00</span> 秒</p>
  <p id="rankPreview"></p>
  <input id="playerName" type="text" maxlength="10" placeholder="输入昵称">
  <div class="modal-buttons">
    <button id="saveBtn" class="btn">保存成绩</button>
    <button id="againBtn" class="btn">再来一局</button>
    <button id="closeModalBtn" class="btn gray">关闭</button>
  </div>
 </div>
</div>
```
Need CSS `.modal-buttons` flex gap. .btn.gray maybe.
Need `rankPreview` maybe empty initially.

Need CSS:
```
.btn.gray { background:#7f8c8d; color:#fff; }
.btn.gray:hover { background:#95a5a6; }
```
Need `button:disabled { opacity:0.7; cursor:not-allowed; }`.

Need "body" font.

Potential issue: overlay `position:absolute; top:0; left:50%; transform:translateX(-50%);` If game-area width is 100%, overlay left 50% transforms -50% relative to game-area width; good. But if canvas has aspect ratio and game-area max-width, overlay height 100%; canvas height auto. The game-area height is only as tall as canvas? Since canvas is child, overlay position absolute height 100% uses game-area height? game-area height should match canvas because canvas block contributes. Good.
Need "overlay-content" if overlay size too small maybe. okay.

Need "canvas width:100%" and height auto, but canvas attribute 650; if CSS width 100%, height auto; good. Need `.game-area { display:flex; justify-content:center; }` but canvas width 100%; game-area width 100%; okay. Overlay width 100%. Need "position:relative" on game-area. Good.

Need "leaderboard max-width 800". okay.

Need perhaps "body display flex center" and container min-height. okay.

Need "footer"? no.

Need "target label if target y=320, label y=256; above target. fine.

Need "drawSky cloud" if cloud x positions not affected by canvas scaling. okay.

Potential issue: "ctx.drawImage" no.

Need "look of target T4 at x=930; sun at 960,80; no issue.

Need "arrowCountDisplay" with "箭" text perhaps "已射：0 箭". okay.

Need "speedDisplay" at info bar with "速度：0" no unit. okay.

Need "test instructions" maybe mention "你可以在画布中移动鼠标，观察白色虚线的落点来调整瞄准。" Good.

Let's think about "simulation game" if "gravity" is in px/s², but trajectory preview uses dt with vv variable; if arrow gets stuck at target, no. Good.

Need maybe "target hit if arrow point within target radius" and "hitTargets.size" after reset. Good.

Potential issue with "getAimVector" when mouse is exactly at BOW: dx=0, then set dx=30; dy=0, speed=120; angle=0. good.
Potential issue if mouse.x from canvas coordinate is not updated before first click? If user clicks without moving? mouse default (700,300). fine.

Need "mousemove" if mouse leaves canvas and clicks? no click if outside. okay.

Need "canvas click" if after start and user clicks on canvas, arrowCount increments. If user clicks start button, no arrow; because start button not canvas. Good.

Need "overlay" maybe after reset if state idle and overlay visible, canvas click no. Good.

Need "renderLeaderboard" after save maybe current record in table. If rank >10, table doesn't show current, but rank text shows. okay.

Need "finalTimeValue" if no save and reset, null. Good.

Need "saveButton disabled" after save but if user clicks "再来一局", startGame reset and re-enables. Good.

Need "modal hidden class" initial in HTML. `.hidden { display: none !important; }` handles overlay and modal. Good.
Need if overlay visible and modal hidden. okay.

Need "body background" maybe "linear-gradient(135deg, #0f2b3b, #2d5f6e)". Good.

Let's ensure no mistakes in variable names:
- `hitCountDisplay` from `getElementById('hitCount')`. In info HTML span id="hitCount". Good.
- `finalTimeDisplay` from `finalTime`. Good.
- `rankPreview` from `rankPreview`. Good.
- `playerNameInput` from `playerName`. Good.
- `againBtn` from `againBtn`. Good.
- `closeModalBtn` from `closeModalBtn`. Good.

Need "saveBtn.disabled" maybe in finishGame set false. If user saves and then closes and clicks "保存成绩" impossible because disabled. Good.

Need "if user presses Enter in name input after save disabled, no duplicate. Good.

Need "if modal closed before save and state finished, then if user somehow opens modal? no. okay.

Need maybe "rank preview before save" if records list top10 but loadRecords has all. okay.

Potential issue with `loadRecords` sorting mutates array from localStorage but no issue. But if `records` from localStorage unsorted and contains invalid entries, sort may compare undefined. Use robust:
```
return arr.filter(r => r && typeof r.time === 'number').sort((a,b)=>a.time-b.time);
```
Need but if old records time string? Could handle:
```
const num = Number(r.time);
if (!Number.isNaN(num)) r.time = num;
return arr.filter(...).sort(...);
```
Let's implement robust.

```
function loadRecords() {
  try {
    const data = localStorage.getItem(STORAGE_KEY);
    if (!data) return [];
    const arr = JSON.parse(data);
    if (!Array.isArray(arr)) return [];
    return arr
      .filter(r => r && r.time !== undefined)
      .map(r => ({ name: String(r.name || '匿名射手'), time: Number(r.time), date: r.date || '' }))
      .sort((a,b) => a.time - b.time);
  } catch {
    return [];
  }
}
```
Need if `Number(r.time)` NaN, filter? Could filter `Number.isFinite`.
```
.filter(r => r && Number.isFinite(Number(r.time)))
```
Use. Good.
`saveRecord` name etc. good.

Need `renderLeaderboard` top10, time = r.time already number. okay.

Need "saveRecords" store objects. Good.

Need "escapeHtml" for rank? no.

Need "leaderboard date" if empty.

Need "rank preview before save" uses loadRecords maps, fine.

Need "date locale" if no date? In save we add. Good.

Need "playerNameInput value" `'射手' + (records.length + 1)` with records length maybe top? loadRecords all. Good.

Need "modal if user enters no name save as 匿名射手. Good.

Need "current record" after save and `saveRecords(records.slice(0,50))`; if record outside top50, not stored but rank computed. okay.

Need "Button text" after save `已保存 ✓`. If reset, resetGameState resets.
Need "rankPreview" after save "已保存！当前排名：第 X 名". If user started new game, modal hidden. okay.

Need "ctx.save/restore" in drawTrajectory resets line dash. Good.

Need "drawSky" gradient from 0 to GROUND_Y; if canvas H=650, groundY=580, fill bottom. good.

Need "target poles" perhaps target4 pole long, but if target's bottom (y+50-6=364) to ground, good. The pole line at x=930; no issue.
Need "target stand base" if x=930, fillRect x-30 to x+30. okay.

Need "drawTarget label" textStroke might use `lineWidth=4`; after draw, lineWidth changed; no issue.

Need "drawBow" maybe string drawn behind arrow. good.

Need "drawNockedArrow" if state idle draws arrow at bow, but overlay maybe; okay.
Need "if state finished, nocked arrow not drawn" but if arrows still stuck, active arrows drawn. If active arrow not stuck and state finished, updateArrows still runs, so they land. okay.

Need "Maybe after finish, active arrows still fly; on hitting target, target already hit so no finish. okay.

Need "If player clicks canvas while state finished, no new arrow. Good.

Need "Should we stop timer at final hit rather than modal? finalTime computed on hit. okay.

Need "Timer starts on start click, not on first arrow." Good. User might spend time aiming; time counts. That's okay.

Need "resetBtn" if clicked while playing: `resetGame()` -> `resetGameState()` sets state idle, clears arrows, hides modal, shows overlay. If user then clicks start, startGame. Good.
Need "resetBtn" if clicked while state idle and overlay visible: resetGameState, overlay remove hidden. okay.

Need "If user closes modal after finish and clicks reset button (visible behind modal? Modal overlay covers entire screen, so not clickable until close. If close, reset clickable. Then resetGame. okay.)

Need "Responsive" canvas width 100%. Overlay content if width small maybe. okay.

Let's now think about "output requirements" "请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" The final should include the code in a code block. The answer may be long but acceptable.

Need maybe "不要查看其他文件" means no "checking other files" not "in final no references". okay.

Let's maybe include "已创建" in final. Good.

Need "code block language HTML" with entire file. Good.

Potential issue: The final response desired oververbosity 5, but code is large. That's okay.

Let's verify CSS class names.

Full HTML with CSS:

```
<style>
* { box-sizing: border-box; }
body { margin:0; font-family:...; background:...; min-height:100vh; display:flex; justify-content:center; align-items:center; }
.container { max-width:1100px; width:100%; padding:20px; }
h1 { text-align:center; color:#ffd966; text-shadow:0 2px 8px rgba(0,0,0,.4); }
.subtitle { text-align:center; color:#dce9ef; margin-bottom:16px; }
.game-area { position:relative; width:100%; max-width:1100px; margin:0 auto; }
canvas { display:block; width:100%; height:auto; border-radius:16px; box-shadow:0 8px 32px rgba(0,0,0,.45); cursor:crosshair; }
.overlay { position:absolute; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,.45); border-radius:16px; display:flex; align-items:center; justify-content:center; z-index:10; }
.overlay.hidden { display:none; }
.overlay-content { background:rgba(20,40,60,.92); border:2px solid #ffd966; border-radius:16px; padding:28px 36px; text-align:center; box-shadow:0 8px 24px rgba(0,0,0,.5); }
.overlay-content p { margin:6px 0; color:#eef7ff; }
...
```
Need if overlay left 0 rather than left 50%; Since game-area width same as canvas width, overlay left 0 width 100%. Good. Because canvas width:100%, game-area max-width same. Use left 0. Simpler. We earlier considered left 50, but left 0 fine. Need if game-area has padding? no. Good.
Need `.game-area` display:block; canvas block. Overlay covers. Good.

Need `.info-bar` with justify-content center; if container max-width 1100, info-bar width 100%. okay.

Need table:
```
.leaderboard table { width:100%; border-collapse:collapse; color:#fff; }
.leaderboard th,.leaderboard td { padding:8px 6px; border-bottom:1px solid rgba(255,255,255,.15); text-align:center; }
.leaderboard th { background:rgba(255,215,0,.15); font-weight:700; }
.empty { color:#aac; padding:20px !important; }
```
Need `thead th` maybe. okay.

Need "modal" if hidden display none. `.modal.hidden { display:none; }`. Without `.modal { display:flex; }` good. Need `.modal` fixed z-index. Good.

Need "modal-content" width. Need "modal-buttons" display flex; justify-content center; gap 10; margin-top 16; flex-wrap wrap. Good.
Need "input" style:
```
.modal input { width:100%; padding:10px; border-radius:8px; border:1px solid #ccc; font-size:16px; margin-top:10px; }
```
Need "rank-preview" maybe no.

Need "buttons" maybe `.btn:active { transform: translateY(1px); }`.

Need "button:disabled" style.

Need "small" class. Good.

Need "PlayerName input maxlength=10" but Chinese chars count as code units? okay.

Potential issue: The leaderboard table is inside `.leaderboard` but table columns not equal? okay.

Need "speedDisplay" perhaps in info bar. Good.

Let's consider if by "排行榜上的名次就越高" maybe should show "rank" in leaderboard sorted ascending. We'll show rank #1 for shortest. Good.

Need "all four targets" if user hits target with arrow but target already hit, arrows stuck; target still green. Good.

Need maybe "target hit count display" if arrow hits a target not hit, increments. If arrow hits target with t.hit true, no count. Good.

Need "if arrow hits target and `finishGame()` called, but after finish, target hit overlay may not draw until next frame; modal appears. okay.

Need "if game state idle, updateArrows none. If user resets while arrows active, arrows array cleared. Good.

Potential issue: "saveRecords(records.slice(0,50))" if current not in top50 but rank text says rank >50; if user closes and sees leaderboard top10, fine. okay.

Need "compute rank after save": We use `records.indexOf(record)`. But if records contain identical object? no. If `saveRecords` not called, no. okay.

Need "use localStorage" if file protocol returns SecurityError; try/catch. Good.

Need "No external resources" good.

Let's maybe write code now in final. Need ensure no Markdown code block broken by triple backticks inside? We'll use ```html. Inside code no triple backticks. okay.

Potential issue: "HTML has Chinese characters and emojis; ensure UTF-8." yes.

Let's maybe include comments in code? Some comments helpful. okay.

Let's evaluate if target positions feasible with our getAimVector. Need maybe target T4 y=320, x=930, BOW=125,510, diff=805,-190. If aim at mouse (850,150): dx=725, dy=-360, dist=809, vx=725, vy=-360. Time to x=930: 805/725=1.1103. y=510 + (-360*1.1103) + 150*1.2328 = 510 -399.7 +184.9 =295.2. Target center 320; distance y 24.8, within radius 50. Good. If arrow stuck at x=930 y=295, okay. T3 x=760,y=460 diff 635,-50. Aim mouse (800,300): dx=675,dy=-210, dist=707, t=635/675=0.9407, y=510 -197.5 +132.8=445.3, distance 14.7. Good. T2 x=540,y=400 diff 415,-110. Aim mouse (700,260): dx=575,dy=-250, dist=627, t=415/575=0.7217, y=510 -180.4 +78.2=407.8, distance 7.8. Good. T1 x=330,y=470 diff205,-40. Aim mouse (550,330): dx=425,dy=-180, dist=461, t=205/425=0.482, y=510 -86.8 +34.9=458.1, distance 11.9. Good. Great.

Need maybe trajectory preview if mouse at (700,260) shows path through T2; good.

Need "target4 label" maybe if x=930 and the target face center y=320, radius50, top 270, label y=256. Good.

Potential issue: The `SHOT_ORIGIN` equals BOW, but bow is at (125,510), target T1 start y at 510; arrow launch from bow center, okay. The trajectory preview starts at bow. Good.

Need "bow string" line at x=117 etc; arrow drawn rotated from BOW. Good.

Need "drawNockedArrow" draws arrow over bow, but bow limbs and string maybe visible; arrow extends to mouse direction. okay.

Need "drawTrajectory" only if state playing; if user clicks start and immediately moves mouse, white line. Good.

Need "If user clicks canvas, arrow starts at bow, but if arrow is drawn at BOW and nocked arrow also drawn, fine. If multiple arrows, nocked arrow always. okay.

Potential issue: "canvas click" event and "mousemove" event should be on canvas, but overlay has `position:absolute` with `pointer-events:auto` by default. When overlay visible, canvas won't get clicks. When overlay hidden, pointer-events? Overlay display none, so canvas receives. good.
Need "resetBtn" not overlay. good.

Need "startBtn" could be clicked after start? overlay hidden. good.

Need "because overlay is sibling after canvas, it covers canvas; when hidden display none, no pointer events." yes.

Need "gray button" CSS. good.

Let's now craft final answer.

Need maybe "完整代码" before code block. Use "已完成，下面是单文件 HTML 游戏". Then code.

Need "运行方式" and "测试说明" after code. Good.

Let's ensure final code "does not view other files" no external dependencies. Good.

Let's write final code.

One more thought: We use `playerNameInput.focus()` when modal opens. On some browsers, focusing after hidden display none? We remove hidden first, then focus. Good. But `startGame` resets modal hidden and start state. Good.

Need maybe if user opens modal and presses Enter before save, `saveBtn.click()` triggers. Good.

Potential issue: `saveBtn.disabled = false` in resetGameState, but if saveBtn text after reset? yes.
Need "closeModalBtn" is `.btn.gray"; no disabled.
Need "startBtn" maybe in overlay and `resetBtn` maybe same state? okay.

Let's verify code in actual sequence:

At top:
```
const overlay = document.getElementById('overlay');
const startBtn = document.getElementById('startBtn');
...
```
Script is at end of body after elements. Good.

Need "requestAnimationFrame(loop)" after event listeners. `lastTime` set before. Good.

Need "drawSky" uses `GROUND_Y`, `W`, `H`. okay.

Need "drawGround" uses `W`. okay.

Need "updateArrows" uses `arrows`, `targets`, `hitTargets`, `finishGame`. okay.

Need "finishGame" uses `loadRecords`, `finalTimeValue`, etc. okay.

Need "saveRecord" uses `saveRecords`, `renderLeaderboard`. okay.

Need "resetGameState" uses `targets`, `hitTargets`, `arrowCount`, etc. okay.

Need "startGame" uses `lastTime`, `startTime`. okay.

Need "canvas click" uses `SHOT_ORIGIN`. okay.

Need "draw" uses `arrows`; if state finished and modal open, draw still every frame. okay.

Potential issue with `arrows.splice(i, 1)` inside reverse loop while active arrow out-of-bounds. Suppose `i` is index; after splice, outer loop decrements to i-1, good. But if a previous arrow at index >i? Since reverse loop from high to low, after splice lower index unaffected. Good.
If arrow stuck, no splice. okay.

Potential issue: With `a.stuck = true` on target, then inner target loop `break;` and then bounds check skipped due `if(!a.stuck)`. Good.
If arrow hits target, `a.vy` already updated; `a.angle` is from velocity before hit. okay.

Potential issue: "If arrow hits ground, a.stuck true; if a.x out of bounds later, not checked. Fine.

Need "target hit overlay" maybe if target hit at y=320, check "✓" at y+2 maybe. okay.

Need "canvas click" if click with mouse over canvas but state playing; no need check if mouse is "valid aim". If mouse x < bow, dx clamped to 30, still shoots. Good.

Need "speedDisplay" maybe show `速度：120` when min. okay.

Need "leaderBoard" if records old with time decimal; okay.

Need "No external dependencies" final.

Let's write final now.