# 服务层 (Services)

## 📍 概述

CCB 的服务层包含各种功能模块，从 API 调用到监控追踪。

## 🔑 服务列表

### 1. API 服务 (`services/api/`)

**claude.ts** - 核心客户端

```typescript
export async function createMessage(params: {
  messages: Message[]
  tools: Tools
  systemPrompt: string
  model: string
}): Promise<AsyncStream> {
  // 构建请求
  // 调用 Anthropic SDK
  // 返回流
}
```

**功能**:
- 构建请求参数
- 处理流式响应
- 错误重试
- Rate limit 处理

### 2. MCP 服务 (`services/mcp/`)

**client.ts** - MCP 客户端

```typescript
export class MCPClient {
  async connect(serverConfig: ServerConfig): Promise<void>
  async callTool(name: string, params: unknown): Promise<ToolResult>
  async listTools(): Promise<Tool[]>
  disconnect(): void
}
```

**功能**:
- 连接 MCP 服务器
- 调用 MCP 工具
- 管理连接生命周期

### 3. 压缩服务 (`services/compact/`)

**compact.ts** - 上下文压缩

```typescript
export async function compactMessages(params: {
  messages: Message[]
  targetSize: number
}): Promise<Message[]> {
  // 1. 分析消息重要性
  // 2. 保留关键消息
  // 3. 压缩非关键内容
  // 4. 返回压缩后消息
}
```

**功能**:
- 自动压缩决策
- 微压缩 (micro-compact)
- 上下文折叠 (context collapse)

### 4. ACP 服务 (`services/acp/`)

**agent.ts** - ACP Agent

```typescript
export class AcpAgent {
  async handleRequest(request: AcpRequest): Promise<AcpResponse>
  async connect(client: AcpClient): Promise<void>
  disconnect(): void
}
```

**功能**:
- ACP 协议实现
- IDE 深度集成
- 会话管理

### 5. 分析服务 (`services/analytics/`)

**langfuse/** - Langfuse 集成

```typescript
export function trackTurn(params: {
  sessionId: string
  turnId: string
  model: string
  tokens: TokenCount
}): void
```

**功能**:
- 追踪 Agent 循环
- 成本分析
- 性能监控

### 6. Goal 服务 (`services/goal/`)

**goal.ts** - 持续驱动

```typescript
export class GoalEngine {
  async setGoal(goal: string): Promise<void>
  async pause(): Promise<void>
  async resume(): Promise<void>
  async continue(): Promise<void>
  getStatus(): GoalStatus
}
```

**功能**:
- 跨轮自动完成
- Token 预算管理
- 完成度检查

### 7. 技能搜索 (`services/skillSearch/`)

**search.ts** - 技能发现

```typescript
export async function searchSkills(query: string): Promise<Skill[]>
export async function loadSkill(skill: Skill): Promise<void>
```

**功能**:
- TF-IDF 语义搜索
- 按需加载技能
- 技能执行

## 📊 服务依赖

```
services/
├── api/          (Claude API)
├── mcp/          (MCP 客户端)
├── compact/      (压缩服务)
├── acp/          (ACP 协议)
├── analytics/    (Langfuse, Sentry)
├── goal/         (目标驱动)
├── skillSearch/  (技能搜索)
├── toolUseSummary/ (工具摘要)
└── langfuse/     (监控)
```

## 🔗 相关文档

- [API 层](../concepts/api-layer.md)
- [MCP 集成](../concepts/mcp-integration.md)
- [ACP 协议](../concepts/acp-protocol.md)

---

*最后更新: 2026-06-23*
