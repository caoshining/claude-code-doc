# Claude Code Best - 源码知识库索引

## 🚀 快速导航

### 📖 按阅读顺序

1. **[项目概述](concepts/project-overview.md)** - 项目简介和目标
2. **[架构总览](concepts/architecture-overview.md)** - 整体架构设计
3. **[核心循环](concepts/core-loop.md)** - 理解主流程
4. **[工具系统](concepts/tool-system.md)** - 工具机制
5. **[状态管理](concepts/state-management.md)** - 状态模式
6. **[API 层](concepts/api-layer.md)** - API 集成

### 🗂️ 按分类

#### Concepts (概念)
| 文档 | 描述 | 状态 |
|------|------|------|
| [Project Overview](concepts/project-overview.md) | 项目概述和背景 | ✅ |
| [Architecture Overview](concepts/architecture-overview.md) | 整体架构设计 | ✅ |
| [Core Loop](concepts/core-loop.md) | 核心循环机制 | ✅ |
| [Tool System](concepts/tool-system.md) | 工具系统设计 | ✅ |
| [State Management](concepts/state-management.md) | 状态管理 | ✅ |
| [Bridge Protocol](concepts/bridge-protocol.md) | 桥接协议 | 📝 |
| [MCP Integration](concepts/mcp-integration.md) | MCP 集成 | 📝 |
| [ACP Protocol](concepts/acp-protocol.md) | ACP 协议 | 📝 |
| [Compact Strategy](concepts/compact-strategy.md) | 压缩策略 | 📝 |

#### Entities (实体)
| 文档 | 描述 | 状态 |
|------|------|------|
| [Entry Points](entities/entry-points.md) | 入口文件分析 | 📝 |
| [Core Modules](entities/core-modules.md) | 核心模块 | 📝 |
| [UI Components](entities/ui-components.md) | UI 组件 | 📝 |
| [Services](entities/services.md) | 服务层 | 📝 |
| [Packages](entities/packages.md) | 包结构 | 📝 |

#### Sources (源码)
| 文件 | 描述 | 行数 | 状态 |
|------|------|------|------|
| [cli.tsx](sources/cli.tsx.md) | 入口文件 | ~200 | 📝 |
| [main.tsx](sources/main.tsx.md) | CLI 主文件 | ~5600 | 📝 |
| [query.ts](sources/query.ts.md) | API 查询 | ~2400 | 📝 |
| [QueryEngine.ts](sources/QueryEngine.ts.md) | 查询引擎 | ~1400 | 📝 |
| [REPL.tsx](sources/REPL.tsx.md) | REPL 界面 | ~800 | 📝 |
| [tools.ts](sources/tools.ts.md) | 工具注册 | ~600 | 📝 |
| [Tool.ts](sources/Tool.ts.md) | 工具接口 | ~900 | 📝 |

#### Comparisons (对比)
| 文档 | 描述 | 状态 |
|------|------|------|
| [CCB vs Official](comparisons/ccb-vs-official.md) | 与官方对比 | 📝 |

#### Templates (模板)
| 模板 | 描述 | 状态 |
|------|------|------|
| [Tool Template](templates/tool-template.md) | 工具开发 | 📝 |
| [Command Template](templates/command-template.md) | 命令开发 | 📝 |

## 🔍 关键问题快速查找

### 我想了解...

- **项目整体是什么?** → [Project Overview](concepts/project-overview.md)
- **数据如何流动?** → [Core Loop](concepts/core-loop.md)
- **工具如何工作?** → [Tool System](concepts/tool-system.md)
- **如何添加新工具?** → [Tool Template](templates/tool-template.md)
- **状态如何管理?** → [State Management](concepts/state-management.md)
- **如何远程控制?** → [Bridge Protocol](concepts/bridge-protocol.md)
- **MCP 如何集成?** → [MCP Integration](concepts/mcp-integration.md)

## 📊 项目统计

```
总代码量:    ~80,000 行 TypeScript
组件数:      149 个 React Ink 组件
工具数:      60+ 内置工具
包数:        17 个 workspace 包
服务数:      50+ 服务模块
```

## 🛠️ 技术栈

- **Runtime**: Bun (>=1.3.0)
- **Language**: TypeScript (strict mode)
- **UI**: React Ink (forked @anthropic/ink)
- **Build**: Bun.build + Vite
- **Package**: 17 workspace packages
- **Lint**: Biome
- **Test**: Bun test

## 📝 符号说明

- ✅ 已完成
- 📝 进行中
- 📋 计划中

---

*最后更新: 2026-06-23*
