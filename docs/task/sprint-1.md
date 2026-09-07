# Sprint 1 - 環境與資料模型設計

> **Sprint 目標**：建立乾淨的專案目錄架構、設定環境變數、連線至 MySQL 資料庫，並透過 SQLAlchemy 定義 `Memo` 資料表模型。

---

## 任務清單 (Task Checklist)

- [ ] 建立專案目錄與 Python 虛擬環境 (`.venv`)
- [ ] 安裝核心依賴：`fastapi`, `uvicorn[standard]`, `sqlalchemy`, `pymysql`, `python-dotenv`, `pydantic-settings`
- [ ] 建立 `.env` 與 `.env.example`，避免敏感密碼進 Git
- [ ] 撰寫 `database.py` 建立 MySQL 連線池與 `Base`
- [ ] 撰寫 `models/memo.py` 定義 Memo ORM Model
- [ ] 撰寫 `main.py` 並實作 `/health` 端點測試資料庫連線
- [ ] 啟動專案，確認 Swagger UI (`/docs`) 正常運行且 MySQL 資料表成功建立

---

## 實作指引

### 1. 專案目錄結構

建議初始化目錄結構如下：
```text
nccupass-memo-service/
├── app/
│   ├── core/
│   │   └── config.py        # 讀取 .env 設定
│   ├── models/
│   │   └── memo.py          # SQLAlchemy ORM Model
│   ├── schemas/
│   │   └── memo.py          # Pydantic Schemas (Sprint 2 擴充)
│   ├── database.py          # 資料庫連線引擎
│   └── main.py              # FastAPI 進入點
├── .env                     # 本地連線機密 (加入 .gitignore)
├── .env.example             # 設定範本
├── .gitignore
└── requirements.txt
```

### 2. 環境變數設定 (`app/core/config.py`)

使用 `pydantic-settings` 優雅管理環境變數：

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    PROJECT_NAME: str = "NCCUpass Memo Service"
    DB_HOST: str = "localhost"
    DB_PORT: int = 3306
    DB_USER: str = "nccu_user"
    DB_PASSWORD: str = "nccu_password"
    DB_NAME: str = "nccu_memo_db"

    @property
    def DATABASE_URL(self) -> str:
        return f"mysql+pymysql://{self.DB_USER}:{self.DB_PASSWORD}@{self.DB_HOST}:{self.DB_PORT}/{self.DB_NAME}"

    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

settings = Settings()
```

### 3. Memo 資料表規格需求

`models/memo.py` 需包含以下欄位：
- `id`：整數主鍵（Auto Increment）
- `title`：字串，最長 100 字，必填，建立索引
- `content`：長文字，選填
- `is_completed`：布林值，預設為 `False`
- `priority`：整數 (1~5)，預設為 1
- `created_at`：時間戳記，預設為當前 UTC 時間
- `updated_at`：時間戳記，更新時自動帶入當前 UTC 時間

---

## 驗收條件 (Acceptance Criteria)

1. 執行 `uvicorn app.main:app --reload` 能夠正常無報錯啟動。
2. 訪問 `GET http://localhost:8000/health` 能回傳 `{"status": "healthy", "database": "connected"}`。
3. 開啟 DBeaver 查看 MySQL 資料庫，確認已成功生成 `memos` 資料表與對應欄位。
