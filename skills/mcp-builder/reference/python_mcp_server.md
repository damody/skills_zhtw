# Python MCP 伺服器實作指南

## 概述

本文件提供使用 MCP Python SDK 實作 MCP 伺服器的 Python 特定最佳實踐和範例。涵蓋伺服器設定、工具註冊模式、使用 Pydantic 的輸入驗證、錯誤處理，以及完整的工作範例。

---

## 快速參考

### 關鍵匯入
```python
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
```

### 伺服器初始化
```python
mcp = FastMCP("service_mcp")
```

### 工具註冊模式
```python
@mcp.tool(name="tool_name", annotations={...})
async def tool_function(params: InputModel) -> str:
    # 實作
    pass
```

---

## MCP Python SDK 和 FastMCP

官方 MCP Python SDK 提供 FastMCP，這是一個用於建構 MCP 伺服器的高階框架。它提供：
- 從函式簽章和 docstring 自動產生描述和 inputSchema
- Pydantic 模型整合用於輸入驗證
- 使用 `@mcp.tool` 的裝飾器式工具註冊

**完整 SDK 文件，請使用 WebFetch 載入：**
`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`

## 伺服器命名慣例

Python MCP 伺服器必須遵循此命名模式：
- **格式**：`{service}_mcp`（小寫加底線）
- **範例**：`github_mcp`、`jira_mcp`、`stripe_mcp`

名稱應該：
- 通用（不綁定特定功能）
- 能描述正在整合的服務/API
- 容易從任務描述推斷
- 不包含版本號或日期

## 工具實作

### 工具命名

使用 snake_case 命名工具（例如 "search_users"、"create_project"、"get_channel_info"），並使用清晰、以動作為導向的名稱。

**避免命名衝突**：包含服務上下文以防止重疊：
- 使用 "slack_send_message" 而不是僅 "send_message"
- 使用 "github_create_issue" 而不是僅 "create_issue"
- 使用 "asana_list_tasks" 而不是僅 "list_tasks"

### 使用 FastMCP 的工具結構

工具使用 `@mcp.tool` 裝飾器定義，並使用 Pydantic 模型進行輸入驗證：

```python
from pydantic import BaseModel, Field, ConfigDict
from mcp.server.fastmcp import FastMCP

# 初始化 MCP 伺服器
mcp = FastMCP("example_mcp")

# 定義用於輸入驗證的 Pydantic 模型
class ServiceToolInput(BaseModel):
    '''服務工具操作的輸入模型。'''
    model_config = ConfigDict(
        str_strip_whitespace=True,  # 自動去除字串的空白
        validate_assignment=True,    # 在賦值時驗證
        extra='forbid'              # 禁止額外欄位
    )

    param1: str = Field(..., description="第一個參數描述（例如 'user123'、'project-abc'）", min_length=1, max_length=100)
    param2: Optional[int] = Field(default=None, description="帶約束的可選整數參數", ge=0, le=1000)
    tags: Optional[List[str]] = Field(default_factory=list, description="要套用的標籤清單", max_items=10)

@mcp.tool(
    name="service_tool_name",
    annotations={
        "title": "Human-Readable Tool Title",
        "readOnlyHint": True,     # 工具不修改環境
        "destructiveHint": False,  # 工具不執行破壞性操作
        "idempotentHint": True,    # 重複呼叫不會產生額外效果
        "openWorldHint": False     # 工具不與外部實體互動
    }
)
async def service_tool_name(params: ServiceToolInput) -> str:
    '''工具描述會自動成為 'description' 欄位。

    此工具對服務執行特定操作。它在處理前使用
    ServiceToolInput Pydantic 模型驗證所有輸入。

    Args:
        params (ServiceToolInput): 驗證過的輸入參數，包含：
            - param1 (str): 第一個參數描述
            - param2 (Optional[int]): 帶預設值的可選參數
            - tags (Optional[List[str]]): 標籤清單

    Returns:
        str: 包含操作結果的 JSON 格式化回應
    '''
    # 此處為實作
    pass
```

## Pydantic v2 關鍵功能

- 使用 `model_config` 而不是巢狀的 `Config` 類別
- 使用 `field_validator` 而不是已棄用的 `validator`
- 使用 `model_dump()` 而不是已棄用的 `dict()`
- 驗證器需要 `@classmethod` 裝飾器
- 驗證器方法需要型別提示

```python
from pydantic import BaseModel, Field, field_validator, ConfigDict

class CreateUserInput(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True
    )

    name: str = Field(..., description="使用者全名", min_length=1, max_length=100)
    email: str = Field(..., description="使用者電子郵件地址", pattern=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    age: int = Field(..., description="使用者年齡", ge=0, le=150)

    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Email cannot be empty")
        return v.lower()
```

## 回應格式選項

支援多種輸出格式以提供靈活性：

```python
from enum import Enum

class ResponseFormat(str, Enum):
    '''工具回應的輸出格式。'''
    MARKDOWN = "markdown"
    JSON = "json"

class UserSearchInput(BaseModel):
    query: str = Field(..., description="搜尋查詢")
    response_format: ResponseFormat = Field(
        default=ResponseFormat.MARKDOWN,
        description="輸出格式：'markdown' 為人類可讀或 'json' 為機器可讀"
    )
```

**Markdown 格式**：
- 使用標題、清單和格式以提高清晰度
- 將時間戳轉換為人類可讀格式（例如 "2024-01-15 10:30:00 UTC" 而不是 epoch）
- 顯示名稱並在括號中附上 ID（例如 "@john.doe (U123456)"）
- 省略冗長的中繼資料（例如只顯示一個個人頭像 URL，而不是所有大小）
- 邏輯地分組相關資訊

**JSON 格式**：
- 回傳完整的結構化資料，適合程式化處理
- 包含所有可用欄位和中繼資料
- 使用一致的欄位名稱和型別

## 分頁實作

對於列出資源的工具：

```python
class ListInput(BaseModel):
    limit: Optional[int] = Field(default=20, description="回傳的最大結果數", ge=1, le=100)
    offset: Optional[int] = Field(default=0, description="分頁時要跳過的結果數", ge=0)

async def list_items(params: ListInput) -> str:
    # 發送帶分頁的 API 請求
    data = await api_request(limit=params.limit, offset=params.offset)

    # 回傳分頁資訊
    response = {
        "total": data["total"],
        "count": len(data["items"]),
        "offset": params.offset,
        "items": data["items"],
        "has_more": data["total"] > params.offset + len(data["items"]),
        "next_offset": params.offset + len(data["items"]) if data["total"] > params.offset + len(data["items"]) else None
    }
    return json.dumps(response, indent=2)
```

## 錯誤處理

提供清晰、可操作的錯誤訊息：

```python
def _handle_api_error(e: Exception) -> str:
    '''所有工具的一致錯誤格式化。'''
    if isinstance(e, httpx.HTTPStatusError):
        if e.response.status_code == 404:
            return "Error: Resource not found. Please check the ID is correct."
        elif e.response.status_code == 403:
            return "Error: Permission denied. You don't have access to this resource."
        elif e.response.status_code == 429:
            return "Error: Rate limit exceeded. Please wait before making more requests."
        return f"Error: API request failed with status {e.response.status_code}"
    elif isinstance(e, httpx.TimeoutException):
        return "Error: Request timed out. Please try again."
    return f"Error: Unexpected error occurred: {type(e).__name__}"
```

## 共用工具程式

將常用功能提取為可重用函式：

```python
# 共用 API 請求函式
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    '''所有 API 呼叫的可重用函式。'''
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()
```

## Async/Await 最佳實踐

對於網路請求和 I/O 操作始終使用 async/await：

```python
# 好：非同步網路請求
async def fetch_data(resource_id: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"{API_URL}/resource/{resource_id}")
        response.raise_for_status()
        return response.json()

# 不好：同步請求
def fetch_data(resource_id: str) -> dict:
    response = requests.get(f"{API_URL}/resource/{resource_id}")  # 阻塞
    return response.json()
```

## 型別提示

全程使用型別提示：

```python
from typing import Optional, List, Dict, Any

async def get_user(user_id: str) -> Dict[str, Any]:
    data = await fetch_user(user_id)
    return {"id": data["id"], "name": data["name"]}
```

## 工具 Docstring

每個工具必須有包含明確型別資訊的完整 docstring：

```python
async def search_users(params: UserSearchInput) -> str:
    '''
    按名稱、電子郵件或團隊搜尋 Example 系統中的使用者。

    此工具搜尋 Example 平台中的所有使用者個人資料，
    支援部分匹配和各種搜尋過濾器。它不會
    建立或修改使用者，只搜尋現有的。

    Args:
        params (UserSearchInput): 驗證過的輸入參數，包含：
            - query (str): 用於匹配名稱/電子郵件的搜尋字串（例如 "john"、"@example.com"、"team:marketing"）
            - limit (Optional[int]): 回傳的最大結果數，介於 1-100 之間（預設：20）
            - offset (Optional[int]): 分頁時要跳過的結果數（預設：0）

    Returns:
        str: 包含搜尋結果的 JSON 格式化字串，具有以下 schema：

        成功回應：
        {
            "total": int,           # 找到的總匹配數
            "count": int,           # 此回應中的結果數
            "offset": int,          # 目前的分頁偏移量
            "users": [
                {
                    "id": str,      # 使用者 ID（例如 "U123456789"）
                    "name": str,    # 全名（例如 "John Doe"）
                    "email": str,   # 電子郵件地址（例如 "john@example.com"）
                    "team": str     # 團隊名稱（例如 "Marketing"）- 可選
                }
            ]
        }

        錯誤回應：
        "Error: <error message>" 或 "No users found matching '<query>'"

    Examples:
        - 使用時機："Find all marketing team members" -> 參數 query="team:marketing"
        - 使用時機："Search for John's account" -> 參數 query="john"
        - 不要使用時機：您需要建立使用者（請改用 example_create_user）
        - 不要使用時機：您有使用者 ID 且需要完整詳情（請改用 example_get_user）

    Error Handling:
        - 輸入驗證錯誤由 Pydantic 模型處理
        - 如果請求過多（429 狀態）則回傳 "Error: Rate limit exceeded"
        - 如果 API 金鑰無效（401 狀態）則回傳 "Error: Invalid API authentication"
        - 回傳格式化的結果清單或 "No users found matching 'query'"
    '''
```

## 完整範例

請參見以下完整的 Python MCP 伺服器範例：

```python
#!/usr/bin/env python3
'''
Example Service 的 MCP 伺服器。

此伺服器提供與 Example API 互動的工具，包括使用者搜尋、
專案管理和資料匯出功能。
'''

from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
from pydantic import BaseModel, Field, field_validator, ConfigDict
from mcp.server.fastmcp import FastMCP

# 初始化 MCP 伺服器
mcp = FastMCP("example_mcp")

# 常數
API_BASE_URL = "https://api.example.com/v1"

# 列舉
class ResponseFormat(str, Enum):
    '''工具回應的輸出格式。'''
    MARKDOWN = "markdown"
    JSON = "json"

# 用於輸入驗證的 Pydantic 模型
class UserSearchInput(BaseModel):
    '''使用者搜尋操作的輸入模型。'''
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True
    )

    query: str = Field(..., description="用於匹配名稱/電子郵件的搜尋字串", min_length=2, max_length=200)
    limit: Optional[int] = Field(default=20, description="回傳的最大結果數", ge=1, le=100)
    offset: Optional[int] = Field(default=0, description="分頁時要跳過的結果數", ge=0)
    response_format: ResponseFormat = Field(default=ResponseFormat.MARKDOWN, description="輸出格式")

    @field_validator('query')
    @classmethod
    def validate_query(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Query cannot be empty or whitespace only")
        return v.strip()

# 共用工具函式
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    '''所有 API 呼叫的可重用函式。'''
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()

def _handle_api_error(e: Exception) -> str:
    '''所有工具的一致錯誤格式化。'''
    if isinstance(e, httpx.HTTPStatusError):
        if e.response.status_code == 404:
            return "Error: Resource not found. Please check the ID is correct."
        elif e.response.status_code == 403:
            return "Error: Permission denied. You don't have access to this resource."
        elif e.response.status_code == 429:
            return "Error: Rate limit exceeded. Please wait before making more requests."
        return f"Error: API request failed with status {e.response.status_code}"
    elif isinstance(e, httpx.TimeoutException):
        return "Error: Request timed out. Please try again."
    return f"Error: Unexpected error occurred: {type(e).__name__}"

# 工具定義
@mcp.tool(
    name="example_search_users",
    annotations={
        "title": "Search Example Users",
        "readOnlyHint": True,
        "destructiveHint": False,
        "idempotentHint": True,
        "openWorldHint": True
    }
)
async def example_search_users(params: UserSearchInput) -> str:
    '''按名稱、電子郵件或團隊搜尋 Example 系統中的使用者。

    [如上所示的完整 docstring]
    '''
    try:
        # 使用驗證過的參數發送 API 請求
        data = await _make_api_request(
            "users/search",
            params={
                "q": params.query,
                "limit": params.limit,
                "offset": params.offset
            }
        )

        users = data.get("users", [])
        total = data.get("total", 0)

        if not users:
            return f"No users found matching '{params.query}'"

        # 根據請求的格式格式化回應
        if params.response_format == ResponseFormat.MARKDOWN:
            lines = [f"# User Search Results: '{params.query}'", ""]
            lines.append(f"Found {total} users (showing {len(users)})")
            lines.append("")

            for user in users:
                lines.append(f"## {user['name']} ({user['id']})")
                lines.append(f"- **Email**: {user['email']}")
                if user.get('team'):
                    lines.append(f"- **Team**: {user['team']}")
                lines.append("")

            return "\n".join(lines)

        else:
            # 機器可讀的 JSON 格式
            import json
            response = {
                "total": total,
                "count": len(users),
                "offset": params.offset,
                "users": users
            }
            return json.dumps(response, indent=2)

    except Exception as e:
        return _handle_api_error(e)

if __name__ == "__main__":
    mcp.run()
```

---

## 進階 FastMCP 功能

### Context 參數注入

FastMCP 可以自動將 `Context` 參數注入到工具中，用於進階功能如記錄、進度報告、資源讀取和使用者互動：

```python
from mcp.server.fastmcp import FastMCP, Context

mcp = FastMCP("example_mcp")

@mcp.tool()
async def advanced_search(query: str, ctx: Context) -> str:
    '''帶有上下文存取的進階工具，用於記錄和進度報告。'''

    # 報告長時間操作的進度
    await ctx.report_progress(0.25, "Starting search...")

    # 記錄除錯資訊
    await ctx.log_info("Processing query", {"query": query, "timestamp": datetime.now()})

    # 執行搜尋
    results = await search_api(query)
    await ctx.report_progress(0.75, "Formatting results...")

    # 存取伺服器配置
    server_name = ctx.fastmcp.name

    return format_results(results)

@mcp.tool()
async def interactive_tool(resource_id: str, ctx: Context) -> str:
    '''可以向使用者請求額外輸入的工具。'''

    # 在需要時請求敏感資訊
    api_key = await ctx.elicit(
        prompt="Please provide your API key:",
        input_type="password"
    )

    # 使用提供的金鑰
    return await api_call(resource_id, api_key)
```

**Context 功能：**
- `ctx.report_progress(progress, message)` - 報告長時間操作的進度
- `ctx.log_info(message, data)` / `ctx.log_error()` / `ctx.log_debug()` - 記錄
- `ctx.elicit(prompt, input_type)` - 向使用者請求輸入
- `ctx.fastmcp.name` - 存取伺服器配置
- `ctx.read_resource(uri)` - 讀取 MCP 資源

### 資源註冊

將資料公開為資源，用於高效、基於範本的存取：

```python
@mcp.resource("file://documents/{name}")
async def get_document(name: str) -> str:
    '''將文件公開為 MCP 資源。

    資源適用於不需要複雜參數的靜態或半靜態資料。
    它們使用 URI 範本進行靈活存取。
    '''
    document_path = f"./docs/{name}"
    with open(document_path, "r") as f:
        return f.read()

@mcp.resource("config://settings/{key}")
async def get_setting(key: str, ctx: Context) -> str:
    '''將配置公開為帶上下文的資源。'''
    settings = await load_settings()
    return json.dumps(settings.get(key, {}))
```

**何時使用資源 vs 工具：**
- **資源**：用於具有簡單參數的資料存取（URI 範本）
- **工具**：用於具有驗證和業務邏輯的複雜操作

### 結構化輸出型別

FastMCP 支援字串以外的多種回傳型別：

```python
from typing import TypedDict
from dataclasses import dataclass
from pydantic import BaseModel

# TypedDict 用於結構化回傳
class UserData(TypedDict):
    id: str
    name: str
    email: str

@mcp.tool()
async def get_user_typed(user_id: str) -> UserData:
    '''回傳結構化資料 - FastMCP 處理序列化。'''
    return {"id": user_id, "name": "John Doe", "email": "john@example.com"}

# Pydantic 模型用於複雜驗證
class DetailedUser(BaseModel):
    id: str
    name: str
    email: str
    created_at: datetime
    metadata: Dict[str, Any]

@mcp.tool()
async def get_user_detailed(user_id: str) -> DetailedUser:
    '''回傳 Pydantic 模型 - 自動產生 schema。'''
    user = await fetch_user(user_id)
    return DetailedUser(**user)
```

### 生命週期管理

初始化跨請求持久存在的資源：

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def app_lifespan():
    '''管理伺服器生命週期內存在的資源。'''
    # 初始化連線、載入配置等
    db = await connect_to_database()
    config = load_configuration()

    # 讓所有工具可用
    yield {"db": db, "config": config}

    # 關閉時清理
    await db.close()

mcp = FastMCP("example_mcp", lifespan=app_lifespan)

@mcp.tool()
async def query_data(query: str, ctx: Context) -> str:
    '''透過上下文存取生命週期資源。'''
    db = ctx.request_context.lifespan_state["db"]
    results = await db.query(query)
    return format_results(results)
```

### 傳輸選項

FastMCP 支援兩種主要傳輸機制：

```python
# stdio 傳輸（用於本機工具）- 預設
if __name__ == "__main__":
    mcp.run()

# Streamable HTTP 傳輸（用於遠端伺服器）
if __name__ == "__main__":
    mcp.run(transport="streamable_http", port=8000)
```

**傳輸選擇：**
- **stdio**：命令列工具、本機整合、子程序執行
- **Streamable HTTP**：Web 服務、遠端存取、多客戶端

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

### Python 特定最佳實踐

1. **使用型別提示**：始終為函式參數和回傳值包含型別註解
2. **Pydantic 模型**：為所有輸入驗證定義清晰的 Pydantic 模型
3. **避免手動驗證**：讓 Pydantic 使用約束處理輸入驗證
4. **適當的匯入**：分組匯入（標準函式庫、第三方、本機）
5. **錯誤處理**：使用特定的例外類型（httpx.HTTPStatusError，而不是通用的 Exception）
6. **非同步上下文管理器**：對需要清理的資源使用 `async with`
7. **常數**：在模組層級使用大寫定義常數

## 品質檢查清單

在完成您的 Python MCP 伺服器實作前，確保：

### 策略設計
- [ ] 工具支援完整工作流程，而不僅僅是 API 端點包裝
- [ ] 工具名稱反映自然的任務細分
- [ ] 回應格式最佳化代理上下文效率
- [ ] 適當使用人類可讀識別碼
- [ ] 錯誤訊息引導代理正確使用

### 實作品質
- [ ] 專注實作：實作最重要和最有價值的工具
- [ ] 所有工具有描述性名稱和文件
- [ ] 類似操作的回傳型別一致
- [ ] 為所有外部呼叫實作錯誤處理
- [ ] 伺服器名稱遵循格式：`{service}_mcp`
- [ ] 所有網路操作使用 async/await
- [ ] 常用功能提取為可重用函式
- [ ] 錯誤訊息清晰、可操作且具教育性
- [ ] 輸出經過適當驗證和格式化

### 工具配置
- [ ] 所有工具在裝飾器中實作 'name' 和 'annotations'
- [ ] 正確設定註解（readOnlyHint、destructiveHint、idempotentHint、openWorldHint）
- [ ] 所有工具使用 Pydantic BaseModel 進行輸入驗證，並帶有 Field() 定義
- [ ] 所有 Pydantic Field 有明確的型別和帶約束的描述
- [ ] 所有工具有包含明確輸入/輸出型別的完整 docstring
- [ ] Docstring 包含 dict/JSON 回傳的完整 schema 結構
- [ ] Pydantic 模型處理輸入驗證（不需要手動驗證）

### 進階功能（視情況適用）
- [ ] 使用 Context 注入進行記錄、進度或徵求輸入
- [ ] 為適當的資料端點註冊資源
- [ ] 為持久連線實作生命週期管理
- [ ] 使用結構化輸出型別（TypedDict、Pydantic 模型）
- [ ] 配置適當的傳輸（stdio 或 streamable HTTP）

### 程式碼品質
- [ ] 檔案包含適當的匯入，包括 Pydantic 匯入
- [ ] 在適用處正確實作分頁
- [ ] 為可能的大型結果集提供過濾選項
- [ ] 所有非同步函式使用 `async def` 正確定義
- [ ] HTTP 客戶端使用遵循帶適當上下文管理器的非同步模式
- [ ] 全程使用型別提示
- [ ] 常數在模組層級使用大寫定義

### 測試
- [ ] 伺服器成功執行：`python your_server.py --help`
- [ ] 所有匯入正確解析
- [ ] 範例工具呼叫如預期運作
- [ ] 錯誤場景優雅處理
