# Cline扩展架构

## Cline 核心功能
 
 Cline是一个VSCode扩展，通过核心扩展后端和基于React的webview前端提供AI辅助功能。该扩展使用TypeScript构建，以下是 Cline 核心功能和逻辑的概述：
 
 ### 1. 任务初始化
 
 -   **任务创建**: 当用户提供任务描述和可选的图片时，Cline 会创建一个新的任务，并生成一个唯一的任务 ID。
 -   **历史任务恢复**: Cline 允许用户从历史记录中恢复任务。恢复任务时，Cline 会加载之前的对话历史、消息记录等信息。
 
 ### 2. 对话管理
 
 -   **API 对话历史**: Cline 使用 `apiConversationHistory` 存储与 API 提供者（例如 OpenAI、Anthropic）的对话历史。
 -   **Cline 消息**: Cline 使用 `clineMessages` 存储与用户的交互消息，包括用户输入、助手回复、工具使用情况等。
 -   **消息存储**: Cline 会将 API 对话历史和 Cline 消息存储到磁盘，以便在任务恢复时加载。
 
 ### 3. 核心循环：`recursivelyMakeClineRequests`
 
 -   **用户输入处理**: Cline 接收用户输入（文本、图片等），并将其转换为 API 可以理解的格式。
 -   **环境信息收集**: Cline 收集当前的工作区环境信息，例如可见文件、打开的标签页、终端状态、诊断信息等，并将这些信息添加到上下文中。
 -   **API 请求**: Cline 使用 `attemptApiRequest` 函数向 API 提供者发送请求，并获取助手的回复。
 -   **助手消息处理**: Cline 解析助手返回的消息，提取文本、工具使用信息等。
 -   **工具使用**: 如果助手建议使用工具，Cline 会根据工具的类型执行相应的操作，例如：
     -   **文件读写**: 读取或写入文件内容。
     -   **命令行执行**: 在终端中执行命令，并获取输出。
     -   **代码定义查找**: 查找代码定义。
     -   **文件搜索**: 根据正则表达式搜索文件。
     -   **浏览器操作**: 启动浏览器，执行点击、输入等操作。
 -   **用户反馈**: Cline 呈现助手消息和工具执行结果给用户，并等待用户反馈。用户可以批准、拒绝或提供反馈意见。
 -   **循环迭代**: Cline 根据用户反馈和任务状态，决定是否继续执行工具、完成任务或重新评估任务。
 
 ### 4. 上下文管理
 
 -   **上下文截断**: 为了避免上下文窗口超出限制，Cline 会根据需要截断对话历史。
 -   **提及解析**: Cline 会解析用户输入中的提及（例如 `@file`、`@url`），并将相应的内容添加到上下文中。
 
 ### 5. 状态管理
 
 -   **检查点**: Cline 会在任务执行过程中创建检查点，以便用户可以回滚到之前的状态。
 -   **Diff 视图**: Cline 使用 Diff 视图向用户展示文件变更，并允许用户编辑或还原更改。
 
 ### 6. 其他功能
 
 -   **自动审批**: Cline 可以根据用户设置自动批准某些工具的使用。
 -   **忽略列表**: Cline 尊重 `.clineignore` 文件，避免访问被忽略的文件和目录。
 -   **自定义指令**: 用户可以通过 `.clinerules` 文件或设置自定义指令来定制 Cline 的行为。
 
 ### 7. 终端管理
 
 -   **终端创建和管理**: Cline 能够创建和管理 VS Code 的集成终端，用于执行 CLI 命令。
 -   **命令执行**: Cline 可以在终端中执行命令，并实时获取命令输出。
 -   **Shell 集成**: Cline 利用 VS Code 的 Shell 集成 API，实现与终端的无缝集成。
 
 ### 8. 浏览器管理
 
 -   **浏览器会话**: Cline 能够启动和控制无头浏览器，用于执行 Web 相关的任务。
 -   **浏览器操作**: Cline 可以模拟用户在浏览器中的操作，例如点击、输入、滚动等。
 -   **截图和日志**: Cline 可以捕获浏览器截图和控制台日志，用于调试和测试。
 
 ### 9. 错误处理
 
 -   **API 错误**: Cline 能够处理 API 请求失败的情况，并向用户提供重试选项。
 -   **工具执行错误**: Cline 能够捕获工具执行过程中的错误，并向用户报告。
 
 ### 10. 模式切换
 
 -   **计划模式**: 在计划模式下，Cline 专注于信息收集、问题提问和解决方案设计。该模式下可以使用 plan_mode_respond 工具。

 -   **执行模式**: 在执行模式下，Cline 执行具体的任务步骤。可以使用除 plan_mode_respond 之外的所有工具。


## 主要流程
![image](https://github.com/user-attachments/assets/6272e0d7-2460-415f-9f9a-308637d1dcf1)

模型的返回包含 thinking（plan 模式下不需要 thinking） 和 execute_command（即要使用的 tool 两部分），如果没有使用 tool，会再伪造一个 user requirement，提示它要使用 tool，发给大模型。

模型使用 attempt_completion tool 来结束当前任务循环，等待用户反馈

[https://us.cloud.langfuse.com/project/cm9b7c5n001k8ad08t3scbv0l/traces/62a66175-b536-48f7-93b9-41cb47c898ad](https://us.cloud.langfuse.com/project/cm9b7c5n001k8ad08t3scbv0l/traces/ab64bc93-79fa-41dc-98e6-82b117f37b40)

一次只能使用一个 tool


- @file，把选中文件的内容给大模型 （在 prompt 中提示 see below for file content）
- @Folder，把文件目录结构给大模型
- @problem，把当前工作区的诊断信息给大模型，例如 Pylance Error 等
- @url， 用 puppeteer 获取链接内容给大模型
- @terminal，把 Terminal Output 给大模型
- @git commits，把提交对应的 commit msg，以及该提交的 diff 给大模型

## 架构概述

### 核心扩展架构

核心扩展遵循明确的层次结构：
1. **WebviewProvider**(src/core/webview/index.ts)：管理webview生命周期和通信
2. **Controller**(src/core/controller/index.ts)：处理webview消息和任务管理
3. **Task**(src/core/task/index.ts)：执行API请求和工具操作


# Task类实现总结

## 关键功能实现

### 任务执行循环

```typescript
async initiateTaskLoop(userContent: UserContent): Promise<void> {
  // 循环执行直到任务被中止
  while (!this.abort) {
    // 发送API请求并获取流式响应
    const stream = this.attemptApiRequest()
    
    // 处理流式响应
    for await (const chunk of stream) {
      // 解析和呈现内容块
      // ...
    }
    
    // 等待用户消息内容就绪
    await pWaitFor(() => this.userMessageContentReady)
    
    // 递归处理用户内容
    const recDidEndLoop = await this.recursivelyMakeClineRequests(
      this.userMessageContent
    )
  }
}
```

### API请求处理

```typescript
async *attemptApiRequest(previousApiReqIndex: number): ApiStream {
  // 等待MCP服务器连接
  await pWaitFor(() => this.controllerRef.deref()?.mcpHub?.isConnecting !== true)

  // 管理上下文窗口
  const previousRequest = this.clineMessages[previousApiReqIndex]
  if (previousRequest?.text) {
    const { tokensIn, tokensOut } = JSON.parse(previousRequest.text || "{}")
    const totalTokens = (tokensIn || 0) + (tokensOut || 0)
    
    // 如果接近上下文限制，截断对话
    if (totalTokens >= maxAllowedSize) {
      this.conversationHistoryDeletedRange = this.contextManager.getNextTruncationRange(
        this.apiConversationHistory,
        this.conversationHistoryDeletedRange,
        totalTokens / 2 > maxAllowedSize ? "quarter" : "half"
      )
    }
  }

  // 处理流式响应和错误
  // ...
}
```

![image](https://github.com/user-attachments/assets/76e1cdbb-f424-4d0f-9af2-d063f375d924)

![image](https://github.com/user-attachments/assets/9284c5db-2c6b-41bc-8e41-bfd019b4a9f8)


maxAllowedSize = Math.max(contextWindow - 40_000, contextWindow * 0.8)

![image](https://github.com/user-attachments/assets/fb29c900-8516-40a1-af6b-88763cc6a8bf)

messagesToRemove = Math.floor((messages.length - startOfRest) / 4) * 2


https://us.cloud.langfuse.com/project/cm9b7c5n001k8ad08t3scbv0l/traces/008184f0-693a-431d-b5e7-29bf015e7d27

### 工具执行

```typescript
async executeCommandTool(command: string): Promise<[boolean, ToolResponse]> {
  // 获取或创建终端
  const terminalInfo = await this.terminalManager.getOrCreateTerminal(cwd)
  terminalInfo.terminal.show()

  // 执行命令并处理输出
  const process = this.terminalManager.runCommand(terminalInfo, command)
  
  // 处理实时输出
  let result = ""
  process.on("line", (line) => {
    result += line + "\n"
    if (!didContinue) {
      sendCommandOutput(line)
    } else {
      this.say("command_output", line)
    }
  })

  // 等待完成或用户反馈
  // ...
}
```

### 上下文管理

```typescript
async getEnvironmentDetails(includeFileDetails: boolean = false) {
  // 获取环境详情
  // ...
  
  // 计算上下文窗口使用情况
  const lastApiReqMessage = this.clineMessages.find(msg => {
    return getTotalTokensFromApiReqMessage(msg) > 0
  })

  const lastApiReqTotalTokens = lastApiReqMessage ? getTotalTokensFromApiReqMessage(lastApiReqMessage) : 0
  const usagePercentage = Math.round((lastApiReqTotalTokens / contextWindow) * 100)

  // 添加当前模式信息
  details += "\n\n# Current Mode"
  if (this.chatSettings.mode === "plan") {
    details += "\nPLAN MODE\n" + formatResponse.planModeInstructions()
  } else {
    details += "\nACT MODE"
  }

  return `<environment_details>\n${details.trim()}\n</environment_details>`
}
```


## MCP(模型上下文协议)集成

### MCP架构
MCP系统由以下部分组成：
1. **McpHub类**：src/services/mcp/McpHub.ts中的中央管理器
2. **MCP连接**：管理与外部MCP服务器的连接
3. **MCP设置**：存储在JSON文件中的配置
4. **MCP市场**：可用MCP服务器的在线目录
5. **MCP工具与资源**：连接的服务器公开的功能

MCP系统支持两种类型的服务器连接：
- **Stdio**：通过标准I/O通信的命令行服务器
- **SSE**：通过服务器发送事件(Server-Sent Events)通信的HTTP服务器


## Cline Memory Bank
https://cline.bot/blog/memory-bank-how-to-make-cline-an-ai-agent-that-never-forgets

Cline Memory Bank 是一个专门设计的系统，用于解决 AI 助手在会话之间丢失上下文的问题。它通过强制 Cline 维护完整的项目文档，确保即使在会话重置后，Cline 仍能理解项目背景并继续有效工作。

### 核心概念

Memory Bank 的核心概念是：Cline 的"记忆"在会话之间完全重置，这不是一个限制，而是驱动它维护完美文档的动力。每次重置后，Cline 完全依赖 Memory Bank 来理解项目并继续工作。

### 文件结构

Memory Bank 由必需的核心文件和可选的上下文文件组成，全部采用 Markdown 格式。文件之间通过清晰的层次结构相互关联：

#### 核心文件（必需）

1. **projectbrief.md**
   - 塑造所有其他文件的基础文档
   - 在项目开始时创建（如果不存在）
   - 定义核心需求和目标
   - 项目范围的真相来源

2. **productContext.md**
   - 项目存在的原因
   - 解决的问题
   - 预期的工作方式
   - 用户体验目标

3. **activeContext.md**
   - 当前工作重点
   - 最近的变更
   - 下一步计划
   - 活跃的决策和考虑因素

4. **systemPatterns.md**
   - 系统架构
   - 关键技术决策
   - 使用的设计模式
   - 组件关系

5. **techContext.md**
   - 使用的技术
   - 开发设置
   - 技术约束
   - 依赖关系

6. **progress.md**
   - 已完成的功能
   - 待构建的内容
   - 当前状态
   - 已知问题

#### 额外上下文

当有助于组织时，在 memory-bank/ 中创建额外的文件/文件夹：
- 复杂功能文档
- 集成规范
- API 文档
- 测试策略
- 部署程序

### 核心工作流程

#### 计划模式

在计划模式下，Cline 会：
1. 读取 Memory Bank
2. 检查文件是否完整
3. 如果不完整，创建计划并在聊天中记录
4. 如果完整，验证上下文，制定策略并呈现方法

#### 执行模式

在执行模式下，Cline 会：
1. 检查 Memory Bank
2. 更新文档
3. 根据需要更新 .clinerules
4. 执行任务
5. 记录变更

### 文档更新

Memory Bank 在以下情况下更新：
1. 发现新的项目模式
2. 实施重大变更后
3. 用户请求"update memory bank"时（必须审查所有文件）
4. 需要澄清上下文时

当由"update memory bank"触发时，Cline 必须审查每个 Memory Bank 文件，即使某些文件不需要更新。特别关注 activeContext.md 和 progress.md，因为它们跟踪当前状态。

