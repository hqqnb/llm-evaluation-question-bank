The user wants a complete Counter-Strike style FPS game in the browser. This is a massive request. I need to build a single HTML file (or a small project) with Three.js-based FPS game including:

- 3 maps including Dust2 replica
- Full bomb defusal mode
- Economy system
- Buy menu
- Bots AI
- Weapons (pistols, SMG, rifles, sniper, shotgun, grenades, flash, smoke, molotov)
- HUD, radar, etc.
- Sounds (WebAudio synthesized)

Given the constraints, I'll create a single self-contained HTML file with Three.js from CDN, everything procedural. This will be a very large file. Let me write it carefully.

I should deliver:
- index.html (complete game)
- README with run instructions

Let me architect:

Core systems:
1. Three.js scene, PointerLockControls-like custom camera
2. Map builder: geometry from box primitives with textures (canvas-generated textures for sand walls etc.)
3. Collision: AABB-based player vs boxes collision, simple physics (gravity, jump, crouch)
4. Shooting: raycast against walls and enemies, spread, recoil, tracers, muzzle flash
5. Bots: waypoint navigation on a nav graph per map, simple states (patrol, attack, plant, defuse, hold)
6. Economy: money, buy menu (DOM UI), prices
7. Round flow: freeze time, round timer, bomb plant/defuse, win conditions, scoreboard
8. Sounds: WebAudio synthesized (gunshots, footsteps, explosion, etc.)
9. Grenades: physics projectiles, flash (screen white), smoke (particle sprites), molotov (fire area)
10. Radar: canvas minimap drawn from map layout
11. Spectate after death

Dust2 map: I need to lay out a reasonably faithful Dust2 layout. Coordinates plan (top-down, simplified but structurally correct):

Let me design Dust2 layout in a 2D grid (units meters, x east, z south):

Key locations (approx, compressed scale):
- T Spawn: bottom-left area (south-west)
- T ramp up to mid
- Mid: central north-south corridor
- Mid doors: mid center
- Xbox: near mid catwalk
- Catwalk (A小): from xbox to A site
- A site: top-right (north-east)
- Long (A大): from long doors (west of A) along east edge... wait in real Dust2, Long is on the east side going north-south, A site north-east corner.
- Long doors, long corner, blue box
- A ramp (A斜坡), goose, site boxes
- B site: top-left (north-west)
- B tunnels (B洞): from T spawn west/north up to B site
- B doors (B门), B window, B platform (B平台), back B
- CT spawn: middle-north between A and B
- Short/Cat: mid -> xbox -> catwalk -> A

I'll define coordinates so it plays right. Let me set map ~ 120x90 meters.

Layout plan (x: -60..60, z: -45..45; north = -z):

- T Spawn: around (-40, z +35) area, south side.
- T ramp/T apps up to mid bottom (mid entrance at (-15, 25)).
- Mid: corridor from (x -15, z 25) going north to (-15, -10), mid doors at z ~ 0. Top mid connects to Xbox at (-12, -15) and to catwalk going right (east) up to A short.
- Catwalk/Short (A小): from xbox (-10, -18) east-north to (25, -25), up stairs to A site plateau at (35, -30).
- A Site: north-east corner region x 30..55, z -40..-20. Goose corner, triple box, ramp down to CT... A ramp connecting site down to cross/long.
- Long (A大): east edge corridor x ~ 58..68? Hmm, in real dust2 long is the far east. Long doors at south end near (55, 10), corridor north to long corner (58, -20), then opens into A site east side, with blue box and cross.
- Pit (A坑): at long south, near long doors outside — pit at (50, 5) with ramp down.
- CT Spawn: center north around (5, -25).
- B site: north-west x -55..-35, z -40..-25. B doors connecting from CT side at (-30, -30), B window from upper tunnels, platform (B平台) at back.
- B tunnels (B洞): from T spawn area west at (-50, 25) going north through tunnel (-50, 0) to upper tunnel exit (-45, -18) into B site.
- Connector from mid to B? In real dust2, mid connects to B via... no, mid connects to CT spawn area through mid doors -> CT, and top mid -> xbox -> cat. B connection is via tunnels only (plus window). CT spawn connects to B via B doors and to A via... CT spawn is between them.

This is fine for a "highly faithful in structure" test map within reason.

For bot navigation, I'll build a waypoint graph per map with named waypoints and edges; bots path with simple A* or BFS and follow waypoints, steering around obstacles.

Given the enormity, I'll write everything in one HTML file, maybe ~4000-5000 lines. That's a lot but doable. Let me be pragmatic: produce clean, complete, working code. I need to be careful about correctness since I can't run it. I'll keep systems robust and simple.

Other two maps:
- "Inferno-like" town map (call it "Stone Town") — medium, banana, mid, apartments feel.
- "Office/Night raid" CQB map — tight corridors.

I'll generate them from a map definition format: list of boxes (x, z, w, d, h, y), textures, waypoints, spawn points, bomb sites (AABB zones), buy zones.

Let me define a MapDef structure:

```
{
  name, sky, fog, ambient,
  boxes: [ {x,y,z,w,h,d,tex, rotY?} ],
  spawns: {T: [...], CT: [...]},
  sites: {A: {minX,maxX,minZ,maxZ}, B: {...}},
  buyZones: {...},
  waypoints: [{id, x, z, y?, links:[ids], tags}],
  radar: drawing info or auto from boxes
}
```

Radar: draw boxes top-down onto canvas — auto from boxes with height threshold. Good.

Collision: player cylinder vs AABBs, resolve by axis separation. Boxes also used for raycast (shooting) — ray vs AABB slab test. Ground: boxes with top at various heights act as floors; simplest: player stands on max ground height under them: compute support height from boxes whose XZ range contains player and top <= playerY+step. Plus base ground plane y=0. Ramps: implement as special "ramp" boxes with slope: height varies linearly along an axis. Support height function handles ramps. Raycast vs ramps: approximate with multiple thin boxes (stair-step of boxes). Simpler: build ramps as series of thin step boxes (visually use a rotated box but collision as steps). Actually easier: collision and visuals both as step boxes but make steps small (0.25m). Hmm, visual quality: steps fine.

Simplify: ramps = rotated box visual; collision via support function that interpolates; raycast treats it as steps? For bullets through ramps, treat ramp as solid box of full height? That would block shots wrongly. Use per-ramp slab sampling: ray march small steps... I'll do: raycast checks AABBs (walls) only, and ramps are low enough that bullets rarely interact; for correctness, implement ramp raycast as ray vs inclined plane within box bounds. I'll implement a decent analytic ramp raycast.

Actually, let me reduce scope smartly: represent collision world as list of AABBs; ramps represented as 8-12 stacked thin slabs generated at map build time from a ramp spec {x,z,w,d,y0,y1,axis}. Visuals: also the slabs (they'll look like stairs) OR a smooth rotated plane. I'll make visuals a smooth rotated box and collisions slabs. Raycast uses the slab AABBs. Fine.

Movement physics:
- Gravity 20 m/s², jump velocity ~5.5, walk 4.2 m/s, run... CS style: default run 5.2 m/s (knife), shift walk 2.6, crouch 1.5? CS doesn't have sprint; user asked for 冲刺 — I'll make shift = 静步 (walk slow) and... they asked both 静步和冲刺. So: Ctrl/C crouch, Shift walk-slow (silent), and maybe sprint with... Actually in CS, Shift = walk. I'll do: default speed 4.8, Shift = slow walk (quiet), no sprint — but user explicitly asked sprint. I'll add sprint key... Let's map: Shift = 静步， Alt or double-W? Simplest: Shift = sprint? No — CS purists. I'll do: Shift = walk silent; CapsLock toggle? I'll add sprint on Shift and silent walk on... Use: Shift = 静步 (silent slow), and no sprint... requirement says both. Fine: Shift = sprint (faster, louder), Ctrl = crouch, Alt = silent walk? Hmm, but CS muscle memory: shift=walk. Compromise: Shift = 静步， and sprint with... honestly I'll do Shift=walk-silent and sprint not bound... no. Decision: Left Shift = 静步； Left Ctrl or C = 下蹲； and "冲刺" via holding... I'll make default speed the "run" speed and that's the sprint. Add explicit sprint on Shift: speed 6.5, louder footsteps; silent walk on... 'V'? Use: Shift=sprint, Ctrl=crouch, Alt(or toggle)=walk silent? Many browser FPS: Shift run. I'll do Shift = 静步 as CS does and 冲刺 = default movement (called run). Hmm the requirement is just a checklist; I'll implement: Shift = sprint (faster), Ctrl = crouch, and walking silently = crouch or walking... plus add 'C' toggles silent walk? Too complex. Final: Shift=静步（slow, silent), Ctrl=蹲， and normal = run. Plus sprint = Shift? I'll just implement both keys: Shift 静步， and "冲刺" via default speed being high and document it. Actually simplest honest: implement sprint on Shift and silent-walk on Alt. Browsers: Alt may trigger menu. Use 'C' for crouch too... I'll bind: Shift = 静步， Ctrl/C = 蹲， and sprint via double-tap W? Skip—I'll bind sprint to Shift and silent walk to Ctrl? But Ctrl+W closes tab! Dangerous in browsers. Ctrl+Crouch is standard in CS though; pointer lock usually captures ctrl fine but Ctrl+W still closes. OK final bindings:
- WASD move
- Shift (hold): 静步 slow walk
- Ctrl or C (hold): crouch
- Space: jump
- Default move = run （冲刺 feel). I'll also add F? no F is... R reload. Add "Alt"? skip.

Hmm requirement literally: "跳跃、下蹲、静步和冲刺". I'll add sprint on... Shift doubling: tap = walk? I'll do: Shift = sprint, Ctrl/C = crouch, CapsLock or 'T'? silent walk on... let me just use: Shift = 静步， and sprint = not needed... OK enough. Decision locked: Shift = sprint (6.8 m/s, louder), Ctrl/C = crouch, and 静步 = walking while... I'll add both: hold Shift = 静步？ Gah.

LOCKED: 
- Shift = 静步 (silent, slow) — CS-true
- Ctrl or C = 下蹲
- 冲刺： default run speed 5.4; sprint boost key = 'X'? no... I'll make sprint happen automatically when moving forward without shift (that's "running"). Document: 移动即为奔跑冲刺， Shift 静步. Good enough, and I'll note it. Actually to satisfy checklist I'll add a real sprint: hold Shift = sprint, silent walk = hold Ctrl? and crouch = C toggle... Ctrl+crouch... fine:

FINAL FINAL: 
- W/A/S/D move (default run = 冲刺速度 5.2)
- Left Shift: 静步 (2.4, silent)
- Left Ctrl or C: 下蹲 (toggle on C, hold Ctrl)
- Space: jump
Documented clearly. The "冲刺" is the default run. Done deliberating.

Weapons data: glock, usp, deagle, p250? Keep: 
- Pistols: Glock (T default), USP (CT default), Deagle, P250? maybe Five-Seven... keep Glock/USP/Deagle/P250.
- SMG: MP5, P90, UMP? Keep MP5, P90, MAC-10? MP5+P90.
- Rifles: AK47, M4A4, FAMAS? Galil? Keep AK, M4, AUG? Keep AK47, M4A4, Galil, FAMAS? AK/M4/FAMAS/Galil.
- Sniper: AWP, Scout? AWP + SSG08.
- Shotgun: Nova, XM1014? Nova + XM.
- Machine gun: Negev? skip... add M249? optional. Skip to save space.
- Grenades: HE, Flash, Smoke, Molotov/Incendiary.
- Equipment: Kevlar, Kevlar+Helmet, Defuse kit.

Prices CS-like. Damage models with headshot multipliers, armor penetration values, range falloff.

Gun view models: build from boxes/cylinders groups attached to camera; procedural muzzle flash (light + sprite), recoil kick on camera pitch, shell eject particle (small box with physics). Sounds: WebAudio synth — gunshot = noise burst + lowpass + click; different per weapon class.

Bots: 
- Nav graph waypoints; bot AI states: FREEZE → decide strategy (rush B / split A / default); roam to objective; engage on sight (raycast LOS + FOV); aim with error based on difficulty and reaction time; fire bursts; reload; plant at site; CT defuse; retake logic; use nades occasionally.
- Difficulty: easy/normal/hard → reaction ms, aim error, aggression, nade usage.

Round flow:
- Menu → map select, side select, bot count, difficulty → start.
- Freeze 3s (buy time 15s?), buy menu with B key during buy time in buy zone.
- Round timer 1:45. Bomb plant 3s, explode 40s, defuse 5s (kit) / 10s.
- Win conditions: elimination, explosion (T), defuse (CT), timeout (CT).
- Economy: loss streaks, plant bonus, kill rewards per weapon.
- Max rounds 15? MR12 (first to 13)? Use MR15 halves of 12? Keep simple: first to 13, halftime switch at 12? To keep simpler: first to 8 wins? I'll do standard-ish: 15 rounds max per half? Simplicity: no halftime side switch unless selected? I'll implement halves: 12 rounds each half, switch sides, first to 13, overtime none. Hmm, simpler: maxRounds 16, first to 9? Classic casual CS is MR15 first to 16... I'll do first to 9, switch sides after 8 rounds. Reasonable and shorter.

Actually let me do: half length 8, first to 9 wins (MR8). Short matches good for testing.

Spectate: after death, cycle through alive teammates' first-person view.

Kill feed UI, damage indicators, hitmarker, crosshair dynamic spread, health/armor bars, ammo, money, timer, scoreboard (Tab).

Sounds via WebAudio: procedurally generate gunshots (noise envelope), footsteps (short filtered noise), reload clicks, explosion (big noise + sub sine), beep for bomb, plant/defuse sounds, round start "go go go"? Skip voice.

Radar: canvas 2D, top-down boxes in tan, player arrow, teammates dots, bomb marker, enemies shown when visible? Show enemies when shooting/recently seen — keep simple: show enemies if visible to any teammate... just show when they fired recently or visible. Simplify: show all? No—show enemies only when they made noise recently. Fine.

Muzzle flash, tracers (line), impact particles + decals? Particles enough, plus bullet hole sprites maybe skip; add simple spark/dust puff.

Smoke grenade: spawn 6-10 big translucent sprites, persist 15s, blocks bot vision (check smoke sphere along LOS ray — bots: if LOS segment passes within smoke radius, no see). Player: visual occlusion.

Flash: on explode, compute for each viewer: distance + facing angle + LOS → white overlay fade. Bots: blinded timer.

Molotov: fire area, damage over time, orange particles, blocks path? Bots avoid fire cells (mark waypoint area as blocked temporarily — simply bots take damage and walk through... better: check fire positions in steering, steer away).

HE grenade: damage falloff, reduced by walls (LOS check), physics bounce.

Grenade physics: gravity, bounce with restitution vs AABBs and ground. Simple: integrate, on collision reflect velocity *0.4, friction.

Map definitions: I need to actually lay out Dust2 with many boxes. This is the most painstaking part. Let me define with a helper: wall(x, z, w, d, h, tex) with y0=0. I'll design Dust2 on graph paper mentally.

DUST2 LAYOUT (units meters). North = -Z (top). Map bounds x: -64..64, z: -48..48.

Let me place key areas:

- T Spawn: rect x[-58,-34], z[32,44]. Ground y=0.
- T ramp/plat to mid: corridor x[-30,-14], z[26,40] (T side open). "T plat".
- Mid entrance (bottom mid): x[-22,-10], z[14,26] corridor.
- Mid corridor: x[-22,-10], z[-6,14]. Mid doors at z=0..2: double doors — two boxes leaving a gap in middle? Real mid doors: doors on the wall crossing mid. Wall across mid at z=0 with door frames: boxes at x[-22,-18] and x[-12,-10], gap x[-18,-12]? Mid doors = two door panels with gap between: I'll make wall segments leaving a 2-door visual: solid boxes x[-22,-17] and x[-13,-10], plus two "door" thin boxes at x[-17,-15.5] and x[-11.5,-10]... simpler: wall with two 1.6m gaps? Real: double doors side by side with small gap. I'll do: full wall across mid at z=0 with a 3.2m wide opening x[-18.2,-15], filled with two door boxes each 1.5 wide leaving 0.2 gap — bullets blocked, vision blocked; CTs can... In real dust2 mid doors are closed (vision blocker) with gaps at edges you can peek through. I'll make doors solid boxes with small 0.3 gaps on sides for peeks.

- Top mid: x[-22,-10], z[-20,-6], opens to: xbox at (-16,-18) area leading catwalk right, and left to short?? no. Top mid goes: right → Xbox → catwalk (A小） → A site; straight → mid doors → CT spawn; left → suicide (drop to T ramp)? Suicide is between T spawn and mid... it's the pit area at mid entrance T side. I'll include "suicide" as an open pit area x[-32,-24], z[8,20] with drop.

- Xbox: box at (-14,-16) size 3x3 h1.2 you can jump on; catwalk starts east of xbox elevated y=1.2? In real dust2 catwalk is elevated walkway from xbox area rising to A short. Catwalk: x[-12,20], z[-20,-14], floor y≈1.0 ramping up to y=2? A short plateau y=2.6 near site. Hmm heights: A site ground y=0? In real dust2 A site is elevated plateau; catwalk leads onto it; long comes up via ramp; CT side via A ramp down.

Simplify heights: keep most ground y=0; A site plateau y=0 as well but approached by slopes (A ramp from CT side is slope up; long is flat-ish with slight rise; short/cat is elevated walkway y=1.2 dropping onto site via small jump). For gameplay fidelity without complexity: 
- Catwalk floor y=1.1 (elevated, with wall on CT side? no—the CT-side of cat has "short boost" and stairs). 
- A site plateau y=2.2? If site is elevated and CT must retake up ramps, that's true to dust2. Let me set: A site plateau y=2.0. Approaches: 
  - Long: long corridor floor rises from y=0 (pit area y=-1?) to y=2.0 at site via long ramp slope. Pit (A坑） at y=-1.2, ramp out.
  - Short/cat: catwalk y=1.2 → stairs/ramp up to y=2.0 at "short" then onto site.
  - CT: from CT spawn (y=0) up "A ramp"? no wait — A斜坡 is the ramp from A site down toward long/cross? In dust2, "A ramp/goose ramp" is on site west side going down to CT/cross area? Actually: A site plateau; "ramp" is where site descends toward CT spawn / crossroad ("goose" corner near ramp). CT retakes come up A ramp (from CT) or via long cross. So: CT spawn y=0 → A ramp slope up to site y=2.0 on site's south-west; cross (junction) between long and ramp.

- CT Spawn: rect x[-4,14], z[-28,-16], y=0. Connects: to mid via mid doors area (through "CT corridor" x[-10,-2], z[-16,-6]? that path passes under catwalk? In real dust2, path from mid doors to CT spawn passes... "CT mid" — after mid doors you're in "CT" area near Xbox with catwalk above-right. So CT area x[-16,0], z[-16,-4] y=0, with catwalk overhead? Catwalk at y=1.1 only 1.1 high—can't walk under. In real dust2 catwalk ("short") is much higher than CT mid (you boost onto it). Let me raise: catwalk/short floor y=2.6, same as A site (site y=2.6). Xbox (1.2 high box) used to boost... bots can't boost; give xbox area a ramp/stairs from CT side? In real dust2, from top mid you can jump on xbox then onto cat; CTs reach cat via stairs from... "short" has stairs down to CT side? Yes! Catwalk/short has stairs near site going down to "CT"/"short" junction... Actually short connects: top mid (via xbox) and A site; CTs access short by going through A site or... CTs boost too or go via mid/top mid. Real geometry: catwalk runs from top-mid/xbox up the slope to A site; below it is CT mid corridor from mid doors to CT spawn? No — mid doors exit to area with xbox on the right (elevated), CT spawn straight-right. Hmm.

Real Dust2 mid structure: T side mid goes north, passes mid doors, reaches "top mid" — an elevated-ish open area. From top mid: right = catwalk entrance (elevated walkway along the wall to A short), straight-ish right-down = ramp down to CT mid / "outside mid doors" near CT spawn, and Xbox sits between mid doors and catwalk — a box you jump on to get onto catwalk from CT side. Catwalk is high (≈2.5m) above CT mid. There are also "cat stairs"?? No stairs; you use xbox.

For bots, jumping on xbox is doable if I add jump impulse when waypoint flagged. I'll add "jump" flag on edges: bot jumps when traversing certain links. Manageable: waypoint link property {jump:true}. Bot physics same as player so jump works with flat landing.

Simplify geometry decision (keeping tactical structure):
- Ground level y=0 everywhere except: A site plateau y=2.4, catwalk y=2.4, B tunnels floor y=-0.0 (flat, simpler) with upper tunnel exit slightly raised? keep flat 0, B site y=0, B platform (B平台） elevated y=1.2 with ramp, long rises 0→2.4, pit y=-1.2 with ramps, CT spawn 0, T spawn 0.
- Xbox: solid box h=1.2 at catwalk base; waypoint edge top-mid→xbox-top flagged jump, xbox→cat flagged jump (2.4-1.2=1.2 jumpable? jump height with v=5.5, g=20: h=v²/2g=1.51. Yes 1.2 ok. From ground to xbox 1.2 ok.)
- CT mid: corridor from mid doors (z=0) area x[-10,-2] going north-east to CT spawn, passing under catwalk? Catwalk at y=2.4 with support pillars — walk under it, fine (2.4 clearance).

A site details (plateau y=2.4, region x[26,60], z[-44,-18]):
- Goose corner: north-west of site corner wall niche at (28,-40).
- Triple stack / site boxes near plant zone center (42,-34): stack boxes.
- Ramp (A斜坡）: site's south edge slope down from y=2.4 to y=0 toward south (x[30,44], z[-18,-8] slope down southward) leading to cross area y=0.
- Cross: junction x[44,58], z[-16,-6], connects CT spawn (west), long corner (east), A ramp (north).
- Long corridor: east edge x[56,64], z[-10,26], floor ramping: from cross y=0 at z=-10 rising to... wait long leads INTO A site: long corner (60,-12) → north to site edge (60,-20)? Site plateau covers z[-44,-18]; long corridor along east from long doors (58, 22) north to (58,-16) then turn west into site via "long corner/cross"? In real dust2: long doors (south) → long corridor north along far east → arrives at A site's east side at plateau level (corridor rises) → "long corner/blue box" → onto site, with "cross" being the gap/corner connecting long to CT side. And pit is OUTSIDE long doors on the west side of the corridor? Pit is before/at long doors — a dug-out with ramp, used to peek long. Layout: long corridor floor starts y=0 at doors? and rises to 2.4 near site. Pit at (48, 16) y=-1.2 beside the corridor entrance.
- Long doors: double door structure at (56..62, z 20) — two door frames, gaps between: wall segment with two 1.8m openings (passable) — real long doors are open doorways (two arches). Make: pillars + two arch openings, passable.
- Blue box near long corner (58,-16) h1.1.

B site (y=0, region x[-60,-34], z[-44,-22]):
- B doors (B门）: on east wall of B site connecting to CT/upper-mid area: wall at x=-34 with two door boxes (like mid doors, closed panels with side gaps) around z=-30. Real B doors: double doors usually closed, at B site east connecting to "window/counter" room? B site connects to CT spawn via "B doors" corridor. 
- B window: opening in wall north of B doors at z=-38, x=-34, a window (y 1.0..2.2) you can jump through, connecting CT-side corridor to B site.
- B platform (B平台）: elevated platform at back/north-west of site y=1.2, x[-58,-46], z[-42,-36], with ramp.
- Site boxes: big box stack center (e.g., (-46,-30) 2x2x1.2), car? (dust2 B has car) add car box.
- Upper tunnel exit (B洞出口）: opening in south wall of B site at (-40, z=-22)... tunnels approach from south-east. Upper tunnels floor slightly above lower; exit into B site through arch at x[-44,-38]? Let me route tunnels: T spawn → west → lower tunnels (x[-60,-50], z[10,30]) → north through corridor to upper tunnels (x[-52,-42], z[-18,10]) → exit north into B site south wall opening at (-46,-22). Also upper tunnel connects mid? No (that's dust1). OK.
- B back site: behind platform for retake.

CT spawn connects west to B via corridor x[-34,-10], z[-36,-26] ("CT to B" corridor) with the B doors and window on B site's east wall. Also CT spawn connects east to A via cross area x[14,44]... make corridor x[10,30], z[-14,-6]? CT spawn at x[-4,14], z[-28,-16]. East of CT spawn: corridor to cross x[14,46], z[-14,-4], then cross splits north (A ramp) and east (long corner).

Mid: bottom mid entrance from T ramp; mid corridor x[-22,-10]; top mid x[-22,-8], z[-20,-6]; xbox at (-12,-14); catwalk from (-8,-16) east x[-8,24], z[-20,-14] floor y=2.4 sloping slightly? keep 2.4 flat, meeting A site plateau west edge at x=26 with an opening ("short" enters site at (26,-17) region... site region z[-44,-18], so cat enters at site's south-west corner via an arch at (26,-16)).

A小 = catwalk/short. ✔.

Sniping lines: T spawn → mid doors see CT? mid doors gap sightline down mid to CT spawn ✔. Long sightline from pit/long doors to site ✔. B site from window/doors ✔.

T spawn to B tunnels: corridor west of T spawn x[-58,-50], z[24,34] connecting lower tunnels.

Also "suicide": pit beside mid entrance x[-30,-22], z[12,24], y=-1.2? optional; include small for flavor: a drop area with ramp back.

Waypoints: I'll place ~40 waypoints for Dust2 with links. Bots path via BFS. I'll also tag site waypoints, camp spots (defense angles: bot faces specified direction when holding).

This is getting huge but let's commit. Same structure for other two maps but I'll design them quicker:

Map 2 "Stonehaven" (Inferno-ish): 
- T spawn south, mid west→arch→A site north-west, "banana" east curving to B site north-east, CT spawn north center, apartments. Keep ~120x90.

Map 3 "Terminal/Office" CQB indoor: tight corridors, two sites, low ceilings, lots of corners, short engagement distances. Smaller 70x50.

Given the code volume, I'll write a compact but complete engine. Let me now write the code.

Structure of the single file:

1. HTML/CSS: menus (main menu, map select thumbnails (canvas-drawn?), settings), HUD elements, buy menu, scoreboard, killfeed, crosshair, hitmarker, damage overlay, flash overlay, scope overlay, spectator bar, round banner.
2. JS:
   - Audio engine (synth)
   - Input
   - Map definitions (3)
   - Map builder (meshes from boxes with canvas textures, colliders, ramps→slabs, nav graph)
   - Player controller + physics
   - Weapon defs + viewmodel builder (box-built guns), shooting, recoil, tracers, shells, decals-ish particles
   - Grenades (he/flash/smoke/molly) physics + effects
   - Bot AI (movement along nav, combat, objectives, planting/defusing, difficulty)
   - Round manager + economy + bomb logic
   - HUD/radar/scoreboard updates
   - Spectate
   - Main loop

Because of size, I must be efficient but complete. Let me estimate ~4500 lines. That's fine—I'll write it.

Key implementation details:

Physics/collision:
```
colliders = [{min:Vector3-ish, max}] as {x0,y0,z0,x1,y1,z1}
moveActor(a, dt): 
  a.vel.y -= g*dt
  move x, resolve; move z, resolve; move y, resolve (ground check)
```
AABB resolve for cylinder approximated as AABB (radius r, height h, crouch height). Step-up: if horizontal collision and obstacle top - feet <= 0.55 and standing possible, step up. That handles stairs/slabs (use 0.3 slabs so step works).

Ramps: implement support-based: I'll just build ramps as slab steps (0.25 h each) so step-up code handles them, visual smooth ramp mesh separate (non-colliding). Raycast for bullets: test against collider AABBs (includes slabs). Slight inaccuracy acceptable.

Ray vs AABB slab method with t near.

LOS check for bots: ray vs colliders (skip slabs below? include all; fine).

Bots movement: 
- state: current path (list of waypoint positions), target waypoint index. Move toward next point; if within 0.6, advance. Jump flag on edge → if close and needs jump, set jump.
- Stuck detection: if no progress 1.5s, repath or jump.
- Combat: scan enemies: FOV ~160? Reaction: if visible & LOS, after reactionTime (difficulty), start firing; aim = target pos + error*difficultyFactor decreasing while aiming; burst fire per weapon; strafe sometimes; crouch when holding angle.
- Objectives: T with bomb: after reaching site, plant (channel 3s). CT: if bomb planted, go to site, defuse channel. Guard: pick camp waypoint near site, face angle.
- Nade usage: hard bots throw HE toward known enemy position occasionally; smoke at choke when rushing; simple.

Economy: standard: start 800, win 3250 (defuse/explode 3500), loss 1400+500*streak(max3400?), kill rewards (awp 100, smg 600 except p90 300, shotgun 900, knife 1500, default 300). Plant bonus +800 for T team even on loss; planter +300. Max 16000.

Buy menu: DOM overlay, categories, click to buy, keys 1-9. Buy time 20s at round start (freeze 3s), only in buy zone.

Bomb: T player gets bomb (if player is CT, a bot gets it; if player is T, player gets it — give to player for fun; bot takes if player drops? keep: player T gets bomb). Plant: hold E in site 3s → bomb entity with beeping accelerating; 40s explode; CT defuse hold E near bomb 10s/5s kit. HUD progress bar.

Round end: banner, money award, 5s intermission, respawn, side switch at half.

Weapons table:

```
WEAPONS = {
 glock: {name:'Glock-18', type:'pistol', price:400? actually 200, dmg:30, hs:x4, rpm:400, mag:20, reserve:120, spread, moveAcc, armorPen:0.47? ...}
}
```
I'll use approximate CS values:
- Glock: 200, dmg 28, armorPen 0.47, rpm 400, mag 20/120
- USP: 200, dmg 34, pen 0.5, rpm 352, mag 12/24... use 12/100 for fun? CS: 12/24. Keep CS.
- P250: 300, dmg 38, pen 0.64, rpm 400, 13/26
- Deagle: 700, dmg 53, pen 0.932, rpm 267, 7/35
- MP5: 1500, dmg 26, pen 0.625, rpm 750, 30/120
- P90: 2350, dmg 25, pen 0.69, rpm 857, 50/100
- FAMAS: 2050, dmg 30, pen 0.7, rpm 666, 25/90
- Galil: 1800, dmg 30, pen 0.775, rpm 666, 35/90
- AK47: 2700, dmg 36, pen 0.775, rpm 600, 30/90
- M4A4: 3100, dmg 33, pen 0.7, rpm 666, 30/90
- AUG: 3300 scope... skip aug.
- SSG08: 1700, dmg 88, pen 0.85, rpm 41 (bolt), 10/90, scope
- AWP: 4750, dmg 115, pen 0.975, rpm 41, 5/30, scope
- Nova: 1050, 8 pellets x 9? dmg 26/pellet... pellets 8, dmg 9? CS nova: 26 dmg x9 pellets? whatever, 8 pellets * 12, rpm 68
- XM1014: 2000, 7 pellets*12 auto, rpm 171
- Knife: dmg 40/90 backstab.
- HE: dmg up to 98; Flash; Smoke; Molotov: 300 (T) / incendiary.

Headshot multiplier 4x (shotguns 2.5?), armor reduces dmg by pen factor: dmg_armored = dmg * pen; headshot with helmet similarly.

Range falloff: dmg * (0.85)^(dist/10) or CS-like per-weapon falloff: dmg *= pow(falloff, dist/500)... I'll use: dmg *= max(0.6, 1 - dist*0.004*falloffFactor). Simple.

Spread/recoil: each shot adds to bloom; inaccurate while moving; crosshair gap visual; camera punch (pitch kick) per shot with recovery.

Viewmodels: procedural box models per weapon class positioned bottom-right; ADS for scope = overlay + fov zoom + hide viewmodel. Muzzle flash: point light + plane sprite briefly. Shell: small yellow boxes ejected with velocity.

Hit effects: blood puff red particles; wall dust + spark; hitmarker sound + X; damage numbers? maybe killfeed.

Footsteps: timed noise clicks, volume by speed; bots' footsteps audible by distance.

Radar: canvas, draw colliders top-down (only tall walls as solid), player cone, teammates green/blue dots, bomb, planted bomb flashing, enemies red when recently noisy/visible. Rotate? Keep north-up fixed.

OK, also pause menu (Esc → pointer unlock shows pause overlay), resume, restart, quit to menu.

Map select UI: three cards with generated preview (draw radar of each map on small canvas!). Nice touch: render top-down preview per map for selection card.

Team/side select: T or CT. Bot count: per-team bots 1-5 (default 5v5 incl. player? choose total team size 5: player + 4 bots vs 5 bots; slider bots 1..9 total?). I'll do: enemies & teammates counts: "每队人数 1-5" meaning team size including player.

Difficulty: 简单/普通/困难.

Now write the maps concretely. I'll define helper functions inside map defs: 

```
function B(x,z,w,d,h,o={}) => collider {x0:x-w/2,... y0:o.y||0, tex:o.t||'wall', mesh:true}
```
Coordinates: I'll use center-based boxes: box(cx, cz, w, d, h, {y, tex, ry}).

Ramp helper: ramp(cx,cz,w,d, y0,y1, axis='z', dir) → generates steps + smooth visual (rotated box). I'll implement ramp as function generating N=Math.ceil(dh/0.25) slab colliders and a rotated box visual mesh.

Let me now sketch Dust2 coordinates carefully (center coords, sizes). Ground plane 128x96.

Perimeter walls: 4 big walls h=6 around map (so nades stay in).

I'll enumerate regions:

T SPAWN area: open yard x[-58,-34], z[32,44]:
- walls around its west/south partly perimeter.
- exit north to T ramp area through gap at x[-44,-30]? T spawn north wall at z=32 with opening x[-44,-34].

T ramp ("T plat" path to mid): open area x[-34,-10], z[24,40], with boxes as cover (T ramp boxes). Connects west to T spawn (gap), north to bottom-mid corridor.

Bottom mid entrance: corridor x[-22,-10], z[12,26]: walls both sides (west wall x=-22 from z12..26, east wall x=-10). North opening into mid.

Suicide: pit west of mid entrance: x[-34,-24], z[10,24], floor y=-1.3, ramp from T ramp side down at its south, and a wall on north side (you can watch mid from suicide). Include ramp (z direction) at z[20,24] rising northward to 0? Simplify: suicide pit with stairs back up to T ramp on east side. Eh—keep: pit floor -1.3, ramp on south edge back to y0.

Mid corridor: x[-22,-10], z[-8,14]. West wall x=-22 (z-8..14), east wall x=-10. 

Mid doors at z=-2..0 across mid: boxes: left door x[-20.5,-16.5]? Make: door frame wall segments: box x[-22,-18] (w4), doors: box1 x[-18,-14.6] thin z, box2 x[-13.4,-10], gap between box1/box2 = x[-14.6,-13.4] 1.2m?? Real mid doors: two doors, small gaps at frame edges. Doors must block vision. So: door A: x[-18,-14.8], door B: x[-13.2,-10], center gap 1.6m x[-14.8,-13.2] — that gap lets people pass! Mid doors are closed; you go around through... you CAN'T pass mid doors in dust2 (they're closed; small gap to squeeze? no). In real dust2, mid doors are passable? NO — mid doors are closed doors; Ts cross via top mid→cat or window... Wait, actually in CS:GO Dust2 mid doors: there are double doors, CLOSED, but there's a gap on each side? People shoot through the gap... The doors have a gap between them? The famous "mid doors" wallbang/peek is through the gap between the two doors (a thin vertical slit) and you cannot pass through; crossing mid T→CT side happens through top mid → xbox area?? No! Top mid IS on T side of the doors. To reach CT spawn from mid you go cat→A or tunnels→B. Hmm, but CTs cross mid doors how? CTs don't; CT spawn is on CT side of doors. The doors separate T-mid from CT area. So doors = solid wall with thin peek gaps. 

But wait: in CS:GO the mid doors actually have a passable gap? Let me recall: "Suicide" is where Ts watch mid; CTs sometimes push "through mid doors"? You cannot walk through mid doors. Correct: impassable, with a narrow gap between the two doors for vision (AWP battles). I'll make two door boxes with 0.35m gap between (bullets pass, bodies don't — min body width ~0.7). 

Top mid: x[-22,-6], z[-22,-8]. North wall at z=-22 with: opening to tunnels? no. Top mid connects: east to catwalk base/xbox area x[-8,4]? Hmm wait catwalk should run along here. Let me re-place: catwalk along z[-18,-12], from x[-6] to x[24], elevated y=2.4. Xbox at (-4,-15)? In real dust2, xbox is right after mid doors on the catwalk corner. Place xbox box centered (-2,-16)? That's inside catwalk footprint... place xbox at (-6,-16)?? Let me restructure top-mid area:

Top mid open area: x[-22,-4], z[-22,-8], y=0. 
- At its east end: xbox box at (-6,-15) size 3x2.4 h1.2. 
- Catwalk: elevated walkway y=2.4 from x[-4] to x[26], z[-18,-12] (6m wide, wider near site = "short"). Access from ground: jump top-mid→xbox→catwalk. Also CT-side access: from CT spawn area there's "cat stairs"?? In real dust2 CTs get on cat via xbox too (from CT mid area, jump on xbox from north side). Xbox reachable from both sides then. OK.
- North wall of top mid: z=-22 wall from x-22..-4.
- East of top mid at ground level: corridor "CT mid" from mid doors area x[-10,6], z[-8,0]?? I'm conflating. Let me define CT-side mid area: region x[-10,10], z[-10,2], y=0, bounded south by mid doors wall (z=0 across x[-22,-10]) and by walls x=-10? Hmm.

I think cleaner: The mid doors wall runs along z=0 from x=-22 to x=-10 (across mid corridor only). East of mid corridor at ground level (x[-10,10], z[-8,2]) is "CT mid/short-below" area connecting mid-doors exit → east to CT spawn. Its south boundary: building wall z=2 from x-10..10 except opening? It connects south to nothing (T side blocked by mid-doors wall only across corridor; east part at z=2 is wall). North: z=-8 opens to top-mid (west part x[-22,-4]) and to CT spawn (east part x[-4,10]... CT spawn is at z[-28,-16] though, so corridor from x[-4,10], z[-16,-8] connects up into CT spawn. OK:

- CT mid area: x[-10,10], z[-8,2].
- Corridor CT: x[-4,10], z[-16,-8] opening into CT spawn (x[-6,14], z[-28,-16]).

Walls needed: around these. Under catwalk (z[-18,-12], x[-4,26]) the ground area is CT-side territory ("below cat"); catwalk needs support wall/pillars; in real dust2 catwalk edge has a wall on its north side (site side) partially? The catwalk is against a building on north. I'll put wall on catwalk north edge (z=-18, height from 2.4 to 5) so cat walkers are protected from site?? No wait—cat LEADS to site; the wall would block entry. Real: catwalk north side is open toward A site plateau (cat is at site level, site continues north). Actually catwalk/short: elevated path hugging the building; its south edge overlooks CT mid (drop down), north edge is a wall (building face) EXCEPT at the east end where it opens onto A site ("short" exits onto site near "default/boost"). And there's the famous "short stairs"? none. So: catwalk z[-18,-12], north wall at z=-18 from x[-4]..x[20] (h 2.4→5), open from x[20,26] (entry to site). Site plateau starts x=26? then short exit at x[20,26] is a gap onto site — make site plateau x[24,62].

Also below-cat ground: x[-4,26], z[-18,-12], y=0 — connects CT mid (west, x=-4? need opening at x=-4 between z[-12? ...]) hmm CT mid is z[-8,2]; below-cat z[-18,-12]; gap z[-12,-8] — wall or corridor? Make wall across z=-10 from x=-4 to x=26 with an opening at x[16,20]?? That's "CT to B"? no... In real dust2, below catwalk connects: CT spawn ←(CT mid)→ below cat → (east) "B link"? No, east of below-cat is A site CT approach? Real dust2: from CT spawn going east: "CT" corridor to A via "A ramp/cross". The area below catwalk on the east leads to... "short" has a drop into CT side near "cat corner". Honestly the area under short in dust2 is "B site"? No.

Let me look at real dust2 layout mentally (I've played it): 
- CT spawn is central-north. 
- From CT spawn going south: mid area with mid doors (west) — "CT mid" is the area between CT spawn and mid doors, and xbox is there (xbox is CT side!). YES — Xbox is CT-side, at the corner where CT mid meets catwalk. Ts coming up mid see the doors; going right at top mid there's "top mid" and the catwalk entrance is above via... Ts get on cat from top mid via the sloped "catwalk" that starts at top mid level?? Real: catwalk from top mid: you walk up the slope at top mid right side onto catwalk (no jump needed from T side!); from CT side you need xbox jump. YES that's right: "cat" slope starts in top mid (T side), rises along the wall to short/A site. Xbox below the cat-CT-side corner lets CTs climb up.

Revised: 
- Top mid: x[-22,-4], z[-22,-8], y=0. 
- Catwalk ramp: starts at top mid east end (x=-6, z=-16) y=0 rising eastward to y=2.4 by x=2, then flat catwalk x[2,26], z[-18,-12] y=2.4. North wall along z=-18 (from x=-6..20, above cat level). South edge: drop to CT mid below (with railing? no). East end opens to A site at x[24,26]? Site plateau x[24,62], z[-44,-16]. The junction cat→site at (24,-15) is "short". There's a wall corner at site edge making the short entrance a corner ( elbow). Also "short boost" boxes on cat at (18,-15): a box h0.9 you can jump to peek over... skip or include a small box.
- CT mid: x[-10,10], z[-10,0]? and xbox at (0,-14)?? CT mid area: region south of catwalk, north of mid doors: x[-10,14], z[-12,0], y=0. Xbox at (-2,-13)?? that's under catwalk z[-18,-12]... xbox at the base of cat near its west end on CT side: place at (0,-13.5), size 2.6x2.4, h1.2. Cat at that x has y=2.4? cat ramp reaches 2.4 at x=2; at x=0 y≈2.2, jump from xbox(1.2)→cat need 1.0: ok fine.

Hmm wait, catwalk ramp from top mid: top mid is z[-22,-8]; cat z[-18,-12] overlaps top-mid z range at x[-6,-4]—start ramp at x=-6. But top mid east boundary at x=-4... extend top mid to x=-6→ cat ramp begins within top mid. And north wall z=-18? top mid z[-22,-8] so top mid extends north of cat's north wall: region x[-22,-6], z[-22,-18] is open T area (call "top mid north"). Its north boundary: wall z=-22? That region in real dust2 has "outside long"? no. It's just top mid corner with wall. OK.

- CT spawn: x[-6,16], z[-30,-14], y=0. Connects: south to CT mid via opening x[-4,10] at z=-14→ corridor z[-14,-10]? Merge: CT mid z[-12,0]; corridor x[-4,10] z[-14,-12]; CT spawn opens at z=-14 wall with opening x[-4,10]. CT spawn north wall z=-30, west wall x=-6 with opening to B link corridor.
- B link (CT→B corridor): x[-36,-6], z[-34,-24], y=0 ("B hall"), enters CT spawn through west wall opening at z[-30,-26]? Let me: opening in CT spawn west wall x=-6 at z[-30,-26]; corridor x[-36,-6] z[-32,-24]; at west end, B site east wall x=-36 with B doors (two closed door boxes with gap? B doors in real dust2 ARE passable? B doors: double doors, closed, with gap between you can... you CANNOT pass B doors either; they're closed. Wait no—in CS:GO, B doors ARE open? The "B doors" from tunnels?? The B site doors (from CT/"window room") are closed double doors with a gap? Hmm. B site has: "window" (jumpable hole in wall from CT corridor) and "B doors" — I believe the doors are closed/impassable but have bullet gaps, and CTs enter B via window jump or... no wait, CTs retake B through B doors! Yes—B doors you walk through? Let me recall playing: retaking B on dust2 as CT: you come from CT through the corridor and enter site through... "B doors" — the double doors are closed and you CAN'T open them; retakes come through window?? No no. In CS:GO dust2, B site: "doors" (double doors between site and tunnels exit?? "upper tunnels" exits to site via an opening; "B doors" refers to the doors between B site and CT corridor — they are CLOSED but there's a gap beside? Actually I remember: you CAN walk through B doors — they're partially open double doors with a gap in the middle you can slip/see through. Yes! B doors have a center gap ~1m, passable-ish? In-game, B doors: two doors slightly ajar; you CANNOT pass; you see through the slit; CTs enter B via WINDOW (jump) or... CTs enter B from spawn via the corridor that leads to "B platform back"? 

Let me settle: In CS:GO Dust2, B site connects to CT side via: (1) B tunnels (upper tuns exit), (2) window (a hole in the wall at head height, jumpable, in the "wall" between site and CT corridor), and (3) B doors — double doors that are CLOSED (impassable, but visible slit; wallbangable). Hmm, but then CT retake only via window+tunnels? I genuinely recall CTs coming through "doors" during retakes... In 1.6 dust2, B doors you pass through? I'm fairly sure CS:GO B doors are impassable and the window is the CT entry, plus "B doors" gap you can crouch?? 

Checking memory of gameplay: "He's going window!" — yes window jumps. And B doors: you can actually pass through the middle gap? There's a clip of people going through B doors gap... I believe the gap between B doors is too small to pass (like mid doors). Retakes: window + tunnels + "car" side? The CT corridor to B ("B doors corridor") ends AT the doors+window wall; so retake = window or flank through tunnels. OK I'll make B doors impassable with 0.35 slit, and window a 1.4w x 1.2h hole at y0.9 (jumpable), since that's authentic-ish. Hmm but risk: bots CT can't retake B except window jump + tunnels. I'll add jump-flagged waypoint through window. Fine. Also for fun, allow doors gap passable? Keep authentic: impassable, window jumpable. But bot pathfinding through window: waypoint outside window, waypoint inside, edge flagged jump; bot aims at window center and jumps. The window hole y[0.9,2.1]: bot must jump through a hole — collision-wise the bot AABB while jumping rises 1.5 max; hole bottom 0.9 top 2.1 (1.2 tall); crouched bot height 1.2? standing 1.7 won't fit! Real window requires crouch-jump. Bots can't crouch-jump easily... make window hole y[0.8, 2.6] (1.8 tall) so standing jump fits. Slightly less authentic but works. Player can also jump through (crouch not needed). OK.

Alternatively make B doors passable (one door ajar with 1.2m gap). You know, gameplay-wise passable doors is better for bots and flow, and many dust2-likes make B doors passable. Decision: B doors = two door panels ajar with 1.0m center gap PASSABLE (authentic to CS 1.6-ish? whatever) — plus window jumpable. This guarantees bot flow both ways. Good.

- B site: x[-62,-36], z[-44,-20], y=0. East wall x=-36 (doors at z[-30,-26], window at z[-38,-34]). South wall z=-20 with upper-tunnels exit opening at x[-50,-44]. North wall z=-44. Platform (B平台） along north-west: elevated y=1.2: x[-60,-48], z[-42,-34], ramp on east side (x[-48,-44] slope) plus jump-up edges. "Car" box at (-44,-24)? car near doors corner: box (car) at (-42,-25) size 3.4x1.8 h1.1. Site boxes: stack at (-52,-28) 2.2x2.2 h1.3 + smaller box. Plant zone: rect x[-58,-44], z[-40,-26].
- B tunnels: lower: from T spawn west side: corridor x[-62,-52], z[12,34] (entry from T spawn at z=34? T spawn x[-58,-34] z[32,44]; opening in T spawn west wall x=-58? T spawn touches perimeter x=-64... move T spawn x[-56,-32]. Opening at T spawn west wall x=-56, z[20,26]?? hmm tunnels entry south of T spawn: In real dust2, tunnel entrance is at T spawn's north-west going down. Let me: tunnel entrance corridor from T ramp area west: opening at x=-34 (T ramp west wall) z[28,32] → lower tunnels corridor x[-52,-34]? Getting messy. 

Simplify tunnels: 
- Tunnel entry from T spawn north-west corner: opening in T spawn north wall at x[-52,-46] (z=32 wall) → corridor "upper entry" x[-52,-46]... 

Cleaner: tunnels = two parallel-ish corridors on far west:
- Lower tunnels: x[-60,-52], z[-2,34], y=0... with entry at south end from T spawn (T spawn west edge x=-56... opening at x[-60,-52] z=32? T spawn x[-56,-32]: doesn't cover x[-60,-52]. Move T spawn to x[-58,-30], z[32,44]; west wall x=-58 opening at z[34,40] into lower tunnels at z[34,40]? Lower tunnels x[-60,-52] z[6,40]. Entry overlap z[34,40] ✔.
- Lower→upper: at north end of lower, turn east: junction x[-60,-44], z[-2,6]; stairs/ramp up? Upper tunnels slightly elevated y=0.8 (real upper tuns higher). Ramp in junction rising northward y0→0.8.
- Upper tunnels: x[-52,-40], z[-18,4], y=0.8, corridor; south end connects junction (opening at z=4, x[-48,-44]); north end exit into B site via arch at B site south wall z=-20, opening x[-50,-44]. Tunnel roof h: 2.6 ceiling (add ceiling boxes for tunnels, dark). 

Mid-to-B link? None (authentic).

T spawn north wall z=32 openings: to T ramp (x[-44,-36]) and... T ramp x[-34,-10], z[24,40]: its west wall x=-34 z[24,32]? opening to T spawn at z[32,40] shared boundary — T ramp touches T spawn along z=32 x[-34,-30]? T spawn x[-58,-30]; shared edge x[-34,-30] at z=32: make opening there (no wall segment between x[-34,-30] on z=32). Then T ramp north wall z=24 with opening to bottom-mid corridor x[-22,-10] (opening x[-20,-12]) and opening to suicide pit? Suicide pit west of mid: x[-32,-22], z[12,24]?? T ramp west region... suicide should be adjacent to mid corridor west wall, near T side: place suicide pit x[-32,-24], z[10,22], floor -1.3, entry from T ramp north-west (opening in T ramp north wall x[-30,-26] with ramp down), and a chest-high wall on its north edge overlooking... nothing? Real suicide overlooks mid doors area for AWP. North wall of suicide = the mid west wall extension; make it h1.1 so you can see/scope over into CT mid?? That'd be a strong sightline (it is, in game, sort of—suicide sees top mid). Keep wall h=1.15 at z=10 x[-32,-24]: from pit floor -1.3, eye at 0.32 — can't see over. Make pit shallower: -0.9, wall h 1.0: eye 0.71 vs wall 1.0 no. Whatever: make suicide just a pit with ramp, sightline over its north wall to mid corridor top? Fine—flavor area; or skip suicide. SKIP suicide to save complexity? It's not in required list. Required list: T出生， CT出生， A包点， B包点， 中路， A大， A小， A坑， A斜坡， 中门， 双门， Xbox, B洞， B门， B平台， 警家， 长门， 箱体掩体， 狙击对枪线， 下包点， 防守架点， 进攻路线， 回防路线， 绕后路线. 双门 = double doors (mid doors). 长门 = long doors. A坑 = pit. No suicide needed. Skip suicide — put solid building block west of mid corridor (x[-34,-22], z[10,26] as building mass) — simpler. T ramp then x[-34,-10] z[24,40] with building at north-west corner. OK.

Now A side details:
- CT spawn east: opening at CT spawn east wall x=16? CT spawn x[-6,16]; east wall x=16 with opening z[-24,-18] → corridor "CT→A link" x[16,34], z[-26,-16], y=0. This is "cross/CT ramp area" leading to A ramp.
- A ramp (A斜坡）: slope from y=0 at (x[30,44], z=-16) up to y=2.4 at z=-28? A site plateau z[-44,-16]... hmm ramp enters site from south: site south edge z=-16 for x[24,44] is the ramp slope down to cross area y=0. So site plateau x[24,62], z[-44,-16] y=2.4. South of it: "cross/under-ramp" area x[24,50], z[-16,-2], y=0, with the ramp occupying x[30,44], z[-16,-6]? Ramp from cross (y0, z=-6) up north to site edge (y2.4, z=-16) — wait that's backwards: site edge at z=-16; ramp slopes from z=-6 (y0) to z=-16 (y2.4) over 10m: slope 24%: fine (like dust2 A ramp). Region x[30,44].
  - West of ramp x[24,30], z[-16,-6]: under-cat/short-side area connecting to below-cat x[-4,26], z[-18,-12]... below-cat y=0 region x[-4,26], z[-18,-12]: connects at x=24..26 to ramp area ✔ (opening at x=24, z[-16,-12]).
- Long approach: cross area x[44,56], z[-16,-2] y=0 connects: west to CT-A link (x=34..44? extend cross x[24,56] z[-16,-2]), east to long corridor via "long corner". Long corridor along east edge: x[54,64], z[-16,26], floor rising from y0 at z=-2 (cross level) to y=2.4 at z=-16?? No wait: long corridor approaches site at plateau level at its north end: corridor floor y=2.4 for z[-16,-8], sloping down southward to y=0 by z=2, continuing y=0 to long doors at z=18..22. "Long corner" at (58,-12) plateau level with blue box; the site connection: opening in site east edge? Site plateau includes x up to 62; long corridor x[54,64] overlaps site x range [24,62] at z[-16,-8] — so corridor north end merges into site (no wall between at z[-16,-8], x[54,62]) ✔. Wall between site and corridor for z[-8,?]: at x=54? For z[-8,2] corridor is sloped 2.4→0, adjacent site? site z range ends -16; south of site (z[-16,-2]) at x[44,54] is cross y=0. Wall x=54 z[-16,-2]? That would block corridor from cross. Real dust2: long corridor at "long corner" is at site level; going south down the corridor descends ("long ramp"); the corridor's west wall separates it from cross/CT area; you enter long from cross only at the corner (north end) or via long doors far south. ✔ so wall x=54 from z=-2 to z=22 (south of corner slope start?) hmm if corridor slope spans z[-16,2], then for z[-16,-8] corridor is open to site (no wall) and for z[-8,2] wall x=54? The wall would cut through slope... simplify: corridor flat y=2.4 for z[-16,-10]; slope z[-10,0] down to y0; flat y0 z[0,22]. Wall x=54: z[-10,22] (separates corridor from cross/doors area). Opening cross→corridor at z[-16,-10] (north end, plateau level) ✔.
- Long doors （长门）: at z=20 across corridor x[54,64]: two arch doorways: pillars boxes: x[54,56], x[60,64]?? Real long doors: building with two door openings side by side. Make wall z=20 x[54,64] h4 with two openings each 2.2w: openings x[55,57.2] and x[60.8,63]: center pillar x[57.2,60.8] 3.6 wide? Hmm make openings x[55,57.5] & x[60.5,63], pillars: left x[54,55], mid x[57.5,60.5], right x[63,64]. Above openings: lintel boxes y[2.6,4].
- South of long doors: "outside long" area x[46,64], z[20,34], y=0, connecting west to T ramp?? In dust2, outside long connects to T side via "long doors" area and "pit". Route T→long: from T ramp (x[-34,-10], z[24,40]) eastward: corridor along south edge x[-10,46], z[26,38]?? That's a long south connector ("T to long"). In real dust2, from T spawn you go east through "T ramp/long entrance"... the route: T spawn → east → "outside long" (passing south of mid) → pit → long doors → long. Yes there's a south-east corridor. Add: corridor x[-8,48], z[28,40], y=0, with cover boxes; west end opening into T ramp east wall x=-10 at z[30,36]; east end opens into outside-long area (x[46,64] z[20,34] overlaps at x[46,48] z[28,34] ✔).
- Pit (A坑）: dug area x[46,54], z[12,20], floor y=-1.4, with ramp down from north side (from outside-long going south into pit? pit is just outside long doors on the west side: entry ramp from x[46,54] z[20,24]? Let me: outside-long area z[20,34]; pit z[12,20] north of it, i.e., pit is between outside-long and long doors — you drop into pit to fight long. Entry: ramp on pit south edge (from outside-long, z=20, y0 down to z=14? make ramp x[48,52], z[16,20], from y0 at z=20 to y-1.4 at z=16; then pit floor z[12,16]. North wall of pit: chest-height? From pit you see through long doors north—actually pit's value: from pit you can peek over its north edge toward long corridor?? Pit is south of long doors; pit north edge is at the doors. You use pit to hold the doors exit / peek over pit edge toward corridor? You crouch in pit, stand to peek over its south edge toward... Pit overlooks the approach from T side? In dust2, pit is used to hold long exit: you sit in pit with AWP aiming through long doors up long... no wait, from pit you look INTO long corridor through the doors? Pit is on T side of the doors; from pit (elevated edge) you can see through the two doors up the corridor — yes! Ts in pit can hold long. And "pit ramp" lets you walk from pit up to doors level. So pit floor -1.4 relative to doors area, with sloped ramp on its north side up to y0 at doors (z=20). So: pit x[46,54], z[14,22], floor -1.4; ramp along north: x[48,52], z[18,22] rising z18(y-1.4)→z22(y0)?? z22 > doors z20... Let me put pit z[16,24] with north ramp z[16,20] rising to doors plane z=20 y0; outside-long area south of pit z[24,34] with a drop into pit (1.4, jumpable down, and a small side ramp east x[52,54] z[20,24] for climbing back? or make pit south edge a 0.5 step? Just make pit floor -0.9 so you can jump out (jump 1.5 ✔). Simplify: pit depth 0.9, no internal ramps needed except north slope to doors level: actually if depth 0.9 < jump 1.5, you can jump out anywhere ✔, and north edge at doors y0 gives the peek: standing in pit eye height = -0.9+1.62=0.72 vs surrounding y0 ground → can't see over! Pit only works if it overlooks something lower... Real pit: ground slopes; from pit you see through long doors because doors area is at same level as pit surroundings and corridor beyond is visible through doorway. Since doors plane is at pit north edge at y0, from pit (eye 0.72) through a doorway (opening y0..2.6) you CAN see up the corridor ✔ (doorway is tall). And pit gives cover from corridor fire (your body below ground relative to corridor? corridor at that z is y0 sloped...). Good enough. Pit depth 0.9, rectangular, with slight north ramp? skip ramp—jump out.

OK and "goose": niche at site NW corner: wall alcove at (26,-42) with boxes. Site boxes: triple stack at (38,-32): three boxes 1.4³,1.4x1.4 h2.8? "triple box" stack near default plant: boxes at (38,-32) h2.6 column + neighbors. Default plant zone: x[34,46], z[-38,-28]. Ramp-side "default" boxes, "boost" box near short entry at (26,-20)? Also "A site" big green boxes at (50,-36) 2 stacks (like site "quad"/"big box"), "ramp box" at (40,-20) edge. Blue box at long corner (56,-14) h1.1.

Sniping lines ✔: T ramp→mid (through mid gap to CT mid/CT spawn edge), pit→long, cat→site, B doors corridor→site.

Waypoints Dust2 (id: x, z, [links], opts):
T side:
1 tspawn (-46,38) links: 2,3(tunnels)
2 tramp (-22,32) links 1,4(mid),5(longconn)
3 tunn_low (-56,30) l:1,6
4 midbot (-16,20) l:2,7
5 longconn_w (-4,34) l:2,8
6 tunn_junc (-52,2) l:3,9(jump? no, ramp)
7 midtop (-16,-15) l:4,10(cat ramp start), and midbot... also link to? mid is dead-end at doors (closed) ✔ authentic.
8 longconn_e (30,34) l:5,11
9 tunn_up (-46,-8) l:6,12
10 catbase (-6,-16) l:7,13 (ramp up, walking)
11 outlong (50,30) l:8,14(pit),15(doors)
12 tunn_exit (-47,-22)→ in B site l:9,16
13 catmid (10,-15, y2.4) l:10,14? no: l:10,17(short entry)
14 pit (50,19, y-0.9) l:11,15
15 longdoors (58.5,21) l:11,14,18(longcor)
16bsite_center (-50,-32) l:12,19(platform),20(doors)
17 short (25,-15,y2.4) l:13,21(site),22(goose)
18 longcor (58,4) l:15,23(longcorner)
19 bplatform (-54,-38,y1.2) l:16,20? l:16
20 bdoors_ctside (-33,-28) l:16? bdoors gap at (-36,-28): wp inside site (-38,-28) l:16, and outside (-33,-28) l:24(bhall). So: 20 (-38,-28) l:16,25.
21 asite (40,-34,y2.4) l:17,22,23? l:17,26(ramp top),27(longcorner site)
22 goose (27,-41,y2.4) l:21
23 longcorner (57,-13,y2.4) l:18,21
24 bhall (-20,-28) l:25(ctspawn? no), CT side: bhall corridor x[-36,-6] z[-32,-24]: wp (-20,-28) l:20(bdoors out (-33,-28)),26(ctspawn)
25 bdoors_out (-33,-28) l:20,24
26 ctspawn (5,-22) l:24,27? links: bhall wp24, ctmid wp28, ctA link wp29
27? renumber later in code.
28 ctmid (0,-6) l:26(ctspawn via corridor), xbox wp (0,-13.5)? xbox top wp l: ctmid (jump), cat (jump).
29 ctA (25,-10) l:26,30(cross),31(belowcat)
30 cross (46,-8) l:29,23(longcorner via opening z[-16,-10]... cross wp (46,-8)→longcorner wp23 (57,-13): path through opening ✔),32(aramp bot)
31 belowcat (10,-15) l:29? below cat ground y0 z[-18,-12]... connects to CT mid? CT mid z[-12,0]: boundary z=-12: is there wall? catwalk above z[-18,-12]; below-cat ground open to CT mid at z=-12 ✔ (no wall) for x[-4,20]? but then Ts from top mid could drop from cat ramp to CT mid — that's authentic (drop from cat). Also wall needed between top-mid (z[-22,-8] x[-22,-4]) and CT mid at x=-4?? Top mid east wall: x=-4 z[-22,-8]? If open, Ts walk top mid→CT mid directly bypassing doors — NOT authentic (mid doors block). The mid doors wall z=0 spans only corridor x[-22,-10]. East of x=-10 at z=0 is CT mid south boundary — wall z=0 from x=-10 to x=14? CT mid z[-12,0]; south wall z=0 x[-10,14] ✔ solid (that's the mid doors building east face). North of CT mid: cat above for x[-4,26]... for x[-10,-4] z[-12,-8]: this strip connects CT mid to top mid? Wall needed at x=-4 z[-22,-8]? Top mid x[-22,-4]; CT mid z[-12,0] x[-10,14]: they overlap x[-10,-4] z[-12,-8]!! Overlap region must be assigned: make it wall (building mass): solid box x[-10,-4], z[-12,-8]. Then top mid boundary east = x=-4 for z[-22,-12]? and cat ramp starts at (-6,-16) rising eastward over CT mid?? cat ramp from (-6,-16) y0... but below-cat ground at x[-6,-4] z[-18,-12] is top-mid? Ugh. Let me simplify: catwalk ramp starts in top mid at x=-8: top mid x[-22,-4] z[-22,-8]; cat ramp occupies z[-18,-12], x[-8,2] rising 0→2.4 (ramp over top-mid east portion). Below-cat ground exists only x[2,26], z[-18,-12]. Wall at x=-4? top mid east boundary at x=2?? cat ramp footprint x[-8,2] means top mid extends to x=2 under the ramp start (ramp low there, height at x=-8 is 0 rising to 2.4 at x=2: clearance under ramp for x<0 is <1.2 — put solid fill under ramp x[-8,2] z[-18,-12] below ramp surface (fill box). Then top mid effectively x[-22,-8]. CT mid x[-10,14] z[-12,0] but x[-10,2] z[-12,-8] under ramp fill → CT mid effective x[2,14]?? That kills CT mid corridor to below-cat... CT mid connects: mid doors (z=0, x[-22,-10]... wait mid corridor is x[-22,-10]; CT mid is beyond doors... 

Hold on, mid doors are at z=0 across mid corridor x[-22,-10]; north of doors (z<0) at x[-22,-10] is top mid (T side?) NO — top mid is T side (south of doors), doors at z=0, top mid z[-22,-8]?? I had mid corridor z[-8,14] with doors at z=-2..0 — I flipped. Let me redo orientation: T spawn SOUTH (z=+40). Mid runs north-south: T end at z=14 (bottom mid), north to z=-8 (top mid). Doors sit at z≈+2?? Real: from T ramp you enter "mid" going north; doors are about 60% up; beyond doors (north) is "top mid" (still T territory, dead-end with cat entrance on right). CT side of doors: there IS no walkable CT side at mid level — behind doors is... the doors are IN a wall that spans the whole width between T mid and CT area; north of that wall IS top mid?? No!! 

Real dust2 mid geography (final answer): Mid corridor from T ramp north: lower mid → mid (with suicide on left) → mid doors (the double doors cross the corridor) → beyond doors is "top mid" — and top mid is where? CTs can reach top mid? Top mid has: catwalk entrance (right/east, elevated slope up to cat), and left wall, dead end north. CTs see top mid through the door gap. The doors separate "mid" (south, T) from "top mid" (north, still effectively contested/T). CT side of the door wall: the wall extends EAST beyond the corridor — the door building's back side faces CT mid?? 

Honestly: In CS:GO dust2, mid doors: the double doors are in the corridor; just north of them is top mid with xbox on the RIGHT (east) at the base of catwalk, and CTs access top mid?? CTs coming from CT spawn go to "CT mid" which is... there IS a passage: CTs can walk to top mid?? YES — top mid connects to CT spawn area around the east side of the doors building: you go CT spawn → south → "CT mid"/"outside mid" → west → top mid?? Hmm, no! I recall: CT mid is the area in front of (north of) the mid doors?? And xbox sits in CT mid against the catwalk corner. CTs boost xbox→cat. Ts boost... Ts enter cat from top mid via the cat slope?? and top mid is T side, doors between mid and top mid?? Then how do CTs contest mid? Through the door gap from CT mid... 

DEFINITIVE memory: Playing dust2: As CT, from spawn you can run to "mid" — you arrive at an area with xbox; ahead of you are the mid doors (closed, with the gap); through the gap you see T mid (suicide/T ramp). As T, you come up mid, and at the top you face the mid doors; to your right is... top mid continues right (east) — "top mid" is the T-side area before the doors?? When T, at top mid you can go right onto cat slope?? The cat entrance is CT side near xbox... 

OK here's the actual map: Mid doors wall runs diagonally/horizontally separating T mid (south) from CT mid (north). Top mid = T side right before doors. Catwalk: from TOP MID (T side), the right side has a ramp up ("cat") that curves over/around... and xbox is in CT mid at catwalk's west end bottom; CTs jump xbox→cat to contest. YES. This matches: "cat" fights happen Ts coming up the slope vs CTs on cat/short. And CT mid = area north of doors with xbox, connecting east to CT spawn. That's what I'll build:

- T mid corridor: x[-22,-10], z[2,14] (bottom mid from T ramp), continuing z[-16,2] as "top mid" up to doors at z=2!! Doors at z=2 across x[-22,-10]. Top mid = x[-22,-10], z[-16,2]?? But top mid needs the cat ramp on its east (right when facing doors/north? facing north, right = east = x+). Cat ramp at top mid east: but corridor east wall x=-10... opening in east wall z[-14,-8] → cat ramp area x[-10,-2], z[-16,-10], slope rising eastward y0→2.4 by x=-2, then catwalk flat x[-2,26], z[-16,-10], y=2.4. Catwalk north wall z=-16 (h to 5), x[-10,22]; east end opens to site at x[22,26]... site plateau z[-44,-16] though — gap z[-16,-10] at x[22,26]: extend site plateau south edge to z=-10 for x[22,26]?? Make site plateau x[24,62], z[-44,-10] (south edge z=-10 for the short/ramp side; CT side ramp descends from z=-10 southward). Wait earlier cross/ramp: A ramp from cross y0 at z=-2 up to site at z=-10? Let's set: site plateau z[-44,-10]; A ramp slope z[-10,0] x[30,44] descending south to cross y0 at z=0; cross area x[24,56], z[0,10]?? but CT spawn is at z[-30,-14]... cross should connect CT: CT-A link corridor x[16,34] z[-26,-16]... that ends at z=-16 which is site plateau south?? conflict.

Simplify CT-A connection: CT spawn east opening → corridor east x[16,30], z[-24,-14] → turns south into cross x[24,56], z[-14,-2], y=0 → A ramp z[-10,-2]?? ramp z[-10,-2] too short for 2.4 rise (8m ok! slope 30%). Ramp x[30,44], z[-10,-2]: y0 at z=-2 → 2.4 at z=-10 ✔. Cross z[-14,-2] minus ramp footprint ✔ connects long corner opening at x[54,62]? long corridor x[54,64], z[-14,26]?? earlier long corridor z[-16,26]: keep long corridor x[54,64], z[-14,26]: plateau z[-14,-8] y2.4 (merge with site at x[54,62] — site covers x[24,62] z[-44,-10]; corridor plateau z[-14,-8] x[54,64]: overlap region x[54,62] z[-14,-10] shared ✔ open). Slope z[-8,0] down to y0. Flat y0 z[0,20]. Long doors at z=18 across x[54,64]. Outside long x[46,64] z[18,34]. Pit x[44,54] z[10,18]?? pit adjacent south of doors: pit z[18,26]? doors z=18; pit should be just south of doors: x[44,54], z[18,26], floor -0.9. Outside long around it x[44,64] z[18,34]. Cross-to-corridor opening: wall x=54 z[-8,18] (separates corridor from cross & outside-long); opening at z[-14,-8] (plateau level) ✔. Cross east boundary x=56... cross x[24,56] meets corridor x[54,64]: overlap x[54,56] z[-14,-8] opening ✔ (no wall there), wall x=54 from z=-8 to z=18 ✔, and for z[-14,-8] x=54 open ✔.

Wait, cross z[-14,-2] and corridor plateau z[-14,-8]: cross at y0, corridor at y2.4 there — the "opening" at plateau level means cross can't reach it (cross y0, corridor floor 2.4!). In real dust2, long corner IS at site level and cross connects via "long corner" being... CTs go A site via ramp or via long corner from cross?? Real: cross is at site level! The "cross" is the corner where CT-side path meets long corridor at plateau level, overlooking the ramp. Hmm: A site plateau; ramp descends south from plateau to... "CT" lower area? No — in dust2, A ramp descends from site toward CT spawn and the area at ramp bottom ("CT ramp"/"cross") is at CT level (y0). Long corridor: at "long corner" the corridor floor is at SITE level?? No! Long corridor is LOWER than site — site overlooks long; you drop from site into long? NO — you walk from long onto site at the same level near "long corner"... 

Actual dust2 A-site geography: A site is a plateau. "Long": corridor along east; its north end ("long corner") is at plateau level (corridor rises via slope from doors). The corner where long meets site and the CT-side path is "cross". The CT-side path from CT spawn: you go up "A ramp"? No — CTs from spawn reach site via "CT ramp"? Real callouts around A site: "ramp" (the slope from site down toward CT spawn, on the west side of site), "goose" (corner near ramp top), "cross" (where CT path crosses to long corner — at plateau level? "cross" is the area between ramp bottom and long corner)... I believe: from CT spawn toward A: you reach a junction ("CT"/"A cross"): left = up "A ramp" to site (goose side), right = "long corner" path which is LOWER (long corridor at lower level there, with site above on your left). And "long corner" area is at lower level, with the site plateau edge above it — Ts on site drop down to long corner to exit toward long/CT. YES that's right! Long corridor is at LOWER level (y0) along its whole length? No... long has a slope: from long doors the corridor is flat then rises slightly?? 

CS:GO dust2 long: from long doors going north, corridor is FLAT-ish, slight rise; at the end "long corner" you turn left (west) and there's the site ABOVE you (you boost/jump? no...). At long corner you can go up to site via the corner being at site level?? Argh.

Memory of playing: pushing long as T: through doors, up the corridor (slightly uphill), reach long corner area with blue box; from there the path continues left/west UP a short slope onto A site (site is higher). As CT holding site, you look down at long corner from site edge ("elevation" advantage) — you can drop from site to long corner (fall damage small? no damage — it's like 1.5m drop? site is ~2m above long corner; dropping is fine). And cross: between CT corridor and long corner, at long-corner level (y0), with site edge wall above. The ramp: from site down to CT side on the WEST of cross ("A ramp" from goose corner down to CT). 

REVISED A geometry: site plateau y=2.4, x[24,62], z[-44,-14]. 
- Long corridor y≈0 entire: x[54,64], z[-10,26], flat y0 (slight slope optional—skip). Long corner = corridor north end (58,-8) with blue box. 
- Site↔long corner: site's east edge above corridor; opening in site east edge at z[-14,-10]?? For Ts to walk site→long without damage, drop 2.4m (no fall damage in our game — fine!) and Ts coming FROM long corner to site need to climb: real dust2 has a slope/boxes at long corner up to site? There IS "long corner" with a path up?? I recall at long corner you can walk up onto site via a slope on the left as you enter ("the corner rises")... you know what, I'll add a slope: site plateau extends a ramp tongue at its east edge: ramp x[54,62], z[-14,-6], from y0 (z=-6, corridor level) up to y2.4 (z=-14, site edge). So long corner connects up to site by this slope ("long corner slope"). ✔ playable both ways, matches feel (Ts push up the slope from long corner onto site; CTs hold from above).
- Cross: y0 area x[40,56], z[-10,2], east = corridor (wall x=54? corridor is x[54,64] — cross overlaps x[54,56]: opening z[-10,2]?? that'd let cross walk straight into corridor at y0 ✔ fine and authentic-ish (you can go cross→long corridor directly, that's the "long corner" route). So no wall x=54 for z[-10,2]; wall x=54 for z[2,18] (separating corridor from outside-long south part). Wait outside long is x[44,64] z[18,34]; corridor south end at z=26?? Let me: corridor z[-10,22]; long doors at z=20; corridor continues z[20,22]?? doors ARE the south exit: corridor z[-10,20], doors wall at z=20, south of doors = outside long x[44,64] z[20,34]. Wall x=54 between corridor & outside-long not needed (doors handle it). Wall x=54 needed between corridor and pit/cross south? pit x[44,54] z[10,20] west of corridor south portion: wall x=54 z[2,20] ✔ (pit people can't enter corridor except via doors ✔ authentic).
- Cross connects west to A ramp bottom & CT corridor: A ramp: x[26,40], z[-14,-2] slope: y2.4 at z=-14 (site edge... wait site z ends -14) → y0 at z=-2; ramp footprint x[26,40]. West of ramp: x[24,26] z[-14,-2] walk strip at y0 ("under goose"). South of ramp/cross: wall z=2 x[24,56]?? cross/cross south boundary z=2 wall x[24,44]? and CT link corridor enters cross from west: corridor x[10,26], z[-6,2], y0 connecting CT mid east (x=10?) to cross west (x=26). CT mid x[2,16] z[-10,2]... CT spawn x[-6,16] z[-30,-14] connects to CT mid via corridor x[0,10] z[-14,-10]. So route CT spawn→CT mid→(east) CT link corridor→cross→A ramp or long corner ✔. And CT spawn→B hall→B doors/window ✔. CT spawn→mid doors: CT mid extends west to doors: CT mid x[-8,16], z[-10,2]; mid doors wall at z=2 x[-22,-8]?? doors across mid corridor x[-22,-10] at z=2; wall continues x[-10,-8]?? set doors wall from x[-22,-10] with door boxes; wall x[-10,-8] z=2 solid; CT mid south boundary z=2 from x=-8 to x=16 solid wall. CT mid west boundary x=-8 z[-10,2] solid (building). So mid doors separate T top-mid from CT mid ✔. Xbox in CT mid at (-4,-8)?? catwalk west end: catwalk x[-2,26], z[-16,-10], y2.4. CT mid north boundary z=-10 x[-8,16]: area below catwalk east portion x[-2,16]... catwalk is ABOVE z[-16,-10]; CT mid at z[-10,2] is south of catwalk — xbox sits at catwalk west end on its south side: xbox at (-4,-9) size 2.4x2 h1.2, adjacent to catwalk west end (-2,-13)? Cat ramp from top mid: opening in top-mid east wall z[-14,-8] at x=-10 → ramp x[-10,-2], z[-16,-10] rising y0(x=-10)→y2.4(x=-2). Hmm wait top mid is x[-22,-10] z[-16,2]; its east wall x=-10 with opening z[-14,-8] leads into ramp x[-10,-2] z[-16,-10] ✔. Xbox at (-4,-9): inside ramp footprint? ramp z[-16,-10]; xbox z[-10,-7]? place xbox center (-4,-8.5), size 2.4x2, top 1.2; jump xbox→catwalk: catwalk at x≥-2 y2.4; from xbox top (1.2) jump 1.5 → reach 2.7 > 2.4 ✔ then move onto cat (x from -4 to -2 while airborne) ✔. CTs: stand CT mid, jump onto xbox (1.2 ✔), then cat ✔. 

Catwalk south edge (z=-10): overlooks CT mid (y0) — drop ok. Catwalk north wall z=-16 from x=-2..20 (h 2.4..5). East end x[20,26]: "short" — opening into site at x[24,26], z[-16,-14]? Site south edge z=-14; site covers z[-44,-14]; cat z[-16,-10]; overlap z[-16,-14] at x[24,26]: wall between? site edge x=24 z[-16,-14] opening ✔ plus elbow wall: site corner box at (24,-12)? Add small wall piece x[22,24], z[-14,-10] h2.4 forming the short elbow corner ✔ (cover). Also short has "short boost" box on cat at (14,-13) h0.8 (jump to see over north wall? north wall h2.6 above cat... make north wall h 2.4→4.4 (2m above cat); boost box 0.8 + eye 1.62 = 4.82... >4.4 ✔ can see over into site. eh fine).

"Below cat" at ground: region x[-2,26], z[-16,-10] below catwalk is SOLID fill (catwalk is a building mass: "the cat building") — put solid box x[-2,26] z[-16,-10] y0..2.4 with only top walkable... then CT mid north boundary IS that building face z=-10 x[-2,16] ✔ and xbox against it. But then catwalk ramp x[-10,-2] also solid fill below ✔. And no under-cat passage — fine (real dust2 has no passage under cat either... actually there IS "cat lower"?? no. ok).

Hmm wait: if cat building fills x[-2,26] z[-16,-10] to height 2.4, then site plateau (y2.4) connects seamlessly at x=24/26 ✔ nice. But CT spawn x[-6,16] z[-30,-14]: between CT spawn south (z=-14) and cat building north (z=-16) there's strip z[-16,-14] x[-6,16] walkable? That would let CT spawn walk along cat building north face to... x=16→? east: CT mid x[-8,16] is z[-10,2]... this strip z[-16,-14] connects where? West x=-6: B hall? B hall x[-36,-6] z[-32,-24] — no. This strip is a dead pocket; make it wall (extend CT spawn south wall / cat building). Solid box x[-6,16] z[-16,-14] h3? But CT spawn↔CT mid corridor x[0,10] z[-14,-10] passes through... I said corridor from CT spawn (z=-14 wall opening x[0,10]) to CT mid (z=-10). That corridor occupies z[-14,-10] x[0,10] — which overlaps cat building x[-2,26] z[-16,-10]!! Conflict: cat building east extent should stop at x=10? Set cat building x[-2,10]?? but catwalk must reach site at x=24. Catwalk could be elevated on pillars for x[10,26] (open below): below-cat ground x[10,26] z[-16,-10] y0 walkable, connecting CT mid (z=-10 opening x[10,16]) to cross?? cross z[-10,2] x[40,56]... below-cat x[10,26] east end at x=26 = ramp footprint x[26,40] z[-14,-2]: overlap z[-14,-10] x[26]: opening → below-cat connects to A ramp bottom area ✔. In real dust2 there IS space under short near site?? There's "short" with the under-area near ramp ("pirate"/" ninja" corner?). Eh, acceptable: gives CTs a second route to A ramp bottom — actually this models "CT→A ramp" nicely. And site plateau y2.4 above z[-16,-14] for x[24,26]?? site z starts -14; the strip x[24,26] z[-16,-14] is the short entry at plateau... catwalk x[-2,26] z[-16,-10] y2.4 TOP surface; below y0..2.4: solid for x[-2,10], open (pillars) for x[10,26]. So below-cat corridor x[10,26] z[-16,-10] exists with ceiling at 2.4 ✔ (2.4 clearance ok). Add pillar boxes at (16,-13) & (22,-13) 0.8x0.8.

Then corridor CT spawn→CT mid x[0,10] z[-14,-10]: passes under catwalk west part?? catwalk x[-2,10] solid below — conflict again. Move CT spawn→CT mid connection: make CT mid adjacent to CT spawn directly: CT spawn x[-6,16] z[-30,-14]; CT mid x[-8,16] z[-10,2]; gap z[-14,-10] fill solid EXCEPT opening corridor x[-4,4]: but cat building x starts -2... set cat building solid part x[4,10]?? Ugh.

Cleaner: shift catwalk north: catwalk z[-18,-12], CT mid z[-10,2], corridor CT spawn↔CT mid x[0,10] z[-14,-10], strip z[-12,-10] x[-8,0] open? Let me lay final CT-side coordinates:

- CT spawn: x[-6,16], z[-30,-14]. 
- South wall of CT spawn at z=-14 with opening x[0,8] → corridor x[0,8], z[-14,-10] → CT mid.
- CT mid: x[-10,16], z[-10,2]. North boundary z=-10: solid from x=-10..-2? and x? catwalk above z[-18,-12]... between z[-12,-10] strip x[-10,16]: walkable CT-mid extension (north strip) ✔ with catwalk south face at z=-12 above? Catwalk occupies z[-18,-12]; its south face at z=-12; CT mid strip z[-12,-10] runs along catwalk face ✔. Xbox against catwalk face at (-4,-11), top 1.2, jump to cat (y2.4) ✔. Catwalk solid below for x[-2,16] (fill to y2.4) — but corridor x[0,8] z[-14,-10] overlaps fill region x[0,8] z[-14,-12]!! Conflict again because corridor z[-14,-10] reaches z=-14 north of cat face -12.

Fix: corridor x[0,8] z[-14,-10]; cat building fill x[-2,16] z[-18,-12] — overlap region: x[0,8] ∩ [-2,16] = [0,8]; z[-14,-10] ∩ [-18,-12] = [-14,-12]. Overlap exists. Move corridor west: x[-6,0]? CT spawn opening x[-4,0], corridor x[-4,0] z[-14,-10], cat building x[-2,16]... overlap x[-2,0] z[-14,-12] still (corridor x[-4,0] ∩ [-2,16] = [-2,0]). Shift cat building east: fill x[2,16]; corridor x[-4,0] ✔ no overlap (x[-4,0] ∩ [2,16] = ∅). Xbox then at (0,-11)? jump from xbox top at x=0 onto cat at x=2 — 2m horizontal gap, jump distance ~2.5m at sprint... make xbox (1,-11) size 2.4: spans x[-0.2,2.2] touches cat edge x=2 ✔. Corridor x[-4,0] east wall = xbox/building ✔ fine.

- CT mid east: x[16]? CT mid x[-10,16] z[-10,2]; east opening to CT-A link: opening at x=16, z[-8,-2] → link corridor x[16,26], z[-8,-2] → cross x[26,56], z[-10,2]?? Wait A ramp bottom... A ramp x[26,40] z[-14,-2] slope y2.4(z=-14)→y0(z=-2). Cross = x[40,56] z[-10,2] plus ramp bottom strip x[26,40] z[-2,2]. Link corridor enters at x[26] z[-8,-2]?? that's ramp slope region (z[-14,-2])... opening at x=26 z[-2,2]?? link corridor z[-8,-2] hits ramp west face at z[-8,-2] where ramp height at z=-8: y = 2.4*( (-2)-(-8) )/( (-2)-(-14) ) = 2.4*6/12=1.2 → wall of dirt 1.2 — blocks. Reroute link corridor: x[16,28], z[-2,2]?? south strip: CT mid z[-10,2] already includes z[-2,2]! CT mid IS connected to cross if no wall between x=16 and x=26... CT mid x[-10,16]; region x[16,26] z[-10,2]: make it open corridor (part of CT mid east extension) with north side: below-cat corridor x[10,26] z[-12,-10]?? I previously had below-cat corridor; with cat building fill x[2,16], below-cat open part x[16,26] z[-18,-12]... getting too tangled.

STOP. Final simplification for CT-A connectivity, clean version:

- Catwalk: walkway x[-2,26], z[-18,-12], y2.4, supported by SOLID fill beneath for entire length: box x[-2,26], z[-18,-12], y[0,2.4]. No under-cat corridor. 
- CT spawn: x[-6,16], z[-30,-16]. South wall z=-16 opening x[0,10] → corridor x[0,10], z[-16,-12]?? that hits cat fill at z=-12... cat fill z[-18,-12]: corridor z[-16,-12] ⊂ cat fill z → conflict. 

Move CT mid SOUTH of cat fill with a wall between? CT mid x[-10,18], z[-10,2]. Corridor CT spawn→CT mid must cross z[-16,-10] without touching cat fill (x[-2,26] z[-18,-12]): route corridor WEST of cat fill: corridor x[-10,-4], z[-16,-10] connecting CT spawn (opening x[-8,-2] at z=-16) south to CT mid (x[-10,-4] z[-10,2] part) ✔ no overlap with cat fill (x[-10,-4] ∩ [-2,26] = [-2,-4]?? [-10,-4] ∩ [-2,26] = ∅ since -4 < -2 ✔). 
- Xbox: at (0,-11)?? cat face z=-12; xbox center (0,-10.8) size 2.4x2.2 → spans x[-1.2,1.2], z[-11.9,-9.7], top y1.2, adjacent to cat face ✔ jump onto cat ✔. In CT mid ✔.
- CT mid east → A: opening at x=18? CT mid x[-10,18]; link corridor x[18,28], z[-6,0] → cross. Cross x[28,56], z[-12,2] (y0). A ramp x[28,42], z[-14,-4]: slope y2.4 at z=-14?? site z[-44,-14] ✔ ramp from y0 z=-4 → y2.4 z=-14 (10m run) ✔ footprint x[28,42]. Cross remainder: x[42,56] z[-12,2] + x[28,42] z[-4,2] + x[28,42] z[-12,-14]? that strip is north of ramp top — it's site? site z ends -14 ✔ so z[-14,-12] x[28,42] is cross strip at y0 BELOW site edge (site wall z=-14 face, h2.4) ✔ "under site" strip. Also x[28,42] z[-12,-4] is ramp footprint ✔. So cross = {x[42,56] z[-12,2]} ∪ {x[28,42] z[-4,2]} ∪ {x[28,42] z[-14,-12]}... simplify cross region rect x[28,56] z[-14,2] with ramp overlapping x[28,42] z[-14,-4] — collision system handles overlapping (support fn takes max standable). Fine!
- Link corridor x[18,28] z[-6,0] opens into cross at x=28 ✔ (cross z[-14,2] ⊃ z[-6,0] ✔).
- Cross → long corridor: opening at x=56? long corridor x[54,64] z[-12,20], y0 flat. Overlap x[54,56] z[-12,2]: open (no wall) ✔ "long corner" zone (58,-9) with blue box. Long corner slope up to site: ramp x[54,62], z[-18,-12]: y0 at z=-12 → y2.4 at z=-18?? site z[-44,-14]: at z=-18 we're inside site z range ✔ ramp tongue into site: from corridor level z=-12 (y0) rising north to z=-18 (y2.4 = site) — 6m run, 40% slope, steep but ok (step slabs 0.25 walkable). Footprint x[54,62]. But site plateau surface covers z[-18,-14] at y2.4 there ✔ consistent. Corridor at z[-12,-18]? corridor z[-12,20]; ramp z[-18,-12]: they meet at z=-12 ✔.
- Site boxes etc. Goose at site NW (26,-42). Site west edge x=24: short entry at z[-16,-14]?? catwalk ends x=26 z[-18,-12] y2.4; site x[24,62] z[-44,-14]: overlap x[24,26] z[-18,-14] — cat top meets site corner ✔ walk from cat onto site ✔. Elbow cover wall: box x[26,28], z[-14,-12] h2.8 (the short corner) ✔.
- Site south edge wall z=-14 for x[24,28]? cat z[-18,-12] covers x[24,26]; for x[26,28] z[-14,-12] elbow wall ✔; x[28,42] ramp ✔; x[42,54] z=-14 edge: site wall above cross strip? cross z[-14,-12] x[42,54] at y0 below site face z=-14 ✔ that's fine (site face wall h2.4 from y0: actually the plateau edge — represent as fill: site plateau is fill box y[0,2.4] over site rect! Then cross strip z[-14,-12] must NOT be inside site rect: site z[-44,-14] ✔ strip z[-14,-12] outside ✔ good).
- So site = solid fill box x[24,62] z[-44,-14] h2.4 with top walkable; its faces: north z=-44 (perimeter), west x=24 (goose side, faces top mid?? west of site x<24 z[-44,-14]: what's there? North-west region x[-2,24] z[-44,-18]?? Unassigned! Cat north wall z=-18 x[-2,20]; north of cat x[-2,24] z[-44,-18]: in real dust2 this is... "A site" extends west to goose at x~24; west of that is wall/buildings then B site/CT. Fill with building mass: box x[-6,24], z[-44,-18], h4 (building) — forms the north wall behind cat ✔ and goose is site NW corner against it... goose should be a nook: make building x[-6,24] z[-44,-20], leaving goose nook x[24,?]... goose is ON site: site x starts 24; goose nook = corner at (26,-42) against building face x=24?? Site west face x=24 z[-44,-20] (building z[-44,-20]) then site edge z[-20,-14] west face x=24 borders... region x[20,24] z[-20,-14]?? cat x ends 26... cat x[-2,26] z[-18,-12]: covers x[20,24] z[-18,-14]?? z[-18,-12]∩z[-20,-14]=[-18,-14] yes cat there. OK: building mass x[-6,24] z[-44,-20] h5; site west face: x=24 z[-20,-14]: borders region x[20,24] z[-20,-14]: partially cat (z[-18,-14]) and strip z[-20,-18] x[20,24]?? cat z starts -18: strip z[-20,-18] x[20,24] is... pocket. Fill: extend building mass x[-6,24] z[-44,-18]? then cat north wall at z=-18 IS the building face for x[-2,20] ✔ and pocket gone (building z[-44,-18] covers z[-20,-18] x[20,24]? no: building x ends 24 ✔ covers x[20,24] z[-20,-18]? z[-20,-18] ⊂ z[-44,-18] ✔ yes covered). Then site west face x=24 z[-18,-14]: short entry opening ✔ (from cat x ends 26 > 24 ✔). Site west face z[-44,-18] = building face (h5 above 2.4: box y[0,5]) ✔ goose corner at (26,-41) against it, add goose nook detail: small wall bump x[24,27], z[-38,-36] h2.8? fine (cover).

- B side: B site x[-62,-36] z[-44,-20]; east wall x=-36: doors at z[-30,-26]: door boxes: (-36, z[-30,-28]) & (z[-27,-26])?? passable gap 1.2 at z[-28,-27]?? Let me: wall x=-36 z[-44,-20] solid except: doors opening z[-30,-26] (4m) filled with two door panels z[-30,-28.6] & z[-27.4,-26] leaving gap z[-28.6,-27.4] (1.2m, passable ✔); window opening z[-38,-34]: hole y[0.9,2.5] (jumpable, 1.6 tall ✔ standing jump fits). B hall corridor x[-36,-6], z[-32,-24] y0 connecting CT spawn west opening (x=-6, z[-30,-26]) to B doors/window ✔. Wait CT spawn x[-6,16]: west wall x=-6 opening z[-30,-26] ✔. B hall south wall z=-24, north wall z=-32.
- B platform (B平台）: elevated y1.2: x[-60,-48], z[-42,-34] fill box h1.2; ramp x[-48,-44], z[-40,-34]? slope y0(x=-44)→1.2(x=-48) — 4m run 1.2 rise ✔. Platform edges jumpable ✔.
- B site boxes: big stack (-52,-28) 2.4x2.4 h1.4; second box (-49,-25)? "car": box (-42,-24) size 3.6x2 h1.2 near doors corner. Plant zone x[-58,-44] z[-40,-26].
- Tunnels: lower x[-60,-52] z[4,38]?? entry from T spawn: T spawn x[-58,-30] z[32,44]: overlap x[-58,-52] z[32,38] — opening ✔ (no wall between). Lower tunnels y0, z[4,38]; junction at north: x[-60,-44], z[-2,6]; ramp up in junction: x[-52,-46] z[-2,6]?? upper tunnels y0.9: corridor x[-52,-42], z[-18,4], y0.9 flat; ramp from junction y0 at z=4 → y0.9 at z=-2 (6m) ✔ footprint x[-50,-44]. Upper exit into B site: B site south wall z=-20 opening x[-50,-44]; upper tunnel z[-18,4] → gap z[-20,-18] x[-50,-44]: corridor piece x[-50,-44] z[-20,-18] y0.9 ✔ (exit arch). Tunnel ceilings: boxes y[3.4,4] over tunnel corridors (dark inside, nice). Tunnel walls: west x=-60/x=-52 sides etc.
- Junction connects lower (x[-60,-52]) and upper: junction x[-60,-44] z[-2,6] ✔ overlaps lower top z[4,6] x[-60,-52] ✔ and upper bottom z[-2,4] x[-52,-44]?? upper x[-52,-42]: overlap x[-52,-44]?? [-60,-44]∩[-52,-42] = [-52,-44] ✔ at z[-2,4] ✔.
- Region between tunnels and mid/B-hall: fill building masses: x[-52,-22] z[-2,26]? that region separates lower tunnels from mid corridor (mid x[-22,-10] z[2,14]) and T ramp... fill x[-52,-24] z[-2,24] h5; fill x[-44,-10]? hmm region x[-44,-22] z[-18,-2] between upper tunnels and top mid: fill. Region x[-36,-10] z[-24,-16]?? between B hall (z[-32,-24]) and CT spawn/cat... fill x[-36,-6] z[-24,-16]?? CT spawn x[-6,16] z[-30,-16]... B hall z[-32,-24] x[-36,-6] shares wall z=-24 with CT spawn? CT spawn z[-30,-16]: B hall z[-32,-24]: overlap z[-30,-24] x[-36,-6]!! B hall x ends -6, CT spawn x starts -6: share face x=-6: B hall z[-32,-24] vs CT spawn z[-30,-16]: overlap z[-30,-24] — the opening x=-6 z[-30,-26] planned ✔ but rest of overlap region z[-30,-24]∩... B hall and CT spawn volumes: B hall x[-36,-6], CT spawn x[-6,16]: they share only the plane x=-6 ✔ no volume overlap. Opening in that plane z[-30,-26] ✔. Wall x=-6 z[-32,-24] minus opening; CT spawn west wall x=-6 z[-30,-16]: opening z[-30,-26] ⊂ [-30,-16] ✔. B hall north wall z=-32 x[-36,-6] ✔, south wall z=-24 x[-36,-6] ✔ (behind it: fill region x[-36,-6] z[-24,-16]? that area is west of CT spawn, south of B hall: fill h4 ✔). And region x[-36,-10] z[-16,-10]?? CT mid x[-10,18]... region x[-36,-10] z[-16,2] between B-hall-south-fill and CT mid: this is west of CT mid — in real dust2, west of CT mid is the mid-doors building ✔ fill x[-36,-8]? CT mid x starts -10: fill x[-36,-10] z[-16,2] h4 ✔ (west of CT mid, includes doors building backside). Doors building: doors wall z=2 x[-22,-10]: doors; wall z=2 x[-10,-8]?? CT mid west boundary x=-10 z[-10,2] ✔ already; mid corridor east wall x=-10 z[2,14] ✔ consistent. Fill x[-36,-10] z[-16,2] also covers doors wall backside ✔. And top mid west wall x=-22 z[-16,2] ✔; region west of top mid x[-44,-22] z[-16,2]: fill (between top mid and upper tunnels) ✔ h5. Top mid north wall z=-16 x[-22,-10] ✔; region x[-22,-10] z[-44,-16]: between top-mid-north-wall and B hall/B site: fill h5 ✔ except B hall x[-36,-6] z[-32,-24] overlaps x[-22,-10] z[-32,-24]!! B hall is a corridor there ✔ exclude: fill x[-22,-10] z[-44,-16] minus B hall → two fills: x[-22,-10] z[-44,-32] and x[-22,-10] z[-24,-16]?? B hall z[-32,-24] ✔ so fills z[-44,-32] & z[-24,-16] for x[-22,-10] ✔. Hmm B hall x[-36,-6] extends east of x=-22... fill regions must avoid B hall rect: use fills: A: x[-22,-6] z[-44,-32]? that extends east to -6 ✔ (north of B hall, south of B site east wall x=-36... region x[-36,-6] z[-44,-32]: between B site north wall (z=-44) and B hall north wall (z=-32) ✔ fill x[-36,-6] z[-44,-32] h5 ✔ covers x[-22,-6] part too ✔ single fill). B: x[-22,-6] z[-24,-16] h5 (south of B hall, west of CT spawn corridor... CT spawn x starts -6 ✔; corridor x[-10,-4] z[-16,-10] is SOUTH of z=-16 ✔ no conflict). ✔.

- T ramp region x[-34,-10] z[24,40]: walls: south z=40 (part of T spawn north? T spawn z[32,44] x[-58,-30]: T ramp x[-34,-10]: overlap x[-34,-30] z[32,40]!! T ramp overlaps T spawn — merge: T spawn x[-58,-30] z[32,44]; T ramp x[-34,-10] z[24,34]; overlap x[-34,-30] z[32,34] shared opening ✔ define: T spawn rect x[-58,-30] z[32,44]; T ramp rect x[-34,-10] z[24,34]; union connects at x[-34,-30] z[32,34] ✔. T ramp east wall x=-10 z[24,34] with opening z[28,32]?? long connector: corridor x[-10,46] z[28,36] y0 ("south connector to long"): opening in T ramp east wall z[28,34]? connector z[28,36]: opening x=-10 z[28,34] ✔. Connector north wall z=28 x[-10,46], south wall z=36 x[-10,46] (south of it fill x[-10,46] z[36,44]? that's map south-east corner region x[-10,64] z[34,48]: fill). Connector east end opens into outside-long x[44,64] z[20,34] ✔ (overlap x[44,46] z[28,34] ✔). Outside-long north: pit x[44,54] z[12,20] floor -0.9 + doors area... outside long rect x[44,64] z[20,34]; pit x[44,54] z[12,20] floor -0.9 adjacent north ✔ opening between them (pit south edge z=20 x[44,54]: drop 0.9, jumpable ✔). Long doors wall z=20 x[54,64] with two door openings as planned ✔. Region north of pit x[44,54] z[2,12]: between pit and cross south wall... cross z[-14,2]: region x[44,54] z[2,12]: fill? corridor x[54,64] z[-12,20] is east; west of pit x[36,44] z[2,20]: region south of cross x[28,44] z[2,12]: fill all: fill x[28,54] z[2,20] minus pit?? pit z[12,20] x[44,54]... fill A: x[28,44] z[2,20] h4; fill B: x[44,54] z[2,12] h4 ✔ (pit z[12,20] excluded) ✔. Wait connector south wall z=36 x[-10,46] and outside-long x starts 44: overlap ✔ fine.
- Corridor x[54,64] z[-12,20] walls: east x=64 (perimeter), west: wall x=54 z[-12,-4]?? west of corridor: site ramp tongue x[54,62] z[-18,-12] north of corridor... corridor west boundary x=54: z[-12,-4]: cross is x[28,56] which includes x[54,56] z[-12,2] — intended OPEN (cross→corridor) ✔ no wall z[-12,2]; wall x=54 z[2,20] (corridor vs fill B/pit) ✔. 
- Long corner slope: ramp x[54,62] z[-18,-12] y0→2.4 ✔ (north end at site level). Corridor z[-12,20] flat y0 ✔.

- Region south of mid corridor x[-22,-10] z[14,24]: between bottom-mid (z[14]? mid corridor z[2,14]... wait I set mid corridor z[2,14] bottom, top mid z[-16,2]. Bottom mid connects T ramp (z=24) to mid: corridor x[-22,-10] z[14,24] IS part of mid corridor: set mid corridor x[-22,-10] z[2,26] (merging bottom), T ramp opening at z=24/26: T ramp z[24,34] overlap z[24,26] x[-22,-10] ✔ opening (no wall there on T ramp north edge; T ramp north wall z=24 for x[-34,-22] and x[-10? corridor...]. Simplify: mid corridor x[-22,-10] z[2,26]; T ramp x[-34,-10] z[24,34]: overlap x[-22,-10] z[24,26] ✔ open.
- Doors at z=2 across corridor ✔. Corridor walls: west x=-22 z[2,26] ✔, east x=-10 z[2,26] ✔.
- Region x[-34,-22] z[2,24]: west of mid, east of lower-tunnels-fill: fill x[-34,-22] z[0,24]? earlier fill x[-52,-24] z[-2,24] (tunnels/mid separator): covers x[-34,-24]... gap x[-24,-22]: fill x[-34,-22] z[-2,24] h5 then ✔ (extend). Combined fill x[-52,-22] z[-2,24] ✔ covers region west of mid corridor & east of tunnels... lower tunnels x[-60,-52] ✔ upper x[-52,-42] z[-18,4]: overlap with fill x[-52,-22] z[-2,24] at x=-52 plane? upper x[-52,-42]: fill x[-52,-22]: share plane x=-52 z[-2,4]?? upper z[-18,4]: overlap z[-2,4] at plane x=-52: junction x[-60,-44] z[-2,6] covers x[-52,-44] z[-2,4]: so opening between junction and upper at... they overlap in volume x[-52,-44] z[-2,4] ✔ open ✔ but fill x[-52,-22] z[-2,24] borders junction at x=-44: junction x ends -44 ✔ wall there ✔ fine. And junction west/south walls: lower tunnels x[-60,-52] z[4,38]; junction x[-60,-44] z[-2,6]: overlap x[-60,-52] z[4,6] ✔ open. Junction east wall x=-44 ✔ (fill). Junction north wall z=-2: opening to ramp x[-50,-44] ✔ rest wall.

- T spawn walls: north z=32? T spawn z[32,44] — perimeter? map bounds z=48: T spawn south = perimeter z=44? make perimeter walls at x=±64, z=±48 and T spawn z[32,44] has south wall z=44 h4 + fill z[44,48]. West wall x=-58 z[32,44] with opening to lower tunnels? lower tunnels x[-60,-52] z[4,38]: overlap x[-58,-52] z[32,38] — wait that's VOLUME overlap (open area) ✔ fine, they're connected spaces; wall segments only where needed. T spawn east wall x=-30 z[32,44]: T ramp x[-34,-10] z[24,34] — opening at z[32,34] x[-30]?? T ramp x starts -34 < -30: overlap region x[-34,-30] z[32,34] open ✔; wall x=-30 z[34,44] ✔.

OK. I've spent enormous effort; the layout is now consistent enough. I'll codify it with the box list and waypoints in code, and it'll be approximately dust2. 

Now waypoints (final, using zones above):
T: ts1(-50,40) ts2(-38,38); T ramp: tr1(-28,29), tr2(-14,29); mid: mb(-16,22), m1(-16,12), mtop(-16,-4), mtop2(-16,-12); cat ramp: catr(-6,-13) [through east wall opening z[-14,-8] x=-10... ramp x[-10,-2] z[-16,-10]?? wait new cat: catwalk z[-18,-12], ramp from top mid: opening in top-mid east wall x=-10 z[-16,-12]?? top mid x[-22,-10] z[-16,2]: east wall x=-10 opening z[-15,-11]?? cat ramp x[-10,-2] z[-18,-12] rising y0(x=-10)→2.4(x=-2) ✔ opening x=-10 z[-16,-12] (in wall x=-10 z[-16,2]... but mid doors wall is at z=2; east wall x=-10 z[2,26] separate ✔; wall x=-10 z[-16,2] with opening z[-16,-12]?? the ramp mouth: z[-16,-12] 4m wide ✔.
cat: cat1(4,-15,y2.4), cat2(16,-15), short(24,-15), site entries...
CT mid: ctm(2,-4), ctmw(-6,-4), xbox(1,-11,y1.2 top) jump links: ctm→xbox (jump), xbox→cat1 (jump: from (1,-11,1.2) to (4,-15,2.4)? distance ~3.6 horizontal... cat west end x=-2: xbox→catwest(-1,-15,2.4): horizontal dist from (1,-11) to (-1,-15) = 4.5 — too far for jump (range ~3m at walk speed?). Make xbox bigger/closer: xbox center (0,-13) size 3x2.4 spans x[-1.5,1.5] z[-14.2,-11.8], adjacent to cat face z=-12?? cat z[-18,-12]: face at z=-12; xbox z[-14.2,-11.8] touches ✔. Jump xbox top (y1.2) → cat (y2.4): need to land on cat at z<-12: jump from z=-12.5ish moving north: horizontal needed ~1m ✔ vertical 1.2 ✔ OK. Waypoints: xbox top wp (0,-13, y1.2); catwest wp (0,-15, y2.4)?? cat x starts -2: (0,-15) valid on cat ✔. Edge xbox→catwest jump ✔. Edge ctm→xbox: jump onto 1.2 ✔.
ctspawn: cs1(4,-22); corridor: cc(-7,-13) (x[-10,-4] z[-16,-10]) links cs1, ctmw.
B hall: bh1(-12,-28), bh2(-28,-28); bdoors_out(-33,-28); bdoors_in(-39,-28); window_out(-34,-36)?? window at x=-36 z[-38,-34]: wp outside (-33.5,-36), inside (-38.5,-36), jump edge ✔.
bsite: bs1(-50,-32), bs2(-44,-24)(car side), bplat(-54,-38,y1.2), btun_in(-47,-22.5,y0.9) (tunnel exit inside site).
tunnels: tl1(-56,30), tl2(-56,10), tj(-52,2), tu1(-47,-6,y0.9), tu2(-47,-16,y0.9) → btun_in.
long: lc1(2,32), lc2(24,32), lc3(40,30), ol(50,28), pit(49,16,y-0.9), ldoor(58.5,19), lcor(58,8), lcor2(58,-4), lcorner(58,-10) → lcslope(58,-15,y1.5?)→ lctop(56,-20,y2.4) site.
cross: cr1(48,-6), cr2(32,0)?? ramp bottom (34,-2), rampmid(35,-9,y1.2), ramptop(35,-13,y2.4)→site.
ctA link: ca(22,-3) links ctm(east), cr2.
site: as1(40,-30,y2.4), as2(52,-36,y2.4), as3(30,-20,y2.4) (ramp top west?), goose(27,-41,y2.4), shortsite(26,-17,y2.4).

Edges & camp spots (defense waypoints with facing): CT camps: goose (face west/cat+long?), site as2 face long corner, ramp top face ramp, cat2 face cat west, ctm face mid doors, bplat face tunnel exit, bs1 face doors/tunnel, window... T camps after plant: pit, long corner, site boxes, cat, tunnel.

Bot logic: T: pick strategy: rush B (all tunnels), split A (2 long 3 cat?), default (spread). Waypoint pathing BFS. Plant at site wp. CT: 2 A (one site one long/cat), 1 mid, 2 B. Rotate when bomb seen/planted.

OK. Map 2 & 3 I'll design simpler but with real structure:

Map 2 "石林镇 Stonehaven" (inferno-like): 
- T spawn south-west. 
- Route 1 (A): 中路 mid north from T, up to "arch" → A site north-center; A short/apps on east: apartments corridor.
- Route 2 (B): "banana" east curving corridor → B site north-east.
- CT spawn north between sites.
Different pace: more enclosed, more corners, vertical: A site on hill y1.5.

Map 3 "货运站 Freight Yard" (trainyard night): industrial, containers, train cars, lanes; sites at north-west (A, on train deck y2) and east (B, warehouse). CQB-mid mix, night lighting.

I'll define these quickly in code with reasonable coordinates (less fidelity pressure).

Alright — coding time. I'll write the file now, being careful with:
- Pointer lock flow
- Fixed timestep-ish loop with dt clamp
- Performance: merged geometry? With maybe 150-250 boxes per map as individual meshes it's fine, but better: merge by texture. I'll merge static boxes into few meshes via BufferGeometryUtils? Not in CDN core... I can manually merge: create BufferGeometry from boxes manually (write my own box-merger producing position/normal/uv arrays). I'll write a simple mergeBoxGeoms function. Textures: canvas-generated (sand wall, plaster, brick, crate, door, floor, metal). One material per texture; boxes grouped by texture → merged mesh per texture. 

- Colliders array separate (including step slabs, fills).
- Raycast: custom vs colliders (fast slab test; few hundred boxes fine per shot at 60hz? shots are infrequent; bot LOS checks each frame per bot ~10 bots * few enemies = fine).
- Particles: simple Points or small mesh pool. I'll use a pool of small boxes/sprites with velocity for blood/dust/sparks, and shell casings.
- Tracers: THREE.Line segments pool or thin stretched box; use additive line segments updated briefly.
- Smoke: sprite planes with canvas radial texture, ~10 per smoke, fade in/out, 16s life.
- Muzzle light: one reusable PointLight moved to muzzle on shots.

Bot models: build humanoid from boxes: torso, head, legs, arms, gun box; team colors: T = tan/desert camo, CT = blue/black. Animate legs swing when moving (simple sin), aim arm toward target. Death: fall over (rotate group) fade after 5s, drop weapon pickup (weapon box on ground, E to pick up? player walks over to pick if no weapon in slot — implement pickup by walking over + press E? auto-pickup if slot empty... CS uses walk-over + G drop; implement: walk over weapon auto-picks if you have none of that slot; E swaps). Simplify: walking over a dropped weapon picks it up if slot empty or same... and dropping current. Player drop weapon key G. Requirement mentions 掉枪 (drop on death) ✔ and 保枪 (save).

Bomb carrier indicator. 

Player entity shares "Actor" structure with bots: {pos, vel, yaw, pitch, hp, armor, helmet, team, alive, weapons:{primary,secondary,knife?}, currentSlot, nades[], money(bots too for realism? bots get money but auto-buy), isBot}.

Bot buying: auto by money: full armor, rifle (ak/m4), nades on hard.

Sounds: WebAudio functions: shot(type), step(), reload(), explosion(), flashbang(), beep(), plantTick, defuse, uiClick, hitmark, headshot ding, roundStart horn? Use noise buffers + oscillators. Distance attenuation: simple gain by distance.

Round states: 'freeze','live','planted','roundend','matchend'. Buy allowed during freeze + first 15s? CS: buy time 20s. I'll allow buy during freeze & buytime (18s) within buy zone (spawn areas).

Match: halfLength 8, winScore 9, maxRounds 16? If 8-8, next wins? just play to 9 (could exceed? 8-8 → 17th round? I'll cap: if round 16 ends 8-8, draw). Fine.

Side switch at half: swap teams, reset money to 800, reset weapons.

Spectate: on death, camera follows teammate (cycle with mouse click or space). If none alive, free cam? just fixed overhead. Also killcam skip.

Kill feed: top-right list.

Damage: apply with hitbox: bots have head box (head part) — ray test: check head sphere first (r=0.18 at head pos), then body capsule/AABB. Headshot x4.

Player hit: screen red vignette + direction indicator.

Flash effect: white overlay opacity by intensity, decays. Bots blinded: skip targeting while blind.

Smoke LOS: segment vs smoke spheres → blocked for bots; player visual.

Molotov: on ground: fire radius 2.2, dps 40, 7s, particles; bots avoid: when pathing near fire, add repulsion.

HE: fuse 1.6s after throw; damage by distance + LOS.

Now the buy menu DOM: categories column; items with price; updates affordability (gray out). Keys 1-6 categories then items? Simpler: single panel with sections, number keys 1-9 per visible list? I'll do clickable + hotkeys per item shown.

Scoreboard: Tab: table both teams, K/D/money, ping? no.

Radar canvas: 180px top-left. Draw walls from colliders (top-down rects) scaled; rotate? fixed north-up; player arrow center? CS radar is fixed map with icons. I'll do fixed map scaled to fit, with icons.

Hitmarker: 4 lines CSS animated.

Let me also handle "购买阶段" UI hint, round banner center text, "炸弹已安放" etc in Chinese.

Restart/pause: Esc toggles pause (exit pointer lock). Buttons: 继续， 重新开始本图， 返回主菜单.

Testing: I can't run a browser here. I must be extra careful with syntax. I'll write clean code and mentally check. Maybe I can run node to syntax-check the JS by extracting? The file is HTML; I can copy JS into a .js and run node --check. I'll do that if feasible (no DOM at parse time issues — node --check only parses, doesn't run; fine as long as no import of three via CDN at parse — node --check just syntax checks, references undefined ok).

Let me write the code now. I'll aim for well-organized, dense code. Given message length limits, I'll write the whole thing in one go, being disciplined.

Key global structure:

```js
// ============ UTIL
// ============ AUDIO
// ============ TEXTURES (canvas)
// ============ MAP DATA (dust2, stonehaven, freight)
// ============ MAP BUILD (meshes merged, colliders, nav)
// ============ INPUT
// ============ WEAPONS
// ============ ENTITIES (player, bots, nades, particles)
// ============ AI
// ============ ROUND/ECONOMY
// ============ HUD/UI
// ============ MAIN LOOP
```

Let me think about the physics step function:

```js
function actorMove(a, dt){
  // a: {p:Vector3 (feet), v:Vector3, r:0.35, h: current height, onGround}
  a.v.y -= 22*dt;
  // horizontal
  tryMove(a, a.v.x*dt, 0);
  tryMove(a, 0, a.v.z*dt);
  // vertical
  a.p.y += a.v.y*dt;
  // ground support
  let sup = supportHeight(a);
  if(a.p.y <= sup){ a.p.y = sup; a.v.y = 0; a.onGround=true; } else a.onGround=false;
  // head collision
}
function tryMove(a, dx, dz){
  a.p.x += dx; a.p.z += dz;
  for(const c of colliders){
    if(overlapXZ(a,c) && a.p.y + 0.05 < c.y1 && a.p.y + a.h > c.y0){
      // step up
      if(c.y1 - a.p.y <= 0.55 && a.onGround && headroomFree(a, c.y1)){ a.p.y = c.y1; continue; }
      // push out along min axis
      ...resolve
    }
  }
}
```
Support height: max over colliders containing (x,z) footprint center (use center point + radius sample?) with c.y1 <= a.p.y + 0.55 of c.y1; also ground y=0 (or map-specific base). Use center point only — simpler and fine with 0.35 radius? Edges: player could float half over ledge; acceptable.

Push-out resolution: compute overlap on x and z, push along smaller.

Ramps as slabs: slab count = rise/0.25.

Bullets raycast: function rayWorld(o, d, maxDist) returns {t, point, normal} min over colliders; also ground plane y=0 handled by a big ground collider box (y[-1,0] full map) ✔ just add ground as collider.

Pit floors: ground plane covers whole map at y=0; pit floor at -0.9 needs its own floor box and ground must not fill pit — but ground box y[-1,0] spans everything including pit (top y=0 > pit floor -0.9 → pit inside ground!). Fix: ground handled per-map as several ground boxes excluding pit areas; or support function uses map.groundRects list: array of {x0,z0,x1,z1,y}. supportHeight = max over rects containing point of rect.y where rect.y <= py+0.55, plus collider tops. Bullets: ray vs ground rects (as thin boxes y[rect.y-0.5, rect.y]) — generate thin collider boxes for each ground rect (mesh visual = single big ground plane though; pits need holes in visual ground... visual: build ground from the rects as thin boxes with floor texture — merged, fine! So ground visuals = rect boxes.)

Dust2 ground rects (y=0 unless noted):
Full base: I'll compose from region list:
- Main y0 regions: everything except pit & tunnels-upper & platform & cat & site fills.
- Easiest: base ground rect whole map y=-0.9?? no.
List approach: one big rect y0 x[-64,64] z[-48,48], then pit rects at y-0.9 would be UNDER the y0 rect → support takes max → y0 wins in pit ✘. So must exclude pit from base: carve: base split into rects around pit x[44,54] z[12,20]:
- r1: x[-64,64] z[-48,12]
- r2: x[-64,44] z[12,48]
- r3: x[54,64] z[12,48]
- r4: x[44,54] z[20,48]
- pit floor: x[44,54] z[12,20] y=-0.9
- pit walls: boxes at edges h0.9 (x=44 face etc.) — the r-carve creates vertical faces automatically? No—rects are flat; pit needs side walls: add 4 thin wall boxes around pit y[-0.9,0]. ✔
Also upper tunnels floor y0.9: rect x[-52,-42] z[-20,4] y0.9? But base r1 covers z[-48,12] at y0 → max(0.9,0) fine ✔ (0.9 wins, and base y0 below is hidden—visual overlap z-fighting? The 0.9 rect renders above ✔). Ramp junction: slabs handle via colliders; visuals: ramp mesh.
- B platform: fill box h1.2 (box, not ground rect) ✔.
- Catwalk & site & fills: boxes ✔ walkable tops via collider tops ✔ (support fn uses collider tops too ✔).

Good: supportHeight(a) = max( groundRects.y (containing), colliders.y1 (containing & y1<=py+0.55) ) but must only consider colliders whose y1 <= py+0.55 (can't teleport on top of a 2m box) ✔ and standing check for headroom... fine.

Vertical: if v.y<0 and py<sup → land. If v.y>0, head hit: check colliders where py+h in [c.y0,c.y1] & XZ overlap → v.y=0.

Jump: v.y=6.2 → h=6.2²/44≈0.87?? g=22: h=v²/2g=38.4/44=0.87. Too low for 1.2 boxes! CS jump ~0.7? CS jump height ~55 units (~1.05m?) hmm. I need jump onto 1.2 xbox + then 2.4 cat... 1.2 requires v: h=1.2+ → v=sqrt(2*22*1.35)=7.7. That's floaty-high. Adjust: make xbox h=0.9, cat y=2.1?? Still high. CS handles via crouch-jump. For bots & simplicity: set jump v=7.2 (h≈1.18), gravity 23. Slightly floaty but enables flow. Player crouch-jump adds height? skip. Set xbox h1.0, cat 2.2: jump 1.18 > 1.0 ✔; xbox(1.0)→cat(2.2): need 1.2 > 1.18 ✘. cat y=2.1: 1.1<1.18 ✔. Set plateau heights 2.1 instead of 2.4. Cat 2.1, site 2.1, xbox 1.0, platform B 1.0, boost boxes 0.7. Long slope rise 2.1. OK.

Fall damage: skip (heights ≤3).

Bot steering: pos → waypoint; waypoints store y (auto-compute support at build? I'll compute wp.y at runtime when bot arrives / for link jump detection: store y in definition for elevated ones; bots set their target xz; ground follow automatic via physics).

Path: BFS on links. Recompute on strategy start / when stuck / every few seconds toward objective.

Combat engagement: each bot every 0.1s: find best visible enemy (dist < 60, FOV 170°? while guarding use facing 120°), LOS clear (ray vs colliders AND smoke check). If found: state=engage; reaction timer by difficulty (easy 700ms, normal 450, hard 280 + skill variance); aim at enemy chest/head (hard aims head more); error angle decreasing with time-on-target; fire respecting weapon rpm, burst patterns (rifle: 3-4 round bursts with pauses; awp: scoped slow); move: strafe perpendicular randomly or crouch-stand still when holding.

If heard enemy (footsteps within radius or shots): turn toward.

Objectives (T): if bomb not planted: if bot has bomb & at site zone → plant channel; else move per strategy waypoint chain; after plant: camp site.
CT: if bomb planted & site known: move to site, defuse when close (channel, kit halves); else hold camp wps; rotate if bomb seen.

Nades: on strategy start (hard): throw smoke toward choke wp when passing; HE when enemy known position & dist 8-25 & cooldown; flash before entering site (hard only). Implement simple: bot throws nade with velocity toward target point arc.

Defuse/plant channels with progress; interrupted by damage? CS: damage doesn't interrupt; moving cancels. Keep: leaving radius cancels.

Bomb plant zone check via site rects.

Now UI text in Chinese. Let's write.

Also weapon viewmodels: build simple but decent box-models:
function buildViewmodel(wid) returns THREE.Group with parts; materials dark metal, wood for AK etc. Attach muzzle empty for flash position. Add subtle idle sway + recoil kick animation (group.position.z += kick, recovers).

World gun models for bots: same builder scaled? Just build small gun boxes attached to bot arm.

Dropped weapons: group on ground, spin? static; pickup radius 1.2.

Knife: implement as slot 3 weapon melee: raycast 1.8m damage 40.

Slots: 1 primary, 2 secondary, 3 knife, 4 HE, 5 flash, 6 smoke, 7 molotov? CS uses 4 cycle nades; I'll map keys 4-7 direct. Bot nade usage simplified.

Buy menu items: 
手枪： Glock(T)/USP(CT) 200? Actually default free; buy P250 300, Deagle 700, Dualies? skip.
霰弹： Nova 1050, XM1014 2000
冲锋： MP5 1500, P90 2350
步枪： Galil 1800(T)/FAMAS 2050(CT), AK 2700(T)/M4 3100(CT)
狙击： SSG08 1700, AWP 4750
装备： 防弹衣 650, 防弹衣+头盔 1000, 拆弹器 400(CT), 手雷 300, 闪光 200, 烟雾 300, 燃烧瓶 400(T)/500(CT)
Team restriction: AK T-only, M4 CT-only etc. If player T: show AK not M4.

Economy awards as planned.

Sounds synth details:
- shot: buffer noise 0.15s through lowpass 1800Hz + gain env exp decay; add body thump: sine 120Hz decay; vary per weapon (rifle bigger, awp huge w/ echo?).
- distant shots: lowpass freq by distance, gain by distance.
- step: noise 0.05s lowpass 400 gain 0.15.
- reload: two clicks timed.
- explosion: noise 1s lowpass sweep 3000→100 + sub sine 45Hz decay 0.8.
- flash: high sine 3500Hz ring 2s quiet + pop.
- smoke pop: whoosh noise bandpass.
- beep: square 1000Hz 0.07s.
- plant/defuse: soft clicks.
- round start: short brass? just beep sequence.

Radar: draw from map.radarRects (I'll reuse colliders with h>1.2 & area, as dark rects on tan bg) + sites letters A/B + icons.

OK writing now. This will be long (~3500-4500 lines). Also need match flow: after map/side/difficulty selection → startMatch(config) → build map, spawn actors, begin rounds.

Spectate cam: copy teammate camera transform each frame.

Player death: drop weapons (spawn dropped entities), show "你已阵亡" + spectate prompt; click cycles target.

Round end detection each frame.

Freeze time: 3s countdown; players can't move (vel zero) but can look/buy.

Buy time: 15s from round start; buy menu open with B (auto-open at round start for player).

Timer: 105s round; bomb timer 40s; plant 3.2s; defuse 10/5s.

Also handle player choosing CT: bots fill T 5, CT 4 bots + player.

Team sizes: config.teamSize (1-5). T side gets bomb → if player is T, player carries; else random T bot.

Alright, also doors visual: door texture boxes. Arches: lintel boxes.

Extra visual: sky color per map, fog, directional light + hemisphere; shadows? Shadows expensive with merged geo — enable simple: renderer.shadowMap off; use baked-ish ambient via vertex? Skip shadows; use hemisphere+dir light, fine.

Let me write the final code. I'll be careful about typos. Because of length, I'll keep comments minimal.

Plan the collider representation: {x0,y0,z0,x1,y1,z1} plus helper addBox(cx,cy? ,cz,w,h,d, opts) where cy = bottom y.

Map def format: functions that push into arrays: 
```
M.boxes: visual+collide boxes {cx,cz,w,d,h,y(bottom),tex, ry? (skip rotation except ramps)}
M.ramps: {cx,cz,w,d,y0,y1,axis,dir} → slabs colliders + visual rotated plane
M.ground: [{x0,z0,x1,z1,y,tex}]
M.fills → just boxes with wall tex
M.spawns {T:[[x,z],...], CT:[...]}
M.sites {A:{x0,z0,x1,z1}, B:{...}}
M.waypoints: [{x,z,y?,l:[indices],jump? per-edge? }]
```
Edge jump flags: store on links as "a-b!"? I'll store links as array of [targetIdx, jump?] pairs. Simpler: links: [idx...], jumpLinks: [idx...].

Nav BFS fine.

Radar: from boxes with h>=1.5 → rects; draw scaled.

Let me now write everything. I'll write in one pass, then syntax check with node.

Also remember: pointer lock mouse-look, FOV 75, ADS fov 40 (scope 15 with overlay).

Crosshair: CSS 4 lines + center gap var by spread.

Weapon switching animation: lower/raise timer 0.4s.

Reload: timer per weapon; R key.

Ammo HUD. Buy zone hint.

Given enormity, begin writing. I'll write to a single file `index.html` plus `README.md`.

Let me carefully write the JS. I'll structure with a big script. Let me draft key numbers:

Player: height 1.75, crouch 1.25, radius 0.35, eye 1.62/1.15, speed: run 5.0 (rifle), knife 5.6, awp 4.6, shift-walk x0.52, crouch x0.34, air control low. Acceleration: ground accel fast (friction-based): simple: vel = lerp(vel, wish*speed, 1-exp(-10dt)) ground; air: wish adds small.

Recoil: pitch kick per shot (rifle 0.7°, awp 3°), recover 8/s; bloom adds spread.

Spread model: base per weapon + move penalty + air big + crouch bonus; degrees.

Damage falloff: dmg * clamp(1 - (dist-15)*0.006, 0.55, 1).

Armor: dmgToHp = dmg * armorPen (if armored body part); armor absorbs (dmg - dmgToHp)*0.5 reduce armor points.

Bots count ~9 + player: 10 actors. OK.

Now bot visual model builder: 
```
function buildSoldier(team): group with:
 - legs L/R (0.22x0.75x0.22) at ±0.11
 - torso 0.5x0.6x0.28 y0.75-1.35
 - head 0.26 cube at 1.5 + helmet box
 - arms: right arm holding gun forward
 - gun box
 team colors: T: #8a6b3c torso, #6b5230; CT: #3a4a63 torso, #2c3a50; head skin.
```
Animate: legs.rotation.x = sin(t*speed)*0.6 when moving; group faces yaw; slight bob.

Death anim: rotate group to side over 0.3s, sink after 6s remove; drop gun.

Blood: red particles.

Now, bot shooting accuracy: aim point = enemy chest (p.y+1.2); error radius: base (easy 0.09 rad? let's use meters at distance): err = (easy 0.9, normal 0.55, hard 0.3) * (moving?1.6:1) * (1 + dist*0.02)... plus first-shot after reaction bigger, shrinking over 1s. Hit determination comes naturally from ray spread.

Bot fire rate control with weapon def rpm & burst.

Damage to player: red flash + hp bar.

Knife for bots: if dist<1.6 melee.

Bots avoid fire: sample next waypoint direction; if fire within 1.5 of path point, offset perpendicular.

OK — also "中门/双门" peek: door gap 0.35 allows bullets ✔ ray passes.

Time to write. I'll now produce the final file content. I need to be really careful with matching braces. I'll keep functions compact.

Let me also decide bot count default: 5v5 (player+4 vs 5).

Also implement "eco round" behavior for bots: if money < 2000, save (buy only pistol/armor?) — simple auto-buy: priority: armor+helmet if > 900, rifle if afford after armor else SMG else keep pistol; nades if hard & money > 5000.

Player kept weapons on round win/survive; lose on death (drop). Standard.

Round restart: reset positions, hp, ammo? CS: weapons kept, ammo kept; buy fills. Reset nades? kept. Bomb given to random T.

Match end: show final scoreboard + MVP + 返回菜单.

I'll write the code in the artifact now. Given the size, I'll go section by section and be careful.

For Three.js CDN: use r128 from cdnjs (reliable, non-module): https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js — global THREE. Good (no modules hassle).

Now write. I'll try to keep the whole thing correct. Special attention areas:

1. rayAABB: 
```
function rayBox(o,d,b){ let t0=0,t1=Infinity; for axes... slab; return t0 if hit }
```
Implement with arrays.

2. LOS: rayBoxes returns earliest t; losBlocked(a,b): dir=b-a; t=rayWorld < dist-eps → blocked. Also smokeSpheres check: segment-sphere intersect.

3. Grenade sim: vel, gravity 18; collide: integrate; for each collider: if point inside expanded box → push out along min penetration axis, reflect velocity component *0.45, tangential *0.8; if speed<0.5 rest. Ground rects as boxes included (thin). Then timer-based effects.

4. Smoke blocks bot vision: smokeBalls [{p, r:2.6, until}]; segment intersect sphere test.

5. Flash: for actor: intensity = f(dist, angle between actor view dir and grenade dir, LOS). white overlay for player; bot.blind = duration.

Now writing the HTML skeleton:

- #menu (title, buttons: 开始游戏 → setup panel: map cards (canvas previews), side select, difficulty, team size, 开始）
- #hud: crosshair, hitmarker, health/armor, ammo, money, timer, score, radar canvas, killfeed, round banner, buy hint, progress bar (plant/defuse), damage vignette, flash overlay, scope overlay, spectator bar, interact hint, nade slots, weapon slots
- #buymenu
- #scoreboard
- #pause
- #mathend

Fonts: system, bold military style.

Alright, final code. I'll write it now completely, then verify syntax via node --check by extracting script.

Given how much code this is, let me write very carefully and consistently. Begin.

Also map preview thumbnails: function drawMapPreview(canvas, mapDef) reuse radar drawer. 

Waypoint list for dust2 — I'll finalize indices when coding. I'll write waypoints as array of {x, z, y?, l: [i...], j: [i...] (jump links), camp?: {yaw}}. I'll compute y at build if undefined (support height + 0.1).

Strategies for T: list of waypoint routes:
- "rushB": tunnels chain
- "splitA": long & cat
- "midToB"? no mid-to-B. Strategies: A长 (long heavy), A小 (cat heavy), B洞rush, 默认 (spread then hit).
CT setups: 3A2B / 2A1mid2B etc.

Bot decision: pick random strategy at round start; assign roles.

Defuse kit for CT bots hard difficulty.

Let me write the actual code.

(Writing...)

I'll now carefully compose the entire file. To manage risk of truncation, I'll write efficiently but completely — target ~2600-3300 lines. Let me go.

Some final decisions to keep code manageable:
- No crouch for bots (except holding? skip).
- Bots jump only on flagged edges.
- One shared geometry cache.

Also colliders count dust2: boxes ~140 + slabs (ramps: cat ramp 0→2.1 over 8m = 9 slabs; site ramp tongue 6 slabs; A ramp 10 slabs; junction ramp 4; platform ramp 5; long corner 9... total ~50 slabs) fine.

Radar: precompute rects.

Write now. I'll do my best to keep it bug-free. After writing, I'll do a mental review pass + node syntax check.

Actually, since I can't be 100% sure node is available in this environment, let me just try. I'll create files:

- /project/index.html
- /project/README.md

Let me write the code.

Key subtle bug areas to double check while writing:
- Array-based ray vs colliders loop.
- Actor collision push-out: implement per axis move then check all colliders:
```
function collideAxis(a){ // after moving, resolve all overlaps
 for c in colliders:
   if overlap(a.cbox, c):
     if step-up possible: a.p.y=c.y1; continue
     push out min axis
}
```
I'll implement move X then resolve, move Z then resolve (classic).

- Support height check each frame after Y move.

Waypoints: bot target wp pos = (wp.x, wp.y, wp.z); reach when horizontal dist < 0.5.

Bot path following with smoothing: head to path[i]; if i+1 exists and LOS straight, skip.

Stuck: track pos every 0.5s; if moved <0.3 while trying → jump + repath.

OK writing the file.

For the buy menu structure:

```
sections = [
 {t:'手枪', items:['p250','deagle']},
 {t:'霰弹枪', items:['nova','xm1014']},
 {t:'冲锋枪', items:['mp5','p90']},
 {t:'步枪', items:[ side=='T'?'galil':'famas', side=='T'?'ak47':'m4a4']},
 {t:'狙击枪', items:['ssg08','awp']},
 {t:'装备', items:['vest','vesthelm','kit'(CT),'he','flash','smoke','molotov']},
]
```
Numbers 1-6 select section? Simplest: all visible grid; click to buy; hotkeys shown as numbers across full list maybe skip hotkeys except B close. I'll add hotkeys: each item index key displayed (1..9,0..) per section row: row key = section number, then item number? Two-press: press 1 → highlights pistols, then 2 → buys deagle. Implement two-stage buy keys. Also auto-buy F1? Add "自动购买" button.

Alright, writing!

One more: weapon defs table with all fields:

```
W = {
 knife:{n:'匕首',slot:3,dmg:40,rate:1.2,range:1.9,melee:true,price:0,spd:5.6}
 glock:{n:'Glock-18',slot:2,price:200,dmg:28,pen:0.47,hs:4,rate:400,mag:20,res:120,rel:2.2,spr:0.7,kick:0.5,spd:5.4,auto:false,snd:'pistol',kill:300}
 usp:{...dmg:34,pen:0.5,rate:352,mag:12,res:100,rel:2.2}
 p250:{dmg:38,pen:0.64,rate:400,mag:13,res:26,price:300}
 deagle:{dmg:53,pen:0.932,rate:267,mag:7,res:35,price:700,kick:1.6}
 mp5:{slot:1,dmg:26,pen:0.625,rate:750,mag:30,res:120,price:1500,auto:true,kill:600}
 p90:{dmg:25,pen:0.69,rate:857,mag:50,res:100,price:2350,kill:300}
 nova:{slot:1,shotgun:true,pellets:8,dmg:12(pellet),pen:0.5,rate:68,mag:8,res:32,price:1050,kill:900,pump:true}
 xm1014:{pellets:7,dmg:11,rate:200(auto),mag:7,res:32,price:2000,kill:900}
 famas:{dmg:30,pen:0.7,rate:666,mag:25,res:90,price:2050}
 galil:{dmg:30,pen:0.775,rate:666,mag:35,res:90,price:1800}
 ak47:{dmg:36,pen:0.775,rate:600,mag:30,res:90,price:2700,kick:0.85}
 m4a4:{dmg:33,pen:0.7,rate:666,mag:30,res:90,price:3100}
 ssg08:{dmg:88,pen:0.85,rate:50(bolt delay 1.2),mag:10,res:90,price:1700,scope:2,kick:2}
 awp:{dmg:115,pen:0.975,rate:41(1.45s),mag:5,res:30,price:4750,scope:2,kill:100}
 he:{nade} flash smoke molotov
}
```

Alright. Writing the file for real now.

I also need muzzle flash world light & viewmodel flash quad.

I'll manage code size by disciplined helper reuse. Let me finally produce it.

After writing, I'll extract JS and run `node --check`.

Let me write out the file in my response. It'll be very long; that's expected and required.

While writing map builder: texture list: 'sand' (yellow plaster), 'sand2' darker, 'floor' (concrete), 'crate' (wood), 'crateMetal', 'door', 'brick', 'metal', 'roof', 'stone'(map2), 'wood'(map2), 'container'(map3 colors), 'asphalt'(map3). Generate each as 128x128 canvas with noise + patterns.

Merged geometry: for each box, compute 24 verts transformed; push into per-texture arrays; create BufferGeometry with position/normal/uv; mesh with MeshLambertMaterial({map}).

Ground rects as thin boxes too.

Ramps visual: build as custom quad strip following slope — or just use slab boxes visual too (stairs look acceptable-ish for ramps? A ramp as stairs looks off). I'll generate smooth ramp visual: a box rotated: geometry BoxGeometry(w, h, d) rotated around x or z axis positioned so top surface matches slope; plus it will intersect ground (fine). Colliders = slabs (step-up handles 0.25 steps). Visual angle: length L along axis, rise R: angle=atan2(R,L). Position center at midpoint, y = (y0+y1)/2 - small. Also add side skirt? fine.

Now bot names: array of names.

Kill feed: "A 武器 B".

Let me write.

Also player spawn protection? none.

Also "保枪" — happens naturally when surviving.

Also drop bomb on death → pickup by T walking over (auto). Bomb icon on radar.

Bomb explosion: kill within radius 25 (damage falloff, through walls), round to T. Camera shake.

Alright, code:

I'll now write the complete index.html. Careful bracket discipline. Here goes.

(Note: in the interest of actually finishing, I'll write compact functions and avoid over-engineering.)

Let me draft some core functions inline as I write.

I'll write it out now in the tool call as a file creation. Then run node --check on extracted JS if possible. Since I can't actually extract easily, I could instead write game.js separately and include via <script src>? But single-file requirement-ish... The user said complete project; multiple files allowed ("项目结构由你自行判断"). I'll do index.html + game.js + README.md. That lets me node --check game.js directly. 

Run instructions: needs a static server (python -m http.server) because of pointer lock? Pointer lock works from file:// generally? Canvas textures fine from file. Three.js CDN needs internet. I'll note: open via local server or double-click (CDN requires network). Everything else local.

Let me write game.js sections:

1. Const & utils
2. Audio
3. Textures
4. Weapon defs
5. Map defs (dust2, stonehaven, freight) — biggest chunk
6. World build
7. Input
8. Player
9. Combat
10. Grenades
11. Bots
12. Round manager
13. HUD
14. Menus
15. Loop

Now writing for real. I'll allocate: dust2 def ~350 lines, other maps ~200 each, engine ~2000 lines.

Final check on required dust2 features mapping:
- T出生点 ✔, CT出生点 ✔, A包点 ✔, B包点 ✔, 中路 ✔, A大（long) ✔, A小（cat) ✔, A坑(pit) ✔, A斜坡(ramp) ✔, 中门/双门 ✔, Xbox ✔, B洞 ✔, B门 ✔, B平台 ✔, 警家（CT spawn/hall) ✔, 长门 ✔, 箱体掩体 ✔ (site boxes, blue box, crates), 狙击线 ✔, 下包点 ✔, 防守架点 ✔ (camp waypoints), 进攻路线 ✔, 回防路线 ✔ (CT spawn→both sites), 绕后路线 ✔ (tunnels flank, long flank).

Waypoint labels for bot callouts not needed.

Begin writing the actual files.

For bot AI structure:

```
bot = {actor fields, ai:{state, path[], pi, target, reactT, aimErr, burstN, nextShot, holdUntil, campWp, role, strategy, nadeCd, blindUntil, lastSeen, plantT, defuseT, repathT, stuckT, lastPos}}
```

Bot think(dt) at 10Hz per bot (stagger).

States: 'execute' (following role route), 'engage' (seen enemy), 'plant', 'defuse', 'camp' (hold), 'retake'.

I'll write think logic:

```
if blinded: stumble.
scan enemies → if visible: lastSeen update; if reactT passed → combat = enemy
if combat enemy alive:
   face enemy with error; fire per weapon; maybe strafe; if lost sight >2.5s → clear combat, go lastSeen
else follow objective:
   T: if bombPlanted: camp near site (pick camp wp) ; if hasBomb & in site: plant channel ; else follow route to assigned site
   CT: if bombPlanted: if at site & near bomb: defuse; else path to site ; else: go to assigned camp wp, face camp yaw; if heard/seen bomb carrier → rotate
movement along path with jump edges
nades occasionally
```

Fire control: if aimAngleDiff < threshold and dist ok: shoot via same fireWeapon function (shared for player & bots) with bot spread.

fireWeapon(shooter, weapon, origin, dir) → raycast vs world & actors (excluding shooter team? friendly fire off), apply damage, spawn tracer/impact, sound.

Also recoil for bots irrelevant.

Hearing: shots register noise events with pos & radius; bots store heardPos if within radius.

OK. Writing now, finally. I'll produce the files.

Actually one more consideration for ray vs actors: represent actor hitbox as: body AABB centered (x, p.y+0.9, z) half (0.35, 0.9, 0.35); head sphere center (x, p.y+1.55, z) r 0.22 (crouch adjust). Ray-sphere test first; then ray-AABB body. Return closest & part.

Damage falloff + armor + headshot.

Player HUD damage direction: skip direction, just vignette.

Alright. Writing files now. This is going to be a huge output. Let me be systematic and complete.

I'll also include a simple FPS counter? skip.

Let me carefully write the Dust2 box list from my plan. I'll use helper in map code:

```
const D=(cx,cz,w,d,h,y=0,t='sand')=>({cx,cz,w,d,h,y,t});
```

Dust2 boxes list (from plan, heights: plateau 2.1, cat 2.1, xbox 1.0, platform 1.0):

Perimeter: 4 walls h6: 
 (-64? use thickness 2) N: (0,-49, 132,2,6) z=-49; S:(0,49,...); W:(-65,0,2,100,6); E:(65,0,2,100,6). (map bounds x±64 z±48)

T spawn region x[-58,-30] z[32,44]:
 walls: W perimeter covers x=-58? perimeter at x=-65: T spawn west wall: box(-58,38,1,12? ...). Let me define T spawn walls:
 - south: (-44,44, 28,1,4) covering x[-58,-30] ✔
 - west: (-58,38, 1,12, 4) z[32,44] but opening to tunnels z[34,40]?? tunnels lower x[-60,-52] z[4,38] overlap T spawn x[-58,-52] z[32,38] — that's volume overlap → they're connected regardless of this wall... The west wall x=-58 would cut through the overlap. So NO wall between T spawn and lower tunnels there: skip west wall segments z[32,38]; wall (-58, 41, 1,6,4) covering z[38,44] only. And lower tunnels area west wall = perimeter-ish: box(-60.5? ...) tunnels west wall: (-61, 21, 1, 34, 4) z[4,38]?? x=-61 wall along tunnels west side ✔. Tunnels south wall: (-56,38.5, 10,1,4) covering x[-60,-52] at z=38... but T spawn extends z[32,44] x[-58,-30]: overlap region with tunnels x[-58,-52] z[32,38]: tunnels south wall z=38 would cut T spawn... T spawn occupies z up to 44 there. Just make tunnel entry explicit: opening at z=38? The overlap means T spawn and tunnels share space x[-58,-52] z[32,38] — treat as one area, wall off its south at z=44 (T spawn south wall covers x[-58,-52]? T spawn south wall (-44,44,28,1,4) covers x[-58,-30] ✔ includes [-58,-52] ✔) and west x=-61 wall z[4,38]... gap x[-61,-58]?? walls thickness: use x=-60.5 for tunnel west wall; T spawn west edge x=-58 needs wall z[38,44]: box(-58,41,1,6,4) ✔; region x[-61,-58] z[38,44]: pocket west of T-spawn wall & north of tunnel-south?? tunnel south wall at z=38 covers x[-60,-52]... region x[-61,-58] z[38,44] is enclosed pocket → fill with box(-59.5,41,3,6,4). OK.

This is extremely fiddly; accept minor imperfections (extra pockets get filled).

Let me just systematically define fills to cover ALL unassigned space — easier: after defining playable rects, fill complement? Too complex manually. Alternative pragmatic approach: define the map as positive space (playable areas) and surround each corridor with explicit walls; small gaps may exist but perimeter contains everything; bots use waypoints so unaffected; player might find voids — mitigate by generous fills.

I'll write the list carefully region by region, accepting ~150 boxes.

Let me write dust2 def in code order:

1) Perimeter (4)
2) T spawn walls + fill
3) Lower tunnels walls/ceiling
4) Junction + ramp + upper tunnels walls/ceiling + exit arch
5) Fills: west-mid separator x[-52,-22] z[-2,24]; tunnels-vs-topmid x[-44,-22] z[-18,-2]; north-west big fills x[-36,-6] z[-44,-32] & x[-22? reuse]; x[-36,-6] z[-24,-16]; x[-36,-10] z[-16,2]... wait upper tunnels x[-52,-42] z[-20,4]: fill x[-52,-22] z[-2,24] overlaps upper tunnels x[-52,-42] z[-2,4]!! Conflict: upper tunnels z[-18,4] vs fill z[-2,24]: overlap z[-2,4] x[-52,-42] ✘. Adjust fill: x[-42,-22] z[-2,24] (east of upper tunnels) + x[-52,-42] z[4,24] (south of upper tunnels) + x[-52,-42] z[-2? junction is x[-60,-44]: between junction(x ends -44) and fill... upper tunnel east wall x=-42 z[-18,4]: fill x[-42,-22] z[-2,24] covers east ✔; region x[-44,-42]?? junction x[-60,-44]: gap x[-44,-42] z[-2,4]: fill small box. And upper tunnel vs top-mid: top mid x[-22,-10] z[-16,2]; fill x[-42,-22] z[-18,-2] ✔ separates.

Continue list... I'll finalize in code with care. Given the volume, minor overlaps are OK (boxes overlapping visually fine; colliders overlapping fine).

6) Mid corridor walls x=-22 z[2,26] (box(-22.5,14,1,24,5)), x=-10 z[2,26]?? east wall x=-10 z[2,26]: but bottom-mid opening to T ramp z[24,26]: wall x=-10 z[2,24] ✔ box(-9.5,13,1,22,5) z[2,24].
   West wall x=-22: opening?? none ✔ box(-22.5,14,1,24,5) z[2,26]?? T ramp overlap z[24,26] x[-22,-10] open ✔ so west wall z[2,26] box z-center 14 len 24 covers z[2,26] — at z[24,26] x=-22 that's the T ramp west boundary at that segment ✔ fine (T ramp x starts -34).
7) Doors at z=2: frames: box(-21,2,2.2? let me: wall pieces: box(-21,2, 2,0.8, 4.5) x[-22,-20]; door panels: doorA box(-16.8,2, 3.4,0.5, 3.6,'door') x[-18.5,-15.1]; doorB box(-12.4,2, 3.4,0.5,3.6,'door') x[-14.1,-10.7]; gap between: x[-15.1,-14.1]=1.0m?? Too big — bodies pass! 1.0 > 0.7 width... make gap 0.5: doorA x[-18.5,-15.25], doorB x[-14.75,-10.7]? gap x[-15.25,-14.75]=0.5 < 0.7 ✔ blocks bodies passes bullets ✔. Frame pieces: x[-22,-18.5] box(-20.25,2,3.5,0.8,4.5), x[-10.7,-10] box(-10.35,2,0.7,0.8,4.5). Lintel above doors: box(-16,2, 12,0.8, 1.4, y=3.1) covering x[-22,-10] y[3.1,4.5].
   Also top-mid side of doors: nothing.
8) Top mid walls: west x=-22 z[-16,2] box(-22.5,-7,1,18,5); north z=-16 x[-22,-10] box(-16,-16.5,12,1,5); east x=-10 z[-16,2] with opening z[-15,-12] (cat ramp mouth): segments box(-9.5,-15.5,1,1,5) z[-16,-15], box(-9.5,-4.5,1,13,5) z[-11,2]. Cat ramp mouth z[-15,-12]?? ramp z[-18,-12]: mouth z[-15,-12] 3m wide ✔ hmm ramp width along z: z[-18,-12] 6m; mouth only z[-15,-12] ok.
9) Cat ramp: ramp spec {x0:-10,x1:-2? ramp along x: ramp(cx=-6, cz=-15, w=8 (x dir), d=6, y0=0 at x=-10, y1=2.1 at x=-2, axis 'x')}. Slabs: 9 slabs. Fill below ramp? The ramp low end is in top mid; below-ramp space x[-10,-2] z[-18,-12] under slope — block with fill box approx triangle? Just add solid box x[-10,-2] z[-18,-12] y[0,0.9]? That would block walking under low part — walking under happens only at ramp start in top mid... top mid x ends -10 ✔ so region x[-10,-2] is ONLY reachable from cat/ct-mid side? CT mid z[-10,2]: region x[-10,-2] z[-18,-10]: north of CT mid... below-cat area! Earlier decided cat solid fill x[-2,26]; for x[-10,-2] (under ramp): CT mid is z[-10,2]; strip z[-12,-10] x[-10,-2]: walkable pocket between CT mid and ramp face? Ramp face at z=-12 (ramp z[-18,-12])... ramp low edge at x=-10 height 0 → near x=-10 the ramp is basically ground level, so area under ramp near x=-10 is walkable ground anyway. Put fill box under ramp: x[-8,-2] z[-18,-12] y[0,1.2] (leaving x[-10,-8] as walkable slope start). Eh — slabs handle walking ON ramp; walking UNDER ramp near x=-10 where height<1.75 should be blocked: add fill box x[-6,-2] z[-18,-12] y[0,1.8] (under higher part) ✔ and x[-10,-6] leave (height there 0..1.05, slope walkable, can't stand under... a player at ground y0 under slab at 0.5? slabs are colliders: standing under slab region where slab bottom... slabs stack from y0! Each slab is a full box from y0? NO — slabs should be boxes from y0 up to local height (solid columns) → then "under ramp" is inherently solid ✔✔. Slab i: box with y[0, h_i]. So ramp = solid stepped mass ✔ no under-walk, and that's fine/authentic (cat is a building). Same for all ramps ✔. So cat ramp slabs: columns y[0,h]. Visual smooth ramp box above? Visual: stepped is fine actually if slabs have sand tex (like stairs)... CS cat slope is smooth but stairs read okay. To keep visual quality, I'll render smooth ramp quad + slabs invisible-collide only?? Colliders don't render anyway — visuals are separate boxes list! I said boxes produce BOTH mesh & collider. For ramps: slabs = colliders only (no mesh), plus a visual smooth ramp mesh. I'll add flag noMesh on slab boxes. And ramp visual via custom geometry function.
   
   So box() default mesh+collide; slab boxes mesh:false. Ramp visual: rotated box mesh (no collider). Good.

10) Catwalk: solid fill box x[-2,26] z[-18,-12] y[0,2.1] tex sand — walk on top ✔. North wall on cat: box(11? x[-2,20] z=-18: box(9,-18.5, 22,1, 3, y=2.1) h3 → top 5.1 ✔ from x[-2,20]. Short open x[20,26].
    Cat south edge: no rail (drop to CT mid) ✔. West end: ramp continues ✔.
    Short boost box: (16,-13.2, 1.6,1.6, 0.7, y=2.1,'crate') top 2.8.
11) CT mid: bounded: north z=-10?? CT mid z[-10,2]; cat fill z[-18,-12] leaves strip z[-12,-10] x[-2,26]: walkable ledge along cat south face — fine (part of CT mid extension x[-2,16]?) east of x=16: strip z[-12,-10] x[16,26]: leads to cross? cross z[-14,2] x[28,56]: strip x[16,26] z[-12,-10] vs cross x starts 28: gap x[26,28] z[-12,-10]: fill box. So strip z[-12,-10] x[-2,26] connects CT mid to... at x=26 dead end (fill at 26-28). Hmm that strip gives a second CT→near-A-route? It dead-ends; acceptable (or fill it). Fill it: box x[-2,26] z[-12,-10]?? That's against cat face — fill y[0,2.1]? It'd merge with cat visually (sand) ✔ do it: box(12,-11, 28,2, 2.1). Wait but then CT mid z[-10,2] north boundary = this fill face z=-10 ✔ clean. And xbox against it at (0,-9)?? xbox z[-10.1,-7.9]: box(0,-9,2.6,2.2,1.0). Jump xbox→cat top 2.1: xbox top 1.0, need 1.1 < 1.18 ✔; horizontal: xbox z max -7.9, cat face now z=-10: gap 2.1m z... xbox must hug cat face: fill face at z=-10; xbox center z=-9, d=2.2 → spans z[-10.1,-7.9] ✔ adjacent ✔. Jump up 1.1 and move north 0.3 ✔ lands on cat? cat surface at z≤-10 y2.1 ✔ (fill x[-2,26] includes x=0 ✔).
    CT mid walls: south z=2: box(3,2.5, 26,1, 4.5) x[-10,16] ✔ (east of doors). west x=-10 z[-10,2]: box(-10.5,-4,1,12,5) ✔. east x=16 z[-10,2] with opening z[-6,0]?? link corridor to cross: planned x[18,28] z[-6,0]; opening x=16 z[-6,0]: wall segments: box(16.5,-8,1,4,5) z[-10,-6]; box(16.5,1,1,2,5) z[0,2]. Link corridor: x[16,28] z[-6,0]; its north wall z=-6: box(22,-6.5,12,1,5) x[16,28]; south wall z=0: box(22,0.5,12,1,5) x[16,28]?? but CT mid z[-10,2] x[-10,16]: corridor extends east from opening ✔ walls cover x[16,28] ✔. Corridor opens into cross x[28,56] z[-14,2] at x=28 ✔ (cross z[-14,2] ⊃ z[-6,0] ✔).
12) CT spawn: x[-6,16] z[-30,-16]: walls: north z=-30: box(5,-30.5,22,1,5); east x=16: box(16.5,-23,1,14,5) z[-30,-16]; west x=-6 z[-30,-16] with opening z[-30,-26] (to B hall): segments: box(-6.5,-21,1,10,5) z[-26,-16]; south z=-16 x[-6,16] with opening x[-8? corridor x[-10,-4]... corridor to CT mid: x[-10,-4] z[-16,-10]: opening in CT spawn south wall x[-8,-2]?? corridor x[-10,-4]: opening x[-9,-5] at z=-16: wall segments: box(2,-16.5, 8? x[-6? ... CT spawn x[-6,16]: south wall z=-16 from x-6 to 16 minus opening x[-9,-5]→ but x[-9,-5] extends west of CT spawn (x<-6)! The corridor x[-10,-4] is WEST of CT spawn mostly. Conflict: corridor connects CT spawn's south-west? Make corridor connect via WEST wall instead: opening x=-6 z[-16,-12]?? CT spawn z[-30,-16]: z=-16 is the south face... corridor x[-10,-4] z[-16,-10] attaches to CT spawn at point (-6..-4, -16) corner. Redefine: corridor x[-10,-2] z[-16,-10]?? cat fill x[-2,26]... corridor east wall x=-2 ✔; opening in CT spawn south wall x[-6,-2] (z=-16): CT spawn x[-6,16] ✔ opening x[-6,-2] ⊂ ✔. Corridor x[-10,-2] z[-16,-10]; walls: west x=-10 box(-10.5,-13,1,6,5) z[-16,-10]; north z=-16 box(-6,-16.5, 8,1,5)?? that's the CT spawn south wall segment x[-10,-2] minus opening x[-6,-2]: segment x[-10,-6] box(-8,-16.5,4,1,5) ✔; opening x[-6,-2] ✔; corridor south z=-10: opening to CT mid x[-10,-2] ✔ full (CT mid z[-10,2] x[-10,16] ✔ overlap x[-10,-2] at z=-10 ✔).
    CT spawn south wall full: z=-16: segments: x[-6,-2] opening (corridor), so wall x[-2,16]: box(7,-16.5,18,1,5) ✔; plus x=-6 corner... west wall segments done above (z[-26,-16] box(-6.5,-21,1,10,5) covers z[-26,-16] ✔ and z[-30,-26] opening ✔). But south wall x[-2,16] box z=-16.5... wait wall thickness 1 centered z=-16: z[-16.5,-15.5]: overlaps corridor z[-16,-10]?? corridor z[-16,-10] x[-10,-2]: wall x[-2,16] doesn't overlap corridor x ✔. CT spawn interior z[-30,-16] ✔ wall at z=-16 is its boundary ✔ hmm wall center z=-16 spans z[-16.5,-15.5] → intrudes into corridor strip z[-16,-15.5] for x[-2,16]: corridor x[-10,-2] ✔ no x overlap ✔ fine.
13) B hall: x[-36,-6] z[-32,-24]: north wall z=-32 box(-21,-32.5,30,1,5); south wall z=-24 box(-21,-24.5,30,1,5); east: CT spawn west wall x=-6 with opening z[-30,-26] ✔ (already); west: B site east wall x=-36 with doors z[-30,-26] & window z[-38,-34]?? window z[-38,-34] is south of B hall z[-32,-24]!! Window must be on B hall side... B site z[-44,-20]; B hall z[-32,-24] attaches at x=-36 z[-32,-24] only. Window should be within z[-32,-24] to connect B hall↔B site?? But then doors+window both in z[-32,-24]: doors z[-30,-26], window z[-26,-24]?? Crowded. Real dust2: from CT corridor to B: doors (main) + window (north of doors? the window is on the site wall facing the corridor bend...). Simplify: B hall z[-32,-24]; doors at x=-36 z[-30,-27]; window at x=-36 z[-26,-24.5]? Both in hall wall. Window hole y[0.9,2.3]. Hmm z[-26,-24.5] adjacent doors. Alternatively extend B hall north branch: B hall main z[-32,-24] PLUS branch z[-38,-32] x[-36,-30] leading to window at x=-36?? window on B site east wall z[-38,-34]... branch x[-36,-30] z[-38,-32]: east of it x=-30 wall; it attaches B hall at z=-32 opening x[-36,-32]?? B hall north wall z=-32 opening x[-34,-30] → branch x[-34,-30] z[-38,-32]?? then window at... branch west wall x=-34?? I'm overcomplicating. FINAL: B hall is L-shaped: main x[-36,-6] z[-32,-24]; plus north-west nook x[-36,-28] z[-40,-32] connecting to window on B site east wall x=-36 z[-38,-34] ✔. Nook walls: north z=-40 box x[-36,-28]; east x=-28 z[-40,-32]; B site east wall x=-36 z[-44,-20] segments: solid z[-44,-38]: box(-36.5,-41,1,6,5); window opening z[-38,-34]: lintel above: box(-36.5,-36,1,4, 2.2, y=2.3) y[2.3,4.5]; sill below: box(-36.5,-36,1,4,0.9) y[0,0.9]; solid z[-34,-30]: box(-36.5,-32,1,4,5); doors opening z[-30,-26]: door panels: box(-36.5,-28.6,0.7? doors thickness 0.4: doorA box(-36,-28.7, 0.5,2.6, 3.4,'door') z[-30,-27.4]; doorB box(-36,-26.3,0.5,1.4? ... gap 0.5: doorA z[-30,-27.75], doorB z[-27.25,-26], gap z[-27.75,-27.25]=0.5 ✔; solid z[-26,-20]: box(-36.5,-23,1,6,5). Lintel over doors: box(-36,-28,1,4,1.5,y=3.0). Nook: x[-36,-28] z[-40,-32] walls: north z=-40 box(-32,-40.5,8,1,5); east x=-28 box(-27.5,-36,1,8,5) z[-40,-32]; south: B hall north wall z=-32 has opening x[-36,-28]?? B hall north wall box(-21,-32.5,30,1,5) covers x[-36,-6]: replace with segments: x[-28,-6]: box(-17,-32.5,22,1,5); opening x[-36,-28] ✔ (nook mouth). ✔
14) B site: x[-62,-36] z[-44,-20]: walls: north z=-44 box(-49,-44.5,26,1,5) x[-62,-36]; west x=-62 box(-62.5,-32,1,24,5) z[-44,-20]; south z=-20 x[-62,-36] with tunnel exit opening x[-50,-44]: segments: box(-56,-20.5,12,1,5) x[-62,-50]; box(-40,-20.5,8,1,5) x[-44,-36]; exit arch lintel box(-47,-20.5,6,1,1.6,y=2.8). 
    Platform: fill box x[-60,-48] z[-42,-34] y[0,1.0] tex sand2? use 'sand'; ramp x[-48,-44] z[-40,-34]: slope y0(x=-44)→1.0(x=-48): ramp axis x. Slabs columns. Visual ramp.
    Boxes: box(-52,-28,2.6,2.6,1.3,'crate'); box(-49.5,-25.5,1.4,1.4,0.9,'crate'); car: box(-42,-24,3.6,2,1.15,'metal') + box(-42,-24,2,2.2,0.5,y=1.15,'metal')?? car top: skip, one box + smaller cabin box(-42.3,-24,1.6,1.8,0.5,y=1.15) total 1.65? fine.
    Plant zone B: x[-58,-44] z[-40,-26].
15) Tunnels: lower x[-60,-52] z[4,38]: walls: west x=-60 box(-60.5,21,1,34,4); east x=-52 z[4,38]: box(-51.5,21,1,34,4)?? junction x[-60,-44] z[-2,6] attaches north ✔; south: tunnel south wall z=38 x[-60,-52]: but T spawn overlap x[-58,-52] z[32,38]!! T spawn occupies that → tunnels open INTO T spawn at z[32,38]?? then tunnel south wall z=38 cuts T spawn... T spawn south wall at z=44 covers x[-58,-30]; region x[-60,-52] z[38,44]: west of T spawn: fill box(-56,41,8,6,4) → then tunnels have no south wall needed (open into T spawn at z=38? tunnels z[4,38] x[-60,-52]; T spawn z[32,44] x[-58,-30]: overlap x[-58,-52] z[32,38] shared ✔ tunnels east wall x=-52 z[4,38] also walls T spawn? T spawn x extends to -30 > -52 ✔ so box(-51.5,21,1,34,4) z[4,38] sits inside T spawn area z[32,38] x=-52!! It would wall off part of T spawn — that's correct: T spawn x[-58,-30]: the tunnel east wall x=-52 for z[32,38] divides T spawn west strip x[-58,-52] (tunnel mouth) from rest ✔ intended: tunnel mouth region = that strip ✔. T spawn south wall: x[-52,-30]: box(-41,44, 22? x[-52,-30] center -41 w22 ✔. Plus fill x[-60,-52] z[38,44] box(-56,41,8,6,4) ✔. Ceiling over tunnels: box(-56,21,8,34,0.5,y=3.2,'sand2') y[3.2,3.7] covers x[-60,-52] z[4,38] ✔ (dark tunnel). Wall heights 4 > ceiling 3.2 ✔.
16) Junction x[-60,-44] z[-2,6]: walls: west x=-60 box(-60.5,2,1,8,4); north z=-2 x[-60,-44] with ramp opening x[-50,-44]: segments box(-55,-2.5,10,1,4) x[-60,-50]; box(-42,-2.5,4? x[-44,-40]?? upper tunnel x[-52,-42] z[-18,4]... ramp in junction: from junction floor y0 up to upper tunnel y0.9: ramp at x[-50,-44]?? junction x[-60,-44]: ramp x[-50,-44] z[-2,6]: axis z: y0 at z=6 → 0.9 at z=-2?? upper tunnel z[-18,4] y0.9: ramp goes north-down?? junction z[-2,6] connects upper at z[-2,4] x[-52,-44]... upper x[-52,-42]: overlap x[-52,-44]?? [-60,-44]∩[-52,-42]=[-52,-44] ✔ z[-2,4] ✔. Ramp: x[-52,-46], z[-2,6], rise from y0 (z=6) to 0.9 (z=-2)? But upper tunnel continues z<4 at 0.9 ✔ mismatch at z[-2,4]: ramp at z=4: h=0.9*(6-4)/8=0.225; upper at z=4 is 0.9 ✘!! Ramp should reach 0.9 where upper begins (z=4): ramp z[4,10]?? junction z[-2,6]... Put ramp INSIDE upper tunnel south end: ramp x[-50,-44] z[0,8]: y0 at z=8?? junction z ends 6... ugh. Simplest: make upper tunnel floor y=0.9 via ground rect starting z=-18..2, and junction→upper step via ramp z[2,6] x[-50,-44]: y0(z=6)→0.9(z=2): 4m run 0.9 rise ✔ then upper rect z[-18,2] y0.9 covers x[-52,-42] ✔ junction floor z[-2,6] y0 (base rect) ✔ overlap z[2,6] x[-50,-44] ramp slabs above base ✔. Upper east wall x=-42 z[-18,2] box(-41.5,-8,1,20,4); west wall x=-52 z[-18,2] box(-52.5,-8,1,20,4)?? junction west wall x=-60... upper west x=-52: region x[-60,-52] z[-18,-2]: between junction north and B site south: fill box(-56,-10,8,16,4) ✔. Upper ceiling: box(-47,-8,10,20,0.5,y=3.0) x[-52,-42] z[-18,2]. Upper exit: north into B site z=-20 opening x[-50,-44] ✔ (B site south wall segments done ✔). Gap z[-20,-18] x[-52,-42]: corridor walls x=-52 z[-20,-18] box, x=-42 box, ceiling box(-47,-19,10,2,0.5,y=3.0) ✔ floor: upper rect extend z[-20,2] ✔.
17) Fills: 
 F1 x[-42,-22] z[-2,24] h5 (mid/tunnels separator) box(-32,11,20,26,5)
 F2 x[-42,-22] z[-18,-2] h5 box(-32,-10,20,16,5)
 F3 x[-36,-6] z[-44,-32] h5 box(-21,-38,30,12,5) (between B site north & B hall north; B hall z[-32,-24] ✔)
 F4 x[-36,-6] z[-24,-16] h5 box(-21,-20,30,8,5)?? B hall south z=-24; CT spawn x[-6,16]... region x[-36,-6] z[-24,-16] ✔ but corridor x[-10,-2] z[-16,-10] is z>-16 ✔ no conflict; nook x[-36,-28] z[-40,-32] ⊂ F3!! F3 covers z[-44,-32] x[-36,-6] including nook x[-36,-28] z[-40,-32] ✘ carve: F3a x[-28,-6] z[-44,-32] box(-17,-38,22,12,5); F3b x[-36,-28] z[-44,-40] box(-32,-42,8,4,5) ✔ nook z[-40,-32] free ✔.
 F5 x[-36,-10] z[-16,2] h4 box(-23,-7,26,18,4) (west of CT mid/corridor) — corridor x[-10,-2] ✔ east of -10 ✔ no conflict ✔. Hmm CT mid x[-10,16] ✔.
 F6 x[-10,28] z[-6,2]?? link corridor x[16,28] z[-6,0] ✘ conflict. Region north of CT mid south wall & south of... region x[-10,16] z[2,26]: east of mid corridor, south of CT mid: fill box(3,14,26,24,5) x[-10,16] z[2,26]?? long connector x[-10,46] z[28,36]: z[2,26] no conflict ✔. T ramp x[-34,-10] ✔ west of -10 ✔. ✔ F6.
 F7 x[16,46] z[2,28]?: region east of CT mid south, between connector(z[28,36]) and CT-mid-south-wall/link: link corridor x[16,28] z[-6,0]... region x[16,28] z[0,2]: strip: fill box(22,1,12,2,5)? and region x[16,46] z[2,28]: cross x[28,56] z[-14,2] ✔ z>2 free to fill: box(31,15,30,26,5) x[16,46] z[2,28]?? connector z[28,36] ✔ no overlap; outside-long x[44,64] z[20,34] x starts 44: overlap x[44,46] z[20,28] ✘!: shrink F7 x[16,44] box(30,15,28,26,5) ✔. Strip x[16,28] z[0,2]: covered by F7? F7 z[2,28] no; add box(22,1,12,2,5) ✔. Also x[28,44] z[0,2]? cross z ends 2 ✔ region x[28,44] z[0,2] is cross? cross z[-14,2] ✔ included ✔ so F7 z starts 2 ✔ but cross south wall needed at z=2 x[28,44]: F7 face ✔ ok.
 F8 x[28,44] z[2,12]?? covered F7 ✔ (z[2,28]) ✔. pit x[44,54] z[12,20] ✔ not covered ✔. region x[44,54] z[2,12]: fill box(49,7,10,10,5) F9 ✔ (between cross east & pit north). region x[44,54] z[20,?]: outside-long x[44,64] z[20,34] ✔ open ✔.
 F10 south-east corner x[-10,64] z[36,48] h5 box(27,42,74,12,5) ✔ (south of connector/outside-long). connector z[28,36] ✔.
 F11 x[46,64] z[34,36]? outside-long z[20,34] ✔ south wall z=34: F10 covers z[36,48]: wall strip z[34,36] x[44,64]: box(54,35,20,2,5)?? perimeter z=49 far. outside-long south wall z=34 x[44,64]: box(54,34.5,20,1,5) ✔. T ramp south?? T ramp z[24,34] south boundary z=34 x[-34,-10]: T spawn z[32,44] covers x[-58,-30]: region x[-30,-10] z[34,44]: pocket → fill box(-20,39,20,10,4) ✔. connector south wall z=36 x[-10,46]: F10 face at z=36 ✔. connector north wall z=28 x[-10,46]: F6 face (x[-10,16] z ends 26?? F6 z[2,26]: gap z[26,28] x[-10,16]!! fill box(3,27,26,2,5) ✔; x[16,46] z[26,28]: F7 z[2,28] ✔ face ✔; x[44,46]? covered ✔.
 T ramp west wall x=-34 z[24,34]?? T spawn x ends -30: overlap x[-34,-30] z[32,34] open ✔; wall x=-34 z[24,32]: box(-34.5,28,1,8,5) ✔; north wall z=24 x[-34,-22]: box(-28,23.5,12,1,5) ✔ (mid corridor opens x[-22,-10] ✔). F1 x[-42,-22] z[-2,24] face at z=24 ✔ covers x[-42,-22]... T ramp north wall x[-34,-22] ✔ consistent.
18) A site: fill box x[24,62] z[-44,-14] y[0,2.1] box(43,-29,38,30,2.1) tex sand2? top walkable ✔. Goose nook: box(25,-38,2,3,1.2,y=2.1,'crate')?? goose corner wall: building behind: F12 x[-6,24] z[-44,-18] h5 box(9,-31,30,26,5) (north of cat, west of site) ✔ covers x[-6,24]: CT spawn z[-30,-16] x[-6,16]: overlap z[-30,-18] x[-6,16] ✘!! CT spawn interior z[-30,-16]: F12 z[-44,-18] overlaps z[-30,-18] ✘. Shrink F12: x[-6,24] z[-44,-30]?? then strip z[-30,-18] x[16,24]: region east of CT spawn, north of cat: fill box(20,-24,8,12,5) x[16,24] z[-30,-18] ✔; strip z[-30,-18] x[-6,16] is CT spawn ✔. F12 box(9,-37,30,14,5) z[-44,-30] ✔. Cat north wall: box(9,-18.5,22,1,3,y=2.1) x[-2,20] ✔ then gap z[-30,-18.5] x[-2,16]?? CT spawn is z[-30,-16]... wait CT spawn z[-30,-16] vs cat z[-18,-12]: strip z[-18,-16] x[-6,16]: that's CT spawn south part? CT spawn z[-30,-16] includes up to -16: strip z[-16,-12] x[-2,16]: corridor x[-10,-2] z[-16,-10] ✔ covers x[-10,-2]; region x[-2,16] z[-16,-10]: vs cat fill z[-18,-12]... region x[-2,16] z[-12,-10]: earlier filled box(12,-11,28,2,2.1) ✔; region x[-2,16] z[-16,-12]: part of... CT spawn ends z=-16 ✔ so z[-16,-12] x[-2,16]: south of CT spawn: is it corridor? corridor x[-10,-2] ✔ no. cat fill z[-18,-12] covers z[-16,-12] x[-2,16] ✔ YES cat fill covers it. ✔ And x[16,24] z[-18,-12]: cat fill x ends 26 ✔ covers. x[24,26]? cat ✔. OK: so north of cat: x[-6,24] z[-44,-30] F12 ✔ + box(20,-24,8,12,5) F13 x[16,24] z[-30,-18] ✔ + CT spawn ✔ + cat ✔. Region x[-6,?] z[-18,-16] x[-10,-6]?? corridor x[-10,-2] z[-16,-10] ✔ z[-18,-16] x[-10,-6]: pocket between CT spawn west wall?? CT spawn x starts -6: region x[-10,-6] z[-18,-16]: fill box(-8,-17,4,2,5) ✔. region x[-6,-2] z[-18,-16]: CT spawn x[-6,16] z[-30,-16] ✔ covered. ✔
    Site props: triple box: box(38,-32,1.5,1.5,1.5,y=2.1,'crate'), box(39.6,-32,1.2,1.2,1.2,y=2.1,'crate'), box(38,-30.4,1.2,1.2,0.8,y=2.1,'crate'); big stack (quad): box(52,-38,2.4,2.4,1.4,y=2.1,'crate'), box(52,-38,1.4,1.4,1.0,y=3.5,'crate'); ramp-side box: box(34,-16,1.6,1.6,1.0,y=2.1,'crate')?? near ramp top; goose box: box(26.5,-40,1.4,1.4,1.1,y=2.1,'crate'); short elbow wall: box(27,-13,2,2,1.4,y=2.1)? earlier elbow at x[26,28] z[-14,-12]: box(27,-13,2,2,1.5,y=0)?? it sits at cross level? short corner: box x[26,28] z[-14,-12] y[0, 3.5]?? That blocks cross strip x[28,42] z[-14,-12]?? x[26,28] only ✔: box(27,-13,2,2,3.5) — a pillar corner ✔ (short elbow) ✔.
    Blue box: (57,-11,1.2,1.2,1.0,'crateMetal') at long corner y0 ✔.
19) A ramp: ramp x[28,42] z[-14,-4]: y0 at z=-4 → 2.1 at z=-14, axis z. Slabs columns + visual. Ramp sides: walls? west side of ramp: region x[24,28] z[-14,-4]: "under goose" strip at y0: walkable ✔ connects cross to...? dead end at cat fill? cat x ends 26: strip x[24,28] z[-14,-4]: south of cat face z=-12 for x[24,26]... it's open to cross ✔ fine ("under ramp" corner, ninja spot ✔ authentic!).
    Ramp east side: x[42,54] z[-14,-12] cross strip ✔ open.
20) Long corridor x[54,64] z[-12,20]: east wall x=64 box(64.5,4,1,32,5) z[-12,20]; west wall x=54: z[-4,20]: box(53.5,8,1,24,5)?? cross overlap x[54,56] z[-14,2]: opening z[-12,2]? corridor z[-12,20]: west wall segments: z[2,20] box(53.5,11,1,18,5) ✔; z[-12,2] open (cross↔corridor ✔). North: long corner slope ramp x[54,62] z[-18,-12] y0(z=-12)→2.1(z=-18) axis z ✔ slabs+visual. Region x[62,64] z[-18,-12]: fill box(63,-15,2,6,5) ✔. Region x[54,64] z[-14,-12]?? corridor z starts -12: strip z[-14,-12]: site fill x[24,62] z[-44,-14] ✔ no... site z[-44,-14]: strip z[-14,-12] x[54,62]: open? ramp z[-18,-12] covers x[54,62] z[-18,-12]: includes z[-14,-12] ✔ ramp occupies it (low end ~y1.0? ramp y at z=-14: (z from -12 to -18: h=2.1*(-12-z)/-6?? at z=-14: h=2.1* ( (-12)-(-14) )/6? h(z) = 2.1 * (z-(-12))/((-18)-(-12)) → z=-14: 2.1*(-2)/(-6)=0.7 ✔ walk up ✔.
    Long doors at z=18?? corridor z[-12,20]: doors wall at z=18: frame: box(54.5,18,1? x[54,55] box(54.5,18,1,1,4.5); openings x[55.5,58] & x[60,62.5]; pillars: x[58,60] box(59,18,2,1,4.5); x[62.5,64] box(63.25,18,1.5,1,4.5); lintels: box(56.75,18,2.5,1,1.4,y=2.6), box(61.25,18,2.5,1,1.4,y=2.6). Doors passable ✔ (two arches) authentic 长门 ✔.
    Corridor continue z[18,20]: then south into outside-long x[44,64] z[20,34] ✔ opening z=20 x[54,64] ✔ (no wall).
    Outside long walls: east x=64 z[20,34] box(64.5,27,1,14,5); north: pit/doors wall z=20 x[44,54]?? pit z[12,20]: pit north edge z=12... pit is x[44,54] z[12,20]: north of pit z[2,12] F9 fill ✔ so outside-long north wall for x[44,54] = pit face (drop) ✔; for x[54,64]: doors wall z=18 + strip z[18,20] open corridor ✔ hmm outside-long z[20,34] vs corridor z[-12,20]: boundary z=20: opening x[54,64] ✔, wall x[44,54]?? pit z[12,20] ⊂ z<20 ✔ so x[44,54] at z=20: pit south face ✔ (pit edge) ✔ open drop ✔.
    Pit: floor rect y-0.9 x[44,54] z[12,20]; walls: box(44,16,0.5? pit side walls: x=44: box(43.75,16,0.5,8,0.9) y[-0.9,0]; x=54: box(54.25,16,0.5,8,0.9); z=12: box(49,11.75,10,0.5,0.9); z=20: box(49,20.25,10,0.5,0.9). ✔ jumpable out (0.9<1.18 ✔).
    Sightline pit→long: from pit (49,-? eye y=-0.9+1.62=0.72) through doors arches (y0..2.6 x[55.5,58] or [60,62.5])... pit x[44,54] vs doors x[55.5,58]: diagonal view through arch up corridor ✔ ok-ish. Move pit east: x[48,58]?? doors at x[55.5,58]: pit x[48,58] z[12,20]: overlapping corridor west wall x=54 z[2,20]!! corridor z[-12,20]: pit x[48,58] overlaps corridor x[54,58] ✘. Keep pit x[44,54]; sightline diagonal through arch x[55.5,58] from (50,16): line passes wall x=54 z=16..18? wall x=54 z[2,20] blocks!! wall at x=54 z[2,20] is between pit and doors ✘✘. Fix: wall x=54 only z[2,12] (box(53.5,7,1,10,5)), opening z[12,18]?? then corridor connects to pit area z[12,18] x[54,...]: corridor x[54,64] meets pit x[44,54] at x=54 z[12,18]: pit floor -0.9 vs corridor 0: drop into pit ✔ fine (they connect: corridor side drop into pit — weird but workable: actually that models "long doors area connects to pit"). Hmm but then Ts in pit can walk east into corridor without doors ✘ bypasses doors! Not authentic. Better: keep wall, accept pit sightline through arch is blocked... then pit is useless. Alternative: shift pit north to align with doors: pit x[50,58]?? corridor x[54,64] overlap again. 

    Rethink: In real dust2, pit is WEST of long corridor's south portion, near doors, at lower level; from pit you look NORTH through the doors? No—pit is south-west of the doors; from pit you look through the two doors (which face south-west-ish?)... Long doors wall runs east-west?? The long corridor runs north-south; doors cross it east-west. Pit is OUTSIDE (south of) the doors, to the WEST side. From pit you can see through the WEST arch up the corridor — the line from pit (west) through west arch: passes through the door plane at an angle — wall x=54 (corridor west wall) only exists NORTH of doors (z<18)?? The corridor west wall runs the corridor length... From pit at (48,21), eye 0.72, aiming at arch opening x[55.5,58] z=18, continuing north into corridor: the line crosses x=54 at z≈19.5?? Let's compute: from (48,21) to (56,18): direction (8,-3); at x=54: t=6/8=0.75 → z=21-3*0.75=18.75 — crosses x=54 at z=18.75 which is SOUTH of doors plane (z=18)?? 18.75 > 18 → that's outside corridor (z>18 region x=54 is corridor exterior wall segment? corridor z[-12,20]: at z=18.75 x=54 is corridor west wall IF wall covers z up to 20. Set corridor west wall: z[-12,18]?? then z[18,20] x=54 open: corridor connects laterally to outside-long at z[18,20]?? outside-long z[20,34]... gap z[18,20] x[44,54] belongs to? I had outside-long z[20,34] and pit z[12,20] x[44,54]. Redefine: pit x[44,54] z[14,20]; outside-long x[44,64] z[20,34]; strip x[44,54] z[18,20]: pit ✔ (pit z[14,20] covers). Corridor west wall: x=54 z[-12,14] box(53.5,1,1,26,5)?? z[-12,14] center 1 len 26 ✔. Then z[14,18] x=54: open: pit (x up to 54) touches corridor (x from 54) at z[14,18]: pit floor -0.9, corridor y0: so from corridor you can drop into pit at that seam ✔ and from pit you can see/jump up into corridor?? jump 0.9 ✔ — pit connects to corridor without doors ✘... BUT the doors are right there at z=18; the seam z[14,18] is INSIDE the doors plane region?? doors at z=18: seam z[14,18] is north of doors → pit connects to corridor north of doors = pit bypass ✘.

    Resolve by accepting compromise (authenticity vs geometry sanity): pit at x[44,52] z[12,20], corridor west wall x=54 z[-12,20] full (box(53.5,4,1,32,5)) EXCEPT cross opening z[-12,2]... wait cross opening was z[-12,2] — wall segments: z[2,20] box(53.5,11,1,18,5) ✔ (as before). Pit sightline: from pit through WEST door arch: arch x[55.5,58] z=18 plane. Pit max x=52. Line from (50,17) eye0.72 → arch center (56.75,18) → continues to corridor interior: crosses x=54 at z≈17.4?? from (50,17)→(56.75,18): at x=54: t=(54-50)/6.75=0.59 → z=17+0.59=17.59 → z=17.59 <18: point (54,17.59) — is there wall at x=54 z=17.59? Wall x=54 z[2,20] YES ✘ blocked!! The wall x=54 extends to z=20 which is SOUTH of doors (18)... wall z[2,20] but doors at z=18 — south of doors should be... corridor z[-12,20]: I defined corridor extends to z=20, doors at z=18: so z[18,20] is corridor piece south of doors (vestibule) ✔ wall x=54 z[18,20] is vestibule west wall ✔ needed. Pit sightline through arch must cross x=54 at z>18 (in vestibule): from pit (x≤52) to arch (x≥55.5, z=18): line crosses x=54 before reaching z=18 → at z<18 → hits wall ✘. Conclusion: with corridor axis N-S and pit west of vestibule, pit can't see through arch. Real dust2: pit is directly SOUTH of the west arch, i.e., the west arch faces the pit area (arch in a wall running E-W at z=18 spanning x[54,64], pit south of it at x[54,64]?? no pit is west...). 

    FINAL fix: make pit x[50,58], z[20,28]?? south of doors overlapping outside-long ✔ and corridor x[54,64] z[-12,20] meets pit at x[54,58] z=20 seam (drop 0.9) ✔ south of doors ✔ authentic-ish: pit south of doors, slightly west; sightline from pit (54,22) north through west arch x[55.5,58] ✔ line stays x>54 → no wall crossing ✔✔. Pit connects to corridor at z=20 seam x[54,58] (drop/jump 0.9) — south of doors ✔ authentic (pit↔vestibule). And outside-long x[44,64] z[20,34] overlaps pit x[50,58] z[20,28] → pit is a depression within outside-long ✔ carve ground rects accordingly. F9 fill x[44,54] z[2,12] stays. Strip x[44,50] z[12,20]: fill box(47,16,6,8,5) ✔. Strip x[58,64]? corridor covers x[54,64] z[-12,20] ✔. Region x[44,50] z[20,?]: outside-long ✔ open.
    Ground rects update: pit floor x[50,58] z[20,28] y=-0.9; carve from base: base pieces around it.
    Pit walls: x=50 box(49.75,24,0.5,8,0.9); x=58 box(58.25,24,0.5,8,0.9); z=28 box(54,28.25,8,0.5,0.9); z=20 box(54,19.75,8,0.5,0.9)?? vestibule seam: keep wall at z=20? Then pit↔corridor blocked; fine actually — pit only reachable from outside-long ✔ simpler: wall z=20 exists ✔ (drop from corridor into pit blocked, ok). Hmm authentic? "good enough".
    Wait then from pit sightline: pit z[20,28] south of doors z=18: from (54,22) eye 0.72 through arch (56.75,18, y0-2.6): line z decreases, x increases slightly; crosses x=54?? starts x=54... choose (53,24)→(56.5,18): crosses x=54 at z≈22.9: wall x=54 z[2,20]: z=22.9 > 20 ✔ no wall (outside-long region) ✔ then through arch into corridor ✔✔ SIGHTLINE WORKS.

21) Long connector: x[-10,46] z[28,36]: east end: outside-long x[44,64] z[20,34]: overlap x[44,46] z[28,34] ✔ open. T ramp opening x=-10 z[28,34] ✔ (wall x=-10 z[2,24] done; add wall x=-10 z[34,?]: T ramp z ends 34: box(-9.5,30,1,4? opening z[28,34]: no wall ✔; wall x=-10 z[24,28]? T ramp z[24,34]: connector z[28,36]: opening z[28,34] ✔ wall segments x=-10: z[24,28] box(-9.5,26,1,4,5) ✔.
    Connector cover: box(18,32,2,2,1.1,'crate'), box(30,33,1.5,1.5,0.9,'crate').
22) Mid walls done. T ramp cover: box(-26,30,2,2,1.1,'crate').

Ground rects (y0 base with pit carve):
 pit x[50,58] z[20,28] y-0.9.
 base rects:
  A: x[-64,64] z[-48,20] y0
  B: x[-64,50] z[20,48] y0
  C: x[58,64] z[20,48] y0
  D: x[50,58] z[28,48] y0
  pit: x[50,58] z[20,28] y-0.9
 upper tunnels overlay: x[-52,-42] z[-20,2] y0.9 (rect above base ✔)
 (B platform/cat/site/fills are boxes ✔)

Waypoints — finalize (x,z,y auto via support except listed):
```
0 tspawn(-46,39)
1 tspawnW(-55,36)
2 trampW(-30,29)
3 tramp(-20,29)
4 trampE(-13,31)
5 midBot(-16,22)
6 mid(-16,10)
7 midTop(-16,-2)
8 midTopN(-16,-12)
9 catRamp(-7,-13.5)  (on ramp)
10 cat(4,-15) y2.1
11 catMid(12,-15) y2.1
12 short(22,-15) y2.1
13 shortEnd(25,-16) y2.1
14 aSiteW(30,-20) y2.1
15 aSite(40,-32) y2.1
16 goose(27,-40.5) y2.1
17 aSiteE(52,-34) y2.1
18 longTop(58,-16) y2.1  (slope top, on site edge?) slope x[54,62] z[-18,-12] top at z=-18 y2.1: wp (58,-17) y2.1
19 longCorner(58,-10)
20 longCor(58,4)
21 longDoorsN(56.7,17)  (through west arch)
22 longDoorsS(56.7,21)
23 outLong(50,31)
24 pit(54,24) y-0.9
25 connE(38,32)
26 connW(4,32)
27 tunnLow(-56,28)
28 tunnLowN(-56,8)
29 tunnJunc(-52,2)
30 tunnUpS(-47,4) y0.5 (on ramp)
31 tunnUp(-47,-8) y0.9
32 tunnExit(-47,-19) y0.9
33 bSite(-50,-30)
34 bPlat(-54,-38) y1.0
35 bPlatRamp(-46,-37) y0.5
36 bCar(-43,-25)
37 bDoorsIn(-39,-28)
38 bDoorsOut(-33,-28)
39 bHall(-20,-28)
40 bHallE(-10,-28)
41 ctSpawn(5,-22)
42 ctSpawnW(-4,-24)
43 ctCor(-6,-13)  corridor
44 ctMid(-4,-4)
45 ctMidE(8,-4)
46 xbox(0,-9) y1.0
47 ctLink(22,-3)
48 crossW(32,-1)
49 rampBot(35,-5) y~0.3? ramp z[-14,-4]: at z=-5 h≈0.2 → y auto
50 rampMid(35,-9) y1.1
51 rampTop(35,-13) y2.05
52 crossE(48,-4)
53 underRamp(26,-8)
54 windowOut(-33,-36)   nook
55 windowIn(-39,-36)   (site, via window jump)
56 ctSpawnN(10,-26)
```
Links (add both directions):
0-1,0-2,0-3? 0-2 (via trampW? 0(-46,39)→2(-30,29): path crosses T spawn north-east... fine straight-ish), safer: 0-1,0-3,1-27(tunnel),2-0,2-3,3-4,3-5(mid),4-26(conn),5-6,6-7,7-8,8-9,9-10,10-11,11-12,12-13,13-14,14-15,14-16,15-16,15-17,17-18,18-19,19-20,20-21,21-22,22-23,23-24,23-25,25-26,26-4,1-27,27-28,28-29,29-30,30-31,31-32,32-33,33-34,34-35,33-36,33-37,36-37,37-38,38-39,39-40,40-42,42-41,41-56,41-43,43-44,44-45,45-47,44-46(jump 46),46-10(jump),47-48,48-49,49-50,50-51,51-14? ramp top z=-13 → site z=-14 edge: 51-14 (14 at (30,-20)? path over site edge ✔) also 51-15,48-52,52-19(cross→corridor: (48,-4)→(58,-10) ✔ opening ✔),48-53,53-44?? (26,-8)→(-4,-4): passes CT mid? under cat? z=-8..-4: crosses corridor x[-10,-2] z[-16,-10]?? line from (26,-8) to (-4,-4) passes x[16,-2] z≈-7..-4: CT mid z[-10,2] ✔ open ✔ crosses link corridor? x[16,28] z[-6,0]: line at x=16: z≈-5.3 ✔ inside corridor z[-6,0] ✔ ok ✔ so 53-44 valid ✔. Also 45-56? no. 41-44? via corridor 43 ✔. 39-nook-54: 39-54 (( -20,-28)→(-33,-36): crosses B hall north wall z=-32 x[-28,-6]?? opening x[-36,-28] ✔ line at z=-32: x≈-30.5 ✔ inside opening x[-36,-28] ✔ ✔), 54-55 jump (window), 55-33, 55-34? ( -39,-36)→(-54,-38): inside site ✔ add 55-33.
Camp spots (CT): 15(face W),16(face S/ramp),17(face E/long),12(face W cat),11(face W),44(face S doors),34(face SE tunnels),33(face S),55(face S),19(face S),49(face N? T side)... store camp yaw per wp optionally; bot faces nearest threat direction fallback.

Jump links: 44→46 (onto xbox), 46→10 (xbox→cat), 54→55 (window), 33→34? platform h1.0: jump 33→34 ✔ or via ramp 35 ✔ add both. Also 24→23 (pit out, 0.9 jump) ✔ and 23→24 walk (drop) ✔. 22→24? vestibule wall z=20 blocks ✔ no. 19→18? slope ✔ walk. 14→53? drop from site edge z=-14 to under-ramp? site face 2.1 drop: allowed (fall, no damage) add 14→53 one-way? bots use links bidirectional — drop down ok but climbing back impossible → make directed? My BFS uses undirected; risk: bot tries to path up site face ✘ stuck. Avoid: don't link 14-53; 53 connects via 48/44 only ✔.

CT strategies: assign: A: wp 15/17/12; mid: 44/46/10; B: 34/33. Retake: path to site wps.
T strategies: 
 rushB: [1,27,28,29,30,31,32,33] then plant at 33 zone.
 rushA_long: [4,26,25,23,22,21,20,19,18,17] plant 15.
 A_split: half long half cat [3,5,6,7,8,9,10,11,12,13,14,15].
 default: slow: hold then execute.
Implementation: each T bot assigned a route (array of wp indices) with progress target = next unreached; simply set objective = final site; pathing BFS handles route; strategy only affects which site + maybe "via" waypoint to bias route (BFS with waypoint cost: to force via long vs cat, add big cost penalty on the other route's entry wp per-bot). Implement: pathfinding with per-bot blocked set: for long route: block wp 9,10 (cat entry); for cat route: block 21,22 (long doors). ✔ simple.

CT rotate: when bomb planted, all CT path to site wps.

Bomb plant wp: site A zone center (40,-32) wp15; site B ( -50,-30) wp33. Plant zone rects: A: x[34,48] z[-40,-26]; B: x[-58,-42] z[-38,-24].

Buy zones: T spawn rect & CT spawn rect.

OK! Also spawns: T: 5 points around (-46,39)... CT: (5,-22) area points.

DONE planning Dust2. Maps 2 & 3: I'll design more freely & compactly.

Map2 "古镇 Stonehaven" (inferno-esque), bounds x±56 z±42, ground y0 mostly, A site elevated y1.6.
Regions:
- T spawn: x[-50,-30] z[28,38].
- 中路 mid: x[-20,-8] z[-2,30] from T ramp area north; mid top splits: west "arch" → A site; east "library/link" → CT.
- A site: x[-40,-8] z[-38,-16] y1.6 plateau (big), ramps: "A short" from mid top x[-20,-8] z[-16,-8] slope up; "apps/阳台" east: building corridor x[6,20] z[-30,-10] y1.6 with stairs down to CT? Apps route: T→east corridor x[10,24] z[10,28] ("T apps") → stairs up → apps hall x[6,24] z[-10,10]?? elevated y1.6 → balcony drop to A site ✔.
- B site: x[28,52] z[-38,-14] y0; "banana" curved: from T east corridor x[20,34] z[20,34] → north corridor x[34,44] z[-8,22] → turns west into B site at x[28,34]? B site west wall opening z[-20,-14]?? banana enters B at south: opening z=-14 x[36,42].
- CT spawn: x[-2,16] z[-40,-28]; connects A (west corridor x[-8,6] z[-34,-24] + ramp up), B (east corridor x[16,30] z[-32,-22]), mid (south opening).
Cover: crates, wells, carts.
Waypoints ~40. Sniping: mid long line, banana line.

Map3 "货运站 Freight", night, bounds x±48 z±36:
- Lanes: three horizontal corridors (north/mid/south) + containers everywhere.
- A site: west end elevated train deck y1.8 x[-44,-24] z[-30,-8]; B site: east warehouse x[24,44] z[-30,-6] with big doors.
- T spawn south x[-14,10] z[24,32]; CT spawn north x[-10,12] z[-34,-26].
- Mid corridor x[-6,6] z[-24,24] with train cars crossing; connectors at x[-30,-20] ("west link") and x[16,26] ("east link").
- Containers as cover; low ceilings in warehouse (y3).

Both simpler; I'll write them with the same box/waypoint format, ~25-35 waypoints each.

Time to WRITE. I'll now produce the code files. Given the message length, I'll write game.js in one big file. Let me go section by section and keep it tight but complete. I'll skip some polish where needed but keep all features functional.

Also player interact: E for plant/defuse/pickup bomb? Plant: hold E? Use E hold for plant/defuse; bomb auto-carried. Weapon pickup: walk over + auto if slot free, else E to swap. Simplify: auto-pickup if slot empty; E swaps.

Mouse wheel cycles weapons. Q quick-switch? skip.

OK writing files now. For real this time. I'll count on careful writing.

After files written, I'll run node --check on game.js.