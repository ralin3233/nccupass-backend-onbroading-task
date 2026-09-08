# 02. Python 現代特性與 Pydantic

現代 Python（3.10+）引入了強大的**型別提示（Type Hints）**，而 FastAPI 的強大核心正是建立在 **Pydantic** 與 Type Hints 之上。

---

## 1. Type Hints（型別提示）

在過去 Python 是純動態型別語言，而在大型專案中，標註型別能讓 VS Code 提供超強自動補全，並在執行前抓出 80% 的愚蠢 Bug。

```python
# 傳統寫法 (沒有提示，難以知道引數該傳什麼)
def create_user(name, age, tags):
    return {"name": name, "age": age}

# 現代 Type Hints 寫法
def create_user(name: str, age: int, tags: list[str] | None = None) -> dict[str, str | int]:
    return {"name": name, "age": age}
```

常見型別：
- 基本：`str`, `int`, `float`, `bool`
- 容器：`list[str]`, `dict[str, Any]`, `set[int]`
- 聯集：`int | None`（代表可以是整數，也可以是 None）

---

## 2. Pydantic 核心概念

Pydantic 是 Python 中最主流的**資料驗證與轉換套件**。

當前端傳來 JSON 時，Pydantic 會自動完成：
1. **型別檢查**（例如前端傳 `"123"` 字串，但 schema 宣告 `int`，Pydantic 會自動轉換成整數 `123`）。
2. **格式約束驗證**（字串長度、數值範圍、Email 格式）。
3. **序列化**（將物件轉成乾淨的 Python dict 或 JSON）。

### 範例：定義一個 Memo 建立 Schema

```python
from datetime import datetime
from pydantic import BaseModel, Field

class MemoCreateRequest(BaseModel):
    title: str = Field(..., min_length=1, max_length=100, description="備忘錄標題")
    content: str | None = Field(None, max_length=1000, description="備忘錄詳細內文")
    priority: int = Field(default=1, ge=1, le=5, description="優先權: 1(最低) ~ 5(最高)")
    is_completed: bool = False

# 測試資料驗證
data = {
    "title": "繳交 NCCUpass 專案作業",
    "content": "記得推送到 GitHub",
    "priority": 5
}

memo = MemoCreateRequest(**data)
print(memo.title)        # "繳交 NCCUpass 專案作業"
print(memo.is_completed) # False (自動帶入預設值)
```

若前端傳送 `priority: 10`，Pydantic 會直接拋出驗證錯誤，FastAPI 會自動轉譯成漂亮的 `422 Unprocessable Entity` 回應！

---

## 3. Async / Await 非同步初探

FastAPI 天生支援非同步處理，讓單一伺服器可以在等待資料庫讀寫時，同時處理成千上萬個使用者的請求：

```python
import asyncio

# 定義非同步函式
async def fetch_student_data(student_id: str) -> dict:
    # 模擬等待資料庫回應 1 秒
    await asyncio.sleep(1)
    return {"student_id": student_id, "name": "政大小明"}
```

---

## 外部推薦優質資源 (施工中)
- [Pydantic 官方文件 (英文)](https://docs.pydantic.dev/latest/)
- [FastAPI 官方文檔 - Python 型別介紹 (繁中/簡中)](https://fastapi.tiangolo.com/zh/python-types/)
- [Python Async/Await 快速入門 (YouTube)](https://www.youtube.com/results?search_query=python+async+await+tutorial)
