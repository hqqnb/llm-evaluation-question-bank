The user wants an elevator simulator in a single HTML file. Requirements:

- User can generate a certain number of people on different floors
- Each elevator can carry only one person at a time
- 3 elevators total
- When a passenger boards an elevator, other passengers stay and wait; when the next elevator (any of the 3) arrives, they board in order (FIFO queue)
- Each person has a randomly assigned destination floor
- Hovering the mouse over a person displays a tooltip showing the destination floor
- Smooth animations, good visuals, clean design
- All in HTML/CSS/JS, single file
- Reply in Chinese (request is in Chinese)

Design thinking: make it feel like a cross-section of a real building. A building cross-section with floors, a shaft area with 3 elevator cars moving vertically, waiting passengers rendered as small figures with subtle variations (color, height). Controls: floor selection, number of people, spawn button. Or per-floor spawn buttons.

Visual direction: an architectural blueprint / control-panel aesthetic fits a building cross-section well. Let me think about palettes:

Options: deep teal/petrol background with warm amber interior lighting — like a nighttime building cross-section, windows glowing. That's characteristic: a building cross-section at dusk with glowing windows. Colors: deep blue-teal sky, warm amber window light, coral/gold accents. Elevator cars in distinctive colors.

Or a daytime look: soft sky gradient, concrete floors. Hmm, a nighttime building with glowing interiors feels more atmospheric. Layered background: gradient sky with stars, faint moon, city silhouette behind the building. Inside floors: warm lighting.

Typography: distinctive display face + readable body. Single file, so I can use Google Fonts via link (still one file, external fonts OK? "全部使用 HTML、CSS 和 JavaScript 构建。只给我一个文件" — one file; Google Fonts link is generally acceptable, but for offline safety I could use system fonts with fallbacks. Include the Google Fonts link with fallbacks so it degrades gracefully). Let me pick something with character: "Bungee" for display is very distinctive and signage-like — good for elevator/industrial signage. Body: "Outfit" or "Manrope". Numbers: "IBM Plex Mono" for floor indicators. Let me use Bungee for headings, Manrope for body, IBM Plex Mono for digits.

Layout plan:
- Left side (or top bar): control panel — title, floor selector, passenger count input, "呼叫乘客/生成" button, stats (waiting, riding, delivered), elevator status indicators.
- Main: building cross-section. Say 8 floors (1–8). Building shows floors as horizontal slabs, waiting people standing on each floor. To the right (or left) of the building: 3 elevator shafts with cars moving.

Actually, a typical elevator sim: shafts in the middle or side, people wait on each floor next to the shafts. Let me put 3 shafts on the left side of the building and floors extend to the right. People wait in a queue next to the shafts on each floor. When an elevator arrives at a floor and there are waiting people, the first person walks to the elevator, boards, and the elevator moves to the destination floor, person exits (walks out and fades/disappears, or walks to a "delivered" area and fades).

Elevator logic:
- 3 elevators, each state: idle, moving-to-pickup, boarding, moving-to-destination, dropping off.
- Each elevator carries only one person.
- Global request queue: when people spawn on a floor, that floor has a pickup request. Idle elevators get assigned to floors with waiting passengers (nearest-first). When an elevator arrives at a floor with waiting people, the front person boards (others stay). Then the elevator takes them to the destination. After drop-off, if there are still waiting floors, move to the next pickup (nearest), else idle.

Simpler and robust approach: maintain a set of floors with waiting passengers. Elevator dispatch: when an elevator becomes free, find the nearest floor with waiting people (by distance from current position), move there. On arrival, board one person (front of that floor's queue), then move to that person's destination. On arrival, exit animation, then check for more waiting floors; if any, move there; else idle. Multiple elevators may target the same floor; that's fine — each boards one person. To avoid all 3 targeting the same floor when only 1 person waits, we can track "incoming" elevators per floor and subtract capacity: floor demand = waiting count; assign considering other elevators already heading there. Keep it reasonably simple: when assigning, pick the floor that minimizes distance while maximizing unmet demand (waiting - assigned incoming). Good enough.

Doors: elevator cars have sliding doors; floor side has shaft doors too. Animate door open on arrival, person walks in, doors close, move.

Movement: use requestAnimationFrame loop, elevators move at constant speed with easing (ease-in-out between floors). Position in pixels computed from floor height.

Person rendering: small SVG or CSS figures. Make each person a div with CSS shapes: head circle, body (rounded rectangle), random color from palette, slight height variation. Idle sway animation. Tooltip on hover: shows "目标楼层 X" — implement with a custom tooltip div that follows, or a CSS ::after with data attribute. Custom tooltip via data attribute + CSS is simplest and smooth.

Boarding animation: person walks horizontally from queue position to elevator door, then fades (enters car). Actually show the person inside the car during the ride? Nice touch: display a small figure inside the elevator car while riding. Yes — show a mini passenger inside the car. On arrival at destination, the figure walks out onto the floor and fades out (or walks a bit and disappears), count delivered.

Queue positioning: people line up horizontally on the floor. Floor slab; queue starts near the shaft and extends to the right. Each person's position: left offset = queueStart + index * spacing. When the front boards, others shift left (animated with CSS transition on transform/left).

Spawning: user selects floor (dropdown or clicking a floor?) and number (input or slider), clicks "生成乘客". Also allow clicking directly on a floor to spawn 1? Keep controls: floor selector (buttons 1..8 or select), number input (1–10), spawn button. Also a "随机生成" fun button? Keep it clean. Maybe also a "清空" reset button.

Stats: waiting count, riding count, delivered count, plus per-elevator status lamps.

Visual details:
- Sky background: deep dusk gradient (deep teal → warm horizon), twinkling stars (CSS), soft moon, drifting clouds (slow CSS animation), city silhouette layer behind the building.
- Building: facade with floor slabs, warm interior glow per floor, subtle wall texture, roof with antenna/water tank, ground floor entrance/lobby with awning. Floor number labels on the left.
- Shafts: 3 vertical dark channels with rails, elevator cars with colored bodies, doors (2 panels sliding), floor indicator above each car or in a top panel showing current floor + direction arrow.
- Elevator colors: distinguishable — e.g. amber, coral/red, cyan? With the dusk teal background, cars could be: #ffb703 (amber), #ef476f (raspberry), #06d6a0 (mint)? These pop nicely.
- Micro-interactions: button hover/press effects, floor row highlight on hover, person sway, tooltip pop, elevator button glow when assigned, counter tick animation.

Sound? Skip (keep simple, autoplay issues).

Also add per-elevator indicator lights at top of shaft: current floor number + direction arrow.

Control panel design: a side panel styled like an elevator control room / panel: dark metallic with toggle-like buttons, LED indicators. Left panel ~300px, building takes the rest. On narrow screens, stack.

Implementation structure:

Constants:
- FLOORS = 8 (1..8). Floor height in px: computed from container height. Use fixed pixel layout for precision: building height = FLOORS * FLOOR_H + roof etc. FLOOR_H ~ 84px. Person ~ 34px tall.

Actually make it responsive: scale the building with transform? Simpler: fixed design size, allow horizontal scroll on small screens, or use CSS clamp. Build with fixed pixel geometry (JS uses px constants), and center it; on small screens, scale the whole stage with CSS transform based on viewport (JS computes scale = min(1, availW/designW)). That keeps animation math simple.

Layout:
- .app: grid: sidebar + stage.
- Stage contains .scene: sky background layers + .building.
- Building: width ~ 760px: shaft area (3 shafts × ~86px + gaps) on left, then lobby/corridor area where people stand on the right ~ 420px wide.

Hmm, people wait on each floor; queue extends horizontally. Building width needs to accommodate up to ~12 people per floor with spacing 30px → 360px. OK.

Per-floor structure:
- Floor slab (horizontal concrete band)
- Interior background (warm gradient, maybe furniture silhouettes? Keep subtle: window pattern on right wall)
- Queue container (absolute) where person divs are placed
- Shaft doors per elevator on that floor: 3 door units aligned with shafts.

Shafts run full height on the left. Cars positioned absolutely inside shafts, translateY.

Coordinate mapping: floor f (1-indexed from bottom). y position of the floor surface (top of slab) = buildingBottom - f*FLOOR_H ... let me define: building content height = FLOORS*FLOOR_H. Top of floor i surface (i=1..FLOORS) measured from top of building: y = (FLOORS - i) * FLOOR_H + (FLOOR_H - SLAB) ... cars align so their floor matches the walking surface of the floor.

Simplify: define floorY(i) = top offset of walking surface for floor i (people stand on it, car floor aligns). Car height ~ FLOOR_H - slab thickness - clearance.

Let me set: FLOOR_H = 88. SLAB = 12 (thickness of floor slab). Interior clear height = 76. Person height ~40. Car height = 70, width = shaft width - 8.

floorSurfaceY(i) (from top of floor stack) = (FLOORS - i) * FLOOR_H + (FLOOR_H - SLAB). People stand with bottom at that y. Car top when at floor i = floorSurfaceY(i) - CAR_H.

Doors: on each floor, each shaft has a door frame aligned with the shaft; doors slide open/close. Person boards: walks from queue x to door x (center of shaft), then hides, appears inside car.

Elevator state machine (per elevator):
States: IDLE, TO_PICKUP, DOORS_OPENING_PICKUP, BOARDING, DOORS_CLOSING_TO_DEST, TO_DEST, ARRIVED_DEST (doors open), ALIGHTING, DOORS_CLOSING, then look for next task.
Simplify with timed phases using animation loop with progress.

Movement: animate carY from current to target with easeInOutQuad over time proportional to distance (speed px/s ~ 160, min time). Use a generic tween helper.

Doors: doorOpen value 0..1 animated; panels translate accordingly.

Boarding: person element: it's positioned absolutely in the floor layer. On board, animate its x from queue position to door center (walking, with slight bob), duration ~ distance/speed. On arrival, hide the person and set car.passenger = {color, height, dest}; render a passenger figure inside the car. On destination arrival, reverse: create/figure walks out from car to a spot on the corridor (say 60px to the right of door), then fades out; delivered++.

Queue management: array of person objects per floor. Each person: id, dest, color, h (height scale), el, floor. Position: x = QUEUE_X0 + index*SPACING (+ small random jitter? Keep orderly: index*spacing). Use transform: translateX with transition for shifting.

Spawn: create person with pop-in animation (scale from 0, drop slightly). Random dest ≠ current floor.

Tooltip: each person element has data-tip="目标 · 5F"; CSS ::after tooltip on hover with transition. Make sure tooltip is visible above the person; may overflow building — set overflow visible appropriately. Person z-index above.

Also hover on person: slight scale up / highlight ring.

Dispatch logic:
- waitingByFloor: array of queues.
- assignedCount[floor] = number of elevators currently targeting that floor for pickup (TO_PICKUP or boarding there).
- findNextPickup(elev): among floors with waiting count - assigned > 0, pick the one minimizing distance |elevFloor - f|; tiebreak random or lower. Set elev.target = f, assigned++.
- On arrival & boarding: assigned-- (for that floor), remove front person, set destination.
- After drop-off: find next pickup; if none, IDLE (doors close, remain at floor).

Also, if elevator is idle and a new person spawns, trigger dispatch for idle elevators.

Edge case: multiple elevators arriving at same floor: each boards next person in queue; if queue becomes empty before arrival, elevator should re-route: on arrival, if no waiting people, cancel: doors stay closed, find next task. Handle: on TO_PICKUP arrival, check queue length; if 0, immediately re-dispatch (no door open). Also if person already boarded by another elevator during travel... assigned accounting mostly prevents overshoot; but if queue shrinks (shouldn't happen except via boarding), fine.

Direction indicator: arrow up/down based on movement; show "—" when idle.

Per-car panel at top of shaft: floor number (mono font), arrow. Also colored LED strip on car matching elevator color.

Control panel:
- Title: "电梯模拟器" with display font, small subtitle.
- Floor selection: row of floor buttons (1–8) — highlight selected. Or a select. Buttons are better. 8 buttons in grid 4×2.
- Number: stepper with − / + and value, range 1..12. Or slider. Stepper buttons are nice.
- Big "生成乘客" button (accent color) — spawns N people on selected floor with staggered pop.
- Secondary: "随机生成" (spawn 1–3 on random floors)? Nice addition. And "清空重置".
- Stats: 等待 / 运送中 / 已送达 counters with mono digits, tick animation on change.
- Elevator status: 3 rows with color dot, state text (空闲/前往接客/载客中…), current floor.

Also a small hint text: "提示：鼠标悬停查看目标楼层".

Header maybe integrated into sidebar top. Keep sidebar as "control room" panel.

Additional details:
- Blinking beacon on rooftop.
- Drifting clouds.
- Twinkling stars.
- Windows on building right side randomly lit? Floors have warm glow; maybe add small window grid on the right wall area behind people, some lit.
- Counterweight moving opposite in shaft? Nice detail but adds complexity; skip or simple: a thin counterweight bar moving opposite to car. Cheap: inside each shaft, a counterweight div with translateY = -(carY) mirrored. Let me add — nice touch. Position: counterY = (trackHeight - carY - carH). Just compute in render.

Cables: line from top of shaft to top of car, drawn as div with height = carY. Cheap: inside each shaft, cable div height = carY. Update in render.

Performance: rAF loop updates car transforms, cables, counterweights, door panels, indicators. People CSS-animated mostly; walking handled by JS tween on transform.

Person walking: implement as tween of person.x over time with walking bob (CSS class 'walking' adds bob animation on inner body). Person element positioned by transform translate(x, y) — y fixed per floor (bottom aligned). Use left:0; transform: translateX(x). Bottom set by floor layer.

Let me define DOM structure:

<div class="app">
  <aside class="panel"> ... controls ... </aside>
  <main class="stage">
    <div class="sky"> stars, moon, clouds, skyline </div>
    <div class="scene" id="scene"> (scaled)
      <div class="building">
        <div class="roof"> antenna, beacon, water tank? </div>
        <div class="halls"> (right part: floors)
          per floor: <div class="floor" data-floor="i">
             <div class="slab"></div>
             <div class="room"> window decor </div>
             <div class="queue"></div> (people appended here)
             <div class="floor-label">3</div>
          </div>
        </div>
        <div class="shafts">
          per elevator: <div class="shaft">
             <div class="rails"></div>
             <div class="cable"></div>
             <div class="counterweight"></div>
             <div class="car"> doors L/R, interior, passenger figure, LED </div>
             door frame per floor?
          </div>
        </div>
        Per-floor doors need to overlay shafts at each floor: place door units inside shaft at fixed y per floor: .hall-door per floor per shaft with 2 panels + frame; car behind. Actually visually: shaft is a vertical channel; at each floor a doorway (call button panel) exists. Car is behind doors; when doors open you see the car interior. So per-shaft, per-floor door assembly positioned at floor opening y. Car moves behind. Z-order: doors above car. When car is not at that floor, doors closed show dark metal. Good.

Simplify: each shaft contains: rails, cable, counterweight, car (z 2), then a .doors layer (z 3) containing FLOORS door units positioned at each floor opening. Door unit: frame + 2 sliding panels + small call lamp that lights when floor has waiting people.

Call lamp: nice — a lamp per floor per shaft? Real buildings have one per floor; but we have 3 shafts. Put a small lamp above the middle door? Or skip per-door lamps; instead show waiting glow. Add a small "call" indicator dot on the right side of the hall (shared per floor) that lights when queue non-empty, blinks. Place in hall: .call-lamp near queue start.

Top of building: sign "ELEV·SIM"? Maybe a rooftop sign with display font, glowing. Cute: neon-ish sign "电梯大厦" or "LIFT SIM". Let me do rooftop sign "LIFT·SIM" in Bungee, warm glow. Chinese title in sidebar.

Ground: below floor 1, a ground strip with entrance? Floor 1 is also usable (people spawn there too). Building sits on ground; add bushes/streetlamp in front for charm. Keep moderate.

Color palette (dusk):
- Sky: #0e1b2e → #1d3550 → warm horizon #e76f51 hint? Dusk teal to amber: top #0b1626, mid #14324a, bottom #b2543a-ish? Careful not to become too pink. Teal-to-amber sunset: top #0a1a2f, mid #10394f, low #d97b3f glow near horizon behind skyline. Stars in upper area.
- Building facade: deep slate blue #22303f with edge highlights; slabs #37475a lighter concrete; interior warm #ffd9a0 glow gradient.
- Shaft interior: very dark #0d1520 with rail lines.
- Cars: E1 amber #ffb703, E2 coral #ef6351, E3 mint/teal #2ec4b6. Car body colored, dark windows, glowing interior when occupied.
- Panel UI: dark steel #101820 with amber accent #ffb703, off-white text #e8eef4. Buttons chunky with pressed states.

Fonts: Bungee (display, Latin only — fine for "LIFT SIM" sign and digits), Manrope or "Outfit" for body Latin; Chinese text falls back to system (PingFang/Microsoft YaHei) — fine. Use font-family stack: display: 'Bungee', 'Microsoft YaHei', sans; body: 'Outfit', 'PingFang SC', 'Microsoft YaHei', sans-serif. Mono digits: 'IBM Plex Mono' or use Bungee for counters? Bungee digits are fun; use Bungee for big counters and floor labels. Keep to 2 fonts: Bungee + Outfit. Add via Google Fonts link with preconnect; fallbacks ensure offline function.

Now JS architecture:

const FLOORS=8, FLOOR_H=88, SLAB=14, N_CARS=3;
Geometry constants computed after DOM built.

Build DOM dynamically: floors, shafts, doors.

State:
queues = Array(FLOORS+1) of arrays.
elevators = [{id, color, y (px top of car), floorFloat, state, targetFloor, passenger, doors:0..1, tween:{...}, assignedFloor}]

Main loop: rAF; dt clamp.

Elevator update:
- If tween active: progress; y = ease(from,to,t). When done → onArrive.
- Doors animate toward target (open 1 / closed 0) at speed.
- State:
 IDLE: doors target 0 (closed). Check dispatch opportunities (call tryDispatch()).
 TO_PICKUP: moving; on arrive → if queue empty: tryDispatch again or idle. Else state=BOARDING: open doors; when doors >= 0.95 start boarding: take front person, assigned--, start walk-in animation; when person reaches car (callback): hide person, car.passenger=figure shown, state=LEAVING: close doors; when doors closed → start move to dest (TO_DEST).
 TO_DEST: on arrive → state=ALIGHT: open doors; when open: passenger figure walks out to drop-off spot (x on floor to right of door), fades; when done: passenger=null, delivered++, close doors; then tryDispatch.

tryDispatch(e): find best floor: candidates where queues[f].length - incoming[f] > 0; incoming counts elevators with assignedFloor===f (TO_PICKUP or boarding not yet boarded). Pick min |currentFloor - f|; tie → prefer direction continuation or random. Set state TO_PICKUP, assignedFloor=f, incoming[f]++, startMoveTo(f).

Global: on spawn and on elevator free, loop elevators and tryDispatch idle ones. Also after drop-off.

startMoveTo(e, floor): compute targetY, duration = dist/SPEED (SPEED ~ 150 px/s) + ease; set tween. Update direction indicator.

currentFloor display: round based on y during movement: floor = FLOORS - round((y - minY)/FLOOR_H)? Compute nearest.

Person walking tween: generic animator list: {el?, from, to, dur, t, onUpdate, onDone}. Use for person x movement and fades.

Person object: {id, floor, dest, hue/color, h, x, el, state:'wait'|'walking-in'|'riding'|'walking-out'}

Positioning: queue x: doorCenterX of the middle? People queue next to shafts. Shafts occupy left side of building; halls are to the right. Queue start x (in hall coords) = 26 (a bit away from doors), spacing 30. Person index i → x = 26 + i*30. When front boards, others shift: recompute x for each remaining, animate via CSS transition on transform (set transition on person element transform 0.5s ease; but walking uses same transform... conflict). Solution: person outer element positioned by style.left (with left transition) — left transition works but less performant; fine for small numbers. Or use transform but disable transition during walking (add class .walking { transition:none }). Let me do: outer .person { transition: transform .55s cubic-bezier }; when walking, add .free class (transition none) and drive transform via JS animator. Boarding: person needs to move from hall coords to door coords — different parent? If person is inside floor layer that spans full building width (including shaft area), then x is in building coords: door center x is known. So make each floor layer full building width, absolute; queue people positioned in building coords. Simpler: one .people-layer per floor spanning full width.

So structure: .building > .floors (stack of floor backgrounds) + .peopleLayer per floor? Let me restructure:

.building (position relative, fixed W×H)
  .shaftblock (left, contains 3 shafts full height) — z 1
  .floorbg × FLOORS (right area backgrounds + slabs full width?) slab should span full width including in front of shafts? Floor slab visually crosses the whole building; the shaft doors sit on slab line. Draw slab as full-width band at each floor bottom, but shaft area shows doors above slab.
  Actually simpler: draw per floor: .floorunit absolute positioned at floor y, height FLOOR_H, full width. Contains: room background (right part), slab (bottom full width). Shaft block drawn separately behind/above with doors; car moves inside shaft. Doors on shaft at each floor.

Z-order: shaftblock (with cars and doors) z2; floorunits z1 (backgrounds); people layer z3 (people walk in front of shaft doors? when entering door they should disappear at door — walk to door center then fade/hidden just as doors close; acceptable if person visible above doors until hidden). Person walks to door center X of assigned elevator shaft, then hidden.

Door center X: shaft k x0 = PAD + k*(SHAFT_W+GAP) + SHAFT_W/2.

Building width: SHAFT area = 3*SHAFT_W + 2*GAP + 2*PAD. SHAFT_W=78, GAP=10, PAD=14 → 3*78+20+28=282. Hall width = 460. Total ~ 742 + walls. Let me do BUILDING_W=760.

Queue capacity visual: (460-40)/30 ≈ 13 per floor. Limit spawn per click to 12; total per floor maybe cap at 15 with warning shake? Just allow; overflow goes off — clamp spawn input to 1..10. Fine.

Heights: FLOORS=8 × 88 = 704 + roof ~64 + ground 40 → scene height ~ 810. Design width: panel 300 + scene 800 → ~1140. Scale to fit.

Hmm 704 is tall; viewport 900 with header fits with scale ~0.95. OK. Maybe FLOOR_H=84 → 672. Keep 88 with person 42 for roomy.

Roof: parapet + sign + beacon + antenna. Ground: dark strip with door? Floor 1 hall serves as lobby.

Tooltip: .person::after content attr(data-tip); position above head; opacity 0 → 1 on hover, translateY. Add small arrow. Ensure z high. Person hover: outline glow.

Person visual: container 26 wide × h tall (h ~ 40±6). Inside: .head (circle 10px), .torso (rounded rect), maybe .legs? Simple: head + body capsule. Random color from palette list [amber, coral, mint, sky, lilac?]. Avoid too many hues; pick 6 pleasant ones: #ffb703, #ef6351, #2ec4b6, #4cc9f0? A single small person in lilac is fine; let me pick: '#ffb703','#ef6351','#2ec4b6','#5aa9e6','#e5989b','#94d2db'? Let me pick: amber #ffb703, coral #ff6b5e, teal #2ec4b6, blue #5b9bd5... hmm. Use: ['#ffb703','#ff6b5e','#2ec4b6','#6ab2ff','#d98bd6'?]. Keep to 5: amber, coral, teal, sky, warm gray? Fine.

Random: small chance of hat? Skip.

Idle animation: gentle sway (translateY 1px) with random delay per person; blink? Too small. Sway + occasional head tilt? Just sway.

Walking animation: faster bob + slight lean; add .walking class with keyframes bob.

Spawn pop: keyframes scale 0 → overshoot → 1.

Delivered exit: walk out then opacity 0 & scale .6, then remove.

Car interior passenger: .car contains .rider (mini person, static, no tooltip). Show/hide.

Elevator panel UI rows update via JS each frame or on change: state text mapping:
IDLE 空闲, TO_PICKUP 前往接客, BOARDING 上客中, TO_DEST 载客运行, ALIGHTING 下客中, CLOSING... simplify displayed states.

Also direction arrow: ▲ ▼ with color; idle ●.

Call lamps per floor: .lamp on right side? Position next to doors: small dot above middle shaft door per floor? Simplest: per floor, one lamp on hall left edge near shaft (like an up/down call button). Place at x = shaftblock width + 8, vertically centered in floor: small pill with dot; active class when queue>0 → glowing pulse.

Stats counters animate: on value change, add .bump class (scale flash), remove after.

Buttons: floor select buttons 1..8 (grid). Selected has accent. Count stepper: − value +. Spawn button big amber, hover lift, active press. Random button ghost. Reset button ghost red.

Also "速度" control? Not requested; skip to keep clean. Maybe add speed slider as bonus... keep clean.

Edge cases:
- Spawn on floor f with dest random from 1..FLOORS excluding f.
- Person boards elevator: dest known; elevator takes to dest.
- Reset: clear people, elevators to floor 1 idle, counters zero.

Initial state: maybe pre-spawn a few people (say 2 on floor 3, 1 on floor 6) so scene feels populated immediately. Yes — auto demo on load.

Sound: none.

Now let me write code carefully.

Geometry constants:
const FLOORS=8, FLOOR_H=88, SLAB=14, ROOM_H=FLOOR_H-SLAB (=74),
SHAFT_W=80, SHAFT_GAP=12, PAD_L=16 (left padding inside building for shafts),
SHAFTS_X0 = PAD_L,
HALL_X = PAD_L*2 + 3*SHAFT_W + 2*SHAFT_GAP (=16*2+240+24=296),
BUILDING_W = 780,
HALL_W = BUILDING_W - HALL_X - 16 (right margin) ≈ 780-296-16=468.
CAR_W = SHAFT_W-10 (=70), CAR_H = 64. Person height ~40 fits in 74 clear height. Car interior height 64 — person inside scaled to 0.8.

Floor stack: top of building content (floor area) at y=0 for topmost floor (floor 8) top. floorTop(i) = (FLOORS - i)*FLOOR_H. Walking surface y (top of slab) = floorTop(i) + ROOM_H (since slab is at bottom of floor unit of thickness SLAB). Person bottom = floorTop+ROOM_H. Car floor align: car bottom should equal walking surface → carTop(i) = floorTop(i)+ROOM_H-CAR_H = (FLOORS-i)*88 + 74-64 = (FLOORS-i)*88+10.

Door opening per floor in shaft: opening height = ROOM_H - maybe 6 top frame: door height 66, width SHAFT_W-16, positioned at floorTop(i)+ (ROOM_H-66)=floorTop+8, centered horizontally in shaft.

Building total height: FLOORS*FLOOR_H = 704. Roof zone above: 70. Ground below: 46. Scene size: W=BUILDING_W + margins? Scene = building + sky around. Let me make stage full area with sky background (fixed, covers stage), and scene (building + ground) centered.

Scale: JS computes scale = min(1, (stageW-40)/SCENE_W, (stageH-40)/SCENE_H) and applies transform scale to .scene, transform-origin top center. SCENE_W = BUILDING_W + 120 (side margins for streetlamps?) Let me do SCENE_W=860, SCENE_H=704+70+46=820.

Hmm 820 is tall; typical laptop 800 viewport minus padding → scale ~0.85. Acceptable. Could reduce FLOORS to 7? 8 floors is nice for sim. Keep 8, FLOOR_H=84 → 672, scene 788. Person 38. Let me do FLOOR_H=84, ROOM_H=70, SLAB=14, CAR_H=60, door h=62. Person height base 36 (+/-4). OK.

Cable: from shaft top to car top: cable height = carTop. Counterweight: small block moving opposite: cwTop = (trackH - carTop - CAR_H) where trackH = 704 - something. CW height 40, width 12, on right side of shaft. Rails: 2 vertical lines.

Car element: absolute positioned inside shaft, width CAR_W, left (SHAFT_W-CAR_W)/2 =5, top = carTop - floorTop? Car coords are inside shaft which spans full 704 height (shaft element height 704, top aligned with floor area top). So car.top = carTop(i) as computed (relative to floor area top). Shaft top aligns with floor area top. Good.

Door layer: per shaft, per floor door unit positioned at left 3, top floorTop(i)+4, width SHAFT_W-6, height 66. Contains .panel.left/.right (each 50% width, translateX ∓100% * openAmount). Frame border. Also dark .pit behind (shaft interior visible when open and no car → dark).

When doors open and car present, you see car interior. Car is behind doors (car z 2, doors z 3). Car has its own front doors? For simplicity, car has open front (interior visible) and floor-level doors act as both. When car at floor and doors open, you see rider inside. Good enough visually.

Car visual: colored body with roof LED strip (color), interior dark warm, rider figure, small floor number plate on car? Show current floor in top-of-shaft indicator: per elevator, .indicator above shaft in roof zone: shows floor number + arrow, colored border.

Dispatch & movement code:

let elevators=[];
function makeElevator(i){ return {i, x: shaftX(i), y: carTopY(1), state:'IDLE', doors:0, doorTarget:0, tween:null, assigned:-1, passenger:null, dir:0, floor:1}; }

carTopY(f) = (FLOORS-f)*FLOOR_H + (ROOM_H - CAR_H). With numbers: (8-f)*84 + 10.

startTween(e, toY){ from=e.y; dist=abs; dur = Math.max(0.45, dist/SPEED); SPEED=170 px/s; e.tween={from,to:toY,dur,t:0}; e.dir = toY<e.y?1:-1 (up=1); }

In update: if tween: t+=dt; k=min(1,t/dur); e.y=from+(to-from)*easeInOutQuad(k); if k>=1 → tween=null; onArrive(e).

Door speed: doors move toward doorTarget at rate 2.2/s.

onArrive(e):
 if state==='TO_PICKUP': const q=queues[e.assigned]; if(!q.length){ releaseAssigned(e); setIdleOrNext(e); } else { e.state='BOARDING'; e.doorTarget=1; }
 if state==='TO_DEST': e.state='ALIGHT'; e.doorTarget=1;

Update per state (each frame):
 BOARDING: if doors>0.96 && !e.boardingStarted: start: e.boardingStarted=true; person=q.shift(); releaseAssigned(e) (incoming--); updateQueuePositions(floor); personWalkIn(person, e, ()=>{ // done
    person.el.remove(); e.passenger = makeRider(person); showRider(e); e.state='DEPART'; e.doorTarget=0; });
 DEPART: when doors<0.05: startTween to carTopY(person.dest); state='TO_DEST'.
 ALIGHT: when doors>0.96 && !e.alightStarted: start: riderWalkOut(e, ()=>{ e.passenger=null; delivered++; e.state='CLOSE'; e.doorTarget=0; });
 CLOSE: when doors<0.05: setIdleOrNext(e).
 IDLE: doors target 0.

setIdleOrNext(e): e.state='IDLE'; tryDispatch(e).
tryDispatch(e): best floor f where queues[f].length - incoming(f) > 0; incoming = elevators.filter(o=>o.assigned===f && (o.state==='TO_PICKUP')).length — track e.assigned only during TO_PICKUP; clear after boarding. Compute score = abs(eFloorFloat - f). Pick min; tiebreak: prefer f in current direction, else smaller. If found: e.assigned=f; e.state='TO_PICKUP'; startTween(e, carTopY(f)); else remain idle.

eFloorFloat: compute from y: f = FLOORS - (e.y - 10)/FLOOR_H... carTopY(f)=(FLOORS-f)*84+10 → f = FLOORS - (y-10)/84.

Call dispatchAll() after spawn, after setIdleOrNext (already), and periodically in idle check (cheap): in loop, for each idle elevator with no tween, if any floor has demand, tryDispatch. Guard with flag to avoid spam: tryDispatch only sets if found; if not found, set e.nextCheck = now+0.5 to avoid per-frame scan; fine, scan is trivial (8 floors × 3 elevators).

Person walking-in: from current x (building coords) to door center X of elevator shaft minus small offset; person element is inside floor people layer (absolute positioned in building coords, full width). Walk distance ~ hall to shaft. Duration = dist/ WALK_SPEED (90 px/s), min .5. During: add .walking class; animate x via animator; on done: callback. Person y stays at floor surface.

Also ensure doors remain open during walking (state BOARDING waits for callback before DEPART). Good.

RiderWalkOut: create person element on destination floor at door center X, walk to x = HALL_X + 60 (+ random 0..40), then fade & remove. delivered++ after fade start. Rider figure color from passenger.

Rider inside car: .rider div inside car, styled like mini person (scale .85), shown when passenger. Position at bottom center of car.

updateQueuePositions(f): for each person in queues[f] index i → targetX = QUEUE_X0 + i*SPACING; set person.x and el.style.transform translate(x, bottomY)? Person element positioned by left? Let me position person with style.transform = translate(x px, y px), element at top-left of building, y = surfaceY - personH. Transition transform .5s for queue shift. When walking (JS-driven), add class .noTrans.

Person element structure:
<div class="person" data-tip="目标 · 5F" style="--pc:#ffb703; height:h">
  <div class="p-inner"><div class="p-head"></div><div class="p-body"></div></div>
</div>
Sway animation on .p-inner (so transform positioning isn't affected).

Tooltip via ::after on .person using attr(data-tip). Positioned bottom: 100% + 6px, center. White-space nowrap.

Hover: .person:hover .p-inner scale 1.08 + glow; cursor default? pointer.

Spawn: function spawn(floor, n): for k in n: setTimeout staggered k*120: createPerson(floor). Random dest ≠ floor. Push to queue; position at index; pop animation. dispatchAll().

Cap: if queue length > 14, ignore extra (flash hint). Provide hint message element in panel for feedback ("已在 3F 生成 4 名乘客").

Reset: remove all person els, queues=[], elevators reset to floor 1, state idle, doors 0, tween null, riders cleared, counters zero, incoming cleared.

Stats: waiting = sum of queue lengths; riding = elevators with passenger; delivered counter. Update DOM on change (poll each frame cheaply or event-driven; let me just compute each frame and only update text on change).

Per-elevator indicator: floor number = nearest floor from y: Math.round(FLOORS - (y-10)/84) clamp 1..8. Arrow: dir.

Elevator status text: map:
IDLE→'待机', TO_PICKUP→'接客中', BOARDING→'上客', DEPART→'关门', TO_DEST→'载客', ALIGHT→'下客', CLOSE→'关门'. Show target like "→5F".

Panel elevator row: color chip, name "1号梯", state text, floor badge. Update on change.

Now visual CSS details — I'll write carefully:

Sky: stage background: linear-gradient(#081120 0%, #0d2236 45%, #1b4a5e 75%, #b4552f? ...). Dusk teal → amber:
background: linear-gradient(180deg, #070f1d 0%, #0c2136 38%, #14425a 62%, #7a4a33 88%, #c96b3a 100%)? Horizon glow behind building. Add radial glow near horizon: radial-gradient at 50% 88%.

Stars: generate ~70 dots in JS in top 60%, with random animation-delay for twinkle animation. Moon: circle with glow, top-right. Clouds: 2-3 blurred oval divs drifting slowly (translateX loop, 120s). Skyline: distant buildings silhouette (CSS: repeating linear gradient? Better: inline SVG or box-shadow trick). Simplest: two .skyline divs with background: linear-gradient(transparent, #0a1622) and clip-path? Use JS to generate skyline: create divs with random heights in a flex row, dark colors, some lit windows (small dots via background-image gradients). Simpler: build skyline with many small window divs? Keep cheap: buildings as divs with background #0b1a29 and background-image: radial dots for windows via repeating-linear-gradient with low opacity. Fine.

Ground: dark strip with subtle gradient; streetlamp with glow cone? Add one streetlamp to right of building for charm: pole + lamp + radial glow. And bushes (rounded divs).

Building facade: outer wall color #2b3947? With left/right edge pilasters darker. Floor units: room background: linear-gradient warm dim: rgba(255,190,110,.14) to transparent, back wall pattern: vertical panel lines? Add "window" strip on right edge of each floor: a bright rectangle (like a lit window) with frame. And slab: concrete #46586c top highlight, front face #33424f.

Also add subtle room lighting variation: some floors randomly get stronger glow (set via inline style opacity on .glow at build).

Shaft block background: dark panel #10181f with border. Shafts: interior gradient dark, rails: 2 vertical lines via repeating gradient? Use ::before with 2 vertical lines (left 12, right 12) color #1e2a35.

Doors: panels metallic gradient #5a6b7c → #3c4a58 with center seam line; frame border #22303c; small transom light above each door? Add .d-lamp: small dot above door that lights when that elevator is at/near that floor? Extra; skip. Call lamp per floor in hall: pill with up/down arrow? Just a glowing amber dot when waiting.

Car: body: colored roof strip (elevator color), body: gradient steel #6b7c8c with big window showing interior? Front view: car face = doors? Since floor doors act as car doors, car face should show interior when open. Design: car element = open box: colored frame sides/top, interior warm lit, rider stands inside. When floor doors closed, you see floor doors (metallic) — car hidden behind. When both open (aligned), you see interior. But during travel, car is visible between floors: you see car body (colored frame + interior + rider). Looks like an open-front car — acceptable, actually charming, like a dollhouse. Add subtle glow beneath car. Also colored LED strip on top of car + car number.

Indicator housing above shafts (roof zone): small dark box with Bungee amber digits + arrow.

Roof: parapet band, sign "LIFT·SIM" letter glow (Bungee), antenna with blinking red beacon, maybe small water tank. Keep: sign + antenna + beacon + vents.

Lobby/ground floor: floor 1 room has entrance vibe: plants? Add potted plant decor to a fixed floor (like floor 1 right side) — but people stand there; fine, decor behind (z low).

Panel design: width 300, dark steel background with subtle top light border; sections separated. Title: "电梯模拟器" big bold, small tagline "ELEVATOR SIM · 三梯 × 八层". Floor buttons grid 4×2: chunky, selected amber. Stepper: [-] 4 [+]. Spawn button full-width amber, dark text, shadow, hover translateY(-2px), active press. Random & Reset: half-width ghost buttons. Stats: 3 tiles with big Bungee numbers (等待/运送/送达). Elevator row: color dot pulsing when active, name, state, floor. Hint footer: "悬停小人可查看目标楼层".

Feedback toast: small line under spawn button showing last action, fade.

Responsive: @media max-width 980: panel becomes top bar horizontal scroll? Simpler: grid → column; panel becomes row-wrapped sections. Keep basic: column layout, stage min-height 70vh, scene scaled to width.

Scale computation: function fit(){ const r=stage.getBoundingClientRect(); s=Math.min((r.width-32)/SCENE_W,(r.height-32)/SCENE_H,1.15); scene.style.transform=`translate(-50%,0) scale(${s})`; } Scene positioned absolute left 50% top 12. Also allow scale >1 slightly? Cap at 1.

SCENE_W: BUILDING_W(780) + streetlamp margin: 900. SCENE_H: roof 84 + floors 672 + ground 56 = 812.

Hmm tall. Stage height: 100vh minus padding; on 1080 screen fine; on 768 laptop: scale ≈ (768-40)/812 ≈ 0.89 → OK.

Let me define exact numbers:
FLOORS=8, FLOOR_H=84, SLAB=13, ROOM_H=71,
SHAFT_W=80, GAP=12, PAD=16,
SHAFTS_W = 3*80+2*12 = 264; shaftblock left=PAD=16, width 264+? plus inner padding: shaftblock width = 264 + 16 (inner 8 each side) = 280? Let me do: shaftblock x=14, width = 3*SHAFT_W + 2*GAP + 2*10 = 264+20=284. Shaft k left inside block = 10 + k*(80+12).
HALL_X = 14+284+10 = 308 (people queue start ~ 322).
BUILDING_W = 780. Right margin inside 14 → hall usable width = 780-308-14=458. SPACING=30, QUEUE_X0=326. Max visible ~ (780-14-326)/30 ≈ 14. Good.

CAR_W=68 (shaft 80 → margin 6), CAR_H=58. carTopY(f)=(FLOORS-f)*84 + (71-58)= (8-f)*84+13.
Door unit: height 64, top = floorTop+ (71-64)=floorTop+7? Let me do doorH=62, top=floorTop+6, left=4, width=72.

Person height: base 34 + rand*6 (34–40). Fits in room 71 and car 58 (rider scale .8 → ~30).

surfaceY(f) = floorTop(f)+ROOM_H = (8-f)*84+71. Person top y = surfaceY - h.

Queue x: QUEUE_X0=330, SPACING=30.

Walk-in target x: doorCenterX(e) = shaftblockX + 10 + e.i*(92) + 40 → building coords: 14+10+ e.i*92 +40 = 64 + e.i*92. Person walks from hall to that x; person center vs left: person element width 24, anchor left; target left = doorCenter-12.

Rider walk-out target: HALL_X + 40 + rand*80 → ~ 350–430.

OK. Indicator: place above each shaft in roof area: x aligned with shaft.

Now write animator helper:
const anims=[]; function animate(obj){anims.push({...obj,t:0});}
Each frame: t+=dt; k=min(1,t/dur); obj.step(ease(k)); if done remove & obj.done&&obj.done().

Use for person walking (step sets transform), fade, etc.

Person transform: el.style.transform = `translate(${x}px, ${y}px)`. Save p.x,p.y. Queue shift: since transition enabled (default), just set style transform → CSS transition animates. But walking uses animator with .free class (transition:none). After boarding done... person removed. Walk-out person also .free.

Potential conflict: initial spawn sets transform immediately with transition → will slide from 0,0. To avoid, set initial transform with .free class, force reflow, then remove class. Or set transition none first. Let me do: el.classList.add('free'); set transform; void el.offsetWidth; el.classList.remove('free'); then pop animation class.

Tooltip content: `目标 · ${dest}F` plus maybe "等待于 3F". Keep short: `目的楼层 ${dest}F`.

Add title attribute too? No, custom tooltip enough.

Now elevator indicator DOM: .ind per elevator in roof: `<div class="ind"><span class="ind-f">1</span><span class="ind-a">▲</span></div>` colored border-left.

Panel elevator row: `<div class="erow"><i class="dot" style="--c"></i><div class="einfo"><b>1 号梯</b><span class="est">待机</span></div><span class="efloor">1F</span></div>`.

State text mapping function.

Counter bump: wrap number in span; on change add class 'bump', remove after 300ms.

Toast: div#toast in panel; show(text): set text, class show, timeout to hide.

Initial demo: spawn(3,2), spawn(6,1), spawn(1,2)? Let me do: after load, spawn 2 on floor 2, 1 on floor 5, 2 on floor 7 with slight delay. Nice motion immediately.

Also add keyboard? Skip.

Reset button: confirm? Just do it.

Random button: pick random floor, n=1..3.

Prevent spawn when floor queue > 14: toast "该楼层已满".

Also handle visibilitychange: clamp dt (max 0.05) to avoid jump.

Now let me write CSS carefully. Sizes: panel width 304px; app grid: grid-template-columns: 304px 1fr; height 100vh; overflow hidden. Stage position relative; overflow hidden.

Font link:
<link href="https://fonts.googleapis.com/css2?family=Bungee&family=Outfit:wght@400;600;800&display=swap" rel="stylesheet">

Font stack: --disp:'Bungee','Outfit','PingFang SC','Microsoft YaHei',sans-serif; --body:'Outfit','PingFang SC','Microsoft YaHei',system-ui,sans-serif. Bungee is Latin-only for sign/digits; Chinese falls back to Outfit/PingFang bold — fine.

Colors:
--bg:#070f1b; --panel:#0e161f; --panel2:#131e29; --line:#22303d; --ink:#e9f1f7; --mut:#8fa3b4; --amber:#ffb703; --amber2:#ffd166; --coral:#ff6b5e; --teal:#2ec4b6; --sky:#5fc2ff?

Elevator colors: ['#ffb703','#ff6b5e','#3ddad0']? teal-ish. ok: E1 #ffb703, E2 #ff6b5e, E3 #4cc9f0 (sky blue). Distinct from person colors maybe; fine.

Person palette: ['#ffd166','#ff8f6b','#63d3b6','#7cc4ff','#e78fb3','#c3a6ff']? One lilac in a crowd is fine. Use: #ffd166 (sand), #ff8f6b (coral), #63d3b6 (mint), #7cc4ff (sky), #e78fb3 (rose), #b8e05a? lime. Pick 6: ['#ffd166','#ff8f6b','#63d3b6','#7cc4ff','#e78fb3','#9ad1ff']? Duplicate blue. Final: ['#ffd166','#ff8f6b','#63d3b6','#7cc4ff','#e78fb3','#d4e09b'].

Now let me write everything. Aim for clean, commented code.

Door panel detail: .d-panel left: width 50%, transform translateX(0); open: translateX(-96%). Right mirrored. Transition handled per frame via style (set transform from doors value): left panel tx = -doors*100%, right = +doors*100%. Add slight ease on doors value (linear move with easing at ends? Linear fine, rate constant).

Also add door frame inner shadow, and a "shaft dark" behind: .shaft::before dark.

Car detail DOM:
<div class="car" style="--c:#ffb703">
  <div class="car-led"></div>
  <div class="car-no">1</div>
  <div class="car-in"><div class="rider">...</div></div>
  <div class="car-base"></div>
</div>
Car height 58: led strip 6px top, interior rest. Rider positioned bottom center of interior.

Cable: .cable absolute top 0 left center width 2, height = carTop (set each frame). Counterweight: .cw right:2 width 10 height 34, top = (TRACK_H - (carTop) - 34)? Mirror: cwTop = TRACK_H - carTop - CAR_H - something... just cwTop = (TRACK_H - CAR_H) - carTop, TRACK_H=672. When car at bottom (carTop=672-58=614... wait carTopY(1)= (8-1)*84+13=601; track height 672; car bottom 659; cwTop = 672-58-601=13 top. When car at top carTop=13 → cwTop=672-58-13=601 bottom.

Add subtle shaft lighting at each floor: small glow line at door top when doors open? Skip.

Call lamp: .call per floor positioned at x=HALL_X-? Position at left: 296 (just right of shaftblock), vertically middle of floor: small vertical pill 8×18 with dot; active → amber glow + pulse.

Floor label: on left edge? Shafts occupy left. Place floor number on right side of hall: big ghost Bungee number low opacity on right of each floor: .fno at right:18, font Bungee 28px, color rgba white .08 → subtle depth. Also small label near left of hall? Right ghost number is classy.

Room decor: on each floor, add a .win (lit window) on right edge: rectangle 46×34 with frame, warm glow, random flicker? Static with random warmth opacity. Place at right:70? Ghost number right:14, window right: 90. People queue extends to ~766 max; window at right side may overlap with people visually behind — fine (z below people).

Also add wall clock? Skip.

Ground decor: entrance mat? Building sits on ground slab; add 2 bushes + streetlamp on right at x≈BUILDING_W+30.

Roof sign: "LIFT SIM" Bungee 22px letter glow amber, dark board with border; antenna line + beacon with red blinking animation; small vent boxes.

Now state display text:
const STXT={IDLE:'待机',TO_PICKUP:'前往接客',BOARDING:'开门上客',DEPART:'关门',TO_DEST:'载客运行',ALIGHT:'开门下客',CLOSE:'关门'};
Also show target: if TO_PICKUP → `→ ${f}F 接客`; if TO_DEST → `→ ${dest}F`.

Compose einfo span.

Update panel rows each frame only on change (cache string).

Stats: waiting computed; riding; delivered.

Edge: person's dest may equal elevator current... fine.

Let me write easeInOutQuad k<.5?2k²:1-((-2k+2)²)/2.

Walk speed 95 px/s.

Boarding sequence timing: doors open rate 2.4/s → ~0.42s. Walk-in distance ~ hall person x (330+) to door (~64..248): 80–400px → 0.9–4s. Reasonable.

Alight walk-out: door center → ~360+rand60: distance 110–300.

Doors stay open until walking done — good, realistic.

Multiple elevators same floor: doors open independently; queue shift handles.

Another detail: when person is walking in, they've left the queue (shifted already), others advance — nice visual.

Incoming accounting: e.assigned set at TO_PICKUP dispatch; cleared when boarded (or when queue empty on arrival). incoming(f) = elevators.filter(o=>o.assigned===f).length. Demand = queues[f].length - incoming(f). Candidates demand>0.

Tiebreak: sort by distance, then by |?|, pick first; add small random.

Also, elevator just dropped off at floor f, and there are people waiting at same floor: distance 0 → picks immediately.

Idle scan: in loop, for each elevator if state IDLE && !tween: tryDispatch(e).

tryDispatch returns bool.

Spawn staggering: setInterval-like via setTimeout chain; guard reset during? Trivial, accept.

Now let me write HTML skeleton:

<!DOCTYPE html><html lang="zh-CN"><head>meta, title 电梯模拟器, fonts, <style>...</style></head>
<body>
<div class="app">
 <aside class="panel">
   <header class="p-head"><h1>电梯模拟器</h1><p class="sub">ELEVATOR SIM · 三部电梯 × 八层</p></header>
   <section class="sec"><div class="sec-t">生成乘客</div>
     <div class="floors" id="floorBtns"></div>
     <div class="row">
       <div class="stepper"><button id="minus">−</button><div class="num" id="numVal">3</div><button id="plus">＋</button></div>
       <button class="btn primary" id="spawnBtn">生成乘客</button>
     </div>
     <div class="row">
       <button class="btn ghost" id="randBtn">随机生成</button>
       <button class="btn ghost warn" id="resetBtn">清空重置</button>
     </div>
     <div class="toast" id="toast"></div>
   </section>
   <section class="sec"><div class="sec-t">运行状态</div>
     <div class="stats">
       <div class="stat"><b id="stWait">0</b><span>等待</span></div>
       <div class="stat"><b id="stRide">0</b><span>运送中</span></div>
       <div class="stat"><b id="stDone">0</b><span>已送达</span></div>
     </div>
     <div class="elist" id="elist"></div>
   </section>
   <footer class="p-foot">提示：将鼠标悬停在小人上，可查看 TA 的目标楼层。</footer>
 </aside>
 <main class="stage" id="stage">
   sky layers... <div class="scene" id="scene"> building built by JS + roof + ground decor </div>
 </main>
</div>
<script>...</script>

Sky inside stage: .stars (JS fills), .moon, .cloud c1 c2 c3, .skyline (JS builds), .glowhor.

Scene content built mostly by JS into #building container; roof and ground static HTML? Build all in JS for consistency with constants. Scene HTML: <div class="scene" id="scene"><div class="roof" id="roof"></div><div class="building" id="building"></div><div class="ground"></div> decor </div> with CSS positioning: scene width SCENE_W height SCENE_H; building absolute left ((SCENE_W-BUILDING_W)/2), top ROOF_H.

SCENE_W=900 → building left=60. ROOF_H=86. Ground height 54. SCENE_H=86+672+54=812.

Streetlamp at right of building: x=60+780+24.

Now let me write JS build functions:

buildScene():
 - roof: parapet div, sign, antenna+beacon, vents; indicators for elevators positioned above each shaft: x = BL + shaftblockPad... compute shaftCenterX(i) building-relative, then add building offset; place indicators inside roof (roof spans building width, positioned above).
 - building: shaftblock div with 3 shafts (each: rails via CSS, cable, cw, car, doors layer with FLOORS door units).
 - floor units: for f=1..8: .floor at top floorTop, height FLOOR_H: .room (right area from HALL_X-? actually room background spans from shaftblock right edge to right wall), .slab full width, .fno number, .win window, .call lamp.
 - people layer: one .people per floor absolute positioned at floorTop, height ROOM_H, full width, z 5 — append persons with y computed relative to layer? Simpler: person y in layer coords: y = ROOM_H - h. Layer left 0 width BUILDING_W. But layer needs to be above doors (z). Door z inside shaftblock stacking; give people layer z-index 6, shaftblock z 3, floor z 1. Person walks above shaft door area — visually OK (in front of door).

Persons appended to floor's people layer.

Person y within layer = ROOM_H - p.h (bottom = ROOM_H = surface). Good; SLAB below.

Tooltip overflow: building overflow visible; stage overflow hidden but tooltip is small.

Cars: create per elevator with rider placeholder.

Indicators: inside roof: left = buildingLeft + shaftCenter - w/2.

Now JS logic as planned. Let me write update loop:

let last=performance.now();
function loop(now){ dt=min(.05,(now-last)/1000); last=now; stepAnims(dt); elevators.forEach(updateElevator); render(); updateHUD(); requestAnimationFrame(loop);}

updateElevator(e,dt):
 // doors
 const dTarget = (e.state==='BOARDING'||e.state==='ALIGHT')?1:0;
 move e.doors toward dTarget at rate 2.6.
 // tween
 if(e.tween){...on complete → arrive(e)}
 // state gates
 switch... as above with flags e.bsy (boardingStarted) etc. Reset flags on state transitions.

arrive(e): if TO_PICKUP: if queues[e.assigned].length===0 { e.assigned=-1; e.state='IDLE'; } else { e.state='BOARDING'; e.floor=e.assigned; }
 if TO_DEST: e.state='ALIGHT'; e.floor=e.passenger.dest;

BOARDING handling: if(e.doors>0.95 && !e.bsy){ e.bsy=true; const p=queues[e.floor].shift(); e.assigned=-1; layoutQueue(e.floor); walkIn(p,e,()=>{ p.el.remove(); e.passenger={color:p.color,h:p.h,dest:p.dest}; showRider(e,true); e.state='DEPART'; });}
DEPART: if(e.doors<=0.02 && !e.tween){ startMove(e, carTopY(e.passenger.dest)); e.state='TO_DEST'; }
ALIGHT: if(e.doors>0.95 && !e.bsy){ e.bsy=true; walkOut(e, ()=>{ showRider(e,false); e.passenger=null; delivered++; e.state='CLOSE'; }); }
CLOSE: if(e.doors<=0.02){ e.state='IDLE'; }
IDLE: tryDispatch(e).

startMove(e,y): e.tween={from:e.y,to:y,dur:Math.max(.5,Math.abs(y-e.y)/SPEED),t:0}; e.dir = y<e.y?1:-1;

render(): for each elevator: car.style.transform=translateY(e.y); cable height; cw top; door panel transforms (need per-floor door refs: doors[e.i][f] with panels). Only doors at e.floor open — door open state tied to elevator when at floor: doorOpenAmount for floor f shaft i = (e.doors if e aligned at f, else 0). Aligned = |e.y - carTopY(f)|<2 && (state in BOARDING/ALIGHT/DEPART/CLOSE/TO_*) basically when e.floor===f. Track e.floor (last arrived floor). During tween, floor is stale but doors are 0 anyway. So panel transform = e.doors applied only to doors[e.i][e.floor]. Others stay closed (set once). Manage: when e.floor changes, reset previous door transforms to 0.

render also: indicator text/arrow; call lamps: toggle class if queues[f].length>0.

updateHUD: stats and elevator rows.

walkIn(p,e,done): p.el.classList.add('free'); const tx=shaftCenterX(e.i)-12; const dist=Math.abs(tx-p.x); animate({dur:Math.max(.5,dist/WALK), step:k=>{p.x=lerp(p.x0,tx,k); p.el.style.transform=...}, done:()=>{p.el.classList.remove('walking'); done();}}); save p.x0=p.x first; add .walking.

walkOut(e,done): create person at dest floor layer at x=shaftCenterX-12, y; class free walking; target tx=HALL_X+30+Math.random()*90; after reaching: fade: animate opacity → then remove; delivered incremented by caller after fade starts. Let me increment on walk completion, then fade.

Rider figure: .rider div inside car-in; set --pc color and height scale; show class.

showRider(e,show): e.riderEl.style.display... set color.

layoutQueue(f): queues[f].forEach((p,i)=>{ p.x=QUEUE_X0+i*SPACING; p.el.style.transform=`translate(${p.x}px, ${p.y}px)`; }); (with transition enabled since not .free).

createPerson(f): dest = random excluding f; color random; h=34+rand*7; el built; p={...}; initial x=QUEUE_X0+queues[f].length*SPACING + maybe spawn offset: spawn at x+40 and slide in? Pop in place with scale animation. Set transform with free trick. Append. queues[f].push(p). Waiting++.

Tooltip data-tip=`目标楼层 ${dest}F`.

Also add small random delay to sway via style.animationDelay.

HUD cache: string compare.

Fit function + resize listener.

Stars: 80 spans random left/top (0–55%), size 1-2.5px, delay for twinkle.

Skyline: container bottom aligned to ground line: generate ~26 buildings widths 30–70 heights 60–180, color #0a1826, some with lit windows: add child dots? Use background-image: repeating-linear-gradient for window grid with low opacity. Two layers (far lighter, near darker). Position behind scene (z below scene). Stage position absolute inset 0, aligned bottom to ground visual — ground line is at stage bottom area depending on scale... tricky. Simplify: skyline positioned absolute bottom: 8% of stage, full width, z 0; scene z 1. Ground strip in scene covers building base; skyline behind appears on horizon. Good enough aesthetically.

Clouds: absolute, drift keyframes translateX(-10vw → 110vw) 90–160s linear infinite, blurred rounded shapes, opacity .12.

Moon: top right with box-shadow glow.

Also subtle vignette overlay on stage.

Toast style: min-height, small, amber text, opacity transition.

Buttons CSS:
.btn {font-family body 700; padding 10px 14px; border-radius 10px; border:1px solid var(--line); background:#18242f; color:var(--ink); cursor:pointer; transition: .18s; }
.btn:hover{transform:translateY(-1px); border-color:#3a4c5d; background:#1d2c39}
.btn:active{transform:translateY(1px) scale(.98)}
.primary{background:linear-gradient(180deg,#ffcf3f,#ffb703); color:#241a02; border-color:#b98300; box-shadow:0 6px 16px rgba(255,183,3,.25); font-weight:800}
.primary:hover brighter.

Floor buttons: .fbtn {height 38px; Bungee? digits} grid 4 columns gap 6. Selected: amber background dark text + glow.

Stepper: flex; buttons 36px square; num Bungee 20px width 40 center.

Stats: grid 3; tile bg #121d28 border; b Bungee 26 amber/teal/coral color per stat: waiting amber, riding sky, delivered teal. Bump animation scale 1.25.

Erow: grid; dot 10px round with color + pulse when active; name 12px; state 11px muted; floor badge Bungee.

Panel scrollbar styling? Panel may need overflow-y auto on short screens: overflow:auto.

Now let me finalize numeric constants in JS:

const FLOORS=8, FLOOR_H=84, SLAB=13, ROOM_H=71;
const SHAFT_W=80, SHAFT_GAP=12, SB_PAD=10;
const SB_X=14, SB_W=3*SHAFT_W+2*SHAFT_GAP+2*SB_PAD; //284
const HALL_X=SB_X+SB_W+8; //306
const BUILDING_W=780;
const CAR_W=68, CAR_H=58;
const DOOR_H=62, DOOR_W=72;
const QUEUE_X0=HALL_X+26; //332
const SPACING=30;
const ROOF_H=88, GROUND_H=56;
const SCENE_W=900, BL=(SCENE_W-BUILDING_W)/2; //60
const SCENE_H=ROOF_H+FLOORS*FLOOR_H+GROUND_H; //88+672+56=816
const SPEED=175, WALK=95, DOOR_SPEED=2.6;

carTopY(f)=(FLOORS-f)*FLOOR_H+(ROOM_H-CAR_H); // +13
surfaceY(f)=(FLOORS-f)*FLOOR_H+ROOM_H;
shaftLeft(i)=SB_PAD+i*(SHAFT_W+SHAFT_GAP); // inside shaftblock
shaftCenterX(i)=SB_X+shaftLeft(i)+SHAFT_W/2; // building coords

floorTop(f)=(FLOORS-f)*FLOOR_H.

Doors per shaft: top=floorTop(f)+ (ROOM_H-DOOR_H) → +9; left=(SHAFT_W-DOOR_W)/2=4.

Indicator top: roof area; y position: roof height 88; indicator box height 34 positioned at bottom-aligned near building top: top = ROOF_H-46? Roof element spans 0..ROOF_H above building. Indicator inside .roof at top: ROOF_H-50, left = BL+shaftCenterX(i)-24 (width 48).

Sign centered? Indicators take left area above shafts; sign positioned right side of roof: right: 30. Antenna left corner? Place antenna at right end near sign... Composition: parapet full width; sign board right center; antenna+beacon at left above shafts? Indicators are there. Antenna at right end, sign middle-right, small vent box. Fine — arrange: indicator row (left), vent, sign (right), antenna at far right with beacon.

Ground: full scene width strip, height GROUND_H at bottom: dark asphalt gradient; add sidewalk line. Bushes: 2 rounded divs near building base; streetlamp on right: pole height 90 rising from ground into building area — position absolute right area; glow ellipse.

Also doormat in front? No.

Building walls: .building background: facade color with side edges: use border-left/right 6px solid darker + subtle vertical gradient. Also add thin left/right wall strips for depth (absolute .edge.l/.r width 8 darker).

Shaftblock: absolute left SB_X top 0 height 672 width SB_W; background #0c141c; border:1px solid #1d2a36; border-radius 6px; overflow hidden? overflow hidden clips doors? Doors are inside. Cars inside. Yes, overflow hidden fine (cars don't exceed). But cable from top OK.

Floor unit: absolute left 0 width BUILDING_W top floorTop height FLOOR_H:
 .room: absolute left HALL_X-6? Room background should cover hall area behind people: left: SB_X+SB_W (=298), right:8, top:0, height ROOM_H: warm gradient + back wall line. Also add subtle ceiling shadow.
 .slab: absolute bottom 0 left 0 right 0 height SLAB: concrete gradient + top edge highlight line.
 .fno: right 16 top 8 Bungee 30 rgba(233,241,247,.07) content f.
 .win: right 78? Let me do: right: 84, top 12, w 44 h 30: frame border, warm fill opacity random .25–.6, plus random flicker animation on some? Static fine.
 .call: left: HALL_X-2 (=304)? overlaps room edge; place at left: HALL_X+2, top: (ROOM_H-20)/2: pill 10×20 dark with dot 4px; active dot amber glow pulse.

People layer: absolute left 0 top floorTop width BUILDING_W height ROOM_H z 6. Person y within layer = ROOM_H - h.

Wait: tooltip above person may be clipped by layer height? Layer overflow default visible — fine.

Z-order inside building: floor units z1, shaftblock z3, people layers z6, roof separate.

Person CSS:
.person{position:absolute; left:0; top:0; width:24px; transition: transform .55s cubic-bezier(.3,.7,.3,1); will-change:transform;}
.person.free{transition:none}
.p-inner{position:absolute; bottom:0; left:50%; transform:translateX(-50%); width:20px; height:100%; animation: bob 2.6s ease-in-out infinite;}
.p-head{width:11px;height:11px;border-radius:50%;background:var(--pc); margin:0 auto; box-shadow: inset -2px -2px 0 rgba(0,0,0,.18);}
.p-body{width:16px; height:calc(100% - 13px); margin:1px auto 0; background:var(--pc); border-radius:7px 7px 4px 4px; box-shadow: inset -3px -3px 0 rgba(0,0,0,.16);}
Head and body same color = onesie look; cute. Maybe head skin color? Same color = stylized figure; fine, clean.

Actually nicer: head neutral (#f2d6b3 skin?) with colored body. Let me do head #ffd9b0 with slight shading, body var(--pc). More readable as "person". Yes.

.person:hover .p-inner{filter:brightness(1.15) drop-shadow(0 0 6px rgba(255,215,120,.6)); }
Tooltip: .person::after{content:attr(data-tip); position:absolute; bottom:calc(100% + 8px); left:50%; transform:translateX(-50%) translateY(4px); background:#0d1720; color:#ffd166; border:1px solid #2c3d4d; padding:4px 8px; border-radius:6px; font-size:11px; white-space:nowrap; opacity:0; pointer-events:none; transition:.18s; z-index:50; font-weight:600}
.person::before{arrow triangle} use border trick, opacity synced.
.person:hover::after{opacity:1; transform:translateX(-50%) translateY(0)}

Walking bob: .walking .p-inner{animation: walkbob .32s linear infinite} faster + tilt.

Spawn pop: .pop .p-inner{animation: pop .45s cubic-bezier(.34,1.56,.64,1), bob ... } combining is tricky; let me apply pop on .person outer? Outer transform used for position... use scale on .p-inner via keyframes pop then switch to bob: set animation: pop .45s; on animationend switch to bob class. Simpler: use a separate wrapper for scale: add .p-scale wrapper between outer and inner to handle pop/bob? Structure: .person > .p-pos (handles pop scale animation) > .p-inner (bob) > head/body. Pop keyframes on .p-pos: scale 0→1. Bob on .p-inner.

.person .p-pos{position:absolute; bottom:0; left:50%; transform:translateX(-50%); width:20px; height:100%;}
.pop .p-pos{animation:pop .5s cubic-bezier(.34,1.56,.64,1)}
@keyframes pop{0%{transform:translateX(-50%) scale(0)}100%{transform:translateX(-50%) scale(1)}}
.p-inner{width:100%;height:100%; animation:bob ...}

OK.

Rider inside car: .rider{position:absolute; bottom:3px; left:50%; transform:translateX(-50%) scale(.82); width:20px; display:none} with head/body same markup. .car.has-pax .rider{display:block}. Also car interior light up: .car.has-pax .car-in{brighter background}.

Car CSS:
.car{position:absolute; left:50%; margin-left:-34px; width:68px; height:58px; top:0; will-change:transform;}
transform translateY(e.y). Inside: .car-led{height:7px; background:var(--c); border-radius:4px 4px 0 0; box-shadow:0 0 10px var(--c);} .car-no small on led? Number in center of led in dark text, font Bungee 8px.
.car-in{position:absolute; top:7px; left:4px; right:4px; bottom:4px; background:linear-gradient(180deg,#20303e,#141f29); border:2px solid #45586a; border-top:none; border-radius:0 0 4px 4px; overflow:hidden}
Interior warm when occupied: has-pax .car-in{background:linear-gradient(180deg,#3b3320? ...) } use box-shadow inset amber: inset 0 0 14px rgba(255,190,90,.28).
.car-base{position:absolute; bottom:0; height:4px; left:2px; right:2px; background:#0a0f14; border-radius:2px}

Cable: .cable{position:absolute; top:0; left:50%; width:2px; margin-left:-1px; background:linear-gradient(#3a4a58,#22303c); z:1}
cw: .cw{position:absolute; right:3px; width:10px; height:34px; background:#26333f; border:1px solid #38495a; border-radius:2px}

Rails: .shaft::before{content:''; position:absolute; inset:0; background: repeating-linear-gradient(90deg, transparent 0 18px, rgba(255,255,255,.03) 18px 20px)? Simpler 2 lines: use linear-gradient with 2 vertical lines at 30% and 70%: background-image: linear-gradient(90deg, transparent calc(30% - 1px), #1c2833 30%, transparent 30% ...). Let me just add two .rail divs per shaft.

Shaft background: linear-gradient(180deg,#0a1118,#0d1620); plus a subtle glow at floor levels? Add tick marks per floor: repeating-linear-gradient vertical: rgba(255,255,255,.04) line every 84px: background-image: repeating-linear-gradient(180deg, transparent 0 82px, rgba(120,160,190,.08) 82px 84px). Nice.

Door unit: .du{position:absolute; width:72px; height:62px; z:5} inside shaft (shaft position relative, overflow hidden? Doors inside shaft bounds, OK).
.d-frame{absolute inset 0; border:2px solid #2b3a48; border-radius:3px; background:#05090d (pit behind); overflow:hidden}
.d-panel{position:absolute; top:0; bottom:0; width:50%; background:linear-gradient(180deg,#5d7082,#3a4a59 60%,#2f3d4b); box-shadow: inset 0 0 0 1px rgba(255,255,255,.06);}
.d-panel.l{left:0; transform-origin:left} .r{right:0}
Set transform translateX(∓doors*100%) via JS.
Seam: .l has right border 1px dark.
Also small step light: .d-frame::after top glow when open? Skip.

Call lamp pulse keyframes.

Indicator: .ind{position:absolute; width:52px; height:36px; background:#0b131c; border:1px solid #243444; border-radius:8px; display:flex; align-items:center; justify-content:center; gap:5px; box-shadow:0 4px 10px rgba(0,0,0,.4)} .ind b{font-family Bungee; font-size:17px; color:var(--amber2)} .ind i{font-style:normal; font-size:11px; color:var(--c)} arrow colored by elevator color. Add top notch triangle? Fine.

Roof: .roof{position:absolute; left:BL; top:0; width:BUILDING_W; height:ROOF_H}
Parapet: .parapet{position:absolute; bottom:0; left:-6; right:-6; height:16px; background:#33424f; border-radius:4px 4px 0 0; box-shadow:0 -2px 0 rgba(255,255,255,.06)}
Sign: .sign{position:absolute; right:36px; bottom:22px; padding:6px 14px; background:#0d151d; border:2px solid #24303c; border-radius:8px; font-family:Bungee; color:var(--amber); font-size:20px; letter-spacing:2px; text-shadow:0 0 12px rgba(255,183,3,.55); box-shadow:0 0 24px rgba(255,183,3,.12)}
Text "LIFT·SIM". Flicker animation subtle: @keyframes neon flicker opacity 96%.
Antenna: .ant{position:absolute; right:14px? place at left: 320? Indicators occupy x 14..~300 (3 indicators at shaft centers 64,156,248 → ind left 38..274). Sign right. Antenna between: left: 340, bottom:16, width:3, height:52, background #45586a; beacon: ::after top -6 red circle blink.
Vent boxes: .vent{left:300; bottom:16; w:26; h:14; bg:#2c3a47; radius 3} maybe 2.

Ground: .ground{position:absolute; left:0; bottom:0; width:SCENE_W; height:GROUND_H; background:linear-gradient(180deg,#101a24,#0a121a);} top edge line highlight. Sidewalk strip lighter 6px top.
Bushes: .bush{position:absolute; bottom:GROUND_H-10; width:46; height:22; background:#173226; border-radius:12px 12px 4px 4px; box-shadow: inset 0 4px 0 rgba(120,200,150,.12)} two near building corners (left BL-30, right).
Streetlamp: .lamp{position:absolute; bottom:GROUND_H; left: BL+BUILDING_W+26; width:4; height:96; background:#2e3d4b; border-radius:2px} .lamp::before arm + light: add child .lamp-head{position:absolute; top:-6; left:-8; width:20; height:10; background:#3a4a58; border-radius:4px} .lamp-glow{position:absolute; top:2; left:-26; width:56; height:120; background:radial-gradient(ellipse at top, rgba(255,205,110,.22), transparent 70%);}

Also entrance signage on ground floor? Skip.

Stage vignette: .stage::after{content:''; position:absolute; inset:0; pointer-events:none; background:radial-gradient(120% 90% at 50% 40%, transparent 60%, rgba(0,0,0,.35))}

Panel styling: background: linear-gradient(180deg,#0f1822,#0b131c); border-right:1px solid #1d2a36; display:flex; column; gap; padding:18px 16px; overflow:auto.
h1: font-size 24; font-weight 800 (Outfit 800); letter-spacing .5. Add small elevator glyph before title: use emoji 🛗? Inconsistent rendering; draw a mini icon: span.ico with CSS elevator icon (rectangle with arrows). Simple: use "▲▼" glyph styled amber inside a box. .logo{width:34;height:34;border-radius:9px;background:linear-gradient(#ffcf3f,#ffb703); color:#231a02; display:grid;place-items:center;font-weight:900} content "⇅"? Use text "▲▼" small. Fine.

sub: 11px muted, letter-spacing 2px uppercase "ELEVATOR SIMULATOR".

sec-t: 11px, muted, letter-spacing 3px, with line before amber.

Stats b colors: wait #ffd166, ride #7cc4ff, done #63d3b6.

Erow floor badge: Bungee 12 amber.

Footer hint: 11px muted with 💡? Use "☝"? Just text with amber key span.

Toast: color #ffd166; font 12; min-height 16; opacity 0 → transition.

Now HUD update code and caching.

Also handle window resize → fit().

Initial: buildFloorBtns default selected 3; numVal 3.

Spawn button: spawn(selFloor, num) toast `已在 ${f}F 生成 ${n} 名乘客`.

Random: f=rand 1..8, n=1..3.

Reset: clear all: remove person els; queues empty; anims filtered to remove person-related? Animators referencing removed els: guard with existence check. Simplest: anims.length=0; elevators reset: y=carTopY(1), state IDLE, doors 0, tween null, assigned -1, passenger null, bsy=false, floor=1; hide riders; delivered=0; toast '已重置'. Also pending spawn timeouts: track in array, clear on reset.

Delivered counter persists until reset.

Another consideration: person walking in while doors closing? Doors stay open until BOARDING→DEPART after callback. Good.

If reset during walk, anims cleared and els removed — fine.

Also when elevator arrives TO_PICKUP but queue emptied between dispatch and arrival (possible if another elevator boarded the last person despite incoming accounting? Incoming accounting should prevent over-dispatch: demand considers incoming, so 2 elevators won't both target a floor with 1 waiting... unless queue shrinks by other means. Boarding only reduces queue by the elevator itself. So arrival always finds person if demand>0 at dispatch time and no other factor reduces. But edge: elevator A targeting f with demand 1 (1 waiting, 0 incoming) — dispatches. Elevator B idle: demand f = 1-1=0 → won't target. Good. Still keep empty check guard.

Also handle: person's dest floor equals current floor? Excluded at creation.

Now indicator arrow: '▲' when dir=1 (up), '▼' when -1, '•' when idle. Show during tween; else dim dot.

Also add direction to car? Skip.

Perf: fine.

Let me write updateHUD:
waiting = queues.reduce; riding = elevators.filter(e=>e.passenger).length;
setStat(el,val) with bump.
Elevator row: for each e: stateTxt: base = STXT[e.state]; target info: if TO_PICKUP `→${e.assigned}F`; if TO_DEST `→${e.passenger.dest}F`; floor badge: current nearest floor: nf = clamp(Math.round(FLOORS-(e.y-(ROOM_H-CAR_H))/FLOOR_H),1,8).

Cache per elevator string.

Also add dot active class when state!=='IDLE'.

Now let me write the whole file. Also include meta viewport. Title 电梯模拟器.

Let me write CSS carefully with variables.

:root{
 --bg:#070f18; --p0:#0f1822; --p1:#131f2b; --line:#22303d; --ink:#e9f1f7; --mut:#8ba0b2;
 --amber:#ffb703; --amber2:#ffd166; --coral:#ff6b5e; --teal:#43d9c9; --blue:#7cc4ff;
 --disp:'Bungee','Outfit','PingFang SC','Microsoft YaHei',sans-serif;
 --body:'Outfit','PingFang SC','Microsoft YaHei',system-ui,sans-serif;
}

Body font 14.

Grid app: height 100vh; grid-template-columns 300px 1fr.

@media (max-width:900px){ grid-template-columns:1fr; grid-template-rows:auto 1fr; panel becomes horizontal? Simplest: panel max-height 45vh overflow auto; fine.}

Let me write JS in full; double-check function ordering and typos in my head.

Also: person hover shouldn't block clicks? No clicks needed on person. pointer-events default fine.

Potential issue: tooltip on upper floors (floor 8) above person → may exceed scene top; overflow visible so OK; stage overflow hidden may clip only at stage edges — fine.

Person walking in crosses shaftblock area: person z 6 above doors — will appear in front of closed doors while approaching, then hidden (removed) just before doors close. Acceptable.

Rider walk-out at dest floor: create at door center and walk right — passes over call lamp area fine.

Let me also make sure that when elevator boards, it faces queue: door center x is shaft center; person stops at shaftCenter-12 (center of door). Then hidden.

Now let me write code.

Also double-check carTopY(8)=13, carTopY(1)=7*84+13=601. Track area height 672; car bottom at floor 1: 659, slab top at surface 7*84+71=659. ✔ Car floor aligns with walking surface.

Door top at floor f: floorTop+9, height 62 → bottom floorTop+71 = surface. ✔ Door bottom aligns with slab top.

Person bottom at layer ROOM_H=71 → surface. ✔

Indicator floor compute: nf = Math.round((672-58 - e.y)/84)+1? Let me derive: carTopY(f)=601-84*(f-1)... (8-f)*84+13 = 685-84f. f=(685-y)/84. Round: nf=Math.round((685-e.y)/84) clamp.

Check: y=601→(84)/84=1 ✔; y=13→672/84=8 ✔.

Use constant CAR_BASE_Y=carTopY(1)=601: f=(601-y)/84+1.

OK let me write.

Also door reset on floor change: on arrive set e.floor; previous door transforms: keep map of last open door element per elevator; on state change to moving, set e.doors=0 target and render applies transform to doors[e.i][e.floor] with e.doors → will animate close visually. Since render always applies e.doors to current floor doors each frame, closing animates naturally. When elevator leaves (tween), e.doors continues to 0 and panels close while car starts moving — but DEPART waits for doors<=0.02 before starting tween. ✔ After movement start, e.floor still old; render applies e.doors (0) to old doors — static closed, fine. On arrive at new floor, set e.floor=new, doors open there.

But render applies transform only to doors[e.i][e.floor]; other doors never move — ensure initial closed. ✔

Cable height = e.y (car top). cw top = (672 - CAR_H) - e.y → at y=13: 601... wait (672-58)-13=601 ✔ bottom; at y=601: 13 ✔ top.

Now let me write the file. Keep comments brief Chinese.

Additional polish: subtle interior light spill on floor when doors open: add .spill div on each floor at each shaft? Skip — features are enough.

Panel elevator row markup generated in JS into #elist.

Time to write. Let me write ~600-700 lines. Ensure no syntax errors.

Keyframes list: twinkle, drift, bob, walkbob, pop, blink (beacon), pulse (call lamp/dot), bump, neon flicker, fadeout (use animators instead for person fade — use opacity via animator).

Walk-out fade: animate opacity 1→0 over .4 after arrival, then remove el, done().

Delivered increment: on walk-out arrival (before fade).

Also stats riding includes during boarding with passenger? Passenger exists only from board-complete to alight-complete. Fine.

Let me write updateHUD with cached strings to avoid layout thrash.

Edge: spawn while queue big — x exceeds building right: cap queue length at 15: if queues[f].length>=15 toast '该楼层排队已满' skip remaining.

Now let me write final code.

function el(html){const t=document.createElement('div'); t.innerHTML=html... } mostly use createElement.

Let me just write it in one go with comments.

Also fit(): scene absolute left:50% top:50%? Vertical center: transform: translate(-50%,-50%) scale(s). Compute s=min((W-24)/SCENE_W,(H-24)/SCENE_H). Cap max at 1.25.

Scene fixed size width SCENE_W height SCENE_H, position absolute left 50% top 50%.

Sky elements in stage coords (vw-based) — independent of scale. Skyline bottom: align to scene ground approximately: ground line is at stage center + scaled SCENE_H/2... varies. Place skyline bottom: 12% and ground fog gradient at stage bottom to blend. Acceptable.

Actually better: put skyline inside scene behind building, scaled with it: 2 skyline strips absolute positioned at bottom GROUND_H, width SCENE_W, behind building (z 0), heights up to 300. Then alignment always correct. Clouds/moon/stars stay in stage. Yes — skyline inside scene.

Skyline generation inside scene: container .skyline{position:absolute; left:-40; right:-40; bottom:GROUND_H-2; height:280; display:flex; align-items:flex-end; gap:6; opacity:.9} buildings random. Behind building (z 0 vs building z auto) — set scene children z: skyline z0, ground z0, building z2, roof z2. Ground strip covers bottom.

Distant layer also lighter and taller? One layer is enough + fog gradient overlay near ground: .fog{absolute bottom GROUND_H-6 height 90 left/right 0; background:linear-gradient(180deg,transparent, rgba(10,20,30,.8))} in front of skyline, behind building? Place z1.

OK final write.

One more: Bungee font for Chinese fallback — floor buttons digits fine, stats digits fine, sign Latin fine, fno digits fine, indicator digits fine.

Let me write code:

<script>
'use strict';
/* ---------- Constants ---------- */
const FLOORS=8, FLOOR_H=84, SLAB=13, ROOM_H=FLOOR_H-SLAB;
const SHAFT_W=80, SHAFT_GAP=12, SB_PAD=10, SB_X=14;
const SB_W=SHAFT_W*3+SHAFT_GAP*2+SB_PAD*2;
const HALL_X=SB_X+SB_W+8;
const BUILDING_W=780;
const CAR_W=68, CAR_H=58;
const DOOR_W=72, DOOR_H=62;
const QUEUE_X0=HALL_X+26, SPACING=30, MAX_Q=15;
const ROOF_H=88, GROUND_H=56;
const SCENE_W=900, SCENE_H=ROOF_H+FLOORS*FLOOR_H+GROUND_H, BL=(SCENE_W-BUILDING_W)/2;
const SPEED=175, WALK=95, DOOR_SPEED=2.6;
const TRACK_H=FLOORS*FLOOR_H;
const CAR_BASE=(FLOORS-1)*FLOOR_H+(ROOM_H-CAR_H);
const E_COLORS=['#ffb703','#ff6b5e','#4cc9f0'];
const P_COLORS=['#ffd166','#ff8f6b','#63d3b6','#7cc4ff','#e78fb3','#d4e09b'];

const floorTop=f=>(FLOORS-f)*FLOOR_H;
const carTopY=f=>(FLOORS-f)*FLOOR_H+(ROOM_H-CAR_H);
const shaftLeft=i=>SB_PAD+i*(SHAFT_W+SHAFT_GAP);
const shaftCX=i=>SB_X+shaftLeft(i)+SHAFT_W/2;
const easeIO=k=>k<.5?2*k*k:1-Math.pow(-2*k+2,2)/2;
const rnd=(a,b)=>a+Math.random()*(b-a);
const ri=(a,b)=>Math.floor(rnd(a,b+1));

DOM refs...

Build functions. Then state, loop.

Note: create doorRefs[i][f]={l,r}.

Let me write it all concretely now, checking each function.

Also stars: generate in #stars.

I think I'm ready. Let me also ensure that spawn staggering uses pendingTimeouts array cleared on reset.

Let me write the final answer file. Also include short usage/test instructions after code block (运行方式与测试说明), concise. Reply language: Chinese.

Now let me carefully write out the code.

Potential subtlelety: .person transition transform .55s — when I set initial position with free trick, then remove free; subsequent layoutQueue sets new transform → animates. During walkIn I add free so no transition. After walk-in done, person removed. Walk-out person created with free from start. ✔

Pop animation: add class 'pop' at creation; .pop .p-pos animation pop .5s once. Keep class, fine.

Bob animation delay: set on .p-inner style animationDelay random -3..0s, duration rnd 2.2-3.2.

Walking: add class 'walking' → .walking .p-inner{animation:walkbob .3s linear infinite}. Overrides bob. Remove on done (person removed anyway).

Tooltip during walking: fine.

Now elevator HUD row: build once:
row: <div class="erow"><span class="edot" style="background:c;box-shadow:0 0 8px c"></span><div class="emeta"><b>1 号梯</b><span class="es">待机</span></div><span class="ef">1F</span></div>
Update .es textContent and .ef.

Stats bump: compare, on change set text and restart animation: el.classList.remove('bump'); void offsetWidth; add.

Toast timer.

Fit on resize + initial + after fonts maybe.

Also stage background sky gradient CSS.

Clouds: .cloud{position:absolute; width:180px;height:44px;border-radius:40px;background:rgba(180,210,235,.10); filter:blur(2px); animation:drift Xs linear infinite} with ::before/::after for bumps? Keep simple oval blur. 3 clouds with different top/duration/delay (negative delay to spread).

drift keyframes: from translateX(-260px) to translateX(calc(100vw + 260px))? Stage width = 100% of container; use vw, fine.

Moon: .moon{position:absolute; top:64px; right:12%; width:54px;height:54px;border-radius:50%; background:radial-gradient(circle at 35% 35%, #fff7e0, #ffe9b0 60%, #f5c86a); box-shadow:0 0 40px 10px rgba(255,230,160,.25); } with craters? Skip.

Stars twinkle: .star{position:absolute;background:#cfe6ff;border-radius:50%;opacity:.7;animation:twk var(--d) ease-in-out infinite} keyframes twk{50%{opacity:.15; transform:scale(.7)}}.

OK final write. Also add prefers-reduced-motion? Extra; skip.

Also verify: panel floor buttons: 8 buttons labeled 1..8 (with "F"?). Label "1".."8" with small F? Show "3F"? Bungee digits "3". Add title. Selected default 3.

Num stepper min 1 max 10.

Spawn click: n from numVal.

Also allow clicking on a floor in the building to select? Bonus; skip for cleanness... actually a nice micro-interaction: clicking a floor room selects it in panel. Cheap: add click handler on floor unit → select floor + toast? Maybe accidental. Add cursor pointer + hover highlight on room + click selects. Include — improves interactivity. .floor:hover .room{brighter}.

Hover highlight: .floor .room transition background filter; on hover brightness 1.15.

OK write.

Also one more check on updateElevator ordering: process tween first, then arrive may set state; then state logic same frame — fine.

Pseudocode for updateElevator(e,dt):
 // door target
 const open = (e.state==='BOARDING'||e.state==='ALIGHT');
 const dt2 = DOOR_SPEED*dt;
 if(open){e.doors=Math.min(1,e.doors+dt2)} else {e.doors=Math.max(0,e.doors-dt2)}
 // movement
 if(e.tween){ e.tween.t+=dt; const k=Math.min(1,e.tween.t/e.tween.dur); e.y=e.tween.from+(e.tween.to-e.tween.from)*easeIO(k); if(k>=1){e.tween=null; arrive(e);} }
 switch(e.state){
  case 'IDLE': tryDispatch(e); break;
  case 'BOARDING': if(e.doors>=.97&&!e.bsy){e.bsy=true; beginBoard(e);} break;
  case 'DEPART': if(e.doors<=.03&&!e.tween){ startMove(e, carTopY(e.passenger.dest)); e.state='TO_DEST'; } break;
  case 'ALIGHT': if(e.doors>=.97&&!e.bsy){e.bsy=true; beginAlight(e);} break;
  case 'CLOSE': if(e.doors<=.03){e.state='IDLE';} break;
 }

arrive(e):
 if(e.state==='TO_PICKUP'){ const f=e.assigned; if(!queues[f].length){ e.assigned=-1; e.state='IDLE'; } else { e.floor=f; e.state='BOARDING'; } }
 else if(e.state==='TO_DEST'){ e.floor=e.passenger.dest; e.state='ALIGHT'; }

Reset e.bsy=false on entering BOARDING/ALIGHT: set e.bsy=false when setting those states.

beginBoard(e): const f=e.floor; const p=queues[f].shift(); e.assigned=-1; layoutQueue(f); walkIn(p,e,()=>{ p.el.remove(); e.passenger={color:p.color,h:p.h,dest:p.dest}; setRider(e,true); e.state='DEPART'; });

beginAlight(e): const p=e.passenger; walkOut(e,p,()=>{ setRider(e,false); e.passenger=null; delivered++; e.state='CLOSE'; });

walkOut creates person at floor p.dest.

tryDispatch(e): if(e.tween) return; let best=-1,bd=1e9; for f 1..8: demand=queues[f].length - incoming(f); if(demand>0){ d=Math.abs(((685-e.y)/84+? use current continuous floor) - f) → cur=(CAR_BASE - e.y)/FLOOR_H + 1; d=Math.abs(cur-f); if(d<bd-1e-6 || (Math.abs(d-bd)<1e-6 && Math.random()<.5)){bd=d;best=f;} } if(best>0){ e.assigned=best; e.state='TO_PICKUP'; startMove(e,carTopY(best)); }

incoming(f)=elevators.filter(o=>o.assigned===f&&o.state==='TO_PICKUP').length — note e.assigned stays set during BOARDING too until boarded; but state!=='TO_PICKUP' so not counted... but should still count? Once boarding, that elevator has already taken its person from queue (shift happens in beginBoard). Between arrive and shift (doors opening ~0.4s), queue still contains that person, and another elevator might compute demand including it and dispatch — slight over-dispatch possible: 2nd elevator comes, by arrival person gone → handled by empty check (goes idle). Rare and self-healing. Alternatively keep counting assigned until boarded: incoming = o.assigned===f (any state) — cleared after shift (assigned=-1 in beginBoard). Let me do that: incoming(f)=elevators.filter(o=>o.assigned===f).length. assigned set at dispatch, cleared at beginBoard or on empty arrival. ✔ Better.

startMove(e,y){ if(Math.abs(y-e.y)<1){ // already there (e.g. same floor)
   simulate immediate arrive: handle by calling arrive after setting state? For same-floor pickup: tween dur min .5 → tiny movement; fine, skip special case. }
 e.tween={from:e.y,to:y,t:0,dur:Math.max(.45,Math.abs(y-e.y)/SPEED)}; e.dir=y<e.y?1:-1; }

render():
 elevators.forEach(e=>{
  e.carEl.style.transform=`translateY(${e.y}px)`;
  e.cableEl.style.height=e.y+'px';
  e.cwEl.style.top=((TRACK_H-CAR_H)-e.y)+'px';
  const d=doorRefs[e.i][e.floor]; const tx=e.doors*100;
  d.l.style.transform=`translateX(${-tx}%)`; d.r.style.transform=`translateX(${tx}%)`;
  // indicator
  const nf=Math.min(FLOORS,Math.max(1,Math.round((CAR_BASE-e.y)/FLOOR_H+1)));
  update ind text if changed; arrow: e.tween? (e.dir>0?'▲':'▼') : '·';
 });
 // call lamps
 for f: lamps[f].classList.toggle('on', queues[f].length>0);

HUD each frame.

walkIn(p,e,done): const x0=p.x, x1=shaftCX(e.i)-12; p.el.classList.add('free','walking'); const dur=Math.max(.55,Math.abs(x1-x0)/WALK); addAnim({dur, step:k=>{const x=x0+(x1-x0)*k; p.el.style.transform=`translate(${x}px,${p.y}px)`;}, done:()=>{done();}});

addAnim(o){o.t=0;anims.push(o)} stepAnim: o.t+=dt; k=min(1,o.t/o.dur); o.step(easeIO(k)); if k>=1 remove & o.done&&o.done(). Use ease for walking too (smooth start/stop) ✔.

walkOut(e,p,done): const layer=peopleLayers[p.dest]; const elP=makePersonEl(p.color,p.h, null); // no tooltip? give tooltip too, fun. data-tip `到达 · ${p.dest}F`? Keep same style: `已到达 ${p.dest}F`.
 const x0=shaftCX(e.i)-12, y=ROOM_H-p.h; elP.classList.add('free','walking'); elP.style.transform=translate(x0,y); layer.append(elP);
 const x1=HALL_X+30+rnd(0,90); const dur=...; addAnim({dur, step:k=>{elP.style.transform=`translate(${x0+(x1-x0)*k}px,${y}px)`}, done:()=>{ elP.classList.remove('walking'); delivered++; updateHUDNow(); addAnim({dur:.45, step:k=>{elP.style.opacity=1-k; elP.style.transform+=... keep x; }, done:()=>{elP.remove(); done();}}); }});
Hmm transform overwrite: in fade step only set opacity. ✔ done() sets passenger null etc.

makePersonEl(color,h,tip): build div structure; return.

createPerson(f): if(queues[f].length>=MAX_Q) return false; dest: let d; do d=ri(1,FLOORS) while d===f; color pick; h=ri(33,41); el=makePersonEl(color,h,`目标楼层 ${d}F`); const idx=queues[f].length; const x=QUEUE_X0+idx*SPACING, y=ROOM_H-h; el.classList.add('free'); el.style.transform=`translate(${x}px,${y}px)`; peopleLayers[f].append(el); void el.offsetWidth; el.classList.remove('free'); el.classList.add('pop'); const p={el,x,y,h,color,dest:d}; queues[f].push(p); return true;

layoutQueue(f): queues[f].forEach((p,i)=>{p.x=QUEUE_X0+i*SPACING; p.el.style.transform=`translate(${p.x}px,${p.y}px)`;});

setRider(e,on): e.carEl.classList.toggle('has-pax',on); if(on){set color/height on e.riderEl}.

Rider el inside car-in: <div class="rider"><div class="p-head"></div><div class="p-body"></div></div> with --pc.

Reset(): pending.forEach(clearTimeout); pending=[]; anims.length=0; queues.forEach((q,f)=>{q.forEach(p=>p.el.remove()); q.length=0;}); elevators.forEach(e=>{Object.assign(e,{y:carTopY(1),state:'IDLE',doors:0,tween:null,assigned:-1,passenger:null,bsy:false,floor:1,dir:0}); setRider(e,false);}); delivered=0; toast('已重置模拟');

Spawn(f,n): let c=0; for k<n: pending.push(setTimeout(()=>{ if(createPerson(f)){c++; } dispatchAll(); updateHUDNow(); }, k*130)); after: toast immediately `正在 ${f}F 生成 ${n} 名乘客…`. dispatchAll also called in loop via idle scan; fine.

dispatchAll(): elevators.forEach(e=>{if(e.state==='IDLE'&&!e.tween) tryDispatch(e);});

Loop handles idle scan anyway.

HUD: function updateHUD(){ compute waiting, riding; setStat; elevators row: st = STXT[e.state] + (e.state==='TO_PICKUP'?` · ${e.assigned}F`:e.state==='TO_DEST'?` · ${e.passenger.dest}F`:''); ef=nf+'F'; cache compare.}

STXT defined.

Now build DOM code:

function build(){
 const scene=$('#scene'); scene.style.width=SCENE_W+'px'; scene.style.height=SCENE_H+'px';
 // skyline
 const sk=div('skyline'); for(let i=0;i<24;i++){const b=div('sb'); b.style.width=ri(26,64)+'px'; b.style.height=ri(50,240)+'px'; b.style.opacity=rnd(.5,.95); if(Math.random()<.6)b.classList.add('lit'); sk.append(b);} scene.append(sk);
 fog.
 // roof
 const roof=div('roof'); roof.style.left=BL+'px'; roof.style.top='0'; roof.style.width=BUILDING_W+'px'; roof.style.height=ROOF_H+'px';
 parapet, sign (text LIFT·SIM), vents, antenna+beacon; indicators: for each elevator i: ind positioned at left=shaftCX(i)-26 (building coords) since roof is aligned with building (left BL) → inside roof coords = shaftCX(i)-26. top=ROOF_H-52.
 scene.append(roof).
 // building
 const bld=div('building'); bld.style.left=BL+'px'; bld.style.top=ROOF_H+'px'; bld.style.width=BUILDING_W+'px'; bld.style.height=TRACK_H+'px';
 // floor units
 for f=1..FLOORS: fu=div('floor'); top floorTop; room, slab, fno text f, win (random opacity), call lamp (save lamps[f]), click to select floor.
 // shaftblock
 sb=div('shaftblock'); for each i: shaft=div('shaft'); left shaftLeft(i), width SHAFT_W, top 0 bottom 0; rails r1 r2; cable; cw; car (build, save); doors layer: for each f: du positioned at left (SHAFT_W-DOOR_W)/2, top floorTop(f)+(ROOM_H-DOOR_H); frame with l/r panels; save doorRefs.
 bld.append(sb);
 // people layers
 for each f: pl=div('people-layer'); top floorTop(f); height ROOM_H; bld.append(pl); peopleLayers[f]=pl.
 scene.append(bld).
 // ground and decor
 ground div bottom; bushes; streetlamp.
}

Floor unit z-index vs shaftblock: floor first, then shaftblock → shaftblock naturally on top. People layers after → on top. Explicit z anyway.

Room left position: left: (SB_X+SB_W) px = 298, right:10.

Call lamp left: SB_X+SB_W+6=304? overlaps room start 298; place at left:306 top:(ROOM_H-22)/2.

Actually QUEUE_X0=332; call lamp at 306 fine.

fno: right:14 top:6.

win: right:70 top:14 w:46 h:30.

Max queue x: 332+14*30=752 <780 ✔ (person width 24 → 776).

Indicators overlap with sign? Sign right:36 width ~150 → occupies x 594..744 in building coords. Indicators max x 274+52=326. Antenna at 350-360, vent 300. OK.

Streetlamp left: BL+BUILDING_W+30=870 within SCENE_W 900 ✔.

Now CSS for .sb (skyline building): background:#0b1a28; border-radius:2px 2px 0 0; position relative; .sb.lit::after{content:'';position:absolute;inset:6px 4px; background-image:repeating-linear-gradient(0deg, transparent 0 7px, rgba(255,190,110,.16) 7px 9px), repeating-linear-gradient(90deg, transparent 0 6px, rgba(255,190,110,.10) 6px 9px)? messy; simpler: background-image: radial-gradient(rgba(255,200,120,.35) 1px, transparent 1.5px); background-size: 9px 12px; opacity .5. ok.

Skyline container: absolute left:-30 right:-30 bottom:GROUND_H-4; display flex; gap:6px; align-items:flex-end; z-index:0; filter: brightness(.9).

Add distant mountain too? No.

Ground: absolute left:0 bottom:0 width 100% height GROUND_H; z 1; background linear-gradient(180deg,#15222e,#0a1219); border-top:2px solid #24384a.

Fog: bottom:GROUND_H; height:110; z:0? in front of skyline, behind building: z 1, building z 2. Ground z 1. Fine.

Bushes z 3 (in front of building base). Streetlamp z 3.

Building z 2, roof z 2.

Now panel HTML as sketched. JS wiring for buttons.

fit(): const st=$('#stage').getBoundingClientRect(); s=Math.min((st.width-28)/SCENE_W,(st.height-28)/SCENE_H,1.18); scene.style.transform=`translate(-50%,-50%) scale(${s})`;

Scene CSS: position:absolute; left:50%; top:50%; transform-origin:center; will-change:transform.

Init: build(); buildPanel(); fit(); requestAnimationFrame(loop); demo spawn after 400ms: setTimeout spawn(2,2) etc.

Also add resize listener.

Double-check rider markup CSS: .rider{position:absolute;bottom:2px;left:50%;transform:translateX(-50%) scale(.8);width:18px;height:34px;display:none} .car.has-pax .rider{display:block} .rider .p-head{width:10px;height:10px;...} reuse classes p-head p-body with fixed height: rider height 34 fixed.

Car-in overflow hidden clips rider nicely.

Also add subtle car glow shadow beneath: .car::after{content:'';position:absolute;bottom:-3px;left:8px;right:8px;height:6px;background:radial-gradient(ellipse,rgba(0,0,0,.5),transparent 70%)}

Also add small direction LED on car? Skip.

Now STXT and colors for elevator rows: dot uses --c.

I think I'm ready to write final. Also note: font for .ind b is Bungee amber; arrow i color per elevator via style.

Also add "运行日志"? No.

Edge: Bungee loading may delay; fine.

Let me write the answer: brief intro line, code block, then 运行方式 and 测试说明 bullets. Keep surrounding text concise.

Let me now write out the whole file text carefully, checking each function.

Also: `numVal` display update; minus/plus clamp 1..10.

Floor button container id floorBtns; save selectedFloor variable; render selected class.

Floor click in scene: select same.

Toast function with timer handle.

Write CSS for .toast{height:18px;font-size:12px;color:var(--amber2);opacity:0;transition:.3s;transform:translateY(-2px)} .toast.show{opacity:1;transform:none}

Ensure panel sections compact.

Also add subtle noise? Skip.

Final check on animation loop start and dt.

Now let me write it all. Count z-indexes: people layer z 6 but tooltip above everything: ::after z 99.

Also .floor hover effect: .floor{position:absolute;left:0;width:100%} .floor .room{transition:filter .25s} .floor:hover .room{filter:brightness(1.25)} cursor:pointer.

Room background: linear-gradient(180deg, rgba(255,183,77,.16), rgba(255,183,77,.05) 60%, rgba(0,0,0,.12)), plus back wall base color #1b2836? Let me do: background: linear-gradient(180deg,#233448 0%, #1d2b3a 100%); and add .room::after warm glow overlay: radial-gradient(60% 80% at 70% 20%, rgba(255,190,100,.20), transparent 70%). Randomize glow opacity per floor via inline --glow.

Room ceiling shadow: box-shadow inset 0 6px 10px rgba(0,0,0,.35).

Slab: background:linear-gradient(180deg,#4a5c6e,#33424f 40%,#2a3743); border-top:2px solid #5d7082; no radius.

Building background (facade behind everything): #26333f with side shading: .building{background:linear-gradient(90deg,#202c38,#283644 8%,#283644 92%,#1d2935); border-radius:4px 4px 0 0? top is roof; radius 0. box-shadow: 0 30px 60px rgba(0,0,0,.45), inset 0 0 0 1px rgba(255,255,255,.03);}

Also left/right pilasters: .building::before/::after width 6 darker full height.

Shaftblock: background:#0b1219; border:1px solid #1e2c39; border-radius:6px; box-shadow: inset 0 0 24px rgba(0,0,0,.6); overflow:hidden.

Shaft: position:absolute; top:6; bottom:6; background:linear-gradient(180deg,#0a1017,#0d151d); plus tick marks via repeating gradient; border-inline:1px solid #142029.

Car left:50% margin -34.

Cable z below car: cable z1, cw z1, car z2, du (doors) z3.

Doors frame border color #2e3f4f; panels gradient steel with slight vertical brush lines: background-image: linear-gradient(90deg, rgba(255,255,255,.05), transparent 30%, ...). Keep simple.

Also small transom light above each door: .du::before{content:'';position:absolute;top:-5px;left:50%;transform:translateX(-50%);width:14px;height:3px;border-radius:2px;background:#1a2833} skip.

OK final write. Also don't forget person head skin color: #ffd9b3 with inset shading.

Rider head same.

Now let me write the answer. Also mention external fonts (optional, degrades gracefully offline).

For test instructions: list scenarios: basic spawn, one-person-per-car constraint, queue order, tooltip, hover, reset/random.

Now generate the file. Write in one continuous block, being meticulous.

Potential bug watch:
- Template literals in style strings fine.
- doorRefs indexing [i][f]: create array.
- lamps[1..8].
- peopleLayers[1..8].
- queues[1..8] (index 0 unused).
- elevators array with carEl etc.

In render, ind refs: e.indF (b), e.indA.

HUD row refs: e.rowEl, e.stEl, e.efEl, e.dotEl.

Compose.

Also note: initial e.floor=1, doors closed.

Demo spawn: setTimeout(()=>{spawn(2,2);spawn(5,1);},300); setTimeout(()=>spawn(7,2),900);

Ensure spawn toast doesn't spam on demo: pass silent flag.

spawn(f,n,silent).

OK, writing.

Also: `.person` width 24 but inner 20 centered; tooltip center OK.

Also add cursor:default for person; hover pointer? Use cursor:help? default fine.

Recheck keyframes names unique.

Also panel: overflow-y:auto; thin scrollbar styling webkit.

Media query <960px: app grid rows; panel order 2? Better keep panel top with horizontal wrap: .panel{flex-direction:row; flex-wrap:wrap; align-items:flex-start} sections width auto... messy; simpler: keep column, panel max-height:46vh; stage rest. Acceptable.

Let me write final HTML.

Also during DEPART→TO_DEST startMove uses e.passenger.dest — passenger set. ✔

beginAlight captures p before callback clears.

In walkOut done callback: setRider false, passenger null, state CLOSE. Delivered incremented inside walkOut on arrival. Fine.

updateHUDNow just calls updateHUD (called each frame anyway).

Stats riding computed from elevators.

Now writing.

After writing JS, quick mental syntax scan.

Also add `user-select:none` on scene/panel buttons.

Deliver.