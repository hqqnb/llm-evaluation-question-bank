The user wants a 3D wristwatch animation that satisfies the following conditions:
- Runs on the current time
- Smoothly sweeping second hand, minute hand, hour hand
- Date and day-of-week display
- Supports dual time zones
- Fully 3D wristwatch presentation

Should be delivered as a single runnable HTML file using Three.js (via CDN). Since the request is in Chinese, I'll respond in Chinese.

Design goals: make it rich, lively, layered, with an atmospheric background, motion and micro-interactions. The watch itself should be front and center from the start.

Let me think about the design:

- Three.js scene with a detailed 3D wristwatch: case, bezel, lugs, crown, dial, indices, hands (hour/minute/second), date window, day window, strap segments, sapphire crystal with reflections.
- Smooth sweep: second hand moves continuously (sweep like a mechanical watch — 8 beats/sec or continuous). Use requestAnimationFrame with milliseconds.
- Dual time zone: a toggle or a second subdial (GMT hand). Good approach: a GMT hand (24-hour hand) + a rotating bezel, or a subdial for the second time zone. Also a UI panel to select the home/local time zones.
- Date and day display: day-of-week window at 12 or day window at top, date window at 3 o'clock. In 3D, can be rendered via canvas texture updated daily, or drawn on the dial texture. Better: separate small plane with canvas texture for the date/day, updated when date changes.
- Micro-interactions: OrbitControls for rotation, hover effects, buttons for time zone selection, a "movement" toggle (quartz tick vs mechanical sweep), light changes.
- Background: atmospheric gradient environment, soft studio lighting, subtle particles/dust, reflective floor.

UI: side panel with two time zone selectors (local and GMT), current digital readout, date/day display. Typography: distinctive display face + readable body face. Watchmaking vibe: Cinzel or Marcellus for display, Manrope/Space Grotesk for body. Since the UI is in Chinese, I need a Chinese-capable stack; Latin display font for numerals/branding, system Chinese font for text. Use Google Fonts: "Cormorant Garamond"? Watch brands often use serif or geometric. Let me use "Marcellus" (elegant, watch-like) for display, "Manrope" for body, plus "Noto Serif SC" for Chinese display. Keep it simple: Marcellus + Noto Sans SC.

Watch design: luxurious feel — dark green or midnight blue sunburst dial, gold/steel case. Sunburst dial via canvas texture with radial brushed gradient. Applied indices via 3D geometry (boxes). Hands as extruded shapes.

Implementation plan (single HTML file):

1. Load Three.js from CDN (unpkg or jsdelivr). Use module imports with importmap: three@0.160.0 and OrbitControls from examples/jsm. Use jsdelivr: https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js and OrbitControls from https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/controls/OrbitControls.js.

Note: In some environments CDNs may be blocked; but this is standard. I'll use jsdelivr. Also provide fallback instructions.

2. Scene setup:
- Renderer with antialias, shadows, ACESFilmicToneMapping, sRGB.
- Camera perspective, positioned to view the watch at an angle.
- OrbitControls with damping, min/max distance.
- Environment: use RoomEnvironment (three/examples/jsm/environments/RoomEnvironment.js) + PMREMGenerator for beautiful metallic reflections. This significantly improves the appearance of metal.

3. Watch construction (a group `watch`):
- Case: torus/cylinder ring — use LatheGeometry for a nice case profile, or a combination of cylinders. Simpler: main case cylinder (metal), bezel torus on top, caseback cylinder.
- Lugs: 4 rounded boxes (capsule-like) — use rotated CylinderGeometry or RoundedBox. Can approximate with scaled spheres/capsules.
- Crown: knurled cylinder on the side (cylinder with small ridges — a cylinder with low radial segments + normal trick, or just a fluted cylinder via multiple thin boxes; simpler: CylinderGeometry with 24 segments and flat shading for a knurled look).
- Dial: circle with canvas texture (sunburst gradient, minute track, brand text, "GMT" text). Canvas 1024x1024.
- Indices: 12 applied batons (boxes, gold metal), doubled at 12. Skip at 3 if there's a date window? Place date window at 3, day window at 12? Classic: day+date window at 3 (like Rolex Day-Date with day arched at 12). Let's do: date window at 3 o'clock, day window at 12 o'clock (rectangular). Both as small planes with canvas texture slightly above the dial, with a metal frame.
- Hands:
  - Hour: extruded shape (dauphine-style) — build with Shape + ExtrudeGeometry, thin.
  - Minute: longer, thinner.
  - Second: thin needle + counterweight, red/gold accent.
  - GMT hand: arrow-tipped, 24-hour, different color (e.g., orange/red arrow), rotates once per 24 hours.
- Each hand in a pivot group at the dial center, slightly stacked in z.
- Crystal: slightly domed sphere segment, MeshPhysicalMaterial with transmission, low roughness, ior 1.5, thin. Transmission can be expensive but fine. Or transparent with opacity 0.15 + envmap. Use physical material with transmission=1, thickness 0.5, roughness 0.02 — beautiful.
- Strap: series of curved link boxes extending up and down, curving as if wrapping around a wrist — use a curve (CatmullRom) to place links along an arc. Or a simple curved leather strap via TubeGeometry/extrude along a curve. Metal bracelet links look great: place rounded boxes along a circular arc (as if wrapping around a wrist of radius ~2.2 behind the watch).
- Bezel: 24-hour GMT bezel option? Can add bezel numerals via canvas texture on a ring — nice touch: rotating GMT bezel. Keep static with engraved-look texture.

4. Time logic:
- Get time zone via Intl API: use new Date().toLocaleString('en-US', { timeZone: tz }) or formatToParts to get h/m/s. Better: compute offset with Intl.DateTimeFormat's formatToParts. To get smooth seconds with ms: get ms from local Date.now(), and compute the time zone time as UTC + offset. Offset calculation: use a helper that computes the tz offset (in minutes) via Intl.DateTimeFormat with that timeZone and formatToParts for a given timestamp. Then tzTime = Date.now() + offset*60000 treated as UTC. This gives continuous ms.

Implementation:
```js
function tzOffset(ts, timeZone){
  const dtf = new Intl.DateTimeFormat('en-US',{timeZone, hour12:false, year:'numeric',month:'2-digit',day:'2-digit',hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const parts = dtf.formatToParts(ts).reduce((a,p)=>(a[p.type]=p.value,a),{});
  const asUTC = Date.UTC(parts.year, parts.month-1, parts.day, parts.hour==='24'?0:parts.hour, parts.minute, parts.second);
  return (asUTC - Math.floor(ts/1000)*1000) / 60000; // minutes
}
```
Cache the offset, refresh every minute. Then effective time = new Date(now + offset*60000), use getUTCHours etc. plus now%1000 for ms.

- Hand angles:
  - seconds s = sec + ms/1000 → angle = -(s/60)*2π (clockwise).
  - minutes m = min + s/60.
  - hours h = (hr%12) + m/60.
  - GMT hand: 24-hour of second time zone: angle = -(h24 + m/60)/24 * 2π.
- Movement mode: "机械扫秒" (continuous sweep) vs "石英跳秒" (tick). Default sweep. Also add subtle hand bounce on tick? Keep simple: toggle.

5. Date/day display:
- Canvas texture for date (number) and day (3-letter or Chinese 星期X). Update when value changes. Day shown as "MON" etc., or Chinese "周一"? Provide both? Show Chinese "星期三" in window? Windows are small; use 3-letter English + Chinese below in UI. Day window canvas: show like "WED". Date window: number.
- Also show full date in UI panel: e.g., 2025年… 星期…

6. Dual time zone UI:
- Two selects with common time zone list (Asia/Shanghai, Asia/Tokyo, Europe/London, Europe/Paris, America/New_York, America/Los_Angeles, UTC, Asia/Dubai, Australia/Sydney, Asia/Singapore, etc.).
- "本地时间" (main hands) and "GMT 第二时区" (GMT hand + digital sub-readout).
- Swap button.
- Digital readout with large numerals for both zones, live update.
- Bezel shows 24-hour scale; GMT hand points to it.

7. Atmosphere:
- Background: deep gradient (CSS behind transparent canvas, or scene fog + large gradient plane). Use renderer alpha and CSS radial gradient background — layered: deep teal/charcoal with vignette, subtle animated grain? Keep GPU light: CSS gradient + canvas with alpha.
- Add floating dust particles in 3D (Points with additive blending, slow drift) — a lively background.
- Floor: subtle reflective disc? Can use a shadow-catcher plane with ShadowMaterial for soft shadow. Yes: soft shadow under watch.
- Watch idle animation: gentle floating (y bob) and slow rotation when not interacting; pause on user drag (OrbitControls 'start' event), resume after idle timeout.
- Light: key directional with shadow, rim light, RoomEnvironment for reflections.

8. Page layout:
- Full-screen canvas. Overlay UI: top-left brand plate ("CHRONOS · 双时区三维腕表" style), right/bottom control panel with time zones, digital time, toggles (扫秒/跳秒, 自动旋转, 灯光主题?). Keep controls: 时区 A, 时区 B, swap, movement toggle, auto-rotate toggle, "对时" (sync) maybe unnecessary.
- Micro-interactions: button hover states, styled selects, panel with backdrop blur (only the panel, not site-wide glass — a single functional panel with slight translucency is OK, but let's use a solid dark panel with a border instead). Use dark panel #10181c with a gold hairline border and subtle shadow.
- Loading screen with spinner while textures/environment build, fade out.
- Small hint: "拖拽旋转 · 滚轮缩放".

Colors: deep charcoal-green background, dial in midnight green sunburst with gold accents (gold hands/indices), red-tipped second hand, orange GMT arrow. Background: deep petrol/ink with warm vignette. Accent gold #d4af37-ish.

Fonts: Marcellus (display, watch-like serif caps) + Noto Sans SC (body). Import Google fonts. If offline, fall back gracefully.

9. Testing notes: describe how to run (open the file directly — but module scripts from CDN need internet; file:// works with CDN imports, yes, importmap + modules work from file:// as long as they're from https CDN. OrbitControls import from jsdelivr works). Provide steps: save as watch.html, open in modern browser, needs internet for three.js CDN and fonts. Provide fallback: if CDN blocked, can swap URL.

Also keyboard shortcuts: Space to toggle sweep/tick? Maybe. Keep: R to reset view.

Now let me carefully write the code. Main pitfalls:
- ExtrudeGeometry hand shapes: define shape in XY plane, extrude in Z, then rotate so hand lies in dial plane and points along +Y at angle 0. Actually simpler: build hands as flat geometry pointing up (+Y), pivot at origin; then rotation.z = -angle.
- Dial plane at z = some value; watch faces +Z. Watch group: dial faces camera at z=0.6 or so.
- Case profile:
  - Case body: CylinderGeometry(r=2.05, r=2.1, h=0.55) rotated so axis is Z (cylinder axis is Y by default; rotate x by 90°).
  - Bezel: TorusGeometry(radius 1.95, tube 0.16) at front z.
  - Actually build with axis along Z: create cylinder and rotateX(Math.PI/2).
- Dial radius 1.75, z=0.28.
- Crystal: dome of SphereGeometry with phiStart etc.: sphere radius R, scale flat, positioned so cap covers dial; or use LatheGeometry dome. Simple: SphereGeometry(2.2, 64, 32, 0, 2π, 0, 0.55) and scale z? Let's do: create sphere, take top cap (thetaLength ~0.6), rotate so cap faces +Z, position so base circle radius matches ~1.9. Sphere radius Rs, thetaLength θ, cap base radius = Rs*sin θ. Want base radius 1.85: θ=0.6 → sinθ=0.5646 → Rs=3.28. Dome height = Rs(1-cosθ)=3.28*0.175=0.575 — too tall; scale z by 0.5 → 0.29. OK.
- Transmission material may have issues on some GPUs; fallback: if renderer.capabilities.isWebGL2 is false, use transparent opacity. Use MeshPhysicalMaterial with transmission; generally fine.

- Canvas dial texture: draw sunburst: radial gradient + many radial lines with varying alpha for sunray effect; minute track ticks; brand text "MERIDIAN" + "自动机械 · GMT"; outer chapter ring. Also small "SWISS MADE"? Use fictional brand "子午 · MERIDIAN". Cute: brand "MERIDIAN 子午" fits time zones.

- Day/date canvas: create a small canvas (128x96) with white/cream background, black text; update via texture.needsUpdate.

- GMT bezel ring: RingGeometry inner 1.85 outer 2.15? Actually bezel top surface: flat ring at z=0.56 with canvas texture 24 numerals. Hmm, torus bezel + flat ring top with numerals texture. Let me do: bezel = cylinder ring (Cylinder with hole? no) — use LatheGeometry for a nice profile. Simpler composition:
  - Outer bezel ring: Torus(1.98, 0.14) at z=0.5, polished metal.
  - GMT numerals ring: flat RingGeometry(1.62, 1.95) at z=0.565, dark ceramic texture with 24-hour numerals. Slight tilt? Flat is fine.
  Hmm, dial radius 1.75 > ring inner 1.62 overlap. Adjust: dial radius 1.55, rehaut ring 1.55→1.7 dark, GMT ring 1.7→2.0, torus around 2.0. Let me define:
  - Case outer radius 2.2, height 0.6 (z from -0.3 to 0.3).
  - Dial radius 1.52, z=0.30.
  - Rehaut (chapter ring) ring 1.52→1.72, z=0.34, dark texture with minute ticks? Minute track can be on the dial itself. Keep rehaut as a simple angled dark metal ring — skip complexity: flat ring with gradient texture.
  - GMT bezel insert: Ring(1.72, 2.02), z=0.46, canvas with 24-hour numerals + triangle at 0.
  - Bezel metal edge: torus(2.05, 0.13) at z=0.45.
  - Crystal dome base radius ~1.95, sitting at z=0.48.
  
  Indices at radius ~1.25, length 0.22.
  Hand lengths: hour 0.85, minute 1.3, second 1.45, gmt 1.6 (reaches bezel numerals).

- Strap: links along a circular arc in the Y-Z plane? Watch faces +Z, strap extends in ±Y and curves back in -Z (as if wrapping around a wrist). Circle center at (0, 0, -R0), radius R0+? Let wrist radius Rw = 3.2, center at (0,0,-2.6)? Parameterize: for t in 0..1, angle φ from 0.15 to 1.3 rad: position y = sin(φ)*Rw, z = cos(φ)*Rw - Rw (so z=0 at φ=0). Upper strap φ positive (y>0), lower negative. Links: rounded boxes of width 1.3 (x), length 0.34 along arc, thickness 0.16, oriented tangent. Center links polished gold, outer brushed steel — alternate two-tone: each row = 3 boxes (center wider gold, sides steel). Looks like a jubilee bracelet. About 9 links per side. End: clasp hint.

- Lugs: 4 small tapered boxes connecting case and strap at y=±2.0.

- Crown at x = 2.25, z=0: cylinder axis X, radius 0.22, height 0.18, plus stem. Knurling: use CylinderGeometry(0.24,0.24,0.2,20) with flatShading.

- Caseback: cylinder radius 2.0, h 0.1, z=-0.33.

- Shadow: directional light from (4,6,6), castShadow; watch casts onto ShadowMaterial plane at z… floor at y=-3.4 (horizontal), watch floats above. Watch is vertical (facing +Z), so shadow falls on floor below. Watch center y ~0.4, floor y=-2.8.

Idle animation: watchGroup.rotation.y = sin(t*0.3)*0.25 + base, gentle bob. But OrbitControls rotates camera; auto-rotate camera is also nice. Let's do: subtle watch sway (rotation.x/y small) + optional camera autoRotate toggle (default on, pause on interaction via controls events, resume after 3s idle).

Particles: 200 points in a sphere of radius 12, additive, slow upward drift + wrap, size attenuation, gold-ish color, low opacity.

Postprocessing: skip (keep single file light); rely on tone mapping.

UI layout:
- Top-left: brand block: small gold rule, "MERIDIAN 子午" large display, sub "双时区 · 三维机械腕表".
- Bottom-left: hint text.
- Right side: control panel (fixed right, vertically centered):
  - Section 主时区 (local): select + large digital time HH:MM:SS + date line (2025年6月… 星期三).
  - Section GMT 第二时区: select + digital time + date.
  - Swap button ⇄.
  - Toggles: 扫秒模式 (机械/石英), 自动环游 (camera auto rotate), maybe 表灯 (light warmth) — keep 2 toggles + a "正视表盘" reset button.
- Panel styling: dark #0d1416ee, 1px border rgba(212,175,55,.35), sharp corners with small radius 10px, gold headings with letter-spacing.

Digital time numerals: use tabular numbers via font-variant-numeric; Marcellus isn't tabular; use 'Space Grotesk' for numerals? Add a third font? Marcellus for headings, 'Manrope' for body, numerals can be Manrope 600 tabular-nums (does Manrope support tnum? unsure). Use font-feature-settings "tnum". If unsupported, minor jitter acceptable. Or fixed-width via monospace fallback for seconds: use 'IBM Plex Mono'? Keep font count small: Marcellus + Noto Sans SC + maybe system mono for numerals: font-family: ui-monospace, 'SF Mono', Consolas, monospace — clean and guarantees tabular. Actually, a refined choice: numerals in 'Marcellus' look elegant; jitter is minor. Use monospace stack with letter-spacing for digital readout — technical contrast with serif brand. Good.

Time zone list: curated ~14 with Chinese labels:
- UTC 协调世界时
- Asia/Shanghai 北京
- Asia/Hong_Kong 香港
- Asia/Tokyo 东京
- Asia/Singapore 新加坡
- Asia/Dubai 迪拜
- Europe/Moscow 莫斯科
- Europe/Paris 巴黎
- Europe/London 伦敦
- America/New_York 纽约
- America/Chicago 芝加哥
- America/Los_Angeles 洛杉矶
- Pacific/Auckland 奥克兰
- Australia/Sydney 悉尼

Default: primary = Asia/Shanghai (or guess via Intl.DateTimeFormat().resolvedOptions().timeZone), GMT = Europe/London? Default primary = user's local tz (resolvedOptions), GMT = America/New_York. If user's tz is in the list, select it, otherwise add dynamically. Simpler: primary default = resolved tz (add option dynamically if missing), GMT default = 'Europe/London'. Good.

Date/day windows on dial: which time zone does the watch display? Primary zone. Update canvas when day/date changes in primary zone.

Day window at 12 o'clock: rectangle width 0.55, height 0.26, y=0.95. Date at 3 o'clock: x=1.05, 0.4x0.34. Skip index at 12? Keep index at 12 but move day window inside (y=0.85), index at radius 1.25 — no conflict. At 3 o'clock, replace index with date window (skip index at 3).

Hand shape:
- Hour: dauphine: Shape [(-0.055,0.15),(0,1.0),(0.055,0.15),(0.03,-0.18),(-0.03,-0.18)] hmm let's make it simpler: elongated kite. Points: (0, len), (w, 0.1), (w*0.6, -tail), (-w*0.6,-tail), (-w,0.1). Extrude depth 0.035, small bevel.
- Minute: similar thinner and longer.
- Second: thin box: use Shape rectangle width 0.012, length 1.5 + tail 0.45, plus circular counterweight (torus/cylinder) at tail. Build as group: box geometry.
- GMT: Shape with arrow tip: shaft width 0.035 up to 1.2, then arrow triangle 1.2→1.62 width 0.14, plus tail. Different color: orange lacquer (emissive-ish).
- Center caps: small gold cylinder stack.

Hand materials: gold metal (hour/minute, with facet illusion via two-tone material — keep single gold, flatShading false). Second hand: crimson with gold? Classic red second hand + gold GMT might clash; GMT orange, second hand steel-blue? Let's do: hour/minute polished gold; GMT hand orange lacquer; second hand red lacquer with gold counterweight? Two warm accents is fine. Actually make second hand steel-blue for contrast against gold — nice: heat-treated blue second hand is horological. Second hand = #2f6fd8 metallic blue. GMT = warm orange #ff7a2f. Good contrast on green dial.

Dial color: deep green sunburst (#0d3b2e → #06201a radial with rays). Rehaut black. Bezel insert: dark green ceramic with gold numerals? Or black with gold numerals for 24-hour. Let's do dark charcoal-green with cream numerals.

Background CSS: radial-gradient at 30% 20% #16221f → #0a0f0e 70%, plus vignette, plus subtle noise via repeating-conic? Use SVG noise data URI overlay with low opacity — cheap and adds texture. Also soft gold glow behind watch via CSS radial.

Loading overlay: brand monogram, spinner ring, fade out after first frame.

Performance: shadows 2048, pixelRatio min(devicePixelRatio, 2).

Resize handler.

Also add double-click on canvas → reset camera.

Auto rotate: controls.autoRotate = true, speed 0.8; on 'start' set false and clear timer; on 'end' timer 4s → true (if toggle on).

Movement toggle: sweep vs tick. Tick: seconds advance floor(s) + easing bounce: use step with small overshoot: pos = floor + easeOutBack(min(1, frac*8)). Implement: if tick mode, displayed seconds = floor(s) + easeOutBack(min(1, frac*8)) — gives mechanical snap. easeOutBack with small overshoot 1.2.

Also subtle tick of minute/hour hands — they are continuous anyway.

Date change animation: quick flip? Just update texture.

Extra "life": second hand shadow on dial (castShadow on hands, dial receive — but dial is textured plane; enable receiveShadow, light angle gives hand shadows — nice depth. Shadow camera tight.)

Also add slight parallax: hands at different z (hour 0.06, minute 0.10, gmt 0.08, second 0.14 above dial at 0.30 → absolute z). Crystal at 0.48+.

Let me write out coordinates:
- Dial plane z = 0.30.
- Hour pivot z 0.345, minute 0.375, GMT 0.33, second 0.405. Cap at 0.42.
- GMT under hour hand? Usually GMT is under hour/minute or above; put GMT at 0.33 (just above dial), then hour, minute, second. Good.

Bezel insert z=0.47, torus z=0.47 center, tube 0.13 → top at 0.60. Crystal base z=0.50, dome up to ~0.78. Case front z=0.30.. case body from -0.28 to 0.34.

Case side profile via Lathe may be nicer but cylinder stack is fine:
- Body: Cylinder(rTop=2.16, rBottom=2.05, h=0.56), rotated X 90°, z center 0.02. Metal brushed steel.
- Caseback: Cylinder(1.9,1.95,0.12) z=-0.32.
- Bezel base ring: Cylinder(2.2,2.16,0.14) z=0.37? Then torus on top. Let me simplify: torus bezel radius 2.05, tube 0.16, z=0.50 — covers edge. Insert ring inside.

Lugs: box (0.34 x 0.55 x 0.22) at (±0.78, ±2.15, -0.02), rotated slightly in z (tilt outward). Actually lugs pair at top and bottom: positions (±0.72, 2.12, 0) and (±0.72, -2.12, 0), tilted to follow strap curve (rotate x by ∓0.25). Use capsules: CapsuleGeometry(0.11, 0.5) rotated z 90°? Lugs run along Y. Capsule axis Y default: radius 0.12, length 0.45, at those positions, rotate x slightly.

Strap arc: wrist radius Rw=3.4, center z = -3.4+0.05. Links at angle φ: pos y=±(sin φ * Rw), z = cos φ * Rw - Rw + 0? At φ=0: z=0. Link at φ0=0.16 to start. Link orientation: rotate x by ∓φ so tangent. Link size: x 1.35 (width), y 0.30 (arc length), z 0.14. 3-piece: center piece width 0.55 gold, slightly thicker z 0.15, sides each 0.38 steel. Gap between links 0.03. Step of Δφ = (0.33)/Rw ≈ 0.097 rad. Count 10 per side, φ up to ~1.13 rad (65°) — nice curve. Fade out last link? Add end-cap link slightly larger (clasp). OK.

Bracelet material: steel brushed (metalness .9, roughness .35), gold (color 0xd9b36c, metalness 1, roughness .25).

Environment intensity: material envMapIntensity ~1.2.

Lights:
- Key: DirectionalLight (0xfff1dd, 3.0) pos (5,7,6), castShadow, shadow camera ortho ±6, mapSize 2048, radius 4 (PCFSoft).
- Fill: DirectionalLight (0xbfd9ff? cool, 0.8) from (-6,2,4), no shadow.
- Rim: SpotLight from behind (-3,4,-6) gold-ish 1.5 for edge highlights.
- Ambient via RoomEnvironment (scene.environment). Set scene.environmentIntensity? (r160 supports scene.environmentIntensity? environmentIntensity added in r163). Use renderer + material envMapIntensity instead. Use three 0.165? Let me pin 0.160.0 and control via material envMapIntensity (works). Set toneMappingExposure 1.1.

Background: keep renderer alpha:true, scene.background = null, CSS gradient shows through. Fog unnecessary.

Particles: BufferGeometry positions random in box 24x14x16 around, PointsMaterial size 0.05, color 0xd8b877, transparent opacity .5, additive blending, depthWrite false. Animate: y += speed, wrap. Store velocity array.

Ground shadow plane: PlaneGeometry(30,30) rotated x -90°, y=-3.2, ShadowMaterial opacity 0.35.

Also subtle "pedestal" glow ring on floor? Can add faint radial gradient texture plane — skip, shadow is enough; maybe add a soft gold point light below.

Camera: fov 38, pos (3.2, 1.4, 7.4), target (0,0.2,0). minDistance 4, maxDistance 14, maxPolarAngle 1.65.

Reset view button: tween camera to default (lerp over 600ms) — implement simple animation.

Now digital clock UI updates every frame (cheap textContent update only when string changes).

Date line: use Intl in that tz with locale zh-CN: new Intl.DateTimeFormat('zh-CN',{timeZone, year:'numeric',month:'long',day:'numeric',weekday:'long'}).format(date).

Day window text: weekday short English (['SUN','MON',...]) or Chinese 周X? Window is small; use English 3-letter classic. Date window: number.

Compute tz offset every frame? Cache offset per tz, refresh every 30s. Offset function as above.

Edge: parts.hour '24' handling.

Structure code with clear sections and comments (Chinese comments OK).

Also handle WebGL failure: try/catch → show message.

Now HTML/CSS. Fonts:
```html
<link href="https://fonts.googleapis.com/css2?family=Marcellus&family=Noto+Serif+SC:wght@600&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
```
Display: Marcellus + Noto Serif SC fallback for Chinese headings. Body: Noto Sans SC.

Panel width ~300px, right 24px, top 50% translateY(-50%). On small screens, panel becomes bottom sheet (media query: right auto, bottom 0, full width, horizontal scroll?). Provide basic responsiveness.

Brand top-left:
```
MERIDIAN
子 午 · 双时区腕表
```
With gold hairline.

Bottom-left hint: "拖拽旋转 / 滚轮缩放 / 双击复位".

Toggle UI: custom switch (checkbox styled) — gold accent.

Buttons: 交换时区 ⇄, 复位视角.

Also display a small "GMT" label near the orange hand? Legend in panel: color dot legend: 金色=本地时分, 橙色=GMT 24时, 蓝色=秒针. Nice touch.

Loading overlay with rotating bezel-like ring CSS spinner + "正在装配机芯…". Fade.

Now let me write the JS. Careful with three r160 API:
- THREE.CapsuleGeometry exists (r142+). Good.
- RoomEnvironment: import { RoomEnvironment } from '.../environments/RoomEnvironment.js'; new RoomEnvironment() — in r160 RoomEnvironment constructor doesn't need renderer arg (older took renderer? RoomEnvironment() no arg; PMREMGenerator(renderer).fromScene(new RoomEnvironment(), 0.04).texture). In some versions RoomEnvironment(renderer)? Signature: class RoomEnvironment extends Scene { constructor() } — historically no arg. r160 example: `const environment = new RoomEnvironment( renderer ); pmremGenerator.fromScene( environment )`? Actually docs: `scene.environment = pmremGenerator.fromScene( new RoomEnvironment( renderer ), 0.04 ).texture;` — some versions of RoomEnvironment accept renderer param (added later, ignored earlier). Passing renderer is safe either way? If constructor ignores params, fine. Let me pass renderer.

Hmm, in r160 RoomEnvironment signature is constructor( renderer = null )? Let me just call new RoomEnvironment() — works across versions. Because renderer param was optional for… actually I recall the example webgl_materials_physical_transmission used `new RoomEnvironment()` with no arg. Keep no-arg.

- OutputColorSpace: renderer.outputColorSpace = THREE.SRGBColorSpace (r152+). Good.
- texture.colorSpace = THREE.SRGBColorSpace for canvas textures.

Anisotropy for dial texture: renderer.capabilities.getMaxAnisotropy().

Hand shadow: castShadow true on hand meshes; dial receiveShadow true; but thin extruded hand shadows are subtle — OK.

Second hand bounce in tick mode: implement in update.

Angles: rotation.z = -angle, angle measured clockwise from 12. Hand geometry points +Y. rotation.z negative = clockwise viewed from +Z. Correct.

GMT hand: 24-hour: angle = ((h24 + m/60 + s/3600)/24)*2π. Points once around per 24h; at 12:00 points to "24/0" triangle at top. Bezel 24 numerals with 24 at top. Good.

Bezel texture: canvas 1024, ring drawing: background dark #101512, ticks every 1.5h? Standard GMT bezel: numerals every 2 hours: 2,4,...,24. Draw numerals every 2h, small ticks every 1h. Triangle at top (24). Cream color #e8dcc0.

Dial texture drawing (1024):
- Background radial gradient, center #1a5c48 → edge #07231c (rich green).
- Sunrays: for i in 0..720: line from center at angle i*0.5°, alpha varying by sin, strokeStyle rgba(255,255,255,0.02..0.05) length to edge — draw with gradient? Simpler: rotate and draw thin triangles alternating alpha. Use: for i<360: ctx line with alpha = 0.03+0.03*sin(i*7). Enough.
- Outer minute track ring at radius ~0.93R: small ticks; 5-minute marks longer.
- Text: brand at y 0.32R: "MERIDIAN" Marcellus (canvas font: '600 64px Marcellus, serif' — canvas can use webfont if loaded; ensure via document.fonts.ready before building dial, else fallback serif acceptable. Let me await document.fonts.ready before init to get fonts right, with timeout fallback.)
- Below brand: "子 午" small Chinese + "AUTOMATIC · GMT" tiny.
- Bottom: "SWISS LEVER · 21 JEWELS"? Use fictional "21 JEWELS · 双时区". Keep classy.

Date/day window: draw recessed frame: on dial texture, draw window cutout (dark rectangle border) at position? Better: separate plane with white canvas + gold frame mesh (thin box frame). Let's do: frame = 4 thin gold boxes or a ring; simpler: plane slightly larger behind gold color, then white canvas plane on top with inset. Implementation: group at position: backPlane (0.46x0.36 gold metal), windowPlane (0.40x0.30 canvas). Position at 3 o'clock: x=1.02, y=0, z=0.305/0.31. Day at 12: y=0.92, wider 0.62x0.28.

Skip indices at 3 o'clock. Indices: at angles i*30°, skip i=3 (90°… index 3). 12 double baton: two boxes offset x ±0.06.

Index geometry: BoxGeometry(0.07, 0.26, 0.05), positioned at radius 1.28, rotated z so length is radial: box length along Y, rotate z = -angle. Slight lume? Gold with small emissive 0. Add small lume dots past indices? Skip.

Brand text on dial already includes minute track.

Now UI wiring:
- selLocal, selGmt: options from ZONES array; set defaults; on change update tzLocal/tzGmt, refresh offset, update date window immediately.
- Swap: swap values + dispatch change.
- chkSweep (checked = sweep). chkOrbit (auto rotate). btnReset.

Digital update: elLocalTime textContent = HH:MM:SS (with seconds), date line below. Same for GMT. Use requestAnimationFrame loop; cache last string.

Also show tz offset label like "UTC+8": compute from offset minutes.

Legend dot colors match materials.

Now sizes: keep code organized. Let me write.

Also note: `renderer.shadowMap.enabled = true; type PCFSoftShadowMap`.

Transmission material renders transparent via transmission pass; with alpha:true canvas and CSS background, transmission samples scene render target — background null → transmissive object sees scene background (transparent) — crystal may look odd but mostly shows dial behind (dial is in scene, transmission samples framebuffer of scene without transmissive objects). Should be fine.

Crystal: MeshPhysicalMaterial({ transmission:1, thickness:0.6, roughness:0.03, ior:1.52, clearcoat:1, attenuation? none, transparent:true? not needed with transmission; set opacity 1 }). Add subtle envMapIntensity 1.5. Also side: front only. Slight tint? clear.

Fallback if transmission heavy: acceptable.

Also inner reflection ring: "rehaut" cone: CylinderGeometry(1.72,1.55,0.10, openEnded) dark metal, positioned z 0.36, tilted wall — use Cylinder open-ended with radiusTop>radiusBottom rotated X 90: axis z; radiusTop (front) 1.72, bottom 1.55 → wall slants outward. Actually cylinder axis after rotateX(π/2) is along Z: the "top" faces +Z. Good.

Now strap link loop:

```js
const Rw=3.45, cz=-Rw+0.02;
for side of [1,-1]:
 for k=0..9:
   phi = 0.18 + k*0.098;
   y = side*Math.sin(phi)*Rw; z = Math.cos(phi)*Rw + cz;
   link group at (0,y,z), rotation.x = -side*phi (so tangent: as phi increases, direction of travel rotates around x axis; for side=+1 (top), link tilts back: rotation.x = -phi? Let me verify: point on circle center (0,0,cz) radius Rw: p=(0, sinφ Rw, cz+cosφ Rw). Tangent dp/dφ = (0, cosφ Rw, -sinφ Rw). Link's local +Y should align with tangent → rotate around X by angle θ, R_x(θ)·(0,1,0) = (0, cosθ, sinθ). Want ∝ (0, cosφ, -sinφ) → θ = -φ. So rotation.x = -φ for side +1; for side -1: p=(0,-sinφ Rw, cz+cosφ Rw), tangent along increasing φ: (0,-cosφ,-sinφ) → want (0,cosθ,sinθ)= (0,-cosφ,-sinφ) → θ = π-φ… but link is symmetric in Y, so θ = φ+π ≡ visually flip; use rotation.x = φ + Math.PI? That would flip z normal. Simpler: for side -1 use rotation.x = Math.PI + φ? Hmm, link box is symmetric in y and z? Box is symmetric; orientation matters for which face points outward (+Z face should point outward from wrist center, i.e., away from cz). Outward normal = (p - center)/Rw = (0, side sinφ, cosφ). R_x(θ)·(0,0,1) = (0, -sinθ, cosθ). Want (0, side sinφ, cosφ) → -sinθ = side sinφ, cosθ = cosφ → θ = -side·φ. For side=+1: θ=-φ ✓ matches above. For side=-1: θ=+φ gives normal (0, sinφ·? -sin(φ)= -sinφ ✓ side=-1 wants (0,-sinφ,cosφ) ✓, cos ✓. So rotation.x = -side*φ works for both.
```

Great: rotation.x = -side*phi.

Lugs similarly at φ≈0.10: y=±sin(0.10)*3.45≈0.34? Too close — lug should be at y≈2.1 (case radius 2.2). Mismatch: strap should start near case edge y≈2.1. With Rw=3.45, need sinφ=2.1/3.45=0.61→φ0=0.656 rad. Then links from φ0+0.05. Let me set φ0 = 0.66, step 0.095, count 8 → φmax≈1.33 (76°). z at φ0: cos(0.66)*3.45+cz = 0.79*3.45-3.43= -0.70. So strap starts slightly behind case — but lug bridges from case (z≈0) to there. Hmm, big z drop. Reduce curvature: Rw=4.6, cz=-4.55: φ0=asin(2.15/4.6)=0.485; z0=cos(.485)*4.6-4.55=4.07-4.55=-0.48. Better. Step Δφ=0.34/4.6=0.074; count 9 → φmax≈1.15 (66°), end y=sin(1.15)*4.6=4.2, z=cos(1.15)*4.6-4.55=-1.17… wait cos(1.15)=0.408→ z=1.88-4.55=-2.67. Nice wrap.

Lugs: capsules from case to first link: position (±0.72, ±2.05, -0.12), rotation.x = ∓0.35, length 0.5. Also springbar hint skip.

Crown at +X (3 o'clock side) — but date also at 3; crown at x=+2.3 fine.

Also add correctors: small pushers at 2 and 4? Skip.

Caseback engraving: skip (not visible mostly).

Now reset camera animation: save default camera pos/target; on click lerp over 700ms with easing; disable controls meanwhile? Just lerp in loop with anim state.

Double-click also triggers reset.

Idle resume: variable userInteracting; controls.addEventListener('start',...) 'end'...

Write digital clock with padded.

Offset display: format "UTC±H(:MM)".

Now let me finalize time zone list with labels:
```js
const ZONES=[
 ['Pacific/Auckland','奥克兰'],['Australia/Sydney','悉尼'],['Asia/Shanghai','北京'],['Asia/Hong_Kong','香港'],['Asia/Singapore','新加坡'],['Asia/Tokyo','东京'],['Asia/Seoul','首尔'],['Asia/Dubai','迪拜'],['Asia/Kolkata','新德里'],['Europe/Moscow','莫斯科'],['Europe/Paris','巴黎'],['Europe/London','伦敦'],['UTC','协调世界时'],['America/Sao_Paulo','圣保罗'],['America/New_York','纽约'],['America/Chicago','芝加哥'],['America/Denver','丹佛'],['America/Los_Angeles','洛杉矶'],['Pacific/Honolulu','檀香山'],
];
```
Kolkata is +5:30 good for offset test.

Default local: Intl.DateTimeFormat().resolvedOptions().timeZone; if not in list, prepend with generic label (city from tz string).

Now date/day canvas update: function setWindow(tex, canvas, text). Date: large number centered. Day: 3-letter.

Also update digital date line: full format.

Potential issue: Intl with timeZone 'UTC' works.

Offset cache: map tz→{offset, t}. Refresh if >60s.

getZoneTime(tz):
```js
function zoneNow(tz){
  const off = getOffset(tz); // minutes
  const d = new Date(Date.now() + off*60000);
  return { h:d.getUTCHours(), m:d.getUTCMinutes(), s:d.getUTCSeconds(), ms:d.getUTCMilliseconds(), day:d.getUTCDay(), date:d.getUTCDate(), month:d.getUTCMonth(), year:d.getUTCFullYear(), offset:off };
}
```
ms: since offset computed at second resolution, ms comes from Date.now()%1000: better: const nowMs = Date.now(); secFloor = Math.floor(nowMs/1000); d = new Date((secFloor + off*60)*1000); ms = nowMs - secFloor*1000. Good — smooth.

getOffset uses formatToParts at secFloor*1000.

Angle update:
```js
const L = zoneNow(tzLocal);
const secL = L.s + L.ms/1000 (sweep) or tick easing;
secAngle = secL/60*TAU;
minAngle = (L.m + (L.s+L.ms/1000)/60)/60*TAU;
hrAngle = ((L.h%12) + L.m/60 + L.s/3600)/12*TAU;
const G = zoneNow(tzGmt);
gmtAngle = ((G.h + G.m/60 + G.s/3600)/24)*TAU;
```
Set rotation.z = -angle.

Tick easing:
```js
function tickSec(s, ms){ const f = ms/1000; const k=Math.min(1, f*6); const e = 1.70158; const back = 1 + (e+1)*Math.pow(k-1,3) + e*Math.pow(k-1,2)? 
```
easeOutBack(k)=1 + c3*(k-1)^3 + c1*(k-1)^2? Standard: c1=1.70158, c3=c1+1; f(x)=1+c3*pow(x-1,3)+c1*pow(x-1,2). At x=0 → 1 - c3 + c1 = 1-2.70158+1.70158=0 ✓. Overshoot ~1.1 around x≈0.7. Use min(1,f*7).
Displayed seconds = s + easeOutBack(min(1, ms/1000*7)). Slight overshoot past next second then settle — mechanical feel. But at s=59 overshoot → 59.95→ angle slightly past 0 → fine (wrap).

Minute/hour continuous regardless.

Update date window when L.date/day changes: keep last.

UI digital: localStr = pad(h)+':'+pad(m)+':'+pad(s). Date line: `${year}年${month+1}月${date}日 · ${WEEK_CN[day]}`. WEEK_CN=['日','一',...] as 星期X. GMT line maybe use Intl with timeZone for correctness — same as computed; use computed values.

Offset label: fmtOff(off): sign, hh = Math.trunc(abs/60), mm=abs%60 → `UTC${sign}${hh}${mm? ':'+mm:''}`.

Now build UI HTML.

Panel structure:
```html
<aside id="panel">
  <div class="p-head">机 芯 控 制 台<span>MVT-024</span></div>
  <section class="tz-block">
    <div class="tz-label"><i class="dot dot-gold"></i>本地时间 · LOCAL</div>
    <select id="selLocal"></select>
    <div class="digital" id="dLocal">00:00:00</div>
    <div class="dateline" id="dateLocal">—</div>
  </section>
  <button id="btnSwap">⇅ 交换时区</button>
  <section class="tz-block">
    <div class="tz-label"><i class="dot dot-orange"></i>第二时区 · GMT</div>
    <select id="selGmt"></select>
    <div class="digital gmt" id="dGmt">00:00:00</div>
    <div class="dateline" id="dateGmt">—</div>
  </section>
  <div class="switches">
    <label class="switch"><input type="checkbox" id="chkSweep" checked><span class="track"></span>扫秒 · 机械模式</label>
    <label class="switch"><input type="checkbox" id="chkOrbit" checked><span class="track"></span>自动环游</label>
  </div>
  <div class="btn-row">
    <button id="btnReset">复位视角</button>
  </div>
  <div class="legend">
    <span><i style="--c:#e6c47f"></i>时分针</span>
    <span><i style="--c:#ff8a3d"></i>GMT 24时</span>
    <span><i style="--c:#5b8def"></i>秒针</span>
  </div>
</aside>
```

Brand:
```html
<header id="brand">
  <div class="brand-rule"></div>
  <h1>MERIDIAN</h1>
  <p class="brand-sub">子午 · 双时区三维腕表</p>
  <p class="brand-cal">CAL. QW-24 自动机芯 · 日期 / 星期</p>
</header>
```

Hint bottom-left: `拖拽旋转 · 滚轮缩放 · 双击复位视角`.

Loader:
```html
<div id="loader"><div class="l-ring"></div><div class="l-text">MERIDIAN<span>正在装配机芯…</span></div></div>
```

CSS: color variables: --bg0 #0b100f, --gold #d4af6a, --ink #e8e4d8, --panel #0f1514f2.

Background: layered radial gradients + noise SVG.

Now write JS module. Careful with order: define helpers, build scene after fonts ready.

Also handle context: `const canvas = document.getElementById('scene')`.

Renderer size: window.

Let me write it all out concretely.

Canvas texture functions:

```js
function makeCanvas(size){ const c=document.createElement('canvas'); c.width=c.height=size; return [c,c.getContext('2d')]; }
```

Dial texture (1024):
```js
function dialTexture(){
 const S=1024,[c,x]=makeCanvas(S); const R=S/2;
 // base
 let g=x.createRadialGradient(R,R,40,R,R,R);
 g.addColorStop(0,'#1c6b52'); g.addColorStop(.55,'#0f4433'); g.addColorStop(1,'#06231b');
 x.fillStyle=g; x.fillRect(0,0,S,S);
 // sunrays
 x.save(); x.translate(R,R);
 for(let i=0;i<720;i++){ const a=i*Math.PI/360; x.save(); x.rotate(a);
   x.fillStyle=`rgba(255,255,240,${0.012+0.02*Math.abs(Math.sin(i*0.35))})`;
   x.fillRect(0,-1,R,2); x.restore(); }
 x.restore();
 // vignette ring
 g=x.createRadialGradient(R,R,R*0.72,R,R,R); g.addColorStop(0,'rgba(0,0,0,0)'); g.addColorStop(1,'rgba(0,0,0,.55)'); x.fillStyle=g; x.fillRect(0,0,S,S);
 // minute track at r=0.90R..
 x.save(); x.translate(R,R);
 for(let i=0;i<60;i++){ const a=i/60*TAU; const maj=i%5===0;
   x.save(); x.rotate(a);
   x.fillStyle= maj? '#e9dcb8':'rgba(233,220,184,.55)';
   const len= maj? 34:16, w= maj?5:2;
   x.fillRect(-w/2, -R*0.955, w, len); // ticks inward from top edge
   x.restore(); }
 x.restore();
```
Wait, ticks: draw at top then rotate — rect y=-0.955R, height len inward (y increasing toward center) ✓.

 Text:
```js
 x.textAlign='center';
 x.fillStyle='#ead9ac';
 x.font='600 74px Marcellus, serif'; x.fillText('MERIDIAN', R, R-205);
 x.font='500 40px "Noto Serif SC", serif'; x.fillText('子 午', R, R-140);
 x.font='400 26px Manrope, sans-serif'; x.fillStyle='rgba(233,220,184,.75)';
 x.fillText('AUTOMATIC · DUAL TIME', R, R-100);
 x.font='400 24px Manrope, sans-serif';
 x.fillText('21 JEWELS · CHRONOMETER', R, R+250);
```
Positioning relative: R=512; brand above center y=307 (R-205). Day window at 12 will be at y≈0.92/1.52*… mapping: dial radius 1.52 world = 512 px → 1 world = 336.8 px. Day window center y=0.92 world → 310 px above center → y_px=512-310=202. Brand at y=307 px (0.61 world above center) — below day window? Day window at 202px (top area), brand at 307 — window above brand. OK spacing: window half-height 0.13 world=44px → window spans 158..246; brand text at 307 fine.

Date window at x=1.02 world=343 px right: x_px=512+343=855, y=512. Minute track ticks at that area will be covered by window — draw window frame later? Window plane covers. But ticks behind window edge may peek; window plane (0.40x0.30 world = 135x101 px) covers area x 787..922 — ticks at radius 0.955R=489 px from center → x=512+489·cos…at 3 o'clock tick x≈1001 — outside window. Fine.

Also draw small cross/brand at center.

Bezel texture (1024, ring):
```js
 bg #14181a? dark charcoal-green #101715; ticks every 1h small, numerals every 2h: value 2..24. Font Marcellus 60px cream. Triangle at top for 24: draw gold triangle + '24'.
 Draw ring band between r0=0.62R and r1=0.98R slightly lighter? Insert is flat; draw outer gold line at 0.98R, inner line at 0.62R.
```
Numerals placed at r=0.80R, rotated upright (GMT bezel numerals usually upright). Draw text at position without rotation, upright.

Day/date canvas:
```js
function windowCanvas(w,h){...} date: 256x192, bg #f3ead6, text #1c1c1c font 'bold 120px Manrope'. day: 320x140? use 384x160, text 90px.
```
Add subtle inner shadow gradient.

Now materials:

```js
const M = {
 steel: new THREE.MeshStandardMaterial({color:0xc8ccd2, metalness:1, roughness:0.32, envMapIntensity:1.1}),
 steelBrush: ... roughness .45,
 gold: new THREE.MeshStandardMaterial({color:0xe0b96a, metalness:1, roughness:0.22, envMapIntensity:1.25}),
 goldDark: for frames {color:0xb98f3e...},
 black: MeshStandardMaterial({color:0x0c0f0e, metalness:.6, roughness:.5}),
 lacquerRed? second hand blue: MeshStandardMaterial({color:0x3a6fe0, metalness:.85, roughness:.28}),
 gmtOrange: MeshStandardMaterial({color:0xff7f36, metalness:.35, roughness:.4, emissive:0x?? none}),
};
```

Hands extruded:
```js
function handShape(pts){ const s=new THREE.Shape(); s.moveTo(...)... }
hour: [( -0.016,-0.22),(0.05,0.05),(0.028,0.92),(0,1.02),(-0.028,0.92),(-0.05,0.05)] close → dauphine-ish.
```
Let me make hour: base tail -0.25, width .055, tip 0.98:
pts: (0.0,-0.25)→(0.06,-0.02)→(0.035,0.85)→(0,0.98)→(-0.035,0.85)→(-0.06,-0.02) close.
minute: (0,-0.28)→(0.048,0)→(0.022,1.22)→(0,1.34)→(-0.022,1.22)→(-0.048,0).
gmt: shaft: (0,-0.3)→(0.035,-0.05)→(0.035,1.05)→(0.12,1.05)→(0,1.5)→(-0.12,1.05)→(-0.035,1.05)→(-0.035,-0.05) close. Arrow.
Extrude settings: depth 0.03, bevelEnabled true, bevelThickness .008, bevelSize .006, bevelSegments 2. Then geometry.translate(0,0,-depth/2)? Keep front.

Second hand group:
- Needle: BoxGeometry(0.016, 1.9, 0.02) positioned y = 0.55 → spans -0.4..1.5. Actually center y: length 1.9, tail to -0.45: center y = (1.5-0.45)/2=0.525.
- Counterweight: ring: TorusGeometry(0.09,0.028) at y=-0.32, plus small disc? Gold torus.
- Hub: Cylinder r 0.05.

Cap: gold cylinder r0.075 h0.05 on top + small dome sphere.

Pivot groups added to watch group at z positions.

Now indices:
```js
for i in 0..11: skip 3. angle=i*30°. if i==0 double.
pos x=sin(a)*1.24, y=cos(a)*1.24, rotZ=-a rad.
```
Angle in rad: a=i/12*TAU.

Also small lume dots: small circle plane at r=1.02 with slight emissive #cfe? Skip for cleanliness. Add printed dots on dial texture instead: draw small cream dots at each hour on texture except 3. Cheap: in dial texture loop draw circle at r=0.70R. Let's do that.

Bezel insert mesh: RingGeometry(1.72,2.02,128) with texture; material MeshStandardMaterial({map, metalness:.4, roughness:.5}).

Rehaut cone: CylinderGeometry(1.72,1.53,0.12,128,1,true) rotateX(PI/2): after rotation, top radius faces +Z? Cylinder +Y top → rotateX(π/2) makes +Y→+Z? Rotate around X by +90°: Y axis maps to +Z? Rotate vector (0,1,0) by R_x(90°): (0, cos90*1 - sin90*0, sin90*1 + cos90*0) = (0,0,1) ✓. So radiusTop faces +Z. Want front (z bigger) radius 1.72 → radiusTop=1.72 ✓, bottom 1.53. Position z=0.36 (from 0.30 to 0.42). Material dark steel.

Case body: main CylinderGeometry(2.18, 2.06, 0.5, 128) rotX90, z=0.05. Plus lower bevel CylinderGeometry(2.06,1.9,0.14) z=-0.27. Caseback CylinderGeometry(1.86,1.86,0.1) z=-0.36. Bezel side wall: CylinderGeometry(2.2,2.18,0.16) z=0.38? Then torus at z=0.5 radius 2.05 tube 0.14. Insert ring at z=0.475 (slightly below torus inner top). Crystal base z=0.5.

Check insert outer 2.02 vs torus inner edge 2.05-0.14=1.91 → torus covers insert outer edge from 1.91.. ✓ nice seating.

Crown: group at x=2.3: stem cylinder r0.09 length 0.2 axis X (rotateZ 90), head cylinder r0.21 h0.16 axis X with 18 segments flatShading gold, plus cap cone? Position x=2.42.

Lug capsules: CapsuleGeometry(0.13,0.42,8,16) at (±0.74, ±2.02, -0.1), rotation.x = ∓0.3, rotation.z = ±0.08 outward tilt.

Strap: as computed Rw=4.6, cz=-4.55, φ0=0.5, step 0.075, count 9? φ from 0.50 to 1.10. y0=sin(0.5)*4.6=2.21 ✓ just outside lug. Link: for each k: group position (0, side·sinφ·Rw, cosφ·Rw+cz), rotation.x=-side·φ. Children: center box Box(0.6,0.32,0.15) gold, sides Box(0.36,0.32,0.13) steel at x=±0.5. Slight gap via step. Last link (k=8): all steel wider (clasp) — or keep same. Add taper: scale x by 1 - k*0.03.

castShadow on links (many boxes → shadow cost OK).

Whole watch group: watchGroup = new Group; add all; watchGroup.position.y = 0.3; scene.add.

Idle sway: inner group `swing` containing watch parts; animate swing.rotation.y = sin(t*0.5)*0.16, rotation.x = sin(t*0.35)*0.05, position.y bob 0.05. When user drags, dampen sway? Keep subtle always; fine.

Floor shadow at y=-3.1.

Camera target (0,0.3,0).

Lights as planned. Key light shadow camera: left/right/top/bottom ±5, near 1 far 25.

Particles: 260 points.

Now UI JS wiring + loop.

Loop:
```js
function animate(t){
 requestAnimationFrame(animate);
 const time=t*0.001;
 updateWatch(); updateUI(); // ui throttled
 particles drift;
 swing anim;
 controls.update();
 cameraAnim step;
 renderer.render(scene,camera);
}
```

updateWatch computes zone times, sets rotations, checks date window.

updateUI every frame but only writes on change.

Loader fade after first render + fonts: init after fonts.ready (with 1500ms timeout race), then start loop, then setTimeout to fade loader.

Edge: some browsers block Google Fonts — fallback fine.

WebGL check: try { renderer = new WebGLRenderer } catch → show error div.

Also add `renderer.setClearColor(0x000000,0)` for alpha.

Tone mapping ACES, exposure 1.15.

Now let me write CSS details:

```css
:root{--gold:#d4af6a;--gold-dim:rgba(212,175,106,.35);--ink:#e9e4d6;--mut:#9aa39b;--bg:#0a0f0e;--panel:rgba(13,19,18,.92)}
*{box-sizing:border-box;margin:0}
html,body{height:100%}
body{font-family:'Noto Sans SC',system-ui,sans-serif;color:var(--ink);overflow:hidden;
 background:
  radial-gradient(1200px 800px at 68% 38%, rgba(214,178,106,.10), transparent 60%),
  radial-gradient(900px 700px at 20% 80%, rgba(64,120,100,.12), transparent 60%),
  radial-gradient(160% 120% at 50% 45%, #131c19 0%, #0a0f0e 55%, #060908 100%);
}
#noise{position:fixed;inset:0;pointer-events:none;opacity:.05;mix-blend:overlay;background-image:url("data:image/svg+xml,...fractalNoise...")}
canvas#scene{position:fixed;inset:0;display:block}
```

Noise SVG data URI: `data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='160' height='160'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2'/></filter><rect width='160' height='160' filter='url(%23n)' opacity='0.6'/></svg>` — need to URL-encode #. Use %23. OK.

Brand header top-left, panel right.

Digital font: `font-family: ui-monospace, 'SF Mono', 'Cascadia Mono', Consolas, monospace; font-size:30px; letter-spacing:2px; color: #f2e7c8; font-variant-numeric:tabular-nums;` GMT digital in orange-ish #ffc79a.

Selects: dark styled, gold border on focus.

Switch: custom checkbox → track 34x18, knob 14, gold when checked.

Buttons: transparent, 1px gold-dim border, hover bg gold/10, active translateY(1px).

Legend dots: i{width:9px;height:9px;border-radius:50%;background:var(--c);display:inline-block;margin-right:6px;box-shadow:0 0 8px var(--c)}.

Loader: fixed inset 0 bg var(--bg) display flex column center; ring: 54px border 2px solid gold-dim, border-top-color gold, spin 1s; fade .6s opacity 0 then visibility hidden.

Media query max-width 860px: panel → bottom sheet: left:12px right:12px top:auto bottom:12px transform none max-height 46vh overflow:auto; brand shrinks; hint hidden.

Now let me write it all out. Also include small "calendar" note: date/day follows 本地时区.

Double-check hand rotation sign: rotation.z = -angle so positive angle → clockwise viewed from +Z (standard screen view). Camera looks at +Z face. ✓.

GMT bezel numerals orientation: 24 at top, 6 at right (6 o'clock position = 90° clockwise = 6h) ✓ standard.

Bezel texture mapping to RingGeometry: RingGeometry UVs: default UVs are planar in unit circle? RingGeometry UVs are based on position within the ring… actually RingGeometry uv: uses vertex position normalized to [0,1] box (planar). So drawing the canvas as a full square image (ring drawing) matches planar mapping ✓ (texture drawn as ring in a square canvas). Yes: draw the entire bezel face as a 2D image (donut with numerals), map planar. RingGeometry UV planar covers the bounding square → canvas square ✓.

Dial CircleGeometry UV planar ✓.

Draw bezel image: transparent outside? RingGeometry only covers ring area so background irrelevant; fill dark.

Numerals: for h=2;h<=24;h+=2: angle = h/24*TAU - PI/2 (24 at top). Position at r=0.80R: x=R+cos·r? angle measured clockwise from top: x = R + Math.sin(a)*r, y = R - Math.cos(a)*r where a=h/24*TAU. Draw text upright, centered. Font '48px Marcellus'. Color cream. Also ticks every hour: small rects rotated: save translate rotate(a) fillRect(-1.5, -0.97R, 3, 14) etc.

Triangle marker at top: draw gold triangle at (R, R-0.90R) pointing down? Plus '24' text just inside at 0.78R. Draw triangle at r 0.90R and skip 24 numeral? Put both: triangle at outer, '24' at inner same angle. OK.

Dial texture: also draw date/day window openings? The physical planes cover; draw a thin gold outline on texture at those positions for integration? Frame mesh handles. Skip.

Frame: for date: gold box frame: create from 4 thin boxes, or PlaneGeometry with gold material slightly larger behind window canvas: back plane 0.46x0.34 gold at z 0.306, canvas 0.40x0.28 at z 0.312. Reads as framed. ✓ Day: back 0.66x0.30, canvas 0.60x0.24.

Wait, day window at 12 o'clock y=0.92: might overlap with printed minute track? Track at r>0.9R·(1.52)=… track inner tick radius 0.955R-34px→ world r ≈ (489-34)/336.8=1.35. Window top edge y=0.92+0.12=1.04 < 1.35 ✓.

Indices at r 1.24, length 0.26 → inner edge 1.11 — day window top 1.04 close but OK; index at 12 at y≈1.24±, window at 0.92 — no overlap.

Brand text at world y: canvas y=307 → (512-307)/336.8=0.61 above center ✓ below day window (bottom edge 0.92-0.12=0.80) ✓.

GMT hand length 1.5 reaches r=1.5 → over insert inner 1.72? Tip 1.5 < 1.72 — slightly short of numerals; extend to 1.62 (arrow 1.30→1.62). Set arrow base at 1.28, tip at 1.64. Shaft up to 1.28.

Second hand tip 1.5, minute 1.34, hour 0.98 ✓.

Now case shadow to floor: watch at y≈0.3, floor -3.1; light from (5,7,6) → shadow falls to left back. Fine.

Auto rotate speed 0.7, direction default.

Also `controls.enablePan=false`, `controls.enableDamping=true`, dampingFactor .06.

Reset animation: store start pos/target, ease t 0→1, camera.position.lerpVectors(start,def,ease), controls.target.lerp… then controls.update.

Keyboard: Space toggles sweep? Might scroll page — preventDefault. Add: key 'r' reset, 'o' orbit toggle. Minor.

Now finalize time zone list with labels.

Default local: Intl.DateTimeFormat().resolvedOptions().timeZone; if not in list, prepend with generic label (city from tz string).

Now date/day canvas update: function setWindow(tex, canvas, text). Date: large number centered. Day: 3-letter.

Also update digital date line: full format.

Potential issue: Intl with timeZone 'UTC' works.

Offset cache: map tz→{offset, t}. Refresh if >60s.

I think ready to write. Also include <meta viewport>. Title "子午 MERIDIAN · 双时区 3D 腕表".

Also handle visibilitychange: clock uses Date.now() so auto-corrects ✓ (nice note in test).

Also throttling: when tab hidden rAF pauses; on return time is correct ✓.

Now let me write the code, careful with syntax.

One more consideration: `new OrbitControls(camera, renderer.domElement)`.

PMREM RoomEnvironment:
```js
const pmrem = new THREE.PMREMGenerator(renderer);
scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
pmrem.dispose();
```

Import map: use three 0.160.0.

Also `THREE.ColorManagement` default on.

For canvas textures: tex.colorSpace = THREE.SRGBColorSpace; tex.anisotropy = maxAniso.

Extruded hand UV irrelevant, plain material.

Second hand: to reduce shadow acne on dial: light.shadow.bias = -0.0005, normalBias 0.02.

Dial material: MeshStandardMaterial({map:dialTex, metalness:.4, roughness:.55})? For slight sheen on dial use metalness .25 roughness .5. Add MeshPhysicalMaterial clearcoat? Standard is fine.

Also add subtle circular "texture ring" for rehaut: skip.

Now count: many meshes (~90 links boxes + others) fine.

Write UI update:
```js
function pad(n){return String(n).padStart(2,'0')}
```
Date line: `${t.year}年${t.month+1}月${t.date}日 · 星期${'日一二三四五六'[t.day]} · ${fmtOff(t.offset)}` maybe split: dateline shows date+weekday; offset chip near select? Put offset in tz-label row: `<span id=offLocal class=off>UTC+8</span>`. Add to tz-label row.

Panel head with small movement number.

Also add tooltip title attributes.

Edge: swap should also swap offsets and windows — handled by next frame.

Now let me write the final file. Aim for clean commented code.

Also loader hidden after first frame: after first render requestAnimationFrame → loader.classList.add('done').

Font loading:
```js
const ready = Promise.race([document.fonts.ready, new Promise(r=>setTimeout(r,1600))]);
ready.then(init);
```
But module script with imports — init defined inside.

Also guard: if !window.WebGL2RenderingContext? Just try/catch renderer creation.

Error overlay div #fatal hidden; show message on failure.

Now write. Also small feature: click on watch crown? Skip.

Also for "smooth" requirement: sweep uses ms — ✓; mention in test.

Also ensure `renderer.setAnimationLoop(animate)` — better than rAF for XR none; either works. Use setAnimationLoop.

animate(timestamp) — timestamp in ms.

For sway use timestamp.

Camera reset animation state: {active,t0,fromPos,fromTgt}.

controls 'start' → userHold=true; cancel reset anim? Fine. 'end' → schedule resume.

Auto orbit logic:
```js
let orbitOn=true, resumeTimer=null, holding=false;
controls.addEventListener('start',()=>{holding=true; clearTimeout(resumeTimer); controls.autoRotate=false;});
controls.addEventListener('end',()=>{holding=false; resumeTimer=setTimeout(()=>{controls.autoRotate = orbitOn;},3500);});
chkOrbit.onchange: orbitOn=checked; controls.autoRotate = orbitOn && !holding;
```

Sweep toggle: chkSweep.checked → sweep, else tick.

Now hand z-fighting prevention: separate z with 0.02-0.03 gap ✓; polygonOffset not needed.

Crystal dome may intersect second hand cap at center (cap 0.44+0.03=0.47 vs dome center 0.50) ✓.

Bezel torus at z=0.50 tube .14: front 0.64 > dome edge base… dome at edge z=0.50, torus covers rim ✓.

Insert at z=0.47, below dome (dome base plane 0.50) — dome covers insert: dome transmission shows insert through glass ✓ nice.

Dial 0.30, below dome ✓.

Case top: bezel wall cylinder z 0.38±0.08 → top 0.46; torus center 0.50 covers ✓.

OK, also add subtle emissive to dial? No.

One more: `scene.background = null` default; renderer alpha true → CSS background visible. But environment lighting still from RoomEnvironment ✓.

Particles behind watch visible through transmission? Transmission pass renders scene without transmissive → particles included ✓.

Now let me write the file. Also include run instructions and test checklist in chat response (concise), and also in HTML comment at top of file.

Response language: Chinese.

Let me write out the full code carefully.

Additional check — RingGeometry planar UV: RingGeometry generates uv from vertex position: uv.x = (vertex.x / outerRadius + 1)/2 etc. Yes planar ✓.

CircleGeometry uv planar centered ✓.

Dial texture orientation: CircleGeometry planar uv maps +Y up in texture to +Y in world ✓ (canvas y down → texture flipped: CanvasTexture flipY default true → canvas top = v=1 = +Y ✓). So drawing canvas with y down: "top" of dial (12) is canvas y small ✓ consistent with drawing ticks at top with -R offset ✓.

Bezel same ✓.

Day window text canvas: flipY fine.

Now hand rotation at 12 points +Y ✓ matches.

Crown position x positive = 3 o'clock ✓ matches date at 3.

Alright — also small pusher at 2/4 for style? Skip.

Time to write. Also include fallback in case `document.fonts` undefined: guard.

Also ensure selects built before use; init UI in init().

Let me write updateUI:
```js
const uiCache={};
function setText(el, s){ if(uiCache[el.id]!==s){uiCache[el.id]=s; el.textContent=s;} }
```

Digital shows seconds → updates every second (tick mode) or every second boundary; fine.

Also GMT block label shows tz name.

fmtOff(off): const sign=off<0?'−':'+'; a=Math.abs(off); h=a/60|0; m=a%60; return `UTC${sign}${h}${m?':'+pad(m):''}`;

Now particle construction:
```js
const N=240, pos=new Float32Array(N*3), spd=new Float32Array(N);
for i: x=(rand-.5)*26, y=(rand-.5)*14, z=(rand-.5)*18 -2; spd=.2+rand*.5;
geo.setAttribute position. mat=PointsMaterial({color:0xd8bd85,size:.06,transparent:true,opacity:.45,blending:Additive,depthWrite:false,sizeAttenuation:true});
loop: y+=spd*dt; if y>8 y=-8... store base x with slight sway: x = baseX + sin(t*.3+i)*.3 → need base array. Keep simple: only y drift + whole point cloud slow rotation.y.
points.rotation.y = t*0.02;
```

dt from clock: use fixed dt=1/60 in setAnimationLoop enough or compute delta from THREE.Clock. Use THREE.Clock.

OK final write. Also `watch` scale: overall diameter ~4.4 world; camera distance 7.6 fov 38 → fits nicely with panel.

controls.target (0,0.2,0).

Also `renderer.toneMapping = THREE.ACESFilmicToneMapping`.

Also `renderer.shadowMap.type = THREE.PCFSoftShadowMap`.

Materials needing flatShading: crown head flatShading true.

Also `MeshStandardMaterial` for dial texture with `roughness:.45, metalness:.3`.

Also `MeshPhysicalMaterial` for crystal with `transparent:false` (transmission handles), `transmission:1, thickness:.5, roughness:.02, ior:1.5, envMapIntensity:1.6, clearcoat:1, clearcoatRoughness:.06, specularIntensity:1`.

Also `side:THREE.DoubleSide`? Front only fine; but dome edge may show backface from low angle — set side: FrontSide default; fine.

Also `depthWrite:true` default OK.

Transmission + alpha canvas: known artifact: transmission uses scene color behind — with transparent canvas background, area behind glass shows only scene objects; background gradient not included (scene.background null → transmission buffer is black where nothing). Crystal mostly covers dial so fine.

To be safe, set scene.background = new THREE.Color(0x0a0f0e)? Then CSS gradient hidden behind opaque canvas. Lose layered CSS background. Compromise: keep alpha; dial fills crystal view ✓.

Edge view at grazing angle: through dome edge may see black sliver — acceptable.

Alternatively give scene.background a subtle gradient via large sphere… skip.

Now finalize response. Also mention: if CDN blocked, can download three and change importmap.

Also test note: verify dual tz (select London vs Beijing, GMT hand position), sweep vs tick, date window changes at midnight (can't wait — can temporarily override? Provide console hook: expose `window.__setZone`? Simpler: mention date follows system; to test, change system clock or pick tz that crosses date boundary (e.g., pick Auckland vs Los Angeles to see different dates) ✓ nice test).

Also expose debug: `window.watch3d = {setLocal, setGmt}` minor; skip for cleanliness. Actually useful for testing: expose API to set zones programmatically. Add small: window.MERIDIAN = { setZones(a,b) }. Cheap, include.

Now write everything. Also double-check no `const` name collisions.

Also add `overflow:hidden` on body; panel scrollable inside on small screens.

Also `touch-action:none` on canvas for OrbitControls mobile.

Now generate the final code.

Also add title attribute on buttons.

Also `user-select:none` on UI.

Also select option text: `${city} · ${tz}` maybe long; use city only + offset computed later. Option label: city (e.g., 北京). Value tz. Add title with full tz.

Ensure default local tz added if missing:
```js
const localTz = (Intl.DateTimeFormat().resolvedOptions().timeZone)||'UTC';
if(!ZONES.find(z=>z[0]===localTz)) ZONES.unshift([localTz, localTz.split('/').pop().replace(/_/g,' ')]);
```
Default gmt: pick 'Europe/London' if localTz!=='Europe/London' else 'America/New_York'.

Now write. Also add `aria-label` minor.

Also loader visible before JS: pure CSS spinner animation; hide via JS. If JS fails (CDN blocked), loader stuck → add <noscript> message + a timeout note in loader "加载时间较长？请检查网络（CDN）". Add small hint text in loader after 6s via CSS animation delay: use CSS: .l-tip{opacity:0;animation:tipIn .5s 6s forwards}.

Good. Write final.

Also double-check easeOutBack formula:
```js
function easeOutBack(x){const c1=1.70158,c3=c1+1;return 1+c3*Math.pow(x-1,3)+c1*Math.pow(x-1,2);}
```
At x=1 → 1 ✓.

Tick display: secDisp = s + easeOutBack(min(1, ms/1000*6.5)); at ms=0 → x=0 → 0 ✓.

Minute hand in tick mode: keep continuous (real watches minute creeps) ✓.

Also second hand shadow: needle thin — shadow subtle ✓.

Also `swing` group: contains case + dial + hands + strap; sway rotation small.

Also floor shadow moves with sway ✓ automatic.

Also slight watch tilt toward camera: swing.rotation.x base -0.05.

Final camera default: pos (3.4, 1.6, 7.2), target (0,0.15,0).

fov 36.

Also `controls.minPolarAngle=0.35, maxPolarAngle=1.9` to allow slight below view.

Also `controls.minDistance=4.2, maxDistance=15`.

Now write the file.

After code, response text: brief 运行方式 + 测试要点 bullet list. Keep response concise (code is the main artifact).

Also let me mentally scan for syntax errors as I write.

One concern: `CylinderGeometry` openEnded parameter position: (radiusTop, radiusBottom, height, radialSegments, heightSegments, openEnded) ✓.

CapsuleGeometry(radius, length, capSegments, radialSegments) ✓ r160.

TorusGeometry(radius, tube, radialSegments, tubularSegments) ✓.

RingGeometry(inner, outer, thetaSegments) ✓.

For crown knurling: CylinderGeometry(0.21,0.21,0.16,18) with material flatShading → facets ✓.

Also crown cap: CylinderGeometry(0.13,0.18,0.05,18) at outer end + small sphere logo? Keep.

Stem: CylinderGeometry(0.07,0.07,0.24,12) rotateZ(π/2) at x=2.24.

Head at x=2.44.

Case radius 2.18 → stem from 2.12 to 2.36 ✓ head 2.36..2.52.

Lug x ±0.74, capsule radius .13 → inner gap 0.74-? Strap width: link center piece .6 + sides .36*2 + gap → total ≈ 0.6+0.02+0.36*2 ≈ 1.36 width vs lug spacing 1.48 ✓.

Now dial texture dots: hour marker dots at r=0.72R, skip? Draw small at all 12 except 3: cream circle r=7px. Also at 12 two dots? Just one. Fine.

Also small text at 6 o'clock position: '21 JEWELS' at y=R+250 ✓ (0.49 world below center) — second hand counterweight area passes over, fine.

Also small "GMT" label at center-left? Skip.

Now write. Also include HTML comment at top of file with run instructions.

Also `id="scene"` canvas created in HTML: `<canvas id="scene"></canvas>`.

Renderer: `new THREE.WebGLRenderer({canvas, antialias:true, alpha:true})`.

Set pixel ratio and size on resize.

Also `renderer.shadowMap.enabled=true`.

Lights:
```js
const key=new THREE.DirectionalLight(0xfff0da,3.2); key.position.set(5,8,6); key.castShadow=true; key.shadow.mapSize.set(2048,2048); key.shadow.camera.near=2; far=25; left=-6 right=6 top=8 bottom=-6; bias=-0.0004; normalBias=.02; radius=6;
const fill=new THREE.DirectionalLight(0xbcd4e8,.7); pos(-7,3,5);
const rim=new THREE.DirectionalLight(0xffd9a0,1.6); pos(-4,5,-7);
const amb=new THREE.AmbientLight(0x223028,.5)? environment provides ambient; skip or small 0.25.
```
PCFSoft ignores radius? radius works with PCFSoft? shadow.radius works with PCFShadowMap/PCFSoft? Actually radius has no effect on PCFShadowMap? Docs: radius works with PCFShadowMap? VSM uses radius. PCFSoft ignores radius. Fine, remove radius.

Floor: ShadowMaterial opacity .38, color default black; also add faint radial glow plane: MeshBasicMaterial with radial gradient canvas texture (gold faint) as additive under watch, y=-3.09, size 12 — adds grounding. Let's do: canvas 256 radial rgba(216,180,110,.16)→0. Plane rotated flat, blending additive, depthWrite false. Nice.

Now particle color slightly green-gold.

Ready. Also `THREE.MathUtils` not needed much.

Also ensure `selLocal.onchange` also updates day/date window immediately: updateWatch runs every frame anyway ✓.

Also expose window.MERIDIAN.setZones.

Now write the final code block. Also short Chinese section comments.

Potential subtle bug: offset cache keyed by tz but offset depends on ts (DST) — refresh 60s ✓.

getOffset:
```js
const offCache=new Map();
function getOffset(tz){
 const nowSec=Math.floor(Date.now()/1000)*1000;
 const c=offCache.get(tz);
 if(c && nowSec-c.t<60000) return c.off;
 let off=0;
 try{
  const dtf=new Intl.DateTimeFormat('en-US',{timeZone:tz,hour12:false,year:'numeric',month:'2-digit',day:'2-digit',hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const p={}; dtf.formatToParts(nowSec).forEach(x=>p[x.type]=x.value);
  const asUTC=Date.UTC(+p.year,+p.month-1,+p.day, p.hour==='24'?0:+p.hour, +p.minute, +p.second);
  off=(asUTC-nowSec)/60000;
 }catch(e){ off=0; }
 offCache.set(tz,{t:nowSec,off});
 return off;
}
```

zoneNow:
```js
function zoneNow(tz){
 const ms=Date.now(); const sec=Math.floor(ms/1000); const milli=ms-sec*1000;
 const off=getOffset(tz);
 const d=new Date((sec+off*60)*1000);
 return {h:d.getUTCHours(),m:d.getUTCMinutes(),s:d.getUTCSeconds(),ms:milli,day:d.getUTCDay(),date:d.getUTCDate(),month:d.getUTCMonth(),year:d.getUTCFullYear(),off};
}
```

Good.

Now hand refs: pivots: hourP, minP, secP, gmtP.

updateWatch:
```js
const L=zoneNow(TZ.local), G=zoneNow(TZ.gmt);
const sl = sweep? (L.s+L.ms/1000) : (L.s+easeOutBack(Math.min(1,L.ms/1000*6.5)));
secP.rotation.z = -sl/60*TAU;
minP.rotation.z = -((L.m+(L.s+L.ms/1000)/60)/60)*TAU;
hourP.rotation.z = -(((L.h%12)+L.m/60+L.s/3600)/12)*TAU;
gmtP.rotation.z = -((G.h+G.m/60+G.s/3600)/24)*TAU;
if(L.date!==lastDate||L.day!==lastDay){ setDateWin(L.date); setDayWin(L.day); lastDate=L.date; lastDay=L.day; }
```

UI:
```js
setText(dLocal, `${pad(L.h)}:${pad(L.m)}:${pad(L.s)}`);
setText(dateLocal, `${L.year}年${L.month+1}月${L.date}日 · 星期${CNW[L.day]}`);
setText(offLocal, fmtOff(L.off));
... gmt
```

OK. Now compose the whole file. Also add `lang="zh-CN"`.

Also for small screens: brand shrinks.

Also panel with `backdrop-filter:none`, solid background — use rgba(12,17,16,.94) + 1px border gold-dim + shadow. Radius 12px.

Section separators: 1px rgba gold .15.

Head: letter-spacing 4px, gold, small; right tag mono tiny.

Digital size 26px.

Also add hover on panel sections? Subtle border glow on tz-block hover — micro-interaction ✓.

Button hover transitions .2s.

Loader ring: border spin + inner dot? Fine.

Now write. Also remember `OrbitControls` import path 'three/addons/controls/OrbitControls.js'.

Also RoomEnvironment import.

Final check importmap CDN paths exist:
- https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js ✓
- https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/ ✓

Write the response with brief usage + tests around the code block. Response in Chinese.
