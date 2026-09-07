# 備忘錄系統 API 規格書 (API Specification)

本規格書定義政大通 Onboarding Task「備忘錄系統（Memo Service）」的所有 RESTful API 端點與資料交換合約。

---

## 基礎資訊 (Base Info)

- **Base URL**：`http://localhost:8000/api/v1`
- **Content-Type**：`application/json; charset=utf-8`
- **Authentication**：HTTP Bearer Token（`Authorization: Bearer <JWT_TOKEN>`）

---

## 認證模組 (Authentication APIs)

### 1. 註冊新帳號
- **Endpoint**: `POST /auth/register`
- **Auth**: 不需要

**Request Body**:
```json
{
  "email": "student@nccu.edu.tw",
  "username": "nccu_coder",
  "password": "SecurePassword123!"
}
```

**Response (201 Created)**:
```json
{
  "user_id": 1,
  "email": "student@nccu.edu.tw",
  "username": "nccu_coder",
  "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
  "token_type": "bearer"
}
```

---

### 2. 登入取得 Token
- **Endpoint**: `POST /auth/login`
- **Auth**: 不需要

**Request Body**:
```json
{
  "email": "student@nccu.edu.tw",
  "password": "SecurePassword123!"
}
```

**Response (200 OK)**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

---

## 備忘錄模組 (Memo APIs)

> [!NOTE]
> 以下所有備忘錄 API 皆必須在 Request Header 帶上有效的 Bearer Token。

---

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
  "owner_id": 1,
  "created_at": "2026-09-07T14:30:00Z",
  "updated_at": "2026-09-07T14:30:00Z"
}
```

---

### 2. 查詢備忘錄列表 (分頁與過濾)
- **Endpoint**: `GET /memos`
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
      "created_at": "2026-09-07T14:30:00Z"
    }
  ]
}
```

---

### 3. 取得單筆備忘錄
- **Endpoint**: `GET /memos/{id}`
- **Path Parameter**: `id` (整數)

**Response (200 OK)**:
```json
{
  "id": 10,
  "title": "資結期末專案討論",
  "content": "討論後端架構設計與 ER Model",
  "priority": 4,
  "is_completed": false,
  "owner_id": 1,
  "created_at": "2026-09-07T14:30:00Z",
  "updated_at": "2026-09-07T14:30:00Z"
}
```

---

### 4. 更新備忘錄 (部分更新)
- **Endpoint**: `PATCH /memos/{id}`
- **Path Parameter**: `id` (整數)

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
  "updated_at": "2026-09-07T15:10:00Z"
}
```

---

### 5. 刪除備忘錄
- **Endpoint**: `DELETE /memos/{id}`
- **Response (204 No Content)**: 無回傳 Body

---

## 統一錯誤回應格式 (Error Specifications)

| 狀態碼 | 錯誤情境 | 回應範例 JSON |
| :--- | :--- | :--- |
| **400 Bad Request** | 業務邏輯錯誤（如信箱已被註冊） | `{"success": false, "error_code": "EMAIL_ALREADY_REGISTERED", "message": "此 Email 已被註冊"}` |
| **401 Unauthorized** | Token 缺失或過期 | `{"success": false, "error_code": "UNAUTHORIZED", "message": "請先登入取得存取憑證"}` |
| **403 Forbidden** | 存取或修改別人的資料 | `{"success": false, "error_code": "FORBIDDEN", "message": "您無權操作此備忘錄"}` |
| **404 Not Found** | 資源不存在 | `{"success": false, "error_code": "MEMO_NOT_FOUND", "message": "找不到指定的備忘錄"}` |
| **422 Validation Error**| 欄位格式或型別錯誤 | `{"success": false, "error_code": "VALIDATION_ERROR", "message": "欄位驗證失敗", "details": [...]}` |
