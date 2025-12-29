# Node/TypeScript MCP 伺服器實作指南

## 概述

本文件提供使用 MCP TypeScript SDK 實作 MCP 伺服器的 Node/TypeScript 特定最佳實踐和範例。涵蓋專案結構、伺服器設定、工具註冊模式、使用 Zod 的輸入驗證、錯誤處理，以及完整的工作範例。

---

## 快速參考

### 關鍵匯入
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import express from "express";
import { z } from "zod";
```

### 伺服器初始化
```typescript
const server = new McpServer({
  name: "service-mcp-server",
  version: "1.0.0"
});
```

### 工具註冊模式
```typescript
server.registerTool(
  "tool_name",
  {
    title: "Tool Display Name",
    description: "What the tool does",
    inputSchema: { param: z.string() },
    outputSchema: { result: z.string() }
  },
  async ({ param }) => {
    const output = { result: `Processed: ${param}` };
    return {
      content: [{ type: "text", text: JSON.stringify(output) }],
      structuredContent: output // 結構化資料的現代模式
    };
  }
);
```

---

## MCP TypeScript SDK

官方 MCP TypeScript SDK 提供：
- 用於伺服器初始化的 `McpServer` 類別
- 用於工具註冊的 `registerTool` 方法
- Zod schema 整合用於執行時輸入驗證
- 型別安全的工具處理器實作

**重要 - 僅使用現代 API：**
- **應使用**：`server.registerTool()`、`server.registerResource()`、`server.registerPrompt()`
- **不要使用**：舊的已棄用 API，如 `server.tool()`、`server.setRequestHandler(ListToolsRequestSchema, ...)` 或手動處理器註冊
- `register*` 方法提供更好的型別安全性、自動 schema 處理，且是推薦的方法

請參閱參考文件中的 MCP SDK 文件以獲取完整詳情。

## 伺服器命名慣例

Node/TypeScript MCP 伺服器必須遵循此命名模式：
- **格式**：`{service}-mcp-server`（小寫加連字號）
- **範例**：`github-mcp-server`、`jira-mcp-server`、`stripe-mcp-server`

名稱應該：
- 通用（不綁定特定功能）
- 能描述正在整合的服務/API
- 容易從任務描述推斷
- 不包含版本號或日期

## 專案結構

為 Node/TypeScript MCP 伺服器建立以下結構：

```
{service}-mcp-server/
├── package.json
├── tsconfig.json
├── README.md
├── src/
│   ├── index.ts          # 主要進入點，包含 McpServer 初始化
│   ├── types.ts          # TypeScript 型別定義和介面
│   ├── tools/            # 工具實作（每個領域一個檔案）
│   ├── services/         # API 客戶端和共用工具程式
│   ├── schemas/          # Zod 驗證 schema
│   └── constants.ts      # 共用常數（API_URL、CHARACTER_LIMIT 等）
└── dist/                 # 建置的 JavaScript 檔案（進入點：dist/index.js）
```

## 工具實作

### 工具命名

使用 snake_case 命名工具（例如 "search_users"、"create_project"、"get_channel_info"），並使用清晰、以動作為導向的名稱。

**避免命名衝突**：包含服務上下文以防止重疊：
- 使用 "slack_send_message" 而不是僅 "send_message"
- 使用 "github_create_issue" 而不是僅 "create_issue"
- 使用 "asana_list_tasks" 而不是僅 "list_tasks"

### 工具結構

工具使用 `registerTool` 方法註冊，有以下需求：
- 使用 Zod schema 進行執行時輸入驗證和型別安全
- `description` 欄位必須明確提供 - JSDoc 註解不會自動提取
- 明確提供 `title`、`description`、`inputSchema` 和 `annotations`
- `inputSchema` 必須是 Zod schema 物件（不是 JSON schema）
- 明確定義所有參數和回傳值的型別

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({
  name: "example-mcp",
  version: "1.0.0"
});

// 用於輸入驗證的 Zod schema
const UserSearchInputSchema = z.object({
  query: z.string()
    .min(2, "Query must be at least 2 characters")
    .max(200, "Query must not exceed 200 characters")
    .describe("Search string to match against names/emails"),
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Maximum results to return"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Number of results to skip for pagination"),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Output format: 'markdown' for human-readable or 'json' for machine-readable")
}).strict();

// 從 Zod schema 推導的型別定義
type UserSearchInput = z.infer<typeof UserSearchInputSchema>;

server.registerTool(
  "example_search_users",
  {
    title: "Search Example Users",
    description: `Search for users in the Example system by name, email, or team.

This tool searches across all user profiles in the Example platform, supporting partial matches and various search filters. It does NOT create or modify users, only searches existing ones.

Args:
  - query (string): Search string to match against names/emails
  - limit (number): Maximum results to return, between 1-100 (default: 20)
  - offset (number): Number of results to skip for pagination (default: 0)
  - response_format ('markdown' | 'json'): Output format (default: 'markdown')

Returns:
  For JSON format: Structured data with schema:
  {
    "total": number,           // Total number of matches found
    "count": number,           // Number of results in this response
    "offset": number,          // Current pagination offset
    "users": [
      {
        "id": string,          // User ID (e.g., "U123456789")
        "name": string,        // Full name (e.g., "John Doe")
        "email": string,       // Email address
        "team": string,        // Team name (optional)
        "active": boolean      // Whether user is active
      }
    ],
    "has_more": boolean,       // Whether more results are available
    "next_offset": number      // Offset for next page (if has_more is true)
  }

Examples:
  - Use when: "Find all marketing team members" -> params with query="team:marketing"
  - Use when: "Search for John's account" -> params with query="john"
  - Don't use when: You need to create a user (use example_create_user instead)

Error Handling:
  - Returns "Error: Rate limit exceeded" if too many requests (429 status)
  - Returns "No users found matching '<query>'" if search returns empty`,
    inputSchema: UserSearchInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: UserSearchInput) => {
    try {
      // 輸入驗證由 Zod schema 處理
      // 使用驗證過的參數發送 API 請求
      const data = await makeApiRequest<any>(
        "users/search",
        "GET",
        undefined,
        {
          q: params.query,
          limit: params.limit,
          offset: params.offset
        }
      );

      const users = data.users || [];
      const total = data.total || 0;

      if (!users.length) {
        return {
          content: [{
            type: "text",
            text: `No users found matching '${params.query}'`
          }]
        };
      }

      // 準備結構化輸出
      const output = {
        total,
        count: users.length,
        offset: params.offset,
        users: users.map((user: any) => ({
          id: user.id,
          name: user.name,
          email: user.email,
          ...(user.team ? { team: user.team } : {}),
          active: user.active ?? true
        })),
        has_more: total > params.offset + users.length,
        ...(total > params.offset + users.length ? {
          next_offset: params.offset + users.length
        } : {})
      };

      // 根據請求的格式格式化文字表示
      let textContent: string;
      if (params.response_format === ResponseFormat.MARKDOWN) {
        const lines = [`# User Search Results: '${params.query}'`, "",
          `Found ${total} users (showing ${users.length})`, ""];
        for (const user of users) {
          lines.push(`## ${user.name} (${user.id})`);
          lines.push(`- **Email**: ${user.email}`);
          if (user.team) lines.push(`- **Team**: ${user.team}`);
          lines.push("");
        }
        textContent = lines.join("\n");
      } else {
        textContent = JSON.stringify(output, null, 2);
      }

      return {
        content: [{ type: "text", text: textContent }],
        structuredContent: output // 結構化資料的現代模式
      };
    } catch (error) {
      return {
        content: [{
          type: "text",
          text: handleApiError(error)
        }]
      };
    }
  }
);
```

## 用於輸入驗證的 Zod Schema

Zod 提供執行時型別驗證：

```typescript
import { z } from "zod";

// 帶驗證的基本 schema
const CreateUserSchema = z.object({
  name: z.string()
    .min(1, "Name is required")
    .max(100, "Name must not exceed 100 characters"),
  email: z.string()
    .email("Invalid email format"),
  age: z.number()
    .int("Age must be a whole number")
    .min(0, "Age cannot be negative")
    .max(150, "Age cannot be greater than 150")
}).strict();  // 使用 .strict() 禁止額外欄位

// 列舉
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

const SearchSchema = z.object({
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Output format")
});

// 帶預設值的可選欄位
const PaginationSchema = z.object({
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Maximum results to return"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Number of results to skip")
});
```

## 回應格式選項

支援多種輸出格式以提供靈活性：

```typescript
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

const inputSchema = z.object({
  query: z.string(),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Output format: 'markdown' for human-readable or 'json' for machine-readable")
});
```

**Markdown 格式**：
- 使用標題、清單和格式以提高清晰度
- 將時間戳轉換為人類可讀格式
- 顯示名稱並在括號中附上 ID
- 省略冗長的中繼資料
- 邏輯地分組相關資訊

**JSON 格式**：
- 回傳完整的結構化資料，適合程式化處理
- 包含所有可用欄位和中繼資料
- 使用一致的欄位名稱和型別

## 分頁實作

對於列出資源的工具：

```typescript
const ListSchema = z.object({
  limit: z.number().int().min(1).max(100).default(20),
  offset: z.number().int().min(0).default(0)
});

async function listItems(params: z.infer<typeof ListSchema>) {
  const data = await apiRequest(params.limit, params.offset);

  const response = {
    total: data.total,
    count: data.items.length,
    offset: params.offset,
    items: data.items,
    has_more: data.total > params.offset + data.items.length,
    next_offset: data.total > params.offset + data.items.length
      ? params.offset + data.items.length
      : undefined
  };

  return JSON.stringify(response, null, 2);
}
```

## 字元限制和截斷

加入 CHARACTER_LIMIT 常數以防止回應過大：

```typescript
// 在 constants.ts 的模組層級
export const CHARACTER_LIMIT = 25000;  // 回應大小上限（字元數）

async function searchTool(params: SearchInput) {
  let result = generateResponse(data);

  // 檢查字元限制，如需要則截斷
  if (result.length > CHARACTER_LIMIT) {
    const truncatedData = data.slice(0, Math.max(1, data.length / 2));
    response.data = truncatedData;
    response.truncated = true;
    response.truncation_message =
      `Response truncated from ${data.length} to ${truncatedData.length} items. ` +
      `Use 'offset' parameter or add filters to see more results.`;
    result = JSON.stringify(response, null, 2);
  }

  return result;
}
```

## 錯誤處理

提供清晰、可操作的錯誤訊息：

```typescript
import axios, { AxiosError } from "axios";

function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404:
          return "Error: Resource not found. Please check the ID is correct.";
        case 403:
          return "Error: Permission denied. You don't have access to this resource.";
        case 429:
          return "Error: Rate limit exceeded. Please wait before making more requests.";
        default:
          return `Error: API request failed with status ${error.response.status}`;
      }
    } else if (error.code === "ECONNABORTED") {
      return "Error: Request timed out. Please try again.";
    }
  }
  return `Error: Unexpected error occurred: ${error instanceof Error ? error.message : String(error)}`;
}
```

## 共用工具程式

將常用功能提取為可重用函式：

```typescript
// 共用 API 請求函式
async function makeApiRequest<T>(
  endpoint: string,
  method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
  data?: any,
  params?: any
): Promise<T> {
  try {
    const response = await axios({
      method,
      url: `${API_BASE_URL}/${endpoint}`,
      data,
      params,
      timeout: 30000,
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    });
    return response.data;
  } catch (error) {
    throw error;
  }
}
```

## Async/Await 最佳實踐

對於網路請求和 I/O 操作始終使用 async/await：

```typescript
// 好：非同步網路請求
async function fetchData(resourceId: string): Promise<ResourceData> {
  const response = await axios.get(`${API_URL}/resource/${resourceId}`);
  return response.data;
}

// 不好：Promise 鏈
function fetchData(resourceId: string): Promise<ResourceData> {
  return axios.get(`${API_URL}/resource/${resourceId}`)
    .then(response => response.data);  // 較難閱讀和維護
}
```

## TypeScript 最佳實踐

1. **使用嚴格 TypeScript**：在 tsconfig.json 中啟用 strict 模式
2. **定義介面**：為所有資料結構建立清晰的介面定義
3. **避免 `any`**：使用適當的型別或 `unknown` 而不是 `any`
4. **Zod 用於執行時驗證**：使用 Zod schema 驗證外部資料
5. **型別守衛**：為複雜型別檢查建立型別守衛函式
6. **錯誤處理**：始終使用帶有適當錯誤型別檢查的 try-catch
7. **空值安全**：使用可選鏈結（`?.`）和空值合併（`??`）

```typescript
// 好：使用 Zod 和介面的型別安全
interface UserResponse {
  id: string;
  name: string;
  email: string;
  team?: string;
  active: boolean;
}

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  team: z.string().optional(),
  active: z.boolean()
});

type User = z.infer<typeof UserSchema>;

async function getUser(id: string): Promise<User> {
  const data = await apiCall(`/users/${id}`);
  return UserSchema.parse(data);  // 執行時驗證
}

// 不好：使用 any
async function getUser(id: string): Promise<any> {
  return await apiCall(`/users/${id}`);  // 沒有型別安全
}
```

## 套件配置

### package.json

```json
{
  "name": "{service}-mcp-server",
  "version": "1.0.0",
  "description": "MCP server for {Service} API integration",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "clean": "rm -rf dist"
  },
  "engines": {
    "node": ">=18"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.6.1",
    "axios": "^1.7.9",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/node": "^22.10.0",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 完整範例

```typescript
#!/usr/bin/env node
/**
 * Example Service 的 MCP 伺服器。
 *
 * 此伺服器提供與 Example API 互動的工具，包括使用者搜尋、
 * 專案管理和資料匯出功能。
 */

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import axios, { AxiosError } from "axios";

// 常數
const API_BASE_URL = "https://api.example.com/v1";
const CHARACTER_LIMIT = 25000;

// 列舉
enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

// Zod schema
const UserSearchInputSchema = z.object({
  query: z.string()
    .min(2, "Query must be at least 2 characters")
    .max(200, "Query must not exceed 200 characters")
    .describe("Search string to match against names/emails"),
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Maximum results to return"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Number of results to skip for pagination"),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Output format: 'markdown' for human-readable or 'json' for machine-readable")
}).strict();

type UserSearchInput = z.infer<typeof UserSearchInputSchema>;

// 共用工具函式
async function makeApiRequest<T>(
  endpoint: string,
  method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
  data?: any,
  params?: any
): Promise<T> {
  try {
    const response = await axios({
      method,
      url: `${API_BASE_URL}/${endpoint}`,
      data,
      params,
      timeout: 30000,
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    });
    return response.data;
  } catch (error) {
    throw error;
  }
}

function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404:
          return "Error: Resource not found. Please check the ID is correct.";
        case 403:
          return "Error: Permission denied. You don't have access to this resource.";
        case 429:
          return "Error: Rate limit exceeded. Please wait before making more requests.";
        default:
          return `Error: API request failed with status ${error.response.status}`;
      }
    } else if (error.code === "ECONNABORTED") {
      return "Error: Request timed out. Please try again.";
    }
  }
  return `Error: Unexpected error occurred: ${error instanceof Error ? error.message : String(error)}`;
}

// 建立 MCP 伺服器實例
const server = new McpServer({
  name: "example-mcp",
  version: "1.0.0"
});

// 註冊工具
server.registerTool(
  "example_search_users",
  {
    title: "Search Example Users",
    description: `[如上所示的完整描述]`,
    inputSchema: UserSearchInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: UserSearchInput) => {
    // 如上所示的實作
  }
);

// 主函式
// 用於 stdio（本機）：
async function runStdio() {
  if (!process.env.EXAMPLE_API_KEY) {
    console.error("ERROR: EXAMPLE_API_KEY environment variable is required");
    process.exit(1);
  }

  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP server running via stdio");
}

// 用於 streamable HTTP（遠端）：
async function runHTTP() {
  if (!process.env.EXAMPLE_API_KEY) {
    console.error("ERROR: EXAMPLE_API_KEY environment variable is required");
    process.exit(1);
  }

  const app = express();
  app.use(express.json());

  app.post('/mcp', async (req, res) => {
    const transport = new StreamableHTTPServerTransport({
      sessionIdGenerator: undefined,
      enableJsonResponse: true
    });
    res.on('close', () => transport.close());
    await server.connect(transport);
    await transport.handleRequest(req, res, req.body);
  });

  const port = parseInt(process.env.PORT || '3000');
  app.listen(port, () => {
    console.error(`MCP server running on http://localhost:${port}/mcp`);
  });
}

// 根據環境選擇傳輸
const transport = process.env.TRANSPORT || 'stdio';
if (transport === 'http') {
  runHTTP().catch(error => {
    console.error("Server error:", error);
    process.exit(1);
  });
} else {
  runStdio().catch(error => {
    console.error("Server error:", error);
    process.exit(1);
  });
}
```

---

## 進階 MCP 功能

### 資源註冊

將資料公開為資源以實現高效、基於 URI 的存取：

```typescript
import { ResourceTemplate } from "@modelcontextprotocol/sdk/types.js";

// 使用 URI 範本註冊資源
server.registerResource(
  {
    uri: "file://documents/{name}",
    name: "Document Resource",
    description: "Access documents by name",
    mimeType: "text/plain"
  },
  async (uri: string) => {
    // 從 URI 提取參數
    const match = uri.match(/^file:\/\/documents\/(.+)$/);
    if (!match) {
      throw new Error("Invalid URI format");
    }

    const documentName = match[1];
    const content = await loadDocument(documentName);

    return {
      contents: [{
        uri,
        mimeType: "text/plain",
        text: content
      }]
    };
  }
);

// 動態列出可用資源
server.registerResourceList(async () => {
  const documents = await getAvailableDocuments();
  return {
    resources: documents.map(doc => ({
      uri: `file://documents/${doc.name}`,
      name: doc.name,
      mimeType: "text/plain",
      description: doc.description
    }))
  };
});
```

**何時使用資源 vs 工具：**
- **資源**：用於具有簡單基於 URI 參數的資料存取
- **工具**：用於需要驗證和業務邏輯的複雜操作
- **資源**：當資料相對靜態或基於範本時
- **工具**：當操作有副作用或複雜工作流程時

### 傳輸選項

TypeScript SDK 支援兩種主要傳輸機制：

#### Streamable HTTP（推薦用於遠端伺服器）

```typescript
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const app = express();
app.use(express.json());

app.post('/mcp', async (req, res) => {
  // 為每個請求建立新的傳輸（無狀態，防止請求 ID 衝突）
  const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: undefined,
    enableJsonResponse: true
  });

  res.on('close', () => transport.close());

  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

#### stdio（用於本機整合）

```typescript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await server.connect(transport);
```

**傳輸選擇：**
- **Streamable HTTP**：Web 服務、遠端存取、多客戶端
- **stdio**：命令列工具、本機開發、子程序整合

### 通知支援

在伺服器狀態變更時通知客戶端：

```typescript
// 當工具清單變更時通知
server.notification({
  method: "notifications/tools/list_changed"
});

// 當資源變更時通知
server.notification({
  method: "notifications/resources/list_changed"
});
```

謹慎使用通知 - 僅在伺服器功能真正變更時使用。

---

## 程式碼最佳實踐

### 程式碼可組合性和可重用性

您的實作必須優先考慮可組合性和程式碼重用：

1. **提取常用功能**：
   - 為跨多個工具使用的操作建立可重用的輔助函式
   - 建構共用 API 客戶端用於 HTTP 請求，而不是複製程式碼
   - 將錯誤處理邏輯集中在工具函式中
   - 將業務邏輯提取到可以組合的專用函式中
   - 提取共用的 markdown 或 JSON 欄位選擇和格式化功能

2. **避免重複**：
   - 永不在工具之間複製貼上相似的程式碼
   - 如果您發現自己寫了兩次相似的邏輯，將其提取為函式
   - 分頁、過濾、欄位選擇和格式化等常用操作應該共用
   - 認證/授權邏輯應該集中化

## 建置和執行

在執行前始終建置您的 TypeScript 程式碼：

```bash
# 建置專案
npm run build

# 執行伺服器
npm start

# 帶自動重載的開發模式
npm run dev
```

在認為實作完成前，始終確保 `npm run build` 成功完成。

## 品質檢查清單

在完成您的 Node/TypeScript MCP 伺服器實作前，確保：

### 策略設計
- [ ] 工具支援完整工作流程，而不僅僅是 API 端點包裝
- [ ] 工具名稱反映自然的任務細分
- [ ] 回應格式最佳化代理上下文效率
- [ ] 適當使用人類可讀識別碼
- [ ] 錯誤訊息引導代理正確使用

### 實作品質
- [ ] 專注實作：實作最重要和最有價值的工具
- [ ] 所有工具使用 `registerTool` 並帶有完整配置
- [ ] 所有工具包含 `title`、`description`、`inputSchema` 和 `annotations`
- [ ] 正確設定註解（readOnlyHint、destructiveHint、idempotentHint、openWorldHint）
- [ ] 所有工具使用 Zod schema 進行執行時輸入驗證，並帶有 `.strict()` 強制
- [ ] 所有 Zod schema 有適當的約束和描述性錯誤訊息
- [ ] 所有工具有包含明確輸入/輸出型別的全面描述
- [ ] 描述包含回傳值範例和完整 schema 文件
- [ ] 錯誤訊息清晰、可操作且具教育性

### TypeScript 品質
- [ ] 為所有資料結構定義 TypeScript 介面
- [ ] 在 tsconfig.json 中啟用嚴格 TypeScript
- [ ] 不使用 `any` 型別 - 改用 `unknown` 或適當型別
- [ ] 所有非同步函式有明確的 Promise<T> 回傳型別
- [ ] 錯誤處理使用適當的型別守衛（例如 `axios.isAxiosError`、`z.ZodError`）

### 進階功能（視情況適用）
- [ ] 為適當的資料端點註冊資源
- [ ] 配置適當的傳輸（stdio 或 streamable HTTP）
- [ ] 為動態伺服器功能實作通知
- [ ] 使用 SDK 介面實現型別安全

### 專案配置
- [ ] Package.json 包含所有必要的依賴項
- [ ] 建置腳本在 dist/ 目錄產生可運作的 JavaScript
- [ ] 主要進入點正確配置為 dist/index.js
- [ ] 伺服器名稱遵循格式：`{service}-mcp-server`
- [ ] tsconfig.json 正確配置為嚴格模式

### 程式碼品質
- [ ] 在適用處正確實作分頁
- [ ] 大型回應檢查 CHARACTER_LIMIT 常數並帶有清晰訊息截斷
- [ ] 為可能的大型結果集提供過濾選項
- [ ] 所有網路操作優雅處理逾時和連線錯誤
- [ ] 常用功能提取為可重用函式
- [ ] 類似操作的回傳型別一致

### 測試和建置
- [ ] `npm run build` 成功完成且無錯誤
- [ ] dist/index.js 已建立且可執行
- [ ] 伺服器可執行：`node dist/index.js --help`
- [ ] 所有匯入正確解析
- [ ] 範例工具呼叫如預期運作
