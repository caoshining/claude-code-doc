# 工具开发模板 (Tool Template)

## 📋 模板结构

创建一个新工具需要以下步骤：

### 1. 创建工具目录

```bash
mkdir -p packages/builtin-tools/src/tools/MyTool
```

### 2. 实现工具

**文件**: `packages/builtin-tools/src/tools/MyTool/MyTool.ts`

```typescript
import { type Tool } from '../../../../src/Tool.js'

export const MyTool: Tool = {
  // 工具名称 (snake_case)
  name: 'my_tool',

  // 工具描述 (AI 会看到这个)
  description: '做什么的简短描述',

  // 输入参数 JSON Schema
  inputSchema: {
    type: 'object',
    properties: {
      param1: {
        type: 'string',
        description: '参数 1 的说明'
      },
      param2: {
        type: 'number',
        description: '参数 2 的说明',
        minimum: 0,
        maximum: 100
      }
    },
    required: ['param1']
  },

  // 详细文档 (可选)
  detailedDescription: `
    更详细的说明文档。
    可以包含多行文本、使用示例等。
  `,

  // 权限级别 (可选)
  permissionLevel: 'read',  // 'read' | 'write' | 'execute'

  // 分类标签 (可选)
  tags: ['category', 'subcategory'],

  // 执行函数
  async execute(params, context) {
    // 1. 参数验证
    const { param1, param2 } = params as {
      param1: string
      param2?: number
    }

    // 2. 执行逻辑
    try {
      // 你的代码
      const result = doSomething(param1, param2)

      // 3. 返回结果
      return JSON.stringify(result)
    } catch (error) {
      return `Error: ${error.message}`
    }
  }
}
```

### 3. 添加索引

**文件**: `packages/builtin-tools/src/tools/MyTool/index.ts`

```typescript
export { MyTool } from './MyTool.js'
```

### 4. 注册工具

**文件**: `src/tools.ts`

```typescript
import { MyTool } from '@claude-code-best/builtin-tools/tools/MyTool/index.js'

export async function getTools(): Promise<Tools> {
  return {
    // ... 其他工具
    my_tool: MyTool,
  }
}
```

## 📝 完整示例

### 文件读取工具示例

```typescript
import { readFile } from 'fs/promises'
import { type Tool } from '../../../../src/Tool.js'

export const FileReadTool: Tool = {
  name: 'read_file',
  description: 'Read the contents of a file',
  inputSchema: {
    type: 'object',
    properties: {
      file_path: {
        type: 'string',
        description: 'The path to the file to read'
      }
    },
    required: ['file_path']
  },
  permissionLevel: 'read',

  async execute(params, context) {
    const { file_path } = params as { file_path: string }

    try {
      const absolutePath = resolvePath(file_path, context.cwd)
      const content = await readFile(absolutePath, 'utf-8')
      return content
    } catch (error) {
      return `Error reading file: ${error.message}`
    }
  }
}
```

## 🎯 最佳实践

### 1. 命名规范

- 工具名: `snake_case`
- 类名: `PascalCase`
- 文件名: `PascalCase.ts`

### 2. 错误处理

```typescript
async execute(params, context) {
  try {
    // 主要逻辑
    return result
  } catch (error) {
    // 优雅处理错误
    if (error.code === 'ENOENT') {
      return 'File not found'
    }
    return `Error: ${error.message}`
  }
}
```

### 3. 路径处理

```typescript
// 使用 cwd 作为相对路径基准
const absolutePath = path.resolve(context.cwd, relativePath)

// 或使用项目根目录
const absolutePath = path.resolve(context.projectRoot, relativePath)
```

### 4. 权限声明

```typescript
// 明确声明权限级别
permissionLevel: 'read'    // 只读
permissionLevel: 'write'   // 写入
permissionLevel: 'execute' // 执行
```

## 🔗 相关文档

- [Tool.ts](../sources/Tool.ts.md) - 工具接口
- [工具系统](../concepts/tool-system.md) - 系统设计

---

*最后更新: 2026-06-23*
