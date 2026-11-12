# SimplePlanes 2 — Chad/Carlotta → Universal VRM Avatar Mod: Development History

> 개발 기록 갱신: **2026-11-13** (KST)
> 대상: *SimplePlanes 2*의 Chad/Carlotta 애니메이션을 이용한 비공식 VRM 아바타 교체 모드
> 공개 저장소: [Aeoeo0115/SimplePlanes2-Chad-to-Carlotta](https://github.com/Aeoeo0115/SimplePlanes2-Chad-to-Carlotta)
> 내부 실험 소스 최신 패키지: **v86.19-AH Seat Detector GC Fix1**
> 실제 게임 로그로 검증된 최신 성능 최적화: **v86.19-AG**
> **주의:** GitHub에 공개된 `v0.1` 표기와 내부 개발 버전 `v86.19-*`는 서로 다른 버전 체계다. 이 문서는 곧바로 공개 릴리스가 되었음을 의미하지 않는다.

## 1. 프로젝트 개요

### 1.1 목표

이 프로젝트는 SimplePlanes 2의 기본 조종사 캐릭터(Chad)를 다른 Humanoid/VRM 캐릭터로 시각적으로 대체하면서 **게임의 기존 조종·보행·착석 애니메이션은 그대로 이용**하기 위해 시작했다. 첫 번째 구현 대상은 *Wuthering Waves*의 Carlotta였으며, 이후 임의의 호환 가능한 `.vrm`을 실행 중 불러오는 **Universal VRM Avatar** 구조로 범위를 확장했다.

개발의 기본 원칙은 **원본 Chad/Carlotta 리그와 게임의 조종 장치·물리 객체를 가급적 수정하지 않는 것**이다. 원본 애니메이션의 포즈를 새 VRM의 Humanoid 본에 전달하고, 체형 차이로 발생하는 손·골반·발 오차를 별도의 IK와 착석 보정으로 해결한다. 표정·시선·의상 물리·셰이더는 새로 임포트한 아바타에 한정해 적용하며, 실패 시 원본 렌더러와 설정을 되돌릴 수 있게 한다.

### 1.2 개발 환경

| 항목 | 구성 |
|---|---|
| 게임 | SimplePlanes 2, Unity **6000.3.20f1** 기반 |
| 모드 프레임워크 | BepInEx **5.4.23.6**, Mono |
| 언어 / 타깃 | C#, `netstandard2.1` |
| VRM 런타임 | UniVRM 기반 Runtime Import, VRM 0.x / 1.0 메타데이터 대응을 단계적으로 추가 |
| 렌더링 | Unity URP, `Unlit`/`Lit` 호환 처리 및 외부 AssetBundle의 **MToon10** |
| 분석 | PowerShell 로그 수집, C# 진단 컴포넌트, Unity Editor, RenderDoc |
| 로컬 개발 폴더 | `C:\SP2Moding` |
| 로컬 게임 폴더 | `D:\Steam\steamapps\common\SimplePlanes 2` |
| 핵심 플러그인 | `BepInEx\plugins\SP2ChadProbe.dll` |
| 설정 | `BepInEx\config\local.sp2.chadprobe.cfg` |
| VRM 파일 폴더 | `BepInEx\plugins\SP2Avatars` |

위 경로는 개발자의 로컬 환경 예시로, 다른 사용자의 설치 경로는 다를 수 있다.

### 1.3 런타임 처리 구조

```text
SimplePlanes 2 native animation / Animator
   └─ Chad / Carlotta source rig (원본 상태 보존)
       └─ Humanoid pose capture / retargeting
           └─ Imported VRM (UniVRM runtime GameObject)
               ├─ Body scale / pelvis and seat alignment / foot ground proxy
               ├─ Aircraft control IK / vehicle steering-wheel two-hand IK
               ├─ VRM expressions / blinking / Chase-only gaze
               ├─ SpringBone (on-foot) / vehicle-local cloth drape (seated)
               └─ URP MToon10 shader/material mapping
```

정상 실행 경로에서는 **포즈 전달 → 위치·크기 정렬 → 팔/상체 IK → 눈/표정 → 보조 물리/렌더링** 간 업데이트 순서를 조정해 서로의 Transform을 덮어쓰지 않도록 했다. 실제 갱신 순서는 기능별 Unity 스크립트 실행 순서와 런타임 상태에 의존하므로 새로운 모듈 추가 시 반드시 확인해야 한다.

---

## 2. 초기 개발: Carlotta 전용 모드

### 2.1 v0.x — 게임 내 캐릭터 교체와 렌더링 기반 확보

- 게임 씬과 캐릭터 계층을 분석하고 BepInEx 플러그인에서 원본 Chad 인스턴스를 탐색했다.
- 원본 리그를 제거하는 대신 렌더링을 숨기고 다른 Humanoid 아바타에 포즈를 복사하는 방식을 채택했다.
- 초기 문제는 스킨 메시, 핑크/검은 셰이더, 흰자와 홍채, 재질 투명도, 카메라 근접 시 얼굴이 사라지는 현상이었다.
- v0.28~v0.51 구간에서는 얼굴/눈 Renderer를 분리하여 셰이더, 깊이, ZWrite, RenderQueue, 스키닝 Bounds를 조사했다. RenderDoc의 캡처 및 Pixel History를 이용해 실제 렌더링 단계를 확인했다.
- 여러 원인 후보를 확인하기 전에는 **재질 전체를 한 번에 교체하지 않고**, 모델·재질 슬롯·렌더러를 개별 시험하는 원칙을 확립했다.

### 2.2 v52~v73 — 게임 동작과 연결

- 착석 및 보행에서 원본 애니메이션을 새 아바타로 전달했다.
- 머리카락, 치마, 가슴, 소매의 2차 움직임과 기체 조작 시 시각적 가속 반응을 추가했다.
- 조종석의 눈·고개 추적을 구현한 뒤, 부자연스러운 Orbit 카메라 반응을 막기 위해 **Chase에서만 추적**하도록 조건을 좁혔다.
- F6 표정 순환, 자동 눈깜빡임, F8의 원본 Chad/교체 아바타 전환, 하차/로비 대기 모션 등을 실험했다.
- 초기 G-force 반응은 차량/항공기의 물리적 g-load를 정확히 측정하는 것이 아니라 **속도·회전 변화에 기초한 시각적 반응 추정**으로 구현했다.

### 2.3 v74~v78 — VRMA 기반 대기 애니메이션

- 외부 VRMA 애니메이션을 프로젝트 자체의 `.spmotion` 트랙 포맷으로 변환하는 경로를 구성했다.
- 처음에는 월드 본 회전을 기준으로 적용해 캐릭터가 뒤틀리는 문제가 있었고, **초기 T-pose 기준 로컬 회전**을 저장해 상대 회전으로 적용하도록 수정했다.
- 루트·골반 위치 트랙 미지원으로 발생한 공중 부유 현상을 발 접촉/골반 높이 보정으로 완화했다.
- 정지 상태·지면 접촉·좌석 상태를 검사한 뒤 보행 중 대기 모션을 재생하는 구조로 발전시켰다.

### 2.4 v79~v82.4 — 머리카락 및 의상 물리 튜닝

- 시간 기반 보간, 깊은 헤어 체인 탐색, 머리카락 관성·반응성 등을 비교했다.
- 특정 관성 수식이 오히려 머리카락의 움직임을 둔화시키는 회귀가 있어 안정적인 이전 솔버로 되돌렸다.
- 긴 뒷머리용 보조 본 실험은 별도 모델 리그/스키닝 수정이 필요하다는 점을 확인했다.
- F6 표정 순환에서 입모양/수동 LookAt 항목을 제외하고 얼굴 표정 중심으로 정리했다.

### 2.5 v86.3~v86.17 — 씬·좌석·외형과 배포 안정화

| 버전/단계 | 해결하려던 문제 및 접근 |
|---|---|
| v86.3~v86.6 | MainMenu / Designer / Terrain의 실제 캐릭터 계층과 Seat Anchor가 다름을 확인하고 바인딩·골반 정렬 순서 수정 |
| v86.5.1 | Unity 6 참조 DLL과 .NET 참조 충돌(`CS1705`)을 `netstandard2.1` 타깃으로 수정 |
| v86.7~v86.9 | 얼굴 근접 클리핑, 홍채 표시, Fly/OnFoot 스키닝 갱신과 Renderer 진단 |
| v86.8~v86.10 | Fly→Build 착석 오프셋 및 기체 변경 시 틀어지는 좌석 위치를 **기체 인식 기반 좌석 전달**로 보정 |
| v86.11~v86.12 | 좌석 캐시 안전화, DLL 빌드·설치·복원 스크립트와 배포 패키징·로딩 진단 강화 |
| v86.17 | 범용 아바타 프로필 진단: Humanoid 필수 본/얼굴·눈 블렌드셰이프/SpringBone 일치 여부 검증 |

v86.17에서 사용자가 제공한 실제 로그는 `requiredBones=14/14`, `springHair=16/16`, `springBust=12/12`, `springDress=34/34` 등 프로필 매핑을 보여줬다. 이는 해당 모델과 설정에서의 매핑 결과이며 임의 모델 모두를 검증한 수치는 아니다.

---

## 3. v86.19 — Carlotta 전용에서 Universal VRM으로 전환

### 3.1 런타임 VRM 로더 및 기준 포즈 (v86.19-A~H)

1. `.vrm` 파일을 런타임에서 파싱/임포트하는 UniVRM 경로를 도입하고, 모델의 Humanoid/Animator/Renderer 유효성을 검사했다.
2. 실험용 미리보기와 포즈 복사를 분리했다. **원본 Chad/Carlotta 리그에는 쓰지 않고**, 새 임포트 VRM에만 포즈를 적용하도록 했다.
3. 메인 메뉴에서 사전 준비(prewarm)한 VRM을 Fly/Build 등에 연결하는 방식을 실험했다.
4. 실제 씬 전환 시 잘못된 Anchor를 잡지 않도록 MainMenu/Designer/Terrain 계층과 파일 import, Rig rebind를 진단했다.
5. 런타임 VRM 오버레이는 당시 20초 테스트로 시작해 이후 **120초 실험용 지속 시간**으로 확장됐다. **영구 상시 교체 구현은 아직 완료되지 않았다.**
6. 최초 런타임 임포트/포즈 전달 시험에서는 VRM의 `humanoid=True`, 본 매핑 `15/15`, 실제 장면 내 렌더링 및 원본 복원 로그를 확보했다.

### 3.2 좌석·손·SpringBone·유리 조사 (v86.19-I~T)

| 단계 | 변경 내용 | 중요한 제한 |
|---|---|---|
| I | 좌우 손목 오차 측정과 선택적 two-bone IK | 손가락 파지나 실제 접촉 메시 판정은 아님 |
| J | VRM 0.x/1.0의 SpringBone 메타데이터, Skinning 가중치, Collider 읽기 전용 분석 | metadata 존재가 물리 충돌 보장을 뜻하지 않음 |
| K | 공식 SpringBone 루트에 대한 기본 물리 실험, 착석시 고정 | 모델별 Joint 파라미터와 충돌은 후속 과제 |
| L | 조종석 손 주변의 Renderer/Collider 및 조종간/스로틀 후보 위치 진단 | 이름/거리 기반 후보일 뿐 실제 접촉은 미검증 |
| M~S | `Joystick_Top/GripTarget` 중심과 Carlotta 손바닥 상대 벡터를 이용한 동적 IK, 접촉 조건·프레임 업데이트·유리 깊이 A/B 반복 개선 | 특정 기체의 Collider 계층과 체형에 의존 |
| T Fix11 | 조종석 유리의 `sortingOrder`를 선택적으로 보정하고 장면 변경 시 복원 | 원본 유리 셰이더/색상 자체를 복원하는 기능은 아님 |

조종간·스로틀·차체 유리 등 **원본 게임 오브젝트를 직접 변경하는 실험은 가능한 한 제한**했다. 유리 정렬 기능은 사용자 관찰에서 개선 효과가 있었지만 모든 기체와 카메라의 시각적 일치를 보장하지 않는다.

### 3.3 v86.19-U~Y — 양손 IK 및 체형 차이 보정

| 버전 | 변경 사항 |
|---|---|
| **U** | 조종간/스로틀 로컬 좌표 기준으로 양손 목표점을 cm 단위 수동 보정하는 핫키 및 설정 추가 |
| **V** | 양손 도달 오차를 기준으로 중립 상태에서도 상체 숙임 후보 탐색, 한 손을 심하게 악화시키는 후보 거부 |
| **W** | 좌우 흔들림 대신 **정면 Forward Lean** 우선, Spine/Chest/UpperChest에 기울기 분배, 부드러운 복귀 |
| **X** | **Reach-first**: 먼저 팔 IK로 해결한 뒤 부족한 경우에만 상체 숙임 적용 |
| **Y** | 좌석 높이와 중립 조종간 도달 범위를 함께 보정, 상체 과숙임 억제 |

중요한 설계 판단: 단순히 상체를 크게 숙이는 것보다 **팔 IK → 최소 필요 상체 숙임 → 중립 복귀** 순서가 자연스럽고 안정적이었다. 도달 가능 여부는 실제 팔 길이와 좌석 위치를 기준으로 판단하며, 자동차 핸들/항공기 조종간 표면의 실제 접촉을 정확히 계산하는 시스템은 아니다.

### 3.4 v86.19-Z Fix2 — 범용 표정 및 기존 Idle 포즈 연결

- 특정 캐릭터의 BlendShape 고정 인덱스 대신 UniVRM 런타임의 **ExpressionKeys**를 이용한 범용 표정 매핑을 도입했다.
- F6 표정 순환, 자동 눈깜빡임과 표정 컨트롤러 연결을 복구했다.
- `RuntimeVrmPosePreviewV86_19E.Source=Carlotta`로 설정하여 기존 Carlotta/VRMA 대기 동작을 새 VRM에 전달했다.
- 사용자 로그: 18개 Expression clip 중 **8개 매핑**, `Blink/Happy/Angry/Sad/Relaxed/Surprised/BlinkLeft/BlinkRight`, `blinkAvailable=True`, `writesAllowed=True`.
- **비지원 10개**는 해당 모델의 표정 메타 구성과 매핑 범위에 관한 수치이지 파서 오류가 10개라는 의미는 아니다.
- 모델 자체의 표정 모양이 어색한 현상과 모드의 매핑 오류는 구분했다.

### 3.5 v86.19-AA — Chase 전용 범용 시선 추적

- UniVRM 런타임 `LookAt.SetLookAtYawPitch`와 `LookAtTargetType=SetYawPitch`를 탐지하여 새 VRM 눈동자를 제어한다.
- Native Chase 카메라 + 착석 상태에서만 활성화하고 Orbit/보행에서는 해제한다.
- `Source=Carlotta`일 때 이미 전달된 고개 회전을 **중복 적용하지 않는다**. 다른 포즈 소스는 필요한 경우에만 Head/Neck 본 회전.
- 실제 적용 로그: `setterFound=True`, `targetTypeWritable=True`, `eyeFailures=0`, Chase의 `eyeFrames=1726`, Orbit에서 `eyeOverrideActive=False`.
- **상태: 사용자 게임 로그 및 시각적 확인.**

### 3.6 v86.19-AB — 호흡·시각적 가속 반응의 범용화

- 현재 포즈 소스가 Carlotta이면 기존 호흡 및 G-force 반응이 포즈에 상속될 수 있어 **추가로 회전하지 않는 정책**을 구현했다(`policy=inherited_no_double_write`).
- `Source=Chad`인 경우를 위한 VRM 직접 보정 경로를 추가했다.
- 사용자 로그: `source=Carlotta`, `addedBodyWrites=False`, `exceptions=0`.
- **상태: 중복 회전 방지 동작 확인.** 실제 가속량 측정이나 모든 포즈 소스의 시각적 품질까지 검증한 것은 아니다.

### 3.7 v86.19-AB Vehicle Wheel Grip — 자동차 핸들 전용 양손 IK

- 기존 항공기 IK가 `Joystick/GripTarget` 및 `Throttle`를 탐색하므로 자동차 핸들에서 실패하는 문제를 확인했다.
- `Steering_Wheel/MoveRoot/Model1`을 이름 및 두 손 근접도로 탐색하고, 원본 Carlotta의 애니메이션된 손바닥을 목표로 하는 독립적인 양팔 IK를 추가했다.
- 처음 Inspect에서는 좌·우 손 목표 오차가 약 **14.2cm**, 팔의 도달 길이 부족이 약 **7cm**였다.
- Apply 후 약 **13.3cm → 6.9~7.3cm**로 오차를 줄였지만, 체형 자체의 팔 길이가 짧아 잔여 오차가 남았다.
- 차량용 보정은 원본 핸들 Transform 및 항공기 조종간을 수정하지 않는다. 접촉점은 **Carlotta 손바닥 프록시**이지 실제 메시 표면 판정이 아니다.

### 3.8 v86.19-AC — 0.1배 단위 실시간 모델 크기 조절

- 단축키로 VRM 모델의 배율을 실시간 변경: `Ctrl+Alt+PageUp/PageDown`(±0.1), `Ctrl+Alt+Home`(1.0 복원), `Ctrl+Alt+End`(설정 저장).
- 기존 `ScalePolicy=Locked`와 충돌하지 않도록 Live Scale 계수를 크기 계산에 포함하고, 재조정 시 좌석/발/양손 IK 도달 길이·상체 Neutral Reach를 재측정했다.
- Seat/Foot 보정은 관절 기반 프록시이며 실제 차체 쿠션/지면 메시 충돌 검사와 동일하지 않다.
- **실제 차량 테스트:**

| Live scale factor | 오른손 IK 후 오차 | 왼손 IK 후 오차 | 팔 도달 부족 |
|---|---:|---:|---|
| 1.1 | 3.64cm | 4.01cm | 양쪽 남음 |
| 1.2 | 0.37cm | 0.88cm | 양쪽 0cm |
| 1.4 | 약 0.46cm | 약 0.44cm | 양쪽 0cm |

이는 **손바닥 목표 프록시 기준 오차**다. 실제 핸들 표면에 손가락이 닿았다는 직접 증거는 아니다. 모델 배율이 증가할 때 머리·계기판·지붕 간섭 가능성도 남는다.

### 3.9 v86.19-AD — 등록된 전체 SpringBone 적용

- 기존 특정 헤어·의상 이름의 소규모 스프링 실험에서 벗어나, VRM 0.x의 `secondaryAnimation`과 VRM 1.0의 `VRMC_springBone` 메타데이터를 이용해 공식 SpringBone 루트/관절 체인을 탐색했다.
- 헤어·치마·소매·장식 등 공식 등록 체인을 포함하고, 조건이 맞는 일부 미등록 의상 후보를 실험적으로 추가했다.
- 모델이 제공하는 구형/캡슐형 Collider 메타데이터가 있으면 근사 충돌에 사용하지만 **실제 메시 자체 충돌이나 좌석 충돌을 구현한 것은 아니다.**
- 테스트 VRM의 로그: `officialLinks=103`, `extraWeightedLinks=4`, `totalLinks=107`, `metadataColliders=0`; 사용자는 서 있기·걷기 중 움직임이 정상이라고 보고했다.
- 기존 AD에서는 착석 중에도 소프트 스프링 계산이 계속되어 특정 속도에서 의상이 펄럭이거나 사라지는 현상이 나타났다.

### 3.10 v86.19-AE — 착석 중 기체 좌표계 기준 의상 고정

- 착석 상태에서 관성 기반 스프링 업데이트 대신, `Seat` Transform 기준 **기체의 바닥 방향**으로 정렬된 의상 자세를 계산한다.
- 비행기가 롤·피치·전복되어도 월드 중력이 아니라 좌석의 로컬 아래 방향을 유지한다.
- 보행/하차 시에는 AD의 일반 SpringBone 동작으로 복귀한다.
- 소매 바깥쪽 돌출 억제도 모델별로 제한적으로 적용하되, 소매 체인이 식별되지 않은 VRM에서 완전한 보정을 보장하지 않는다.
- 실제 로그: `seatPolicy=vehicle_local_drape_kinematic_no_inertia`, `seatFrame=Seat`, `gravityWorldAppliedWhileSeated=False`, `seatedKinematicFrames=3004`. 사용자도 시각적으로 정상에 가깝다고 보고했다.
- **한계:** 이 결과는 기체 기준 동작의 증거이며 실제 의상·좌석 메시 접촉을 검증한 것은 아니다.

### 3.11 v86.19-AE Material Compatibility / MToon10 Bundle Bridge

**문제:** 런타임 임포트는 VRM의 기본 텍스처를 불러오지만 원본 VRM의 MToon 명암과 윤곽선을 재현하지 못하고 `URP/Unlit`에 대체되는 현상.

**해결 단계:**

1. 재질별 Base Map/Normal/Emission/Alpha/Render Queue와 게임 셰이더 지원을 기록하는 진단을 추가했다.
2. URP/Unlit과 URP/Lit을 비교했고, `MtoonInspect`에서 컴파일된 MToon10 사용 가능 여부를 확인했다.
3. Unity Editor 프로젝트의 `VRM10/Universal Render Pipeline/MToon10` 셰이더를 Windows x64 AssetBundle `sp2_mtoon10_urp`로 빌드하여 게임에 설치했다.
4. 런타임에서 번들을 로드해 새 VRM 재질을 MToon10으로 생성·매핑했다. 번들 실패 시 `URP/Lit`으로 폴백하도록 했다.

**사용자 게임에서 확인한 결과:**

- `mtoon_bundle status=ready`, `supported=True`.
- `MaterialMode=MtoonBundle`, **10/10 재질에 MToon10 적용**, `texturedSlots=10`, `unsupportedSlots=0`, `transparentSlots=3`.
- 해당 테스트 번들 파일 크기는 **82,955 bytes**.
- 기존 Chad/Carlotta의 재질과 원본 VRM 에셋은 수정하지 않았다.

**제한:** 해당 로그는 런타임 셰이더 바인딩에 대한 증거다. VRM 0.x에서 MToon10으로 이전할 때 모든 Toon 속성이 완전히 일치하는 것은 아니며, URP 전용 MToon Outline Render Feature가 없다면 외곽선까지 원본과 같다고 보장할 수 없다. 다른 VRM의 투명도/노멀맵 표현도 별도 테스트가 필요하다.

### 3.12 여러 `.vrm` 파일로 호환성 시험

- `BepInEx\plugins\SP2Avatars`의 파일과 설정 `FileName`을 통해 대상 VRM을 변경하는 구조로 시험했다.
- 사용자 보고에 따르면 다른 VRM도 대체로 정상 작동했다.
- 성능 테스트에서는 `sop2.vrm`을 사용했고, 이 모델의 로그는 `humanoid=True`, `mappedBones=15/15`, `renderers=2`, `blendShapes=63`, `officialLinks=211`, `extraWeightedLinks=6`을 보여줬다.
- 따라서 범용 로더의 재사용성은 확인됐지만 **모든 VRM 규격·비표준 리그·누락된 표정/물리본까지 완벽하게 지원한다고 주장하지 않는다.**

---

## 4. v86.19-AF~AH — 성능 진단 및 스터터링 최적화

### 4.1 AF: 모듈별 프레임 프로파일러와 1차 할당 개선

- 프레임 평균/95백분위 근사/최장 프레임, 33·50·100ms 초과 프레임 수, 관리 힙과 GC 발생 횟수를 기록했다.
- 포즈 복사, 오버레이, SpringBone, 항공기·차량 IK, 시선, 표정 각각의 CPU 시간을 분리했다.
- 원본 Carlotta Renderer 숨김 목록 캐싱 및 일부 임시 배열 생성 축소를 시도했다.
- `sop2.vrm` 테스트에서는 SpringBone **217 links** 처리 평균이 주로 약 **0.32~0.37ms**였으며, 포즈 복사는 주로 **0.03~0.07ms** 수준이었다.
- 반면 항공기 IK 계측의 최대값이 **102.20ms**, 전체 프레임의 순간 최대 지연이 **597.36ms**까지 기록됐다. 일부 심한 지연 구간에 `gc0Delta=1`이 함께 관찰됐다.
- 이는 **CPU·GC 병목 후보**를 좁혀 준 로그이며 GPU 렌더링 시간이나 실제 원인 순서를 단독으로 확정하지는 못한다.

### 4.2 AG: 조종간 탐색 스터터링 최적화 — 사용자 로그 검증

**발견한 문제:** 항공기 조종간이 없는 차량에서 항공기용 `Joystick/GripTarget`을 찾기 위해 대규모 Collider 탐색이 반복될 수 있었다.

**적용 변경:**

- 필요 없는 `FindObjectsOfType`의 정렬 비용과 문자열 할당을 줄였다.
- 조종간 탐색 실패 시 재시도 간격을 **2초 → 4초 → 최대 8초**로 증가시켰다. Rig 변경 시 즉시 초기화한다.
- 차량 핸들이 이미 감지되면 항공기용 목표 탐색을 생략한다. 원래 팔 IK 수치와 손·상체 계산 자체는 유지한다.

**AG 실제 게임 로그:**

| 지표 | 관찰값 |
|---|---:|
| 항공기용 전역 조종간 검색 누적 | 4회 |
| 차량 핸들 감지로 생략한 검색 프레임 | 7,128 |
| 전체 조종간 검색 최대 시간 | 22.59ms |
| 5초 측정 창의 안정 구간 평균 프레임 시간 | 약 13.6~15.7ms |
| 초기 창의 최대 프레임 지연 | 166.67ms |
| 초기 창 이후 100ms 초과 프레임 | 첨부된 후속 측정 창에서 0회 |

AF 시절 90~102ms의 항공기 IK 순간값이 AG에서는 제시된 윈도우의 대부분에서 매우 작거나 최대 약 24ms 수준으로 나타났다. **서로 다른 게임 실행 결과이므로 절대적인 성능 향상률을 계산하지 않는다.** 또한 AG에서도 초기 대규모 탐색 및 드문 GC 프레임 지연은 남는다.

### 4.3 AH: 착석 판정 GC 할당 최적화 — 소스 패치, 실게임 미검증

- `CarlottaSeatDetector`의 Animator 클립 검사에서 매 호출 `GetCurrentAnimatorClipInfo()`가 반환하는 배열을 새로 생성하는 문제를 대상으로 했다.
- 재사용 가능한 클립 목록과 동일 프레임의 판정 캐시를 적용하도록 수정했다.
- 프로파일러에 `seatCacheHits`, `seatCacheMisses`, `seatClipListQueries` 로그를 추가했다.
- AG 대비 **변경 C# 파일은 `CarlottaSeatDetector.cs`, `RuntimeVrmPerfAf.cs` 두 개**이고 나머지 통합 기능 C# 소스는 보존했다.
- 패키지 정적/ZIP 무결성 검사만 완료되어 있고, **2026-11-13 기준 사용자 Windows 빌드·게임 프로파일링 결과는 아직 전달되지 않았다.** 따라서 AH로 GC 및 스터터링이 줄었다고 기록하지 않는다.

### 4.4 성능 해석에서 주의할 점

- `avgFrameMs`, `maxFrameMs`, `gc0Delta`, `managedHeapMb`는 게임 프로세스 전체의 값이다.
- Spring/IK/Pose 등의 `*AvgMs`, `*MaxMs`는 선택한 플러그인 구간의 **CPU 계측**이며 GPU 비용과 동기화 대기를 완전히 포함하지 않는다.
- `managedHeapMb`가 증가한 뒤 GC로 줄어드는 현상은 **할당/회수 주기**를 보여주지만, 그 자체로 메모리 누수를 입증하지 않는다.
- 장면·차량·VRM 및 측정 조건이 같지 않은 로그 사이에서 FPS 향상률을 확정하지 않는다.

---

## 5. 실제로 확인한 주요 결과 요약

| 범주 | 검증 관찰 | 상태 |
|---|---|---|
| 런타임 VRM 로딩 | 서로 다른 VRM 모델을 임포트하고 Humanoid 본 매핑 로그 확보 | 사용자 게임 로그/보고 |
| 일반 동작 | 보행·대기·착석 및 VRMA/기본 애니메이션 전달 | 사용자 게임 보고, 모델별 차이 가능 |
| 표정 | 런타임 ExpressionKeys 8개 매핑, F6·Blink 동작 | 게임 로그 및 사용자 확인 |
| 시선 | Chase 눈동자 추적, Orbit 해제, eyeFailures=0 | 게임 로그/사용자 확인 |
| 자동차 양손 IK | 핸들 탐색 성공 및 두 손 목표 오차 감소 | 게임 로그, 실제 메시 접촉은 미검증 |
| 실시간 모델 크기 | 배율 1.2 이상에서 차량 팔 도달 부족 0cm | 게임 로그, 차량/모델별 차이 있음 |
| 보행 SpringBone | 공식 체인과 일부 보조 체인 활성, 보행 중 정상 보고 | 게임 로그/사용자 확인 |
| 착석 의상 | Seat 좌표계 기준 비관성 자세, 보행 복귀 | 게임 로그/사용자 확인 |
| MToon10 | 번들 준비 및 10/10 재질 적용, 텍스처 10/10 | 게임 런타임 바인딩 확인 |
| AF 프로파일러 | 평균 프레임과 장시간 순간 지연/GC 후보 계측 | 사용자 게임 로그 |
| AG 스터터링 수정 | 검색 4회, 7,128프레임 검색 생략 | 사용자 게임 로그 |
| AH GC 수정 | 파일 제작/정적 점검 완료 | **사용자 실행 결과 대기** |

---

## 6. 알려진 한계 / 남은 과제

1. **실험용 120초 오버레이:** 기본 시스템의 정식 상시 로더 및 씬 전환 간 라이프사이클은 아직 추가 정비가 필요하다.
2. **임의 VRM에 대한 범용성:** 올바른 Humanoid 본 및 VRM 메타데이터가 있어야 하며, 표정·LookAt·SpringBone이 없는 모델에서는 관련 기능을 건너뛸 수 있다.
3. **접촉 IK 정확도:** 손 목표는 Collider 중심/원본 Carlotta 손바닥을 이용한 프록시다. 실제 손가락 파지, 좌석 쿠션, 지면 메시 충돌까지 해결한 것은 아니다.
4. **MToon 완전 재현:** VRM 0.x→MToon10 재질 이전, 아웃라인, 복잡한 투명 재질은 계속 확인이 필요하다.
5. **의상 충돌:** 실제 모델 메시·좌석·기체와의 충돌 및 자체 충돌은 미지원이다. SpringBone 메타 Collider가 없는 VRM에서는 접촉 보정을 과장하지 않는다.
6. **성능:** 전체 씬 검색/로딩, 캐릭터 교체, 셰이더 변형, GC·렌더러·GPU 병목 등은 별도 계측과 튜닝이 필요하다.
7. **설정 저장:** VRM 파일은 교체할 수 있지만 캐릭터별 크기·IK·표정·물리 프로필의 자동 분리는 아직 구현되지 않았다.
8. **회귀 테스트:** 다양한 차량/비행기/VRM, MainMenu/Build/Fly/보행, 장시간 반복 전환을 조합한 자동화 검증은 미완료다.

## 7. 후속 개발 로드맵

- [ ] **AH 검증:** AG와 동일한 기체/VRM/배율에서 GC와 최대 프레임 지연 비교
- [ ] **정식 로더:** 120초 프리뷰 제한 제거, 씬 변경·기체 변경·재탑승에 안전한 상시 유지
- [ ] **다중 VRM 선택:** 게임 내 목록/교체, 런타임 자산 캐시 및 명시적 Dispose
- [ ] **캐릭터별 프로필:** 모델별 배율, Seat offset, 손 IK, 표정·물리 강도 저장
- [ ] **손가락 IK:** 핸들·조종간·스로틀의 접촉면 기반 파지
- [ ] **재질 확장:** MToon Outline, VRM 0.x 재질 속성 이전 정확도, 유리/머리카락 알파 정렬
- [ ] **고급 물리:** 모델별 Collider·좌석·소매 관통 완화 및 장시간 안정성
- [ ] **프로파일러 확장:** GPU/GC Alloc/로드 단계/VRAM/프레임타임 P99 및 다양한 모델 비교

---

## 8. 개발·테스트·롤백 규칙

### 8.1 원칙

- 기능 추가는 **Inspect(읽기 전용) → Apply → 로그/화면 비교 → 회귀 검증** 순서로 수행한다.
- 정상 동작하는 C# 파일은 불필요하게 수정하지 않는다. 버전별 소스와 변경 차이 및 해시를 남긴다.
- 빌드가 성공한 경우에만 DLL 설치를 진행하고 이전 DLL 및 설정을 백업한다.
- 게임 실행 중 DLL을 덮어쓰지 않는다.
- 임포트 VRM·메시·표정에만 변경을 가하고 원본 Chad/Carlotta Animator·기체 물리·조종 장치를 보존한다.
- **소스 구현**, **ZIP/정적 검사**, **Windows 빌드**, **런타임 로그**, **시각적 성공**, **다양한 모델 회귀 테스트**를 서로 다른 증거 단계로 기록한다.

### 8.2 로그 및 설정 예시

```powershell
$game = 'D:\Steam\steamapps\common\SimplePlanes 2'
$log = "$game\BepInEx\LogOutput.log"
Get-Content -LiteralPath $log -Tail 120
```

```ini
[RuntimeVrmPreviewV86_19D]
FileName = sop2.vrm
MaterialMode = MtoonBundle
```

버전별 `Check-v86_19*.ps1`은 해당 ZIP의 루트 폴더에 있다. 실제 설정 섹션 이름과 실행 스크립트는 **현재 설치된 패키지와 맞춰서** 사용해야 한다.

### 8.3 GitHub 게시·라이선스 유의

이 프로젝트는 비공식/비상업 팬 프로젝트이며 Jundroo 및 해당 외부 캐릭터의 권리자와 제휴하지 않는다. **게임 원본 DLL, 제3자 VRM/텍스처/모션/AssetBundle을 허가 없이 Git 저장소나 Release에 배포하지 않는다.** 가능하면 자체 C# 소스, 로컬에서 빌드하는 도구, 독립적으로 작성한 스크립트, 허가받은 샘플만 게시한다. 타사 코드를 함께 배포할 때는 그 라이선스·저작권 표시·소스/재배포 조건을 별도로 확인한다.

---

## 9. 현재 결론

**Carlotta 전용 교체 실험에서 출발한 모드는 현재 호환 VRM 파일을 런타임 임포트하고, 게임 애니메이션·좌석·양손 IK·표정·시선·SpringBone·MToon10을 연결하는 범용 아바타 실험 시스템으로 발전했다.** 실제 사용자 게임 로그로 주요 기능과 AG 최적화의 동작이 확인됐지만, 완전한 범용 호환성·원본 재질의 100% 일치·120초 제한 없는 상시 교체·GPU 병목 제거는 아직 완료되지 않았다. **현재 최신 AH는 성능 최적화 소스 패치이며 Windows 게임 실측 결과를 기다리는 단계다.**
