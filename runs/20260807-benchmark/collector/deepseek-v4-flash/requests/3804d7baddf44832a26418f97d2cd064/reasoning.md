We need answer in Chinese likely. Need create single HTML file with Three.js 3D contact lens case model. Need L/R labels, lid, click lid to open. Need use Three.js. Need deliver code and run instructions. Need think through implementation.

Need build self-contained HTML. Use CDN three.js. Need interactive: OrbitControls? click lid to open. Need maybe raycasting on lid. Model: contact lens case: cylindrical base, two compartments, lids? Actually contact lens case typically two small containers with L/R caps? But request "隐形眼镜盒" with L/R markings and lid. Could be dual lens case with two screw caps? But says "盒盖" singular maybe overall lid? Need interpret. Need user click lid to open. Could be a box with hinged lid. Need 3D model with L/R labels inside? Let's plan.

Use three.js scene. Create a "contact lens case" as a rounded box/case, lid on top hinged. Click lid rotates open revealing two lens holders labeled L/R. Or create two cylindrical containers and lids? "眼镜盒应有醒目的 L 和 R 标识，并配有盒盖。此外，用户应能点击盒盖将其打开。" Maybe contact lens case is a flat rectangular case with two compartments and one hinged lid. That is common. Need L and R markings on inner surface? "醒目" maybe on lid outside? Actually contact lens cases have L and R on the inside compartments to distinguish left/right lenses. Need lid open to see L/R. Or labels on top? Let's design rectangular case with hinged lid openable; inside has two circular recesses for lenses, with L/R labels.

Need use Three.js import maps? Simpler: use script tags from CDN with three and OrbitControls in UMD? Modern three r128? We need robust.

Use ES modules from unpkg? Since single file can use importmap. Need run. If offline? Could use CDN. Use <script type="importmap"> { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js", "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/" } } </script>. Then module script. Need ensure Works. Use CSS full screen.

Could use three.js from cdnjs? Simpler no modules? Use classic script with three.min.js and OrbitControls.js. But current CDN versions? Use https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js and https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js. That should work. But need text labels in 3D. Use CanvasTexture with sprites or geometry. Need create text.

Let's use r128 because examples/js available. But text can be created with canvas sprite. Use raycasting for lid. r128 raycasting with Sprite? We'll raycast mesh.

Need create durable model.

Let's design geometry:

Case base:
- BoxGeometry(width=6, height=1.2, depth=4), at y=0? Let's define y up. Base bottom at y=0? Use coordinate: base box center at y=1, height=2 maybe. Lid at top. Hinge along back edge (z negative? Actually closed lid top).

Better define center of case at origin, lid on top. Base cuboid width=5, depth=3.5, height=1.0. Center y=0? Let's set base center y=0, top y=0.5. Lid is another slightly larger cuboid width=5.2, depth=3.7, height=0.3, center y=0.65 when closed (sits atop base). Hinge at back edge: we need lid rotate around hinge axis along x at back edge (z = -depth/2 perhaps). Need pivot group at back edge. If lid group origin at hinge line (x=0, y=0.75, z=-depth/2). Then lid geometry offset so closed position has lid body extending from hinge to front and upward. Rotating around x negative? Let's define.

Use a pivot Group at hinge position: (0, baseTopY + maybe, -depth/2). Set position. Lid mesh is a child with offset such that closed orientation: center of lid at (0, 0.65? need relative). Let's compute.

Base: BoxGeometry width=5, height=1, depth=3.5, center y=baseCenterY = 0.5 maybe, bottom at 0. Top at 1.0. So base occupies y 0 to 1. Let's set base center y=0.5.

Lid closed: Lid box width=5.2, height=0.4, depth=3.7. It sits on top of base, maybe lower y=0.95 to 1.35. Center y=1.15, z=0. Hinge at back edge z=-1.85? Actually lid depth=3.7, center z=0, so back edge z=-1.85; hinge line at x-axis at z=-1.85, y maybe 1.15. If pivot at (0, 1.15, -1.85). Lid mesh offset relative to pivot: When closed, its world center should be (0,1.15,0). Since pivot at z=-1.85, offset is (0,0,+1.85). So place lid mesh at local position (0,0, depth/2=1.85). Rotate pivot 0 => closed. To open, rotate pivot around x by positive angle? Need axis x; rotating +around x: z' = z cos - y sin? Hmm y and z. For a point offset (0,0,d) in local coordinates. Rotating around positive x by θ (right-hand: y -> z? Actually x rotation by θ: y' = y cosθ - z sinθ, z' = y sinθ + z cosθ. For offset (0,0,d): z'=d cosθ, y'=-d sinθ. Hmm positive θ moves point downward if y=0,z=d. To open upward (front edge up, back hinge fixed), we need rotate around negative x? Let's set pivot rotation.x = -angle? Let's derive: If hinge at back, lid rotates upward: front edge (z=+d) should rise (y positive) and move backward/up? y = -d sinθ. For θ negative, -d sin(-a)=d sin a positive. Good. So rotation.x = -a to open. The lid rotates around x axis clockwise? It will open.

Need add a "cap" maybe a lid. Click lid toggles open/closed. Use animation to smoothly rotate. Add opening using lerp target angle. Need avoid if lid clicked? Use raycast. Need also detect clicking lid only maybe. But base also could be clicked? We can add transparent clickable? Use lid group children. Need raycaster with meshes. OrbitControls may interfere. Need distinguish click vs drag. Implement pointerdown/up with small distance to detect click. Use Three.js event? We can use DOM pointer events.

Need add "L" and "R" markings. Could create text sprites or canvas textures inside base top. Need "醒目" maybe on base top surface. Place two circular indentations / lens seats, each with L/R text. Because when lid opens, user sees L/R. Also maybe on lid outside? But if lid closed, maybe L/R not visible. "醒目的 L 和 R 标识" likely inside. But we can also add L/R on the front of lid or base? Hmm contact lens cases have L/R on the corresponding removable caps. But here one lid. We can put large L/R on the inner top of the lid? Wait inside base has two lens cases, labeled L/R. We'll create.

Need model details:
- Base: plastic material (maybe light blue/white). Use MeshStandardMaterial or Phong.
- Lid: colored slightly translucent? We can make opaque.
- Interior: top surface has two circular openings/counters? Simpler: Create two circular "wells" (cylinders) protruding from base top? Actually contact lens case has two small circular cups (left/right) with screw caps. Could model two white plastic holders with lids? But the request says "盒盖" singular. Hmm maybe "contact lens case" is actually a folding case with a lid. But to be safe, show a dual lens case with two separate compartments and one overall hinged lid. Inside base, create two circular cylindrical "lens holders" (open basins) each with L/R label. Use cylinders with open top; maybe ring at bottom.

Let's make base top surface with two cylindrical recesses? Instead of boolean geometry, can use two circles on top platform? Create shallow ring and inner circle with text. Could be acceptable.

Use simple geometry: Base box; on top, add two small cylinders (height 0.2) representing lens seats? But if they sit on top of base, lid would hit them. Need lid interior maybe has cutout? Simpler: Make base box top at y=1.0; add two CylinderGeometry radius=1.0, height=0.35, center y=1.175, base top? So they protrude above base surface; lid closed at y=0.95? Not possible. Instead add recessed holes by creating cylinders above and below? We can use "visual recesses": add black/darker circles at top surface. Use CircleGeometry at y=1.001 with dark material. Then L/R text sprites at y=1.01 inside each circle. This is easy. No actual holes, but visually looks like compartments. Need maybe add raised rims? Add Torus or ring around circles to simulate raised edge. Ring geometry: ring radius=1.0, tube=0.08 at y=1.0. Use MeshStandardMaterial. That looks like rim. The inside circle is dark. Good.

Need add L/R label via CanvasTexture on PlaneGeometry or Sprite. Place at center of each circle. Need create label texture with transparent background and large bold letter. Use Sprite to always face camera? But "醒目" can be a text plane on top that lies flat. Sprite would be visible from all angles, but when looking from above, okay. It might be visible through? Use MeshBasicMaterial with map, transparent. Plane geometry on top at y=1.02, oriented horizontal (rotation.x=-Math.PI/2) with canvas texture. That lies flat. Need ensure texture orientation maybe. When view from above, letter appears. Use plane with CanvasTexture. But if plane is horizontal, and camera is above, okay. It will be hidden when lid closed because lid covers it. Good. Also maybe use CSS3DRenderer? Not needed.

Need labels L and R on top of lid maybe? We can put "L R" on the lid outside maybe? If lid closed, maybe visible on top. But the lid is a single box, perhaps with "L" and "R" labels on top to show which side is left/right. Add two sprites or planes on top surface of lid: left side "L", right side "R" maybe corresponds to compartments. But the lid covers them when opened? Wait closed top is outside, open rotates to back; inside of lid faces upward? Need if opened, inside of lid (bottom surface) is visible? We can add labels to both top and bottom? Maybe "L" and "R" on front edge/top? Let's include large L/R on inner base and also on lid outer top. That satisfies.

Need maybe "盒盖" with click to open. We can implement lid pivot group with lid body. Clicking lid toggles open/closed. Need raycast. Need handle if open, click lid maybe close. But if lid is tilted backward, raycast works.

Need create "hinge" cylinders perhaps at back edges. Add two small knuckles? Optionally.

Need create scene:
- renderer, camera, lights, ground/grid maybe.
- OrbitControls for rotate/zoom.
- model.

Need likely use function createTextureLabel(text, color, bg?) generate CanvasTexture. Use font bold 800 "Arial". We'll make transparent background with large letter colored. Maybe add subtle shadow.

Need create "L" and "R" on top of base:
- CircleRadius=1.25. Center positions: x = -1.6, +1.6 maybe. Base width=5, depth=3.5. Need fit. Use left x=-1.6, right x=1.6, z=0. Separate circles radius 1.0.
- Ring rim: TorusGeometry(radius=1.0, tube=0.08, radialSegments=32, tubularSegments=64) rotated x=PI/2 to lie horizontally. Position y=1.0.
- interior circle: CircleGeometry(radius=0.95, segments=64), rotated x=-PI/2? Note plane geometry faces default +z. To lie flat on top (normal +y), set rotation.x=-Math.PI/2. Then at y=1.01. Material dark gray/blue.
- labels: Text plane geometry width=0.8, height=0.8, placed at x=-1.6, z=0, y=1.02, rotation.x=-Math.PI/2. But if plane rotated, text can be mirrored? Need set orientation? Let's verify: PlaneGeometry in XY plane facing +z. Rotate by -PI/2 about X maps +z normal? R_x(-90): y -> z? Let's derive: point (0,1,0) (top of plane) becomes (0,0,-1)?? Actually R_x(-90) rotates y to z? Point (0,1,0) -> y'=cos(-90)*1=0, z'=-sin(-90)*1=1? Wait formula y'= y cosθ - z sinθ, z'= y sinθ + z cosθ. θ=-90: y'=0, z'=-1. So top of plane goes to -z. The plane lies horizontal with normal +y? The +z normal? Plane normal is (0,0,1): R_x(-90): z'=0, y'=0? Formula y'=0*0 -1*(-1)=1, z'=0*0+1*0=0 => normal +y. Good. The plane's x axis remains x, y axis maps to -z? So the original top direction (positive y) points to -z. Thus if we draw "L" with canvas, text top is -z, okay. Looking from above (camera +y), text left/right? Need not worry too much.

Maybe use Sprite instead of plane: Sprite always faces camera; when camera above, labels readable. But when lid opened, if camera below, sprite also faces camera? For SpriteMaterial, it always faces camera, so label on base will be visible regardless of view. It might be okay, but if looking from side, a label standing vertical? Sprite is billboard, not flat. Might be less realistic. Use plane for realism. Need maybe use text on plane.

Could also use CanvasTexture with text "L" and add to plane. Need set text use fillText and maybe stroke.

Need "case base" material maybe a pastel teal/blue. Use MeshStandardMaterial with roughness 0.5. Add edges? Maybe use Rounded? Not necessary.

Need lid:
- Use BoxGeometry(width=5.4, height=0.4, depth=3.9). Slightly larger than base. Center of lid relative to pivot. We need make lid group. But note "lid" in common contact lens case is a thin cover attached to back, not full cap. We'll use a box with rounded? Could use "Capsule"? But box is fine. To make more polished, use RoundedBoxGeometry? Not available. Use Box with chamfer? Not needed.

Need add a handle/tab on lid front to indicate click. Use small sphere/box at front edge. Also maybe knob to click. Create mesh "lidKnob" in lid group at front center. When raycasting, include all children. Need animate. Could add label "Open" maybe.

Need pivot group:
- position (0, 1.15, -1.85) if base center y=0.5 and height=1? Wait base top y=1.0. Lid box height=0.4, center when closed y=1.2? Let's set base bottom y=0, top=1.0. Lid should close just above base with gap? If lid lower at y=1.0, center y=1.2 (height 0.4). So pivot y=1.2? Hinge line is at center height of lid, y=1.2, z=-1.9. If lid is a box length in z from -1.9 to 1.9, center z=0 relative to pivot offset +1.9. Good. Set pivot position (0, 1.2, -1.9). But when lid rotates, back edge at z=-1.9 fixed. It will rotate around its center line at back edge. Nice. Need lid box local center (0,0,1.9). Width x 5.4, height y 0.4, depth z 3.8. It will extend from z=0 to 3.8 in local coordinates? Actually center at z=1.9, depth 3.8 -> from z=0 to 3.8. At pivot local z=0, world z = pivot.z = -1.9, so back edge at -1.9. Front local z=3.8 -> world z=1.9. Great. But there is a gap at hinge? Because center of lid y=0 in local, so the lid vertical center at y=1.2. The bottom of lid local y=-0.2 -> world y=1.0, touches top of base. Good.

When rotate pivot.rotation.x = -openAngle (e.g., -PI/2? 100 degrees). The front edge will move up/back? For local front z=1.9; y' = -1.9 sin(-1.2?) for angle a = 1.2 rad (~69°), y=1.9*sin(1.2)=1.77, z'=1.9*cos(1.2)=0.69 -> world z=-1.21. Good. It clears. Could use 110 degrees (1.9 rad) maybe too far. Use max 1.25 rad (~72 degrees), enough. Need not clip into camera. Use 1.1 rad maybe.

But if pivot at y=1.2, opening with negative rotation lifts front edge; the lid plane rotates so it goes from horizontal to tilted upward/back. Good.

Need lid interior surface? For a box, inside surface? Since it's a solid box, not hollow. But if closed, its bottom face is visible when opened? Actually if opened 72°, the bottom face faces upward/back, not fully visible from front. It's a panel with thickness. We can add an inner panel? Maybe not necessary. But for realism, the lid underside could be visible. We can add inner lid surface maybe a plane slightly below bottom face? But raycast? Let's create lid group with:
- main lid box: material maybe a light color (e.g., gray-blue). 
- lid cushion inside? Add a thin PlaneGeometry on the bottom face? The bottom face is local y=-0.2. If placed at y=-0.199, facing down? Actually plane default normal +z; to face down (-y), need rotation.x=+PI/2? Let's think. We'll add a plane on bottom of lid, normal -y, colored / dark, with labels? To be visible when lid opened, from front, the underside (local -y) faces? Let's set lid group local orientation: closed state world same as local: x right, y up, z forward (toward front). The bottom face of lid is world -y. When opened (rotation around x -a), the bottom face normal rotates? The lid's -y normal after R_x(-a): For local normal n=(0,-1,0), R_x(-a) gives y'=-cos(-a)=-cos a, z'=-sin(-a)*(-1)? Wait formula z' = y sinθ + z cosθ = (-1)*sin(-a)= sin a. So n=(0,-cos a, sin a). For a=1.1, n=(0,-0.45,0.89). So bottom face normal points up/back. Thus visible from camera if camera at front/up. Good. We can add a dark plane or label on bottom maybe.

Maybe add "L R" on lid underside? Actually contact lens case lid inside is just a flap. We can leave.

Need create "box" separately. Let's create base as a group:
- baseBox: BoxGeometry(5, 1, 3.5), center at y=0.5. Actually if base bottom y=0, center=0.5.
- corners rounded? Use BoxGeometry not rounded. Maybe apply bevel? `RoundedBoxGeometry` from examples/jsm/geometries/RoundedBoxGeometry.js`? r128 includes RoundedBoxGeometry? Yes examples/jsm/geometries/RoundedBoxGeometry.js. But to keep simple, use BoxGeometry. We can add shadows.

Need perhaps "contact lens case" should be rectangular with rounded corners. RoundedBoxGeometry would make nicer. We can import from `examples/jsm/geometries/RoundedBoxGeometry.js` using importmap. But if we use r128 UMD, we can't import. Better use ES modules. But maybe use RoundedBoxGeometry for lid and base. Let's decide.

Option 1: Use latest ES modules with importmap. It supports RoundedBoxGeometry, OrbitControls. Need include importmap. Code:
```
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
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
...
</script>
```
Need file://? import maps require internet. Good. But if run without internet, won't work. But requirement likely accepts CDN. Need mention run by opening html with internet. Could also use local? We only provide one file, so CDN.

Question says "不要查看其他文件，直接创建" from prompt "Do not look at other files, directly create." It just wants single file. So yes.

Need choose version. Use `three@0.160.0` because importmap known. But RoundedBoxGeometry in examples/jsm/geometries/RoundedBoxGeometry.js from unpkg works? Need check path. In three.js 0.160.0, RoundedBoxGeometry path is `examples/jsm/geometries/RoundedBoxGeometry.js` (capitalized). Yes. OrbitControls path is `examples/jsm/controls/OrbitControls.js`. Good. Use import map to `three/addons/` mapping. Need no modules errors.

Could avoid RoundedBoxGeometry to reduce. But nice. Need make sure `RoundedBoxGeometry` supports `width,height,depth,segments,radius`. Use:
```
new THREE.RoundedBoxGeometry(5, 1, 3.5, 3, 0.15)
```
However RoundedBoxGeometry from examples may have rounded edges only, not "smooth" if segment small. It's fine.

Need base top at y=1? If rounded box height=1, center y=0.5; top y=1.0? With radius, top face at center+0.5. Good. Slight radius changes but okay.

Need lid rounded box length 5.4, height=0.4, depth=3.8, radius=0.08? But pivot local offset center with depth 3.8. If RoundedBoxGeometry extends to ±half depths; okay.

Need use `MeshStandardMaterial` with `roughness`, `metalness`.

Need materials:
- baseMat: color 0x8fd6d6 (teal), roughness 0.4, metalness 0.05.
- lidMat: color 0x6ec9c9? Slightly different. Or translucent dark? Let's use cyan-white.
- interiorMat: color 0x1c3b3b? for recesses.
- rimMat: white maybe.

Need also add an environment or Ground with shadow. Add hemisphere light, directional light. Use `renderer.shadowMap.enabled = true`. Need set mesh castShadow and receiveShadow.

Need "click lid to open" can be done by raycasting against `lidGroup` children. Use pointerdown and pointerup. But OrbitControls rotates camera; click may also rotate. Use pointerup with movement threshold. Need detect if pointerup on lid and camera not dragged. Implementation:
```
let downX=0, downY=0, dragging=false;
renderer.domElement.addEventListener('pointerdown', e => { downX=e.clientX; downY=e.clientY; dragging=false; });
renderer.domElement.addEventListener('pointermove', e => { if (down && distance>5) dragging=true; });
renderer.domElement.addEventListener('pointerup', e => {
 if (!dragging) { handleClick(e); }
 down=false;
});
```
But `pointermove` with OrbitControls rotates continuously; event fires. Need set dragging if move distance >5. On pointerup, if distance <=5, click. Use clientX/Y.

HandleClick:
```
const rect = renderer.domElement.getBoundingClientRect();
const pointer = new THREE.Vector2(((e.clientX-rect.left)/rect.width)*2-1, -((e.clientY-rect.top)/rect.height)*2+1);
raycaster.setFromCamera(pointer, camera);
const intersects = raycaster.intersectObjects(lidGroup.children, true);
if (intersects.length > 0) { toggleLid(); }
```
Need include lidGroup itself maybe but children enough. Need if clicking lower layer? Only lid. Use `raycaster.intersectObjects(lidGroup.children, true)`. But if lid is open, front/back? okay. The lid is made of meshes. If click on knob, toggles.

Need if the lid is closed and center covers base, raycast on lid top. Good.

Need animate lid:
```
let lidAngle = 0; // radians open
let targetLidAngle = 0;
let isOpen = false;
function toggleLid(){
 isOpen = !isOpen;
 targetLidAngle = isOpen ? 1.15 : 0;
}
function animate() {
 lidGroup.rotation.x += (targetLidAngle - lidGroup.rotation.x) * 0.1; // but rotation.x is negative open, target negative
}
```
Wait rotation.x = -angle. Need target = isOpen ? -1.15 : 0. Use `rotation.x += (target - rotation.x)*0.1` with delta? Since animation frame not necessarily 60fps but fine. Use `THREE.MathUtils.lerp`. In r128 name `THREE.Math.lerp`? Use `targetLidAngle = isOpen ? -1.15 : 0; lidGroup.rotation.x += (targetLidAngle - lidGroup.rotation.x) * 0.12;`. But with negative? If current -1, target 0, difference 1, rotation.x increases to 0. Good. If target -1.15, current 0, diff -1.15, rotation decreases. Good.

Need maybe set `lidGroup.rotation.x = 0` initial.

Need add labels L/R on base top. Let's implement `createTextSprite` or `createTextPlane`.

Canvas texture:
```
function createLabelTexture(letter, color = '#ff3b30') {
 const canvas = document.createElement('canvas');
 canvas.width = 256;
 canvas.height = 256;
 const ctx = canvas.getContext('2d');
 ctx.clearRect(0,0,256,256);
 ctx.fillStyle = color;
 ctx.font = 'bold 180px Arial, sans-serif';
 ctx.textAlign = 'center';
 ctx.textBaseline = 'middle';
 // Maybe add white stroke to stand out
 ctx.lineWidth = 12;
 ctx.strokeStyle = 'rgba(255,255,255,0.8)';
 ctx.strokeText(letter, 128, 128);
 ctx.fillText(letter, 128, 128);
 return new THREE.CanvasTexture(canvas);
}
```
Need label plane:
```
const texture = createLabelTexture('L');
const mat = new THREE.MeshBasicMaterial({ map: texture, transparent: true, depthWrite: false, side: THREE.DoubleSide });
const plane = new THREE.Mesh(new THREE.PlaneGeometry(0.9, 0.9), mat);
plane.rotation.x = -Math.PI / 2;
plane.position.set(-1.5, 1.02, 0);
baseGroup.add(plane);
```
Need maybe "R" for right. Color red? L blue? Use "L" green/blue, "R" red. On label inside, maybe "L" and "R". Need use different colors? okay.

Need also add L/R labels on top of lid? If lid group, top face world changed when open. Need place label planes on top of lid at positions corresponding to left and right. But lid is a child group; can add two planes on top surface in local coords. Top surface y = +0.2 (relative to lid group). Place planes y=0.201, z coordinate? Lid local coordinate: center z=1.9, depth from 0 to 3.8. Top surface extends x ±2.7, z 0..3.8. Need if labels on top of closed lid: left label at x=-1.5, z=1.2; right at x=1.5, z=1.2 maybe. These correspond to left/right compartments. Add planes horizontal? Top face of lid is local +y, so plane normal +y: `rotation.x = Math.PI/2`? Need default plane normal +z, to rotate to normal +y? We used -PI/2 for normal +y. Does that make top of plane face +y. For a plane placed on top of lid, yes use rotation.x=-Math.PI/2. But if text top direction maps to -z, so when closed, text "L" reads front/back correctly? We need verify. The lid local z axis points forward (toward front when closed). If plane top (canvas up) maps to -z? Let's see with rotation.x=-PI/2, plane's +y (canvas top) maps to -z. So on top of lid, the letter top is toward back (z negative), so when looking from front, text appears upside down? Wait if canvas texture text "L", the top of the character is along canvas +y. When viewed from camera +y looking down, what is screen orientation? Camera's view up maybe +z? Need complicated. To avoid, use Sprite on top of lid, always faces camera. But if lid rotates, sprite orientation? Sprite always faces camera, so labels readable regardless. But adding a sprite on top of lid is okay. However on base, sprite after lid opens can also face camera. Maybe use Sprite for all labels for readability? But the "L/R" inside should be physically on base; sprite billboard is acceptable for a stylized 3D model. But it might look like floating if viewed from side? Sprites are always perpendicular to camera, but anchored at position; not physically lying flat. We can use sprites with `sizeAttenuation: true`, `scale = (1,1,1)`. That guarantees "醒目的 L/R". For realism, maybe not. But the user likely wants visible labels. Using sprites is acceptable.

Alternative use CSS2DRenderer? No.

Let's use text planes with fixed orientation, and if it appears mirrored, can fix by setting texture `center`? Hmm. Let's reason carefully to place horizontal text plane correctly.

A THREE.PlaneGeometry lies in the XY plane, with +X right, +Y up. Its normal is +Z. CanvasTexture is placed with (0,0) bottom-left? Actually texture coordinates: u corresponds X (right), v corresponds Y (up). The canvas's top-left corresponds v=1, bottom-left v=0. But geometry's top is +Y. So if you view plane from +Z side, the texture appears upright. Rotating plane by R_x(-90) maps normal +Z to +Y (as used). The local +Y axis (up in texture) maps to world -Z? Let's compute R_x(-90): matrix:
[1 0 0; 0 cos(-90) -sin(-90) = [0, 1? Wait -sin(-90)=1], 0 sin(-90) cos(-90)= [-1, 0]? Let's write:
x' = x
y' = y cosθ - z sinθ
z' = y sinθ + z cosθ
θ=-90: cos=0, sin=-1.
y' = y*0 - z*(-1) = z
z' = y*(-1) + z*0 = -y
So local +Y vector (0,1,0) -> (0,0,-1). So the top of texture points to -Z. If camera looks from +Z toward base, seeing the top of plane? Wait the plane normal +Y, seen from above with camera at +Y looking down. The screen horizontal? The texture top pointing -Z means in the camera view, if camera's up direction is -Z? Let's set camera position (0,5,6) looking at origin. When looking down/from front, the projection of world -Z (back side) onto screen is toward top or bottom? Need determine. Camera looks toward origin from +Z, so world -Z is farther away from camera, but the base's front is +Z. When looking at a horizontal plane from above, the "top" of the label pointing -Z would be away from the viewer (back), while "bottom" pointing +Z toward viewer. So text appears upside down? Actually in the camera view, +Z (front) is near bottom of screen? Let's set typical camera at (3,4,6) looking at (0,1,0). The screen up vector is roughly world +Y projected? But we're looking down from positive Z; the horizontal plane's -Z direction is farther (up in screen) maybe. Text top pointing -Z would be at the "top" of the label in screen, which is actually "up", so it might be normal. If top of label at back edge (far) corresponds to up on screen, then text is not upside down because screen up is away from viewer? Hmm, need test.

Maybe easiest: create canvas texture with text upside down in advance if needed? But hard to know. Could just use Sprite to avoid. Sprite always faces camera; but if the sprite is on base interior, when camera is slightly tilted, the text still faces camera. It won't look like it's on the surface, but for clicking and visual, okay. But "3D model" with sprites can be less realistic but acceptable. Use `THREE.Sprite` with SpriteMaterial. Scale e.g. 1.4, 1.4. Position at y=1.15. It will be visible. But when lid closed, sprite is hidden behind lid? Yes if lid mesh occludes? Sprites render as billboards; hidden if behind opaque lid? Should be. When lid opens, visible.

Need also add labels on lid top maybe sprites on lid group? If Sprite is child of lid group, it will move with lid, but Sprite material always faces camera; its position is correct, but when lid rotates, sprite is still billboard at that point. Good. For top labels, create at local (x, y=0.21, z=1.3). Since Sprite is billboard, if camera from side, it will face camera but anchored on lid surface; might intersect. Fine.

But raycast for sprites? The `intersectObjects(lidGroup.children, true)` includes sprites? Raycaster can intersect sprites? Maybe yes if material? But we don't need label sprites clickable? If included, click on a sprite on lid can toggle. But if sprites are children of lidGroup, raycast might hit them. Nonsphere? Need `raycast` method exists for Sprite. okay. But to avoid weird, include meshes only, or raycaster.intersectObjects(lidGroup.children, true) includes sprites; should be okay. However sprites on top of lid might make clicking easier. But if near lid, fine.

Need maybe use "BoxGeometry" for lid and base not Rounded? We'll import RoundedBoxGeometry and use. Need install? CDN.

Need add `renderer.outputColorSpace`? Three r160 standard. Need use `THREE.SRGBColorSpace`. But no need maybe. Use `renderer.toneMapping = THREE.ACESFilmicToneMapping; renderer.toneMappingExposure = 1.2;`. Fine.

Need add contact lens case "L/R compartments" with small circular floor, rims, and maybe a little lens (thin blue disc) inside each? Could create a small contact lens? Might be nice:
- Add a translucent disc inside each circle, representing lens solution/lens. Use CircleGeometry with material transparent opacity and color? But the circle should be slightly below or on base. Could add "liquid" cylinder? Maybe not.

Need add "rings" at top of base:
```
function createCompartment(x, z, label) {
 const circleGeom = new THREE.CircleGeometry(0.9, 48);
 const circleMat = new THREE.MeshStandardMaterial({ color: 0x223844, roughness: 0.15 });
 const circle = new THREE.Mesh(circleGeom, circleMat);
 circle.rotation.x = -Math.PI / 2;
 circle.position.set(x, 1.005, z);
 baseGroup.add(circle);

 const ringGeom = new THREE.TorusGeometry(0.9, 0.07, 16, 48);
 const ringMat = new THREE.MeshStandardMaterial({ color: 0xf0f8ff, roughness: 0.3 });
 const ring = new THREE.Mesh(ringGeom, ringMat);
 ring.rotation.x = Math.PI / 2; // Torus lies in XY plane normally? TorusGeometry around z axis? Let's check. THREE.TorusGeometry creates torus in XY plane centered at origin, normal +Z. To make it lie horizontal with normal +Y, rotate by -PI/2? Wait if torus's "hole" axis is +Z. We want axis vertical (+Y). So rotate x = Math.PI/2? For a ring on floor, need torus normal +Y, so rotate X by +PI/2? Let's compute. The torus is a tube around a circle in XY plane, normal +Z. To make circle in XZ plane, rotate about X by PI/2: +Z normal transforms? R_x(90): z' = y? Actually normal (0,0,1) -> y'= -sin90*1 = -1, z'=0. So normal -Y. That means ring lies horizontal but faces down? But torus has no direction, circle in XZ plane either. For TorusGeometry, orientation? It's symmetric, so rotating +PI/2 or -PI/2 works. Use `ring.rotation.x = Math.PI / 2;` then the torus circle is in XZ plane? Need yes due symmetry: circle originally in XY plane, after rotation x=90, Y axes maps to Z? Points (R,0,0) unchanged; (0,R,0) -> (0,0,-R?) formula y'=0, z'= -R. So circle is in XZ plane. Good. Use +PI/2. 
 ring.position.set(x, 1.0, z);
 baseGroup.add(ring);
```
But `TorusGeometry` centered at origin; ring at y=1.0. Good. Or use `RingGeometry` flat? The torus gives raised rim.

Need "lens solution" maybe a shallow cylinder:
```
const liquidGeom = new THREE.CylinderGeometry(0.75, 0.75, 0.02, 32);
const liquidMat = new THREE.MeshPhysicalMaterial({ color: 0x88ccff, transparent: true, opacity: 0.5, roughness: 0.1, metalness:0 });
const liquid = new THREE.Mesh(liquidGeom, liquidMat);
liquid.position.set(x, 1.015, z);
baseGroup.add(liquid);
```
But if it's liquid inside well, the well doesn't actually have depth. Could add a cylinder with bottom at y=1.0 and height 0.04, but it would protrude. Maybe skip. Need keep simple.

Need "L/R text" perhaps place in the interior circle, not on liquid. Use Sprite. Let's create a sprite:
```
function createLabelSprite(label, color) {
 const canvas = makeLabelCanvas(label, color);
 const texture = new THREE.CanvasTexture(canvas);
 texture.colorSpace = THREE.SRGBColorSpace;
 const material = new THREE.SpriteMaterial({ map: texture, transparent: true, depthTest: true, sizeAttenuation: true });
 const sprite = new THREE.Sprite(material);
 sprite.scale.set(1.4, 1.4, 1);
 return sprite;
}
```
`SpriteMaterial` default depthTest true. Need place y=1.08? If scale 1.4, extends ±0.7, could pierce lid when closed. Since base top at y=1 and lid bottom y=1.0, if sprite center at y=1.1 with radius 0.7, it extends above lid bottom, causing z-fighting/intersection? Actually when lid closed, sprite (billboard) is at y=1.1; lid box occupies y 1.0 to 1.4, so sprite center is inside lid! That would be hidden? Opaque lid covers it, but sprite might render above? Render order? It uses depthTest, center inside lid, pixels behind lid? But because it's a billboard, the plane passes through lid. Could be partially visible through lid? Probably not if depth test against lid. But better place at y=1.01 (on top of base) and scale smaller 1.0, extends 0.5 above base = y=1.51? Wait sprite in world coordinates scale size independent of geometry? Sprite with sizeAttenuation true: scale=(1,1,1) means half extents 0.5 in world units? Actually sprite size equals scale? I think Sprite scale is world units; scale.set(1,1,1) makes sprite 1 world unit across? Yes, half-width 0.5. If center y=1.01 and scale=1, top at y=1.51, inside lid. Closed lid covers. But if center y=1.01, lower half y=0.51 inside base; Sprite is billboard, depth test with base may hide lower half? It could be partially inside base, maybe render weird. Need choose small scale 0.7, center y=1.05. top 1.4, still at top of lid? It could clip. Hmm.

Better use flat plane with small height 0.8, placed at y=1.02; no issue because plane is horizontal, no vertical. Use plane to avoid inside geometry. But text orientation issue maybe acceptable. Or use sprite but set `material.depthWrite = false` and render behind? Not ideal.

Let's use textured plane oriented horizontal at y=1.02. Since plane is flat, no protrusion. Need orientation can be fixed. We can test mentally or adjust. Let's create plane and rotate x=-PI/2. If text appears mirrored, perhaps set `plane.rotation.x = Math.PI / 2`? Let's derive effect. The plane normal should be +Y to face camera from above. Both -PI/2 and +PI/2 have normal? R_x(+90) maps normal +Z? Formula with θ=90: y'= -1, z'=0 => normal -Y. Actually R_x(+90): normal (0,0,1) -> (0,-1,0) (down). If we view from above, the plane is transparent? MeshBasicMaterial DoubleSide or side? If using `side: THREE.DoubleSide`, it's visible from both sides, but text mapping on back side is mirrored? If normal -Y, from above, you'd see the back side, showing text mirrored. So use -PI/2 for normal +Y. The texture top maps to -Z as computed. If camera is from +Z, "-Z" is away from camera, so text appears upside down? Let's test with a simple perspective: Camera at (0, 5, 6) looking at (0,0,0). The view direction is roughly down and -Z. The horizontal plane is XZ. The texture coordinates on the XZ plane after rotation: original plane local X maps to world X. Original local Y maps to world -Z. So a texture drawn with "L" occupying upper half (local +Y) is at world -Z. The camera sees points with larger z (closer) at bottom of screen? Let's think camera's screen vertical corresponds to its local Y. Camera looks along -Z, with up +Y. Because camera is above and in front, the projection of the horizontal plane onto the camera's view: points with z larger (closer) are lower? Actually in perspective looking down from +Z, the far side (negative z) appears upper in screen, near side (positive z) appears lower. So local +Y (text top) maps to world -Z = far side -> upper screen. Thus text appears upright (top at top). Good! So rotation.x = -PI/2 is correct. For a camera from +Z/front. Great.

Need base top label positions. Wait base top y=1.0. Label plane at y=1.02 with width 0.8. If the label is horizontal, from camera at side it will be thin; but when looking at top, visible. Need "醒目" maybe okay. Could also add a vertical plane on front side "L/R"? But not needed.

Need add L/R on lid outer top. Use horizontal planes on lid top. But if lid rotates, labels rotate too. Use same orientation. Add to lidGroup at local y=0.21, z=0.6? Let's choose positions correspond to left/right compartments maybe at local x=-1.5, z=1.0; x=1.5,z=1.0. But if lid is closed, these labels are on top; when open, they're on the back/top. Good. Use small planes with white background? We can create label texture with background circle? On case top, labels should be colored green/blue/red. Use same label texture perhaps with a contrasting dark outline. For top, use a white translucent disc? But no need.

Need maybe "L" and "R" labels in base should be visible through lens solution? Let's place at y=1.02 (on top of base) inside ring. Since the ring at y=1.0, maybe label plane at y=1.005 with no space? If ring has tube radius 0.07, torus center at y=1.0, tube top at 1.07? The circle interior at y=1.0 maybe lower than rim. Label plane at y=1.02 is below rim top? It might be hidden by ring? Not hidden, plane is within circle, rim is around. Need okay.

Need perhaps create a shallow bowl/well by making a cylinder with dark material and top at y=1.0? Could create actual wells by using "CylinderGeometry" open-ended? Let's be a bit more 3D. Maybe create cutout effect with "negative" geometry is too much. Use "CSG" not.

But maybe we can make a more realistic base using two circular "cups" on top? Design: Instead of flat box with recesses, have a base platform with two protruding small cup housings (like jars) for lenses, and the lid open covers them. But lid singular? Let's think of common spectacle contact lens case: A small flat case with a hinged lid has two cavities inside, not protruding cups. So our flat box is appropriate.

Need add compartments maybe "well" by using a cylinder with open top? We can create a `CylinderGeometry` dark material with height 0.25, top at y=1.0, bottom at y=0.75; but need place inside base? We can place the top exactly at base top and lower part inside base, but base box is solid opaque; if we add an opaque cylinder inside the base, it won't cut a hole; it will just be hidden except top/bottom? Actually a cylinder inside the base below top surface is hidden because base material surrounds it. If its top face is at base top y=1.0, it appears as a circle on top (if we add top face). We can create a cylinder geometry with closed ends and place so its top face at y=1.0 and bottom at y=0.9 inside the base, but the base box is also there; render order/opacity could show? Since both opaque, the cylinder is inside the base but if it extends above base top (top face at y=1.0) the top face is coplanar with base top and may z-fight. But the sides of cylinder inside the base are hidden. This just creates a circle on the top surface? If cylinder material differs, the top face would show as a circle exactly at top. But coplanar z-fighting. We can set top at y=1.002, but then it protrudes slightly. It might look like a recessed area? Not a hole, but a colored circle. To make recess, create an outer ring plus inner dark circle. Good.

Need maybe use `new THREE.CylinderGeometry(0.9,0.9,0.02,32)` and place y=1.01? That is a disc on top, not recess. Better use ring.

Need "盒盖" click to open. Need maybe lid should have a "button" that says "点击" maybe. We can create a handle:
```
const knobGeo = new THREE.BoxGeometry(0.8, 0.15, 0.5);
const knob = new THREE.Mesh(knobGeo, knobMat);
knob.position.set(0, -0.1, 3.7); // front edge of lid
lidGroup.add(knob);
```
But if knob at front edge and lid closed, it's at world z=1.85? Wait local front z=3.8, so knob at z=3.8 maybe. With pivot z=-1.9 -> world z=1.9, half depth 3.8? Actually local z=3.8 world z=1.9. Base front z=1.75; knob slightly overhang. Good. But user can click knob too. We'll include in lidGroup. It will rotate with lid. Need maybe knob protrudes beyond front edge. Create a small rounded box/sphere.

Need ensure lidGroup raycast doesn't hit base? We use intersectObjects(lidGroup.children, true), including knob. Good.

Need add `cursor: pointer` when hovering over lid? Could implement raycast on pointermove and set cursor. Nice.

Need add `renderer.domElement.style.touchAction = 'none'`.

Need add "reset"/"open" button? Requirement only says click lid. We can add a small hint in UI: "点击盒盖打开/关闭". Good.

Need maybe show "L" and "R" on front of base too? Maybe not. But need "醒目". We'll add a front panel with two text planes on the base front face: one L left, one R right. If lid closed, labels on front visible. Hmm, that could satisfy even more. Let's consider.

Base front face at z = +1.75 (front), y=0.5. We can add vertical text planes on the front side:
- PlaneGeometry at z=1.755, using label "L" and "R" maybe rotated 0 with normal +z, positions x=-1.5, x=1.5, y=0.5. This makes L/R visible even when lid closed. But a contact lens case wouldn't have L/R on the outside front? Maybe it could. "醒目的 L 和 R 标识" maybe exactly on outside, so user can identify left/right before opening. Let's add them to outside front to ensure requirement. But if both inside and outside, fine. Need use label texture with dark background? Could create a circular badge? Actually simpler: put "L" and "R" on top of base near front edge? Hmm.

Let's create a "badge" on base front:
- PlaneGeometry (1.0 x 1.0) at z = baseDepth/2 + 0.01 = 1.76, oriented facing +Z (default). Position x=-1.6,y=0.5, x=1.6,y=0.5. The labels are at front. The base box front face is at z=1.75; badge at z=1.76 avoids z-fighting. Use `MeshBasicMaterial` with transparent texture. The texture may be just letter (no background), so visible as colored letter against base. Need create label texture with color. Use dark/white? If base teal, L/R red/blue visible. Good. Add "L" and "R" on front. This also helps.

Need lid labels on top maybe more. But maybe not needed. Could include all.

Need "contact lens case" maybe has two separate cap lids? The request says "配有盒盖" singular. Good.

Need maybe add a "base top" plane with "R" and "L" in compartments. Need label scale big.

Let's plan the scene in code concisely.

Need use importmap. Let's craft code carefully.

Potential issue: Three r160 `RoundedBoxGeometry` uses `this.parameters`, okay. Need import. Need `THREE.ColorManagement.legacyMode = false`? optional. `texture.colorSpace = THREE.SRGBColorSpace`.

Need maybe `MeshStandardMaterial` in r160? yes.

Need use `renderer.outputColorSpace = THREE.SRGBColorSpace;` in r160? In 0.160.0, renderer.outputColorSpace exists by default `SRGBColorSpace`; okay.

Need `renderer.shadowMap.type = THREE.PCFSoftShadowMap`.

Need animate:
```
const clock = new THREE.Clock();
function animate() {
 requestAnimationFrame(animate);
 const dt = clock.getDelta();
 // use independent lerp
 let current = lidGroup.rotation.x;
 let target = isOpen ? -MAX_OPEN_ANGLE : 0;
 lidGroup.rotation.x += (target - current) * Math.min(1, dt * 6);
 controls.update();
 renderer.render(scene, camera);
}
```
Using dt*6 maybe. Need `dt` small. Use `let lerpFactor = 1 - Math.pow(0.001, dt)`? Simpler:
```
lidGroup.rotation.x += (target - lidGroup.rotation.x) * Math.min(1, 0.15);
```
Could be 60fps. Fine.

Need handle `camera` positions. Use:
```
const camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(5, 4, 7);
camera.lookAt(0, 0.6, 0);
```
Base center y=0.5. Good.

Need scene background. Use gradient? Set `scene.background = new THREE.Color(0xe6f2f5);` or add fog. Add grid or ground:
```
const grid = new THREE.GridHelper(10, 20, 0x99bbcc, 0xccddee);
grid.position.y=0;
scene.add(grid);
```
Base bottom y=0. Good. Ground shadow:
```
const groundMat = new THREE.ShadowMaterial({ opacity: 0.3 });
const ground = new THREE.Mesh(new THREE.PlaneGeometry(10,10), groundMat);
ground.rotation.x=-Math.PI/2;
ground.receiveShadow=true;
scene.add(ground);
```
Could add.

Need lighting:
```
const hemi = new THREE.HemisphereLight(0xffffff, 0xbfd9e8, 0.8);
scene.add(hemi);
const dir = new THREE.DirectionalLight(0xffffff, 2.5);
dir.position.set(3,6,4);
dir.castShadow = true;
dir.shadow.mapSize.width=1024; ...
scene.add(dir);
const fill = new THREE.DirectionalLight(0xffffff, 0.7);
fill.position.set(-3,2,-3);
scene.add(fill);
```
Need toneMapping exposure maybe.

Need materials for labels. Let's make label texture function:
```
function makeLabelTexture(text, colors) {
 const canvas = document.createElement('canvas');
 canvas.width = 256;
 canvas.height = 256;
 const ctx = canvas.getContext('2d');
 ctx.clearRect(0,0,256,256);
 if (colors.background) {
   ctx.fillStyle = colors.background;
   ctx.beginPath();
   ctx.arc(128,128,120,0,Math.PI*2);
   ctx.fill();
 }
 ctx.fillStyle = colors.color;
 ctx.font = 'bold 190px Arial, sans-serif';
 ctx.textAlign = 'center';
 ctx.textBaseline = 'middle';
 ctx.lineWidth = 14;
 ctx.lineJoin = 'round';
 ctx.strokeStyle = colors.stroke || 'rgba(255,255,255,0.85)';
 ctx.strokeText(text, 128, 132);
 ctx.fillText(text, 128, 132);
 return new THREE.CanvasTexture(canvas);
}
```
Need use `ctx.font = 'bold 190px Arial'` maybe letter fits. For "R" width 256, okay.
Need set texture `needsUpdate = true`? CanvasTexture set. `texture.colorSpace = THREE.SRGBColorSpace`.

Need create plane label:
```
function createLabelOnFace(text, color, opts = {}) {
 const canvas = ...
 const texture = ...
 const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true, side: THREE.DoubleSide, depthWrite: false });
 const mesh = new THREE.Mesh(new THREE.PlaneGeometry(opts.width||0.9, opts.height||0.9), material);
 if (opts.rotation) mesh.rotation.set(...)
 mesh.position.set(...)
 return mesh;
}
```
Need for base top: rotation.x = -Math.PI/2.
For base front: rotation.z=0? The front face normal +z. PlaneGeometry default normal +z. So to place on front face, no rotation needed? Wait plane is in XY plane and normal +Z. The base's front face is at z positive and points +Z. So no rotation. Texture orientation upright when viewed from front. Good.
For lid top: same as base top. But if lidGroup local, plane normal should +y locally. Use rotation.x=-Math.PI/2. Good.

Need create label sprites maybe no. Use planes.

Need maybe add "click lid" hint via HTML overlay:
```
<div id="hint">点击盒盖打开/关闭 · 拖动旋转视角</div>
```
Need CSS style.

Need create "case" function:
```
function createCase() {
 const group = new THREE.Group();

 // base body
 const baseGeo = new RoundedBoxGeometry(5, 1, 3.5, 4, 0.12);
 const baseMesh = new THREE.Mesh(baseGeo, baseMat);
 baseMesh.position.y = 0.5;
 baseMesh.castShadow = true;
 baseMesh.receiveShadow = true;
 group.add(baseMesh);

 // interior circles/rims
 addCompartment(-1.5, 0, 'L');
 addCompartment(1.5, 0, 'R');
 // front labels
 addFrontLabel('L', -1.5, 0.5, 1.76);
 addFrontLabel('R', 1.5, 0.5, 1.76);

 // lid group
 const lidGroup = new THREE.Group();
 lidGroup.position.set(0, 1.2, -1.9);
 scene.add(lidGroup); // or add to group? Need global coordinates.
 ...
 return { group, lidGroup };
}
```
If lidGroup is child of base group, base group is at origin; fine. For raycasting need lidGroup in scene. `group.add(lidGroup)` or scene.add? Since base group at origin, add lidGroup to caseGroup. We'll have `caseGroup` and add to scene. `lidGroup` accessible.

Need hinge axis position? If base rounded and top at y=1.0, but RoundedBoxGeometry height=1 center y=0.5, top face maybe y=1.0. Good.

Need lid closed: if lidGroup position y=1.2, the bottom of lid at y=1.0. But when using RoundedBoxGeometry with radius 0.08, vertical top/bottom maybe half height minus? Actually RoundedBoxGeometry still has overall height = specified? Yes.

Potential issue: RoundedBoxGeometry from three examples has `radius` cannot be greater than half of min dimension. For lid height=0.4, radius must be <=0.2. Use 0.08. Good.

Need create lid geometry:
```
const lidGeo = new RoundedBoxGeometry(5.5, 0.4, 3.9, 4, 0.08);
const lidMesh = new THREE.Mesh(lidGeo, lidMat);
lidMesh.position.set(0, 0, 1.95); // center of lid relative to pivot. Wait if depth=3.9 and pivot at back edge -1.95 from center, local center z = 1.95. Because lid spans local z 0 to 3.9, center at 1.95. Let's set pivot z = -1.95. base depth=3.5 => back z=-1.75. Lid depth=3.9 so back edge z=-1.95, overhangs behind base by 0.2. Center of lid local z=1.95. At closed, pivot z=-1.95 + 1.95=0. Good. The lid is slightly wider and deeper. Its front edge z=+1.95, base front z=+1.75, overhang 0.2. Good.
```
Need choose pivot position z=-1.95. Set `lidPivot.position.set(0, 1.2, -1.95)`. Then lidMesh center local z=1.95. Good. But base back face at z=-1.75, pivot z=-1.95, hinge behind base. The lid bottom at y=1.0. Nice.

But when lid rotates, the pivot is behind the base, so the hinge won't intersect base. Fine. Could place hinge at z=-1.75? If lid depth=3.8, pivot at back edge exactly on back face. Let's use base depth=3.5, back z=-1.75. Set lid depth=3.7 and pivot z=-1.75, lid center local z=1.85? But lid would front edge z=1.95? Wait if depth=3.7, pivot z=-1.75, center local z=1.85, front world z=0.1? Actually local spans 0 to 3.7; world center = -1.75+1.85=0.1, front = -1.75+3.7=1.95. Good. Use lid width=5.3 depth=3.7. Pivot at z=-1.75. More realistic at back edge. The lid center local z=1.85; local geometry center (0,0,1.85). Good. Let's set base depth=3.5, back=-1.75; lid depth=3.7, pivot z=-1.75, lidMesh position local z=1.85, lid extends from local z=0 to 3.7. Good. Lid front overhang 0.2. Great.

Need if lid group position y=1.2, local center y=0. It is centered at height 1.2. The lid bottom y=1.0. Good. When closed, lid top y=1.4. Base top y=1.0. Nice.

Need if lid outer top labels at local y = 0.2+0.001 = 0.201, z=0.8? The lid top surface local y=0.2, yes. But rounded edge may be slightly less; use y=0.21. Place positions x=-1.5, z=0.9; x=1.5,z=0.9. If depth=3.7, z ranges 0 to 3.7, center of top surface 1.85. Left/right positions near front/back? Need if top labels should be near front (z=0.6?) Actually when closed, the front is z >? Wait world z = pivot z + local z = -1.75 + local z. The lid front at local z=3.7 -> world z=1.95. Back local z=0 -> world -1.75. So local z=1.0 is world -0.75, near back. If labels should be at center of left/right compartments, compartments at world z=0. Local z = world z - pivot z = 0 - (-1.75)=1.75. So place top labels at local z=1.75, x=±1.5. Great. If the lid top closed, labels directly above compartments. Add them to lidGroup at local z=1.75.

Need add inner bottom labels maybe at base top y=1.02. Need positions x=±1.5,z=0. Labels visible. Use plane geometry width=0.8 height=0.8. Good.

Need maybe the lid group when opened rotates; top labels move. Fine.

Need maybe add lid underside labels? If open, interior labels on base visible. Good.

Need create handle on lid front:
```
const knobGeo = new RoundedBoxGeometry(0.8, 0.2, 0.3, 2, 0.08);
const knob = new THREE.Mesh(knobGeo, lidMat);
knob.position.set(0, -0.1, 3.65); // front edge local z=3.7, protruding slightly; y below lid bottom? Actually lid center local y=0, bottom y=-0.2. If knob position y=-0.2? To be a tab at front lower side, place y=-0.15, z=3.7. It extends below/forward. But if closed, it would be at world z=1.95, y=1.05, just below lid bottom, above base top? It may intersect base if extends below y=1? Position y=-0.15 relative to group y=1.2 -> world y=1.05, bottom of knob at 0.95? Wait knob height 0.2, center y=1.05, bottom 0.95 which is below base top 1.0, so it would intersect base front edge. Better place knob at y=0? center lid, local y=0 (world 1.2). But a tab on lid front side at mid-height, protrudes forward. When closed, knob extends from y=1.1 to 1.3, fine. Position (0,0,3.75) slightly beyond front edge. That is a handle. Use radius? okay.
```
But raycast on knob should work. Need if knob at front edge, when closed it's in front of base near center; clickable. Good. Could add a text "OPEN" on knob? Not necessary. Maybe add a sphere knob? Let's do small rounded box:
```
const knob = new THREE.Mesh(new RoundedBoxGeometry(0.7, 0.35, 0.3, 4, 0.1), lidMat);
knob.position.set(0, 0, 3.75);
lidGroup.add(knob);
```
The lid front edge is at local z=3.7, so knob protrudes 0.05? RoundedBoxGeometry depth=0.3 extends ±0.15, so front face at z=3.9? Wait center z=3.75, half depth 0.15 => front 3.9, back 3.6. So protrudes 0.2 beyond lid front. Good.

Need maybe add a hinge cylinder along back edge to make realistic:
- CylinderGeometry? Actually hinge can be a thin cylinder along X axis at pivot position, length 5.5, radius 0.1, rotated? CylinderGeometry axis is Y, need rotate z=PI/2 to align X. Add meshes (knuckles) maybe. Not necessary.

Need add `scene.add(caseGroup);`

Need maybe `caseGroup.add(baseMesh)` but baseMesh position y=0.5. If group at origin, okay. But front labels positions are in group coords. lidGroup also child of group, group origin same. Good. Need if caseGroup position maybe (0,0,0). We'll use.

Need click lid to open: In `handleClick`, intersect with `lidGroup.children` true. Since lidGroup is child of caseGroup, world mat updated. `raycaster.intersectObjects(lidGroup.children, true)` works with object matrix? Yes, raycaster uses world matrices. But if lidGroup's children include nested group? RoundedBoxGeometry Mesh only. Need include knob. Good.
Potential issue: lid mesh has RoundedBoxGeometry; raycasting robust? yes.

Need if click on sprite/label on lid that is child of lidGroup? We won't add sprites as children, only planes. But raycast includes planes? `Mesh` with transparent material still raycast. If labels on lid top are planes, clicking on them counts. okay. But if label plane inside lid surface at y=0.21, it could be hit by ray before box? Raycast intersects with both, but some might intersect. Good.

Need `controls.target.set(0, 0.7, 0);` maybe.

Need `handlePointerMove` hover:
```
function onPointerMove(e) {
  if (dragging) return;
  const rect = renderer.domElement.getBoundingClientRect();
  const pointer = new THREE.Vector2(...);
  raycaster.setFromCamera(pointer, camera);
  const hits = raycaster.intersectObjects(lidGroup.children, true);
  renderer.domElement.style.cursor = hits.length ? 'pointer' : 'grab';
}
```
But OrbitControls also listens pointermove. We must attach pointermove after? We'll use same. Need set dragging state on pointerdown/move. If not dragging, update cursor. OrbitControls may set cursor to grabbing. Need maybe `controls.domElement.style.cursor`. We'll manage.

Need if `lidGroup.children` includes labels that are maybe on top and raycast intersects even when lid is open? That's fine. Need if lid closed, labels on lid top are hit. Good.

Need `pointerdown` store position. Need `dragging` false. But OrbitControls pointerdown will also rotate. On pointerup, if not dragged, handleClick. Need set down=false.

Potential issue: `pointermove` fires during OrbitControls drag, and we set `dragging=true`; but after release, cursor remains pointer? Need on pointerup maybe call hover update. Fine.

Need maybe use `document.addEventListener('pointerup', ...)` not just renderer to catch outside. We'll implement on renderer? If user drags outside, need. Use renderer.domElement.

Need "testing instructions": Provide run instructions in output: open HTML in browser with internet; use mouse drag rotate, scroll zoom, click lid. That's enough.

Need create final answer with code block. Since user asked "直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明." So final includes explanation and code.

Need ensure code is correct and no syntax errors. Let's draft code carefully.

Let's write complete HTML:

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>隐形眼镜盒 3D</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { overflow: hidden; font-family: Arial, sans-serif; }
  #info {
    position: fixed;
    top: 16px; left: 50%; transform: translateX(-50%);
    background: rgba(0,0,0,0.55);
    color: #fff;
    padding: 10px 18px;
    border-radius: 24px;
    font-size: 14px;
    pointer-events: none;
    z-index: 10;
    backdrop-filter: blur(6px);
  }
  #status {
    position: fixed; bottom: 16px; left: 50%; transform: translateX(-50%);
    background: rgba(0,0,0,0.35); color: #fff; padding: 4px 12px; border-radius: 12px;
    font-size: 12px; pointer-events: none; z-index: 10;
  }
</style>
</head>
<body>
<div id="info">👆 点击盒盖打开 / 关闭 · 🖱️ 拖动旋转 · 滚轮缩放</div>
<div id="status">状态：关闭</div>

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
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
...
```

Need `THREE.SRGBColorSpace` exists in r160. yes. `texture.colorSpace = THREE.SRGBColorSpace`.

Need create scene:
```
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xe8f4f8);

const camera = new THREE.PerspectiveCamera(40, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(6, 4.5, 7);
...
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;
document.body.appendChild(renderer.domElement);
```
Need maybe set renderer.outputColorSpace = THREE.SRGBColorSpace; default but set.
```
renderer.outputColorSpace = THREE.SRGBColorSpace;
```
In r160, `outputColorSpace` exists. Good.

Need controls:
```
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 0.8, 0);
controls.enableDamping = true;
controls.dampingFactor = 0.08;
controls.minDistance = 4;
controls.maxDistance = 20;
controls.update();
```

Need lights. Add ground:
```
scene.add(new THREE.HemisphereLight(0xffffff, 0xcfe0ea, 0.9));
const dir = new THREE.DirectionalLight(0xffffff, 2.6);
dir.position.set(4, 8, 5);
dir.castShadow = true;
dir.shadow.mapSize.set(1024,1024);
dir.shadow.camera.left=-6; ...
scene.add(dir);
const fill = new THREE.DirectionalLight(0xffffff, 0.8);
fill.position.set(-4, 2, -3);
scene.add(fill);
const ground = new THREE.Mesh(new THREE.PlaneGeometry(12,12), new THREE.ShadowMaterial({ opacity: 0.25 }));
ground.rotation.x = -Math.PI/2;
ground.position.y = -0.01;
ground.receiveShadow = true;
scene.add(ground);
const grid = new THREE.GridHelper(12, 20, 0x9cc4d8, 0xc0dce8);
grid.position.y = 0;
scene.add(grid);
```
Need possibly ground plane ShadowMaterial at y=-0.01, grid at y=0, base bottom y=0. Slight shadow. okay.

Need materials:
```
const baseMat = new THREE.MeshStandardMaterial({ color: 0x86c5c9, roughness: 0.45, metalness: 0.05 });
const lidMat = new THREE.MeshStandardMaterial({ color: 0x6fb7bd, roughness: 0.4, metalness: 0.05 });
const rimMat = new THREE.MeshStandardMaterial({ color: 0xe8f7f7, roughness: 0.3, metalness: 0.0 });
const interiorMat = new THREE.MeshStandardMaterial({ color: 0x1e3a3f, roughness: 0.25, metalness: 0.3 });
```
Need maybe lid slightly different.

Need `window.addEventListener('resize')`.

Need define label functions before create. Let's write:

```
function makeLabelTexture(text, options = {}) {
  const canvas = document.createElement('canvas');
  canvas.width = 256;
  canvas.height = 256;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, 256, 256);
  ctx.save();
  if (options.background) {
    ctx.fillStyle = options.background;
    ctx.beginPath();
    ctx.arc(128, 128, 108, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.font = `bold ${options.fontSize || 170}px Arial, sans-serif`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.lineJoin = 'round';
  ctx.lineWidth = options.strokeWidth || 12;
  ctx.strokeStyle = options.stroke || 'rgba(255,255,255,0.9)';
  ctx.strokeText(text, 128, 130);
  ctx.fillStyle = options.color || '#ffffff';
  ctx.fillText(text, 128, 130);
  ctx.restore();
  const tex = new THREE.CanvasTexture(canvas);
  tex.colorSpace = THREE.SRGBColorSpace;
  tex.anisotropy = 8;
  return tex;
}

function makeLabelPlane(text, options={}) {
  const texture = makeLabelTexture(text, options);
  const material = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    side: THREE.DoubleSide,
    depthWrite: false,
  });
  const mesh = new THREE.Mesh(new THREE.PlaneGeometry(options.width || 0.8, options.height || 0.8), material);
  return mesh;
}
```
Need if `depthWrite:false` with transparent, if multiple labels maybe okay. But if labels inside base, lid occludes? Since `depthWrite=false` but `depthTest` true default. If lid is rendered after, it will occlude? Actually depthWrite false means it won't write depth, but it will still test against previous depth. When lid renders after, lid writes depth and a pixel with lid is closer, so label not visible. Good. But if transparent labels render after opaque, depth test hides. Fine.

Need create compartments:
```
function addCompartment(parent, x, z, label) {
  // inner floor
  const floor = new THREE.Mesh(new THREE.CircleGeometry(0.95, 48), interiorMat);
  floor.rotation.x = -Math.PI / 2;
  floor.position.set(x, 1.005, z);
  parent.add(floor);
  // raised rim
  const rim = new THREE.Mesh(new THREE.TorusGeometry(0.95, 0.07, 16, 64), rimMat);
  rim.rotation.x = Math.PI / 2;
  rim.position.set(x, 1.0, z);
  parent.add(rim);
  // label inside
  const labelMesh = makeLabelPlane(label, { color: label === 'L' ? 0x59c7ff : 0xff6b6b, width: 0.85, height: 0.85 });
  labelMesh.rotation.x = -Math.PI / 2;
  labelMesh.position.set(x, 1.02, z);
  parent.add(labelMesh);
}
```
Need label inside with color? `makeLabelTexture` expects color as string or hex? `ctx.fillStyle` accepts hex number too. For `options.color: 0x59c7ff` works in canvas? Canvas fillStyle accepts CSS color string; numeric values? It may accept `#xxxxxx`, not numbers. Need convert to string. Use `labelColor(label)` returning string.
```
function labelColor(letter) {
  return letter === 'L' ? '#59c7ff' : '#ff6b6b';
}
```
For text stroke maybe dark? On interior dark background, colored letters with white stroke? If letter color light blue/red, white stroke okay. On base top floor dark, maybe visible. Good.

Need front labels:
```
function addFrontLabel(parent, label, x, y, z) {
  const mesh = makeLabelPlane(label, { color: labelColor(label), width: 0.9, height: 0.9 });
  mesh.position.set(x, y, z);
  parent.add(mesh);
}
```
Plane default normal +Z, no rotation. But `MeshBasicMaterial` side DoubleSide. Good. If on front face, the plane at z=1.76, base front at z=1.75. Good. But due rounded box, front face not exactly 1.75 at corners; at x=±1.5,y=0.5, face is at 1.75. Fine.

Need maybe make front labels bigger with white circle background? If letter on teal, visible. Use `background: 'rgba(255,255,255,0.85)'`? Could create circular badge. But `makeLabelTexture` with background option draws circle. Use background `#ffffff` maybe. But if background circle, labels more prominent. Let's set for front labels:
```
const mesh = makeLabelPlane(label, { color: labelColor(label), background: '#ffffff', width: 1.0, height: 1.0 });
```
For inside labels, no background? Maybe maybe background. But on dark interior, colored letter with white stroke visible. Need no background so appears on floor. For front labels, background white circular badge. `makeLabelTexture` background circle radius 108, plane width 1.0 => badge diameter 0.84, okay. It might obscure base. Good.

Need if background circle, the square plane's transparent corners; okay.

Need "makeLabelPlane" with `background` as string. Use options.
Need `makeLabelTexture` background options draws circle. But if foreground color is '#ff6b6b', fill. Great.

Need three.js color? no.

Need text rendering: `ctx.strokeText` then `fillText`. If `strokeStyle` white, it will outline. Good.

Need create case group and lid group.

Let's code case creation inside module:
```
const caseGroup = new THREE.Group();
scene.add(caseGroup);

const baseMat... 
// base body:
const base = new THREE.Mesh(new RoundedBoxGeometry(5, 1, 3.5, 5, 0.12), baseMat);
base.position.y = 0.5;
base.castShadow = true;
base.receiveShadow = true;
caseGroup.add(base);

addCompartment(caseGroup, -1.5, 0, 'L');
addCompartment(caseGroup, 1.5, 0, 'R');
addFrontLabel(caseGroup, 'L', -1.5, 0.5, 1.76);
addFrontLabel(caseGroup, 'R', 1.5, 0.5, 1.76);

// lid
const lidGroup = new THREE.Group();
lidGroup.position.set(0, 1.25, -1.75);
caseGroup.add(lidGroup);

const lidMesh = new THREE.Mesh(new RoundedBoxGeometry(5.4, 0.4, 3.7, 5, 0.08), lidMat);
lidMesh.position.set(0, 0, 1.85);
lidMesh.castShadow = true;
lidGroup.add(lidMesh);

// handle/knob
const knobGeo = new RoundedBoxGeometry(0.8, 0.3, 0.3, 3, 0.08);
const knob = new THREE.Mesh(knobGeo, lidMat);
knob.position.set(0, 0, 3.68);
knob.castShadow = true;
lidGroup.add(knob);

// top labels on lid
const lidLabelL = makeLabelPlane('L', { color: '#59c7ff', background: '#ffffff', width: 0.75, height: 0.75 });
lidLabelL.rotation.x = -Math.PI/2;
lidLabelL.position.set(-1.5, 0.205, 1.75);
lidGroup.add(lidLabelL);

const lidLabelR = makeLabelPlane('R', { color: '#ff6b6b', background: '#ffffff', width: 0.75, height: 0.75 });
lidLabelR.rotation.x = -Math.PI/2;
lidLabelR.position.set(1.5, 0.205, 1.75);
lidGroup.add(lidLabelR);
```
Wait lid top labels: The lid's top surface is at local y=0.2. The top label plane is at y=0.205, normal +y. But lidMesh is a rounded box with top surface. The label plane is above top, fine. When lid rotates, labels move. But if lid top labels at x=±1.5, z=1.75, there are also labels inside base at same x/z but at y=1.02. When lid closed, top labels at world y=1.455? Lid group position y=1.25 + local y=0.205 =1.455, top of lid. Good. Base labels at y=1.02, hidden inside lid box (lid bottom y=1.25-0.2=1.05). Wait lid group y=1.25, lidMesh center local y=0, height 0.4, bottom local y=-0.2 -> world y=1.05; top local y=0.2 -> world y=1.45. But base top y=1.0, there is a 0.05 gap between base top and lid bottom. The lid bottom at 1.05, base top at 1.0. Slightly gap. Could lower lidGroup to y=1.20, bottom=1.0, top=1.40. We earlier used y=1.2. Let's use `lidGroup.position.y = 1.2`. Good. Then top labels y=1.405. The base top labels y=1.02, lid bottom=1.0, okay maybe lid covers them but gap 0.02? Actually base labels at y=1.02; lid bottom at y=1.0 if lid group y=1.2, so label plane intersects lid? y=1.02 is above lid bottom, inside lid box. It becomes hidden. Fine. If lid bottom at 1.0 exactly, base label at 1.02 is inside lid (slightly). No problem. But if label depthWrite false, lid box rendered after? Need avoid z-fighting? It's just inside. Good.

Need if lid bottom at y=1.0 and base top at y=1.0, overlap? Lid is a solid box, base top is solid; faces touch. Good.

Need add hinge maybe. Let's add a thin cylinder along x axis at pivot? pivot at z=-1.75, y=1.2. But lidGroup itself is pivot; adding cylinder to scene not lidGroup? If cylinder at same line, when lid rotates it stays fixed. Add to caseGroup:
```
const hinge = new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.1,5.6,16), lidMat);
hinge.rotation.z = Math.PI/2;
hinge.position.set(0, 1.2, -1.75);
caseGroup.add(hinge);
```
Cylinder axis Y; rotate z=PI/2 -> axis X. It spans along X length 5.6, at back edge. Nice. But when lid rotates, hinge remains fixed. okay.

Need maybe add a small "hinge pin" at both sides? Not necessary.

Need label "L" and "R" might be too large with background circle. For front labels, plane width 1.0 and background circle diameter 0.84. Good. Base width 5, so left x=-1.5, right x=1.5; planes extend ±0.5, gap between them 1? left plane right edge -1.0, right plane left edge 1.0, gap 2.0. Good.

Need for compartments: left compartment center x=-1.5, radius 0.95, right center x=1.5, gap between rims 2*1.5 - 2*0.95 =1.1. Good.

Need base width 5, compartments fit.

Need maybe create lens inside compartments? Could add a subtle "lens" disc:
```
const lens = new THREE.Mesh(new THREE.CircleGeometry(0.6, 32), new THREE.MeshPhysicalMaterial({ color:0x88ccff, transparent:true, opacity:0.6, roughness:0.1, side:THREE.DoubleSide }));
lens.rotation.x = -Math.PI/2;
lens.position.set(x, 1.01, z);
parent.add(lens);
```
But label plane at y=1.02 above lens. It could look nice. But if lens is transparent and label plane above, okay. Need physical material in r160 supports transparent. Fine. Add inside compartment before label. However if label is visible, lens may obscure? Label at y=1.02, lens at y=1.01, from above label above lens, so visible. Good. Let's add to make model more realistic. Use `MeshPhysicalMaterial` with `transmission`? Not necessary. But if transparent lens over dark interior, might look like water. Add.
```
const lensMat = new THREE.MeshPhysicalMaterial({ color: 0x7fd4ff, transparent: true, opacity: 0.45, roughness: 0.05, metalness: 0, side: THREE.DoubleSide });
const lens = new THREE.Mesh(new THREE.CircleGeometry(0.55, 32), lensMat);
lens.rotation.x = -Math.PI/2;
lens.position.set(x, 1.02, z);
parent.add(lens);
```
Need label plane at y=1.03 so above. But if lens at y=1.02, label at y=1.03, okay. But if lid bottom at y=1.0? Wait lid bottom is y=1.0, lens top at 1.02, so when closed, lens inside lid, okay? Actually lid bottom at 1.0, lens at 1.02 is above 1.0, inside lid bottom (lid is solid box from 1.0 to 1.4), so it collides visually. We don't need physical collision. But if lid bottom at y=1.0, a lens at y=1.02 would be inside lid, hidden. But before opening, not visible. Fine. After lid opens, lens visible. But if lens is inside lid geometry, depth test will hide? Since lid starts at y=1.0, lens plane at y=1.02 is inside the lid (opaque), so it's hidden until lid opens. But when lid opens, lens is at y=1.02 and no lid there, so visible. Great. But if lid bottom at y=1.0 exactly, the lens plane is above, so inside lid; depth test with lid will hide. That's okay. But if z-fighting? no.
Need if base top labels at y=1.02 too, they are also inside lid. Label plane transparent, but hidden. Good.
Need maybe `depthWrite: false` for label plane could cause it to render through lid? Wait if transparent objects render after opaque, and label is inside lid but depthTest true, so hidden behind lid. Good.

Need maybe label plane at y=1.02 and lens at y=1.02 both, z-fighting? Slight. Set label at y=1.035, lens at y=1.015. Compartment floor at y=1.0. Need floor at y=1.0? The circle plane lies at base top y=1.0, with interiorMat. It is coplanar with base top? Actually base top face at y=1.0, circle plane at y=1.0, z-fighting. Better set floor at y=1.01. Then no z-fight? But the base top face is at y=1.0; circle at y=1.01 could be visible above base. But it's inside lid. Fine. Or use base top with separate interior? Let's add a shallow "recessed" look by placing floor slightly below rim? Hmm. Use floor at y=1.01; rim torus at y=1.0, tube radius 0.07, so floor at y=1.01 is near bottom of rim? Torus outer/inner: The ring's tube center at y=1.0, radius .07, so upper/lower extent ±0.07. The floor at y=1.01 is below rim top and slightly above torus center. The rim inner side maybe at radius 0.88, not full wall. But floor at y=1.01 as a disc inside ring. Good.

Need if base top at 1.0, floor at 1.01, no z-fighting. lips. Let's set:
- floor circle y=1.005
- lens y=1.015
- label y=1.025
- rim y=1.0, tube radius 0.08.
This puts lens/label slightly above base top but within rim height. If lid bottom at y=1.0, lens at 1.015 is inside lid. Fine.
Need if lid open, label visible. Good.

But if lid closed, lid bottom at y=1.0, and components above y=1.0 inside lid. That's okay. But if lid bottom exactly at base top, there is no space for components. Visually, closed lid may rest on base and if components are inside, no issue. It means the lens wells are not actually inside base, but above base; hidden by lid. Acceptable.

Could make lid bottom at y=0.98 (slightly lower?) No, then gap. Let's set base height=1.2? Not.

Need maybe add "case" to support open lid. Good.

Need "click to open" if user clicks lid; but if lid is closed, raycast on lid mesh. The lid's top label plane is at y=1.405, but lid mesh is below; raycast intersects both, okay. Need if clicking on top labels, because label plane is at y=1.405 and transparent, raycast intersects it too. The `intersectObjects` includes label planes. It counts as click. Fine.
Need but if labels have `depthWrite:false` and are transparent, intersectObjects still returns? Yes.

Need handle raycasting with `RoundedBoxGeometry` and `MeshBasic` transparent. okay.

Need maybe because labels are separate meshes and not clickable? We want clicking lid to open. If user clicks label on lid, counts. Good.

Need implement state:
```
let isOpen = false;
const MAX_OPEN = 1.15; // radians
function setStatusLabel() { document.getElementById('status').textContent = isOpen ? '状态：已打开' : '状态：已关闭'; }

function toggleLid() {
  isOpen = !isOpen;
  setStatusLabel();
}
```
Need animate lidGroup.rotation.x target:
```
const targetLidRotation = () => isOpen ? -MAX_OPEN : 0;
```
In animate:
```
const target = isOpen ? -MAX_OPEN : 0;
lidGroup.rotation.x += (target - lidGroup.rotation.x) * Math.min(1, 0.14);
```
Need if `MAX_OPEN` 1.15 radians ~66°, okay. Maybe a bit more 1.25 ~72°. Use 1.25.
Need if rotation.x decreases negative; open lid tilted back. Good.
Need if lidGroup has child lidMesh local center z=1.85. At rotation.x=-1.25, front edge local y? Let's compute local point at front edge (0,0,3.7) rotated: y = -3.7 sin(-1.25)=3.7*0.949=3.51, world y=1.2+3.51=4.71. That's high, nice. z = 3.7 cos(-1.25)=3.7*0.315=1.17, world z=-1.75+1.17=-0.58. Front edge above and behind center. Lid top labels maybe visible? Good.
Need if camera position (6,4.5,7) sees open lid. Good.

Need maybe update status when open animation fully? Set immediately. okay.

Need handle lid clicking if lid is open: The lid is tilted and may not be clicked easily? User can rotate camera. But okay.

Need if clicking on front labels or base doesn't toggle. Good.

Need "hover cursor" with lidGroup children. But if user clicks lid when open, raycast may hit lid labels or knob; okay.

Need if OrbitControls rotates on pointerup after drag; our `dragging` detects. Need set `down = false` on pointerup. Implementation:
```
let pointerDownPos = { x:0, y:0};
let pointerDownId = null;
let dragging = false;

renderer.domElement.addEventListener('pointerdown', (e) => {
  pointerDownPos.x = e.clientX;
  pointerDownPos.y = e.clientY;
  dragging = false;
});

renderer.domElement.addEventListener('pointermove', (e) => {
  if (pointerDownPos.x !== null && (Math.abs(e.clientX-pointerDownPos.x)>5 || Math.abs(e.clientY-pointerDownPos.y)>5)) {
    dragging = true;
  }
  updateHover(e);
});

renderer.domElement.addEventListener('pointerup', (e) => {
  if (!dragging) {
    handleClick(e);
  }
  pointerDownPos.x = null;
  pointerDownPos.y = null;
  dragging = false;
});
```
But OrbitControls uses pointerdown on element and may prevent default; our pointerup fires. Good.
Need if `pointerdownPos.x = null` initially; pointermove before down? okay.
Need if `pointerup` after dragging, we still update hover? no.

Need `updateHover` set cursor:
```
function updateHover(e) {
  const rect = renderer.domElement.getBoundingClientRect();
  const ndc = new THREE.Vector2(((e.clientX - rect.left)/rect.width)*2 - 1, -((e.clientY - rect.top)/rect.height)*2 + 1);
  raycaster.setFromCamera(ndc, camera);
  const hits = raycaster.intersectObjects(lidGroup.children, true);
  renderer.domElement.style.cursor = hits.length > 0 ? 'pointer' : 'default';
}
```
Need `controls.enableRotate` uses cursor? We can also set `renderer.domElement.style.cursor = 'grab'` by default? OrbitControls sets. But okay.

Need handleClick:
```
function handleClick(e) {
  const rect = renderer.domElement.getBoundingClientRect();
  const ndc = new THREE.Vector2(((e.clientX - rect.left)/rect.width)*2 - 1, -((e.clientY - rect.top)/rect.height)*2 + 1);
  raycaster.setFromCamera(ndc, camera);
  const hits = raycaster.intersectObjects(lidGroup.children, true);
  if (hits.length > 0) {
    toggleLid();
  }
}
```
Potential issue: When lid is closed, lidGroup children include label planes at top lid, and lid mesh. When clicking near the handle/knob, if knob at front edge, it might be in front of base and intersect. Good.
Need if lid is open and lidGroup children are somewhere behind. Clicking them toggles close. Good.

Need maybe `raycaster.intersectObjects(lidGroup.children, true)` includes `lidMesh` and `lidLabelL/R` and `knob`. Good.

Need if click on lid mesh but there are front labels on base below? No, because we test lidGroup only. Good.

Need `controls` rotation with damping not affect pointerdown? okay.

Need need `const raycaster = new THREE.Raycaster();` global.

Need after `renderer.domElement.style.touchAction = 'none';` to prevent scrolling.

Need final animate:
```
function animate() {
  requestAnimationFrame(animate);
  const target = isOpen ? -MAX_OPEN : 0;
  lidGroup.rotation.x += (target - lidGroup.rotation.x) * 0.12;
  if (Math.abs(target - lidGroup.rotation.x) < 0.0005) lidGroup.rotation.x = target;
  controls.update();
  renderer.render(scene, camera);
}
animate();
```
Need perhaps `MathUtils.lerp` but okay.

Need window resize:
```
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

Need maybe "only one file". The code will be in one file.

Need maybe use `RoundedBoxGeometry` is in importmap. Does unpkg serve `examples/jsm/geometries/RoundedBoxGeometry.js`? Let's verify path. In three.js repo, file is `examples/jsm/geometries/RoundedBoxGeometry.js`. Yes.
Need if importmap `three/addons/` maps to `https://unpkg.com/three@0.160.0/examples/jsm/`. Good.

Need issue: Cross-origin? unpkg supports. Need if run from local file, importmap works as long internet. good.

Need perhaps use `type="module"` after importmap. Good.

Need maybe in `<script type="importmap">` if browser doesn't support importmap? Modern browsers support. okay.

Need maybe if using r160, OrbitControls constructor expects `camera, domElement`. yes.

Need maybe `THREE.RoundedBoxGeometry` not in core; imported. Good.

Need maybe `RoundedBoxGeometry` uses `this.parameters` for raycast? yes.

Need maybe `renderer.outputColorSpace = THREE.SRGBColorSpace;` but in r160 `THREE.SRGBColorSpace` may be `NullColorSpace`? Let's recall Three.js r152 introduced color management; `THREE.SRGBColorSpace` exists. yes.

Need "RoundedBoxGeometry" in r160 accepts `(width, height, depth, segments, radius)`; if `segments=5`, okay.
Need maybe `new RoundedBoxGeometry(0.8, 0.3, 0.3, 3, 0.08)` radius 0.08 <= 0.15. okay.
Need if "radius" causes geometry slightly smaller? okay.

Need maybe material for label uses `MeshBasicMaterial` with transparent and `depthWrite:false`. For front labels, the label plane is at z=1.76 in front of base. There might be no z-fighting because offset 0.01. Good.

Need "status" at bottom. Good.

Need maybe "L" and "R" on front labels with white circle background. The background circle maybe white but the text color orange. Need make `makeLabelTexture` background circle color `#ffffff`; if no background, text. For front labels, clear.

Need maybe inside labels on compartments should have background white circle too? If background white circle and interior dark, it will look like a badge, not part of well. But "醒目" could be good. However contact lens cases have L/R on the bottom of wells; maybe use colored letter with white outline, not white background. Let's not use background for inside labels. But then letter on dark floor visible; yes.

Need if inside label plane is horizontal and the floor is dark; letter color blue/red with white stroke. okay.

Need more realistic "compartment" with a "well" wall. Could create a ring cylinder using `CylinderGeometry` open ended? Let's see. On base top, add a short cylinder open ended (height 0.12) just above base top? Actually a ring wall around the floor. Use `CylinderGeometry` with `openEnded = true`? It is vertical cylinder (axis Y) with radius 0.98, height 0.12. Its bottom at y=1.0? Place position at (x,1.06,z) so cylinder extends from y=1.0 to 1.12. But torus already is a raised rim. Maybe add both? Use torus only, simpler.

Need maybe if lid closed and top gap, the lid bottom at y=1.0 and base top at y=1.0 exactly? Actually base rounded box top face at y=1.0. The lid bottom at 1.0. This produces a nice seam. But with RoundedBoxGeometry, the bottom edge of lid may be rounded, so contact at edges only. okay.

Need maybe set `base.position.y = 0.5`, and bottom at y=0.0. Ground grid at y=0. Nice.

Need maybe add `scene.fog = new THREE.Fog(0xe8f4f8, 12, 24);` Optional. no.

Need maybe use `ACESFilmicToneMapping` with `MeshBasicMaterial`? Basic material unaffected? okay.

Need potential issue: `MeshBasicMaterial` with `transparent: true, depthWrite: false` and tone mapping? no.

Need maybe "status" text uses Chinese; okay.

Need maybe if file opened directly, importmap from unpkg can be blocked by CORS? no.

Need maybe if user has no internet, not work. Mention need network.

Let's think if there are any bugs with lid pivot rotation direction. We set `lidGroup.rotation.x = -MAX_OPEN` to open. Let's verify. We need the lid rotates upward (front edge rises). We derived using local point (0,0,d), R_x(-a): y' = -d sin? Wait formula with θ=-a: y' = y cosθ - z sinθ = 0 - d sin(-a) = d sin a. Yes positive. z' = y sinθ + z cosθ = d cos a. So front edge local z decreases? Actually z' = d cos a, less than d; so it moves backwards toward hinge. Good. So open. The top surface local +Y after rotation: y'=cos a, z'=-sin a? For point (0,1,0): y'=cos(-a)=cos a, z'=sin(-a)=-sin a. So top normal points up and toward -Z (back). So the lid's top surface tilts up at front. Great. Good.

Need if `lidGroup.rotation.x` is negative, the lid rotates about X axis such that front edge lifts. yes.

Need maybe if we add hinge cylinder at z=-1.75; pivot same. Good.

Need perhaps if lid is open and camera from front, the lid is above and behind, not occluding base. good.

Need maybe if user clicks lid to close while open: Since lid is tilted, the lidGroup children are not necessarily in front; but raycast from camera maybe if camera at default sees it. okay.

Need maybe add an "open angle" that doesn't hit ground? max 1.25, fine.

Need maybe if initial `lidGroup.rotation.x` is 0. Need if target open negative and animate `lidGroup.rotation.x += (target - lidGroup.rotation.x) * 0.12`; current 0, target -1.25, diff -1.25, add -0.15 per frame. good. Close current -1.25, target 0, diff 1.25, add 0.15. okay.
Need maybe if animation lag and user clicks mid-animation, target changes. It will smoothly reverse. Good.

Need maybe `Math.abs(target - lidGroup.rotation.x) < 0.0005` but if not exactly, okay.

Need maybe if `document.getElementById('status')` updates immediately. okay.

Need perhaps include a "click hint" if no click. okay.

Need maybe "L/R labels on lid top" maybe with white background. But when lid is closed, top labels are visible from above; good. But if lid top labels are at local y=0.205, x=±1.5, z=1.75; the lid mesh is rounded with top surface. The label planes are above; if viewed from side, they might be barely visible because horizontal. Good.

Need maybe if label plane on lid top has `side: THREE.DoubleSide`, from above normal +y. From below normal -y, if lid open, it may be visible from other side (with texture mirrored?) Because `DoubleSide`, when viewed from below, the back side of texture appears mirrored, so labels could be reversed on underside. But labels on lid top not important. okay.

Need maybe "L/R" labels on front of base: The plane normal +Z, if base front face is slightly rounded due rounded box, at x=±1.5,y=0.5 the surface z maybe exactly 1.75. Plane at z=1.76. Good. But if RoundedBoxGeometry with radius 0.12, the front face across x is flat until near corners; at x=±1.5 within half width 2.5, plane coordinate relative to center -1.5, not near corner (corner at ±2.38), so flat. Good.
Need maybe `label.position.set(-1.5, 0.5, 1.76)` but base center y=0.5, top at 1.0, bottom at 0.0. The labels centered at mid-height. Good.

Need maybe "R" and "L" on front of base? Their positions: L at x=-1.5, R at x=1.5. If looking at the front of the case, is L on left? Actually from viewer facing front, x=-1.5 is to viewer's left, yes. Good.

Need maybe inside compartments: If looking from above, left compartment is L, right is R. Good.

Need maybe "contact lens case" should have two separate cap lids? Hmm. The request "配有盒盖" singular means overall lid. Good.

Need maybe "用户应能点击盒盖将其打开" — our lid is whole box hinged at back. Good.

Need maybe "L/R 标识" on case: On lid top and front and inside. Good.

Need maybe implement "click" with raycasting. Good.

Need maybe use `THREE.MathUtils` in animate? not.

Need maybe if `RoundedBoxGeometry` is not available due import map? Let's verify path. In three r160, the file is `examples/jsm/geometries/RoundedBoxGeometry.js`. The module exports class. Import:
```
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';
```
Works. But some versions use `RoundedBoxGeometry.js` importing `BufferGeometry`, `BoxGeometry`, etc. Fine.

Need maybe if `MeshPhysicalMaterial` for lens has `transmission`? no.

Need maybe if `ShadowMaterial` with ground and `receiveShadow` no `castShadow`? okay.

Need maybe if we add `dir.shadow.camera` with left/right etc. Need set near/far:
```
dir.shadow.camera.near = 0.5;
dir.shadow.camera.far = 20;
dir.shadow.camera.left = -6;
...
```
Not necessary, but okay.

Need maybe if using `RoundedBoxGeometry` and `castShadow` true. okay.

Need perhaps set `controls.update()` after initial. yes.

Need maybe if "L/R text" using `CanvasTexture` and `texture.colorSpace = THREE.SRGBColorSpace` in r160. If using `CanvasTexture` and colorSpace, okay.

Need maybe if `fillStyle` receives `#` strings. We'll define colors as strings. Good.

Need maybe if `ctx.font = 'bold 170px Arial, sans-serif'` no fallback if Arial unavailable. fine.

Need maybe if `lineWidth` 12 and `strokeStyle` white, stroke extends; okay.

Need maybe if `ctx.textBaseline = 'middle'` and `fillText` at y=130 gives vertical centering. Good.

Need maybe if `makeLabelTexture` with background circle and alpha? We clear then draw background circle. The plane outside circle transparent. Good.

Need maybe if `depthWrite: false` for labels causes transparent sorting? okay.

Need maybe if `material.depthWrite = false` for front labels means the label won't write depth, but since it's on front of opaque base, if another object behind maybe? no.

Need perhaps `renderer.domElement.style.cursor = 'default'` default. okay.

Need maybe if drag is not detected because OrbitControls changes camera and pointermove might not fire before 5px? It will. Good.

Need perhaps if pointerdown on lid and small movement but no drag, handleClick toggles. If user clicks and rotates a tiny bit <5px, OrbitControls may also rotate slight; but click toggles. Accept.

Need maybe if pointerup after drag, `pointerDownPos` reset. Good.

Need perhaps if user clicks on lid while dragging? no.

Need maybe if "status" should update after open? immediate. okay.

Need maybe if "test" instructions: "用鼠标拖拽旋转、滚轮缩放；点击盒盖/把手尝试开合；状态栏会切换。" Enough.

Need maybe if user wants "只给我一个文件" — final answer with code in one block, no separate files. okay.

Let's ensure code not too long? It's okay.

Let's think about labels on lid top: We add `lidLabelL` and `lidLabelR` as children of lidGroup. But `lidLabelL.rotation.x = -Math.PI/2`. This plane at local y=0.205. When lid closed, label is above lid top. But because `lidGroup` children include these label planes, raycasting and hover will treat them as clickable. But `lidLabelL` has `side: DoubleSide` and transparent; raycast likely intersects. Good.

Need if lid top labels plane lies horizontal at y=0.205, but the lid mesh top surface is y=0.2. Ray from camera above hits label. Good.
Need if ray from camera at side doesn't hit a horizontal plane (edge-on) except maybe, so clicking lid from side uses box. Good.

Need perhaps add a "div" hint clickable? no.

Need maybe if "unpkg" importmap with `three/addons/` cannot resolve if using "three" module? It can. Good.

Let's test mentally the import map:
```
"imports": {
 "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
 "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
}
```
When import `three/addons/geometries/RoundedBoxGeometry.js`, it resolves to `https://unpkg.com/three@0.160.0/examples/jsm/geometries/RoundedBoxGeometry.js`. Good.
Need likely `RoundedBoxGeometry.js` imports `{ BoxGeometry } from 'three'`; import map maps. Good.

Need if browser caches? okay.

Need maybe if `renderer.outputColorSpace` in r160 default is SRGB, but setting to `THREE.SRGBColorSpace` works. In r160, `THREE.SRGBColorSpace` string. Good.

Need maybe if `texture.colorSpace` in r160 and `texture.needsUpdate`? CanvasTexture set with colorSpace before first use; okay.

Need maybe if `RoundedBoxGeometry` with `segments=5` and `radius=0.08` has 5 smooth segments, but maybe not enough? okay.

Need maybe if `knob` material same as lid but lidMat defined before? yes.
Need maybe if `knobGeo` `RoundedBoxGeometry(0.8, 0.3, 0.3, 3, 0.08)` radius 0.08 <=0.15. good.
Need maybe if knob position local z=3.68, but lidMesh local front z=3.7? Wait lidMesh local center z=1.85, depth=3.7, extends z=0 to 3.7. So front face at z=3.7. Knob center at z=3.68, depth=0.3, extends z=3.53 to 3.83; so half sticks out. Good. But lidMesh front face at z=3.7; knob intersects front face. Good.
Need if knob y=0, height 0.3, extends y=-0.15 to 0.15, so it is vertically centered on lid front edge. Good. This looks like a latch/tab.

Need maybe if `knob` and `lidMesh` overlap; raycast okay.

Need maybe if `lidGroup.position.y=1.2` and knob center y local 0 => world 1.2, front face. Good.

Need maybe if hinge cylinder at y=1.2, z=-1.75, x-axis. But lidGroup also rotates around x axis at y=1.2, z=-1.75. The hinge cylinder is inside? It is a permanent cylinder along back edge, but when lid rotates, the lid back edge remains near hinge? Wait lidGroup pivot at z=-1.75, but lidMesh local z spans 0 to 3.7. The back face of lidMesh at local z=0. When closed, back face at pivot z=-1.75. When open, back face is still at local z=0 but rotated? At pivot, point local (x, y, 0) lies on hinge axis, so it remains at z=-1.75. Good. The hinge cylinder at same line; okay.

Need maybe if the lid is open, the lidMesh back face local z=0 and local y ±0.2 rotates around hinge, so the back edge at local y? Actually points on hinge axis are unchanged including thickness? The back face of lid at local z=0, but local y varies; if local z=0, rotation does not move? R_x(-a) of (0,y,0): y'=y cos a, z'=-y sin a. So points at top/bottom of back face move slightly in z. Thus the lid's back edge (a rectangle in the y-z plane) pivots around x-axis at origin, so it rotates in y-z plane. The hinge cylinder at pivot axis stays fixed, but the lid's back face rotates around it, so the back face is not fixed relative to cylinder; okay.

Need click "lid to open" maybe user must click lid specifically. Good.

Need perhaps if the lid when opened at 1.25 rad, the front edge world y=4.71, z=-0.58. But the lid back face at pivot y=1.2, z=-1.75, all within range. Since pivot y=1.2, the lid rotates around a horizontal line smaller than base top; he top surface lifts. Good.

Need maybe add `scene.add(caseGroup);` after creating.

Need perhaps if `caseGroup` contains `lidGroup`, and raycasting with `lidGroup.children` uses world matrix. If caseGroup at origin, fine. If not, still okay. We'll not move caseGroup.

Need maybe if status update "状态：关闭" while open animation still in progress. okay.

Need maybe if click lid immediately after toggling open but before animation complete, target changes. The lid's rotation starts moving to target. Good.

Need perhaps if `isOpen` true when target open but lidGroup.rotation.x hasn't reached target; user clicks again closes. That's okay.

Need maybe if top labels on lid use same color as front. Good.

Need maybe if labels on front have white background and are large, could occlude base material but okay.

Need maybe if "RoundedBoxGeometry" import path perhaps not "examples/jsm/geometries/RoundedBoxGeometry.js" but "examples/jsm/geometries/Something"? Let's double-check: In three.js repo, file name is `RoundedBoxGeometry.js`, class name `RoundedBoxGeometry`. It was introduced as "RoundedBoxGeometry" in examples. yes.

Need maybe if "unpkg.com/three@0.160.0" build uses module. yes.

Need maybe if "three/addons" path should include "examples/jsm"></script> yes.

Need maybe if `controls.update()` before first render. okay.

Need perhaps if `controls.autoRotate = false` no.

Need perhaps if `renderer.shadowMap.type = THREE.PCFSoftShadowMap` works. yes.

Need perhaps if `dir.castShadow = true` but `dir.shadow.mapSize.set` in r160 returns ShadowMap? Actually `Vector2.set(width,height)`. yes.

Need maybe if `new THREE.ShadowMaterial({ opacity: 0.25 })` and `ground.receiveShadow = true`; okay.

Need maybe if use `MeshPhysicalMaterial` for lens; `MeshPhysicalMaterial` in r160 with no `transmission`. okay.

Need maybe if `transparent: true` on lens with `opacity: 0.5` and `depthWrite: true` default. If lens is above floor and below label, if depthWrite true it may occlude labels? Label at y=1.025, lens at 1.015; from above, lens in front of label? Wait they are almost same plane, but label above lens. The lens is transparent; it may render before label, and depthWrite? if depthWrite true, it writes depth at plane y=1.015, so label at y=1.025 might be behind? Actually label slightly above lens, but due coplanar, label could z-fight and possibly be occluded by lens depth buffer. To avoid, set lens `depthWrite: false` or remove lens. Let's set lens depthWrite false. But then transparent render order? Hmm. Simpler: omit lens to avoid. The compartments with labels enough. Let's omit lens or make lens very subtle with depthWrite false and maybe label above. But no need.

I might include a thin "water" circle inside each compartment with `depthWrite: false`, but if it's transparent it might obscure label. Better not include lens. Let's keep floor with L/R labels. Could add "lens" as a tiny ring? No.

Maybe add a contact lens as a small "disc" under label? Not necessary.

Need maybe add "solution" as a bluish "liquid" in compartments? But no.

Let's simplify: compartment floor dark circle, rim, label plane. Good.

Need maybe if label plane on base top at y=1.02 and floor at y=1.005; no lens. good.

Need maybe if interior floor at y=1.005 and label at y=1.02; both inside rim. fine.

Need perhaps if `addCompartment` label uses `makeLabelPlane(label, { color: labelColor(label), width: 0.85, height: 0.85 });` but no background; letters may be transparent and on dark floor. Good.

Need maybe if `makeLabelTexture` sets `ctx.save/restore` but no need. okay.

Need maybe if `labelColor` with # strings. Define:
```
function labelColor(label) {
 return label === 'L' ? '#5ac8fa' : '#ff5e66';
}
```
Need use "R" red, "L" blue.

Need maybe if `ctx.strokeStyle = options.stroke || 'rgba(255,255,255,0.9)'`. For front labels with white background, white stroke on white background is invisible; but colored fill visible. The stroke helps inside. It will also add white outline around letter, okay.
Need maybe if `options.strokeWidth` maybe 10. good.

Need maybe if using `RoundedBoxGeometry` with `radius` 0.08 and `segments` 3, the knob shape may not be smooth but okay.

Need maybe if scene lights: `HemisphereLight` with intensity 0.9; `DirectionalLight` intensity 2.6. In r160, physically correct lights? Wait `useLegacyLights` default changed in r155? In r160, lights use physically correct mode? The intensity scale changed maybe. DirectionalLight intensity 2.6 fine. HemisphereLight 0.9 okay. No issue.

Need maybe if `renderer.toneMapping = THREE.ACESFilmicToneMapping` might make colors a bit washed. okay.

Need maybe if `MeshBasicMaterial` labels not affected by lighting, good.

Need perhaps if `background` circle in label texture with white color, and then `strokeText` white outline; On white background, the fill color visible but stroke white may create white border. It's okay.

Need maybe if front labels should be "醒目" large. Plane width 1.0 height 1.0, letter within circle radius 108/256*0.5=0.21? Actually plane width 1.0 corresponds to texture width 256, circle radius in texture 108 => radius 0.42. Letter fills 170/256*0.5=0.33. That's good. Overall badge diameter .84. Good.

Need maybe if inside labels plane width 0.85, letter height 0.29? okay.

Need maybe if "R" in red with white stroke and dark floor, visible.

Need maybe if "L/R" on lid top include white circle badges; with lid color teal, nice. Good.

Need maybe if "status" text says "点击盒盖" but if lid is open, clicking lid close. okay.

Need maybe if no interaction after initial? Good.

Potential bug: `RoundedBoxGeometry` imported but when using `new RoundedBoxGeometry(5,1,3.5,5,0.12)` radius 0.12 <= half height 0.5 ok. For lid height 0.4 radius 0.08 <=0.2 ok. For knob height 0.3 radius 0.08 <=0.15 ok. Good.

Need maybe if `base.position.y = 0.5` but `RoundedBoxGeometry` has radius 0.12, so the bottom of rounded face at y=0.0? Yes the overall bounding box extends ±0.5. Good.

Need perhaps if `ground.position.y = -0.01` and grid at y=0. Base bottom at 0. okay.

Need maybe if `camera.lookAt` after controls target? We set controls.target and update. Should not call lookAt. okay.

Need maybe if `controls.enablePan = false`? not necessary. Could allow. no.

Need maybe if `renderer.domElement.addEventListener('pointermove', updateHover)` and also `controls` update; updating cursor every pointermove okay.
Need maybe if `raycaster` intersects with `lidGroup.children` but `lidGroup.children` includes `lidLabelL` and `lidLabelR` MeshBasic with transparent. If clicking on lid labels when lid closed, toggles. okay.

Need maybe if `lidGroup.children` includes `lidMesh` and `knob`; but `lidMesh` geometry large. Good.
Need maybe if `intersectObjects(lidGroup.children, true)` recursively tests but if children include groups? none. Good.

Need perhaps if `handleClick` and `updateHover` compute NDC with `rect`. If canvas has CSS not full? It is full. okay.
Need maybe if `raycaster` is global and `setFromCamera` before camera updated? In pointer events after controls update? The camera matrixWorld is updated by render. But before first render? Controls update in animate. If user clicks before first render, likely not. okay.

Need maybe if `lidGroup.rotation.x` animation uses `Math.min(1, 0.12)`? We used `0.12` no delta. Fine.

Need perhaps if user opens lid and then drags camera around, the lid remains open. good.

Need maybe if "user should be able to click lid to open" but in UI hint we say "点击盒盖打开/关闭". good.

Need maybe if "盒盖" has an Open status. okay.

Let's maybe include an "auto camera title" no.

Need final answer structure:
- Intro: "代码如下，单文件可直接运行。"
- Code block.
- "运行方式" bullet: 保存为 .html, 浏览器打开, 需要联网 (Three.js CDN). 测试说明: 鼠标拖拽旋转; 滚轮缩放; 点击盒盖/把手开合; 状态显示.

Need maybe "不要查看其他文件" — no mention. Just deliver.

Let's ensure code after importmap and module script no `</script>` inside strings? none. Good.

Let's now write final code carefully.

Let's draft full code in my head:

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>隐形眼镜盒 3D 交互</title>
<style>
* { margin:0; padding:0; }
body { overflow:hidden; background:#e8f4f8; font-family: Arial, sans-serif; }
#info { ... }
#status { ... }
</style>
</head>
<body>
<div id="info">👆 点击盒盖打开/关闭 · 拖动旋转 · 滚轮缩放</div>
<div id="status">状态：关闭</div>

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
import { RoundedBoxGeometry } from 'three/addons/geometries/RoundedBoxGeometry.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color('#e8f4f8');

const camera = new THREE.PerspectiveCamera(40, innerWidth/innerHeight, 0.1, 100);
camera.position.set(6, 4.5, 7);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;
document.body.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 0.8, 0);
controls.enableDamping = true;
controls.dampingFactor = 0.08;
controls.minDistance = 3.5;
controls.maxDistance = 18;
controls.update();

// Lights
const hemi = new THREE.HemisphereLight('#ffffff', '#cfe0ea', 0.95);
scene.add(hemi);
const dir = new THREE.DirectionalLight('#ffffff', 2.5);
dir.position.set(4, 8, 5);
dir.castShadow = true;
dir.shadow.mapSize.set(1024, 1024);
dir.shadow.camera.left = -6;
dir.shadow.camera.right = 6;
dir.shadow.camera.top = 6;
dir.shadow.camera.bottom = -6;
dir.shadow.camera.near = 0.5;
dir.shadow.camera.far = 20;
scene.add(dir);
const fill = new THREE.DirectionalLight('#ffffff', 0.8);
fill.position.set(-4, 2, -3);
scene.add(fill);

// Ground/shadow/grid
const ground = new THREE.Mesh(new THREE.PlaneGeometry(12, 12), new THREE.ShadowMaterial({ opacity: 0.25 }));
ground.rotation.x = -Math.PI / 2;
ground.position.y = -0.01;
ground.receiveShadow = true;
scene.add(ground);
const grid = new THREE.GridHelper(12, 20, '#9cc4d8', '#c0dce8');
grid.position.y = 0;
scene.add(grid);

// Materials
const baseMat = new THREE.MeshStandardMaterial({ color: '#86c5c9', roughness: 0.45, metalness: 0.05 });
const lidMat = new THREE.MeshStandardMaterial({ color: '#6fb7bd', roughness: 0.40, metalness: 0.05 });
const rimMat = new THREE.MeshStandardMaterial({ color: '#e8f7f7', roughness: 0.30 });
const interiorMat = new THREE.MeshStandardMaterial({ color: '#1e3a3f', roughness: 0.25, metalness: 0.2 });

// Label helpers
function labelColor(label) {
  return label === 'L' ? '#5ac8fa' : '#ff5e66';
}
function makeLabelTexture(text, options = {}) { ... }
function makeLabelPlane(text, options = {}) { ... }

// Case group
const caseGroup = new THREE.Group();
scene.add(caseGroup);

const base = new THREE.Mesh(new RoundedBoxGeometry(5, 1, 3.5, 5, 0.12), baseMat);
base.position.y = 0.5;
base.castShadow = true;
base.receiveShadow = true;
caseGroup.add(base);

function addCompartment(parent, x, z, label) { ... }
addCompartment(caseGroup, -1.5, 0, 'L');
addCompartment(caseGroup, 1.5, 0, 'R');

function addFrontLabel(parent, label, x, y, z) { ... }
addFrontLabel(caseGroup, 'L', -1.5, 0.5, 1.76);
addFrontLabel(caseGroup, 'R', 1.5, 0.5, 1.76);

// Hinge
const hinge = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 5.6, 16), lidMat);
hinge.rotation.z = Math.PI / 2;
hinge.position.set(0, 1.2, -1.75);
caseGroup.add(hinge);

// Lid group
const lidGroup = new THREE.Group();
lidGroup.position.set(0, 1.2, -1.75);
caseGroup.add(lidGroup);

const lidMesh = new THREE.Mesh(new RoundedBoxGeometry(5.4, 0.4, 3.7, 5, 0.08), lidMat);
lidMesh.position.set(0, 0, 1.85);
lidMesh.castShadow = true;
lidGroup.add(lidMesh);

const knob = new THREE.Mesh(new RoundedBoxGeometry(0.8, 0.3, 0.3, 3, 0.08), lidMat);
knob.position.set(0, 0, 3.68);
knob.castShadow = true;
lidGroup.add(knob);

const lidLabelL = makeLabelPlane('L', { color: labelColor('L'), background: '#ffffff', width: 0.75, height: 0.75 });
lidLabelL.rotation.x = -Math.PI / 2;
lidLabelL.position.set(-1.5, 0.205, 1.75);
lidGroup.add(lidLabelL);
const lidLabelR = makeLabelPlane('R', { color: labelColor('R'), background: '#ffffff', width: 0.75, height: 0.75 });
lidLabelR.rotation.x = -Math.PI / 2;
lidLabelR.position.set(1.5, 0.205, 1.75);
lidGroup.add(lidLabelR);

// Interaction
const raycaster = new THREE.Raycaster();
let isOpen = false;
const MAX_OPEN_ANGLE = 1.25;
let pointerDownPos = { x: 0, y: 0 };
let dragging = false;

const statusEl = document.getElementById('status');
function setStatus() { statusEl.textContent = isOpen ? '状态：已打开' : '状态：已关闭'; }

function toggleLid() { isOpen = !isOpen; setStatus(); }

function getNdc(e) {
 const rect = renderer.domElement.getBoundingClientRect();
 return new THREE.Vector2(((e.clientX - rect.left)/rect.width)*2 - 1, -((e.clientY - rect.top)/rect.height)*2 + 1);
}

function handleClick(e) {
 raycaster.setFromCamera(getNdc(e), camera);
 const hits = raycaster.intersectObjects(lidGroup.children, true);
 if (hits.length) toggleLid();
}

function updateHover(e) {
 raycaster.setFromCamera(getNdc(e), camera);
 const hits = raycaster.intersectObjects(lidGroup.children, true);
 renderer.domElement.style.cursor = hits.length ? 'pointer' : 'default';
}

renderer.domElement.addEventListener('pointerdown', (e) => {
 pointerDownPos.x = e.clientX;
 pointerDownPos.y = e.clientY;
 dragging = false;
});

renderer.domElement.addEventListener('pointermove', (e) => {
 if (pointerDownPos.x !== 0 || pointerDownPos.y !== 0) {
   if (Math.abs(e.clientX - pointerDownPos.x) > 5 || Math.abs(e.clientY - pointerDownPos.y) > 5) dragging = true;
 }
 updateHover(e);
});
```
Wait `pointerDownPos.x !== 0` issue if initial pointerDown at (0,0) possible? We set pointerDownPos = {x:0,y:0}; if pointerdown at (0,0) then condition false? But not likely. Better use `let pointerIsDown = false; let pointerDownPos = {x:0, y:0};`
Set `pointerIsDown = true` on down, false on up. Then in move:
```
if (pointerIsDown && ... ) dragging = true;
```
Let's implement better.

```
let pointerIsDown = false;
let pointerDownPos = {x:0,y:0};
let dragging = false;
...
pointerdown: pointerIsDown = true; ...
pointermove: if (pointerIsDown && ...) dragging = true; updateHover(e);
pointerup: if (!dragging) handleClick(e); pointerIsDown=false; dragging=false;
```
Need also `pointercancel`? optional.

Need if pointerup after drag, still updateHover? no.

Need `renderer.domElement.style.touchAction = 'none';`
Need `window.addEventListener('resize'...)`.

Need animate:
```
function animate() {
 requestAnimationFrame(animate);
 const target = isOpen ? -MAX_OPEN_ANGLE : 0;
 lidGroup.rotation.x += (target - lidGroup.rotation.x) * 0.12;
 if (Math.abs(target - lidGroup.rotation.x) < 0.0005) lidGroup.rotation.x = target;
 controls.update();
 renderer.render(scene, camera);
}
animate();
```
Need maybe if `renderer.domElement.style.cursor` default? okay.

Need define `addCompartment` before use:
```
function addCompartment(parent, x, z, label) {
 const floor = new THREE.Mesh(new THREE.CircleGeometry(0.95, 48), interiorMat);
 floor.rotation.x = -Math.PI / 2;
 floor.position.set(x, 1.005, z);
 parent.add(floor);

 const rim = new THREE.Mesh(new THREE.TorusGeometry(0.95, 0.08, 16, 64), rimMat);
 rim.rotation.x = Math.PI / 2;
 rim.position.set(x, 1.0, z);
 parent.add(rim);

 const labelMesh = makeLabelPlane(label, { color: labelColor(label), width: 1.0, height: 1.0 });
 labelMesh.rotation.x = -Math.PI / 2;
 labelMesh.position.set(x, 1.025, z);
 parent.add(labelMesh);
}
```
Need if floor at y=1.005, label at y=1.025. If lid bottom at y=1.0, all are above base top and inside lid. okay. But floor circle at y=1.005, torus rim tube center at y=1.0 with radius 0.08, top of rim at 1.08. The label at y=1.025 is below rim top but inside. Good. Actually label plane is horizontal at y=1.025; the rim is a torus around it; from above, the label is visible within rim. Good.

Need if label plane with width 1.0 inside inner radius ~0.95 - tube radius 0.08 = 0.87, so label plane maybe extends beyond inner space. The plane is square, corners may extend outside circle, but texture letter itself within circle? The plane transparent, but if width=1.0, corners outside rim. But since material transparent, only letter/badge pixels draw. If no background, only letter. It can be partially clipped by rim? Actually plane is at y=1.025, rim top at 1.08, so if letter extends beyond inner radius, the vertical rim wall? Wait rim is a torus tube around circumference; the inner edge of rim has a curved wall rising from y=1.0 to 1.08. The label plane at y=1.025 may intersect that wall. In render, the plane is above the floor and within the rim. If the letter extends beyond inner radius, it might be hidden by the rim wall due depth? Since rim is opaque and label plane is at y=1.025 through the wall region, parts outside inner radius would be inside the rim and hidden. But the letter is centered and likely within inner radius. The plane corners outside are transparent. Fine.

Need perhaps use `labelMesh.position.y = 1.02`. okay.

Need if `makeLabelPlane` options `width: 1.0, height: 1.0` with `background` not specified for inside labels. Transparent plane still there. Raycast will hit the plane even on transparent corners? Raycaster intersects the geometry plane, not texture alpha. That means lidGroup? For inside labels not lidGroup, no. For lid top labels, if used, raycasting may hit transparent plane corners even if outside lid shape? But lid top labels are centered within lid and corners on lid, okay. Front labels not in lidGroup. No issue.
Need if `updateHover` intersects lidGroup children including lid top label planes; if pointer over transparent corner but not lid? The labels are over lid, okay.

Need maybe if `lidGroup.children` includes `lidLabelL/R` (planes) that are children of lidGroup, and they are horizontal; Raycaster intersects them if ray passes through. Could perhaps when lid is closed, pointer over the gap between labels but still over lid mesh, hits lid mesh. okay.

Need maybe if lid top labels at local y=0.205, but lid mesh RoundedBoxGeometry top surface maybe at y=0.2. When raycasting, if ray is not exactly vertical, it may hit label plane before lid mesh. Fine.

Need maybe if `knob` at local z=3.68, but lidMesh local front z=3.7. The knob extends outward; when lid closed, knob is at z=-1.75+3.68=1.93, front of base (base front z=1.75). A click on knob hits. Good.

Need maybe if the lid group rotation target uses `lidGroup.rotation.x` negative. But the hinge cylinder remains fixed at z=-1.75. However if lidGroup position y=1.2, and pivot line is at the back edge of the base, okay. But the lidMesh local center z=1.85; when closed, world center z=0.1, not 0. Actually base center z=0, lid center z=0.1, slight front overhang. Front edge z=1.95. Good. If we want lid centered, use pivot z=-1.75, lid center local z=1.75? Wait if lid depth=3.7, pivot at back edge -1.75, to be flush with back, local center should be depth/2=1.85, so world center z=0.1. The lid overhangs in front by 0.2 and in back? The back is flush. That's realistic. Good.
Base front z=1.75, lid front z=1.95, overhang 0.2. Good.

Need maybe if lid top labels at local z=1.75 => world z=0 when closed, exactly above compartments. Good.

Need perhaps if hinge cylinder length 5.6, base width 5, extends beyond by 0.3 each side. Good.

Need maybe if `RoundedBoxGeometry` for lid at local center z=1.85, so when pivot z=-1.75, back face local z=0. But because rounded radius on back edge, geometry actually extends slightly behind z=0? RoundedBoxGeometry bounding box extends half depth=1.85 from center, so from 0 to 3.7. Yes back face exactly 0, but rounded corners may have small radius behind? In RoundedBoxGeometry, the overall bounding box is exactly width/depth/height, and the rounded corners are inside, so back face at z=0. Good.

Need maybe if hinge cylinder is at z=-1.75, and lid back face at z=-1.75 when closed. okay.

Need maybe if `ShadowMaterial` ground receives shadow; but base y=0.0, grid y=0.0, ground y=-0.01. Good.

Need maybe if `renderer.domElement` should be set `style.display = 'block'`? Append to body, canvas inline; CSS body overflow hidden, okay.

Need maybe if `touchAction = 'none'` after append:
```
renderer.domElement.style.touchAction = 'none';
```
Good.

Need maybe if status "状态" not update if `isOpen` set; okay.

Need maybe if user clicks lid and the lid closes, but the top labels perhaps become on top. okay.

Potential issue: `makeLabelTexture` uses `ctx.save/restore`; fine. But if `options.background` is set, the background circle is drawn before text. The text stroke uses white but if background is white, text's white stroke invisible. fine.

Need maybe if `ctx.strokeText` before `fillText`; with stroke style white, the stroke is drawn around text. good.

Need perhaps if `texture.anisotropy = 8` might not be available? It is. okay.

Need maybe if `renderer.toneMapping = THREE.ACESFilmicToneMapping` and `MeshBasicMaterial` labels (unlit) maybe tone mapped? Actually MeshBasicMaterial is tone mapped? In three.js, tone mapping applies to all materials except maybe? It may affect color. Labels still fine.

Need perhaps if `HemisphereLight` intensity 0.95 and directional 2.5 with ACES may be bright. okay.

Need maybe if `renderer.physicallyCorrectLights = true` not in r160? default false? Actually `useLegacyLights` default changed. In r160, physically correct lights? Hmm. Intensities of DirectionalLight in three.js before r165? In r155, `useLegacyLights` default changed from true to false? Let's recall. Three.js r155 changed lighting intensity units; DirectLight intensity in physical mode candela? But for directional, intensity remains. HemisphereLight with intensity 0.95 okay. Directional 2.5 maybe okay. If too bright, adjust. We'll trust.

Need maybe if `dir.shadow.mapSize.set(1024,1024)` works but if using `dir.shadow.mapSize = new THREE.Vector2(1024,1024)` maybe. set okay.

Need maybe if `renderer.outputColorSpace = THREE.SRGBColorSpace;` but default in r160 is already. Set.

Need maybe if `texture.colorSpace = THREE.SRGBColorSpace` could not exist in r128 but we use r160.

Need maybe if `RoundedBoxGeometry` from examples imports `three` and uses `THREE.BufferGeometry` etc. With import map okay.

Need final answer should not include "data URI" etc. fine.

Let's think if there is any issue with "clicking lid" while OrbitControls uses pointer capture? The pointer events on renderer.domElement should fire. However OrbitControls might call `setPointerCapture` on pointerdown. Our pointerup is also on same element, receives event. Good.

Need maybe if OrbitControls also prevents `pointermove`? no. `updateHover` runs.

Need maybe if `renderer.domElement.style.cursor` set to 'default' on hover no lid, but OrbitControls might set 'grabbing' while dragging. We set after pointermove during dragging? Our pointermove first checks if pointerIsDown and movement >5 -> dragging=true, then updateHover. During drag, after pointermove, updateHover sets cursor based on hits, likely 'default' even if dragging. That could override OrbitControls 'grabbing'. Not critical. To avoid, in pointermove if dragging, skip updateHover:
```
if (pointerIsDown && ... ) dragging = true;
if (!dragging) updateHover(e);
```
But if dragging false until movement >5, updateHover runs. On drag, once dragging true, no update. After pointerup, maybe cursor remains previous. Could update after pointerup. Not necessary.
Maybe implement:
```
if (pointerIsDown && moved) dragging = true;
if (!dragging) updateHover(e);
```
Need if pointerIsDown true and moving small amount but not dragging, update hover. okay.
On pointerup, if !dragging handleClick; updateHover? no.
If dragging, after release, OrbitControls sets cursor maybe. We can call `updateHover(e)` on pointerup after reset? But handleClick toggles or not. Fine. Not needed.

Need perhaps if `updateHover` raycasts every pointermove, can be heavy but okay for simple scene.

Need maybe if `controls.enableRotate` and `renderer.domElement.style.touchAction='none'` okay.

Need maybe if CSS `#info` pointer-events none. Good.

Need perhaps if `#status` pointer none. Good.

Need maybe if user's browser doesn't support `importmap` (older). Maybe not. But requirement likely modern.

Could avoid ES modules and importmap by using CDN UMD for broad compatibility. But importmap is okay. However if user double-click file in Chrome, importmap works. If user uses Safari 16.4+ supports. Good.

Need maybe if some CDN providers block? unpkg reliable. But maybe in China, unpkg sometimes slow. Could use `https://cdn.jsdelivr.net/npm/three@0.160.0/...` maybe. Which CDN is better? unpkg is standard. The prompt likely doesn't care. Maybe use jsdelivr for speed. Let's choose unpkg? Could use `https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js` and examples/jsm/`. jsdelivr sometimes more reliable. Which do we use? Need ensure path. Both. Let's use unpkg because importmap example from three docs uses unpkg. Actually three docs use `https://unpkg.com/three@0.160.0/examples/jsm/` etc. Good.

Need maybe if code uses `three/addons/` mapping to unpkg but "three" bare specifier mapping uses unpkg. Fine.

Let's consider whether to include a "Testing instructions" with "需要网络加载 Three.js CDN". Good.

Need maybe if "not view other files" irrelevant.

Let's now formulate final answer. The final answer should include code block with the complete HTML. Before code maybe a short sentence. Then after code, run/test section. The user specifically "请直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。" So final can have "运行方式" after code. It is okay. Need maybe "代码即为全部内容，无其他文件。"

Need ensure no markdown issues with code block. Use ```html. Good.

Let's write final code fully polished.

Double-check code complete:

```
function makeLabelTexture(text, options = {}) {
  const canvas = document.createElement('canvas');
  canvas.width = 256;
  canvas.height = 256;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, 256, 256);
  ctx.save();
  if (options.background) {
    ctx.fillStyle = options.background;
    ctx.beginPath();
    ctx.arc(128, 128, 104, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.font = `bold ${options.fontSize || 160}px Arial, sans-serif`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.lineJoin = 'round';
  ctx.lineWidth = options.strokeWidth || 12;
  ctx.strokeStyle = options.stroke || 'rgba(255,255,255,0.9)';
  ctx.strokeText(text, 128, 130);
  ctx.fillStyle = options.color || '#ffffff';
  ctx.fillText(text, 128, 130);
  ctx.restore();
  const tex = new THREE.CanvasTexture(canvas);
  tex.colorSpace = THREE.SRGBColorSpace;
  tex.anisotropy = 8;
  return tex;
}
```
If background circle radius 104, plane width 1.0 corresponds to circle diameter 208/256=0.8125. Good. If none background, letter height 160/256=0.625 of plane width. For label width 1.0, letter height 0.625. Nice.

Need if `ctx.font` uses 160px but letter "R" maybe not fill full height. okay.

Need `makeLabelPlane`:
```
function makeLabelPlane(text, options={}) {
  const tex = makeLabelTexture(text, options);
  const mat = new THREE.MeshBasicMaterial({ map: tex, transparent: true, side: THREE.DoubleSide, depthWrite: false });
  return new THREE.Mesh(new THREE.PlaneGeometry(options.width || 0.8, options.height || 0.8), mat);
}
```
Potential: `depthWrite: false` for all labels; transparent. Good.

Need `addFrontLabel`:
```
function addFrontLabel(parent, label, x, y, z) {
  const mesh = makeLabelPlane(label, { color: labelColor(label), background: '#ffffff', width: 1.0, height: 1.0 });
  mesh.position.set(x, y, z);
  parent.add(mesh);
}
```
Plane default orientation normal +Z. Good. But if the front labels have white circular background and no stroke? okay.

Need `addCompartment`:
```
function addCompartment(parent, x, z, label) {
  const floor = new THREE.Mesh(new THREE.CircleGeometry(0.95, 48), interiorMat);
  floor.rotation.x = -Math.PI / 2;
  floor.position.set(x, 1.005, z);
  parent.add(floor);

  const rim = new THREE.Mesh(new THREE.TorusGeometry(0.95, 0.08, 16, 64), rimMat);
  rim.rotation.x = Math.PI / 2;
  rim.position.set(x, 1.0, z);
  parent.add(rim);

  const labelMesh = makeLabelPlane(label, { color: labelColor(label), width: 1.0, height: 1.0 });
  labelMesh.rotation.x = -Math.PI / 2;
  labelMesh.position.set(x, 1.025, z);
  parent.add(labelMesh);
}
```
Need label PlaneGeometry is 1.0 x 1.0; because `rotation.x=-PI/2`, it lies horizontally. It will face up. good.

Need if `rim.rotation.x = Math.PI / 2` and torus is planar. Use `THREE.MathUtils` no. good.

Need "floor" circle at y=1.005. But rim tube center y=1.0. The floor is above center and near bottom of rim? Actually the torus is a ring; its center line is y=1.0. The tube cross-section in y-z? For torus lying in XZ plane, at inner/outer edges, tube extends y ±0.08. At the center (r=0.95), the tube's cross-section? Need imagine torus in XZ plane: the tube is centered at radius 0.95 from origin in XZ plane, and tube cross-section is in vertical plane containing the radius and Y. At r=0.95, y=0? Actually the tube center at (0.95,0,0). The tube extends radially in x from 0.87 to1.03 and vertically y -0.08 to0.08. The floor disc radius 0.95 at y=1.005 will intersect the tube? The floor extends from center to radius 0.95. The torus tube inner edge starts at radius 0.87; floor exists up to 0.95, so part of floor (between 0.87 and0.95) is inside the torus tube's radial extent, but at y=1.005 near top of tube. Since floor is opaque, it might show as a disc that overlaps the rim. We want floor inside the ring. If floor radius 0.9 and rim center radius 0.95, tube radius 0.08, inner radius 0.87. Floor radius 0.9 covers the gap between inner wall? It might intersect the rim wall at 0.87-0.9. To avoid intersection, set floor radius = 0.82, rim center radius=0.95, tube radius=0.08 -> inner edge 0.87. Floor radius 0.82 fits inside, no intersection. Label within. Let's adjust: floor radius 0.85? Inner edge 0.87, so floor 0.85 fits. Use `CircleGeometry(0.85, 48)`, rim radius 0.95. Label width 1.0 is fine. Need not intersect. Good. Floor dark circle radius 0.85. Label letter maybe 0.62, fits. Good.
But if floor radius 0.85 and label plane width 1.0, corners outside but transparent. okay.

Need maybe if rim tube radius 0.08, top at y=1.08. Label y=1.025 is below top. Good.

Need maybe if floor at y=1.005, and rim inner wall vertical? The torus tube is rounded; floor is slightly above center, no issue. Could set floor y=1.0? It would be coplanar with base top? Actually circle is at y=1.0, and base top face at y=1.0, z-fighting. Use y=1.005. Fine.

Need maybe if floor at y=1.005 is above rim center but below top; it appears as dark bottom inside the raised rim. good.

Need maybe if label at y=1.025 is inside rim but above floor; good.

Need perhaps if lid bottom y=1.0 and floor at 1.005, lid closed intersects floor. But lid is a solid with bottom at 1.0, so floor is inside lid. No visual issue. But if lid is closed, a ray from top will hit lid before floor, so floor hidden. Good.

Need maybe if lid bottom at y=1.0 and rim torus extends from y=0.92 to1.08; when lid closed, the upper part of rim y>1.0 intersects lid. Again hidden. Good.

Need maybe if case top has raised rims but lid closes over them; physically impossible, but visually okay. Could instead make base top at y=1.0 and lid bottom at y=1.08? Then gap. Hmm. We could make lid bottom at y=1.08 to accommodate rims, but then label top? Let's consider: We can set lidGroup.position.y = 1.28, then lid bottom when closed at 1.08, above rim top 1.08? Actually rim top at base top+0.08=1.08. Lid bottom at 1.08 exactly, good. But then there is a gap of 0.08 between base top and lid bottom, visible from side. That may actually accommodate the rims. But if the rims protrude, the lid should sit above them. Base top at y=1.0, rim top at y=1.08. Set lidGroup.y = 1.28? Wait lid height=0.4, bottom at group.y -0.2. For bottom at 1.08, group.y =1.28. Top at1.48. Good. Then lid bottom at 1.08, rim top at1.08; fine. But there is a visible gap between base top (1.0) and lid bottom (1.08) except where rims fill. It might look like lid is slightly open by 0.08. Not ideal. But maybe if rims are considered part of base top, the lid rests on rims. The base top around rims is at1.0, but lid bottom at1.08, so gap. Hmm.

Better make rims as flush rings on top with floor at y=1.0, no protrusion? But our torus protrudes. We can reduce protrusion by using a flat ring/ring geometry not torus? Use `RingGeometry` flat at y=1.002, no protrusion. Labels at y=1.005. Then lid can be at y=1.2 with bottom 1.0, no collision. The "compartment" won't have raised rim, but dark floor encircled by lighter ring; looks like printed recess. That avoids physical conflict. Maybe use `RingGeometry` instead of TorusGeometry for rim: `new THREE.RingGeometry(0.85,1.05,48)` rotated horizontal, placed at y=1.002. This is a flat ring on the top surface, with material rimMat. It doesn't protrude much (0.002). But it's not a raised rim. Could also add a very thin cylinder for raised edge? Hmm.

Maybe use a short "cylinder ring" with `CylinderGeometry(0.85, 0.85, 0.12, 48, 1, true)` placed at y=1.06? It forms a wall around the floor. But bottom at 1.0, top at1.12. It would protrude and collide with lid if lid bottom 1.0. Could adjust base height? no.

Simplest: Keep torus rim, set lidGroup.y = 1.3? Wait if lid bottom at 1.1, rim top 1.08, no collision. But there will be gap. However from typical viewing angle, gap is small (0.02? no, lid bottom at1.1, base top at1.0, gap 0.1). That could look like the case is not fully closed. Not good.

Maybe set rims just as decorative flat rings, no protrusion. Then lid bottom at 1.0. Let's use flat ring geometry instead of torus to avoid collision and maintain closed appearance. But we want 3D wells. We can create a recessed look by using `RingGeometry` and dark floor. The lid closes flush. Let's do that. But "3D model" maybe okay. Need maybe add a shallow "cup" without collision by placing a cylinder inside the base? Hmm.

Alternative: Use `CylinderGeometry` with short height (0.1) and place it such that its top is at base top y=1.0 and the cylinder extends downward inside the base, not upward. Since base is a solid box, it's hidden, but its top face (circle) at base top shows as a colored circle. But no recess. Add `RingGeometry` around it to create a "rim". It won't protrude. Good.
Let's implement compartments with flat dark disc and a slightly larger ring:
```
const floor = new THREE.Mesh(new THREE.CircleGeometry(0.9, 48), interiorMat);
floor.rotation.x = -Math.PI / 2;
floor.position.set(x, 1.005, z);
const ring = new THREE.Mesh(new THREE.RingGeometry(0.85, 1.05, 48), rimMat);
ring.rotation.x = -Math.PI / 2;
ring.position.set(x, 1.01, z);
```
RingGeometry lies in XY plane by default; rotate -PI/2 to lie XZ. This creates flat ring. But `RingGeometry` inner radius 0.85 outer 1.05, width 0.2. At y=1.01, it slightly above base. No collision with lid if lid bottom at1.0? It is inside lid but tiny; okay.
Need if `RingGeometry` normal +y after rotation -PI/2? Default normal +z; R_x(-90) maps to +y. yes.
Then `floor` radius 0.85, label inside. This is simpler. But maybe less 3D. We can combine torus with no collision? Hmm.

Maybe we can use torus but place lid bottom at 1.0 and torus rim top at 1.0? Could use `TorusGeometry` with position y=0.96, tube radius 0.04, so top at1.0. But floor y=0.96? The torus center at y=0.96, top at1.0, bottom0.92. Floor at y=1.0? Need not. If torus center at y=0.96, it is nested into base top? Actually torus tube extends from0.92 to1.0, so top flush with base top. It creates a bead around top. Floor at y=1.0? Coplanar with base top; set floor y=1.002. The floor is inside ring; the rim's inner wall is from0.92 to1.0. But because it's below top, from above it appears as a rim. This is better. Let's set torus position y=0.96, radius=0.95, tube=0.04? Top at1.0? Actually tube radius 0.04, center at0.96 => top at1.0, bottom0.92. The inner edge of torus at radius=0.91 (0.95-0.04), floor radius=0.90. Perfect. Level with base top. Then lid bottom at1.0 contacts torus top maybe no protrusion. Nice. But torus tube radius 0.04 might look small. Use tube radius 0.05, center at0.95, top1.0. Inner radius 0.90; floor radius 0.88. Good. That yields a subtle raised rim flush with top. Let's set:
- `floor` circle radius 0.85, y=1.002
- `rim` TorusGeometry radius=0.92, tube=0.055, position y=0.945? Wait top y = center + tube = 1.0, so center = 0.945. Inner radius = 0.865. Floor radius 0.85. Good. But torus center below top, tube radius .055, top .??? 0.945+.055=1.0. Good. Since torus is centered at y=0.945, it is partially below base top surface (base top y=1.0). The torus intersects the base box, but both opaque; the lower half hidden by base? The base top face is at y=1.0, so any part below is inside the base and hidden. The upper half (0.9725 to1.0) is above base top? Wait center .945, tube .055, upper part from .945 to1.0; actually all up to1.0 is below or at base top? Base top is 1.0, so torus is entirely below/at base top? Center .945, top1.0. It is inside the base, not visible. Hmm if base top face at 1.0, the torus below it is hidden, but the top surface of torus at 1.0 would be exactly flush; if it doesn't protrude, only its top circle appears? Actually torus is a tube; if its upper surface is flush with base top, then the visible part is a ring of top surface with profile? The side of tube below top is hidden; the top tangent line is at y=1.0, so from above, the torus top at exactly y=1.0 is a line? Not enough. If center lower and tube radius bigger, top protrudes. Need protrusion.

Thus flat rings are easier.

Let's use flat ring plus torus? Maybe not.

Maybe use "compartment" as a flat dark circle with L/R label inside; the lid opens. That's acceptable. We can keep torus with center y=1.0, tube radius 0.08; it protrudes 0.08 above base. The lid bottom at1.0 will intersect. But we can set lid bottom y=1.0 exactly and torus top 1.08; impossible. Let's either remove torus or modify lid.

What if we make the base height 1.0 and place the two compartments as actual depressions below the top, by creating "wells" with lower floors? We can use top circle at y=1.0, floor lower than top, and rim around. But without CSG. For visual, we can create a `CylinderGeometry` with open top and place it so that the top opening is at y=1.0, the cylinder extends downward inside the base. The base box is opaque but the cylinder is also opaque; if we set cylinder material dark and its visible top edge at y=1.0, from above you see a dark circle. The sides of the cylinder below y=1.0 are hidden by the base because the cylinder is inside the base and the base top face covers it? Actually the base box is opaque; the cylinder is inside, but if your ray from above hits the base's top face before the cylinder's interior, you won't see the cylinder. Unless the cylinder's top cap is visible, but it's coplanar. Need not.

Could use a `RingGeometry` with raised edge.

But maybe no need for actual wells. The label inside a circle outline is okay. Let's use flat rings instead of torus to keep lid flush. The ring is 2D, but with dark floor and white ring, it visually suggests wells. Add the torus? Maybe no.

Need "3D model" with enough detail. It's okay.

Let's adjust compartment:
```
const floor = new THREE.Mesh(new THREE.CircleGeometry(0.85, 48), interiorMat);
floor.rotation.x = -Math.PI / 2;
floor.position.set(x, 1.0, z);
parent.add(floor);

const ring = new THREE.Mesh(new THREE.RingGeometry(0.85, 1.05, 48), rimMat);
ring.rotation.x = -Math.PI / 2;
ring.position.set(x, 1.005, z);
parent.add(ring);

const label = makeLabelPlane(... width:0.8);
label.rotation.x=-Math.PI/2;
label.position.set(x, 1.015, z);
parent.add(label);
```
This avoids protrusion; lid closed at 1.0. The ring is slightly above by .005, inside lid. fine.
Need if ring at y=1.005 and floor at y=1.0, no z-fighting. Good. The ring appears as a ridge around the floor but flat. Good.
Maybe use `RingGeometry` with `phiStart` etc. no.

Need if lid bottom at1.0 and ring at1.005 (inside lid). Hidden when closed. good.

Need maybe if ring at y=1.005 and label at1.015, both inside lid. fine.

Let's use this. Maybe add a torus rim with radius .92, tube .02 at y=1.02? That would protrude 0.04 above base, could be acceptable if lid bottom at1.0? It would intersect. no.

No torus. Good.

Need maybe if "L/R" labels on lid top with white background may look like inlays. okay.

Need perhaps if base front labels with white circle and lid top labels with white circle. Good.

Need maybe if status text says 状态：关闭. okay.

Let's think about if `RingGeometry` is oriented with `rotation.x = -Math.PI/2`. Yes normal +Y. To make ring in XZ plane. Good. The ring's inner radius 0.85, outer 1.05. It is at y=1.005. Floor radius 0.85 exactly same as inner, no gap. But if ring inner radius 0.85 and floor radius 0.85, they meet at edge; z-fighting? Maybe floor at y=1.0, ring y=1.005, so okay. To have a visible dark circle, floor radius can be 0.84, ring inner 0.85, gap? If there's a small gap of base color, not bad. Let's set floor radius 0.85, ring inner 0.85. The ring starts at 1.005 and floor at 1.0; no z-fighting. Could be okay.

Need if label plane width 0.9, text within inner radius? label letter width 0.9* (160/256?) letter actual around .56? okay.

Need maybe use "RoundedBoxGeometry" for base and lid. Good.

Need perhaps if `RoundedBoxGeometry` with radius .12 means top face at y=1.0 except near corners. The floor/ring at y=1.0 are at x coordinates ±1.5, z=0, which is not near corners; top face flat. good.

Need maybe if lid is a rounded box with radius .08, bottom face at y=1.0 except near corners. okay.

Need maybe if hinge cylinder at y=1.2 and z=-1.75; but base back face rounded, the cylinder might intersect? fine.

Need perhaps if `lidGroup.position.y = 1.2`, lid bottom =1.0, top=1.4. But the base top =1.0. Great.
Need if top labels on lid at y=0.205 -> world y=1.405, just above top. Good.
Need if we add hinge cylinder length 5.6, but lid width 5.4; hinge extends slightly. Fine.

Need maybe if `knob` at local z=3.68, when closed, knob world z=1.93; base front z=1.75. The knob protrudes. Good.

Need maybe if `lidGroup.children` include `lidMesh` and `knob` and `lidLabelL/R`. `raycaster.intersectObjects` on children with `recursive` true. Good.

Need maybe if if we update `lidGroup.rotation.x` but also the `lidGroup.position` y=1.2, the hinge cylinder fixed. okay.

Need perhaps if after opening lid, the labels on base are visible, but lid label planes may be in the way? The lid is above/behind. No.

Need maybe if in `updateHover`, raycaster intersects lidLabel planes even when lid is open and labels on top face; but if the label plane is facing away? `raycast` intersects DoubleSide planes? Raycaster intersects front side only? Three.js `Mesh.raycast` checks material.side: if FrontSide, backface ignored; if DoubleSide, both. Our labels use DoubleSide, so yes. This means clicking on lid labels always works. okay.

Need perhaps if `MeshBasicMaterial` side DoubleSide for labels on base front means from inside base? no.

Need maybe if "front labels" plane at z=1.76, but the white background circle appears on both sides; from behind (inside base) not visible due base. okay.

Need maybe if "shadow" of base on ground: base castShadow, ground receiveShadow. Good.

Need maybe if `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` no.

Need maybe if `renderer.domElement.style.touchAction = 'none';` after append:
```
renderer.domElement.style.touchAction = 'none';
```
Good.

Need maybe if `window.innerWidth` variables not defined? In module, `innerWidth` global. okay.

Need maybe if imports at top `import ...` in module script can access `innerWidth`. yes.

Need maybe if no `export` etc. okay.

Need maybe if "使用 Three.js" if CDN unavailable. okay.

Let's now produce final answer. Ensure code block correct. At final, after code, "运行方式" and "测试说明" maybe.

Let's write final response with code. Need not be too verbose maybe. But include necessary.

Let's ensure final code includes all whitespace. Good.

One more thought: If using `RoundedBoxGeometry`, the class is in examples/jsm/geometries/RoundedBoxGeometry.js. But there might be an issue with `RoundedBoxGeometry` and `raycast` after rotations? It extends BoxGeometry with indexed BufferGeometry; raycast uses geometry bounding sphere? Should be okay.

One potential issue: `Mesh` with `RoundedBoxGeometry` uses groups/material indices? It is a single BufferGeometry with materials? In examples, RoundedBoxGeometry creates a BoxGeometry with added vertices; it has groups for 6 materials? Actually if no material array, uses material. Raycast works.

Need maybe if `THREE.RingGeometry` and `CircleGeometry` both centered at x,z. good.

Let's final.

Wait, prompt "不要查看其他文件，直接创建" maybe from a benchmark that expects code only. The final can include code. Good.

Need perhaps if "使用 HTML、CSS 和 JavaScript 构建" — we do.

Let's now produce final answer.
