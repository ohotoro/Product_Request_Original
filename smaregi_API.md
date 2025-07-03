````markdown
# Smaregi Platform API – POS 스코프 & 엔드포인트 정리  
<small>최종 업데이트 2025-07-03</small>

---

## 1. 인증 (Client Credentials Flow)

| 항목 | 값 |
|------|----|
| **토큰 URL** | `{TOKEN_URL}/token` |
| **메서드** | `POST` (`application/x-www-form-urlencoded`) |
| **필드** | `grant_type=client_credentials`<br>`client_id={CLIENT_ID}`<br>`client_secret={CLIENT_SECRET}`<br>`scope=공백으로 구분된 스코프` |
| **응답 예시** | ```json { "access_token":"xxx", "token_type":"Bearer", "expires_in":3600, "scope":"pos.stock:read pos.transactions:read" }``` |

> 발급된 `access_token` 은 **3600 초(1시간)** 유효. 만료 5 분 전 새로 발급 권장.

---

## 2. POS API 스코프 목록

| Scope | 설명 | 권한 |
|-------|------|------|
| `pos.products:read` | 상품·부문 조회 | 읽기 |
| `pos.products:write` | 상품·부문 등록·수정 | 쓰기 |
| `pos.stock:read` | 재고 조회 | 읽기 |
| `pos.stock:write` | 재고 조정(입출고) | 쓰기 |
| `pos.transactions:read` | 판매(거래) 조회 | 읽기 |
| `pos.transactions:write` | 거래 수기 등록·취소 | 쓰기 |
| `pos.stores:read` | 지점 정보 조회 | 읽기 |
| `pos.stores:write` | 지점 정보 수정 | 쓰기 |
| `pos.orders:read` | 발주(주문) 조회 | 읽기 |
| `pos.orders:write` | 발주 등록·수정·취소 | 쓰기 |
| `pos.customers:read` | 회원·포인트 조회 | 읽기 |
| `pos.customers:write` | 회원·포인트 수정 | 쓰기 |
| `pos.suppliers:read` | 공급처 조회 | 읽기 |
| `pos.suppliers:write` | 공급처 수정 | 쓰기 |

> 여러 스코프는 공백(` `)으로 구분하여 `scope` 파라미터에 전달  
> 예) `pos.stock:read pos.transactions:read`

---

## 3. 주요 엔드포인트

### 3-1. 재고 조회 `/pos/stock`

````

GET [https://api.smaregi.jp/{contract\_id}/pos/stock](https://api.smaregi.jp/{contract_id}/pos/stock)
Header: Authorization: Bearer {access\_token}

Query Parameters
storeId   int   (옵션)
productId int   (옵션)
page      int   (기본 1)
limit     int   1-1000 (기본 100)

````

**응답 예시**

```json
[
  {
    "productId": 1001,
    "storeId": 1,
    "stockAmount": 12,
    "updatedAt": "2025-07-03T12:34:56+09:00"
  }
]
````

---

### 3-2. 판매(거래) 조회 `/pos/transactions`

```
GET https://api.smaregi.jp/{contract_id}/pos/transactions
Header: Authorization: Bearer {access_token}

Query Parameters
  date      YYYY-MM-DD  (필수)
  storeId   int         (옵션)
  page      int         (기본 1)
  limit     int         1-1000
```

**응답 예시**

```json
[
  {
    "transactionId": "T202507030001",
    "storeId": 1,
    "transactionDate": "2025-07-03T10:15:00+09:00",
    "total": 12000,
    "items": [
      { "productId": 1001, "qty": 1, "price": 12000 }
    ]
  }
]
```

---

## 4. 공통 HTTP 오류 코드

| 코드  | 의미·대응                             |
| --- | --------------------------------- |
| 401 | 토큰 누락·만료 → 새 토큰 발급 후 재시도          |
| 403 | Scope 부족 → 앱 스코프 설정 확인            |
| 429 | Rate limit 초과 → 요청 간격 늘리기         |
| 500 | 내부 오류 → 재시도 후 지속 시 Smaregi 지원팀 문의 |

---

## 5. 호출 베스트프랙티스

1. **페이지네이션** – `limit`(최대 1000)과 `page` 반복 호출로 전량 수집
2. **트래픽 제한** – 병렬 호출 지양, 1초당 요청 수 제한 준수
3. **조건 필터링** – `date`, `storeId` 등으로 데이터 범위 최소화
4. **토큰 캐싱** – ScriptProperties에 5 분 캐싱 후 갱신

---

## 6. 토큰 발급 cURL 예시

```bash
curl -X POST https://id.smaregi.jp/app/token \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "scope=pos.stock:read pos.transactions:read"
```

---

> **Claude 지시 예시**
> `update SmaregiSync.gs using the spec in Smaregi_API.md`

```
```
