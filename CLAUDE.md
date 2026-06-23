# Claude Code Best - LLM Wiki

> 基于 Karpathy LLM Wiki 方法论的 Claude Code Best 源码知识库

## 项目概述

Claude Code Best (CCB) 是一个反向工程复刻 Anthropic 官方 Claude Code CLI 的开源项目，基于 Bun + TypeScript + React Ink 构建。

## Wiki 结构

### 📚 concepts/ - 核心概念
抽象的概念和设计模式，理解项目的"为什么"

- **Core Loop** - 核心循环架构
- **Tool System** - 工具系统设计
- **Query Engine** - 查询引擎原理
- **State Management** - 状态管理模式
- **Bridge Protocol** - 桥接协议设计
- **MCP Integration** - MCP 集成机制
- **ACP Protocol** - ACP 协议实现
- **Compact Strategy** - 上下文压缩策略

### 🏗️ entities/ - 代码实体
具体的代码模块、文件、类、函数

- **Entry Points** - 入口文件分析
- **Core Modules** - 核心模块解析
- **UI Components** - UI 组件树
- **Service Layer** - 服务层模块
- **Package Structure** - 包结构说明

### 📖 sources/ - 源码分析
关键源文件的详细解读

- **cli.tsx** - 入口文件
- **main.tsx** - CLI 主文件
- **query.ts** - API 查询
- **QueryEngine.ts** - 查询引擎
- **REPL.tsx** - 交互界面
- **tools.ts** - 工具注册

### 🔄 comparisons/ - 对比分析
不同实现的对比和选择

- **CCB vs Official** - 与官方版本对比
- **Ink vs Original** - UI 框架对比
- **Provider Comparison** - API 提供商对比

### 📋 templates/ - 代码模板
常用代码模式和模板

- **Tool Template** - 工具开发模板
- **Command Template** - 命令开发模板
- **Service Template** - 服务开发模板

## 使用指南

1. **初学者**: 从 `concepts/` 开始，理解核心概念
2. **开发者**: 查看 `entities/` 了解具体实现
3. **深度分析**: 阅读 `sources/` 源码解读
4. **对比学习**: 参考 `comparisons/` 理解设计选择

## 技术栈

- **Runtime**: Bun (>=1.3.0)
- **Language**: TypeScript
- **UI**: React Ink (Terminal)
- **Build**: Bun.build + Vite
- **Package**: 17 workspace packages

## 项目信息

- **仓库**: github.com/claude-code-best/claude-code
- **版本**: v2.8.1
- **代码量**: ~80K 行 TypeScript
- **组件数**: 149 个 React Ink 组件
- **工具数**: 60+ 内置工具

---

*最后更新: 2026-06-23*
