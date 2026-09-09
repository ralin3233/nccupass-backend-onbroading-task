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

---

## 實作指引

### 1. 擴充 Pydantic Schemas (`app/schemas/memo.py`)

```python
from datetime import datetime
from pydantic import BaseModel, ConfigDict, Field
from app.schemas.memo import MemoBase, MemoResponse

# 部分更新請求 Schema (所有欄位皆為選填)
class MemoUpdate(BaseModel):
    title: str | None = Field(None, min_length=1, max_length=100)
    content: str | None = None
    is_completed: bool | None = None
    priority: int | None = Field(None, ge=1, le=5)

# 列表分頁回應 Schema
class MemoListResponse(BaseModel):
    total: int
    items: list[MemoResponse]
```

### 2. 擴充 CRUD 與業務邏輯 (`app/crud/memo.py`)

實作多條件組合查詢與動態欄位更新：

```python
from sqlalchemy.orm import Session
from app.models.memo import Memo
from app.schemas.memo import MemoUpdate

def get_memos(
    db: Session,
    skip: int = 0,
    limit: int = 20,
    is_completed: bool | None = None,
    search: str | None = None,
) -> tuple[int, list[Memo]]:
    query = db.query(Memo)

    # 狀態過濾
    if is_completed is not None:
        query = query.filter(Memo.is_completed == is_completed)

    # 關鍵字搜尋 (標題或內文包含關鍵字)
    if search:
        query = query.filter(
            (Memo.title.contains(search)) | (Memo.content.contains(search))
        )

    total = query.count()
    items = query.offset(skip).limit(limit).all()
    return total, items

def update_memo(db: Session, db_memo: Memo, memo_in: MemoUpdate) -> Memo:
    update_data = memo_in.model_dump(exclude_unset=True)
    for field, value in update_data.items():
        setattr(db_memo, field, value)

    db.commit()
    db.refresh(db_memo)
    return db_memo
```

### 3. 實作進階路由端點 (`app/routers/memos.py`)

```python
from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy.orm import Session
from app.database import get_db
from app.crud import memo as crud_memo
from app.schemas.memo import MemoListResponse, MemoResponse, MemoUpdate

router = APIRouter(prefix="/api/v1/memos", tags=["Memos"])

@router.get("", response_model=MemoListResponse)
def list_memos(
    skip: int = Query(0, ge=0, description="跳過前幾筆"),
    limit: int = Query(20, ge=1, le=100, description="取得筆數"),
    is_completed: bool | None = Query(None, description="依完成狀態篩選"),
    search: str | None = Query(None, description="關鍵字搜尋"),
    db: Session = Depends(get_db),
):
    total, items = crud_memo.get_memos(
        db=db, skip=skip, limit=limit, is_completed=is_completed, search=search
    )
    return MemoListResponse(total=total, items=items)

@router.patch("/{memo_id}", response_model=MemoResponse)
def update_memo(memo_id: int, memo_in: MemoUpdate, db: Session = Depends(get_db)):
    db_memo = crud_memo.get_memo_by_id(db=db, memo_id=memo_id)
    if not db_memo:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="找不到指定的備忘錄")
    return crud_memo.update_memo(db=db, db_memo=db_memo, memo_in=memo_in)
```

### 4. 撰寫進階測試 (`tests/test_memos_advanced.py`)

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_list_and_filter_memos():
    # 建立兩筆測試資料
    client.post("/api/v1/memos", json={"title": "期中考複習", "content": "計算機結構"})
    res2 = client.post("/api/v1/memos", json={"title": "買文具", "content": "2B 鉛筆"})
    id2 = res2.json()["id"]

    # 將第二筆標記為已完成
    client.patch(f"/api/v1/memos/{id2}", json={"is_completed": True})

    # 測試篩選未完成項目
    res = client.get("/api/v1/memos?is_completed=false")
    assert res.status_code == 200
    for item in res.json()["items"]:
        assert item["is_completed"] is False

    # 測試關鍵字搜尋
    search_res = client.get("/api/v1/memos?search=鉛筆")
    assert search_res.status_code == 200
    assert search_res.json()["total"] >= 1
```

---

## 驗收與 PR 繳交方式

1. 執行完整的測試套件：
   ```bash
   pytest
   ```
   確認所有測試皆順利通過（All Green）。
2. 將所有變更 Commit 並推送到遠端分支（例如 `feat/sprint-2-list-and-update`）。
3. 前往 GitHub 開啟 Pull Request，標題遵循 Conventional Commits 格式（例如 `feat: implement memo list filtering, patch update and tests`）。
4. 在 PR 說明中附上測試執行結果截圖，Tag 你的 Mentor 請求 Code Review！
