# cli.tsx - 入口文件源码分析

## 📄 文件信息

- **路径**: `src/entrypoints/cli.tsx`
- **行数**: ~200 行
- **职责**: 主入口点、快速路径处理

## 🔍 源码结构

### 1. 导入与初始化

```typescript
#!/usr/bin/env bun
// 性能 shim MUST 第一个导入
import '../utils/performanceShim.js'
import { feature } from 'bun:bundle'
import { isEnvTruthy } from '../utils/envUtils.js'
```

**关键点**:
- `performanceShim` 必须第一个导入，替换全局 `performance` 对象
- 使用 `bun:bundle` 的 `feature()` 进行 tree-shaking

### 2. MACRO 定义回退

```typescript
// 运行时回退（未通过 build/dev defines 注入时）
if (typeof globalThis.MACRO === 'undefined') {
  (globalThis as any).MACRO = {
    VERSION: process.env.CLAUDE_CODE_VERSION || '2.1.888',
    BUILD_TIME: new Date().toISOString(),
    FEEDBACK_CHANNEL: '',
    ISSUES_EXPLAINER: '',
    NATIVE_PACKAGE_URL: '',
    PACKAGE_URL: '',
    VERSION_CHANGELOG: '',
  }
}
```

**说明**:
- 开发模式或直接运行时，提供默认值
- 构建时会被 `MACRO.*` defines 替换为字面量

### 3. 快速路径

#### --version (零模块加载)

```typescript
if (args.length === 1 && (args[0] === '--version' || args[0] === '-v' || args[0] === '-V')) {
  console.log(`${MACRO.VERSION} (Claude Code)`)
  return
}
```

**特点**:
- 零模块加载
- MACRO.VERSION 编译时内联
- 最快启动速度

#### --dump-system-prompt

```typescript
if (feature('DUMP_SYSTEM_PROMPT') && args[0] === '--dump-system-prompt') {
  profileCheckpoint('cli_dump_system_prompt_path')
  const { enableConfigs } = await import('../utils/config.js')
  enableConfigs()
  const { getMainLoopModel } = await import('../utils/model/model.js')
  const model = args[modelIdx + 1] || getMainLoopModel()
  const { getSystemPrompt } = await import('../constants/prompts.js')
  const prompt = await getSystemPrompt([], model)
  console.log(prompt.join('\n'))
  return
}
```

**说明**:
- Feature gated (仅内部构建)
- 用于 prompt 评估
- 动态导入最小依赖

#### --acp (ACP 模式)

```typescript
if (feature('ACP') && process.argv[2] === '--acp') {
  profileCheckpoint('cli_acp_path')
  const { runAcpAgent } = await import('../services/acp/entry.js')
  await runAcpAgent()
  return
}
```

**说明**:
- ACP 协议模式
- stdio 通信
- IDE 集成

### 4. 环境变量处理

```typescript
// 强制交互模式
if (isEnvTruthy(process.env.CLAUDE_CODE_FORCE_INTERACTIVE)) {
  for (const stream of [process.stdin, process.stdout, process.stderr]) {
    if (!stream.isTTY) {
      Object.defineProperty(stream, 'isTTY', {
        value: true,
        configurable: true,
      })
    }
  }
}

// CCR 环境堆内存限制
if (process.env.CLAUDE_CODE_REMOTE === 'true') {
  const existing = process.env.NODE_OPTIONS || ''
  process.env.NODE_OPTIONS = existing
    ? `${existing} --max-old-space-size=8192`
    : '--max-old-space-size=8192'
}
```

### 5. Ablation 基线

```typescript
if (feature('ABLATION_BASELINE') && process.env.CLAUDE_CODE_ABLATION_BASELINE) {
  for (const k of [
    'CLAUDE_CODE_SIMPLE',
    'CLAUDE_CODE_DISABLE_THINKING',
    'DISABLE_INTERLEAVED_THINKING',
    'DISABLE_COMPACT',
    'DISABLE_AUTO_COMPACT',
    'CLAUDE_CODE_DISABLE_AUTO_MEMORY',
    'CLAUDE_CODE_DISABLE_BACKGROUND_TASKS',
  ]) {
    process.env[k] ??= '1'
  }
}
```

**说明**:
- 用于 A/B 测试
- 禁用所有优化
- 建立性能基线

## 🎯 设计要点

### 1. 性能优化

- **零模块加载**: `--version` 快速路径
- **动态导入**: 按需加载模块
- **Feature gating**: Tree-shaking 未使用代码

### 2. 环境适配

- **强制 TTY**: CI/CD 环境支持
- **内存限制**: 容器环境优化
- **回退机制**: 开发/生产兼容

### 3. 调试支持

- **性能分析**: `profileCheckpoint` 钩子
- **Prompt 导出**: `--dump-system-prompt`
- **Ablation 模式**: 性能基线测试

## 🔗 相关文件

- [main.tsx](./main.tsx.md) - CLI 主文件
- [性能优化](../concepts/compact-strategy.md) - 压缩策略

---

*最后更新: 2026-06-23*
