# 状态管理 (State Management)

## 🗄️ 状态架构

CCB 使用**混合状态管理**策略:
- **全局状态**: Bootstrap state + Zustand store
- **组件状态**: React hooks
- **会话状态**: QueryEngine 内部状态

```
┌────────────────────────────────────────────────────────────┐
│                   Bootstrap State                           │
│              (模块级单例 - bootstrap/state.ts)               │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  • sessionId: string                                 │ │
│  │  • cwd: string                                       │ │
│  │  • projectRoot: string                               │ │
│  │  • tokenCounts: TokenCounts                         │ │
│  │  • modelOverrides: ModelConfig                      │ │
│  │  • clientType: ClientType                            │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                   AppState (Zustand)                        │
│                  (React 全局状态)                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  • messages: Message[]                                │ │
│  │  • tools: Tools                                       │ │
│  │  • permissions: PermissionState                       │ │
│  │  • mcpConnections: MCPServerConnection[]              │ │
│  │  • currentPlan: Plan | null                           │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                 QueryEngine State                          │
│                (查询引擎内部状态)                           │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  • turnCount: number                                 │ │
│  │  • fileStateCache: FileStateCache                   │ │
│  │  • attributionState: AttributionState               │ │
│  │  • autoCompactState: AutoCompactTrackingState       │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                   Component State                          │
│                  (React 组件本地状态)                       │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  • input: string                                     │ │
│  │  • isProcessing: boolean                            │ │
│  │  • selectedMessage: Message | null                 │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
```

## 🔑 状态层级

### 1. Bootstrap State (启动状态)

**文件**: `src/bootstrap/state.ts`

**特点**:
- 模块级单例
- 进程生命周期
- 跨模块共享

**主要内容**:
```typescript
// 会话标识
let sessionId: string

// 工作目录
let cwd: string
let projectRoot: string

// Token 计数
let totalInputTokens: number = 0
let totalOutputTokens: number = 0

// 模型覆盖
let modelOverrides: ModelConfig = {}

// 客户端类型
let clientType: ClientType = 'cli'
```

**用途**:
- 跨模块访问无需传参
- 全局配置存储
- 会话级别统计

### 2. AppState (Zustand Store)

**文件**: `src/state/AppState.tsx`, `src/state/store.ts`

**特点**:
- React Context 传递
- 响应式更新
- 组件订阅

**主要内容**:
```typescript
interface AppState {
  // 消息历史
  messages: Message[]

  // 工具列表
  tools: Tools

  // 权限状态
  permissions: {
    mode: PermissionMode
    pendingApprovals: PendingApproval[]
  }

  // MCP 连接
  mcpConnections: Map<string, MCPServerConnection>

  // 当前计划 (Plan Mode)
  currentPlan: Plan | null

  // 统计信息
  stats: {
    totalCost: number
    totalTokens: number
    turnCount: number
  }

  // 操作方法
  addMessage: (message: Message) => void
  updateTools: (tools: Tools) => void
  setPermissionMode: (mode: PermissionMode) => void
}
```

**用法**:
```typescript
// 在组件中使用
const messages = useAppState(state => state.messages)
const addMessage = useAppState(state => state.addMessage)

// 直接使用 store
import { store } from './state/store.js'
store.getState().addMessage(msg)
```

### 3. QueryEngine State

**文件**: `src/QueryEngine.ts`

**特点**:
- 实例级别
- 查询生命周期
- 不直接暴露给 UI

**主要内容**:
```typescript
class QueryEngine {
  private turnCount: number = 0
  private fileStateCache: FileStateCache
  private attributionState: AttributionState
  private autoCompactState: AutoCompactTrackingState
}
```

**用途**:
- 追踪查询回合
- 管理文件快照
- 记录修改归因

### 4. Component State

**特点**:
- 组件本地
- React useState/useRef
- UI 交互状态

**示例**:
```typescript
function REPL() {
  const [input, setInput] = useState('')
  const [isProcessing, setIsProcessing] = useState(false)
  const selectedMessage = useRef<Message | null>(null)

  // ...
}
```

## 🔄 状态更新流程

```
用户操作
   ↓
Component State 更新
   ↓
调用 Action (addMessage, updateTools, etc.)
   ↓
AppState 更新 (Zustand)
   ↓
订阅组件重新渲染
   ↓
Bootstrap State 更新 (如需要)
```

## 📊 关键状态数据

### Message (消息)

```typescript
interface Message {
  id: string
  role: 'user' | 'assistant' | 'system'
  content: ContentBlock[]
  timestamp: number
  metadata?: {
    turnId?: string
    toolCalls?: ToolUse[]
    cost?: number
    tokens?: number
  }
}
```

### ToolUse (工具调用)

```typescript
interface ToolUse {
  id: string
  name: string
  input: JSONObject
  result?: string
  status: 'pending' | 'approved' | 'denied' | 'completed'
  error?: string
}
```

### PermissionState (权限)

```typescript
interface PermissionState {
  mode: PermissionMode  // 'auto' | 'ask' | 'read-only' | 'custom'
  pendingApprovals: PendingApproval[]
  deniedTools: Set<string>
  autoApprovedTools: Set<string>
}
```

## 🧩 状态持久化

### 会话存储

**文件**: `src/utils/sessionStorage.ts`

**存储内容**:
- 消息历史
- 会话配置
- Token 统计
- 文件状态缓存

**位置**:
```
~/.claude/sessions/<sessionId>/
├── transcript.jsonl    # 消息历史
├── state.json          # 会话状态
└── cache.json          # 文件缓存
```

### 配置存储

**位置**:
```
~/.claude/
├── config.json         # 全局配置
├── settings.json       # 项目设置
└── memory/            # 记忆存储
```

## 🔗 相关模块

- [核心循环](core-loop.md) - 状态如何流转
- [工具系统](tool-system.md) - 工具状态管理
- [API 层](api-layer.md) - API 调用状态

---

*下一步: [API 层](api-layer.md)*
