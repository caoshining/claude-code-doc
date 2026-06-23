# tools.ts - 工具注册源码分析

## 📄 文件信息

- **路径**: `src/tools.ts`
- **行数**: ~600 行
- **职责**: 工具注册表

## 🔍 源码结构

### 1. 核心工具导入

```typescript
// 核心工具 (始终加载)
import { AgentTool } from '@claude-code-best/builtin-tools/tools/AgentTool/AgentTool.js'
import { BashTool } from '@claude-code-best/builtin-tools/tools/BashTool/BashTool.js'
import { FileEditTool } from '@claude-code-best/builtin-tools/tools/FileEditTool/FileEditTool.js'
import { FileReadTool } from '@claude-code-best/builtin-tools/tools/FileReadTool/FileReadTool.js'
import { FileWriteTool } from '@claude-code-best/builtin-tools/tools/FileWriteTool/FileWriteTool.js'
// ... 33 more
```

### 2. 条件工具 (Feature Gated)

```typescript
// ANT-ONLY 工具
const REPLTool =
  process.env.USER_TYPE === 'ant'
    ? require('@claude-code-best/builtin-tools/tools/REPLTool/REPLTool.js').REPLTool
    : null

// Feature flag 工具
const SleepTool =
  feature('PROACTIVE') || feature('KAIROS')
    ? require('@claude-code-best/builtin-tools/tools/SleepTool/SleepTool.js').SleepTool
    : null
```

### 3. 工具注册函数

```typescript
export async function getTools(): Promise<Tools> {
  return {
    // 文件操作
    read_file: FileReadTool,
    write_file: FileWriteTool,
    edit_file: FileEditTool,

    // Shell 操作
    bash: BashTool,
    powershell: PowerShellTool,

    // Agent 系统
    agent: AgentTool,
    task_create: TaskCreateTool,
    task_stop: TaskStopTool,

    // Web 操作
    web_search: WebSearchTool,
    web_fetch: WebFetchTool,

    // MCP
    mcp: MCPTool,
    mcp_auth: McpAuthTool,

    // 调度
    schedule_cron: CronCreateTool,
    cron_delete: CronDeleteTool,
    cron_list: CronListTool,

    // 规划
    enter_plan_mode: EnterPlanModeTool,
    exit_plan_mode: ExitPlanModeV2Tool,

    // ... 60+ tools
  }
}
```

### 4. 核心工具常量

```typescript
// CORE_TOOLS 白名单 (38 个)
export const CORE_TOOLS = new Set([
  'FileReadTool',
  'FileEditTool',
  'FileWriteTool',
  'BashTool',
  'PowerShellTool',
  'AgentTool',
  'TaskCreateTool',
  'TaskStopTool',
  'WebSearchTool',
  'WebFetchTool',
  'MCPTool',
  'CronCreateTool',
  'CronDeleteTool',
  'CronListTool',
  'EnterPlanModeTool',
  'ExitPlanModeV2Tool',
  // ... 22 more
] as const)
```

## 🎯 设计特点

### 1. 分层加载

```
┌─────────────────┐
│  核心工具       │ 38 个，始终加载
│  (CORE_TOOLS)   │ 白名单制
└────────┬────────┘
         ↓
┌─────────────────┐
│  条件工具       │ Feature flag 控制
│  (Feature Gated)│ 按需启用
└────────┬────────┘
         ↓
┌─────────────────┐
│  延迟工具       │ TF-IDF 搜索
│  (Deferred)     │ 按需加载
└─────────────────┘
```

### 2. Tree-shaking 优化

```typescript
// Feature flag 控制编译时包含
const tool = feature('FEATURE_NAME')
  ? require('./Tool.js').Tool
  : null  // 被编译器删除
```

### 3. 动态加载

```typescript
// 延迟工具通过搜索动态加载
const tool = await searchExtraTools(query)
if (tool) {
  const { Tool } = await import(tool.path)
  tools.add(Tool)
}
```

## 📊 工具分类

| 分类 | 工具数 | 示例 |
|------|--------|------|
| 文件操作 | 6 | Read, Write, Edit, Glob, Grep |
| Shell 执行 | 3 | Bash, PowerShell, REPL |
| Agent 系统 | 5 | Agent, Task, Workflow |
| Web/MCP | 4 | Search, Fetch, MCP, Auth |
| 调度 | 3 | CronCreate, CronDelete, CronList |
| 规划 | 3 | EnterPlan, ExitPlan, Verify |
| 其他 | 36+ | Skill, Todo, Artifact, LSP |

## 🔗 相关文件

- [Tool.ts](./Tool.ts.md) - 工具接口
- [工具系统](../concepts/tool-system.md) - 系统设计

---

*最后更新: 2026-06-23*
