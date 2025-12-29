---
name: MCP 建構器
description: 創建高品質 MCP（Model Context Protocol）伺服器的指南，使 LLM 能夠透過精心設計的工具與外部服務互動。在使用 Python（FastMCP）或 Node/TypeScript（MCP SDK）建構 MCP 伺服器以整合外部 API 或服務時使用。
license: 完整條款請見 LICENSE.txt
---

# MCP 伺服器開發指南

## 概述

創建 MCP（Model Context Protocol）伺服器，使 LLM 能夠透過精心設計的工具與外部服務互動。MCP 伺服器的品質取決於它如何有效地使 LLM 能夠完成實際任務。

---

# 流程

## 高層級工作流程

創建高品質 MCP 伺服器涉及四個主要階段：

### 階段 1：深入研究和規劃

#### 1.1 理解現代 MCP 設計

**API 覆蓋範圍 vs. 工作流程工具：**
在全面的 API 端點覆蓋與專門的工作流程工具之間取得平衡。工作流程工具對特定任務可能更方便，而全面覆蓋給予代理靈活性來組合操作。效能因客戶端而異——某些客戶端受益於組合基本工具的程式碼執行，而其他客戶端則更適合更高層級的工作流程。當不確定時，優先考慮全面的 API 覆蓋。

**工具命名和可發現性：**
清晰、描述性的工具名稱幫助代理快速找到正確的工具。使用一致的前綴（例如 `github_create_issue`、`github_list_repos`）和動作導向的命名。

**上下文管理：**
代理受益於簡潔的工具描述和過濾/分頁結果的能力。設計返回聚焦、相關資料的工具。某些客戶端支援程式碼執行，可幫助代理有效地過濾和處理資料。

**可操作的錯誤訊息：**
錯誤訊息應引導代理朝向解決方案，提供具體建議和後續步驟。

#### 1.2 研究 MCP 協定文件

**導覽 MCP 規格：**

從站點地圖開始尋找相關頁面：`https://modelcontextprotocol.io/sitemap.xml`

然後使用 `.md` 後綴獲取特定頁面的 markdown 格式（例如 `https://modelcontextprotocol.io/specification/draft.md`）。

要審查的關鍵頁面：
- 規格概述和架構
- 傳輸機制（可串流 HTTP、stdio）
- 工具、資源和提示定義

#### 1.3 研究框架文件

**建議的技術堆疊：**
- **語言**：TypeScript（高品質 SDK 支援，在許多執行環境中有良好相容性，如 MCPB。此外 AI 模型擅長生成 TypeScript 程式碼，受益於其廣泛使用、靜態類型和良好的 linting 工具）
- **傳輸**：遠端伺服器使用可串流 HTTP，使用無狀態 JSON（比有狀態會話和串流回應更簡單擴展和維護）。本地伺服器使用 stdio。

**載入框架文件：**

- **MCP 最佳實踐**：[查看最佳實踐](./reference/mcp_best_practices.md) - 核心指南

**TypeScript（建議）：**
- **TypeScript SDK**：使用 WebFetch 載入 `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [TypeScript 指南](./reference/node_mcp_server.md) - TypeScript 模式和範例

**Python：**
- **Python SDK**：使用 WebFetch 載入 `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [Python 指南](./reference/python_mcp_server.md) - Python 模式和範例

#### 1.4 規劃你的實作

**理解 API：**
審查服務的 API 文件以識別關鍵端點、認證需求和資料模型。根據需要使用網頁搜尋和 WebFetch。

**工具選擇：**
優先考慮全面的 API 覆蓋。列出要實作的端點，從最常見的操作開始。

---

### 階段 2：實作

#### 2.1 設定專案結構

參見語言特定指南了解專案設定：
- [TypeScript 指南](./reference/node_mcp_server.md) - 專案結構、package.json、tsconfig.json
- [Python 指南](./reference/python_mcp_server.md) - 模組組織、依賴項

#### 2.2 實作核心基礎設施

創建共享工具：
- 帶認證的 API 客戶端
- 錯誤處理輔助函數
- 回應格式化（JSON/Markdown）
- 分頁支援

#### 2.3 實作工具

對於每個工具：

**輸入 Schema：**
- 使用 Zod（TypeScript）或 Pydantic（Python）
- 包含約束和清晰描述
- 在欄位描述中添加範例

**輸出 Schema：**
- 盡可能為結構化資料定義 `outputSchema`
- 在工具回應中使用 `structuredContent`（TypeScript SDK 功能）
- 幫助客戶端理解和處理工具輸出

**工具描述：**
- 功能的簡潔摘要
- 參數描述
- 返回類型 schema

**實作：**
- 對 I/O 操作使用 async/await
- 帶可操作訊息的適當錯誤處理
- 適用時支援分頁
- 使用現代 SDK 時同時返回文字內容和結構化資料

**註解：**
- `readOnlyHint`：true/false
- `destructiveHint`：true/false
- `idempotentHint`：true/false
- `openWorldHint`：true/false

---

### 階段 3：審查和測試

#### 3.1 程式碼品質

審查：
- 沒有重複程式碼（DRY 原則）
- 一致的錯誤處理
- 完整的類型覆蓋
- 清晰的工具描述

#### 3.2 建構和測試

**TypeScript：**
- 執行 `npm run build` 驗證編譯
- 使用 MCP Inspector 測試：`npx @modelcontextprotocol/inspector`

**Python：**
- 驗證語法：`python -m py_compile your_server.py`
- 使用 MCP Inspector 測試

參見語言特定指南了解詳細的測試方法和品質檢查清單。

---

### 階段 4：創建評估

實作 MCP 伺服器後，創建全面的評估來測試其效果。

**載入[評估指南](./reference/evaluation.md)以獲取完整的評估指南。**

#### 4.1 理解評估目的

使用評估來測試 LLM 是否能有效地使用你的 MCP 伺服器回答實際、複雜的問題。

#### 4.2 創建 10 個評估問題

要創建有效的評估，請遵循評估指南中概述的流程：

1. **工具檢查**：列出可用工具並理解其功能
2. **內容探索**：使用唯讀操作探索可用資料
3. **問題生成**：創建 10 個複雜、實際的問題
4. **答案驗證**：自己解決每個問題以驗證答案

#### 4.3 評估需求

確保每個問題是：
- **獨立**：不依賴其他問題
- **唯讀**：只需要非破壞性操作
- **複雜**：需要多次工具調用和深入探索
- **實際**：基於人類會關心的真實使用案例
- **可驗證**：有單一、清晰的答案，可透過字串比較驗證
- **穩定**：答案不會隨時間改變

#### 4.4 輸出格式

創建具有此結構的 XML 檔案：

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- More qa_pairs... -->
</evaluation>
```

---

# 參考檔案

## 文件庫

在開發過程中根據需要載入這些資源：

### 核心 MCP 文件（首先載入）
- **MCP 協定**：從 `https://modelcontextprotocol.io/sitemap.xml` 的站點地圖開始，然後使用 `.md` 後綴獲取特定頁面
- [MCP 最佳實踐](./reference/mcp_best_practices.md) - 通用 MCP 指南，包括：
  - 伺服器和工具命名慣例
  - 回應格式指南（JSON vs Markdown）
  - 分頁最佳實踐
  - 傳輸選擇（可串流 HTTP vs stdio）
  - 安全性和錯誤處理標準

### SDK 文件（在階段 1/2 載入）
- **Python SDK**：從 `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md` 獲取
- **TypeScript SDK**：從 `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md` 獲取

### 語言特定實作指南（在階段 2 載入）
- [Python 實作指南](./reference/python_mcp_server.md) - 完整的 Python/FastMCP 指南，包括：
  - 伺服器初始化模式
  - Pydantic 模型範例
  - 使用 `@mcp.tool` 的工具註冊
  - 完整工作範例
  - 品質檢查清單

- [TypeScript 實作指南](./reference/node_mcp_server.md) - 完整的 TypeScript 指南，包括：
  - 專案結構
  - Zod schema 模式
  - 使用 `server.registerTool` 的工具註冊
  - 完整工作範例
  - 品質檢查清單

### 評估指南（在階段 4 載入）
- [評估指南](./reference/evaluation.md) - 完整的評估創建指南，包括：
  - 問題創建指南
  - 答案驗證策略
  - XML 格式規格
  - 範例問題和答案
  - 使用提供的腳本執行評估
