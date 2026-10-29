# Double Agent — Stealth Prototype

An original third-person, over-the-shoulder sneaking / tactical espionage
prototype. It borrows only generic stealth mechanics (vision cones, a
detection meter, alert / caution / search states, noise, cover and silent
takedowns). It contains no characters, names, story, music, dialogue,
logos or assets from Metal Gear Solid or any other existing game.

## Play it

No build step. This is a static site, ready for GitHub Pages.

Locally, serve the folder (module scripts and the manifest fetch do not
work reliably from `file://`):

```sh
cd double-agent
python3 -m http.server 8000
# open http://localhost:8000/
```

Then click **Click to infiltrate**. Clicking locks the mouse. Press `Esc`
to release it.

## Objective

1. Collect all 3 glowing yellow intel documents in the courtyard.
2. Reach the green EXFIL gate in the far corner (38, -38) with all 3.
3. Avoid being seen. If your health reaches 0, the mission fails.

Guards patrol waypoints. Security cameras sweep. Walls and crates block
line of sight — use them.

## Controls

| Action | Key / mouse |
|---|---|
| Move | W A S D |
| Look | Mouse (pointer lock) |
| Sprint (loud) | Shift |
| Crouch / sneak (quieter, slower) | C (toggle) |
| Jump | Space |
| Aim | Hold right-mouse |
| Fire | Left click |
| Reload | R |
| Pistol | 1 |
| Rifle | 2 |
| Interact / pick up intel / exfil | E |
| Silent takedown | F, from behind an unaware guard, within ~2.8 m |
| Toggle player loadout (white / black set) | L |

Silent start is preserved: the player spawns at (-38, 38) in a walled
alcove in the southwest corner, facing northeast. The nearest guard
spawn is 42 m away, the nearest camera 33.5 m, the nearest patrol
waypoint 38.8 m — nothing can see or hear you at the start.

Control clash note: the original brief listed both "F aim" and
"F takedown". Those cannot both be F. This build resolves it as
right-mouse = aim, F = takedown only.

## How the stealth works

- **Vision:** each guard has a yellow cone, ~19 m range, ~75° field.
  Walls block sight (raycast line-of-sight check). Crouching slows
  detection gain.
- **Detection meter:** rises while you are seen, falls when hidden.
  Under ~25: patrol. 25–99: suspicious / caution. 100: alert.
- **Alert:** the guard chases your last-known position and shoots
  when it has line of sight.
- **Search:** after ~4 s without sight, an alert guard searches your
  last-known position for ~6 s, then returns to patrol.
- **Caution / suspicious:** the guard turns toward you but does not
  yet chase.
- **Noise:** sprinting near a guard raises its meter; gunfire alerts
  guards within ~26 m to your position.
- **Cameras:** orange cones, ~16 m. Being seen by a camera raises the
  global detection meter; at 100, all guards go alert.
- **Takedown:** only from behind (guard facing away from you) while
  the guard is not alert and its meter is under 80. Alert guards
  cannot be taken down.

## HUD and minimap

Health bar, detection bar, ammo, current weapon, current procedural
animation state, intel count and position are shown on screen. The
minimap (top right) shows walls, guards (yellow = patrol/suspicious,
red = alert), cameras (orange), uncollected intel (yellow) and the
exfil gate (green). You are the green arrow.

## Animations — read this before assuming anything

`assets/animations/` contains **62 real Mixamo FBX files**, copied
byte-for-byte from `~/workspace/double-agent/animations/` with their
exact filenames (35 MB total). They are FBX Binary, Skin: Without
Skin (animation-only, no mesh), 30 FPS, rigged for Kyle's own Mixamo
upload shown on Mixamo as "KYLE".

`assets/animation-manifest.json` maps game actions to the correct
file, for example:

- idle → Standing_Idle.fbx
- walk → Walking.fbx, run → Running.fbx, sprint → Standing_Sprint_Forward.fbx
- crouchIdle → Crouch_Idle.fbx, crouchWalk → Crouched_Walking.fbx
- aimIdle → Rifle_Aiming_Idle.fbx, aimWalk → Walking_While_Aiming_Rifle.fbx
- fireRifle → Firing_Rifle.fbx, reloadRifle → Reloading_Rifle.fbx
- firePistol → Shooting_Pistol.fbx, reloadPistol → Reloading_Walking.fbx
  (generic gun reload — no pistol-specific reload was found on Mixamo)
- sneak → Sneaking_Forward.fbx, crawl → Crawling.fbx, cover → Taking_Cover_Idle.fbx
- drive → Driving.fbx, enterCar → Entering_Car.fbx, exitCar → Exiting_Car.fbx

**Player model, stated plainly:** `assets/Agent_Kyle.fbx` (1.6 MB) is
now in the repo and the game loads it on start, replacing the
procedural capsule (bob, arm swing, crouch squash) — with a clean
fallback to the capsule only if that file fails to load. No model
file was invented. `index.html` contains a loader
(`tryLoadFBXAnimations`) that feeds the matching FBX clip to
Agent_Kyle.fbx. The clips were made on the Mixamo "KYLE"
rig — if `Agent_Kyle.fbx` is that same rig they should apply, and if
not, playback will look wrong in-game and the rig mismatch is the
thing to check. The HUD shows whether the current character is
Agent_Kyle.fbx or the capsule fallback.

## Asset map — which real models are wired in

**Weapon split (user-corrected, standing): weapons WITHOUT a `white_`
or `black_` prefix are for the ENEMIES. The player's weapons are the
`white_` and `black_` versions — two different player sets.**

| Role | File(s) | How it is used |
|---|---|---|
| Player character | `assets/Agent_Kyle.fbx` | **Present in the repo (1.6 MB).** The game tries to load it on start; if it is present it replaces the procedural capsule, is normalised to ~2.2 m tall with feet on the ground, and the Mixamo animation clips are fed to it via `tryLoadFBXAnimations`. If it is missing, the game falls back cleanly to the capsule — no error screen, no broken game. **To make it appear: the real `Agent_Kyle.fbx` file is at `assets/Agent_Kyle.fbx` and reload. No code change is needed, and no file was invented in its place.** |
| Guards 1–5 | `assets/enemy-terror1.fbx`, `enemy-terror2.fbx`, `enemy-terror3.fbx`, `enemy-terror4.fbx`, then terror1 again for guard 5 | Each guard's red capsule body/head is hidden and the FBX model is added to the same guard group, so position, yaw, vision cone, patrols and AI are unchanged. If a guard's FBX fails to load, that guard keeps its capsule. |
| Player loadout 1 (white set) | `assets/white_glock.glb` (pistol, key 1), `assets/white_mp5k.glb` (rifle, key 2) | Loaded with GLTFLoader, attached to the player group, toggled by weapon. Press **L** to switch to the black set. |
| Player loadout 2 (black set) | `assets/black_glock_threaded.glb` (pistol), `assets/black_mp5k.glb` (rifle) | Same as above, for the black set. |
| Other player-set variants (in assets, not assigned to a slot yet) | `white_walther.glb` / `black_walther.glb`, `white_makarov.glb` / `black_makarov.glb`, `white_aks74u.glb` / `black_aks74u.glb`, `white_RPG7.glb` / `black_rpg.glb`, plus `white_` / `black_` suppressors, knives, etc. | Present in `assets/` but not wired to a key yet — they are the same two player sets' alternate weapons/gadgets, not enemy weapons. Swap them into `PLAYER_LOADOUTS` in `index.html` to use them. |
| Enemy weapons | `assets/mp5k.glb` (unprefixed) | Attached to every guard as a visual rifle. Guard shooting logic is unchanged (timed damage tick, not a projectile). The other unprefixed weapons (`ppk.glb`, `aks74u.glb`, `rpg.glb`, etc.) are enemy weapons in assets but are **not** assigned to the player and are not currently attached to guards. |
| Props / gadgets (not yet wired) | `white_lambo.glb`, `tire.glb`, `tire2.glb`, bags, C4, claymore, grenades, glasses, watch, pen, lockpicks, cufflinks, and the remaining `white_` / `black_` variants | Present in `assets/` only. The courtyard still uses its procedural car prop, not `white_lambo.glb`, and gadgets have no gameplay yet. |

**Scaling / rotation assumptions Kyle should check in-game:**
characters are normalised by bounding box to ~2.2 m tall, feet at
ground level, centred on their group. The player model is rotated PI
inside its group so it faces away from the over-shoulder camera
(guard models face the same way as their vision cone, so they get no
extra rotation). Weapons are normalised so their longest dimension
lies along Z at ~0.32 m (pistols) / ~0.60 m (rifles) and are centred
on the hand pivot. The barrel-vs-stock direction cannot be known
from the file alone — if a weapon points backwards, or a character
faces the wrong way, flip that model's yaw in `index.html`
(`normalizeCharacter`'s yaw offset for characters, `fitWeapon`'s
rotation for weapons).

## Known gaps / simplifications

- `Agent_Kyle.fbx` is now in the repo, so the player loader uses it;
  `assets/` the player is still a procedural capsule (guards and
  weapons above are real models, with capsule / box-gun fallbacks).
  No model file was invented to fill that gap.
- No vehicle is drivable yet: the courtyard has a parked car prop,
  and the drive / enter / exit FBX clips are mapped for a future
  vehicle system, not wired to driving gameplay.
- Switching to the rifle currently sets its magazine to 30 in this
  prototype; per-weapon magazine persistence is a small follow-up.
- Missing Mixamo clips (no result found, nothing substituted with
  unrelated art): Pistol Crouch Walk, Pistol Crouch Fire, Checking
  Watch, a distinct Driving Turn, Motorcycle Riding / Driving.
- Guard shooting is a simple timed damage tick with line of sight,
  not a simulated projectile. No sound, no save system, no
  multiplayer, no purchases, no fake social features.
- Requires an internet connection for the Three.js CDN import map.
  The FBX files themselves are local.

## Files

- `index.html` — the whole game, single file, Three.js via CDN import map
- `assets/animation-manifest.json` — action → FBX mapping, plus notes
- `assets/animations/*.fbx` — 62 real Mixamo animation-only clips
- `assets/enemy-terror1–4.fbx` — real guard models (see Asset map)
- `assets/white_*.glb` / `assets/black_*.glb` — player weapon sets
- `assets/mp5k.glb` (unprefixed) and other unprefixed GLBs — enemy weapons / props
- `assets/Agent_Kyle.fbx` — **present**; player model, loaded with fallback to capsule (see Asset map)
