# API 层 (API Layer)

## 🌐 API 架构

CCB 支持 **7 种 API 提供商**，通过统一的接口抽象：

```
┌────────────────────────────────────────────────────────────┐
│                    Provider Selector                       │
│                   (src/utils/model/providers.ts)            │
└───────────────────────┬────────────────────────────────────┘
                        ↓
┌────────────────────────────────────────────────────────────┐
│                  Provider Abstraction                       │
│                  (@ant/model-provider)                       │
└───────────────────────┬────────────────────────────────────┘
                        ↓
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ firstParty   │ │  bedrock     │ │   vertex     │
│ (Anthropic)  │ │   (AWS)      │ │  (Google)    │
└──────────────┘ └──────────────┘ └──────────────┘
        ↓               ↓               ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│    openai    │ │   gemini     │ │    grok      │
│  (OpenAI)    │ │  (Google)    │ │   (xAI)      │
└──────────────┘ └──────────────┘ └──────────────┘
                        ↓
                ┌──────────────┐
                │   foundry    │
                │  (内部平台)  │
                └──────────────┘
```

## 🔑 Provider 列表

### 1. firstParty (Anthropic 官方)

**SDK**: `@anthropic-ai/sdk`

**特点**:
- 原生支持
- 所有功能可用
- 支持 extended thinking

**配置**:
```json
{
  "apiKey": "sk-ant-...",
  "baseURL": "https://api.anthropic.com"
}
```

### 2. bedrock (AWS)

**SDK**: `@aws-sdk/client-bedrock-runtime`

**特点**:
- AWS 部署
- 区域端点
- IAM 认证

**配置**:
```json
{
  "provider": "bedrock",
  "region": "us-east-1",
  "credentials": {
    "accessKeyId": "...",
    "secretAccessKey": "..."
  }
}
```

### 3. vertex (Google Cloud)

**SDK**: `@anthropic-ai/vertex-sdk`

**特点**:
- GCP 集成
- OAuth 2.0
- 区域端点

**配置**:
```json
{
  "provider": "vertex",
  "projectId": "...",
  "region": "us-central1"
}
```

### 4. openai (兼容)

**SDK**: `openai`

**特点**:
- OpenAI API 格式
- 兼容第三方
- 非 Anthropic 官方

**配置**:
```json
{
  "provider": "openai",
  "baseURL": "https://api.openai.com/v1",
  "apiKey": "sk-..."
}
```

### 5. gemini (兼容)

**SDK**: `Anthropic SDK` + 转换层

**特点**:
- Google Gemini
- 格式转换
- 非 Anthropic 官方

**配置**:
```json
{
  "provider": "gemini",
  "baseURL": "...",
  "apiKey": "..."
}
```

### 6. grok (xAI)

**SDK**: OpenAI 兼容

**特点**:
- xAI Grok 模型
- OpenAI 格式
- 非 Anthropic 官方

**配置**:
```json
{
  "provider": "grok",
  "baseURL": "https://api.x.ai/v1",
  "apiKey": "..."
}
```

### 7. foundry

**SDK**: `@anthropic-ai/foundry-sdk`

**特点**:
- Anthropic 内部平台
- 企业级功能
- 特殊权限

**配置**:
```json
{
  "provider": "foundry",
  "environment": "production"
}
```

## 🔧 Provider 选择逻辑

```typescript
// src/utils/model/providers.ts
function selectProvider(modelType?: string): Provider {
  // 1. 优先: 模型指定
  if (modelType?.startsWith('bedrock')) return bedrockProvider
  if (modelType?.startsWith('vertex')) return vertexProvider
  // ...

  // 2. 其次: 环境变量
  if (process.env.ANTHROPIC_BASE_URL) return customProvider

  // 3. 默认: firstParty
  return firstPartyProvider
}
```

## 📦 API 调用流程

```
QueryEngine
    ↓
query.ts
    ↓
services/api/claude.ts
    ↓
Provider Selector
    ↓
Specific Provider SDK
    ↓
External API
    ↓
Streaming Response
```

## 🌊 流式处理

```typescript
// 流式响应处理
async function streamQuery(params) {
  const stream = await client.messages.create(params)

  for await (const event of stream) {
    switch (event.type) {
      case 'content_block_delta':
        // 内容增量
        onContentDelta(event.delta)
        break
      case 'content_block_stop':
        // 内容块结束
        onContentStop()
        break
      case 'tool_use':
        // 工具调用
        onToolUse(event.input)
        break
      // ...
    }
  }
}
```

## 🔐 认证方式

| Provider | 认证方式 |
|----------|----------|
| firstParty | API Key |
| bedrock | AWS Credentials |
| vertex | OAuth 2.0 / Service Account |
| openai | API Key |
| gemini | API Key |
| grok | API Key |
| foundry | Internal Auth |

## 📊 模型映射

### Claude 模型

| 模型 ID | 名称 | 上下文 |
|---------|------|--------|
| claude-opus-4-6 | Opus 4.6 | 200K |
| claude-sonnet-4-6 | Sonnet 4.6 | 200K |
| claude-haiku-4-5-20251001 | Haiku 4.5 | 200K |

### 兼容模型

```typescript
// 模型类型转换
function mapModel(provider: Provider, model: string): string {
  switch (provider) {
    case 'openai':
      return model.replace('claude-', 'gpt-')
    case 'gemini':
      return model.replace('claude-', 'gemini-')
    // ...
  }
}
```

## 🔗 相关模块

- [核心循环](core-loop.md) - API 如何被调用
- [状态管理](state-management.md) - 配置如何存储
- [工具系统](tool-system.md) - 工具如何使用 API

---

*下一步: [Bridge Protocol](bridge-protocol.md)*
