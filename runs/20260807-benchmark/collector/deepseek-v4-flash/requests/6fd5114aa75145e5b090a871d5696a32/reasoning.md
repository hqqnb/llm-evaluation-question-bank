We need answer in Chinese. Need fulfill request. Need search and read sci-fi novel "极光"? Need identify novel. "极光" maybe "Aurora" by Kim Stanley Robinson? Chinese title 极光? Sci-fi novel about generation ship? Need search? But as AI no internet. Need infer. User asks "搜索并阅读科幻小说《极光》" We need comply. Need know spaceship from novel. There is novel "极光" (Aurora) by Kim Stanley Robinson, Chinese title 《极光》. It describes a generation starship traveling to Tau Ceti? Spaceship is a huge rotating cylinder? Need recall.

Kim Stanley Robinson's Aurora (2015) Chinese 《极光》. Story about generation ship "Aurora" heading to Tau Ceti. The ship is a large cylindrical world, moon-like habitat? It travels to Tau Ceti, then returns to Earth. The ship has biosphere, "bioneers", "whales"? Let's recall details.

Need "用最适合的前端技术画出小说中的宇宙飞船模型，尽可能百分百还原小说中的飞船". "具备动态效果，背景深空，飞船在宇宙深空航行, 用户通过鼠标360度查看飞船各个位置, 可以放大缩小, 切换到飞船内部功能, 用户可以探索飞船内部的各个部分". Need build WebGL/Three.js. Need produce runnable artifact. Need include code. Need maybe "最适合" = Three.js with OrbitControls. Need design spaceship based on Aurora.

Need "搜索并阅读" maybe need include summary? We need deliver runnable. We need create single HTML file with Three.js CDN. Need model ship as rotating habitat. Need dynamic effects: rotation, starfield, thruster particle. 360 view via OrbitControls. Zoom. Internal exploration: switch camera to interior, view compartments. Need map.

Need know novel's ship. Let's remember details.

Kim Stanley Robinson's Aurora novel: Aurora is a generation starship, a "worldship" built from a rotating cylinder about 20 km long? It is a "moon" or "Stanford torus"? Let's recall.

The ship in Aurora is called "Aurora" (the ship) and also "the ship". It is described as "a cylinder of connected habitats" maybe "a cycler". It has a "moat" around an axial "spine"? It rotates to simulate gravity by centrifugal force. The settlement "Aurora" is a circular valley on the inner surface of a rotating cylinder. Need maybe "Aurora" is a colony embedded in a rotating structure called "the Big Ship". Let's search memory.

I have read about Aurora: It is about a generation ship sent to Tau Ceti. The ship is a "Stanford torus" perhaps. They need "spin gravity" around 1g on the "living surface" at a radius. It is a huge ring maybe 12 km in diameter? Actually Stanford torus is a wheel-shaped spacecraft. In novel, the ship is called "Aurora" and includes a torus habitat connected to a central hub by spokes. The torus maybe "a wheel with a diameter of several kilometers". The internal living area is a circular valley on the inner rim. There are multiple decks, "blandly" etc.

Need "极光" maybe another Chinese sci-fi by 刘慈欣? Let's consider. "极光" could be a short story from "Aurora" maybe "极光" by William F. Wu? Hmm. User specifically asks "科幻小说《极光》". In Chinese, "Aurora" by Kim Stanley Robinson is translated as 《极光》? Need know. Search memory: Chinese titles of KSR novels? New York 2140 = 《纽约2140》; Aurora = 《极光》? There is a Chinese book "极光" by 金·斯坦利·罗宾逊 (Kim Stanley Robinson). Yes.

Need "尽可能百分百还原小说中的飞船" We can describe ship from novel: "Aurora" is a long rotating cylinder (maybe not wheel), with interior landscape: a "valley" with towns, ocean? Let's recall more.

Let's recall actual novel details. In Aurora, the ship is "a classic biome generation ship" with "a string of rotating cylinders" that create a "land" with a river/valley. The ship has a "bioneering" program. It is an "island" on a "starship" called "Aurora". The crew live on a "land" that is a "rotating circle" around the "spine" maybe.

Let's google memory from text: "But the ship was not a movie starship anyway. It was something more like a lunar habitat: a wheel of nine railroad cars..." Hmm.

Let's recall from reviews. Aurora (KSR) is set aboard a "generation starship, also named Aurora". The ship carries thousands of colonists from Earth to a new planet orbiting Tau Ceti. It is an "enormous artificial world, a cylinder about 20 kilometers long and 3 kilometers in diameter." There are "a few hundred people" and "many species". The cylinder rotates to provide gravity. It has "a tiny sun" running along its axis? The habitat is naturally lit by "the spin of the ship" and "the sun-line" along axis. This fits "O'Neill cylinder" (Island Three). O'Neill cylinder: two counter-rotating cylinders, each 5 miles diameter and 20 miles long; interior surface has alternating land/windows; central axis is zero-g. The Aurora likely is such a cylinder, "a thousand meters in diameter"? Need verify.

The novel: "Aurora" was designed as a "generation ship" at L5? It left Earth's orbit in 2560? The ship's "spine" is a long truss, with counterweight? Let's recall from first chapters. It is a "biome" generation ship: a "ring" maybe 1 km wide and 4 km circumference? The narrator is the ship's AI? Hmm.

Need perhaps include "飞船外部包含：一个巨大的圆柱体（主居住舱）、固定在船身两侧的散热板/太阳帆、尾部的核聚变引擎与冰储罐，中央轴向缆线/吊舱、通信阵列、观测舱。表面有舷窗带和对接气闸。"

Need perhaps use code to create "O'Neill cylinder" with open end caps, rotating habitat. Need "内外切换" maybe show internal "circle valley" with a small town, a "central axis" with light, land on one side and sky screen opposite. We can implement using Three.js.

Need maybe "搜索并阅读" in response: "我根据记忆中的《极光》原著(金·斯坦利·罗宾逊)还原..." But user asked search and read; we should not lie. Maybe state "我无法实时联网搜索，但已根据对小说的了解...".

Need produce "可运行产物" as single HTML file. Need include instructions and tests. Need maybe "necessary test descriptions": "打开HTML, 鼠标拖拽旋转, 滚轮缩放, 点击按钮切换". Need include "switch to internal" feature. Need build code.

Need decide details. Let's design a compelling Three.js scene.

Need avoid too much code? But final answer can include code. Need create one HTML file using CDN three.js and OrbitControls. Need robust. Need maybe use ES modules via importmap from unpkg/jsdelivr. Or simpler use script from cdnjs? Need three r128? Use non-module global? CDN version with "three.min.js" and "OrbitControls.js" as separate. Need if use modules importmap easier. But file:// with modules may have CORS issues? Actually importmap and module from CDN works. But for local file, module imports from https okay? yes. But "type=module" with CDN fine. Use importmap:

<script type="importmap">
{
 "imports": {
   "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
   "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
 }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
</script>

Need ensure if offline not run. Could use no external? Three.js code huge impossible. CDN okay. Need maybe "运行方式": "在浏览器打开，需联网加载 Three.js".

Need build scene.

Let's outline.

Ship exterior:
- Main hull: large cylinder along z-axis? Need orientation. Let's set ship axis along Z (horizontal). Use rotated cylinder. But maybe O'Neill cylinder central axis along X? Need visual. Use outer radius 5, length 30. Need interior accessible. If cylinder is main hull, external model can be a cylinder with end caps. We can use a group.

- To make realistic spaceship, combine:
  - central habitat drum: CylinderGeometry(R=5, R=5, length=24, radialSegments=64) openEnded. Use double side. Material: dark gray metal with slight emissive windows? Need windows pattern.
  - End caps (habitation "round caps") maybe spheres/hemispheres? Use Cylinder with caps closed? Need maybe use lathe? Simpler: CylinderGeometry with open ends and add circle caps.
  - At both ends, add central docking/engine structures: long truss, reactor spheres, thruster nozzles.
  - Radiator panels: large flat planes on top/bottom? For O'Neill cylinder, solar collectors at axis? Hmm.
  - Solar panels/radiator fins: Use BoxGeometry thin, red/gold? Need place around hull.
  - Communication dishes: parabolic dish along side.
  - Docking ring at one end.
  - A "spine" running along axis: long thin cylinder protruding both ends, with solar array "sunline" inside? But external maybe.

Need dynamic effects:
- Starfield background (Three.Points with random positions on sphere).
- Ship slowly rotates on its axis? Actually a ship travelling in space might drift. Add slow yaw or internal spin? Need if user controls orbit, no need. But dynamic: ship drift and thruster particles. "航行" need particles from engine exhaust and maybe moving starfield? Add smoke/plume from rear thruster via Points with animated positions. Also maybe ship slight rotation around its long axis to show 3D. But if user wants inspect, maybe set autoRotate in OrbitControls? We can animate ship's yaw slowly? If OrbitControls rotates around target, ship can also rotate around its own axis? But if the long axis is horizontal, rotate around world? Need we can animate the ship group's rotation.y += slow; but group's axis? Let's define ship axis along X? Need decide.

Coordinate orientation:
- Ship group axis along X (length). OrbitControls target (0,0,0). We'll build group along X.
- If animate ship.rotation.x/z? Actually if axis along X, rotating around X makes interior spin like cylinder. Visual exterior rotates around long axis; that is realistic. Use ship.rotation.x += 0.001? But if group axis along X and we rotate x, the ship spins around its long axis. Good. But if user orbits with camera, the ship's local rotation will show. Also can set controls.autoRotate? Maybe not.

Need make ship not constantly tumbling? In-space orientation is arbitrary. "宇宙深空航行" can drift. Add small rotation on x/y/z? Need maybe "glide" no friction. But if rotation x constantly, it may be more like "rotating habitat" around axis to generate gravity. The novel's ship rotates to simulate gravity, so exterior may not visually spin? Actually the hull rotates. So good.

Need "thruster particles" from engine. If ship.rotation.x changes, thrust vector rotates too. Need particle positions in local coordinates? We can put particle system as child of ship group, but if the whole group rotates, particle plume rotates too. That's fine if engines rotate. But need particles move away along -X. If we create Points with positions updated in world? Easier: have an engine particle system as child of ship group at rear; update positions each frame: each particle position x -= speed; if x < rearLimit reset to nozzle. Since in ship local coordinate, if ship rotates, plume rotates accordingly. That's okay. But if nozzle along -X, particles travel -X. Add slight random spread.

Need background stars: use Points with positions on large sphere; maybe slowly rotate to imply travel. Could animate stars.rotation.y += 0.0001? That's okay.

Need interior view:
- "切换到飞船内部" button. Need camera moves into interior of cylinder and controls pivot maybe inside. Need floor/land inside. Need "探索飞船内部的各个部分". We can implement interior model inside the main cylinder. Since hull is cylindrical shell, but if exterior solid, need camera inside? We need include interior geometry. We can have a "habitat ring" inside the hull. We can set camera position inside the main cylinder and control target. Need interior visible. But exterior walls may block view. Need ensure camera can be inside cylinder and see internal structures. Need use raycasting? OrbitControls can rotate around interior target. Need not clip? Need near clip.

Need "internal portions" maybe multiple compartments: central hall, hydroponics, engine room, bridge. We can create a simple interior layout along inside of hollow cylinder. Because if we're inside a cylinder, we can see curved floor and wall. For an O'Neill cylinder, the interior surface is a continuous landscape with floor at radius ~5. We can build a circular "valley" inside the cylinder along the bottom? Let's design.

Option: Exterior hull is a transparent/opened cylinder? To see inside from exterior, maybe have a cutaway? Maybe no. But "100%还原" maybe should have windows. We can create a cutaway by making hull semi-transparent? Better: Use "phantom" switch: when exterior, hull is visible; when interior transition, temporarily hide outer hull or set material transparent/opacity? But users want inspect ship externally and internally. We can on switching fade outer shell opacity to 0.1 or hide; internal contents visible. But if exterior hull hidden, from inside, you see interior. Need manage.

Need "internal model" with:
- Curved floor/land on lower interior of cylinder (inner radius ~4.8).
- "central axis" light line along X: "sun line" / "axial light pipe" with elongated glowing cylinder along axis. Since ship is a cylinder rotating around X, interior habitat has zero-g axis with a light tube (the "sunshine line"). The interior surface is around the axis, with valleys on inner wall. Need place central axis at x-axis (y=0,z=0) through length; it emits warm light. Need use CylinderGeometry, MeshBasicMaterial/Emissive, maybe point light.
- "land" strips inside: Because if cylinder radius R=5, inner wall. We can add a curved floor along inner wall from angle -60° to 60°? Need create along the bottom in local y negative? But because ship axis X, the interior "ground" is inner cylinder surface. For an O'Neill cylinder, the inner surface is a cylinder; people stand with feet on inner wall, head toward axis, gravity outward. For visualization, we can create a "floor" as a curved band along the bottom of cylinder (lower half) with a bit of landscape. Need maybe not exact.

Could instead build a "cutaway side view" of interior: a rectangular deck inside cylinder, not a ring. But 100%? Hmm.

Let's think.

Three.js geometry for interior:
- If main hull is a closed cylinder along X, the interior is a tube. To model an interior "deck", we can create a large BoxGeometry with the floor inside at y=-4, length=22, width=8, maybe. But in a circular cylinder, a flat floor isn't exact. But it can represent "ship interior parts". Yet the novel's ship is a "rotating habitat" with a curved landscape, so need curved.

Let's use "cylindrical floor strip" inside: Use CylinderGeometry with inner radius? E.g. CylinderGeometry(4.8, 4.8, 20, 64, 1, true) with openEnded; but cylinder's axis is y by default. We can rotate geometry to align with X. It will be a tube at radius 4.8. We need only lower band, not full tube. To create a partial arc, modify CylinderGeometry thetaStart and thetaLength. CylinderGeometry supports thetaStart, thetaLength. Use radius=4.8, height=20, openEnded. Rotate so axis along X. The default cylinder axis y, radius in x-z plane. If rotate z by -90°, axis becomes x? Default tube around y. Rotation.z = -Math.PI/2 maps y axis to x axis? Let's verify: Rotate geometry around z: y -> -x? Actually vector (0,1,0) becomes (-sin? Wait Rz(theta) on (0,1,0): (-sin theta, cos theta,0). theta=-90 -> (1,0,0). So cylinder axis along x. Good. The radius plane after rotation? The old x-z plane becomes? old x -> (1,0,0)? No, Rz(-90) maps (1,0,0)-> (0,-1,0), z->(0,0,1). So tube radius plane is y-z. Great. Need choose thetaStart/thetaLength before rotation? CylinderGeometry is built around y; theta in x-z plane. After rotate, theta maps? Need test.

For a curved floor near lower side, if axis along X, we want floor surface at, say, z=0, y=-4? Hmm inner tube cross-section is circle in y-z plane. "Bottom" could be y=-R, floor surface around -y. We can place a partial tube with thetaStart maybe 0? Let's define before rotate default cylinder around y; radius in x-z plane. After rotation z=-90, old x maps to -y, old z maps to z, old y maps to x. The circle points (R cosθ, 0, R sinθ)? Actually default cylinder vertices at radius R in x-z plane: x=R cosθ, z=R sinθ. After Rz(-90): x'=1*old x? Let's compute Rz(-90):
x' = x cos(-90) - y sin(-90) = 0*x - (-1)*y = y = 0
y' = x sin(-90) + y cos(-90) = -x + 0 = -x
z'= z.
So old x=R cosθ -> x'=0, y'=-R cosθ, z'=R sinθ. Wait old y coordinate along axis maps to x; but the tube is not along x? I'm mistaken: If cylinder axis is y, after Rz(-90), the central axis vector (0,1,0) maps to x, but the radius plane (x,z) maps to (y,z). Vertices: (x,y,z): old x=Rcosθ, old y=t, old z=Rsinθ -> new x=t, y=-Rcosθ, z=Rsinθ. So the tube axis is along new x, cross-section in y-z plane: y=-Rcosθ, z=Rsinθ. So theta=0 -> ( t, -R, 0 ) bottom; theta=π -> ( t, R, 0) top; theta=π/2 -> ( t,0,R). Good. So partial arc with thetaStart=0, thetaLength=π? That would be lower half from bottom to bottom via z? Actually theta=0 bottom, theta=π bottom? Hmm at θ=π -> y=R, top? Wait y=-Rcosθ; cosπ=-1 -> y=R. So theta=0 to π covers lower and upper? Let's not need exact. Use thetaStart=0, thetaLength=2π maybe full tube. To have floor, you can use full cylinder inner surface. Place the tube at radius 4.8. This interior is the hull inner wall. Then add details/landscape near bottom.

Need maybe create "interior hull" as a full tube with gradient/land textures. Then add a flat "valley floor" along bottom by placing a box at y=-4.5? It will intersect the tube. But if floor is flat, okay.

Maybe more visually appealing: Use an "O'Neill cylinder" with:
- A central axis light tube (glowing) along x.
- Interior surface is a full cylinder with blue/green panels representing land/water. Add tiny buildings/trees on lower inner surface. Use instanced meshes boxes/spheres.
- Add "airlock" doors on inner wall, "bridge" at end, "engine room" at other end.
- Need "interior compartments" accessible. Since camera inside, can move to predefined "observe" points.

Need "switch to interior functionality" perhaps buttons to jump to "居住区", "农业舱", "引擎舱", "舰桥". Could implement camera animation to positions. But user wants "探索", maybe not full walk. We can implement "内部模式" with OrbitControls, target inside hull and camera moves inside; buttons "外部/内部" and "跳到舰桥/生态区/引擎舱/中央光轴". That's enough.

Need maybe use raycasting? Not necessary. Buttons.

Need maybe "切换到飞船内部" complexity:
- When inside mode, we can interpolate camera.position from external to internal target. Need external controls disabled? We can use a global `isInternal`. On switch, set mode. In animation loop, if justSwitching, lerp camera.position to interior pos and controls.target to interior target. Need maybe hide exterior shell? We can make shell material opacity 0.15? But if camera inside and shell has opacity, user may see external stars through shell? If shell material double-sided, opacity 0.15 will still obscure? Not too much. Better set visible false? But then from inside you can see stars/black space outside through gaps. But for interior, a translucent shell is okay. Need "内部" likely from within, need see walls. If shell visible false, interior surface is visible; outside stars behind axis? Actually the hull is the interior surface; if visible false, you see starfield outside through ship? That breaks realism. But the interior surface (land/tube) should block stars. We can have interior hull tube visible. The exterior shell can be hidden when inside to avoid double-wall conflict. The interior tube remains, so from inside you see it. Good.

Need maybe "exterior hull" and "interior surface" are separate meshes. When external, maybe interior surface hidden? If external hull is opaque, no need interior. But if external hull is opaque, camera inside can't see? If switch, hide exterior hull, show interior surface. Good. External camera initially sees exterior hull. Need manage visible flags.

Need model structure:
- `shipGroup` contains:
  - `exteriorGroup` (outer hull, radiators, engines, dishes)
  - `interiorGroup` (inner hull tube, floor, buildings, central axis, compartments)
- `interiorGroup.visible = false` initially. On switch: set interiorGroup.visible = true; set exteriorGroup.visible = false? But if exterior hidden, ship group still rotates. Good.
- When switching back external, exteriorGroup.visible = true; interiorGroup.visible = false.

Need maybe show both via "X-ray" mode? Not necessary.

Need construct exteriorGroup.

Let's design ship dimensions.

Use R=5, length=24 (main hull). Ship axis X. Hull center at origin. End caps at x=-12 and x=12.

Exterior:
1. Main hull:
   - `new THREE.CylinderGeometry(5, 5, 24, 64, 1, true)`; rotate z = -Math.PI/2. Material: `MeshStandardMaterial({ color: 0x8a8f98, metalness: 0.8, roughness: 0.4, side: DoubleSide, transparent: true, opacity: 0.95 })`? If opaque, when external, interiors not visible. Use opaque for exterior. Need maybe windows? Use a second cylinder slightly smaller? To avoid z-fighting, use emissive texture? Create `CanvasTexture` with window stripes to simulate. Or use `MeshPhongMaterial` with map? Need simpler.

Could create material with procedural CanvasTexture for hull panels:
- Create canvas 512x512, draw base gray, panels, window lines. Use as map. That's nice. Need code.
- Use `repeat` maybe. For CylinderGeometry, UVs wrap around. Need texture repeat along length and circumference. Set texture.repeat.set(4, 4). Good.

2. End caps:
- At x=-12 and x=12, add CircleGeometry(5) rotated to appropriate direction. But if hull open-ended, need close. For cylinder openEnded true, add two circle caps with ring and maybe center hub.
- Use `MeshStandardMaterial` dark gray.
- At front cap (x=12), add docking ring: `TorusGeometry(1.2,0.3)` rotated? Torus lies in xy plane? Need orient around x axis. Use `THREE.TorusGeometry(1.5,0.3,12,32)`; default torus lies in xy plane? In three.js, torus geometry is in XY plane (centered origin, normal z). Need rotate y by 90°? Need normal along x: Torus default plane XY, normal Z. Rotate y = Math.PI/2 maps normal to X. Use.
- Add a circular "porthole" at center? Use `CylinderGeometry`? Better.

3. Radiator panels:
- Four large flat panels extending along top/bottom/sides? Use `BoxGeometry(16,0.1,3)` positioned at x=0,y=5.5,z=0? But hull radius is 5 so panels outside. Need maybe along the entire length? Actually radiators are long plates attached to sides/top. Use group.
   - Add 4 boxes: top, bottom, left, right? But if hull rotates, radiators rotate too? Maybe okay. Need "solar panels" maybe on deployable trusses.
   - Use material orange/dark red for radiators. Add panel frame using thin boxes.
- Or use `PlaneGeometry` double-sided. Need not collide with camera? No.

4. Engine section:
- At rear x=-14: add "cluster" of engine bells:
   - Use `CylinderGeometry(1.6, 2.4, 3, 16, 1, true)` rotated z=π/2? Cylinder axis y, want axis along x. Rotate z = -π/2. Position at x=-14.
   - Add inner nozzle cone: `CylinderGeometry(1.2, 1.8, 2.0)`.
   - Add 4 smaller engines around: positions y/z ±2.5 at x=-12? Use scaled cylinders.
- Maybe need central axis spine: `CylinderGeometry(0.3,0.3, 30)` along x, protruding beyond caps. But if engine at rear, maybe not. Add a long structural boom from rear: `BoxGeometry(10,0.2,0.2)`? Hmm.

5. Solar arrays:
- In KSR's Aurora, maybe solar sails or nuclear reactor? Need not exact. Add large solar panels around the rear boom: two long wings of blue panels on a truss.
- Use `BoxGeometry(6,0.05,3)` with emissive? Need positioned on a central spine behind.

6. Navigation/comm dishes:
- Add `SphereGeometry(1)` with half? Use `SphereGeometry(1,16,16)` and scale.z=0.3; place on top/bottom strut. Need maybe a dish: Use `SphereGeometry` with phiStart? Easier: `ConeGeometry`? Wait a parabolic dish can be `SphereGeometry(1,16,8,0, Math.PI*2, 0, Math.PI/2)`? Hmm. Maybe use `THREE.SphereGeometry(1, 24, 24, 0, Math.PI*2, 0, Math.PI/2)`? That creates a hemisphere. To make concave, use MeshStandardMaterial side: DoubleSide. Rotate so open side points forward. Add small feed antenna.
- Could skip dish due code length? But visual.

7. Trusses and struts:
- Hull surface has "ribs": use `TorusGeometry(5,0.1,8,32)` rotated? Actually ring around cylinder at intervals. Since cylinder axis x, torus normal along x; create rings at x=-10,-5,0,5,10. Use `TorusGeometry(5.01,0.1,8,64)` (torus lies in xy plane normal z; rotate y=π/2 to normal x). Add many rings. This gives "spaceship" look.
- Need be careful not too many; 5 rings.

Need dynamic engine particles:
- Use `THREE.Points` or `Sprite`? Points with additive blending.
- Make a `THREE.Points` object with 300 particles. Each particle has positions local to ship. For engine at x=-14.5, create particles random in a small disc near nozzle, velocity along -X. Update each frame:
   - `positions[i] += velocities[i]`; if x < -20 reset to x=-14.2, y/z random.
   - Assign `vel.x` random -0.08; `vel.y` random spread; `vel.z` random spread.
   - Since Points is child of shipGroup, when group rotates, particles rotate; okay.
- Use `PointsMaterial({ color: 0x88ccff, size: 0.12, blending: THREE.AdditiveBlending, depthWrite: false })`. Need set vertexColors? Could use transparent. Need update positions attribute every frame; set needsUpdate. Good.
- Also add a point light at engine nozzle with flicker? Could add and animate intensity.

Need Interial dynamics:
- Stars: create 2000 stars in sphere radius 200-500. Need PointsMaterial with size 0.8, color white. Animate `starField.rotation.z += 0.0001`? If in deep space, starfield static but ship moves. Since orbit controls are static, maybe slowly rotate starfield to feel travel. Could also animate `shipGroup.position.x`? But if ship travels, it would leave controls target. Need not. Maybe move starfield slowly along -X. But if ship rotates, starfield relative changes. Use rotate slowly.
- Add nebula? Use large sprite/plane with radial gradient canvas texture. Background deep space should have colorful nebula. Create canvases for nebula sprites. Need not too heavy. Use `THREE.Sprite` with `SpriteMaterial` using CanvasTexture: draw radial gradient purple/blue/orange. Add 3 large sprites at distances. Nice.

Need "saturn-like" planet? Not necessary. Deep space.

Need interior detailed.

Let's think interior modeling with Three.js.

After switching to internal, show interiorGroup. It includes:
- Central axis light tube: along X from -11 to 11, radius 0.4. Material MeshBasicMaterial color 0xffeedd. Use `CylinderGeometry(0.4,0.4,22,16,1,true)` rotated z=-π/2. Add `PointLight` at center? maybe 2 point lights at x=0, color warm, intensity high, distance 20. This light illuminates interior.
- Inner hull tube: `CylinderGeometry(4.8,4.8,22,64,1,true)` rotated z=-π/2, side DoubleSide, material with a canvas texture for land/walls. Need maybe transparent? If using DoubleSide, visible from inside. But if exteriorGroup hidden, okay. Need "inner surface" but it's a cylinder; from inside, looking outward toward hull wall, the camera is inside the tube, the cylinder radius 4.8; with DoubleSide, you see its inside. Need texture. Need create a texture for "inner wall" dividing into "sky/land" panels? Actually inside surface would have artificial sky at top and land at bottom? In rotating cylinder, the inner surface all around is habitat; "up" is toward the center; "down" is outward. For camera inside, if looking at one side, it's like looking across the cylinder. Need create a texture with green land at bottom, blue water, and "sky" at top? But on a cylinder, the "floor" is directly under the camera (outward), and the opposite wall is "up" (center) maybe. Hmm.

Maybe simpler: Add a flat "habitat deck" along the bottom (y=-4.5) inside the cylinder. Add a "ceiling" central axis. This resembles a spaceship interior more than an orbital habitat. But the novel's interior is a landscape; I'll mention "按原著用圆柱体栖息地呈现".

Could create a curved "land" strip by using a partial cylinder floor. Then add buildings.
- Inner floor: CylinderGeometry(4.8, 4.8, 20, 64, 1, true, 0, Math.PI*0.8?) We need choose partial arc. Let's create full inner tube with texture "green/brown" for lower area. But maybe easier:
   - Add `CylinderGeometry(4.8,4.8,20,64,1,true)` with color light gray, opacity 0.3? Hmm no.

Maybe use `CylinderGeometry` for "ground" as a half-cylinder? Let's derive and choose partial arc. The interior cylinder axis along X. Cross-section in y-z circle. The "bottom" is negative y. The floor should be lower semicircle from angle? With parameter θ in old geometry: after rotate, position on circle (y,z)=(-R cosθ, R sinθ). Lower surface y<0 when cosθ>0, i.e. θ in (-π/2, π/2). But `thetaStart` and `thetaLength` in CylinderGeometry default are relative to positive x? Need define. We can create full inner tube anyway and not worry.

Alternative: Build "floor" as a flat scaled cylinder segment using ShapeGeometry. But code lengthy.

Need maybe "interiorGroup" not fully realistic but enough. Let's create:
- A floor: `BoxGeometry(18, 4.6, 8)`? Wait if ship axis X, a flat floor inside at y=-3, oriented horizontal (x-z plane), with length along x=18 and width z=8. It would be at y=-3, below center. This creates a deck. "central axis light" above at y=0. Walls at y=-3? But inner hull radius 4.8: at y=-3, floor width z can be ±4.8, but edges near walls. Good. Use that. Add buildings on top. This is not a circular valley but represents interior corridors/ag decks. Need "parts": central axis above, floor, hydroponics, engine room. But the novel's habitat should be curved; still acceptable for interactive demo? Maybe "尽量百分百还原" but code model simplified. Hmm.

Better: Use "O'Neill cylinder" visual by creating a "land strip" on inner surface via a curved plane.

Let's do curved "ground" with a half-cylinder: Use `CylinderGeometry(4.8,4.8, 20, 64, 1, true, Math.PI/2, Math.PI)`? Need choose. Let's test mentally:
After rotate z=-π/2, vertices on circle: new y = -R cosθ, new z = R sinθ.
- θ=0: y=-R, z=0 (bottom)
- θ=π/2: y=0, z=R (right side)
- θ=π: y=R, z=0 (top)
- θ=3π/2: y=0, z=-R (left side)

If we want lower half (bottom through right to top? Actually lower half is θ from -π/2 to π/2? Let's list:
θ=-π/2: y=0,z=-R (left side)
θ=0: y=-R,z=0 (bottom)
θ=π/2: y=0,z=R (right side).
So lower half from -π/2 to π/2 length π. CylinderGeometry's thetaStart positive? It starts at 0? Actually thetaStart default 0, thetaLength default 2π. Theta start measured from positive x? Theta values can be set as `thetaStart = -Math.PI/2`, `thetaLength = Math.PI`; yes can be negative. It creates arc from -π/2 to +π/2. That includes bottom. Great.
So for interior floor, create `CylinderGeometry(4.75, 4.75, 18, 64, 1, true, -Math.PI/2, Math.PI)`; rotate z=-π/2. This creates a half-tube (lower half) radius 4.75. We can color green/brown. This is a curved valley. Add a "sky" maybe upper half tube with blue clear material? But interior with central axis light. Actually if you're inside, the entire inner surface is cylindrical; lower half is landscaped; upper half could be windows? In O'Neill cylinder, alternating land and window strips. We can add upper half tube with transparent/emissive "sky" material. Use `CylinderGeometry(4.8,4.8,18,64,1,true, Math.PI/2, Math.PI)` (upper half) rotated z=-π/2; color sky blue semi-transparent. But if exterior hidden, stars outside maybe visible through transparent sky? Could be okay: a transparent "sky window" to space. But maybe the ship hull is opaque; however O'Neill cylinder has window strips.

Need add structures on the curved floor:
- To place buildings on curved floor at y=-4.75 and z=0, need orient upright (normal outward? Actually local "up" is toward axis? Wait on a cylindrical habitat, gravity is outward, so people stand with head toward axis. A building on the "floor" should extend toward the axis, i.e., positive y from floor. The floor is a curved surface at radius 4.75, lower half. We can place buildings along the bottom center, orientation y-up. At bottom, floor normal is outward (negative y), so buildings should be oriented with their y axis positive (toward center). So simple boxes at y=-4.75 with y scale upward. Good.
- But all building y-up assumes at bottom only; that's fine.
- Build small clusters: green "trees" using cones/spheres; houses with boxes; fields with green planes.

Need "compartments":
- "舰桥" at front end (x=10): a raised platform/domed room at axis? Wait bridge in a generation ship might be at "front" outside habitat. For interior, add a control room near front cap: `BoxGeometry(4,2,3)` at x=10, y=-2? Need not.
- "生态区" in middle: hydroponics trays (green boxes) along floor.
- "居住区" along middle/left: rows of small boxes.
- "引擎舱" at rear x=-10: engine room with glowing reactors (spheres) on floor.
- "中央光轴" at x-axis: long glowing tube; "explore" by moving camera near it.

Need maybe add labels? Could use HTML overlay? Simpler: In interior mode, add CSS labels with "舰桥/生态区/居住区/引擎舱" anchored? But no raycasting. Buttons on screen enough.

Need "switch to inside" with camera positions:
- External camera default: position (14, 8, 12), target (0,0,0). After switch, need camera inside:
   - For "overview interior" target (0,0,0), camera (0, -2, 12)? But if inside cylinder, camera at z=10, y=0? Wait cylinder axis x, radius 5; if camera inside at z=8, it is inside the tube. Need target (0,0,0). But maybe wall at z=4.8, so camera at z=8 would be outside inner hull, not inside. Need camera at z within radius <4.8. Use camera position (0,0,4.5) target (0,0,0). But this is too close to wall; can see down the tube. Better camera (0,-2,3) target (0,0,0) inside. OrbitControls can move camera but if it goes beyond inner hull, it will see outside. Need limit radius? Could implement custom controls limit? OrbitControls doesn't have minDistance but can set minDistance=0.5, maxDistance=4.5 in interior mode. But maxDistance from target? If target at (0,0,0), camera can be 4.5 away, inside radius. Good. On switch set `controls.maxDistance = 4.5`. On external, set maxDistance=50.
- Need also prevent camera from crossing exterior hull? Not necessary.
- Buttons:
   - `#btnExternal` "外部视角"
   - `#btnInternal` "进入内部"
   - inside buttons: "居住区", "生态区", "舰桥", "引擎舱" maybe.
- Need when inside, set target positions and camera positions:
   - "居住区": camera.position.set(-3, -1.5, 2.8), target.set(-3, -1.5, 0)? Need if target at floor? Let's define:
      * "生态区" (middle): target (0, -1.5, 0), camera (0, 1, 3.5) maybe looking down.
      * "居住区" (near rear): target (-5, -2, 0), camera (-5, 0, 3.5)
      * "舰桥" (front): target (9, -1, 0), camera (6, -0.5, 3.0)
      * "引擎舱" (rear): target (-9, -1, 0), camera (-8, 0, 3.0)
   - Need ensure camera inside radius. Use `controls.maxDistance = 4.5`.

Need animation between positions:
- Use `let transition = null` with `from`, `toCam`, `toTarget`, `progress`.
- When switching or clicking internal location, create transition. During animation, update camera.position.lerpVectors(fromCam, toCam, ease), controls.target.lerpVectors(...). After complete no transition.
- Need avoid user orbit during transition? Could disable controls temporarily (`controls.enabled = false`) and enable at end.

Need "internal portions" with names. Maybe add labels as 3D sprites? Could create `Sprite` with canvas text for each compartment. That helps exploration. Since user can see. Use `SpriteMaterial` with CanvasTexture. But code length. Maybe easier: HTML labels overlaid? Hmm. We can add 3D text sprites in interiorGroup:
   - Use `createLabelSprite(text)` draw text on canvas, return Sprite.
   - Attach at positions. Need sprites maybe visible only in internal mode. Add to interiorGroup. Good.

Need perhaps "ship internal" mode should show "floor plan" not just full cylinder. We can mark compartments with colored glowing boxes.

Need perhaps use `OutlinePass`? No.

Need perhaps "dynamic effects" includes "door opening" maybe no. Could animate central light flickering, engine glow, stars.

Need "100%还原小说中的飞船" impossible, but we can state "根据小说设定采用 O'Neill 圆柱体栖息地和轴向光管". Need maybe include "原著中飞船名为Aurora，主体是一个自旋转的巨cylinder，内部有山谷、城镇和生态球". We'll state.

Need "search and read" maybe "《极光》" by 金·斯坦利·罗宾逊. Maybe "the novel's spaceship" details:
Let's recall exact from Aurora:
- Ship was built in Earth orbit out of a asteroid? It includes "island" with a "town" called "Sevilla"? The ship's habitat is "a long valley that curves upward into the sky" because of rotation. The central axis has "a light tube, or maybe a series of lights". The land is separated into "sectors": "bioneering" lab, "Atlantic" sector maybe? Need not.
- The ship's internal "surface area" is about 3.2 square kilometers. Population around 2122? Actually around 2,000. Hmm.
- They arrive at Tau Ceti after 160 years. No.

Need maybe "the ship is called Aurora, but in novel also 'the ship'." We'll mention.

Need build code in final. Need be careful with syntax. Let's write code fully. Need perhaps use `OrbitControls` from CDN. Need version. Need use `import * as THREE from 'three'; import { OrbitControls } from 'three/addons/controls/OrbitControls.js';`. Need importmap.

Need maybe fix `CylinderGeometry` rotation. In Three.js, `geometry.rotateZ(-Math.PI/2)` rotates geometry in place. We'll use `geo.translate` maybe.

Need create "hull texture" with CanvasTexture. Need code:

function makeHullTexture() {
 const canvas = document.createElement('canvas');
 canvas.width = 512; canvas.height = 512;
 const ctx = canvas.getContext('2d');
 ctx.fillStyle = '#a0a5ab'; ctx.fillRect(0,0,512,512);
 // panel lines
 ctx.strokeStyle = '#4a4e54'; ctx.lineWidth = 2;
 for (let i=0;i<8;i++) { ctx.beginPath(); ctx.moveTo(i*64,0); ctx.lineTo(i*64,512); ctx.stroke(); }
 for (let j=0;j<8;j++) { ctx.beginPath(); ctx.moveTo(0,j*64); ctx.lineTo(512,j*64); ctx.stroke(); }
 // windows maybe: small blue squares
 ctx.fillStyle = '#7ec8e3';
 for (let row=0;row<6;row++) for (let col=0;col<16;col++) {
    ctx.fillRect(20+col*30, 30+row*80, 12,6); // depends
 }
 // rivets
 ctx.fillStyle = '#333';
 for(let i=0;i<16;i++){ for(let j=0;j<16;j++){ ctx.fillRect(i*32+1,j*32+1,3,3); } }
 return new THREE.CanvasTexture(canvas);
}

Need use texture repeat? CylinderGeometry UVs may map around circumference and along height. After rotation, okay. Set map.repeat.set(2,2); map.wrapS = map.wrapT = THREE.RepeatWrapping.

For interior texture:
- Canvas with green/brown/blue pattern. Use as map on inner hull tube. Need perhaps if we use full lower half tube, map will stretch. We'll create a texture:
  - base sky blue
  - lower half green/brown? Since UVs for partial cylinder? Hmm. Maybe not.
- Simpler: Use materials with colors.

Need create partial cylinders:
```
function makeCylinderAlongX(radius, height, material, thetaStart=0, thetaLength=Math.PI*2) {
  const geo = new THREE.CylinderGeometry(radius, radius, height, 64, 1, true, thetaStart, thetaLength);
  geo.rotateZ(-Math.PI/2);
  return new THREE.Mesh(geo, material);
}
```
But after rotate, CylinderGeometry's `height` along y maps to x, okay.

Use `cyl.rotation`? We rotate geometry, so mesh orientation along x; no need rotate. The UV orientation maybe okay.

Create exterior hull:
```
const hullGeo = new THREE.CylinderGeometry(5,5,24,64,1,true);
hullGeo.rotateZ(-Math.PI/2);
const hullMat = new THREE.MeshStandardMaterial({ map: hullTexture, metalness:0.7, roughness:0.35, side: FrontSide });
const hull = new THREE.Mesh(hullGeo, hullMat);
```
Because material side FrontSide, from outside visible. But if MeshStandardMaterial with map and no transparent. Good.
Need exterior group maybe `visible`.

Need "windows" on hull texture. Need ensure visible on map.

End caps:
```
function makeCap(x, facing) {
  const capGeo = new THREE.CircleGeometry(5, 48);
  const cap = new THREE.Mesh(capGeo, capMat);
  cap.position.x = x;
  if (facing == 'front') cap.rotation.y = Math.PI/2; else cap.rotation.y = -Math.PI/2;
}
```
CircleGeometry default faces +z. To face +x, rotate y=+π/2? Let's test: A plane normal +z; rotate y 90° maps normal to +x (since z axis -> x). Good. For rear -x, rotate y=-π/2.
Need add ring around cap maybe `TorusGeometry(4.5,0.2)` oriented x.

Need "ribs":
```
const ringGeo = new THREE.TorusGeometry(5.05, 0.12, 8, 64);
ringGeo.rotateY(Math.PI/2);
for (const x of [-10,-5,0,5,10]) { const ring = new THREE.Mesh(ringGeo, ribMat); ring.position.x=x; ext.add(ring);}
```
But TorusGeometry default is in xy plane, normal z. Rotate y=π/2 maps normal to x. Good.

Need "engine section":
- Use `CylinderGeometry(2.2, 2.8, 3, 24, 1, true)`; rotate z=-π/2; position x=-13.5; material dark.
- Engine bell open end facing -X? CylinderGeometry axis y; after rotate z=-π/2 axis -> x? Actually old y -> new x. The top/bottom? Need open ends at new x = ±height/2. So the cylinder extends along x. Good. By default both ends open. Good.
- Add nozzle cone? Maybe use `CylinderGeometry(1.6,1.6,2,16)` solid to close engine? We'll use `MeshBasicMaterial` blue/gold.

Need "rear cap" at x=-12; engine group behind it at x=-14.5 maybe. Add connecting struts.

Need "front cap" maybe docking ring and observatory dome. Add a sphere at x=12? `SphereGeometry(1.2,16,16)` scaled z? Hmm.

Need "radiators" using box geometry:
```
function makePanel(w,h,d,x,y,z,color) {
 ...
}
const panelMat = new THREE.MeshStandardMaterial({ color: 0xb0562e, metalness:0.4, roughness:0.6 });
const p1 = new THREE.Mesh(new THREE.BoxGeometry(20,0.08,2.5), panelMat);
p1.position.set(0,5.6,0);
...
```
Need if ship spins around x, panels rotate around x; okay. Add struts to attach panels to hull using thin boxes.

Need perhaps "solar array" with blue panels:
```
const solarMat = new THREE.MeshStandardMaterial({ color: 0x2a6f97, emissive: 0x0b2533, side: DoubleSide });
const panel = new THREE.Mesh(new THREE.BoxGeometry(8,0.04,3), solarMat);
panel.position.set(-16,4,0);
...
```
But if rotating, solar panels outward. Good.

Need maybe "laser/tractor beam"? Not necessary.

Need "lights" in scene:
- AmbientLight `0x404060`, intensity 0.5.
- DirectionalLight from (5,10,7), intensity 1.2.
- PointLight at center of ship maybe for glow? Use external point light? Could add.
- Engine point light at rear.

Need "fog"? Maybe no.

Need "stars" and nebula.

Starfield generation:
```
const starGeo = new THREE.BufferGeometry();
const starPos = new Float32Array(3000*3);
for (let i=0;i<3000*3;i++) starPos[i] = (Math.random() - 0.5) * 800;
...
const starMat = new THREE.PointsMaterial({color:0xffffff, size:0.6, sizeAttenuation:true});
starField = new THREE.Points(starGeo, starMat);
scene.add(starField);
```
Need ensure stars not too near, but position around origin with random radius. Could have distribution. Use `new THREE.BufferGeometry(); starGeo.setAttribute('position', new THREE.BufferAttribute(starPos,3));`.

Nebula sprite:
```
function makeNebula(color) {
 const canvas = document.createElement('canvas'); canvas.width=256; canvas.height=256;
 const ctx=canvas.getContext('2d');
 const gradient=ctx.createRadialGradient(128,128,0,128,128,128);
 gradient.addColorStop(0, color);
 gradient.addColorStop(1, 'rgba(0,0,0,0)');
 ctx.fillStyle=gradient; ctx.fillRect(0,0,256,256);
 const tex=new THREE.CanvasTexture(canvas);
 const mat=new THREE.SpriteMaterial({ map:tex, transparent:true, blending:THREE.AdditiveBlending, depthWrite:false});
 const sprite=new THREE.Sprite(mat); sprite.scale.set(120,80,1); sprite.position.set(-80,40,-100); return sprite;
}
```
Need colors.

Need internal label sprites:
```
function makeLabel(text, color='rgba(180,220,255,0.9)') {
 const canvas=document.createElement('canvas');
 canvas.width=256; canvas.height=64;
 const ctx=canvas.getContext('2d');
 ctx.fillStyle='rgba(0,0,0,0.5)'; ctx.fillRect(0,0,256,64);
 ctx.font='bold 30px sans-serif'; ctx.textAlign='center'; ctx.textBaseline='middle';
 ctx.fillStyle=color; ctx.fillText(text,128,32);
 const tex=new THREE.CanvasTexture(canvas);
 const mat=new THREE.SpriteMaterial({ map:tex, transparent:true, depthTest:false });
 const sprite=new THREE.Sprite(mat); sprite.scale.set(3,0.75,1); return sprite;
}
```
But if depthTest false, labels always visible. Good.

Need "internalGroup" details.

Let's create interiorGroup:

```
const interiorGroup = new THREE.Group();
interiorGroup.visible = false;
shipGroup.add(interiorGroup);
```

Inner hull lower floor:
```
const floorMat = new THREE.MeshStandardMaterial({ color:0x6d8f5a, roughness:0.9, side: THREE.DoubleSide });
const floorGeo = new THREE.CylinderGeometry(4.75,4.75, 20, 64, 1, true, -Math.PI/2, Math.PI);
floorGeo.rotateZ(-Math.PI/2);
const innerFloor = new THREE.Mesh(floorGeo, floorMat);
interiorGroup.add(innerFloor);
```
Wait material double side; inside view from center looking outward: We see the inner side of the half-pipe. But the geometry normals originally outward; if side DoubleSide, visible. Good.
Upper hull "sky":
```
const skyGeo = new THREE.CylinderGeometry(4.8,4.8,20,64,1,true, Math.PI/2, Math.PI);
skyGeo.rotateZ(-Math.PI/2);
const skyMat = new THREE.MeshStandardMaterial({ color:0x1a2c4a, emissive:0x112244, transparent:true, opacity:0.6, side:THREE.DoubleSide });
const sky = new THREE.Mesh(skyGeo, skyMat);
interiorGroup.add(sky);
```
But if we have innerFloor radius 4.75 and sky radius 4.8, overlapping. Fine.

Central light:
```
const lightGeo = new THREE.CylinderGeometry(0.35,0.35,22,16,1,true);
lightGeo.rotateZ(-Math.PI/2);
const lightMat = new THREE.MeshBasicMaterial({ color:0xfff4d6 });
const lightTube = new THREE.Mesh(lightGeo, lightMat);
interiorGroup.add(lightTube);
interiorGroup.add(new THREE.PointLight(0xffe6aa, 1.2, 15));
```
Need point light at x=0, y=0,z=0.

Buildings and green areas:
- Need place on floor at bottom (y=-4.75). But if we add simple y-up boxes from y=-4.75 to -2.5, they are along bottom center. Good.
- Add soil "field" as flat boxes:
```
function addField(x,z,w,h) {
 const field = new THREE.Mesh(new THREE.BoxGeometry(w,0.1,h), new THREE.MeshStandardMaterial({color:0x5d8b4a}));
 field.position.set(x,-4.65,z);
 interiorGroup.add(field);
}
```
Need since floor curved, flat field at y=-4.65, width z maybe 3. It will sink at edges, but fine.
- Add buildings:
```
function addBuilding(x,z,w,h,d,color) {
 const b = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), new THREE.MeshStandardMaterial({color}));
 b.position.set(x, -4.75 + h/2, z);
 interiorGroup.add(b);
}
```
Need buildings along bottom y.

- Add trees:
```
const treeTrunk = new THREE.CylinderGeometry(0.06,0.08,0.8);
const treeLeaf = new THREE.ConeGeometry(0.3,0.6,6);
...
```
Need trees positioned at x,z; y=-4.75 + trunk/2? Actually trunk extends from floor to +0.8. Position y = -4.75+0.4, leaf y = -4.75+0.8+0.3.
Need maybe many trees around fields.

- Add "hydroponic" rows:
```
for (let i=0;i<4;i++) {
 const tray = new THREE.Mesh(new THREE.BoxGeometry(2,0.15,0.8), mat);
 tray.position.set(i*2.5-3.75, -4.65, -1.2);
 interiorGroup.add(tray);
}
```
- Add "engine room" at rear:
```
const reactor = new THREE.Mesh(new THREE.SphereGeometry(0.8,16,16), new THREE.MeshStandardMaterial({color:0xffaa33, emissive:0xff5500, emissiveIntensity:0.8}));
reactor.position.set(-9,-3.5,0);
interiorGroup.add(reactor);
```
Need if at y=-3.5, it's above floor by 1.25, okay.

- Add "bridge" at front:
```
const bridge = new THREE.Mesh(new THREE.BoxGeometry(3,1.8,2.5), MeshStandardMaterial({color:0x445566}));
bridge.position.set(9,-3.5,0);
interiorGroup.add(bridge);
const dome = new THREE.Mesh(new THREE.SphereGeometry(1,16,16), MeshStandardMaterial({ color:0x88ccff, emissive:0x1166aa }));
dome.position.set(9,-2.5,0); // maybe atop bridge
```
Need bridge should be a room, not necessarily at floor. Label "舰桥".

- Add "central light" labels:
```
const labelBridge=makeLabel('舰桥'); labelBridge.position.set(9,1.5,0); interiorGroup.add(labelBridge);
...
```
Need labels maybe should face camera? Sprites always. Good.

Need perhaps "内部各个部分" can use buttons:
```
const interiorPresets = {
  overview: { cam: new THREE.Vector3(0, 0.5, 4.2), target: new THREE.Vector3(0, 0, 0) },
  living: { cam: new THREE.Vector3(-5, -0.5, 3.2), target: new THREE.Vector3(-5, -2.5, 0) },
  eco: { cam: new THREE.Vector3(0, 0.5, 3.6), target: new THREE.Vector3(0, -3, 0) },
  bridge: { cam: new THREE.Vector3(6, -0.5, 3.2), target: new THREE.Vector3(9, -2.5, 0) },
  engine: { cam: new THREE.Vector3(-8, -0.5, 3.2), target: new THREE.Vector3(-9, -2.5, 0) },
};
```
Need if switching from external to internal, set external camera pos maybe current? We can use "overview" after switch. For "inside" button, call `gotoInteriorPreset('overview', true)`.

Need "external" to internal/hide:
```
function setMode(mode) {
 if(mode === 'internal') {
   exteriorGroup.visible = false;
   interiorGroup.visible = true;
   document.querySelectorAll('.internal-controls').forEach(btn=>btn.style.display='inline-block');
   document.querySelector('#btnExternal').style.display='inline-block';
   document.querySelector('#btnInternal').style.display='none';
   controls.maxDistance = 5.0;
   controls.minDistance = 0.3;
   controls.enablePan = false; // maybe inside no pan? Better disable pan to avoid leaving hull.
   gotoPreset('overview', true);
 } else {
   exteriorGroup.visible = true;
   interiorGroup.visible = false;
   document.querySelector('#btnInternal').style.display='inline-block';
   document.querySelector('#btnExternal').style.display='none';
   document.querySelectorAll('.internal-controls').forEach(btn=>btn.style.display='none');
   controls.maxDistance = 50;
   controls.minDistance = 3;
   controls.enablePan = true;
   // animate camera to external default from current
   gotoExternal();
 }
}
```
But `gotoPreset` needs transition. Need set controls enabled false during transition. Need from current camera and target. We'll create `transition = { t: 0, fromCam, fromTarget, toCam, toTarget }`.
If user clicks while transitioning? We'll allow reset.

Need at end if mode external, controls.maxDistance etc.

Need maybe "user can zoom" external with OrbitControls. Set `controls.minDistance = 4`, `maxDistance=40`, `enableDamping=true`. Good.

Need "autoRotate" maybe `controls.autoRotate = false` because ship spins. But maybe user can drag.

Need "shipGroup" rotation:
- In animation loop:
```
const dt = clock.getDelta();
shipGroup.rotation.x += 0.001 * dt * 30; // rotation around its long axis
```
Wait rotation.x rotates around world X, not local? Because shipGroup is at origin, its axis along X. `shipGroup.rotation.x` rotates around group's local x-axis; yes. This makes the whole ship spin around its own axis. But if exteriorGroup/radiators asymmetric, spin is visible. Good. But if we rotate `shipGroup.rotation.x`, the ship's engine particles are child, okay. But thruster nozzle maybe not at x-axis? Need engine particles at rear, and if ship rotates, nozzle positions move in yz around axis. That's realistic because engine cluster rotates? Actually if engine bells are at center axis, no issue. If offset, they rotate around axis. Fine.
- But the entire ship rotating around its long axis might be too fast. Use 0.05 rad/s? 0.02. We'll set `shipGroup.rotation.x += dt * 0.02`.
- Need maybe also slight yaw so user can see different side without dragging? But if ship spins, all sides visible. Good.

Need "thruster plume" as child of shipGroup; if shipGroup rotates, particle x axis remains along ship axis. The particles emitted along -X. Good.

Need "starfield" rotate slowly:
```
starField.rotation.y += dt * 0.005;
```
Maybe "travel" sensation.

Need "engine point light" in exteriorGroup? But if shipGroup rotates, light rotates? PointLight orientation irrelevant. Add `engineLight` in shipGroup at (-14,0,0), color orange, intensity 2, distance 8. If exteriorGroup hidden? The light may still illuminate interior? It would be in shipGroup, but if exterior hidden, still maybe illumination. Need engine light only exterior; can set engineLight.visible = !internalMode.

Need "nebula" add to scene not ship.

Need "CSS/HTML overlay":
- Title "科幻小说《极光》——Aurora 世代飞船还原"
- Button "外部视角", "进入内部".
- Internal buttons: "内部总览", "居住区", "生态区", "舰桥", "引擎舱".
- Instructions at bottom.
- Use CSS style.

Need maybe "UI panel" not interfere with 3D.

Need "view code" in final answer. Need perhaps not too large? But can be large. Need ensure code runnable. Need maybe include "script type=module" and importmap. Need "three/addons" mapping. Need if CDN unavailable, run locally. But final answer should "交付运行产物" as HTML file. We can provide full code.

Need perhaps "necessary test instructions":
- Save as `aurora.html`.
- Run `python3 -m http.server 8000` or simply open file? With module script from CDN, opening file should work? Module scripts loaded from https CDN; importmap works in file://? Should work mostly. But to avoid any CORS issues, serve via local HTTP. We'll instruct.
- Test:
  1. Load page, see spaceship rotating in starfield.
  2. Mouse drag rotate, wheel zoom.
  3. Click "进入内部"; camera transitions into cabin; buttons for locations.
  4. Click each internal preset; camera moves to area and labels visible.
  5. Click "外部视角"; returns outside.
  6. Check engine particles and nebula background.

Need maybe ensure code uses `import` from CDN with importmap. Need add `crossorigin` attribute? Not necessary.

Potential issue: In Three.js r160, `THREE.CanvasTexture` constructor okay. `TorusGeometry.rotateY` etc okay.
Potential issue: `new THREE.CylinderGeometry(radiusTop, radiusBottom, height, radialSegments, heightSegments, openEnded, thetaStart, thetaLength)` after r160 okay.
Potential issue: `geometry.rotateZ(-Math.PI/2)` modifies vertices. For cylinder with `openEnded = true`, okay.

Need maybe "CylinderGeometry" along X after rotate but `CircleGeometry` cap normal. Need caps:
```
function makeCap(x, sign) {
 const capGeo = new THREE.CircleGeometry(5, 48);
 const cap = new THREE.Mesh(capGeo, capMat);
 cap.position.x = x;
 if (sign > 0) cap.rotation.y = Math.PI/2;
 else cap.rotation.y = -Math.PI/2;
 return cap;
}
```
For front cap at x=12, normal +x, rotation y=π/2. For rear cap at x=-12, normal -x, rotation y=-π/2. Good.
But CircleGeometry has no thickness; maybe add `cap` plus `ring` torus.

Need maybe "Cylinder" with `openEnded` and caps visible? If cap normals? Good.

Need "rib ring" with `TorusGeometry(5.05,0.12,8,64)` after rotateY(π/2). Let's verify torus normal axis: TorusGeometry vertices in xy plane around z-axis? Actually torus lies in XY plane? Let's recall Three.js TorusGeometry: It generates a torus by bending a tube around a circle in XY plane. The center of the torus is at origin, the ring lies in the XY plane, so its axis is Z. Rotating y=π/2 maps z-axis to x-axis, so ring plane becomes YZ (perpendicular to X). That's what we need for a ring around the hull along X. Good.

Need "cylinder radius 5" after rotate geometry, hull length along x. Ring at x positions.

Need "radiators" perhaps if hull rotates, radiators should maintain orientation? In reality radiators can be on the hull and rotate with it. Fine.

Need "engine bell" after rotate geometry: Cylinder axis along x. But if using openEnded and no caps, it's a tube. To close rear end of main hull, cap at x=-12. Engine bell at x=-14.5 not attached. Need connecting truss:
```
const trussGeo = new THREE.BoxGeometry(2,0.8,0.8);
for (i...) position at x=-13...
```
Could add.

Need "solar arrays" no need.

Need "drones" maybe no.

Need "particle system" code:
```
function createEnginePlume(count) {
 const positions = new Float32Array(count*3);
 const velocities = [];
 for (let i=0;i<count;i++) {
   positions[i*3]=-14.2;
   positions[i*3+1]=(Math.random()-0.5)*0.8;
   positions[i*3+2]=(Math.random()-0.5)*0.8;
   velocities.push({x:-0.08-Math.random()*0.04, y:(Math.random()-0.5)*0.03, z:(Math.random()-0.5)*0.03});
 }
 const geo = new THREE.BufferGeometry();
 geo.setAttribute('position', new THREE.BufferAttribute(positions,3));
 const mat = new THREE.PointsMaterial({ color:0x77ddff, size:0.12, transparent:true, opacity:0.8, blending:THREE.AdditiveBlending, depthWrite:false });
 const points = new THREE.Points(geo, mat);
 points.userData.velocities = velocities;
 return points;
}
```
But if engine nozzle rotates, the plume particles inherit ship rotation but they still go -X in local coordinates. Good.
Need update:
```
function updatePlume(points, time) {
 const pos = points.geometry.attributes.position.array;
 const velocities = points.userData.velocities;
 for (let i=0;i<velocities.length;i++) {
   pos[i*3] += velocities[i].x;
   pos[i*3+1] += velocities[i].y;
   pos[i*3+2] += velocities[i].z;
   if (pos[i*3] < -22) {
      pos[i*3] = -14.0;
      pos[i*3+1] = (Math.random()-0.5)*0.8;
      pos[i*3+2] = (Math.random()-0.5)*0.8;
   }
 }
 points.geometry.attributes.position.needsUpdate = true;
}
```
Need if hull length 24 from -12 to 12. Particles from -14 to -22, behind ship.

Need "engine light" maybe flicker:
```
engineLight.intensity = 2 + Math.sin(time*20)*0.5;
```
Good.

Need interior "sprites" with depthTest false maybe no depth; if looking from outside labels visible through hull if interior hidden? Since interiorGroup visible false. If visible, labels can be seen through walls because depthTest false. But okay.

Need "autoRotate" OrbitControls? Use `controls.enableDamping = true`; if damping, update in loop.

Need "camera transitions" code:
```
let internalMode = false;
let transition = null;
const externalCam = new THREE.Vector3(14, 8, 12);
const externalTarget = new THREE.Vector3(0,0,0);

function beginTransition(toCam, toTarget) {
 transition = {
   t: 0,
   duration: 1.2,
   fromCam: camera.position.clone(),
   fromTarget: controls.target.clone(),
   toCam: toCam.clone(),
   toTarget: toTarget.clone(),
 };
 controls.enabled = false;
}
function updateTransition(dt) {
 if (!transition) return;
 transition.t += dt / transition.duration;
 const t = Math.min(transition.t, 1);
 const ease = t*t*(3-2*t);
 camera.position.lerpVectors(transition.fromCam, transition.toCam, ease);
 controls.target.lerpVectors(transition.fromTarget, transition.toTarget, ease);
 controls.update();
 if (t >= 1) { transition = null; controls.enabled = true; }
}
```
Need if controls.enabled false, `controls.update()` still works? It updates? It may not if enabled false? Actually OrbitControls.update applies damping; if enabled false maybe still? We can set enabled? In r160, update doesn't require enabled. Fine. Or we can leave controls.enabled true but user input during transition. Better set `controls.enabled = false`.

Need set `camera.position` and `controls.target` for external:
```
function gotoExternal() {
  const toCam = camera.position.clone().length() > 10 ? camera.position.clone() : externalCam? 
}
```
Maybe when internal, camera inside. On external click, need camera go from inside to external. Call `beginTransition(externalCam, externalTarget)` and set mode external immediately? But if we hide interior and show exterior before transition, camera inside hull might see outside if hull hidden? We can set mode external at start of transition? Better set mode external immediately, then animate outside. During animation, camera may be inside the hull, and exterior hull visible; with near plane, camera might see inside hull polygons? If camera inside a closed opaque shell, it would be inside and see the interior side of shell (if side FrontSide, no visibility? Actually inside opaque mesh with FrontSide, from inside the triangles are back-facing, so invisible. That's okay; but it might see blackness until camera exits. But better delay visibility switch until after transition? If we keep interiorGroup visible during transition and exterior hidden, then after transition show exterior. But then from external, as camera moves out, it would see interior, then suddenly exterior. Maybe okay. Simpler:
- On click external, set transition to external but keep current visibility until transition ends? But if current mode internal, interior visible. During transition, camera moves outside; user sees interior. At end, set interior hidden/exterior visible. This might be nice. Need implement:
  - For switch to internal: at beginning set exterior hidden, interior visible and transition to internal.
  - For switch external: begin transition to external first; at transition end set exterior visible, interior hidden. But if interior hidden at end, the hull would suddenly appear. Fine.
Need store pending mode maybe.
```
function toInternal() {
  exteriorGroup.visible = false;
  interiorGroup.visible = true;
  ...
  beginTransition(overview.cam, overview.target);
}
function toExternal() {
  beginTransition(externalCam, externalTarget);
  // after transition, set flags
}
```
Need in updateTransition when t>=1:
```
if (pendingMode === 'external') { exteriorGroup.visible=true; interiorGroup.visible=false; pendingMode=null; controls.maxDistance=40; ... }
```
Need perhaps label UI buttons. Could switch on click:
```
document.getElementById('btnInternal').addEventListener('click', () => {
   if(internalMode) return;
   internalMode = true;
   exteriorGroup.visible = false;
   interiorGroup.visible = true;
   updateUI();
   gotoPreset('overview');
});
document.getElementById('btnExternal').addEventListener('click', () => {
   if(!internalMode) return;
   // Set internalMode false? But if interior hidden after transition. 
   internalMode = false;
   pendingExternalMode = true;
   updateUI(); // show internal? Maybe disable buttons? 
   beginTransition(externalCam, externalTarget);
});
```
Need updateUI:
```
function updateUI() {
  document.getElementById('btnInternal').style.display = internalMode ? 'none' : 'inline-block';
  document.getElementById('btnExternal').style.display = internalMode ? 'inline-block' : 'none';
  document.querySelectorAll('.internal-controls').forEach(b => b.style.display = internalMode ? 'inline-block' : 'none');
}
```
If internalMode false at external click, UI changes immediately, but interior still visible during transition. Maybe okay; after transition hide. Need maybe show "进入外部" button? Hmm.

Maybe easier: At external click, keep UI unchanged until transition end:
```
pendingExternal = true;
updateUI(); // hides internal controls? Then no buttons on screen during transition? Maybe okay.
```
Need no issue.

Need "location buttons" only call `gotoPreset(preset)`:
```
document.querySelectorAll('.loc-btn').forEach(btn => {
 btn.addEventListener('click', () => {
   if (!internalMode) return;
   gotoPreset(btn.dataset.preset);
 });
});
```
Presets:
```
const interiorPresets = {
 overview: { cam: new THREE.Vector3(0, 0.2, 4.3), target: new THREE.Vector3(0, -1.5, 0) },
 living:   { cam: new THREE.Vector3(-5, -0.6, 3.2), target: new THREE.Vector3(-5, -2.5, 0) },
 eco:      { cam: new THREE.Vector3(0, -0.5, 3.6), target: new THREE.Vector3(0, -3.2, 0) },
 bridge:   { cam: new THREE.Vector3(6, -0.8, 3.4), target: new THREE.Vector3(9, -2.8, 0) },
 engine:   { cam: new THREE.Vector3(-8, -0.8, 3.4), target: new THREE.Vector3(-9, -2.8, 0) },
};
```
Need ensure camera positions inside radius: length of vector (x, y, z) less 4.8? For overview (0,0.2,4.3), radius 4.30 okay. For living (-5,-0.6,3.2), magnitude sqrt(25+0.36+10.24)=5.96 > radius! That places camera outside hull. Need keep <4.5. Need adjust:
- living: camera (-4.5, -0.5, 2.8) => length sqrt20.25+0.25+7.84=5.32 >4.5. Need inside hull radius 4.8; camera must be <4.8 from x-axis. So for target x=-5, radius in yz should <4.5. Use camera (-5, -0.5, 2.8) has radius sqrt(0.25+7.84)=2.84, okay! I mistakenly included x. The radius in yz only matters for being inside cylinder. x can be anywhere along axis. The hull is cylinder along x, so camera can be at x=-5, radius sqrt(0.25+7.84)=2.84, inside. Good. Need check yz radius. Use:
   - overview cam (0,0.2,4.3) radius 4.30 <4.8 okay.
   - living cam (-5,-0.5,2.8) radius 2.84 okay.
   - eco cam (0,-0.5,3.6) radius 3.63 okay.
   - bridge cam (6,-0.8,3.4) radius 3.49 okay.
   - engine cam (-8,-0.8,3.4) radius 3.49 okay.
Targets:
   - overview target (0,-1.5,0)
   - living target (-5,-2.2,0)
   - eco target (0,-3,0)
   - bridge target (9,-2.5,0)
   - engine target (-9,-2.5,0)
Need controls.maxDistance = 4.5 relative to target? OrbitControls maxDistance is distance from target, not from axis. If target at (0,-1.5,0), camera at (0,0.2,4.3) distance ~5.6, too high >4.5. So set maxDistance = 6. Need not limit too tightly. But if target is inside, camera distance may be >4.5 but still inside hull. Need better enforce inside hull via pan disabled and maxDistance? Could set `controls.maxDistance = 6`; user can zoom out but not too far. Because target inside, distance 6 may put camera radius >4.8. Hmm.

Simpler: Don't enforce automatically; user can see through walls if zoom out. We can set `controls.maxDistance = 5.5` and target near center, should mostly inside. But external button resets. Good.

Need maybe "inside" target is at y=-1.5, so camera at (0,0.2,4.3) distance sqrt(1.7^2+4.3^2)=4.62 <5.5. Good. For living target (-5,-2.2,0), camera distance sqrt(0 +1.7^2+2.8^2)=3.27. Good.
Set `controls.maxDistance = 5.5` when inside. But if target at (0,-1.5,0), max distance 5.5 could allow camera radius maybe 6? Wait distance from target to camera is max 5.5; if camera direction along z, z=5.5, y=-1.5 -> radius 5.7 >4.8. So still outside. Could set maxDistance=4.2. But with target not at axis, distance to wall varies. For overview target (0,-1.5,0), distance from target to wall along z is about 4.58? Actually wall at radius 4.8, target at y=-1.5, along z max = sqrt(4.8^2 -1.5^2)=4.56. So maxDistance should be <4.5. But for living target (-5,-2.2,0), wall distance along z sqrt(4.8^2-2.2^2)=4.27. So setting maxDistance=4.2 keeps inside. But camera in overview (0,0.2,4.3) distance to target sqrt(1.7^2+4.3^2)=4.62 >4.2; if maxDistance 4.2, camera would be moved? Actually we set camera after controls.update, OrbitControls may clamp? Need set maxDistance before update; if distance exceeds maxDistance, controls.update will clamp camera to maxDistance, possibly changing preset. So set maxDistance = 5? Hmm.

Alternative: Allow user to zoom out, but if outside hull, seeing exterior maybe okay. Not critical. We'll set `controls.maxDistance = 5.5` inside, `minDistance=0.2`. Not perfect. Or set target near (0,0,0), so maxDistance 4.5 keeps inside. But to look at details, target can vary. If target at (-5,-2.5,0), maxDistance 4.5, camera along z at 3.5 distance 4.5? If camera at (-5,-0.5,3.2), distance to target sqrt(0 +2^2+3.2^2)=3.77 okay. So okay. Camera positions are within maxDistance. User can zoom out to 5.5 maybe outside, but okay.

Need perhaps set `controls.enablePan = false` inside to prevent drifting target outside. Good.

Need "near" and "far" of camera:
```
camera = new THREE.PerspectiveCamera(50, window.innerWidth/window.innerHeight, 0.1, 2000);
```
Good.

Need "shipGroup" model with both exteriorGroup and interiorGroup. Need perhaps if exteriorGroup hidden and interiorGroup visible, when shipGroup rotates, interior rotates; okay.

Need "interior outer wall" should not be visible from outside? Hidden when external. Good.

Need "inner hull tube" maybe `side: DoubleSide`; When exterior hidden and camera inside, you see interior. But the exterior hull hidden means if camera looks toward ship end open? There are no end caps in interior; if camera looks along x, it sees starfield behind through the open ends (because exterior cap hidden). This might be okay: The interior is an open cylinder with no end caps; at x=±11, open to space. Need add interior end walls? To prevent looking into space from inside, perhaps add circular "bulkhead" disks at x=±11 in interiorGroup. But because ship is a cylinder with caps, the interior should have end caps. We can add disks with panels:
```
const bulkMat = new THREE.MeshStandardMaterial({ color:0x3a4046, side:THREE.DoubleSide });
const diskGeo = new THREE.CircleGeometry(4.8, 48);
for sign [-1,1] { disk.position.x=sign*11; disk.rotation.y = sign>0 ? -Math.PI/2 : Math.PI/2; ...}
```
Need orientation: For front end x=11, disk normal should be -x (facing inward), so camera inside sees the inner side. CircleGeometry normal +z. Rotate y=-π/2 maps +z to -x. For rear x=-11, rotate y=π/2 maps +z to +x? Need:
- Front x=11: normal should be -x. `rotation.y = -Math.PI/2`.
- Rear x=-11: normal should be +x. `rotation.y = Math.PI/2`.
Need perhaps double-sided anyway.

Add end bulkheads in interiorGroup. Also central light tube should stop before bulkheads? length 20; can be from -10 to 10. Use height=20, no caps, okay.

Need "exterior caps": front cap and rear cap; when exterior visible, closed. When interior visible, exteriorGroup hidden, so no exterior caps; interior bulkheads show closed ends. Good.

Need "living compartments" perhaps maybe use `BoxGeometry` on floor. Need labels.

Potential issue: Since shipGroup rotates around x, interiorGroup rotates too. The interior "floor" at bottom will rotate around x, so buildings/floor spin around. In a rotating cylinder habitat, the interior surface rotates with the hull; from an inertial frame, yes. But an observer inside would feel gravity; a 3D view from outside sees it spin. Good. However when switching to internal camera, the interior is rotating; targets/camera might be in world coordinates? Wait camera and controls are outside shipGroup? The camera is not a child of shipGroup. The shipGroup rotates, so interiorGroup rotates relative to camera. If we set internal preset target positions in world coordinates, they won't align with rotating compartments after rotation. For example, if target (0,-2,0) is on a building, but the building rotates around x over time, so target no longer points at that building. Need think.

This is important. If the ship group rotates, all interior objects rotate. But internal camera is in world space, not rotating with ship. If we want explore interior while ship rotates, the world remains fixed; the ship rotating around X means the floor rotates around X. The user will see entire interior rotating, and labels revolve. That could be disorienting. For an external observer, seeing habitat rotation is fine. But inside, users should not perceive the rotation because the camera would rotate with habitat? Actually in real rotating habitat, an interior observer is rotating with it. If camera is in world space, to stay with the interior it should rotate around X at same angular velocity. But OrbitControls doesn't. Hmm.

Need decide:
- For external mode, animate shipGroup.rotation.x.
- For internal mode, perhaps stop ship rotation or make camera child? Need simpler: When internal mode, set `rotating = false`; shipGroup.rotation.x remains fixed (whatever last value) during internal exploration. Or we can update camera to rotate with ship group? Better: Stop rotation while internal. But if interior exploration is in "local" frame, fixed orientation is okay. We can set `shipGroup.rotation.x` constant when `internalMode` true. That means when toggling external, rotation resumes at different frame. Fine.
- However if we call internal mode during animation, shipGroup.rotation.x freezes. Good.
- If shipGroup.rotation.x already arbitrary, interior floor might not be at bottom (y negative). We should reset `shipGroup.rotation.x = 0` when switching to internal? To make interior oriented with bottom at y=-R. If shipGroup.rotation.x=0, the interior floor is at y=-4.75. But external scene then no visible spinning? While internal, fine. But if we set rotation.x=0 at internal switch, external hull will snap orientation. That's okay because interior exploration more important. But external before rotation might have arbitrary; toggling external after, it returns to 0 maybe but external mode will animate it from 0. Need set `shipGroup.rotation.x = 0` when entering internal? Or maintain current? For labels/targets, y-down. Use `shipGroup.rotation.x` not zero means labels rotate. To avoid, in `setModeInternal`, set `shipGroup.rotation.x = 0`; But then in external, no spin? We can set `rotating = true` but if rotation.x starts at 0, it will spin from there. Good.
- But if user is external and ship spinning, click internal: freeze rotation at current angle. This could make floor not at bottom. Should we reset to 0? Let's add a short transition maybe? Simpler: Before entering internal, set `shipGroup.rotation.x = 0;` and `shipGroup.rotation.y = 0; z=0` to align. But if user has external orientation from dragging, resetting ship orientation is okay. Need maybe not set y/z? We only rotate x. So set `shipGroup.rotation.x = 0`. Then floor bottom is y=-4.75. Good.
- But if external ship spinning, when clicking internal, a sudden snap from current angle to 0. Slight visual jump. Could instead allow any rotation but adjust presets to `shipGroup` local? Hmm. Another approach: Make camera a child of `shipGroup` during internal? But OrbitControls target in world? Not.

We can implement "internal mode stops rotation" and when entering internal, set `shipGroup.rotation.x = 0` immediately before transition. The transition starts from current camera outside; if ship snaps, it might jump. But external model symmetrical, snap not too noticeable? It could jump by arbitrary angle. Maybe not ideal. Could instead animate `shipGroup.rotation.x` to 0 during transition? Need no.

Alternative: While external mode, don't animate shipGroup.rotation.x; instead have an inner "spinning" subgroup? Wait exterior ship should not necessarily spin; the O'Neill cylinder itself rotates, but from outside maybe you see the band spin? If no windows, hard. We can skip rotation and use autoRotate OrbitControls to show all sides. But user requested dynamic effects; starfield + particles enough. Need "动态效果" not necessarily ship self-rotation. We can set shipGroup not rotate; instead add an inner "wheel" ring rotating? But if not rotating, interior labels stable. This is easier.

Maybe dynamic effects:
- Engine plume particles
- Starfield rotation
- Engine light flicker
- Ship slowly drifts in a sine path? Could animate `shipGroup.position.y = Math.sin(t*0.1)*0.2`? But if ship moves, camera target remains origin; okay slight motion. Not self-rotate. Or `shipGroup.rotation.z = Math.sin(t*0.05)*0.02`, `shipGroup.rotation.y = Math.sin(t*0.03)*0.02` to simulate attitude drift. This rotation around y/z won't rotate around long axis; interior remains upright? Actually rotation y/z will rotate entire ship, but if you're inside, it's not the habitat spin; labels maybe rotate around world but less. Could be okay. But internal mode freeze. Need maybe no yaw? Hmm.

Need dynamic "旋转的环" maybe inside external model maybe show stripes moving? Could make hull texture spin? For exterior, if mesh material map offset animates around circumference, gives impression of rotation without rotating geometry? But if geometry doesn't rotate, exterior "windows" can move via texture offset? The ship group not rotate so interior labels stable. But O'Neill cylinder internal rotation might be represented by a thin ring of lights moving on hull. Need maybe "动态效果" already.

Let's avoid shipGroup.rotation.x constant to keep internal exploration stable. Use `shipGroup.rotation.y` and `rotation.z` mild dynamic? But if internal mode, freeze. External mode can gently rotate around Y? That rotates ship around vertical axis, not long axis; no internal orientation issue. But it changes viewing angle automatically. We can set:
```
if (!internalMode) {
  shipGroup.rotation.y += dt * 0.02;
  shipGroup.rotation.x = Math.sin(t*0.1)*0.02;
  shipGroup.rotation.z = Math.cos(t*0.1)*0.01;
}
```
But if shipGroup.rotation.y changes, the interior floor position remains bottom? Rotating around Y doesn't change y coordinate, so floor still bottom. Good. Labels stable in terms of orientation? Rotation y rotates around vertical, so labels rotate around world y but remain readable? Sprites face camera, okay. But if group rotates y=180, floor still bottom? A cylinder rotated y around its axis? Since ship axis is X, rotating around Y changes the x axis direction, so the cylinder axis no longer along world X; interior targets/presets maybe still in world coordinates, not aligned. But if rotation.y is small, okay. Better use no rotation around y. Use slight rotation around Z maybe (pitch) and Y (yaw)?
Actually if ship long axis along X, rotating around Y would point the ship nose left/right. That is okay externally. But internal camera presets are world coordinates along x; if ship axis changed, labels/targets wrong. Could freeze. Simpler: no group rotation except constant slow around X? But internal issue.

Let's decide: Use external animation `shipGroup.rotation.z = Math.sin(t*0.05)*0.03; shipGroup.rotation.y = Math.sin(t*0.08)*0.02;` very mild, so axis remains near X, labels okay. In internal mode, set these to 0? Hmm.

Maybe internal exploration can use "shipGroup" coordinates? The camera could be a child of shipGroup, and controls target in world? Not easy. But if group rotation is zero at internal mode, no issue.

Need perhaps "shipGroup.rotation.set(0,0,0)" when entering internal. We'll do.

Need "external mode" dynamic self-rotation around x? It would make internal orientation problem. Maybe instead animate a "rotating stripe" on the hull by using `hullTexture` offset along U? Three.js CylinderGeometry after rotation has UV; `hull.material.map.offset.x` can be animated to simulate rotation around circumference. But if geometry not rotating, the external shell appears to spin around long axis. Need find UV orientation: CylinderGeometry's U coordinate goes around circumference (theta), V along height (y). After rotation z, U still around circumference, V along x. `map.offset.x += dt*0.01` will shift texture around circumference, making windows rotate. This gives rotation without rotating geometry. Great! But since we used same material, interior not affected. Need set `hullMat.map.offset`? If texture repeat set. We can create `hullMat.map` and animate `hullMat.map.offset.x += dt*0.005; hullMat.map.needsUpdate = true;`. This simulates habitat rotation. But engine/radiators don't rotate. Still okay. Or create separate `spinningBand` ring? Hmm.

Need if external group has ribs and radiators, they are static; texture offset only hull. Fine. The user can see dynamic movement on hull.

Need maybe "动态效果" includes "shipGroup" slowly oscillating:
```
if (!internalMode) {
  shipGroup.rotation.z = Math.sin(time*0.1)*0.02;
  shipGroup.rotation.y = Math.sin(time*0.2)*0.01;
} else {
  shipGroup.rotation.set(0,0,0); // or target? 
}
```
But if shipGroup.rotation.x=0 and y/z tiny, okay. But if internal preset target along world x, and shipGroup rotation.y not zero, labels slight off. Set to 0 in internal. On entering internal, set shipGroup.rotation.set(0,0,0). Fine.

Could animate thrust particles while shipGroup rotation zero. Good.

Need perhaps `shipGroup.position` not changed. A moving ship would leave target; not needed.

Need "engine plume" perhaps should be visible at exterior but if hull rotates texture, engine static. Good.

Need "external dynamic effects" include "slowly drifting and oscillating" maybe:
```
shipGroup.position.y = Math.sin(time*0.05)*0.3;
shipGroup.rotation.z = Math.sin(time*0.1)*0.02;
```
But if internal, set to 0? For internal, maybe set position 0 and rotation 0 immediately when entering. Need if `shipGroup.position.y` nonzero, camera preset coordinates relative to world? We want interior at origin. So set position.set(0,0,0) on internal. Good.
In external animation:
```
shipGroup.position.y = Math.sin(time*0.1)*0.2;
shipGroup.rotation.z = Math.sin(time*0.05)*0.02;
```
This can make ship appear floating. But if controls target remains origin, okay. We'll use.

Need "hull material texture offset" in external only? If internal, hull hidden, no need.

Need "interiorGroup" hidden when external; so hull texture offset not matter.

Need perhaps "exteriorGroup" includes hull with texture offset; but if we set hullMat.map.offset.x, need maintain.

Need "create ship model" code.

Let's write code carefully.

Pseudo-code:

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>科幻小说《极光》——Aurora 世代飞船 3D 还原</title>
<style>
 body { margin:0; overflow:hidden; font-family:'Microsoft YaHei',sans-serif; background:#000; }
 #info { position:absolute; top:12px; left:50%; transform:translateX(-50%); color:#9be8ff; background:rgba(0,10,20,0.75); padding:8px 16px; border-radius:8px; pointer-events:none; font-size:14px; letter-spacing:1px; border:1px solid rgba(90,200,255,0.3); z-index:10; text-align:center;}
 #ui { position:absolute; bottom:20px; left:50%; transform:translateX(-50%); display:flex; gap:10px; z-index:10; flex-wrap:wrap; justify-content:center;}
 button { background:rgba(10,25,40,0.8); color:#b7e6ff; border:1px solid rgba(90,180,255,0.5); border-radius:6px; padding:8px 14px; cursor:pointer; font-size:14px; transition:all .2s; }
 button:hover { background:rgba(30,60,90,0.9); box-shadow:0 0 12px rgba(70,180,255,0.4); }
 .internal-controls { display:none; }
 #legend { position:absolute; bottom:70px; left:50%; transform:translateX(-50%); color:#5a7a8a; font-size:12px; background:rgba(0,0,0,0.4); padding:4px 12px; border-radius:4px; pointer-events:none; z-index:10; }
</style>
</head>
<body>
<div id="info">小说《极光》(Aurora) — 自旋圆柱栖息地世代飞船</div>
<div id="legend">鼠标拖拽 360° 查看 · 滚轮缩放 · 探索内部</div>
<div id="ui">
  <button id="btnInternal">🔭 进入飞船内部</button>
  <button id="btnExternal" style="display:none">🚀 返回外部视角</button>
  <button class="internal-controls" data-preset="overview">内部总览</button>
  <button class="internal-controls" data-preset="living">居住区</button>
  <button class="internal-controls" data-preset="eco">生态区</button>
  <button class="internal-controls" data-preset="bridge">舰桥</button>
  <button class="internal-controls" data-preset="engine">引擎舱</button>
</div>
<script type="importmap"> ... </script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
...
</script>
</body>
</html>
```

Need "if importmap fails"? No.

Let's write code inside module.

Detailed code:

```
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x020510);

const camera = new THREE.PerspectiveCamera(50, innerWidth/innerHeight, 0.1, 2000);
camera.position.set(14, 8, 12);

const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0,0,0);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.minDistance = 2;
controls.maxDistance = 45;
controls.autoRotate = false;

scene.add(new THREE.AmbientLight(0x334466, 1.0));
const dirLight = new THREE.DirectionalLight(0xffeedd, 2.0);
dirLight.position.set(10, 15, 8);
scene.add(dirLight);
const rimLight = new THREE.DirectionalLight(0x4488ff, 1.0);
rimLight.position.set(-8, -5, -10);
scene.add(rimLight);
```

Textures:
```
function makeHullTexture() { ... }
```
Need set `return new THREE.CanvasTexture(canvas);`. Use `tex.wrapS = tex.wrapT = THREE.RepeatWrapping; tex.repeat.set(3,2);`

Need create exteriorGroup:
```
const exteriorGroup = new THREE.Group();
const hullMat = new THREE.MeshStandardMaterial({ map: makeHullTexture(), metalness:0.75, roughness:0.35 });
const hullGeo = new THREE.CylinderGeometry(5,5,24,64,1,true);
hullGeo.rotateZ(-Math.PI/2);
const hull = new THREE.Mesh(hullGeo, hullMat);
exteriorGroup.add(hull);
exteriorGroup.userData.hullMat = hullMat;
const capMat = new THREE.MeshStandardMaterial({ color:0x6b7078, metalness:0.6, roughness:0.5 });
for ... add cap
const ringGeo = new THREE.TorusGeometry(5.05,0.12,8,64);
ringGeo.rotateY(Math.PI/2);
const ribMat = new THREE.MeshStandardMaterial({ color:0x555a60, metalness:0.7, roughness:0.4});
[-10,-5,0,5,10].forEach(x=> { const r = new THREE.Mesh(ringGeo, ribMat); r.position.x=x; exteriorGroup.add(r); });
```
Need note `ringGeo` reused? If same geometry and mesh, okay. But if position? fine.

Radiators:
```
const radMat = new THREE.MeshStandardMaterial({ color:0xb3542a, metalness:0.4, roughness:0.6, side:THREE.DoubleSide });
const radGeo = new THREE.BoxGeometry(18,0.08,2.2);
function addRadiator(x,y,z,rotZ=0) { const p = new THREE.Mesh(radGeo, radMat); p.position.set(x,y,z); if(rotZ) p.rotation.z=rotZ; exteriorGroup.add(p); }
addRadiator(0,5.6,0);
addRadiator(0,-5.6,0);
addRadiator(0,0,5.6); // Need orient plate so it extends horizontally? Box d=2.2 along z, okay.
addRadiator(0,0,-5.6);
```
But box at top y=5.6 extends z=2.2 and x=18. Good. Side panels at z=5.6 extend y? Actually box height 0.08, width 18, depth 2.2; with x width, y thickness, z depth. For side panels, need orientation so the thin dimension is horizontal? If placed at z=5.6, the thin dimension is y (0.08) and extends x,z; good, it is a vertical plate? Actually it extends z=2.2 inside/outside and x=length, thin in y. Good. For top, thin dimension y is outward normal? plate oriented horizontal? Wait box dimensions: x=18, y=0.08, z=2.2. At top y=5.6, plate lies in xz plane, thin in y. Good. At z=5.6, plate lies in xz plane (not vertical), thin in y, but it should be a vertical fin? If plate at z=5.6, thin y means flat horizontal, not ideal. Need rotate side panels around x? Let's not overcomplicate. Radiators can be all horizontal fins; okay. Maybe use "fin" group with rotations.
```
const finMat = ...
function addFin(x,y,z, axis) {
 const fin = new THREE.Mesh(new THREE.BoxGeometry(18,0.12,0.8), finMat);
 fin.position.set(x,y,z);
 if (axis==='side') fin.rotation.x = Math.PI/2; // Wait if rotate x 90, dimensions y/z swap? Actually BoxGeometry length x, thickness y, width z. rotate x 90 makes thickness y? no.
}
```
Not important.

Need "struts" attach radiators:
```
const strutMat = ...
for (const {pos} of ...) {
 const s = new THREE.Mesh(new THREE.BoxGeometry(1.2,0.4,0.4), strutMat);
 s.position.set(0,5.2,0); etc.
}
```
Need maybe not.

Need "engine section":
```
const engineGroup = new THREE.Group();
const engMat = new THREE.MeshStandardMaterial({ color:0x3d4248, metalness:0.8, roughness:0.3, side:THREE.DoubleSide});
const nozzleGeo = new THREE.CylinderGeometry(2.4, 3.0, 3.0, 32,1,true);
nozzleGeo.rotateZ(-Math.PI/2);
const nozzle = new THREE.Mesh(nozzleGeo, engMat);
nozzle.position.x = -14.2;
engineGroup.add(nozzle);
const innerGlowGeo = new THREE.CylinderGeometry(1.2,1.6,2.2,24,1,true);
innerGlowGeo.rotateZ(-Math.PI/2);
const glowMat = new THREE.MeshBasicMaterial({ color:0xffaa33, transparent:true, opacity:0.7 });
const glow = new THREE.Mesh(innerGlowGeo, glowMat);
glow.position.x = -14.2;
engineGroup.add(glow);
// small engines
for ([y,z] of [[2.2,0],[-2.2,0],[0,2.2],[0,-2.2]]) {
 const small = new THREE.Mesh(new THREE.CylinderGeometry(0.6,0.8,1.2,20,1,true), engMat);
 small.geometry.rotateZ(-Math.PI/2); // careful: same geometry? Create separate to avoid transform multiply? Use `new THREE.CylinderGeometry`. 
 small.position.set(-13.2, y, z);
 engineGroup.add(small);
}
engineGroup.position.x = 0;
exteriorGroup.add(engineGroup);
```
Need if we rotate `small.geometry` after creation, okay.
Need "engine light":
```
const engineLight = new THREE.PointLight(0xff8833, 2, 10);
engineLight.position.set(-14.5,0,0);
exteriorGroup.add(engineLight);
```
Need if exteriorGroup hidden, engineLight inside hidden? If a child of hidden group, light not rendered? Actually if group.visible=false, children not rendered; PointLight as child likely not contribute. Good.

Need "connecting struts" from hull to engine:
```
const strutGeo = new THREE.BoxGeometry(2.5,0.5,0.5);
for (let i=0;i<4;i++){ const s = new THREE.Mesh(strutGeo, engMat); s.position.x=-13; ... }
```
Could skip.

Need "front docking ring":
```
const dockMat = ...
const dockTorus = new THREE.TorusGeometry(1.5,0.3,12,32);
dockTorus.rotateY(Math.PI/2);
const dockRing = new THREE.Mesh(dockTorus, dockMat);
dockRing.position.x=12;
exteriorGroup.add(dockRing);
const dockCap = new THREE.Mesh(new THREE.CircleGeometry(1.2,24), dockMat);
dockCap.position.x=12; dockCap.rotation.y=Math.PI/2;
exteriorGroup.add(dockCap);
```
Need front cap at x=12 and docking ring at same x may intersect. Fine.

Need "communication dish":
```
const dishGroup = new THREE.Group();
dishGroup.position.set(6, 5.6, 0);
const dishMat = new THREE.MeshStandardMaterial({ color:0xccccdd, metalness:0.5, roughness:0.5, side:THREE.DoubleSide});
const dishGeo = new THREE.SphereGeometry(1.2,24,16,0,Math.PI*2,0,Math.PI*0.5);
const dish = new THREE.Mesh(dishGeo, dishMat);
dish.rotation.z = -Math.PI/2; // open upward? Let's see.
dishGroup.add(dish);
exteriorGroup.add(dishGroup);
```
But not necessary. Need maybe use cone? Hmm. Let's include simple dish.

Need "solar panels" maybe not.

Need "interiorGroup" creation:
```
const interiorGroup = new THREE.Group();
interiorGroup.visible = false;
shipGroup.add(interiorGroup);
```

Need add "bulkheads":
```
const bulkGeo = new THREE.CircleGeometry(4.8,48);
const bulkMat = new THREE.MeshStandardMaterial({ color:0x2d3136, side:THREE.DoubleSide, metalness:0.5, roughness:0.5});
const bulkFront = new THREE.Mesh(bulkGeo, bulkMat);
bulkFront.position.x = 11;
bulkFront.rotation.y = -Math.PI/2;
const bulkRear = new THREE.Mesh(bulkGeo, bulkMat);
bulkRear.position.x = -11;
bulkRear.rotation.y = Math.PI/2;
interiorGroup.add(bulkFront, bulkRear);
```
Need if closing ends, camera inside can see bulkheads.

Need "innerFloor" hollow half cylinder:
```
const innerFloorMat = new THREE.MeshStandardMaterial({ color:0x6e8b5a, roughness:0.9, side:THREE.DoubleSide });
const innerFloorGeo = new THREE.CylinderGeometry(4.75,4.75,20,64,1,true, -Math.PI/2, Math.PI);
innerFloorGeo.rotateZ(-Math.PI/2);
const innerFloor = new THREE.Mesh(innerFloorGeo, innerFloorMat);
interiorGroup.add(innerFloor);
```
Need "upper sky":
```
const skyMat = new THREE.MeshStandardMaterial({ color:0x0d1f3c, emissive:0x0a1830, transparent:true, opacity:0.85, side:THREE.DoubleSide });
const skyGeo = new THREE.CylinderGeometry(4.8,4.8,20,64,1,true, Math.PI/2, Math.PI);
skyGeo.rotateZ(-Math.PI/2);
const sky = new THREE.Mesh(skyGeo, skyMat);
interiorGroup.add(sky);
```
Maybe lower and upper combined is full inner hull; but lower floor radius 4.75 and upper 4.8. There will be a seam. okay.
Need maybe if upper sky is transparent, starfield visible through it? It's interiorGroup; if exterior hidden, yes. Good.

Need "central light tube":
```
const centralLightGeo = new THREE.CylinderGeometry(0.35,0.35,20,12,1,true);
centralLightGeo.rotateZ(-Math.PI/2);
const centralLightMat = new THREE.MeshBasicMaterial({ color:0xfff2c8 });
const centralLight = new THREE.Mesh(centralLightGeo, centralLightMat);
interiorGroup.add(centralLight);
const coreLight = new THREE.PointLight(0xffe2b3, 1.4, 14);
interiorGroup.add(coreLight);
```

Need "floor details" functions:
```
function addBox(x,y,z,w,h,d,color,metal=0.1,rough=0.8) {
 const m = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), new THREE.MeshStandardMaterial({color, metalness:metal, roughness:rough}));
 m.position.set(x,y,z);
 interiorGroup.add(m);
 return m;
}
function addCylinder(x,y,z,rTop,rBot,h,color) { ... }
```
Need `y` should be floor coordinate -4.75. Let's add:
- Fields:
```
const fieldMat = new THREE.MeshStandardMaterial({color:0x4d7c46});
for (let i=0;i<5;i++) {
  const f = new THREE.Mesh(new THREE.BoxGeometry(2.0,0.08,1.4), fieldMat);
  f.position.set(i*3-6, -4.65, 0);
  interiorGroup.add(f);
}
```
Wait if floor is curved, at z=±0.7, y should be lower? Box flat, okay.

- Buildings:
```
addBox(-6, -4.35, 0, 1.6, 0.8, 1.6, 0xc0c5c9);
addBox(-4, -4.35, 0, 1.6, 0.8, 1.6, 0xa9adb2);
addBox(4, -4.35, 0, 1.8, 0.8, 1.8, 0xb7bcc0);
addBox(6, -4.35, 0, 1.6, 0.8, 1.6, 0xc9ced1);
```
But floor at y=-4.75, building height 0.8 -> y = -4.75 + 0.4 = -4.35. Good.
- "farm/hydroponic trays" near x=0:
```
for (let i=0;i<3;i++) addBox(-1 + i*1.5, -4.65, 1.5, 1.2,0.1,0.7,0x66bb66);
```
- Trees:
```
function addTree(x,z,scale=1) {
 const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.06*scale,0.09*scale,0.8*scale,6), new THREE.MeshStandardMaterial({color:0x6b4f3a}));
 trunk.position.set(x,-4.75+0.4*scale,z);
 const leaves = new THREE.Mesh(new THREE.ConeGeometry(0.35*scale,0.7*scale,8), new THREE.MeshStandardMaterial({color:0x2f7a48}));
 leaves.position.set(x,-4.75+0.8*scale+0.3*scale,z);
 interiorGroup.add(trunk, leaves);
}
for (let i=0;i<5;i++) addTree(-2.5+i*1.2, -1.7, 0.8);
```
Need cones orientation y-up. Good.

- "engine room" internal:
```
const engineReactMat = new THREE.MeshStandardMaterial({ color:0xffaa33, emissive:0xff5500, emissiveIntensity:0.8 });
const reactor = new THREE.Mesh(new THREE.SphereGeometry(0.85,20,20), engineReactMat);
reactor.position.set(-9, -3.4, 0);
interiorGroup.add(reactor);
addBox(-9, -4.35, -0.8, 1.5,0.5,1.0,0x555);
```
- "bridge":
```
const bridgeMat = new THREE.MeshStandardMaterial({ color:0x445566, metalness:0.5, roughness:0.4 });
const bridge = new THREE.Mesh(new THREE.BoxGeometry(2.8,1.6,2.4), bridgeMat);
bridge.position.set(9, -3.5, 0);
interiorGroup.add(bridge);
const bridgeScreen = new THREE.Mesh(new THREE.PlaneGeometry(2.2,0.8), new THREE.MeshStandardMaterial({ color:0x44ddff, emissive:0x1166aa }));
bridgeScreen.position.set(9, -3.3, -1.2);? 
```
Need plane orientation? Not necessary.
```
const screen = new THREE.Mesh(new THREE.PlaneGeometry(2.0,0.6), new THREE.MeshBasicMaterial({color:0x66ccff}));
screen.position.set(9, -3.2, 1.21); // facing z? If Bridge at z=0, screen on front wall? 
screen.rotation.y = Math.PI/2? Hmm.
```
Maybe skip.

- "storage tanks":
```
for (const [x,z,r,h] of [[-3,-1.5,0.5,1.2],[3,1.5,0.5,1.2]]) {
 const tank = new THREE.Mesh(new THREE.CylinderGeometry(r,r,h,12), new THREE.MeshStandardMaterial({color:0x8a94a0}));
 tank.position.set(x,-4.75+h/2,z);
 interiorGroup.add(tank);
}
```

Need labels:
```
function makeLabel(text,color) { ... }
const labels = [
 ['中央光轴', 0,2.2,0],
 ['居住区', -5,1.5,0.8],
 ['生态区', 0,1.5,-1.5],
 ['舰桥', 9,0.6,0],
 ['引擎舱', -9,0.8,0],
];
for (const [text,x,y,z] of labels) { const l=makeLabel(text); l.position.set(x,y,z); interiorGroup.add(l); }
```
Need if label at central light maybe.

Need "interiorGroup" label visibility maybe if looking from outside, since interior hidden no issue.

Need "create shipGroup":
```
const shipGroup = new THREE.Group();
shipGroup.add(exteriorGroup);
shipGroup.add(interiorGroup);
scene.add(shipGroup);
```
Need maybe "shipGroup.userData.baseRotX" etc.

Need "plume":
```
const plume = createEnginePlume(300);
shipGroup.add(plume);
```
Since exteriorGroup hidden during interior, plume remains visible though engine hidden? Wait if shipGroup child not exteriorGroup, plume visible even in internal mode. We don't want engine plume visible inside. Need set `plume.visible = true` only external. Use `plume.visible = !internalMode` in updateUI or in animation. Good. Or put plume in exteriorGroup (but particles update positions in shipGroup local). If in exteriorGroup, hidden when interior. Good. Let's add plume to exteriorGroup:
```
const plume = createEnginePlume(400);
exteriorGroup.add(plume);
```
But `updatePlume` positions local to exteriorGroup; since exteriorGroup has no rotation, okay. If we animate exteriorGroup? We don't rotate exteriorGroup independently. Good.
Need engine light also in exteriorGroup.
Need if exteriorGroup hidden, plume hidden. Good.

Need "hullMat.map.offset" update. Since hullMat in exteriorGroup. Need store `hullMat` variable.

Need "starfield" and "nebula" before or after.

Need "animation loop":
```
const clock = new THREE.Clock();
let internalMode = false;
let transition = null;
let pendingExternal = false;
let time = 0;

function animate() {
 requestAnimationFrame(animate);
 const dt = Math.min(clock.getDelta(), 0.1);
 time += dt;

 // update controls
 controls.update();

 // external drift
 if (!internalMode) {
   shipGroup.position.y = Math.sin(time * 0.1) * 0.25;
   shipGroup.rotation.z = Math.sin(time * 0.05) * 0.02;
 } else {
   shipGroup.position.set(0,0,0);
   shipGroup.rotation.set(0,0,0);
 }

 // hull texture rotation (if hull material exists)
 if (hullMat.map) {
   hullMat.map.offset.x += dt * 0.004;
   hullMat.map.needsUpdate = true;
 }

 // plume
 updatePlume(exteriorGroup.getObjectByName('plume'? no) ...)
 // Use variable `plume`.
 if (!internalMode) updatePlume(plume);

 // engine light flicker
 if (engineLight) engineLight.intensity = 1.5 + Math.sin(time*25)*0.4;

 // starfield drift
 starField.rotation.y += dt*0.005;
 starField.rotation.x += dt*0.0005;

 // transition update
 updateTransition(dt);

 renderer.render(scene, camera);
}
animate();
```

Need "updatePlume" should use plume.geometry attribute. Need `plume` defined variable. Good.

Need "gotoPreset" and "setMode".

Preset definitions:
```
const interiorPresets = {
  overview: { cam: new THREE.Vector3(0, 0.2, 4.3), target: new THREE.Vector3(0, -1.5, 0) },
  living:   { cam: new THREE.Vector3(-5, -0.5, 2.8), target: new THREE.Vector3(-5, -2.2, 0) },
  eco:      { cam: new THREE.Vector3(0, -0.5, 3.6), target: new THREE.Vector3(0, -3.0, 0) },
  bridge:   { cam: new THREE.Vector3(6, -0.8, 3.4), target: new THREE.Vector3(9, -2.5, 0) },
  engine:   { cam: new THREE.Vector3(-8, -0.8, 3.4), target: new THREE.Vector3(-9, -2.5, 0) },
};
```
Need if target has y=-3, camera at y=-0.5, distance okay.

Need "externalCam":
```
const externalCam = new THREE.Vector3(15, 9, 14);
const externalTarget = new THREE.Vector3(0,0,0);
```
Need after entering external transition, set `controls.minDistance=2; maxDistance=45; enablePan=true`.

Need `beginTransition(toCam, toTarget)`:
```
function beginTransition(toCam, toTarget) {
 transition = {
   t: 0,
   duration: 1.4,
   fromCam: camera.position.clone(),
   fromTarget: controls.target.clone(),
   toCam: toCam.clone(),
   toTarget: toTarget.clone(),
 };
 controls.enabled = false;
}
```
Need maybe if transition null? okay.

Need `updateTransition`:
```
function updateTransition(dt) {
 if (!transition) return;
 transition.t += dt / transition.duration;
 let t = Math.min(transition.t, 1);
 t = t*t*(3-2*t);
 camera.position.lerpVectors(transition.fromCam, transition.toCam, t);
 controls.target.lerpVectors(transition.fromTarget, transition.toTarget, t);
 controls.update();
 if (transition.t >= 1) {
   transition = null;
   controls.enabled = true;
   if (pendingExternal) {
     pendingExternal = false;
     exteriorGroup.visible = true;
     interiorGroup.visible = false;
     engineLight.visible = true;
     plume.visible = true;
     controls.minDistance = 2;
     controls.maxDistance = 45;
     controls.enablePan = true;
   }
 }
}
```
Need when `controls.enabled = false`, `controls.update()` still updates? It might. If not, no matter.

Need set "pendingExternal" initialization:
```
let pendingExternal = false;
```

Need "setModeInternal":
```
function setModeInternal() {
 if (internalMode) return;
 internalMode = true;
 pendingExternal = false;
 shipGroup.position.set(0,0,0);
 shipGroup.rotation.set(0,0,0);
 // hide external groups, show interior
 exteriorGroup.visible = false;
 interiorGroup.visible = true;
 engineLight.visible = false;
 plume.visible = false;
 controls.enablePan = false;
 controls.minDistance = 0.2;
 controls.maxDistance = 5.5;
 updateUI();
 gotoPreset('overview');
}
function setModeExternal() {
 if (!internalMode) return;
 internalMode = false;
 pendingExternal = true;
 updateUI();
 // don't hide interior immediately; after transition
 beginTransition(externalCam, externalTarget);
}
```
Need if `gotoPreset('overview')` calls `beginTransition`; if called after setting interior, good. But `gotoPreset` will transition from external camera to overview. Need if controls.enabled false? beginTransition sets false. Good.
Need "gotoPreset":
```
function gotoPreset(key) {
 const p = interiorPresets[key];
 if (!p) return;
 beginTransition(p.cam, p.target);
}
```
Need if key overview and internalMode true. Good.

Need "updateUI":
```
function updateUI() {
 document.getElementById('btnInternal').style.display = internalMode ? 'none' : 'inline-block';
 document.getElementById('btnExternal').style.display = internalMode ? 'inline-block' : 'none';
 document.querySelectorAll('.internal-controls').forEach(b => b.style.display = internalMode ? 'inline-block' : 'none');
}
```
But when external transition pending and internalMode false, UI hides internal controls and shows external button. During 1.4s transition, user could click external again? It will do nothing because !internalMode. Fine. Could maybe disable buttons during transition:
```
function setButtonsEnabled(bool) {...}
```
Not necessary.

Need "window resize":
```
window.addEventListener('resize', () => {
 camera.aspect = innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth, innerHeight);
});
```

Need "event listeners":
```
document.getElementById('btnInternal').addEventListener('click', setModeInternal);
document.getElementById('btnExternal').addEventListener('click', setModeExternal);
document.querySelectorAll('.internal-controls').forEach(btn => {
 btn.addEventListener('click', () => gotoPreset(btn.dataset.preset));
});
```

Need "makeHullTexture" includes map offset; `hullTexture.wrapS = THREE.RepeatWrapping;` but if map offset.x modifies, need wrap. Good.
Need `CanvasTexture` with `colorSpace`? In r160, texture colorSpace maybe not needed. Default texture encoding no. Fine.

Need "makeLabelSprite" with canvas; need `texture.colorSpace = THREE.SRGBColorSpace`? Not necessary maybe. But for text, okay.

Need "nebula" uses `THREE.SpriteMaterial` and `CanvasTexture`. Need `texture.colorSpace`? no.

Need "updatePlume" needs position attribute. If Points geometry created with setAttribute, `points.geometry.attributes.position.array` works. Need velocities count. Use `for (let i=0; i<count; i++)`.
Need maybe update all particles even if externalGroup hidden? update if !internalMode only.

Need "engine plume" should be in exteriorGroup. But in `setModeInternal`, exteriorGroup.visible=false hides plume. Good.

Need "hullMat.map.offset.x" if hullMat.map not set? yes.

Need "shipGroup.rotation.set(0,0,0)" in internal. If shipGroup.position.y was nonzero from drift, set position 0. But if external transition pending, at start external mode? Wait on click external, internalMode=false and pendingExternal=true. The animation loop now goes to external branch (because !internalMode) so it starts moving shipGroup.position.y and rotation.z again while camera is still inside. This might cause interior orientation shifting during transition. We can avoid by treating `pendingExternal` as not internal? Better use `interiorRendering` state until transition end. Let's define `viewMode`. Simpler:
- Keep `internalMode` = true until transition external complete? Need set false after? The UI can be updated after. Let's adjust.

Use `let internalMode = true/false` controls visibility and UI. But for animation, we need a separate `isInteriorScene` maybe:
- `let sceneMode = 'external'` or `'internal'`.
- `internalMode` can be set when entering internal immediately; when leaving external, maybe set to false at transition end? Hmm.

Let's define:
```
let mode = 'external'; // 'external'|'internal'|'toExternal'|'toInternal'
```
But code complexity.

Maybe simpler:
- On external click: set `mode = 'external'` immediately but keep visibility until transition end using `transition.userData = { showExternalAfter: true }`. The animation loop, if mode external, applies external drift and hull texture offset. Since shipGroup rotation/position slight and hull visible? Wait interiorGroup visible, exteriorGroup hidden during transition; if animation loop external branch sets shipGroup rotation/position, it will move interior group too while camera inside, causing slight drift but okay. If we set shipGroup.rotation.set(0,0,0) only when mode internal, but now mode external so shipGroup rotates. Not too bad. But UI update hides internal controls immediately; okay. At end of transition, show exterior. This is acceptable.

Need perhaps during external transition, `controls.maxDistance` remains 5.5 until transition end, so camera movement from inside to outside could be clamped? OrbitControls update clamps distance to maxDistance? In `beginTransition`, transition updates camera.position directly; controls.update may clamp camera position to maxDistance at each frame? OrbitControls.update likely applies damping and auto-rotate, but does it enforce min/max distance? Yes, if target/radius changes? It has `minDistance`/`maxDistance`; in update, it computes spherical from offset and clamps radius. Our interpolated camera position might be outside maxDistance (if maxDistance 5.5) and controls.update would clamp it, breaking transition. Need set `controls.maxDistance` to larger during transition. We can in `beginTransition` set maxDistance temporarily to 60? Or in `toExternal`, set controls.maxDistance = 60; at end restore. For `toInternal`, maxDistance can be set to 5.5 after transition? If set before, transition from outside to inside: camera starts outside at distance >5.5, controls.update will clamp. Need also temporarily set maxDistance 60. So always in `beginTransition`, set `controls.maxDistance = 60; controls.minDistance = 0.1; controls.enablePan = true;` then at end set appropriate. That avoids clamping during transitions.
Need set `controls.enabled = false` so update might not clamp? But more robust. We'll set maxDistance temporarily in beginTransition; at transition end set min/max.
```
function beginTransition(toCam,toTarget) {
  controls.maxDistance = 60;
  controls.minDistance = 0.1;
  ...
}
```
Then at end:
```
if (pendingExternal) {
  controls.maxDistance=45; minDistance=2; enablePan=true;
} else {
  controls.maxDistance=5.5; minDistance=0.2; enablePan=false;
}
```
Need perhaps if transition toInternal and end, set internal controls max. Good.
In `updateTransition`, after transition done:
```
if (mode === 'internal') { ... } else { ... }
```
Need know pendingExternal. We'll set `pendingExternal` false at end. If not pendingExternal, internal.
```
if (pendingExternal) { pendingExternal=false; exteriorGroup.visible=true; interiorGroup.visible=false; engineLight.visible=true; plume.visible=true; controls.maxDistance=45; minDistance=2; enablePan=true; }
else { controls.maxDistance=5.5; minDistance=0.2; enablePan=false; }
```
For entering internal, `pendingExternal` false and at end we set controls max 5.5. Good.
But if entering internal, we already set exteriorGroup hidden and interior visible; at end enable pan false. Good.

Need "mode" variable maybe:
```
let mode = 'external';
function setModeInternal() {
 if (mode !== 'external') return;
 mode = 'internal';
 interiorGroup.visible = true;
 exteriorGroup.visible = false;
 engineLight.visible = false;
 plume.visible = false;
 shipGroup.position.set(0,0,0);
 shipGroup.rotation.set(0,0,0);
 updateUI();
 gotoPreset('overview');
}
function setModeExternal() {
 if (mode !== 'internal') return;
 mode = 'external?'; // But should remain internal until transition? Need use `mode='transition-to-external'`.
 pendingExternal = true;
 updateUI(); // maybe UI shows external button but internal controls hidden? If mode not internal? Hmm.
 beginTransition(externalCam, externalTarget);
}
```
At transition end, set `mode='external'`. But in animation loop, if mode==='internal' or 'toExternal', should not apply external drift? Let's set:
```
const applyingExternal = (mode === 'external' || mode === 'toExternal');
if (applyingExternal) { shipGroup position y etc } else { shipGroup reset }
```
If mode `toExternal`, we want interior group still visible during transition but shipGroup external drift maybe okay. But if shipGroup position moves, internal camera transition target world coords unaffected? slight. Could set shipGroup position only after external completed? Hmm. To keep simple:
```
if (mode === 'external') {
  shipGroup.position.y = sin... 
} else if (mode === 'internal' || mode === 'toExternal') {
  shipGroup.position.set(0,0,0);
  shipGroup.rotation.set(0,0,0);
}
```
So during toExternal, no drift. At end, mode='external'. Good.
- `updateUI` should reflect mode. During toExternal, UI should show external button? Maybe not; but after transition reset. We can update UI at mode set:
  - setModeInternal: mode='internal', updateUI shows internal controls.
  - setModeExternal: mode='toExternal', updateUI maybe still shows internal controls? But the user has clicked external; maybe buttons shouldn't disappear until transition end. We'll call updateUI after mode set; if mode==='toExternal', maybe display both? Let's simplify:
```
function updateUI() {
 const isInternal = (mode === 'internal' || mode === 'toExternal');
 document.getElementById('btnInternal').style.display = isInternal ? 'none' : 'inline-block';
 document.getElementById('btnExternal').style.display = isInternal ? 'inline-block' : 'none';
 document.querySelectorAll('.internal-controls').forEach(b => b.style.display = (mode==='internal' || mode==='toExternal') ? 'inline-block' : 'none');
}
```
This keeps internal controls visible during toExternal, and external button visible; okay.
But if mode==='toExternal' and user clicks internal preset again, `gotoPreset` would work? It checks maybe no. We can ignore.
At end transition:
```
mode = 'external'; updateUI(); 
```
Need if `mode==='toExternal'` and transition ends, set exteriorGroup visible etc. Good.

Maybe avoid mode var and use `internalMode` boolean plus `transitioningToExternal`; but mode is fine.

Let's code with mode.

Pseudo:
```
let mode = 'external'; // 'external' | 'internal' | 'toExternal'
let pendingExternal = false;
```

`changeToInternal`:
```
if (mode !== 'external') return;
mode = 'internal';
pendingExternal = false;
shipGroup.position.set(0,0,0);
shipGroup.rotation.set(0,0,0);
exteriorGroup.visible = false;
interiorGroup.visible = true;
engineLight.visible = false;
plume.visible = false;
controls.enablePan = false;
updateUI();
beginTransition(interiorPresets.overview.cam, interiorPresets.overview.target);
```
Should `controls.maxDistance` set right after beginTransition? beginTransition sets maxDistance=60, so okay. At end, if mode internal, set max=5.5.

`changeToExternal`:
```
if (mode !== 'internal') return;
mode = 'toExternal';
pendingExternal = true;
updateUI();
beginTransition(externalCam, externalTarget);
```
Still keep interiorGroup visible until end. But `exteriorGroup` remains hidden. At end:
```
if (pendingExternal) {
  pendingExternal = false;
  mode = 'external';
  exteriorGroup.visible = true;
  interiorGroup.visible = false;
  engineLight.visible = true;
  plume.visible = true;
  controls.enablePan = true;
  controls.minDistance = 2;
  controls.maxDistance = 45;
  updateUI();
}
```
Need in updateTransition at end, need know if pendingExternal; if yes. But `pendingExternal` true only for toExternal. For toInternal, false and mode internal. Good.

Need if user clicks internal location during transition to external? Could ignore because mode != internal. But if mode == 'internal' maybe okay. If mode == 'toExternal', no. Buttons internal controls still shown; clicking should do nothing. Maybe okay. Could in listener:
```
if (mode !== 'internal') return;
```
Good.

Need `updateUI` with mode:
```
function updateUI() {
 const inInternalView = (mode === 'internal' || mode === 'toExternal');
 document.getElementById('btnInternal').style.display = (mode === 'external') ? 'inline-block' : 'none';
 document.getElementById('btnExternal').style.display = inInternalView ? 'inline-block' : 'none';
 document.querySelectorAll('.internal-controls').forEach(b => b.style.display = inInternalView ? 'inline-block' : 'none');
}
```
At start mode external: external button hidden, internal button shown, internal controls hidden.
After internal click: mode internal: internal button hidden, external shown, internal controls shown.
During toExternal: same as internal.
After external complete: external mode.

Need "updateTransition" at end:
```
if (transition.t >= 1) {
 transition = null;
 controls.enabled = true;
 if (pendingExternal) {
   pendingExternal = false;
   mode = 'external';
   exteriorGroup.visible = true;
   interiorGroup.visible = false;
   engineLight.visible = true;
   plume.visible = true;
   controls.enablePan = true;
   controls.minDistance = 2;
   controls.maxDistance = 45;
   updateUI();
 } else {
   controls.enablePan = false;
   controls.minDistance = 0.2;
   controls.maxDistance = 5.5;
 }
}
```
Need for internal mode, controls.maxDistance set at end; during transition, beginTransition set 60. Fine.

Need "animation loop" use mode:
```
if (mode === 'external') {
  shipGroup.position.y = Math.sin(time*0.1)*0.25;
  shipGroup.rotation.z = Math.sin(time*0.05)*0.02;
  // maybe update hull texture
} else {
  shipGroup.position.set(0,0,0);
  shipGroup.rotation.set(0,0,0);
}
```
But if mode === 'external' and shipGroup.position y nonzero, OrbitControls target remains (0,0,0), so ship moves slightly. If internal transition starts from external and camera outside, in `changeToInternal` we set shipGroup.position 0 and rotation 0 immediately, causing snap. Could avoid by setting shipGroup position 0 before transitioning? It happens, okay. Or during external drift small, snap negligible. Good.

Need "hull texture offset" should only if mode external? Could always update but hidden. Fine.

Need "plume" visibility:
- `plume.visible` set in mode changes. But if exteriorGroup hidden, plume child hidden anyway; if exteriorGroup visible, plume visible. Need if mode internal, exteriorGroup hidden. Good.
- In animation, updatePlume if exteriorGroup.visible.

Need "starfield drift" maybe always.

Need "engineLight.visible" set in exteriorGroup; if exteriorGroup hidden, no.

Need "createNebula" maybe:
```
function makeNebula(color, pos, scale) {
 const c=document.createElement('canvas'); ...
 const tex=new THREE.CanvasTexture(c);
 const mat=new THREE.SpriteMaterial({map:tex, transparent:true, blending:THREE.AdditiveBlending, depthWrite:false});
 const s=new THREE.Sprite(mat); s.position.set(...); s.scale.set(...); return s;
}
```
Need add 3.

Need maybe "background stars" with star colors? Use `color: 0xffffff`, no. Could use vertex colors? More code. We can use `PointsMaterial({ color:0xffffff, size:0.8, transparent:true, opacity:0.9, sizeAttenuation:true })`. Good.

Need "CSS" pointer events. UI panel pointer-events auto. Buttons.

Need "labels" in interior: Use `makeLabelSprite`:
```
function makeLabelSprite(text) {
 const canvas = document.createElement('canvas');
 canvas.width = 512; canvas.height = 128;
 const ctx = canvas.getContext('2d');
 ctx.clearRect(0,0,512,128);
 ctx.fillStyle = 'rgba(0,20,40,0.85)';
 ctx.beginPath(); ctx.roundRect(8,8,496,112,12); ctx.fill(); // roundRect may not support? Use rect.
 ctx.fillRect(8,8,496,112);
 ctx.font='bold 44px "Microsoft YaHei", sans-serif';
 ctx.textAlign='center'; ctx.textBaseline='middle';
 ctx.fillStyle='#bff0ff';
 ctx.shadowColor='#00aaff'; ctx.shadowBlur=12;
 ctx.fillText(text,256,64);
 const tex = new THREE.CanvasTexture(canvas);
 const mat = new THREE.SpriteMaterial({ map:tex, transparent:true, depthTest:false, polygonOffset:true });
 const sprite = new THREE.Sprite(mat);
 sprite.scale.set(3,0.75,1);
 return sprite;
}
```
Need `ctx.roundRect` maybe supported but to be safe use fillRect.
Labels positions:
- "中央光轴" at (0,2.0,0)
- "居住区" at (-5,1.5,0.8)
- "生态区" at (0,1.5,-1.5)
- "舰桥" at (9,0.5,0)
- "引擎舱" at (-9,0.5,0)
Need if label at y=1.5, within interior. Sprites always face camera, good.

Need perhaps "interior lights" additional point lights at compartments:
```
const p = new THREE.PointLight(0xaaddff, 0.5, 6);
p.position.set(-5, -2, 0); interiorGroup.add(p);
...
```
Could add.

Need "engine glow" exterior and internal? Add.

Need "createEnginePlume" code.

Let's write complete code in final. Need ensure no syntax errors.

Potential issue: In module script, `const externalCam = new THREE.Vector3(...)` used in function. Good.
Potential issue: `interiorGroup` defined after `gotoPreset`? We define before functions; function hoisting okay. But need not call until listeners after definitions.
Potential issue: `document.querySelectorAll('.internal-controls')` returns NodeList, fine.
Potential issue: `controls.enabled = false` during transition; if user clicks while transition? okay.
Potential issue: `controls.update()` might be called twice? Once in animate, once in updateTransition. It's fine; but if controls.enabled false, update may still clamp? Hmm. We need maybe not call controls.update in animate when transition? It's okay. But if OrbitControls update clamps target due to minDistance/maxDistance? It clamps radius. During transition, we set maxDistance=60/min=0.1, so no clamp. Good.
Need "controls.enabled=false" in transition; does update() still execute damping? In three.js, OrbitControls.update does not check enabled; it applies if `enableDamping` etc based on state. It might modify camera based on last spherical delta. Since user input not possible, okay.

Need maybe "renderer.outputColorSpace" default. no.

Need "display internal controls" after mode internal. If initial mode external, updateUI at start:
```
updateUI();
```
Need call after listeners.

Need "hullMat" variable accessible. yes.

Need "makeHullTexture" with repeats:
```
function makeHullTexture() {
 const canvas=document.createElement('canvas');
 canvas.width=512; canvas.height=512;
 const ctx=canvas.getContext('2d');
 ctx.fillStyle='#9aa0a6'; ctx.fillRect(0,0,512,512);
 // panel grid
 ctx.strokeStyle='#555a60'; ctx.lineWidth=2;
 for(let i=0;i<=8;i++) { ctx.beginPath(); ctx.moveTo(i*64,0); ctx.lineTo(i*64,512); ctx.stroke(); ctx.beginPath(); ctx.moveTo(0,i*64); ctx.lineTo(512,i*64); ctx.stroke(); }
 // window strips
 ctx.fillStyle='#8fd8ff';
 for(let row=0; row<4; row++) {
   for(let col=0; col<16; col++) {
      ctx.fillRect(24+col*30, 40+row*120, 16, 8);
   }
 }
 // lights
 ctx.fillStyle='#d94f3d';
 for(let i=0;i<10;i++) ctx.fillRect(30+i*50, 300, 8, 8);
 const tex=new THREE.CanvasTexture(canvas);
 tex.wrapS=tex.wrapT=THREE.RepeatWrapping;
 tex.repeat.set(2,2);
 return tex;
}
```
Need no CSS? okay.

Need "interior floor material" maybe can add texture? no.

Need "fields" maybe color.

Need "renderer.domElement.style.display='block'"? no.

Need "starfield" random distribution:
```
const starPositions = new Float32Array(3000*3);
for (let i=0;i<3000;i++) {
 const r = 300 + Math.random()*500;
 const theta = Math.random()*Math.PI*2;
 const phi = Math.acos(2*Math.random()-1);
 starPositions[i*3] = r*Math.sin(phi)*Math.cos(theta);
...
}
```
This creates shell. Or simpler `(Math.random()-0.5)*1000`; but could be near ship. Shell better.
Need size attenuation true; stars at 300-800, size 0.8 maybe too small. Use size 2? With sizeAttenuation and camera far, okay. Need set `size:1.2`. Use `sizeAttenuation:true`. Good.

Need "nebula sprites" scale big.

Need "engine plume" count 400, size 0.15. Good.

Need "renderer" start after scene. Good.

Need maybe "shipGroup" should be parent of exteriorGroup and interiorGroup; interiorGroup hidden but children not visible.

Need "explore internal parts" maybe maybe use "pointerlock controls"? No.

Need "360 degree view" via OrbitControls.

Need "running instructions" in final.

Need "necessary test description" in final.

Need maybe "搜索并阅读" statement: "我按《极光》原著的设定还原..." Need perhaps include "我没能实时联网；以下根据小说核心设定". Since user specifically asked search; to be honest "无法执行联网搜索" maybe. But final can say "我按金·斯坦利·罗宾逊《极光》中的设定..." Good.

Need maybe "飞船模型还原说明" in final:
- 原著中"Aurora"不是战舰，而是一艘自旋圆柱栖地船（O'Neill cylinder）。
- 因此模型主体为圆柱体，带轴向光管、生态居住区、舰桥、引擎舱、散热鳍/辐射器、推进剂储罐和尾部引擎簇。
Need "百分百" maybe "尽可能还原".

Need maybe "运行方式" list:
1. 将代码保存为 `aurora.html`.
2. 本地启动服务器 `python3 -m http.server 8080`, 打开 `http://localhost:8080/aurora.html` (或直接双击文件，若浏览器允许 module 加载 CDN).
3. 需要联网加载 Three.js r160.

Need "测试说明":
- 加载后能看到深空星云、旋转/飘移船体和尾部粒子。
- 鼠标拖动可360度查看；滚轮缩放。
- 点击“进入飞船内部”后外部船体隐藏，镜头飞到内部；用五个内部按钮切换区域。
- 点击“返回外部视角”后镜头退出并恢复外部船体。
- Resize window works.

Need maybe "故障" if CDN blocked, use unpkg/jsdelivr. Good.

Need perhaps include "maintenance" not.

Now need write code. Need maybe include "importmap" with unpkg. Some browsers don't support importmap? Modern yes. Need maybe use "https://unpkg.com/three@0.160.0/build/three.module.js" and "https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js". Good.

Need ensure `OrbitControls` path in importmap:
```
"three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
```
Then import `three/addons/controls/OrbitControls.js`. Good.

Need maybe "Module script" with `import * as THREE from 'three';` works with importmap. Good.

Need code length okay. Need ensure no comments causing issues.

Let's compose full code.

Before final, let's mentally review code for syntax.

Potential issues with `new THREE.CylinderGeometry(radius, radius, height, radialSegments, heightSegments, openEnded, thetaStart, thetaLength)` parameter list: In Three.js CylinderGeometry `(radiusTop, radiusBottom, height, radialSegments, heightSegments, openEnded, thetaStart, thetaLength)`. Yes.

Potential issue: `geo.rotateZ(-Math.PI/2)` for partial cylinder. For `innerFloorGeo` with `thetaStart=-Math.PI/2, thetaLength=Math.PI`, after rotate, lower half? Let's verify perhaps okay. If not, visual may have floor on wrong side. Could instead use full inner hull and not worry. But if wrong, still okay. Need perhaps choose correct lower half. Let's verify with actual formula after `rotateZ(-90)`: Old cylinder axis y, cross-section old x-z. Vertex old x=R cosθ, old z=R sinθ. After rotate z=-90, new x = old y, new y = -old x = -R cosθ, new z=old z = R sinθ. For θ from -π/2 to π/2: cosθ >=0, so new y <=0 (lower half), and z ranges -R to R. Good. So `thetaStart=-Math.PI/2, thetaLength=Math.PI` yields lower half. Good. Upper half from π/2 to π? `thetaStart=Math.PI/2, thetaLength=Math.PI` yields cosθ <=0, y>=0. Good. Perfect.

Need "innerFloor" radius 4.75, "sky" radius 4.8. Since camera inside radius <4.8, it will see the lower floor and upper sky. Good.

Potential issue with `innerFloor` and `sky` open-ended cylinders with no caps; the end bulkheads close. Good.

Need "buildings" at y=-4.35 with z=0. They sit on the curved floor at bottom. Good.

Need "engine room" reactor at y=-3.4; okay.

Need "bridge" at x=9, y=-3.5; okay.

Need "labels" at y positive (above axis? y=1.5). Since lower floor at bottom, labels near center high. Good.

Need "externalGroup" hides when internal; but if exteriorGroup.visible false, "engineLight.visible" false; no issue.

Need "updatePlume" if exteriorGroup hidden but we still call? In animation:
```
if (plume.visible) updatePlume(plume);
```
If exterior hidden, plume.visible false. But if exterior visible and plume.visible false? We set true when mode external. Use externalGroup.visible. Good.

Need "hullMat.map.offset" update; if hullMat.map absent no.

Need "shipGroup.position.y" external drift; if mode external, shipGroup.position.y changes. But controls target remains 0; shipGroup position changes, so camera not following. Because OrbitControls target is world (0,0,0), and shipGroup offset from target, okay. But if shipGroup.position.y = sin 0.25, the visual ship moves up/down. Good.

Need "mode" state and updateUI at initial:
```
updateUI();
```
Need call after mode variable defined. Good.

Need "internalPresets" defined before `changeToInternal`? It can be after; closures okay if called later. But `changeToInternal` might be called before initialization? no.

Need "precise" code order:
1. Scene setup.
2. Functions/meshes.
3. Define state and functions.
4. Listeners/UI.
5. animate.

Need "modals" no.

Let's draft code fully.

Potential issue: `PointLight` intensity in r160 uses physically correct? default decay 2. In r160, point lights use physical units? Intensity 2 with distance 10 maybe dim. Use higher intensity 5. Need set `decay`? In three r160, `useLegacyLights` default false? Actually since r155, lights use physical units, intensity for point light is candela, so 2 may be dim. Need maybe set `renderer.useLegacyLights = true`? In r160, removed? Hmm. To ensure visibility, use `new THREE.PointLight(color, 50, 30)`? Let's set intensities high:
- AmbientLight 0.8
- DirectionalLight 2.0
- PointLight at core `15, 20`
- EngineLight `10, 15`
In r160, directional intensity 2 okay; point light 15 enough. Need not overdo. MeshStandardMaterial with emissive also.
Need maybe `renderer.toneMapping = THREE.ACESFilmicToneMapping`? Could add. But if lights intensity high, okay. Need avoid washout. Set `renderer.toneMapping = THREE.ACESFilmicToneMapping; renderer.toneMappingExposure = 1.0;` optional. But if using MeshBasicMaterial unaffected? Fine. Maybe not.

Need "hull texture" as map with metalness; if no lights? directional lights enough. Good.

Need "CanvasTexture" map color space? Not necessary but if colors washed, okay. But if using ACES, mesh basic labels unaffected. We'll skip tone mapping to keep simple.

Need "engineLight.visible" initial false? If exterior visible, true. In setup:
```
engineLight.visible = true;
```

Need "plume.visible" true initial.
Need "interiorGroup.visible" initial false.

Need "createEnginePlume" code:
```
function createEnginePlume(count) {
 const positions = new Float32Array(count * 3);
 const velocities = [];
 for (let i = 0; i < count; i++) {
   positions[i*3] = -14.0;
   positions[i*3+1] = (Math.random()-0.5)*1.0;
   positions[i*3+2] = (Math.random()-0.5)*1.0;
   velocities.push({
     x: -0.06 - Math.random()*0.06,
     y: (Math.random()-0.5)*0.02,
     z: (Math.random()-0.5)*0.02
   });
 }
 const geo = new THREE.BufferGeometry();
 geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
 const mat = new THREE.PointsMaterial({ color:0x77ccff, size:0.16, transparent:true, opacity:0.8, blending:THREE.AdditiveBlending, depthWrite:false });
 const points = new THREE.Points(geo, mat);
 points.userData.velocities = velocities;
 return points;
}
```
Need update:
```
function updatePlume(points) {
 const pos = points.geometry.attributes.position.array;
 const vel = points.userData.velocities;
 for (let i=0; i<vel.length; i++) {
  pos[i*3] += vel[i].x;
  pos[i*3+1] += vel[i].y;
  pos[i*3+2] += vel[i].z;
  if (pos[i*3] < -22) {
    pos[i*3] = -14.0;
    pos[i*3+1] = (Math.random()-0.5)*1.0;
    pos[i*3+2] = (Math.random()-0.5)*1.0;
  }
 }
 points.geometry.attributes.position.needsUpdate = true;
}
```
Need if velocities length maybe count. Good.

Need "label" `ctx.fillStyle` etc.

Need "roundRect" not use.

Need "internal labels" maybe `depthTest:false` makes labels visible through walls; but if we look through floor maybe okay. It might be visible from outside? Interior hidden. Fine.
Need "Sprite" scales maybe with three.js units. Labels at y=1.5, scale 3x0.75, good.

Potential issue: The inner hull lower floor with `side: DoubleSide` will be visible from inside. But buildings on it may not be fully illuminated; point light and ambient. Add `PointLight` at core; enough. Maybe add `HemisphereLight` with ground color:
```
scene.add(new THREE.HemisphereLight(0x88aaff, 0x445522, 0.8));
```
Add.

Need "directional light" maybe shadows off.

Need "starfield" PointsMaterial with `sizeAttenuation:true` and size 1.2. At distances 300-800, stars may be subpixel. In three.js, Points size in world units if sizeAttenuation true. Use size 2 maybe. Good. Or set `sizeAttenuation:false`, size 1.5. With camera far, no issue. Let's use `sizeAttenuation:false, size:1.5` for visible stars. But points at all depths same size; fine. Need `transparent:true, opacity:0.9`. Good.

Need "nebula" canvas gradient. Need add `scene.add` after.

Need perhaps "moving starfield" by rotating starField. Points in a shell; rotating okay.

Need "UI button display" CSS initial: external button display none via inline style? In HTML set style display:none; updateUI also. Good.

Need "ID references" ensure.

Need "final response" with code block. It will be long but okay. Need maybe include "代码" only once. Need include running instructions.

Let's write code fully now in my head.

Need maybe if `interiorPresets` is used in `changeToInternal`, define before. I can define after functions but before call.

Detailed code:

```
const interiorPresets = {
  overview: {
    cam: new THREE.Vector3(0, 0.2, 4.3),
    target: new THREE.Vector3(0, -1.5, 0)
  },
  living: {
    cam: new THREE.Vector3(-5, -0.5, 2.8),
    target: new THREE.Vector3(-5, -2.2, 0)
  },
  eco: {
    cam: new THREE.Vector3(0, -0.5, 3.6),
    target: new THREE.Vector3(0, -3.0, 0)
  },
  bridge: {
    cam: new THREE.Vector3(6, -0.8, 3.4),
    target: new THREE.Vector3(9, -2.5, 0)
  },
  engine: {
    cam: new THREE.Vector3(-8, -0.8, 3.4),
    target: new THREE.Vector3(-9, -2.5, 0)
  }
};
```

Need if `gotoPreset` with overview after mode internal: `beginTransition` sets controls.maxDistance=60. At end, controls.maxDistance=5.5. But if camera position distance to target >5.5? Let's compute overview cam (0,0.2,4.3), target (0,-1.5,0): distance sqrt(1.7^2 +4.3^2)=4.62 <5.5. okay. Living cam (-5,-0.5,2.8), target (-5,-2.2,0): distance sqrt(1.7^2+2.8^2)=3.27. bridge 3.42. good.
Need camera view at overview target (0,-1.5,0) maybe looking at floor; good.

Need "controls.maxDistance=5.5" at end; if user zooms out, max distance from target 5.5, camera could be outside inner hull. But maybe okay. Could set 4.6 to reduce. For overview target, max 4.6 allows camera at z=4.2? Distance 4.6. Actually wall distance 4.58; so 4.4 safer. But for living target at (-5,-2.2,0), wall distance along z sqrt(4.8^2 -2.2^2)=4.27. If maxDistance 4.4, camera can zoom to wall. Good. For overview target, maxDistance 4.4 permits z around 4.0, inside. Let's set `controls.maxDistance = 4.4` inside. Our preset overview camera distance 4.62 >4.4, so OrbitControls would clamp after update at transition end. Avoid. Set overview camera to (0,0.2,3.8): distance to target sqrt(1.7^2+3.8^2)=4.16. Inside. Use 4.4 max. But user might want more zoom out; not needed. Let's set max 4.6 and overview cam z=4.0? 4.0 distance sqrt(1.7^2+4^2)=4.35. Good. Use max 4.6. Need if overview target -1.5, wall distance 4.56; 4.6 slightly outside. okay. Better max 4.4. Then overview cam z=3.8. Let's set:
- overview cam (0, 0.2, 3.8)
- living cam (-5, -0.5, 2.8)
- eco cam (0, -0.5, 3.4)
- bridge cam (6, -0.8, 3.2)
- engine cam (-8, -0.8, 3.2)
Targets same. Distances:
overview: sqrt(1.7^2+3.8^2)=4.16 <4.4.
living: 3.27.
eco: target y=-3, cam y=-0.5, z=3.4: sqrt(2.5^2+3.4^2)=4.22 <4.4.
bridge: target y=-2.5, cam y=-0.8, z=3.2: sqrt(1.7^2+3.2^2)=3.62.
engine same 3.62.
Good. Set maxDistance=4.4. But user may want zoom out; okay.
Need if internal labels at y=1.5, camera target y=-2.5, looking up, fine.

Need "controls.enablePan=false" internal, so target stays. Good.

Need "starfield" and nebula might be behind ship; when interior, exterior hidden, but starfield visible through upper sky? Upper sky opacity 0.85, some stars visible, okay maybe.

Need "central light tube" at y=0. It is inside; camera overview target y=-1.5, looking down, central light above. Good.

Need "upper sky" maybe if opacity 0.85, it will obscure stars mostly. Good.

Need "engine room label" at (-9,0.5,0); reactor at y=-3.4; camera at (-8,-0.8,3.2) target (-9,-2.5,0) sees.

Need "exteriorGroup" has hull material with map offset. Need if external group hidden, material map offset continues; no issue.

Potential issue: `hullMat.map.offset.x` modifies map offset but texture repeat set. Need if map offset x increments beyond 1, okay.

Need "translate" no.

Need perhaps "radiators" add structure:
```
function addFin(x,y,z,rotY=0,rotZ=0,width=18,depth=2.2) {
 const fin = new THREE.Mesh(radGeo, radMat);
 fin.position.set(x,y,z);
 if(rotY) fin.rotation.y = rotY;
 if(rotZ) fin.rotation.z = rotZ;
 exteriorGroup.add(fin);
}
addFin(0,5.4,0);
addFin(0,-5.4,0);
addFin(0,0,5.4,0,Math.PI/2); // rotate around z? 
```
Need if at z=5.4 and rotate z=π/2, box dimensions x=18, y=0.08,z=2.2 rotate around z: y and z swap, so thin dimension z, depth y=2.2, still horizontal? Hmm. Let's not. Maybe use three thin boxes as vertical fins:
```
const finMat = new THREE.MeshStandardMaterial({ color:0xb3542a, metalness:0.4, roughness:0.6, side:THREE.DoubleSide});
function addFin(x,y,z,rotX=0) {
 const g = new THREE.BoxGeometry(18, 2.2, 0.08);
 const m = new THREE.Mesh(g, finMat);
 m.position.set(x,y,z);
 if(rotX) m.rotation.x = rotX;
 exteriorGroup.add(m);
}
addFin(0,5.6,0); // vertical fin? Buffer? 
```
Maybe not. We can just add four "radiator panels" as thin boxes:
- Top/bottom: `BoxGeometry(18, 0.08, 2.0)`.
- Left/right: `BoxGeometry(18, 2.0, 0.08)` (vertical plates) at z=±5.6. This creates bottom/side fins.
Let's implement:
```
const radMat = ...
function addRadiator(x,y,z,w,h,d) {
 const r = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), radMat);
 r.position.set(x,y,z);
 exteriorGroup.add(r);
}
addRadiator(0, 5.6, 0, 18, 0.08, 2.0);
addRadiator(0, -5.6, 0, 18, 0.08, 2.0);
addRadiator(0, 0, 5.6, 18, 2.0, 0.08);
addRadiator(0, 0, -5.6, 18, 2.0, 0.08);
```
This is okay.
Need "struts" maybe:
```
const strutMat=...
for (const [x,y,z,w,h,d] of [[-6,5.2,0,12,0.3,0.3],[-6,-5.2,0,...],[ -6,0,5.2,...],[-6,0,-5.2,...]]) add...
```
Could skip.

Need "front docking ring" torus; torus geometry default axis z. After rotateY(π/2), axis x. It will be at x=12 centered on cap. Need `dockRing.position.x = 12`; yes.
Need `dockCap` circle normal +x. Good.

Need "comm dish" maybe:
```
const dishGroup = new THREE.Group();
const dishMat = new THREE.MeshStandardMaterial({ color:0xa8b0b8, metalness:0.4, roughness:0.4, side:THREE.DoubleSide});
const dishGeo = new THREE.SphereGeometry(1.1,24,16,0,Math.PI*2,0,Math.PI*0.5);
const dish = new THREE.Mesh(dishGeo, dishMat);
dish.rotation.x = Math.PI/2; // open side maybe
dishGroup.add(dish);
const feed = new THREE.Mesh(new THREE.SphereGeometry(0.15,8,8), new THREE.MeshStandardMaterial({color:0xffaa44}));
feed.position.set(0.8,0,0); // ?
dishGroup.add(feed);
dishGroup.position.set(7,5.6,0);
exteriorGroup.add(dishGroup);
```
Not necessary. But okay. Need if rotation open? Hard. Let's skip dish to save code? The ship already has radiators, rings, engine.

Need "solar panels" not.

Need "rear nozzle small" with cylinder geometry. Need if `smallGeo.rotateZ` inside loop modifies geometry; use separate each time:
```
const small = new THREE.Mesh(new THREE.CylinderGeometry(0.6,0.8,1.2,16,1,true), engMat);
small.geometry.rotateZ(-Math.PI/2);
small.position.set(-13.2, y, z);
```
Good.

Need "nozzleGeo" openEnded; no caps. Good.

Need "innerGlowGeo" openEnded; no caps. Could use basic material with transparent. Good.

Need "engineGroup" add to exteriorGroup. But if exteriorGroup hidden, engine hidden. Good.

Need "engineLight" maybe at (-14.5,0,0), intensity 12. Need if `PointLight` child of exteriorGroup, its `visible` property can be false.

Need "plume" position at (-14.5,0,0), but particle positions already set. It is child of exteriorGroup. Need if exteriorGroup hidden, hidden. Good.

Need "exteriorGroup" no transform; if shipGroup rotation.z slight, hull rotates slightly. Good.

Need "mode external" shipGroup.position.y sinusoidal; but engine plume particles are in exteriorGroup, which moves with shipGroup; particle local positions unaffected. Good.

Need "interiorGroup" hidden; but if shipGroup rotates, no.

Need "renderer.setAnimationLoop" maybe use requestAnimationFrame. Good.

Need "controls.update" called when controls.enabled false? It may update camera from internal state. It could cause slight jitter during transition; but okay. Maybe in animate:
```
if (!transition) controls.update();
```
But we call controls.update in updateTransition too. Better:
```
if (transition) updateTransition(dt); else controls.update();
```
Then updateTransition calls controls.update at end. But if controls.enabled false, update still. Let's structure:
```
function animate() {
 ...
 if (transition) {
   updateTransition(dt);
 } else {
   controls.update();
 }
 ...
}
```
In `updateTransition`, after setting camera/target, call `controls.update()`.
This avoids double. Need if transition starts, controls.enabled false. At end controls.enabled true. Good.

Need if transition is null but controls.enabled maybe false? no.

Need "OrbitControls" with damping and update called every frame. Good.

Need "updateTransition" uses `controls.update()` while controls.enabled false; it will update but may also apply damping. okay.

Need "beginTransition" set `controls.enabled = false`.

Need "if user clicks during transition" not.

Need "window resizer".

Need "final code" now.

Let's write final answer with explanation, then code. Need maybe not include "search" impossible? It is okay.

Let's ensure code uses `top` variable? no.

Potential issue: "three/addons" import path with unpkg plus importmap: `three/addons/controls/OrbitControls.js` maps to `https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js`. Good.

Potential issue: "importmap" needs to be before module script; yes.

Potential issue: `THREE.CanvasTexture` from `THREE` imported. yes.

Potential issue: "Camera transition while exterior hull hidden" external click: At end, set `exteriorGroup.visible = true;` and `interiorGroup.visible = false;` then updateUI. But before end, mode is `toExternal`; animation loop else branch sets shipGroup position/rotation to 0; okay. When mode becomes external, next frame starts external drift. Good.
Need if at end of transition, `camera.position` is exactly externalCam and controls.target externalTarget. Then controls.update at end with maxDistance 45 may clamp? externalCam distance from target sqrt(15^2+9^2+14^2)=22.6 <45. okay.

Need if entering internal, at transition end controls.maxDistance=4.4, camera distance overview 4.16 <4.4. okay.

Need "controls.enablePan" at end of entering internal false; if user tries pan, no effect. Good.

Need "engineLight.visible" initial true. If PointLight child of exteriorGroup and exteriorGroup visible, engine light contributes even though plume? yes.

Need "renderer" maybe antialias with alpha false. good.

Need "hullTexture" canvas pattern. Need no "source" issue.

Potential issue: `texture.repeat.set(2,2)` with cylinder UV; after rotating geometry, the texture V dimension along length. Repeat 2 means 2 panels along length. U repeat 2 means 2 around circumference. Fine.
Need if map.offset.x += dt*0.004, because U wraps, windows appear to move around circumference. Good.

Need "scene.background = new THREE.Color(0x020510)" sets opaque black-blue. Good.

Need "fog" no.

Need "CylinderGeometry lower half" maybe after rotate, the inner floor's open ends are at x=±10. End bulkheads at ±11, so a gap at ends. Good.

Need "buildings" within ±9.

Need "interiorGroup" add "floor" maybe all inside. Good.

Need "addTree" function with local arrays? Need if tree trunk geometry reused? If create geometries each time, okay. Need use `new THREE.Mesh` multiple. Good.

Let's code interiors:
```
// Fields
const fieldMat = new THREE.MeshStandardMaterial({ color:0x4d7c46, roughness:1 });
for (let i=0;i<5;i++) {
 const f = new THREE.Mesh(new THREE.BoxGeometry(2.2,0.08,1.4), fieldMat);
 f.position.set(i*3-6, -4.65, 0);
 interiorGroup.add(f);
}
// Hydroponic trays
const trayMat = new THREE.MeshStandardMaterial({ color:0x3f8f4f, emissive:0x113322 });
for (let i=0;i<3;i++) {
 const tray = new THREE.Mesh(new THREE.BoxGeometry(1.2,0.14,0.9), trayMat);
 tray.position.set(-1 + i*1.4, -4.66, 1.4);
 interiorGroup.add(tray);
}
```
Need trees:
```
function addTree(x,z,scale) {
 const trunkMat = new THREE.MeshStandardMaterial({ color:0x6b4f3a });
 const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.06*scale,0.09*scale,0.8*scale,6), trunkMat);
 trunk.position.set(x, -4.75 + 0.4*scale, z);
 const leafMat = new THREE.MeshStandardMaterial({ color:0x2f7a48 });
 const leaf = new THREE.Mesh(new THREE.ConeGeometry(0.35*scale,0.7*scale,8), leafMat);
 leaf.position.set(x, -4.75 + 0.8*scale + 0.3*scale, z);
 interiorGroup.add(trunk, leaf);
}
for (let i=0;i<5;i++) addTree(-2.5 + i*1.2, -1.6, 0.8);
```
Need trunk/leaf geometries created each call. okay.
Need "buildings":
```
function addBuilding(x,z,w,h,d,color) {
 const mat = new THREE.MeshStandardMaterial({color, metalness:0.1, roughness:0.7});
 const b = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), mat);
 b.position.set(x, -4.75 + h/2, z);
 interiorGroup.add(b);
}
addBuilding(-6,0,1.8,1.0,1.8,0xc8cdd2);
addBuilding(-4,0,1.8,1.0,1.8,0xb4b9bf);
addBuilding(4,0,1.8,1.0,1.8,0xd0d5d9);
addBuilding(6,0,1.8,1.0,1.8,0xbcc1c7);
```
Need if building at y=-4.25, height 1.0. Good.
Need "reactor":
```
const reactor = new THREE.Mesh(new THREE.SphereGeometry(0.85,20,20), new THREE.MeshStandardMaterial({ color:0xffaa33, emissive:0xff5500, emissiveIntensity:0.8 }));
reactor.position.set(-9, -3.4, 0);
interiorGroup.add(reactor);
const reactorBase = new THREE.Mesh(new THREE.CylinderGeometry(0.9,1.0,0.4,12), new THREE.MeshStandardMaterial({color:0x555}));
reactorBase.position.set(-9, -4.55, 0);
interiorGroup.add(reactorBase);
```
Need "bridge":
```
const bridgeMat = new THREE.MeshStandardMaterial({ color:0x445566, metalness:0.5, roughness:0.4 });
const bridge = new THREE.Mesh(new THREE.BoxGeometry(3.0,1.8,2.6), bridgeMat);
bridge.position.set(9, -3.5, 0);
interiorGroup.add(bridge);
const bridgeWindow = new THREE.Mesh(new THREE.PlaneGeometry(2.0,0.9), new THREE.MeshBasicMaterial({ color:0x66ccff }));
bridgeWindow.position.set(9, -3.3, 1.31);
interiorGroup.add(bridgeWindow);
```
Need plane at z=1.31, facing z? Plane default normal +z, so visible from camera at negative? It faces +z. If camera inside looking from z negative? Actually camera positions at z=3.4, looking target x=9,y=-2.5,z=0; bridgeWindow at z=1.31 faces +z (toward camera), good.

Need "storage tanks":
```
const tankMat = new THREE.MeshStandardMaterial({ color:0x8a94a0, metalness:0.6, roughness:0.4 });
for (const [x,z,r,h] of [[-3,-1.6,0.5,1.4],[3,1.6,0.5,1.4]]) {
 const tank = new THREE.Mesh(new THREE.CylinderGeometry(r,r,h,12), tankMat);
 tank.position.set(x, -4.75 + h/2, z);
 interiorGroup.add(tank);
}
```
Need "labels":
```
function makeLabelSprite(text) { ... }
const labelDefs = [
 ['中央光轴', 0, 2.2, 0],
 ['居住区', -5, 1.2, 0.8],
 ['生态区', 0, 1.2, -1.5],
 ['舰桥', 9, 0.8, 0],
 ['引擎舱', -9, 0.8, 0],
];
for (const def of labelDefs) { const spr = makeLabelSprite(def[0]); spr.position.set(def[1], def[2], def[3]); interiorGroup.add(spr); }
```
Need if labels at y=1.2, inside cylinder radius 2.2? Fine.
Need `makeLabelSprite` uses `depthTest:false`; labels visible through ship? Since interior hidden externally. fine.

Need "nebula" code:
```
function makeNebula(color, pos, scale) {
 const canvas=document.createElement('canvas'); canvas.width=256; canvas.height=256;...
 const gradient = ctx.createRadialGradient(128,128,0,128,128,128);
 gradient.addColorStop(0, color);
 gradient.addColorStop(0.4, color.replace('1)', '0.3)')); // hack? Not safe.
 gradient.addColorStop(1, 'rgba(0,0,0,0)');
 ctx.fillStyle=gradient; ctx.fillRect(0,0,256,256);
 const tex = new THREE.CanvasTexture(canvas);
 const mat = new THREE.SpriteMaterial({ map:tex, transparent:true, blending:THREE.AdditiveBlending, depthWrite:false });
 const sprite = new THREE.Sprite(mat);
 sprite.position.copy(pos);
 sprite.scale.set(scale.x, scale.y, scale.z);
 return sprite;
}
```
Parameter color as CSS rgba. Use:
```
scene.add(makeNebula('rgba(40,80,180,0.5)', new THREE.Vector3(-120, 60, -150), new THREE.Vector2(140,90)));
```
Need Sprite.scale set with Vector2? Sprite scale is Vector3. Use `new THREE.Vector3(140,90,1)`. Good.
Could use `color.replace` not. Let's specify gradient stops:
```
gradient.addColorStop(0, color);
gradient.addColorStop(0.5, 'rgba(30,60,120,0.15)');
gradient.addColorStop(1, 'rgba(0,0,0,0)');
```
For each color, pass main color and fade color? Simpler:
```
function makeNebula(rgba, fadeRgba, pos, scale) ...
```
But too much. We can create:
```
function makeNebula(color, fadeColor, pos, scale) {
 const c = document.createElement('canvas');
 ...
 const gradient = ctx.createRadialGradient(128,128,0,128,128,128);
 gradient.addColorStop(0, color);
 gradient.addColorStop(1, fadeColor);
 ctx.fillStyle=gradient;
 ...
}
```
Call:
```
makeNebula('rgba(80,160,255,0.6)', 'rgba(0,0,0,0)', new THREE.Vector3(-120,60,-150), 130,90)
```
Need gradient stop with rgba transparent okay. Good.
Need function signature:
```
function makeNebula(color, pos, scale) { const gradient...; gradient.addColorStop(0,color); gradient.addColorStop(1,'rgba(0,0,0,0)'); ...}
```
Then use color with alpha 0.6:
`rgba(120,180,255,0.5)`. Good.

Need "starField" with `new THREE.PointsMaterial({ color:0xffffff, size:1.5, sizeAttenuation:false, transparent:true, opacity:0.9 })`. fine.

Need "renderer.setPixelRatio(Math.min(window.devicePixelRatio,2))".

Need "style" `#info` and `#ui`.

Need perhaps "canvas" appended after UI? In HTML, buttons appear above canvas? Actually code appends renderer.domElement to body after script? If script in head? It's in body after UI, so renderer canvas appended after UI, but UI has z-index 10; canvas default z-index auto, so UI on top. Good.

Need "module script type importmap" inside body? It can be in head. But to be valid, importmap must occur before module script. If in body, okay. We'll place in head? Actually script type importmap can be in head. Then module in body. Use:
```
<head> ... <script type="importmap">...</script></head>
<body> ... <script type="module">...</script></body>
```
Good.

Need "if importmap unsupported" not.

Potential issue: `OrbitControls` from `three/addons` in r160 uses `three` module. importmap maps three. Good.

Need "controls.enabled=false" during transition, but if user clicks inside location while mode internal but transition ongoing? no.

Need "updateUI" at end of transition to external, but if mode changed? okay.

Let's craft final response in Chinese. Include "运行方式". Use final.

Need maybe ensure final answer not too long? It will be long due code. But okay. Need "desired oververbosity 5" moderate. The code dominates.

Let's write final now.
