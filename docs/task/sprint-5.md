# Sprint 5 - 自動化測試與 Pull Request 交付

> **Sprint 目標**：使用 `pytest` 撰寫自動化測試確保 API 功能無虞，並在 GitHub 上發起正式的 Pull Request 進行 Code Review 與驗收結業！

---

## 任務清單 (Task Checklist)

- [ ] 安裝測試套件：`pytest`, `httpx` (FastAPI 官方推薦 TestClient 客戶端)
- [ ] 設定 `conftest.py`：建立獨立的測試用 SQLite in-memory 資料庫或測試 DB Session
- [ ] 撰寫 `test_auth.py`：測試註冊、登入成功與密碼錯誤情況
- [ ] 撰寫 `test_memos.py`：測試備忘錄 CRUD 流程與使用者資料隔離邏輯
- [ ] 執行 `pytest` 確認所有測試 100% 通過（All Green）
- [ ] 建立 `.github/workflows/ci.yml` 實現 GitHub Actions 自動跑測試
- [ ] 發起 Pull Request，填寫 PR 範本並邀請 Mentor 進行 Code Review

---

## 測試範例 (`tests/test_memos.py`)

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_and_get_memo():
    # 1. 先註冊並登入取得 Token
    reg_res = client.post("/api/v1/auth/register", json={
        "email": "test@nccu.edu.tw",
        "username": "testuser",
        "password": "Password123!"
    })
    token = reg_res.json()["access_token"]
    headers = {"Authorization": f"Bearer {token}"}

    # 2. 建立一筆備忘錄
    create_res = client.post(
        "/api/v1/memos",
        json={"title": "期末專案開會", "priority": 3},
        headers=headers
    )
    assert create_res.status_code == 201
    memo_id = create_res.json()["id"]

    # 3. 查詢單筆備忘錄
    get_res = client.get(f"/api/v1/memos/{memo_id}", headers=headers)
    assert get_res.status_code == 200
    assert get_res.json()["title"] == "期末專案開會"
```

---

## 提交 PR 與驗收指引

1. 確保所有 Commit 遵循 Conventional Commits 格式。
2. 將所有變更推送到遠端分支：
   ```bash
   git push origin feat/sprint-5-tests
   ```
3. 前往 GitHub 點擊 **Compare & pull request**。
4. 依據 [Pull Request 規範](../standards/pr-guideline.md) 填寫說明：
   - 標題格式：`feat: complete memo service onboarding task`
   - 附上 pytest 執行成功輸出截圖或 CI 綠燈截圖。
5. Assign 你的 Mentor，並在 Discord `#onboarding-help` 告知：「嗨 @Mentor，我的 Onboarding 任務 PR 已經發起，再麻煩幫我 Review！」

恭喜你完成所有 Sprint 任務！
