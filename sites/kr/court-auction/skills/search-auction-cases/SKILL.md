---
name: search-auction-cases
description: 대한민국 법원경매 사이트에서 사용자가 제공한 경매사건번호를 정규화해 후보 사건을 검색하고 선택 가능한 목록으로 반환한다. 사건번호 검색, 법원별 동명이 사건 구분, 상세 조회 전 후보 선택이 필요할 때 사용한다.
---

# 법원경매 사건 검색

## 절차

1. 입력 사건번호에서 연도, 사건구분, 일련번호와 선택적 법원 정보를 분리한다.
2. `../../site.yaml`과 `../../operations/search-case.yaml`을 검증하고 `search_case` Operation을 실행한다.
3. 법원 정보가 없으면 어댑터가 허용한 법원 범위만 낮은 동시성으로 조회한다.
4. 사건번호와 법원 코드 조합으로 결과를 중복 제거한다.
5. 각 후보에 안정적인 `selection_id`, 법원명, 사건번호, 상태, 물건 수, `detail_url`을 포함한다.
6. 후보가 하나여도 상세 추출을 자동 실행하지 말고 선택 가능한 목록을 반환한다.

입출력 계약은 [references/io-contract.md](references/io-contract.md)를 따른다.

## 실패 처리

- 형식이 잘못되면 `invalid_input`을 반환한다.
- 403, 429 또는 challenge 페이지를 검색 결과로 파싱하지 않는다.
- CAPTCHA나 사람 인증은 `user_authentication_required`로 반환한다.
- 선택자가 맞지 않으면 `adapter_outdated`를 반환한다.
- 확보한 후보가 일부뿐이면 `partial`과 경고를 함께 반환한다.

## 안전 규칙

- CAPTCHA를 우회하지 않는다.
- 사용자 쿠키, 토큰, 개인정보를 저장하지 않는다.
- 어댑터의 요청 속도와 동시성 제한을 지킨다.
- 원본 HTML 전체를 모델 컨텍스트에 전달하지 않는다.
