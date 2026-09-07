# Git Commit 訊息規範

在政大通團隊中，我們嚴格遵循業界標準的 **[Conventional Commits](https://www.conventionalcommits.org/zh-hant/)** 規範。良好的 Commit 歷史能讓團隊迅速追蹤版本變更、生成自動 Release Notes 並降低維護成本。

---

## Commit 訊息基本結構

```text
<type>(<optional scope>): <description>

[optional body]

[optional footer(s)]
```

### 範例：
```text
feat(auth): implement JWT token generation and login endpoint

- Add pyjwt dependency
- Hash password with bcrypt
- Return access_token with 1 hour expiration

Closes #14
```

---

## 常用 Type 前綴清單

| Type | 說明 | 範例 |
| :--- | :--- | :--- |
| **`feat`** | 新增功能 (Feature) | `feat(memo): add pagination support to list memos` |
| **`fix`** | 修復 Bug (Bug Fix) | `fix(auth): handle expired token with 401 response` |
| **`docs`** | 僅修改文件 (Documentation) | `docs: update API specification for memo patch endpoint` |
| **`style`** | 不影響程式碼邏輯的格式微調 (排版、分號、空格) | `style: format python code with black` |
| **`refactor`**| 重構程式碼（非新增功能也非修 Bug） | `refactor(db): extract database session dependency into separate module` |
| **`perf`** | 提升效能的修改 | `perf(query): add database index on memo title and owner_id` |
| **`test`** | 新增或修改測試 (Tests) | `test(memo): add unit test for deleting non-existent memo` |
| **`chore`** | 建置流程、依賴更新或輔助工具變動 | `chore(deps): upgrade fastapi to 0.110.0` |
| **`ci`** | CI/CD 腳本調整 (GitHub Actions) | `ci: add pytest automated testing workflow` |

---

## 撰寫原則與黃金守則

1. **以動詞開頭**：使用繁體中文或英文祈使句（例如：`feat: 實作備忘錄分頁查詢` 或 `feat: implement memo pagination`）。
2. **單一 Commit 只做一件事**：不要把修 Bug、改排版、寫新功能全部塞進同一個 Commit。
3. **避免無意義訊息**：嚴禁出現 `update`, `fix bug`, `wip`, `test1234` 這種無法傳遞資訊的 Commit 訊息！
