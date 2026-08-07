The user is asking me to search for and read "Aurora" — likely Kim Stanley Robinson's 2015 novel about a generation starship — but I don't have search capabilities available in this conversation, so I'll need to be upfront about that limitation while offering what I can recall about the book. The ship generates about 1 g of spin gravity in the biomes, and Freya lives in the Nova Scotia biome on Ring B. The biomes are named after Earth biogeographic regions—I'm confident about Costa Rica, Amazonia, Patagonia, Labrador, Nova Scotia, Iran, Siberia, Mongolia, The Steppes, Yangtze, Bengal, Tasmania, Olympia, Sonora, and Pampas, though I'm less certain about some of the others. The ship was launched from Sa—

actually, it departed from Saturn's system in a transfer orbit, heading toward Tau Ceti about 11.9 light years away with a journey time of roughly 170 years at 0.1c. The population is around 2000 people, distributed across two rings with 12 biomes each, separated by locks and walls to maintain distinct ecologies. A central spine runs through both rings, housing the fuel tanks, fabrication areas, docks, ferries, and the quantum computer that serves as the ship's AI narrator. The ship decelerated using a magnetic or laser beam system—

I'm trying to recall the exact mechanics from the novel, whether it was a beam from the Solar System pushing on it initially, then fusion drive plus magnetic scoop for deceleration, or if the return journey used laser beam deceleration followed by aerobraking through Earth's atmosphere.

The ship itself is a massive rotating structure with a protective shield against interstellar dust, featuring multiple biomes with spin gravity that generates 1g, complete with lakes, rivers, forests, and artificial lighting along the central axis. From what I remember, it's a 24-biome, two-ring design with roughly 4 square kilometers of habitable land and a total mass around 22 million tonnes, though I'm not entirely certain of those figures.

I should note upfront that I can't verify these details against the source material right now, so I'll work from my recollection of the Aurora design and flag where I'm uncertain—like whether the rings counter-rotate or spin in the same direction. Since I don't have access to tools or the ability to write files, I'll present the model code directly inline.

I'm building a complete, self-contained HTML file using Three.js from a CDN with OrbitControls for camera movement. The scene will include a deep space backdrop with stars, nebulae, and Tau Ceti visible in the distance, plus motion blur effects. The main feature is toggling between an exterior view of the rings and an interior biome mode where you can explore the curved ground, terrain, vegetation, water features, and the spine lamp structure. I'll add UI controls for selecting different biomes, adjusting zoom, and labeling key components.

Writing the Three.js scene... I need to set up an importmap to handle Three.js module resolution, since OrbitControls uses bare specifiers that need to point to the CDN URLs. This lets me import OrbitControls cleanly without worrying about path resolution issues.

Now I'm working out the ship's geometry — settling on a scale where 1 unit equals 10 meters, so the 10 km spine becomes 1000 units. For the rotating rings, I'm calculating dimensions where 12 biomes arranged in a circle form a torus with the right spin radius to generate 1g of artificial gravity. I'll use Three.js TorusGeometry to create each biome as a curved segment of the ring, with 12 segments arranged in a circle and small gaps between them to show the separation, plus connector locks joining them together.

For the spokes connecting the rings to the central spine, I'm placing 4-6 spokes per ring that run radially inward to the hub. The spine itself is a long cylinder running along the axis with a large forward shield disc that needs to be sized to cover the ring diameter.

At the aft end, I'm adding the propulsion systems with fusion engines and plasma nozzles, magnetic coils for deceleration, and a glowing exhaust effect. I'll also include the docking hubs, fuel tank rings along the spine, radiator panels that glow red, a communication dish, and the Aurora ferries docked at the aft hub. For animation, the rings will counter-rotate and I'll add star streaks flowing past.

Now layering in dynamic effects: engine plume flickering, shield bow-shock glow, pulsing radiators, and biome lights visible through windows. The ship moves relative to the camera through starfield and dust particle motion. I'm adding a nebula backdrop with a gradient sphere shader and placing Tau Ceti as a bright distant star ahead with a lens flare effect.

For the interior mode, I'll select from the biome names for each ring and create a long cylindrical tube interior representing the habitat section. I'm building this as a curved cylinder geometry with inward-facing normals so the camera can view it from inside.

The biome floor will have terrain with a lake, river, instanced trees, a small village of buildings, and scattered animals. I'm adding a lamp strip along the central axis with a day/night cycle animation, plus windows and locks at both ends. I'll include a person-scale figure for reference and implement first-person walking controls alongside orbit mode, with fog and bird particles for atmosphere.

For the interior locations, I'm planning three main areas to keep things manageable: the biome habitat, the spine corridor, and the hub/dock area.

I'm thinking through the biome implementation with 24 presets that each define a name, climate-based color palette, and vegetation type. The spine corridor and hub dock will be simpler scenes. I also need to add a UI panel with ship information from the novel—though the ship itself is never actually named in the text, just referred to as "the ship" by the crew.

Now I'm recalling the timeline details: the ship departs from Saturn's orbit around 2545, arrives at Tau Ceti (specifically Aurora, a moon of Planet E) around 2715. The novel's narrative begins around ship year 161 with a population of 2122, and arrival happens near year 170. I need to verify these dates and display them in the UI.

For the info panel, I'll focus on the specs I'm confident about: 24 biomes across two rings, approximately 2000 people, traveling at 0.1c, 11.9 light-years to Tau Ceti, destination is Aurora (moon of planet E), the ship has its own AI narrator, spine length around 10 km, ring diameter about 4 km, spin gravity at 1g, forward shield, and deceleration capability. Since I don't have web access this session, I'll mark uncertain timeline details as potentially imprecise from memory.

Now I'm setting up a single HTML file with Three.js to visualize the ship, defining the scale factor and unit conversions for the 3D model.

I'm configuring the WebGL renderer with antialiasing and proper tone mapping, then planning to use separate scenes for exterior and interior views that swap when toggled, each with their own camera and orbit controls.

For the exterior scene, I'm starting with a starfield made from thousands of points distributed on a large sphere to create the space backdrop.

Adding star streaks as a particle system that moves backward relative to the ship's forward direction, plus a nebula backdrop using either a shader-based gradient or textured sprites to give the scene atmospheric depth. Adding ship details... The hotspot labels cover all the major ship systems—the two biome rings with their 12 sections each, the spokes and hub, fuel tanks, radiators, engines, and ferry docks. Clicking a label or ship component animates the camera to focus on it and displays an info card, while raycasting on biome segments shows tooltips with names and lets you double-click to enter that biome's interior. For the interior, I'm setting up a cylinder geometry to represent the biome tube, with the player standing on the inner curved surface rather than a flat floor. I'll model the terrain as a parametric curved surface using noise for height variation, then add a water layer as a transparent mesh conforming to the ground at a fixed radius. For the lake, I'll use a simple approach with a standard material set to transparent and low roughness to keep it performant.

For vegetation, I'm placing around 1200 instanced trees across different biome types, with each tree oriented radially inward along the terrain's local surface normal. The buildings are just simple boxes for now. The lamp runs along the central axis as an emissive cylinder with point lights, and I'll animate the day-night cycle by varying the lamp's intensity and color while toggling small lights in the town on and off during night. The end walls are two disks with doors and emissive frames, possibly with a window showing the next biome through.

For the interactive elements, I'm adding animated birds as point particles and exponential fog that varies by biome for depth perception. The controls will combine OrbitControls for looking around with a first-person walking mode that uses pointer lock and WASD movement along the cylinder's surface—the camera stays at about 1.7 meters above the terrain as the player walks.

I'm also planning simpler interior locations like a zero-gravity spine corridor with handrails and blue lighting, plus a hub dock area, all built as separate scenes that swap in when needed. The UI will include a top title bar with mode-switching buttons. I'm working through the full list of 24 biome names from the novel, noting which ones are explicitly mentioned and marking the uncertain ones collectively rather than individually to keep things clean.

Writing the Three.js code... Adding maintenance light strips... The shield at the nose gets a shallow cone dish with an ablative rim that glows orange-blue using additive blending, plus a bow-shock sprite with a radial gradient that pulses. I'm also adding dust streaks as animated line segments moving backward along the Z-axis to create a warp effect, with speed controlled by the slider. The ship itself stays centered while the camera orbits, and I'm considering adding a distant planet in the background.

For the destination, I'll place a simple bluish sphere representing Tau Ceti e far ahead with a smaller moon orbiting it, lit by the star direction—toggleable and low-detail to keep performance light. I'm also implementing an HTML overlay system that projects labels onto the screen and hides them when they're behind the camera or too distant. Then I'll set up raycasting on pointer events to detect interactions with the ship meshes in exterior mode.

Now I'm building the interior system with terrain generation using value noise and fractional Brownian motion...

Finishing the noise function...

Setting up a cylindrical terrain grid with 96 points across an arc and 140 along the length, scaling height by biome amplitude and tapering it toward the edges and ends, then coloring vertices based on height and water level.

For positioning, I'm using a cylinder with the Z-axis as the vertical and defining points as (R - h) * (sin(a), -cos(a), 0) + (0, 0, z), where the upward direction at each point is (-sin a, cos a, 0). Computing vertex normals for the terrain mesh, then creating a separate water mesh at radius R - wl across the full arc and length with transparency and depth writing disabled so it renders as a water surface.

For trees, I'm sampling random points and filtering out those below the water level plus a small margin or near the arc edges, then placing them using InstancedMesh for both trunks and canopies. To orient each tree correctly, I'm setting up a dummy object at each point and using quaternion conversion to align the up vector with the local surface normal.

For the shell structure, I'm using a cylinder geometry rotated so its axis runs along Z, with a dark grey material on the back side. The key is splitting this into two parts: a lower structural shell for the walls and an upper section that acts as a sky dome using theta parameters to create just the upper hemisphere with a soft blue emissive material.

Now I'm working out the theta mapping after the rotation — the cylinder's cross-section gets transformed, so I need to figure out which theta range corresponds to the top half to create that sky effect.

For the sky mesh, I'll use thetaStart at π/2 with a length of π to cover the upper hemisphere, while the wall and floor shell will use thetaStart at -π/2 with the same length. For the lamp, I'm placing a long emissive cylinder near the axis at around y = +R*0.6 to simulate the spine lamps mentioned in O'Neill designs. I'll set up the lighting with a directional light pointing downward, ambient light, and a couple of point lights along the lamp, then animate the day/night cycle by modulating the light intensities and colors, with the lamp's emissive intensity ramping down at night.

At night, I'll activate emissive windows on the town buildings to simulate lights coming on. The end caps are two circular disks at z=0 and z=L with a dark material and an emissive frame ring, plus a lit rectangular door element. The town itself is a cluster of boxes on the terrain with animated emissive windows, and I'm adding a reference figure (a capsule) labeled with a scale marker. I'm also animating birds as points moving in loops, and setting up the spine corridor as a long cylinder with a smaller radius to represent the interior passage.

For the corridor interior, I'm using a 1-unit-to-10-meter scale with ring ribs every 2 units, handrails, emissive strips, floating supply crates, and blue-white lighting with fog. The zero-g mode will let players fly instead of walk. The hub dock is a large cylindrical bay with two ferries, docking clamps, a big door showing stars beyond, cranes, and lights. Given the code volume, I'm keeping these two extra interiors relatively modest in scope.

For camera controls, the exterior uses OrbitControls with a minimum distance around 400 meters and maximum around 40 kilometers with damping, while the interior will use OrbitControls with different constraints.

Writing the implementation... Finishing the label projection logic to handle screen positioning and depth culling, then setting up anchor objects for the ship. Now I'm defining the scale constants for the Ringworld structure — converting real meters to game units by dividing by 10, so the spine runs 960 units long with a 9-unit radius, the rings sit at z-positions of +200 and -160, and the biome tubes have a 20-unit radius.

Adjusting the ring positions to be symmetric at ±180 for balance, then placing the shield generator at the nose (z=+480) with a 240-unit radius to encompass the rings and tubes, engines at the stern (z=-480), fuel capsules clustered around the spine in the mid-aft section, four large radiator panels angled near the aft, and a 60-unit antenna dish pointing backward toward Sol.

The ship spans roughly 10 kilometers, so I'm positioning the default camera at distance ~1100 units for a good view. The scene includes a star sphere at radius 60,000, a planet at z=+45,000 with radius 2,500, and a nebula sphere at 90,000, requiring a far plane of 200,000 and enabling logarithmic depth buffering to handle the extreme depth ratio. For interior views, I'll use a tighter near plane of 0.05 and far of 3,000, and I need a disposal helper for rebuilding the interior when switching between exterior and cross-section modes.

For the cutaway effect, I'm toggling hull material transparency to reveal an inner emissive core—a slightly smaller torus arc per biome segment with climate-specific colors (greens and blues for the ecological layer). This ties the exterior and interior visually together. Now I'm setting up the biome presets with 24 entries, each containing Chinese and English names, climate type, ground and vegetation colors, tree type and count, water level, terrain amplitude, and sky/fog colors.

Defining biome presets... Finalizing the Ring A roster with a mix of confirmed and speculative locations, then moving into the code implementation.

Writing Three.js setup... Writing starfield shader... Writing nebula shader... Setting up the lighting rig with a directional light mimicking Tau Ceti's orange-yellow hue, a hemisphere light for fill, and a cyan point light at the rear for engine glow, while skipping shadows for performance but potentially using ambient occlusion instead.

For the interior, I'll enable shadows with a directional light and a 2048 shadow map casting downward from the lamp position, keeping the shadow camera orthographic to cover the biome area—I'll make this toggleable with shadows defaulting on indoors only.

Writing the complete implementation now with mode switching UI, component data, and pointer lock handling for interior walk mode...

Setting up pointer lock so clicking in walk mode requests lock, then tracking mouse movement to update yaw and pitch angles, and calculating the ground radius at any angular position using the same noise function as the terrain to determine the player's vertical position. Setting up camera movement where yaw controls horizontal direction and pitch controls vertical angle, then updating position based on arc angle and height while keeping the camera oriented to the curved surface. Clamping the arc angle and height to valid ranges, and using the up vector to maintain proper camera orientation relative to the surface.

For interior spaces, I'm implementing two movement modes: walk mode for the biome with gravity, and free-flight 6DOF for the other interiors, toggled by pressing F or clicking the walk/explore button. I need a shared terrain noise function parameterized per biome with a seeded random generator.

Generating terrain noise function... Writing noise functions... Writing HTML structure... Defining the parts data structure with ship components and their descriptions, then moving into the core JavaScript implementation — making sure the raycasting only targets exterior geometry and writing the cleanup function for disposing of Three.js objects.

Now I'm handling the interior scene management by setting up variables to track which biome is active and building out the interior environments, though I need to be careful not to dispose of shared textures like the glow texture.

For each biome interior, I'm calculating the tube length based on the ring circumference divided into twelve sections, which gives me about 1045 units. The main shell geometry uses a cylinder with specific parameters to create the lower interior surface, applying a standard material with a dark metallic finish and back-side rendering. Writing material setup... Adding scene elements...

I'm setting up exponential fog that shifts with the day/night cycle, which runs on a configurable timer with smooth transitions affecting the lamp, sky, and window colors. The camera starts inside the capsule looking down the corridor, and I'm building the spine structure as a large tube with ribs and emissive strips running along its length.

For the spine corridor, I'm adding handrails, rotating crates, wall hatches, and end bulkheads with doorways, using a limited set of point lights supplemented by emissive materials to keep performance in check. The hub dock features a large cylindrical bay with an open door showing a starfield, plus docked ferries with wings and nozzles, cranes, and crates lit by point lights.

I'm keeping both areas reasonably simple given the scope, then moving on to finalize the code. I need to add a mode switch to properly reset controls between exterior and interior views. Setting up the animation loop now with delta time clamping, mode-based scene updates, and control handling that switches between free-fly exterior controls and walk mode for the interior.

Writing animation loop... Writing the final implementation now. I need to set up click detection on biome segments with raycasting to show tooltips and biome info panels, plus double-click to enter directly. I'm also confirming that CapsuleGeometry and TorusGeometry are available in the Three.js version, then configuring the torus parameters for the biome tube exterior.

Now I'm working through the geometry details for the torus segments — calculating the arc angle, rotation offsets to center gaps, and deciding how to cap the open ends of the torus rings. I'm considering using small connector cylinders positioned at the gaps to seal the openings visually.

For the windows and lights, I'm planning to add thin emissive torus arcs at the same radius with a small tube diameter, positioned along the axial sides of the main tube to create a glowing strip effect. Instead, I'll use an instanced mesh of small boxes positioned on the torus surface as windows—about 30 per biome, placed with normals pointing outward and emissive cyan or warm colors. With 12 biomes that's 360 instances per ring, which is efficient. Now I'm working out the parametric positioning on the torus surface to place these windows correctly. Writing the scene code... I'll add a scale indicator to the HUD showing the ship's 10 km length for reference. For the interior lighting, I'm enabling shadow mapping with a 2048 resolution shadow map covering the lamp's ortho camera bounds, letting terrain and trees cast shadows while keeping the exterior unaffected since there are no shadow casters there. I need to be careful about toggling shadowMap.enabled at runtime and ensure materials update accordingly.

I'll simplify by keeping shadows on in the interior by default and adding a low-quality toggle that disables shadows and reduces pixel ratio with a material update loop for mobile devices. Now I'm starting on the ferry geometry using a capsule shape.

For the pointer events on the labels overlay, I need to set it to none globally but enable it on individual label chips so clicks can focus parts. I'm also implementing a camera focus animation that smoothly lerps the camera position and target over about a second using a tween system updated each frame.

Writing the code... Defining biome preset entries... Generating biome data table... Adding the final biome entries, then organizing the color palettes by climate type to streamline the code and avoid repetition across biomes. Defining color palettes for different biomes... Defining tree geometries for each type with appropriate heights and canopy shapes, then setting up instanced meshes for efficient rendering with random placement across the terrain while avoiding water areas and using noise-based density clustering. The player capsule is being scaled down to match the world dimensions.

For the town, I'm creating a cluster of buildings with emissive windows that can animate at night, and adding a system to generate 3D text sprite labels for points of interest throughout the biome using canvas textures—these will scale based on distance for better visibility.