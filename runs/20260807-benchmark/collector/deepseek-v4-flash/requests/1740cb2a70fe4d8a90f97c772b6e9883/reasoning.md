We need answer in Chinese likely. Need create 3D watch animation running current time, smooth hands, date/week, dual timezone, complete 3D watch. Need deliver runnable artifact. Since this is chat, can provide HTML file with Three.js? Need maybe use CSS 3D? Need "3D wristwatch" can be WebGL. Need "directly complete task and deliver runnable product; provide run instructions and test instructions." Need produce code likely single HTML file. Need include current time, dual timezone, date/week. Need smooth second hand (continuous). Need 3D watch with dial, hands, case, strap, maybe digital subdial for second timezone/date/week. Need ensure "正常运行" in browser. We need craft code. Need not use external? Could use CDN Three.js, but maybe offline? We can include three.js via CDN. Need "complete 3D watch" maybe use Three.js with OrbitControls. We should create self-contained HTML that loads Three.js from CDN. Need maybe include fallback? Let's design.

Need satisfy: "秒针、分针和时针" smooth. Need date/week display. Need support two timezones. Need "full 3D wristwatch" with case, bezel, dial, crown, strap, hands. Use Three.js primitives and maybe text via CanvasTexture for date/week/timezone labels. Need current time update each frame: rotate hands based on time. Need handle timezones. Could display second timezone in a small digital window on dial using Canvas texture? Need dynamic text on 3D dial. In Three.js, dynamic canvas texture can update per frame. Use one canvas for dial? Or overlay HTML labels? But output wants 3D wristwatch; can use HTML overlay for date/timezone? Better integrate into 3D via canvas textures: create a canvas representing dial with marks and date/window and second timezone window. But hands are 3D objects rotating. Need update date/week and timezone display every frame, redraw texture maybe. Could do dynamic texture for dial subwindows.

Alternative: Use a 3D scene and HTML/CSS overlay on top for date/week/timezone. But "complete 3D wristwatch" maybe acceptable? However user specifically asks "以完整的 3D 腕表形式呈现" and "提供日期和星期显示功能" and "追踪两个时区". We can implement 3D dial with markers and use dynamic canvas texture to update digital date and timezone. That's better.

Need think about implementation details.

We can use Three.js r128 or newer via CDN. Need code robust. Use ES module? Simpler use non-module build from CDN (three.min.js). For older, OrbitControls. Need maybe use cdnjs https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js. OrbitControls can be included from unpkg? But no need? We can implement simple orbit with pointer? But for viewing 3D, user needs rotate around. Better include OrbitControls. r128 examples/js controls? Not with module? We can set up custom camera controls manually to avoid external controls. But easier: include OrbitControls via script from cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js. Need check load order. But maybe CDN examples js works with global THREE. We'll use:

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

Need maybe if internet? Our answer can say needs internet for CDN. Fine.

Need design.

Scene:
- Renderer WebGLRenderer antialias, setSize, setPixelRatio.
- Camera perspective, position (8, 6, 10), lookAt origin.
- OrbitControls with enableDamping, minDistance, maxDistance, maybe autoRotate? Could auto rotate slowly when idle? Not necessary.
- Lights: ambient, directional, point.
- Ground? maybe shadow? could use grid? But wristwatch complete. Use subtle environment? We can add a reflective floor? Avoid.

Watch group:
- Coordinate axes: dial in XZ plane, facing +Z? Need hands rotate around Z? Better: watch face lies in XY plane? Let's choose face in XZ plane, normal +Y? Then hands rotate around Y axis. But date texture on dial plane? Easier use plane geometry in XY? Need text orientation.

Let's decide: Watch lies in XY plane, dial faces camera at +Z. Hands rotate around Z axis. But then wrist strap extends along X? If watch wristband horizontal. Camera sees watch face. 3D case has thickness along Z. This is natural for viewing. Need hand rotations around Z: angle = - (seconds/60)*2*PI? In Three.js, if dial on XY plane and 12 o'clock at +Y, rotating around Z from +Y to +X? Actually angle 0 = +Y. At t=15s, should point to +X? Need compute.

For clock hands, use geometry pointing up +Y. Rotate around Z by angle. At 12:00:00, angle 0. At 3:00, hour hand angle -Math.PI/2? Wait rotating object with rotation.z = theta rotates +Y axis to ( -sin theta? Need formula: In Three.js, rotation about Z by angle: vector (0,1,0) becomes (-sinθ, cosθ,0). If θ positive, counterclockwise from X to Y? For θ=-90 degrees, vector becomes (1,0,0) = 3 o'clock? Let's check: coordinate system right-handed, positive rotation about Z is CCW (from X to Y). At θ=+90, Y axis rotates to (-? x=-1?). Wait (0,1,0) under Rz(θ): (x', y') = (x cosθ - y sinθ, x sinθ + y cosθ) with x=0,y=1 => (-sinθ, cosθ). θ=+90 -> (-1,0) = 9 o'clock. θ=-90 -> (1,0) = 3 o'clock. Good. At 12, θ=0. At 3:00, need +X => θ=-π/2. At 6:00, -Y => θ=-π. So angle = - (s / 60) * 2π? At 15 seconds, s=15 => -π/2 => +X. Yes. So for seconds smooth: rotation.z = -( (milliseconds/1000) /60)*2π? Actually seconds hand should point at 12 at 0 seconds, at 15 sec = 3 o'clock. So angle = - (seconds fraction /60)*2π. For minute: angle = - (minutes + sec/60)/60*2π. For hour: angle = - ((hour%12 + minutes/60 + sec/3600)/12)*2π.

Need create hand geometry. Need pivot at center. For each hand: group? Use mesh with geometry offset so base at origin? We can create Object3D at center, add mesh whose geometry extends upward from (0,0,0). Set rotation.z. Date window/dial stay plane.

Dial:
- Use CylinderGeometry for case back/body? Maybe watch case: CylinderGeometry radius=3 height=0.8, axis Z, rotated? CylinderGeometry default axis Y. Need rotate to align with Z if face in XY plane. Let's choose easier: watch face in XY plane, normal +Z. Use CylinderGeometry rotation.x = Math.PI/2? Actually cylinder axis Y, after rotate x=π/2, axis becomes Z. Use for case bezel and body.

We can create group `watch`.

Case:
- `new THREE.Mesh(new THREE.CylinderGeometry(3, 3, 0.8, 64), new THREE.MeshStandardMaterial({color:0x2a2a2a, metalness:0.8, roughness:0.3}))`; rotate.x=Math.PI/2. Position z=0? If axis Z, height along Z. Put back at z=-0.4 to z=0.4. Add bezel ring: cylinder radius 3, height 0.15, maybe at z=0.5? Wait if axis z after rotation, height 0.1 along z; position z=0.35? We'll use torus? Need face at z=0.5? Let's set case cylinder from z=-0.4 to +0.4. Bezel ring torus at z=0.4? Actually dial face should be at z=0.42. Dial disc slightly in front. Need not overcomplicate.

Can build:

const caseMat = new THREE.MeshStandardMaterial({ color:0x3a3a3a, metalness:0.8, roughness:0.4 });
const caseBody = new THREE.Mesh(new THREE.CylinderGeometry(3, 3, 0.8, 64), caseMat);
caseBody.rotation.x = Math.PI/2; // axis Z. Then cylinder's top/bottom circular face in XY plane? Wait CylinderGeometry: radius top/bottom, height. Axis along Y. Rotating x=π/2 maps Y to Z? Let's verify: rotation.x=π/2: Y axis -> Z? For R_x, vector (0,1,0) -> (0,0,1). So cylinder axis along Z. The circular cap lies in plane perpendicular to axis, i.e. XY plane. Good. Position z=0. The top cap (at y=+h/2) maps to z=+h/2? Yes. So face at z=0.4.

Dial disc: CircleGeometry radius=2.8 with dynamic texture, plane perpendicular to Z. Create Mesh with CircleGeometry and MeshStandardMaterial map. CircleGeometry in XY plane, normal +Z. Position z=0.38? Need not z-fight. Put at z=0.38, just in front of case front? Actually dial face should be within case at z=0.39 maybe. Case top at z=0.4, so dial at z=0.395? But circle mesh is 2D, can be at z=0.39. To avoid z-fight, set z=0.4 exactly? Use polygonOffset. Better add bezel ring in front to create recess: case front opening, dial inside at z=0.35. Need implement dial plane at z=0.39 maybe. We'll set case body height 1.0, dial at z=0.49? Wait if case body from z=-0.5 to z=0.5, front cap at z=0.5. We need dial at z=0.48. The circle plane can be at z=0.48. There is no visible cap? CylinderGeometry has caps, so at z=0.5 there's circular cap. If dial plane at z=0.48, it's inside the cylinder volume just behind front cap. But front cap would hide dial (opaque). Need not have front cap? Use `CylinderGeometry(3,3,1,64,1,true)` open-ended to show dial? Or create case body open-ended and add dial at front. Easier: use open-ended cylinder for side wall, and dial circle as front face. Need also back face. Better create two circle meshes: case back and dial.

Watch case construction:
- `caseSide`: CylinderGeometry(3, 3, 0.8, 64, 1, true) open-ended, rotation.x=π/2, axis Z, position z=0. This is tube from z=-0.4 to +0.4.
- `dialPlate`: CircleGeometry(2.95, 64) with dial texture, position z=0.4? Actually plane at z=0.4 closes front.
- `caseBack`: CircleGeometry(3, 64) with metal mat, position z=-0.4, rotation? CircleGeometry normal +Z; at back should face -Z. Need rotate y=π? Or use material side: BackSide? Easier create circle geometry and rotation.y=Math.PI. position z=-0.4. But if side tube and back circle, nice.
- Bezel: `TorusGeometry(2.95, 0.15, 16, 64)` rotated x? Torus default in XY plane, normal Z. We can place at z=0.4? Torus lies around origin in XY plane, no need rotation? Actually TorusGeometry is in XY plane, hole along Z. Yes. Position z=0.4? If centered at z=0.4, it forms ring around dial. But if dial at z=0.38, ring at z=0.42. Need torus tube radius 0.15. It may protrude beyond. Add bezel slightly in front. Also add crystal glass? Could use transparent sphere/cylinder? Maybe not needed. Add glass dome: CircleGeometry or sphere segment with transparent material? Use `MeshPhysicalMaterial` with transmission? r128 may support? Better simple transparent `MeshPhongMaterial` with opacity=0.2, side=DoubleSide, color=0x99ccff. Add a flat disc at z=0.45? Need no.

- Crown: CylinderGeometry(0.3, 0.3, 0.5, 16) positioned at side +X? Watch case radius=3, crown at x=3.3, z=0. Need rotate? Cylinder axis Y, but crown should protrude along X if it's at side. Use rotation.z=Math.PI/2? Cylinder axis along Y, rotate z=π/2 maps Y to? Rz(π/2): (0,1,0)->(-1,0) axis X. Position x=3.4. Good.
- Lugs and strap: Need "wristwatch" complete. We can add a strap extending along X? Watch face centered at origin. Lugs at top and bottom? Wait typical watch strap is along 12-6 direction (vertical in face plane). But our face in XY plane, 12 at +Y; strap along Y. If watch viewed face-on, strap extends up/down (Y). But wristwatch 3D could have strap from top and bottom, curving. Need create two strap segments from x? Actually watch face in X-Y plane; strap should connect to top (y=3) and bottom (y=-3) and curve around wrist. But if wrist orientation? We can create a curved band along Y, with two parts extending from lugs at y=3 and y=-3, curving backward? Wait strap should go around wrist behind watch, not in same plane. The watch face is on top of wrist, strap wraps around wrist in Y-Z plane maybe. Need coordinate: Face normal +Z. The strap should extend away from face along -Z after passing lugs? Hmm imagine watch on wrist, face normal points outward (away from wrist). The strap attaches at 12 and 6 positions, wraps around wrist. The wrist axis maybe along X? Let's set wrist axis along X, face normal +Z? But strap wraps in plane perpendicular to wrist axis, around Y/Z. The watch face located at z? Actually if wrist axis along X, the watch face lies in XY plane? Wait wrist axis along X, around wrist (Y-Z circular cross-section). Watch face tangent to wrist at top, normal +Z. The strap attaches at 12 and 6 (y positive/negative) and wraps around Y-Z? Actually 12 and 6 positions are along y-axis in face plane, yes. The strap goes from top and bottom lugs around the wrist, behind face. This is complex but can make an annular/cylinder strap centered at some point behind watch: a torus segment? Use a large torus oriented around X? Let's attempt simple: Create a torus along X? Actually strap as partial torus centered at (0,0,-4) maybe, radius 4? Need attach to lugs at y=±3, z=0. A circle of radius 4 centered at (0,0,-3)? Points at y=3, z=0 satisfy 3^2+3^2=18 not. Maybe use TubeGeometry along a curve that starts at y=3, z=0 and curves to y=-3,z=0 through z negative. Could use CatmullRomCurve3 from (0,3,0) to (0,-3,0) with control points (0,2,-0.8), (0,0,-2.8), (0,-2,-0.8)? Wait watch face at z=0, strap behind at z negative. But the watch's case has back at z=-0.4; strap should attach at y=±3, z=0, then go backward. Yes. Use a curve from (0,3,0) to (0,-3,0) passing through (0,1.5,-1.5), (0,-1.5,-1.5)? Actually a curve in Y-Z plane. But then the watch face normal +Z, strap protrudes negative Z. That's correct: watch face is at front, strap behind. But if viewer sees front, strap mostly hidden. The 3D watch will have strap behind case. Need render enough. Use TubeGeometry with radius 0.8 along curve, color 0x111111. Also need lugs? Add two small rectangular blocks at y=±3, z=0 connecting case to strap. But if strap curve starts at y=3.2,z=0 and goes to y=-3.2,z=0, with control points (0,3.2,-1.2), (0,0,-2.5), (0,-3.2,-1.2). Actually if the curve passes through (0,3.2,0) and (0,-3.2,0), it is behind. Use `CatmullRomCurve3([new THREE.Vector3(0,3.2,0.2), new THREE.Vector3(0,1.5,-1.8), new THREE.Vector3(0,-1.5,-1.8), new THREE.Vector3(0,-3.2,0.2)])`. TubeGeometry radius=0.7. But tube may intersect lugs. Good.

Need also add lugs: small boxes from case edge to strap? Maybe not necessary; case and strap enough. But a wristwatch has lugs. Could create two boxes at top/bottom: BoxGeometry(1.0,0.6,0.4) positioned at y=3,z=0.1? Actually attach to case. But not essential.

Need dial texture:
Need create Canvas for dial face 1024x1024. Draw:
- Background gradient? radial gradient from white/cream to gold? Use canvas.
- Outer minute markers: 60 ticks, 12 hour markers.
- Numbers 1-12.
- Brand text maybe "TZ DUAL".
- Date window at 3 o'clock (or 4:30) and weekday at 6? Actually dynamic text overlay should update per frame. We can either draw whole dial texture once and update only date/week and second timezone subdial dynamically. To simplify, create dial canvas texture and redraw all every few frames? Could be okay: 1024 canvas drawing per frame maybe expensive. Better draw static dial once as base texture, and have separate small dynamic canvas textures for date/window and second timezone. Or one combined dial texture update only every minute? But seconds hand rotates independently; date/week won't need per frame. Timezone digital seconds? Could update per second. We can update dynamic subtextures every few frames. Need seconds hand smooth independent of texture.

Let's design components:

1. Main dial:
- Static `dialCanvas` 1024x1024, background, tick marks, numerals, brand. Use `new THREE.CanvasTexture(dialCanvas)`; assign as map to dial material.
- Add "Automatic / 50m" text.

2. Date window:
- Need display current date (day number) and weekday (一二三四五六日 maybe). Could be implemented using a separate small mesh plane positioned over dial at 3 o'clock? But if dial texture includes window recess, dynamic plane can show number. We can create a small `CanvasTexture` for a "date wheel" that shows double-digit date and weekday? Need date and week functions. Use one canvas for a small window at 3 o'clock, e.g. width 200, height 120, with background white/cream, text date big, weekday small? But "日期和星期显示" can be separate: date at 3 o'clock, weekday at 6 o'clock? We can create two dynamic planes: date window near 3 o'clock (or between 3 and 4) and weekday subdial near 6. But must be visible integrated.

3. Second timezone:
- Need support "simultaneously track two time zones." Could add a subdial at 6 o'clock with a small hand? That is watch-like: GMT second timezone. But need "two time zones" and display. We can add a 24-hour subdial with its own hand? More elegant.
- Alternatively, a digital display at 9 o'clock showing "NY 14:32:45" via dynamic canvas texture. But in 3D, digital display on watch face could be small lcd. Need dynamic texture update.
- We can implement both: a 24-hour GMT hand and a digital LCD subdial. But "3D wristwatch" maybe with 24h bezel? Let's choose a second timezone subdial at 6 o'clock: a small 12/24-hour dial, with a second timezone hour hand (and minute maybe) rotating. But user wants current time and two time zones. We can also show current local timezone and other timezone in an HTML overlay? Need integrate.

Let's think: The watch can have:
- Main hands for local time (hour, minute, smooth second).
- 24-hour GMT hand, centered (or subdial) indicating second timezone (UTC or any configured timezone). This is a standard GMT watch. Also date and weekday.
- Need "simultaneously track two time zones" satisfied.

But "具有平滑转动的秒针、分针和时针" only for main hands. Need second timezone hand smooth too? Could make GMT hand rotate once per 24 hours. Need maybe subdial at 6 with 24-hour markers and small hand. But a full 3D watch is complex but doable.

Need date/week and timezone are dynamic. Could add separate "digital subdial" using canvas texture that updates every frame. This is straightforward and clearly displays second timezone with time, date, weekday. But a wristwatch with digital display plus analog hands is a hybrid watch. It can be "complete 3D wristwatch." Let's do that to reduce complexity of GMT hand? Actually a digital subdial is easy.

Need create a small rectangular LCD display on dial at 6 o'clock or 9: maybe using CanvasTexture. It can show:
Line1: "LON 14:32:45" (second timezone)
Line2: "2026-05-14 THU" maybe? But date/week already elsewhere.
Need date/week display maybe also in digital subdial. But user asked date and week; a digital subdial can display both. Need main analog hands too. That is okay.

Maybe implement two modes:
- Date/week window at 3 o'clock (analog watch with date window)
- Second timezone digital display at 9 o'clock or bottom subdial.

Need dynamic textures per frame? Could update a small canvas texture for LCD. We can use CanvasTexture, update `needsUpdate = true` every frame? That's okay, small canvas (256x128) drawing each frame maybe fine but can optimize every second. Seconds smooth in analog; LCD can update every second or every 100ms. For date/week, update every minute/day. We can update digital LCD every frame with current time in other timezone, but drawing per frame maybe okay for a small canvas. Need not update static dial.

Need choose in code. Implement with two dynamic canvases:
- `dateCanvas`: draws date number and weekday; update every minute? Actually date and weekday change at midnight local; we can update each frame or every minute. Simpler redraw in animation loop every 500ms maybe. But CanvasTexture update every frame can affect performance; but 256x128 minimal. We'll set update throttles.

- `tzCanvas`: draws second timezone name and HH:MM:SS; update every 250ms or every frame.

Could also use HTML overlay for date/timezone to avoid texture update. But in 3D, if user rotates, overlay not part of 3D? We could overlay just maybe no. Better texture.

Need implement date/week in Chinese? User requested Chinese, we can use "星期一" etc. Maybe include English/Chinese? Use "周一" etc. Date format `2026/05/14`? Need date and week. Use browser current local date. For second timezone, use `Intl.DateTimeFormat` with `timeZone`? Need allow configuration. We can set secondTimeZone = 'America/New_York' perhaps. But user may be in China; need "同时追踪两个时区"; maybe local and UTC? Could provide UI to select second timezone. Need support. Since no user interaction? We can add a small HTML control overlay to select timezone and date format? But "runnable product" maybe include drop-down to choose second timezone. The prompt: "它应支持同时追踪两个时区" means maybe watch should display two time zones. We can implement with a select dropdown for second timezone in HTML overlay. That is acceptable. The watch then displays local time and selected timezone. Need include one or more timezones.

Need maybe add "local timezone" displayed on dial? For second timezone, digital subdial show selected IANA timezone abbreviation/time. For local time, maybe main dial label "LOCAL" and at bottom LCD? We can also show local time in digital subdial? Hmm.

Need design digital subdial:
- Place a rectangle at position (-1.2, -1.5, dialZ + 0.02)? Need orientation. Dial face is XY plane at z=0.38. For dynamic digital display, use PlaneGeometry(1.8, 0.7) at position x=-0.8, y=-1.4, z=0.381. But it must be on top of dial. Use CanvasTexture. Create plane with `MeshBasicMaterial` transparent, depthWrite false. Because dynamic texture includes background rectangular LCD. Need set transparent true with alpha? Use canvas with LCD background rounded rect and text. The plane itself if material transparent with map alpha? We can draw background on canvas. Ensure not z-fight: position z = dialZ + 0.01. But if dial is circle at z=0.38, this plane z=0.39. Hands rotate at z=0.42? Need order. Need hand pivot at z=0.42 maybe. The digital plane can be in dial plane. Hands should be above all dial elements. Need set hand z positions: hour hand z=0.44, minute 0.46, second 0.48. But if the digital plane at 0.39, hands pass over. Fine.

Need maybe date window at 3 o'clock: static? Actually we can use dynamic date window plane at (1.2, 0.6, 0.39) or (1.15, 0.5) away from hands? But date window at 3 o'clock often between center and 3. Hands can cover it but okay. Need dynamic dateCanvas small plane at position (1.4, 0.4, 0.39), maybe size 1.2x0.6. The date and weekday in one window? At 3 o'clock, window can show date number and weekday maybe too small. Better place date at 3 o'clock and weekday at 9 o'clock? But second timezone digital could be bottom.

Need maybe not clutter. Let's design:

Dial layout:
- Center analog hands.
- 3 o'clock: Date window (dynamic plane) at (1.6, 0.6).
- 9 o'clock: Weekday window at (-1.6, 0.6)? This is unusual.
- 6 o'clock: Digital dual-time display (dynamic plane) at (0, -1.8).

But watch face with date at 3 and weekday at 3? Many watches have date window at 3 (day+date at 3). Day and date combined in a rectangular window: e.g. "THU 14" at 3 o'clock. We can combine date and weekday in one dynamic plane at 3 o'clock. That is simpler. Use a small canvas with two lines:
"THU"
"14"
At 3 o'clock. Need maybe date also includes month? Maybe date number only. User asks 日期和星期; date number and weekday enough. Could include month "MAY 14". The window could show "MAY 14 THU". Nice.

Second timezone digital display at 6 o'clock:
"NYC 08:32:45"
Maybe local time? Actually second timezone. Need show timezone name. Could include date maybe "05/14 THU" in small line.

Need also show selected second timezone in HTML overlay and maybe update if changed. We'll put a select in top-right with common timezones. The 3D digital display updates. Need maybe local timezone displayed? Main analog hands show local time. We can add label "LOCAL" near 12? Static dial.

Need 3D objects:
- Hands: Use meshes with custom shapes. Use ExtrudeGeometry? Could create shapes:
  - Hour hand: a tapered polygon (oblong) or "paddle". Use Shape and ExtrudeGeometry depth 0.02. Need pivot at bottom. But Shape coordinates in XY plane. ExtrudeGeometry extrudes along Z, good. Need shape spans from y=0 to y=1.8 with width tapering. Use `new THREE.Shape();` moveTo(-0.12,0); lineTo(-0.15,1.2); lineTo(-0.05,1.8); lineTo(0.05,1.8); lineTo(0.15,1.2); lineTo(0.12,0); lineTo(0.3,-0.15); lineTo(-0.3,-0.15); close? Need tail. Actually create shape including tail. Extrude depth 0.02, bevel false. Since shape lies XY, extrude along Z. Rotate? Mesh at z=0.43? The shape orientation points +Y at 12. Rotation.z sets orientation. Need position z so hand lies in XY plane, thickness along Z. OK.
  - Minute hand: length 2.3, thinner.
  - Second hand: length 2.6, needle thin, with counterweight tail.
- Center pin: small cylinder? Use cylinder along Z? Actually cylinder axis Y by default; need rotate x=π/2 to align Z. Or use sphere. Use small sphere at center.

Need ensure hand shape geometry extends from origin along +Y. For minute hand, shape from y=0 to 2.3, tail negative. Pivot at origin. Good. Need if taper shape has y negative tail; rotation z rotates whole shape. OK.

Need maybe add shadows? Could enable castShadow. Need directional light. But not necessary.

Need "smooth" seconds hand: Use current time with milliseconds. In animation loop:
```
const now = new Date();
const local = {h: now.getHours(), m: now.getMinutes(), s: now.getSeconds(), ms: now.getMilliseconds()};
const secondsFraction = (now.getSeconds() + now.getMilliseconds()/1000);
const minutesFraction = now.getMinutes() + secondsFraction/60;
const hoursFraction = (now.getHours() % 12) + minutesFraction/60;

secondHand.rotation.z = - (secondsFraction / 60) * 2 * Math.PI;
minuteHand.rotation.z = - (minutesFraction / 60) * 2 * Math.PI;
hourHand.rotation.z = - (hoursFraction / 12) * 2 * Math.PI;
```
Need maybe sweep behavior? Smooth.

For GMT/second timezone digital display:
Need compute local time in selected timezone. Use `Intl.DateTimeFormat` with timeZone, hour12 false. But to update every second, get parts:
```
function getTimeInZone(tz) {
  const now = new Date();
  const parts = new Intl.DateTimeFormat('en-GB', {
    timeZone: tz,
    hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false
  }).formatToParts(now);
  const obj = {};
  for (const p of parts) obj[p.type] = p.value;
  return `${obj.hour}:${obj.minute}:${obj.second}`;
}
```
Need timezone abbreviation? `Intl.DateTimeFormat('en-US', {timeZone: tz, timeZoneName:'short'})` get "EST". Use two formatters? Could use one with timeZoneName: 'short' and hourCycle h23. But include date? For LCD, display e.g. "NY" + time. Need maybe selected timezone label. We'll show timezone abbreviation and time.

Need maybe update date/week:
```
const weekday = now.toLocaleDateString('en-US', {weekday:'short'}).toUpperCase();
const month = now.toLocaleDateString('en-US', {month:'short'}).toUpperCase();
const day = now.getDate();
```
Could show "THU MAY 14". Need local date. For Chinese? Use English names easier in small window. But user Chinese; perhaps use "周四 05-14" or "星期四". Need font support in Canvas: Chinese fonts can be rendered by browser but extended chars; okay. But 3D texture using Chinese text with default fonts maybe okay. Need maybe in Canvas "周一" etc. But to ensure cross-platform, use English abbreviations for date and weekday. But prompt Chinese not requiring Chinese UI. We can include Chinese in HTML overlay.

Let's make date/week display in Chinese:
- Weekdays: ['周日','周一','周二','周三','周四','周五','周六']; date: `${month}月${day}日` maybe. Need Canvas font "bold 48px Arial". Chinese characters fine. Could use "周四" and "05/14". Need not.

But if browser's font doesn't support Chinese? Most do. Use "Microsoft YaHei, PingFang SC, sans-serif". Good.

Need use date window compact:
`drawDateWindow(ctx, canvas)`:
- clear
- background rounded rect white? Actually color #f7f3e8.
- text area: maybe left line "周四", right line "14" or "5月14日".
Maybe create two squares? Date window at 3 o'clock normally one aperture showing two discs. We'll draw a small rounded rectangle, draw weekday small at top, date big below:
```
ctx.fillStyle = '#0f172a'; // text
ctx.font = 'bold 42px sans-serif'; textAlign='center'; fillText(weekdayChinese, 120, 55);
ctx.font = 'bold 64px sans-serif'; fillText(String(day).padStart(2,'0'), 120, 115);
```
Canvas size 240x160. Plane size 1.5 x 1.0 maybe. Position (1.55,0.6,?).
Need dial static includes a cutout rectangle outline around date window? We can draw a rectangle border in static dial at same position/size. Then dynamic plane placed inside. But if dial plane at z=0.38 and dynamic at z=0.382? Need align. We can draw window outline in static, then dynamic plane exactly covers the inner area. OK.

Second timezone LCD:
- Canvas size 512x256 maybe. Draw dark blue/black rounded rectangle, green/cyan digital font? Use monospace. Text:
  line1: "NYC" small, line2 "08:32:45" large, line3 "05/14 THU" small. Maybe.
- Need update each second. We'll create plane size 2.6 x 1.3 at position (0, -1.85) maybe. But watch radius 3, bottom area y=-1.85 is within dial. Need hands may pass over it but okay. However date window at 3 and LCD at 6 leaves center. Need hour markers at 6 behind LCD? Maybe not. We can avoid drawing markers at 6? Dial static still has maybe. But LCD covers. We can place LCD at y=-1.5, size 2.0 x 1.2; leave marker at 6 if visible outside? Could be okay. But a digital display with large area maybe.

Maybe better second timezone subdial as a small window at 9 o'clock? Let's place at (0, -1.8), not as digital lcd but an analog subdial with 24h? Hmm.

Need "simultaneously track two time zones" more visible. Maybe use a digital LCD at 6 o'clock. It can update. Great.

Potential issue: Dynamic texture plane will be covered by hands. Digital LCD at bottom may be behind hands when minute hand points down. That's fine.

Need add 3D "complete wristwatch": Need maybe strap "behind" should be visible from front's perspective. But if watch face is in XY plane, strap behind along -Z. The camera at z positive sees face, strap behind maybe invisible except around edges. If user rotates, can see strap. Need wrist? Could be acceptable. Maybe add a wrist cylinder underneath? That would make "wristwatch" more complete. But user asked wristwatch animation, not human wrist. We can add a stylized wrist/strap band behind the case. The strap should wrap around wrist. Let's implement a curved strap using TubeGeometry. It will be behind, can be seen from rotated angles. Also add lugs. Good.

Need maybe add a small "bezel rotation" or not.

Need provide "运行方式和必要的测试说明." In final, include artifact code block and instructions. Need "直接完成任务并交付可运行产物" so final should include complete HTML code. Could also mention "保存为 .html, 浏览器打开." Need "测试说明" maybe change system time? But in browser cannot set system time. We can test by observing hands match local time, date/week, select timezone. Need maybe in code allow manual override? For testing, perhaps add time offset? User asks current time. For test, we can add a "time scale" or "time offset" maybe not needed. But "necessary test instructions" can include comparing with system clock and changing timezone dropdown. We can include a little "Now" test? Hmm.

Need maybe include `controls` to rotate. If no menu, how choose timezone? Need add an HTML overlay with select. Good.

Need maybe add `renderer.outputEncoding = THREE.sRGBEncoding` for colors.

Need use r128? Need know methods. `THREE.CanvasTexture`, `THREE.Shape`, `THREE.ExtrudeGeometry`, `THREE.CatmullRomCurve3`, `THREE.TubeGeometry`, `THREE.OrbitControls`.

Need maybe webgl context lost? No.

Need ensure no coding errors. Let's write full code in final. Need be careful.

Let's construct code step by step.

HTML skeleton:
```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>3D 腕表动画</title>
<style>
 body { margin:0; overflow:hidden; background: radial-gradient(circle,#1a1a2e,#000); font-family: sans-serif; }
 #info { position:absolute; top:16px; left:50%; transform:translateX(-50%); color:#eee; background:rgba(0,0,0,0.4); padding:8px 16px; border-radius:20px; font-size:14px; pointer-events:none; z-index:10;}
 #controls { position:absolute; top:16px; right:16px; color:#fff; z-index:10; background:rgba(0,0,0,0.4); padding:10px 14px; border-radius:12px; font-size:14px;}
 #controls label { color:#fff; }
 select { margin-left:8px; padding:3px; font-size:13px; border-radius:6px; }
 #local { position:absolute; bottom:16px; left:50%; transform:translateX(-50%); color:#ccc; background:rgba(0,0,0,0.4); padding:8px 16px; border-radius:20px; font-size:13px; z-index:10; pointer-events:none;}
</style>
</head>
<body>
<div id="info">🖱 拖动旋转 · 滚轮缩放</div>
<div id="controls"><label>第二时区: <select id="tzSelect"> ... </select></label></div>
<div id="local"></div>
<script src=...></script>
<script>
...
</script>
</body>
</html>
```

Need choose timezones. Include Asia/Shanghai, Asia/Tokyo, Europe/London, America/New_York, Australia/Sydney, UTC? Use options. Maybe default America/New_York? Since user may be in China? We can default "Europe/London" for visible contrast? The watch label line1 can show abbreviation. Need use `Intl` with timeZone. If default "Asia/Shanghai", second timezone same as local if user in China, less interesting. We can default "UTC" maybe. Let's set default "UTC" to show universal time. But "同时追踪两个时区" - local time plus UTC. Good.

Need local text bottom: "本地时间: 2026-05-14 周四 14:32:45 | 第二时区 UTC: ..." updated.

Need main dial static. Let's build function `createDialTexture()`.

Dial canvas size 1024. Since texture on CircleGeometry radius 2.8, pixel ratio: canvas 1024, circle radius 2.8, coverage roughly from -2.8 to 2.8. Need draw all within center 512 border.

Dial face:
```
const c = document.createElement('canvas'); c.width = c.height=1024;
const ctx = c.getContext('2d');

// background radial gradient
const g = ctx.createRadialGradient(512,512,50,512,512,512);
g.addColorStop(0,'#fdfdf5');
g.addColorStop(1,'#cfd2d8');
ctx.fillStyle=g; ctx.fillRect(0,0,1024,1024);

// minute markers
for (let i=0;i<60;i++) {
 const angle = (i/60)*Math.PI*2;
 const outer = 430; // radius relative to 512? Need scale.
 // Because canvas 1024, radius max 512. Dial circle radius 2.8 world; on screen circle radius fills to 430? We'll draw in canvas radius 470.
 const outerR=460, innerR= i%5===0? 420 : 435;
 ...
}
```
Need transform coordinates: Canvas coordinate y down, angle mapping: Clock 12 = -π/2, 3 = 0. For each marker, angle = (i/60)*2*Math.PI - Math.PI/2. x = cx + r*cos(angle), y=cy + r*sin(angle). Draw lines. For hour markers maybe draw rectangles/numbers.

Need draw circle border at r=470, inner circle at r=350? We'll draw.

Need draw hour numerals 12,1,...11. Use font bold 64px, textAlign center. Need positions maybe r=340. Use `ctx.fillText(num, cx+340*cos(angle), cy+340*sin(angle)+? )` Need adjust y by ~20. For text, y coordinate baseline. Use `ctx.textBaseline='middle'`; then y = cy + r*sin(angle).

Need draw "AUTOMATIC" at y=300? and "DUAL TIME" at y=380? maybe. Need date window frame at 3 o'clock? Let's design positions.

Dial static with:
- Outer ring: "60" markers.
- Hour markers: at 12,3,6,9 as thick rectangles maybe; but date/LCD may cover below. We'll draw.

Maybe drawer:
```
function line(ctx, x1,y1,x2,y2) { ... }
```
Need if use `ctx.lineWidth` and stroke.

Need draw brand text `3D WATCH` at upper half below 12:
```
ctx.font="italic 48px Georgia, serif";
ctx.fillStyle="#3a3a3a";
ctx.fillText("TZ · GMT", 512, 240);
```
But maybe no.

Need add date window frame at position x=760, y=600? We can define dynamic plane size. Let's define date window world position (1.55,0.6) and world size 1.5x1.0. Because dial radius 2.8, pixel conversion 512/2.8 ≈ 183 px per world unit. So dynamic plane size in pixels = world size * 183 = 274x183. Let's set canvas 300x200. Static frame at center (1.55*183, 0.6*183) ? Actually dial center 512,512. World x to canvas: x*183 +512. World y: canvas y = 512 - y*183 (because world +Y up, canvas +Y down). So date center world (1.55,0.6) -> canvas (512+1.55*183≈795, 512-0.6*183≈402). Date window size world 1.6x1.0 -> pixels 293x183. We can draw a frame at (795,402) around. But dynamic plane size 1.6x1.0. Need align exactly. We'll create dynamic date canvas 320x200. The plane's geometry world size matches ratio 1.6 x 1.0; if canvas is 320x200 ratio 1.6:1. Good. Static frame in dial at x=795, y=402 with width=320, height=200. Need draw frame (black/white border) around that area. But dynamic plane will cover inner area; static outline can be drawn behind. Need if dynamic plane exactly covers frame, no border visible. We can draw static frame as "window" with border and then dynamic plane slightly smaller. Or draw static frame and dynamic plane same size, but dynamic plane has own border. It may overlay static. Fine.
Maybe easier: static dial includes a dark rectangular outline: `ctx.strokeRect(795-160, 402-100, 320, 200)`. Dynamic plane covers it. Good.

Need digital LCD plane at world bottom center (0,-1.85) size 2.4 x 1.3. In canvas: center (512, 512+1.85*183=850) for width 2.4*183=439, height 238. Static dial can include an outline. But if we use dynamic LCD with its own background, static outline not needed. Need maybe draw "DUAL TIME" label above LCD? We can draw in static at (512, 780). Then dynamic plane below? Wait LCD plane centered at (0,-1.85) means y canvas 851. It covers bottom of dial. But marker at 6 at r=430? y canvas = 512+430=942, so LCD at y 851-119=732 to 970. It would cover marker. Not good. Maybe center LCD at (0,-1.5) size 2.0 x 1.1. Canvas center y=787, size from 686 to 888. Hour marker at 6 (y=942) still visible. Good. Use world center (0,-1.5), size 2.2 x 1.2. In canvas center y=787. Need maybe crosses center? not.

Maybe put LCD at 6 o'clock but below center, okay.

Need choose date window at 3 position (1.5,0.5). It will partly cover markers? Fine.

Need maybe static dial draw ring for subdial around digital? Not necessary.

Need create dynamic date plane:
```
function makeDateTexture() {
 const c=document.createElement('canvas'); c.width=320; c.height=200; return {canvas:c, texture:new THREE.CanvasTexture(c)};
}
function updateDateTexture(dateCanvas, dateTexture) {
 const ctx = dateCanvas.getContext('2d');
 ctx.clearRect(0,0,dateCanvas.width,dateCanvas.height);
 // white background with rounded rect
 ctx.fillStyle = '#fffdf5';
 ctx.fillRect(0,0,320,200);
 // text
 const now = new Date();
 const weekdays = ['周日','周一','周二','周三','周四','周五','周六'];
 const weekday = weekdays[now.getDay()];
 const month = now.getMonth()+1;
 const day = now.getDate();
 ctx.fillStyle = '#0b0b0b';
 ctx.textAlign='center'; ctx.textBaseline='middle';
 ctx.font='bold 30px "Segoe UI", Arial, sans-serif';
 ctx.fillText(weekday, 160, 45);
 ctx.font='bold 80px "Segoe UI", Arial, sans-serif';
 ctx.fillText(`${month}月${day}日`, 160, 115); // Might be too big; "5月14日" 5 chars at 80 px width 400, too wide. Use "05-14" or "14" only.
}
```
Need date and week in one window. Maybe draw:
- Line1 weekday (Chinese)
- Line2 date `05/14` (month/day) in bold 56px.
Let's make:
```
ctx.fillText(weekday, 160, 55);
ctx.font='bold 62px "Arial"';
ctx.fillText(now.toLocaleDateString('en-GB', {day:'2-digit', month:'2-digit'}), 160, 120);
```
But "05/14" width about 150, okay. But month not explicit. Could use "14" big; maybe enough. User asks date; date number plus weekday okay. But date includes month? We can include "5月14日" with smaller font:
`ctx.font='bold 48px "Microsoft YaHei", "PingFang SC", Arial, sans-serif';`
`ctx.fillText(month + '月' + day + '日', 160, 115);` String "5月14日" length 4, 48px ~ 192px, fits 320. Good.
Need if month/day 2 digits, "12月31日" length 5, 48px ~ 240, fits. Good.

Update every minute? But if day changes, dateTexture update. We can call every 1000ms? Simpler every frame? But date texture update every frame maybe okay but not needed. We'll set `lastDateUpdate` time and update once per second? Might be okay.

Need create LCD texture:
```
function createLCDTexture() { c.width=640; c.height=280; ...}
function updateLCDTexture(...) {
 const now = new Date();
 // second timezone date/time using Intl
 const tz = select.value;
 const time = getTimeInZone(tz);
 const date = getDateInZone(tz);
 const tzAbbr = getTimeZoneAbbr(tz, now);
 // clear/draw
 ctx.clearRect...
 // draw rounded rect with LCD background
 ctx.fillStyle='#eaf6ff'; roundRect(...);? Actually LCD can be dark with light text. Let's use dark greenish:
 ctx.fillStyle='#0b1f16'; roundRect(0,0,640,280,16); ctx.fill();
 ctx.fillStyle='#7dfbaa'; // LCD green
 ctx.font='bold 36px "Courier New", monospace';
 ctx.fillText(tzAbbr, 40, 50); // hmm
 ctx.font='bold 100px "Courier New", monospace';
 ctx.fillText(hh:mm:ss, 320, 150);
 ctx.font='bold 32px "Courier New", monospace';
 ctx.fillText(MM/DD WEEK, 320, 235);
}
```
Need use `Intl.DateTimeFormat` with timeZone. Need no repeated formatter too expensive? We can create once and cache per timezone? Use function:
```
function getParts(tz) {
 return new Intl.DateTimeFormat('en-GB', {
   timeZone: tz, hour:'2-digit', minute:'2-digit', second:'2-digit', hourCycle:'h23'
 }).formatToParts(new Date());
}
```
Need `hourCycle: 'h23'` in r128? Browser supports. Good. For date:
```
new Intl.DateTimeFormat('en-GB', {timeZone: tz, day:'2-digit', month:'2-digit', weekday:'short'}).formatToParts(now)
```
Could be expensive but okay. We can call 3 formatters each second. Fine.

Potential issue: `Intl.DateTimeFormat` with timeZone and `hourCycle` in older browsers? okay.

Need HTML local overlay update:
```
function updateLocalInfo() { ... }
```
Maybe update in animation.

Need main 3D building.

Let's write full script.

Set up:
```
let scene, camera, renderer, controls;
const HOUR_HAND_ANGLE...;
const watchGroup = new THREE.Group();
let hourHand, minuteHand, secondHand;
let datePlane, lcdPlane;
let dateTexture, lcdTexture;
let dateCanvas, lcdCanvas;
let lastDateUpdate = 0, lastLCDUpdate = 0;
const tzSelect = document.getElementById('tzSelect');
```
Need init:
```
function init() {
 renderer = new THREE.WebGLRenderer({antialias:true, alpha:true});
 renderer.setSize(window.innerWidth, window.innerHeight);
 renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
 renderer.shadowMap.enabled = true;
 document.body.appendChild(renderer.domElement);
 scene = new THREE.Scene();
 camera = new THREE.PerspectiveCamera(40, innerWidth/innerHeight, 0.1, 100);
 camera.position.set(5,4,8);
 camera.lookAt(0,0,0);
 controls = new THREE.OrbitControls(camera, renderer.domElement);
 controls.enableDamping = true; controls.dampingFactor=0.05;
 controls.minDistance=4; controls.maxDistance=16; controls.target.set(0,0,0);
 // lights
 scene.add(new THREE.AmbientLight(0xffffff, 0.5));
 const dir = new THREE.DirectionalLight(0xffffff, 1.2);
 dir.position.set(3,5,6);
 scene.add(dir);
 const dir2 = new THREE.DirectionalLight(0xffffff, 0.3);
 dir2.position.set(-4,-2,3);
 scene.add(dir2);
 // build watch
 buildWatch();
 // event listeners
 window.addEventListener('resize', onResize);
 tzSelect.addEventListener('change', () => { updateLCDTexture(); });
 // start loop
 animate();
}
```
Need use `THREE.sRGBEncoding`? In r128, `renderer.outputEncoding = THREE.sRGBEncoding;` Supported. But if not, okay. Need set `texture.encoding = THREE.sRGBEncoding` for color textures? Maybe not needed.

Build watch:
```
function buildWatch() {
 const mat = (color, opts={}) => new THREE.MeshStandardMaterial({color, metalness:opts.metalness??0.6, roughness:opts.roughness??0.4});

 const caseGroup = new THREE.Group();
 const caseMat = mat(0x556677, {metalness:0.85, roughness:0.3});
 // side
 const side = new THREE.Mesh(new THREE.CylinderGeometry(3,3,0.7,64,1,true), caseMat);
 side.rotation.x = Math.PI/2;
 side.castShadow = true;
 caseGroup.add(side);
 // back
 const back = new THREE.Mesh(new THREE.CircleGeometry(3,64), caseMat);
 back.rotation.y = Math.PI;
 back.position.z = -0.35;
 caseGroup.add(back);
 // dial
 const dialTex = createDialTexture();
 const dialMat = new THREE.MeshStandardMaterial({map:dialTex, roughness:0.6, metalness:0.1});
 const dial = new THREE.Mesh(new THREE.CircleGeometry(3,64), dialMat);
 dial.position.z = 0.35;
 // Case side tube height 0.7 from -0.35 to 0.35, dial at front z=0.35. CircleGeometry normal +Z at z=0.35 closes front. But side open-ended at z=0.35 has rim, dial plane at exactly z=0.35; z-fighting? The rim edge of cylinder at same z but circles have no thickness? Actually open cylinder has no cap, edge at z=0.35. Circle at z=0.35 intersects edge; could look okay. To avoid, set dial z=0.36. But then dial sits slightly in front of side edge. Fine.
 dial.position.z = 0.36;
 dial.castShadow = true;
 caseGroup.add(dial);
 // bezel
 const bezel = new THREE.Mesh(new THREE.TorusGeometry(3,0.12,16,64), caseMat);
 bezel.position.z = 0.36;
 caseGroup.add(bezel);
 // Bezels inner? Torus radius 3 tube 0.12; okay.
 // crystal
 const glassMat = new THREE.MeshPhysicalMaterial({color:0xffffff, metalness:0, roughness:0.05, transparent:true, opacity:0.18, side:THREE.DoubleSide});
 const glass = new THREE.Mesh(new THREE.CircleGeometry(2.88,64), glassMat);
 glass.position.z=0.42;
 caseGroup.add(glass); // It may obscure hands? Hands should be below glass? Actually hands on dial inside, glass above them. If glass at z=0.42 and hands at z=0.46, glass should be above hands (z larger). But if CircleGeometry plane at z=0.42, and hands z=0.46, hands are in front of glass from camera, not inside. Need place glass after? Hmm. In reality, hands are beneath crystal. Crystal should be z=0.7 (outside). Hands z=0.5. But crystal is a flat disc, if at z=0.7, it's thin, okay. If at z=0.42, hands in front. We'll set glass at z=0.55 or 0.72? But case side only to z=0.35; bezel at z=0.36 with torus extends to 0.48; glass at z=0.45? Need hands are within case? Let's define:
  - Case side from -0.4 to 0.4.
  - Dial at z=0.36.
  - Hands z: hour 0.44, minute 0.46, second 0.48. They protrude from dial. Crystal at z=0.55, transparent, in front of hands. But if no visible side beyond z=0.4, crystal seems floating above bezel. Need add a bezel ring/tube from z=0.36 to z=0.6 to enclose hands. Complex. Could ignore crystal or make it at back? Better not include crystal unless using 3D dome. Since user didn't ask. We can remove glass to avoid conflict. Or add glass dome that covers hands but not necessary. Let's remove.

Need add lugs: small boxes:
```
const lugMat = caseMat;
const lugGeo = new THREE.BoxGeometry(1.2, 0.7, 0.5);
const lugTop = new THREE.Mesh(lugGeo, lugMat);
lugTop.position.set(0,3.2,0.1);
caseGroup.add(lugTop);
const lugBottom = new THREE.Mesh(lugGeo, lugMat);
lugBottom.position.set(0,-3.2,0.1);
caseGroup.add(lugBottom);
```
But strap curve starts at y=3.2. Good.

Need strap:

```
function createStrap() {
 const points = [
  new THREE.Vector3(0,3.45,0.35),
  new THREE.Vector3(0,1.8,-0.8),
  new THREE.Vector3(0,0,-1.8),
  new THREE.Vector3(0,-1.8,-0.8),
  new THREE.Vector3(0,-3.45,0.35)
 ];
 const curve = new THREE.CatmullRomCurve3(points);
 const geo = new THREE.TubeGeometry(curve, 64, 0.75, 16, false);
 const mat = new THREE.MeshStandardMaterial({color:0x1c1c1c, roughness:0.9, metalness:0.1});
 const tube = new THREE.Mesh(geo, mat);
 tube.castShadow = true;
 return tube;
}
```
Need attach to watch? Strap behind case, from lugs. But if it starts/ends at y=±3.45,z=0.35, it is at case edge. Good. But watch group includes strap maybe. Need if strap is separate, add to watchGroup. We'll add to watchGroup.

Need perhaps orient strap along Y? It wraps behind -Z; at the back, its center y=0,z=-1.8. That's plausible. But if strap width is 1.5, it may be visible.

Need crown:
```
const crown = new THREE.Mesh(new THREE.CylinderGeometry(0.28,0.28,0.7,12), caseMat);
crown.rotation.z = Math.PI/2;
crown.position.set(3.3,0,0.1);
watchGroup.add(crown);
```
With cylinder axis after rotation z? Let's verify: CylinderGeometry axis Y. rotation.z=π/2 maps Y? Rz(π/2): Y axis -> -X? Wait earlier formula: Rz(θ) maps (0,1,0) to (-sinθ, cosθ,0). At θ=π/2 -> (-1,0). So axis points -X. Still along X. Position x=3.3. OK. But if axis -X, cylinder symmetric, fine. Also add crown ridges? Use small cylinder? okay.
Need crown at x=3.3 may intersect side. Good.

Need hands build. We need shapes.

Create `createHandMesh(shape, color, metalness, depth)`:
```
const mesh = new THREE.Mesh(new THREE.ExtrudeGeometry(shape, {depth: depth, bevelEnabled:false}), mat);
mesh.castShadow = true;
return mesh;
```
Need extrude depth along Z centered? ExtrudeGeometry with depth positive extends from z=0 to z=depth. To center, set shape translate? Actually mesh.geometry.translate(0,0,-depth/2) after creation. Need use `geo.translate(0,0,-depth/2)`. Good.

Define shape functions:
```
function hourShape() {
 const s = new THREE.Shape();
 s.moveTo(-0.12,0);
 s.lineTo(-0.16,1.0);
 s.lineTo(-0.07,1.45);
 s.quadraticCurveTo(0,1.65, 0.07,1.45);
 s.lineTo(0.16,1.0);
 s.lineTo(0.12,0);
 s.lineTo(0.28,-0.3);
 s.lineTo(-0.28,-0.3);
 s.closePath();
 return s;
}
function minuteShape() {
 const s = new THREE.Shape();
 s.moveTo(-0.08,0);
 s.lineTo(-0.1,1.8);
 s.lineTo(-0.04,2.3);
 s.quadraticCurveTo(0,2.5,0.04,2.3);
 s.lineTo(0.1,1.8);
 s.lineTo(0.08,0);
 s.lineTo(0.2,-0.35);
 s.lineTo(-0.2,-0.35);
 s.closePath();
 return s;
}
function secondShape() {
 const s = new THREE.Shape();
 s.moveTo(-0.02,0.3);
 s.lineTo(-0.04,2.6);
 s.lineTo(0.04,2.6);
 s.lineTo(0.02,0.3);
 s.lineTo(0.06,-0.6);
 s.lineTo(-0.06,-0.6);
 s.closePath();
 return s;
}
```
Need second hand color red maybe.
Need center cap: `new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.1,0.1,16), secondMat? position z=0.48)`. Cylinder axis Y; need rotation.x=π/2 and position z=0.475? If axis along Z after rotation. `centerPin.rotation.x = Math.PI/2; centerPin.position.z=0.48;` But position z? need length along z 0.1. OK.
Could add a small black cap under? Not necessary.

Need hand materials:
```
const hourMat = new THREE.MeshStandardMaterial({color:0x222222, metalness:0.4, roughness:0.6});
const minuteMat = same;
const secMat = new THREE.MeshStandardMaterial({color:0xcc2020, metalness:0.3, roughness:0.4});
```
Need hands z:
```
hourHand.position.z = 0.45;
minuteHand.position.z = 0.47;
secondHand.position.z = 0.49;
```
If each extrude depth 0.02 and geometry centers z=0, mesh at z=0.45 spans 0.44-0.46. Good.

Need pivot order? We use three separate meshes. Fine.

Need hands within watch? Dial at z=0.36, case front cap? Actually case side open at z=0.4? Wait side tube height 0.7 from -0.35 to 0.35, dial z=0.36. There is no front cap, dial is front. Hands z=0.45 in front of dial, but no side wall beyond 0.35. They are not encased, but visually they sit on face. Need maybe add a cylindrical bezel wall in front? Maybe a ring/torus at 0.36 with tube radius .12 extends to z≈0.48; hands cannot be covered. Need maybe add a short open cylinder from z=0.35 to z=0.55 to represent crystal/rehaut, to enclose hands. Let's add a "rehaut" ring:
```
const rehaut = new THREE.Mesh(new THREE.CylinderGeometry(2.9, 2.9, 0.2, 64, 1, true), caseMat);
rehaut.rotation.x = Math.PI/2;
rehaut.position.z = 0.44;
caseGroup.add(rehaut);
```
But this would be a transparent? Not needed. Hands are needle-thin; no issue.

Need if dial at z=0.36, hands in front of dial but behind crystal absent. The second hand at z=0.49 is outside the case (case front z=0.35). But since camera sees front, it appears on dial. But when rotating, it may be visibly in front of case edge. That's okay? It might look like hands are above face, natural.

Need maybe set dynamic planes z=0.37? Since hands at z=0.45, digital/date textures lie on dial at z=0.37. Good. If dynamic plane z=0.365 and dial z=0.36, no z-fight if offset. Need use `depthWrite: true` maybe. Good.

Need create dynamic planes:
```
function createDatePlane() {
 const canvas = document.createElement('canvas');
 canvas.width=320; canvas.height=200;
 const texture = new THREE.CanvasTexture(canvas);
 const mat = new THREE.MeshBasicMaterial({map:texture, transparent:true, depthWrite:false});
 const geo = new THREE.PlaneGeometry(1.7, 1.05); // ratio 1.62; use 1.6x1.0.
 const mesh = new THREE.Mesh(geo, mat);
 mesh.position.set(1.5,0.5,0.365);
 watchGroup.add(mesh);
 return {canvas, texture, mesh};
}
```
Need date window size maybe 1.6x1.0. But canvas 320/200 = 1.6. Good.
Need if canvas texture is 0 at first, update before render.

LCD plane:
```
function createLCDPlane() {
 canvas.width=640; canvas.height=280; // ratio 2.286;
 texture = new THREE.CanvasTexture(canvas);
 mat = new THREE.MeshBasicMaterial({map:texture, transparent:true, depthWrite:false});
 geo = new THREE.PlaneGeometry(2.4, 1.05); // ratio 2.286; if canvas 640x280, geometry height=2.4/2.286=1.05. Good.
 mesh.position.set(0,-1.45,0.365);
 watchGroup.add(mesh);
 return...
}
```
Need perhaps add "LCD" no.

Need create dial texture. Need static label positions align with date plane? If date plane at (1.5,0.5), the static dial maybe draw date window frame at corresponding location. Let's compute pixel conversion. Need define `const scale = 512/2.8` because circle radius 2.8 world maps to edge at 512? Actually Canvas 1024, CircleGeometry radius 3 world maps to canvas radius? Wait the texture is applied across CircleGeometry radius 3. In texture coordinate space, u/v from 0 to1 across the circle, with center at 0.5. The canvas 1024 pixels covers entire circle of radius 3. So pixels per world unit = 1024 / (2*3) ≈ 170.67, not 183. Need compute: CanvasTexture maps full canvas to full circle geometry UV (0-1). CircleGeometry radius 3, diameter 6. World coordinate x in [-3,3] maps to u in [0,1] linearly? For CircleGeometry, vertices have uv = (u,v) from plane positions? In Three.js CircleGeometry maps UV across the bounding square? Actually CircleGeometry positions use x,y; UVs? Let's check: CircleGeometry(radius, segments) generates vertices around circle at (x,y,0), with uv I think equal to x/(2r)+0.5? For CircleGeometry, uv coordinates are computed as (x/r+1)/2? Since radius 3 -> x=3 maps to u=1, yes. So pixels per world unit = 1024/(2*3)=170.67. Good. Need use that for alignment. But canvas can be 1536 maybe for higher? Use 1024, enough. Let's define `const PX = 1024 / (2 * 3) ≈ 170.6667`. For center CX=512,CY=512. World coordinate to canvas:
`x_canvas = 512 + worldX * PX`
`y_canvas = 512 - worldY * PX`
Because world +Y up, canvas y down. Good.

Date plane world center (1.5,0.5): canvas (512+256=768, 512-85=427). Plane size world 1.6x1.0 -> canvas 273x171. Canvas dynamic itself 320x200, but plane geometry aspect 1.6; if canvas 320x200, texture aspect 1.6 matches. Good. Static frame should be drawn at 768±160? Wait dynamic plane canvas 320x200 corresponds to world 1.6x1.0. The static dial texture should draw a frame of same world dimensions: pixel width 1.6*PX=273, height=171, centered at (768,427). But dynamic plane canvas (320x200) displays full texture on the same world geometry. The geometry aspect is 1.6 (width/height). The canvas texture aspect 1.6 (320/200) also matches, so the texture fills exactly without distortion. So static frame should use world dimension in pixels 273x171. But if dynamic plane's actual size 1.6x1.0, yes. Use `PX` in static draw. Good. But if dynamic canvas has its own background with rounded rectangle covering full canvas, it will cover static frame. To show a border, static frame can be larger or draw dynamic canvas with transparent margins and border? Simpler: dynamic plane draws a rectangular window with white background; static dial has dark outline just behind. Since dynamic plane covers same area, outline hidden. Need dynamic plane smaller than static outline by a few pixels; set geometry size 1.56 x 0.96, and static frame 1.7x1.1. Or dynamic plane's canvas includes no border; static frame visible around edges if dynamic plane is smaller. Let's do that:
- Date plane world size 1.5 x 0.95 (aspect 1.579). Canvas 320x200 ratio 1.6, slight distortion if not exact. Better use canvas 300x200 ratio 1.5. Then plane size 1.5x1.0 ratio 1.5. Good. Date window center (1.55,0.5). Size 1.5x1.0. Static frame size 1.7x1.2. Dynamic plane covers inner area. Need static frame pixels: width 1.7*PX=290, height=205. Dynamic plane size 1.5*PX=256, height=171. Canvas 300x200 ratio 1.5; geometry 1.5x1.0. Good.

Let's set date plane canvas 300x200. `PlaneGeometry(1.5,1.0)`. Position (1.55,0.5). Static frame centered at (512+1.55*PX, 512-0.5*PX) = (776, 427) if PX=170.7. Draw `ctx.strokeRect(cx - 145, cy - 102, 290, 205)`. Maybe draw dark/white border. Dynamic plane at z=0.365 covers inner part, leaving border visible.
Need if dynamic plane's canvas background white and no border; static frame behind visible around edges. Good.

LCD plane canvas 640x280 ratio 2.2857. Use `PlaneGeometry(2.4, 1.05)` ratio 2.2857. Slight ratio mismatch? 2.4/1.05=2.2857. Good. Center (0,-1.5). Static label maybe "DUAL TIME" at world (0,-0.85). Need not.

Need LCD static frame maybe not.

Need maybe dynamic plane positions z=0.37. The dial material at z=0.36; if MeshBasicMaterial plane is transparent but no depthWrite, rendered maybe. Need set `renderOrder`? Set datePlane.renderOrder=2, lcdPlane.renderOrder=3, dial renderOrder=0, hands renderOrder=1? Maybe if transparent materials render after opaque, okay. Since MeshBasicMaterial transparent true and side? `transparent:true` with depthWrite false; render order default. Need set `depthWrite:false` to avoid hiding hands? Actually dynamic planes on dial, hands above. If transparent plane depthTest true and depthWrite false, it will be occluded by hands if hands in front? Plane at z=0.37, hands z=0.45. When camera sees hands, depth test: hand closer, plane behind, so hand draws over. Good.

Need maybe hands material opaque, so render before transparent? The transparent dynamic planes (renderQueue transparent) render after opaque hands? Wait render order: opaque objects first (hands), then transparent objects. If dynamic planes are transparent, they render after hands. With depthTest true, hands have depth values closer; plane fragments behind fail depth, so plane won't draw over hands. Good. If plane in front, it could draw over? But plane z smaller (behind hands), so okay. Transparent glass? no.

Need maybe dynamic LCD should be opaque but we set transparent true because texture has rounded corners? It can be opaque but with alpha? If `transparent:true`, it sorts after hands, but depthTest prevents overdrawing. OK.

Need set `renderer.sortObjects`? no.

Need update date/LCD textures after creation:
```
updateDateTexture();
updateLCDTexture();
```

Need dynamic texture update:
```
let lastDateMinute = -1, lastLCDSecond = -1;
function updateDynamicTextures() {
 const now = new Date();
 const keyDate = now.getFullYear()+'-'+now.getMonth()+'-'+now.getDate()+'-'+now.getDay();
 if (keyDate != lastDateKey) { drawDate(); lastDateKey=keyDate; }
 const tzKey = tzSelect.value + now.getSeconds() + now.getMinutes();
 if (now.getSeconds() !== lastLCDSecond || tzChanged) { drawLCD(); lastLCDSecond=now.getSeconds(); }
}
```
Need date/week at local. Update date once a day or if date changed; draw on init. But maybe month/day key should include date. Use `dateKey = new Date(now.getFullYear(), now.getMonth(), now.getDate()).getTime()`. Update if different. In animation, compare. Good.

Need LCD update every second. But for smooth? Digital time can update every second. Need maybe update at first millisecond after second changed. Good.

Need dynamic draw:
```
function drawDate() {
 const ctx = dateCanvas.getContext('2d');
 ctx.clearRect(0,0,300,200);
 // background
 ctx.fillStyle = '#fffdf2';
 roundRect(ctx, 0,0,300,200,12); ctx.fill();
 // border
 ctx.strokeStyle='#aaa'; ctx.lineWidth=2; ctx.stroke();
 // text
 const now = new Date();
 const weekdays = ['周日','周一','周二','周三','周四','周五','周六'];
 ctx.fillStyle='#222';
 ctx.textAlign='center'; ctx.textBaseline='middle';
 ctx.font='bold 30px "Microsoft YaHei","PingFang SC",sans-serif';
 ctx.fillText(weekdays[now.getDay()], 150, 45);
 ctx.font='bold 50px "Microsoft YaHei","PingFang SC",sans-serif';
 ctx.fillText(`${now.getMonth()+1}月${now.getDate()}日`, 150, 115);
 dateTexture.needsUpdate = true;
}
```
Need date window at 3. The date plane static frame is behind. Dynamic plane at z=0.365; perhaps at angle, the date texture might be occluded by hands.

LCD draw:
Need create helper for rounded rect:
```
function roundRect(ctx,x,y,w,h,r) { ctx.beginPath(); ctx.moveTo(x+r,y); ...; ctx.closePath(); }
```
Need careful with context. Use `ctx.roundRect`? Not supported in old; implement.

`drawLCD`:
```
const ctx = lcdCanvas.getContext('2d');
ctx.clearRect(0,0,640,280);
// LCD panel
roundRect(ctx,0,0,640,280,18); ctx.fillStyle = '#0b1f16'; ctx.fill();
ctx.strokeStyle = '#2a4a38'; ctx.lineWidth=4; ctx.stroke();
const now = new Date();
const tz = tzSelect.value;
const fmtTime = new Intl.DateTimeFormat('en-GB',{timeZone:tz, hour:'2-digit', minute:'2-digit', second:'2-digit', hourCycle:'h23'});
const parts = fmtTime.formatToParts(now); let h=...;
const timeStr = `${h}:${m}:${s}`; // if hourCycle with 24, h may be "24"? With hourCycle h23 should not. Use hour12:false could return 24 for midnight in some browsers. Use hourCycle:'h23'.
// date in that timezone
const dateParts = new Intl.DateTimeFormat('en-GB', {timeZone:tz, year:'numeric', month:'2-digit', day:'2-digit', weekday:'short'}).formatToParts(now);
let y,m,d,weekday; ...
// tz abbreviation
const abbrParts = new Intl.DateTimeFormat('en-US', {timeZone:tz, timeZoneName:'short'}).formatToParts(now);
let abbr = ... or tz.split('/').pop();
```
Need `timeZoneName:'short'` can return "GMT+8" or "EDT". Good.
Draw:
```
ctx.fillStyle = '#8dffb0';
ctx.textAlign='left'; ctx.textBaseline='middle';
ctx.font='bold 40px "Courier New", monospace';
ctx.fillText(abbr, 40, 45);
ctx.font='bold 100px "Courier New", monospace';
const timeX = (640 - ctx.measureText(timeStr).width)/2;
ctx.fillText(timeStr, timeX, 150);
ctx.font='bold 32px "Courier New", monospace';
ctx.textAlign='center';
ctx.fillText(`${y}/${m}/${d} ${weekday.toUpperCase()}`, 320, 240);
lcdTexture.needsUpdate = true;
```
Need if `weekday` from en-GB maybe "Thu"; uppercase. Good.
Need perhaps second timezone date and weekday in selected timezone, not local. This is additional date display. Nice.

Need "local time" bottom overlays:
```
function updateOverlay() {
 const now = new Date();
 const localStr = now.toLocaleString('zh-CN', {hour12:false});
 document.getElementById('local').textContent = localStr;
}
```
But this creates string. Could update every second. Need maybe not necessary.

Need animation:
```
function animate() {
 requestAnimationFrame(animate);
 const now = new Date();
 const ms = now.getMilliseconds();
 const s = now.getSeconds() + ms/1000;
 const m = now.getMinutes() + s/60;
 const h = (now.getHours() % 12) + m/60;
 hourHand.rotation.z = - (h/12) * Math.PI * 2;
 minuteHand.rotation.z = - (m/60) * Math.PI * 2;
 secondHand.rotation.z = - (s/60) * Math.PI * 2;
 updateDynamicTextures();
 updateLocalOverlay();
 controls.update();
 renderer.render(scene,camera);
}
```
Need if hourHand is object3D not mesh? We can rotate mesh. Good.
Need if watchGroup maybe rotate? no.

Need maybe second hand smooth with real time. Use `now.getMilliseconds()` in seconds, so smooth.

Need maybe if system clock has timezone? okay.

Need maybe set all rotation from 12 o'clock. Need our hand shapes point +Y, rotation.z negative. Good.

Potential issue: The shape coordinate y=0 at pivot but shape extends +Y. The extrude geometry center z=0. The mesh rotation.z rotates around pivot (0,0). Good.
Need if we create shape with negative y tail, rotation also rotates tail. Fine.

Need perhaps add hour and minute hand colors. Fine.

Need maybe add "GMT" for second timezone? No.

Need maybe create complete 3D watch with "watchGroup" orient? Currently watch face faces +Z, strap behind -Z. The camera default position (5,4,8) looks at origin, so sees face and side. Good.

Need perhaps "complete 3D watch" needs a wrist? We can add a stylized wrist cylinder behind. Could be nice but maybe overcomplicates. Could add a "wrist" cylinder aligned along X behind the watch? Let's see. The strap curve already suggests wrist. A thick cylinder along X of radius 2.2, length 10, centered at z=-2, maybe passes behind the strap. It would show a wrist form. But if wrist cylinder is large and intersects strap, might look weird. Let's add a subtle arm/wrist under the strap? Hmm, user asked wristwatch animation; a watch floating in space with strap is enough. But "wristwatch" means watch with band, not wrist. We can skip actual wrist.

Need maybe add "floor" with soft shadow? Could add a circular platform? But not needed.

Need perhaps if `THREE.OrbitControls` from CDN doesn't load? Need ensure. Let's use cdnjs for three r128 and cdn.jsdelivr for OrbitControls. URL:
`https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`
`https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`
Should work. But in newer three r128 examples/js controls uses `THREE.OrbitControls` global. Good.

Need maybe `THREE.MeshPhysicalMaterial` with `transmission`? We won't use.

Need maybe use `THREE.sRGBEncoding` with CanvasTexture? Fine.

Need maybe if there is CSP? no.

Need maybe "complete code" includes external CDN dependencies. Need mention internet required.

Need perhaps include fallback no module. Good.

Let's craft code carefully.

Potential issue: `new THREE.CylinderGeometry(3, 3, 0.7, 64, 1, true)` open-ended. Need set `side = THREE.DoubleSide`? Since side tube visible from outside; open-ended has normals outward, fine. But after rotation x=π/2, face normal? Cylinder side surface normals are perpendicular to axis. Without side set, default FrontSide. Rotation no issue. Need if open cylinder, camera from front sees inner? Not usually. We can leave FrontSide; but if rotating behind, inner back side maybe hidden? The tube from front? Actually camera sees outer surface from outside, okay. Back cap is visible from outside. Good.

Dial circle at z=0.36: `CircleGeometry` front side normal +Z. Camera at z positive sees front. Good. Back at z=-0.35 rotated y=π, normal -Z, visible from back. Fine.

Need maybe case side height 0.7 with dial at 0.36 and back at -0.35. There is gap? Dial at z=0.36 slightly outside side edge 0.35, so dial disc sits in front of side. It may cover side. Good. Back at z=-0.35 exactly. No gap.

Need date and LCD planes z=0.36? Let's set dial z=0.36. Dynamic planes at z=0.365? The difference 0.005 might z-fight if camera far? Actually 0.005 at scale 3 can be significant near; but with depth buffer precision maybe at distance 10, near plane 0.1, far100, depth precision enough? 0.005 maybe okay but could flicker. Better set dynamic planes z=0.38, with a visible gap. But then hands at z=0.44, still above. Use 0.37. Let's set dial z=0.36, datePlane z=0.38, lcdPlane z=0.38. But date/LCD appear above dial by 0.02. Fine. Hands at 0.44. Need if dynamic planes drawn over dial. Good.
Need if datePlane at z=0.38 and dial at z=0.36, the date window frame on dial behind; dynamic plane covers an area with some offset; border visible from slow angles due to parallax. Since planes parallel and separated 0.02, at far view parallax negligible. But maybe okay. Use 0.37 maybe. Need no z-fight; 0.01 enough. Let's set 0.37.

Need if second hand at z=0.49, minute 0.47, hour 0.45. Center pin at z=0.48. Good.

Need maybe set hands' `renderOrder` after planes? Opaque; no issue.

Need if dynamic textures update every frame? Need `texture.needsUpdate = true` only when draw. Good.

Need create `drawDate` and `drawLCD` before build? Need dateCanvas and lcdCanvas exist. We'll create in buildWatch? Actually buildWatch creates datePlane, lcdPlane. We need callback references. We can create dateCanvas global before update. Let's arrange:
```
let dateCanvas, lcdCanvas, dateTexture, lcdTexture;
function buildWatch() {
  ...
  const dateObj = createDatePlane(); dateCanvas = dateObj.canvas; dateTexture=dateObj.texture;
  const lcdObj = createLCDPlane(); lcdCanvas = lcdObj.canvas; lcdTexture = lcdObj.texture;
  drawDate(); drawLCD();
}
```
Need function declarations hoisted.

Need create dynamic plane geometry sizes and positions.

Let's choose coordinates carefully:
- Dial radius 3 world.
- Date window center at (1.55, 0.5) is near 3, inside radius: sqrt(1.55^2+0.5^2)=1.63 <3. Good.
- LCD center at (0,-1.5) inside radius 1.5. Size width 2.4, height 1.05; half width 1.2, bottom y=-1.5-0.525=-2.025, within radius? sqrt(0^2+2.025^2)=2.025 <3. Good. But the LCD plane as a rectangle extends x from -1.2 to 1.2; corners at (-1.2,-2.025) radius 2.35 <3. Good. It won't cross bezel. It may overlap date window? date y=0.5 to1.0? no.

- Date plane position y=0.5, half height 0.5 -> y from0 to1.0. LCD top y=-0.975 (if center -1.5, height1.05 -> top -0.975). Gap 0.975. Good.
- 3 o'clock marker at radius 2.8? Wait dial markers at r=2.6? If date window at x=1.55, okay. Need not draw 3 marker behind date? We can draw all markers; date plane covers area, marker at 3 (x=2.8,y=0) outside date plane because date plane x from0.8 to2.3; marker x=2.8 outside. Good.

Need static dial labels:
- Markers: outer radius 2.65? World -> PX: 2.65*170.7=452. Use canvas radius 470? Let's define:
```
const rOuter=470; // close to edge
const rInner = 390;
```
Wait radius world = rOuter/PX = 470/170.7=2.75. Good. Markers from world 2.75 to 2.58. Hour markers maybe longer. Good.
- If CircleGeometry radius 3, texture is applied to circle edge, so dial background extends to radius 3. But the case inner radius is 3. The static dial may fill full circle; markers at 2.75 are inside. Good.

Need maybe dial background circle should be clipped to circle? Canvas fills full square so outer corners covered? CircleGeometry texture maps square to circle; only within circle visible. The canvas corners outside circle not visible. Good. We can fill entire canvas with gradient; circle will show circular portion. Good.

Need draw a circular bezel ring in dial texture? At r=470 maybe stroke. Good.

Need create dial texture:
```
function createDialTexture() {
  const canvas = document.createElement('canvas');
  canvas.width = canvas.height = 1024;
  const ctx = canvas.getContext('2d');
  const CX=512,CY=512;
  const PX=1024/6; // 170.667
  const grad=ctx.createRadialGradient(CX,CY,80,CX,CY,470);
  grad.addColorStop(0,'#f8f6ef'); grad.addColorStop(0.8,'#e8e4d8'); grad.addColorStop(1,'#c9c6bd');
  ctx.fillStyle=grad;
  ctx.beginPath(); ctx.arc(CX,CY,470,0,Math.PI*2); ctx.fill();
  // outer ring
  ctx.strokeStyle='#333'; ctx.lineWidth=3; ctx.beginPath(); ctx.arc(CX,CY,465,0,Math.PI*2); ctx.stroke();
  // minute markers
  for (let i=0; i<60; i++) {
    const angle = (i/60)*Math.PI*2 - Math.PI/2;
    const outer = i%5===0 ? 445 : 455;
    const inner = i%5===0 ? 410 : 435;
    ctx.strokeStyle = i%5===0 ? '#111' : '#444';
    ctx.lineWidth = i%5===0 ? 5 : 2;
    ctx.beginPath();
    ctx.moveTo(CX+outer*Math.cos(angle), CY+outer*Math.sin(angle));
    ctx.lineTo(CX+inner*Math.cos(angle), CY+inner*Math.sin(angle));
    ctx.stroke();
  }
  // hour numerals
  ctx.fillStyle='#1a1a1a';
  ctx.font='bold 66px Georgia, "Times New Roman", serif';
  ctx.textAlign='center'; ctx.textBaseline='middle';
  for (let i=1; i<=12; i++) {
    const angle = (i/12)*Math.PI*2 - Math.PI/2;
    const r=340;
    ctx.fillText(i.toString(), CX+r*Math.cos(angle), CY+r*Math.sin(angle));
  }
  // brand
  ctx.font='700 42px "Segoe UI", Arial, sans-serif';
  ctx.fillStyle='#3a3a3a';
  ctx.fillText('TZ · DUAL', CX, 220);
  ctx.font='300 32px "Segoe UI", Arial, sans-serif';
  ctx.fillText('AUTOMATIC', CX, 282);
  // maybe date window frame
  const dw = 1.7*PX, dh=1.2*PX; // width,height px
  const dx=CX+1.55*PX - dw/2, dy=CY-0.5*PX - dh/2;
  ctx.strokeStyle='#222'; ctx.lineWidth=3;
  ctx.strokeRect(dx,dy,dw,dh);
  // "DUAL TIME" label above LCD
  ctx.font='600 30px "Segoe UI", Arial, sans-serif';
  ctx.fillStyle='#555';
  ctx.fillText('DUAL TIME', CX, CY - 0.75*PX); // world y 0.75? actually CY - (-0.75? Wait LCD center at y=-1.5, label at y=-0.7? Need canvas y = CY - worldY*PX. For worldY=-0.6, canvas y=512+102=614. Text baseline middle. LCD top at world y=-0.975; label at -0.6 is above LCD. Fine.
```
Need `ctx.strokeRect` frame for date. But dynamic date plane at (1.55,0.5), size 1.5x1.0. Frame size 1.7x1.2. The frame center exact. Good.
Need maybe draw "TZ · DUAL" at world y = (512-220)/PX? 512-220=292/170.7=1.71. So text at y=1.71, below 12? Wait canvas y=220 corresponds world Y = (CY - y)/PX = (512-220)/170.7 = 1.71. That's near 12? 12 o'clock is world y=2.8. Good. "AUTOMATIC" at canvas y=282 -> world y=1.35. Fine.

Need if date window at world y=0.5, canvas y=427. Frame height 1.2*PX=205, from y=324 to529. Wait center y=427, half=102 -> 325-529. This is below center. Fine. It may overlap numeral 3 at r=340: 3 o'clock canvas y=512; x=852. Date frame center (776,427), width 290, x=631-921, y=325-529. It overlaps the numeral 3 (at x=852,y=512) border at y=529, okay. Date plane covers y=342-512? Actually dynamic plane center 427, half 85 -> 342-512. It might cover lower half of numeral 3. Hmm date plane x=631-921, y=342-512; numeral 3 at x=852,y=512; its middle y=512. It might just touch. Fine. Maybe move date window to y=0.7? Then y canvas=392, plane y=307-477, not cover numeral 3 at y=512. Center (1.6,0.7), radius sqrt(2.56+0.49)=1.75, okay. Let's place date window at (1.55,0.65). Date plane y=0.15..1.15. More natural. Static frame world y center 0.65. Canvas y=512-111=401. Plane y=316-486. Numeral 3 center y=512. Good.
Need hands can cover date window at 3? The hour/minute will pass over. Fine.
Let's set date position `(1.55, 0.65)`.

LCD center maybe `(0,-1.55)`. Label above at world y=-0.75? Wait canvas y for world -0.75 = 512+128=640. LCD center y=-1.55 -> canvas y=777. Top of LCD (height1.05) y=777-90=687. So label at canvas y=640, above. Good. Use `ctx.fillText('DUAL TIME', CX, CY - (-0.75)*PX?)` Actually canvas y = CY - worldY*PX. If worldY=-0.75, canvas y=512+128=640. Code `ctx.fillText('DUAL TIME', CX, CY - (-0.75) * PX)` which is 640. Fine.

Need if "DUAL TIME" label at world y=-0.75, and LCD top at -1.025? If LCD center -1.55, height1.05 -> top -2.075? Wait center y=-1.55, half height 0.525 -> top=-1.025, bottom=-2.075. Ah I miscalc: if center=-1.55, top=-1.025 (closer to center), bottom=-2.075. So label at y=-0.75 is above top by 0.275, okay. Good.

Need update dynamic LCD: if lcd plane center y=-1.55, its top at -1.025. It won't reach center; good. But LCD is quite low. It might be hard to see but okay. Need maybe make bigger? width 2.4, height1.05, bottom y=-2.075, within radius 3. Good. Good.

Need date plane center at y=0.65, top=1.15, bottom=0.15. Fine.

Need maybe static frame for date use center (1.55,0.65), size 1.7x1.2. Good.

Need perhaps date plane and lcd plane maybe not exactly fit with frame? Need dynamic planes no border and no background? They fill canvas. Need updateDate canvas 300x200 has white background, but not transparent. If no border, static frame visible around. Since plane is smaller than frame by 0.1 each side (frame width 1.7 vs plane 1.5, diff 0.1 per side?). Static frame size 1.7x1.2; plane size 1.5x1.0; border margin 0.1 world. That's 17 px at canvas, visible. Good. Need if dynamic plane background white fills entire plane, leaving static frame border. Nice.

Need dynamic LCD maybe large background dark, no static frame. Fine.

Need drawDate displays "5月14日" maybe dynamic. Good.

Need create date frame static at dx = CX + 1.55*PX - (1.7*PX)/2; dy = CY - 0.65*PX - (1.2*PX)/2. Let's write:
```
const windowCenterX = CX + 1.55*PX;
const windowCenterY = CY - 0.65*PX;
const windowW = 1.7*PX, windowH = 1.2*PX;
ctx.strokeRect(windowCenterX - windowW/2, windowCenterY - windowH/2, windowW, windowH);
```
Need if `PX` is not integer but okay.

Need maybe draw "TZ · DUAL" and "AUTOMATIC"; no.

Need maybe draw static "LOCAL" under center? Could draw "LOCAL" near center? Maybe not.

Need if second hand smooth and dateTexture not update every second, but if dateTexture's canvas clear? okay.

Need perhaps renderer antialias. Good.

Need maybe add `watchGroup.rotation.z`? No.

Need maybe if user rotates, the second hand appears to rotate within plane. Good.

Potential issue: In Three.js, `CircleGeometry` UV mapping: I think CircleGeometry uses `uv` coordinate based on position from center: `(vertex.x / radius + 1) / 2`, y similar. Thus texture maps center to 0.5, circle radius to 1. But if radius=3, x=3 maps to 1. So PX=1024/(2*3). Good. But wait circle geometry vertices at x,y=radius*cos, radius*sin, uv = (x/radius+1)/2? If radius=3, x=3 -> (1+1)/2=1. yes. Good.

Need dynamic plane `CanvasTexture` mapping: PlaneGeometry UV spans 0-1; texture width/height maps. Plane size 1.5x1.0; canvas ratio 1.5; okay.

Need maybe if CanvasTexture with text has transparent? We draw background opaque. Fine.

Need "smooth" second hand. Use constant rotation per frame. Good.

Need perhaps use `now.getMilliseconds()` but requestAnimationFrame frame time. okay.

Need perhaps update overlay bottom with local time and second timezone. Need avoid updating every frame/string too heavy; update every second:
```
let lastOverlaySecond = -1;
function updateLocalOverlay() {
  const now = new Date();
  if (now.getSeconds() === lastOverlaySecond) return;
  lastOverlaySecond = now.getSeconds();
  const local = now.toLocaleString('zh-CN', {hour12:false});
  const tz = tzSelect.value;
  const other = new Intl.DateTimeFormat('zh-CN', {timeZone: tz, hour:'2-digit', minute:'2-digit', second:'2-digit', hour12:false}).format(now);
  document.getElementById('local').textContent = `本地时间：${local}  |  第二时区 ${tz}：${other}`;
}
```
This also updates LCD maybe every sec. Fine. Need if `toLocaleString` includes date. Good.

Need maybe HTML select options:
```
<select id="tzSelect">
<option value="UTC">UTC</option>
<option value="Asia/Shanghai">上海</option>
<option value="Asia/Tokyo">东京</option>
<option value="Europe/London">伦敦</option>
<option value="America/New_York">纽约</option>
<option value="Australia/Sydney">悉尼</option>
</select>
```
Default UTC. Good.

Need maybe if selected timezone equal local? fine.

Need maybe if computer local timezone is e.g. UTC+8 and default UTC, LCD shows UTC. Good.

Need maybe include "第二时区" date? yes.

Need maybe "complete 3D wristwatch" with "3D" in code.

Let's write code fully. Need be careful with functions.

Potential issue: `THREE.TubeGeometry(curve, 64, 0.75, 16, false)` with curve points start/end at z=0.35 and y=±3.45. The tube radius 0.75 might protrude into the case side and be visible in front. At y=±3.45,z=0.35, the tube center is at case edge, radius 0.75 extends into y=2.7..4.2 and z=-0.4..1.1. It may intersect the case side at y=3? Actually lugs at y=3.2, but strap center at y=3.45. It is somewhat thick. Maybe reduce radius to 0.55. Strap width 1.1. Use radius 0.55. Tube begins at y=3.45? Need if start at y=3.2 maybe. Let's set lugs at y=3.3 and strap curve start/end y=3.2? Use:
```
const points = [
 (0,3.2,0.3),
 (0,1.8,-0.5),
 (0,0,-1.4),
 (0,-1.8,-0.5),
 (0,-3.2,0.3)
];
```
Tube radius 0.6. It connects behind.
Lugs positions at y=3.2 and -3.2. Use boxes at (0,3.2,0.1). Good. The strap starts at y=3.2,z=0.3, slightly in front of lug? case z=0.35. okay.
Need perhaps strap visible from front? The tube radius 0.6, at y=3.2, z=0.3; it may appear near side, but not cover dial.

Need maybe adjust initial camera to see strap. Camera pos (6,4.5,9) looks at origin. It can see side and top lugs. Great.

Need maybe "lugs" boxes: Use `BoxGeometry(1.2, 0.5, 0.5)`; position (0,±3.2,0). But case radius=3, so lug center y=3.2, extends from y=2.95 to3.45, connected to case. Good. Add z position -0.1? Actually case side z from -0.35 to0.35; lug can be z=-0.1. Use:
```
const lugGeo = new THREE.BoxGeometry(1.2,0.55,0.6);
lugTop.position.set(0,3.2,0.0);
```
It connects to case at y=2.9? case radius 3, so overlap. Good.

Need perhaps crown at x=3.3. Fine.

Need maybe create watchGroup includes caseGroup, strap, lugs, hands, planes. The lugs can be part of caseGroup. The strap separate. The watchGroup is added to scene.

Need perhaps `watchGroup.rotation.set(0,0,0)` no.

Potential issue: In Material for `MeshStandardMaterial`, if map with `CanvasTexture` and `color=white`. Fine.

Need maybe `renderer.outputEncoding = THREE.sRGBEncoding` with `map`? It will brighten. okay.

Need perhaps if using MeshBasicMaterial for dynamic planes, `color` white. okay.

Need perhaps if `MeshBasicMaterial` with map and transparent true, but date/lcd background opaque, still transparent render queue. It might show through? No issue.

Need maybe one bug: `new THREE.CylinderGeometry(3, 3, 0.7, 64, 1, true)` open-ended side material with MeshStandardMaterial, no backside. If camera looks from front at an angle, the side tube interior might be hidden? Outer side visible. Good.

Need maybe `CircleGeometry(3,64)` front side only. Back cap rotated y=π, front side? When rotate y=π, the normal originally +Z becomes -Z, so it faces camera from back. Good.

Need maybe dial circle at z=0.36, front side normal +Z, visible. Good.

Need maybe Watch face should be facing camera but camera at z positive. Good.

Need maybe if the watch is "right-handed", crown on right. yes.

Need maybe if `OrbitControls` imported from examples/js after three, uses `THREE`. Good.

Potential issue: `three@0.128.0/examples/js/controls/OrbitControls.js` may be in UMD and expects global THREE. Should work. But if using CDN with "three@0.128.0", path is `examples/js/controls/OrbitControls.js`. I think yes. But to avoid dependency, we could implement a simple pointer orbit ourselves. But OrbitControls is common. Let's use CDN and mention internet. Need maybe if cdnjs and jsdelivr both unavailable? no.

Need maybe include version with `r128` but `THREE.OrbitControls` might not exist if file not loaded. Could include `if (THREE.OrbitControls) controls = new THREE.OrbitControls(...)` else no? But need controls. Could implement fallback manual? Maybe not.

Let's verify OrbitControls CDN URL:
`https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js` indeed has `THREE.OrbitControls`. Good.

Need maybe "complete" code includes `<script src>` from cdnjs three r128 and jsdelivr. Good.

Need perhaps in final answer include note "请在有网络环境打开". Good.

Need maybe "test instructions":
1. Open HTML in browser (Chrome/Edge/Firefox).
2. Observe watch hands synchronize with system time, seconds hand moves smoothly.
3. Date/week window at 3 o'clock shows local date and weekday; check with system.
4. LCD at 6 o'clock shows UTC time/date (or selected timezone); use dropdown to change second timezone, label and time update.
5. Drag with mouse to rotate 3D, scroll zoom.

Need perhaps mention if "秒针" continuous, not jumping. Good.

Need maybe "current time" from local system; if system time changes, watch changes.

Need maybe "Full code" in final. Need perhaps include code block with HTML. Good.

Need maybe we should include "three.js CDN" but no package. Good.

Let's refine code.

Detailed code skeleton:

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<title>3D 腕表动画</title>
<style>
* { box-sizing:border-box; }
body { margin:0; overflow:hidden; background: radial-gradient(circle at 50% 35%, #252a3a 0%, #05070d 80%); font-family: system-ui, sans-serif; }
#controls { position:absolute; top:18px; right:18px; z-index:10; background:rgba(15,17,26,.75); border:1px solid rgba(255,255,255,.12); border-radius:14px; padding:12px 16px; color:#fff; font-size:14px; backdrop-filter: blur(6px); }
#controls select { margin-left:8px; padding:4px 8px; border-radius:8px; border:1px solid #3a3f55; background:#1b1f2e; color:#fff; font-size:13px; outline:none; cursor:pointer; }
#info { position:absolute; top:20px; left:50%; transform:translateX(-50%); z-index:10; color:#cdd3e5; background:rgba(15,17,26,.55); border:1px solid rgba(255,255,255,.08); padding:6px 16px; border-radius:999px; font-size:13px; pointer-events:none; }
#local { position:absolute; bottom:18px; left:50%; transform:translateX(-50%); z-index:10; color:#aab3ce; background:rgba(15,17,26,.55); border:1px solid rgba(255,255,255,.08); padding:8px 18px; border-radius:999px; font-size:13px; text-align:center; white-space:nowrap; }
</style>
</head>
<body>
...
```

Script:
```
'use strict';
// Global state
let scene, camera, renderer, controls, watchGroup;
let hourHand, minuteHand, secondHand;
let dateCanvas, lcdCanvas, dateTexture, lcdTexture;
let datePlane, lcdPlane;
let lastDateKey = '';
let lastOverlaySecond = -1;
const tzSelect = document.getElementById('tzSelect');

function init() { ... }
function buildWatch() { ... }
...
init();
```

Need if `init()` called before DOM loaded? Script at end body, so okay.

Need Windows resize:
```
function onWindowResize() { camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); }
```

Need `roundRect` helper:
```
function roundRect(ctx, x, y, w, h, r) {
 r = Math.min(r, w/2, h/2);
 ctx.beginPath();
 ctx.moveTo(x+r, y);
 ctx.lineTo(x+w-r, y);
 ctx.quadraticCurveTo(x+w, y, x+w, y+r);
 ctx.lineTo(x+w, y+h-r);
 ctx.quadraticCurveTo(x+w, y+h, x+w-r, y+h);
 ctx.lineTo(x+r, y+h);
 ctx.quadraticCurveTo(x, y+h, x, y+h-r);
 ctx.lineTo(x, y+r);
 ctx.quadraticCurveTo(x, y, x+r, y);
 ctx.closePath();
}
```
Need for `ctx.fill()` after. Good.

Need `createDialTexture`:
Implement with `PX=1024/6`. Need use `ctx.textBaseline='middle'`. For numbers, "12" at top. Good.

Potential issue: `ctx.textAlign='center'` and `textBaseline='middle'` for numbers. At y coordinate adjusted? Because font middle in canvas: Using `textBaseline='middle'` exactly centers around y. Good.

Need "dat window frame" static drawn after labels? If draw date frame after numerals, it's okay. Dynamic plane will cover inner area. Static frame visible. Need if static frame line extends over numeral 3? no.

Need perhaps draw small rectangular outline for date window at world coordinates. Let's compute:
```
function worldToCanvasX(wx){ return CX + wx*PX; }
function worldToCanvasY(wy){ return CY - wy*PX; }
```
Can define inside function. Use for label.

Need "createDialTexture" code:
```
const grad = ctx.createRadialGradient(CX, CY, 80, CX, CY, 470);
...
ctx.fillStyle = grad;
ctx.beginPath(); ctx.arc(CX,CY,470,0,Math.PI*2); ctx.fill();
...
ctx.strokeStyle='#333'; ctx.lineWidth=4; ctx.beginPath(); ctx.arc(CX,CY,462,0,Math.PI*2); ctx.stroke();
```
Need if gradient stops with alpha? fill circle. Good.

Need minute markers:
```
for (let i=0;i<60;i++) {
 const a = i*Math.PI/30 - Math.PI/2;
 const isHour = i%5===0;
 const outer = isHour ? 438 : 454;
 const inner = isHour ? 405 : 438;
 ctx.strokeStyle = isHour ? '#1a1a1a' : '#666';
 ctx.lineWidth = isHour ? 5 : 2;
 ...
}
```
Need "hour numerals" radius 335. "12" at y=512-335=177. Brand text at y=220; okay.

Need "seconds track" maybe dots? Not necessary.

Need "apply texture":
```
const texture = new THREE.CanvasTexture(canvas);
texture.encoding = THREE.sRGBEncoding;
return texture;
```

Need create hand shapes. Need if ExtrudeGeometry includes shape coordinates with decimal. Good.

Need create hands:
```
function createHand(shape, color, z, depth) {
 const geo = new THREE.ExtrudeGeometry(shape, {depth: depth || 0.02, bevelEnabled:false});
 geo.translate(0,0,-(depth||0.02)/2);
 const mat = new THREE.MeshStandardMaterial({color: color, metalness:0.35, roughness:0.45, side:THREE.DoubleSide});
 const mesh = new THREE.Mesh(geo, mat);
 mesh.position.z = z;
 mesh.castShadow = true;
 watchGroup.add(mesh);
 return mesh;
}
```
Need `side: THREE.DoubleSide` because shape extrude depth thin, BackSide maybe not visible from front? Front side visible, but when viewed from back, hands may disappear. Use DoubleSide. Good.
Need `color` can be `0x222831`.
Second hand red.

Need date/lcd plane create:
```
function createDatePlane() {
 const canvas = document.createElement('canvas');
 canvas.width=300; canvas.height=200;
 const texture = new THREE.CanvasTexture(canvas);
 const mat = new THREE.MeshBasicMaterial({map:texture, transparent:true, depthWrite:false});
 const mesh = new THREE.Mesh(new THREE.PlaneGeometry(1.5,1.0), mat);
 mesh.position.set(1.55,0.65,0.375);
 mesh.renderOrder = 2;
 watchGroup.add(mesh);
 return {canvas, texture, mesh};
}
```
Need maybe `mesh.rotation` none, plane is in XY plane normal +Z. Good.
LCD:
```
const canvas=document.createElement('canvas'); width=640; height=280;
const texture = new THREE.CanvasTexture(canvas);
const mat = new THREE.MeshBasicMaterial({map:texture, transparent:true, depthWrite:false});
const mesh = new THREE.Mesh(new THREE.PlaneGeometry(2.4, 1.05), mat);
mesh.position.set(0,-1.55,0.375);
mesh.renderOrder=3;
```
Need if `PlaneGeometry` in XY plane, texture orientation? PlaneGeometry is in XY plane with normal +Z, same as dial. Good.

Need maybe date/lcd plane not cast shadow. okay.

Need "drawDate" should set `dateTexture.needsUpdate = true;`.
Need "drawLCD" uses `tzSelect.value`. Need if `Intl.DateTimeFormat` with timeZone invalid? no. Need if `hourCycle` not supported in older Safari? It is. To be safe, use `hour12: false` instead of hourCycle? But with `hour12:false`, midnight can be "24" in some locales. Could normalize:
```
if (hour === '24') hour = '00';
```
Let's use `hourCycle:'h23'` for modern and normalize. Need if `hourCycle` in `formatToParts` supported. Good.

Need implement `formatInTimeZone` maybe:
```
function getPartsForTZ(tz, now) {
 const timeParts = new Intl.DateTimeFormat('en-GB', {timeZone: tz, hourCycle:'h23', hour:'2-digit', minute:'2-digit', second:'2-digit'}).formatToParts(now);
 ...
}
```
Need date and tz abbreviation:
```
const dateParts = new Intl.DateTimeFormat('en-GB', {timeZone: tz, year:'numeric', month:'2-digit', day:'2-digit', weekday:'short'}).formatToParts(now);
```
Need if `weekday` returned "Thu"; `toUpperCase`.
Need `abbr`:
```
const abbr = new Intl.DateTimeFormat('en-US', {timeZone: tz, timeZoneName:'short'}).formatToParts(now).find(p=>p.type==='timeZoneName')?.value || tz;
```
But if timeZoneName returns "GMT+8" maybe okay. Need if value "UTC" etc. Use `?.` supported. We can avoid optional chaining for compatibility? Modern browsers okay. But if final code in older, okay. Maybe use:
```
const abbrPart = parts.find(...);
const abbr = abbrPart ? abbrPart.value : tz;
```
Need optional chaining not necessary. We'll use `.find`.

Need `drawLCD`:
```
const ctx = lcdCanvas.getContext('2d');
ctx.clearRect(0,0,640,280);
roundRect(ctx,0,0,640,280,18);
ctx.fillStyle='#0a1a12'; ctx.fill();
ctx.strokeStyle='#2f5a40'; ctx.lineWidth=4; ctx.stroke();
ctx.fillStyle='#8dfcb0';
ctx.textBaseline='middle';
ctx.textAlign='left';
ctx.font='bold 38px "Courier New", monospace';
ctx.fillText(abbr, 30, 44);
const timeStr = `${h}:${m}:${s}`;
ctx.font='bold 104px "Courier New", monospace';
ctx.textAlign='center';
const timeWidth = ctx.measureText(timeStr).width;
ctx.fillText(timeStr, 340, 148);
ctx.font='bold 30px "Courier New", monospace';
ctx.fillText(`${y}/${m}/${d} ${weekday.toUpperCase()}`, 340, 240);
lcdTexture.needsUpdate = true;
```
Need if time string "08:32:45" width ~ 100px at 104 font? Actually monospace 104px char width ~62, 8 chars ~500. Center 340 fits. Good. Could use 92px if too wide. Use `ctx.font='bold 92px "Courier New", monospace'` to be safe. But `textAlign='center'`, it's okay.

Need maybe show "LOCAL" in date window? no.

Need updateDate:
```
function updateDateTexture() {
 const now = new Date();
 const key = `${now.getFullYear()}-${now.getMonth()}-${now.getDate()}-${now.getDay()}`;
 if (key === lastDateKey) return;
 lastDateKey = key;
 const ctx = dateCanvas.getContext('2d');
 ctx.clearRect(0,0,300,200);
 roundRect(ctx,0,0,300,200,10);
 ctx.fillStyle='#fffaf0'; ctx.fill();
 ctx.strokeStyle='#d8d0be'; ctx.lineWidth=2; ctx.stroke();
 ctx.fillStyle='#111';
 ctx.textAlign='center'; ctx.textBaseline='middle';
 ctx.font='bold 28px "Microsoft YaHei","PingFang SC",sans-serif';
 ctx.fillText(weekdays[now.getDay()], 150, 44);
 ctx.font='bold 48px "Microsoft YaHei","PingFang SC",sans-serif';
 ctx.fillText(`${now.getMonth()+1}月${now.getDate()}日`, 150, 112);
 dateTexture.needsUpdate = true;
}
```
Need if `weekdays` global. Good.
Need if date string "5月14日" width maybe 4*48=192, fits.

Need updateLCD every second:
```
let lastLCDSec = -1;
let lcdTZ = '';
function updateLCDTexture() {
 const now = new Date();
 const sec = now.getSeconds();
 if (sec === lastLCDSec && tz === lcdTZ) return;
 lastLCDSec = sec; lcdTZ = tz;
 ...
}
```
Need if minute changes but sec same? lcd shows seconds; if sec same no need. But if timezone changed at same second, tzTZ changed. Good.
Need if sec same but minute changed? impossible if sec same and minute changed? Actually after one minute, seconds same 0, but minute changed. If we use only seconds, at 10:00:00 to 10:01:00, sec same 0, lcdTZ same, update skipped, so LCD would not update minute! Need update key include minute/hour too or update every animation? We should update when seconds OR minute OR hour changes. Better update every second by store `lastLCDTimeKey = `${h}:${m}:${s}:${tz}``. But h,m,s included. Use:
```
const key = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}:${tz}`;
if (key === lastLCDKey) return;
lastLCDKey = key;
```
Then it will update each second. Good. But if timezone changed, key changes. Good.
Need if `now.getHours()` local, not selected timezone; key can be local. It changes every second anyway. Fine.

Need update overlay similarly:
```
const key = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;
if (key===lastOverlayKey) return;
```
Need date included? if date changes while seconds same? At midnight, hour/min/sec maybe 0:00:00 same as previous midnight? But previous day not same day? Wait date changes only while time crosses midnight; at 23:59:59 to 00:00:00, key changes from 23:59:59 to 0:0:0. fine. If second same but date changed impossible because date changes at 00:00:00 with hour/min/sec also changed. okay.

Need maybe updateDate key includes date and day. Good.

Need animation loop:
```
function animate() {
  requestAnimationFrame(animate);
  const now = new Date();
  const ms = now.getMilliseconds();
  const sec = now.getSeconds() + ms/1000;
  const min = now.getMinutes() + sec/60;
  const hour = (now.getHours() % 12) + min/60;

  secondHand.rotation.z = -(sec / 60) * Math.PI * 2;
  minuteHand.rotation.z = -(min / 60) * Math.PI * 2;
  hourHand.rotation.z = -(hour / 12) * Math.PI * 2;

  updateDynamicTextures();
  updateLocalOverlay();
  controls.update();
  renderer.render(scene, camera);
}
```
Need if `updateLCDTexture` called every frame but returns if key same. It will compute key every frame, etc. Fine.

Need perhaps `updateDynamicTextures`:
```
function updateDynamicTextures() {
 updateDateTexture();
 updateLCDTexture();
}
```
Good.

Need if `tzSelect` change event should call `drawLCD` immediately (next animation anyway). Add:
```
tzSelect.addEventListener('change', () => { lastLCDKey = ''; updateLCDTexture(); });
```
Good.

Need perhaps `lastLCDKey` global initialized ''. But if `tzSelect` options not loaded? yes.

Need perhaps set `document.querySelector('#local').textContent` no.

Need maybe add "current time in title" no.

Need maybe "correct current time" with system time. Good.

Need maybe add "test by manually changing OS timezone" no.

Potential issue: CanvasTexture with `MeshBasicMaterial` and `transparent:true` will be sorted after opaque hands. But the date/LCD planes are behind hands, depthTest true. However transparent objects in Three.js are sorted by distance from camera; if a transparent plane is behind an opaque hand, it renders after the opaque hand but depthTest fails where hand is closer, so hand remains. Good.

Need maybe if the LCD plane is partially behind the hands but also if camera from back, the date/lcd planes face away and won't be visible. That's okay.

Need perhaps "watch should run according to current time" maybe if system is slow. okay.

Need maybe if "second hand smooth" we use millisecond, but `now` is captured once per frame. Good.

Need perhaps "two timezones" should include "simultaneously tracking" via LCD. Good.

Need perhaps "日期和星期显示" date window displays both. Good.

Need perhaps "完整的 3D 腕表" includes case, bezel, crown, lugs, strap, hands. Good.

Need maybe add "local timezone" label on dial? We have "TZ·DUAL" brand. The main analog is local. Could add a small static text "LOCAL" near center? Maybe no.

Need maybe if user selects timezone, the LCD displays date/week in that timezone. That is more than second time. Good.

Need maybe if local timezone differs from selected, date can differ. Good.

Need maybe if the selected timezone is UTC and user in China, LCD date may differ near midnight. Fine.

Potential issue: `Intl.DateTimeFormat` `timeZoneName:'short'` with `formatToParts` can return "GMT+8" or "UTC". For line1, "GMT+8" might be too long at 38px in 640 width, okay. But maybe "China Standard Time" if not short? `timeZoneName:'short'` returns "CST" for US? Actually for Asia/Shanghai returns "GMT+8" maybe. Good. If `timeZoneName:'long'` could be long. We use short.

Need maybe if `formatToParts` not supported? It is. Good.

Need maybe if `tzSelect.value` at load maybe 'UTC'. Good.

Need maybe if browser autoplay? no.

Need maybe if WebGL not supported? no.

Need maybe include "运行方式" in final outside code. The final answer should not be too long but enough. Need include code block. Since "desired oververbosity 5", final can be moderately detailed. We'll include complete code and instructions. But code itself is large. Need make sure final answer doesn't omit parts.

Let's write final code in final. But before final, let's simulate code for mistakes.

Let's draft code mentally:

```
<script>
'use strict';

let scene, camera, renderer, controls, watchGroup;
let hourHand, minuteHand, secondHand;
let dateCanvas, lcdCanvas, dateTexture, lcdTexture;
let lastDateKey = '', lastLCDKey = '', lastOverlayKey = '';

const tzSelect = document.getElementById('tzSelect');
const weekdays = ['周日','周一','周二','周三','周四','周五','周六'];

function init() {
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.outputEncoding = THREE.sRGBEncoding;
  document.body.appendChild(renderer.domElement);

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.set(5.5, 4.5, 9);
  camera.lookAt(0,0,0);

  controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.08;
  controls.minDistance = 4;
  controls.maxDistance = 18;
  controls.target.set(0,0,0);

  scene.add(new THREE.AmbientLight(0xffffff, 0.55));
  const dirLight = new THREE.DirectionalLight(0xffffff, 1.15);
  dirLight.position.set(3,5,7);
  scene.add(dirLight);
  const fillLight = new THREE.DirectionalLight(0xffffff, 0.35);
  fillLight.position.set(-4,-3,4);
  scene.add(fillLight);
  const rimLight = new THREE.PointLight(0xffeedd, 0.25);
  rimLight.position.set(-2,4,-3);
  scene.add(rimLight);

  buildWatch();

  window.addEventListener('resize', onResize);
  tzSelect.addEventListener('change', () => { lastLCDKey=''; drawLCD(); });
  animate();
}
```
Need if `THREE.sRGBEncoding` in r128? yes.

Need `onResize`:
```
function onResize() {
 camera.aspect = window.innerWidth / window.innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(window.innerWidth, window.innerHeight);
}
```

Need `buildWatch`:
```
function buildWatch() {
 watchGroup = new THREE.Group();
 scene.add(watchGroup);

 const caseMat = new THREE.MeshStandardMaterial({color:0x4a5668, metalness:0.85, roughness:0.35});
 const darkMat = new THREE.MeshStandardMaterial({color:0x222831, metalness:0.5, roughness:0.5});

 // case side
 const side = new THREE.Mesh(new THREE.CylinderGeometry(3,3,0.7,64,1,true), caseMat);
 side.rotation.x = Math.PI / 2;
 side.castShadow = true;
 watchGroup.add(side);

 // case back
 const back = new THREE.Mesh(new THREE.CircleGeometry(3,64), caseMat);
 back.rotation.y = Math.PI;
 back.position.z = -0.35;
 watchGroup.add(back);

 // dial
 const dialTexture = createDialTexture();
 const dial = new THREE.Mesh(new THREE.CircleGeometry(3,64), new THREE.MeshStandardMaterial({map: dialTexture, roughness:0.55, metalness:0.1}));
 dial.position.z = 0.36;
 watchGroup.add(dial);

 // bezel ring
 const bezel = new THREE.Mesh(new THREE.TorusGeometry(3,0.13,16,64), caseMat);
 bezel.position.z = 0.36;
 watchGroup.add(bezel);

 // lugs
 const lugGeo = new THREE.BoxGeometry(1.1,0.6,0.55);
 const lugTop = new THREE.Mesh(lugGeo, caseMat);
 lugTop.position.set(0,3.2,0.0);
 watchGroup.add(lugTop);
 const lugBottom = new THREE.Mesh(lugGeo, caseMat);
 lugBottom.position.set(0,-3.2,0.0);
 watchGroup.add(lugBottom);

 // crown
 const crown = new THREE.Mesh(new THREE.CylinderGeometry(0.3,0.3,0.65,16), caseMat);
 crown.rotation.z = Math.PI / 2;
 crown.position.set(3.35,0,0.1);
 watchGroup.add(crown);

 // strap
 const strapCurve = new THREE.CatmullRomCurve3([
   new THREE.Vector3(0,3.2,0.3),
   new THREE.Vector3(0,1.8,-0.55),
   new THREE.Vector3(0,0,-1.45),
   new THREE.Vector3(0,-1.8,-0.55),
   new THREE.Vector3(0,-3.2,0.3)
 ]);
 const strap = new THREE.Mesh(new THREE.TubeGeometry(strapCurve,64,0.6,16,false), new THREE.MeshStandardMaterial({color:0x1d1f27, roughness:0.9, metalness:0.1}));
 strap.castShadow = true;
 watchGroup.add(strap);
```
Need lugs/strap might z-fight? no.

Need if `side` open-ended and `bezel` torus around radius 3 at z=0.36 might intersect side at edge. fine.

Need if back cap at z=-0.35 and cylinder side from -0.35 to0.35. Good.

Need create date/lcd planes after dial? They should be on dial:
```
 const dp = createDatePlane();
 dateCanvas = dp.canvas; dateTexture=dp.texture;
 const lp = createLCDPlane();
 lcdCanvas = lp.canvas; lcdTexture=lp.texture;
```
Need define functions before build. Good.

Need hands:
```
 const hourShape = new THREE.Shape();
 ...
 hourHand = createHand(hourShape, 0x222831, 0.44, 0.02);
 minuteHand = createHand(minuteShape, 0x222831, 0.46, 0.02);
 secondHand = createHand(secondShape, 0xc81e2b, 0.48, 0.015);
```
Need if `createHand` adds to watchGroup. Good.

Need center cap:
```
 const cap = new THREE.Mesh(new THREE.CylinderGeometry(0.12,0.12,0.12,16), darkMat);
 cap.rotation.x = Math.PI/2;
 cap.position.z=0.48;
 watchGroup.add(cap);
```
Need if center cap over second hand at same z; okay.

Need if cap axis along Z and position z=0.48; it spans z=0.42 to0.54. It may protrude beyond second hand z=0.48? actually second hand extrude depth 0.015 centered at z=0.48 spans 0.4725 to0.4875. Cap spans 0.42 to0.54, so covers. Good.

Need if `createHand` uses `mesh.position.z = z`, and geometry depth centered. Good.

Need maybe if hour/min shapes have tapered point at "quadraticCurveTo" but shape must be closed. Use `closePath()` after lineTo. Good.

Need `createHand`:
```
function createHand(shape, color, z, depth) {
 const geo = new THREE.ExtrudeGeometry(shape, {depth: depth, bevelEnabled: false});
 geo.translate(0,0,-depth/2);
 const mat = new THREE.MeshStandardMaterial({color, metalness:0.35, roughness:0.45, side:THREE.DoubleSide});
 const mesh = new THREE.Mesh(geo, mat);
 mesh.position.z = z;
 mesh.castShadow = true;
 watchGroup.add(mesh);
 return mesh;
}
```
Need use `watchGroup` already defined. Good.

Potential issue: Shape with `quadraticCurveTo(0,1.65,0.07,1.45)` and then lineTo(0.16,1.0) etc. The outline is valid. Fine.

Need createDatePlane and createLCDPlane:
```
function createDatePlane() {
 const canvas = document.createElement('canvas');
 canvas.width = 300; canvas.height=200;
 const texture = new THREE.CanvasTexture(canvas);
 const mesh = new THREE.Mesh(new THREE.PlaneGeometry(1.5,1.0), new THREE.MeshBasicMaterial({map:texture, transparent:true, depthWrite:false}));
 mesh.position.set(1.55,0.65,0.375);
 mesh.renderOrder = 2;
 watchGroup.add(mesh);
 return {canvas, texture};
}
```
Need if MeshBasicMaterial with transparent true but no alpha? It will still draw opaque background and not occlude? Since transparent true, it enters transparent pass; background opaque. It may be sorted by distance; hands are opaque and rendered first; depth test protects. Good.
Need createLCDPlane similar with width 640/height280; geometry 2.4/1.05. Good.

Need `drawLCD` uses `lcdTexture.needsUpdate`. Good.

Need `drawDate` if `dateTexture` undefined? called after create. Good.

Need `createDialTexture` uses `new THREE.CanvasTexture(canvas)` and `texture.encoding = THREE.sRGBEncoding`. Good.

Need maybe `createDialTexture` should return `THREE.CanvasTexture`. Good.

Need perhaps if `MeshStandardMaterial` with `map` CanvasTexture and `transparent`? no.

Need maybe `renderer.outputEncoding = THREE.sRGBEncoding` means color values brighter. okay.

Need maybe if `THREE.CanvasTexture` with `encoding` may need `texture.needsUpdate = true`? CanvasTexture sets. Good.

Need `drawDate`:
```
function drawDate() {
 const now = new Date();
 const key = `${now.getFullYear()}-${now.getMonth()}-${now.getDate()}-${now.getDay()}`;
 if (key === lastDateKey) return;
 lastDateKey = key;
 const ctx = dateCanvas.getContext('2d');
 ...
}
```
Need if `dateCanvas` undefined? no.

Need `drawLCD`:
```
function drawLCD() {
 const now = new Date();
 const tz = tzSelect.value;
 const key = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}:${tz}`;
 if (key === lastLCDKey) return;
 lastLCDKey = key;
 const ctx = lcdCanvas.getContext('2d');
 // time parts
 const timeParts = new Intl.DateTimeFormat('en-GB', {timeZone: tz, hourCycle:'h23', hour:'2-digit', minute:'2-digit', second:'2-digit'}).formatToParts(now);
 let hour='00', minute='00', second='00';
 for (const p of timeParts) {
   if (p.type === 'hour') hour = p.value;
   else if (p.type === 'minute') minute = p.value;
   else if (p.type === 'second') second = p.value;
 }
 if (hour === '24') hour = '00';
 // date
 const dateParts = new Intl.DateTimeFormat('en-GB', {timeZone: tz, year:'numeric', month:'2-digit', day:'2-digit', weekday:'short'}).formatToParts(now);
 let y='', mo='', d='', wd='';
 for (const p of dateParts) {
  if (p.type==='year') y=p.value; else if ... 
 }
 // abbr
 const abbrParts = new Intl.DateTimeFormat('en-US', {timeZone: tz, timeZoneName:'short'}).formatToParts(now);
 const abbrObj = abbrParts.find(p => p.type === 'timeZoneName');
 const abbr = abbrObj ? abbrObj.value : tz;
 // draw
 ctx.clearRect(...)
 ...
 lcdTexture.needsUpdate = true;
}
```
Need if `hourCycle` not accepted with `hour12`? no.

Need `updateOverlay`:
```
function updateOverlay() {
 const now = new Date();
 const key = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;
 if (key === lastOverlayKey) return;
 lastOverlayKey = key;
 const tz = tzSelect.value;
 const local = now.toLocaleString('zh-CN', {hour12:false});
 const other = new Intl.DateTimeFormat('zh-CN', {timeZone: tz, hour:'2-digit', minute:'2-digit', second:'2-digit', hour12:false}).format(now);
 document.getElementById('local').textContent = `本地时间：${local}   |   第二时区 ${tz}：${other}`;
}
```
Need if toLocaleString returns "2026/5/14 14:32:45" maybe. Good.
Need maybe if `hour12:false` returns "24:00:00" for midnight? no.

Need `updateDynamicTextures`:
```
function updateDynamicTextures() {
 drawDate();
 drawLCD();
 updateOverlay();
}
```
Could include overlay.

Need `animate`:
```
function animate() {
 requestAnimationFrame(animate);
 const now = new Date();
 const ms = now.getMilliseconds();
 const sec = now.getSeconds() + ms / 1000;
 const min = now.getMinutes() + sec / 60;
 const hour = (now.getHours() % 12) + min / 60;

 hourHand.rotation.z = - (hour / 12) * Math.PI * 2;
 minuteHand.rotation.z = - (min / 60) * Math.PI * 2;
 secondHand.rotation.z = - (sec / 60) * Math.PI * 2;

 updateDynamicTextures();
 controls.update();
 renderer.render(scene, camera);
}
```
Need if `hourHand` undefined? after init okay.

Need maybe if timezone selection changes, `drawLCD` called immediately; but `updateDynamicTextures` next frame. Good.

Need perhaps if user's browser doesn't support `Intl` timeZone? Modern. fine.

Need maybe if "three.js r128" from cdn has no `THREE.sRGBEncoding`? I think yes. If not, code error. In r128, `THREE.sRGBEncoding` exists. Good.

Potential issue: `new THREE.CircleGeometry(3,64)` might not have UVs that map the entire canvas square, because CircleGeometry positions and UVs are as described? Let's verify Three.js CircleGeometry source: It defines grid with x,y = radius * sin/cos; uv = (vertex.x / radius + 1) / 2, (vertex.y / radius + 1) / 2. yes. Good.

Need perhaps if dial texture static circle only fills radius 3? The CircleGeometry radius 3; if map texture, the texture square includes a radial gradient circle drawn at r=470px. But the texture's visible area is a circle radius 3; at the corners of canvas, not visible. The dial surface itself is full circle; the background gradient from r=80 to470 fills a circle area of radius 470, which maps to radius 2.75 (since 470/170.7=2.75). Wait if texture canvas maps full circle from radius 0 to 3 to pixels 0 to 512? Let's re-evaluate: In UV, texture coordinate goes from u=0 at x=-3 to u=1 at x=3. The circle geometry's outer radius 3 maps to u/v coordinates = 0 or 1 at four cardinal points: at x=3,y=0, u=1, v=0.5. The diagonal corners of the texture square are not on the circle; they are outside the geometry. Thus the entire texture square maps across the circle's bounding square, but only points satisfying (u-0.5)^2+(v-0.5)^2 <= (0.5)^2? Wait with uv = x/(2r)+0.5, y/(2r)+0.5. At outer radius r, x^2+y^2 = r^2 -> ((u-0.5)^2 + (v-0.5)^2) * (2r)^2 = r^2 -> (u-0.5)^2+(v-0.5)^2 = 0.25. So outer circle corresponds to uv circle radius 0.5, inscribed in texture square. Thus texture outside the inscribed circle is not visible. The center pixel (512,512) maps to u=0.5,v=0.5. The outer radius maps to distance 0.5 in UV, i.e. 512 pixels from center. So pixels per world unit = 512 / radius = 512/3 = 170.67. A circle drawn at 470 px in canvas radius maps to world radius 470/170.67=2.75. Good. The canvas corners (0,0) are outside geometry and not visible. Our dial gradient circle at radius 470 is within geometry. The CircleGeometry surface outside that gradient but inside radius 3 (i.e. pixels from 470 to512) will show whatever canvas pixels exist there. We fill entire canvas first with gradient, so okay. The dial edge at radius 3 maps to pixel 512, but we don't draw a circle at 512; the case side covers. Good.

Need date window static frame at center (1.55,0.65) -> pixel = CX + 1.55*170.67 = 512+264.5=776.5; y=512-111=401. Good. The frame size 1.7*170.67=290; centered. Dynamic plane world 1.5 -> 256 px, centered at same. Static frame margin 17 px each side. Good.

Need LCD static label "DUAL TIME" at world y=-0.75 -> pixel y=512+128=640. LCD center top? lcd plane center y=-1.55, height1.05 -> top y=-1.025 -> pixel y=512+175=687. Label at 640 above. Good.

Need maybe date window frame at x=1.55±0.85 from 0.7 to2.4, y=0.65±0.6 from0.05 to1.25. It doesn't overlap LCD. Good.

Need hour markers: The numeral 3 at world y=0, x=2? Wait r=340 px = 1.99 world. Date window center x=1.55, y=0.65; frame extends x=0.7..2.4, y=0.05..1.25. The numeral 3 at (2.0,0) outside frame (y=0 <0.05). Fine.

Need maybe date plane at x=1.55, y=0.65; hand rotation can cover it. okay.

Need maybe if LDC plane bottom at y=-2.075, within dial but close to 6 marker at y=-2.75? The 6 marker at radius 2.75? Actually marker at 6 world y=-2.75, outside LCD bottom y=-2.075. Good.

Need maybe "hour markers" small lines from r=2.55? Wait our marker outer/inner: outer=454 px=2.66, inner=405 px=2.37. So 6 marker at y=-2.66, below LCD bottom -2.075. Good.

Need maybe "strp tube" at center y=0,z=-1.45 may intersect wireframe? no.

Need maybe if watchGroup is at origin, strap at z=-1.45 behind; camera can see it if rotates. Good.

Need maybe if `controls.target` at 0,0,0 but watchGroup origin at center of case, strap extends behind; focus okay. Good.

Need maybe if camera position (5.5,4.5,9) distance sqrt(30+20+81)=11.4, within max18. Good.

Need maybe if "3D" not "animated in 3D" but stationary watch. Orbit controls gives 3D. good.

Need maybe "complete 3D 腕表动画" maybe should include automatic rotation? If no user interaction, watch is static except hands. The prompt says "动画" mainly hands. Could also have watch gently rotate? Maybe add subtle auto-rotation? But if OrbitControls enabled, auto-rotate might annoy. Could set `controls.autoRotate = true; controls.autoRotateSpeed = 1.0;` to slowly rotate around watch, but user can interact. That would make 3D animation more dynamic. Should we enable? Could be nice. But if autoRotate and user drags, it continues. We can set `controls.autoRotate = true;` maybe. Then watch slowly rotates, demonstrating 3D. Need perhaps if user interacts, damping. But autoRotate can be disabled by interaction? OrbitControls autoRotate continues until user changes? Actually if user drags, it doesn't disable autoRotate. It might re-rotate. Could be okay. To avoid annoyance, set `controls.autoRotate = false` but user can drag. The requirement only says 3D animation not camera. Maybe "动画" is hands. I'll keep autoRotate false, but mention user can rotate. Maybe add "若想自动旋转，取消注释 controls.autoRotate = true"? Not necessary.

Need maybe final code should include "TZ select" with "UTC" default; if testing, change.

Need maybe if code in final should be "as is". Good.

Let's think if there are any hidden bugs with `Intl.DateTimeFormat` and `hourCycle: 'h23'` in `en-GB`. Some browsers return hour as "24" at midnight if `hourCycle: 'h23'`? Actually h23 allows 00-23, so okay. If `hourCycle` not supported, it may use default 12-hour? Wait `en-GB` default is 24-hour. But to ensure, can use:
```
hour12: false
```
with `hourCycle` set? Using both not allowed? Actually `hour12` and `hourCycle` are mutually exclusive? In ECMA, hour12 can be used instead of hourCycle; specifying both might throw RangeError? Let's check. `hourCycle` is a valid option; `hour12` is also; if both specified maybe error? In `DateTimeFormat`, "hourCycle" and "hour12" are mutually exclusive? The spec says if hour12 is not undefined, hourCycle is ignored? Need not risk. Use `hourCycle:'h23'` only. For older browsers that support timeZone but not hourCycle? Most modern. Could use `hour12: false` only and normalize 24. `hour12:false` is widely supported. But with `hour12:false`, midnight may format as "24" in some locales? Actually with hour12 false, hourCycle is "h24" or "h23"? In ECMA, hour12 false maps to hourCycle "h23" if locale prefers? I think most use h23. But to be safe, normalize '24' -> '00'. Let's use `hour12: false` only; simpler. But `hourCycle` is okay. Which is safer? Both supported. Let's use `hourCycle:'h23'` for no 24. It's supported in all modern browsers. Good.

Need if `formatToParts` with `weekday:'short'` and `timeZone` returns weekday in English. Good.

Need if `tzSelect.value` maybe "UTC"; `timeZoneName:'short'` returns "UTC". Good.

Potential issue: `ctx.fillText('TZ · DUAL', CX, 220)` uses "·" maybe font. okay.

Need maybe "date/week display" in Chinese; if Canvas font "Microsoft YaHei" not on Mac, fallback "PingFang SC" maybe not recognized? The font string `"Microsoft YaHei", "PingFang SC", sans-serif` okay. If not, sans-serif. Fine.

Need perhaps if `MeshStandardMaterial` with map and no `side` -> front side only. The dial is viewed from front, okay. If user rotates behind, dial hidden but caseBack visible. That's realistic.

Need maybe if the strap starts at lugs and extends behind, but the watch case back at z=-0.35 might be behind strap? Strap at z=0.3 at lugs, so in front of case back, okay.

Need perhaps if crown cylinder after rotation z=π/2 becomes axis along -X. The cylinder's flat caps face ±X. Good.

Need maybe if case side open-ended has no caps, but dial at front and back at back close. Need maybe the back cap rotated y=π; `CircleGeometry` front side is +Z, rotating y=π makes normal -Z, visible from back. But if camera from front, back cap's backside? no. Good.

Need maybe if `back.position.z = -0.35` and side open-ended from -0.35 to0.35, side ring and back edge match. Good.

Need maybe if `dial.position.z = 0.36`, it is in front of side front edge by 0.01. The bezel torus at z=0.36 with tube radius 0.13 extends from z=0.23 to0.49. It may be in front of hands? The torus is around circumference, not center, so doesn't block. It extends far, but at z=0.49 it's at edge. Hands at center don't reach edge except second hand length 2.6; torus inner radius =3-0.13=2.87, so no overlap. Good.

Need maybe if hour/minute hands are at z=0.44/0.46/0.48, above dial and below? The bezel torus at edge extends z=0.49. If hand at full extension near edge, at same z maybe slightly in front of torus? But no matter. Could set hands z=0.42,0.44,0.46 and bezel at z=0.36. Let's adjust to avoid second hand higher than bezel? Actually hand length 2.6, at edge near z=0.48, torus top at z=0.49 and inner edge at r=2.87, hand at r=2.6 is inside torus, so no collision. Fine.

Need maybe if center cap at z=0.48 and second hand at 0.48, cap covers. Good.

Need perhaps if "smooth" second hand and "date/week display" maybe the date window doesn't update at midnight until date changes; yes.

Need maybe if the LCD plane at z=0.375, date plane same, both transparent; if they intersect? no. Date plane at x=1.55, LCD at x=0. Fine.

Need maybe if `MeshBasicMaterial` with transparent true and no `depthWrite:false`? We set. Good.

Potential issue: The `watchGroup` is a `Group`. `createHand` adds hand meshes to `watchGroup`, but hands' rotation.z around center; yes. If the watchGroup had rotation, nothing. Good.

Need perhaps if `controls` should update after render? Usually `controls.update()` before render. We do before render. Fine.

Need perhaps if `tzSelect` change should call `drawLCD()` but `drawLCD` may be called before `lcdCanvas` defined if user changes before init? Select exists before script, but user can't change before init? Could if page loading slow; no. In event listener added after build, so okay.
Need perhaps if `tzSelect` change event sets `lastLCDKey=''`; if `lcdCanvas` exists. yes.

Need "运行方式" in final:
- Save as `3d-watch.html`, double-click or open in browser.
- Requires network for Three.js CDN; no build step.
- Drag mouse to rotate; scroll wheel zoom.
- On dropdown select second timezone.

Need "测试说明":
- Compare analog hands to system time; seconds hand should sweep continuously (not tick once per second).
- Date/window at 3:00 updates with current date/week.
- LCD at 6:00 shows selected second timezone's time/date; change dropdown and verify.
- Check 3D by rotating: case, crown, lugs, strap visible.

Need perhaps "second timezone supports simultaneous tracking" in code.

Need maybe "完整的 3D 腕表" maybe in final say "已包含表壳、表镜、表冠、表耳、表带、表盘和指针". We didn't include "表镜" but okay. We have no crystal. Could say "表壳/表冠/表耳/表带". Better not claim glass. But "完整" enough.

Could add crystal glass now as transparent dome? Might complicate. Maybe add a flat glass disc at z=0.55 transparent? It would be visible when rotating; but it could obscure? Let's consider adding a transparent `CircleGeometry` at z=0.52 with opacity 0.12. It would sit in front of hands? Hands at z<=0.48, glass at 0.52. The glass is thin and transparent. It would show specular; nice. But if CircleGeometry at z=0.52 and no side, it is a flat disc floating just above bezel. Could look like watch glass. Let's add:
```
const glassMat = new THREE.MeshPhysicalMaterial({color:0xffffff, metalness:0, roughness:0.05, transparent:true, opacity:0.14, side:THREE.DoubleSide});
const glass = new THREE.Mesh(new THREE.CircleGeometry(2.92,64), glassMat);
glass.position.z = 0.52;
watchGroup.add(glass);
```
But MeshPhysicalMaterial in r128 supports transparent opacity; no transmission. A flat transparent disc at z=0.52 over hands: It is closer to camera; transparent objects render after opaque hands. With depthTest true, hands closer? Hands at z=0.48, glass at z=0.52. Camera at z positive: hand depth is smaller? Wait from camera at z=9, z=0.48 is farther? Actually camera looks along -Z. Depth buffer depth increases with distance. Hand at z=0.48, glass at z=0.52. Glass is closer? Camera at z=9, along -Z; distance to z=0.52 is 8.48, to z=0.48 is 8.52, so glass is slightly closer. Transparent glass would render after hands; depth test for glass fragment: its depth closer than hand, maybe it passes and blends over hand. But opacity 0.14, okay. It might make hands slightly tinted. Good. But a flat disc at z=0.52 with side DoubleSide can be seen. However from an angle, it floats above hands but no side. That's okay. Could add. Need if glass is transparent, transparent objects sorted by distance from camera; it may render before/after dynamic planes? No issue. Need if `MeshPhysicalMaterial` with transparent maybe no transmission. Good.

But a flat transparent disc might reflect light? It won't reflect, but okay. Could use `MeshPhongMaterial` with specular. Simpler. Let's add? The user didn't require. "完整 3D 腕表" maybe yes. But adding glass with transparency could cause depth sorting with hands; if it blends, okay. We'll add with opacity 0.08 to not obscure.

Need maybe if transparent glass at z=0.52 and hands at z=0.48, from camera looking +Z, glass is in front. If you rotate behind, glass is behind. Good.
Need maybe if glass circle radius 2.92, slightly smaller than case 3, inside bezel. Good.
Let's include:
```
const glassMat = new THREE.MeshPhysicalMaterial({color:0xffffff, metalness:0, roughness:0.05, transparent:true, opacity:0.12, side:THREE.DoubleSide});
const glass = new THREE.Mesh(new THREE.CircleGeometry(2.90,64), glassMat);
glass.position.z = 0.52;
watchGroup.add(glass);
```
This plane's normal +Z. When viewed from front, transparent. Good. If user rotates, one side. side DoubleSide.

Need if glass is at z=0.52 and bezel torus top z=0.49, glass inside. Good.
Need if glass is rendered before hands? Transparent after opaque, but depth test. It is closer than hands, so it will blend. Good.

Potential issue: The glass is a transparent plane, but because it has no thickness, from front it may reflect/refract no, just tint. Fine.

Need maybe add "back display" transparent? no.

Need maybe "current time" in date window should be local date, not selected timezone. Good.

Need maybe "second timezone" in LCD also includes date/week. Good.

Need maybe "support two time zones" selected via dropdown, not automatic. Good.

Need perhaps if "current local time" if system timezone changes, analog updates because `new Date()` uses system time. Good.

Potential bug: `new THREE.CylinderGeometry(3,3,0.7,64,1,true)` with `openEnded=true` but no `side=THREE.DoubleSide`. The side tube's normals point outward, and front faces are outside. When camera inside tube? no. If camera at z positive looking at front, sees outer side. Good. If camera at side, sees outer. If camera at back, sees outer. Good. If camera somehow inside strap? no.

Potential bug: Lugs BoxGeometry default faces front; okay. lugs at z=0, case side at z=-0.35..0.35, lugs extend from z=-0.275..0.275, not connected to side? At y=3.2, case side radius 3, center y=3.2. Lug box centered y=3.2 with height 0.6 extends from y=2.9..3.5, overlapping case radius 3.0; z from -0.275..0.275, within case side z [-0.35,0.35]. So connected. Good.

Potential bug: Strap starts at y=3.2,z=0.3, but lug box extends z=-0.275..0.275; strap center z=0.3, radius 0.6 extends z=-0.3..0.9, so connected. Good.

Potential bug: The strap is behind the watch at y=0,z=-1.45 with radius 0.6, z=-2.05..-0.85. It doesn't intersect case back z=-0.35? closest z=-0.85, no. Good.

Potential bug: If watch face in XY plane, the strap center at z=-1.45 means it curves behind. The watch case back at z=-0.35; there is a gap between case back and strap. It looks like strap extends behind with lugs, okay.

Need maybe add "crown" at x=3.35, but if user rotates, can see. Good.

Need perhaps if "second hand" shape is very long and thin, extrude depth 0.015; material DoubleSide. Good.

Need perhaps if "minute hand" shape with quadraticCurveTo from (0.04,2.3) to (0.1,1.8)? Actually shape:
```
minuteShape.moveTo(-0.08,0);
lineTo(-0.1,1.8);
lineTo(-0.04,2.3);
quadraticCurveTo(0,2.5,0.04,2.3);
lineTo(0.1,1.8);
lineTo(0.08,0);
lineTo(0.2,-0.35);
lineTo(-0.2,-0.35);
closePath();
```
This has a pointed tip via quadraticCurveTo. Need if the tip segment from -0.04,2.3 to quadratic control 0,2.5 to 0.04,2.3 creates rounded point. Good.
Hour shape similar.

Need maybe if the shape's tail at negative y makes the pivot in center? At y<0, tail extends to -0.3. Rotation.z rotates entire shape, so tail moves. Good.

Need perhaps if hands should be tapering on both sides? yes.

Need maybe if second hand has a circle counterweight? Not necessary; shape tail rectangular. Could add small circle at tail? no.

Need perhaps if second hand appears too thin? width 0.04 at top, 0.02 near center. Fine.

Need maybe if `MeshStandardMaterial` with `side:THREE.DoubleSide` and `metalness`; okay.

Need maybe if "smoothly turning" second hand has discrete "tick" due to rendering per frame but continuous math. yes.

Potential issue: In `updateLCDTexture`, key based on local hour/min/sec; if timezone same but local minute changes, key changes. fine. But if local second returns from 59 to 0, key changes. good.

Need maybe if `drawDate` uses `lastDateKey` including `getDay()`; if date same but weekday same. fine.

Need perhaps if `drawDate` is only called once and not at midnight if page stays open? In animate `drawDate()` each frame checks key. At midnight, key changes, draws. Good.

Need maybe if `tzSelect` value changes, lastLCDKey reset. Good.

Potential issue: `dateCanvas` and `lcdCanvas` are `HTMLCanvasElement` created by `document.createElement('canvas')`; if WebGL texture uploads, all good.

Need perhaps if `MeshBasicMaterial` with transparent true and `map` but no `side`; date/lcd plane visible from front. If user rotates behind, invisible. That's okay. Could set `side: THREE.DoubleSide` so dynamic display visible from behind? But from behind you'd see the backside of a digital display? Not necessary. But if "complete" maybe not. Leave front side only.

Need perhaps if "local time bottom overlay" also shows selected timezone; okay.

Need maybe if "AUTOMATIC" static text has no relation to "Automatic watch". Good.

Need perhaps if "TZ · DUAL" label maybe too close to 12 markers. Fine.

Need maybe if `ctx.measureText(timeStr)` for LCD not used? We use center. no need.

Need maybe if abbr from `timeZoneName:'short'` can be "GMT+8" and time string maybe long, but line1 only. Good.

Need perhaps if `Intl.DateTimeFormat` with timeZone `'UTC'` and `timeZoneName:'short'` returns "UTC". Good.

Need perhaps if selected timezone "Australia/Sydney" date parts weekday "Sat" and LCD date. Fine.

Need maybe if `tzSelect` options:
```
<option value="UTC">UTC</option>
<option value="Asia/Shanghai">上海 (Asia/Shanghai)</option>
<option value="Asia/Tokyo">东京 (Asia/Tokyo)</option>
<option value="Europe/London">伦敦 (Europe/London)</option>
<option value="America/New_York">纽约 (America/New_York)</option>
<option value="Australia/Sydney">悉尼 (Australia/Sydney)</option>
```
Need if "Asia/Shanghai" local and user in China; okay.

Need perhaps if there is no "second time zone" analog GMT hand. But dual time via LCD is legitimate. The prompt says "支持同时追踪两个时区"; LCD counts. Good.

Need perhaps if user expects "日期和星期显示" in the 3D watch; date window. Good.

Need maybe if there should be "秒针、分针和时针" all smooth. yes.

Need perhaps if "秒针" should be red. Good.

Potential issue: `THREE.MeshPhysicalMaterial` transparent and `CircleGeometry` at z=0.52 could be sorted incorrectly with dynamic planes at z=0.375 (transparent) and hands opaque. Transparent object render order is sorted by distance; glass closer than hands, dynamic planes behind. It may render after both, but depth test? Glass closer than hands; if glass renders after hands, it will blend over hands. okay. If glass renders before dynamic plane? sorted by distance: glass closer, dynamic planes farther, so dynamic planes render before glass? Actually transparent objects sorted back-to-front: dynamic planes first, glass last. Glass blends over hands and planes. Good. Great.

Need maybe if glass at z=0.52, center cap at z=0.48; glass over cap. okay.

Need perhaps if watch's "bezel" torus at z=0.36 and glass at z=0.52 inside, but torus tube radius 0.13 top z=0.49, glass sits slightly above bezel, fine.

Need maybe if no "anti-reflective" no.

Potential issue: The "dial texture" static date window frame drawn at z=0.36, dynamic date plane at z=0.375, but if dynamic plane smaller than frame, static frame visible. The static frame border line at z=0.36 may be covered by the dial material; okay. But because dynamic plane is 0.015 in front, from an angle there is parallax between the static border and dynamic plane, which may reveal a slight gap. Could avoid by putting the date window frame on the dynamic plane itself. Actually if dynamic plane is smaller, the static frame behind is offset due to parallax; maybe okay. To avoid parallax, draw a frame on the dynamic plane (border around the white window), and make plane exactly same size as frame. Then no static frame needed. That might be better. But then no visible "window cutout" if plane moves? Hmm.

Let's redesign date window: dynamic plane itself has a border frame, same size as plane? If plane is 1.5x1.0, draw border at its edge. Static dial no frame. Then date window appears as a framed window at the plane's location, no parallax. That's simpler. But if the plane is 1.5x1.0, it covers an area; it can have a black border. We can draw on canvas a white background and a black rounded rectangle border. Then no static frame needed. The plane's own background covers the dial area. This suppresses need for static frame. Let's do that. Then date plane is a window with frame. We don't need draw static date frame in dial. But then the date plane is essentially a rectangle on dial. It might look like a digital display, not analog date window. But okay? Could draw just a small white rectangle with black frame. It is okay. Need ensure the plane's background doesn't cover dial markers? It's small. Let's remove static date frame from dial texture and draw frame on dynamic canvas.

Date canvas 300x200; draw:
```
ctx.clearRect(0,0,300,200);
ctx.fillStyle='#fffaf0';
roundRect(ctx, 2,2,296,196,10); ctx.fill();
ctx.strokeStyle='#444'; ctx.lineWidth=4; ctx.stroke();
```
Then plane size 1.5x1.0. This creates a visible window. No static frame. Good. Let's remove date frame from dial texture. Need maybe static dial has no date window frame. Good.

Then if date plane at z=0.375, it's a distinct element. Fine.

Need maybe LCD plane with its own frame. Good.

Need perhaps if date plane as a rectangle on dial is not recessed in 3D; but acceptable. If want recessed, could add small box frame around but too complex.

Need updateDate canvas with border.

Need remove static date frame from createDialTexture. Let's adjust.

Need if date plane covers a little dial area, no static frame behind. Good.

Need static dial should not draw date window frame. Good.

Need maybe we can draw "DATE" label above date window? Not needed.

Need if date plane at x=1.55,y=0.65, size 1.5x1.0, top at y=1.15, bottom at0.15. It's a small rectangular window. Good.

Need maybe if date plane canvas background is opaque transparent? It's opaque; material transparent true for render order. Good.

Need `drawDate` background:
```
roundRect(ctx, 0,0,300,200,10);
ctx.fillStyle = '#fffdf4';
ctx.fill();
ctx.lineWidth = 4;
ctx.strokeStyle = '#222';
ctx.stroke();
```
Need do not clear? If we fill, no need clear. But to avoid stale, clear. good.

Need maybe if no static frame, date plane itself has border; no issue.

Need maybe if the "date window" changes but the plane area is a rectangle not dial window; okay.

Need maybe if "完整的 3D 腕表" should have date window "3点钟" maybe. okay.

Potential issue: `CanvasTexture` on transparent material with opaque background and border; because transparent true, but the border alpha opaque. Good.

Need perhaps if date plane has no `side: THREE.DoubleSide`, from back invisible. Fine.

Need maybe if LCD plane has no `side`, from front visible. fine.

Need perhaps if `lcdPlane` and `datePlane` have `renderOrder` but transparent sorting may ignore? no issue.

Need maybe if no `renderer.sortObjects`, but fine.

Need maybe if `Controls` uses touch events; okay.

Need maybe if user "test" by changing system time? We can mention "不要调整代码，观察系统时间即可". Good.

Need maybe if final answer "直接完成任务并交付可运行产物" includes code. Good.

Let's refine code further mentally.

`createDialTexture` static: Need not draw date frame but maybe draw small rectangular outline? We'll omit. But if date plane has border, the dial texture behind no frame. Good.

Need maybe draw "DUAL TIME" label static; okay. If LCD plane covers bottom, label above.

Need maybe "LCD" at y=-1.55, height1.05, top -1.025, label at -0.7? Let's set label at world y=-0.6 -> canvas y=614, top of LCD 687. So label 70px above. Use `ctx.fillText('DUAL TIME', CX, CY - (-0.6) * PX)` or in code `CX + 0.6*PX`? Wait world y=-0.6, canvas y = CY - worldY*PX = 512 - (-0.6*170.7) = 512+102=614. Use `CY - (-0.6)*PX`. In code:
```
const lcdLabelY = CY - (-0.6 * PX);
ctx.fillText('DUAL TIME', CX, lcdLabelY);
```
Good.

Need perhaps static "LOCAL" label? Maybe add near 12 under brand? no.

Need if dial texture has "TZ · DUAL" at canvas y=210 and "AUTOMATIC" at y=272. Good.

Need maybe createDialTexture `ctx.fillStyle = grad; ctx.fillRect(0,0,1024,1024);` Then draw circle? If fill full square, outside circle not visible, but inside circle all filled. Good. Use `ctx.fillRect(0,0,1024,1024)` rather than circle. But gradient defined with radius 470; outside gradient edge to square maybe transparent? Actually if `createRadialGradient(CX,CY,80,CX,CY,470)`, colors stops from 0 to1; outside 470, the last stop color is used? For canvas gradient, outside radius uses last color? I think yes. But fillRect will fill corners with last color. Fine.
Need perhaps draw a circular rim of dial at radius 460. Use:
```
ctx.beginPath(); ctx.arc(CX,CY,460,0,Math.PI*2); ctx.strokeStyle='#666'; ctx.lineWidth=2; ctx.stroke();
```
Good.

Need maybe minute markers:
- At 12 position, angle = -π/2, y negative? Wait canvas y = CY + outer*sin(angle). At angle -π/2, sin=-1 -> y=CY-outer (top). Good. Great.

Need hour numerals positions:
```
const a = (i/12)*Math.PI*2 - Math.PI/2;
ctx.fillText(String(i), CX + r*Math.cos(a), CY + r*Math.sin(a));
```
At 12, top. Good.

Need "TZ · DUAL" at CX, 210. It's near 12 but below markers. okay.

Need if "AUTOMATIC" at 272, below "TZ". okay.

Need if `ctx.font='700 42px "Segoe UI", Arial, sans-serif'` uses 700; okay.

Need maybe "second timezone" displayed on dial label "DUAL TIME" static. Good.

Need "LCD" dynamic canvas: `roundRect(ctx,0,0,640,280,18)`; fill. Need if `roundRect` doesn't include fill/stroke, call separately. Good.

Need perhaps `ctx.clearRect` before fill. good.

Need if `lcdTexture` needs update after drawing. good.

Need perhaps `drawDate` should draw "日期" text in Chinese but the date plane is small; maybe date text at 3 o'clock not readable? It's okay.

Need maybe if user scales window, render. Good.

Need if `renderer.setPixelRatio` and canvas texture pixel ratio not. okay.

Potential issue: `THREE.OrbitControls` with three r128 examples/js may use `THREE.MOUSE` and `THREE.TOUCH`? no. Good.

Need maybe if using `renderer.outputEncoding = THREE.sRGBEncoding`, but `THREE.MeshBasicMaterial` textures get color space? In r128, if texture.encoding not set, it might be linear? Actually outputEncoding only affects output; if texture not marked, colors might be washed? But okay. Could set `texture.encoding = THREE.sRGBEncoding` for all color textures. We set dialTexture. Need set date/lcd? They are MeshBasicMaterial; if outputEncoding sRGB, need mark textures too to display correctly? In Three.js r128, if texture.encoding default LinearEncoding and outputEncoding sRGB, the shader converts from linear to sRGB? Maybe it might brighten? Better set `texture.encoding = THREE.sRGBEncoding` for all color textures: dial, date, lcd. But note CanvasTexture with text colors: if texture is marked sRGB, the raw sRGB values are decoded? In output encoding pipeline, textures with sRGB encoding are decoded to linear, then output encoded. Good. Use.
Need set in `createDialTexture`: `texture.encoding = THREE.sRGBEncoding;`
In createDatePlane and createLCDPlane: after CanvasTexture creation, `texture.encoding = THREE.sRGBEncoding;`
Need if not, colors might be okay. Let's include.

Need if `MeshBasicMaterial` with map and texture.encoding sRGB will be correct. Good.

Potential issue: `renderer.outputEncoding = THREE.sRGBEncoding` with alpha? no.

Need maybe if `MeshStandardMaterial` hand colors are specified in linear? Actually material color in r128 is linear? If outputEncoding, colors are converted? not important.

Need perhaps if `new THREE.Color(0x222831)` in sRGB? okay.

Need maybe if no `devicepixelratio` for CRT? no.

Need maybe if `target` in controls set after camera? yes.

Need maybe if "watchGroup" not added to scene until build. okay.

Need maybe if `scene.background` not set; body CSS behind. renderer with alpha true, body background visible. Good. If renderer alpha true and no scene.background, transparent. Good.

Need maybe if no "ground shadow", okay.

Need perhaps if "orbit controls" target at (0,0,0) but watchGroup at origin; yes.

Need maybe if final code in answer should be in one block. Good.

Let's perhaps include "说明：代码使用 Three.js CDN，需联网；若无法访问，可替换本地 three 文件." Good.

Need maybe if "测试必要说明" not too long. Good.

Let's write final code fully now. Need ensure no missing comma/semicolon.

Let's draft code carefully in final.

Potential issue: `THREE.MeshPhysicalMaterial` with transparent and opacity but no `transmission` in r128 is fine. Could use `MeshPhongMaterial` to avoid physical material complexity. Let's use `new THREE.MeshPhysicalMaterial({color:0xffffff, metalness:0, roughness:0.05, transparent:true, opacity:0.12, side:THREE.DoubleSide})`. Need if physical material requires `roughness` property; yes.

Potential issue: `MeshPhysicalMaterial` with `transparent` renders after opaque; good. But it may be affected by lights; no issue.

Need perhaps if glass plane at z=0.52 and camera at front, it's closer than hands, so it renders over them. But because opacity 0.12, hands slightly tinted. Good.
Need perhaps if glass plane's material `depthWrite: false` should be set to avoid transparent object writing depth? For transparent objects, should set `depthWrite:false` to avoid issues. If not, glass may write depth and hide dynamic planes? Wait glass is closer, if depthWrite true, it writes depth closer, so dynamic planes behind fail depth and won't render? Actually transparent objects can depthWrite false to prevent. Let's set `depthWrite:false` for glass, date, lcd. But for opaque hands, depthWrite true. Good. Add `depthWrite:false` to glass material. But if glass is transparent and depthWrite false, dynamic planes behind still render before glass? Transparent objects sorted back-to-front; glass rendered after, blending. But dynamic planes may have depthWrite false too, so order okay.
Set `glassMat.depthWrite = false`.
Need if transparent objects with depthWrite false and depthTest true; okay.
Need perhaps render order for glass should be high. Set `glass.renderOrder = 5`. Dynamic planes 2,3, glass 5. Good.

Need if `renderer.sortObjects` sorts by z distance, renderOrder not necessary. But okay.

Need createDatePlane/lcdPlane with material `depthWrite:false`. Good.

Need if date/lcd planes are transparent and depthWrite false, but they are behind hands. When hands opaque rendered first, then transparent date/lcd maybe if camera from front, date/lcd fragments behind hands fail depth test because hands wrote depth. Good. But if transparent objects sort and render after opaque, depth test is against depth buffer generated by opaque. Good. If glass is transparent and depthWrite false, it doesn't update depth, so hands remain visible through glass. Good.

Need maybe if glass is closer than hands, and depthTest true, glass fragment passes (no depth from glass, hand depth is farther), so it blends over hand. Correct.

Need maybe if transparent date/lcd planes are behind glass, glass blends over them. good.

Need perhaps if `MeshBasicMaterial` with map and transparent true but no alpha in map? It still uses alpha channel. Since our canvas has opaque pixels, alpha=1. Good. The material transparent true allows alpha blending. The canvas starts transparent (alpha=0) if not filled. We fill background, so alpha=1. clearRect before fill gives transparent corners? Wait if we fill rounded rect only, the corners of canvas remain transparent. That's good! Date plane can have rounded corners and transparent outside. But if `clearRect` then fill rounded rect, outside transparent. Plane geometry rectangular; outside alpha 0. Great. That means the plane's corners don't cover dial. Good. Need material transparent true. Good.
For LCD, fill rounded rect; outside transparent. Great.
Need if `drawDate` clearRect and then `roundRect` fill; but the border maybe. Good.
Need if `roundRect(ctx,2,2,296,196,10)` and fill; corners outside transparent. Good.

Need perhaps if `ctx.clearRect` before fill; yes.

Need maybe if `drawLCD` fill rounded rect; outside transparent. Good.

Need if `drawDate` text not clipped to rounded rect; if text within window, okay.

Potential issue: The date window dynamic plane is transparent outside rounded rect, but the plane itself with depthWrite false and transparent true may be rendered after opaque hands. However if it's behind hands, okay. If its center at z=0.375 and hands at z=0.44, it is behind; depth test rejects over hands. Good.

Need perhaps if the transparent rounded rect corners allow dial background visible. Nice.

Need maybe if static dial has "AUTOMATIC" text, date plane cover it? Date plane at y=0.15..1.15, "AUTOMATIC" at y=1.35? Wait "AUTOMATIC" at canvas y=282 => world y=1.35, above date plane top 1.15. no.

Need if "TZ · DUAL" at y=1.77. okay.

Need perhaps if "LCD" label at y=-0.6, LCD top -1.025. good.

Need maybe if LCD date line at bottom might be clipped by rounded rect? The rounded rect has height 280, fill, text at y=240, bottom margin 40. okay.

Need perhaps if time string at font 92px, center y=148, line height ~92, within 280. okay.

Need perhaps if abbr text at x=30,y=44, line1. okay.

Need perhaps if time string length "08:32:45" 8 chars * 55px? 92px monospace char width ~55, total 440. Fits 640. Good.

Need perhaps if in LCD material, map includes transparent corners; if material transparent true, good.

Need perhaps if date plane canvas 300x200, rounded rect at 2,2,296,196; border width 4. Text fits.

Need perhaps if `drawDate` uses `ctx.font='bold 48px "Microsoft YaHei", "PingFang SC", sans-serif'`; if not found, falls back. Good.

Need perhaps if "weekdays" Chinese chars in canvas require font and fine.

Need maybe if `ctx.font='bold 28px'` and `fillText(weekday, 150, 44)`. Good.

Need if date plane at 3 o'clock "5月14日" text width maybe 50*5=250, but "月" and "日" maybe 50, "14" narrower. Fits 300? If date "12月31日", chars: 12,月,31,日 = 5 glpyhs; with CJK 50px each, "12" ~60, "31"~60, "月"50 "日"50 total 220. good.

Need perhaps if "TZ · DUAL" static text includes dot and spaces; okay.

Need maybe if "AUTOMATIC" static no.

Need perhaps if "createDialTexture" uses `const PX = 1024 / 6;` But if we use for labels. Good.

Let's write code in final. Need ensure no HTML entities? code block with triple backticks. Good.

Need maybe if "user asked output direct runnable" and final includes run instructions. Good.

Let's prepare final answer in Chinese with concise explanation and code.

Need maybe add "可以直接保存为 .html 打开" etc. Good.

Let's craft final.

Potential issue: In `drawLCD`, `const abbrParts = new Intl.DateTimeFormat('en-US', {timeZone: tz, timeZoneName:'short'}).formatToParts(now);` For some timezones "UTC" returns `timeZoneName` "UTC"; for "Asia/Shanghai" returns "GMT+8" maybe. Good.
Need maybe if `timeZoneName:'short'` with `formatToParts` returns "GMT+8" for `Asia/Shanghai`. Good.

Potential issue: If `tzSelect.value` changes, `drawLCD` event calls; but if `drawLCD` called before `lcdTexture` exists? Let's ensure event listener added after buildWatch. In init:
```
buildWatch();
window.addEventListener...
tzSelect.addEventListener...
```
Yes.

Potential issue: If `animation` run before `tzSelect` event? yes.

Potential issue: In `updateOverlay`, `now.toLocaleString('zh-CN', {hour12:false})` may not include seconds? Actually default includes seconds? The default `toLocaleString` with date and time includes seconds? It includes "2026/5/14 14:32:45" yes. Fine.

Need maybe if user locale not zh-CN, still okay.

Potential issue: If `lastLCDKey` includes `tz` and local seconds, but if `drawLCD` called by event before animation, it uses current local seconds. At same second, lastLCDKey set to key. Then in animate same second returns. if timezone changed later at same second, lastLCDKey reset and draw. good.

Need perhaps if `drawDate` uses key with date and day; if date changes, draws. At first frame, lastDateKey empty, draws. good.

Need maybe if `dateTexture.encoding = THREE.sRGBEncoding;` and material transparent; okay.

Need perhaps if `dialTexture.encoding = THREE.sRGBEncoding;` and MeshStandardMaterial map; okay.

Need maybe if `createDialTexture` returns `texture`; if `texture.encoding` set after creation. Good.

Potential issue: In `createDatePlane` and `createLCDPlane`, the texture encoding set but if MeshBasicMaterial and outputEncoding sRGB, the output? okay.

Need maybe if `renderer.outputEncoding = THREE.sRGBEncoding;` may make canvas textures with sRGB encoding display correct. Good.

Need perhaps if `fog` no.

Need maybe if watchGroup should cast shadows? not necessary. Some meshes cast but no receive. okay.

Need perhaps if "lights" and "shadow" no floor. okay.

Need maybe if `TubeGeometry` with CatmullRomCurve3 points and no `closed`; default curve not closed? We pass false. Good. But `CatmullRomCurve3` default closed false; okay.
Need if curve points start/end at y=3.2 and -3.2, but the curve interpolation may overshoot in z? CatmullRom might overshoot, causing strap to curve weird. Could use `curveType:'catmullrom'` default. Fine. If overshoot top? okay. Could use `new THREE.CatmullRomCurve3(points)`; no. Should be okay.
Need maybe if strap radius 0.6 and curve points close to case; okay.

Need maybe if no "wrist" cylinder, the strap is just a loop. But okay.

Need perhaps if "3D wristwatch" should be "腕表" not "怀表". Strap makes it wristwatch. Good.

Need perhaps if "current time" uses browser local timezone; if user wants "synchronizes with current time", yes.

Need maybe if "test instructions" in final mention "日期/星期窗口在 3 点位置，双时区 LCD 在 6 点位置". Good.

Need perhaps if final code is huge but acceptable.

Let's now formulate final answer.

Double-check code for `buildWatch`: `const dialTexture = createDialTexture();` createDialTexture uses `new THREE.CanvasTexture(canvas)`. CanvasTexture constructor expects canvas. okay.
Need maybe if `CanvasTexture` not defined in r128? yes.

Need perhaps if `new THREE.MeshStandardMaterial({map: dialTexture, roughness:0.55, metalness:0.1})` no color default white. Good.

Need perhaps if "dial" CircleGeometry at z=0.36 but `side` default front; if camera from front, okay.

Need perhaps if "back" CircleGeometry at z=-0.35 with rotation.y=Math.PI; if camera from front, backside not visible; okay. If camera from back, front side visible? Wait back mesh rotated y=π. Its local +Z normal becomes -Z. If camera at z negative, it looks in +Z direction. The front face of the mesh is the side whose normal points toward camera? The normal after rotation points -Z, which points toward camera at z<0? Camera at z=-8 looks towards +Z, the face normal -Z is opposite to camera? Actually camera at -Z looks along +Z, a surface at z=-0.35 with normal -Z is facing away from camera (normal points away from camera direction? Camera direction is +Z; normal -Z, angle 180, backface). Wait! Let's analyze.

CircleGeometry lies in XY plane with front face normal +Z. To create a back cap visible from z negative (back), we need the face's normal to point -Z (toward camera at z negative). `rotation.y = Math.PI` rotates the normal +Z to -Z? Rotation around Y by π: (0,0,1) -> (0,0,-1). Yes. A plane with normal -Z at z=-0.35. A camera at z=-8 looks in +Z direction toward origin. The front face normal -Z points toward camera? The camera direction vector is (0,0,1). Surface normal is (0,0,-1). The dot product is negative, so it's back-facing? Wait for a camera at z=-8 to face +Z, triangle front faces should have normals with +Z component? Let's think: A standard plane at z=0, normal +Z is visible from camera at z=+8 looking -Z. It is not visible from camera at z=-8 looking +Z. If we rotate it 180°, normal -Z, it is visible from camera at z=-8 looking +Z? The face normal -Z points towards negative z, which is the same direction as the camera's position relative to the plane (camera at -8). The camera direction is +Z. In OpenGL/Three, a face is front-facing if the normal points toward the camera? Actually front-facing determined by winding order in screen space, not lighting normal. The front face of a primitive is the side where vertices appear counterclockwise. Normal is computed for front face. For a camera at z=-8 looking +Z, the side of the plane facing the camera is the z-negative side. The normal -Z is pointing from the front face outward toward the camera. The camera viewing direction is +Z, but lighting uses normal to camera vector; camera vector from surface to camera is (0,0,-8) - (0,0,-0.35) = (0,0,-7.65), dot with normal (-Z) = positive. So it is front-facing. Yes. Good. So back cap visible from back.

Need if dial at front z=0.36 normal +Z visible from front. Good.

Need if `side` open-ended cylinder after rotation.x=π/2: CylinderGeometry side normals radial outward; rotation doesn't alter. Good.

Need perhaps if the side material has no `side:DoubleSide`; from inside tube not visible, but from outside okay. Good.

Need if `bezel` torus default in XY plane with normal Z, visible from front and back? Torus geometry is DoubleSide? Default FrontSide, but torus side surfaces are oriented outward; from outside, visible. Good.

Need if `crown` cylinder after rotation.z=π/2: CylinderGeometry axis Y, after rotation axis -X; caps face x negative/positive. The side surface normals radial outward perpendicular to axis, visible from most directions. Fine.

Need if `strap` TubeGeometry default FrontSide, visible from outside. good.

Need if `lugs` BoxGeometry visible.

Need perhaps if "glass" at z=0.52, visible from front but if camera from back it's backside? side DoubleSide, visible. Good.

Need perhaps if dynamic date/lcd planes have front side +Z, visible from front. From back, not visible, but there is case back. Good.

Potential issue: `createDialTexture` uses `ctx.fillText('TZ · DUAL', CX, 220)`; if the texts are at y=220, world y=1.71, but 12 numeral at y=335 px = world y=1.04? Wait 12 numeral at r=335 px -> world y=1.96? Let's compute: canvas y = CY - r = 512-335=177 -> world y=(512-177)/170.7=1.96. Brand at y=220 -> world y=1.71. Good, below 12. fine.

Need perhaps if "AUTOMATIC" at y=282 -> world y=1.35. Date plane top 1.15, so no overlap.

Need perhaps if "TZ · DUAL" and "AUTOMATIC" both at center, okay.

Need perhaps if hour numerals added before brand, brand overlays nothing. Fine.

Need perhaps if `ctx.font='700 42px ...'`; `font` shorthand includes weight. Good.

Potential issue: In `drawLCD`, `const abbrParts = new Intl.DateTimeFormat('en-US', {timeZone: tz, timeZoneName:'short'}).formatToParts(now);` If `timeZoneName:'short'` not supported when no time? It is. Good.

Need perhaps if `Intl.DateTimeFormat` with `hourCycle:'h23'` and `formatToParts` on Safari can include hour with `"24"`? We normalize.

Need perhaps if `drawLCD` date parts `month` variable name `mo` to avoid conflict. Good.

Need maybe if `updateOverlay` uses `Intl.DateTimeFormat('zh-CN', {timeZone: tz, hour:'2-digit', minute:'2-digit', second:'2-digit', hour12:false}).format(now)`; with `hour12:false`, okay. Could return `24:00:00`; no matter. Maybe use same `hourCycle`? Use `hourCycle:'h23'` and `hour12:false`? Don't. Use `hourCycle: 'h23'`:
```
{ timeZone: tz, hourCycle:'h23', hour:'2-digit', minute:'2-digit', second:'2-digit' }
```
Good. But if `hourCycle` and `hour12` absent, okay. For `toLocaleString('zh-CN', {hour12:false})` okay.

Need perhaps if `hourCycle:'h23'` and `zh-CN` no issue.

Need maybe if `tzSelect` ID at top; if script at end, element exists. yes.

Need maybe if `watchGroup` name global; buildWatch uses `watchGroup` variable. `scene.add(watchGroup)`. okay.

Need perhaps if `createHand` uses `side:THREE.DoubleSide` but `metalness` maybe. okay.

Need perhaps if `createHand` shape mesh "castShadow" but no floor. okay.

Potential issue: Extruded shape at z = z - depth/2 to z + depth/2. If second hand z=0.48, depth=0.015, from 0.4725 to0.4875. Center cap at z=0.48 spans 0.42..0.54. It covers. Good.

Need perhaps if hour hand z=0.44, minute z=0.46, second z=0.48. The glass at 0.52, hands inside. Good.

Need perhaps if date/lcd planes at z=0.375, and dial at z=0.36. Date/lcd planes are closer by 0.015. If hands at z=0.44, they are 0.065 away. okay.

Need maybe if "date window" dynamic plane at z=0.375 but hands pass in front, okay.

Need perhaps if `watchGroup` has no `rotation` and scene coordinate Y up; face in XY plane; okay.

Need perhaps if "3D" watch face facing z positive; if user rotates with OrbitControls, camera can go around. Good.

Need perhaps if "complete 3D" wants "under the hood" maybe no.

Need perhaps if "double timezone" and local date overlap? no.

Need perhaps if "local info" overlay includes second timezone, maybe redundant but helpful. Good.

Let's now craft final answer. Since final should be "输出要求：请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" We'll answer with code and instructions. Good.

Need perhaps not too much preamble. Use "下面是完整可运行的 HTML 文件". Good.

Let's produce final.

But before final, consider if code uses optional "roundRect" function. Need ensure no name conflict with CanvasRenderingContext2D.roundRect not used. Good.

Need maybe if `ctx.lineWidth = 4` after `roundRect`; `stroke()` uses line. fine.

Need perhaps if in `drawDate` we set `ctx.strokeStyle` after fill; need call `ctx.stroke()`. Good.

Let's write final code with careful indentation. Need maybe not too overlong but okay.

One more possible issue: `new THREE.CylinderGeometry(3, 3, 0.7, 64, 1, true)` rotation.x = Math.PI/2. In three.js, open-ended cylinder has no caps; after rotation, side tube from z=-0.35 to z=0.35. Yes. But because the default cylinder top at y=+0.35 rotates to z=+0.35? For rotation.x=π/2, Y axis maps to Z axis? Let's verify with matrix: Rx(π/2) maps Y to Z and Z to -Y. So top cap center at y=+0.35 maps to z=+0.35. Good. Side axis along Z. Good.

Need maybe if back cap at z=-0.35 and dial at z=0.36, side tube ends at -0.35 and +0.35. Dial is 0.01 in front of side, so closes front. Good.

Potential issue: `new THREE.TorusGeometry(3,0.13,16,64)` default lies in XY plane with hole along Z. It is centered at z=0.36. However torus radius 3, tube radius .13, its outer edge radius 3.13; it might protrude beyond case side radius 3 by 0.13. That forms a bezel wider than case. Fine. Its inner radius 2.87, slightly smaller than case, covering the dial edge. It may overlap markers at r=2.75? Actually markers at 2.75 (<2.87), so not covered. The dial texture extends to radius 3 but bezel covers outer 0.13. Good.
Need if bezel z=0.36, at z=0.49 top; glass at z=0.52 just above. Good.

Need perhaps if lugs connect to case at radius 3, but bezel outer radius 3.13 at z=0.36, lugs at y=3.2,z=0.0. No overlap? The bezel at y=3.2? Torus around origin radius 3; at y=3.2, the torus center y? Actually torus tubes are circular around origin; the point (0,3.2) is outside radius 3+0.13=3.13 by 0.07. Lugs at y=3.2 just outside bezel, okay.

Need maybe if crown at x=3.35, torus outer radius 3.13, so crown just outside. good.

Need perhaps if "texture" of dial should be rotated? Canvas y down, Three.js UV v up? Wait important! For a texture applied to a mesh, the v coordinate origin is bottom-left in UV? In Three.js, images are uploaded with flipY=true by default for textures. CanvasTexture flipY default true? For CanvasTexture, `flipY` defaults true. When drawing on canvas, the top-left origin becomes UV v=1? Need Three.js handles flip. Generally CanvasTexture displays correctly (not upside down). So no need. Good.

Need maybe if dial texture axis: canvas coordinate y down; Three.js map v up after flip; should align with standard coordinate. Good.

Need perhaps if dynamic planes CanvasTexture with text: Usually CanvasTexture displays right way up. Good.

Need maybe if `texture.encoding = THREE.sRGBEncoding` changes flip? no.

Potential issue: The `MeshBasicMaterial` for date/lcd with transparent true but map has `encoding` sRGB; if outputEncoding sRGB, transparent alpha not issue. Good.

Need maybe if `drawLCD` uses `new Intl.DateTimeFormat` each second, a bit of allocation but fine. Good.

Need perhaps if no internet, code won't run. mention.

Need perhaps if "完整的 3D 腕表" maybe should have "glass" but physical material might be too reflective? no.

Let's produce final answer.