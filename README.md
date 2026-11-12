# SimplePlanes 2 — Carlotta Avatar Replacement Mod

A custom VRM avatar replacement project for SimplePlanes 2.

The project began as a Carlotta replacement for the default Chad character and now also supports experimental runtime loading of compatible VRM humanoid avatars, while reusing the game's character animations.

## Project Vision — Universal VRM Avatar Support

The ultimate goal of this project is to evolve from a Carlotta-specific character replacement into a **universal VRM avatar loader for SimplePlanes 2**.

Ideally, users will be able to simply place any compatible `.vrm` file into a designated folder, and the mod will automatically recognize, load, and adapt the avatar to the game's existing animation system.

This includes automatic skeleton mapping, body proportion adjustments, IK calibration, cockpit seating alignment, and support for facial expressions and secondary motion where available.

**Carlotta remains the reference prototype** for developing and testing these systems alongside the experimental runtime VRM path. The long-term objective is to make custom avatar replacement as close to plug-and-play as possible, without requiring manual rigging or per-character configuration.

## Features

- Humanoid animation retargeting
- Eye and head tracking
- Cockpit controls and hand IK
- Acceleration, deceleration, and lateral G-force reactions
- Walking, swimming, and falling animations
- Custom idle animations for on-foot and lobby states
- Procedural hair and clothing physics
- Facial expressions and automatic blinking
- Avatar replacement across Main Menu, Build, Fly, and On-Foot modes

## Gameplay Demonstrations

### Sight Tracking
  The avatar dynamically adjusts its eye and head orientation based on the camera's viewing direction while inside the cockpit. Eye movement is kept subtle, while larger directional changes are handled primarily by head rotation. Tracking is limited to appropriate camera modes to prevent unnatural movement.

<img width="400" height="300" alt="sight2" src="https://github.com/user-attachments/assets/ba7407e6-5d76-482e-8b2e-3e9b9ff29248" />


### Cockpit Controls
  The mod adapts Chad's original cockpit control animations to Carlotta's humanoid skeleton, allowing her to interact with the aircraft's controls. Hand and arm positions are adjusted using inverse kinematics (IK) to accommodate differences in body proportions. The system also maintains proper seating alignment across different aircraft and when switching between Build and Flight modes.

<img width="400" height="302" alt="control" src="https://github.com/user-attachments/assets/0ce42992-84e2-48a7-a9e5-bcaa7d4d2fb1" />

### G-Force Reactions
  The avatar responds to aircraft acceleration, deceleration, and lateral G-forces by dynamically adjusting its body posture. These reactions are layered on top of the existing cockpit animations, making the character lean and shift naturally during maneuvers rather than remaining rigidly seated.

<img width="400" src="https://github.com/user-attachments/assets/44616d6e-659a-457a-a37b-2a822a931c7e" /> <img width="400" alt="gforce2" src="https://github.com/user-attachments/assets/81058743-a0de-4b2c-adcd-1a7cf565ef84" />

### On-Foot Animations
  The mod adapts the game's existing character animations to Carlotta's humanoid skeleton, supporting walking, swimming, and falling. Animation retargeting allows the replacement avatar to follow the original movement states, while procedural secondary motion adds natural movement to hair and clothing.

 <img width="300" alt="walk" src="https://github.com/user-attachments/assets/b7c755d1-2bcd-44fe-9199-1e446ad1175f" /> <img width="300" alt="swim" src="https://github.com/user-attachments/assets/095b5539-3687-466e-98e3-2b7da45ac887" /> <img width="300" alt="dive" src="https://github.com/user-attachments/assets/ecddde16-6ea9-40ad-ab6d-c520a537498d" />

### Lobby Idle Animations
  The custom avatar remains animated in the main menu instead of appearing as a static model. Idle gestures, including waving, are combined with procedural hair and clothing motion to maintain a natural appearance outside of gameplay.

<img width="400" height="225" alt="lobby" src="https://github.com/user-attachments/assets/b79fd936-8dc3-4f69-b677-89133b79719e" /> <img width="400" alt="wave" src="https://github.com/user-attachments/assets/fe6c645c-9132-4dea-9f16-9dbb19ba12d0" />

## Development Progress — Universal VRM Support (November 2026)

The project has progressed from a Carlotta-specific avatar replacement prototype to an experimental **runtime Universal VRM avatar system**. Compatible VRM files can be imported at runtime and driven by SimplePlanes 2's existing animation system, with additional body-size adjustment, cockpit/vehicle IK, facial expressions, camera-aware gaze, secondary motion, and MToon10 material support.

### Highlights

- **Runtime VRM import and retargeting:** Support for compatible Humanoid VRMs using UniVRM, while preserving the original Chad/Carlotta rig.
- **Aircraft and vehicle interaction:** Aircraft joystick/throttle hand targets, two-hand steering-wheel IK, reach-first torso posture, and seat alignment.
- **Live avatar scaling:** 0.1× increments with recalculated arm reach, seating and foot alignment proxies.
- **Expressions and gaze:** UniVRM expressions, blinking, and Chase-only cockpit eye tracking.
- **Secondary motion:** VRM SpringBone chains for on-foot motion and vehicle-local, non-inertial cloth drape while seated.
- **MToon10 materials:** Unity-built URP shader AssetBundle loaded at runtime; one test avatar successfully mapped 10/10 materials to MToon10.
- **Profiling and optimization:** Frame-time/GC/module profiling, reduced redundant aircraft-control searches, and further seat-detection allocation improvements under test.

**Development status:** The latest internal source package is `v86.19-AH` (GC optimization; in-game validation pending). The latest performance patch confirmed in user logs is `v86.19-AG`. This is an experimental development build, not a stable public release. The runtime overlay still has an experimental duration limit, and complete compatibility with all VRM models, full MToon appearance parity, mesh-based cloth collisions, and finger grip are not yet guaranteed.

For technical details and validation notes, see [Development History](docs/DEVELOPMENT_HISTORY.md) and [Changelog](docs/CHANGELOG.md).

> **Asset notice:** Third-party character models, textures, animations, game DLLs, and proprietary bundles are not included for redistribution without appropriate permissions.

## Disclaimer

This is an unofficial, non-commercial fan project. It is not affiliated with Jundroo or Kuro Games.

Third-party character and animation assets are not distributed without the appropriate permissions.
