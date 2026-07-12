# Agent-Friendly Web Site Skill Repository Design

## 0. 프로젝트 식별자

- GitHub 조직: `Knowlter`
- GitHub 저장소: `Knowlter/Skills`
- 프로젝트명: `Skills`
- CLI 명령: `KTS` (`KnowlTer Skills`)
- 패키지 범위: `@knowlter/skills-*`

문서와 사용자 안내에서는 고유명사와 실행 명령을 각각 `Knowlter/Skills`,
`KTS`로 표기한다. GitHub 저장소 URL은
`https://github.com/Knowlter/Skills`를 기준으로 한다.

## 1. 목적

이 저장소는 사람이 분석한 웹사이트의 검색·조회 규칙을 코딩 에이전트가 재사용할 수 있도록 공유하는 오픈소스 레지스트리다.

단순한 스크래핑 코드 모음이 아니라 다음 요소를 함께 관리한다.

- 사이트별 사용 절차를 설명하는 `SKILL.md`
- 요청과 파싱 규칙을 선언하는 `adapter.yaml`
- 실제 응답을 익명화한 fixture
- 예상 결과 JSON
- 회귀 테스트
- MCP·CLI·에이전트 도구로 연결할 수 있는 표준 인터페이스
- 코딩 에이전트가 새 사이트를 분석하고 기여할 수 있는 작업 규칙

첫 번째 대표 사례는 대한민국 법원경매 사이트의 사건번호 검색이다.

---

## 2. 핵심 원칙

### 2.1 Fetch-only 우선

가능하면 브라우저 자동화보다 일반 HTTP 요청으로 기능을 재현한다.

우선순위는 다음과 같다.

1. 공식 API
2. 공개 GET 요청
3. 세션을 포함한 GET·POST 요청
4. 필요한 JavaScript 계산만 별도 실행
5. 브라우저 런타임
6. 사용자 직접 인증이 필요한 브라우저 세션

브라우저는 최종 구현보다 요청 흐름을 분석하는 도구로 우선 사용한다.

### 2.2 선언형 명세 우선

사이트별 URL, 파라미터, 세션, 파싱 규칙은 가능한 한 코드가 아니라 `adapter.yaml`에 선언한다.

코드는 다음 경우에만 추가한다.

- 동적 토큰 계산
- 복잡한 응답 변환
- 선언형 파서로 처리하기 어려운 예외
- 사이트별 특수 인코딩
- 여러 요청을 연결하는 상태 머신

### 2.3 재현 가능성

모든 어댑터는 다음 산출물을 포함해야 한다.

- 입력 예시
- 요청 흐름
- fixture
- 예상 출력
- 자동 테스트
- 마지막 검증 날짜
- 알려진 제한 사항

### 2.4 안전한 접근

다음 작업은 지원하지 않는다.

- CAPTCHA 자동 우회
- 사람 인증 절차 자동화
- 인증정보·쿠키·세션 토큰 공유
- 접근 제한 회피
- 과도한 병렬 요청
- 개인정보가 포함된 fixture 커밋
- 사이트 이용약관 또는 robots 정책을 무시하는 기본 구현

---

## 3. 저장소 구조

```text
Skills/
├── README.md
├── AGENTS.md
├── SPEC.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
│
├── schemas/
│   ├── site-adapter.schema.json
│   ├── normalized-result.schema.json
│   └── skill-metadata.schema.json
│
├── sites/
│   ├── kr/
│   │   └── court-auction/
│   │       ├── SKILL.md
│   │       ├── adapter.yaml
│   │       ├── README.md
│   │       ├── fixtures/
│   │       │   ├── search-response.html
│   │       │   └── detail-response.html
│   │       ├── expected/
│   │       │   ├── search-result.json
│   │       │   └── detail-result.json
│   │       └── tests/
│   │           ├── search-case.test.ts
│   │           └── parse-detail.test.ts
│   └── common/
│       ├── html-table/
│       └── form-session/
│
├── packages/
│   ├── adapter-runtime/
│   ├── adapter-validator/
│   ├── adapter-cli/
│   └── mcp-server/
│
├── scripts/
│   ├── validate-all
│   ├── test-site
│   ├── redact-fixture
│   └── update-status
│
└── .github/
    ├── ISSUE_TEMPLATE/
    ├── PULL_REQUEST_TEMPLATE.md
    └── workflows/
        ├── validate.yml
        ├── test-fixtures.yml
        └── scheduled-live-check.yml
```

---

## 4. 코딩 에이전트를 위한 `AGENTS.md`

저장소 루트의 `AGENTS.md`는 에이전트가 가장 먼저 읽어야 할 실행 규칙이다.

예시는 다음과 같다.

```markdown
# AGENTS.md

## 저장소 목적

이 저장소는 공개 웹사이트의 검색·조회 규칙을
Skill, 선언형 어댑터, fixture, 테스트로 관리한다.

## 작업 원칙

1. 기존 어댑터 구조를 먼저 확인한다.
2. 새 사이트 분석 전 공식 API 존재 여부를 확인한다.
3. 브라우저 자동화보다 fetch-only 재현을 우선한다.
4. 실제 사용자 쿠키, 토큰, 개인정보를 커밋하지 않는다.
5. fixture는 반드시 익명화한다.
6. 기존 사이트 테스트를 깨뜨리지 않는다.
7. 사이트별 코드는 최소화하고 공통 런타임을 우선 사용한다.
8. CAPTCHA 우회 기능은 구현하지 않는다.
9. 실시간 사이트 호출 테스트는 기본 CI에서 실행하지 않는다.
10. 변경 후 스키마 검증과 fixture 테스트를 모두 수행한다.

## 필수 검증

- adapter schema validation
- fixture parser test
- expected JSON comparison
- secret scan
- format/lint
- existing regression tests

## 변경 범위

새 사이트를 추가할 때는 원칙적으로 다음 디렉토리만 변경한다.

- sites/<country>/<site-id>/
- 공통 기능이 필요한 경우 packages/adapter-runtime/

공통 런타임 변경 시 기존 모든 fixture 테스트를 실행한다.
```

이 파일은 Codex, Claude Code, KTA 등 다양한 코딩 에이전트가 저장소 작업 규칙을 빠르게 이해하도록 한다.

---

## 5. 사이트별 `SKILL.md`

`SKILL.md`는 에이전트가 해당 사이트 기능을 언제 사용하고 어떤 순서로 실행해야 하는지 설명한다.

```markdown
---
name: korean-court-auction-search
description: 대한민국 법원경매 사이트에서 사건번호로 사건 후보를 검색한다.
version: 1
locale: ko-KR
adapter: ./adapter.yaml
---

# 사용 조건

다음 상황에서 이 Skill을 사용한다.

- 사용자가 `2025타경123` 형식의 사건번호를 입력한 경우
- 법원경매 사건 검색을 요청한 경우
- 법원을 지정하지 않은 사건번호 후보 검색이 필요한 경우

# 입력 정규화

사건번호를 다음 필드로 분리한다.

- year
- case_type
- serial
- optional court_code

# 실행 절차

1. `adapter.yaml`을 로드한다.
2. 사건번호 형식을 검증한다.
3. 세션 초기화 요청을 실행한다.
4. 법원 코드가 있으면 해당 법원만 검색한다.
5. 법원 코드가 없으면 허용된 범위에서 후보 법원을 순회한다.
6. 검색 결과를 표준 사건 스키마로 변환한다.
7. 중복 사건을 제거한다.
8. 조회 시각과 출처를 포함한다.

# 결과 표현

후보가 여러 개면 다음 항목을 포함한다.

- 법원명
- 사건번호
- 사건 상태
- 물건 수
- 상세 조회 가능 여부

# 실패 처리

- 403, 429, challenge 응답은 일반 콘텐츠로 처리하지 않는다.
- CAPTCHA가 확인되면 `user_authentication_required`를 반환한다.
- 사이트 구조가 변경된 경우 `adapter_outdated`를 반환한다.
- 부분 데이터만 확보되면 `partial` 상태를 명시한다.

# 금지 사항

- CAPTCHA 자동 해결
- 사용자 인증정보 저장
- 무제한 병렬 검색
- 원본 HTML 전체를 모델 컨텍스트에 전달
```

---

## 6. 선언형 `adapter.yaml`

```yaml
schema_version: 1

site:
  id: kr-court-auction
  name: 대한민국 법원경매
  country: KR
  locale: ko-KR
  base_url: https://example.invalid
  access: public

maintenance:
  status: experimental
  last_verified: 2026-07-12
  verification_mode: fixture-only
  known_limits:
    - court code may be required
    - live endpoint must be verified before release

capabilities:
  - search_case
  - get_case_detail

inputs:
  case_number:
    type: string
    pattern: '^(?<year>\d{4})(?<case_type>타경)(?<serial>\d+)$'

session:
  bootstrap:
    request:
      method: GET
      path: /search/init
    cookies:
      persist: true

operations:
  search_case:
    request:
      method: POST
      path: /search/cases
      encoding: form
      headers:
        Referer: '${site.base_url}/search/init'
      parameters:
        year: '${input.year}'
        caseType: '${input.case_type}'
        serial: '${input.serial}'
        courtCode: '${input.court_code}'
    response:
      format: html
      parser: search-result

parsers:
  search-result:
    rows:
      selector: table.case-list tbody tr
    fields:
      court_name:
        selector: td:nth-child(1)
        transform: trim
      case_number:
        selector: td:nth-child(2)
        transform: normalize-space
      status:
        selector: td:nth-child(3)
        transform: trim
      detail_url:
        selector: td:nth-child(2) a
        attribute: href
        transform: absolute-url

limits:
  concurrency: 3
  requests_per_second: 1
  timeout_seconds: 30
  retries: 2

errors:
  challenge_patterns:
    - captcha
    - verify you are human
    - access denied
  map:
    403: forbidden
    429: rate_limited
```

실제 사이트 주소와 파라미터는 검증 후 반영한다.

---

## 7. 코딩 에이전트가 Skill을 적용하는 절차

에이전트가 특정 Skill을 사용하려면 다음 순서를 따른다.

```text
1. 사용자 요청 분류
2. 관련 Skill 검색
3. SKILL.md 읽기
4. adapter.yaml 검증
5. adapter-runtime으로 실행
6. 표준 JSON 결과 수신
7. Skill의 표현 규칙에 따라 응답
```

에이전트 호출 예시는 다음과 같다.

```bash
KTS run kr/court-auction \
  search_case \
  --input '{"case_number":"2025타경123"}'
```

결과:

```json
{
  "status": "ok",
  "query": {
    "case_number": "2025타경123"
  },
  "items": [
    {
      "court_name": "부산지방법원",
      "case_number": "2025타경123",
      "status": "진행",
      "detail_url": "https://example.invalid/detail/123"
    }
  ],
  "source": {
    "site_id": "kr-court-auction",
    "fetched_at": "2026-07-12T15:00:00+09:00"
  }
}
```

---

## 8. MCP 제공 방식

하나의 어댑터 명세를 MCP 도구로 자동 노출할 수 있다.

```text
adapter.yaml
   ↓
adapter-runtime
   ↓
MCP server
   ├── list_site_skills
   ├── describe_site_skill
   ├── run_site_operation
   ├── validate_site_adapter
   └── test_site_fixture
```

범용 MCP 도구 예시:

```json
{
  "name": "run_site_operation",
  "arguments": {
    "site_id": "kr-court-auction",
    "operation": "search_case",
    "input": {
      "case_number": "2025타경123"
    }
  }
}
```

사이트별 별칭 도구도 생성할 수 있다.

```text
court_auction_search_case
court_auction_get_case_detail
```

초기에는 범용 도구 수를 적게 유지하고, 사용 빈도가 높은 어댑터만 별칭 도구로 제공하는 것이 좋다.

---

## 9. 코딩 에이전트가 새 사이트를 기여하는 절차

에이전트가 새 사이트 어댑터를 추가하는 표준 흐름은 다음과 같다.

### 9.1 조사

1. 사이트 목적과 공개 접근 범위를 확인한다.
2. 공식 API가 존재하는지 확인한다.
3. robots.txt와 이용 조건을 기록한다.
4. 로그인, CAPTCHA, 세션 요구 여부를 분류한다.
5. 브라우저 개발자 도구 또는 HAR을 이용해 요청 흐름을 분석한다.
6. fetch-only 재현 가능성을 검토한다.

### 9.2 최소 어댑터 작성

```text
sites/<country>/<site-id>/
├── SKILL.md
├── adapter.yaml
└── README.md
```

초기 단계에서는 하나의 기능만 구현한다.

예:

```text
search_case
search_notice
lookup_business
get_public_document
```

### 9.3 fixture 생성

실제 응답을 저장한 뒤 다음 데이터를 제거한다.

- 이름
- 전화번호
- 주소 중 개인 식별 가능 부분
- 주민번호·사업자번호
- 세션 쿠키
- CSRF 토큰
- 추적 식별자
- 개인 계정 정보

```bash
KTS redact-fixture input.html \
  --output sites/kr/example/fixtures/search-response.html
```

### 9.4 예상 결과 생성

```json
{
  "items": [
    {
      "id": "sample-001",
      "title": "익명화된 예시",
      "status": "active"
    }
  ]
}
```

### 9.5 테스트 작성

```typescript
import { runFixture } from "@knowlter/skills-runtime";

test("search response is normalized", async () => {
  const result = await runFixture({
    site: "kr/example",
    operation: "search",
    fixture: "search-response.html",
  });

  expect(result).toEqual(
    require("../expected/search-result.json")
  );
});
```

### 9.6 검증

```bash
KTS validate sites/kr/example
KTS test sites/kr/example
pnpm test
```

### 9.7 PR 생성

PR에는 다음 내용을 포함한다.

- 사이트와 기능 설명
- 공식 API 확인 결과
- fetch-only 여부
- 세션 필요 여부
- 브라우저 사용 여부
- fixture 익명화 여부
- 테스트 결과
- 마지막 실사이트 검증 날짜
- 알려진 제한 사항

---

## 10. 에이전트용 기여 프롬프트

저장소에 다음 프롬프트를 제공하면 코딩 에이전트가 일관되게 작업하기 쉽다.

```markdown
# 새 사이트 어댑터 기여 작업

목표:
공개 웹사이트의 특정 검색 또는 조회 기능을
fetch-only 방식으로 재현하는 최소 어댑터를 추가한다.

필수 작업:
1. AGENTS.md와 SPEC.md를 읽는다.
2. 기존 sites/ 아래 유사 어댑터를 찾는다.
3. 공식 API 존재 여부를 확인한다.
4. 요청 흐름을 분석한다.
5. 하나의 operation만 먼저 구현한다.
6. SKILL.md와 adapter.yaml을 작성한다.
7. 익명화한 fixture와 expected JSON을 추가한다.
8. fixture 기반 테스트를 작성한다.
9. 전체 검증 명령을 실행한다.
10. 변경 파일과 알려진 제한을 요약한다.

제약:
- CAPTCHA 우회 금지
- 로그인 정보 저장 금지
- 실사용자 쿠키 커밋 금지
- 무제한 크롤링 금지
- 기존 어댑터 구조 임의 변경 금지
- 라이브 사이트 테스트를 기본 CI에 추가하지 말 것

완료 기준:
- schema validation 통과
- fixture test 통과
- secret scan 통과
- 기존 테스트 통과
- PR 설명에 검증 날짜와 제한 사항 포함
```

---

## 11. 이슈 템플릿

### 새 사이트 요청

```markdown
## 사이트

- 이름:
- URL:
- 국가/언어:

## 필요한 기능

- [ ] 검색
- [ ] 상세 조회
- [ ] 목록 조회
- [ ] 문서 다운로드
- [ ] 기타:

## 입력 예시

예: `2025타경123`

## 예상 출력

예: 사건번호, 법원명, 상태, 상세 URL

## 접근 특성

- 공식 API:
- 로그인:
- 세션:
- JavaScript:
- CAPTCHA:
- robots.txt 확인:

## 참고 자료

브라우저 요청 캡처, HAR, 공개 문서 등이 있으면 첨부한다.
민감한 토큰과 쿠키는 제거한다.
```

### 어댑터 오류 신고

```markdown
## 사이트 ID

## 실패한 operation

## 마지막 정상 동작 날짜

## 현재 오류

- HTTP 상태:
- parser 오류:
- 예상 선택자 부재:
- challenge 페이지 여부:

## 재현 방법

## 민감정보 제거 확인

- [ ] 쿠키 제거
- [ ] 토큰 제거
- [ ] 개인정보 제거
```

---

## 12. PR 템플릿

```markdown
## 변경 유형

- [ ] 새 사이트
- [ ] 새 operation
- [ ] parser 수정
- [ ] 공통 런타임 수정
- [ ] 문서 수정

## 사이트

- ID:
- 이름:
- 국가/언어:

## 구현 기능

## 접근 방식

- [ ] 공식 API
- [ ] GET
- [ ] POST
- [ ] 세션 쿠키
- [ ] JavaScript 계산
- [ ] 브라우저 필요
- [ ] 사용자 인증 필요

## 안전성 확인

- [ ] CAPTCHA 우회 없음
- [ ] 인증정보 없음
- [ ] fixture 익명화 완료
- [ ] 요청 제한 설정
- [ ] robots/이용 조건 기록

## 검증

- [ ] schema validation
- [ ] fixture test
- [ ] expected JSON comparison
- [ ] secret scan
- [ ] 기존 테스트

## 마지막 실사이트 검증

YYYY-MM-DD

## 알려진 제한

## 에이전트 작업 기록

사용한 에이전트와 주요 판단을 간략히 기록한다.
원본 프롬프트나 긴 추론 과정은 포함하지 않는다.
```

---

## 13. CI 설계

### 기본 PR CI

외부 사이트를 호출하지 않고 fixture만 검증한다.

```text
PR
├── YAML/JSON schema validation
├── fixture parser tests
├── expected output comparison
├── secret scan
├── formatting/lint
└── regression tests
```

### 예약된 실사이트 검증

실사이트 검증은 별도 워크플로로 제한한다.

```text
scheduled-live-check
├── 낮은 빈도
├── 동시 요청 제한
├── 허용된 사이트만 실행
├── 실패 시 status issue 생성
└── 자동 우회 또는 반복 재시도 금지
```

상태 예시:

```yaml
maintenance:
  status: verified
  last_verified: 2026-07-12
  verification_mode: live
```

실패가 반복되면:

```yaml
maintenance:
  status: broken
  last_verified: 2026-07-01
  known_limits:
    - search endpoint structure changed
```

---

## 14. 에이전트 친화적인 CLI

```bash
# 저장소 검사
KTS doctor

# 사용 가능한 사이트 나열
KTS list

# 사이트 설명
KTS describe kr/court-auction

# 명세 검증
KTS validate kr/court-auction

# fixture 테스트
KTS test kr/court-auction

# operation 실행
KTS run kr/court-auction search_case \
  --input '{"case_number":"2025타경123"}'

# 새 사이트 골격 생성
KTS init kr/example-site

# HAR에서 요청 후보 추출
KTS inspect-har example.har

# fixture 익명화 검사
KTS scan-fixture kr/example-site

# MCP 서버 실행
KTS mcp serve
```

CLI 출력은 에이전트가 파싱할 수 있도록 `--json` 옵션을 제공한다.

```bash
KTS validate kr/court-auction --json
```

```json
{
  "status": "ok",
  "schema": true,
  "fixtures": 2,
  "tests": 2,
  "warnings": []
}
```

---

## 15. 코딩 에이전트별 적용

### Codex / Claude Code / KTA

저장소 루트의 `AGENTS.md`와 사이트별 `SKILL.md`를 읽고 CLI 또는 MCP를 호출한다.

```text
Agent
├── AGENTS.md
├── sites/.../SKILL.md
└── KTS CLI 또는 MCP
```

### MCP 클라이언트

다음 MCP 서버를 등록한다.

```json
{
  "mcpServers": {
    "knowlter-skills": {
      "command": "KTS",
      "args": ["mcp", "serve"]
    }
  }
}
```

### Skill 설치형 에이전트

사이트별 Skill 디렉토리를 직접 설치할 수 있도록 배포한다.

```bash
KTS install github:Knowlter/Skills/sites/kr/court-auction
```

단, 실행 로직은 Skill 자연어에 의존하지 않고 공통 runtime과 `adapter.yaml`을 사용한다.

---

## 16. 공통 런타임 인터페이스

```typescript
export interface SiteAdapterRuntime {
  describe(siteId: string): Promise<SiteDescription>;

  validate(siteId: string): Promise<ValidationResult>;

  execute(input: {
    siteId: string;
    operation: string;
    args: Record<string, unknown>;
    mode?: "fixture" | "live";
  }): Promise<NormalizedResult>;
}
```

표준 오류:

```text
invalid_input
unsupported_operation
network_error
forbidden
rate_limited
user_authentication_required
browser_required
adapter_outdated
parse_error
partial
```

표준 결과:

```json
{
  "status": "ok",
  "items": [],
  "warnings": [],
  "source": {
    "site_id": "kr-court-auction",
    "operation": "search_case",
    "fetched_at": "2026-07-12T15:00:00+09:00"
  }
}
```

---

## 17. 유지보수 전략

사이트별 규칙은 시간이 지나면 깨진다. 따라서 코드 양보다 변경 감지와 복구 절차가 중요하다.

```text
사이트 변경
→ 예약 검증 실패
→ 자동 이슈 생성
→ fixture 또는 live 응답 비교
→ parser/요청 규칙 수정
→ 회귀 테스트 추가
→ status 복구
```

각 사이트에는 관리자를 지정할 수 있다.

```yaml
maintenance:
  owners:
    - github-user
  status: verified
  last_verified: 2026-07-12
```

관리자가 없는 어댑터는 `unmaintained` 상태로 전환한다.

---

## 18. MVP 범위

### v0.1

```text
- 법원경매 어댑터 1개
- SKILL.md
- adapter.yaml
- fixture
- expected JSON
- parser 테스트
- validate/test/run CLI
- AGENTS.md
```

### v0.2

```text
- 국내 공공사이트 2~3개
- MCP 서버
- GitHub Actions
- 상태 배지
- PR/이슈 템플릿
```

### v0.3

```text
- HAR 분석 보조
- adapter 스캐폴딩
- Skill 자동 생성
- 사이트별 별칭 MCP 도구
- 예약된 실사이트 상태 검증
```

---

## 19. 최종 구조

```text
사람 또는 코딩 에이전트
        ↓
사이트 동작 분석
        ↓
SKILL.md + adapter.yaml
        ↓
fixture + expected JSON + 테스트
        ↓
GitHub PR
        ↓
공개 레지스트리
        ↓
CLI / MCP / 에이전트 Skill
        ↓
여러 코딩 에이전트에서 재사용
```

핵심은 에이전트가 자연어 문서를 읽고 임의로 웹 요청을 만드는 구조가 아니다.

```text
Skill
= 언제, 왜, 어떤 operation을 사용할지 설명

Adapter
= 정확한 요청과 파싱 규칙

Runtime
= 결정적인 실행

Fixture/Test
= 변경 감지와 재현

MCP/CLI
= 여러 에이전트가 호출하는 인터페이스
```

이 구조를 사용하면 코딩 에이전트는 저장소의 소비자이면서 동시에 기여자가 될 수 있다.

- 기존 Skill을 검색해 바로 적용한다.
- 사이트 분석 결과를 표준 형식으로 작성한다.
- fixture와 테스트를 생성한다.
- 검증 후 PR을 제출한다.
- 사이트 변경 시 실패를 분석하고 어댑터를 복구한다.

따라서 저장소의 목표는 단순한 Skill 모음이 아니라 다음과 같이 정의하는 것이 적절하다.

> 공개 웹사이트의 검색·조회 프로토콜을 사람이 분석하고 코딩 에이전트가 재현·검증·기여할 수 있도록 만드는 오픈소스 Site Skill Registry.
