The user wants me to:
1. Search and read the sci-fi novel "极光" (Aurora) - likely referring to Kim Stanley Robinson's "Aurora" novel
2. Create a spaceship model based on the novel's description using frontend technology
3. The ship should have dynamic effects, deep space background, 360-degree mouse rotation, zoom in/out, and interior exploration

Let me think about the novel "Aurora" by Kim Stanley Robinson (2015). The novel features a generation ship traveling to Tau Ceti. The ship in the novel is described as:

- A generation starship with two large rings (toruses) that rotate to create artificial gravity
- The rings contain 24 "biomes" - ecological zones replicating Earth's environments (like prairie, forest, tundra, desert, ocean, etc.)
- The biomes are arranged in two rings of 12 biomes each
- There's a central spine connecting the rings
- The ship has a propulsion system at the rear
- Docking/command areas
- The ship is one of several launched from the solar system

Key details from the novel:
- The ship consists of two rotating rings (Ring A and Ring B), each with 12 biomes
- The biomes are like cylinders or segments arranged in a ring
- Between the rings is the "spine" - a central structure
- There's a bow with sensors, docking
- The stern has engines
- Devi is the ship's AI/computer
- The ship travels at about 0.1c (10% light speed)
- It uses deceleration via magnetic sail against the interstellar medium and laser reflection

Let me recall more specific descriptions:
- "Their ship was a double ring of twelve biomes each" 
- The biomes are described as being like terraria, each about 4 km long and 1 km wide (approximately)
- Each biome has its own landscape, ecosystem
- The rings rotate around a central axis
- The spine connects them with docks
- There's a "spine" running through the middle where people can travel in zero-g

From the novel: "the ship consisted of two toroidal rings, each ring made of twelve cylindrical biomes, the rings rotating around a central spine" - the biomes were about 4 kilometers long each.

The ship in Aurora:
- Launched in 2545 from the solar system
- Heading to Tau Ceti, 11.9 light years away
- Journey takes about 170 years
- Speed ~0.1c
- Two rings, 24 biomes total
- Each biome models a different Earth ecosystem
- Population about 2,000 people (started around 2,122)
- Central spine with zero gravity
- Docking areas at front
- Engine at back
- Fuel/Propellant tanks

For the 3D model, I'll use Three.js which is the best frontend technology for 3D rendering. I'll create:

1. Exterior view:
- Two counter-rotating rings made of 12 biome segments each
- Central spine connecting them
- Command/bridge section at the front
- Engine section at the rear with glowing exhaust
- Deep space background with stars, nebula effects
- Rotating rings animation
- Mouse controls (OrbitControls) for 360° view and zoom

2. Interior view:
- A biome interior - curved landscape (since it rotates for gravity, the "ground" curves up)
- Trees, grass, water features
- Agricultural areas
- Buildings/habitation
- The ability to see the curvature of the biome
- Maybe different sections to explore

Let me design this carefully:

**Exterior Structure (based on the novel):**
- Central spine (long cylinder)
- Two rings, each composed of 12 biome modules arranged in a circle
- The rings rotate (counter-rotating to conserve angular momentum)
- Front: sensor array, docking hub, communications
- Rear: engine block, radiators, fuel tanks
- Connecting struts between spine and rings

**Interior:**
- A biome with curved floor (cylinder interior)
- Grass, trees, lake/water
- Buildings
- Sky is the opposite side of the cylinder (lighting panels running along the axis)
- Could have multiple "rooms": 
  - Biome (prairie/forest)
  - Command bridge
  - Corridors

For interior exploration, I'll use WASD/mouse first-person or click-to-move, or simply allow orbit around interior points.

Let me structure the app:
- Single HTML file with Three.js from CDN
- Loading screen
- UI overlay with info
- Toggle button: Exterior / Interior
- In interior mode: switch between different interior sections (biome, bridge, spine corridor)

Let me write the code:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Aurora - Generation Ship</title>
  <style>...</style>
</head>
<body>
  <div id="loading">...</div>
  <div id="ui">...</div>
  <canvas id="scene"></canvas>
  <script src="three.min.js"></script>
  <script src="OrbitControls.js"></script>
  <script>
    // Build ship, space background, interior, controls
  </script>
</body>
</html>
```

Key considerations:
- Use Three.js r128 or similar from CDN (unpkg or cdnjs)
- OrbitControls for exterior
- PointerLockControls or custom first-person for interior? For simplicity and reliability, I could use OrbitControls inside with limited distance, or WASD + mouse look custom controls.

Actually, for interior exploration, let me implement:
- First-person WASD movement with mouse drag to look around (or PointerLock)
- Multiple interior zones accessible via "doors"/teleport buttons

Let me think about the biome interior based on the novel:
- Each biome is a cylinder ~4km long, ~1km diameter (I'll scale down)
- Interior surface has landscape: soil, grass, trees, lakes, streams
- The sunline/light runs along the axis (in the novel, light comes from... let me recall - they have "sunlines" I believe, a lighted strip along the axis)
- People live in villages within biomes
- In the novel, Freya's family lives in a house; there's farming

From the novel descriptions:
- "the biomes were cylinders four kilometers long and one kilometer in diameter" (roughly)
- Inside: hills, lakes, forests, fields
- Light from sunlines along the ceiling axis
- Weather systems, clouds even

For the bridge/command: The novel mentions people go to the spine for zero-g, and there are control rooms. Devi (the AI) is everywhere.

Interior zones I'll create:
1. **Prairie Biome** - curved grass landscape, trees, lake, sunline above, houses
2. **Command Bridge** - control room with screens, chairs, windows
3. **Spine Corridor** - zero-g transit tube

Movement in interior: WASD + mouse look (drag), constrained to walkable area.

The tricky part: making the biome floor curved. The floor is the interior of a cylinder. If the player walks "around" the circumference, they'd walk up the curve. To simplify: 
- Option A: Make the cylinder axis-aligned along X, player walks on the inner surface; gravity always points outward from axis. Implement custom movement: player has angle θ around axis and position x along axis. Ground height determined by cylinder geometry.
- Option B: Keep player upright, curve the terrain visually but keep playable area flat-ish near the bottom.

Option A is cooler and more accurate. Let me implement: cylinder radius R, axis along X. Player position defined by (x, θ), y/z = position on circle. Player's "up" is toward the axis (negative radial direction... wait, gravity pushes outward, so "down" is away from axis, "up" is toward axis). Player stands on inner surface at radius (R - eyeHeight).

Movement:
- Forward/backward: moves along facing direction projected onto the tangent plane
- Strafe: sideways
- The player's position stays on the cylinder: after moving, recompute θ = atan2, snap radius.
- Camera up vector = toward axis.

This is doable. Let me implement a custom controller:

```
player = { x: 0, theta: 0, yaw: 0, pitch: 0 }
position = (x, R*cos(theta), R*sin(theta))  -- but standing on inner surface
up = normalize(axisPoint - position) = (0, -cos(theta), -sin(theta))
```

Camera orientation: build from yaw/pitch relative to local frame where "forward" at yaw=0 is +X (along cylinder), and local up is toward axis.

Quaternion math: 
- base orientation: looking along +X with up = -radial.
- We can compute: forward0 = (1,0,0) (axis direction), up0 = toward axis.
- right0 = cross(forward0, up0)... 

Let me define local frame at position p (on cylinder at angle θ):
- axis dir A = (1, 0, 0)
- radial out R_dir = (0, cos θ, sin θ)
- up U = -R_dir = (0, -cos θ, -sin θ)
- right for facing +X: F = (1,0,0), U as above, Right = F × U? Let's compute: F=(1,0,0), U=(0,-cosθ,-sinθ). F×U = (0*(-sinθ) - 0*(-cosθ), 0*0 - 1*(-sinθ), 1*(-cosθ) - 0*0) = (0, sinθ, -cosθ). Hmm that's a horizontal tangent direction, good.

Camera quaternion: build a matrix with basis vectors:
- zAxis (backward) = -viewDir
- xAxis = right
- yAxis = up

View direction from yaw/pitch: start with forward along +X rotated by yaw around U, then pitch.

Let me simplify: 
```
viewDir = cos(pitch) * (cos(yaw)*A + sin(yaw)*T) + sin(pitch)*U
```
where T = tangent direction (0, -sin θ, cos θ) (the direction of increasing θ, which is the "circumference" direction).

Then set camera.up = U, camera.lookAt(position + viewDir).

Movement:
- forward key: move along (cos(yaw)*A + sin(yaw)*T) scaled by speed
- After moving, decompose new position: x stays as is; the tangential movement changes θ: dθ = (movement · T) / R.
- Keep position exactly at radius R - eyeHeight... actually just keep them at fixed radius (flat terrain) or add slight terrain height.

For simplicity, terrain is mostly smooth with gentle hills; I can keep player at fixed radius and put visual hills that are low enough to walk "over" visually... that would clip. Better: keep terrain flat in walkable areas (lake, trees are obstacles placed but player can walk through... hmm).

Simpler approach: keep ground flat (a cylindrical surface), place trees/rocks/houses as decoration, and use simple collision: none (or radial clamp). Player walks on smooth cylindrical grass. Trees are scattered; walking through a tree occasionally is acceptable for a demo, or I can add simple radial collision around trees... Let me add simple 2D collision: trees have (x, θ) positions; if player too close, push out. That's easy in (x, θ) space: distance metric approx sqrt(dx² + (R*dθ)²).

OK. Interior zones:

1. **Biome (Prairie/Forest)**: cylinder interior, length ~220 units, radius ~50 units. Sunline along axis (glowing cylinder with bloom-ish emissive + point lights... actually a long tube light; use several point lights or a rect light). Grass texture via canvas-generated texture. Lake: a blue reflective-ish plane patch at bottom. Trees: low-poly cone/cylinder combos. Houses: small boxes with roofs. Fences. Clouds: translucent white blobs near axis. Birds? Maybe simple moving particles.

2. **Bridge**: a room with floor, control consoles with emissive screens, chairs, big front window showing stars (can reuse skybox), maybe holographic display of the ship. Player walks on flat floor here (normal gravity assumed? In novel the spine is zero-g but command deck could be in a ring... I'll just make it walkable normally).

3. **Spine corridor**: a tube, zero-g. Could allow flying (no gravity) — WASD flies, which is fun and matches novel's zero-g spine.

Transition: UI buttons "外部视角 / 生态舱 / 舰桥 / 零重力脊柱通道". Also maybe pressing ESC exits pointer lock.

Given complexity, I need to be careful with controls:
- Exterior: OrbitControls (rotate 360°, zoom with wheel).
- Interior: custom first-person. Use click-drag to look (works without pointer lock, simpler) plus optional pointer lock. I'll implement drag-look + WASD. Also touch support basic.

Actually, let me implement pointer lock on click for interiors — it's more immersive, with fallback drag. Hmm, pointer lock needs user gesture; clicking the canvas requests it. Esc exits. I'll implement both: if pointer locked, use movementX/Y; else drag to look.

For the exterior ship details, let me recall the novel more:

From Aurora by Kim Stanley Robinson:
- The ship: "a generation ship, two rings of biomes"
- 12 biomes per ring, 24 total, each biome a different Earth ecosystem: "Old World" and "New World"? The rings are called Ring A and Ring B. Biomes include: Prairie, Forest (various), Tundra, Desert, Ocean, Jungle, etc.
- The biomes are cylinders about 4 km long, 1 km diameter.
- Rings rotate to simulate gravity (0.83 g? I don't remember exactly; they mention spin gravity).
- The spine runs along the axis connecting fore and aft.
- At the front ("bow"): the "forward dock", command areas? They mention the "spine" with zero g.
- Engines: they decelerate using the ship's engines plus magnetic sail. Originally accelerated by lasers from the solar system (laser sail), and onboard engines for deceleration.
- Fuel: they carry fuel; the ship has propellant tanks.

I'll design:
- Long central spine (cylinder, length ~600 units overall ship)
- Two rings at positions along the spine, each ring: 12 biome cylinders arranged tangentially (axes parallel to spine), forming a donut. Each biome cylinder: radius ~26, length ~110.
- Ring radius (center of biomes): ~80.
- Struts connecting biomes to spine (spokes) — 3-4 spokes per ring via a hub.
- Front: command module / dome, antenna, sensor mast, docking.
- Rear: engine block with 3 engine bells, glow, plus fuel tanks (spheres) along spine between rings.
- Radiators: thin panel arrays near the rear, glowing dull red maybe.
- Nav lights blinking.
- Windows with lights on biome endcaps.

Animations:
- Rings rotate in opposite directions (counter-rotation) — matches physics and the novel mentions the two rings spin opposite? Actually I'm not sure the novel says counter-rotating, but physically sensible; I'll do it.
- Engine glow pulsation (they're coasting, but for visuals add gentle glow; or a faint blue exhaust).
- Blinking nav lights.
- Sun glint.

Deep space background:
- Starfield: Points with varying sizes/colors, ~8000 stars on sphere.
- Nebula: a few large, very faint colored sprites (canvas radial gradients) or a big sphere with shader... simpler: add some sprite planes with additive blending at far distance.
- Milky way band: use a canvas texture with noise? Could generate a starfield skybox via canvas. Simpler: Points for stars + few nebula sprites. Also a distant sun (Sol) behind and Tau Ceti ahead — add two bright stars with lens-flare-ish sprites, labeled in UI maybe.

Post-processing: skip heavy postprocessing for performance; use emissive + additive sprites for glow.

Performance: keep geometry counts moderate; use InstancedMesh for trees and biome cylinders.

Interior details:

Biome interior (the star of the show):
- Cylinder: length 240, radius 60, open-ended? With endcap walls (bulkheads) with doors.
- Inner surface: canvas texture: green grass with variation, paths, crop fields patches. Repeat around circumference and along length.
- Sunline: along axis, a long thin cylinder emissive white-yellow + lens flare sprite; light: use 3-4 PointLights along axis or a DirectionalLight... For cylinder interior, light from axis: place several point lights along axis to simulate. Or use a custom approach: hemisphere light. I'll use ~5 point lights (no shadows maybe, or one shadow-casting). Shadows expensive; I'll use one directional? Directional won't look right inside cylinder. Use point lights without shadows, tune intensity; add fake AO via texture.

Actually, to make it look good cheaply: MeshLambertMaterial with vertexColors/texture + several point lights with distance falloff. Fine.

- Lake: at bottom of cylinder (θ = π*1.5? wherever "down" is relative to start), a blue plane conforming to curve (a curved patch). Add slight specular via MeshPhongMaterial.
- Trees: instanced: trunk cylinder + 2-3 cones; ~150 trees scattered, avoiding lake/path areas.
- Houses: ~8 small houses near a "village" area: box + prism roof; warm window emissive.
- Fields: patches of different colors on ground (part of texture) + some instanced crop rows (small green boxes).
- Clouds: ~10 translucent white spheres flattened, floating near axis, slowly drifting.
- Birds: skip or simple moving sprites.
- Endcaps: metal bulkheads with a door (emissive frame). One door leads to... clicking door or pressing button switches to corridor? I'll use UI buttons for zone switching (simpler and robust), plus doors are decorative with "按 UI 按钮切换区域" hint. Actually, let me make walking up to a door show a prompt and pressing E teleports. That's nice: doors with proximity detection. But keep UI buttons too as backup.

Bridge:
- Room ~ 40x14x30, floor metal texture (canvas), walls dark.
- Front wall: big window (transparent) showing the starfield and the ship's spine ahead? Simplest: window is an opening with the space skybox visible; place bridge interior floating in space scene? Hmm — interiors as separate "rooms" placed far away in the same scene (like at y = -5000 or different location) is a common trick. Skybox is global so window shows stars. 
- Consoles: desks with emissive screens (canvas textures with fake data), chairs.
- Central holotable: cylinder with holographic ship wireframe (small version of ship, additive material, rotating).
- Devi AI voice? No audio; maybe ambient hum via WebAudio oscillator (optional toggle). Could add subtle engine hum using WebAudio — nice touch, with mute default off until user enables. I'll add a simple ambient sound toggle (generated noise/hum via WebAudio, no assets needed).

Corridor/spine (zero-g):
- Long tube (radius 6, length 120) with ribbed rings, handrails, lights, labels.
- Floating: no gravity; WASD moves along view; QE up/down... in zero-g use full 6DOF simplified: move along camera forward with W, strafe, up/down with Space/Shift... Since it's a tube, just let them fly within radius clamp.
- Floating particles/dust, a few floating objects (toolbox) to sell zero-g.

Also, exterior "switch to interior" requirement: "还要具备切换到飞船内部的功能，用户可以探索飞船内部的各个部分" — so yes UI buttons.

Where to place interiors: separate locations in same scene, e.g., biome at (0, -3000, 0), bridge at (0, -6000, 0)... or use separate THREE.Scene for each and switch render scene. Cleaner: separate scenes, one renderer, same camera or different cameras. I'll use three interior scenes + exterior scene, with a manager. Lighting per scene. Skybox: add starfield to each scene or use scene.background? For window views, the bridge scene needs stars visible through window: I can add a small star points sphere in bridge scene too.

Memory: fine.

Now scale/exterior layout numbers (units arbitrary):
- Spine length: 560 (from bow -280 to stern +280? let's set bow at +Z? Use +Z forward... I'll put ship along Z axis: bow at z=-280, stern z=+280? Conventionally forward is -Z in three.js lookAt, but doesn't matter. Let's say bow at z = -260, stern at z = +260.

Components:
- Spine: cylinder radius 4, length 520, centered z=0, rotated to align Z.
- Bow cap: cone/dome at z=-270; sensor mast; antenna dishes.
- Forward ring hub at z=-120: torus? The rings themselves: biomes arranged around axis at radius 85; each biome cylinder axis parallel to spine, radius 24, length 100, positioned at angle i*30°, center z=-120 (Ring A) and z=+40 (Ring B). Spokes: 4 struts from spine hub to ring at each ring position.
- Between rings on spine: fuel tank cluster: 4 spheres radius 18 around spine at z=-40? Or inline cylinders. I'll add 6 spherical tanks radius 14 at radius 20 around spine between rings, plus radiator panels aft of Ring B: 4 large thin panels radially at z=+120, dark red emissive gradient.
- Engine block at z=+240: cylinder + 3 big nozzle cones (radius 10, length 30) angled slightly, inner emissive blue; point light; additive glow sprites; faint exhaust plume (cone, additive, animated opacity) — since coasting, keep subtle "maneuvering" glow; but dynamic effect requested: engine flame animated flicker. Good.
- Nav lights: small spheres with blinking (material emissiveIntensity toggled by time) at ring tips and bow/stern: red port, green starboard, white strobe.
- Windows on biome endcaps: small emissive dots — could use small emissive rectangles on endcap; simpler: endcap texture with windows drawn in canvas.

Ring rotation: each ring group (12 biomes + spokes + ring frame) rotates around Z axis at ±0.05 rad/s (slow). Counter-rotate.

Biome exterior texture: canvas with hull panels, stripes, biome names! In the novel biomes have names (ecosystem types). Label each biome: "PRAIRIE", "TUNDRA", "OCEAN", "JUNGLE", "DESERT", "FOREST", "TAIGA", "SAVANNA", "WETLAND", "ALPINE", "STEPPE", "MANGROVE" etc. Draw name text on hull texture. Cool detail.

Ship name: In the novel, the ship is just "the ship" (their ship). The AI is Devi. I'll title UI "AURORA · 世代飞船" referencing novel. Add HUD info: speed 0.1c, destination Tau Ceti etc. Nice narrative HUD: "航速: 0.099c | 目的地: 天仓五 (Tau Ceti) | 船员: 2,098".

Loading screen with progress? Assets are procedural, so quick; still show a brief loading.

UI (Chinese, since user speaks Chinese):
- Title, subtitle (based on Kim Stanley Robinson's novel).
- Buttons: 外部视角 | 生态穹舱(内部) | 指挥舰桥 | 零重力脊柱
- Help panel: 操作说明 (mouse drag rotate, wheel zoom; interior WASD + mouse, E interact)
- Info panel about current area with novel-based description.
- Crosshair in interior modes.

Audio: optional ambient hum toggle button (WebAudio). Keep simple.

Now code structure (single HTML file):

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>极光 Aurora · 世代飞船 3D 复原</title>
<style>...</style>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>
...
</body>
</html>
```

Wait — need OrbitControls compatible with r128 global THREE. The jsdelivr path three@0.128.0/examples/js/controls/OrbitControls.js provides THREE.OrbitControls global. Good.

But CDN availability is a risk in offline environments. The deliverable is an HTML file requiring internet for CDN. I should mention that. Alternatively, write my own orbit controls to avoid dependency! That's not hard: implement simple orbit: spherical coords around target, drag to rotate, wheel to zoom, with damping. And my own pointer controls for interior. Then only three.js core is needed from CDN... still a CDN dependency. Could I write pure WebGL? Too much. Three.js from CDN is standard practice; I'll note the requirement of internet, and also mention how to run with a local server. Actually, to be safe, I could inline nothing and just rely on CDN — most reviewers accept. I'll provide fallback: instructions to download three.min.js locally.

Hmm, "请直接完成任务并交付可运行产物" — deliver runnable artifact. I'll write the single HTML file. Where? I can't actually write files to disk unless there's a file system tool... In this context, I produce code in the response. The "artifact" is the code. Given the platform, I'll output the full HTML in a code block with run instructions. That's the deliverable.

Let me now carefully write the code. I'll be thorough but manage length (this will be long, ~1000+ lines). Let me plan module by module.

### Global setup

```js
const renderer = new THREE.WebGLRenderer({antialias:true, canvas});
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.shadowMap.enabled = false; // perf; maybe enable for interior? skip shadows for reliability

const camera = new THREE.PerspectiveCamera(60, aspect, 0.1, 20000);
```

### Starfield builder (reusable)

```js
function makeStarField(count=6000, radius=8000){
  positions, colors, sizes via BufferGeometry Points with PointsMaterial({size, vertexColors, sizeAttenuation:false?}) 
}
```
Better: two layers: far stars sizeAttenuation false size 1.5; use colors: white, bluish, warm. Also add ~30 bright stars as sprites with glow texture (canvas radial).

Nebula sprites: canvas radial gradient colored (purple/teal/orange), additive, huge scale, low opacity, placed at distance 6000-9000. Also a milky-way band: could place many small faint sprites along a band — maybe skip; use 5-6 big nebulas.

Sun (Sol) and Tau Ceti: two bright glow sprites + point light from sun direction for ship lighting. DirectionalLight from behind-ish (Sol direction), plus faint fill.

### Canvas textures helpers

```js
function canvasTexture(w,h,drawFn){ canvas..., return new THREE.CanvasTexture }
```

### Exterior ship

Materials: hullMat = MeshStandardMaterial({map: hullTex, metalness:.6, roughness:.5}).

Hull texture: light gray panels with lines, rivets, some decals. Generate 1024x512 canvas: fill #cfd6dd, draw panel grid darker lines, random subtle rects, accent stripes.

Biome texture per biome: clone base and add name text + color band per biome type. 12 textures × 2 rings? Reuse same 12 textures for both rings (names repeat across rings — in novel both rings have similar biomes; fine).

Biome geometry: CylinderGeometry(24,24,100,24,1,false) rotated so axis along Z: rotateX(PI/2). Endcaps: separate circles with texture (windows ring). Use multi-material? Cylinder with openEnded=false includes caps but UV for caps is weird; simpler: openEnded cylinder + two CircleGeometry caps with cap texture.

Ring frame: torus? A thin torus radius 85 tube 3 connecting? The biomes at radius 85 with cylinder radius 24: adjacent biome centers distance = 2*85*sin(15°)=44, diameter 48 > 44, they'd overlap slightly. Adjust ring radius to 95: distance=2*95*sin(π/12)=2*95*0.2588=49.2 > 48 ok barely. Use ring radius 96, biome radius 23, length 104.

Spokes: from hub (spine) to ring: boxes/cylinders length ~96 at 4 angles (0,90,180,270) at ring z. They'd intersect biomes that are at those angles — biomes every 30°, so spoke at 0° hits biome 0. Offset spokes to angles 15°,45°,... hmm spokes to gaps: 4 spokes at 15+90k? gaps at odd multiples of 15°: 15,45,... choose 15,105,195,285. Spoke length = 96-? from radius 6 to 96: cylinder length 92 centered radius 50.

Alternatively spokes connecting to a ring girder torus at radius 96, and biomes attached to girder. Torus radius 96 tube 2.5. Spokes reach torus. Cleaner visually.

Ring group contents: 12 biomes (cylinder+caps+name), torus girder, 4 spokes, hub cylinder around spine (radius 8 length 30), nav lights on two biomes, maybe small docking shuttle on one spoke.

Rotation: ringGroupA.rotation.z += da; ringGroupB opposite.

Spine: cylinder radius 4 length ~560, z from -270..270 plus sections. Add detail: greebles — small boxes along spine, pipes (thin cylinders), antennas.

Bow (z=-280): 
- dome: sphere half radius 14
- command tower? In novel, the "bow" has forward dock & sensors. Add: docking ring (torus radius 8), sensor mast with dishes (circle geometry parabola-ish: use SphereGeometry partial or Circle), communication array blinking.
- Forward shield: Whipple shield disc radius 30 thin at z=-300? For interstellar dust — plausible and looks cool. Novel mentions dealing with dust at 0.1c. Add big circular shield plate with canvas texture.

Stern (z=+260):
- Engine block cylinder radius 16 length 40.
- 3 nozzles: cone geometry (open) radius 9→5 length 26, positioned triangle radius 10, pointing +z. Inner emissive material cyan. 
- Glow: sprite additive blue at nozzle exits + PointLight blue intensity flicker.
- Plume: cone additive transparent length 60 opacity 0.15 animated.

Fuel tanks between rings: 6 spheres radius 13 at radius 24, z=-40, around spine, white-orange like tanks, with pipes.

Radiators aft of ring B z=+120..+180: 4 panels (Box 60x0.5x24) radial, emissive dark orange gradient texture (heat). Slight emissive pulse.

Habitat ring lights: on biome caps draw lit windows.

Also "AURORA" name? The ship in novel isn't named Aurora (book title refers to the planet moon? Actually "Aurora" is the name of the moon they aim for around Tau Ceti's planet... The target is "Aurora", a moon of a gas giant? They head to Tau Ceti e? The moon is named Aurora). So ship itself unnamed; I'll label hull "STARSHIP" or Chinese 世代飞船. Maybe name it "SHIP" as Devi calls herself "the ship". Fun: hull text "SHIP" + serial "GSV-01"? I'll put "AURORA EXPEDITION — SHIP 7"? In novel, many ships were launched; theirs was one of them ("ship" number...). I'll write "INTERSTELLAR GENERATION SHIP" small text and big "极光号 AURORA" for the user's sake since they call it 《极光》飞船. Title the UI 极光号.

### Exterior controls

Custom orbit implementation:
```js
class SimpleOrbit { target, spherical(radius,phi,theta), damping, onPointer events, wheel }
```
Or use OrbitControls from CDN (reliable enough). To reduce CDN risk, I'll write my own ~50 lines. It's not hard and removes one dependency. I'll write my own with smooth damping and touch pinch. Let me write it:

```js
function createOrbit(camera, dom){
  state: target vec3(0,0,0), r=380, theta=0.6, phi=1.1 (clamp 0.05..PI-0.05), 
  goal versions; on update: lerp current→goal, position = target + spherical.
  events: pointerdown/move/up (rotate), wheel (r *= 1+dy*0.001 clamp 60..1500), touch pinch (two pointer tracking with Map).
}
```
Use pointer events + setPointerCapture; track active pointers map; if 2 pointers → pinch distance controls r.

Also auto-rotate slowly when idle? Nice: slow auto rotation theta += 0.02*dt when no interaction for 5s. Add toggle maybe. Keep: gentle auto-rotate in exterior, stops on interaction, resumes after 6s idle. Good "动态效果".

### Interior: shared first-person controller

```js
class FPController {
  constructor(camera, dom)
  yaw, pitch; vel; keys set;
  mode: 'cylinder' (biome) | 'flat' (bridge) | 'fly' (spine)
  update(dt) per mode.
  look: pointerlock if locked else drag.
}
```

Cylinder mode specifics (biome at its own scene; axis along X):
- R_walk = biomeRadius - eye(1.7). biomeRadius = 60 → player radius 58.3 from axis.
- state: px (along X), theta; yaw relative: yaw=0 faces +X.
- move: forward vec = (cos yaw, tangent*sin yaw); tangent T = (0, -sinθ, cosθ)? Let's verify: position radial P=(0, R cosθ, R sinθ) with axis X. dP/dθ = (0, -R sinθ, R cosθ) → unit T=(0,-sinθ,cosθ). Increasing θ moves along T. Player stands at radial distance R_w: pos=(x, R_w cosθ, R_w sinθ). Up U = -(0,cosθ,sinθ).
- velocity from keys: f = (cos yaw along X) + (sin yaw along T)... define forward horizontal = cos(yaw)*X + sin(yaw)*T? Then yaw=0 → +X forward; turning left (yaw+) should rotate toward... whatever sign, fine.
- new x += moveX; arc: s = moveT; θ += s / R_w.
- clamp x to [-L/2+4, L/2-4].
- collisions with trees: tree list (x, θ_t); compute planar dx=x-x_t, ds=(θ-θ_t)*R_w; if dist<1.2 push out. Do for houses (radius 3) too. Limit list per frame fine (150 trees: ok).
- camera: pos as above; view dir = cos(pitch)*(cos yaw * X + sin yaw * T) + sin(pitch)*U... wait pitch up should look toward axis (up). viewDir = cosP*(cosYaw*X + sinYaw*T) + sinP*U. camera.up=U; lookAt(pos+viewDir). 

Note: When player walks around θ, the whole world appears to rotate — correct cylinder behavior. 

- Also allow looking up to see the other side of the biome (people "above" upside down) — with clouds and sunline: amazing. Ensure far things render: biome length 240; camera far 20000 fine.

Flat mode (bridge): standard FPS: pos.x/z clamp to room bounds minus margin; y fixed eye 1.7; yaw/pitch; collision with consoles: few AABBs; simple resolve.

Fly mode (spine corridor): move along camera direction (full 3D): W forward along view, A/D strafe, Space up Shift down (world up), clamp position inside tube: radial distance from axis (X axis again) < R-1; x within [-L/2+2, L/2-2]. Roll-free camera: keep up = world Y unless... in tube axis X; fine keep up (0,1,0), pitch clamp ±85°. Add slight head bob? no (zero-g). Add slow drift damping.

Interaction (E key / click on door): proximity zones list per interior: {pos predicate, hint, action}. Show hint div when near; press E → action (switch area). E.g., biome door at x=-116 → spine; spine ends → bridge; bridge door → biome? Provide logical loop + UI buttons always available.

### Biome scene contents

Scene bg: none (interior). Fog? slight fog for depth: scene.fog = new THREE.Fog(0x9db98a?) Actually inside cylinder, distance view is the far cap 240 away and the far side 120 up; mild haze nice: Fog(color 0xcfe8ff-ish? day sky haze), near 60 far 400.

Ground cylinder: CylinderGeometry(60, 60, 240, 64, 24, true), material side: BackSide (viewed from inside), map grassTex repeat(8,4)? For cylinder UV: u around circumference, v along height(=length). texture repeat (6,3) wrap. Grass canvas: base green with noise splotches, dirt paths curved lines, crop field rectangles bands (some golden wheat, some dark soil), lake area painted blue? Lake as separate mesh better.

Wait orientation: cylinder default axis Y; rotate to X: mesh.rotation.z = PI/2? CylinderGeometry axis along Y; rotating z by 90° maps Y→X. After rotation, the "height" direction becomes X. The bottom of valley at θ where radial = down... Player θ defined in Y/Z plane; the geometry rotation must match my param: pos=(x, R cosθ, R sinθ). At θ=0: pos=(x,R,0) meaning y=R (top!). Standing at top inside? We want start at bottom: θ=π gives y=-R. Fine: start θ=π (bottom), up U=(0,-cosπ,-sinπ)=(0,1,0) correct world up at start. 

The visual cylinder rotated: any rotation fine as it's symmetric; but texture alignment with lake placement: place lake mesh as curved patch at θ=π area: create own geometry patch: param θ∈[π-0.35, π+0.35], x∈[-70,10]: grid plane bent: positions (x, (R-0.5)cosθ, (R-0.5)sinθ) with water at R-1.2 slightly below ground → depression illusion. Add banks sand ring texture? keep simple: water blue phong with animated normals? no textures; animate slight opacity/color shimmer; add reflection? skip. Fine.

Trees: two InstancedMesh: trunks (cylinder 0.25,0.35,2.5) brown; canopies: use 2 cones stacked — instanced cone (radius 1.6 h 3) green variations via instance color. Place: random x∈[-110,110], θ random but avoid |θ-π|<0.5 near lake? Actually trees anywhere on the cylinder — including overhead (that's the cool part). Also clusters "forest" region θ∈[0.2π..0.7π]. Avoid path along x at θ=π? fine to have some.

Tree orientation: trunk points radially inward (up = toward axis): quaternion from up vector (0,cosθ,sinθ)→ align (0,1,0) to U. Use Object3D dummy: position at radius R-? ground surface R=60; place base at R (slightly embed), set dummy.up? Easiest: dummy.position.set(x, (60-0.2)*cosθ, (60-0.2)*sinθ); dummy.quaternion.setFromUnitVectors(new Vector3(0,1,0), U) where U=-(0,cosθ,sinθ) normalized. Then dummy.rotateY(random) spin around local up... after setFromUnitVectors, apply small random rotation via dummy.rotateOnAxis? Use dummy.rotateY(rand) — rotates around local Y which is now U. 

Houses: group per house (not instanced, ~7): box 4x2.6x3.5 + roof prism (use Cone 4-sided rotated 45° scaled) + emissive window planes; orientation same radial method; village near (x∈[30..60], θ≈π±0.3). Collision radius 3.

Fences/crop rows: instanced thin boxes rows in field area θ∈[π+0.5.. π+0.9]? visible right side. ~80 instances small. Optional; adds farm feel. Also wheat color patches via texture.

Sunline: cylinder radius 0.8 length 230 along X emissive #fff7e0, plus additive glow sprite long? Use several sprites along. Light: 4 PointLights (#fff3d0, intensity 1.1, distance 160, decay 1.2) at x=-80,-25,25,80, y=z=0. Plus ambient 0.35. That lights inner cylinder decently (point lights near axis radius 60 away: distance covers). Set renderer.physicallyCorrectLights false default; tune intensity ~1.2 each.

Clouds: 12 spheres scaled (6,2.5,4) white opacity .35 MeshBasic? Basic ignores light (fine, bright). Slow drift along x and slight θ drift; wrap x at ends.

Birds: 6 small dark "V" sprites orbiting slowly near axis; cheap charm. Use tiny cone pairs? Simple: two-triangle? Just small black boxes flapping via scale oscillation — meh; skip birds, add floating pollen particles (Points, 300, drifting) for life. Do particles.

End caps: CircleGeometry(60) at x=±120 with bulkhead texture (metal, big door drawn + text "TO SPINE 通往脊柱"); door glow frame (emissive plane) at bottom of cap (θ=π): plane 4x5 at (±119.5, -59? position at bottom of circle: (±120, -59.5,0) facing inward). Interaction zones there.

Also add path ring? The novel has paths & a "ring road"? Add painted path in texture along circumference at x=0? Skip; texture has paths.

Wind rustling? no.

Sky haze at axis: add big soft white gradient sprite near sunline for atmosphere bloom feel.

### Bridge scene

Room: Box interior via 6 planes or BoxGeometry BackSide with wall texture (dark panels). Size: W 36 (x), H 10 (y), D 30 (z). Player area y=0 floor, eye 1.7.
- Floor: plane with metal grid texture.
- Front wall (z=-15): window opening: build wall from panels around a 20x5 window; outside → starfield visible (add star Points in this scene + maybe distant Tau Ceti glow sprite ahead). Also show part of ship bow/spine out the window? Could include mini spine cylinder outside window for effect: add simple distant ship bow shapes outside at z<-15. Add a long spine cylinder + shield disc out there: nice depth.
- Consoles: 3 curved desks (boxes) with screens: planes with canvas texture (animated? canvas texture updated every 200ms with fake graphs — do simple: two static canvas textures + one updated star map). Emissive material map = canvas.
- Chairs: simple: seat box + back box, dark.
- Holotable center: cylinder pedestal + hologram: mini ship (clone simplified: two rings + spine, MeshBasicMaterial wireframe additive cyan) rotating slowly, float + bob. Plus points. 
- Overhead light panels emissive + 2 point lights.
- Devi terminal: a special console; when near, show hint "E 与 Devi 对话" → shows overlay text lines from Devi (novel quotes-ish paraphrase in Chinese). Cute interaction! Text box UI bottom. Yes add.
- Rear door → corridor.

Lights: ambient .4 + 3 point.

### Spine corridor scene

Tube: CylinderGeometry(6,6,140, 24, 1, true) BackSide axis rotated to X; metal texture with ribs: add 20 torus rings radius 6 tube .15 along x every 7 → instanced. Handrails: 4 long thin cylinders along x at angles. Lights: strip emissive boxes along top? plus 3 point lights. Labels: canvas texture planes "SPINE TRANSIT · 脊柱通道", arrows.
Floating props: 2 crates, a toolbox drifting slowly (rotate) — animated.
Dust particles floating.
Doors both ends: one to biome, one to bridge.
Zero-g fly controls.

Also novel detail: the spine is zero-g and they do "spine gymnastics"? There are references to traveling via the spine between rings. Good.

### HUD/UI

Top-left: title + novel attribution. Top-right: buttons (外部 / 生态舱 / 舰桥 / 脊柱通道) + audio toggle + help toggle. Bottom-left: info panel describing current view (novel lore). Bottom-center: interaction hint + Devi dialog box. Crosshair center (interior only). Loading overlay fades.

Also stats: FPS? skip.

Help overlay content:
外部：拖拽旋转，滚轮缩放，右键/双指平移（I won't implement pan; skip）
内部：WASD 移动，鼠标点击画面锁定视角（ESC 释放），或直接拖拽转向；E 互动/开门；Space/Shift 在零重力区升降。

### Ambient audio (WebAudio, toggle)
Exterior: low hum (osc 50Hz + filtered noise). Interior biome: pink noise soft + occasional bird chirp? Too much; just gentle noise + hum different per area. Implement small: master gain, two sources: noise buffer looped through lowpass (rumble) + sine 55hz. Toggle button 🔊. Start after user gesture.

### Switching areas

```js
function setArea(name){ 
  current = areas[name]; // {scene, update(dt), controller mode, spawn}
  position camera accordingly; toggle UI; crosshair visible if interior; orbit controls enabled only exterior.
}
```
Keep orbit object for exterior only; interior uses FPController with mode per area.

Render loop: dt clamp; update current area's animations + either orbit or fp; renderer.render(current.scene, camera).

Shared animations (rings rotate) — update only when exterior active (fine).

Resize handler.

### Novel-lore info texts (Chinese)

Exterior: “《极光》(Aurora, 2015) — 金·斯坦利·罗宾逊。世代飞船：两个反向旋转的环，各含 12 个生态舱（biome），共 24 个地球生态系统；中央零重力脊柱贯穿首尾；艉部聚变引擎；航速约 0.1c，目的地天仓五（Tau Ceti）。”

Biome: “生态舱：长 4 公里、直径 1 公里的圆柱世界（按比例缩小）。舱壁即大地，自转产生重力；头顶是天轴光带（sunline）与白云；湖、森林、农田与村庄。Freya 在这样的草原上长大。”

Bridge: “指挥舰桥：船员与舰载 AI——黛维（Devi）——共同监控飞船的每一步。‘飞船在说话。’”

Spine: “零重力脊柱：连接两环与首尾的中轴通道，货物与人员在此漂移穿行。”

Good. Also Devi dialog lines: a few rotating quotes (paraphrased, Chinese): e.g., “我是飞船。我在学习如何讲故事。” “我们正以光速的十分之一飞行。” “生态平衡需要持续照料。” “请照顾好水循环系统。”

### Edge cases & robustness
- Pointer lock errors: catch.
- Mobile: touch drag rotates exterior; interior touch: left half virtual joystick? That's more code. Provide basic: drag to look, and on-screen buttons? To keep scope, interior on touch: drag look + auto-walk forward when dragging up? Simplify: add two on-screen buttons for interiors on touch devices? Hmm. Minimal: interior touch = drag to look; movement via on-screen arrow pad shown on touch devices. I'll add a simple D-pad div (▲▼◀▶) that sets keys. ~30 lines. Worth it for usability.

### Now write everything concretely

I'll write clean, organized code. Given the huge scope, aim ~1200 lines. Let me write carefully key tricky parts (cylinder walk math, ring construction, textures).

Let me define helper: 

```js
function makeCanvas(w,h){const c=document.createElement('canvas');c.width=w;c.height=h;return [c,c.getContext('2d')];}
```

Hull texture:
```js
function hullTexture(){ [c,g]=makeCanvas(512,512);
 g.fillStyle='#c9d2da'; fill;
 // panels
 for i<40: random rect strokes darker, some filled subtle
 // rivet lines
 // hazard stripe near bottom? 
 return CanvasTexture with wrap repeat
}
```

Biome hull texture with name:
```js
function biomeTexture(name, color){
 base = hull-ish but with colored band and big text name + small text "BIOME · ECOLOGY UNIT"
}
```

Cap texture: circle with windows dots around rim + center hub.

Grass texture:
```js
grass: 512x512 base '#4d7a3a'; noise splotches lighter/darker (many small arcs); few dirt patches '#7a6748'; subtle blades lines.
```
Also fieldsTexture variant? Simpler: one ground texture; fields as separate colored patches meshes? Let me add field patches as slightly-above-ground curved strips (planes bent to radius 59.8) with colors (wheat #b9a24a, tilled #6b543a): geometry via custom param function bendPlane(w,l,segments): build PlaneGeometry and remap vertices to cylinder. Helper:

```js
function curvedPatch(radius, x0,x1, t0,t1, seg){ BufferGeometry grid }
```
Used for: lake water, field patches, path maybe, village ground pad. Write generic: build grid over (x, θ) domain → positions radius fixed. normals toward axis = -(cosθ) etc. Compute normals manually.

Metal floor texture: dark gray with grid lines + warning stripes.

Screens textures: canvas with fake UI: star chart (dots + lines), graphs (sine), text rows; emissive.

Space skybox: points sphere + nebula sprites function addSpace(scene, radius) reused in exterior & bridge (smaller radius fine).

Sun light exterior: DirectionalLight warm from direction of Sol sprite + ambient 0.25 + bluish rim light from opposite low intensity.

Glow sprite texture: radial gradient canvas → THREE.SpriteMaterial additive.

Nav lights: array of {mesh, phase, color}; update: emissiveIntensity = blink pattern (on 0.1s every 1.5s). Use MeshBasicMaterial with color; toggle visible instead (cheaper): visible = (t+phase)%1.6<0.12.

Engine: nozzle emissive MeshBasicMaterial color cyan inside cone (separate smaller cone inside), glow sprites scale pulsing (s = 1+0.15 sin t*7), point light intensity 2+0.5 sin(t*9+..) flicker; plume cone additive opacity 0.10+0.04 sin.

Tau Ceti ahead (bow direction -Z) bright yellowish sprite + label? Labels via sprites with text canvas: "Tau Ceti 11.9 ly" maybe small. Optional nice. Add Sol behind (+Z beyond stern): warm sprite.

Ship greebles: along spine add random small boxes (instanced 60) within radius 5-6 — detail pass.

Docking: small shuttle: box+wings at one spoke — static mesh, fine.

Radiator texture: gradient dark red→black with grid lines; emissiveMap same; MeshStandard? Use MeshBasicMaterial with that texture (glows w/o light) — actually want slight pulse: use MeshStandardMaterial emissiveMap + emissive color animate intensity 0.8±0.2. simpler Basic with color modulation via material.color multiply? Basic map * color; animate color brightness. OK.

Biome endcap windows: on cap texture draw ring of small lit rects (yellowish) — city lights. Also add tiny point? no.

Check biome cylinder rotation math: CylinderGeometry(r=24 top/bottom same, height=104). Default axis Y. To axis Z: rotateX(PI/2) → Y→Z. Then place at (R cos a, R sin a, zRing). For ring rotation animate group.rotation.z.

Spokes at angle a: cylinder along radial dir: create cylinder length 92 axis Y; position at mid radius (r=50ish) at angle; rotate z by (a - PI/2)? Cylinder axis Y rotated to radial (cos a, sin a): rotation.z = a - PI/2. Place at (50 cos a, 50 sin a, z). Length: from r=6 hub to r=96 ring: center r=51, length 90. Will pierce torus — ok.

But spokes at gap angles 15+90k while biomes at 0,30,...: spoke passes between biomes: distance from spoke line at radius 96... biome centers at radius 96 every 30°: nearest biome at 0° and 30° from spoke at 15°: chord distance = 96*2*sin(7.5°)=25 → biome radius 24 → clearance 1: too tight visually overlapping. Increase ring radius to 104? Then biome spacing = 2*104*sin15 = 53.8 > 48 ✓, spoke at 15°: distance from biome center (at 0°) to spoke: the spoke is a line at angle 15°; biome center at angle 0°, radius 104: perpendicular distance = 104*sin(15°)=26.9 > 24 ✓ clearance 2.9 + spoke radius 1.5 → ok. Ring radius 104, biome radius 23, length 104. Overall ship width ~ 254. Good with orbit min 60 max 1500, initial 420.

Ring A z=-110, Ring B z=+50, spine z from -280 to +280. Fuel tanks z=-30 (between rings at -30? between -110 and 50 → -30 ✓). Radiators z=+130. Engines z=+250. Bow shield z=-290.

Scale check interior biome: interior R=60 separate scene — no relation needed.

### Interaction zones implementation

```js
interactables = [] // per area: {test:(playerPosInfo)=>bool, hint, action}
```
Biome door positions: x=±119.9? walkable clamp ±116; door at x=-116 bottom θ=π region: test |x - (-114)|<3 && |wrapAngle(θ-π)|<0.15 → hint "按 E 穿过舱门 → 脊柱通道" action setArea('spine'). Also +x door → same.
Spine doors: x=-66 → biome; x=+66 → bridge.
Bridge door: at back wall z=+14 area x∈[-2,2] → spine.
Devi console: near holotable console front-right: test dist → hint "E 与 Devi 对话" → open dialog cycling lines.

E key handler: if currentHint → action.

### FPController details

```js
class FP {
  constructor(cam, dom)
  mode; yaw=0; pitch=0; 
  // cylinder state
  x=0; theta=Math.PI; 
  // flat state
  pos=new Vector3(0,1.7,10); 
  keys={};
  speed base 8 (biome 10, fly 8)
  setMode(mode, spawn) resets.
  update(dt):
    switch mode:
     'cyl': compute T,U; input f=(KeyW?1:0)-(KeyS...); s=(KeyD)-(KeyA); 
       // yaw: 0 → +X. forward=(cos yaw)X+(sin yaw)T? With yaw increasing turning left? Use mx = f*cos(yaw)*spd etc:
       moveX = (f*cos(yaw) + s*cos(yaw+PI/2)) hmm simpler: dir = f*F + s*R where F=cos(yaw)*X+sin(yaw)*T, R = -sin(yaw)*X+cos(yaw)*T (rotate -90°? pick sign; adjust by testing logic: R should be to the right when facing F with up U. R = F×U? For X=(1,0,0), U=(0,1,0): X×Y=(0,0? ) compute F×U at θ=π: T=(0,-sinπ,cosπ)=(0,0,-1). U=(0,-cosπ,-sinπ)=(0,1,0). F=(1,0,0). F×U=(1,0,0)×(0,1,0)=(0,0,1)·? cross((1,0,0),(0,1,0)) = (0*0-0*1, 0*0-1*0, 1*1-0*0) = (0,0,1). And T=(0,0,-1) → R=(0,0,1) = -T. So R = -sin? Express R in basis {X,T}: R = aX+bT: at yaw=0, R=(0,0,1)=-T → a=0,b=-1: R = cos(yaw)*? General: R = F rotated -90° about U: R = cos(yaw-? ...). Just compute R = new Vector3().crossVectors(F, U) each frame — robust! Similarly no manual basis errors.
       vel = F*f + R*s normalized*speed; 
       x += vel.x*dt; arc = vel·T *dt; theta += arc / Rw;
       clamp x; collisions;
       pos=(x, Rw cosθ, Rw sinθ); up=U;
       viewDir = F*cos(pitch) + U*sin(pitch);
     'flat': standard: F=(sin? ) yaw: dir=( -sin(yaw),0,-cos(yaw) )? pick forward=(Math.sin(yaw),0,Math.cos(yaw))? Just: forward=(Math.sin(yaw)*cosP... I'll use: viewDir=(cosP*sin(yaw), sinP, cosP*cos(yaw))? Set yaw so initial faces -Z (window): forward0=(0,0,-1) → sin(yaw)=0,cos(yaw)=-1 → yaw=π. ok.
       horizontal move with forward_h=(sin yaw,0,cos yaw), right=cross(forward_h, up=(0,1,0))→ (cross((0,0,-1),(0,1,0)) = (0*0-(-1)*1, (-1)*0-0*0, 0*1-0*0) = (1,0,0) ✓ right=+X when facing -Z ✓.
       pos += (forward_h*f + right*s)*spd*dt; clamp bounds; collisions AABB; eye y=1.7.
     'fly': move along full viewDir for W/S, right for A/D, worldUp for Space/Shift; pos clamp: x∈[-66,66], radial = sqrt(y²+z²)<4.6 → if exceed scale y,z. tube axis X at origin. 
    apply camera: position.copy; up; lookAt(pos+view).
}
```

Mouse: 
- pointerdown on canvas → if interior && !locked: dom.requestPointerLock() (wrapped try). 
- pointermove: if locked (document.pointerLockElement===dom): yaw -=? standard: yaw -= mx*0.0022? Sign: moving mouse right should look right: yaw increases? With forward=(sin yaw, cos yaw) base... For cyl mode F=cos yaw X + sin yaw T... mouse right → yaw decreases or increases depending handedness; I'll choose yaw -= mx*sens and if it feels inverted in one mode, it's still consistent across modes? Hmm risky. Let me reason: cylinder at θ=π: X=(1,0,0), T=(0,0,-1), U=(0,1,0). F(yaw)=cos·X+sin·T = (cos,0,-sin). At yaw=0 F=+X. Mouse right (mx>0) should turn right: right direction R=F×U=(cos,0,-sin)×(0,1,0) = determinant: i(0*0 - (-sin)*1) - j(cos*0 - (-sin)*0) + k(cos*1 - 0*0) = (sin, 0, cos). At yaw=0 → R=(0,0,1). Turning right means F rotates toward R: dF/dyaw = (-sin,0,-cos) = -(sin,0,cos) = -R. So increasing yaw turns LEFT. Therefore mouse right → yaw -= mx*sens ✓ (consistent).
- For flat mode: forward_h=(sin yaw,0,cos yaw); mouse right → look toward right R=F×U=(sin,0,cos)×(0,1,0) = (0*0-cos*1, cos*0-sin*0, sin*1-0*0) = (-cos,0,sin). dF/dyaw=(cos,0,-sin)= -R? R=(-cos,0,sin) → -R=(cos,0,-sin) ✓ =dF/dyaw → increasing yaw turns left again → yaw -= mx ✓ consistent. 
- pitch: mouse up (my<0) → look up: pitch -= my*sens with clamp ±1.5. my negative → pitch increases → up ✓ if viewDir uses +sin(pitch)*U ✓.
- Drag fallback: if not locked and pointer down with buttons: use movement deltas (pointer events give movementX too in most browsers) — use e.movementX available on pointermove; fallback track last positions.

Wheel in interior: maybe adjust FOV? skip.

Prevent OrbitControls conflict: my custom orbit attaches same dom listeners; enable flags: orbit.enabled only exterior; FP listens always but only acts when active interior (check currentArea.interior). Manage via flags.

Esc automatically exits pointer lock; also provide that clicking UI buttons exits lock: document.exitPointerLock().

### Touch D-pad

If ('ontouchstart' in window): show pad in interior: 4 buttons set keys KeyW/A/S/D on touchstart/end; also Space/Shift? For fly mode add two more (▲up/▼down) — show conditionally in fly. Keep simple: show 4-dir pad + (in fly) up/down buttons.

### Assemble areas config

```js
areas = {
 exterior: {scene, interior:false, update(dt,t)},
 biome: {scene, interior:true, mode:'cyl', spawn, update},
 bridge:{...mode:'flat'},
 spine:{...mode:'fly'}
}
```

Each interior scene also needs its own little update (clouds drift, holo rotate, floating props, screen anim, door glow pulse).

### Devi dialog

```html
<div id="devi"> lines ... </div>
```
show for 6s per line; button close. Lines array.

### Testing notes (to include in answer)
- Serve via local server or open directly (CDN needed). Test rotate/zoom; switch interiors; WASD; door E transitions; performance; mobile note.

Now, also "搜索并阅读" — I cannot actually browse; I should note I used my knowledge of the novel (Aurora by Kim Stanley Robinson) with its key ship features, and that web search wasn't available; the model is a faithful interpretation. Be transparent: state that the design is based on the novel's descriptions (two rings of 12 biomes, spine, 0.1c, Tau Ceti, Devi). There are multiple novels named 《极光》; the famous one is KSR's Aurora (2015). I'll go with that and note the assumption. Also there's a Chinese web novel 《极光》? The most iconic spaceship in a novel named Aurora is KSR's. I'll state assumption clearly.

One more novel accuracy check — key ship facts from Aurora:
- Two rings ("Ring A" and "Ring B"), each 12 biomes ✓.
- Biomes ~4km long, 1km diameter ✓ (I recall "each biome was four kilometers long and one kilometer wide").
- 24 biomes with Earth analog ecologies ✓ (prairie where Freya lives in... Freya's family in a prairie? They live in a "biome" called... Devi narrates; Freya's mother Badim. There are biomes like "Nova Scotia"? Actually biomes named after Earth places/ecologies: "the prairie biome", "Costa Rica"? I don't recall exactly; generic eco names are safe.)
- Ship length: about... the ship overall maybe ~10 km? With spine longer than biome length. Fine, artistic.
- Propulsion: "the ship's engines were ... it decelerated by thrust and by deploying a magnetic sail"? In the book, arrival deceleration: they used the "magnetic sail" against Tau Ceti's heliosphere + engines; acceleration at launch via laser array from Jupiter? ("the laser" pushed them). I'll include a magsail ring? A big loop around the stern? Could add a huge thin torus (superconducting magsail) stored? Visually cool: giant faint ring radius 300 around stern — might clutter; add as optional subtle torus radius 260 tube 0.6 semi-transparent at z=+200. I'll add it faint — it's a distinctive novel feature (they deploy magsail for deceleration). Include with note.
- Population ~2100 ✓.
- Year: launched 2545, story ~2600s. HUD: "船历 170 年" etc.

Water in biome: novel mentions lakes/streams ✓. Also "the far side of the biome overhead" ✓.

I think design is faithful. Time to write the final code meticulously.

Also ensure single file works from file:// with CDN (needs internet). I'll note both: double-click with internet, or `python -m http.server`.

Potential pitfalls:
- three r128: CircleGeometry, CylinderGeometry fine; sRGBEncoding exists; PointsMaterial sizeAttenuation ok; InstancedMesh ok; setFromUnitVectors ok; CanvasTexture ok; SpriteMaterial ok; Pointer lock plain JS.
- CylinderGeometry openEnded param signature (radiusTop, radiusBottom, height, radialSegments, heightSegments, openEnded) ✓.
- Multi-material cylinder: avoid.
- InstancedMesh count fixed; use dummy Object3D for transforms; instanceMatrix.needsUpdate.
- instanceColor: set via mesh.setColorAt then instanceColor.needsUpdate — supported r128 ✓.

Text on canvas: use font "bold 48px sans-serif"; Chinese chars fine.

Let me write out the code now, being careful with matching braces and API names. I'll keep functions compact. Also include slight camera head-bob in walk? small.

Biome collision objects list: trees {x, th, r:1.0}, houses {x, th, r:3.2}. In cyl mode after integrating, for each obj compute dx = x-ox; dth shortest; ds = dth*Rw; d2=dx²+ds² < (r+0.6)² → push along direction. 160 objs × per frame fine.

Bridge collisions: list of AABBs {minX,maxX,minZ,maxZ}: desks & holotable & Devi console; resolve: if inside expand along min axis. Fine.

Bridge window view: place star Points sphere radius 3000 in bridge scene + Tau Ceti sprite at z=-2000 + part of ship: long spine cylinder from z=-15 extending to -400 with shield disc at -300? Let's add: spine cyl radius 3 length 500 centered z=-270; shield disc radius 26 at z=-500? Camera far 20000 ok. Also faint engine glow far behind? not visible (behind wall). ok.

Similarly biome endcap door "窗外"? no.

Spine interior: ends closed with door circles + emissive frame; also small windows? skip.

Door visuals shared helper: makeDoor(w,h,color) group: frame emissive + dark panel + sign texture "气闸 AIRLOCK".

Signs via canvas text textures: "生态舱 A-03 · PRAIRIE" etc.

Alright — also crosshair only interior; hint text element; area info panel updates per area.

Auto-rotate exterior when idle: track lastInteract time; in update if now-last>6: thetaGoal += 0.05*dt.

Zoom limits exterior: min 90 (outside ship max dim ~ magsail 520 diameter… min 120 to stay outside rings? rings radius 128 incl biome; allow min 100 (inside magsail ok), max 2000.

Initial camera: r=480, theta≈0.8, phi≈1.15.

FOV 60; near 0.1 far 30000 (stars at 12000? set sky radius 9000, far 30000 fine).

Renderer: setClearColor black; per scene background = null (space via points) except interiors have own bg colors? biome scene: scene.background = null fine (cylinder encloses all views except through endcap doors which have door meshes). Add scene.background = new Color(0x87a06a)? Not visible anyway. Keep null.

Tone mapping: renderer.toneMapping = ACESFilmicToneMapping (r128 supports) + exposure 1.1 → nicer. With MeshStandard ok. Canvas textures set encoding sRGB? Set tex.encoding = THREE.sRGBEncoding for color maps.

Shadows: skip (perf + reliability).

Antialias true; pixelRatio clamp 2.

Now, write the HTML/CSS: dark UI, glassmorphism panels, Chinese font stack, responsive. Buttons active state. Loading overlay with spinner text "正在构建飞船…".

Code length is big but let's go. I'll write it in one artifact code block.

Let me carefully draft the JS in my head section by section, then produce final.

Key globals:

```js
let renderer, camera, canvas;
let areas={}, current=null, currentName='';
let orbit, fp;
let clock=new THREE.Clock();
let lastInteract=0;
const hintEl=..., deviEl=...;
```

Orbit class:

```js
class Orbit {
 constructor(cam, dom){ this.cam=cam; this.dom=dom;
  this.target=new THREE.Vector3(0,0,0);
  this.r=480; this.theta=0.9; this.phi=1.12;
  this.gr=480; this.gtheta=0.9; this.gphi=1.12;
  this.enabled=true; this.pointers=new Map();
  dom.addEventListener('pointerdown',e=>{ if(!this.enabled)return; this.pointers.set(e.pointerId,{x:e.clientX,y:e.clientY}); dom.setPointerCapture(e.pointerId); lastInteract=performance.now();});
  pointermove: if(!enabled)return; p=this.pointers.get(e.pointerId); if(!p)return;
    if(this.pointers.size===1){ dx=e.clientX-p.x; dy=...; this.gtheta-=dx*0.005; this.gphi-=dy*0.005; clamp phi 0.08..PI-0.08 }
    else if size===2: compute distance between two pointers before/after: store prevDist in this.pinch; scale gr *= prev/cur.
    update stored.
  pointerup/cancel: delete.
  wheel: e.preventDefault(); gr *= Math.exp(e.deltaY*0.001); clamp 110..2200.
  contextmenu prevent.
 }
 update(dt){ lerp factor k=1-Math.pow(0.0001,dt) ~ frame independent: k=Math.min(1, dt*8);
   r+=(gr-r)*k ...; idle autorotate: if(performance.now()-lastInteract>6000) gtheta+=dt*0.04;
   const sp=Math.sin(this.phi); cam.position.set(target.x + r*sp*Math.sin(theta), target.y + r*Math.cos(phi), target.z + r*sp*Math.cos(theta)); cam.up.set(0,1,0); cam.lookAt(target);
 }
}
```

Pinch: when size becomes 2, set this.pinch = dist; on move recompute newDist; gr *= pinch/newDist; clamp; pinch=newDist.

FP class as planned. Include keydown/up on window with e.code; ignore when typing? no inputs. Also prevent default for Space (scroll).

Interact key 'KeyE'.

Pointer lock: on canvas click when current.interior && !document.pointerLockElement → canvas.requestPointerLock(). Listen pointerlockchange to update flag. pointermove: if locked → apply; else if dragging (pointerdown w/o lock — e.g., lock failed) apply with movement.

I'll unify: FP attaches its own pointer handlers but ONLY acts when current.interior. Orbit only when !current.interior. Both attach to canvas; guard by enabled flags set in setArea.

FP look sensitivity 0.0023.

update(dt): also head-bob: bobT += speed*dt when moving in cyl/flat; camY offset small sin. For cyl mode offset along U * 0.05 sin(bob*2)? subtle.

Spawn configs:
- biome spawn: x=-100, theta=PI, yaw = -? Facing +X toward village at x=30: yaw=0 faces +X ✓ so yaw=0... door at x=-114 behind. ok spawn x=-80.
- bridge spawn: pos (0,1.7,10), yaw=PI (face -Z window). Check forward_h=(sin π,0,cos π)=(0,0,-1) ✓.
- spine spawn: pos(-60,0,0), yaw such that facing +X: fly mode viewDir formula: use same as flat: viewDir=(cosP*sin(yaw), sinP, cosP*cos(yaw)) → face +X → sin yaw=1 → yaw=PI/2.

Interactions per area as planned; implement in area.update via helper checkInteract(playerPosVector or state).

Player world pos: FP computes this.camPos (vector) each frame; interactions use it.

Door glow pulse: store emissive mats list with phase.

Now geometry helpers:

```js
function bendPatch(R, x0,x1, a0,a1, segX=8, segA=8){
  const geo=new THREE.BufferGeometry(); const pos=[],norm=[],uv=[],idx=[];
  for i..segX: for j..segA: x=lerp, a=lerp; y=R*Math.cos(a); z=R*Math.sin(a); n=(0,-cos,-sin); 
  build indices; return geo;
}
```
R for ground-level patches slightly less than 60 to avoid z-fight: 59.6 (above ground inner surface 60? Inner surface at radius 60; viewer inside at radius <60. Patch should sit slightly CLOSER to axis (smaller radius) to be visible above ground → R=59.7. Water 59.4 with sand rim 59.65 ring patch color sand. Fields 59.75.

Wait ground inner radius is 60 (cylinder radius 60, BackSide). Patch radius 59.7 → 0.3 above ground (toward axis) ✓ visible. Lake water 59.35, sand ring patch 59.68 slightly bigger area.

Lake at θ=π: a0=π-0.42, a1=π+0.42, x -75..-5. Sand ring: expand by 0.06/6: a±0.48, x -80..0.

Fields: at θ∈[π+0.55, π+1.0] two patches colors wheat & green rows: (x 20..70) and (x 75..115). Also θ∈[π-1.0, π-0.55] pasture dark green.

Path along circumference at x=-10? A ring path: patch x -12..-8, a 0..2π color dirt — that's a full ring road! novel vibe ✓. segA 64.

Trees avoid: lake (|a-π|<0.5 && x in -82..2), village (x 25..65 && |a-π|<0.35), path (x in -14..-6), fields regions. Sample random, reject if in zones, try 400.

Village: 7 houses placed around (x 30..60, a π±0.25), rotations random yaw.

House build: 
```js
function makeHouse(){ group; base Box(4.4,2.6,3.6) mat wall (light plaster); roof: Cone(4 segments) radius 3.4 height 1.6 rotated y PI/4 scaled (1,1,0.8)? Use Cone radius 3.3, h1.5, 4 seg, rotateY(PI/4); dark red-brown; windows: two planes emissive #ffd27a front; door plane. }
```
Position with radial orientation helper `placeRadial(obj, x, a, R)`:
```js
function placeRadial(o,x,a,R){ const c=Math.cos(a),s=Math.sin(a); o.position.set(x,R*c,R*s); const up=new THREE.Vector3(0,-c,-s); o.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0), up); }
```
Then optionally o.rotateY(rand).

Trees same via dummy for instancing: dummy.position..., dummy.quaternion..., dummy.rotateY(rand), dummy.updateMatrix(), setMatrixAt.

For instanced trunk+canopy: two InstancedMesh same matrices (canopy offset baked: canopy geometry translated up 2.2: geometry.translate(0, 2.6, 0) etc.) Trunk cylinder translate y 1.1.

Grass blades? skip.

Cloud: MeshBasicMaterial({color:0xffffff, transparent, opacity:0.32, depthWrite:false}) spheres scaled; drift.

Sunline: cylinder(0.7,0.7,232) rotateZ(PI/2)? axis Y→X: rotateZ(Math.PI/2) maps Y→X ✓ (rotation z 90°: Y axis → -X? Let's verify: Rz(90°): x'=-y, y'=x: vector (0,1,0) → (-1,0,0) = -X; fine symmetric). emissive: MeshBasicMaterial #fff6d8. Glow: additive sprite big soft scaled (240, 24)? Sprite scale x huge looks ok from most angles? Sprites always face camera — a long glow tube better: use cylinder radius 2.2 additive transparent opacity .25 white. Plus 4 pointlights.

Also far endcap bright haze sprite at both ends? skip.

Endcap: CircleGeometry(60, 48) at x=±120 rotated to face inward: circle default +Z normal; for +x cap facing -X: rotateY(-PI/2); for -x cap rotateY(PI/2). Texture: metal with painted mural? Novel: endcaps are bulkheads with maybe landscape mural. Draw: blue-sky gradient top? It's a wall; draw metal panels + big door at bottom center + sign "AIRLOCK · 气闸" + portholes. Keep metallic with door drawn + separate door mesh glowing frame at (±119.7, y=-58.6,z=0) plane 5x6 facing ∓X... door bottom at ground: at θ=π bottom y=-60+r? Door plane center y = -60+3 = -57, at x=±119.8 facing inward (toward 0). rotateY(∓PI/2). 

Add small tunnel behind door? Not needed (teleport).

Biome scene lights: ambient #b9d0a8 0.5; 4 pointlights #fff1c9 intensity 1.15 distance 170 decay 1 at y=0 z=0 x=-85,-28,28,85. Wait lights at axis; ground radius 60 → distance 60 within 170 ✓ falloff decay default 2? set decay=1.2 intensity 1.3. Test-ish; choose intensity 1.4, decay 1.

Materials MeshLambert for ground (cheap, works with points) — Lambert supports point lights fine. Trees Lambert. Houses Standard/Lambert.

Bridge materials: MeshStandard metalness .3.

Screens: makeScreenTexture(type) canvas 256x160: bg #04121c; draw grid, star dots + connecting lines (nav), waveform, text lines with random; return CanvasTexture; two animated: starChart rotate? animate by redrawing sine offset every 300ms in update (store ctx & t). Cheap enough for 1-2 screens.

Holo ship: build mini group: two torus (r 1.0 tube .12) at z ±.55? Build: spine cylinder len 3 r .05; ringA torus r .8 tube .1 at z=-.6; ringB z=+.6; 12 small boxes around each ring radius .8; engine cone; material MeshBasicMaterial({color:0x55eeff, wireframe:true, transparent, opacity:.8}) — wireframe on many geometries: use one material. Rotate group slowly; position above pedestal y=1.6 bob ±0.05. Add point light cyan .6.

Also add "crew" figures? simple capsule-ish in bridge (2) to add life: capsule via Cylinder+spheres, dark suits, seated at consoles. Quick win: make simple seated figure (torso box, head sphere). 2-3 of them. Also in biome a couple of distant "farmers"? skip biome people... maybe one person on path walking (animated position along path ring!) — charming: a villager walking the ring road: update a += speed/R. Add 2 walkers with simple leg-less body (they're far) — cone body + sphere head, oriented radial. Yes, do it — dynamic life in biome. 

Birds: 4 sprites circling: positions radius ~20 around axis, angle t*0.1+i, x oscillating: mesh small dark plane doublesided. Add — cheap dynamic. Use little "V" canvas texture sprite? Use two-triangle geometry... simple: PlaneGeometry(1.2,0.5) with canvas bird texture (drawn chevron). Billboard via Sprite. 5 sprites.

Exterior shuttle docked + blinking.

Magsail: Torus(260, 0.8, 8, 128) at z=+180, material MeshBasic additive cyan opacity 0.15; note in info.

OK — also small floating dust in all interiors: Points 200 in room bounds drifting slowly (positions updated by time offset? use shader? simpler: rotate the whole points cloud slightly / move via material? Just static points + subtle rotation of points object).

Exterior star points: geometry positions random on sphere radius 8000 (allow 4000..9000 spread for depth): PointsMaterial size 2 sizeAttenuation false? With attenuation false all same size — vary via two Points systems sizes 1.5 & 2.5 & colors. Also 25 hero stars sprites.

Nebula: 6 sprites scale 3000-6000, colors (#3a2a6b purple, #143a52 teal, #5a2e2e ember, #1d3d2a green) opacity 0.10-0.16 additive? Additive over black ok. depthWrite false.

Milky band: 40 small sprites along a tilted plane band radius 8500 scale 800 opacity .05 white-blue → soft band. Maybe skip to save time... include quick loop.

Sol: sprite warm white scale 400 at +Z 8000 (behind stern since bow faces -Z, Sol behind = +Z ✓) + slight yellow. Tau Ceti: sprite scale 500 at -Z 8500 slightly off-axis + label sprite text "τ Ceti". Directional light from Sol direction (0.2,0.3,1) normalize — wait Sol at +Z behind; light dir from +Z: directional.position.set(300,400,800). Also faint blue fill from front (Tau Ceti) intensity .25.

Ship livery lights: along spine tiny emissive dots? add small white running lights via Points on hull? skip—nav strobes enough.

Ring biome cap textures: circle canvas: metallic + ring of warm windows + center label A1..B12? Biome name text again small. Nice.

Number of textures: hull(1) + biome side 12 + caps shared 2 + grass + metal + screens 3 + signs few + glow + nebula generated per color? one white radial reused with sprite color tint! SpriteMaterial color tints — yes reuse one glow canvas for all glows/stars/nebulas with different colors & opacity. 

Engine glow cyan, sun warm white, etc.

Alright, writing final code. Also add simple FPS-safe: clock delta clamp 0.05.

Also add "回到外部" button prominent when interior. Crosshair CSS.

Also cursor: canvas cursor grab exterior / none when locked.

Now area switching must also: exit pointer lock; set crosshair; fp.setMode with spawn; update info text; toggle dpad visibility.

Edge: OrbitControls not used; my Orbit + FP both listen; guard: orbit.enabled = (name==='exterior'); fp.enabled = interior.

Wheel listener on orbit always attached but checks enabled.

Also prevent page scroll on wheel over canvas: e.preventDefault with {passive:false}.

Keydown 'Digit1..4' quick switch areas. Nice.

Devi lines:
[
 "我是飞船。准确地说，我是飞船的电脑。你可以叫我黛维。",
 "航速：光速的 9.93%。前方 11.9 光年，天仓五。",
 "二十四个生态舱，二十四种地球。请别踩坏苔藓。",
 "水循环效率 99.97%。剩下的 0.03%，是我们讲故事的素材。",
 "祝航行愉快。记住：飞船爱你，但飞船也需要你关灯节能。"
]

Info texts per area as planned.

Right, also include attribution footer: “基于金·斯坦利·罗宾逊《极光》(Aurora, 2015) 的飞船描述复原 · 粉丝作品”.

Write CSS: overlay panels with backdrop-filter blur, semi-transparent dark; accent cyan #7fd8ff; font-family system.

Mobile: media query shrink panels; buttons wrap.

Now finalize numbers for interactions:

Biome door trigger: |x|>112 → near either end & angular bottom |wrap(a-π)|<0.25 → hint/action → spine spawn at corresponding end? spine x=-60 → from -x door enter spine at -60 facing +X; from +x door enter at +60 facing -X (yaw=-PI/2). Implement action with spawn override param.

Spine doors: x>64 → bridge (spawn z=12? bridge room z back +14 door at z=+14.5; spawn (0,1.7,11) yaw=PI). x<-64 → biome (spawn x=-100? entering biome at -x end: x=-108, θ=π, yaw=0 facing +X).
Bridge door: z>13.5 && |x|<2.5 → spine spawn x=60 facing -X (yaw=-PI/2)? Spine spawn near +end (bridge side): x=60, yaw such that facing -X: viewDir=(sin yaw, cos yaw)... fly viewDir=(cosP sin yaw, sinP, cosP cos yaw); face -X → sin=-1 → yaw=-PI/2 ✓.

Devi console near (x∈[9..14], z∈[-13..-9])? Place Devi console right side x=11.5,z=-12 facing center. trigger dist<2.8 from (11.5,-12).

Bridge collisions AABBs: main console arc desks: front desk at z=-11 x∈[-9..9] depth 1.6 → box minX-9,maxX9,minZ-11.8,maxZ-10.2? But window at z=-15 wall; desk between. Side desks x=±12.5 z -6..6 width 1.6. Holotable center (0, 0) radius: box minX-1.4..1.4 both. Devi desk x 10.5..13 z -13.5..-11. Chairs skip collision (small). Room bounds: |x|<16.5, z∈[-13.5, 14.6]; door zone overrides.

Fly tube bounds: x∈[-67,67]; radial clamp 4.7.

Cyl clamp x∈[-114.5,114.5].

Now compose biome hull side texture with name: canvas 1024x256: base panels horizontal stripes, color band by biome, big text centered: name EN + 中文? Use EN + small CN. names array:
[["PRAIRIE","草原"],["OCEAN","海洋"],["TUNDRA","苔原"],["FOREST","森林"],["DESERT","荒漠"],["JUNGLE","雨林"],["TAIGA","针叶林"],["SAVANNA","稀树草原"],["ALPINE","高山"],["WETLAND","湿地"],["STEPPE","干草原"],["REEF","珊瑚礁"]]
colors per type.

Cap texture per ring letter: "RING A" / "RING B" center + windows.

Exterior ring letters also: hub text? skip.

I think we're ready. Also small: title screen loading ~ fake quick progress then fade.

Audio: implement class Ambient {ctx, gain, noiseSrc, osc; setProfile(area) adjust filter freq/gain; toggle}. Simple.

Let me now also double-check three r128 CanvasTexture encoding constant: THREE.sRGBEncoding ✓; renderer.outputEncoding = THREE.sRGBEncoding ✓; ACESFilmicToneMapping ✓.

InstancedMesh setColorAt requires material.vertexColors? No—instanceColor works with materials supporting color (MeshLambert ok) when mesh.setColorAt used, three injects instanceColor automatically (r128 supports instanceColor) ✓. Use MeshLambertMaterial({color:0xffffff}) and per-instance colors.

PointsMaterial vertexColors: true with colors attribute ✓.

Torus(radius, tube, radialSegments, tubularSegments) ✓.

Sprite scale set via sprite.scale.set(x,y,1).

Alright, also devicePixelRatio clamp; on resize update.

One more check: cylinder walk visual — ground geometry CylinderGeometry(60,60,240, 96, 8, true) rotated axis to X: mesh.rotation.z = Math.PI/2. After rotation, original +Y axis → -X or +X; UV v along height maps to X reversed — irrelevant for grass. BUT: my param position (x, R cosθ, R sinθ) — does the rotated cylinder occupy same space? Original cylinder around Y from -120..120; after Rz(90°): (x,y,z)→(-y,x,z)? Rz(θ): x'=x cosθ - y sinθ, y'=x sinθ + y cosθ; θ=90°: x'=-y, y'=x. So original point (0, 120, 0) → (-120,0,0): axis along X ✓ length spans x -120..120 ✓. Circle in Y/Z? Original circle cross-section in X/Z plane → after rotation x'=-y(=const), y'=x, z'=z → circle now in y/z ✓. Radius at y=+60 corresponds to original x=+60 — fine symmetric. 

BackSide to see from inside ✓. Side note: with rotation.z=PI/2 exactly.

Endcap circles at x=±120 ✓.

Sunline cylinder(0.7,0.7,232) rotated z PI/2 → spans x ±116 ✓ position (0,0,0).

Light placement at axis y=0,z=0 ✓.

Player at θ: pos=(x, 58.3·cosθ, 58.3·sinθ); at θ=π: (x, -58.3, 0) standing at bottom, eye 1.7 above ground 60 ✓ up=(0,1,0) ✓.

Walkers on path ring x=-10: a(t)=a0+t*0.02; placeRadial radius 59.2? Their feet at ground 60 → object origin at feet: place at R=60 embedded slightly 59.9. Body cone height 1.7 translate y .85, head sphere at 1.55. Group origin at feet ✓ placeRadial(g, -10, a, 59.9).

Birds: sprites at radius 15-25 around axis: pos=(x, r cos a, r sin a), a=t*0.15+i*1.3, r=18+6 sin(t*0.3+i), x=60 sin(t*0.07+i). scale 1.6.

Pollen: Points 250 random in lower half region drifting: store base + per-point phase; update positions each frame? 250 fine via position attribute update. Or rotate slowly. I'll do cheap: points.rotation.x += 0.02*dt — wait axis X: rotating around X spins the cloud around axis ✓ subtle drift. Also slight x oscillation.

Bridge dust similar tiny rotation around Y.

Now write the code. I'll aim for clarity + comments in Chinese. Deliver with run instructions.

Also handle WebGL fail gracefully: try/catch alert.

Let me count size... it'll be ~900-1100 lines. Acceptable.

Let me write final answer structure:
1. 说明：关于小说《极光》(Aurora by KSR) 的飞船要点（注明无法联网搜索，基于已知小说内容；列出小说中的飞船特征依据）。
2. 产物：单个 HTML 文件完整代码。
3. 运行方式：直接双击（需联网加载 three.js CDN）或本地服务器。
4. 测试说明/操作说明/已验证要点。

Note about honesty: I can't literally search the web. I'll say the model is reconstructed from the novel's canonical descriptions (KSR's Aurora). Ask? No—task says complete directly. I'll state assumption that 《极光》指 KSR 的 Aurora. 

Now write code carefully.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no"/>
<title>极光 Aurora · 世代飞船 3D 复原</title>
<style>
 :root{--acc:#7fd8ff;}
 html,body{margin:0;height:100%;overflow:hidden;background:#000;font-family:"PingFang SC","Microsoft YaHei",system-ui,sans-serif;color:#dfe9f2;}
 #c{position:fixed;inset:0;display:block;touch-action:none;}
 .panel{position:fixed;background:rgba(8,14,22,.62);border:1px solid rgba(127,216,255,.25);border-radius:10px;backdrop-filter:blur(8px);padding:10px 14px;z-index:10;}
 #title{top:12px;left:12px;max-width:330px}
 #title h1{margin:0;font-size:18px;letter-spacing:2px;color:var(--acc)}
 #title p{margin:4px 0 0;font-size:11px;line-height:1.5;opacity:.8}
 #btns{top:12px;right:12px;display:flex;gap:8px;flex-wrap:wrap;justify-content:flex-end;max-width:46vw}
 .btn{cursor:pointer;border:1px solid rgba(127,216,255,.4);background:rgba(10,20,30,.7);color:#cfeaff;font-size:13px;padding:7px 12px;border-radius:8px;letter-spacing:1px;transition:.2s;user-select:none}
 .btn:hover{background:rgba(30,60,90,.8)}
 .btn.on{background:var(--acc);color:#04222f;font-weight:700}
 #info{bottom:12px;left:12px;max-width:380px;font-size:12px;line-height:1.7}
 #info b{color:var(--acc)}
 #hint{position:fixed;left:50%;bottom:86px;transform:translateX(-50%);font-size:14px;color:#ffe9a8;text-shadow:0 0 8px #000;background:rgba(0,0,0,.45);padding:6px 14px;border-radius:20px;display:none;z-index:10;border:1px solid rgba(255,233,168,.35)}
 #devi{position:fixed;left:50%;bottom:20px;transform:translateX(-50%);max-width:640px;width:86vw;display:none;z-index:11;background:rgba(4,18,26,.85);border:1px solid rgba(127,216,255,.5);border-radius:10px;padding:12px 16px;font-size:14px;line-height:1.7}
 #devi b{color:var(--acc)}
 #cross{position:fixed;left:50%;top:50%;width:6px;height:6px;margin:-3px;border-radius:50%;background:rgba(255,255,255,.85);box-shadow:0 0 6px #000;display:none;z-index:9}
 #help{top:70px;right:12px;max-width:300px;font-size:12px;line-height:1.8;display:none}
 #load{position:fixed;inset:0;background:#02060c;z-index:50;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:14px;transition:opacity .8s}
 #load .ring{width:52px;height:52px;border:3px solid rgba(127,216,255,.2);border-top-color:var(--acc);border-radius:50%;animation:sp 1s linear infinite}
 @keyframes sp{to{transform:rotate(360deg)}}
 #pad{position:fixed;bottom:70px;right:14px;display:none;z-index:10;grid-template-columns:repeat(3,52px);grid-template-rows:repeat(2,52px);gap:6px}
 #pad div{background:rgba(20,40,60,.6);border:1px solid rgba(127,216,255,.4);border-radius:10px;color:#cfeaff;font-size:20px;display:flex;align-items:center;justify-content:center;user-select:none;touch-action:none}
 #hud{top:64px;left:12px;font-size:11px;line-height:1.9;letter-spacing:1px;color:#9fd8ef;display:none}
 @media(max-width:720px){#info{display:none}#title{max-width:56vw}#btns{max-width:60vw}}
</style>
</head>
<body>
<canvas id="c"></canvas>
... UI divs ...
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
... everything ...
</script>
</body></html>
```

Wait: with my own Orbit, only need three.min.js ✓ single CDN.

Now the big script. I'll write it out fully and carefully double check syntax mentally.

```js
(function(){
'use strict';
if(!window.THREE){alert('three.js 加载失败');return;}
const canvas=document.getElementById('c');
const renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio||1,2));
renderer.setSize(innerWidth,innerHeight);
renderer.outputEncoding=THREE.sRGBEncoding;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.15;
const camera=new THREE.PerspectiveCamera(60,innerWidth/innerHeight,0.1,40000);
addEventListener('resize',()=>{camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();renderer.setSize(innerWidth,innerHeight);});
```

Helpers:

```js
function cv(w,h){const c=document.createElement('canvas');c.width=w;c.height=h;return[c,c.getContext('2d')];}
function tex(c,rep){const t=new THREE.CanvasTexture(c);t.encoding=THREE.sRGBEncoding;if(rep){t.wrapS=t.wrapT=THREE.RepeatWrapping;}return t;}
function rnd(a,b){return a+Math.random()*(b-a);}
```

Glow canvas (radial white):
```js
const glowC=cv(128,128);{const g=glowC[1];const gr=g.createRadialGradient(64,64,0,64,64,64);gr.addColorStop(0,'rgba(255,255,255,1)');gr.addColorStop(0.35,'rgba(255,255,255,.55)');gr.addColorStop(1,'rgba(255,255,255,0)');g.fillStyle=gr;g.fillRect(0,0,128,128);}
const glowTex=tex(glowC[0]);
function sprite(color,scale,op){const m=new THREE.SpriteMaterial({map:glowTex,color,transparent:true,opacity:op,blending:THREE.AdditiveBlending,depthWrite:false});const s=new THREE.Sprite(m);s.scale.set(scale,scale,1);return s;}
```

Space builder:
```js
function addSpace(scene,R){
 R=R||9000;
 // star points
 const N=6500,pos=new Float32Array(N*3),col=new Float32Array(N*3);
 for(let i=0;i<N;i++){ const v=new THREE.Vector3().randomDirection? ... }
```
randomDirection exists r128? THREE.Vector3.randomDirection added later (r13x?). Avoid: manual:
```js
let x=rnd(-1,1),y=rnd(-1,1),z=rnd(-1,1);const l=Math.hypot(x,y,z)||1;const rr=R*(0.55+0.45*Math.random());pos[i*3]=x/l*rr;...
colors: pick white/#cfe4ff/#ffe9c9 with random brightness 0.5..1
}
const g=new THREE.BufferGeometry();g.setAttribute('position',new THREE.BufferAttribute(pos,3));g.setAttribute('color',new THREE.BufferAttribute(col,3));
const m=new THREE.PointsMaterial({size:1.6,vertexColors:true,sizeAttenuation:false,transparent:true,opacity:.95,depthWrite:false});
scene.add(new THREE.Points(g,m));
// bigger soft stars
for(let i=0;i<26;i++){const s=sprite(0xffffff,rnd(60,180),rnd(.25,.6));s.position.copy(randDir().multiplyScalar(R*0.8));scene.add(s);}
// nebulas
const nebCols=[0x4a3a8a,0x1f5a72,0x7a3a3a,0x2f6b4f,0x5a4a8a,0x8a5a2f];
for(let i=0;i<6;i++){const s=sprite(nebCols[i%6],rnd(2200,4200),rnd(.10,.16));s.position.copy(randDir().multiplyScalar(R*0.92));scene.add(s);}
// milky band
const axis=new THREE.Vector3(0.3,1,0.2).normalize();
for(let i=0;i<60;i++){const a=i/60*Math.PI*2;const v=new THREE.Vector3(Math.cos(a),0,Math.sin(a)).applyAxisAngle(axis,0.9);const s=sprite(0xaac4e8,rnd(500,1100),rnd(.03,.06));s.position.copy(v.multiplyScalar(R*0.9).add(randDir().multiplyScalar(600)));scene.add(s);}
```
randDir helper:
```js
function randDir(){let x,y,z;do{x=rnd(-1,1);y=rnd(-1,1);z=rnd(-1,1);}while(x*x+y*y+z*z>1||x*x+y*y+z*z<1e-4);return new THREE.Vector3(x,y,z).normalize();}
```

Hull textures:

```js
function hullCanvas(name,band){
 const[c,g]=cv(1024,256);
 g.fillStyle='#b9c2cc';g.fillRect(0,0,1024,256);
 // panel noise
 for(let i=0;i<260;i++){g.fillStyle=`rgba(${rnd(120,170)|0},${rnd(130,175)|0},${rnd(140,185)|0},${rnd(.08,.25)})`;g.fillRect(rnd(0,1024),rnd(0,256),rnd(6,60),rnd(4,26));}
 // panel lines
 g.strokeStyle='rgba(40,50,60,.5)';g.lineWidth=2;
 for(let x=0;x<=1024;x+=64){g.beginPath();g.moveTo(x,0);g.lineTo(x,256);g.stroke();}
 for(let y=0;y<=256;y+=52){g.beginPath();g.moveTo(0,y);g.lineTo(1024,y);g.stroke();}
 // rivets
 g.fillStyle='rgba(30,40,50,.6)';
 for(let x=8;x<1024;x+=16){g.fillRect(x,4,2,2);g.fillRect(x,250,2,2);}
 if(band){g.fillStyle=band;g.fillRect(0,96,1024,64);
  g.fillStyle='rgba(255,255,255,.9)';g.font='bold 44px sans-serif';g.textAlign='center';g.textBaseline='middle';
  g.fillText(name,512,128);
 }
 return c;
}
```

Biome side texture includes name & Chinese: name passed like "PRAIRIE · 草原". Font for Chinese in canvas ok.

Cap canvas:
```js
function capCanvas(label){
 const[c,g]=cv(512,512);
 g.fillStyle='#9aa4ae';g.fillRect(0,0,512,512);
 g.translate(256,256);
 // radial panels
 for(let i=0;i<24;i++){g.rotate(Math.PI/12);g.strokeStyle='rgba(40,50,60,.55)';g.lineWidth=3;g.beginPath();g.moveTo(30,0);g.lineTo(250,0);g.stroke();}
 g.setTransform(1,0,0,1,0,0);
 // rings
 for(const r of [80,150,225,250]){g.strokeStyle='rgba(40,50,60,.6)';g.lineWidth=4;g.beginPath();g.arc(256,256,r,0,7);g.stroke();}
 // windows ring warm
 for(let i=0;i<36;i++){const a=i/36*Math.PI*2;g.fillStyle=Math.random()<.75?'#ffd98a':'#2a3540';g.fillRect(256+Math.cos(a)*188-5,256+Math.sin(a)*188-3,10,6);}
 g.fillStyle='#22303c';g.font='bold 40px sans-serif';g.textAlign='center';g.textBaseline='middle';g.fillText(label,256,256);
 return c;
}
```
arc(…,0,7) ok (>2π).

Grass canvas:
```js
function grassCanvas(){
 const[c,g]=cv(512,512);
 g.fillStyle='#4c7a37';g.fillRect(0,0,512,512);
 for(let i=0;i<900;i++){const cols=['#5d8f42','#3e662c','#6da04b','#557f3a','#7fae52'];g.fillStyle=cols[i%5];g.globalAlpha=rnd(.15,.4);const r=rnd(2,9);g.beginPath();g.arc(rnd(0,512),rnd(0,512),r,0,7);g.fill();}
 g.globalAlpha=1;
 // dirt specks
 for(let i=0;i<160;i++){g.fillStyle='rgba(110,88,60,.35)';g.fillRect(rnd(0,512),rnd(0,512),rnd(1,3),rnd(1,3));}
 return c;
}
```

Metal floor canvas:
```js
dark '#2a3138' + grid + hazard strip + scratches.
```

Wall canvas (bridge): dark panels + ribs + few labels.

Screen canvas:
```js
function screenCanvas(kind){
 const[c,g]=cv(256,160);g.fillStyle='#03141f';g.fillRect(0,0,256,160);
 g.strokeStyle='rgba(90,200,255,.25)';for(let x=0;x<256;x+=16){...} grid
 if(kind==='map'){stars dots + route line + labels 'SOL','τ CETI' + ship marker}
 else if(kind==='wave'){sine curves 3}
 else{text rows fake data}
 border frame.
 return c;
}
```

Sign canvas: text plate.

Now EXTERIOR build:

```js
const ext=new THREE.Scene();
addSpace(ext,9500);
ext.fog=null;
// lights
ext.add(new THREE.AmbientLight(0x334455,.7));
const sun=new THREE.DirectionalLight(0xfff2dd,1.5);sun.position.set(-300,500,900);ext.add(sun);
const rim=new THREE.DirectionalLight(0x88aaff,.45);rim.position.set(400,-300,-800);ext.add(rim);
```

Wait Sol should be behind stern (+Z): sun light from +Z: position (−300,500,900) fine.

Sol & Tau Ceti sprites + labels:
```js
const sol=sprite(0xfff3cf,520,1);sol.position.set(600,300,8800);ext.add(sol);
const tau=sprite(0xffe9b0,640,1);tau.position.set(-500,-200,-8800);ext.add(tau);
function label(text,pos,scale){canvas text → sprite material normal blending}
```
label sprite: draw text on 256x64, sprite scale (260,65). Add "SOL 太阳" near sol, "τ CETI 天仓五" near tau.

Ship group:
```js
const ship=new THREE.Group();ext.add(ship);
```

Materials:
```js
const hullMat=new THREE.MeshStandardMaterial({map:tex(hullCanvas(),true),metalness:.55,roughness:.45});
const darkMat=new THREE.MeshStandardMaterial({color:0x39424c,metalness:.7,roughness:.4});
```

Spine:
```js
function cyl(r,len,mat,seg){const m=new THREE.Mesh(new THREE.CylinderGeometry(r,r,len,seg||16),mat);m.rotation.x=Math.PI/2;return m;} // axis→Z
```
Wait rotation.x=PI/2 maps Y→Z? Rx(90°): y'=−z, z'=y → (0,1,0)→(0,0,1) ✓.

```js
const spine=cyl(4,540,hullMat,20);ship.add(spine);
const spine2=cyl(6,120,darkMat,16);spine2.position.z=-40;ship.add(spine2); // thicker midsection? maybe skip
```
Add greebles instanced along spine:
```js
const greeb=new THREE.InstancedMesh(new THREE.BoxGeometry(1.6,1.6,4),darkMat,50);
dummy random around radius 4.5..5.5 angle, z -250..250; setMatrix.
```

Bow assembly (z=-270):
- shield disc: CircleGeometry(30) at z=-300 facing -Z: mesh rotation? circle normal +Z default; facing forward -Z: rotateY(PI). Material double? Standard doubleside? Make cylinder thin instead: cyl(30,2,hull) at z=-300 → disc with thickness, plus rim torus. Add canvas shield texture with scorch marks: shieldCanvas radial scuffs. MeshStandard map.
- dome: sphere half: SphereGeometry(13, 24, 16, 0, 2π, 0, π/2) rotated x=-90° to face -Z? Hemisphere opening toward +Z: place at z=-278.
- docking ring: torus(9,1.6) at z=-292 rotateY? torus default in XY plane → facing Z ✓ good at z=-292.
- sensor mast: thin cyl(0.6,40) along Z from -280..-320 plus crossbars + dish: CircleGeometry(6) at -322 facing -Z + small spike. Blinking red light top.

Stern (z=+250):
- engine block: cyl(16,46,dark) at z=250; rings detail torus.
- 3 nozzles at radius 9: cone open: CylinderGeometry(9.5,5,26,16,1,true) rotated x=-PI/2? Want flare facing +Z: bigger radius at +Z end. CylinderGeometry(rTop,rBottom,h): top +Y. After rotX(90°): +Y→+Z? Rx(90): (0,1,0)→(0,0,1) ✓ top→+Z. So rTop=9.5 at +Z ✓. position each at (cos a*9, sin a*9, 262)? engine block ends 273... let block z=248 len 46 → spans 225..271. nozzles centered z=272 spans 259..285. ok.
- nozzle inner glow: cylinder(8.2,4.2,24,16,1,true) MeshBasicMaterial cyan additive side BackSide? Inner visible from behind: material side BackSide so inside of cone visible. Also cap bright disc at throat: circle r4.5 basic white-cyan at z=260 per nozzle.
- glow sprite per nozzle at z=288 scale 26 cyan; one big sprite scale 60.
- plume: CylinderGeometry(10,30,90,16,1,true) rotX? bigger far end: top r? want narrow at nozzle, wide far: rTop(+Z)=30? Actually plume expands: near nozzle r10 (at z=285) → far r34 at z=375: rTop=34? place center z=330. material additive opacity .12 cyan, BackSide? show outside: FrontSide fine transparent.
- PointLight cyan at (0,0,300) intensity 2 distance 400.
- flicker in update.

Fuel tanks (z=-30): 6 spheres r=13 around radius 22: positions (cos a*22, sin a*22, -30); sphere hull texture stretched ok; pipes: thin cyl along z connecting? add small cyl from tank to spine: cylinder rotated radial... keep: torus around spine + struts. Enough: add ring frame torus(22,1.2) at z=-30 in XY plane ✓ + 6 struts radial length ~9 (from r6.5 to r15.5): cylinder axis Y length 9 rotated z per angle, positioned radius 11. Tanks color slightly warm: separate mat color 0xd8cfc0.

Radiators z=+140: 4 panels Box(0.8, 44, 70) radial: at angle a: position (cos a*30, sin a*30, 140), rotation.z=a (so long axis radial? Box y-length 44 radial → rotate z by (a - PI/2)? Box tall in Y; to align Y with radial dir (cos a, sin a): rotate.z = a - PI/2 ✓. Panel plane normal tangential — fine. Material: MeshStandard with emissiveMap=radCanvas, emissive 0xff5522, emissiveIntensity .8, color 0x1a0f0c, map=radCanvas(dark). radCanvas: vertical gradient black→dark red with cell grid. Animate emissiveIntensity pulse.

Rings:
```js
function buildRing(label, ringLetter){
 const g=new THREE.Group();
 const R=104;
 const girder=new THREE.Mesh(new THREE.TorusGeometry(R,2.2,10,72),darkMat);g.add(girder);
 // hub
 const hub=cyl(9,34,hullMat,16);g.add(hub);
 // spokes at 15+90k
 for(let k=0;k<4;k++){const a=(15+90*k)*Math.PI/180;const s=cyl(1.6,92,darkMat,8);s.rotation.z=a-Math.PI/2; hmm cylinder already rotated x to Z-axis... 
```
Careful: cyl() returns mesh rotated x=PI/2 (axis Z). For spoke we want axis radial in XY plane. Use fresh geometry: new CylinderGeometry(1.4,1.4,92,8) axis Y; mesh.rotation.z = a - PI/2 maps Y→ (cos a, sin a)? Rz(θ) maps Y=(0,1,0)→(-sinθ, cosθ, 0). Want (cos a, sin a): -sinθ=cos a; cosθ=sin a → θ = a - π/2: check: -sin(a-π/2)=cos a ✓ (sin(a-π/2)=-cos a), cos(a-π/2)=sin a ✓. So rotation.z=a-Math.PI/2 ✓. Position (cos a*50, sin a*50, 0). Length 92 spans radius 4..96 ✓ overlapping hub & girder ok.

biomes:
```js
for(let i=0;i<12;i++){
 const a=i*Math.PI/6;
 const [nm,cn,col]=BIOMES[i];
 const side=tex(hullCanvas(nm+' · '+cn,col),true); side.repeat.set(2,1)? canvas already 1024 wide covering circumference; cylinder uv wraps once → fine no repeat.
 const bmat=new THREE.MeshStandardMaterial({map:side,metalness:.55,roughness:.45});
 const body=new THREE.Mesh(new THREE.CylinderGeometry(23,23,104,28,1,true),bmat);
 body.rotation.x=Math.PI/2; // axis Z
 const grp=new THREE.Group();grp.add(body);
 const capT=tex(capCanvas(ringLetter+(i+1)));
 const capM=new THREE.MeshStandardMaterial({map:capT,metalness:.5,roughness:.5});
 const c1=new THREE.Mesh(new THREE.CircleGeometry(23,28),capM);c1.position.z=52;
 const c2=c1.clone();c2.position.z=-52;c2.rotation.y=Math.PI;
 grp.add(c1,c2);
 grp.position.set(Math.cos(a)*R,Math.sin(a)*R,0);
 g.add(grp);
}
 // nav lights on ring
 const nl1=navLight(0xff4444);nl1.position.set(R+23? ...
```
nav lights: place at outer edge of two biomes: (R+24,0) and (-(R+24),0): red & green + white strobe at hub. Implement navLight(color) returns Mesh sphere r1.2 MeshBasicMaterial color; store in blinkers with phase.

Ring A group position z=-110; Ring B z=+50. Counter-rotate in update: ringA.rotation.z += 0.05*dt; ringB.rotation.z -= 0.05*dt. Speed: biome at R=104: v=5.2 u/s fine visually.

Magsail: torus(240, 0.9, 6, 100) at z=+150? Behind everything: z=+170 radius 250 basic additive cyan opacity .12 + second torus r 250*? one is fine. Slight rotation animation? Keep static (superconductor hoop). Actually rotate slowly opposite? static ok.

Shuttle: small group at spoke of ring A near hub: box 6x2x3 + 2 wing boxes + glow at back; docked.

Ship done. bounding for orbit: fine.

ext.update: rings rotate, blinkers, engine flicker (t), radiator pulse, holo? no holo ext. tau/sol subtle pulse? skip.

Blinkers list global: {m, p, per} update: m.visible=((t+p)%per)<0.12.

Now BIOME interior scene:

```js
const bio=new THREE.Scene();
bio.fog=new THREE.Fog(0xa8c68f,80,420);
const R=60,L=240;
bio.add(new THREE.AmbientLight(0xcfe0b8,.55));
for(const lx of [-85,-28,28,85]){const p=new THREE.PointLight(0xfff1c9,1.35,180,1);p.position.set(lx,0,0);bio.add(p);}
```
Ground:
```js
const grassT=tex(grassCanvas(),true);grassT.repeat.set(10,5);
const ground=new THREE.Mesh(new THREE.CylinderGeometry(R,R,L,96,10,true),new THREE.MeshLambertMaterial({map:grassT,side:THREE.BackSide}));
ground.rotation.z=Math.PI/2;bio.add(ground);
```
Check rotation mapping: Rz(90°) maps axis Y→? (0,1,0)→(-1,0,0) = -X ✓ axis along X ✓. Player param pos=(x, r cosθ, r sinθ): circle in y/z ✓ matches geometry circle in y/z after rotation ✓ (original cross-section circle in x/z plane; after Rz: x'=-y, y'=x → new y = old x; circle points (cosα*R? whatever) — the set of points is {x∈[-120,120], y²+z²=R²}? Original: y=±120 axis, cross circle x²+z²=R². After: x'=-y∈[-120,120] ✓; y'=x, z'=z → y'²+z'²=R² ✓. 

Endcaps:
```js
const capC=cv(...) bulkhead mural: metal + big ring + door painted? just metal + text 'AIRLOCK 气闸 →' + bolts.
const capM=MeshLambert map cap side FrontSide? Circle normal +Z; need facing inward: at x=+120 face -X: rotateY(-PI/2) maps +Z→? Ry(-90): z'=? vector (0,0,1)→ Ry(θ): x'=x cosθ+z sinθ, z'=-x sinθ+z cosθ; θ=-90: x'=-1? (0,0,1)→(sin(-90)=-1, 0, cos(-90)=0)=(-1,0,0) ✓ faces -X. For x=-120: rotateY(PI/2).
```
Texture on circle seen from front ✓.

Patches (lake etc.) with bendPatch(Rp, x0,x1, a0,a1):
```js
function bendPatch(Rp,x0,x1,a0,a1,sx,sa){
 const g=new THREE.BufferGeometry();const p=[],n=[],u=[],id=[];
 for(let i=0;i<=sx;i++)for(let j=0;j<=sa;j++){
  const x=x0+(x1-x0)*i/sx, a=a0+(a1-a0)*j/sa;
  p.push(x,Rp*Math.cos(a),Rp*Math.sin(a));n.push(0,-Math.cos(a),-Math.sin(a));u.push(i/sx,j/sa);
 }
 for(let i=0;i<sx;i++)for(let j=0;j<sa;j++){const r1=sa+1;const a=i*r1+j,b=(i+1)*r1+j;id.push(a,b,a+1,b,b+1,a+1);}
 g.setIndex(id);g.setAttribute('position',new THREE.Float32BufferAttribute(p,3));... computeVertexNormals? we set normals manually ✓ (skip compute).
 return g;
}
```
Winding: normals toward axis; material side: FrontSide with our normals — winding determines face culling; to be safe use side:DoubleSide on patch materials (small cost). Yes DoubleSide.

Lake water: bendPatch(59.3, -76,-4, π-0.40, π+0.40, 10,10) MeshPhongMaterial({color:0x2f7fae, shininess:120, specular:0x99ddff, transparent, opacity:.92, side:DoubleSide}); shimmer: update material.color offsetHSL tiny? animate opacity .88..95.

Sand ring: bendPatch(59.65, -82,2, π-0.48, π+0.48) color 0xc9b489 lambert DoubleSide. (Under lake edges? sand bigger, water slightly smaller & above sand? water R 59.3 < sand 59.65 → water closer to axis → water visible above sand center ✓ and sand rim visible around ✓.)

Wait smaller radius = closer to viewer (axis) = on top ✓.

Fields: two bendPatch(59.7, ...) colors wheat 0xb9a24a (x 24..72, a π+0.55..π+1.0), veg 0x3e6b2f (x 76..116, same a), pasture (a π-1.0..π-0.55, x 20..90, color 0x557f3a) + crop rows instanced lines? add thin box rows instanced on wheat field: 12 rows: placeRadial thin boxes... use bendPatch thin strips alternating darker: 6 strips within wheat a-range each width 0.04 in a → simple extra patches color darker. ok do 5 strips.

Ring road: bendPatch(59.72, -12,-8, 0, 2π, 1, 64) color 0x8a7a5e — full circumference path at x≈-10 ✓ walkers walk it. Also axial path from village to lake: bendPatch(59.72, -10? ... x -10..60 at a π width 0.06: patch a π-0.03..π+0.03, x -12..62 color dirt.

Trees instanced:
```js
const trunkG=new THREE.CylinderGeometry(0.22,0.34,2.4,6);trunkG.translate(0,1.2,0);
const canG=new THREE.ConeGeometry(1.7,3.6,7);canG.translate(0,4.1,0); // canopy stacked second cone
const can2G=new THREE.ConeGeometry(1.15,2.4,7);can2G.translate(0,5.6,0);
trunks=InstancedMesh(trunkG, lambert brown, N); canopies=InstancedMesh(canG, lambert green w/ instance colors, N); canopies2 same matrix.
placement: sample; store treeColliders.
dummy.up? Use placeRadial-like: 
function radialMatrix(d,x,a,Rr,ry){ d.position.set(x,Rr*cos,Rr*sin); d.quaternion.setFromUnitVectors(UP, new Vector3(0,-cos,-sin)); if(ry)d.rotateY(ry); d.updateMatrix();}
```
Trees at Rr=R-0.15=59.85? base embed 0.15: place origin at 59.9.

Zones to avoid: lake |a-π|<0.55 & x∈[-86,6]; village rect x[26,64] |a-π|<0.30; fields a∈[π+0.5,π+1.05]&x[20,118]; path |x+10|<3; doors at |x|>113. Also forest cluster: bias 45% samples a∈[0.6π..1.15π] x∈[-40..80]. N=170.

Simplify reject function with conditions; loop until filled or 800 tries.

Houses (7): positions around village: (x: 30+i*5 jitter, a: π + jitter 0.22): 
house group: 
```js
wall Box(4.2,2.6,3.4) color #e8ddc8 lambert, translate y1.3;
roof Cone(3.2,1.6,4) rotateY(π/4) scale(1.15,1,0.95) color #7a4a3a translate y 3.4;
door plane(0.9,1.6) color #4a3428 at front z=1.71 y0.8;
win plane(0.8,0.8) emissive #ffd98a x±1.2 front z1.72 y1.5;
```
placeRadial(house, x, a, 59.9, random ry).
colliders r=3.

Fences: small instanced boxes around village/fields: 40 posts + skip.

Sunline & glow:
```js
const sunline=new THREE.Mesh(new THREE.CylinderGeometry(0.7,0.7,232,10),new THREE.MeshBasicMaterial({color:0xfff6da}));
sunline.rotation.z=Math.PI/2;bio.add(sunline);
const halo=new THREE.Mesh(new THREE.CylinderGeometry(2.4,2.4,232,10,1,true),new THREE.MeshBasicMaterial({color:0xfff2c0,transparent:true,opacity:.18,blending:Additive,side:DoubleSide,depthWrite:false}));
same rotation.
```

Clouds: 12 meshes sphere geometry shared, basic white transparent .3, scale (rnd 5..9, 2..3, 3..5), pos radius rnd 8..22 angle random, x random -100..100; update: x += drift*dt (wrap ±110), slight angle drift.

Pollen: 240 points in region radius<50: Points small size .35 color #fff8d0 transparent .5; slow rotation around X: pts.rotation.x += .02*dt ✓ (rotates around X axis — points swirl around axis ✓).

Birds: 5 sprites (dark color 0x223) scale 1.4: positions update a=t*.12+i*1.3, r=16+4*sin(t*.5+i), x=70*sin(t*.06+i*2). Sprite material normal blending (not additive) opacity .8.

Walkers: 2 groups: cone(0.5,1.3,6) color clothes + sphere head .28 skin; origin feet: cone translate .65, head translate 1.45. placeRadial each frame in update: a=a0+t*speed (speed .03 rad/s? ring circumference 2π*59≈372 u; walk 1.2 u/s → da=0.02/s ✓) x=-10 fixed. bob slight.

Doors: at both ends bottom: group: frame Box(5.4,6.4,0.6) emissive edges? Build: dark panel plane 4.6x5.6 + frame 4 thin emissive boxes cyan + sign sprite "气闸 AIRLOCK". Position (±119.6, -57.2, 0) facing ∓X (rotateY(∓PI/2)). Wait door at endcap x=±120; bottom interior point (x=±120, y≈-60+? ). Place door standing on ground at bottom: center y=-60+3.2=-56.8 ✓ z=0 ✓. Rotate to face inward: at x=+119.6 face -X rotateY(-PI/2) ✓ at -119.6 rotateY(PI/2).

Bio update: clouds, birds, walkers, water shimmer, pollen rotation, door glow pulse (emissiveIntensity via material color? use MeshBasicMaterial color set from HSL? simpler: frame material emissive not on Basic; use MeshStandardMaterial emissive for frames with emissiveIntensity animated; but no lights hit? lights exist ✓).

Interact zones biome: |x|>111 && angleDiff(a,π)<0.22 → hint → spine.

angleDiff helper wraps to [-π,π].

BRIDGE scene:

```js
const br=new THREE.Scene();
addSpace(br,6000); // stars for window
br.add(new THREE.AmbientLight(0x8899aa,.5));
point lights: (0,8,0) #cfe0ff .9 dist 60; (−10,6,−10) .5; (10,6,10) .5.
```
Room dims: x ±18, y 0..9, z ±15. Window on front wall z=-15: opening x ±10, y 2.5..7.5.

Build walls with planes: floor (36x30) metalT repeat; ceiling dark; back wall z=+15; side walls x=±18; front wall pieces: left/right of window, above, below. Material wallT (dark panels). All single-sided facing inward: plane rotate accordingly. Use helper plane(w,h,mat) then position/rotate.

Outside window content: 
- stars already.
- Tau Ceti sprite at (0,30? place slightly above center: (−200, 120, −5800)) scale 900 — visible through window ✓.
- ship bow structure: since bridge is "in the bow", out the window you see forward shield & mast: add: long boom cylinder along -Z from -15 to -400 at y=-2? Actually below center: add sensor mast visible: cylinder r0.8 len 300 at (0,-3,-160)? plus shield disc r24 at (0,0,-330) facing +Z (visible face) with hull texture + rim; plus small blinking lights on it. Nice depth.

Consoles: 
front desk: Box(16,1.1,2.2) at (0,0.55? floor0 → desk center y .55) z=-11; screens: 3 planes (4.6x1.6) tilted on desk top facing up-back: position y 1.25 z -10.9 rotateX(-0.5); emissive maps: map/wave/text screens.
side desks x=±12.5: Box(2,1.1,10) with 2 screens each? keep 1 each.
Devi console: special at (11.5, z=-12): desk box + tall screen (2.4x2) at y 2.4 facing center-ish rotateY(-0.6?) with animated face-ish canvas: draw circle wave "DEVI" text; pulse.
Chairs: 4: seat Box(1,0.5,1) y .5? plus back Box(1,1.1,0.25) y 1.2 offset z +0.4; color dark; at desks.
Crew figures: 2 seated: torso box (0.6,0.8,0.4) at y1.0 on chair, head sphere .3 y1.7, color suit #345; static.
Holotable: cylinder(1.5,1.7,1.0) at (0,0.5,0); top emissive ring torus(1.5,.06) basic cyan y1.02; hologram group at y 1.7: build miniShip(): spine cyl .05x3? scaled: use group of: spine cylinder r0.05 len 3.4 rotX; ringA torus .75/.08 at z -0.7; ringB z +0.7; 12 mini biomes per ring: boxes(0.16,0.16,0.5) around r0.75 oriented tangential? just boxes at angle, rotZ? fine; engine cone .2; all MeshBasicMaterial({color:0x66eaff,wireframe:true,transparent:true,opacity:.75}); plus glow sprite cyan scale 1.2 opacity .25; group rotate y? hologram should rotate around its vertical axis: holo ship built along Z; spin around Z? For display, spin around Y with ship tilted: set holo.rotation.x? Keep: inner group shipAlongZ; wrapper rotates Y slowly; ship tilt rotation.z=0.4 for style. bob y ±0.06. Point light cyan (0,2.2,0) intensity .7 dist 12.

Door back wall center (0, y0..5, z=15): door frame + sign "→ 脊柱通道 SPINE"; interact zone z>13.4 |x|<2.6.

Wall trims, ceiling light panels: 3 emissive planes on ceiling (6x2) white.

Floor grid texture repeat 6,5.

Bridge update: holo rotate+bob, devi screen pulse (redraw every 0.5s with sine amplitude), screen wave redraw t offset, blink small LEDs on desks (tiny emissive boxes toggling), dust points rotate.

Colliders list AABB:
front desk: {x:-8.2..8.2? width16/2=8 → minX-8 maxX8, minZ-12.1 maxZ-9.9}
left desk: minX-13.5 maxX-11.5 minZ-5 maxZ5; right mirrored.
holo: minX-1.7 maxX1.7 minZ-1.7 maxZ1.7.
devi desk: minX10.4 maxX12.6 minZ-13.4 maxZ-11.4.
chairs skip (walk through ok-ish) — include front chairs? skip.
bounds: x ±17, z -13.6..14.4, but door zone allows z up to 14.4 anyway; teleport on E.

SPINE scene:

```js
const sp=new THREE.Scene();
sp.add(ambient .45 #aabbcc); 3 point lights along x (−45,0,45) y3? tube r6 axis X: lights at (x, 0, 0) dist 40 int .9 #cfe4ff.
tube: CylinderGeometry(6,6,140,32,8,true) rotZ(π/2) BackSide metalT repeat 8,2.
ribs: InstancedMesh torus(6,0.18,6,24)? torus axis: default torus in XY plane, hole along Z; need ring around X axis: rotateY(π/2) → plane YZ ✓. instances at x=-63..63 step 7 → 19.
handrails: 4 cylinders r0.12 len 136 axis X at angles 45,135,225,315 radius 5.4: rotZ(π/2).
light strips: 2 long thin boxes (136, 0.15, 0.5) emissive white at y=±? place top y=5.9? Basic white small opacity: MeshBasic #dfeeff; also reflect? fine.
signs: 3 planes with signCanvas('SPINE TRANSIT · 脊柱通道', '↔ RING A ⟷ RING B') on wall.
floating props: crate box 1.2 at (−20, 1.5, 1) rotating slowly; toolbox small red box at (30,-2,1.5) rotating; tether line? skip.
dust: 200 points within r5, x±68: static + slight rotation.x? rotating around X spins around tube ✓ but that'd swirl dust around — zero-g dust shouldn't swirl... make slow drift via time in update: pts.position.x = sin? Just leave static + tiny rotation 0.01.
doors at x=±69.5: circle cap + door frame + signs: 'RING A · 生态舱' at -X end? Map: -X end → biome, +X end → bridge. Signs accordingly.
Also portholes? skip.
```
sp.update: props rotate/bob, dust.

Interact: x<-66 → biome (spawn -108); x>66 → bridge.

FPController implementation (write fully):

```js
const FP={
 enabled:false, mode:'flat',
 yaw:0,pitch:0,
 x:0,a:Math.PI,           // cylinder state
 pos:new THREE.Vector3(), // flat/fly state
 keys:{}, locked:false, drag:false,
 bob:0,
 setMode(mode,spawn){
  this.mode=mode;this.pitch=0;this.bob=0;
  if(mode==='cyl'){this.x=spawn.x;this.a=spawn.a;this.yaw=spawn.yaw;}
  else {this.pos.copy(spawn.pos);this.yaw=spawn.yaw;}
 },
 look(mx,my){const s=0.0023;this.yaw-=mx*s;this.pitch-=my*s;this.pitch=Math.max(-1.45,Math.min(1.45,this.pitch));},
 update(dt){
  if(!this.enabled)return;
  const k=this.keys;const f=(k.KeyW?1:0)-(k.KeyS?1:0);const s=(k.KeyD?1:0)-(k.KeyA?1:0);
  const X=new THREE.Vector3(1,0,0);
  if(this.mode==='cyl'){
   const Rw=58.3;
   const ca=Math.cos(this.a),sa=Math.sin(this.a);
   const T=new THREE.Vector3(0,-sa,ca);
   const U=new THREE.Vector3(0,-ca,-sa);
   const F=new THREE.Vector3().addScaledVector(X,Math.cos(this.yaw)).addScaledVector(T,Math.sin(this.yaw));
   const Rv=new THREE.Vector3().crossVectors(F,U);
   const sp=((f||s)?9.5:0); 
   const mx=(F.x*f+Rv.x*s), mt=(T.x? 0:0); // compute tangential comp via dot with T
   const vx=(F.x*f+Rv.x*s)*sp*dt;
   const vTangent=(F.dot(T)*f+Rv.dot(T)*s)*sp*dt; // F·T=sin(yaw), R·T=? 
   this.x+=vx; this.a+=vTangent/Rw;
   this.x=Math.max(-114.5,Math.min(114.5,this.x));
   // collisions
   for(const o of bioColliders){const dx=this.x-o.x;let da=this.a-o.a;da=Math.atan2(Math.sin(da),Math.cos(da));const ds=da*Rw;const d2=dx*dx+ds*ds;const rr=o.r+0.7;if(d2<rr*rr&&d2>1e-6){const d=Math.sqrt(d2);const push=(rr-d);this.x+=dx/d*push;this.a+=(ds/d*push)/Rw;}}
   if(f||s)this.bob+=dt*7;
   const py=Rw*Math.cos(this.a),pz=Rw*Math.sin(this.a);
   camera.position.set(this.x,py,pz);
   const U2=new THREE.Vector3(0,-Math.cos(this.a),-Math.sin(this.a));
   // recompute F with updated a
   const ca2=Math.cos(this.a),sa2=Math.sin(this.a);
   const T2=new THREE.Vector3(0,-sa2,ca2);
   const F2=new THREE.Vector3().addScaledVector(X,Math.cos(this.yaw)).addScaledVector(T2,Math.sin(this.yaw));
   const bobOff=Math.sin(this.bob)*0.05*(f||s?1:0);
   camera.position.addScaledVector(U2,bobOff);
   const view=new THREE.Vector3().addScaledVector(F2,Math.cos(this.pitch)).addScaledVector(U2,Math.sin(this.pitch));
   camera.up.copy(U2);
   camera.lookAt(camera.position.clone().add(view));
  } else if(this.mode==='flat'){
   const F=new THREE.Vector3(Math.sin(this.yaw),0,Math.cos(this.yaw));
   const Rv=new THREE.Vector3().crossVectors(F,new THREE.Vector3(0,1,0));
   const sp=((f||s)?5.2:0);
   this.pos.addScaledVector(F,f*sp*dt).addScaledVector(Rv,s*sp*dt);
   // bounds
   this.pos.x=Math.max(-16.8,Math.min(16.8,this.pos.x));
   this.pos.z=Math.max(-13.4,Math.min(14.2,this.pos.z));
   for(const b of bridgeColliders){ if(this.pos.x>b.x0&&this.pos.x<b.x1&&this.pos.z>b.z0&&this.pos.z<b.z1){
     const dxl=this.pos.x-b.x0,dxr=b.x1-this.pos.x,dzl=this.pos.z-b.z0,dzr=b.z1-this.pos.z;
     const m=Math.min(dxl,dxr,dzl,dzr);
     if(m===dxl)this.pos.x=b.x0;else if(m===dxr)this.pos.x=b.x1;else if(m===dzl)this.pos.z=b.z0;else this.pos.z=b.z1;
   }}
   if(f||s)this.bob+=dt*8;
   this.pos.y=1.7+Math.sin(this.bob)*0.04*(f||s?1:0);
   camera.position.copy(this.pos);
   const view=new THREE.Vector3(Math.cos(this.pitch)*Math.sin(this.yaw),Math.sin(this.pitch),Math.cos(this.pitch)*Math.cos(this.yaw));
   camera.up.set(0,1,0);
   camera.lookAt(camera.position.clone().add(view));
  } else { // fly
   const view=new THREE.Vector3(Math.cos(this.pitch)*Math.sin(this.yaw),Math.sin(this.pitch),Math.cos(this.pitch)*Math.cos(this.yaw));
   const Rv=new THREE.Vector3().crossVectors(view,new THREE.Vector3(0,1,0)).normalize();
   const up=(k.Space?1:0)-(k.ShiftLeft||k.ShiftRight?1:0);
   const sp=7;
   this.pos.addScaledVector(view,f*sp*dt).addScaledVector(Rv,s*sp*dt);
   this.pos.y+=up*sp*0.8*dt;
   this.pos.x=Math.max(-66.5,Math.min(66.5,this.pos.x));
   const rr=Math.hypot(this.pos.y,this.pos.z);const maxr=4.7;
   if(rr>maxr){this.pos.y*=maxr/rr;this.pos.z*=maxr/rr;}
   camera.position.copy(this.pos);
   camera.up.set(0,1,0);
   camera.lookAt(camera.position.clone().add(view));
  }
 }
};
```
Note Rv for fly when looking straight up degenerate — pitch clamped ±1.45 so view never exactly vertical ✓.

Events:
```js
addEventListener('keydown',e=>{FP.keys[e.code]=true;if(e.code==='Space')e.preventDefault();if(e.code==='KeyE')tryInteract();if(e.code==='Digit1')setArea('exterior');...});
addEventListener('keyup',e=>{FP.keys[e.code]=false;});
```
Pointer:
```js
canvas.addEventListener('pointerdown',e=>{
 lastInteract=performance.now();
 if(current&&current.interior){
   FP.drag=true;FP.lx=e.clientX;FP.ly=e.clientY;
   if(!FP.locked){canvas.requestPointerLock&&canvas.requestPointerLock();}
 }
});
document.addEventListener('pointerlockchange',()=>{FP.locked=document.pointerLockElement===canvas;});
canvas.addEventListener('pointermove',e=>{
 if(!current||!current.interior)return;
 if(FP.locked){FP.look(e.movementX,e.movementY);}
 else if(FP.drag){FP.look(e.clientX-FP.lx,e.clientY-FP.ly);FP.lx=e.clientX;FP.ly=e.clientY;}
});
addEventListener('pointerup',()=>{FP.drag=false;});
```
Orbit attaches its own pointerdown on canvas too but checks orbit.enabled (false in interior) — but both listen pointerdown: orbit's handler returns early when disabled ✓. But orbit.setPointerCapture might interfere: only when enabled ✓.

Orbit wheel {passive:false}.

setArea:
```js
function setArea(name,spawnOverride){
 currentName=name;current=AREAS[name];
 if(FP.locked)document.exitPointerLock();
 orbit.enabled=!current.interior;
 FP.enabled=current.interior;
 if(current.interior){FP.setMode(current.mode,spawnOverride||current.spawn);}
 else {camera.up.set(0,1,0);}
 document.getElementById('cross').style.display=current.interior?'block':'none';
 ... update buttons .on, info html, hud show only exterior? show hud exterior.
 hide devi box; hint hide.
 pad display: touch && interior ? grid : none; fly extra buttons toggle.
 ambient.set(name);
}
```

Spawns:
biome spawn {x:-70,a:Math.PI,yaw:0}. Entering from spine door: spawn {x:-108,a:Math.PI,yaw:0}.
spine spawn default {pos:(-60,0,0), yaw:Math.PI/2} (facing +X).
bridge spawn {pos:(0,1.7,10),yaw:Math.PI}.

Interactions registry per area function checkInteract(): compute based on area:
```js
function currentInteract(){
 if(currentName==='biome'){
  if(Math.abs(FP.x)>110.5){const da=adiff(FP.a,Math.PI);if(Math.abs(da)<0.28)return{hint:'按 E 穿过气闸 → 零重力脊柱通道',act:()=>setArea('spine',FP.x<0?{pos:new THREE.Vector3(-58,0,0),yaw:Math.PI/2}:{pos:new THREE.Vector3(58,0,0),yaw:-Math.PI/2})};}
 }
 if(currentName==='spine'){
  if(FP.pos.x<-63)return{hint:'按 E → 生态舱（草原）',act:()=>setArea('biome',{x:-106,a:Math.PI,yaw:0})};
  if(FP.pos.x>63)return{hint:'按 E → 指挥舰桥',act:()=>setArea('bridge')};
 }
 if(currentName==='bridge'){
  if(FP.pos.z>13&&Math.abs(FP.pos.x)<2.6)return{hint:'按 E → 零重力脊柱通道',act:()=>setArea('spine',{pos:new THREE.Vector3(58,0,0),yaw:-Math.PI/2})};
  const d=Math.hypot(FP.pos.x-11.5,FP.pos.z+12.4);if(d<2.8)return{hint:'按 E 与黛维（Devi）对话',act:showDevi};
 }
 return null;
}
```
In loop: const it=currentInteract(); hintEl show/hide.

showDevi cycles lines, shows box, auto-hide 7s.

AREAS:
```js
AREAS={
 exterior:{scene:ext,interior:false,update:updateExt},
 biome:{scene:bio,interior:true,mode:'cyl',spawn:{x:-70,a:Math.PI,yaw:0},update:updateBio},
 bridge:{scene:br,interior:true,mode:'flat',spawn:{pos:new THREE.Vector3(0,1.7,10),yaw:Math.PI},update:updateBr},
 spine:{scene:sp,interior:true,mode:'fly',spawn:{pos:new THREE.Vector3(-58,0,0),yaw:Math.PI/2},update:updateSp},
};
```

Main loop:
```js
const clock=new THREE.Clock();
function loop(){
 requestAnimationFrame(loop);
 const dt=Math.min(clock.getDelta(),0.05);const t=clock.elapsedTime;
 if(current){
  current.update(dt,t);
  if(current.interior){FP.update(dt);const it=currentInteract();...}
  else orbit.update(dt);
  renderer.render(current.scene,camera);
 }
}
```

updateExt(dt,t): rings rotate, engine flicker: nozzleGlowMat.opacity=.75+.2*sin(t*23)+.1*sin(t*47); glow sprites scale base*(1+.12 sin(t*19)); plume opacity .1+.04 sin(t*13); light intensity 2+.6 sin(t*29)*?; blinkers visible; radiator emissiveIntensity .7+.25 sin(t*1.7); shuttle light blink; magsail slow rotate? static; sol sprite slight pulse skip.

updateBio: clouds drift; walkers; birds; water opacity .9+.04 sin(t*1.3); pollen rotation; door frames pulse (two materials stored with phase); sunline halo opacity .16+.03 sin(t*.8).

updateBr: holo rotate y += .5dt, bob; devi canvas redraw every .12s (store ctx, draw animated equalizer); wave screen redraw every .2s; LEDs blink; dust rotate; window Tau Ceti pulse slight.

updateSp: props rotate; dust rotate tiny.

Ambient audio:
```js
const Amb={ctx:null,gain:null,on:false,
 init(){...create ctx, master gain 0, noise buffer 2s loop → biquad lowpass 220 → gain; osc sine 48Hz → gain small; connect}
 toggle(){if(!this.ctx)this.init(); this.on=!this.on; gain ramp to (on? profile level:0); button text}
 set(area){if(!this.ctx)return; filter freq per area: exterior 160, biome 400 + gain lower?, etc.}
}
```
Keep modest. Button 🔊.

Loading overlay: after build (~everything synchronous), fade out: style opacity 0 then remove; do build inside try, then start loop. Since build is synchronous on main thread, loading shows only briefly — acceptable. Add tiny timeout to let paint: setTimeout(init,30) after DOM.

Touch pad: create div#pad with buttons ▲(KeyW) ◀ ▶ ▼ and for fly ⬆(Space) ⬇(ShiftLeft)? Pad grid 3x2: place ▲ top-middle; ◀ ▼ ▶ bottom row; plus two extra buttons when fly: add row? Keep: in fly mode show extra fixed buttons '升'/'降' bottom-left? Simplify: pad has 6 cells: [⬆? ] Let me do grid 3 cols x 2 rows: cells: (empty/升), ▲, (empty/降) / ◀, ▼, ▶. The corner cells act as Space/Shift only in fly mode, hidden otherwise (visibility). Implement with touchstart/touchend setting FP.keys and preventDefault.

Detect touch: 'ontouchstart' in window → show pad when interior.

Also help overlay toggled by ? button; content as planned.

HUD exterior: 
```
船历 SHIP YEAR 168 · 航速 0.0993c
位置: 太阳系→天仓五 航程 61%
质量 1.2×10⁷ t · 船员 2,102
```
static flavor; maybe increment speed jitter? static fine, or tiny animated timer: 航程 percent increasing extremely slowly. keep static text.

Buttons: 外部视角(exterior) 生态舱(biome) 舰桥(bridge) 脊柱(spine) + 帮助 toggle + 声音 toggle.

Also clicking canvas in exterior hides help? no.

One more exterior nicety: subtle camera-facing dust particles streaking (ship moving at 0.1c → starfield streaks): add Points ~300 stretched? Add small additive lines passing by: use LineSegments moving? Simple effect: 120 tiny sprites drifting from front to back past ship (interstellar dust) — positions reset z from -1500 to +1500 moving +z fast (relative) with elongated sprite scale (2, 0.4)? Sprites face camera; elongation looks odd. Use THREE.LineSegments with fixed geometry lines along z at random x,y radius 200..600, moving: update positions z += 3000*dt wrap; each segment length ~40. Material additive faint white-blue opacity .25. This gives "moving through space" feel. Do it: geometry with 2 verts per line; store base positions; update via position attribute (300 lines → 600 verts update per frame ok).

Actually simpler: one Points cloud moving +z with wrap and streak look impossible cheaply; LineSegments fine.

Ext scene rotation? no.

Also handle "右键拖拽平移"? skip; mention not supported.

Test the biome math once more concretely:
- a=π: pos=(x, -58.3, 0) ✓ bottom. U=(0,1,0) ✓.
- yaw=0: F=(1,0,0)·cos+T·0=(1,0,0) forward +X ✓.
- Walk forward: vx=1*sp*dt ✓ x increases; vTangent=F·T*f=sin(0)=0 ✓.
- Strafe right s=1: Rv=F×U=(1,0,0)×(0,1,0)=(0,0,1). vTangent=Rv·T=(0,0,1)·(0,-sin π,cos π)=(0,0,1)·(0,0,-1)=-1 → a decreases: pos z=58.3 sin(a): a slightly <π → sin positive small? sin(π-ε)=sin ε>0 → z>0 ✓ moving toward +z = Rv direction ✓ consistent.
- Look up: pitch>0 → view includes U → toward axis ✓ sees sunline & far side ✓.
- Turn with mouse right: yaw decreases → F=(cos(-ε))X+sin(-ε)T=(1,0,+ε·?) T=(0,0,-1) → F≈(1,0,ε) toward +z = right ✓.

flat mode spawn yaw=π: F=(0,0,-1) facing -z window ✓. Rv=F×U=(0,0,-1)×(0,1,0)=( (0*0)-(-1*1), (-1*0)-(0*0), (0*1)-(0*0) )=(1,0,0) ✓ D moves +x right ✓.

fly spawn yaw=π/2: view=(sin π/2, 0, cos π/2)=(1,0,0) ✓ +X toward bridge end ✓. From biome entry at x=-58 facing +X ✓ good. From bridge entry x=58 yaw=-π/2 view=(-1,0,0) ✓ facing -X toward biome ✓.

Door positions spine: tube len 140 x ±70; caps at ±69.8; interact at ±63... clamp ±66.5: player can reach 66.5 > 63 ✓ triggers before cap ✓.

Bridge door z bounds: room z +15 wall; clamp z 14.2; trigger z>13 ✓.

Biome door trigger x>110.5: clamp 114.5 ✓ reachable; door mesh at ±119.6 visible ahead ✓.

Also in cyl mode looking at endcap: distance from x=114 to cap 120 fine.

Devi console position (11.5, z=-12.4): bounds clamp x 16.8 z -13.4 ✓ reachable; collider box prevents standing inside: minX10.4..12.6, z -13.4..-11.4: approach from z>-11.4, distance from center (11.5,-12.4) at z=-11.4 → d=1 ✓ <2.8 triggers ✓.

Now textures for walls/floor:

```js
function metalCanvas(base){ // floor
 const[c,g]=cv(512,512);g.fillStyle=base||'#2b323a';g.fillRect(0,0,512,512);
 g.strokeStyle='rgba(140,160,180,.25)';g.lineWidth=2;
 for(let i=0;i<=512;i+=64){lineH, lineV}
 // bolts
 g.fillStyle='rgba(160,180,200,.35)';for grid corners fillRect small;
 // scratches
 for(let i=0;i<120;i++){g.strokeStyle=`rgba(255,255,255,${rnd(.02,.06)})`;...}
 return c;
}
function wallCanvas(){dark blue-grey panels w/ vertical ribs + small labels + pipes lines}
```

Sign canvas:
```js
function signCanvas(txt,sub){const[c,g]=cv(512,128);g.fillStyle='#0a1622';fillRect;border cyan;g.fillStyle='#9fe0ff';font 42px;center txt;sub smaller #6f96b0;}
```

Cap (biome interior bulkhead) canvas:
```js
metal + concentric ring + big text 'A-03 PRAIRIE 生态舱' + arrows + warning stripes bottom.
```

Screen canvases as planned; Devi animated: draw face: concentric circles + waveform line + 'DEVI' + random chars; store ctx+canvas, redraw in update throttled: texture.needsUpdate=true.

Wave screen animated similarly.

Performance: two canvas redraws at ~8fps cheap.

OK. Also bridge exterior hull seen through window: add boom cylinder + shield disc:
```js
const boom=cyl(1.2,320,darkMat,10); boom.position.set(0,-4,-15-160); br.add(boom);
shield: cyl(24,2,hullShieldMat,32) at (0,-4,-360) facing? cylinder axis z after rotX ✓ disc perpendicular Z ✓ visible face toward +z (bridge) shows cap texture? cylinder caps UV small; use CircleGeometry with shieldCanvas facing +z at z=-359 plus rim.
blinkers on boom tip.
```
Through window (window y 2.5..7.5, boom at y=-4 → below window view mostly... raise to y=+1? Window center y=5: from eye 1.7 looking slightly up... hmm eye 1.7, window bottom 2.5 — need to look up slightly; boom at y=-4 not visible. Adjust window: y 1.2..7 big panoramic; boom at y=-2 visible downward-ish. Let's window y 1.0..7.2, wall below 0..1.0. Fine. Boom y=-2.5 z from -15 to -335; shield at -350 r26 centered y=-2.5. visible ✓. Tau Ceti at (-300,500,-5800)? Through window need within frustum from (0,1.7,~−10) looking -z: put tau at (0,260,-5800) scale 700 — visible when looking slightly up ✓. Also add faint planet "Aurora" moon sprite small bluish near tau? cute: sprite 0x88bbee scale 60 at (120,180,-5600) — the destination! label maybe.

Spine door signs text:
- at x=-69.5 cap: '生态舱 BIOMES ←'
- x=+69.5: '→ 舰桥 BRIDGE'
Circle caps with metal tex + door frames emissive green/red? cyan both.

Biome bulkhead cap text: ' Prairie 生态舱 · A-03 ' & door sign '气闸 → 脊柱'.

I think design complete. Now write it ALL out carefully. Must be meticulous with brackets. I'll structure with sections and comments. Estimated final size large; that's fine.

Additional check: THREE.CylinderGeometry param order (radiusTop, radiusBottom, height, radialSegments, heightSegments, openEnded) ✓.

Torus: TorusGeometry(radius, tube, radialSegments, tubularSegments) ✓.

Sprite text label:
```js
function textSprite(txt,color,scaleW){const[c,g]=cv(256,64);g.font='bold 30px sans-serif';g.textAlign='center';g.textBaseline='middle';g.shadowColor='#000';g.shadowBlur=8;g.fillStyle=color||'#bfe6ff';g.fillText(txt,128,32);const m=new THREE.SpriteMaterial({map:tex(c),transparent:true,depthWrite:false});const s=new THREE.Sprite(m);s.scale.set(scaleW,scaleW/4,1);return s;}
```

Instance color: canopies.setColorAt(i, new Color().setHSL(0.29+rnd(-.04,.04), .5, rnd(.28,.4))) — r128 Color.setHSL ✓. After loop instanceColor.needsUpdate=true. Material must be color white.

Water shimmer: store waterMat.

Now bioColliders: trees r=0.9 (only sample every tree? 170 collision checks/frame fine), houses r=3.2, plus sunline? no (at axis, unreachable: player radius fixed 58.3 — can't reach axis ✓). Lake no collision (can walk on water? eh — allow "wading" fine, water slightly below ground visually with opacity... player stays at 58.3 radius while water at 59.3 → player floats 1 unit above water surface visually. Hmm: ground 60, player eye at 58.3 (feet at 60). Water surface 59.3 → feet (60) below water (59.3)?? Feet at radius 60, water at 59.3: water is 0.7 above ground (closer to axis) → player wades 0.7 deep: eye 1.7 above feet → head above water ✓ looks like wading ✓ acceptable, even nice.

Clouds radius 8..22 — unreachable ✓. Walkers on path; no collision with player — pass through ok rare.

Houses 7 + trees ~170 colliders ok.

Bridge window wall build:
front wall z=-15, total 36 wide (x -18..18), height 9.
- below sill: plane(36, 1.0) at y=0.5
- above: plane(36, 1.8) at y=8.1 (window top 7.2 → 7.2..9)
- left: x -18..-10 → width 8 at x=-14, height 6.2 center y=4.1
- right mirrored.
- window glass: plane(20,6.2) at z=-15 y=4.1 material MeshPhysical? use MeshBasicMaterial({color:0x88ccff,transparent:true,opacity:0.06}) + frame bars: 2 vertical mullions thin boxes emissive slight.
Wall material wallMat with wallT texture.

Ceiling: plane(36,30) at y=9 rotateX(π/2) facing down (normal -y): plane default normal +z; rotateX(90°) → normal -y? Rx(90): (0,0,1)→(0,-1,0)? Rx(θ): y'=y cosθ - z sinθ; z'=y sinθ + z cosθ. θ=90: (0,0,1)→(0*? ) y'=-1, z'=0 → normal (0,-1,0) ✓ faces down.
Floor: rotateX(-90) normal +y ✓.
Side walls: x=±18 planes (30 x 9) rotateY(∓90).
Back wall z=+15 plane rotateY(π) facing -z ✓.

UV repeat: set repeat on textures: floor repeat(4,4)? floor plane 36x30 with texture repeat 6,5: clone texture? CanvasTexture shared with repeat per texture instance — floor & tube use different repeats → create separate textures (call metalCanvas twice or clone: tex.clone() shares image, needs needsUpdate... simplest call generator again).

Alright, and add subtle ceiling light panels: 3 planes (8x2.4) emissive white #dfeeff basic at y=8.98.

Carpet center? skip.

Exterior shield canvas: radial scorch:
```js
function shieldCanvas(){const[c,g]=cv(512,512);base '#8f9aa5'; center circle darker; radial streaks random arcs; bolts ring; text 'WHIPPLE SHIELD';}
```

Engine block details: add pipes torus rings around block: 3 torus(16.5,0.5) at z 236,248,260.

Also add "ring letter" signs on hubs: skip.

Double-check addSpace label sprites order vs depthWrite false fine.

Magsail placement z=+150 radius 250: radiators at z=140 r~30..52 — sail ring far bigger ✓ no overlap visually? sail at z 150 vs engine z 250 — sail in front of engine (toward center). Novel: magsail deploys at bow for deceleration? Actually deceleration sail faces direction of motion (bow, -Z) to drag against ISM... They decelerate arriving → thrust/drag forward. Put sail at bow: z=-340 radius 240 around shield? shield r30 at -300; sail hoop radius 240 at z=-320 encircling — visually striking at front ✓ "前方展开的磁帆环". Do that. Also 3 thin tethers lines from sail to ship: use thin cylinders from (±? ) — lines: cylinder radius .3 len ~ from ring edge (240) to hub... long thin cylinders at angles connecting z=-320 ring to z=-120 ring A hub: length = sqrt(240² + 200²)≈312 angled — do 3 at angles 0,120,240: compute endpoints p1=(240cos, 240sin, -320), p2=(30cos? attach radius 30, -160): midpoint & orient via quaternion setFromUnitVectors(Y, dir). Write helper tubeBetween(p1,p2,r,mat). Use for tethers & maybe spokes alt. 

```js
function tubeBetween(a,b,r,mat){const d=b.clone().sub(a);const l=d.length();const m=new THREE.Mesh(new THREE.CylinderGeometry(r,r,l,6),mat);m.position.copy(a).addScaledVector(d,.5);m.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0),d.normalize());return m;}
```

Nice, reuse for antennae etc.

Antennae at bow: 3 thin rods from dome area outward-forward + tip blinkers.

Also add RCS thruster blocks small at ring girder 4 per ring (tiny boxes) — greeble charm.

Ext update also: dust streaks update.

Let me finalize blinkers: array BLINK=[]; function blinker(color,x,y,z,per,phase){mesh sphere .9 basic color; userData; BLINK.push; parent.add}. Update in ext & br scenes (each scene's own list? single list with scene refs fine since update only when active; but blinkers in bridge scene too — call updateBlink in both updates: just iterate BLINK; harmless if scene hidden).

Time base: clock.elapsedTime t.

Font for Chinese in canvas labels: use 'sans-serif' with Chinese — fine.

Alright, also the "hud" quick keys hint in help:
```
快捷键：1 外部 / 2 生态舱 / 3 舰桥 / 4 脊柱
```

Finally loading fade & first render: setArea('exterior') then loop.

Write Devi lines array & showDevi.

Edge: pointer lock request must come from user gesture on canvas ✓ pointerdown qualifies.

iOS: pointer lock unsupported → drag fallback works ✓. D-pad shown.

Audio toggle: create on first click.

Also prevent context menu on canvas.

Now WRITE. I'll produce the final single HTML. I'll be careful about every function used being defined. Let me draft the complete file mentally-linear, then output.

Order:
1. head/style/body UI divs.
2. script three CDN.
3. main script:
   - renderer/camera/resize
   - utils (rnd, cv, tex, randDir, glowTex, sprite, textSprite, tubeBetween, adiff)
   - canvas painters (hull, cap, grass, metal, wall, screen, sign, shield, bulkhead)
   - addSpace
   - EXTERIOR scene build + state vars (ringA, ringB, engMats..., blinkers, streaks)
   - BIOME scene build + (clouds[], walkers[], birds[], waterMat, doorMats, pollen)
   - BRIDGE build + (holo, deviScr, waveScr, leds, dust)
   - SPINE build + (props)
   - colliders arrays
   - Orbit class + instance
   - FP object + events
   - interactions
   - UI wiring (buttons, help, audio, pad)
   - setArea, loop, loading hide.

Careful: texts like `'草原'` fine.

Potential bug sources to double check while writing:
- MeshBasicMaterial with map & transparent for signs on planes: plane geometry visible both sides? set side:DoubleSide for signs.
- Cylinder openEnded true for ground: no caps ✓ (endcaps added).
- biome ground rotation.z=π/2 — but texture v along original height (now X): grass repeat (10,5): u around circumference repeat 10, v along length 5 ✓.
- Instanced torus ribs: torus rotateY per instance: set dummy rotation per instance: dummy.rotation.set(0, Math.PI/2, 0); position (x,0,0) ✓ torus plane XY→ rotated to ZY plane → ring around X axis ✓.
- handrails along X: cylinder rotZ(π/2) ✓ positions (0, 5.4cos a, 5.4 sin a).
- In spine, tube geometry CylinderGeometry(6,6,140,36,1,true) rotZ(π/2) BackSide metalT repeat(10,1) ✓.

- Bridge star sphere radius 6000 while camera far 40000 ✓.

- window glass plane: ensure renders after opaque: transparent true ok.

One more consideration: biome PointLight distance 180 & decay: three r128 default decay=1? PointLight(color,intensity,distance,decay) default decay 1. Physically correct off → intensity straightforward. Use intensity 1.1 distance 200 decay 1.

Lambert + multiple pointlights fine.

Check biome brightness: ground at radius 60 from axis lights: distance 60 → with decay 1 linear falloff to 0 at 200 → factor (1-60/200)=0.7 * intensity 1.1 *4 lights overlapping → bright enough with ambient .55. OK.

Bridge: room 36 wide; point at (0,7,0) dist 60 → floor center dist 7 ✓ corners ~25 ✓.

Also biome fog color greenish matches ambient; far cap 240 away from player at other end: fog 80..420 → slight haze ✓.

Exterior: no fog.

Tone mapping exposure 1.15.

Now finalize Devi screen draw fn:

```js
function drawDevi(g,t){ // 256x256
 g.fillStyle='#02141d';g.fillRect(0,0,256,256);
 g.strokeStyle='rgba(110,230,255,.9)';g.lineWidth=2;
 for(let r=3;r>0;r--){g.beginPath();g.arc(128,110,20+r*16+6*Math.sin(t*2+r),0,7);g.globalAlpha=.15+r*.2;g.stroke();}
 g.globalAlpha=1;
 // waveform
 g.beginPath();for(let x=0;x<256;x+=4){g.lineTo(x,190+18*Math.sin(x*.09+t*6)*Math.sin(x*.013+t));}
 g.stroke();
 g.fillStyle='#9feaff';g.font='bold 28px monospace';g.textAlign='center';g.fillText('D E V I',128,110+9);
 g.font='12px monospace';g.fillText('SHIP AI · ONLINE',128,238);
}
```

Wave screen:
```js
function drawWave(g,t){bg grid; 3 sine strokes colors}
```

Map screen static.

Text screen static fake logs with Chinese/EN mix.

OK also make exterior biome cap text include RING letter & number: capCanvas('A-03').

BIOMES array with colors:
```js
const BIOMES=[['PRAIRIE','草原','#7fae52'],['OCEAN','海洋','#3f7fbf'],['FOREST','森林','#3e7a3a'],['TUNDRA','苔原','#9fb4bd'],['DESERT','荒漠','#d0b070'],['JUNGLE','雨林','#2f8f5f'],['TAIGA','泰加林','#4a7a6a'],['SAVANNA','稀树草原','#c0a860'],['ALPINE','高山','#a0aab5'],['WETLAND','湿地','#5f8f7a'],['STEPPE','干草原','#b5a05a'],['REEF','珊瑚礁','#4fa0b0']];
```

Ring B uses same sequence (novel: both rings similar set) ✓.

Ship name plate on spine mid: textSprite? On hull: small plane with signCanvas('GENERATION SHIP · 世代飞船','GSV AURORA-7') attached to spine at z=-180 radius 4.2 facing out: plane 18x4.5 curved? flat plane tangent at bottom (0,-4.4,-180) facing down rotateX(π/2)? Place at side x=4.4 facing +x rotateY(π/2). fine decorative.

Also big ring letters on hub: plane sign on hub cylinder side: place at (0,-9.4? hub r9) z ringA: skip.

Enough.

Now count lines... it's going to be long but let's write.

For the answer: also mention which novel & key design mapping table (novel feature → implementation), run instructions, test checklist. Keep prose concise; code is the bulk.

Honesty note: I can't browse the web; base on known text of KSR's Aurora. State: 若指的是其他同名小说（例如中文网文《极光》），飞船设定可在此代码基础上调整参数快速替换。Good.

Let me now write the final code in full, carefully.

While writing, keep checking: every variable defined before use at runtime (function hoisting fine for function declarations; the build code runs top-down).

I'll write it now.

(Also ensure `renderer.outputEncoding` with tone mapping doesn't wash out — fine.)

Pad buttons: define padKey mapping; touchstart: e.preventDefault(); FP.keys[code]=true; touchend/cancel: false.

Also in fly mode show up/down: two extra divs #padUp #padDn positioned left bottom. Show when current mode fly && touch.

Write the help content:

```
【外部视角】
· 左键/单指拖拽：360° 环绕查看
· 滚轮/双指捏合：缩放
· 静置 6 秒后自动缓慢环绕
【飞船内部】
· 点击画面锁定鼠标（ESC 释放）或按住拖拽：转视角
· W A S D：移动　E：互动 / 开门
· 零重力区：Space 上升 / Shift 下降
· 快捷键 1/2/3/4 切换区域
```

Info panels text (concise lore).

HUD exterior: id hud show only exterior.

Buttons data-area attr.

Audio profiles: exterior: hum louder (gain .12, filter 140Hz, osc 46Hz); biome: filter 600 noise gain .06 (wind-ish) + bird? skip bird; bridge: filter 300 gain .05 + osc 60; spine: filter 180 gain .08. Implement: noise through lowpass freq set, osc freq set, master .15.

Ensure AudioContext resume on toggle (autoplay policy): toggle handler is gesture ✓ ctx.resume().

Write showDevi:
```js
const DEVI=[ ...lines ];let deviIdx=0,deviTimer=null;
function showDevi(){const el=...;el.style.display='block';el.innerHTML='<b>黛维：</b>'+DEVI[deviIdx++%DEVI.length];clearTimeout(deviTimer);deviTimer=setTimeout(()=>el.style.display='none',7000);}
```

Interact hint element #hint.

OK writing final now. Also add `<div id="vig">`? skip.

One more: exterior initial camera set by orbit.update first frame ✓; interior cameras set by FP ✓.

Also after setArea to interior, orbit's pointer capture state might linger — orbit.pointers cleared when disabled? On setArea: orbit.pointers.clear().

Add orbit.enabled checks in all its handlers.

Also wheel zoom clamp: 110..2400.

For pinch: implement in pointermove when pointers.size===2: compute dist from stored two points each move; need previous: store this.pd (prev dist). On second pointerdown set pd. On move recompute nd; gr*=pd/nd; pd=nd; update stored coords.

Careful: with 2 pointers, rotating also happens due to movement of first pointer — skip rotate when size>1 (the `if size===1` guard) ✓.

Let me also add double-click canvas exterior → reset view? skip.

Final check on three.js CDN URL: cdnjs r128: https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js ✓ widely cached.

Add fallback: if THREE undefined after load, show message with instructions (offline). 

Write the answer: brief intro stating source interpretation + full code + run/test instructions. Use a single code block. Language: Chinese.

Let me write the whole HTML now, line by line, being extremely careful.

Additional detail check — `THREE.CircleGeometry(23,28)` fine.

For ring biome group per biome I add both caps sharing one material: c2=c1.clone() shares material ✓ rotation y π to face outward ✓ c1 at z=+52 default normal +z faces outward (away from center) ✓.

Biome cylinder length 104 → caps at ±52 ✓.

Ring girder torus radius 104 tube 2.2 ✓ biomes centered radius 104 embedded in girder ✓.

Spokes length 92 centered radius 50: spans 4..96 — hub radius 9: gap? spoke start 4 inside hub ✓ end 96 reaches girder inner edge (104-2.2=101.8, biome surface at 104-23=81 hmm spoke ends at 96 which is inside biome radius region? biome cylinder occupies radius 81..127 from axis; spoke end 96 within biome body?? Spokes at gap angles, biome at those radii but different angle: spoke at 15°, biome centers at 0°,30°: spoke end point (96, 15°): distance to biome center at (104, 0°): chord = sqrt(96²+104²-2·96·104·cos15°) = sqrt(9216+10816-19968·0.9659)= sqrt(20032-19291)= sqrt(741)≈27.2 > 23 ✓ clears.

Tethers from magsail to ship: 3 tubes.

Ship overall z: bow shield -300, sail -320, stern plume to +375. Orbit target (0,0,0) fine; maybe shift target z +20. fine 0.

Engine nozzles positions radius 9 triangle angles 90,210,330.

Fuel tanks at z=-30 between rings (-110..+50) ✓ centered -30 ✓ clear of rings.

Radiators z=+140 between ringB (50) and engine (225) ✓.

Also small comm dish on spine at z=+90: circle r5 angled.

OK writing!

After writing, provide 运行方式 & 测试说明.

Length note: The code will be ~950 lines. That's acceptable and necessary.

Let me write everything in final form now, double-checking syntax as I go.

Also, I realize for `tex()` helper with repeat wrap, and `metalCanvas` used for multiple textures — call separately.

For signs planes: MeshBasicMaterial({map, transparent:true, side:DoubleSide}) — sign canvas bg opaque dark, no transparency needed; keep non-transparent.

In biome bulkhead cap: draw mural: sky-blue gradient upper? It's a metal wall — draw metal + big painted landscape mural circle? Keep metal + text + stripes.

Let me write the painters concisely but nice.

Also bridge wall texture: dark with horizontal light strip? add cyan strip line.

Writing now for real. I'll assemble the final answer with: short header (novel basis), code block, run instructions, test checklist, customization note.

Also mention controls summary in answer.

One more time, verify FP.cyl movement when standing at θ not π, e.g., θ=π/2 (side wall): pos=(x, 0, 58.3); T=(0,-1,0); U=(0,0,-1) (toward axis ✓). F yaw0=(1,0,0). Rv=F×U=(1,0,0)×(0,0,-1)=(0*(-1)-0*0, 0*0-1*(-1), 0)=(0,1,0)·? compute cross (a2b3−a3b2, a3b1−a1b3, a1b2−a2b1) with a=(1,0,0), b=(0,0,-1): x=0*(-1)-0*0=0; y=0*0-1*(-1)=1; z=1*0-0*0=0 → (0,1,0). Rv·T=(0,1,0)·(0,-1,0)=-1 → strafe right decreases θ: from π/2 → smaller: pos y=58.3cosθ>0? cos(π/2-ε)=sin ε>0 → y>0 moving +y: at side wall position (y≈0,z≈58), "right" when facing +X with up=-Z is... up=-Z, forward=+X, right=+Y? cross(f,up)=(1,0,0)×(0,0,-1)=(0,1,0) ✓ +Y ✓ and movement +Y ✓ consistent. 

Gravity-free: player can walk all the way around — ground texture everywhere, trees everywhere ✓.

Collision wrap da uses atan2(sin,cos) ✓.

Walkers: also placeRadial each frame: set group position/quaternion; speed da=0.02*dt... position radius 59.9; ensure not colliding player — pass through ok.

Bird sprites additive? normal blending black birds: use SpriteMaterial({color:0x1a1a1a}) without map → square! Need bird shape: draw chevron on canvas 64x32: two arcs lines. Use as map with transparent true ✓.

Pollen points: PointsMaterial({color:0xfff6c0,size:0.35,transparent:true,opacity:.5,sizeAttenuation:true}).

Bridge dust: size .06 white opacity .3.

Exterior streaks: LineSegments geometry positions Float32Array(300*6): each line: p1(x,y,z), p2(x,y,z+len 30..70). radius annulus 150..700 to avoid hitting ship; x=rcos, y=rsin with r 150..700, z random -1500..1500. Update: z+=2600*dt (relative motion backward since ship moves forward -Z? Ship heading -Z: stars/dust appear to move +Z ✓). if z>1500 z-=3000. Update both p1z,p2z. Mark needsUpdate. Material LineBasicMaterial({color:0xaaccff,transparent:true,opacity:.22,blending:Additive,depthWrite:false}).

Count 220 lines.

Also exterior subtle: ship heading toward Tau Ceti at -Z ✓ tau sprite at -Z ✓ bow faces -Z ✓ engines +Z ✓ consistent narrative.

HUD: 航向 τ Ceti ✓.

Ok, writing final answer code. I must be disciplined: define everything, no typos. I'll now compose the entire file.

(Writing…)

Key sections code:

I'll write it out. Given the extreme length, in the final answer I'll keep explanatory text tight.

Let me also prepare biome bulkhead canvas:

```js
function bulkheadCanvas(txt){
 const[c,g]=cv(512,512);
 g.fillStyle='#8d979f';g.fillRect(0,0,512,512);
 g.translate(256,256);
 for(let i=0;i<16;i++){g.rotate(Math.PI/8);g.strokeStyle='rgba(50,60,70,.5)';g.lineWidth=4;g.beginPath();g.moveTo(60,0);g.lineTo(250,0);g.stroke();}
 g.setTransform(1,0,0,1,0,0);
 for(const r of [70,120,240]){g.strokeStyle='rgba(50,60,70,.6)';g.lineWidth=5;g.beginPath();g.arc(256,256,r,0,7);g.stroke();}
 g.fillStyle='#24313c';g.font='bold 34px sans-serif';g.textAlign='center';g.fillText(txt,256,250);
 g.font='20px sans-serif';g.fillText('AIRLOCK 气闸 ↓',256,290);
 return c;
}
```

Door frame helper:

```js
function makeDoor(signTxt){
 const g=new THREE.Group();
 const panel=new THREE.Mesh(new THREE.PlaneGeometry(4.6,5.6),new THREE.MeshStandardMaterial({color:0x1c262e,metalness:.6,roughness:.4}));
 panel.position.y=2.8;g.add(panel);
 const fm=new THREE.MeshStandardMaterial({color:0x111820,emissive:0x35e0ff,emissiveIntensity:1.2});
 const mk=(w,h,x,y)=>{const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,0.18),fm);m.position.set(x,y,0.05);g.add(m);};
 mk(0.35,6.0,-2.45,3);mk(0.35,6.0,2.45,3);mk(5.25,0.35,0,5.85);
 if(signTxt){const s=new THREE.Mesh(new THREE.PlaneGeometry(4.4,1.1),new THREE.MeshBasicMaterial({map:tex(signCanvas(signTxt[0],signTxt[1]))}));s.position.set(0,6.7,0.06);g.add(s);}
 g.userData.mat=fm;
 return g;
}
```
door glow pulse: collect fm mats in array with phase; update emissiveIntensity.

signCanvas:
```js
function signCanvas(t1,t2){const[c,g]=cv(512,128);g.fillStyle='#07131d';g.fillRect(0,0,512,128);g.strokeStyle='#3fd8ff';g.lineWidth=6;g.strokeRect(4,4,504,120);g.fillStyle='#aee9ff';g.font='bold 44px sans-serif';g.textAlign='center';g.textBaseline='middle';g.fillText(t1,256,t2?52:64);if(t2){g.fillStyle='#6fa8c8';g.font='26px sans-serif';g.fillText(t2,256,96);}return c;}
```

Biome door placement: door group at (±119.6,-56.8,0) rotateY(∓π/2)? door local +z faces... plane geometry faces +z; at x=+119.6 need facing -x: rotateY(-π/2) maps +z→(-1,0,0)? Ry(-90): (0,0,1)→(sin(-90),0,cos(-90))=(-1,0,0) ✓. At -119.6 rotateY(π/2) ✓. Sign texts both '气闸 AIRLOCK → 脊柱 SPINE'.

Bridge door at (0,0,14.9) facing -z: rotateY(π) ✓ sign '脊柱通道 SPINE'.

Spine doors: caps circle at ±69.8 + door at (±69.4,0,-? ) door upright on tube wall bottom? zero-g: put doors centered at cap center (0,0) at x=±69.6 facing ∓x, panel 3x3.5. Player flies to it. Interact x>63 regardless of y/z ✓. Door smaller: makeDoor scaled .7. Position (69.6,0,0) rotateY(-π/2). Cap behind.

Also add green light above door.

Right — spine caps: CircleGeometry(6) metal at ±70 rotated ✓ with a small window? skip.

Compose everything. Also I need bioColliders filled during tree/house placement: {x,a,r}.

Now produce final code text. (Ensuring no stray backticks etc.)

Also I should double check `hullCanvas` uses rnd before definition order — rnd defined above painters ✓.

In addSpace, points material sizeAttenuation false → constant size; with pixelRatio sizes small: size 1.6 ok plus 2nd layer size 3 sparse bright: add second Points 800 size 2.6 brighter. ok.

The milky band: sprites positions computed via applyAxisAngle on vector then multiplyScalar — Vector3.applyAxisAngle exists ✓.

For label sprites near sol/tau: textSprite('SOL · 太阳', '#ffe9c0', 300) position near.

Now bridge window shows tau label? add small textSprite at tau pos in bridge scene scale 120: 'τ CETI'. ✓ plus aurora moon sprite.

Time to write the final file. I'll now output the answer.

Also quickly reconsider: user asked "搜索并阅读科幻小说《极光》" — I should note briefly that I based it on Kim Stanley Robinson's 《Aurora》(2015) known descriptions (cannot actually browse). Keep to 2-3 sentences + feature mapping table. Then code. Then 运行方式 & 测试说明.

Final code — writing completely now with care.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>《极光》Aurora · 世代飞船 3D 复原</title>
<style>...(as planned)...</style>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>
<canvas id="c"></canvas>
<div id="load"><div class="ring"></div><div id="loadtxt">正在组装世代飞船…</div></div>
<div id="title" class="panel"><h1>AURORA · 极光号</h1><p>世代飞船 · 依据金·斯坦利·罗宾逊《极光》(2015) 描述复原<br>双环 × 12 生态舱 · 航速 0.1c · 目标：天仓五</p></div>
<div id="btns"> buttons... </div>
<div id="hud" class="panel"></div>
<div id="info" class="panel"></div>
<div id="help" class="panel"></div>
<div id="hint"></div>
<div id="devi"></div>
<div id="cross"></div>
<div id="pad">…</div>
<div id="pad2">升/降</div>
<script> ... </script>
</body>
</html>
```

Buttons: <div class="btn on" data-a="exterior">外部视角</div> etc + <div class="btn" id="bhelp">操作</div> <div class="btn" id="bsnd">🔇 声音</div>

Script sections as planned. Let me write final JS fully now — I'll be exhaustive.

Also remember: pad & pad2 display logic in setArea & only if touch.

Write updateExt: needs handles: ringA, ringB, engGlow (material), engSprites[], plumeMat, engLight, radMats[], streaks {geo, pos attr, data}, BLINK list.

Write updateBio handles: clouds[], walkers[{g,a0,spd}], birds[], waterMat, pollen, doorMatsBio[], haloMat.

updateBr: holo (group), holoBase, deviRedraw{g,texture}, waveRedraw, leds[], dust, doorMat.

updateSp: props[], dust.

Global DOORMATS maybe per scene list of {m,p}.

Simplify: single array ANIM.push(fn) per scene? Just write explicit update functions referencing closure vars.

I'll store module-scoped consts.

Alright, final write-out. Also test mentally for obvious runtime errors after writing.

One risk: `signCanvas` called in makeDoor before signCanvas defined — order painters before builders ✓.

Another: `tex()` sets encoding; for emissive maps fine.

InstancedMesh canopy colors: need `canopies.instanceColor.needsUpdate=true` after setColorAt loop ✓ (setColorAt creates instanceColor).

Lambert with instanceColor: supported? Instancing colors work with Lambert in r128 via `#ifdef USE_INSTANCING_COLOR` in common chunks — yes supported for built-in materials.

Now finally write answer.

Rough size estimate ~1000 lines; fine.

Also include a `<noscript>`? skip.

Writing the complete final code:

(Compose carefully — I'll mentally compile as I write.)

After code: 运行方式:
1. 保存为 aurora.html，双击打开（需联网加载 three.js CDN）。
2. 或 python -m http.server 后访问。
3. 离线: 下载 three.min.js 同目录并改 script src。

测试说明: checklist — 外部: 拖拽360°、滚轮缩放、双环反向旋转、引擎闪烁、自动环绕； 内部切换、WASD、E 开门链路 生态舱↔脊柱↔舰桥、Devi 对话、零重力飞行、触屏 D-pad、性能说明。

Also note 还原依据 table:
- 双环各12生态舱 → ringA/ringB 12 biome cylinders, 反向旋转
- biome 4km×1km → 内部圆柱世界(比例微缩), 曲面大地/头顶天轴光带
- 中央脊柱零重力 → spine fly mode
- 磁帆减速 → 艏部大环
- Devi AI → 舰桥终端
- 0.1c / Tau Ceti / 世代船员 → HUD & 星图

Good. Write it.