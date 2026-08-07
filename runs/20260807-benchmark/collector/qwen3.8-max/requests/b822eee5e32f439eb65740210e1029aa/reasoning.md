The user wants a 3D data visualization globe using Three.js in a single HTML file. Requirements:

1. 3D Earth: realistic earth texture (continents, oceans), rotation interaction, atmospheric glow
2. Data visualization layer:
   - Major city markers, glowing points
   - Arcs between cities (flight routes), flow animation
   - Country/region heat coloring
3. Interaction:
   - Mouse drag to rotate
   - Wheel to zoom
   - Click city to show info popup
   - Hover on route to show flight info
4. Control panel:
   - Switch data metrics (GDP, population, flight volume)
   - Timeline playback showing data changes over time
5. Visual effects:
   - Earth auto-rotation
   - Day/night terminator line
   - City lights effect, especially on the night side
6. Technical: Three.js, realistic earth texture, importmap + ESM, single HTML file

Key challenge: "realistic earth texture" — need real earth textures. Since it's a single HTML file, we can load textures from CDN. Reliable sources:
- threejs.org's examples textures (threejs.org/examples/textures/planets/earth_atmos_2048.jpg, earth_lights_2048.png, earth_normal_2048.jpg, earth_specular_2048.jpg)
- unpkg: https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg, earth-night.jpg, earth-topology.png, earth-water.png, night-sky.png
- three-globe's textures are pretty reliable via unpkg/jsdelivr.

However, CORS: threejs.org serves with CORS enabled? I think threejs.org allows cross-origin (it's a static site). unpkg definitely has CORS enabled. jsdelivr too.

But to be safe and robust, we should also provide a procedural fallback texture in case texture loading fails (offline). Can generate a canvas-based fallback earth texture. That adds robustness. Actually, if we draw continents procedurally on canvas, we can make a decent fallback — but that's heavy. Alternative: use a simple gradient fallback with a note. Hmm.

Better approach: try loading from multiple CDN sources with fallback chain:
1. unpkg three-globe textures
2. jsdelivr three-globe
3. procedural canvas fallback (draw a simple equirectangular map from embedded low-res land mask? too large)

Actually, we can embed a compact land mask. There's a known trick: encode a small earth map as a base64 PNG. A 360x180 or 720x360 earth map compressed as base64 PNG might be around 10-30KB. Can't generate accurate base64 PNG from memory. So procedural fallback: draw approximate continent shapes with polygons. Can hand-code rough continent polygons (very coarse). It's possible but ugly. Or just a stylized "grid ocean" fallback.

Given the deliverable is expected to run with internet (CDN Three.js via importmap anyway), texture loading from CDN is acceptable. Three.js itself has to come from CDN via importmap. So network is a precondition anyway. Note in run instructions: needs internet, serve via local server or open directly (ES modules from CDN work from file://? importmap + remote modules works from file:// in Chrome? module scripts loaded from https URL are fine; importmap itself works as inline. So opening file:// with importmap pointing to CDN should work. But texture loading from https via file:// origin — CORS: unpkg sends Access-Control-Allow-Origin: *, so it works even with null origin. WebGL texture from cross-origin image requires crossOrigin='anonymous'; THREE.TextureLoader sets it via LoadingManager or loader.setCrossOrigin('anonymous'). Good.

Plan texture sources:
- Day: https://unpkg.com/three-globe@2.31.1/example/img/earth-blue-marble.jpg (4096x2048? actually 4096x2048 blue marble). Also earth-day.jpg? three-globe's example img contains: earth-blue-marble.jpg, earth-dark.jpg, earth-day.jpg, earth-night.jpg, earth-topology.png, earth-water.png, night-sky.png. Pin version for stability, use three-globe@2.31.0 or just @latest? Pin: three-globe@2.31.1. Actually let's use jsdelivr which is often faster: https://cdn.jsdelivr.net/npm/three-globe@2.31.1/example/img/earth-blue-marble.jpg

- Night lights: earth-night.jpg
- Bump/topology: earth-topology.png
- Specular/water mask: earth-water.png (three-globe has earth-water.png which is a water mask).
- Star sky: night-sky.png (from three-globe's example) or generate stars procedurally with points. Do procedural starfield with THREE.Points — reliable and lightweight.

Also atmospheric glow: custom shader (Fresnel) — slightly larger sphere with BackSide shader.

Day/night terminator: custom shader material that blends day and night textures based on sun direction. Classic approach:

uniforms: dayTexture, nightTexture, sunDirection.
Fragment: float cosAngle = dot(normalize(vNormal), sunDir); blend night→day with smoothstep. Also specular on ocean if water mask available. Keep it moderate.

Sun direction rotates slowly (or fixed with earth rotating relative to it → terminator moves). Earth auto-rotates and sun stays fixed in world space, so terminator sweeps across surface — good.

City lights: night texture already contains lights; additionally add glowing point markers for cities that brighten on night side. City markers: use THREE.Points with custom shader, or individual sprites. For hover/click picking, raycast against Points with threshold, or use small mesh spheres. With ~30-40 cities, individual sprites/meshes are fine. Make it a Group of small sphere meshes + additive sprite glow. Raycasting to sphere meshes is easy.

Actually better: city marker = small sphere (raycast target) + pulsing ring on surface + glow sprite. Size/color depends on metric value.

Arcs (flight routes): quadratic/cubic Bezier lifted above surface, TubeGeometry or Line with dashed animation. Flow animation: use shader with dash offset, or move a "pulse" along the curve. Approach: create curve points, use THREE.Line with ShaderMaterial animating a moving bright segment (based on progress uniform and per-vertex t attribute). Or use LineDashedMaterial animating dashOffset — simpler but less pretty. Custom shader: attribute aT (0..1 along line), fragment intensity = smooth pulse at fract(t - time). Additive blending. Good flow effect.

Hover on route: raycasting to lines is tricky (Line raycast threshold). Use raycaster.params.Line.threshold. With ~40-60 routes, fine. On hover, highlight route and show tooltip with flight info (flight number, airline, departure/arrival, duration, frequency). Show tooltip that follows mouse.

Country/region heat coloring: hard without geo data. Options: load GeoJSON countries from CDN (world-atlas topojson). Extra dependency + complexity but doable: fetch https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json and convert topojson→geojson (need topojson-client). Then draw countries on canvas texture with per-country metric-based color, overlay as second sphere slightly larger with transparent texture, or blend in shader.

Simpler and robust approach: draw heat overlay on 2048x1024 canvas: fill each country's polygon with color from scale, use as texture on transparent overlay sphere (radius +0.002). Toggleable. This is a legit "country/region heat coloring".

Topojson conversion: can include topojson-client via importmap ESM: https://cdn.jsdelivr.net/npm/topojson-client@3/+esm. And fetch world-atlas countries-110m.json. This is a runtime network fetch; if it fails, gracefully skip heat layer (fallback: radial heat blobs around each city drawn on canvas). Actually good fallback: draw radial gradients at city positions on canvas, colored by metric — "regional heat". Implement both: if GeoJSON loads, country choropleth; else city radial heat blobs.

Hmm, but per-country metric: need data per country. Dataset is cities with country code, so country metric = aggregate of its cities (sum/average). Fine — define metric per country via ISO numeric id? world-atlas's countries-110m has id = ISO 3166-1 numeric. Can map city country to numeric code for the countries we include. Simpler: color countries that contain one of our cities, using that country's aggregated value; other countries neutral gray. Need mapping country name → numeric id. Can hardcode numeric ids for ~20 countries in dataset. Manageable.

Or, skip topojson and draw heat as radial gradients only. But requirement explicitly says "country/region heat coloring". Country choropleth is much more impressive. Let's do it with topojson-client ESM from jsdelivr and world-atlas fetch. Provide fallback.

Let's design dataset: ~28 global cities with lat, lon, name, country, metrics over time: GDP (billion USD), population (millions), flights (daily departures), plus maybe "data traffic"? Keep 3 metrics: GDP, population, flight volume. Time axis: 2019-2024 years with per-year values (can generate with base + growth function to keep code compact, but deterministic and plausible). Store base values and per-city annual growth rate, compute values. Deterministic, compact.

Routes: define pairs with flight info: flight number, airline, weekly flights, distance. Generate arcs between major hub pairs (~40 routes).

Control panel:
- Metric switch (segmented buttons): GDP / Population / Flights
- Layer toggles: routes, cities, heat, atmosphere, auto-rotate
- Timeline: play/pause, year slider 2019–2024, speed. On year change: city marker size/color updates, heat recolors, route intensity scales with flight metric, small "year" HUD.
- Stats readout: total value, top city.

Info popup on city click: HTML overlay panel positioned near screen projection of city, showing name, country, per-metric values with mini bar/sparkline across years. Sparkline: small canvas or SVG. Nice touch: SVG sparkline of selected metric across years.

Hover tooltip for route: flight info.
Hover on city also shows small tooltip with name.

UI design: this is a dark space scene inherently. Use deep space blue-black with multi-hued data palette (amber for GDP, cyan for flights, coral/green for population?) rather than a single neon accent. Palette idea: background deep navy #050914, panels dark slate with warm amber + cyan + magenta data colors. Use a multi-hue system: metric colors — GDP: warm gold #ffb347/#f5a524; population: coral/rose #ff6b6b or spring green; flights: cyan #4dd8ff. UI chrome with two accent hues and warm off-white text.

Typography: distinctive display + readable body. Since Chinese content, pick font: display could be "Big Shoulders Display" or "Chakra Petch" (tech vibe) for Latin/numbers + Chinese fallback to system. Chinese webfonts are heavy; use system Chinese stack ("PingFang SC", "Microsoft YaHei", "Noto Sans SC"). For display numbers/headers: "Chakra Petch" or "Rajdhani" or "Space Grotesk" from Google Fonts. Rajdhani fits HUD vibe. Body: system Chinese + "IBM Plex Sans"? Keep: Rajdhani (display/numbers) + Noto Sans SC? Loading Noto Sans SC webfont is heavy but Google Fonts handles subsetting via unicode-range slices; fine. Include Noto Sans SC 400/700 — browser will download only needed slices. And Rajdhani 500/600/700 for numeric/HUD.

Layout: full-screen canvas; top-left title block (mission-control style), right side control panel, bottom timeline bar. Corner HUD elements (FPS? coords of cursor lat/lon — nice touch: show lat/lon under cursor). Legend for heat scale. Loading overlay with progress.

Micro-interactions: buttons hover states, panel sections, animated value counters, pulsing markers, glow on selected city ring.

Three.js version: use importmap:
```
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
    "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm"
  }
}
```
Do we need addons? OrbitControls — but we want custom drag rotate (rotate globe itself vs camera). OrbitControls provides drag rotate + wheel zoom + damping easily. But auto-rotate of earth with fixed sun: if camera orbits, sun fixed in world is fine. However "earth rotation animation" — rotate earth group itself; markers/arcs are children of earth group so rotate together. OrbitControls with autoRotate off; rotate earth mesh. Drag rotates camera via OrbitControls — acceptable as "drag to rotate globe". But mixing earth auto-rotation + camera orbit is fine.

Hmm, one nuance: raycasting for tooltips needs screen projection — standard.

Zoom: OrbitControls min/max distance. enableDamping.

Or custom controls: drag rotates earth group directly (feels like spinning the globe), wheel zooms camera. Many globe demos rotate the globe on drag. OrbitControls is simpler and robust; but rotation of earth with fixed camera keeps lighting consistent relative to viewer… actually with sun fixed in world and camera orbiting, terminator position relative to viewer changes — that's realistic. With earth spinning + camera orbit, both fine.

Let me implement custom controls for the feel of "grabbing the globe": drag rotates earthGroup (with inertia), wheel zooms camera z (with clamp + smoothing). Plus idle auto-rotation that resumes after N seconds. This is more crafted. Implement carefully:

- pointerdown: record, disable auto-rotate.
- pointermove while down: dx,dy → targetRotY += dx*0.005; targetRotX += dy*0.003 clamp to ±~1.1 rad? Rotating earth group on X tilts it; clamp to avoid flipping. Actually rotating group on X with markers is fine.
- Inertia: velocity decays.
- Smooth lerp current→target each frame.
- Wheel: targetDist clamp [1.5R, 5R].

Also distinguish click vs drag: track movement distance; if < 5px and pointerup → click → raycast cities.

Hover: raycast on pointermove (throttled) against city meshes and route lines.

Sun: directional uniform in shader. Sun direction can slowly rotate around Y to simulate day cycle, or fix and earth rotates → terminator moves across surface (earth rotates under fixed sun). Since earth group rotates on Y for spin, if sun is fixed in world, terminator sweeps across surface.

Earth shader:
```
uniform sampler2D dayMap, nightMap, specMap(optional);
uniform vec3 sunDir;
varying vNormal (world), vUv, vWorldPos?
float nd = dot(normalize(vNormalW), sunDir);
float dayAmt = smoothstep(-0.12, 0.18, nd);
color = mix(nightColor*nightBoost + cityGlow, dayColor*lighting, dayAmt)
```
Add subtle blue atmospheric rim on day side within surface shader (fresnel) + outer halo shell.

Night texture: earth-night.jpg. Multiply by 1.4 and add slight warm tint.

Ocean specular: use earth-water.png as mask; compute specular from sun and view direction; add subtly. Optional — include if texture loads; keep shader with uniform hasSpecMap fallback to constant. Simplify: skip water mask and use luminance of day texture to estimate specular? Meh. Include water mask as 4th texture; fallback 0.

Texture loading manager with progress bar. Fallback: if error, generate procedural canvas texture (simple: ocean gradient + noise + grid) so scene still renders. Write a small procedural generator (ocean blue with subtle noise, faint continents… maybe skip continents; just ocean + lat/lon grid). Acceptable degraded mode. But primary path uses real textures.

Atmosphere: sphere at radius*1.045, ShaderMaterial BackSide, fresnel glow color mix (day side cyan-blue, night side darker). Additive.

Also inner rim fresnel on earth shader for limb glow.

Stars: 2 layers of THREE.Points with random spherical positions, sizes, slight twinkle via shader or opacity oscillation. Keep simple: PointsMaterial with vertex colors, animate rotation slowly.

City markers: for each city:
- Anchor position via latLonToVector3(lat, lon, R).
- Marker group children of earthGroup.
- Core: small SphereGeometry(0.012R * scale) MeshBasicMaterial metric color.
- Glow sprite: canvas radial gradient texture, AdditiveBlending, scale pulses.
- Ring: RingGeometry on tangent plane (orient with lookAt from center), pulses outward (scale + fade) — shader or animate scale/opacity in JS. With ~30 cities, JS animation is fine.
- Size ∝ metric value (normalized), color per metric.

On metric change: animate sizes/colors (lerp in update loop).

Routes: for each route, curve = great-circle-ish arc: interpolate between endpoints on sphere, lift altitude by sin(t*π)*h, h ∝ angular distance. Points ~64. Geometry with attribute 'aT'. ShaderMaterial:
```
uniform uColor, uTime, uSpeed, uIntensity, uHover;
attribute float aT;
varying vT;
frag: base = uColor * 0.35; pulse: p = fract(vT - uTime*uSpeed); glow = smoothstep(0.12,0.0,p); // comet head
Actually multiple pulses: float f = fract(vT*2.0 - uTime*speed); glow = pow(smoothstep(0.25,0.,f),2.)
color = base + uColor*glow*intensity; alpha similarly; additive.
```
Direction: pulses travel A→B. Also add moving "plane" dot? Can add a small sprite moving along curve — nice touch ("aircraft" light). With 40 routes, one sprite each moving at parameter t — do it, lightweight. Or skip for perf; the pulse is enough. Let's add a moving dot per route for liveliness (Points or small sprite). Implement as single Points geometry containing all routes' moving dots, updating positions each frame from curve.getPoint — 40 evaluations per frame is trivial.

Route hover: raycaster.params.Line.threshold = 0.02 (world units, R=1). On hover: set uHover=1 (brighter, thicker? line width not adjustable; brighten + show tooltip). Also raise renderOrder.

Heat layer: overlay sphere radius R*1.001 with canvas texture (transparent). Build canvas 2048x1024:
- If countries geojson available: for each country feature, get metric value (from mapping by numeric id), fill path with color from scale; stroke with faint boundary. Equirectangular projection: x = (lon+180)/360*w, y = (90-lat)/180*h. Path polygons via Path2D. Handle antimeridian? countries-110m geometries are already split (world-atlas is pre-split? topojson's land is split I think; polygons crossing ±180 like Russia/Fiji/Antarctica might render stretched). world-atlas countries-110m.json — Russia crosses 180: its geometry has coordinates beyond ±180? Natural Earth data usually clips at antimeridian with multi-polygon on both sides. I think world-atlas is pre-split. Accept minor artifacts.
- Color scale: metric → quantize → color ramp. Ramps per metric: GDP: deep blue→gold; population: teal→coral; flights: navy→cyan. Provide legend gradient.
- Countries with no data: fill with rgba(255,255,255,0.03) or skip (skip → transparent, shows earth). Slight tint for continuity? Skip.
- Fallback (no geojson): radial gradient at city locations, radius ∝ value, additive-ish color with alpha.

Rebuild canvas on metric/year change (debounced; canvas 2048x1024 redraw with ~180 countries is fast enough, maybe 10-30ms; fine on change events, not per frame).

Texture: CanvasTexture, needsUpdate on redraw. Overlay material: MeshBasicMaterial transparent, depthWrite false. Lighting? Basic is fine. But should darken on night side… keep it glowing (data layer), acceptable and actually nice.

Terminator only affects earth shader; overlay stays visible — fine for data viz.

Timeline: years 2019..2024. Play: advance fractional year at speed (e.g., 1 year per 2.5s); interpolate values continuously for smooth transitions (city sizes, heat recomputed at integer year boundaries, or interpolate? Recomputing heat every frame is heavy; recompute when floor(year) changes, but interpolate city sizes continuously). Slider input range 0..(N-1)*100.

Value computation: value(city, metric, yearFloat) = base[metric] * (1+growth[metric])^(yearFloat-2019) * seasonal? Keep monotonic growth + small sinusoidal wobble for liveliness: * (1 + 0.03*sin(2π*(yearFloat))). Deterministic, fine. 2020 dip for flights (pandemic!) — nice storytelling: flights drop in 2020 then recover. Encode per-metric year multiplier array: flights: [1.0, 0.55, 0.72, 0.88, 0.97, 1.08] (2019-2024), gdp: [1, 0.97, 1.03, 1.09, 1.16, 1.23], population steady growth. Global multiplier per metric per year; city-level growth rate adds variance. This gives timeline real narrative.

City dataset (about 30): name (Chinese), country, lat, lon, base gdp (billion USD), base population (millions), base flights (daily departures ~), growth. Let me write plausible numbers:

- Beijing PEK 39.9,116.4 China GDP 680, pop 21.9, flights 1100
- Shanghai 31.2,121.5 GDP 720, pop 24.9, 1050
- Guangzhou 23.1,113.3 GDP 420, 18.7, 800
- Shenzhen 22.5,114.1 490, 17.6, 640
- Chengdu 30.6,104.1 320, 21.2, 620
- Hong Kong 22.3,114.2 360, 7.5, 700
- Tokyo 35.7,139.7 1050, 37.4, 980
- Seoul 37.6,127.0 820, 26.0, 890
- Singapore 1.35,103.8 400, 5.9, 900
- Bangkok 13.75,100.5 240, 10.7, 780
- Delhi 28.6,77.2 320, 32.9, 610
- Mumbai 19.1,72.9 310, 20.9, 540
- Dubai 25.2,55.3 110, 3.6, 950
- Istanbul 41.0,28.98 240, 15.6, 850
- Moscow 55.75,37.6 260? GDP ~, 12.6, 520
- London 51.5,-0.13 850, 9.5, 1300
- Paris 48.85,2.35 760, 11.0, 1150
- Frankfurt 50.1,8.68 400, 7.6? metro ~, 800
- Amsterdam 52.37,4.9 200, 1.1? use metro 2.5? whatever, 700
- Madrid 40.4,-3.7 230, 6.7, 640
- New York 40.7,-74.0 1800, 20.1, 1400
- Los Angeles 34.05,-118.24 1100, 13.2, 900
- Chicago 41.88,-87.63 770, 9.5, 950
- Atlanta 33.75,-84.39 470, 6.1, 1250 (ATL busiest)
- Dallas 32.78,-96.8 620, 7.9, 1000
- Mexico City 19.43,-99.13 520? ~, 22.0, 700
- São Paulo -23.55,-46.63 430, 22.4, 620
- Buenos Aires -34.6,-58.38 210, 15.4, 380
- Sydney -33.87,151.2 400, 5.4, 520
- Auckland? maybe skip. Johannesburg -26.2,28.05 180, 6.3, 350
- Cairo 30.04,31.24 190, 21.8, 420
- Lagos 6.45,3.4 90, 15.9, 300
- Nairobi? optional. Toronto 43.65,-79.38 680? ~, 6.3, 620. Vancouver? skip.
- Honolulu? for trans-Pacific arcs maybe Anchorage? skip.
- San Francisco 37.77,-122.42 1000? ~, 4.9, 780. Include SF, skip Seattle.

That's ~33. Good.

Country numeric ISO codes for choropleth (world-atlas ids are numeric strings):
China 156, Japan 392, South Korea 410, Singapore 702 (might be too small in 110m? Singapore may be missing in 110m — tiny; that's fine, skip gracefully), Thailand 764, India 356, UAE 784, Turkey 792, Russia 643, UK 826, France 250, Germany 276, Netherlands 528, Spain 724, USA 840, Mexico 484, Brazil 76, Argentina 32, Australia 36, South Africa 710, Egypt 818, Nigeria 566, Canada 124.

Country value = sum of its cities' values (for current metric/year). Color ramp by value.

Routes (~36): hub pairs:
PEK-LAX, PEK-SFO? use city keys. Define routes by city index pairs + airline + flight number + weekly frequency. Generate flight numbers from airline code. Airlines: CA Air China, MU China Eastern, CZ China Southern, HU Hainan, NH ANA, JL JAL, KE Korean, SQ Singapore, EK Emirates, TK Turkish, BA British, AF Air France, LH Lufthansa, AA American, UA United, DL Delta, QF Qantas, etc.

Route list (from, to, airline, flight number, weekly):
- Beijing↔Los Angeles CA983
- Beijing↔London CA855? fine
- Shanghai↔San Francisco MU587
- Shanghai↔Tokyo MU523
- Guangzhou↔Sydney CZ325
- Shenzhen↔Singapore CZ8057? ok
- Chengdu↔Frankfurt CA431
- Hong Kong↔London BA? use CX Cathay CX251
- Hong Kong↔Sydney CX100
- Tokyo↔Los Angeles NH106
- Tokyo↔Singapore NH841? JL
- Seoul↔Los Angeles KE17
- Seoul↔Dubai? KE951? ok
- Singapore↔London SQ322
- Singapore↔Sydney SQ231
- Singapore↔Dubai EK? use SQ318? whatever EK318? Flight numbers just plausible: EK349 SIN-DXB.
- Bangkok↔Tokyo TG676? Thai TG.
- Delhi↔Dubai EK511? use AI131 Air India.
- Mumbai↔London AI111? use BA198? either.
- Dubai↔London EK001
- Dubai↔Paris EK073? EK075.
- Dubai↔São Paulo EK261
- Istanbul↔New York TK001? TK1.
- Istanbul↔Cairo TK694.
- London↔New York BA215? use BA117? BA Concorde legacy lol; BA243? whatever BA178? fine.
- Paris↔New York AF006? AF022.
- Frankfurt↔Chicago LH430? LH432.
- Amsterdam↔Atlanta? KL? skip KL, use DL ATL-AMS DL47.
- Madrid↔Mexico City IB6400? IB6402.
- New York↔Los Angeles AA001? use AA1? "AA 100"? fine.
- New York↔São Paulo AA950? use LATAM LA8067? keep AA973? whatever.
- Chicago↔Tokyo NH111? NH111 is ORD-NRT yes.
- Atlanta↔Paris AF? DL285? ok.
- Dallas↔London AA050? AA50? fine.
- Mexico City↔Los Angeles AM? Aeroméxico AM570? ok.
- São Paulo↔Buenos Aires AR? Aerolíneas AR1130? ok.
- Sydney↔Auckland? Auckland not included; Sydney↔Los Angeles QF11.
- Johannesburg↔Dubai EK762? ok.
- Cairo↔Moscow? SU401? ok.
- Lagos↔London BA075? ok.
- Toronto↔London AC848? ok. Toronto included? said maybe; include Toronto and Vancouver? just Toronto.
- Moscow↔Beijing CA? SU200? ok.

That's ~40 routes. Each route: weekly frequency (flights/week), aircraft type (77W, 789, A359, 748?), distance computed from arc length * R_earth (6371km) — compute great-circle distance from lat/lons for display.

Route intensity for flights metric: width/brightness ∝ weekly frequency.

Hover tooltip content: "CA983 · Beijing Capital ⇌ Los Angeles | Aircraft: B777-300ER | Weekly: 21 flights | Distance: 10,400 km".

City click popup: panel with city name CN/EN, country, per-metric values with animated counters, sparkline SVG across 2019-2024 for selected metric, rank. Position: project 3D→screen, clamp to viewport; hide when city rotates to back (dot product with camera < 0). Update position each frame while open. Close button + click elsewhere to close.

Also "click city to show info popup" — done.

HUD additions:
- Top-left: title "寰宇 · GLOBAL PULSE" or "星轨地球"? Something distinctive: "寰宇数据台 ORBIS DATA CONSOLE". Title: "寰宇脉搏 · TERRA PULSE". Let's pick: main title "寰宇视野 GLOBAL LENS"? Go with "寰宇数据仪 · ORBIS". Hmm keep: big display "ORBIS · 寰宇数据地球" subtitle. OK.
- Bottom-left: cursor lat/lon readout + altitude/zoom level.
- Top-right: control panel.
- Bottom center: timeline bar with play button, year ticks, current year big display.
- Legend: heat gradient bar bottom right, labels min/max.

Loading screen: full overlay with animated orbital spinner + progress %, fade out.

Fonts: Google fonts link: Rajdhani:wght@500;600;700 & Noto Sans SC:wght@400;500;700 & maybe "Saira Condensed"? Keep 2 families. Display: Rajdhani for Latin/numbers; Chinese headings fall back to Noto Sans SC bold — acceptable pairing.

Colors:
- bg space: #030612
- panel: rgba(10,16,32,0.82), 1px border rgba(120,160,255,0.18)
- text: #e8eefc, muted #8fa3c8
- accent gold #f5b64a (GDP), cyan #46d8ff (flights), coral #ff7a6b (population).
- Buttons active: metric color tint.
Panel corners: clip-path chamfered corners for HUD look (avoid uniform rounded-2xl). Chamfered corners via clip-path polygon — distinctive.

Scanlines/texture? Subtle: faint grid on panel background? Keep light.

Now code structure (single HTML):

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
meta, title, fonts, <style>...</style>
<script type="importmap">...</script>
</head>
<body>
  <div id="app">
    <canvas id="scene"? > (renderer creates canvas; append)
    overlays...
  </div>
  <script type="module"> main </script>
</body>
```

Main module:
1. Imports: THREE from 'three', * as topojson from 'topojson-client' (dynamic import with catch for fallback).
2. Constants: R=1.
3. Data arrays.
4. Scene setup: renderer (antialias, setPixelRatio clamp 2), scene, camera (fov 42, pos z ~3.2), no lights needed (custom shader) but add ambient for standard materials (rings use basic).
5. Stars.
6. Earth group: earth mesh (SphereGeometry(1, 96, 96) with ShaderMaterial), atmosphere shell, heat overlay sphere, city group, route group, dot points.
7. Texture loading with THREE.LoadingManager; build materials after load; procedural fallback on error.
8. Controls: custom drag/zoom + inertia + idle auto-rotate.
9. Raycaster interactions.
10. UI wiring: metric buttons, layer toggles, timeline play/slider/speed, popup, tooltip, legend, counters.
11. Heat canvas builder (async geojson fetch).
12. Animation loop: time uniforms, marker pulse, route flow, dot movement, earth rotation, camera smoothing, popup reposition, stats.

Edge cases:
- Pointer events on canvas only (UI panels stopPropagation).
- Resize handler.
- prefers-reduced-motion? optional, skip.

Let me write the earth shader carefully.

Vertex:
```
varying vec2 vUv; varying vec3 vNormalW; varying vec3 vPosW;
void main(){ vUv=uv; vNormalW = normalize(mat3(modelMatrix)*normal); vec4 wp = modelMatrix*vec4(position,1.); vPosW=wp.xyz; gl_Position = projectionMatrix*viewMatrix*wp; }
```
Note: non-uniform scale? none. modelMatrix includes earthGroup rotation — good, normal in world space rotates with earth. Sun direction in world space fixed → terminator fixed in world, earth rotates under it.

Fragment:
```
uniform sampler2D dayMap, nightMap, waterMap; uniform vec3 sunDir; uniform vec3 camPos; uniform float uHasWater;
varying...
void main(){
 vec3 n = normalize(vNormalW);
 float sunDot = dot(n, normalize(sunDir));
 float dayMix = smoothstep(-0.15, 0.25, sunDot);
 vec3 day = texture2D(dayMap, vUv).rgb;
 vec3 night = texture2D(nightMap, vUv).rgb;
 // lighting
 float diff = clamp(sunDot,0.,1.);
 vec3 dayCol = day * (0.25 + 0.9*diff);
 // slight warm tint at terminator
 dayCol += vec3(0.12,0.05,0.0)* pow(1.-abs(sunDot), 3.) * step? use smooth band:
 float dusk = smoothstep(-0.25,0.,sunDot)*smoothstep(0.35,0.,sunDot); // peaks near 0
 dayCol = mix(dayCol, dayCol*vec3(1.25,0.9,0.7), dusk*0.6);
 vec3 nightCol = night*vec3(1.15,1.05,0.9)*1.35 + vec3(0.01,0.02,0.045); // faint blue base
 vec3 col = mix(nightCol, dayCol, dayMix);
 // ocean specular
 vec3 v = normalize(camPos - vPosW);
 vec3 h = normalize(normalize(sunDir)+v);
 float spec = pow(max(dot(n,h),0.),42.)* uHasWater * texture2D(waterMap,vUv).r * dayMix;
 col += vec3(0.5,0.65,0.8)*spec*0.55;
 // rim atmosphere tint
 float fres = pow(1.-max(dot(n,v),0.), 2.5);
 col += vec3(0.25,0.5,1.0)*fres*(0.25+0.5*dayMix);
 gl_FragColor = vec4(col,1.);
}
```
camPos uniform = camera.position (world) each frame.

Atmosphere shell shader (BackSide, additive):
```
varying vNormalW, vPosW;
frag: v=normalize(camPos-vPosW); intensity = pow(0.62 - dot(vNormal? ...) )
```
Standard: for BackSide sphere, use dot(viewDir, normal) trick:
`float i = pow(max(0., dot(v, -n) )... ` hmm. Common glow: material side BackSide, intensity = pow(0.65 - dot(vNormal, vec3(0,0,1) in view), c). Let me compute in world: n outward; for backside fragments, n points away from viewer roughly; glow = pow(smoothstep(-0.1,0.6?, dot(v, n))...). Let me think: on backside sphere (radius 1.15), visible fragments are on far side; their normals point away from camera → dot(v, n) is negative near center, approaches ~0 at limb? Actually at silhouette edge of shell, dot(v,n)→0… For halo that's strongest just outside earth's limb and fades outward: shell radius 1.16; brightness peaks where line of sight is tangent to earth (r=1). Parameterize: d = dot(v, n). Backside: n points away; d is -1 at shell center (behind earth, occluded by earth anyway) → 0 at shell silhouette. Earth limb occurs where… simpler empirical: intensity = pow(clamp(d + 1.0, 0., 1.), 3.5)? d=-1→0, d=0→1 gives max at outer silhouette — wrong direction (want fade outward). Hmm actually halo that fades outward: brightest at inner edge (near earth limb) fading to 0 at shell edge. Inner edge (tangent to earth surface) has d = ? Point at distance r=1.16 from center, angle between outward normal and view direction to camera: cos = sqrt(1 - (1/1.16)^2)? For tangent-to-earth point: sinθ = 1/1.16 → angle between normal and view ~… let me just: f = dot(v, n) ∈ [-1, 0] visible range. At earth-tangent point, f = -sqrt(1-(1/1.16)^2) ≈ -0.506. At shell silhouette f=0. So intensity = smoothstep(0.0, -0.55, f) i.e. peaks at f≈-0.55 (inner), zero at f=0 (outer). Use: `float i = smoothstep(0.05, -0.5, f)`? smoothstep with edge0>edge1 is undefined-ish; use `float i = 1.0 - smoothstep(-0.55, -0.05, f);` f=-0.55→1, f=-0.05→0. Then pow for falloff: i = pow(i, 2.2). Also modulate by sun: brighter on day side: s = clamp(dot(n?, sunDir)) — use position direction: dir = normalize(vPosW - earthCenter(0)); sunFactor = smoothstep(-0.5,0.5, dot(dir, sunDir)); color = mix(nightBlue dim, dayCyan, sunFactor). Additive blending, depthWrite false. Also slight warm at terminator: skip.

Also faint inner glow handled in earth shader rim.

City marker glow texture: generate radial gradient canvas once (white center → transparent), tint via sprite material color.

Ring pulse: mesh with RingGeometry(0.02,0.024,48) oriented tangent: mesh.lookAt(pos*2) i.e. orient along normal. Animate scale 1→2.6 and opacity 0.9→0 looping, offset per city. AdditiveBlending, MeshBasicMaterial transparent. Cheap.

City core sphere radius: base 0.011 + norm*0.012 (R=1). Glow sprite scale ~6x core.

Value normalization: norm = (v - min)/(max-min) across cities for current metric and year, use sqrt for size.

Metric colors: GDP #ffb54d, population #ff6f61, flights #4fd6ff.

Route color: base cyan-ish but tinted slightly by route "metric intensity"? Keep route color independent of metric (since they're flights) — but when metric=flights, route brightness scales with frequency more strongly. Route base color: #57c8ff with warm alternate? Use two-tone: routes colored by length: short #5ad0ff, long-haul #9a8bff? Subtle variation adds richness. Assign per-route hue between cyan and violet-blue based on distance. Avoid indigo-violet-pink gradient cliché — just individual route colors, fine, keep mostly cyan family with a few teal.

Heat ramp: define per metric:
- GDP: ["#0b2a5a","#155e97","#2f9e8f","#8fc64a","#ffd24d"]? That's viridis-ish. Or gold ramp: #12304f→#2c6f8f→#3fae9c→#e8c15a→#ffdf8a.
- Population: #143a5e→#7a4a9e? avoid violet… pop ramp: #173d5c→#4e6f8f? hmm want coral end: #123252, #33607e, #7d8f6f? messy. Use sequential: deep blue → teal → warm coral: ["#123a63","#1f6f8b","#57a17f","#d9a441","#ff6b4a"].
- Flights: ["#0d2b52","#185a9d","#2f9edb","#7fd4ff","#e8fbff"].
Interpolate in JS (lerp between stops).

Legend shows ramp gradient + min/max labels (formatted with units: GDP billion USD, population million, flights flights/day).

Country value mapping: countryId → aggregate. Also color countries with no cities? leave transparent.

Draw countries: need Path2D from geojson polygon coords: for each ring: moveTo/lineTo with x=(lon+180)/360*W, y=(90-lat)/180*H. Fill with evenodd? Use Path2D with 'evenodd' fill rule for holes. ctx.fill(path, 'evenodd').

Also stroke with thin line rgba(140,190,255,0.25) width 1 for boundary glow.

Canvas 2048x1024 might be heavy for texture memory but fine.

GeoJSON fetch: `fetch('https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json')` then `topojson.feature(data, data.objects.countries)`. Import topojson-client via importmap "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm". If fetch/import fails → fallback blob heat.

Timeline mechanics:
- state.yearFloat ∈ [2019, 2024]. Play: yearFloat += dt*speed (speed years/sec, e.g., 0.6). Loop at end → wrap to 2019.
- On integer year change → rebuild heat + update "year" display + stats.
- Slider: input range 0..500 → yearFloat = 2019 + v/500*5.
- Year ticks 2019..2024 below slider.
- Current year display: big Rajdhani number + progress fill on slider (styled range with gradient fill via background-size trick or custom track overlay). Style range input with accent-color and add tick labels; plus a progress bar behind. Keep custom: div track with fill width % + draggable? Simpler: input[type=range] styled with webkit thumb; fill via linear-gradient background updated in JS.

Stats panel (in controls or top-left): global total for metric at current year + top 3 cities mini list with animated bars? Add "TOP 5 ranking" list in panel with animated bar widths — updates with year. This adds data richness. Bars colored by metric.

Panel sections:
1. Data Metric (3 buttons with icons: GDP ¥/$, population, flights ✈)
2. Layers (toggles: city markers, flight routes, regional heat, atmosphere glow, auto-rotate, city lights? night lights are inherent; toggle "clouds"? no clouds texture… could add clouds layer texture? three-globe has no clouds; threejs examples has earth_clouds_1024.png? examples/textures/planets/earth_clouds_1024.png exists. If we load from threejs.org… mixing hosts; skip clouds to reduce failure points. Toggles: markers/routes/heat/atmosphere/stars? auto-rotate.)
3. Ranking TOP5 (auto)
Bottom bar: timeline.

Also small "compass"/view reset button.

Tooltip: absolutely positioned div, follows mouse with offset, HTML content, pointer-events none.

City click popup card:
- Header: city name CN + EN + country
- Rows: GDP value (billion USD), resident population (millions), daily flights (flights/day) — highlight active metric
- Sparkline of active metric 2019→2024 with dot on current year
- Rank badge
- Close ×
Position near projected point with connector line? Connector is complex; just offset card + small caret. Reposition each frame; hide if behind globe (check dot product of normalized city world position and camera direction: compute city world pos, vector to camera; if dot product of (cityPos - earthCenter) with (cameraPos - cityPos) < 0 → occluded by horizon? Approximation: angle between city normal and camera direction: visible if dot(normalize(cityWorld), normalize(camPos)) > ~0.15? Actually visible if dot product of city normal and camera direction > 0 (front hemisphere) — use > 0.05. Also raycast occlusion not needed.

Cursor lat/lon: on pointermove, raycast to earth sphere; if hit, convert to lat/lon from earth-local point (apply earthGroup's inverse matrix!). Since earth rotates, need local coords: earthGroup.worldToLocal(point). Then lat = asin(y/r), lon = atan2(?,?) matching our mapping. Our latLonToVec3:
```
const phi = (90-lat)*deg2rad; const theta = (lon+180)*deg2rad;
x = -r sin(phi) cos(theta); z = r sin(phi) sin(theta); y = r cos(phi);
```
(standard three-globe mapping that aligns with equirectangular textures). Inverse: lat = 90 - acos(y/r)*rad; lon = ((atan2(z, -x))*rad) - 180 → normalize to [-180,180]. Verify: from forward: -x = s·cosθ, z = s·sinθ → θ = atan2(z, -x); lon = θ·rad - 180. Yes.

Auto-rotation: earthGroup.rotation.y += rotSpeed*dt when idle (no drag for 3s) and toggle on. rotSpeed ~0.05 rad/s. Also rotX tilt fixed initial -0.25? Tilt earth slightly: earthGroup.rotation.x = 0.15? Drag adjusts rotY and rotX (clamp rotX to ±0.6). Hmm rotating group on X after Y rotation produces tumbling order YXZ; set earthGroup.rotation.order='YXZ'? With YXZ, rotation.y spin + rotation.x tilt works intuitively. Actually with drag: horizontal → rotY, vertical → rotX clamp [-1.0, 1.0]? Clamp to ±0.9 to avoid weirdness. Fine. But wait: heat overlay, markers, atmosphere are all children of earthGroup → rotate together. Atmosphere shading depends on world normal — fine.

Sun direction: fixed, e.g., normalize(1, 0.25, 0.4)? If earth rotates on Y, terminator sweeps longitudinally. Also slowly animate sun longitude for extra motion: sunAngle += dt*0.01. Keep mostly static + tiny drift.

Initial view: camera at (0, 0.6, 3.1) looking at origin; earth starts with rotation.y such that Asia-Pacific faces camera? Compute: lon 105°E to face camera at +Z. theta=(105+180)=285°… forward mapping places lon at angle theta around Y with x=-cos, z=sin. Point at lon L is at direction θ=(L+180)°. Its world position after group rotY α: angle θ+α… we want resulting direction ≈ +Z which corresponds to θ_total=90° (since z=sinθ max at 90°). So α = 90° - (L+180°) = -90°-L → for L=105: α=-195° ≡ 165°. Set initial targetRotY = THREE.MathUtils.degToRad(165). Fine — tune visually anyway; pick value that shows Asia/Pacific.

Route arc: build curve points:
```
function arcBetween(a,b,h){ // a,b Vector3 unit
 const angle = a.angleTo(b);
 const pts=[]; const N=72;
 for(i=0..N){ t=i/N;
   // slerp
   const p = slerpVec(a,b,t).multiplyScalar(R + h*Math.sin(Math.PI*t));
 }
}
```
slerp: use THREE.Vector3().copy(a).applyAxisAngle? Easiest: Quaternion.setFromUnitVectors? For slerp: use `new THREE.Quaternion().setFromUnitVectors(a, b)` then rotate a by t*angle: q.setFromUnitVectors(a,b); axis-angle… Simpler: use formula: sin((1-t)Ω)/sinΩ * a + sin(tΩ)/sinΩ * b. Implement directly. Handle antipodal-ish (none in data). h = 0.18*angle? angle in rad max ~π; h = 0.05 + angle*0.16, clamp 0.45.

Route geometry: BufferGeometry from points; attribute aT = i/N. Also store curve object for dot movement (store points array and interpolate by index — cheaper: keep curve as CatmullRom of control points? just store points array and interpolate between segments by t*(N)).

Route shader:
```
vertex: attribute float aT; varying float vT; void main(){ vT=aT; gl_Position=... }
frag: uniform vec3 uColor; uniform float uTime,uSpeed,uOpacity,uHover,uWidth?;
 float head = fract(vT - uTime*uSpeed);
 float pulse = smoothstep(0.18,0.0,head); // bright head trailing behind
 float head2 = fract(vT*1.0 - uTime*uSpeed + 0.5); second pulse?
 base = 0.28 + uHover*0.5;
 float glow = pulse*(1.2+uHover);
 vec3 col = uColor*(base + glow*1.6);
 float alpha = (base*0.8 + glow)*uOpacity;
 gl_FragColor = vec4(col, alpha);
```
Blending additive, transparent true, depthWrite false. Additive on bright day side will wash out slightly — acceptable (routes are data).

Also fade routes on back side? depthTest true keeps them occluded by earth — yes keep depthTest true so back-side routes hidden. Markers same.

Moving dots: THREE.Points with PointsMaterial(size ~5, sizeAttenuation true, additive, map glow, vertexColors). Update positions each frame: for each route, tDot = fract(time*speed*? + offset) → position along points array. Color = route color. Per-point size not possible in PointsMaterial; use ShaderMaterial? Keep single size ~0.02 world via attenuation. Fine.

Performance: 33 cities × (sphere + sprite + ring) ≈ 100 draw calls + 40 routes + earth + atmo + heat + stars ≈ 150 draw calls — fine.

Raycast targets: cityCoreMeshes array; routeLines array. On move: raycast cities first (closer priority), else routes with threshold. Set raycaster.params.Line.threshold = 0.018, also Points threshold to avoid. Note: Line raycast in world space — threshold in world units; camera distance ~3 → 0.018 reasonable.

Hover city: cursor pointer, tooltip "City · GDP xxx", marker glow boost (lerp scale).
Hover route: tooltip with flight info; uHover lerp.

Click: if city → open popup; if empty space click (not drag) → close popup.

Distinguish drag vs click: track pointerdown position and time; on up, if distance<6px → click.

Touch: unify with pointer events; pinch zoom? implement basic two-pointer distance → zoom. Add simple pinch handling for completeness (pointer events cache). Keep code moderate.

Now UI styling details:

Font import:
```
<link href="https://fonts.googleapis.com/css2?family=Rajdhani:wght@500;600;700&family=Noto+Sans+SC:wght@400;500;700;900&display=swap" rel="stylesheet">
```

CSS: dark space bg with radial vignette behind canvas (canvas covers anyway). Panels: clip-path chamfer:
`clip-path: polygon(12px 0, 100% 0, 100% calc(100% - 12px), calc(100% - 12px) 100%, 0 100%, 0 12px);`
Border via outline? clip-path removes border visibility at cut corners; use background + inner 1px box-shadow inset; or wrap with gradient border trick. Simpler: keep border:1px solid rgba(...); clip-path cuts border at corners leaving gaps — actually stylistically acceptable? Gap looks like missing border along diagonal. Alternative chamfer via corner background image linear-gradient: skip perfection: use small 2px radius + top accent line + corner notch decorations (::before L brackets). HUD corner brackets look great: panel with thin border, plus 4 corner bracket elements. Only use brackets on main panel.

Metric buttons: segmented, each with color dot + label + live total value below? Show unit.

Toggles: custom checkbox → switch style.

Timeline bottom bar: full-width strip with play button (SVG), year big display (Rajdhani 40px), slider with ticks, speed selector (0.5×/1×/2×).

Title block top-left:
```
<div class="brand">
  <div class="brand-en">ORBIS · DATA GLOBE</div>?
```
Let me craft: small eyebrow: "REALTIME PLANETARY INDEX" then big "寰宇脉搏" then sub "Global City · Route · Economy Data Visualization". Plus status line with blinking dot "系统在线 · 数据源 SIM-2024".

Bottom-left HUD: cursor coords + zoom + FPS.

Legend bottom-right: gradient bar + title "Regional Heat · GDP".

Loading overlay: center: rotating dashed ring + orbiting dot CSS, progress %, tip text.

Also handle WebGL not available → show message.

Now let me write the data code.

Cities array fields: id, cn, en, country (cn), cc (ISO numeric string), lat, lon, gdp (2019 base, billion USD), pop (millions), flt (daily departures), g (generic annual growth rate). Values computed:
```
YEAR0=2019, YEARS=6 (2019..2024)
GLOBAL = {
 gdp: [1.00,0.965,1.03,1.10,1.18,1.27],
 pop: [1.00,1.006,1.012,1.019,1.026,1.033],
 flt: [1.00,0.52,0.68,0.85,0.99,1.12],
}
value(city, metric, yf): base * GLOBAL[metric][floor] interp? For continuous: interpolate GLOBAL at yf (lerp between integer years) * (1+city.g)^(yf-2019) * (1+0.02*sin(yf*2.1+hash))
```
Keep wobble small or zero for GDP/pop; flights maybe slight. Skip wobble for clarity of numbers.

City-specific growth: give each city a growth multiplier per metric? Simplify: single growth factor g (e.g., 0.03 for emerging, 0.015 for developed) applied to gdp and flt; pop uses g*0.3. Fine.

Ranking computed at current metric/year.

Routes array: [fromId, toId, airline, flightNo, weekly, aircraft]. Distance computed via haversine.

Route brightness ∝ weekly/ maxWeekly.

Now heat legend min/max: from country aggregates.

Number formatting: gdp: e.g., "1,842 billion USD" (value in 100M? base is billion; after growth NY 2024 ≈ 1800*1.27≈2286 billion USD). Display with toLocaleString.

Popup sparkline: SVG polyline across 6 years for active metric, normalized; marker at current yearFloat.

Also on metric change: update legend title/ramp, marker colors, heat redraw, ranking, button states, popup if open.

Auto-rotate toggle default on; pause on interaction (resume after 4s idle if toggle on).

Also "earth rotation animation" satisfied by idle spin.

Day/night: sun fixed; add small sun drift.

City lights: night texture + marker glows brighter on night side? Marker glow sprite: could modulate opacity by sunDot at city position (world normal vs sunDir) → glow more prominent at night. Nice: in animate, for each city compute sunDot from world position; glowMaterial opacity = lerp(0.35, 1.0, nightFactor). 33 sprites each with own material (clone) — fine.

Ring color = metric color.

Now let me write code. Long; organize with section comments. Careful with correctness — no chance to test, so be conservative with Three.js API (r160):

- THREE.SphereGeometry ok.
- ShaderMaterial with uniforms.
- CanvasTexture.
- TextureLoader with LoadingManager: manager.onProgress(url, loaded, total).
- texture.colorSpace = THREE.SRGBColorSpace for color textures (r152+). renderer.outputColorSpace default SRGB in r152+. Set renderer.toneMapping = THREE.ACESFilmicToneMapping? Tone mapping might dull night lights; use NoToneMapping default and keep colors as authored. Keep default (NoToneMapping) — shader outputs final color; gl_FragColor bypasses tonemapping? ShaderMaterial ignores tone mapping unless included. Fine.
- renderer.setClearColor(0x030614, 1).

Stars: BufferGeometry positions on sphere radius 40-60, PointsMaterial size 0.05? sizeAttenuation true, size ~0.35, opacity varying? Use vertex colors for brightness variation; material vertexColors true, transparent, opacity .9, depthWrite false. Twinkle: cheap — rotate starfield slowly + subtle material.opacity oscillation. Fine.

Also add subtle nebula? Skip; maybe large gradient sprite behind? Keep stars 2 layers (near brighter fewer, far dimmer many).

Camera: fov 40, near 0.1, far 100, pos (0, 0.55, 3.4). lookAt origin each frame after zoom change (camera stays looking at origin; camera position fixed direction, only distance changes along its direction). Implement zoom as scalar distance: camera.position.setLength(dist). Initial direction normalized (0,0.35,1).

Drag rotate: apply to earthGroup.

Pinch: track two pointers, distance delta → targetDist.

Wheel: targetDist *= 1 + deltaY*0.001 clamp [1.6, 6].

Zoom display: altitude km = (dist-1)*6371.

Popup projection: vector.project(camera) → screen px.

Heat overlay sphere: radius 1.002, same segment count; material MeshBasicMaterial({map, transparent:true, depthWrite:false}); renderOrder above earth. But should be occluded by earth on back side — depthTest true default, and overlay is slightly larger so front is fine, back hidden by earth? Back side of overlay sphere is behind earth → depthTest against earth (which is drawn writing depth) hides it. Earth drawn first (renderOrder), overlay after. Set earth.renderOrder=0, heat=1, routes=2, markers=3, atmo=4? Atmosphere backside should render behind earth? Backside shell with additive should render after earth with depthTest true — earth occludes its central portion; visible ring remains. OK.

Actually routes depthTest true: arcs are above surface, front arcs visible, back arcs occluded by earth. Good.

Sprites (glow) depthTest true might clip into sphere at horizon; keep depthTest true to hide back cities. Sprite at surface position: half embedded; sprite faces camera at that point; if city on limb, sprite center on surface → visible. Back-side cities: occluded by earth depth. Good.

City core sphere: place slightly above surface: pos * 1.005.

Ring: place at pos*1.004, orient: ring mesh lookAt(center*2?) ring plane perpendicular to normal: mesh.lookAt(pos.multiplyScalar(2)) — lookAt orients +Z toward target; RingGeometry lies in XY plane facing +Z → correct.

Popup connector: skip.

Timeline heat rebuild: on integer year change or metric change → rebuildHeat(). rebuildHeat draws canvas (sync, ~ms). Also update legend labels.

Ranking update: on year integer change and metric change; animate bar widths via CSS transition.

Counter (global total): animate numeric tween on change; also update on integer year change.

Also show "current year" big display updating continuously during play (show yearFloat.toFixed(1)? show integer year + progress). Big year = floor(yearFloat); small decimal progress bar.

Slider sync during play (update value unless user is dragging).

Speed buttons: 0.5x 1x 2x (years/sec = 0.45*speed).

Play button toggles ▶/❚❚ SVG.

Also "回到现在" button? Skip.

View reset button (⌂): reset targetRotY/X and dist with tween.

Edge: pointer events on canvas only (UI panels stopPropagation).

Resize handler.

prefers-reduced-motion? optional, skip.

Let me write earth shader carefully.

Vertex:
```
varying vec2 vUv; varying vec3 vNormalW; varying vec3 vPosW;
void main(){ vUv=uv; vNormalW = normalize(mat3(modelMatrix)*normal); vec4 wp = modelMatrix*vec4(position,1.); vPosW=wp.xyz; gl_Position = projectionMatrix*viewMatrix*wp; }
```

Let me write the shader code concretely.

Earth material:

```js
const earthUniforms = {
  dayMap:{value:null}, nightMap:{value:null}, waterMap:{value:null},
  sunDir:{value:sunDir}, camPos:{value:new THREE.Vector3()}, uHasWater:{value:0}
};
const earthMat = new THREE.ShaderMaterial({
 uniforms: earthUniforms,
 vertexShader:`
  varying vec2 vUv; varying vec3 vN; varying vec3 vP;
  void main(){
    vUv = uv;
    vN = normalize(mat3(modelMatrix) * normal);
    vec4 wp = modelMatrix * vec4(position,1.0);
    vP = wp.xyz;
    gl_Position = projectionMatrix * viewMatrix * wp;
  }`,
 fragmentShader:`
  uniform sampler2D dayMap, nightMap, waterMap;
  uniform vec3 sunDir, camPos; uniform float uHasWater;
  varying vec2 vUv; varying vec3 vN; varying vec3 vP;
  void main(){
    vec3 n = normalize(vN);
    vec3 sd = normalize(sunDir);
    float sa = dot(n, sd);
    float dayMix = smoothstep(-0.12, 0.22, sa);
    vec3 day = texture2D(dayMap, vUv).rgb;
    vec3 night = texture2D(nightMap, vUv).rgb;
    float diff = max(sa, 0.0);
    vec3 dayCol = day * (0.18 + 1.05*diff);
    float dusk = smoothstep(-0.3, 0.05, sa) * (1.0 - smoothstep(0.05, 0.45, sa));
    dayCol *= mix(vec3(1.0), vec3(1.28,0.92,0.68), dusk*0.65);
    vec3 nightCol = night * vec3(1.35,1.18,0.92) + vec3(0.008,0.016,0.038)*(1.0-dayMix);
    vec3 col = mix(nightCol, dayCol, dayMix);
    vec3 v = normalize(camPos - vP);
    vec3 h = normalize(sd + v);
    float spec = pow(max(dot(n,h),0.0), 48.0) * uHasWater * texture2D(waterMap, vUv).r * dayMix;
    col += vec3(0.55,0.7,0.85) * spec * 0.5;
    float fres = pow(1.0 - max(dot(n, v), 0.0), 3.0);
    col += vec3(0.22,0.5,1.0) * fres * (0.16 + 0.45*dayMix);
    gl_FragColor = vec4(col, 1.0);
  }`
});
```

Atmosphere:

```js
const atmoMat = new THREE.ShaderMaterial({
 uniforms:{ sunDir:{value:sunDir}, camPos:{value:new THREE.Vector3()} },
 vertexShader: same pattern,
 fragmentShader:`
  uniform vec3 sunDir, camPos; varying vec3 vN; varying vec3 vP;
  void main(){
    vec3 n = normalize(vN);
    vec3 v = normalize(camPos - vP);
    float f = dot(v, n);           // backside: negative
    float ring = 1.0 - smoothstep(-0.58, -0.06, f);
    ring = pow(max(ring,0.0), 2.0);
    vec3 dir = normalize(vP);
    float dayF = smoothstep(-0.35, 0.45, dot(dir, normalize(sunDir)));
    vec3 col = mix(vec3(0.05,0.12,0.32), vec3(0.3,0.62,1.0), dayF);
    float dusk = smoothstep(-0.35,0.0,dot(dir,normalize(sunDir)))*(1.0-smoothstep(0.0,0.4,dot(dir,normalize(sunDir))));
    col += vec3(0.9,0.45,0.15)*dusk*0.25;
    gl_FragColor = vec4(col * ring * 1.35, ring);
  }`,
 side: THREE.BackSide, transparent:true, depthWrite:false, blending: THREE.AdditiveBlending
});
```
Wait, `dir` should be earth-center-relative position: earth at origin (group at origin), so vP world = fine (group at origin, only rotation). Yes, earthGroup at origin.

Hmm, ring formula: f=dot(v,n). Backside visible fragments: f ∈ [-1,0]. Want intensity 1 at f≈-0.5 (inner/earth limb), 0 at f≈0 (outer edge). smoothstep(-0.58,-0.06,f): f=-0.58→0, f=-0.06→1 → ring=1-that: f=-0.58→1, -0.06→0. But also f < -0.58 (deeper, occluded by earth anyway) → smoothstep clamps to 0 → ring=1 → those fragments are behind earth and depthTest hides them (earth writes depth; atmo sphere radius 1.16, its far-side fragments are farther than earth → occluded where earth covers them; earth angular radius from camera: asin(1/3.4)=17°, atmo shell back center is behind earth center → occluded. Fragments between earth limb and tangent point at f slightly < -0.5… visible ring region between earth limb and shell silhouette corresponds to f ∈ [-0.5,0]. Our ramp covers. Good. depthTest true, renderOrder after earth.

Also add second inner atmosphere? Skip.

Sun drift: sunAngle slowly: sunDir.set(cos(a),0.28,sin(a)) normalize, a += dt*0.008 → very slow. But terminator motion mostly from earth spin 0.05 rad/s. OK.

City marker construction:

```js
const glowTex = makeGlowTexture(); // canvas 128 radial white
cities.forEach(c=>{
 const pos = latLonToVec3(c.lat,c.lon,1.0);
 const g = new THREE.Group(); g.position.copy(pos).multiplyScalar(1.004);
 // orient group outward: g.lookAt(pos*2)? Group at pos, lookAt(2pos) orients +Z outward.
 core = Mesh(SphereGeometry(1,16,16) scaled later, MeshBasicMaterial(color metric))
 sprite glow...
 ring...
});
```
Simpler: add core mesh at group origin; glow sprite at same pos (sprite always faces camera, add directly to earthGroup at pos). Ring mesh at pos with lookAt.

Store per city: {core, spriteMat, ring, ringPhase, baseScale}.

Update on metric/year: targetSize = 0.011 + 0.016*sqrt(norm); animate core.scale lerp; color lerp to metric color (material.color.lerp). Sprite scale = size*7. Ring color set.

Night factor for sprite: worldPos = pos applied by earthGroup matrix; n = normalize(worldPos); nightF = smoothstep(0.15,-0.15, dot(n,sunDir)) in JS: use MathUtils.mapLinear clamp. spriteMat.opacity = 0.3 + 0.7*nightF… also base pulse: *(0.85+0.15*sin(t*2+phase)).

Ring animation: s = (t*0.5+phase)%1; ring.scale = size*(1+s*2.2)/0.02? RingGeometry base radius ~1: create RingGeometry(0.75,1,48) then scale to size*… Let's define ring base outer radius 1: RingGeometry(0.82,1.0,48). ring.scale.setScalar(citySize*(1.2+ s*2.4)); opacity=(1-s)*0.85. citySize is core world radius.

Route building:

```js
function buildRoute(a,b){
 const va=latLonToVec3(...,1), vb=...;
 const ang = va.angleTo(vb);
 const N=72; pts=[], ts=[];
 const sinA=Math.sin(ang);
 for(i<=N){ t=i/N;
  let p;
  if(sinA<1e-4) p=va.clone();
  else p = va.clone().multiplyScalar(Math.sin((1-t)*ang)/sinA).add(vb.clone().multiplyScalar(Math.sin(t*ang)/sinA));
  const h = 1 + (0.05 + ang*0.16)*Math.sin(Math.PI*t);
  p.multiplyScalar(h); pts.push(p);
 }
 geometry setFromPoints + aT attribute.
}
```
Wait p after slerp formula is unit length (yes, unit sphere slerp). Multiply by radius factor. Good.

Route material uniforms per route: uColor, uTime (shared via global uniform object? per-material uniform but update all in loop: store material list; set uTime.value=t). Use shared uniforms object reference: `const sharedTime={value:0}` and in each material's uniforms uTime: sharedTime — same object shared; update once.

Route opacity per frequency: uOpacity = 0.55+0.45*(weekly/maxWeekly).

Hover: per-route uHover uniform; lerp toward target in loop.

Dots: positions Float32Array(routes*3), colors; each frame: tDot = fract(clock*speed_i + phase_i); idx = tDot*(N); interpolate pts. Speed_i = 0.12 + 0.1*(weekly/max) cycles/sec? Route traversal ~6-9s. Points geometry needsUpdate.

PointsMaterial({size:0.032, map:glowTex, transparent, depthWrite:false, blending:Additive, vertexColors:true, sizeAttenuation:true}).

Heat layer:

```js
const heatCanvas=document.createElement('canvas'); W=2048,H=1024; ctx.
let countriesGeo=null; // geojson features
async loadGeo(){ try{ const [mod, res] = await Promise.all([import('topojson-client'), fetch(URL).then(r=>r.json())]); countriesGeo = mod.feature(res, res.objects.countries).features; }catch(e){ countriesGeo=null; } rebuildHeat(); }
```
topojson-client +esm exports {feature}. `import * as topo` then topo.feature.

rebuildHeat():
```
ctx.clearRect
if(countriesGeo){ for each feature: v = countryValue(feature.id); if v==null continue; color=ramp(norm(v)); path=buildPath(feature.geometry); ctx.fillStyle=color; ctx.fill(path,'evenodd'); ctx.strokeStyle='rgba(150,200,255,.28)'; ctx.lineWidth=1; ctx.stroke(path);} 
else { radial blobs per city }
heatTex.needsUpdate=true; update legend
```
countryValue: aggregate cities by cc===id (world-atlas id is numeric string like "156"; some features have undefined id). Cities have cc string.

Also stroke all countries faintly even without data for context: stroke with rgba(120,160,220,0.10) — nice map feel. Do: first pass stroke all, second pass fill data.

Ramp normalization: min/max of country values; norm=(v-min)/(max-min || 1); apply sqrt curve for spread: n^0.7.

Ramp function: stops array of hex; lerp between.

Legend: CSS gradient built from same stops.

Popup & tooltip DOM.

Interaction wiring: canvas 'pointerdown/move/up', 'wheel', also 'pointerleave'.

Raycaster from pointer NDC.

Hover state machine: hoverCity, hoverRoute; update tooltip position/content on move.

Click detection: on pointerup if !moved → pick city → open popup; else close.

Popup content builder with sparkline SVG:
```
points across years: x = i/(YEARS-1)*w; y = h - (v-min)/(max-min)*(h-6)-3
```
Include area fill gradient.

Also show rank: compute rank of city by metric.

Now controls panel markup:

```html
<aside id="panel">
  <div class="panel-head">
    <span class="dot live"></span> 控制台 CONSOLE
  </div>
  <section>
    <h3>数据指标 <em>METRIC</em></h3>
    <div class="seg">
      <button data-m="gdp" class="on">GDP</button>
      <button data-m="pop">人口</button>
      <button data-m="flt">航班量</button>
    </div>
    <div class="total"><span id="totalVal">—</span><span id="totalUnit"></span></div>
  </section>
  <section>
    <h3>图层 <em>LAYERS</em></h3>
    toggles...
  </section>
  <section>
    <h3>城市排行 <em>TOP 5</em></h3>
    <ol id="rank"></ol>
  </section>
</aside>
```

Timeline:

```html
<footer id="timeline">
 <button id="play">svg</button>
 <div class="year"><span id="yearBig">2019</span><span class="unit">年</span></div>
 <div class="track">
   <input type="range" id="yearRange" min="0" max="1000" value="0">
   <div class="ticks">2019..2024 labels</div>
 </div>
 <div class="speed"><button data-s="0.5">0.5×</button><button data-s="1" class="on">1×</button><button data-s="2">2×</button></div>
</footer>
```

Legend:

```html
<div id="legend"><div class="lg-title">区域热力 · <span id="lgMetric">GDP</span></div><div class="lg-bar"></div><div class="lg-lab"><span id="lgMin"></span><span id="lgMax"></span></div></div>
```

HUD bottom-left: `#hud` coords/zoom/fps.

Title top-left `#brand`.

Loading `#loader`.

Tooltip `#tip`, popup `#popup`.

Now CSS: quite a bit. Use CSS variables. Chamfered corners on panel via clip-path + border via ::before for corner brackets? clip-path clips border too but border still shows along non-cut edges; cut corners show background edge without border — generally looks fine if bg differs. Use clip-path chamfer + 1px border; cut corners will have missing border segments of ~12px diagonals — stylistically acceptable? Actually might look broken. Alternative: no clip-path; use corner brackets: panel with 1px border rgba, plus ::before/::after L brackets at top-left/bottom-right in accent color. Clean HUD look, no risk. Do brackets on panel and timeline.

Buttons/toggles with hover transitions, focus-visible outline.

Range input styling: webkit-slider-thumb custom (diamond? 14px square rotated? keep circle with glow), track gradient fill set via JS: `range.style.background = linear-gradient(90deg, var(--acc) p%, rgba p%)`.

Font sizes: brand h1 34px 900 Noto; eyebrow Rajdhani letter-spacing 0.3em 11px.

Colors:
```
--bg:#04070f; --ink:#eaf1ff; --mut:#8ea3c7; --line:rgba(140,170,230,.16);
--panel:rgba(8,14,28,.78); --panel2:rgba(12,20,38,.6);
--gold:#f6b64b; --cyan:#45d6ff; --coral:#ff7059; --acc:var(--cyan) (default metric flights? default metric GDP → gold accent).
```
Dynamic accent: set document.documentElement.style.setProperty('--acc', metricColor).

Metric colors: gdp #f6b64b, pop #ff7059, flt #45d6ff.

Loader: black overlay, ring spinner CSS: dashed border circle rotating + inner dot orbiting; progress text Rajdhani.

Now animation loop order:
```
dt=clock.getDelta(); t=clock.elapsedTime;
// controls smoothing
rotY += (tRotY-rotY)*k ... apply earthGroup.rotation.set(rotX,rotY,0) order YXZ? set rotation.x=rotX, rotation.y=rotY with order 'YXZ'.
camera.position.setLength(dist smoothed); lookAt(0,0,0)
auto-rotate: if enabled && idle>3s: tRotY += 0.04*dt*60? use rad/s 0.045.
sun drift.
uniform updates: earth camPos, atmo camPos, sunDir shared (use same Vector3 instance across uniforms → single update).
sharedTime.value=t;
city updates (size lerp, color lerp, sprite opacity, ring anim) — per frame ~33 fine.
route hover lerp; dot positions.
stars rotate slowly.
raycast hover if pointer moved since last frame (flag).
popup reposition.
HUD update throttled (every 250ms).
render.
```

City size target recompute triggers: metric change, integer year change, continuous yearFloat during play? During play sizes should breathe with values — recompute targets each frame during play? Recompute norm each frame for 33 cities is trivial (value computations 33*3). Do each frame during play; otherwise on change. Lerp handles smoothness.

Heat & ranking & total: only on integer year change/metric change (and initial).

Total = sum of city values for metric at yearFloat (display at integer year for stability — use current float but update on integer ticks; also tween number).

Number tween: simple rAF-based tween function for totalVal.

Popup live update during play? If popup open, update its numbers on integer year change. Fine.

Cursor lat/lon: raycast to earth mesh; use earthGroup.worldToLocal(hit.point.clone()).

FPS: count frames per second.

Initial camera distance 3.35, min 1.5, max 7.

Wheel zoom: e.preventDefault (passive:false).

Pinch: pointer events cache; if two: distance delta → targetDist.

Also keyboard? Skip.

View reset button (⌂): reset targetRotY/X and dist with tween.

Now fallback procedural textures: function makeFallbackDay(): canvas 1024x512: fill ocean vertical gradient #0a2a52→#0d3b66 with noise speckle + faint lat/lon lines; night: black with random dots; water: white all over. Use only if loading error. Implement minimally.

LoadingManager: onError → fallback flag for that texture. Since multiple textures, track individually: onLoad for each sets value; manager.onError logs; after manager.onLoad finalize: if dayMap missing → fallback. Simplify: load via promises with TextureLoader.loadAsync, Promise.allSettled; fallback per texture. Progress: manual counter update loader UI (allSettled doesn't stream; use loadAsync with onProgress? loadAsync doesn't expose progress. Use loader.load(url, resolve, undefined, reject) with manual promise and progress counter.) Write small helper.

Texture URLs (three-globe via jsdelivr, pinned 2.31.1):
Base = 'https://cdn.jsdelivr.net/npm/three-globe@2.31.1/example/img/'
- earth-blue-marble.jpg (day) — 4096x2048 ~1.2MB? Actually ~700KB. OK.
- earth-night.jpg
- earth-topology.png (bump — skip bump? adds realism via shader bump? skip; use water mask instead)
- earth-water.png (water mask) — does it exist? three-globe's example/img has: earth-blue-marble.jpg, earth-dark.jpg, earth-day.jpg, earth-night.jpg, earth-topology.png, earth-water.png, night-sky.png. Yes earth-water.png exists (used for reflections in their example).
Set anisotropy from renderer.capabilities.getMaxAnisotropy().

FlipY default true works with equirectangular and SphereGeometry's uv — three-globe textures work with standard sphere. Yes.

world-atlas: 'https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json'.

topojson importmap entry: "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm". Dynamic import inside try/catch.

Note: dynamic import of URL via importmap mapping works.

Now popup sparkline: build svg string.

Ranking list item: rank number, name, bar, value. Bar width % relative to top. transition width .6s.

Toggles: cities, routes, heat, atmosphere, stars? auto-rotate. Implement layer visibility: cityGroup.visible etc. Heat toggle sets heatMesh.visible. Atmosphere mesh. Auto-rotate flag.

Also "city lights" is inherent.

Edge: heat overlay + popup z-order: canvas z-index 0, UI above.

Pointer events: canvas receives; UI elements pointer-events auto; container overlay pointer-events none.

Cursor: grab / grabbing / pointer on hover city.

Also add subtle vignette overlay div (radial gradient) pointer-events none — adds depth. And faint scanlines? skip scanlines; vignette yes.

Title area small "data source is simulated" note for honesty: "演示数据 · SIMULATED DATASET". Good practice.

Now let me write it all out. Must be careful and complete. Let me draft the key JS functions precisely.

latLonToVec3:
```js
const R = 1;
function latLonToVec3(lat, lon, r){
  const phi = (90 - lat) * Math.PI/180;
  const theta = (lon + 180) * Math.PI/180;
  return new THREE.Vector3(
    -r * Math.sin(phi) * Math.cos(theta),
     r * Math.cos(phi),
     r * Math.sin(phi) * Math.sin(theta)
  );
}
```
Verify alignment with texture: three-globe uses exactly this mapping with standard equirectangular textures. Good.

Inverse for coords display:
```js
lat = 90 - Math.acos(p.y / len) * 180/Math.PI;
lon = Math.atan2(p.z, -p.x) * 180/Math.PI - 180; normalize: while(lon<-180) lon+=360;
```
atan2 returns [-180,180]; subtract 180 → [-360,0]; add 360 if <-180 → [0? ...] let me compute: lon0 = atan2deg ∈ [-180,180]; lon = lon0 - 180 ∈ [-360, 0]; if lon < -180 → lon += 360 ∈ (180,360]→ wait result range: lon0=-180→lon=-360→+360=0? hmm lon0=180→0-? let me redo: lon0=180 → lon=0? lon0-180=0. lon0=-180 → -360 → +360 → 0. Both ends 0. lon0=0 → -180. So maps to [-180,180]. Check forward: lon=105 → theta=285°, x=-sinφ cos285 = -sinφ*0.2588, z=sinφ*sin285=-sinφ*0.9659. Inverse: atan2(z,-x)=atan2(-0.9659s, 0.2588s)= atan2(-0.9659,0.2588)= -75°. lon=-75-180=-255→+360=105 ✓.

Great-circle distance km:
```js
function distKm(a,b){ const d = Math.sin(a.lat*D)*Math.sin(b.lat*D)+Math.cos(a.lat*D)*Math.cos(b.lat*D)*Math.cos((a.lon-b.lon)*D); return Math.acos(Math.min(1,Math.max(-1,d)))*6371; }
```

Value function:
```js
const YEARS=[2019,2020,2021,2022,2023,2024];
const GLOBAL={ gdp:[1,.965,1.035,1.11,1.19,1.28], pop:[1,1.006,1.013,1.02,1.027,1.035], flt:[1,.52,.66,.84,.98,1.12] };
function metricValue(c,m,yf){
  const i=Math.min(4,Math.floor(yf-2019)), f=THREE.MathUtils.clamp(yf-2019-i,0,1);
  const g=GLOBAL[m][i]*(1-f)+GLOBAL[m][i+1]*f;
  const base = c[m]; // gdp/pop/flt base
  const gr = m==='pop'? c.g*0.35 : c.g;
  return base*g*Math.pow(1+gr, yf-2019);
}
```
yf clamp [2019,2024].

Normalization for size: compute min/max across cities for metric at yf.

Country aggregate:
```js
function countryValues(m,yf){ const map=new Map(); CITIES.forEach(c=>{ map.set(c.cc,(map.get(c.cc)||0)+metricValue(c,m,yf)); }); return map; }
```

Ramp:
```js
const RAMPS={ gdp:['#0e2f52','#1b5f8a','#2fa08c','#9fc455','#ffd44d'], pop:[...], flt:[...] };
function rampColor(m,n){ stops; seg=(stops-1)*n; i=floor; lerp rgb }
```
hex→rgb helper.

Heat drawing with Path2D:
```js
function geoToPath(geom){ const p=new Path2D(); function ring(r){ r.forEach(([lon,lat],i)=>{ const x=(lon+180)/360*W, y=(90-lat)/180*H; i?p.lineTo(x,y):p.moveTo(x,y); }); p.closePath(); } if(geom.type==='Polygon') geom.coordinates.forEach(ring); else if(geom.type==='MultiPolygon') geom.coordinates.forEach(poly=>poly.forEach(ring)); return p; }
```
Holes handled by evenodd fill.

Legend gradient CSS: `linear-gradient(90deg, stops at %...)` built from ramp.

Units: gdp "billion USD", pop "millions", flt "flights/day". Metric labels: GDP, Population, Flight Volume.

Total formatting: gdp: sum ~maybe 20,000 billion → display "2.31 trillion USD"? Keep billion with toLocaleString: "23,400 billion USD" fine. pop sum ~450 million people. flt sum ~25,000 flights/day.

Popup rows show all 3 metrics with units; active highlighted.

Flight tooltip: `${airline} ${flightNo} · ${from.cn} ⇌ ${to.cn}`, rows: aircraft, weekly flights, distance, "on-time rate"? Add pseudo on-time rate deterministically from hash: 78-95%. Fun. Also add "weekly N flights".

Route data: define with city ids (use index into CITIES). Let me assign ids as short codes and map.

City list with codes:
```
PEK Beijing, SHA Shanghai (use "SH"?), CAN Guangzhou, SZX Shenzhen, CTU Chengdu, HKG Hong Kong, TYO Tokyo, SEL Seoul, SIN Singapore, BKK Bangkok, DEL Delhi, BOM Mumbai, DXB Dubai, IST Istanbul, MOW Moscow, LON London, PAR Paris, FRA Frankfurt, AMS Amsterdam, MAD Madrid, NYC New York, LAX Los Angeles, CHI Chicago, ATL Atlanta, DFW Dallas, MEX Mexico City, SAO São Paulo, BUE Buenos Aires, SYD Sydney, JNB Johannesburg, CAI Cairo, LOS Lagos, YYZ Toronto, SFO San Francisco
```
34 cities.

Route list (from, to, airline code+name, flight number, weekly, aircraft):
1 PEK-LAX CA Air China CA983 14 B777-300ER
2 PEK-LON CA CA855 11 B777-300ER? fine 747? keep
3 SHA-SFO MU China Eastern MU587 14 A350-900? MU587 is 77W. B777-300ER
4 SHA-TYO MU MU523 21 A330? use B787-9
5 CAN-SYD CZ China Southern CZ325 7 A330-300
6 SZX-SIN CZ CZ8057 14 B787-9
7 CTU-FRA CA CA431 7 A350-900? CA uses A350 yes
8 HKG-LON CX Cathay CX251 14 A350-1000
9 HKG-SYD CX CX100 10 A350-900? CX to SYD is A330/A350 fine
10 TYO-LAX NH ANA NH106 14 B777-300ER
11 TYO-SIN NH NH841 14 B787-9? NH SIN is 787/A380? fine
12 SEL-LAX KE Korean Air KE17 11 B747-8i? KE17 is A380. A380-800
13 SEL-DXB? KE951 7 B777-300ER
14 SIN-LON SQ Singapore Airlines SQ322 14 A380-800
15 SIN-SYD SQ SQ231 21 A350-900? SQ SYD is A380/350? fine
16 SIN-DXB EK? use SQ EK349? route SIN-DXB airline EK Emirates EK349 21 A380-800
17 BKK-TYO TG Thai TG676 14 B787-9? TG is A350? use A350-900
18 DEL-DXB AI Air India AI131? actually EK operates heavily; keep AI131 B787-9 18? weekly 18 fine
19 BOM-LON AI AI111 10 B777-300ER? AI111 is 77W yes
20 DXB-LON EK EK001 28 A380-800
21 DXB-PAR EK EK073 14 B777-300ER
22 DXB-SAO EK EK261 7 B777-300ER
23 IST-NYC TK Turkish TK001? TK1? use TK1 14 B777-300ER? TK JFK is 77W/A350? fine
24 IST-CAI TK TK694 11 A321neo
25 LON-NYC BA British Airways BA178? use BA215? whatever BA117? fine BA178 28 B777-300ER? BA JFK: 77W/A380? fine
26 PAR-NYC AF Air France AF006 21 A350-900? AF006 is 77W. B777-300ER
27 FRA-CHI LH Lufthansa LH432 14 A340? use B747-8? LH ORD is 748? use B747-8i? LH uses A330/748. "A330-300"? pick B787-9? LH has 789 now. fine
28 AMS-ATL DL Delta DL47? actually KL; use DL DL47 11 A330-300? DL AMS-ATL is 767? use B767-400? keep A330-300? DL has A339. "A330-900"
29 MAD-MEX IB Iberia IB6400 10 A350-900
30 NYC-LAX AA American AA1? use AA100? "AA 1"? historically AA1 is JFK-LAX? AA flight 1? use AA100? whatever: AA100 35 B737? transcon A321T. "A321neo"
31 NYC-SAO AA AA950? use AA973? whatever AA950 10 B777-200? use B787-9
32 CHI-TYO NH NH111 7 B787-8? NH111 ORD-NRT 788 yes
33 ATL-PAR DL DL285? AF? use DL DL82 11 A330-900? fine
34 DFW-LON AA AA50 7 B777-200? AA LON-DFW 772 yes
35 MEX-LAX AM Aeroméxico AM570? actually AM570? whatever 17 B787-9? AM has 788. B787-8
36 SAO-BUE AR? LATAM LA? use LA8067? Aerolíneas Argentinas AR1130? hmm AR1270? whatever AR1130 21 B737-800? GRU-EZE widebody? A330-200? use "A330-200"? keep B737-800 fine
37 SYD-LAX QF Qantas QF11 10 B787-9
38 JNB-DXB EK EK762 11 A380-800? EK762 is A380 yes? fine
39 CAI-MOW SU Aeroflot SU401 7 A320neo? fine
40 LOS-LON BA BA075? BA75 is LOS-LHR yes 7 B787-9? BA75 is 789? fine
41 YYZ-LON AC Air Canada AC848? AC848? whatever 10 B787-9? AC has 789 yes? AC848 might be 77W. B777-300ER
42 MOW-PEK SU? CA? use SU200? SU200? whatever "SU 204"? keep CA? pick SU SU204 7 B777-300ER? Aeroflot has 77W yes
43 SFO-TYO UA United UA837? UA SFO-NRT is UA837? whatever 11 B787-9
44 SFO-SYD UA UA863? actually UA863 is AKL? whatever 7 B787-9
45 HKG-TYO CX CX520? whatever 14 A330-300
46 BKK-SIN TG? SQ? use SQ SQ7069? hmm use TG TG403 21 A350-900
47 DEL-SIN SQ? AI? use SQ SQ405? SQ DEL? yes SQ405? whatever 10 B787-9? fine
48 CAI-DXB EK EK924? whatever 14 B777-300ER
49 IST-FRA TK TK1581? whatever 17 A321neo
50 PAR-CAI AF AF570? whatever 10 B777-300ER? A350? fine

That's ~50; maybe trim to ~44 for perf/visual clarity. Keep ~46. Fine.

Weekly max 35 (NYC-LAX). Dot speed and brightness scale.

Route color: base cyan #58c9ff, lerp toward #8fe3ff by distance norm? Slight variance: hue shift by hash: pick from palette [#4fd2ff,#63b0ff,#7de3d0,#a5b8ff]? Keep tasteful blue-cyan-teal set.

Hover highlight color whiter.

Now popup positioning: prefer right of point; clamp within viewport minus panel width? Panel is on right — popup may overlap panel; acceptable, or clamp to left of panel: right margin = panelWidth+16 when panel visible (desktop). Compute simply: clamp x to [12, innerWidth - w - 12], y to [12, innerHeight - h - 12]; if overlaps panel area and x+w > innerWidth-360 → place on left of point. Keep simple clamp.

Behind-globe check: cityWorld = pos.clone().applyMatrix4(earthGroup.matrixWorld); visible if cityWorld.dot(camera.position) > 0 (both roughly from origin): normalize: dot(norm(cityWorld), norm(camPos)) > 0.12. If not visible → hide popup (opacity 0 / display none) but keep open state.

Tooltip similar: hide during drag.

Auto-rotate resume: lastInteract timestamp.

Also pause auto-rotate when popup open? Nice: pause spin while popup open to allow reading. Resume on close. Do it.

Reduced motion: skip.

Now loader progress: count textures (4) + geo (1). Update %. On all settled → fade out loader (add .done → opacity 0 → remove after 600ms). Start render immediately with fallback? Simplest: wait for allSettled then init scene with what we have, then fade loader. But scene init doesn't depend on textures except earth material — build material with placeholder color and assign textures as they arrive; start render loop immediately, loader overlay fades when all settled. Nice: globe appears as textures pop in. But earth shader needs samplers bound; give fallback canvas texture immediately, replace on load. Do: create small placeholder canvas texture (dark blue) initially; on each load, assign + needsUpdate.

Manager approach: manual.

Write loadTexture(url, isColor) returning promise with progress callback.

Now stars:
```js
function makeStars(count, rMin, rMax, size, opacity){ random positions on spherical shell; colors: mostly white with some blue/warm; Points }
```
Twinkle: shader overkill; animate material.opacity = base*(0.9+0.1 sin t*0.7) and rotate slowly.

Also add faint milky-way band? skip.

Vignette overlay CSS radial-gradient — subtle.

Also "corner ticks" decoration on viewport (HUD frame): 4 corner L marks fixed — adds crafted feel.

Now sizes: panel width 300px right side; timeline bottom center width min(720px, 90vw); brand top-left; legend bottom-right above timeline? Timeline is bottom center full-width-ish; legend bottom-right may clash with panel? Panel right side vertical; legend bottom-right below panel — panel height might reach bottom. Panel max-height with inner scroll (thin scrollbar styled). Legend bottom-left above HUD? HUD bottom-left has coords. Move legend to bottom-left above HUD block. Or attach legend as part of panel section (heat legend inside panel under layers). Cleaner: put legend inside panel as its own section that updates. But requirement visibility… panel section is fine and tidy. Also floating legend bottom-left above coords HUD. Actually simplest and robust: legend inside panel section "区域热力" with gradient bar + min/max. Do that. Remove separate floating legend.

HUD bottom-left: coords + zoom + fps + "drag to rotate · wheel to zoom" hint that fades after 6s.

Hint toast center-bottom initially: "拖拽旋转 · 滚轮缩放 · 点击城市查看详情" auto-fade. Nice onboarding.

Now ranking rows: `1 Shanghai 2,340` with bar. Click on ranking row → focus that city (rotate to it + open popup)? Nice interaction: on click, tween targetRotY/X to bring city to front and open popup. Compute required rotY: want city's world direction ≈ camera direction (mostly +Z with slight y). City local direction d (unit). After rotY α and rotX β… solving exactly with both rotations is fiddly; approximate: set targetRotY so city's longitude faces camera: want θ_total ≈ 90° (facing +Z). θ_local=(lon+180)°. rotY = deg(90 - (lon+180)) → = -90 - lon degrees. Then rotX target = -lat? Rotating group on X by β moves city's latitude: want city at y ≈ camera elevation ~small; set targetRotX = lat*0.6? Let me think: rotation order YXZ: apply rotY then rotX (intrinsic). After rotY, city is at longitude facing front with latitude φ (y=sinφ). Then rotX β rotates around X: y' = y cosβ - z sinβ; with z≈1 (front): to bring y'→~0.2: β ≈ φ-ish. Set targetRotX = degToRad(lat)*0.9 clamp. Sign check: rotation.x positive rotates +z toward -y? Rotation around X by β: y'=y cosβ - z sinβ, z'=y sinβ + z cosβ. For northern city y>0, want reduce y → need sinβ>0 → β>0. So targetRotX = +latRad*0.85 clamp [-1,1]… but wait our drag maps vertical drag to rotX with same convention; consistent. Also shortest-path rotY: normalize delta to [-π,π]. Implement focusCity(c).

Nice bonus: ranking click + double-click on marker to focus. Keep ranking click.

View reset button in panel header area (icon ⌖ "reset view").

OK, popup should also show mini "related routes" count: number of routes touching city. Sure, one line: "Associated routes: N".

Time to write final code. Also include <noscript> and WebGL check.

Careful with shader compilation pitfalls: declare precision? Three auto-adds. Multiple sampler declarations `uniform sampler2D dayMap, nightMap, waterMap;` valid GLSL? Yes comma-separated uniform declarations are valid.

Also for route shader, need `attribute float aT;` — in WebGL2 three converts automatically. Use ShaderMaterial (not RawShaderMaterial) so three injects attribute/varying compat. Fine.

Points moving dot: update positions each frame: geometry.attributes.position.needsUpdate.

Popup sparkline: width 240 height 56.

Number format helper: fmt(n): n>=10000 → (n/10000).toFixed(2)+' trillion'? For GDP sum ~2.3 trillion. Provide fmtVal(m,v): gdp: v>=10000? (v/10000).toFixed(2)+' trillion USD' : Math.round(v).toLocaleString()+' billion USD'; pop: v.toFixed(1)+' million'; flt: Math.round(v).toLocaleString()+' flights/day'.

Ranking values use fmt short.

Total tween: animateNumber(el, from, to, fmtFn) with rAF 500ms.

Let me write CSS carefully. Use system font stack for Chinese: 'Noto Sans SC', 'PingFang SC','Microsoft YaHei'. Display: 'Rajdhani' for numbers + Latin with fallback to Noto.

Panel sections separated by dashed dividers. h3 style: 12px letter-spacing, em English small muted right-aligned.

Segmented buttons: grid 3 cols; each button with colored dot top? Design: vertical mini-cards: value icon glyph (¥/👥/✈ — use text symbols: "$" "人" "✈"? use simple unicode: GDP "＄"? Use labels: GDP / 人口 / 航班. Active: border accent + bg tint + glow.

Toggle list: label with switch: input hidden, span slider 28x16.

Ranking: ol with counter.

Timeline: bar with backdrop blur, border top accent; content flex: play button (44px circular border), year display, slider stretch, speed group.

Ticks: absolute-positioned labels under slider at 0,20,…100%.

Play icon SVG toggle.

Now hint toast + loader.

Also add `#fallbackNote` if texture fallback used: small toast "Textures load failed, using placeholder textures".

Edge: wheel on panel should scroll panel not zoom — wheel listener only on canvas. Panel has its own scroll.

Pointer capture on drag.

Also prevent context menu? right-drag pan not implemented; disable contextmenu on canvas to avoid interruption.

DPR clamp 2. Antialias true.

renderer.setClearColor(0x04070f).

Also add slight camera parallax on mouse? camera fixed; skip.

Now let me write it all out. Must double-check every DOM id referenced in JS exists.

List of ids:
- loader, loadBar? loadPct, loadTip
- brand (static)
- hud: hudCoords, hudZoom, hudFps
- hint (toast)
- panel: metric buttons (.seg button[data-m]), totalVal, totalUnit, toggles: tglCities, tglRoutes, tglHeat, tglAtmo, tglSpin; resetView button; rankList; legend: lgBar, lgMin, lgMax, lgTitle? (metric name span lgMetric)
- timeline: playBtn, playIco? yearBig, yearRange, speed buttons (.speed button[data-s])
- popup (with inner content injected)
- tip
- stage (canvas container)

JS globals: state = { metric:'gdp', yearFloat:2019, playing:false, speed:1, layers..., hoverCity, hoverRoute, selCity, lastInteract }

Initial: metric gdp, yearFloat 2019, playing true? auto-play timeline on load gives immediate sense of motion — yes playing=true default (with earth spin too). User can pause.

Auto-rotate default on.

Now heat rebuild also on metric/year change; also when geo loads.

One more: route visibility when metric != flt? Routes always shown (they're flight routes) but brightness could scale slightly with metric? Keep constant; fine.

Route flow speed: uSpeed uniform ~0.10-0.22 (cycles/sec) based on weekly.

Route shader details:
```
uniform vec3 uColor; uniform float uTime,uSpeed,uOpacity,uHover;
varying float vT;
void main(){
  float x = fract(vT - uTime*uSpeed);
  float pulse = smoothstep(0.22, 0.0, x);       // head at x=0, tail behind
```
Wait direction: as uTime increases, pattern moves in +vT direction (since fract(vT - t): fixed phase vT = t + c moves toward increasing vT). Pulse head: brightest where x is just above 0? smoothstep(0.22,0.0,x) is 1 at x=0 → 0 at 0.22: so bright at head (x=0) fading back over 0.22 behind (x is [0,0.22] which corresponds to vT slightly greater than moving origin? hmm x = vT - t (mod 1). Points with small positive x are "ahead" of wavefront… Wavefront at vT = t (x=0). Points with x ∈ (0,0.22] have vT ∈ (t, t+0.22] → ahead of front. We want tail behind front (points already passed: vT < t → x near 1). So use tail: glow = smoothstep(0.78,1.0,x) → 0 at 0.78 → 1 at 1 (just behind front). Combine: glow = smoothstep(0.75,0.98,x); plus small head sparkle: glow += smoothstep(0.03,0.0,x)*0.9? At x→0+ ahead is small. Let me define:
```
float tail = smoothstep(0.62, 1.0, x);      // 0→1 approaching front from behind
float head = smoothstep(0.05, 0.0, x);      // right at/ahead of front
float glow = tail*tail*1.1 + head*0.9;
```
Base line alpha 0.22. Color = uColor*(0.35 + glow*1.8 + uHover*0.8). Alpha = uOpacity*(0.28 + glow*0.9 + uHover*0.4). Additive.

Also second slower shimmer? enough.

Dot movement direction matches A→B.

Now marker color per metric; also heat ramp per metric; legend gradient per metric; UI accent per metric. Cohesive.

City color transition: lerp material.color toward target each frame (k=0.08). Size lerp too.

Ring color set immediately (material.color.set).

Sprite color same as metric.

Popup HTML sample:

```html
<div class="pp-head">
  <div class="pp-name">上海 <span>SHANGHAI</span></div>
  <button class="pp-close">×</button>
</div>
<div class="pp-sub">中国 · Rank #2 · Associated routes 3</div>
<div class="pp-grid">
  <div class="pp-row [active]"><label>GDP</label><b>7,812</b><u>billion USD</u></div>
  ...
</div>
<div class="pp-spark"><svg>...</svg><div class="pp-spark-lab">GDP trend 2019–2024</div></div>
```

Tooltip HTML:
City: `<b>东京</b><span>TOKYO · 日本</span>` + small metric line.
Route: header + rows.

Now also compute "on-time rate" pseudo: (78 + (hash%17))%.

Hash: sum of char codes.

OK — also ensure popup doesn't block canvas drag: popup pointer-events auto (need close button); drag starting on popup shouldn't rotate — canvas listener only on canvas element, popup is separate element → fine.

Hover raycast should ignore when pointer is over UI: pointermove listener on canvas only, so no updates over UI; also hide tooltip when leaving canvas (pointerleave).

Now let me write final HTML. Also include meta viewport, lang zh-CN, title "寰宇脉搏 · 3D Data Globe".

Let me write JS in one module with clear sections. Carefully bracket-match.

Also note: `import * as THREE from 'three'`.

Pin three@0.160.0.

importmap:
```json
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
    "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm"
  }
}
```
We might not use addons; keep mapping anyway, harmless.

Actually no OrbitControls (custom), so addons unused — keep anyway.

Potential pitfall: `+esm` build of topojson-client — jsdelivr's +esm usually works for it (ESM conversion). If fails → catch → fallback blobs. Good.

world-atlas fetch: also try unpkg fallback:
```js
const SOURCES=['https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json','https://unpkg.com/world-atlas@2.0.2/countries-110m.json'];
```
Try sequentially.

Texture sources also with fallback host: jsdelivr → unpkg.
```js
function texURLs(p){ return [`https://cdn.jsdelivr.net/npm/three-globe@2.31.1/example/img/${p}`, `https://unpkg.com/three-globe@2.31.1/example/img/${p}`]; }
```
Loader tries in order.

three-globe version: does 2.31.1 exist? versions: 2.31.0, 2.31.1? I recall 2.31.0 and 2.31.1 exist (early 2024). Safer: use @2.31.0? or no version → @latest risky for path stability but img path has been stable for years. Pin 2.31.0 — I'm fairly sure 2.31.0 exists. Hmm risk if version doesn't exist → 404 → then unpkg also 404 → fallback texture (ugly). To be safer: use "three-globe@2" version range: jsdelivr supports ranges: https://cdn.jsdelivr.net/npm/three-globe@2/example/img/earth-blue-marble.jpg — resolves to latest 2.x. unpkg also supports @2. Use @2 range — robust.

Similarly world-atlas@2 range OK. three pinned exactly fine.

Also add earth-day? blue-marble is enough.

Night: earth-night.jpg. Water: earth-water.png. Also load topology for bump? skip.

Now let me write code. Also don't forget `crossOrigin` — TextureLoader default crossOrigin is 'anonymous'? THREE.TextureLoader sets crossOrigin = 'anonymous' by default. Yes.

Color space: tex.colorSpace = THREE.SRGBColorSpace for day/night. Water mask linear (default NoColorSpace) fine.

MagFilter default linear, minFilter default mipmap — need POT? WebGL2 supports NPOT mipmaps. 4096x2048 fine.

Anisotropy set.

Now marker count and draw calls fine.

Let me write animate loop with clock.

Also handle visibilitychange to avoid huge dt: clamp dt ≤ 0.05.

Write updateTimelineUI on yearFloat change: slider value, big year, progress gradient.

Play wrap: if yearFloat>=2024 → yearFloat=2019 (loop) and rebuild heat.

Integer year detection: const iy=Math.floor(yearFloat+1e-6); if iy!==lastIY → onYearTick.

onYearTick: rebuildHeat, updateRanking, updateTotal, update popup if open.

During play, sizes updated each frame (cheap).

Also totalVal continuous? Only on tick + metric change. Fine.

Also add subtle "data pulse": when year ticks, flash year display (CSS class pop animation).

Now popup sparkline uses active metric; when metric changes while popup open → re-render popup.

focusCity rotation: also open popup after rotation? Open immediately; popup hidden while city behind; as globe rotates to front, popup appears. Nice.

Shortest angle:
```js
function normAngle(a){ while(a>Math.PI)a-=2*Math.PI; while(a<-Math.PI)a+=2*Math.PI; return a; }
targetRotY += normAngle(desired - targetRotY);
```
desired = THREE.MathUtils.degToRad(-90 - c.lon). Verify: want city at world direction +Z. Local direction θ=(lon+180). After rotY α, azimuth θ+α should equal 90° (+Z corresponds to θ=90 since z=sinθ). α = 90 - (lon+180) = -90 - lon degrees. ✓.
targetRotX = clamp(degToRad(c.lat)*0.85, -1.05, 1.05)? Sign: positive rotX moves front point downward? Earlier: y' = y cosβ - z sinβ; with z≈1, β>0 reduces y → northern city (y>0) needs β>0 to center → targetRotX = +latRad*k. But wait drag convention: dragging down should rotate globe so we see north? Standard: drag up → globe tilts showing north pole? Let me define drag: dy>0 (drag down) → rotX -= dy*s? Let me pick: rotX target change = -dy*0.004 (drag up (dy<0) → rotX increases → northern hemisphere tilts toward viewer? with β>0, front point moves down on screen, so we see more of north — drag up typically rotates globe so front moves up, showing southern regions… honestly either feels OK; pick rotX += dy*0.0045? Let me simulate: user drags downward (dy>0), expects globe's front surface to move downward, revealing top (north). Front moving down = y decreases = β>0 increase. So rotX += dy*k. OK: targetRotX = clamp(targetRotX + dy*0.0045, -1.1, 1.1). And horizontal: drag right (dx>0) → surface moves right → globe rotates so new stuff comes from left → rotY += dx*k? Front moving right = rotation around Y… +Y rotation moves +Z toward -X? Rotation around Y by α: x'=x cosα + z sinα, z'=-x sinα + z cosα. Point at +Z (0,0,1): x'=sinα → α>0 moves front toward +X (right on screen? camera looks down -Z at origin from +Z; screen right = +X). Yes α>0 moves front to right. Drag right → front right → rotY += dx*k. ✓.

Auto-rotate: tRotY += dt*0.05 (slow, ~2 min/rev… 0.05 rad/s → 126s/rev good).

Camera elevation fixed direction (0,0.32,1) normalized * dist.

Also earthGroup initial rotation.x? Start rotX 0.18 slightly? Set initial targetRotX=0.15, rotX same. And initial rotY to face Asia: lon 110 → α=-90-110=-200° ≡ 160°. targetRotY=degToRad(160).

Sun initial angle: sunDir from azimuth ~ degToRad(35)? Pick so terminator visible over Pacific initially: sun at azimuth such that Asia is day-ish, Americas dusk. sunDir = normalize(cos(az)*?, y, sin(az)). Set az = degToRad(200)? whatever — pick az so dot with Asia direction (+Z-ish initially) is positive ~0.5: sunDir=(0.55,0.22,0.8) normalized. Terminator crosses Africa/Americas. Fine; drifts anyway.

City night glow: nightF = smoothstep from sunDot: in JS: n=world pos normalized; s=n.dot(sunDir); nightF = clamp((0.1 - s)/0.3, 0, 1)? s<0.1→night. map: nightF = THREE.MathUtils.clamp((0.12 - s)/0.35, 0, 1).

Sprite opacity = (0.35 + 0.65*nightF) * pulse.

Core material: MeshBasicMaterial — not affected by sun (always bright) — that's fine for data markers; maybe dim slightly on day? Keep bright.

Heat overlay always full opacity — fine.

Now ranking bar color = accent.

Total unit text per metric.

Also add panel collapse button for small screens? Add media query: on <900px panel becomes narrower/hidden behind toggle? Provide simple: on <860px, panel becomes bottom sheet? Keep simple: panel width 260px, allow page scroll? Canvas fixed. Add toggle button (⚙) top-right to show/hide panel on small screens. Implement #panelToggle fixed top-right; panel transform translateX. Default visible on desktop; on mobile default hidden (matchMedia check). Small code.

OK also FPS meter: frame counter update each second.

Now let me write everything. Also include meta viewport, lang zh-CN, title "寰宇脉搏 · 3D Data Globe".

Careful with GLSL string quoting (template literals) inside module.

Also ensure no stray Chinese quotes in code strings breaking (use them only in text content, fine).

Popup route count: ROUTES.filter(r=>r.a===c||r.b===c).length.

City data structure final:
```js
const CITIES=[
 {id:'PEK',cn:'北京',en:'BEIJING',cc:'156',country:'中国',lat:40.0,lon:116.6,gdp:680,pop:21.9,flt:1080,g:0.035},
 ...
];
```
Use airport-ish lat/lon but city center fine; use approximate city coords.

Let me finalize numbers (gdp 2019 base billion RMB? no—billion USD approximate metro GDP, plausible):

Beijing 680, Shanghai 720, Guangzhou 430, Shenzhen 500, Chengdu 330, Hong Kong 370, Tokyo 1050, Seoul 830, Singapore 400, Bangkok 250, Delhi 300, Mumbai 310, Dubai 115, Istanbul 240, Moscow 260, London 860, Paris 760, Frankfurt 410, Amsterdam 230, Madrid 240, New York 1800, Los Angeles 1100, Chicago 780, Atlanta 470, Dallas 640, Mexico City 420, São Paulo 430, Buenos Aires 200, Sydney 400, Johannesburg 170, Cairo 190, Lagos 90, Toronto 620, San Francisco 1050.

pop (millions metro): 21.9,24.9,18.7,17.6,21.2,7.5,37.4,26.0,5.9,10.7,32.9,20.9,3.6,15.6,12.6,9.5,11.0,7.6,2.5? Amsterdam metro ~2.4 use 2.4? make 2.6? fine 2.6? actually Amsterdam metro 2.4; use 2.4. Madrid 6.7, NYC 20.1, LA 13.2, CHI 9.5, ATL 6.1, DFW 7.9, MEX 21.8, SAO 22.4, BUE 15.4, SYD 5.4, JNB 6.3, CAI 21.8? Cairo ~21.3 use 21.3, LOS 15.9, YYZ 6.3, SFO 4.9.

flt (daily departures approx): PEK 1050, SHA 1000, CAN 780, SZX 620, CTU 640, HKG 600, TYO 950 (HND+NRT), SEL 850, SIN 820, BKK 760, DEL 560, BOM 500, DXB 900, IST 820, MOW 480, LON 1250, PAR 1100, FRA 780, AMS 690, MAD 600, NYC 1350, LAX 880, CHI 930, ATL 1300, DFW 1000, MEX 620, SAO 560, BUE 330, SYD 480, JNB 330, CAI 400, LOS 260, YYZ 590, SFO 760.

g growth: Chinese cities 0.045 (Chengdu/Shenzhen higher .05), India .055, Singapore .03, Tokyo .01, Seoul .015, Bangkok .03, Dubai .035, Istanbul .03, Moscow .01 (with dip? keep), European .012-.018, US .02, Mexico City .02, SAO .015, BUE .01 (inflation weird, fine), SYD .025, JNB .015, CAI .04, LOS .04, YYZ .02, SFO .025.

Fine — plausible enough for demo (labeled simulated).

Routes final (~46) as listed above with ids.

Let me write ROUTES entries: {a:'PEK',b:'LAX',al:'中国国际航空',no:'CA983',wk:14,ac:'B777-300ER'} etc. Compose list from above (46 entries). Ensure all ids exist in CITIES.

Route ids used: PEK LAX LON SHA SFO TYO CAN SYD SZX SIN CTU FRA HKG SEL DXB BKK DEL BOM IST NYC PAR CHI ATL AMS MAD MEX SAO BUE JNB CAI MOW LOS YYZ AC? no. SFO. All present ✓.

Route color palette: ['#59d1ff','#6db4ff','#7fe6d8','#9db1ff'] pick by index hash. Hmm #9db1ff is periwinkle; ok as minor.

Dot color = route color.

Also route width illusion: can't vary line width; instead brighter for higher frequency.

Hover detection on lines: raycaster.intersectObjects(routeLines) — Line raycast needs threshold param; set raycaster.params.Line.threshold=0.02. Intersections may include hidden back segments? Line raycast ignores depth (pure geometry) — may pick route on back side! Check: intersection point distance vs earth occlusion: test if segment midpoint occluded… simpler: for picked route, compute closest point on line to camera ray… occlusion test: for intersection point p, if ray from camera to p passes through sphere of radius 1 → occluded: distance from origin to ray < 1 and intersection is beyond tangent point… Compute: occluded = raycaster.ray.intersectsSphere(unit sphere at origin) AND intersection distance > distance to sphere hit. Use ray.intersectSphere(origin,1) → if hit and sphereHitDist < pDist - epsilon → occluded → skip. Implement filter: pick first non-occluded intersection.

Same for city raycast: spheres on surface; back cities occluded by earth mesh? Raycast against city meshes returns front and back hits (ray passes through earth but earth not in test). Sort by distance; nearest hit is front-most — but a back city directly behind a front one? nearest is front — but a city on far side: ray reaches it only if it passes through earth — geometrically ray hits far sphere only if it passed through sphere region; check occlusion same sphere test. Apply same filter to city hits.

Implement pickAt(pointer): set raycaster; hits = intersectObjects(cityCores); filter occluded; if any → city. Else routeHits similarly filtered.

Occlusion helper:
```js
const _sph=new THREE.Sphere(new THREE.Vector3(0,0,0), 1.0);
function occluded(point){ const d = raycaster.ray.distanceToPoint(point); ... }
```
Hmm simpler: distance from ray to origin < 1 and point is beyond tangent point… Compute: occluded = raycaster.ray.intersectsSphere(unit sphere at origin) AND intersection is beyond sphere hit. Use ray.intersectSphere(sphere, target) → returns point (first hit, entry) or null; if entry exists and entryDist < pointDist - 0.02 → occluded. ✓ (earth radius 1; markers at 1.004 so front markers: entry hit distance vs marker distance: for front-side marker, ray hits sphere after passing marker? no: ray from camera outside, hits marker (1.004) first, then sphere entry? Sphere entry distance < marker distance only if marker is behind sphere surface along ray… front marker at 1.004: ray reaches marker before sphere surface → markerDist < sphereEntryDist → not occluded ✓. Back city: sphereEntry < cityDist → occluded ✓.)

Good.

Also skip hover during drag.

Now heat overlay radius 1.003 to avoid z-fighting with earth (polygonOffset alternative). 1.003 fine.

Render order: earth 0, heat 1, routes 2, dots 3, rings/markers default fine (depthTest), atmo 10.

Sprites depthTest true fine.

Atmosphere renderOrder high + depthWrite false.

Stars renderOrder -1? default fine (far away).

One more embellishment: subtle "grid" graticule toggle? skip, features are enough.

Also "city lights effect, especially night area" handled by night texture + glow sprites stronger at night ✓.

Now let me write final code. Long; be systematic. Also double-check every DOM id referenced in JS exists.

Potential GLSL compile issue: `uniform sampler2D dayMap, nightMap, waterMap;` — valid. ✓

three r160 ShaderMaterial: `varying` OK (WebGL2 auto-converted). ✓

PointsMaterial map + alphaTest? Additive with soft map fine.

Sprite: THREE.Sprite with SpriteMaterial({map:glowTex, color, transparent, blending:Additive, depthWrite:false}). Set scale.

RingGeometry(0.82,1,48).

City core: SphereGeometry(1,14,14) scaled by size.

Also store city.sizeCurrent for ring scaling.

Popup route count: ROUTES.filter(r=>r.a===c||r.b===c).length.

Also add keyboard: space toggles play — nice; add keydown listener (ignore when typing — no inputs). Add.

Also 'r' for reset view? skip, space is enough.

Also ensure canvas doesn't capture pointer when on panel: canvas is only renderer.domElement sized to full window but panel is above it (z-index) → pointer events go to panel naturally. ✓

Wheel: attach on renderer.domElement.

Prevent page scroll: body overflow hidden.

Mobile: touch drag works via pointer events; pinch via 2-pointer tracking; wheel N/A.

Now let me generate the code.

Also small detail: `renderer.outputColorSpace = THREE.SRGBColorSpace;` default anyway.

Textures: day.colorSpace=SRGB; night SRGB; water default linear ✓.

Let me write animate():

```js
const clock=new THREE.Clock();
function animate(){
 requestAnimationFrame(animate);
 const dt=Math.min(clock.getDelta(),0.05), t=clock.elapsedTime;
 // idle spin
 if(state.spin && !dragging && t-lastInteract>2.5 && !state.selCity) state.tRotY += dt*0.05;
 // smoothing
 state.rotY += (state.tRotY-state.rotY)*(1-Math.exp(-dt*7));
 state.rotX += (state.tRotX-state.rotX)*(1-Math.exp(-dt*7));
 state.dist += (state.tDist-state.dist)*(1-Math.exp(-dt*6));
 earth.rotation.order='YXZ'; earth.rotation.y=state.rotY; earth.rotation.x=state.rotX;
 camera.position.copy(CAM_DIR).multiplyScalar(state.dist); camera.lookAt(0,0,0);
 // sun drift
 sunAz += dt*0.006; sunDir.set(Math.cos(sunAz),0.24,Math.sin(sunAz)).normalize();
 uniforms... camPos
 sharedTime.value=t;
 updateCities(t,dt); updateDots(t); hover lerp routes; stars rotate;
 if(needPick){ doHover(); needPick=false; }
 if(state.playing){ advance year; }
 popup reposition; hud throttle;
 renderer.render(scene,camera);
 fps count;
}
```

lastInteract uses performance-based t (clock elapsed). Set on pointerdown/move-drag/wheel.

Year advance:
```js
state.yearFloat += dt*0.45*state.speed;
if(state.yearFloat>2024) state.yearFloat-=5;
syncTimeline(); checkYearTick();
```
syncTimeline updates slider + year display (cheap per frame OK).

checkYearTick: iy=floor(yearFloat); if iy!==lastYear → lastYear=iy; onYearTick().

onYearTick(): rebuildHeat(); updateRanking(); updateTotal(); refresh popup if open; flash year.

Also on metric change: onYearTick-ish + accent + legend + marker targets + popup re-render.

updateCities each frame recomputes min/max for metric at yearFloat (34 values) → sizes target; lerp.

Perf fine.

Popup reposition each frame if selCity.

Now write updateCities:

```js
function updateCities(t,dt){
 const m=state.metric, yf=state.yearFloat;
 let mn=Infinity,mx=-Infinity; const vals=CITIES.map(c=>{const v=metricValue(c,m,yf); ...});
 compute mn/mx;
 CITIES.forEach((c,i)=>{
  const n=(vals[i]-mn)/(mx-mn+1e-9);
  const target=0.012+0.02*Math.sqrt(n);
  c.size += (target-c.size)*(1-Math.exp(-dt*6));
  c.core.scale.setScalar(c.size);
  c.coreMat.color.lerp(metricColorObj, 1-Math.exp(-dt*6));
  c.spriteMat.color.copy? also lerp;
  // world position for night factor
  _v.copy(c.pos); earth.localToWorld(_v); const s=_v.normalize().dot(sunDir);
  const nightF=THREE.MathUtils.clamp((0.12-s)/0.35,0,1);
  const pulse=0.85+0.15*Math.sin(t*2.2+c.phase);
  c.spriteMat.opacity=(0.3+0.7*nightF)*pulse*(layers.cities?1:1);
  c.sprite.scale.setScalar(c.size*(5.5+2.5*nightF));
  // ring
  const rp=((t*0.55)+c.phase*0.13)%1;
  c.ring.scale.setScalar(c.size*(1.3+rp*2.6));
  c.ringMat.opacity=(1-rp)*0.75*(c===state.hoverCity?1:0.8);
  c.ringMat.color.copy? set from metric color (set once on metric change)
 });
}
```
Sprite scale in world units (sprite is child of earth group which only rotates → scale unaffected). ✓

Hover boost: if c===hoverCity → core.scale extra *1.35: incorporate: scale = c.size*(hover?1.4:1) with smoothing via c.hoverAmt lerp.

Add c.hoverAmt.

Ring lookAt set once at build: ring.position=c.pos*1.004; ring.lookAt(c.pos*2). Ring is child of earth group ✓.

Sprite position = c.pos*1.006.

Core position = c.pos*1.004.

Store c.pos unit.

Dots update:
```js
ROUTES.forEach((r,i)=>{ const sp=0.06+0.10*(r.wk/maxWk); const tt=(t*sp + r.phase)%1; const f=tt*(r.pts.length-1); const i0=Math.floor(f), i1=Math.min(i0+1,r.pts.length-1); _p.lerpVectors(r.pts[i0],r.pts[i1],f-i0); posArr.set([...],i*3); });
attr.needsUpdate=true;
```

Route hover lerp: r.hover += ((r===hoverRoute)?1:0 - r.hover)*k; mat.uniforms.uHover.value=r.hover.

doHover(): raycast; set hoverCity/hoverRoute; update cursor and tooltip content/position (position updated on pointermove too; here just content and visibility). Tooltip position stored from event.

Tooltip show(x,y,html).

Click: pick city → openPopup(city) & focusCity? just open popup (no auto rotate). Also small focus? Keep just popup.

openPopup(c): state.selCity=c; render html; show.
closePopup(): hide; selCity=null.

Popup position each frame:
```js
_v.copy(c.pos); earth.localToWorld(_v);
const facing=_v.clone().normalize().dot(camera.position.clone().normalize());
if(facing<0.18){popup.classList.add('hidden')} else {remove; project → px; place with offset; }
```
project: _v.project(camera); x=(v.x*0.5+0.5)*w; y=(-v.y*0.5+0.5)*h.

Place card at x+18,y-20 clamp.

OK final write. Also add `will-change: transform` on popup for smoothness.

Popup content function buildPopup(c): compute values at current integer year (use yearFloat). Rows for 3 metrics with active class on state.metric. Sparkline across YEARS for state.metric.

Sparkline svg:
```js
const vals=YEARS.map(y=>metricValue(c,m,y));
min/max → pts string; area path.
dot at fractional position of current yf.
```
Color accent.

Also show delta% 2019→2024: nice: `+23.4% vs 2019`.

Route tooltip content:
```
<b>CA983 · 中国国际航空</b>
<div class=tt-route>北京 ⇌ 洛杉矶</div>
机型 B777-300ER
每周 14 班 · 距离 10,424 km
准点率 87%
```

City tooltip:
```
<b>北京</b> BEIJING
GDP 8,120 billion · Population 21.9M · 1,050 flights/day (show only active metric + hint "click for details")
```
Keep 2 lines.

Now finalize heat legend labels: lgMin/lgMax formatted with fmtVal short (without long units): gdp "9,200 billion" etc. Provide fmtShort(m,v): gdp→`${Math.round(v).toLocaleString()}亿`, pop→`${v.toFixed(0)}M`, flt→`${Math.round(v).toLocaleString()}班`.

Total uses fmtVal long.

OK, really writing now. Generate final answer with brief intro, code block, then run/test notes.

Also for safety on `import('topojson-client')` — some browsers resolve importmap for dynamic import ✓.

Also note: jsdelivr +esm for topojson-client@3.1.0 — package has module field? topojson-client ships ESM (src/module)? has "module": "src/index.js"? jsdelivr +esm will convert CJS anyway. Should work.

Let me write out the full file. (Careful with matching braces.)

Also HUD coords: display "lat 34.2° N lon 108.9° E" format.

fmtCoord: `${Math.abs(lat).toFixed(1)}°${lat>=0?'N':'S'} ${Math.abs(lon).toFixed(1)}°${lon>=0?'E':'W'}`.

Zoom display: `altitude ${(dist-1)*6371|0} km`.

FPS.

Hint text.

Also add `#stage canvas{display:block}`.

Deliver.

One more consideration for earth shader: placeholder texture before load: make a small DataTexture? Use CanvasTexture 2x1 dark. Initialize uniforms with placeholder canvas texture, swap later. nightMap placeholder black, waterMap placeholder black (uHasWater 0 anyway until loaded).

makePlaceholder(color): canvas 16x8 fill.

On day load: earthUniforms.dayMap.value=tex; on water: uHasWater=1.

Also heat mesh created after geo load or fallback: create immediately with empty transparent canvas; redraw later.

Now generate final answer. Code will be long; that's fine.

Also double-check ring geometry orientation: RingGeometry lies in XY plane, faces +Z. mesh.lookAt(target) orients +Z toward target. Position p, lookAt(2p) → +Z outward ✓.

Sprite inside group? Add directly to citiesGroup with position.

citiesGroup & routesGroup & dots are children of earth.

earth group named globe.

Star material size: size 1.6? sizeAttenuation true with radius ~50 → small. Use size 0.55 and 0.3 for two layers? At distance 50, size 0.5 → tiny. Points size in world units with attenuation: apparent = size * (scale/ -mvPosition.z)… size 0.6 fine; tune: layer1 count 900 size 0.5 opacity .95; layer2 1800 size 0.28 opacity .6. Colors vary.

Twinkle: mat.opacity oscillate ±10%.

OK write.

Also don't forget `renderer.domElement.id='gl'`.

Also add `window.addEventListener('resize')`.

Also `matchMedia('(max-width:900px)')` → panel hidden by default (add class), toggle button visible always.

Panel toggle button top-right: "⚙ 控制台" chip.

Final check on popup hidden class vs display: use .off {opacity:0; pointer-events:none; transform scale .98} transition.

Now output. Write code directly in final answer, mindful of correctness.

While writing, keep all ids consistent. Let me draft the JS in final form.

Also: `const D2R=Math.PI/180`.

metricColorObj = new THREE.Color(METRICS[m].color) updated on metric change.

Ring color set on metric change: cities.forEach(c=>c.ringMat.color.set(col)).

Sprite color lerp similarly to core.

Route colors fixed.

Dot colors set once from route color.

updateTotal tween: keep simple: animateNumber(el, currentShown, target, fmt). Store el._val.

Ranking build: sort desc by value at Math.floor? use yearFloat. Slice 5. Row html with width %.

Row click: focusCity(city)+openPopup.

focusCity(c): compute targetRotY shortest, targetRotX=clamp(lat rad*0.8), lastInteract=t (to pause spin), also state.selCity set by caller openPopup.

Also resetView button: tRotY=init, tRotX=0.15, tDist=3.35.

initRotY = D2R*(-90-115) = D2R*-205 → normalize? fine as-is (any rad OK).

Also initial rotY = same (no animation) or start offset -0.6 for entrance spin-in: rotY = tRotY - 0.7 → eases in. Nice entrance. Also initial dist 5.2 → tDist 3.35 ease-in. Cinematic load ✓.

lastInteract= -10 so spin starts after entrance? Spin condition idle>2.5s; set lastInteract=0 on load → spin begins ~2.5s later. OK.

Edge: popup open pauses spin ✓ (condition !state.selCity).

Now finalize sun az initial: sunAz=Math.atan2(0.8,0.55)? Just set sunAz=0.9 rad; sunDir=(cos0.9,0.24,sin0.9)=(0.62,0.24,0.78) → day side covers front-right — with initial view Asia front → Asia day, Americas terminator.

Write final.

Also `#tip` transform translate3d for perf.

Tooltip flip near right edge: if x> w-260 place on left (x - width-28). Simple clamp.

OK go. Also include console.info banner.

Answer structure:
1. One-line description
2. Complete code
3. ▶ Run instructions
4. ✓ Test checklist (feature mapping)
5. Notes (network dependency / fallback / data disclaimer)

Write in Chinese.