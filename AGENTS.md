# AGENTS.md

## 저장소 목적

Knowlter Skills는 공개 웹사이트의 검색·조회 절차를 Site, Operation, Skill, fixture와 테스트로 관리한다.

## 필수 읽기 순서

1. `AGENTS.md`
2. `README.md`
3. `SPEC.md`
4. 대상 사이트의 `site.yaml`
5. 대상 Skill의 `SKILL.md`
6. 연결된 Operation과 `references/io-contract.md`

## 작업 원칙

1. 공식 API가 있는지 먼저 확인한다.
2. 브라우저보다 fetch-only 방식을 우선한다.
3. 한 번에 하나의 명확한 Operation을 구현한다.
4. 실제 사용자 쿠키, 토큰과 인증정보를 커밋하지 않는다.
5. fixture를 반드시 익명화한다.
6. 검증되지 않은 값은 `unverified`로 표시한다.
7. CAPTCHA 또는 사람 인증을 우회하지 않는다.
8. 사이트 공통 설정을 Skill마다 복제하지 않는다.
9. 공통 runtime 수정 시 전체 fixture 테스트를 실행한다.
10. 라이브 사이트 호출을 기본 PR CI에 넣지 않는다.

## 설치 지시

1. `https://github.com/knowlter/Skills.git`을 clone한다.
2. 위 필수 읽기 순서를 따른다.
3. `kts` 실행 파일이 실제로 존재하는지 확인하기 전에는 계획된 명령을 실행 가능하다고 가정하지 않는다.
4. 설치 과정에서 원격 수정, push 또는 인증 설정을 임의로 수행하지 않는다.

## 기여 지시

1. 기존 `sites/`에서 유사한 Site, Operation, Skill을 찾는다.
2. `site.yaml`에는 공통 접근·세션·제한 정책만 둔다.
3. `operations/`에는 요청, 응답과 파싱 규칙을 둔다.
4. `SKILL.md`에는 operation의 사용 조건과 절차만 간결하게 둔다.
5. 실제 응답을 쓸 때 익명화 fixture, expected JSON과 회귀 테스트를 함께 추가한다.
6. 검증 결과, 마지막 실사이트 확인 날짜와 알려진 제한을 보고한다.

## 변경 범위

새 사이트나 Skill은 원칙적으로 `sites/<country>/<site-id>/` 아래만 변경한다. 공통 형식 변경이 필요하면 `schemas/`와 `SPEC.md`를 함께 갱신하고 기존 전체 검증을 수행한다.

## 필수 검증

- schema validation
- fixture parser test
- expected JSON comparison
- secret scan
- existing regression tests
- `git diff --check`

아직 검증 도구나 fixture가 없는 단계에서는 누락 사실을 완료 보고에 명시하고 통과했다고 주장하지 않는다.

## 완료 보고

- 변경 파일
- 수행한 검증과 결과
- 마지막 실사이트 확인 날짜
- 알려진 제한 사항
- 아직 구현되지 않은 기능
