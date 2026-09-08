# VS Code 推薦配置與外掛

Visual Studio Code (VS Code) 是政大通後端組推薦的標準開發工具。搭配合適的擴充套件，能極大提升開發效率與語法除錯速度。

---

## 參考擴充套件（Extensions）

請在 VS Code 側邊欄的 **Extensions**（快捷鍵 ++ctrl+shift+x++ 或 ++cmd+shift+x++）中搜尋並安裝以下套件：

| 擴充套件名稱 | 套件 ID | 用途說明 |
| :--- | :--- | :--- |
| **Python** | `ms-python.python` | 微軟官方 Python 語言支援 |
| **Pylance** | `ms-python.vscode-pylance` | 超高速型別檢查、代碼自動補全與定義跳轉 |
| **Black Formatter** | `ms-python.black-formatter` | 官方標準 Python 格式化工具 |
| **GitLens** | `eamodio.gitlens` | 強大的 Git 視覺化工具，即時顯示每一行是誰在何時寫的 |
| **Even Better TOML** | `tamasfe.even-better-toml` | `pyproject.toml` 設定檔高亮與驗證 |
| **Database Client** (選配) | `cweijan.vscode-database-client2` | 在 VS Code 內直接瀏覽和查詢 MySQL 資料庫 |

---

## 推薦專案設定檔 (`.vscode/settings.json`)

在你的專案根目錄下建立 `.vscode/settings.json`，可以讓 VS Code 自動幫你在存檔時格式化代碼：

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "python.analysis.typeCheckingMode": "basic",
  "files.autoSave": "onFocusChange"
}
```

> [!TIP]
> 記得按下 ++ctrl+shift+p++（或 ++cmd+shift+p++），輸入 `Python: Select Interpreter`，並選擇帶有 `('.venv': venv)` 的解釋器路徑，確保 VS Code 讀取的是虛擬環境內的套件！
