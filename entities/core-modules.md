# 核心模块 (Core Modules)

## 📍 概述

CCB 的核心模块是项目的"大脑"，负责处理 AI 交互的主要逻辑。

## 🔑 模块列表

### 1. QueryEngine.ts

**文件**: `src/QueryEngine.ts`
**行数**: ~1,400 行
**职责**: 高级编排器

**主要功能**:
- 会话管理
- 压缩决策
- 文件历史快照
- Token 预算管理
- 消息归因

**关键方法**:

```typescript
class QueryEngine {
  // 主查询方法
  async query(userInput: string): Promise<QueryResult>

  // 构建请求
  private async buildRequest(): Promise<RequestParams>

  // 处理响应
  private async processResponse(): Promise<void>

  // 检查压缩
  private shouldCompact(): boolean

  // 更新文件历史
  private updateFileHistory(): void
}
```

### 2. query.ts

**文件**: `src/query.ts`
**行数**: ~2,400 行
**职责**: API 查询执行

**主要功能**:
- 构建请求参数
- 调用 Claude API
- 处理流式响应
- 执行工具调用
- 错误重试

**核心函数**:

```typescript
async function query(
  params: QueryParams
): Promise<QueryResult> {
  // 1. 构建请求
  const request = buildRequest(params)

  // 2. 调用 API
  const stream = await api.messages.create(request)

  // 3. 处理流
  for await (const event of stream) {
    switch (event.type) {
      case 'content_block_delta':
        handleDelta(event.delta)
        break
      case 'tool_use':
        await handleToolUse(event.input)
        break
    }
  }

  // 4. 返回结果
  return buildResult()
}
```

### 3. Tool.ts

**文件**: `src/Tool.ts`
**行数**: ~900 行
**职责**: 工具接口定义

**主要类型**:

```typescript
interface Tool {
  name: string
  description: string
  inputSchema: JSONObject
  execute(params, context): Promise<string>
  detailedDescription?: string
  permissionLevel?: 'read' | 'write' | 'execute'
}

interface ToolUseContext {
  cwd: string
  projectRoot: string
  sessionId: string
  // ...
}
```

**工具函数**:

```typescript
// 查找工具
function findToolByName(name: string, tools: Tools): Tool | undefined

// 匹配工具名
function toolMatchesName(tool: Tool, pattern: string): boolean
```

### 4. tools.ts

**文件**: `src/tools.ts`
**行数**: ~600 行
**职责**: 工具注册表

**主要内容**:

```typescript
export async function getTools(): Promise<Tools> {
  return {
    // 核心工具
    read_file: FileReadTool,
    edit_file: FileEditTool,
    bash: BashTool,

    // Agent 工具
    agent: AgentTool,
    task_create: TaskCreateTool,

    // Web 工具
    web_search: WebSearchTool,
    web_fetch: WebFetchTool,

    // MCP 工具
    mcp: MCPTool,

    // ... 60+ 工具
  }
}
```

### 5. commands.ts

**文件**: `src/commands.ts`
**行数**: ~900 行
**职责**: Slash 命令定义

**命令类型**:

```typescript
interface Command {
  name: string
  description: string
  handler: (args) => Promise<void>
  permissionLevel?: PermissionLevel
}
```

**命令列表**:

| 命令 | 功能 |
|------|------|
| `/help` | 显示帮助 |
| `/clear` | 清空对话 |
| `/exit` | 退出程序 |
| `/login` | 配置登录 |
| `/mcp` | MCP 管理 |
| `/goal` | 目标驱动 |
| `/plan` | 规划模式 |
| `/skills` | 技能系统 |

## 📊 模块依赖

```
QueryEngine.ts
    ├── query.ts
    │   ├── services/api/claude.ts
    │   ├── Tool.ts
    │   └── tools.ts
    ├── services/compact/
    ├── bootstrap/state.ts
    └── utils/fileHistory.ts
```

## 🔗 相关文档

- [核心循环](../concepts/core-loop.md)
- [工具系统](../concepts/tool-system.md)
- [状态管理](../concepts/state-management.md)

---

*最后更新: 2026-06-23*
