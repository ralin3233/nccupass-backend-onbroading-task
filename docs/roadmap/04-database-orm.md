# 04. MySQL 資料庫與 SQLAlchemy 2.0 ORM

在後端系統中，資料庫負責資料的持久化儲存。我們使用 **SQLAlchemy 2.0** 將 Python 物件映射為 MySQL 中的資料表。

---

## 1. 什麼是 ORM (Object-Relational Mapping)？

不用手寫繁瑣容易出錯的 SQL 字串（如 `INSERT INTO memos ...`），ORM 讓我們可以直接用 Python Class 操作資料表：

```mermaid
graph LR
    Py["Python 物件<br/>memo = Memo(title='開會')"] -->|SQLAlchemy ORM 轉換| SQL["標準 SQL 語句<br/>INSERT INTO memos VALUES(...)"]
    SQL --> DB[("MySQL 資料庫")]
```

---

## 2. 連線設定與 Session 管理 (`database.py`)

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

# MySQL 連線字串格式: mysql+pymysql://<user>:<password>@<host>:<port>/<dbname>
SQLALCHEMY_DATABASE_URL = "mysql+pymysql://nccu_user:nccu_password@localhost:3306/nccu_memo_db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    pool_pre_ping=True,      # 自動檢查連線是否活著，避免斷線
    pool_recycle=3600        # 每隔一小時重新建立連線 pool
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# FastAPI 專用依賴注入函式：自動建立並在請求結束後關閉 Session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 3. 定義 Model 模型 (`models/memo.py`)

```python
from datetime import datetime
from sqlalchemy import Column, Integer, String, Text, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from database import Base

class Memo(Base):
    __tablename__ = "memos"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    title = Column(String(100), nullable=False, index=True)
    content = Column(Text, nullable=True)
    is_completed = Column(Boolean, default=False, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

---

## 4. 基礎 CRUD 操作範例

```python
from sqlalchemy.orm import Session
from models.memo import Memo

# 1. Create (新增)
def create_memo(db: Session, title: str, content: str) -> Memo:
    db_memo = Memo(title=title, content=content)
    db.add(db_memo)
    db.commit()        # 送交交易
    db.refresh(db_memo) # 取得自增 ID 與預設欄位
    return db_memo

# 2. Read (查詢多筆與單筆)
def get_memos(db: Session, skip: int = 0, limit: int = 20) -> list[Memo]:
    return db.query(Memo).offset(skip).limit(limit).all()

def get_memo_by_id(db: Session, memo_id: int) -> Memo | None:
    return db.query(Memo).filter(Memo.id == memo_id).first()

# 3. Update (更新)
def update_memo_status(db: Session, memo_id: int, is_completed: bool) -> Memo | None:
    db_memo = get_memo_by_id(db, memo_id)
    if db_memo:
        db_memo.is_completed = is_completed
        db.commit()
        db.refresh(db_memo)
    return db_memo

# 4. Delete (刪除)
def delete_memo(db: Session, memo_id: int) -> bool:
    db_memo = get_memo_by_id(db, memo_id)
    if db_memo:
        db.delete(db_memo)
        db.commit()
        return True
    return False
```

---

## 5. 什麼是 Alembic 資料庫遷移（Migration）？

當你需要「在已經有資料的資料表新增一個 `priority` 欄位」時，不能直接刪掉資料表重做。  
**Alembic** 就像資料庫的 Git，能為每次資料庫綱要（Schema）的變更建立版本記錄（例如 `alembic revision --autogenerate -m "add priority to memos"`），並安全升級或回滾資料庫結構。

---

## 外部推薦優質資源 (施工中)
- [SQLAlchemy 2.0 官方文件 (英文)](https://docs.sqlalchemy.org/en/20/)
- [FastAPI 官方文檔 - SQL (Relational) Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)
- [Alembic 官方教學 (英文)](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
