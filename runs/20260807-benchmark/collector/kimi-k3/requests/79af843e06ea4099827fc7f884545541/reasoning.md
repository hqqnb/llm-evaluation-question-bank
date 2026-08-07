The user wants a 3D contact lens case using Three.js:
- Contact lens case with L and R markings
- Has lids/caps
- User can click lids to open them
- Single HTML file with HTML, CSS, JavaScript
- Provide run instructions and test notes

Let me design this:

1. A contact lens case typically has:
   - A base with two cylindrical wells (left and right)
   - Two screw-on caps, usually marked L and R (often different colors - e.g., white for L, blue for R, or both white with letters)
   - The caps screw off

2. Features:
   - L and R text labels - I can create these with canvas textures
   - Clickable caps that animate open (unscrew + move up, maybe tip over)
   - Liquid in the wells maybe (nice touch)
   - OrbitControls for rotation
   - Raycasting for click detection
   - Animation for opening/closing (screw motion: rotate while moving up)

Let me build this in a single HTML file. I'll use Three.js from CDN. Use import maps for modules (OrbitControls).

Structure:
- Base: rounded rectangular box or two connected cylinders. Real contact lens cases are often two cylinders connected by a bridge, or a rounded rectangle with two circular wells.
- Let me do: a base made of two cylinders joined together (capsule-like shape) with two hollow wells. To make wells look hollow, I can:
  - Create the base as cylinders
  - Add a darker cylinder inside to simulate depth (inner wall), and a liquid disc slightly below the rim
- Caps: cylinders with L/R text on top, knurled edges (ridges) for grip
- Text: use CanvasTexture to draw "L" and "R" on cap tops

Animation:
- Click a cap: it unscrews (rotates while rising), then moves to the side and tilts, resting next to the case. Click again to put back (screw down).
- Use a simple animation system with easing, or use GSAP-like manual tweening in the render loop.

Let me write manual tweening to avoid extra dependencies... Actually I can include GSAP via CDN but manual tween is fine and keeps it self-contained. I'll write a small tween helper.

Raycasting: on click, intersect cap meshes; toggle state.

UI: title, instructions ("点击盒盖打开/关闭"), maybe buttons to open both.

Let me also add:
- Shadows
- Nice lighting (ambient + directional + environment)
- Ground plane
- Background gradient
- Colors: L cap white, R cap blue (common convention)

Details on geometry:

Base:
- Two cylinder wells side by side. Well radius ~1, depth ~0.6. Base height ~0.7.
- Bridge connecting them: a box with rounded edges.
- Actually simplest: create base as a rounded box (using Shape + ExtrudeGeometry with capsule/stadium shape) — a stadium shape (two circles connected) extruded. Then wells are visual: dark inner cylinders + liquid.

Stadium shape via THREE.Shape: absarc for both ends. Extrude with bevel for rounded edges.

Wells: 
- Outer rim: torus or cylinder ring at top of base? Actually the cap screws onto a threaded neck. Simplify: each well has a raised neck (short cylinder) with thread ridges (torus rings or helical tube). Cap covers the neck.
- Inside the neck: dark circle (cylinder interior) and liquid disc (semi-transparent blue-ish).

Cap:
- Cylinder with slightly domed or flat top
- Knurling: use many small boxes around the edge, or a ridged cylinder via LatheGeometry, or just use a texture. Simpler: create ridges using a cylinder with a canvas bump or actual small boxes. Small boxes: 24 boxes around radius — cheap and looks good.
- Top: circle mesh with canvas texture showing "L" or "R".

Screw animation:
- Phase 1 (opening): rotate cap N turns while lifting by thread pitch — duration ~1s
- Phase 2: move cap to the side (next to its well) and lay it down or keep upright
- Closing reverses: move back above well, then rotate down.

State machine per cap: 'closed', 'opening', 'open', 'closing'. Prevent clicks during animation.

Implementation plan:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>隐形眼镜盒 3D</title>
  <style>...</style>
</head>
<body>
  <div id="ui">...</div>
  <canvas or container div>
  <script type="importmap">three CDN</script>
  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    ...
  </script>
</body>
</html>
```

Import map:
```json
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
```

Scene setup:
- Renderer with shadows, antialias, ACES tone mapping
- Camera perspective
- OrbitControls with damping, limits
- Lights: hemisphere/ambient + directional with shadow + maybe a fill

Dimensions:
- Well radius R = 1
- Centers at x = -1.25 (L) and +1.25 (R)
- Base height 0.55, extruded stadium shape radius 1.35 around each center? Let's define: stadium from two circles radius 1.3 at centers ±1.2. Extrude depth 0.5 with bevel 0.08.

Actually ExtrudeGeometry extrudes along z; I'll rotate -90° around x so it lies flat, height along y.

Neck: cylinder radius 0.95, height 0.35, sitting on base top (y from 0.5 to 0.85). Threads: 2-3 torus rings radius 0.95, tube 0.03, around the neck.

Interior of well (visible when cap is off): 
- Inner cylinder wall: use a cylinder with openEnded, material side BackSide, dark color, radius 0.8, from base top down to 0.15.
- Bottom disc at y=0.15.
- Liquid: cylinder radius 0.78, height 0.25, transparent blue, top at ~0.42. Slight emissive/transmission. Use MeshPhysicalMaterial with transparency.

Cap:
- Radius 1.08, height 0.45. Group: 
  - main cylinder (top closed)
  - knurl ridges: small boxes around circumference
  - top label: circle with canvas texture (L/R letter)
  - inner skirt maybe
- Cap closed position: group at well center, base of cap at y ~0.8 (overlapping neck). Cap covers neck from 0.8 to 1.25... let me compute: neck top 0.85, cap height 0.45, cap bottom at 0.55 so it overlaps base a bit? Real caps cover the neck fully. Let cap bottom = 0.62 (slightly above base top 0.5+bevel), cap top = 1.07. Hmm, just make it look right.

Let me set numbers concretely:
- Base extrude height (thickness): 0.5, plus bevel 0.06 each side => total ~0.62. Rotate so base sits on ground y=0 to ~0.62. Let's say baseTopY = 0.58.
- Neck: radius 0.92, from y=0.5 to 0.92 (height 0.42), centered at 0.71. Slightly embedded.
- Threads: tori at y = 0.72, 0.80, 0.88, radius 0.93, tube 0.025.
- Well interior: open cylinder radius 0.8, from y=0.12 to 0.92, BackSide dark. Bottom disc radius 0.8 at y=0.12.
- Liquid: cylinder radius 0.78, height 0.3, center y=0.27 (top at 0.42). Physical material, color #7ec8e3, transparent opacity 0.75, roughness 0.1. Maybe add a slight meniscus disc.

Cap closed: cap group origin at cap bottom center. bottom y = 0.60, so cap spans 0.60–1.05. Neck top 0.92 < 1.05 good, hidden inside cap. Cap radius 1.05 > neck 0.92 good.

Open animation:
- Screw: rotate around Y by 4π (2 turns) while y goes 0.60 → 1.25 over 0.9s.
- Then move cap to a "parked" position: e.g., x offset outward (away from center) and toward viewer? Parked at side of its well: parked pos = (centerX ± 2.6, 0.22, 1.2)? And maybe tilt slightly or lay on side. Keep upright, place on ground: y = 0.225 (half cap height? no—if upright, bottom at 0 → group origin y = 0). Group origin is bottom center, so parked y=0.02. Add slight rotation tilt for style: keep upright, simple. Actually laying the cap upside down (like people do) is cute: rotate π around X or Z so top faces down... but then the L/R label faces down, hidden. Keep upright; label visible. Good.

Parked positions: L cap parks to the left-front: (-1.2 - 2.2, 0.02, 1.4)? The base extends x from -1.2-1.3=-2.5 to 2.5. Park at (-3.4, 0.02, 1.6) and (3.4, 0.02, 1.6). Fine, ground plane is big.

Close: reverse — move to above well (x, 1.25+?, z), then screw down (rotate -4π while descending to 0.60).

Rotation on close: from current rotation (ended at 4π after open... I rotated +4π opening). On close rotate back to 0. Track a rotation value.

Simpler: parametrize animation with keyframes per cap: store cap.userData with target transforms and animate via custom tween of {y, rotY, x, z} through phases. I'll implement a small tween system:

```js
const tweens = [];
function tween(obj, to, duration, ease, onComplete) { ... }
```

Or chain with promises. Let me write a promise-based tween:

```js
function animateValue(duration, onUpdate, ease = easeInOutCubic) {
  return new Promise(resolve => {
    tweens.push({ t: 0, duration, onUpdate, ease, resolve });
  });
}
```

In render loop: advance tweens by dt, call onUpdate(easedT), resolve when done.

Cap open sequence:
```js
async function openCap(cap) {
  cap.userData.state = 'opening';
  const d = cap.userData;
  // unscrew
  await animateValue(900, t => {
    cap.position.y = lerp(d.closedY, d.liftY, t);
    cap.rotation.y = lerp(0, Math.PI * 4, t);
  });
  // park
  const start = cap.position.clone();
  const startRot = cap.rotation.y;
  await animateValue(700, t => {
    cap.position.lerpVectors(start, d.parkPos, t);
    // arc
    cap.position.y += Math.sin(t * Math.PI) * 0.6;
    cap.rotation.y = startRot + t * Math.PI * 0.5;
  });
  d.state = 'open';
}
```

Careful: lerpVectors then adding arc each frame — position.y gets overwritten by lerpVectors then arc added; fine.

Close sequence reverses:
```js
async function closeCap(cap) {
  d.state = 'closing';
  const start = cap.position.clone(); const startRot = cap.rotation.y;
  await animateValue(700, t => {
    cap.position.lerpVectors(start, new Vector3(d.x, d.liftY, d.z), t);
    cap.position.y += Math.sin(t*Math.PI)*0.6;
    cap.rotation.y = lerp(startRot, Math.PI*4, t); // align
  });
  await animateValue(900, t => {
    cap.position.y = lerp(d.liftY, d.closedY, t);
    cap.rotation.y = lerp(Math.PI*4, 0, t);
  });
  d.state = 'closed';
}
```

Rotation ends at 0 so the label orientation is consistent. Good.

Raycast: pointerdown + pointerup distinguishing click vs drag (OrbitControls). Track downPos; if moved < 5px, treat as click. Raycast against cap groups (children recursive true). Find ancestor with userData.isCap.

Hover: pointermove raycast → cursor pointer + slight emissive highlight.

Label texture:
```js
function makeLabelTexture(letter, bgColor, fgColor) {
  const c = document.createElement('canvas'); c.width = c.height = 256;
  const ctx = c.getContext('2d');
  // background circle
  ctx.fillStyle = bgColor; fillRect; or draw ring
  ctx.font = 'bold 160px Arial';
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillStyle = fgColor; ctx.fillText(letter, 128, 138);
  const tex = new THREE.CanvasTexture(c); tex.anisotropy = 8;
  return tex;
}
```

Cap top: circle geometry radius ~0.9 at top of cap, material with map. Also emboss ring around. L cap: white cap with blue letter? Common: L cap white, R cap blue or green. Let's do: L cap white (#f5f5f5) with gray-blue letter; R cap blue (#2f7fd1) with white letter. Also add L/R text on the base in front of wells (small decals): use small plane with canvas texture "L"/"R" on the base top between wells or on the bridge. Add text on front face maybe. Simpler: small plane slightly above base top near each well front edge.

Also add subtle "L" and "R" embossed on base? Plane decals are fine.

Base shape (stadium):
```js
const shape = new THREE.Shape();
// two circles radius 1.32 at (±1.2, 0)
shape.absarc(-1.2, 0, 1.32, Math.PI/2, Math.PI*1.5, false); // left circle lower? 
```
Stadium outline: right circle arc from -90° to 90° (right side), then line to left circle top, arc 90° to 270°, line back.

```js
const Rr = 1.32, cx = 1.2;
shape.absarc(cx, 0, Rr, -Math.PI/2, Math.PI/2, false);   // right semicircle (bottom→top going CCW? angles -90..90 passes through 0 = rightmost)
shape.absarc(-cx, 0, Rr, Math.PI/2, Math.PI*1.5, false); // left semicircle
shape.closePath();
```
absarc with aclockwise param; need correct ordering so shape is continuous. First arc ends at (cx + Rr*cos(90°), Rr*sin(90°)) = (cx, Rr) top of right circle. Second arc starts at (-cx + Rr*cos(90°), Rr) = (-cx, Rr) — top of left circle. Line connects automatically (Shape will lineTo). Ends at (-cx, -Rr), closePath lines back to start (cx, -Rr). 

Extrude: depth 0.5, bevelEnabled true, bevelThickness 0.06, bevelSize 0.06, bevelSegments 4, curveSegments 48. Then rotateX(-PI/2) → shape XY plane becomes XZ, extrude z→ -y? rotateX(-π/2): z axis → y... Let me think: geometry.rotateX(-Math.PI/2) maps +z to +y? Rotation about x by -90°: y→ -z? A point (0,0,1) rotated by -90° about x: (0, 1, 0)? Rx(θ): y' = y cosθ - z sinθ; z' = y sinθ + z cosθ. θ=-90°: cos=0, sin=-1. y' = z, z' = -y. So (0,0,1) → (0,1,0)? y' = 0*0 - 1*(-1) = 1. Yes +z → +y. And (0,1,0) → y'=0, z'=-1 → (0,0,-1): +y → -z. Good: shape's y becomes -z (mirror in z, fine for symmetric shape). Extrusion from z=0..0.5+bevels becomes y=0..~0.62? With bevel, z from -0.06 to 0.56 → y -0.06..0.56. Shift geometry up by 0.06: translate(0, 0.06, 0) → base spans y 0..0.62. baseTop ≈ 0.62. Then neck from 0.55 to 0.97 etc. Let me just recompute cleanly:

- BASE_TOP = 0.62
- Neck: height 0.4, center y = 0.62+0.2-0.08(overlap) → from 0.54 to 0.94. radius 0.92.
- Threads tori at 0.70, 0.80, 0.90.
- Well inner wall radius 0.8 from 0.12 to 0.94. Bottom at 0.12.
- Liquid top ~0.45.
- Cap: radius 1.06, height 0.5, closed bottom at 0.58 (slightly overlapping base top to hide seam), top 1.08. Lift Y = 1.35 (cap bottom), above neck top 0.94 with clearance. Screw: y 0.58→1.35 while rotating 2 turns.

Cap group structure (origin at bottom center):
- shell: CylinderGeometry(1.06, 1.06, 0.5, 64, 1, true)? openEnded with separate top: top disc circle radius 1.06 at y=0.5. Plus top label circle radius 0.8 with texture slightly above (0.501 → avoid z-fight, use 0.505 and polygonOffset). Also a slight domed ring: torus radius 0.95 at top edge. 
- knurls: 28 boxes size (0.06, 0.34, 0.1)? placed around radius 1.06: each box at angle, rotated. Box dims: width tangential 0.12, height 0.36, depth radial 0.07; position radius 1.045 embedded. Use InstancedMesh or merge — just 28 meshes per cap ×2 = 56 meshes, fine. Or a single cylinder with flat shading + bump texture. I'll do boxes for realism; group them.

Actually simpler & cleaner: use one knurl cylinder via CylinderGeometry with many segments and a canvas roughness/bump? Boxes look better. Keep boxes but create once and clone.

Bottom inner skirt of cap: dark cylinder inside (visible when cap parked? from bottom). Add cylinder radius 0.9, height 0.3, dark gray, open at bottom? Eh, optional: add inner disc at y=0.02 dark to suggest interior. Since parked upright, bottom not visible. Skip; but during flight maybe. Add a simple dark inner cylinder openEnded BackSide radius 0.92 height 0.45 + inner top disc. Cheap.

Ground: big circle, ShadowMaterial or standard material light gray. Soft shadows (PCFSoft). 

Background: scene.background gradient — use scene.fog + solid color, or CSS gradient behind transparent canvas (renderer alpha true). I'll do CSS radial gradient body background, renderer alpha: true. Nice look.

Also add a subtle contact lens prop? Maybe not needed. Keep scope: case + caps + liquid + labels + click.

Extra UI: buttons "全部打开" / "全部关闭", hint text, cap status display. Loading is instant.

Edge cases:
- Rapid clicks: guard by state.
- Both caps animate independently.
- OrbitControls conflicts: click vs drag detection via movement threshold and also only on pointerup within time.

Sound? skip.

Let me also handle WebGL absence gracefully-ish. Fine.

Now write the tween system with clock delta:

```js
const activeTweens = new Set();
function tween({duration, ease = easeInOutCubic, onUpdate}) {
  return new Promise(resolve => {
    activeTweens.add({ elapsed: 0, duration, ease, onUpdate, resolve });
  });
}
// in loop:
for (const tw of [...activeTweens]) {
  tw.elapsed += dt * 1000;
  const raw = Math.min(tw.elapsed / tw.duration, 1);
  tw.onUpdate(tw.ease(raw));
  if (raw >= 1) { activeTweens.delete(tw); tw.resolve(); }
}
```

Easing functions: easeInOutCubic, easeOutBack maybe for parking.

Liquid wobble when cap opens? Nice-to-have: animate liquid top slight scale pulse. Skip or simple: on open, small ripple via scaling liquid. I'll add a tiny bounce on the liquid when cap finishes opening — cheap charm. Actually keep code focused; maybe skip. Hmm, small detail adds delight. I'll add a quick wobble: liquid.scale pulsate for 500ms. Easy with tween.

Hover highlight: store each cap's materials, on hover set emissive intensity. MeshStandardMaterial emissive. For label circle too. Implement cap.userData.setHighlight(bool).

Colors & materials:
- Base: white-ish plastic (#f2f4f6), roughness 0.35, slight clearcoat? MeshPhysicalMaterial clearcoat 0.6. 
- Bridge maybe same.
- Neck & threads: same as base.
- Interior: #dfe6ea dark? Actually interior should look shadowed: color #b9c4cc with roughness, BackSide. Hmm BackSide renders inside of cylinder — set color #cfd8de.
- Liquid: #8fd3f4 transparent.

L/R base decals: plane 0.5×0.5 with transparent canvas texture (letter only), placed on base top between the wells? The wells occupy most top area. Stadium width: centers ±1.2, radius 1.32 → top area around each neck radius 0.92+~0.14... Cap radius 1.06 covers most. Decal on front vertical face? Curved. Put decals on the base top near front edge: z = +0.95? Distance from center axis: the stadium edge at z max = 1.32. Neck radius 0.92 occupies up to z 0.92 at x=±1.2... Decal at (±1.2, 0.625, 1.12)? That's on the bevel maybe. Radius from well center (1.2,0) to (1.2,1.12) = 1.12 < 1.32 ✓, but neck at 0.92 — 1.12 > 0.92 ✓. Cap radius 1.06 covers up to 1.06 → decal at 1.12 partially under cap? Cap bottom 0.58 above base top 0.62? Wait cap bottom 0.58 < base top 0.62 — cap overlaps base side region radius 1.06 around center. Decal at 1.12 radial distance → outside cap ✓. Base top at y 0.62 minus bevel... bevelSize 0.06 shrinks top face to radius 1.32-0.06=1.26 flat region. 1.12 < 1.26 ✓. Place decal plane at y 0.621 rotated -90° x. Good: letters L/R on base in front of each cap. 

Text orientation: plane rotated so letter faces up and reads from front (+z). Plane rotated x=-π/2; canvas texture upright → after rotation, canvas up direction maps to -z? rotateX(-π/2): +y→ -z... plane's local +y (texture top) maps to -z (away from viewer). So letter top points away; viewer at +z sees it upright? If letter top points to -z and viewer looks from +z toward -z, the top of the letter is far side — appears upright. ✓.

Cap label: circle geometry rotated -π/2 facing up; same orientation logic: texture top → -z. When cap parked at front (z+), label reads upright from front view. During unscrew rotation it spins—fine, ends aligned (rotation 0 or multiple of 2π... open ends at 4π + 0.5π park rotation! Then label rotated 90°. On close we return to 4π then to 0. So parked label turned 90° — acceptable, or make park rotation 2π to keep aligned: startRot=4π, add π*0.5 → 4.5π ≡ 0.5π. Meh. Let me make park spin full 2π extra? That's a lot of spinning. Alternative: no extra rotation during park; keep 4π ≡ 0 → label upright when parked. Nice and clean. I'll add tiny tilt instead? Keep upright, no tilt. Fine.)

Cap colors: L white cap, R blue cap; base letters: L blue-ish gray, R blue. Also little colored dots? Keep.

Also bridge grip texture on base sides? skip.

Shadow: directional light position (5, 8, 4), shadow camera sized ~10.

Ground: circle radius 30, color #e8edf2 (or transparent with ShadowMaterial opacity 0.2 over CSS gradient). ShadowMaterial gives clean look with CSS bg. Use ShadowMaterial opacity 0.25. But then no ground plane visible—fine, floating look with soft shadow. Also add fog? skip.

Add subtle environment lighting: RoomEnvironment via PMREMGenerator for nice plastic reflections:
```js
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
```
This greatly improves materials. Include it. (RoomEnvironment available in 0.160; in newer versions RoomEnvironment() doesn't need renderer arg — correct, fromScene(new RoomEnvironment()).)

Version pin: three@0.160.0 unpkg. Good.

UI overlay:
- Top-left: title "隐形眼镜盒 · Contact Lens Case", hint "点击盒盖可拧开 / 合上 · 拖动旋转视角 · 滚轮缩放"
- Bottom center buttons: 打开双盖 / 关闭双盖
- Status text per cap? optional small legend: "L 左眼 / R 右眼". Add legend.

Cursor pointer on hover.

Let me now also think about cap construction detail:

```js
function createCap(letter, capColor, letterColor) {
  const group = new THREE.Group();
  const mat = new THREE.MeshPhysicalMaterial({ color: capColor, roughness: 0.25, clearcoat: 0.8, clearcoatRoughness: 0.25 });
  const shell = new THREE.Mesh(new THREE.CylinderGeometry(1.06, 1.09, 0.5, 72, 1, true), mat);
  shell.position.y = 0.25;
  // top plate
  const top = new THREE.Mesh(new THREE.CircleGeometry(1.06, 72), mat);
  top.rotation.x = -Math.PI/2; top.position.y = 0.5;
  // slight dome? add torus rim
  const rim = new THREE.Mesh(new THREE.TorusGeometry(0.98, 0.08, 16, 72), mat);
  rim.rotation.x = Math.PI/2; rim.position.y = 0.5;
  // label
  const label = new THREE.Mesh(new THREE.CircleGeometry(0.78, 64), new THREE.MeshStandardMaterial({ map: makeLabelTexture(letter, letterColor), roughness: 0.4, transparent: true }));
  label.rotation.x = -Math.PI/2; label.position.y = 0.585; // above rim? rim tube at 0.5+0.08=0.58
```

Hmm rim torus at y=0.5 with tube 0.08 → top 0.58. Label at 0.505 flat inside rim circle (rim radius 0.98, label 0.78 fits). Label texture: draw letter on transparent background? Cap top color shows through. Or label as printed letter directly — transparent texture with just letter, applied on circle. Good: makeLabelTexture draws only the letter (transparent bg). Then no separate plate needed; but a subtle inset circle looks nice: draw a faint ring + letter.

Knurling: instead of boxes, alternative: use CylinderGeometry with 36 radial segments and flatShading + modifying vertices? Boxes simpler:

```js
const knurlGeo = new THREE.BoxGeometry(0.09, 0.36, 0.14);
for (let i=0;i<36;i++){
  const a = i/36*Math.PI*2;
  const k = new THREE.Mesh(knurlGeo, mat);
  k.position.set(Math.cos(a)*1.06, 0.25, Math.sin(a)*1.06);
  k.rotation.y = -a; // box depth axis radial: box z-dim 0.14 radial
  group.add(k);
}
```
Box (0.09 x, 0.36 y, 0.14 z); rotated so z points outward: at angle a position (cos, sin); to have local +z radial, rotation.y = ... default box z-axis points +z; we need +z direction = (cos a, 0, sin a) → rotation.y = π/2 - a? Rotation about y: local z (0,0,1) → (sin ry, 0, cos ry). Want (cos a, 0, sin a) → sin ry = cos a, cos ry = sin a → ry = π/2 - a. Yes.

Shadows: castShadow on cap parts & base; ground receive. 36+ boxes cast shadows—ok.

Performance fine.

Inner of cap (visible mid-air): add inner cylinder BackSide:
```js
const inner = new THREE.Mesh(new THREE.CylinderGeometry(0.95, 0.95, 0.42, 48, 1, true), new THREE.MeshStandardMaterial({ color: 0x9aa7b0, side: THREE.BackSide, roughness: 0.6 }));
inner.position.y = 0.24;
const innerTop = new THREE.Mesh(new THREE.CircleGeometry(0.95, 48), sameMatFront); innerTop.rotation.x = Math.PI/2 (faces down?) → face downward visible from below: rotation.x = Math.PI/2 makes normal -y. position.y = 0.45.
```
Also threads inside? skip.

Well assembly per side:
```js
function createWell(x) {
  const g = new THREE.Group();
  // neck
  neck cylinder r 0.92 h 0.42 at y 0.73 (0.52..0.94), white mat
  // threads: 3 tori r 0.925 tube 0.028 at y 0.66, 0.78, 0.90
  // inner wall: cylinder r 0.8 openEnded BackSide color #c7d2da from 0.12 to 0.93 h 0.81 center 0.525
  // bottom: circle r 0.8 at y 0.125 facing up color same
  // liquid: cylinder r 0.78 h 0.3 center y 0.30 (0.15..0.45), physical transmission-ish: color 0x9fd8f0, transparent, opacity 0.7, roughness 0.05, metalness 0, ior 1.33? transmission 0.6 maybe heavy; simple transparent fine + a top disc slightly more opaque to fake surface
}
```
Add a tiny contact lens in the right well? A small transparent curved disc (sphere segment) floating — cute detail! Lens: SphereGeometry(0.45, 32, 16, 0, 2π, 0, ~0.5) scaled y, transparent. Could be nice but risky visually; include subtly in one well. I'll add a lens in each well: very transparent (opacity 0.25) bowl at liquid surface. Lens: use LatheGeometry or sphere cap: SphereGeometry(radius 0.5, thetaLength 0.6 rad), oriented opening up, positioned y ~0.42. Looks like a contact lens. OK add.

Base decals: "L"/"R" planes + maybe small text "左"/"右"? Just L and R letters per requirement.

Also the caps themselves have big L/R on top — main requirement ✓, plus base decals ✓.

Raycast targets: cap groups. userData: { isCap, side:'L'|'R', state:'closed', closedY:0.58, liftY:1.5, home:Vector3, park:Vector3 }.

liftY: neck top 0.94; cap bottom must clear 0.94 → lift to 1.0 with margin during screw: y 0.58→1.05 while unscrew? Threads would collide visually but fine — screwing means threads engaged; visually rising while rotating is the effect. Lift end 1.15. Park: (±3.6, 0.02, 1.9). Ground plane at y=0, cap origin bottom → parked y=0.02? Caps parked on ground plane which is y=0 → y=0. Actually put 0.0. Hmm knurls bottom at 0.25-0.18=0.07, shell from 0. Fine, bottom edge at y=0 sits on ground ✓ (shell cylinder from y 0..0.5, bottom open — edge touches ground, fine visually).

Park further: z=2.0, x=±3.7. Camera default at (4.5, 4.5, 7)? Controls target (0, 0.5, 0). Ensure parked caps visible ✓.

Buttons: open both / close both call toggle with desired state.

Also keyboard? skip.

Now write makeLabelTexture:

```js
function makeLabelTexture(letter, color) {
  const s = 512;
  const c = document.createElement('canvas'); c.width = c.height = s;
  const ctx = c.getContext('2d');
  ctx.clearRect(0,0,s,s);
  // subtle ring
  ctx.strokeStyle = color; ctx.globalAlpha = 0.35; ctx.lineWidth = 14;
  ctx.beginPath(); ctx.arc(s/2, s/2, s*0.44, 0, Math.PI*2); ctx.stroke();
  ctx.globalAlpha = 1;
  ctx.fillStyle = color;
  ctx.font = `900 ${s*0.58}px Arial, sans-serif`;
  ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  ctx.fillText(letter, s/2, s/2 + s*0.03);
  const tex = new THREE.CanvasTexture(c);
  tex.anisotropy = renderer.capabilities.getMaxAnisotropy();
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}
```

Base decal texture similar with transparent bg, letter only, maybe with small "左/右"? Keep L/R plus Chinese under: draw letter large + small text? Nice: L with "左" beneath? The requirement says 醒目的 L 和 R 标识 — big L and R. I'll include small 左/右 subtitle on base decal. On cap just big letter.

Decal plane: 0.62×0.62 at (±1.2, 0.625, 1.13), rotation.x=-π/2. polygonOffset to avoid z-fight; small offset y 0.626 enough.

Wait base top flat at y = 0.5+0.06(bevel)=0.56? Recompute extrude: depth 0.5, bevelThickness 0.06 both sides: z from -0.06 to 0.56. After rotateX(-π/2): y from -0.06 to 0.56. translate y +0.06 → 0..0.62. Top flat face at y=0.62, flat region radius shrunk by bevelSize 0.06 → 1.26. ✓ baseTop=0.62. Decal y=0.626.

Neck overlap: neck from 0.5 to 0.94: cylinder h 0.44 center 0.72, r 0.92. Threads at y 0.62? Threads should be on exposed neck above base: 0.66..0.92: tori at 0.70, 0.79, 0.88, r 0.925.

Cap closed bottom 0.58: covers threads from 0.66? bottom 0.58 covers 0.58..1.08 → covers threads ✓ and overlaps base top slightly (0.58<0.62) hiding seam ✓. Cap radius 1.09 bottom flare vs base radius 1.32 fine.

Unscrew lift: bottom 0.58 → 1.30 (above neck top 0.94 with clearance) over rot 4π. Then park.

Camera: fov 40, pos (5.2, 4.2, 8.2), target (0,0.6,0). minDistance 4, maxDistance 20, maxPolarAngle ~1.45 (avoid below ground).

Resize handler. Pixel ratio clamp 2.

Click detection:
```js
let downX, downY, downTime;
dom.addEventListener('pointerdown', e => { downX=e.clientX; downY=e.clientY; downTime=performance.now(); });
dom.addEventListener('pointerup', e => {
  if (Math.hypot(e.clientX-downX, e.clientY-downY) < 6 && performance.now()-downTime < 400) handleClick(e);
});
```

handleClick: raycast; find cap; toggle.

Hover: pointermove → raycast → canvas.style.cursor; highlight.

Highlight: cap.userData.materials array; set emissive. MeshPhysicalMaterial has emissive. On hover: emissive 0x222222? Better: use emissive color matching cap with intensity. Simpler: store mats; hover → m.emissive.setHex(0x333333); off → 0x000000. Label mat standard also.

Now assemble full code. Also status text update: "L 盖：已打开" etc. Provide small panel.

Also double-click empty space to reset camera? skip; add "重置视角" button — helpful. controls.reset() needs saved state: controls.saveState() after init. Add button.

Test instructions: open file via local server (module imports from CDN require internet; file:// with type=module + importmap works? importmap works on file://, CDN fetch requires network but not blocked by CORS since unpkg sends CORS headers ✓. Actually ES modules from file:// can import https URLs fine. Yes, opening file directly should work in Chrome as long as online. But some browsers restrict module scripts on file://? Chrome allows module scripts from https imports within file:// page... Module scripts on file:// are subject to CORS; importing remote https module from file origin — allowed with CORS headers (unpkg has Access-Control-Allow-Origin: *). I believe it works. Still, recommend `npx serve` or python http.server as fallback.)

Testing notes: 
- Verify caps open on click with screw animation
- Verify click again closes
- Drag vs click disambiguation
- Buttons work
- Responsive resize

Let me write the final HTML now, complete and careful.

Also colorSpace: renderer.outputColorSpace default SRGB in modern three ✓.

One more detail: cap label texture orientation — CircleGeometry rotated x=-π/2: texture v... CircleGeometry lies in XY facing +z; rotate -π/2 about x → faces +y, local +y(texture top) → -z ✓ reads from front(+z) upright. Same as decals ✓.

Liquid surface disc: add slightly emissive top circle for sparkle.

Lens: 
```js
const lensGeo = new THREE.SphereGeometry(0.5, 48, 24, 0, Math.PI*2, 0, 0.62);
const lensMat = new THREE.MeshPhysicalMaterial({ color: 0xdff3ff, transparent: true, opacity: 0.35, roughness: 0.05, side: THREE.DoubleSide, depthWrite:false });
lens scaled y 0.55, positioned y such that rim ~0.45: sphere cap top at r*cos? Cap of sphere radius 0.5 thetaLength 0.62: rim circle radius sin(0.62)*0.5≈0.29, top y = 0.5. scaled y0.55 → top 0.275?? That's below liquid top 0.45. Hmm lens should sit at surface: place lens so its rim is at 0.45: rim y = 0.5*cos(0.62)*0.55 ≈ 0.224 relative to sphere center... position center y = 0.45 - 0.224 = 0.226; top of cap y = 0.226+0.275=0.501 > 0.45 pokes above liquid — looks like floating lens ✓ maybe nice. Rim radius 0.29 — small lens. Scale up: sphere radius 0.55, rim sin(0.62)*0.55≈0.32. Lens diameter ~0.64 in well radius 0.78 ✓ plausible.
```
Actually a contact lens floats edge-up sometimes; fine, subtle. depthWrite false to avoid sorting artifacts with liquid (liquid also transparent — render order: set lens renderOrder after liquid). Liquid opaque-ish? Make liquid opacity 0.55 with depthWrite true? Transparent objects: three sorts by distance; lens inside liquid could glitch. Simplify: make liquid NOT transparent? Real wells look bluish-white opaque-ish due to plastic... Liquid clearly visible adds life. Compromise: liquid transparent opacity 0.75, depthWrite: false, renderOrder 1; lens renderOrder 2, depthWrite false. Should be OK.

Hmm, or skip lens to avoid artifact risk? I'll include; artifacts minor.

Total meshes ~150, fine.

Write easing:
```js
const easeInOutCubic = t => t<.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2;
const easeOutCubic = t => 1 - Math.pow(1-t,3);
```

Park hop uses sin arc ✓.

Open/close all buttons iterate caps; also allow mixed states: openAll only triggers closed ones; closeAll only open ones.

Status update function updates spans.

Reset view: controls.reset() after saveState.

Let me write the whole file mentally-check syntax.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>隐形眼镜盒 · 3D</title>
<style>
  html,body { margin:0; height:100%; overflow:hidden; font-family: ...; }
  body { background: radial-gradient(...); }
  #app { position:fixed; inset:0; }
  .panel { position:fixed; ... glass style }
  buttons...
</style>
</head>
<body>
<div id="app"></div>
<div class="panel top-left"> title, hints, status </div>
<div class="panel bottom"> buttons </div>
<script type="importmap">...</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

... code ...
</script>
</body>
</html>
```

Full JS:

```js
const container = document.getElementById('app');
const renderer = new THREE.WebGLRenderer({ antialias:true, alpha:true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.05;
container.appendChild(renderer.domElement);

const scene = new THREE.Scene();
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;

const camera = new THREE.PerspectiveCamera(40, innerWidth/innerHeight, 0.1, 100);
camera.position.set(5.4, 4.6, 8.6);

const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 0.55, 0);
controls.enableDamping = true;
controls.dampingFactor = 0.06;
controls.minDistance = 4; controls.maxDistance = 22;
controls.maxPolarAngle = Math.PI * 0.49;
controls.update();
controls.saveState();
```

Lights:
```js
const hemi = new THREE.HemisphereLight(0xffffff, 0xb0bec5, 0.5);
const dir = new THREE.DirectionalLight(0xffffff, 1.6);
dir.position.set(5, 9, 5);
dir.castShadow = true;
dir.shadow.mapSize.set(2048, 2048);
dir.shadow.camera.left = -8; ... right 8, top 8, bottom -8, near 1, far 25;
dir.shadow.bias = -0.0004;
const fill = new THREE.DirectionalLight(0xdfefff, 0.4); fill.position.set(-6,4,-4);
scene.add(hemi, dir, fill);
```

Ground:
```js
const ground = new THREE.Mesh(new THREE.CircleGeometry(30, 64), new THREE.ShadowMaterial({ opacity: 0.22 }));
ground.rotation.x = -Math.PI/2; ground.receiveShadow = true;
```

Materials shared:
```js
const baseMat = new THREE.MeshPhysicalMaterial({ color: 0xf4f6f8, roughness: 0.3, clearcoat: 0.7, clearcoatRoughness: 0.3 });
```

Base:
```js
const WELL_X = 1.2, BASE_R = 1.32;
const shape = new THREE.Shape();
shape.absarc(WELL_X, 0, BASE_R, -Math.PI/2, Math.PI/2, false);
shape.absarc(-WELL_X, 0, BASE_R, Math.PI/2, Math.PI*1.5, false);
shape.closePath();
const baseGeo = new THREE.ExtrudeGeometry(shape, { depth: 0.5, bevelEnabled: true, bevelThickness: 0.06, bevelSize: 0.06, bevelSegments: 4, curveSegments: 64 });
baseGeo.rotateX(-Math.PI/2);
baseGeo.translate(0, 0.06, 0);
const base = new THREE.Mesh(baseGeo, baseMat);
base.castShadow = true; base.receiveShadow = true;
scene.add(base);
```
Wait: rotateX(-π/2) maps shape (x, y) → (x, ?, ?): point (x,y,0): y' = y cos(-90) - 0 = 0?? Rx(-90): y' = y*0 - z*(-1) = z; z' = y*(-1) + z*0 = -y. So (x,y,0) → (x, 0, -y) ✓ stadium in XZ with z=-y (mirrored, symmetric fine). Extrude z=depth → y ✓.

Note absarc second call: does Shape automatically lineTo between arcs? Shape.absarc adds curve; if current point differs from arc start, ShapePath connects with line? In THREE.Shape, absarc calls absarc → this.currentPoint... Path.absarc: it does lineTo start if needed? Path.absarc implementation: `this.absarc(aX, aY, aRadius, aStartAngle, aEndAngle, aClockwise)` → absellipse → CurvePath... Actually EllipseCurve added; when extracting points, if first point of curve != last point of previous, it inserts line. Yes, ShapeGeometry/Extrude handle connecting lines automatically via CurvePath.getPoints? Hmm — CurvePath.getPoints just concatenates curve points without connecting lines?? Let me recall: In Shape.getPoints → uses curves; ShapeUtils... The docs pattern for rounded rect uses absarc sequences and relies on automatic line connection. Path.absarc source:

```js
absarc( aX, aY, aRadius, aStartAngle, aEndAngle, aClockwise ) {
  this.absellipse( aX, aY, aRadius, aRadius, aStartAngle, aEndAngle, aClockwise );
  return this;
}
absellipse(...) {
  const curve = new EllipseCurve(...);
  if ( this.curves.length > 0 ) {
    const firstPoint = curve.getPoint( 0 );
    if ( ! firstPoint.equals( this.currentPoint ) ) {
      this.lineTo( firstPoint.x, firstPoint.y );
    }
  }
  this.curves.push( curve );
  ...
}
```
Yes ✓ auto lineTo. 

Wells & caps per side:

```js
const caps = [];
const liquids = [];
['L','R'].forEach((side, i) => {
  const sign = i === 0 ? -1 : 1;
  const x = sign * WELL_X;
  scene.add(createWell(x));
  const cap = createCap(side, ...);
  cap.position.set(x, CAP_CLOSED_Y, 0);
  ...
});
```

createWell(x):
```js
function createWell(x) {
  const g = new THREE.Group();
  g.position.x = x;
  const neck = new THREE.Mesh(new THREE.CylinderGeometry(0.92, 0.94, 0.46, 64), baseMat);
  neck.position.y = 0.71; neck.castShadow = true; neck.receiveShadow = true;
  g.add(neck);
  for (let i = 0; i < 3; i++) {
    const th = new THREE.Mesh(new THREE.TorusGeometry(0.925, 0.026, 12, 64), baseMat);
    th.rotation.x = Math.PI/2; th.position.y = 0.70 + i*0.09;
    g.add(th);
  }
  const innerMat = new THREE.MeshStandardMaterial({ color: 0xc4d0d8, roughness: 0.7, side: THREE.BackSide });
  const wall = new THREE.Mesh(new THREE.CylinderGeometry(0.8, 0.8, 0.82, 48, 1, true), innerMat);
  wall.position.y = 0.53;
  const bottom = new THREE.Mesh(new THREE.CircleGeometry(0.8, 48), new THREE.MeshStandardMaterial({ color: 0xc4d0d8, roughness: 0.7 }));
  bottom.rotation.x = -Math.PI/2; bottom.position.y = 0.12;
  g.add(wall, bottom);
  // liquid
  const liquid = new THREE.Mesh(new THREE.CylinderGeometry(0.78, 0.78, 0.3, 48), new THREE.MeshPhysicalMaterial({ color: 0x7cc7ea, transparent: true, opacity: 0.75, roughness: 0.08, depthWrite: false }));
  liquid.position.y = 0.30; liquid.renderOrder = 1;
  g.add(liquid);
  // lens
  const lens = new THREE.Mesh(new THREE.SphereGeometry(0.55, 48, 24, 0, Math.PI*2, 0, 0.62), new THREE.MeshPhysicalMaterial({ color: 0xeaf7ff, transparent: true, opacity: 0.35, roughness: 0.05, side: THREE.DoubleSide, depthWrite: false }));
  lens.scale.y = 0.55; lens.position.y = 0.226; lens.renderOrder = 2;
  g.add(lens);
  liquids.push(liquid);
  return g;
}
```
Wait lens position computed earlier center y 0.226 with radius .55 scaled y .55: top = 0.226+0.55*0.55=0.529? sphere top y = radius (0.55) scaled 0.55 → 0.3025; center 0.226 → top 0.528 > liquid top 0.45 pokes out 0.08 ✓. Rim y = cos(0.62)*0.55*0.55=0.246+0.226? rim relative center = 0.55*cos(0.62)*0.55 ≈ 0.246 → rim y = 0.472 ≈ at surface ✓. rim radius = 0.55*sin(0.62)=0.32 ✓.

Cap closed Y: 0.58. Cap top 1.08.

createCap(letter, cfg):
```js
function createCap(letter, color, letterColor) {
  const group = new THREE.Group();
  const mat = new THREE.MeshPhysicalMaterial({ color, roughness: 0.22, clearcoat: 0.9, clearcoatRoughness: 0.2 });
  const mats = [mat];
  const shell = new THREE.Mesh(new THREE.CylinderGeometry(1.06, 1.09, 0.5, 72, 1, true), mat);
  shell.position.y = 0.25;
  const topPlate = new THREE.Mesh(new THREE.CircleGeometry(1.06, 72), mat);
  topPlate.rotation.x = -Math.PI/2; topPlate.position.y = 0.5;
  const rim = new THREE.Mesh(new THREE.TorusGeometry(0.97, 0.075, 16, 72), mat);
  rim.rotation.x = Math.PI/2; rim.position.y = 0.5;
  group.add(shell, topPlate, rim);
  // knurls
  const knurlGeo = new THREE.BoxGeometry(0.085, 0.36, 0.13);
  for (let i = 0; i < 36; i++) {
    const a = (i / 36) * Math.PI * 2;
    const k = new THREE.Mesh(knurlGeo, mat);
    k.position.set(Math.cos(a) * 1.07, 0.24, Math.sin(a) * 1.07);
    k.rotation.y = Math.PI/2 - a;
    group.add(k);
  }
  // label
  const labelMat = new THREE.MeshStandardMaterial({ map: makeLabelTexture(letter, letterColor), transparent: true, roughness: 0.5 });
  mats.push(labelMat);
  const label = new THREE.Mesh(new THREE.CircleGeometry(0.8, 64), labelMat);
  label.rotation.x = -Math.PI/2; label.position.y = 0.578;
  group.add(label);
  // interior
  const innerMat = new THREE.MeshStandardMaterial({ color: 0x8f9aa3, roughness: 0.6, side: THREE.BackSide });
  const inner = new THREE.Mesh(new THREE.CylinderGeometry(0.94, 0.94, 0.44, 48, 1, true), innerMat);
  inner.position.y = 0.24;
  const innerTop = new THREE.Mesh(new THREE.CircleGeometry(0.94, 48), new THREE.MeshStandardMaterial({ color: 0x8f9aa3, roughness: 0.6 }));
  innerTop.rotation.x = Math.PI/2; innerTop.position.y = 0.46;
  group.add(inner, innerTop);
  group.traverse(o => { if (o.isMesh) { o.castShadow = true; } });
  return { group, mats };
}
```
innerTop rotation.x = π/2 → normal faces -y (downward) visible from below ✓.

Shadow casting for transparent label fine.

Cap data:
```js
const capConfigs = [
  { side: 'L', name: '左', x: -WELL_X, color: 0xf7f7f9, letter: '#3a6ea5', park: new THREE.Vector3(-3.7, 0, 2.0) },
  { side: 'R', name: '右', x: WELL_X, color: 0x2e7cd6, letter: '#ffffff', park: new THREE.Vector3(3.7, 0, 2.0) }
];
```
Wait L letter on white cap → blue letter ✓; R white letter on blue cap ✓.

Build caps:
```js
const CAP_CLOSED_Y = 0.58, CAP_LIFT_Y = 1.35;
const caps = capConfigs.map(cfg => {
  const { group, mats } = createCap(cfg.side, cfg.color, cfg.letter);
  group.position.set(cfg.x, CAP_CLOSED_Y, 0);
  group.userData = { isCap: true, ...cfg, state: 'closed', mats, home: new THREE.Vector3(cfg.x, CAP_CLOSED_Y, 0) };
  scene.add(group);
  return group;
});
```

Base decals:
```js
capConfigs.forEach(cfg => {
  const tex = makeDecalTexture(cfg.side, cfg.side === 'L' ? '#3a6ea5' : '#2e7cd6', cfg.name);
  const p = new THREE.Mesh(new THREE.PlaneGeometry(0.66, 0.66), new THREE.MeshStandardMaterial({ map: tex, transparent: true, roughness: 0.5, polygonOffset: true, polygonOffsetFactor: -2 }));
  p.rotation.x = -Math.PI/2;
  p.position.set(cfg.x, 0.622, 1.14);
  p.receiveShadow = true;
  scene.add(p);
});
```
Check radial: point (x=±1.2, z=1.14) → distance from own well center 1.14 < 1.26 ✓ and beyond cap radius 1.09 ✓ (1.14 > 1.09 ✓ barely; cap knurls extend to 1.07+0.065=1.135... knurl outer = 1.07 + half depth 0.065 = 1.135 < 1.14 ✓ phew, but cap bottom flare radius 1.09. OK.) Make z=1.16 for margin: 1.16<1.26 ✓. Use 1.16. Plane 0.66 wide → half 0.33: corner at radial sqrt(1.16²+0.33²)=1.206 <1.26 ✓.

makeDecalTexture: letter + small Chinese char:
```js
function makeDecalTexture(letter, color, sub) {
  const s = 256; const c = document.createElement('canvas'); c.width=c.height=s;
  const ctx = c.getContext('2d');
  ctx.fillStyle = color;
  ctx.font = `900 ${s*0.52}px Arial`;
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillText(letter, s/2, s*0.38);
  ctx.font = `700 ${s*0.16}px "PingFang SC", "Microsoft YaHei", sans-serif`;
  ctx.globalAlpha = 0.75;
  ctx.fillText(sub + '眼', s/2, s*0.78);
  ...
}
```

Tween system + animations as planned. Wobble liquid on open:
```js
function wobbleLiquid(i) {
  const lq = liquids[i];
  tween(700, t => {
    const s = 1 + Math.sin(t * Math.PI * 3) * 0.05 * (1 - t);
    lq.scale.set(s, 1, s);
  });
}
```

Toggle:
```js
async function toggleCap(cap) {
  const d = cap.userData;
  if (d.state === 'opening' || d.state === 'closing') return;
  if (d.state === 'closed') await openCap(cap); else await closeCap(cap);
  updateStatus();
}
```

openCap:
```js
async function openCap(cap) {
  const d = cap.userData;
  d.state = 'opening'; updateStatus();
  await tween(850, t => {
    cap.position.y = THREE.MathUtils.lerp(CAP_CLOSED_Y, CAP_LIFT_Y, t);
    cap.rotation.y = t * Math.PI * 4;
  });
  const start = cap.position.clone();
  const dur = 650;
  await tween(dur, t => {
    cap.position.lerpVectors(start, d.park, t);
    cap.position.y += Math.sin(t * Math.PI) * 0.5;
  }, easeInOutCubic);
  cap.position.copy(d.park);
  d.state = 'open';
  wobbleLiquid(...);
  updateStatus();
}
```
rotation.y ends 4π ≡ 0 ✓.

closeCap:
```js
async function closeCap(cap) {
  const d = cap.userData;
  d.state = 'closing'; updateStatus();
  const start = cap.position.clone();
  const above = new THREE.Vector3(d.home.x, CAP_LIFT_Y, d.home.z);
  await tween(650, t => {
    cap.position.lerpVectors(start, above, t);
    cap.position.y += Math.sin(t * Math.PI) * 0.5;
  });
  await tween(850, t => {
    cap.position.y = THREE.MathUtils.lerp(CAP_LIFT_Y, CAP_CLOSED_Y, t);
    cap.rotation.y = Math.PI * 4 * (1 - t);
  });
  cap.rotation.y = 0;
  cap.position.copy(d.home);
  d.state = 'closed'; updateStatus();
}
```

tween signature: tween(durationMs, onUpdate, ease). Implement:
```js
const activeTweens = new Set();
function tween(duration, onUpdate, ease = easeInOutCubic) {
  return new Promise(resolve => activeTweens.add({ elapsed: 0, duration, onUpdate, ease, resolve }));
}
```
Loop update with dt seconds → elapsed += dt*1000.

Raycaster & pointer:
```js
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
function pickCap(e) {
  pointer.x = (e.clientX / innerWidth) * 2 - 1;
  pointer.y = -(e.clientY / innerHeight) * 2 + 1;
  raycaster.setFromCamera(pointer, camera);
  const hits = raycaster.intersectObjects(caps, true);
  if (!hits.length) return null;
  let o = hits[0].object;
  while (o && !o.userData.isCap) o = o.parent;
  return o;
}
```

Hover highlight:
```js
let hovered = null;
renderer.domElement.addEventListener('pointermove', e => {
  const cap = pickCap(e);
  if (cap !== hovered) {
    if (hovered) setHighlight(hovered, false);
    hovered = cap;
    if (hovered) setHighlight(hovered, true);
    renderer.domElement.style.cursor = cap ? 'pointer' : 'grab';
  }
});
function setHighlight(cap, on) {
  cap.userData.mats.forEach(m => { m.emissive && m.emissive.setHex(on ? 0x2a2a2a : 0x000000); });
}
```
pointermove each frame raycast — fine.

Buttons:
```js
document.getElementById('openAll').onclick = () => caps.forEach(c => { if (c.userData.state === 'closed') openCap(c).then(updateStatus); });
```
Ensure updateStatus called inside open/close already; buttons just call toggle-like. I'll create openCapIf/closeCapIf wrappers or reuse toggleCap for mixed? "全部打开" → for closed caps call openCap. Guard state. Since toggleCap guards busy states, simpler: caps.forEach(c => c.userData.state==='closed' && toggleCap(c)).

Reset view button: controls.reset().

Status: two spans:
```js
function updateStatus() {
  caps.forEach(c => {
    const el = document.getElementById('status' + c.userData.side);
    const map = { closed: '已关闭', opening: '打开中…', open: '已打开', closing: '关闭中…' };
    el.textContent = map[c.userData.state];
  });
}
```

Render loop:
```js
const clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  const dt = clock.getDelta();
  for (const tw of [...activeTweens]) {
    tw.elapsed += dt * 1000;
    const raw = Math.min(tw.elapsed / tw.duration, 1);
    tw.onUpdate(tw.ease(raw));
    if (raw >= 1) { activeTweens.delete(tw); tw.resolve(); }
  }
  controls.update();
  renderer.render(scene, camera);
}
animate();
```

Resize:
```js
addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
```

UI HTML/CSS: frosted glass panels, Chinese text. Include title, hint, legend with colored dots for L/R, status, buttons.

Double check import map URLs:
```json
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
```
✓ RoomEnvironment path: examples/jsm/environments/RoomEnvironment.js ✓. In 0.160 RoomEnvironment constructor: `constructor()` — earlier versions took renderer param (removed after r157? It was `new RoomEnvironment()`; older needed renderer). In r160 signature: class RoomEnvironment extends Scene { constructor() } ✓.

CanvasTexture colorSpace SRGB ✓.

Potential z-fight: label at 0.578 vs rim top 0.575 (0.5+0.075). Label radius 0.8 vs rim inner radius 0.97-0.075=0.895 → label inside rim, label y 0.578 slightly above rim top — floating 0.078 above topPlate — visible edge-on? It's thin; parked upright viewed from above-ish fine. Lower to 0.505: rim curves around it, letter partially occluded by rim at grazing angles only (rim inner 0.895 > label 0.8 ✓ no occlusion from above). Set label y = 0.506, polygonOffset -1. And give label its own thin white disc? transparent texture shows topPlate beneath ✓ good.

Also add slight bevel ring on topPlate edge: shell top edge radius 1.06 at y 0.5; topPlate radius 1.06 ✓ flush.

Decal plane y 0.622 vs base top 0.62: 0.002 gap + polygonOffset ✓.

Neck top 0.94 vs cap closed interior: inner cylinder radius 0.94 ≥ neck 0.92 ✓ hides. Cap shell inner? shell is open cylinder outer surface; side material default FrontSide — from outside fine; when cap parked upright you can't see inside ✓; during flight bottom visible: inner BackSide cylinder shows interior ✓.

Ground shadow of caps during flight ✓.

Edge: OrbitControls cursor: set canvas cursor 'grab' default via CSS.

One more: `controls.saveState()` before any user interaction, after initial update ✓ (home0 saved). controls.reset() restores ✓.

Status initial call.

Now, the park hop: parked at z=2.0 in front; base front edge z max 1.32; cap radius 1.09 → parked cap back edge z = 0.91 overlaps base front? Parked x=±3.7, base extends x to ±(1.2+1.32)=±2.52. Cap at x 3.7 with radius ~1.13 → x from 2.57 ✓ no overlap.

Camera far parked caps visible ✓.

Also add gentle initial intro animation? skip.

Check `pickCap` with caps array of groups ✓ intersectObjects(caps, true).

preventOrbit on cap click? Click toggles and orbit damping may also rotate slightly — threshold 6px handles.

Touch support: pointer events cover touch ✓. Buttons for mobile ✓.

Write CSS:

```css
* { box-sizing: border-box; }
html, body { margin: 0; height: 100%; overflow: hidden; }
body {
  font-family: "PingFang SC", "Microsoft YaHei", "Segoe UI", sans-serif;
  background: radial-gradient(1200px 800px at 50% 30%, #fdfdff 0%, #dfe9f2 60%, #c7d6e4 100%);
}
#app { position: fixed; inset: 0; }
#app canvas { display: block; cursor: grab; }
#app canvas:active { cursor: grabbing; }
.panel { position: fixed; background: rgba(255,255,255,.72); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,.6); border-radius: 14px; box-shadow: 0 8px 30px rgba(50,70,90,.12); padding: 14px 18px; color: #2c3e50; }
#info { top: 18px; left: 18px; max-width: 320px; }
#info h1 { font-size: 18px; margin: 0 0 6px; }
#info p { font-size: 12.5px; margin: 4px 0; color: #5b6b7b; }
.legend { display:flex; gap:14px; margin-top:8px; font-size:13px; }
.dot { display:inline-block; width:10px; height:10px; border-radius:50%; margin-right:5px; vertical-align: middle;}
.status { ... }
#toolbar { bottom: 20px; left: 50%; transform: translateX(-50%); display: flex; gap: 10px; padding: 10px 12px; }
button { border: none; border-radius: 10px; padding: 9px 16px; font-size: 13.5px; cursor: pointer; background: #2e7cd6; color: #fff; transition: .2s; }
button:hover { background: #1f66bd; transform: translateY(-1px); }
button.secondary { background: #eef3f8; color: #2c3e50; }
button.secondary:hover { background: #dde7f0; }
```

Status HTML:
```html
<div class="legend">
  <span><i class="dot" style="background:#f0f0f4;border:1px solid #aab"></i>L（左）<b id="statusL">已关闭</b></span>
  <span><i class="dot" style="background:#2e7cd6"></i>R（右）<b id="statusR">已关闭</b></span>
</div>
```
Style b with margin-left and color.

Hint: "🖱️ 点击盒盖：拧开 / 拧上　拖动：旋转视角　滚轮：缩放"

Buttons: 全部打开 / 全部关闭 / 重置视角(secondary).

Everything good. Also mention run instructions & testing in the answer text.

Double-check tween conflict: two tweens modifying same cap concurrently can't happen due to state guard and awaited sequences ✓. wobbleLiquid tween independent ✓. Hover emissive during anim fine.

One risk: `THREE.MathUtils.lerp` exists ✓.

Liquid scale wobble pivots at cylinder center — scale x/z only ✓.

Cap label letter for R cap white on blue: texture transparent bg with white letter + ring ✓ visible on blue top ✓.

Also small detail: ring alpha 0.35 white on blue nice.

Font weight 900 Arial fine.

colorSpace for CanvasTexture: `tex.colorSpace = THREE.SRGBColorSpace;` ✓ r160.

PMREM dispose: pmrem.dispose() after use, optional — include.

Also `renderer.outputColorSpace` default is SRGBColorSpace in r152+ ✓.

ShadowMaterial ground with alpha renderer: body gradient shows ✓.

Check neck visible above base: base top 0.62; neck 0.48..0.94 (center 0.71 h 0.46) ✓ threads 0.70/0.79/0.88 ✓ all above 0.62 ✓. Cap covers 0.58..1.08 when closed → threads hidden ✓ cap bottom overlaps base top by 0.04 ✓.

Well wall from 0.12 to 0.94 (h 0.82 center 0.53) ✓ interior visible when open, above liquid ✓.

Cap lift 1.35 clears neck 0.94 + cap bottom travels from 0.58 ✓.

Camera maxPolarAngle 0.49π prevents going under ✓.

I think the code is solid. Let me also add `overflow hidden` small-screen media for panel? fine.

Add `touch-action: none` on canvas for mobile pointer events (OrbitControls sets it itself ✓ actually it does).

Final answer: provide the single HTML file code block + run instructions + test notes, in Chinese (user wrote Chinese). Keep explanation concise but include 运行方式 and 测试说明 as requested.

Let me write out the complete file carefully, watching syntax.

Also handle `makeLabelTexture` needing renderer for anisotropy — define after renderer created ✓.

Order of code:
1. renderer, scene, env, camera, controls, lights, ground
2. textures helpers
3. materials, base, wells, decals, caps
4. tween system, animations
5. interaction (raycast, events), UI
6. loop, resize

One more consideration: two tweens arrays iteration `[...activeTweens]` per frame ✓.

Guard: openCap/closeCap are async; toggleCap awaits but callers don't await — fine.

Buttons disable during animation? Not necessary due to guards; keep enabled.

Write it.

Also: initial `updateStatus()` after DOM ready — script module at end of body ✓.

For "只给我一个文件" — deliver exactly one self-contained HTML file. Provide code in one block.

Count cap meshes: shell, top, rim, 36 knurls, label, inner, innerTop = 42 per cap ×2 = 84 + wells ~ 9×2 + base + decals + ground ≈ 105 draw calls ✓ fine.

Knurls: could share geometry ✓ shared, material shared ✓.

Let me finalize numbers for park: L park (-3.7, 0, 2.0), R park (3.7, 0, 2.0).

Hmm, when both parked, they sit symmetric in front — nice.

Write the code now. Also include subtle base front text "CONTACT LENS CASE"? Extra; skip to keep clean... A tiny branding text on the bridge between caps? skip.

Edge: `pointer.y = -(e.clientY / innerHeight) * 2 + 1` use window.innerWidth/Height ✓.

For pickCap on pointermove: raycasting 100 meshes every mousemove — ok.

Done. Compose final answer with brief intro, code, 运行方式, 测试说明.