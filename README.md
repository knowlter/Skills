# Knowlter Skills

`Knowlter/Skills`는 공개 웹사이트의 검색·조회 절차를 코딩 에이전트가 재현하고 검증할 수 있도록 관리하는 오픈소스 Site Skill Registry입니다.

단순한 스크래핑 코드 대신 다음 요소를 함께 관리하는 것을 목표로 합니다.

- 에이전트의 사용 절차를 설명하는 `SKILL.md`
- 요청과 파싱 규칙을 선언하는 `adapter.yaml`
- 개인정보를 제거한 fixture와 예상 결과
- 사이트 변경을 감지하는 회귀 테스트
- 여러 에이전트가 공통으로 호출할 수 있는 CLI와 MCP 인터페이스

프로젝트의 CLI 명칭은 **KTS(KnowlTer Skills)** 입니다.

> 현재 상태: 초기 설계 및 스킬 골격 단계입니다. KTS 런타임과 CLI는 아직 구현되지 않았으며, 법원경매 어댑터의 실제 엔드포인트와 선택자도 검증 전입니다. 현재 `example.invalid` 또는 `__unverified__`로 표시된 설정을 라이브 호출에 사용하지 마세요.

## 제공 중인 스킬

### 법원경매 사건 검색

경매사건번호를 정규화하고, 법원별 후보 사건을 선택 가능한 목록으로 반환합니다.

- 경로: [`sites/kr/court-auction/skills/search-auction-cases`](sites/kr/court-auction/skills/search-auction-cases)
- operation: `search_case`
- 주요 출력: `selection_id`, 법원명, 사건번호, 상태, 물건 수, 상세 URL

### 법원경매 사건 상세 추출

검색 결과에서 선택한 사건 페이지로부터 사건, 물건, 기일, 감정가와 최저가를 구조화합니다.

- 경로: [`sites/kr/court-auction/skills/extract-auction-case-detail`](sites/kr/court-auction/skills/extract-auction-case-detail)
- operation: `get_case_detail`
- 입력 연결: 검색 스킬의 `selection_id`와 검증된 `detail_url`

## 저장소 구조

```text
Skills/
├── README.md
├── docs/
│   └── agent_friendly_web_site_skill_repository_design.md
└── sites/
    └── kr/
        └── court-auction/
            └── skills/
                ├── search-auction-cases/
                │   ├── SKILL.md
                │   ├── adapter.yaml
                │   ├── agents/openai.yaml
                │   └── references/io-contract.md
                └── extract-auction-case-detail/
                    ├── SKILL.md
                    ├── adapter.yaml
                    ├── agents/openai.yaml
                    └── references/io-contract.md
```

전체 설계는 [`docs/agent_friendly_web_site_skill_repository_design.md`](docs/agent_friendly_web_site_skill_repository_design.md)를 참고하세요.

## 설치

### 저장소 개발자로 설치

GitHub 저장소가 공개된 이후 다음과 같이 복제합니다.

```bash
git clone https://github.com/Knowlter/Skills.git
cd Skills
```

현재는 별도의 패키지 설치나 빌드 과정이 없습니다. KTS CLI가 구현되기 전까지 에이전트는 필요한 스킬의 `SKILL.md`, `adapter.yaml`, `references/`를 직접 읽어야 합니다.

### 에이전트 스킬로 설치

현재 자동 설치 명령은 제공하지 않습니다. 사용하는 에이전트가 로컬 스킬 디렉터리를 지원한다면, 필요한 스킬 폴더 전체를 해당 에이전트의 스킬 경로로 복사하거나 심볼릭 링크로 연결할 수 있습니다.

폴더 하나에 포함된 `SKILL.md`, `agents/`, `references/`, `adapter.yaml`의 상대 경로를 유지해야 합니다. `SKILL.md`만 단독 복사하지 마세요.

향후 KTS CLI에서는 다음 형태의 설치 인터페이스를 제공할 예정입니다.

```bash
KTS install github:Knowlter/Skills/sites/kr/court-auction/skills/search-auction-cases
```

위 명령은 아직 구현되지 않은 계획 인터페이스입니다.

## 기여 방법

### 1. 작업 전 확인

1. 이 README와 설계 문서를 읽습니다.
2. 추가하려는 사이트가 공식 API를 제공하는지 먼저 확인합니다.
3. 사이트의 robots 정책, 이용 조건, 로그인·세션·CAPTCHA 요구 여부를 기록합니다.
4. 기존 `sites/`에서 가장 비슷한 스킬과 어댑터를 찾습니다.
5. 한 번의 기여에서는 가능한 한 하나의 명확한 operation부터 구현합니다.

### 2. 스킬 구성

스킬 이름은 소문자 영문, 숫자, 하이픈만 사용하고 동작 중심으로 작성합니다.

```text
sites/<country>/<site-id>/skills/<skill-name>/
├── SKILL.md
├── adapter.yaml
├── agents/
│   └── openai.yaml
└── references/
    └── io-contract.md
```

`SKILL.md` frontmatter에는 `name`과 `description`만 둡니다. description에는 무엇을 하는지와 언제 사용해야 하는지를 모두 포함합니다.

### 3. 실제 동작을 구현할 때

접근 방식은 다음 우선순위를 따릅니다.

1. 공식 API
2. 공개 GET 요청
3. 세션을 포함한 GET·POST 요청
4. 필요한 JavaScript 계산만 별도 실행
5. 브라우저 런타임
6. 사용자 직접 인증이 필요한 브라우저 세션

브라우저는 우선 요청 흐름을 분석하는 도구로 사용하고, 가능하면 최종 동작은 fetch-only 방식으로 재현합니다.

### 4. Fixture와 테스트

실제 응답을 fixture로 추가할 때 다음 정보를 제거합니다.

- 이름, 전화번호 및 개인을 식별할 수 있는 상세 주소
- 주민등록번호, 사업자번호 등 고유 식별자
- 쿠키, 세션 ID, CSRF 토큰과 인증 헤더
- 추적 식별자와 개인 계정 정보

fixture에는 대응하는 예상 JSON과 회귀 테스트를 함께 추가해야 합니다. 라이브 사이트 호출은 기본 PR 테스트에 포함하지 않습니다.

### 5. 제출 전 검사

현재 자동 검증 도구가 완성되기 전까지 최소한 다음 항목을 확인합니다.

```bash
git diff --check
git status --short
```

추가로 확인할 사항:

- `SKILL.md`의 YAML frontmatter와 스킬 이름
- `adapter.yaml`의 YAML 구문 및 요청 제한
- fixture의 개인정보·토큰 제거 여부
- expected JSON과 파서 결과의 일치 여부
- 기존 스킬의 회귀 테스트 통과 여부

KTS CLI가 구현되면 다음 명령을 표준 검증 절차로 사용할 예정입니다.

```bash
KTS validate <site-or-skill-path>
KTS test <site-or-skill-path>
```

### 6. Pull Request 작성

PR 설명에는 다음 내용을 포함합니다.

- 사이트와 operation 설명
- 공식 API 조사 결과
- fetch-only 재현 여부와 브라우저 사용 여부
- 세션, 로그인, CAPTCHA 요구 여부
- robots 정책 및 이용 조건 확인 결과
- fixture 익명화 여부
- 수행한 테스트와 결과
- 마지막 실사이트 검증 날짜
- 알려진 제한 사항

## 주의사항

다음 행위는 이 저장소에서 지원하지 않습니다.

- CAPTCHA 또는 사람 인증 절차 자동 우회
- 접근 제한, 속도 제한 또는 사이트 보안 정책 회피
- 사용자 인증정보, 쿠키, 세션 토큰의 저장·공유·커밋
- 개인정보가 포함된 원본 응답 커밋
- 과도한 병렬 요청이나 무제한 법원·사건 순회
- 사이트 이용약관이나 robots 정책을 무시하는 기본 구현
- 검증하지 않은 엔드포인트와 선택자를 정상 동작하는 것처럼 표시

HTTP 403, 429, CAPTCHA, challenge 페이지는 정상 콘텐츠로 파싱하지 않습니다. 사이트 구조가 바뀌었으면 `adapter_outdated`, 일부 정보만 확보했으면 `partial` 상태를 명시해야 합니다.

법원경매 정보에는 개인정보 또는 민감한 재산 정보가 포함될 수 있습니다. 필요한 최소 필드만 처리하고, fixture와 로그에는 반드시 비식별화된 값만 남기세요.

## 코딩 에이전트 지시문

다음 지시문은 Codex, Claude Code 등 저장소를 직접 수정하는 코딩 에이전트에 전달할 수 있습니다.

### 저장소 설치 지시문

```text
Knowlter/Skills 저장소를 로컬 작업 공간에 설치하라.

1. https://github.com/Knowlter/Skills.git 을 clone하고 저장소 루트로 이동하라.
2. README.md와 docs/agent_friendly_web_site_skill_repository_design.md를 먼저 읽어라.
3. 사용할 스킬의 SKILL.md를 읽고, 연결된 adapter.yaml과 references/ 파일을 확인하라.
4. KTS 명령이 실제로 존재하는지 확인하기 전에는 문서의 예정 명령을 실행 가능한 것으로 가정하지 마라.
5. 설치 과정에서 원격 저장소 수정, push, 인증정보 변경을 임의로 수행하지 마라.
6. 설치 결과로 저장소 경로, 현재 브랜치, 사용 가능한 스킬과 미구현 기능을 요약하라.
```

### 새 스킬 기여 지시문

```text
Knowlter/Skills 저장소에 공개 웹사이트용 최소 스킬 하나를 기여하라.

필수 절차:
1. README.md와 설계 문서를 읽어라.
2. sites/ 아래에서 유사한 스킬을 찾아 구조와 계약을 재사용하라.
3. 공식 API, robots 정책, 이용 조건, 로그인·세션·CAPTCHA 요구 여부를 조사하라.
4. 브라우저 자동화보다 fetch-only 방식을 우선하고 하나의 operation만 먼저 구현하라.
5. sites/<country>/<site-id>/skills/<skill-name>/ 아래에 SKILL.md, adapter.yaml,
   agents/openai.yaml, references/io-contract.md를 작성하라.
6. SKILL.md frontmatter에는 name과 description만 사용하라.
7. 실제 응답을 사용하는 경우 개인정보와 모든 토큰을 제거한 fixture 및 expected JSON을 추가하라.
8. fixture 기반 테스트를 작성하고 기존 회귀 테스트와 함께 실행하라.
9. 검증되지 않은 URL, 파라미터, 선택자는 unverified로 표시하고 정상 동작을 주장하지 마라.
10. 변경 파일, 검증 결과, 마지막 실사이트 확인 날짜와 알려진 제한을 요약하라.

금지 사항:
- CAPTCHA 우회
- 인증정보 또는 사용자 쿠키 저장
- 접근 제한 회피
- 무제한 크롤링
- 개인정보가 포함된 fixture 커밋
- 라이브 사이트 테스트를 기본 CI에 추가
- 사용자 승인 없는 push 또는 PR 생성
```

### 법원경매 스킬 작업 지시문

```text
법원경매 스킬을 작업할 때 search-auction-cases와
extract-auction-case-detail을 분리해 유지하라.

검색 스킬은 선택 가능한 후보 목록과 안정적인 selection_id를 반환해야 한다.
상세 스킬은 사용자가 선택한 후보만 처리하고 URL의 스킴과 허용 호스트를 다시 검증해야 한다.
검색 결과가 하나뿐이어도 상세 조회를 자동 실행하지 마라.
누락된 값은 추측하지 말고 null과 warnings 또는 partial 상태로 표현하라.
```

## 라이선스 및 공개 전 확인

현재 저장소에는 라이선스 파일이 아직 추가되지 않았습니다. 외부 기여를 받거나 코드를 재배포하기 전에 프로젝트 정책에 맞는 오픈소스 라이선스와 `CONTRIBUTING.md`, `SECURITY.md`를 확정해야 합니다.
