# Sprint 2 - 列表過濾、進階業務邏輯與完整測試

> **Sprint 目標**：擴充 Pydantic Schemas，實作備忘錄列表多條件篩選（分頁、完成狀態、關鍵字搜尋）與部分更新（PATCH）業務邏輯，並完成進階自動化測試與發起 PR。

---

## 任務清單 (Task Checklist)

- [ ] **擴充 Pydantic Schemas** (`app/schemas/memo.py`)
    - [ ] `MemoUpdate`：部分更新請求格式（欄位皆為選填）
    - [ ] `MemoListResponse`：分頁列表回應格式（包含 `total: int` 與 `items: list[MemoResponse]`）
- [ ] **擴充 CRUD / 業務邏輯函式** (`app/crud/memo.py`)
    - [ ] `get_memos(db, skip, limit, is_completed, search)`：支援分頁、狀態過濾與關鍵字搜尋
    - [ ] `update_memo(db, db_memo, memo_in)`：部分更新欄位並提交資料庫
- [ ] **實作進階 API 路由端點** (`app/routers/memos.py`)
    - [ ] `GET /api/v1/memos`：列表查詢端點（支援 Query 參數 `skip`, `limit`, `is_completed`, `search`）
    - [ ] `PATCH /api/v1/memos/{id}`：部分更新端點（更新標題/內文/優先級/完成狀態）
- [ ] **撰寫進階自動化測試** (`tests/test_memos_advanced.py`)
    - [ ] 測試列表分頁與過濾邏輯（如 `is_completed=false`、搜尋關鍵字）
    - [ ] 測試 PATCH 部分更新（確認只更新有帶入的欄位）
    - [ ] 測試更新不存在的 ID 回傳 `404 Not Found`
- [ ] **發起 Pull Request** 進行 Code Review

