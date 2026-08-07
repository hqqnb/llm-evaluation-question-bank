The user is asking for a complete Counter-Strike-style FPS game that runs in the browser, delivered all at once. This is an extremely huge request. I need to deliver a complete, runnable artifact as a single deliverable. Given the constraints of this environment, the most practical approach is a single HTML file (self-contained) that uses raw WebGL or Canvas. Using Three.js from a CDN is an option, but the user said third-party libraries are my choice. However, to guarantee offline execution, I should consider writing my own rendering. Actually, if I use Three.js from a CDN, it breaks without internet. Writing a complete 3D FPS with raw WebGL is possible but extremely verbose. Alternative: raycasting engine (Wolfenstein-style) — but the user wants 3D scenes with slopes, height differences, and sniper lines. A raycaster can't handle slopes well.

Best approach: write a custom WebGL renderer with a compact engine, all in one HTML file. It's a lot of code but feasible. I need to be mindful of response length limits. The deliverable will be a large single HTML file. I should aim for a comprehensive but manageable implementation.

Let me plan the architecture:

**Tech**: single `index.html`, custom WebGL2 renderer, no external dependencies (guarantees execution). All sounds synthesized with WebAudio. Textures procedurally generated on Canvas.

**Core systems**:
1. Math library (vec3, mat4, quaternion-ish stuff).
2. WebGL renderer: mesh batching, per-vertex color + texture, directional light + simple fog. Static geometry baked into a single buffer per map. Dynamic entities (players as boxes/capsules, bullets, grenades) drawn as dynamic meshes.
3. Map format: list of boxes (AABB) + ramps (wedges) + materials. Collision: player capsule vs AABB, with slope support. Waypoint graph for bots.
4. Player controller: WASD, sprint (Shift? In CS, Shift is walking/silent movement; the user asked for both silent walk and sprint — Shift = silent walk, Ctrl = crouch, Space = jump, mouse look via Pointer Lock).
5. Weapons: pistol (USP/Glock), SMG (MP5), rifle (AK/M4), sniper (AWP), shotgun, plus grenades HE/flash/smoke/Molotov, knife. Recoil pattern, spread, reload, ADS (right-click for sniper scope; slight zoom for others).
6. Hit detection: hitscan raycast against AABBs and characters; headshot boxes; armor; damage falloff.
7. Game mode: bomb defusal. Round timer (e.g., 1:55), bomb plant (4-second hold at site), defuse (5s/10s with kit), round-end conditions, freeze time for buying, halftime? Keep it simple: first to N rounds, e.g., 8-round win (MR8 style) with team switch at halftime. Economy: starting $800, kill rewards, win/loss bonuses (with loss streak), weapon prices, armor, helmet, defuse kit, grenades. Buy menu (B key) during freeze time/buy time, only near buy zone (spawn).
8. Bot AI: waypoint nav, states: patrol/attack/plant/defuse/retreat; reaction time and aim error by difficulty; grenade usage (simple: throw HE at visible clustered enemies, occasionally smoke/flash when pushing). Bots buy weapons according to economy.
9. UI: HUD (health, armor, ammo, money, round timer, score, radar minimap, crosshair, kill feed, hit marker), buy menu, main menu (map selection, team, bot count, difficulty), pause menu, scoreboard (Tab), end-of-match screen.
10. Audio: WebAudio-synthesized gunshots (noise burst + click), footsteps, reload clicks, explosions, bomb beeps, plant/defuse sounds, round start/end stingers, ambient hum.
11. Effects: muzzle flash (sprite), tracers, impact particles, shells, smoke volume (billboard cluster), flashbang white overlay, fire patches from Molotov, blood decals (simple), damage indicators.

**Maps**: three maps:
1. **Dust2 replica** — this is the big one. I need to build a layout that approximates classic Dust2's topology. Classic Dust2 layout (top-down, T spawn south, CT spawn north-center):
   - T spawn: southwest area.
   - Long A: from T spawn east through "Long Doors" (double doors) to Long A corridor, corner ("Long corner"), then north up to A site via "A Long"/stairs; A platform with boxes; "A ramp" connecting CT spawn to A site.
   - Mid: from T spawn north through "Top Mid"/"Suicide" area, Xbox crate in mid, double doors ("Mid doors") in center of mid, CT side of mid connects to short ("Catwalk") via stairs, and to CT spawn.
   - Short A / Catwalk: mid → stairs up to catwalk → A short stairs → A site.
   - B tunnels: from T spawn west/northwest through upper tunnels, down to B site (B platform with boxes, B doors to CT spawn side).
   - CT spawn: north, connects to mid (via mid doors area), A ramp, B doors/CT-side tunnels.
   
   Build this with box geometry: walls, ramps, stairs, crates, arches (approximated with boxes forming an arch shape), doors with gaps. Provide named waypoints: TSpawn, CTSpawn, ALong, AShort, Mid, B, Xbox, Goose?, Pit (A pit — the user asked for "A 坑" (A pit) — classic Dust2's A site has "pit"? Actually "A 坑" might refer to the "pit" near A long entrance (the recessed corner at A long where players hide) — yes, "pit" is the sunken area at the A long entrance to A site. Include a sunken corner at A long entrance.), A slope (A ramp), mid doors (中门), double doors (双门 — Long doors), Xbox, B tunnels (B 洞), B doors (B 门), B platform (B 平台), CT base (警家 = CT spawn), long door (长门 = long doors? "长门" and "双门" both refer to doors; 双门 = mid doors? In Chinese CS terminology: 中门 = mid doors (the double doors in the middle of mid), 双门 might also refer to long doors... actually 长门 might be "long doors" and 双门 might be... hmm. Include both: long doors (long A entrance) and mid double doors. That covers it.)
   
   Sniper lines: long A straight, mid (through door gap), from Xbox to mid doors.
   
   Approximate scale: CS units → meters. Dust2 is roughly 64x64 meters in playable area? Actually larger; long A from doors to A site is about 30m. Build at plausible scale: total map about 90x90 meters.

2. **Map 2: "Ruin" / Temple** — close-quarters jungle/temple map with verticality, tight corridors, a central courtyard, two sites, faster pace. Different rhythm: close-range combat, more cover.

3. **Map 3: "Compound" / Industrial** — long sightlines, containers, warehouse, night theme; medium-long range, flanking routes through vents/alleys.

Each map: geometry list + spawns + bomb sites + waypoints + nav links + bot strategy hints (site assignment, route selection).

Given the enormous scope, I need to write very compact but complete code. This will be a very long response. Let me do it strategically: write dense, efficient code. Probably 3000-5000 lines. That's a lot but necessary. Actually, I should compress: use terse variable names in engine internals, but keep it readable enough. The response token limit is fairly large (I can output quite a lot), but I need to be careful. Aim for a single HTML file of about 200-300KB? That might be too much for one message. Hmm. Output limit: can probably output about 30k-60k tokens. A well-compressed implementation might fit in about 25k-40k tokens. Let me design to be economical:

- Single-file HTML with embedded JS.
- Compact math.
- Geometry builder helpers: `box(x,y,z,w,h,d,mat)`, `ramp(...)`, `stairs(...)`.
- Map defined in data-driven functions.
- Renderer: one shader, vertex format: position, normal, color/UV, texture index → pack material ID into attribute, sample texture atlas. Procedural texture atlas generated on a single canvas (sand wall, sand floor, stone, crate, metal, door, etc.). Use NEAREST-ish filtering for crispness? Use linear.

Simplification: per-face color + one detail texture? To keep visuals decent ("not programmer-art placeholder blocks"), generate a texture atlas with several procedural textures: plastered sand wall, sand floor, stone brick, wooden crate, metal panel, dark door, concrete, grass?, etc. Apply per-face UVs with world-scale tiling.

Lighting: simple directional + ambient, plus baked per-vertex AO-ish darkening? Simple approach: face normal shading + distance fog. Add a cheap "sun shadow"? Skip real shadows; use fake blob shadows? Skip. Add vertex AO: darken vertices near the ground? Skip. Face-based shading + slight per-face variation will look decent.

Characters: draw as multi-box humanoids (torso, head, arms, legs, gun) with team colors and simple walk animation (leg swing). View model: draw gun in screen space (attached to camera) — build gun models from boxes, with recoil kick animation, muzzle flash.

Collision: player = cylinder (radius 0.35, height 1.8, crouch 1.2). Movement with AABB collision resolution against world AABB list and ramps (treated as AABB steps? Ramps need slope walking). Implement ramp support: represent ramps as prisms; compute ground height at (x,z) from floor faces: for simplicity, each map provides a `groundHeight(x,z)` function computed from a list of "floor boxes" (top faces) and ramps. Player vertical: gravity, ground snap. Horizontal collision against wall list (AABBs marked as walls, full-height or above step threshold). Step-up: allow auto step-up of 0.4m (CS is 18 units ≈ 0.46m).

Simpler and more robust approach: voxel-ish? No. Use AABB list with axis separation: move X, resolve; move Z, resolve; then Y with ground detection. Ramps: treat as special AABBs with height function; when horizontally overlapping ramp AABB, ground = ramp top at that position. That works.

Bullets: raycast against world AABBs (slab method) and entity capsules (approximated as spheres/cylinders). Return nearest hit.

Bot pathfinding: waypoint graph with A* or Dijkstra. Waypoints placed at key locations per map with links. Bots pick routes to objectives. Combat: if enemy visible (raycast LOS), stop/strafe, aim with error, fire with reaction delay by difficulty. Grenade usage: occasionally throw HE at enemy position if within range and LOS-ish; flash before pushing site (throw in direction); smoke at mid doors on Dust2 (scripted-ish: if difficulty ≥ hard and near mid doors, throw smoke at doors). Keep it simple but functional.

Round flow: freeze time (buy, 5s... CS is 15s; use 8s), round time 1:55, bomb timer 40s, halftimes with team swap, first to 8 wins (configurable). End-of-round conditions: elimination, bomb explode, defuse, timeout (CT win if bomb not planted... in CS if bomb not planted T loses on timeout).

Economy: start 800. Kill rewards: rifle/SMG 300, pistol 300, AWP 100, shotgun 300? CS: AWP 100, rifle 300, SMG 600 (MP5 600), shotgun 900? Simplify: pistol 300, SMG 600, rifle 300, AWP 100, shotgun 900? Actually CS2: shotgun 900? OK. Win: 3250, loss: 1400 base +500 per loss streak up to 3400. Plant bonus +800 for T even on loss; defuse? Keep: plant gives planter +300, T team gets 800 even if they lose after plant. Buy anywhere? No — buy zone only during freeze/buy time near spawn. Bots auto-buy at freeze.

Weapons:
- Pistols: USP-S (CT) / Glock (T) $200/$200? Actually both $200; P250 $300? Keep: USP, Glock, Deagle $700.
- SMG: MP5 $1500.
- Rifles: AK-47 $2700 (T), M4A4 $3100 (CT), also FAMAS/Galil? Skip for size, or include cheap rifles: Galil $1800, FAMAS $2050 — good for economy rhythm. Include if space allows. Maybe include.
- Sniper: AWP $4750, also Scout $1700? Include SSG08 $1700 for sniper variety.
- Shotgun: Nova $1050? XM1014 $2000? Include one: Nova $1050 (pump) or M3. Go with "M3" shotgun $1800? OK: Nova-style pump.
- Grenades: HE $300, flash $200, smoke $300, Molotov $400 (T)/firebomb $600 (CT) — same $400 is fine. Armor $650, helmet $1000 (combined), defuse kit $400.

Weapon stats: damage, penetration? Skip wall penetration. RPM, mag, reserve, reload time, spread, recoil, price, kill reward, move speed factor, range falloff.

Sound synthesis: function `sfx.shot(type)` etc., using noise buffer + filter + envelope; footsteps = short filtered noise taps; explosion = lowpassed noise boom; flash = high-pitched ringing; bomb beep = oscillator; round stinger = simple chord arpeggio.

UI: DOM-based HUD (divs) + Canvas radar. Crosshair dynamic. Kill feed. Buy menu DOM. Scoreboard DOM. Menus DOM. Damage flash vignette. Flash white overlay. Scope overlay for AWP.

Radar: draw map top-down from geometry (pre-computed per-map minimap canvas: draw floor/wall rectangles), player as dots, bomb as icon, only teammates visible (no enemies unless spotted? Simple: show teammates + bomb carrier).

Pause: Esc opens menu (release pointer lock). Resume, restart, back to menu.

Observation: on death, spectate a teammate (cycle with click), or free view? Simple: spectate next living teammate; if none, wait for round end.

Also "sprint": CS has no sprint; user asked for 冲刺 (sprint). Add Shift = walk (silent), and... sprint? Maybe double W? Add: hold Shift = silent walk; no sprint to keep CS feel? User explicitly asked for 冲刺. Compromise: sprint exists but greatly increases spread and sound radius; default speed is CS-like. Implementation: Ctrl crouch, Shift walk, Alt? Hmm. Sprint = Shift double tap? Simplest: pressing Shift while moving = walk (silent). Sprint: press Q? No. Sprint = Ctrl? No that's crouch. Use "R"? No. Decision: Shift = silent walk; sprint = hold "E"? Weird. Many browser CS clones use Shift walk; sprint is... W+W tap (double-tap forward) = sprint, like some games. Actually CS itself has no sprint; but the requirement says implement it. Make double-tap W = sprint toggle? Simpler: hold Shift = walk; sprint = mouse wheel? No.

Alternative: Shift = sprint, Ctrl = walk? In CS, Shift is walk. But user lists both 静步 and 冲刺. Mapping: Shift = silent walk (静步), ` (or Caps?)... Let me do this: double-tap W/A/S/D = sprint in that direction (like CoD momentary). Implementation: double-tap W within 300ms → sprint while W held. Sprint increases speed 25%, big spread, loud footsteps, disables for 0.3s after firing. Good.

ADS: right-click — sniper scope (with overlay), other weapons slight zoom (FOV -10) with small spread reduction. OK.

Bot difficulty: Easy/Normal/Hard/Expert → reaction 500/320/200/120ms, aim error in degrees, spray control, grenade probability, route variety, hearing? Keep: reaction, aim sigma, firing delay, HP? No. Also bot count setting 1-8 per team? User: bot count setting. Provide 1v1..5v5; player is always on chosen team; fill with bots. Also "practice" alone? Keep team sizes: total bots configurable.

Map selection: three maps. Team selection: T/CT/random.

Now Dust2 geometry. Let me design coordinates. Units in meters. Axis: x east, z south (map top-down: north = -z). Player height 1.7 eye.

Classic Dust2 orientation: T spawn south, CT spawn north, A site northeast, B site northwest, mid running north-south in center.

Let me lay out a simplified but faithful plan (about 100x100m). Coordinates (x,z), north = -z.

- T spawn: rectangle x∈[-10,15], z∈[38,52]. Walls around, exits: north to upper mid (gap x∈[0,6] at z=38), west to tunnels (gap x=-10, z∈[40,44]), east to long doors (gap x=15, z∈[42,46]).
- Long A: corridor east from T spawn: x∈[15,22] z∈[40,48] to long doors (double door structure with two door openings? Classic long doors: two door panels with gap in middle when open; doors closed at round start? In CS, long doors are closed at round start, T opens them. Model as two sliding doors that open on touch/use? Simpler: closed double doors that open when player presses E near them or on collision? CS: doors open on push. Implementation: door entity that opens when pushed (collides) — nice detail but complex. Simplify: doors start closed; any player walking into them opens them (slides into wall) with creaking sound. Implement as animated AABB that becomes non-colliding when open. Apply to long doors and mid doors. Mid doors: CT can peek through gap. Classic mid doors have a gap? Two doors with small gap in middle. Model mid doors as two doors with 0.35m gap between (bullets pass, players can't without opening; doors open on push).
- After long doors: "Long A" long straight corridor running north: x∈[18,24], z from 40 down to 8 — long sniper corridor. At north end, west turn to A site: "Long corner"/"Blue?" then stairs up to A platform. A pit: sunken area at A long entrance near A site (the pit). A site platform elevated +1.2m with stairs ("A long stairs").
- A site: x∈[8,26], z∈[-6,8] elevated platform with boxes (default site box, "A default"), goose position? Keep: site box, ramp entrance from CT (A ramp sloping from CT spawn up to site... actually A ramp goes down from CT to site? CT spawn is higher; A site is raised relative to long; CT ramp connects. Simplify heights: ground 0; A site platform +0.9; CT spawn 0; A ramp from CT (0) to site (0.9) slope.
- Mid: from T spawn north: "top mid" x∈[-2,4], z from 38 down to 20, with stairs down ("suicide" is open area west?). Xbox crate at z≈18, x≈1 in mid (crate 1.5m). Mid continues north to mid doors at z≈2, double doors; CT side of mid opens to CT spawn; catwalk/short: from mid at z≈14, stairs up east to catwalk (raised walkway) leading northeast to A short stairs up to A site.
- Short A: catwalk x∈[4,10] z∈[6,12] raised +1.5? Then stairs down/up to A site.
- CT spawn: x∈[-6,10], z∈[-16,-2]. Connects: south to mid doors, east to A ramp, west to B doors/CT-side tunnels.
- B tunnels: from T spawn west: upper tunnels x∈[-24,-10] z∈[38,44] corridor, then turn north and down to B site: tunnel corridor x∈[-26,-20], z from 40 to 6 with low ceiling (2.2m) — classic tunnels are low. Exit at B site.
- B site: x∈[-26,-12], z∈[-8,6], with platform/boxes, "B doors" to CT spawn on northeast side (door structure), back tunnel exit, window? Keep: B site box, platform.
- Also "window"/"CT" overlook? Skip.

Sniper lines: long A corridor (30m+), mid from top mid through mid doors gap to CT (40m), Xbox to mid doors, B tunnels straight.

Waypoints: TSpawn, LongDoors, LongA, LongCorner, ALongStairs, APit, ASite, AShort, Catwalk, MidTop, Xbox, MidDoors, CTSpawn, ARamp, BDoors, BTunnels (multiple), BSite, TunnelsT... with links and cost; bots A*.

This is heavy but doable with builder helpers. Write `M` (map builder) with functions: wall(x1,z1,x2,z2,h,mat) → box; floor areas; crate(x,z,s); ramp; stairs sequence; door entities.

Maps 2 and 3 simpler: reuse helpers, about 60% size of Dust2.

Map 2 "Ruin" (temple): central courtyard with broken pillars, A site east temple with stairs, B site west crypt, surrounding corridors, more CQC, jungle green/stone palette, night? Daytime jungle. Routes: T south → three lanes; CT north.

Map 3 "Depot" (industrial night): container yard with long lanes, warehouses, train tracks; medium-long range; two sites inside warehouses; flanking via vents. Night blue fog, floodlights (fake light cones as translucent geometry? Skip, just bright patches).

Given size constraints, maps 2 and 3 can be built with compact procedural-ish layouts using the same helpers.

Bots per map need nav graph — generate waypoints semi-manually per map. To save code, define waypoint list + links as arrays.

OK, engine details:

Renderer: WebGL1 for compatibility. One program: attributes pos(3), normal(3), uv(2), tint(3)? Use texture atlas 1024x1024 with 8x8 grid of 128px tiles? Procedurally generate about 12 textures at 128px each on offscreen canvas, pack into atlas, compute per-material UV offset. Vertex format: Float32 interleaved [x,y,z,nx,ny,nz,u,v,r,g,b] where rgb is tint multiplier. Draw static map as one big buffer. Dynamic objects (characters, grenades, view model, doors, shells, particles?) — particles as point sprites? Use a second small dynamic buffer updated per frame for particles/tracers (GL_LINES for tracers with additive blending).

Entities drawn as boxes: build small mesh per entity type per frame? Better: pre-build unit cube mesh; per draw, set uniform model matrix + tint + material. Draw calls: about 10 boxes per character × up to 10 characters = 100 draws + map 1 + effects. Fine.

Shader:
```
attribute vec3 aP; attribute vec3 aN; attribute vec2 aT; attribute vec3 aC;
uniform mat4 uM,uVP; varying...
color = texture(atlas, aT)*aC * lighting(n) * fog
```
Lighting: `float l = max(dot(n, sunDir),0.0)*0.55 + 0.45;` plus slight top-face boost. Fog: mix to fogColor by distance.

Muzzle flash: additive-blended billboard quad at muzzle; light? Skip dynamic lights, just flash quad + brief screen-space brightness? OK.

Smoke: spawn N=40 grey billboards with life, growing, drifting; render with alpha blending, depth write off. Smoke occlusion for AI LOS: check if ray passes within radius of any active smoke → block LOS. Same for player visibility (visually blocked by billboards).

Fire (Molotov): fire patch = cluster of fire billboards on ground area for 12s, damages entities inside.

Flash: if flashbang detonates with LOS to player camera, compute white overlay alpha based on angle/distance; also bots: if bot has LOS to flash within range → blinded state (wander/no shoot) for duration.

Grenade physics: sphere with gravity, bounce on AABB collision (reflect velocity *0.4), fuse: HE 1.8s? CS HE about 1.7? Use 1.6; flash 1.7; smoke 1.5 after landing? Smoke deploys on first strong hit or 1.5s; Molotov breaks on impact → fire patch.

Throw: initial velocity = aim direction * (12 + runup?) + player velocity*0.6, plus upward bias? CS throws mostly straight. v = dir*14 + up*2? Use dir*13 + playerVel*0.8 + up*1.5.

Bomb: T player (or bot) carries; plant: stand in site, hold use? CS: hold use for 4s (now 3s? Classic 3s? CS 1.6 was 3s, GO is 3s). Use 3s plant, 5s defuse without kit, 3s? CS defuse is 10s without kit, 5s with kit. Keep that. Key: E hold (use). Bomb drops on carrier death (dropped bomb visible, T can pick up with E). Round timer 115s (1:55). Bomb timer 40s, accelerating beep.

Observation: on death → spectate teammate (click cycles), or free view? Simple: spectate next living teammate; if none, wait for round end.

Also "sprint": as decided above.

Now bot AI detail (per tick, think every 100ms):
- Sense: visible enemies = LOS + within FOV? Bots have 360 hearing? Give: visible enemies via LOS ray within 60m; hearing: enemy fired within 25m or footsteps within 12m → lastKnown position.
- States: PATROL (follow route to objective), ENGAGE (strafe + shoot), PLANT, DEFUSE, GUARD (guard planted bomb near bomb), RETREAT? Keep: PATROL/ENGAGE/PLANT/DEFUSE/GUARD.
- Route: at round start, T bots pick site with weights (e.g., 2 A-long, 1 mid, 2 B, etc., random per difficulty), compute waypoint path; rush or default pace (walk some). On reaching site area with bomb → plant (bot with bomb). Others take defensive positions at site (defensive waypoint with facing). CT bots: assign positions: A ramp/long, mid doors, B doors, etc.; on bomb plant → retake: path to site via nearest; on bomb dropped → go there.
- Combat: if visible enemy: turn toward predicted position with aim error by difficulty; fire after reaction time; burst fire with recoil; occasionally crouch (hard+); if low HP and outnumbered → retreat to cover waypoint? Skip retreat, keep simple: backpedal while shooting.
- Grenades: on engage start, if distance >8m, probability (difficulty-based) to throw HE at enemy position; flash when pushing site entrance (pre-computed "flash spot" per map: when bot at waypoint X targeting site, throw flash toward site); smoke at mid doors (Dust2) if bot passes top mid (hard). Implement generic: map defines `smokeSpots` and `flashSpots` arrays {pos, target}. Bots use when near pos with probability.

Bot movement: follow path points; arrive within 0.6m → next; strafe jitter during combat; jump? Rarely, skip except for map-specific (skip).

Bot buy: at freeze, per money: priority: if hard/expert and money ≥ 4750+armor → AWP (one per team?), else rifle + armor + helmet + grenades; if poor → pistol/deagle; SMG if mid money.

Player buy menu: B opens during buy time and near spawn (within buy radius). Categories. Also auto-buy? Add "1. auto-buy" quick key? Skip, keep menu.

Scoreboard Tab: player list with K/D/money? K/D + ping fake.

Kill feed top-right: "A [weapon] B" with HS icon.

Round messages: "Round Start", "Bomb has been planted", "Counter-Terrorists Win", etc., with stinger.

Match end: overlay with stats, buttons: rematch, change map (back to menu).

Pause: Esc toggles menu overlay (pointer lock released). Buttons: resume, restart round? Restart match, back to main menu, settings (sensitivity slider, volume).

Settings: sensitivity, volume — save in memory.

Radar: top-left 180px canvas: pre-render map footprint (draw walls from geometry list scaled), rotate? CS radar is static north-up with player arrow. Keep static, player-centered? Simple: whole map fits, player as arrow, teammates as dots, bomb as blinking red when planted/dropped.

Also HUD: bottom-left HP/armor bars; bottom-right ammo; top-center timer/score/bomb state; money; weapon name; buy hint.

Crosshair: dynamic gap based on spread; hit marker X flash; damage numbers? Skip.

Now view model: draw gun from boxes attached to camera with bob and recoil kick and reload dip; muzzle flash quad at muzzle. ADS: sniper → hide view model, show scope overlay (DOM circle) and zoom FOV 20; rifle → FOV lerp -8, view model centered.

Shells: small yellow box particles ejecting with gravity, bounce once, fade.

Tracers: line from muzzle to hit point, additive, 40ms.

Impact: particle burst (dust color by material) + decal? Skip decals (time), particles suffice. Add bullet holes as small dark quads stuck to surface for up to 50 — nice touch, implement as oriented quads slightly offset along normal. OK, cheap.

Blood: red particles on character hit; on death, ragdoll-ish: fall over (rotate body to lying position) keep corpse as lying box for the round.

Character model: boxes: legs (2), torso, arms (2, holding gun forward), head, gun. Team color: T = tan/militia, CT = navy. Walk anim: swing legs with sin(time*speed). Death: fall over (rotate body to lying position). Keep simple.

Draw distance and fog per map theme.

Performance: static buffer maybe 30-60k triangles — fine.

Audio: create AudioContext on first user gesture. Master gain. Functions:
- noiseBuf cached.
- shot(type): noise burst + click osc; distance attenuation for others.
- footstep(surface?): short noise lowpass, volume by speed/crouch.
- reload: sequence of clicks via setTimeout.
- explosion: big lowpass noise + sub sine down.
- flash: high sine ring.
- beep: square osc 880/1760.
- plant/defuse: beep pattern.
- stinger: win/lose chord.
- ambient: low brown noise loop, low volume.
- hit marker tick.
- headshot "ding"? Classic CS has metallic headshot "tink". Add.

3D sound: simple stereo pan by relative angle + distance attenuation (no HRTF). Compute pan from dot(right, dirTo). Use StereoPannerNode.

Now code structure in single file:

```
<!DOCTYPE html><html><head><meta charset>... <style>...</style></head>
<body>
 <div id=menu>...</div>
 <div id=hud>...</div>
 <canvas id=gl></canvas>
 <canvas id=radar></canvas>
 ...
<script>
// ===== math
// ===== audio
// ===== textures
// ===== GL renderer
// ===== weapons data
// ===== map builder + 3 maps
// ===== entities (players, grenades, bomb)
// ===== game/round/economy
// ===== bot AI
// ===== input
// ===== UI
// ===== main loop
</script>
```

Given the massive scope, I'll write as tight as possible while keeping completeness. Let me draft key algorithms.

**Collision**:
World = list of solid AABBs {x1,y1,z1,x2,y2,z2, mat, ramp?}. Ramps stored separately {x1,z1,x2,z2, y0,y1, axis, dir} → height function.
Player move:
```
tryMove(dx,dz): 
 for axis in [x,z]: pos+=d; for each nearby solid: if cylinder intersects AABB (expanded by r) and solid top > feet+step and bottom < head: push back along axis.
```
Step-up: after horizontal block, check if raising by stepUp would clear and there's headroom → allow? Simpler: rely on ramps/stairs built as successive step boxes each ≤ 0.4 tall; horizontal collision only blocks if boxTop - feetY > 0.42. So stairs with 0.3 risers are walkable.

Ground: groundY = max over floor candidates: top face of solid containing (x,z) with top ≤ feet+0.42; ramp height; also standing on boxes. Compute each frame; if feet ≤ groundY+eps and vy ≤ 0 → grounded, feet=groundY.

Gravity vy -= 20*dt (CS-like gravity ~ use 19.6). Jump vy=5.4 (≈0.75m? CS jump ~1.5m: v=sqrt(2*g*h)= sqrt(2*19.6*0.9)? CS jump height about 64u=1.6m → v= sqrt(2*19.6*1.6)=7.9. Use g=22, v=8.2 → h=1.53. OK.

Speeds: walk 4.8 m/s (rifle ~200u/s≈5.1? AK 215u/s=5.46m/s). Base 5.2*weaponFactor; walk (Shift) 2.4; crouch 2.0; sprint 6.8.

**Raycast vs world**: slab test per AABB, nearest t; also ramp prisms: approximate ramp as AABB for bullets (acceptable) — but bullets through ramp wedge gaps look weird; acceptable for scope? Sniper lines don't cross ramps. Keep AABB-only collision for bullets and grenades; ramps only affect movement ground height and are drawn as wedges. Grenade bounce: treat as AABB too. OK.

**Raycast vs player**: capsule → approximate: sphere at chest (r .45) + sphere at head (r .22 at eye-0.1) + legs? Use two spheres: head sphere (headshot) center at feet+1.62 r 0.16? and body capsule as segment-sphere: test segment vs vertical segment (feet+0.1..feet+1.5) with radius 0.38 via closest approach of segment-segment. Implement segment-segment distance; if < r → body hit. Head: segment-sphere. Compare t.

**Damage**: base dmg * falloff * (head ? hsMult : 1) * armor calc. Armor: if helmet or (headshot needs helmet), absorb 50% of damage into armor loss (CS: armor absorbs ~50%). Implement: if armor > 0 and (not head or helmet): dmgToHP = dmg*0.5? CS formula: absorbed = dmg*armorRatio(0.5), hp = dmg - absorbed... actually CS: if armored, damage is reduced: newDamage = damage * 0.5? Roughly: with armor, HP damage is about 50%? Let me do: with armor applicable, hpDmg = dmg*0.55, armor -= dmg*0.4. Headshot without helmet: full*4? AK dmg 36, HS x4 = 144 kill. Use hsMult 4 for most, AWP 4 (459*? AWP 115*4). OK.

**Recoil**: per-weapon pattern: pitch kick per shot + diminishing sine-ish horizontal; spray increases spread. Track shotsFired; view punch += pattern[i]; recover over time. Spread = base + move + air + spray.

**Weapon switch**: slots 1 primary, 2 secondary, 3 knife, 4 grenades (cycle with G? CS: G drop). Keys: 1/2/3/4, wheel cycle grenades? 4 selects grenade, then click cycles? Keep: 4 cycles through owned grenades. Q swap previous? Skip. G drop weapon (drops as pickup entity!). Drop/pickup: dropped weapons as world entities, E or walk over? CS: E to pick up? Walk over bomb to pick up; weapons: E to pick up or auto? CS uses E. Implement E pickup nearest within 1.5m. Also dropped bomb.

**Use key E**: open door (push-to-open handled automatically), plant (hold), defuse (hold), pickup.

**Doors**: entity {aabb closed, openOffset, state, t}. On collision by player (overlap while closed) → start opening (slide along axis), sound. After open, no collision. Mid doors same. Long doors same. B doors too. Doors drawn as box with door material.

**Bomb plant**: player in site volume, grounded, holding E, not moving fast? CS allows moving? Must stand? In CS you can plant while crouched/still; require speed < 1. Progress bar UI. On complete: bomb entity planted at position, timer 40s, sound, message, T win condition changes.

**Defuse**: CT near planted bomb (<1.6m), LOS-ish (skip), hold E; progress; complete → CT win.

**Round end**: message + freeze 4s showing outcome, then reset next round (positions, bomb, dead revived, keep money, weapons? CS: keep weapons if survived; dead buy new. Dropped weapons persist? Simplify: survivors keep loadout; dead reset to pistol? CS dead lose all → buy. Implement: survivors keep weapons + ammo refilled? CS refills ammo? No, but keep mags; keep and refill reserve. Dead: lose everything (must buy), keep money of course.

Halftime: after round 7 (of MR7? First to 8, half at 7 rounds played? Match: first to 8 wins, max 15 rounds, halftime after round 8? CS MR15 halves at 15... For short: play 8 rounds per half? First to 8 wins, halftime after 8 rounds played with team swap (economy reset). If 8-8? Sudden death next round wins? Cap: first to 8; if 7-7 after 14... hmm: play up to 16 rounds, halftime after 8, first to 8.5? Simplify: first to 8 wins; halftime after 8 rounds played (score could be 8-0 → match already ended). So: match = max 15 rounds? Let me do: half length 8 rounds; team with more wins after 16 rounds? Tie possible. Simplest and robust: first to 8 round wins; swap sides when totalRounds == 8 (if no one reached 8 yet); if tied 8-8 after 16 → overtime single round? Declare draw? Let me do: first to 8; swap after 8 rounds played; if tied 8-8 after 16 → sudden death rounds until someone leads (no more swaps). OK.

Team swap: player team swaps? In CS, you swap sides. Player's team swaps too (announce). Bots swap accordingly.

**Buy zone**: within 8m of own spawn center & freeze time + 10s after round start? CS buy time = freeze time only? CS buy time is about 20s into round near spawn. Implement buyTime = freeze (8s) + 12s after round start, must be within spawn area.

**Bot count**: menu slider bots per team 0-5? Player + bots: e.g., team size 1-5 each. Provide "team size" 1..5 (player included on chosen team; other team all bots; own team filled with bots).

Difficulty: 1-4.

Now maps 2 and 3 design (compact):

**Map 2: "Ruin" (temple ruins, jungle day)** 80x80.
- T spawn south plateau; CT spawn north temple steps.
- Mid: central courtyard with broken pillars (cylinders approximated by boxes), fountain block; mid leads to "Hall" to A (east) and crypt stairs to B (west).
- A site: east temple raised 1.2, two stair entries (from mid hall, from east flank alley), cover: altar box, columns.
- B site: west sunken crypt (-1.2), stairs entry from mid and from west alley; cover: sarcophagus boxes, low walls.
- Flank: east alley connects T side to A rear? and west alley to B; CT connectors between sites behind sites.
- Rhythm: CQC, many corners, few long lanes (mid courtyard ~25m).

**Map 3: "Depot" (container yard/night industrial)** 90x90.
- Long sightlines: container rows forming lanes; A site = open loading dock with crane base boxes; B site = inside warehouse with catwalk? Height: warehouse floor +0? Add roofed warehouse (ceiling) for B; A is open with long lane from T (sniper line 45m) and CT.
- Mid = rail track cutting across map with train car blocks (duck under/around), connecting both flanks.
- Vents/flank: side maintenance alley.
- Night theme: dark blue fog, brighter ground patches.

Both need waypoints and sites and spawns.

Given code budget, define maps as functions returning {solids, ramps, decor?, spawns, sites, waypoints, links, theme}.

Waypoint format: `W=[ [x,z], ... ]`, links `L=[[i,j],...]` (bidirectional), plus named indices via object. Bot path: Dijkstra on links with distance cost; recompute occasionally.

Also need "defensive positions": per map: ctSpots: list of {wp, face} for A/B/mid; tEntrySpots per site for post-plant guarding.

OK, and "sniper lines" requirement is met by geometry (long straight corridors).

Now, let me think about view model and character mesh construction.

`meshFromBoxes(list)` builds arrays: list of [cx,cy,cz,sx,sy,sz, mat, tint?] → push 24 vertices per box with UVs tiled by size.

Character drawn per frame: compose boxes in local space with leg swing; transform by yaw; build into dynamic buffer per frame (few characters → fine). Or draw per-box with model matrix uniform: implement `drawBox(model, mat, tint)` using unit cube VBO with per-face normals and UVs; set uniforms per call. That's simplest: one cube VBO, uniform uModel, uMat (uv offset/scale for material), uTint. Static map uses big buffer with baked transforms (no per-draw uniforms). Both via same shader (uModel identity for static path? Static uses pre-baked positions and per-vertex UVs; dynamic uses unit cube with uModel and per-face UVs computed in shader? Unit cube UVs 0..1 per face, then scale by uMatRepeat). Simplify shader: attributes pos/normal/uv/tint; uniforms uModel, uUVScale (tile), uUVOff (atlas tile), uTint, uUseAtlasTile? For static bake final uv into atlas coords; for dynamic compute uv = uUVOff + uvLocal*uUVScale. Add uniform flag uMode: if 1, use attribute uv directly; if 0, compute from local. Implement: `vUV = mix(uOff + aT*uScale, aT, uMode)`.

Materials (atlas tiles): 0 sand plaster wall, 1 sand floor/dirt, 2 stone brick, 3 crate wood, 4 metal panel, 5 dark door metal, 6 concrete, 7 brick red, 8 roof/tile?, 9 grass?, 10 gravel, 11 container colors (use tint), 12 marble/pillar, 13 rail/dark metal, 14 tarp? Keep 12-14 tiles. Generate procedurally with noise on 128px tiles.

Sky: clear color gradient? Draw a big sky: use fog color + clear color per theme; add sun billboard? Skip. Maybe gradient with fullscreen quad? Keep: solid sky color + fog blends horizon. Add distant "haze". OK.

Also Dust2 needs arches: fake arches with two pillar boxes + top lintel box. OK.

Now UI DOM structure:

- #menu (main): title, map cards (3 with preview canvas thumbnails? Render mini top-downs — nice: draw map footprint to small canvas), team select, team size, difficulty, start button, controls list.
- #hud: crosshair div, bottom bars, top bar, kill feed, radar canvas, money, bomb icon, messages center, hit marker, damage vignette, flash overlay, scope overlay, progress bar (plant/defuse), buy menu panel, scoreboard, pause menu, end screen, spectate banner, chat? No.

Buy menu: grid of categories; click to buy; show price, disable if not enough money/time.

Now let me enumerate weapon data:

```
W = {
 knife:{dmg 55? keep 40, range 2, rate 1.5/s},
 usp:{dmg 30? CS USP 35? use 34, hs 4x? pistol HS x4? CS pistol HS x4 (with armor...). Keep hs x4, rpm 400? USP 352? use 360, mag 12, reserve 24, reload 2.2, spread .012 base, price 200, reward 300, speed 1.0, auto false}
 glock:{dmg 28, rpm 400, mag 20, reserve 120? 40? CS 20/120. use 20/80, price 200}
 deagle:{dmg 53, rpm 267, mag 7, reserve 35, reload 2.2, spread .02, price 700, hs x4 → 212}
 mp5:{dmg 26, rpm 750? CS MP5 800? use 750, mag 30/120, reload 2.4, price 1500, reward 600, spread .014, auto}
 galil:{dmg 30, rpm 666, mag 35/90, price 1800, reward 300, auto}
 famas:{dmg 28, rpm 666, mag 25/90, price 2050, reward 300}
 ak47:{dmg 36, rpm 600, mag 30/90, reload 2.5, price 2700, spread .010, recoil high}
 m4:{dmg 33, rpm 666, mag 30/90, reload 2.6? 3.1? use 2.8, price 3100}
 ssg:{dmg 80? SSG 88? use 85, rpm 45, mag 10/90, reload 3.5, price 1700, reward 300, scope zoom 40}
 awp:{dmg 115, rpm 41, mag 10/30? 5/30? CS AWP mag 10? CS2 AWP mag 5? CS:GO AWP mag 10. Use 10/30, reload 3.6, price 4750, reward 100, speed 0.75, scope 25}
 nova:{dmg 22x8 pellets, rpm 68? pump 1 shot/s? use 60rpm? Nova 68rpm? Pump: 1 shot per 1.1s? use rpm 55, mag 8/32, reload shell-by-shell 0.45, price 1050, reward 900, spread .05}
}
```
Grenade prices as above.

Kill reward per weapon class.

Bot aim: aim at enemy chest + error; if difficulty ≥ 2 and first bullet and distance < 20 and random, aim at head.

Recoil pattern: array of [pitch, yaw] increments, e.g., AK: first 5 shots climb 0.9° each, then alternating sway. Implement generic: pitchKick = w.rk[0] * falloff(shots), yaw = sin(shots*1.7)*w.rk[1]*rand. Apply to camera pitch/yaw directly (view punch that recovers? CS applies directly). Apply directly to aim (no recovery) + small camera shake. Bots compensate perfectly (they have pattern knowledge) — just reduce their spread.

Spread → ray direction: dir + randomInCone(spread). spread = base*(1 + moveFactor*1.5 + (air?2) + sprayFactor + sprintPenalty). ADS reduces by 0.6x; sniper scoped: spread 0.0005 (when still).

Sniper: must scope for accuracy; unscoped spread .06.

Movement noise: footstep interval by speed; bots hear → investigate.

Now bot think loop cost: 9 bots × LOS raycasts (against ~400 AABBs) — fine.

LOS check: ray from eye to target eye; if any solid hit with t < dist → blocked; also smoke volumes block.

Grenade AI throw: compute ballistic? Just throw direction = normalize(target - eye) with arc: dir = normalize((target-eye) + up*dist*0.18), speed 13. Good enough.

Plant spots: site center ± random 2m.

Now write Dust2 geometry concretely. Let me define coordinates carefully (x east, z south; north = -z). Wall height 3.2 default, tunnels 2.4, outer walls 5.

Let me place (drawing mental grid, 1 unit = 1m):

Outer bounds: x -34..34, z -22..56.

**T Spawn**: floor area x -12..16, z 40..54. Surrounding walls:
- South wall z=54, x -12..16.
- West wall x=-12, z 40..54, gap for tunnel entry z 44..48 → tunnel corridor west.
- East wall x=16, z 40..54, gap z 44..49 → long doors corridor.
- North wall z=40: segments x -12..-2 (blocks), gap x -2..4 (top mid entry), segment x 4..16.
Spawns around (2, 48).

**Upper Mid / Top Mid**: corridor x -2..4, z 18..40, walls on both sides (x=-2 wall z 18..40; x=4 wall z 18..40) but east wall has gap z 20..24 → stairs up to catwalk (short). Actually classic: from T spawn, mid is open "suicide" then Xbox. Mid floor goes down? Keep flat. Xbox crate at (1, 16) size 1.8, sits in mid just before doors.
Mid continues z 4..18, narrows? Mid walls continue to z=4 where mid doors sit (z=4, doors spanning x -2..4, two panels each 2.7 wide, with 0.4 gap in middle? Classic mid doors meet in middle; small gap? Make gap 0.3).
CT side of mid: z -6..4, x -2..4, opens to CT spawn at z=-6 (no wall) — CT can peek mid through doors gap.
Also T-side mid: side alcove? Skip.

**Catwalk / Short**: from mid east gap (x 4, z 20..24) stairs up +1.5 to catwalk platform: x 4..12, z 18..24 (raised floor 1.5, with side walls), then north: catwalk continues x 8..12, z 6..18? Then A short stairs down to A site? Classic short: catwalk leads to A short stairs up to A site (A site higher than catwalk? A site ~ same/above). Simplify: catwalk height 1.5; A site height 1.2; short stairs connect at x 12..16, z 8..12, descending to A site. Hmm heights: let me set A site platform +1.2, catwalk +1.5, short stairs descending from catwalk to site (2 steps). OK.

**Long A**: from T spawn east gap (x16, z44..49) → short corridor x 16..20 z 44..49 (long doors corridor), long doors at x=20 (doors slide into walls north/south of opening? Doors slide along z). Then Long A corridor: x 18..24? Let me set long corridor x 18..23, z from 44 down to 10: walls at x=18 (west) z 10..44 and x=23 (east) z 10..44. This is long sniper straight (~34m). At north end z=10: corridor opens west to "long corner": area x 12..23, z 6..10; then A long stairs up to A site: stairs at x 12..18, z 2..6 rising to +1.2; A pit: sunken area at east side of stair base? Classic pit is at long entrance to site, sunken corner next to stairs: place pit as 1m deep hole at x 20..23, z 2..6? Actually pit is adjacent to A site at long entrance. Place: A site platform z -4..8, x 8..20 (+1.2). Long stairs x 12..18 climb z 6..2? Let me restructure:

A site platform: x 6..22, z -8..4, height 1.2.
Long approach: corridor ends at z=10, area x 12..23 z 6..10 (ground 0) "long corner/goose area"; stairs x 12..20, z 4..6 rise 0→1.2 (4 steps of 0.3). Pit: sunken 0.9 deep at x 20..23, z 4..8 (next to east of stairs) — player can hide; from long corridor you can jump into pit? Add step. OK.
A site boxes: "default" crate at (12,-2) 1.6³? plus small box. Also "goose" wall? Add low wall at (18,0).
A ramp (CT side): from CT spawn (x 6..10, z -10) sloping up to site edge z=-8? Ramp x 6..10, z -12..-8 rising 0→1.2. CT spawn ground 0.

**CT Spawn**: x -8..10, z -20..-10. Walls: north z=-20; west x=-8, gap z -18..-14 → B doors corridor; east x=10, gap for A ramp z -14..-10? Ramp is x 6..10 z -12..-8 — adjust: east wall x=10 z -20..-12, then opens to ramp area. South side: z=-10 wall segment x -8..-2? and x 4..10? Middle gap x -2..4 → CT mid (z -6..4 area) — wait, CT mid z -6..4 connects CT spawn (z -10) — set CT mid corridor x -2..4, z -10..4, walls x=-2 (z -10..4) and x=4 (z -10..4). So CT spawn south wall z=-10 has gap x -2..4 (to mid) and east opening for ramp x 6..10? Ramp area x 6..10, z -12..-8: CT spawn east wall x=10 z -20..-12, then ramp zone, then site. Also B corridor: from CT spawn west gap x=-8 z -18..-14 → corridor x -14..-8, z -18..-14 heading west to B doors at x=-14 (doors slide along z), then B side corridor x -20..-14 z -18..-14? then opens to B site north end.

**B Site**: x -26..-12, z -12..4, ground 0 with raised platform? Classic B has platform with boxes and tunnels exit at south, doors at northeast, "window" overlook from CT? Keep: floor 0; platform +0.6 at x -22..-14, z -10..-4 with boxes on top? Simplify: crates: B box at (-18,-2) 1.8³, another at (-22,2) 1.2. Walls: west x=-26 z -12..6; north z=-12 x -26..-16 (with gap x -16..-14? no—doors corridor enters from north at z=-14? Let me set B doors corridor at z -18..-14, x -20..-8; B site north wall z=-12, x -26..-20 and x -16..-12? Gap x -20..-16 leads from corridor into site? Corridor z -18..-14; site starts z -12; area z -14..-12 x -20..-16 is opening. OK.
East wall of B site x=-12, z -12..0, then tunnel entry: south end: tunnel exit at z 4..8, x -22..-18 (from tunnels north). B site south wall z=4: x -26..-22 and x -18..-12.

**B Tunnels**: from T spawn west gap x=-12 z 44..48 → corridor x -20..-12, z 44..48 (upper tunnels), ceiling 2.4; then turn north: corridor x -22..-18, z 8..44? Long straight tunnel (~35m) with low ceiling — B tunnel sniper line; slight bend at z 26 (offset x by 2 with gap) to break sightline? Classic tunnels have curve ("stairs" area). Add bend: south segment x -22..-18 z 26..44; north segment x -20..-16 z 8..26; connecting area z 26..28 x -22..-16 (turn). Then exit to B site at z 8 → opening into site z 4..8? Set tunnel exit z 8, site south wall gap x -22..-18 (z=4). Hmm tunnel z 8..26 x -20..-16 exits at z=8 to area x -22..-18 z 4..8 (B site south vestibule) then into site. OK.

Ceilings: tunnels ceiling slab y 2.4..2.6 (blocks view from above, adds enclosure). Also mid doors area open sky. Long A open. Add some lintels/arches: arch over T spawn→mid entry? Decorative arches at long corner and site entries.

Heights: outer walls 4, inner 3.2, tunnel walls 2.6 + ceiling slab.

Also "sniper lines" ✓ (long, mid, tunnels), "double doors" = long doors? and mid doors ✓, "Xbox" ✓, "A pit" ✓, "A slope" = A ramp ✓, "B洞" tunnels ✓, "B门" B doors ✓, "B platform" — add raised platform at B ✓ (make platform +0.6 area x -22..-14, z -8..-2 with step 0.3? Make it two-step or ramp; use two 0.3 steps). "警家" CT spawn ✓. "长门" — long doors ✓ (双门 = mid doors? both exist).

Waypoints (Dust2):
0 TSpawn(2,48)
1 TLong entry (18,46)
2 LongDoors T-side (20.5? doors at x=20 → wp (20,46.5)? Let me place: corridor x16..20 z44..49; doors at x=20. wp: LDT (18,46.5)? Let me list:
- 0 (2,48) TSpawn
- 1 (14,46) TSpawn east, near long entry
- 2 (18,46) long corridor before doors
- 3 (20.5,46) just past doors? Doors at x=20; wp after (21.5,46)
- 4 (20.5,30) mid long
- 5 (20.5,12) long end north
- 6 (16,8) long corner
- 7 (16,5) base of stairs
- 8 (14,0) A site south (on platform)
- 9 (10,-4) A site default west
- 10 (18,-6) A ramp top? Ramp at x6..10 z-12..-8 → top (8,-8.5)
- 11 (8,-14) CT spawn east
- 12 (0,-15) CT spawn center
- 13 (-6,-16) CT spawn west (B corridor entry)
- 14 (-11,-16) B doors corridor
- 15 (-17,-16) past B doors
- 16 (-18,-12) B site north
- 17 (-18,-4) B site platform
- 18 (-20,1) B site south / tunnel mouth
- 19 (-19,14) tunnel north segment
- 20 (-20,27) tunnel bend
- 21 (-20,38) tunnel south segment
- 22 (-15,46) T spawn west / tunnel entry
- 23 (1,38) top mid entry
- 24 (1,26) mid south (Xbox south)
- 25 (1,20) Xbox
- 26 (1,12) mid north of Xbox
- 27 (1,6) mid doors T-side
- 28 (1,0) mid doors CT-side
- 29 (6,22) short stairs bottom (gap in mid east wall z20..24 → wp (5,22))
- 30 (8,21) catwalk south
- 31 (10,14) catwalk middle
- 32 (13,10) short stairs top? Short enters A site at x12..16 z8..12: wp (14,10) height 1.2? Catwalk 1.5 → stairs descend; wp (14,9).
- Connect 32→8? (14,9) to (14,4)? A site south edge z=4? Site z -8..4; short entry at z 8..12 is south of site... hmm let me set site z -8..6 to meet short. Adjust A site platform z -8..6, x 6..22. Long stairs z 4..6 climb. Short entry x 12..16 z 6..10 descending to site? Overlap. Simplify geometry: A site platform x 6..22, z -8..6. South edge z=6: long stairs x 12..20 z 6..8 (rise 0→1.2); short stairs x 6..10? No short enters from west? Classic short enters site from mid side (southwest of site?). Real Dust2: short stairs ascend to A site from catwalk on the south/west side of site, near ramp? Short stairs are adjacent to ramp area. Let me place short entry at site south edge x 6..10, z 6..9, descending from catwalk height 1.5 → 1.2 (small step). Catwalk: x 4..8? Let me redo: mid east wall gap z 20..24 at x=4 → stairs up x 4..8, z 20..24 (0→1.5); catwalk corridor x 6..10? Hmm catwalk runs north along mid's east side: x 6..10, z 8..24, raised 1.5, walls east x=10 (z 8..24), west edge overlooking mid? Classic catwalk overlooks mid with low wall? Actually catwalk is enclosed-ish. Keep walls both sides: west wall x=6 z 8..24 (separates from mid), east wall x=10 z 8..24. Then at north end z=8: opening x 6..10 z 6..8 descending to site (1.5→1.2, one step 0.3). Good: short stairs wp (8,7).

So waypoints: 29 (5,22) stairs bottom; 30 (8,22)?? Stairs x4..8 z20..24: bottom (6,23.5) top (6,20.5). Catwalk wp: (8,16), (8,9), short stairs (8,7) → A site west (8,2).

Links: chain 0-1-2-3-4-5-6-7-8; 8-9; 9-10 (ramp top); 10-11; 11-12; 12-13; 13-14-15-16; 16-17-18; 18-19-20-21-22-0; 0-23-24-25-26-27-28-12 (mid to CT); 25-29? Xbox(1,20) to short stairs bottom (5.5,22): link 24-29? 24 (1,26) no; 25 (1,20) link to 29 (5,22)? Path via gap z20..24 x4: wp 29 (5,22). Link 25-29, 29-30 (6,20.5 top) → 31 (8,16) → 32 (8,9) → 33 (8,7) → 9? (8,2) = wp 34 A site west; 34-8. Also 28 (1,0) mid CT-side → 12 CT spawn; and 28 → ? CT mid to short? None.
Also link 7-33? Long corner to short? No.
A ramp: 10 (8,-8.5) top of ramp; 11 (8,-13) CT. Ramp geometry x 6..10, z -12..-8.

B doors at x=-14, corridor z -18..-14: wp 14 (-11,-16), 15 (-17,-16).

Long doors at x=20 z 44..49: two panels: panel A z 44..46.4, panel B z 46.6..49, gap .2? Classic long doors: both panels open outward? Both slide open on push. OK.

Mid doors at z=4, x -2..4: two panels x -2..0.9, x 1.1..4, gap 0.2 (bullets pass).

Also need site volumes: A site box x6..22 z-8..6 (y any), B site x-26..-12 z-12..4.

Buy zones: T spawn rectangle, CT spawn rectangle.

Defensive spots CT: A: ramp top (10) facing site? Positions: [(9,-9, facing site), (14? )...] Provide list of {p:[x,z], site:'A'|'B'|'MID'} with facing direction computed toward likely enemy direction. Bots pick random from site's list.

T post-plant guard spots similar.

OK. Maps 2/3 with similar but lighter detail (fewer waypoints ~16-20).

**Map 2 "Ruin"** layout (70x70, x -30..30, z -28..34):
- T spawn: x -8..8, z 26..33. Exits north: three: west alley x -8 gap → west corridor; mid x -2..2; east alley.
- Mid courtyard: x -8..8, z 4..24, pillars (2x2 boxes) at (-4,18),(4,18),(-4,10),(4,10), fountain at (0,16)? Mid opens north to CT via hall x -3..3 z -2..4? and side halls to A/B.
- A (east): hall x 8..12, z 8..14 → A site x 14..28, z -6..10, raised 1.2, stairs at entry; cover boxes; second entry: east alley from T: corridor x 12..16? Let me do: east alley x 10..14, z 14..26, from T spawn east side, curves north to A site southeast entry. CT connector: x 14..18? CT spawn north: x -10..10, z -24..-16; CT to A: corridor x 10..14, z -16..-6 → A site north entry. CT to B mirrored west.
- B (west): sunken -1.2: site x -28..-14, z -6..10, with stairs down from west hall x -12..-8? and from mid west hall x -12..-8, z 4..10? Plus west alley from T: x -14..-10, z 14..26.
- CT mid hall: x -3..3, z -16..-2 with door? Add "temple gate" double doors at z=-2.
Waypoints ~18.

Theme: stone/moss, green fog light.

**Map 3 "Depot"** (night, 80x80):
- Containers as long boxes (2.4 wide, 2.6 tall, 6-12 long) forming lanes.
- T spawn south x -6..6 z 30..37.
- Mid: rail line east-west at z 12..16, with two train cars (long boxes 3 tall) with gaps to pass through; connects east and west flanks; mid approach from T: open yard with container cover.
- A site east: loading dock x 14..28, z -8..6, raised 0.9, crane container stacks; entries: from mid east along rail, from T east lane (container corridor x 10..14, z 16..30), from CT east corridor.
- B site west: warehouse x -28..-14, z -10..4, with ceiling 4.5, entries: west rail end, T west lane, CT west corridor; inside crate rows (CQC).
- CT spawn north x -8..8, z -30..-22, connectors east/west.
- Long sniper lane: T spawn → A dock straight 40m along x 12 lane? Provide one long straight.
Waypoints ~18.

OK. Now also plan bot strategic data per map: `strat = { tRoutes: [ {site:'A', path:[wp indices...]}, ...], ctSpots: {A:[...], B:[...], M:[...]}, plantSpots:{A:[x,z],...}, smokeSpots:[{at:wp, from:[x,z], to:[x,z]}], flashSpots:[...] }`. Bots pick route by site assignment.

Simplify: bot T: assign site with random weights; path from spawn wp to a random site wp (pathfinding will pick shortest; to force variety, add intermediate via wp: e.g., via mid or via long). Implement route as list of via waypoints; bot follows sequentially with A* segments.

CT: assign spot wp from ctSpots; go there and hold (scan with small rotation). On bomb plant/drop → path to bomb.

Now, code size management: I'll write it all now. Let me be disciplined and write compact but complete code. Use short helper names. Avoid over-commenting.

Also note: response must include run instructions and test notes, in Chinese. Deliverable: single HTML file content. Given the size, I'll output the full file. Let me write.

Key risk areas to get right:
- Pointer lock and canvas click.
- WebGL init and matrix math.
- Collision stability.
- Round state machine.

Let me write math first:

```js
function mat4(){return new Float32Array(16)}
identity, multiply, perspective, lookAt via yaw/pitch + position (build view from camera: translate then rotate).
```
View matrix: V = Rx(-pitch)*Ry(-yaw)*T(-pos). Compose manually.

For model: translation + yaw rotation + scale: build quickly.

Project point for HUD? Not needed except for damage direction? Skip.

Renderer details:

```
vs: attribute vec3 p,n; attribute vec2 t; attribute vec3 c;
uniform mat4 uMVP, uM; uniform float uMode; uniform vec2 uOff,uSc;
varying vec2 vT; varying vec3 vC; varying vec3 vN; varying float vD;
void main(){ vec4 wp = uM*vec4(p,1.0); gl_Position = uMVP*vec4(p,1.0)? 
```
Note: static buffer uses uM=identity and baked atlas UVs (uMode=1). Dynamic uses uM=model, uMode=0 with uv=uOff+t*uSc.

Fragment:
```
vec4 tx = texture2D(uTex, vT);
float sun = max(dot(normalize(vN), uSun),0.0);
float li = uAmb + sun*uDif; // per-face shading
vec3 col = tx.rgb * vC * li;
fog: float f = clamp((vD-uFogN)/(uFogF-uFogN),0.,1.); col=mix(col,uFogC,f);
gl_FragColor=vec4(col, tx.a*uAlpha);
```
vD = distance in view space: compute length((uMV)*p)? Simpler: vD = length(wp.xyz - uCam).

Blending: default opaque; particles/smoke/muzzle/flash drawn with blend and depthMask false.

Buffers: static VBO per map. Dynamic: build array per frame for entities → one VBO dynamic (bufferData STREAM_DRAW) with same layout, drawn with uMode=1 (bake model transforms on CPU and atlas UVs on CPU). This avoids per-draw uniforms entirely! CPU transform per box: 24 vertices × ~300 boxes = 7200 vertex transforms/frame — fine. So single dynamic draw call (+ separate additive pass for effects, + alpha pass for smoke).

So geometry push helper:
```js
function pushBox(arr, cx,cy,cz, sx,sy,sz, yaw, mat, tint, opts)
```
Compute 8 corners rotated by yaw, push 6 faces with normals rotated, UVs from material tile with world-scale tiling (use sx etc. per face).

Material tile info: MATS[i] = {u0,v0,scale?}: atlas 4x4 tiles of 256 → atlas 1024. Tile uv size .25. Add small inset to prevent bleeding (0.002).

Texture generation: canvas 1024, draw each tile with noise function.

Now entities:

Player object: {team, isBot, pos:[x,y,z] (feet), vel, yaw, pitch, hp, armor, helmet, money, weapons:[{id, mag, reserve}], slot, grenades:{he:n,fl:n,sm:n,mo:n}, alive, state..., bombCarrier, kit, name, kills, deaths, reaction etc.}

Local player index 0.

Camera eye = pos + [0, 1.62 (crouch 1.1), 0].

Update order per frame: input → local player physics → bot think/move → bullets handled instantly on fire → grenades update → bomb/round logic → effects → render → UI.

Fire logic (hitscan):
```
function fire(p, w): if mag<=0 → click sound; else mag--, compute dir with spread, raycast world + players + doors + grenades? (can shoot grenades? Skip), apply damage, effects (tracer from muzzle, shell, muzzle flash, sound), recoil.
```
Muzzle position: eye + forward*0.35 + right*0.12 + up*-0.08.

Shotgun: 8 pellets each raycast.

Damage direction for hit feedback? Skip.

Kill: credit, money reward, kill feed, check round end (all dead).

Bomb carrier: if T dies, bomb drops as entity at position (world bomb). T can pick up (E). Plant requires carrying.

Bot plant: reach site, stand, channel 3s (progress), if shot → interrupt? CS plant interrupts on damage? Plant cancels on taking damage? In CS plant cancels on being shot? Yes, damage interrupts plant. Defuse same. Implement channel cancel on damage.

Defuse: CT near planted bomb, hold E; bots defuse if no visible enemy and near bomb (hard bots with kit priority).

Grenade entity: {type, pos, vel, fuse, owner, team}. Update: gravity 15, integrate, collide with world (AABB): on hit reflect with component *0.35, friction. HE explode: radius dmg 350? CS HE max 98 dmg? Dmg = 90 * exp(-d/4.5)? With falloff, ignore walls? CS HE damage goes through walls partially — implement: LOS check: if no LOS, dmg *0.35. Radius 6m.

Flash: on explode: for each alive player: vector to, dist < 12? and LOS → blindT = clamp((1 - d/14),0,1)* (facing factor) *3.2s. Local: white overlay. Bots: blindT disables enemy engage and moves slowly forward.

Smoke: spawns smoke cloud entity at pos {t:18s}. Render 30 puffs. LOS block: segment-point distance < 2.2.

Molotov: on impact (first collision or fuse) → if in air explodes? CS molotov breaks on impact → fire patch radius 3.5, 12s, 40 dps inside (check per entity). Fire rendering: fire puffs.

Bot grenade inventory and usage as planned.

Sound positioning: function s3(name, pos, opts) computes distance attenuation and pan.

Now UI code: DOM updates each frame (cheap enough) or on change; fine.

Radar: pre-render map walls once to offscreen per map (draw top-down: iterate solids, draw rects scaled). Each frame: draw base + entities.

Buy menu construction from WEAPONS with team filters (AK only T, M4 only CT, etc.). Also grenades for both.

Scoreboard: table rows.

Spectate: on local death, camera follows living teammate (cycle with Fire1 or E?). Use click. Show banner "Observing X — click to switch".

Match flow states: MENU → GAME (phase: 'freeze', 'live', 'bombplanted' (same as live), 'roundend') → match end overlay.

Restart: reinit match with same settings.

Pause: only local can; bots pause too (skip update while paused).

Also handle window resize.

Now — "sprint" double-tap: track lastTap time per key.

Weapon pickup entity: {id, pos, owner?}. Render as small gun box on ground. E to pick up: swap current slot weapon? CS: picking up a weapon replaces same slot weapon. Implement: replace slot weapon; drop current on ground.

Dropped on death: player drops carried primary + secondary? Just drop primary and secondary as pickups. Nice for "drop gun". Also G to manually drop current weapon.

Bot economy and buying handled.

Now, team swap at halftime: player team changes — announce "Halftime — sides swapped". Player entity remains controlled but team flipped; bots regenerated per side. Simplest: rebuild all players at halftime with same names/skills, swapping teams, reset money to 800? CS halftime money reset? Money resets at halftime (yes, to 800). Weapons reset.

Match config: teamSize (1..5), difficulty (0..3), map. Player team choice.

Names: pool of bot names.

OK, also need "hit detection, headshot" — done; "observation and respawn" — respawn next round; observation during death — done.

Environment sound: low wind loop; per-map ambience (Dust2 wind, Ruin birds? Just vary filter), Depot hum.

Round start sound: "Go go go" beep? Use stinger.

Also add simple "voice-over" announcements via beeps? Skip; use text + stinger.

Now, let me write it all. I'll structure the code carefully. I'll write the final code in one shot, being careful with syntax. It's long; keep functions tight.

Also note: must not use external network. All procedural. Good.

Let me define constants and helpers:

```js
const clamp=(v,a,b)=>v<a?a:v>b?b:v;
const rnd=(a=1,b)=>b===undefined?Math.random()*a:a+Math.random()*(b-a);
const V={ add:(a,b)=>[a[0]+b[0],a[1]+b[1],a[2]+b[2]], sub:..., mul:(a,s)=>..., dot, len, norm, cross, lerp }
```

Matrices: write minimal:
```js
function persp(fov,asp,n,f){...}
function viewMat(pos,yaw,pitch){ // returns mat4
 // camera transform: T(pos)*Ry(yaw)*Rx(pitch) inverse
}
```
Compute view directly:
```
const cy=cos(yaw), sy=sin(yaw), cp=cos(pitch), sp=sin(pitch);
right = [cy,0,-sy]? 
```
Define yaw such that forward = [ -sin(yaw)? ... ]. Let me define: yaw=0 → look -z (north). forward = [sin(yaw)*? ...]. Use: forward = [ -sin(yaw)*cp? ... ]. Standard: 
fx = Math.sin(yaw)*Math.cos(pitch)? Choice: mouse right → yaw increases → look east (+x). forward = [sin(yaw)*cos(pitch), sin(pitch)? sign...]. pitch positive = look up.
forward = [ cp*sin(yaw), sp, -cp*cos(yaw) ]? yaw=0 → (0,0,-1) north. yaw=+90° → (1,0,0) east. Good, mouse right increases yaw (movementX>0 → yaw += dx*sens).
right = [cos(yaw), 0, sin(yaw)] (yaw 0 → (1,0,0) east ✓).
up = cross(right, forward)? up = [ -sp*sin(yaw)? compute via cross(right,forward) ].

View matrix rows: [right; up; -forward] with translation -dot. Construct:
```
m = [ r0,u0,-f0,0, r1,u1,-f1,0, r2,u2,-f2,0, -dot(r,p),-dot(u,p),dot(f,p),1 ] (column-major for WebGL: careful)
```
WebGL mat4 is column-major: m[0..3] = first column. View matrix V where v_view = V * w. V = R^T * T(-p). Columns: write in row-major conceptual then transpose when filling. Fill:
```
vm[0]=r[0]; vm[4]=r[1]; vm[8]=r[2];  vm[12]=-dot(r,p);
vm[1]=u[0]; vm[5]=u[1]; vm[9]=u[2];  vm[13]=-dot(u,p);
vm[2]=-f[0];vm[6]=-f[1];vm[10]=-f[2];vm[14]=dot(f,p);
vm[3]=0;vm[7]=0;vm[11]=0;vm[15]=1;
```
Yes.

Perspective standard.

MVP = P*V (no model since CPU-baked). Need multiply function for MVP: write mat4mul(a,b).

Actually since we bake transforms on CPU, shader only needs uVP.

Ray-AABB:
```js
function rayBox(o,d,b){ // b={x1,y1,z1,x2,y2,z2}
 slab method returning t or Infinity
}
```

Ray vs world: iterate solids (array of Float32? use objects {a:[x1,y1,z1], b:[x2,y2,z2], mat}). Count maybe ~600 per map; per-shot loop fine; bot LOS every 100ms fine.

Spatial optimization: skip; fine.

Segment-segment distance for body capsule:
```js
function segSeg(p1,p2,q1,q2){...standard closest points...return dist}
```

Ground height function per map: solids with 'floor' flag? Compute: for each solid whose horizontal bounds contain (x,z) (expanded by 0), candidate top = b[1]; take max top with top <= feetY + 0.45. Also ramps: list {x1,z1,x2,z2,h0,h1,axis} → height interpolation. Player.groundY = max(candidates). Standing on solid: also require solid top >= feet-? Standard.

Horizontal collision: for each solid: if solidTop > feet+0.45 && solidBottom < feet+1.7 (head clearance check: bottom < eyeTop): resolve circle vs AABB in XZ: find nearest point, if dist < r → push out along axis moved. Do per axis after moving that axis.

Doors: when closed, included in solids list; when open, removed. Manage: doors have solid object; toggle 'open' flag; collision loop skips open doors.

Ramps drawn as wedges: custom prism push (5 faces). pushPrism helper.

Stairs: sequence of step boxes.

Now character rendering:

```js
function drawCharacter(p, t){
 if !p.alive: draw lying corpse (rotated boxes) — simpler: draw flat boxes at ground.
 yaw = p.yaw; walk = p.walkPhase;
 boxes: 
  legs: 2 boxes 0.16w 0.75h, offset ±0.12 x local, swing z offset by sin(walk)*0.25 (rotate? just offset z and y slightly)
  torso: 0.42x0.55x0.24 at y 1.05..1.5? 
  head: 0.22 cube at 1.62
  arms: forward holding gun: box from torso forward
  gun: dark box forward 0.5 long
 team tint: T skin tan + clothes brown/grey; CT navy.
}
```
Push all to dynamic array with yaw rotation.

View model (local): render gun boxes attached to camera: compute muzzle transform: pos = eye + right*0.18 + up*-0.15 + fwd*0.45; build boxes along fwd. Apply bob: offset by sin(walkPhase)*amt; recoil: offset backward + pitch. Draw after world with depth? Draw with normal depth (may clip walls — acceptable, or draw with depth test on but offset slightly; fine).

Muzzle flash: additive quad at muzzle when fireT>0.

Now particle system: array of {p, v, t, life, size, col, grav, add?}. Render as camera-facing quads in dynamic alpha pass. Tracers: {a,b,t} rendered as thin quads? Use GL_LINES with separate small shader? Simpler: render as stretched box (thin quad facing camera). Implement line as quad: dir = b-a, side = normalize(cross(dir, camDir))*0.015.

Shells: small boxes with gravity and bounce, as particles with grav and box render? Render as small yellow quads. OK as particles with grav, size .04, col brass.

Bullet holes: {p, n, t:10s} rendered as small dark quads oriented to normal + slight offset.

Smoke cloud: entity list; each has puffs [{off, r, seed}]; render as big soft quads with grey alpha; use soft-edged radial gradient texture tile (generate 'puff' tile in atlas with alpha). Atlas tiles need alpha support — use RGBA canvas. Yes.

Fire: similar with fire gradient tile, additive.

Muzzle flash tile: star gradient.

So atlas tiles: 0 sand wall, 1 dirt floor, 2 stone, 3 crate, 4 metal, 5 door, 6 concrete, 7 brick, 8 container (plain, tinted), 9 marble, 10 dark metal/rails, 11 grass?, 12 puff (soft white), 13 fire, 14 flash star, 15 sky? OK 16 tiles 4x4.

Particles use tiles 12/13/14 with additive/alpha.

Now bot AI implementation plan (concrete):

```js
function botThink(b, dt){
 b.thinkT -= dt; if(b.thinkT>0 && b.state not changed) ...
 each think (0.12s):
  sense enemies: for each alive enemy: dist<65 && LOS(eye,eye) && !smokeBlocked → visible; track b.visT (time visible), lastSeen pos.
  if b.blindT>0: wander slowly, no engage.
  decide:
   if visible enemy && b.weapon ready: ENGAGE: aim point = enemy chest/head; b.aim += turn toward with turnRate by difficulty; if angleDiff < aimErr && reaction elapsed → burst fire.
   movement during engage: strafe randomly, maybe crouch.
   else objective:
    T: if bombCarrier: go to assigned site plant spot; if at spot & safe-ish (no visible enemy? still plant after 1.5s) → plant channel.
       else if bomb dropped: nearest T goes pick up.
       else go to route target (site entry) then take site flank position / guard.
    CT: if bomb planted: nearest CTs converge to defuse: go to bomb; if at bomb & no visible enemy → defuse channel; others guard nearby.
       else if bomb dropped (carrier died, bomb on ground in T area?) → hold/retake position.
       else: go to assigned defensive spot; hold, scan (yaw oscillation).
  grenades: on engage start with dist>6: probability throw HE at enemy pos (if have). At flashSpots/smokeSpots when passing: use.
 path follow: b.path list of [x,z]; move to next; if blocked (stuckT) recompute.
}
```

Bot fire: call same fireWeapon with bot's spread (difficulty-based added spread, plus move penalty). Bot reaction: on first becoming visible, set b.reactT = reaction; only fire after.

Bot aim smoothing: b.yaw/pitch lerp toward target with speed (deg/s by difficulty) + noise sigma.

Turn rate: easy 180°/s, expert 600.

Bots hear gunshots: on fire, for each bot within 28m (or 12 if silenced?) → if enemy team, set investigate pos. Footsteps: skip hearing footsteps for scope? Add: within 8m loud (sprint) → investigate. OK minor.

Now round/match manager:

```js
G = { state:'freeze'|'live'|'end', tRound, tFreeze, bomb:{state:'carried'|'dropped'|'planted'|'defused'|'exploded', pos, t, carrier, defuseProg, plantProg}, score:[ct,t], round, half, ... }
```
Round end reasons: 'T eliminated', 'CT eliminated', 'Bomb exploded', 'Bomb defused', 'Time expired'. Award money, message, state 'end' 5s → next round or halftime or match end.

Freeze: 6s (buy allowed), players locked? CS freeze time: can't move but can buy. Implement: during freeze, block movement input but allow view + buy. Bots stand. Then live.

Buy time extends 12s into live near spawn.

Bomb plant: only in site volume & carrier alive & grounded; channel 3s with progress UI and beeps; on complete: bomb.planted at pos.

Defuse: CT within 1.8m of planted bomb, channel (5s/10s? I said 10 without kit? CS: 5 with kit, 10 without). Yes.

Bomb explode: 40s, damage 500 radius? Kill all in radius 12 with falloff (big). Then round end.

Timer UI.

Now economy constants:

```
prices: as above.
rewards: kill per weapon class; win 3250; loss 1400 + lossStreak*500 (cap 3400); plant: planter +300, all T +800 if lose with bomb planted? CS: losing team still gets loss bonus + plant bonus $800? T gets extra 800 for plant even on loss? Actually plant bonus: each T gets +800 on plant (even on loss)? In CS: plant gives $300 to planter, and if T lose after plant, they get loss bonus + $800? Simplify: on plant: planter +300 immediately; if T lose round after plant: extra +800 each. CT defuse: +300 to defuser? CS defuse reward $300. CT win by time: +3250 standard.
max money 16000.
```

Bot buy logic:
```
function botBuy(b){
 need armor+helmet (1000) if money>=1000+weaponCost...
 priority: if money >= 4750+1000 && bot.sniperOk → awp; else if money>=2700/3100+1000 → team rifle; else if >=2050 → famas/galil; else if >=1500 → mp5; else if >=700 → deagle; else default pistol.
 buy grenades: he/smoke/flash by difficulty and money.
 kit for CT if >=400 after weapon (hard+).
}
```

Player buy: menu.

Now UI layout/CSS: military style: dark panels, stencil-ish font (use system: 'Arial Black'/impact? use font-family: "Segoe UI", with letter-spacing; or monospace mix). Keep clean HUD: white text with black outline (text-shadow).

Crosshair: 4 divs or canvas? Use single div with CSS box-shadow trick; simpler: canvas overlay? Draw crosshair with 4 absolutely positioned divs positioned by JS (gap dynamic). OK.

Kill feed: stacked divs top-right, fade.

Center messages: big text fade.

Buy menu: left panel with categories; item rows with name/price; hotkeys 1-9? Click only + number categories. Provide close (B/Esc).

Scoreboard: Tab hold.

Radar top-left 170px, with border.

Also bottom-center: round score pills and timer, bomb icon.

Progress bar for plant/defuse bottom-center.

Spectate bar.

End screen: victory/defeat, stats table, buttons.

Pause menu overlay with sliders (sensitivity, volume) and buttons.

Main menu: title "BROWSER STRIKE // 战术突袭"? Give it original-ish name: "CounterZone"? User asked CS-style; name: "DUST PROTOCOL"? Let me title: 《STRIKE FORCE · 浏览器战术射击》? Keep: "COUNTER STRIKE WEB — 战术行动"? Use "TACTICAL STRIKE // 战术打击". Map cards with thumbnails rendered from geometry (draw walls to canvas). Team select buttons, team size slider, difficulty select, start.

Controls list in menu.

Now, code size management: I estimate ~2600-3200 lines of dense JS. That's a lot of tokens (~40k+). Risky but let me try to compress smartly while keeping completeness. Maybe aim for ~2000-2400 lines with terse style. Compress map data with terse helpers.

Let me write compact helpers for map building:

```js
function MB(){ // map builder
 const S=[]; // solids {a,b,m}
 const R=[]; // ramps
 const D=[]; // doors
 function wall(x1,z1,x2,z2,h=3.2,m=0,y=0){ // axis-aligned wall from (x1,z1) to (x2,z2) with thickness .5
   if(Math.abs(x2-x1)>Math.abs(z2-z1)) S.push({a:[Math.min(x1,x2),y,Math.min(z1,z2)-.25],b:[Math.max(x1,x2),y+h,Math.max(z1,z2)+.25],m});
   else S.push({a:[Math.min(x1,x2)-.25,y,Math.min(z1,z2)],b:[Math.max(x1,x2)+.25,y+h,Math.max(z1,z2)],m});
 }
 function box(x,z,w,d,h=1.2,m=3,y=0){S.push({a:[x-w/2,y,z-d/2],b:[x+w/2,y+h,z+d/2],m})}
 function floor? // ground is implicit plane y=0; raised floors: box with small h? Use slab: box(x,z,w,d,0.2,...) but then walls sit on top... Manage heights manually.
 function slab(x1,z1,x2,z2,y,m=1){S.push({a:[x1,0? ...
```
Hmm raised platforms: make slab from 0 to y (solid block) — fine (no interior access). Use `block(x1,z1,x2,z2,y0,y1,m)`.

Ramps: {x1,z1,x2,z2, axis:'x'|'z', h0,h1, m} → ground height linear; drawn as wedge.

Stairs: `stairs(x1,z1,x2,z2,h0,h1,n)` → n step blocks along axis.

Doors: `door(x,z,w,axis,m)` create door solid with open animation offset along wall direction.

Ceilings: block with y0=2.4,y1=3.0 (also solid for bullets — fine).

Map solids count: Dust2 maybe ~250 solids. Raycast 250 per shot OK.

Ground height: default 0; slabs/blocks provide tops; ramps provide function. Collect per map: `groundAt(map,x,z,feetY)`:
```
let g=0? Actually base ground 0 always. Then for each solid s: if x,z inside (s.a[0]..s.b[0], s.a[2]..s.b[2]): top=s.b[1]; if top<=feetY+0.45 && top>g → g=top.
For each ramp r: if inside: h=lerp; if h<=feetY+0.45 && h>g → g=h.
```
Note: standing on block of height 1.2 (crate): approach requires jump (1.2 > step). OK.

But iterating ~250 solids per frame per player (10 players) = 2500 checks/frame — fine.

Horizontal collision: for each solid: if s.b[1] > feet+0.45 && s.a[1] < feet+1.6: circle vs rect resolve on moved axis. Doors closed included.

Also keep player within outer bounds clamp.

Grenade collision: same solids; reflect: determine hit face by penetration depths.

Bullet: rayBox over solids + doors (closed) → nearest; also check characters.

Now LOS for bots: same raycast.

Smoke LOS block: for each smoke cloud: distance from segment to cloud center < 2.3 → blocked.

Fire damage: for each fire patch: entity within radius & roughly same y → dps.

Now, "open door on push": door.closed=true; when player cylinder intersects door AABB expanded by 0.1 → door.opening=true, animate t 0→1 over 0.8s, move offset by slide direction (store openVec). When t=1, collision off. Sound creak. Bots also open (same check). Mid doors: same. This gives functional double doors.

Also "long doors" visual: two big green metal panels.

Now Dust2 visual dressing: add decorative non-solid? All solids fine. Add: arches at T spawn exit (lintel over entry), wooden beams? Add crates clusters, barrels (as small boxes, mat metal), low walls ("goose" wall at A), window frames? Keep moderate.

Also add sky sun direction per theme; fog color sand haze for Dust2 (0.78,0.72,0.6); Ruin greenish; Depot dark blue.

Ground texture outside walkable area: big base slab under whole map at y -0.05..0 (mat floor) so no holes seen. Add per map.

Now waypoint nav: implement `findPath(map, a, b)` Dijkstra on adjacency; positions W[i]=[x,z]. Bot movement uses path points at feet; convert to ground y via groundAt when moving (bot y = ground). Bots don't jump. Ensure waypoints are on walkable ground (I'll place accordingly; heights: catwalk wps have ground 1.5 via slab solids below — groundAt handles).

Bot stuck detection: if moved < 0.1 in 0.6s while intending → recompute path with slight random alternative; also simple avoidance: if blocked by teammate, sidestep.

Now strat data per map:

```js
map.strat = {
 tSpawnWp:0, ctSpawnWp:12,
 sites:{A:{wps:[8,9,34], plant:[[12,-2],[16,0]] , guard:[[...]]}, B:{...}},
 tRoutes:[ {site:'A', via:[1,2,3,4,5,6]}, {site:'A', via:[23,24,25,29,30,31,32]}, {site:'MID'?? no mid site}, {site:'B', via:[22,21,20,19,18]} , {site:'A', via mid then short...} ],
 ctSpots:{A:[[10,...facing],[...]], B:[...], M:[...]},
 smoke:[{at:25, to:[1,3.5], type:'sm'}], // smoke mid doors from Xbox
 flash:[{at:6, to:[14,2]}...]
}
```
Bot T picks route weighted random; follows via list: path spawn→via1→...→site wp. On reaching final, if no bomb action, pick guard spot near site.

CT spots: {wp or pos, yaw}. Bot goes, holds, scans ±30°.

Retake: on plant, all CTs path to bomb pos; Ts near bomb guard (pick guard spot), others continue/come.

Bomb drop: nearest T path to it.

Defuse logic: on CT reaching bomb & no visible enemy (or hard CT sometimes defuses under fire? Require no visible enemy, or bot has kit and teammate engaging? Keep: no visible enemy within 25m LOS) → defuse channel; interrupted by damage or visible enemy → resume.

Bot plant channel same.

Channel: b.channel={type:'plant'|'defuse', t}. Progress while stationary. Use same plant/defuse function as player (shared G.bomb.plantProg? Separate progress per entity but plant is global: only one plants at a time; allow both to add? Keep single: if any planter channeling, progress advances by dt/3; if all stop, decays. Same for defuse.)

Nice simplification: G.bomb.plantProg and defuseProg global scalars advanced by any channeling entity, decay when none.

Now player plant: hold E in site with bomb → set planterActive.

Bomb UI: progress bar.

OK, now let me write HUD updates.

Also "hit indicator": on local player hit: red vignette flash + damage direction? Skip direction.

Hit marker: on local damage dealt: white X briefly; on kill: red.

Now also add simple footsteps of enemies audible (3D).

Ambient: per-map loop noise with filter.

Announcer text: center message.

I think the plan is solid. Time to write the code. I'll write it as one continuous HTML. I'll be careful with syntax. I'll structure the code carefully. Given the huge size, I'll allow terse but readable code.

Also don't forget: response language Chinese for surrounding text; code comments can be minimal English/Chinese.

Let me draft the full code section by section in my head, then output.

### HTML/CSS skeleton

```html
<!DOCTYPE html><html lang="zh"><head><meta charset="utf-8"><title>...</title><style>...</style></head>
<body>
<canvas id="cv"></canvas>
<div id="ui">
  <div id="flash"></div><div id="dmg"></div>
  <div id="scope" class="hid"></div>
  <div id="hud" class="hid">
    <canvas id="radar" width="180" height="180"></canvas>
    <div id="topbar"><span id="s_ct"></span><span id="timer"></span><span id="s_t"></span><div id="bombicon"></div></div>
    <div id="killfeed"></div>
    <div id="msg"></div>
    <div id="pbar" class="hid"><div id="pfill"></div><span id="ptext"></span></div>
    <div id="bl"><div id="hp"></div><div id="ap"></div><div id="money"></div></div>
    <div id="br"><div id="wname"></div><div id="ammo"></div></div>
    <div id="cross"><i></i><i></i><i></i><i></i><b></b></div>
    <div id="hitmark" class="hid">✕</div>
    <div id="spec" class="hid"></div>
    <div id="buyhint"></div>
  </div>
  <div id="buy" class="hid">...</div>
  <div id="score" class="hid"></div>
  <div id="menu">...</div>
  <div id="pause" class="hid">...</div>
  <div id="end" class="hid"></div>
</div>
<script> ... </script>
```

CSS: fullscreen canvas, HUD absolute, military font, panels with rgba(10,14,10,.85) and olive borders.

### JS sections

1. Utils/math
2. Audio
3. Textures/atlas + materials
4. GL setup + shader + draw arrays
5. Geometry push helpers
6. Map definitions (3)
7. Nav/path
8. Weapon data
9. Entity/player creation, buy, economy
10. Input
11. Combat (fire, damage, grenades, bomb)
12. Bot AI
13. Round/match flow
14. Effects (particles, tracers, decals, smoke, fire)
15. Render (static build, dynamic build, view model, sky)
16. UI (HUD update, radar, menus, buy, scoreboard)
17. Main loop, init

Let me now write the actual code. I'll do it in one shot, being careful. I'll number materials:

```
MATS = [
 {tile:0, rep:.14}, // sand plaster wall
 {tile:1, rep:.14}, // dirt/sand floor? use for ground blocks sides
 {tile:2}, stone
 {tile:3}, crate wood
 {tile:4}, metal panel
 {tile:5}, door green metal
 {tile:6}, concrete
 {tile:7}, brick red
 {tile:8}, container plain (tinted)
 {tile:9}, marble/pillar stone light
 {tile:10}, dark steel
 {tile:11}, grass/moss ground
 {tile:12}, puff (alpha)
 {tile:13}, fire (alpha)
 {tile:14}, flash star
 {tile:15}, wood plank dark (tunnel)
]
```
rep = UV repeats per meter.

Atlas gen: function genAtlas(): create canvas 1024; draw each tile 256px with helper noise.

Tile drawing helpers:
```
function tile(i, fn){ ctx.save(); ctx.translate((i%4)*256, (i>>2)*256); fn(); ctx.restore(); }
noise fill: base color + random speckles + streaks.
```
Write specific generators: sand wall: base #c8b088, speckles darker, horizontal streaks, some cracks lines; floor dirt: #b39b6f speckles; stone brick: draw brick pattern #9a938a with mortar; crate: planks + cross diagonal + border, color #8a6b3f; metal: #6e7477 with rivets and vertical seams; door: #4c5b4a with cross beam and warning stripe? CS doors are greenish; concrete #9aa0a3 with stains; brick red #8d5a44 pattern; container #556 with corrugation vertical lines; marble #cfd0c8 with veins; dark steel #3a3f44; grass #5d7a4a with speckles; planks dark #5a4632 horizontal.
Alpha tiles: puff radial gradient white→transparent; fire radial orange; star cross white.

UV tile rect: u0=(i%4)/4+pad, size .25-2pad.

pushBox UV: per face, uv = tileRect with (coord*rep) mod 1 → uv = u0 + frac*usize. Compute per vertex.

Simplify: per face pick 2 axes for (u,v) = world coords along face axes * rep; uv = u0 + (frac(c*rep))*usize.

Implement generic pushQuad(arr, p0,p1,p2,p3, n, mat, tint, repAxisVals...). Write pushBox with explicit 6 faces.

Box faces with yaw rotation: compute corners:
```
function rotY(v,yaw){...}
local corners (±sx/2,±sy/2,±sz/2) → rotate → add center.
```
Faces: +x,-x,+y,-y,+z,-z with proper winding CCW as seen from outside. Standard.

UV mapping per face: e.g., +y (top): u=x*rep, v=z*rep. Sides: u = horizontal along face, v = y*rep.

Wedge (ramp): vertices: 4 bottom corners, 2 top edge corners. Faces: 2 triangles slope, 2 side triangles, bottom optional (skip), back face rectangle. Implement pushRamp(r): axis 'z' means height varies along z from h0 at z1 to h1 at z2. Compute.

Now static build: for map: push base slab, all solids as boxes, ramps as wedges, decoration; create GL buffer. Also build radar image from solids (top-down: draw rects for solids with y1>0.3, color walls; floor light).

Doors drawn dynamically (animated) — exclude from static; drawn each frame via dynamic path.

Now entities dynamic render per frame:
- Characters (alive + corpses)
- Dropped weapons/bomb on ground
- Grenades (small sphere-ish box)
- Doors
- Planted bomb (blinking)
Push all to dyn array (opaque), then particles to alpha array, additive to add array.
Upload and draw 3 passes.

View model: push gun boxes with camera-based transform after world (with depth test on); but gun may clip walls; acceptable.

Actually view model drawn into same dyn array — fine.

Muzzle flash quad to additive array.

Sky: clear color = theme sky; plus draw distant sun quad additive? Add sun disk at sun direction for Dust2 — nice touch: big additive quad far away. Cheap. OK.

Now input handling:

Keys: KeyW/A/S/D, Space jump, Ctrl crouch (use ControlLeft; prevent default), Shift walk, R reload, B buy, Tab score (prevent default), E use, G drop, 1..4 slots, Q prev? skip, Esc handled via pointerlockchange → pause.
Mouse: movementX/Y → yaw/pitch (sens). mousedown 0 fire start (auto continues), 2 ADS toggle/hold. Wheel: cycle weapon slots / cycle grenades if slot 3? Wheel cycles through owned weapons list.

Double-tap W etc. for sprint: track per direction keydown time; if within 280ms → sprint=true while that key held.

Fire auto: while mouse held & weapon auto & cooldown OK → fire. Semi: only on mousedown edge.

Now player update:

```js
function updPlayer(p,dt){
 if(!p.alive) return;
 input: if p.isBot → botMove sets p.mf/ml (forward/strafe -1..1), wantJump etc.
 speeds: base 5.2 * wspeed(p) ; crouch *0.4? crouch 2.2; walk 2.5; sprint *1.28 (not for awp?) fine.
 accel model: CS has air control etc. — use simple: target vel = wish*speed; ground: lerp with accel 12; air: accel 2 with cap.
 apply friction on ground.
 jump if grounded & want: vy=8.1.
 gravity 20.
 integrate with collision as designed.
 footsteps timer when grounded & speed>1.5: interval = 2.2/speed? play step, loudness flag.
 weapon: handle fire input (local) or bot trigger; reload timer; switch timer; ADS lerp fov; recoil recovery; spray counter decay.
 channel plant/defuse (if local holding E or bot channeling).
}
```

Local fire decision in updPlayer when p===me && mouseDown.

Recoil application: on fire: p.punchP += pat; adjust camera pitch/yaw directly (for local: me.pitch += kick*(sens scale? raw rad)). For bots: apply to their aim? Bots compensate: skip.

Spray: p.spray = min(p.spray+1, 12); spread += p.spray*0.0035 (rifle). Decay when not firing 20/s.

Now damage and kill:

```js
function hurt(victim, dmg, attacker, weaponId, isHead){
 armor calc; victim.hp -= hpDmg; spawn blood particles; if victim===me damage flash; if hp<=0 → death: alive=false, drop weapons and bomb (bomb drops), kills/deaths, money reward to attacker, kill feed, check team wipe → round end.
 if victim channeling → cancel.
}
```

Round end check after each death and timeout.

Now bot buy at freeze start; player can buy during freeze & buy time.

Now let me write map code concretely. This is the biggest chunk. Let me write Dust2 builder with helpers and explicit calls. I'll finalize coordinates (iterating in my head for consistency):

Constants: wall h=3.4, thickness .5 (wall() adds ±.25).

Dust2 solids plan (x,z). Let me write list:

Base slab: block(-36,-24,36,58, -0.5,0, mat floorDirt) — top face at y=0 everywhere. Then raised floors: A site platform block(6,-8,22,6, 0,1.2, sand floor top? sides sand wall) — top face mat? Single mat per box; want top floor mat and sides wall mat — allow matTop override in pushBox? Add matTop param to block helper. OK: block(x1,z1,x2,z2,y0,y1,m,mTop).

Catwalk slab: block(6,8,10,24,0,1.5,m wall, top mat floor).
Short stairs: descending from catwalk north end to site: site top 1.2, catwalk 1.5: step at z 6..8: block(6,6,10,8,0,1.2)? That merges with site. Actually site z -8..6; add slab block(6,6,10,8,0,1.2) as extension so catwalk (1.5) steps down via 0.3 step at z6..8 to site level? Then catwalk z 8..24. Steps: block(6,7.2,10,8,0,1.2)? Simplify: catwalk slab z 8..24 h1.5; site extension slab block(6,6,10,8, 0,1.2); site z -8..6 h1.2. Step 1.5→1.2 at z=8 (0.3 down) walkable (step ≤0.45 ✓), then 1.2→1.2 flat.

Long stairs: rise 0→1.2 over z 8..6? Wait long approach is z≥6 area. Long corridor ends at z=10; "long corner" area x 12..23 z 6..10 ground 0; stairs x 12..20 z 6..8? Site south edge z=6 (site z -8..6). Stairs x 12..20, z 6..8? That overlaps corner z 6..10: place stairs z 6..8: 4 steps of width .5: heights .3,.6,.9,1.2 as z decreases (step closest to site highest). stairsN helper.

Pit: x 20..23, z 4..8: sunk: floor at -0.9: make hole: base slab top 0 everywhere; pit: overlay block with top -0.9? Can't subtract. Alternative: build pit as raised surroundings? Simpler: pit area floor = block with top -0.9 not possible with additive only. Alternative: raise surroundings: keep pit floor 0 and add rim walls making it enclosed pocket: "pit" as alcove: 3 walls of height 1.2 around area x20..23 z4..8 with opening toward stairs, plus step? Player can hide there; from long you can shoot over? Classic pit is sunken. With additive-only, fake sunken by raising platform around? Too complex. Compromise: pit = alcove with low rim wall (crouch to hide), named PIT. Acceptable.

Actually additive sunken is possible: make whole long corner area floor at +0.9 (platform), and pit stays at 0 → appears sunken relative. So: area x 12..24, z 4..10 block top 0.9? Then long corridor floor 0 → need stairs up to corner at corridor north end (classic long has stairs up to site anyway). Let me do: long corridor z 10..44 floor 0. Corner/platform area x 12..24 z 4..10: slab top 0.9 (3 steps of 0.3 at south edge z 8..10? Stairs along z 9..10?). Then stairs from this platform (0.9) to site (1.2): single 0.3 step at z=6? Then pit: area x 20..23, z 4..8 remains 0 (no slab) → sunken 0.9 below platform, accessible via gap (no slab there) and drop-in from platform edge (drop 0.9 OK) and jump out? Jump 0.9 < 1.5 ✓. Great: real sunken pit!
So: platform slab block(12,4,24,10,0,0.9) but exclude pit rect x20..23 z4..8 → split slab into pieces:
 s1: block(12,4,20,10,...) x12..20
 s2: block(23,4,24,10) thin east strip? Pit x20..23: strip x23..24 z4..10? and s3: block(20,8,23,10) z8..10 over pit area (north part), leaving hole x20..23 z4..8.
Pit walls: hole edges are slab sides (0.9 tall vertical) ✓. Entry: gap at south? Pit south z=4: beyond is? Site z -8..6 x6..22 — pit x20..23 z4..8 overlaps site x range to 22. Hmm site x 6..22, z -8..6. Pit z4..8 x20..23: overlap area x20..22 z4..6 is both site platform (1.2) and pit hole (0)?? Conflict. Move pit east/south: pit x 20.5..23.5, z 6..9.5, platform z 4..10 → hole in platform; site edge z=6 x≤22... overlap x20.5..22 z6..? site top 1.2 vs hole 0 — conflict again.
Simplify: shrink A site platform to x 6..20. Long stairs x 12..18 climb to site at z 6. Pit x 20..23.5 z 5..9 (east of site, next to stairs) hole in corner platform. From pit you can peek long and stairs.

Final A area:
- Site platform: block(6,-8,20,6, 0,1.2).
- Long corner platform: pieces around pit hole (x20..23.5, z5..9):
  p1 block(12,4,20,10, 0,0.9) [west of hole]
  p2 block(23.5,4,25,10, 0,0.9) [east strip] (x up to 25; long corridor east wall x=23? Conflict: long corridor x 18..23 walls. Hmm long corridor east wall x=23 z10..44. Corner area z 4..10 is north of corridor end. Extend east boundary wall to z 4..10 at x=25. OK: outer east wall x=25 z 4..44? Let me set long corridor x 18..24, walls x=18 and x=24, z 8..44. Corner z 4..8. Adjust: corridor z 8..44; corner platform z 4..8, x 12..25; pit hole x 20..24, z 4.5..7.5? Then stairs x 12..18 z 6..8 climb 0.9→1.2? Only 0.3 rise — 1 step. Long approach: corridor floor 0 → corner platform 0.9: need stairs at corridor north end z 8..10? Add: corridor z 10..44 floor 0; transition stairs x 18..24 z 8..10: 3 steps of 0.3 up to 0.9. Then corner platform 0.9 z 4..8. Then step up to site at z=6 x12..18 (0.9→1.2).
 Wait site z -8..6, so z=6 boundary; stairs x12..18 z 5..6? Place step block(12,5,18,6,0,1.2)? Site slab already covers z≤6 up to 1.2; corner platform 0.9 at z 6..8? Let me set corner platform z 6..8 (not 4..8): pieces: p1 block(12,6,20,8,0,0.9), p2 block(24,6,25,8,0,0.9), p3 block(20,7.2,24,8,0,0.9)? Pit hole x20..24 z6..7.2? Small. Let me make pit bigger: corner platform z 5..9; pit hole x 20..24, z 5.5..8.5. Pieces: west block(12,5,20,9), east block(24,5,25,9), north strip block(20,8.5,24,9), south strip block(20,5,24,5.5). All h 0.9. Stairs from corridor: corridor ends z=10? Corridor z 11..44; stairs x 18..24 z 9..11 (0→0.9, 3 steps). Site entry step x 12..18 z 5..6? Site slab z -8..6 covers to z=6; corner platform from z=5? Overlap z5..6 x12..20: site slab 1.2 vs platform 0.9 — groundAt takes max ≤ feet+0.45 → 1.2 (if player on top). Visual z-fight overlap? Slabs overlap in region x12..20 z5..6 with different heights: platform 0.9 inside site 1.2 — hidden inside, top face 1.2 visible ✓ no z-fight (different heights).
 Step up: from platform 0.9 (z>6? no platform is z≥5...). Hmm corner platform z 5..9 overlaps site z5..6. Player walks north on platform at 0.9, at z=6 hits site slab face 0.3 tall → auto step ✓.
 Long stairs for vehicles? Fine.
- Pit: hole in platform at x20..24 z5.5..8.5 floor 0. Drop-in from platform edge (0.9 fall), exit by jump (0.9). Also from long corridor? Corridor east of stairs? Pit is east of stairs; entry from corner platform only + line of sight from long corridor north end. Classic enough.
- Goose/low wall on site: low wall block(16,-2,17,2,0? on top of site: y0=1.2,y1=2.3) — wait block takes absolute y: block(16,-2,17,2,1.2,2.4,m). Site crate: box at (11,-1) size 1.7 top 1.2+1.7; plus smaller box (13.5,2) 1.1. Ramp? A ramp = CT ramp: from CT spawn ground 0 to site 1.2: ramp x 6..10, z -12..-8, h0=0 (z=-12) h1=1.2 (z=-8) axis z. ✓ (A 斜坡)
- "A 坑" also might be pit ✓.

Mid:
- Mid corridor walls: west wall x=-2 z 4..40; east wall x=4 z 4..18 (then z 18..20 solid, then gap z 20..24 for short stairs, then wall x=4 z 24..40? But T spawn north wall covers z=40). Wait east wall x=4 z 4..40 with gap z 20..24. West wall x=-2 z 4..40 no gap.
- Mid doors at z=4: opening x -2..4; door panels: panel1 x -2..0.85, panel2 x 1.15..4, at z 3.75..4.25 (thickness .5), mat door. Slide open along x into walls? Slide into wall recess: panel1 slides to x -4.6..-1.85 (into CT side wall?) — walls at x=-2 thickness .5 → x -2.25..-1.75; can't hide panel. Just slide panels along z (north) against CT mid west/east walls? Simplest: slide along x outward overlapping walls (looks like panel slides into wall edge) — visually acceptable: panel1 open offset -2.6 x, panel2 +2.6.
- Xbox: crate at (1,16)? Mid z 4..40; Xbox classic just north of T spawn mid entrance... Actually Xbox is in mid, about 2/3 toward CT? It's mid-T side? Xbox crate is in mid, closer to top mid? It's around mid middle, used as cover peeking mid doors. Place at (1,10) size 1.8 → blocks mid lane; gap around sides (mid width 6, crate 1.8 centered → 2.1 gap each side ✓).
- CT mid: z -10..4 x -2..4 walls same x=-2,4 extended z -10..4. Opens to CT spawn at z=-10 (gap in CT spawn south wall x -2..4).
- Top mid entrance from T spawn: gap x -2..4 at z=40 ✓. Add "suicide" open west of top mid? Skip.

Short/catwalk:
- Stairs up from mid gap: x 4..8? Wait mid east wall is at x=4; gap z 20..24; stairs within x 4..8, z 20..24 rising 0→1.5 west→east? Player goes east through gap then up: stairs along x: at x4 h0 → x8 h1.5. 5 steps.
- Catwalk corridor: x 6..10? If stairs top x8 (1.5), catwalk runs north: x 6..10, z 8..24? Stairs overlap x4..8 z20..24. Catwalk slab block(6,8,10,20,0,1.5) plus merge with stairs top area x6..8 z20..24 (covered by stairs steps top). Walls: catwalk west wall x=6 z 8..20 (between catwalk and mid below? mid is x≤4; area x4..6 z8..20 is... void? catwalk slab x6..10; gap x4..6 z8..20: hole to ground? Add slab block(4,8,6,20,0,1.5)? Then catwalk width 6 (x4..10) with west wall at x=4 continuing above? But mid east wall x=4 z 8..20 exists at ground level; catwalk slab above at 1.5 sits above that wall — wall h 3.4 > 1.5 so slab intersects wall — fine (embedded). So catwalk slab x 4..10, z 8..24 h1.5, but mid gap z20..24 x4..? Stairs occupy. OK: catwalk slab block(4,8,10,24,0,1.5) minus? Stairs x4..8 z20..24 rise 0→1.5: overlapping slab area x4..8 z20..24: slab top 1.5 = stairs top ✓ visually merged; but stairs only climb via steps on south side? Player enters gap at z 20..24 from mid (x≤4) walking east: at x 4..8, z 20..24: stairs along x: steps: x4..4.8 h .3, ... x7.2..8 h1.5. Then catwalk slab continues north z<20 at 1.5 ✓. But slab block(4,8,10,24) covers z20..24 too at full 1.5 → blocks stairs (wall of height 1.5 at x=4). Fix: slab z 8..20 only: block(4,8,10,20,0,1.5). Stairs x4..8 z20..24. East of stairs x8..10 z20..24: wall or slab: add block(8,20,10,24,0,1.5) as landing. ✓
- Catwalk walls: east wall x=10 z 8..24 h3.4 from ground (also blocks mid view). West wall x=4 z 8..20 upper part: since mid wall x=4 exists h3.4, catwalk west side already walled ✓. At z 20..24 west side is open (that's the entry gap) with stairs ✓. North end z=8: opening x 4..10 → descend to site extension: site extension slab block(6,6,10,8,0,1.2)? Site x6..20 z-8..6 h1.2. Extension x 4..10? Site starts x6. Add extension block(4,6,10,8,0,1.2)? Then at z=8 catwalk (1.5) → extension (1.2) step down 0.3 ✓, then at z=6 extension → site same 1.2 ✓. But extension x4..6 z6..8 adjacent to pit? Pit x20..24 far. ✓ Also wall under catwalk north edge? Fine.
- Short stairs (A 小 stairs) = catwalk→site ✓. Waypoint heights fine.

Catwalk east wall x=10 z 8..24, then A ramp area x6..10 z-12..-8, CT spawn... Wall x=10 z -8..8? Add segment x=10 z -8..8 to close A site east side between ramp and short extension: A site east edge x=20? Site x6..20: east side open toward long corner (z≤6)? At z 4..6 x20..? corner platform p1 ends at x20. Site east face x=20 z -8..6 visible. Wall above site? Site is platform; edges are vertical faces ✓. North side of site z=-8: wall block along z=-8 x6..20 h3.4 from y1.2? Wall behind A site (A wall? "A wall"? classic has "A doors"?). Place wall z=-8 x 6..20, y 1.2..4.6, with gap x 6..10 (ramp entry)? Ramp enters from north at z=-8 x6..10 → gap there: wall segments z=-8: x10..20. Also CT side x=10 wall z -12..-8? Ramp bounded east by wall x=10 z -12..-8 (y0 0..3.4), west side x=6 z -12..-8 wall. Ramp wedge between ✓.

CT spawn: x -8..10, z -20..-10. Walls: north z=-20 x -8..10; west x=-8 z -20..-10 with gap z -18..-14 (B corridor); east x=10 z -20..-14? then ramp walls handle z -14..-8? East boundary x=10 z -20..-12 solid, then ramp side wall x=10 z -12..-8 ✓ (continuous). South z=-10: segments x -8..-2 and x 4..10? Gap x -2..4 (CT mid entry) ✓; also gap x 6..10 for ramp? Ramp area z -12..-8 is north of z=-10 line... CT spawn z -20..-10; ramp z -12..-8 overlaps CT spawn z range (-12..-10). Hmm ramp x6..10 z-12..-8: part z-12..-10 within CT spawn bounds. South wall z=-10 x 4..6 solid; x6..10 open (ramp passage continues from CT spawn floor 0 → ramp rises). So south wall segments: x -8..-2, x 4..6. ✓ CT spawn also needs east wall x=10 z -20..-12 ✓ and no wall at z -10..-8 east side (ramp wall x=10 z -12..-8 covers).

B corridor (CT side): from CT spawn west gap (x=-8, z -18..-14) → corridor x -14..-8, z -18..-14 (walls z=-18 and z=-14, x -14..-8). B doors at x=-14 (panels z -18..-16.15 and z -15.85..-14, gap .3? two panels along z with gap in middle; slide open along z). Then B site side corridor x -20..-14, z -18..-14, then open into B site: B site north boundary z=-12? Corridor z -18..-14; area z -14..-12 x -20..-14 is opening (no walls), then site z ≥ -12. B site walls: north wall z=-12 x -26..-20 and x -14..-12 (gap x -20..-14 = entry ✓). West wall x=-26 z -12..6. East wall x=-12 z -12..-2? then tunnel side: B site east boundary x=-12 z -12..0 solid; z 0..4 open? Tunnel exit is south. B site south wall z=4? Site z -12..4: south wall z=4 x -26..-22 and x -18..-12, gap x -22..-18 (tunnel exit).
B platform: raised 0.6: block(-22,-9,-15,-3,0,0.6) with step 0.3 on south? Add step block(-22,-3,-15,-2.4? make two-tier: outer step block(-22,-3.6,-15,-2.4,0,0.3)?? Simplify: single 0.6 with step block along south edge: block(-22,-3,-15,-2.4,0,0.3) as step (0.3), then platform 0.6 → step up 0.3 ✓. Boxes on platform: crate (-18.5,-6) 1.6 (on 0.6 → top 2.2), another (-15,-8)? plus loose crate (-24,1) 1.3 on ground, barrel.
B site floor 0.

Tunnels:
- T spawn west gap x=-12 z 44..48 → upper tunnel corridor x -20..-12, z 44..48: walls z=44 and z=48 x -20..-12; ceiling slab y 2.4..2.7 x -20..-12 z 44..48. Then turn north at x -20..-12? West turn: corridor continues west? Classic upper tunnels: from T spawn go west then north. Let me do: west corridor x -22..-12 z 44..48 (walls), then north corridor x -22..-18 z 26..44 (walls x=-22 and x=-18), bend area x -22..-18 z 44..48 merges ✓ (corner). Then offset bend at z 26: north segment x -20..-16 z 8..26 (walls x=-20,x=-16), bend area z 24..28 x -22..-16 open (connecting south corridor (x-22..-18) and north corridor (x-20..-16)) — S-bend breaking LOS ✓. Then tunnel exit z=8: opening into vestibule x -22..-18? North corridor x -20..-16 exits at z=8 to area z 4..8: south wall z=4 gap x -22..-18? Align: vestibule x -22..-16 z 4..8 bounded by walls: east x=-16? Hmm B site east wall x=-12; area between tunnel x-20..-16 and site... Let me simplify: north tunnel corridor x -20..-16 z 8..26 exits at z=8 directly into B site south area: B site south wall z=4 x -26..-20 and x -16..-12, gap x -20..-16 z=4?? But site z starts -12..4; area z 4..8 x -20..-16 between tunnel end and site wall: bounded by walls x=-20 (z 4..8) and x=-16 (z 4..8). Then gap at z=4 x -20..-16 → into site ✓. So tunnel final straight z 8..4? Merge: corridor x -20..-16 z 4..26 with walls x=-20,x=-16 z 4..26, open at z=4 to site, ceiling throughout. ✓ Tunnel straight length ~22m + south segment 18m.
- Tunnel ceilings: slab y 2.4..2.8 over all tunnel segments (blocks view from above; adds enclosure). Also low lights? Skip.
- South tunnel segment x -22..-18 z 26..44 wait bend overlap: south segment z 26..44 x -22..-18; bend z 24..28 x -22..-16 (open, with ceiling); north segment z 8..26? overlap with bend z 24..26 fine.

Long doors corridor: T spawn east gap x=16 z 44..48? Set z 43..48. Corridor x 16..18, z 43..48 walls z=43,z=48 x 16..18? Then long corridor x 18..24 z 8..44? West wall x=18 z 8..44 — but corridor x16..18 z43..48 enters at x=18 z43..48 → gap in west wall z 43..48 ✓. Long doors at x=20? Doors should be right after turn? Classic long doors are between T spawn and Long A, at a kink. Place doors at x=19, z 43..48 (just after corridor start): panels z 43..45.4 and z 45.6..48, thickness .4, slide ±z. Then Long A x 18..24 z 8..43? West wall x=18 z 8..43 (gap z43..48 for entry), east wall x=24 z 8..44, then corner area. Long length: z 43→11 stairs ≈ 32m ✓ sniper line. Add slight cover in long: broken crate at (21,26)? Classic long has no cover ("no cover on long") — keep clean ✓.
- Long corridor north end z=8..11: stairs up: x 18..24 z 9..11: 3 steps of 0.3 up to 0.9 (corner platform). Walls continue around corner: east wall x=25 z 4..11? Corner platform x 12..25 z 5..9 (h 0.9) pieces around pit hole (x20..24, z5.5..8.5):
  pA block(12,5,20,9,0,0.9)
  pB block(24,5,25,9,0,0.9)
  pC block(20,8.5,24,9,0,0.9)
  pD block(20,5,24,5.5,0,0.9)
  Pit hole floor 0 ✓ walls = slab sides 0.9 ✓
  Stairs from long: corridor floor 0 z≥11; steps z 9..11 x 18..24 (0.3/0.6/0.9) ✓ then platform z 5..9.
  Step to site: site x6..20 z-8..6 h1.2; platform z 5..9 x12..20 h0.9: overlap zone x12..20 z5..6: ground = max(0.9,1.2?) site slab exists at z≤6: at z5..6 both: top 1.2 (site) — step face 0.3 at z=6 ✓ walkable.
  Hmm wait platform pA z5..9 x12..20 overlaps site slab x6..20 z-8..6 in z5..6 — embedded fine.
- Pit entry: from platform east side? Hole x20..24 z5.5..8.5 surrounded by platform 0.9 — jump in/out (0.9 < jump 1.5 ✓). Also from long stairs? Fine.
- Goose wall on site near long entry: low wall block(18.5,3,19.3,6,1.2,2.2)? Place at site edge overlooking pit/long ✓.

T spawn walls: south z=54 x -12..16; west x=-12 z 40..54 gap z 44..48; east x=16 z 40..54 gap z 43..48; north z=40 x -12..-2 and x 4..16 (gap x -2..4 mid). T spawn arches: lintel over mid gap: block(-2,39,4,40? decorative lintel y 2.6..3.4 spanning gap ✓ nice.
Crates in T spawn: (8,50) 1.5, (-6,46) 1.2 etc.

Mid walls from T: x=-2 z 4..40 (west), x=4 z 4..18, z 24..40 (east, gap 20..24). Also mid west side: "suicide" open area? Keep corridor width 6.

Add mid ceiling? None (open sky) ✓ classic.

Xbox at (1,10)? Mid doors at z=4; Xbox should provide cover from doors: place at (1,9) size 1.8 h1.8? Classic Xbox is about person-height crate. box(1,9,1.8,1.8,1.7,m crate).

CT mid walls x=-2,4 z -10..4 ✓. Mid doors at z=4 spanning x -2..4.

CT spawn south wall z=-10 x -8..-2 and x 4..6 (gap x -2..4 mid, x 6..10 ramp).

Also wall between CT spawn and A ramp east side ✓ done.

Also close map edges: big perimeter walls x=-34/x=34? Place boundary walls: along z=58 south? Add perimeter: block walls around x -30..30? Map extent: x -26..25 used; add perimeter walls x=-28 (z -20..54), x=27, z=-22, z=56, height 5, mat sand. Also fill gaps between structures with connecting walls to avoid leaks (bots/players escaping). Add some filler walls: e.g., between long corridor east wall x=24 and perimeter x=27: wall z 4..44 at x=27? Perimeter handles. Area east of A site x 20..27 z -8..5: wall at z=5 x20..27? Add: block(20,4.5,27,5, ...) hmm corner platform south edge z=5 x12..25; east x25..27 z4..9: wall x=25? pB covers x24..25. Add wall segment z=5 x 25..27? and x=27 handled by perimeter. Also area between corner platform and perimeter z 9..44 east of long wall x=24: sealed by perimeter x=27? Gap x24..27 z8..43: walled off by perimeter x=27 and long wall x=24 — inaccessible ✓ fine (sealed area).
Area between CT spawn and A site north: wall z=-8 x10..20 ✓; x20..27 z-8: add wall z=-8 x20..27 ✓. Area x6 west of ramp: wall x=6 z -12..-8 ✓ ramp west wall. Between CT mid west wall x=-2 and CT spawn west... CT spawn interior x -8..10 ✓.
Area between mid east wall x=4 and catwalk west... catwalk slab x4..10 z8..20 above; ground level x4..6? Wait mid east wall at x=4; catwalk slab x4..10 → slab starts at wall ✓. Area x4..10 z 20..24 ground level = stairs ✓. East of catwalk x10..? z8..24: wall x=10 ✓; beyond x10..27 z -8..24: big sealed area? A site x≤20, ramp... area x10..25 z -8..5 north of corner? Corner platform z5..9 x12..25; site z-8..6 x6..20; so area x20..27 z -8..5 sealed by walls z=5 (x20..27 add) and z=-8 (x20..27 add) and perimeter ✓ sealed.
Area x10..12 z 5..9? Between site east face x=20? No. West of corner platform x12: area x10..12 z 6..9: bounded by catwalk east wall x=10 (z8..24), site edge... add wall x=12 z 6..9? pA west face x=12 z5..9 ✓ vertical face 0.9; area x10..12 z6..9 ground 0 pocket: entry? Bounded by x=10 wall (z8..24 covers z8..9; z6..8 gap?) catwalk wall x=10 z 8..24; extension slab x4..10 z6..8 h1.2 covers west. Pocket x10..12 z6..8: bounded west by extension edge (x=10, z6..8, h1.2 face), north pA? pA z5..9 x12..20: east face x=12 ✓, south: corner? z=6 line: site slab x6..20 z≤6 → face at z=6 x10..12 h1.2 ✓, north z=8? pA starts z=5 so pA south face? pA z5..9: at z=9? Pocket z6..8 bounded north by pA? pA x12..20: its west face x=12 z5..9 ✓. Pocket sealed (no entry) ✓ fine.
I'll trust perimeter + these walls; small sealed pockets OK.

B area seals: between B site west x=-26 and perimeter -28: sealed. Between tunnel west wall x=-22 and perimeter: area x -28..-22 z 8..44: sealed by perimeter and walls, no entry ✓. North of B site z -12..-20 west of CT corridor: area x -26..-14 z -20..-14: bounded by CT spawn west wall x=-8? That area: B corridor x -14..-8 z -18..-14; area x -26..-14 z -20..-18: wall z=-18 x -26..-14 add ✓ (north side of B corridor). And z -20 perimeter? Perimeter z=-22 covers. Area x -26..-14 z -14..-12: bounded by corridor south wall z=-14 x -20..-14? Corridor walls z=-14 x -20..-8? Set corridor z -18..-14 walls x -20..-8? Earlier: corridor x -14..-8 (CT side) + x -20..-14 (B side) both z -18..-14: walls z=-18 x -20..-8 and z=-14 x -20..-8 ✓. Then area z -14..-12 x -26..-20: bounded by B site north wall z=-12 x -26..-20 ✓, north by corridor wall z=-14 ✓, west perimeter ✓, east: x=-20 z -14..-12: opening? That's the B entry corridor from doors to site: passage x -20..-14 z -14..-12 open (connects corridor to site) ✓ intended. And x -14..-12 z -14..-12: bounded by site north wall z=-12 x -14..-12 ✓ and site east wall x=-12 ✓ and corridor wall z=-14 ✓ → sealed pocket ✓ fine.

OK. Also need walls around T spawn to long corridor: corridor z 43..48 x16..18 walls z=43 x 16..18 and z=48 x 16..18 ✓; T spawn east wall x=16 gap z43..48 ✓.

Waypoints (final, [x,z]):
```
0 (2,48) T spawn
1 (13,46) 
2 (17,45.5) long corridor before doors
3 (21,45.5) after doors
4 (21,30) long middle
5 (21,12) long north end
6 (21,7) corner platform east? on platform (0.9) 
7 (15,7) corner platform west / base of site stairs
8 (15,2) A site south (on 1.2)
9 (10,-2) A site default west
10 (8,-9) ramp top? ramp z-12..-8: top at z=-8.5 → (8,-8.5)
11 (8,-14) CT spawn east
12 (0,-15) CT spawn center
13 (-6,-16) CT spawn west
14 (-11,-16) B corridor before doors
15 (-17,-16) after doors
16 (-17,-12.5) B entry
17 (-18,-6) B platform
18 (-18,0) B site south
19 (-18,6) tunnel vestibule? corridor x-20..-16 → wp (-18,6)
20 (-18,16) tunnel north
21 (-19,26) bend
22 (-20,36) tunnel south
23 (-14,46) tunnel T entry
24 (1,38) top mid
25 (1,26) mid south
26 (1,15) Xbox west? Xbox at (1,9): wp (1,15) south of Xbox? Path around Xbox: wp (1,15) then (1,12)? Xbox blocks center; path straight through x=1 hits Xbox (1.8 wide centered x1 → x .1..1.9). Move Xbox to (2.2,9)? Then west gap x -2..1.3 wide 3.3 ✓ path x=0 clear. Place Xbox at (2.2,9). wp 26 (0,12), 27 (0,6) mid doors T-side, 28 (0,0) mid CT-side? mid doors z=4: wp (0,6.5) and (0,1.5).
29 (5,22) short stairs bottom (gap z20..24: enter at (5,22))
30 (7,21) stairs top? stairs x4..8 z20..24 along x: top at x8: wp (7.5,22)? Let me do: 29 (4.8,22) bottom? Actually from mid (x≤4) enter east: bottom wp (5,22), top wp (8,22)? Then catwalk north: 30 (8,18), 31 (8,10), 32 (7,7) extension, 33 (9,3)? A site east-mid: (12,3)? Site x6..20: wp 33 (12,2).
```
Let me finalize indices:
```
W=[ [2,48],[13,46],[17,45.5],[21,45.5],[21,30],[21,12],[21,7],[15,7],[15,2],[10,-2],[8,-8.5],[8,-14],[0,-15],[-6,-16],[-11,-16],[-17,-16],[-17,-12.5],[-18,-6],[-18,0],[-18,6],[-18,16],[-19,26],[-20,36],[-14,46],[1,38],[1,26],[0,12],[0,6.5],[0,1.5],[5,22],[8,22],[8,16],[8,10],[7,7],[12,2] ]
```
Wait 26 (0,12): Xbox at (2.2,9) size 1.8 → x 1.3..3.1, z 8.1..9.9. Path x=0 clear ✓. 27 (0,6.5) before doors, 28 (0,1.5) after.
Links:
```
0-1,1-2,2-3,3-4,4-5,5-6,6-7,7-8,8-9,9-10,10-11,11-12,12-13,13-14,14-15,15-16,16-17,17-18,18-19,19-20,20-21,21-22,22-23,23-0,
0-24,24-25,25-26,26-27,27-28,28-12,
25-29? (1,26) to (5,22): path via gap z20..24 x4: wp 29 (5,22). Link 25-29? Distance ok straight line crosses x=4 wall? Wall x=4 z 24..40 and z 4..18; gap z 18..24? Earlier said gap z 20..24; wall x=4 z 4..18 and z 24..40 → gap z 18..24. Straight from (1,26)→(5,22) crosses x=4 at z=23 ✓ within gap ✓.
29-30 (8,22)? Stairs x4..8 z20..24: (5,22)→(8,22) along stairs ✓.
30-31 (8,16), 31-32 (8,10), 32-33 (7,7)? 32 (8,10) → 33 (7,7): crosses z=8 catwalk edge → extension (1.2) ✓. 33-8? 33 (7,7) → 9 (10,-2)? Crosses z=6 site edge ✓ link 33-9? Distance ~9.4 fine. Also 33-8 (15,2)? Add.
Also 6-? pit? Add pit wp 34 (22,7) (inside pit, ground 0): links 6-34 (drop), 34-5? Jump out to long stairs? 34-6 only (bots rarely use pit; fine).
Also link 16-18? B entry (−17,−12.5) → platform (−18,−6)=17 ✓ have.
Also 12-28 ✓, 11-? ramp ✓.
```
CT spots:
A: [(8,-8.5, toward site? ramp hold), (10,-2)? on site pre? CT can push site: spots: ramp top (8,-8.5), site default behind box (9,-3), site east near goose (17,3), short extension (7,7)? risky. Use: [[8,-9],[9,-3.5],[16,4],[7,6.5]]
B: [[-14.5,-16]? at doors inside corridor hold, [-17,-11] entry, [-16,-5]? platform edge? platform x-22..-15 z-9..-3: spot on platform (-16,-4), behind box (-20,-7), site south near tunnel (-17,1)]
Mid: [[0,0.5] CT-side of doors hold, [0,-8] deeper]
T post-plant guards A: [[10,-2]? near default box, [16,4] long side? on site, [7,7] short, [22,7] pit]
B: [[-18,-6] platform, [-20,-7], [-17,1] tunnel mouth, [-14.5,-13] entry]
Plant spots A: [[12,0],[16,-2],[9,-5]]; B: [[-18,-6],[-20,-4],[-16,-8]] — must be within site volume ✓ (B site x-26..-12 z-12..4 ✓; A site x6..20 z-8..6 ✓).

T routes (via lists):
A long: [1,2,3,4,5,6,7] → site
A short: [24,25,29,30,31,32,33]
Mid→short: same as short.
B: [23,22,21,20,19,18]
Mid split fake: [24,25,26,27] then decide? Keep 4 routes + variant A long with smoke at doors.

Smoke spots: T at wp2 (17,45.5) before long doors? Classic smoke is CT side of mid doors from Xbox/top mid: bot at 25/26 with smoke → smoke target mid doors CT side (0,2)? Actually smoke "mid doors" to block CT peek: land at (0,3.5). Trigger when bot near (1,15)±? Use: {at:[0,12], to:[0,3], type:'sm'} — bot near (0,12) throws smoke to (0,3) (lands before doors? Doors at z=4; CT peeks from z<4: smoke at (0,2.5) blocks view from CT mid through door gap ✓). Also A long: CT smokes? Keep T only.
Flash spots: A long entry: at (21,12) throw flash to (15,4) (over stairs to site); at (15,7)? Short: at (8,10) flash to (8,4)? Site. B: at (-18,6) flash to (-18,-2); at B doors (-14,-16) flash to (-17,-13).
Implement flashSpots: {at:[x,z], to:[x,z]}, bots within 2m of 'at' with flash and difficulty ≥ 1 and about to push (route active) → throw once (mark per round per bot).

CT pushback nades: CT holds spots: if enemy visible and dist > 8 && has HE → throw.

OK. Maps 2 and 3 similar data but simpler. Let me sketch Map 2 "RUIN":

Bounds x -30..30, z -26..34. Theme jungle stone.

- T spawn: x -8..8, z 26..33 (walls around, gaps: west x=-8 z 28..31; mid north z=26 x -2..2; east x=8 z 28..31).
- West lane: corridor x -14..-8? Let me do: west corridor x -16..-12, z 12..31? Connect T spawn gap (x=-8 z28..31) via connector x -16..-8 z 28..31? Simplify lanes:
  West lane: x -16..-12, z 2..31, walls x=-16,x=-12; T connector: x -16..-8 z 28..31 (walls z=31? T spawn wall handles; add walls z=28 x -16..-8? T spawn south... let me do: T spawn west gap x=-8 z 28..31 → corridor x -16..-8, z 28..31 east-west (walls z=28, z=31 x -16..-8), then turn north: x -16..-12, z 2..28? overlap at corner ✓.
  West lane leads to B site south entry at z=2? B site x -26..-12, z -8..2, sunken -1.2: stairs down x -16..-12 z 0..2 (0→-1.2, 4 steps).
- East lane mirrored: x 12..16, z 2..31, connector x 8..16 z 28..31; A site x 12..26, z -8..2, raised +1.2: stairs up x 12..16 z 0..2.
- Mid: x -2..2? Wider courtyard: x -6..6, z 6..26, walls x=-6 (z 6..26) and x=6, pillars inside: boxes at (-3,20),(3,20),(-3,12),(3,12) size 1.4, fountain block (0,16) 2.4x2.4x0.9? Cover. Mid north: hall x -3..3, z -4..6, with gate double doors at z=-4? Doors at z=-4 (panels x -3..-0.15, 0.15..3). CT side z -14..-4 x -3..3 corridor to CT spawn.
- CT spawn: x -10..10, z -24..-16; walls; gaps: south z=-16 x -3..3 (mid hall); west x=-10 z -22..-19 → CT west corridor x -16..-10 z -22..-19 → then north-south corridor x -16..-13? z -22..-10? Let me do: CT west connector: x -18..-10, z -22..-19 (east-west), then corridor x -18..-14, z -19..-8? then opens to B site north end z=-8? B site z -8..2: north entry gap x -18..-14 z=-8 ✓ (B site north wall z=-8 x -26..-18 and x -14..-12).
  CT east mirrored: x 10..18 z -22..-19, corridor x 14..18 z -19..-8, A site north wall z=-8 x 12..14 and x 18..26? A site x 12..26: north wall z=-8 x 12..26 gap x 14..18 ✓.
- B site sunken: floor -1.2: dig: cannot subtract → raise surroundings? B site sunken means surroundings higher — complex. Instead make B site floor 0 and surroundings +1.2? Also complex. Keep B site floor 0, entries flat; add cover: sarcophagus boxes, low walls; site slightly enclosed with walls h3. A site raised 1.2 with stairs ✓ (gives asymmetry: A high, B low enclosed).
- A site cover: altar block (19,-4) 2x2x1 on 1.2, crates (15,-1) 1.5, columns at corners.
- B site cover: boxes (-19,-2) 1.8, (-15,-5) 1.4, low walls.
- Cross connector between CT corridors behind sites: CT spawn itself ✓.
- T-side cross connector between lanes mid-way: corridor z 14..17 x -12..-6 (west lane to mid) and x 6..12 z 14..17 (mid to east lane) → flanks/rotation ✓.
Waypoints:
```
0 (0,30) T
1 (-10,29.5) W connector? path: T→west: (-4,29.5)? Let me list:
0 (0,30) T spawn
1 (-12,29.5) west connector
2 (-14,20) west lane south
3 (-14,8) west lane north
4 (-14,0) B stairs south
5 (-19,-2) B site
6 (-16,-7) B site north
7 (-16,-12) CT west corridor south? corridor x -18..-14 z -19..-8: wp (-16,-12)
8 (-16,-20) CT west connector
9 (0,-20) CT spawn
10 (16,-20) CT east connector
11 (16,-12) CT east corridor
12 (16,-7) A site north
13 (19,-3) A site
14 (14,0) A stairs north
15 (14,8) east lane north
16 (14,20) east lane south
17 (12,29.5) east connector
18 (0,22) mid south? courtyard z6..26: wp (0,22)
19 (0,14) mid center (fountain side)
20 (0,7) mid north before hall
21 (0,-2) hall CT-side of doors? doors z=-4: wp (0,2) T-side, (0,-6) CT-side
22 (-9,15.5) west cross corridor? corridor z14..17 x -12..-6: wp (-9,15.5) links mid (−6?) to west lane: link 19-22? (0,14)→(-9,15.5) crosses x=-6 wall at z≈15 ✓ within gap z14..17 ✓. 22-2? (-9,15.5)→(-14,8)? crosses x=-12 wall at z≈? line from (-9,15.5) to (-14,8): at x=-12 z≈11.3 — wall x=-12 z 2..28 solid → blocked! Fix: cross corridor connects to west lane via gap in x=-12 wall at z 14..17 ✓ so wp 22a (-9,15.5), 22b (-13.5,15.5)? then link 22b-2? (-13.5,15.5)→(-14,8) along lane ✓ (lane x-16..-12, wp x=-14 ✓). Add wp 23 (-13.5,15.5)? Merge: 22 (-10,15.5), 23 (-14,15.5) link 19-22? crosses x=-6 at z≈14.9 ✓ gap ✓; 22-23 ✓ (crosses x=-12 at z 15.5 ✓ gap); 23-2 ✓.
 East mirrored: 24 (10,15.5), 25 (14,15.5), links 19-24,24-25,25-15.
Also mid doors wps: 26 (0,2), 27 (0,-6); link 20-26,26-27,27-9? (0,-6)→(0,-20)? hall corridor x-3..3 z -14..-4 then CT spawn gap z=-16 x-3..3: hall z -14..-4; CT spawn z -24..-16; area z -16..-14 x -3..3 connects ✓. Link 27-9 straight crosses z=-16 wall? Wall z=-16 x -10..-3 and x 3..10 (gap x -3..3) ✓ line x=0 passes gap ✓.
```
Links list accordingly.
Sites: A x12..26 z-8..2; B x-26..-12 z-8..2.
CT spots A: [[16,-7],[19,-6]? behind altar, [13,-1] on stairs top? [22,-4]? east edge? use [[16,-7],[20,-5],[14,-1]]
B: [[-16,-7],[-20,-6],[-14,-1]]
Mid: [[0,-6],[0,-12]? hall]
T routes: A: [17,16,15,14]; via mid→A? Mid doesn't connect to A except via CT hall (no). Routes: A east lane, B west lane, mid push → hall → CT spawn → split to A/B (mid-to-CT attack route: interesting). Also cross corridor flank: route B via cross: [18,19,22,23,2,3,4]. A via cross east.
Plant spots A: [[19,-4],[15,-4],[22,-1]] within site ✓; B: [[-19,-2],[-16,-5],[-22,-1]].
Smoke: mid gate: at (0,7) throw to (0,-5)? Doors at z=-4: smoke (0,-5) blocks CT. Flash: A stairs at (14,4) to (16,-2); B stairs (-14,4) to (-16,-2); hall at (0,0) to (0,-8).

Map 3 "DEPOT" (night industrial):
Bounds x -32..32, z -30..36.
- T spawn x -7..7, z 28..35.
- Mid rail: east-west strip z 10..15 open with train cars: two train boxes: (-20..-8, z 11.5..14, h3) and (6..20, z 11.5..14, h3), with gaps at x -8..6 (mid crossing) and ends. T → mid approach: yard x -6..6 z 15..28 with container covers: containers at (-3,22) size 2.4x6? Oriented north-south: box(-4,22,2.4,7,2.6), box(4,19,2.4,7,2.6) staggered.
- A site east: dock platform x 14..30, z -10..4, h 0.9; cover: container stacks: box(20,-4,2.6,6,2.6) on platform? Plus crates; crane base block (27,-7) 3x3x3? Entries: from mid east along rail gap x 20..30? Rail z10..15 east end x 20..30 open → approach A south: A south edge z=4: stairs up x 16..20 z 4..6 (0→0.9)? Place stairs x 14..18 z 4..6. Also east lane from T: corridor x 10..14, z 15..28 (walls x=10,x=14) → exits at z=15 to rail area x10..14? Then around containers to A south. Also CT east corridor: x 22..26, z -22..-10 → A north entry gap z=-10 x 22..26 (A north wall z=-10 x 14..22 and x 26..30).
- B site west: warehouse x -30..-14, z -12..2: floor 0, ceiling slab y 4..4.4 over site, walls with openings: south wall z=2 x -30..-24 and x -20..-14 (gap x -24..-20 south entry from rail west end? Rail z10..15; area z 2..10 x -24..-20 corridor from rail to B ✓ walls x=-24,x=-20 z 2..10); north wall z=-12 x -30..-26 and x -22..-14 (gap x -26..-22 → CT west corridor x -26..-22, z -22..-12); east wall x=-14 z -12..-4? then gap z -4..2 (east entry from mid west via yard x -14..-10?) mid west area: yard x -10..-6? Add west flank corridor x -14..-10, z 2..15 connecting rail to B east entry ✓ walls x=-10 z 2..15? and warehouse wall x=-14 z -12..2 with gap z -4..2 ✓.
  B cover: container rows inside: box(-24,-6,2.4,5,2.6), box(-19,-2,2.4,5,2.6), crates (-27,-9) etc. Catwalk? Skip.
- CT spawn x -8..8, z -28..-20; gaps: south z=-20 x -3..3 (mid approach north? CT → mid: corridor x -3..3, z -20..-10? then opens to rail z 10..15? Area z -10..10 x -6..6 central yard with containers; walls x=-3? Let me do: CT mid corridor x -3..3 z -20..-8 (walls), then opens to central yard x -6..6 z -8..15? Yard bounded by walls x=-6 (z -8..15) and x=6, with container cover inside; south opens to rail z=15 ✓ (gap x -6..6 at z=15).
  CT east connector: from CT spawn east gap x=8 z -26..-23 → corridor x 8..26? East-west corridor z -26..-23 x 8..22, then south corridor x 22..26 z -22..-10 → A north ✓. Walls accordingly.
  CT west mirrored: corridor z -26..-23 x -22..-8, then x -26..-22 z -22..-12 → B north ✓.
- Long sniper lane: east lane x 10..14 z 15..28 straight 13m? Extend: from T spawn east gap x=7 z 30..33 → connector x 7..14 z 30..33? then lane x 10..14 z 15..30: total straight ~15m + across rail to A ~25m. Also rail straight east-west ~50m sniper line through train gaps ✓.
Waypoints:
```
0 (0,31) T
1 (10,31.5)? east connector: T east gap x=7 z30..33 → corridor x 7..10? Let me simplify: east lane walls x=10..14 z 15..30; T connector: x 7..14 z 30..33 (walls z=30? T spawn wall z=28? T spawn z 28..35, wall z=28 x -7..7 gap? Hmm T spawn: walls: south z=35, west x=-7, east x=7, north z=28 gaps: mid x -3..3? no—T north wall z=28: gaps x -6..-3 (west yard route), x -1..1? Let me do: gaps: x -6..-2 (to west yard), x 2..6 (to east yard)? East lane needs x10..14: connector corridor z 28..30 x 2..14? Add: T north wall z=28 x -7..-6? Getting messy. Simplify T exits: north gap x -2..2 → central yard (main); west gap x=-7 z 30..33 → west connector corridor x -14..-7 z 30..33 → west lane x -14..-10 z 15..30; east gap x=7 z 30..33 → connector x 7..14 z 30..33 → east lane x 10..14 z 15..30.
Wps:
0 (0,31) T
1 (-10,31.5) west connector
2 (-12,24) west lane south
3 (-12,17) west lane north (at rail)
4 (-12,12.5)? rail west within z10..15: (-12,12.5) rail west
5 (-22,12.5) rail west end near B south corridor
6 (-22,6) B south corridor
7 (-22,-2) B site south? warehouse z -12..2: wp (-22,-2)
8 (-24,-8) B site north-west
9 (-24,-14) CT west corridor south? corridor x -26..-22 z -22..-12: wp (-24,-16)
10 (-16,-24.5) CT west east-west corridor? corridor z -26..-23 x -22..-8: wp (-16,-24.5)
11 (0,-24) CT spawn
12 (16,-24.5) CT east corridor
13 (24,-16) CT east corridor south
14 (24,-8) A north entry? A z -10..4: wp (24,-8)
15 (22,-2) A site east
16 (16,0) A site west / stairs top? stairs x14..18 z4..6: wp (16,3)? on platform (16,2)
17 (16,8) A south below stairs? ground 0: (16,7)
18 (12,12.5) rail east
19 (12,20) east lane north? lane x10..14 z15..30: wp (12,20)
20 (12,28) east lane south
21 (10,31.5) east connector
22 (0,24) T yard north? yard x -6..6 z 15..28? wait central yard z -8..15 bounded x±6; T → yard: T north gap x -2..2 z=28 → corridor x -2..2 z 15..28 (walls x=-2,x=2)? then yard z 15 x -6..6 opens. Wps: 22 (0,22) corridor, 23 (0,8) yard south? yard z -8..15: wp (0,8), 24 (0,0) yard north, 25 (0,12.5)? rail mid crossing x -8..6 gap: yard meets rail at z15? Rail z10..15: yard south edge z=15 → step into rail area. wp 25 (-1,12.5) rail center.
Links rail: 4-5 (west rail, along z12.5 passing under/around train? Train cars (-20..-8): wp x -12 within train x range → train occupies z11.5..14! Wp at z12.5 collides with train box. Move rail path to z 9.5 (south of trains)? Rail strip z10..15; trains at z 11.5..14 → pass along z 10.5 south side ✓ set rail wps z=10.6: 4 (-12,10.6), 5 (-22,10.6), 25 (-1,10.6), 18 (12,10.6)? East train x 6..20 z11.5..14: pass at z10.6 south of it ✓ then 18 (12,10.6) → 17 (16,7)? crosses z 4..6 stairs area? fine.
Also link 3-4 (west lane → rail), 19-18? East lane north (12,20)? Lane x10..14 z15..30 ends at z=15 → rail z10..15: link 19 (12,20) → 18 (12,10.6)? crosses z15 wall? Lane opens at z=15 (no wall) ✓ but crosses train? Train x6..20 z11.5..14: line x=12 z20→10.6 crosses train z11.5..14 at x=12 → blocked! Detour: gap between trains x -8..6 is mid; east side: pass around east end of east train x>20: waypoint 19 (12,20) → (21?) add wp (22,12)? Path: (12,20)→(12,16) then east to (22,12)? Crosses train x6..20 z11.5..14 at x12..22 z≈? Line from (12,16) to (22,10.6): at z=14 x≈15.7 <20 → blocked. Route around: (12,16)→(22,15.2)? then (22,10.6): along x=22 z15.2→10.6 crosses z14..11.5 at x=22 > 20 ✓ clear. Add wps: 26 (12,16), 27 (22,15.3)? Hmm z15.3 barely south of train z11.5..14 ✓. Then 27→18? (22,15.3)→(12,10.6): crosses train? At z=14 x≈? Line from (22,15.3) to (12,10.6): parametric z 15.3→10.6, x 22→12: at z=14: t=(15.3-14)/4.7=0.276 → x=22-10*0.276=19.2 <20 → blocked (train x6..20). Instead 27→17 directly (16,7): (22,15.3)→(16,7): at z=14: t=1.3/8.3=0.157→x=22-6*.157=21.1>20 ✓; at z=11.5: t=3.8/8.3=0.458→x=19.25<20 ✗ blocked (train extends z11.5..14 x6..20; at z=11.5 x=19.25 within 6..20 ✗). Ugh. Trains are annoying. Simplify: shorten east train to x 6..16 → gap x16..30 open east. Then 27 (20,12.5)? Let me redo: trains: west x -20..-8, east x 4..16 (z11.5..14, h3). Open gaps: x -32..-20, x -8..4 (mid), x 16..32. Rail wps: west route: 5 (-22,12.5) west of west train ✓ (x -22 < -20 ✓). Mid gap wp 25 (-2,12.5) ✓ (within -8..4 ✓). East gap wp 18 (20,12.5) ✓ (>16 ✓). Links: 4? Redo west lane wp: 3 (-12,17) → 4 (-12,12.5)? crosses west train x -20..-8 z11.5..14 at x=-12 → z17→12.5 crosses z14..11.5 at x=-12 ∈[-20,-8] ✗. Move west lane to x -14..-10? Train starts x -20..-8: lane x=-12 still crosses. West lane meets rail at z15; to reach B south corridor x -24..-20: go west along rail south side z 10.5? South side of train: z 10..11.5 strip: wp z=10.7: 4 (-12,10.7): from 3 (-12,17) straight crosses train z11.5..14 ✗ again (x=-12 within train). So lane must bend: at z=15, move to z10.7 strip via x < -20 or x ∈ (-8..4): nearest: west of train x<-20: path 3 (-12,17) → (-22,17)? open area z15..? Area between lane and rail z 15..? Open yard west: fine: wp 3 (-12,17) → 4 (-22,16)? then 5 (-22,12.5) ✓ (x=-22 clear of train ✓) → 6 (-22,6). Similarly east: 19 (12,20) → 26 (20,16) → 18 (20,12.5) ✓ (x=20 >16 ✓) → 17 (16,7)? (20,12.5)→(16,7): crosses train z11.5? At z=11.5: t=1/5.5=.18→x=19.3>16 ✓ clear ✓.
Mid yard → rail: yard south edge z=15 x -6..6 → wp 25 (-2,12.5) within gap ✓ link 24? Yard wps: 23 (0,8)? wait yard z -8..15: rail at z10..15 overlaps yard z10..15?? Conflict: yard x -6..6 z -8..15 and rail strip z10..15 full width → overlap area x -6..6 z10..15: fine, same ground 0, yard walls x=±6 z -8..10 only (stop at rail). Adjust: yard walls x=-6 z -8..10, x=6 z -8..10. Yard open to rail at z=10 ✓ and to T corridor at z=15? T corridor x -2..2 z 15..28 walls. So yard z -8..15? South part z10..15 merges with rail ✓. Wps: 23 (0,6), 24 (0,-2)? yard north opens to wall z=-8? Yard north edge z=-8: wall z=-8 x -6..-3 and x 3..6, gap x -3..3 → corridor x -3..3 z -20..-8 to CT ✓ (CT mid). Wps 24 (0,-5), 28 (0,-12) corridor, 29? Link to CT 11 (0,-24): crosses CT south wall z=-20 gap x -3..3 ✓.
Also yard → B east flank: corridor x -14..-10 z 2..15? Connects rail west? At z 2..10 x -14..-10: bounded by walls x=-10 z 2..15 and warehouse wall x=-14. Entry from rail: at z=10? Rail z10..15: corridor x -14..-10 z 2..15 open at z=15?? At z15 area x -14..-10: west lane x -14..-10 z15..30 ends → merges ✓ so corridor x -14..-10 z 2..30 = west lane extended! Great: west lane x -14..-10 z 2..30 straight (walls x=-14,x=-10 z 2..30), opens at z=2 to warehouse east gap (warehouse wall x=-14 gap z -4..2 ✓ adjacent). So west lane wp: 2 (-12,24), 3 (-12,12)? Wait lane x -12 center z 2..30 crosses rail z10..15: train west x -20..-8 z11.5..14 → lane x=-12 crosses train!! Move west train to x -24..-14? Then lane x -14..-10 clear (train x -24..-14, gap x -14..-8? Gap -14..-8 width 6 ✓ mid gap x -8..4 ✓). West train x -24..-14. Then B south corridor x -24..-20 conflicts with train x -24..-14 z11.5..14? B south corridor z 2..10 (south of rail) ✓ fine, enters rail at z10..15 at x -24..-20: train covers x -24..-14 z11.5..14 → corridor x -24..-20 blocked by train at z11.5..14! Shift B south corridor to x -28..-24? Warehouse x -30..-14: south wall gap x -28..-24; corridor x -28..-24 z 2..10; rail entry: x -28..-24 < -24 ✓ west of train ✓. Wps: 6 (-26,6), 5 (-26,12.5), then along rail: 5→4? (-26,12.5) to west... rail west end → around? West of train is open x -32..-24 ✓. Rail path west→mid: 5 (-26,12.5) → 25 (-2,12.5): crosses west train x -24..-14 z11.5..14 at z=12.5 ✗!! Rail traversal must go around trains via gaps: gap x -14..-8: path 5 (-26,12.5) → (-17,12.5)? crosses train ✗. So rail traversal only via mid gap x -8..4 and east gap x>16: from B side, reach mid gap: 5 (-26,12.5) → (-11,12.5)?? crosses train x -24..-14 ✗. Hmm: B side rail access only west of train (x<-24) — dead end except into B corridor. That's fine: rail west segment serves B approach only. Mid gap x -8..-4 reachable from yard (yard x -6..6 ✓ adjacent). So rail as through-route: yard ↔ mid gap; east gap ↔ A/east lane. West segment: B only. OK realistic (trains block).
Final rail wps: 25 (-2,12.5) mid gap; 18 (20,12.5) east gap; 5 (-26,12.5) west pocket; links: 25-24? Yard: 23 (0,6) → 25 (-2,12.5)? crosses z=10 wall? Yard walls x=±6 z -8..10: no wall at z=10 line (open) ✓. 25-18? crosses east train x4..16 z11.5..14 at z=12.5 ✗. So 25 and 18 not directly linked: route via south? South of trains z<11.5: strip z 10..11.5 width 1.5 too narrow? Or north z 14..15 strip width 1: no. So mid gap and east gap connect only via yard → T corridor → east lane? Or via south of trains: open area south of rail z 15+? North of rail z<10: open ground x -8..30 z 2..10? Area between rail (z10) and warehouse/A south (z~4): open yard east: x -14..30 z 2..10 with scattered containers → connect mid gap to east gap south of trains ✓: wps 30 (-2,8)? Hmm z 8 south of train ✓: 25 (-2,12.5) → 30 (-2,9)? crosses z=10? no wall ✓ → 31 (10,6)? containers scattered — keep this zone mostly clear with 2 containers at (2,5)? and (12,7)? Path wps: 30 (0,7), 31 (10,6), then 17 (16,7)? 31→17: (10,6)→(16,7) ✓ clear. And 18 (20,12.5) → 17 (16,7) ✓ (as before). East gap also links to 26 (20,16) → east lane.
OK good enough; bots will manage.
```
This is getting complex; let me finalize Depot waypoints:
```
0 (0,31) T
1 (-10,31.5) W connector (corridor x -14..-7 z 30..33)
2 (-12,24) west lane
3 (-12,17) west lane mid
4 (-12,6) west lane south end (z 2..30 lane; at z=6)
5 (-16,0)? warehouse east entry: gap z -4..2 x=-14: wp (-13,-1)? Let me do: 5 (-16,-2) B site east? within warehouse x -30..-14: wp (-16,-2)
6 (-24,-4) B site west
7 (-24,-9) B site NW
8 (-24,-16) CT west corridor
9 (-16,-24.5) CT west east-west corridor
10 (0,-24) CT spawn
11 (16,-24.5) CT east east-west corridor
12 (24,-16) CT east corridor
13 (24,-6) A site NE? A x14..30 z -10..4 h0.9: wp (24,-6)
14 (20,-2) A site middle
15 (16,2) A site west (top of stairs)
16 (16,7) A south ground
17 (20,12.5) rail east gap
18 (20,16) east rail north? open area z15..: (20,16)
19 (12,20) east lane north (lane x10..14 z15..30)
20 (12,27) east lane south
21 (9,31.5) east connector
22 (0,22) T corridor (x -2..2 z15..28)
23 (0,8) yard south? yard x -6..6 z -8..10 + rail overlap: wp (0,8)
24 (0,0) yard north
25 (-2,12.5) rail mid gap
26 (-26,12.5) rail west pocket
27 (-26,6) B south corridor (x -28..-24 z2..10)
28 (0,-12) CT mid corridor
29 (10,6) south strip east
30 (-8,6)? south strip west: link 4? West lane south end (−12,6) → warehouse east gap: 4 (-12,6) → 5 (-16,-2)? crosses warehouse wall x=-14 at z≈? Line (-12,6)→(-16,-2): at x=-14: t=0.5 → z=2 ✓ gap z -4..2 ✓ (just). OK link 4-5.
Also 26-27 ✓, 27-? B south corridor into warehouse south gap z=2 x -28..-24: 27 (-26,6) → 6 (-24,-4)? crosses z=2 at x≈-25.3 ∈[-28,-24] ✓. Link 27-6.
Links: 0-1,1-2,2-3,3-4,4-5,5-6,6-7,7-8,8-9,9-10,10-11,11-12,12-13,13-14,14-15,15-16,16-17? (16,7)→(20,12.5): crosses east train x4..16 z11.5..14? At z=11.5: t=(11.5-7)/5.5=0.818→x=16+4*0.818=19.3>16 ✓. Link ✓. 17-18,18-19,19-20,20-21,21-0,0-22,22-23,23-24,24-28,28-10,23-25,25-? 25 to 17? crosses east train ✗; 25-29? (-2,12.5)→(10,6): crosses east train z11.5..14 x4..16: at z=11.5: t=(12.5-11.5)/6.5=0.154→x=-2+12*0.154=-0.16 <4 ✓; at z=14: t=neg? z decreasing from 12.5 → only crosses z=11.5 ✓ x=-0.16 not in 4..16 ✓ clear. Link 25-29 ✓. 29-16? (10,6)→(16,7) ✓ link 29-16. Also 25-26? (-2,12.5)→(-26,12.5) crosses west train x -24..-14 ✗. No link. 26-27 ✓. Also 3-26? (-12,17)→(-26,12.5): crosses west train z11.5..14? At z=14: t=(17-14)/4.5=0.667→x=-12-14*0.667=-21.3 ∈[-24,-14] ✗ blocked. Skip.
Also 18-29? (20,16)→(10,6)? crosses east train z11.5..14 x4..16: at z=14: t=0.2→x=18 → >16 ✓; at z=11.5: t=0.45→x=15.5 ∈[4,16] ✗ blocked. Skip.
```
Sites: A x14..30 z-10..4 (platform 0.9); B x-30..-14 z-12..2.
CT spots: A [[24,-8]? north entry, [20,-6]? behind container, [15,0] west]; B [[-24,-10] north entry, [-20,-8], [-26,-3]? south-west]; mid [[0,-10]? corridor end? yard north (0,-4)].
T routes: A: [21,20,19,18,17,16]; via mid strip: [22,23,25,29,16]; B: [1,2,3,4,5]; B south: [1,2,3,? no link to 26... skip]; mid→CT? Yard→CT corridor route [22,23,24,28] then CT spawn → split (attack CT spawn!): fun.
Plant A: [[20,-4],[24,-2],[16,-6]]; B: [[-22,-6],[-26,-8],[-18,-2]].
Smoke: at 23 (0,8) smoke to (0,-9)? Block CT corridor view (CT pushes from corridor z -20..-8): smoke at (0,-7) ✓. Also A: at 19 (12,20)? Smoke A stairs? From (12,20) to (16,5)? Meh: {at:[12,17], to:[16,5]}.
Flash: A: at 16 (16,7) to (18,0); B: at 4 (-12,4)? to (-18,-2); at 27 (-26,4) to (-24,-4).

Good enough. Now heights: A platform 0.9 via block; stairs x14..18 z 4..6: 3 steps of 0.3.

Warehouse ceiling: slab y 4..4.5 x -30..-14 z -12..2 → dark inside; add light shaft? Skip; ambient handles.

Train cars: boxes h3 mat metal tinted, wheels? Skip.

Containers: various tinted boxes h2.6.

Now per-map theme: {sky:[r,g,b], fog:[r,g,b], fogN, fogF, sun:[x,y,z], amb:'wind'|'jungle'|'hum', name}.

Dust2: sky (0.55,0.7,0.9), fog (0.82,0.76,0.62) N 30 F 120, sun normalize(-0.5,0.8,0.35)? warm.
Ruin: sky (0.6,0.75,0.65), fog (0.62,0.7,0.6) 25..90, sun higher.
Depot: sky (0.05,0.07,0.12), fog (0.08,0.1,0.16) 20..80, sun (0.2,0.7,-0.4) bluish dim, brighter ambient? Night: lower sun diffuse, higher ambient blue. Also floodlight fake: bright ground patches? Skip.

Character tint: T: shirt (0.55,0.45,0.32), pants (0.35,0.3,0.25), skin (0.75,0.6,0.45); CT: shirt (0.2,0.25,0.4), pants (0.15,0.18,0.28), skin (0.7,0.58,0.45). Vest darker.

Now weapon view model geometry per class: rifle: body box 0.09x0.12x0.55 dark + magazine + stock + barrel; pistol smaller; sniper long + scope cylinder (box); shotgun fat; SMG compact. Build via drawGun(p) function that pushes boxes with muzzle transform. Compute basis: fwd/right/up from camera yaw/pitch + bob offset + recoil offset.

Muzzle world pos for tracer origin: eye + fwd*0.55 + right*0.10 + up*-0.12 (approx).

Now audio implementation concretely:

```js
const AC = new (AudioContext)(); created on first click.
master = gain(0.8) → destination? Add compressor.
noiseBuf: 1s white noise.
function env(g,t0,a,d,peak)...
sfxShot(type, vol, pan): 
 pistol: noise burst hp 1200 + square osc 250→80 60ms
 rifle: noise lp 3000 bigger + osc 180→60
 awp: huge lp 1500 + sub
 smg: smaller
 shotgun: wide lp 800 boom
 use: src=noiseBuf, filter freq, gain env exp decay .08-.25s + click osc.
sfxStep(vol,pan): noise lp 500, 50ms, vol .1
sfxExplo: noise lp 400 1.2s + sine 90→30 .8s
sfxFlash: sine 2800 ring 1.2s? + pop
sfxBeep(f): square 0.07s
sfxHit: tick 1200 20ms
sfxHS: metallic: square 2100 + 2600 short "tink"
sfxPlant: beep sequence handled
sfxStinger(win): notes [392,523,659] triangle; lose: [392,330,262]
ambient: loop noise through lowpass 220 gain .03 + slow LFO.
```
Pan: StereoPanner if available.

3D helper: s3(fn, pos): compute d, pan, vol scale 1/(1+d*0.08)? and call fn(vol,pan).

Now let me write bot names and details.

Match init:
```
function startMatch(cfg){ map=buildMap(cfg.map); players=[]; me = players[0] team cfg.team; fill bots; scores 0; round=1; half=1; lossStreaks [0,0]; startRound(); state menu→game; request pointer lock on canvas click. }
function startRound(){ positions reset per team spawn points (list of spawn positions per map per team: generate grid within spawn rect); bot buy; me keep weapons if survived prev? Track per-player persist: on round reset: if p.aliveLast → keep weapons (refill reserve), else default loadout (pistol + knife) unless first round → pistol. Bomb → assigned to random T (bot or player if T). G timers.}
```
Spawn points: map.spawns = {T:[ [x,z]... ], CT:[...]} generate up to 5 each.

Player spawn assignment: shuffle.

Now local player team can be T or CT; if T, can carry bomb.

Buy zone check: within spawn rect of own team & time OK.

Now let me write the fire function:

```js
function tryFire(p){
 const w = curWeapon(p); // {def, mag, reserve}
 if(p.reloadT>0||p.switchT>0) return;
 if(w.def.slot===3){ knife swing: rate; dmg ray short }
 if(w.def.slot===4){ // grenade primed → handled by throwGrenade
 }
 if(w.mag<=0){ click sound; auto reload; return }
 if(now < p.nextShot) return;
 p.nextShot = now + 60/w.def.rpm;
 w.mag--; p.shots++;
 spread compute; for pellets: dir jitter; raycast; apply.
 recoil: p.pitch += kick... (for bot adjust aim? skip)
 muzzle flash timer, shell, tracer, sound (3D for others), alert nearby bots.
 if(w.mag===0) auto reload start.
}
```
Grenade throw: if slot 4 and grenade selected and click: throwGrenade(p, type): consume, spawn grenade entity with velocity; switch to previous weapon? CS switches to previous automatically after throw. Implement: after throw, revert to last weapon slot.

Grenade slot UI: cycle types owned with 4; show icon name.

Reload: p.reloadT = def.reload; on complete: transfer ammo. Shotgun shells one by one: reloadT per shell; can interrupt.

Switch: p.switchT=0.35.

ADS: p.ads (0..1 lerp); sniper scoped only if p.ads>0.95 → accurate. FOV = base(75) - ads*(def.zoom? 55 : 12).

Now bot fire uses same tryFire with p.trigger=true; bot pitch/yaw set by aim.

Bot aim compute:
```
target = enemy chest pos + lead? (skip lead or small: + enemy.vel*0.1)
desired yaw/pitch = dirTo(target + err)
err: gaussian sigma by difficulty and distance: sigma = diff.aim*(0.5 + d*0.02)
turn: max turn rate*dt toward desired.
fire if angleTo < threshold and reactT elapsed and LOS.
```
Bots also burst: fire 3-6 shots then pause 0.2-0.4 (except SMG/longer for easy).

Bot crouch on engage (hard+) sometimes: p.crouch=true for stability.

Now damage → bot reaction: if victim is bot and attacker known → set lastKnown, if not engaging → turn.

Team wipe check: after death: if all T dead → CT win (unless bomb planted and timer running → round continues! CS: if T all dead but bomb planted, round continues). Implement: elimination win only if bomb not planted.

Time out: if bomb not planted → CT win.

Now bomb plant function:
```
G.bomb = {state:'carried'|'dropped'|'planted', pos, carrier, timer:40, plantProg, defuseProg}
```
Carrier dies → state 'dropped', pos at death location.
Pickup: T player near dropped bomb presses E → carrier.
Plant: carrier in site & grounded & speed<2 & hold E: plantProg += dt/3 (global: if any planter channeling); else decay *? plantProg = max(0, plantProg - dt/3? decay slowly). On ≥1 → planted at planter pos (clamp in site), state 'planted', timer 40, message + sound, planter +300, T can leave; bomb beep interval decreasing.
Defuse: CT within 1.8 of planted bomb hold E & no? Allow under fire too (CS allows): defuseProg += dt/(kit?5:10); on ≥1 → CT win.
Explode: timer ≤ 0 → big explosion at bomb pos: kill all within 12 (huge dmg), T win.
```
Beep: sfx beep every interval (2s → 0.3s near end) at bomb pos (3D) + red icon blink.

Now round end function endRound(winner, reason): award money, message, state 'end', t=4.5s, then nextRound or halftime or match end. Also drop? Keep weapons for survivors.

Also "保枪" (save): implicit — survivors keep weapons; player can choose not to buy.

Also add "drop gun" G: drops current weapon (slot 1/2) as pickup entity; switch to knife? To pistol if have else knife.

Pickup entity: {w: weaponId, pos, t? never expire until round end? Persist through round; clear each round}. E near → if same slot, replace (drop current), else if empty slot, fill.

Now UI buy menu data:

Categories: 1 pistol (deagle), 2 SMG (mp5), 3 rifle (ak/m4/galil/famas), 4 sniper (ssg/awp), 5 shotgun (nova), 6 equipment (armor, armor + helmet, defuse kit CT only), 7 grenades (he, flash, smoke, molotov).
Show only team-valid rifles. Buy → deduct, grant. Grenade limits: he 1, flash 2, smoke 1, molotov 1.

Also auto refill? No.

Scoreboard: name, K, D, money, alive status.

Kill feed entries: "killer [wpn icon] victim" with HS marker.

Radar drawing: scale = 180/mapSize; draw walls from solids (rects where y1>0.5) dark, floors lighter? Draw base bg, then wall rects, then site areas with dashed outline A/B letters, player arrow (white local, green teammates), bomb red blink when planted/dropped, no enemies.

Rotate radar? Keep north-up.

Now main menu map thumbnails: reuse radar drawing function to small canvas.

Pause: Esc → if pointer locked, unlock and show pause; resume button relocks. Also P? Esc fine. Note: browser releases pointer lock on Esc; detect pointerlockchange → if in game and not menu → show pause.

Also prevent context menu (right-click ADS).

Handle death: me.alive=false → spectate mode: find living teammate; camera = their eye + their view; click to cycle; if none → overview free camera at map center? Just wait.

Match end overlay: winner, score, MVP (most kills), buttons: rematch (same settings), main menu.

Also "restart" from pause: restart match.

OK, also handle window resize.

Now — let me write it all. Also include favicon? Skip.

Another consideration: performance of groundAt/collision iterating all solids per player per frame: 10 players × ~300 solids × few checks = fine.

Bullet raycast per pellet: 300 slab tests × 8 pellets × maybe 3 shots per frame = 7k tests — fine.

Bot LOS: think staggered.

Now let me write the code. I'll use `let`/`const` and arrow functions for compactness. I'll write about 2000+ lines. Let me go.

Actually one simplification for view model: draw gun as part of scene with same projection (FOV). Fine.

Scope overlay: DOM div with radial-gradient mask (two circles) + crosshair lines.

Also handle: weapon FOV zoom applies to projection.

Camera roll? Small roll on strafe? Skip.

Head bob: view model bob and slight camera y bob.

Let me write the final answer: brief intro (Chinese), the complete HTML in one code block, then run/test instructions. Keep prose short given code size.

Now let me actually write the code carefully. I'll compose it section by section, keeping names consistent. Key global state:

```
let cv, gl, W_, H_;
let MAP; // current map object
let players=[], me=null, grenades=[], pickups=[], effects parts...
let G; // match state
let CFG; // settings
```

Define WEAPONS:

```js
const WPN = {
 knife:{n:'匕首',slot:3,dmg:40,rpm:90,mag:0,res:0,rld:0,price:0,rw:1500? no reward 1500 CS knife reward 1500. ok rw:1500,spd:1.05,auto:false,spread:0,kick:0,pellets:0,range:2.2},
 usp:{n:'USP-S',slot:2,dmg:33,rpm:352,mag:12,res:24,rld:2.2,price:200,rw:300,spd:1,auto:false,spread:.011,kick:.012,hsf:4},
 glock:{n:'Glock-18',slot:2,dmg:28,rpm:400,mag:20,res:80,rld:2.3,price:200,rw:300,spd:1,auto:false,spread:.012,kick:.011,hsf:4},
 deagle:{n:'Desert Eagle',slot:2,dmg:53,rpm:267,mag:7,res:35,rld:2.2,price:700,rw:300,spd:.95,auto:false,spread:.018,kick:.03,hsf:4},
 mp5:{n:'MP5-SD',slot:1,dmg:26,rpm:750,mag:30,res:120,rld:2.4,price:1500,rw:600,spd:1,auto:true,spread:.013,kick:.008,hsf:4},
 galil:{n:'Galil-AR',slot:1,dmg:30,rpm:666,mag:35,res:90,rld:2.6,price:1800,rw:300,spd:.95,auto:true,spread:.011,kick:.014,hsf:4},
 famas:{n:'FAMAS',slot:1,dmg:28,rpm:666,mag:25,res:90,rld:2.6,price:2050,rw:300,spd:.95,auto:true,spread:.010,kick:.013,hsf:4},
 ak47:{n:'AK-47',slot:1,dmg:36,rpm:600,mag:30,res:90,rld:2.5,price:2700,rw:300,spd:.9,auto:true,spread:.009,kick:.017,hsf:4,team:'T'},
 m4:{n:'M4A4',slot:1,dmg:33,rpm:666,mag:30,res:90,rld:2.8,price:3100,rw:300,spd:.92,auto:true,spread:.0085,kick:.015,hsf:4,team:'CT'},
 ssg:{n:'SSG-08',slot:1,dmg:88,rpm:45,mag:10,res:90,rld:3.4,price:1700,rw:300,spd:.95,auto:false,spread:.05,kick:.05,hsf:4,scope:45,mag?},
 awp:{n:'AWP',slot:1,dmg:115,rpm:41,mag:10,res:30,rld:3.6,price:4750,rw:100,spd:.8,auto:false,spread:.06,kick:.07,hsf:4,scope:22},
 nova:{n:'Nova',slot:1,dmg:22,rpm:55,mag:8,res:32,rld:.45(shell),price:1050,rw:900,spd:.95,auto:false,spread:.045,kick:.04,pellets:8,hsf:3? use 2},
};
```
Sniper scoped spread: when ads>0.9 && speed<1: spread=0.001.

Grenade defs: {he:{price:300}, flash:200, smoke:300, molotov:400}.

Kill reward uses weapon rw.

Economy constants: START 800, WIN 3250, LOSS_BASE 1400, LOSS_STEP 500 cap 3400, PLANT_BONUS 800 (team on loss? apply: on plant: planter +300; if T lose after plant: each T +800), DEFUSE_BONUS 300, MAX 16000.

Bot difficulty table:
```
DIFF=[
 {n:'新兵',react:.55,sigma:.028,turn:3.2,gren:.15,kit? ,burst:[2,4]},
 {n:'老兵',react:.38,sigma:.02,turn:4.5,gren:.35,burst:[3,5]},
 {n:'精英',react:.24,sigma:.013,turn:6.5,gren:.6,burst:[3,6]},
 {n:'宗师',react:.14,sigma:.008,turn:9,gren:.85,burst:[4,7]},
]
turn rad/s.
```

Bot think interval 0.1s.

Now player movement constants: GRAV 20, JUMP 8.0, STEP 0.45, R 0.35, EYE 1.62, EYEC 1.15, HEIGHT 1.8.

Speed base 5.1 * weapon spd; walk 2.5; crouch 2.1; sprint 6.6 (only rifle/pistol? all). Sprint penalty: spread ×3, loud steps.

Acceleration: ground lerp vel toward wish with factor min(1, dt*10); air: add wish*2*dt capped.

Friction: on ground and no input: vel *= max(0,1-dt*8).

Now write collision resolve:

```js
function collide(p){
 // p.pos feet center
 for(const s of solidsIter()){ // solids + closed doors
   if(s.b[1] <= p.pos[1]+STEP || s.a[1] >= p.pos[1]+1.75) continue;
   // circle vs rect in XZ
   const cx=clamp(p.pos[0],s.a[0],s.b[0]), cz=clamp(p.pos[2],s.a[2],s.b[2]);
   let dx=p.pos[0]-cx, dz=p.pos[2]-cz; let d2=dx*dx+dz*dz;
   if(d2 < R*R){
     if(d2>1e-6){const d=Math.sqrt(d2); p.pos[0]=cx+dx/d*R; p.pos[2]=cz+dz/d*R;}
     else { // center inside rect: push along smallest axis penetration
       ...compute overlaps, push
     }
   }
 }
}
```
Called after X and Z integration (or once per frame with both combined; single call works reasonably).

Doors: door object {a,b (closed), open:[dx,dz], t, closed}. Current AABB = closed shifted by open*t. Collision uses current AABB if t<0.95.

Ground:
```js
function groundAt(x,z,feet){
 let g=0;
 for(s of solids){ if(x>=s.a[0]&&x<=s.b[0]&&z>=s.a[2]&&z<=s.b[2]){ const t=s.b[1]; if(t<=feet+STEP&&t>g)g=t; } }
 for(r of MAP.ramps){ if inside: h=lerp; if(h<=feet+STEP&&h>g)g=h; }
 return g;
}
```
Include closed doors as solids (they're in solids list with flag). Doors mostly full height.

Vertical update:
```
p.vel[1]-=GRAV*dt; p.pos[1]+=p.vel[1]*dt;
const g=groundAt(...,p.pos[1]);
if(p.pos[1]<=g && p.vel[1]<=0? ) { if(p.pos[1]<g){p.pos[1]=g; p.vel[1]=0;} p.grounded=true } else p.grounded=false;
```
Hmm if falling and pass through g: snap. If standing: pos==g, each frame gravity dips slightly then snap ✓. Walking off edge: g drops → fall ✓. Walking up step/ramp: g rises ≤ STEP above feet → snap up ✓.

Ceiling: skip (rare); jump into tunnel ceiling: add ceiling check: if head hits solid bottom → vel1=0: check solids where x,z inside and a[1] between head and head+? Minor; add simple: if p.pos[1]+1.8 > lowest ceiling above → clamp. Compute ceilAt similar min a[1] where a[1]>=feet+1.2. OK add.

Now grenade physics similar with radius .08, bounce: on collision, find axis with max penetration, reflect vel component *0.4, tangential *0.7.

Ray-AABB standard:

```js
function rayBox(o,d,b){
 let t1=-1e9,t2=1e9;
 for(i=0..2){ const inv=1/d[i]; let ta=(b.a? use arrays)... }
 return t within [0, tmax]
}
```
Solids stored as arrays a=[x,y,z], b=[x,y,z].

World raycast:
```js
function castWorld(o,d,max){ let bt=max,hit=null; for s: t=rayBox; if(t<bt){bt=t;hit=s;} return {t:bt,s:hit, p:o+d*t} }
```
Normal computation: derive from which slab was last — implement rayBox to return t and axis/sign.

Full raycast including players:
```js
function castAll(o,d,max,shooter){
 let best=castWorld(o,d,max);
 for p of players: if p!==shooter && p.alive && p.team!==shooter.team? (FF off: skip teammates) {
   head sphere: c=p.pos+[0,1.58,0], r=.17: ray-sphere t
   body: segment-segment distance from ray to vertical segment (p.pos+[0,0.15,0])-(p.pos+[0,1.45,0]) with radius .38: approximate with closest approach of ray to segment: implement segSegDist(ray o→o+d*tmax, seg). t approximation = project.
   Simplify: two spheres: chest c=p.pos+[0,1.1,0] r=.42, head r=.17, legs c=p.pos+[0,0.4,0] r=.34. Check each ray-sphere; head priority.
 }
 return {t, p?, head?, point}
}
```
Good enough hit detection.

Ray-sphere:
```js
function raySph(o,d,c,r){ const oc=[o-c]; b=dot(oc,d); cc=dot(oc,oc)-r*r; h=b*b-cc; if(h<0)return Inf; t=-b-sqrt(h); return t>0?t:Inf }
```

Damage calc:
```
base = def.dmg * (pellets each)
falloff: f = clamp(1 - dist/def.rangeFalloff?, ...) add rng falloff per weapon: rifle effective 40m: mult = clamp(1 - (d-15)/60, .5,1)? Keep: mult = 1/(1+d*0.012).
head: × hsf (4), armor: if hp armor>0 and (not head or helmet): hpDmg = dmg*0.55? CS ratio ~ armor absorbs half. Use: absorbed=dmg*0.5; armor-=absorbed*0.5? Keep: hpDmg = dmg*0.5; armor -= dmg*0.35; if armor<0 → add overflow to hp.
```
Also friendly fire off.

Now bot LOS: castWorld only + smoke check.

Smoke check:
```js
function smokeBlocked(a,b){ for sm of smokes: segPointDist(a,b,sm.p) < 2.4 → true }
```

Fire patch damage and Molotov.

Now effects arrays:
```
parts=[] // {p,v,t,life,sz,col,grav,add,tile}
tracers=[] // {a,b,t}
holes=[] // {p,n,t}
shells via parts with grav and bounce? Simple parts.
smokes=[] // {p,t,puffs:[{o,r,s}]}
fires=[] // {p,t,r}
```

Render pass: build opaque dyn array: characters, doors, pickups, bomb, grenades, view model. Alpha array: smoke puffs, fire, particles (normal blend). Additive: muzzle flash, tracers, flash star? Tracers additive.

Draw order: static → opaque dyn → alpha (depth test on, mask off) → additive.

Sun disk: additive quad at cam + sunDir*300.

Now character draw detail:

```js
function drawHuman(p){
 const yaw=p.yaw, f=[sin? use forward from yaw only]
 base y=p.pos[1];
 walk=p.walkPh; crouch=p.crouch;
 if(!p.alive){ corpse: lying: push torso box rotated 90 (yaw), on ground, darkened tint; simple: 2 boxes lying. return }
 legSwing = sin(walk)*0.35*speedFactor
 boxes (local space, +z forward? define local +z = forward):
  legL: c=[-0.11,0.4+?, sin*...]: center (−0.11, 0.42, legSwing*0.3?) height .8: legs from 0..0.8: center y .4; offset z = sin(walk)*.22 (opposite phase)
  legR mirrored with -sin
  torso: [0, 1.12, 0] size .46,.6,.26
  vest: [0,1.15,.02]? skip
  armL/R: [±.26,1.25,.15] size .12,.12,.3? angled forward: just boxes forward: center [±.18,1.22,.28] size .1,.1,.34
  head: [0,1.62,0] .24,.26,.24 + skin
  gun: [0.12? ,1.35,.45] size .08,.12,.6 dark
 crouch: scale y positions by .8.
 team tint and push with yaw rotation around pos.
 name tag? skip.
}
```
walkPh += speed*dt*2.2.

Corpse: box at pos lying: torso .5x.3x1.5 along yaw? Push box with yaw and flat: center y .15, size .5,.3,1.6 rotated by yaw, plus head box offset. Dark red pool quad? Add dark red quad on ground. OK.

View model draw:

```js
function drawView(){
 if(!me.alive||me.scopeOn) return? sniper scoped hides gun.
 compute basis from me yaw/pitch + bob: bobX=sin(walkPh)*.02*spd, bobY=|cos|...
 root = eye + r*(.22+bobX) + u*(-.18+bobY + recoilBack) + f*(.42 - recoilBack*2)
 gun boxes along f: body size (.07,.11,.5) center root + f*.1; barrel f*.35 smaller; stock behind; mag below; if reloading: rotate down (offset u by -reload progress dip).
 muzzle pos = root + f*.55 → save for flash/tracer origin (me.muzzle).
}
```
Recoil visual: me.vmKick decays.

Now HUD crosshair gap = (spread*300 + 4)px.

Now let me write round/match code:

```js
G = {phase:'freeze', t:CFG? freeze 6, roundT:115, round:1, half:1, score:[0,0], lossStreak? per team [0,0], bomb:{...}, msgQ...}
update: 
 freeze: t-=dt → live (sound go). During freeze players can't move (skip movement input) but can look/buy.
 live: roundT-=dt; if ≤0 && bomb not planted → endRound(CT,'time').
 planted: bomb.timer-=dt beep; ≤0 → explode → endRound(T,'boom').
 end: t-=dt → nextRound().
```
nextRound(): check win score ≥ 8 → match end; check halftime: roundsPlayed==8 && half==1 → halftime (swap). Else round++ startRound.

Halftime: swap each player's team? Rebuild: simplest: for each p: p.team = 3-p.team; reset money to 800; clear weapons to default; swap scores? Scores stay with team index (0 CT, 1 T) — team swap means player group swaps: score array indexed by team stays. Player's new team = opposite. me.team swaps too. Announce.

startRound(): assign spawns, reset hp/armor? Armor persists? CS armor doesn't persist. Reset armor to 0 (keep helmet flag? No, reset). Keep weapons if survived last round (p.survived). Bomb → random T. Reset grenades? CS keeps grenades if survived — keep. plantProg/defuseProg 0. Bot state reset, assign routes/spots.

Money award at endRound: winners each +3250; losers +lossBonus (streak). Reset streaks.

Also players keep money.

Bot buy at startRound freeze (immediate).

Now bot assignment:
```
For T bots: distribute routes: shuffle route list; each bot takes next route (weighted by site). Bomb carrier forced to a route.
For CT bots: assign spots cycling through sites [A,B,mid] based on count.
```
Bot objective state machine in think.

Bot movement execution:
```
function botMove(b,dt){
 if(b.blindT>0){ move slowly along last path; return }
 target = b.path[0]...
 if engaged with enemy: strafe: b.strafeDir flips every 0.7-1.4s; move = strafe*0.7 + maybe advance/retreat based on distance and weapon (knife? no).
 else follow path: dir to wp; if dist<0.8 → shift wp.
 set b.mf,b.ms (forward/strafe in own yaw frame) via desired move direction relative to b.yaw? Bots face movement direction when not engaging: lerp yaw toward move dir.
 stuck detect.
}
```
Also bot jump: if blocked and grounded occasionally? Skip.

Bot use doors: automatic (doors open on touch) ✓.

Bot plant: at plant spot: b.channel='plant' while stationary; interrupted on damage.
Bot defuse similar.

Bot grenade use:
- HE: on engage, if dist 6..30 && rnd<diff.gren*0.5 && cooldown → throw at enemy feet (arc).
- Flash: on flashSpots pass (route push) → throw.
- Smoke: on smokeSpots → throw.
- Molotov: on engage clustered enemies? Rare: if 2+ enemies within 4m → throw.
Track b.gcd.

Bot hearing: on shot event: for each bot of opposite team within 30m: if no visible enemy → b.investigate = shotPos (path there, one-time).

Now waypoint path: build adjacency from links with distances. Dijkstra per query (35 nodes trivial).

```js
function findPath(a,b){ standard with arrays }
```
Bot repath: b.path = findPath(nearestWp(from), targetWp) mapped to coords; prepend current pos.

nearestWp: min distance.

Also CT retake path to bomb pos: nearestWp(bomb) target.

Now smoke rendering and LOS.

Flash effect on local: me.flashT.

OK — also need "observation": on death: me.spec target index; camera copies target's eye and yaw/pitch. Click cycles. If no teammates alive → free overview camera at map center high looking down? Implement simple orbit: pos = map center + [0, 40, 30] looking at center. Fine.

Now let me write UI DOM and CSS. Keep it tight but polished: olive/dark military, stencil letters via letter-spacing, scanlines? Subtle.

Given the enormity, let me start writing the final code now. I'll write it continuously, carefully. (Special care: no stray Chinese quotes in code; consistent variable names.)

Also: `requestAnimationFrame` loop with dt clamp 0.05.

Pointer lock: canvas click → if in game and alive and not paused → cv.requestPointerLock().

Mouse handlers gated by document.pointerLockElement===cv.

Buy menu open releases pointer lock? Keep pointer lock and show menu with cursor? Need cursor for click: release lock while buy open; relock on close. Same for pause. Implement showBuy toggles.

Actually simpler: buy menu navigable by number keys? Provide both: click (unlock) — choose unlock approach: on openBuy: document.exitPointerLock(); on close: relock.

Edge: pointerlockchange triggers pause menu — need flag to distinguish buy-open unlock vs Esc. Use `uiLock` flag when intentionally unlocked (buy/pause menu/end).

Now let me write the code. Also include favicon? Skip.

Another consideration: performance of groundAt/collision iterating all solids per player per frame: 10 players × ~300 solids × few checks = fine.

Bullet raycast per pellet: 300 slab tests × 8 pellets × maybe 3 shots per frame = 7k tests — fine.

Bot LOS: think staggered.

Now let me write the code. I'll use `let`/`const` and arrow functions for compactness. I'll write about 2000+ lines. Let me go.

Actually one simplification for view model: draw gun as part of scene with same projection (FOV). Fine.

Scope overlay: DOM div with radial-gradient mask (two circles) + crosshair lines.

Also handle: weapon FOV zoom applies to projection.

Camera roll? Small roll on strafe? Skip.

Head bob: view model bob and slight camera y bob.

Let me write the final answer: brief intro (Chinese), the complete HTML in one code block, then run/test instructions. Keep prose short given code size.

Now let me actually write the code carefully. I'll compose it section by section, keeping names consistent. Key global state:

Given the extreme length, I'll allow some terse repetition. Let me go.

While writing, I'll keep all function names consistent. Let me define the final list of globals/functions to avoid mismatches:

Utils: clamp, lerp, rnd, ri(a,b) int, V3 helpers inline arrays, dist2d.
Audio: A = {ac, master, init(), shot(kind,vol,pan), step(vol,pan), explo(vol,pan), beep(f,vol), tink(), hit(), stinger(win), pop(), ring(), amb(theme)} and s3(pos, fn).
Tex: buildAtlas() → canvas; MATS list {t: tileIndex, rep}.
GL: initGL(), prog, buffers, drawArraysData(arr, mode).
Geo: pushBox(A, c, s, yaw, mat, tint), pushRamp(A, r), buildStatic(map), quad helpers.
Maps: MAPS = {dust2: {...builder}, ruin:..., depot:...}; function loadMap(id).
Nav: buildNav(map), findPath(map, a, b), nearestWp.
WPN data; GRE data.
Players: mkPlayer(team, isBot, name, diff).
Items: giveWeapon(p,id), curW(p) current weapon object {id, def, mag, res}, grenades count p.gren = {he,fl,sm,mo}; p.gsel selected type.
Combat: tryFire(p), doReload(p), startReload, throwNade(p), hurt(...), kill(...).
Grenades: updNades(dt), explodeHE, doFlash, doSmoke, doFire.
Bomb: updBomb(dt), plantTick, defuseTick.
Bots: botThink(b,dt), botMove integrated in updPlayer via p.isBot branch, botBuy(b), assignRound(b).
Round: startMatch(cfg), startRound(), endRound(winner,reason), nextStep(), halftime(), matchEnd().
Effects: addPart, addTracer, addHole, muzzle(p), updFx(dt).
Render: render(dt): buildDyn, draw.
UI: initUI(), updHUD(), radarDraw(), openBuy/close, buildBuy(), updScore, msg(text,sub), killfeed(a,b,w,hs), showEnd, showPause.
Input: key handlers, mouse.
Main loop.

Also door updates: updDoors(dt).

Also pickups: updPickups? Draw + pickup action.

Also fire damage in upd.

OK writing now. (Special care: no stray Chinese quotes in code; consistent variable names.)

Also: `requestAnimationFrame` loop with dt clamp 0.05.

Given the extreme length, I'll allow some terse repetition. Let me go.

One more note: I'll include the Dust2 waypoint heights implicitly (groundAt). Wp on platform fine.

Also bot path smoothing: skip.

Also ensure bots don't stack on same spot: add small personal offset.

Final: write the answer.

Additional check — bomb plant site volume test:
```js
function inSite(p){ for s of MAP.sites: if x within and z within → return s.id }
```

Bomb plant position clamped inside site.

Explosion at bomb: big: parts, sound, kill within radius 14 with falloff dmg 500 → all near die.

Also planted bomb drawn: box with blinking light (t).

Dropped bomb drawn: dark green box.

Bomb carrier HUD icon.

OK — write.

Given the enormity of what follows, I'll write with high attention to matching brackets. Let me draft inline in final form.

(Composing... I'll write the entire file.)

Some concrete numeric decisions while writing:

- Canvas fullscreen; DPR cap 1.5.
- FOV 75.
- Sens default 1.0 → rad = dx*0.0022*sens.
- Player radius .35.

Bot count: CFG.size = team size (1..5). Bots per team: size - (1 if player on that team).

Difficulty index 0..3.

Match: WIN_ROUNDS = 8, HALF_ROUNDS = 8.

Freeze 6s, round 115s, buy window freeze + 12s.

Bot think 0.1s, staggered with b.thinkT initial rnd.

OK final code. Also need `performance.now` based `now` seconds.

Also handle tab visibility pause? Skip.

Now write — final.

After code, provide brief run/test instructions.

Let me write Dust2 builder code concretely (final coordinates as planned):

```js
function buildDust2(){
 const B=new MB();
 const W=B.wall, BL=B.block, BX=B.box, ST=B.stairs, RM=B.ramp, DR=B.door;
 // base
 BL(-36,-24,36,58,-0.6,0,1,1);
 // perimeter
 W(-30,-22,-30,56,5,0); W(28,-22,28,56,5,0); W(-30,-22,28,-22,5,0); W(-30,56,28,56,5,0);
```
Wait wall() decides orientation by length; vertical wall x=-30 from z -22..56 ✓.

Hmm perimeter corners fine.

```
 // T spawn
 W(-12,54,16,54); // south
 W(-12,40,-12,44); W(-12,48,-12,54); // west with gap 44..48
 W(16,40,16,43); W(16,48,16,54); // east gap 43..48
 W(-12,40,-2,40); W(4,40,16,40); // north gap -2..4
 BL(-2.5,38.6,4.5,40,2.6,3.4,0); // lintel over mid entry? decorative arch over gap: y 2.6..3.4 spanning gap ✓ (block absolute y)
 // crates in T spawn
 BX(9,50,1.5,1.5,1.5,3); BX(-7,45,1.2,1.2,1.2,3); BX(-4,51,1.4,1.4,1.0,3);
 // long corridor and doors
 W(16,43,18,43); W(16,48,18,48); // corridor walls z 43..48 x16..18
 DR(19.2,45.5,'x',2.4,2.9,5); // long doors at x≈19 spanning z 43.1..47.9? door(x,z,axis...) define door(cx,cz, slideAxis, w,h): panels along z: total width 4.8 (z 43.1..47.9): panels each 2.3 with gap .2.
```
Define door(cx,cz,axis,len,h,mat): axis = 'x' means door faces x direction (panels span z). Panels: p1 center cz-len/4-... let me implement:
```js
door(cx,cz,ax,len,h=3,m=5){ // ax: 'x' → normal along x (panels along z), slide along z ±
 const g=0.24, pl=(len-g)/2, th=0.35;
 if(ax==='x'){ d1={a:[cx-th/2,0,cz-len/2], b:[cx+th/2,h,cz-g/2], ...slide:[0,0,-(pl)]}? slide p1 by -pl? p1 slides further -z into wall: offset -(pl*0.9). p2 +.
 } else panels along x, slide ±x.
}
```
Store doors: {a0,b0 (closed AABB), off:[dx,dz] open offset, t:0, m}. Current AABB = a0+off*t..b0+off*t. Two door entities per doorway.

Long doors: cx=19.2? Corridor x16..18 then long lane x18..24: doors at x=19.5 within lane start? Classic doors right after T turn. Place at x=19.5, z 43..48 len 5: panels z 43..45.38 and 45.62..48 ✓ slide ±2.2 z.

```
 // Long A walls
 W(18,8,18,43); W(24,8,24,44);
 // long→corner stairs (0→0.9)
 ST(18,9,24,11,0,0.9,3,'z-'); // stairs along z from z11 (low) to z9 (high)? define stairs(x1,z1,x2,z2,h0,h1,n, axis): create n steps with increasing height toward (z2?). Let me define stairsZ(x1,x2, zFrom,zTo, hFrom,hTo, n): steps evenly.
```
Simplify: write `stairs(x1,z1,x2,z2,n,h)` that creates n steps along dominant axis from height 0 to h ascending toward (z2?). Write two explicit helpers: stairsZ(x1,x2,z0,z1,h,n): for i: zi segment, height h*(i+1)/n ascending toward z1. Call carefully.

Long stairs: x 18..24, z 9..11, ascending north (z decreasing): at z 10.33..11 h .3, z 9.66..10.33 h .6, z 9..9.66 h .9. So stairsZ(18,24, 11,9, 0.9,3) ascending toward z=9 ✓.

```
 // corner platform pieces (h .9) around pit hole x20..24 z5.5..8.5
 BL(12,5,20,9,0,0.9,0,1); BL(24,5,25,9,0,0.9,0,1); BL(20,8.5,24,9,0,0.9,0,1); BL(20,5,24,5.5,0,0.9,0,1);
 // walls around corner: east x=25 z 4..11? 
 W(25,5,25,11); // east edge
 W(12,9,18,9)? no—long lane north end z=8..9 opens to stairs; walls: west x=12 z 5..9? corner west edge x=12 z5..9 vertical face is platform side; need wall to prevent falling from site side? Site platform x6..20 z-8..6 adjacent. Add wall segment z=5 x 25..28? and x=25 handled by perimeter x=28? Gap x25..28 z5..9: sealed by W(25,5,25,9)? pB covers x24..25 solid h0.9; area x25..28 z4..9: add wall x=25 z 5..11 ✓ (done), z=9 wall x 24..28? Area between long east wall x=24 (z8..44) and perimeter... z 9..11 x 24..28: stairs x18..24 z9..11; east of stairs x24..28: wall z=9 x 24..28 add? W(24,9,28,9) ✓. South: z=5 wall x 20..28? pD covers x20..24 z5..5.5 h.9; add W(20,5,28,5)? area z 4..5 x12..28 under corner platform: bounded by platform side faces (0.9 tall) — can player walk in strip z 4..5? Bounded south by wall z=5? Add W(12,5,28,5)? But site platform x6..20 z-8..6 occupies z≤6 → overlap area x12..20 z5..6 is site (1.2). Strip x20..28 z4..5: add wall z=5 x20..28 ✓ W(20,5,28,5). And x12..20 z5..6 covered by site ✓. East x=28 perimeter ✓. So pocket sealed ✓.
 // A site platform
 BL(6,-8,20,6,0,1.2,0,1);
 // site step from corner (z 5..6 overlap handles)
 // goose low wall
 BL(18.2,2.6,19,5.8,1.2,2.3,0);
 // site crates
 BX(11,-1,1.7,1.7,1.7,3); BX(13.6,2.2,1.1,1.1,1.1,3); BX(8,-6,1.2,1.2,1.2,3)? near ramp top.
 // A ramp
 RM(6,-12,10,-8,0,1.2,'z'); // h0 at z=-12 (0) → h1 at z=-8 (1.2)
 W(6,-12,6,-8); W(10,-12,10,-8);
 // pit rim? sides are slab sides ✓ add ladder? jump only.
 // mid walls
 W(-2,4,-2,40); W(4,4,4,18); W(4,24,4,40);
 // Xbox
 BX(2.2,9,1.8,1.8,1.7,3);
 // mid doors
 DR(0? cx=1? mid x -2..4 center x=1, z=4, axis 'z' (normal along z, panels along x), len 6: panels x -2..0.9 and 1.1..4 ✓
 // CT mid walls
 W(-2,-10,-2,4); W(4,-10,4,4);
 // CT spawn
 W(-8,-20,10,-20); // north
 W(-8,-20,-8,-18); W(-8,-14,-8,-10); // west gap -18..-14
 W(10,-20,10,-12); // east (ramp walls cover -12..-8)
 W(-8,-10,-2,-10); W(4,-10,6,-10); // south gap: -2..4 (mid) and 6..10 (ramp)
 BX(2,-17,1.3,1.3,1.2,3); BX(-4,-13,1.2,1.2,1.0,3);
 // B corridor CT side
 W(-14,-18,-8,-18); W(-14,-14,-8,-14); // corridor x -14..-8 z -18..-14
 DR(-14,-16,'x',4,3); // B doors at x=-14 spanning z -18..-14: panels z -18..-16.12, -15.88..-14 ✓ slide ±z
 // B corridor to site
 W(-20,-18,-14,-18); W(-20,-14,-14,-14);
 // B site walls
 W(-26,-12,-20,-12); W(-14,-12,-12,-12); // north gap -20..-14
 W(-26,-12,-26,4); // west
 W(-12,-12,-12,0); // east partial
 W(-26,4,-22,4); W(-16,4,-12,4); // south gap -22..-16? tunnel exit x -20..-16: gap x -22..-16? Set south wall segments x -26..-22 and x -16..-12, gap -22..-16 ✓ (tunnel corridor x -20..-16 walls extend z 4..26)
 // tunnel corridor walls
 W(-20,4,-20,26); W(-16,4,-16,26);
 // bend area z 24..28 x -22..-16 walls: 
 W(-22,24,-22,28)? South corridor x -22..-18 z 26..44: walls x=-22 z 26..44 and x=-18 z 28..44? Let me do: south segment walls: W(-22,28,-22,44); W(-18,28,-18,44); bend box area x -22..-16 z 24..28 bounded: north wall z=24 x -22..-20? and x -16..-16? Opening to north corridor x -20..-16 at z=24? North corridor z 4..26 → overlap z24..26 ✓ so bend connects. Bend south wall z=28 x -22..-18 opening? South corridor z 28..44 x -22..-18: opening at z=28 x -22..-18 ✓ no wall there; wall z=28 x -18..-16 ✓ W(-18,28,-16,28). Bend west wall x=-22 z 24..28 ✓ W(-22,24,-22,28). Bend east wall x=-16 z 24..28? North corridor wall x=-16 covers z 4..26; z 26..28: W(-16,26,-16,28) ✓. Bend north wall z=24 x -22..-20 ✓ W(-22,24,-20,24).
 // tunnel upper (T side) corridor
 W(-22,44,-12,44)? T spawn west gap z44..48: corridor x -22..-12 z 44..48: walls z=44 x -22..-12 ✓ and z=48 x -22..-12 ✓; connects to bend via south corridor x -22..-18 z 28..44 ✓ wait south corridor z 28..44 but corridor z 44..48: overlap at z=44 x -22..-18 ✓ opening (no wall at z=44 segment x -22..-18? Wall z=44 x -22..-12 blocks! Fix: wall z=44 segments x -18..-12 only ✓ W(-18,44,-12,44). And z=48 wall x -22..-12 ✓ W(-22,48,-12,48). West wall x=-22 z 28..48? Covers corridor west: W(-22,28,-22,48) ✓ (merge with above). East side x=-18 z 28..44 ✓ + z 44..48 east side opens to T spawn? Corridor x -22..-12 z44..48 east end x=-12 → T spawn wall x=-12 gap z44..48 ✓.
 // tunnel ceilings
 BL(-22,24,-16,48,2.5,3.0,0,0)? Cover bend + south + upper: x -22..-16 z 24..48 ceiling ✓ but south corridor x -22..-18 only: ceiling over x -22..-16 z28..44 covers extra x -18..-16 (over wall, fine).
 BL(-20,4,-16,24,2.5,3.0,0,0); // north corridor ceiling
 // B platform and cover
 BL(-22,-9,-15,-3,0,0.6,0,1); BL(-22,-3.8,-15,-3,0,0.3,0,1); // step
 BX(-18.5,-6,1.6,1.6,1.6,3); BX(-15.5,-8,1.2,1.2,1.2,3)? on platform y .6: BX with y param: box(x,z,w,d,h,m,y). BX(-18.5,-6,1.6,1.6,1.6,3,0.6); BX(-24,1,1.3,1.3,1.3,3); barrel: BX(-13,-10,0.7,0.7,1.1,4);
 // mid/CT filler walls
 W(10,-8,20,-8); W(20,-8,28,-8); // north A wall segments (gap x6..10 ramp) — wait site north edge z=-8 x6..20: wall above site? Wall z=-8 x 10..20 ✓ W(10,-8,20,-8) and x20..28 W(20,-8,28,-8) ✓ (ramp gap x6..10 ✓).
 W(6,-8,6,-12)? ramp west wall done.
 // short/catwalk
 BL(4,8,10,20,0,1.5,0,1); // catwalk slab
 ST stairs x4..8 z20..24 ascending east 0→1.5: stairsX(4,8,20,24,1.5,5): steps along x.
 BL(8,20,10,24,0,1.5,0,1); // landing
 W(10,8,10,24); // catwalk east wall
 // mid east wall gap z 18..24: wall x=4 z 4..18 and z 24..40 ✓ set earlier (W(4,4,4,18); W(4,24,4,40)) ✓ gap 18..24; stairs enter at z 20..24? Adjust gap to 20..24: change to W(4,4,4,20); W(4,24,4,40) → gap 20..24 ✓. Stairs x4..8 z20..24 ✓.
 // short → site extension
 BL(4,6,10,8,0,1.2,0,1);
 // catwalk west lower wall = mid wall x=4 z 8..20 ✓ exists (W(4,4,4,20) covers z 4..20 ✓).
 // area under catwalk x4..10 z8..20 sealed? Slab bottom at 0 → solid block (full 0..1.5) so no underpass ✓ (classic catwalk has no underpass anyway).
```
Wait BL fills 0..1.5 solid: catwalk is solid pedestal — fine visually (looks like raised walkway) ✓.

Similarly A site platform solid block ✓, corner platform solid ✓.

Ramps: RM wedge drawn + ground function.

Also add decorative arches at long corner? Add lintel over site entry from long: BL(12,5.6,18,6.4? no—keep clear.

Add barrels/boxes along long for visuals? Long is intentionally clean.

Add "window" box at CT mid? Skip.

Dust2 spawns: T: within x -8..12, z 44..52: generate 5 points: [(-6,50),(0,51),(6,50),(3,46),(-3,47)]? Ensure not inside crates: crates at (9,50), (-7,45), (-4,51): adjust: T spawns [(0,51),(5,52),(-4,52),(8,47)? (9,50) crate near... (7,52), (-8,50)? (-7,45) crate. Use [(0,51.5),(4,52),(-3,52),(7,52),(-7,51)]. CT spawns: [(0,-13),(-4,-16),(4,-16),(-6,-12)? ramp... (6,-13)? near ramp fine, (2,-18)]: [(0,-13),(-4,-17),(4,-17),(7,-14),(-6,-18)]? Crate at (2,-17) — move crate. Adjust CT crate to (-4,-13) and (5,-18). Spawns [(0,-14),(-4,-17),(4,-15),(7,-18),(-6,-18)]? Wall x=-8 gap z-18..-14 corridor... (-6,-18) fine.

Sites: A {x1:6,x2:20,z1:-8,z2:6}, B {x1:-26,x2:-12,z1:-12,z2:4}.

Buy zone = spawn rect expanded.

OK. Dust2 done (~90 solids + stairs).

Stairs helper:
```js
stairsX(x0,x1,z0,z1,h,n): for i<n: xa=x0+(x1-x0)*i/n, xb=xa+(x1-x0)/n; height h*(i+1)/n ascending toward x1: block(xa,z0,xb,z1,0,hh).
stairsZ similar ascending toward z1? Define ascending toward smaller end? Just pass explicit: stairs(x0,z0,x1,z1,h,n,axis): ascending from (x0/z0 side height h/n) to (x1 side h)? Implement: along axis, step i (0..n-1) from start side: height h*(i+1)/n. Caller passes start/end accordingly.
```
Long stairs: ascending toward site (north, z decreasing): start z=11 (low) end z=9 (high): stairsZ(x1=18,x2=24, zStart=11, zEnd=9, h=0.9, n=3).
Short stairs: ascending east: stairsX(xStart=4→xEnd=8, z 20..24, h=1.5, n=5).
A ramp uses RM wedge.
B platform step: manual block.
Depot A stairs: stairsZ(x 14..18, zStart=6, zEnd=4, h=0.9, n=3).
Ruin A stairs up: zStart=2? A site z -8..2 h1.2, stairs x12..16 z 2..4: stairsZ(12,16, zStart=4,zEnd=2, h=1.2,n=4). B: flat.

Ramp ground: ramps list entry {x1,z1,x2,z2,h0,h1}: height(x,z) = h0 + (h1-h0)*((z-z1)/(z2-z1)) (for z-axis ramp).

Dust2 ramp: {6,-12,10,-8, h0:0 at z=-12, h1:1.2 at z=-8}.

Wedge drawing for ramp: corners bottom y=0? Draw solid wedge from y=0 to slope: vertices: (x1,0,z1),(x2,0,z1),(x2,0,z2),(x1,0,z2),(x1,h0? slope top: at z1 height h0, at z2 height h1: top edge points (x1,h0,z1),(x2,h0,z1),(x2,h1,z2),(x1,h1,z2). Faces: top slope quad, 2 side triangles+rect, front/back rects, skip bottom. Implement generically.

OK now maps 2/3 builders similarly with less detail. I'll write them more quickly.

Ruin builder sketch:
```
Base slab; perimeter walls stone.
T spawn x -8..8 z 26..33 walls with gaps (west x=-8 z28..31; north z=26 x -2..2; east x=8 z 28..31). Pillars inside decor.
West connector x -16..-8 z 28..31 walls z=28? walls: W(-16,28,-8,28)? T spawn south wall z=? T spawn z 26..33: south wall z=33. Connector z 28..31: walls z=28 x -16..-8 and z=31 x -16..-8 ✓.
West lane x -16..-12 z 2..28 walls x=-16 z 2..31? and x=-12 z 2..28 ✓ (connector overlap corner).
East mirrored x 12..16.
Mid courtyard walls x=-6 z 6..26, x=6 z 6..26; T mid corridor x -2..2 z 26..28? T spawn north gap x -2..2 z=26 → mid z 6..26: walls x=-2 z 26..28? Connector: gap x -2..2 direct (mid wall starts z=6? Area x -6..-2 and 2..6 z 26..? sealed by T spawn north wall segments: z=26 wall x -8..-2 and x 2..8 ✓.
Pillars: BX(-3,20,1.4,1.4,3.2,2), BX(3,20,...), BX(-3,12,...), BX(3,12,...); fountain BX(0,16,2.6,2.6,0.9,9).
Mid north hall x -3..3 z -4..6 walls x=-3 z -4..6, x=3; gate doors at z=-4 (panels along x len 6, axis 'z').
CT hall corridor x -3..3 z -16..-4 walls x=±3 z -16..-4 ✓ opens to CT spawn gap z=-16 x -3..3.
CT spawn x -10..10 z -24..-16 walls; gaps: z=-16 x -3..3; x=-10 z -22..-19; x=10 z -22..-19.
CT west east-west corridor x -18..-10 z -22..-19 walls z=-22 x -18..-10 and z=-19 ✓; then corridor x -18..-14 z -19..-8 walls x=-18 z -19..-8 and x=-14 z -19..-8 ✓ opens to B north gap.
East mirrored x 14..18, corridor x 14..18 z -19..-8.
A site platform x 12..26 z -8..2 h1.2; north wall z=-8 x 12..14 and x 18..26; east wall x=26 z -8..2; west wall x=12 z -8..0? then open z 0..2 for stairs entry? Stairs at south: A south edge z=2: stairs x 12..16 z 2..4 ascending north 0→1.2; south wall z=2 x 16..26 ✓; west wall x=12 z -8..2 ✓; east lane opens at z=2..4 x 12..16 stairs ✓ (east lane x 12..16 z 2..28? lane x 12..16: walls x=12 z 4..28? and x=16 z 2..28; hmm stairs z 2..4 x12..16: lane z 4..28 between walls x=12,x=16 ✓.
Wait east lane earlier x 12..16 z 2..31 with connector x 8..16 z 28..31. Set lane walls x=12 z 4..31? and x=16 z 2..31? Connector z 28..31 x 8..16: walls z=28 x 8..12? (lane east wall x=16 covers east side; connector north wall z=28 x 8..16 with gap x 12..16 (lane continues) → wall z=28 x 8..12 ✓; connector south wall z=31 x 8..16 ✓; T spawn east wall x=8 gap z 28..31 ✓.
A site cover: altar BX(20,-4,2.2,2.2,1.1,9,1.2); crate BX(14,-1,1.5,1.5,1.5,3,1.2); pillars BX(13,-7,1.2,1.2,3,2,1.2)? on platform.
B site x -26..-12 z -8..2 floor 0: walls: north z=-8 x -26..-18 and x -14..-12 (gap -18..-14 CT entry ✓); west x=-26 z -8..2; east x=-12 z -8..2? south entry gap: south wall z=2 x -26..-16 and x -12? West lane stairs? B south entry: west lane x -16..-12 ends at z=2 → opening into B at z=2 x -16..-12 ✓ so south wall segments x -26..-16 ✓ and none east (x -12 wall covers). B cover: BX(-19,-2,1.8,1.8,1.8,3), BX(-15,-5,1.4,1.4,1.4,3), low wall BL(-23,-6,-21,-1,0,1.1,2)? etc.
Cross corridors: z 14..17 x -12..-6: walls z=14 x -12..-6 and z=17 ✓ with gaps in lane wall x=-12 z 14..17 ✓ (split wall) and mid wall x=-6 z 14..17 ✓ gap. East mirrored x 6..12.
```
Ruin waypoints as listed. Also add some trees? Jungle: green boxes "foliage"? Add decorative pillars and mossy crates. Fine.

Depot builder as planned:
```
Base slab dark asphalt mat 6? Use concrete dark.
Perimeter walls metal/concrete h5.
T spawn x -7..7 z 28..35 walls: south z=35, west x=-7, east x=7, north z=28 segments: x -7..-6? Gaps: west x=-7 z 30..33; east x=7 z 30..33; north z=28 x -2..2.
West connector x -14..-7 z 30..33 walls z=30 x -14..-7 and z=33 ✓.
West lane x -14..-10 z 2..30 walls x=-14 z 2..30? and x=-10 z 2..30 ✓ (connector corner overlap).
East connector x 7..14 z 30..33 walls z=30 x 7..14? gap? walls z=30 x 7..14 and z=33 x 7..14? But T east gap x=7 z30..33 ✓. East lane x 10..14 z 15..30 walls x=10 z 15..30, x=14 z 15..30 ✓; area x 7..10 z 15..30? Between connector and lane: wall x=10 covers; strip x 7..10 z 15..30 sealed? Bounded west by? Add wall x=7 z 15..30? Actually T east area: connector z30..33 x7..14; lane x10..14 z15..30; pocket x7..10 z15..30: bounded east by lane wall x=10 ✓, north z=15? open to rail area z<15 ✓ so pocket opens north → fine (extra yard space) ✓.
Central T corridor x -2..2 z 15..28 walls x=-2 z 15..28, x=2 ✓ opens to yard at z=15 ✓ and T gap z=28 ✓.
Yard x -6..6 z -8..15 walls x=-6 z -8..10, x=6 z -8..10 (open at z10 to rail? Rail z10..15: yard z10..15 merges with rail ✓ open), north wall z=-8 x -6..-3 and x 3..6 (gap x -3..3 → CT corridor).
CT corridor x -3..3 z -20..-8 walls x=±3 z -20..-8 ✓ opens to CT spawn gap z=-20 x -3..3.
CT spawn x -8..8 z -28..-20 walls; gaps z=-20 x -3..3; x=-8 z -26..-23; x=8 z -26..-23.
CT west east-west corridor x -22..-8 z -26..-23 walls z=-26 x -22..-8 and z=-23 ✓; south corridor x -26..-22 z -22..-12 walls x=-26 z -22..-12? and x=-22 ✓ opens to B north gap z=-12 x -26..-22 ✓. Wait B north wall gap: warehouse north wall z=-12 x -30..-26 and x -22..-14 ✓ gap x -26..-22 ✓.
East mirrored: east-west corridor x 8..22 z -26..-23; south corridor x 22..26 z -22..-10; A north wall z=-10 x 14..22 and x 26..30 gap x 22..26 ✓. A site platform x 14..30 z -10..4 h0.9: walls around edges? Platform block; walls: east x=30 z -10..4; south z=4 x 18..30 (gap x 14..18 stairs); west x=14 z -10..? open west side to south strip? Keep wall x=14 z -10..2? then open z 2..4 to stairs area? Stairs x 14..18 z 4..6 ascending north 0→0.9. South ground z>4.
Warehouse x -30..-14 z -12..2: walls: north z=-12 segments; south z=2 x -30..-28 and x -24..-20? Wait south gap x -28..-24 (B south corridor)? Earlier: B south corridor x -28..-24 z 2..10 walls x=-28 z 2..10 and x=-24 ✓; warehouse south wall z=2: segments x -30..-28 and x -24..-20? and x -16?? East gap z -4..2 at x=-14 (east wall gap). South wall: x -30..-28, x -24..-14 ✓ (gap -28..-24). East wall x=-14 z -12..-4 and z 2? Gap z -4..2 ✓ so x=-14 segments z -12..-4 and none z 2? South wall covers z=2 x -24..-14 ✓ corner. West wall x=-30 z -12..2 ✓.
Warehouse ceiling BL(-30,-12,-14,2,4,4.5,4,4) ✓ dark inside.
Warehouse cover: containers: BX(-24,-6,2.4,5,2.6,8,0,tint red), BX(-19,-1,2.4,5,2.6,8,0,tint blue), crate (-27,-9) 1.4, (-16,-9) 1.2.
Trains: BX(-19,12.75,10,2.5,3,4,0,tint)? box(x,z,w,d,h): west train center x=-19 (x -24..-14), z 12.75 (z 11.5..14): w=10,d=2.5 ✓ h3. East train center x=10 (x 4..16), z 12.75 ✓. Wheels skip. mat 4 tinted dark.
Rail ground strip: darker slab? Decorative: thin slab BL(-32,10,32,15,0,0.05,10)? Subtle; skip or add rails lines: skip.
A dock cover: container BX(22,-4,2.6,6,2.6,8,0.9,tint green) on platform, crate (17,-7) 1.5 on platform, crane leg BL(28,-8,29.5,-6.5,0.9,4,4)? Decorative.
South strip cover x -14..30 z 2..10: scattered small containers: BX(2,5,2.4,4,2.6,8,0,tint), BX(-8,7,2.4,4,2.6,8,0,tint2)? Ensure not blocking wp paths (wps at z 6..7 x -2,10): place at (-8,4)? and (6,3)? and (24,7)? Careful. Place: (-9,4), (5,3.2)? wp 29 (10,6) → 16 (16,7) path: avoid x 10..16 z 5..8: place (22,7). And 25 (-2,12.5) → 29 (10,6): crosses z 10..6 x -2..10: avoid boxes there: (-9,4) fine, (5,3.2) fine (z 3.2 south of path? Path at x5 z≈7.4 ✓ clear).
Floodlight poles: thin tall boxes + glow quad? Add 4 poles with additive glow quads at top (static additive? draw as emissive bright box). Simple: bright color box with tint (2,2,1.8) * no lighting? Shader multiplies by light... make them additive particles? Skip poles; just brighter ambient for Depot.
```
Depot night lighting: sun low intensity, ambient 0.5 bluish, fog dark.

OK. Now waypoint lists as planned.

Now strat data per map (routes/spots/smoke/flash/plant) as planned.

Now finally: write it all out. Given the message size, I'll write compact code. Let me finalize some remaining implementation details as I write:

- `MB` builder returns {S (solids), R ramps, D doors, ...} and map object assembles.
- Solids entries: {a:[x,y,z], b:[x,y,z], m, mt?} mt = top material override. When drawing box with different top mat: pushBox supports matTop param.
- For ground/collision, material irrelevant.

pushBox signature: pushBox(A, cx,cy,cz, sx,sy,sz, yaw, m, tint, mTop).

Tint default [1,1,1].

Static build loop: for each solid: center = (a+b)/2, size = b-a, pushBox with m and mTop.

Ramps drawn separately.

Doors drawn dynamically each frame: pushBox with current offset.

Now shader light: uniform uSun (vec3 normalized), uAmb, uDif. Depot: uAmb .5, uDif .35 bluish; Dust2 amb .45 dif .6 warm.

Fog per theme.

Character tint multiplied.

Also add simple directional "fake AO": darken faces by y? Skip.

Time to write. I'll now produce the final file text. (Careful with matching brackets.) I'll write about 1900 lines. Given message limits, I'll write efficiently but completely, resisting further feature additions.

Also: don't forget user asked for "sprint": implement double-tap. And "open door" automatic. And observation. And difficulty settings. And pause/restart/map selection/team/bot count/difficulty and end screen ✓.

Final answer structure:
1. One paragraph intro (Chinese).
2. Complete index.html code block.
3. Run instructions + test checklist + controls (concise).

Now write the code — with utmost care.

While writing, I'll count critical cross-references. Let me define everything before use within reasonable order (script at end of body, functions hoisted — use function declarations for hoisting safety).

Let me write it.
