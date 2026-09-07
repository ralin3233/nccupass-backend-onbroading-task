# Sprint 3 - 驗證與全域錯誤處理

> **Sprint 目標**：提升 API 健壯度與前後端對接體驗，設計嚴格的輸入驗證與統一格式的 JSON 錯誤回應。

---

## 任務清單 (Task Checklist)

- [ ] 在 Pydantic 模型中加入自訂欄位驗證（例如標題不能全為空白字元）
- [ ] 定義團隊統一的標準錯誤回應結構（Standard Error Response JSON）
- [ ] 撰寫自訂例外類別（例如 `MemoNotFoundError`, `PermissionDeniedError`）
- [ ] 在 `main.py` 註冊全域例外處理器（Global Exception Handlers）
- [ ] 攔截 `RequestValidationError` (422) 並轉譯成對前端更親切的錯誤格式

---

## 實作指引

### 1. 統一錯誤格式標準

後端在任何錯誤情況下，都應回傳結構一致的 JSON：

```json
{
  "success": false,
  "error_code": "MEMO_NOT_FOUND",
  "message": "找不到 ID 為 99 的備忘錄",
  "details": null
}
```

### 2. 自訂例外與全域處理器

```python
from fastapi import Request, status
from fastapi.responses import JSONResponse

class AppException(Exception):
    def __init__(self, error_code: str, message: str, status_code: int = 400):
        self.error_code = error_code
        self.message = message
        self.status_code = status_code

class MemoNotFoundError(AppException):
    def __init__(self, memo_id: int):
        super().__init__(
            error_code="MEMO_NOT_FOUND",
            message=f"找不到指定的備忘錄 (ID: {memo_id})",
            status_code=status.HTTP_404_NOT_FOUND
        )

# 在 main.py 中捕獲並統整回應
@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error_code": exc.error_code,
            "message": exc.message,
            "details": None
        }
    )
```

---

## 驗收條件 (Acceptance Criteria)

1. 當傳送空字串或純空白標題 `{"title": "   "}` 建立備忘錄時，API 回傳清晰的驗證失敗訊息。
2. 查詢或刪除不存在的 ID 時，API 回傳符合標準格式的 JSON 錯誤訊息，而非 FastAPI 預設的陽春 `{"detail": "..."}`。
