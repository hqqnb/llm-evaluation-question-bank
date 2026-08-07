We need answer in Chinese. Need create a complete archery shooting simulation game. Need produce runnable artifact. We need likely HTML/CSS/JS single file? Need include 4 targets, timing, leaderboard ranking by shortest time. Need not inspect other files. Need provide run instructions and test notes. Need build game. Let's think requirements: "制作一个弓箭射击模拟游戏。游戏中应有四个靶子。玩家命中所有目标所用的时间越短，在排行榜上的名次就越高。不要查看其他文件，直接创建。 输出要求：请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。"

Need deliver code maybe in one HTML file. Need implement archery shooting simulation game with 4 targets. Need define mechanics: player clicks/drags to aim/shoot? We need make playable. Could be crosshair aiming and click to shoot. Need 4 targets placed in scene. Need hit all four as fast as possible. Timer starts on first click/shoot? Leaderboard localStorage. Need rank based on shortest time among attempts. Need include four targets; each target has rings; hit target center? Need probably hit score? Since "命中所有目标" time to hit all targets; not necessarily points. Need maybe if hit outer ring counts? Need target must be hit (any ring?) Need center? We need design: Four targets at different positions/distances in 2D canvas. Player clicks "shoot"; arrow flies to mouse position. If it hits a target, target is marked hit. Timer ends when all 4 hit. Leaderboard stores best times.

Need maybe include arrow trajectory? Archery simulation perhaps with bow, arrow, gravity? We can implement 2D side-view? "射击模拟游戏" Could be top-down? Need "弓箭射击" with targets. Need maybe simpler: A first-person/perspective range: targets standing on field, aiming crosshair moves with mouse, click shoots arrow; arrow flies to crosshair in a straight line maybe with slight arc? In archery arrows fly with gravity. But time to hit all targets means player's speed not arrow flight. We can include bow at bottom center and targets on a field; click to release arrow toward mouse cursor. Need 4 targets. Need target hit detection based on crosshair coordinates, not arrow flight time. But visual arrow flight from bow to target would be nice.

Need decide implementation.

Option: Full 3D using Three.js? But no external? Could use Canvas 2D to simulate perspective. But "不要查看其他文件" just create. Need runnable in browser. Single HTML.

Need maybe include no external dependencies. We can use Canvas 2D. Draw a realistic archery range with a shooting line, target stands, sky, grass. Four targets at different positions maybe left, mid-left, mid-right, right; different distances? In 2D, targets have different y positions? Let's make pseudo-3D perspective: Ground plane with vanishing point; targets in scaled sizes. Crosshair at mouse position; clicking shoots arrow from bottom center. Need arrow flight path from bow to mouse/target? Actually if crosshair aims at target, arrow lands at crosshair (within tolerance). We can animate arrow from archer position to crosshair point over 0.2-0.5s; when arrow reaches target point, check target hit. But if targets are in 2D perspective, need a target at that point. We'll represent each target as an ellipse/circle at (x, y) with radius scaled. Hit condition: distance from landing crosshair to target center <= radius. Since click exactly on target center. Need maybe allow small tolerance; maybe "命中" means hitting any part of target. To reduce difficulty, hit if distance <= radius. But target radius maybe 30-60 px; good.

Need timer: starts when player clicks the "Start" button? Or when first shot is released. To avoid time before first shot? "玩家命中所有目标所用的时间" likely from start of attempt. We can include "点击开始/开始计时" button. But if "start" button then timer starts. Could start on first shot? Leaderboard time should include aiming time? Usually score from start of round. Let's implement: There is a "开始游戏" button that resets targets, starts timer. When active, click on range shoots. Timer stops when all four targets hit. On completion, if no existing leaderboard entry or time better, add to leaderboard. Need "排行榜" list sorted by time ascending. Since "名次越高" means shortest time top. Could rank among previous attempts. Need maybe store attempts in localStorage with timestamp. Display rank.

Need game flow:
- Page with heading.
- Canvas for game.
- UI: Timer display, button "重新开始 / 开始游戏", message.
- Leaderboard panel.

Need interact:
- Initially targets visible? Need "game not started"? Maybe targets visible but timer 0. Button starts. Could set "准备开始" and crosshair maybe disabled. Need click button to start. But if click button, maybe onClick of canvas? Need not.

Could implement:
State: 'idle', 'playing', 'finished'. targets = array with id, x, y, radius, hit bool, color maybe. Timer interval (requestAnimationFrame) updates elapsed time from startTime when playing. When finished, stops.

Need on click canvas:
- If state != 'playing' return.
- Need shoot arrow from archer position (bottom center maybe 400,550) to (mouse.x, mouse.y).
- We can animate arrow. But if multiple shots, arrow animation. Target hit is determined by where arrow lands, not by line of flight. We'll set pending arrow landing point. Draw arrow from bow to landing point over 0.35 sec; at landing, evaluate hit. But because the arrow path may cross multiple targets, maybe target hit if path crosses? Simpler: hit based on landing coordinate. However if clicking near target but arrow line passes through another target, should maybe hit that target too? Not needed. But "射击模拟" maybe arrow should fly straight; if target is in front, arrows don't hit behind. We can determine hit by path intersection with target in z-order. Better: Because targets in perspective, arrow from bottom center to click point; if line passes through a target, the first target along path might be hit. But top-down? Hmm. If archer at bottom center fires toward a target, a target closer? If target is an object in 2D as drawn, line from bow to target point may pass through other target's region. In real archery, if one target is in front of another, arrow could hit the wrong one. But with 4 targets placed not overlapping in screen, line to one target may pass near others. Could hit nearest along line. Need determine.

Let's define targets at positions:
Targets in perspective:
- Left far? maybe target0 at (180, 260), radius 34 (small)
- Right far? (620, 260), radius 34
- Left near? (120, 420), radius 48
- Right near? (680, 420), radius 48
But if crosshair click far target, line from bow (400, 570) to (180,260) may pass through near target at (120,420)? Unlikely; line x at y=420: from bottom (400,570) to top (180,260), slope dx/dy = (400-180)/(570-260)=220/310=0.71; x = 400 + 0.71*(420-570)=400-106=294, not near 120. So not.

Need maybe target positions with "behind" visual. But for hit detection, if click on target center, arrow lands within radius. Need include "hit target" if distance <= radius. But if two targets overlap? no.

Need arrow animation:
- Store shots array: { startX, startY, targetX, targetY, startTime, duration, hit? }.
- In update loop, if currentTime >= startTime + duration, arrow at end; then check hit and add effect. But if multiple shots, process each. Need target hit only once. If already hit, clicking on it shouldn't matter but animation still. Could maybe show "已命中" marker.
- Need crosshair: mouse move track crosshair. Draw crosshair at mouse position (if inside canvas). During idle maybe crosshair visible. If state finished? maybe no.
- Need crosshair hidden when mouse leaves canvas.
- Need bow: draw at bottom center; maybe rotate toward mouse. Use mouse angle. Draw bow arc and string. Draw arrow from bow to crosshair? Could animate. Need perhaps each shot: arrow flies from bow to target. But if arrow remains at landing, draw arrows sticking in target? Simpler: draw line segments.

Need maybe sound? Not necessary. Could use WebAudio for "shoot" and "hit" if possible. But output not require. Maybe include simple beep using AudioContext. But can skip to avoid complexity. Could include.

Need scoring: hit all targets. Need maybe "命中所有目标所用时间" not "score". If arrow misses target, no time penalty, but player must retry. We can show "脱靶" message. To make game interesting, maybe target hit only if within inner rings? If click anywhere on target circle, hit. Need perhaps center 10? Wait "靶子" has rings; 命中 target can mean hit any ring. But to be accurate, "target" is a target board, arrow hitting anywhere on board counts. Good.

Need leaderboard:
- On finish, get currentBest from localStorage key "archeryLeaderboard" array. Add time; sort ascending; keep top maybe 10. Determine rank 1-based index; if time equal, by timestamp earlier? Need rank among all attempts. We'll store attempts as {time, date}. The new attempt rank = index+1 after insertion sorted ascending. If equal, maybe compare date. Use sort by time ascending then date ascending. Display table: 排名, 用时, 时间. Also highlight the new attempt's rank? Could.

Need "排行榜上的名次就越高" meaning rank higher if time lower. We'll compute rank.

Need maybe "玩家命中所有目标所用的时间越短，在排行榜上的名次就越高" Could mean compare to other players? We'll maintain local leaderboard. If localStorage unavailable, in-memory.

Need maybe "不要查看其他文件，直接创建" means not referencing other files. So single HTML file.

Need "提供运行方式和必要的测试说明" in final answer. We'll include instructions: Save as .html and open in browser; or run via local server; test by clicking start, shoot all four targets; leaderboard persists. But "交付可运行产物" likely code block. Need perhaps only code, plus instructions. We can include both. Need ensure code complete.

Let's design detailed implementation.

Canvas dimensions: 800x600. UI with panels around.

HTML structure:
```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>弓箭射击模拟游戏</title>
<style>...</style>
</head>
<body>
<div id="app">
 <h1>🏹 弓箭射击模拟游戏</h1>
 <div id="main">
  <div id="gamePanel">
   <div id="topBar">
     <button id="startBtn">开始游戏</button>
     <span id="timer">用时: 0.00s</span>
     <span id="status">点击“开始游戏”开始</span>
   </div>
   <canvas id="gameCanvas" width="800" height="600"></canvas>
   <div id="instructions">点击靶子射击；全部命中后结束。用时越短排名越高。</div>
  </div>
  <div id="leaderboardPanel">
    <h2>🏆 排行榜</h2>
    <table><thead><tr><th>排名</th><th>用时</th><th>完成时间</th><th>标记</th></tr></thead><tbody id="leaderboardBody"></tbody></table>
    <button id="clearScoresBtn">清空排行榜</button>
  </div>
 </div>
</div>
<script>...</script>
</body>
</html>
```

Need style responsive.

Game state:
```
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const startBtn = ...
const timerEl = ...
const statusEl = ...
const leaderboardBody = ...
const clearBtn = ...

let state = 'idle'; // idle/playing/finished
let startTime = 0;
let elapsedTime = 0;
let rafId = null;
let lastFrameTime = 0;
let mouse = {x:400,y:300, inside:false};
let shots = [];
let targets = [];
let effects = [];
let finishedFlag = false;
```
Targets:
```
function createTargets() {
 return [
   {id:0, name:'左远', x:170, y:240, r:34, hit:false, color:'#e74c3c'},
   {id:1, name:'右远', x:630, y:240, r:34, hit:false, color:'#3498db'},
   {id:2, name:'左近', x:120, y:420, r:50, hit:false, color:'#f1c40f'},
   {id:3, name:'右近', x:680, y:420, r:50, hit:false, color:'#2ecc71'} 
 ];
}
```
Need perhaps there are four target "stands" with legs. Draw target board with rings and "hit" markers.

Need draw function:
- Draw sky gradient, grass horizon maybe at y=200? Need targets at y 240 and 420. Horizon maybe y=180. Grass gradient.
- Draw shooting line: maybe bottom at y=560.
- Draw archer at bottom: a simple stylized bow, person? Could draw just bow and arrow. Need perhaps "玩家" at bottom center. Let's implement a simple archer silhouette? Need not complex. Draw bow and maybe arm. Since game uses mouse click, maybe crosshair.

Maybe better to draw "archer at bottom center" as a bow with arrow nocked and string. Need rotate toward mouse. We can draw lines:
- Bow: arc with radius 28 around bow center at (400, 540) if mouse angle. But if target above, arrow from bow center. Draw arc opposite string? Hmm.

Could use canvas transforms:
```
const archerX=400, archerY=540;
let angle = Math.atan2(mouse.y - archerY, mouse.x - archerX);
ctx.save();
ctx.translate(archerX, archerY);
ctx.rotate(angle);
ctx.strokeStyle = '#6d4c41';
ctx.lineWidth=4;
ctx.beginPath();
ctx.arc(0,0,28, -Math.PI/2 - 0.4, -Math.PI/2 + 0.4, false); // draw bow arc? Actually bow arc is symmetric around aim direction? If rotate angle, the arc should be on opposite side? For a bow held vertical, when aiming up, bow arc is side. We'll draw a vertical bow: arc from -1.2 to 1.2? Need visual.
```
Need not overcomplicate. Could draw bow as a simple arc facing left/right? Since targets above, arrow angle around -60° to -110°? We can draw bow with rotate(angle + Math.PI/2), an arc. But okay.

Need maybe use an emoji? Canvas can draw text "🏹" at bottom center, rotated? Maybe draw a bow shape manually.

Let's design archer:
- Bow center at (400, 530). If mouse is to right? But target positions from 120 to 680, so angle can be from left/right. We can calculate angle. Draw bow as arc with radius 30 from -1.1 to 1.1 relative to vertical? Suppose rotate to angle (atan2). Draw an arc to the side? Let's define:
```
ctx.save();
ctx.translate(bowX, bowY);
ctx.rotate(angle);
ctx.beginPath();
ctx.arc(0, 0, 30, -1.2, 1.2, false); // arc on right side? If rotate angle, arrow points to mouse. For a bow, the bow limb should be on the side perpendicular to arrow; Actually archer holds bow vertical if shooting horizontal; if angle is from +x axis, the bow plane is perpendicular? Wait in side view, bow is vertical and arrow horizontal. If aiming up at angle, bow still vertical (not rotated) maybe. But for simplicity, rotate bow to face mouse. Let's draw bow as arc from -1.2 to 1.2 around the direction to mouse? Hmm, if angle=0 (to right), arrow should point to right, bow arc should be vertical: arc from -1.2 to 1.2 centered at 0? That is arc on right side, not vertical. Wait maybe use rotate(angle + Math.PI/2)? Let's find: In canvas, ellipse arc with angles measured from positive x-axis. A vertical bow is arc from -1.2 to 1.2? Actually arc from -1.2 to 1.2 is small arc on right side, like a bow facing right. If arrow direction right, that's okay. If arrow direction up (angle = -PI/2), rotating by -PI/2 makes arc on top, okay. Good.
```
Need string: line from one end of arc to the other. But if bow arc is on right side, string should be vertical at x=0? We'll draw line from (cos(-1.2)*30, sin(-1.2)*30) to (cos(1.2)*30, sin(1.2)*30) in rotated frame. This is a vertical line (constant x~10.9? Wait cos of ±1.2 = 0.362, sin=±0.932, so endpoints (10.9,±28). So string x=10.9. Arrow from bow center to mouse. Good enough.
- Draw arrow line from bow center to crosshair maybe (if no shot). Need arrow line maybe thin.
Need not draw person; just bow.

Need crosshair:
```
function drawCrosshair(x,y) {
 ctx.strokeStyle='rgba(255,255,255,0.9)';
 ctx.lineWidth=1.5;
 ctx.beginPath(); arc x,y,10...
 lines...
}
```

Need draw target:
Function drawTarget(t):
- Determine if hit: show "✓" maybe.
- Draw stand: legs from bottom of target to ground line? For each target, compute groundY = min(t.y + t.r? maybe depending on perspective). Draw two lines from stand base to bottom? Hmm.
Could draw a wooden target stand: rectangle/board with rings and legs. Since target x,y is center of board. We can draw legs below:
```
ctx.strokeStyle = '#8b5a2b';
ctx.lineWidth=4;
ctx.beginPath();
ctx.moveTo(t.x - t.r*0.6, t.y + t.r*0.85);
ctx.lineTo(t.x - t.r*1.2, t.y + t.r*2.2);
ctx.moveTo(t.x + t.r*0.6, t.y + t.r*0.85);
ctx.lineTo(t.x + t.r*1.2, t.y + t.r*2.2);
ctx.stroke();
```
But if target near bottom, legs could extend to y=500. Fine. Need grass covers.

- Draw board circle (outer white) and rings:
```
const rings = [
 {color:'#2c3e50', ratio:1.0},
 {color:'#ffffff', ratio:0.85},
 {color:'#2c3e50', ratio:0.7},
 {color:'#ffffff', ratio:0.55},
 {color:'#c0392b', ratio:0.4},
 {color:'#f1c40f', ratio:0.2}
];
```
Actually archery target has rings: white, black, blue, red, yellow. Use colors: outer white, black, blue, red, yellow. 
```
const ringDefs = [
 {r:1.0, color:'#fff'},
 {r:0.8, color:'#222'},
 {r:0.6, color:'#2980b9'},
 {r:0.4, color:'#e74c3c'},
 {r:0.2, color:'#f1c40f'}
];
```
Draw each with fillCircle centered t.x,t.y with radius t.r * ratio. For target board, draw subtle shadow.

- If hit: maybe green overlay "HIT" and checkmark. But still rings? Could draw target with a dark overlay? Need show hit. Maybe draw green ring around board and a checkmark. Could leave target visible and add "已命中" text. Let's add:
```
if (t.hit) {
 ctx.strokeStyle='#2ecc71'; lineWidth=4; strokeCircle r+5; // green outline
 ctx.fillStyle='rgba(46,204,113,0.3)'; fill circle? Not hide rings.
}
```
Need maybe show arrow stuck in target? We can draw arrow at landing point if hit. We'll store shots with hit target. Draw each shot arrow as small line from maybe bottom to target? Hmm.

Need draw arrows in flight/landed:
- For each shot object:
  - start = {x: archerX, y: archerY}; end = {x: shot.targetX, y: shot.targetY}.
  - If animating progress p = clamp((now - shot.time)/duration,0,1); current position = lerp.
  - Draw arrow as line from start to current + maybe a fletching. Actually arrow length ~ 40 px in direction from start to end. Use drawArrowLine from current to current - direction*35? Wait arrow points from bow to target; arrowhead at current. Draw line from behind to current, with tip. We'll draw a line with arrowhead at current, and small feathers at back. Simpler:
```
function drawArrow(shot, now) { ... 
 const angle = Math.atan2(end.y-start.y,end.x-start.x);
 ctx.save(); ctx.translate(cx,cy); ctx.rotate(angle);
 ctx.strokeStyle = '#8b4513'; ctx.lineWidth=3;
 ctx.beginPath(); ctx.moveTo(-30,0); ctx.lineTo(0,0); ctx.stroke();
 ctx.fillStyle = '#999'; // head
 ctx.beginPath(); ctx.moveTo(6,0); ctx.lineTo(-3,-4); ctx.lineTo(-3,4); ctx.closePath(); fill();
 // fletching
 ctx.fillStyle='#e74c3c'; triangle at -30...
 ctx.restore();
}
```
Need if arrow landed, p=1, draw at target point. If multiple arrows, okay.

Need hit detection at end of shot:
```
function resolveShot(shot) {
 let hitTarget = null;
 // Sort targets by distance from archer? Because line may pass through multiple. But if click target center, line to target exactly passes through. But if click entirely miss, no hit. We can use distance from end point to target center <= radius. This is sufficient. However if arrow trajectory line passes through a closer target before missing, we'd miss that. To simulate realistic, need check along line maybe. But target boards are at different z; if an arrow flies to a far target, it should pass in front of near target? Let's think: In 2D screen, the line from archer to far target may pass over or through a near target. If it passes through the near target's screen region, it should hit that near target. But if the near target is drawn on screen, and arrow line crosses it, visually arrow would pass through target, which should count. We can implement path intersection with targets using line segment from start to end and circular targets, taking first (smallest parameter) target hit. Need determine "z-order" by first along the line from archer. That would be more realistic. But if we count targets only by endpoint, visual line crossing target could be ignored. Let's implement line-circle intersection to be more robust.

Line segment from archer (start) to end. For each target not hit, if segment intersects circle (center, r), compute t (0..1) where distance from center to line <= r and projection t along segment. The first intersection with smallest t. If a target is at the endpoint, t near 1. If another target is closer along line, it hits that one. Need "first along path". But in archery, if the near target is in front of far target, yes. However targets drawn as billboards; due to perspective, "near" target might be at bottom of screen. Line from archer to far target might cross near target's region if near target is not exactly on same ray. If crosses, hit near. Good.

Need consider target "already hit" should still block? In real life, an arrow hitting a target already hit would stop; but game maybe you can still shoot it but no need. We should skip already hit targets? For hit resolution, if path crosses an already hit target, should arrow be blocked? Since you don't need to hit it again, but it's physically there; arrow hitting it shouldn't count and shouldn't hit target behind? Hm. To be fair, if there are already hit target boards, they still block arrows. But if user wants to hit other targets, an arrow crossing an already hit target might be blocked. That could be annoying. But can be realistic. However target boards are arranged so lines to each target don't cross other target circles? Need check with chosen positions.

Let's choose positions so no cross-contamination. Need ensure straight line from any target center to archer does not cross other target circles. Let's compute. Archer (400,530). Targets:
A left near (120,420,r50)
B right near (680,420,r50)
C left far (170,240,r38)
D right far (630,240,r38)
Need line from archer to C (170,240) vs A (120,420). At y=420, line x = 400 + (400-170)/(400? Actually from y530 to 240, dy=-290; dx=-230. At y=420, t=(530-420)/(530-240)=110/290=0.379; x=400 + (-230)*0.379=312.8. A circle at x120,r50, no. Good.
Line to D vs B: at y=420, x = 400 + (230)*0.379=487; B at 680, no.
Line to A vs C? From archer to A, at y=240, t=(530-240)/(530-420)=290/110=2.64 >1, not between, so far target behind? Actually A is nearer bottom, line from archer to A extended beyond A doesn't go to C. So no.
Line to B vs D: similarly no.
Line to A vs D? from archer to A at y=240, x=400 + (120-400)/(530-420)*(240-530?) Let's compute line slope from archer to A: dx/dy = (120-400)/(420-530)= -280/-110=2.545; at y=240, x=400 + 2.545*(240-530)=400 -738 = -338, no. So no cross.
Line to B vs C: at y=240, x=400 + (680-400)/(420-530)*(240-530)=400 + (-2.545)*(-290)=1138, no. So no cross. Great.

Need maybe use line-circle intersection to be realistic, but endpoints check enough. We can do both maybe. Since targets are separated, no issue.

But with arrows animating, you might click a target, arrow lands in target, hit. If you miss, arrow sticks in grass. Need maybe "脱靶" message if no target. We'll set status message "命中！" or "脱靶" for 1 sec.

Need timer:
Use performance.now() for elapsed. In RAF:
```
function frame(now) {
  if (state === 'playing') {
    elapsedTime = (now - startTime) / 1000;
    timerEl.textContent = `用时: ${elapsedTime.toFixed(2)}s`;
  }
  update(now);
  draw(now);
  rafId = requestAnimationFrame(frame);
}
```
Need startTime should be performance.now() when button clicked.

Start button:
- If clicked: reset all targets hit false; shots = []; effects = []; state='playing'; startTime=performance.now(); statusEl.textContent = '游戏进行中，射击全部 4 个靶子！'; button text maybe '重新开始'; timer start.
- If finished and click start, same.

Need "finished" check after resolving hit. Need update in loop? If shot ends and target hit, check all hit. Could do resolveShot at end of shot animation. But if multiple shots in flight and second one ends after first? Need process when shot.time + duration <= now. For each shot and not resolved. Since duration 0.35s, maybe multiple shots. Need when a shot resolves, if hit target, mark hit. Then check all targets hit. But if we use path intersection with target and target already hit? Need maybe only not hit targets. We can determine nearest target hit along line (including already hit? Need maybe if already hit target crossed, should block; but we can skip already hit to avoid issues). Let's decide: Skip already hit targets when resolving path? Then arrow can pass through already hit target to hit behind. Not realistic but okay. The goal is to hit all targets as quickly as possible; if your arrow path crosses a previously hit target, you can still pass through and hit another. That's maybe not physical. But to keep gameplay smooth, use endpoint distance to unhit targets and ignore already hit. However in shooting range, targets are separated not overlapping. So no difference. We'll implement endpoint condition: if landing point within unhit target radius, hit it. This is simple.

Need maybe "hit all targets" could be too easy if clicking each target center. But user must aim. Good.

Need maybe "4 targets" in game. Need maybe allow target positions random per attempt? If fixed, easy to memorize. But "simulation" maybe fixed range. Could randomize target positions each reset to add challenge. But if random, line crossing issue maybe. Need leaderboard times comparable if positions random? Could be less fair. Fixed positions consistent. Need maybe "targets" at same positions but visual maybe enough. Could randomize? No, keep fixed.

Need "排行榜" can store local attempts. Need if same player multiple attempts, all attempts counted. "名次越高" maybe ranking among all records. We'll display rank for each attempt sorted by time. If user beats their best, added. Need maybe "best time" at top. Could include "最佳成绩" display. Let's add top best maybe.

Need if localStorage available:
```
const STORAGE_KEY = 'archery_leaderboard_v1';
function loadScores() {
 try { const data = localStorage.getItem(STORAGE_KEY); return data ? JSON.parse(data) : []; } catch { return []; }
}
function saveScores(scores) { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(scores)); } catch {} }
```
Scores entries: { time: number, date: string, id: string }.

On finish:
```
function finishGame(now) {
  state='finished';
  elapsedTime = (now - startTime)/1000;
  const time = elapsedTime;
  statusEl.textContent = `🎉 命中全部靶子！用时 ${time.toFixed(2)} 秒`;
  const scores = loadScores();
  scores.push({time, date: new Date().toLocaleString(), id: Date.now()+''+Math.random()});
  scores.sort((a,b)=>a.time - b.time || (new Date(a.date) - new Date(b.date))); // date string compare maybe okay
  const maxEntries = 50; if >50 slice.
  saveScores(scores);
  renderLeaderboard();
  // Highlight new entry maybe id.
}
```
Need if time 0? Could finish on all targets? Start button ensures.

Need "rank" of new attempt: after push sorted, index = scores.findIndex(s=>s.id===id)+1. But if scores sorted and maxEntries truncation? If attempt not in top 50, no rank. But likely fine. Need maybe keep all? We'll keep top 50. But rank among top 50? Could say rank not saved if not in list. Better keep all attempts? LocalStorage can handle many. But to show leaderboard, only top 10 perhaps. Need "rank" should be among all attempts? If we truncate, rank inaccurate. Let's store all attempts except maybe cap at 100? Could still. We'll keep top 100. But rank among all attempts in storage. If we cap 100, worst rank maybe 100. Good.

Render leaderboard:
- Load scores sorted ascending. Clear tbody. If empty, show one row "暂无记录".
- For first maybe 10 rows, include rank, time, date, tag "最新"/"最佳"? Need highlight new attempt. We'll maintain `lastPlayId`. In render, for each score up to 10, row class if id===lastPlayId. First row "🥇", second "🥈", third "🥉".
- Need maybe display "名次" "用时" "完成时间" "备注". Good.

Need "clear scores" button with confirm? We'll implement.

Need maybe input player name? Not asked. Could maybe "玩家" same. But leaderboard could have attempts. Fine.

Need maybe "time to hit all targets" starts on button, not first click. Good.

Need maybe if player clicks "开始游戏", timer starts immediately; if not ready, okay.

Need "test instructions" include:
- Open file in modern browser.
- Click "开始游戏".
- Move mouse to aim; click on a target to shoot; crosshair indicates aim.
- Hit all 4 targets to finish; timer stops automatically.
- After each finish, leaderboard updates and sorts by time; best (shortest) at top.
- "清空排行榜" to reset.

Need perhaps "必要的测试说明": If using automated tests? Could mention:
- Manual: verify four targets exist, timer starts after clicking start, each click shoots an arrow, target hit changes to green "HIT", all hit stops timer, leaderboard saves and sorts.
- Timing: compare displayed timer to stopwatch. Arrow flight time not counted? Wait timer starts on button; arrow flight time is included (0.35s per shot). That's okay.
- Multiple attempts: click 开始游戏 again; previous leaderboard retained. Need if click during playing "重新开始"? Button text "重新开始"; but if clicked while playing, it resets. Hmm.

Need perhaps "开始游戏" button during playing should reset and start new attempt. But accidental click resets. Maybe okay. Could set text "重新开始" and confirm? Not necessary.
Need maybe "state finished" if click "重新开始", reset. Good.

Need maybe "shooting" during idle not allowed. But if click canvas idle, status says "请先点击开始游戏". Good.

Need maybe "shots" from archer to mouse with duration proportional to distance? Could set duration = min(0.25 + distance/2000, 0.6). Need arrow speed constant? If short target, 0.25s; long target, 0.5. Use:
```
const dist = Math.hypot(end.x-start.x, end.y-start.y);
duration = Math.min(0.5, 0.15 + dist/1800);
```
Maybe okay.
Need if multiple shots clicked rapidly, arrow animations all. Need resolve each. Good.

Need "timer" and "shot resolution" timing with RAF. Use `performance.now()`.

Need "mouse" coordinates relative to canvas:
```
function getMousePos(e) {
 const rect = canvas.getBoundingClientRect();
 mouse.x = (e.clientX - rect.left) * (canvas.width / rect.width);
 mouse.y = (e.clientY - rect.top) * (canvas.height / rect.height);
}
canvas.addEventListener('mousemove', ...);
canvas.addEventListener('mouseenter', ...);
canvas.addEventListener('mouseleave', ...);
```
Need maybe "click" uses same coordinates. Use `mousemove` already.

Need "canvas" fixed width 800 height 600; CSS max-width 100%.

Need draw ground/background:
```
function drawBackground() {
 // sky
 const sky = ctx.createLinearGradient(0,0,0,200);
 sky.addColorStop(0,'#87ceeb'); sky.addColorStop(1,'#cfe9f7');
 ctx.fillStyle=sky; ctx.fillRect(0,0,800,200);
 // sun?
 ctx.fillStyle='#ffd700'; ctx.beginPath(); arc(80,60,25); fill();
 // hills maybe
 // ground
 const ground = ctx.createLinearGradient(0,180,0,600);
 ground.addColorStop(0,'#6aa84f'); ground.addColorStop(1,'#3e7d32');
 ctx.fillStyle=ground; ctx.fillRect(0,180,800,420);
 // horizon line
 ctx.strokeStyle='#2f5d27'; lineWidth=4; line 0,180 to 800,180
 // grass lines maybe
}
```
Need targets draw after background? Need "behind" maybe draw far targets first, then near targets, then archer. But if all targets on same plan, draw far targets first, then near. We'll define target order by y maybe. Already positions. Draw sorted by y descending? To appear in perspective, farther targets (higher y) should be drawn first, then nearer (lower y) drawn over. So:
```
const sortedTargets = [...targets].sort((a,b)=>a.y - b.y);
for (const t of sortedTargets) drawTarget(t);
```
Since far y=240, near y=420. Good.

Need draw shadows:
- For each target, draw ellipse shadow at t.y + t.r*0.8? Actually target board is centered; bottom of board at y + r. Stands legs extend. We can draw shadow on ground under target? But if target is "floating", not. Simpler:
```
ctx.fillStyle='rgba(0,0,0,0.15)';
ctx.beginPath(); ctx.ellipse(t.x, t.y + t.r*1.4, t.r*1.2, t.r*0.35, 0,0,2PI); fill();
```
Need stand legs. Because target is 2D side view, center y and radius define target board; legs below.

Need draw target board:
```
function drawTarget(t) {
 // shadow
 ctx.fillStyle='rgba(0,0,0,0.2)';
 ctx.beginPath();
 ctx.ellipse(t.x, t.y+t.r*1.1, t.r*1.4, t.r*0.4, 0, 0, 2*Math.PI);
 ctx.fill();
 // legs
 ctx.strokeStyle='#5d3a1a'; ctx.lineWidth=5;
 ctx.beginPath();
 ctx.moveTo(t.x - t.r*0.5, t.y + t.r*0.8);
 ctx.lineTo(t.x - t.r*1.3, t.y + t.r*2.0);
 ctx.moveTo(t.x + t.r*0.5, t.y + t.r*0.8);
 ctx.lineTo(t.x + t.r*1.3, t.y + t.r*2.0);
 ctx.stroke();
 // board outer
 const ringDefs = [
 {ratio:1.0, color:'#ffffff'},
 {ratio:0.8, color:'#2c3e50'},
 {ratio:0.6, color:'#2980b9'},
 {ratio:0.4, color:'#e74c3c'},
 {ratio:0.2, color:'#f1c40f'}
 ];
 // Draw black border
 ctx.fillStyle='#8B4513'; ctx.beginPath(); ctx.arc(t.x,t.y,t.r+3); fill();
 for (let ring of ringDefs) { ctx.fillStyle=ring.color; ctx.beginPath(); arc(t.x,t.y,t.r*ring.ratio); fill(); }
 // highlight
 ctx.strokeStyle='rgba(255,255,255,0.3)'; lineWidth=2; arc(t.x- t.r*0.2, t.y-t.r*0.2, t.r*0.8?) no.
 if (t.hit) {
   ctx.strokeStyle='#2ecc71'; lineWidth=5; ctx.beginPath(); arc(t.x,t.y,t.r+6); stroke();
   ctx.fillStyle='rgba(0,0,0,0.35)'; ctx.beginPath(); arc(t.x,t.y,t.r*0.9); fill();
   ctx.fillStyle='#fff'; font='bold 24px sans-serif'; textAlign='center'; textBaseline='middle'; fillText('✓', t.x,t.y);
   // maybe label "已命中"
 }
}
```
Need if hit overlay not hide? If fill black 0.35 over rings, still okay. But maybe "HIT" text.

Need "target names" maybe display above? Not needed. But could draw small labels "1","2","3","4" above targets to communicate four targets. But in archery, targets have numbers. Let's add number labels:
```
ctx.fillStyle='#fff'; ctx.font='bold 16px sans-serif'; ctx.textAlign='center'; ctx.textBaseline='bottom'; ctx.fillText(t.id+1, t.x, t.y - t.r - 5);
```
Need if target far, label.

Need "draw archer":
```
function drawArcher(angle) {
 const archerX=400, archerY=540;
 ctx.save();
 ctx.translate(archerX, archerY);
 ctx.rotate(angle);
 // bow arc
 ctx.lineWidth=5;
 ctx.strokeStyle='#5d3a1a';
 ctx.beginPath();
 ctx.arc(0,0,30, -1.2, 1.2);
 ctx.stroke();
 // string
 ctx.strokeStyle='rgba(220,220,220,0.8)';
 ctx.lineWidth=1.5;
 ctx.beginPath();
 ctx.moveTo(Math.cos(-1.2)*30, Math.sin(-1.2)*30);
 ctx.lineTo(Math.cos(1.2)*30, Math.sin(1.2)*30);
 ctx.stroke();
 // maybe arrow nocked (if not shooting) draw arrow to 50?
 // restore
 ctx.restore();
}
```
Need "arrow ready" drawn from bow to crosshair maybe before shooting? Actually if no shot, arrow nocked and points toward mouse. Draw an arrow from bow center to min(60, distance) in direction. But if no arrow on bow, crosshair not. We can draw a "nocked arrow" line from bow to bow+60 along angle. But that's small.
Need maybe bow trigger? Could draw "click to shoot" text near bow.

Need "draw ready arrow" maybe:
```
function drawNockedArrow(angle) {
 const bowX=400, bowY=540;
 ctx.save(); ctx.translate(bowX, bowY); ctx.rotate(angle);
 ctx.strokeStyle='#c0392b'; lineWidth=3;
 ctx.beginPath(); ctx.moveTo(10,0); ctx.lineTo(70,0); ctx.stroke();
 // head at 78
 ctx.fillStyle='#bbb'; triangle...
 ctx.restore();
}
```
But if there is a shot in flight, no nocked arrow? Could still draw nocked arrow if not waiting? Real bow after release would be unloaded. But not necessary.

Need "shots" list maybe contains all. Draw shots:
```
function drawShots(now) {
 for (const shot of shots) {
   const p = Math.min(1, (now - shot.time) / shot.duration);
   const x = shot.startX + (shot.endX - shot.startX)*p;
   const y = shot.startY + (shot.endY - shot.startY)*p;
   const angle = Math.atan2(shot.endY - shot.startY, shot.endX - shot.startX);
   ctx.save(); ctx.translate(x,y); ctx.rotate(angle);
   ctx.strokeStyle='#8B4513'; ctx.lineWidth=3;
   ctx.beginPath(); ctx.moveTo(-28,0); ctx.lineTo(4,0); ctx.stroke();
   ctx.fillStyle='#c0c0c0'; ctx.beginPath(); ctx.moveTo(12,0); ctx.lineTo(-2,-5); ctx.lineTo(-2,5); fill();
   ctx.fillStyle='#f39c12'; // fletching
   ctx.beginPath(); ctx.moveTo(-28,0); ctx.lineTo(-38,-6); ctx.lineTo(-38,6); fill();
   ctx.restore();
 }
}
```
Need arrowhead place at x,y? Wait line from -28 to 4, head at 12; arrow center maybe x? The line doesn't extend to tip. Better: line from -30 to 0, head at 0. In current coordinate, shot position (x,y) is tip? We'll make tip at 0. Draw line from -36 to 0, head triangle at 0. Use:
```
ctx.beginPath(); ctx.moveTo(-36,0); ctx.lineTo(0,0); stroke();
ctx.beginPath(); ctx.moveTo(8,0); ctx.lineTo(-4,-5); ctx.lineTo(-4,5); fill();
# fletching at -36
ctx.beginPath(); ctx.moveTo(-36,0); ctx.lineTo(-45,-6); ctx.lineTo(-45,6); fill();
```
This arrow points in angle direction.

Need if shot p=1 and arrow hits target, arrow remains stuck in target. Draw after target maybe? If arrow drawn after target, it appears in front, okay. If arrow hit a target, it should be at the target center? Wait shot end is mouse landing point. If exactly center, arrow tip at center; but if path intersects target, arrow might stop at edge. But endpoint distance condition with landing point inside circle means arrow tip may be at click point, not at target center. For hit target, arrow is drawn at click point; if click near edge, arrow sticks in edge. Good.
Need if shot misses, arrow remains in grass? Draw at landing point in front of background but before archer? We'll draw shots after targets but before archer. If arrow in sky misses, okay.
Need "effects" maybe hit particles. Could implement simple ring explosion:
```
effects.push({x,y,time:now,type:'hit'});
```
Draw in update:
- For effects with type hit, draw expanding ring alpha. Use current time - effect.time.
- For miss, maybe crosshair "脱靶" text? Maybe status message only.

Need "draw crosshair" after shots? Should be on top unless mouse leaves. Crosshair at mouse position.

Need "statusEl" updates:
- idle: "点击“开始游戏”开始"
- playing: "游戏进行中... 已命中 X/4"
- finished: "🎉 命中全部靶子！用时 X 秒"
Need when shot hit/miss:
- Maybe statusEl temporarily "命中靶子 2/4!" or "脱靶，再试一次". But timer status? Could display on a separate message overlay? We'll use `statusEl.textContent` and perhaps revert? But if we set to hit/miss, we lose hit count. We can maintain `statusEl` with "已命中 X/4" always by default. Need hit feedback maybe via effects and target green. Maybe no need status message on every shot. But "脱靶" could be useful. We can implement a `toast` message displayed in status for 1s. Need avoid conflict.

Simpler: statusEl is "已命中 X/4 | 用时 ..." Maybe no toast. Add an overlay text on canvas for "脱靶" / "命中" using effects. Need maybe in draw:
```
function drawFloatingTexts(now) { ... }
```
Effect includes text. On shot resolve:
- If hit: effect {type:'text', x:endX, y:endY-40, text:'命中!', color:'#2ecc71', time:now}
- If miss: effect {type:'text', x:endX, y:endY-20, text:'脱靶', color:'#e74c3c', time:now}
Draw floating text rising/fading over 0.8s.
Need draw effects:
- Ring around target:
```
if (effect.type==='ring') { const age=now-effect.time; if age<0.5; radius=age*150; alpha=1-age/0.5; ctx.strokeStyle...; circle }
```
- Text:
```
if age<1; alpha = 1-age; y = e.y - age*40; font bold 20.
```
Need store effect.time as performance.now.

Need "hit detection" perhaps should happen at shot resolution time `now` not at click time. Need if shot resolves after target hit by earlier shot, and end point within target but already hit, no count. We check `if (!t.hit && distance <= t.r*??)`. Need tolerance maybe radius. But if target has rings; click on target circle. Need maybe target radius t.r. Use `distance <= t.r`. Good. But if click at edge, target hit. Since target boards have outer ring white; yes.

Need maybe "hit all targets" if one shot simultaneously hits same target? impossible.

Need "targets" maybe have `hit` property. "Create four targets" yes.

Need "background drawing" with horizon, sky, grass. Need maybe draw a lane line from archer to target? Could draw shooting line at y=550:
```
ctx.strokeStyle='rgba(255,255,255,0.6)'; lineWidth=3;
ctx.beginPath(); moveTo(100,550); lineTo(700,550); stroke();
```
Need draw "archer" at 550.

Need "arrow fly" from archer to click; but if target clicked, arrow flies from bottom to top; if clicked below archer? If user clicks below archer, arrow flies downward. But targets only above. Could allow but no hit. The bow can rotate any angle. Need maybe restrict clicks to "range" above shooting line? Not needed. If click below, arrow lands in grass. But crosshair can move anywhere. Fine.
Need "startBtn" click handler:
```
startBtn.addEventListener('click', () => {
  resetGame();
  state = 'playing';
  startTime = performance.now();
  startBtn.textContent = '重新开始';
  statusEl.textContent = '游戏进行中... 已命中 0/4';
});
```
Need if state playing, clicking start resets without confirmation. Okay.
Need "clear scores" button:
```
clearBtn.addEventListener('click', () => {
 if (confirm('确定清空排行榜吗？')) { localStorage.removeItem(STORAGE_KEY); lastPlayId=null; renderLeaderboard(); }
});
```
Need maybe in final, confirm if not dangerous? okay.

Need "RAF loop":
```
function loop(now) {
 if (!lastFrameTime) lastFrameTime = now;
 if (state === 'playing') {
   elapsedTime = (now - startTime) / 1000;
   timerEl.textContent = `用时: ${elapsedTime.toFixed(2)}s`;
 }
 updateShots(now);
 updateEffects(now);
 draw(now);
 rafId = requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```
Need not use dt; update shots based on now.

Need `startTime` is performance.now at click even if loop started? Good. RAF `now` used in timer; if startTime is performance.now, now-startTime is correct. Both are same timebase.

Need `resetGame()`:
```
targets = createTargets();
shots = [];
effects = [];
elapsedTime = 0;
timerEl.textContent = '用时: 0.00s';
lastPlayId = null;
```
Need not reset state there.

Need `canvas click`:
```
canvas.addEventListener('click', (e) => {
 const rect=canvas.getBoundingClientRect();
 const x=(e.clientX-rect.left)*canvas.width/rect.width;
 const y=(e.clientY-rect.top)*canvas.height/rect.height;
 if (state !== 'playing') {
   statusEl.textContent = state==='finished' ? '已结束，点击“重新开始”再来一局' : '请先点击“开始游戏”';
   return;
 }
 // create shot
 const duration = Math.min(0.5, 0.15 + Math.hypot(x-bowX,y-bowY)/2500);
 shots.push({startX:bowX, startY:bowY, endX:x, endY:y, time:performance.now(), duration, resolved:false});
});
```
Need if click when arrow in flight, multiple shots. Fine.
Need if state finished, canvas click says "click button". Good.

Need `updateShots(now)`:
```
for (const shot of shots) {
 if (shot.resolved) continue;
 if (now - shot.time >= shot.duration) {
   shot.resolved = true;
   resolveShot(shot, now);
 }
}
```
Need maybe if multiple shots resolve in same frame, order by shot.time ascending. Loop shots in chronological order. If target hit by first, second can hit others. Fine.
Need `resolveShot`:
```
function resolveShot(shot, now) {
 let hit = false;
 for (const t of targets) {
   if (t.hit) continue;
   const d = Math.hypot(shot.endX - t.x, shot.endY - t.y);
   if (d <= t.r) {
     t.hit = true;
     hit = true;
     effects.push({type:'ring', x:t.x, y:t.y, time:now, color:'#2ecc71'});
     effects.push({type:'text', x:t.x, y:t.y - t.r - 10, text:'命中!', color:'#2ecc71', time:now});
     break;
   }
 }
 if (!hit) {
   effects.push({type:'text', x:shot.endX, y:shot.endY - 15, text:'脱靶', color:'#e74c3c', time:now});
 }
 if (targets.every(t=>t.hit)) {
   finishGame(now);
 }
 // status update
 const hitCount = targets.filter(t=>t.hit).length;
 if (state === 'playing') statusEl.textContent = `游戏进行中… 已命中 ${hitCount}/4`;
}
```
Need if arrow "hit" target but target already hit, it would be "miss" if using continue. But if arrow lands on already hit target, should not count and maybe no "脱靶"? It's a hit on a target, but not needed. Could check if within any target (including hit) to decide whether "脱靶".:
```
let inAnyTarget = false;
for (const t of targets) {
  const d=...
  if (d <= t.r) {
    inAnyTarget=true;
    if (!t.hit) { t.hit=true; hit=true; ... break; }
  }
}
if (!hit && !inAnyTarget) effects.push(text '脱靶');
```
If hits already hit target, perhaps no "脱靶" text; but no need. Let's implement:
```
let hitNew = false;
let hitAny = false;
for (const t of targets) {
 const d=...
 if (d <= t.r) {
   hitAny = true;
   if (!t.hit) { t.hit = true; hitNew = true; effect...; break; }
 }
}
if (!hitAny) { effects.push({type:'text', text:'脱靶', ...}); }
```
If hit old target, effects maybe "已命中过" text? Could ignore. But if click old target, no feedback. Maybe add text "已命中过" if hitAny && !hitNew. Good.
Need if all targets hit after finish, status override. Good.

Need `finishGame(now)`:
```
function finishGame(now) {
 if (state !== 'playing') return;
 state = 'finished';
 elapsedTime = (now - startTime) / 1000;
 timerEl.textContent = `用时: ${elapsedTime.toFixed(2)}s`;
 statusEl.textContent = `🎉 命中全部靶子！用时 ${elapsedTime.toFixed(2)} 秒`;
 startBtn.textContent = '再来一局';
 const scores = loadScores();
 const id = Date.now().toString(36) + Math.random().toString(36).slice(2,7);
 const entry = { id, time: elapsedTime, date: new Date().toLocaleString() };
 scores.push(entry);
 scores.sort((a,b) => a.time - b.time || (a.id.localeCompare(b.id)));
 saveScores(scores);
 lastPlayId = id;
 renderLeaderboard();
}
```
Need note `elapsedTime` uses `now` from RAF, maybe same as performance.now. Good.

Need `renderLeaderboard()`:
```
function renderLeaderboard() {
 const scores=loadScores();
 tbody.innerHTML='';
 if (!scores.length) { row...; return; }
 const top=scores.slice(0,10);
 top.forEach((s,i)=>{
   const tr=document.createElement('tr');
   if (s.id===lastPlayId) tr.className='highlight';
   const rank=i+1;
   const rankText = rank===1?'🥇 1':rank===2?'🥈 2':rank===3?'🥉 3':rank;
   tr.innerHTML = `<td>${rankText}</td><td>${s.time.toFixed(2)}s</td><td>${s.date}</td><td>${s.id===lastPlayId?'⭐ 本次':''}</td>`;
   tbody.appendChild(tr);
 });
}
```
Need if many entries, top 10, but "名次越高" If newer attempt not top10, no highlight. Could also display "当前名次：第 X 名" in status? Hmm. The leaderboard only top 10; but rank should be among all. Need maybe determine and display. We can compute rank of last play and store? In finishGame, after sort, `rank = scores.findIndex(s=>s.id===id)+1;` if within top 100. Status could include `当前排名：第 ${rank} 名`. But leaderboard table top 10 may not show if >10. Could include "最佳成绩" and "本次排名". Let's add in statusEl:
```
statusEl.textContent = `🎉 命中全部靶子！用时 ${time.toFixed(2)} 秒，本次排名第 ${rank} 名`;
```
Need compute rank after save. If scores length >100 and entry not kept, index -1? We'll keep all? Let's keep all scores in localStorage (no cap). But could grow; okay for game. If user clears, fine. We'll cap maybe 200. But rank if >200 not shown? Hmm. We'll cap at 100 and if entry not in list, rank = 101? Not good. Let's protect by keeping all attempts. Storage likely fine. We'll not cap. But leaderboard table top 10 only. Good.

Need "all attempts" local; if too many, storage maybe okay (small). Good.

Need `current rank` after finish:
```
const rank = scores.findIndex(s => s.id === id) + 1;
statusEl.textContent = `🎉 命中全部靶子！用时 ${time.toFixed(2)} 秒，本次排名第 ${rank} 名`;
```
Need "rank" if tie? Sorted by time then id. Good.

Need "best time" maybe display in leaderboard header. Fine.

Need "update effects" no need if effects array too big; filter:
```
function updateEffects(now) {
 effects = effects.filter(e => now - e.time < 1200);
}
```
Need `effects` const? Use `let effects = []`.

Need `draw`:
```
function draw(now) {
 ctx.clearRect(0,0,800,600);
 drawBackground();
 // draw target stands far to near
 const sortedTargets = [...targets].sort((a,b)=>a.y - b.y);
 for (const t of sortedTargets) drawTarget(t);
 drawShots(now);
 drawEffects(now);
 drawBowAndArrow(now);
 drawCrosshair();
}
```
Need `drawBowAndArrow` uses mouse angle and maybe draw ready arrow. However if there are shots, bow still has arrow. Could draw bow, no ready arrow? If state idle, no. We can always draw bow; but arrow maybe if playing and no arrow? In a game, you have unlimited arrows, so an arrow is always nocked. Draw an arrow on bow from bow position to bow+60 in angle direction. But if we draw it, multiple arrows maybe weird. It's okay.

Need `drawBowAndArrow`:
```
function drawBowAndArrow(now) {
 const angle = Math.atan2(mouse.y - bowY, mouse.x - bowX);
 ctx.save();
 ctx.translate(bowX, bowY);
 ctx.rotate(angle);
 // draw nocked arrow if state === 'playing' or idle? show always
 ctx.strokeStyle = '#8b4513'; ctx.lineWidth=3;
 ctx.beginPath(); ctx.moveTo(10,0); ctx.lineTo(70,0); ctx.stroke();
 ctx.fillStyle='#c0c0c0'; // head
 ctx.beginPath(); ctx.moveTo(78,0); ctx.lineTo(64,-5); ctx.lineTo(64,5); fill();
 ctx.fillStyle='#e74c3c'; // fletching
 ctx.beginPath(); ctx.moveTo(10,0); ctx.lineTo(-2,-6); ctx.lineTo(-2,6); fill();
 // bow arc
 ctx.lineWidth=5; ctx.strokeStyle='#5d3a1a';
 ctx.beginPath(); ctx.arc(0,0,30, -1.2, 1.2); ctx.stroke();
 // string
 ctx.lineWidth=1.5; ctx.strokeStyle='#aaa';
 ctx.beginPath();
 const ax = Math.cos(-1.2)*30, ay=Math.sin(-1.2)*30;
 const bx = Math.cos(1.2)*30, by=Math.sin(1.2)*30;
 ctx.moveTo(ax,ay); ctx.lineTo(bx,by); ctx.stroke();
 ctx.restore();
}
```
But arrow from 10 to 70 and bow arc at center 30 maybe arrow stuck through bow. If bow arc is on right side, arrow should pass through center of bow? Let's visualize: With angle to mouse, bow center at (0,0), arrow points positive x from 10 to 70, bow arc is a vertical-ish arc at x ~11 to 30? Actually arc radius 30 from -1.2 to 1.2: points from (10.9,-28) to (10.9,28), so the bow string/limb is at x=10.9; arrow starts at 10 and passes through/behind bow. Could be okay. But arrowhead at 78 beyond bow. Need draw arrow first (behind bow) maybe. Good.
Need "bow arc" maybe too small; okay.

Need "state idle" show "开始游戏"; crosshair still. Good.

Need "canvas cursor" maybe crosshair CSS? We can set `cursor: crosshair;` on canvas. But our draw crosshair too. Use `cursor: crosshair`. Good.

Need "four targets" maybe target labels 1-4. Good.

Need "Leaderboard rank" maybe "排行榜" panel.

Need "test instructions" in final maybe include exact code. But before final, let's refine.

Need ensure code valid. Let's draft code fully.

Potential issue with `effects` variable defined with `let` and reassigned in updateEffects. In draw, effects filtered. Need no error.

Potential issue with `startTime` initial 0 and timer with state idle not update. Good.

Potential issue with `lastFrameTime` not used; remove.

Potential issue `statusEl.textContent` after hit count update: if finishGame sets state finished, then update after? In resolveShot:
```
if (!hitAny) ...;
const hitCount = ...;
if (state === 'playing') statusEl.textContent = ...;
if (targets.every(...)) finishGame(now);
```
Need order: Should finishGame set status after. If we set status "已命中 4/4" before finishGame, finishGame overrides. Fine.
But if all targets hit, finishGame called; after return, no more. Good.

Need maybe "finishGame" called after `state` is playing. Good.

Need if multiple shots resolve in same frame and all targets hit after first; finishGame sets state finished, but loop continues resolving later shots. `resolveShot` for later shots will check state maybe? Need if later shot when state finished, do we still resolve? `state === 'playing'` check? In `updateShots`, for each resolved false, if time reached, call resolveShot regardless. If state finished, resolveShot will still process and might add effects, but if all targets hit, no need. Could skip if state !== 'playing':
```
if (shot.resolved) continue;
if (now - shot.time >= shot.duration) {
 shot.resolved = true;
 if (state === 'playing') resolveShot(shot, now);
}
```
If shot was in flight when game finished by an earlier shot, it will just disappear. Okay. But arrow visual should continue? If skipped, arrow not drawn? It remains at path before finish maybe. But finish state stops? Actually loop continues drawing shots with p computed; if resolved true and p=1 (or maybe if now >= duration), draw arrow at end. If state finished, it will draw arrow at end but no resolve. Fine. We can still resolve? If state finished, no effect. Let's implement:
```
if (shot.resolved) continue;
if (now - shot.time >= shot.duration) {
  shot.resolved = true;
  if (state === 'playing') resolveShot(shot, now);
}
```
Need if target hit all, `finishGame` called inside resolveShot. Good.

Need "drawShots" for resolved shots with p=1:
```
const p = shot.resolved ? 1 : Math.min(1, (now - shot.time)/shot.duration);
```
If state finished and shot not resolved, p reaches 1 eventually. Good.

Need `resolveShot` should perhaps not call finishGame inside loop before all shots resolved? okay.

Need "hit target" if click at target center; arrow duration from archer to target; target hit after duration. Good.

Need "target radius" values maybe too small? Need consider canvas 800x600. Far target r=35, near r=50. Good. But click on target board with radius 35 is okay. Need maybe ensure target stands don't overlap. Far targets y=240; near targets y=420. Far target board bottom = 275; near target board center=420, but legs from near target bottom to y=520, so may overlap with far target? On screen, far target at x=170; near target at x=120; no overlap horizontally. But far target legs extend to y=240 + 70=310? Wait legs t.y + t.r*2.0 = 240+70=310, near target top = 420-50=370. No overlap. Good.
Need "far target" label maybe above y=190. Good.

Need "drawBackground" horizon at y=200 maybe. Target y=240 on grass. Need target stands on grass. Good.

Need "ground" gradient from 200 to 600. Nice.

Need "shadow" for target at y+t.r*1.1? For far target with r=35, shadow at 278. Good.

Need "archer" at bottom center y=540; near target legs to y=420+100=520, so archer at 540 after legs, good. But near target at y=420 with r=50, board bottom=470, legs to 520. Archer at 540. Good.

Need "crosshair" drawn always; if mouse.inside false, maybe not draw. But at start mouse x,y default; draw crosshair maybe. Need track mouseenter/leave. Simpler draw crosshair if mouse.inside. But if mouse leaves, no visible. Good.

Need "click" coordinates if mouse not inside? Click requires inside. Good.

Need "target hit" if arrow lands within radius. But if target circle drawn with labels? okay.

Need "leaderboard date" formatting maybe "2026/5/1 12:00:00". Good.
Need `new Date(a.date)` not used. Good.

Need "sort by time ascending" if equal by id maybe okay.
Need "loadScores" returns array. If corrupt, catch.

Need "clear scores" may not remove in-memory if lastPlayId still and render? It will render empty. Good.
Need if `lastPlayId` references old score and after clear scores, not set null. In render no scores. Fine. If later new finish and scores empty, okay. But if clear then render no highlight. Fine.
Need "renderLeaderboard" called at startup. Good.

Need "instructions" text maybe:
"点击“开始游戏”后，移动鼠标瞄准靶子并点击射击。命中全部 4 个靶子计时结束。"
Need maybe "Arrow" flies toward crosshair; if target hit, target turns green. Good.

Need "No external files" satisfied.

Potential issue: "排行榜上的名次就越高" maybe means if shorter time, top rank. We can include "排行榜按用时从短到长排序" in UI.

Need "test notes" maybe say "打开 HTML 后，点击开始游戏，依次射击四个靶；计时结束后检查排行榜；刷新页面后成绩仍然保留". Good.

Need maybe "DOCTYPE" etc.

Let's think about "simulation game" maybe need "弓箭射击" not just mouse click. Could use drag to pull bow? But user didn't specify. But "射击模拟" maybe requires aiming. Could include "按住鼠标拉弓，松开射击" for more simulation? Hmm. The requirement "玩家命中所有目标所用的时间越短" suggests speed. If player needs drag and release, time includes aiming and shooting. But clicking is simpler. Could we implement "click to shoot" enough? Maybe yes.

However, "弓箭射击模拟游戏" could be expected more immersive. Maybe implement "按住鼠标瞄准，松开射击"? But if using time, click is okay. Let's consider if "弓箭射击" needs arrow trajectory with gravity. Maybe not. We could implement a more realistic bow: hold mouse to draw, release to shoot; crosshair while holding maybe. But then timer includes draw time. Need not.

Maybe include "弓的力量条" by holding? But target hit based on crosshair and release. If hold to draw, no arrow flight? We can add but complexity. The requirement doesn't demand. Should we add "按住鼠标左键拉弓，松开射击" instead of click. This would be more simulation-like. But then "time" includes draw time; leaderboard more skill. Yet if user clicks, it's just a point-and-click. Hmm.

Let's evaluate. The phrase "弓箭射击模拟游戏" and "玩家命中所有目标所用的时间越短" suggests an arcade shooting game. Clicking target to shoot is okay. But "模拟" maybe not. We can implement a hybrid: `mousedown` starts drawing (pull arrow back), `mouseup` releases; while drawing, a small power meter increases. But if timing matters, holding too long increases time; maybe not needed. Need arrow should fly to crosshair when released. But if target hit based on release crosshair, no issue. It adds physical feel. But the instructions "不要查看其他文件" maybe just want code not overcomplicate. We can implement "点击发射" but mention "移动鼠标瞄准，点击射击". That's acceptable.

Could maybe add "按住鼠标拉弓，松开射箭" with change minimal:
- `mousedown` on canvas sets `drawing=true`, `drawStartTime=performance.now()`, not shoot.
- `mouseup` shoots if state playing. But then timer starts at start button. The arrow is released on mouseup. Need visual draw bow while drawing. More complexity but manageable. Need "time" includes pull duration. But user may find holding awkward. Need maybe allow quick clicks? If mousedown immediately mouseup, arrow shoots. Good. But if user wants to click quickly, it works. Need game may feel more archery.

However, with current click handler, clicking is straightforward. We can add "按住拖动拉弓？" Maybe no.

Need maybe "shooting" with "click" can be interpreted as click release where arrow shoots automatically. Fine.

Need perhaps "archery" should have bow at bottom and target; clicking is release. Good.

Let's decide final code with click only to keep code robust and shorter. But maybe include `mousedown`/`mouseup`? Let's think if final answer with click only might be considered insufficient? The user didn't require drag. It says "弓箭射击模拟游戏" not "点击射击". But many simple HTML games use click. Should be okay.

Need maybe include "发射" by pressing space? no.

Need maybe "all targets" hit count maybe 4. Good.

Let's refine code now.

Full JS:

```
'use strict';
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const startBtn = document.getElementById('startBtn');
const timerEl = document.getElementById('timer');
const statusEl = document.getElementById('status');
const leaderboardBody = document.getElementById('leaderboardBody');
const clearScoresBtn = document.getElementById('clearScoresBtn');

const STORAGE_KEY = 'archery_leaderboard_v1';
const BOW_X=400, BOW_Y=545;

let state='idle';
let startTime=0;
let elapsedTime=0;
let targets=createTargets();
let shots=[];
let effects=[];
let lastPlayId=null;
let mouse={x:400,y:300,inside:false};

function createTargets() { ... }
function loadScores() { ... }
function saveScores(scores) { ... }

function startGame() {
  targets = createTargets();
  shots = [];
  effects = [];
  elapsedTime = 0;
  lastPlayId = null;
  timerEl.textContent = '用时: 0.00s';
  state = 'playing';
  startTime = performance.now();
  startBtn.textContent = '重新开始';
  statusEl.textContent = '游戏进行中… 已命中 0/4';
  renderLeaderboard();
}

startBtn.addEventListener('click', startGame);

clearScoresBtn.addEventListener('click', function() { ... });

canvas.addEventListener('mousemove', (e) => { ...; mouse.inside=true; });
canvas.addEventListener('mouseleave', () => { mouse.inside=false; });
canvas.addEventListener('click', (e) => { ... });

function shootAt(x,y) {
 if (state !== 'playing') return;
 const dx=x-BOW_X, dy=y-BOW_Y;
 const dist=Math.hypot(dx,dy);
 const duration=Math.min(0.5, 0.15 + dist/2500);
 shots.push({startX:BOW_X, startY:BOW_Y, endX:x, endY:y, time:performance.now(), duration, resolved:false});
}
```
Need if user clicks while mouse.inside false? Click event always has coords. Good.

Need "shootAt" maybe if click on canvas idle, status message:
```
if (state === 'idle') {
 statusEl.textContent = '请先点击“开始游戏”';
 return;
}
if (state === 'finished') {
 statusEl.textContent = '本轮已结束，点击“重新开始”再来一局';
 return;
}
shootAt(x,y);
```
Need if click just after finish? state finished, no shoot.

Need `mouse` angle uses mouse.x/y. If mouse not inside, use default. Good.

Need `updateShots`:
```
function updateShots(now) {
 for (const shot of shots) {
   if (shot.resolved) continue;
   if (now - shot.time >= shot.duration) {
     shot.resolved = true;
     if (state === 'playing') resolveShot(shot, now);
   }
 }
}
```
Need `resolveShot`:
```
function resolveShot(shot, now) {
  let hitAny = false;
  let hitNew = false;
  for (const t of targets) {
    const d = Math.hypot(shot.endX - t.x, shot.endY - t.y);
    if (d <= t.r) {
      hitAny = true;
      if (!t.hit) {
        t.hit = true;
        hitNew = true;
        effects.push({type:'ring', x:t.x, y:t.y, time:now});
        effects.push({type:'text', x:t.x, y:t.y - t.r - 12, text:'命中!', color:'#2ecc71', time:now});
      }
      break;
    }
  }
  if (hitNew) {
    const hitCount = targets.filter(t => t.hit).length;
    if (state === 'playing') statusEl.textContent = `游戏进行中… 已命中 ${hitCount}/4`;
    if (targets.every(t => t.hit)) {
      finishGame(now);
    }
  } else if (hitAny) {
    effects.push({type:'text', x:shot.endX, y:shot.endY - 15, text:'已命中过', color:'#f1c40f', time:now});
  } else {
    effects.push({type:'text', x:shot.endX, y:shot.endY - 15, text:'脱靶', color:'#e74c3c', time:now});
  }
}
```
Need if hitNew false and hitAny true but shot hit old target while not all hit, status remains. Good.
Need if hitNew true but not all, status updates. If all, finishGame override.

Need if a shot misses, status should remain; good.

Need `finishGame`:
```
function finishGame(now) {
 state='finished';
 elapsedTime = (now - startTime) / 1000;
 timerEl.textContent = `用时: ${elapsedTime.toFixed(2)}s`;
 const scores = loadScores();
 const id = Date.now().toString(36) + Math.random().toString(36).slice(2,7);
 scores.push({ id, time: elapsedTime, date: new Date().toLocaleString() });
 scores.sort((a,b) => a.time - b.time || a.id.localeCompare(b.id));
 saveScores(scores);
 lastPlayId = id;
 const rank = scores.findIndex(s => s.id === id) + 1;
 statusEl.textContent = `🎉 命中全部靶子！用时 ${elapsedTime.toFixed(2)} 秒，本次排名第 ${rank} 名`;
 startBtn.textContent = '再来一局';
 renderLeaderboard();
}
```
Need if `elapsedTime` toFixed maybe 0.00? possible if programmatic? okay.
Need "rank" if scores empty impossible. Good.

Need `renderLeaderboard`:
```
function renderLeaderboard() {
 leaderboardBody.innerHTML='';
 const scores=loadScores();
 if (scores.length===0) {
  const tr=document.createElement('tr');
  tr.innerHTML='<td colspan="4" class="empty">暂无记录</td>';
  leaderboardBody.appendChild(tr);
  return;
 }
 scores.slice(0,10).forEach((s,i)=>{
  const tr=document.createElement('tr');
  if (s.id===lastPlayId) tr.classList.add('highlight');
  let rankText = `${i+1}`;
  if (i===0) rankText='🥇 1';
  else if (i===1) rankText='🥈 2';
  else if (i===2) rankText='🥉 3';
  const isNew = s.id===lastPlayId ? '⭐ 本次' : '';
  tr.innerHTML = `<td>${rankText}</td><td>${s.time.toFixed(2)}s</td><td>${s.date}</td><td>${isNew}</td>`;
  leaderboardBody.appendChild(tr);
 });
}
```
Need if lastPlayId not in top10, no highlight. But status shows rank. Good.
Need maybe "排行榜按用时从短到长排列，最多显示前10名" in panel header. Good.

Need "Draw" functions.

Let's write draw code carefully.

`drawBackground`:
```
function drawBackground() {
 const sky = ctx.createLinearGradient(0,0,0,210);
 sky.addColorStop(0,'#4a90d9');
 sky.addColorStop(1,'#bcdff1');
 ctx.fillStyle=sky;
 ctx.fillRect(0,0,800,220);
 // sun
 ctx.fillStyle='#ffe066';
 ctx.beginPath();
 ctx.arc(700,55,28);
 ctx.fill();
 // clouds maybe
 ctx.fillStyle='rgba(255,255,255,0.7)';
 ctx.beginPath();
 ctx.ellipse(120,60,40,22,0,0,Math.PI*2);
 ...
 // ground
 const ground = ctx.createLinearGradient(0,210,0,600);
 ground.addColorStop(0,'#6f9e4f');
 ground.addColorStop(1,'#3e6b2e');
 ctx.fillStyle=ground;
 ctx.fillRect(0,210,800,390);
 // horizon line
 ctx.strokeStyle='#2f5d27';
 ctx.lineWidth=3;
 ctx.beginPath();
 ctx.moveTo(0,210);
 ctx.lineTo(800,210);
 ctx.stroke();
 // shooting line
 ctx.strokeStyle='rgba(255,255,255,0.35)';
 ctx.lineWidth=3;
 ctx.beginPath();
 ctx.moveTo(50, BOW_Y+5);
 ctx.lineTo(750, BOW_Y+5);
 ctx.stroke();
}
```
Need if BOW_Y=545, line below archer. Fine.

Need `drawTarget(t)`:
```
function drawTarget(t) {
 const r = t.r;
 // shadow
 ctx.fillStyle='rgba(0,0,0,0.18)';
 ctx.beginPath();
 ctx.ellipse(t.x, t.y + r*1.1, r*1.5, r*0.35, 0, 0, Math.PI*2);
 ctx.fill();
 // legs
 ctx.strokeStyle='#5d3a1a';
 ctx.lineWidth=5;
 ctx.lineCap='round';
 ctx.beginPath();
 ctx.moveTo(t.x - r*0.55, t.y + r*0.8);
 ctx.lineTo(t.x - r*1.3, t.y + r*2.0);
 ctx.moveTo(t.x + r*0.55, t.y + r*0.8);
 ctx.lineTo(t.x + r*1.3, t.y + r*2.0);
 ctx.stroke();
 // board
 ctx.fillStyle='#8b5a2b';
 ctx.beginPath();
 ctx.arc(t.x, t.y, r+3, 0, Math.PI*2);
 ctx.fill();
 const rings = [
 {ratio:1.0,color:'#ffffff'},
 {ratio:0.8,color:'#2c3e50'},
 {ratio:0.6,color:'#2980b9'},
 {ratio:0.4,color:'#dd2c2c'},
 {ratio:0.2,color:'#f1c40f'}
 ];
 for (const ring of rings) {
   ctx.fillStyle=ring.color;
   ctx.beginPath();
   ctx.arc(t.x, t.y, r*ring.ratio, 0, Math.PI*2);
   ctx.fill();
 }
 // hit overlay
 if (t.hit) {
   ctx.strokeStyle='#2ecc71';
   ctx.lineWidth=5;
   ctx.beginPath();
   ctx.arc(t.x, t.y, r+8, 0, Math.PI*2);
   ctx.stroke();
   ctx.fillStyle='rgba(0,0,0,0.35)';
   ctx.beginPath();
   ctx.arc(t.x, t.y, r*0.95, 0, Math.PI*2);
   ctx.fill();
   ctx.fillStyle='#2ecc71';
   ctx.font=`bold ${Math.round(r*0.8)}px sans-serif`;
   ctx.textAlign='center';
   ctx.textBaseline='middle';
   ctx.fillText('✓', t.x, t.y);
 }
 // label
 ctx.fillStyle='rgba(255,255,255,0.9)';
 ctx.font='bold 14px sans-serif';
 ctx.textAlign='center';
 ctx.textBaseline='bottom';
 ctx.fillText(`靶 ${t.id+1}`, t.x, t.y - r - 6);
}
```
Need if target far, font 14. Good.

Need `drawShots(now)`:
```
function drawShots(now) {
 for (const shot of shots) {
   const p = shot.resolved ? 1 : Math.min(1, (now - shot.time) / shot.duration);
   const x = shot.startX + (shot.endX - shot.startX) * p;
   const y = shot.startY + (shot.endY - shot.startY) * p;
   const angle = Math.atan2(shot.endY - shot.startY, shot.endX - shot.startX);
   ctx.save();
   ctx.translate(x,y);
   ctx.rotate(angle);
   ctx.lineWidth=3;
   ctx.strokeStyle='#7a4a21';
   ctx.beginPath();
   ctx.moveTo(-36,0);
   ctx.lineTo(0,0);
   ctx.stroke();
   // arrowhead
   ctx.fillStyle='#c0c0c0';
   ctx.beginPath();
   ctx.moveTo(10,0);
   ctx.lineTo(-4,-5);
   ctx.lineTo(-4,5);
   ctx.closePath();
   ctx.fill();
   // fletching
   ctx.fillStyle='#e74c3c';
   ctx.beginPath();
   ctx.moveTo(-36,0);
   ctx.lineTo(-46,-6);
   ctx.lineTo(-46,6);
   ctx.closePath();
   ctx.fill();
   ctx.restore();
 }
}
```
Need if p=0 at start, arrow at bow; if shot time exactly now. Good.

Need `drawEffects(now)`:
```
function drawEffects(now) {
 for (const effect of effects) {
   const age = now - effect.time;
   if (effect.type === 'ring') {
     if (age > 0.6) continue;
     const progress = age / 0.6;
     const radius = 10 + progress * 60;
     const alpha = 1 - progress;
     ctx.strokeStyle = `rgba(46,204,113,${alpha})`;
     ctx.lineWidth = 4;
     ctx.beginPath();
     ctx.arc(effect.x, effect.y, radius, 0, Math.PI*2);
     ctx.stroke();
   } else if (effect.type === 'text') {
     if (age > 1.0) continue;
     const alpha = 1 - age;
     ctx.globalAlpha = Math.max(0, alpha);
     ctx.font = 'bold 22px sans-serif';
     ctx.textAlign = 'center';
     ctx.textBaseline = 'middle';
     ctx.fillStyle = effect.color || '#fff';
     ctx.fillText(effect.text, effect.x, effect.y - age * 35);
     ctx.globalAlpha = 1;
   }
 }
}
```
Need updateEffects filter maybe:
```
function updateEffects(now) {
 effects = effects.filter(e => now - e.time < 1100);
}
```
Need if ring effect age >600 filtered after 1100; okay.

Need `drawBow(now)`:
```
function drawBow() {
 const angle = Math.atan2(mouse.y - BOW_Y, mouse.x - BOW_X);
 ctx.save();
 ctx.translate(BOW_X, BOW_Y);
 ctx.rotate(angle);
 // draw nocked arrow
 ctx.lineWidth = 3.5;
 ctx.strokeStyle = '#7a4a21';
 ctx.beginPath();
 ctx.moveTo(18,0);
 ctx.lineTo(74,0);
 ctx.stroke();
 ctx.fillStyle = '#c0c0c0';
 ctx.beginPath();
 ctx.moveTo(84,0);
 ctx.lineTo(68,-6);
 ctx.lineTo(68,6);
 ctx.closePath();
 ctx.fill();
 ctx.fillStyle = '#e74c3c';
 ctx.beginPath();
 ctx.moveTo(18,0);
 ctx.lineTo(6,-6);
 ctx.lineTo(6,6);
 ctx.closePath();
 ctx.fill();
 // bow limbs
 ctx.strokeStyle = '#5d3a1a';
 ctx.lineWidth = 5;
 ctx.beginPath();
 ctx.arc(0,0,32, -1.2, 1.2);
 ctx.stroke();
 // string
 ctx.strokeStyle = '#ddd';
 ctx.lineWidth = 1.5;
 ctx.beginPath();
 ctx.moveTo(Math.cos(-1.2)*32, Math.sin(-1.2)*32);
 ctx.lineTo(Math.cos(1.2)*32, Math.sin(1.2)*32);
 ctx.stroke();
 // grip
 ctx.fillStyle='#8b5a2b';
 ctx.beginPath();
 ctx.arc(0,0,6,0,Math.PI*2);
 ctx.fill();
 ctx.restore();
}
```
Need problem: arrow at 18-74 and bow arc at 32 from x~11.7 to 32? Actually arc at x=11.7; arrow starts at 18, okay. The bow limbs are a curve from (11.7,-29.7) to (11.7,29.7) but radius 32; arc center at origin, endpoints at x=11.7. So the bow is at x=11.7, slightly behind arrow. Arrow passes through. Good.

Need if arrow line from 18 to74 and arrowhead at84, fletching at6; okay.

Need `drawCrosshair`:
```
function drawCrosshair() {
 if (!mouse.inside) return;
 const x=mouse.x, y=mouse.y;
 ctx.save();
 ctx.strokeStyle='rgba(255,255,255,0.95)';
 ctx.lineWidth=1.5;
 ctx.beginPath();
 ctx.arc(x,y,10,0,Math.PI*2);
 ctx.stroke();
 ctx.beginPath();
 ctx.moveTo(x-16,y); ctx.lineTo(x-5,y);
 ctx.moveTo(x+5,y); ctx.lineTo(x+16,y);
 ctx.moveTo(x,y-16); ctx.lineTo(x,y-5);
 ctx.moveTo(x,y+5); ctx.lineTo(x,y+16);
 ctx.stroke();
 ctx.fillStyle='rgba(255,255,255,0.9)';
 ctx.beginPath(); ctx.arc(x,y,1.5,0,Math.PI*2); ctx.fill();
 ctx.restore();
}
```
Need crosshair maybe red for contrast on white target? White with black outline? Use shadow? We can add black outline:
```
ctx.shadowColor='rgba(0,0,0,0.8)'; ctx.shadowBlur=4; ...
```
Need no.

Need "draw" order: Background, targets, shots, effects, bow, crosshair. Bow before crosshair. But if arrow shot, nocked arrow still visible. Good.

Need "canvas" width/height attributes. Good.

Need "responsive" CSS. Let's write:

```
* { box-sizing: border-box; }
body {
 margin:0; font-family:'Segoe UI', Arial, sans-serif;
 background:#1e2a34; color:#eee;
 min-height:100vh; display:flex; justify-content:center; align-items:center;
}
#app { width:100%; max-width:1200px; padding:20px; }
h1 { text-align:center; color:#ffd700; margin:0 0 15px; }
#main { display:flex; gap:20px; flex-wrap:wrap; justify-content:center; }
#gamePanel { flex:1 1 820px; max-width:820px; background:#2a3945; border-radius:12px; padding:15px; box-shadow:0 4px 20px rgba(0,0,0,0.4); }
#topBar { display:flex; align-items:center; gap:15px; margin-bottom:10px; flex-wrap:wrap; }
button { background:#f1c40f; border:none; color:#1e2a34; font-weight:bold; padding:8px 16px; border-radius:6px; cursor:pointer; font-size:15px; }
button:hover { background:#ffd700; }
#timer { font-size:18px; font-weight:bold; color:#fff; }
#status { font-size:14px; color:#ddd; flex:1; text-align:right; }
canvas { width:100%; height:auto; background:#3e6b2e; border-radius:8px; display:block; cursor:crosshair; }
#instructions { font-size:13px; color:#aaa; margin-top:8px; text-align:center; }
#leaderboardPanel { flex:0 0 320px; background:#2a3945; border-radius:12px; padding:15px; box-shadow:0 4px 20px rgba(0,0,0,0.4); }
#leaderboardPanel h2 { margin-top:0; color:#ffd700; font-size:20px; text-align:center; }
table { width:100%; border-collapse:collapse; font-size:14px; }
th,td { padding:8px; text-align:center; border-bottom:1px solid #3a4a55; }
th { color:#ffd700; }
tr.highlight { background:rgba(241,196,15,0.15); }
.empty { color:#888; padding:20px; }
#clearScoresBtn { margin-top:12px; width:100%; background:#e74c3c; color:#fff; }
#clearScoresBtn:hover { background:#ff6b5e; }
```
Need if main panel width 820 and leaderboard 320, total 1160; max-width 1200 okay. On small screen wrap.

Need "canvas" CSS width 100% but attribute 800x600; scaling might make mouse coordinate mapping okay. Need if canvas CSS width 100% but height auto, rect width maybe not 800 but mouse mapping uses rect width and canvas.width. Good.
Need "button" for clear scores red. Good.

Need "leaderboard table" columns rank, time, date, mark. Good.

Need "test instructions" in final maybe after code.

Potential issue "clearScoresBtn" confirm uses `window.confirm`. Good.

Potential issue `renderLeaderboard` called at startup before `loadScores`. Need call after script:
```
renderLeaderboard();
```
Need "startGame" maybe initial state idle. On load, targets drawn. Timer 0. Good.
Need "startBtn" click calls startGame. If state playing, resets. Good.

Potential issue "canvas click" uses mouse mapping but if `mouse.inside` false, still can click? click event on canvas only if pointer over. Good.
Potential issue "statusEl.textContent" after startGame if no shot, okay.
Potential issue "targets" sorted by y; if hit overlay green ring; good.
Potential issue "targets" with `hit` false, if all hit and state finished, draw green. Good.

Need maybe "leaderboard rank" if time equal and same id? id unique. Good.
Need maybe "date string" sorting if equal? We use id. Good.
Need maybe "elapsedTime" has many decimals; store full. Good.

Need perhaps "finishGame" called twice? Could happen if two shot resolutions same frame and state still playing? After finishGame sets state finished, subsequent `resolveShot` won't be called because updateShots checks state. But what if both shots resolve in same frame and first doesn't hit all (e.g., 3/4), second hits last, calls finishGame; then loop over remaining shots continues but state finished so skip. Good.

Need maybe "statusEl" after finishGame and then renderLeaderboard; no conflict. Good.

Need maybe "hit all targets" if target already hit and arrow hits old target, hitAny true, no "脱靶"; okay.
Need maybe "finished" after all targets hit, if shot in flight resolves no effect. Good.

Need maybe "ranking" should update after finish and render. Good.

Need maybe "startGame" resets `lastPlayId=null`, so previous highlight gone. Good.

Need maybe "clearScores" while game playing? It clears leaderboard but not game. Fine.

Need maybe "unlimited arrows" no ammo count. Fine.

Need maybe "targets" could be hit by click outside board but due to line-collision? We use endpoint. Good.

Need perhaps to make target hit more satisfying, play sound. Since user didn't require. We can include WebAudio with simple beep. But perhaps avoid. Let's consider adding optional sound? It could be nice but not required. Need maybe no external files. WebAudio can generate.

Implement sound:
```
let audioCtx = null;
function playTone(freq, duration, type='sine', gain=0.03) {
 try {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  osc.type=type; osc.frequency.value=freq;
  g.gain.setValueAtTime(gain, audioCtx.currentTime);
  g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
  osc.connect(g); g.connect(audioCtx.destination);
  osc.start(); osc.stop(audioCtx.currentTime + duration);
 } catch(e) {}
}
```
Then in shootAt: `playTone(180,0.1,'triangle',0.02);` In hit: `playTone(880,0.15,'square',0.02);`; in finish: melody? Could. But if audio not allowed, catch. Need user gesture starts audio? Clicking start button is user gesture; audioCtx should work after. But if no sound, no issue. Let's include minimal? It adds code but okay. Need not overcomplicate. Since final answer code maybe 400 lines; okay.
Need "shootAt" called from canvas click; audioCtx creation in click, allowed. startBtn click not. Good.
Need if audioCtx state suspended? no.
Let's add sound? Could make game more enjoyable. But not necessary. Keep code a bit shorter. User asked "production". I can include simple sound. Let's include `playTone` and call on shoot/hit/finish. But if "排行榜" no sound. Fine.

Need maybe "hit ring effect" with sound. Good.

Need "miss" sound maybe low tone:
- shoot: `playTone(220,0.08,'triangle',0.02);`
- hit: `playTone(880,0.12,'square',0.02);`
- miss: `playTone(140,0.15,'sawtooth',0.01);`
- finish: `playTone(523,0.12,'square',0.02); setTimeout...` but setTimeout in finish? Could play two tones with slight delay? We can call:
```
playTone(523,0.15,'square',0.03);
setTimeout(()=>playTone(659,0.15,'square',0.03),120);
setTimeout(()=>playTone(784,0.2,'square',0.03),240);
```
Need not. If using setTimeout with audioCtx, okay. But maybe no.

Need "audioCtx" variable at top. Good.

Need "drawArrow" maybe if shot resolved and landed in target, arrow remains. But if target hit and green overlay drawn after target, arrow drawn before? Draw order: targets then shots. So arrow appears on top of target. If target overlay includes dark fill, arrow drawn after overlay, okay. But if arrow at target edge, visible. Good.

Need "drawTarget" label after hit overlay? label always on top. Good.

Need "drawShots" after targets but before bow. If arrow in flight, it appears above targets; if arrow goes behind a near target? Since targets sorted and drawn, then all shots drawn on top, so arrows always in front of targets. That's okay for arcade. But if arrow goes to far target behind near target, line could be in front of near target visually; maybe okay.
Need "archer" drawn after shots, so a shot starting at archer appears behind bow? Actually shots drawn before bow, then bow drawn over shot near start. Fine.

Need "drawCrosshair" after bow; crosshair on top.

Need "game loop" even when state idle? Need draw background and targets. So start RAF at load:
```
requestAnimationFrame(loop);
```
Need `lastFrameTime` not needed. `loop(now)`:
```
function loop(now) {
 if (state === 'playing') {
   elapsedTime = (now - startTime) / 1000;
   timerEl.textContent = `用时: ${elapsedTime.toFixed(2)}s`;
 }
 updateShots(now);
 updateEffects(now);
 draw(now);
 requestAnimationFrame(loop);
}
```
Need if state idle, draw. Good.

Potential issue "startTime" is `performance.now()`; RAF `now` is also DOMHighResTimeStamp from page load, same time origin? `performance.now()` same. Good.

Need "mouse" default x=400,y=300; crosshair not drawn until inside. But drawBow angle default to 300, okay.

Need "window" resize? no.

Need "clear scores" button maybe if confirm dialog blocks in some contexts? okay.

Need "Leaderboard" maybe if localStorage unavailable, still in-memory? Our loadScores returns [] if error; saveScores no-op. Good.

Need "test instructions" final:
```
运行方式：
1. 将代码保存为 archery.html。
2. 用 Chrome/Edge/Firefox 双击打开即可；无需安装/联网。
3. 点击「开始游戏」，鼠标瞄准，点击画布射击。命中全部4个靶后结束，成绩自动存入 localStorage。
```
测试说明:
```
- 功能测试：开始后计时增加；每命中一个靶子，该靶子出现绿色✓；全部命中后计时停止并弹排名。
- 排行榜测试：再次开始并完成，成绩按用时升序排列，最短时间在第一名；刷新后记录仍存在。
- 重置测试：点击「清空排行榜」可清除记录。
- 边界操作：未开始时点击画布不会射击；结束后点击画布不会射击；点击「重新开始」可随时开新一局。
```
Need maybe "不能多文件" no.

Let's think if final should include code only or code + instructions. User asks "请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" So final should include "运行方式" and code. Could include code block then instructions. But "交付可运行产物" likely code block. Need not include extra files. I can say "下面是单个 HTML 文件，保存为 ... 即可运行。" Then code.

Need maybe "不要查看其他文件" because user in some ChatGPT context with files? We don't inspect, just create. Fine.

Need ensure code no syntax errors. Let's write full code carefully.

Let's draft full code with syntax:

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
...
<script>
(() => {
'use strict';
...
})();
</script>
</body>
</html>
```
Use IIFE to avoid globals. Need ensure event listeners inside.

Let's write full code now in final. But before final, let's simulate mentally for syntax.

IIFE:
```
(() => {
  'use strict';
  const canvas = ...
  ...
})();
```
Need `audioCtx` variable:
```
let audioCtx = null;
function playTone(freq, duration, type = 'sine', volume = 0.02) { ... }
```
Need `audioCtx.currentTime` if audioCtx created but suspended? no.

Targets:
```
function createTargets() {
  return [
    { id: 0, name: '左远', x: 170, y: 240, r: 36, hit: false },
    { id: 1, name: '右远', x: 630, y: 240, r: 36, hit: false },
    { id: 2, name: '左近', x: 120, y: 420, r: 52, hit: false },
    { id: 3, name: '右近', x: 680, y: 420, r: 52, hit: false }
  ];
}
```
Need labels use t.id+1, not name. Good.

Danger: Near target r=52, x=120, left edge=68, okay. Right edge=172. Far target x=170, r=36, left edge=134, right edge=206. Near target at x=120 overlaps horizontally with far? At y? No, near target at y=420, far at y=240. But draw order far first. On screen, near target board center x=120, top y=368, left/right 68/172; far target at x=170,y=240, bottom y=276, so no vertical overlap. Good. Labels far at y=198, near at y=362; no overlap.
Right near x=680, far x=630; similarly.

Need "drawTarget" legs for far target: bottom y=240+36=276, legs to y=240+72=312. Near target top=420-52=368. No overlap. Good.

Need "shootAt" duration:
```
const distance = Math.hypot(dx, dy);
const duration = Math.min(0.55, 0.15 + distance / 1800);
```
For far distance sqrt((230)^2+(300)^2)=378; duration=0.36. For near distance sqrt((280)^2+(125)^2)=306; duration=0.32. Good.
Need "arrow speed" okay.

Need "updateShots" if multiple shots and one finishes after game finish due to earlier shot; state finished so no resolve. But arrow draw visible. Good.

Need "resolveShot" check `state === 'playing'` at start? Not necessary but add:
```
if (state !== 'playing') return;
```
Could be called when state maybe finished? updateShots checks. Fine.

Need "finishGame" uses `now` from RAF; if resolveShot called with now, good.
Need if state finished and user clicks "再来一局", startGame resets. Good.

Need "renderLeaderboard" uses `lastPlayId` and `scores.slice(0,10)`. Good.

Need "HTML" table columns:
```
<tr><th>排名</th><th>用时</th><th>完成时间</th><th>备注</th></tr>
```
Need "备注" column small. Good.

Need "clearScoresBtn" text and start button in top bar.

Need "instructions" line.

Let's write final code with Chinese comments perhaps. Good.

Potential issue "drawBow" arrow angle when mouse y below bow; arrow points down, bow arc below; okay.
Potential issue "canvas.addEventListener('click', ...)" uses `state` from closure. Good.
Potential issue "mouse.inside" if mouse leaves, drawBow angle remains last mouse; okay.

Need "html code block" huge but okay.

Let's ensure "leaderboard rank" sorted by time ascending. In finishGame:
```
scores.sort((a,b) => a.time - b.time || a.id.localeCompare(b.id));
```
Good.
Need "if scores.length > 100" no cap? We can keep all. Good.
Need "saveScores" maybe local storage can store many attempts. okay.

Need "state" after finish and `startBtn.textContent='再来一局'`. Good.
Need "clearScoresBtn" perhaps if no scores, confirm still; okay.
Need "renderLeaderboard" after clear, lastPlayId remains maybe but no rows. Fine. But if then startGame sets lastPlayId=null. Good.
Need "startGame" calls renderLeaderboard? It does; but lastPlayId null, so no highlight. Good.

Need "if state idle and user clicks start, startTime set. If user clicks start again after 2s, state playing, resets. Good.

Potential issue "timerEl" during idle shows "用时: 0.00s"; after finish "用时: xxx"; after start reset. Good.

Need "playTone" with setTimeout in finish:
```
function finishGame(now) {
 ...
 playTone(523,0.15,'square',0.03);
 setTimeout(() => playTone(659,0.15,'square',0.03), 120);
 setTimeout(() => playTone(784,0.2,'square',0.03), 240);
}
```
Need if audioCtx not created at finish, setTimeout creates? The function called from canvas click, user gesture? Actually finish happens after shot click, so user gesture context? `audioCtx` created on shoot click maybe. If not, playTone creates audioCtx inside setTimeout after user gesture? Autoplay policy may block if no gesture. But likely okay because page had click. To be safe, in playTone if audioCtx state 'suspended', call resume:
```
if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
```
Not necessary.

Need "playTone" in shootAt called on click; audioCtx created and resumes due to user gesture. Good.

Let's craft final code now.

Need maybe no hidden `''` issues. Let's write code with backticks? HTML code block. Good.

Let's also include "startGame" maybe when already playing, confirm? no.

Now, final answer should include code block and run/test instructions. The user said "不要查看其他文件，直接创建" so final could say "已完成，代码如下". Need not mention "I didn't view files". But can.

Need maybe "输出要求：请直接完成任务并交付可运行产物" So final should start with "可运行产物（单个 HTML 文件）：" then code. Then "运行方式" and "测试说明". Good.

Let's ensure code includes no unescaped `</script>` string? In code, no. Need if using `font='bold ...'` okay.

Let's think through draw target ring colors. In archery, from outer to inner: white, black, blue, red, gold. We draw white outer ring with radius r*1.0, black r*0.8, blue r*0.6, red r*0.4, gold r*0.2. But black ring over white with radius 0.8 leaves white outer 0.2. Good. Need no black border? Board background brown under rings. Good.
Need if hit overlay dark fill over rings, then green checkmark. Good.

Need maybe draw "target names" with "靶1" etc. Good.

Need maybe "four targets" in game and hit counter. Good.

Potential issue "drawTarget" label after hit overlay: If target hit, dark overlay + green check, then label above. Good.
Need "ring effect" drawn after target and shots? drawEffects after shots, so ring over arrow. Good.

Need "drawShots" if arrow in flight but p maybe computed with now from RAF; if shot.resolved true, p=1. Good.

Need "updateEffects" filters before draw. It uses `effects = effects.filter(...)`; since `effects` let in IIFE. Good.

Need "mousemove" mapping:
```
function setMouseFromEvent(e) {
 const rect = canvas.getBoundingClientRect();
 mouse.x = (e.clientX - rect.left) * canvas.width / rect.width;
 mouse.y = (e.clientY - rect.top) * canvas.height / rect.height;
}
```
Need if canvas CSS width maybe 100%, rect.width not canvas.width; mapping correct. Good.
Need "canvas.addEventListener('click', e => { setMouseFromEvent(e); ... })" But mousemove already; still set. Good.
Need if canvas click event `e.clientX` etc. okay.

Need "mouseenter" maybe:
```
canvas.addEventListener('mouseenter', (e) => { mouse.inside = true; setMouseFromEvent(e); });
canvas.addEventListener('mouseleave', () => { mouse.inside = false; });
```
Need "mousemove" sets inside true too. Good.

Potential issue "mouse.inside" false initially, drawBow angle default to point (400,300) which is slightly up-right; okay. Crosshair not drawn until inside. Good.

Need "drawBow" always draws a nocked arrow, even when state idle. Fine.

Need "startBtn" perhaps if clicked during playing, should reset timer. Good.

Need "clearScoresBtn" with confirm maybe in test automation might block. But manual. Fine.

Need "leaderboard panel header" maybe include "用时越短名次越高". Add `<h2>🏆 排行榜 <small>用时短→名次高</small></h2>` maybe. But small in h2. Let's include:
```
<h2>🏆 排行榜</h2>
<p class="rankTip">按用时从短到长排列</p>
```
Need CSS for rankTip. Fine.

Let's craft final code fully.

Wait, requirement "玩家命中所有目标所用的时间越短，在排行榜上的名次就越高" If our leaderboard includes all attempts, not players. Could mention "每次完成都会保存一条成绩；排行榜按用时升序排名". Good.
Need if multiple attempts, each attempt is a row. Fine.

Could maybe implement "若当前时间优于排行榜第一，显示新纪录" But not required.
Need maybe "best time" sort. Good.

Let's consider if "target hit" should be "all targets" but if clicking one target and missing, timer continues. Good.

Need maybe "hit all targets" after all four target centers; target hit if arrow endpoint within target radius. Good.

Potential issue "finishGame" rank computed using `scores.findIndex` after `renderLeaderboard`? No, before. Good.
Potential issue "lastPlayId" used in render after `scores` sorted. Good.
Potential issue "date" localization includes commas; okay.

Let's final.

Need perhaps "remove const BOW_X? Use `const BOW_X = 400; const BOW_Y = 545;`" in code. Good.

Let's write code with comments moderate.

Let's scrutinize arrow draw coordinate. In `drawBow`, after rotate(angle), arrow line from 18 to 74. But bow arc radius 32 from -1.2 to 1.2 yields endpoints at x = 32*cos(1.2)=11.6. The bow limbs are around x=11.6; arrow line starts at 18, so arrow doesn't intersect bow limbs? It starts behind the string? Actually the string is from endpoints (11.6,-29.8) to (11.6,29.8), a vertical line at x=11.6. Arrow line from 18 to 74 is to right of string, okay. But in real bow, arrow rests on bow, so okay.
Need fletching at 6 to -? Wait arrow line starts at 18, fletching at 6? In drawBow code planned:
```
ctx.beginPath(); ctx.moveTo(18,0); ctx.lineTo(74,0); stroke();
ctx.fillStyle = '#c0c0c0';
ctx.beginPath(); ctx.moveTo(84,0); ctx.lineTo(68,-6); ctx.lineTo(68,6); fill();
ctx.fillStyle = '#e74c3c';
ctx.beginPath(); ctx.moveTo(18,0); ctx.lineTo(6,-6); ctx.lineTo(6,6); fill();
```
This draws arrow shaft from 18 to74, head triangle from 68 base to 84 tip, fletching at 18? Wait fletching triangle from (18,0) to (6,-6),(6,6) puts feathers behind the start of shaft, not on shaft? The fletching should be at back of arrow, maybe near 18. Good. But the arrow tip is at 84. The shaft line from 18 to74 doesn't reach tip; head triangle covers from 68 to84. Good.
But the bow arc at x~11.6, arrow starts at x=18, so there is a gap between string and arrow. Fine.
Need "drawShots" uses arrow line from -36 to0 and head from -4 to10, tip at 10? Wait line from -36 to0; arrowhead triangle from (10,0) to (-4,-5),(-4,5), so head covers from -4 to10. Tip at 10. Shaft doesn't extend to head? It ends at 0, head starts at -4, okay. Good.
Fletching at -36 to -46. Good.

Need "drawShots" arrowhead too large? okay.
Need "drawShots" for p=1, arrow tip at end point. If target hit, tip in target; if miss, in grass. Good.

Need "hit detection" uses `shot.endX/Y` (mouse click point). But arrow tip drawn at end point; yes. If click at target center, tip at center. Good.

Need "canvas click" perhaps if user clicks on target board but crosshair position is outside? no.

Need "updateShots" doesn't use `now` for shots created with `performance.now()`; RAF now same origin. Good.

Need "drawEffects" uses `ctx.globalAlpha`, resets to 1. Good.
Need "updateEffects" filters by `now - effect.time < 1100`; ring effect age 600, text 1000. Good.

Need maybe "effects" array can contain `type:'text'` from finish? no.

Need "sound" if playTone called on every shot and hit; okay. Need `osc.stop` maybe if duration 0.15. Good.

Let's write final.

One more thought: The user said "不要查看其他文件" maybe they expect no reading from project files, but our final with code block is fine.

Need maybe "提供运行方式和必要的测试说明" after code. We'll include.

Let's craft final answer elegantly.

Wait, need maybe "交付可运行产物" Could be "archery.html" not just code block? We can say "将下面的完整代码保存为 `archery.html` 并用浏览器打开即可。" That's a deliverable.

Let's ensure code in final not too long? It's okay.

Let's proceed.
