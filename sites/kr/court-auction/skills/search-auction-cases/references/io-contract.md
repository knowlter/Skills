# 입출력 계약

## 입력

```json
{"case_number":"2025타경123","court_code":"optional"}
```

## 출력

```json
{
  "status": "ok",
  "query": {"case_number": "2025타경123"},
  "items": [
    {
      "selection_id": "kr-court-auction:<court-code>:2025타경123",
      "court_code": "<court-code>",
      "court_name": "<court-name>",
      "case_number": "2025타경123",
      "status": "<status>",
      "item_count": 0,
      "detail_url": "https://example.invalid/__unverified__/detail"
    }
  ],
  "warnings": [],
  "source": {
    "site_id": "kr-court-auction",
    "operation": "search_case",
    "fetched_at": "<RFC3339>"
  }
}
```

`selection_id`는 상세 추출 스킬로 전달하는 기본 식별자다. URL은 반드시 허용된 법원경매 호스트인지 다시 검증한다.
