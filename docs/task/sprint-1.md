# Sprint 1 - 基礎 CRUD API 與 Schemas 實作

> **Sprint 目標**：在既有的資料庫模型與連線基礎下，定義 Pydantic 資料結構（Schemas）、實作備忘錄的核心建立、單筆查詢與刪除 API 端點，並撰寫基礎自動化測試。

---

## 前置說明

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

---

## 實作指引

### 1. Pydantic Schemas 設計 (`app/schemas/memo.py`)

```python
from datetime import datetime
from pydantic import BaseModel, ConfigDict, Field

# 基礎共用屬性
class MemoBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=100, description="備忘錄標題")
    content: str | None = Field(None, description="備忘錄內文")
    priority: int = Field(default=1, ge=1, le=5, description="優先級 (1~5)")

# 建立請求 Schema
class MemoCreate(MemoBase):
    pass

# 回應 Schema (將 SQLAlchemy ORM 物件序列化為 JSON)
class MemoResponse(MemoBase):
    id: int
    is_completed: bool
    created_at: datetime
    updated_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### 2. CRUD 函式封裝 (`app/crud/memo.py`)

將資料庫的存取邏輯封裝在 CRUD 函式中，讓 Router 保持簡潔：

```python
from sqlalchemy.orm import Session
from app.models.memo import Memo
from app.schemas.memo import MemoCreate

def create_memo(db: Session, memo_in: MemoCreate) -> Memo:
    db_memo = Memo(**memo_in.model_dump())
    db.add(db_memo)
    db.commit()
    db.refresh(db_memo)
    return db_memo

def get_memo_by_id(db: Session, memo_id: int) -> Memo | None:
    return db.query(Memo).filter(Memo.id == memo_id).first()

def delete_memo(db: Session, db_memo: Memo) -> None:
    db.delete(db_memo)
    db.commit()
```

### 3. API 路由端點 (`app/routers/memos.py`)

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from app.database import get_db
from app.crud import memo as crud_memo
from app.schemas.memo import MemoCreate, MemoResponse

router = APIRouter(prefix="/api/v1/memos", tags=["Memos"])

@router.post("", response_model=MemoResponse, status_code=status.HTTP_201_CREATED)
def create_memo(memo_in: MemoCreate, db: Session = Depends(get_db)):
    return crud_memo.create_memo(db=db, memo_in=memo_in)

@router.get("/{memo_id}", response_model=MemoResponse)
def get_memo(memo_id: int, db: Session = Depends(get_db)):
    memo = crud_memo.get_memo_by_id(db=db, memo_id=memo_id)
    if not memo:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="找不到指定的備忘錄")
    return memo

@router.delete("/{memo_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_memo(memo_id: int, db: Session = Depends(get_db)):
    memo = crud_memo.get_memo_by_id(db=db, memo_id=memo_id)
    if not memo:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="找不到指定的備忘錄")
    crud_memo.delete_memo(db=db, db_memo=memo)
    return None
```

### 4. 撰寫自動化測試 (`tests/test_memos_basic.py`)

使用 FastAPI 提供的 `TestClient` 撰寫測試案例：

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_and_get_memo():
    # 1. 建立備忘錄
    res = client.post("/api/v1/memos", json={"title": "買牛奶", "content": "全脂鮮乳", "priority": 2})
    assert res.status_code == 201
    data = res.json()
    assert data["title"] == "買牛奶"
    memo_id = data["id"]

    # 2. 查詢該備忘錄
    get_res = client.get(f"/api/v1/memos/{memo_id}")
    assert get_res.status_code == 200
    assert get_res.json()["id"] == memo_id

    # 3. 刪除該備忘錄
    del_res = client.delete(f"/api/v1/memos/{memo_id}")
    assert del_res.status_code == 204

    # 4. 再次查詢確認已不存在
    not_found_res = client.get(f"/api/v1/memos/{memo_id}")
    assert not_found_res.status_code == 404
```

---

## 驗收條件 (Acceptance Criteria)

1. 啟動伺服器後，能於 Swagger UI (`http://localhost:8000/docs`) 正常操作 `POST`, `GET /{id}`, `DELETE /{id}` 端點。
2. 開啟 **TablePlus**（`Ctrl+R` / `Cmd+R` 重新整理），能看到 `memos` 資料表中確實有寫入對應的資料列與欄位值。
3. 執行 `pytest tests/test_memos_basic.py`，所有測試通過並顯示綠燈（All Passed）。
