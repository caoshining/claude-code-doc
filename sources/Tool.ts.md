# Tool.ts - 工具接口源码分析

## 📄 文件信息

- **路径**: `src/Tool.ts`
- **行数**: ~900 行
- **职责**: 工具接口定义

## 🔍 核心类型

### Tool 接口

```typescript
export interface Tool {
  // 工具标识符
  name: string

  // 工具描述 (AI 看到的说明)
  description: string

  // 输入参数 JSON Schema
  inputSchema: JSONObject

  // 执行函数
  execute: (params: unknown, context: ToolUseContext) => Promise<string>

  // 详细文档 (可选)
  detailedDescription?: string

  // 权限级别 (可选)
  permissionLevel?: 'read' | 'write' | 'execute'

  // 是否延迟加载 (可选)
  deferred?: boolean

  // 分类标签 (可选)
  tags?: string[]
}
```

### ToolUseContext

```typescript
export interface ToolUseContext {
  // 工作目录
  cwd: string

  // 项目根目录
  projectRoot: string

  // 会话 ID
  sessionId: string

  // Turn ID
  turnId: string

  // 模型信息
  model: string

  // 权限检查函数
  canUseTool: (tool: Tool) => Promise<boolean>

  // 权限请求函数
  requestPermission: (tool: Tool) => Promise<boolean>
}
```

### Tools 类型

```typescript
export type Tools = Record<string, Tool>
```

## 🔍 工具函数

### findToolByName

```typescript
export function findToolByName(
  name: string,
  tools: Tools
): Tool | undefined {
  // 精确匹配
  if (name in tools) {
    return tools[name]
  }

  // 模糊匹配
  for (const [key, tool] of Object.entries(tools)) {
    if (toolMatchesName(tool, name)) {
      return tool
    }
  }

  return undefined
}
```

### toolMatchesName

```typescript
export function toolMatchesName(
  tool: Tool,
  pattern: string
): boolean {
  // 精确匹配
  if (tool.name === pattern) {
    return true
  }

  // 前缀匹配
  if (pattern.startsWith(tool.name)) {
    return true
  }

  // 别名匹配
  if (tool.aliases?.includes(pattern)) {
    return true
  }

  return false
}
```

### validateToolInput

```typescript
export function validateToolInput(
  tool: Tool,
  input: unknown
): ValidationResult {
  // JSON Schema 验证
  const schema = tool.inputSchema
  const validator = new Ajv()

  const valid = validator.validate(schema, input)

  if (!valid) {
    return {
      valid: false,
      errors: validator.errors,
    }
  }

  return { valid: true }
}
```

## 🎯 设计特点

### 1. 接口简洁

```typescript
// 最小工具定义
const SimpleTool: Tool = {
  name: 'simple',
  description: 'A simple tool',
  inputSchema: { type: 'object', properties: {} },
  execute: async () => 'Hello'
}
```

### 2. 类型安全

```typescript
// 类型推断
const tool: Tool = {
  name: 'my_tool',
  description: 'My tool',
  inputSchema: {
    type: 'object',
    properties: {
      param1: { type: 'string' }
    }
  },
  execute: async (params) => {
    // params 类型为 unknown
    // 需要运行时验证
    const validated = validateToolInput(this, params)
    // ...
  }
}
```

### 3. 上下文感知

```typescript
// 工具可以访问上下文
const ContextAwareTool: Tool = {
  name: 'context_aware',
  description: 'Uses context',
  inputSchema: { type: 'object', properties: {} },
  execute: async (params, context) => {
    // 可以使用 cwd, projectRoot, sessionId 等
    console.log(context.cwd)
    return 'OK'
  }
}
```

## 🔗 相关文件

- [tools.ts](./tools.ts.md) - 工具注册
- [工具系统](../concepts/tool-system.md) - 系统设计

---

*最后更新: 2026-06-23*
