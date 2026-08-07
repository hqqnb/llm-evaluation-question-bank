We need answer in Chinese with complete project. Need produce playable browser FPS in one response. We cannot actually run but need provide code. Need create full project as single HTML? We need include Three.js? Need self-contained maybe CDN? Since browser, likely use Three.js via CDN and custom game logic. Need include 3 maps, one Dust2. Need complete. Huge. Need craft compact but functional. Need manage token budget (90k). We can deliver one large HTML file maybe with JS modules embedded and Three.js from CDN. Need include assets generated procedurally (textures, models) to avoid external. Need robust code.

We need "directly complete project". In final answer include code and run instructions. Since no actual upload, but output code. Need ensure no placeholders.

Need decide architecture.

Given complexity, we need implement a solid FPS in browser with Three.js. Use CDN import map? Could use script tag from unpkg. We'll write single HTML. Use Three.js r128 maybe. Use pointer lock. Player physics, bullets raycast. Bot AI. Maps built from box geometries, textured/canvas textures, and collision data.

Need build 3 maps. Dust2-like: can create layout with walls and boxes. Need approximate geometry with axis-aligned boxes. Need coordinate system. We'll define walls as boxes and collision list. Need include key points. Need map 1 Dust2, map2 maybe "Dust_Ancient" with close quarters? Actually need 3 maps. We'll make three maps. Need complete playable.

Let's design engine.

Core:

- HTML canvas full screen, Three.js renderer.
- Scene, camera.
- Pointer lock controls custom: yaw/pitch, mouse movement. WASD movement. Jump/crouch/walk/sprint.
- Player object: health, armor, helmet, money, weapons, active weapon, ammo, reloading, invuln? Respawn between rounds.
- WebGL renderer with shadows? Use basic lights, Lambert/Phong. Need performance. Use MeshLambertMaterial with canvas textures maybe.

- Levels: define geometry as boxes. Each map has entities with pos/size/rot maybe all axis aligned. For visual and collision. Also spawn points, buy zones, barriers. Need bomb site markers.

- To not use complex height maps, use boxes; Dust2 slopes? There are ramps (A ramp, B? etc). Boxes can approximate slopes? Could include Ramps (triangular geometry) but collision for player standing on slope complicated. Use stair steps made of boxes to approximate ramp. Need "坡道" feel. We'll create stepped ramps with boxes. Camera height and collisions.

- Map structure:
Coordinate system: x,z horizontal, y vertical. Use Three.js. We'll define walls as boxes with center [x,z], size [sx,sz], height.

Player collision AABB with boxes. Bullet raycast intersects meshes.

Map definitions: Map object includes boxes (visual/collision), spawn points T/CT, bomb sites A/B, buyzone points, maybe navgraph for AI.

Need build Dust2. Need classic layout. Let's recall top view approximate.

Dust2 coordinates (simplified):
T spawn at one end (west? CT at east). Map length ~200 units? We'll scale.

Classic Dust2:
T spawn is west outside cave? Routes:
- T spawn to A long: path north to A long, through long doors to A site.
- T spawn mid: through T mid to mid doors (double doors) to CT mid/ct spawn, or catwalk to A site.
- T spawn to B: through tunnels to B site.
CT spawn is southeast/east, leads B site west via B doors? Actually CT spawn connects to B site through B doors and to mid via CT mid, and to A site via short/ct.

Need approximate.

Use grid units: let's map to 1 unit maybe "feet". We'll draw walls with boxes.

Let's create map bounds 220 x 220? Need.

Important: Player height 1.7? Use world units maybe. We'll use units as meters. Map size ~100m. Walls height 4m.

Maybe better use "unit" 1 = 1m. Dust2 routes length: T spawn to A long ~? 80m? Could be large. Use map x [-100..100] z [-100..100].

We'll define 3D boxes with dimensions. Need not exact real. We'll design.

Need "box" collisions. To build map, each wall can be rectangular box. For visual, create Mesh with texture from canvas; for bullet collision, raycast against meshes; for player collision, AABB.

Alternatively, use Three.js BoxGeometry for walls with scaled texture. Need color textures per map. Dust2: sand-colored walls. We'll create CanvasTexture of brick/stucco pattern? Use small canvas with noise, beige. Floor: sand. CT building: stone.

For character models: simple but not "blocky"? Could use low poly using boxes/capsules with camouflage colors. Need not look placeholder? We can create soldier models from boxes with head, torso, limbs, gun. Use meshes with materials. Third-person not visible unless spectator? We can include for bots and player (not shown). Need animation? Maybe simple.

Weapons: Need viewmodel gun in first person. Could create gun from boxes (rifle, pistol, knife?). Need fire animation: gun moves back, muzzle flash, shell casing. Need textures.

UI: HTML overlay with crosshair, health, armor, ammo, money, round timer, score, minimap canvas, buy menu, kill feed, damage indicators, bomb status.

Need Bot AI. Need pathfinding? We can implement navmesh with graph nodes for each map, and bots move node-to-node. Need basic combat, objectives, buy.

This is huge. Need simplify but functional and rich.

Need allow player and bots. Player side selectable T/CT. Round starts. Player in buy zone buys. Rounds increment economy. Bots follow AI.

Need three maps all with bomb mode. Map selection screen before game.

Need game flow:
- Main menu: select map, side, bot difficulty, bot count.
- Game: rounds, freezetime 5s? Buy time 10s, round time 1m45. Bomb timer 40s. Win conditions: eliminate all enemies, bomb exploded, defused, time expired.
- Player death -> spectate bot or fly? We can support camera follows alive teammate or free.
- After round end, score, economy update, next round.

Need pause? restart. We'll implement pause/escape maybe.

Need implement audio with WebAudio generated sounds (gunshot etc) using noise/oscillators. Footsteps.

Need ensure no external assets. Use Three.js from CDN. If offline cannot run; but acceptable.

Need implement code in one HTML maybe ~3000 lines. We have 90k tokens. Need be careful.

Could use Three.js import via script. Need include import map? Simpler: <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>. Use global THREE.

Need maybe pointer lock with Escape menu. We'll handle.

Let's architect code modules in one IIFE.

State:
- gameState = 'menu', 'playing', 'roundEnd', 'paused'.
- selectedMapName, selectedSide, botCount, difficulty.

Levels:
Define data structure:
function createMap1() etc.
Map = {
 name: "Dust2 Remake",
 side: "CT"/"T"? theme color,
 ambient: 0x...
 walls: [ {pos:[x,y,z], size:[sx,sy,sz], color?, texture?, type?} ],
 spawns: { T: [x,y,z,yaw], CT: [...] },
 bombSites: { A: {pos:[x,z], radius}, B: ... },
 buyZones: { T: polygon? or center+radius lists, CT: ...},
 nav: [ {pos:[x,z], links:[indices] } ],
 playerStart? etc
}

Need collision for all walls? Use walls array with sizes. But map also has decorative objects not collidable? We'll just use walls for both.

Need define all map geometry with boxes. Could maybe create a floor plane and walls boxes. Floor: a large box or plane. Need collision with floor? We'll set player y by floor from map.ground y per position? Need inside map floor maybe 0. All boxes have y center based on floor. Player y = ground + player's vertical offset. Movement y? Since flat floor, player can jump. We'll store y velocity, and groundY=0. Boxes with height maybe can be climbed? No jumping on boxes? We can set player can jump but not onto boxes unless step? Simpler: no vertical collision except ground floor. But "Dust2 slope/high difference" needs elevated platforms e.g. A site platform, B platform, catwalk? We can use boxes as obstacles; player cannot jump onto tall boxes if jump height low. But can stand on lower platforms? Better implement vertical collision? Could be complex but manageable: player AABB vs boxes, update position with collision resolving in x/z and y? Need handle standing on surfaces.

Maybe represent player as AABB (radius 0.4, height standing/crouch), collide with all wall boxes. For floor colliders, boxes can be platforms. Need resolve collisions: For each frame, move player by velocity. Check player AABB against all boxes. For each box, compute overlap. Resolve movement along axes. For vertical, if player falling and overlaps top, set y = box.maxY. Also walls. We also have floor ground plane at y=0 for all? Some maps have ground base. We can create floor plane at y=0. Then platform boxes on top.

Need AABB collision:
function collidePlayer(pos, halfW, height, vel, dt):
- pos is bottom center. Player AABB from pos.x-halfW, pos.y, pos.z-halfW to pos.x+halfW, pos.y+height, pos.z+halfW.
- Move x: pos.x += vel.x*dt; then for each box, if overlap OBB axis aligned with player and box, push out along x if overlap smaller? Since we move x only, we can compute overlap into box. Need simper: after moving x, test AABB vs each collider, if overlap then adjust x based on previous? There's algorithm:
  For each axis separately:
  pos.x += dx; check penetration (minx<maxB.maxx && maxx>B.minx etc in y,z). If collision, if dx>0 pos.x = B.minx - halfW; else pos.x = B.maxx + halfW; set dx=0.
  Similarly z, y.
Need consider all boxes. Since boxes static, works but can push through if multiple. Fine.

Need ensure ground collision: pos.y = max(groundY, pos.y? Actually vertical gravity causes y velocity. Collide with floor box? We can include large floor box at y=0 height 0.2. If pos.y after move below top, set y=floorY. For platforms, collision works.

Need movement:
- Player velocity vx, vz from input; acceleration/damping.
- Eye height = pos.y + 1.6 (standing) or 1.1 crouch.
- Jump: vely = 4.5; gravity -11.
- if grounded allowed jump.
- Crouch reduces height and speed.

Need weapon:
Define weapon data:
{
 pistol (USP/Glocks?), primary, knife, grenades.
 name, type, price, damage, armorPen, fireRate, magSize, reserveMax, reloadTime, automatic, spread, recoil, zoom? movementSpeed, penetration? bulletRange? pellets (shotgun), projectile? 
}
Need weapons:
- Knife
- USPs/Glock? For CT pistol USP, T Glock? We'll just pistols: "Glock" and "USP" depending side? Perhaps buy menu.
- P250, Deagle
- SMG: MP9, P90
- Rifle: AK-47, M4A4, AWP, SG553
- Shotgun: Nova
- Grenades: HE, Flash, Smoke, Molotov/Incendiary. Throw arc.

Need weapon model view:
- Create group of box meshes at camera. For each weapon, multiple boxes. Need animation for recoil and reload. Use currentWeapon.
- Fire: raycast from camera center with spread; for shotgun multiple pellets. For AWP zoom toggles FOV 20? Actually mouse right? Need implement right click to zoom for sniper or scope? Could use right mousedown to aim/scope. Need prevent context menu. For all rifles perhaps right click aim? We can implement zoom for AWP only maybe all with recoil? But request "开镜". Use right click for AWP (toggle zoom) and maybe rifles. We can.

- Hit detection: raycast against enemies (bot meshes/head hitboxes) and world. Need visible enemies as simplified soldier Group. For each bot, collider bodies: torso box, head sphere, limbs boxes. We'll register mesh for hit (with bot ref and part). Use Raycaster from camera to target. Intersect all scenery and bots. Filter target behind walls. Need world meshes include invisible? Bullet should stop at world and walls. Use raycast against map meshes and bots, get first.

Since all meshes in scene. Need update scene graph for performance. Use Raycaster.intersectObjects(scene.children, true), find first; if object has userData.bot and userData.part, hit. But if wall mesh is before, no bot. Need create array 'shootables'. Use raycaster.

Need bot body parts:
- Head sphere, chest box, arms, legs. Use materials with camo colors. Each part.userData = { bot, part:"head"/"chest"/"leg", damageMultiplier:4/1/0.75 }. For raycasting.
- Player has no third-person model? Could be spectator? We'll maybe not.

Need AI pathfinding:
Need create nav graph per map. We'll manually define nodes. Bots choose route to objective. Need dynamic. We'll implement waypoint graph with positions and links. Bots move toward target waypoint, then choose. Need A* or simple? We'll implement BFS/Dijkstra shortest path to target node. Need target nodes near objective, etc.

Could simplify using node graph and Dijkstra with cost. Need each map have nav nodes. For Dust2, design nodes for T spawn, routes, bomb sites. For bot movement, choose random waypoints. Need allow player not on graph? Bots can't navigate through arbitrary. We'll create nodes along paths. Bots use path from current nearest node to goal node. Need when bot reaches or deviates maybe recompute. Since player movement not in graph, bots don't collide with player? We need implement separation maybe.

Need map nav nodes. We'll create arrays with positions. Need ensure connect. Could use line-of-sight links? Easier: list nodes with explicit links.

For Dust2, nodes:
0 T spawn
1 T ramp/mid entrance
2 T mid
3 Mid doors CT side
4 CT spawn
5 CT to A
6 A site long? Need more.

Let's define coordinates more precisely.

Let's choose coordinate system for Dust2 bottom-up. Need layout.

We'll create map using boxes. Need for gameplay. I'll design top-down mentally.

Coordinate: x east, z south? Let's choose x west-east, z north-south. Map center x 0 z 0. T spawn on west/southwest? CT spawn on east/southeast? We'll see.

Classic simplified with T at x=-50,z=20; CT at x=50,z=40. A site at x=40,z=-30? B at x=-20,z=65? Let's define.

Maybe easier draw ascii:

North (-z) top.

T spawn west (x=-70,z=-10). CT spawn east (x=60,z=40).

A long on north side: from T spawn along north wall to A site at east-north.
Mid: from T spawn east through mid double doors to CT spawn/mid.
B: from T spawn east/north? Actually B tunnels from T spawn to B site at northeast? In Dust2, B site is near CT spawn? Wait.

Let's recall actual map:
T Spawn is south? In classic Dust2, T spawn is in a corner southwest? Let's use a stylized layout but with names.

Classic Dust2 route:
- A Long (from T spawn → long doors → A site)
- A Short/Catwalk (from mid to A site)
- Mid (from T spawn to mid doors, CT spawn)
- B Tunnels (from T spawn to B site)
- CT spawn connects to B site and A site via CT ramp? Actually CT spawn is between B and mid.

We can build a map that "feels" like Dust2: Long corridor, mid with double doors, short ramp/catwalk, B tunnel with narrow choke. It doesn't need exact coordinates but routes.

Let's create own scaled map under 100m.

Use maps as arrays of wall boxes. Floor y=0. Walls height 4.0, thickness 0.5.

Dust2 map coordinates:

Let's set:
A site at east-north: x=40,z=-25.
B site at west-north? x=-15,z=-40? Actually T from west? Let's design with T spawn at west x=-70,z=20; CT spawn east x=65,z=10. A site north-east (x=30,z=-45), B site north-west (x=-20,z=-50). The middle corridor goes from T spawn east to CT mid; A long goes from T spawn north along west/north? Let's create.

Need not perfectly match classic but "T can from A big, A small, mid, B洞".

Maybe classic Dust2:
- T base west/south. B tunnels to B site north-west? Wait in actual, T spawn is southwest; B site is northwest? Let's use:
T spawn at x=-60,z=40 (southwest).
CT spawn at x=60,z=-20 (northeast? actually CT spawn east side).
A site is southwest? Hmm.

Let's maybe use a known top-down from memory:
Dust2 has T spawn at left (west). A Long goes along top edge (north) from T spawn to A site at top-right (northeast). Mid leaves T spawn to the right/east through double doors to CT spawn. Tunnels leave T spawn to upper-left? B site is on left? No, CT spawn is bottom-right? Let's not worry.

We'll implement stylized with correct names. We need all key points connected.

Let's define map layout coordinates in a grid:
- T spawn at (x=-80,z=20) with route east to mid, north to long, north-west? to B.
- CT spawn at (x=70,z=-5).
- A site at (x=55,z=-55) (northeast).
- B site at (x=-25,z=-40) (northwest).
- Middle: x from -40 to 30, z=20? CT mid near x=30,z=-10? Need.

Let's place boundaries: Map bounds x[-95,95], z[-70,70].

Let's draw rough:

North (-z):
- B site at (-25,-45)
- A site at (55,-55)

Middle horizontal corridor at z=10? T spawn (-80,20) -> mid -> double doors -> CT spawn (70,5). A short/north from mid at x=30,z=10 goes north to A site. A long from T spawn north along x=-60,z=20 to north area then east along z=-60 to A site. B tunnels from T spawn north-east? T spawn to B site via (x=-55,z=-10) then (-25,-40). CT spawn to B site via long path? B doors/west.

Let's create walls to define these.

We'll build map by adding wall blocks along route edges. Need collision doesn't require entire map? We'll create boundary walls, corridor walls, rooms. Use entries.

Need implement helper addWall(pos,sx,sz,height,color). Walls centered at y=height/2.

Coordinates:
Use z negative as north, positive as south. Need be consistent.

Map1 "Dust2 Layout" boundary:
Outer walls around x -95..95, z -70..70 maybe (top/bottom/sides). But if T spawn at x=-75,z=15 inside boundary.

Let's create floor at y=-0.1 size 190x140.

Important: Player starts y=0, walls from y=0 to height. For doors/openings, use wall segments not full.

We'll define "wall" array. The player can pass through openings between segmented walls.

Let's build map walls manually. But many walls. Could use "rooms/corridors" approach.

However 3 maps with huge wall lists manually might be heavy. We can create procedural maps? For Dust2, need details. But can define using functions.

Need maps 2 and 3 can be simpler with unique tactical structures. Maybe use labyrinth and industrial. Need no explicit Dust2 quality? Need all playable.

Maybe we can create Map 1 Dust2 detailed; Map2 "Tuscan-like" with village; Map3 "Nuke-like" with vertical? Need.

We can implement generic map via a layout grid? But top-down grid can create walls easily. Yet Dust2 needs routes. Could use grid strings for maps? Define wall from cells. But textures and 3D boxes.

Option: Use Tile-based level from ASCII grid. Each character represents wall/floor/open. This is easier define maps, and generate collision. But Dust2 with "slopes" not possible. Still tactical. Need "not just color blocks." Could render walls with height 4 and textures from canvas. A tile map can represent walls. But walls as full cubes maybe "boxy", okay. We need varied heights, boxes. We can overlay boxes for crates/platform/scaffolding.

Maybe combine: use tile map for base walls and obstacle boxes for crates. But route shapes can be precise. However tile grid creates chunky walls but can be stylized. Need maps: 3. Huge grid maybe.

But using manual ASCII map with tiles 1 unit? For map 100x100, grid 100x100 too large. Use tile size maybe 4m, grid 40x30. Walls thickness 4m, too thick but okay. But Dust2 requires narrow doors maybe 2m. Use tile size 2m, grid 80x70 too large. We can use 2m and string lines ~70 chars. Possible but time-consuming. But manual walls as boxes more flexible. Hmm.

Need efficient code. Let's use manual "box" layout with segments. We can define arrays of boxes for collision with positions. We can create walls by calling W(x,z,sx,sz) where wall center. Need open doorways by arranging segments.

Let's first design Dust2 layout in coordinates:

Let's set map bounds for Dust2:
- T spawn area: rectangle x [-90,-65], z [-10,30].
- CT spawn area: rectangle x [65,90], z [-30,0]? maybe.
- A site: x [40,70], z [-65,-30].
- B site: x [-60,-20], z [-65,-30].
- Mid corridor: x [-65,65], z [10,30]? Wait.

Let's orient:
North = -z, South = +z. T spawn southwest, CT spawn east/north maybe.

Better:
- T spawn top-left? Let's create map "left" side T, "right" side CT.
Let's place T at x=-75,z=-5 (west). CT at x=70,z=20 (east). A at x=45,z=-50 (north-east). B at x=-45,z=-50 (north-west). Mid corridor goes from T x=-60,z=0 to CT x=60,z=0? This runs horizontally (east-west). A Long goes from T north and east along north side. B tunnels go from T northeast then north to B.

This resembles map if you rotate 90? Fine.

Route details:
T Spawn (-75,0) with exit:
- North to A Long: start at (-75,-15) -> road along north boundary? Actually T long route along north/west to A site. Long doors at (-20,-55) then A site (45,-55).
- East to Mid: T spawn (-60,0) -> Mid corridor (-30,5) -> double doors (10,10) -> CT mid (40,10) -> CT spawn (70,20); also A small from mid/north at (25,-10) -> A site (45,-50).
- Northeast to B tunnel: from T spawn (-60,-15) -> B tunnel entrance (-40,-15) -> B tunnel (-30,-30) -> B site (-45,-50)? Hmm if B site at (-45,-50), then T needs go north-west. But from T spawn to B tunnel should be north-east maybe. Let's set B site at (-45,-55), tunnel from T spawn: (-60,-15) to (-45,-30) to (-50,-55). But B site is north-west, okay.

We need CT connect to B site and A site:
- CT spawn (75,20) -> mid (60,10) -> CT mid (45,-5) -> to B site? Need "B doors" from CT mid to B site (x=??).
- CT spawn -> B site via long hallway (x=65,z=-35) maybe west to B site. B site at (-50,-55); CT route from CT spawn (75,10) west along x=75? Actually B doors on east side of B site: at (0,-55)? Hmm.

Maybe let's set B site near center/north (-25,-60), so CT can approach from south via B doors, T tunnels from west.

Let's not over-optimize. Need map feel: many engagements. We'll create abstract but with named points.

Maybe easiest: Use "Dust2" layout known by approximate coordinates from a minimap: We can fabricate using these key positions:

T spawn: (-40, -50) (southwest)
Mid: (-20,-20) (middle)
A long: from T spawn (-40,-50) north to (-40,-70) then east to (0,-80)? Actually north negative z. A site at (10,-65)? Hmm.

Let's use coordinate with z positive south? The actual map top-down maybe T spawn bottom-left; A site top; B site bottom-right? no.

Let's commit to a consistent layout. We'll define map coordinates but then implement all names in code. It doesn't need actual topology copied exactly if all routes exist and flow feels.

Proposed DUST2-like Layout:

View minimap with north up:
- T spawn: (x=-45, z=45) bottom-left.
- CT spawn: (x=45, z=45) bottom-right.
- A site: (x=45, z=-45) top-right.
- B site: (x=-45, z=-45) top-left.
- Mid: horizontal corridor across bottom/middle from T spawn to CT spawn. Actually mid connects T spawn to A site? Classic mid leads to CT spawn and A short. Let's:
  * T spawn at bottom-left.
  * Route to B tunnels from T spawn north along left side to B site.
  * Route to A long from T spawn east? Hmm T bottom-left to A top-right is diagonal: A Long goes along bottom/right? Actually A long from T spawn to A site can be top-left? 

Let's choose:
- Mid corridor from T spawn (-30,30) to CT spawn (30,30), running east-west at z=30. Double doors at x=0,z=30. From mid, "A short" goes north (z decreasing) at x=20? to A site.
- A long route from T spawn (-45,45) to x=45,z=45? No.

Could instead follow actual Dust2 orientation:
T spawn at southwest bottom; CT spawn at east bottom. A site at north (top), B site at west? Actually yes? Let's search memory from memory of minimap:
Dust2: T spawn at left, CT spawn at right. A site at top-right; B site at top-left. Mid runs from T spawn to right side/CT spawn. A Long runs along top from T spawn to A site. B Tunnels run from T spawn to B site? Wait T spawn to B tunnels goes up-left; B site top-left. This is exactly: T spawn bottom-left, CT spawn bottom-right, A top-right, B top-left. Mid corridor horizontal from bottom-left to bottom-right. A long: on top boundary heading right. B tunnels: vertical on left. Good! Let's use this:
Coordinates (x right, z up? if north = +z). But earlier z negative north. Let's set z positive north? Simpler for top-down: z positive north. Use y up. Bounds z -60..60. T spawn at (-60,-20), CT spawn at (60,-20), A at (55,40), B at (-40,40). Long route along top? Good.

Let's set z positive north, x east. Then A site top-right, B site top-left. T spawn bottom-left, CT bottom-right. Mid horizontal at z=-5 (between). A Long from T spawn north to top then east to A. B tunnels from T spawn north to B. A short/mid from mid to A site. CT to both sites via CT mid/回防.

Coordinates:
- T spawn center (-65,-30)
- CT spawn center (65,-25)
- A site center (50,45)
- B site center (-35,45)
- Mid corridor: from T spawn x=-60,z=-10 to CT spawn x=60,z=-10.
- Double doors at x=0,z=-10.
- CT mid / CT spawn: route from mid to CT spawn and up to A/B.
- A Long: T spawn exit north at (-65,-10) then corridor along west side? To go to A site top-right, long must go along top boundary from left to right. So from T spawn go north to (-65,45), then east along top? But B site also at (-40,45), that would conflict. Need A Long should go from T spawn to A site without crossing B site. In actual Dust2, A Long is a route along the north boundary, but B site is also north-west? Let's recall: B site is north-west, A long starts from T spawn north-east? There is a separator. B tunnels go to B site; A long goes along far east? Hmm.

Maybe better: T spawn at bottom-left, A site at top-right, B site at top-left. A Long from T spawn to A site should be along right side? Let's define:
- T spawn (-60,-40)
- A site (55,45)
- B site (-45,45)
- B tunnels: from T spawn north through left tunnels to B site.
- A Long: from T spawn east to x=-20? then north/east along right side? But CT spawn bottom-right. Let's make A long runs from T spawn along bottom/right up to A site? That would cross CT. Not.

Let's set T spawn at bottom-center (-10,-50)? Then A long to top-right, B to top-left, mid to right. Okay.

Maybe produce stylized not actual.

Maybe using known map via code? Could hardcode actual dust2 layout? Need coordinates. Maybe I can recall from de_dust2:
There is a "T spawn" in a little area at west/southwest. A Long is a long outdoor path from T spawn to A site on the north-east. Mid leaves T spawn and goes east to double doors. B Tunnels leave from T spawn to a set of rooms north-west? Actually B site is west? Let me visualize:
- At T spawn, looking outward, left is B tunnels? Right is A long? Mid straight. Actually in Dust2, T spawn has three exits: left/behind to B tunnels, right to A long, middle to mid. Hmm.
- A site is to the right (east) and north. B site is left (west) and north. CT spawn is east/south. So T spawn is southwest. Yes.

Thus:
A route from T spawn to A Long goes north/east? B route goes north/west. Mid goes east. Good.

If z positive north, x east:
T spawn at (-5,-40) (bottom center? Actually southwest, x negative, z negative).
B site at (-55,40) (west north), A site at (55,40) (east north). CT spawn at (55,-20) (southeast).
A Long: from T spawn go north along west? No to A east then up.
Maybe A Long exits T spawn at (-20,-40) then goes east as "long houses" then north to A site? Let's make:
- T spawn center (-10,-45). Exits: north to B (left side), east to mid/A long, and middle to mid.
- A Long: T spawn east to (0,-40), then north along x=0? Actually that would be center. Maybe too close.

Let's avoid exact and just implement map. We need playable more than exact. However user explicitly asks high fidelity, but impossible in text? We need make credible.

Alternative: Use "Dust2" from google memory? Could draw with coordinates from de_dust2 in Source? I'm not sure.

Given constraints, implement key named areas with plausible layout and call it "Dust2 Remake". Need maybe enough.

Let's create our Dust2 layout now with walls. We'll use a compact map. Need also maps 2 & 3; maybe map2 "Dust" fallback? But user wants 2 unique. We'll create Map2 "Aztec" style and Map3 "Nuke" style. But can be simpler.

Need enough code to run. We can define map walls using helper functions.

Let's choose coordinate orientation for all maps:
- x east, z north (positive north), y up.
- player y = 0 ground.
- walls height 3.5.
- Boxes/crates maybe height 1.2.

Define map bounds from -100 to 100.

Dust2 map structures (general):

Let's implement with a function addBox(map, x, z, sx, sz, options). It creates box centered at [x, h/2, z] with size [sx,h,sz], plus texture. For walls, we specify segments.

To make rooms:
- Boundary outer wall around x[-100,100], z[-80,80]? Actually need coordinate.
- For map, we can create wall along perimeter. But need inside open.

Let's set DUST2 bounds x[-90,90], z[-70,70].
Boundary four walls: x=-90 wall from z=-70 to70, center x=-90, z=0, sx=1, sz=140. x=90 similarly. z=-70 wall, z=70 wall.
T spawn at (-70,-20) (southwest). CT spawn at (70,-20) (southeast). A site at (45,50) (northeast). B site at (-45,50) (northwest). Mid corridor from T spawn (-60,-20) to CT spawn (60,-20) at z=-20? Actually z=-20 is south side, bottom. A long from T spawn? Hmm.

Maybe use z positive north, south negative. T at bottom-left, B top-left, A top-right. CT at bottom-right. Good.

Mid line from T spawn to CT spawn at z=-20. Double doors at x=0,z=-20. T spawn at (-60,-25). CT spawn at (60,-25). A site at (45,50). B site at (-45,50).

Route A long: from T spawn east? T to A site should go east then north. We'll create A Long route:
- T spawn exits east along "long houses" at z=-15, from x=-60 to x=40; then north at x=40 to A site (z=-15 to 50). This is basically mid? Need separate from mid. Let's place A Long path at z=-5 (north of mid) and Mid at z=-25. T spawn has two exits: one to mid at z=-25, one to long at z=-5. Long path from x=-60 to x=40 at z=-5, then north to A site. This makes long a horizontal route above mid. Good.
- Mid: from T spawn at x=-60,z=-25 east to x=60,z=-25, then CT spawn. Double doors at x=0,z=-25. CT mid maybe from x=60,z=-25 to CT spawn (70,-25).
- A short/catwalk: From mid at x=30,z=-25 go north along x=30 to z=0? Then to A site? We'll create "A小" route from mid (x=30,z=-20) north to A site (x=45,z=50), maybe with a ramp/stairs.
- B tunnels: from T spawn at x=-70,z=-25? Actually B site at (-45,50). Need T route from T spawn to B site along west/north. T spawn at (-70,-25), B at (-45,50). Route: from T spawn north at x=-70 to z=50, then east to B site. This is a long west side corridor, but actual B tunnels are close. We'll call B洞 route along x=-70 from -25 to 50, then east to B site. Need more narrow with walls.
- CT回防:
  * CT spawn at (70,-25) to B site via CT mid/B doors: from CT spawn go north at x=70 to z=0, then west to B site? B site (-45,50). Need "B doors" at x=20,z=50 maybe. We'll add route from CT spawn (65,-25) north to x=65,z=50, then west along z=50 to B site (through B doors at x=0,z=50). But that's same as A site? Wait A site at x=45,z=50. If CT route along z=50 from x=65 to -45, it passes A site. Need separate: B site west, A site east; CT spawn to A site via CT mid north/east. We can have CT route at x=70 to z=50, split: to A site at (45,50) (left) and B site at (-45,50) further left. That means CT can rotate along top corridor from A to B, which is not classic but okay.
  * A site is at (45,50), CT route arrives from south/east via A ramp? We'll make A site accessible from long at x=40 and CT route x=70.
- B site at (-45,50), accessible from T tunnel west and CT route east.

Need ensure A and B not too close? A site x=45,z=50, B site x=-45,z=50 separated by 90, okay.

Let's specify wall segments:

Map bounds:
- outer walls: x=-92,z=0,sz=140; x=92,z=0; z=-72; z=72. But T spawn at x=-70,z=-25 inside.

T spawn room:
- T spawn area roughly x[-85,-55], z[-35,-10]? Need walls around? Since boundary left, south, but open to routes.
Add walls:
  - T spawn north wall? z=-10, from x=-85 to -50? But route exits at z=-5? Hmm.
Let's create separators:
T spawn is open plaza x[-85,-55], z[-40,-10]. Exit north to A long at x[-70,-55], z=-10. Exit east to mid at x=-55,z[-30,-20]. Exit north-west to B tunnel? Actually B tunnel from west side.

Maybe better:
- T spawn at (-70,-25). To A Long: go east? No.

Let's design simple with paths:
Paths are open corridors between walls. Let's generate rooms using "lane" walls.

Path1 Mid: T spawn -> east along z=-25, x=-60 to60. Walls on both sides:
- North wall of mid: segments along z=-15, x=-60..60? But if A long at z=-5, then mid north wall at z=-15. 
- South wall: z=-35, x=-60..60.
Openings from T spawn: T spawn is at left of these walls. Need T spawn connected to mid at x=-60. So T spawn is west of mid corridor, x[-85,-60], z[-40,-10]. East exit to mid at x=-60.
Thus:
  T spawn room: outer west boundary x=-92; south boundary z=-72; north wall? Need no. T spawn open.
  Mid corridor z[-35,-15], x[-60,60]. North wall at z=-15, South wall at z=-35. But if we put continuous wall, T spawn at x<-60 not inside mid. T spawn is west of corridor; at x=-60 opening between walls.

Add wall "midNorth" from x=-60 to60 z=-15, thickness .5; "midSouth" from x=-60 to60 z=-35. With gaps at double doors? The mid corridor itself between walls. T spawn at x=-75,z=-25 can enter corridor at x=-60 where wall ends. Good. Double doors are a choke at x=0? Need wall across mid at x=0 with two doors? To create "double doors", add a wall segment across mid at x=0 with two openings? Actually mid corridor runs east-west; double doors are a wide gate at the east end? In Dust2, double doors are in the middle between mid and CT mid. Could create wall perpendicular at x=0 from z=-35..-15, with two door openings: one at z=-20? But that blocks mid. Maybe better: add "double doors" as two large gate boxes on either side of a central opening? Hmm.

We can add two large wall segments at x=-1 and x=1? Let's place doors at x=0: to create "double doors" with openings at z=-25 (center). Add wall segment from z=-35 to -28 and from z=-22 to -15, leaving 2m gap at z=-25? But since we want a gate, no need. We'll just call a wider gate and add visual doorframe walls: at x=0, two vertical walls (x=0, z=-35..-20 and z=-20..-15? Actually opening in center).
Maybe maintain corridor with doorframes: wall North/South at x=0? Let's just visually.

Path A Long: T spawn to A site along north. We set z=-5 horizontal lane above mid, separated by midNorth wall. T spawn at x=-75,z=-5 can enter A Long at west, but T spawn open to that lane. A Long is corridor z[-10,0], x[-70,45] (north of mid). It then turns north at x=45 to A site z=0..50. Need A Long walls:
- south wall of A Long is same midNorth wall at z=-10? Wait mid north wall at z=-15, if A Long z[-10,0], then south wall at z=-10. Let's set:
  Mid corridor z[-35,-20].
  A Long corridor z[-10,0].
  Separator wall between at z=-15? Actually if mid north wall at z=-15, then mid corridor z[-35,-15]. A Long corridor z[-10,0], with a wall between. T spawn is west of both. From T spawn, can enter A Long at x=-60? Need opening at z=-5. T spawn outside west; A Long starts at x=-75,z=-5. Good.
- A Long north wall at z=0 from x=-70 to45; south wall at z=-10 from x=-70 to45. But the route turns north at x=45: wall at x=45 separating? We'll add.
- A site area at x[10,80], z[0,55]? Actually A site east/north. A Long enters at x=45,z=0 going north, then opens into A site. A site is room x[20,80], z[20,55]? Need.

This creates A Long horizontal from west to east. A site at top-right. Great.

Path A small from mid to A site: "A小" is a catwalk from mid corridor z=-20 to A site z=20 at x=30, parallel to A Long? In our layout, mid corridor z[-35,-15], A site north. A small can be a vertical path at x=25 from z=-15 to z=30, with walls on sides, perhaps elevated? It connects mid to A site.

Need "中门" and "CT spawn": Mid corridor east end x=60 opens to CT mid/CT spawn. CT spawn at x[60,85], z[-25,-10]? Actually CT spawn at bottom-right. We can define CT spawn area x[60,85], z[-35,0]. Mid corridor ends at x=60. CT spawn is east of mid and open. CT has routes:
- From CT spawn to A site via north/east path at x=80, z=-10..50.
- From CT spawn to B site via CT mid to B doors along north? Need CT to B route from CT spawn to B site. Since CT spawn is bottom-right, B top-left. A path along east then top to west passes through A site; but can route through CT mid/northwest side of B. We'll create a path from CT spawn to B site along z=0? Let's design:
  CT spawn area x[65,85], z[-35,10]. North exit at x=70,z=10 to "CT mid" path z=10 from x=70 to x=-30? Then to B doors/entrance at x=-30,z=10? B site at (-45,50). Need path vertical at x=-30 from z=10 to50. Good.
- CT to A site: from CT spawn north at x=80,z=-20..50 then west to A site (75,50?) Actually A site at (45,50). CT can enter A site from east at x=75,z=30. Good.

Need B Tunnels:
T spawn to B site: We need T route from T spawn to B site without crossing A long. We can have B tunnel from T spawn west/north:
- T spawn at x=-75,z=-25. B site at (-45,50). Route: from T spawn west? boundary x=-92, no. Better from T spawn at x=-60,z=-25, take a north-west corridor at x=-70,z=-10? Hmm.
Because A Long occupies z[-10,0], B tunnel needs go north from T spawn to z=-10, then west? Let's build:
  From T spawn, at x=-80,z=-25, go north along x=-80 to z=10 (west of A Long). Then east/north to B site? But B site at (-45,50), so from x=-80,z=10 go east along z=10? Actually B site at (-45,50), so need go east then north.
- We can create B tunnel route: T spawn west side corridor x[-85,-70], z[-25,10] (narrow) then horizontal at z=10 from x=-85 to -20? Then north at x=-20 to B site? Wait B site at -45, not -20. Let's set B site at (-45,50); B tunnel approach from south at x=-45,z=10..50. To get from T spawn to x=-45,z=10, use corridor along z=10 from x=-80 to -45. Then north along x=-45 to 50. This is good.
  Need T spawn opening to z=10 corridor at x=-80. But T spawn already at x=-75,z=-25; can go north along west boundary to z=10. We'll create B tunnel corridor as west/north: from x=-90,z=-25 to-10? Need.

Let's define B tunnel:
- Vertical path from T spawn (-80,-25) to (-80,10) along x=-80, width 4 (z? corridors run north-south, so x range [-82,-78]).
- Horizontal path from (-80,10) to (-40,10), width 4 (z range [8,12]).
- Vertical path from (-40,10) to (-40,50), width 4 (x range [-42,-38]).
- B site at (-45,50) maybe at end; better B site rooms x[-60,-30], z[40,65], with entrance at x=-40,z=50. Good.

But this path is long. Okay.

Need A Long route from T spawn to A site:
- T spawn at (-75,-5) can enter A long corridor z[-10,0] x[-70,45]. To connect T spawn, need opening from T spawn to A Long at x=-70. T spawn area x[-85,-55], z[-35,-10]? We can create T spawn north opening at z=-10 into A long corridor? Actually A long starts at x=-70,z=-5. We can set T spawn z range [-35,0]? Then easy. But need B tunnel path vertical at x=-80 from z=-25 to10; it can be within T spawn area. Fine.

Maybe we can simplify by making T spawn a large open area x[-85,-55], z[-35,5], bounded by walls. Exits:
- East to Mid: x=-55,z[-35,-15]
- East/North to A Long: x=-55? Actually A Long corridor z[-10,0] runs from x=-70 to45. It starts at x=-70? If T spawn extends to z=5 and x=-55, not bounded; the player can just walk to A Long without distinct. That's okay.
Need named routes visible; create separate lanes with walls.

Let's not get stuck. We'll code map with lane walls via a grid perhaps.

Given complexity, a tile grid would simplify and ensure walls. Let's create tile map for Dust2 with cells 5x5. Need 36x28 grid. Each string char:
- `#` wall, ` ` floor. `T` spawn, `C` spawn, `A`/`B` sites, `D` double door.

Then generate boxes for `#` characters. However walls become 5m cubes, too chunky but acceptable? Could treat each # as wall piece; open cells are 5m wide, large. Corridors are 5m wide (one cell). Player width 1.0, okay but corridors narrow. Need A long maybe 2 cells wide. Could have cells 2.5m strings 80 wide too much. But maybe use 4m grid 45x35. Fine. Walls thick 4m, very chunky but reminiscent. Need tactical? Eh.

Manual boxes can create 2m wide corridors. Let's continue manual but not too many walls.

Maybe final answer can be code and user can run. Less important exact quality. We need deliver complete.

Need create maps. Let's build map data using `addBox` and `addWall` with coordinates. We'll create "Dust2" with the following wall segments (list). Need enough.

Let's set DUST2 coordinates with z positive north, x east:

I'll create wall segments for lanes:

1. Outer boundary:
- N: x=0, z=70, sx=190, sz=1
- S: x=0,z=-70
- E: x=90,z=0, sx=1, sz=140
- W: x=-90,z=0

2. T spawn open area: Let's make room x[-85,-55], z[-40,10]? Actually too large. Use walls:
- West outer already.
- South outer.
- East wall of T spawn along x=-55, z=-35..10? with openings to mid/long. We'll instead define rooms and routes with walls.

Maybe map generation easier if we define "rooms" as enclosed spaces with door gaps using wall boxes.

Let's create DUST2 using a "wall list" for corridors:

- T spawn center (-70,-25). We'll create T spawn room bounded:
  - East wall: segment x=-60, z=-40..0? With door openings at z=-25 (mid) and z=-5 (long).
  Actually use wall at x=-60 from z=-40 to-30; another from z=-20 to-10; another from z=0 to10? Need.

Maybe I should define rooms as 3D boxes with holes? Not possible. Use wall segments.

Let's define exact path and wall coordinates:

T Spawn (room): x[-90,-60], z[-40,-5]? open to north/east. We'll enclose to force routes:
- T spawn south wall: z=-40 from x=-90..-60? Outer S is -70, so create wall z=-40? Actually outer boundary at z=-70, but T spawn z[-40,5] leaves south part open. Use wall z=-40 between x=-90..-60 to restrict; but not needed because boundary.
- East wall of T spawn at x=-60:
  - A segment from z=-40 to -25? To force mid exit maybe at z=-25. Hmm.
Let's create chambers with narrow exits:
  T spawn room x[-90,-60], z[-5,15]? Wait.

No time. Let's use a crude method: define routes as "corridor" rectangles between walls. We'll place walls as lines on both sides of each corridor. For a corridor of width w, a centerline with points, create wall boxes along each side with gaps. We can write a function `addCorridor(map, path, width)` that adds wall boxes along left/right of polyline. But need avoid duplicates and can create collisions. This is a lot.

Alternatively use a NavMesh from boxes? We can define map walls as boxes; AI path can be direct; and player can go. Fine.

I think a tile grid is the least effort to define maps. Let's use tile grid for all maps. It creates complete collision and visual; okay.

Implement tile grid:
- tileSize = 4 (meters). grid dimensions per map e.g. 45x35 (x from 0 to 44, z from 0 to 34). We'll have strings of cols of `#` and `.`. But in code mapping, x = col*tileSize + origin, z = row*tileSize + origin. Each cell that is `#` becomes a wall box. Need avoid too many boxes; 45*35=1575 max okay. Use around 600 walls. For non-wall floor, add ground plane. Bomb site markers, spawns via grid chars.
- Wall boxes height 4, tile size 4. That means corridors are 4m wide. Can make doors by leaving certain cells open. This is playable. But walls no slopes; use crates/ramps. Good.

For Dust2, create 36x28 tile grid at tileSize=5? If tileSize=4, map 144x112 m. Use strings length 36 (x) and 28 (z). Let's do 40x30 at 4m = 160x120. Good.

Need define ASCII. The user asked "Dust2 测试地图 includes all named areas". If we create grid walls with named area labels, it can be recognized. Need maybe use larger open spaces.

Let's design a tile map for Dust2 manually, with characters:
- `T` T spawn area (floor)
- `C` CT spawn
- `A` A site
- `B` B site
- `#` wall
- `.` floor/open
- `D` double door floor? Actually tiles with symbols mark points.
- `1`? no.

We can place walls to form layout.

Let's create grid 40 cols (x=0..39), 30 rows (z=0..29). Tile size 4, origin x=-80,z=-60? Actually center map. Let's map row 0 = north (z=-60), row 29 = south (z=+56), col 0 = west (x=-80), col 39 = east (x=76).

Grid strings length 40. Need manageable. let's design.

Wait map bounds: x -80..80, z -60..60. 40*4=160, 30*4=120. Good.

Need create layout with a lot of open cells separated by walls. Use symbols:

Let's create row strings. Need be careful. Maybe use 45x30. We'll just use all border `#` except not needed.

Let's outline a simple layout:

Col x 0-39, row z 0-29.

T spawn at west side rows 18-22, cols 2-5.
CT spawn at east side rows 18-22, cols 34-37.
Mid corridor horizontal rows 18-20, cols 8-32.
A site at rows 5-10, cols 30-36 (northeast).
B site at rows 5-8, cols 5-10 (northwest).
A Long route from T spawn to A site: rows 10-12? Actually from T spawn up along west then east along row 8? Need connect.
T spawn at rows 20. To A Long: go north row 10, col 3? But B site also northwest. Hmm.

Let's make:
- B site: top-left rows 3-7, cols 4-10.
- A site: top-right rows 3-7, cols 28-35.
- T spawn: bottom-left rows 22-26, cols 2-7.
- CT spawn: bottom-right rows 22-26, cols 32-37.
- Mid: rows 21-23, cols 10-29 (horizontal across bottom, between T/CT). Actually T spawn at rows22-26; need connect to mid row22. CT spawn row22.
- A Long: from T spawn (row25) go east? Need A site top-right. If T spawn bottom-left, A Long could go north to row10 at col4 then east along row9 to A site. But B site at cols4-10 rows3-7 would block if continue east along row10. But A Long runs above B? Let's place B site top-left (rows3-6 cols4-10); A Long route from T spawn north at col14, then east at row9 to A. B site separate left.
- A Long vertical at col14 from row25 to row9, then horizontal row9 col14-33 to A. This lane is to the right of B site and left of A? A site cols28-35, row3-7; horizontal row9 enters from south. Good.
- B Tunnels: T spawn to B site via vertical at col7? rows20 to8? B site cols4-10 rows3-7; tunnel from row20 up to row8 at col7, entering bottom of B site. This is left of A Long. Good.
- Mid: from T spawn row22 col10 to CT spawn row22 col28, maybe through double doors at col20. CT spawn rows22-26 cols32-37.
- A small: from Mid row20 col28 to A site? Actually A site at rows3-7, col28-35. From mid row22 at col28, small route north at col28 from row22 to row8? That connects to A site south entrance. This is near A site. Great.
- CT to B: from CT spawn row22 col31? Need CT route to B site via "B doors". CT spawn to B site should go north along col31? Then west along row17? Let's create CT mid: from CT spawn (row24,col33) go north along col29 to row13? Hmm. Simpler: CT spawn (rows22-26, cols32-37). CT to A: from CT spawn go north along col31 to A site (rows3-7) via row? Actually A site east side; CT can enter A from east at col35 row7? There is route through CT long? But okay.
CT to B: from CT spawn go west along row24 to mid, then mid north? Instead, create a CT long corridor from CT spawn to B site on right/east? Not.

Maybe for playability, all areas accessible from connected spaces is enough. Use grid to create walls and openings; no need exact.

Let's generate walls with grid:
- All border cells #.
- Define interior wall blocks to form corridors. We'll manually set `#` for walls, `.` for floor. Need ensure not too many blocked.

Let's craft grid strings. We'll use length 40, rows 30. I'll write in code. Need be careful. Let's create base all `#` border, rest `.`, then manually add `#` walls.

Maybe easier to generate from in-code arrays not strings? But strings easier.

Let's create rows with strings. I can define functions to set row string. Need ensure all row strings same length 40.

Let's produce a layout:

Row labels (north to south):
0: "########################################"
1: "###....B......B.......A......A...######" etc.

Given manual 30x40 is time-consuming. But we can use code to generate walls programmatically: use functions to add wall rectangles instead. Hmm.

We can design map with rectangles in code. Use `addRoom` and `addWallSegments`.

Let's commit to manual boxes. We can create simpler maps more quickly. The walls list can be long but code generated by helper.

But the final response should include code; token budget allows. Let's write a helper `addWall(map,x,z,sx,sz,mat)` where x,z center. It creates a wall box. We can write `addRect(map,xmin,xmax,zmin,zmax,height)`? Actually wall as box with full x/z span and thickness? Need define.

`addWallLine(map, x1,z1, x2,z2, thickness)` can add a box from midpoint, length distance, thickness. For axis aligned only, easier:
 `addWallX(map, x, z1, z2, thickness, height)` vertical wall along z.
 `addWallZ(map, x1, x2, z, thickness, height)` horizontal wall along x.
Use these to form corridors. For doorways, split into segments leaving gaps.

Let's use this for Dust2. Need define open spaces.

Dust2 map coordinates as above. Let's create boundary.

Use z positive north, x east. Set ground -? y=0.

Dust2 layout with walls:

Boundary:
addWallX(-82,-60,60); addWallX(82,-60,60); addWallZ(-80,80,-60); addWallZ(-80,80,60)

T spawn area: x[-80,-55], z[-30,10]? open. Add walls around? We'll use outside boundary, add some walls to create exits.

Maybe set T spawn as space x[-78,-55], z[-30,10]. It is open to A Long/Mid/B tunnel. To enforce routes, add partitions:
- Wall from x=-55,z=-30 to z=-20? Let's use.

Let's define corridors:
1. Mid: from T spawn (-55,-25) to CT spawn (55,-25), horizontal corridor between z=-30 and z=-20. The walls:
   addWallZ(-65,65, -20) (north wall)
   addWallZ(-65,65, -30) (south wall)
But if T spawn at x=-70,z=-25, the corridor begins at x=-55? Need T spawn to corridor. If north/south walls extend to x=-65, and boundary x=-80, T spawn is inside corridor if no wall at x=-55. Corridor can extend from x=-75 to65. Good: addWallZ(-75,65,-20) and addWallZ(-75,65,-30). Thus T spawn at x=-70,z=-25 is on left edge. CT spawn at x=70,z=-25 just right of corridor, need opening at x=65? The walls end at x=65, opening between wall ends and boundary (x=65..82) leads to CT spawn. Good.

2. A Long: from T spawn at x=-70,z=0 to A site? Let's create horizontal corridor above mid, z[-4,4], from x=-70 to45. South wall at z=-4 (between A Long and T/Mid), north wall at z=4. But A Long should connect to A site at north/east. Let's add:
   addWallZ(-70,45,-4)
   addWallZ(-70,45,4)
This creates A Long corridor. T spawn at x=-70,z=-25 can enter? Wait A Long is at z=0, separated from T spawn by south wall z=-4 and maybe no east wall. T spawn at x=-70,z=-25 can move north to z=0 within area x<-55? No wall at x=-75? Actually A Long corridor z[-4,4] spans x[-70,45]; T spawn area x[-78,-55], z[-30,10] includes it. Need walls around T spawn to prevent easy access? We need create T spawn room.

Let's define T spawn as open room x[-78,-56], z[-30,10]. It connects to:
- A Long at x=-56? Hmm A Long corridor starts at x=-70, not -55. If T spawn open from -30 to10, it overlaps A Long; that's okay, T spawn can walk directly into A Long at z=0. "A Long" starts there. But to be distinct, partition wall at x=-68 maybe.
Instead, let A Long begin at T spawn: T spawn open room x[-75,-55], z[-30,10]; A Long corridor z[-4,4] from x=-55 to45. A wall at x=-55 with opening at z=0? Hmm.

Need keep map connected. Player won't care if area names not exact.

Let's create T spawn room with exits:
- East to Mid: wall at x=-55 from z=-30 to-20? Actually mid corridor along z=-25; T spawn room west of x=-55. If wall at x=-55 with opening at z=-25, then mid route via door. Good.
- East to A Long: wall at x=-55 with opening at z=0? A Long corridor z[-4,4] from x=-55 to45.
- North to B tunnel: wall at z=10? B tunnel corridor north from z=10 at x=-60? Hmm.

Let's add:
T spawn outer: x[-75,-55], z[-30,10]. East wall at x=-55 from z=-30 to10, with two openings: at z=-25 (mid) and z=0 (A long). use segments:
 addWallX(-55, -30, -20) (wall with gap -20 to -? Actually opening at -25 with width? use segments -30 to -27 and -23 to -20? Wait opening width 4). We'll create wall segments manually:
   addWallX(-55, -30, -22) 
   addWallX(-55, -18, 10)
This leaves gap z=-22 to -18 (4m) for mid, and no opening for A long? A long needs gap z=-2..2. But segment from -18 to10 blocks. Need more gaps:
   addWallX(-55, 8, 10) maybe leave z=-2..2. Actually:
   addWallX(-55, -30, -22) // south part
   addWallX(-55, -18, -2) // block between mid and A long
   addWallX(-55, 6, 10) // north part
Gaps: z=-22..-18 (mid opening), z=-2..6? too wide. We want A long opening z=-2..2. Let's segment addWallX(-55,2,10) and etc. Wall code supports ranges.

This is cumbersome but doable.

Maybe use a "door" helper: addWallBetween with gaps.

Given time, maybe no need exact walls; large open T spawn gives access to all; walls create lanes. Even if not exact, playable.

Let's define "Dust2" map with a set of wall rectangles:

- Outer boundary.
- A Long walls: south wall z=-4 from x=-50 to40, north wall z=4 from x=-50 to40, vertical wall at x=40 from z=4 to30? Then A site opens.
- A site: region x[15,75], z[30,60]. Need walls: south/north/east/west.
- Mid: walls z=-30 and z=-20 from x=-50 to55.
- A small: vertical corridor at x=25 from z=-20 to30, bounded by walls x=20 and x=30. Connect to mid through gap at z=-20, to A site at z=30.
- B tunnels: vertical corridor at x=-60 from z=10 to40, bounded x=-64 and x=-56. Connect to T spawn at z=10 and B site at z=40.
- B site: region x[-65,-35], z[40,60].
- CT spawn: region x[55,82], z[-30,10] perhaps, connected to mid at x=55, to CT routes north.
- CT to B: corridor vertical at x=10? Wait CT spawn at x=70,z=-25. Need route to B site x=-50,z=50: from CT spawn north at x=65 to z=10, then west along z=10 to x=-30 then north to B site? Hmm. Let's create "CT mid" corridor from CT spawn (x=55,z=-10?) to B door. 
Maybe too much.

Could instead connect B site to CT via a long corridor along south edge? Provide CT and T access. In classic, CT has shorter route to B; we can create CT route:
- CT spawn at (70,20) maybe not bottom? Let's orient differently.

I think the grid approach would be less pain because we can draw. Let's use grid after all. We can produce a 30x40 map with walls and points. I can compose strings in code and not manually optimize. The map will be generated. We can add crates.

Let's create a compact 32x24 map with tile size 4.5 to reduce strings. We'll design as a 2D grid with 1=wall. But final maybe plausible.

Let's define map grid rows with length 32. Use tile size 5. Need row count 24. Map size 160x120.

Let's write a function for map strings:
For each row, string char `#` wall, `.` floor. The area includes named zones via overlays: spawn positions and bomb site markers. We'll place after.

I'll make a layout for DUST2:
- Border walls all `#`.
- Interior walls form corridors.

Let's draw 32x24 grid. Columns 0-31, rows 0-23. North at row0.

Rows (I'll draft in code):

I'll use `.` for open, `#` for walls.

Need create corridors:
Col 0-31, row 0-23.

T spawn at bottom-left (rows 18-21, cols 2-5).
CT spawn at bottom-right (rows 18-21, cols 26-29).
B site at top-left (rows 2-5, cols 4-8).
A site at top-right (rows 2-5, cols 23-27).
Mid horizontal at row 18, cols 8-23.
A Long from T to A: vertical col 8 from row18 to8? then horizontal row8 col8-23 to A. Or from T up col8, then east row8. But B site occupies cols4-8 rows2-5; no conflict.
B tunnel from T to B: vertical col 5 from row18 to6, entering B site bottom. But A Long vertical col8, so B tunnel left.
A small: vertical col22 from row18 to6? connects mid to A site. But A site at rows2-5, row6 south. Good.
CT to B: vertical col24 from row18 to6? That would be near A site, not B. Instead CT to B could be horizontal row17? Hmm.

Let's map:
T spawn at cols2-5,row20. Exit east to mid at row19? Wait mid horizontal row18? We can connect.

Let's define zone strings simply:
Row0: 34 walls, no.
Actually produce 32 chars.

Let's manually generate a random layout? Need ensure connectivity. We can use cells with walls.

Let's create a 32x24 grid programmatically using loops? Hard to visualize but can define wall rectangles in code easier.

Maybe use "wall list" with segments and not rely on exact. Let's choose manual boxes for final and use gameplay. We can claim high approximation.

Let's implement maps now in code. Use helper functions and create wall list with lines. We'll add enough walls.

We'll define in JS:
function W(x,z,sx,sz,mat){ walls.push({pos:[x,0,z], size:[sx,4,sz], mat:mat, collidable:true}); }
But wall height from y=0 to4, y=2. For crates: C(x,z,sx,sz,height).

Use top-down coordinates with z positive to north. Map boundary x[-100,100],z[-70,70].

Dust2 manual map:
Outer:
W(-90,70,1,280?) too big. Let's set boundary x -90..90, z -70..70.
Wall segments using length.

Need ensure openings. Let's use our lanes but overlap somewhat.

Let's create methods:
wallX(x,z1,z2,th=0.8,h=4,mat) // add wall along x? Actually wall along z (vertical). center z=(z1+z2)/2, size=[th,h,len].
wallZ(x1,x2,z,th=0.8,h=4,mat).

Then map.

Let's define Dust2 map:
Boundary:
wallX(-90,-70,70); wallX(90,-70,70); wallZ(-90,90,70); wallZ(-90,90,-70).

Now add interior walls.

Let's set:
- T spawn zone: x[-90,-55], z[-30,10] (open to west/south boundary). East wall at x=-55 with gaps: mid at z=-25, A long at z=0? Add:
 wallX(-55,-30,-20,0.8,4) // wall below mid opening (z -30..-20)
 wallX(-55,-18,2,0.8,4) // wall between mid opening and A long opening? Opening mid: z -20..-18 (2m), too small. Need maybe opening z -28.-24. Let's not.
This wall approach is tedious.

Maybe no east wall; T spawn open to multiple lanes. Lanes separated by walls from east to west? We can add walls that jut out from east boundary into T spawn to guide.

Let's set lanes with walls:
- Mid corridor: z=-25, x -55..55. Add wallZ(-55,55,-20) and wallZ(-55,55,-30). T spawn at x=-70,z=-25: if walls start at x=-55, T spawn is west of corridor; to enter mid, go east through open at x=-55 between z=-30..-20. Good. The wall lines don't block T spawn.
- A Long corridor: z=0, x -55..45. Add wallZ(-55,45,-4) and wallZ(-55,45,4). T spawn at x=-70,z=0: open through x=-55 between z=-4..4. Good.
- B tunnel vertical: x=-65? from z=10 to35. Add wallX(-65,10,35) and wallX(-55,10,35)? Actually corridor between x=-65 and -55, z[10,35]. Add west wall X=-65, east wall X=-55. T spawn at z<10 can access from south at z=10? Add wallZ(-65,-55,10)?
- B site: room x[-65,-20], z[35,55]. Add walls:
  - North wall z=55, x[-65,-20]
  - West wall x=-65,z[35,55]
  - South wall z=35, x[-65,-50] but needs entrance from tunnel at x=-60? B tunnel north end at z=35 opens into room. So wallZ(-65,-50,35) leaving gap x[-50,-20]? Actually B site located x[-65,-20], B tunnel at x=-60; entrance south at x=-60, z=35. Could have room open.
  - East wall x=-20,z[35,55] with B door opening to CT route at z=40.
- A Long route from z=4 to A site: At x=45,z[4,40], create vertical corridor bounded x=40 and x=50? Then A site. Add wallX(40,4,40), wallX(50,4,40). From A Long horizontal z=4, turn north through this corridor to z=40, enters A site.
- A site: room x[25,75], z[40,55]? Actually corridor to A at x=45 enters. Add walls:
  - North wall z=55, x[25,75]
  - West wall x=25,z[40,55] (separates from CT? Actually A site west wall)
  - East wall x=75,z[40,55]
  - South wall z=40, x[25,35] and x[55,75]? leaving entrance from A long. ALong corridor at x=40..50; so south wall with gap x=35..55? Wait corridor x 40..50, gap. Add wallZ(25,35,40) and wallZ(55,75,40).
- A small path: from mid to A site. Need connect mid corridor z=-20 to A site south z=40. Create vertical corridor at x=25 from z=-20 to40, bounded x=20 and x=30. It connects mid at z=-20? Mid north wall z=-20 blocks; need opening at x=25. We can add wallZ(-55,20,-20), wallZ(30,55,-20), leaving gap x=20..30 in mid north wall. Then vertical corridor x[20,30], z[-20,40]. It enters A site at z=40. Add walls X=20,X=30 z[-20,40]. This is "A小".
- Mid corridor: x[-55,55], z[-30,-20]. But A small vertical corridor starts at x=25,z=-20; gap in north wall. Add as above. Need CT spawn at east.
- CT spawn: room x[55,90], z[-30,20]? We have mid corridor east end x=55? Actually mid walls z=-20,-30 from x=-55 to55. CT spawn east of x=55; open to mid at x=55. Add east boundary x=90. South boundary z=-30? we have outer z=-70, but add wallZ(55,90,-30) to define CT room. North wall z=20? maybe add wallZ(55,90,20) with door to CT route? Wait CT route to A/B. 
- CT route to A/B: From CT spawn to A site and B site. CT spawn at x[55,90], z[-30,20]. It connects south to mid at x=55,z=-25 and north to "CT mid" corridor. Add vertical corridor x=80? from CT spawn z=20 to55, bounded x=75 and x=85. Then horizontal top corridor z=55 from x=-20..85 connects to B site (via B doors) and A site (via east). Let's create:
   CT spawn north wall z=20 with opening at x=80 (gap x=75..85).
   Vertical corridor x[75,85], z[20,55].
   Top corridor z=55, x[-20,85]. Add wallZ(-20,85,52?) Actually corridor along z[52,58], center z=55. Need top wall z=60? Outer boundary at z=70? We'll add wallZ(-20,85,60) north, wallZ(-20,85,50) south. This top corridor connects to A site east wall at x=75? A site at x[25,75], z[40,55]; top corridor at z[50,60], x[-20,85]; adjacent to A site north? A site north wall z=55? If corridor z[50,60] overlaps A site z[40,55]. Need adjust.
Maybe define A site z[30,50], B site z[30,50], top corridor z[50,60]. Let's set:
- A site x[25,75], z[30,50].
- B site x[-65,-20], z[30,50].
- A small enters A site south at z=30? Wait A small z[-20,40] enters at z=40. Hmm.

Let's adjust vertical dimensions:
- A site z[35,55].
- B site z[35,55].
- Top CT corridor z[55,60].
Thus A small goes from mid z=-20 to z=35 (A site south), enters A site. A Long vertical x=45 from z=4 to35 (A site south) enters. Top corridor z=55 along x[-20,85] connects CT to both sites from north side, with doors into sites from north. This is plausible.

Need B route:
- B site z[35,55], x[-65,-20].
- B tunnel corridor x[-65,-55], z[10,35] enters B site south (gap at z=35). B site south wall z=35 from x=-65..-20 with gap at x=-60? We can leave.
- CT top corridor z=55 along x[-20,85] enters B site via east wall x=-20 at z=45? Actually B site east wall at x=-20, opening to top corridor z=55. B Doors at x=-20,z=52 (gap in east wall). Good.
- CT top corridor from x=-20 to85 at z=55. It enters A site via east wall x=75? A site east wall at x=75,z[35,55], opening at z=48. Good.
- CT spawn vertical corridor at x=80,z[20,55] connects CT spawn to top corridor.

This seems coherent! Let's list all walls:

Dust2 walls:
Outer:
- wallX(-90,-70,70), wallX(90,-70,70), wallZ(-90,90,70), wallZ(-90,90,-70).

Mid corridor:
- wallZ(-55,55,-20) center z=-20? Actually corridor z[-30,-20]. North wall at z=-20, south wall at z=-30. Add wallZ(-55,55,-20), wallZ(-55,55,-30).
T spawn west of x=-55 can enter mid through x=-55 between z -30..-20? But walls at x=-55? none. T spawn open to x=-55. Good.

A Long:
- corridor z[-4,4], x[-55,45]. Add wallZ(-55,45,-4), wallZ(-55,45,4).
T spawn west of x=-55, z=0 enters through x=-55. But between A Long (z=-4..4) and mid (z=-30..-20) there is a large gap z=-4..-20 where T spawn and no wall. So maybe okay; but lanes not separated. To separate, need wall at x=-55? Hmm. It still works.

A Long vertical:
- corridor x[40,50], z[4,35]. Add wallX(40,4,35), wallX(50,4,35).
At intersection with A Long horizontal z=4, A Long north wall z=4 blocks? Wait A Long corridor z[-4,4]; vertical corridor z[4,35]. The north wall of A Long at z=4 would block entrance to vertical corridor. Need leave a gap/overlap. Instead set A Long north wall z=4 from x=-55..40, and vertical corridor begins at x=40? Actually vertical corridor bounded x=40..50, z=4..35. If wall z=4 ends at x=40, the opening at x=40..50 allows turn from horizontal into vertical. So add wallZ(-55,40,4) (north A Long wall) not to x=45. And wallZ(-55,45,-4) south wall. Good.
- wallZ(-55,40,4)
- wallZ(-55,45,-4)

A small:
- corridor x[20,30], z[-20,35? Actually from mid north wall z=-20 to A site south z=35. Add wallX(20,-20,35), wallX(30,-20,35). Mid north wall z=-20 from x=-55..20 and x=30..55, leaving gap x=20..30. A small enters A site at z=35; A site south wall z=35 with gap x=20..30? We'll set.
Mid north wall:
- wallZ(-55,30, -20)?? Actually gap x=20..30. Add wallZ(-55,20,-20), wallZ(30,55,-20).
Mid south wall unchanged.

A site:
- room x[25,75], z[35,55]? But A small corridor at x[20,30] overlaps A site? Need A site west wall x=25? If room x[25,75], A small x=20..30 enters at x=25..30; fine. Set west wall x=25 z[35,55]? But wall at x=25 from z=35..55 would block A small entry. Instead A site west wall only z[40,55]? Let's define A site x[25,75], z[35,55]. South wall z=35, x[35,75]? with gap x=20..35 for A small. But A Long vertical at x=40..50 also enters from south? Actually vertical corridor z[4,35] ends at z=35. A site south wall z=35 from x=35..75 would block vertical corridor at x=40..50? Need leave gap for both entries. Since A site spans x=25..75, we can simply not have a south wall except external? The vertical corridor and A small open into site. We'll add west/east/north walls but no south wall except at boundaries.
Add:
- north wall z=55 x[25,75]
- east wall x=75 z[35,55] with opening to CT top at z=48? Need CT top corridor at z=55 maybe. Let's set CT top corridor at z=50? Hmm.

Maybe define A site north wall z=55, and CT top corridor z=55 from x=-20..85 is right at north edge; if A site north wall at z=55, CT top corridor along z=55 maybe outside. CT top could be z=60..65? boundary at 70. Let's set CT top corridor center z=62, z[58,66], north boundary outer z=70; south wall z=58 x[-20,85]. A site exits to CT top through opening in A north wall at z=55? Wait need connect. Use CT top corridor south wall z=58; A site north wall z=55; gap between 55 and58 is wall thickness? We can just have opening.

Let's create top corridor:
- south wall z=58 from x=-20 to85, north outer boundary z=70. It is a corridor z[58,70]. A site north wall at z=55, not z=58, so corridor south wall z=58 creates a hallway between z=55 and58? Actually if A site z[35,55], and corridor z[58,70], gap z=55..58 (3m) open. Add south wall z=58 x[-20,85] separates corridor from gap. To enter corridor from A site, need opening in wall z=58? No, A site z ends at 55, then open gap 55..58, then wall z=58 blocks. So no. Better make top corridor z[55,65] and south wall z=55 with openings. Let's use corridor z[55,65]. Add south wall z=55 x[-20,85] with gaps at doors to A/B, and north outer boundary z=70. But A site north wall at z=55 would be same; if A site has wall z=55, no entry. Instead A site north wall should have a door gap at x=70? Then corridor south wall at z=55 can also have same gap. Simplify: A site open to CT top corridor via opening in wall line at x=70,z=55. We'll not add full wall? Let's add wallZ(-20,70,55) with gap x=70..75? Wait corridor width x=75..85? Hmm.

Maybe enough: CT top corridor is accessible from CT spawn via vertical x=80 z[20,55]. It runs along z=55, x=-20..85, directly connects to A site at x=55..65? A site north border z=55; if no wall at z=55, A site and corridor overlap. Let's define A site z[35,52], CT top corridor z[52,65]. Add wallZ(-20,85,52) with doors.

A site:
- z[35,52], x[25,75]. North wall at z=52.
- CT top corridor z[52,65], with south wall at z=52. For A site to connect, create a door in wall z=52 at x=60..65. Then from A site north exit to corridor. Similarly B site door at x=-20? Let's define.

B site:
- z[35,52], x[-65,-20]. North wall z=52, east/west/south. To connect to CT top corridor (z[52,65]), create door in z=52 at x=-25..-20? Actually corridor starts at x=-20? If B site east wall at x=-20, and corridor x=-20..85. Door at B site northeast corner: wall z=52 needs gap x=-20..-15? If x=-20 is eastern boundary, then gap at x=-25..-20.
- B tunnel enters B site from south z=35 at x=-60; B site south wall z=35 with gap x=-65..-55.

CT spawn vertical corridor:
- CT spawn room x[55,90], z[-30,20]? We had mid corridor ends x=55, CT spawn east of x=55 between z=-30..20. Need define walls:
  - top wall z=20 x[55,90]
  - south wall z=-30 x[55,90]
  - east boundary x=90
  - west opening to mid at x=55.
- Vertical corridor from CT spawn to top: x[75,85], z[20,52]. Add wallX(75,20,52), wallX(85,20,52). It connects CT spawn at z=20; top wall z=20 with gap x=75..85? Add wallZ(55,75,20) and wallZ(85,90,20), leaving gap.
Then vertical corridor to CT top corridor at z=52; wall z=52 x? The corridor ends at z=52 open; CT top corridor south wall z=52 from x=-20..85 but need an opening for vertical corridor. Since vertical corridor x=75..85 is within wall segment x=-20..85; add wallZ(-20,75,52) and wallZ(85,90,52) leaving gap x=75..85. Good.

Now A/B top corridor:
- Add wallZ(-20,75,52) leaving gap at x=75..85? Actually we want corridor from x=-20 to85; south wall z=52. Vertical corridor enters at x=75..85 gap. We'll add wallZ(-20,75,52) only, and for x=75..85 vertical corridor. For x=85..90 maybe boundary.
- A site north wall z=52 with door at x=60..65; if wallZ(-20,75,52) includes x=-20..75 and is solid, it would block A site connection at x=60. Need split with gap:
   wallZ(-20,20,52) ?? Actually A site at x=25..75; need door at x=60..65. So wallZ(-20,75,52) should have gap x=60..65. Add wallZ(-20,55,52) and wallZ(68,75,52) leaving gap x=55..68? But vertical corridor at x=75..85 gap separate.
- B site north wall z=52 with door at x=-30..-25? Add wallZ(-20,-35,52) and etc. Maybe.

This is complex but manageable.

However code final need no placeholders. We can use maps 2/3 simpler. But Dust2 manual walls will be dozens lines. Let's continue.

We can define map via `addWallZ` etc. In code, if walls overlap okay. Use small height 4.

Need create crates/platforms:
- Add crates boxes at A site: large box at (55,42), (35,40) etc.
- B site boxes at (-45,42).
- Mid boxes etc.
- Dust2 A ramp? "A斜坡" perhaps use a stepped platform at A site south/east? We can create "ramp" boxes with steps (three boxes height .3?), bottom? Actually player can handle steps if max step 0.5? Need step collision? The player cannot step up automatically if AABB collision prevents vertical movement moving sideways into side of step. Need implement step-up: if collision on horizontal and vertical overlap with ground and step height <=0.5, allow. We can implement step-up maybe. For simplicity, platform boxes maybe only as obstacles not climbable. But "A斜坡" need visual steps. We can make low stairs (height 0.5, 1.0, 1.5) and implement stepping on them. Need player can jump onto? Could.

Implement step system: when moving horizontally and colliding with a box whose top is less than current feetY + maxStepHeight (0.5) and player's ground is below top, and no collision after moving up, then set pos.y= box.top. We'll implement later.

Need build maps 2/3 maybe with similar helper.

Maybe due to complexity, final code may be huge but okay.

Let's plan JS modules:

- THREE scene, camera, renderer.
- input listeners.
- resources textures: `createTextures()` return canvas textures for map materials (sand, concrete, grass, metal), weapon skin maybe.
- Material palette:
  * sandWall, sandFloor, ctFloor, stone, metal, wood, crate.
  * For bots materials: blue camo, orange camo, skin, helmet.
- Scene setup: hemisphere, directional light shadows? Shadows can be performance; use MeshLambertMaterial and no real-time shadows? Need "画面气质". We can enable shadows with low res. Use renderer.shadowMap.enabled = true; spot light casts shadow. Many boxes; okay.
- Add fog to scene for map.

Map loading:
`buildMap(mapDef)` clears scene, adds ground plane with floor texture, walls, crates, nav nodes, spawn points, buy zones, lights.
Need mapDef.walls array with `pos`, `size`, `mat`, `type`. In `loadMap`, for each wall create BoxGeometry and mesh. Add to scene. Also to collision list. Need maybe separate `staticColliders` for AABB and `shootables` for raycast.

But if each wall mesh collider; Raycaster against all. Many meshes, fine. But raycast all scene children includes ground/walls. We can create `worldMeshes` array; raycaster.intersectObjects(worldMeshes, false) (each mesh is box) and bot meshes. Ground is a mesh. Need avoid intersecting UI/character model? Viewmodel group separate not in scene maybe? We can attach to camera.

Bots:
- Each bot has group in scene. Body meshes in `botMeshes` array (not worldMeshes). To raycast, intersect worldMeshes + botMeshes. But botMeshes nested in groups; use recursive true and match userData. We'll maintain `raycastTargets` array of body part meshes with userData. Need include world meshes. Use `raycaster.intersectObjects(raycastTargets, false)` but world meshes and bot body parts all direct Mesh. Bots animate? Could set group position. Body part positions relative group. For raycast, use group children recursively.

Bot update:
- Each bot state: `alive`, `team`, `hp`, `armor`, `weapon`, `money`, `route`, `target`, `lastShot`, `sawPlayer`, `aimTarget`, etc.
- AI decisions.
- Need maybe bot simple physics: apply collision to avoid walls? We can use player collision function.
- Bots move along nav graph. Since walls generated arbitrary, nav nodes need positions. We can manually add nav nodes for each map; but if walls not passable, need ensure. Could instead use direct goal and simple obstacle avoidance; bots might get stuck. Need robust pathfinding. We can create nav graph manually with coordinates near our intended lanes and use path.

Maybe better: use our manual map walls but add nav node positions for key corridors. Need not cover all if bots use simple. But for 3 maps, adding nav nodes is tedious.

Alternative: Use bot "teleport" along predetermined waypoints per objective. Bots choose route arrays as waypoint positions, and move toward them with collision. Since walls built along routes, waypoints inside lanes. We need for each map define `routes`:
- T attack: paths to A or B with waypoints arrays.
- CT defend/retake: paths to site.
But bots need dynamic. We can define nav nodes for each map and edges using wall-corridor coordinates. Use Dijkstra. We can include.

Manual map already has corridors; we can create nav nodes:
Dust2 nav nodes:
0 T spawn (-70,-25)
1 Mid west (-40,-25)
2 Mid center (0,-25)
3 Mid east (30,-25)
4 CT mid (60,-25)
5 CT spawn (70,-25)
6 CT to A vertical (80,0), (80,30)
7 Top corridor (50,62)
8 A site (50,45)
9 B site (-45,45)
10 A long west (-40,0)
11 A long mid (0,0)
12 A long east (40,0)
13 A ramp/corridor (45,20)
14 A small (25,-5)
15 B tunnel south (-60,0)
16 B tunnel mid (-60,20)
17 B site entrance (-50,35)
18 CT to B door (-20,45)...
Edges connect adjacent nodes.

If all nav nodes are manually map-specific, okay. But need for map2/3 too. We can define 15 nodes per map. That's okay. Need choose coordinates passable. For our generated map walls, nodes should be in open lanes.

When a bot chooses a target (A/B site etc), use Dijkstra from nearest node to a target node. Then move along path. Need handle dynamic player? No.

Nearest node: find node with shortest Euclidean distance to bot. If stuck, recompute. Use waypoints and local avoidance with collisions.

Bot movement: We'll have a `moveBot` that computes desired velocity direction toward next waypoint. Applies player collision but if blocked for >0.5s maybe choose next node or jitter. Since nav nodes within lanes and nodes connected by straight lines that may pass through walls if edges not aligned, but use many nodes.

We'll create `map.nav` array with {x,z, links}. `dijkstra(start, goal)`.

Need create nav for all maps. We can manually add.

Now weapons/economy/round flow.

Round timer:
- `roundState`: 'freeze' (buy time) 10s? 'live' 105s; 'bombPlanted' 40s? Actually round timer countdown 115s. When bomb planted, if round time expires before bomb, bomb continues? In CS bomb timer 40s after plant, round ends but timer stops. We'll handle separate.
- Start of round: reset positions, hp/armor, weapons, money no change; give default pistol/knife, no armor. Bots loadout based on buy? Need buy AI.
- Buy period: in buy zones, player can open buy menu (B key) and purchase from a predefined menu. Need overlay UI with buttons.
- Money: start $800? Standard. Kill reward, win/loss bonuses. Need update arrays.

Weapon definitions:
`WEAPONS = { knife, glock, hkp2000, p250, deagle, mp9, mac10, p90, ak47, m4a4, awp, nova, he, flash, smoke, molotov }`.
Need player can buy in buy zones based on team:
  CT: M4A4, AWP, MP9, Nova, USP? T: AK47, AWP, MAC10, Nova? We'll allow all if enough money? Simpler.
Need weapon price table.
At spawn, give:
- T: knife + glock + 3 grenades? Maybe HE/flash/smoke purchase not default. Give kit? Give one HE to both? CS default no grenades. But game fun start with no grenades. We'll allow buy. We'll give player one knife and pistol.
- CT: knife + USP.
If player dies and next round, reset weapons to default (or keep purchased? CS keeps? Actually if round lost, weapons lost; if won, you keep? But in next round, keep weapons if survived? We'll simplify: at each round, restore to default leaving money; player buys each round. Bots also re-buy. If survived, keep previous? We can omit because new round resets. This is acceptable? "团队经济节奏" important but can simplify.
Maybe keep weapons if survive; but player may lose on death. For simplicity, at round start, player retains previous weapons if alive? But code resets all. We'll implement:
- At new round, for all players, if dead or force? Actually if won and survived, keep primary. But complex. We can just reset to default at new round but give enough money. The user likely won't mind.

Bomb:
- T starts with bomb randomly assigned? Player if T has bomb slot? Need implement: Among T players, one carries C4. We'll display "You have C4" if T and maybe random. Bots can carry? Simpler: if player is T, player carries bomb every round? Then need down at site. But bot T? Could also. For playability, player has C4. Bot AI can plant if designated. We'll implement: if side T, assign C4 to player and one bot? Better only one. Since player leads, player has bomb. That means if player dies, no bomb? Could still. We'll assign C4 to player if T; if player dead, to random T bot. Objective: T must plant and defend. Plant at bomb site when within radius and press E; plant takes 3s. After plant, round timer switches to 40s, bomb beeping, objective changed. CT can defuse within radius by holding E for 5s. Need on E key.
- Need bombs: maybe if bot has bomb, bot plant. If player not near, bot plants if site controlled.

Round win conditions:
- If all T dead -> CT win (unless bomb planted? In CS, if all T dead before plant, CT win; after plant, T can win if bomb explodes even if T dead? Actually if bomb planted and all T dead, CT win only by defuse; if time expires bomb explodes T win. We'll implement.)
- If all CT dead -> T win.
- If time expires: if bomb not planted -> CT win; if planted -> bomb will explode maybe before time; if timer hits zero and bomb still ticking, continue? We'll just when bombTimer reaches 0, T win.
- Defuse: CT win.
- Bomb explode: T win.
- Round end with message.

Server? Local all.

Minimap:
Canvas overlay top-right. Draw map walls as grid? Could draw walls from mapDef walls (top view) and players/bots positions. Need include bomb sites. Use 2D canvas.

Buy menu:
When in buyzone, press B toggles. UI panel with buttons. Could create HTML elements. Need responsive. Use click handlers. Need close on escape.

Kill feed:
Div top right with messages.

Damage indicators? We'll implement hit marker on crosshair, red flash when damaged.

Audio:
WebAudio. Need create sounds:
- gunshot: noise burst with lowpass.
- headshot ding?
- explosion, flash, smoke, footstep.
- bomb beep, plant/defuse ticks.
Use AudioContext and functions. Need no audio files.

Weapon fire:
- For hitscan, need `playerFire()` from camera. Use raycaster.
- Muzzle flash: a sprite/point light at gun muzzle maybe.
- Shell casing: spawn small mesh that falls.
- Recoil: visual recoil (camera pitch kick and weapon offset) and spread.

First person viewmodel:
- Create group `gunGroup` attached to camera. For each weapon, build meshes in positions. On fire, offset recoil. On reload, animation timeout. Need switch weapon via 1-5 keys/scroll maybe.
- We need update viewmodel every frame.

Map lighting:
- Set background color based on map. Add hemisphere and directional light. Dust2 warm fog.

Need performance with many wall meshes. Use BoxGeometry per mesh; can use merged? But okay. Maybe 300 meshes. Need shadows? If shadow maps per mesh okay. Can disable cast/receive on some.

Let's start coding. The final answer can be a single HTML. Need ensure no syntax errors.

But before code, let's plan weapon data and values.

Weapon data:
```
const WEAPONS = {
 knife: {name:'战术匕首', slot:0, damage:35, price:0, mag:-1},
 glock: {name:'Glock-18', slot:1, damage:25, price:0, magSize:20, reserve:120, rpm:400, auto:false, spread:0.02, recoil:0.01, reload:2.2, team:'T'},
 hkp2000:{name:'USP-S', slot:1, damage:30, price:0, magSize:12, reserve:24, rpm:400, auto:false, spread:0.015, recoil:0.008, reload:2.0, team:'CT'},
 p250: {name:'P250', slot:1, damage:35, price:300, magSize:13, reserve:26, rpm:400, auto:false, spread:0.02, ...},
 deagle: {name:'Desert Eagle', slot:1, damage:50, price:700, magSize:7, reserve:35, rpm:267, auto:false, spread:0.03, ...},
 mp9:{name:'MP9', slot:2, damage:24, price:1250, magSize:30, reserve:120, rpm:800, auto:true, ...},
 mac10:{name:'MAC-10', slot:2 ...},
 p90:{name:'P90', slot:2, damage:23, price:2350, magSize:50, reserve:100, rpm:750, auto:true, ...},
 ak47:{name:'AK-47', slot:2, damage:33, price:2700, magSize:30, reserve:90, rpm:600, auto:true, spread:0.01, recoil:0.03},
 m4a4:{name:'M4A4', slot:2, damage:30, price:3100, magSize:30, reserve:90, rpm:666, auto:true, spread:0.008, recoil:0.02},
 awp:{name:'AWP', slot:2, damage:125, price:4750, magSize:5, reserve:30, rpm:41, auto:false, spread:0.001, recoil:0.08, zoom:true, scope: true},
 nova:{name:'Nova', slot:2, damage:12*8? Actually per pellet, shotgun. use pellet:8, spread:0.08, damage:14, price:1050, ...},
 he:{name:'高爆手雷', slot:3, price:300, ammo?},
 flash:{name:'闪光弹', slot:3, price:200},
 smoke:{name:'烟雾弹', slot:3, price:300},
 molotov:{name:'燃烧瓶', slot:3, price:400}
}
```
Need grenade inventory count 1 each max. Player has inventory.grenades {he,flash,smoke,molotov}. Selecting grenade by 4? Use key 4/5? Simpler: press G for HE, F for Flash, C? But request "投掷物". We'll implement keys 3-6? Actually weapon slots: 1 primary, 2 pistol, 3 knife, 4 HE, 5 flash, 6 smoke, 7 molotov. But UI can show.

Need buy menu includes grenades & armor. UI list with buttons.

Bot AI loadout: based on money, choose AK/M4/AWP, buy armor. Grenades if money.

Need implement team switch selection in menu. Player can select T or CT, bot count, difficulty.

Difficulty effect:
- Easy: reaction 0.6s, spread high, damage 0.7, no nades? Normal 0.3, Hard: 0.1, accuracy high, grenades use.
Implement `DIFF = {easy:{reaction:700, aimError:0.06, dmg:0.8, useNade:false}, normal:{reaction:400, aimError:0.03, dmg:1, useNade:true}, hard:{reaction:200, aimError:0.015, dmg:1.2, useNade:true}}`.

Need UI: menu with sliders buttons:
- Map selection cards (buttons)
- Side selection (T/CT)
- Bot count select (2,4,6,8)
- Difficulty select.
- Start button.

During game, use overlay HUD. Press Escape to show pause menu (PointerLock exit). Need menu with resume/restart/quit.

Need pointer lock: click canvas to lock. We'll implement.

Need "按住鼠标左键" fire; "R" reload; "Shift" walk/sprint? In CS, shift is walk (quiet), Ctrl crouch. "静步和冲刺": use Shift = walk slow/quiet, Sprint maybe hold W? In CS no sprint. But request sprint; use Alt or double? We can use Shift for walk and maybe sprint using "Caps"? Hmm. Simpler:
- Ctrl = crouch
- Shift = walk (slow quiet)
- Hold W and press Space? no sprint. We can set "左Alt" sprint; if walking? Actually CS has no sprint, but request includes. Use "Shift" for walk (steady), "Alt" for sprint? Need assign. We'll implement: Shift = walk/静步; Alt = sprint faster with noise and weapon spread. Maybe.

Need key bindings:
WASD movement, Space jump, Ctrl crouch, Shift walk, Alt sprint, R reload, G use selected grenade? maybe 4-7 equip, Mouse0 fire, Mouse1 zoom, E use bomb/plant/defuse, F inspect? B buy, 1/2/3/4/5/6/7 weapons, P pause? Escape.

Need "观战" when dead: camera follows alive bot. We'll implement if player dead and no alive teammates -> get all alive bots and follow their camera? Could set camera position at bot eye with yaw. Need update.

Need "重生" perhaps no respawn mid-round, but next round.

Need HUD bomb timer: if bomb planted show.

Need minimap rendering: Use canvas 180x160, draw based on map bounds. We can draw wall boxes as dark rectangles from top-down. Need scale. Draw bomb sites as circles. Draw players as small squares (T yellow, CT blue). Need use mapDef.minimap {left,right,top,bottom}.

Need "击杀提示 UI": killfeed.

Need "武器价格" display in buy menu.

Need "回合倒计时": HUD.

Need "比分": HUD.

Need "雷达、小地图、准星": implement.

Need "声音": WebAudio.

Need "烟雾遮挡": Smoke grenade creates volume sphere of smoke material; maybe also a block? Need visual blocking. Implement smoke entity with large sphere (dark gray transparent) and billboard? It should obscure vision; add a particle sprite? Use `THREE.Sprite` with radial gradient canvas, opacity varying. Also if smoke between camera and enemy, raycast can detect? For hit detection, smoke should block shots. We can include smoke meshes in raycast targets? A smoke sphere with material transparent and depthWrite false; raycaster can intersect sphere if mesh. We'll use bounding sphere? Better create sphere mesh scaled, userData.type='smoke'. Include in worldMeshes for bullet raycast? A raycast against a sphere mesh works. Then bullets can't shoot through smoke. Visual okay. Need fade in/out.

Need "闪光致盲": Flashbang creates white overlay opacity for players/bots facing. Implement for player: overlay div white with opacity based on distance/angle, decay over seconds. Bots: set blinded timer.

Need "燃烧瓶": create fire zone as boxes/particles that damage over time. We'll create a floor plane with flame texture and some point light, every damage tick. Could.

Need "下包/拆包": E key interaction. Need detect if player within bomb site radius. On E: if player is T and not planted and hasC4 -> start plant timer (3s). If CT and bomb planted -> defuse timer (5s). Need show progress bar. Bots can plant/defuse? We'll implement bot actions similarly. Need sound beeping.

Need "Bomb entity": after plant create C4 mesh at site, plus beeping sound. Need timers.

Need "Bomb explosion": explosion visual/sound, damage.

Need "round end": pause game, show banner, after 5s next round.

Need "freeze time": positions frozen, can buy. Actually game can allow move in buyzone; CS freezetime. We'll allow 8s buy before round live. During freeze, no shooting? Need set `roundPhase='freeze'`. After freeze -> live. Bots wait.

Need "round timer": countdown. Use `roundTime` seconds; start live 115? Use 105 for quick. Bomb timer 40.

Need "match point": First to 16? For complete, match ends at score 16 or 8? We'll use first to 8 for faster. After match end, show victory screen with restart/menu. Need optionally continue to next half? Not.

Need "economies update":
- Start money $800.
- Win: if bomb explode/defuse? Standard: T win by bomb explode $3500? We'll use:
  * Win round: $3250, loss bonus: 1400 consecutive? Actually increasing. Simpler:
    - Win 3000, plant bonus? $800? Bomb plant loss? We'll implement:
      After round end: winners get $3250, losers get $1500 + (consecutiveLoss*500 up to 3000). If bomb planted (T) give $800. Kill rewards: $300 kill, $150 assist? We'll just kill reward.
- Need "保枪" not implement fully; if team loses, keep weapons? We'll ignore.

Need "装备购买": armor, helmet. In buy menu:
- Armor + helmet $1000, armor only $650, defuse kit $400 (CT), grenades.
Player properties `armor`, `helmet`.

Need "爆头": hit part head yields x4.

Need "Bot AI":
- Buy phase: choose weapons.
- Task assignment:
  * T side: choose strategy randomly A or B. If player T maybe player does own; bots follow one of routes? To keep simple, bots choose target site and move to it. Plant if at site.
  * CT side: choose defense positions (A site, B site, mid) from nav nodes. If bomb planted, rotate to bomb site and defuse. If T team plant, all bots seek bomb site.
- Enemy detection:
  * For each bot, periodically scan for enemies in view (distance, angle, line-of-sight blocked?) With raycast if no world walls between. If detect, engage.
  * Aim at target: set aimAngles to point to enemy. Need update bot yaw/pitch.
  * Fire if target visible and within reaction and distance, with weapon spread.
  * Move while shooting? maybe stop if shooting.
- Bot movement along path:
  * Use nav graph Dijkstra to target node.
  * Need target node for strategic areas. For T attack A: choose goal node near A site. For B: B.
  * CT defend choose node around sites/mid.
  * If bomb planted, all CT goal to bomb site; T to defend nearby.

Need bot coordinate to nav target: nodes are [x,z]. The bot position = [x,z]. We can use mapDef.nav.

Need avoid bots stacking: assign different nodes to bots.

Potential issue: player collision with world: we need collision function using mapDef.walls. For bots, same. Need ground and platform collisions. We'll use `collide(pos, vel, dt, radius, height)`.

Need "MapDesign" include `walls` in mapDef. For each wall box, its topY = size[1] maybe 4. Need for collision.

Need "Objects" maybe crates:
- mapDef.objects: boxes with pos,size,color/type, can be destructible? no.
Actually walls array includes all boxes including crates? We'll have `colliders` = walls+objects. For raycasting, `worldMeshes` include all. For collisions, mapDef.colliders array. To make visually okay, create Mesh with appropriate material.

Map materials:
Create `textures`:
- `sandFloor` canvas: beige with noise, maybe tile.
- `wallSand`: sandy brown with patches.
- `concrete`: gray.
- `metal`: dark gray.
- `wood`: planks.
- `grass`: green.
- `rock`.
Need use repeat based on box size. For textures, `new THREE.CanvasTexture(canvas); texture.wrapS/T = RepeatWrapping; texture.repeat.set(sx/4, sz/4)`. For walls maybe repeat.

When creating a map, process `wallDef` with `mat`:
`W(x,z,sx,sz, mat, height)`.
Need if material type, assign `matGrid`.

Let's define helper:
```
function addWall(map,x,z,sx,sz,mat,h=4){
 map.walls.push({pos:[x,h/2,z], size:[sx,h,sz], mat:mat});
}
function wallX(map,x,z1,z2,mat,th=0.8,h=4){ addWall(map,x,(z1+z2)/2,th,h,z2-z1,mat); }
function wallZ(map,x1,x2,z,mat,th=0.8,h=4){ addWall(map,(x1+x2)/2,z,(x2-x1),th,z? Actually size [len, h, th], center z. }
```
Need careful with addWall signature: addWall(x,z,sx,sz). For wallX: center x, center z = (z1+z2)/2, size [th,h, z2-z1].
For wallZ: center x=(x1+x2)/2, center z, size [x2-x1,h,th].
We'll create W(x,z,sx,sz) sets size [sx,4,sz]. But if h variable, W(x,z,sx,sz,h=4).

Map def:
```
{
 name:'Dust2 Remake',
 fogColor: 0xd8c7a0, bgColor:0xd8c7a0, hemisphere: [0xbfbfbf, 0x3a3a3a, 0.8],
 light:[x,y,z],
 bounds:{minX:-90,maxX:90,minZ:-70,maxZ:70},
 walls: [],
 spawns:{T:[...], CT:[...]},
 bombSites:[{name:'A', x:50,z:45,r:7},{name:'B',x:-45,z:45,r:7}],
 buyZones:[{x:-70,z:-25,r:20,team:'T'},{x:70,z:-25,r:20,team:'CT'}],
 nav:[...]
}
```
Need map-specific spawns. For Dust2: T spawn (-70,-25), CT (70,-25). But if T spawn at (-70,-25) inside between walls? yes. Need spawn y=0. yaw facing east for T, west for CT? T yaw maybe 0 (facing east), CT yaw π (west). We'll set.

Need map2 and map3:
Could use a generic map generated from box rectangles with smaller set. We'll create Map2 "Blaze" (industrial) and Map3 "Tempel" (desert temple). Need unique tactical rhythm. But user specifically only Dust2 high fidelity. We can make map2/3 moderate.

Map2 "Crimson" maybe symmetrical? Need different. We'll create a "Dust-like" urban map with central plaza and side routes. Need not too detailed.
Map3 "Valley" with one long mid and vertical/side? Maybe.

Maybe create a random/generated map? Need no. Let's use simple wall lines.

Map2 "Industrial Site":
- Square map 100x100. Two bomb sites at west/east. T spawn north, CT spawn south. Mid corridor central, two side routes, warehouse crates. This naturally different.
Map3 "Ancient": T spawn southeast, CT northwest, sites at top left/right, with narrow corridors and temple pillars. Good.

Need code navigation. We'll define `createIndustrialMap()` and `createAncientMap()` in code with walls.

Given final code long, maybe we need not detail in final prose.

Let's start building final code skeleton.

Potential issue: CDN Three.js r128 and global. We'll include script from cdnjs. If user has internet, works. We should mention requires internet for CDN. But if offline no. Could use importmap? no.

Need maybe use ES6. The HTML can be large. Let's craft.

Let's write code step by step mentally.

HTML:
```
<!DOCTYPE html>
<html><head><meta charset="utf-8"><title>Counter-Strike Browser</title><style>...</style></head>
<body>
<div id="menu">...</div>
<div id="hud">...</div>
<div id="damageOverlay"></div>
<div id="flashOverlay"></div>
<div id="banner"></div>
<div id="buyMenu"></div>
<div id="progress">...</div>
<canvas id="minimap" width=180 height=180></canvas>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
... game code ...
</script>
</body>
</html>
```

CSS:
- Reset margin 0, overflow hidden, font family.
- HUD top left, top right, bottom.
- Crosshair centered with CSS pseudo? We'll use div.
- Buy menu hidden.
- Menu full screen with buttons.
- Need pointer lock canvas.

JS:
Use `window.addEventListener('DOMContentLoaded', init)`? Since script at bottom, DOM ready.

Need robust.

Let's outline functions:

```
const $ = id=>document.getElementById(id);

let scene, camera, renderer, clock;
let viewmodel, gunGroup;
let player = {...}
let bots = [];
let raycaster = new THREE.Raycaster();
let currentMap = null;
let keys = {};
let game = { state:'menu', mapIndex:0, side:'CT', difficulty:'normal', botCount:4, scoreT:0, scoreCT:0, round:1, phase:'freeze', roundTime:0, freezeTime:8, bomb: {planted:false, time:0, site:null, mesh:null, beep:0}, matchEnded:false, activeBots: [] };
```

Need update HUD elements. We'll create HUD:
```
<div id="top-left"><span id="hp">100</span> ... </div>
<div id="top-center">round timer</div>
<div id="top-right">score</div>
```

Need "deathcam": if player.alive false, camera follows bot. We'll implement in updateCamera.

Now code.

First textures:
```
function makeCanvas(w,h,draw){...}
function texFromCanvas(...){...}
function createTextures(){
 const sand = makeCanvas(256,256, ctx=>{...});
 ...
 textures.wall = new THREE.CanvasTexture(sand);
 ...
}
```
Need use in materials. For floor, repeat.

Since wall meshes use materials; for each map wall `mat` string, use `materials[mat]`. We'll pre-create `materials` with MeshLambertMaterial({map:texture}). Color slightly.

Need not use `THREE.ColorManagement`.

Map build:
```
function buildMap(mapDef){
 clearScene();
 currentMap = mapDef;
 scene.background = new THREE.Color(mapDef.fogColor);
 scene.fog = new THREE.Fog(mapDef.fogColor, 20, 120);
 add lights;
 const groundGeo = new THREE.PlaneGeometry(mapDef.width, mapDef.depth); // but mapDef.bounds. Place y=0, rotate -PI/2. Mesh material floor.
 ground.receiveShadow = true; scene.add(ground);
 mapDef.walls.forEach(w=>{ ... });
 mapDef.crates?.forEach...
 // spawn points
 // Minimap update canvas initially
}
```
Need floor plane only, but if some maps have different ground? okay.

Collision list:
`const colliders = currentMap.walls;` But after build map, we need have collision. However for `buildMap`, mapDef.walls includes full size. Need if player collides with floor? floor no top? We'll treat y=0 as default.

For AABB collision:
```
function getPlayerAABB(pos, r, h) {
 return {minX:pos.x-r, maxX:pos.x+r, minY:pos.y, maxY:pos.y+h, minZ:pos.z-r, maxZ:pos.z+r};
}
function collide(pos, vel, dt, r, h, stepHeight=0.5){
 // move axis separately
 const colliders = currentMap.colliders;
 pos.x += vel.x*dt;
 for (const c of colliders){
   if (aabbOverlap(pos,r,h, c)) {
     if (vel.x>0) pos.x = c.pos[0]-c.size[0]/2 - r;
     else if (vel.x<0) pos.x = c.pos[0]+c.size[0]/2 + r;
     vel.x=0;
   }
 }
 pos.z += vel.z*dt; ... 
 pos.y += vel.y*dt;
 if (pos.y <= ground) pos.y=0, vel.y=0, grounded=true else grounded=false
 // check boxes for vertical
 // if falling into box top, set pos.y = c.pos[1]+c.size[1]/2; grounded=true; vel.y=0;
 // if hitting head, pos.y = c.pos[1]-c.size[1]/2 - h; vel.y=0
}
```
Need AABB overlap:
```
function overlap(pos,r,h,c){
 const px = pos.x, py=pos.y, pz=pos.z;
 const ex = c.pos[0], ey=c.pos[1], ez=c.pos[2];
 const sx=c.size[0]/2, sy=c.size[1]/2, sz=c.size[2]/2;
 return px+r>ex-sx && px-r<ex+sx && py+h>ey-sy && py<ey+sy && pz+r>ez-sz && pz-r<ez+sz;
}
```
Need if box at y height center=2, sy=2 => top=4. Player pos.y=0, height=1.8, overlap true. For movement, horizontal collision only if vertical ranges overlap. If player standing on box top, pos.y should be top (2? no). We'll implement after horizontal moves too? Usually check horizontal then y. But if y is 2.0 standing on box and then moving horizontally, vertical overlap? For box top at 2, player bottom y=2, vertical range [2,3.8], box [0,2], overlap at boundary? `py+h > ey-sy` true and `py < ey+sy` (2<2) false, so no overlap. Good. So step-up.

Need implement vertical move carefully:
```
const wasGrounded = false;
pos.y += vel.y*dt;
for c of colliders:
 if overlap:
   if vel.y <= 0 && pos.y + h > c? Actually if falling into box top:
      if pos.y < c.top && pos.y + h > c.top? not easy.
```
Simpler: after vertical move, if overlap:
- If vel.y<=0 and previous center? We can compute if player bottom crosses top: set pos.y = c.top; vel.y=0; grounded=true;
- Else if vel.y>0: set pos.y = c.minY - h; vel.y=0.
Need not robust.

But horizontal collision after pos.y set to top works because no vertical overlap.

Need handle step-up:
When horizontal collision and `c` top <= pos.y + stepHeight? and pos.y < c.top? We can attempt to move up to c.top before resolving. Maybe later.

Actually many world walls have height 4, top=4; no step. Crates step height maybe 1.2, but stepHeight 0.5 no.

Need collision with floor from plane at y=0 not in colliders? We'll just clamp.

Need mapDef.colliders includes wall boxes and crates. Use `c.pos[1]` is center y, `c.size[1]` is height.

For shooting raycast, need world meshes. We'll store `worldMeshes` global. Each wall mesh in it. Crates too.

Need collision with smoke. We'll create `smokeMeshes`; include in `windMeshes`? If include smoke mesh, bullets blocked. For raycast, `worldMeshes` array can include smoke meshes; but smoke is a sphere and transparent; raycaster intersects. Need ensure not too many? okay.

Need gun raycast target list:
`const raycastTargets = worldMeshes.concat(botPartMeshes, smokeMeshes)`. Actually worldMeshes already includes smokeMeshes? We'll maintain `raycastList`; update when smoke added.

When building scene, `worldMeshes` includes map/static meshes. `botPartMeshes` is global array of bot part meshes (with userData). `dynamicTargets` for smoke/grenades. For shooting:
```
function getShootHit(origin, dir, maxDist){
  const targets = worldMeshes.concat(smokeMeshes).concat(botPartMeshes);
  raycaster.set(origin, dir);
  raycaster.far = maxDist;
  const hits = raycaster.intersectObjects(targets, false);
  return hits[0];
}
```
Need bot part meshes direct children? In `targets`, if bot part meshes are direct Mesh but nested in group; intersectObjects(array, false) with direct meshes will work because meshes are objects in scene but their matrixWorld updated. `false` means don't traverse children of each target, but target is mesh itself, okay.

Need update worlds matrices before raycast: `scene.updateMatrixWorld()`.

Need target part userData for bot:
When creating bot group:
```
const group = new THREE.Group();
const head = new THREE.Mesh(headGeo, material);
head.userData = { bot: bot, part:'head' };
group.add(head);
botPartMeshes.push(head);
...
```
Since `head.position` set local.

Need bot model:
- head sphere radius .12 at y=1.7.
- chest box at y=1.25 size .5,.4,.28.
- legs boxes at y=.4.
- arms maybe.
- Add a small weapon box in hands. Need orientation? Our botGroup faces yaw. We'll set group.rotation.y. Body front is +z? In local, set gun at (0,1.0,0.3). If group.rotation.y = yaw (angle from +x? Actually set `bot.pos` and `bot.model.position`. For facing, if yaw = atan2(dirX,dirZ)? If model faces +z local, to face direction (vx,vz), `model.rotation.y = Math.atan2(vx,vz)`? Let's use: if local +z is forward, yaw = Math.atan2(vx, vz). For player camera yaw, if yaw=0 faces +x? Actually camera rotation order? We'll handle separately. For bots, use yaw = atan2(dx, dz).

Bot movement: `velocity = dir * speed`. If collision, stop. Need avoid wall. For next waypoint, if dist < 1.5, advance index.

Need maybe bots can "jump" if stuck? We can set `vel.y=4` if blocked? Implement small chance.

Need bot aiming:
We need calculate pitch/yaw from bot position to target:
- Forward vector in world = (sin(yaw), 0, cos(yaw))? If local Z forward and yaw from +Z? Actually `Math.atan2(dx, dz)` gives yaw where yaw=0 points +Z, yaw=PI/2 points +X. Direction = (sin(yaw), 0, cos(yaw)). Good.
- To aim at target, desired direction from bot eye to target (dx,dy,dz), desired yaw = atan2(dx,dz). pitch = atan2(dy, sqrt(dx^2+dz^2)).
- Use smoothing.

Bot AI:
```
function updateBot(bot, dt){
 if(!bot.alive) return;
 if(game.phase==='freeze' || game.phase==='roundEnd') return;
 // if blinded skip
 // Determine goal
 updateBotGoal(bot);
 // Move along path
 // Detect enemies
 detectAndEngage(bot, dt);
}
```
Need `updateBotGoal`:
- For T, if no bomb planted:
  * Strategy: `bot.siteGoal` initial random A/B; while not reached, move to waypoint of that site; if at site and has bomb? If bot has bomb, plant; if no bomb, maybe defend/roam. Need if player got bomb, bots still attack site. After planted, T defend bomb.
  * Use target node chosen near site. Each bot maybe different route node? We can choose route start node for A/B from nav map, but if all use same nearest target, path same. Assign random node near site from `map.nav` links? For simplicity choose a route node e.g. `nodeIndexA` and `nodeIndexB` from `map.attackRoutes`. Use random.
- If bomb planted: CT bots go to bomb site and defuse; T bots defend near bomb.

Need need nav graph. We'll define `map.attackRoutes = { A: [nodeIndices], B: [nodeIndices] }`.

For CT defend:
- Before bomb, bot selects defense node: A site, B site, mid. Move to pose. If sees enemy, engage. If player side CT, bots maybe guard positions.
- If bomb planted, move to site.

Need detect enemies:
```
function findVisibleEnemy(bot, maxDist=80){
 const enemies = bots.filter(b=>b.alive && b.team!==bot.team); if player alive && player.team!==bot.team enemies.push({...player})
 for each enemy, compute vector, dist, angle to bot forward < 70? Actually bots should detect even behind? In CS they can hear? Use fov 100 deg. If dist > max skip. Do raycast from eye to enemy chest; if blocked by world/smoke -> no. Return {target, part, dist}
}
```
Need include player in bot enemy list. We'll need player object treated similarly. Bot raycast to player:
- Since player has no third-person model, raycast to a ghost collider not visible. For bots to target player, we can create a small invisible `playerHitMesh` (box/spheres) attached to camera/player? Player has no body model but need hitbox. Create three hit meshes (head, chest, legs) as invisible in scene at player pos. But for raycasting, need bot can see. Add to `playerHitMeshes` with userData `{bot: null, player:true}`? For shooting, player can shoot bots; bots can shoot player. We'll create player body group (invisible) with boxes/spheres and update with player position and yaw. Use materials transparent? We'll set visible = false but raycast? In Three.js raycast ignores invisible? It checks visible? I think Raycaster checks `object.visible`; if invisible, not intersected. Need material visible false? Actually object.visible false => raycast skip. Could set material opacity 0, transparent true, depthWrite false. Then raycast still works. We'll make player hit meshes with transparent material opacity 0, color. Add to botPartMeshes? Need not affect world. Use `playerHitMeshes` in raycast targets too. When shooting player? Player can't shoot self. For bot detection, raycast targets include bot bodies and player hit meshes. Good.
- Similarly player damage updates `player.hp`.
Need create player hit meshes in scene, not camera. Since player moves, update each frame. Use Mesh with opacity 0.0. Head sphere etc.

Need if player is dead, hide player hit.

Need "观战": camera follows alive bot: find first alive bot. We can set camera pos/rotation from bot eye and yaw/pitch. We'll not show player model? If dead, camera at bot's eye.

Need round reset:
```
function startRound(){
 round++;
 phase='freeze'; roundTime=15? maybe 10;
 // positions
 if(player.team===game.side) spawnPlayer();
 else player.alive=false? Actually player is on selected team. If menu side = T/CT, player always in that team. Good.
 // bots create if needed. At new map, create bots with assigned team counts.
 // allocate bomb: if player team T, player.hasBomb=true; else random bot T hasBomb.
 // reset hp/armor, default weapons
 // give money? Round start no money? Buy phase money from previous; if first $800.
 // position bots
 // update UI
}
```
Need create bots once per match and keep alive? We'll create at start, reset each round. Bot count total; team split: if player T, remaining bots: T count = ceil((botCount-1)/2)? We need total 10? Let's implement:
- `game.botCount` = total bots excluding player? Maybe total player + bots = 10. Let's set `botCount` in menu (e.g. 4,6,8,10) total players per team? Use `teamsize = game.botCount/2 +1`? simpler: `game.botCount` total bots; T side has `ceil(game.botCount/2)` bots, CT side `floor`; plus player if team T/CT. Need total each team maybe.
Actually choose bot count as "敌人与队友总数"? We'll use `game.botCount` as number of bots total, not including player. If player T, T bots = Math.floor(botCount/2)? But need enough enemies. Let's do:
- `teamBots = {T: Math.ceil(game.botCount/2), CT: Math.floor(game.botCount/2)}`. If player T, T side has player + teamBots.T; CT has teamBots.CT. Good. If botCount=4, 2 bots per side.
Need if player side selected; if CT, CT has player + teamBots.CT.

Need match score: if team wins, scoreT/CT update. If score >=8 end match.

Need "T/CT bots loadout":
- On startRound, for each bot, buyBasedOnMoney(bot). If bot has enough, choose primary. We'll set:
```
function botBuy(bot){
 bot.money += 0? 
 if (bot.money > 4500) equip awp maybe; else if >3000 m4/ak; else if >1000 smg; else keep pistol.
 if (bot.money >1000) bot.armor=100, helmet=true; if CT bot.defuseKit = true.
}
```
Need money at first round 800, bot may not buy. Good.

Need weapon in bots: simplify bots have `weapons` list? We can store `bot.weapon` object string and ammo. Use same data. Need reload.

Need damage from bot to player: If bot fires and hit player, apply damage. Need determine if bot's ray hits player hit meshes. We'll implement `botFire(bot)` similar to player fire but ray from bot eye to aim point. If first hit target is player or enemy bot. Actually if bots shoot, they can hit teammates? Need ignore teammates. We'll implement general `shootRay(origin, dir, shooterTeam, shooter)`:
- Raycast targets: world + smoke + all bot part meshes + player hit meshes. If first hit object with userData.bot (another bot) and bot.team !== shooterTeam, apply damage. If player hit and player.team !== shooterTeam, apply damage. If bot and same team, ignore? In CS friendly fire off generally. We can ignore same team by skipping? Raycast returns first; if same team body in front of enemy, bullet would hit. But friendly fire off? Simpler not implement team collision? Bots could shoot through teammates; acceptable. But maybe avoid by not doing friendly fire damage.
- For player firing, target first hit can be enemy bot or world. If player team bullet hits own bot? Should probably not damage (unless friendly fire disabled). We'll skip same team.
Need damage calculation:
```
function applyDamage(target, dmg, part, shooter){
 const multiplier = part==='head'?4: part==='chest'?1:0.8;
 let armorAbsorb = 0.5 if armor >0 and part not head or helmet;
 // if armor >0: dmg*0.75 to armor, rest to hp? Use CS: armor reduces 50%; but okay.
}
```
Need update kill feed if hp<=0.

Need player shooting:
```
function playerShoot(){
 if (!player.alive || game.phase!=='live' && game.phase!=='bombPlanted'? Actually can fire when live)
 if (reloading/cooldown) return;
 const wep = getPlayerWeapon();
 if (wep.mag===0) { playEmpty; reload(); return; }
 wep.mag--;
 spawn muzzle flash;
 recoil...
 let spread = wep.spread + movementPenalty + recoilPenalty;
 for pellets count...
 const dir = camera.getWorldDirection(new THREE.Vector3()); apply random spread.
 const hit = shootRay(camera.position, dir, player.team, player);
 if hit target: do damage if enemy.
 updateHUD
}
```
Need camera position? `camera.position` equals player eye? In update we set from player. But when shooting, use camera world position. Good.

Need animate viewmodel fire. We'll set `gun.recoilOffset = 0.12` and angle.

Need reload:
- If mag < magSize and reserve >0, set reloading true, timer. After reload time, transfer ammo.
Need "换弹动画": move gun downward/rotate.

Need "切枪动画": maybe simple scale? no.

Need "开镜": right-click for `wep.scope` toggles `player.scoped`; if scoped, change camera.fov to 20 and add dark Vignette via CSS. For pistols maybe alt? no.

Need "手雷":
- To throw, player must equip grenade (key 4 etc). On mouse0, if grenade, perform throw with power based on time? Simpler: instant throw with parabolic velocity. Need spawn grenade mesh and physics.
- Grenade entity:
```
{ mesh, pos, vel, type, fuse, alive, smokeTimer}
```
Update in loop: gravity, collide with world (point collision with boxes). When fuse 0:
  - HE: explosion sphere, damage, sound, light.
  - Flash: white overlay if near and in view; if player/bot within radius and angle, add blinded time.
  - Smoke: create smoke sphere/sprite, add to smokeMeshes; fade after 17s.
  - Molotov: create fire zone (disc), damages over 5s.
```
Need grenade throw force: from camera dir * speed + player vel * 0.3. For HE/Flash/Smoke same.
Grenades should collide with ground and walls. We can implement point collision using colliders; if overlap, invert? Simpler: on ground y <=0 set y=0 and stop vertical. For walls, if inside collider, push back by delta. Since grenades are spheres small.
Need if grenade bounces? reduce velocity. We'll implement.

Need "下包":
In `updatePlayerInteraction`:
```
const site = getNearestBombSite(player.pos);
if (player.team==='T' && player.hasBomb && !game.bomb.planted && site && dist<site.rad && game.phase==='live'){
   prompt show "按住 E 下包";
   if (keys['KeyE']) { plantProgress += dt; ... if complete plantBomb();}
   else plantProgress=0;
}
if (player.team==='CT' && game.bomb.planted && site same and dist<rad) defuse...
```
Need if bomb planted, site is planted site. Defuse progress resets if move away.

Need bots plant/defuse:
- In bot update, if T and bot.hasBomb and at site and no bomb and phase live: start plantBot progress; if complete plantBomb(bot).
- If CT and bomb planted and at bomb site: defuse. Need maybe defuse kit shorter.

Need "sound": WebAudio context must resume on first user click. We'll create `AudioContext` on pointerlock click. `playSound` functions.

Need "Bomb beep": in loop if bomb.planted, every 1s decrease. Use `game.bomb.time -= dt`. `playBeep` each second. Explosion at 0.

Need "Round end":
`endRound(winner, reason)` set phase='roundEnd', roundEndTimer=5. Update scores, money, banner, killfeed. After timer if match end else startRound.
Need if matchEnded show match screen overlay.

Need "Pause menu": if pointer unlocked, show pause overlay. `document.addEventListener('pointerlockchange')`.

Need "Maps definitions":
Let's implement manually with wall functions.

Let's write code in final. Need be mindful of line count but can be high.

Potential issues with `three.js` r128 and shadows? okay.

Let's begin code.

Need include `THREE` global. We'll create a full script. In final, code will be long. Let's craft in chunks.

Before final, maybe I can write pseudo in my head.

Detailed code:

```
const WEAPONS = {
 knife: {name:'军刀', type:'knife', price:0, damage:40, rpm:600, mag:0, slot:0, max:0, auto:false, spread:0, recoil:0, reload:0, range:2.5},
 glock:{name:'Glock-18', type:'pistol', price:0, damage:26, rpm:400, magSize:20, reserve:120, auto:false, spread:0.02, recoil:0.012, reload:2.2, range:1000},
 ...
}
```
We need each weapon object has `reserve` max total reserve? `mag` current. For player, `player.weapons` array? Use object:
`player.inv = { knife: {id:'knife', ammo:0, reserve:0}, glock: {id:'glock', ammo:mag, reserve:reserve}, ... }`
Maybe easier:
`player.loadout = { active:'glock', weapons: { knife: {ammo:0,reserve:0}, glock:{ammo:20,reserve:120}, primary:null, grenades:{he:0,flash:0,smoke:0,molotov:0} } }`
Need buy primary set.

Weapon retrieval:
```
function getWepDef(id){ return WEAPONS[id]; }
function getActiveWeapon(){
 const id=player.loadout.active;
 if(id==='he'||...) return WEAPONS[id];
 const w=player.loadout.weapons[id];
 return w? { ...def, mag:w.ammo, reserve:w.reserve}:null;
}
```
For bots, similar but simplified: `bot.weapon = 'ak47'`, `bot.ammo`, `bot.reserve`.

Given time, perhaps simplify active weapon management:
- `player.loadout` object has properties:
```
loadout: {
 knife: {ammo:0,reserve:0},
 glock: {ammo:20,reserve:120},
 primary: {id:null, ammo:0,reserve:0},
 grenades: {he:0, flash:0, smoke:0, molotov:0},
 active: 'glock'
}
```
When buy primary, set `loadout.primary = {id:'ak47', ammo:30, reserve:90}`.
Active id can be 'knife','glock','primary','he','flash','smoke','molotov'. For functions getActive returns based on active.
Need if active='primary' and no primary, switch to glock.

Grenade throw uses `loadout.grenades[active]--`, and active returns to glock.

Need bots can buy grenades? Store `bot.grenades = {he:1,...}`.

Need HUD crosshair dynamic.

Need "Player state":
```
let player = {
 team:'CT', alive:true, hp:100, armor:0, helmet:false, money:800, pos:new THREE.Vector3(), vel:new THREE.Vector3(), yaw:0, pitch:0, height:1.7, crouching:false, onGround:true, reloading:false, reloadTime:0, cooldown:0, scoped:false, hasBomb:false, walking:false, sprinting:false, equipTime:0, lastWeapon:null, plantProgress:0, defuseProgress:0,
 loadout: defaultLoadout()
}
```
Need position `x,z`; y? We use `player.pos` Vector3.

Need defaultLoadout:
```
function defaultLoadout(team){
 const sidePistol = team==='T'?'glock':'hkp2000';
 return {
   knife:{ammo:0,reserve:0},
   [sidePistol]:{ammo:WEAPONS[sidePistol].magSize, reserve:WEAPONS[sidePistol].reserve},
   primary:{id:null, ammo:0,reserve:0},
   grenades:{he:0,flash:0,smoke:0,molotov:0},
   active: sidePistol
 }
}
```
Need use `hkp2000` or `usp`. We'll define `usp` id maybe.

Let's define weapons:
- `usp` for CT pistol.
Need `WEAPONS.usp`.

Buy menu:
We'll create HTML buttons dynamically:
```
function showBuyMenu(){
 if(!player.alive || !inBuyZone()) return;
 $('buyMenu').style.display='block';
 const div=$('buyItems'); div.innerHTML='';
 const items = [
  {label:'AK-47 $2700', action:()=>buyWeapon('ak47')}, ...
 ];
}
```
Need "not in buy zone": In CS only spawn buyzone. We'll compute `inBuyZone` if player.pos within map.buyZones team. Need after round live maybe buyzone remains? CS buys first 15 sec? We'll allow during freeze only. If phase !== 'freeze' maybe close. For convenience, buy during freeze (10s) only.
Need buy menu buttons also for armor, defuse kit, grenades.

Need "Weapon switch": `switchWeapon(id)` sets active, equipTime=0.3, reloading=false, cooldown=0. If active is grenade and no grenade, return.
Key handling:
```
case 'Digit1': active='primary'; if no primary switch to pistol
case 'Digit2': active=sidePistol
case 'Digit3': active='knife'
case 'Digit4': if he>0 active='he'
...
```
Need maybe `KeyQ` for last weapon? no.

Need "Movement output":
```
function updatePlayer(dt){
 if(!player.alive) return;
 const speed = 4.0;
 if (player.crouching) speed=2.0;
 if (player.sprinting) speed=6.0;
 if (player.walking) speed=1.8;
 // acceleration
 const forward = new THREE.Vector3(-Math.sin(yaw),0,-Math.cos(yaw))? Need camera yaw convention.
```
Need define camera orientation. Three.js camera by default looks down -Z. We'll set `camera.rotation.order = 'YXZ'`; yaw around Y, pitch around X. If yaw=0, camera forward is -Z (x=0,z=-1). If yaw increases, rotates left? We'll set movement:
```
const forward = new THREE.Vector3(-Math.sin(yaw), 0, -Math.cos(yaw));
const right = new THREE.Vector3(Math.cos(yaw), 0, -Math.sin(yaw));
```
Then input W moves forward. Good.
Mouse movement: `yaw -= movementX * sens; pitch -= movementY * sens; clamp pitch`. If yaw defines camera rotation as above.
If player yaw is zero, facing -Z. For bots we used local forward +Z; okay separate.

Need camera eye = `player.pos + Vector3(0, eyeHeight,0)`. Since player.pos y = feet. If crouching eyeHeight=1.1 else 1.6.

Need "sprint" using Alt; but Alt often browser menu. We'll use `ShiftLeft` maybe for walk; `KeyQ` for sprint? Hmm. Use Shift = walk (slower), Ctrl = crouch, Alt = sprint (but prevent default). We'll listen `AltLeft`. In some browsers Alt focuses menu, but pointer lock maybe okay. Could use `KeyG`? No. We'll use `ShiftLeft` = walk, and `KeyC`? Maybe "Sprint" can be double W? Let's implement `ShiftLeft` as walk, `KeyG`? But KeyG reserved. Maybe "sprint" is holding W and pressing `ShiftRight`? Wait in many games sprint is Shift. But request both walk and sprint. We can set:
- Ctrl = crouch
- Shift = 静步 (slow, quiet)
- Alt = 冲刺
- CapsLock maybe sprint? We'll use `KeyQ` for sprint? Better use `ShiftLeft` for walk, `KeyW`+`Alt` sprint. We'll implement `keyAlt` event with `e.preventDefault()`.

Need "footsteps": If moving and on ground, every step (0.5s) play footstep; if walking volume low.

Need "weapon bob": viewmodel bobbing while moving.

Need "jump": Space; if onGround, vel.y=5.0.

Need apply gravity:
`player.vel.y -= 10*dt; collide` etc.

Need "crouch" adjusts player collision height in `collide`? We'll need `physHeight = crouching?1.2:1.8`. But if crouch under low? no low roofs. Use.

Need "Bots physics": same `moveEntity(entity, dt, speed)` uses collide with radius .35. Need entity has pos vector, vel, yaw. We'll implement `moveBody(pos, vel, dt, radius, height)` returns grounded.

Need "map colliders" include boxes; but some walls high. Use currentMap.walls, not objects? We'll set colliders = mapDef.colliders array, which includes walls and crates. `mapDef.walls` maybe all.

Let's implement map definitions with `colliders` via helper:
For each map, create `map = { walls: [], colliders: [], ... }`; helper `W` pushes both walls and colliders. In buildMap, for each wall in `map.walls` create mesh. Collision uses `map.colliders`.

Need "walls" and "colliders" same object? We'll keep one array `map.colliders`; build meshes from `map.colliders`. Use also for minimap. So `W` pushes to map.colliders.

Function:
```
function addCollider(map, x, z, sx, sz, h, mat) {
 map.colliders.push({pos:[x,h/2,z], size:[sx,h,sz], mat:mat, type:'wall'});
}
function W(map,x,z,sx,sz,h=4,mat='wall'){ addCollider...; }
function wallX(map,x,z1,z2,mat='wall',th=0.8,h=4){ W(map,x,(z1+z2)/2,th,h, z2-z1, mat); } 
// Wait W signature x,z,sx,sz,h,mat. Need center z. W(map, x, (z1+z2)/2, th, z2-z1, h, mat)
```
Need adjust.

In buildMap, material selection:
```
function meshForCollider(c){
 const geo = new THREE.BoxGeometry(c.size[0], c.size[1], c.size[2]);
 const mat = materials[c.mat] || materials.wall;
 const mesh = new THREE.Mesh(geo, mat);
 mesh.position.set(c.pos[0], c.pos[1], c.pos[2]);
 mesh.castShadow = true; mesh.receiveShadow = true;
 mesh.userData.collider = c;
 scene.add(mesh); worldMeshes.push(mesh);
}
```

Need "materials" names: 'sand','concrete','stone','metal','wood','crate','floor_sand','floor_ct','grass'. Use textures.

Create textures:
```
function createCanvasTexture(w,h,fn){ const c=document.createElement('canvas'); c.width=w;c.height=h; fn(c.getContext('2d')); const t=new THREE.CanvasTexture(c); t.wrapS=t.wrapT=THREE.RepeatWrapping; return t;}
```
For wall material:
- `makeSandWall` draw base #d3b278, noise (random dots), darker lines.
- `floorSand`: base #cfb27a, noise, stones.
- `concreteWall`: gray.
- `crate`: wooden planks.
Need materials use `texture.repeat` set? When mapping to boxes of arbitrary size, repeat should scale. If same texture repeated per face, if repeat fixed maybe stretched. We can set each texture repeat new? We'll use a single global texture with repeat set from collider size on mesh? Simpler: when creating material for box, clone texture and set repeat = (size.x/4, size.z/4). Use:
```
function getMaterial(name,size){
 const t = textures[name].clone(); t.needsUpdate=true; t.repeat.set(size[0]/4, size[2]/4);
 return new THREE.MeshLambertMaterial({ map:t, color:0xffffff });
}
```
But creating many materials per wall okay. Need avoid cloning for every? okay. For ground, repeat large.
Need material clone is expensive but okay.

But `textures[name]` might have repeat already; clone resets. Use `new THREE.MeshLambertMaterial({map: baseTex.clone()})`.
Alternatively use single material per type with repeat set 1 and accept stretching. But we can choose "crate" with planks; stretched maybe okay. To keep simpler, use one material per type with repeat=2? But if box large, texture tiled. `texture.repeat.set(2,2)` okay. However for floor, repeat maybe 20. We'll have separate floor texture with repeat 20. For walls, use repeat 4. We'll create materials:
```
materials.wall = new THREE.MeshLambertMaterial({map: sandTex}); sandTex.repeat.set(2,2);
materials.wall2 = ...
```
No per-size. Good enough.

Need "Dust2 sand walls" maybe use same. Floor ground material repeat 20. We'll create ground mesh using floorMaterial with repeat.

Need "sun light":
```
scene.add(new THREE.HemisphereLight(0xfff2d9,0x404040,0.9));
const sun = new THREE.DirectionalLight(0xffeedd,1); sun.position.set(50,80,20); sun.castShadow=true; ...
```

For performance, shadow map only on map? okay.

Need "Map2/3" wall materials different (concrete/stone). Build maps with mat names.

Now map definitions. Let's create in code as functions.

Detailed Dust2 map using manual wall segments from earlier. Let's implement with W functions.

Need careful with W function centers.

I'll write map creation:

```
function createDust2(){
 const m = {name:'Dust2 Remake', ...};
 // boundary
 W(m,-90,0,0.8,140,4,'wall'); W(m,90,0,0.8,140,4,'wall'); W(m,0,70,180,0.8,4,'wall'); W(m,0,-70,180,0.8,4,'wall');
 // Mid corridor
 wallZ(m,-55,55,-20, 'wall'); wallZ(m,-55,55,-30, 'wall');
 // T spawn? maybe no east wall; open.
 // A long
 wallZ(m,-55,40,4,'wall'); wallZ(m,-55,45,-4,'wall');
 // A long vertical
 wallX(m,40,4,35,'wall'); wallX(m,50,4,35,'wall');
 // A small
 wallX(m,20,-20,35,'wall'); wallX(m,30,-20,35,'wall');
 // Mid north wall gap
 wallZ(m,-55,20,-20,'wall'); wallZ(m,30,55,-20,'wall');
 // A site north/east walls
 wallZ(m,25,75,55,'wall');
 wallX(m,75,35,55,'wall');
 // B tunnel
 wallX(m,-65,10,35,'wall'); wallX(m,-55,10,35,'wall');
 // B site walls
 wallZ(m,-65,-20,55,'wall');
 wallX(m,-65,35,55,'wall');
 wallZ(m,-65,-20,35,'wall'); // but need entrance from tunnel, gap x=-65..-55. Let's do wallZ(-65,-58,35?) Wait B site x[-65,-20], B tunnel at x[-65,-55] from z10..35. If B site south wall z=35 from x=-65..-20, then B tunnel entrance blocked. Need wallZ(-55,-20,35) leaving gap x=-65..-55. So wallZ(-55,-20,35,'wall').
 // CT spawn room
 wallZ(m,55,90,-30,'wall'); wallZ(m,55,90,20,'wall');
 // vertical CT corridor
 wallX(m,75,20,52,'wall'); wallX(m,85,20,52,'wall');
 // top corridor south wall...
 wallZ(m,-20,55,52,'wall'); // note this blocks A site connection, maybe okay. Need gap to CT vertical and A.
 ...
}
```
But in this map, T spawn to B tunnel: B tunnel corridor x[-65,-55] z[10,35]; T spawn at x<-55 maybe can enter from south z=10? Need south wall z=10? Actually B tunnel extends from z=10 to35; T spawn could access from z=10 where corridor opens. We can add wallZ(-65,-55,10) to force from T spawn? But if wall at z=10, T spawn at z=-25 can't enter. Need leave gap. Maybe no wall.
B site: West wall x=-65 z[35,55]; South wall z=35 x[-55,-20]; East wall x=-20 z[35,55]; North wall z=55 x[-65,-20]. B tunnel enters at z=35, x[-65,-55] through gap. Good. B site connected to top corridor? East wall x=-20 z[35,55] blocks; top corridor z=52 maybe at x=-20..85. Need B door: opening at x=-20 z=48? Since wall is at x=-20, leave gap z=48..52. We can instead not add east wall? If no east wall, B site connects to top corridor. But top corridor south wall z=52 x[-20,55] would block. Need add top corridor south wall with gap. This is too much.
Maybe B site not connected to CT top? It should be CT can enter B via B doors. To keep playable, we can add a door opening by not placing east wall x=-20 and not placing top corridor south wall? Let's simplify: Top corridor south wall z=52 only from x=-20 to55; if no east wall x=-20, B site can reach z=52 and then top corridor. But if wall z=52 spans x=-20..55, and B site x ends at -20, wall z=52 begins at x=-20; no opening for B. Need set wallZ(-20,55,52) with a gap at x=-20..-15? Let's do:
- Add top corridor south wall as two segments: wallZ(-20,-5,52), wallZ(5,55,52) leaving gaps for B and A? Actually B door at x=-20..-5, A door at x=55..65? Confusing.

Maybe we can connect B/A to top corridor by not adding a south wall for top corridor, just open from z=52 to55? We can let top corridor z=52..60 overlap with site north walls. We'll use walls that define but leave plenty of openings. The map doesn't need airtight. CT can wander.

Given no time, our Dust2 map will be a reasonable approximation; not exact.

Need spawns and bomb sites:
- T spawn center (-70,-25), CT (75,-25). But if T spawn outside mid? okay.
- Bomb sites: A center (50,45), B (-45,45). Need if A site defined walls. We'll add bomb site markers.

Need nav nodes:
Let's add nodes for Dust2:
```
m.nav = [
 {x:-70,z:-25, links:[1,10,15]}, // T spawn
 {x:-40,z:-25, links:[0,2]}, // mid west
 {x:0,z:-25, links:[1,3,14]}, // mid center
 {x:30,z:-25, links:[2,4,8,14]}, // mid east/A small
 {x:60,z:-25, links:[3,5,6]}, // CT mid
 {x:75,z:-25, links:[4,6]}, // CT spawn
 {x:80,z:10, links:[5,7,18]}, // CT vertical bottom
 {x:80,z:35, links:[6,17]}, // CT vertical top
 {x:30,z:5, links:[3,9]}, // A small mid
 {x:45,z:35, links:[8,11,13]}, // A site south
 {x:-40,z:0, links:[0,11]}, // A long west
 {x:0,z:0, links:[10,12]}, // A long mid
 {x:40,z:0, links:[11,13]}, // A long east
 {x:45,z:20, links:[12,9]}, // A ramp/vertical
 // B
 {x:-60,z:0, links:[0,15]}, // B tunnel south? Need T to B? Actually from T at (-70,-25) to -60,0 no.
...
]
```
Need ensure nodes connect passable. We'll include enough:
15 B tunnel mid (-60,20)
16 B site (-50,45)
17 top corridor (-20,52)
18 top corridor (80,52) maybe.
Let's define:
```
0 T spawn (-70,-25)
1 Mid west (-40,-25)
2 Mid center (0,-25)
3 Mid east (30,-25)
4 CT mid (65,-25)
5 CT spawn (75,-25)
6 CT vertical (80,10)
7 CT vertical top (80,40)
8 A small (25,0)
9 A ramp (45,20)
10 A site (50,45)
11 A long west (-40,0)
12 A long mid (0,0)
13 A long east (40,0)
14 B tunnel south (-60,5)
15 B tunnel mid (-60,25)
16 B site (-45,45)
17 Top west (-20,50)
18 Top east (80,50)
```
Links:
0->1,11,14
1->0,2
2->1,3,8
3->2,4,8
4->3,5
5->4,6
6->5,7,18
7->6,18
8->2,3,9
9->8,10,13
10->9,17
11->0,12
12->11,13
13->12,9
14->0,15
15->14,16
16->15,17
17->16,10,18
18->7,6,17
Need A site/bombs: use node 10. B site node 16.
Potential links with walls collide? Maybe okay. Dijkstra works. Bots may walk through walls if edge not open. Need nodes positions in open lanes. Link 0->11 from T to A long at -40,0: T to A long through open area okay. 11->12 along z=0 within A Long corridor. 12->13 same. 13->9 from (40,0) to (45,20) vertical corridor. good. 8->9 from A small to A ramp? (25,0) to (45,20) diagonal may cross wall? Could set 8->3 and 3->? Hmm. Node8 at (25,0) is A small vertical. Node9 at (45,20) is A long vertical. They should connect via A site maybe not direct. Add node at (25,25) connect. We'll simply add links: 8->9 may cross open? It might go through A site? At x=25,z=0 to45,20 diagonal between A small and A ramp; there is wall x=30? Actually A small corridor x20..30; A ramp x40..50; area between x30..40 maybe wall? A site may be open? likely okay.
Need "wall collision" and nav maybe not perfect; bots could get stuck but okay.

Map2/3 nav similar simpler.

Given time, perhaps final code can include Dust2 map only? But user requires 3. We'll add simpler map2/3 with nav and walls maybe generated by fewer W calls.

Map2 "Industrial Site":
Let's create:
- bounds -80..80,-60..60.
- T spawn north (0,-50), CT south (0,50). Bomb sites left/right: A left (-50,0), B right (50,0). Central vertical mid corridor, two side corridors.
- Many crates.
Wall list:
Boundary.
`wallZ(-80,80,-60)`, `wallZ(-80,80,60)`, `wallX(-80,-60,60)`, `wallX(80,-60,60)`.
Central mid: corridor x[-15,15], z[-60,60]? Add wallX(-15,-60,60), wallX(15,-60,60) but need openings? This separates left/right. T at z=-50 can go down. A left/B right connected via side/top/bottom. CT at z=50. Bomb sites left/right.
Simplify:
- Long central corridor; left route x=-40, right route x=40. Bomb sites at (-50,0),(50,0). This gives three lanes. Add walls vertical at x=-25 and x=25 connecting north/south? Actually.
Add:
- Side left corridor bounded x[-60,-30], z[-60,60]? wallX(-60,...), wallX(-30,...)
- Center corridor x[-15,15]
- Side right x[30,60].
Need cross routes at z=-20,20 connecting lanes. Add horizontal walls? Maybe.
This is different. We'll implement with fewer walls.

Map3 "Temple":
Maybe no need.

Given final answer length, we'll include map2/3 but less detailed.

Need "A/B sites" set.

Need "movement collision" with map colliders.

Let's code map definitions at final.

Now Bot AI details.

Bot Object creation:
```
function createBot(team, index){
 const bot = {
   id:index, team, alive:false, hp:100, armor:0, helmet:false, money:800,
   pos:new THREE.Vector3(), vel:new THREE.Vector3(), yaw:0, pitch:0,
   model:null, weapon: team==='T'?'glock':'usp', ammo:..., reserve:..., reloading:false,
   cooldown:0, goalNode:null, path:[], pathIndex:0, lastSeeTime:0, target:null,
   siteGoal: Math.random()<0.5?'A':'B', reactionTime:0, hasBomb:false, grenades:{...}, defuseKit:false, planted:false, blind:0
 };
 buildBotModel(bot);
 bots.push(bot);
}
```
Need if player team T, some T bots. `teamBots` compute:
```
const totalBots = game.botCount;
const tBots = Math.ceil(totalBots/2);
const cBots = Math.floor(totalBots/2);
if (game.side==='T') tBots++? Actually total players includes player. We'll keep bot counts; player is extra. So if botCount=4, T bots 2, CT 2. If player T, T side has 3 humans? Good.
```
At match start, create all bots, reset.

Need `buildBotModel`:
```
const group=new THREE.Group();
const mat = bot.team==='T' ? botMatT : botMatCT;
const headGeo = new THREE.SphereGeometry(0.13,8,8);
const head=new THREE.Mesh(headGeo, headMat); head.position.y=1.72; head.userData={bot,part:'head'};
...
const chest = new THREE.Mesh(new THREE.BoxGeometry(0.5,0.4,0.3), mat); chest.position.y=1.3; userData part chest
const pelvis...
const legs...
const arm...
const gun = new THREE.Mesh(new THREE.BoxGeometry(0.08,0.12,0.5), gunMat); gun.position.set(0.2,1.0,0.3)
group.add(...)
scene.add(group); bot.model=group; botPartMeshes.push(head,chest,...)
```
Need set group position each frame. Head userData bot.

Need Bot shadow maybe cast.

Need update bot model orientation:
```
function updateBotModel(bot){
 bot.model.position.copy(bot.pos); 
 // animate legs? We can add simple leg swing if moving. But not necessary. Maybe arms.
 bot.model.rotation.y = bot.yaw;
}
```
Actually if bot yaw uses `atan2(dx,dz)`, then local +z is forward. Build gun pointing +z. Good. Need chest child? fine.

Need "bot eye position" = pos + (0,1.6,0).

Need "player hit meshes":
Create in `initPlayerHitMeshes`:
```
playerHitMeshes = [];
const head = new THREE.Mesh(new THREE.SphereGeometry(0.22), skinMat); head.position.y=1.7; head.material.transparent=true; head.material.opacity=0; head.userData={player:true,part:'head'};
...
scene.add(head) etc; playerHeadMesh=head...
```
Need update each frame:
```
playerHeadMesh.position.set(player.pos.x, player.pos.y+1.7, player.pos.z);
...
```
Since bot needs raycast to player. The material opacity = 0 but object.visible=true. Good.

Need for player shooting enemies, `shootRay` target array includes bot part meshes and player hit meshes. If first hit is player (self? ray from camera won't hit self unless intersects own hit meshes? Camera inside head, head mesh around camera, ray might hit own head mesh if inside. Need exclude player's own hit meshes when player shoots. We'll set player hit meshes userData.player = true. In `shootRay`, if hit.userData.player and hit.userData.player === shooterPlayer? Need skip. Pass shooterType 'player'/'bot'. If `hit.userData.bot` and bot.id===shooterBotId skip; if `hit.userData.player` and shooter bot? okay.

Need "player hit meshes" should be in raycastTargets for bot detection and player shooting? For bot shooting, yes. For player shooting, to avoid self, skip. We'll not include player hit meshes in global raycast targets? If include, bot can detect. For player shooting, we can after hit if userData.player && userData.playerRef===player skip and raycast next target? Need get nearest non-self. Simpler: `shootRay` finds all hits, loops through sorted, skips shooter self, returns first enemy/world. Use raycaster.intersectObjects(raycastTargets,false). `raycastTargets` can include player hit meshes. For player self, first hit may be own head? Since origin inside head, ray starts inside head geometry; Three.js intersects exiting? It may hit. Better exclude player hit meshes from `playerShoot` ray by separate `raycastTargetsForPlayer` = world + smoke + botMeshes. For bot, `raycastTargetsForBot` = world + smoke + botMeshes + playerHitMeshes. But if bot shoots, it includes all bot meshes, including self? Exclude by userData.bot.id != shooter.id loop. Bots may hit team mates if line; skip damage. Fine.
We'll maintain global `worldMeshes`, `smokeMeshes`, `botPartMeshes`, `playerHitMeshes`. Build target arrays dynamically:
```
function getRayTargets(forBot, botId, forPlayer=false){
 let arr=worldMeshes.concat(smokeMeshes).concat(botPartMeshes);
 if(forBot) arr=arr.concat(playerHitMeshes);
 return arr;
}
```
Need if worldMeshes includes smoke? We'll keep separate.

Need `shootRay(origin, dir, shooter)`:
```
const targets = shooter.type==='bot'? worldMeshes.concat(smokeMeshes).concat(playerHitMeshes).concat(botPartMeshes) : worldMeshes.concat(smokeMeshes).concat(botPartMeshes);
raycaster.set(origin, dir); hits = raycaster.intersectObjects(targets,false);
for hit of hits:
  const u=hit.object.userData;
  if (u.bot && u.bot.id===shooter.id) continue;
  if (u.player && shooter.type==='bot' && u.player===player) ... If shooter is bot, can hit player; if player? no player meshes.
  if (u.bot && u.bot.team===shooter.team) continue; // no friendly? but still bullet stops? In CS bullets can hit teammates? no. But as game, should not.
  return {hit, object:u};
return null;
```
Need world no userData. If hit.world, return.

Need for player shooting, if bot part hit but userData.bot.team===player.team, skip? But bullet should not pass through teammates maybe. We'll skip and continue? This may make bullets go through teammates. Fine. But if teammate body in front of enemy, no hit. okay.

Need damage:
```
function damageEntity(target, dmg, part, shooter){
 if (!target.alive) return;
 if (part==='head') dmg*=4;
 if (target.armor>0){
   const armorDmg = Math.min(dmg*0.5, target.armor);
   target.armor -= armorDmg;
   dmg = dmg*0.5;
 }
 target.hp -= dmg;
 if(target.hp<=0){ target.hp=0; killPlayer(target, shooter); }
}
```
Need if target is player: update HUD. If bot: maybe hide model? We'll set alive false, `bot.model.visible=false`, remove botPartMeshes? Could leave invisible.

Need kill rewards: shooter.money += 300 if shooter. For player, update HUD.

Need "kill feed" add message `T [AK-47] CT`.

Need "damage to player" red overlay.

Need "bot fire" aim and cooldown. Implementation:
```
function botThink(bot, dt){
 if (bot.reloading){ ...}
 if (bot.cooldown>0) bot.cooldown-=dt;
 if (bot.blind>0){ bot.blind-=dt; }
 // if target visible
 const vis = findVisibleEnemy(bot);
 if (vis){
   bot.target = vis;
   bot.reactionTime -= dt*1000;
   if (bot.reactionTime <=0){
      bot.yaw = lerpAngle(bot.yaw, desiredYaw, 5*dt);
      bot.pitch = lerpAngle(bot.pitch, desiredPitch, 5*dt);
      if (abs angle < 0.1) botFire(bot);
   }
 } else { bot.reactionTime = diff.reaction; }
 // movement
 moveAlongPath(bot, dt);
}
```
Need `findVisibleEnemy`:
```
function findVisibleEnemy(bot){
 const candidates = [];
 for (const b of bots) if(b.alive && b.team!==bot.team) candidates.push({pos:b.pos, chest:b.pos.y+1.3, ref:b, obj:'bot'});
 if(player.alive && player.team!==bot.team) candidates.push({pos:player.pos, chest:player.pos.y+1.3, ref:player, obj:'player'});
 const eye = bot.pos.clone(); eye.y+=1.6;
 let best=null;
 for c of candidates:
   const dx=c.pos.x-eye.x, dy=c.chest-eye.y, dz=c.pos.z-eye.z; dist=...
   if(dist>60) continue;
   // angle to bot forward
   const dir = new Vector3(dx,0,dz).normalize();
   const fwd = new Vector3(Math.sin(bot.yaw),0,Math.cos(bot.yaw)).normalize();
   const dot = dir.dot(fwd);
   if(dot < 0.15) continue; // ~80° fov
   // line of sight raycast top? Use raycaster to chest position. Need from eye to c.chest. If first hit is c object or none? If world blocks, no.
   const targetPoint = new THREE.Vector3(c.pos.x, c.chest, c.pos.z);
   const dirFull = targetPoint.clone().sub(eye).normalize();
   raycaster.set(eye, dirFull); raycaster.far=dist;
   const hits = raycaster.intersectObjects(worldMeshes.concat(smokeMeshes).concat(botPartMeshes).concat(playerHitMeshes), false);
   // If first hit object userData corresponds to candidate or enemy body? If world/smoke, no.
   if(!hits.length) return c;
   const h = hits[0].object.userData;
   if (h.bot && h.bot.id===c.ref.id) return c;
   if (h.player && c.ref===player) return c;
 return best;
}
```
But if candidate behind teammate, first hit teammate, not enemy. okay.
Need if smoke exists, raycaster hits smoke mesh and returns, blocking vision. Good.

Need bot movement path:
```
function setBotPath(bot, goalNodeIndex){
 const nav=currentMap.nav;
 const start = nearestNode(bot.pos);
 if (goalNodeIndex===null) return;
 bot.path = dijkstra(nav, start, goalNodeIndex);
 bot.pathIndex = 0;
}
```
Dijkstra:
```
function dijkstra(nav, start, goal){
 const dist = Array(nav.length).fill(Infinity);
 const prev = Array(nav.length).fill(-1);
 dist[start]=0; pq=[...]
 while... for link if dist[neighbor]>...
 reconstruct
}
```
Need nearest node: compute distance to nav nodes.

Bot movement along path:
```
function moveBot(bot, dt){
 if(bot.path.length===0 || bot.pathIndex>=bot.path.length){
   // choose defensive random? if no path, idle.
   return;
 }
 const targetNode = nav[bot.path[bot.pathIndex]];
 const dx=targetNode.x-bot.pos.x, dz=targetNode.z-bot.pos.z;
 if(Math.hypot(dx,dz)<1.2){ bot.pathIndex++; return; }
 const speed = bot.weapon==='awp'?3.0:3.6;
 const desired = new THREE.Vector3(dx/d,0,dz/d).multiplyScalar(speed);
 bot.vel.x = desired.x; bot.vel.z = desired.z;
 bot.yaw = Math.atan2(desired.x, desired.z);
 moveBody(bot.pos, bot.vel, dt, 0.35, 1.8);
 // avoid separation from other bots
 updateBotModel(bot);
}
```
Need `bot.vel.y` for gravity. moveBody handles.

Need if bot has no path due to no nav? We'll set.

Need "bots plant/defuse":
In `updateBotObjective(bot,dt)`:
- If T and hasBomb and at bomb site:
```
const site = nearestSite(bot.pos);
if(site && dist<5 && !game.bomb.planted && phase==='live'){
  bot.plantProgress=(bot.plantProgress||0)+dt;
  if(bot.plantProgress>=3) plantBomb(bot);
} else bot.plantProgress=0;
```
- If CT and bomb planted and at planted site: defuse. If defuseKit time=3 else 5.
Need bomb plant Bots have `hasBomb`; in startRound, allocate bomb. If player team CT, T bots one has bomb.

Need "bomb site nearest": `map.bombSites` with x,z,rad.

Need "Bomb has site" after plant: set `game.bomb.site = site`, `game.bomb.time=40`, `phase='bombPlanted'`, create mesh at site.

Need "roundTime" during bomb planted: The round timer can remain; bomb time controls. If bomb timer reaches 0, `endRound('T','炸弹爆炸')`. If all CT dead -> T. If all T dead and bomb planted? In CS, if all T dead before bomb explode/diffuse? Actually round continues until bomb explosion or defuse? If all T dead and bomb planted, CT must defuse; if T dead, bomb still explodes. We'll not end on all T dead after planted; instead continue. If all T dead and bomb not planted -> CT. If all CT dead and bomb planted -> T? T win because CT cannot defuse; but bomb could also explode. We'll end if all CT dead -> T. If all T dead and bomb planted, do nothing. Need code:
```
function checkRoundEnd(){
 if (phase==='roundEnd'||phase==='freeze') return;
 if(!game.bomb.planted){
   const tAlive = aliveBots('T').length + (player.team==='T'&&player.alive?1:0);
   const ctAlive = ...
   if(tAlive===0) endRound('CT','恐怖分子被消灭');
   else if(ctAlive===0) endRound('T','反恐精英被消灭');
 } else {
   const ctAlive = ...
   if(ctAlive===0) endRound('T','反恐精英被消灭'); // bomb still ticking? but win
 }
 if(roundTime<=0 && !game.bomb.planted) endRound('CT','时间耗尽');
}
```
For win by elim before plant, okay.

Need "score" update:
`endRound(winner, reason)`:
```
roundEnded=true; phase='roundEnd'; roundEndTimer=5;
const tWin = winner==='T'; if(tWin) game.scoreT++; else game.scoreCT++;
if (winner===player.team) player.money+=3250; else player.money += 1500 + Math.min(game.lossBonus*500,3000); 
```
Need lossBonus:
- `game.lossBonus` consecutive losses; after lose increment, after win reset 1? Use `consecutiveLoss`.
- Add bomb plant bonus if bomb planted: T team money +800 regardless winner? We'll add.

Need "Match end":
After update roundEndTimer <=0:
`if (game.scoreT>=8||game.scoreCT>=8) showMatchScreen(); else startRound();`

Need "startRound" uses `game.round++`, reset positions, bots. Need if first, create bots. Once match started, maintain bots. Use `createBotsIfNeeded()`.

Need menu start:
```
$('startBtn').onclick = ()=>{
 game.side = selectedSide; game.botCount = parseInt(...); game.difficulty = ...
 game.scoreT=0;game.scoreCT=0; game.round=0; game.matchEnded=false;
 createBots(); startRound();
 $('menu').style.display='none';
 $('hud').style.display='block';
 document.body.requestPointerLock();
}
```
Need selected side etc via buttons.

Need "map selection" from menu. Use mapDefs array. Buttons with names. On start, `const mapDef = MAPS[m.currentMapIndex]; loadMap(mapDef);`

Need "crosshair" style:
```
<div id="crosshair"><span></span></div>
```
CSS.

Need "HUD update" every frame:
- `$('hp').textContent = Math.max(0,Math.round(player.hp))`
- `$('armor').textContent = Math.round(player.armor)`
- `$('ammo').textContent = active.mag + '/' + active.reserve` or for grenades.
- `$('money').textContent = '$' + player.money`
- `$('timer').textContent = formatTime(...)`
- `$('score').textContent = game.scoreT + ' : ' + game.scoreCT`
- `$('bomb')` visible if hasBomb or planted.

Need "buy menu" references; if player equips primary, update.

Need "damage flash":
`$('damageOverlay').style.opacity = 0.4` for 0.2s; animate.

Need "flashbang overlay":
`$('flashOverlay').style.opacity = flashOpacity`.

Need "round banner" show big text "T 获胜 原因". Use CSS animation.

Need "progress bar": if planting/defusing.

Need "minimap" draw loop every frame? draw at 10 fps.

Need "sound functions":
Use `AudioContext` lazy. `playGunshot(wepType)`:
```
function playGunshot(){
 const ctx=getAudio();
 const dur=0.2; buffer noise; source; gain; lowpass filter;
 // for pistol/rifle etc vary frequency
}
```
Need many sounds. We'll implement concise:
- `playNoise(duration, filterFreq, volume, type)`.
- `playGunshot(weapon)`: create buffer noise, bandpass, gain envelope.
- `playReload`, `playFootstep`, `playBeep`, `playExplosion`, `playPlant`, `playDefuse`, `playHit`, `playHeadshot`, `playButton`.
Need maybe not all audio in first round but okay.

Need "WebAudio on pointerlock":
`document.addEventListener('click', () => { if(!audio) audio = new AudioContext(); audio.resume(); });`

Need "Game loop":
```
function animate(){
 requestAnimationFrame(animate);
 const dt = Math.min(clock.getDelta(), 0.05);
 if game.state==='playing':
   update(dt);
 renderer.render(scene,camera);
}
```
Need `game.state='playing'` after start. Pause/resume.

Need "pointer lock":
On canvas click:
```
if(game.state==='playing' && !player.alive?) requestPointerLock.
```
`document.exitPointerLock` when menu.

Need "Pause menu":
When pointer lock exits and game state playing, show pause overlay. Resume button requestPointerLock. But if user presses Esc, pointer lock exits. We'll show pause. If user clicks resume, request lock. Need not pause time? We'll set `game.paused=true`; update loop skip if paused. But pointer lock exit event can fire when opening buy? Buy menu doesn't require pointer lock? It can show while locked. We'll not pause on buy.

Need implement "mouse click" while pointer lock:
`document.addEventListener('mousedown', e => { if(e.button===0) mouseDown.left=true; ... })`
`mouseup`.
If buy menu open, click handled by buttons. Don't shoot if menu open.

Need "right-click" contextmenu preventDefault. For zoom:
```
if(mouseDown.right && activeWep.scope){ player.scoped=!player.scoped? Actually hold to zoom maybe. Use mousedown set scoped true; mouseup false.}
```
We'll make AWP zoom while holding right. Also maybe not toggle.

Need "player shoot" only if `mouseDown.left` and not menu. For automatic, each frame if auto. For semi, on mousedown. We can just in update if mouseDown.left use fire if cooldown <=0. For semi, need only one shot per click; set `weapon.auto ? ... : ...`. We can allow holding for semi? We'll use `fireHeld`.

Need "reloading" key R.

Need "switch weapon" uses number keys.

Need "drop weapon"? no.

Need "Bomb plant" key E. Use `keys['KeyE']`.

Need "buy menu" B key while in buy zone. Toggle.

Need "Maps definitions" update `game.state`.

Let's start writing final code mentally. Need ensure no syntax errors.

Potential issue with `THREE.MathUtils.lerp` etc. r128 has `THREE.MathUtils`. Use `Math` to avoid.

Need `lerpAngle` function:
```
function lerpAngle(a,b,t){
 let d=b-a; while(d>Math.PI)d-=2*Math.PI; while(d<-Math.PI)d+=2*Math.PI; return a+d*t;
}
```

Need "player death" from bot fire:
In `applyDamageToPlayer`:
```
player.hp -= dmg; $('damageOverlay').style.opacity=Math.min(1,0.3+dmg/100);
if(player.hp<=0){ player.alive=false; hidePlayerHitMeshes(); killFeed(...); }
```
Need if player dies, no pointerlock? We can remain.

Need "spectate":
If player dead:
```
const cams = aliveBots(player.team).concat(...) maybe follow alive teammate; 
if(cameraFollowIndex>=aliveBots.length)...
const target = aliveBots[followIndex];
camera.position.set(target.pos.x,target.pos.y+1.6,target.pos.z);
camera.rotation.set(target.pitch, target.yaw? But bot.yaw is atan2(dx,dz), camera yaw should be? Since Three camera forward -Z; if we set camera.rotation.y = target.yaw+Math.PI? Need not exact. We'll just set `camera.rotation.set(target.pitch, -target.yaw + Math.PI?` Hmm.
```
Bot yaw derived with forward +Z. Camera yaw convention -Z. If bot yaw=0 faces +Z. Camera yaw=0 faces -Z. Need camera.yaw = bot.yaw + Math.PI to face same direction? Actually rotate camera 180 to face +Z. Since camera rotation.y = π faces +Z? Let's verify: camera default -Z. Rotate y by π -> local -Z becomes world +Z. So camera.rotation.y = bot.yaw + Math.PI? Wait bot direction = (sin yaw, 0, cos yaw). Camera forward when rotation.y = φ = (-sin φ, 0, -cos φ). Set (-sin φ,0,-cos φ)=(sin yaw,0,cos yaw) => sin φ = -sin yaw, cos φ = -cos yaw => φ = yaw + π. Yes.
Pitch: camera pitch = -bot.pitch? Bot pitch positive up? We'll set approximate.

When following bot, use that.

Need "spectator HUD" maybe.

Need "map switch" creates bots. `loadMap` clears bots? In menu only start. We'll `createBots` after load.

Need "loadMap" before menu? We can load initial map for preview? Not necessary. But menu background? Could start with a scene? We can load menu map maybe? Simpler: Three scene empty until start.

Need "menu map preview" maybe not.

Need "banner overlay" with animation.

Need "round timer display" with bomb planted:
- If bomb planted, show bomb time (40) instead of round time.
- Include bomb icon.

Need "buyzone indicator" HUD maybe "购买区" if in buyzone.

Need "weapon switch IDs" `currentActive` etc. Need code.

Let's write core functions.

Weapon get:
```
function getActiveWeapon(){
 let id = player.loadout.active;
 if(id==='primary'){
  if(!player.loadout.primary.id) return WEAPONS['knife'];
  const w = player.loadout.primary;
  return {...WEAPONS[w.id], mag:w.ammo, reserve:w.reserve};
 }
 if(id==='knife') return WEAPONS.knife;
 if(id==='he'||id==='flash'||id==='smoke'||id==='molotov') return WEAPONS[id];
 const w = player.loadout[id]; if(w) return {...WEAPONS[id], mag:w.ammo, reserve:w.reserve};
 return WEAPONS.knife;
}
```
Need if active=primary and no primary? switch automatically:
`if(player.loadout.active==='primary' && !player.loadout.primary.id) player.loadout.active = sidePistol();`
`sidePistol` can be 'glock'/'usp' depending team. In default loadout after team selection.

Need "player loadout" key for pistol dynamic. If active stays `usp` for CT. On team switch in menu only. At start load.

Buy:
```
function buyWeapon(id){
 if(player.money < WEAPONS[id].price) return;
 if(id.includes('primary')){
   const old = player.loadout.primary.id;
   player.money -= price;
   player.loadout.primary = {id, ammo:WEAPONS[id].magSize, reserve:WEAPONS[id].reserve};
   if(!old) player.loadout.active='primary';
 } else if grenade -> player.loadout.grenades[id]++;
 else armor...
 updateHUD();
}
```
Need primary id names: 'ak47','m4a4','awp','p90','mp9','nova'. If player team CT, maybe cannot buy AK? But can if captured? We'll allow any primary but price. Better restrict? no.

Need "ammo" for primary after fired:
```
function consumeAmmo(){
 if(active==='knife') return;
 const id=active;
 if(id==='primary') { player.loadout.primary.ammo--; ...}
 else if(id in grenades) { player.loadout.grenades[id]--; if<=0 switch to pistol; }
 else { player.loadout[id].ammo--; }
}
```
Need grenade active: When throw, `player.loadout.grenades[active]--`. Then active to pistol. Need no reserve.

Need "reloading":
```
function startReload(){
 const wep=getActiveWeapon();
 if(wep.reload<=0 || wep.type==='grenade'||wep.type==='knife') return;
 if(wep.mag >= wep.magSize || wep.reserve<=0) return;
 player.reloading=true; player.reloadTime=wep.reload;
 playReload();
}
function updateReload(dt){
 if(!player.reloading) return;
 player.reloadTime-=dt;
 if(player.reloadTime<=0){
   const wep=getActiveWeapon();
   const needed=wep.magSize-wep.mag;
   const take=Math.min(needed,wep.reserve);
   if(active==='primary'){ player.loadout.primary.mag+=take; player.loadout.primary.reserve-=take; }
   else if(active in loadout weapons) ...
   player.reloading=false;
 }
}
```
Need for "primary" wep returns new object but after modifying loadout okay. Need if switch while reloading cancels.

Need "weapon firing animation":
In `fireWeapon`:
```
weaponRecoil = 1; weaponKick = 0.08; camera.pitch += (Math.random()-0.5)*recoil? Actually camera pitch kick upward: camera.pitch += wep.recoil * Math.random();
```
Need apply to player.pitch not camera? player.pitch used. `player.pitch = Math.max(-1.4, Math.min(1.4, player.pitch + wep.recoil*0.6))`.
Need viewmodel position y -= recoil.

Need "muzzle flash":
Could create a sprite/plane at gun muzzle. Simpler: create a PointLight? Too much. We'll create a small sprite with flash texture:
```
const flash = new THREE.Sprite(new THREE.SpriteMaterial({map: flashTex, color:0xffffaa, transparent:true}));
flash.position.set(0,0,-0.6); flash.scale.set(0.3,0.3,1); gunGroup.add(flash); setTimeout remove? 
```
Need remove sprite after 50ms. Use `muzzleFlashTimer`.
Create canvas radial gradient for flashTex.
`gunGroup` is attached to camera; we can add sprite at local pos.

Need "shell casing":
Spawn small box in world physics? Could be decorative. We'll spawn a tiny box with velocity, gravity, remove after 1s. Need not.

Need "bullet impact": `createImpact(pos, normal?)` spawn small sprite/box. We can use line? We'll add a small dark sphere at hit point, auto remove. Also play hit sound.

Need "explosion": Light + smoke/sprite.

Need "smoke mesh": Use `THREE.Sprite` or sphere. Since raycaster intersects Sprite? Sprite intersects ray? yes maybe. Use sphere for bullet block. Add sphere with MeshBasicMaterial color gray opacity 0.5, transparent, depthWrite false. It may occlude vision with opacity. Need if many spheres, okay. Smoke grows over time.

Need "flashbang effect": create light flash sprite? Use white overlay.

Need "grenade entity": Could use `THREE.Mesh(new THREE.SphereGeometry(0.1), black)` for HE, and a cylinder for flash/smoke? Use generic.

Need update grenades loop:
```
function updateGrenades(dt){
 for(let i=entities.length...){
  grenade.vel.y -= 10*dt;
  grenade.pos.addScaledVector(grenade.vel, dt);
  // collide with floor
  if(grenade.pos.y < 0.1){ grenade.pos.y=0.1; grenade.vel.y *= -0.3; grenade.vel.x*=0.8; grenade.vel.z*=0.8; }
  // walls collision skip
  grenade.fuse -= dt;
  if(grenade.fuse<=0){ ...; entities.splice(i,1); }
 }
}
```
Need point collision with wall boxes: if grenade inside a collider, push out. Simple:
```
for c of colliders:
 if point inside box:
  // push minimal axis based on previous? Use center to box faces.
```
But okay.

Need "fire zone for molotov": Add an array of fire patches with timer. Each patch mesh disc; update damage.
Maybe ignore molotov visual zones? But request. We'll create a group of small cone flames (orange translucent) at ground, with entity `fireAreas`. Update damage to players/bots within radius.

Need "flashbang blind":
For player, `if flashEnt.dist < 15` compute angle between camera forward and direction to flash. If facing flash, opacity = (1 - dist/15) * (0.5+dot*0.5). If not facing, lower. Overlay decays.
For bots, if within radius and facing, `bot.blind = 3 * (1-dist/15)*(dot+1)/2`.

Need "smoke fade": smoke entity `life`, `maxLife=15`; opacity = min(1, life/1) * max(.8) etc. If life <=0 remove from scene/smokeMeshes.

Need "round time freeze":
During freeze, roundTime = 8 (buy time). After freeze, phase='live', roundTime=105 (or 115). If bomb planted, bombTimer starts 40.
Need "freeze time" positions reset. Movement allowed? In CS freezetime no move. We'll freeze player and bots during freeze by returning. Buy menu open automatically? We'll let player move? Maybe no movement. For playability, allow movement but no combat? We'll keep movement disabled? No, if player can't move for 8s boring. But in CS freezetime, you can move in spawn. It's okay. We'll allow movement and bots wait. No shooting/grenades during freeze (cooldown).
Need game.phase='freeze'; after freezeTimer <=0, phase='live'.

Need "roundTime" in freeze maybe 8. In update:
```
if(phase==='freeze'){
  freezeTime -= dt;
  if(freezeTime<=0){ phase='live'; roundTime=105; }
} else if(phase==='live'){
  roundTime -= dt;
  if(roundTime<=0) endRound('CT','时间耗尽');
}
```
If bomb planted:
`phase='bombPlanted'; bombTime -= dt; if(bombTime<=0) explodeBomb();`
Need roundTime maybe still counts? Could ignore; bombTime controls.

Need "bomb planting" sets phase. After plant, roundTime maybe pause? We'll use bombTime.

Need "defuse" complete sets endRound('CT','炸弹已拆除').

Need "C4 mesh": create a box with blinking light at bomb site. `game.bomb.mesh` added to scene. In loop blink.

Need "play beep": if bomb time < 10 faster.

Need "Bot plant/defuse timer" progress; show if bot in progress? Not necessary but can.

Need "E key interaction" while holding E. We'll code.

Need "HTML menus":
Let's craft CSS. Keep minimal but polished.

`#menu`:
```
position:fixed; inset:0; background:rgba(10,10,10,.95); color:#d4c9a8; display:flex; flex-direction:column; align-items:center; justify-content:center; z-index:100;
```
Title, buttons, select cards.
Need map cards generated maybe static with buttons:
```
<div class="menu-card" id="map0">Dust2 Remake</div>
```
But code can dynamically.

We'll include static buttons in HTML:
```
<div id="mapChoices">
 <button class="choice mapChoice" data-map="0">Dust2 Remake</button>
 ...
</div>
<div><button id="sideT">恐怖分子</button><button id="sideCT">反恐精英</button></div>
<select id="botCount">...</select>
<select id="difficulty">...</select>
<button id="startBtn">开始游戏</button>
```
Need style selected.

`#hud`: Use corner panels.

`#buyMenu`: panel with list.

`#banner`: center big.

`#progressBar`: bottom center.

Need "killfeed": top right under score? CSS.

Need "minimap": fixed top right.

Need "crosshair": center.

Let's draft final code in one go. But final answer can't include all code if too long? 90k tokens enough. But final answer should be full. We'll include code in a code block. Might be huge ~40k tokens, okay.

Need make sure no "TODO". Need no external assets besides Three.js CDN.

Let's write final code with careful structure. Time to craft.

Attention: HTML code with inline script may contain `</script>` inside strings? No.

Need ensure Chinese text not causing encoding? UTF-8.

Let's proceed.