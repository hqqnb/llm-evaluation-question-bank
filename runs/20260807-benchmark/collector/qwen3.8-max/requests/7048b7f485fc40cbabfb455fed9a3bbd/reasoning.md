The user is asking for a 3D model of a contact lens case built with Three.js. Requirements:
- Prominent L and R markings
- A case lid
- Clicking the lid opens it
- HTML, CSS, JavaScript in a single file
- Directly provide a runnable deliverable, along with usage instructions and testing notes

Design considerations:
- A contact lens case typically has two wells (left and right), each with a cap/lid. In fact, a typical contact lens case has a body with two cylindrical wells, and each well has a screw cap. The caps are often marked with L and R.
- The user says "盒盖" (lid) — clicking the lid opens it. So each well has a cap, and clicking should open/close it.

Let's build a nice scene:
- A rounded rectangular body (like a pill-shaped case) with two cylindrical wells
- Two caps (cylinders with rounded tops), one marked "L" and the other "R"
- Click a cap → animation opens it (hinged rotation or unscrew + lift). Hinged rotation is more visually clear. In reality, contact lens caps are screw-type, but a hinged flip-open is more satisfying as an interaction. Unscrew animation (rotation + rising) is also possible — that's more realistic. Let's go with unscrew: the cap rotates a few times and lifts up, then tilts and floats. Actually simpler: a hinged lid like a clamshell? The user says "配有盒盖" and "点击盒盖将其打开".

Design: each cap is a hinged flip-top. Or make it a screw cap with a satisfying unscrew animation (rotation + upward motion). The unscrew is characteristic of a contact lens case. Let's go with the unscrew animation: on click, the cap rotates about 720° and lifts about 1.2 units; on re-click, it returns. Add a subtle lens inside the well visible when open — a small translucent curved disc (contact lens!) inside the well. Nice detail: the lens floats slightly with liquid shimmer.

Environment: make it lively — soft gradient background, ambient lighting, gentle auto-rotation of the scene (orbit-like), floating particles? Keep it tasteful. Add a subtle shadowed ground. Micro-interactions: hover on cap highlights it + cursor pointer + tooltip "点击打开/关闭". HUD showing state (L: 已打开 / 关闭).

Typography: distinctive display face + readable body. Since it's a Chinese request, respond in Chinese. Use Google Fonts? Single file, but can link Google Fonts (needs internet). Three.js also needs CDN. Use CDN for three.js (unpkg or cdnjs). Use ES module via importmap from unpkg/jsdelivr. Use three@0.160.0 or so from CDN via importmap.

Note: must be runnable. Use `<script type="importmap">` with jsdelivr. Provide fallback notes.

L and R markings: create with canvas texture (draw letters on canvas, apply to cap top face) — crisp and easy. Could also use TextGeometry but needs font loading; canvas texture is simpler and reliable. Draw a big "L"/"R" on cap top, colored differently (L = blue-ish, R = red-ish? Convention: L is often blue, R is red). Actually convention: R is red, L is blue. Let's do that: R cap has red accents, L cap has blue accents. Prominent letters.

Structure:
- Scene: soft studio background — CSS radial gradient behind a transparent canvas, or scene fog + gradient via a big sphere? Simpler: renderer alpha true, CSS gradient background. Add a soft shadow plane.

Lighting: hemisphere + directional with shadows + rim light.

Case body: use RoundedBoxGeometry? Not in core; can build with LatheGeometry or capsule-like. Simpler: create body from a rounded extruded shape: ExtrudeGeometry from a rounded rectangle shape (stadium shape), with bevel. Wells: cylinders sunk into body — fake with dark inner cylinders placed slightly above body top? To show the wells when cap is open: place torus rim + dark inner cylinder (the well opening) on the body top.

Plan geometry:
- Body: stadium shape (rounded rectangle with large radius = capsule top-view), extruded about 1.6 height, with bevel. Rotate so extrusion is vertical.
- Body top y ≈ 1.6.
- Wells: at x = -1.1 and +1.1, cylinders of radius 0.85, dark interior: cylinder with open top, MeshStandard material dark, slightly sunken: place a cylinder of height 0.5 so top is slightly below body top to fake depth, plus a ring rim (torus).
- Liquid inside well: slightly transparent bluish disc.
- Lens: small spherical cap (SphereGeometry with limited phi) translucent, floating above liquid when open? Keep lens always inside; visible when cap opens.
- Cap: cylinder radius 0.95, height 0.5, with slight dome top (sphere segment) + ridges on side (small boxes around, or torus knurling). Letters on top via canvas texture.
- Cap pivot: hinge at back? If screw cap, no hinge. Animation: rotate cap 2 turns while lifting 1.4 up, then tilt slightly and float with a bob. Reverse on close.

Animation: simple tween in render loop with easing function, track progress per cap. Or use a small tween helper.

Raycasting for click/hover. Hover: scale cap slightly, show tooltip label, cursor pointer, emissive glow.

Extra life:
- Slow auto-rotate of camera around scene (pause on drag). Add manual rotation via OrbitControls? Importing OrbitControls from three/addons via importmap works. Use OrbitControls with damping + autoRotate. Good.
- Background: layered CSS gradient + subtle floating bokeh dots via CSS animation? Can add soft animated gradient blobs... but I'd rather avoid a generic blurred-blob background. Use a quieter ambient background: soft vignette radial gradient, maybe faint grid or floating dust particles in the 3D scene (Points). Particles in 3D: small floating specks (like water/air) — subtle. Good.
- HUD: top-left title in display font, status chips for L/R state, hint text at bottom "点击瓶盖开合 · 拖拽旋转 · 滚轮缩放".
- Sound? Can add a small WebAudio "pop" on open/close — nice micro-feedback, no assets needed. Add a soft click/pop via oscillator. Keep it optional but nice. Add subtle sound (guarded by user gesture requirement — click is a gesture so fine).

UI design: palette — fresh "clean/hygiene" theme: aqua/teal + soft coral? Contact lens = clean, water. Let's do a deep teal background scene with soft mint lighting, caps in soft white with colored letters. Page background: gradient from #0e2430 to #14454f? With warm accents. Typography: display font "Unbounded" or "Fraunces"? For Chinese text, display Latin font only affects Latin chars; Chinese falls back to system. Use "ZCOOL KuaiLe" or "Noto Sans SC" for Chinese. Google Fonts has "Noto Sans SC", "ZCOOL QingKe HuangYou" (display-ish). Pair: "ZCOOL QingKe HuangYou" for display headings + "Noto Sans SC" for body. Distinctive and readable.

HUD layout: top-left vertical brand block "LENS CASE · 隐形眼镜盒", status chips top-right, controls hint bottom-center. Strong type contrast: big display letters.

Loading: show a small loader until three loads? Keep simple; maybe fade in.

Implementation details:

Import map:
```html
<script type="importmap">
{ "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js", "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/" } }
</script>
```
jsdelivr might be more reliable: https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js. Use jsdelivr.

Canvas texture for cap top: draw circle background (cap color), big letter, plus small text "LEFT"/"RIGHT"? Keep letter big. Also draw ring ridges.

Cap group structure for animation: capGroup at well position on body top; child mesh centered so animation lifts group.position.y and rotation.y, plus a slight tilt rotation.z at end.

Tween helper:
```js
function animate(obj, from, to, dur, ease, onUpdate, onDone)
```
Simpler: each cap has state {open:0..1 target}, progress each frame lerps toward target with easing? Use time-based: store animStart, animFrom, animTo; progress = clamp((now-animStart)/dur); eased = easeInOutCubic; value = from + (to-from)*eased. Then set:
- y = baseY + value * liftHeight
- rotY = value * Math.PI * 4 (2 turns)
- tilt: additional easing on value>0.6? Use another easing on value: tilt = smoothstep(0.6,1,value)*0.35
- bob when fully open: add sin(time)*0.05*openValue.

Hover effect: lerp scale to 1.06.

Raycast on pointermove (throttled) and click. Distinguish drag from click: record pointerdown position, on pointerup if movement < 5px treat as click. OrbitControls handles drag.

Wells: when cap is open, show lens. Lens: use LatheGeometry or SphereGeometry(r=0.55, phiLength...) — contact lens shape: shallow bowl. SphereGeometry(0.5, 32, 16, 0, Math.PI*2, 0, Math.PI*0.45) gives a dome cap; flip to make bowl. Material: MeshPhysicalMaterial with transmission? Transmission needs env; simpler: transparent, opacity 0.5, slight iridescence via color. MeshPhysicalMaterial({transmission:0.9, roughness:0.1, thickness:0.2}) might work even without env map but looks dark; add scene.environment via RoomEnvironment addon! PMREMGenerator + RoomEnvironment gives nice studio reflections — perfect for plastic. Yes: import RoomEnvironment from addons, set scene.environment. This will make plastic caps look great.

Water in wells: circular mesh with MeshPhysicalMaterial light blue, transparent, slight opacity, plus animate subtle scale/ripple? Can animate a slight normal-ish shimmer via opacity pulsing. Keep simple: gentle y bob of lens + water surface slight rotation.

Body material: MeshPhysicalMaterial clearcoat for glossy plastic. Color: soft mint/white? Body in pale aqua (#dff3f0?), caps white with colored top letter discs. Actually make cap body colored: L cap tinted pale blue, R cap tinted pale red/pink, with big letters in strong blue/red. Prominent marking ✓.

Also emboss letters on cap side? Top is enough, but also add a small ring band with L/R texture? Keep top only, big letters.

Ground: circular soft shadow — use ShadowMaterial plane + directional light casting shadow. Plus a subtle reflective disc? Keep to shadow only.

Particles: ~120 tiny points floating slowly upward, aqua color, additive blending, low opacity. Adds life.

Auto-rotate: controls.autoRotate = true, speed 0.8; pause on user interaction (controls 'start' event), resume after 3s idle.

HUD elements:
- Header: small eyebrow "INTERACTIVE 3D", big title "隐形眼镜盒" with display font, sub "点击瓶盖 · 开合盖".
- Status chips: two chips "L 左眼 — 已关闭" and "R 右眼 — 已关闭", update text + color when toggled.
- Bottom hint bar: 拖拽旋转 / 滚轮缩放 / 点击瓶盖.
- Maybe a "全部打开" button? Extra control: buttons to open both / close both. Nice addition: two small buttons. Add: "打开全部 / 关闭全部" toggle. Keep to one button that toggles both.

Sound: WebAudio small pop: on open — quick rising chirp; on close — low thud. Implement `playPop(open)` with oscillator + gain envelope.

Cursor: pointer on hover of cap.

Tooltip: floating div that follows mouse showing "打开 L 盖" etc. Nice micro-interaction. Implement a positioned div updated on pointermove when hovering cap.

Resize handler. Pixel ratio clamp 2.

Performance: shadows 1024, small scene.

Edge: fonts via Google Fonts link; if offline, fallback stack fine. Three.js requires internet — mention in usage.

Also add slight intro animation: case scales/floats in, camera eases. Simple: fade-in via CSS + case group y from -0.5 with scale 0.9→1 over 1s. Implement with same tween utility on load.

Idle motion: gentle bob of case group (sin) — but shadow should follow; subtle enough fine.

Let's write the code carefully.

Geometry details:

Stadium shape (top-view): width W=4.6, radius r=1.35 → shape via absarc on both ends:
```js
const shape = new THREE.Shape();
const halfLen = 1.15; // distance of centers of arcs from origin
const R = 1.45;
shape.absarc(-halfLen, 0, R, Math.PI/2, Math.PI*1.5, false);
shape.absarc(halfLen, 0, R, Math.PI*1.5, Math.PI/2, false); // hmm direction
```
Better: use rounded rectangle shape helper:
```js
function roundedRectShape(w, h, r){ ... standard }
```
w=4.8 (x), h=2.9 (y→z after rotation), r=1.4 (nearly stadium since h/2=1.45). Extrude depth 1.5, bevel enabled bevelSize 0.18, bevelThickness 0.18, bevelSegments 5, curveSegments 48. Then rotateX(-PI/2) so extrusion is vertical; center: extrusion is 0..depth in +z; after rotateX(-90°), z→y? rotateX(-PI/2): (x,y,z)→(x,z,-y). So depth becomes height y∈[0,1.5]. Good. Bevel adds to top and bottom. Translate y so bottom sits at 0: geometry.translate(0, 0, 0) fine, bottom at 0 minus bevel? Bevel extends beyond extrusion at both ends: with bevelThickness, geometry extends -bevelThickness..depth+bevelThickness along z before rotation → after rotation y∈[-bt, depth+bt]. Translate up by bt. Then body top y = depth + bt ≈ 1.5+0.18=1.68... let's define BODY_H = 1.5, bevel 0.16 → top ≈ 1.66.

Wells at x=±1.15, z=0, radius well 0.78. Well visual:
- Rim: TorusGeometry(0.8, 0.07) at top y, rotated flat, color slightly darker body color.
- Interior: CylinderGeometry(0.78, 0.7, 0.9, 48, 1, true) open-ended, BackSide? To see inside: use a cylinder with material side: DoubleSide, dark color; position so top is at body top y-0.02 and goes down 0.9 — but body is solid so cylinder intersects; the visible part is the opening. Since body is solid extrusion, the cylinder pokes through sides? Cylinder radius 0.78 < body, depth 0.9 < body height, fully inside except top face — but z-fighting with top face? Cut a hole in top face? Can't easily boolean without addons. Trick: place a dark disc ("well bottom" fake) slightly above body top? That would look like a dark circle, not a hole.

Better trick: place well interior as a short cylinder sticking up? No.

Alternative: build body top with actual holes via Shape.holes! ExtrudeGeometry supports holes: add two circular holes at well positions to shape. Then extrude gives real holes through body. Then insert an interior cylinder (closed at bottom) into each hole: CylinderGeometry(0.78,0.78,1.2, 48, 1, false) with open top? Cylinder has caps on both; we need open top: use CylinderGeometry(..., openEnded=true) for walls + a circle at bottom. Or just a cylinder with top cap positioned below body top so cap acts as well bottom — walls visible from above: cylinder top cap at y = topY - 0.7 → you see walls + bottom cap. But cylinder outer wall vs hole edge: hole radius 0.8, cylinder radius 0.78 — small gap sees through. Match cylinder radius to 0.8 exactly, or make hole radius 0.78 and cylinder 0.79. Use hole radius 0.8, inner cylinder radius 0.8 (flush), dark material; slight z-fighting risk at side overlap but different x? Same axis — coplanar cylinder walls may z-fight. Make inner cylinder radius 0.795 slightly smaller; gap 0.005 barely visible from above (rim torus covers seam). Add rim torus radius 0.8 tube 0.06 on top to hide seam.

So:
- Holes in shape at (±1.15, 0), radius 0.8 (in shape coordinates: shape is XY plane before rotation; after rotateX(-PI/2), shape X→x, shape Y→ -z? rotateX(-π/2): y→z? Compute: rotateX(θ): y' = y cosθ - z sinθ; z' = y sinθ + z cosθ. θ=-π/2: cos=0, sin=-1 → y' = z, z' = -y. So shape Y maps to -z. Symmetric so fine; holes at (±1.15, 0) map to (±1.15, z=0). Good.
- Inner well cylinder: radiusTop 0.8, radiusBottom 0.8, height 1.0, openEnded false, position y = topY - 1.0 + something: cylinder center at topY - 0.5 - 0.28? Want top cap at topY - 0.35 (well depth 0.35 below top surface... but body height 1.5 + bevel; well depth up to 0.9 available). Water surface at topY - 0.18, lens floats at topY - 0.12. Well bottom at topY - 0.6. Cylinder height 0.6, center at topY - 0.3. Interior color: slightly darker pale aqua, or bluish white. Add water: CircleGeometry(0.78) at topY - 0.16, MeshPhysicalMaterial color #7fd4e8, transparent 0.75 opacity, roughness 0.05 — will look like water with env reflections.

Lens: bowl shape. SphereGeometry(0.42, 32, 16, 0, 2π, 0, 0.5π*0.5?) thetaLength ~ 1.1 rad for shallow dome; then flip (scale -1?) to be concave up? Contact lens in case sits like a bowl (concave up, holding water). Sphere cap with thetaStart 0 top: dome convex up. To make bowl, use material side DoubleSide and rotate π around X? Rotating dome by π gives bowl opening up? Rotate dome 180° around X: cap points down → like an upside-down bowl (like a dome pushed down?) Actually a contact lens floating in solution can be either bowl-up or lens on bottom. Simplest visual: small translucent shallow dome (convex up) floating just above water — reads as lens. Or bowl-up: SphereGeometry cap, side:DoubleSide, looks like a bowl. Let's do bowl-up: geometry = SphereGeometry(0.42, 48, 24, 0, π*2, 0, 1.0) — cap around north pole, convex up. To make it bowl-up (concave up), rotate π around Z? Rotate cap by π around X: north pole cap → south pole cap, still convex down = looks like bowl opening up? Cap surface is curved like a bowl opening up? Cap around south pole: surface curves like a bowl opening up? Points near south pole, sphere surface — outer face is convex down; viewed from above you see inside concave. Yes: rotate X by π and you get bowl opening upward. Good, DoubleSide.

Position lens at y = topY - 0.1, animate gentle bob & slow rotation when cap open (or always, subtle).

Cap: group per well:
- Side wall: CylinderGeometry(0.92, 0.98, 0.5, 64) slight taper, colored plastic.
- Top disc: CircleGeometry(0.92) with canvas texture letter, at y=+0.25+? Add dome: SphereGeometry(0.92, ..., thetaLength ~0.35π) squashed vertically (scale y 0.35) placed on top → gentle dome; texture on dome is tricky; instead place a flat circle with letter texture slightly above dome center? Dome + flat circle looks weird.

Simpler cap: cylinder with rounded top via LatheGeometry: profile points: (0.98,0) bottom outside → (0.98,0.32) → curve to (0.6,0.5) → (0,0.55). Lathe gives smooth dome cap. Then top face is nearly flat at center — apply letter as a separate thin CircleGeometry(0.62) with canvas texture, placed at y=0.53, slightly above dome apex region... dome apex is 0.55 at r=0; circle at 0.53 intersects dome where dome y at r=0.62 is ~0.49 → circle floats 0.04 above dome at edges, visible gap from side. Alternative: apply letter texture directly to the dome via UV — lathe UVs map v along profile; texture would wrap weird.

Cleaner: cap = cylinder (radius 0.95, height 0.42) + top face uses a cylinder's cap material with letter texture via material array! CylinderGeometry material groups: [side, top cap, bottom cap]. So CylinderGeometry(0.95,0.95,0.42,64) with materials [sideMat, topMat(letter texture), bottomMat]. Then add a torus on top edge for soft rounded edge (torus radius 0.95 tube 0.05)? Torus overlaps cap edge — fine, same color. Plus knurling ridges: small boxes around side? Use torus with bumps? Simpler: add 24 thin vertical boxes around side for grip ridges — cheap and adds detail. Or ridges via side texture with stripes (canvas). Canvas side texture with vertical stripes + color: easy: create stripe canvas, map to side with repeat. Let's do that: side material with canvas texture of subtle vertical ridges (light/dark stripes) repeated around.

Letter canvas: 512x512, background cap color, big bold letter (font "900 340px 'Arial Black', sans-serif") in strong color, plus a small ring text? Add small "LEFT"/"RIGHT" text below letter? Might clutter; add small dot ring near edge. Keep: big letter + thin ring.

Colors:
- L: cap base #eaf6ff pale, letter #1668d8 blue, ring blue.
- R: cap base #ffeef0 pale, letter #e2384d red.
Body: #f2fbf9 minty white, clearcoat.
Water #8fd9ec.
Scene background: CSS radial gradient deep teal #062a33 → #0b4652, plus vignette. Particles aqua.

Directional light warm white, plus fill from opposite side cool.

HUD styling: chips with colored border-left; title uses 'ZCOOL QingKe HuangYou'. Eyebrow letterspaced. Bottom hint pill.

Buttons: "双盖开/合" button bottom-right? Place two buttons: 全部打开, 全部关闭. Style as pills with hover lift.

Tooltip div: absolute, pointer-events none, transform translate(-50%, -140%), shows cap name + action.

Status update function.

Sound: lazy init AudioContext on first click.

playPop(open): osc frequency 520→180 (close) or 220→660 (open), type 'sine' + small noise? Keep to osc with exponential ramp, gain envelope 0.001→0.2→0.001 over 0.18s. Add a second detuned osc for richness. Fine.

Click detection: on pointerdown save x,y; on pointerup: if distance<6 and hoveredCap → toggle.

Also keyboard? Not needed.

Auto rotate resume: controls.addEventListener('start', ...) set userInteracting; on 'end' timeout 2.5s → autoRotate true.

Intro: caseGroup.scale from 0.001? Use simple time-based intro in tick: introT clamp((now-start)/1200); ease out back? scale = 0.6+0.4*easeOutCubic; y offset = (1-e)*... plus CSS overlay fade.

Cap animation state:
```js
caps = [ {group, well:{x}, open:false, anim:0 (current 0..1), target:0, ...} ]
```
Each frame: anim += (target - anim) * (1 - Math.pow(0.001, dt))? Use exponential smoothing: anim = lerp(anim, target, 1 - exp(-dt*4)). Smooth and simple, no need to track start time. Then:
- eased = easeInOutCubic(anim)? Already smooth; apply directly:
- y = baseY + anim*1.5 + bob
- rotY = anim * Math.PI * 3 (1.5 turns)
- tiltZ = anim * 0.28, tiltX small
- bob = sin(t*2 + phase) * 0.06 * anim
Hover scale: cap.scale lerp to hover?1.05:1.

Emissive highlight on hover: set side material emissive to color, intensity lerp.

Shadow: caps castShadow, body castShadow, ground ShadowMaterial opacity 0.3 receiveShadow. Directional light shadow camera tight.

Environment: RoomEnvironment via PMREMGenerator:
```js
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
```
In r160 RoomEnvironment constructor takes no renderer argument (older took renderer?). In r160: `new RoomEnvironment()` — fine.

Tone mapping ACESFilmic, exposure 1.1. outputColorSpace default SRGB.

Particles: BufferGeometry with random positions in a box around scene, PointsMaterial size 0.035, color #9fe8ff, transparent 0.5, depthWrite false, additive blending. Animate: positions.y += speed*dt; wrap. Store speeds array.

Also a subtle turntable ring under case? A thin torus/disk platform: CylinderGeometry(3.4, 3.6, 0.18) dark teal glossy as pedestal — gives grounding, plus shadow on it. Pedestal + shadow on top face. Nice composition: case on round pedestal. Add pedestal with MeshStandardMaterial color #0d3a44 roughness 0.3 metalness 0.1, receiveShadow. Shadow plane maybe not needed — shadow on pedestal top. Use both: pedestal receives shadow.

Ground far below? Background gradient handles.

Camera: fov 40, pos (5.2, 4.2, 7.5), target (0, 1.1, 0). Controls min/max distance 5..14, maxPolarAngle 1.45.

Letter texture function:
```js
function makeCapTexture(letter, bg, fg, label){
 const c=document.createElement('canvas'); c.width=c.height=512; const g=c.getContext('2d');
 // bg radial
 const grad=g.createRadialGradient(256,256,40,256,256,256); grad.addColorStop(0, lighten(bg)); grad.addColorStop(1,bg);
 g.fillStyle=grad; g.fillRect(0,0,512,512);
 // ring
 g.strokeStyle=fg; g.globalAlpha=.5; g.lineWidth=10; g.beginPath(); g.arc(256,256,215,0,Math.PI*2); g.stroke(); g.globalAlpha=1;
 // letter
 g.fillStyle=fg; g.font='900 300px "Arial Black", "Noto Sans SC", sans-serif'; g.textAlign='center'; g.textBaseline='middle'; g.fillText(letter,256,272);
 // small label
 g.font='700 44px sans-serif'; g.globalAlpha=.75; g.fillText(label,256,455)? maybe skip, ring instead.
 const t=new THREE.CanvasTexture(c); t.anisotropy=8; t.colorSpace = SRGBColorSpace; return t;
}
```
Cylinder top cap UV: circle mapped to unit circle in texture — good.

Note: cylinder top cap orientation: letter may appear mirrored/rotated from some angles; top cap UV of cylinder maps x→u, z→v, so letter reads correctly when viewed from above with camera facing -z? Can't test; acceptable. To be safe rotate texture? Leave as is; standard mapping shows text upright when viewed from +z looking toward -z... camera initial position is (5.2,4.2,7.5) looking at origin: from above-front-right; letters should read approximately upright. Fine.

Side stripe texture:
```js
function makeSideTexture(base){ canvas 64x128? width 512 height 64; draw vertical stripes: for i in 0..32: alternate slightly darker/lighter every 16px; }
```
Set wrapS Repeat, repeat.set(6,1)? Cylinder side UV wraps once around; texture with 32 stripes already enough; repeat 1. Add subtle shading too. Material color multiplies with texture: make texture grayscale stripes near white, set material.color to cap color.

Cap bottom: plain material.

Also small L/R letters on body front? Embossed text near each well on body top: skip, caps carry the markings. But when cap open, well rim should indicate L/R — add small letter decals on body next to well: use small canvas sprite/plane with letter, placed on body top near outer edge of each well. Nice touch: PlaneGeometry(0.4) with transparent letter texture, rotated flat, at x=±2.15? Body half width 2.4, well at 1.15 r 0.8 → outer edge 1.95; place letter at x=±2.15, small. Hmm space tight (body edge ~2.4 with bevel). Place at ±2.12, size 0.34. OK, or place letter on front face of body. Let's place small letters on body top outside each well.

Actually cleaner: front face of body (z+ side): two small printed letters "L" "R" with colored dots. Body front is curved bevel; plane slightly off surface z = bodyHalfDepth + 0.01, y mid. Body half depth = h/2 = 1.45 + bevel... front is at z≈1.45 for mid-height. Plane at z=1.462? Bevel region: at mid-height (y ~0.75) cross-section is full shape: z max = 1.45. Place plane at z=1.455, may intersect bevel curve? At y=0.75 (middle of straight extrusion part), surface is flat at z=1.45. Good: letters at y=0.75, x=±1.15, z=1.456.

Canvas for decal texture: transparent background, letter in color + small label "左/右"? Use Chinese: "L 左" cute. Draw letter + small Chinese below. Canvas 256: big letter, below "左眼"/"右眼" small.

Now HUD copy (Chinese):
- Eyebrow: "THREE.JS · INTERACTIVE"
- Title: 隐形眼镜盒
- Sub: 点击瓶盖，开合 L / R 仓
- Chips: L 左眼 · 已关闭 / R 右眼 · 已关闭; when open: 已打开 + colored.
- Hint: 🖱 拖拽旋转 · 滚轮缩放 · 点击瓶盖开合
- Buttons: 全部打开 / 全部关闭

Loading overlay: simple center "载入 3D 场景…" fades out on first frame. Since module script from CDN may take a bit. If CDN fails, show error message: window.addEventListener('error')? Module import failure: wrap dynamic import? Simpler: static import; add <noscript> plus a timeout check: after 6s if !window.__booted show hint about needing internet. Implement small inline script timer.

CSS: fonts:
```html
<link href="https://fonts.googleapis.com/css2?family=ZCOOL+QingKe+HuangYou&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
```
Display font for title & big letters; body Noto Sans SC.

Background: layered: base linear-gradient teal depths + radial highlight top-left + subtle noise? Use conic? Keep: two radial gradients (aqua glow top, deep bottom) + faint grid? Add very subtle animated scene via CSS: slow-moving diagonal sheen? Might distract. Add vignette. Plus 3D particles provide life.

HUD micro-interactions: chip hover lift; button hover; tooltip fade; canvas cursor grab / grabbing while dragging, pointer on cap.

OK let me write the full file.

Details to be careful about:
- renderer = new THREE.WebGLRenderer({antialias:true, alpha:true}); setClearColor 0x000000,0.
- Shadows: renderer.shadowMap.enabled = true; type PCFSoft.
- dirLight position (6,9,4), castShadow, shadow.mapSize 2048, camera bounds ±6, bias -0.0005.
- Hemisphere light (skyColor #bfeaf5, ground #0a2e36, 0.5). Dir intensity ~1.2. Env intensity: r160 supports scene.environmentIntensity? That's r163+. Use default; adjust material envMapIntensity individually (physical material has envMapIntensity). Set body/cap envMapIntensity 0.7.

- OrbitControls target (0,1.05,0), enableDamping, dampingFactor 0.06, autoRotate true speed 1.0, minDistance 4.5 maxDistance 15, maxPolarAngle Math.PI*0.52? Allow slightly below horizon? Keep to 0.55π... pedestal bottom; maxPolar 1.62 rad (~93°) fine? Set to 1.55.

Cap base Y: body top y. Compute: BODY_H=1.5, BEVEL=0.16 → topY = BODY_H + BEVEL = 1.66. After translate up by BEVEL, bottom at 0. Actually extrusion with bevel: total = depth + 2*bevelThickness; after rotateX, y from -bt to depth+bt; translate(+bt): 0..depth+2bt = 1.82? Wait bevelThickness extends along extrusion axis: yes total height = 1.5 + 0.32 = 1.82, top at 1.82 after translate. Hmm define topY = BODY_H + 2*BEVEL = 1.82. Pedestal top at 0, case bottom at 0. Cap sits on top: cap bottom at topY + a bit. Cap cylinder height 0.42 centered → group y = topY + 0.21 + 0.02. Also cap radius 0.95 > hole 0.8 covers the hole. Add small neck cylinder under cap (radius 0.5 height 0.15) visible when cap open? Well rim torus covers.

Wait: cap covers hole; when open, well visible with water and lens. Water at topY - 0.16 → 0.16 below surface: visible through hole r 0.8. Lens at topY - 0.1.

Well inner cylinder: radius 0.8, height 0.9, open? Use openEnded=false; center y = topY - 0.45 → top cap at topY (flush with body top → z-fighting with body top surface? Body has hole there so no body surface at that circle — cylinder top cap at topY coincides with hole edge plane but we want to see inside, so lower it: center y = topY - 0.75, height 0.9 → top cap at topY - 0.3 (well bottom is 0.3 down? that's the top cap acting as bottom of visible well... wait cylinder top cap is at top, we look down into cylinder: we see inner walls + top cap? No — looking down into open cylinder we see the bottom cap. Cylinder has caps on both ends; top cap at topY-0.3 blocks view from above! Need open top: use openEnded=true walls + CircleGeometry bottom.

So: wellWall = CylinderGeometry(0.8, 0.8, 0.9, 64, 1, true), material DoubleSide, center y = topY - 0.45 → walls from topY-0.9 to topY. Top edge flush with body top (hole edge) — coplanar ring? Wall is vertical, no z-fight with horizontal body face except along edge circle — fine. Bottom: CircleGeometry(0.8) rotated -90°, y = topY - 0.88. Water circle r 0.79 at topY - 0.2. Lens at topY - 0.12.

Inner wall color: slightly darker pale aqua than body, or bluish white #cfe9ea; bottom a bit darker.

Rim torus: TorusGeometry(0.82, 0.055, 24, 64), rotateX(π/2), y=topY+0.01, color body-ish white — hides seam between wall and hole edge.

Cap closed y: cap bottom should sit at topY + rim top (~topY+0.06). Group base y = topY + 0.06 + 0.21 = topY + 0.27.

Cap skirt: slightly wider at bottom: radiusTop 0.9, radiusBottom 0.97.

Knurling via side texture stripes.

Cap top edge torus: radius 0.9 tube 0.045 at y=+0.21 (top edge), same color as side.

Cap group children: sideMesh (cylinder with material array), edgeTorus, letterTop handled via material array (top cap). CylinderGeometry groups: order is side(0), top(1), bottom(2). Materials: [sideMat, topMat, botMat].

Hover glow: adjust sideMat.emissive & topMat.emissive. MeshStandardMaterial emissive with map? Emissive is independent. Set emissive color to cap accent, intensity 0 → 0.25 on hover.

Raycast targets: cap side/top mesh; store cap index in userData.

Toggle function(i): caps[i].open = !open; target = open?1:0; playPop(open); updateHUD; also small particle puff? Skip.

HUD update: chip text 已打开/已关闭, chip class open.

Buttons: openAll(true/false) loops toggle without duplicate sound (play once).

Tooltip: on hover show `${cap.label} 眼 · 点击${open?'关闭':'打开'}`.

Also change cursor.

Particles: 140 count.

Intro: caseGroup scale/y anim + HUD elements CSS animation (fade/slide with delay). Use CSS keyframes for HUD entrance.

Auto-rotate resume: controls.addEventListener('start', ...) set userInteracting; on 'end' timeout 2.5s → autoRotate true.

Intro: caseGroup.scale from 0.001? Use simple time-based intro in tick: introT clamp((now-start)/1200); ease out back? scale = 0.6+0.4*easeOutCubic; y offset = (1-e)*... plus CSS overlay fade.

Cap anim state:
```js
caps = [ {group, well:{x}, open:false, anim:0 (current 0..1), target:0, ...} ]
```
Each frame: anim += (target - anim) * (1 - Math.pow(0.001, dt))? Use exponential smoothing: anim = lerp(anim, target, 1 - exp(-dt*4)). Smooth and simple, no need to track start time. Then:
- eased = easeInOutCubic(anim)? Already smooth; apply directly:
- y = baseY + anim*1.5 + bob
- rotY = anim * Math.PI * 3 (1.5 turns)
- tiltZ = anim * 0.28, tiltX small
- bob = sin(t*2 + phase) * 0.06 * anim
Hover scale: cap.scale lerp to hover?1.05:1.

Emissive highlight on hover: set side material emissive to color, intensity lerp.

Shadow: caps castShadow, body castShadow, ground ShadowMaterial opacity 0.3 receiveShadow. Directional light shadow camera tight.

Environment: RoomEnvironment via PMREMGenerator:
```js
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
```
In r160 RoomEnvironment constructor takes no renderer argument (older took renderer?). In r160: `new RoomEnvironment()` — fine.

Tone mapping ACESFilmic, exposure 1.1. outputColorSpace default SRGB.

Particles: BufferGeometry with random positions in a box around scene, PointsMaterial size 0.035, color #9fe8ff, transparent 0.5, depthWrite false, additive blending. Animate: positions.y += speed*dt; wrap. Store speeds array.

Also a subtle turntable ring under case? A thin torus/disk platform: CylinderGeometry(3.4, 3.6, 0.18) dark teal glossy as pedestal — gives grounding, plus shadow on it. Pedestal + shadow on top face. Nice composition: case on round pedestal. Add pedestal with MeshStandardMaterial color #0d3a44 roughness 0.3 metalness 0.1, receiveShadow. Shadow plane maybe not needed — shadow on pedestal top. Use both: pedestal receives shadow.

Ground far below? Background gradient handles.

Camera: fov 40, pos (5.2, 4.2, 7.5), target (0, 1.1, 0). Controls min/max distance 5..14, maxPolarAngle 1.45.

Letter texture function:
```js
function makeCapTexture(letter, bg, fg, label){
 const c=document.createElement('canvas'); c.width=c.height=512; const g=c.getContext('2d');
 // bg radial
 const grad=g.createRadialGradient(256,256,40,256,256,256); grad.addColorStop(0, lighten(bg)); grad.addColorStop(1,bg);
 g.fillStyle=grad; g.fillRect(0,0,512,512);
 // ring
 g.strokeStyle=fg; g.globalAlpha=.5; g.lineWidth=10; g.beginPath(); g.arc(256,256,215,0,Math.PI*2); g.stroke(); g.globalAlpha=1;
 // letter
 g.fillStyle=fg; g.font='900 300px "Arial Black", "Noto Sans SC", sans-serif'; g.textAlign='center'; g.textBaseline='middle'; g.fillText(letter,256,272);
 // small label
 g.font='700 44px sans-serif'; g.globalAlpha=.75; g.fillText(label,256,455)? maybe skip, ring instead.
 const t=new THREE.CanvasTexture(c); t.anisotropy=8; t.colorSpace = SRGBColorSpace; return t;
}
```
Cylinder top cap UV: circle mapped to unit circle in texture — good.

Note: cylinder top cap orientation: letter may appear mirrored/rotated from some angles; top cap UV of cylinder maps x→u, z→v, so letter reads correctly when viewed from above with camera facing -z? Can't test; acceptable. To be safe rotate texture? Leave as is; standard mapping shows text upright when viewed from +z looking toward -z... camera initial position is (5.2,4.2,7.5) looking at origin: from above-front-right; letters should read approximately upright. Fine.

Side stripe texture:
```js
function makeSideTexture(base){ canvas 64x128? width 512 height 64; draw vertical stripes: for i in 0..32: alternate slightly darker/lighter every 16px; }
```
Set wrapS Repeat, repeat.set(6,1)? Cylinder side UV wraps once around; texture with 32 stripes already enough; repeat 1. Add subtle shading too. Material color multiplies with texture: make texture grayscale stripes near white, set material.color to cap color.

Cap bottom: plain material.

Also small L/R letters on body front? Embossed text near each well on body top: skip, caps carry the markings. But when cap open, well rim should indicate L/R — add small letter decals on body next to well: use small canvas sprite/plane with letter, placed on body top near outer edge of each well. Nice touch: PlaneGeometry(0.4) with transparent letter texture, rotated flat, at x=±2.15? Body half width 2.4, well at 1.15 r 0.8 → outer edge 1.95; place letter at x=±2.15, small. Hmm space tight (body edge ~2.4 with bevel). Place at ±2.12, size 0.34. OK, or place letter on front face of body. Let's place small letters on body top outside each well.

Actually cleaner: front face of body (z+ side): two small printed letters "L" "R" with colored dots. Body front is curved bevel; plane slightly off surface z = bodyHalfDepth + 0.01, y mid. Body half depth = h/2 = 1.45 + bevel... front is at z≈1.45 for mid-height. Plane at z=1.462? Bevel region: at mid-height (y ~0.75) cross-section is full shape: z max = 1.45. Place plane at z=1.455, may intersect bevel curve? At y=0.75 (middle of straight extrusion part), surface is flat at z=1.45. Good: letters at y=0.75, x=±1.15, z=1.456.

Canvas for decal texture: transparent background, letter in color + small label "左/右"? Use Chinese: "L 左" cute. Draw letter + small Chinese below. Canvas 256: big letter, below "左眼"/"右眼" small.

Now HUD copy (Chinese):
- Eyebrow: "THREE.JS · INTERACTIVE"
- Title: 隐形眼镜盒
- Sub: 点击瓶盖，开合 L / R 仓
- Chips: L 左眼 · 已关闭 / R 右眼 · 已关闭; when open: 已打开 + colored.
- Hint: 🖱 拖拽旋转 · 滚轮缩放 · 点击瓶盖开合
- Buttons: 全部打开 / 全部关闭

Loading overlay: simple center "载入 3D 场景…" fades out on first frame. Since module script from CDN may take a bit. If CDN fails, show error message: window.addEventListener('error')? Module import failure: wrap dynamic import? Simpler: static import; add <noscript> plus a timeout check: after 6s if !window.__booted show hint about needing internet. Implement small inline script timer.

CSS: fonts:
```html
<link href="https://fonts.googleapis.com/css2?family=ZCOOL+QingKe+HuangYou&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
```
Display font for title & big letters; body Noto Sans SC.

Background: layered: base linear-gradient teal depths + radial highlight top-left + subtle noise? Use conic? Keep: two radial gradients (aqua glow top, deep bottom) + faint grid? Add very subtle animated scene via CSS: slow-moving diagonal sheen? Might distract. Add vignette. Plus 3D particles provide life.

HUD micro-interactions: chip hover lift; button hover; tooltip fade; canvas cursor grab / grabbing while dragging, pointer on cap.

OK let me write the full file.

Details to be careful about:
- renderer = new THREE.WebGLRenderer({antialias:true, alpha:true}); setClearColor 0x000000,0.
- Shadows: renderer.shadowMap.enabled = true; type PCFSoft.
- dirLight position (6,9,4), castShadow, shadow.mapSize 2048, camera bounds ±6, bias -0.0005.
- Hemisphere light (skyColor #bfeaf5, ground #0a2e36, 0.5). Dir intensity ~1.2. Env intensity: r160 supports scene.environmentIntensity? That's r163+. Use default; adjust material envMapIntensity individually (physical material has envMapIntensity). Set body/cap envMapIntensity 0.7.

- OrbitControls target (0,1.05,0), enableDamping, dampingFactor 0.06, autoRotate true speed 1.0, minDistance 4.5 maxDistance 15, maxPolarAngle Math.PI*0.52? Allow slightly below horizon? Keep to 0.55π... pedestal bottom; maxPolar 1.62 rad (~93°) fine? Set to 1.55.

Cap base Y: body top y. Compute: BODY_H=1.5, BEVEL=0.16 → topY = BODY_H + BEVEL = 1.66. After translate up by BEVEL, bottom at 0. Actually extrusion with bevel: total = depth + 2*bevelThickness; after rotateX, y from -bt to depth+bt; translate(+bt): 0..depth+2bt = 1.82? Wait bevelThickness extends along extrusion axis: yes total height = 1.5 + 0.32 = 1.82, top at 1.82 after translate. Hmm define topY = BODY_H + 2*BEVEL = 1.82. Pedestal top at 0, case bottom at 0. Cap sits on top: cap bottom at topY + a bit. Cap cylinder height 0.42 centered → group y = topY + 0.21 + 0.02. Also cap radius 0.95 > hole 0.8 covers the hole. Add small neck cylinder under cap (radius 0.5 height 0.15) visible when cap open? Well rim torus covers.

Wait: cap covers hole; when open, well visible with water and lens. Water at topY - 0.16 → 0.16 below surface: visible through hole r 0.8. Lens at topY - 0.1.

Well inner cylinder: radius 0.8, height 0.9, open? Use openEnded=false; center y = topY - 0.45 → top cap at topY (flush with body top → z-fighting with body top surface? Body has hole there so no body surface at that circle — cylinder top cap at topY coincides with hole edge plane but we want to see inside, so lower it: center y = topY - 0.75, height 0.9 → top cap at topY - 0.3 (well bottom is 0.3 down? that's the top cap acting as bottom of visible well... wait cylinder top cap is at top, we look down into cylinder: we see inner walls + top cap? No — looking down into open cylinder we see the bottom cap. Cylinder has caps on both ends; top cap at topY-0.3 blocks view from above! Need open top: use openEnded=true walls + CircleGeometry bottom.

So: wellWall = CylinderGeometry(0.8, 0.8, 0.9, 64, 1, true), material DoubleSide, center y = topY - 0.45 → walls from topY-0.9 to topY. Top edge flush with body top (hole edge) — coplanar ring? Wall is vertical, no z-fight with horizontal body face except along edge circle — fine. Bottom: CircleGeometry(0.8) rotated -90°, y = topY - 0.88. Water circle r 0.79 at topY - 0.2. Lens at topY - 0.12.

Inner wall color: slightly darker pale aqua than body, or bluish white #cfe9ea; bottom a bit darker.

Rim torus: TorusGeometry(0.82, 0.055, 24, 64), rotateX(π/2), y=topY+0.01, color body-ish white — hides seam between wall and hole edge.

Cap closed y: cap bottom should sit at topY + rim top (~topY+0.06). Group base y = topY + 0.06 + 0.21 = topY + 0.27.

Cap skirt: slightly wider at bottom: radiusTop 0.9, radiusBottom 0.97.

Knurling via side texture stripes.

Cap top edge torus: radius 0.9 tube 0.045 at y=+0.21 (top edge), same color as side.

Cap group children: sideMesh (cylinder with material array), edgeTorus, letterTop handled via material array (top cap). CylinderGeometry groups: order is side(0), top(1), bottom(2). Materials: [sideMat, topMat, botMat].

Hover glow: adjust sideMat.emissive & topMat.emissive. MeshStandardMaterial emissive with map? Emissive is independent. Set emissive color to cap accent, intensity 0 → 0.25 on hover.

Raycast targets: cap side/top mesh; store cap index in userData.

Toggle function(i): caps[i].open = !open; target = open?1:0; playPop(open); updateHUD; also small particle puff? Skip.

HUD update: chip text 已打开/已关闭, chip class open.

Buttons: openAll(true/false) loops toggle without duplicate sound (play once).

Tooltip: on hover show `${cap.label} 眼 · 点击${open?'关闭':'打开'}`.

Also change cursor.

Particles: 140 count.

Intro: caseGroup scale/y anim + HUD elements CSS animation (fade/slide with delay). Use CSS keyframes for HUD entrance.

Auto-rotate resume: controls.addEventListener('start', ...) set userInteracting; on 'end' timeout 2.5s → autoRotate true.

Sound implementation:
```js
let actx;
function pop(open){
  try{
    actx = actx || new (window.AudioContext||window.webkitAudioContext)();
    const t = actx.currentTime;
    const o = actx.createOscillator(), g = actx.createGain();
    o.type='sine';
    o.frequency.setValueAtTime(open?260:520, t);
    o.frequency.exponentialRampToValueAtTime(open?680:180, t+0.16);
    g.gain.setValueAtTime(0.0001,t);
    g.gain.exponentialRampToValueAtTime(0.18, t+0.02);
    g.gain.exponentialRampToValueAtTime(0.0001, t+0.22);
    o.connect(g).connect(actx.destination);
    o.start(t); o.stop(t+0.25);
  }catch(e){}
}
```

Raycaster hover on pointermove: compute pointer NDC, raycast against cap mesh list; set hover index; update tooltip position/content; canvas style cursor.

Click vs drag: pointerdown store; pointerup compute distance; if <6 and hover!=null, toggle.

Mobile: touch works via pointer events; hide tooltip on touch? Fine.

HUD chips: id chipL chipR, with dot.

Write CSS carefully:

:root colors: --ink #eaf7f5; --aqua #6fd6e8; --l #3f8ef0; --r #f0526a; bg gradient.

Layout: #app canvas fixed full. .hud elements fixed with pointer-events none except buttons/chips? Buttons pointer-events auto.

Title block top-left: eyebrow small letterspacing 0.3em aqua; h1 font ZCOOL 44px; sub 13px rgba.

Chips top-right: column gap 8; chip: background rgba(6,30,38,.55), backdrop-filter blur(6px) — keep blur minimal or skip; border:1px solid rgba(140,220,235,.25); border-left 3px solid var(--l/--r); padding 8px 14px; radius 10px; font 13px; state dot.

Chip open state: background tinted, text 已打开, dot glow pulse.

Bottom center hint: pill with icons (unicode). Bottom right buttons: 2 pills.

Tooltip: dark pill, small, transition opacity/transform.

Loader overlay: full, bg matching, spinner ring CSS, text "正在准备 3D 场景…"; fade out class.

Entrance animation: @keyframes rise {from opacity 0, translateY(12px)}.

Vignette overlay div, radial gradient transparent→rgba(0,10,14,.55), pointer-events none.

Also subtle top glow behind title.

Responsive: media query shrink title, stack chips.

Now let me finalize the JS module code fully, carefully checking three r160 API:
- THREE.SRGBColorSpace for texture colorSpace ✓.
- renderer.outputColorSpace default srgb ✓.
- CylinderGeometry groups for multi-material ✓ (groups: 0 side, 1 top, 2 bottom).
- ExtrudeGeometry holes: shape.holes.push(new THREE.Path); path.absarc(cx, cy, r, 0, Math.PI*2, true).
- ExtrudeGeometry({depth, bevelEnabled:true, bevelThickness, bevelSize, bevelSegments, curveSegments, steps:1}).
- Note: holes with bevel — bevel applies to hole edges too (inner bevel) — fine, gives chamfered well opening.

Bevel size 0.16 on hole radius 0.8 → hole at top surface is smaller? Bevel expands shape outward on outer contour and inward... For holes, bevel expands hole at beveled ends? Extrude bevel: at both ends, shape offsets outward for outer contour (expand) — for holes, offset shrinks hole at cap face, widens toward interior? Actually bevel offsets shape outward (bevelSize) at middle section, tapering to original at cap face. For holes this means hole is smaller in bevel region... net: hole opening at top face is radius 0.8 (original), widens inward? Hmm, well wall cylinder r 0.8 flush with opening; rim torus covers seam anyway. Fine.

Body top face at y = depth + bevelThickness (after translate). Define:
BEV_T = 0.16 (thickness along extrusion), BEV_S = 0.16 (in-plane).
topY = BEV_T + BODY_H + BEV_T? After geometry.rotateX(-Math.PI/2) then translate(0, BEV_T, 0): y range 0 .. BODY_H + 2*BEV_T = 1.82. topY = 1.82.

Well depth: wall height 0.95, from topY down: center y = topY - 0.475. Bottom circle at topY - 0.93. Water at topY - 0.22. Lens at topY - 0.13, bob amplitude 0.02.

Cap closed base Y: rim torus tube 0.055 at topY+0.01 → rim top ~topY+0.065. Cap bottom at topY+0.05. Cap cylinder height 0.44 → group y0 = topY + 0.05 + 0.22 = topY + 0.27.

Lift height 1.5.

Cap side texture: canvas 1024x128: base #fff, draw stripes: for x every 32px: darker rgba line width 10 → gives knurling. Material color tints.

Cap top texture letter as described.

Body decal texture: 256 canvas transparent; letter colored; below Chinese label; plane 0.5x0.5 on front z=1.456... wait body half depth: shape height h=2.9 → half 1.45, plus bevel expands in-plane by BEV_S in middle? Bevel expands shape by bevelSize in middle section. So at mid-height, front is at z = 1.45+0.16 = 1.61! And at top/bottom face it's 1.45. Straight extrusion part (middle) is expanded. So at y=0.91 (mid), front is at z ≈ 1.61. Place decal at z=1.617, y = topY/2 ≈ 0.91. Plane faces +z.

Hmm but expanded shape: rounded rectangle w=4.8 → half 2.4 + 0.16 = 2.56 middle. Wells at ±1.15 fine.

Pedestal: radius 3.3 top, 3.55 bottom, height 0.5, y center -0.25 → top at 0. Case bottom at 0 ✓. Add thin aqua emissive ring around pedestal top edge (TorusGeometry r 3.3 tube 0.02, emissive aqua) — subtle glow detail.

Also faint circular "caustic" texture under case? Skip.

Lights:
- hemi (0xcfeef7, 0x0c333d, 0.55)
- key dir (0xffffff, 1.6) pos (5, 8, 4), shadow
- rim dir (0x7fd8ff, 0.7) pos (-6, 4, -5), no shadow
- small point (0x9fe0ff, 0.4, dist 12) front-bottom for water sparkle? Optional, skip.

Env: RoomEnvironment; set material envMapIntensity: body 0.8, cap 0.9, water 1.2.

Water material: MeshPhysicalMaterial({color:0x8fdcef, roughness:0.08, metalness:0, transparent:true, opacity:0.8, transmission? none, clearcoat 1}). Animate slight rotation? Water circle rotation invisible; instead animate scale 1±0.005 pulsing & opacity subtle. Gentle y bob of lens + water surface slight rotation.

Lens material: MeshPhysicalMaterial({color:0xbfeaff, transparent:true, opacity:0.55, roughness:0.05, transmission:0.6? maybe heavy; use opacity approach, side:DoubleSide, clearcoat 1, ior 1.4}). Slight emissive 0x123? Fine.

Particles positions: x∈[-6,6], y∈[0,6], z∈[-6,6]; drift upward slowly + sway. Update in loop: y += sp*dt; if y>6 y=0. Sway: x += sin(t+i)*0.0005? Keep simple with per-particle phase and store in array.

Points material size 0.05, sizeAttenuation, opacity 0.55, color 0x8fd8ee, blending Additive, depthWrite false, transparent.

Auto rotate & damping.

HUD update function & buttons wired after DOM (script at end of body, module deferred anyway).

Boot flag: window.__booted = true after first render; inline script setTimeout 7000 check → if not booted, show error text in loader ("加载 Three.js 失败：请检查网络（需访问 CDN）").

Also handle WebGL unsupported: try/catch renderer creation → loader error message.

Now letter orientation on cap top: cylinder top cap UV: u = (x/r+1)/2? Actually three maps top cap with u = (x/2r)+0.5? v = (z/2r)+0.5? So texture's up (v+) corresponds to +z... camera looks from +z side, so text top faces away? Text appears upright when viewed from above with camera on -z side? Can't test; risk of letter being rotated 180 or mirrored. Mitigation: draw letter rotationally symmetric? L isn't symmetric. Alternative: set texture.rotation? CanvasTexture rotation around center: set texture.center.set(0.5,0.5); texture.rotation = Math.PI if needed — can't verify.

Let me reason precisely: three.js CylinderGeometry top cap: code builds vertices around, uv u = (cos? ) recall from source: for top cap: `uvs.push((vertex.x / radiusTop) * 0.5 + 0.5, (vertex.z / radiusTop) * 0.5 * (1) + 0.5)`? Source (r160):
```
uv.x = ( vertices[i].x / radiusTop ) * 0.5 + 0.5;
uv.y = ( vertices[i].z / radiusTop ) * 0.5 * sign? + 0.5
```
Actual code: `uvs.push( ( x / radiusTop ) * 0.5 + 0.5, ( z / radiusTop ) * 0.5 + 0.5 )` for top? Hmm top has sign=1, bottom has sign=-1, and z uses `z = sin*u? ...` I recall. Let me assume uv.y maps to +z. Texture v=0 is bottom of image (canvas y flipped: CanvasTexture flipY=true default so top of canvas = v=1). So canvas up (small y) → v=1 → +z direction? If uv.y = z/2r+0.5, then v=1 at z=+r: canvas top edge maps to +z side. Camera default looks at scene from +z: looking down at cap from +z side, +z is "down" in view (near camera/bottom of screen)... viewed from above-front, text top (canvas top, +z) faces viewer → letter reads upright when viewed from front-top.

Our initial camera at (5.2,4.2,7.5): front-top → letters should read approximately upright. Even if wrong, user can rotate. Accept.

Body front decal: PlaneGeometry faces +z by default ✓.

Edge case: OrbitControls import path 'three/addons/controls/OrbitControls.js' ✓ with importmap "three/addons/" mapping.

Use jsdelivr:
"three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
"three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"

RoomEnvironment path: examples/jsm/environments/RoomEnvironment.js ✓.

PMREM:
```js
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
pmrem.dispose();
```
r160 RoomEnvironment() takes no args ✓ (older signature accepted renderer optionally; fine).

Now cap hover scale: scale whole group; but group also has position anim; fine.

Let me define cap data:
```js
const CAPS = [
 { key:'L', x:-1.15, accent:0x2f7ff0, bg:'#eaf3ff', label:'左眼' },
 { key:'R', x: 1.15, accent:0xf0475f, bg:'#ffeef1', label:'右眼' },
];
```

Build function returns {group, mats:{side,top,torus}, state...}.

Raycast list: push capMesh (cylinder) with userData.capIndex.

Hover: also raise emissiveIntensity target.

Frame loop:
```js
function tick(now){
 requestAnimationFrame(tick);
 const dt = Math.min((now-last)/1000, 0.05); last=now; const t=now/1000;
 // intro
 // cap anim
 caps.forEach((c,i)=>{ c.anim += (c.target-c.anim)*(1-Math.exp(-dt*3.2)); ... set transforms; hover lerp; });
 // lens bob
 // water shimmer
 // particles
 controls.update();
 renderer.render(scene,camera);
}
```

Set cap transforms:
```js
const a = c.anim;
const lift = a*1.5;
c.group.position.y = c.baseY + lift + Math.sin(t*1.8+i*2)*0.05*a;
c.group.rotation.y = a*Math.PI*3 + (i? -1:1)*0.0; // spin
c.group.rotation.z = a*0.22*(i?1:-1);
c.group.rotation.x = a*0.1;
const s = 1 + c.hoverA*0.05; c.group.scale.setScalar(s);
```
hoverA lerps to hover?1:0.

Emissive: c.mats.side.emissiveIntensity = c.hoverA*0.35.

HUD tooltip follows pointer: store client x,y on pointermove; position tooltip at (x, y-14).

Update chip on toggle.

Button wiring:
```js
btnOpen.onclick = ()=> setAll(true);
btnClose.onclick = ()=> setAll(false);
function setAll(open){ let changed=false; caps.forEach(c=>{ if(c.open!==open){c.open=open; c.target=open?1:0; changed=true;} }); if(changed){ pop(open); updateHUD(); } }
```

updateHUD updates chip text/class and tooltip if visible.

Cursor: canvas.style.cursor = hover!=null ? 'pointer' : (dragging?'grabbing':'grab').

Track dragging via controls start/end.

Loader fade after first frame: in tick, once do loader.classList.add('done').

Resize.

Also prevent context menu? Not needed.

Also add double-click to focus? Skip.

Accessibility: buttons with aria-label; title semantics.

Now let me write CSS fully.

Fonts:
```
--disp:'ZCOOL QingKe HuangYou', 'Noto Sans SC', sans-serif;
--body:'Noto Sans SC', system-ui, sans-serif;
```

Background:
```css
body{
 background:
  radial-gradient(1100px 700px at 18% -10%, rgba(126,222,240,.16), transparent 60%),
  radial-gradient(900px 600px at 85% 110%, rgba(24,105,125,.35), transparent 60%),
  linear-gradient(160deg, #0a3540 0%, #07242d 55%, #051a21 100%);
}
```
Plus vignette overlay & faint grid? Add subtle dot texture via repeating gradient? Keep with gradients + particles.

HUD styling details, hover transitions, chips with dot pulse when open:
```css
.chip .dot{width:8px;height:8px;border-radius:50%;background:#57707a;transition:.3s}
.chip.open .dot{background:var(--c);box-shadow:0 0 10px var(--c);}
```
Chip has style="--c:#2f7ff0".

Buttons: pill, border 1px aqua-ish, hover translateY(-2px) + glow, active scale .97.

Hint bar: bottom center pill, small, opacity .8.

Tooltip: #tip {position:fixed; ...; transition:opacity .15s; transform:translate(-50%,-130%);}

Loader: #loader fixed, display grid place-items center; ring spinner: border animation.

Entrance: .hud-item {animation: rise .8s cubic-bezier(.2,.7,.2,1) both;} with delays.

Media query < 640px: title smaller; chips move to bottom? Keep top-right smaller; hide sub.

Also add small "L/R" legend? Chips cover.

OK, potential pitfalls: ExtrudeGeometry hole winding: shape CCW, hole CW (absarc with clockwise=true). Use `new THREE.Path(); p.absarc(x,y,r,0,Math.PI*2,true); shape.holes.push(p);` ✓.

Shadows on extruded body: castShadow true; receiveShadow true on body for self? Enable receive too.

Pedestal receiveShadow true; material MeshStandardMaterial color 0x0e3a46, roughness .35, metalness .2, envMapIntensity .6.

Ring glow: TorusGeometry(3.32, 0.018, 12, 128) rotateX(π/2), y=0.02, MeshBasicMaterial color 0x6fd6e8 transparent opacity .5. Pulse opacity in loop: 0.35+0.2*sin(t*1.5).

Water shimmer: waterMat.opacity = 0.75+0.08*Math.sin(t*2.2); lens.position.y = base + sin*0.02; lens.rotation.y slow.

Also when cap open, lens rises slightly to peek: lens y += capAnim*0.12? Cute: lens rises when cap opens. Implement: lens.position.y = lensBase + c.anim*0.1 + sin*0.02.

Each well has its own lens & water; store refs per cap: c.lens, c.water.

Cap underside: bottom cap material dark shade + small inner plug cylinder (r 0.55 h 0.1) under cap to look like seal plug — detail: add cylinder at y=-0.24, radiusTop .55 radiusBottom .5, color accent pale. Nice when lifted.

Also add thread ridges under cap? Skip.

OK let me assemble the file. Also include a small favicon? Skip. Title tag: 隐形眼镜盒 · 3D.

Write inline boot-check script before module:
```html
<script>
window.addEventListener('load',()=>{ setTimeout(()=>{ if(!window.__booted){ const l=document.getElementById('loaderText'); if(l) l.innerHTML='Three.js 加载失败…请确认网络可访问 cdn.jsdelivr.net'; } },8000); });
</script>
```

Module sets window.__booted=true after first render.

Pointer events on renderer.domElement.

Raycaster:
```js
const ray=new THREE.Raycaster(); const pNdc=new THREE.Vector2();
function pick(e){ const r=renderer.domElement.getBoundingClientRect(); pNdc.x=((e.clientX-r.left)/r.width)*2-1; pNdc.y=-((e.clientY-r.top)/r.height)*2+1; ray.setFromCamera(pNdc,camera); const hit=ray.intersectObjects(capPickMeshes,false); return hit.length? hit[0].object.userData.ci : null; }
```

pointermove: hover=pick; update tooltip text/position; cursor.
pointerdown: record downX/Y, downHover=hover.
pointerup: if distance<7: const ci=pick(e); if(ci!=null) toggle(ci).

Note: OrbitControls also listens on pointerdown; fine.

Tooltip content: `L · 左眼 — 点击打开` etc.

updateHUD():
```js
caps.forEach(c=>{ chipEl.textContent? use span state: chip.querySelector('.st').textContent = c.open? '已打开':'已关闭'; chip.classList.toggle('open', c.open); });
```

Chip markup:
```html
<div class="chip" id="chipL" style="--c:#4b93ff"><span class="dot"></span><b>L</b> 左眼 <span class="st">已关闭</span></div>
```

Buttons:
```html
<div class="actions hud-item">
 <button id="btnOpen">全部打开</button>
 <button id="btnClose">全部关闭</button>
</div>
```

Hint:
```html
<div class="hint hud-item"><span>🖱 拖拽旋转</span><i>·</i><span>⇅ 滚轮缩放</span><i>·</i><span>👆 点击瓶盖开合</span></div>
```

Header:
```html
<header class="hud-item">
 <div class="eyebrow">THREE.JS · INTERACTIVE MODEL</div>
 <h1>隐形眼镜盒</h1>
 <p class="sub">双仓护理盒 · 点击 L / R 瓶盖开合</p>
</header>
```

Big decorative letters behind? Maybe a giant faint "L·R" watermark on right side: .watermark {position:fixed; right:-20px; bottom:-30px; font-family disp; font-size:200px; color:rgba(140,220,240,.05); pointer-events:none;} adds boldness to design. Add "L/R" watermark.

OK let me write the final code, careful with syntax.

Cap top texture drawing:

```js
function capTopTexture(letter, bg, fg){
  const s=512, c=document.createElement('canvas'); c.width=c.height=s; const g=c.getContext('2d');
  const grad=g.createRadialGradient(s/2,s/2,60,s/2,s/2,s/2);
  grad.addColorStop(0,'#ffffff'); grad.addColorStop(0.55,bg); grad.addColorStop(1, shade(bg,-12));
  g.fillStyle=grad; g.fillRect(0,0,s,s);
  // outer ring
  g.strokeStyle=fg; g.lineWidth=14; g.globalAlpha=.9;
  g.beginPath(); g.arc(s/2,s/2,225,0,Math.PI*2); g.stroke();
  // tick marks
  g.globalAlpha=.5; g.lineWidth=6;
  for(let i=0;i<12;i++){ const a=i/12*Math.PI*2; g.beginPath(); g.moveTo(s/2+Math.cos(a)*196, s/2+Math.sin(a)*196); g.lineTo(s/2+Math.cos(a)*210, s/2+Math.sin(a)*210); g.stroke(); }
  g.globalAlpha=1;
  // letter with slight shadow
  g.fillStyle=fg;
  g.font='900 300px "Arial Black","Helvetica Neue",sans-serif';
  g.textAlign='center'; g.textBaseline='middle';
  g.shadowColor='rgba(0,0,0,.18)'; g.shadowBlur=18; g.shadowOffsetY=8;
  g.fillText(letter, s/2, s/2+14);
  const tex=new THREE.CanvasTexture(c); tex.colorSpace=THREE.SRGBColorSpace; tex.anisotropy=8; return tex;
}
```
shade helper: simple: precompute bg manually; skip shade function — use fixed colors: L bg '#e8f2ff', R bg '#ffeef1'. Gradient stops: '#ffffff', bg, bg. Fine, drop shade.

Side knurling texture:
```js
function knurlTexture(){
  const c=document.createElement('canvas'); c.width=1024; c.height=64; const g=c.getContext('2d');
  g.fillStyle='#ffffff'; g.fillRect(0,0,1024,64);
  for(let x=0;x<1024;x+=32){
    const gr=g.createLinearGradient(x,0,x+32,0);
    gr.addColorStop(0,'rgba(0,0,0,0)'); gr.addColorStop(.5,'rgba(0,0,0,.28)'); gr.addColorStop(1,'rgba(0,0,0,0)');
    g.fillStyle=gr; g.fillRect(x,0,32,64);
  }
  const t=new THREE.CanvasTexture(c); t.wrapS=THREE.RepeatWrapping; t.repeat.x=1; t.colorSpace=THREE.SRGBColorSpace; return t;
}
```
Shared across caps; material color tints.

Body decal texture:
```js
function decalTexture(letter, fg, cn){
  const c=document.createElement('canvas'); c.width=256;c.height=256; const g=c.getContext('2d');
  g.clearRect? default transparent.
  g.textAlign='center';
  g.fillStyle=fg;
  g.font='900 130px "Arial Black",sans-serif'; g.textBaseline='alphabetic';
  g.fillText(letter,128,130);
  g.font='700 44px "Noto Sans SC",sans-serif'; g.globalAlpha=.85;
  g.fillText(cn,128,190);
  const t=new THREE.CanvasTexture(c); t.colorSpace=THREE.SRGBColorSpace; return t;
}
```
Plane 0.55.

Font on canvas: 'Noto Sans SC' may not be loaded when texture drawn; fallback sans-serif OK. Can redraw after document.fonts.ready — trivial; use generic for Chinese decal: acceptable. Or draw Chinese after fonts.ready: implement redraw callback? Keep simple; system sans-serif renders Chinese fine.

Body material: MeshPhysicalMaterial({color:0xf4fbfa, roughness:0.32, clearcoat:1, clearcoatRoughness:0.25, envMapIntensity:0.8}). Slight sheen.

Now extrusion code:

```js
function stadiumShape(w,h,r){
  const s=new THREE.Shape();
  const x=w/2-r, y=h/2-r;
  s.absarc(x, y, r, 0, Math.PI/2, false);          // hmm simpler rounded rect path:
  ...
}
```
Standard rounded rect:
```js
function roundedRect(w,h,r){
  const s=new THREE.Shape();
  const hw=w/2, hh=h/2;
  s.moveTo(-hw+r,-hh);
  s.lineTo(hw-r,-hh);
  s.absarc(hw-r,-hh,r,-Math.PI/2,0,false);
  s.lineTo(hw,hh-r);
  s.absarc(hw-r,hh-r,r,0,Math.PI/2,false);
  s.lineTo(-hw+r,hh);
  s.absarc(-hw+r,hh-r,r,Math.PI/2,Math.PI,false);
  s.lineTo(-hw,-hh+r);
  s.absarc(-hw+r,-hh+r,r,Math.PI,Math.PI*1.5,false);
  return s;
}
```
w=4.8, h=2.9, r=1.42 (h/2=1.45 → nearly capsule).

Holes:
```js
[-1.15,1.15].forEach(x=>{ const p=new THREE.Path(); p.absarc(x,0,0.8,0,Math.PI*2,true); shape.holes.push(p); });
```

Extrude:
```js
const geo=new THREE.ExtrudeGeometry(shape,{depth:1.5,bevelEnabled:true,bevelThickness:0.16,bevelSize:0.16,bevelSegments:6,curveSegments:48,steps:1});
geo.rotateX(-Math.PI/2);
geo.translate(0,0.16,0);
```
After rotateX(-π/2): original z (0..1.5 + bevel ±0.16 → -0.16..1.66) maps to y? Point (x,y,z) → rotateX(-90°): y'=y cos(-90) - z sin(-90) = 0 + z*1? cos(-90)=0, sin(-90)=-1. y' = y*0 - z*(-1) = z. z' = y*sin(-90)+z*cos(-90) = -y. So y' = z ∈ [-0.16, 1.66]; translate +0.16 → [0, 1.82]. topY=1.82 ✓. Shape y (±1.45) → z' = -y: symmetric ✓. Hole x unchanged ✓.

topY = 1.82.

Wells at world x=±1.15, z=0 ✓ (shape y=0 → z=0).

Body mesh castShadow receiveShadow.

Now full list of cap group children:
```js
function buildCap(cfg, i){
  const g=new THREE.Group();
  const sideMat=new THREE.MeshStandardMaterial({color:cfg.capColor, map:knurlTex, roughness:0.4, metalness:0.05, envMapIntensity:0.9, emissive:new THREE.Color(cfg.accent), emissiveIntensity:0});
  const topMat=new THREE.MeshStandardMaterial({map:capTopTexture(cfg.key,cfg.bg,cfg.fg), roughness:0.35, metalness:0.05, envMapIntensity:0.8, emissive:new THREE.Color(cfg.accent), emissiveIntensity:0});
  const botMat=new THREE.MeshStandardMaterial({color:cfg.capColor, roughness:0.5});
  const cyl=new THREE.Mesh(new THREE.CylinderGeometry(0.9,0.97,0.44,64,1), [sideMat,topMat,botMat]);
  cyl.castShadow=true; cyl.userData.ci=i; g.add(cyl);
  const edge=new THREE.Mesh(new THREE.TorusGeometry(0.9,0.045,16,64), sideMat.clone()? or reuse capColor plain mat);
```
Reuse plain mat for edge: capPlain = new MeshStandardMaterial({color:cfg.capColor, roughness:.35, envMapIntensity:.9}). edge.rotation.x=Math.PI/2; edge.position.y=0.22; castShadow.
Plug: cylinder(0.55,0.5,0.14) at y=-0.26, color pale accent (#dfe9f0?) use 0xd8e6ea roughness .5.
Also colored band: torus radius .94 tube .03 at y=0.05, color accent — colored ring on cap side for L/R identification ✓. Material MeshStandardMaterial color accent roughness .3.

g.position.set(cfg.x, baseY, 0).

Pick mesh: cyl (covers top and side). Also edge torus pickable? Just cyl is fine.

baseY = topY + 0.27.

Now per-well lens & water:
```js
const water=new THREE.Mesh(new THREE.CircleGeometry(0.79,48), waterMat.clone());
water.rotation.x=-Math.PI/2; water.position.set(x, topY-0.22, 0);
const lens=new THREE.Mesh(new THREE.SphereGeometry(0.42,48,24,0,Math.PI*2,0,1.05), lensMat);
lens.rotation.x=Math.PI; // bowl up
lens.position.set(x, topY-0.13, 0);
```
Wait rotation.x=π flips bowl; but rotation also flips which side faces up; DoubleSide handles.

Lens geometry thetaLength 1.05 rad → cap half-angle 60°, radius .42 → rim radius .42*sin(1.05)=.365, depth .42*(1-cos1.05)=.21. Bowl height .21 — a bit deep; use 0.9: rim .328, depth .16. OK thetaLength .95.

Water visible through hole r .8 ✓.

Rim torus per well.

Inner wall: cylinder(0.8,0.8,0.95,64,1,true), material MeshStandardMaterial({color:0xbfdfe2, roughness:0.6, side:THREE.DoubleSide}), position y topY-0.475. Bottom circle color 0xa8cdd2, y topY-0.92.

Also add solution slight glow? Skip.

Front decal: plane at (±1.15, 0.95, 1.617). Wait mid-height y: body y 0..1.82, straight extrusion part is from bevel top of bottom (0.16..?) to 1.66; mid y≈0.91. Front z at straight part = 1.45+0.16=1.61. Decal z=1.615, y=0.91. Plane size 0.5.

Check decal x range: ±1.15 fine.

Pedestal & ring & particles & lights as planned.

Camera pos (5.4,4.6,7.6), target (0,1.15,0).

Controls: minPolar 0.15? default fine.

Intro anim: caseGroup (contains body, wells, caps, lens, water, rim, decals) scale from .72, y from -0.35, easeOutCubic over 1.1s; pedestal static. Implement:
```js
const introStart=performance.now();
in tick: const it=Math.min((now-introStart)/1100,1); const e=1-Math.pow(1-it,3); caseGroup.scale.setScalar(0.72+0.28*e); caseGroup.position.y=(1-e)*-0.4;
```
Also slight rotation.y from -0.6→0: caseGroup.rotation.y=(1-e)*-0.5.

Idle bob: caseGroup.position.y after intro = sin(t*0.9)*0.03? Combined: baseYIntro + bob*e. Fine: caseGroup.position.y = (1-e)*-0.4 + Math.sin(t*0.8)*0.03*e.

Shadow moves subtly — alive ✓.

Particles group static in scene.

Renderer clear alpha 0 to show CSS background ✓.

Fog? No (transparent bg).

Now HUD wiring & tooltip.

Also `controls.enablePan=false` to keep composition.

Write sound pop with 2 oscillators (fundamental + fifth) for pleasant sound.

Edge: AudioContext must resume on gesture — created inside click handler ✓.

Now let me write the whole file. Also include small favicon? Skip. Title tag: 隐形眼镜盒 · 3D.

Chip color: L #4b93ff, R #ff5d76.

fg colors for canvas: L '#1f6fe0', R '#e63b55'. capColor (side): L 0xeef5ff, R 0xfff0f3. Band accent: L 0x2f7ff0, R 0xf0475f.

Body decal fg: same as accent (hex string).

Water color 0x8fd9ec.

Let me write.

Also ensure pointermove doesn't raycast every frame with heavy cost — fine, small scene.

Tooltip hidden when hover null or during drag.

One more: on cap open and user drags to rotate, cap floats — good.

Also add slight landing "settle" of cap when closing: exponential smoothing gives soft landing; add small case bounce? Skip.

Final answer structure:
1. Brief description
2. Full code block
3. Usage (open file, needs internet for CDN/fonts)
4. Testing notes (interaction checklist, browser support).

Keep prose concise, in Chinese.

Let me write the code.

Also: `renderer.toneMapping = THREE.ACESFilmicToneMapping; renderer.toneMappingExposure = 1.05;`

Shadow config: dLight.shadow.camera left/right/top/bottom ±6, near 1 far 25, mapSize 2048, bias -0.0004, radius? PCFSoftShadowMap.

Check MeshStandardMaterial with map + color: color multiplies map (knurling is white-based) ✓.

Cap cylinder radial segments 64 smooth ✓.

Controls target y ~1.2.

Particles:
```js
const N=150; const pos=new Float32Array(N*3); const spd=new Float32Array(N);
for(...){ pos[i3]= (Math.random()-0.5)*13; pos[i3+1]=Math.random()*6.5; pos[i3+2]=(Math.random()-0.5)*13; spd[i]=0.12+Math.random()*0.3; }
```
Update: y+=spd*dt; if >6.5 → 0. needsUpdate.

Points material size 0.05.

OK writing.

Also guard: if WebGL context fails (renderer throws), catch and show loader error.

Let me draft the whole file in my head then output.

Potential issue: importmap + module top-level await? Not used.

Also `document.fonts.ready.then(()=>{ /* optionally redraw decals */ })` — implement decal redraw: keep reference to decal canvases and redraw + texture.needsUpdate. Nice for Chinese glyphs. Implement simply: store decal entries {tex, letter, fg, cn}; on fonts.ready redraw each and set needsUpdate. Cap letters use Arial Black (system) so fine.

Time to write the final.

CSS details:

```css
*{margin:0;padding:0;box-sizing:border-box}
html,body{height:100%}
body{font-family:var(--body); color:#e7f5f3; overflow:hidden; background:...;}
#scene{position:fixed;inset:0;display:block;cursor:grab}
#scene:active{cursor:grabbing}
.vignette{position:fixed;inset:0;pointer-events:none;background:radial-gradient(120% 90% at 50% 40%, transparent 55%, rgba(2,12,16,.55) 100%);}
header{position:fixed;top:28px;left:32px;...}
.eyebrow{font:600 11px/1 var(--body); letter-spacing:.42em; color:#7fd6e8; margin-bottom:12px}
h1{font-family:var(--disp); font-weight:400; font-size:clamp(34px,5vw,54px); letter-spacing:.04em; text-shadow:0 4px 30px rgba(0,0,0,.35)}
h1 em{font-style:normal;color:#7fd6e8} maybe color accent on "眼镜"? Keep single color + small L/R badge after title: inline spans .tag-l .tag-r small rounded colored "L" "R" — reinforces theme. Add after h1: <span class="tag l">L</span><span class="tag r">R</span>.
.sub{margin-top:10px;font-size:13px;color:rgba(214,240,240,.66);letter-spacing:.08em}
.chips{position:fixed;top:32px;right:32px;display:flex;flex-direction:column;gap:10px}
.chip{display:flex;align-items:center;gap:10px;min-width:170px;padding:10px 14px;border-radius:12px;background:rgba(7,32,40,.62);border:1px solid rgba(127,214,232,.18);font-size:13px;letter-spacing:.05em;transition:transform .25s, border-color .25s, background .25s;}
.chip:hover{transform:translateY(-2px)}
.chip b{font-family:var(--disp);font-size:20px;color:var(--c);width:16px}
.chip .dot{width:8px;height:8px;border-radius:50%;background:#3d5b64;transition:.3s}
.chip .st{margin-left:auto;color:rgba(214,240,240,.6);font-size:12px}
.chip.open{border-color:color-mix(in srgb, var(--c) 55%, transparent); background:rgba(10,40,52,.7)}
.chip.open .dot{background:var(--c);box-shadow:0 0 12px var(--c);animation:pulse 1.6s infinite}
.chip.open .st{color:var(--c)}
@keyframes pulse{50%{opacity:.5}}
.hint{position:fixed;left:50%;bottom:26px;transform:translateX(-50%);display:flex;gap:12px;align-items:center;padding:10px 20px;border-radius:999px;background:rgba(7,32,40,.6);border:1px solid rgba(127,214,232,.16);font-size:12.5px;color:rgba(214,240,240,.75);letter-spacing:.06em}
.hint i{opacity:.35;font-style:normal}
.actions{position:fixed;right:32px;bottom:26px;display:flex;gap:10px}
.actions button{font-family:var(--body);font-size:13px;font-weight:600;letter-spacing:.1em;padding:11px 20px;border-radius:999px;border:1px solid rgba(127,214,232,.35);background:rgba(10,44,54,.7);color:#dff6f6;cursor:pointer;transition:.25s}
button:hover{transform:translateY(-2px);border-color:#7fd6e8;box-shadow:0 8px 24px rgba(20,120,140,.35);background:rgba(16,64,78,.85)}
button:active{transform:translateY(0) scale(.97)}
#btnOpen{background:linear-gradient(135deg, rgba(47,127,240,.85), rgba(60,180,200,.85)); border-color:transparent} hmm gradient button — keep flat: use solid aqua-ish. Use background:#12505f.
#tip{position:fixed;left:0;top:0;pointer-events:none;opacity:0;transform:translate(-50%,-140%) scale(.94);transition:opacity .15s, transform .15s;background:rgba(6,26,33,.92);border:1px solid rgba(127,214,232,.35);padding:7px 12px;border-radius:9px;font-size:12.5px;letter-spacing:.06em;white-space:nowrap;z-index:30}
#tip.on{opacity:1;transform:translate(-50%,-140%) scale(1)}
#tip b{color:#7fd6e8}
.watermark{position:fixed;right:-10px;bottom:-46px;font-family:var(--disp);font-size:min(34vw,300px);line-height:1;color:rgba(127,214,232,.05);pointer-events:none;user-select:none;z-index:0}
#loader{position:fixed;inset:0;display:grid;place-items:center;background:linear-gradient(160deg,#0a3540,#051a21);z-index:50;transition:opacity .6s;} 
#loader.done{opacity:0;pointer-events:none}
.spinner{width:46px;height:46px;border-radius:50%;border:3px solid rgba(127,214,232,.2);border-top-color:#7fd6e8;animation:spin 1s linear infinite;margin:0 auto 16px}
@keyframes spin{to{transform:rotate(360deg)}}
#loader p{font-size:13px;letter-spacing:.2em;color:rgba(214,240,240,.7);text-align:center;line-height:1.9}
.hud-item{animation:rise .9s cubic-bezier(.22,.75,.25,1) both}
delays via style attribute or nth. Use inline style="animation-delay:.15s".
@keyframes rise{from{opacity:0;transform:translateY(14px)}}
.hint uses translateX(-50%) — conflict with rise transform! Fix: wrap hint inner? Give hint its own keyframe riseC {from{opacity:0;transform:translate(-50%,14px)} to{transform:translate(-50%,0)}}. Similarly chip hover transform vs rise 'both' end state none? animation fill both ends with keyframe end (no transform) — after animation completes, hover transitions work. During first .9s hover doesn't apply — fine.
Media query max-width 640: header left 20 top 20; chips scale .9 right 16 top auto? Chips top-right may overlap title on narrow: move chips to bottom-left? Simplest: chips {top:auto; bottom:84px; right:16px} hint smaller; hide watermark? Keep but smaller; actions bottom-right may collide with hint center — hide hint on mobile.
@media (max-width:700px){ .hint{display:none} .chips{top:auto;bottom:88px;right:16px} .actions{right:16px;bottom:24px} header{left:20px;top:20px} }
```

Title tags: `<h1>隐形眼镜盒 <span class="tag l">L</span><span class="tag r">R</span></h1>`
.tag{display:inline-grid;place-items:center;width:.62em;height:.62em;border-radius:.18em;font-family:var(--body);font-weight:800;font-size:.42em;vertical-align:.34em? adjust;margin-left:.28em;color:#fff}
.tag.l{background:#2f7ff0}.tag.r{background:#f0475f}
Cute.

Now finalize JS numbers and write.

One more check — OrbitControls autoRotate + damping needs controls.update() each frame ✓.

Cap hover pick during cap animating up: raycast still hits ✓.

Also disable tooltip during drag: track isDown.

Write pop():
```js
let AC=null;
function pop(open){
 try{
  AC=AC||new (window.AudioContext||window.webkitAudioContext)();
  if(AC.state==='suspended') AC.resume();
  const t0=AC.currentTime;
  [[open?240:520, open?660:170, .16],[open?360:780, open?990:255,.12]].forEach(([f0,f1,dur],k)=>{
    const o=AC.createOscillator(), g=AC.createGain();
    o.type=k?'triangle':'sine';
    o.frequency.setValueAtTime(f0,t0);
    o.frequency.exponentialRampToValueAtTime(f1,t0+dur);
    g.gain.setValueAtTime(0.0001,t0);
    g.gain.exponentialRampToValueAtTime(k?0.06:0.16,t0+0.015);
    g.gain.exponentialRampToValueAtTime(0.0001,t0+dur+0.08);
    o.connect(g); g.connect(AC.destination);
    o.start(t0); o.stop(t0+dur+0.1);
  });
 }catch(e){}
}
```

HUD refs & update.

OK — also "全部打开/关闭" buttons reflect? Fine.

Now let me write it all out. Ensure no stray Chinese quotes in code. Use standard quotes.

Also set `<meta name="viewport" content="width=device-width,initial-scale=1">`.

Deliver with brief usage/test section in Chinese.

For the final answer: concise Chinese description, single code block, then usage and testing notes.