# 包结构 (Packages)

## 📍 概述

CCB 使用 Bun workspaces 管理多个包，实现模块化架构。

## 📦 包列表

### @ant/* (Anthropic 包)

#### @ant/ink

**路径**: `packages/@ant/ink/`
**用途**: Forked Ink 框架
**内容**:
- components/ - UI 组件
- core/ - 渲染引擎
- hooks/ - React hooks
- keybindings/ - 快捷键系统
- theme/ - 主题系统

#### @ant/computer-use-mcp

**路径**: `packages/@ant/computer-use-mcp/`
**用途**: Computer Use MCP 服务
**功能**: 截图、键鼠、剪贴板

#### @ant/computer-use-input

**路径**: `packages/@ant/computer-use-input/`
**用途**: 键鼠模拟
**平台**: Darwin, Win32, Linux

#### @ant/computer-use-swift

**路径**: `packages/@ant/computer-use-swift/`
**用途**: 截图 + 应用管理
**平台**: macOS (Swift)

#### @ant/claude-for-chrome-mcp

**路径**: `packages/@ant/claude-for-chrome-mcp/`
**用途**: Chrome 浏览器控制
**启用**: `--chrome` 参数

#### @ant/model-provider

**路径**: `packages/@ant/model-provider/`
**用途**: Model provider 抽象层
**功能**: 统一 7 种 API 提供商接口

### 功能包

#### builtin-tools

**路径**: `packages/builtin-tools/`
**用途**: 60+ 内置工具
**结构**:
```
src/tools/
├── FileReadTool/
├── FileEditTool/
├── BashTool/
├── AgentTool/
├── WebSearchTool/
├── MCPTool/
└── ... (60+ tools)
```

#### agent-tools

**路径**: `packages/agent-tools/`
**用途**: Agent 工具集
**功能**: 子 Agent 管理工具

#### mcp-client

**路径**: `packages/mcp-client/`
**用途**: MCP 客户端库
**功能**: MCP 协议实现

#### acp-link

**路径**: `packages/acp-link/`
**用途**: ACP 代理服务器
**功能**: WebSocket → ACP agent 桥接

#### cloud-artifacts

**路径**: `packages/cloud-artifacts/`
**用途**: HTML 托管服务
**架构**: Cloudflare Worker + R2
**功能**: 上传、托管、自动过期

#### remote-control-server

**路径**: `packages/remote-control-server/`
**用途**: 自托管 RCS
**架构**: React + Vite + Radix UI
**功能**: Web UI、会话管理

### 原生模块

#### audio-capture-napi

**路径**: `packages/audio-capture-napi/`
**用途**: 音频捕获
**平台**: 全平台

#### color-diff-napi

**路径**: `packages/color-diff-napi/`
**用途**: 颜色差异计算
**测试**: 11 tests passing

#### image-processor-napi

**路径**: `packages/image-processor-napi/`
**用途**: 图像处理
**功能**: 缩放、裁剪、格式转换

#### modifiers-napi

**路径**: `packages/modifiers-napi/`
**用途**: 键盘修饰键检测
**平台**: macOS (FFI)

#### url-handler-napi

**路径**: `packages/url-handler-napi/`
**用途**: URL scheme 处理
**功能**: 深度链接支持

### 其他包

#### weixin

**路径**: `packages/weixin/`
**用途**: 微信集成
**命令**: `ccb weixin`

#### workflow-engine

**路径**: `packages/workflow-engine/`
**用途**: Workflow 引擎
**功能**: 多 Agent 确定性编排

## 📊 依赖关系

```
claude-code (主项目)
├── @ant/ink (UI 框架)
├── @ant/model-provider (API 抽象)
├── builtin-tools (工具)
├── agent-tools (Agent 工具)
├── mcp-client (MCP 客户端)
├── acp-link (ACP 服务器)
└── remote-control-server (RCS)
```

## 🔗 相关文档

- [架构总览](../concepts/architecture-overview.md)
- [工具系统](../concepts/tool-system.md)

---

*最后更新: 2026-06-23*
