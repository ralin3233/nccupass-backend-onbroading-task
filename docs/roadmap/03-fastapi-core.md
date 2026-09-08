# 03. FastAPI 核心觀念

**FastAPI** 是目前 Python 世界最受歡迎、執行速度最快且開發體驗極佳的 Web 框架之一。

---

## 1. 最小可執行範例 (Hello NCCUpass)

建立一個檔案 `main.py`：

```python
from fastapi import FastAPI

app = FastAPI(
    title="政大通備忘錄 API",
    description="NCCUpass Backend Onboarding API",
    version="1.0.0"
)

@app.get("/")
def read_root():
    return {"message": "Welcome to NCCUpass Backend API!"}

@app.get("/health")
def health_check():
    return {"status": "ok", "service": "memo-api"}
```

啟動服務：
```bash
uvicorn main:app --reload
```
- 開啟瀏覽器訪問 `http://127.0.0.1:8000/docs`，你將看到**全自動生成的 Swagger UI 互動式 API 規格書**。
- 訪問 `http://127.0.0.1:8000/redoc` 則有另一款優雅的 ReDoc 介面。

---

## 2. 請求參數三劍客：Path, Query, Body

FastAPI 根據參數宣告的方式自動區分資料來源：

```python
from fastapi import FastAPI, Query, Path
from pydantic import BaseModel

app = FastAPI()

class MemoCreate(BaseModel):
    title: str
    content: str | None = None

# 1. Path Parameter (路徑參數，例如 /memos/42)
@app.get("/memos/{memo_id}")
def get_memo(memo_id: int = Path(..., description="備忘錄唯一識別碼", ge=1)):
    return {"memo_id": memo_id, "title": "查詢到的備忘錄"}

# 2. Query Parameter (查詢參數，例如 /memos?limit=10&is_completed=true)
@app.get("/memos")
def list_memos(
    limit: int = Query(default=20, ge=1, le=100),
    is_completed: bool | None = Query(default=None)
):
    return {"limit": limit, "filter_completed": is_completed, "data": []}

# 3. Request Body (請求本體，JSON Payload)
@app.post("/memos", status_code=201)
def create_memo(payload: MemoCreate):
    return {"id": 101, **payload.model_dump()}
```

---

## 3. 依賴注入系統（Dependency Injection / `Depends`）

這是 FastAPI 最強大、也是大型專案架構最核心的機制。它常用於：
1. **取得資料庫 Session**（確保每次請求結束自動關閉連線）
2. **驗證使用者 Token**（檢查當前請求的使用者是誰）
3. **共用權限驗證邏輯**

```python
from fastapi import Depends, HTTPException, status

# 定義一個共用依賴函式
def get_current_user_token(authorization: str | None = None):
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="無效或缺失的認證憑證"
        )
    token = authorization.split(" ")[1]
    return {"user_id": 1, "username": "nccu_student", "token": token}

# 在 API 端點中透過 Depends 注入使用
@app.get("/me/memos")
def get_my_memos(current_user: dict = Depends(get_current_user_token)):
    return {
        "user": current_user["username"],
        "memos": ["微積分作業", "政大通後端例會"]
    }
```

---

## 外部推薦優質資源 (施工中)
- [FastAPI 官方完整教學 (繁中/簡中)](https://fastapi.tiangolo.com/zh/)
- [FastAPI Course for Beginners (freeCodeCamp YouTube)](https://www.youtube.com/watch?v=tLKKmouUAMS)
- [Tiangolo (FastAPI 作者) 經典教學文章](https://tiangolo.com/)
