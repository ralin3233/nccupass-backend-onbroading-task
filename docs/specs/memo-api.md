# 備忘錄系統 API 規格書 (API Specification)

本規格書定義政大通 Onboarding Task「備忘錄系統（Memo Service）」的所有 RESTful API 端點與資料交換合約。

---

## 基礎資訊 (Base Info)

- **Base URL**：`http://localhost:8000/api/v1`
- **Content-Type**：`application/json; charset=utf-8`

---

## 備忘錄模組 (Memo APIs)

### 1. 建立備忘錄
- **Endpoint**: `POST /memos`
- **Status Code**: `201 Created`

**Request Body**:
```json
{
  "title": "資結期末專案討論",
  "content": "討論後端架構設計與 ER Model",
  "priority": 4
}
```

**Response (201 Created)**:
```json
{
  "id": 10,
  "title": "資結期末專案討論",
  "content": "討論後端架構設計與 ER Model",
  "priority": 4,
  "is_completed": false,
  "created_at": "2026-09-07T14:30:00Z",
  "updated_at": "2026-09-07T14:30:00Z"
}
```

---

### 2. 查詢備忘錄列表 (分頁與過濾)
- **Endpoint**: `GET /memos`
- **Status Code**: `200 OK`
- **Query Parameters**:
  - `skip` (整數，預設 0)：跳過前幾筆
  - `limit` (整數，預設 20，最大 100)：取得筆數
  - `is_completed` (布林值，選填)：`true` 或 `false`
  - `search` (字串，選填)：搜尋標題或內容關鍵字

**Response (200 OK)**:
```json
{
  "total": 1,
  "items": [
    {
      "id": 10,
      "title": "資結期末專案討論",
      "content": "討論後端架構設計與 ER Model",
      "priority": 4,
      "is_completed": false,
      "created_at": "2026-09-07T14:30:00Z",
      "updated_at": "2026-09-07T14:30:00Z"
    }
  ]
}
```

---

### 3. 取得單筆備忘錄
- **Endpoint**: `GET /memos/{id}`
- **Path Parameter**: `id` (整數)
- **Status Code**: `200 OK`

**Response (200 OK)**:
```json
{
  "id": 10,
  "title": "資結期末專案討論",
  "content": "討論後端架構設計與 ER Model",
  "priority": 4,
  "is_completed": false,
  "created_at": "2026-09-07T14:30:00Z",
  "updated_at": "2026-09-07T14:30:00Z"
}
```

---

### 4. 更新備忘錄 (部分更新)
- **Endpoint**: `PATCH /memos/{id}`
- **Path Parameter**: `id` (整數)
- **Status Code**: `200 OK`

**Request Body (所有欄位皆選填)**:
```json
{
  "is_completed": true,
  "priority": 2
}
```

**Response (200 OK)**:
```json
{
  "id": 10,
  "title": "資結期末專案討論",
  "content": "討論後端架構設計與 ER Model",
  "priority": 2,
  "is_completed": true,
  "created_at": "2026-09-07T14:30:00Z",
  "updated_at": "2026-09-07T15:10:00Z"
}
```

---

### 5. 刪除備忘錄
- **Endpoint**: `DELETE /memos/{id}`
- **Path Parameter**: `id` (整數)
- **Response (204 No Content)**: 無回傳 Body
