# Onboarding Task：政大通備忘錄系統 (Memo Service)

歡迎來到後端組的實戰演練任務！這個任務被設計成一個聚焦於 API 開發、業務邏輯與測試的實戰專案。

---

## 專案背景與需求場景

政大通 App 預計推出新功能「政大個人備忘錄 / 代辦事項」，讓政大學生可以記錄隨手靈感、待繳作業或重要校園日程。

為了讓大家能直接專注在 **API 設計**、**Pydantic 資料轉換**、**業務邏輯處理** 與 **自動化測試**，專案的**資料庫模型（SQLAlchemy Model）與連線設定（Database Engine / Session）已經由團隊預先配置好**。你的任務是接續完成各項 RESTful API 端點與對應的 CRUD 存取邏輯。

```mermaid
graph TD
    subgraph Client ["客戶端"]
        Web["Web / App 前端"]
        Swagger["Swagger UI /docs"]
        TestClient["pytest 自動化測試"]
    end

    subgraph Backend ["你的實作重點 (FastAPI)"]
        Router["API Routers 路由層<br/>(端點路由、請求接收與回應)"]
        Schema["Pydantic Schemas 驗證層<br/>(Request & Response 格式定義)"]
        Service["CRUD / Data Access 邏輯層<br/>(分頁、過濾、ORM 操作)"]
    end

    subgraph Template ["已預先配置完成"]
        Model[("SQLAlchemy Memo Model")]
        DB[("Database Engine & Session")]
    end

    Client -->|HTTP / JSON| Router
    Router --> Schema
    Router --> Service
    Service --> Model
    Model --> DB
```

---

## 標準分層結構概覽

```text
nccupass-memo-service/
├── app/
│   ├── api/                     # 1. 路由層 (Controllers / Presentation Layer)
│   │   ├── deps.py              # 共用依賴注入 (get_db, get_current_user)
│   │   └── v1/
│   │       ├── api.py           # v1 總路由整合
│   │       └── endpoints/
│   │           ├── auth.py      # 認證相關端點
│   │           └── memos.py     # 備忘錄相關端點
│   ├── core/                    # 2. 核心設定層 (Core Config & Security)
│   │   ├── config.py            # 環境變數載入 (Pydantic Settings)
│   │   ├── security.py          # 密碼雜湊 (bcrypt) 與 JWT 生成/解析
│   │   └── exceptions.py        # 自訂例外類別
│   ├── crud/                    # 3. 資料存取層 (Data Access Layer / Repository)
│   │   ├── crud_user.py         # 使用者資料庫操作 (Query / Insert / Update)
│   │   └── crud_memo.py         # 備忘錄資料庫操作
│   ├── models/                  # 4. ORM 模型層 (Database Models)
│   │   ├── base.py              # SQLAlchemy Base
│   │   ├── user.py              # User Table ORM Class
│   │   └── memo.py              # Memo Table ORM Class
│   ├── schemas/                 # 5. 資料傳輸與驗證層 (DTO / Pydantic Schemas)
│   │   ├── common.py            # 統一 API 回應格式 (ResponseWrapper)
│   │   ├── user.py              # UserCreate, UserResponse
│   │   └── memo.py              # MemoCreate, MemoUpdate, MemoResponse
│   ├── database.py              # 資料庫 Engine & SessionLocal
│   └── main.py                  # FastAPI 主程式入口與 Middleware 設定
├── tests/                       # 6. 自動化測試層 (Automated Tests)
│   ├── conftest.py              # 測試 fixtures
│   ├── test_auth.py
│   └── test_memos.py
├── alembic/                     # 資料庫遷移腳本
├── .env.example
├── requirements.txt
└── README.md
```

---

## 敏捷 Sprint 里程碑規劃

我們將任務拆解為 2 個循序漸進的 Sprint：

| Sprint | 主題名稱 | 核心學習與交付成果 |
| :--- | :--- | :--- |
| **[Sprint 1](sprint-1.md)** | **基礎 CRUD API 與 Schemas 實作** | 定義基礎 Pydantic 模型、實作建立/單筆查詢/刪除端點、撰寫基礎單元測試 |
| **[Sprint 2](sprint-2.md)** | **列表過濾、進階業務邏輯與完整測試** | 實作分頁列表、完成狀態篩選、關鍵字搜尋、部分更新（PATCH）與完整測試 |

---

## 任務繳交與驗收方式

1. **Clone Starter 練習專案**：
   從團隊提供的 Starter 範本 Repository 開始開發：
   ```bash
   git clone https://gitlab.com/nccupass/nccupass-memo-service-for-beginner.git
   cd nccupass-memo-service-for-beginner
   ```
   > 練習用 Repo 連結：[nccupass-memo-service-for-beginner (GitLab)](https://gitlab.com/nccupass/nccupass-memo-service-for-beginner.git)

2. **遵守分支與 Commit 規範**：依據 Sprint 開發分支（例如 `feat/sprint-1-basic-api`、`feat/sprint-2-list-and-update`）。
3. **完成後發起 MR**：發起 Merge Request (MR) 並 Tag 你的 Mentor，在 MR 描述中附上 `pytest` 綠燈截圖或 Swagger `/docs` 測試結果。
4. **Code Review**：Mentor 會給予程式碼架構與優化建議，討論並修改完成後即算過關！確認後會將你的分支合併到 `main`。
5. **進度匯報**：每個 Sprint 結束後會開會請各位講解這個 Sprint 做了什麼並 DEMO。
