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

**Known limitation, stated plainly:** the character you see in the
prototype is a procedural capsule (bob, arm swing, crouch squash).
The FBX clips are **not** playing on that capsule, and no KYLE model
file is included or invented in this repo. `index.html` contains a
loader stub (`tryLoadFBXAnimations`) that will load the matching
FBX clip only when a compatible rigged model is supplied later;
with no rigged model present it falls back cleanly to the capsule.
The HUD shows the FBX file that *would* correspond to the current
action ("FBX ready: …") — that is a mapping label, not playback.

## Known gaps / simplifications

- No rigged character model is included, so FBX playback is a stub
  (see above). No model file was invented to fill that gap.
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
