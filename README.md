# SimplePlanes 2 — Carlotta Avatar Replacement / Experimental VRM Loader

An unofficial, non-commercial avatar-modding project for **SimplePlanes 2**. It began by visually replacing the default Chad pilot with **Carlotta** from *Wuthering Waves*, while retaining the game's native animation and control systems. Development has since expanded into an **experimental runtime loader for compatible VRM avatars**.

> **Development status — November 13, 2026:** Latest internal source patch: **v86.19-AH** (Windows build and in-game performance results pending). Latest performance optimization supported by user game logs: **v86.19-AG**. Internal versions are **not public releases**.
>
> **Repository scope:** Project overview, demonstrations, and engineering documentation. This repository does **not** distribute game binaries, VRM character models, or the experimental C# source.

## Project Goal

The long-term goal is to let players supply a compatible `.vrm` avatar without preparing a character-specific Unity AssetBundle. The runtime loader uses UniVRM for import and adapts the game's existing Humanoid animation, scene binding, scale, seating, hand placement, expressions, gaze, and secondary motion.

The original **Carlotta-specific replacement** remains a separate reference implementation. The **runtime VRM path** has been demonstrated with more than one VRM file, but it is not a finished plug-and-play solution: results depend on the model's Humanoid rig, material setup, expressions, SpringBone metadata, and proportions. The test overlay currently has an approximately **120-second experimental duration** rather than production-grade persistent replacement.

## Current Capabilities

| Area | Implemented or observed behavior | Qualification |
|---|---|---|
| Runtime VRM import | Load compatible `.vrm` files using UniVRM and transfer native Humanoid poses | Limited model coverage; experimental overlay |
| Scene and animation integration | Tested Main Menu, Build/Designer, Fly/Terrain, and on-foot states | Rig rebinding and seat offsets remain model/scene dependent |
| Aircraft cockpit IK | Adaptive arm targeting with reach-first torso adjustments | Proxy targets, not verified physical contact |
| Vehicle steering-wheel IK | Separate two-hand IK and live avatar scaling in 0.1× steps | Proportions affect reach and clearance |
| Expressions and gaze | ExpressionKey mapping, F6 expression cycling, blinking, seated Chase-camera eye tracking | Requires corresponding avatar capabilities |
| Secondary motion | Metadata-driven SpringBone processing; vehicle-local clothing drape when seated | No full body/seat/cloth mesh collision solver |
| Rendering | Runtime URP MToon10 shader bundle and VRM material mapping | Complete VRM0 material fidelity and outlines are not yet guaranteed |
| Performance | Frame-time/GC/module profiling and reduced redundant aircraft-control searches | AH memory-allocation optimization is pending validation |

## Gameplay Demonstrations

The following captures document the **original Carlotta-based path**. They are not claims that every external VRM reproduces these visuals exactly.

### Sight Tracking

The pilot adjusts eye and head direction in appropriate cockpit camera modes. Eye movement is kept subtle, while larger changes are handled primarily by head rotation.

<img width="400" height="300" alt="Carlotta cockpit sight tracking" src="https://github.com/user-attachments/assets/ba7407e6-5d76-482e-8b2e-3e9b9ff29248" />

### Cockpit Controls

Native control animation is retargeted to the replacement avatar, with additional seating and hand IK corrections where necessary.

<img width="400" height="302" alt="Carlotta cockpit controls" src="https://github.com/user-attachments/assets/0ce42992-84e2-48a7-a9e5-bcaa7d4d2fb1" />

### G-Force Reactions

The avatar responds visually to changes in aircraft motion. This is a procedural visual reaction, not a calibrated measurement of physical g-load.

<img width="400" alt="G-force reaction example 1" src="https://github.com/user-attachments/assets/44616d6e-659a-457a-a37b-2a822a931c7e" /> <img width="400" alt="G-force reaction example 2" src="https://github.com/user-attachments/assets/81058743-a0de-4b2c-adcd-1a7cf565ef84" />

### On-Foot Animations

The character inherits walking, swimming, and falling states from the game, with optional procedural secondary motion.

<img width="300" alt="Walking" src="https://github.com/user-attachments/assets/b7c755d1-2bcd-44fe-9199-1e446ad1175f" /> <img width="300" alt="Swimming" src="https://github.com/user-attachments/assets/095b5539-3687-466e-98e3-2b7da45ac887" /> <img width="300" alt="Diving" src="https://github.com/user-attachments/assets/ecddde16-6ea9-40ad-ab6d-c520a537498d" />

### Lobby Idle Animations

Custom idle and gesture tracks are layered with hair and clothing motion.

<img width="400" height="225" alt="Lobby idle" src="https://github.com/user-attachments/assets/b79fd936-8dc3-4f69-b677-89133b79719e" /> <img width="400" alt="Idle wave" src="https://github.com/user-attachments/assets/fe6c645c-9132-4dea-9f16-9dbb19ba12d0" />

## Documentation

- [Development History](docs/DEVELOPMENT_HISTORY.md) — architecture, investigations, test measurements, regressions, and unresolved issues.
- [Changelog](docs/CHANGELOG.md) — short, version-by-version summary of changes and validation status.

## Limitations and Attribution

This is an experimental project, not a universal compatibility guarantee or an official release of the game, its characters, or their assets. It is **not affiliated with Jundroo or Kuro Games**. Third-party character models, textures, animations, and proprietary game files are not redistributed here without the necessary permissions.
