The user wants a 3D data visualization globe with:
- 3D Earth with real textures, atmosphere glow
- City markers with glowing points
- Flight routes (arcs) with flowing animation
- Country heatmap coloring
- Interactions: drag rotate, scroll zoom, click city for popup, hover route for flight info
- Control panel: switch metrics (GDP, population, flight volume), timeline playback
- Visual effects: auto-rotation, day/night terminator, city lights on night side
- Tech: Three.js, real earth textures, importmap + ESM, single HTML file

Let me build a comprehensive single HTML file. I'll use Three.js from CDN with importmap. For textures, I'll use the standard three.js examples textures from unpkg or jsdelivr (three-globe style textures). Common reliable texture URLs:

- `https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg`
- `https://unpkg.com/three-globe/example/img/earth-topology.png`
- `https://unpkg.com/three-globe/example/img/earth-night.jpg`
- `https://unpkg.com/three-globe/example/img/earth-day.jpg`

These are commonly available. I can use earth-day.jpg, earth-night.jpg, and earth-topology.png for bump.

Day/night terminator: I can implement a shader that blends day and night textures based on the sun direction dot product with the normal. That's the proper way. I'll write a custom ShaderMaterial for the earth that:
- samples day texture and night texture
- computes sun direction uniform (updated over time, or fixed angle)
- mixes based on smoothstep of dot(normal, sunDir)
- also adds bump/specular maybe. Bump in custom shader is more complex; could do a simple derivative-based bump or skip. I can use onBeforeCompile on MeshPhongMaterial to inject night texture blending — that gives lighting plus bump map support. Actually simpler: custom ShaderMaterial with lambert-ish lighting. But bump mapping adds realism. Let me use MeshPhongMaterial with onBeforeCompile to mix night lights on dark side. That gives specular highlights for oceans (specular map), bump map for topology, and day/night mixing.

Actually a well-known approach: use MeshPhongMaterial with map=day texture, bumpMap=topology, and in fragment shader after sampling diffuseColor, mix with night texture based on sun direction. Need sun direction in view space... easier in world space. I can pass world normal via varying.

Let me write onBeforeCompile:

```js
material.onBeforeCompile = (shader) => {
  shader.uniforms.sunDirection = sunUniform;
  shader.uniforms.nightMap = { value: nightTexture };
  shader.vertexShader = shader.vertexShader
    .replace('#include <common>', '#include <common>\nvarying vec3 vWorldNormal;')
    .replace('#include <defaultnormal_vertex>', '#include <defaultnormal_vertex>\nvorldNormal = normalize(mat3(modelMatrix) * objectNormal);');
  shader.fragmentShader = shader.fragmentShader
    .replace('#include <common>', '#include <common>\nuniform vec3 sunDirection;\nuniform sampler2D nightMap;\nvarying vec3 vWorldNormal;')
    .replace('#include <map_fragment>', `
      #ifdef USE_MAP
        vec4 dayColor = texture2D(map, vMapUv);
        vec4 nightColor = texture2D(nightMap, vMapUv);
        float sunDot = dot(normalize(vWorldNormal), normalize(sunDirection));
        float mixAmount = smoothstep(-0.15, 0.25, sunDot);
        vec4 texelColor = mix(nightColor * 1.2, dayColor, mixAmount);
        diffuseColor *= texelColor;
      #endif
    `);
};
```

Note the night texture from three-globe has black background with city lights — good.

Sun direction: slowly animate it or keep it static-ish. Real sun direction changes with time of day; I can animate over a "day" cycle tied to the timeline or just slowly rotate. I'll animate sun direction rotating around Y axis slowly (e.g., full cycle in 60s), or tie to timeline. Simplest: animate slowly; user can see terminator move.

Atmosphere glow: a slightly larger sphere with shader (fresnel backside additive blending), classic glow.

Heatmap coloring by country: Real country polygons would require GeoJSON data. Loading world GeoJSON from CDN: `https://unpkg.com/world-atlas@2/countries-110m.json` (TopoJSON). To color countries, I'd need topojson-client to convert. That's possible: load topojson, convert to geojson, draw countries as line meshes on globe with heat color fills. Filling spherical polygons is complex — could use three-globe's polygons but we're hand-rolling. Alternative approach: generate heat coloring procedurally via a canvas texture — draw colored country shapes onto a canvas using equirectangular projection of GeoJSON polygons, then use that canvas as an overlay texture on a slightly larger sphere with transparency. That's a solid approach: draw filled polygons with canvas 2D using equirectangular projection (geojson coordinates are lon/lat, map directly to canvas x = (lon+180)/360*w, y = (90-lat)/180*h). This works well for 110m data.

So plan:
- Load `https://unpkg.com/world-atlas@2/countries-110m.json` and `topojson-client` from CDN (import map: `topojson-client` from `https://unpkg.com/topojson-client@3?module` or jsdelivr esm). Use jsdelivr: `https://cdn.jsdelivr.net/npm/topojson-client@3/+esm` works as ESM.

But network reliability... The environment may or may not have internet. Since requirement says importmap + ESM and real textures, assume internet available. I should add fallbacks: if texture fails, generate procedural canvas texture. Add error handling with onError for texture loads — generate fallback canvas-based earth texture. Good for robustness.

Heat color per country: define data for ~30-50 major countries across metrics (GDP, population, flights) with values for years 2000-2023 (time steps). Others get default/interpolated. Map country by ISO numeric id (world-atlas 110m has id = numeric ISO code as string, and properties.name). I'll build a data table keyed by ISO numeric code (as string, e.g., "840" USA, "156" China). Generate synthetic-but-plausible data programmatically: base value per country per metric, growth curve per year. That keeps the data table compact — define base values for major countries, and compute yearly values with a growth function (deterministic pseudo-random based on hash of id).

Time axis: years 2000..2024, play button animates year, updates heat colors, city marker sizes, route intensity maybe.

Cities: ~24 major cities with lat/lon, name, country. Marker: glowing point — use sprite with radial gradient texture + small sphere. Points scale with metric value (e.g., city GDP/population). Click → raycast → popup HTML with city info (name, country, values for current metric and year).

Routes: arcs between cities (e.g., 20 routes). Arc: QuadraticBezier/CubicBezier from A to B with control point elevated. Flow animation: moving dash — use TubeGeometry with shader? Simpler: Line with dashed material animating dashOffset, plus moving glow "pulse" points traveling along curve (small sprites whose position = curve.getPoint(t), t animated). I'll do: Line (fat lines not necessary) with color, plus animated pulses. Hover route → raycast lines (threshold via Raycaster.params.Line.threshold) → tooltip with flight info (from-to, flights per year for current year).

Controls: OrbitControls with enableDamping, rotate on drag, zoom on wheel. Auto-rotation: rotate earth group slowly when user not interacting (OrbitControls autoRotate = true works with damping; pause on user interaction via 'start'/'end' events).

City lights at night: handled by night texture blending.

Control panel UI: top-left panel with metric buttons (GDP / 人口 / 航班量）, timeline slider + play/pause at bottom, legend showing color scale, info popups.

Also add starfield background.

Let me also handle DPR, resize.

Now the data. Cities (name, lat, lon, country):
- New York (40.71, -74.00, USA)
- Los Angeles (34.05, -118.24)
- London (51.50, -0.12)
- Paris (48.85, 2.35)
- Frankfurt (50.11, 8.68)
- Tokyo (35.68, 139.69)
- Beijing (39.90, 116.40)
- Shanghai (31.23, 121.47)
- Hong Kong (22.32, 114.17)
- Singapore (1.35, 103.82)
- Dubai (25.20, 55.27)
- Sydney (-33.87, 151.21)
- São Paulo (-23.55, -46.63)
- Mexico City (19.43, -99.13)
- Moscow (55.76, 37.62)
- Mumbai (19.08, 72.88)
- Delhi (28.61, 77.21)
- Johannesburg (-26.20, 28.05)
- Cairo (30.04, 31.24)
- Toronto (43.65, -79.38)
- Seoul (37.57, 126.98)
- Chicago (41.88, -87.63)
- Istanbul (41.01, 28.98)
- Bangkok (13.76, 100.50)

City data per metric: GDP (billion USD), population (millions), flights (thousands/year). Give base (2024) values and growth factors; value(year) = base * f(year - 2000) per metric, e.g., gdp grows ~4%/yr, population ~1%, flights ~3%, with per-city variation. Simple: value(year) = base / Math.pow(1+rate, 2024-year). Deterministic.

Routes: pick ~22 routes among cities with a "flights per day" base that scales over time.

Country heat data: metric per country ISO. Use topojson feature id (numeric string). Base values for major countries; default for others = hash-based small value. Color scale: logarithmic-ish mapping to gradient (dark blue → cyan → green → yellow → red). Draw to canvas 4096x2048 (or 2048x1024 for perf; 4096 is heavy but fine) with alpha ~0.55 fill + borders.

For Antarctica etc., fill transparent.

Canvas overlay sphere: radius 1.002, MeshBasicMaterial with map=CanvasTexture, transparent true, depthWrite false. Slight opacity.

Border lines: draw country borders on same canvas with strokeStyle rgba(255,255,255,0.25).

Handling polygon drawing with antimeridian crossing: 110m data has issues with polygons crossing the antimeridian (e.g., Russia, Fiji, US/Alaska). Simple approach: draw each ring; if a segment jumps more than 180° in longitude, moveTo instead of lineTo — prevents streaks. Russia will have its Chukotka part drawn near -180..-170 and main part 30..180; with jump handling it draws both pieces as separate subpaths but fill should still work if we just moveTo (fill will connect across? With fill, moveTo starts new subpath; fill fills each subpath — since it closes each subpath, the eastern tip piece will be filled separately, fine).

Antarctica polygon spans full width in 110m (it's cut along the antimeridian with poles) — drawing may produce a giant fill; acceptable with jump handling.

Actually, the 110m countries TopoJSON from world-atlas: `objects.countries` geometries with `arcs`. topojson.feature gives MultiPolygon/polygons in lon/lat. Antarctica's polygon typically covers (-180,-90)..(180,-63) roughly with jump handling fine.

Sun direction: compute from time — animate: sun rotates around Y (ecliptic simplified). sunDir = (cos(a), 0.15*sin(a*0.3)... keep simple: rotate around Y axis with slight z tilt. angle = elapsed * 2π / 120 (2 min per day cycle).

Terminator smooth: smoothstep(-0.2, 0.2, dot).

Also boost night lights: nightColor * something, and also add slight emissive feel.

Specular: use specular map? three-globe has `earth-water.png`? Not sure. There's `earth-topology.png` for bump. Skip specular map; set shininess moderate, specular gray — oceans will look okay-ish. Fine.

Clouds? Optional — could add `https://unpkg.com/three-globe/example/img/earth-clouds.png`? I don't think that file exists there. There's a clouds texture at `https://unpkg.com/three-globe/example/img/earth-clouds.png`? Actually three-globe example img folder contains: earth-blue-marble.jpg, earth-day.jpg, earth-dark.jpg, earth-night.jpg, earth-topology.png, earth-water.png, night-sky.png. I believe it does have those. Not 100% sure about clouds. Skip clouds to be safe.

Textures to load:
- day: earth-blue-marble.jpg (has nice colors) or earth-day.jpg. I'll use earth-blue-marble.jpg as primary with fallback to earth-day.jpg? Keep one: earth-blue-marble.jpg.
- night: earth-night.jpg
- bump: earth-topology.png

Fallback procedural texture generator if load fails: draw gradient ocean + random blobs continents + night version with dots. I'll implement modest fallback.

Importmap:
```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/",
    "topojson-client": "https://unpkg.com/topojson-client@3.1.0/dist/topojson-client.esm.js"
  }
}
</script>
```

topojson-client has ESM build at dist/topojson-client.esm.js — yes it exists ("module": "dist/topojson-client.esm.js"). unpkg resolves. Good. Alternatively jsdelivr +esm. I'll use unpkg direct esm file.

OrbitControls from three/addons/controls/OrbitControls.js.

Also load world-atlas: fetch('https://unpkg.com/world-atlas@2.0.2/countries-110m.json'). If fetch fails, skip heat layer gracefully (or draw random heat by latitude bands? just hide with note).

Raycasting cities: use sprites — raycastable. Give each city an invisible larger hit-sphere for easier clicking.

Route hover: raycast against line objects with threshold. Also add invisible thicker hit proxy? Raycaster Line threshold works in world units; set params.Line.threshold = 0.02 and scale appropriately. Globe radius 1, camera distance ~2.5-3. Threshold 0.015 okay. Might be finicky; also generate TubeGeometry invisible hit tubes? That's 20 tubes × low segments — cheap and reliable. I'll create invisible tubes (radius 0.008) for raycast, visible line + pulses on top.

Popup: absolutely positioned div; on click city, project marker position each frame while open to keep it anchored; close button.

Tooltip for routes: follows mouse, shows on hover.

Legend: gradient bar with min/max labels updating per metric.

Metric affects:
- Heat colors per country (data table)
- City marker scale (city data)
- Route pulse speed/thickness intensity (flights metric mainly; keep routes tied to flights data but scale pulses by selected year)

Timeline: slider 2000–2024, play toggles interval advancing year, updates everything, shows year label.

Sun animation independent.

Auto-rotate: controls.autoRotate = true, speed 0.5; pause on 'start' event, resume after 3s idle on 'end'.

Now country data. world-atlas 110m ids are numeric ISO 3166-1 numeric as strings (e.g. "840" USA, "156" China, "250" France... but France in 110m has id "250"? France id is "250" including... in some versions France id is "-99" with Norway... For countries-110m.json from world-atlas@2, ids are mostly ISO numeric; France = "250", Norway = "578". Kosovo = "-99" maybe. Somaliland "-99". Fine.)

Data table (base 2024 values):

GDP (trillion USD, 2024-ish): 
- 840 US 27.4, 156 China 17.8, 392 Japan 4.2, 276 Germany 4.5, 826 UK 3.3, 356 India 3.6, 250 France 3.0, 380 Italy 2.2, 076 Brazil 2.1, 124 Canada 2.1, 643 Russia 2.0, 410 South Korea 1.7, 036 Australia 1.7, 724 Spain 1.6, 484 Mexico 1.8, 360 Indonesia 1.4, 528 Netherlands 1.1, 792 Saudi 1.1, 792? Saudi = 682. Turkey 792 = 1.1, 756 Switzerland 0.9, 616 Poland 0.8, 032 Argentina 0.6, 710 South Africa 0.38, 818 Egypt 0.4, 566 Nigeria 0.36, 398? Kazakhstan 0.26, 784 UAE 0.5, 752? no 752 Sweden 0.6, 566? Singapore 702 = 0.5, 704 Vietnam 0.43, 764 Thailand 0.5, 458 Malaysia 0.4, 608 Philippines 0.44, 586 Pakistan 0.34, 050 Bangladesh 0.45, 144? Bangladesh = 050. 

Population (millions 2024):
- 156 China 1412, 356 India 1420, 840 US 335, 360 Indonesia 275, 586 Pakistan 240, 076 Brazil 216, 566 Nigeria 223, 050 Bangladesh 173, 643 Russia 144, 484 Mexico 128, 392 Japan 125, 608 Philippines 117, 818 Egypt 112, 704 Vietnam 98, 800? DR Congo 180 = 102, 792 Turkey 85, 364 Iran 89, 276 Germany 84, 764 Thailand 72, 826 UK 67, 250 France 68, 710 South Africa 60, 380 Italy 59, 410 South Korea 52, 724 Spain 48, 032 Argentina 46, 124 Canada 39, 792 Saudi 682 = 37, 036 Australia 26, 458 Malaysia 34, 528 Netherlands 18.

Flights (million passengers or flights k? use "million passengers/year" per country 2024... simpler: 万架次？ I'll use "万架次" (10k flights). US ~1000 (10M flights), China ~500... use index-ish numbers: US 980, China 520, UK 200, Germany 190, Japan 130, India 110, Spain 170, France 160, Brazil 90, Turkey 180, UAE 95, Russia 85, Canada 95, Australia 75, Italy 140, Mexico 90, Thailand 75, Indonesia 85, Korea 70, Netherlands 70.

This is getting long; I'll define data as compact objects keyed by ISO with arrays [gdp, pop, flights] 2024 base, then yearly = base * growth^(2024-year) with per-metric growth rates and per-country growth variance from hash. GDP growth: developed ~2%, developing higher (China 8%, India 6%). I can encode a growth multiplier per metric per country implicitly via hash + region? Simpler: growthRate = base rate per metric + (hash(id)%5 - 2)*0.005. China's GDP in 2000 was 1.2T → factor 27/1.2 ≈ 22 over 24 years → annual 13.7% nominal USD... too high variance to model per-country properly. Use per-country growth override for a few: China gdp growth 0.115, India 0.07, Vietnam 0.08; default gdp 0.03; pop default 0.012 (China 0.004, Nigeria 0.026, India 0.011, Japan -0.001); flights default 0.035 (China 0.09, India 0.07).

Good enough for a demo; it's synthetic plausible data. I'll note in UI "示例数据".

Growth function: value(year) = base * Math.pow(1+g, year-2024).

Color scale: normalize per metric across country values at current year: use log scale between min and max of defined countries. Map t to gradient stops: [ '#081d3a', '#0b3d66', '#0e7490', '#22a06b', '#a3c14a', '#f2c14e', '#ef8354', '#d62828' ]? Use HSL interpolation: hue 210→0. I'll implement gradient array and lerp.

City data: [gdp B$, pop M, flights k/day? ] 2024 base with same growth approach.

New York gdp 2100B? Metro GDP ~2.1T. Use metro GDP billions: NY 2100, LA 1200, London 900, Paris 850, Tokyo 1800, Shanghai 700, Beijing 680, HK 400, Singapore 420, Dubai 130, Sydney 400, SãoPaulo 500, MexicoCity 400, Moscow 450, Mumbai 350, Delhi 330, Johannesburg 120, Cairo 150, Toronto 400, Seoul 800, Chicago 750, Istanbul 300, Bangkok 250.
Population M (metro): NY 19.5, LA 12.5, London 14.3, Paris 11, Tokyo 37.4, Shanghai 29, Beijing 21.5, HK 7.5, Singapore 6, Dubai 3.6, Sydney 5.3, SãoPaulo 22.6, MexicoCity 22.3, Moscow 12.5, Mumbai 21.3, Delhi 32.9, Johannesburg 6.2, Cairo 21.8, Toronto 6.4, Seoul 26, Chicago 8.9, Istanbul 15.8, Bangkok 11.
Flights: daily departures approx: NY 1200, LA 1100, London 1100, Paris 950, Tokyo 900, Shanghai 800, Beijing 900, HK 700, Singapore 650, Dubai 900, Sydney 500, SãoPaulo 700, MexicoCity 600, Moscow 650, Mumbai 550, Delhi 700, Johannesburg 300, Cairo 350, Toronto 700, Seoul 700, Chicago 1400, Istanbul 900, Bangkok 550. (per day)

Routes (from,to, daily flights): 
NY-London 28, NY-Paris 12, NY-Tokyo 8, London-Dubai 14, London-Singapore 9, Tokyo-Shanghai 15, Shanghai-Singapore 12, Singapore-Sydney 10, Dubai-Mumbai 18, Dubai-London(dup skip), Paris-Cairo 6, Frankfurt-NY? add Frankfurt-Beijing 7, HongKong-Tokyo 16, HongKong-Singapore 14, SãoPaulo-MexicoCity 8, MexicoCity-LA 12, LA-Tokyo 10, Sydney-LA 6, Moscow-Istanbul 10, Istanbul-Dubai 8, Johannesburg-Dubai 5, Seoul-Shanghai 18, Chicago-NY 30, Toronto-London 8, Bangkok-Singapore 15, Delhi-Dubai 12, Frankfurt-Istanbul 9.

City marker scaling: radius by sqrt(value/max) for current metric & year.

Pulses: for each route 1-2 pulses, speed ~ proportional to flights.

Heat overlay redraw on metric/year change: draw to 2048x1024 canvas. Drawing ~250 countries × paths each change — fine (< 50ms likely). Could also just redraw on change only.

Country names for legend/hover: skip country hover (not required).

Also "国家/地区热力着色" — done via canvas overlay.

Night lights boost: also add subtle additive glow of night texture: it's in the mix already.

Starfield: Points with random positions on sphere radius 50, size attenuation off... PointsMaterial size 1.5, sizeAttenuation false? That makes them uniform pixels. Use texture circle. Simple white points fine, maybe slight color variance. Or use night-sky.png as background sphere? Points simpler.

Loading manager: show loading overlay until textures + topojson loaded (or failed).

Let me now think about the onBeforeCompile injection details for three r160.

MeshPhongMaterial fragment shader chunks: `#include <map_fragment>` exists. In r160, `vMapUv` is the varying name (since r151+ they renamed vUv per-map). Yes, in newer three, map uses `vMapUv`. Phong vertex includes `#include <uv_vertex>` creating vMapUv when USE_MAP.

I'll inject:
- vertex: after `#include <common>` add `varying vec3 vWorldNormal;`; after `#include <worldpos_vertex>`? Simpler: after `#include <defaultnormal_vertex>` compute `vWorldNormal = normalize(mat3(modelMatrix) * objectNormal);` — modelMatrix is available in vertex. objectNormal defined in beginnormal_vertex which precedes defaultnormal_vertex. Good.

- fragment: after `#include <common>` add uniforms declarations & varying. Replace `#include <map_fragment>` with custom code using vMapUv.

Also need uniform in shader.uniforms: shader.uniforms.uSunDir = sunUniform; shader.uniforms.uNightMap = { value: nightTex }.

Sun direction world: earth mesh at origin, not rotated itself (rotation group rotates around it? If earth rotates for self-rotation, the normal transform handles it since we use modelMatrix). We rotate a group containing everything (earth + markers + arcs) for user rotation via OrbitControls (camera orbits instead). Earth self-rotation: rotate earth mesh itself slowly? But markers must rotate with it. If OrbitControls orbits camera, earth self-rotation would spin markers — a spinning globe with markers is fine but clicking becomes hard and day/night cycles by earth rotation... Classic approach: keep earth static, autoRotate camera. The "自转动画" requirement: I'll rotate the whole globe group slowly around Y (tilted axis) when idle — implemented via controls.autoRotate which orbits camera; visually identical to earth rotating. Markers stay aligned with earth. Good — use autoRotate.

But the day/night terminator: sun fixed direction (animating slowly). Earth static means countries stay in same light unless we rotate earth relative to sun. With camera autoRotate, lighting on globe appears to change as camera moves (since light fixed in world). Fine. Plus animate sun direction slowly so terminator drifts over surface. Good.

Atmosphere: ShaderMaterial backside sphere radius 1.15 (globe r=1):

vertex: pass normal & view dir; intensity = pow(0.65 - dot(vNormal, viewDir)... standard:

```
varying vec3 vNormal;
void main(){ vNormal = normalize(normalMatrix * normal); gl_Position = projectionMatrix*modelViewMatrix*vec4(position,1.0); }
```
fragment:
```
varying vec3 vNormal;
void main(){ float intensity = pow(0.7 - dot(vNormal, vec3(0,0,1.0)), 3.0); gl_FragColor = vec4(0.3,0.6,1.0,1.0)*intensity; }
```
side: BackSide, blending Additive, transparent. Also inner atmosphere: slight fresnel on front side sphere radius 1.02 for rim. I'll do backside glow + subtle frontside rim.

Glow texture sprite for city markers: canvas radial gradient.

Pulses: small sprites (same glow texture, colored orange/white) moving along curve. Each route: pulses count = 1 + floor(flights/10) capped 3? Use 2. Store t and speed.

Arc geometry: 
```
function arcPoints(a, b){
  const start = latLonToVec3(a.lat,a.lon,1.005);
  const end = latLonToVec3(b.lat,b.lon,1.005);
  const dist = start.distanceTo(end);
  const mid = start.clone().add(end).multiplyScalar(0.5).normalize().multiplyScalar(1.005 + dist*0.25);
  const curve = new THREE.QuadraticBezierCurve3(start, mid, end);
  return curve;
}
```
Line: BufferGeometry from curve.getPoints(60), LineBasicMaterial color, transparent, opacity 0.6. Additive blending for glow feel.

latLonToVec3: 
```
const phi = (90-lat)*Math.PI/180; const theta = (lon+180)*Math.PI/180;
x = -r sinφ cosθ, y = r cosφ, z = r sinφ sinθ
```
Standard three-globe mapping matching equirectangular texture orientation.

Verify: texture u=0 at lon -180... With sphere geometry default, three.js SphereGeometry UV maps such that this formula works (it's the common one used with these textures). Yes.

Heat canvas overlay must match same mapping: x = (lon+180)/360 * W, y = (90-lat)/180 * H. Since the texture wraps the sphere with standard mapping, this matches the marker formula. 

UI Layout:
- Left panel: title, metric buttons (GDP/人口/航班量）, legend with gradient + min/max, info text.
- Bottom: timeline with play button, year display, slider.
- Top-right: hint （拖拽旋转 / 滚轮缩放 / 点击城市 / 悬停航线）.
- Popup div for city.
- Tooltip div for routes.
- Loading overlay.

Also toggle buttons: 自转 on/off, 热力图层 on/off, 航线 on/off maybe. Nice extras — keep a few toggles: 自动旋转， 热力图， 航线， 城市标记. Checkboxes.

Language: UI in Chinese since request is Chinese.

Now code structure (module script):

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import * as topojson from 'topojson-client';
```

Guard: if topojson fetch fails, catch.

Data definitions as above.

Renderer: antialias, alpha false, clearColor black... use scene background null + CSS gradient? Stars via Points; background near-black.

Camera fov 45, pos z=3.2? Globe radius 1. z=3 gives globe filling nicely.

Lights: ambient 0.35 (so night side not pure black? Night texture provides lights; ambient adds base), directional sun light from sunDir * 10 with intensity ~1.6. Directional light position updated with sun dir. Actually with custom day/night mixing, do we even need directional light shading? The mixing handles day/night; Phong directional would double-shade. Simpler: use MeshBasicMaterial-style custom shader? But bump mapping needs lights. Alternative: use full custom ShaderMaterial (lambert term = clamp(dot(n, sun))) with day/night mix, fake bump via texture offset sampling (cheap normal perturbation using topology luminance gradient). Hmm, complexity.

Cleaner: MeshPhongMaterial with map=day, bumpMap=topology, directional light = sun (animated). Phong shades day side dark on night side automatically; then inject: add night lights where sun dot < 0 as emissive-ish addition. In fragment, after lighting computed, add nightColor * nightMask. That is the classic trick: replace `#include <emissivemap_fragment>`? Actually simplest injection point: at very end of main, before dithering: `gl_FragColor.rgb += nightColor * mask;`. In r160 phong fragment ends with:

```
#include <output_fragment>
#include <tonemapping_fragment>
#include <colorspace_fragment>
#include <fog_fragment>
#include <premultiplied_alpha_fragment>
#include <dithering_fragment>
```

In r160 output_fragment = `gl_FragColor = vec4( outgoingLight, diffuseColor.a );`. I can replace `#include <output_fragment>` with custom adding night contribution to outgoingLight:

```
vec4 nightColor = texture2D(uNightMap, vMapUv);
float ndotl = dot(normalize(vWorldNormal), normalize(uSunDir));
float nightMask = 1.0 - smoothstep(-0.25, 0.05, ndotl);
outgoingLight += nightColor.rgb * nightMask * 1.4;
gl_FragColor = vec4( outgoingLight, diffuseColor.a );
```

And directional light handles day shading. Night side lit only by ambient — set ambient low (0.08) so nights are dark, lights pop. 

Also day texture on night side gets ambient*small — fine, plus night lights added.

Bump map on phong uses vBumpMapUv etc. fine.

Tone mapping: renderer.toneMapping = ACESFilmic, exposure 1.0. Colors get nice. outputColorSpace default srgb in r160. Texture colorSpace = SRGBColorSpace for day/night maps.

Sun direction animate: 
```
const sunAngle = t*0.02; sunDir.set(Math.cos(sunAngle), 0.25, Math.sin(sunAngle)).normalize();
dirLight.position.copy(sunDir).multiplyScalar(5);
```
Tilt 0.25 y offset gives seasonal-ish variation. Terminator visible.

Now heat overlay canvas draw function:

```js
function drawHeat(year){
  const W=2048,H=1024; ctx.clearRect...
  if(!worldFeatures) return;
  for(const f of worldFeatures){
    const id = String(f.id).padStart(3,'0');
    const val = countryValue(id, currentMetric, year);
    if(val==null) { fill default very subtle } 
    const t = norm(val);
    ctx.fillStyle = colorFor(t, alpha 0.55);
    drawGeom(f.geometry);
    stroke
  }
  texture.needsUpdate = true;
}
```

colorFor: gradient stops array of [r,g,b], lerp.

Undefined countries: fill rgba(30,50,80,0.25) base so overlay still visible subtly. Good — gives coverage everywhere.

countryValue: if table has id → base * growth; else hash-based pseudo value scaled to metric's low range.

Normalization: precompute min/max over "defined" countries at current year using log10. norm = (log(v)-logMin)/(logMax-logMin), clamp.

Performance of drawHeat: ~250 features, each maybe 1-3 polygons with arcs totaling ~thousands of points; trivial.

drawGeom with antimeridian handling:

```js
function ring(ctx, coords){
  ctx.moveTo(...first);
  let prevLon = coords[0][0];
  for(let i=1;i<coords.length;i++){
    const [lon,lat]=coords[i];
    if(Math.abs(lon-prevLon)>180){ ctx.moveTo(x(lon),y(lat)); }
    else ctx.lineTo(x(lon),y(lat));
    prevLon=lon;
  }
  ctx.closePath();
}
```
Begin new path per polygon: for polygon: ctx.beginPath(); ring(exterior); for holes ring(hole); ctx.fill('evenodd'); ctx.stroke().

Hmm, stroke will draw seams at antimeridian cuts — acceptable.

City markers group; store per-city: sprite, hitMesh, data. Sprite scale updated per metric/year: s = 0.03 + 0.09*sqrt(v/vmax).

Sprite material: SpriteMaterial with glow texture, color per metric (GDP gold, pop cyan, flights magenta?), depthWrite false. Additive blending.

Hit mesh: SphereGeometry(0.035) invisible (material visible:false? Raycaster skips invisible? Raycaster tests meshes regardless of visibility? Mesh.raycast doesn't check visibility... Actually Raycaster intersectObject checks `object.visible`? No — Raycaster doesn't check visible; but intersectObjects with recursive... Material.visible false still raycasts. To be safe use opacity 0 transparent, or `visible=false` prevents raycast? THREE.Raycaster does intersect invisible objects? Looking at source: Mesh.raycast doesn't check this.visible. But `intersectObjects` iterates and calls raycast regardless of visibility. Hmm, actually in three r160, `intersectObject` checks `object.layers` but not visible. Wait — there is a check: in `intersect` function... I recall invisible objects ARE raycast (a common gotcha). Yes, invisible objects still raycast. So use visible=false? Actually no: common gotcha is that setting visible=false does NOT prevent raycasting... let me recall precisely. In three.js Raycaster, `intersectObject(object, recursive)` → `intersect(object)` → `object.raycast(raycaster, intersects)` — no visibility check. Confirmed gotcha exists. So I can just use visible = false? Hmm, if visible=false still raycasts, fine — that's what I want: invisible but raycastable. But careful: sprite raycast also possible. I'll use invisible spheres (visible=false) — wait, if visible false doesn't prevent raycast, great. But some versions added `if (object.visible === false) return` in intersectObject? Let me think... I believe in recent three (r150+), there's still no visibility check in Raycaster. Hmm, actually there IS: three.js docs for Raycaster: "Raycaster... ignores objects with visible = false"? I don't trust memory. Safer: use material.transparent=true, opacity=0, depthWrite=false, colorWrite? Use `new THREE.MeshBasicMaterial({visible:false})`? material.visible false — Mesh.raycast checks `if (material === undefined) return;` only. So any material works; use opacity 0 with depthWrite false to be safe. I'll do `transparent:true, opacity:0, depthWrite:false`.

Similarly route hit tubes: opacity 0 tubes radius 0.01, TubeGeometry(curve, 32, 0.012, 6, false). 24 routes × 32×6 segments — cheap.

Popup anchoring: on click, set active city; each frame if popup visible, project city world position (apply group matrix — group may rotate? we use OrbitControls camera movement only, group static; but keep general: getWorldPosition). If marker behind globe (dot(camDir, pos) < 0... if the marker's world pos is on far side, hide popup? Just clamp. Compute: vector.project(camera), to screen coords. Hide if z>1.

Also prevent popup when marker occluded: check normalized pos dot camera direction: occluded if dot(pos.clone().normalize(), camera.position.clone().normalize()) < 0.1 → hide.

Route tooltip: on pointermove, raycast tubes; show tooltip at mouse with route info for current year (flights scale by year: flights(year) = base * pow(1.03, year-2024)).

Click vs drag distinction: pointerdown record pos; pointerup if moved < 5px → click raycast cities.

Auto-rotate resume: controls.addEventListener('start', ...) set autoRotate false; 'end' → setTimeout 2500 → autoRotate true (if toggle enabled).

Timeline play: setInterval or in animate loop accumulate; advance year every 0.8s; loop.

Update on year/metric change: updateHeat(), updateMarkers(), updateLegend(), updatePopupContent if open.

Legend: canvas or CSS gradient bar + labels min/max with units. Build gradient CSS from stops.

Units/format:
- GDP: 万亿美元 (trillion). format: v>=1 ? v.toFixed(1)+' 万亿' : (v*1000).toFixed(0)+' 十亿'
- Pop: 亿 (100M): (v/100).toFixed(2)+' 亿' (data in millions)
- Flights country: 万架次： toFixed(0)
City: GDP 十亿美元， pop 百万， flights 架次/日.

Heat overlay material: MeshBasicMaterial({map: canvasTex, transparent:true, opacity:0.85, depthWrite:false}) on sphere 1.002. Also polygonOffset maybe. OK.

Also grid/graticule? skip.

Loading overlay: count of loads; hide when all done or errored. Use THREE.LoadingManager for textures; fetch for topojson separately; hide loader after Promise.allSettled.

Fallback texture if day texture fails: generate canvas 1024x512: ocean gradient blue, draw random continent blobs greenish/tan. Night fallback: black with scattered yellow dots. Bump fallback: grayscale noise. I'll implement makeFallbackEarth(night?).

Edge: importmap + topojson esm from unpkg — if offline, whole module fails (three fails too). Provide fallback CDN? Could add error message div if module load fails: window.addEventListener('error')... Keep simple; assume CDN reachable. Maybe use jsdelivr for everything as it's reliable:

- three: https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js
- addons path: https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/
- topojson-client: https://cdn.jsdelivr.net/npm/topojson-client@3/+esm
- textures: https://cdn.jsdelivr.net/npm/three-globe@2.31.0/example/img/earth-blue-marble.jpg — jsdelivr serves npm package files. three-globe npm package includes example/img? The npm package "three-globe" — does it ship example folder? I think yes, since unpkg.com/three-globe/example/img/... works, and jsdelivr mirrors npm. Use jsdelivr for textures too, or unpkg — either. I'll use unpkg for textures and data (commonly used), jsdelivr for libs. Mixed fine.

Actually to reduce risk, use unpkg consistently for textures: https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg etc. And world-atlas: https://unpkg.com/world-atlas@2/countries-110m.json.

Now write the full code carefully.

Key pieces:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D 全球数据可视化地球仪</title>
<style>...</style>
<script type="importmap">...</script>
</head>
<body>
<div id="app"><canvas id="scene"></canvas></div>
...UI...
<script type="module">
...code...
</script>
</body>
</html>
```

CSS: dark theme, glassmorphism panels (backdrop-filter), accent #4fc3f7.

Let me write data:

```js
const YEARS = {min:2000, max:2024};

const METRICS = {
  gdp: {name:'GDP', unit:'万亿美元', color:0xffc857, growth:0.03, fmt:v=> v>=1? v.toFixed(2)+' 万亿': Math.round(v*1000)+' 十亿'},
  pop: {name:'人口', unit:'百万人', color:0x4fc3f7, growth:0.011, fmt:v=> v>=100? (v/100).toFixed(2)+' 亿' : v.toFixed(1)+' 百万'},
  flights: {name:'航班量', unit:'万架次/年', color:0xf06292, growth:0.035, fmt:v=> Math.round(v)+' 万架次'}
};
```

Country data: ISO → {g: gdpT2024, p: popM2024, f: flights万2024, gg?, pg?, fg?} optional growth overrides.

```js
const COUNTRIES = {
 '840':{g:27.4,p:335,f:980},
 '156':{g:17.8,p:1412,f:520,gg:0.105,pg:0.004,fg:0.085},
 '392':{g:4.2,p:125,f:130,gg:0.005,pg:-0.002},
 '276':{g:4.5,p:84,f:190,gg:0.015,pg:0.002},
 '826':{g:3.3,p:67,f:200,gg:0.018},
 '356':{g:3.6,p:1420,f:110,gg:0.07,pg:0.011,fg:0.07},
 '250':{g:3.0,p:68,f:160,gg:0.014},
 '380':{g:2.2,p:59,f:140,gg:0.008},
 '076':{g:2.1,p:216,f:90,gg:0.02,pg:0.007},
 '124':{g:2.1,p:39,f:95,gg:0.02,pg:0.009},
 '643':{g:2.0,p:144,f:85,gg:0.01,pg:-0.001},
 '410':{g:1.7,p:52,f:70,gg:0.025,pg:0.001},
 '036':{g:1.7,p:26,f:75,gg:0.022,pg:0.012},
 '724':{g:1.6,p:48,f:170,gg:0.015},
 '484':{g:1.8,p:128,f:90,gg:0.02,pg:0.008},
 '360':{g:1.4,p:275,f:85,gg:0.045,pg:0.008},
 '528':{g:1.1,p:18,f:70,gg:0.018},
 '682':{g:1.1,p:37,f:60,gg:0.025,pg:0.018},
 '792':{g:1.1,p:85,f:180,gg:0.035,pg:0.01,fg:0.05},
 '756':{g:0.9,p:8.8,f:55,gg:0.015},
 '616':{g:0.8,p:38,f:45,gg:0.03},
 '032':{g:0.6,p:46,f:40,gg:0.01,pg:0.008},
 '710':{g:0.38,p:60,f:25,gg:0.015,pg:0.012},
 '818':{g:0.4,p:112,f:30,gg:0.04,pg:0.016},
 '566':{g:0.36,p:223,f:12,gg:0.03,pg:0.026},
 '784':{g:0.5,p:9.5,f:95,gg:0.035,pg:0.015,fg:0.06},
 '752':{g:0.6,p:10.5,f:40,gg:0.018},
 '702':{g:0.5,p:6,f:65,gg:0.04,pg:0.008,fg:0.045},
 '704':{g:0.43,p:98,f:35,gg:0.075,pg:0.009,fg:0.09},
 '764':{g:0.5,p:72,f:75,gg:0.03,pg:0.003},
 '458':{g:0.4,p:34,f:50,gg:0.04,pg:0.013},
 '608':{g:0.44,p:117,f:30,gg:0.05,pg:0.016},
 '586':{g:0.34,p:240,f:18,gg:0.035,pg:0.02},
 '050':{g:0.45,p:173,f:12,gg:0.06,pg:0.011},
 '364':{g:0.4,p:89,f:20,gg:0.025,pg:0.007},
 '398':{g:0.26,p:19,f:10,gg:0.03,pg:0.011},
 '170':{g:0.36,p:52,f:30,gg:0.03,pg:0.008},
 '152':{g:0.33,p:19.6,f:25,gg:0.025,pg:0.007},
 '604':{g:0.27,p:34,f:20,gg:0.035,pg:0.01},
 '372':{g:0.55,p:5.3,f:30,gg:0.04,pg:0.012},
 '620':{g:0.29,p:10.4,f:35,gg:0.015},
 '300':{g:0.24,p:10.4,f:25,gg:0.012},
 '040':{g:0.52,p:9.1,f:35,gg:0.016},
 '056':{g:0.63,p:11.7,f:40,gg:0.016},
 '208':{g:0.41,p:5.9,f:30,gg:0.017},
 '578':{g:0.58,p:5.5,f:30,gg:0.015},
 '246':{g:0.3,p:5.6,f:25,gg:0.015},
 '203':{g:0.33,p:10.5,f:20,gg:0.02},
 '376':{g:0.51,p:9.8,f:25,gg:0.03,pg:0.019},
 '554':{g:0.25,p:5.2,f:20,gg:0.02,pg:0.009},
 '012':{g:0.24,p:45,f:10,gg:0.02,pg:0.018},
 '504':{g:0.15,p:37,f:15,gg:0.03,pg:0.012},
 '404':{g:0.11,p:55,f:8,gg:0.045,pg:0.02},
 '231':{g:0.16,p:126,f:8,gg:0.08,pg:0.025},
 '834':{g:0.08,p:67,f:4,gg:0.05,pg:0.03},
 '800':{g:0.05,p:48,f:3,gg:0.05,pg:0.03},
 '024':{g:0.09,p:36,f:4,gg:0.03,pg:0.032},
 '180':{g:0.07,p:102,f:3,gg:0.035,pg:0.033},
 '144':{g:0.08,p:22,f:6,gg:0.03,pg:0.008}, // hmm 144 = Sri Lanka
 '724' dup? no. ok
};
```

Wait ISO numeric codes: Spain 724 ✓, Saudi 682 ✓, UAE 784 ✓, Sweden 752 ✓, Singapore 702 ✓, Vietnam 704 ✓, Thailand 764 ✓, Malaysia 458 ✓, Philippines 608 ✓, Pakistan 586 ✓, Bangladesh 050 ✓, Iran 364 ✓, Kazakhstan 398 ✓, Colombia 170 ✓, Chile 152 ✓, Peru 604 ✓, Ireland 372 ✓, Portugal 620 ✓, Greece 300 ✓, Austria 040 ✓, Belgium 056 ✓, Denmark 208 ✓, Norway 578 ✓, Finland 246 ✓, Czechia 203 ✓, Israel 376 ✓, New Zealand 554 ✓, Algeria 012 ✓, Morocco 504 ✓, Kenya 404 ✓, Ethiopia 231 ✓, Tanzania 834 ✓, Uganda 800 ✓, Angola 024 ✓, DR Congo 180 ✓, Sri Lanka 144 ✓. South Africa 710 ✓. Nigeria 566 ✓. Egypt 818 ✓. Argentina 032 ✓. Poland 616 ✓. Switzerland 756 ✓. Netherlands 528 ✓. Indonesia 360 ✓. Mexico 484 ✓. Australia 036 ✓. Korea 410 ✓. Russia 643 ✓. Canada 124 ✓. Brazil 076 ✓. Italy 380 ✓. France 250 ✓. India 356 ✓. UK 826 ✓. Germany 276 ✓. Japan 392 ✓. China 156 ✓. US 840 ✓. Turkey 792 ✓.

Also Greenland 304 (Denmark) leave default.

value function:
```js
function countryValue(id, metric, year){
  const d = COUNTRIES[id];
  const key = {gdp:'g',pop:'p',flights:'f'}[metric];
  const gkey = {gdp:'gg',pop:'pg',flights:'fg'}[metric];
  if(d){
    const base = d[key]; if(base==null) return null;
    const gr = d[gkey] ?? METRICS[metric].growth;
    return base * Math.pow(1+gr, year-2024);
  }
  // pseudo for unlisted countries
  let h=0; for(const c of id) h=(h*31 + c.charCodeAt(0))>>>0;
  const r = (h%1000)/1000; // 0..1
  const defaults = {gdp:0.03+0.12*r, pop:1+18*r, flights:2+13*r};
  return defaults[metric]*Math.pow(1+METRICS[metric].growth, year-2024);
}
```

Normalization per metric/year over all features present (use COUNTRIES keys + sample of others): simpler: fixed log ranges per metric:
gdp: log10 range [log10(0.02), log10(30)] — since values change over years (2000 China ~1.6T fine).
pop: [log10(0.5), log10(1500)]
flights: [log10(1), log10(1000)]
Fixed ranges keep colors stable across years — better for time animation. 

```js
const RANGES = {gdp:[0.02,30], pop:[0.5,1500], flights:[1,1100]};
function normVal(m,v){ const [a,b]=RANGES[m]; const t=(Math.log10(v)-Math.log10(a))/(Math.log10(b)-Math.log10(a)); return Math.min(1,Math.max(0,t)); }
```

Gradient stops (t→rgb): 
```js
const STOPS = [
 [0.0, [13,32,68]],
 [0.25,[14,84,120]],
 [0.45,[16,140,130]],
 [0.62,[80,170,90]],
 [0.78,[210,190,70]],
 [0.9, [235,120,50]],
 [1.0, [214,40,57]]
];
```

Cities data:

```js
const CITIES = [
 {name:'纽约', en:'New York', country:'美国', lat:40.71, lon:-74.00, g:2100, p:19.5, f:1200},
 {name:'洛杉矶', en:'Los Angeles', country:'美国', lat:34.05, lon:-118.24, g:1200, p:12.5, f:1100},
 {name:'芝加哥', en:'Chicago', country:'美国', lat:41.88, lon:-87.63, g:750, p:8.9, f:1400},
 {name:'多伦多', en:'Toronto', country:'加拿大', lat:43.65, lon:-79.38, g:400, p:6.4, f:700},
 {name:'墨西哥城', en:'Mexico City', country:'墨西哥', lat:19.43, lon:-99.13, g:400, p:22.3, f:600},
 {name:'圣保罗', en:'São Paulo', country:'巴西', lat:-23.55, lon:-46.63, g:500, p:22.6, f:700},
 {name:'伦敦', en:'London', country:'英国', lat:51.50, lon:-0.12, g:900, p:14.3, f:1100},
 {name:'巴黎', en:'Paris', country:'法国', lat:48.85, lon:2.35, g:850, p:11.0, f:950},
 {name:'法兰克福', en:'Frankfurt', country:'德国', lat:50.11, lon:8.68, g:300, p:5.8, f:800},
 {name:'莫斯科', en:'Moscow', country:'俄罗斯', lat:55.76, lon:37.62, g:450, p:12.5, f:650},
 {name:'伊斯坦布尔', en:'Istanbul', country:'土耳其', lat:41.01, lon:28.98, g:300, p:15.8, f:900},
 {name:'开罗', en:'Cairo', country:'埃及', lat:30.04, lon:31.24, g:150, p:21.8, f:350},
 {name:'约翰内斯堡', en:'Johannesburg', country:'南非', lat:-26.20, lon:28.05, g:120, p:6.2, f:300},
 {name:'迪拜', en:'Dubai', country:'阿联酋', lat:25.20, lon:55.27, g:130, p:3.6, f:900, gg:0.06, fg:0.07, pg:0.03},
 {name:'孟买', en:'Mumbai', country:'印度', lat:19.08, lon:72.88, g:350, p:21.3, f:550, gg:0.06},
 {name:'德里', en:'Delhi', country:'印度', lat:28.61, lon:77.21, g:330, p:32.9, f:700, gg:0.06},
 {name:'新加坡', en:'Singapore', country:'新加坡', lat:1.35, lon:103.82, g:420, p:6.0, f:650, gg:0.04},
 {name:'曼谷', en:'Bangkok', country:'泰国', lat:13.76, lon:100.50, g:250, p:11.0, f:550},
 {name:'香港', en:'Hong Kong', country:'中国', lat:22.32, lon:114.17, g:400, p:7.5, f:700, gg:0.03},
 {name:'北京', en:'Beijing', country:'中国', lat:39.90, lon:116.40, g:680, p:21.5, f:900, gg:0.09, fg:0.08},
 {name:'上海', en:'Shanghai', country:'中国', lat:31.23, lon:121.47, g:700, p:29.0, f:800, gg:0.09, fg:0.08},
 {name:'首尔', en:'Seoul', country:'韩国', lat:37.57, lon:126.98, g:800, p:26.0, f:700, gg:0.025},
 {name:'东京', en:'Tokyo', country:'日本', lat:35.68, lon:139.69, g:1800, p:37.4, f:900, gg:0.005},
 {name:'悉尼', en:'Sydney', country:'澳大利亚', lat:-33.87, lon:151.21, g:400, p:5.3, f:500},
];
```

cityValue similar with keys g/p/f meaning GDP十亿， pop百万， flights架次/日.

Routes:
```js
const ROUTES = [
 ['纽约','伦敦',28],['纽约','巴黎',12],['纽约','东京',8],['芝加哥','纽约',30],
 ['洛杉矶','东京',10],['洛杉矶','墨西哥城',12],['悉尼','洛杉矶',6],
 ['伦敦','迪拜',14],['伦敦','新加坡',9],['伦敦','多伦多',8],
 ['巴黎','开罗',6],['法兰克福','北京',7],['法兰克福','伊斯坦布尔',9],
 ['莫斯科','伊斯坦布尔',10],['伊斯坦布尔','迪拜',8],
 ['迪拜','孟买',18],['迪拜','德里',12],['迪拜','约翰内斯堡',5],
 ['东京','上海',15],['东京','香港',16],['上海','新加坡',12],
 ['香港','新加坡',14],['新加坡','悉尼',10],['曼谷','新加坡',15],
 ['首尔','上海',18],['圣保罗','墨西哥城',8],['北京','新加坡',10],
];
```
29 routes fine.

Now rendering details.

Scene setup:

```js
const renderer = new THREE.WebGLRenderer({canvas, antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.toneMapping = THREE.ACESFilmicToneMapping;
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(42, ar, 0.1, 100);
camera.position.set(0, 0.6, 3.4)? position (0.5,0.8,2.8) nice angle.
controls = new OrbitControls(camera, canvas); enableDamping .06; minDistance 1.6; maxDistance 8; autoRotate true speed 0.6; enablePan false.
```

Globe group with axial tilt: group.rotation.z = 0.2? Tilt around Z? Earth tilt 23.4° — apply group.rotation.x? If autoRotate orbits camera around Y, tilt of globe group adds realism: globeGroup.rotation.z = THREE.MathUtils.degToRad(23.4)*0.4 subtle? But markers/labels inside group tilt too — fine. Actually OrbitControls rotates camera around target; globe tilt static — looks like tilted earth. Nice. But terminator from sun with y offset plus tilt may look odd; keep small tilt 0.18 rad on z.

Hmm wait — OrbitControls autoRotate rotates camera azimuthally. Globe group static tilted. Markers attached to globe group. Popup projection uses world positions — fine.

Earth mesh: SphereGeometry(1, 96, 96)? 64 enough. Use 128 for smoothness? 96 ok.

Textures:
```js
const texLoader = new THREE.TextureLoader(manager);
function loadTex(url, srgb=true){ const t = texLoader.load(url); t.colorSpace = srgb? THREE.SRGBColorSpace : THREE.NoColorSpace; t.anisotropy = renderer.capabilities.getMaxAnisotropy(); return t; }
```
With error fallback: TextureLoader.load(url, onload, undefined, onerr) — set fallback manually in onerr by generating canvas texture and assigning to material.map, needsUpdate.

Simpler robust approach: use async loadTexture wrapper returning promise resolving to texture (or fallback on error):

```js
function loadTextureAsync(url, fallbackFn, srgb){
  return new Promise(res=>{
    new THREE.TextureLoader().load(url,
      t=>{prep(t);res(t);},
      undefined,
      ()=>{const t=new THREE.CanvasTexture(fallbackFn());prep(t);res(t);});
  });
}
```

Fallback earth day canvas (1024x512): fill ocean radial-ish gradient, draw ~14 random continent blobs with bezier... decent: use noise-ish circles cluster. I'll draw simple: ocean #0b3d66→#06264a gradient vertical; continents: random seeded blobs in green/tan with darker edges. Night: black #010409 with scattered warm dots concentrated near blob areas? Too complex; just scatter dots globally weighted to mid latitudes.

OK.

Earth material:

```js
const earthMat = new THREE.MeshPhongMaterial({
  map: dayTex, bumpMap: topoTex, bumpScale: 0.05,
  specular: new THREE.Color(0x333333), shininess: 12,
});
earthMat.onBeforeCompile = shader => { ...night mix... };
```

Ambient light 0x223344 intensity 0.5? AmbientLight(0x334455, 0.6). Night side would show day texture dimly + night lights added. Might wash nights; set ambient 0.35 and night boost 1.6.

Directional light intensity 2.2 color 0xfff2e0.

With ACES tone mapping, may need higher intensities; r160 default useLegacyLights=false? In r155+, default is physically correct (useLegacyLights default false), light intensities: directional ~2-3 works with tone mapping.

Night mix injection (also slight day darken at terminator): outgoingLight += night * mask.

Uniform sun: keep object `const sunDir = {value:new THREE.Vector3(1,0.3,0.5).normalize()};` and reference in shader.uniforms.uSun = sunDir — shared object so updates propagate.

Atmosphere shaders as planned; plus inner rim shader:

```js
const rimMat = new THREE.ShaderMaterial({
 vertexShader: `varying vec3 vN; varying vec3 vP; void main(){ vN=normalize(normalMatrix*normal); vec4 mv=modelViewMatrix*vec4(position,1.); vP=mv.xyz; gl_Position=projectionMatrix*mv;}`,
 fragmentShader:`varying vec3 vN; varying vec3 vP; void main(){ float f=pow(1.0-abs(dot(normalize(vN),normalize(-vP))),3.0); gl_FragColor=vec4(vec3(0.35,0.6,1.0)*f*0.9, f*0.9);}`,
 transparent:true, blending:THREE.AdditiveBlending, depthWrite:false});
sphere radius 1.02.
```

Outer glow radius 1.18 backside.

Stars:

```js
const starGeo = new THREE.BufferGeometry();
positions 3000 on sphere radius 40-60 random; colors slight variation.
PointsMaterial({size:1.2, sizeAttenuation:false, vertexColors:true, transparent, opacity:.9, depthWrite:false})
```

City glow texture: canvas 64: radial gradient white center → transparent. Sprite per city: scale uniform then set scalar.

Also small dot mesh at city (circle)? Sprite suffices, plus additive blending.

City label? skip (popup shows names).

Marker pulse animation: subtle scale breathing: s *= 1+0.08*sin(t*2+i).

City pick logic & popup content: show name, country, year, current metric value + other two metrics values small.

Routes building:

```js
routesGroup; for each route: curve; line geo; line mat color 0x64d8ff? Color by metric? Keep cyan-orange: from color 0x4fc3f7 to gradient? LineBasicMaterial single color; use vertexColors gradient along line: create colors attribute lerp between two colors (start #ffd166, end #4fc3f7)? Pretty. Use LineBasicMaterial({vertexColors:true, transparent:true, opacity:0.55, blending:Additive, depthWrite:false}).
Pulses: 2 sprites per route orange.
Hit tube: Mesh(TubeGeometry(curve, 40, 0.012, 6), invisibleMat).
```

Pulse speed: base speed 0.12 + flights/200; direction random. In animate: t=(t+dt*speed)%1; pos=curve.getPoint(t). Also scale pulse by year factor: flights(year)/flights(2024).

Line opacity could scale with flights too: opacity 0.25+0.5*f/maxf.

Heat texture: CanvasTexture; colorSpace SRGB; anisotropy. Sphere radius 1.003. Render order after earth. transparent.

Also borders on heat canvas: stroke rgba(255,255,255,0.18) lineWidth 1.

Draw heat also fills unlisted countries with subtle base.

Update strategy: drawHeat(metric, year) called on changes; during play each tick.

Slider input → year change → updateAll.

Legend gradient CSS: build from STOPS: `linear-gradient(90deg, rgb(...), ...)`.

Min/max labels: RANGES per metric formatted.

Play button: toggles playing; in rAF loop if playing, acc+=dt; if acc>0.9 {acc=0; year++; if year>max year=min; update}.

Now the popup anchoring with tilted group: get world position via sprite.getWorldPosition.

Occlusion check: camera direction vs marker normal: marker world pos normalized dot (camera.position normalized) — globe center at origin. If dot < 0.12 → behind → hide popup (add class). Similarly hover route tooltip fine anywhere.

Pointer events: canvas pointermove → update mouse ndc; raycast routes on throttle (every frame ok). pointerdown store x,y; pointerup: if dist<6 → raycast cities (hitSpheres) → open popup or close if clicked empty space.

Raycaster.params.Line not needed since tubes.

Cursor: pointer when hovering city/route.

Handle resize.

Loading overlay: hide after Promise.allSettled([dayT, nightT, topoT, topojsonFetch]) then init-dependent parts (heat needs geojson maybe null).

If geojson fetch fails: heatLayer hidden, note in panel "热力数据加载失败".

Order of building: create everything after textures ready? We can create materials with fallback textures immediately then swap. Simpler: await Promise.allSettled then build scene with whatever we got. Show loader first. Build after load — fine since loader quick.

topojson import: static import — if network blocked entirely, nothing works anyway. Keep static.

world-atlas fetch:
```js
async function loadWorld(){
 try{ const r= await fetch('https://unpkg.com/world-atlas@2/countries-110m.json'); const topo = await r.json(); const fc = topojson.feature(topo, topo.objects.countries); return fc.features; }catch(e){ return null; }
}
```

Note: feature ids: f.id may be number or string; pad: String(f.id).padStart(3,'0')? ISO numeric in table like '076' for Brazil — world-atlas uses '076'? ids in world-atlas countries-110m are strings of numeric codes without leading zeros? Let me recall: properties.name exists; id is like "004" (Afghanistan)? I believe ids are zero-padded 3-digit strings ("004","840","076"). Hmm, in world-atlas v2 countries-110m.json, ids are strings like "004"? Looking at examples: us-atlas uses FIPS unpadded numbers. world-atlas... In observable examples, they do `data.get(d.id)` with names keyed... I recall ids like "010" (Antarctica), "156" etc. Actually world-atlas README: "countries-110m.json ... each feature has an id property representing the ISO 3166-1 numeric code". Values might be strings with padding ("004") — TopoJSON generated from Natural Earth with iso_n3. I've seen code `d.id === "010"`? Not certain. Handle both: normalize id by stripping leading zeros for lookup key? My table uses padded '076','050','032' etc. Normalize: key = String(parseInt(f.id,10)) and store table with unpadded keys. Safer: COUNTRIES keyed unpadded: '840','156','76','50','32','818','566','710','12','24','144','180','231','404','504','834','800','364','398','170','152','604','372','620','300','40','56','208','578','246','203','376','554','792','682','784','752','702','704','764','458','608','586','360','484','724','36','410','643','124','380','250','356','826','276','392','528','616','756'. And lookup `COUNTRIES[String(parseInt(id,10))]`. Also handle f.id undefined/-99 (Kosovo, Somaliland, N. Cyprus, France? France in 110m: id '250'? There's a known quirk: France id is '-99'?? In Natural Earth 110m, France's ISO_N3 was -99 due to Norway issue in older versions; world-atlas v2 fixed? I'll map '-99' with name check: if properties.name==='France' → '250'; 'Norway'→'578'; 'Kosovo'→'792'? no, give pseudo. I'll add name fixes for France/Norway.)

Actually let me add: NAME_FIX = {'France':'250','Norway':'578','Somaliland':'706'?} just France & Norway.

Pseudo-default handles missing anyway.

Time to also consider: heat canvas performance when dragging timeline — redraw per change fine.

City hit spheres list; route hit tubes list.

Info panel content: title, metric switch buttons, toggles (checkboxes: 自动旋转 ✓, 热力图层 ✓, 航线 ✓, 城市标记 ✓), legend, data source note.

Also show current metric description.

Bottom bar: play/pause button (▶/⏸), year big display, range slider 2000-2024 step1, speed maybe fixed.

Hint panel top-right with instructions.

Popup: styled card with close ×, city name + en, country, metric rows with bars maybe. Keep rows: 指标 values for current year: GDP xx 十亿美元， 人口 xx 百万， 航班 xx 架次/日. Highlight active metric. Small sparkline? skip.

Tooltip: "纽约 → 伦敦｜2024年 日均 28 班" plus scaled by year: flights(year)=round(28*pow(1.025, year-2024)).

Now write the code fully. Careful with sizes; it'll be long (~700-900 lines). Let's write.

Also make sure SphereGeometry UV seam with canvas texture: fine.

Antimeridian: arcs in topojson may include stitched points; fine with jump check.

One more: heat overlay should rotate with globe group (add to globeGroup) ✓. Markers & arcs also in globeGroup. Sun/dir light in scene (not group).

Terminator visible when camera autorotates ✓.

City lights especially night — night texture does it ✓ plus our night boost.

Compute sun also from timeline year? Keep continuous animation.

Let me write HTML/CSS.

Panels: position fixed; left top panel width 250px; bottom center timeline; top-right hint; popup absolute; tooltip fixed pointer-events none.

Font: system + "PingFang SC","Microsoft YaHei".

Colors: bg #04070f; panel rgba(10,18,32,0.72); border rgba(120,180,255,0.18); accent #4fc3f7.

Metric buttons: three buttons inline; active state accent bg.

Legend gradient bar height 10px with rounded, labels below.

Loading overlay full screen with spinner (CSS) and text "正在加载地球纹理与数据…"; fade out.

Failure notice div hidden.

Now the JS. Write carefully with all pieces. Also FPS-friendly: raycast on pointermove only.

Potential pitfall: Sprite raycast not needed; we use hit spheres.

getWorldPosition for sprite inside group with tilt — fine.

Popup update each frame: project worldPos: 
```js
const v = cityWorld.clone().project(camera);
if(v.z>1 || v.z<-1){hide}else{ x=(v.x*0.5+0.5)*w; y=(-v.y*0.5+0.5)*h; popup.style.transform=translate(-50%,-100%) translate(x,y-14px) }
```
Use left/top instead of transform for simplicity with translate(-50%,-115%).

Occlusion: 
```js
const camDir = camera.position.clone().normalize();
const facing = cityWorld.clone().normalize().dot(camDir);
if(facing < 0.15) popup hide
```
Note globe group static so world pos stable except none. Also when camera orbits (autorotate), facing changes — popup hides when rotated away. Good.

City marker base scale per metric: compute vMax across cities for metric at 2024 fixed (gdp max 2100, pop 37.4, flights 1400). Use fixed max per metric for stable sizes across years:
CITY_MAX = {gdp:2100, pop:38, flights:1400}. scale = 0.05 + 0.11*sqrt(v/max) (world units sprite). Sprite scale in world units; at distance 3.4, 0.16 world ≈ decent.

Breathing: multiply (1+0.07*sin(t*3+phase)).

Also add small white core dot sprite? One sprite enough with strong center.

Route pulse sprite scale 0.035.

Pulse count per route: 1 + (flights>=12?1:0) + (flights>=20?1:0) max 3.

Pulse color 0xffb347.

Route line gradient colors: from #ff9a3c (origin) to #3ec6ff? Actually symmetrical routes; gradient just aesthetic: c1 #ffd166 c2 #4ecdc4? Use per-vertex lerp along the 64 points.

Hover highlight: store hovered route; on hover set line material opacity 1 & tube cursor; others dim? Simple: hovered line material.color set white / opacity 0.95; restore after. Since materials shared? Create per-route materials (29 lines, fine).

Update route visual per year: opacity = 0.2+0.6*(flightsYear/30 capped).

Now writing updateHeat():

```js
function drawHeat(){
  const ctx=heatCtx, W=2048,H=1024;
  ctx.clearRect(0,0,W,H);
  if(!worldFeatures){ heatTex.needsUpdate=true; return; }
  const xOf=lon=>(lon+180)/360*W, yOf=lat=>(90-lat)/180*H;
  for(const f of worldFeatures){
    let id = f.id!=null ? String(parseInt(f.id,10)) : null;
    if((id==null||id==='NaN'||f.id===-99||f.id==='-99') && f.properties && NAME_FIX[f.properties.name]) id=String(parseInt(NAME_FIX[f.properties.name],10));
    const val = id? countryValue(id,currentMetric,currentYear):defaultVal...
```

Wait countryValue handles unlisted via hash of id string; if id null → use hash of name? For null id use generic pseudo. Simplify: idKey = id ?? ('x'+hash(name)). countryValue(idKey...) where hash works on string. parseInt('-99') → -99 → String → '-99' → hash fine, gives pseudo. OK simpler: idKey = f.id!=null? String(parseInt(f.id,10)) : ('n'+name). NAME_FIX for France/Norway.

Then:
```js
    const t = normVal(metric, val);
    ctx.fillStyle = rgba(colorAt(t), 0.5+0.25*t);
    draw geometry
    ctx.strokeStyle='rgba(180,220,255,0.25)'; ctx.lineWidth=0.6; stroke
```

Path drawing with 'evenodd' fill.

Function drawPolygon(rings): beginPath; for each ring: moveTo first; iterate with jump handling; closePath. Then fill('evenodd') & stroke per feature (all its polygons in one path? beginPath per feature, add all polygons as subpaths, fill evenodd, stroke once). Strokes holes too — fine.

heatTex.needsUpdate = true.

Canvas 2048×1024 RGBA fine.

Also heat sphere slightly larger radius 1.004 to avoid z-fight with markers at 1.005? Markers at 1.01. Arcs radius up to ~1.3. OK.

Heat material: transparent, opacity 0.9, depthWrite false, side FrontSide.

renderOrder: earth 0, heat 1, arcs 2, markers 3, atmosphere 4 (atmo additive backside drawn after, depthTest true for backside? backside sphere depthTest true fine).

Atmo outer: depthWrite false, side BackSide.

Now sun animation: 
```js
sunTime += dt*0.05; const sa=sunTime; sunDir.value.set(Math.cos(sa), 0.35*Math.sin(sa*0.7)+0.05, Math.sin(sa)).normalize();
dirLight.position.copy(sunDir.value).multiplyScalar(6);
```
Full day cycle ~2min at 0.05 rad/s → 126s. Good.

DirectionalLight target default origin ✓ (add light.target to scene? default target at (0,0,0) but must add target to scene for matrix updates—three handles default target internally? DirectionalLight.target default Object3D at origin not in scene; its matrixWorld stays identity → position (0,0,0) works. Fine.)

Stars subtle rotation: stars.rotation.y += dt*0.005.

Controls autoRotate & damping in loop: controls.update().

Pause autorotate on pointerdown (controls 'start'), resume timer on 'end'.

Toggle checkbox 自动旋转 → autoRotateEnabled flag.

Toggle 热力图层 → heatMesh.visible; 航线 → routesGroup.visible; 城市 → markersGroup.visible.

Metric switch: currentMetric; update heat, markers sizes/colors (sprite material color set per metric: gdp 0xffc857, pop 0x4fc3f7, flights 0xf06292 — set sprite material color), legend, popup content refresh.

Sprites materials per city (24) fine.

Legend min/max text: RANGES formatted per metric fmt.

Now popup content builder:

```js
function cityPopupHTML(c){
 return `<div class="pop-head"><div><div class="pop-title">${c.name} <span class="en">${c.en}</span></div><div class="pop-sub">${c.country} · ${currentYear}年</div></div><button class="pop-close">×</button></div>
 <div class="pop-rows">${METRIC_KEYS.map(k=>row(c,k)).join('')}</div>`;
}
row: value = cityValue(c,k,year); bar width = sqrt(v/CITY_MAX[k])*100... show label, value formatted with city units: gdp: v.toFixed(0)+' 十亿美元'; pop: v.toFixed(1)+' 百万'; flights: Math.round(v)+' 架次/日'.
active row highlighted.
```

Close button event delegation.

Tooltip content: route from→to names, current year flights/day scaled: `Math.round(base*Math.pow(1.025,year-2024))`.

Alright — also initial camera: position.set(1.8, 0.9, 2.6) looking origin.

Canvas sizing: full window.

Write CSS nicely.

Also add metric description text under buttons.

Edge: WebGL context creation fail — try/catch show message.

Time to write the final file. I'll aim for clean, complete code.

Additional check — `topojson-client` ESM from jsdelivr `+esm` returns module with named exports `feature`. Use `import { feature as topoFeature } from 'topojson-client';`.

Import map:
```json
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
    "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm"
  }
}
```

Textures via jsdelivr npm three-globe: does the three-globe package on npm include example/img? Package files... three-globe package.json "files"? If they don't whitelist, npm includes everything except .npmignore'd. unpkg.com/three-globe/example/img/earth-night.jpg is widely used and works → files included in npm tarball → jsdelivr mirrors npm → https://cdn.jsdelivr.net/npm/three-globe@2.31.0/example/img/earth-night.jpg works. Use jsdelivr consistently. world-atlas: https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json.

Fallback textures via canvas anyway.

Let me also double check MeshPhongMaterial output_fragment replace string exists in r160: yes `#include <output_fragment>` in meshphysical/ phong? meshphong fragment includes:
```
#include <output_fragment>
```
Yes (renamed from encodings... output_fragment still present in r160? In r162 they removed output_fragment... In r160 it exists. Hmm — risk. In r154+ "encodings_fragment" renamed "colorspace_fragment"; output_fragment remained until WebGLPrograms... r161/r162 changed gl_FragColor handling? Let me recall: output_fragment chunk existed through r162? I know by r165 phong still has `#include <opaque_fragment>`?? — that rename (output_fragment → opaque_fragment) happened in r162 or r163? I believe `opaque_fragment` rename occurred in r163 ("output_fragment" renamed). Not 100% sure.

To be safe, inject after `#include <lights_fragment_end>` or modify via a different anchor stable across versions: `#include <aomap_fragment>`? Safer approach: append to fragmentShader's main by replacing `#include <fog_fragment>` (exists long-standing) — but our addition must come before tonemapping/colorspace conversions (which happen after fog? order in r160 phong main:

```
#include <output_fragment>
#include <tonemapping_fragment>
#include <colorspace_fragment>
#include <fog_fragment>
#include <premultiplied_alpha_fragment>
#include <dithering_fragment>
```

So replacing fog_fragment would be after colorspace — wrong place (values already sRGB encoded; adding linear-ish night color would be off but visually maybe acceptable... better do it right).

Alternative robust anchor: replace `#include <lights_fragment_end>`? That is before output, and outgoingLight defined later in `lights_fragment_maps`? Structure: 

```
vec3 outgoingLight = reflectedLight.directDiffuse + reflectedLight.indirectDiffuse + totalEmissiveRadiance;
#include <envmap_fragment>
#include <output_fragment>
```

Hmm outgoingLight is defined after lights chunks in phong? meshphong_frag main:
```
#include <clipping_planes_fragment>
vec4 diffuseColor = vec4( diffuse, opacity );
ReflectedLight reflectedLight = ...
vec3 totalEmissiveRadiance = emissive;
#include <logdepthbuf_fragment>
#include <map_fragment>
#include <color_fragment>
#include <alphamap_fragment>
#include <alphatest_fragment>
#include <alphahash_fragment>
#include <roughnessmap_fragment>
#include <metalnessmap_fragment>
#include <normal_fragment_begin>
#include <normal_fragment_maps>
#include <emissivemap_fragment>
#include <lights_physical_fragment>?? no phong uses lights_phong_fragment
#include <lights_phong_fragment>
#include <lights_fragment_begin>
#include <lights_fragment_maps>
#include <lights_fragment_end>
#include <aomap_fragment>
vec3 outgoingLight = reflectedLight.directDiffuse + reflectedLight.indirectDiffuse + totalEmissiveRadiance;
#include <envmap_fragment>
#include <output_fragment>
```

So anchor on `vec3 outgoingLight = reflectedLight.directDiffuse + reflectedLight.indirectDiffuse + totalEmissiveRadiance;`? That's stable across versions for phong. Replace with itself + night addition after it. That string is stable r150→r16x. I'll use `.replace('vec3 outgoingLight = reflectedLight.directDiffuse + reflectedLight.indirectDiffuse + totalEmissiveRadiance;', same + '\n' + nightCode)`. If replace fails (not found), night mixing silently absent — acceptable degradation; but also do a fallback: if shader.fragmentShader.includes check, else append before output_fragment if exists. I'll do two-step: try outgoingLight anchor; if unchanged, try output_fragment anchor. Good.

nightCode:
```glsl
{
  vec3 wN = normalize(vWorldNormal);
  float sd = dot(wN, normalize(uSunDir));
  float nightMask = 1.0 - smoothstep(-0.25, 0.12, sd);
  vec3 nightCol = texture2D(uNightMap, vMapUv).rgb;
  outgoingLight += nightCol * nightMask * 1.5;
}
```
vMapUv available since USE_MAP defined. Need `varying vec3 vWorldNormal;` added to both shaders and assigned in vertex after defaultnormal_vertex (objectNormal exists there? defaultnormal_vertex uses transformedNormal; objectNormal defined in beginnormal_vertex before). Use `vWorldNormal = normalize(mat3(modelMatrix) * objectNormal);` after `#include <defaultnormal_vertex>`.

Since earth mesh has no rotation (group tilt only, modelMatrix includes it — correct world normal) ✓.

Uniforms: uSunDir (shared {value:Vector3}), uNightMap.

Also directional light from same dir gives Phong shading; ambient small.

Bump: bumpMap topology (grayscale). colorSpace NoColorSpace (linear). bumpScale 0.04.

Now, markers raycast: pointer NDC from event with canvas bounds.

Also hover cities → cursor pointer & slight enlarge? Do cursor change on city hit test during pointermove (raycast cities + routes).

Raycast cost fine.

Play button char: use ▶ / ❚❚.

Slider styling minimal.

Also year display big in bottom bar and also maybe on panel. fine.

Let me now also implement "根据年份更新航线流量" — route opacity/pulse speed by year ✓ and tooltip value.

City sizes by year ✓.

Heat by year ✓.

Popup values by year ✓ (refresh content on year change if open).

Write final code now. Also include a small FPS-friendly guard: if document.hidden skip heavy.

Also handle WebGL fail with try/catch → overlay message.

Let me count glow texture creation:

```js
function makeGlowTexture(){
 const c=document.createElement('canvas'); c.width=c.height=128;
 const g=c.getContext('2d');
 const grad=g.createRadialGradient(64,64,0,64,64,64);
 grad.addColorStop(0,'rgba(255,255,255,1)');
 grad.addColorStop(0.25,'rgba(255,255,255,0.85)');
 grad.addColorStop(0.5,'rgba(255,255,255,0.25)');
 grad.addColorStop(1,'rgba(255,255,255,0)');
 g.fillStyle=grad; g.fillRect(0,0,128,128);
 const t=new THREE.CanvasTexture(c); return t;
}
```
SpriteMaterial color tints it.

Core dot: also add tiny white sphere mesh per city? Sprite with strong center suffices.

Now assemble everything. Also `controls.target.set(0,0,0)`.

For mobile: touch works via OrbitControls; popups via tap (pointerup). OK.

CSS details — write once.

One more consideration: heat overlay drawn with alpha; underlying earth visible — combined look good. Also at night side heat overlay dims (MeshBasicMaterial ignores lights!) — heat would glow at night unrealistically. Better make heat material Lambert? Lambert with transparent canvas — gets directional shading & ambient, dark at night. But then day side colors slightly shaded — acceptable and more realistic (terminator affects heat too). However Lambert lighting also multiplies by diffuse lighting color — fine. But blending of transparent Lambert over earth: okay.

But Lambert + our custom? Not needed custom. Use MeshLambertMaterial({map:heatTex, transparent:true, opacity:0.85, depthWrite:false}). Night side → lit only by ambient 0.35*color... slightly visible. Reduce ambient effect? It's fine.

Hmm but emissive-ish pop desired? Keep Lambert; realistic.

Actually wait: MeshBasic vs Lambert: Basic looks like glowing data layer (common in these dashboards) but ignoring night looks wrong with terminator showcase. Choose Lambert — integrates with day/night. Good.

depthWrite false + transparent → renders after earth (transparent pass sorted by distance — both at ~same center distance; sorting by z of object position: earth center (0,0,0) same as heat... three sorts transparent objects by depth; ties → renderOrder decides. Set heatMesh.renderOrder=1, arcs renderOrder=2, sprites renderOrder 3, atmo renderOrder 4. Earth is opaque (rendered first anyway).)

Atmo additive BackSide with depthTest true — will occlude correctly behind earth front? BackSide sphere behind earth: depth test against earth's written depth ✓ glow appears around edges. Front part of backside sphere is farther than earth front surface → fails depth → only halo outside silhouette visible... Actually classic atmo: backside sphere radius 1.18: its front-facing-away surface in front of earth (between camera and earth) is closer than earth → passes depth, covering earth with additive tint... that's why intensity formula uses dot(vNormal, view) with BackSide normals flipped... The standard shader: intensity = pow(0.75 - dot(vNormal, vec3(0,0,1)), 2.0) where vNormal for backside faces point inward-ish... Known-good shader (from atmosphere glow tutorials):

vertex: `varying vec3 vNormal; void main(){ vNormal = normalize(normalMatrix * normal); gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0); }`
fragment: `varying vec3 vNormal; void main(){ float intensity = pow(0.65 - dot(vNormal, vec3(0.0,0.0,1.0)), 4.0); gl_FragColor = vec4(0.3,0.6,1.0,1.0) * intensity; }`
with side: BackSide, blending Additive. For backside, normalMatrix*normal gives flipped normals pointing toward viewer at center of disc → dot ≈ -1 at center → 0.65-(-1)=1.65 pow4 huge... hmm that gives bright center covering earth. In the classic example the atmosphere sphere is bigger and this yields rim bright at edge: at silhouette edge, normal perpendicular → dot≈0 → 0.65^4≈0.18; center: dot≈-1 → big → but those fragments are behind earth? No—backside sphere's far half is what renders; far half is behind earth (farther from camera) → depth-tested against earth → hidden behind globe, visible only outside silhouette where intensity moderate → creates halo. Yes! Since depthTest true and depth of far hemisphere > earth surface where earth covers it. So halo only outside edge. Classic and correct. Use radius 1.25, coefficient 0.6, pow 4, color (0.35,0.65,1.0)*intensity*1.5.

Also rim shell additive frontside for subtle limb brightening: skip? Keep small inner atmosphere: sphere 1.015 frontside with fresnel edge → nice blue rim on the globe edge. intensity = pow(1.0 - abs(dot(normal,viewDir)),3). Add color*0.6. Keep.

OK writing now. Also note OrbitControls import path.

Let me write the whole HTML mentally then output.

Also include meta description. Provide usage instructions after code: run via any static server or direct open (module scripts + fetch require http? fetch of CDN fine from file://? ES modules from CDN with file:// page: module scripts allowed cross-origin with CORS; file:// origin null — jsdelivr sends CORS * so ok. fetch to jsdelivr from null origin: CORS * allows null? Access-Control-Allow-Origin: * works with null origin. TextureLoader from null origin: image CORS * fine. So file:// double-click likely works in Chrome? ES module importmap on file:// works. But some browsers restrict module scripts on file://? Chrome allows remote module imports from file pages I think (it's https). Safer to recommend local server: `python -m http.server`. I'll mention both.

Deliverable: one HTML file + brief run/test notes. Since they asked to "直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明" — output the code block plus concise run/test instructions.

Now, write code. I'll be careful about every function. Also keep an eye on total length — big but fine.

Let me define hash fn:

```js
function hashStr(s){let h=2166136261;for(let i=0;i<s.length;i++){h^=s.charCodeAt(i);h=Math.imul(h,16777619);}return (h>>>0)/4294967295;}
```

countryValue(idKey, metric, year): if COUNTRIES[idKey] use; else pseudo based on hashStr(idKey+metric).

default growth per metric: GDP 0.032, pop 0.012, flights 0.036.

City value similar; city growth overrides few.

METRIC keys: 'gdp','pop','flights'. In city objects keys g,p,f; map via K2={gdp:'g',pop:'p',flights:'f'}.

City fmt: gdp: `${v>=1000?(v/1000).toFixed(2)+' 万亿':v.toFixed(0)+' 十亿'}美元`; pop: `${v.toFixed(1)} 百万`; flights: `${Math.round(v)} 架次/日`.

Country legend fmt as METRICS fmt earlier.

Legend values: RANGES min/max formatted.

Slider min 2000 max 2024 value 2024.

Update functions:

```js
function refreshAll(){ updateHeat(); updateCities(); updateRoutes(); updateLegend(); refreshPopup(); updateYearLabel(); }
```

updateRoutes: line opacity by flights(year)/30; pulse speedFactor.

Pulses array: {sprite, curve, t, speed, route}.

animate loop: dt from clock; sun rotate; pulses advance (speed * globalYearFactor); sprite breathing; controls.update; popup reposition; render.

Route tooltip positioning: store lastMouse client; on hover show tooltip at client+14.

Also hide tooltip when none.

Now popup reposition each frame if activeCity.

City hit test function on click:

```js
raycaster.setFromCamera(ndc, camera);
const hits = raycaster.intersectObjects(cityHitMeshes, false);
if(hits.length){ openPopup(hits[0].object.userData.city); } else { closePopup(); }
```

Route hover:

```js
const rh = raycaster.intersectObjects(routeHitMeshes,false);
```

cityHitMeshes: Mesh sphere r=0.045? At globe scale with zoom min 1.6 that's clickable. Use 0.05.

Cursor pointer if either hover.

Performance: raycast against ~24 spheres + 29 tubes per pointermove — fine.

Heat drawing details with MultiPolygon:

```js
function drawFeature(ctx, geom, xOf, yOf){
 const polys = geom.type==='Polygon'? [geom.coordinates] : geom.type==='MultiPolygon'? geom.coordinates : [];
 ctx.beginPath();
 for(const rings of polys){
  for(const ring of rings){
   for(let i=0;i<ring.length;i++){
    const lon=ring[i][0], lat=ring[i][1];
    const x=xOf(lon), y=yOf(lat);
    if(i===0) ctx.moveTo(x,y);
    else { const plon=ring[i-1][0]; if(Math.abs(lon-plon)>180) ctx.moveTo(x,y); else ctx.lineTo(x,y); }
   }
   ctx.closePath();
  }
 }
 ctx.fill('evenodd'); ctx.stroke();
}
```

Antarctica: covers whole bottom; drawing with moveTo jump handling: the ring includes points (-180,-63)...(180,-63) then down? Natural Earth Antarctica polygon includes the pole via corner points (180,-90),(-180,-90). Fill covers bottom band — fine, gives Antarctic subtle color.

Also wrap-around fill issue: for Russia split, ok.

lineWidth 1 at 2048 wide → 0.5px lines on globe; use 1.

Heat texture min filter linear; generateMipmaps true default fine; colorSpace SRGB.

Also to avoid heat covering oceans too heavily with pseudo values for small islands: pseudo alpha low since t low → alpha 0.5+0.25t hmm base alpha 0.5 might be strong for oceans? Oceans get no fill (only land polygons). Small island nations get pseudo low → subtle. OK.

Wait: pseudo value for unlisted countries — hashStr(idKey+metric) idKey like '4' → fine.

Baseline fill for land with no data: t from pseudo (0.02-0.15 gdp → t≈0). alpha 0.5 minimum → all land tinted dark blue — nice.

updateCities():

```js
for(const c of CITIES){
 const v=cityValue(c,currentMetric,currentYear);
 const s=0.05+0.11*Math.sqrt(v/CITY_MAX[currentMetric]);
 c._baseScale=s; c.sprite.material.color.setHex(METRICS[currentMetric].color);
}
```
sprite scale set in animate with breathing.

City glow sprite material: SpriteMaterial({map:glowTex, color, transparent:true, depthWrite:false, blending:AdditiveBlending}). Additive sprites over bright day ocean may wash — fine.

Also add per-city core: small Mesh(SphereGeometry(0.006), MeshBasicMaterial({color:0xffffff})) at position — crisp dot. Add both to a per-city Group? Sprite at pos, core at pos. Add to markersGroup.

Route building colors: gradient from #ffb347 to #48c6ff.

```js
const pts=curve.getPoints(64); colors lerp.
```

Pulses color 0xffd166, scale 0.05? pulse sprite 0.045 world.

Hit tubes: 29 × TubeGeometry(curve,48,0.014,6) ≈ 29*48*6*2 tris ≈ 16k tris fine.

Route info: {from,to,base,curve,line,pulses:[],hitMesh}.

updateRoutes: 
```js
const f = Math.pow(1.025, currentYear-2024);
for r: const cur=r.base*f; r.line.material.opacity=Math.min(0.9,0.25+cur/35); r._speedFactor=0.5+cur/30; 
```

Tooltip: `${from} ⇄ ${to}<br>${year}年 日均约 ${Math.round(cur)} 班`.

Sun & terminator done.

Self-rotation: OrbitControls autoRotate true, speed 0.8.

Also I want slight globe "breathing" none. fine.

Now finalize code writing. Also add keyboard? no.

Panel content:

```
<h1>🌍 全球数据地球仪</h1>
<div class="seg"> buttons data-metric gdp/pop/flights: GDP / 人口 / 航班量</div>
<div class="metric-desc" id="metricDesc"></div>
<div class="legend"><div class="legend-bar" id="legendBar"></div><div class="legend-labels"><span id="legendMin"></span><span id="legendMax"></span></div></div>
<div class="toggles"> label*4 checkbox: 自动旋转/热力图层/航线/城市标记</div>
<div class="note">数据为示例模拟数据 · 拖拽旋转 · 滚轮缩放 · 点击城市查看详情 · 悬停航线查看航班</div>
```

Hint panel maybe merge into note; skip separate hint panel (note covers).

Bottom bar:

```
<button id="playBtn">▶</button>
<div class="year" id="yearLabel">2024</div>
<input type="range" id="yearSlider" min=2000 max=2024 step=1 value=2024>
<div class="tl-labels"><span>2000</span><span>2024</span></div>
```

Loading overlay + error toast.

CSS ~150 lines.

Let me write JS in order:

1. imports
2. constants & data
3. helpers (hash, latLon, colorAt, value funcs)
4. UI refs & state
5. three init
6. loaders & fallback textures
7. build scene (async main)
8. interactions
9. animate
10. UI wiring

Write fallback earth generator:

```js
function fallbackEarthCanvas(night){
 const W=1024,H=512,c=document.createElement('canvas');c.width=W;c.height=H;const g=c.getContext('2d');
 if(night){ g.fillStyle='#020409';g.fillRect(0,0,W,H);
   // city lights clusters
   for(let i=0;i<2500;i++){const x=Math.random()*W;const y=H*0.15+Math.random()*H*0.7; if(Math.random()<0.75&&!onFallbackLand(x/W,y/H))continue; g.fillStyle=`rgba(255,${200+Math.random()*55|0},120,${0.3+Math.random()*0.7})`; g.fillRect(x,y,1.5,1.5);} return c;}
 const grad=g.createLinearGradient(0,0,0,H); grad.addColorStop(0,'#0a2f55');grad.addColorStop(0.5,'#0d3a63');grad.addColorStop(1,'#0a2f55'); g.fillStyle=grad;g.fillRect(0,0,W,H);
 // continents blobs deterministic
 const rnd=mulberry32(42);
 for(const [cx,cy,nr] of CONT_BLOBS){ draw blobs }
}
```

Simplify fallback: use predefined blob centers resembling continents (lon/lat): NA(-100,45), SA(-60,-15), EU(15,50), AF(15,5), AS(90,45), SEA(105,10), AUS(134,-25). Draw each as cluster of ~40 random circles r 10-40px at canvas coords with colors mix '#2e6b3e','#7a8c4f','#b5a06a'. Add ice caps white bands top/bottom. onFallbackLand(lonFrac,latFrac): near blob centers → reuse same centers check distance. Implement simply.

mulberry32 standard.

Night fallback lights: place dots near blob centers with noise.

OK.

Loading: overlay hidden after build. Also if topojson world fetch fails, set flag & show small toast "国家边界数据加载失败，热力图已禁用" and disable toggle.

Textures count 3 + fetch; use Promise.allSettled.

Now write out everything. Let me draft the JS fully in final answer; ensure no syntax errors.

Careful details:

- OrbitControls: `controls.autoRotate=true; controls.autoRotateSpeed=0.7; controls.enableDamping=true; controls.dampingFactor=0.08; controls.minDistance=1.5; controls.maxDistance=6; controls.enablePan=false;`

- Renderer setSize on resize.

- ndc: 
```js
const rect=renderer.domElement.getBoundingClientRect();
mouse.x=((e.clientX-rect.left)/rect.width)*2-1;
mouse.y=-((e.clientY-rect.top)/rect.height)*2+1;
```

- popup element: position absolute inside container #app relative; use left/top computed.

Container: canvas fullscreen fixed; UI above via z-index.

Popup positioning relative to window: use fixed positioning. left = x px, top = y px, transform translate(-50%, calc(-100% - 16px)).

- When activeCity set, store; popup content update on year/metric change.

Now write it. Also ensure `sprite.center` default fine.

For popup arrow: CSS ::after triangle.

I need to double check three r160 SpriteMaterial + ACES: sprites tone-mapped; ok.

Points star material with sizeAttenuation false → size in px.

Star colors: mostly white, some blue/orange.

Let me also add subtle sphere "inner glow" at night side? skip.

Final check on importmap + module script ordering: importmap script before module script ✓.

Write the answer with: brief intro, complete HTML code, 运行方式 (local server / direct open, online requirement), 测试说明 (checklist mapping to requirements, fallback behavior). Keep post-code text concise per instructions (don't over-explain).

Let me write the code now carefully, end to end.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>3D 全球数据可视化地球仪</title>
<style>
:root{--accent:#4fc3f7;--panel:rgba(8,16,30,.78);--border:rgba(110,180,255,.16);--text:#dce8f5;--muted:#8aa3bd;}
*{box-sizing:border-box;margin:0;padding:0;}
html,body{height:100%;overflow:hidden;background:#04070f;color:var(--text);font-family:"PingFang SC","Microsoft YaHei","Segoe UI",system-ui,sans-serif;}
#scene{position:fixed;inset:0;display:block;}
.panel{position:fixed;background:var(--panel);border:1px solid var(--border);border-radius:14px;backdrop-filter:blur(12px);-webkit-backdrop-filter:blur(12px);box-shadow:0 8px 32px rgba(0,0,0,.45);z-index:10;}
#controlPanel{top:18px;left:18px;width:262px;padding:16px 18px;}
#controlPanel h1{font-size:16px;font-weight:600;letter-spacing:.5px;margin-bottom:12px;display:flex;align-items:center;gap:8px;}
#controlPanel h1 .dot{width:8px;height:8px;border-radius:50%;background:var(--accent);box-shadow:0 0 8px var(--accent);}
.seg{display:flex;gap:6px;margin-bottom:10px;}
.seg button{flex:1;padding:7px 0;font-size:13px;border-radius:8px;border:1px solid var(--border);background:rgba(255,255,255,.04);color:var(--muted);cursor:pointer;transition:.2s;}
.seg button:hover{color:var(--text);border-color:rgba(110,180,255,.4);}
.seg button.active{background:linear-gradient(135deg,rgba(79,195,247,.25),rgba(79,195,247,.08));color:var(--accent);border-color:rgba(79,195,247,.55);box-shadow:0 0 12px rgba(79,195,247,.15) inset;}
.metric-desc{font-size:12px;color:var(--muted);line-height:1.5;min-height:34px;margin-bottom:10px;}
.legend{margin:4px 0 12px;}
.legend-bar{height:9px;border-radius:5px;margin-bottom:5px;border:1px solid rgba(255,255,255,.08);}
.legend-labels{display:flex;justify-content:space-between;font-size:11px;color:var(--muted);}
.toggles{display:grid;grid-template-columns:1fr 1fr;gap:6px 10px;margin-bottom:10px;}
.toggles label{display:flex;align-items:center;gap:6px;font-size:12px;color:var(--muted);cursor:pointer;user-select:none;}
.toggles input{accent-color:var(--accent);cursor:pointer;}
.note{font-size:11px;color:#63788f;line-height:1.7;border-top:1px solid rgba(255,255,255,.06);padding-top:9px;}
#timeline{left:50%;bottom:20px;transform:translateX(-50%);display:flex;align-items:center;gap:14px;padding:12px 20px;min-width:min(640px,92vw);}
#playBtn{width:38px;height:38px;border-radius:50%;border:1px solid rgba(79,195,247,.5);background:rgba(79,195,247,.12);color:var(--accent);font-size:15px;cursor:pointer;transition:.2s;flex:none;}
#playBtn:hover{background:rgba(79,195,247,.25);box-shadow:0 0 14px rgba(79,195,247,.3);}
#yearLabel{font-size:24px;font-weight:700;font-variant-numeric:tabular-nums;color:#fff;text-shadow:0 0 16px rgba(79,195,247,.6);min-width:64px;text-align:center;}
.tl-right{flex:1;display:flex;flex-direction:column;gap:4px;}
#yearSlider{width:100%;appearance:none;height:6px;border-radius:3px;background:linear-gradient(90deg,var(--accent) var(--fill,100%),rgba(255,255,255,.12) var(--fill,100%));outline:none;cursor:pointer;}
#yearSlider::-webkit-slider-thumb{appearance:none;width:16px;height:16px;border-radius:50%;background:#fff;border:3px solid var(--accent);box-shadow:0 0 10px rgba(79,195,247,.8);cursor:pointer;}
#yearSlider::-moz-range-thumb{width:12px;height:12px;border-radius:50%;background:#fff;border:3px solid var(--accent);cursor:pointer;}
.tl-labels{display:flex;justify-content:space-between;font-size:10px;color:var(--muted);}
#popup{position:fixed;z-index:20;width:236px;background:var(--panel);border:1px solid rgba(110,180,255,.25);border-radius:12px;backdrop-filter:blur(14px);padding:12px 14px;transform:translate(-50%,calc(-100% - 18px));pointer-events:auto;box-shadow:0 10px 36px rgba(0,0,0,.55);display:none;}
#popup::after{content:'';position:absolute;left:50%;bottom:-7px;transform:translateX(-50%) rotate(45deg);width:12px;height:12px;background:var(--panel);border-right:1px solid rgba(110,180,255,.25);border-bottom:1px solid rgba(110,180,255,.25);}
.pop-head{display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:8px;}
.pop-title{font-size:15px;font-weight:600;}
.pop-title .en{font-size:11px;color:var(--muted);font-weight:400;margin-left:4px;}
.pop-sub{font-size:11px;color:var(--accent);margin-top:2px;}
.pop-close{background:none;border:none;color:var(--muted);font-size:18px;cursor:pointer;line-height:1;padding:0 2px;}
.pop-close:hover{color:#fff;}
.pop-row{display:flex;align-items:center;gap:8px;padding:5px 8px;border-radius:7px;font-size:12px;}
.pop-row.active{background:rgba(79,195,247,.1);border:1px solid rgba(79,195,247,.25);}
.pop-row .k{width:44px;color:var(--muted);flex:none;}
.pop-row .bar{flex:1;height:5px;border-radius:3px;background:rgba(255,255,255,.08);overflow:hidden;}
.pop-row .bar i{display:block;height:100%;border-radius:3px;}
.pop-row .v{width:86px;text-align:right;font-variant-numeric:tabular-nums;flex:none;color:#fff;}
#tooltip{position:fixed;z-index:25;pointer-events:none;background:rgba(10,20,36,.92);border:1px solid rgba(110,180,255,.35);border-radius:8px;padding:7px 11px;font-size:12px;line-height:1.6;display:none;box-shadow:0 6px 20px rgba(0,0,0,.5);}
#tooltip .tt-title{color:var(--accent);font-weight:600;}
#loading{position:fixed;inset:0;z-index:100;background:#04070f;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:18px;transition:opacity .6s;}
#loading.hide{opacity:0;pointer-events:none;}
.spinner{width:52px;height:52px;border-radius:50%;border:3px solid rgba(79,195,247,.15);border-top-color:var(--accent);animation:spin 1s linear infinite;}
@keyframes spin{to{transform:rotate(360deg);}}
#loading p{font-size:13px;color:var(--muted);letter-spacing:1px;}
#toast{position:fixed;top:16px;left:50%;transform:translateX(-50%);z-index:60;background:rgba(120,40,40,.85);border:1px solid rgba(255,120,120,.4);padding:8px 16px;border-radius:8px;font-size:12px;display:none;}
@media(max-width:640px){#controlPanel{width:210px;padding:12px;}#timeline{min-width:94vw;padding:10px 12px;gap:8px;}}
</style>
<script type="importmap">
{"imports":{
 "three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
 "three/addons/":"https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
 "topojson-client":"https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm"
}}
</script>
</head>
<body>
<canvas id="scene"></canvas>
...panels...
<script type="module">...</script>
</body></html>
```

Now module script. Let me write it thoroughly.

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { feature as topoFeature } from 'topojson-client';

/* ============ 基础配置与数据 ============ */
const YEAR_MIN=2000, YEAR_MAX=2024;
const GLOBE_R=1;
const METRIC_KEYS=['gdp','pop','flights'];
const METRICS={
 gdp:{name:'GDP',desc:'各国国内生产总值（万亿美元）· 城市为大都市区 GDP（十亿美元）',color:0xffc857,growth:0.032,fmtCountry:v=>v>=1?v.toFixed(2)+' 万亿':(v*1000).toFixed(0)+' 十亿'},
 pop:{name:'人口',desc:'各国总人口（亿）· 城市为大都市区人口（百万）',color:0x4fc3f7,growth:0.012,fmtCountry:v=>v>=100?(v/100).toFixed(2)+' 亿':v.toFixed(1)+' 百万'},
 flights:{name:'航班量',desc:'各国年起降架次（万架次）· 城市为日均航班班次',color:0xf06292,growth:0.036,fmtCountry:v=>v.toFixed(0)+' 万架次'}
};
const RANGES={gdp:[0.02,30],pop:[0.5,1500],flights:[1,1100]};
const CITY_MAX={gdp:2100,pop:38,flights:1400};
```

Then COUNTRIES (unpadded keys), CITIES, ROUTES, NAME_FIX.

Helper functions. State. Init.

I must be disciplined to finish. Write now the complete final code.

For city sprite scale animate: store c.sprite, c.core, c.phase.

For bars in popup: bar width pct = sqrt(v/CITY_MAX[k])*100 clamp.

Colors per metric for bars: css hex strings: gdp '#ffc857', pop '#4fc3f7', flights '#f06292'. METRICS add cssColor.

Legend gradient: from STOPS colors.

STOPS as [[t,[r,g,b]],...]; colorAt(t) returns [r,g,b].

Legend bar background: linear-gradient(90deg, rgb stops).

updateLegend(): min/max text = fmtCountry(RANGES[m][0]) / fmtCountry(RANGES[m][1]).

Slider fill percent: ((year-YEAR_MIN)/(YEAR_MAX-YEAR_MIN))*100 + '%'.

Play loop in animate: acc.

Alright — also disabling toggle for heat if world data missing.

One more detail: heat canvas texture — creating before world data arrives? We await all loads first, so fine.

Animate sun; also rotate starfield slowly.

Popup reposition: need activeCity world pos: city.group? store c.sprite; use c.sprite.getWorldPosition(tmp).

Sprites don't have geometry offset; world pos = position transformed ✓.

Also hide popup if hidden behind (facing<0.1): popup.style.visibility.

OK also "hover 航线显示航班信息" — done via tooltip.

Also when metric changes, pulse colors could change; keep.

Time to produce final. Also double-check `ctx.fill('evenodd')` support — yes.

Also the heat overlay may z-fight with bump-mapped earth? Bump is shading only, geometry same sphere radius 1; heat radius 1.004 — no z-fight.

City markers at radius 1.012 (above heat 1.004) ✓. Arc endpoints 1.012.

latLonToVec3(lat,lon,r).

Route mid elevation: dist*0.3 + small min 0.08.

Also add arcs slightly above: fine.

Now write final answer. Structure:

1. 完整代码 (single HTML).
2. 运行方式.
3. 测试/验证清单.
4. 说明 (data simulated, CDN dependency, fallbacks).

Code length large; proceed.

One concern: jsdelivr +esm for topojson-client — `https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/+esm` returns ESM bundle with exports {feature, mesh, ...}. import {feature} works. ✓

Another: world-atlas 2.0.2 path countries-110m.json at package root ✓ (unpkg.com/world-atlas@2/countries-110m.json known working; jsdelivr equivalent fine).

three-globe textures on jsdelivr: `https://cdn.jsdelivr.net/npm/three-globe/example/img/earth-blue-marble.jpg` — three-globe package includes example dir? unpkg widely referenced: `//unpkg.com/three-globe/example/img/earth-night.jpg` used in three-globe docs themselves. ✓ Pin version 2.31.0? Use @2 to get latest 2.x: `three-globe@2/example/img/...`. Hmm jsdelivr resolves semver ranges ✓. I'll pin @2.41.0? Not sure exists. Use @2 safe.

Actually let me double-check names: example img files in three-globe repo: `earth-blue-marble.jpg`, `earth-day.jpg`, `earth-dark.jpg`, `earth-night.jpg`, `earth-topology.png`, `earth-water.png`, `night-sky.png`, `earth-clouds.png`? I recall docs listing: earth-blue-marble, earth-dark, earth-day, earth-night, earth-topology, earth-water, night-sky. Good: use blue-marble (day), earth-night (lights), earth-topology (bump).

night-sky.png could be background: skip, stars generated.

Write final code now. Also ensure `THREE.NoColorSpace` exists r160 ✓.

Bump texture colorSpace: default NoColorSpace ✓ (leave default).

For heat canvas texture colorSpace SRGB ✓.

Renderer: `renderer.outputColorSpace = THREE.SRGBColorSpace` default ✓.

Ambient light intensity: with r160 default physical lights, AmbientLight intensity 0.4 fine; Directional 2.4.

Tone mapping exposure 1.1.

Let me write out. I'll also include a `window.__globe` debug handle maybe unnecessary.

Also `controls.addEventListener('start'...)`: OrbitControls dispatches 'start','change','end' ✓.

City/Route raycast on pointermove — also handle when popup open? fine.

Prevent context menu? not needed.

Final: write everything.

Popup bar color uses metric css color; active row highlight.

cityValue(c,metric,year):
```js
const d=c; const k={gdp:'g',pop:'p',flights:'f'}[metric]; const gk={gdp:'gg',pop:'pg',flights:'fg'}[metric];
const gr = d[gk] ?? METRICS[metric].growth;
return d[k]*Math.pow(1+gr, year-2024);
```

countryValue similar with COUNTRIES.

Pseudo:
```js
function pseudoCountry(idKey,metric,year){
 const r=hashStr(idKey+'|'+metric);
 const base = metric==='gdp'? 0.02+0.15*r : metric==='pop'? 0.6+20*r : 1.5+12*r;
 return base*Math.pow(1+METRICS[metric].growth, year-YEAR_MAX);
}
```

normVal as earlier.

drawHeat uses normVal(metric,val) for alpha & color.

Also overlay for missing worldFeatures: fill nothing.

Route name lookup: cityByName map.

Build routes after CITIES.

OK writing final HTML now. Also include instructions comments in code lightly.

For the timeline slider input event: `currentYear=+e.target.value; refreshAll();` and update fill css var.

Play button toggles `playing`; label ▶ / ❚❚.

Year advance in animate: if playing: yearTimer+=dt; if(yearTimer>0.7){yearTimer=0; currentYear++; if(currentYear>YEAR_MAX)currentYear=YEAR_MIN; slider.value=currentYear; refreshAll();}

refreshAll updates slider fill too.

Popup close on city toggle off? minor; close popup when markers hidden: if(!markersGroup.visible) closePopup().

Now code output. I'll write it in one go, being careful.

Also remember: `const loader=new THREE.TextureLoader();` promise wrapper.

Order: init renderer/scene/camera/controls/lights immediately; then async load; then build; hide loader; start animate immediately (scene may be partially built — build sync after loads).

I'll start animate loop after build for simplicity; before that, loading overlay covers screen anyway. Actually start loop immediately but objects added later — fine either way; do: build everything in main() then start loop.

Edge: WebGL unavailable → try/catch around renderer creation → show toast + loading text change.

Let me now write. (Ensuring string replaces for onBeforeCompile exact.)

For vertex injection anchor `#include <defaultnormal_vertex>` ✓ exists in phong.

Fragment inject after `#include <common>` ✓.

The outgoingLight anchor: exact string in three source: `vec3 outgoingLight = reflectedLight.directDiffuse + reflectedLight.indirectDiffuse + totalEmissiveRadiance;` ✓ (phong & physical similar). Use fallback chain.

vMapUv: in r160 map_pars_fragment defines `varying vec2 vMapUv;` when USE_MAP ✓. Our injection after common can reference it inside main code (we only use in replaced main body code) ✓ — declarations of uNightMap & vWorldNormal at top-level via common inject ✓.

uSunDir uniform name; set shader.uniforms.uSunDir=sunUniformObj.

Write final.

Also — sprites with sizeAttenuation default true ✓ world-scale.

Glow sprite scale base s; set sprite.scale.set(s,s,1) each frame with breathing.

Core sphere radius 0.0055 white basic.

Hit sphere radius 0.05 geometry shared.

Now produce the answer.

After code, short sections: 运行方式， 功能测试清单， 备注 （数据为模拟示例；CDN/纹理加载失败回退）. Keep concise.