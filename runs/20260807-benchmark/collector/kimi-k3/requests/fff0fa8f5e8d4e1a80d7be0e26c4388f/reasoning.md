The user wants an elevator simulator in a single HTML file with:
- Multiple floors where users can spawn a certain number of people
- 3 elevators, each can carry only 1 person at a time
- Passengers queue up; when one boards, others wait; next elevator picks up next person
- Each person has a random target floor
- Hovering on a person shows a tooltip with their target floor
- Smooth animations, good visuals, clean
- HTML, CSS, JavaScript all in one file

Let me design this:

Building layout: say 10 floors. Three elevator shafts side by side. People wait at floors (in a waiting area to the right of the elevators, or left). Controls at top: select floor, number of people, spawn button.

Elevator logic:
- Each elevator has current floor, target, state (idle, moving, loading, unloading)
- Each elevator capacity = 1 person
- People queue at their spawn floor
- When elevator arrives at a floor with waiting people, the first person in queue boards (with animation of person moving into elevator)
- Elevator then goes to that person's target floor, person exits (and maybe walks off / fades out)
- Elevator becomes idle again and scheduler assigns next task

Scheduler: assign idle elevators to floors with waiting passengers. Simple approach: each tick, for each idle elevator, find nearest floor with waiting people, go there. Avoid two elevators going to the same pickup — track which floors are "claimed" or claim specific persons.

Actually simpler: maintain a global queue of pickup requests (floors with waiting people). When an elevator is idle, it takes the oldest unassigned request. But people queue per floor — one elevator picks one person, then the floor may still have people, so it remains a request.

Let me design: `waiting` = Map of floor -> array of person objects. When elevator becomes idle, find nearest floor with waiting people that isn't already targeted by another elevator for pickup... Or simply assign: nearest floor with waiting count > assigned pickups for that floor.

Simple robust approach:
- For each elevator when idle: compute candidate floors where waitingCount(floor) > claimedCount(floor). Claim one (increment claimedCount), set elevator state to "pickup", target floor.
- When elevator arrives at pickup floor, decrement claim, take first person from queue, person animates into elevator, doors close, elevator moves to person's target floor, doors open, person exits (walks away and fades), elevator idle.

Person visuals: simple person shape made with divs (head circle + body rounded rect) or emoji 🧍. Using emoji is simplest but walking animation into elevator could be a translate transition. I'll draw a CSS person (head + body) with a random color. That looks nicer.

Tooltip: on hover show "目标楼层: X" via a tooltip div positioned near the person, or CSS ::after with data attribute. CSS tooltip with attr(data-target) is simplest, but we need Chinese text + number — data attribute works: `data-tooltip="目标: 5F"`. But when person is inside elevator, tooltip should maybe still work — fine.

Animations:
- Elevator moves via CSS transition on transform/bottom, duration proportional to floors traveled.
- Person walking into elevator: person element position absolute in the scene; animate left/top transitions. Could get complex. Simpler: person element positioned in floor waiting area; when boarding, transition its left to the elevator shaft position; then attach to elevator (change parent? reparenting breaks transitions). Alternative: keep person absolutely positioned in the main scene container with coordinates; elevator also has coordinates; person "rides" by updating person's bottom with same transition as elevator. That works: person is absolutely positioned in scene; to board, set left to elevator's x, then to ride set bottom to target floor y with same duration as elevator, then when arriving, animate left to exit area and fade out.

That's clean. Let me set up coordinates:
- Scene container fixed height: e.g., 720px for 10 floors, each floor 72px. Floor i (1-based) has bottom = (i-1)*72 px.
- Elevator shafts at fixed x positions. Elevator car element inside shaft container, absolutely positioned, bottom animated with transition.
- People queue in waiting area to the right of shafts (or left). Queue position: left offset based on index in queue.

Layout horizontally: [floor labels] [elevator 1] [elevator 2] [elevator 3] [waiting area wide]. Waiting people line up starting right after shaft 3, spaced ~34px each. Might overflow with many people; allow horizontal wrap? Could do two rows or shrink spacing. Keep spacing 30px and allow the waiting area to be wide (e.g., 500px). If overflow, people overlap slightly — acceptable. Or queue positions: index % 12 per row, second row behind (slightly higher? floors are one level). Maybe offset y a bit for second row (people standing behind, draw smaller/higher). Keep it simple: single row, spacing 28px, allow overlap if too many — set a max spawn like 8 per click, and total per floor cap maybe 15. Actually overlapping is fine visually if z-index handles it; spacing 26px with person width 22px is fine.

Elevator doors: nice touch — car with two door halves that slide open/close via CSS. I'll add doors that open when loading/unloading.

Timing/steps for elevator cycle:
1. idle -> assigned pickup floor: close doors (if open), move (transition bottom, duration = 350ms per floor, ease-in-out), arrive -> open doors.
2. Person walks in: animate person left to elevator center over 400ms, then person opacity? No — person steps in, doors close with person inside (person z-index below doors so doors cover). Actually person inside elevator: person element positioned over elevator car, doors slide over. Then move both elevator and person bottom to target floor. Person bottom transition same duration — need to sync: set person's bottom with same duration and easing at same time; they animate together.
3. Arrive: doors open, person walks out to the right/left and fades, then remove person element. Doors close, elevator idle, scheduler ticks.

Scheduler loop: run every 200ms check: for each idle elevator, find best floor. Also when new people spawn, trigger scheduler.

Claims: claimedPickups per floor count. When elevator assigned to floor F, claims[F]++. Effective waiting = queue.length - claims... but careful: claim represents "an elevator is coming to pick one person". When elevator arrives and takes person, claims[F]--, queue shifts.

Target floor assignment: random 1..10 excluding spawn floor.

Direction indication: elevator display shows current floor number, arrows ▲▼. Add small floor indicator above each shaft or on the car.

Stats: people waiting, trips completed, people delivered. Nice small panel.

Controls: floor selector (dropdown 1-10), count input (1-8), "生成乘客" button. Maybe "随机生成" button spawning random counts at random floors. Keep simple: floor select, count, add button. Maybe also a small "随机" button.

Building visuals: floors with background lines, floor labels on left (10F..1F), elevator shafts with darker background, cars with gradient, doors. People: head (circle) + body (rounded), random pastel color, small face? Keep simple.

Tooltip: person hover -> CSS ::after content: attr(data-tip), styled dark bubble above person. Person needs position absolute; tooltip position absolute above. z-index high. Works.

Ground: add a ground line at bottom, maybe building title.

Let me now think about coordinates carefully.

Scene: 
- FLOORS = 10, FLOOR_H = 68px. Scene height = 10*68 = 680px.
- Left labels column width 48px.
- Shaft width 70px each, gap 10px. Shafts start at x = 56. Shaft i x = 56 + i*80. Three shafts: 56, 136, 216; width 70 → shafts end at 286.
- Waiting area starts x = 300. Person width ~24, spacing 30. Person queue base x = 310 + index*30.
- Scene width: 310 + 15*30 = 760+. Let's set scene width 780px.

Floor Y: floor f (1-based) bottom = (f-1)*FLOOR_H. Person stands on floor: person bottom = floorBottom + 4. Elevator car height = FLOOR_H - 8 = 60, car bottom = floorBottom + 4.

Elevator car: position absolute within scene, left = shaft x + 4, width 62, height 60.

Person element: absolute in scene, left and bottom set with transitions (left 0.45s ease, bottom set dynamically per ride). Person height ~38 (head 10 + body 26), width 20.

Boarding: target left = shaft center - personWidth/2 = shaftX + 35 - 10 = shaftX + 25.

Riding: person's bottom must match elevator's. Elevator bottom = (f-1)*68 + 4. Person bottom while standing on floor f = (f-1)*68 + 4. Same formula — so when riding, set person bottom = targetFloorBottom + 4 with transition duration = elevator move duration, easing same. Set them in the same frame to sync. Use transition style strings: `bottom ${dur}ms cubic-bezier(.45,0,.3,1)`.

Exit: person left -> exitX (shaftX + 35 + 40?) animate left 500ms and opacity fade, then remove. Exit direction: walk right toward waiting area? People exit... walking right is fine, fade out.

Doors: car contains two door divs (left half, right half). Closed: each width 50%. Open: width 0 with transition 300ms. Behind doors, inside of car darker. Person z-index: person element is in scene, car z-index say 5, doors z-index 7 within car (car creates stacking context). Person z-index 6 (above car background below doors)? Person is sibling of car in scene; stacking: car z-index 5, person z-index 6 → person appears in front of car but doors are inside car's stacking context, so doors can't cover person. Hmm. To have doors close over the person, doors must be in a higher stacking context. Alternative: make doors siblings in scene overlay? Complicated.

Simpler: when person boards and doors close, fade person slightly or just leave visible — but then doors sliding closed would be behind the person, looks odd.

Alternative approach: person opacity to 0 quickly when doors close (as if hidden inside), then teleport visually via bottom animation while invisible, fade back at arrival, walk out. That works and is simple: person walks in (left animates), then doors close, person opacity 0 (fast 150ms) — slightly before/while doors close. During ride person invisible. On arrival doors open, person opacity 1, walks out.

Actually nicer: keep person visible through glass doors? Let's just do: doors have slight transparency? Overkill. Go with fade approach — clean and looks intentional if timed: person walks in → doors close (300ms) → at 200ms into closing, person fades 150ms. Fine.

Alternatively put person inside car element when riding: reparent person into car (position absolute inside car). Reparenting after the walk-in transition completes: compute person's position relative to car, set as left/top within car, then car moves with person inside, doors (inside car, higher z) slide over person. On arrival, doors open, reparent back to scene or animate person within car then move out... reparenting out would jump unless computed. Could keep person inside car and animate left within car to outside (left beyond car width) with overflow visible on car? Car overflow hidden for doors? Doors are inside car; overflow hidden would clip exiting person. Set car overflow visible but doors are absolutely positioned halves covering the opening — person walking out to left=-40 would render above car background but below doors if z-index ordering. Doors when open have width 0, so no visual conflict. This is actually clean:

- Person element created in scene for queue.
- On boarding: animate left in scene to car center (must match car's current screen position — car at pickup floor, person same floor, so only left changes). When done, reparent: person.style.left within car = (carWidth - personWidth)/2 = 21px, bottom = 0 (relative to car interior). Set person bottom relative: car interior floor is car bottom; person bottom: 0 within car works if car height aligns. Append person to car element. z-index person 1, doors z-index 2 (both inside car). Doors close over person. 
- Ride: only car moves; person rides automatically. 
- Arrive: doors open, then animate person left from 21 to 70 (outside right of car) over 450ms with car overflow visible, then fade. Person stays inside car DOM but visually outside; fine, then remove element.
- Car overflow: visible; but doors closed + car interior: car background covers person? Person z-index 1 above car background (car background is the element's background — children always render above element background). Good. Doors above person. 

Edge: while person walking in scene toward car, car z-index 5, person z-index 6 so person walks "in front" of open doors — acceptable since doors open (width 0). Car frame: I'll style car as a frame with interior opening; doors cover opening. Person entering overlaps interior — looks like stepping in. Good.

Queue rearrangement: when first person leaves queue, others shift left: update each person's left with transition — nice shuffle-forward animation. Implement renderQueue(floor): for each person in queue array, set style.left = queueX(index). Transitions handle animation.

Elevator state machine per elevator:
states: 'idle', 'toPickup', 'boarding', 'riding', 'exiting'
Use async/await with sleep helper for clarity. Each elevator runs a loop? Simpler: scheduler function `assignElevator(elev)` called when elevator idle and on spawn events; elevator async method runCycle(floor) performs steps and at end sets idle and calls scheduler again.

Concurrency: claims map floor->count. assign: among floors where queue.length > claims[f], choose nearest to elevator current floor; tie → lowest floor. If none, stay idle.

Person object: { id, spawnFloor, targetFloor, el, state }.

Spawn: create n people at floor f with random targets, push to waiting[f], render queue, kick scheduler (call assign for all idle elevators).

Move duration: base 300ms + 260ms per floor? dist * 280 + 200, cap maybe. Ease cubic-bezier(0.45, 0.05, 0.35, 1) or ease-in-out.

Door open/close: 280ms. Boarding walk: 420ms. Exit walk: 500ms. Between-floor indicator update during ride? Update elevator's displayed floor gradually — optional: setInterval during move to compute nearest floor from elapsed time; or simpler: show target with arrow. I'll show current floor number updated at arrival, plus arrow during movement. Could do a smooth floor counter using requestAnimationFrame reading car's computed bottom... simpler: approximate with timed updates. Skip fancy: display shows target floor + arrow while moving, current floor when stopped. Actually let me do a nicer touch: during movement, update the number as it passes floors using timeouts per floor. For move from a to b over duration d, schedule updates at i*step. Minor, doable: in moveTo, for k=1..|b-a|, setTimeout at (d*k/|b-a|) to update display = a + dir*k. Good.

Stats: delivered count, waiting count, active trips. Update on changes.

Now controls UI: top bar with title, floor select (1-10 buttons or select), count select (1-8), button "生成乘客", maybe "随机生成" spawns 1-6 people at a random floor. Also stat chips.

Visual style: dark theme? Building cross-section with soft gradient background, floor slabs as horizontal lines, floor labels, elevator shafts subtle darker columns, cars with gradient and glowing indicator. People colorful. Tooltip dark pill with arrow.

Let me also handle: hover tooltip on person inside elevator — fine, still works via CSS.

Tooltip CSS:
.person::after { content: attr(data-tip); position:absolute; bottom:110%; left:50%; transform:translateX(-50%) translateY(4px); opacity:0; pointer-events:none; transition: .18s; background:#111; color:#fff; padding:3px 8px; border-radius:6px; font-size:11px; white-space:nowrap; z-index:50; }
.person:hover::after { opacity:1; transform:translateX(-50%) translateY(0); }

Person z-index on hover raise.

Person markup: div.person > div.head + div.body. Colors: random hue: `background: hsl(h,70%,60%)`. Head slightly lighter.

Person width 20, height: body 22 + head 10 → total ~34. Set person bottom alignment: person bottom offset = floorBottom + 2.

When reparented into car: car height 60; person bottom: 2px relative car. Car inner width 62; person left 21.

Elevator car structure:
<div class="car">
  <div class="car-top"><span class="dir"></span><span class="cfloor">1</span></div>? 
Hmm car height 60, indicator above shaft instead: each shaft has a header display above showing floor + arrow. That's easier and looks like real elevator hall indicator. Put header above each shaft at top of scene? Scene is tall (680px) — header above building, aligned per shaft. Put indicators in a row above the scene aligned to shaft x positions. Also small floor display on the car itself (tiny). I'll do: shaft header panel above scene (positioned absolutely over each shaft), plus tiny floor number on car top.

Simplify: car has a small display strip at its top showing current floor number and arrow. Height 14px within car. Doors occupy rest. Person inside below display. Fine.

Doors: .door.left {left:0} .door.right{right:0}, both top:14px bottom:0 width:50% → closed. Open: width:0. Transition width .3s. Wait — with width transition and right door anchored right, opening slides them outward. Good.

Car interior: background dark (#2a2f3a), doors metallic gradient.

Car frame: border, border-radius, shadow.

Scene floors: for each floor, a div.floor with bottom at f*FLOOR_H? Actually floor slab lines: horizontal line at each floor bottom. Use repeating-linear-gradient background on scene for lines, plus floor labels column. Waiting area per floor could have subtle band shading alternating. Use background: repeating-linear-gradient(to top, #f5f7fa 0 67px, #d8dee9 67px 68px) — lines. Labels absolutely positioned.

Building container: position relative, width 820, height 680, background gradient, border-radius, overflow hidden? If overflow hidden, exiting people walking right fade before edge — ok keep overflow hidden? Person walking out at shaft3 x~250+70=320, scene 820 — fine inside. Tooltip near top floor might clip with overflow hidden. Tooltip bottom:110% of person who stands ~36px above floor; top floor person top is at y ~ 680-4-36=640 from bottom → 40px from top; tooltip needs ~26px — fits. Keep overflow hidden for clean rounded corners? Shafts extend full height; labels inside. I'll keep overflow visible but add border radius carefully... simpler: overflow hidden, tooltip fits as computed. Person queue overflow: waiting area width 820-300=520 → up to 17 people at 30px spacing. Cap total waiting per floor at 16 to be safe; spawn count max 8 per click but multiple clicks could exceed — cap: if queue length + claims >= 16, ignore extra. Fine.

Elevator moveTo implementation:

function moveCar(elev, toFloor) returns Promise:
  const from = elev.floor; const dist = Math.abs(to-from); if dist==0 resolve immediately (still update display).
  const dur = 240 + dist*240;
  set car.style.transition = `bottom ${dur}ms cubic-bezier(.5,0,.25,1)`;
  car.style.bottom = y(toFloor)+'px';
  elev.state moving; update arrow dir; schedule floor number updates.
  await sleep(dur); elev.floor = toFloor; update display.

Floors y: (f-1)*FLOOR_H + 4.

Doors: openDoors/closeDoors toggle class, await 300ms.

Full cycle for elevator e with assigned floor F:
```
async function runElevator(elev){
  while(true){
    const floor = findJob(elev);
    if(floor==null){ elev.state='idle'; await waitForJob(elev); continue; }
    ...
  }
}
```
Simpler event-driven: function tryAssign(elev): pick floor, claim, run async steps, at end tryAssign again. And spawn triggers tryAssignAll idle.

Need to guard: elev.state==='idle' check before assign; set state='busy' synchronously when claiming to prevent double-assign within same tick.

findJob(elev): candidates floors 1..10 with waiting[f].length > claims[f]. Choose min |f - elev.floor|, tie prefer lower f? Also maybe prefer the floor where elevator already is. distance handles that.

Cycle steps:
```
claims[f]++
elev.state='busy'
await closeDoors(elev) // if open; initially open? Start doors open at floor 1? Start closed.
if(elev.floor!==f) await moveCar(elev,f)
await openDoors(elev)
claims[f]--
const p = waiting[f].shift()
renderQueue(f) // shift others
// person walks in
await personEnter(elev,p) // animate left in scene to car center, then reparent into car
await closeDoors(elev)
update hall indicator direction
await moveCar(elev,p.target)
await openDoors(elev)
await personExit(elev,p) // animate left out + fade, remove
delivered++
elev.state='idle'
tryAssign(elev) // next job
```

Note doors: after exiting, close doors before next move — handled at start of cycle (closeDoors). If idle with open doors, fine to leave open.

personEnter: p.el.style.left = carSceneLeft + 'px'; await sleep(450); then reparent: car.appendChild(p.el); p.el.style.left='21px'; p.el.style.bottom='2px'; p.el.style.transition='left .45s ease, opacity .3s'; (reset bottom transition to none so it follows car). Actually when reparenting, bottom becomes relative to car; set bottom 2px. Since parent changes, position jumps — we set correct relative values so no visual jump: car scene-left = shaftX+4; person width 20; center left = shaftX+4+21. Inside car left 21 matches. Person bottom in scene was floorBottom+2; car bottom = floorBottom+4; relative bottom = -2... hmm: person bottom (scene) = floorBottom+2 → relative to car = 2-4 = -2. Set bottom:-2px inside car. Or just make person standing bottom = floorBottom+4 consistently (people stand at +4 like car). Fine: person scene bottom = (f-1)*FH+4. Then inside car bottom:0? Person relative bottom = sceneBottom - carBottom = 0. Set bottom:0 inside car. Person height 36; car interior below display (top 14) → person top at 36 < 60-14=46 ok.

Transitions: person default transition: left .45s ease, bottom .25s? During queue shuffle only left changes. bottom only set at spawn. So transition: left .45s ease, opacity .3s ease. Good — no bottom transitions needed anywhere since riding handled by reparenting. 

personExit: p.el.style.left = '86px' (car width 62, exit to right outside), opacity fade at end: set transition left .5s ease, opacity .4s ease .25s? Then opacity 0 after slight delay: setTimeout(()=>p.el.style.opacity=0, 250). await sleep(650); remove element. Car overflow visible allows seeing person walk out. But person is child of car at z-index 1, doors z 2 — open doors width 0, fine. Person walking out passes over shaft edge / waiting area — visually ok since car overflow visible. But car has overflow visible always — doors are contained anyway.

Also person walking out should walk to the right toward waiting area then fade — good.

Hall indicators above shafts: element positioned at left = shaftX, width 70, top of page above building. Show floor number + arrow ▲/▼/–. Color arrow green up, orange down.

Also highlight floor when elevator arrives? Skip.

Floor select for spawn: buttons 1..10 might be many; use <select>. Count: number input 1-8. Buttons: 生成, 随机. Stats chips: 等待中 X · 已送达 Y · 电梯 3.

Random button: spawn 1-5 people at random floor.

Add subtle CSS: building shadow, title font, gradient background page.

Door timing detail: when elevator arrives at pickup and doors open, person enters — need person still in queue visually until shift; shift at that moment triggers others to move up with 450ms transition — nice.

Edge: two elevators same floor simultaneously — allowed (different claims). Cars overlap visually? They're in separate shafts, no overlap.

Edge: spawn on floor where elevator currently boarding — queue handles.

Scheduler also should trigger when? tryAssign called: after cycle ends, after spawn, initially. If no job, elevator idle; when spawn occurs, call tryAssign for idle ones. Good.

Also initial state: elevators at floor 1, doors closed.

Let me write floor label column: absolutely positioned labels at each floor: left 8px, bottom floorBottom+24.

Floor separator lines: background gradient on building; but shafts overlay with their own bg — shafts should also show lines? Real shafts hide floors. Shaft bg: darker translucent column full height with inner side borders. Cars slide within. Lines behind shafts not needed — set shaft bg opaque.

Building structure HTML:

```
<div class="stage">
  <div class="headers"> hall indicators x3 positioned by JS </div>
  <div class="building" id="building">
     floor lines via CSS bg
     <div class="labels">...generated</div>
     <div class="shaft" x3 generated, each contains .car > .display, .door.left, .door.right
     people appended here
  </div>
  <div class="ground"></div>?
</div>
```

Shafts: position absolute, top:0 bottom:0, left set by JS, width 70, background linear-gradient(#39404d,#2c333f), with inset shadow. Cars absolute within building (not shaft) for z simplicity? If car inside shaft, shaft overflow visible needed — fine. Put car inside shaft: left:4px. People appended to building for queue; boarding left coordinate = shaftX + 4 + 21.

Compute constants in JS: FLOOR_H=68, FLOORS=10, SHAFT_W=70, SHAFT_GAP=10, SHAFT_X0=58, CAR_W=62, PERSON_W=20, QUEUE_X0=58+3*70+3*10+16 = 58+210+30+16=314. Wait gap: shafts at x = X0 + i*(W+GAP): 58, 138, 218. End = 218+70=288. QUEUE_X0=306. Building width = 306 + 16*30 + 20 = 806 → set 820.

Person queue x = QUEUE_X0 + idx*30.

Person height: head 10px (width 10, left 5), body 20px h, width 20 → total 32? Let's do person height 34: body 22, head 11 + gap 1. bottom aligned.

Person CSS:
.person{position:absolute;width:20px;height:34px;z-index:6;transition:left .45s cubic-bezier(.4,0,.2,1),opacity .35s ease;cursor:pointer}
.head{width:11px;height:11px;border-radius:50%;margin:0 auto;background:var(--skin, hsl)...}
Actually color: body colored, head lighter shade: use CSS var --c for hue: head background hsl(var(--h),65%,72%), body hsl(var(--h),65%,55%).
.body{width:16px;height:22px;margin:1px auto 0;border-radius:7px 7px 5px 5px;position relative} add little legs? keep blob — cute enough. Maybe add simple legs via ::before? Keep blob.

Tooltip data-tip set: `→ ${target}F` or `目标：${target} 层`.

Display on car: .panel{position:absolute;top:3px;left:50%;transform:translateX(-50%);background:#10151d;color:#7CFC9A;font-family:monospace;font-size:11px;padding:1px 6px;border-radius:3px;z-index:3}

Doors top offset: doors top 16px bottom 0. Car interior behind: background #232a35.

Car: width 62 height 60 bottom set by JS, left 4 within shaft. z-index 5.

Doors styling: background linear-gradient(90deg,#9aa5b1,#c7d0da 50%, #8b95a1) with center line for left/right. .door.left{border-right:1px solid #5c6672}.

Now the floor number updates during move: schedule with setTimeout per passed floor:

```
function moveCar(elev, to){
  return new Promise(res=>{
    const from=elev.floor, dist=Math.abs(to-from);
    elev.arrow = to>from?'▲':(to<from?'▼':'');
    if(dist===0){updatePanel(elev);res();return;}
    const dur=250+dist*250;
    elev.car.style.transition=`bottom ${dur}ms cubic-bezier(.55,.06,.35,1)`;
    elev.car.style.bottom=carY(to)+'px';
    const dir=Math.sign(to-from);
    for(let k=1;k<=dist;k++){
      setTimeout(()=>{elev.shown=from+dir*k;updatePanel(elev);}, dur*k/dist);
    }
    setTimeout(()=>{elev.floor=to;res();},dur+20);
  });
}
```

updatePanel sets textContent `${arrow} ${shown}` or floor.

Sleep helper: const sleep=ms=>new Promise(r=>setTimeout(r,ms));

Doors:
```
function doors(elev,open){elev.car.classList.toggle('open',open);return sleep(320);}
```
CSS .car.open .door.left{width:0} etc. Default width 50%.

Person enter:
```
async function personEnter(elev,p){
  p.el.style.zIndex=8; // above doors while walking in? doors open anyway
  p.el.style.left=(elev.enterX)+'px';
  await sleep(460);
  elev.car.appendChild(p.el);
  p.el.style.bottom='0px';
  p.el.style.left='21px';
}
```
Note: after reparent, transition left .45 still applies but value set to equivalent — no movement. z-index within car stacking context now; set p.el z-index 1; doors z-index 2 so doors close over. But walking-in phase person z 8 in building context — above car(5) fine.

Person exit:
```
async function personExit(elev,p){
  p.el.style.left='92px';
  setTimeout(()=>{p.el.style.opacity='0';},260);
  await sleep(700);
  p.el.remove();
}
```
Car overflow visible needed: .car{overflow:visible}. But doors overflow? doors are positioned within car bounds, fine. Panel fine.

Wait: person z-index 1 inside car; when walking out to left 92, the building content behind (shaft bg) is behind car? Car z-index 5 within shaft; shaft z-index? Shafts z-index 3, people z 6 in building. Person inside car at stacking context of car (z5) — walking out person renders below queue people but above shafts — fine.

Boarding when elevator is at person's floor — always true (pickup floor = person's floor).

Queue rendering: people wait with bottom = floorY(f) where floorY = (f-1)*FH+4.

Spawn:
```
function spawn(floor,count){
  const q=waiting[floor];
  let added=0;
  for(let i=0;i<count;i++){
    if(q.length>=MAXQ) break;
    let target=randFloorExcept(floor);
    const p=createPerson(floor,target);
    q.push(p); building.appendChild(p.el);
    // stagger appearance: start slightly right & fade in
    added++;
  }
  renderQueue(floor);
  updateStats();
  kickScheduler();
}
```

Appear animation: set initial left = queueX + 30, opacity 0, then requestAnimationFrame set to final with opacity 1. Since transition handles, nice slide-in. Simpler: set final left but opacity 0 → next frame opacity 1 (fade in place). Do fade-in.

renderQueue(floor): q.forEach((p,i)=> p.el.style.left=(QUEUE_X0+i*30)+'px'); Only for people still in scene (not boarding) — those removed from array already.

createPerson: hue random 0-360; el.dataset/attr data-tip = `目标 ${target} 层`; also title attribute fallback? CSS tooltip suffices.

Scheduler:
```
function findJob(elev){
  let best=null,bd=1e9;
  for(let f=1;f<=FLOORS;f++){
    const eff=waiting[f].length-(claims[f]||0);
    if(eff>0){
      const d=Math.abs(f-elev.floor);
      if(d<bd){bd=d;best=f;}
    }
  }
  return best;
}
function tryAssign(elev){
  if(elev.state!=='idle')return;
  const f=findJob(elev);
  if(f==null)return;
  elev.state='busy';
  claims[f]=(claims[f]||0)+1;
  runCycle(elev,f);
}
function kickScheduler(){elevators.forEach(tryAssign);}
```

runCycle async as described; wrap in try/catch? fine.

Careful: tryAssign sets state busy synchronously — good for concurrency within same event loop.

Also update stats: waiting total = sum queue lengths; delivered count; update DOM.

Header indicators above shafts: div.hall positioned at shaftX width 70, text centered: shows ▲/▼ and current shown floor of each elevator. Place in a .headers row above building with position relative height 34; each hall absolute left shaftX.

Also maybe show elevator letter A/B/C on hall. hall: `A · 3 ▲`.

Panel on car also shows. Both update via updatePanel(elev).

Init: build shafts, cars, labels, elevators array [{id:'A',car,...,floor:1,shown:1,state:'idle'}...]. Set car bottom via style (no transition initially).

Ground/grass: page bottom bar under building: simple. Building sits on a .base div (height 14, dark). Then some sky gradient page background.

Also doors initial closed; maybe open them at floor1 initially for looks? Keep closed.

Add keyboard? no.

One more nuance: "一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。" — matches our design: capacity 1, others wait for next elevator.

Random target excludes current floor. 

Test scenarios to describe: spawn multiple people same floor → elevators come one by one, each takes one; hover shows target; stats update; works across floors.

Also handle MAXQ=16 per floor.

Now transitions conflict: person transition includes left .45s — during exit I set left 92 and want .5s; fine same-ish.

When person reparented into car, hover tooltip still positions above person relative person — inside car top area; bottom:110% may go above car — visible since overflow visible. good.

Edge: user spawns people on floor where elevator doors open and boarding in progress — queue arrays handle.

Potential bug: claim decrement timing — decrement right when person leaves queue (shift). Do claims[f]-- after shift.

Another: findJob uses elev.floor which updates only after move completes; during 'busy' not used.

Elevator busy but waiting at pickup with doors open for person... person always exists since claimed. ok.

Let me also add slight idle behavior: if no job, doors closed already. Fine.

Write CSS polish: page bg gradient (#1f2733? or light?). Let's do a pleasant light-sky theme: body bg linear-gradient(#8ec5fc,#e0c3fc)? Maybe dusk: linear-gradient(#0f2027,#203a43,#2c5364) dark with building light. Dark page + light building pops nicely. Text white. Controls dark glass.

Building bg: #f4f6fb with lines #d9e0ec. Shafts: #2f3a4a. Cars: gradient #e8edf3 frame with darker interior. Doors metallic. Hall indicators: dark chips with neon text.

Fonts: system font stack; title bold.

Controls row: floor select styled, count input, buttons with gradients + hover lift. Stats chips.

Let me write the code now.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>电梯模拟器</title>
<style>...</style>
</head>
<body>
<div class="app">
  <header>
    <h1>🛗 电梯模拟器</h1>
    <p class="sub">3 部电梯 · 每梯限乘 1 人 · 悬停乘客查看目标楼层</p>
  </header>
  <div class="toolbar">
    <label>楼层 <select id="floorSel"></select></label>
    <label>人数 <input id="countInp" type="number" min="1" max="8" value="3"></label>
    <button id="spawnBtn">生成乘客</button>
    <button id="randBtn" class="ghost">随机生成</button>
    <div class="stats">
      <span class="chip">等待 <b id="statWait">0</b></span>
      <span class="chip">已送达 <b id="statDone">0</b></span>
      <span class="chip">运行 <b id="statRun">0</b></span>
    </div>
  </div>
  <div class="stage">
    <div class="headers" id="headers"></div>
    <div class="building" id="building"></div>
    <div class="base"></div>
  </div>
</div>
<script>...</script>
</body>
</html>
```

CSS details:

```
*{box-sizing:border-box;margin:0;padding:0}
body{min-height:100vh;display:flex;justify-content:center;align-items:flex-start;padding:28px 16px;font-family:"Segoe UI",system-ui,-apple-system,"PingFang SC","Microsoft YaHei",sans-serif;background:radial-gradient(1200px 600px at 50% -10%,#33506b,#101822 70%);color:#eef3fa;}
.app{width:900px;max-width:100%}
header{margin-bottom:14px}
h1{font-size:26px;letter-spacing:1px}
.sub{color:#9fb2c8;font-size:13px;margin-top:4px}
.toolbar{display:flex;flex-wrap:wrap;gap:10px;align-items:center;background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.12);backdrop-filter:blur(6px);padding:10px 14px;border-radius:12px;margin-bottom:18px}
label{font-size:13px;color:#c6d3e2;display:flex;align-items:center;gap:6px}
select,input{background:#0e1622;color:#e8eef7;border:1px solid #33465e;border-radius:8px;padding:6px 8px;font-size:13px;outline:none}
input{width:60px}
button{background:linear-gradient(135deg,#4f8cff,#6a5cff);color:#fff;border:none;border-radius:8px;padding:7px 16px;font-size:13px;cursor:pointer;transition:transform .15s,box-shadow .15s;box-shadow:0 4px 14px rgba(80,120,255,.35)}
button:hover{transform:translateY(-1px);box-shadow:0 6px 18px rgba(80,120,255,.5)}
button:active{transform:translateY(0)}
button.ghost{background:rgba(255,255,255,.1);box-shadow:none;border:1px solid rgba(255,255,255,.2)}
.stats{margin-left:auto;display:flex;gap:8px}
.chip{background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.14);padding:5px 10px;border-radius:999px;font-size:12px;color:#c6d3e2}
.chip b{color:#7CFC9A;margin-left:2px}
.stage{position:relative}
.headers{position:relative;height:36px;margin-bottom:6px}
.hall{position:absolute;top:0;width:70px;text-align:center;font-size:12px;background:#0c141f;border:1px solid #2c3d55;color:#8fe3a8;border-radius:8px;padding:5px 0;font-family:ui-monospace,Consolas,monospace;letter-spacing:1px;box-shadow:inset 0 0 8px rgba(124,252,154,.15)}
.building{position:relative;width:860px;height:680px;border-radius:10px 10px 0 0;background:linear-gradient(#fbfcfe,#eef1f7);background-image:repeating-linear-gradient(to top,transparent 0 67px,#cfd8e6 67px 68px),linear-gradient(#fbfcfe,#eef1f7);box-shadow:0 20px 50px rgba(0,0,0,.45);overflow:hidden}
```

Wait width: computed 820 earlier; let me recompute final numbers: X0=58, shafts at 58,138,218, width 70 → end 288. QUEUE_X0=306. 16 people: 306+15*30=756 + person width 20 = 776. Building width 800 enough. Set width 820, hall/labels fine. Height 680.

Labels:
```
.flabel{position:absolute;left:10px;font-size:11px;color:#8b98ab;font-weight:600}
```
bottom set inline per floor: floorY + 22.

Shaft:
```
.shaft{position:absolute;top:0;bottom:0;width:70px;background:linear-gradient(90deg,#28303e,#333d4d 50%,#28303e);box-shadow:inset 0 0 0 1px rgba(0,0,0,.25), inset 8px 0 14px rgba(0,0,0,.35), inset -8px 0 14px rgba(0,0,0,.35);z-index:3}
```
Add shaft floor ticks? subtle: repeating gradient lines on shaft too (semi transparent) to show movement reference: background-image repeating lines rgba(255,255,255,.05). ok combine.

Car:
```
.car{position:absolute;left:4px;width:62px;height:60px;background:linear-gradient(#d9e1ea,#b9c4d2);border-radius:6px;box-shadow:0 4px 12px rgba(0,0,0,.4),inset 0 0 0 1px rgba(0,0,0,.2);z-index:5}
.car .inner{position:absolute;left:5px;right:5px;top:16px;bottom:3px;background:#1d2530;border-radius:3px;overflow:hidden?} 
```
Hmm doors inside car: structure:
.car > .panel, .cab (interior), .door.left, .door.right
.cab{position:absolute;left:5px;right:5px;top:16px;bottom:3px;background:#1c242f}
.door{position:absolute;top:16px;bottom:3px;width:26px;/* (62-10)/2 */transition:width .3s ease;background:linear-gradient(90deg,#97a3b2,#cdd6e0 45%,#8d99a8);z-index:2}
.door.left{left:5px;border-right:1px solid #5f6b7a}
.door.right{right:5px;border-left:1px solid #5f6b7a;background:linear-gradient(90deg,#8d99a8,#cdd6e0 55%,#97a3b2)}
.car.open .door{width:0}

Person inside car appended as child: person bottom:0 relative to car; car padding? car interior bottom is 3px; person bottom 0 → slightly into frame bottom, fine visually (feet at car bottom edge). Person left 21: center of 62 = 31; person width 20 → left 21. But cab opening starts at left 5 → person at 21 spans 21-41, opening spans 5-57 fine.

Person z-index inside car: set .person.in{z-index:1}. Doors z 2 close over. Panel z 3.

When person walks out (left→92px) it passes over door area right side — doors open width 0, ok, and over car frame edge — fine, then over shaft bg.

Person base CSS in building context: z-index 6.

Tooltip:
```
.person::after{content:attr(data-tip);position:absolute;bottom:calc(100% + 6px);left:50%;transform:translateX(-50%) translateY(3px);background:#101720;color:#8fe3a8;border:1px solid #2c3d55;font-size:11px;padding:3px 8px;border-radius:6px;white-space:nowrap;opacity:0;pointer-events:none;transition:opacity .18s,transform .18s;z-index:99}
.person::before? arrow? add small triangle via border trick requires another pseudo — ::before used for head? Person uses child divs for head/body, so ::before free: triangle:
.person::before{content:'';position:absolute;bottom:calc(100% + 1px);left:50%;transform:translateX(-50%);border:5px solid transparent;border-top-color:#101720;opacity:0;transition:opacity .18s;z-index:99}
.person:hover::after,.person:hover::before{opacity:1;transform:translateX(-50%) translateY(0)}
.person:hover{z-index:40}
```
Triangle transform on hover: keep same translateX. fine.

Head/body:
.person .h{width:11px;height:11px;border-radius:50%;margin:0 auto;background:hsl(var(--hue),70%,74%);box-shadow:inset -2px -2px 3px rgba(0,0,0,.15)}
.person .b{width:16px;height:22px;margin:2px auto 0;border-radius:7px 7px 6px 6px;background:hsl(var(--hue),65%,55%);box-shadow:inset -3px 0 5px rgba(0,0,0,.15)}
Person total height = 11+2+22=35 → set height:35px.

Base under building:
.base{height:16px;width:880px;margin-left:-10px;background:linear-gradient(#3a4relas} — simpler: width:840? Building 820 wide; base width 840 margin-left -10; background linear-gradient(#46536a,#2c3547); border-radius:0 0 8px 8px. And ground: .ground{height:10px;background:#1b2330;margin-top:0;width:100%;border-radius:4px}? Skip extra.

JS constants:
```
const FLOORS=10,FH=68,X0=58,SW=70,SG=10,CARW=62,PW=20,QX0=306,QGAP=30,MAXQ=16;
const floorY=f=>(f-1)*FH+4;
const shaftX=i=>X0+i*(SW+SG);
```

State:
```
const waiting={};for(let f=1;f<=FLOORS;f++)waiting[f]=[];
const claims={};
let delivered=0;
```

Build DOM:
```
const building=document.getElementById('building');
// labels
for(let f=1;f<=FLOORS;f++){const l=document.createElement('div');l.className='flabel';l.textContent=f+'F';l.style.bottom=(floorY(f)+22)+'px';building.appendChild(l);}
// elevators
const elevators=['A','B','C'].map((id,i)=>{
  const shaft=document.createElement('div');shaft.className='shaft';shaft.style.left=shaftX(i)+'px';building.appendChild(shaft);
  const car=document.createElement('div');car.className='car';car.innerHTML='<div class="panel">1</div><div class="cab"></div><div class="door left"></div><div class="door right"></div>';
  car.style.bottom=floorY(1)+'px';
  shaft.appendChild(car);
  const hall=document.createElement('div');hall.className='hall';hall.style.left=shaftX(i)+'px';hall.textContent=id+' · 1';
  headers.appendChild(hall);
  return {id,car,shaft,hall,panel:car.querySelector('.panel'),floor:1,shown:1,state:'idle',arrow:''};
});
```

Car inside shaft: shaft overflow visible default; car absolute within shaft left 4. Person reparented into car — car overflow visible, shaft overflow visible, building overflow hidden — exiting person walking right at top floor stays inside building (shaft3 x 218+4+92=314 <820 ok).

updatePanel(e):
```
function panel(e){e.panel.textContent=(e.arrow?e.arrow+' ':'')+e.shown;e.hall.textContent=`${e.id} · ${e.arrow?e.arrow+' ':''}${e.shown}`;}
```
Color hall arrow? keep simple.

moveCar as earlier; also toggle e.arrow; after arrive arrow='' update.

doors(e,open): e.car.classList.toggle('open',open); return sleep(330);

runCycle:
```
async function runCycle(e,f){
  try{
    await doors(e,false);
    if(e.floor!==f) await moveCar(e,f);
    await doors(e,true);
    claims[f]--;
    const p=waiting[f].shift();
    if(!p){e.state='idle';return tryAssign(e);} // safety
    renderQueue(f);
    // walk in
    p.el.classList.add('in');
    p.el.style.left=(shaftX(e.index)+4+21)+'px';
    await sleep(470);
    e.car.appendChild(p.el);
    p.el.style.left='21px';
    p.el.style.bottom='0px';
    await sleep(30);
    await doors(e,false);
    await moveCar(e,p.target);
    await doors(e,true);
    // walk out
    p.el.style.left='96px';
    setTimeout(()=>p.el.style.opacity='0',280);
    await sleep(720);
    p.el.remove();
    delivered++;
    updateStats();
    await sleep(120);
  }finally{
    e.state='idle';
    tryAssign(e);
  }
}
```
Hmm finally sets idle then tryAssign — but tryAssign sets busy before starting next cycle; recursion depth ok (async chain).

Wait: e.index — add index when creating.

Note bug risk: `claims[f]--` then shift — if waiting[f] empty (shouldn't happen). ok.

Also door close at start even if already closed → 330ms delay each cycle; acceptable (adds rhythm). Could check: if car.classList.contains('open') await doors close else skip. Optimize: `if(e.car.classList.contains('open')) await doors(e,false);`

findJob/tryAssign/kick as before.

renderQueue:
```
function renderQueue(f){waiting[f].forEach((p,i)=>{p.el.style.left=(QX0+i*QGAP)+'px';});}
```

createPerson:
```
function createPerson(f,target){
  const el=document.createElement('div');
  el.className='person';
  el.style.setProperty('--hue',Math.floor(Math.random()*360));
  el.setAttribute('data-tip','目标 '+target+' 层');
  el.innerHTML='<div class="h"></div><div class="b"></div>';
  el.style.bottom=floorY(f)+'px';
  el.style.left=(QX0+waiting[f].length*QGAP)+'px';
  el.style.opacity='0';
  requestAnimationFrame(()=>requestAnimationFrame(()=>el.style.opacity='1'));
  return {el,home:f,target};
}
```

Spawn handler:
```
function spawn(f,n){
  n=Math.max(1,Math.min(8,n|0));
  let added=false;
  for(let i=0;i<n;i++){
    if(waiting[f].length>=MAXQ)break;
    let t=f;while(t===f)t=1+Math.floor(Math.random()*FLOORS);
    const p=createPerson(f,t);
    waiting[f].push(p);building.appendChild(p.el);added=true;
  }
  renderQueue(f);updateStats();
  if(added)kick();
}
```

updateStats:
```
function updateStats(){
  let w=0;for(let f=1;f<=FLOORS;f++)w+=waiting[f].length;
  statWait.textContent=w;statDone.textContent=delivered;
  statRun.textContent=elevators.filter(e=>e.state!=='idle').length;
}
```
statRun changes during cycles without updateStats calls — call updateStats in tryAssign and at cycle end too. Cheap. Add updateStats() inside tryAssign and in finally.

Random button: spawn(random floor, 1+rand(5)).

Init: fill floorSel options; kick() initially (no jobs, fine). Panels init.

Double-check moveCar timing: dur=250+250*dist. For 9 floors: 2500ms. ok.

Car transition property only bottom with cubic-bezier ease. Set once per move. Also initial no transition — set style.transition='' initially fine.

One subtlety: elevator assigned to floor where it already is and doors open? Initial doors closed. After completing a cycle at target floor, doors remain open; if next job is same floor, skip close (classList check), skip move, open doors (already open — toggling 'open' true no-op but sleep 330; skip if open? doors(e,true) when open: classList already has; transition no visual change but we still sleep 330 — fine or skip: if not contains open, await; else nothing). I'll add guards.

Person tooltip while riding: data-tip still shows target — good ("目标 X 层").

Also add small "ding"? no audio.

Also add direction arrow coloring: hall indicator uses monospace; fine.

Person transition property: `transition:left .45s cubic-bezier(.4,0,.2,1), opacity .35s ease;` When reparenting we change left from scene-coord to 21 instantly — same computed pixel, no jump, but transition may still animate if value differs — we set equivalent value so no animation triggers (value change triggers transition only if different; it IS different (e.g., 239px→21px) but it's a different containing block! Transition would animate 239→21 within new parent causing visual slide! Problem!

When reparenting, style.left='239px' relative to car would be off-screen; we set to 21px — the change happens after appendChild in same style recalc? appendChild then immediately set left — browser recalcs once; transition triggered by property change from old computed (239 in old parent) to new (21 in new parent). Transitions don't run across reparenting in most browsers (element detached/reattached resets) — generally, moving an element to a new parent does NOT animate transitions because style is recomputed fresh? Actually transitions can trigger since computed value changes. Chrome: reparenting cancels running transitions and new style applied... It's ambiguous. Safe approach: temporarily disable transition:
```
p.el.style.transition='none';
e.car.appendChild(p.el);
p.el.style.left='21px';p.el.style.bottom='0px';
p.el.offsetHeight; // reflow
p.el.style.transition='';
await sleep(30);
```
This guarantees no animation. Good.

Similarly exit: person inside car transitions left 21→96 with .45s — desired walk-out.

Another subtlety: person bottom '0px' inside car vs standing at floorY: car bottom = floorY(1)... at pickup floor f, car bottom = floorY(f), person scene bottom = floorY(f) → relative 0. Good.

Person walking in: starts left at queue pos, same floor — only left animates. enterX = shaftX+4+21: shaft1: 58+25=83; shaft2:163; shaft3:243.

During walk-in, person z-index 6 > car 5 — person walks over car frame: looks like entering. Could set z lower so frame overlaps person slightly (feet behind frame bottom edge). Person bottom at car bottom 0 → feet at frame bottom edge; frame has 3px bottom border — fine.

Doors closing over person: person child of car z default (auto, 0-ish stacking as child) — doors z-index 2 → doors above. Panel z 3. Person appears behind doors. 

But person CSS `.person` z-index:6 applies — inside car stacking context, person z 6 vs doors z 2 → person ABOVE doors! z-index on .person class is global. Need override: `.car .person{z-index:1}` and `.car .person:hover{z-index:1}`? Hover z-index 40 — override inside car too: `.car .person, .car .person:hover{z-index:1}`. Tooltip pseudo z 99 within car context — still above doors? pseudo belongs to person (z1 stacking?) z-index 99 on pseudo within person's context — pseudo with z-index 99 but person creates stacking context? Person has z-index:1 + position absolute → creates stacking context; pseudo z99 inside stays under doors (doors z2 > person z1). Tooltip under doors when riding — doors open when stopped so visible anyway; while moving, tooltip hidden behind closed doors — acceptable.

But overflow: building hidden clips nothing relevant.

Queue overflow: MAXQ 16 per floor; queue x max = 306+15*30=756,+20=776<820. good.

Now check initial render: building width 820; stage width = building 820; app width 860? set .app width 880 and center building. base width 840 centered (margin-left:-10 relative building). Simpler: .stage{width:820px;margin:0 auto}. .base{width:840px;margin-left:-10px}. .headers width 820.

halls at left shaftX within headers (position absolute) — matches building shafts since same width origin. Good.

Hall text e.g. "A · 1" and during move "A · ▲5". panel similar small.

Also maybe add per-floor waiting count badge? skip.

Accessibility/labels fine.

Write the final code carefully. Also add `<meta viewport>`. Language zh.

Let me also consider elevators preferring nearest; ties: fine.

One more feature check: "允许用户在不同楼层生成一定数量的人" — floor select + count + button. ✓.

Test instructions: open file in browser; spawn 5 people on floor 1 → watch elevators A/B/C each take one person sequentially, remaining wait; hover person shows tooltip; people delivered to random floors; stats update. Provide run: just open the HTML file (double-click) — no server needed.

Now finalize moveCar with floor display scheduling. Also clear arrow on arrival.

```
function moveCar(e,to){
  return new Promise(res=>{
    const from=e.floor,dist=Math.abs(to-from);
    if(dist===0){res();return;}
    e.arrow=to>from?'▲':'▼';panel(e);
    const dur=260+260*dist;
    e.car.style.transition=`bottom ${dur}ms cubic-bezier(.55,.05,.35,1)`;
    e.car.style.bottom=floorY(to)+'px';
    const dir=Math.sign(to-from);
    for(let k=1;k<=dist;k++)setTimeout(()=>{e.shown=from+dir*k;panel(e);},Math.round(dur*k/dist));
    setTimeout(()=>{e.floor=to;e.shown=to;e.arrow='';panel(e);res();},dur+30);
  });
}
```

panel():
```
function panel(e){
  const t=(e.arrow?e.arrow+' ':'')+e.shown;
  e.panel.textContent=t;
  e.hall.innerHTML=`${e.id} · ${e.arrow?`<span style="color:${e.arrow==='▲'?'#7CFC9A':'#ffb86b'}">${e.arrow}</span> `:''}${e.shown}`;
}
```
innerHTML each update fine.

Doors guards:
```
function setDoors(e,open){
  if(e.car.classList.contains('open')===open)return Promise.resolve();
  e.car.classList.toggle('open',open);return sleep(320);
}
```

runCycle final:
```
async function runCycle(e,f){
  try{
    await setDoors(e,false);
    if(e.floor!==f)await moveCar(e,f);
    await setDoors(e,true);
    claims[f]--;
    const p=waiting[f].shift();
    if(!p)return;
    renderQueue(f);updateStats();
    p.el.style.left=(shaftX(e.idx)+25)+'px'; // 4+21
    await sleep(480);
    p.el.style.transition='none';
    e.car.appendChild(p.el);
    p.el.style.left='21px';p.el.style.bottom='0px';
    void p.el.offsetWidth;
    p.el.style.transition='';
    await setDoors(e,false);
    await moveCar(e,p.target);
    await setDoors(e,true);
    p.el.style.left='98px';
    setTimeout(()=>{p.el.style.opacity='0';},300);
    await sleep(750);
    p.el.remove();
    delivered++;updateStats();
    await sleep(100);
  }finally{
    e.state='idle';
    kick();
  }
}
```
Wait — p.el.style.bottom while in scene: queue bottom = floorY(f); inside car 0. transition none during reparent covers bottom too. Then transition restore to CSS value (class-based). ok since inline '' reverts to stylesheet.

But wait: .person transition defined in stylesheet; inline '' → stylesheet applies. Good.

enterX = shaftX+4+21 = shaftX+25.

After setDoors(false) post-board, tiny pause maybe sleep(80) then move — fine without.

kick():
```
function kick(){elevators.forEach(e=>{if(e.state==='idle')tryAssign(e);});updateStats();}
```
tryAssign:
```
function tryAssign(e){
  if(e.state!=='idle')return;
  let best=-1,bd=1e9;
  for(let f=1;f<=FLOORS;f++){
    if(waiting[f].length-(claims[f]||0)>0){
      const d=Math.abs(f-e.floor);
      if(d<bd){bd=d;best=f;}
    }
  }
  if(best<0)return;
  e.state='busy';
  claims[best]=(claims[best]||0)+1;
  runCycle(e,best);
}
```

Elevator creation add idx.

Spawn button handler reads select value & count.

Random: const f=1+rand(10); spawn(f, 1+rand(5)).

Also seed demo? Maybe spawn a few on load for instant life: spawn(1,4); spawn(5,2)? Nice touch — yes, small initial spawn so user sees action immediately. I'll spawn 3 at floor 1 and 2 at floor 7 after slight delay.

updateStats also updates statRun via states — call within kick and cycle end (finally → kick → updateStats ✓) and tryAssign start? state changes to busy in tryAssign; kick calls updateStats after forEach — covers. runCycle end finally calls kick → updates. Good.

Edge: tryAssign called directly from runCycle finally through kick — fine.

CSS for .flabel z-index 2 behind shafts.

Also add subtle person bobbing while waiting? Could add keyframes idle sway:
.person .b{animation:...}? Might conflict with walking... add gentle head bob animation infinite — cute:
.person .h{animation:bob 2.4s ease-in-out infinite}
@keyframes bob{0%,100%{transform:translateY(0)}50%{transform:translateY(-1.5px)}}
Different delays: set el.style animationDelay random via inline on .h? Use el.querySelector('.h').style.animationDelay=(Math.random()*2)+'s'. Nice.

Ground/base + maybe small trees? skip.

Check building background lines align with floorY: floors at bottom (f-1)*68; line every 68px from top? Use `to top` repeating gradient: transparent 0→67, line 67→68. So lines at multiples of 68 from bottom — matches floor boundaries. Car bottom = boundary+4, height 60, top at 64 < 68 next line, 4px gap below ceiling line. Person bottom +4 stands on line region (line is 1px at 67-68? Actually line at y 67-68 from bottom means just below next floor boundary... hmm: repeating to top with period 68: each period 0-67 transparent, 67-68 colored. Starting from bottom: colored at bottom 67..68 → that's the boundary between floor1 and floor2 (y=68). Floor1 occupies 0..68; its ceiling line at 67..68. Person bottom 4 → stands 4px above floor1 bottom (y=4) — but the "floor surface" for floor1 is y=0 (bottom of building). Person floats 4px above bottom edge. Car similarly. There's no line at y=0. Looks fine (small margin). Actually people on floor 1 stand near building bottom — ok.

Labels bottom = floorY+22 = (f-1)*68+26.

Now writing hall indicator position: headers relative; hall absolute left shaftX. Hall width 70 matches shaft.

Responsive: building fixed 820 — on small screens overflow; add .stage{overflow-x:auto}? Wrap: .stagewrap{overflow-x:auto}. Fine add.

Also person data-tip for riding shows target; after delivery removed.

Count input clamp in spawn.

Let me also ensure select options 1..10 label "1 层" etc.

Final check on z-order: building children: labels (z2), shafts(z3) contain cars(z5 within shaft context — shaft z3 creates context; car z5 inside shaft; people z6 in building context → people above shafts AND cars. Walking in: person (z6, building) above car ✓. Once inside car, person within shaft context (z3 overall) below other queue people ✓ fine.

Tooltip of queue person z: person hover z40 → tooltip above all ✓.

Door open state visual: doors width 0 reveals cab dark interior; person inside visible standing — good.

Car panel at top: top:3px within car, car height 60; doors top:16 → panel area visible above doors. Panel z 3 within car ✓ (above doors 2? panel positioned top area only, no overlap anyway).

Car box-shadow gives depth on shaft.

I think design solid. Also mention capacity rule in UI text.

Delivered counter + waiting counter.

Also disable spawn button spam? not needed.

Now write final file. Also include brief comments in code (Chinese) for readability. Then provide run instructions + test notes concisely.

Double-check person inside car bottom: car height 60, person 35, bottom 0 → head top at 35, panel at top 3..~17; doors top 16 — head (top 35 from bottom = 25 from top) below panel ✓.

person walk-out left 98: car 62 wide; 98-62=36 beyond right edge, person width 20 → from 98 to 118 fully outside at 98? person spans 98..118, car right edge 62 → fully outside ✓. Building coords: shaftX+4+98: shaft3: 218+102=320 → 320..340 within building ✓.

Everything good. Also handle prefers-reduced-motion? skip for brevity... could add quickly: media query killing animations — optional, skip.

Write it.

Also one more: when elevator arrives at pickup and doors open, before person walks — add small sleep(120) for readability? Boarding walk 480 covers.

Potential race: two elevators assigned same floor sequentially — claims prevent: after first claims, eff = len-1; if len 2, second elevator can also claim same floor → both go, each takes one person. That matches "下一部电梯到达时他们再依次上电梯" ✓.

If len 1 and claimed, second elevator finds other floors or idles ✓.

After elevator takes person and claims[f]--, remaining len-1 with claims 0 → other idle elevator gets assigned via kick? kick called at cycle end and spawns only. Remaining people wait until some elevator finishes → finally→kick ✓. But if an elevator is idle (never got job because everything claimed) it stays idle until next kick event. Kicks happen at every cycle completion — enough ✓. Also kick after setDoors? The boarding completion kicks only at very end of cycle; but right after taking a person (claims--), other idle elevators could immediately target remaining queue — kick at that point too: after renderQueue(f), call kick() (but current elevator is busy; other idles get assigned) ✓ better responsiveness. Add kick() there — careful recursion: kick calls tryAssign on idle elevators only; current is busy. Fine.

Also after moveCar to target and person exits, elevator might take new job from same floor if people waiting there — handled by kick in finally.

Delivered stat increments then kick.

Alright, also the exit fade uses opacity transition .35s defined ✓.

Also on spawn, people fade-in: set opacity 0 then rAF→1 ✓.

Now finalize HTML and write out. Also test mentally syntax.

One thing: `.car{transition:...}` — we set inline per move; initial car placement without transition fine.

Also `e.car.style.transition` persists; subsequent moves overwrite ✓.

Font sizes small. OK.

Write the answer: brief intro, the single HTML file, run instructions, test checklist. Keep reasonably concise per instructions but complete.