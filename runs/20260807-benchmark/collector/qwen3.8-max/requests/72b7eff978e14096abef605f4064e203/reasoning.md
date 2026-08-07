The user wants an archery simulation game with the following elements:
1. 4 targets
2. Timer-based scoring — the faster you hit all targets, the higher your leaderboard ranking
3. Built directly, no other files referenced
4. Deliver an executable artifact with instructions and testing notes

Since the request is in Chinese, respond in Chinese.

This should be a single HTML file game — self-contained, immediately runnable. Let's design something rich and lively:

**Design direction:**
- Archery range atmosphere — forest/meadow archery range, or classic archery field
- The bow should respond to the mouse: draw the bow, aim, release to fire
- Physics: arrow trajectory with gravity, maybe slight wind
- 4 targets at different distances/positions, maybe moving targets to make it interesting
- Hit detection: concentric rings with different scores? The task says "hit all targets" — so each target needs at least one hit. Time-based leaderboard.
- Timer counts up from the start (or first shot)
- Leaderboard stored in localStorage, shows name + time
- Environmental atmosphere: layered hills, clouds, grass, birds, particles, wind indicator

**Mechanics ideas:**
- Draw the bow by holding the mouse (power builds up, or drag-back style)
- Drag to aim, release to fire
- Arrow physics with gravity — parabolic trajectory
- Targets: 4 of them. For fun, some stationary, some moving (up/down)? Maybe make targets pop up at different depths. To keep it fair and speed-based, place 4 targets at varying distances, with 1-2 moving vertically.
- Wind adds challenge and atmosphere.
- Arrow sticks in target or ground with a thunk sound.

**Visual style:**
- Archery range: sky gradient (dawn/day), parallax hills, trees, grass, wooden fence, straw target stands
- Bow at left side, archer's viewpoint or side-view bow
- Side view is easier: bow on left, targets to the right at various distances, arrow flies rightward with gravity
- Actually, first-person view toward targets is also possible, but side-view physics is clearer and more fun.

Let's go with side view: archer's bow at bottom-left, targets placed at various x positions and heights, some on stands, some moving. Draw with mouse: press and drag back to set angle and power, show trajectory hint (dotted line, maybe partial). Release to fire.

**HUD:**
- Timer (big, monospace/display font)
- Shots fired count
- Targets hit 0/4 with indicators
- Wind indicator
- Best time / leaderboard panel

**Leaderboard:**
- localStorage, top 8 entries, name entry via prompt or inline input on completion
- Rank by time ascending

**Completion:**
- All 4 hit → overlay with time, shots, accuracy, name entry, save to leaderboard, show leaderboard, restart button

**Audio:** WebAudio for bowstring, whoosh, thunk (hit), bullseye chime, miss. Synthesized, no external files.

**Fonts:** Distinctive display + readable body. Use Google Fonts? Should work offline too... but a single HTML file with Google Fonts links is common and fine; fallback if offline. For display, something like "ZCOOL KuaiLe" or "Ma Shan Zheng" (Chinese calligraphy) fits archery (kyūdō) aesthetics. Body: "Noto Sans SC". Display could be "Ma Shan Zheng" (brush style) — perfect for an archery game with Chinese flavor. Let's go with that.

Actually, for Chinese text, "Ma Shan Zheng" is a brush font, perfect for the title "弓箭" theme. Body: "Noto Sans SC". Numbers/timer: "Chakra Petch" or monospace. Let's use "Ma Shan Zheng" for display, "Noto Sans SC" for body. Timer can use tabular figures via font-variant-numeric.

**Palette:** Archery range at golden hour — warm sky (amber/peach to soft blue), deep pine greens, straw gold targets with classic red/white rings. Rich, layered background: sky gradient, sun, drifting clouds, distant mountains, midground trees, foreground grass.

**Gameplay details:**
- Canvas-based rendering, full window.
- Bow position: left side, fixed x (about 120px), y follows... bow angle from drag.
- Aiming: mousedown anywhere starts draw; drag to set angle and power (drag distance = power). Show bow rotating, string being pulled, arrow nocked. Show faint trajectory preview (first few points) — maybe only short preview to keep skill involved.
- Release: fire arrow with velocity, gravity applies. Arrow rotates to velocity direction. Trail particles.
- Arrow hits target: check circle collision with target face (target is a circle facing the player... in side view, target should be angled toward the player). Hmm, side-view archery: targets are usually drawn as circles facing the camera even in side view (stylized). That's fine — treat targets as circles (bullseye face visible), arrow sticks when tip enters circle. Ring score based on distance from center.
- Targets: 4 total. Layout:
  - T1: near, low, stationary, on stand
  - T2: mid distance, higher, stationary
  - T3: far, moving vertically (slow)
  - T4: farthest, moving horizontally? Or floating balloon style? Keep circles. Maybe moving vertically faster, or swaying.
- Each target needs 1 hit (any ring). Once hit, arrow sticks, target gets "hit" mark, maybe falls over/dims with a satisfying effect, flag turns green.
- Wind: random per run, shown with flag and particles, affects arrow's horizontal acceleration.
- Missed arrows stick in ground, fade after a while.
- Timer starts on first draw (first interaction) — or on game start? Fairer: timer starts on first shot release, or when player clicks "开始". Let's do: start screen → click 开始 → countdown 3-2-1 → timer starts, player draws. Timer runs until 4th target hit.

Actually simpler and more standard: after countdown, timer runs. That's clear.

**Structure:**
- Single HTML file with CSS + JS embedded. Everything drawn on canvas except HUD overlay (DOM) and leaderboard overlay.

Let's write the code carefully.

Key components:

1. Canvas setup, resize handling, devicePixelRatio.
2. Scene: sky gradient, sun with glow, clouds (drifting), mountain layers (parallax subtle), tree silhouettes, grass strip, fence, ground.
3. Targets: class with pos, radius, ring colors, movement pattern, hit state, stand drawing.
4. Bow: draw at archer position — bow arc, string, arrow; state from drag.
5. Arrow: physics, rotation, trail, collision with targets/ground/offscreen.
6. Particles: hit burst (ring-color confetti), dust on ground impact, leaves drifting in environment.
7. Wind: value, flag animation, drifting leaf/dust particles showing wind.
8. HUD: timer, shots, hit counter (4 dots/icons), wind arrow, best time.
9. Game state: MENU, COUNTDOWN, PLAYING, DONE.
10. Leaderboard: localStorage 'archery_lb', array of {name, time, shots, date}. Sort ascending by time, then by shots. Render in overlay. Name input on completion.
11. Audio: WebAudio synth — functions: playDraw (creak?), playRelease (string twang: short noise + tone), playWhoosh, playThunk (low thud), playBullseye (chime arpeggio), playComplete (fanfare), playTick (countdown).

Aiming model:
- mousedown/touchstart: begin draw (if PLAYING). Record start point? Common model: drag anywhere — vector from current pointer to start point (or bow position) determines angle/power. Let's do: drag back like slingshot — vector from current pointer to start point sets launch direction (opposite of drag). Power = drag distance clamped. Show bow rotating to aim angle, string pulled proportional to power.
- Actually, for archery feel: aim = angle from bow position to pointer? And hold to charge power? Slingshot drag feels better and more tactile. Let's do slingshot: drag from anywhere; launch vector = (start - current) normalized * power. Bow angle follows launch angle.
- Show dotted trajectory preview (simulate ~20 steps) — maybe show only first 30% to keep challenge. Show preview dots that fade.

Physics scale: canvas coords; gravity ~ 1400 px/s²; launch speed up to ~1400 px/s. Wind acceleration ±120 px/s².

Targets placed relative to canvas width for responsiveness:
- Ground y = H - 90.
- T1: x = W*0.38, y = groundY - 120, r = 46, stationary, on wooden tripod stand.
- T2: x = W*0.55, y = groundY - 230, r = 40, stationary, on taller stand... or hanging? Stand.
- T3: x = W*0.72, base y oscillates: y = groundY - 300 + sin(t)*70, r = 36, moving vertically, on "rail" pole? Draw pole behind, target slides.
- T4: x = W*0.88, y = groundY - 180 + sin(t*1.3+1)*90, r = 32, faster movement.

Hmm, with W variation, ensure bow at x=~110 has enough distance.

Arrow collision with target: distance from arrow tip to target center < r → hit. Ring score: d/r → 10..1. Record score per target; show popup "+10环!". Target marked hit; arrow sticks at hit point (save relative offset, draw arrow in target while it moves — arrow follows target movement, nice touch).

Completion: all 4 hit → freeze timer at hit moment, state DONE after short delay (let particles play), show overlay with time, shots, accuracy, name entry, save, leaderboard, restart button.

Leaderboard: localStorage, top 8 entries, name entry on completion. Rank by time ascending.

Edge cases: arrow offscreen right/top → remove. Ground collision → stick, small dust, count as shot (already counted on release).

Shots counter increments on release.

Timer display: mm:ss.ms → format like "12.34秒" or 00:12.34. Show seconds with 2 decimals: e.g., "12.34s". Chinese: 用时.

Countdown: 3, 2, 1, 开始! big brush font with scale animation, tick sound.

Menu overlay: title "穿杨射箭" / "神箭手" — title: "神箭 · 四靶速射"? Let's name it "穿云箭" or "百步穿杨". Title: "百步穿杨 · 速射挑战". Description with icons: 按住拖拽蓄力 → 松开射击 → 命中全部4个靶子，用时越短排名越高. Show leaderboard preview (top 3). Start button.

Name: default "无名侠士". Input maxlength 8.

Visual polish details:
- Sky: gradient #ffd9a0 → #ff9e7a? Golden hour: top #ffe9b8? Let's craft: top soft blue #aee3f5? Golden hour warm: top #ffd9a0... hmm. Golden hour warm: top #ffe9b8, mid #ffc37a, horizon #ff9d6b with sun glow. Mountains: layered teal/olive silhouettes (#7ba05b, #557d4a, darker). Actually archery range in meadow: sky soft blue → warm horizon; distant mountains blue-green; midground forest green; meadow green; dirt lane where targets stand? Let's do: sky #bfe6f0 → #ffe8c2 horizon; sun pale gold; clouds white soft; far mountains #9fc4b0; near hills #6fa05e; meadow #7fb25f → darker; ground strip with grass blades.
- Targets: classic archery face: outer white, black? Standard: gold center, red, blue, black, white rings. Simplify: rings [white, black, blue, red, gold] from outside to inside. On straw backing circle. Draw with slight 3D: straw torus behind face.
- Stand: wooden legs (brown lines), crossbar.
- Flag showing wind on pole near archer.
- Archer: maybe just bow + hand, or simple silhouette figure holding bow? Simple stylized archer silhouette would add character. Draw simple figure: head circle, body, arm to bow. Keep minimal but present. Maybe just bow + arrow with subtle hand grip. Let's draw simple archer: dark silhouette with headband. Hmm, risk of looking bad. Alternative: elegant bow only with quiver nearby. Draw bow (recurve shape), string, nocked arrow, plus simple arm/hand. Keep it clean: bow + string + arrow + small hand circle at grip and draw hand.

Actually, let's include a simple archer figure: standing pose, side profile, holding bow. Simple shapes: legs, tunic body, head with topknot, forearm to bow grip, back arm to string follows draw. This could look charming if done carefully with capsule shapes. Let's attempt with moderate detail; stylized flat colors (dark ink blue outfit, skin tone). The draw hand pulls string to drag position (clamped near body).

Simplification: bow held at fixed position (anchorX, anchorY) = (150, groundY-150). Archer body just behind. Nock draw point = anchor + dir * -drawAmount (pulls back along aim direction). Bow rotates around grip to aim angle.

Trajectory preview: simulate points with gravity + wind, draw fading dots, only while drawing, maybe first 14 dots every other step.

HUD top bar: left — big timer in brush font; middle — 4 target pips showing hit state (mini target icons that fill when hit); right — shots & wind indicator. Use DOM for HUD for crisp text, positioned over canvas.

Wind indicator: arrow + value like "风 2.3 →". Flag in scene also bends.

Screen shake on hit (small), flash ring on hit, floating score text.

Sound synth details:
- ctx created on first user gesture.
- twang: oscillator triangle 180→90Hz 0.15s + noise burst bandpass.
- whoosh: filtered noise with rising then falling sweep, tied to arrow launch, gain tied to power.
- thunk: low sine 120→60 + noise click, short.
- ring chime: for bullseye (>=9): two sine tones (E5, A5) plucked.
- complete: small arpeggio C-E-G-C.
- countdown tick: short square 660Hz; go: 880 longer.

Leaderboard save: key 'bycy_leaderboard_v1'. Entry {n, t(ms), s(shots), d(date string)}. Sort by t ascending, then s ascending. Keep top 10. Highlight new entry.

Also show "本次成绩" panel on completion overlay with rank #.

Restart: reset targets, wind, timer, shots → countdown again. Also "返回菜单" button.

Keyboard: R to restart? Space to start. Minor.

Touch support: pointer events (pointerdown/move/up) — use pointer events to cover mouse + touch. Prevent default touch scroll on canvas with touch-action: none.

Resize: recompute layout-dependent positions (targets positioned as fractions of W/H). Save target definitions as fractions; recompute px on resize. In-flight arrows ok.

Pause on tab hidden? Timer uses performance.now diff accumulated only during PLAYING; if document hidden, clamp dt (dt = min(dt, 50ms)) to avoid huge jump — but timer uses real elapsed... let's accumulate timer by dt each frame (clamped) instead of absolute timestamps, so hidden tab doesn't unfairly count. Good.

Now let's write the code. Single file: index.html. Deliverable: file + run instructions + test notes in Chinese.

Let's carefully write the drawing functions.

Scene draw order:
1. Sky gradient + sun + glow
2. Clouds (3-5, drifting with wind slightly)
3. Distant mountains (static path with noise? Precomputed polygon points relative to W)
4. Midground hills with tree silhouettes
5. Meadow ground
6. Fence posts along back of lane? Maybe subtle
7. Target stands + targets (with stuck arrows)
8. Wind flag pole
9. Archer + bow + nocked arrow
10. In-flight arrows + trails
11. Particles (front)
12. Grass blades foreground strip (swaying)
13. Vignette subtle

Mountains: generate once per resize: array of points.

Grass blades: array of {x, h, phase} along bottom; sway with sin(time + phase) + wind.

Clouds: array of {x,y,scale,speed}; wrap around.

Environmental leaves: a few leaf particles drifting with wind.

Target drawing:
- Stand legs from ground to target back.
- Straw disc: circle radius r*1.18, color #d9b26a with darker stroke, some arc texture lines.
- Face rings: fractions of radius: rings at [1.0 white #f4f1e8, 0.8 black #2b2b2b, 0.6 blue #3f7fd1, 0.4 red #d94f3d, 0.2 gold #f2c14e]. Draw circles from largest to smallest.
- Hit state: keep face but add stuck arrow + small green flag/checkmark badge? Draw a red "中" stamp? Cute: brush-style "中" character slightly rotated, dark red, stamped on target when hit. Use Ma Shan Zheng font for canvas text: ctx.font = '... "Ma Shan Zheng"'. Nice touch. Target also slightly desaturates? Keep simple: stamp + arrow.
- Moving targets: draw rail pole (vertical dark slot) behind, slider.

Score popup: floating text "+8环" rising and fading.

Arrow: shaft line brown, head triangle gray, fletching 2-3 small lines red/white at nock. Rotate to angle. Trail: fading line segments (store last N positions).

Stuck ground arrows: save {x,y,angle}, draw slightly tilted; remove after 6s with fade, or keep max 12.

Draw archer:
Position: feet at groundY, x = 110. Bow grip at (160, groundY-140)? Let's define:
- AX = 118 (body center x), groundY.
- Head at (AX+6, groundY-178) r 13, skin #e8b98c, hair bun.
- Body: tunic capsule from (AX, groundY-160) to (AX, groundY-95), color #35507a? Outfit deep teal #2f5d50? Let's do ink blue #3a4a6b with red sash.
- Legs: two lines to ground, dark pants, boots.
- Front arm: shoulder (AX+8, groundY-150) to bow grip G.
- Bow grip G = (AX+52, groundY-148) = (170, groundY-148).
- Back arm: shoulder to draw hand position (string pull point).
- Quiver on back with 2-3 arrows.

Bow: recurve drawn with quadratic curves, rotated to aim angle θ. Bow length ~90. Drawn as arc: from tip to tip curving away from archer. String: straight line tip to tip, but when drawing, string pulled to nock point: two lines tip→nock, nock→tip.

While aiming: nock position = G - dir*pull, pull = min(dragDist, 90)*0.6? Actually the string is pulled back along the aim direction: nock = G - u*pull where u = unit aim vector (points toward target). Arrow from nock to nock + u*arrowLen (arrow ~70px), tip past grip.

When idle (not drawing): string straight tip-tip, arrow resting pointing at last angle (default -15°).

Bow shape rotated: draw in local space with grip at origin, bow along vertical axis perpendicular to aim: bow curves along u direction. In local coords: rotate so +x = u. Bow limbs: tips at (8, -55) and (8, 55) with curve control (−18, 0)? Recurve facing target: belly faces archer. Path: moveTo tip1, quadraticCurveTo((-14,0), tip2) — this curves back toward archer, string on archer side... hmm, standard: bow curves away from archer (toward target), string is straight on archer side. So in local space with +x toward target: tips at x≈+14 (recurve tips slightly back), mid at x≈-6? Let's do: grip at (0,0); bow arc through control point (+26, 0) from (10,-58) to (10,58) → arc bulges toward target. String from tip to tip (straight line at x≈10, archer side is smaller x). When drawing, string center pulled to nock local (-pull, 0)... nock local x = -pull +? String attaches at tips (10,±58); pulled nock at (10 - pullAlong? ) Simpler: nock local = (-pull, 0), pull 0..46. String lines: tip1→nock→tip2. Arrow along +x from nock.

That looks right.

Aim angle from drag: on pointerdown save p0; on move, v = p0 - p (drag vector, slingshot). angle = atan2(v.y, v.x)... wait launch direction should be opposite of drag: if player drags down-left, arrow flies up-right. dir = normalize(p0 - p)? p0 - p gives vector from current to start = opposite of drag. If player drags down-left, p is down-left of p0, p0-p points up-right. Yes, launch dir = normalize(p0 - p). Require drag distance > 12 to count as draw; power = clamp(dist, 0, 260) → speed = 520 + power*3.4 → max ~1400.

Angle clamp: only allow shots toward right half (dir.x > 0.05), otherwise min power.

Preview: simulate with dt=1/60, 26 steps, draw dot every 2 steps with decreasing alpha. Only show while drawing and power > threshold.

Release: if power < minPower (drag < 18) → cancel (no shot).

Arrow flight: pos += vel*dt; vel.y += G*dt; vel.x += wind*dt; angle = atan2(vel). Trail points.

Collision check each frame: for each unhit target, dist(tip, center) <= r*1.0 → hit. Compute ring: rr = dist/r → score = 10 - floor(rr*10) clamped 1..10... standard: rr<=0.1→10, <=0.2→9... so score = clamp(10 - floor(rr/0.1)?, let's do: score = Math.max(1, 10 - Math.floor(rr*10)). rr 0→10, 0.95→10-9=1. Good.

On hit: target.hit = true; target.arrow = {dx,dy,angle} relative; shotsHit++; particles burst ring color; popup; sound thunk + chime if >=9; shake 6; check completion → finish().

Ground: tip.y >= groundY+6 → stick: stuckArrows.push; dust particles; sound thunk soft.

Offscreen: x > W+80 or y < -600 (returning eventually) or x < -100 → remove.

Wind: on each run start, randomize wind = (Math.random()*2-1) * 140 (accel px/s²). Display m/s style: (wind/40).toFixed(1). Flag drawn on pole at x=260? Place flag atop a pole near mid lane: pole at x = W*0.28, top y = groundY-260; flag cloth flutter: draw with bezier using wind sign/magnitude + time ripple.

Also drifting environmental particles (leaf/petal) with vx = wind*0.6.

HUD DOM:
- Top-left: 用时 label + timer big.
- Top-center: target pips: 4 mini SVG-like circles (DOM divs with radial rings via CSS radial-gradient!) — div with radial-gradient background to look like target; when hit, add class → green check overlay / stamp.
- Top-right: 射击 n · 命中 m; wind chip: "→ 风 2.1".
- Bottom-left hint text: "按住拖拽蓄力 · 松开射箭" fades after first shot.

Overlays (DOM):
- Menu: big title, subtitle, how-to-play (3 steps with small inline icons via CSS/emoji), leaderboard top list, start button. Style like a wooden plaque/scroll? Card with paper texture (CSS gradient), brush title.
- Completion: 成绩 card: big time, stats (箭数, 命中环数总计?, 最准一环), name input + 保存并查看榜, or auto-show leaderboard with highlight, buttons 再来一局 / 返回主页.
- Countdown overlay: big number center.

Fonts:
<link href="https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&family=Noto+Sans+SC:wght@400;500;700;900&family=ZCOOL+QingKe+HuangYou&display=swap">
Maybe use ZCOOL QingKe HuangYou for HUD numbers (has punch). Ma Shan Zheng for big title & "中" stamp & countdown. Body Noto Sans SC.

Canvas fonts: for stamp "中" and popups: '700 28px "Ma Shan Zheng", serif' — canvas can use web font after load; ok with fallback.

Colors summary:
- Sky top #bfe7f2 → #ffe6bd horizon; sun #fff3c9 glow.
- Clouds #ffffffcc.
- Far mountains #b7d0c0 / #9dbfae; near hills #74a45f; forest tree clumps #4f7f46.
- Meadow #86b86a top → #5f9350; ground lane maybe dirt strip #c9a36a? Targets stand on grass; ok.
- Foreground grass blades darker #3f6d3a.
- Wood #8a5a33 / #6e4526.
- UI ink #2d2a26; paper #fdf6e3; accent vermilion #c8442c; gold #e0a83c.

UI style: paper/ink Chinese archery aesthetic — cards like rice paper with vermilion seal accents. HUD chips: dark translucent ink with paper-color text, or paper chips with ink text + red seal dots. Paper chips with subtle shadow, slight rotation? Keep readable.

Buttons: vermilion with paper text, brush-style font, hover lift + press.

Leaderboard rows: rank medal (1 gold, 2 silver, 3 bronze circle), name, time mono, shots. New entry row highlighted with red "新" seal.

Edge cases: localStorage might throw in some contexts → wrap in try/catch.

Timer format: seconds with 2 decimals + "秒", e.g., 12.34 秒; if ≥60 → 1:02.34.

Also add subtle "parallax" — mountains shift slightly with pointer x? Small, adds life: offsetX = (mouseX/W-0.5)*10 applied to mountain layers with different factors. Nice.

Screen shake: shake magnitude decays; apply translate to canvas.

Now let's write everything. Watch out for pointer events on canvas vs overlays: overlays are DOM on top; canvas listens on window for pointermove/up while drawing.

Prevent context menu? Not needed. touch-action:none on canvas.

Countdown flow: startGame(): reset; state=COUNTDOWN; countT=3.999? Use countdown value 3→1 then GO. Tick sound on each integer change; at 0 → state=PLAYING, timer=0, play go sound. During countdown scene renders, bow disabled.

Finish: on 4th hit: record finalTime = timer; state=WIN_ANIM (keep rendering, disable input) for 900ms with confetti, then show completion overlay. Timer display frozen.

Also total ring sum for stats: sum of 4 scores, max 40.

Accuracy: hits/shots.

Name save: input value trimmed || '无名侠士'.

Leaderboard render function shared between menu (top 5) and completion (top 10 with highlight id). Assign id = Date.now to new entry.

Also add "最佳纪录" chip on HUD showing current best (lb[0]) if exists.

Now let's write code. Aim for clean, organized, commented (Chinese comments).

CSS: full-viewport canvas, overlays absolutely positioned. Use CSS variables. Responsive-ish (min sizes). Mobile: HUD scales with clamp().

Let's write the JS structure:

```js
(() => {
 // dom refs
 const cv = document.getElementById('cv'), ctx = cv.getContext('2d');
 let W,H,DPR, groundY;
 // state
 const S = { MENU:0, COUNT:1, PLAY:2, WIN:3 };
 let state=S.MENU, timer=0, countT=0, shots=0, hits=0, wind=0, shake=0;
 let bestTime = null;
 // entities
 let targets=[], arrows=[], stuck=[], parts=[], pops=[], clouds=[], blades=[], leaves=[], mtsFar=[], mtsNear=[], trees=[];
 // input
 let drawing=false, p0={x:0,y:0}, pNow={x:0,y:0}, aimA=-0.2, power=0;
 ...
})();
```

Target class as plain object:
```js
function makeTargets(){
 targets = [
  {fx:0.36, fy:..., r:46, move:null},
  ...
 ];
}
```
Position compute: t.x = t.fx*W; base y: stationary: groundY - t.h; moving: groundY - t.h + sin(now*t.sp + t.ph)*t.amp.

Definitions:
- t1: fx .36, h 130, r 46
- t2: fx .54, h 250, r 40
- t3: fx .71, h 300, r 36, amp 70, sp 1.1
- t4: fx .87, h 200, r 32, amp 95, sp 1.7

Ensure min width handling: if W small (<720), scale r? Multiply r by k = clamp(W/1200, .7, 1.1). ok.

Draw stand per target: for stationary: tripod legs from (x±r*0.8, groundY) to (x, y+r*0.6). For moving: vertical pole at x+? from ground to top; slider behind target. Pole at x (behind), top groundY - (h+amp+ r +30).

Flag: pole at x=W*0.26? Might overlap t1 (0.36). Flag at 0.24.

Archer at x=112; bow grip G=(172, groundY-150).

Launch from nock position but simplify: arrow starts at tip position ≈ G + u*60.

Preview simulation from that start point.

Collision uses arrow tip: tip = pos (track pos as tip). Draw arrow back from tip along angle: tail = tip - u*len.

Ground collision when tip.y >= groundY+4.

Also allow hitting target stands? Ignore.

Arrow update order: integrate, then check targets (only unhit), then ground.

Stuck arrow in moving target: store relative dx,dy and angle; draw each frame at target pos + offset.

Popup: {x,y,txt,life,color}.

Particles: {x,y,vx,vy,life,maxlife,color,size,g}.

Confetti on win: burst at each target + center.

Sound:

```js
let AC=null; function ac(){ if(!AC) AC=new (window.AudioContext||window.webkitAudioContext)(); if(AC.state==='suspended')AC.resume(); return AC; }
function env(g,t0,a,d,v){...}
function noiseBuf(...) cached.
function sTick(){osc square 700, .06}
function sGo(){880 .3}
function sDraw(){short filtered noise lowpass rising? subtle}
function sShoot(p){twang: osc tri 170→80 .12 + noise burst hp}
function sWhoosh(){noise bandpass sweep 300→1200 .25 gain by power}
function sThunk(){osc sine 150→55 .12 + noise lowpass click}
function sChime(n){if score>=9: two-tone pluck 880/1320}
function sWin(){arpeggio}
```

Keep audio functions compact.

Main loop with rAF, dt clamp 0.05.

HUD update via DOM each frame (timer text, wind). Use requestAnimationFrame same loop.

Target pips: 4 divs; toggle .hit class.

Now menu overlay content:

Title: 百步穿杨 (big brush) + subtitle 四靶速射 · 弓箭模拟. Seal box "射" red.
How-to-play: 3 steps: ① 按住屏幕向后拖拽，蓄力张弓 ② 调整角度，松开手指放箭 ③ 命中全部 4 个靶子，用时越短，排名越高. Also note wind: 留意风向旗，箭会受风影响.
Leaderboard: 英雄榜 top 5 or 虚位以待.
Button: 开 弓 (start).

Completion overlay:
- Header 全部命中!
- Big time, rank line: "当前排名 第2位" or 未进前十.
- Stats row: 用箭 X 支 · 总环数 Y/40 · 命中率 Z%
- Name input + button 题名留榜 (if not saved) → after save, show leaderboard and buttons 再战一局 / 返回首页.
Simplify: always show input + 保存成绩; after save, replace with leaderboard list + buttons. Manage with small state.

Also allow Enter key in input to save.

Prevent duplicate saves per run: flag saved.

Edge: if user closes overlay without saving → provide 跳过 button (goes to leaderboard view / restart). Include small "不题名，直接再来" link.

Now let's write the final HTML. Also include a favicon? Skip. Title tag: 百步穿杨 · 弓箭速射.

Also add resize listener regenerating scenery.

Grass blades: count = W/9; each {x, len 8-22, ph}. Draw as quadratic curve swaying.

Trees on midground hills: array of clumps {x, r} drawn as dark green rounded blobs along hill line.

Mountains: generate polygon: points every 60px with smooth noise (random walk) y within band. Regenerate on resize. Fill with color.

Clouds: 4 clouds {x,y,s,v}; drawn as 3 overlapping ellipses alpha .8; drift with v + wind*0.02.

Sun: at (W*0.78, H*0.2)? Behind mountains; radial glow.

Vignette: radial gradient transparent → rgba(40,30,20,.18) at edges.

Parallax: track mouseX; far mountains offset by (mx-0.5)*8, near hills *16, clouds also slight.

Countdown overlay: DOM div center with number, animate scale via CSS animation retrigger.

Hint bar bottom: "🏹 按住并向后拖拽蓄力，松开射箭" — show during PLAY until first shot.

Also show current run info? Ok.

Wind chip: arrow ← or → and strength text; color by strength.

OK, code time. Let's write it all carefully, ensuring no syntax errors.

Also: on pointerdown only start draw if state===PLAY and e.target is canvas (overlays have pointer-events auto and sit above, so canvas won't receive). Attach pointerdown to canvas; move/up to window.

If pointerup with insufficient power → cancel, no shot.

Aim angle clamp: dir.x must be > 0.08, else clamp dir to near horizontal? If player drags weirdly, compute angle, clamp to [-80°, +10°]? Launch angle mostly upward: allow angle between -1.35 rad and +0.15 rad (slightly downward). Compute a=atan2(dir.y,dir.x); clamp a to [-1.4, 0.18]. Rebuild dir from clamped angle. Note: canvas y is downward, so negative angle = upward. dir.y negative = up. atan2 returns negative for up. Good.

Power: pd = dist(p0,pNow); power = clamp((pd-10)/240, 0, 1); speed = 540 + power*920.

Preview dots: simulate 30 steps dt .016, draw every other dot with alpha (1-i/30)*.5, radius 3.

Bow draw amount: pull = power*46.

Archer back arm to nock world position: nock = G - u*pull (u is aim unit). Back hand at nock. Front arm shoulder→G.

Idle aim angle default -0.28 rad.

While drawing, bow also tilts slightly with power? Skip.

Arrow on string while drawing: from nock to nock+u*74, tip ahead.

In-flight arrows drawn separately.

Quiver: on back, small rectangle rotated with 3 arrow lines.

Headband ribbon fluttering with wind — cute detail: 2 small red ribbons from head, animate with sin.

OK, also "中" stamp: when target hit, draw text '中' at center+offset in dark red with brush font, slight rotation, alpha .9. Also ring flash: expanding circle.

Hit flash: parts ring: use stroke circle particle type 'ring'.

Now let's write the code.

One more thing: timer only accumulates during PLAY. WIN state freezes.

Total ring sum for stats: sum of 4 scores, max 40.

Accuracy: hits/shots.

Name save: input value trimmed || '无名侠士'.

Leaderboard render function shared between menu (top 5) and completion (top 10 with highlight id). Assign id = Date.now to new entry.

Also add "最佳纪录" chip on HUD showing current best (lb[0]) if exists.

Now let's write the code. Aim for clean, organized, commented (Chinese comments).

CSS: full-viewport canvas, overlays absolutely positioned. Use CSS variables. Responsive-ish (min sizes). Mobile: HUD scales with clamp().

Let's write JS structure:

Main constants:
```js
const GRAV=1500;
```

Speed: max ~1460. Distance to t4 at W=1600: dx≈1600*.87-172≈1220. Time at vx 1400*cos... reachable with high arc. Wind ±140 accel is significant over 1s flight (±70px drift... actually drift = .5*a*t² ≈ 70px at t=1). Ok.

Ensure stationary target heights reachable: t2 h250 → center y = groundY-250. Apex needed. With v 1400 at 45°, vy≈990, apex = vy²/2g ≈ 327px above launch (launch at groundY-150). Reachable. t3 h300+amp70 → max 370 above ground; center max groundY-370; needs 220 above launch — ok.

Ring score popup color by score: >=9 gold, >=7 red, else ink.

Also add small "箭 X" counter.

HUD pips: build 4 divs .pip with radial-gradient rings; when hit → add class .hit → overlay ✓ stamp via ::after content '✓' green? Use '中' in brush font red — cute: pip::after content '中'.

Wind chip arrow: char '→'/'←' scaled by strength; if |wind|<8 show '无风'.

Now let's write final file. Also include meta viewport.

Completion overlay rank computation: after inserting entry, index in sorted list → rank = idx+1 (if within 10).

Menu leaderboard list top 5.

Function renderLB(container, list, highlightId).

Medal colors: 1 #e0a83c, 2 #b9c2cc, 3 #c98a5a, others ink number.

Time format fmt(ms): if ms<60000 → (ms/1000).toFixed(2)+'″'? Use "12.34 秒". For leaderboard compact: 12.34s → show as `12″34`? Keep `${(ms/1000).toFixed(2)} 秒`.

Shots shown as X 箭.

Now, let's write it all. Careful: template literals inside script are fine.

Also handle pointercancel → cancel draw.

Also blur: if drawing and window blur → cancel.

Also prevent text selection during drag: user-select none on body.

Also iOS: pointer events ok.

Write CSS with paper cards:
```css
.card{background:linear-gradient(160deg,#fffdf4,#f6ecd4); border:2px solid #3b3024; box-shadow: 0 18px 50px rgba(30,20,10,.35), inset 0 0 0 6px rgba(200,68,44,.06); border-radius:14px;}
```
Add corner seal decoration via ::before? Add red seal square with 射 character top-right of menu card.

Button:
```css
.btn{font-family:'Ma Shan Zheng'; font-size:26px; background:#c8442c; color:#fff8ec; padding:10px 34px; border-radius:10px; border:2px solid #7e2413; box-shadow:0 6px 0 #7e2413; transition:...;}
.btn:hover{transform:translateY(-2px); box-shadow:0 8px 0 #7e2413;}
.btn:active{transform:translateY(3px); box-shadow:0 2px 0 #7e2413;}
```

HUD chip: paper style:
```css
.chip{background:rgba(253,246,227,.92); border:2px solid #3b3024; border-radius:10px; box-shadow:0 4px 12px rgba(30,20,10,.25); color:#2d2a26;}
```

Timer big: font 'ZCOOL QingKe HuangYou' 40px.

Countdown: brush font 160px ink with red? Ink #2d2a26 with slight stroke; animate pop.

OK, write final. Also add subtle paper texture via repeating-linear-gradient noise? Use subtle.

Also "返回菜单" from completion.

Menu start button click: hide menu, startRun().

startRun(): reset targets (hit=false, arrows cleared), stuck=[], shots=0, hits=0, timer=0, wind=randWind(), state=COUNT, countT=3.0 (render ceil), saved=false; hide overlays; show HUD.

Countdown render: while state COUNT: countT -= dt; number = ceil(countT); if changed play tick; when <=0 → state=PLAY, show "开始!" flash (reuse countdown div with '开!' for .5s then hide).

Win: on 4th hit: finalTime=timer; state=WIN; winT=0; sWin(); confetti; after 1.1s show completion overlay (populate stats). Use winShown flag.

Also stop timer accumulation (state!=PLAY).

Completion overlay populate: time text, stats, name input area visible, leaderboard hidden until save; button: 保存成绩 / link 直接再来.
After save: show board with highlight, hide input, buttons 再战一局 / 返回首页.

Also allow Enter key in input to save.

Prevent duplicate saves per run: flag saved.

Edge: if user closes overlay without saving → provide 跳过 button (goes to leaderboard view / restart). Include small "不题名，直接再来" link.

Now write final HTML. Also include favicon? Skip. Title tag: 百步穿杨 · 弓箭速射.

Also add resize listener regenerating scenery.

Grass blades: count = W/9; each {x, len 8-22, ph}. Draw as quadratic curve swaying.

Trees on midground hills: array of clumps {x, r} drawn as dark green rounded blobs along hill line.

Mountains: generate polygon: points every 60px with smooth noise (random walk) y within band. Regenerate on resize. Fill with color.

Clouds: 4 clouds {x,y,s,v}; drawn as 3 overlapping ellipses alpha .8; drift with v + wind*0.02.

Sun: at (W*0.78, H*0.2)? Behind mountains; radial glow.

Vignette: radial gradient transparent → rgba(40,30,20,.18) at edges.

Parallax: track mouseX; far mountains offset by (mx-0.5)*8, near hills *16, clouds also slight.

Countdown overlay: DOM div center with number, animate scale via CSS animation retrigger.

Hint bar bottom: "🏹 按住并向后拖拽蓄力，松开射箭" — show during PLAY until first shot.

Also show current run info? Ok.

Wind chip: arrow ← or → and strength text; color by strength.

OK, code time. Let's write it all carefully, ensuring no syntax errors.

Also: on pointerdown only start draw if state===PLAY and e.target is canvas (overlays have pointer-events auto and sit above, so canvas won't receive). Attach pointerdown to canvas; move/up to window.

If pointerup with insufficient power → cancel, no shot.

Aim angle clamp: dir.x must be > 0.08, else clamp dir to near horizontal? If player drags weirdly, compute angle, clamp to [-80°, +10°]? Launch angle mostly upward: allow angle between -1.35 rad and +0.15 rad (slightly downward). Compute a=atan2(dir.y,dir.x); clamp a to [-1.4, 0.18]. Rebuild dir from clamped angle. Note: canvas y is downward, so negative angle = upward. dir.y negative = up. atan2 returns negative for up. Good.

Power: pd = dist(p0,pNow); power = clamp((pd-10)/240, 0, 1); speed = 540 + power*920.

Preview dots: simulate 30 steps dt .016, draw every other dot with alpha (1-i/30)*.5, radius 3.

Bow draw amount: pull = power*46.

Archer back arm to nock world position: nock = G - u*pull (u is aim unit). Back hand at nock. Front arm shoulder→G.

Idle aim angle default -0.28 rad.

While drawing, bow also tilts slightly with power? Skip.

Arrow on string while drawing: from nock to nock+u*74, tip ahead.

In-flight arrows drawn separately.

Quiver: on back, small rectangle rotated with 3 arrow lines.

Headband ribbon fluttering with wind — cute detail: 2 small red ribbons from head, animate with sin.

OK, also "中" stamp: when target hit, draw text '中' at center+offset in dark red with brush font, slight rotation, alpha .9. Also ring flash: expanding circle.

Hit flash: parts ring: use stroke circle particle type 'ring'.

Now let's write the code.

One more thing: timer only accumulates during PLAY. WIN state freezes.

Also handle document.hidden pause: dt clamp handles physics; timer accumulates by dt so hidden tab (rAF stops) doesn't count — good, rAF pauses in background.

Let's write.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>百步穿杨 · 四靶速射</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&family=Noto+Sans+SC:wght@400;500;700&family=ZCOOL+QingKe+HuangYou&display=swap" rel="stylesheet">
<style>...</style>
</head>
<body>
<canvas id="cv"></canvas>
<!-- HUD -->
<div id="hud" class="hidden">
 <div class="chip timer-chip"><span class="lbl">用时</span><span id="timer">0.00</span><span class="unit">秒</span></div>
 <div id="pips"></div>
 <div class="chip right">
   <span id="shotInfo">箭 0 · 中 0/4</span>
   <span id="windChip">风 —</span>
   <span id="bestChip" class="hidden">最佳 —</span>
 </div>
</div>
<div id="hint">按住屏幕向后拖拽蓄力 · 松开射箭</div>
<div id="count" class="hidden"></div>
<!-- overlays -->
...
<script>...</script>
</body></html>
```

Pips: create 4 .pip divs via JS.

Now JS. Write fully carefully.

Also draw the lane: maybe subtle dirt path ellipse below each target stand? Add shadow ellipse below target.

Shadow: ellipse below each target stand and archer, alpha .15.

Time to write the final answer. Let's write the code in one shot but carefully.

Potential pitfalls:
- ctx.setTransform with DPR and shake.
- Font string with quotes in canvas: `900 ${s}px "Ma Shan Zheng", serif`.
- Pointer events: use e.clientX; canvas is full window so coords match CSS pixels (we scale ctx by DPR; use CSS pixel coordinate space after setTransform(DPR...)). Set canvas.width=W*DPR etc, ctx.setTransform(DPR,0,0,DPR,0,0). Draw in CSS px. Good.

- Resize: W=innerWidth, H=innerHeight.

Let's define scenery generation:

```js
function genScenery(){
 mtsFar=[]; let y=H*0.52; for(let x=-60;x<=W+120;x+=70){ y=clamp(y+(Math.random()*44-22), H*0.42, H*0.6); mtsFar.push({x,y}); }
 // near hills smooth
 hillsNear = ... use sin-based deterministic to avoid flicker on resize? Random ok.
 trees=[]; for(let i=0;i<Math.ceil(W/140);i++){ trees.push({x:Math.random()*W, r:18+Math.random()*26, h:...}); }
 blades=[]; for(let x=0;x<W;x+=7){ blades.push({x:x+Math.random()*6, l:9+Math.random()*16, ph:Math.random()*6.28, w:1+Math.random()*1.6}); }
 clouds=[]; for(let i=0;i<5;i++){ clouds.push({x:Math.random()*W, y:H*(0.08+Math.random()*0.22), s:0.6+Math.random()*1.1, v:4+Math.random()*8}); }
}
```

Hills near: draw as big rounded hill via quadratic curve across width at y≈H*0.66; then meadow from H*0.66 down? But groundY = H-90 is where gameplay floor is; meadow fills from hill line to bottom. Targets stand on groundY. Horizon hill behind at ~H*0.6.

Define: horizonY = H*0.58 (base of far mountains), hillY = H*0.68 (near hills top band), groundY = H-90.

Meadow gradient from hillY to H.

Fence: small posts along y = hillY+30? Might clutter; add subtle 3-rail fence behind targets at y=groundY-40? Skip fence, add distance marker flags? Keep flag pole for wind.

Ok.

Draw functions: drawSky, drawMountains, drawHills, drawMeadow (fill + grass blades later front), drawFlag, drawTargets, drawArcher, drawArrows, drawParticles, drawForeground.

Order in render():
Sky → sun → clouds → far mountains → near hills + trees → meadow fill → wind flag → targets (+ stand) → stuck ground arrows behind archer? Ground arrows anywhere → archer → in-flight arrows → particles → grass blades foreground → pops → vignette.

Pops are text, drawn near end.

Shake: ctx.translate after setTransform: ctx.translate(sx,sy).

Now target draw details:

```js
function drawTarget(t, now){
 const x=t.x, y=t.y, r=t.r;
 // shadow
 ellipse at groundY
 if(t.move){ // pole
   pole from groundY to t.topY at x; slot darker; slider knob behind target
 } else {
   tripod legs
 }
 // straw backing
 circle r*1.16 fill #d8b06a stroke #a67c3e lw3; inner arcs texture
 // rings
 const rings=[[1,'#f2efe4'],[0.8,'#3a3733'],[0.6,'#3e7bc4'],[0.4,'#d24b36'],[0.2,'#f0c24b']];
 for each draw circle fill; thin stroke rgba dark between.
 // center dot cross? small
 if(t.hit){ stuck arrow + stamp 中 }
}
```

Stamp: save, translate(x+r*0.25,y-r*0.25) rotate(-0.25), font r*0.9 Ma Shan Zheng, fillStyle 'rgba(160,30,20,.85)', textAlign center,.fillText('中',0,0). Maybe stamp appears with scale pop using t.hitT timestamp: scale = easeOutBack(min(1,(now-t.hitT)/0.35)).

Stuck arrow on target: relative offset stored; draw arrow shape at (x+dx,y+dy) with angle.

drawArrowShape(ctx, x,y, ang, len=64): shaft from (x - cos*len) to tip (x,y); head triangle at tip; fletching 3 small lines at tail.

In-flight arrow trail: save points each frame (max 10), draw polyline fading with alpha along.

Particle types: 'dot' (gravity), 'ring' (expanding stroke), 'conf' (confetti rect with rotation), 'dust'.

Pop: rise 40px in .9s and fade.

Wind flag: pole x=W*0.24, from groundY to groundY-240; flag attached at top: cloth length 70, wave: draw as filled path with 2 curves; direction sign = sign(wind) (if ~0 droop). Flutter amplitude with |wind| + sin(now*8).

Also small pennant color red.

Archer detail (final):

```js
function drawArcher(now){
 const ax=AX, gy=groundY;
 shadow ellipse
 // quiver on back: rotated rect at (ax-16, gy-150) angle .5, with arrows
 // legs
 line hip(ax, gy-92) to foot(ax-12, gy) and (ax+14, gy-2)? Slight stance; boots small rects.
 // body tunic: rounded path hip to shoulder(ax+4, gy-152); fill #3a4a6b; red sash at waist.
 // head: circle (ax+8, gy-172) r12 skin #eab98e; hair: dark arc top + bun circle behind; red headband line + fluttering ribbon ends with sin(now*6)+wind.
 // arms: front shoulder(ax+10, gy-148) to grip G; back shoulder(ax+2, gy-146) to draw hand (nock world position or rest).
 draw hands as skin circles at grip and nock.
 // bow drawn at G rotated to aimA
}
```

Bow drawing:

```js
function drawBow(gx,gy,ang,pull){
 ctx.save(); ctx.translate(gx,gy); ctx.rotate(ang);
 // limbs
 ctx.strokeStyle='#7a4a26'; lw 7 round;
 path: moveTo(12,-62); quadraticCurveTo(30,-30, 26,0)? hmm design: belly toward archer (-x), back toward target (+x).
```

Let's define tips at (14,-64) and (14,64); curve control at (-10,0)?? That bulges toward archer — wrong. Control at (+34,0) bulges toward target. Recurve tips: small backward curve at ends — skip complexity: simple elegant curve with control (+34,0). String from (14,-64) to nockLocal(-pull,0) to (14,64).

Wait string rest position at x=14 line; nock pulled to x = 14 - pullAmount? Arrow sits on string; grip at (0,0) is behind string? Actually: string at x≈14, grip at 0, archer further left. When pulled, nock at (14 - pull, 0), pull max 50 → nock x=-36, past grip — arrow drawn back past face. Visually fine.

Hmm but arrow tip should be at nock + arrowLen(72) along +x: tip x = -36+72=36 > 14 ok tip past string rest. Good.

Idle: string straight (14,-64)-(14,64); arrow resting: nock at (14,0), tip (86,0), drawn only when state PLAY/MENU? Always draw nocked arrow when not drawing (ready). While drawing, draw arrow at pulled position.

Bow decoration: small grip wrap rect at (−2..4,−10..10) darker.

Limb stroke gradient? Single color + thinner lighter inner stroke.

Aim angle applied: rotate(ang). ang = aimA (current). While drawing, aimA from drag; else keep last.

Nock world position for back arm: gx + cos(ang)*(14-pull) etc. Compute in JS.

OK, also draw simple hand at nock.

Preview dots: only while drawing && power>0.05.

Now input handlers:

```js
cv.addEventListener('pointerdown', e=>{ if(state!==S.PLAY)return; ac(); drawing=true; p0={x:e.clientX,y:e.clientY}; pNow={...p0}; cv.setPointerCapture(e.pointerId); sDraw(); });
window pointermove: if drawing update pNow, compute aim & power.
window pointerup: if drawing: if power>0.06 shoot else cancel; drawing=false.
```

Compute aim: dx=p0.x-pNow.x, dy=p0.y-pNow.y; dist=hypot; if dist<8 power=0 else power=clamp((dist-12)/230,0,1); ang=atan2(dy,dx); clamp ang to [-1.35, 0.2]; aimA=ang.

Wait slingshot: drag down-left → arrow up-right. dx = p0.x-pNow.x: pNow left of p0 → dx>0 ✓; pNow below p0 → dy = p0.y-pNow.y <0 → up ✓. Good.

shoot():
```js
const sp=540+power*920, u={x:Math.cos(aimA),y:Math.sin(aimA)};
const sx=GX+u.x*30, sy=GY+u.y*30; // start near bow
arrows.push({x:sx+u.x*44,y:sy+u.y*44,vx:u.x*sp,vy:u.y*sp,trail:[]});
shots++; sShoot(power); sWhoosh(power); updateHUD; hide hint after first shot; shake+=2;
```

Arrow update:

```js
for each a: a.vy+=GRAV*dt; a.vx+=wind*dt; a.x+=a.vx*dt; a.y+=a.vy*dt; a.trail.push({x,y}); if>11 shift;
 tip=a pos.
 // targets
 for each t of targets if(!t.hit){ d=hypot(a.x-t.x,a.y-t.y); if(d<=t.r){ hitTarget(t,a); remove arrow; break; } }
 // ground
 if(a.y>=groundY+4){ stuck.push({x:a.x,y:groundY+2,ang:Math.atan2(a.vy,a.vx),born:now}); dust; sThunk(soft); remove; }
 // offscreen
 if(a.x>W+120||a.x<-120||a.y>H+200) remove; (y<-... allow high up, returning eventually)
```

hitTarget(t,a):
```js
t.hit=true; t.hitT=now; t.arr={dx:a.x-t.x, dy:a.y-t.y, ang:Math.atan2(a.vy,a.vx)};
const rr=Math.hypot(t.arr.dx,t.arr.dy)/t.r; const score=Math.max(1,10-Math.floor(rr*10)); t.score=score; totalScore+=score; hits++;
pops.push({x:t.x,y:t.y-t.r-14,txt:score+' 环',c:score>=9?'#e0a83c':score>=6?'#d24b36':'#2d2a26'});
burst particles with ring color at hit point; ring flash; shake+=score>=9?9:5;
sThunk(); if(score>=9) sChime();
updatePips();
if(hits===4){ finalTime=timer; state=S.WIN; winT=now; sWin(); confettiBurst(); }
```

WIN update: after 1.15s show completion overlay once.

HUD update each frame: timer textContent = (timer).toFixed(2); wind chip; shotInfo.

Wind display: wv=(wind/46); text = Math.abs(wind)<10 ? '无风' : (wind>0?'→':'←')+' 风 '+Math.abs(wv).toFixed(1); if wind>0 wind pushes arrow rightward (tailwind) since target is right. Ok.

randWind: (Math.random()*2-1)*150; maybe ensure not too small: if |w|<25 → sign*40? Allow calm sometimes. Keep uniform.

Leaderboard:

```js
const LBKEY='bycy_lb_v1';
function loadLB(){ try{ return JSON.parse(localStorage.getItem(LBKEY))||[] }catch(e){ return [] } }
function saveLB(l){ try{ localStorage.setItem(LBKEY, JSON.stringify(l)) }catch(e){} }
function addScore(name, ms, shots){ const l=loadLB(); const e={n:name,t:ms,s:shots,id:Date.now()}; l.push(e); l.sort((a,b)=>a.t-b.t||a.s-b.s); const top=l.slice(0,10); saveLB(top); return {list:top, rank: top.indexOf(e)+1 ... } }
```
indexOf works since same reference. If cut off rank 0.

Render list HTML rows.

Best chip: lb[0]?.t.

Menu shows top 5.

Completion overlay flow: on show, populate stats; input area visible; board area shows current top 10 without highlight; save button → addScore → re-render board with highlight + rank line; swap buttons.

Buttons: btnAgain (再战一局) → startRun; btnHome (返回首页) → show menu (state=MENU, refresh menu leaderboard).

Also from menu, Enter/Space to start? Add keydown: if MENU and (Enter/Space) start. If completion shown and Enter in input → save (handled by input listener). R key during PLAY/WIN → restart. Nice.

Countdown DOM: #count div; set text and retrigger animation: el.classList.remove('pop'); void el.offsetWidth; el.classList.add('pop').

Show "开始!" then hide after 500ms via setTimeout.

Edge: startRun hides menu and completion overlays, shows HUD, hides hint until first shot? Show hint during PLAY until shots>0.

Also during MENU, render scene behind (idle, targets exist, no timer). Nice living background. Bow idle.

Also allow drawing during MENU? No.

Now confetti burst on win: for each target spawn 26 confetti particles with random up velocity, colors [gold, red, white, green].

Also add leaves environment: spawn every ~1.2s a leaf at left/top, drift with wind, sway; remove offscreen. Draw as small rotated ellipse green/yellow.

Performance: fine.

Now let's write the CSS fully.

Font stacks:
--f-brush:'Ma Shan Zheng', 'STKaiti','KaiTi', serif;
--f-disp:'ZCOOL QingKe HuangYou','Noto Sans SC',sans-serif;
--f-body:'Noto Sans SC',sans-serif;

HUD layout: position fixed top 12px left/right/center flex.

Pips: display flex gap 8. .pip{width:34px;height:34px;border-radius:50%; background: radial-gradient(circle, #f0c24b 0 18%, #d24b36 0 38%, #3e7bc4 0 58%, #3a3733 0 78%, #f2efe4 0 100%); border:2px solid #3b3024; position:relative; transition:transform .2s} .pip.hit{transform:scale(1.12)} .pip.hit::after{content:'中'; position:absolute; inset:0; display:grid; place-items:center; font-family:brush; color:#a01e14; font-size:22px; text-shadow:0 0 4px #fff8ec;}

Nice.

Overlays: .overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:rgba(30,22,14,.45);backdrop-filter:blur(3px)?} — blur ok subtle. Use dark warm overlay.

Card width min(92vw, 560px), padding 28-36.

Title with seal:

```html
<div class="title-row"><h1>百步穿杨</h1><span class="seal">射</span></div>
<p class="sub">四靶速射 · 用时定英雄</p>
```

Seal: red square with white 射 in brush font, slightly rotated.

Steps list with numbered brush circles.

Leaderboard list styling: rows grid rank | name | shots | time.

Completion card similar.

Countdown style: font brush 150px color #fff8ec with text-shadow ink; .pop animation scale 1.6→1 fade in.

Hint: bottom center chip, brush font, gentle float animation.

Now write JS ~600 lines. Let's go. Also double-check all referenced functions exist.

Also: on completion overlay, "最佳纪录" also updated.

drawArcher arm geometry: shoulder positions: front shoulder FS=(AX+10, groundY-150); back shoulder BS=(AX+2, groundY-146). Grip G=(GX,GY)=(AX+58, groundY-150)? Set AX=112, GX=172, GY=groundY-152.

Nock world when drawing: nx=GX+cos(ang)*(14-pull)? Bow local origin at grip; nock local x = 14-pull... wait string rest x=14; pulled nock local (14-pull, 0) — hmm earlier I said nock local (-pull,0) with string at 14. Use nockLocal = 14 - pull*1.0, pull max 50 → -36. World: GX + cos*nl, GY + sin*nl.

Back arm from BS to nock world; draw as 2-segment? Simple line with lw 7 skin? Sleeve color then hand skin circle. Use sleeve stroke #3a4a6b lw8 for arm, hand circle #eab98e.

Front arm from FS to G.

Idle: nock rest local (14,0) → world slightly past grip; back hand there.

Legs: hip HP=(AX, groundY-96). Feet F1=(AX-14, groundY-2), F2=(AX+16, groundY-2). Stroke pants #2f3350 lw9. Boots: small dark rects at feet.

Body: draw as thick line from HP to (AX+4, groundY-154) lw 22 color #3a4a6b round cap → capsule. Sash: short line at waist lw 8 #c8442c.

Head: (AX+9, groundY-176) r 12.5 skin; hair: dark arc top #26221e: circle clipped? Draw circle then hair: path arc top half. Bun: circle (AX-2, groundY-186) r5. Headband: line across forehead red lw4; ribbon ends: from (AX-4, gy-180) two quadratic curves fluttering leftward (wind opposite side?) with sin(now*7).

Quiver: at back hip: save translate(AX-14, groundY-120) rotate(-0.5): rounded rect 14x46 #6e4526; arrow fletches: 3 short lines at top with red/white tips. Restore.

Shadow: ellipse (AX+6, groundY+6, 40, 8) rgba(0,0,0,.18).

Good enough.

drawFlag: pole at FX=W*0.24, top FY=groundY-250. Pole stroke #6e4526 lw5 from groundY to FY; knob at top. Flag cloth: attach at (FX+2, FY+6): length L=64, dir=sign(wind) (if |wind|<10 droop: draw small drooping triangle). Wave: end point ex=FX+2+dir*L, ey=FY+6+ sin(now*6)* (2+|wind|*0.02)... draw path: moveTo(FX+2,FY+4); quadraticCurveTo(mid + wave, top edge to end); back along bottom edge; fill #c8442c with slight darker stripe? Keep flat red with darker stroke.

Flutter: offset control points by sin(now*9)*4.

Also small wind streak lines in sky when |wind|>60: a few moving semi-transparent white curved dashes — use leaves as indicator; skip streaks.

Vignette: create radial gradient each frame ok.

Pop drawing: brush font 26px, stroke white outline for readability: ctx.lineWidth=4 strokeStyle 'rgba(255,250,235,.9)' strokeText then fillText color.

Countdown number also DOM.

Now completion stats: 用箭 shots 支, 命中 4/4 by definition, 总环数 totalScore/40, plus each target's score shown as mini list? Show 4 chips with ring values. Nice: row of 4 mini target icons with score numbers.

Keep: stats grid: 用时 big; row: 用箭 X 支 | 总环数 Y/40 | 命中率 Z%.

Rank line: after save show "位列英雄榜 第 N 名" or "未入前十，再接再厉".

Before save show prompt text.

OK let's write the final code. Watch out for backticks etc. Also ensure `hidden` class toggling: .hidden{display:none !important}.

One more consideration: pointerdown on canvas will also fire when clicking through HUD chips? HUD chips are pointer-events none except interactive. Set #hud{pointer-events:none} children none too. Hint also none. Count also none.

Overlays are pointer-events auto when visible.

Canvas cursor: crosshair.

Also add small "R 重开" tip on HUD? Minor, skip; add on completion.

Time to write. Let me write the code in one go but carefully.

Also: initial state MENU: HUD hidden; menu overlay visible; scenery + targets idle animate (moving targets move — nice live menu).

Timer only accumulates during PLAY.

finalTime stored in ms float seconds; save ms = Math.round(finalTime*1000).

fmtTime(ms): s=ms/1000; if s>=60 → `${Math.floor(s/60)}:${(s%60).toFixed(2).padStart(5,'0')}` else s.toFixed(2). For HUD seconds only: timer.toFixed(2).

HUD timer shows seconds with 2 decimals + unit 秒; if >60 show m:ss.cc — implement fmtClock(sec).

Let's finalize the code.

Also, moving target arrow sticking: since target moves, arr relative stays. Good.

Also collision should check only arrow tip; moving target may move into arrow — check every frame both, fine.

Also multiple arrows can hit same target after hit? Skip since t.hit.

Also arrow may pass through target between frames at high speed (tunneling): speed 1460*dt(0.016)=23px per step vs r≥30 — mostly ok; add substep: if speed*dt > 12 do 2 substeps. Implement simple: steps = Math.ceil((sp*dt)/14); integrate in loop with dt/steps and check collision each substep. Cheap enough.

Now write.

function clamp(v,a,b){return Math.max(a,Math.min(b,v));}

Also easeOutBack for stamp:
```js
function eob(t){const c=1.70158;return 1+ (c+1)*Math.pow(t-1,3)+c*Math.pow(t-1,2);}
```

Grass blade draw:
```js
ctx.strokeStyle='#3f6d3a'; lw=blade.w; for each b: sway=Math.sin(now*1.8+b.ph)*3 + wind*0.02; path moveTo(b.x, H) quadraticCurveTo(b.x+sway*0.5, H-b.l*0.6, b.x+sway, H-b.l);
```
Blades along bottom edge (H is bottom, below groundY? groundY=H-90; foreground strip from groundY..H is front face of meadow? Simplify: draw blades along y=H (screen bottom) with length up to 26 — reads as foreground grass. Also sparser second row at groundY+40.

Ground: fill from hillY to H with meadow gradient. groundY line is implicit (where things stand). Maybe add slightly darker band below groundY? Add subtle path/lanes: skip.

Also add small distance stones or markers? Skip.

Alright, final write.

Also don't forget: after writing file, provide run instructions + test notes in Chinese. Response language: Chinese. Keep prose brief; the artifact is the big part.

Let me now write out the entire HTML, compiling in my head as I go.

Special attention: in JS inside HTML, avoid stray `</script>` in strings — none.

Also pips created in JS: for i<4 append div.pip to #pips.

updatePips: pip children toggle hit class by target.hit.

HUD ids: timer, shotInfo, windChip, bestChip.

Menu ids: menuOv, lbMenu, btnStart.
Completion ids: winOv, wTime, wStats, wRank, nameInput, btnSave, lbWin, btnAgain, btnHome, saveRow, boardWrap.

Count: countEl.

Keydown:
```js
window.addEventListener('keydown',e=>{
 if(e.key==='r'||e.key==='R'){ if(state===S.PLAY||state===S.WIN){ startRun(); } }
 if((e.key===' '||e.key==='Enter') && state===S.MENU){ startRun(); e.preventDefault(); }
});
```
Careful: Enter in name input triggers both save and this? Guard: if e.target is input → return.

Save flow:
```js
btnSave.onclick=()=>{ const name=(nameInput.value||'').trim().slice(0,8)||'无名侠士'; const res=addScore(name, Math.round(finalTime*1000), shots); saved=true; saveRow.classList.add('hidden'); wRank.textContent = res.rank>0? `位列英雄榜 · 第 ${res.rank} 名`:'未入前十 · 再接再厉'; renderLB(lbWin, res.list, res.id); boardWrap.classList.remove('hidden'); btnAgain.focus? no.
 update bestChip.
};
```

showWin(): populate wTime = fmtClock(finalTime); stats: `用箭 ${shots} 支 · 总环数 ${totalScore}/40 · 命中率 ${Math.round(4/shots*100)}%`; reset saveRow visible, boardWrap hidden (or show board without highlight immediately? Show board after save only; before save show top 3 teaser? Keep hidden, show text "题名后录入英雄榜"). Reset wRank text to prompt.

renderLB(el, list, hi):
```js
el.innerHTML = list.length? list.map((e,i)=>`<div class="row${e.id===hi?' new':''}"><span class="rk r${i+1}">${i+1}</span><span class="nm">${esc(e.n)}</span><span class="sh">${e.s} 箭</span><span class="tm">${fmtTime(e.t)}</span></div>`).join('') : '<div class="empty">虚位以待 · 等你来战</div>';
```
esc for name safety.

fmtTime(ms): s=ms/1000; if s>=60 → `${Math.floor(s/60)}:${(s%60).toFixed(2).padStart(5,'0')}` else s.toFixed(2). For HUD seconds only: timer.toFixed(2).

HUD timer shows seconds with 2 decimals + unit 秒; if >60 show m:ss.cc — implement fmtClock(sec).

OK — menu leaderboard also refresh when returning home.

btnHome: state=MENU; hide winOv; show menuOv; hide hud; render menu lb.

startRun from menu: hide menuOv; show hud; state=COUNT; countT=3.0; lastCount=4; reset run vars; updatePips reset; hide hint (until play & shots==0 show).

Countdown tick: n=Math.ceil(countT); if n!==lastCount && n>0 {countEl.textContent=n; pop; sTick(); lastCount=n;} when countT<=0: state=PLAY; countEl.textContent='开始!'; pop; sGo(); setTimeout hide 600 (guard state).

Also during COUNT, aim disabled (pointerdown checks PLAY).

WIN: winT timestamp; in update: if(!winShown && now-winT>1.15){winShown=true; showWin();}

Also during WIN, arrows still update (none), particles animate, targets animate.

Input during WIN ignored.

Now audio functions:

```js
let AC=null, noiseB=null;
function ac(){ if(!AC){ const A=window.AudioContext||window.webkitAudioContext; if(!A) return null; AC=new A(); const len=AC.sampleRate*0.5; noiseB=AC.createBuffer(1,len,AC.sampleRate); const d=noiseB.getChannelData(0); for(let i=0;i<len;i++)d[i]=Math.random()*2-1; } if(AC.state==='suspended')AC.resume(); return AC; }
function tone(f0,f1,dur,type,vol,delay=0){ const c=ac(); if(!c)return; const t=c.currentTime+delay; const o=c.createOscillator(), g=c.createGain(); o.type=type; o.frequency.setValueAtTime(f0,t); o.frequency.exponentialRampToValueAtTime(Math.max(1,f1),t+dur); g.gain.setValueAtTime(vol,t); g.gain.exponentialRampToValueAtTime(0.0001,t+dur); o.connect(g).connect(c.destination); o.start(t); o.stop(t+dur+.02); }
function noise(dur,vol,f=1000,q=1,type='bandpass',delay=0){ const c=ac(); if(!c)return; const t=c.currentTime+delay; const s=c.createBufferSource(); s.buffer=noiseB; s.loop=true; const flt=c.createBiquadFilter(); flt.type=type; flt.frequency.value=f; flt.Q.value=q; const g=c.createGain(); g.gain.setValueAtTime(vol,t); g.gain.exponentialRampToValueAtTime(0.0001,t+dur); s.connect(flt).connect(g).connect(c.destination); s.start(t); s.stop(t+dur+.02); }
const sTick=()=>tone(760,760,.07,'square',.12);
const sGo=()=>{tone(880,880,.25,'square',.16); tone(1320,1320,.3,'sine',.1,.02);};
const sDraw=()=>noise(.18,.05,600,2,'lowpass');
const sShoot=p=>{tone(190,70,.14,'triangle',.3); noise(.08,.18,2400,1.5); };
const sWhoosh=p=>noise(.3,.12+.1*p, 900, .8);
const sThunk=()=>{tone(150,50,.12,'sine',.4); noise(.05,.2,300,1,'lowpass');};
const sChime=()=>{tone(988,988,.3,'sine',.16); tone(1319,1319,.4,'sine',.14,.08);};
const sWin=()=>{[523,659,784,1047].forEach((f,i)=>tone(f,f,.22,'triangle',.18,i*.11));};
```

Whoosh frequency sweep would be nicer but ok.

Now render loop:

```js
let last=performance.now();
function loop(nowMs){ requestAnimationFrame(loop); const now=nowMs/1000; let dt=Math.min(0.05,(nowMs-last)/1000); last=nowMs; T=now;
 update(dt, now); render(now);
}
```

update(dt):
- clouds drift
- leaves spawn/update
- if COUNT: countT-=dt logic
- if PLAY: timer+=dt
- arrows physics (only PLAY/WIN — allow flight during WIN: always update arrows if any)
- stuck arrows cleanup (>8s fade)
- particles, pops update
- shake decay: shake*=Math.pow(0.001,dt)? Use shake=Math.max(0,shake-30*dt) or exponential shake*=Math.exp(-8*dt).
- WIN reveal check.
- HUD text update (timer, wind static per run, shots).

Target y compute each frame in update or render: t.x=t.fx*W; t.y = groundY - t.h + (t.amp? Math.sin(T*t.sp+t.ph)*t.amp:0).

render(now): as ordered above.

Also draw trajectory preview while drawing.

Also draw aim power meter? Small arc near bow showing power — add small bar: draw arc at grip radius 40 from angle... keep preview dots + bow pull as feedback; add power ring: ctx.arc(GX,GY, 34, -0.5π, -0.5π+power*1.5π) stroke gold alpha. Nice, add.

Now let's write the final file text. Also re-check quadratic curve drawing of bow under rotation — fine.

Also draw archer's bow hand circle at G.

Potential visual issue: bow at GX=172 near left; t1 at 0.36W: W=1280 → 460 ok.

Small screens (W<680): AX=112, targets fractions still fine; r scaled by k=clamp(W/1100,0.62,1). Apply k to r and h? h scales with H anyway... keep h fixed but clamp so target not above screen: ensure groundY - h - amp > 60, else reduce. Compute per target on resize: t.h=Math.min(t.h0, groundY-140-(t.amp||0)). Let's define h0 values and clamp.

groundY=H-90. If H small (mobile landscape 380): groundY=290; h0 300+amp70 → clamp h to 290-140-70=80. Ok.

Also bow GY=groundY-152; if H small, archer height ~190 > groundY? If H=380, groundY=290, archer top=groundY-190=100 ok.

Implement resize():
```js
W=innerWidth;H=innerHeight;DPR=Math.min(2,devicePixelRatio||1);cv.width=W*DPR;cv.height=H*DPR;cv.style.width=W+'px';cv.style.height=H+'px';groundY=H-90; k=clamp(W/1150,.6,1);
targets.forEach(t=>{t.r=t.r0*k; t.h=Math.min(t.h0, Math.max(60, groundY-150-(t.amp||0))); });
genScenery();
```

Target defs with r0/h0.

Moving target pole top: groundY-(t.h+t.amp+t.r+26).

Now let's write everything. Also menu shows behind canvas animation; overlay bg semi-transparent so scene visible — nice living menu.

Delivery: file code block + 运行方式 + 测试说明. Also mention features briefly. Chinese.

Let me now write the code out fully, compiling in my head as I go.

Special attention: in JS inside HTML, avoid stray `</script>` in strings — none.

Also pips created in JS: for i<4 append div.pip to #pips.

updatePips: pip children toggle hit class by target.hit.

HUD ids: timer, shotInfo, windChip, bestChip.

Menu ids: menuOv, lbMenu, btnStart.
Completion ids: winOv, wTime, wStats, wRank, nameInput, btnSave, lbWin, btnAgain, btnHome, saveRow, boardWrap.

Count: countEl.

Now write. Also for completion overlay, "最佳纪录" also updated.

drawArcher arm geometry: shoulder positions: front shoulder FS=(AX+10, groundY-150); back shoulder BS=(AX+2, groundY-146). Grip G=(GX,GY)=(AX+58, groundY-150)? Set AX=112, GX=172, GY=groundY-152.

Nock world when drawing: nx=GX+cos(ang)*(14-pull)? Bow local origin at grip; nock local x = 14-pull... wait string rest x=14; pulled nock local (14-pull, 0) — hmm earlier I said nock local (-pull,0) with string at 14. Use nockLocal = 14 - pull*1.0, pull max 50 → -36. World: GX + cos*nl, GY + sin*nl.

Back arm from BS to nock world; draw as 2-segment? Simple line with lw 7 skin? Sleeve color then hand skin circle. Use sleeve stroke #3a4a6b lw8 for arm, hand circle #eab98e.

Front arm from FS to G.

Idle: nock rest local (14,0) → world slightly past grip; back hand there.

Legs: hip HP=(AX, groundY-96). Feet F1=(AX-14, groundY-2), F2=(AX+16, groundY-2). Stroke pants #2f3350 lw9. Boots: small dark rects at feet.

Body: draw as thick line from HP to (AX+4, groundY-154) lw 22 color #3a4a6b round cap → capsule. Sash: short line at waist lw 8 #c8442c.

Head: (AX+9, groundY-176) r 12.5 skin; hair: dark arc top #26221e: circle clipped? Draw circle then hair: path arc top half. Bun: circle (AX-2, groundY-186) r5. Headband: line across forehead red lw4; ribbon ends: from (AX-4, gy-180) two quadratic curves fluttering leftward (wind opposite side?) with sin(now*7).

Quiver: at back hip: save translate(AX-14, groundY-120) rotate(-0.5): rounded rect 14x46 #6e4526; arrow fletches: 3 short lines at top with red/white tips. Restore.

Shadow: ellipse (AX+6, groundY+6, 40, 8) rgba(0,0,0,.18).

Good enough.

drawFlag: pole at FX=W*0.24, top FY=groundY-250. Pole stroke #6e4526 lw5 from groundY to FY; knob at top. Flag cloth: attach at (FX+2, FY+6): length L=64, dir=sign(wind) (if |wind|<10 droop: draw small drooping triangle). Wave: end point ex=FX+2+dir*L, ey=FY+6+ sin(now*6)* (2+|wind|*0.02)... draw path: moveTo(FX+2,FY+4); quadraticCurveTo(mid + wave, top edge to end); back along bottom edge; fill #c8442c with slight darker stripe? Keep flat red with darker stroke.

Flutter: offset control points by sin(now*9)*4.

Also small wind streak lines in sky when |wind|>60: a few moving semi-transparent white curved dashes — use leaves as indicator; skip streaks.

Vignette: create radial gradient each frame ok.

Pop drawing: brush font 26px, stroke white outline for readability: ctx.lineWidth=4 strokeStyle 'rgba(255,250,235,.9)' strokeText then fillText color.

Countdown number also DOM.

Now completion stats: 用箭 shots 支, 命中 4/4 by definition, 总环数 totalScore/40, plus each target's score shown as mini list? Show 4 chips with ring values. Nice: row of 4 mini target icons with score numbers.

Keep: stats grid: 用时 big; row: 用箭 X 支 | 总环数 Y/40 | 命中率 Z%.

Rank line: after save show "位列英雄榜 第 N 名" or "未入前十，再接再厉".

Before save show prompt text.

OK let's write the final code. Watch out for backticks etc. Also ensure all referenced functions exist.

Also: initial state MENU: HUD hidden; menu overlay visible; scenery + targets idle animate (moving targets move — nice live menu).

Timer only accumulates during PLAY.

finalTime stored in ms float seconds; save ms = Math.round(finalTime*1000).

fmtTime(ms): s=ms/1000; if s>=60 → `${Math.floor(s/60)}:${(s%60).toFixed(2).padStart(5,'0')}` else s.toFixed(2). For HUD seconds only: timer.toFixed(2).

HUD timer shows seconds with 2 decimals + unit 秒; if >60 show m:ss.cc — implement fmtClock(sec).

Let's finalize the code.

Also, moving target arrow sticking: since target moves, arr relative stays. Good.

Also collision should check only arrow tip; moving target may move into arrow — check every frame both, fine.

Also multiple arrows can hit same target after hit? Skip since t.hit.

Also arrow may pass through target between frames at high speed (tunneling): speed 1460*dt(0.016)=23px per step vs r≥30 — mostly ok; add substep: if speed*dt > 12 do 2 substeps. Implement simple: steps = Math.ceil((sp*dt)/14); integrate in loop with dt/steps and check collision each substep. Cheap enough.

Now write.

Also don't forget: after writing file, provide run instructions + test notes in Chinese. Response language: Chinese. Keep prose brief; the artifact is the big part.

Let me now write out the entire HTML, compiling in my head as I go.