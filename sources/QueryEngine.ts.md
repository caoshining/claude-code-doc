# QueryEngine.ts - 查询引擎源码分析

## 📄 文件信息

- **路径**: `src/QueryEngine.ts`
- **行数**: ~1,400 行
- **职责**: 高级编排器

## 🔍 核心类

### QueryEngine

```typescript
export class QueryEngine {
  private turnCount: number = 0
  private fileStateCache: FileStateCache
  private attributionState: AttributionState
  private autoCompactState: AutoCompactTrackingState

  constructor(
    private tools: Tools,
    private canUseTool: CanUseToolFn,
    private model: string
  ) {
    this.fileStateCache = createFileStateCache()
    this.attributionState = createAttributionState()
    this.autoCompactState = createAutoCompactState()
  }

  async query(userInput: string): Promise<QueryResult>
  private async buildRequest(): Promise<RequestParams>
  private async processResponse(response: AsyncIterable): Promise<void>
  private shouldCompact(): boolean
  private updateFileHistory(): void
}
```

## 📊 执行流程

```
QueryEngine.query()
    ↓
┌─────────────────────────────────────────┐
│  1. 前处理                               │
│     - 更新 turnCount                     │
│     - 检查压缩需求                       │
│     - 快照文件状态                       │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  2. 构建请求                            │
│     - 收集消息历史                       │
│     - 添加系统提示                       │
│     - 添加工具定义                       │
│     - 设置 token 预算                    │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  3. 执行查询 (query.ts)                 │
│     - API 调用                           │
│     - 流式处理                           │
│     - 工具执行                           │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  4. 后处理                              │
│     - 更新消息历史                       │
│     - 记录 token 使用                    │
│     - 更新文件归因                       │
│     - 检查是否需要压缩                   │
└─────────────────┬───────────────────────┘
                  ↓
            返回结果
```

## 🔑 关键逻辑

### 1. 压缩决策

```typescript
private shouldCompact(): boolean {
  const totalTokens = getTotalInputTokens() + getTotalOutputTokens()

  // 检查 1: 接近上下文上限
  if (totalTokens > getContextWindowSize() * 0.9) {
    return true
  }

  // 检查 2: 最近响应超过 200k
  if (doesMostRecentAssistantMessageExceed200k()) {
    return true
  }

  // 检查 3: 自动压缩启用且达到阈值
  if (isAutoCompactEnabled() && this.turnCount % 10 === 0) {
    return true
  }

  return false
}
```

### 2. 文件历史快照

```typescript
private async snapshotFileState(): Promise<void> {
  const files = getModifiedFiles(this.messages)

  for (const file of files) {
    const before = this.fileStateCache.get(file)
    const after = await readFile(file)

    this.fileStateCache.set(file, {
      before,
      after,
      timestamp: Date.now(),
      turnId: this.turnCount,
    })

    // 记录归因
    this.attributionState.recordChange({
      file,
      turnId: this.turnCount,
      changes: diff(before, after),
    })
  }
}
```

### 3. Token 预算管理

```typescript
private async buildRequest(): Promise<RequestParams> {
  const budget = getTurnOutputTokenBudget()

  return {
    model: this.model,
    messages: this.messages,
    tools: this.tools,
    system: await buildSystemPrompt(),
    max_tokens: budget.remaining,
    betas: ['prompt-caching-2024-01-29'],
  }
}
```

### 4. 会话恢复

```typescript
async resume(sessionId: string): Promise<void> {
  // 1. 加载会话状态
  const session = await loadSession(sessionId)

  // 2. 恢复消息历史
  this.messages = session.messages

  // 3. 恢复文件缓存
  this.fileStateCache = session.fileStateCache

  // 4. 恢复归因状态
  this.attributionState = session.attributionState

  // 5. 恢复 turn 计数
  this.turnCount = session.turnCount
}
```

## 🎯 设计特点

### 1. 状态分层

- **会话级**: sessionId, turnCount
- **查询级**: messages, fileStateCache
- **工具级**: attributionState

### 2. 增量更新

- 文件状态增量快照
- Token 使用累加
- 归因逐步记录

### 3. 自动优化

- 压缩决策自动化
- Token 预算自适应
- 缓存大小限制

## 🔗 相关文件

- [query.ts](./query.ts.md) - API 执行
- [bootstrap/state.ts](../entities/core-modules.md) - 全局状态

---

*最后更新: 2026-06-23*
