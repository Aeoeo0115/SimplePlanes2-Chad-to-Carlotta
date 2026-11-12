# Development History and Engineering Notes

**Project:** SimplePlanes 2 — Chad/Carlotta Avatar Replacement and Experimental Runtime VRM Loader
**Last reviewed:** November 13, 2026 (KST)
**Latest internal source package:** v86.19-AH — Seating Detector Allocation Reduction (**not yet verified in-game**)
**Latest locally tested performance optimization:** v86.19-AG

> This document consolidates the former `DEVELOPMENT_PROGRESS.md` and the later development-history draft. Historical experiments and failed approaches are retained when they provide technical lessons. Versions such as `v86.19-AH` are **internal build identifiers**, not public release versions. Observations apply to tested models and environments only.

## 1. Goals, Architecture, and Scope

The project began as a visual replacement of the game's default Chad pilot with Carlotta, preserving the original game animation and behavior. It later expanded into a runtime loader for compatible VRM Humanoid models without requiring a character-specific Unity AssetBundle for each avatar.

The guiding engineering constraints are:

- Keep the game's original character rig, Animator, controls, and vehicle physics intact whenever possible.
- Retarget the native Humanoid pose to a separately imported avatar.
- Correct proportion-dependent differences through scale locking, seat/pelvis calibration, and independent aircraft/vehicle hand IK.
- Apply gaze, expressions, and secondary motion to the imported avatar without double-writing transforms already driven by the existing Carlotta pipeline.
- Use diagnostics, optional/inspect modes, reversible changes, and verified rollback paths.

### Runtime architecture

```text
SimplePlanes 2 original pilot Animator / rig
   ├─ Established Carlotta AssetBundle path (reference implementation)
   │   ├─ Native pose, idle motions, facial systems, cockpit corrections
   │   └─ Cached assets and aircraft-aware seating
   └─ Experimental runtime VRM path
       ├─ UniVRM import / Humanoid validation / rig binding
       ├─ Native pose capture and retargeting
       ├─ Scale lock / live scale / seat, foot, and pelvis alignment
       ├─ Aircraft joystick/throttle and vehicle steering-wheel IK
       ├─ Expressions, blinking, and seated Chase-camera gaze
       ├─ SpringBone motion on foot / seat-local drape in vehicles
       └─ URP material fallback or MToon10 AssetBundle shader
```

The two paths remain distinct. A successful imported-avatar overlay does not demonstrate identical behavior to the original Carlotta-specific implementation. Runtime import has been tested with multiple `.vrm` files, but extensive cross-avatar validation is incomplete. The overlay remains an **approximately 120-second experimental test mechanism**, not yet a persistent, polished avatar selector.

### Environment used for development

| Component | Development setup |
|---|---|
| Game | SimplePlanes 2, Unity 6000.3.20f1 |
| Plugin framework | BepInEx 5.4.23.6 (Mono) |
| Implementation | C#, `netstandard2.1` |
| Runtime import | UniVRM, with VRM 0.x and VRM 1.0 metadata-handling experiments |
| Rendering | Unity URP, URP/Lit or URP/Unlit fallback, external MToon10 shader bundle |
| Diagnostics | BepInEx logging, PowerShell, Unity Editor, RenderDoc |
| Example developer workspace | `C:\SP2Moding` |
| Example game location | `D:\Steam\steamapps\common\SimplePlanes 2` |
| Plugin / config | `BepInEx\plugins\SP2ChadProbe.dll`; `BepInEx\config\local.sp2.chadprobe.cfg` |
| VRM selection | `BepInEx\plugins\SP2Avatars` and the `FileName` configuration option |

These local paths are examples, not universal installation paths. The public GitHub repository intentionally contains documentation rather than the private experimental source or third-party assets.

## 2. Carlotta-Specific Foundation

### 2.1 Initial experiments: native rig, rendering, and camera behavior (v0.x–v51)

- Located the game's Chad instances and built a BepInEx-based overlay: hide the original visible renderers rather than destroying the animation source.
- Applied the original Humanoid pose to the replacement model and investigated skeleton binding, skinned meshes, facial materials, eye geometry, transparency, shader compatibility, and close-camera clipping.
- Used RenderDoc frame capture and Pixel History to investigate the face/iris/sclera depth path and determine whether geometry was missing, clipped, or incorrectly rendered.
- Kept renderer/material tests isolated to avoid masking a texture or depth issue by replacing every shader simultaneously.

**Lesson:** Shader choice, render queue, depth writes, skinning bounds, and near clipping are separate failure modes. A model can be loaded and animated correctly while its face or eyes are still invisible.

### 2.2 Gameplay and procedural animation (v52–v73)

- Integrated seating, walking, swimming, falling, and lobby/on-foot idle behaviors.
- Added expression cycling (F6), automatic blinking, limited eye/head tracking, and visual reactions to acceleration/deceleration/turning.
- Restricted gaze to appropriate cockpit/Chase contexts after Orbit view produced unnatural head/eye motion.
- Developed hair, bust, sleeve, and skirt secondary motion, with on-foot and seated policies separated over time.

**Qualification:** The G-force animation is based on motion-derived visual cues; it is not a calibrated measurement of physical g-force.

### 2.3 VRMA idle motion conversion (v74–v78)

- Converted external VRMA motions to an internal `.spmotion` format with reusable bone tracks.
- Corrected a broken rest-pose approach: a moving Chad pose was initially used as the reference, distorting the new avatar. Later tracks used a fixed initial T-pose/rest rotation and local-relative transforms.
- Addressed root/pelvis translation gaps with foot-ground and pelvis-height proxies; gated idle playback by movement and seating state.

**Regression to avoid:** Cached motion/keyframe data may be reused, but instance-specific bone bindings and rest transforms must be recomputed for each avatar.

### 2.4 Hair and clothing experiments (v79–v82.4)

- Tested interpolation, deep hair chains, motion inertia, and local angle restrictions.
- Reverted an inertia/collision change that reduced natural motion rather than improving it.
- Identified the need for model-specific rig and skinning changes when extra hair or sleeve joints were absent.

### 2.5 Scene, seat, and build compatibility (v86.3–v86.17)

| Stage | Engineering outcome |
|---|---|
| v86.3–v86.6 | MainMenu/Designer/Terrain binding and seat-pelvis alignment required scene-specific hierarchy checks |
| v86.5–v86.6 | Corrected facial close-camera clipping and Unity 6/.NET assembly reference issues |
| v86.8–v86.11 | Aircraft-aware Fly → Build seat transfer replaced a single cached offset that failed when changing aircraft |
| v86.12 | Added cache/attach diagnostics and safer build, distribution, and rollback procedures |
| v86.13–v86.14 | Cached parsed idle motions and experimented with asynchronous AssetBundle/prefab preloading |
| v86.17 | Added avatar-profile audits for required Humanoid bones, face/eye shapes, and named secondary-motion chains |

A v86.17 user test reported `requiredBones=14/14`, `springHair=16/16`, `springBust=12/12`, and `springDress=34/34` for that specific avatar profile. These are mapping results, not evidence of arbitrary model compatibility.

### 2.6 Asset loading and warm-attach performance (November 7–8)

The Carlotta AssetBundle was about **21.48 MiB** and seven converted `.spmotion` files totaled **4.18 MiB**. Repeated initialization caused noticeable transition hitches.

`PERF_ATTACH` measured **synchronous CPU elapsed time inside the avatar-attachment routine**. It did not measure the full frame time or whole-scene transition.

| Local build/test | Cold / first synchronous attach | Warm subsequent attach | Sample count |
|---|---:|---:|---|
| v86.12 Fix1 | 1,664 ms average | 97.6 ms average | Cold n=2; warm n=18 |
| v86.13 | 1,542 ms | 7.8 ms average | Cold n=1; warm n=11 |
| v86.14 with async preload | 151 ms **after preload** | 7.8 ms average | First n=1; warm n=5 |

- The warm attach improvement from 97.6 to 7.8 ms is approximately **92%**, but comes from separate local tests rather than a full frame-time benchmark.
- v86.14 moved work out of the synchronous attach: `bundleWaitMs=5528`, `prefabWaitMs=247`, `totalWaitMs=5775`. These are asynchronous wait durations, **not proven frozen-frame durations**.
- One v86.13 session recorded **77 motion-cache hits / 7 misses** over repeated attachments.
- A `startupAgeMs=-2002` trace was invalid and should not be used in startup timing comparisons.

**Outcome:** Warm attachment CPU cost was reduced. This does not establish an equivalent improvement in FPS, scene transition latency, GPU upload, or long-run memory stability.

## 3. Runtime VRM Loader Bring-Up (v86.19-A–K)

### 3.1 UniVRM dependencies, material fallback, and import (A–D)

- Added runtime `.vrm` parsing and UniVRM dependency initialization; tested importing a character without building a character-specific Unity AssetBundle.
- Used renderer, Animator, and Humanoid audits to distinguish successful import from visually correct rendering.
- Added URP shader fallbacks and alpha-mode diagnostics where the game's URP environment lacked original VRM shaders.
- Encountered managed type-loading issues inside Designer. Prewarming dependencies in MainMenu enabled later imports; the exact low-level cause was not independently isolated.
- Some runtime imports required approximately **2–5 seconds** in local sessions, a separate metric from Carlotta's `PERF_ATTACH` results.

### 3.2 Humanoid retargeting, overlays, and scenes (E–H)

- Used `HumanPoseHandler` to transfer **95 Humanoid muscle values** from the original character rig to the imported model.
- Established MainMenu, Designer (Build), and Terrain (Fly/on-foot) pilot positioning and rebind behavior.
- Kept the original rig intact and used temporary imported-avatar rendering and safe restoration.
- Began with a **20-second** test overlay; later tests extended it to approximately **120 seconds**.

| Game state | Observed scene | Example native rig context |
|---|---|---|
| Main Menu | `MainMenu` | `ChadAirlineRigged(Clone)` under a menu character container |
| Build | `Designer` | `ChadFSRigged(Clone)` near ejection-seat hierarchy |
| Fly / walking | `Terrain` | `ChadFSRigged` or `ChadAirlineRigged(Clone)` under graphics hierarchy |

Objects can disappear and reappear during transitions. The loader must wait for scene readiness and revalidate bindings; permanently caching one rig object is unsafe.

### 3.3 Initial wrist IK and scale lock (I–J)

- Introduced `Measure`/`Apply` hand-IK modes, wrist-error diagnostics, arm reach limits, and a two-bone solver.
- In an early Fly test, left wrist error fell from **18.24 to 11.20 cm**, and right error from **16.93 to 10.02 cm**. This was partial correction, not reliable contact.
- Found that frame-dependent renderer bounds changed apparent avatar size between seated and standing poses. Introduced a locked applied scale rather than recomputing scale on every rebind.
- Investigated VRM0 spring metadata and skinned-bone weights. For an early Cartethyia audit: **18/18** official spring roots matched; **8** skirt/clothing weighted candidates were not registered as official spring roots; embedded spring colliders were absent.

### 3.4 Initial SpringBone solver (K)

- Added read-only `Inspect` and experimental `Apply` modes with spring-chain mapping, damping/gravity, angle caps, and seat-state awareness.
- A local session identified **103** derived spring links from **18/18** matched official roots, recorded **901** Apply frames, and restored imported bone transforms on shutdown.
- A peak of **10°** showed the configured angle limit was reached, not that the physics was visually or physically accurate.
- Early builds required a missing Unity JSON assembly reference fix and a C# brace/syntax fix before they could compile; failed builds were not installed.

## 4. Grip, Animation, and Physics Generalization (v86.19-L–AE)

### 4.1 Aircraft controls, glass, and adaptive body posture (L–Y)

| Stage | Change / lesson |
|---|---|
| L | Surveyed joystick/throttle Renderers and Colliders; a named candidate is not verified physical contact |
| M–S | Iterated device-local hand targets, GripTarget candidates, contact-gated updates, and glass sorting/depth fixes |
| T Fix11 | Applied conditional cockpit-glass `sortingOrder` adjustment with restoration on scene changes |
| U | Added centimeter-level grip calibration in the device's local coordinate system |
| V | Evaluated minimal two-hand neutral torso assistance while rejecting large deterioration of either hand |
| W | Preferred forward-only lean, distributed over Spine/Chest/UpperChest, with smooth return |
| X | Changed to **reach-first**: arm IK before any necessary torso correction |
| Y | Reduced excessive neutral lean and refined seat height/reach geometry |

The posture strategy became **arm IK → minimum necessary torso lean → neutral recovery**. Control and palm proxies cannot guarantee fingertip or mesh-surface contact. Original aircraft control transforms are not intentionally rewritten by this solver.

### 4.2 Runtime expressions and idle source (Z Fix2)

- Replaced character-specific blendshape-index assumptions with UniVRM runtime **ExpressionKeys**.
- Restored F6 expression cycling, blinking, and inherited idle motion via the selected Carlotta animation source.
- In a local test, **8** of **18** available expression clips mapped to supported keys, including Blink/Happy/Angry/Sad/Relaxed/Surprised/BlinkLeft/BlinkRight.
- Unmapped clips were not necessarily parser failures; not every avatar supplies the same expression set.

### 4.3 Seated Chase gaze and inherited body motion (AA–AB)

- Used UniVRM LookAt where supported, active in seated **Chase** mode and disabled in Orbit/on-foot contexts.
- Avoided duplicate head rotation when inheriting a Carlotta-driven pose. A test reported `eyeFrames=1726` with `eyeFailures=0`.
- Reused existing breathing and visual acceleration reactions where possible, suppressing duplicate writes (`policy=inherited_no_double_write`).

### 4.4 Steering-wheel IK and live scale (AB–AC)

- Added separate automobile wheel identification (e.g., `Steering_Wheel/MoveRoot/Model1`) and two-hand IK, independent of aircraft control targets.
- Initial steering-wheel hand-proxy error remained around **7 cm** after IK due to model reach limitations.
- Added live scaling in **0.1×** steps, retaining the scale-lock policy and recomputing seat, ground, arm-reach, and IK references.
- Tested steering-wheel proxy errors at **1.2×**: right **0.37 cm**, left **0.88 cm**; at **1.4×**: approximately **0.46/0.44 cm**.
- These are **hand-target proxy distances**, not direct validation of fingertip contact, wheel penetration, or vehicle clearance.

### 4.5 Full spring mapping and the seated-vehicle regression (AD)

- Expanded SpringBone parsing for VRM0 `secondaryAnimation` and VRM1 `VRMC_springBone` chains; cautiously included selected additional weighted clothing chains.
- One tested avatar had **103 official links + 4 additional links = 107** total, with **0** supplied metadata colliders. On-foot motion was reported visually acceptable.
- During high-speed vehicle motion, inertial secondary motion could cause extreme garment flapping or disappearance. This was not fixed by merely increasing or decreasing damping.

### 4.6 Vehicle-local cloth drape (AE)

- While seated, computed a **seat-local kinematic drape**, aligned to the seat's local down direction rather than world gravity or aircraft acceleration.
- On foot, returned to ordinary spring motion. Sleeve limitations remained dependent on the availability and classification of separate deformable bones.
- Local logs reported `seatFrame=Seat`, `seatedKinematicFrames=3004`, and `gravityWorldAppliedWhileSeated=False`; the user also reported that the visual result worked.
- This demonstrates a controlled vehicle-local pose policy, **not** fabric/body/seat collision fidelity.

### 4.7 MToon10 material bridge (AE)

The initial external VRM texture path worked, but URP/Unlit or URP/Lit fallback did not reproduce the original toon shading. The later bridge:

1. Diagnosed base, normal, emission, transparency, render queue, and shader support per material.
2. Added `MtoonInspect` versus `MtoonBundle` modes, with URP/Lit fallback.
3. Built `VRM10/Universal Render Pipeline/MToon10` into a **Windows x64 Unity AssetBundle** named `sp2_mtoon10_urp`.
4. Loaded the bundle at runtime and mapped material descriptors for the imported VRM.

A user runtime log showed `mtoon_bundle status=ready`, `supported=True`, and **10/10 MToon10 materials** with textures, **0 unsupported material slots**, and **3 transparent slots**. The bundle file was **82,955 bytes** in that test.

**Important limitation:** Successful shader assignment does not prove complete VRM0-to-MToon10 property parity, correct ordering in every transparent mesh, or a functioning URP MToon outline RendererFeature.

### 4.8 Multiple VRMs

The tested configuration can select an external VRM by filename and restart the game. User feedback indicated another VRM could also be loaded reasonably well. A subsequent `sop2.vrm` session reported `humanoid=True`, **15/15** mapped bones, **2** skinned Renderers, **63** blendshapes, and **217** spring links (**211 official + 6 selected extra**).

These are promising reuse results, not proof of support for arbitrary VRM0/VRM1 assets or unusual rigs.

## 5. Frame-Time and Stutter Optimization (v86.19-AF–AH)

### 5.1 AF — Whole-frame and per-module profiling

The first optimization pass reduced some repeated renderer/pose work and added windowed observations of process-wide frame time, garbage collection, managed heap, and CPU time of selected mod subsystems.

On `sop2.vrm`:

| Metric | Observed range/value |
|---|---|
| Typical measured average frame time | Approximately 14–17 ms in multiple windows |
| Approximate P95 frame-time bin | 21–28 ms |
| Largest recorded frame interval | 597.36 ms |
| SpringBone CPU average (217 links) | About 0.32–0.37 ms |
| Pose-transfer CPU average | About 0.03–0.07 ms |
| Aircraft IK maximum in a window | 102.20 ms |
| Garbage collection | Some long-frame windows had `gc0Delta=1` |

This narrowed likely stutter candidates to expensive searches/IK and GC-adjacent events. It did not identify every slow frame's root cause. Module CPU timings are **not** complete GPU/frame timings.

### 5.2 AG — Missing-control search optimization (Tested)

The aircraft-grip subsystem could repeatedly search thousands of Collider objects even while the avatar occupied a vehicle with a detected steering wheel.

The fix reduced sorted-object enumeration and avoidable string work, added **2 → 4 → 8-second** negative-search backoff, reset that backoff when the rig changed, and suppressed unnecessary aircraft searches while a vehicle wheel was verified.

Observed in one AG user session:

| Counter | Result |
|---|---:|
| Full aircraft-control scene scans | 4 total |
| Aircraft searches suppressed due to recognized vehicle wheel | 7,128 frames |
| Peak full-scan time | 22.59 ms |
| Typical stable-window average frame time | Approximately 13.6–15.7 ms |
| First measured window's maximum frame delay | 166.67 ms |
| Later measured windows with frames exceeding 100 ms | None in the provided sample |

The reduced search frequency and suppressed frames are **directly supported by counters**. Claims of a controlled overall FPS improvement are **not** supported, because AF/AG came from different local runs and settings (including scale changes).

### 5.3 AH — Seat-detector allocation/cache fix (Pending)

- Replaced the per-call array-returning `GetCurrentAnimatorClipInfo()` path with reusable clip-info storage.
- Added same-frame cached seated-state results to avoid recomputation across systems.
- Added `seatCacheHits`, `seatCacheMisses`, and `seatClipListQueries` counters.
- Restricted code edits relative to AG to the seating detector and performance profiler.
- **Only source/package/static integrity checks are documented at this stage.** A successful Windows compile, in-game GC reduction, and frame-time improvement have **not** yet been demonstrated.

### 5.4 Interpretation rules

- `avgFrameMs`, `maxFrameMs`, `managedHeapMb`, and `gc0Delta` cover the game process as observed by the plugin; they are not isolated attribution to the avatar mod.
- `springAvgMs`, `aircraftIKAvgMs`, and similar module values are sampled plugin CPU costs, not GPU cost or complete game-frame time.
- A rising managed heap followed by GC does not by itself demonstrate a memory leak.
- A reduction in worst-case module samples across **different** sessions must not be reported as a controlled optimization percentage.

## 6. Known Limitations and Regression Risks

| Area | Current limitation / reason for caution |
|---|---|
| Persistent replacement | Runtime VRM overlay remains an approximately 120-second experiment; full production lifecycle is unfinished |
| Avatar compatibility | Humanoid mapping, facial keys, eye bones, spring metadata, body proportions, and shader requirements differ by model |
| Aircraft/vehicle grip | IK targets are estimated from control/proxy geometry; finger contact and precise mesh-surface constraints remain unfinished |
| Scale and seats | Locked size and dynamic refits improve consistency but do not guarantee fit in every cockpit or vehicle |
| Garment behavior | No general fabric/skin/seat collision system; missing sleeve/skirt chains cannot be synthesized safely by name alone |
| MToon | RendererFeature outlines, full VRM0 property transfer, and unusual transparency require further verification |
| Memory/frames | Asset loading, mesh upload, shader variants, full GPU cost, GC allocations, and P99 remain incompletely isolated |
| Runtime switching | Restart-based VRM selection works for testing; in-game selection, safe disposal, and per-avatar profiles are not complete |

### Lessons that should remain explicit

- **Native animation source:** keep original Chad/Carlotta transforms and vehicle physics intact where possible.
- **Rest pose:** never substitute an arbitrary animated pose for a fixed bind/T-pose reference.
- **Aircraft-specific seats:** do not reuse a Fly/Build alignment snapshot blindly across different craft.
- **Spring resets:** on-foot and seated policies must be separated; reset large time-gap or state-transition accumulators.
- **Dependency loading:** prewarm/wait for the actual scene and managed runtime readiness rather than assuming a single startup check is sufficient.
- **Caching:** distinguish per-asset reusable data from per-instance Transform/bone references.
- **Evidence:** source edits, static package checks, Windows compilation, runtime logs, visual quality, and multi-model compatibility are separate validation stages.

## 7. Next Engineering Tasks

1. **Verify AH** against AG using the same VRM, aircraft/vehicle, scale, graphics settings, test duration, and clean game launches; inspect GC and long-frame distributions.
2. **Persist the runtime avatar** safely across scenes and character rebinds; eliminate the experimental overlay timeout only after cleanup and rollback tests.
3. **Add per-avatar profiles** for scale, seat offset, arm-reach calibration, expression keys, and spring strength.
4. **Build an avatar selector** with resource reuse, explicit disposal, and repeat-load validation.
5. **Improve physical interaction** through control-surface and finger contact handling and optional body/cockpit/cloth collision constraints.
6. **Complete rendering validation** for outlines, VRM0-to-MToon10 migration, glass/alpha sorting, and other models.
7. **Expand profiling** to GPU frame time, GC allocations per frame, VRAM, shader warm-up, P99 and scene-transition spikes.
8. **Systematically test diverse legally usable VRMs**, including missing optional bones, different proportions, VRM0/VRM1 material layouts, and unusual clothing chains.

## 8. Engineering Workflow, Diagnostics, and Distribution

### Development and testing discipline

1. Prefer **Inspect (read-only) → Apply → log review → visual test → regression test**.
2. Preserve known-good baseline source and backups; do not replace game DLLs after failed builds or while the game is running.
3. Record versioned patches and change scope, then compare with a matching baseline before making performance claims.
4. Check menu, Designer, Terrain, aircraft/vehicle seating, on-foot, departure, re-entry, and avatar cleanup.

Example local diagnostics (paths are developer-specific):

```powershell
$game = 'D:\Steam\steamapps\common\SimplePlanes 2'
$log = "$game\BepInEx\LogOutput.log"
Get-Content -LiteralPath $log -Tail 120
```

Example test configuration:

```ini
[RuntimeVrmPreviewV86_19D]
FileName = sop2.vrm
MaterialMode = MtoonBundle
```

Each development package has its own build, install, inspection, and rollback scripts. Settings and script names must match the specific experimental version being tested.

### Licensing and publication scope

This is an unofficial, non-commercial research/modding project, unaffiliated with Jundroo, Kuro Games, and third-party avatar creators. No game DLLs, copyrighted VRM models, extracted proprietary assets, or third-party animation bundles should be published without appropriate rights. The GitHub repository currently serves as documentation of engineering progress; internal source archives and runtime dependencies are not automatically cleared for public redistribution.

## 9. Summary

The project progressed from a character-specific Carlotta replacement into an experimental runtime system capable of importing compatible VRMs and connecting them to native animation, cockpit and steering-wheel IK, gaze, expressions, on-foot SpringBone motion, vehicle-local drape, and MToon10 rendering.

The **MToon10 binding** and **AG search optimization** have supporting local runtime logs. The latest **AH seat-detector allocation patch is pending Windows build and in-game performance validation**. Broad plug-and-play compatibility, permanent runtime replacement, complete material fidelity, and full physical collision handling remain future work.
