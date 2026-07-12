---
name: extract-auction-case-detail
description: search-auction-cases 결과에서 사용자가 선택한 대한민국 법원경매 사건의 상세 페이지를 조회해 사건, 물건, 기일, 감정가와 최저가 정보를 구조화한다. 후보 선택 후 상세정보 추출이나 선택된 사건 페이지 분석이 필요할 때 사용한다.
---

# 법원경매 사건 상세 추출

## 절차

1. 검색 결과의 `selection_id`와 `detail_url` 또는 동등한 사건 식별자를 받는다.
2. URL의 스킴과 호스트를 어댑터의 허용 범위와 대조한다. 임의 외부 URL은 거부한다.
3. `../../site.yaml`과 `../../operations/get-case-detail.yaml`을 검증하고 `get_case_detail` Operation을 실행한다.
4. 사건 기본정보, 물건 목록, 기일, 가격과 상태를 표준 결과로 정규화한다.
5. 금액은 통화와 정수 값을 분리하고, 날짜는 원문과 정규화 값을 보존한다.
6. 누락된 필드를 추측하지 말고 `null` 및 경고로 표시한다.

입출력 계약은 [references/io-contract.md](references/io-contract.md)를 따른다.

## 실패 처리

- 검색 결과와 무관한 URL이면 `invalid_input`을 반환한다.
- 403, 429 또는 challenge 페이지를 상세정보로 파싱하지 않는다.
- CAPTCHA나 사람 인증은 `user_authentication_required`로 반환한다.
- 필수 사건 식별정보가 없으면 `parse_error`를 반환한다.
- 일부 섹션만 추출되면 `partial`과 누락 섹션 경고를 반환한다.

## 안전 규칙

- CAPTCHA를 우회하지 않는다.
- 사용자 쿠키, 토큰, 개인정보를 저장하지 않는다.
- 상세 페이지에 포함된 개인 식별정보를 fixture에 커밋하지 않는다.
- 원본 HTML 전체를 모델 컨텍스트에 전달하지 않는다.
