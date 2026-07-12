# Knowlter Skills 명세

## 핵심 모델

```text
Skill → Operation 선택 → Site 설정 사용 → Runtime 실행 → Fixture/Test 검증
```

## Site

사이트 공통 식별정보, 허용 호스트, 접근·세션 정책, 요청 제한, challenge 탐지와 유지보수 상태를 `site.yaml`에 선언한다.

## Operation

결정적인 실행 단위다. `operations/<operation-id>.yaml`에 입력, 요청 파라미터 연결, 응답 형식과 파싱 규칙을 선언한다.

## Skill

코딩 에이전트가 Operation을 언제, 왜, 어떤 순서로 사용할지 설명한다. 실행 세부사항을 중복하지 않고 Site와 Operation을 참조한다.

## Fixture

익명화된 실제 응답이다. `fixture-only` 상태를 사용하려면 fixture, expected JSON과 파서 회귀 테스트가 모두 존재하고 통과해야 한다.

## 유지보수 상태

- `design-only`: 문서와 명세 골격만 존재
- `fixture-only`: 익명화 fixture와 파서 테스트 통과
- `live`: 실제 사이트 요청까지 검증
- `browser-required`: fetch-only 재현 불가
- `broken`: 기존 검증이 더 이상 통과하지 않음

## 표준 결과

```json
{
  "status": "ok",
  "items": [],
  "warnings": [],
  "source": {
    "site_id": "kr-court-auction",
    "operation": "search_case",
    "fetched_at": "2026-07-12T00:00:00Z"
  }
}
```

## 표준 오류

`invalid_input`, `unsupported_operation`, `network_error`, `forbidden`, `rate_limited`, `user_authentication_required`, `browser_required`, `adapter_outdated`, `parse_error`, `partial`을 사용한다.

## `selection_id`

형식은 `<site-id>:<court-stable-id>:<normalized-case-number>`이다.

- URL이나 법원 표시명을 포함하지 않는다.
- 안정적인 법원 코드를 사용한다.
- 사건번호의 공백과 표기 변형을 정규화한다.
- 상세 URL 변경과 무관하게 동일 사건 ID를 유지한다.
- 물건 식별자는 `<selection-id>:item:<item-number>`로 분리한다.

## 스키마

- `schemas/site.schema.json`
- `schemas/operation.schema.json`
- `schemas/normalized-result.schema.json`
- `schemas/error.schema.json`
