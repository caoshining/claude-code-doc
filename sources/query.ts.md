# query.ts - API 查询源码分析

## 📄 文件信息

- **路径**: `src/query.ts`
- **行数**: ~2,400 行
- **职责**: Claude API 查询执行

## 🔍 核心函数

### query() - 主查询函数

```typescript
export async function query(params: QueryParams): Promise<QueryResult> {
  // 1. 构建请求
  const request = await buildRequest(params)

  // 2. 调用 API
  const stream = await apiClient.messages.create(request)

  // 3. 处理流式响应
  const result = await processStream(stream, params)

  // 4. 返回结果
  return result
}
```

## 📊 执行流程

```
query()
    ↓
buildRequest()
    ├── 构建消息列表
    ├── 添加系统提示
    ├── 添加工具定义
    └── 设置模型参数
    ↓
createMessage()
    ├── 发送 API 请求
    └── 返回流
    ↓
processStream()
    ├── 处理 content_block_delta
    ├── 处理 tool_use
    └── 处理 message_delta
    ↓
handleToolUse()
    ├── 查找工具
    ├── 检查权限
    ├── 执行工具
    └── 收集结果
    ↓
buildResult()
    └── 返回 QueryResult
```

## 🔑 关键逻辑

### 1. 请求构建

```typescript
async function buildRequest(params: QueryParams): Promise<MessageCreateParams> {
  const messages = normalizeMessagesForAPI(params.messages)

  return {
    model: params.model,
    messages,
    system: await buildSystemPrompt(params),
    tools: buildToolsDefinition(params.tools),
    betas: ['prompt-caching-2024-01-29'],
    stream: true,
  }
}
```

### 2. 流式处理

```typescript
async function processStream(
  stream: AsyncStream,
  params: QueryParams
): Promise<QueryResult> {
  const contentBlocks: ContentBlock[] = []
  const toolUseBlocks: ToolUseBlock[] = []
  let stopReason: StopReason | null = null

  for await (const event of stream) {
    switch (event.type) {
      case 'content_block_start':
        onContentBlockStart(event)
        break

      case 'content_block_delta':
        contentBlocks.push(event.delta)
        onContentDelta(event.delta)
        break

      case 'tool_use':
        const toolResult = await handleToolUse(event.input, params)
        toolUseBlocks.push({ ...event, result: toolResult })
        break

      case 'message_delta':
        stopReason = event.delta.stop_reason
        updateUsage(event.usage)
        break

      case 'message_stop':
        break
    }
  }

  return { contentBlocks, toolUseBlocks, stopReason }
}
```

### 3. 工具调用处理

```typescript
async function handleToolUse(
  toolUse: ToolUseBlock,
  params: QueryParams
): Promise<string> {
  // 1. 查找工具
  const tool = findToolByName(toolUse.name, params.tools)
  if (!tool) {
    throw new Error(`Tool not found: ${toolUse.name}`)
  }

  // 2. 检查权限
  const canUse = await params.canUseTool(tool)
  if (!canUse) {
    return 'Permission denied'
  }

  // 3. 执行工具
  try {
    const result = await tool.execute(toolUse.input, params.context)
    return result
  } catch (error) {
    return `Error: ${error.message}`
  }
}
```

### 4. 错误重试

```typescript
async function queryWithRetry(
  params: QueryParams,
  maxRetries = 3
): Promise<QueryResult> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await query(params)
    } catch (error) {
      if (isRetryableError(error)) {
        await backoff(attempt)
        continue
      }
      throw error
    }
  }
}
```

## 🎯 设计特点

### 1. 流式优先

- 全程流式处理
- 增量 UI 更新
- 低延迟响应

### 2. 工具递归

- 支持嵌套工具调用
- 自动收集结果
- 继续 API 调用

### 3. 错误处理

- 分类错误类型
- 自动重试机制
- 优雅降级

## 🔗 相关文件

- [QueryEngine.ts](./QueryEngine.ts.md) - 高级编排
- [Tool.ts](./Tool.ts.md) - 工具接口
- [api/claude.ts](../entities/services.md) - API 客户端

---

*最后更新: 2026-06-23*
