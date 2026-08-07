The user is asking to search for and read the sci-fi novel "Aurora" (《极光》), then build a 3D spaceship model based on it using frontend technology. However, there's a problem here — I can't actually search the internet. Let me think about which novel this is.

There are several possibilities for a sci-fi novel called "Aurora":
1. Kim Stanley Robinson's "Aurora" (2015) — a generation ship novel. The ship is a rotating double-ring spacecraft (two torus-shaped habitat rings connected by a central axis/spine), with propulsion systems, a deflector shield at the front, and a "seed" ship at the rear. The ship rotates for artificial gravity. The rings contain biomes. This is the most famous sci-fi novel called "Aurora."

Details of the ship from Kim Stanley Robinson's "Aurora":
- A rotating generation ship with two habitat rings (toruses) connected by a central spine
- The rings rotate to provide artificial gravity
- Each ring contains multiple biomes (the ship is like a tiny world)
- The ship has a central axis with elevators
- There's a deflector/shield at the front (for protection against interstellar dust)
- Propulsion is at the rear — fusion drive, with a large fuel tank
- The ship is described as having a "whale" shape, or a long cylindrical spine with rings
- The ship's AI is a character
- Traveling at about 0.1c (10% of light speed)
- The ship is about 1 km long? Let me try to recall… I think the ship was described as being several kilometers long. The rings are about 2 km in diameter? I don't remember the exact numbers.

Actually, in the novel, the ship is described thus: two rotating rings, one forward and one aft, connected by a long central axis/spine containing elevators, cargo holds, and machinery. The front has a shield/deflector dish. The rear has engines/fuel tanks. There are also "seeds" or landing vehicles.

Since I can't actually browse, I should be honest about that while leveraging my knowledge of Kim Stanley Robinson's "Aurora" (the most famous sci-fi novel called "Aurora"). The user says "search and read" — I can't do live search, but I have knowledge about this novel. I should briefly note that I'm basing the model on the description from Kim Stanley Robinson's "Aurora" (2015).

Actually, let me think: could it be a Chinese sci-fi novel called "Aurora"? There might be one. But the most famous is Kim Stanley Robinson's. Given the context (spaceship model, deep space voyage, interior exploration), the generation ship from "Aurora" fits perfectly.

Key features of the ship from Kim Stanley Robinson's "Aurora":
1. **Two rotating habitat rings** (toruses) — each about 2 km in diameter? Actually, I recall the rings being about 1 km in radius. The rings rotate to produce 1g. Each ring is divided into biomes — agricultural areas, forests, lakes, and even a beach/ocean biome.
2. **Central spine/axis** — a long non-rotating (or partially rotating) structural axis connecting the rings, containing elevators, cargo bays, workshops, and the ship's machinery.
3. **Forward deflector shield** — a large dish or magnetic shield protecting against interstellar dust at relativistic speeds.
4. **Aft propulsion** — fusion rockets with enormous fuel tanks (water/ice tanks). I recall "tanks" being described as large cylinders.
5. **The ship's AI** — distributed throughout the ship.
6. **Landing vehicle/"seed"** — a lander used on the destination planet.

From the novel (my memory): the ship left Earth in 2545, destination is a moon of a planet orbiting Tau Ceti… no wait, destination is a planet/moon in the Tau Ceti system — the moon is called "Aurora" (orbiting a gas giant). Travel time about 160 years. Ship speed peaks at about 0.1c.

Description of the ship in the novel: "the ship" — often described as a long spindle with two rotating rings. The rings are called "rings." Inside the rings, the biomes curve upward (classic O'Neill cylinder interior). There's an axis elevator going from hub to ring. The front shield is described as an umbrella/parasol-like structure. The rear has engines and a large radiator.

I'll build a Three.js-based 3D experience:
- Deep space background: starfield, nebulae, distant sun, dust particles streaking past (to convey motion at 0.1c)
- Ship model built procedurally with Three.js geometry:
  - Central spine (long cylinder with truss details)
  - Two rotating torus rings with habitat windows, biome lights
  - Forward shield dish (concave disk with layered structure)
  - Aft engines with glowing fusion plumes, radiators
  - Fuel tanks
  - Antennas, sensors, docking ports
- OrbitControls for 360° rotation, zoom
- Interior mode: switch to interior view — explore the interior of the ring (curving biome landscape, upward-curving terrain), the spine elevator shaft, bridge/AI core
- UI: HUD with ship info, mode toggle, section selection buttons (teleport to sections), labels
- Dynamic effects: ring rotation, engine glow pulsation, blinking navigation lights, streaking dust particles, subtle camera shake, star parallax

Tech choice: Three.js (via CDN import map, ES modules). Single HTML file for portability. OrbitControls from three/examples. Interior mode can be achieved by swapping scenes or teleporting the camera inside a separate interior scene. Simpler: build the interior as a separate scene/group and switch.

Interior design:
1. **Ring biome interior** — inside a torus: curved floor, sky above (opposite side of the ring visible overhead), trees/fields as simple geometry, sunlight strips along the ring's interior (mirror/light panels). This is the most iconic visual. Can build a section of the torus interior: curved floor plane bending upward in the distance, gradient sky, rows of crops, water channels.
2. **Axis hub / elevator** — cylindrical shaft with lights.
3. **Bridge / AI core** — a room with holographic displays.

Given the complexity, I'll make the interior a separate THREE.Scene with its own controls (first-person WASD + mouse look, or simple orbit). To keep it robust, use pointer-lock or drag-look + WASD movement. Simpler: drag to look, WASD to move, that's enough.

Actually, to keep the code manageable but rich, I'll implement:
- Exterior scene: OrbitControls, ship group with animations.
- Interior scene: three selectable locations (ring biome, spine axis corridor, command/AI core). First-person controls: WASD/arrow keys + mouse drag look. Buttons to jump between locations.

UI design: since this is a sci-fi HUD, I'll make it feel like the ship's AI interface (the novel's narrator is the ship's AI! a lovely thematic touch — the UI can speak as the ship's brain). Use a distinctive display font + readable body font. Fonts: since content is Chinese, use Google Fonts Chinese-supporting fonts: "Orbitron" for Latin numerals + "Noto Sans SC" for body, and maybe "ZCOOL QingKe HuangYou" for display Chinese headings. ZCOOL QingKe HuangYou is a nice tech-style Chinese display font available on Google Fonts.

Color palette: deep space navy/black, aurora teal/green + warm amber accents (aurora colors — green/teal with hints of magenta). Aurora theme fits: green-cyan glow.

Structure of deliverable:
- Single self-contained HTML file? Three.js needs to come from CDN. Use import map with three@0.160.0 from unpkg/jsdelivr. That's standard.
- Provide run instructions: just open the HTML file (needs internet for CDN), or serve with a local server.

Note on "search and read": I should be transparent — I can't browse live, but I know the novel (Kim Stanley Robinson's "Aurora," 2015, Chinese translation "Aurora" published by Readom/Sichuan Science and Technology Press? Actually, the Chinese edition "Aurora" was published by Chongqing Press? I don't fully remember. I'll keep the reference brief and focus on the ship's canonical description). The user asked to search — I'll briefly state that I'm basing it on the novel's text description, and summarize the ship's canonical features I've reproduced. Don't overclaim exact dimensions; the novel gives some numbers: ship length about 1 km? Hmm. Let me recall: in the novel, the ship is… the rings are about 2 km in diameter? I recall the ring diameter being about 2 km, the ship's overall length being several km. I'll present approximate numbers as "novel setting" with some hedging ("about").

Actually, let me try to recall more specifically from "Aurora": the ship is described as "a small world" — two rings, each with 12 biomes? I recall biomes like "County Donegal," "Brittany," "Kerala," "Finmark" — the biomes are named after Earth regions. There are 24 biomes total (12 per ring?). The rings are connected by the axis, and the axis has "zero-g" areas. The ship rotates… do the rings counter-rotate? I think the two rings counter-rotate to cancel gyroscopic effects. Front shield: the ship has a "shield" or "deflector" at the front — a big ice block? Hmm, I recall Whipple shield / magnetic field. Also, the ship has a "sail" for deceleration? Actually, in the novel the ship uses a laser sail for launch? No — the ship is fusion-powered, and… I recall there being a big structure. Let me stick with generally accepted descriptions: two counter-rotating habitat rings, central spine, forward shield, aft fusion drive, radiators, fuel tanks.

I'll add an info panel with annotations of the ship's specs and parts, and clickable hotspots/labels (CSS2D-style via screen-space projection) to identify parts: shield, ring A, ring B, spine, engines, tanks, radiators, hub, lander bay.

Dynamic effects:
- Rings slowly counter-rotate
- Engine plume flicker (shader or sprite scale/opacity noise)
- Blinking navigation lights
- Star streaks/dust particles passing by to convey motion
- Subtle ship drift/bob
- Aurora-like nebula background (canvas-generated texture or shader plane)
- Tau Ceti destination star growing brighter ahead? Nice touch: bright star ahead.

Interaction:
- OrbitControls: rotate, zoom (min/max distance), pan maybe disabled.
- Buttons: exterior overview / ring biome interior / central axis / command core.
- Interior: WASD movement + mouse drag look; on-screen hints; return button.
- Part info: click part or hover to display description panel.
- HUD: speed 0.10c, distance traveled, day counter (ship time), rotating ring RPM, etc.

Let me design the ship geometry carefully:

Overall length ~200 units (scene units). Layout along Z axis (nose toward -Z forward… let's say forward = -Z, engines at +Z).

Components:
1. **Forward shield (deflector)**: at z = -90. Large shallow cone/disk facing forward, radius ~14, layered: outer dish (slightly concave, made with LatheGeometry), inner support struts, maybe hex panel texture via canvas. Add faint emissive shimmer (dust impact flashes — small random sparks on shield! nice dynamic detail: random small flash sprites on shield surface).

2. **Shield mast**: connects shield to spine.

3. **Central spine**: cylinder from z=-80 to z=+70, radius ~3, with truss rings, docking modules, tanks. Composed of segments: forward sensor section, midsection with tanks and radiators, aft engineering section.

4. **Habitat rings**: two toruses at z = -35 (ring A) and z = +25 (ring B). Torus major radius ~26, tube radius ~4.5. Each ring connects to spine via 4 spokes (cylinders). Ring surface: hull panels + window band (emissive strip with window texture from canvas). Interior biome glow: warm green emissive band. Counter-rotate around Z axis.

To make the rings look like habitats: main torus (metallic dark hull), plus a slightly larger partial torus for window band (emissive), plus panel lines. Use canvas texture with window grid for emissive map.

5. **Spokes**: 4 per ring, cylinders from spine to ring, radius 0.8, plus small hub collars.

6. **Fuel tanks**: cluster of cylinders/spheres around spine near aft: e.g., 6 cylindrical tanks around spine at z=45..60, radius 2, length 12, arranged hexagonally. Or big spherical tanks. The novel mentions water tanks as shielding/radiation protection. Make it a cluster.

7. **Radiators**: flat glowing panels extending from spine near engines: 2-4 large thin boxes with emissive red-orange gradient, angled.

8. **Engines**: at z=+70..+85: cluster of 3-4 fusion thruster bells (cone/lathe), with glowing throats, plus plume: additive shader cone/plane sprites extending backward, flickering. Also magnetic nozzle rings.

9. **Lander/seed**: small shuttle docked in a bay near mid-spine — a small wingless lifting body shape.

10. **Antennas/sensors**: dish near forward spine, long booms, blinking lights.

11. **Running lights**: small emissive spheres at ring extremities and spine ends, blinking (red/green nav lights).

Materials: MeshStandardMaterial with envMap? Use a simple PMREM from generated environment (RoomEnvironment) for nice metal reflections. three's RoomEnvironment + PMREMGenerator works well. Scene background: custom starfield (Points) + nebula sprites.

Lighting: key directional light (distant star), faint fill, engine point light (flickering orange-blue), interior emissive.

Starfield: 2 layers of Points with different sizes/colors; plus streaking "dust" particles moving fast from front to back (to convey 0.1c motion), rendered as short lines (LineSegments with additive material) — regenerate positions.

Nebula: 3-4 large canvas-texture sprites with radial gradient in teal/magenta/deep blue, additive, very low opacity, placed far away. Also a faint aurora ribbon shader plane in the distance? Maybe a big plane with aurora gradient shader — but keep it subtle. The novel's destination is Tau Ceti; I'll place a bright warm star ahead with a glow sprite.

Post-processing: UnrealBloomPass would be great for engine glow and windows. Use EffectComposer from three examples. Adds imports but worth it. Use import map with three/addons/.

Interior scenes:

**A. Ring biome interior (ring biome)**:
Simulate inside of torus: floor curving upward. Build a long curved "valley": floor is a section of cylinder interior (radius = ring major radius ~260 in interior scale? Use a separate scale for interior). Simplify: build a straight corridor with floor curving upward at both ends using a bent plane geometry (custom geometry: plane bent along a circle). Place low-poly trees (cones + cylinders), crop rows (boxes), water channels (reflective plane), habitat pods on the sides. "Sky" above: opposite side of ring — render as a hazy band with tiny details far overhead? Simplify: gradient sky dome (large sphere with shader gradient from warm horizon to deep blue zenith), plus a bright "sunline" — in the novel, lighting comes from… the rings are enclosed; light from axial windows/mirrors? I'll add a bright strip light along the ceiling.

Simpler robust approach: interior is a large cylinder tube (inside of torus tube, radius ~8, length ~120, slightly curving). Camera inside tube; floor flattened with terrain; curved walls; ceiling with light strips. Add gentle bend by curving the tube: build custom geometry with TubeGeometry along a circular arc, with BackSide material. Floor: strip along bottom of tube with grass texture (canvas), fields, trees. This reads as "inside a rotating ring habitat."

Implementation: create an arc path (CircleCurve) with radius R=120, arc ~50°, TubeGeometry radius 9, tubularSegments 100, radialSegments 24, material BackSide with hull panel texture + light strip (emissive line along top). Floor: another geometry — a ribbon along the bottom inside of the tube: custom BufferGeometry, or flattened TubeGeometry? Easier: place a curved plane (bent along same circle) at the bottom of the tube, width ~10, with grass/farmland canvas texture. Then scatter trees/crops along it. Water channel strip. Distant haze: fog.

Player walks along arc path (WASD: W forward along path, A/D strafe, mouse look). Constrain position to stay near floor: y = floor height. Simple.

**B. Central axis (zero-g axis corridor)**: cylindrical corridor with ribs, lights, cargo, elevator car. Zero-g feel: floating cargo boxes slowly rotating, drifting dust particles. Player can fly (WASD + R/F up/down).

**C. Command core / AI core**: room with holographic star map (points + lines), screens (emissive planes with canvas texture showing telemetry), central AI column with pulsating light. The novel's AI is beloved; label it "ship brain."

Interior controls: implement a simple FlyControls-lite: track keys, drag look (pointerdown/move rotates yaw/pitch), per-frame update. Touch: maybe skip, but at least look-drag works.

Mode switching UI: bottom-left panel with buttons: exterior overview / ring biome / central axis / command core. Also "return to exterior." On switch: fade overlay, swap active scene + camera + controls.

Part labels in exterior: implement screen-space labels: array of {object, name, desc, offset}; each frame project to screen, position HTML divs. Click label → open info card. Hotspot dots too. Keep labels toggleable.

Info panel content (novel-based):
- Forward deflector shield: protects the ship from interstellar dust and gas at 0.1c.
- Habitat ring A/B: counter-rotating, providing 1g artificial gravity via rotation; inside are biomes named after Earth's landscapes, with lakes, farmland, forests.
- Central spine: ship's backbone, zero-g transport trunk line, containing elevator, pipes, cables.
- Fusion engine cluster: aft, continuously accelerating/decelerating.
- Fuel/water tank cluster: reaction mass and radiation shielding.
- Radiator panels: dissipate waste heat from the reactor.
- Seed lander: for landing on the satellite "Aurora."
- Ship brain (AI): manages all ship systems, narrator of the novel.

HUD top: ship name "AURORA · interstellar ark," speed 0.100c, voyage day counter (ship time), distance traveled, rotating ring RPM, etc.

Now fonts: Google Fonts: Orbitron (display Latin), Noto Sans SC (body), ZCOOL QingKe HuangYou (display Chinese). Load via <link>. If offline, fallback to system fonts — fine.

Palette: space #05070f; aurora green #6ff2c0 / #38e8b0; cyan #7fd8ff; warm amber #ffb35c for habitat windows; alert coral #ff6b5e. UI panels: dark translucent with 1px glowing border, corner accents (clip-path notches) — sci-fi HUD style, not full glassmorphism.

Code structure (single index.html):
- <head>: fonts, styles.
- <body>:
  - #app canvas container
  - HUD: top bar (title, telemetry), left panel (mode buttons), right info card (part info), bottom hint bar, label layer, fade overlay, loading screen.
- <script type="importmap">: three + addons from unpkg (pin to 0.160.0). Use jsdelivr: https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js and addons path.
- <script type="module">: main code.

Key implementation details:

Import map:
```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
  }
}
</script>
```

Imports: OrbitControls, EffectComposer, RenderPass, UnrealBloomPass, OutputPass.

Renderer: antialias, ACESFilmicToneMapping, exposure ~1.1.

Exterior scene setup:
- scene.fog: none (space).
- Environment: PMREM RoomEnvironment for reflections, scene.environment = env; but keep background black.

Ship group hierarchy:
```
ship (Group) — slow bob
 ├─ spineGroup
 ├─ ringA (Group at z=-35) — rotates +z
 │   ├─ torus hull, window band, spokes (spokes connect to spine — but spine doesn't rotate; spokes should rotate with ring; visually spokes attach to a hub bearing that rotates — fine, spokes rotate with ring, hub collar on spine is static)
 ├─ ringB (Group at z=+25) — rotates -z
 ├─ shieldGroup (z=-88)
 ├─ engineGroup (z=+78)
 ├─ tanksGroup
 ├─ radiatorGroup
 ├─ lander
 ├─ antennas, lights
```

Ring construction details:
- TorusGeometry(26, 4.2, 24, 120) rotated so its hole faces Z (torus by default is in XY plane with hole facing Z — good, ring plane perpendicular to spine axis Z. Yes, TorusGeometry lies in XY plane, axis = Z.
- Hull material: color #9aa7b8 metalness .85 roughness .35, with panel texture? Use canvas texture with subtle lines as roughness/map. Keep it simple: color + emissive window band as separate torus: TorusGeometry(26, 4.35, 8, 120, ...) only outer half? Window band: thin torus of radius 26, tube 4.26 but scale-flattened? Better: create window strip as torus with emissive map: canvas with window dots; apply to full torus but emissiveMap only shows windows. Use same torus geometry slightly larger (tube 4.28) with material transparent? Overlapping surfaces cause z-fighting; offset tube radius slightly (4.3) — windows appear as raised band. OK.

Actually, cleaner: build ring as: main torus (hull), plus 2 "window band" toruses of radius 26 tube 0.55 offset above/below? Hmm. Let me do this: main torus tube 4.2; window band = torus tube 4.32 with emissiveMap of window grid, but material.polygonOffset to avoid z-fighting? Easiest: make window band tube radius 4.35 so it protrudes 0.15 above hull — appears as a glazed band around ring. emissiveMap canvas: dark base with warm window pixels in horizontal strip rows. Set material.color dark, emissive warm, emissiveMap texture, emissiveIntensity 1.6. Bloom picks it up.

Also add "biome glow" band: another thin torus of tube 0.35 at outer equator with green emissive? Maybe skip; window band suffices. Add ring end details: small boxes around ring as module pods (InstancedMesh of boxes placed around circumference with random slight offsets) — adds detail. Use InstancedMesh of 60 pods per ring.

Spokes: 4 per ring at 45° offset; cylinders from spine radius 3 to ring inner radius ~21.8. Cylinder along… compute: spoke from (0,0,zRing) to point on ring center circle (cos a*26, sin a*26, zRing). Length ~23, oriented in XY plane. Use CylinderGeometry rotated: create cylinder along Y, then orient with quaternion setFromUnitVectors(Y, dir). Add collar torus at hub.

Spine: main cylinder radius 2.6, length 150 (z from -78 to +72), rotate 90° around X so cylinder axis is along Z (CylinderGeometry axis is Y; rotateX(PI/2)). Segmented with ring ribs (torus of radius 2.8 tube .25 every ~10 units) — InstancedMesh. Also 2 thicker sections (cylinder radius 3.4 length 12) at ring planes as hub bearings. Forward sensor mast: cone + dish (lathe or partial sphere) at z=-70 offset. Docking ring near z=-55.

Tanks: 6 cylinders radius 1.9 length 14 arranged hexagonally around spine at radius 5.5, z center 48, with spherical caps (use CapsuleGeometry? CapsuleGeometry exists in three r160: CapsuleGeometry(radius, length, capSegs, radialSegs)). Capsules look like tanks. Material pale metal with orange stripes? Canvas texture stripes — keep color plain + slight emissive none. Add piping: thin cylinders connecting to spine.

Radiators: 2 large panels: BoxGeometry(0.15, 18, 26)? Orientation: extend laterally (X) and slightly angled; place at z=60, one +Y one -Y? Classic look: two flat panels extending vertically. Material: MeshStandardMaterial emissive #ff5a3c emissiveIntensity .8, color #3a1f1c, with gradient emissiveMap canvas (hot near spine, cool at tips). DoubleSide. Add thin support arms.

Engines: cluster: central large + 3 around? Make 4 engines in 2x2 arrangement around spine end at radius 3.2, plus central spine cap. Each engine: lathe nozzle (LatheGeometry profile flaring out), radius .6 to 2.2, length 6, dark metal with emissive throat disk inside (circle geometry, emissive #7fd4ff hot blue-white). Plume: for each engine, additive cone (ConeGeometry open) length ~30 flaring backward, ShaderMaterial or MeshBasicMaterial with alphaMap gradient canvas, blending additive, depthWrite false, color cyan-blue. Animate scale/opacity flicker with noise. Plus a glow sprite at nozzle exit.

Engine light: PointLight cyan at aft, intensity flicker.

Shield: LatheGeometry dish: profile from center (0,0) to rim (14, 2.5) with slight curve — concave facing forward (-Z). Material: dark metal front? Two materials: outer face metallic dark #4c5668 with faint hex texture; back with support ribs (radial boxes) + central mast to spine. Add faint emissive edge ring (torus radius 14 tube .12 emissive teal). Dust impact flashes: pool of small sprites at random positions on dish face, flash quickly (additive), spawn every ~0.3-1s.

Lander ("seed"): at z=-5, docked below spine (y=-4.5): body = capsule scaled, plus small canopy emissive, landing legs? Docked cradle. Keep simple: capsule + box fins + small window band.

Antennas: forward: thin cylinder boom along -Z length 12 from spine tip, small dish (SphereGeometry phi segment or LatheGeometry) at tip. Side booms with sensors.

Navigation lights: small spheres r .18 emissive red (port side? In space convention: red on left/port, green on starboard) at ring outer edges x=±26 at each ring, plus white strobes at spine ends, blinking via emissiveIntensity pulse (store material refs, pulse with time function: strobe pattern).

Starfield:
- Distant stars: 3000 points, sphere radius 1500-2500, size 1-2.5, colors white/blue/amber mix, PointsMaterial sizeAttenuation false? Use sizeAttenuation true with size ~2 at that distance… simpler: 2 point clouds with sizeAttenuation false and sizes 1.5 and 2.5, vertexColors.
- Dust streaks: 400 line segments: each a pair of points (p, p + dir*len) with dir = +Z (moving toward camera/backward relatively). Move z += speed*dt; wrap when past camera. LineSegments with additive material color #9fd8ff opacity .35. Place within a box around ship (x,y within ±80, z within -200..+200). Speed ~180 u/s to feel fast but not overwhelming. Slight parallax with camera.
- Nebula sprites: generate canvas 512 with radial gradient blob (teal #1b4d5c, magenta #4a2a55, deep blue), 5 sprites scale 800-1400, opacity .12-.2, additive? Normal blending with low opacity to avoid blowout; use additive with very low opacity. Place far (z -1200.., sides).
- Tau Ceti destination: bright sprite ahead (-Z far): warm white glow sprite scale 60 at (0, 40, -1600) + point? Plus lens flare-ish cross. Sun behind: small warm directional light from forward-upper? Key light direction: from forward-upper-left to nicely light shield and rings: DirectionalLight position (-150, 120, -200) targeting origin, intensity 2.2 color #eaf4ff. Rim from behind engines: faint blue point light.

Bloom: UnrealBloomPass(resolution, strength .85, radius .6, threshold .8)? Threshold .8 so only emissives bloom; windows ~1.6 emissiveIntensity bloom nicely; engine plumes additive bright. Set renderer.toneMapping ACES; composer with OutputPass for correct color.

Camera exterior: PerspectiveCamera fov 50, start position (70, 34, 120) looking at ship center (0,0,-5). OrbitControls: enableDamping, minDistance 18, maxDistance 420, target (0,0,-5). autoRotate? Off by default; maybe a toggle "auto orbit." Add double-click on part to focus? Keep: click on label focuses camera (tween target). Simple tween via lerp in animate.

Labels: array of {pos (Vector3 in ship space), title, desc}. Each frame: project to NDC → screen px; hide if behind camera (z>1) or occluded? Skip occlusion (costly); fine. Label element: dot + text. Click → opens info card on right with title + description + maybe stats. Also "focus" button in card → tween controls.target and camera to preset view for that part.

Interior scenes: create one interior scene? Or separate scene per location to control content/lighting. I'll make 3 scenes (sceneBiome, sceneAxis, sceneCore) each with own camera rig. Shared interior camera (fov 70). Movement controller class.

Biome interior construction:
- Arc path: circle radius R=140 centered at (0, R, 0)? Floor at y=0: center at (0, 140, 0)?? If tube centerline is circle of radius R in vertical plane (Y up, path along Z curving upward): parameterize: center C=(0,0,0)?? Let me define: path is circle in YZ plane: P(θ) = (0, R·(1-cosθ)?? hmm.

Simpler: path curve: THREE.CatmullRomCurve3? Use custom: for θ in [-θmax, θmax], point = (0, R - R·cosθ, R·sinθ) → at θ=0: (0,0,0), tangent along +Z, curves upward as |θ| grows (y = R(1-cosθ) ≥ 0). This makes floor rise ahead and behind — classic ring interior look. R=160, θmax=0.55 rad → arc length ≈ R·2θmax ≈ 176 units, rise at ends = R(1-cos .55) ≈ 160·0.147 ≈ 23.5 — nice gentle rise.

Tube: TubeGeometry(curve, 120, 9, 20, false), material BackSide, hull texture with panel grid canvas + faint emissive ceiling strip? Ceiling strip separately: another TubeGeometry with smaller radius? Strip: build a ribbon geometry along top interior of tube: sample curve, at each sample compute top point = pos + up_local*8.6, where up_local = direction from tube center? Tube cross-section orientation: TubeGeometry uses Frenet frames; ribbon placement is approximate: top of tube = curve point + normal*? For our curve in YZ plane, binormal is X (constant), normal is in YZ plane. "Up" inside tube (away from ring center) = direction from ring center axis… ring center is at (0, R, z?) hmm: our path curves upward, so ring center is at (0, R, z)?? Circle center: point (0, R, 0) is equidistant? P(θ)=(0, R - R cosθ, R sinθ): distance from (0,R,0): sqrt((R cosθ)² + (R sinθ)²)=R ✓. So ring center O=(0,R,0). Outward direction at P = (P - O)/R = (0, -cosθ, sinθ). Hmm, that's downward-forward… wait, outward from ring center means away from O: dir = (P−O)/R = (0, −cosθ, sinθ). At θ=0: (0,−1,0) = downward. So the "floor" (outward side, where artificial gravity pushes you) is at the bottom of the tube: floor point = P + (0,-1,0)*? Gravity pushes outward = downward at θ=0. So floor is at bottom of tube, "sky"/ceiling is toward O (upward). Good: floor = P + outward*(tubeR - 1.5)?? Floor surface is at radius ~7.5 below tube centerline: build ribbon: width 8, along curve, at offset (0,-cosθ, sinθ)*7.2, oriented with normal pointing inward-upward (toward tube center). Construct custom BufferGeometry: sample N=140 points, at each compute frame: tangent T = derivative normalize ≈ (0, sinθ, cosθ); outward O_dir=(0,−cosθ, sinθ); side = T × O_dir = X axis (1,0,0) or -X. Floor vertices: center F = P + O_dir*7.2; left = F + X*4, right = F − X*4. Normal = −O_dir (upward into tube). Build triangle strip. UV: u along length (0..N), v across. Texture: farmland canvas: green patches, field rows, paths. Repeat along length.

Walls: tube itself (BackSide) with hull panel texture; add emissive "sunstrip": a thin ribbon along top interior (offset toward O direction: P − O_dir*8.4), emissive warm white, gives light. Plus actual lights: several PointLights along arc (3-4) warm; ambient hemisphere.

Props along floor: at intervals: trees: cone (foliage) + cylinder trunk, random size, placed at lateral offsets ±(1..3.2); crop rows: thin boxes with green emissive-ish color; water channel: long thin plane along center section with animated shimmer (emissive cyan, opacity pulse) — maybe a pond: circle plane. Habitat pods: few white domes (hemispheres) with windows. Also distant haze: scene.fog = new FogExp2(0x0a1420, 0.006) to fade rising ends — atmospheric.

Overhead: opposite side of ring: add faint band high above? Our tube interior already implies that; skip. Maybe add small distant detail lights along the arc above? Actually, opposite side of ring would be overhead across the ring diameter (huge distance). In our tube view, we only see tube interior — that's accurate for a closed torus habitat (you can't see across). But the novel's ring has… enclosed torus, yes. Fine.

Player in biome: walk along curve parameter s (arc length), lateral offset x. W/S change s, A/D change lateral (clamp ±3.4), camera y = floorY + 1.7. Mouse drag adjusts yaw/pitch freely. Compute position from θ = s/R. Provide gentle head bob? Skip.

Axis corridor interior:
- Long cylinder tube radius 4, length 160, along Z, BackSide, rib rings (torus every 8), light strips (emissive boxes along sides), floating cargo: 12 boxes with random slow rotation/drift (animate), dust motes (Points), end doors (circle with emissive ring). Zero-g: free fly: WASD relative to camera facing, R/F or Space/Ctrl up/down, clamp within tube (radius 3.2, z ±75). Slow drift damping for floaty feel.

Command core interior:
- Room: cylinder radius 7 length 18 along Z (or a sphere). Central AI column: cylinder with pulsating emissive teal core + rotating rings (gyroscope-like torus). Holographic star map: Points sphere + lines (route Earth → Tau Ceti) rotating slowly, additive. Console desks: boxes with emissive screens (canvas texture with fake graphs/text, update? Static canvas fine, maybe animate one with scanline via texture offset). Windows: large trapezoid window showing stars (a plane with star texture + parallax? Just emissive dark with star dots). Chairs: simple. Lights: teal point light from column, warm console glow.

Interior transition: overlay fade (div opacity), swap renderer scene/camera, enable interior input, update HUD mode text, show back button.

Also add subtle audio? No — skip (autoplay issues).

Performance: moderate geometry; fine.

Resize handler, pixelRatio clamp 2.

Loading screen: since all procedural, load is fast; show brief boot screen with title and "initializing ship brain…" then fade. Nice polish.

Now UI text (Chinese). Title: "AURORA · 极光号" — interstellar generation ship. HUD telemetry:
- Speed: 0.100 c (29,979 km/s)
- Voyage: day counter incrementing (ship time day 58,4xx — 160 years ≈ 58,400 days; start at day 51,233 mid-journey?) Novel: launched 2545, arrives ~2700s. Display "Ship time year 137 · Day xxx" ticking.
- Ring rotation: 1.9 rpm (for 1g at radius 26m? Real radius is ~1000m; our model scale is arbitrary — display "1.8 rpm").
- Hull temperature, shielding status: nominal.
- Destination: Tau Ceti e? In novel: destination is a moon orbiting a gas giant in Tau Ceti, colonists name it… they call the planet "Aurora"? Actually, the moon is called "Aurora"? I recall the moon's name was Aurora? The starship is named after…? Ship unnamed ("the ship"), destination moon is Aurora? Hmm. In the novel: destination is a moon of a gas giant in Tau Ceti; the moon is called Aurora? I think the moon is called Aurora — the title refers to the destination. Some sources: the planet is Aurora? Let me phrase: "Destination: Tau Ceti · Aurora satellite" — hedged: "Destination: Tau Ceti system · Aurora." I recall the moon Aurora orbits a gas giant… anyway, phrase as "Tau Ceti · Aurora."

Also note in info/intro: based on Kim Stanley Robinson's "Aurora" (2015). Add a small "About this model" card: description of novel and reproduced features, plus honest note: the novel doesn't give complete engineering drawings, model reconstructed from text description (two counter-rotating habitat rings, central spine, forward shield, aft fusion engines, etc.).

Part info data (title, subtitle, desc, focus camera pos):
1. Forward deflector shield — at 0.1c, even dust grains carry enormous kinetic energy; the shield is the ship's first line of defense.
2. Habitat Ring A / B — counter-rotating to cancel gyro; inside, Earth biomes named after Earth regions (farmland, forests, lakes).
3. Central spine — zero-g trunk corridor, elevator, pipes, and cables run through it.
4. Fusion engine cluster — continuous low-thrust acceleration for decades, then flip and decelerate.
5. Water/fuel tank cluster — reaction mass for the engines, also serves as radiation shielding.
6. Radiator panels — radiate reactor waste heat into space, glowing dull red.
7. Seed lander — ferries colonists between the ship and the surface of Aurora.
8. Ship brain — the AI that runs the ship and narrates the novel.

Mode buttons: exterior overview / ring biome / zero-g axis / command core.

Controls hint bar: exterior: left-click drag rotate · wheel zoom · right-click pan; interior: WASD move · drag to look · (axis: Space/C rise/descend? Use R/F) — keep: WASD move · mouse drag look · Q/E up/down.

Let me write the movement controller:

```js
class InteriorControls {
  constructor(camera, dom){ this.camera=camera; this.dom=dom; this.yaw=0; this.pitch=0; this.keys={}; this.enabled=false; this.mode='walk'; this.walk=null;
    dom.addEventListener('pointerdown', e=>{ if(!this.enabled) return; this.dragging=true; this.last=[e.clientX,e.clientY]; dom.setPointerCapture(e.pointerId);});
    dom.addEventListener('pointermove', e=>{ if(!this.enabled||!this.dragging) return; const dx=e.clientX-this.last[0], dy=e.clientY-this.last[1]; this.last=[e.clientX,e.clientY]; this.yaw -= dx*0.0032; this.pitch = clamp(this.pitch - dy*0.0028, -1.35, 1.35);});
    dom.addEventListener('pointerup', ...);
    keydown/keyup listeners on window.
  }
  update(dt){ if(!this.enabled) return; camera.quaternion.setFromEuler(new Euler(pitch, yaw, 0, 'YXZ')); compute forward/right from yaw (walk mode: ignore pitch for movement); velocity...
  }
}
```

Walk mode (biome): maintain s (arc position) and lat. Forward key increases s along tangent; but yaw is free… simplest: W moves in camera's horizontal projected direction; then project resulting position to curve parameter: since corridor is roughly along Z, just move x,z freely and clamp: lateral |x| ≤ 3.4; z within arc range, compute floorY from z: from z = R sinθ → θ = asin(z/R), y_floor = R(1-cosθ) − 7.2 + ? Wait, floor surface y at z: floorY(z) = R − R cosθ − 7.2, θ=asin(z/R)… hmm, P(θ) y = R(1−cosθ), z = R sinθ. Floor is 7.2 outward: floorY = R(1−cosθ) − 7.2. At z=0: floorY = −7.2. So set camera base y accordingly: place everything so floor at z=0 is y=0: offset whole biome group by +7.2 in y? Easier: build geometry with offset: use P'(θ) = (0, R(1−cosθ)+7.2, R sinθ) for tube centerline? Then floor at θ=0 is y≈0. Let me define curve points with y offset +7.2. Then floorY(z): θ=asin(clamp(z/R)), y = R(1−cosθ)+7.2 − 7.2 = R(1−cosθ). Wait, floor = centerline − 7.2 → floorY = R(1−cosθ). At z=0 → 0 ✓, rises at ends ✓.

Camera y = floorY(z) + 1.7. Movement: velocity in XZ from keys and yaw; clamp z to ±(R sin θmax − 2), x to ±3.4. Works, robust, no curve bookkeeping.

Free fly (axis, core): velocity along camera forward (full 3D), right, up (Q/E), clamp position within bounds; add damping: vel *= (1 - 3*dt) style, accelerate with keys.

Interior scene camera: same camera object reused; on enter, set position and yaw/pitch.

Renderer renders active scene with active camera; composer: RenderPass references scene/camera — update composer passes on switch: renderPass.scene = activeScene; renderPass.camera = cam. UnrealBloom stays. Should work.

Fog: biome scene fog; axis/core maybe faint fog. Exterior no fog. renderer.setClearColor handles per-scene background: scene.background = new Color(0x02040a) for interior; exterior scene.background = black + star points.

Exterior labels: implement as divs in #labels layer with class .tag; each has a dot and name; pointer-events auto for click. Update each frame: get world position (ship is roughly static but rings rotate — anchor labels to static parts: shield, ring A center (anchor to non-rotating hub collar), spine mid, engines, tanks, radiators, lander, AI core position at spine forward section). Project with active camera (exterior only). Hide labels when interior mode. Occlusion by ship: skip.

Focus tween: on card "focus" click or label double click: animate controls.target → part position, camera position → preset offset. Implement simple tween manager: store from/to/t; ease in animate.

Auto telemetry updates: voyage days increment (e.g., 1 day per 2 real seconds? Show fractional). Speed display constant 0.100c with tiny flicker ±0.0001. Ring rpm 1.8. Shield integrity 99.7% with jitter. Reactor output GW.

Ship time year: novel journey ~160 years; set current "Voyage year 137" day counter 0..365 ticking.

Now, dust streaks: LineSegments geometry with positions for N=350 segments; store array; each frame z += speed*dt for both endpoints; if z > camZ+50 → reset to far ahead (z -= 500) with new random x,y. Segment length ~6-14. Ship at origin; camera orbits ~100-400 away; keep streaks in box x±140, y±100, z −350..250, wrap length 600. Speed 160 u/s. Direction: dust moves relatively from front (-Z) to back (+Z): positions increase z ✓ (ship moving -Z). Streaks also near shield more? Uniform fine. Also occasional shield impact flash.

Engine plume shader: use MeshBasicMaterial with map = gradient canvas (radial/linear alpha), blending: AdditiveBlending, transparent, depthWrite:false. Geometry: ConeGeometry(2.4, 34, 24, 1, true) translated so tip at nozzle, flares backward: cone default has tip up +Y… use CylinderGeometry(rTop=0.7, rBottom=2.6, h=30, openEnded) with gradient alpha along length via canvas (top opaque → bottom transparent), rotate X to align with Z. Animate material.opacity flicker + slight scale.

Also add big soft glow sprite behind engines.

Shield flashes: pool of 8 Sprite with additive radial texture; on spawn: position at random point on dish face (local), scale .3→1.5→0 over .25s, opacity spike.

Window emissive texture canvas: 1024x128: dark base #0a0e14; rows of small rectangles warm #ffd9a0 with random on/off; also some teal. Wrap around torus: repeat 8x1.

Hull panel texture canvas: 512: base #8f9aab? Draw grid lines darker + subtle noise + a few panel rectangles; use as map with slight color; also roughnessMap? Keep as map + color tint.

Biome floor texture: 512x512 canvas: green base variations, field stripes in different greens/browns, thin paths; repeat (1, 24) along length? Our ribbon UV u along width? Set repeat accordingly.

Ceiling sunstrip: emissive #fff4d6 intensity 2.4.

Axis corridor texture: panels + colored conduit stripes.

Console screen canvas: dark with teal graphs, "AURORA SYS" text.

Star map holo: Points of ~400 points in sphere radius 3 + line Earth(−2,0,0)→TauCeti(2,0.5,1) with midpoint marker; additive teal; group rotates slowly.

AI column: cylinder r .5 h 4 emissive teal pulsing + 2 torus rings rotating on different axes + point light.

Core room: cylinder shell BackSide r 7 length 16 along Z? Make it a disc room: cylinder along Y (radius 7, height 4.5) — a round room; floor with emissive ring inlays; window strip around? Add big viewport on one side showing stars (plane with star canvas, slight emissive). Consoles around perimeter.

OK. Also add a vignette overlay div (radial gradient) for cinematic feel + scanline? Keep subtle vignette.

Intro overlay: full screen, title "AURORA 极光号," subtitle "Kim Stanley Robinson 'Aurora' · interstellar generation ship reconstruction," button "Board" to start (also satisfies user gesture). Fade out. Include brief description + controls list. This is the opening — start with something distinctive: ship AI boot log lines typing out ("Ship brain self-diagnostic… biome systems normal…"). Nice touch: boot sequence text animates then button appears. Keep fast (~2s, skippable).

Right info card: hidden by default; show on label click; close button.

Left mode panel: vertical buttons with icons (unicode/svg). Also "labels" toggle and "auto orbit" toggle.

Bottom center hint bar: contextual controls help.

Top bar: left: title AURORA 极光号 + "interstellar generation ship · KSR-2545"; right: telemetry chips (speed / ship time / ring rpm / shielding).

Fonts: Orbitron for numbers/Latin; ZCOOL QingKe HuangYou for Chinese headings; Noto Sans SC body.

CSS: sci-fi panels with clipped corners (clip-path polygon), thin teal borders, background rgba(6,12,20,.72), backdrop-filter blur(6px) minimal. Corner brackets via ::before. Buttons: hover glow, active state teal fill. Micro-interactions: button hover translate, label pulse.

Edge cases: WebGL fail → show message. CDN fail → loading screen shows error text after timeout? Add <noscript> and note in loading "requires network to load three.js."

Mobile: touch works with OrbitControls; interior drag look works via pointer events; movement keys not available on mobile — add on-screen joystick? Too much; add simple touch: double-tap to move forward? Provide small on-screen arrow buttons for interior on coarse pointer. Add 4 directional buttons shown on touch devices (media (pointer:coarse)) that set key flags. Reasonable.

Now let me write the code. It'll be long (~1300+ lines). Structure carefully.

Key risk areas:
- Import map + addons path correctness (three@0.160.0 examples/jsm/controls/OrbitControls.js etc.)
- Composer imports: 'three/addons/postprocessing/EffectComposer.js', RenderPass, UnrealBloomPass, OutputPass.
- RoomEnvironment: 'three/addons/environments/RoomEnvironment.js'. In r160, RoomEnvironment constructor takes no renderer arg? `new RoomEnvironment()` — pmremGenerator.fromScene(env, 0.04).texture. In some versions RoomEnvironment(renderer)… r160: `class RoomEnvironment extends Scene { constructor() }` I think no arg needed; older examples show `new RoomEnvironment( renderer )`? Let me recall: three r150+ examples: `const environment = new RoomEnvironment( renderer ); pmremGenerator.fromScene( environment )`. Hmm, in r157 webgl_materials_envmaps_exr? The RoomEnvironment source (r160): `constructor()` builds lights and boxes… Actually I recall it was changed in r162+ to accept renderer parameter for `renderer.useLegacyLights`? To be safe, avoid RoomEnvironment: build my own small env scene: black background + a few colored area-ish emissive boxes/planes, PMREM it. Simple and version-robust:

```js
const envScene = new THREE.Scene();
const geo = new THREE.SphereGeometry(50, 16, 16); // big emissive sphere?
```
Better: make gradient env: big sphere BackSide with ShaderMaterial gradient (top deep blue, horizon teal, bottom black) + 2 bright emissive planes (key softbox). PMREM from that. This gives nice reflections on metals.

Actually, simplest robust: `scene.environment = pmrem.fromScene(envScene).texture`, envScene = new THREE.Scene() with: mesh sphere BackSide MeshBasicMaterial vertexColors gradient? Use canvas texture on sphere: 256x128 canvas with vertical gradient (top #0a1a2f, mid #123, bottom #000) + draw bright blobs; map to sphere BackSide. Plus one bright white plane near "sun" direction for specular highlight. Good.

- TorusGeometry orientation: axis along Z ✓ (lies in XY plane). Ring plane should be perpendicular to spine (Z axis) ✓.

- Ring rotation: ringA.rotation.z += speed*dt; ringB -=.

- Spokes inside ring group rotate with ring — but they visually connect to static spine hub: add static hub collar (torus) on spine at ring z; rotating spokes slide against it — acceptable, looks like a bearing.

- Windows emissive band rotating with ring — fine.

- Label anchor for ring: use static hub collar position (0,0,zRing) + offset outward (0, 30?, zRing)? Place label at ring top: (0, 31, zRing) static ✓.

Focus camera presets per part: position = anchor + offset direction.

- UnrealBloomPass with transparent additive plumes: fine.

- OutputPass handles tonemapping/sRGB in composer chain (r160 has OutputPass ✓).

- Interior scene rendering with bloom: bloom threshold .8; interior emissives above threshold will glow nicely.

- When switching scenes, keep single composer; set renderPass.scene.

- Dust streak line material: LineBasicMaterial({color:0x9fd0ff, transparent:true, opacity:.35, blending:AdditiveBlending, depthWrite:false}). linewidth ignored, fine.

- Shield dish lathe: LatheGeometry(points, 48). Points from (0.01, 0) to (14, 3): curve: for t 0..1: r = 14*t, y = 3*t*t (parabolic). Lathe rotates around Y axis; then rotate geometry to face -Z: rotateX(-90°?) Lathe axis is Y; dish opens +Y; we want it to open -Z (forward): rotateX(-PI/2) maps +Y → -Z? rotateX(-90°): Y axis → Z? Rotate -90 around X: Y→ -Z? Rotation of vector (0,1,0) by Rx(θ): y' = y cosθ - z sinθ, z' = y sinθ + z cosθ. θ=-90°: cos=0, sin=-1 → y'=0*… y' = 1*0 - 0*(-1)=0; z' = 1*(-1)+0 = -1 → (0,0,-1) ✓ opens toward -Z. Good: geometry.rotateX(-Math.PI/2). Then position z=-88. Back side (facing ship) visible from behind: material DoubleSide. Ribs: 8 boxes from center to rim on back side (z+ side). Add rim torus.

Dust flashes on dish: dish face is at z ≈ -88 - (0..3)?? After rotation, dish surface extends from center z=-88 to rim z=-88-3=-91 toward -Z? Wait, opening toward -Z means concave face faces -Z; lathe y values 0..3 map to z 0..-3: center at local z=0 (which is aft, z=-88 world), rim at z=-91 forward. Hmm, concave face faces -Z ✓ (front). Flash positions: pick random r≤13, angle: local point (r cos a, r sin a, -(3*(r/14)²)) + group position. Place sprite slightly forward (z -0.2). ✓.

- Engine glow flicker: intensity = base*(0.85+0.3*noise(t)); use sum of sines.

- Ring pod instances: for i in 60: angle = i/60*2π + jitter; position on torus outer equator: radius 26+4.2=30.2 → place box at (cos*30.4, sin*30.4, 0) in ring group, orient facing outward: rotation.z = angle. Box size (1.6, 0.8, 2.2)? Randomize scale slightly. Use InstancedMesh with matrix compose. ✓

- Also add ring inner "dock" ring? Skip.

- Lander cradle: 2 arms from spine.

Numbers check: ring major radius 26, tube 4.2 → outer diameter 60.4. Spine length ~160. Shield radius 14 at z=-88. Ship overall length ~ -91 to +95 (plume) ≈ 186. Camera maxDistance 420 fine.

Ring z positions: -35 and +25 — spacing 60, ring diameter 60: looks balanced. Tanks at z 42..58, radiators at z 55 (panels extending ±Y), engines at z 72..80, plume to +110. Forward: shield -88, mast -80..-70, sensor boom -70..-84 above? Antenna dish near spine top at z=-60.

Wait, radiators and tanks overlap region 42..58; move radiators to z=63, engines 74. ok.

Ring B at +25, tanks start 42 — clearance ok (ring outer z extent is only tube width ~±4.2 → 20.8..29.2).

Lander at z=-8, below spine y=-5.

Hub collars at z=-35, +25 radius 3.6 torus tube 0.8.

Forward section: docking ring torus at z=-58 radius 4.5 tube .6.

AI core location: forward section z=-62 (label anchor there).

Now interior axis corridor: length 150 along z -70..+80? Keep -70..70. Radius 4.

Command core room: place conceptually at forward spine; interior is its own scene anyway.

Biome fog color match background #060a12? Biome interior background: dark blue-green haze: scene.background = color #0b1626; fog same color density .008 → rising ends fade into haze. Add "sky" glow: big soft gradient plane high above? Tube blocks sky; fine.

Biome lights: hemisphere (sky teal / ground green) .5; 4 warm point lights along arc at y≈6 intensity 30 distance 40 decay 2 (r160 physical lights? renderer.useLegacyLights default false in r160 → intensity is candela-ish; point light intensity ~20-60 works with decay 2). Tune: intensity 40, distance 45.

Sunstrip emissive intensity 2.

Exterior directional light intensity 2.5 (r160 non-legacy: directional intensity is "lux-ish" but 1-3 typical). Ambient: use HemisphereLight (space blue / black) 0.25.

Env map gives metals life.

Bloom strength .9 radius .5 threshold .75.

Tone mapping exposure 1.15.

Now telemetry ticker: setInterval 100ms updating DOM: day += 0.05 per tick? Ship day ~ real 1.2s? Let's do: voyageDay increments by 1 per 1.5s, display "Year 137 Day 214" cycling 365. Speed: (0.1000 + sin*0.0002).toFixed(4). Shield 99.6±.2. Reactor 3.42 GW ±.

Ring rpm: for 1g: ω=√(g/r); r real ~1000m → 0.95 rpm? Display 0.95 rpm. Hmm, novel doesn't specify; display "0.9 rpm."

Now info card data and focus presets:

parts = [
 { id:'shield', name:'Forward Deflector Shield', anchor:(0,0,-90), camPos:(28,10,-150), desc:'…' },
 { id:'ringA', name:'Habitat Ring · A', anchor:(0,31,-35), camPos:(55,26,-70) ... },
 { id:'ringB', ... },
 { id:'spine', name:'Central Spine', anchor:(0,4,-10), camPos:(40,14,-10) },
 { id:'tanks', name:'Water and Fuel Tank Cluster', anchor:(6,7,50)?, ... },
 { id:'radiator', name:'Radiator Panels', anchor:(0,16,63) },
 { id:'engines', name:'Fusion Engine Cluster', anchor:(0,0,80), camPos:(36,-8,130) },
 { id:'lander', name:'Seed Lander', anchor:(0,-5,-8) },
 { id:'ai', name:'Ship Brain', anchor:(0,4,-62) },
]

Labels show dot + short name; click → card with details + [Focus View] + [Enter Interior] for ringA (→ biome), spine (→ axis), ai (→ core).

Nice cross-linking.

Now let me write the HTML/CSS/JS. Careful with syntax. Also handle context menu suppression on canvas for right-click pan (OrbitControls handles).

Interior entry points: mode buttons always available.

Transition function:

```js
function setMode(mode){ // 'exterior' | 'biome' | 'axis' | 'core'
  fade overlay in (250ms) → configure → fade out.
}
```

Configure:
- exterior: activeScene=sceneExt; camera fov 50; controls.enabled=true; labels visible; hint text exterior; autoRotate preserved.
- biome: activeScene=sceneBiome; camera fov 70; set pos (0,1.7,0) yaw π (looking along +Z? Look down arc: yaw=0 looks -Z; pick yaw=π so we see arc rising ahead? Our arc rises at both ±z; look toward +Z: with yaw 'YXZ', camera default looks -Z at yaw 0; yaw=π looks +Z ✓). Controls mode 'walk'.
- axis: sceneAxis; pos (0,0,-60) yaw=π (looking +Z down corridor); mode 'fly', bounds {r:3.1, zmin:-68, zmax:68}.
- core: sceneCore; pos (0,1.6,5.5) yaw=π*? Look at central column at origin: camera at (0,1.7,6) looking -Z → yaw 0 ✓. mode 'fly' bounds r 6.2 z ±6.5 y ±? Room height: cylinder along Y radius 7 height 4.6: y -2.3..2.3; camera y clamp 0.6..2.0. Custom bounds per mode: store clamp function.

InteriorControls.update applies per-mode clamp.

Walk mode movement: direction from yaw: forward = (−sin yaw, 0, −cos yaw)?? With Euler YXZ yaw: camera forward = (−sin yaw·cos pitch, sin pitch?, −cos yaw·cos pitch). Horizontal forward = (−sin yaw, 0, −cos yaw). At yaw=π: (0,0,1) → +Z ✓.

right = (cos yaw, 0, −sin yaw)? cross(forward, up)= forward × up = (fz*0? ) compute: f=(−s,0,−c), up=(0,1,0): right = f × up = (0*0−(−c)*1, (−c)*0−(−s)*0, (−s)*1−0*0 ) = (c, 0, −s). At yaw=π: (−1,0,0). Hmm, right should be… looking +Z, right hand is -X? Looking down +Z with Y up, right is… screen right = -X? Cross(up, forward) gives right in right-handed: up × f = (1*(−c)−0, 0−(1*(−s))·? Let me just compute: u×f = (uy*fz−uz*fy, uz*fx−ux*fz, ux*fy−uy*fx) = (1*(−c)−0, 0−(−s)*1·? wait f=(−s,0,−c): u×f = (1*(−c) − 0*0, 0*(−s) − 1*(−c)?? I'm muddling. u=(0,1,0), f=(−s,0,−c). u×f = (u_y f_z − u_z f_y, u_z f_x − u_x f_z, u_x f_y − u_y f_x) = (1·(−c) − 0·0, 0·(−s) − 0·(−c), 0·0 − 1·(−s)) = (−c, 0, s). At yaw=0 (looking −Z): (−1,0,0) → that's left (looking −Z, right is +X? Looking −Z with Y up: +X is… right-handed: X cross Y = Z. Looking down −Z, right is +X? Standard: camera looks −Z, right = +X ✓. So u×f gives left; right = f×u = (c,0,−s). At yaw 0: (1,0,0) ✓ right. At yaw π: (−1,0,0): looking +Z, right = −X ✓ (consistent). Good: right = (cos yaw, 0, −sin yaw).

Fly mode: full forward vector with pitch: f = (−sin yaw cos pitch, sin pitch, −cos yaw cos pitch). Note: pitch positive should look up → forward y = sin pitch ✓ with Euler(pitch, yaw, 0,'YXZ')? Euler order YXZ applies Y then X… camera rotation: quaternion from Euler(pitch, yaw, 0, 'YXZ'): rotation = R_y(yaw)·R_x(pitch). Forward = R·(0,0,−1): R_x(pitch)·(0,0,−1) = (0, sin pitch, −cos pitch). Then R_y(yaw): x' = x cos + z sin = −cos pitch·sin yaw? R_y: (x cos y + z sin y, y, −x sin y + z cos y). With x=0, z=−cos p: x' = −cos p · sin yaw? x' = 0·cos yaw + (−cos p)(sin yaw) = −cos p sin yaw ✓; y' = sin p ✓; z' = −0·sin yaw + (−cos p)(cos yaw) = −cos p cos yaw ✓. Matches. Right vector = R·(1,0,0) = (cos yaw, 0, −sin yaw) ✓ (independent of pitch).

Key mapping: KeyW forward, S back, A left (−right), D right, Q down? Let's do Q/E up/down plus Space/Shift. Hint: "Q/E ascend/descend."

Velocity: target = dir*speed; vel lerp to target with 1−exp(−8dt); pos += vel*dt. Walk speed 6, fly 8 (shift ×2.2 sprint).

Touch buttons: div with 4 buttons (▲◀▶▼) + maybe ascend/descend not needed for walk. Set keys flags on pointerdown/up. Show only on coarse pointer and when interior.

Now labels update: for each part: worldPos (static, since ship group is at origin with only small bob — include bob: use vector temp getWorldPosition from stored Object3D anchor). Project: v.project(camera); if v.z>1 hide; x=(v.x*.5+.5)*w, y=(−v.y*.5+.5)*h. Set transform translate. Also scale/fade with distance? Keep simple, hide when interior or labels off.

Ship bob: ship.position.y = sin(t*.3)*.6; rotation.z = sin(t*.2)*.004; rotation.x small. Subtle.

Auto-orbit toggle: controls.autoRotate = checked; speed .5.

Also add screenshot button? Skip.

Intro boot lines typing: CSS animation with steps? Simpler: JS adds lines sequentially with delays, each line fades in. Then enable button.

Error handling: window.addEventListener('error') → show message in loader? Only for module load failure: add timeout: if !window.__booted after 8s, show error hint in loader (network/CDN). Set __booted=true when init done.

Also add keyboard shortcuts: 1-4 modes, L labels toggle, Esc return to exterior.

Now let me write CSS carefully: distinctive HUD. Colors:
--bg:#04070d; --panel:rgba(8,16,26,.78); --line:rgba(105,220,190,.35); --teal:#5ee8b6; --cyan:#8fd8ff; --amber:#ffb45e; --text:#cfe3e6; --dim:#6d8a93.

Display font stack: 'ZCOOL QingKe HuangYou','Orbitron',sans-serif for headings; numbers: 'Orbitron','Noto Sans SC'. Body 'Noto Sans SC'.

Panel corner cut: clip-path: polygon(0 0, calc(100% - 14px) 0, 100% 14px, 100% 100%, 14px 100%, 0 calc(100% - 14px)); plus 1px border via outline? clip-path cuts border; use background + inset box-shadow? Simpler: border:1px solid var(--line) with clip-path — border will be cut at clipped corners, acceptable aesthetic. Add small corner accent ::before with 2px teal line at top-left.

Buttons: display flex column; each button: index number + icon glyph + label; hover: background rgba(94,232,182,.08), translateX(3px), border-left teal. Active mode: filled.

Top bar: full width, height 56, flex; left title with aurora glyph ◈; right chips: each chip label + Orbitron value.

Bottom hint: centered pill.

Right card: width 300, slide in (transform translateX).

Labels: .tag { position:absolute; transform:translate(-50%,-130%); } dot with pulse ring animation; text small with letter-spacing.

Vignette: pointer-events none radial gradient.

Loader: full black, centered column, title big display, boot log monospace lines, progress shimmer, then button.

Fade overlay for mode transitions: full black opacity 0 pointer-events none, transition .28s.

Now let me write JS. Organize with comments. Roughly:

1. Imports
2. Constants/state
3. Renderer/camera/composer
4. Texture helpers (canvas makers)
5. Env map
6. Exterior scene build (functions per component)
7. Starfield/dust/nebula
8. Interior scenes (biome/axis/core)
9. Controls (orbit + interior)
10. Labels and parts data
11. UI wiring (buttons, card, telemetry, intro)
12. Mode switching
13. Animate loop
14. Resize

Let me write texture helpers:

```js
function canvasTexture(w,h,draw,{repeat=[1,1]}={}){ const c=document.createElement('canvas'); c.width=w;c.height=h; const g=c.getContext('2d'); draw(g,w,h); const t=new THREE.CanvasTexture(c); t.wrapS=t.wrapT=THREE.RepeatWrapping; t.repeat.set(...repeat); t.anisotropy=4; return t; }
```

Windows texture:
```js
const windowsTex = canvasTexture(1024,128,(g,w,h)=>{ g.fillStyle='#05080d'; g.fillRect(0,0,w,h);
 for(let x=6;x<w;x+=14){ for(const row of [34,58,82]){ if(Math.random()<.72){ const warm=Math.random()<.8; g.fillStyle= warm?`rgba(255,${190+Math.random()*40|0},${120+Math.random()*60|0},${.55+Math.random()*.45})`:'rgba(140,230,255,.9)'; g.fillRect(x,row+(Math.random()*4-2),7,9); } } }
});
```
Use as emissiveMap for band torus; color #111, emissive #ffffff, emissiveIntensity 1.5, emissiveMap windowsTex → emissive color white × map color ✓. repeat x 10: set repeat on tex (10,1).

Hull texture: base #7e8b9c with panel lines:
```js
g.fillStyle='#78848f'? then random darker rectangles, grid lines rgba(0,0,0,.25), rivet dots.
```
Use as map for spine/ring hull, color tint per material.

Radiator gradient emissive map: horizontal gradient #ff7a45 → #5a1207.

Plume alpha map: vertical gradient white → black.

Glow sprite texture: radial gradient white center → transparent.

Nebula texture: several radial gradients on 512 canvas, teal/magenta/blue variants (pass hue param).

Floor (biome) texture 512:
```js
base '#1d3a24'; draw vertical strips of various greens/browns (fields) across width; add noise dots; thin path lines.
```
Ribbon UV: u across width (v?), set repeat (1, 26) along length.

Corridor texture: panels with conduit stripes teal/amber.

Screen texture: dark #041018, teal text lines and graphs; draw "AURORA // SHIP MIND" etc.

Star map: points.

Env sphere texture: vertical gradient + bright blob.

Now geometry construction code — write carefully.

Ring builder:

```js
function buildRing(zPos, dir){ const grp=new THREE.Group(); grp.position.z=zPos;
 const hull=new THREE.Mesh(new THREE.TorusGeometry(26,4.2,26,140), hullMat); grp.add(hull);
 const band=new THREE.Mesh(new THREE.TorusGeometry(26,4.34,18,140), new THREE.MeshStandardMaterial({color:0x0c1016, roughness:.4, metalness:.2, emissive:0xffffff, emissiveMap:windowsTex, emissiveIntensity:1.5})); grp.add(band);
```
Wait, band tube 4.34 vs hull 4.2: band protrudes .14 all around — whole ring looks slightly larger glazed sleeve; fine (windows all around). But z-fighting where surfaces nearly coplanar? They don't coincide (different radii) ✓.

Add equator trim: torus radius 26 tube 4.36 but only thin? Skip.

Pods: InstancedMesh(BoxGeometry(2.2,1.1,1.4), podMat, 56):
```js
const m=new THREE.Matrix4(), q=new THREE.Quaternion(), s=new THREE.Vector3();
for i: a=i/56*2π+rand*.05; r=30.5; pos(cos a*r, sin a*r, (rand-.5)*1.5); q from Euler(0,0,a); scale random .8-1.3; compose.
```
Box oriented: after z rotation, box's local Y points radially ✓ (height 1.1 radial). ok.

Spokes:
```js
for k in 4: a=k*π/2+π/4; from=(cos a*3, sin a*3, 0) to=(cos a*22.5, sin a*22.5,0); cylinder r .7 length = |to-from| ≈19.5, positioned at midpoint, quaternion from (0,1,0) to dir.
```
Also add secondary thin pipe spokes offset? Add one more set at a+π/8 with r .25 for detail. Nice.

Ring group rotation animation: ringA.rotation.z += dt*0.12 (rad/s → ~1.15 rpm visually slow), ringB -=. Visual plausibility over realism.

Spine:
```js
const spineMat = hull with texture repeat (4, 12)? Cylinder UV: around x, along y. Set repeat(6,1)?
Main tube: CylinderGeometry(2.6,2.6,150,28,1,true?) openEnded false; rotateX(π/2); position z=-3 (center of -78..72).
Ribs: InstancedMesh(TorusGeometry(2.85,.22,10,36), ribMat, 14) at z -70..64 step ~10.
Hub collars at ring z: TorusGeometry(3.6,.9,12,40).
Forward module: CylinderGeometry(3.4,2.6,10) at z=-63? Taper.
Aft engineering block: CylinderGeometry(3.2,3.8,14) at z 66.
```

Docking ring at z=-52: torus radius 5 tube .5 + 4 small dock boxes.

Shield group at z=-86:
- Dish lathe r 14 depth 3, rotateX(-π/2) → opens -Z? Recompute: geometry.rotateX(-Math.PI/2) maps +Y → -Z ✓ (verified). So dish concave faces -Z ✓ rim forward (more negative z). Center at group origin z=-86; rim at -89.
- Material MeshStandardMaterial({color:0x556275, metalness:.85, roughness:.38, side:DoubleSide, map:hullTex?}) plus faint emissive none. Add hex texture? Reuse hullTex.
- Rim glow ring: TorusGeometry(14,.14,8,80) rotated to face Z (torus axis Z by default ✓) at z=-3 (rim plane) local? Rim is at local z=-3 plane: position (0,0,-3). Emissive teal 2.
- Back ribs: 10 boxes (0.3,0.3,?) from center hub to rim at back (local z>0 side? Back faces +Z (toward ship). Ribs: radial boxes at local z=+0.4: BoxGeometry(13.5,.35,.35) rotated around Z per angle, positioned at half radius. Plus central hub cylinder r 1.6 length 3 along Z.
- Mast: cylinder r .8 from z=-86+? Group local: mast from z=+1 to spine start -78 → world -85..-78: place mast as separate mesh at world z=-81.5 length 8.

Engine group at z=+72:
- Base plate: CylinderGeometry(4.6,4.2,3,24) rotateX(π/2) at z=72.
- 4 engines at radius 3, angles 45,135,225,315: each subgroup at (cos*3, sin*3, 74): nozzle lathe: profile points: (0.55,0),(0.7,1),(1.0,2.2),(1.7,4),(2.3,5.6) → LatheGeometry(pts,28), rotateX(π/2)? Lathe axis Y; want axis Z flaring toward +Z: rotateX(π/2) maps +Y → +Z ✓ (check: (0,1,0) under Rx(90°): y'=cos90*1−sin90*0=0; z'=sin90*1=1 ✓). So nozzle flares toward +Z ✓. Material dark metal, inside emissive? Add throat disk: CircleGeometry(0.62) at local z=0.1 facing +Z? Circle faces +Z by default ✓ emissive #bfe8ff intensity 3.
- Plume mesh per engine at z=+5.6+: CylinderGeometry(0.6, 2.2, 26, 20, 1, true) rotateX(π/2) → axis Z, small r top at -z side? rotateX(π/2): +Y → +Z so cylinder top (rTop, +Y) maps to +Z. Want narrow at engine (−z side) and wide at back (+z): rTop is +Z end → rTop=2.2 wide at back, rBottom=0.6 at engine ✓ so CylinderGeometry(2.2, 0.6, 26). Position z = 5.6+13 = 18.6 local → world 74+18.6 ≈ 92.6 center. Material: MeshBasicMaterial({map? use alphaMap gradient along Y… after rotate, UV v along axis ✓ (cylinder v is along height). Use map: gradientTex (white → transparent) with transparent:true, blending additive, color #6fd2ff, depthWrite false, side DoubleSide? front only fine… visible from inside? OK DoubleSide.
Gradient: canvas 8x128 vertical: top (v=1?) cylinder v=0 at bottom (−Y → engine side after rotate? bottom −Y maps to −Z (engine side) ✓ v0 at engine. Want bright at engine fading out: v0 white → v1 transparent. Canvas y down = v? CanvasTexture: y=0 top corresponds to v=1. Draw gradient accordingly: top (v1, far end) transparent, bottom (v0, engine) bright.
- Big glow sprite at (0,0,80) scale 26 additive subtle, plus PointLight (0x66ccff, 220, 160, 2) at z=82 flickering.

Also central spine engine: single bigger nozzle at (0,0,74)? Cluster of 4 around spine end + spine cap nozzle. ok add center engine with scale 1.3.

Tanks: 6 capsules around spine at radius 5.6, z center 48: CapsuleGeometry(1.9, 11, 6, 18) axis Y → rotateX(π/2) axis Z. Color #aab6c2 metal, plus stripe texture? Plain + small teal emissive ring decals skip. Struts: thin cylinders from spine to each tank (2 per tank).

Radiators at z=60: 2 panels: BoxGeometry(0.18, 20, 14)? Orientation: extend in Y: panel from y 4 to 24: box height 20 centered y=14, depth (z) 14, thin x .18. One at +Y, one at -Y. Material with emissiveMap gradient along Y: canvas gradient hot bottom (near spine) → dark tip. UV for box: tricky per face; acceptable: use material emissiveMap with gradient; box UV v is along which axis depends on face; side faces (±X) have v along height Y ✓ those are the big faces ✓.
Emissive color #ff6a3d intensity 1.2, color #241416, roughness .6. Also faint red PointLight nearby.

Lander at (0,-4.8,-6): body CapsuleGeometry(1.2,3.2) rotateX(π/2)? Axis along Z; scale y .8; canopy: small sphere emissive front; fins: 2 boxes; cradle arms from spine: 2 cylinders.

Antennas: forward top: at z=-70 boom cylinder r .12 length 10 along -Z from spine surface y=2.6 up? Place dish: LatheGeometry small dish at (0,4.5,-72) facing forward. Also 2 side sensor booms at z=-45: thin cylinders along ±X length 8 with tip spheres (nav lights!).

Nav lights: red sphere at ring A x=-30.5? Attach to static? Ring rotates; put nav lights at static spine ends and hub collars: red (−X) at (-3.8,0,-35)? Better: on shield rim: red at (-14,0,-89), green at (14,0,-89)? Convention port red — ship's port = -X? Whatever: red -X, green +X on rim. White strobe at spine aft top (0,4,70) and forward tip (0,0,-92)? Blink pattern: strobe: intensity = (t%1.2<0.08)?4:0. Nav steady glow 1.2 + slight pulse.

Materials list (shared): hullMat (map hullTex, color #98a4b3, metal .8, rough .4), darkMat (#39424f metal .7 rough .5), accentMat teal emissive for trim, glassMat etc.

Starfield points:
```js
function makeStars(count, rMin, rMax, size, opacity){ positions random on sphere shell; colors: 70% white, 15% #aac6ff, 15% #ffd9b0; PointsMaterial({size, sizeAttenuation:false, vertexColors:true, transparent:true, opacity, depthWrite:false}); }
```
Two layers: (2600, 1400, 2400, 1.6, .9) and (900, 1200, 2000, 2.6, 1).

Tau Ceti ahead: sprite glow scale 90 at (0, 60, -2200) color #fff2d0 + core small sprite 24. Also Earth/Sun behind? Small blue sprite at (140,-40, 2300) #9fc9ff scale 30 (Sol, receding). Nice storytelling.

Nebula sprites: 6 sprites: positions around at radius ~1800-2600, scales 900-1600, opacities .10-.16, additive, colors teal #2a7f7f, magenta #7f4f9f? Aurora palette: green #3fae82, teal, violet-blue #4a63c8, sparingly. Also a wide "milky way band": big sprite stretched (scale 3000x800) low opacity .08 rotated.

Dust streaks as described.

Now interior biome construction details:

R=160, θmax=0.55. Curve points: for i 0..N: θ = -θmax + 2θmax*i/N; p=(0, R(1-cosθ)+7.2, R sinθ). CatmullRomCurve3(pts). TubeGeometry(curve, 160, 9, 22, false). Material MeshStandardMaterial({map:corrTex? use hull variant with big repeat, side:BackSide, roughness .7, metalness .3, color #8fa0a8}). Interior sees BackSide ✓ lighting: standard material BackSide lighting works (normals auto inverted? three flips normals for back faces in lighting? Actually three does `#ifdef DOUBLE_SIDED` normal flip — yes, with side:BackSide three flips normals in lighting ✓).

Floor ribbon: custom geometry:
```js
const N=160, W=4.1; positions[], normals[], uvs[], indices;
for i: θ; T tangent=(0, sinθ, cosθ) normalized (already); outward=(0,-cosθ, sinθ); side=(1,0,0);
center = (0, R(1-cosθ)+7.2, R sinθ) + outward*7.35;
vL = center + side*W, vR = center - side*W; normal = -outward.
uv: u = i/N * 30 (repeat along), v 0..1.
indices quads.
```
Material MeshStandardMaterial({map:fieldTex, roughness .9, metalness 0, color #cfe? tint #c8d8c0}) — texture provides greens.

Also add edge soil borders: 2 thin ribbons at ±W..W+0.8 with darker dirt color? Skip, texture edge darkened.

Sunstrip ribbon on top: offset -outward*8.5 (toward ring center = up), width 1.6, MeshBasicMaterial color #fff3d8 → with bloom (basic bright > threshold) ✓. Also add faint secondary strip teal at side walls: offset side ±? Add 2 thin strips along walls at 45°: position center + (-outward*6 + side*±6)? Compute: up-ish. Keep to 1 top strip + point lights.

Trees: for i ~40: z random in ±(arcLen/2 - 6) → convert: pick θ random ±(θmax-0.06), lateral ±(1.2..3.6) choose side; pos = curvePoint + outward*7.35 + side*lat (floor point); tree group: trunk CylinderGeometry(.08,.12,h) color #6b4a2f; foliage: 2-3 cones stacked color varied green (#2f7d43, #3c9a55, #57b06b) slight emissive 0.05? Standard material. Height 1.2-2.6. Orient along outward (tree "up" = -outward): quaternion from (0,1,0) to -outward.

Crops: rows: ~10 patches: BoxGeometry(1.1, .25, 3.5) color #4c8f3f / #7fae4a alternating, placed similarly.

Water channel: ribbon along center lateral offset 0? Central path: instead: path strip: thin ribbon width .9 at center with gravel texture color #9a8f7a; plus a pond: CircleGeometry(1.8) at one spot, material MeshStandardMaterial color #1c4d5e roughness .15 metalness .1 emissive #0a2a33 .4 — plus animated shimmer via onBeforeCompile? Keep static + slight opacity? Fine.

Habitat pods: 3 domes: SphereGeometry(1.4, 20, 12, 0, 2π, 0, π/2) color #d7dde2 + window band emissive small torus; placed on floor.

Rocks: few dodecahedrons gray.

Fog: sceneBiome.fog = FogExp2(0x0d1826, 0.0075); background same color.

Lights: HemisphereLight(0x9fd0d8, 0x2a3d2a, .55); 5 PointLights along arc at θ -0.4..0.4, pos = curve + (-outward*6) (near ceiling), color #ffe8c0, intensity 60, distance 46, decay 2.

Also add distant "end doors": at both arc ends, dark disk with emissive ring (airlock) — a circle facing inward at curve end. Nice finishing touch.

Axis corridor:
Tube: CylinderGeometry(4,4,150,24,1,true) rotateX(π/2) BackSide map corrTex (panels with conduits). Ribs: InstancedMesh torus (4.05,.18) 18 count. Light strips: 2 long thin boxes at ±45° upper: BoxGeometry(.25,.25,148) emissive #cfe8ff 1.8, positioned at (±2.6, 2.6, 0)? Upper diagonal. Floor grating: ribbon along bottom width 3 y=-3.9 dark with grid texture, standard.
Floating cargo: 14 boxes/crates (BoxGeometry random .6-1.4) material crate color (#7a6a4f, #5f6f7a, teal-striped?) random rotation, slow angular velocity, slight position oscillation (store base + sin offset). Zero-g dust: 300 points Points size .03? PointsMaterial size .06 sizeAttenuation true, additive faint, slow drift.
End doors: at z=±74: CircleGeometry(4) dark + torus ring emissive amber, plus "HUB AIRLOCK" texture? Skip text.
Lights: PointLight every 25 (6 lights) #cfe0ff intensity 30 distance 30 decay 2 + hemi .3.

Command core:
Room: CylinderGeometry(7,7,4.8,40,1,false) along Y; material BackSide? Need to see inner walls: side:BackSide for walls; plus separate floor/ceiling disks (CircleGeometry r 7) with emissive ring inlays texture (canvas with concentric teal circles) standard.
Central column: at origin: CylinderGeometry(.55,.55,4.4) material glassy: MeshStandardMaterial({color:0x0c1a1c, emissive:0x35e0b0, emissiveIntensity:1.4, roughness:.2, metalness:.1}) + inner core cylinder .3 emissive 3 pulsing; gyroscope rings: 2 TorusGeometry(1.3,.06) and (1.7,.05) emissive teal, rotating on different axes; point light teal intensity 40 distance 20.
Holo star map: group at (0, 3.4?) room height 4.8: center y 0? Column occupies center; put holo map around column? Put holo map above column y=3? Ceiling 2.4… room h 4.8 → y ±2.4. Column full height. Holo map: offset to side: at (-3.5, 0.6, -2)? Radius 1.6 sphere of points + route line. Rotates slowly. Additive.
Consoles: 5 desks around perimeter radius 5.6, facing center: box (2.4,.9,1) dark + tilted screen plane with screenTex emissive 1.6; stools: small cylinders.
Viewport window: on wall at +Z side? Cut? Instead place a window panel: PlaneGeometry(6,2.2) at (0, 0.6, -6.9) facing inward, material MeshBasicMaterial map starViewTex (dark with stars + faint aurora) — reads as window. Frame boxes.
Lights: hemi .25, teal point from column, warm points at consoles 2 (#ffd0a0 int 18 dist 12).
Also floor ring emissive: torus r 3 tube .05 at floor y=-2.35 emissive teal 1.5, and r 6.

Interior animate: cargo rotation, holo rotation, column pulse, screen flicker (emissiveIntensity noise), dust drift.

Now UI wiring and data — write copy (Chinese):

Intro boot lines:
```
> Ship brain boot sequence … OK
> Self-diagnostic: biome loop · water cycle · atmospheric composition … normal
> Counter-rotation bearings of the two habitat rings … normal
> Fusion reactor output … 3.42 GW
> Welcome back. This is the 51,208th day of the voyage.
```
Title: 极光号 AURORA; sub: Kim Stanley Robinson "Aurora" · interstellar generation ship interactive reconstruction; button: Board.

About text (in intro or card): The novel doesn't give engineering drawings; this model is a reconstruction based on text clues: two counter-rotating habitat rings, central zero-g spine, forward deflector shield, aft fusion engine cluster, water tank cluster and radiators, seed lander, and the ship brain that watches over everything.

Part descriptions (concise, evocative):
- Shield: Even dust at a tenth of light speed hits like a bullet. The forward deflector shield takes the brunt of the interstellar medium for the entire ship, its surface constantly blooming with tiny impact flashes.
- Ring A/B: The ring rotates to grant gravity. Inside are biomes named after Earth places — farmland, forest, lakes, beaches. Two rings counter-rotate to cancel gyroscopic torque. (Novel: 24 biomes)
- Spine: The ship's backbone. Elevators, pipes, cables, and zero-g cargo trunk run inside; it's the only route connecting the two rings and the engines.
- Engines: Fusion engines push for decades, then the ship flips and burns for decades to slow down. The pale blue plume is the brightest thing on the ship.
- Tanks: Water is reaction mass, and the best shield against radiation and cosmic rays.
- Radiator: In the vacuum there's no air to cool; waste heat can only be radiated away as infrared, and the panels always glow dull red.
- Lander: The "seed." When they arrive, it will carry people from the ring to the surface of Aurora.
- Ship brain: It has no body, and its body is the whole ship. In the novel, it is the narrator.

Mode descriptions in buttons: Exterior Overview EXTERIOR / Ring Biome RING BIOME / Zero-g Axis ZERO-G SPINE / Command Core SHIP MIND.

Hints:
Exterior: Drag to rotate · Wheel zoom · Right-click pan · Click labels for part archive · Keys 1-4 switch areas
Interior walk: WASD move · Drag to look · Shift run · Esc return to exterior
Axis fly: WASD move · Q/E ascend/descend · Drag to look · Shift accelerate

Telemetry chips: SPEED 0.1000 c | SHIP TIME Y137·D214 | RING SPIN 0.9 rpm | SHIELD 99.6%.

Now — write the code. Careful with syntax. Also handle canvas pointer events: labels layer above canvas but pointer-events none except tags.

Also prevent OrbitControls when interior (controls.enabled=false).

Composer and renderPass scene swap; bloom pass shared; OutputPass last.

One more consideration: UnrealBloomPass and transparent sprites: fine.

Renderer alpha false; setClearColor per scene via scene.background.

Resize: update camera aspect, renderer, composer size, bloom resolution.

Now let me write the final file. Aim for clean, organized, commented code. Also include a small "about" modal? Intro covers it; add "?" button top-right to reopen intro (as info). Keep: button "About the Novel" opens card with about text.

File name: aurora-ship.html.

Also provide run instructions in chat + a README snippet. Deliverable: single HTML. Mention: needs network for CDN (three.js + fonts); alternative: python http server. Test checklist.

Let me write out the code fully, compiling in my head as I go.

Potential pitfalls to double-check while writing:
- THREE.CapsuleGeometry exists in r160 ✓ (added r142).
- OutputPass import path 'three/addons/postprocessing/OutputPass.js' ✓ r160.
- OrbitControls import ✓.
- InstancedMesh matrix set + instanceMatrix.needsUpdate.
- CanvasTexture colorSpace: for emissive/color maps set tex.colorSpace = THREE.SRGBColorSpace ✓ r160.
- PointsMaterial vertexColors.
- FogExp2 for biome only.
- When interior, hide exterior labels and dust (different scene anyway).
- Controls: create OrbitControls on renderer.domElement; disable when interior.
- Pointer events: interior look-drag on renderer.domElement; ensure OrbitControls doesn't interfere when disabled (enabled=false stops handlers ✓).
- Keys: track e.code; preventDefault for Space/arrows when interior to avoid scroll (page doesn't scroll anyway).
- Mode switch cleanup: stop autoRotate when leaving exterior.

Camera focus tween: implement `tweens` array of {obj:{}, from, to, t, dur, ease, apply}. Simpler: single cameraTween {t0, dur, camFrom, camTo, tgtFrom, tgtTo} updated in loop; disable controls during tween; re-enable after. Trigger from part card "focus." Also initial intro flight: camera from far (0, 60, 520) to (70,34,120) over 3s after boarding — cinematic ✓.

Label DOM: create per part: <div class="tag"><span class="dot"></span><span class="txt">name</span></div>. Click → openCard(part).

Card: title, English subtitle, description, stats row maybe, buttons: focus view (exterior parts), enter interior (mapping), close.

Part → interior mapping: ringA/ringB → biome; spine → axis; ai → core.

Now telemetry interval and day counter: startDay = 214; increment 1 per 2s; wrap 365 → year++.

Shield flash scheduler: in animate: if(t > nextFlash){ spawn; nextFlash = t + .2+Math.random()*1.2 }.

Plume flicker: for each plume material: opacity = .55+.25*sin(t*17+i)*sin(t*7.3+i*2)… plus scale.x/y pulse.

Engine light intensity flicker similarly.

Ring rotation speed: 0.14 rad/s and -0.14.

Ship bob small.

Strobe: mat.emissiveIntensity = (t % 1.4) < .07 ? 5 : .2 — need separate materials per strobe (clone).

Nav lights steady pulse: 1.1+.4 sin(t*2).

Interior cargo animate: store {mesh, angVel:Vector3, baseY, phase}: rotation += angVel*dt; position.y = base + sin(t*.5+phase)*.4.

Biome water shimmer: skip (pond static).

Sunstrip flicker very subtle: material color constant; fine.

Holo rotation .2 rad/s; column pulse emissiveIntensity 1+.6 sin(t*2.4); ring gyros rotate.

Screen flicker: screenMat.emissiveIntensity = 1.3 + .15 sin(t*30)?? Too strobey; use slow .1 sin(t*3)+ noise small.

Now size: total code ~1100-1400 lines. Acceptable.

Also mobile touch buttons: div #touchpad with buttons ▲ ▼ ◀ ▶ shown when interior and (pointer:coarse). Wire to key flags (KeyW etc.).

Also add "return to exterior" floating button when interior (bottom-right) — mode buttons also always visible on left.

Edge: label projection when camera close — labels may overlap; fine.

Let me write hull texture draw:
```js
const hullTex = canvasTexture(512,512,(g,w,h)=>{
 g.fillStyle='#8b97a5'; g.fillRect(0,0,w,h);
 // panel noise
 for(let i=0;i<900;i++){ g.fillStyle=`rgba(${20+Math.random()*40|0},${30+Math.random()*40|0},${40+Math.random()*40|0},${Math.random()*.08})`; const s=2+Math.random()*26; g.fillRect(Math.random()*w,Math.random()*h,s,s*.6);}
 // grid
 g.strokeStyle='rgba(15,20,28,.5)'; g.lineWidth=2;
 for(let x=0;x<=w;x+=64){ line } for(let y=0;y<=h;y+=64){}
 // some hatches
 g.fillStyle='rgba(30,38,48,.35)'; a few rectangles.
});
```
Set repeat per usage: clone texture for different repeats? CanvasTexture repeat set per texture instance; need multiple: spine (repeat 8,3?), ring hull (repeat 24,3), etc. Make helper returning new texture per call (each call creates a new canvas — fine, or clone with needsUpdate). Call canvasTexture multiple times with same draw function but different repeats — draw uses Math.random → slight variation, fine.

Corridor texture:
```js
base '#5c6672'; horizontal panel lines; color conduits: 2 teal stripes, 1 amber; darker floor grid.
```
Repeat along length (1, 40)? For cylinder along Z after rotateX: cylinder V axis = original Y → maps to Z ✓. Set repeat (6, 30).

Field texture:
```js
g.fillStyle='#24401f'? draw strips along v? Our ribbon UV: u along length (set to repeat 30), v across width. Strips across width (farm plots): draw horizontal bands in v direction: various greens/tans bands along canvas Y, each with crop line pattern (thin darker lines along band). Canvas 256x256, repeat(2, 24)? u along length repeats 24 times: set repeat(24,2)? Hmm, canvasTexture sets repeat via param; ribbon UV u = i/N*30 (0..30), v 0..1. Texture repeat multiplies uv: set repeat(1,1) and bake: draw bands along X (u direction)? Fields should be plots along walking direction… simpler: draw patchwork grid: random green rectangles in both directions + path lines; repeat(28, 3). Looks like farmland from inside. ✓ Plus central path: draw on separate ribbon? Skip — add gravel path as separate thin ribbon width .9 at offset 0 with color #8d8468.

Actually, water channel: replace path with shallow channel: ribbon width 1.0, material color #14424f, roughness .1, metalness .3, emissive #06222b .6 → looks like water. Add small border boxes. ok.

Screen texture:
```js
bg '#03121a'; teal grid; header text 'AURORA // SHIPMIND 3.42GW'; random bar graphs; sine wave lines; small text rows (rectangles as text). Font 'Orbitron' may not be loaded on canvas — use monospace.
```

Star view texture (viewport window): black + 200 star dots + faint teal nebula blob.

Crate texture? Plain color fine.

Env texture: 256x128: vertical gradient: top '#0e2233', mid '#0a141f', bottom '#030507'; bright blob at (0.7w, .3h) radial white-teal radius 30; another faint magenta blob. Map onto sphere BackSide. Plus PMREM.

envMap intensity: default 1; material envMapIntensity via scene.environment applies to all standard materials ✓. Set renderer.toneMapping ACES.

Now labels/parts data with anchors and camera presets:

```js
const PARTS=[
 {id:'shield', name:'Forward Deflector Shield', en:'DEFLECTOR SHIELD', pos:[0,2,-89], view:[26,8,-138], desc:...},
 {id:'ringA', name:'Habitat Ring · A', en:'HABITAT RING A', pos:[0,32,-35], view:[62,24,-88], interior:'biome', desc:...},
 {id:'ringB', name:'Habitat Ring · B', en:'HABITAT RING B', pos:[0,-32,25], view:[-58,-20,72], interior:'biome', desc:...},
 {id:'spine', name:'Central Spine', en:'CENTRAL SPINE', pos:[0,5,-12], view:[46,16,-30], interior:'axis', desc:...},
 {id:'tanks', name:'Water and Fuel Tank Cluster', en:'WATER / PROPELLANT TANKS', pos:[7,6,48], view:[40,14,66], desc:...},
 {id:'rads', name:'Radiator Panels', en:'RADIATOR PANELS', pos:[0,18,60], view:[-34,30,86], desc:...},
 {id:'eng', name:'Fusion Engine Cluster', en:'FUSION DRIVE', pos:[0,-3,84], view:[34,-10,140], desc:...},
 {id:'lander', name:'Seed Lander', en:'SEED LANDER', pos:[0,-6.5,-6], view:[18,-14,-26], desc:...},
 {id:'mind', name:'Ship Brain', en:'SHIP MIND / AI', pos:[0,4.5,-62], view:[-26,10,-92], interior:'core', desc:...},
];
```

Views chosen to frame each part. Focus tween: controls.target → pos, camera → view.

Ring B label at bottom (0,-32,25) for variety.

Now animate loop structure:

```js
const clock=new THREE.Clock();
function animate(){ requestAnimationFrame(animate); const dt=Math.min(clock.getDelta(), .05); const t=clock.elapsedTime;
 updateShip(dt,t); updateDust(dt); updateFlashes(t); updateInteriorAnim(dt,t);
 if(mode==='exterior'){ controls.update(); updateLabels(); } else { ic.update(dt); }
 if(camTween) updateCamTween();
 composer.render();
}
```

renderPass.scene always activeScene.

updateLabels: only when mode exterior and labelsOn.

Mode switch function with fade:

```js
function setMode(m){ if(m===mode) return; fadeEl.classList.add('on'); setTimeout(()=>{ applyMode(m); fadeEl.classList.remove('on'); },300); }
function applyMode(m){ mode=m; ... }
```

applyMode details:
```js
controls.enabled = (m==='exterior');
labelsLayer.style.display = (m==='exterior'&&labelsOn)?'':'none' (handled in updateLabels anyway)
switch(m){
 case 'exterior': activeScene=sceneExt; camera.fov=50; camera.position.set(70,34,120)? — better preserve last exterior view: save extPos/extTarget on leave, restore. Simpler: keep camera as is; on enter exterior, if coming from interior, restore saved exterior camera state. Save extCam = {pos, target} before leaving.
 case 'biome': activeScene=sceneBiome; camera.fov=70; camera.position.set(0,1.7,0); ic.set('walk'); ic.yaw=Math.PI; ic.pitch=0.05;
 case 'axis': activeScene=sceneAxis; fov=72; pos (0,0.5,-62); yaw=Math.PI; pitch=0; ic.set('fly', boundsAxis);
 case 'core': activeScene=sceneCore; fov=66; pos (0,0.8,5.6); yaw=0? Look at column from +Z toward -Z: yaw=0 ✓ pitch=-0.05; ic.set('fly', boundsCore);
}
camera.updateProjectionMatrix(); renderPass.scene=activeScene;
Update hint text; update mode button active; update HUD mode name; touchpad visibility.
```

Bounds:
- Axis: clamp: r = sqrt(x²+y²) ≤ 3.2 → scale back; z ∈ [-70,70].
- Core: r ≤ 6.3 (x,z), y ∈ [-1.6, 1.9].

Walk: x ∈ [-3.6,3.6], z ∈ [-86, 86]? Arc z range = R sin θmax = 160*sin(.55)=160*.5227=83.6 → clamp ±80. y = floorY(z)+1.7 where floorY = R(1-cos(asin(z/R))) → compute: θ=asin(z/R); y=R(1-cosθ). ✓

Also allow looking around freely in walk.

IC.update:
```js
update(dt){ if(!this.enabled) return; const c=this.camera;
 c.quaternion.setFromEuler(_euler.set(this.pitch,this.yaw,0,'YXZ'));
 const sp=(keys.ShiftLeft?2.1:1)*this.speed;
 // desired direction
 let f=(KeyW?1:0)-(KeyS?1:0), r=(KeyD?1:0)-(KeyA?1:0), u=(KeyE?1:0)-(KeyQ?1:0);
 if(this.mode==='walk'){ fwd=(-sin yaw,0,-cos yaw), right=(cos yaw,0,-sin yaw); move = fwd*f + right*r; normalize if len>1; targetVel = move*sp; }
 else { fwd full 3D with pitch; right=(cos yaw,0,-sin yaw); up=(0,1,0)*u; targetVel=... }
 vel.lerp(targetVel, 1-Math.exp(-8*dt)); c.position.addScaledVector(vel, dt);
 clamp per mode; if walk set y.
}
```
Keys stored as this.keys[e.code]=bool.

Note: 'KeyW' etc.

Touchpad buttons set ic.keys via pointerdown/up/leave.

Now intro sequence:
```js
const bootLines=[...]; async type them into #bootlog with 260ms delay each; then show #enterBtn.
enterBtn.onclick = () => { intro.classList.add('hide'); startIntroFlight(); }
startIntroFlight: camera.position.set(-160, 90, 480); tween to (70,34,120) target (0,0,-5) over 3.2s ease inOut. Also start telemetry.
```
Set __booted flag after init to cancel loader error timeout.

Loader: since modules load async, show loader immediately (pure CSS), hide when init done (add class). If error/timeout 9s: show error text in loader.

Esc key → setMode('exterior'). Keys 1-4.

Auto-orbit checkbox and labels checkbox.

I think I'm ready to write. Also include a small favicon via data URI? Optional, skip… add <link rel="icon" href="data:,"> to avoid 404 noise. Fine.

Double-check bloom with interior fog: FogExp2 + bloom fine.

One more: exterior scene.background = new THREE.Color(0x020409). Star points render on top ✓.

Sprite for glow: THREE.Sprite with SpriteMaterial({map:glowTex, color, blending:Additive, depthWrite:false, transparent:true}). Scale.

Shield flashes: pool of 10 sprites created, each with {life: -1}; spawn sets life=0, position; update: life+=dt; k=life/.28; scale = .4+2.2*k; opacity=(1-k)*1.4? Sprite material opacity per sprite needs unique material — clone material per sprite (10 materials fine).

Dust: LineSegments positions Float32Array(N*2*3); velocities constant; update in loop: for each i: z1+=sp*dt etc.; wrap. Also store per-segment length fixed at init. N=320.

Also slight camera shake when exterior? Skip (orbit controls conflict).

Ring windows: emissiveIntensity 1.6 with bloom threshold .75 → windows bloom softly ✓.

Sunstrip in biome: MeshBasicMaterial color 0xfff2d0 — brightness 1.0 < threshold? Basic white ~1.0 luminance, after tone mapping… bloom threshold works on pre-tonemap linear values; white basic = 1.0 → luminance ~1.0 > .75 ✓ blooms.

Radiator emissive 1.2 orange blooms slightly ✓.

Plume additive bright ✓.

Set bloom strength .85, radius .55, threshold .72.

Also OutputPass.

Renderer: powerPreference 'high-performance'.

Now write. Careful with brace matching. Also ensure Chinese text is proper.

For delivery: provide the complete HTML file in a single code block, plus run instructions and test checklist, plus a short note about the novel source and honest disclaimer about "search": since I can't browse in this environment, the reconstruction is based on my knowledge of the novel's text. Keep that note brief (1-2 lines), not over-claimed.

Actually — one more consideration about the novel identification: Chinese "Aurora" most likely refers to Kim Stanley Robinson's "Aurora" (Chinese translation titled "Aurora," published by Readom Culture/Chongqing Press 2018? I think the Chinese edition exists). State: "Based on Kim Stanley Robinson's "Aurora" (Aurora, 2015)." If the user meant a different "Aurora," the model/answer can be adjusted — add one line inviting correction. Good.

Let me finalize dimensions/numbers for HUD plausibility: novel ship: length ~1 km? Just display scale note: "Model scale 1:20 (novel's ship is about several kilometers long)"? Avoid over-claiming specific numbers; keep HUD telemetry to speed/time/rpm/shield.

Let me write the code.

Additional details — spokes with quaternion:
```js
function strutBetween(a, b, radius, mat, parent){ const dir=b.clone().sub(a); const len=dir.length(); const mesh=new THREE.Mesh(new THREE.CylinderGeometry(radius,radius,len,10), mat); mesh.position.copy(a).addScaledVector(dir,0.5); mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0), dir.normalize()); parent.add(mesh); return mesh; }
```

Ring pods InstancedMesh:
```js
const podGeo=new THREE.BoxGeometry(1.1,2.4,1.6); // y radial
const pods=new THREE.InstancedMesh(podGeo, podMat, 64);
const M=new THREE.Matrix4(),Q=new THREE.Quaternion(),P=new THREE.Vector3(),S=new THREE.Vector3(1,1,1),Z=new THREE.Vector3(0,0,1);
for(let i=0;i<64;i++){ const a=i/64*Math.PI*2+Math.random()*.04; const r=30.3; P.set(Math.cos(a)*r,Math.sin(a)*r,(Math.random()-.5)*2.2); Q.setFromAxisAngle(Z,a); const s=.7+Math.random()*.7; S.set(s,s,s); M.compose(P,Q,S); pods.setMatrixAt(i,M);} pods.instanceMatrix.needsUpdate=true;
```
Box y dimension radial ✓ (after rotation around Z by a, local Y points radially ✓).

Rib instances similarly with positions.

Tanks capsule rotation: CapsuleGeometry axis Y; rotateX(Math.PI/2) → axis Z ✓.

Engine nozzle lathe profile:
```js
const pts=[]; [[.55,0],[.62,.6],[.78,1.6],[1.15,3],[1.75,4.6],[2.3,5.8]].forEach(p=>pts.push(new THREE.Vector2(p[0],p[1])));
const noz=new THREE.LatheGeometry(pts,28); noz.rotateX(Math.PI/2); // flare toward +Z ✓
```
Material DoubleSide so inside visible from behind? Inside visible from rear front; set side DoubleSide.

Throat glow circle at z .2 radius .55.

Plume cylinder: CylinderGeometry(2.3, .6, 26, 20, 1, true); rotateX(π/2) → +Y end (rTop=2.3) → +Z ✓ wide at back.

Gradient alpha map: create canvas 16x256: gradient along y: y=0 (top → v=1 → far end) transparent; y=256 (v=0, engine) white. Use as `map` with color set? MeshBasicMaterial({color:#7fd8ff, alphaMap:gradTex, transparent:true, opacity:.85, blending:Additive, depthWrite:false, side:THREE.DoubleSide}). alphaMap uses g channel ✓ grayscale fine.

Hmm, v mapping on cylinder: v=0 at bottom (−Y → engine side after rotate? bottom −Y maps to −Z (engine side) ✓ v0 at engine. Want bright at engine fading out: v0 white → v1 transparent. Canvas y down = v? CanvasTexture: y=0 top corresponds to v=1. Draw gradient accordingly: top (v1, far end) transparent, bottom (v0, engine) bright. ✓

Glow sprite behind engine cluster at z≈96 scale 30 opacity .5.

Shield lathe profile: pts (0.4,0)→(14,3): parabolic: for i 0..12: r=.4+13.6*i/12, y=3*(i/12)^2. LatheGeometry(pts, 56). rotateX(-π/2): +Y → −Z ✓ so dish depth extends toward −Z (forward), concave faces −Z ✓.

Wait, verify concave direction: lathe surface normal points outward (away from axis)… concave side is the "inside" of the bowl facing +Y originally → after rotation faces −Z ✓ front. From ship (looking forward from +Z side) we see convex back — correct with ribs there.

Ribs on back (+Z local side): radial boxes at local z=+.3.

Mast: cylinder r .7 length 7 along Z at world z=-82 (from -85.5 to -78.5): CylinderGeometry rotateX(π/2).

Also add shield secondary layer: smaller dish r 9 offset z +1.5 behind? Adds depth. Sure: second lathe r 9 depth 2 at local z=+1.2, darker.

Antenna dish: small lathe r 2 depth .6 at (0, 5.2, -66) facing -Z (rotateX(-π/2)), plus boom from spine.

OK, docking bay near lander: 2 cradle arms.

Now biome airlock ends: at θ=±θmax: circle radius 9.2 facing inward: position at curve end + tangent*0.5, lookAt center. MeshStandard dark + emissive ring torus r 6 tube .15 amber. Compute orientation: mesh.lookAt(centerPoint). Circle faces +Z by default; lookAt orients +Z toward target ✓.

Biome point light positions: for k -2..2: θ=k*.18; pos = curvePoint(θ) + (-outward)*5.5 (near ceiling, since ceiling direction is -outward). Wait, ceiling = toward ring center = -outward? Outward = away from ring center (down toward floor). So up = -outward ✓ ceiling offset -outward*5.5 from centerline; tube radius 9 so ceiling is at -outward*9; lights at -outward*5.5 hang down ✓.

Sunstrip offset -outward*8.3 width 1.8.

Also add thin teal wall strips at side ±: offset side*±8.3, oriented? Skip.

Trees up direction = -outward ✓ (grow from floor upward toward ceiling).

Floor offset outward*7.35 (tube r 9; floor slightly above tube bottom inside 9 → 7.35 gives ground thickness).

Camera eye 1.7 above floor.

Tube radius 9 → ceiling height above floor ≈ 9+7.35? Floor at outward 7.35; tube top at -9 outward direction… total clearance = 7.35+9 = 16.35?? No: floor is at +7.35 outward from centerline; ceiling is at -9 outward (i.e., 9 in opposite direction): distance = 16.35 units ≈ meters — grand interior ✓ nice (big habitat).

Hmm, but then floor width: chord at offset 7.35 in tube r 9: half-width = sqrt(9²−7.35²)=sqrt(81−54)=sqrt(27)=5.2 ✓ floor width up to 10.4; use W=4.1 walkable with walls curving up on either side ✓.

Great.

Axis corridor radius 4 fine.

Now HUD mode name element update: "Current area: Exterior Overview" etc.

Card interior button label: "Enter: Ring Biome" etc.

I think I'm ready to write. Also don't forget meta viewport for mobile.

One more thing: OrbitControls target (0,0,-5); minDistance 16 maxDistance 480; enablePan true but screenSpacePanning false; dampingFactor .06.

Initial camera before intro flight: set to flight start position.

Also `renderer.setAnimationLoop(animate)` alternative to rAF — use setAnimationLoop, cleaner.

Font loading: <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&family=Orbitron:wght@500;700&family=ZCOOL+QingKe+HuangYou&display=swap">.

Now let me write the whole thing. Also add a subtle scanline overlay? Skip; vignette yes.

Card stats: add per-part "DATA" row: e.g., shield: diameter ~28 m (model) / status: normal with micro-impacts; keep generic: each part: {stat1:[label,value], stat2}. Add small.

Time to write the final answer. Structure:
1. Brief description (novel identification + reconstruction basis + disclaimer about not being able to browse live).
2. Complete code in single file.
3. Run instructions.
4. Test checklist.
5. Feature list mapped to novel elements.

Keep prose concise; the code is the deliverable. All Chinese.

Now let me write the code carefully, compiling in my head.

Special care: template literals inside module script are fine. Avoid `<` issues? None.

Also: `new OrbitControls(camera, renderer.domElement)`.

Interior look drag: attach to renderer.domElement; but OrbitControls also listens; when disabled (enabled=false) it ignores events ✓.

Pointer capture on domElement for drag.

Also ignore drag start on UI (UI elements are separate DOM above canvas, events don't reach canvas ✓).

Labels layer: position absolute inset 0, pointer-events none; children .tag pointer-events auto.

Update label positions each frame with transform translate3d.

Hide labels when card open? Keep.

Focus tween function:
```js
let camTween=null;
function flyTo(pos, tgt, dur=1.6){ camTween={t:0,dur, p0:camera.position.clone(), p1:new THREE.Vector3(...pos), t0:controls.target.clone(), t1:new THREE.Vector3(...tgt)}; controls.enabled=false; }
in loop: if(camTween){ camTween.t+=dt; k=ease(min(1,t/dur)); camera.position.lerpVectors(p0,p1,k); controls.target.lerpVectors(t0,t1,k); if done → camTween=null; controls.enabled = (mode==='exterior'); }
```
ease = k<.5? 2k² : 1−(−2k+2)²/2.

Intro flight uses same flyTo with dur 3.4 from initial position.

Set camera initial position (-140, 80, 460), target (0,0,-5) before board; on board → flyTo((70,34,120),(0,0,-5),3.4).

During intro (before board), animate runs rendering scene (nice backdrop behind intro overlay? Intro overlay is opaque-ish dark with slight transparency showing stars? Make intro background rgba(3,6,12,.82) so ship faintly visible + slow autoRotate on before board: controls.autoRotate=true speed .4; on board: set autoRotate to checkbox state (default off? Default on for gentle motion? Default auto-orbit ON at .35 speed for showcase; user drag temporarily overrides (OrbitControls autoRotate continues after interaction; that's fine, toggle available). Actually autoRotate + user drag can be annoying; keep default ON slow (.4) — cinematic, and toggle visible. Hmm, choose default ON slow.

Telemetry starts after board.

Edge: label click during camTween fine.

Now write. (Also don't forget `controls.autoRotateSpeed=0.5`.)

For interior scene backgrounds: biome #0c1624, axis #05080e, core #070b12.

Biome fog FogExp2(0x0c1624, .0075). Axis fog FogExp2(0x05080e, .012) for depth. Core fog slight .008.

Interior bloom: same pass.

Also add faint aurora ribbon inside biome? The novel's ring has artificial lighting; fine.

Count interior lights for perf: biome 5 point + hemi; axis 6 point + hemi; core ~4. ok.

Now generate final code. Write as one continuous block, double-checking syntax as I go. Also include `window.__AURORA_BOOTED`.

Rough line plan (JS sections). Write it.

function makeCanvas(w,h,fn){...return canvas}
function ct(w,h,fn,rep=[1,1],srgb=true){ const c=makeCanvas...; const t=new THREE.CanvasTexture(c); t.wrapS=t.wrapT=THREE.RepeatWrapping; t.repeat.set(rep[0],rep[1]); if(srgb) t.colorSpace=THREE.SRGBColorSpace; t.anisotropy=8; return t; }

Textures:
- hullTex(rep)
- windowsTex
- corrTex(rep) corridor
- fieldTex
- gradAlphaTex (linear gradient, no srgb? alphaMap should be linear: srgb=false)
- glowTex (radial) — for sprites, srgb fine
- nebulaTex(colorA, colorB)
- screenTex
- starViewTex
- ringFloorTex (core floor with concentric circles)
- envTex

Materials:
- matHull: MeshStandardMaterial({map:hullTex([10,2]), color:0xaeb9c6, metalness:.82, roughness:.42})
Wait, map multiplied by color; hull texture is grayish so color tint brightens. ok.
- matHullDark: color 0x59636f, map hullTex([14,3]), metal .75 rough .5
- matBand (windows) per ring (share 1 material, different texture repeat? Same texture repeat 10 fine for both rings)
- matAccent emissive teal: MeshStandardMaterial({color:0x0a251f, emissive:0x35e0b0, emissiveIntensity:1.2, metalness:.4, roughness:.4})
- matDark: {color:0x2c343e, metalness:.7, roughness:.55}
- matTank: {color:0xb8c2cc, metalness:.85, roughness:.35, map:hullTex([3,2])?} plain.

Env: build after renderer; scene.environment = pmrem.

Apply environment to interior scenes too? scene.environment is per-scene property — set on all scenes (nice for interior metals).

PMREM: const pmrem=new THREE.PMREMGenerator(renderer); const envScene=new THREE.Scene(); sphere with envTex BackSide basic; plus 2 bright planes MeshBasicMaterial white; envRT=pmrem.fromScene(envScene, .05? blur sigma); scene.environment=envRT.texture.

fromScene signature: fromScene(scene, sigma=0, near=0.1, far=100) ✓ sigma .04 slight blur? Keep 0.

Now write. Also small helper addLabelDOM(part).

Also `document.getElementById` shorthand $.

OK, final structure. Now write it out (long). Ensure no stray Chinese quotes in JS strings (use standard). Use single quotes in JS.

Also: prevent context menu? OrbitControls right-drag pan uses contextmenu suppression itself (adds contextmenu listener preventing). ✓

Touch: OrbitControls touch rotate/pinch ✓.

Interior on mobile: touchpad shown (media query pointer coarse). Include ▲▼◀▶ plus "return to exterior."

Final check on imports used: THREE, OrbitControls, EffectComposer, RenderPass, UnrealBloomPass, OutputPass.

Write the answer with concise sections. Given the huge code, keep prose minimal.
