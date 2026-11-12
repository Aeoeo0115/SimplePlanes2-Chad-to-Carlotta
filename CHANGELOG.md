# Development Changelog — SimplePlanes 2 Universal VRM Mod

> Updated: 2026-11-13 (KST)  
> **Internal experimental version history.** Do not confuse these entries with the public release version (`v0.1`).  
> Status labels: **Tested** = user game log and/or visual confirmation; **Experimental** = partial/limited validation; **Pending** = source/packaging only, no user game result yet.

## v86.19-AH — Seat Detector GC Fix1 (Pending)

- Replaced repeated `GetCurrentAnimatorClipInfo()` array allocations with reusable clip-info storage and per-frame seating-state cache.
- Added `seatCacheHits`, `seatCacheMisses`, and `seatClipListQueries` profiling counters.
- Changed only `CarlottaSeatDetector.cs` and `RuntimeVrmPerfAf.cs` relative to AG; all other functionality retained.
- **Windows build and actual GC/frame-time results not yet verified.**

## v86.19-AG — Grip Search Stutter Fix1 (Tested)

- Reduced unnecessary full-scene searches for missing aircraft `Joystick/GripTarget` controls.
- Applied 2s → 4s → 8s negative-search backoff, reset on rig rebind; skipped aircraft searches when vehicle wheel was confirmed.
- Observed: 4 total aircraft control scans; 7,128 suppressed frames; peak scan 22.59ms.
- Stable-window frame averages about 13.6–15.7ms; rare spikes remain.

## v86.19-AF — Performance Pass 1 (Tested profiler)

- Cached repeated renderer/pose work and added whole-frame/GC plus module CPU timing diagnostics.
- Profiling on `sop2.vrm`: SpringBone 217 links, ~0.32–0.37ms typical module cost.
- Detected transient frame spikes up to 597ms and aircraft IK sampling up to 102ms; cause not completely isolated.

## v86.19-AE — MToon10 Bundle Bridge (Tested)

- Compiled URP MToon10 in Unity Editor as `sp2_mtoon10_urp` AssetBundle and loaded it at runtime.
- Supported Inspect/fallback to URP/Lit and MToon10 material mapping for newly imported VRMs.
- Observed: bundle `status=ready`, shader supported, 10/10 materials mapped, 10/10 textured, 0 unsupported slots.
- Full VRM0-to-MToon10 parity and outline pass remain unverified.

## v86.19-AE — Material Compatibility (Tested)

- Compared URP/Unlit vs URP/Lit; audited shader, base/normal/emission textures, transparency and render queue.
- Preserved existing Carlotta materials and other avatar systems.

## v86.19-AE — Vehicle Local Drape (Tested)

- Seated: replaced world-inertial secondary motion with seat-local kinematic clothing drape, following vehicle floor direction during roll/pitch.
- On-foot: restored normal SpringBone motion.
- Observed `seatedKinematicFrames=3004`, `seatFrame=Seat`, `gravityWorldAppliedWhileSeated=False`.
- Real mesh/seat collision is not simulated.

## v86.19-AD — All Movable Spring (Tested, with seated regression)

- Generalized SpringBone mapping for registered VRM0/VRM1 chains, plus optional weighted clothing chains.
- Tested example: 103 registered + 4 auxiliary links; metadata colliders 0.
- On-foot motion worked; seated speed-dependent flapping prompted AE local drape fix.

## v86.19-AC — Live Scale & Seat/Grip Refit (Tested)

- Hotkeys for 0.1× VRM size adjustments, reset/save; recalculated seat/foot references and arm reach/IK.
- Vehicle IK after-error at factor 1.2×: right 0.37cm, left 0.88cm; at 1.4×: ~0.46/~0.44cm.
- Figures are proxy hand target errors, not verified physical grip.

## v86.19-AB — Vehicle Wheel Grip (Tested)

- Added `Steering_Wheel` identification and separate vehicle two-hand IK, while keeping aircraft joystick/throttle logic unchanged.
- Original hand target error ~13–14cm, reduced to ~7cm; remaining arm reach limitation motivated AC.

## v86.19-AB — Universal Body Reaction (Tested inheritance policy)

- Reused inherited Carlotta breathing/visual acceleration response without double-rotating the imported VRM.
- Observed `policy=inherited_no_double_write`, zero added writes/exceptions for `Source=Carlotta`.

## v86.19-AA — Universal Cockpit Gaze (Tested)

- Added UniVRM LookAt integration, Chase-and-seated-only activation, Orbit disable, Carlotta-inherited-head duplicate prevention.
- Observed 1,726 eye frames and zero eye driver errors in user test.

## v86.19-Z Fix2 — Expressions & Idle Source (Tested)

- Replaced character-specific BlendShape mapping with UniVRM Runtime ExpressionKeys.
- Restored F6/automatic blink and Carlotta pose/VRMA idle source for imported VRM.
- Observed 8 mapped expressions out of 18 available clips.

## v86.19-U~Y — Adaptive Aircraft Cockpit IK (Iterative)

- U: device-local hand grip calibration.
- V: minimum-cost neutral dual-hand torso lean.
- W: forward-only distributed Spine/Chest lean and smooth return.
- X: reach-first, arm IK before torso correction.
- Y: seat fit and reduced neutral overlean.

## v86.19-A~T — Runtime VRM Bring-up / Scene / Physics / Glass (Iterative)

- Runtime UniVRM `.vrm` loading, MainMenu prewarm, Humanoid pose transfer, Terrain/Designer overlay, scene binding and rebind safety.
- Two-bone hand diagnostics, VRM SpringBone metadata/skin weights, joystick/throttle survey and glass `sortingOrder` safety fix.
- Experimental overlay duration expanded from 20s to 120s; permanent replacement remains planned.

## v86.3~v86.17 — Carlotta Foundation (Iterative)

- Fixed Designer/Build/Fly seat alignment, aircraft-aware transfer, face/eye skinning and near clipping, `.NET Standard 2.1` Unity6 build compatibility, release/rollback scripts and profile audit.

## Earlier v0.x~v82.4 — Initial Prototype and Animation/Physics Work

- Chad/Carlotta replacement with Humanoid pose retargeting and non-destructive original-rig hiding.
- RenderDoc eye/face material investigations, Chase-only cockpit gaze, reactions and on-foot/lobby animations.
- VRMA-to-spmotion conversion, T-pose-corrected tracks, foot contact and hair/cloth dynamics tuning.

See [the full development history](docs/DEVELOPMENT_HISTORY.md) for implementation rationale, user-observed measurements, outstanding limitations, and future tasks.
