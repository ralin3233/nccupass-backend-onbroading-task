# 06. 後端專案架構與根目錄檔案解析

當你第一次 Clone 一個後端專案時，看到根目錄下一堆 `.gitignore`、`.env.example`、`requirements.txt` 以及 `app/` 下各種資料夾，是否曾覺得眼花撩亂？

本篇將帶你逐一解構現代 Python / FastAPI 後端專案的 **根目錄檔案職責** 與 **分層架構（Layered Architecture）** ，幫助你迅速掌握「哪份程式碼該放哪裡」與「一個請求是如何在專案中流轉的」。

---

## 1. 專案完整目錄樹狀圖

以政大通 Onboarding Task 的 `nccupass-memo-service` 為例，標準的專案目錄結構如下：

```text
nccupass-memo-service/
├── .env.example             # 環境變數設定範本（提供給開發者複製）
├── .gitignore               # Git 版本控制忽略清單
├── README.md                # 專案說明文件、安裝啟動指引
├── requirements.txt         # Python 專案依賴套件清單
├── alembic.ini              # Alembic 資料庫遷移設定檔
├── alembic/                 # 資料庫遷移（Migration）版本腳本目錄
│   ├── env.py
│   └── versions/            # 每次 migration 產生的版本紀錄
├── app/                     # 後端主應用程式原始碼
│   ├── main.py              # FastAPI 程式入口點與中介軟體配置
│   ├── database.py          # 資料庫連線引擎與 Session 配置
│   ├── api/                 # 1. 路由層（API Routers / Endpoints）
│   │   ├── deps.py          # 依賴注入共用邏輯（如 get_db, get_current_user）
│   │   └── v1/
│   │       ├── api.py       # v1 API 總路由匯整
│   │       └── endpoints/   # 各業務模組端點（memos.py, auth.py）
│   ├── core/                # 2. 核心設定層（Config & Security）
│   │   ├── config.py        # 環境變數讀取與全域設定 (Pydantic Settings)
│   │   ├── security.py      # 密碼雜湊 (bcrypt) 與 JWT Token 加解密
│   │   └── exceptions.py    # 自訂例外類別與錯誤處理
│   ├── schemas/             # 3. 資料驗證與轉換層（Pydantic Schemas / DTO）
│   │   ├── common.py        # 通用回應格式（如 ResponseWrapper）
│   │   ├── memo.py          # Memo 請求/回應結構定義
│   │   └── user.py          # User 請求/回應結構定義
│   ├── crud/                # 4. 資料存取層（CRUD Operations / Repository）
│   │   ├── crud_memo.py     # Memo 資料庫查詢/新增/更新邏輯
│   │   └── crud_user.py     # User 資料庫操作邏輯
│   └── models/              # 5. ORM 資料表模型層（SQLAlchemy Models）
│       ├── base.py          # SQLAlchemy Declarative Base
│       ├── memo.py          # Memo 資料表映射類別
│       └── user.py          # User 資料表映射類別
└── tests/                   # 6. 自動化測試目錄
    ├── conftest.py          # pytest 全域 fixtures（測試資料庫、客戶端）
    ├── test_auth.py         # 認證相關端點測試
    └── test_memos.py        # 備忘錄相關端點測試
```

---

## 2. 根目錄檔案詳解（它們在做什麼？）

根目錄的檔案通常負責 **專案環境配置、套件相依性、版本控制規則與維運設定** 。

### ① `.env` 與 `.env.example`（環境變數設定）
- **`.env.example`**：公開的設定範本。裡面列出專案執行所需的環境變數名稱與預設值（例如 `DATABASE_URL=mysql+pymysql://user:password@localhost:3306/memo_db`、`SECRET_KEY=your-secret-key`）。這個檔案**會**提交到 Git。
- **`.env`**：你本機的實際設定檔（包含真實密碼、金鑰）。
- **關鍵守則**：
    > [!CAUTION]
    > **`.env` 絕對不能 Commit 到 Git！**  
    > 裡面包含資料庫密碼與 JWT 密鑰，一旦推上 GitHub 可能會導致資安外洩。`.env` 必須被加入 `.gitignore` 中。

---

### ② `.gitignore`（版控忽略規則）
告訴 Git 哪些檔案或資料夾**不要**納入版本控制：
- 虛擬環境：`.venv/`, `venv/`, `env/`
- Python 快取檔案：`__pycache__/`, `*.pyc`
- 機密檔案：`.env`, `*.pem`, `*.key`
- 編輯器/系統產生的設定：`.vscode/`, `.idea/`, `.DS_Store`
- 測試產生物：`.pytest_cache/`, `htmlcov/`, `.coverage`

---

### ③ `requirements.txt`（依賴套件清單）
記錄本專案所依賴的第三方 Python 套件與版本號，例如：
```text
fastapi>=0.110.0
uvicorn[standard]>=0.28.0
sqlalchemy>=2.0.0
pymysql>=1.1.0
pydantic-settings>=2.2.0
pytest>=8.0.0
```
- 開發者只需執行 `pip install -r requirements.txt` 即可一鍵安裝所有相依套件。

---

### ④ `README.md`（專案使用說明）
專案的門面，主要內容包含：
1. 專案功能與特色簡介
2. 本機開發環境啟動步驟（如何建立虛擬環境、設定 `.env`、啟動 FastAPI 服務）
3. 測試執行方式（`pytest`）與 API 文件連結（Swagger `/docs`）

---

### ⑤ `alembic.ini` 與 `alembic/`（資料庫版本遷移）
當資料庫結構變更時（例如 Memo 新增 `priority` 欄位），Alembic 負責記錄資料表結構的演進歷史：
- **`alembic.ini`**：Alembic 的主設定檔（設定資料庫連線路徑與遷移腳本位置）。
- **`alembic/versions/`**：每次執行 `alembic revision --autogenerate` 所生成的遷移腳本檔案，確保團隊每位成員的資料庫結構保持同步。

---

### ⑥ 其他常見根目錄設定檔（補充）
- **`Dockerfile` / `docker-compose.yml`**：將後端服務與 MySQL 資料庫容器化，方便在任何機器上一鍵打包與部署。
- **`pytest.ini`**：pytest 測試框架的設定檔（例如指定測試搜尋路徑、預設參數等）。
- **`mkdocs.yml`**：本新人手冊專案所使用的 MkDocs 文件網站配置檔。

---

## 3. `app/` 核心分層架構詳解（Layered Architecture）

我們遵循 **關注點分離（Separation of Concerns）** 原則，將後端程式碼拆分為各司其職的層級：

```mermaid
graph TD
    Client["客戶端 (Web / App / Postman)"] -->|1. HTTP 請求| Main["app/main.py<br/>程式入口與中介層"]
    Main -->|2. 路由分派| Router["app/api/v1/endpoints/<br/>路由層：接收請求與參數"]
    
    Deps["app/api/deps.py<br/>依賴注入：取得 DB Session 與驗證 Token"]
    Config["app/core/<br/>全域設定與安全金鑰"]

    Router -.->|注入 Session 與 User| Deps
    Router -.->|讀取設定| Config
    
    Router -->|3. 資料驗證與轉換| Schema["app/schemas/<br/>Pydantic 驗證 Request Body"]
    Router -->|4. 呼叫資料庫邏輯| CRUD["app/crud/<br/>資料存取層：執行 Query 或 Insert"]
    
    CRUD -->|5. ORM 映射| Model["app/models/<br/>SQLAlchemy 資料表模型"]
    Model -->|6. SQL 查詢| DB[("MySQL 資料庫")]
    
    DB -->|7. 原始資料列| Model
    Model -->|8. ORM 物件| CRUD
    CRUD -->|9. 資料回傳| Router
    Router -->|10. 序列化成 JSON| Schema
    Schema -->|11. HTTP 回應 200 或 201| Client
```

---

### 各層職責與關鍵守則

| 層級目錄 | 中文名稱 | 主要職責 | 關鍵規範 |
| :--- | :--- | :--- | :--- |
| **`app/main.py`** | 程式進入點 | 建立 `FastAPI()` 實例、掛載 CORS / Middleware、註冊 API 總路由、設定啟動與關閉生命週期事件。 | 保持乾淨，不在此撰寫具體業務 API 端點。 |
| **`app/database.py`** | 資料庫核心 | 建立 SQLAlchemy `Engine`、`SessionLocal` 工廠函式，提供連線池基礎設施。 | 僅負責建立資料庫連線實例。 |
| **`app/core/`** | 核心設定與安全 | 讀取 `.env` 環境變數（`config.py`）、密碼雜湊與 JWT 加解密（`security.py`）、全域自訂 Exception（`exceptions.py`）。 | 不依賴 API 路由層，提供純工具與設定。 |
| **`app/api/`** | 路由與控制層 | 定義 RESTful 端點路徑、HTTP Method、接收 Path/Query/Body 參數、呼叫對應的 CRUD 函式並決定回傳 HTTP 狀態碼。 | **禁止直接寫 SQL 查詢**，一律呼叫 CRUD 模組。 |
| **`app/api/deps.py`** | 依賴注入 | 提供 `get_db()` 生成器（確保每次請求結束自動關閉 DB Session）與 `get_current_user()`（解析 JWT Token）。 | 所有需要 DB 連線或身分驗證的 Endpoint 都透過 `Depends()` 注入。 |
| **`app/schemas/`** | 資料契約層 (DTO) | 使用 Pydantic 定義請求與回應的資料規格（如 `MemoCreate`, `MemoResponse`），負責資料驗證、過濾敏感欄位與格式轉換。 | 區分 Create、Update 與 Response，避免敏感資料（如密碼）外洩。 |
| **`app/crud/`** | 資料存取層 (Repository) | 封裝所有與資料庫溝通的 SQLAlchemy ORM 操作（如 `db.query()`, `db.add()`, `db.commit()`）。 | **不碰 HTTP 概念**（不拋出 `HTTPException`，無 Request/Response 概念）。 |
| **`app/models/`** | ORM 模型層 | 定義與 MySQL 資料表對應的 SQLAlchemy 類別（定義欄位名稱、型別、主鍵、外鍵關聯 `relationship` 等）。 | 僅描述資料庫 Schema，不含業務邏輯。 |
| **`tests/`** | 自動化測試 | 撰寫單元測試與 API 整合測試（使用 pytest + TestClient），驗證端點正確性。 | 測試應使用獨立的測試資料庫或 SQLite，不污染開發資料。 |

---

## 4. 為什麼 Model 與 Schema 要分開？（新手常見疑惑）

很多剛接觸 FastAPI 的同學會問：「**既然 Model 和 Schema 欄位長得差不多，為什麼要寫兩次？**」

```mermaid
graph LR
    subgraph Schemas["Pydantic Schemas (API 介面契約)"]
        Req["MemoCreate<br/>- title: 必填<br/>- content: 選填"]
        Res["MemoResponse<br/>- id: 系統自動產生<br/>- title: 標題<br/>- created_at: 建立時間"]
    end

    subgraph Models["SQLAlchemy Model (資料庫內部結構)"]
        DBModel["Memo 資料表<br/>- id: 主鍵 Auto-increment<br/>- title: VARCHAR<br/>- content: TEXT<br/>- hashed_token: 內部機密欄位<br/>- created_at: TIMESTAMP"]
    end

    Req -->|API 輸入驗證寫入| DBModel
    DBModel -->|API 輸出過濾與隱藏敏感欄位| Res
```

1. **職責不同**：
    - **`models/`**：代表「**資料庫存什麼**」，定義資料表欄位型態、主鍵、外鍵關聯、索引等底層設定。
    - **`schemas/`**：代表「**API 想接收什麼、回傳什麼**」，負責資料型別驗證、預設值、必填欄位檢查與序列化。
2. **安全性與資料過濾**：
    - 使用者資料表有 `hashed_password` 欄位（Model 必須儲存）。
    - 但回傳給前端的 `UserResponse`（Schema）**絕對不能**包含該欄位。
3. **靈活性**：
    - `MemoCreate` 不需要 `id` 與 `created_at`（由資料庫自動產生）。
    - `MemoUpdate` 的欄位通常都是選填（Optional）。
    - `MemoResponse` 則需要包含完整的 `id` 與時間戳記。

---

## 5. 新手開發小結：當我要加一個新功能時，該怎麼走？

假設今天我們要為系統新增一個「標籤功能（Tag）」：

1. **`models/tag.py`**：定義 `Tag` 的 SQLAlchemy ORM Model。
2. **`schemas/tag.py`**：定義 `TagCreate`、`TagResponse` 等 Pydantic Schemas。
3. **`crud/crud_tag.py`**：撰寫 `get_tag`、`create_tag` 等資料庫查詢/新增函式。
4. **`api/v1/endpoints/tags.py`**：建立路由端點，使用 `Depends(get_db)` 注入 Session，呼叫 `crud_tag` 並回傳資料。
5. **`api/v1/api.py`**：將 `tags.router` 註冊到總路由中。
6. **`tests/test_tags.py`**：撰寫 pytest 測試案例確認功能正常！
