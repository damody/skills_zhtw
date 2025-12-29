# MCP 伺服器最佳實踐

## 快速參考

### 伺服器命名
- **Python**：`{service}_mcp`（例如 `slack_mcp`）
- **Node/TypeScript**：`{service}-mcp-server`（例如 `slack-mcp-server`）

### 工具命名
- 使用帶有服務前綴的 snake_case
- 格式：`{service}_{action}_{resource}`
- 範例：`slack_send_message`、`github_create_issue`

### 回應格式
- 支援 JSON 和 Markdown 兩種格式
- JSON 用於程式化處理
- Markdown 用於人類可讀性

### 分頁
- 始終遵守 `limit` 參數
- 回傳 `has_more`、`next_offset`、`total_count`
- 預設為 20-50 個項目

### 傳輸
- **Streamable HTTP**：用於遠端伺服器、多客戶端場景
- **stdio**：用於本機整合、命令列工具
- 避免 SSE（已棄用，改用 streamable HTTP）

---

## 伺服器命名慣例

遵循這些標準化的命名模式：

**Python**：使用格式 `{service}_mcp`（小寫加底線）
- 範例：`slack_mcp`、`github_mcp`、`jira_mcp`

**Node/TypeScript**：使用格式 `{service}-mcp-server`（小寫加連字號）
- 範例：`slack-mcp-server`、`github-mcp-server`、`jira-mcp-server`

名稱應該是通用的、能描述正在整合的服務、容易從任務描述推斷出來，且不包含版本號。

---

## 工具命名和設計

### 工具命名

1. **使用 snake_case**：`search_users`、`create_project`、`get_channel_info`
2. **包含服務前綴**：預期您的 MCP 伺服器可能與其他 MCP 伺服器一起使用
   - 使用 `slack_send_message` 而不是僅 `send_message`
   - 使用 `github_create_issue` 而不是僅 `create_issue`
3. **以動作為導向**：以動詞開頭（get、list、search、create 等）
4. **具體明確**：避免可能與其他伺服器衝突的通用名稱

### 工具設計

- 工具描述必須狹義且明確地描述功能
- 描述必須精確符合實際功能
- 提供工具註解（readOnlyHint、destructiveHint、idempotentHint、openWorldHint）
- 保持工具操作聚焦且原子化

---

## 回應格式

所有回傳資料的工具都應支援多種格式：

### JSON 格式（`response_format="json"`）
- 機器可讀的結構化資料
- 包含所有可用欄位和中繼資料
- 一致的欄位名稱和類型
- 用於程式化處理

### Markdown 格式（`response_format="markdown"`，通常為預設）
- 人類可讀的格式化文字
- 使用標題、清單和格式以提高清晰度
- 將時間戳轉換為人類可讀格式
- 顯示名稱並在括號中附上 ID
- 省略冗長的中繼資料

---

## 分頁

對於列出資源的工具：

- **始終遵守 `limit` 參數**
- **實作分頁**：使用 `offset` 或基於游標的分頁
- **回傳分頁中繼資料**：包含 `has_more`、`next_offset`/`next_cursor`、`total_count`
- **永不將所有結果載入記憶體**：對於大型資料集尤其重要
- **預設為合理的限制**：20-50 個項目是典型值

分頁回應範例：
```json
{
  "total": 150,
  "count": 20,
  "offset": 0,
  "items": [...],
  "has_more": true,
  "next_offset": 20
}
```

---

## 傳輸選項

### Streamable HTTP

**最適合**：遠端伺服器、Web 服務、多客戶端場景

**特性**：
- 透過 HTTP 的雙向通訊
- 支援多個同時連線的客戶端
- 可部署為 Web 服務
- 啟用伺服器到客戶端的通知

**使用時機**：
- 同時服務多個客戶端
- 部署為雲端服務
- 與 Web 應用程式整合

### stdio

**最適合**：本機整合、命令列工具

**特性**：
- 標準輸入/輸出串流通訊
- 簡單設定，無需網路配置
- 作為客戶端的子程序執行

**使用時機**：
- 為本機開發環境建構工具
- 與桌面應用程式整合
- 單一使用者、單一工作階段場景

**注意**：stdio 伺服器不應記錄到 stdout（使用 stderr 進行記錄）

### 傳輸選擇

| 標準 | stdio | Streamable HTTP |
|-----------|-------|-----------------|
| **部署** | 本機 | 遠端 |
| **客戶端** | 單一 | 多個 |
| **複雜度** | 低 | 中 |
| **即時** | 否 | 是 |

---

## 安全性最佳實踐

### 認證和授權

**OAuth 2.1**：
- 使用具有認可機構憑證的安全 OAuth 2.1
- 在處理請求前驗證存取權杖
- 僅接受專門用於您伺服器的權杖

**API 金鑰**：
- 將 API 金鑰儲存在環境變數中，永不放在程式碼中
- 在伺服器啟動時驗證金鑰
- 當認證失敗時提供清楚的錯誤訊息

### 輸入驗證

- 清理檔案路徑以防止目錄遍歷
- 驗證 URL 和外部識別碼
- 檢查參數大小和範圍
- 防止系統呼叫中的命令注入
- 對所有輸入使用 schema 驗證（Pydantic/Zod）

### 錯誤處理

- 不要向客戶端暴露內部錯誤
- 在伺服器端記錄安全相關錯誤
- 提供有幫助但不洩露資訊的錯誤訊息
- 在錯誤後清理資源

### DNS 重新綁定保護

對於在本機執行的 streamable HTTP 伺服器：
- 啟用 DNS 重新綁定保護
- 在所有傳入連線上驗證 `Origin` 標頭
- 綁定到 `127.0.0.1` 而不是 `0.0.0.0`

---

## 工具註解

提供註解以幫助客戶端理解工具行為：

| 註解 | 類型 | 預設值 | 描述 |
|-----------|------|---------|-------------|
| `readOnlyHint` | boolean | false | 工具不修改其環境 |
| `destructiveHint` | boolean | true | 工具可能執行破壞性更新 |
| `idempotentHint` | boolean | false | 使用相同參數重複呼叫不會產生額外效果 |
| `openWorldHint` | boolean | true | 工具與外部實體互動 |

**重要**：註解是提示，不是安全保證。客戶端不應僅基於註解做出安全關鍵決策。

---

## 錯誤處理

- 使用標準 JSON-RPC 錯誤碼
- 在結果物件中報告工具錯誤（不是協定層級錯誤）
- 提供有幫助、具體的錯誤訊息並建議下一步
- 不要暴露內部實作細節
- 在錯誤時正確清理資源

錯誤處理範例：
```typescript
try {
  const result = performOperation();
  return { content: [{ type: "text", text: result }] };
} catch (error) {
  return {
    isError: true,
    content: [{
      type: "text",
      text: `Error: ${error.message}. Try using filter='active_only' to reduce results.`
    }]
  };
}
```

---

## 測試需求

全面的測試應涵蓋：

- **功能測試**：驗證有效/無效輸入的正確執行
- **整合測試**：測試與外部系統的互動
- **安全性測試**：驗證認證、輸入清理、速率限制
- **效能測試**：檢查負載下的行為、逾時
- **錯誤處理**：確保正確的錯誤報告和清理

---

## 文件需求

- 提供所有工具和功能的清楚文件
- 包含可運作的範例（每個主要功能至少 3 個）
- 記錄安全性考量
- 指定所需的權限和存取層級
- 記錄速率限制和效能特性
