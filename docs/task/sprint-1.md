# Sprint 1 - 基礎 CRUD API 與 Schemas 實作

> **Sprint 目標**：在既有的資料庫模型與連線基礎下，定義 Pydantic 資料結構（Schemas）、實作備忘錄的核心建立、單筆查詢與刪除 API 端點，並撰寫基礎自動化測試。

---

## 前置說明

開始前，請確保你已 Clone 練習用專案 Repo：

```bash
git clone https://gitlab.com/nccupass/nccupass-memo-service-for-beginner.git
cd nccupass-memo-service-for-beginner
```

> 練習用 Repo：[nccupass-memo-service-for-beginner (GitLab)](https://gitlab.com/nccupass/nccupass-memo-service-for-beginner.git)

在 Starter 專案中，Mentor 已經幫你準備好了：
- `app/database.py`：資料庫連線引擎與 Session 依賴項（`get_db`）
- `app/models/memo.py`：SQLAlchemy `Memo` 資料表模型（包含 `id`, `title`, `content`, `is_completed`, `priority`, `created_at`, `updated_at` 欄位）

**你的任務是專注在 API 路由、Pydantic 轉換、CRUD 邏輯與測試！**

---

## 任務清單 (Task Checklist)

- [ ] **定義 Pydantic Schemas** (`app/schemas/memo.py`)
    - [ ] `MemoBase`：基礎共用欄位
    - [ ] `MemoCreate`：建立備忘錄請求格式
    - [ ] `MemoResponse`：回傳單筆備忘錄格式（設定 `from_attributes=True`）
- [ ] **實作 CRUD 資料存取函式** (`app/crud/memo.py`)
    - [ ] `create_memo(db, memo_in)`：建立新備忘錄並寫入資料庫
    - [ ] `get_memo_by_id(db, memo_id)`：依 ID 查詢單筆備忘錄
    - [ ] `delete_memo(db, memo_id)`：刪除指定備忘錄
- [ ] **實作 FastAPI 路由端點** (`app/routers/memos.py`)
    - [ ] `POST /api/v1/memos`：建立備忘錄（回傳 `201 Created`）
    - [ ] `GET /api/v1/memos/{id}`：取得單筆備忘錄（查無資料回傳 `404 Not Found`）
    - [ ] `DELETE /api/v1/memos/{id}`：刪除備忘錄（回傳 `204 No Content`，查無資料回傳 `404`）
- [ ] **撰寫單元測試** (`tests/test_memos_basic.py`)
    - [ ] 使用 `TestClient` 測試建立、單筆查詢與刪除流程
    - [ ] 執行 `pytest` 確保所有測試通過

