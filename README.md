# Knowlter Skills

Knowlter Skills는 공개 웹사이트의 검색·조회 절차를 코딩 에이전트가 재현하고 검증할 수 있도록 관리하는 오픈소스 Site Skill Registry입니다.

각 사이트를 다음 구조로 표현합니다.

```text
Site       사이트 공통 설정과 접근 정책
Operation  실제 요청과 파싱을 정의하는 실행 단위
Skill      Operation을 언제, 왜, 어떻게 사용할지 설명하는 에이전트 지침
```

CLI 명칭은 `kts`(Knowlter Skills CLI)입니다.

> 현재는 명세 골격만 존재하는 `design-only` 단계입니다. `kts` CLI, fixture 기반 런타임, 실제 법원경매 엔드포인트와 선택자는 아직 구현·검증되지 않았습니다. `example.invalid`와 `__unverified__` 값을 라이브 호출에 사용하지 마세요.

## 제공 Skill

### 법원경매 사건 검색

- Skill: [`search-auction-cases`](sites/kr/court-auction/skills/search-auction-cases/SKILL.md)
- Operation: [`search_case`](sites/kr/court-auction/operations/search-case.yaml)
- 역할: 사건번호로 법원별 후보를 검색하고 안정적인 `selection_id`를 포함한 선택 목록 반환

### 법원경매 사건 상세 추출

- Skill: [`extract-auction-case-detail`](sites/kr/court-auction/skills/extract-auction-case-detail/SKILL.md)
- Operation: [`get_case_detail`](sites/kr/court-auction/operations/get-case-detail.yaml)
- 역할: 사용자가 선택한 사건의 기본정보, 물건, 기일과 가격 정보 구조화

## 빠른 시작

```bash
git clone https://github.com/knowlter/Skills.git
cd Skills
```

현재는 별도의 설치·빌드 명령이 없습니다. 에이전트는 루트 [`AGENTS.md`](AGENTS.md)를 먼저 읽고, 대상 사이트의 `site.yaml`, `SKILL.md`, operation과 입출력 계약을 차례로 확인해야 합니다.

향후 최소 CLI는 다음 인터페이스를 목표로 합니다. 아래 명령은 아직 구현되지 않았습니다.

```bash
kts validate <site-or-skill-path>
kts test <site-or-skill-path>
kts run <site-id> <operation> --fixture <fixture>
```

## 참여하기

- 에이전트 작업 규칙: [`AGENTS.md`](AGENTS.md)
- 기여 절차: [`CONTRIBUTING.md`](CONTRIBUTING.md)
- 명세: [`SPEC.md`](SPEC.md)
- 보안 및 안전 정책: [`SECURITY.md`](SECURITY.md)

검증되지 않은 기능을 정상 동작하는 것처럼 표시하지 않는 것이 이 저장소의 핵심 원칙입니다.

## 라이선스

이 프로젝트는 [MIT License](LICENSE)로 배포됩니다.
