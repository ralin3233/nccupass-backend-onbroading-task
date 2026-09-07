# 政大通 NCCUpass | 後端組新人手冊與 Onboarding 任務庫

歡迎來到國立政治大學 **政大通（NCCUpass）** 後端組新人手冊與任務庫。

本專案使用 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 建置，旨在為新加入政大通後端團隊的夥伴提供平滑、清晰且具實戰感的學習路徑與 Onboarding Task。

---

## 本地開發與預覽

### 1. 建立並啟動 Python 虛擬環境
```bash
# 建立虛擬環境
python -m venv .venv

# 啟動虛擬環境 (Windows PowerShell)
.venv\Scripts\Activate.ps1

# 啟動虛擬環境 (macOS / Linux)
source .venv/bin/activate
```

### 2. 安裝依賴套件
```bash
pip install -r requirements.txt
```

### 3. 啟動本地即時預覽伺服器
```bash
mkdocs serve
```
瀏覽器開啟 `http://127.0.0.1:8000` 即可預覽網站與即時熱重載（Hot-reload）。

### 4. 建置靜態網站
```bash
mkdocs build --strict
```

---

## GitHub Pages 部署

本專案已配置 GitHub Actions 工作流程（`.github/workflows/deploy.yml`）。
當推送到 `main` 分支時，GitHub Actions 會自動建置並發布至 `gh-pages` 分支。

### 首次設定步驟：
1. 將本專案推送到 GitHub Repository（例如 `NCCUpass/backend-onboarding`）。
2. 在 GitHub Repo 頁面進入 **Settings** -> **Pages**。
3. 在 **Build and deployment** 下的 **Source** 選擇 **Deploy from a branch**。
4. Branch 選擇 `gh-pages` / `/(root)` 並儲存。
5. 稍候片刻即可透過 `https://<organization>.github.io/<repo-name>/` 瀏覽正式文件網站。

---

## 目錄架構

```text
.
├── .github/workflows/deploy.yml    # GitHub Actions 自動部署
├── docs/                           # 所有 Markdown 文件內容
│   ├── index.md                    # 首頁
│   ├── stylesheets/
│   │   └── extra.css               # 自訂樣式與 Mermaid 顯示優化
│   ├── intro/                      # 團隊簡介與文化
│   ├── setup/                      # 開發環境安裝指南
│   ├── roadmap/                    # 基礎學習資源與觀念
│   ├── task/                       # Onboarding Task (Sprint 1 ~ 5)
│   ├── specs/                      # API 規格書
│   ├── standards/                  # 團隊協作與程式碼規範
│   └── faq.md                      # 常見問題
├── mkdocs.yml                     # MkDocs 配置檔
├── requirements.txt               # 依賴清單
└── README.md
```
