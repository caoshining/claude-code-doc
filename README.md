# Claude Code Best - 源码知识库

> 基于 Karpathy LLM Wiki 方法论的 Claude Code Best 源码解读文档

## 📖 简介

这是 [Claude Code Best](https://github.com/claude-code-best/claude-code) 项目的完整源码知识库，采用 Andrej Karpathy 的 LLM Wiki 方法论组织。

## 🎯 Claude Code Best 是什么？

Claude Code Best (CCB) 是一个反向工程复刻 Anthropic 官方 Claude Code CLI 的开源项目，基于 Bun + TypeScript + React Ink 构建。

## 📚 知识库结构

```
claude-code-doc/
├── index.md           # 索引页面（从这里开始）
├── CLAUDE.md          # 项目配置
├── log.md             # 变更日志
│
├── concepts/          # 核心概念（理解"为什么"）
│   ├── project-overview.md
│   ├── architecture-overview.md
│   ├── core-loop.md
│   ├── tool-system.md
│   ├── state-management.md
│   └── api-layer.md
│
├── entities/          # 代码实体（理解"是什么"）
│   ├── entry-points.md
│   ├── core-modules.md
│   ├── services.md
│   └── packages.md
│
├── sources/           # 源码分析（深入"如何实现"）
│   ├── cli.tsx.md
│   ├── query.ts.md
│   ├── QueryEngine.ts.md
│   ├── tools.ts.md
│   └── Tool.ts.md
│
└── templates/         # 开发模板
    └── tool-template.md
```

## 🚀 快速开始

### 推荐阅读顺序

1. **[项目概述](concepts/project-overview.md)** - 了解项目背景和目标
2. **[架构总览](concepts/architecture-overview.md)** - 理解整体架构
3. **[核心循环](concepts/core-loop.md)** - 掌握主流程
4. **[工具系统](concepts/tool-system.md)** - 了解工具机制
5. **[源码分析](sources/)** - 深入具体实现

### 🎨 使用 Obsidian 阅读

**强烈推荐使用 [Obsidian](https://obsidian.md) 打开此知识库**，获得最佳阅读体验。

#### 为什么选择 Obsidian？

| 特性 | 好处 |
|------|------|
| **双向链接** | 文档之间通过 `[[链接]]` 自动关联，点击即可跳转，构建知识网络 |
| **图谱视图** | 可视化展示文档关联关系，快速理解模块依赖和知识结构 |
| **快速搜索** | 全文搜索 + 路径搜索 + 标签过滤，秒级定位所需内容 |
| **反向链接** | 查看哪些文档引用了当前概念，理解上下文和影响范围 |
| **大纲预览** | 自动生成文档大纲，快速导航到指定章节 |
| **本地优先** | 完全离线可用，无网络时也能流畅阅读 |
| **插件生态** | 超过 1000+ 社区插件，可按需扩展功能 |
| **跨平台** | Windows / macOS / Linux / iOS / Android 全平台同步 |

#### 与其他阅读方式对比

| 方式 | 双向链接 | 图谱视图 | 离线可用 | 搜索能力 | 便携性 |
|------|----------|----------|----------|----------|--------|
| Obsidian | ✅ | ✅ | ✅ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| GitHub Web | ❌ | ❌ | ❌ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| VS Code | ✅ | ❌ | ✅ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Markdown 编辑器 | ❌ | ❌ | ✅ | ⭐⭐ | ⭐⭐ |

#### 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/caoshining/claude-code-doc.git

# 2. 下载 Obsidian: https://obsidian.md/download

# 3. 打开 Obsidian，选择 "打开文件夹作为知识库"
#    选择克隆的 claude-code-doc 目录

# 4. 享受阅读体验！
```

#### 推荐插件（可选）

安装后可进一步增强阅读体验：

- **Dataview** - 高级查询和数据视图
- **Excalidraw** - 手绘架构图
- **Advanced Tables** - 表格编辑增强
- **Heatmap Calendar** - 阅读进度追踪

## 📊 项目统计

| 指标 | 数值 |
|------|------|
| 总代码量 | ~80,000 行 TypeScript |
| 组件数 | 149 个 React Ink 组件 |
| 工具数 | 60+ 内置工具 |
| 包数 | 17 个 workspace 包 |
| 文档数 | 18 个解读文档 |

## 🛠️ 技术栈

- **Runtime**: Bun (>=1.3.0)
- **Language**: TypeScript
- **UI**: React Ink (Terminal)
- **Build**: Bun.build + Vite
- **Package**: 17 workspace packages

## 🔗 相关资源

- **主仓库**: [claude-code-best/claude-code](https://github.com/claude-code-best/claude-code)
- **在线文档**: [ccb.agent-aura.top](https://ccb.agent-aura.top)
- **Discord**: [交流群组](https://discord.gg/uApuzJWGKX)

## 📝 贡献指南

欢迎提交 PR 来完善文档！

1. Fork 本仓库
2. 创建新分支 (`git checkout -b feature/amazing-doc`)
3. 提交更改 (`git commit -m 'Add amazing doc'`)
4. 推送到分支 (`git push origin feature/amazing-doc`)
5. 开启 Pull Request

## 📄 许可证

本项目文档采用 MIT 许可证。

## 🙏 致谢

- [Andrej Karpathy](https://github.com/karpathy) - LLM Wiki 方法论
- [Claude Code Best](https://github.com/claude-code-best/claude-code) - 原项目

---

*最后更新: 2026-06-23*
