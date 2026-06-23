# 工具系统 (Tool System)

## 🔧 什么是工具?

工具是 Claude 可以调用的**外部函数**，允许 AI 与系统交互:
- 读写文件
- 执行命令
- 搜索代码
- 调用 API
- 启动子 Agent

## 📐 工具架构

```
┌─────────────────────────────────────────────────────────────┐
│                       Tool Registry                         │
│                     (tools.ts)                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • 导入所有工具定义                                 │   │
│  │  • 注册到工具列表                                   │   │
│  │  • Feature flag 控制                                │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                       Tool Interface                        │
│                      (Tool.ts)                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  interface Tool {                                    │   │
│  │    name: string                                     │   │
│  │    description: string                               │   │
│  │    inputSchema: JSON Schema                         │   │
│  │    execute(params, context): Promise<ToolResult>    │   │
│  │  }                                                   │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                   Tool Implementations                      │
│                (@claude-code-best/builtin-tools)              │
│  ┌─────────────┬─────────────┬─────────────┬───────────┐   │
│  │ File Ops    │ Shell Ops   │ Agent Ops   │ Web/MCP   │   │
│  │ Read/Edit   │ Bash/PS     │ Agent/Task  │ Search    │   │
│  │ Write/Glob  │ REPL        │ Workflow    │ Fetch     │   │
│  └─────────────┴─────────────┴─────────────┴───────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 🔑 工具接口

```typescript
interface Tool {
  // 工具标识
  name: string

  // 工具描述 (AI 看到的说明)
  description: string

  // 输入参数 JSON Schema
  inputSchema: JSONObject

  // 执行函数
  execute(params: unknown, context: ToolUseContext): Promise<string>

  // 可选: 详细文档
  detailedDescription?: string

  // 可选: 权限级别
  permissionLevel?: 'read' | 'write' | 'execute'

  // 可选: 是否延迟加载
  deferred?: boolean
}
```

## 📂 工具分类

### 1. 文件操作工具

| 工具 | 功能 | 权限 |
|------|------|------|
| **FileReadTool** | 读取文件内容 | read |
| **FileWriteTool** | 完整写入文件 | write |
| **FileEditTool** | 精确替换内容 | write |
| **GlobTool** | 模式匹配文件 | read |
| **GrepTool** | 搜索文件内容 | read |
| **NotebookEditTool** | 编辑 Jupyter Notebook | write |

### 2. Shell 执行工具

| 工具 | 功能 | 权限 |
|------|------|------|
| **BashTool** | 执行 Bash 命令 | execute |
| **PowerShellTool** | 执行 PowerShell 命令 | execute |
| **REPLTool** | 执行 REPL 代码 | execute |

### 3. Agent 系统工具

| 工具 | 功能 | 说明 |
|------|------|------|
| **AgentTool** | 启动子 Agent | 委托复杂任务 |
| **TaskCreateTool** | 创建后台任务 | 异步执行 |
| **TaskOutputTool** | 获取任务输出 | 查看结果 |
| **TaskStopTool** | 停止任务 | 中止执行 |

### 4. 规划工具

| 工具 | 功能 | 说明 |
|------|------|------|
| **EnterPlanModeTool** | 进入规划模式 | 制定实施计划 |
| **ExitPlanModeV2Tool** | 退出规划模式 | 开始执行 |
| **VerifyPlanExecutionTool** | 验证执行结果 | 检查完成度 |

### 5. Web/MCP 工具

| 工具 | 功能 | 说明 |
|------|------|------|
| **WebFetchTool** | 获取网页内容 | HTTP GET |
| **WebSearchTool** | 网页搜索 | Bing/Brave |
| **MCPTool** | 调用 MCP 服务 | MCP 协议 |
| **McpAuthTool** | MCP 认证 | OAuth/API Key |

### 6. 调度工具

| 工具 | 功能 | 说明 |
|------|------|------|
| **CronCreateTool** | 创建定时任务 | Cron 表达式 |
| **CronDeleteTool** | 删除定时任务 | 取消调度 |
| **CronListTool** | 列出定时任务 | 查看状态 |

### 7. 搜索与发现

| 工具 | 功能 | 说明 |
|------|------|------|
| **SearchExtraToolsTool** | 搜索扩展工具 | TF-IDF 语义搜索 |
| **ExecuteExtraTool** | 执行扩展工具 | 按需加载 |
| **SyntheticOutputTool** | 结构化输出 | CORE_TOOLS 白名单 |

### 8. 其他工具

| 工具 | 功能 | 说明 |
|------|------|------|
| **SkillTool** | 调用 Skill | 技能系统 |
| **TodoWriteTool** | 更新待办事项 | 任务列表 |
| **ArtifactTool** | 上传 HTML | Artifacts 服务 |
| **LSPTool** | LSP 协议交互 | 代码智能 |
| **ConfigTool** | 配置管理 | settings.json |

## 🔄 工具调用流程

```
1. Claude 决定调用工具
   ↓
2. 检测 tool_use_block
   ↓
3. 查找 Tool (by name)
   ↓
4. 检查权限
   ↓
5. 请求用户批准 (如需要)
   ↓
6. 执行 Tool.execute()
   ↓
7. 收集结果
   ↓
8. 构建 tool_result_block
   ↓
9. 发送回 Claude API
```

## 🎯 工具发现

### 核心工具 (CORE_TOOLS)

38 个核心工具始终加载，在白名单内:

```typescript
const CORE_TOOLS = [
  'FileReadTool',
  'FileEditTool',
  'BashTool',
  'WebSearchTool',
  // ... 34 more
]
```

### 延迟工具 (Deferred Tools)

非核心工具按需加载，通过 **TF-IDF 搜索**:

```typescript
// 用户查询
"如何部署到 AWS?"

// TF-IDF 匹配
searchExtraTools(query)
  → 找到 "DeployTool" (AWS 部署)

// 动态加载
import('@claude-code-best/builtin-tools/tools/DeployTool')

// 临时注册
tools.add(DeployTool)
```

## 🔐 权限系统

### 权限模式

| 模式 | 行为 |
|------|------|
| **auto** | 自动批准所有工具 |
| **ask** | 每次请求用户批准 |
| **read-only** | 只允许读取操作 |
| **custom** | 自定义规则 |

### 权限检查

```typescript
function canUseTool(
  tool: Tool,
  permissionMode: PermissionMode
): boolean {
  switch (permissionMode) {
    case 'auto':
      return true
    case 'read-only':
      return tool.permissionLevel === 'read'
    case 'ask':
      return await askUser(tool)
    case 'custom':
      return checkCustomRules(tool)
  }
}
```

## 🧩 工具开发

### 模板

```typescript
// packages/builtin-tools/src/tools/MyTool/MyTool.ts
import { type Tool } from '../../../../src/Tool.js'

export const MyTool: Tool = {
  name: 'my_tool',
  description: '做什么的',
  inputSchema: {
    type: 'object',
    properties: {
      param1: {
        type: 'string',
        description: '参数说明'
      }
    },
    required: ['param1']
  },

  async execute(params, context) {
    // 实现逻辑
    return '结果'
  }
}
```

### 注册

```typescript
// src/tools.ts
import { MyTool } from '@claude-code-best/builtin-tools/tools/MyTool/MyTool.js'

export function getTools(): Tools {
  return {
    // ... 其他工具
    my_tool: MyTool
  }
}
```

## 📊 工具统计

```
总工具数:    60+
核心工具:    38 (常驻)
延迟工具:    22+ (按需)
内置工具:    来自 builtin-tools 包
自定义工具:  用户可通过 MCP/Plugin 添加
```

## 🔗 相关模块

- [核心循环](core-loop.md) - 工具如何被调用
- [MCP 集成](mcp-integration.md) - 扩展工具系统
- [权限系统](state-management.md) - 权限如何管理

---

*下一步: [状态管理](state-management.md)*
