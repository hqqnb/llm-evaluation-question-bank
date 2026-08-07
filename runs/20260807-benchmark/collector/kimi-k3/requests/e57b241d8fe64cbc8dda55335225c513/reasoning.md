The user wants me to create an archery shooting simulation game. Requirements:
1. Four targets in the game
2. Player hits all targets - the shorter the time, the higher the rank on the leaderboard
3. Don't look at other files, create directly
4. Deliver a runnable product with run instructions and test instructions

I'll create an HTML5 canvas-based archery game that's self-contained in a single HTML file. This makes it easy to run - just open in a browser.

Game features:
- 4 targets at different positions
- Player aims with mouse, shoots arrows (maybe with physics - gravity affecting arrow)
- Timer starts when game starts
- Leaderboard stored in localStorage
- Rank based on completion time

Let me design the game:

1. **Game mechanics:**
   - Bow on the left side of screen
   - Player clicks and drags to aim (slingshot style) or clicks to shoot
   - Arrow physics with gravity
   - 4 targets placed at various distances/heights
   - Hit detection on targets
   - Timer from first shot (or game start) until all 4 targets hit

2. **Leaderboard:**
   - Store best times in localStorage
   - Show top 10 with player names
   - Prompt for name after completing

3. **Visual polish:**
   - Draw bow, arrow, targets with rings
   - Trajectory preview when aiming
   - Particles on hit
   - Nice background

Let me write the code:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>弓箭射击模拟</title>
<style>...</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="ui">...</div>
<script>
// game code
</script>
</body>
</html>
```

Game design details:
- Canvas 960x600
- Archer position at left (x=80, y=450)
- Aim: mousedown near archer, drag back, release to shoot (power based on drag distance)
- Or: hold mouse, aim angle follows mouse, power charges, release to fire

I'll go with drag-to-aim slingshot style: click anywhere, drag, arrow shoots opposite to drag direction with speed proportional to drag length. Actually more intuitive: drag back from bow like a slingshot. Let me do: press and hold mouse button, drag to set angle and power (the further you drag from the bow, the more power), release to shoot. Direction from drag point to bow = shoot direction.

Actually simplest intuitive: mouse position determines aim angle; hold to charge power; release to shoot. Let's do hold-and-charge:
- Mouse position relative to bow gives angle
- Hold LMB: power increases 0→100 (oscillate or clamp)
- Release: shoot arrow with that angle and power
- Show trajectory dots preview

Arrow physics: gravity = 500 px/s², power maps to speed 300-900 px/s.

Targets: 4 targets, circles with rings (white/red/blue/yellow or classic). Positions:
- (700, 150) - high far
- (800, 300) - mid far
- (650, 450) - low mid
- (880, 480) - low far

Make them slightly moving? Could add optional movement for difficulty, but keep static for fairness in timing. Maybe add slight floating motion... For a time-based leaderboard, static is fair. Let me add a subtle bob to make it interesting but still fair. Actually keep it simple - static targets.

Hit detection: arrow tip enters target radius → hit. Target falls/disappears with particle effect. Also arrows stick into ground/walls.

Timer: starts on first shot or immediately when game starts? Start on game start (first click). Display elapsed. When all 4 hit → stop, show time, ask name, save to leaderboard (localStorage, top 10).

Leaderboard panel on side or overlay showing top times.

Restart button.

Wind? Could add but keep simple. Maybe add slight wind indicator for challenge... Keep it out for fairness.

Let me also add:
- Trajectory preview dots while aiming
- Power bar
- Arrows count (unlimited)
- Sound? Could use WebAudio simple beeps. Add simple synthesized sounds for shoot/hit.

Let me write the complete code carefully.

```javascript
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const W = canvas.width = 960, H = canvas.height = 600;

const GRAVITY = 600;
const bow = { x: 90, y: 430 };

let arrows = [];       // flying arrows
let stuckArrows = [];  // stuck arrows
let particles = [];
let targets = [];
let aiming = false;
let aimAngle = 0;
let power = 0;
let chargeDir = 1;
let startTime = null;
let endTime = null;
let hits = 0;
let shots = 0;
let gameState = 'ready'; // ready, playing, done

function makeTargets() {
  targets = [
    { x: 620, y: 160, r: 34, hit: false, fall: 0 },
    { x: 800, y: 260, r: 34, hit: false, fall: 0 },
    { x: 560, y: 430, r: 34, hit: false, fall: 0 },
    { x: 840, y: 470, r: 34, hit: false, fall: 0 },
  ];
}
```

Aiming: mouse position → angle from bow to mouse. Restrict angle to reasonable range (-80° to 40° maybe, in canvas coords y down, so angle negative = up).

Actually allow any angle but clamp so you can't shoot backwards weirdly: angle between -85° (up) and 30° (down-ish), and only to the right.

Power charging: while holding, power += chargeDir * dt * speed, oscillate between 0.2 and 1, or just ramp 0→1 and stay. Oscillating adds skill. Let me oscillate (ping-pong) - adds timing skill.

Hmm, but that could be frustrating. Let me do ramp 0→1 over 1.2s and clamp. Simpler and predictable. Actually ping-pong makes timing skill matter for leaderboard. I'll do ping-pong with visual power bar. Hmm, for leaderboard fairness, ping-pong is fine - everyone faces same mechanic. Let me do ping-pong but start at low power.

Actually let me reconsider: simplest satisfying mechanic = drag slingshot: mousedown anywhere, drag away from bow, a line from bow to cursor shows pull, arrow direction is from cursor toward bow extended. Power = drag distance capped. Release fires. This is very intuitive (Angry Birds style). No timing element, pure aim skill — good for speedrun leaderboard. I'll go with slingshot drag.

Implementation:
- mousedown: aiming = true, record drag start
- mousemove: drag vector = bow - mouse (pull back); power = clamp(len / maxDrag, 0, 1); angle = atan2(dragVec) — shoot direction is from mouse toward bow direction, i.e., arrow flies along (bow - mouse) normalized... Wait in slingshot, you pull back opposite to shoot direction. So shoot direction = normalize(bow - mouse). Yes: if mouse is down-left of bow, arrow flies up-right.
- Show dotted trajectory preview with current power.
- mouseup: fire arrow with speed = power * maxSpeed (min speed threshold, else cancel), aiming = false.

Angle clamp: if shoot dir points left (dir.x < 0.1), clamp or cancel. Let me clamp angle to [-80°, 60°] relative to horizontal right. Actually compute angle = atan2(dir.y, dir.x); clamp to [-1.5, 1.0] radians roughly; recompute dir.

Trajectory preview: simulate steps with gravity, draw ~15 dots.

Arrow update: pos += vel*dt; vel.y += g*dt; rotation = atan2(vel.y, vel.x). Check collision with targets: distance from arrow tip to target center < target.r → hit. Arrow tip = pos + dir * arrowLength/2. Simpler: check arrow head position.

Ground: y > groundY (e.g., 560) → stuck. Off right/bottom → stuck/remove.

Target hit: mark hit, spawn particles, play sound, hits++. fall animation: target rotates and falls, fades. When hits == 4 → endTime = now, gameState done, show overlay with time, name input, save.

Timer: starts at first mousedown (first aim) or first shot? Let's start when the first arrow is fired... or when game starts/reset. I'll start on first shot release — that gives player time to understand aim. Actually starting at game start adds urgency for leaderboard. Hmm, allow player to prepare: start timer on first shot. That's fair and standard.

Leaderboard: localStorage key 'archery_leaderboard', array of {name, time, date, shots}, sorted by time asc, top 10. Show panel on right side of screen or below canvas. I'll draw an HTML leaderboard panel beside canvas.

Sounds: WebAudio oscillator - shoot (short noise/whoosh), hit (ding). Simple.

UI elements (HTML/CSS):
- Header title
- Canvas
- Right panel: timer display, targets hit count, leaderboard list, restart button, instructions
- Overlay modal on finish: time, rank, name input, save button

Let me also handle target stands (draw a stick under targets).

Let me now write full code.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🏹 弓箭射击模拟</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: "Segoe UI", "Microsoft YaHei", sans-serif;
    background: linear-gradient(135deg, #1a2a1a, #0d1a2b);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 16px;
    color: #eee;
  }
  h1 { ... }
  .layout { display: flex; gap: 16px; flex-wrap: wrap; justify-content: center; }
  canvas { background: skyblue gradient; border-radius; box-shadow; cursor: crosshair; }
  .panel { width: 260px; ... }
  ...
</style>
```

Canvas background: draw sky gradient, clouds, grass, maybe hills inside canvas draw.

Draw archer: simple stick figure with bow. Draw bow as arc, string. When aiming, draw arrow nocked pointing in shoot direction.

Let me write drawing functions:

```javascript
function drawBackground() {
  // sky
  const sky = ctx.createLinearGradient(0,0,0,H);
  sky.addColorStop(0,'#87CEEB');
  sky.addColorStop(0.7,'#cfefff');
  sky.addColorStop(1,'#eafbe7');
  ctx.fillStyle = sky; ctx.fillRect(0,0,W,H);
  // sun
  ...
  // clouds (static positions)
  // hills
  ctx.fillStyle = '#7ec850';
  // ground
  ctx.fillStyle = '#5aa73c';
  ctx.fillRect(0, GROUND_Y, W, H-GROUND_Y);
}
```

GROUND_Y = 545.

Draw bow & archer:
```javascript
function drawArcher() {
  // body
  ctx.strokeStyle = '#333';
  // simple: head circle, body line, legs
  // bow: arc rotated to aimAngle when aiming else vertical
  const ang = aiming ? aimAngle : -Math.PI/2 * 0 ... 
}
```

Keep it stylized: a person silhouette at bow.x, bow.y area. Bow at (bow.x, bow.y).

Bow drawing: arc of radius 28 centered at bow pos, spanning angle perpendicular... A bow shape: arc from angle+90°-60° to angle+90°+60°... Let me think: bow limb is an arc; string connects tips, pulled back. When aiming, string pulled toward drag point.

Simplify: draw arc centered at bow pos with radius 26, from aimAngle + π/2 + 0.5π? Hmm.

Bow arc: the bow curves around the archer, opening toward shoot direction. Arc spans from (aimAngle + 90°) to (aimAngle - 90°) going through (aimAngle + 180°)? Actually the bow's belly faces archer (opposite shoot direction). So arc centered at bow pos, radius 26, from aimAngle+60° to aimAngle-60° passing through aimAngle+180°. i.e., startAngle = aimAngle + Math.PI*(2/3), endAngle = aimAngle - Math.PI*(2/3), drawn counterclockwise... Let me param: tip1 = bow + 26*(cos(aimAngle+θ), sin(aimAngle+θ)), tip2 = bow + 26*(cos(aimAngle-θ), sin(aimAngle-θ)) with θ = 70°. Arc through the back: ctx.arc(bow.x, bow.y, 26, aimAngle+θ, aimAngle-θ+2π... hmm careful. The arc should go the long way around (through aimAngle+π). Using ctx.arc(x, y, r, a1, a2, counterclockwise=true) from aimAngle+θ down... 

If a1 = aimAngle + θ (≈ +70° relative), and we go counterclockwise (increasing... in canvas, angles increase clockwise because y down). Ugh. Let me just use: ctx.arc(bow.x, bow.y, 26, aimAngle + θ, aimAngle - θ, false) — draws clockwise from a1 to a2 (in canvas, clockwise = increasing angle visually downward). Since y is flipped, "clockwise" math direction appears counterclockwise. Easier approach: draw arc through the far side: use arc from aimAngle+θ to aimAngle-θ with the flag that goes the long way. The arc not passing through shoot direction.

Simplest robust: draw bow as a quadratic curve between tip1 and tip2 with control point behind bow pos: ctrl = bow + 26*1.2*(cos(aimAngle+π), sin(aimAngle+π)) — bulges away from target. That looks like a bow (curve bulging backward). String: line from tip1 to pullPoint to tip2, where pullPoint = bow - dir * pullDist (pullDist = power*22 when aiming, 6 idle). Arrow drawn from pullPoint along dir, length 40.

That's clean. Do that.

Arrow drawing: line + arrowhead (triangle) + fletching lines. Rotation = atan2(vy, vx) when flying.

Targets drawing: pole from ground to target, circle rings: outer white, then black? Classic archery target: white, black, blue, red, yellow center. Radii fractions: 1, 0.8, 0.6, 0.4, 0.2. If hit: play fall animation — rotate around bottom and drop alpha. Simple: when hit, animate fall: target falls down with rotation over 0.6s then stays as faded, or disappears with particles. I'll do: fallProgress 0→1; y += fall*80, rotation = fall*0.5, alpha = 1-fall. Then removed (draw nothing after).

Particles: small colored circles with velocity, gravity, fade.

Collision: for each flying arrow, tip = pos (store pos as tip position, arrow drawn trailing behind). Check each unhit target: dist(tip, target) < r → hit.

Also arrow stuck when tip y >= GROUND_Y or x > W-10: push to stuckArrows with rotation; stuck arrows fade after few seconds.

Timer display: HTML element updated via requestAnimationFrame or in game loop. Show "时间： 0.00s", "命中： 0/4", "箭数： n".

Finish: overlay div shows final time, your rank (compute where time would place), input name, button 保存成绩， button 再来一局.

Rank calculation: insert into sorted list, find index. If saved, highlight entry.

Leaderboard rendering: function renderBoard() reads localStorage, fills <ol>.

Also "清空排行榜" button maybe. Add small.

Reset function: re-make targets, clear arrows/particles, hits=0, shots=0, startTime=null, endTime=null, state='ready'.

Edge: while overlay open, ignore canvas events.

Sound via WebAudio:
```javascript
let actx;
function beep(freq, dur, type='sine', vol=0.15){ ... }
function shootSound(){ noise whoosh — use quick descending square 400→100 }
function hitSound(){ two-tone ding }
```
Create AudioContext lazily on first user gesture.

Trajectory preview: simulate p = bow pos, v = dir*speed; step dt=0.05, 18 steps, draw dots fading.

Power→speed: speed = 250 + power*650 (range 250–900). Max drag distance 150px.

Angle clamp: dir = normalize(bow - mouse). If dir.x < 0.05 → too far back; clamp angle: a = atan2(dir.y, dir.x); clamp a to [-1.45, 1.2]; if |a|>π/2 (pointing left) set a = -1.45? Let's do: if cos(a) < 0 → a = (sin(a) >= 0 ? 1.2 : -1.45)? Hmm messy. Better: clamp a into [-1.45, 1.2]; but atan2 returns in [-π, π]; if a > 1.2 && a <= π → could be 1.5 (left-up). Handle: if a > π/2 → a = ... user pulling right-down means shooting left-up; we clamp to up: a = -1.45? If dir points left, just clamp a to nearest of [-1.45, 1.2]? a in (π/2, π] (up-left): clamp to... we want shoot up: -1.45 is up (negative angle = up since y down? angle -1.45 → sin negative → dir.y negative → up. yes). So if a > π/2 → a = -1.45? That snaps weirdly but acceptable. If a < -π/2 (down-left): a = 1.2? a from atan2 in (-π, -π/2) → pointing down-left → clamp to 1.2 (down-right steep). Wait 1.2 rad ≈ 69° downward — fine for close low target.

Let me simplify: compute a, then:
```
if (a > Math.PI/2) a = -1.45;       // was pointing up-left -> shoot nearly straight up
else if (a < -Math.PI/2) a = 1.2;   // down-left -> steep down-right
a = Math.max(-1.45, Math.min(1.2, a));
```
Hmm, snapping up-left to straight-up is odd. Alternative: just allow full range except directly backward: clamp so cos(a) >= 0.15. If user pulls so mouse is right of bow, arrow would shoot left — clamp to vertical. It's fine.

Actually simplest UX: most players pull down-left to shoot up-right. Clamp range [-1.45, 1.2] covers everything needed. OK.

Minimum power: if power < 0.08 on release → cancel shot (no arrow, and don't start timer).

Timer start on first released shot that actually fires.

Game loop: requestAnimationFrame with dt clamp.

Also draw trajectory dots only while aiming.

Wind: skip.

Clouds drift slowly for life. Store clouds array with x moving.

Also add small flag/banner maybe. Keep scope reasonable.

Let me also add keyboard R for restart.

Now the HTML panel with instructions:
- 操作：按住鼠标左键并向后拖动以拉弓，松开射击
- 命中全部 4 个靶子，用时越短排名越高
- R 键重新开始

Leaderboard storage format: JSON array [{name, time (seconds float), shots, date}]. Sort by time. Keep 10.

Name input: default "玩家". On save: push, sort, slice, save, re-render, show rank highlight.

Also show best time in panel ("个人最佳").

Test instructions: 
- Open file in browser (double-click or serve via python http.server)
- Play: drag & release, hit 4 targets, overlay appears, enter name, save, leaderboard updates
- Verify localStorage persistence by reload
- Test restart button & R key
- Test cancel shot with tiny drag

I'll write everything in Chinese UI since request is Chinese.

Now write the final file carefully. I need to be careful about correctness since no runtime testing. Let me write clean, careful code.

File: archery.html (or 弓箭射击游戏.html — use archery_game.html to avoid encoding issues). I'll name it `archery_game.html`.

Let me write it:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🏹 弓箭射击模拟</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
    background: radial-gradient(circle at 50% 0%, #23354d, #101825 70%);
    min-height: 100vh;
    color: #e8eef5;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 18px 12px 30px;
    user-select: none;
  }
  h1 { font-size: 26px; letter-spacing: 2px; margin: 4px 0 2px; }
  .subtitle { color: #9fb4c8; font-size: 13px; margin-bottom: 14px; }
  .layout { display: flex; gap: 18px; align-items: flex-start; flex-wrap: wrap; justify-content: center; }
  .stage { position: relative; }
  canvas { display: block; border-radius: 12px; box-shadow: 0 12px 40px rgba(0,0,0,.5); cursor: crosshair; }
  .hud { position: absolute; top: 10px; left: 12px; display: flex; gap: 14px; font-size: 15px; font-weight: 600; text-shadow: 0 1px 2px rgba(0,0,0,.4); pointer-events: none; }
  .hud span { background: rgba(0,0,0,.35); padding: 5px 12px; border-radius: 20px; }
  .panel { width: 270px; background: rgba(255,255,255,.06); border: 1px solid rgba(255,255,255,.12); border-radius: 12px; padding: 16px; }
  .panel h2 { font-size: 17px; margin: 0 0 10px; }
  .panel ol { margin: 0 0 0 22px; padding: 0; font-size: 14px; }
  .panel li { margin: 6px 0; }
  .panel li.me { color: #ffd94d; font-weight: 700; }
  .empty { color: #8ea3b5; font-size: 13px; }
  button { ... }
  .overlay { position:absolute; inset:0; background: rgba(8,14,22,.78); display:flex; align-items:center; justify-content:center; border-radius:12px; }
  .card { background:#16222f; border:1px solid rgba(255,255,255,.15); padding:26px 30px; border-radius:14px; text-align:center; width: 320px; }
  .card .time { font-size: 40px; color:#ffd94d; font-weight:800; margin: 8px 0; }
  .card input { width: 100%; padding: 9px 12px; border-radius: 8px; border: 1px solid #3a4d61; background:#0e1722; color:#fff; margin: 12px 0; font-size: 15px; }
  .hidden { display:none !important; }
  .tips { margin-top: 12px; color:#8ea3b5; font-size:12.5px; line-height:1.7; }
</style>
```

Then body with layout, canvas 960x600, HUD spans (time, hits, shots), panel with leaderboard, best, buttons (重新开始， 清空榜）, tips.

Overlay inside .stage (position absolute over canvas).

Script:

```javascript
(function(){
"use strict";
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const GROUND_Y = 548;
const GRAVITY = 620;
const MAX_DRAG = 160;
const MIN_SPEED = 240, MAX_SPEED = 920;
const ARROW_LEN = 42;

const bow = { x: 92, y: 418 };

let targets = [], arrows = [], stuck = [], particles = [], clouds = [];
let aiming = false, mouse = {x:0,y:0};
let aimAngle = -0.5, power = 0;
let startTime = null, endTime = null;
let hits = 0, shots = 0;
let lastTs = 0;
let audioCtx = null;
let boardHighlight = -1;

const LS_KEY = 'archery_leaderboard_v1';
```

Targets config with colors maybe different per target for clarity? All classic style. Give each a stand.

```javascript
function resetGame(){
  targets = [
    { x: 610, y: 150, r: 34, hit:false, fall:0 },
    { x: 812, y: 235, r: 34, hit:false, fall:0 },
    { x: 560, y: 415, r: 34, hit:false, fall:0 },
    { x: 845, y: 468, r: 34, hit:false, fall:0 },
  ];
  arrows = []; stuck = []; particles = [];
  aiming = false; power = 0;
  startTime = null; endTime = null; hits = 0; shots = 0;
  boardHighlight = -1;
  hide(overlay);
  updateHud();
}
```

Clouds:
```javascript
for(let i=0;i<4;i++) clouds.push({x:Math.random()*W, y:40+Math.random()*90, s:0.7+Math.random()*0.7, v:6+Math.random()*8});
```

Audio:
```javascript
function ensureAudio(){ if(!audioCtx){ try{ audioCtx = new (window.AudioContext||window.webkitAudioContext)(); }catch(e){} } if(audioCtx && audioCtx.state==='suspended') audioCtx.resume(); }
function tone(freq, dur, type, vol, when=0, slideTo){
  if(!audioCtx) return;
  const t = audioCtx.currentTime + when;
  const o = audioCtx.createOscillator(), g = audioCtx.createGain();
  o.type = type||'sine'; o.frequency.setValueAtTime(freq, t);
  if(slideTo) o.frequency.exponentialRampToValueAtTime(slideTo, t+dur);
  g.gain.setValueAtTime(vol||0.15, t);
  g.gain.exponentialRampToValueAtTime(0.001, t+dur);
  o.connect(g); g.connect(audioCtx.destination);
  o.start(t); o.stop(t+dur+0.02);
}
function sndShoot(){ tone(520, 0.12, 'square', 0.06, 0, 120); }
function sndHit(){ tone(880,0.09,'sine',0.18); tone(1320,0.16,'sine',0.15,0.08); }
function sndFinish(){ tone(660,0.12,'sine',0.15); tone(880,0.12,'sine',0.15,0.12); tone(1175,0.25,'sine',0.15,0.24); }
```

Input:
```javascript
canvas.addEventListener('mousedown', e=>{
  if(endTime !== null) return;  // finished
  ensureAudio();
  aiming = true;
  mouse = getPos(e);
  updateAim();
});
window.addEventListener('mousemove', e=>{
  if(!aiming) return;
  mouse = getPos(e);
  updateAim();
});
window.addEventListener('mouseup', e=>{
  if(!aiming) return;
  aiming = false;
  if(power >= 0.08){ fire(); }
  power = 0;
});
function getPos(e){
  const r = canvas.getBoundingClientRect();
  return { x: (e.clientX - r.left) * (canvas.width / r.width), y: (e.clientY - r.top) * (canvas.height / r.height) };
}
function updateAim(){
  let dx = bow.x - mouse.x, dy = bow.y - mouse.y;
  let len = Math.hypot(dx, dy);
  power = Math.min(len / MAX_DRAG, 1);
  if(len < 1){ return; }
  let a = Math.atan2(dy, dx);
  if(a > Math.PI/2) a = -1.45;        // up-left -> straight up
  else if(a < -Math.PI/2) a = 1.2;    // down-left -> steep down
  aimAngle = Math.max(-1.45, Math.min(1.2, a));
}
```

Wait: if a > π/2, that means dx<0, dy<0 (mouse right-below bow → shoot up-left). Setting a=-1.45 (up). And a < -π/2: dx<0, dy>0 → shoot down-left → clamp 1.2. Then clamp. OK. Hmm, but then power direction might mismatch—fine.

fire():
```javascript
function fire(){
  const speed = MIN_SPEED + power*(MAX_SPEED-MIN_SPEED);
  const vx = Math.cos(aimAngle)*speed, vy = Math.sin(aimAngle)*speed;
  arrows.push({ x: bow.x + Math.cos(aimAngle)*20, y: bow.y + Math.sin(aimAngle)*20, vx, vy });
  shots++;
  if(startTime === null) startTime = performance.now();
  sndShoot();
  updateHud();
}
```

Update loop:
```javascript
function update(dt){
  // clouds
  for(const c of clouds){ c.x += c.v*dt; if(c.x > W+80) c.x = -80; }
  // arrows
  for(let i=arrows.length-1;i>=0;i--){
    const a = arrows[i];
    a.vy += GRAVITY*dt;
    a.x += a.vx*dt; a.y += a.vy*dt;
    // target collision
    for(const t of targets){
      if(t.hit) continue;
      const d = Math.hypot(a.x - t.x, a.y - t.y);
      if(d < t.r){
        t.hit = true; t.fall = 0.0001;
        hits++;
        burst(t.x, t.y);
        sndHit();
        // arrow sticks into target briefly: keep as stuck at impact
        stuck.push({ x:a.x, y:a.y, rot:Math.atan2(a.vy,a.vx), life:1.2, fadeAfter:0.6 });
        arrows.splice(i,1);
        if(hits === targets.length){ finish(); }
        break;
      }
    }
    if(!arrows[i] && removedFlag)...
  }
}
```

Careful: after splice in inner loop, we break, but outer loop continues referencing arrows[i] — need guard. Use a flag:

```javascript
for(let i=arrows.length-1;i>=0;i--){
  const a = arrows[i];
  a.vy += GRAVITY*dt; a.x += a.vx*dt; a.y += a.vy*dt;
  let dead = false;
  for(const t of targets){
    if(t.hit) continue;
    if(Math.hypot(a.x-t.x, a.y-t.y) < t.r){
      t.hit = true; hits++;
      burst(t.x,t.y); sndHit();
      stuck.push({x:a.x,y:a.y,rot:Math.atan2(a.vy,a.vx),life:1.4});
      dead = true;
      if(hits === targets.length) finish();
      break;
    }
  }
  if(!dead){
    const rot = Math.atan2(a.vy, a.vx);
    if(a.y >= GROUND_Y){ stuck.push({x:a.x, y:GROUND_Y, rot, life:5}); dead = true; }
    else if(a.x > W+60 || a.y > H+60 || a.x < -80){ dead = true; }
  }
  if(dead) arrows.splice(i,1);
}
```

Stuck arrows life decay, particles update, target fall anim:
```javascript
for(let i=stuck.length-1;i>=0;i--){ stuck[i].life -= dt; if(stuck[i].life<=0) stuck.splice(i,1); }
for(const t of targets){ if(t.hit && t.fall < 1) t.fall = Math.min(1, t.fall + dt*1.6); }
for(let i=particles.length-1;i>=0;i--){
  const p = particles[i];
  p.vy += 500*dt; p.x += p.vx*dt; p.y += p.vy*dt; p.life -= dt;
  if(p.life<=0) particles.splice(i,1);
}
```

burst:
```javascript
function burst(x,y){
  const colors = ['#ffd94d','#ff5a5a','#4da6ff','#ffffff'];
  for(let i=0;i<26;i++){
    const a = Math.random()*Math.PI*2, s = 80+Math.random()*240;
    particles.push({x,y,vx:Math.cos(a)*s,vy:Math.sin(a)*s-60,life:0.5+Math.random()*0.5,color:colors[i%4],r:2+Math.random()*3});
  }
}
```

finish():
```javascript
function finish(){
  endTime = performance.now();
  sndFinish();
  const t = (endTime - startTime)/1000;
  // compute rank against saved board
  const board = loadBoard();
  let rank = 1;
  for(const e of board){ if(e.time < t) rank++; }
  finalTimeEl.textContent = t.toFixed(2) + ' 秒';
  rankInfoEl.textContent = board.length ? ('当前可排第 ' + rank + ' 名' + (rank>10?'（榜单仅保留前 10）':'')) : '首位完赛者！';
  show(overlay);
  nameInput.value = localStorage.getItem('archery_last_name') || '';
  setTimeout(()=>nameInput.focus(), 50);
}
```

Save:
```javascript
saveBtn.onclick = ()=>{
  const t = (endTime - startTime)/1000;
  const name = (nameInput.value.trim() || '无名射手').slice(0,12);
  localStorage.setItem('archery_last_name', name);
  const board = loadBoard();
  board.push({name, time:+t.toFixed(2), shots, date: Date.now()});
  board.sort((a,b)=>a.time-b.time);
  const idx = board.findIndex(e=>e.name===name && e.time===+t.toFixed(2) && e.date? ...) 
```

findIndex to highlight: capture the object reference: create entry obj, push, sort, idx = board.indexOf(entry). Then keep if idx<10 slice. highlight = idx if <10.

```javascript
  const entry = {name, time:+t.toFixed(2), shots, date:Date.now()};
  board.push(entry);
  board.sort((a,b)=>a.time-b.time || a.date-b.date);
  const idx = board.indexOf(entry);
  const top = board.slice(0,10);
  saveBoard(top);
  boardHighlight = idx < 10 ? idx : -1;
  renderBoard();
  resetGame();  // closes overlay
};
```

Wait resetGame sets boardHighlight=-1; renderBoard after? Order: compute highlight, then resetGame (which hides overlay and resets highlight), then set highlight and renderBoard. Let me restructure: in resetGame don't touch boardHighlight; handle separately. Fine.

Also "跳过保存" / clicking restart just resets.

renderBoard():
```javascript
function loadBoard(){ try{ return JSON.parse(localStorage.getItem(LS_KEY)) || []; }catch(e){ return []; } }
function saveBoard(b){ localStorage.setItem(LS_KEY, JSON.stringify(b)); }
function renderBoard(){
  const b = loadBoard();
  boardEl.innerHTML = '';
  if(!b.length){ boardEl.innerHTML = '<div class="empty">暂无记录，快来创造第一个成绩！</div>'; bestEl.textContent = '—'; return; }
  b.forEach((e,i)=>{
    const li = document.createElement('li');
    if(i === boardHighlight) li.className = 'me';
    li.textContent = `${e.name} — ${e.time.toFixed(2)}s（${e.shots}箭）`;
    boardEl.appendChild(li);
  });
  bestEl.textContent = b[0].time.toFixed(2) + ' s';
}
```

Drawing:

```javascript
function draw(){
  drawBackground();
  drawTargets();
  drawStuck();
  drawArcher();
  drawArrows();
  drawParticles();
  if(aiming) drawTrajectory();
}
```

drawBackground: sky gradient, sun, clouds, distant hills, ground with texture line, maybe a few flowers.

drawTargets: for each target: stand (brown rect from ground to target y) — but high targets need long poles; draw pole only for lower ones? All can have poles; looks fine. If hit and fall>=1 → skip (gone). Else apply transform: translate(t.x, t.y + t.fall*140), rotate(t.fall*0.9), alpha 1-fall.

Rings: colors from outer to inner: ['#f5f5f5','#333','#4da6ff','#ff5a5a','#ffd94d'] radii r*[1,0.8,0.6,0.4,0.22]. Classic order: white, black, blue, red, gold. Draw outer to inner. Add thin stroke.

Stand: ctx.fillStyle '#8b5a2b', fillRect(t.x-4, t.y, 8, GROUND_Y - t.y). Draw before target circle. When target hit, keep stand (pole remains). So draw poles separately for all targets (hit or not), then falling target circle.

drawArcher: 
```javascript
function drawArcher(){
  // simple archer figure
  ctx.save();
  ctx.translate(bow.x - 18, bow.y + 10);
  // body
  ctx.strokeStyle = '#4a3520';
  ctx.lineWidth = 5; ctx.lineCap='round';
  // legs
  ctx.beginPath(); ctx.moveTo(0,40); ctx.lineTo(4,78); ctx.moveTo(0,40); ctx.lineTo(-12,78); ctx.stroke();
  // torso
  ctx.beginPath(); ctx.moveTo(0,40); ctx.lineTo(2,8); ctx.stroke();
  // head
  ctx.fillStyle = '#f0c8a0';
  ctx.beginPath(); ctx.arc(3,-4,9,0,Math.PI*2); ctx.fill();
  // arm toward bow
  ctx.beginPath(); ctx.moveTo(2,14); ctx.lineTo(20,8); ctx.stroke();
  ctx.restore();
```

Eh, the figure at (bow.x-18, bow.y+10)... bow.y=418, ground 548. Legs from y+40=458 to y+78=496 — not reaching ground. Let me place figure so feet at ground: translate(bow.x - 16, GROUND_Y - 118). Feet at local y=118 → ground. Bow at approx local (16, 418-(548-118)) = (16, -12)?? Getting complicated. Simplify: figure drawn relative to ground anchor, bow separate. Let me just draw a nice simple archer:

anchor ax = bow.x - 20, feet at GROUND_Y.
- legs: from hip (ax, GROUND_Y-52) to feet (ax-8, GROUND_Y), (ax+12, GROUND_Y)
- torso: hip to shoulder (ax+4, GROUND_Y-96)
- head: circle at (ax+6, GROUND_Y-110), r 9
- arm: shoulder to bow point (bow.x, bow.y)
Bow at bow.y = GROUND_Y - 88 ≈ 460? Set bow.y = GROUND_Y - 90 = 458. Hmm previously 418. Set bow = {x:92, y: GROUND_Y - 92} = (92,456). Shoulder at (ax+4, GROUND_Y-96) = (76, 452); bow (92,456) — arm short line, fine.

Colors: tunic green. Keep it simple and decent.

Bow drawing (at bow pos):
```javascript
function drawBow(){
  const dirx = Math.cos(aimAngle), diry = Math.sin(aimAngle);
  const theta = 1.15; // half span of bow tips
  const R = 30;
  const t1 = { x: bow.x + R*Math.cos(aimAngle+theta), y: bow.y + R*Math.sin(aimAngle+theta) };
  const t2 = { x: bow.x + R*Math.cos(aimAngle-theta), y: bow.y + R*Math.sin(aimAngle-theta) };
  // control bulge behind
  const cx = bow.x - dirx*R*0.9, cy = bow.y - diry*R*0.9;
  ctx.strokeStyle = '#6b4423'; ctx.lineWidth = 5; ctx.lineCap='round';
  ctx.beginPath(); ctx.moveTo(t1.x,t1.y); ctx.quadraticCurveTo(cx,cy,t2.x,t2.y); ctx.stroke();
  // string
  const pull = aiming ? power*20 : 5;
  const px = bow.x - dirx*pull, py = bow.y - diry*pull;
  ctx.strokeStyle = 'rgba(240,240,240,.9)'; ctx.lineWidth = 1.5;
  ctx.beginPath(); ctx.moveTo(t1.x,t1.y); ctx.lineTo(px,py); ctx.lineTo(t2.x,t2.y); ctx.stroke();
  // nocked arrow while aiming
  if(aiming){
    const tail = { x: px, y: py };
    const head = { x: px + dirx*ARROW_LEN, y: py + diry*ARROW_LEN };
    drawArrowShape(tail.x, tail.y, aimAngle, 1);
  }
}
```

Wait default aimAngle when not aiming: keep last or set to -0.35 idle. When idle (not aiming), bow drawn with aimAngle idle value; string slightly pulled. fine.

drawArrowShape(x, y, rot, alpha): arrow from (x,y) along rot: shaft line from tail to head? Define arrow by its head position for flying arrows: pos = tip. Draw shaft from tip - dir*ARROW_LEN to tip; head triangle at tip; fletching at tail.

```javascript
function drawArrowShape(tx, ty, rot, alpha){
  const dx = Math.cos(rot), dy = Math.sin(rot);
  const px = -dy, py = dx; // perpendicular
  const sx = tx - dx*ARROW_LEN, sy = ty - dy*ARROW_LEN;
  ctx.save();
  ctx.globalAlpha = alpha;
  // shaft
  ctx.strokeStyle = '#8a5a2b'; ctx.lineWidth = 3; ctx.lineCap='round';
  ctx.beginPath(); ctx.moveTo(sx,sy); ctx.lineTo(tx - dx*6, ty - dy*6); ctx.stroke();
  // head
  ctx.fillStyle = '#cfd6dd';
  ctx.beginPath();
  ctx.moveTo(tx, ty);
  ctx.lineTo(tx - dx*10 + px*4, ty - dy*10 + py*4);
  ctx.lineTo(tx - dx*10 - px*4, ty - dy*10 - py*4);
  ctx.closePath(); ctx.fill();
  // fletching
  ctx.strokeStyle = '#e74c3c'; ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(sx, sy); ctx.lineTo(sx + dx*8 + px*5, sy + dy*8 + py*5);
  ctx.moveTo(sx, sy); ctx.lineTo(sx + dx*8 - px*5, sy + dy*8 - py*5);
  ctx.stroke();
  ctx.restore();
}
```

For nocked arrow while aiming, "tip" = pull point + dir*ARROW_LEN → drawArrowShape(px+dirx*ARROW_LEN, py+diry*ARROW_LEN, aimAngle, 1). Good — unify.

drawArrows: for flying arrows rot=atan2(vy,vx). Stuck arrows: alpha = min(1, life) (fade last second).

drawTrajectory:
```javascript
function drawTrajectory(){
  if(power < 0.08) return;
  const speed = MIN_SPEED + power*(MAX_SPEED-MIN_SPEED);
  let x = bow.x + Math.cos(aimAngle)*20, y = bow.y + Math.sin(aimAngle)*20;
  let vx = Math.cos(aimAngle)*speed, vy = Math.sin(aimAngle)*speed;
  ctx.fillStyle = 'rgba(255,255,255,.85)';
  const step = 0.06;
  for(let i=0;i<16;i++){
    vy += GRAVITY*step; x += vx*step; y += vy*step;
    if(y > GROUND_Y) break;
    ctx.globalAlpha = 0.9 - i*0.05;
    ctx.beginPath(); ctx.arc(x, y, 3, 0, Math.PI*2); ctx.fill();
  }
  ctx.globalAlpha = 1;
}
```

Power bar near bow:
```javascript
function drawPowerBar(){
  if(!aiming) return;
  const bw=10, bh=90, bx=bow.x-52, by=bow.y-45;
  ctx.fillStyle='rgba(0,0,0,.3)'; ctx.fillRect(bx,by,bw,bh);
  const h = power*bh;
  const grad... color by power: green->red
  ctx.fillStyle = `hsl(${120 - power*120}, 85%, 50%)`;
  ctx.fillRect(bx, by+bh-h, bw, h);
  ctx.strokeStyle='#fff'; ctx.lineWidth=1; ctx.strokeRect(bx,by,bw,bh);
}
```

HUD update via DOM each frame or on events: time display updates every frame while running:
```javascript
function updateHud(){
  hitEl.textContent = `🎯 ${hits} / ${targets.length}`;
  shotEl.textContent = `🏹 ${shots}`;
}
```
Time in loop:
```javascript
let t = 0;
if(startTime!==null){ t = ((endTime||performance.now()) - startTime)/1000; }
timeEl.textContent = '⏱ ' + t.toFixed(2) + 's';
```

Main loop:
```javascript
function loop(ts){
  const dt = Math.min((ts - lastTs)/1000 || 0, 0.033);
  lastTs = ts;
  update(dt);
  draw();
  updateTimeHud();
  requestAnimationFrame(loop);
}
```

Restart button and R key:
```javascript
restartBtn.onclick = ()=>{ resetGame(); };
window.addEventListener('keydown', e=>{ if(e.key==='r'||e.key==='R'){ if(overlay visible) ... just resetGame(); } });
```

Clear board button: confirm, remove LS key, renderBoard.

Overlay buttons: saveBtn saves & resets; againBtn (skip save) resets.

Prevent context menu? Not needed.

Touch support: add touchstart/touchmove/touchend mapping to same handlers, with e.preventDefault to avoid scrolling. Nice-to-have; add basic support:

```javascript
canvas.addEventListener('touchstart', e=>{ e.preventDefault(); startAim(e.touches[0]); }, {passive:false});
canvas.addEventListener('touchmove', e=>{ e.preventDefault(); moveAim(e.touches[0]); }, {passive:false});
canvas.addEventListener('touchend', e=>{ e.preventDefault(); endAim(); }, {passive:false});
```
Unify mouse/touch via pointer-like helpers taking {clientX, clientY}.

I'll implement with functions startAim(p), moveAim(p), endAim().

Note: after finish (endTime set), disable aiming; overlay covers canvas anyway (overlay inset covers canvas → clicks blocked). Since overlay is child of .stage covering canvas, mousedown on canvas won't fire. Good, but keep guard too.

Overlay HTML:
```html
<div id="overlay" class="overlay hidden">
  <div class="card">
    <h2>🎉 全部命中！</h2>
    <div class="time" id="finalTime">0.00 秒</div>
    <div id="rankInfo" class="rankinfo"></div>
    <input id="nameInput" maxlength="12" placeholder="输入你的名字">
    <div class="btnrow">
      <button id="saveBtn">保存成绩</button>
      <button id="skipBtn" class="ghost">不保存</button>
    </div>
  </div>
</div>
```

Also start hint drawn on canvas before first shot: draw text "按住鼠标向后拖动拉弓，松开射击" if startTime===null && !aiming. In draw — nice.

Now panel HTML:
```html
<div class="panel">
  <h2>🏆 排行榜（前 10）</h2>
  <ol id="board"></ol>
  <div class="best">个人最佳：<b id="best">—</b></div>
  <div class="btnrow">
    <button id="restartBtn">重新开始 (R)</button>
    <button id="clearBtn" class="ghost">清空榜单</button>
  </div>
  <div class="tips">
    操作：在画布上按住鼠标左键并<b>向后拖动</b>拉弓（拖得越远力量越大），松开射出。<br>
    计时从第一箭射出开始，命中全部 4 个靶子后停止，用时越短排名越高。<br>
    靶心不分环数，命中即算。
  </div>
</div>
```

best = top of board overall (leaderboard best). "榜首成绩" better. Rename: 榜首成绩.

Now double-check collision timing: finish() called during update; overlay shown; draw continues. aiming guard.

Edge: user releases mouse off-window — mouseup on window handles it. good.

Clouds init before loop.

One more check: `if(a > Math.PI/2) a = -1.45;` — but then power uses drag length; shooting straight up when user pulls straight down-right of bow... Actually if mouse is to the right of bow and below, dx<0, dy<0 → angle in (π/2, π) → we set up. Reasonable.

Now let me also make sure trajectory preview dots match actual arrow start: fire starts arrow at bow + dir*20. Use same in preview. Good.

Arrow speed: MAX 920, gravity 620. Range: v²/g for 45°: 920²/620 ≈ 1365px — plenty. Farthest target 845, distance from bow ~760. Fine.

Can arrows reach high target at (610,150)? dy = -306 up, dx 518. With 45°, speed ~700: range 790, height max v²sin²/2g = 700²*0.5/1240 ≈ 197... not enough height? Max height = (v·sin45)²/(2g) = (495)²/1240 ≈ 197px. Target needs 306px up at distance 518. Hmm! Need higher arcs: shoot more vertical: angle 70°, v=900: vx=308, vy=846. Time to x=518: t=1.68s; y = -846*1.68 + 310*1.68² = -1421+875 = -546 up — overshoots; on the way... it passes y=-306 somewhere. Yes reachable. Max height with v=900 at 70°: 846²/1240 ≈ 577 > 306 ✓.

With power full 920: fine. Gravity 620 maybe makes arrows drop fast; that's fine, it's arcade.

Let me double check lowest far target (845,468): dx=753, dy≈12. v≈800 at ~15°? Reachable easily.

Ground at 548; target at y=468, r=34 → bottom 502 above ground ✓. Pole from 468 to 548 ✓.

Bow y=456. Trajectory to (560,415): dx 468, dy -41. fine.

Now, stuck arrow on ground: y=GROUND_Y, draw with rot — arrow tip at ground, shaft tilted. Good.

Canvas size 960x600. Panel beside. Mobile: wrap.

I think the design is complete. Let me also add `updateTimeHud` and finish guard: after finish, startTime stays, endTime set; time HUD shows frozen final.

renderBoard initial call.

Also add subtle target bobbing? Skip — keep fair.

Write final code now. Also mention test steps and how to run: open file directly, or `python -m http.server`. Provide test checklist:
1. 打开页面，看到4个靶子、弓手、排行榜为空
2. 拖动射击，确认抛物线预览、力量条
3. 命中靶子：粒子效果、计数增加、声音
4. 全部命中：弹出结算、显示时间与名次，输入名字保存 → 排行榜更新且高亮
5. 刷新页面 → 排行榜保留（localStorage）
6. R 键/按钮重开；清空榜单按钮
7. 小拖动弹射取消（不发射不计时）
8. 计时从第一箭开始验证：射前时间 0.00

Deliver single file `archery_game.html`.

Let me be careful with template literal usage inside the HTML script — fine.

One subtle bug risk: in `fire()`, `arrows.push({...})` then `updateHud`. OK.

In update loop, after finish() overlay shown, update continues; arrows remain... fine, game frozen visually except anims. endTime set → aiming disabled by guard in startAim: `if(endTime !== null) return;`.

Also guard: if overlay open and user presses R → resetGame hides overlay. good.

highlight persistence: after save & renderBoard with highlight; boardHighlight reset in resetGame; ensure order: saveBtn handler: compute highlight AFTER resetGame. Let me write:

```javascript
saveBtn.addEventListener('click', ()=>{
  if(endTime===null || startTime===null) return;
  const t = +(((endTime-startTime)/1000).toFixed(2));
  const name = (nameInput.value.trim() || '无名射手').slice(0,12);
  localStorage.setItem('archery_last_name', name);
  const board = loadBoard();
  const entry = { name, time: t, shots, date: Date.now() };
  board.push(entry);
  board.sort((a,b)=> a.time-b.time || a.date-b.date);
  const idx = board.indexOf(entry);
  saveBoard(board.slice(0,10));
  resetGame();
  boardHighlight = (idx >= 0 && idx < 10) ? idx : -1;
  renderBoard();
});
```

skipBtn: resetGame(); boardHighlight=-1; renderBoard(); (no change to board, but re-render fine.)

clearBtn: if(confirm('确定清空排行榜？')){ localStorage.removeItem(LS_KEY); boardHighlight=-1; renderBoard(); }

restartBtn: resetGame().

Also `resetGame` calls updateHud and hides overlay.

Also drawTargets draw order: poles first then circles for all; falling circles alpha.

Pole also for hit targets (still standing). Yes.

Cloud drawing: three circles.

Hills: two ellipses behind ground.

Let me also add flags on poles? Skip.

Also add small "wind-free" note? no.

Also draw a subtle aim guide line from bow while aiming? Trajectory dots suffice.

Write the whole file now. Also ensure `<ol>` numbering styles fine.

Font sizes etc fine.

Also prevent text selection while dragging: body user-select none ✓. Also canvas drag outside: mousemove on window ✓.

Potential issue: `getPos` uses canvas.getBoundingClientRect each time — fine.

Now produce final answer: single HTML file + run instructions + test instructions. Keep response concise-ish but complete with full code. The code is long (~450 lines) but necessary.

Let me write it out fully and carefully, checking syntax as I go.

Also HUD spans: id timeEl etc. Use Chinese labels.

Extra polish: title header "🏹 弓箭射击模拟" and subtitle "命中全部 4 个靶子，用时越短排名越高！".

Final check on the `finish` rank text when board empty: "首位完赛者！" else "当前可排第 X 名". Also mention shots.

Also in card show arrows used: `共射出 N 箭`. Add div.

OK write code.

One more consideration: `drawArcher` colors and positions:
- ax = bow.x - 22 = 70; GROUND_Y=548.
- hip at (70, 496); shoulder (74, 452); head center (77, 438) r9 → top 429.
- arm: shoulder (74,452) → bow (92,456).
- legs: hip(70,496) → (60,548) and (82,548).
- Also back arm pulling? skip or small line toward string pull when aiming: from shoulder toward pull point. Add: if aiming, second arm from shoulder to pull point (px,py) — cute! Let's add.

Colors: torso '#2e7d4f', limbs same, head '#f0c8a0'.

Draw quiver? skip.

Also draw a simple target face emoji? no, rings fine.

Let me finalize numbers: bow = {x:92, y:456}.

updateAim clamp range [-1.45, 1.2]: at 1.2 rad downward — needed for target (560,415)? That's slightly above bow, no. Low targets: (845,468) is 12px below bow over 753 distance — slight down angle or flat with drop. Fine.

Check reachable without crazy precision: target r=34 generous.

Preview dots: 16 × 0.06s = 0.96s flight preview. Good.

Now sounds: ensure AudioContext created on mousedown (user gesture) ✓.

Write final HTML. Also include `<noscript>`? unnecessary.

For rendering the board list element with empty message inside ol — use innerHTML div inside ol is invalid; use separate container: `<div id="boardWrap"><ol id="board"></ol><div id="boardEmpty" class="empty hidden">暂无记录…</div></div>`. Simpler: build via DOM, toggle empty div.

Code:

```javascript
const boardEl = document.getElementById('board');
const emptyEl = document.getElementById('boardEmpty');
function renderBoard(){
  const b = loadBoard();
  boardEl.innerHTML = '';
  emptyEl.classList.toggle('hidden', b.length>0);
  b.forEach((e,i)=>{
    const li = document.createElement('li');
    if(i===boardHighlight) li.className='me';
    li.textContent = `${e.name} — ${e.time.toFixed(2)}s · ${e.shots}箭`;
    boardEl.appendChild(li);
  });
  bestEl.textContent = b.length ? b[0].time.toFixed(2)+' s' : '—';
}
```

Alright. Also when hitting target, arrow stuck at target location with life 1.4 — visually arrow sticking in falling target: but target falls while stuck arrow stays floating — acceptable (brief). Alternatively skip stuck on target hit (arrow consumed by burst). I'll keep short-life stuck; fine, actually it might look odd floating midair after target fades. Set life 0.5 quick fade. OK.

Time HUD formatting: `t.toFixed(2)+' s'`.

Also add `document.title`? fine as is.

Now compose the final answer with:
1. Brief intro
2. Full code block (single file)
3. 运行方式
4. 测试说明 (checklist)
5. 玩法/实现要点 short

Make sure code has no syntax errors. I'll mentally lint as I write.

Careful with arrow shape head triangle coordinates and template strings.

Also the `hsl` template string with power: `hsl(${120 - power*120}, 85%, 50%)` fine.

Potential pitfall: In drawBow when not aiming, pull=5 constant — string slightly bent; fine.

Idle aimAngle: initialize -0.35.

Also draw start hint text:
```javascript
if(startTime===null && !aiming){
  ctx.fillStyle='rgba(0,0,0,.55)';
  ctx.font='16px sans-serif';
  ctx.textAlign='center';
  ctx.fillText('按住鼠标左键向后拖动拉弓，松开射击 —— 计时从第一箭开始', W/2, 30);
}
```
Shadow for readability: add white text with dark bg? Use rounded rect behind. Keep simple: dark translucent rect.

OK writing final file.

Also add `ctx.textAlign='left'` reset where needed.

Let me count DOM ids: game, timeEl, hitEl, shotEl, overlay, finalTime, rankInfo, shotsInfo, nameInput, saveBtn, skipBtn, board, boardEmpty, best, restartBtn, clearBtn.

Write it.

function drawBackground details:

```javascript
function drawBackground(){
  const sky = ctx.createLinearGradient(0,0,0,GROUND_Y);
  sky.addColorStop(0,'#7ec8f2');
  sky.addColorStop(1,'#dff3ff');
  ctx.fillStyle = sky;
  ctx.fillRect(0,0,W,GROUND_Y);
  // sun
  ctx.fillStyle = '#ffe27a';
  ctx.beginPath(); ctx.arc(880, 64, 30, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = 'rgba(255,226,122,.35)';
  ctx.beginPath(); ctx.arc(880, 64, 44, 0, Math.PI*2); ctx.fill();
  // clouds
  ctx.fillStyle = 'rgba(255,255,255,.9)';
  for(const c of clouds){
    ctx.beginPath();
    ctx.arc(c.x, c.y, 18*c.s, 0, Math.PI*2);
    ctx.arc(c.x+20*c.s, c.y+4*c.s, 14*c.s, 0, Math.PI*2);
    ctx.arc(c.x-20*c.s, c.y+5*c.s, 13*c.s, 0, Math.PI*2);
    ctx.fill();
  }
  // hills
  ctx.fillStyle = '#b7e39a';
  ctx.beginPath(); ctx.ellipse(200, GROUND_Y, 260, 70, 0, Math.PI, 0); ctx.fill();
  ctx.beginPath(); ctx.ellipse(760, GROUND_Y, 320, 90, 0, Math.PI, 0); ctx.fill();
  // ground
  const g = ctx.createLinearGradient(0,GROUND_Y,0,H);
  g.addColorStop(0,'#6fbf4a'); g.addColorStop(1,'#4c8f33');
  ctx.fillStyle = g; ctx.fillRect(0,GROUND_Y,W,H-GROUND_Y);
  ctx.fillStyle = 'rgba(0,0,0,.12)'; ctx.fillRect(0,GROUND_Y,W,3);
}
```

ellipse with start π to 2π draws upper half — antclockwise param: ctx.ellipse(x,y,rx,ry,rot,start,end) with start=Math.PI, end=2*Math.PI draws top half going counterclockwise visually? In canvas angles measured clockwise from +x (y down). Angle π = left point; 2π=right point going through π/2? From π to 2π increasing passes 3π/2 = top (since y down, angle 3π/2 → sin=-1 → up). Yes upper half. Good.

Targets draw:

```javascript
const RING_COLORS = ['#f4f4f4','#2c2c2c','#3f8efc','#ef3f4d','#ffd94d'];
const RING_SCALE = [1, .8, .6, .4, .22];
function drawTargets(){
  // poles
  ctx.fillStyle = '#8b5a2b';
  for(const t of targets){
    ctx.fillRect(t.x-4, t.y+t.r-6, 8, GROUND_Y-(t.y+t.r-6));
  }
  for(const t of targets){
    if(t.hit && t.fall>=1) continue;
    ctx.save();
    if(t.hit){
      ctx.globalAlpha = 1 - t.fall;
      ctx.translate(t.x, t.y);
      ctx.rotate(t.fall * 0.9);
      ctx.translate(0, t.fall*120);
      ctx.translate(-t.x, -t.y);
      // hmm combining rotate and fall translation: rotate around target center then drop
    }
    drawTargetFace(t.x, t.y, t.r);
    ctx.restore();
  }
}
function drawTargetFace(x,y,r){
  for(let i=0;i<5;i++){
    ctx.beginPath();
    ctx.fillStyle = RING_COLORS[i];
    ctx.arc(x, y, r*RING_SCALE[i], 0, Math.PI*2);
    ctx.fill();
  }
  ctx.strokeStyle='rgba(0,0,0,.35)'; ctx.lineWidth=2;
  ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.stroke();
}
```

Rotation around center then translation for fall: better: translate(t.x, t.y + t.fall*120); rotate(...); drawTargetFace(0,0,r)? drawTargetFace uses x,y — pass 0,0. ✓

```javascript
ctx.save();
if(t.hit){
  ctx.globalAlpha = 1 - t.fall;
  ctx.translate(t.x, t.y + t.fall*130);
  ctx.rotate(t.fall*0.9);
  drawTargetFace(0,0,t.r);
} else {
  drawTargetFace(t.x,t.y,t.r);
}
ctx.restore();
```

Clean.

Particles draw: circles with alpha = life*2 clamp.

```javascript
function drawParticles(){
  for(const p of particles){
    ctx.globalAlpha = Math.max(0, Math.min(1, p.life*2));
    ctx.fillStyle = p.color;
    ctx.beginPath(); ctx.arc(p.x,p.y,p.r,0,Math.PI*2); ctx.fill();
  }
  ctx.globalAlpha = 1;
}
```

Arrows flying draw: drawArrowShape(a.x, a.y, atan2(a.vy,a.vx), 1).

Stuck: alpha = min(1, life) (life counts down from 5 → alpha 1 until <1). For target-hit stuck life 0.5.

Power bar + trajectory drawn after archer.

Order in draw: background, targets, stuck arrows, archer(figure+bow), flying arrows, particles, trajectory, powerbar, hint text.

Also draw bow after archer figure (arm overlaps fine).

Archer figure with aiming arm:

```javascript
function drawArcher(){
  const ax = bow.x - 22;
  ctx.lineCap = 'round';
  // legs
  ctx.strokeStyle = '#3b2d1e'; ctx.lineWidth = 5;
  ctx.beginPath();
  ctx.moveTo(ax, 496); ctx.lineTo(ax-10, GROUND_Y);
  ctx.moveTo(ax, 496); ctx.lineTo(ax+12, GROUND_Y);
  ctx.stroke();
  // torso
  ctx.strokeStyle = '#2e7d4f'; ctx.lineWidth = 7;
  ctx.beginPath(); ctx.moveTo(ax, 496); ctx.lineTo(ax+4, 452); ctx.stroke();
  // front arm to bow
  ctx.strokeStyle = '#3b2d1e'; ctx.lineWidth = 4.5;
  ctx.beginPath(); ctx.moveTo(ax+4, 458); ctx.lineTo(bow.x, bow.y); ctx.stroke();
  // pulling arm while aiming
  if(aiming){
    const dx = Math.cos(aimAngle), dy = Math.sin(aimAngle);
    const pull = power*20;
    ctx.beginPath(); ctx.moveTo(ax+4, 458); ctx.lineTo(bow.x - dx*pull, bow.y - dy*pull); ctx.stroke();
  }
  // head
  ctx.fillStyle = '#f0c8a0';
  ctx.beginPath(); ctx.arc(ax+7, 439, 9, 0, Math.PI*2); ctx.fill();
  // cap
  ctx.fillStyle = '#2e7d4f';
  ctx.beginPath(); ctx.arc(ax+7, 436, 9, Math.PI, 0); ctx.fill();
}
```

Hmm shoulder at (ax+4,452) vs arm start 458 fine.

Now full loop and init:

```javascript
function init(){
  for(let i=0;i<4;i++) clouds.push({...});
  resetGame();
  renderBoard();
  requestAnimationFrame(loop);
}
init();
```

resetGame defines targets etc.

updateHud inside resetGame sets hits/shots text.

updateTimeHud in loop:

```javascript
function updateTimeHud(){
  let t = 0;
  if(startTime !== null){
    const end = (endTime !== null) ? endTime : performance.now();
    t = (end - startTime)/1000;
  }
  timeEl.textContent = '⏱ ' + t.toFixed(2) + ' s';
}
```

Mouse handlers with preventDefault not needed.

Also add dblclick prevention? no.

Edge: user drags, releases outside window → mouseup on window fires ✓.

I think code is ready. Also ensure `hide`/`show` helpers:

```javascript
function show(el){ el.classList.remove('hidden'); }
function hide(el){ el.classList.add('hidden'); }
```

Overlay starts hidden in markup.

Also nameInput Enter key triggers save:
```javascript
nameInput.addEventListener('keydown', e=>{ if(e.key==='Enter') saveBtn.click(); e.stopPropagation(); });
```
stopPropagation so 'r' in name doesn't restart — keydown on window would catch 'r' while typing name! Important: window keydown handler checks if event target is input: 
```javascript
window.addEventListener('keydown', e=>{
  if((e.key==='r'||e.key==='R') && document.activeElement !== nameInput) resetGame();
});
```
Better check e.target: `if(e.target && e.target.tagName==='INPUT') return;`

OK. Final code assembly. Also make canvas width responsive via CSS max-width:100% — getPos scales properly since we scale by rect. Add `canvas{max-width:100%;height:auto;}`.

Done. Write answer.

Additionally provide 测试说明 and 运行方式 concisely. Also note localStorage requires http(s) or file:// works too (localStorage works on file:// in most browsers; some restrictions in Chrome? localStorage works on file:// generally. If issues, suggest python http.server). I'll mention both.

Length: the code will be long; that's expected and fine.