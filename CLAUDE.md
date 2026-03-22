# Tower Defense - 개발 규칙 (CLAUDE.md)

## 프로젝트 개요

- **프로젝트명**: Tower Defense (유닛 배치형 전략 디펜스)
- **엔진**: Unity (C#)
- **플랫폼**: iOS / Android
- **아트 에셋**: SPUM (메가 에셋)
- **프로젝트 경로**: `~/towerdefense`

---

## 코드 컨벤션

### C# 네이밍

- 클래스, 구조체, 열거형: `PascalCase` (예: `UnitController`, `WaveManager`)
- 퍼블릭 메서드, 프로퍼티: `PascalCase` (예: `GetDamage()`, `MaxHealth`)
- 프라이빗 필드: `_camelCase` (예: `_currentHp`, `_attackSpeed`)
- 로컬 변수, 파라미터: `camelCase` (예: `targetEnemy`, `damageAmount`)
- 상수: `UPPER_SNAKE_CASE` (예: `MAX_UNIT_COUNT`, `BASE_ATTACK_SPEED`)
- 인터페이스: `I` 접두사 (예: `IDamageable`, `IPoolable`)
- ScriptableObject: `SO_` 접두사 (예: `SO_UnitData`, `SO_StageConfig`)

### 파일 구조

```
Assets/
├── _Project/
│   ├── Scripts/
│   │   ├── Core/           # 게임 매니저, 이벤트 시스템, 오브젝트 풀
│   │   ├── Unit/           # 유닛 관련 (배치, AI, 스탯, 스킬)
│   │   ├── Enemy/          # 적 관련 (이동, 스폰, AI)
│   │   ├── Stage/          # 스테이지, 웨이브, 맵 데이터
│   │   ├── UI/             # UI 매니저, 팝업, HUD
│   │   ├── Gacha/          # 가챠, 소환 시스템
│   │   ├── Data/           # ScriptableObject 정의, 데이터 모델
│   │   └── Util/           # 헬퍼, 확장 메서드, 상수
│   ├── Prefabs/
│   ├── Scenes/
│   ├── ScriptableObjects/
│   ├── UI/
│   └── Art/
├── SPUM/                   # SPUM 에셋 (수정 금지)
├── Plugins/
└── Resources/
```

### 코드 규칙

- 한 스크립트 200줄 이하 유지. 초과 시 분리
- MonoBehaviour는 최소한으로. 로직은 일반 C# 클래스에 분리
- `FindObjectOfType`, `GameObject.Find` 사용 금지 → DI 또는 ServiceLocator 패턴
- `Update()`에 직접 로직 넣지 않기 → 상태 머신 또는 이벤트 기반으로
- 매직 넘버 금지 → 상수 또는 ScriptableObject로 관리
- 주석은 "왜(why)"를 설명. "무엇(what)"은 코드가 설명하도록
- 한글 주석 허용, 단 public API에는 영문 XML 주석 필수

### 아키텍처 패턴

- **이벤트 시스템**: ScriptableObject 기반 이벤트 채널
- **오브젝트 풀링**: 유닛, 적, 투사체 등 빈번한 생성/파괴 대상에 필수
- **상태 머신**: 유닛 AI, 적 AI에 FSM 패턴 적용
- **데이터 주도**: 밸런스 수치는 ScriptableObject로 관리, 코드에 하드코딩 금지

---

## Git 규칙

### 브랜치 전략

- `main`: 릴리스 가능 상태
- `develop`: 개발 통합 브랜치
- `feature/기능명`: 기능 개발 (예: `feature/unit-placement`)
- `fix/버그명`: 버그 수정 (예: `fix/wave-spawn-timing`)
- `refactor/대상`: 리팩토링 (예: `refactor/event-system`)

### 커밋 메시지

```
[태그] 한줄 요약

- 상세 내용 (선택)

Co-Authored-By: Claude <noreply@anthropic.com>
```

태그 종류: `feat`, `fix`, `refactor`, `ui`, `data`, `docs`, `test`, `chore`

### .gitignore 필수 항목

```
Library/
Temp/
Obj/
Build/
Builds/
Logs/
UserSettings/
*.csproj
*.sln
*.suo
*.user
*.pidb
*.booproj
.vs/
.idea/
```

---

## SPUM 에셋 사용 규칙

- `SPUM/` 폴더 내 원본 파일 직접 수정 금지
- SPUM 프리팹은 래핑하여 사용: `SPUMCharacterWrapper.cs`로 제어
- 캐릭터 커스터마이징 데이터는 ScriptableObject로 관리
- 애니메이션 상태 전환은 SPUM의 기본 시스템 활용, 커스텀 레이어 추가 시 별도 컨트롤러

---

## 테스트 & 빌드

- Play Mode 테스트 작성 권장 (핵심 시스템: 데미지 계산, 웨이브 스폰, 가챠 확률)
- 빌드 전 `_Project/Scenes/` 내 씬만 Build Settings에 포함
- Android: minSDK 24, targetSDK 최신
- iOS: minimum 15.0

---

## 서브에이전트 활용 가이드

토큰 효율화를 위해 작업 종류에 따라 적절한 모델의 서브에이전트를 활용한다.

### Haiku 서브에이전트 (가볍고 빠른 작업)

`model: "haiku"` 사용 대상:

- **코드 검색 및 탐색**: 파일 찾기, 클래스/함수 위치 파악, 프로젝트 구조 탐색
- **단순 코드 수정**: 변수명 변경, 오타 수정, import 추가, 단순 리팩토링
- **패턴 기반 일괄 수정**: 네이밍 컨벤션 통일, 로그 추가/제거
- **파일 생성 (보일러플레이트)**: 빈 클래스 생성, 인터페이스 스텁, 데이터 모델 껍데기
- **Git 작업**: 상태 확인, 로그 조회, 단순 커밋
- **데이터 입력**: ScriptableObject 값 채우기, JSON/CSV 데이터 변환
- **문법 체크 및 린트**: 코드 스타일 검사, 컨벤션 위반 탐지
- **테스트 실행**: 기존 테스트 돌리고 결과 보고

### Sonnet 서브에이전트 (중간 복잡도 작업)

`model: "sonnet"` 사용 대상:

- **기능 구현**: 새로운 시스템/매니저 클래스 작성, 유닛 AI 로직, UI 시스템
- **버그 수정**: 원인 분석이 필요한 버그, 여러 파일에 걸친 수정
- **리팩토링**: 아키텍처 변경, 패턴 적용, 모듈 분리
- **테스트 작성**: 단위 테스트 설계 및 구현
- **SPUM 연동 작업**: 캐릭터 시스템과 SPUM 에셋 통합
- **UI 구현**: 복잡한 UI 플로우, 팝업 시스템, 화면 전환
- **밸런스 시스템**: 데미지 계산 공식, 스탯 스케일링 로직
- **빌드 & 배포**: 빌드 스크립트, CI/CD 설정

### Opus (메인 에이전트)

직접 처리하는 대상:

- **아키텍처 설계**: 전체 시스템 구조 결정, 패턴 선택
- **기획 판단**: 게임 디자인 관련 의사결정 지원
- **복잡한 디버깅**: 여러 시스템에 걸친 복잡한 버그
- **코드 리뷰**: 서브에이전트가 작성한 코드의 품질 검증
- **문서 작성**: 기획서, 기술 문서, 설계 문서

### 서브에이전트 사용 원칙

1. **독립적인 작업 → 병렬 실행**: 서로 의존성 없는 작업은 여러 서브에이전트를 동시에 실행
2. **결과 검증**: 서브에이전트 결과물은 반드시 확인 후 반영
3. **컨텍스트 최소화**: 서브에이전트에게 전체 프로젝트 설명 대신 해당 작업에 필요한 정보만 전달
4. **Haiku 우선**: 판단이 애매하면 Haiku로 먼저 시도, 품질이 부족하면 Sonnet으로 재시도

---

## 작업 요청 템플릿

Claude에게 작업 요청 시 다음 정보를 포함하면 효율적:

```
[작업 종류] feat / fix / refactor / data
[대상 파일/폴더] Assets/Scripts/Unit/
[요구사항] 구체적인 내용
[제약조건] 있으면 명시
[참고] 관련 코드나 기획 내용
```
