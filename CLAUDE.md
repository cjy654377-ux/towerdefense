# 타워디펜스 프로젝트 개발 규칙

## 프로젝트 개요

- **장르**: 2D 타워디펜스 (유닛 배치형, 아크나이츠 스타일)
- **엔진**: Unity 2022 LTS / C#
- **타겟 플랫폼**: iOS / Android
- **에셋**: SPUM (2D Pixel Unit Maker) 메가 에셋
- **수익 모델**: 가챠 시스템 + 배틀 패스

---

## 1. 프로젝트 폴더 구조

```
Assets/
├── _Project/                      # 프로젝트 전용 에셋 (밑줄로 최상단 정렬)
│   ├── Animations/
│   │   ├── Units/
│   │   │   ├── Controllers/       # Animator Controller 파일
│   │   │   └── Clips/
│   │   └── UI/
│   ├── Audio/
│   │   ├── BGM/
│   │   ├── SFX/
│   │   └── Mixers/
│   ├── Data/                      # ScriptableObject 인스턴스
│   │   ├── Units/                 # UnitData SO
│   │   ├── Stages/                # StageData SO
│   │   ├── Skills/                # SkillData SO
│   │   ├── Gacha/                 # GachaPoolData SO
│   │   └── BattlePass/            # BattlePassData SO
│   ├── Materials/
│   ├── Prefabs/
│   │   ├── Units/
│   │   │   ├── Operators/         # 배치 가능 유닛
│   │   │   └── Enemies/
│   │   ├── UI/
│   │   │   ├── Panels/
│   │   │   ├── Popups/
│   │   │   └── Elements/
│   │   ├── Effects/
│   │   ├── Projectiles/
│   │   └── Stage/
│   ├── Scenes/
│   │   ├── Boot.unity             # 초기화 전용 씬
│   │   ├── Lobby.unity
│   │   ├── Stage_01.unity
│   │   └── ...
│   ├── Scripts/
│   │   ├── Battle/
│   │   │   ├── Units/
│   │   │   ├── Enemies/
│   │   │   ├── Skills/
│   │   │   ├── Projectiles/
│   │   │   └── Map/
│   │   ├── Core/
│   │   │   ├── GameManager.cs
│   │   │   ├── SceneLoader.cs
│   │   │   └── ServiceLocator.cs
│   │   ├── Data/
│   │   │   ├── ScriptableObjects/  # SO 클래스 정의
│   │   │   └── SaveLoad/
│   │   ├── Gacha/
│   │   ├── BattlePass/
│   │   ├── UI/
│   │   │   ├── Lobby/
│   │   │   ├── Battle/
│   │   │   └── Common/
│   │   ├── Network/
│   │   └── Utils/
│   ├── Shaders/
│   ├── Sprites/
│   │   ├── UI/
│   │   ├── Map/
│   │   └── Icons/
│   └── TextMeshPro/               # TMP 에셋 (자동 생성 폴더 아님)
├── SPUM/                          # SPUM 에셋 패키지 (수정 금지)
├── Plugins/                       # 서드파티 플러그인
│   ├── iOS/
│   └── Android/
└── StreamingAssets/               # 런타임 로드 에셋
    └── StageData/
```

### 폴더 규칙

- `_Project/` 밑줄 접두사로 항상 최상단 정렬 유지
- `SPUM/` 폴더는 **절대 수정하지 않는다** — 업그레이드 대비
- 에셋 종류별 폴더 분리 (타입별 > 기능별 순)
- 씬 파일은 반드시 `Scenes/` 폴더에만 위치

---

## 2. C# 코딩 컨벤션

### 네이밍

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 / 구조체 | PascalCase | `OperatorUnit`, `StageData` |
| 인터페이스 | I + PascalCase | `IDamageable`, `IDeployable` |
| public 필드/프로퍼티 | PascalCase | `MaxHp`, `AttackRange` |
| private 필드 | _camelCase (밑줄 접두사) | `_currentHp`, `_animator` |
| 메서드 | PascalCase | `TakeDamage()`, `DeployUnit()` |
| 매개변수 / 로컬 변수 | camelCase | `damageAmount`, `targetEnemy` |
| 상수 | UPPER_SNAKE_CASE | `MAX_DEPLOY_COUNT`, `BASE_ATK` |
| enum 타입 | PascalCase | `UnitClass`, `SkillType` |
| enum 값 | PascalCase | `UnitClass.Vanguard`, `SkillType.Passive` |
| ScriptableObject 클래스 | ~Data / ~Config | `UnitData`, `StageConfig` |
| MonoBehaviour 컴포넌트 | ~Controller / ~Handler / ~View | `BattleController`, `InputHandler` |
| 이벤트 | On~ | `OnUnitDeployed`, `OnStageCleared` |

### 포맷팅

```csharp
// 올바른 예
public class OperatorUnit : MonoBehaviour, IDamageable
{
    // 직렬화 필드는 SerializeField + private
    [SerializeField] private UnitData _data;
    [SerializeField] private Animator _animator;

    // 상수
    private const float DEPLOY_COOLDOWN = 1.5f;

    // 프로퍼티 (getter-only는 한 줄)
    public int CurrentHp { get; private set; }
    public bool IsDead => CurrentHp <= 0;

    // 이벤트
    public event Action<OperatorUnit> OnDeath;

    // Unity 콜백 순서: Awake > OnEnable > Start > Update > OnDisable > OnDestroy
    private void Awake() { }
    private void Start() { }
    private void Update() { }

    public void TakeDamage(int damage)
    {
        CurrentHp = Mathf.Max(0, CurrentHp - damage);
        if (IsDead)
        {
            OnDeath?.Invoke(this);
        }
    }
}
```

### 코드 규칙

- `[SerializeField] private` 사용 — `public` 필드 직접 노출 금지
- `using` 네임스페이스 정렬: System → Unity → 프로젝트 순
- 메서드 길이 50줄 초과 시 분리 검토
- 매직 넘버 금지 — 상수 또는 ScriptableObject로 추출
- LINQ는 Update() 등 매 프레임 호출 경로에서 금지 (GC 발생)
- `string.Format` 대신 `$""` 보간 문자열 사용
- null 체크: `obj != null` 대신 `obj is not null` 또는 null 조건 연산자 `?.` 활용
- 코루틴 캐싱: `WaitForSeconds` 등 반복 사용 객체는 필드에 캐싱

```csharp
// 금지
private IEnumerator BadCoroutine()
{
    yield return new WaitForSeconds(1f); // 매 호출 할당 발생
}

// 권장
private readonly WaitForSeconds _waitOneSec = new WaitForSeconds(1f);

private IEnumerator GoodCoroutine()
{
    yield return _waitOneSec;
}
```

---

## 3. Git 브랜치 전략

### 브랜치 구조 (GitHub Flow 기반)

```
main          ← 항상 빌드 가능한 안정 브랜치 (직접 push 금지)
develop       ← 통합 개발 브랜치
├── feature/  ← 기능 개발
├── fix/      ← 버그 수정
├── hotfix/   ← main 긴급 수정
├── release/  ← 릴리즈 준비
└── art/      ← 에셋/아트 작업 전용
```

### 브랜치 네이밍

```
feature/gacha-system
feature/battle-ui-overhaul
fix/enemy-pathfinding-stuck
hotfix/crash-on-ios-16
release/v1.2.0
art/operator-idle-animation
```

### 커밋 메시지 규칙 (Conventional Commits)

```
<type>(<scope>): <subject>

[본문 - 선택]

[footer - 선택]
```

**타입**:

| 타입 | 사용 상황 |
|------|-----------|
| `feat` | 새 기능 추가 |
| `fix` | 버그 수정 |
| `refactor` | 동작 변경 없는 코드 개선 |
| `perf` | 성능 개선 |
| `art` | 에셋 추가/수정 (스프라이트, 애니메이션 등) |
| `data` | ScriptableObject 데이터 추가/수정 |
| `ui` | UI 레이아웃/스타일 변경 |
| `docs` | 문서 수정 |
| `chore` | 빌드 설정, 패키지 업데이트 등 |
| `test` | 테스트 추가/수정 |

**예시**:

```
feat(gacha): 10연 가챠 시스템 구현
fix(enemy): 코너에서 경로 탐색 멈추는 현상 수정
perf(battle): 오브젝트 풀 적용으로 적 스폰 GC 제거
art(operator): 뱅가드 공격 애니메이션 추가
data(stage): 1-3 스테이지 배치 데이터 작성
```

### PR 규칙

- PR 제목은 커밋 메시지 형식 준수
- 셀프 머지 금지 (팀 프로젝트 시)
- PR 크기: 변경 파일 10개 이하 권장 — 초과 시 분할 검토
- Unity 씬/프리팹 변경 포함 PR은 반드시 스크린샷 첨부

---

## 4. Unity 씬 / 프리팹 관리 규칙

### 씬 구조

```
Boot.unity       - 앱 초기화만 담당 (GameManager, ServiceLocator 초기화)
Lobby.unity      - 로비, 가챠, 배틀패스 UI
Stage_XX.unity   - 스테이지 (번호 두 자리 패딩: Stage_01)
```

- **Boot 씬**은 항상 Build Settings의 첫 번째 씬
- 씬 간 데이터 전달은 `ScriptableObject` 또는 `DontDestroyOnLoad` 매니저를 통해서만
- 씬에 직접 오브젝트 배치 최소화 — 런타임 스폰 또는 프리팹 사용 권장

### 프리팹 규칙

- 씬에 배치한 오브젝트를 수정했다면 반드시 **Prefab Apply** 또는 **Prefab Override** 처리
- Nested Prefab 3단계 초과 금지 (퍼포먼스 및 관리 복잡도)
- 프리팹 네이밍: `[카테고리]_[이름]`
  ```
  Unit_Operator_Vanguard_01
  Unit_Enemy_Soldier_01
  UI_Panel_Lobby
  UI_Popup_Gacha
  Effect_Hit_Slash
  ```
- 프리팹 수정 시 Variant 활용 — 원본 프리팹은 베이스 유지

### 컴포넌트 배치 순서 (Inspector)

1. Transform
2. Renderer (SpriteRenderer, Canvas 등)
3. Physics (Rigidbody2D, Collider2D)
4. 핵심 로직 컴포넌트 (OperatorUnit 등)
5. 보조 컴포넌트 (AudioSource 등)

---

## 5. SPUM 에셋 관련 규칙

### 원칙

- `SPUM/` 폴더 내 파일 **직접 수정 절대 금지** — 업데이트 시 덮어씌워짐
- SPUM 커스터마이징은 반드시 `_Project/` 폴더 내 별도 파일로 오버라이드
- SPUM 컴포넌트(`SPUM_Prefabs`, `SPUMManager` 등)는 래핑 클래스를 통해 사용

### SPUM 래퍼 패턴

```csharp
// 직접 사용 금지
// GetComponent<SPUM_Prefabs>().PlayAnimation(...)

// 래퍼를 통해 사용
public class OperatorAnimationController : MonoBehaviour
{
    [SerializeField] private SPUM_Prefabs _spumPrefabs;

    public void PlayAttack() => _spumPrefabs.PlayAnimation(UnitState.ATTACK);
    public void PlayMove()   => _spumPrefabs.PlayAnimation(UnitState.MOVE);
    public void PlayIdle()   => _spumPrefabs.PlayAnimation(UnitState.IDLE);
}
```

### 커스텀 SPUM 유닛 제작 시

1. SPUM 에디터에서 유닛 생성 → `_Project/Prefabs/Units/` 에 저장
2. 스프라이트 교체 시 `_Project/Sprites/Units/` 에 커스텀 스프라이트 위치
3. 유닛당 고유 ID를 `UnitData` ScriptableObject에 기록
4. SPUM 색상 커스터마이징 데이터는 `_Project/Data/Units/` 에 저장

---

## 6. 아키텍처 패턴

### 전체 구조: MVC + Service Locator + ScriptableObject 데이터 레이어

```
┌─────────────────────────────────────────┐
│              ScriptableObject           │
│         (불변 데이터 / 이벤트 채널)         │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────┐
│    Model (순수 데이터/로직)    │  C# 클래스, SO 인스턴스
├──────────────────────────────┤
│    Controller (게임 로직)     │  MonoBehaviour
├──────────────────────────────┤
│    View (화면 표시)           │  MonoBehaviour + UI
└──────────────────────────────┘
```

### ScriptableObject 활용

**데이터 SO** — 수치 데이터 저장:
```csharp
[CreateAssetMenu(menuName = "TowerDefense/Unit/UnitData")]
public class UnitData : ScriptableObject
{
    public string UnitId;
    public string DisplayName;
    public UnitClass Class;
    public int MaxHp;
    public int Attack;
    public float AttackInterval;
    public float BlockCount;
    public SkillData[] Skills;
    public Sprite Portrait;
    public GameObject Prefab;
}
```

**이벤트 채널 SO** — 씬 간 결합 제거:
```csharp
[CreateAssetMenu(menuName = "TowerDefense/Events/UnitDeployedEvent")]
public class UnitDeployedEventSO : ScriptableObject
{
    public event Action<OperatorUnit, Vector2Int> OnRaised;

    public void Raise(OperatorUnit unit, Vector2Int tile)
        => OnRaised?.Invoke(unit, tile);
}
```

### 서비스 로케이터

```csharp
// 전역 서비스 등록/조회
ServiceLocator.Register<IAudioService>(new AudioService());
var audio = ServiceLocator.Get<IAudioService>();
```

- `GameManager`, `StageManager`, `UIManager` 등 싱글톤 대신 ServiceLocator 사용
- 싱글톤 패턴은 `GameManager` 단 하나로 제한

### 오브젝트 풀

적, 투사체, 이펙트는 반드시 오브젝트 풀 사용:

```csharp
// PoolManager를 통해서만 스폰/반환
var enemy = PoolManager.Instance.Get<EnemyUnit>("Enemy_Soldier_01");
PoolManager.Instance.Return(enemy);
```

---

## 7. 성능 최적화 가이드라인 (모바일 타겟)

### 드로우콜 최적화

- 스프라이트 아틀라스 필수 사용 (`_Project/Sprites/` 하위 폴더별 Atlas 1개)
- UI Canvas 분리: 정적 UI (HUD 배경) / 동적 UI (HP바, 수치) Canvas 분리
- 월드 스페이스 Canvas 사용 금지 — Screen Space Overlay 또는 Camera 사용

### 메모리 / GC

```csharp
// 금지: Update()에서 반복 할당
void Update()
{
    var enemies = FindObjectsOfType<EnemyUnit>(); // 매 프레임 할당!
    var list = new List<EnemyUnit>();             // 매 프레임 할당!
}

// 권장: 캐싱 및 재사용
private readonly List<EnemyUnit> _enemyBuffer = new List<EnemyUnit>();
private EnemyUnit[] _cachedEnemies;

void Update()
{
    _enemyBuffer.Clear();
    // _enemyBuffer 재사용
}
```

- `string` 연산은 `StringBuilder` 또는 `ZString` 사용 (UI 업데이트)
- 이벤트 구독 해제 철저히: `OnEnable`에서 구독 → `OnDisable`에서 해제
- `Camera.main` 반복 호출 금지 → `Awake()`에서 캐싱

### 물리 / 충돌

- `FixedUpdate` 내 물리 연산 집중
- `Physics2D.OverlapCircleNonAlloc()` 등 NonAlloc 변형 사용
- 콜라이더 Shape: Circle > Polygon (연산 비용 순)
- Layer Collision Matrix에서 불필요한 레이어 간 충돌 비활성화

### 렌더링 예산 (타겟 기준)

| 항목 | iOS 타겟 | Android 타겟 |
|------|---------|-------------|
| 드로우콜 | ≤ 100 | ≤ 80 |
| 동시 활성 적 | ≤ 30 | ≤ 20 |
| 파티클 최대 | ≤ 200 | ≤ 150 |
| 텍스처 메모리 | ≤ 150MB | ≤ 100MB |

### 프로파일링 주기

- 신규 기능 완성 후 반드시 Unity Profiler로 CPU/메모리 측정
- Frame Time: 33ms 이하 유지 (30fps 기준)
- 스테이지 전투 중 GC.Alloc 0 목표

---

## 8. 테스트 규칙

### 테스트 구조

```
Assets/
└── _Project/
    └── Tests/
        ├── EditMode/          # Edit Mode 테스트 (순수 로직)
        │   ├── GachaSystemTests.cs
        │   ├── DamageCalculationTests.cs
        │   └── PathfindingTests.cs
        └── PlayMode/          # Play Mode 테스트 (Unity 런타임 필요)
            ├── BattleFlowTests.cs
            └── UnitDeployTests.cs
```

### 테스트 대상 (필수)

- 가챠 확률 계산 로직
- 데미지 / 방어 계산 수식
- 경로 탐색 알고리즘
- 배틀 패스 경험치 / 보상 로직
- 저장/불러오기 직렬화

### 테스트 제외 (선택)

- MonoBehaviour 라이프사이클 자체
- Unity Physics, Animation 동작
- UI 레이아웃

### 테스트 작성 규칙

```csharp
// 네이밍: [테스트대상]_[시나리오]_[기대결과]
[Test]
public void GachaSystem_Pull10Times_GuaranteesRare()
{
    // Arrange
    var gachaData = ScriptableObject.CreateInstance<GachaPoolData>();
    gachaData.PityCount = 10;
    var system = new GachaSystem(gachaData);

    // Act
    var results = system.Pull(10);

    // Assert
    Assert.IsTrue(results.Any(r => r.Rarity >= Rarity.Rare));
}
```

- Arrange / Act / Assert 3단 구조 준수
- 순수 C# 로직은 Edit Mode 테스트 우선
- ScriptableObject 의존 테스트는 `ScriptableObject.CreateInstance<T>()` 사용
- 난수 의존 로직은 시드 고정 또는 인터페이스 주입으로 테스트 가능하게 설계

---

## 9. Rust Token Killer (RTK) 사용 규칙

Claude Code 세션에서 **RTK(Rust Token Killer)** 사용을 필수로 한다.

### 목적

- 장시간 코딩 세션에서 컨텍스트 윈도우가 커질수록 토큰 소모가 급증한다
- RTK는 `git status`, `grep`, `find` 등 빈번한 CLI 명령어 출력을 자동으로 필터링하여 불필요한 토큰을 제거한다
- 60~90% 토큰 절감 효과로 컨텍스트 한계 도달 전까지 더 긴 세션 유지 가능

### 설치 확인

```bash
rtk --version        # RTK 설치 확인
rtk gain             # 누적 토큰 절감량 확인
which rtk            # 바이너리 경로 확인
```

### 사용 방법

RTK는 Claude Code Hook을 통해 **자동으로 동작**한다. 별도 명령 입력 없이 기존 CLI 명령이 RTK를 통해 실행된다:

```bash
# 내부적으로 자동 변환됨
git status     →  rtk git status
git diff       →  rtk git diff
grep ...       →  rtk grep ...
```

### 직접 사용 (디버깅/분석 시)

```bash
rtk gain              # 현재 세션 토큰 절감 통계
rtk gain --history    # 명령어별 절감 이력
rtk discover          # Claude Code 히스토리에서 미사용 최적화 기회 탐색
rtk proxy <cmd>       # 필터링 없이 원본 출력 확인 (디버깅용)
```

### 규칙

- RTK가 설치되지 않은 환경에서 작업 시작 전 반드시 설치
- `rtk gain`으로 주기적으로 절감량 확인 — 컨텍스트 관리 현황 파악
- RTK 관련 이슈 발생 시 `rtk proxy <cmd>`로 원본 출력과 비교하여 디버깅

---

## 추가 규칙

### Inspector 주석

```csharp
[Header("--- 전투 스탯 ---")]
[SerializeField, Range(1, 100)] private int _level = 1;

[Space(10)]
[Header("--- 참조 ---")]
[SerializeField] private Transform _attackOrigin;

[Tooltip("초당 공격 횟수 (1 / AttackInterval)")]
[SerializeField] private float _attackInterval = 1.5f;
```

### 에러 처리

```csharp
// Debug.Log는 빌드에서 제거 — 조건부 컴파일 사용
[System.Diagnostics.Conditional("UNITY_EDITOR")]
private static void Log(string message) => Debug.Log(message);

// 크리티컬 에러만 Debug.LogError 허용
```

### 로컬라이제이션

- 하드코딩 한국어 문자열 금지
- 모든 UI 텍스트는 `LocalizationManager`를 통해 키로 참조
- 텍스트 키 형식: `[화면].[요소].[상태]` → `lobby.gacha.button_pull_10`
