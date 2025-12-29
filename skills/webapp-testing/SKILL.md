---
name: 網頁應用程式測試
description: 使用 Playwright 與本地網頁應用程式互動和測試的工具包。支援驗證前端功能、除錯 UI 行為、擷取瀏覽器截圖和查看瀏覽器日誌。
license: 完整條款請見 LICENSE.txt
---

# 網頁應用程式測試

要測試本地網頁應用程式，撰寫原生 Python Playwright 腳本。

**可用輔助腳本**：
- `scripts/with_server.py` - 管理伺服器生命週期（支援多個伺服器）

**始終先用 `--help` 執行腳本**以查看用法。在嘗試執行腳本並發現絕對需要自定義解決方案之前，不要讀取原始碼。這些腳本可能非常大，因此會污染你的上下文視窗。它們的存在是作為黑盒腳本直接調用，而非被讀入你的上下文視窗。

## 決策樹：選擇你的方法

```
使用者任務 → 是靜態 HTML 嗎？
    ├─ 是 → 直接讀取 HTML 檔案以識別選擇器
    │         ├─ 成功 → 使用選擇器撰寫 Playwright 腳本
    │         └─ 失敗/不完整 → 視為動態（見下方）
    │
    └─ 否（動態 webapp）→ 伺服器已在執行嗎？
        ├─ 否 → 執行：python scripts/with_server.py --help
        │        然後使用輔助腳本 + 撰寫簡化的 Playwright 腳本
        │
        └─ 是 → 偵察後行動：
            1. 導覽並等待 networkidle
            2. 擷取截圖或檢查 DOM
            3. 從渲染狀態識別選擇器
            4. 使用發現的選擇器執行動作
```

## 範例：使用 with_server.py

要啟動伺服器，先執行 `--help`，然後使用輔助腳本：

**單一伺服器：**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**多個伺服器（例如後端 + 前端）：**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

要創建自動化腳本，只包含 Playwright 邏輯（伺服器自動管理）：
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True) # 始終以 headless 模式啟動 chromium
    page = browser.new_page()
    page.goto('http://localhost:5173') # 伺服器已在執行並準備就緒
    page.wait_for_load_state('networkidle') # 關鍵：等待 JS 執行完成
    # ... 你的自動化邏輯
    browser.close()
```

## 偵察後行動模式

1. **檢查渲染的 DOM**：
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **識別選擇器**從檢查結果

3. **執行動作**使用發現的選擇器

## 常見陷阱

- **不要**在動態應用上等待 `networkidle` 之前檢查 DOM
- **要**在檢查前等待 `page.wait_for_load_state('networkidle')`

## 最佳實踐

- **將捆綁腳本作為黑盒使用** - 要完成任務，考慮 `scripts/` 中的可用腳本是否能幫助。這些腳本可靠地處理常見、複雜的工作流程，不會使上下文視窗混亂。使用 `--help` 查看用法，然後直接調用。
- 使用 `sync_playwright()` 進行同步腳本
- 完成後始終關閉瀏覽器
- 使用描述性選擇器：`text=`、`role=`、CSS 選擇器或 ID
- 添加適當的等待：`page.wait_for_selector()` 或 `page.wait_for_timeout()`

## 參考檔案

- **examples/** - 展示常見模式的範例：
  - `element_discovery.py` - 發現頁面上的按鈕、連結和輸入
  - `static_html_automation.py` - 使用 file:// URL 處理本地 HTML
  - `console_logging.py` - 在自動化過程中擷取控制台日誌
