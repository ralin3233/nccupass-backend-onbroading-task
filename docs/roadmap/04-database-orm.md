# 04. MySQL 資料庫、SQL 基礎與 SQLAlchemy 2.0 ORM

在後端系統中，資料庫負責資料的持久化儲存。我們使用 **SQLAlchemy 2.0** 將 Python 物件映射為 MySQL 中的資料表，但在開發與除錯時，理解背後的 **原生 SQL 語法** 與如何使用 **GUI 工具（如 TablePlus）** 對照檢查是後端工程師的必備基本功。

---

## 1. 什麼是 ORM (Object-Relational Mapping)？

不用每次手寫容易出錯且容易引發 SQL Injection 的字串（如 `INSERT INTO memos ...`），ORM 讓我們可以直接使用物件導向的 Python Class 來操作資料表：

```mermaid
graph LR
    Py["Python 物件<br/>memo = Memo(title='開會')"] -->|SQLAlchemy ORM 轉換| SQL["標準 SQL 語句<br/>INSERT INTO memos VALUES(...)"]
    SQL --> DB[("MySQL 資料庫")]
```

---

## 2. 常用 SQL 基礎語法與 ORM 對照指南

當你在寫 ORM 遇到 Bug、或想在 TablePlus 的 SQL 編輯器中驗證資料時，請參考以下常用的 SQL vs SQLAlchemy 2.0 對照表：

### (1) 查詢資料 (SELECT)

| 操作目標 | 原生 SQL 語句 | SQLAlchemy ORM 語法 |
| :--- | :--- | :--- |
| **查詢所有備忘錄** | `SELECT * FROM memos;` | `db.query(Memo).all()` |
| **依 ID 查詢單筆** | `SELECT * FROM memos WHERE id = 1;` | `db.query(Memo).filter(Memo.id == 1).first()` |
| **條件篩選 (未完成)** | `SELECT * FROM memos WHERE is_completed = 0;` | `db.query(Memo).filter(Memo.is_completed == False).all()` |
| **關鍵字模糊搜尋** | `SELECT * FROM memos WHERE title LIKE '%作業%';` | `db.query(Memo).filter(Memo.title.contains("作業")).all()` |
| **排序 (最新優先)** | `SELECT * FROM memos ORDER BY created_at DESC;` | `db.query(Memo).order_by(Memo.created_at.desc()).all()` |
| **分頁 (跳過前 10 筆，取 5 筆)** | `SELECT * FROM memos LIMIT 5 OFFSET 10;` | `db.query(Memo).offset(10).limit(5).all()` |
| **計算總筆數** | `SELECT COUNT(*) FROM memos;` | `db.query(Memo).count()` |

---

### (2) 新增資料 (INSERT)

- **原生 SQL**：
  ```sql
  INSERT INTO memos (title, content, priority, is_completed, created_at, updated_at)
  VALUES ('期末專案討論', '後端架構與資料庫設計', 3, 0, NOW(), NOW());
  ```
- **SQLAlchemy ORM**：
  ```python
  new_memo = Memo(title="期末專案討論", content="後端架構與資料庫設計", priority=3)
  db.add(new_memo)
  db.commit()
  db.refresh(new_memo)  # 取得資料庫自動生成的自增 ID 與預設時間
  ```

---

### (3) 更新資料 (UPDATE)

- **原生 SQL**：
  ```sql
  UPDATE memos 
  SET is_completed = 1, updated_at = NOW() 
  WHERE id = 5;
  ```
- **SQLAlchemy ORM**：
  ```python
  memo = db.query(Memo).filter(Memo.id == 5).first()
  if memo:
      memo.is_completed = True
      db.commit()
      db.refresh(memo)
  ```

---

### (4) 刪除資料 (DELETE)

- **原生 SQL**：
  ```sql
  DELETE FROM memos WHERE id = 5;
  ```
- **SQLAlchemy ORM**：
  ```python
  memo = db.query(Memo).filter(Memo.id == 5).first()
  if memo:
      db.delete(memo)
      db.commit()
  ```

---

## 3. 在 TablePlus 中使用 SQL 快速對照除錯

當你在開發 API 遇到「為什麼畫面沒更新？」或「為什麼 Swagger 查出來是空列表？」時，善用 TablePlus 進行即時驗證：

1. **開啟 SQL 編輯視窗**：在 TablePlus 視窗按下 `Ctrl + E` (Windows) 或 `Cmd + E` (macOS)。
2. **輸入並執行 SQL**：
   ```sql
   -- 1. 查看資料庫裡到底有幾筆備忘錄
   SELECT id, title, is_completed, priority, created_at FROM memos;

   -- 2. 測試你的搜尋條件是否真能命中資料
   SELECT * FROM memos WHERE title LIKE '%專案%' AND is_completed = 0;
   ```
3. **執行選定區塊**：反白該段 SQL，按下 `Ctrl + Enter` (Windows) 或 `Cmd + Enter` (macOS) 立即查看底層資料表結果。
4. **檢查欄位型態與預設值**：點擊右側 `Structure` 分頁，確認 `is_completed` 是否為 `TINYINT(1)`、`id` 是否有 `AUTO_INCREMENT`。

```mermaid
graph TD
    A["API 測試回傳非預期結果"] --> B["開啟 TablePlus (Cmd+E / Ctrl+E)"]
    B --> C["執行 SELECT * FROM memos;"]
    C --> D{"資料庫是否有資料？"}
    D -->|否| E["檢查 POST API 或 db.commit() 是否遺漏"]
    D -->|是| F["檢查 GET API 的過濾條件 (WHERE / filter) 是否過於嚴格"]
```

---

## 4. 連線設定與 Session 管理 (`database.py`)

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker

# 連線字串格式: mysql+pymysql://<user>:<password>@<host>:<port>/<dbname>
SQLALCHEMY_DATABASE_URL = "mysql+pymysql://nccu_user:nccu_password@localhost:3306/nccu_memo_db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    pool_pre_ping=True,      # 自動檢查連線是否活著，避免中斷
    pool_recycle=3600        # 每隔一小時重新建立連線池
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# FastAPI 專用依賴注入函式：自動建立並在請求結束後安全關閉 Session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 5. 定義 Model 模型 (`models/memo.py`)

```python
from datetime import datetime, timezone
from sqlalchemy import Boolean, Column, DateTime, Integer, String, Text
from database import Base

class Memo(Base):
    __tablename__ = "memos"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    title = Column(String(100), nullable=False, index=True)
    content = Column(Text, nullable=True)
    is_completed = Column(Boolean, default=False, nullable=False)
    priority = Column(Integer, default=1, nullable=False)
    created_at = Column(DateTime, default=lambda: datetime.now(timezone.utc), nullable=False)
    updated_at = Column(
        DateTime,
        default=lambda: datetime.now(timezone.utc),
        onupdate=lambda: datetime.now(timezone.utc),
        nullable=False,
    )
```

---

## 6. 什麼是 Alembic 資料庫遷移（Migration）？

當你需要「在已經有資料的資料表新增一個 `due_date` 欄位」時，不能直接刪掉資料表重做。  
**Alembic** 就像資料庫的 Git，能為每次資料庫綱要（Schema）的變更建立版本記錄（例如 `alembic revision --autogenerate -m "add due_date to memos"`），並安全升級或回滾資料庫結構。

---

## 外部推薦優質資源
- [TablePlus 官方文檔與快捷鍵指南](https://docs.tableplus.com/)
- [SQL 語法基礎入門 (W3Schools 中文/英文)](https://www.w3schools.com/sql/)
- [SQLAlchemy 2.0 官方文件 (英文)](https://docs.sqlalchemy.org/en/20/)
- [FastAPI 官方文檔 - SQL (Relational) Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)
