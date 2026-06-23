# 入口点 (Entry Points)

## 📍 概述

CCB 有多个入口点，根据不同的使用场景：

```
用户命令
   ↓
cli.tsx (主入口)
   ├─→ --version          (快速路径)
   ├─→ --dump-system-prompt (快速路径)
   ├─→ --acp              (ACP 模式)
   ├─→ --claude-in-chrome-mcp (Chrome MCP)
   ├─→ --computer-use-mcp (Computer Use)
   └─→ main.tsx          (完整 CLI)
        ├─→ REPL Mode    (交互式)
        ├─→ Headless Mode (非交互)
        └─→ Subcommands   (mcp, server, auth, etc.)
```

## 🔑 主要入口

### 1. cli.tsx - 主入口

**文件**: `src/entrypoints/cli.tsx`

**行数**: ~200 行

**职责**:
- 快速路径处理
- 环境设置
- Feature gate 检查

**快速路径**:

```typescript
// --version: 零模块加载
if (args[0] === '--version') {
  console.log(MACRO.VERSION)
  return
}

// --dump-system-prompt: 仅加载配置
if (args[0] === '--dump-system-prompt') {
  const prompt = await getSystemPrompt()
  console.log(prompt)
  return
}

// --acp: ACP 模式
if (args[0] === '--acp') {
  await runAcpAgent()
  return
}
```

**环境初始化**:

```typescript
// 性能 shim (必须第一个导入)
import './utils/performanceShim.js'

// MACRO 定义回退
if (typeof globalThis.MACRO === 'undefined') {
  globalThis.MACRO = {
    VERSION: process.env.CLAUDE_CODE_VERSION || '2.1.888',
    BUILD_TIME: new Date().toISOString(),
    // ...
  }
}

// Feature flag 检查
if (feature('DUMP_SYSTEM_PROMPT')) { /* ... */ }
```

### 2. main.tsx - CLI 定义

**文件**: `src/main.tsx`

**行数**: ~5,600 行

**职责**:
- Commander.js CLI 定义
- 子命令注册
- 权限处理
- 会话恢复
- 模式分发

**CLI 结构**:

```typescript
program
  .version(MACRO.VERSION)
  .description('Claude Code - AI coding assistant')
  .argument('[prompt...]', 'Prompt to process')
  .option('-p, --prompt', 'Prompt mode')
  .option('--headless', 'Headless mode')
  .option('--non-interactive', 'Non-interactive mode')
  // ... 更多选项

// 子命令
program
  .command('mcp')
  .description('MCP server commands')
  .addCommand(mcpServeCommand)
  .addCommand(mcpAddCommand)
  // ...

program
  .command('server')
  .description('Start local server')
  .action(serverHandler)

// 主 action
program.action(async (prompt, options) => {
  // 权限检查
  // MCP 初始化
  // 会话恢复
  // 模式分发 (REPL/Headless)
})
```

**子命令列表**:

| 命令 | 功能 |
|------|------|
| `mcp` | MCP 服务器管理 |
| `server` | 启动本地服务器 |
| `auth` | 认证管理 |
| `plugin` | 插件管理 |
| `agents` | Agent 管理 |
| `doctor` | 诊断工具 |
| `update` | 更新检查 |
| `ps` | 会话列表 |
| `attach` | 附加到会话 |
| `kill` | 终止会话 |

### 3. init.ts - 一次性初始化

**文件**: `src/entrypoints/init.ts`

**职责**:
- Telemetry 初始化
- 配置设置
- 信任对话框

**执行流程**:

```typescript
export async function init(): Promise<void> {
  // 1. 检查是否首次运行
  if (isFirstRun()) {
    // 2. 显示欢迎信息
    showWelcome()

    // 3. 询问遥测
    const telemetry = await askTelemetry()

    // 4. 询问信任
    await trustDialog()
  }

  // 5. 初始化配置
  await initConfig()

  // 6. 初始化遥测
  await initTelemetry()
}
```

## 🔄 执行流程

```
用户执行 ccb
   ↓
cli.tsx 入口
   ↓
检查快速路径
   ↓
加载 main.tsx
   ↓
Commander.js 解析
   ↓
分发到处理器
   ↓
┌─────────────┬─────────────┬─────────────┐
│  REPL Mode  │ Headless    │ Subcommand  │
│  (交互式)   │  (非交互)   │  (子命令)   │
└─────────────┴─────────────┴─────────────┘
```

## 📦 相关模块

- **QueryEngine**: 核心查询引擎
- **REPL**: 交互界面
- **Bootstrap**: 启动状态

## 🔗 相关文档

- [核心循环](../concepts/core-loop.md)
- [架构总览](../concepts/architecture-overview.md)

---

*最后更新: 2026-06-23*
