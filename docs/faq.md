# 常見問題與除錯指南 (FAQ)

這裡整理了歷屆學長姐與新人最容易遇到的踩坑紀錄與解決方法！

---

## 資料庫與連線問題

### Q1: 連線 MySQL 報錯 `Can't connect to MySQL server on 'localhost' (10061)`？
**原因**：本地的 MySQL 服務尚未啟動，或是埠號（Port）被佔用。  
**解決步驟**：
1. 若使用 Docker，請確認終端機輸入 `docker ps` 能看到 `nccupass_mysql` 容器正在運行。
2. 若沒有，請執行 `docker compose up -d` 重新啟動。
3. 若使用團隊遠端 DB，請確認 `.env` 中的 `DB_HOST` 與校園 VPN / 網路是否正常連通。

### Q2: 執行 Alembic 或 Python 時報錯 `ModuleNotFoundError: No module named 'app'`？
**原因**：Python 找不到專案根目錄。  
**解決方式**：
在專案根目錄執行：
```bash
# Windows PowerShell
$env:PYTHONPATH="."

# macOS / Linux
export PYTHONPATH=.
```
或者在執行 uvicorn 時指定模組路徑：`uvicorn app.main:app --reload`。

---

## Git 與 GitHub 問題

### Q1: 推送代碼時報錯 `Permission denied (publickey)`？
**原因**：你的 SSH Key 尚未正確加入 GitHub 帳號，或是本地沒有載入金鑰。  
**解決方式**：
請參考 [Git 與 GitHub 環境設定](setup/git-github.md)，執行 `ssh -T git@github.com` 確認連線狀態。

### Q2: 不小心把 `.env` 機密檔案 commit 上去了怎麼辦？
**緊急處理**：
1. 立即將 `.env` 加入 `.gitignore`。
2. 從 Git 快取中移除該檔案（保留本地檔案）：
   ```bash
   git rm --cached .env
   git commit -m "chore: remove .env from git tracking"
   git push origin <your-branch>
   ```
3. 立即更換資料庫密碼或 JWT Secret Key！

---

## Python 與套件管理

### Q1: VS Code 顯示找不到套件，但 `pip list` 裡面明明有？
**原因**：VS Code 右下角選到了全域的 Python，而不是你的虛擬環境 `.venv`。  
**解決方式**：
按快捷鍵 ++ctrl+shift+p++（或 ++cmd+shift+p++）輸入 `Python: Select Interpreter`，手動選擇含有 `('.venv': venv)` 的那一項。

---

如果你的問題不在這裡，歡迎隨時在 Discord `#onboarding-help` 提問，並請遵循 [提問範本](intro/culture.md#asking-for-help)！
