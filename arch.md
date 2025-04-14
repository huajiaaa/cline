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


## Task类关键功能实现

### 1. Task 执行系统概述

Task 类是负责执行 AI 请求和工具操作的核心类。每个任务都在 Task 类的独立实例中运行，确保隔离性和正确的状态管理。

### 2. 核心执行循环

```typescript
class Task {
  async initiateTaskLoop(userContent: UserContent, isNewTask: boolean) {
    while (!this.abort) {
      // 1. 发起 API 请求并流式处理响应
      const stream = this.attemptApiRequest()
      
      // 2. 解析并展示内容块
      for await (const chunk of stream) {
        switch (chunk.type) {
          case "text":
            // 解析成内容块
            this.assistantMessageContent = parseAssistantMessage(chunk.text)
            // 向用户展示块
            await this.presentAssistantMessage()
            break
        }
      }
      
      // 3. 等待工具执行完成
      await pWaitFor(() => this.userMessageContentReady)
      
      // 4. 使用工具结果继续循环
      const recDidEndLoop = await this.recursivelyMakeClineRequests(
        this.userMessageContent
      )
    }
  }
}
```


### 3. 工具执行流程

```typescript
class Task {
  async executeToolWithApproval(block: ToolBlock) {
    // 1. 检查自动批准设置
    if (this.shouldAutoApproveTool(block.name)) {
      await this.say("tool", message)
      this.consecutiveAutoApprovedRequestsCount++
    } else {
      // 2. 请求用户批准
      const didApprove = await askApproval("tool", message)
      if (!didApprove) {
        this.didRejectTool = true
        return
      }
    }

    // 3. 执行工具
    const result = await this.executeTool(block)

    // 4. 保存检查点
    await this.saveCheckpoint()

    // 5. 返回结果给 API
    return result
  }
}
```

### 4. 错误处理和恢复

```typescript
class Task {
  async handleError(action: string, error: Error) {
    // 1. 检查任务是否被放弃
    if (this.abandoned) return
    
    // 2. 格式化错误消息
    const errorString = `Error ${action}: ${error.message}`
    
    // 3. 向用户展示错误
    await this.say("error", errorString)
    
    // 4. 添加错误到工具结果
    pushToolResult(formatResponse.toolError(errorString))
    
    // 5. 清理资源
    await this.diffViewProvider.revertChanges()
    await this.browserSession.closeBrowser()
  }
}
```

### 5. 上下文管理

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

![image](https://github.com/user-attachments/assets/76e1cdbb-f424-4d0f-9af2-d063f375d924)

![image](https://github.com/user-attachments/assets/9284c5db-2c6b-41bc-8e41-bfd019b4a9f8)


### 6. API 请求和令牌管理

```typescript
class Task {
  async *attemptApiRequest(previousApiReqIndex: number): ApiStream {
    // 1. 等待 MCP 服务器连接
    await pWaitFor(() => this.controllerRef.deref()?.mcpHub?.isConnecting !== true)

    // 2. 管理上下文窗口
    const previousRequest = this.clineMessages[previousApiReqIndex]
    if (previousRequest?.text) {
      const { tokensIn, tokensOut } = JSON.parse(previousRequest.text || "{}")
      const totalTokens = (tokensIn || 0) + (tokensOut || 0)
      
      // 如果接近上下文限制则截断对话
      if (totalTokens >= maxAllowedSize) {
        this.conversationHistoryDeletedRange = this.contextManager.getNextTruncationRange(
          this.apiConversationHistory,
          this.conversationHistoryDeletedRange,
          totalTokens / 2 > maxAllowedSize ? "quarter" : "half"
        )
      }
    }

    // 3. 处理流式传输和自动重试
    try {
      this.isWaitingForFirstChunk = true
      const firstChunk = await iterator.next()
      yield firstChunk.value
      this.isWaitingForFirstChunk = false
      
      // 流式传输剩余块
      yield* iterator
    } catch (error) {
      // 4. 错误处理和重试
      if (isOpenRouter && !this.didAutomaticallyRetryFailedApiRequest) {
        await setTimeoutPromise(1000)
        this.didAutomaticallyRetryFailedApiRequest = true
        yield* this.attemptApiRequest(previousApiReqIndex)
        return
      }
      
      // 5. 如果自动重试失败则询问用户是否重试
      const { response } = await this.ask(
        "api_req_failed",
        this.formatErrorWithStatusCode(error)
      )
      if (response === "yesButtonClicked") {
        await this.say("api_req_retried")
        yield* this.attemptApiRequest(previousApiReqIndex)
        return
      }
    }
  }
}
```

maxAllowedSize = Math.max(contextWindow - 40_000, contextWindow * 0.8)

![image](https://github.com/user-attachments/assets/fb29c900-8516-40a1-af6b-88763cc6a8bf)

messagesToRemove = Math.floor((messages.length - startOfRest) / 4) * 2


https://us.cloud.langfuse.com/project/cm9b7c5n001k8ad08t3scbv0l/traces/008184f0-693a-431d-b5e7-29bf015e7d27

### 7. 任务状态和恢复

```typescript
class Task {
  async resumeTaskFromHistory() {
    // 1. 加载保存的状态
    this.clineMessages = await getSavedClineMessages(this.getContext(), this.taskId)
    this.apiConversationHistory = await getSavedApiConversationHistory(this.getContext(), this.taskId)

    // 2. 处理中断的工具执行
    const lastMessage = this.apiConversationHistory[this.apiConversationHistory.length - 1]
    if (lastMessage.role === "assistant") {
      const toolUseBlocks = content.filter(block => block.type === "tool_use")
      if (toolUseBlocks.length > 0) {
        // 添加中断的工具响应
        const toolResponses = toolUseBlocks.map(block => ({
          type: "tool_result",
          tool_use_id: block.id,
          content: "Task was interrupted before this tool call could be completed."
        }))
        modifiedOldUserContent = [...toolResponses]
      }
    }

    // 3. 通知中断
    const agoText = this.getTimeAgoText(lastMessage?.ts)
    newUserContent.push({
      type: "text",
      text: `[TASK RESUMPTION] This task was interrupted ${agoText}. It may or may not be complete, so please reassess the task context.`
    })

    // 4. 恢复任务执行
    await this.initiateTaskLoop(newUserContent, false)
  }

  private async saveTaskState() {
    // 保存对话历史
    await saveApiConversationHistory(this.getContext(), this.taskId, this.apiConversationHistory)
    await saveClineMessages(this.getContext(), this.taskId, this.clineMessages)
    
    // 创建检查点
    const commitHash = await this.checkpointTracker?.commit()
    
    // 更新任务历史
    await this.controllerRef.deref()?.updateTaskHistory({
      id: this.taskId,
      ts: lastMessage.ts,
      task: taskMessage.text,
      // ... 其他元数据
    })
  }
}
```

### 8. 终端管理

```typescript
class Task {
  async executeCommandTool(command: string): Promise<[boolean, ToolResponse]> {
    // 1. 获取或创建终端
    const terminalInfo = await this.terminalManager.getOrCreateTerminal(cwd)
    terminalInfo.terminal.show()

    // 2. 执行命令并流式输出
    const process = this.terminalManager.runCommand(terminalInfo, command)
    
    // 3. 处理实时输出
    let result = ""
    process.on("line", (line) => {
      result += line + "\n"
      if (!didContinue) {
        sendCommandOutput(line)
      } else {
        this.say("command_output", line)
      }
    })

    // 4. 等待完成或用户反馈
    let completed = false
    process.once("completed", () => {
      completed = true
    })

    await process

    // 5. 返回结果
    if (completed) {
      return [false, `Command executed.\n${result}`]
    } else {
      return [
        false,
        `Command is still running in the user's terminal.\n${result}\n\nYou will be updated on the terminal status and new output in the future.`
      ]
    }
  }
}
```

### 9. 浏览器会话管理

```typescript
class Task {
  async executeBrowserAction(action: BrowserAction): Promise<BrowserActionResult> {
    switch (action) {
      case "launch":
        // 1. 启动固定分辨率的浏览器
        await this.browserSession.launchBrowser()
        return await this.browserSession.navigateToUrl(url)

      case "click":
        // 2. 处理带坐标的点击操作
        return await this.browserSession.click(coordinate)

      case "type":
        // 3. 处理键盘输入
        return await this.browserSession.type(text)

      case "close":
        // 4. 清理资源
        return await this.browserSession.closeBrowser()
    }
  }
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

2. **productContext.md**

3. **activeContext.md**

4. **systemPatterns.md**

5. **techContext.md**

6. **progress.md**

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

