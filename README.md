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
| Punch (one-shot) | Q |
| Kick (one-shot) | X |
| Wave (one-shot) | G |
| Talk (one-shot) | T |
| Talk on phone (one-shot) | N |
| Male crouch pose — hold, press again to release | V |
| Female crouch pose — hold, press again to release | B |

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

Health bar, detection bar, ammo, current weapon, current FBX
animation state (or procedural capsule fallback), intel count and
position are shown on screen. The
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
in the repo and the game loads it on start, replacing the
procedural capsule (bob, arm swing, crouch squash) — with a clean
fallback to the capsule only if that file or its clips fail to load.
No model file was invented. The clips were made on the Mixamo "KYLE"
rig — if `Agent_Kyle.fbx` is that same rig they should apply, and if
not, playback will look wrong in-game and the rig mismatch is the
thing to check. The HUD shows whether the current character is
Agent_Kyle.fbx with FBX playback, or the procedural capsule fallback.

## Poses & animations — how playback actually works

The player now uses a real `THREE.AnimationMixer` on Agent_Kyle.fbx,
not the old stub. The capsule bob/arm-swing code runs only when no
FBX rig is playing, and it never squashes the real model
(`player.scale.y` is left at 1 while FBX is playing).

**Preloaded first (core locomotion, on model load):**
idle → Standing_Idle.fbx, walk → Walking.fbx, run → Running.fbx,
sprint → Standing_Sprint_Forward.fbx, crouchIdle → Crouch_Idle.fbx,
crouchWalk → Crouched_Walking.fbx, sneak → Sneaking_Forward.fbx.
The capsule stays visible until idle is ready, so there is no
T-pose flash; if idle fails (missing file or rig mismatch) the game
restores the capsule and keeps playing procedurally.

**Lazy-loaded on demand (first use only, then cached):**
aimIdle → Rifle_Aiming_Idle.fbx, aimWalk →
Walking_While_Aiming_Rifle.fbx, aimPistol → Pistol_Aim.fbx,
fireRifle → Firing_Rifle.fbx, fireRifleCrouch →
Firing_Rifle_Crouch_Walking.fbx, reloadRifle → Reloading_Rifle.fbx,
firePistol → Shooting_Pistol.fbx, reloadPistol →
Reloading_Walking.fbx (generic gun reload — no pistol-specific
reload was found), jump → Jump.fbx, fall → Falling.fbx,
land → Landing.fbx, crawl → Crawling.fbx, cover →
Taking_Cover_Idle.fbx, punch → Punching.fbx, kick → Side_Kick.fbx,
hitReaction → Hit_Reaction.fbx, death → Death.fbx,
deathRifle → Rifle_Death.fbx, deathFallBack →
Falling_Back_Death.fbx, knockedOut → Knocked_Out.fbx,
climb → Climbing_Ladder.fbx, vault → Vault_Over_Box.fbx,
turn → Turning.fbx, toRun → Idle_To_Running.fbx,
wave → Waving.fbx, talk → Talking.fbx,
talkPhone → Talking_On_A_Cell_Phone.fbx,
plus weapon-specific loops (pistolWalk → Pistol_Walk.fbx,
pistolRun → Pistol_Run.fbx, pistolWalkArc → Pistol_Walk_Arc.fbx,
rifleWalk → Rifle_Walk.fbx, rifleRun → Rifle_Run.fbx,
rifleSprint → Sprint_Forward.fbx, rifleCrouchWalk →
Rifle_Crouch_Walk.fbx, idleCrouchingRifle → Idle_Crouching.fbx)
and the remaining on-disk clips (headNodYes, honkingHorn,
lookingAround, openingDoor, pointingForward, shakingHeadNo,
shrugging, sittingIdle, standingUp, typing, pickingUp,
grabPistol) bound in `ANIM_MAP` for API / future triggers.

**Loops vs one-shots vs held poses:**
- *Loops* (crossfaded, ~0.18 s): idle, walk, run, sprint,
  crouchIdle, crouchWalk/sneak, crawl, cover, aimIdle, aimWalk,
  aimPistol, fall, drive, sittingIdle, typing, and the
  weapon-specific walk/run variants. The loop is chosen every
  frame from game state (movement, weapon, aiming, crouching):
  e.g. aimIdle is Rifle_Aiming_Idle for the rifle and Pistol_Aim
  for the pistol; crouchWalk is Rifle_Crouch_Walk for the rifle
  and Sneaking_Forward otherwise.
- *One-shots* (play once, then return to the current loop):
  fire, reload, jump, land (on touchdown), punch, kick,
  hitReaction (when a guard shot lands), climb, vault, enter/exit
  car, wave, talk, talkPhone, pickingUp (on intel pickup),
  grabPistol, and the other gesture clips above. Death is a
  one-shot that *holds* at its last frame (clamp, no return to
  loop). Fire/reload/hit are retriggerable — a new shot restarts
  the fire clip.
- *Held poses* (play once, clamp at the last frame, stay until
  toggled off): maleCrouchPose → Male_Crouch_Pose.fbx (key V),
  femaleCrouchPose → Female_Crouch_Pose.fbx (key B). Pressing
  the same key again releases the hold and resumes the loop.
  Poses are never looped as walk cycles.

**Root motion:** game code moves the player, so every clip has
its root-bone (Hips) XZ translation stripped to its first frame
(Y bob kept). Vault/climb therefore play in place rather than
carrying the player — that is intentional in this courtyard,
which has no ladder/vault object yet.

**Guards (experimental, secondary):** guards try the same
idle/walk clips only if their bone names match the clip tracks;
terror models are not guaranteed to share the Mixamo rig, so a
mismatched guard simply stays static. Guard mixers never affect
AI, patrols, vision cones, or hit detection (hit detection walks
the parent chain, as before), and a guard's mixer is stopped on
death. Player animation is the priority; guard playback is a
bonus and may legitimately be absent in-game.

**Fallbacks, in order:** Agent_Kyle.fbx present + clips match →
FBX playback. Agent_Kyle.fbx missing → try legacy Agent.fbx, then
capsule. Any core clip fails the bone-name check (<50% of its
tracks match the model) → that clip is rejected; if idle fails,
the capsule is restored entirely. A single lazy clip failing
never stops the game — the previous loop keeps playing.

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

- Player FBX playback is now wired (see Poses & animations above),
  but it has NOT been live-playtested: rig match, root-motion
  stripping, weapon hand placement (weapons are group-attached, not
  hand-bone attached), model facing, and guard bone compatibility
  all need an in-game check. If a clip looks wrong, the bone-name
  check / rig mismatch is the first thing to investigate, and the
  capsule fallback is still intact.
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
