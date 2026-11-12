# Development Changelog

> Last updated: **November 13, 2026 (KST)**. This covers **internal engineering builds**, not public releases.
>
> **Tested:** observed in user game logs and/or reported visually. **Experimental:** limited or partial verification. **Pending:** source/package prepared but no Windows build and in-game result provided yet.

For implementation background, numeric evidence, and known limitations, see the [Development History](DEVELOPMENT_HISTORY.md). This file intentionally records **changes and status only**; detailed investigations belong in the history.

## v86.19-AH — Seating Detector Allocation Reduction (Pending)

- Replaced repeated Animator clip-info array creation with reusable storage and a per-frame seated-state cache.
- Added `seatCacheHits`, `seatCacheMisses`, and `seatClipListQueries` counters.
- Changed the seat detector and performance profiler only; Windows build and performance impact remain **unverified**.

## v86.19-AG — Aircraft Control Search Stutter Fix (Tested)

- Reduced repeated full-scene control searches, including search work while a steering wheel is already recognized.
- Added 2 → 4 → 8-second negative-search backoff and rig-change reset.
- In a local test: **4** total aircraft-control scans and **7,128** suppressed search frames; some frame spikes remain.

## v86.19-AF — Frame-Time and GC Profiling (Tested)

- Added windowed frame-time, GC, managed-heap, and per-module CPU measurements.
- Reduced redundant renderer and pose-related work.
- Exposed large isolated frame spikes and search/IK/GC candidates without yet identifying every cause.

## v86.19-AE — MToon10 / Material Compatibility (Tested)

- Introduced URP/Lit material diagnostics, a Unity-built Windows MToon10 AssetBundle, runtime loading, and fallback.
- Observed **10/10** mapped MToon10 materials on the tested VRM; exact VRM0 shading and outline fidelity remain open.

## v86.19-AE — Vehicle-Local Seated Drape (Tested)

- Stabilized clothing in the seat's coordinate frame while seated; restored ordinary secondary motion when on foot.
- Removed inappropriate world-gravity/inertial behavior during high-speed vehicle movement.
- Did not add a body, cockpit, or cloth mesh-collision solver.

## v86.19-AD — Expanded SpringBone Handling (Tested; Seated Regression Addressed in AE)

- Extended metadata-driven spring handling to registered chains and selected extra weighted chains.
- On-foot motion was reported to work; vehicle-speed flapping led to the AE seated-drape policy.

## v86.19-AC — Live Scale and Seat/IK Refit (Tested)

- Added 0.1× live avatar scaling and recalculation of seat, foot, arm reach, and hand targets.
- Reduced steering-wheel proxy hand-target errors to below **1 cm** at tested scale factors; physical palm/finger contact was not verified.

## v86.19-AB — Vehicle Steering-Wheel IK and Body-Reaction Policy (Tested)

- Added separate vehicle two-hand IK using detected wheel geometry/reference palms.
- Prevented duplicate breathing/G-force rotations when inheriting the Carlotta animation source.

## v86.19-AA — Universal Cockpit Gaze (Tested)

- Integrated UniVRM LookAt for the imported avatar, active in seated Chase view but not Orbit view.
- Prevented duplicate head rotation with inherited Carlotta poses.

## v86.19-Z Fix2 — Runtime Expressions and Idle Retargeting (Tested)

- Mapped VRM ExpressionKeys rather than hard-coded character-specific blendshape indices.
- Restored F6 expression cycling, blinking, and imported-avatar idle source selection.

## v86.19-U–Y — Cockpit Grip and Reach-First IK (Iterative Testing)

- Added pilot-control-relative hand calibration, constrained two-hand torso assistance, forward-only lean, reach-first correction, and seat-neutral tuning.

## v86.19-L–T — Control Geometry and Rendering Investigations (Iterative Testing)

- Investigated joystick/throttle renderers and colliders, contact-gated grips, and glass sorting/depth issues.
- Added conditional glass `sortingOrder` handling with restoration on scene change.

## v86.19-A–K — Runtime VRM Import Bring-Up (Experimental, Locally Tested)

- Implemented UniVRM import, shader fallback, Humanoid pose transfer, temporary menu/Build/Fly overlays, scene and rig rebinding, and spring metadata diagnostics.
- Introduced scale locking, read-only motion inspection, and optional spring motion on imported VRM bones.
- Original ~20-second preview was later extended to an approximately **120-second experimental overlay**.

## v86.15–v86.17 — Avatar Profile Validation

- Added Humanoid, face/eye, and named secondary-motion profile diagnostics, preserving previous render and seat adjustments.

## v86.12–v86.14 — Attachment / Asset Preload Performance

- Added asset retention, parsed motion caching, and experimental asynchronous bundle/prefab preloading.
- Reduced repeated **synchronous attach CPU time**, with separate frame-time validation still needed.

## v86.3–v86.11 — Scene and Seat Stabilization

- Addressed Main Menu/Designer/Terrain rig binding, face near-clipping, aircraft-aware Build ↔ Fly seat alignment, and assembly compatibility.
- Improved build, backup, deployment, and rollback tooling.

## Earlier v0.x–v82.4 — Prototype, Animation, and Physics

- Developed Chad/Carlotta visual replacement, native Humanoid animation retargeting, eye/face rendering, expressions, gaze, vehicle reactions, and on-foot/lobby motions.
- Converted VRMA tracks to internal motion data, corrected rest/T-pose offsets, and iteratively tuned secondary motion.
