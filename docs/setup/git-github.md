# Git 與 GitHub 環境設定

Git 是現代軟體工程最重要的版本控制工具。請依照本篇教學完成 Git 的安裝與 GitHub SSH 金鑰連線設定。

---

## 1. 安裝 Git

=== "Windows"
    1. 前往 [Git for Windows 官方網站](https://gitforwindows.org/) 下載安裝檔。
    2. 安裝過程中建議勾選 **Git Bash** 與 **Use Visual Studio Code as Git's default editor**。
    3. 安裝完成後，開啟 PowerShell 或 Git Bash 驗證：
    ```bash
    git --version
    ```

=== "macOS"
    若已安裝 Homebrew，直接執行：
    ```bash
    brew install git
    ```
    或在終端機輸入 `git`，系統會提示安裝 Command Line Tools。

=== "Linux (Ubuntu / Debian)"
    ```bash
    sudo apt update
    sudo apt install git -y
    ```

---

## 2. 設定使用者基本資訊

在終端機中設定你的全域名稱與 Email（**請填寫與你 GitHub 帳號相同的 Email**）：

```bash
git config --global user.name "Your Name (例如: Eric Chen)"
git config --global user.email "your.email@example.com"
```

建議同時設定預設分支名稱為 `main`：
```bash
git config --global init.defaultBranch main
```

---

## 3. 設定 GitHub SSH Key（免去每次輸入密碼）

透過 SSH 連線 GitHub 比使用 HTTPS Token 更加安全且便利。

### 步驟 A：產生 SSH Key
開啟終端機，執行以下指令（換成你的 GitHub Email）：
```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
```
遇到提示時一直按 ++enter++ 使用預設路徑與不設定密碼（passphrase）即可。

### 步驟 B：複製公鑰內容
=== "Windows (PowerShell)"
    ```powershell
    Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
    ```
=== "macOS"
    ```bash
    pbcopy < ~/.ssh/id_ed25519.pub
    ```
=== "Linux"
    ```bash
    cat ~/.ssh/id_ed25519.pub
    # 手動複製終端機印出的整行字串 (以 ssh-ed25519 開頭)
    ```

### 步驟 C：貼到 GitHub
1. 開啟 [GitHub SSH Settings 頁面](https://github.com/settings/keys)。
2. 點擊綠色的 **New SSH key**。
3. **Title**：自訂名稱（例如：`My MacBook` 或 `Desktop-Windows`）。
4. **Key**：將剛才複製的公鑰貼上。
5. 點擊 **Add SSH key**。

### 步驟 D：測試連線
在終端機輸入：
```bash
ssh -T git@github.com
```
看到 `Hi username! You've successfully authenticated...` 就代表設定大功告成！
