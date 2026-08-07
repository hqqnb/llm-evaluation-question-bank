User wants a 3D contact lens case model using Three.js. Let me create a single HTML file with:

1. A 3D contact lens case (cylinder-shaped container)
2. Clear L and R labels on the case
3. A lid that can be opened by clicking
4. HTML/CSS/JavaScript all in one file

Let me design this:

- The case body: a cylindrical container with two compartments (left and right)
- The lid: a cylindrical cap that covers the case
- L and R labels on the case body
- Click interaction to open/close the lid

Let me think about the structure:

1. **Case body**: A cylinder shape, maybe with a divider in the middle
2. **Two compartments**: Left and right with L and R labels
3. **Lid**: A larger cylinder that sits on top, can be rotated open

Let me use Three.js from a CDN.

Design plan:
- Case base: A rounded cylinder (like a contact lens case)
- Two circular indentations on top for the lenses
- L and R labels on top surfaces of these indentations
- A lid that covers everything, hinged on one side

Actually, let me simplify a bit but still make it look good:

- Case body: Cylinder with two circular compartments visible on top
- L and R labels on the two compartments
- A lid (dome-shaped or flat cylinder) that rotates open on a hinge

For the click interaction, I'll use raycasting to detect clicks on the lid.

Let me code this up:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>3D Contact Lens Case</title>
    <style>
        body { margin: 0; overflow: hidden; background: #1a1a2e; font-family: Arial, sans-serif; }
        canvas { display: block; }
        #info {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: #fff;
            background: rgba(0,0,0,0.6);
            padding: 10px 20px;
            border-radius: 20px;
            font-size: 14px;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div id="info">点击盖子打开/关闭</div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script>
        // ... Three.js code
    </script>
</body>
</html>
```

Let me think about the geometry more carefully.

The contact lens case:
- A cylindrical base, perhaps with a slightly larger diameter at the bottom
- Two circular compartments on top, side by side
- The lid is a flat cylinder that covers the whole thing

Actually, let me make it simpler but visually appealing:

**Case body**:
- Main body: CylinderGeometry with a nice color
- Top surface: Two circular recesses for the lenses
- Labels: L and R as TextGeometry or sprites

**Lid**:
- A cylinder that covers the top
- Hinged at one side

For opening interaction:
- Use raycasting on the lid mesh
- When clicked, animate the lid rotating open around the hinge (Y axis on one edge)

Actually, rotating a lid around a hinge is a bit tricky. Let me use a simpler approach:
- The lid rotates around an axis at the edge of the case
- Use a pivot group approach

Let me think about the hinge approach:
1. Create a pivot group at the hinge position (at the back edge of the case)
2. Add the lid mesh to this pivot group
3. Rotate the pivot group to open the lid

Actually, let me use a slightly different approach - the lid rotates around the Z axis (or X axis) from one side.

Let me set up the coordinates:
- Case centered at origin
- Case body: cylinder with radius 2, height 1
- Two compartments: at positions (-0.8, 0.5, 0) and (0.8, 0.5, 0) with small radius 0.7
- Lid: cylinder with radius 2.1, height 0.4, positioned at (0, 0.8, 0)

For the hinge, let me put it at the back edge: (0, 0.8, 2)
The lid rotates around the X axis at the hinge point.

So:
1. Create a pivot Object3D at position (0, 0.8, 2)
2. Add the lid as a child of the pivot, positioned relative to the pivot

Wait, let me think again. If the hinge is at (0, 0.8, 2), and the lid is centered at (0, 0.8, 0), then in the pivot's local coordinate system, the lid is at (0, 0, -2).

When we rotate the pivot around the X axis, the lid will swing open.

The rotation should be around the X axis? No, the hinge is along... hmm.

Let me reconsider. The hinge should be along the back edge of the case. If the case is centered at origin and the back edge is at z = radius = 2, then the hinge line is along the X axis at z=2, y=some height.

Wait, let me orient things differently. Let's say the case body sits on the XZ plane with Y up.

Case body:
- Cylinder centered at (0, 0, 0), radius 2, height 1.5
- Bottom at y = -0.75, top at y = 0.75

The lid sits on top, centered at (0, 0.75, 0), with radius 2.1, height 0.4.

The hinge is at the back edge of the case: (0, 0.75, 2) roughly.

For the lid to open, it should rotate around the X axis at the hinge position.

So create a pivot at (0, 0.75, 2), and the lid is positioned relative to this pivot. In the pivot's local space, the lid center is at (0, 0, -2).

When we rotate the pivot around the X axis (in local space), the lid swings backward.

Actually, we need to rotate around the axis that goes through the hinge point. Let me think of this differently.

The lid center is at (0, 0.95, 0) relative to world. The hinge is at (0, 0.95, 2). The vector from hinge to lid center is (0, 0, -2).

If we create a group at the hinge position and add the lid at position (0, 0, -2) in the group's local space, then rotating the group around its local X axis will swing the lid.

But we need the lid to rotate up and back. Rotating around the X axis at the hinge... 

Let's try:
- Pivot position: (0, hingeY, 2) where hingeY is the lid's Y position
- Lid position in pivot local space: (0, 0, -2) 

When pivot.rotation.x = 0: lid is at world (0, hingeY, 0) ✓

When pivot.rotation.x = π/2: the lid's local (0, 0, -2) becomes:
- y' = 0 * cos + (-2) * sin = -2 * sin(π/2) = -2... wait, rotation around X:
  - y' = y*cos - z*sin = 0 - (-2)(1) = 2
  - z' = y*sin + z*cos = 0 + (-2)(0) = 0
  
So world position: (0, hingeY+2, 2) — the lid would be above the hinge point and at z=2. That doesn't look right.

Actually, I need to think of this more carefully. When the pivot rotates around X at the hinge, the lid should swing up from the hinge.

Let me reconsider: The lid rotates around the X axis at the hinge. The X axis at the hinge is parallel to the world X axis, just offset in Y and Z.

The lid is at position (0, lidY, 0). The hinge is at (0, lidY, 2).

In the pivot's local coordinate system (origin at hinge):
- lid position: (0, 0, -2)

Rotation around the local X axis by angle θ:
- lid_local = (0, 0, -2)
- After rotation: (0, 0*cos - (-2)*sin, 0*sin + (-2)*cos) = (0, 2*sin(θ), -2*cos(θ))

So world position:
- x: 0
- y: hingeY + 2*sin(θ)
- z: 2 - 2*cos(θ) = hingeZ - 2*cos(θ)

At θ=0: (0, hingeY, 0) ✓ (lid at its closed position)
At θ=π/2: (0, hingeY+2, 2) — the lid is above the hinge point, which would be like it swung backward and upward.

That seems right! The lid opens backward from the hinge, swinging up and back.

Let me set hingeY to be the top of the case body + half the lid height. So if case body goes up to y=0.75 and lid height is 0.4, then lid center is at y=0.95, and hinge at y=0.95, z=2.

Actually, let me simplify and make the hinge at a more natural position. The hinge is at the edge where the lid meets the case. So maybe at y=0.75 (top of case body), and z=radius (2).

Let me also think about how to make the lid actually look like a lid. It should be a shallow cylinder covering the top.

OK, let me also think about the visual design. I want:

1. **Case body**: A nicely colored cylinder (maybe blue or green), with some detail.
2. **Two compartments**: Visible as circular recesses on top, with L and R text.
3. **Lid**: A slightly larger cylinder that covers the top, maybe with a different color, and a small tab on the front for opening.

For the L and R labels, I can use TextGeometry, or I can use canvas textures with L and R painted on them. TextGeometry is probably easier with Three.js r128.

Actually, TextGeometry requires a font file. Let me use canvas textures instead for simplicity.

Or I can create 3D text using ExtrudeGeometry with a font... that's complex. Let me use sprite or texture mapping.

Actually, the simplest approach: create a canvas texture with "L" or "R" drawn on it, and apply it to the surface of the compartment.

Let me plan the code:

```javascript
// Scene setup
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(5, 4, 6);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// Lights
const ambientLight = new THREE.AmbientLight(0x404040, 0.6);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
directionalLight.position.set(5, 10, 7);
directionalLight.castShadow = true;
scene.add(directionalLight);

// Controls
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.25;

// Materials
const bodyMaterial = new THREE.MeshPhongMaterial({ color: 0x4a9eda });
const lidMaterial = new THREE.MeshPhongMaterial({ color: 0x2c3e50 });
const compartmentMaterial = new THREE.MeshPhongMaterial({ color: 0xecf0f1 });

// Case body
const bodyGroup = new THREE.Group();
scene.add(bodyGroup);

// Main body cylinder
const bodyGeo = new THREE.CylinderGeometry(2, 1.8, 1.5, 32);
const bodyMesh = new THREE.Mesh(bodyGeo, bodyMaterial);
bodyMesh.position.y = 0;
bodyMesh.castShadow = true;
bodyGroup.add(bodyMesh);

// Compartments (recesses on top)
// Two cylinders with smaller radius, slightly sunk into the top
const compRadius = 0.65;
const compHeight = 0.2;
const compGeo = new THREE.CylinderGeometry(compRadius, compRadius, compHeight, 32);

const compL = new THREE.Mesh(compGeo, compartmentMaterial);
compL.position.set(-0.8, 0.75 - compHeight/2 + 0.01, 0);
bodyGroup.add(compL);

const compR = new THREE.Mesh(compGeo, compartmentMaterial);
compR.position.set(0.8, 0.75 - compHeight/2 + 0.01, 0);
bodyGroup.add(compR);

// Labels for L and R
function createLabel(text, position) {
    const canvas = document.createElement('canvas');
    canvas.width = 128;
    canvas.height = 128;
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = '#4a9eda';
    ctx.fillRect(0, 0, 128, 128);
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 64px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 64, 64);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const geo = new THREE.PlaneGeometry(0.5, 0.5);
    const mesh = new THREE.Mesh(geo, material);
    mesh.position.copy(position);
    mesh.rotation.x = -Math.PI / 2;
    return mesh;
}

const labelL = createLabel('L', new THREE.Vector3(-0.8, 0.8, 0));
bodyGroup.add(labelL);

const labelR = createLabel('R', new THREE.Vector3(0.8, 0.8, 0));
bodyGroup.add(labelR);

// Lid
const lidGroup = new THREE.Group();
// Hinge at back edge
const hingePos = new THREE.Vector3(0, 0.75, 2);
lidGroup.position.copy(hingePos);
scene.add(lidGroup);

// Lid geometry - positioned relative to hinge
const lidGeo = new THREE.CylinderGeometry(2.1, 2.1, 0.4, 32);
const lidMesh = new THREE.Mesh(lidGeo, lidMaterial);
lidMesh.position.set(0, 0.45, -2); // Relative to hinge
lidMesh.castShadow = true;
lidGroup.add(lidMesh);

// Lid top detail
const lidTopGeo = new THREE.CylinderGeometry(1.8, 1.8, 0.05, 32);
const lidTopMesh = new THREE.Mesh(lidTopGeo, new THREE.MeshPhongMaterial({ color: 0x3498db }));
lidTopMesh.position.set(0, 0.65, -2);
lidGroup.add(lidTopMesh);
```

Wait, I need to think about the lid positioning more carefully.

The case body has top at y=0.75. Let me set the hinge at y=0.75, z=2 (the edge of the case).

In the lidGroup's local space:
- The hinge is at origin (0,0,0)
- The lid center would be at (0, 0, -2) (moved back from hinge to center)

The lid geometry: radius 2.1, height 0.4. Its center at (0, 0.2, -2) in local space will make it sit on top of the case body (with bottom at y=0.75).

Wait, no. The lidGroup origin is at (0, 0.75, 2). The lid mesh is at (0, 0, -2) in local space. In world space, that's (0, 0.75, 0).

The lid cylinder needs to extend from y=0.75 to y=1.15 (height 0.4). So the lid mesh center should be at (0, 0.2, -2) in local space, meaning world y = 0.75 + 0.2 = 0.95.

Actually, let me look at this more carefully:

lidGroup position: (0, 0.75, 2)
lidMesh local position: (0, 0.2, -2)
lidMesh world position: (0, 0.75+0.2, 2-2) = (0, 0.95, 0) ✓

The lid cylinder goes from 0.95 - 0.2 = 0.75 to 0.95 + 0.2 = 1.15, which covers the top of the case at y=0.75. 

Now when we rotate lidGroup.rotation.x, the lid will swing open around the X axis at the hinge position.

Hmm, but rotation.x rotates around the local X axis. Since the pivot is at (0, 0.75, 2), the X axis goes horizontally, which is correct for the hinge.

At rotation.x = 0: lid is closed.
At rotation.x = -π/2 or π/2: lid is open.

We want the lid to swing backward (towards -z or +z?). Actually, let me think... The hinge is at z=2 (the back of the case). We want the lid to swing up and back, i.e., rotate so that the front of the lid goes up and the lid opens like a book.

Wait, the hinge is at the back (z=2 forward? or z=2 is toward the camera?).

Let me set up coordinates:
- Camera looks at origin from (5, 4, 6)
- Case body centered at origin
- The "back" of the case would be the far side from the camera. If camera is at (5, 4, 6), the back is roughly toward (-5, 0, -6) direction. Hmm, that's not clean.

Let me just position things naturally. Let me put the hinge at the back edge along the Z axis. Camera from (5, 4, 6), so we see the front and top of the case. The hinge is at z = -2 (the far side), and the lid swings back toward -z.

Actually, let me reconsider. Let me put the hinge at z = -2 (back of the case from camera view), and the lid opens by rotating backward (toward -z).

So hinge position: (0, 0.75, -2)

lidGroup.position = (0, 0.75, -2)
lidMesh.local position: (0, 0.2, 2) — because the lid center should be at world (0, 0.95, 0)

lidMesh world: (0, 0.75+0.2, -2+2) = (0, 0.95, 0) ✓

When lidGroup.rotation.x = -45° (or -π/4), the lid swings backward toward -z.

Let me check: with rotation around X by angle -θ:
- local (0, 0.2, 2) rotates to:
  - y' = 0.2*cos(-θ) - 2*sin(-θ) = 0.2*cos(θ) + 2*sin(θ)
  - z' = 0.2*sin(-θ) + 2*cos(-θ) = -0.2*sin(θ) + 2*cos(θ)

Hmm, wait. Actually when rotating around X in Three.js (right-handed), positive rotation goes from Y to Z.

Let me not get too caught up in the math. I'll just experiment with rotation values and make it work. Let me use rotation.x = -1.2 or something for opening, and visually verify.

Actually, I think the arc would look like opening. Let me simulate:
- rotation.x = 0: lid at (0, 0.2, 2) local → world (0, 0.95, 0) ✓
- rotation.x = -π/4:
  - y' = 0.2*cos(-π/4) - 2*sin(-π/4) = 0.2*0.707 - 2*(-0.707) = 0.141 + 1.414 = 1.555
  - z' = 0.2*sin(-π/4) + 2*cos(-π/4) = 0.2*(-0.707) + 2*0.707 = -0.141 + 1.414 = 1.273
  
  So world position: (0, 0.75+1.555, -2+1.273) = (0, 2.305, -0.727)

- rotation.x = -π/2:
  - y' = 0.2*cos(-π/2) - 2*sin(-π/2) = 0 - 2*(-1) = 2
  - z' = 0.2*sin(-π/2) + 2*cos(-π/2) = 0.2*(-1) + 0 = -0.2
  
  World: (0, 0.75+2, -2-0.2) = (0, 2.75, -2.2)

So the lid swings up and toward -z (backward). That looks correct for opening!

OK so opening the lid = rotation.x goes from 0 to about -1.2 radians (about -70°).

Now let me code the full thing.

One issue: the lid geometry is a cylinder, but when it rotates around the X axis, the cylinder itself should be oriented correctly. The cylinder's axis is along Y. When we rotate the pivot group around X, the entire lid (including the cylinder) rotates, so the lid's Y axis would tilt. That's fine — it will look like the lid is tilting open.

Actually wait - when the lid opens, we might want it to still be "horizontal" (flat) but just at an angle. No, actually when you open a hinged lid, the lid tilts — it pivots around the hinge. So the cylinder being tilted is correct.

Let me code this now.

Actually, let me reconsider the overall design. A contact lens case typically has:
- A round base with two screw-top compartments
- Each compartment has its own small cap, OR
- A single flip-top lid covering both

Let me go with the flip-top design, which is simpler to model.

Let me also add some nice visual details:
- A rim around the case body
- Texture detail on the compartments
- Smooth animation

Let me code:

```javascript
// Scene, camera, renderer, controls setup...

// Case body group
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// Body
const bodyGeo = new THREE.CylinderGeometry(2, 1.8, 1.2, 32);
const bodyMat = new THREE.MeshPhongMaterial({ color: 0x3a7bd5, specular: 0x222222, shininess: 30 });
const bodyMesh = new THREE.Mesh(bodyGeo, bodyMat);
bodyMesh.position.y = -0.1;
bodyMesh.castShadow = true;
caseGroup.add(bodyMesh);

// Compartment recesses (on top of body)
const recessGeo = new THREE.CylinderGeometry(0.65, 0.65, 0.15, 32);
const recessMat = new THREE.MeshPhongMaterial({ color: 0xe8e8e8, specular: 0x111111, shininess: 50 });

const recessL = new THREE.Mesh(recessGeo, recessMat);
recessL.position.set(-0.8, 0.6, 0);
caseGroup.add(recessL);

const recessR = new THREE.Mesh(recessGeo, recessMat);
recessR.position.set(0.8, 0.6, 0);
caseGroup.add(recessR);

// Inner lens part (a smaller cylinder on top of each recess)
const lensGeo = new THREE.CylinderGeometry(0.45, 0.45, 0.1, 32);
const lensMat = new THREE.MeshPhongMaterial({ color: 0x9bd4ff, transparent: true, opacity: 0.6 });

const lensL = new THREE.Mesh(lensGeo, lensMat);
lensL.position.set(-0.8, 0.68, 0);
caseGroup.add(lensL);

const lensR = new THREE.Mesh(lensGeo, lensMat);
lensR.position.set(0.8, 0.68, 0);
caseGroup.add(lensR);
```

Hmm, let me reconsider the positions. The body is a cylinder with height 1.2, centered at y=-0.1. So it goes from -0.7 to 0.5.

The compartments should be on top of the body. Let me raise the body or adjust.

Actually, let me just center the body at origin:
- Body: height 1.2, center at (0, 0, 0), so from -0.6 to 0.6
- Top of body at y = 0.6

Compartments: on top of the body, slightly protruding:
- recessL at (-0.8, 0.6 + 0.075, 0) = (-0.8, 0.675, 0) (cylinder height 0.15)
- recessR at (0.8, 0.675, 0)

Lens inside compartments:
- lensL at (-0.8, 0.675 + 0.05, 0) = (-0.8, 0.725, 0) (height 0.1)

Labels: flat planes on top of the compartments:
- L at (-0.8, 0.73, 0), rotated -π/2 around X
- R at (0.8, 0.73, 0), rotated -π/2 around X

Wait, but if the label is a plane, it needs to sit above the compartment. Let me set the label planes at a slight height above the recess.

Actually, maybe the labels should be on the walls of the case, or on the top surface next to the compartments, or better yet, on the top surface of each compartment.

Let me simplify: put the labels as textured planes on the top of each compartment.

Or better: Make the L and R labels as part of the scene using sprites or 3D text.

Actually, let me use a different approach for L and R: create small 3D extruded text on top of each compartment.

For Three.js r128, I can use FontLoader and TextGeometry. Let me load the font from a CDN.

Hmm, but loading external fonts adds complexity. Let me use canvas textures instead. This is simpler.

Let me create a texture for each label:

```javascript
function createLabelTexture(text, bgColor) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    // Background
    ctx.fillStyle = bgColor || '#2c3e50';
    ctx.fillRect(0, 0, 256, 256);
    
    // Text
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 120px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 128);
    
    return new THREE.CanvasTexture(canvas);
}

const labelLTexture = createLabelTexture('L');
const labelRTexture = createLabelTexture('R');
```

Then apply these textures to small circular planes on top of each compartment.

Hmm, actually let me apply the texture to the top of a small cylinder. Or use a plane.

Let me think differently. I'll create a small cylinder (like the compartment cap) with the label texture on its top face.

Actually, let me just use plane geometry for labels and place them on top of the compartments.

OK I'm overthinking this. Let me just write the code and make it work.

Let me plan the complete solution:

**Structure**:
1. HTML with CSS for full-screen canvas and info text
2. Three.js scene setup
3. Case body (cylinder)
4. Two compartments with L/R labels
5. Hinged lid with click detection
6. Animation for opening/closing
7. OrbitControls for camera rotation

**Click detection**:
- Use raycaster to detect clicks on the lid mesh
- When clicked, toggle open/close state
- Animate the lid rotation from current angle to target angle

**Colors**:
- Case body: Blue (#3a7bd5)
- Compartments: Light gray (#e8e8e8)  
- Lid: Dark blue/navy (#2c3e50)
- Labels: White on dark background

Let me code this now. I'll be careful with the lid positioning and rotation.

Actually, let me reconsider the hinge position. If I set the camera at (5, 4, 6) and look at origin, the "back" of the case (the side facing away from camera) is roughly at positive z? Let me think...

Camera looks from (5, 4, 6) toward (0, 0, 0). The direction is (-5, -4, -6). The back side of the case would be at negative z? No wait... 

Actually, the direction from camera to origin is (-5, -4, -6). So the front of the case (facing camera) has normal pointing roughly in the (-5, -4, -6) direction. The back is opposite: (5, 4, 6).

Hmm, this is getting complicated. Let me simplify by making the hinge at the "back" which is at z = 2 (positive z). When the user looks at the case from the camera position, they'll see the front of the case with the hinge at the far side.

Wait, camera at (5, 4, 6), looking at origin. The direction from camera to origin is towards (-5, -4, -6) — mostly negative z. So the parts of the case at positive z are closer to the camera... no wait.

Camera is at (5, 4, 6), which has z=6. Looking at origin (0,0,0). Points at positive z are closer to the camera (camera at z=6). Points at negative z are farther.

So if the hinge is at z=2 (positive z), that's the near side to the camera. If the hinge is at z=-2, that's the far side.

For the lid to open "backward" (away from the camera), it should hinge at z=-2 and swing toward -z.

But for visualization, maybe I should put the hinge at the back (z=-2), so the user sees the case from the front and can click the lid. But when the lid opens, it swings away from the camera, which might be harder to see.

Alternatively, I could put the hinge at the back and have the lid swing up and backward, and the user can rotate the camera with OrbitControls to see it.

Let me just use hinge at (0, y, -2), with rotation around X to open backward.

Actually, let me reconsider. Let me put the hinge at (0, 0, 2) — the near side. Then the lid opens toward the camera. But that would feel like it's opening toward the user, which is weird.

OK let me just stick with the hinge at the back: (0, lidY, -2). The lid opens backward (toward -z). The user can rotate camera to see the opened position.

Let me now write the full code.

Actually, wait. Let me reconsider the lid opening animation. Instead of a hinge at the back, maybe I should make the lid simpler: it just rotates up and back around the X axis at the back edge.

Actually, I realize there's another issue. The CylinderGeometry for the lid is a vertical cylinder. When the pivot rotates, the whole lid rotates. At the fully open position, the lid would be standing nearly vertical. That's actually correct for a flip-top lid.

OK, here's my refined plan. Let me also add:
- A small handle/tab on the lid to make it easier to see where to click
- A highlight when hovering over the lid
- Smooth animation with easing

Let me write the code now.

Actually, let me reconsider the design once more. A contact lens case typically has two separate screw caps, not a flip-top lid. But the user's request says "盒盖" (case lid) and says "点击盒盖将其打开" (click the lid to open it). So a flip-top lid is fine.

Let me also think about whether I need to make the lid clickable only, or the whole case. The instruction says "点击盒盖将其打开" — click the lid to open it. So only the lid responds to clicks.

For hover effect, I'll change the material color or add an emissive glow.

Let me now write the complete code.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>隐形眼镜盒 3D 模型</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            overflow: hidden;
            background: linear-gradient(135deg, #0c0c1d 0%, #1a1a3e 50%, #2d1b69 100%);
            font-family: 'Microsoft YaHei', Arial, sans-serif;
        }
        canvas { display: block; }
        
        #info {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            color: #ffffff;
            background: rgba(0, 0, 0, 0.6);
            padding: 12px 24px;
            border-radius: 30px;
            font-size: 15px;
            letter-spacing: 1px;
            border: 1px solid rgba(255,255,255,0.15);
            backdrop-filter: blur(10px);
            pointer-events: none;
            user-select: none;
            z-index: 10;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        
        #status {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: #ffd700;
            background: rgba(0,0,0,0.5);
            padding: 6px 16px;
            border-radius: 20px;
            font-size: 14px;
            border: 1px solid rgba(255,215,0,0.3);
            pointer-events: none;
            user-select: none;
            z-index: 10;
            opacity: 0;
            transition: opacity 0.3s;
        }
    </style>
</head>
<body>
    <div id="info">🖱️ 点击盖子 打开/关闭 · 拖动旋转视角</div>
    <div id="status"></div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script>
        // ... complete code here
    </script>
</body>
</html>
```

Now let me write the JavaScript. I need to be careful about:

1. Lid positioning relative to hinge
2. Click detection using raycasting
3. Animation using requestAnimationFrame

Here's the plan for the lid motion:
- `lidOpen` state: boolean
- `currentAngle`: current rotation angle of lidGroup
- `targetAngle`: 0 (closed) or -1.2 radians (open)
- Animate by lerping `currentAngle` toward `targetAngle` in the animation loop

Let me code it:

```javascript
// Scene setup
const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(4, 3, 7);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// Controls
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.25;
controls.minDistance = 3;
controls.maxDistance = 15;
controls.target.set(0, 0, 0);

// Lights
const ambientLight = new THREE.AmbientLight(0x404060, 0.6);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.9);
directionalLight.position.set(5, 8, 6);
directionalLight.castShadow = true;
directionalLight.shadow.mapSize.width = 2048;
directionalLight.shadow.mapSize.height = 2048;
directionalLight.shadow.camera.near = 0.5;
directionalLight.shadow.camera.far = 20;
directionalLight.shadow.camera.left = -5;
directionalLight.shadow.camera.right = 5;
directionalLight.shadow.camera.top = 5;
directionalLight.shadow.camera.bottom = -5;
scene.add(directionalLight);

const fillLight = new THREE.DirectionalLight(0x4488cc, 0.4);
fillLight.position.set(-3, 2, -4);
scene.add(fillLight);

// Ground
const groundGeo = new THREE.CircleGeometry(8, 64);
const groundMat = new THREE.MeshPhongMaterial({ 
    color: 0x1a1a3e, 
    specular: 0x111111, 
    shininess: 10 
});
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.position.y = -0.7;
ground.receiveShadow = true;
scene.add(ground);
```

Now the case body:

```javascript
// Case body group
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// Body material
const bodyMat = new THREE.MeshPhongMaterial({ 
    color: 0x4a9eda, 
    specular: 0x222222, 
    shininess: 40 
});

// Body cylinder
const bodyGeo = new THREE.CylinderGeometry(1.8, 1.6, 1.2, 32);
const bodyMesh = new THREE.Mesh(bodyGeo, bodyMat);
bodyMesh.position.y = -0.1;
bodyMesh.castShadow = true;
bodyMesh.receiveShadow = true;
caseGroup.add(bodyMesh);

// Body rim (bottom detail)
const rimGeo = new THREE.TorusGeometry(1.6, 0.08, 16, 32);
const rimMat = new THREE.MeshPhongMaterial({ color: 0x2c6faa });
const rim = new THREE.Mesh(rimGeo, rimMat);
rim.position.y = -0.7;
rim.rotation.x = Math.PI / 2;
caseGroup.add(rim);

// Top surface of body - a flat cylinder
const topSurfaceGeo = new THREE.CylinderGeometry(1.8, 1.8, 0.05, 32);
const topSurfaceMat = new THREE.MeshPhongMaterial({ color: 0x5aaeed });
const topSurface = new THREE.Mesh(topSurfaceGeo, topSurfaceMat);
topSurface.position.y = -0.1 + 0.6 + 0.025; // body center + half height + half surface height
caseGroup.add(topSurface);
```

Wait, the body cylinder is centered at y=-0.1 with height 1.2. So it goes from -0.7 to 0.5. The top surface should be at y = 0.5.

Let me simplify: put the body at y=0, going from -0.6 to 0.6.

```javascript
// Body
const bodyMesh = new THREE.Mesh(
    new THREE.CylinderGeometry(1.8, 1.6, 1.2, 32),
    bodyMat
);
bodyMesh.position.y = 0;
bodyMesh.castShadow = true;
caseGroup.add(bodyMesh);
```

Body goes from -0.6 to 0.6. Top of body at y=0.6.

Compartments on top:
```javascript
// Compartments
const compMat = new THREE.MeshPhongMaterial({ 
    color: 0xe8e8e8, 
    specular: 0x222222, 
    shininess: 60 
});

// Compartment walls (short cylinders)
const wallGeo = new THREE.CylinderGeometry(0.7, 0.7, 0.25, 32, 1, true);
const wallL = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 0.2, 32, 1, true), compMat);
wallL.position.set(-0.85, 0.6 + 0.1, 0);
caseGroup.add(wallL);

const wallR = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 0.2, 32, 1, true), compMat);
wallR.position.set(0.85, 0.6 + 0.1, 0);
caseGroup.add(wallR);

// Compartment interiors (deeper recesses)
const recessGeo = new THREE.CylinderGeometry(0.55, 0.55, 0.15, 32);
const recessMat = new THREE.MeshPhongMaterial({ color: 0xb8d4e8 });
const recessL = new THREE.Mesh(recessGeo, recessMat);
recessL.position.set(-0.85, 0.6 + 0.075, 0);
caseGroup.add(recessL);

const recessR = new THREE.Mesh(recessGeo, recessMat);
recessR.position.set(0.85, 0.6 + 0.075, 0);
caseGroup.add(recessR);
```

Hmm actually, let me reconsider. The compartment walls and interiors are separate meshes. For simplicity, let me just create two cylindrical compartments on top of the body.

Each compartment:
- Outer ring: a cylinder
- Inner recess: a slightly smaller cylinder/surface

Actually, the simplest approach: Create a single cylinder for each compartment, slightly recessed into the body top. The compartments are just cylinders sticking out slightly.

Let me simplify further:

Compartment = a shallow cylinder sitting on top of the body, with a hole in the middle (like a donut). But creating a donut with cylinder geometry is complex.

Alternative approach:
1. Add a larger cylinder (the compartment) on top of the body
2. Add a smaller cylinder (the lens area) on top of that

Or:
1. Create the body with two circular indentations using boolean operations (CSG) — too complex.

Let me go with the simple approach:
- Two shallow cylinders on top of the body (the compartment bases)
- Each has a label (L/R)
- A small ring/donut around each compartment for visual detail

Actually, let me avoid overcomplicating this. Here's my final simple approach:

**Case body**: A solid cylinder with two slightly-raised circular platforms on top.

```javascript
// Compartment platforms
const platformGeo = new THREE.CylinderGeometry(0.8, 0.8, 0.15, 32);
const platformMat = new THREE.MeshPhongMaterial({ color: 0xd0e8f8, shininess: 40 });

const platformL = new THREE.Mesh(platformGeo, platformMat);
platformL.position.set(-0.85, 0.6 + 0.075, 0);
caseGroup.add(platformL);

const platformR = new THREE.Mesh(platformGeo, platformMat);
platformR.position.set(0.85, 0.6 + 0.075, 0);
caseGroup.add(platformR);

// Compartment inner circles (slightly recessed)
const innerGeo = new THREE.CylinderGeometry(0.5, 0.5, 0.08, 32);
const innerMat = new THREE.MeshPhongMaterial({ color: 0x9bc4e0, shininess: 30 });

const innerL = new THREE.Mesh(innerGeo, innerMat);
innerL.position.set(-0.85, 0.6 + 0.04, 0);
caseGroup.add(innerL);

const innerR = new THREE.Mesh(innerGeo, innerMat);
innerR.position.set(0.85, 0.6 + 0.04, 0);
caseGroup.add(innerR);
```

Labels on top of each platform:

```javascript
function createLabel(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, 256, 256);
    
    // Circle background
    ctx.beginPath();
    ctx.arc(128, 128, 110, 0, Math.PI * 2);
    ctx.fillStyle = '#2c3e50';
    ctx.fill();
    
    // Text
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 132);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const plane = new THREE.PlaneGeometry(0.7, 0.7);
    const mesh = new THREE.Mesh(plane, material);
    mesh.rotation.x = -Math.PI / 2;
    mesh.position.y = 0.02;
    return mesh;
}

const labelL = createLabel('L');
labelL.position.x = -0.85;
caseGroup.add(labelL);

const labelR = createLabel('R');
labelR.position.x = 0.85;
caseGroup.add(labelR);
```

Now the lid:

```javascript
// Lid group (pivot at hinge)
const lidGroup = new THREE.Group();
const hingeY = 0.6 + 0.15; // top of platform, approximately
const hingeZ = -1.8; // back edge of case
lidGroup.position.set(0, hingeY, hingeZ);
scene.add(lidGroup);

// Lid mesh
const lidMat = new THREE.MeshPhongMaterial({ 
    color: 0x2c3e50, 
    specular: 0x333333, 
    shininess: 50 
});

const lidGeo = new THREE.CylinderGeometry(1.9, 1.9, 0.35, 32);
const lidMesh = new THREE.Mesh(lidGeo, lidMat);
lidMesh.position.set(0, 0.175 + 0.15, -hingeZ + 0); // relative to hinge
lidGroup.add(lidMesh);
```

Hmm wait, I need to calculate the lid position correctly.

The hinge is at (0, hingeY, hingeZ) = (0, 0.75, -1.8).
The lid should be centered at (0, 0.75 + 0.175, 0) in world space when closed. In the hinge's local space:
- x: 0
- y: 0.75 + 0.175 - 0.75 = 0.175
- z: 0 - (-1.8) = 1.8

So lidMesh.position.set(0, 0.175, 1.8) relative to lidGroup.

Wait, but the lid is a cylinder with height 0.35, centered at y=0.175 in local space. From y=0 to y=0.35 in local space, which corresponds to world y=0.75 to y=1.1. The original body top is at y=0.6, and the platforms are at y=0.6 to 0.75. So the lid at y=0.75 covers the platforms nicely.

Actually, let me check: the platforms are at y=0.6 to 0.75 (height 0.15, center at 0.675). So the top of platforms is at y=0.75. The lid bottom at y=0.75 covers them perfectly.

But I need to also account for the lid extending beyond the body. The body has radius 1.8 at the top, so the lid should be a bit larger: radius 1.9.

OK so:
- hinge at (0, 0.75, -1.8)
- lid mesh at local (0, 0.175, 1.8), world (0, 0.925, 0) — center of lid
- lid bottom at world y=0.75, top at world y=1.1

Wait, if lid center is at world y=0.925 and height is 0.35, then bottom at 0.925-0.175=0.75, top at 0.925+0.175=1.1. ✓

Now when lidGroup.rotation.x changes, the lid swings open.

For the lid to look right, I also want a lid top detail:
```javascript
const lidTopGeo = new THREE.CylinderGeometry(1.7, 1.7, 0.05, 32);
const lidTopMat = new THREE.MeshPhongMaterial({ color: 0x3d5a80 });
const lidTop = new THREE.Mesh(lidTopGeo, lidTopMat);
lidTop.position.set(0, 0.35, 1.8); // top of lid in local space
lidGroup.add(lidTop);
```

Hmm, wait. The lid cylinder goes from y=0 to y=0.35 in local space. The top surface is at y=0.35. So lidTop should be at y=0.35 + 0.025 = 0.375, but actually I want it to be flush, so at y=0.35, with height 0.05 and top at 0.375. Or just at y=0.35 with thickness 0.05, so from 0.325 to 0.375.

Actually, let me just add a decorative disc on top:
```javascript
const lidTop = new THREE.Mesh(
    new THREE.CylinderGeometry(1.7, 1.7, 0.06, 32),
    new THREE.MeshPhongMaterial({ color: 0x3d5a80, shininess: 30 })
);
lidTop.position.set(0, 0.35, 1.8);
lidGroup.add(lidTop);
```

And a small handle on the front of the lid:
```javascript
const handleGeo = new THREE.BoxGeometry(0.3, 0.15, 0.4);
const handleMesh = new THREE.Mesh(handleGeo, new THREE.MeshPhongMaterial({ color: 0x3d5a80 }));
handleMesh.position.set(0, 0.1, 3.6); // front of the lid
lidGroup.add(handleMesh);
```

Hmm, the handle would be at z = 3.6 in local coordinates, which is... the lid radius is 1.9, so the front edge of the lid is at z=1.8+1.9=3.7 from the hinge? Let me recalculate.

The hinge is at z=-1.8. The lid center is at z=0, so the local z from hinge to lid center is 1.8. The lid extends from center: radius 1.9. So in local coordinates, the front edge of the lid is at z = 1.8 + 1.9 = 3.7.

The handle should be at the front edge of the lid, sticking out:
```javascript
handleMesh.position.set(0, -0.02, 3.7);
```

Wait, but the handle should be on the front of the lid (toward the camera). Since the hinge is at the back (z=-1.8), the front is at z=1.8+1.9=3.7 in local space.

Actually, let me reconsider. The camera sees the case from (4, 3, 7). The front of the case is at positive z (toward camera). The hinge is at negative z (back). So the lid opens backward (away from camera).

This is correct! The lid opens away from the camera, like a real flip-top lid.

Now the handle: on the front edge of the lid, sticking outward (toward positive z).

```javascript
const handleMesh = new THREE.Mesh(
    new THREE.BoxGeometry(0.4, 0.2, 0.3),
    new THREE.MeshPhongMaterial({ color: 0x3d5a80 })
);
handleMesh.position.set(0, 0.35, 3.7);
lidGroup.add(handleMesh);
```

The handle should be on top of the lid near the front edge.

OK let me also add a visual indicator for the compartments from the side.

Now, for the raycasting:

```javascript
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

// Track the lid meshes
const clickableMeshes = [lidMesh, lidTop, handleMesh];

renderer.domElement.addEventListener('click', onClick);

function onClick(event) {
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(clickableMeshes);
    
    if (intersects.length > 0) {
        toggleLid();
    }
}
```

For animation:

```javascript
let isOpen = false;
let currentAngle = 0;
let targetAngle = 0;
const OPEN_ANGLE = -1.2; // radians, ~ -69°
const ANIMATION_SPEED = 0.08;

function toggleLid() {
    isOpen = !isOpen;
    targetAngle = isOpen ? OPEN_ANGLE : 0;
    updateStatus();
}

// In animation loop:
function animate() {
    requestAnimationFrame(animate);
    
    // Lerp the angle
    currentAngle += (targetAngle - currentAngle) * ANIMATION_SPEED;
    if (Math.abs(currentAngle - targetAngle) < 0.01) {
        currentAngle = targetAngle;
    }
    
    lidGroup.rotation.x = currentAngle;
    
    controls.update();
    renderer.render(scene, camera);
}
```

Hmm, the ANIMATION_SPEED approach with lerp gives an exponential easing, which looks nice. But let me use a smoother approach with delta time:

```javascript
function animate() {
    requestAnimationFrame(animate);
    
    const delta = clock.getDelta();
    
    // Smooth interpolation
    const lerpFactor = 1 - Math.pow(0.001, delta);
    currentAngle += (targetAngle - currentAngle) * lerpFactor;
    
    lidGroup.rotation.x = currentAngle;
    
    controls.update();
    renderer.render(scene, camera);
}
```

OK, let me put it all together now. Let me also think about potential issues:

1. The label planes need to be oriented correctly. They should be flat on top of the compartments, facing up.
2. The raycasting should only trigger on actual clicks, not drags. I need to distinguish between click and drag.

For issue 2, I'll track mouse down position and mouse up position, and only trigger click if the distance is small.

Let me also add a hover effect on the lid.

Here's my complete implementation plan:

```javascript
// Scene, camera, renderer, controls, lights...

// Case body
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// Body cylinder
const bodyMesh = new THREE.Mesh(
    new THREE.CylinderGeometry(1.8, 1.6, 1.2, 32),
    new THREE.MeshPhongMaterial({ color: 0x4a9eda, shininess: 40 })
);
bodyMesh.position.y = 0;
bodyMesh.castShadow = true;
bodyMesh.receiveShadow = true;
caseGroup.add(bodyMesh);

// Bottom rim
const bottomRim = new THREE.Mesh(
    new THREE.TorusGeometry(1.6, 0.08, 16, 32),
    new THREE.MeshPhongMaterial({ color: 0x2c6faa })
);
bottomRim.position.y = -0.6;
bottomRim.rotation.x = Math.PI / 2;
caseGroup.add(bottomRim);

// Compartment platforms
const platformMat = new THREE.MeshPhongMaterial({ color: 0xd0e8f8, shininess: 40 });
const platformL = new THREE.Mesh(
    new THREE.CylinderGeometry(0.75, 0.75, 0.15, 32),
    platformMat
);
platformL.position.set(-0.85, 0.675, 0);
caseGroup.add(platformL);

const platformR = new THREE.Mesh(
    new THREE.CylinderGeometry(0.75, 0.75, 0.15, 32),
    platformMat
);
platformR.position.set(0.85, 0.675, 0);
caseGroup.add(platformR);

// Inner lens area (smaller circles on platforms)
const innerMat = new THREE.MeshPhongMaterial({ color: 0x9bc4e0, shininess: 30 });
const innerL = new THREE.Mesh(
    new THREE.CylinderGeometry(0.5, 0.5, 0.06, 32),
    innerMat
);
innerL.position.set(-0.85, 0.675 + 0.075, 0);
caseGroup.add(innerL);

const innerR = new THREE.Mesh(
    new THREE.CylinderGeometry(0.5, 0.5, 0.06, 32),
    innerMat
);
innerR.position.set(0.85, 0.675 + 0.075, 0);
caseGroup.add(innerR);

// Labels
function createLabel(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, 256, 256);
    
    // Circle background
    ctx.beginPath();
    ctx.arc(128, 128, 105, 0, Math.PI * 2);
    ctx.fillStyle = '#2c3e50';
    ctx.fill();
    ctx.strokeStyle = '#5d8aaf';
    ctx.lineWidth = 4;
    ctx.stroke();
    
    // Text
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 130px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 134);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const mesh = new THREE.Mesh(new THREE.PlaneGeometry(0.65, 0.65), material);
    mesh.rotation.x = -Math.PI / 2;
    return mesh;
}

const labelL = createLabel('L');
labelL.position.set(-0.85, 0.675 + 0.075 + 0.031, 0);
caseGroup.add(labelL);

const labelR = createLabel('R');
labelR.position.set(0.85, 0.675 + 0.075 + 0.031, 0);
caseGroup.add(labelR);
```

Wait, the inner cylinders are at height 0.06, centered at y = 0.675 + 0.075 = 0.75. So the top of the inner cylinders is at 0.75 + 0.03 = 0.78. The labels should be just above that: at 0.78 + 0.02 = 0.80.

Actually, let me reconsider the layering:
- Platform: center at 0.675, from 0.6 to 0.75
- Inner: center at 0.75 + 0.03 = 0.78 if height is 0.06. That would be above the platform.

Hmm, I want the inner to be a recess, not a protrusion. So the inner should be a cylinder that goes DOWN from the platform top, or... 

Actually, for a contact lens case, the compartments are recesses in the top. So the platform has a concave surface. But I can't easily make concave geometry.

Alternative: Make the inner circle a different color and slightly lower than the platform top:
- The inner cylinder should go from the platform top (0.75) slightly DOWN.

But CylinderGeometry creates a solid cylinder. To create a recess, I'd need to subtract geometry, which is complex.

Simplest approach: Just put a smaller circle (flat disc) on top of the platform with a different color, and add a ring around it. The visual effect will suggest a recess.

```javascript
// Ring around the inner area
const ringGeo = new THREE.TorusGeometry(0.5, 0.03, 16, 32);
const ringMat = new THREE.MeshPhongMaterial({ color: 0x5d8aaf });
const ringL = new THREE.Mesh(ringGeo, ringMat);
ringL.position.set(-0.85, 0.75 + 0.03, 0);
ringL.rotation.x = Math.PI / 2;
caseGroup.add(ringL);

const ringR = new THREE.Mesh(ringGeo, ringMat);
ringR.position.set(0.85, 0.75 + 0.03, 0);
ringR.rotation.x = Math.PI / 2;
caseGroup.add(ringR);
```

OK this is getting complicated. Let me simplify significantly.

I'll create the case with simple shapes:
1. A body cylinder
2. Two colored circles on top (using thin cylinders) with L/R labels
3. A lid

That's it. Clean and simple.

Here's my simplified plan:

```javascript
// Body
const body = new THREE.Mesh(
    new THREE.CylinderGeometry(1.8, 1.6, 1.2, 32),
    new THREE.MeshPhongMaterial({ color: 0x4a9eda, shininess: 40 })
);
body.position.y = 0;
body.castShadow = true;
caseGroup.add(body);

// Top surface: a thinner cylinder covering the top of body
const topDisc = new THREE.Mesh(
    new THREE.CylinderGeometry(1.78, 1.78, 0.04, 32),
    new THREE.MeshPhongMaterial({ color: 0x6bb5ef })
);
topDisc.position.y = 0.6;
caseGroup.add(topDisc);

// Compartment markers: two circles on top
const leftComp = new THREE.Mesh(
    new THREE.CircleGeometry(0.65, 32),
    new THREE.MeshPhongMaterial({ color: 0xc5e4f7 })
);
leftComp.position.set(-0.8, 0.62, 0);
leftComp.rotation.x = -Math.PI / 2;
caseGroup.add(leftComp);

const rightComp = new THREE.Mesh(
    new THREE.CircleGeometry(0.65, 32),
    new THREE.MeshPhongMaterial({ color: 0xc5e4f7 })
);
rightComp.position.set(0.8, 0.62, 0);
rightComp.rotation.x = -Math.PI / 2;
caseGroup.add(rightComp);

// Labels
// ... createLabel planes positioned just above the circles
```

Lid:
```javascript
// Hinge at back edge
const lidGroup = new THREE.Group();
lidGroup.position.set(0, 0.62, -1.8);
scene.add(lidGroup);

// Lid
const lidMesh = new THREE.Mesh(
    new THREE.CylinderGeometry(1.85, 1.85, 0.3, 32),
    new THREE.MeshPhongMaterial({ color: 0x2c3e50, shininess: 50 })
);
lidMesh.position.set(0, 0.15, 1.8); // local
lidGroup.add(lidMesh);
```

Wait, the hinge is at y=0.62. The lid bottom should be at y=0.62. The lid height is 0.3, so center at y=0.62+0.15=0.77. In local coordinates, the lid center is at y=0.15, z=1.8 (from hinge to center of case).

Let me verify: hinge world position (0, 0.62, -1.8). Lid local (0, 0.15, 1.8). World: (0, 0.62+0.15, -1.8+1.8) = (0, 0.77, 0). ✓

The bottom of the lid at world y=0.62, which matches the top of the body + topDisc.

When lidGroup.rotation.x changes from 0 to OPEN_ANGLE, the lid swings up and back.

Now let me also add a lid inner surface (visible when open):
```javascript
const lidInner = new THREE.Mesh(
    new THREE.CircleGeometry(1.75, 32),
    new THREE.MeshPhongMaterial({ color: 0x4a5a6a, side: THREE.DoubleSide })
);
lidInner.position.set(0, 0, 1.8); // at the bottom of the lid
lidInner.rotation.x = -Math.PI / 2;
lidGroup.add(lidInner);
```

The lidInner is at the bottom of the lid, facing downward when closed, but visible when open.

Also, let me add a small lip/rim on the lid for visual detail:
```javascript
const lidRim = new THREE.Mesh(
    new THREE.TorusGeometry(1.85, 0.03, 16, 32),
    new THREE.MeshPhongMaterial({ color: 0x1a2530 })
);
lidRim.position.set(0, 0, 1.8);
lidRim.rotation.x = Math.PI / 2;
lidGroup.add(lidRim);
```

OK, let me also add a small handle/tab on the front of the lid:
```javascript
const lidTab = new THREE.Mesh(
    new THREE.BoxGeometry(0.3, 0.15, 0.15),
    new THREE.MeshPhongMaterial({ color: 0x1a2530 })
);
lidTab.position.set(0, 0.05, 3.65); // front edge of lid
lidGroup.add(lidTab);
```

The lid radius is 1.85, and the lid center is at z=1.8 from hinge. So the front edge is at z=1.8+1.85=3.65. The tab should stick out from the front edge at the bottom.

Let me also think about the handle: should it be on the front or top? Since the lid pivots at the back, the handle on the front makes sense — it would be the part you grab.

Now, for the interaction:
- The user clicks on the lid to toggle open/close
- Hover effect: change lid color

Let me add hover detection:

```javascript
// Track mouse for hover
let isHovering = false;

renderer.domElement.addEventListener('mousemove', (event) => {
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(clickableMeshes);
    
    if (intersects.length > 0) {
        if (!isHovering) {
            isHovering = true;
            document.body.style.cursor = 'pointer';
            lidMat.emissive.setHex(0x112233);
        }
    } else {
        if (isHovering) {
            isHovering = false;
            document.body.style.cursor = 'default';
            lidMat.emissive.setHex(0x000000);
        }
    }
});
```

OK, let me now write the complete code. This will be quite long but let me make it good.

Actually, let me also reconsider the labels. Using canvas texture with text is good. But the label planes need to be positioned correctly.

For the labels, I'll create circular textures with L/R and apply them to small circular meshes on top of the compartments.

Here's a refined label approach:

```javascript
function createLabelMesh(text) {
    // Create canvas
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    // Transparent background
    ctx.clearRect(0, 0, 256, 256);
    
    // Draw circular background
    ctx.save();
    ctx.beginPath();
    ctx.arc(128, 128, 110, 0, Math.PI * 2);
    ctx.fillStyle = '#2c3e50';
    ctx.fill();
    ctx.lineWidth = 6;
    ctx.strokeStyle = '#5d8aaf';
    ctx.stroke();
    ctx.restore();
    
    // Draw text
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 134);
    
    // Create texture
    const texture = new THREE.CanvasTexture(canvas);
    texture.needsUpdate = true;
    
    // Create circular mesh
    const geometry = new THREE.CircleGeometry(0.55, 32);
    const material = new THREE.MeshBasicMaterial({ 
        map: texture, 
        transparent: true,
        side: THREE.DoubleSide
    });
    const mesh = new THREE.Mesh(geometry, material);
    
    return mesh;
}

const labelL = createLabelMesh('L');
labelL.position.set(-0.8, 0.64, 0);
labelL.rotation.x = -Math.PI / 2;
caseGroup.add(labelL);

const labelR = createLabelMesh('R');
labelR.position.set(0.8, 0.64, 0);
labelR.rotation.x = -Math.PI / 2;
caseGroup.add(labelR);
```

But wait, if TWO meshes use the same material, they'll share the texture. But since L and R are different, they need different materials/textures.

That's fine — createLabelMesh creates a new texture and material for each.

Also, the label meshes need to be above the compartment top surface. The compartment circles are at y=0.62. The labels should be just slightly above at y=0.63 or so.

Actually, let me reconsider the compartment design:
- The body has a flat top at y=0.6
- Two compartment circles are flat discs on top of the body
- The labels sit on top of those discs

So:
- Compartment discs at y=0.6 + 0.01 = 0.61 (avoid z-fighting)
- Labels at y=0.61 + 0.01 = 0.62

Hmm, that's too low. Let me give the compartments some actual thickness:

- Two small cylinders as compartment platforms: center at y=0.6+0.12=0.72, height 0.24. Bottom at 0.6, top at 0.84.
- Labels on top of these platforms at y=0.84+0.01=0.85.

That's more visible.

Actually, you know what, let me just make this simpler. Let me raise the body top a bit and put everything on top.

Let me finalize:

**Body**: Cylinder radius 1.8 (top), 1.6 (bottom), height 1.2, centered at origin. Top at y=0.6.

**Compartment platforms**: Two cylinders radius 0.7, height 0.2, sitting on the body top. Centers at (-0.8, 0.7, 0) and (0.8, 0.7, 0). So they go from y=0.6 to y=0.8.

**Labels**: Circle geometry with canvas texture, sitting on top of platforms at y=0.81.

**Lid**: Cylinder radius 1.85, height 0.3, hinged at the back edge (0, 0.8, -1.8).

**Lid position**: Since hinge is at y=0.8, lid center at y=0.8+0.15=0.95. In local: (0, 0.15, 1.8).

Let me code this.

Hmm wait, the lid radius should cover the compartments. The body top radius is 1.8. At y=0.8, the body might have slightly smaller radius. Let me make the lid radius 1.85 to cover everything.

But also, the hinge is at z=-1.8. The lid extends from the hinge to z=1.8 in local space, and has radius 1.85. So the lid would extend from z=-1.8+1.85=-0.05 to z=1.8+1.85=3.65? No...

Actually, the lid is centered at z=1.8 (from hinge). The lid's cylindrical axis is along Y. The radius is 1.85. So the lid extends from z=1.8-1.85=-0.05 to z=1.8+1.85=3.65. The hinge is at z=-1.8. So the lid covers from z=-0.05 to z=3.65. The case body extends from z=-1.8 to z=1.8. So the lid hangs over the front edge (positive z) by 3.65-1.8=1.85? That can't be right.

Oh wait, I'm confused. The lid is a cylinder. The cylinder's center is at z=1.8 (from hinge), with radius 1.85. The cylinder spans z from 1.8-1.85=-0.05 to 1.8+1.85=3.65. 

But the body spans z from -1.8 to 1.8 (radius 1.8 at top, centered at origin). So the lid would NOT cover the back part of the body (from z=-1.8 to z=-0.05). That's a problem!

The lid needs to cover the entire body top. So the lid center should be at z=0 in world coordinates when closed. Since the hinge is at z=-1.8, the lid center in local coordinates should be at z=1.8.

But then the lid extends from z=1.8-1.85=-0.05 to z=1.8+1.85=3.65 in local, which is z=-0.05 to z=3.65 in world. That covers 0 to 1.8 (front half of body) but NOT the back half from z=-1.8 to v=0.

Ah, I need to make the lid centered at the case center (z=0 in world) so it covers the whole top. So lid center at local z=1.8. The lid radius should be at least 1.8 to cover the body.

But 1.8 radius at center z=1.8 gives z from 0 to 3.6... let me recalculate.

If lid center is at local z=1.8 (world z=0), and radius is 1.85:
- The lid spans in z from 1.8-1.85=-0.05 to 1.8+1.85=3.65 (local)
- World: from -0.05-1.8=-1.85? No wait.
- Local z=-0.05 → world z=-1.8+(-0.05)=-1.85
- Local z=3.65 → world z=-1.8+3.65=1.85

So the lid spans from z=-1.85 to z=1.85 in world coordinates. The body top has radius 1.8, so it spans from z=-1.8 to z=1.8. The lid covers it. ✓

Great, that makes sense! The lid just covers the body top nicely.

OK, so the lid is centered at the world origin (z=0) when closed. Radius 1.85 covers the body top (radius 1.8).

Now the hinge at z=-1.8. The lid rotates around the hinge, opening backward (toward -z).

When the lid is open (rotated by -1.2 rad around X):
- The lid swings up and backward

Let me verify it looks right.

OK, I think I have a good plan. Let me write the complete code now.

Wait, I also need to handle the case where the lid has a slight overhang at the front (for easy opening). Let me add that as a separate tab/handle.

Also, I want to make the body look nicer. Let me add a subtle texture or gradient.

Actually, let me use a slightly different approach for the case body: instead of a single cylinder, use a group with multiple cylinders and rings for detail.

OK, let me just write it. Here's the final code:

Actually, one more consideration: I should handle the click vs drag issue. OrbitControls uses mouse drag to rotate. So when the user clicks (mousedown + mouseup without significant movement) on the lid, it should toggle.

I'll track mousedown position and mouseup position, and only trigger click if the distance is small.

```javascript
let mouseDownPos = { x: 0, y: 0 };

renderer.domElement.addEventListener('pointerdown', (e) => {
    mouseDownPos = { x: e.clientX, y: e.clientY };
});

renderer.domElement.addEventListener('pointerup', (e) => {
    const dx = e.clientX - mouseDownPos.x;
    const dy = e.clientY - mouseDownPos.y;
    if (Math.sqrt(dx*dx + dy*dy) < 5) {
        handleClick(e);
    }
});
```

OK, let me write the complete final code now. I'll aim for a polished look.

Let me also add some subtle animations:
- The lid has a spring-like animation (slight bounce)
- A small particle or glow effect? No, too much.

Let me keep it simple and clean.

Here's my final code plan:

```javascript
// ... setup scene, camera, renderer, lights

// GROUND
const ground = new THREE.Mesh(
    new THREE.CircleGeometry(6, 64),
    new THREE.MeshPhongMaterial({ color: 0x1a1a3e, shininess: 10 })
);
ground.rotation.x = -Math.PI / 2;
ground.position.y = -0.75;
ground.receiveShadow = true;
scene.add(ground);

// CASE BODY
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// Body cylinder
const body = new THREE.Mesh(
    new THREE.CylinderGeometry(1.8, 1.6, 1.2, 48),
    new THREE.MeshPhongMaterial({ color: 0x4a9eda, specular: 0x222222, shininess: 40 })
);
body.position.y = 0;
body.castShadow = true;
body.receiveShadow = true;
caseGroup.add(body);

// Body detail ring
const detailRing = new THREE.Mesh(
    new THREE.TorusGeometry(1.7, 0.06, 16, 48),
    new THREE.MeshPhongMaterial({ color: 0x2c6faa })
);
detailRing.position.y = -0.1;
detailRing.rotation.x = Math.PI / 2;
caseGroup.add(detailRing);

// Top disc
const topDisc = new THREE.Mesh(
    new THREE.CylinderGeometry(1.78, 1.78, 0.05, 48),
    new THREE.MeshPhongMaterial({ color: 0x6bb5ef, shininess: 30 })
);
topDisc.position.y = 0.6 + 0.025;
caseGroup.add(topDisc);

// COMPARTMENT PLATFORMS
const platformMat = new THREE.MeshPhongMaterial({ color: 0xd0e8f8, shininess: 40 });
const platformL = new THREE.Mesh(
    new THREE.CylinderGeometry(0.7, 0.7, 0.2, 32),
    platformMat
);
platformL.position.set(-0.8, 0.6 + 0.1, 0);
platformL.castShadow = true;
caseGroup.add(platformL);

const platformR = new THREE.Mesh(
    new THREE.CylinderGeometry(0.7, 0.7, 0.2, 32),
    platformMat
);
platformR.position.set(0.8, 0.6 + 0.1, 0);
platformR.castShadow = true;
caseGroup.add(platformR);

// INNER LENS AREAS
const innerMat = new THREE.MeshPhongMaterial({ color: 0x9bc4e0, shininess: 30 });
const innerL = new THREE.Mesh(
    new THREE.CylinderGeometry(0.5, 0.5, 0.08, 32),
    innerMat
);
innerL.position.set(-0.8, 0.6 + 0.2 - 0.04, 0);
caseGroup.add(innerL);

const innerR = new THREE.Mesh(
    new THREE.CylinderGeometry(0.5, 0.5, 0.08, 32),
    innerMat
);
innerR.position.set(0.8, 0.6 + 0.2 - 0.04, 0);
caseGroup.add(innerR);

// LABELS
function createLabel(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, 256, 256);
    
    // Circle background
    ctx.beginPath();
    ctx.arc(128, 128, 110, 0, Math.PI * 2);
    ctx.fillStyle = '#2c3e50';
    ctx.fill();
    ctx.lineWidth = 5;
    ctx.strokeStyle = '#5d8aaf';
    ctx.stroke();
    
    // Text
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 134);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const mesh = new THREE.Mesh(new THREE.CircleGeometry(0.55, 32), material);
    mesh.rotation.x = -Math.PI / 2;
    return mesh;
}

const labelL = createLabel('L');
labelL.position.set(-0.8, 0.6 + 0.2 + 0.01, 0);
caseGroup.add(labelL);

const labelR = createLabel('R');
labelR.position.set(0.8, 0.6 + 0.2 + 0.01, 0);
caseGroup.add(labelR);

// LID
const lidGroup = new THREE.Group();
const hingeY = 0.6 + 0.2 + 0.15; // top of platform + some offset = 0.95... 
```

Hmm, wait. The platforms are at y from 0.6 to 0.8 (center 0.7, height 0.2). The lids should cover the platforms. So the hinge should be at about y=0.8 (the top of the platforms) or slightly above.

Actually, the lid bottom should seal against the top of the platforms (y=0.8). So:
- Hinge at y=0.8
- Lid bottom at y=0.8
- Lid center at y=0.8 + 0.15 = 0.95 (if height is 0.3)

In local coordinates (relative to hinge at (0, 0.8, -1.8)):
- Lid center: (0, 0.15, 1.8)

OK wait, but the labels are at y=0.8+0.01=0.81. And the lid bottom is at y=0.8. So the labels would be inside the lid! That's fine — when the lid is closed, the labels are hidden. When the lid is open, you see them. That's actually the correct behavior for a contact lens case!

But wait, the labels would actually be INSIDE the closed lid, so they wouldn't be visible. For a real contact lens case, the L and R labels are on top of the compartments, visible when the lid is open. That's actually what we want!

But the user said "眼镜盒应有醒目的 L 和 R 标识" — the case should have prominent L and R labels. This could mean they should be visible even when the lid is closed. Or at least, visible when the lid is open.

Hmm, if the lid covers the labels when closed, that's not ideal. Let me put the labels on the front/side of the case body instead, or on the compartments in a way that they're visible even when the lid is closed.

Actually, let me reconsider. Contact lens cases typically don't have a full cover. They have two separate screw caps. So the L and R are on the caps themselves.

But the user asked for a single "盒盖" (lid). Let me keep the flip-top design but put the labels somewhere visible when the lid is closed.

Options:
1. Put labels on the front of the case body
2. Put labels on top of the lid (but then they'd be upside down when closed)
3. Put labels on the lids of the individual compartments

Let me go with option 1: L and R on the front of the case body, on small label plates.

Or better: put L and R labels on small plaques on the front side of the case body, visible at all times.

Actually, you know what? Let me put the labels on the compartments (platforms), AND also add small labels on the front of the case body. That way, they're visible both when the lid is open and closed.

Or simpler: put the labels on the top surface of the platforms but slightly raised, so even when the lid is closed, you can see them if you look from the front at an angle... no, the lid covers everything from y=0.8 up.

Let me go with putting the L and R labels on the front face of the case body:

```javascript
// Label plaques on the front of the case body
function createLabelPlaque(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 128;
    const ctx = canvas.getContext('2d');
    
    ctx.fillStyle = '#2c3e50';
    ctx.fillRect(0, 0, 256, 128);
    
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 80px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 68);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const mesh = new THREE.Mesh(new THREE.PlaneGeometry(0.5, 0.25), material);
    return mesh;
}

const plaqueL = createLabelPlaque('L');
plaqueL.position.set(-0.55, 0.3, 1.79); // front surface of body
plaqueL.rotation.y = Math.PI; // face outward from body
caseGroup.add(plaqueL);

const plaqueR = createLabelPlaque('R');
plaqueR.position.set(0.55, 0.3, 1.79);
plaqueR.rotation.y = Math.PI;
caseGroup.add(plaqueR);
```

Hmm, but the body is a cylinder at origin with radius 1.8 at top and 1.6 at bottom. At y=0.3, the radius would be between 1.6 and 1.8. Let me calculate.

The body cylinder goes from y=-0.6 (radius 1.6) to y=0.6 (radius 1.8). At y=0.3:
- t = (0.3 - (-0.6)) / (0.6 - (-0.6)) = 0.9 / 1.2 = 0.75
- radius = 1.6 + 0.75 * (1.8 - 1.6) = 1.6 + 0.75 * 0.2 = 1.6 + 0.15 = 1.75

So the surface at y=0.3 is at distance 1.75 from the Y axis. The label plaque should be at (0.55, 0.3, 1.75) and facing outward (radially).

Actually, the label plaques should face outward from the cylinder surface. The normal at point (0.55, 0.3, 1.75) on a cylinder centered at origin points in the direction (0.55, 0, 1.75). Normalized: (0.55, 0, 1.75) / |...|.

This is getting complicated. Let me simplify further: put the labels on the top surface of the platforms (which are visible when the lid is open) and also add a small L/R on the lid itself (visible when closed).

OR, even simpler: put the labels on the platforms and make them visible through a transparent lid? No, that's weird.

Let me try a different approach. Let me make the case WITHOUT a full cover. Instead, each compartment has its own small cap. But the user said "盒盖" which could mean the case has a lid.

You know, let me go with the flip-top lid design, and the L and R labels are on the platforms (inside the case). When the lid is open, you see the L and R labels prominently. When closed, the lid shows a subtle L/R embossed on top.

But actually, real contact lens cases often have the L and R on the outside of the case body, near the compartments. So let me put them on the front exterior of the body, like small engraved plates.

Let me put the labels on the body surface as small circular badges:

```javascript
function createCircularLabel(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.beginPath();
    ctx.arc(128, 128, 110, 0, Math.PI * 2);
    ctx.fillStyle = '#ffffff';
    ctx.fill();
    ctx.lineWidth = 6;
    ctx.strokeStyle = '#2c3e50';
    ctx.stroke();
    
    ctx.fillStyle = '#2c3e50';
    ctx.font = 'bold 130px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 134);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const mesh = new THREE.Mesh(new THREE.CircleGeometry(0.35, 32), material);
    return mesh;
}

// On the front of the body
const badgeL = createCircularLabel('L');
badgeL.position.set(-0.5, 0.3, 1.76); // near front surface
badgeL.lookAt(badgeL.position.clone().multiplyScalar(2)); // face outward
caseGroup.add(badgeL);
```

Using lookAt to make it face outward from the cylinder. But the label needs to face away from the cylinder center.

Actually, let me just place them on a flat position and rotate appropriately:

```javascript
const badgeL = createCircularLabel('L');
badgeL.position.set(-0.5, 0.3, 1.76);
badgeL.rotation.y = 0; // Facing +Z, which is outward since position has positive Z
caseGroup.add(badgeL);
```

Since the badge is a circular plane, it faces +Z by default (with normal along +Z). The body surface at z=1.76, x=-0.5 is facing roughly +Z, so this should work.

OK I'm overthinking this. Let me just write the code with the labels on the platforms (visible when lid is open) and ALSO add badges on the body front. Let me make it work.

Actually, let me simplify once and for all. Here's my final decision:

**Design**: 
- The case body has two platform circles on top
- Labels L and R are on these platforms (as canvas texture circles)
- The lid covers the platforms when closed
- Additionally, the front of the lid has L/R badges? No, let me put them on the body front below the lid.

Actually, let me just put the L and R labels on the platforms and the front of the body. Let me NOT overcomplicate.

FINAL DESIGN:
- Compact case body with two compartment platforms on top
- L/R labels on the platforms, visible when lid is open
- Also helpful: the labels are prominent (big text on contrasting background)
- Flip-top lid that opens backward

That's it. Let me write the code.

Here goes:

```javascript
// ... setup, scene, camera, renderer, lights, controls

// CASE GROUP
const caseGroup = new THREE.Group();
scene.add(caseGroup);

// BODY
const bodyGeo = new THREE.CylinderGeometry(1.8, 1.6, 1.2, 48);
const bodyMat = new THREE.MeshPhongMaterial({ 
    color: 0x4a9eda, specular: 0x222222, shininess: 40 
});
const bodyMesh = new THREE.Mesh(bodyGeo, bodyMat);
bodyMesh.position.y = 0;
bodyMesh.castShadow = true;
bodyMesh.receiveShadow = true;
caseGroup.add(bodyMesh);

// TOP SURFACE
const topSurface = new THREE.Mesh(
    new THREE.CylinderGeometry(1.78, 1.78, 0.05, 48),
    new THREE.MeshPhongMaterial({ color: 0x6bb5ef, shininess: 30 })
);
topSurface.position.y = 0.6 + 0.025;
caseGroup.add(topSurface);

// COMPARTMENTS
const compMat = new THREE.MeshPhongMaterial({ color: 0xd0e8f8, shininess: 40 });
const compGeo = new THREE.CylinderGeometry(0.7, 0.7, 0.2, 32);

const compL = new THREE.Mesh(compGeo, compMat);
compL.position.set(-0.8, 0.7, 0);
compL.castShadow = true;
caseGroup.add(compL);

const compR = new THREE.Mesh(compGeo, compMat);
compR.position.set(0.8, 0.7, 0);
compR.castShadow = true;
caseGroup.add(compR);

// INNER LENS AREAS
const lensMat = new THREE.MeshPhongMaterial({ color: 0x88c8e8, shininess: 50 });
const lensGeo = new THREE.CylinderGeometry(0.5, 0.5, 0.06, 32);

const lensL = new THREE.Mesh(lensGeo, lensMat);
lensL.position.set(-0.8, 0.75, 0);
caseGroup.add(lensL);

const lensR = new THREE.Mesh(lensGeo, lensMat);
lensR.position.set(0.8, 0.75, 0);
caseGroup.add(lensR);

// LABELS (on platforms, above lens area)
function createLabel(text) {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, 256, 256);
    ctx.beginPath();
    ctx.arc(128, 128, 108, 0, Math.PI * 2);
    ctx.fillStyle = '#2c3e50';
    ctx.fill();
    ctx.lineWidth = 4;
    ctx.strokeStyle = '#5d8aaf';
    ctx.stroke();
    
    ctx.fillStyle = '#ffd700'; // Gold text for visibility
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, 128, 134);
    
    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
    const mesh = new THREE.Mesh(new THREE.CircleGeometry(0.45, 32), material);
    mesh.rotation.x = -Math.PI / 2;
    return mesh;
}

const labelL = createLabel('L');
labelL.position.set(-0.8, 0.78, 0); // Just above the lens area
caseGroup.add(labelL);

const labelR = createLabel('R');
labelR.position.set(0.8, 0.78, 0);
caseGroup.add(labelR);
```

Wait, the labels need to be above the lens area. The lens is at y=0.75 (center, height 0.06). Top of lens at 0.78. So the label at 0.78+0.01=0.79 would be just above the lens.

Hmm, but the lens itself is a protrusion. Actually, the lens area should be flush with the platform top. Let me reconsider.

Platform height is 0.2, so top of platform at y=0.6+0.2=0.8.

The lens area should be a small circle ON TOP of the platform, not a cylinder sticking up. Let me use a flat disc:

```javascript
// Lens area (flat disc on platform)
const lensDiscMat = new THREE.MeshPhongMaterial({ color: 0x88c8e8, shininess: 50 });
const lensDiscL = new THREE.Mesh(
    new THREE.CircleGeometry(0.5, 32),
    lensDiscMat
);
lensDiscL.position.set(-0.8, 0.801, 0);
lensDiscL.rotation.x = -Math.PI / 2;
caseGroup.add(lensDiscL);

const lensDiscR = new THREE.Mesh(
    new THREE.CircleGeometry(0.5, 32),
    lensDiscMat
);
lensDiscR.position.set(0.8, 0.801, 0);
lensDiscR.rotation.x = -Math.PI / 2;
caseGroup.add(lensDiscR);
```

And the labels are above the lens discs:
```javascript
labelL.position.set(-0.8, 0.802, 0);
labelR.position.set(0.8, 0.802, 0);
```

Wait, but the label is a CircleGeometry too, rotated to face up. If it's at 0.802 and the lens disc is at 0.801, there's a z-fighting risk? No, they're at different y values with 0.001 difference, which should be fine.

Actually, let me make the labels with some thickness: use a thin cylinder and put the texture on top.

Or simpler: use two overlapping circles at slightly different heights:
- Lens disc at y=0.801
- Label at y=0.803

OK, this should work.

Now, the LID:

```javascript
// Hinge position
const hingeY = 0.8; // top of platforms
const hingeZ = -1.8; // back edge of body

// Lid group
const lidGroup = new THREE.Group();
lidGroup.position.set(0, hingeY, hingeZ);
scene.add(lidGroup);

// Lid material
const lidMat = new THREE.MeshPhongMaterial({ 
    color: 0x2c3e50, 
    specular: 0x333333, 
    shininess: 50 
});

// Lid mesh
const lidMesh = new THREE.Mesh(
    new THREE.CylinderGeometry(1.85, 1.85, 0.3, 48),
    lidMat
);
lidMesh.position.set(0, 0.15, 1.8); // relative to hinge
lidMesh.castShadow = true;
lidGroup.add(lidMesh);

// Lid inner surface
const lidInnerMat = new THREE.MeshPhongMaterial({ 
    color: 0x3d5a80, 
    side: THREE.DoubleSide 
});
const lidInner = new THREE.Mesh(
    new THREE.CircleGeometry(1.75, 48),
    lidInnerMat
);
lidInner.position.set(0, 0.01, 1.8);
lidInner.rotation.x = Math.PI / 2;
lidGroup.add(lidInner);

// Lid top decorative disc
const lidTopDeco = new THREE.Mesh(
    new THREE.CylinderGeometry(1.7, 1.7, 0.05, 48),
    new THREE.MeshPhongMaterial({ color: 0x3d5a80, shininess: 30 })
);
lidTopDeco.position.set(0, 0.3, 1.8);
lidGroup.add(lidTopDeco);

// Front handle/tab on lid
const tabMat = new THREE.MeshPhongMaterial({ color: 0x1a2530 });
const tab = new THREE.Mesh(
    new THREE.BoxGeometry(0.4, 0.15, 0.2),
    tabMat
);
tab.position.set(0, 0.05, 3.65); // front edge of lid
lidGroup.add(tab);
```

Wait, the lid's front edge at local z = 1.8 + 1.85 = 3.65. The tab should be at the front edge, sticking out. Let me put it at z = 3.65+0.1 = 3.75 (with depth 0.2, so from 3.65 to 3.85).

And in Y: the lid bottom is at y=0, the lid top at y=0.3. The tab at y=0.05 means it's near the bottom of the lid. That's where you'd grip it.

Now, for the click detection and animation:

```javascript
// Raycasting
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const clickableObjects = [lidMesh, lidInner, lidTopDeco, tab];

// Click handling
let isOpen = false;
let currentAngle = 0;
let targetAngle = 0;
const OPEN_ANGLE = -1.2; // ~ -69°

// Handle click & drag distinction
let pointerDownPos = null;

renderer.domElement.addEventListener('pointerdown', (e) => {
    pointerDownPos = { x: e.clientX, y: e.clientY };
});

renderer.domElement.addEventListener('pointerup', (e) => {
    if (!pointerDownPos) return;
    const dx = e.clientX - pointerDownPos.x;
    const dy = e.clientY - pointerDownPos.y;
    if (Math.sqrt(dx*dx + dy*dy) < 5) {
        // It's a click, not a drag
        handleClick(e);
    }
    pointerDownPos = null;
});

function handleClick(event) {
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(clickableObjects);
    
    if (intersects.length > 0) {
        toggleLid();
    }
}

function toggleLid() {
    isOpen = !isOpen;
    targetAngle = isOpen ? OPEN_ANGLE : 0;
    updateStatus();
}

function updateStatus() {
    const status = document.getElementById('status');
    status.textContent = isOpen ? '✓ 已打开' : '✓ 已关闭';
    status.style.opacity = 1;
    setTimeout(() => {
        status.style.opacity = 0;
    }, 1500);
}

// Hover effect
renderer.domElement.addEventListener('mousemove', (e) => {
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
    
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(clickableObjects);
    
    if (intersects.length > 0) {
        document.body.style.cursor = 'pointer';
        if (!isHovering) {
            isHovering = true;
            lidMat.emissive.setHex(0x112233);
        }
    } else {
        document.body.style.cursor = 'default';
        if (isHovering) {
            isHovering = false;
            lidMat.emissive.setHex(0x000000);
        }
    }
});

// Animation loop
const clock = new THREE.Clock();

function animate() {
    requestAnimationFrame(animate);
    
    const delta = clock.getDelta();
    
    // Smooth interpolate lid angle
    const diff = targetAngle - currentAngle;
    if (Math.abs(diff) > 0.01) {
        const lerpFactor = 1 - Math.pow(0.001, delta);
        currentAngle += diff * lerpFactor;
    } else {
        currentAngle = targetAngle;
    }
    
    lidGroup.rotation.x = currentAngle;
    
    controls.update();
    renderer.render(scene, camera);
}

animate();

// Resize handler
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
```

I also need to make sure the lid's inner surface (the flat circle) rotates with the lid group. Since it's a child of lidGroup, it will.

But wait, there's an issue. The `lidInner` mesh is a circle with `rotation.x = Math.PI/2`, so it's facing up/down. When the lid rotates open, the inner surface will face the viewer, which is good.

However, I need to double-check the orientation. The circle geometry by default faces +Z. With `rotation.x = Math.PI/2`, it faces +Y (up). So in the closed position, it faces up — that's the underside of the lid you see when you look down at the closed case. When the lid opens, it rotates backward, so the inner surface faces... let me think.

Actually, looking at it from above, the inner circle would be visible as the bottom of the lid. When the lid opens (rotates back around X), the inner circle would now be facing somewhat toward the viewer (upward and toward the front). That's what you want when you open the lid and see the inside.

Wait, I want the inner surface to face DOWN when closed and UP when open. Since the lid rotates around X by -1.2 rad, the circle (which faces up when closed, due to rotation.x = π/2) would after rotation... 

The circle at position (0, 0.01, 1.8) with rotation.x = π/2 faces +Y (up). When the lidGroup rotates by -1.2 rad around X, the circle's world rotation changes. The normal vector (0, 1, 0) in local space, after rotating by -1.2 rad around X:

The rotation matrix for X rotation by angle θ:
[1, 0, 0]
[0, cos(θ), -sin(θ)]
[0, sin(θ), cos(θ)]

For θ = -1.2:
cos(-1.2) ≈ 0.362
sin(-1.2) ≈ -0.932

So normal (0, 1, 0) becomes:
x' = 0
y' = 0*0.362 - 0*(-0.932) = 0
z' = 0*(-0.932) + 0*0.362 = 0

Wait, that gives (0, 0.362, -0.932)? No, let me recalculate.

The matrix multiplication:
[1, 0, 0]   [0]   [0]
[0, cos, -sin] [1] = [cos]
[0, sin, cos]  [0]   [sin]

For θ = -1.2:
cos(-1.2) = cos(1.2) ≈ 0.362
sin(-1.2) = -sin(1.2) ≈ -0.932

So normal becomes:
x' = 0
y' = 0.362
z' = -0.932

So the normal is (0, 0.362, -0.932), which points upward and backward. That's the inside of the lid, facing the viewer when the lid is open. But it's a bit downward too.

Actually, when you open a flip-top lid, you see the inside of the lid from above. The normal should point roughly upward. With (0, 0.362, -0.932), it's pointing mostly backward (negative z) with a bit upward. Hmm, it depends on the angle.

Actually, I want the inner surface visible from above when the lid is open. The normal (0, 0.362, -0.932) has a positive y component, so it IS visible from above. And the backward Z component makes sense because the lid is tilted back.

OK, this should work.

But wait, there's another issue. The circle geometry was rotated with `rotation.x = Math.PI / 2`, which is applied in local space. When added to the lidGroup, the circle's position and rotation are in the lidGroup's local space.

In the lidGroup's local space:
- The circle is at (0, 0.01, 1.8) with rotation.x = π/2. 
- Its normal in local space is (0, 1, 0) (pointing up).

When the lidGroup rotates by -1.2 rad around X, the circle's world-space normal becomes as calculated above: (0, 0.362, -0.932).

Hmm, but actually the circle's rotation is relative to the lidGroup. The lidGroup's rotation is around X. The circle's own rotation (π/2 around X) combined with the group's rotation (-1.2 rad around X) gives a total rotation of (π/2 - 1.2) around X.

Total rotation: π/2 - 1.2 = 0.371 rad. So the circle's normal (originally +Z for CircleGeometry) becomes:
- After total rotation around X by 0.371:
  - y' = sin(0.371) ≈ 0.362
  - z' = cos(0.371) ≈ 0.932

So the normal is (0, 0.362, 0.932)? Hmm, that points up and FORWARD (positive z). That doesn't seem right for an open lid.

Wait, I need to be more careful. The CircleGeometry's normal is +Z. When I set `rotation.x = π/2`, the normal rotates to +Y (up). Then when the lidGroup rotates by -1.2 around X, the normal rotates further.

Actually, rotation is applied in a specific order. The object's rotation (π/2 around X) is applied first in local space, then the group's rotation.

So:
1. CircleGeometry normal: (0, 0, 1)
2. After mesh rotation.x = π/2: normal becomes (0, -1, 0)? 

Wait, rotation around X by π/2:
[1, 0, 0]
[0, cos(π/2), -sin(π/2)] = [0, 0, -1]
[0, sin(π/2), cos(π/2)]   [0, 1, 0]

So normal (0, 0, 1) → (0*1, 0*0 + 0*0 + 1*(-1), 0*0 + 0*1 + 1*0) = (0, -1, 0)

Hmm, that points DOWN? That can't be right for a surface facing up.

Wait, no. The rotation matrix:
```
Rotation around X by angle θ:
| 1    0       0     |
| 0  cos(θ) -sin(θ) |
| 0  sin(θ)  cos(θ) |
```

For θ = π/2:
```
| 1  0  0 |
| 0  0 -1 |
| 0  1  0 |
```

Normal (0, 0, 1):
x' = 1*0 + 0*0 + 0*1 = 0
y' = 0*0 + 0*0 + (-1)*1 = -1
z' = 0*0 + 1*0 + 0*1 = 0

So normal is (0, -1, 0) — pointing DOWN. That means the circle's visible side faces downward after rotation.x = π/2.

But I want the visible side to face UP. So I should use `rotation.x = -π/2`:

For θ = -π/2:
cos(-π/2) = 0, sin(-π/2) = -1
```
| 1  0  0 |
| 0  0  1 |
| 0 -1  0 |
```

Normal (0, 0, 1) → (0, 1, 0) — pointing UP. ✓

So I should use `rotation.x = -Math.PI / 2` for the inner surface and labels.

Wait, but for labels, I was using `rotation.x = -Math.PI / 2` already in my earlier code! Good.

For the lidInner surface, I should also use `rotation.x = -Math.PI / 2`.

OK let me correct that.

Now, let me also reconsider: should the lidInner face up (toward the lid interior) or down (toward the compartments)? 

When you open a real contact lens case lid, the inside of the lid is facing down (toward you when you look down at it). Hmm, actually, the inside of the lid faces the compartments when closed. When open, it faces away from the case.

So the lidInner should have its visible side facing DOWN when closed. With `rotation.x = -Math.PI / 2`, it faces UP. That's correct for when the lid is OPEN — you look up at the underside of the lid and see the inner surface.

Wait, no. When the lid is closed (rotation.x = 0), the lidInner faces up (normal +Y). When the lid opens (rotation.x = -1.2), the normal becomes:
Using the same calculation as before, but starting with (0, 1, 0):
After rotation by -1.2 around X:
(0, cos(-1.2), sin(-1.2)) = (0, 0.362, -0.932)

This points up and backward. That means when the lid is open, the inner surface is facing up and toward the back. That's correct — you can see the inner surface from above.

When closed (rotation = 0), the normal is (0, 1, 0) — pointing up. That's also fine; it's just the underside of the lid.

Actually, wait. If the normal is (0, 1, 0) when closed, then the surface is visible from above. But it's INSIDE the lid, so the outside of the lid is above it. The inner surface is not really visible when closed, because the lid's cylinder geometry covers it. Actually, the CircleGeometry is at y=0.01 (local), which is at the bottom of the lid. The cylinder lid is from y=0 to y=0.3. So the Circle at y=0.01 is essentially at the bottom of the lid, and it faces up. Looking from above, you'd see the cylinder's top (y=0.3) first. Looking from below, you'd see the Circle at y=0.01.

When the lid is closed, the Circle faces down toward the compartments? No, it faces UP. So from below (inside the case), you'd see the BACK of the Circle, which is not visible because MeshBasicMaterial is not DoubleSide.

OK, let me use DoubleSide for the inner surface material so it's visible from both sides:

```javascript
const lidInnerMat = new THREE.MeshPhongMaterial({ 
    color: 0x3d5a80, 
    side: THREE.DoubleSide 
});
```

That way, it doesn't matter which way it faces. Good.

OK, I think I have a solid plan now. Let me write the complete code, being careful with all the details.

One last thing: I need to make sure the lid inner circle doesn't z-fight with the lid cylinder. The lid cylinder goes from y=0 to y=0.3 in local space. The lidInner circle is at y=0.01. That's inside the cylinder, so no z-fighting. The circle is like a thin membrane near the bottom of the lid.

Actually, let me put the inner circle at y=0.01, which is just above the bottom. The lid cylinder's bottom at y=0. When you look at the lid from the open position, you see the inner surface at y=0.01 from above. The lid cylinder's interior walls are visible from the sides. This should look like a hollow lid.

Actually, to make it look like a proper hollow lid, I should also add the inner walls of the lid. But that's complex. Let me just use the single circle as the inner surface. It'll look fine.

OK let me write the full code now. I'll be comprehensive.

Wait, let me reconsider one more thing: the label textures. I was using `MeshBasicMaterial` with a canvas texture. The label circle geometry with rotation.x = -π/2 will face UP. For the label to be visible from above, I need the texture to be on the +Y side.

With `rotation.x = -Math.PI / 2`:
- The CircleGeometry's front face (with the texture) faces +Y (up). ✓

That's correct.

Now, let me also add a subtle environment/scene for aesthetics:
- A gradient background
- A reflective floor
- Maybe a soft shadow

I already have plans for that. Let me write the code.

Actually, let me use a nicer approach for the body. Instead of a simple cylinder, let me use a slightly tapered cylinder with some detail rings and a nice color scheme.

Let me also make the closed lid fit nicely with the body:
- The body radius at top is 1.8
- The lid radius is 1.85, so it slightly overhangs

OK, writing the complete code now:

Let me also think about the compartment labels one more time. I want "L" and "R" to be prominent. I'll use:
- Large text (140px on a 256px canvas)
- Gold color text (#ffd700) on dark blue background (#2c3e50)
- Clearly visible when the lid is open

Also, let me add small L/R labels on the front of the lid or body so users know which compartment is which even when the lid is closed.

Actually... the user said "眼镜盒应有醒目的 L 和 R 标识" — the case should have prominent L and R identifiers. This could be interpreted as:
1. The compartments themselves should have L/R
2. The overall case should show L/R

I think putting the L/R on the top surfaces of the compartments (visible when lid is open) is sufficient, but to be safe, let me also add them to the front of the body as small badges.

OK, final plan:
- L/R labels on compartment platforms (large, gold on dark)
- L/R badges on the front of the body (small, visible when lid is closed)

This covers both interpretations.

Now let me code it. I need to be efficient.

Actually wait, I don't want to make this too complex. Let me stick with one approach: L/R on the compartments, which are the main labels. And when the lid is open, they're clearly visible. The user can open the lid to see them.

But the user said the labels should be "醒目" (prominent). If the lid is closed, you can't see them. So let me put them somewhere visible even when closed.

Compromise: Put the L/R labels on the OUTSIDE of the lid, at the front. When the lid is closed, the user sees L and R on the lid. When opened, they also see them on the compartments.

Actually, let me just put them on the bottom of the case body's front face. That's always visible regardless of the lid position.

Or, better: put them on the body top surface, around the compartments, but still within the lid's footprint? No, they'd be hidden.

OK, simplest solution: Put the L and R labels on the front exterior of the case body, visible at all times. And ALSO on the compartments for when the lid is open.

Let me implement both.

Actually, I'll put the L and R on the front face of the case body as part of the design, and on the compartment platforms as well.

Let me code this now.

For the front badges, I'll create small circular labels and position them on the body surface at the front.

Let me position them:
- Body radius at height y is between 1.6 and 1.8
- At y=0.2: radius ≈ 1.67 (linearly interpolated)
- Front point: (0, 0.2, radius) — but I want them on the sides, not the front center
- Position L at (-0.4, 0.2, 1.65) and R at (0.4, 0.2, 1.65)

These should be visible on the front of the body, below the lid.

OK, let me write the complete code now. I'll stop overplanning and just do it.

Wait, one more thing. The user wants "运行方式" (how to run) and "测试说明" (testing instructions). I'll include those in a comment block at the top of the file, or as HTML comments.

Let me write the code now.

Actually, I realize I should think about whether the lid position makes it hard to see the labels on the compartments when the lid is open. When the lid opens backward, it might block the view from certain angles. But with OrbitControls, the user can rotate around.

OK, here's my final plan for the labels:
1. Large L/R on top of each compartment platform (these are the primary labels)
2. No additional labels on the body front (keep it simple)

If the user wants them visible when the lid is closed, they can open the lid — and the interaction of opening the lid reveals the labels. That's actually a nice user experience.

But "醒目" means prominent... The user might expect to see the L/R immediately. Let me add them to the front of the lid! As embossed or printed text.

Hmm, but the lid covers both compartments. If I put L and R on the lid, it wouldn't match which side is which.

OK, final decision: Put L and R on the front of the case body, below the lid, as small circular badges. These are always visible. And also on the compartment platforms for when the lid is open.

I'll put them on the front curve of the body at positions:
- L: (-0.5, 0.1, 1.65) 
- R: (0.5, 0.1, 1.65)

Where 1.65 is roughly the radius at y=0.1.

Actually, let me calculate the radius at y=0.1:
t = (0.1 - (-0.6)) / (0.6 - (-0.6)) = 0.7 / 1.2 = 0.583
radius = 1.6 + 0.583 * 0.2 = 1.6 + 0.117 = 1.717

So at y=0.1, radius is about 1.72.

Let me put the badges at:
- L: (-0.5, 0.1, 1.72)
- R: (0.5, 0.1, 1.72)

And rotate them to face outward (away from the cylinder center).

Actually, for a cylinder, the outward normal at point (x, y, z) is (x, 0, z) normalized. So at (-0.5, 0.1, 1.72), the normal is (-0.5, 0, 1.72) normalized = (-0.279, 0, 0.960).

So the badge should face that direction. I can use lookAt:
```javascript
badgeL.position.set(-0.5, 0.1, 1.72);
badgeL.lookAt(-0.5 + (-0.279), 0.1, 1.72 + 0.960);
```

But CircleGeometry faces +Z by default, so lookAt would work.

Or I can just rotate it:
```javascript
const angle = Math.atan2(-0.5, 1.72); // angle from +Z axis in XZ plane
badgeL.rotation.y = -angle; // hmm, let me not do this manually
```

Simpler: just use lookAt:
```javascript
badgeL.lookAt(-0.5 * 2, 0.1, 1.72 * 2); // look away from center
```

Actually, `lookAt` points the object's +Z toward the target. For CircleGeometry which faces +Z, this should work.

But wait, the badge also has a texture. The texture orientation matters. If I use lookAt, the badge will face outward but the text might be rotated. Let me adjust.

Actually, the text is symmetric (just "L" or "R"), so orientation doesn't matter much. Let me just use lookAt and not worry about texture orientation.

OK, enough planning. Let me write the code!

Let me also think about the overall aesthetic:
- Background: dark gradient
- The case body is a nice blue
- The lid is dark navy
- Gold accents for labels
- Soft lighting and shadows

Here's the complete code:

Actually, let me think through a potential issue with the opening lid. When the lid opens backward (rotation around X by -1.2), it swings up and back. But since the hinge is at z=-1.8, the lid's center of mass will swing up and backward. At 1.2 radians, the lid should be at about 69 degrees from horizontal, which is a nice open position.

Let me verify once more:
hinge: local origin → lid center: (0, 0.15, 1.8) in local space.

After rotation by -1.2 around X:
y' = 0.15*cos(1.2) - 1.8*sin(1.2) ≈ 0.15*0.362 - 1.8*(-0.932) = 0.054 + 1.678 = 1.732
z' = 0.15*sin(1.2) + 1.8*cos(1.2) ≈ 0.15*(-0.932) + 1.8*0.362 = -0.140 + 0.652 = 0.512

Wait, let me recalculate with the right signs. For rotation by -1.2 around X:

The rotation matrix for angle θ = -1.2:
```
R_x(θ) = | 1    0       0     |
          | 0  cos(θ) -sin(θ) |
          | 0  sin(θ)  cos(θ) |
```

cos(-1.2) = cos(1.2) ≈ 0.362
sin(-1.2) = -sin(1.2) ≈ -0.932

So:
```
R_x(-1.2) = | 1  0      0    |
            | 0  0.362  0.932|
            | 0 -0.932  0.362|
```

Applying to local point (0, 0.15, 1.8):
x' = 0
y' = 0.362*0.15 + 0.932*1.8 = 0.0543 + 1.6776 = 1.7319
z' = (-0.932)*0.15 + 0.362*1.8 = -0.1398 + 0.6516 = 0.5118

So the lid center in world space (relative to hinge at (0, 0.8, -1.8)):
(0, 0.8 + 1.732, -1.8 + 0.512) = (0, 2.532, -1.288)

So the lid is at (0, 2.53, -1.29), which is up and slightly backward from the case. That looks right for an opened lid.

The lid also extends 1.85 in radius from its center. So its front edge would be at approximately:
(0, 2.53, -1.29 + 1.85) = (0, 2.53, 0.56)

That's above the front of the case but not overlapping it. Seems good.

OK, let me now write the complete code! I need to write it carefully.

Let me also handle an edge case: when the user clicks on the lid while the OrbitControls are being dragged. My pointer down/up distance approach should handle this.

Also, I should make sure the OrbitControls don't interfere with the click detection on the lid. The controls use pointer events, and my click handler also uses pointer events. There might be conflicts.

Actually, OrbitControls use mouse events internally. Let me use `addEventListener` on the renderer's DOM element, and I should be fine. The key is distinguishing click from drag, which I'm already doing with the distance check.

One more thing: I'll add a highlight or outline to the lid when hovering.

OK, I'll use an emissive color change for hover.

Let me finalize the code now.

Actually, I realize there might be an issue with using both OrbitControls and custom click events. The OrbitControls' `change` event fires when the camera moves. And my HTML element's click event might fire after OrbitControls is done.

The standard practice is:
1. Use `pointerdown` and `pointerup` events to detect a click (pointer down then up without much movement)
2. OrbitControls also listen to these events, but they need movement to rotate

So the distance check should work: if the user clicks without dragging, it's a click. If they drag (rotate camera), the distance check fails and no click fires.

Let me also prevent the click from registering as part of a drag:

```javascript
let wasDragging = false;

renderer.domElement.addEventListener('pointerdown', (e) => {
    pointerDownPos = { x: e.clientX, y: e.clientY };
    wasDragging = false;
});

renderer.domElement.addEventListener('pointermove', (e) => {
    if (pointerDownPos) {
        const dx = e.clientX - pointerDownPos.x;
        const dy = e.clientY - pointerDownPos.y;
        if (Math.sqrt(dx*dx + dy*dy) > 5) {
            wasDragging = true;
        }
    }
});

renderer.domElement.addEventListener('pointerup', (e) => {
    if (pointerDownPos && !wasDragging) {
        handleClick(e);
    }
    pointerDownPos = null;
});
```

OK, good.

Now, let me write the complete code. I'll be thorough.

Here's the plan for the complete file:

1. HTML + CSS
2. Three.js imports (CDN)
3. Scene, camera, renderer setup
4. Lights
5. Ground
6. Case body (cylinder + details)
7. Compartment platforms
8. Lens areas
9. L/R labels (on platforms)
10. L/R badges (on body front)
11. Lid group with hinge
12. Lid cylinder, inner surface, top decoration, tab
13. Raycasting and click detection
14. Hover effect
15. Animation loop
16. Resize handler

Let me write it!