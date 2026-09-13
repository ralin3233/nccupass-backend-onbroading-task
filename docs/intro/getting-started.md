# 新人第一週 Checklist

這是一份幫助你在第一週有條不紊完成準備的檢查清單。你可以一邊閱讀，一邊在心中打勾！

---

## 第一天：環境與帳號準備 (Day 1)

- [ ] 加入政大通 Discord 工作區與 `#backend-dev` 頻道
- [ ] 設定 GitHub 帳號並將公鑰（SSH Key）加入 GitHub
- [ ] 取得政大通 GitHub Organization 邀請並加入團隊
- [ ] 安裝 Python 3.11+ 並確認終端機能正常執行 `python --version`
- [ ] 安裝 VS Code 與推薦擴充套件（Python, Pylance, Black, GitLens）
- [ ] 下載並安裝資料庫圖形工具（推薦 TablePlus，參考 [資料庫連線工具指南](../setup/database.md)）

---

## 第二天：基礎觀念建立 (Day 2)

- [ ] 閱讀 [Web 與 HTTP 基礎](../roadmap/01-web-basics.md)
- [ ] 閱讀 [Python 現代特性與 Type Hints](../roadmap/02-python-modern.md)
- [ ] 閱讀 [FastAPI 核心概念](../roadmap/03-fastapi-core.md)
- [ ] 在本地練習建立第一個 FastAPI Hello World 範例並成功在瀏覽器打開 `/docs` (Swagger UI)

---

## 第三天至第五天：啟動 Onboarding Task (Day 3 ~ 5)

- [ ] 詳讀 [Onboarding 任務總覽](../task/overview.md) 與 [API 規格書](../specs/memo-api.md)
- [ ] Clone [練習用專案 Repo (GitLab)](https://gitlab.com/nccupass/nccupass-memo-service-for-beginner.git) 並建立虛擬環境
- [ ] **完成 Sprint 1**：實作基礎 Pydantic Schemas、CRUD 端點（建立/單筆查詢/刪除）與基礎測試
- [ ] **完成 Sprint 2**：實作列表多條件篩選（分頁/狀態/搜尋）、PATCH 部分更新與進階測試，並開出 MR 繳交！

---

## 結業里程碑

當你的 Onboarding PR 通過 Code Review 並成功合併（Merged），恭喜你！你已經正式具備參與政大通線上專案開發的全部技能，我們將為你分配第一個正式 Sprint 任務！
