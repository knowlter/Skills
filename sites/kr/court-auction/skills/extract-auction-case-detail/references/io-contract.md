# 입출력 계약

## 입력

검색 스킬이 반환한 후보 하나를 전달한다.

```json
{
  "selection_id": "kr-court-auction:<court-code>:2025타경123",
  "detail_url": "https://example.invalid/__unverified__/detail"
}
```

## 출력

```json
{
  "status": "ok",
  "case": {
    "court_name": "<court-name>",
    "case_number": "2025타경123",
    "case_status": "<status>",
    "filing_date": null
  },
  "items": [
    {
      "item_number": "1",
      "category": "<category>",
      "appraised_value": {"amount": 0, "currency": "KRW"},
      "minimum_bid": {"amount": 0, "currency": "KRW"},
      "sale_date": null,
      "status": "<status>"
    }
  ],
  "warnings": [],
  "source": {
    "site_id": "kr-court-auction",
    "operation": "get_case_detail",
    "fetched_at": "<RFC3339>"
  }
}
```

fixture에는 이름, 연락처, 상세 주소 등 개인을 식별할 수 있는 값을 익명화한다.
