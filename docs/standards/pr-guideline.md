# Pull Request (PR) 與 Code Review 流程

Pull Request 是團隊將新程式碼併入主幹線前的最後一道品質防線。

---

## 提交 PR 前的自我檢查（Self-Checklist）

在點擊建立 PR 之前，請在本地先完成以下檢查：

- [ ] 程式碼已通過 Formatter 格式化（例如 `black`）
- [ ] 沒有遺留未清理的 `print()`、測試用 Hardcoded 資料或密碼機密（`.env`）
- [ ] 所有本地單元測試均執行通過（`pytest`）
- [ ] 分支已與最新的 `main` 同步且無衝突（Merge Conflict）

---

## PR 描述標準範本（Template）

在 GitHub 建立 PR 時，請包含以下三個區塊：

```markdown
### 變更摘要 (Summary)
- 實作備忘錄 CRUD RESTful API (`POST /memos`, `GET /memos`, etc.)
- 加入 Pydantic 欄位驗證與自訂統一錯誤回應
- 新增 `pytest` 測試覆蓋 CRUD 與驗證情境

### 關聯 Issue (Related Issues)
- Closes #12

### 驗證方式與成果 (Testing & Screenshots)
- [x] 本地執行 `pytest` 通過 (12 passed)
- [x] Swagger UI 互動測試正常 (附截圖)

![Swagger Test Screenshot](https://user-images.githubusercontent.com/.../demo.png)
```

---

## Code Review 溝通約定

為了讓 Review 過程高效且友善，我們鼓勵在評論中加上**意圖標籤（Conventional Comments）**：

- `[suggestion]`：非強制的優化建議（例如：「這行可以用 list comprehension 寫得更簡潔」）
- `[question]`：單純想了解為什麼這樣設計（例如：「想請問這裡選擇不用 eager loading 的考量是？」）
- `[nitpick]` / `[nit]`：微小的細節（例如命名錯字、贅字）
- `[blocker]`：嚴重的 Bug、安全性隱患或邏輯錯誤，必須修正後才能 Merge
- `[praise]`：給予讚賞（例如：「這段例外處理寫得非常優雅，學到了！」）
