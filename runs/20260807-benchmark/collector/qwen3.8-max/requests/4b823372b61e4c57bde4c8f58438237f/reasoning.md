The user is asking me to search for and read the sci-fi novel "Aurora" (《极光》/Aurora), and then use frontend technology to build a 3D model of the spaceship from that novel. The requirements are:
1. A dynamic spaceship model
2. Deep-space background
3. The spaceship traveling through deep space
4. 360-degree mouse orbit view
5. Zoom in/out
6. Switch to ship interior mode
7. Ability to explore the various parts of the ship's interior

First, let me think about the novel. "Aurora" is a 2015 science fiction novel by Kim Stanley Robinson. It's about a generation ship traveling to Tau Ceti e. The ship is a rotating vessel with two rings of habitats (biomes), connected by a central spine. Key features:

- The ship is a generation ship / interstellar ark
- Two counter-rotating rings? Actually, let me recall. In Kim Stanley Robinson's "Aurora," the ship is a rotating vessel with rings of habitat biomes. The ship has two rings, each containing biomes (Earth-like ecosystems). The rings rotate to create gravity. There's a central spine/hub. The ship is called an "interstellar ark."

Actually, from the novel: the ship consists of two counter-rotating rings (Ring A and Ring B?) connected by a central axis/spine. Each ring contains biomes — village biomes, farm biomes, etc. The ship is about 1 km long? Let me try to recall details:

From "Aurora" by Kim Stanley Robinson:
- The ship is a rotating double-ring structure
- The rings contain biomes (self-contained ecosystems) — each biome is like a small valley/village
- The ship's AI narrates the story
- Traveling from Earth to Tau Ceti at about 0.1c (10% of light speed)
- Journey takes about 160-170 years
- The ship has a deflector shield / forward shield against interstellar dust
- The central spine connects the rings
- There are "village rings" — actually I recall the ship has two rings, each with multiple biomes (I think about 24 biomes total? Or 12 per ring?)
- The ship can decelerate, and there's a "wake-up" sequence
- There are shuttles/landers

Let me recall more specifically. The ship in "Aurora":
- Length: about 1 km? Diameter of the rings is…
- Two counter-rotating habitat rings (to cancel angular momentum)
- Each ring divided into biomes — I think 12 biomes per ring, or 24 total
- Biomes include: villages, farms, lakes, forests
- Central spine with engineering, reactor (fusion), fuel tanks
- Forward shield (Whipple shield?) for dust protection
- The ship is called "the ship" — the settlers just call it that
- There's a "cryo" section? No, everyone is awake — it's a generation ship
- Actually there was something about frozen embryos?

Hmm, I should search to confirm details. The user says "搜索并阅读" (search and read). I have web search capability. Let me search for details about the ship in "Aurora."

Actually, wait — could 《极光》 refer to a Chinese novel? There might be a Chinese sci-fi novel called 《极光》. Let me think… There's a novel by Kim Stanley Robinson translated into Chinese as 《极光》. Yes, "Aurora" (2015) by Kim Stanley Robinson was translated into Chinese as 《极光》. That's the most likely candidate.

But there might also be other works. Let me search to confirm.

I should use the web search tool to find details about the ship in "Aurora" by Kim Stanley Robinson. Let me search.

Plan:
1. Search for details on 《极光》 Kim Stanley Robinson spaceship structure
2. Read a few sources about the ship's design
3. Build a Three.js-based 3D model:
   - Deep-space background (starfield, nebulae, possibly Tau Ceti and a destination star)
   - Ship: central spine, two counter-rotating rings with biomes, forward shield, engines, antennas
   - OrbitControls for 360° view + zoom
   - Interior mode: camera can enter the ship — e.g., click a button or fly through; interior scenes: biome interior (curved landscape inside the ring), bridge/control room, engine room
   - Dynamic effects: rotating rings, engine glow, blinking lights, moving particles (dust), star streaming

Technical approach: single HTML file using Three.js (via CDN, importmap from unpkg/jsdelivr). OrbitControls from three/addons. Build the ship procedurally with Three.js primitives and custom geometry. For interior, I can make the ring tubes openable, or create a separate interior scene — a torus-shaped interior with a curved landscape, camera moves inside.

Interior exploration: I could implement "click on hotspots" or WASD fly mode. Simpler and more robust approach: two modes:
- Exterior orbit mode (OrbitControls around the ship)
- Interior mode: camera moves along a path inside the ring (biome deck), user can look around with mouse drag, scroll to move forward/backward along the ring; plus buttons to jump to sections: biome ring, bridge, engine room, central spine observatory.

Or use PointerLockControls with WASD for first-person exploration. But pointer lock can be awkward. Maybe a hybrid: interior uses "walk" along the ring corridor with mouse-drag look.

Let me design the ship based on the novel:

Key features of the ship in "Aurora" (from my memory + what I'll confirm with search):
- Interstellar ark, about 1 km? Actually I recall the ship being "a quarter kilometer long"? Hmm. Let me search.

What I remember from the book: the ship has two counter-rotating rings, each ring divided into biomes. Each biome is about the size of a small village/valley. The rings are like a series of connected cylinders/boxes forming a ring (like a bicycle wheel without spokes? Or with spokes?). There's a central spine/hub with elevators. The ship's AI. There's a "shield" at the front. Fusion engines at the back. There's a laser/communication array? The ship also carried "printers" and could manufacture.

Some specific terms from the book: "biomes," "the ship," "village," "farms," "lake biome," "steppes biome," "taiga," "cornwall" (biome names? I think biomes were named after places: e.g., "Cornwall," "Bretagne," "Hokkaido," "Kerala," "Sahara"?). Actually I recall the biomes being named like "Cornwall," "Bretagne," "Hokkaido," "Kerala," "Sahara"? Something like that. The main character Freya's mother Devi is a doctor. Badim is a leader. The ship's AI gradually learns to narrate.

Ring structure: I think each ring had 12 biomes? Or 24 total? Let me search.

Also: the ship rotates at about 2 rpm? Gravity about 1g? Ring radius would need to be large. If the ring radius is about 250 m, then for 1g: ω = sqrt(9.8/250) ≈ 0.198 rad/s ≈ 1.89 rpm. Reasonable.

Ship length: I recall the ship being described as about 1 km long? Or "a spindle" shape. Let me search for specifics.

Let me search:
- "Aurora Kim Stanley Robinson ship design rings biomes"
- "极光 金·斯坦利·罗宾逊 飞船 结构"
- Wiki: aurora (novel) — plot summary mentions the ship.

Let me search.

After research, I'll build a single-file HTML app. Deliverables:
- index.html (self-contained, Three.js via CDN)
- Instructions: open in a browser (needs internet for CDN), or run a local server
- Controls: drag to rotate, wheel to zoom, buttons for interior mode, hotspots

Since the user is a Chinese speaker, the UI should be in Chinese.

Design considerations: I want it rich and alive — layered backgrounds, motion, micro-interactions, strong typography contrast, a distinctive display typeface plus a readable body. Since it's a 3D app, the "page" is mostly the canvas + HUD. I'll make a beautiful HUD with Chinese display typography (e.g., "ZCOOL QingKe HuangYou" or "Noto Serif SC" for display, "Noto Sans SC" for body via Google Fonts). A sci-fi HUD with a teal/amber palette could work. Let me use deep-space blue-black with multiple accent colors (cyan for data, amber/gold for warnings, aurora green for the aurora motif — fitting for "Aurora"). Aurora motif: subtle aurora shader on the destination planet? The novel's title refers to the aurora seen on the planet Aurora (Tau Ceti e). I could render a planet with aurora in the distance. Nice touch: destination star Tau Ceti and a planet with aurora shimmer.

Ship design (procedural in Three.js):

Exterior:
1. Central spine: long cylindrical truss, about 1000 units long? Let me set scale: 1 unit = 1 m. Ship length about 700 m? Ring radius about 250 m? Hmm, that makes the ship wider than long. Let me recall actual numbers from the book. I think the book describes the rings as… I recall the ship being "about 1 km long" with two rings of diameter about 500 m? Let me search.

If exact numbers aren't available, I'll use plausible ones: length 1200 m, two rings each radius 300 m, ring cross-section about 60 m wide × 50 m tall, each ring composed of 12 biome segments separated by bulkheads, spokes connecting to hub (3-4 per ring), forward Whipple shield (layered conical/disc), aft fusion drive cluster with glowing exhaust, radiators, antennas, laser array, docking shuttles.

Counter-rotation: Ring A rotates clockwise, Ring B counterclockwise. Animate that.

Biome segments: slightly different colors (green farms, blue lake segments, gray villages) — small window strips glowing warmly. Terrain on top? The interior of the ring has land on the outer wall (gravity outward), so from outside we see a ring with small windows and possibly transparent sections? In the book I think the biomes are enclosed with no big windows? Maybe there are "skylights"? Not sure. I'll add glowing window bands and some transparent greenhouse sections.

2. Forward shield: large disc/cone at the front, layered, possibly with a faint electromagnetic field shimmer.

3. Aft: fusion engines — cluster of nozzles with animated plasma glow (shader or sprite), plus magnetic nozzle rings.

4. Radiator fins: glowing dull red/orange, near the engines.

5. Hub: command module, observation deck, docking ports.

6. Small moving elements: shuttles moving between rings? Maintenance drones (small lights moving along the spine). Particle stream of interstellar dust passing by. Blinking navigation lights.

Background:
- Starfield: Points at varying sizes/colors (use a generated sprite texture).
- Nebula: large billboard planes with procedural canvas texture (radial gradients, noise) in muted colors, or shader-based. Keep it classy.
- Distant destination: Tau Ceti star (bright point with glow) and a small planet "Aurora" with animated aurora shader — ties to the novel.
- Sun behind, very distant and small? The ship has already left the solar system; I can put a faint yellow dot labeled "太阳 Sol" behind. Nice storytelling.
- Occasional dust streaks passing near the camera (particles moving fast) to convey 0.1c travel.

HUD/UI:
- Top-left: title 《极光》 AURORA — Interstellar Ark · 世代飞船, with stats (speed 0.10c, distance to Tau Ceti 12 ly counting down, year).
- Mode switch: 外部视角 / 内部探索 (exterior/interior)
- Interior: location list: 生态环 (biome ring), 指挥舱 (bridge), 聚变引擎舱 (engine room), 中央轴心 (hub/observation)
- Info panel that shows descriptions of novel elements when hovering/clicking hotspots.
- Controls help overlay.

Interior implementation:
Option A: Build actual interior geometry inside the ring: a torus-shaped corridor with terrain on the outer wall. Camera inside the torus. Movement: constrained first-person along a circular path at ring radius, user can look around freely (drag), scroll or A/D to move along the ring. Gravity direction is outward, so "down" is away from the ship's axis. Interior scene: curved landscape — ground curving upward ahead (classic O'Neill cylinder view), trees (simple cones/spheres), village blocks, sky = inner ceiling with light strips (sunline). This is evocative and feasible.

Simpler and more robust: create a separate "interior scene" that's a straight corridor but visually curved: actually building a torus interior with the camera moving along it is feasible: place camera at radius R (ring radius), angle θ; look direction is tangent. Ground: ring segment (partial torus) with terrain material; place objects along the ring. Render only nearby segments (or fog to hide the distance). Fog helps.

Let me design the interior biome ring:
- Ring radius R = 300 (same as exterior), tube interior: width 40 m, height 30 m.
- Ground: curved plate along the outer wall — use a custom geometry: an arc segment of a "curved floor" that's a cylinder band at radius R..R (floor at radius R, ceiling at radius R-30). Build with parametric geometry: for angle φ ∈ [-φmax, φmax] around the ship axis, and across the width.
- Actually simpler: build a long "valley" that follows the ring's curvature: create geometry in a local frame and rotate copies. Make a "segment" module (e.g., 10° arc) with floor, side walls (biome walls), ceiling with light strips, and content variants (farm, village, lake, forest). Clone around the ring, but 36 segments × geometry might be heavy; use InstancedMesh for trees/houses, and merge geometry where possible. Or render only N segments near the camera and reposition them (sliding window). Sliding window is efficient: keep e.g. 7 segments, reposition ahead/behind as the camera moves. But content variation is… I can vary by index hash.

Hmm, complexity budget. Alternative: interior mode = a curated set of 3-4 static interior scenes (bridge, biome valley, engine room, hub observation), each a small environment, with first-person look-around (drag) and limited movement (WASD or click-to-move). This is more manageable and richer per scene than a full torus.

But "用户可以探索飞船内部的各个部分" — exploring various parts. A scene selector for multiple rooms satisfies "各个部分." And within the biome, allow walking along the valley (A/D or scroll) with the curved world effect. I'll implement:

Interior scenes:
1. 生态环 Biome Valley (inside Ring B, "Cornwall" biome?): curved valley, sky ceiling with sunstrip, rows of farms, trees, small village, lake glimmer. Movement: A/D or ←/→ or scroll to move along the valley; mouse drag to look. Curvature: build valley as an arc of radius R; camera constrained to circle of radius R-2 (eye height 2 m above floor at radius R… wait, gravity is outward so floor is at maximum radius, i.e., floor at radius R_floor = ring outer radius minus wall thickness; head is toward axis, i.e., smaller radius). Camera at radius R_floor - 1.7.
   - Valley geometry: arc segment with floor plane (curved), walls on both sides rising inward, ceiling arc with glowing "sunline" strip.
   - Objects placed by angle: houses (boxes with warm windows), trees (instanced), crops (green planes with stripes), lake (blue reflective plane section), people? Maybe simple animated figures? Skip people; add drifting firefly-like particles and birds? Birds = small flapping triangles — nice living detail. Keep it simple: butterflies/particles.
2. 指挥舱 Bridge (in the hub, forward): a room with a large forward viewport showing stars + shield, holographic table with ship schematic (mini rotating ship hologram — meta!), consoles with blinking screens, captain's chair. Camera inside, drag to look, maybe slight position movement with WASD.
3. 聚变引擎舱 Engine room (aft): massive fusion reactor torus, glowing plasma core, magnetic coils, pipes, catwalk, animated energy pulses traveling along conduits.
4. 中央轴心 Hub observatory/spine: zero-g axial corridor with viewports looking out to the rings and stars; floating dust motes; maybe rotating view.

That's rich. Plus hotspots with labels (CSS2D-like: I'll implement simple screen-space labels by projecting 3D points).

Given the single-file constraint, code will be long (~1500-2000 lines). That's fine.

Performance: keep polycount modest, use fog, limit lights (use emissive materials + few point lights, bake-ish look with vertex colors?). Use MeshStandardMaterial sparingly; MeshLambert/Phong cheaper. Three.js r160+ via importmap from CDN (unpkg or jsdelivr). Use module script.

Potential issue: CDN access in China might be blocked for unpkg/jsdelivr… User is likely in China (Chinese prompt). Google Fonts also blocked in China. Hmm. To be safe:
- Use Three.js from CDN but mention fallback. Options: jsdelivr (sometimes blocked), unpkg (sometimes), cdnjs (Cloudflare — usually accessible in China? Cloudflare is sometimes blocked). No guarantee. Alternative: embed three.js? Too big (~600KB minified) — can't paste.
- Fonts: use system font stack with Chinese fonts ("PingFang SC", "Microsoft YaHei", "Noto Sans CJK") and a display style via letter-spacing/weight instead of web fonts. Safer. For display typeface, use "STZhongsong"/"SimHei"? I'll use font stack: display: "Noto Serif SC", "Songti SC", "SimSun", serif with heavy weight and tracking; body: system sans. Without web fonts, limited but OK. I can try Google Fonts with graceful fallback (font-display swap; if blocked, falls back). I'll include Google Fonts link but ensure fallback stack works. Actually to keep robust, skip external fonts entirely and craft typography with system stack: display: `"Songti SC","Noto Serif CJK SC","SimSun",serif` for a literary feel (novel!), body: `"PingFang SC","Microsoft YaHei",sans-serif`. Serif display for the novel title fits the literary source. Good pairing without external dependencies.

- Three.js CDN: I'll use importmap with multiple fallbacks? Importmap can't easily fallback. I can write a small loader: try loading three from jsdelivr; onerror try unpkg; then cdnjs. For module script, dynamic import with try/catch works:

```js
const CDNS = [
 'https://unpkg.com/three@0.160.0/build/three.module.js',
 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js',
 'https://cdnjs.cloudflare.com/ajax/libs/three.js/0.160.0/three.module.min.js'
];
async function loadThree(){ for (const url of CDNS){ try { return await import(url); } catch(e){} } throw ... }
```

But OrbitControls addon imports 'three' — needs importmap. I can avoid OrbitControls entirely and write my own orbit controls (not hard: spherical coordinates, pointer drag, wheel zoom, damping). Writing custom controls removes the addon dependency — only need core three.module.js. And PointerLockControls isn't needed either (custom drag look). Good decision: custom controls, single dependency (three core), with CDN fallback chain.

Also OrbitControls-like inertia/damping I'll implement myself — no problem.

Interior movement: custom.

Raycasting for hotspots: use THREE.Raycaster.

Labels: project to screen manually.

OK. Also "搜索并阅读" — I should actually do web search to ground the model in the novel. Let me search now.

Search queries:
1. `Aurora Kim Stanley Robinson generation ship design rings biomes structure`
2. `极光 金·斯坦利·罗宾逊 飞船 双环 生态舱`
3. Maybe fetch wiki page: en.wikipedia.org/wiki/Aurora_(novel)

Let me run searches.

After research, I'll write the code. Let me draft the ship spec from the novel (to be verified):

From my memory of the book + reviews:
- Ship = "interstellar ark," about 1 km long? Some sources: "a rotating double ring about 500 meters in diameter"? Need verification.
- Two counter-rotating rings, each with 12 biomes? I recall "twenty-four biomes" total (12 per ring?). Hmm, I recall biomes listed like Cornwall, Bretagne, Hokkaido, Kerala, Sahel, Steppe, Taiga, Prairie, Pampas, Veldt, Altiplano, Outback? Something like that — grassland/region names. Actually I think there were 24 biomes in two rings (12 each). Let me verify with search.
- Speed: 0.1c ("one-tenth the speed of light"), journey about 160 years? Distance to Tau Ceti is 11.9 ly; at 0.1c ≈ 119 years + accel/decel ≈ 137? In the book they departed 2122? Arrived ~2312? Hmm, about 160 years I think. Let me verify: book begins in ship year 153? Something like that. Let me search.
- Ship AI: the narrator.
- There's a "deflector" or shield: I recall the ship has a forward shield of water/ice? Or a "Whipple shield." Also they encounter interstellar dust.
- Drive: fusion ("the engines"), deceleration is a major plot point ("slow-down"), fuel problem — mass ratio issue discovered (the ship can't slow down enough) — central crisis.
- There's a shuttle/lander ("lander") used on Aurora.
- Rings connected to spine by spokes with elevators ("the elevator" / "spokes").
- Zero-g area at the hub; gymnasiums?
- "Printers" for manufacturing; the ship recycles everything ("everything is recycled").
- Aurora's aurora: the planet has a beautiful aurora (magnetic field), and deadly prion-like pathogens.

Let me search to confirm ring count/biome count and dimensions.

Then build.

Let me also plan the HUD content (in Chinese), grounded in the novel:
- Title: 《极光》 AURORA
- Subtitle: 世代飞船 · 地球 → 鲸鱼座τ星 (Tau Ceti) e
- Stats panel: 航速 0.104c / 已航行 X 年 / 剩余距离 / 生态环转速 X rpm / 人工重力 1.0 g
- Mode buttons: 外部视角 · 内部探索
- Interior locations: 生态环·康沃尔 Biome Ring, 指挥舱 Bridge, 聚变引擎舱 Fusion Drive, 轴心观景廊 Hub Spine
- Lore tooltips: quotes? I could include short descriptive text inspired by the novel (avoid long copyrighted quotes; paraphrase). E.g., "飞船的AI用逐渐学会的语言讲述着旅程" etc.

Dynamic numbers: decreasing distance counter, ship year ticking, ring rotation.

OK, let me start searching.

Then I'll write a single HTML file. Let me think through the code structure carefully to get it right in one shot.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" ...>
<title>极光 · AURORA — 世代飞船交互模型</title>
<style> ... HUD styles ... </style>
</head>
<body>
  <canvas id="scene"></canvas> (or a container div; three creates the canvas)
  HUD elements...
  <script type="importmap">? No — use dynamic import fallback. Actually importmap + fallback is complex; use dynamic import of core only.
  <script type="module"> main </script>
</body>
</html>
```

Loading flow:
```js
const boot = async () => {
  setStatus('正在加载 Three.js 渲染引擎…');
  const THREE = await loadThree();
  init(THREE);
};
```
All code inside init that receives the THREE namespace. Dynamic import of a module returns the module namespace — three.module.js exports everything, so `const THREE = await import(url)` works.

CDN list (three@0.160.0 or 0.164): use 0.160.0 (stable, has all features I need). Paths:
- https://unpkg.com/three@0.160.0/build/three.module.js
- https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js
- https://esm.sh/three@0.160.0 (esm.sh wraps; should be fine)
- https://cdnjs.cloudflare.com/ajax/libs/three.js/0.160.0/three.module.min.js — does cdnjs have three.module.min.js? cdnjs hosts three.module.min.js for recent versions. I think so (they have three.module.js and .min). I'll include it.

Custom OrbitControls (exterior):
- State: theta, phi, radius, target (Vector3)
- pointerdown → drag rotate; wheel → radius *= exp(dy*k), clamp [minR, maxR]; right-drag or two-finger to pan? Pan optional — skip pan, keep target on the ship (better UX for viewing the ship). Actually a slight pan might help inspect the aft; but keep simple: no pan; zoom min allows close inspection. Add double-click to focus on a part? Could raycast to set target to a clicked part — nice micro-interaction: click a ship part → camera target eases to that part + info card shows its name/description. This adds the "探索" feel even in exterior. Yes, implement: raycast on click (distinguish click vs drag by movement threshold), ease controls.target to the hit point (or a predefined anchor for named parts), show info panel.

Named parts with info (Chinese):
- 前导防护罩 (forward shield): "多层惠普尔防护盾…"
- 生态环 A/B (rings): counter-rotating, biomes
- 中央脊轴 (spine)
- 聚变推进器 (fusion drive)
- 散热翼 (radiators)
- 通讯天线/激光阵列
- 穿梭机/着陆器 (lander bay)
- 轴心指挥舱 (hub)

Exterior camera easing: implement a simple tween (lerp with easing per frame toward the goal).

Interior mode transitions: fade overlay (CSS opacity), switch scene, place camera. Provide "返回外部" button.

Interior scenes: build each as a THREE.Group added to an interiorScene (or same scene with visibility toggling; simpler: 2 scenes? Renderer can render any scene; use one scene and toggle group visibility, but exterior starfield is visible from interior through windows — actually nice: bridge viewport should show real stars! So keep one scene; interior rooms are groups placed far away? If rooms are inside the ship model, camera near plane and being inside geometry works — but the exterior ship geometry would occlude/intersect. Place interior rooms at a distant offset (e.g., x +5000) in the same scene, with their own local environment; bridge "windows" show stars because the starfield surrounds everything. But the ship itself is at origin — from a distant room, the ship is a small speck; hide ship in interior mode (shipGroup.visible=false) and show the room. Stars remain visible through windows.

Dust streaming particles: also visible in interior? Disable in interior.

Biome valley: build along an arc. Parameters: R = 300 (ring radius to floor). Arc length visible ~400 m → φmax = 200/300 ≈ 0.66 rad ≈ 38°. Build geometry:

Local frame: ring axis = ship spine axis (let's say interior scene uses Y-up as "up" toward axis? Or simpler: build valley along a circle in XZ plane centered at room-local origin, with floor at radius R, "up" (toward axis) = -radial… hmm.

Let me define: valley circle center at (0,0,0) in room group, circle in XZ plane. Floor at radius R (y=0 plane? no — floor is a cylindrical surface of radius R around Y axis, but we stand on the inside… wait, gravity is outward, so we stand on a surface whose normal points inward (toward axis)? No: floor pushes us outward (centrifugal). Feet at larger radius, head toward axis. So floor surface = cylinder of radius R_floor, we stand on its outer side? The normal of the floor at our feet points toward axis (inward) = "up" direction for us. So floor is a cylindrical surface of radius R_floor, and we stand on its concave side? Let me think: O'Neill cylinder: people live on the inside surface of the cylinder; "down" is outward (away from axis), "up" is toward axis. Floor surface normal (pointing toward the person) is inward (toward axis). The surface is concave when viewed from inside — yes, we're inside the cylinder. Looking along the ring, the ground curves upward ahead and behind (both ends rise). Overhead (toward axis, at radius R - H where H = ring interior height ~30-40 m) is the "sky" = ceiling structure with light strips.

So in room-local coordinates: axis = Y? If the circle is in XZ plane around Y axis, then floor = cylinder around Y axis at radius R, camera at radius R - 1.7 (eye height toward axis), with head pointing -radially… wait, up = toward axis = direction from floor point to center. Camera up vector = -radial direction (pointing to center). Camera position at radius (R - 1.7).

Walking: change angle θ; position = (sinθ*(R-1.7), eyeY?, cosθ*(R-1.7)) — but Y is axis direction = "lateral" (width direction of the valley). Valley width W=40 along Y (axis). Floor spans y ∈ [-W/2, W/2]. Walls at y=±W/2 rise from floor (radius R) inward to ceiling radius R-H. Ceiling = cylinder of radius R-H spanning y ∈ [-W/2, W/2]? Plus possibly sloped walls. Simplify: floor cylinder band (radius R, |y|<W/2), ceiling cylinder band (radius R-H, |y|<W/2), two side walls connecting them (surface of constant y, radial from R to R-H) — like a rectangular torus section. Add angled "terraces" later.

Geometry construction: custom BufferGeometry via parametric function:
- floorGeo: positions: for i in 0..segA, j in 0..segW: θ = θ0 + i*dθ, y = -W/2 + j*dy; pos = (sinθ*R, y, cosθ*R). Normal = inward (-sinθ, 0, -cosθ). Indices grid.
- Ceiling similar at radius R-H, normal outward (+radial) (facing down toward floor).
- Walls: at y=±W/2, radius from R-H to R, normal ∓Y.
Build one big merged arc segment covering the full visible arc (e.g., 50° with 100 segments) — static, camera moves within. 100×2 vertices × 4 surfaces ≈ trivial.

Content along the valley (placed by angle θi, lateral y):
- Sunstrip: emissive box along the ceiling center (y=0, radius R-H+0.5), spanning the arc — glowing warm white, acts as "sun." Plus RectAreaLight? Too heavy; use HemisphereLight + DirectionalLight pointing radially outward? Light direction varies along the arc… Simpler: ambient + hemisphere + a few point lights following the camera ("sunlight" rig): a directional light placed at camera θ, pointing from axis outward through the camera position → simulates sunstrip illumination locally. Directional light with target updated. Good.
- Houses: village cluster at some θ range: boxes with gabled roofs (prisms), warm emissive windows (small planes).
- Farm: striped green planes on the floor (slightly offset inward to radius R-0.05), different greens; possibly row instancing.
- Trees: InstancedMesh cones + trunks along edges; ~200 instances fine.
- Lake: blue plane section with animated shimmer (MeshPhong with env? Simple: animated opacity/color via shader? Use MeshStandard with high metalness? Simple: MeshPhongMaterial with specular + slight vertex wave? Keep static with moving highlight from light).
- Butterflies/particles: small Points drifting.
- Birds: 3-4 simple V-shaped sprites flapping along path (scale oscillation) — cute alive detail. Maybe skip birds; particles + light shafts enough. I'll add drifting pollen/fireflies Points with sine motion.

Fog in valley: scene.fog to fade distant arc — but fog would affect other modes too; set per mode (scene.fog = new FogExp2 with density toggled). In exterior, very thin fog or none.

Camera control in valley: drag to look (yaw/pitch around current orientation, with up = local radial-inward), A/D or ←→ or wheel to move along θ. Maintain camera up vector = direction to axis (0 - pos).normalized… For a camera at angle θ, up = (-sinθ, 0, -cosθ)?? Position P = (sinθ*r, y0, cosθ*r); axis point at same y: A=(0,y0,0); up = normalize(A - P) = (-sinθ, 0, -cosθ). Yes.
Look direction: maintain yaw relative to tangent and pitch. Tangent T = (cosθ, 0, -sinθ) (direction of increasing θ). Forward = T*cos(yawOffset)… Compute forward from yaw (around up axis) and pitch: build basis: up U, right Rt = normalize(cross(T?, U))… Let me define: forward = rotate T around U by yaw, then pitch around right axis. Implement with quaternions:
- Basis: U (up), F0 = tangent, R0 = cross(U? sign care)
- q = quaternion from U and yaw/pitch: camera.quaternion = Quaternion().setFromRotationMatrix(m) where m = basis(F, U, R). Compute F = (T rotated by yaw around U) then pitch: F2 = F*cos(pitch) + U*sin(pitch)? Pitch positive = look up (toward axis). Then R = cross(F2, U)… ensure orthonormal: R = normalize(cross(F2, U))? cross(F,U) gives right if left/right? In three.js, camera looks down -Z; use lookAt-style matrix: m.makeBasis(X=right, Y=up, Z=backward) where backward = -F. right = normalize(cross(U, F))? Let me do: back = F.clone().negate(); right = new Vector3().crossVectors(U, back).normalize()? cross(U, back)… hmm sign. Easier: use camera.up = U and camera.lookAt(P + F). camera.lookAt handles orientation given up vector. Yes! Set camera.position=P, camera.up=U, camera.lookAt(P+F).

Bridge scene: hexagonal room, big trapezoidal window in front (transparent), stars visible; consoles: boxes with emissive screens (canvas texture with random glyphs? Generate small canvas texture with grid/telemetry — nice). Holographic ship: mini wireframe ship (scaled-down clone of exterior group as wireframe, rotating) above a circular pedestal — strong "wow" and self-referential. Chairs: simple. Lighting: dim blue + console glow + a spotlight on holo. Camera: drag look from fixed position (or small WASD movement within radius). Add subtle idle sway? No — keep stable.

Engine room: torus reactor core with pulsating emissive shader (use MeshBasicMaterial with animated color intensity + point light), surrounding coils (torus segments), pipes (cylinders), catwalk (box grid), animated energy pulses: small glowing spheres traveling along pipe curves (CatmullRom curve, getPointAt(t)). Warning lights rotating (spotlight with rotating target or emissive blink). Sound? Skip audio (autoplay issues); optional ambient hum via WebAudio on user click? Could add a toggle for a subtle engine hum generated with oscillator + noise. Nice extra but adds complexity; I'll add a small audio toggle that generates a brown noise + low sine via WebAudio, default off. Actually keep scope in check — maybe skip audio. Hmm, "动态效果" is visual. Skip audio to reduce risk. Might include a small one… I'll decide later; low priority.

Hub spine corridor: cylindrical corridor along axis with ring windows on the side showing rings outside (since we hide the ship in interior mode, we'd need to show it… Alternative: put fake rings outside the window? Simpler: hub observatory = a glass dome room at the spine middle, with a 360° view: keep shipGroup visible in this mode but hide the hub's own geometry region? Complicated. Alternative: hub scene is a zero-g observation blister with large windows showing stars + a projected "exterior camera feed" screen showing the rings (a screen showing… can't easily render a second view… actually renderer can render twice with scissor — possible but heavy). Simplify: hub observatory shows stars through the glass + slowly rotating view, plus floating objects (pens, droplets) drifting — zero-g feel. And a large screen showing a schematic. Enough; or make the 4th location = 穿梭机舱 lander bay with a lander vehicle. Let me pick 4 interior locations: 生态环, 指挥舱, 聚变引擎舱, 轴心观景台 (zero-g observation blister with glass dome, floating particles, star view). Good variety: gravity valley, command, engineering, zero-g.

Transitions: black fade 300ms, switch, fade in. Plus a "warp" sound? No.

Hotspot labels in interior: e.g., in biome: "村庄 Village," "农场 Farms," "湖 Lake" as projected labels. In bridge: "全息星图," "导航台." Implement a generic label system: array of {position (world), text, mode} → project each frame, position a div. Hide if behind camera or occluded? Occlusion check via raycast is costly per label per frame — do every few frames or skip occlusion (labels through walls are bad…). Limit label count (~10), raycast per label every 150ms is fine.

Simpler: only show labels when in that mode, and positions are within visible area; skip occlusion (rooms are open). OK.

Exterior info card on part click: name + description + maybe a quote-style line (paraphrased). Card slides in from right.

Top bar: title left; mode tabs center; stats right (live updating). Bottom-left: control hints. Bottom-right: location chips (in interior) or viewpoint presets (in exterior): 全景 / 前部 / 生态环 / 引擎 (camera preset tweens).

Typography:
- Display: serif stack for 《极光》 title, large weight contrast; HUD numerics in monospace ("SF Mono", "Consolas", monospace) for telemetry. Body: system sans.
- Colors: deep space #05070f; accents: aurora green #7dffb0 / cyan #6fe3ff, warm amber #ffb347 for ship lights, soft red #ff6b5e for engine/warning. Multiple hues.

Loading screen: title + progress shimmer + status text (CDN attempts). If all CDNs fail: show error with instructions (需要联网加载 three.js；或下载 three.module.js 放同目录). Also support local fallback: try './three.module.js' last — user can place file locally. Nice touch for offline: mention in 运行方式.

Star texture for Points: generate via canvas radial gradient.

Nebula: 4-6 large sprites with canvas-generated soft blobs (draw many random radial gradients on a 256 canvas, tinted). Place far away, additive blending, slow rotation/drift. Colors: teal, deep blue, faint magenta — keep muted.

Aurora planet in the distance: small sphere with custom shader? Simpler: sprite glow + small sphere with animated emissive bands (shader: sin bands near poles). A small ShaderMaterial sphere with moving green/purple bands = aurora — thematic centerpiece in the distance. Plus a bright star sprite labeled "鲸鱼座τ". And behind, a faint sun.

Dust streaks: LineSegments or Points moving in +Z direction fast near camera, wrap around. In exterior only, subtle.

Ship exterior construction details (procedural, meters):
- Spine: from z=-350 (aft) to z=+450 (fore)? Let me set axis along Z, fore = +Z? Convention: ship travels toward +Z? Let's say fore = -Z? Doesn't matter; fore = +Z, engine at -Z. Actually Tau Ceti point placed at far +Z, ship nose facing it.
- Dimensions (novel-inspired, to be verified): total length ~1000? Ring radius ~250? Ring placement z = -50 and +90? Two rings separated along spine. Spokes: 4 per ring, radial cylinders from hub (r=20) to ring inner radius.
- Ring: torus-like but segmented: 12 segments per ring (boxy arcs) — build each segment as a curved box: use CylinderGeometry arc? Better: custom curved box via ExtrudeGeometry along an arc? Simplest: each segment = a "box" oriented tangentially: 12 segments → each subtends 30°, chord box slightly inset — looks faceted, which fits "biome can" aesthetic. Or use partial torus with rectangular cross-section via custom BufferGeometry (like the interior floor builder but closed box ring). I'll write a helper `arcBox(R, w, h, a0, a1, segs)` that generates a curved box (6 faces) — reuse for ring segments. Ring cross-section: width (axial) 60, height (radial) 45. Between segments, small gaps with visible bulkhead fins.
- Ring windows: thin emissive strips along outer face; plus per-segment color tint (farm green, lake blue-gray, village warm) as subtle panel coloring on outer face — use vertex colors or per-segment material tint (dark with slight hue).
- Spokes: cylinders radius 4, 4 per ring at 45° offset, plus an elevator car (small box) animating along one spoke — alive detail!
- Hub: central cylinder r=16 around spine between rings + command module at fore of hub (sphere/cone with window band), docking arms.
- Forward shield: at fore z=+L: stacked discs: thin disc r=140, then r=120 spaced, with a cone truss behind; slight iridescent shimmer material (MeshPhongMaterial with specular). Plus faint "field" shimmer: transparent additive disc with animated opacity? Subtle.
- Aft engines: cluster: central fusion nozzle: cone + glowing disc; magnetic rings (3 tori decreasing radius) with emissive cyan; exhaust: additive cone sprite/plume with animated shader (use simple ShaderMaterial: gradient + noise flicker) or stacked sprites with pulsing scale/opacity. Plume length ~120, pointing -Z. Plus 4 smaller attitude thrusters.
- Radiators: 2 large flat panels (thin boxes) angled near aft, glowing dull red-orange with emissive gradient (canvas texture), possibly slow shimmer.
- Antennas: forward thin mast with dish (pointing back to Earth? Comm laser points back to Sol — place a small dish pointing -Z/+Z). Add blinking nav lights (small spheres with emissive pulsing + PointLight? Just emissive + sprite glow).
- Landers: 2 small shuttle shapes docked at hub bay.
- Slow whole-ship bobbing? Ship is inertial; rings rotate: ringA.rotation.z += ω*dt, ringB -= ω*dt (rings are in XY plane around Z axis — yes, ring plane ⟂ spine, rotation around spine axis Z).
- Visual scale: for rendering, use meters directly with camera far 200000 (stars at 50000). near=0.5. Z-fighting risk at distance; use logarithmicDepthBuffer? Might hurt perf slightly but fine. Set near=1, far=120000, default depth OK.

Exterior camera default: position at (600, 250, 900) looking at origin; min radius 60 (close), max 6000.

Click-to-focus anchors: predefined named anchors with positions (e.g., shield center, ring A edge, engine nozzle, hub, radiator, antenna, lander). Raycast against ship meshes; map mesh → part via userData.partId; focus target = hit point; show info card.

Numbers/stats (fictional, from the novel): departure 2122? Actually the book: launch ~2120s? Hmm. Book begins in ship year ~153? I'll verify with search; if not found, present: 启航 2122 · 目标 鲸鱼座τ星e "Aurora" · 航程 11.9 光年 · 巡航速度 0.104c · 船上人口 ~2000? I recall ~2000 people (24 biomes × ~100?). Something like "两千名船员." Verify via search.

HUD live telemetry: 剩余距离 decreasing at 0.104 c (per real second → simulate: 1 real second = 30 ship days? Show "时间加速 ×2.6亿"?) Simpler: show 已航行: 153 年 221 天 ticking slowly (advance 1 ship day per real 2 seconds), 剩余: X.XX ly computed. Ring rotation: 1.9 rpm. Speed: 31,200 km/s.

OK, searches first. Let me run web searches now.

Also the user said "搜索并阅读科幻小说《极光》" — I should confirm which novel. I'll search in Chinese too.

Search 1: 极光 小说 飞船 (to identify the work)
Search 2: Aurora Kim Stanley Robinson ship rings biomes details

Then maybe fetch a wiki page.

After research, I'll write the code. Given the response length constraint, the HTML will be big; I'll aim for compact but complete code. Let me be efficient in code style (helper functions, loops) while keeping readability.

Let me start searching.
