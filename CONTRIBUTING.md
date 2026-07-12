# Knowlter Skills 기여 가이드

## 시작하기

1. `AGENTS.md`, `README.md`, `SPEC.md`를 읽는다.
2. 공식 API와 사이트의 robots 정책 및 이용 조건을 확인한다.
3. 로그인, 세션, JavaScript와 CAPTCHA 요구 여부를 기록한다.
4. 기존 구조를 재사용하고 하나의 Operation부터 구현한다.

## 새 Site 추가

`sites/<country>/<site-id>/site.yaml`에 사이트 식별정보, 허용 호스트, 세션 정책, 요청 제한과 유지보수 상태를 선언한다. 실제 검증 전에는 `verification_mode: design-only`를 사용한다.

## 새 Operation 추가

`operations/<operation-id>.yaml`에 입력, 요청 파라미터 연결, 응답 형식과 파서를 선언한다. 검증하지 않은 URL·필드·선택자는 `__unverified__`로 명확히 표시한다.

## 새 Skill 추가

```text
skills/<skill-name>/
├── SKILL.md
├── agents/openai.yaml
└── references/io-contract.md
```

스킬명은 소문자 영문, 숫자와 하이픈만 사용한다. `SKILL.md` frontmatter에는 `name`과 `description`만 둔다.

## Fixture와 테스트

실제 응답에서 이름, 연락처, 식별 가능한 주소, 고유 식별자, 쿠키, 토큰과 추적 ID를 제거한다. 다음 파일을 함께 제출한다.

```text
fixtures/<response>
expected/<result>.json
tests/<operation>.test.*
```

기본 테스트는 외부 사이트를 호출하지 않아야 한다.

## Pull Request 체크리스트

- [ ] 공식 API 조사 결과를 기록했다.
- [ ] fetch-only 여부와 브라우저 사용 여부를 기록했다.
- [ ] robots 정책과 이용 조건을 확인했다.
- [ ] CAPTCHA 우회가 없다.
- [ ] 인증정보와 실제 사용자 쿠키가 없다.
- [ ] fixture를 익명화했다.
- [ ] 요청 속도와 동시성 제한을 설정했다.
- [ ] schema와 fixture 테스트를 실행했다.
- [ ] 마지막 실사이트 검증 날짜와 제한 사항을 기록했다.
- [ ] `git diff --check`를 통과했다.
