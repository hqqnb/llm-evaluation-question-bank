We need answer in Chinese likely. Need create 3D wristwatch animation, running current time, smooth hands, date/day, two timezones, full 3D watch. Need deliver runnable artifact. Need decide format. We can produce HTML file with Three.js? Need "direct task and deliver runnable artifact; provide run instructions and test". We need final answer with code likely. Need create a single HTML file using Three.js from CDN. Include 3D wristwatch model constructed procedurally. Need handle current time based on system, smooth second hand (not ticking? "平滑转动" smooth). Need date and week display. Two timezones tracking. Need full 3D wristwatch representation.

Need think implementation details. We can create a 3D scene with Three.js: watch case, bezel, dial, indices, hands (hour, minute, second), date window, weekday window perhaps. Need a watch strap? "完整 3D 腕表形式" maybe include case, strap, crown. Need animate.

Need support two time zones: display local time and second timezone. Could have dial with two subdials or digital displays in 3D. Need current time. Need date and weekday. Maybe use textures for dial with date and weekday windows. To avoid complicated texture? Could use canvas texture for watch face, with date and weekday windows? But hands are 3D objects. Need two timezone display: perhaps two small digital screens on dial rendered as textures updated each frame? Or two subdials with 24-hour? Need track two timezones. Could implement an analog world timer: main dial shows local time, plus a secondary 24-hour subdial showing time in another timezone. But user asks "同时追踪两个时区" not necessarily display both as analog. Could have two "digital screens" on the watch face showing local and second timezone times with date/week. That might be simpler: create canvas texture for screen? Need update dynamic text. Could use Sprite/CanvasTexture on plane inside dial? We can update texture every frame. But "3D wristwatch" maybe okay.

Need ensure date and weekday display function. We can implement with digital date/weekday on screen. Need maybe "砂" etc.

Need produce self-contained HTML. Need include Three.js library. Need use ES modules? Better use unpkg/three with non-module global? Three.js r128? Need CDN. To avoid module import complexity, use script tag from cdnjs: three.min.js. But recent Three.js removed non-module? We can use r128 from cdnjs: https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js. Then use global THREE. Need import OrbitControls? Could implement own simple rotate or include OrbitControls from CDN. Need user can view full 3D. We can include OrbitControls (examples/js/controls/OrbitControls.js) for r128. Need maybe use script cdn: https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js. This works with global.

Need build watch. Let's design.

Scene:
- background: neutral gradient or room environment. Use renderer, camera, lights.
- Watch group rotated maybe for presentation.

Watch components:
1. Case:
   - CylinderGeometry or LatheGeometry? Need watch case shape: rounded. Could use CylinderGeometry with radius 2, height 0.6, open? We can use a group:
     - Main case: cylinder (r=2, h=0.5) rotated x? In Three.js, cylinder default axis y. We want watch face in XY plane? Hmm. Need coordinate system. To place dial facing camera along z, use geometry rotated x = Math.PI/2 so cylinder axis z? Let's define watch face plane XY? Actually if cylinder axis y, then top/bottom circular faces are perpendicular to y. If we rotate x by 90°, cylinder axis becomes z, circular face faces z. Then dial can be at z=0.25. Good.

Use THREE.CylinderGeometry(2, 2, 0.5, 64). Rotate x = Math.PI / 2. Place at z=0. It becomes cylinder along z with circular face at z? Need verify. Cylinder geometry axis y. Rotate X by PI/2: y -> z? Standard: rotation around x maps y to z? For PI/2, y-axis -> z? Actually R_x(theta): y' = y cos - z sin; z' = y sin + z cos. For theta=90°: y'= -z, z'= y. So a point (0,1,0) goes to (0,0,1). So cylinder axis along z. Good. Top cap at y=0.5 goes to z=0.5? Point (0,0.5,0) -> (0,0,0.5). So face at z=0.5. We can position case at z=0, dial at z=0.25? Need case height 0.6. Place dial at slightly front? Let's think.

Maybe easier: Build watch face in XZ plane? Use standard Three.js plane? But simpler to keep cylinder axis y. Then rotate whole watch group so face points camera? For presentation, group rotation x? But if group rotation changes, coordinate system still relative to group. We can build watch with face in XY plane? Let's choose watch face plane XY, normal along Z? Cylinder axis Z? We can build with cylinder axis z by rotation. This is fine.

Let's define watch group as at origin with face normal +Z. Build:
- Case back/middle: CylinderGeometry(2, 2, 0.6, 64), rotateX(PI/2), material silver metallic. It will have caps at z=±0.3? Wait top cap original y=0.5? Let's be precise.

CylinderGeometry(radiusTop, radiusBottom, height, radialSegments). Default along Y, centered at origin. Top cap at y=+height/2. RotateX(PI/2): y+height/2 maps to z? Using R_x: (0, h/2, 0) -> (0, 0, h/2). So front face at z=+h/2. Good. So case centered z=0, front cap at z=+0.3, back cap at z=-0.3. The circular face (top cap) is plane perpendicular to z, normal +Z after rotation. Good.

- Bezel: torus/ring around front? Could use TorusGeometry(2.0, 0.12, 16,64) in XY plane? Torus lies in XY plane by default? Three.js TorusGeometry is in XY plane centered at origin, normal Z? Actually torus lies in XY plane with hole along Z. So need rotate? Default torus axis Z? Torus in XY plane, yes. We can place z=0.3, radius 1.9? A torus with tube 0.1 around radius 2? Let's use ring shape: maybe LatheGeometry? Simpler use CylinderGeometry open-ended with radius 2 and height 0.15, at bezel front? Or use TorusGeometry(2, 0.1, 16, 64), no rotation, place z=0.2? But torus's central ring radius is 2.0, tube radius 0.1. It would wrap around case edge. If placed at z=0.3? Need align. We can use TorusGeometry(1.95, 0.12, 16, 64) at z=0.3? Need check torus axis is z, so it lies in XY plane at z=0.2. It will be a ring around front. Good.

- Dial: CircleGeometry(1.85, 64) rotated? CircleGeometry lies in XY plane, normal Z by default? Yes PlaneGeometry and CircleGeometry are in XY plane facing +Z. So place at z=0.3? Wait case front cap at z=0.3. If dial at z=0.31, visible. But need room for hands above dial. We can place dial at z=0.31, hands at z=0.35-0.45.

Need create dial with details. Options:
- CanvasTexture for dial face with tick marks, brand text, date and weekday windows, two timezone screens? Need dynamic date and weekday display and second timezone. Could create a static dial texture containing brand, tick marks. For dynamic date/week and second timezone, we can overlay small 3D planes with CanvasTextures updated each frame. Need maybe also have date window and weekday window in dial texture. Could implement entirely with a canvas texture for static dial plus dynamic texture for date/week/second timezone? Let's think.

Need "日期和星期显示" and "two timezones". We can make digital displays on the dial:
- A rectangular black LCD panel at 6 o'clock (or below center) showing:
  Line 1: Local time HH:MM:SS (and date/weekday)
  Line 2: Second timezone HH:MM:SS (and maybe timezone name)
But user specifically "腕表应根据当前时间运行，并具有平滑转动的秒针、分针和时针。还可以提供日期和星期显示功能。它应支持同时追踪两个时区，并以完整的 3D 腕表形式呈现。" A digital display is okay? But "完整3D腕表" with analog hands and digital subdisplay for second timezone/date/week. Need perhaps better: analog main time, plus two small analog subdials for second timezone? But date/week digital easier.

Need decide. We can combine:
- Analog main dial with hands (local time).
- Digital screen at 6 o'clock / subdial area showing:
  - Local date and weekday (e.g., 2026-05-07 Thu)
  - Second timezone time (e.g., 东京 09:32:45)
This satisfies date/week and second timezone. But "以完整的3D腕表形式呈现" likely okay.

Need include two timezones. Need choose second timezone. Could be configurable with a variable offset seconds. User can adjust? We can define SECOND_TIMEZONE_OFFSET = 9 hours (Tokyo) with label "TKY". But for test, we need ability to demonstrate. Maybe include UI controls: timezone offset select, pause/resume, speed, rotate/camera. But if "自动运行 current time" no pause? Could include controls anyway.

Need perhaps "同时追踪两个时区" means main timezone and another timezone can be selected. We can implement a dropdown to choose second timezone from common cities, with offset from local. But if local timezone is system, need compute offset from UTC? We can use Intl or Date.getTimezoneOffset. For second timezone, use offsetSeconds from local. Need support two timezones. We can define list of city offsets from UTC. Need compute current local time? Since Date gives local. For main time, use local date/time. For second, use Date.now() + offsetDifferenceMs. But if user chooses timezone, display its time. Need label.

Need date/week display. For main local date and weekday. For second timezone maybe no need date? But could display date also? We can show "日期 星期" and "第二时区 时间". Need include timezone names.

Need update every frame (smooth hands). For smooth second hand, compute seconds fraction: Date.now()/1000. For main local time, use new Date(). Hours/min/sec/millis. Need set rotation of hands. Need date and weekday update. Use requestAnimationFrame.

Need 3D aesthetic: use mesh materials, metalness, roughness, envMap? Need maybe use MeshStandardMaterial with environment map? Without env map, metals may look dark. Need include ambient and point lights. Could use MeshPhongMaterial with specular for metal. For metal watch, use MeshPhongMaterial ambient/color/specular. Need maybe "silver" color.

Need include strap. Could create two straps: top and bottom around case. Use BoxGeometry with slight curve? Simpler: two boxes extending along Y? Wait watch face normal Z, strap should extend along vertical (Y) from top and bottom of case. Build strap in XY? Let's choose watch dial at center, face normal Z. Strap length along Y? A wristwatch's strap goes up/down relative to watch face. In our coordinate, Y vertical, X horizontal, Z thickness. The strap is a flat band extending along Y. We can create BoxGeometry(width=1.6, length=3.5, thickness=0.35), place behind case extending upward and downward? Need a watch band passes through around wrist. We'll make two strap segments: top from y=2 to 4? Let's think case radius 2, so top edge at y=2. Strap top: from y=1.5 to y=4.5, width 1.6, thickness 0.4, at z=-0.1? But watch case occupies z=±0.3. The strap should attach at back (z=-0.2) and extend outward. We can create a U shape? Maybe too complex. Use two rectangular boxes:
- Upper strap: BoxGeometry(1.6, 3.2, 0.4), position y=3.2, z=-0.3? Wait watch group centered at origin. If strap extends from top of case (y=1.8) upward, center y = 1.8 + 1.6 = 3.4. The thickness in z should align at z=-0.2? Actually watch case front face z=0.3, back z=-0.3. Strap attaches to lugs? Could be behind case, z=0 (center). Need at z=-0.1 to sit behind case. But if strap is in middle z=0.2, it might intersect case. We can just set z=-0.15 and let intersection hidden? But case cylinder has full circle. Strap would be visible beyond case perimeter. Good.

- Lower strap: position y=-3.4, z=-0.15? Since case radius 2, strap starts at y=-2? Need position y=-3.4. Width 1.6, length 3.2. It extends down.

Need strap material: leather brown or black. Use MeshPhongMaterial color #3b2b20? Nice.

Need lugs? Could create small boxes connecting strap to case at top/bottom:
- Lug top: BoxGeometry(0.8, 0.8, 0.6), position y=2.05, z=0. Maybe from case edge to strap. But not necessary.

Need crown: small cylinder on right side. Since case center at origin, crown at x=2.0, z=0.1? Could create CylinderGeometry(0.2,0.2,0.5,16) rotated z? Need axis along x? We can create a small cylinder along x? Default axis y. Rotate z by PI/2? Actually to align axis x, rotate y? Simpler use CylinderGeometry(0.2,0.2,0.6,16). rotateZ(Math.PI/2). Place x=2.1, y=0, z=0.0. Then crown extends outward along x. Need maybe two crown guards? Not necessary.

Need hands: Need create 3D hand shapes with plane geometry? Could use BoxGeometry with flattened dimensions. Need shape:
- Hour hand: elongated rectangular/triangle? BoxGeometry(width=0.18, height=1.1, depth=0.08). But need pivot at center; BoxGeometry centered, so if positioned at y=?? Actually watch face plane XY. A hand extends from center along negative? We need rotate around z? Let's define hand geometry extends along Y from center? In watch analog, hands rotate around center axis perpendicular to dial (Z). If hand is a rectangular bar along Y, rotation around Z changes angle. Need pivot at center. For a box centered at origin, its length extends both +Y and -Y. That's symmetric. We can instead use ShapeGeometry or use a box offset so extends from center to +Y? But rotation of a box centered at origin will have tail. Fine for hour/minute if tail is okay, but looks like a bat shape. Could use THREE.ShapeGeometry with polygon shaped like a hand along +Y and center at origin? Need pivot at origin. A hand shape can be a long thin triangle/rectangle with base at center and tip at +Y. Then rotate around Z. This is better. Need create shape with vertices.

Function createHandMesh({length, width, thickness, color, tail=0.2}) using Shape:
- Shape vertices (in XY plane): (width/2, 0), (width/4, tail), (-width/4, tail), (-width/2, 0), (-width/6, -tail?) Wait shape should be in XY plane. But THREE.ShapeGeometry creates geometry in XY plane. It will have face normal +Z. Then rotate? We can position at z=0.4. Need face plane XY. ShapeGeometry is in XY plane, yes. It has vertices (x,y,0). Good. But for a hand extending along Y, pivot at origin, shape like:
   (0, 0) maybe base? Need define polygon:
   (-w/2, -0.1), (-w/4, length), (w/4, length), (w/2, -0.1), (0, -0.25) maybe tail. Shape can be non-convex. Let's use a nice hand shape:
    start at (0, -0.2) tail,
    (-w/2, 0), (-w*0.3, length*0.7), (-w/4, length), (w/4, length), (w*0.3, length*0.7), (w/2, 0).
Need shape is simple closed. ShapeGeometry requires vertices in order, can be concave. Good.

But ShapeGeometry extrude? For 3D thickness, can use ExtrudeGeometry with depth 0.04 and bevel? That creates depth along Z. Since shape lies XY, extrude along Z. But ExtrudeGeometry depth means centered? It extrudes from z=0 to z=depth. Could set depth=0.04, then translate z-0.02 to center. But ExtrudeGeometry includes bevel maybe. Simpler: use ShapeGeometry and material with side: DoubleSide, no thickness; plus maybe add a back? But we need 3D watch, hands should have thickness. Use ExtrudeGeometry with depth 0.05, bevel disabled. Need ensure face normal? ExtrudeGeometry creates front/back/sides. It may be okay. Need maybe rotate? Because shape lies XY, extruded along Z. This is exactly what we need. We can create geometry, translate z=-0.025 to center.

Need attach hand to pivot at origin. Shape vertices in XY coordinates around origin. ShapeGeometry/ExtrudeGeometry position at origin. Good. Need rotate around Z: handGroup.rotation.z = angle. In Three.js, angle measured from +Y? Wait rotation.z rotates X/Y. Shape originally extends along +Y if we define length along +Y. rotation.z = angle will rotate hand clockwise? For 0°, hand points +Y (12 o'clock). At angle = PI/2, +Y maps to -X? Let's compute rotation.z(theta): x' = x cos - y sin; y'=x sin + y cos. Point (0,1): x'=-sin theta, y'=cos theta. At theta=PI/2, point (-1,0), which is 9 o'clock. But analog clock 3 o'clock is angle? Three.js angle increases counterclockwise. Clockwise rotation uses negative. For time hands, angle = - (minutes/60)*2π? Actually at 0 -> up, 15 minutes -> right. A 15-minute angle should be (15/60)*2π = PI/2. But rotation.z(PI/2) maps up to left (9), wrong. rotation.z(-PI/2) maps up to right. So use negative for clockwise. Or shape can extend along +X? no. So handGroup.rotation.z = - (time fraction * 2π). For hours: - ( (h%12 + m/60)/12 ) * 2π. Minute: - (m/60 + s/3600) * 2π. Second: - (s + ms/1000)/60 * 2π.

Need second hand smooth. Use Date.now() with fractional seconds.

Need date and weekday display: Dynamic canvas textures. Options:
A. Create one canvas "screen" at upper middle? Use a plane with CanvasTexture, update text. Need maintain high quality. We can update every 0.1 s perhaps. But if update every frame, it's okay. Need not flicker. Use context and clear. Create plane/square with material map. Need update map.needsUpdate = true. The plane can be part of dial at z=0.33. Need text legible.

Could create one "digital screen" at bottom half of dial (y=-0.8) with width 2.2, height 1.2. It displays:
- Line 1: "TKY 09:32:45" (second timezone)
- Line 2: "2026-05-07 周四"
Maybe line 3: local? But main analog already shows local. Need maybe date/weekday must correspond to local date. We can show them. But if second timezone's date might differ, but we only need date/weekday for current? It says "还可以提供日期和星期显示功能" likely for local. We can show local date and weekday.

Need maybe also show "第二时区" at subdial. Could use a small digital screen under 12? Let's design aesthetically:
- Dial texture: white/cream with tick marks, brand "AUTOMATIC 3D".
- Date window at 3 o'clock? Traditional watches have a date window at 3. But digital screen can simulate? We can make a window with black text "26" and weekday "THU". Need dynamic. But easier as a small plane overlay at 3 o'clock? Let's consider.

Traditional watches have date and day windows at 3 o'clock. We can implement date and weekday as two small meshes with canvas textures:
- date window at x=0.75, y=0.35, z=0.33? But dial texture can have window border. We can place two small planes:
   - Date number (two digits) at x=0.6, y=0.45
   - Weekday (e.g., "THU") at x=0.95, y=0.45? Need maybe adjacent. But in 3D, with a window background plane and dynamic canvas overlay. Need not be too small. Could use a single plane at x=0.55, y=0.4 with date and weekday text. Need scale. But if user zooms, text may blur. CanvasTexture with high resolution enough.

Need two timezone tracking. Could be an additional digital screen at 6 o'clock. But if we already have date/weekday window at 3, we can put second timezone in a small LCD at 6. Then date/weekday separate. Good.

Need perhaps "second timezone" can be "GMT hand" analog? But digital display is simpler. However, the prompt says "以完整的 3D 腕表形式呈现" not "only analog". We can include digital subdial. Need maybe include a second timezone "subdial" on watch face with a tiny hour hand? Could be visually impressive. But implementing two analog subdials plus date windows is harder but possible. Let's evaluate.

Alternative: Create a "world timer" with main dial plus a 24-hour ring and a GMT hand. But two timezones: local time and UTC offset. Hmm.

Maybe we can implement a second timezone analog via a small subdial at 6 o'clock. It has its own hour/min/sec? Need "simultaneously track two timezones" maybe a small dial with two hands. We can create a subdial plane and tiny hands, update rotations with offset time. That would be more "watch-like" than digital. But date/weekday still need display. Could include date window at 3 and weekday at 9. But implementing in 3D with multiple meshes is doable.

Let's decide. Need produce robust code. A digital screen is easier to maintain, but maybe less "watch". The prompt says "3D腕表动画" and "平滑转动的秒针、分针和时针", so main analog hands are key. Date/week and second timezone can be digital. I think acceptable.

Need maybe include "smooth" second hand: Use continuous rotation, not tick. Need ensure watch hands rotate based on Date.now.

Need support current time. Need use Date object. For two timezones, need compute offset. Let's design code.

HTML structure:
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<title>3D 腕表</title>
<style>
body { margin:0; overflow:hidden; font-family: sans-serif; background:#111; color:white; }
#info { position:absolute; top:10px; left:10px; z-index:10; background:rgba(0,0,0,0.6); padding:10px; border-radius:8px; font-size:12px; }
#controls { position:absolute; bottom:10px; left:10px; z-index:10; background:rgba(0,0,0,0.6); padding:10px; border-radius:8px; }
select, button { padding:4px 8px; margin:4px; }
</style>
</head>
<body>
<div id="info">3D 腕表 | 拖动旋转 / 滚轮缩放</div>
<div id="controls">
  第二时区:
  <select id="tzSelect"> ... options ... </select>
  <label><input type="checkbox" id="syncCrown" checked> 自动旋转展示</label>
</div>
<script src="...three.min.js"></script>
<script src="...OrbitControls.js"></script>
<script>...our code...</script>
</body>
</html>

Need load Three.js from CDN. Need maybe "module" not. Use r128.

Need use OrbitControls. Need import: const controls = new THREE.OrbitControls(camera, renderer.domElement); controls.enableDamping=true; controls.dampingFactor=0.05; controls.target.set(0,0,0); controls.minDistance=3; controls.maxDistance=10.

Need renderer:
const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

Need camera:
const camera = new THREE.PerspectiveCamera(45, w/h, 0.1, 100);
camera.position.set(3.5, 2.5, 5.5);
camera.lookAt(0,0,0);

Lights:
- AmbientLight(0xffffff, 0.3)
- DirectionalLight(0xffffff, 0.8) position(5,5,8)
- DirectionalLight(0xffffff, 0.3) position(-3,-2,-4)
- maybe PointLight

Watch group:

function createWatch() {
 const watch = new THREE.Group();

 // Case body
 const caseGeo = new THREE.CylinderGeometry(2, 1.9, 0.7, 64);
 caseGeo.rotateX(Math.PI/2);
 const caseMat = new THREE.MeshPhongMaterial({ color:0xaaa9a6, specular:0x222222, shininess:80 });
 const caseMesh = new THREE.Mesh(caseGeo, caseMat);
 watch.add(caseMesh);

 // Back? But case geometry includes caps. Need case has front cap visible maybe. But dial covers front? We need a case ring. If CylinderGeometry with caps, front cap covers entire face. We can't have dial on top unless dial slightly above. The front cap is at z=0.35 (if height 0.7) and will be visible behind dial. That's okay, dial at z=0.36 smaller radius. But front cap metal across entire circle, not a ring. It will show only outside dial radius, since dial radius 1.85. Good. But it also blocks view of hands behind? Hands are at z>0.37. Fine. Need opening? No, dial covers all.

 // Bezel ring
 const bezelGeo = new THREE.TorusGeometry(1.95, 0.12, 16, 64);
 const bezelMat = new THREE.MeshPhongMaterial({ color:0x555, specular..., shininess:100 });
 const bezel = new THREE.Mesh(bezelGeo, bezelMat);
 bezel.position.z = 0.36; 
 watch.add(bezel);
 // Maybe glass? Transparent sphere? Could add a crystal glass cylinder above hands? But if not, okay.

 // Lugs? optional.

Need be careful: CylinderGeometry after rotate x? We should create and rotate. The top cap (front) material in CylinderGeometry is one material? We can pass array materials? Not necessary, all same. But if front cap is dark? Need maybe case material all same. Good.

Need dial:
- Static dial canvas texture. Create canvas size 1024x1024. We'll draw:
  - Background: radial gradient from #fff to #eee? Center maybe off-white. Need 1024x1024.
  - Draw minute markers: 60 small dots/ticks around center radius 800? Wait dial radius in world units 1.85 corresponds canvas 512? Need map. In canvas, center at 512,512. Radius for tick marks: 460? Let's design:
    const size=1024, c=512, R=480 (outer edge), rMinute=430, rHour=380 maybe.
    Outer ring maybe.
  - Draw hour markers as bars at every 30 deg. Need labels? We can draw numerals 12,3,6,9 perhaps. Use font "600 80px Arial". Add brand text.
  - Draw second track: 60 lines.
  - Draw date window? Could draw static rectangle at 3 o'clock, but dynamic date overlay will cover. We can draw a white rectangle or frame at position x=340? Wait date window at 3 o'clock relative center: in canvas, 3 o'clock is right of center, x = c + R*cos(0)? Coordinates: Y up? Canvas y downward. Need account. In Three.js dial plane XY, x right, y up. Canvas texture applied to CircleGeometry? Important: CanvasTexture coordinates? On a plane/Circle geometry, UVs map x? For CircleGeometry, UVs? Need ensure orientation. We can maybe use PlaneGeometry for dial (width=3.7,height=3.7) instead of CircleGeometry? But watch face is circular; plane would have square edges. Better use CircleGeometry. CircleGeometry UV mapping may rotate? Need not align exactly to we can ignore labels? Need test. But to avoid orientation issues, we can use a PlaneGeometry for dial and make dial circular by using circle texture with transparent corners? But watch dial should be circle; if a plane with transparent corner, corners invisible if transparent. In Three.js, transparency can be used. PlaneGeometry with CircleTexture transparent outside. That's easier for text orientation? PlaneGeometry UV maps straightforward with x right, y up? Actually PlaneGeometry UVs: (0,0) bottom-left, (1,1) top-right and vertices positions (-0.5,0.5? Need) based on standard. When we draw canvas, y downward but texture coordinate v up. CanvasTexture flipped? Three.js Texture flipsY defaults true for CanvasTexture? It will show correct if using context y down? Need know. Normally CanvasTexture uses flipY = true, so image is flipped vertically? Wait in Three.js, if texture.flipY = true, image bottom row maps to UV v=0? Canvas coordinate y=0 is top, so flipY true flips so canvas top maps to v=1? Let's not overcomplicate. We can set texture.flipY = false? Hmm.

Alternative: use SVG texture? Not needed. We can test orientation mentally:
- In Canvas 2d, coordinate: x right, y down.
- For texture applied to PlaneGeometry, UV (0,1) corresponds to top-left? Actually with flipY=true (default), texture's first row (top) maps to bottom? Wait WebGL uploads image with origin bottom-left? If flipY true, it flips the source along y, so the top row of canvas becomes bottom? Let's recall: In Three.js, CanvasTexture with flipY default true yields image not upside down when applied to geometry, because canvas y-down is converted. In common use, drawing text on canvas and using as texture on plane displays correctly. Thus if we draw at canvas y=c-R, which is above center? Wait canvas y increases downward, so a point at 12 o'clock should be at y = c - R (smaller y). When flipY true, the top of canvas maps to top of texture? I think it maps correctly. So fine.

Maybe use PlaneGeometry for dial and a circular texture with transparent corners. Circle texture created on canvas with transparent outside. Use material transparent=true. Then we can overlay dynamic screens with PlaneGeometry. Hands are on top. This is simpler. The watch case is circular; dial plane square transparent corners will not be visible due to alpha? Actually transparent corners let case cap show through, but if case cap is opaque, okay. Need render order if transparent: you must set depthWrite=false? But the dial plane at z=0.36, the case front cap behind; transparent corners will show the case cap. However transparent object with depthWrite false can allow case visible. Since plane is at z>case front cap, with depthWrite false, it won't occlude. But hand/other opaque objects behind? Need ensure render order. Simpler: Use CircleGeometry and avoid transparency. CircleGeometry with UV mapping maybe rotates but okay. But date window overlay with PlaneGeometry on a circular dial might have rectangular borders that extend outside dial? We can place small planes inside radius, no issue.

Need know CircleGeometry orientation. In Three.js, CircleGeometry lies in XY plane, starts at +X? The UVs maybe map such that theta=0 at +X. Need maybe texture orientation with 12 at top? We can rotate mesh or texture to align. Since canvas can be oriented regardless. Need just make tick marks; if off by some degrees, not big? But date/window overlay positions are 3D coords independent. We can adjust dialMesh.rotation.z maybe. We'll define dial texture with 12 at top. CircleGeometry UV maybe starts at +X? Actually CircleGeometry parameter thetaStart default 0, thetaLength 2PI. Vertices generated from +X axis? It has center and outer vertices. UVs likely assign around; texture orientation may be correct with 3 at right? Hmm.

Maybe easiest: Use PlaneGeometry (square) with dial texture as a circle drawn centered, transparent outside. Then no need CircleGeometry UV mapping. Since plane is square 3.7x3.7, texture image square, circle drawn inside. Transparent corners. Use material transparent true. Need render order? Set depthWrite false? Wait plane is opaque? If material has transparent true, Three.js renders transparent objects sorted. It might sort after hands? Actually hands are opaque, could render before dial; but dial is behind hands, should not matter? If transparent dial has depthWrite false, it may not write depth, hands will be composited anyway. Hands are at z>dial, depth test against case, not dial. If dial doesn't write depth, hands still visible. But hands are opaque and render before transparent? Transparent objects are rendered after opaque, so dial will blend over hands? That would be wrong if dial is behind hands. However transparent material depthWrite false but depthTest true, and it is rendered after opaque hands; because dial is behind hands, depth test compares depth with hand in depth buffer. Since hands wrote depth, dial fragments behind hand fail depth test, so okay. dial doesn't write depth but depth test works. So fine. Need ensure dial doesn't blend over hands incorrectly; depth test prevents. Good.

But transparent dial with depthWrite false may not occlude hands behind dial? There are no hands behind. Fine.

Need position dial plane at z=0.365. Circle drawn radius 930 px? Plane width 3.7 world, texture 1024 px. To map circle radius 1.85, circle radius in px 512 (half of 1024). Use c=512, radius=500? Need if plane size 3.7, half=1.85. So texture circle radius 512 px exactly. To avoid edge, use 500. Good.

Need static dial texture:
- Canvas 1024x1024.
- fill transparent.
- Draw radial gradient circle centered 512,512 radius 500.
- Draw minute track.
- Draw hour markers (12, 1, 2, ...). Use positions:
  For each hour i 0..11 angle = i*30 - 90 degrees (3 o'clock? Need 12 at top). In canvas coords:
   x = c + R * cos(rad)
   y = c + R * sin(rad)
  But canvas y down means positive sin is down, so 12 at angle -90. Good.
  Draw line from r1 to r2. For hour markers, draw thick lines; for minute markers, thin lines.
  Maybe draw small date window frame at 3 o'clock (x=c+?? y=c). But if dynamic overlay at 3 o'clock, we can draw a white rectangle at x offset 110? Need in plane coordinates, 3 o'clock is +X. In canvas, x = c + 250? Wait plane width 3.7, texture circle radius 512. We can place date window at world x=0.75, y=0.3. Since canvas x maps to world X: center 0 at c, scale 512px per 1.85 world. So 0.75 world = 0.75/1.85*512 ≈ 207 px. So date window center around (c+207, c - 0.3/1.85*512? Y up world, canvas y down: world y positive maps to canvas y = c - y/1.85*512. So y=0.3 -> c - 83. Thus draw window at x=719, y=429. But dynamic overlay plane will be at matching world coords. Need not draw static frame if dynamic plane has background. Let's draw a rectangular frame maybe.

Need dynamic date/weekday display:
- Create a plane mesh with its own CanvasTexture. We can place at world position (0.78, 0.28, 0.365?) But if plane is same z as dial, z-fighting. Place slightly in front: z=0.367. Need plane size maybe 1.0 x 0.45. Use MeshBasicMaterial({map, transparent:true, depthWrite:false}). Update canvas text every few seconds.

But if we use a plane at z=0.367 and transparent black background, it will display date and weekday. Need not align with static texture if we don't draw static frame. We can draw frame on dynamic canvas. Good.

Need second timezone display:
- Another plane at (0, -0.85, 0.367) with size 2.0 x 1.0 maybe. It can be a black LCD window with text. Use CanvasTexture.

Need maybe main digital screen at bottom covers too much; okay.

Need maybe "analog subdial" for date? Actually date and weekday can be in date window at 3 o'clock. Good.

Need compute date/weekday:
const date = new Date();
const year = date.getFullYear();
const month = date.getMonth()+1;
const day = date.getDate();
const weekdays = ['SUNDAY','MONDAY','TUESDAY','WEDNESDAY','THURSDAY','FRIDAY','SATURDAY'];
const weekday = weekdays[date.getDay()];
Need display maybe "05/07 THU" or "2026-05-07 周四".

If second timezone selected, need offset. We can implement list:
const timeZones = [
 { label:'东京', offset:9 },
 { label:'伦敦', offset:0 },
 { label:'纽约', offset:-4? Actually current offset depends DST. Better use IANA names and Intl. But in browser, can compute by UTC offset method? Need timezones with DST. Simpler: use option "UTC+09:00 东京", "UTC+08:00 北京", "UTC+05:30 印度", "UTC+00:00 伦敦", "UTC-05:00 纽约", "UTC-08:00 洛杉矶". We can just use fixed offsets. But for current time, real DST not reflected. That's okay for demo? User likely wants current time; if second timezone uses fixed offset, may be off by DST. Could use Intl.DateTimeFormat with timeZone IANA to get current time parts. Then compute date/time. This is more robust. Let's implement timezone display via Intl? Need main local uses Date. For second timezone, we can get parts for desired timezone using Intl.DateTimeFormat every frame (maybe expensive but okay at 60fps? Could optimize update 4 times/s). Better compute offset once per second? Need hands second timezone not necessarily smooth? If digital only, update once per second acceptable. For main analog smooth. For date/week update once per second or minute. Use requestAnimationFrame but update dynamic textures at ~10 Hz. Good.

Need maybe date in second timezone? display "TKY 09:32:45" maybe enough. If date differs, maybe no.

Need support "同时追踪两个时区" with offset list. Use Intl:
function getTimeInZone(timeZoneName) {
 const now = new Date();
 let parts = new Intl.DateTimeFormat('en-GB', { timeZone: timeZoneName, hour12:false, hourCycle:'h23', hour:'2-digit', minute:'2-digit', second:'2-digit', year:'numeric', month:'2-digit', day:'2-digit', weekday:'short' }).formatToParts(now);
 // extract
}
Need be cautious `hourCycle` support. But should. Need use for digital display. For main analog date, use local Date.

Need second timezone dropdown includes IANA names:
- 北京 Asia/Shanghai (UTC+8)
- 东京 Asia/Tokyo (UTC+9)
- 伦敦 Europe/London (UTC±0/1)
- 纽约 America/New_York
- 洛杉矶 America/Los_Angeles
- 悉尼 Australia/Sydney
- 巴黎 Europe/Paris
- 迪拜 Asia/Dubai
- 莫斯科 Europe/Moscow
- 新加坡 Asia/Singapore
Need default Asia/Tokyo.

Need if `formatToParts` returns hour "24"? Use hourCycle:'h23' to avoid 24. Good.

Need create dynamic LCD texture:
function createTextTexture(width=256,height=128) { canvas... }
LCD plane for second timezone:
- size world 2.1 x 0.9 maybe. Canvas 512x256.
- Draw black rounded rectangle background; draw green/amber text.
- Update at interval 100ms.
- Use material.map = texture; material.map.needsUpdate = true.

Need update date window similarly:
- size world 1.1 x 0.5, canvas 256x128.
- Background white/cream? Actually date window on analog watch usually white with black text. Could draw background #fff, border #999. Text "26 THU" black. Use font.

Need place date window at 3 o'clock? On watch, date window at 3 is common. But if watch has digital screen at 6 and date at 3, all good. Need ensure date window doesn't overlap hour markers. At x=0.75, y=0.3 maybe. Date window 1.1 wide, 0.5 high. World x ranges 0.2 to 1.3, y 0.05 to 0.55. It may overlap 3 o'clock marker at x=1.85, y=0. Too far. Overlaps 2/4 markers? Fine. good.

Need second timezone LCD at 6 o'clock: Place at (0, -1.0, z=0.37) size 2.0 x 0.85. But at bottom, hour markers around 5,6,7. LCD ranges y -1.425 to -0.575, x -1 to 1. It may overlap 6 o'clock marker at (0,-1.85) not much; 5/7 markers at x ±0.925, y -1.60? Wait marker at 7 o'clock: x = R cos(210°=-0.866*R), y = R sin(210°=-0.5*R) if R=1.85 => x=-1.60, y=-0.925. LCD left edge -1.0, so no overlap. 5 o'clock x=1.60, no. Good.

Need maybe second timezone display should not obscure date? okay.

Need 3D hands:
- Hour hand length 0.95, width 0.16, thickness 0.04, color #1a1a1a (dark).
- Minute hand length 1.4, width 0.12, thickness 0.035, color #222.
- Second hand length 1.6, width 0.04, thickness 0.02, color red/rose gold.

Need create hand group centered at z=0.4 (above dial). Need maybe a center cap: small cylinder/cylinder along z? Use CylinderGeometry(0.08, 0.08, 0.1, 16), default axis y, rotate x=PI/2 to axis z, place at z=0.42, material metal. Also maybe a second cap for seconds.

Need hand z-order:
- Hour hand at z=0.38
- Minute hand at z=0.40
- Second hand at z=0.42
Need if using ExtrudeGeometry with depth 0.05, translate z=-0.025, then hand.mesh.position.z = desired. Actually geometry extruded along z; if set position.z=0.38, it extends from 0.355 to 0.405? Wait depth 0.04, translate -0.02, so geometry occupies [-0.02,0.02] relative to position. So if position.z=0.38, it occupies 0.36-0.40. Minute position 0.40, occupies 0.38-0.42? Need avoid intersect. Use positions:
  dial at 0.365
  hour at 0.39 (occupies 0.37-0.41)
  minute at 0.42 (0.40-0.44)
  second at 0.46 (0.44-0.48)
Need center cap at 0.47. Looks okay.

Need maybe add glass lens: A transparent sphere section or disc above hands. Could use CircleGeometry/PlaneGeometry with MeshPhysicalMaterial transparent opacity 0.1. Not necessary but adds realism. Let's add a simple "crystal" cylinder? If use CylinderGeometry with transparent, could distort? No. Maybe skip.

Need "完整3D腕表形式" includes lugs, crown, strap. Let's implement.

Detailed construction:
const watch = new THREE.Group();

// Case
const caseMat = new THREE.MeshPhongMaterial({ color: 0xbfc3c7, specular: 0x444444, shininess: 90 });
const caseMesh = new THREE.Mesh(new THREE.CylinderGeometry(2, 2, 0.7, 64), caseMat);
caseMesh.rotation.x = Math.PI / 2;
watch.add(caseMesh);

Wait cylinder radius top=2, bottom=1.9 maybe tapered. If top radius 2 (front), bottom radius 1.9 (back), the side slopes. But after rotation x, top cap (radius 2) at z=+0.35, bottom at z=-0.35. Good. Need material on caps maybe same. okay.

// Case back? Maybe add a darker back disc? Not visible.

// Bezel
const bezelMat = new THREE.MeshPhongMaterial({ color:0x8a8a8a, specular:0x333, shininess:100 });
const bezel = new THREE.Mesh(new THREE.TorusGeometry(1.99, 0.10, 16, 64), bezelMat);
bezel.rotation.x = Math.PI / 2? Wait TorusGeometry default lies in XY plane, axis Z. If we want ring around front of cylinder, no rotation needed. Place z=0.35? Actually torus's central circle in XY plane, so it is a ring around z axis. Good. But if no rotation, it lies in XY plane, exactly around the case front? At z=0.35, radius 1.99. Good. However TorusGeometry tube extends in all directions, so central radius 1.99, tube radius 0.10, it will occupy z=0.25 to 0.45. Good. No rotation needed.
bezel.position.z = 0.26? Wait case front cap at z=0.35. If bezel at z=0.35, half behind cap, half in front. fine. Use z=0.25? Let's set z=0.22? Hmm. The tube in z extends ±0.10. If z=0.25, back at 0.15, front at 0.35. The dial plane at 0.365 would be in front of bezel? Actually bezel front at 0.35, dial at 0.365, slightly above. Good. If bezel at z=0.25, front at 0.35, dial 0.365 just in front. Nice.

But case front cap at z=0.35. Bezel at z=0.25? Tube front 0.35, matching cap. Good. But torus tube radius 0.10 means inner radius 1.89, outer 2.09. Case radius 2.0, okay. Dial radius maybe 1.85, so inner edge bezel covers dial edge. Good.

// Dial plane
const dialGeo = new THREE.PlaneGeometry(3.7, 3.7);
const dialMat = new THREE.MeshBasicMaterial({ map: createDialTexture(), transparent: true, side: THREE.DoubleSide });
const dial = new THREE.Mesh(dialGeo, dialMat);
dial.position.z = 0.365;
watch.add(dial);

Need createDialTexture: returns THREE.CanvasTexture. We need maybe use `texture.anisotropy = renderer.capabilities.getMaxAnisotropy()`. Could set after scene.

// Date window overlay
const dateCanvas = document.createElement('canvas'); dateCanvas.width=256; dateCanvas.height=128;
const dateCtx=...
const dateTex = new THREE.CanvasTexture(dateCanvas);
const dateMat = new THREE.MeshBasicMaterial({ map: dateTex, transparent: true, depthWrite:false });
const dateMesh = new THREE.Mesh(new THREE.PlaneGeometry(1.1, 0.55), dateMat);
dateMesh.position.set(0.75, 0.35, 0.371);
watch.add(dateMesh);

But date texture drawn white background, so transparent false? Use transparent true with background. okay.

// Second timezone LCD
const lcdCanvas = document.createElement('canvas'); lcdCanvas.width=512; lcdCanvas.height=256;
const lcdTex = ...
const lcdMat = new THREE.MeshBasicMaterial({ map: lcdTex, transparent: true, depthWrite:false });
const lcdMesh = new THREE.Mesh(new THREE.PlaneGeometry(2.1, 1.05), lcdMat);
lcdMesh.position.set(0, -0.95, 0.372);
watch.add(lcdMesh);

Need perhaps no need static texture for date window if overlay has frame. Good.

Need create hands:
function createHand({length, width, thickness, color}) {
 const shape = new THREE.Shape();
 shape.moveTo(0, -0.25);
 shape.lineTo(-width/2, 0);
 shape.lineTo(-width*0.3, length*0.7);
 shape.lineTo(-width/4, length);
 shape.lineTo(width/4, length);
 shape.lineTo(width*0.3, length*0.7);
 shape.lineTo(width/2, 0);
 shape.lineTo(0, -0.25); // close
 const extrudeSettings = { depth: thickness, bevelEnabled: false };
 const geo = new THREE.ExtrudeGeometry(shape, extrudeSettings);
 geo.translate(0, 0, -thickness/2);
 const mat = new THREE.MeshPhongMaterial({ color: color, specular:0x222, shininess:60, side:THREE.DoubleSide });
 return new THREE.Mesh(geo, mat);
}
Need note shape with concave polygon? The shape is okay, but ExtrudeGeometry uses triangulation. Need ensure no self-intersection. Fine.
Set hand.rotation.z = -angle.
Need second hand maybe create a long thin line plus counterweight. Use same shape but length 1.6, width 0.05. Need shape tail maybe 0.35. Maybe shape with circular center? The shape starts at y=-0.25 tail, but second hand tail should extend opposite. It's okay.

But ExtrudeGeometry with shape from y=-0.25 to length 1.6, width 0.05 small. Good.

Need minute/hour positions:
const hourFraction = (h % 12) + m/60 + s/3600;
hourHand.rotation.z = -hourFraction/12 * 2π;
minuteHand.rotation.z = -(m + s/60 + ms/1000)/60 * 2π;
secondHand.rotation.z = -( (s + ms/1000) / 60 ) * 2π;
Need hour hand angle smooth? It moves smoothly with minutes/seconds too. Good.

Need maybe if current local timezone is not the main timezone? The main analog tracks current system time. Date/weekday display local. Second timezone selected.

Need maybe two time zones: The main analog is local or "HOME" timezone? We can label with "LOCAL" on dial. The LCD shows second timezone and its label. So tracking both.

Need maybe include "UTC offset" in LCD? no.

Need UI controls:
- `<select id="tz">` with options. On change set `currentTZ`.
- Maybe "暂停" button? User asked normal running. Could include "暂停/继续" for testing. But if paused, current time not real. Should we include? Could include controls for "演示模式" maybe. Need "测试说明" can say use timezone selector; main hands follow system time; if want set time, use browser? But no. Could add buttons:
  - `pauseBtn` toggles pause; when paused, use same value advancing by delta? We can implement `simTimeOffset`? Hmm.
Need "测试说明" in final. Maybe not require pause. But for testing, we need inspect date/week with current system time.

Maybe include a speed slider? Could help test hands. But if user wants normal running, default speed=1. We can add speed control to "timeScale" and pause. That can be useful. Need ensure "should run according to current time" default. If speed changed, not current time. We can still add for testing. But final "运行方式" says open HTML; default current time.

Need if adding pause/speed, need compute time. Use `performance.now()`? To maintain current time with pause/speed:
- `let timeScale = 1; let paused = false; let lastNow = Date.now(); let virtualNow = Date.now();`
But simpler use Date.now() every frame. If speed control, need simulate. But we don't need speed. However for "smooth" we need continuous. Use Date.now().

Need maybe if the page remains open, `Date.now()` gives current. Good.

Need update dynamic texture:
function updateDisplays() {
 const now = new Date();
 // local date/week
 const localDateStr = `${now.getFullYear()}-${pad(now.getMonth()+1)}-${pad(now.getDate())}`;
 const weekDayStr = weekdaysCN[now.getDay()]; // "周四"
 const localTimeStr = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`;
 // second timezone
 const tzParts = getTimeInZone(currentTZ);
 const tzTime = `${tzParts.hour}:${tzParts.minute}:${tzParts.second}`;
 // update date canvas
 dateCtx.clearRect...
 // draw white bg, border, text: `${localDateStr}\n${weekDayStr}`? But date window small; maybe show `24 THU` two lines? Let's design:
   font 'bold 38px monospace'; fill black; textAlign center; textBaseline middle.
   `dateCtx.fillText(`${pad(now.getDate())} ${weekdayShortEN}`, 128, 64);
   // maybe month/day? Use Date "05/07 THU"?
 Need "日期和星期显示" can be date string and weekday. `2026-05-07 周四` is long. Date window width 1.1 world corresponds 256 px; font 30 maybe. We can use two lines: "05-07" and "THU". Better.
 date canvas 256x128. Draw background white, border.
 dateCtx.fillText(`${pad(month)}-${pad(day)}`, 128, 42);
 dateCtx.fillText(weekdayShortEN.toUpperCase(), 128, 86);
 Need maybe not include year. Good.
 // update lcd:
 lcdCtx.clearRect...
 // black bg with green text
 lcdCtx.fillStyle = '#0a0a0a'; fillRect with border.
 // labels:
 lcdCtx.fillStyle = '#9f9'; font='bold 22px monospace'; textAlign='left';
 lcdCtx.fillText(tzLabel + ' ' + tzTime, 20, 40);
 // second line? maybe local date? Wait date already. Maybe second timezone date? If different, could display `TZ DATE`.
 // lcdCtx.fillText(`${tzDate} ${tzWeekday}`, 20, 80)
 // Or show "LOCAL HOME" and second time? But main analog local.
 Let's show:
 line1: `HOME ${localTimeStr}`? But main analog already. Hmm.
 Need maybe LCD should display second timezone only. Let's do:
   Line 1: `${tzLabel} ${tzTime}` (e.g., "TKY 10:32:45")
   Line 2: `${tzDate} ${tzWeekday}` (e.g., "05/07 TUE")
 Need tzDate/weekday from Intl.

But the prompt wants date and week display. date window handles local date; LCD line2 handles second timezone's date. Good.

Need update lcd every frame? Since text update is cheap. But `formatToParts` each frame at 60fps may be heavier. We can update displays at interval 100ms. Hands update every frame. Good.

Need `updateFrame`:
let lastDisplayUpdate = 0;
function animate(t) {
 requestAnimationFrame(animate);
 const nowMs = Date.now();
 // update hands
 updateHands(nowMs);
 // update displays at 5 Hz
 if (nowMs - lastDisplayUpdate > 100) { updateDisplays(); lastDisplayUpdate = nowMs; }
 // controls.update();
 renderer.render(scene, camera);
}
Need ensure updateHands uses `nowMs`; no need Date again.

Need `updateHands(nowMs)`:
const date = new Date(nowMs);
const h=date.getHours(), m=date.getMinutes(), s=date.getSeconds(), ms=date.getMilliseconds();
const sec = s + ms/1000;
const min = m + sec/60;
const hr = h % 12 + min/60;
hour.rotation.z = - (hr/12) * 2π;
minute.rotation.z = - (min/60) * 2π;
second.rotation.z = - (sec/60) * 2π;
Maybe hour should move only with minutes, not seconds? Smooth is okay.
Need maybe second hand should be stainless steel with red tip? Use shape.

Need date and weekday:
const weekdaysEN = ['SUN','MON','TUE','WED','THU','FRI','SAT'];
const weekdaysCN = ['周日','周一','周二','周三','周四','周五','周六']; Perhaps display "周四" in date window? The prompt Chinese, but watch aesthetics maybe English. We can show "THU" on date window; in LCD maybe "周四" for second timezone? Need "日期和星期" in Chinese? The page can have Chinese UI, watch dial can use English/Chinese. Let's use Chinese in LCD maybe. In date window, use "05-07 周四" perhaps. Need font supports Chinese. In Canvas, use "bold 28px 'Microsoft YaHei', sans-serif". For date window, 256x128, text "05-07" and "周四" works. Maybe okay.

Need if using system timezone offset, we need maybe date window local date. Good.

Need second timezone with Intl:
function getTimeZoneInfo(tzName) {
 const now = new Date();
 const parts = new Intl.DateTimeFormat('zh-CN', {
   timeZone: tzName, hour12:false, hourCycle:'h23',
   year:'numeric', month:'2-digit', day:'2-digit',
   hour:'2-digit', minute:'2-digit', second:'2-digit',
   weekday:'short'
 }).formatToParts(now);
 const out={};
 for (const p of parts) out[p.type]=p.value;
 return out;
}
Need format hour maybe "24" with hourCycle h23 should not.

Need second timezone label from select:
<option value="Asia/Tokyo" data-label="东京">东京 (Asia/Tokyo)</option>
Need get label:
select.selectedOptions[0].getAttribute('data-label') || 'TZ'.

Need "自动旋转展示" checkbox. If checked, watch.rotation.y += 0.003 in animate. But if user wants inspect, checkbox can disable. Need if OrbitControls uses target, watch group rotation okay. However watch.rotation.y rotates about Y; with face normal +Z, the watch will spin around vertical axis, showing side/back and front. This is nice. But if user rotates camera, maybe also? It's okay. Need "完整 3D" showcase.
Need maybe if autoRotate true, watch rotates; controls not. We can use `watch.rotation.y += 0.003`. Good.
Need checkbox default? Maybe false to avoid dizzy; or true to showcase. Let's set true default? The prompt wants animation; auto rotate can show 3D. But if user wants inspect, disable. Let's set true? Hmm. If auto rotating, main watch face not always front, but still. Let's default checked to show "full 3D". User can uncheck.

Need maybe also auto-tilt? `watch.rotation.x = -0.2`? We can set initial rotation x = -0.15? But if autoRotate y only, watch face normal z after rotation may not point camera. Camera position (3.5,2.5,5.5) looks at center; watch face initially normal +Z, so camera sees face at angle. Good. Auto-rotate y will show sides. Nice.
Need maybe set watch.rotation.x = 0.1? not needed.

Need environment for shadows. Maybe add a ground plane? The watch is floating. Could add a circular "display pedestal" and maybe a subtle shadow. But "3D wristwatch" maybe floating in space okay. We can add a simple ground/table? But not needed. Could add a large sphere? no.

Need perhaps add a "display case" with glass? no.

Need ensure case material not too dark. With MeshPhongMaterial and lights.

Need maybe add environment map to make metal shiny? Can use `THREE.CubeTextureLoader` from CDN? No. Could use simple lights.

Need maybe use `THREE.PMREMGenerator` with a scene? Too much. Use MeshPhong.

Need create all. Let's plan code in detail.

Outline:
```
let currentTZ = 'Asia/Tokyo';
const tzLabels = { 'Asia/Tokyo':'东京', ... };
let autoRotate = true;
let clockGroup;

init();
animate();

function init() {
 // renderer...
 // scene, camera, controls...
 // lights...
 // clockGroup = createWatch();
 // scene.add(clockGroup);
 // UI listeners...
 // window resize...
}
```

Need createWatch returns group.

Need createDialTexture function:
```
function createDialTexture() {
 const canvas = document.createElement('canvas');
 canvas.width = canvas.height = 1024;
 const ctx = canvas.getContext('2d');
 // transparent outside
 ctx.clearRect(0,0,1024,1024);
 // draw circle with gradient
 const grad = ctx.createRadialGradient(512,512,200,512,512,510);
 grad.addColorStop(0,'#fffdf7');
 grad.addColorStop(0.8,'#f5f2ea');
 grad.addColorStop(1,'#ddd8ce');
 ctx.beginPath(); ctx.arc(512,512,500,0,2*Math.PI); ctx.fillStyle=grad; ctx.fill();
 // outer ring
 ctx.lineWidth=8; ctx.strokeStyle='#333'; ctx.beginPath(); ctx.arc(512,512,470,0,2*Math.PI); ctx.stroke();
 // minute ticks
 for (let i=0; i<60; i++) {
   const ang = i*Math.PI/30 - Math.PI/2;
   const r1=440, r2= i%5===0 ? 400 : 420;
   ctx.beginPath(); ctx.moveTo(512+Math.cos(ang)*r1, 512+Math.sin(ang)*r1); ctx.lineTo(...); ctx.strokeStyle='#555'; ctx.lineWidth= i%5===0 ? 6 : 2; ctx.stroke();
 }
 // hour labels 12,3,6,9? Maybe draw all numbers? For watch, only 12,3,6,9 enough.
 ctx.font='bold 80px Arial'; ctx.fillStyle='#222'; ctx.textAlign='center'; ctx.textBaseline='middle';
 const numbers = { '12':300, '3':300, '6':300, '9':300 }; // maybe all 12? Let's draw all hours? At 1024, numerals 12 small. Need positions: R=330? If R=300, not too close.
 for (let i=1; i<=12; i++) {
   const ang = i*Math.PI/6 - Math.PI/2;
   const x = 512 + Math.cos(ang)*300;
   const y = 512 + Math.sin(ang)*300;
   ctx.fillText(String(i), x, y);
 }
 // brand text
 ctx.font='bold 36px Arial'; ctx.fillStyle='#333'; ctx.fillText('AUTOMATIC',512,80);
 ctx.font='26px Arial'; ctx.fillStyle='#777'; ctx.fillText('3D EDITION',512,520?); 
 // Actually center area should be clear for hands. Put brand below center? Could place at y=620 maybe.
 ctx.fillText('SWISS 3D',512,650);
 const texture = new THREE.CanvasTexture(canvas);
 texture.anisotropy = renderer.capabilities.getMaxAnisotropy();
 return texture;
}
```
Need note CanvasTexture flipY? If draw all hour numbers, if texture appears mirrored? Need test. We can set `texture.flipY = false`? Hmm.

Let's reason carefully. Three.js `CanvasTexture` uses `Texture` with flipY = true by default. That means the canvas's top-left pixel is mapped to texture coordinate (0,1) or (0,0)? Need not know because PlaneGeometry orientation with position (x,y). In many examples:
```
const texture = new THREE.CanvasTexture(canvas);
const material = new THREE.MeshBasicMaterial({ map: texture });
const geometry = new THREE.PlaneGeometry(1,1);
```
Text appears correct. So use default. If not, we can if text reversed? But likely fine.

Need if using `PlaneGeometry` with texture, the texture is applied such that positive v corresponds to y. Canvas default flipY=true flips image, so canvas Y down becomes correct. Good.

Need dynamic textures also default flipY. So drawing uses canvas coordinates and displays correct.

Need `dateTex` and `lcdTex` with flipY default.

Need create date window overlay:
Since dial texture has hour numbers, and date window is overlay at (0.75,0.35). The overlay plane background white will cover part of numbers. Good.
Need if `MeshBasicMaterial` with transparent true and depthWrite false, it renders after opaque hands? It might appear over hands because transparent sorted by distance and no depthWrite. But if it is behind hands, depth test should fail. But since depthWrite false for overlay, it doesn't write depth. Hands write depth anyway. Transparent overlay rendered after opaque hands. The overlay is behind hands, so depth test fails where hand is in front. Good.

Need if overlay at z=0.371 is just before hands; okay.

Need update date texture:
```
function pad(n){return String(n).padStart(2,'0');}
function updateDisplays() {
  const now = new Date();
  const month = now.getMonth()+1;
  const day = now.getDate();
  const weekdayCN = weekdaysCN[now.getDay()];
  // date
  const ctx = dateCtx;
  ctx.clearRect(0,0,256,128);
  ctx.fillStyle = '#fffdf6'; ctx.fillRect(4,4,248,120);
  ctx.strokeStyle='#333'; ctx.lineWidth=2; ctx.strokeRect(4,4,248,120);
  ctx.fillStyle='#111'; ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.font='bold 34px monospace'; ctx.fillText(`${pad(month)}-${pad(day)}`,128,44);
  ctx.font='bold 28px "Microsoft YaHei", sans-serif'; ctx.fillText(weekdayCN,128,88);
  dateTex.needsUpdate = true;
  // lcd
  const lc = lcdCtx;
  lc.clearRect(0,0,512,256);
  lc.fillStyle='#e0e0e0'; lc.fillRect(4,4,504,248);
  lc.fillStyle='#050505'; lc.fillRect(8,8,496,240);
  lc.fillStyle='#9ff09f'; lc.font='bold 28px monospace'; lc.textAlign='left'; lc.textBaseline='middle';
  lc.fillText(`${tzLabel}  ${tzTime}`, 28, 44);
  lc.font='bold 24px monospace'; lc.fillText(`${tzDate} ${tzWeekday}`,28,120);
  lc.fillStyle='#666'; lc.font='20px monospace'; lc.fillText('SECOND TIMEZONE',28,190);
  lcdTex.needsUpdate = true;
}
```
Need maybe LCD canvas green glow? Add shadow in ctx:
```
lc.shadowColor='#0f0'; lc.shadowBlur=10; ...
```
Could be cool.

Need getTimeZoneInfo:
```
function getTimeZoneInfo(timeZone) {
 const now = new Date();
 const parts = new Intl.DateTimeFormat('zh-CN', {
   timeZone, hour12:false, hourCycle:'h23',
   hour:'2-digit', minute:'2-digit', second:'2-digit',
   month:'2-digit', day:'2-digit', weekday:'short'
 }).formatToParts(now);
 const obj = {};
 for (const {type,value} of parts) obj[type]=value;
 return obj;
}
```
Need if Date in timezone offset? Good.

Need timezone label:
```
const tzSelect = document.getElementById('tzSelect');
function getTZLabel(){ return tzSelect.options[tzSelect.selectedIndex].getAttribute('data-label') || tzSelect.value; }
```
Need when select change, maybe no need.

Need create Hands:
```
function createHandMesh(opts) {
 const shape = new THREE.Shape();
 const w=opts.width, len=opts.length;
 shape.moveTo(0, -len*0.12);
 shape.lineTo(-w/2, 0);
 shape.lineTo(-w*0.35, len*0.7);
 shape.lineTo(-w/4, len);
 shape.lineTo(w/4, len);
 shape.lineTo(w*0.35, len*0.7);
 shape.lineTo(w/2, 0);
 shape.lineTo(0, -len*0.12);
 // maybe close
 const geo = new THREE.ExtrudeGeometry(shape, {depth:opts.thickness, bevelEnabled:false});
 geo.translate(0,0,-opts.thickness/2);
 const mat = new THREE.MeshPhongMaterial({ color:opts.color, specular:0x222, shininess:70, side:THREE.DoubleSide });
 return new THREE.Mesh(geo, mat);
}
```
Potential issue: Shape needs `shape.closePath()`? Triangle/moveTo lineTo enough; `ExtrudeGeometry` automatically closes shape. Need perhaps Shape with vertices including a center? Good.

Maybe add a small hole? no.

Need center cap:
```
function createCap(r, z) {
 const geo = new THREE.CylinderGeometry(r, r, 0.1, 20);
 geo.rotateX(Math.PI/2);
 const mat = new THREE.MeshPhongMaterial({color:0x222, specular:0x444, shininess:100});
 const mesh = new THREE.Mesh(geo, mat);
 mesh.position.z = z;
 return mesh;
}
```
Need if cylinder rotated x, its height axis along z. Good. Place at z=0.47. But if cap radius 0.08 and height 0.1, covers hand pivots. Good.

Need create Crown:
```
const crown = new THREE.Mesh(new THREE.CylinderGeometry(0.16,0.16,0.5,16), caseMat);
crown.rotation.z = Math.PI/2;
crown.position.set(2.05, 0, 0.0);
watch.add(crown);
```
Wait default cylinder axis y. Rotate z by PI/2 maps y axis to x? Point (0,1,0) -> (1,0,0) for rotation z? R_z(90): x'=-y, y'=x, so (0,1,0) -> (-1,0,0). Axis becomes -x. Still crown extends along x. okay. Place at x=2.05? Cylinder length 0.5 along x, centered at 2.05, from 1.8 to 2.3. Case radius 2, so it protrudes. Good.
Need maybe also add second crown/buttons? not needed.

Need Lugs:
- Top lug: BoxGeometry(1.2, 0.7, 0.5). position (0,1.9, -0.1)
- Bottom lug: BoxGeometry(1.2,0.7,0.5). position (0,-1.9,-0.1)
But case at top edge y=2, so lug at y=1.75? Let's create small rectangular lugs connecting case to strap:
  `new THREE.Mesh(new THREE.BoxGeometry(1.4, 0.8, 0.5), caseMat); pos(0,1.9,-0.1)`
But since case has circular shape, box will intersect case. Fine. It can be visible. But a box with width 1.4 at top might stick out beyond case edge? At y=1.9, x=±0.7, inside radius sqrt(1.9^2+0.7^2)=2.02, okay. Good. Need maybe strap starts at y=2.2, behind. We can add lugs top and bottom.
- Strap top: BoxGeometry(1.5, 2.6, 0.4), pos(0, 3.3, -0.35). But if lugs at y=1.9, strap should run from y=2.1 upward. Center y=3.4, height 2.6 gives from 2.1 to 4.7. Good. Need z=-0.35? If case thickness z=±0.35, strap should attach to back side but extend upward. Set z=-0.25. But if strap z=-0.25, it's behind the plane? The case center z=0. A strap thickness 0.4, centered at z=-0.3 => z=-0.5 to -0.1. It attaches near back. But lugs z maybe -0.1. Good.
Need lower strap: pos(0,-3.3,-0.3), same.
Need maybe rounded strap ends? no.

Need "buckle" maybe small box at bottom strap end? Could add small box at y=-4.3? Not needed.

Need maybe if `watch.rotation.y` auto rotates, straps rotate too. Good.

Need add a "shadow" plane? not necessary.

Need maybe use `THREE.AmbientLight` and `DirectionalLight`. Need if scene background color:
`scene.background = new THREE.Color(0x101418);`
Need perhaps add a grid/floor? Not needed.

Need OrbitControls:
`controls = new THREE.OrbitControls(camera, renderer.domElement);`
`controls.enableDamping = true; controls.dampingFactor = 0.08;`
`controls.target.set(0, 0, 0); controls.update();`
Need if watch auto-rotates, controls still works.

Need maybe if `watch.rotation.y` updated, the hands/group rotate. But if `watch.rotation.y` changes, local coordinates for date overlay unaffected. Good.

Need maybe "automatic rotate" checkbox event:
```
autoRotateCB.addEventListener('change', () => { autoRotate = autoRotateCB.checked; });
```
Need maybe if uncheck, leave rotation.

Need resize:
```
window.addEventListener('resize', () => {
 camera.aspect=window.innerWidth/window.innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(window.innerWidth, window.innerHeight);
});
```

Need potential issue with CDN network. To ensure "正常运行", need maybe use import map? Use static CDN. Need mention Internet required. But final answer should include code. Since "直接完成任务并交付可运行产物" in an answer, not necessarily file, but can provide HTML code. Need maybe say save as `.html` and open. We should deliver code in a code block. Need include "运行方式和测试说明". Good.

Need think about performance. `Intl.DateTimeFormat` each 100ms is fine. Could create formatter once? But timezone fixed? We can create `let tzFormatter = new Intl.DateTimeFormat('zh-CN', {...})` when timezone changes. But current code creates new DateTimeFormat each call at 10Hz, okay but can optimize. We'll set `let formatter = null;` and rebuild on select or if null. But simpler create inside function every 100ms; might be okay but not ideal. Let's implement caching:
```
let tzFormatter = null;
function getTZFormatter() {
 if (!tzFormatter) {
   tzFormatter = new Intl.DateTimeFormat('zh-CN', {...});
 }
 return tzFormatter;
}
```
But if select changes, formatter must update. We can store `tzFormatter` and `formatterTZ`. 
```
let tzFormatter = null;
let tzFormatterName = '';
function getTimeZoneInfo(timeZone) {
 if (tzFormatterName !== timeZone || !tzFormatter) {
   tzFormatter = new Intl.DateTimeFormat(...);
   tzFormatterName = timeZone;
 }
 const parts = tzFormatter.formatToParts(new Date());
 ...
}
```
Need new Date each call. Good.

Need update date and LCD every 100ms. For second hand smooth, update use `Date.now()` every frame. No issue.

Need perhaps "smooth seconds" not just digital. The second hand smooth rotation means if update every frame with milliseconds. Yes.

Need maybe add "tick marks" and "numbers" to dial. Need maybe "center hole" absent.

Potential issue: `ExtrudeGeometry` with shape including tail and long length; when set hand.rotation.z = -angle, the hand shape's y-axis is up. Good.

Need maybe if shape has a sharp point at center? It has tail. Fine.

Need maybe if minute hand overlaps hour hand. Since they are at different z, okay.

Need maybe if second hand gets hidden behind date window? Date window at z=0.371, second hand at z=0.45, so second hand passes in front of date window. Good.

Need maybe "date window overlay" could cover the second hand when second hand over it? Actually overlay is behind second hand, and since overlay transparent and depthWrite false, second hand renders opaque before overlay? Wait transparent objects render after opaque, so overlay will render after second hand. When overlay is at z=0.371 (behind) and second hand at z=0.45 (in front), depth test for overlay fragment at same (x,y) as second hand: depth buffer has second hand depth 0.45; overlay depth 0.371 is closer? Wait depth of second hand is larger? In Three.js, camera looks from +Z to origin. The second hand z=0.45 is closer to camera than overlay z=0.371? Actually camera at (3.5,2.5,5.5) looks at origin; not along +Z. Need depth from camera not simple. The plane may be behind or in front relative to hand depending angle. But likely hand is closer to camera than dial? Since camera is at z=5.5, dial z=0.36, hand z=0.45, hand is closer to camera (larger z) and in front. Good. Depth test should reject overlay over hand if overlay is farther. The overlay transparent rendered after hand, depth test against depth buffer fails where hand is closer, so it doesn't overpaint. Good.

Need if `depthWrite:false` for overlay means it doesn't write depth, but it uses depth buffer values from opaque objects. Hand wrote depth. Good.

Need maybe dial `transparent:true, depthWrite:false` also doesn't write depth, so hands in front transparent? Actually dial is transparent and depthWrite false. It is rendered after opaque? If dial is transparent and doesn't write depth, hands (opaque) are rendered before. Dial at z=0.365 behind hands, depth test fails where hand exists. So okay. But dial might not write depth, so date overlay at z=0.371 doesn't see dial in depth buffer; but overlay itself is transparent after dial, both at similar depth. Date overlay may blend over dial, okay. No issue.

Need perhaps if dial uses MeshBasicMaterial, not affected by light. Good.

Need maybe if scene background dark, transparent corners show dark not case? Wait transparent corners of dial plane: material transparent; if alpha in texture is 0, it will not draw. But because no depthWrite, the case cap behind is opaque and rendered earlier, so visible. Good.

Need maybe if `PlaneGeometry` with transparent texture doesn't cast shadow. no.

Need perhaps "完整3D腕表" with a watch strap can be made more realistic with a curved strap. Could create a loop with TubeGeometry? Let's explore if we can add a curved strap rather than boxes. But boxes enough. Need maybe wristwatch should be a "wrist" not present. no.

Need maybe create a "bracelet" using boxes along a curve? Too much.

Need perhaps use "CylinderGeometry" for case with a smooth bevel. Use LatheGeometry? If use LatheGeometry, can create a cross-section with a curved outline, but more complex. Current cylinder may look plain. We can add bevel by using `CylinderGeometry(2, 1.9, 0.7)` with smooth side. Good.

Need maybe use "THREE.CylinderGeometry" after rotate x, but the top/bottom caps use material front/back. Fine.

Need maybe if case material with specular and side, the cap at front is visible behind dial. Good.

Need perhaps add a "glass" dome: Could use `SphereGeometry(1.9, 32, 16, 0, Math.PI*2, 0, Math.PI*0.5)`? But transparent and positioned at z=0.35? A hemisphere over dial may look nice. But with transparency and no refraction, maybe not. Skip.

Need perhaps add "tachymeter" ring? Not necessary.

Need perhaps use `THREE.MeshPhysicalMaterial` for case? r128 supports MeshPhysicalMaterial but no env map. Could use `MeshPhongMaterial`. Good.

Need test "date and weekday" update: If date window displays month-day and Chinese weekday, okay.

Need maybe if date window at (0.75,0.35) and dynamic overlay is transparent but draws white background, it will cover a small portion of the dial. Need if at three o'clock, hour marker 3 at x=1.85,y=0. No overlap. good.

Need perhaps "second timezone LCD" should display "第二时区" in Chinese label. The UI already. On LCD, maybe line 1: `TKY  10:32:45`, line 2: `05/07 周四`. Good.

Need perhaps include "current date and week of second timezone" if user wants. Great.

Need maybe main analog date? The date window is local date/week. Good.

Need maybe "second timezone" should also be presented with a small analog subdial? Hmm. The prompt says "它应支持同时追踪两个时区" Could be interpreted as both timezones visible on watch simultaneously. Our LCD does that. Good.

Need perhaps "完整3D腕表" means not just face but a wrist watch with band. We have.

Need maybe "直接完成任务" means we should not just explain; code.

Need final answer: Should we include the entire HTML in a code block. Need possibly long. We can within token budget. Need ensure code is correct and no missing brackets.

Let's write code carefully. Need use Three.js r128 from cdnjs. Need script order:
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
```
Need maybe if using `THREE.OrbitControls` with r128, yes.

Let's draft code.

Potential issue with `CylinderGeometry` rotateX: In `createCase`, if we create `new THREE.CylinderGeometry(2, 2, 0.7, 64)` and rotateX, caps are closed. But `MeshPhongMaterial` may render both sides? The front cap might be visible but normals? Cylinder caps normals initially along Y. After rotation, normals along Z. Good.

Need if cylinder top and bottom cap materials same. okay.

Potential issue with ExtrudeGeometry and ShapeGeometry in Three.js r128: `new THREE.ExtrudeGeometry(shape, {depth: thickness, bevelEnabled:false})` works. Need if shape has no holes. Good.

Potential issue with "shape.moveTo(0,-len*0.12); lineTo(...);" The shape might have a self-intersection around the base? Let's define shape for a hand:
```
shape.moveTo(0, -len*0.12);
shape.lineTo(-w/2, 0);
shape.lineTo(-w*0.35, len*0.7);
shape.lineTo(-w/4, len);
shape.lineTo(w/4, len);
shape.lineTo(w*0.35, len*0.7);
shape.lineTo(w/2, 0);
shape.lineTo(0, -len*0.12);
```
This is a closed shape, but from (0,-len*0.12) to (-w/2,0) to ... to (w/2,0) to back start. It is not self-intersecting for w small. Good. The tail is a point at y=-len*0.12. It's like a pointed tail. For second hand length 1.6, tail -0.192, w=0.06, okay.

Need maybe hand shape should have a central circular counterweight? no.

Need maybe if hour hand length 0.95, w=0.18, tail -0.114. Good.

Need center cap should be above second hand, but if second hand has tail, cap covers only center. Good.

Need maybe create `secondHand` with color `0xcc3333` or `0xd33`. Good.

Need maybe minute/hour dark, with lume? no.

Need maybe add "lume dots" on dial? no.

Need maybe add "texture anisotropy" for dynamic textures? not necessary.

Need maybe use `renderer.outputEncoding = THREE.sRGBEncoding`? Not needed for r128? Could, but colors might change. no.

Need maybe `renderer.physicallyCorrectLights`? no.

Need maybe if `watch.rotation.y` auto rotates, the date window and LCD rotate around Y; if we want face visible while rotating, the dynamic planes remain. okay.

Need perhaps add "controls.enablePan = false" to keep target. Fine.
Need perhaps initial camera position: `camera.position.set(3.2, 2.0, 5.2);` Good.

Need perhaps `controls.target.set(0, 0.2, 0);` if watch center at origin.

Need maybe "ground" with soft shadow? We can add a `CircleGeometry` pedestal below watch at y=-4? But watch already includes strap. No.

Need maybe create "table" plane at y=-4.5 to catch shadows? Too much. Maybe we can add a subtle grid background? Scene background dark enough.

Need maybe use `scene.add(new THREE.HemisphereLight(0xffffff, 0x222222, 0.4))` plus directional. Good.

Need lights:
```
const hemi = new THREE.HemisphereLight(0xffffff, 0x444444, 0.5);
scene.add(hemi);
const dir1 = new THREE.DirectionalLight(0xffffff, 0.8);
dir1.position.set(5, 8, 6);
scene.add(dir1);
const dir2 = new THREE.DirectionalLight(0xffffff, 0.3);
dir2.position.set(-4, -2, -3);
scene.add(dir2);
```
Need if `dir1` casts shadow? no.

Need maybe `renderer.shadowMap.enabled = true` unnecessary. Could skip.

Need maybe "autoRotate" checkbox:
Need if controls update with damping, and watch.rotation.y changes, OrbitControls still okay. But rotating an object while camera orbits around it is fine.
Need maybe if autoRotate true, watch.rotation.y += 0.004; If `watch.rotation.y` exceeds 2π, no issue.
Need maybe if `autoRotateCB` default checked:
`<input type="checkbox" id="autoRotate" checked> 自动旋转展示`

Need maybe add "说明" overlay:
`<div id="info">3D 腕表 · 北京时间/本地时间 + 第二时区<br>鼠标拖动旋转 / 滚轮缩放</div>`
But the main analog is local time, not necessarily Beijing. Use "本地时间". Good.

Need include "测试说明" in final after code. Also "运行方式" in final. Need perhaps not in code? The user asked "请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" We can include in final: "将代码保存为 watch.html 后用浏览器打开..." etc. Code block is complete.

Need perhaps mention "需要互联网访问 CDN 加载 Three.js" in run instructions.

Potential issue: If browser blocks CDN due to network? Can't avoid. We can mention.

Need maybe "date and week display should support" maybe if local timezone formats Chinese. Good.

Let's think about if `Intl.DateTimeFormat` with `timeZone: 'Asia/Tokyo'` and `weekday:'short'` in zh-CN returns "周四"; good. `hourCycle:'h23'` property may be unsupported in some browsers? Modern yes. If not, "hour12:false" enough. Need include both? Could use `hour12: false, hourCycle: 'h23'` but if both conflict? `hourCycle` overrides `hour12`? In MDN, if both hour12 and hourCycle specified, hourCycle may be ignored? Actually spec says if both, hourCycle takes precedence? Need not. We can set only `hour12:false` and `hourCycle:'h23'`? Better use `hour12:false, hourCycle:'h23'` to avoid 24. Should be okay. But if h23 in zh-CN returns "24" for midnight? h23 means 00-23, so no 24. Good.
Need maybe `formatToParts` returns `hour` as "24" if hourCycle 'h12'? no.

Need code for `getTimeZoneInfo`:
```
function getTimeZoneInfo(timeZone) {
  const now = new Date();
  if (!tzFormatter || tzFormatterName !== timeZone) {
    tzFormatter = new Intl.DateTimeFormat('zh-CN', {
      timeZone,
      hour12: false,
      hourCycle: 'h23',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
      month: '2-digit',
      day: '2-digit',
      weekday: 'short'
    });
    tzFormatterName = timeZone;
  }
  const parts = tzFormatter.formatToParts(now);
  const info = {};
  for (const part of parts) info[part.type] = part.value;
  return info;
}
```
Need if timezone changes, rebuild. Good.

Need maybe second timezone date string:
```
const tz = getTimeZoneInfo(currentTZ);
const tzTime = `${tz.hour}:${tz.minute}:${tz.second}`;
const tzDate = `${tz.month}-${tz.day}`;
const tzWeek = tz.weekday || '';
```
Need `tz.day` might be without leading zero? Since we specified month/day 2-digit, yes.

Need maybe if `weekday` in zh-CN returns "周四" or "星期四"? likely "周四". Good.

Need maybe local date window:
```
const now = new Date();
const localDate = `${pad(now.getMonth()+1)}-${pad(now.getDate())}`;
const localWeek = weekdaysCN[now.getDay()];
```
Need if date window overlay size 1.1x0.55, canvas 256x128. Font 34/28 okay. Maybe "05-07" and "周四" fit. Good.

Need maybe use monospace font for English. Chinese font "Microsoft YaHei" in browser. On non-Chinese systems, fallback.

Need if `MeshBasicMaterial` for LCD with `transparent:true` and depthWrite false: If we draw background black opaque, transparent true with alpha 1. Good.

Need maybe if dynamic canvas cleared and re-drawn with `clearRect`, then draw. good.

Need maybe if lcd texture color black and green text, `depthWrite:false` means hands behind? no.

Need maybe if date window overlay at z=0.371 has no depthWrite, and lcd at z=0.372. They don't overlap. okay.

Need perhaps "camera near/far" okay.

Need maybe if the watch is auto rotating around Y, and the camera is z=5.5, the watch face normal +Z points toward camera at initial yaw=0. As yaw rotates, face disappears. That's fine. If default checked, user can uncheck.

Need maybe if `autoRotate` true, after some time the watch back faces camera; if user wants inspect front, can uncheck and adjust. Maybe default false might be better for "watch animation" showing face. But "full 3D" suggests rotate. Let's default checked true? Hmm. If user opens, the watch rotating may be cool but they cannot read time easily. Could set default false but add "auto rotate" button. The prompt says "3D 腕表动画" not "静态". We can set autoRotate true but slowly enough. Maybe maybe users can still see face half the time. I'd set false? Let's decide. The user asked "3D腕表动画" and "正常运行的3D腕表" "完整3D腕表形式". Auto-rotate showcases full 3D. I'd default checked true. But if testing time, they can uncheck. Good.

Need maybe update direction: `watch.rotation.y += 0.005`; at 60fps, full rotation in 20.9 seconds. Good.

Need maybe if controls target set, and autoRotation of watch, but the dial plane normal after rotation maybe not updated? It is part of group. good.

Need maybe "current time" if system timezone is not the local timezone? The main analog uses system time. If user in Beijing, okay. If user in other timezone, local. Date/week local. Good.

Need maybe "two timezones" if second timezone equals local? We can choose another. good.

Need maybe if `Intl.DateTimeFormat` with `timeZone` not available for certain? all major.

Need maybe if `OrbitControls` CDN path is correct? For r128, examples/js/controls/OrbitControls.js is there. Use `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`. It defines `THREE.OrbitControls`. Good.

Need maybe if `three.min.js` cdnjs has r128 global. yes.

Need perhaps page title and meta. Good.

Let's think if code will produce a visible date window when page opens. `updateDisplays` called first in animate after 100ms; but before first draw texture is blank, lcd blank. We can call `updateDisplays()` after creating meshes to fill. Need do.
```
updateDisplays();
```
after UI setup? But `currentTZ` global initialized. yes.

Need maybe if `getTimeZoneInfo` returns `hour` "24" if no h23? no.

Need maybe if `new Date(nowMs)` in `updateHands` each frame creates objects, but okay. Could optimize by passing current date? no.

Need maybe if `animate` gets `performance.now` and `Date.now`; no issue.

Need perhaps "smooth second hand" with milliseconds in an analog watch: a real mechanical watch has sweeping seconds, yes.

Need maybe "smooth turning second/minute/hour" all hands include fractional values:
```
const date = new Date(nowMs);
const s = date.getSeconds() + date.getMilliseconds()/1000;
const m = date.getMinutes() + s/60;
const h = date.getHours() % 12 + m/60;
```
Good.

Need maybe if second hand uses shape and material with side DoubleSide, if shape is on XY plane, extrude along z, so the side walls visible. Good.
Need maybe if `geo.translate(0,0,-thickness/2)` then position.z = z. Good.

Need maybe if shape coordinate y is up, but ExtrudeGeometry shape is in XY plane. When we rotate around z, it rotates. Good.

Need maybe if date window and lcd overlay are `MeshBasicMaterial` and the dial is also `MeshBasicMaterial`, they may be rendered in transparent pass sorted by z. Since date overlay is slightly in front of dial, it should render after dial. It doesn't write depth, but it will blend over dial. Good. Need if the lcd plane is bigger than date, no overlap.

Need maybe if dynamic textures are 256/512 but `PlaneGeometry` in world units, not square? Date mesh aspect 1.1/0.55 = 2, canvas 256/128 = 2. Good. LCD 2.1/1.05=2, canvas 512/256=2. Good.

Need maybe if date window text is stretched? no.

Need perhaps add an inner bezel ring to dial: static texture includes outer stroke. Good.

Need maybe if dial texture plane is square, transparent corners, but `depthWrite:false`; because it is transparent, the case cap behind may show. But if we want the case cap behind to have a circular shadow? no.

Need potential issue: If the dial plane is transparent, it might not occlude the strap behind from some angles? Since transparent object has depthWrite false, it doesn't write depth, but the strap behind is opaque and already in depth buffer. When viewing from the side, dial plane is at z=0.365, strap at z=-0.3 behind. The dial transparent corners should not occlude; no issue. But dial itself is transparent only outside circle. It doesn't write depth, so hands and later transparent objects can render behind? There are no opaque objects behind. Good.

Need maybe if the case cap is opaque but rendered before dial; since dial transparent doesn't write depth, but case cap has depth. okay.

Need perhaps if `watch.rotation.y` changes, the `CircleGeometry` not used, so no UV issue.

Need maybe add `texture.encoding = THREE.sRGBEncoding`? no.

Need maybe create watch group and add to scene; sets initial rotation? We can set `clockGroup.rotation.x = -0.1`? no.

Need maybe if autoRotate rotates around Y, the watch's +Z normal not toward camera at start? camera at z positive, so yes.

Need perhaps "local time display" in LCD label? The LCD line 2 date maybe second timezone date; line 3 "SECOND TIMEZONE". Good. Could also show `HOME`? The date window shows local date; no label. But main analog is local time. Fine.

Need maybe "date and week display" for local date in date window; also LCD has second timezone date. Good.

Need maybe show "TKY 10:32:45" and "05/07 周四" on LCD. The LCD text line 2 is second timezone date/week, not local. But prompt just says can provide date/week; okay.

Need maybe if the user wants date and week display "功能" not necessarily analog; the LCD and date window provide. Good.

Need perhaps include "timezone select" in controls, with option values. Need include `data-label` maybe. Use `<option value="Asia/Tokyo" data-label="东京">东京 (Asia/Tokyo)</option>`. On get label:
```
const sel = document.getElementById('tzSelect');
const label = sel.options[sel.selectedIndex].dataset.label || sel.value;
```
Need `dataset` works.

Need maybe if user changes timezone formatter cached; in `updateDisplays`, `getTimeZoneInfo(currentTZ)` checks currentTZ. good.
Need maybe if `tzSelect` value used. yes.

Need maybe if `currentTZ` is an IANA name, `Intl` handles. good.

Need maybe "测试说明": 
- 保存为 .html，用现代浏览器打开。
- 默认主表盘走本地时间；秒针连续平滑旋转。
- 在页面底部选择第二时区，LCD 屏幕会显示对应时区的时间/日期/星期。
- 日期窗显示本地日期和星期；可切换不同时区验证。
- 使用鼠标拖拽旋转视角，滚轮缩放；取消“自动旋转展示”可固定角度观察。
- 若需验证 12 点/日期切换，可临时修改系统时间（或修改代码中 `new Date(nowMs)` 参数）。
Need maybe mention requires Internet.
Need maybe "直接完成任务" maybe no extra explanation. But final answer should include code and instructions. We can keep final concise.

Let's ensure code is syntactically sound. Let's write code step by step.

Full code:

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<title>3D 腕表</title>
<style>
* { box-sizing:border-box; }
body { margin:0; overflow:hidden; background:#111; color:#fff; font-family: Arial, sans-serif; }
#info { position:absolute; top:10px; left:10px; background:rgba(0,0,0,0.65); padding:10px 14px; border-radius:10px; pointer-events:none; font-size:13px; line-height:1.5; }
#controls { position:absolute; bottom:16px; left:50%; transform:translateX(-50%); background:rgba(0,0,0,0.7); padding:10px 16px; border-radius:10px; display:flex; align-items:center; gap:8px; flex-wrap:wrap; justify-content:center; font-size:13px; z-index:10; }
select { background:#222; color:#fff; border:1px solid #555; border-radius:6px; padding:4px 8px; outline:none; }
label { display:flex; align-items:center; gap:4px; cursor:pointer; }
</style>
</head>
<body>
<div id="info">⌚ 3D 腕表<br>本地时间 + 第二时区<br><small>拖动旋转 / 滚轮缩放</small></div>
<div id="controls">
  <label>第二时区
    <select id="tzSelect">
      <option value="Asia/Tokyo" data-label="东京">东京 (Asia/Tokyo)</option>
      <option value="Asia/Shanghai" data-label="北京">北京 (Asia/Shanghai)</option>
      <option value="Asia/Singapore" data-label="新加坡">新加坡 (Asia/Singapore)</option>
      <option value="Asia/Dubai" data-label="迪拜">迪拜 (Asia/Dubai)</option>
      <option value="Europe/London" data-label="伦敦">伦敦 (Europe/London)</option>
      <option value="Europe/Paris" data-label="巴黎">巴黎 (Europe/Paris)</option>
      <option value="America/New_York" data-label="纽约">纽约 (America/New_York)</option>
      <option value="America/Los_Angeles" data-label="洛杉矶">洛杉矶 (America/Los_Angeles)</option>
      <option value="Australia/Sydney" data-label="悉尼">悉尼 (Australia/Sydney)</option>
    </select>
  </label>
  <label><input type="checkbox" id="autoRotate" checked> 自动旋转展示</label>
</div>
<script src="..."></script>
<script src="..."></script>
<script>
(function() {
 // ... all code
})();
</script>
</body>
</html>
```

Need inside IIFE, variables.

Let's craft code.

Potential issue: `texture.anisotropy = renderer.capabilities.getMaxAnisotropy();` in `createDialTexture` after renderer exists. Yes createWatch called after renderer. Good.

Need if `renderer` not yet defined in `createDialTexture`? It is. Use `renderer.capabilities.getMaxAnisotropy()`. Good.

Let's implement.

Need maybe if `new THREE.PlaneGeometry(3.7, 3.7)` has vertical y up. The canvas texture orientation: We'll draw 12 at top by y = c - R. Since default flipY, it should display correct? Need verify with PlaneGeometry. Let's think: PlaneGeometry's vertices in XY plane: positions are (-0.5,-0.5,0) to (0.5,0.5,0) with UVs (0,0) at -y. texture flipY true: The top of canvas (y=0) becomes V=1? In WebGL, with flipY true when uploading, source image is flipped, so top row becomes v=0? Hmm. There is confusion. Let's look at standard Three.js CanvasTexture example:
```
const texture = new THREE.CanvasTexture( canvas );
const material = new THREE.MeshBasicMaterial( { map: texture } );
const plane = new THREE.Mesh( new THREE.PlaneGeometry( 1, 1 ), material );
```
Text appears not upside down. So canvas y down maps to plane y up. That means 12 drawn at canvas y = c - R (top) appears at plane y positive (top). Good. We'll trust.

Need if date overlay plane at world y=0.35, texture draws text at canvas top? PlaneGeometry UV maps plane top to canvas top? With default flipY, yes. So if we draw date text centered, good.

Need maybe if We use `CircleGeometry` for dial? no.

Need maybe if the hands' shape has y up, and rotation.z negative. Good.

Need maybe if `watch.rotation.y` auto rotates, and `clockGroup.rotation.y` changes, `controls.target` still origin. Good.

Need maybe if `autoRotate` checkbox change:
```
autoRotateCB.addEventListener('change', function(){ autoRotate = this.checked; });
```
Need `this` in arrow? Use function.

Need maybe if `tzSelect` change:
```
tzSelect.addEventListener('change', function(){ currentTZ = this.value; tzFormatter = null; });
```
Need formatter variable global.

Need maybe if `updateDisplays` uses `currentTZ`, `tzLabel` updated each time.

Let's write code in final. Need ensure no syntax errors.

Let's think about `shape` and ExtrudeGeometry with width/height. For second hand, width=0.035 maybe too thin; ExtrudeGeometry with depth 0.02, visible. For color red. Need maybe if width too small, shape triangulation okay. Use `width: 0.05` for second hand.

Need maybe if hour hand shape and minute hand shape are planar, but ExtrudeGeometry's front/back faces along +Z. When viewed from front, okay. But if `side:THREE.DoubleSide`, no issue.

Need maybe if the material for hands is `MeshPhongMaterial`; with no envMap it will be shaded. Good.

Need maybe if the date window overlay is a `MeshBasicMaterial` with `transparent:true` and `depthWrite:false`, but if using `depthWrite:false`, it will be sorted with other transparent objects by distance. good.

Need maybe if the `MeshBasicMaterial` for dial is transparent and `depthWrite:false`, but not `side:THREE.DoubleSide`. PlaneGeometry from front only. If camera moves behind watch, dial invisible. We set `side: THREE.DoubleSide` so visible from behind (with texture mirrored). But watch back and case visible; okay. Need include `side: THREE.DoubleSide` in dial material? If transparent, DoubleSide may render back side; fine. Set.

Need perhaps if dynamic LCD and date overlay should be visible from front only. If camera behind, no need. Set side:THREE.DoubleSide? Could use default FrontSide; if behind, invisible. But if watch rotates, these planes can be seen from back? In their local frame, they face +Z; when watch rotates 180°, their normal faces away from camera, so default FrontSide invisible. That's okay because from back you shouldn't see dial. But if autoRotate, sometimes you see back with no dial. Good. But if user orbits to back, not need. We can keep default. But for the dial, DoubleSide might let you see through when behind? If side DoubleSide and alpha transparent, you would see the reverse of dial (mirrored) through the case? Actually from behind, the dial plane is behind case back? The case back is opaque and shields it. So no issue. For date overlay and LCD, if from behind, they are behind the case back but before? Wait order from back: case back at z=-0.35, dial at z=0.365. The case back is opaque and in front of them when camera behind; it occludes. So no issue. Good.

Need maybe if case material is `MeshPhongMaterial` with color grey and `side` default, front/back caps visible from both sides? MeshPhong material default front side only; but on CylinderGeometry, the outer faces have normals outward, so from outside of watch, visible. The top cap normal +Z, visible from front; bottom cap normal -Z, visible from back. Good. The side wall normal outward, visible from side. So okay.

Need perhaps if `CylinderGeometry` top radius 2, bottom radius 1.9; after rotate, top cap at z=+0.35, bottom at z=-0.35. The dial at z=0.365 is in front of top cap. Good.

Need maybe if bezel torus at z=0.25? Actually if front cap at z=0.35, torus tube radius 0.1 and position z=0.25 means tube z from 0.15 to 0.35. This means it sits behind front cap, no protrusion. If position z=0.35, tube from 0.25 to 0.45, protrudes in front. But dial at z=0.365 would be inside torus front? The torus inner edge radius 1.89; the dial plane radius 1.85, so it sits inside. The front protrusion (z 0.35-0.45) above dial plane around edge, like a bezel. That's good. But if dial at z=0.365, the torus front part (z>0.365) is outside radial ring, no conflict. So set bezel position z=0.3? The tube from 0.2 to 0.4. Dial at 0.365, okay. Let's set bezel `position.z = 0.30`. Then front edge 0.40, back 0.20. Good.

Need if dial plane is at z=0.365, the bezel central tube at radius 1.99, tube radius 0.1, inner radius 1.89. The dial radius 1.85, so bezel covers dial outer edge 0.04. Good.

Need perhaps if case front cap at z=0.35 and dial at z=0.365, the dial is slightly above cap. Good.

Need perhaps if date overlay at z=0.371 is in front of dial but behind hands. Good.

Need perhaps if lcd overlay at z=0.372. fine.

Need maybe if hands at z=0.39 are not too close to dial. Good.

Need maybe if second hand at z=0.46 and center cap at 0.47, okay.

Need maybe if strap z position and watch rotation: If watch face normal +Z, the strap extends along Y; when watch rotates around Y, the strap will also rotate showing side. Good.

Need maybe if the top and bottom straps are "behind" the case. Need set z=-0.3 maybe. But if the strap is at z=-0.3, a side profile shows strap centered behind case, not in center. Real watch strap attaches at top/bottom lugs on the back side but not necessarily centered. Let's set z=-0.25. The case thickness from -0.35 to 0.35; strap thickness 0.4 centered at -0.25 => from -0.45 to -0.05. It protrudes behind but not front. Good.
Lugs maybe at z=-0.15, from -0.4 to 0.1. okay.

Need maybe if the strap should be between lugs: top lug at y=1.8 and z=-0.15, upper strap at y=3.3 and z=-0.25. There is a gap. Fine.
Could add second "keeper" small boxes near strap ends? no.

Need perhaps if case body has no lugs, the strap may intersect case if centered at y=3.3? No.

Need maybe if we want a "wristwatch" not "pocket watch", top strap at y up and bottom strap at y down.

Need maybe if the watch face has no "crown" in 3D? yes.

Need perhaps if the watch case should have "pushers" (chronograph) not needed.

Let's think about if the final answer should include code only or also a demo? We cannot run. Code block enough.

Need maybe if "测试说明" includes "打开页面后若没有画面，请检查网络或等待 CDN 加载". Good.

Potential issue: In a page served from file://, CDN scripts load okay. If offline, not. Fine.

Need maybe if `OrbitControls` uses `THREE.OrbitControls`; with `three.min.js` r128, yes.

Need perhaps if `THREE.ExtrudeGeometry` uses `depth` property, but in older examples use `amount`. In r128, `depth` is correct? Let's verify. In Three.js r128, `ExtrudeGeometry` options: `depth` (formerly `amount`) is used? I think `amount` was renamed to `depth` in r125? Actually three.js docs: `ExtrudeGeometry` uses `amount`? Let's recall. In r128, examples often:
```
new THREE.ExtrudeGeometry( shape, { depth: 2, bevelEnabled: false } );
```
I think `depth` is correct. To be safe, use `amount`? r128 docs: "depth — depth to extrude the shape. Default 100." I think `depth`. Good. If not, `amount` also accepted? Not sure. Use `depth`.
Need maybe if `bevelEnabled: false` default false. okay.

Need perhaps if `shape.moveTo(0, -len*0.12); ...` and last lineTo to start, automatic close. Need if no `closePath`, Shape auto close. Good.

Need maybe if `new THREE.MeshPhongMaterial({color: opts.color, specular:0x222222, shininess:70, side:THREE.DoubleSide})`; okay.

Need maybe if `CylinderGeometry` for cap after rotateX creates a cylinder along z. But the cap's "height" is 0.1 along z; the cap's diameter r. To be at z=0.47, place. It will be centered on hand pivot. Good.

Need maybe if center cap uses same material as case? color dark. Good.

Need maybe if the second hand extends below center, the center cap covers tail. okay.

Need maybe if the second hand is red and thin; maybe add a central red circle? no.

Need maybe if `watch.add` order doesn't matter.

Need maybe if `createWatch` returns `clockGroup`, but add to scene. good.

Need maybe if "display second timezone" lcd updated at 10Hz, might not show seconds smoothly. But it's digital, okay. Main analog hands smooth.

Need perhaps if the date window's date should also update at midnight even if page open. updateDisplays every 100ms so yes.

Need perhaps if currentTZ default Tokyo, but if user local timezone is Tokyo, second timezone equals local. okay. Default could be "America/New_York" to demonstrate different. But option default first is Tokyo. fine.

Need maybe if system timezone is e.g. China, and second timezone Tokyo, LCD differs by 1 hour. Good.

Need perhaps if "local date and weekday" in date window should be in Chinese. We use `weekdaysCN`. good.

Need maybe if date window has "05-07 周四"; but if date number is single digit? pad. good.

Need maybe if `new Date(nowMs)` in updateHands uses milliseconds, but if system clock changes, okay.

Need perhaps if "smooth" second hand with `Date.now()` might have small jitter due to frame rate, but okay.

Need maybe if autoRotate with `watch.rotation.y += 0.004` updates every frame, but when page tab inactive, requestAnimationFrame pauses, so hands not updated. When tab active again, Date.now jumps to current time, so correct. Good.

Need maybe if `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` to avoid performance. Good.
Need maybe if `renderer.setSize(...)` uses window.innerWidth/Height.

Need perhaps if "full 3D wristwatch" should maybe be affected by gravity? no.

Let's consider if the `createDialTexture` function uses `renderer.capabilities.getMaxAnisotropy()` but renderer is created before watch. yes.
Need maybe if `createDialTexture` called inside `createWatch`, which is called after `renderer`. yes.

Let's write code fully with all variables.

Potential issue: `new THREE.CanvasTexture(canvas)` already sets `flipY = true` by default. But if we set `texture.anisotropy`, fine.
Need maybe if `dateTex.needsUpdate = true` after redraw, but because CanvasTexture initialized before first draw? We call `updateDisplays` after textures created, so set needsUpdate. Good.

Need perhaps if LCD canvas initial transparent black? We draw background. good.

Need maybe if `dateCtx` and `lcdCtx` captured before canvas appended? We create canvases offscreen. okay.

Need maybe if `getContext('2d')` returns non-null. yes.

Let's write final code in a single block. Need be careful with Chinese strings and no code comments? Comments okay.

Need maybe if final answer says "请将下列代码保存为 watch3d.html". Then code. Then "运行方式" and "测试说明". Good.

Potential issue with `Intl.DateTimeFormat` formatToParts with `weekday:'short'` for `zh-CN` might produce "星期三" or "周三"; okay. If it produces "Wed" in some locale? But `zh-CN` yields Chinese. Good.

Need maybe if `Intl.DateTimeFormat` with `hourCycle:'h23'` and `hour12:false` in some browsers might throw? I think no. To be safe, only `hour12:false` suffices. Let's use `hour12: false` and no `hourCycle`. But "24:00:00" issue for midnight? With `hour12: false`, some browsers at midnight may output "24:00:00" (hourCycle h24) for en-US? In zh-CN, likely "00:00:00"? To avoid, use `hourCycle:'h23'`. According to MDN, `hour12` and `hourCycle` are mutually exclusive? Actually can specify both; if both, `hour12` true/false takes precedence? Not sure. Let's just set `hourCycle: 'h23'`. It implies 24-hour, not necessarily hour12. Without hour12. Good.
Need maybe if `Intl.DateTimeFormat` options include `hourCycle: 'h23'` and `hour: '2-digit'`, okay.
Let's use:
```
hourCycle: 'h23',
hour: '2-digit',
```
No `hour12`. Good.

Need maybe if `formatToParts` returns `literal` parts; we ignore. good.

Need maybe if the timezone list includes `Asia/Shanghai`; label "北京". Good.

Need perhaps if `currentTZ` should maybe be "Asia/Shanghai" because we mention "北京时间"? In final, main analog local, not necessarily Beijing. The prompt Chinese maybe "Beijing time" but not required. Let's default "Asia/Tokyo" because it's different for many. But "北京" more likely? The UI label can be "本地 + 第二时区". The select default maybe "Asia/Tokyo"; fine.
Need maybe if `tzSelect` label "第二时区" might be confusing: The main analog is local; second timezone selected. Good.

Need perhaps if `autoRotate` default true, the watch rotates but the date/LCD textures are updated with local time; no issue.

Need maybe if `controls.update()` called every frame. Good.

Potential issue: If `autoRotate` true, `watch.rotation.y` changes but the date/LCD plane's normal rotates, and `MeshBasicMaterial` is not affected by lights but still visible from front. If backside, maybe not. good.

Need perhaps if `watch.rotation.y` auto rotates, and the UI controls are at bottom, no issue.

Need maybe if "straps" are boxes and case is a cylinder; when auto rotates, box edges might be obvious. okay.

Need maybe if `caseMesh.rotation.x = Math.PI/2` and `bezel` no rotation. The case top cap at z=0.35; the case side is a cylinder along z. The bezel torus lies in XY plane around z axis. Perfect.

Need perhaps if `crown.rotation.z = Math.PI/2` and `crown.position.x = 2.05`; the crown axis along x. Need if case has radius 2, crown extends from x=1.8 to 2.3 if length 0.5 centered at 2.05. It intersects case side at x=2; okay. Need if crown extends in -x as well, into case. okay.
Need maybe if crown's top/bottom caps oriented along x; default cylinder caps have normals along y; after rotation z, normals along x. Good.

Need maybe if lugs and strap connect to case; no issue.

Need maybe if "watch group" is centered at origin. Good.

Let's think if the dial texture numbers and date overlay orientation:
- In createDialTexture, we draw hour numbers at R=300. The center of dial is at 512,512. The hands extend up to length 1.6 world, which is 0.865? Wait world to canvas: plane size 3.7 world maps to 1024 px. So 1 world = 276.76 px. Radius of dial = 1.85 world = 512 px. Hand length 1.6 world = 443 px. So second hand reaches 443px from center, within dial radius 512px. Good.
- Hour markers at R=400/440 px = 1.45/1.59 world. Good.
- Hour numbers at R=300 px = 1.08 world. Hands can cover them; okay.

Need maybe if the date window at world x=0.75 maps to 207 px right, y=0.35 maps to 97 px up (canvas y=c-97=415). The date window overlay plane is 1.1x0.55 world = 304x152 px. It is centered at (719,415), covering x 567-871, y 339-491. This covers some hour numbers? Hour number 3 at x=812, y=512? Wait hour 3 at angle 0: x=512+300=812, y=512. Our window y 339-491, so just above 3, not cover. Number 4 at angle 30? x=512+260=772, y=512+150=662, no. Number 2 at angle 30? x=772, y=362, covered partly. Hmm date window would cover hour 2? Let's compute: 2 o'clock angle 60°? Actually hour 2 at 60° from 12 clockwise: cos(60)=0.5, sin(60)=0.866 (canvas y down? For 2 o'clock, x=512+150=662, y=512+260=772? Wait 12 at top, 1 at 30° right/down? Need coordinate: 3 o'clock angle 0? Let's use angle = (hour-3)*30? For 2 o'clock angle -30? Hmm. But our draw code:
```
const ang = i*Math.PI/6 - Math.PI/2;
```
For i=1: 30° - 90° = -60°, x=c+300*cos(-60)=662, y=c+300*sin(-60)=252? That places 1 at upper right. For i=2: 60-90=-30°, x=772, y=362. That is upper right, y=362. Date window y 339-491, x 567-871, so it covers 2 o'clock number. That's bad. At 3: ang=0, x=812,y=512. Window y 339-491 so doesn't cover 3. At 2, covered. So date window at x=0.75,y=0.35 might cover 2 and maybe 1? Let's move date window closer to 3 o'clock, e.g., x=1.05, y=0.15. But PlaneGeometry would be partly outside dial? Center x=1.05 world, width 1.1, so right edge 1.6, still inside radius 1.85. good. y=0.15, height 0.55 => y -0.125 to 0.425, covering 3? The 3 number at x=1.08,y=0? Actually hour number 3 at world x=1.08, y=0. The window at x=1.05, y=0.15, height 0.55, covers y -0.125 to 0.425. It might cover 3. Hmm maybe okay? A date window at 3 o'clock usually sits just inside 3, not over it. Actually 3 marker is at edge, so window at x=0.8,y=0.0 and size 0.8x0.4 would cover marker. Traditional date window is at 3 o'clock, replacing the 3 marker sometimes. So okay. But we don't want cover hour 2/4? Let's choose date window at x=0.85, y=0.0, size 1.0x0.5. This covers around 3 o'clock marker. That's actually acceptable; many watches have date at 3 and delete 3 marker. But we draw 3 marker at x=1.08, y=0; the window would cover it. Good. It won't cover 2/4 too much? x 0.35 to 1.35, y -0.25 to 0.25. 2 at x=0.72,y=0.94? Wait 2 number is at x=0.72, y=0.94? Let's convert: 2 at canvas (772,362) -> world x=(772-512)/276.76=0.94, y=(512-362)/276.76=0.54? Hmm not y=0.94. Need recalc. Actually Plane world y positive up; canvas y down. For hour 2, canvas x=772, y=362 -> world x=0.94, y=0.54. So date window y -0.25 to 0.25, no cover 2. Good. At 1: canvas 662,252 -> world x=0.54, y=0.94, no. At 4: canvas 772,662? Wait draw formula with ang -30 gives y=512+300*sin(-30)=362? That was 2? Let's not overcomplicate. For hour 3, canvas x=812, y=512 -> world x=1.08, y=0. Date window centered at x=1.0, y=0.0 would cover 3. That is good.

But original date window at (0.75,0.35) maybe covers 2? We can move to (1.0,0.0) to avoid 2/4. Need adjust static texture? We can also not draw hour number 3 if date window covers. But createDialTexture static can't know. It's okay if window covers 3; watch date window at 3 often replaces 3. Let's set date mesh position `(0.95, 0, 0.371)` and size `(1.1, 0.55)`. This covers around 3 o'clock, x 0.4-1.5, y -0.275-0.275. It touches marker at x=1.85 no. It covers no 2/4. Good.
Need perhaps if date window overlaps minute track? At x=1.4, y=0.2, radial distance 1.42 < 1.85, so inside markers. okay.
Need maybe if date window overlays the "3" number at x=1.08; yes. Good.
Need maybe if on static dial we draw 3; covered by opaque white date window. Fine.

Need perhaps if the second timezone LCD at (0,-0.95) size 2.1x1.05 covers 6 o'clock marker? 6 marker at (0,-1.85). LCD bottom -1.475, so no. Covers center/bottom subdial area, no numbers. Good.
Need perhaps if LCD top -0.425, bottom -1.475; it might cover 7 and 5 markers? 5 marker at x=0.925? Let's calculate 5 o'clock (clockwise from 12 by 150°? Actually 5 at angle 150° from 12? Wait 12 at -90 in code; hour i=5: ang=5*30-90=60°, cos=0.5, sin=0.866 -> world x=0.5*1.08? Wait R=300 px -> x=0.54 world, y=-0.94? Since canvas y=512+260=772 -> world y=(512-772)/276.76=-0.94. So 5 marker is at x=0.54,y=-0.94, and LCD top -0.425, so not cover. 6 at x=0,y=-1.08? Wait hour 6: ang=180-90=90°, x=0, y=512+300=812 -> world y=-1.08. Ah I earlier thought marker at 1.85 but numbers at R=300; minute markers at R=440 px = 1.59. The 6 number at -1.08; LCD bottom -1.475, not cover. Good. LCD large but not covering numbers.
Need perhaps if the hand extends over the LCD, okay.

Need maybe if date window covers the 3 number; maybe we should remove 3 from static texture to avoid white window over number? It's okay.

Need maybe if minute track at R=440 px = 1.59 world, date window at x=1.0,y=0.2, radial sqrt(1^2+0.2^2)=1.02, so no.

Need maybe if LCD at y=-0.95, x=0, overlaps minute track? no.

Need perhaps if the date overlay uses `PlaneGeometry(1.1,0.55)` and transparent true, it will render over dial; if it covers 3 marker, okay.

Need perhaps if `MeshBasicMaterial` for date is transparent and `depthWrite:false`, but if it covers 3 and is white, okay. The static dial's 3 number might still be visible if alpha of date window opaque. It is opaque in canvas background, so covers.

Need perhaps if date window's border line and text are on transparent background? We draw `fillRect` opaque white, so no need transparency, but `transparent:true` allows alpha in canvas corners? Actually date canvas is entire 256x128; we fill rect from 4,4 to 248,120, leaving 4px transparent. Material transparent true, so outside white border transparent. Good.

Need perhaps if LCD canvas entire 512x256; draw border rect, leaving 4px transparent. good.

Need perhaps if layer order: date and LCD are transparent. They are sorted by z. Date at z=0.371, LCD at z=0.372, both in front of dial but behind hands. They may render in arbitrary order, but no overlap, so no issue.

Need maybe if the dial plane itself transparent and `depthWrite:false`; the dial does not write depth. Thus the date overlay at z=0.371 has no depth from dial, but it is okay. The case behind writes depth. The dial transparent corners are rendered after opaque, but before/after overlay? Transparent objects are sorted by distance. Date overlay z slightly > dial, so closer to camera? If camera along +Z, date overlay rendered after dial; if not, maybe. They overlap? Date overlay is on top of dial; if rendered before dial, then dial could blend over date. Since transparent objects sorted by depth from camera; date overlay (z=0.371) is closer than dial (z=0.365), so date overlay rendered after dial. Good. If camera is at some angle, depth order still by distance; likely similar. okay.

Need perhaps if hands are opaque and rendered before all transparent; when transparent date overlay over hand? Depth test prevents. good.

Need maybe if the dial is transparent and `depthWrite:false`, and the hands are opaque and rendered before dial, but the dial transparent after hands, depth test prevents dial over hand where hand is in front. But if a hand is behind dial? No, hands are in front. Good.

Need perhaps if the LCD transparent overlay is behind hands and depth test prevents. okay.

Need perhaps if the `case` material with default side FrontSide and the case cap at front is behind dial; if the dial is transparent and doesn't write depth, hands still write depth. good.

Need maybe if the auto rotate and orbit controls produce weird z-fighting between date overlay/dial? z difference 0.006, at radius small, okay. Could increase to 0.01 to avoid. Let's set dial z=0.365, date/lcd z=0.375. Hands hour at z=0.40. Need if date overlay at 0.375, second hand at 0.46. Good. Let's adjust:
- dial z=0.365
- date/lcd z=0.375
- hour z=0.40 (occupies 0.38-0.42)
- minute z=0.43 (0.41-0.45)
- second z=0.46 (0.44-0.48)
- cap z=0.47

Need if bezel torus front at z=0.40? bezel position z=0.30, tube front 0.40. That matches hour hand plane at z=0.40 at radial edge. But hands inside dial, no issue.

Need perhaps if the hour hand at z=0.40 and date overlay at 0.375, difference 0.025. good.

Need perhaps if strap top extends too high and camera? okay.

Need maybe if "watch case" cylinder with top cap at z=0.35, dial at 0.365, but if dial plane is square transparent, it extends beyond case radius at corners; transparent corners show case cap? Actually the dial plane square extends from -1.85 to 1.85 in x/y; within the case? The case radius is 2, so square corners at (±1.85,±1.85) have radius 2.616, outside the case. The transparent corners will show background, not case, because outside case radius. That's bad: from front, the dial plane square extends beyond the circular case; its transparent corners are invisible, but they also might not occlude. That's okay: outside case radius, background shows; the case still is circular. The dial plane is larger than case but transparent outside circle radius 1.85 (texture). Actually we draw circle radius 500 of 1024, which maps to 1.85 world. The plane is 3.7x3.7, so the circle exactly inscribed in the plane. Outside the circle, alpha 0. So invisible. Good. No issue.
Need maybe if the plane edge extends outside the case and can be seen edge-on from side? The plane has no thickness, so no. Good.

Need perhaps if the case radius is 2, dial radius 1.85, so case bezel ring visible from 1.85 to 2.0. good.

Need perhaps if in texture, circle radius 500 px maps to plane half-size 1.85 exactly. yes.

Need perhaps if `PlaneGeometry(3.7,3.7)` and canvas circle radius 500, the circle is 1000 px diameter, leaving 12 px transparent at each edge (since canvas 1024). Actually radius 500, plane half=1.85, so circle touches edge at 500? Plane edge corresponds 512. So circle doesn't touch edge: there is 12 px transparent margin. Good.

Need perhaps if the date overlay at z=0.375 is on top of dial, but since it's a separate mesh, if the dial plane rotates with watch, fine.

Need maybe if "digital LCD" line 3 "SECOND TIMEZONE" is in English. good.

Need perhaps if "date window" text is drawn with Chinese, but if font lacks Chinese, fallback. okay.

Need perhaps if "currentTZ" label from select when computing display; if no selection, first option. good.

Need maybe if `tzFormatter` not initialized; getTimeZoneInfo checks. good.

Need maybe if `tzLabel` variable defined in updateDisplays, not global. good.

Let's write final code carefully.

One more thought: We use `renderer.capabilities.getMaxAnisotropy()` in `createDialTexture` which is called inside `createWatch` after `renderer` exists. But "renderer" variable is defined in outer scope. Good.

Need maybe if "controls" variable declared but not used outside. In animate, `controls.update()`. good.

Need maybe if "scene" variable. good.

Need maybe if "clockGroup" global. good.

Potential issue with `new THREE.MeshBasicMaterial({ map: dialTexture, transparent: true, side: THREE.DoubleSide })`: `MeshBasicMaterial` default `depthWrite: true`. If we leave depthWrite true, it writes depth. But it is transparent, so should not? Actually transparent + depthWrite true can cause issues. Need set `depthWrite: false` to avoid. Do that for dial, date, lcd. Good.
But if dial does not write depth, the case front cap behind has depth, but if the dial is in front and transparent, okay. If there are transparent objects behind? no.
Need perhaps if the date overlay/lcd don't write depth, but hands write. good.

Need perhaps if the dial is transparent and depthWrite false, the case front cap behind may be visible through dial (except dial alpha 1). Dial itself has alpha 1 in circle, so it covers. no.

Need perhaps if the date overlay and LCD are transparent but alpha 1 in their backgrounds; okay.

Need maybe if `createDialTexture` returns texture and no `needsUpdate` because initial. good.

Need maybe if `dateCtx` and `lcdCtx` are created after `createWatch`? We can create in init before `createWatch`? Let's structure:

Inside IIFE:
```
let currentTZ = 'Asia/Tokyo';
let tzFormatter = null;
let tzFormatterName = '';
let autoRotate = true;
let clockGroup;
let dateTex, lcdTex, dateCtx, lcdCtx;
let renderer, scene, camera, controls;

init();
animate();

function init() {
  renderer = new THREE.WebGLRenderer({ antialias:true });
  ...
  scene = new THREE.Scene();
  ...
  // create dynamic canvases
  const dateCanvas = document.createElement('canvas'); ...
  dateCtx = dateCanvas.getContext('2d');
  dateTex = new THREE.CanvasTexture(dateCanvas);
  const lcdCanvas = ...
  lcdCtx...
  lcdTex...
  clockGroup = createWatch();
  scene.add(clockGroup);
  ...
  updateDisplays();
  ...
}
```
Need createDialTexture uses renderer, so after renderer created. Good.

Need define `createWatch` after functions? Function declaration hoisted. okay.

Need maybe in `createWatch`, use `dateTex` and `lcdTex` from outer. But dateTex/lcdTex are created before watch. Good.

Need perhaps if `createDialTexture` uses `renderer.capabilities` and `renderer` defined. yes.

Let's write code.

Potential issue: `const dateCanvas = document.createElement('canvas'); dateCanvas.width = 256; dateCanvas.height=128;` Then `dateCtx = dateCanvas.getContext('2d');` Then `dateTex = new THREE.CanvasTexture(dateCanvas);` Then in `createWatch`, material uses `dateTex`. Good.
Need `dateTex.anisotropy = renderer.capabilities.getMaxAnisotropy();` maybe not necessary. Could set.

Need maybe if `lcdTex` and `dateTex` need `magFilter = THREE.NearestFilter` for LCD? Maybe not.

Need perhaps if "smooth seconds" and digital LCD update at 100ms, no issue.

Need perhaps if the watch case material uses `MeshPhongMaterial` with `color:0xbfc3c7` and `specular:0x333`; okay.
Need perhaps if the bezel material is dark grey `0x888888`; good.

Need perhaps if the strap material:
```
new THREE.MeshPhongMaterial({ color:0x5a3c2c, specular:0x111, shininess:20 });
```
Need maybe if "leather" color. Good.

Need maybe if the lugs material caseMat. good.

Need maybe if top strap position and lower strap: Since strap height 2.6, center y=3.3, from y=2.0 to 4.6. The case top edge y=2, so it attaches. The strap width 1.5, x -0.75 to 0.75, inside case radius? At y=2.1, x=±0.75, radius 2.23 > 2? Wait at y=2.1, boundary of case? The case is a cylinder of radius 2, so points with y=2.1 are outside (except no). The strap is attached outside the case perimeter? Actually at y=2.1, x=0, radial distance 2.1, outside the case (case radius 2). So the strap is not attached to the case edge, but floats just above it with a gap. Lugs at y=1.8? Let's redesign lugs to bridge.

Case radius 2. The top of case is around y=2. We need a strap or lug from y=1.8 to y=4.5. If strap center y=3.1, height 2.4 gives from 1.9 to 4.3. It will intersect the case at y=1.9? At y=1.9, x=0, radius 1.9 inside case; so strap is partially inside the case and emerges at y>2. Good. Let's set upper strap center y=3.1, height 2.5 -> from 1.85 to 4.35. It starts inside case, reasonable. Width 1.5, z=-0.25. It will be partly hidden by case/behind. Good. Lower strap center y=-3.1. Good.
Add lugs? Maybe no need. But a box crossing the case edge at y=1.85, x=±0.75? The box's corners may be inside case and invisible, but it will show outside. Good.
Need perhaps if top strap intersects the case wall at side near y=1.85; but since strap is behind (z=-0.25) and case extends z [-0.35,0.35], the strap box at z [-0.45,-0.05] intersects the case volume. It will be hidden inside by the case walls? The case is solid but not really; rendering will show the strap mesh inside the case if it intersects because both meshes are opaque and depth test. The case front/side walls may occlude depending angle. The strap inside the case might not be visible due to case side. Fine. No problem.
Need perhaps if the strap boxes extend through the case from front to back? thickness 0.4 centered at z=-0.25, from -0.45 to -0.05; case thickness -0.35 to 0.35; it overlaps. But okay.

Need perhaps if lugs add complexity. Let's include lugs:
```
function addLug(y) {
 const lug = new THREE.Mesh(new THREE.BoxGeometry(1.5,0.7,0.55), caseMat);
 lug.position.set(0, y*1.9? maybe if y=1, position y=1.7, z=-0.2);
 watch.add(lug);
}
addLug(1); addLug(-1);
```
Top lug at y=1.65? Actually case radius 2; lugs should extend from y=1.6 to 2.3. Box height 0.7 centered at 1.9 => from 1.55 to 2.25. Width 1.5, z=-0.25. It bridges case edge. Add top and bottom. Then strap center y=3.15, height 2.4 => from 1.95 to 4.35, overlapping lugs. Good.
Let's implement lugs:
```
function createLug(sign) {
  const lug = new THREE.Mesh(new THREE.BoxGeometry(1.4,0.7,0.5), caseMat);
  lug.position.set(0, sign*1.9, -0.2);
  watch.add(lug);
}
createLug(1); createLug(-1);
```
Then straps:
```
const strapMat = new THREE.MeshPhongMaterial({color:0x5a3c2c, specular:0x111, shininess:20});
const upperStrap = new THREE.Mesh(new THREE.BoxGeometry(1.5, 2.6, 0.4), strapMat);
upperStrap.position.set(0, 3.2, -0.3);
watch.add(upperStrap);
const lowerStrap = upperStrap.clone();
lowerStrap.position.y = -3.2;
watch.add(lowerStrap);
```
Need if strap z=-0.3, lugs z=-0.2. okay.

Need maybe if the strap should be one continuous band around the wrist; two boxes enough.

Need perhaps if auto rotate, the strap boxes make watch look 3D.

Need maybe if the case should have "spring bar" holes? no.

Need maybe if there is a "crown" on the right, but if the strap is boxes, crown visible.

Need perhaps if there are "crown guards" on case? no.

Need maybe if "date window" at 3 o'clock and crown on right, consistent.

Need perhaps if "second timezone LCD" at 6 o'clock, okay.

Need maybe if the date window at 3 o'clock with crown on right, good.

Need maybe if the dial has "AUTOMATIC" text at top; could be under 12. We draw at y=80 (top). The 12 number at y=212? Wait 12 at canvas y=512-300=212. Brand at y=80 is above 12, near edge. okay.
Need perhaps if "SWISS 3D" at y=650, below center but above 6 (y=812), okay. It might be covered by LCD? LCD at y=-0.95 world maps to canvas y=512+263=775, from y 687 to 863. "SWISS 3D" at y=650, not covered. good.

Need perhaps if "AUTOMATIC" at y=80 and hour markers etc. okay.

Need perhaps if the date window at world x=1.0,y=0 maps to canvas x=789,y=512. It covers 3 number at x=812,y=512. good.

Need maybe if the LCD at world (0,-0.95) maps to canvas x=512, y=512+263=775. It covers 6 number? 6 number at y=812, x=512. LCD vertical range from 687 to 863, so it covers 6 number. Wait LCD size 2.1x1.05: in canvas, width 581 px, height 290 px. Center y=775, height 290 => y 630 to 920. It covers 6 number at y=812. Is that okay? A large LCD at 6 might obscure the 6 numeral. But if the LCD has black background, it will cover the 6. Maybe not ideal. We can move LCD slightly lower? No, if lower it covers more. Or smaller and above 6. Let's reduce LCD size to 1.8 x 0.7 maybe, and position y=-1.0. Then canvas height 194 px, center y=775? Wait world y=-1.0 maps to canvas 789, height 194 => 692 to 886, still covers 6 at 812. Hmm. The 6 numeral is at y=-1.08; LCD should be above it, maybe center y=-0.75? If center y=-0.75 maps to canvas 512+207=719, height 194 => 622-816, still covers 6 lower half. The 6 is at 812; it would cover. Maybe place LCD at y=-1.2, but then the 6 marker is at -1.08, LCD below, not cover, but may go outside dial? Dial radius 1.85, LCD bottom -1.2-0.35=-1.55, still inside. But the 6 number is above it. However the second hand at 30s points down to -1.6, crosses numbers, okay. Let's set LCD center y=-1.15, size 1.8 x 0.7. Then top -0.8, bottom -1.5. The 6 number at -1.08 is above top? Actually -1.08 is less than -0.8? Wait coordinate y: -0.8 is above -1.08? On canvas, smaller y = up. The LCD top y=-0.8, 6 marker y=-1.08 is lower than top? Let's compare: top of LCD at -0.8, bottom at -1.5. The 6 marker at -1.08 is within the LCD vertical range (-0.8 to -1.5) because -1.08 is between. So LCD covers it. Move LCD center y=-1.3, top -0.95, bottom -1.65; 6 marker at -1.08 within range again. To avoid 6, LCD must be entirely above or below it. 6 at -1.08. If center y=-0.7, top -0.35, bottom -1.05, it just avoids -1.08. But then LCD overlaps center/bottom; maybe okay. If center y=-1.55, top -1.2, bottom -1.9, outside dial bottom; not good. So place LCD above 6: center y=-0.7, size 1.8x0.7. Then bottom -1.05; 6 marker at -1.08 just below. Good. But the LCD center at -0.7 is between center and 6, may be visually okay. It will overlap the "SWISS 3D" text at y=0.5? Wait world y=-0.7 maps to canvas y=512+194=706; SWISS 3D at y=650, no. The center of dial has hands; LCD top at -0.35, may come close to center but not too much. The minute/hour hands may pass over it, okay. Let's set LCD `position.set(0, -0.7, 0.375)`, `PlaneGeometry(1.9, 0.7)` maybe. World size 1.9x0.7; canvas 512x256? Need canvas aspect 512/256=2, plane 1.9/0.7=2.714, not matching. Need maintain aspect. Let's choose canvas 512x192? Then aspect 2.67, plane 1.9x0.7125. Or simply set plane 2.0x0.75 and canvas 512x192? Wait canvas 512x192 aspect 2.67; plane 2.0/0.75=2.67. Good. Let's use lcdCanvas.width=512, height=192. `lcdMesh` geometry 2.0 x 0.75. Text layout:
  - line1 at y=44
  - line2 at y=100
  - line3 at y=150
Need adjust.
But if canvas height 192, text okay.
Alternatively set lcdCanvas 512x256 and plane 2.0x1.0 to same aspect 2; but 1.0 height large. Hmm.
Let's keep 512x256 and plane 2.1x1.05, but center y=-0.7? Size 1.05 high, top -0.175, bottom -1.225. This covers 6 at -1.08, not good. So reduce height to 0.7. Need canvas aspect of plane 2.0x0.75 = 2.67. Use canvas 512x192. Good.
Let's implement LCD canvas 512x192. Then draw:
```
lc.font='bold 26px monospace'; lc.fillText(`${label}  ${time}`, 20, 32);
lc.font='bold 22px monospace'; lc.fillText(`${tzDate} ${tzWeek}`, 20, 80);
lc.font='18px monospace'; lc.fillStyle='#666'; lc.fillText('SECOND TIMEZONE', 20, 130);
```
Need not be too small. Plane 2.0x0.75, text maps. okay.
Position LCD at y=-0.7. Then bottom -1.075, 6 number at -1.08, just below. Good. But the 6 marker at -1.08 may still be partially overlapped by border (bottom edge). fine.
Need maybe if second timezone LCD is too high, near date? It at y=-0.7, date at y=0. no overlap.
Need perhaps if the lower strap and case behind, no.

Need perhaps if the dial static "SWISS 3D" at y=0.5? In canvas y=650 -> world y=(512-650)/276.76=-0.5. LCD at y=-0.7, top -0.325; it might cover "SWISS 3D" at y=-0.5? top -0.325, so above; "SWISS 3D" at -0.5 is below top, so LCD covers it maybe. Need move "SWISS 3D" elsewhere or remove. We can draw "SWISS 3D" at top? Let's adjust brand text:
- "AUTOMATIC" at y=80 (brand above 12)
- "3D EDITION" at y=600? Actually center y=512, y=600 below center. LCD top -0.325 world maps to canvas y=512+90=602. So if "3D EDITION" at y=600, LCD covers it. Place "3D EDITION" at y=550? That is also covered. Maybe remove lower text or place in upper area below 12? Let's just draw brand text at top and no lower text. Or draw "SWISS 3D" at top y=140? But 12 number at y=212; brand at y=80, maybe okay. We can draw "AUTOMATIC" at y=90 and "3D EDITION" at y=140. The 12 number at y=212, no overlap. Good.
In createDialTexture:
```
ctx.fillText('AUTOMATIC',512,90);
ctx.font='24px Arial'; ctx.fillText('3D EDITION',512,140);
```
Then center clear. Good.

Need maybe if date window covers 3 numeral; okay.

Need maybe if LCD size 2.0 x 0.75, position y=-0.7, top -0.325, bottom -1.075. It covers 6 numeral? 6 numeral at y=-1.08, just below border, so no. It covers minute markers? no.

Need maybe if label "SECOND TIMEZONE" in LCD line3 at y=130; canvas height 192, okay.

Let's set LCD canvas width=512, height=192. Plane 2.0 x 0.75. Good.

Need perhaps if the date window canvas 256x128 and plane 1.1 x 0.55; position (0.95,0,0.375). Good.
Need maybe if date window covers 3 number at (1.08,0) and plane width 1.1 x 0.55; okay.

Need maybe if the second hand when pointing right at 15s passes over date window, but date window is behind second hand. okay.

Need perhaps if "3D wristwatch" should have "lume" maybe not.

Need maybe if "current time" from system and "date/week" in date window; good.

Let's think about if `autoRotate` checkbox and `clockGroup.rotation.y` update causes the watch to rotate, but the `clockGroup` variable is assigned. Need if `autoRotate` true, and user unchecks, rotation stops. Good.

Need maybe if `controls.update()` every frame uses damping; if no damping, okay.

Potential issue: If `autoRotate` is true and `controls.update()` also has autoRotate? We are not using controls.autoRotate. okay.

Need perhaps if `THREE.OrbitControls` in r128 expects `controls.rotateSpeed` etc. default.

Need maybe if `requestAnimationFrame(animate)` with `animate` defined. good.

Let's now write the final code with all these decisions.

Need ensure no accidental undefined variable:
- `weekdaysCN` defined.
- `pad` defined.
- `createDialTexture` defined.
- `createHand` function name maybe `createHandMesh`.
- In `createWatch`, local `dateMesh`, `lcdMesh`. Need use `dateTex` and `lcdTex` from outer.
- Need if `dateTex` and `lcdTex` are initialized before `createWatch`. yes.

Let's draft code in final. We can include comments.

Potential issue: `dateMesh` and `lcdMesh` use `new THREE.MeshBasicMaterial({ map: dateTex, transparent: true, depthWrite: false })` with default `side` FrontSide. When watch rotates and face points away, they vanish; okay.
Need perhaps if they should also be visible from front when watch yaw is not exactly +Z? The plane normal +Z in local group; if group rotates y, normal changes. It is visible when normal has positive component toward camera. When watch rotates 90°, face normal +X, camera at z positive, not visible. That's realistic. Good.

Need perhaps if case material uses no DoubleSide, case side visible from all around. If camera behind, front cap not visible, back cap visible. Good.

Need perhaps if case's cylinder has bottom cap with normal -Z, visible from behind. good.

Need perhaps if the dial plane is transparent and DoubleSide, if camera behind, it could show through the back cap? The back cap is opaque and would occlude, so no.

Need maybe if `MeshPhongMaterial` for case with caps and side has no envMap, but lights enough. yes.

Need perhaps if no `THREE.CubeTextureLoader`; no.

Let's think about if `createDialTexture` uses `ctx.fillText` for all 12 numbers. At 1024x1024, numbers could be too large (80px) and close to minute markers. Use 56px. Let's set `font='bold 56px Arial'`. Good.
Need if labels 12/3/6/9 maybe no. all 12 okay.
Need maybe if text positions for 6/12 with R=300 and font 56; fine.
Need if numbers "12" at top y=212; "6" at bottom y=812; okay.

Need perhaps if the hour markers are drawn after numbers? Actually draw markers then numbers; if numbers overlap markers? With R=300 and markers from 400 to 440, no. Good.
Need perhaps if minute track with markers at r=440/420 and hour markers at r=440/400; okay.
Need outer stroke at r=470; markers within. good.

Need perhaps if the dial circle radius 500, outer stroke at 470, minute markers from 440 to 420, leaving 30px gap. good.

Need perhaps if the date window covers 3 marker, but the static marker at 3 remains under; opaque white covers. good.

Need maybe if the "3D EDITION" text at y=140 and 12 number at y=212; okay.

Need maybe if "AUTOMATIC" too close to 12 top; okay.

Need perhaps if "automatic rotate" is checkbox, but we don't need "pause" etc.

Let's now compose final answer. Need maybe "下面是完整 HTML..." then code block. Then "运行方式" and "测试说明".

Need ensure final code not too long but acceptable. We have enough tokens.

Let's write code now.

Potential issue: In `createWatch`, in `caseMesh` and `bezel`, if the `caseMesh` is rotated x=PI/2, its top cap at z=+0.35. But `CylinderGeometry(2, 2, 0.7)` has top and bottom caps both radius 2. If bottom radius 2, case back cap same. Good. Maybe use radiusTop=2, radiusBottom=1.9 to taper; but if bottom radius 1.9, the cylinder side slants. The top cap (front) radius 2, bottom back radius 1.9. After rotate x, top cap z=0.35, bottom cap z=-0.35. Good. Let's set `CylinderGeometry(2, 1.95, 0.7, 64)`? This gives slight taper. But if bottom radius 1.95, case back slightly smaller. okay. Use `(2, 1.95, 0.7, 64)`. Good.
Need if top cap radius 2, bezel inner radius 1.89, dial radius 1.85. okay.

Need perhaps if the strap boxes are behind case; if case bottom radius 1.95, okay.

Need maybe if caseMesh top cap is at z=0.35; the side slopes inward toward back, so the bezel at z=0.30 with radius 1.99 may not align with side everywhere; but okay.

Need maybe if the bezel torus central radius 1.99 and tube 0.1, so outer radius 2.09; at z=0.30, it protrudes beyond case radius 2. good.

Need perhaps if the case body and bezel materials are same? Different slightly. good.

Need perhaps if "lugs" use caseMat; if caseMat created once and reused, okay.

Need perhaps if "crown" use caseMat; okay.

Need maybe if "strap" material color brown and "lugs" metal; okay.

Need maybe if "watch group" has a transparent dial, but if the `caseMesh` top cap has a circular face, and dial plane is transparent but alpha=1 in circle, it completely covers the cap. good.

Potential issue: `MeshPhongMaterial` with `specular` and `shininess` on case; if the front cap is visible behind dial, okay. If no environment, metals may look flat but okay.

Need perhaps if all time variables in `updateHands` use current date. yes.

Need if `dateTex.needsUpdate = true` each 100ms. good.

Need perhaps if `lcdTex.needsUpdate = true` each 100ms. good.

Need perhaps if `tzFormatter` created with `timeZone` and `hourCycle`; if `hourCycle` unsupported in older browser, might not update? Modern. okay.
Need perhaps if `formatToParts` returns `dayPeriod` for zh-CN? no.

Need perhaps if `tz.month` and `tz.day` strings could include leading spaces? `formatToParts` with 2-digit should no. We can `tz.month.trim()` etc. maybe. Use `String(tz.month).padStart(2,'0')`? But `2-digit` already. We can use `tz.month || ''`. Good.
Need maybe if `tz.hour` undefined if `hourCycle` problem. Maybe fallback:
```
const tzTime = `${tz.hour || '00'}:${tz.minute || '00'}:${tz.second || '00'}`;
```
Use robust.
Need maybe if `date` local contains `now.getFullYear()`; yes.
Need maybe if `dateCtx` font includes Chinese; okay.

Need perhaps if the watch should run "current time" with two timezones; yes.

Let's write final code now.
