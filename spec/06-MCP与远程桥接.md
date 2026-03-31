# 06. MCP 与远程桥接

## 1. MCP 体系总览

当前代码对 MCP 采用双向设计：

- 作为 MCP Client
- 作为 MCP Server

这意味着 Claude Code 既能消费外部 MCP 能力，也能把自身工具暴露给外部客户端。

## 2. 作为 MCP Client

当前主入口包括：

- `src/services/mcp/client.ts`
- `src/services/mcp/config.ts`
- `src/services/mcp/types.ts`
- `src/services/mcp/auth.ts`
- `src/services/mcp/useManageMCPConnections.ts`

从当前实现可确认的传输与接入形态包括：

- `stdio`
- `sse`
- `sse-ide`
- `ws-ide`
- `http`
- `ws`
- `sdk`
- `claudeai-proxy`

### 2.1 MCP Client 的职责

- 解析 MCP 配置
- 建立连接
- 拉取 tools / prompts / resources
- 从 prompts 生成 commands
- 从 skill 资源构造 skill commands
- 在需要鉴权时注入 auth 相关状态与工具
- 把结果写入 `AppState.mcp`

### 2.2 MCP 进入运行时的数据流

```text
MCP 配置
  -> services/mcp/config.ts
  -> services/mcp/client.ts
  -> 建立连接
  -> 拉取 tools / prompts / resources
  -> prompts -> commands
  -> skill resources -> skill commands
  -> resources -> AppState.mcp.resources
  -> tools / commands / resources 合并进运行时
```

## 3. 作为 MCP Server

入口文件：

- `src/entrypoints/mcp.ts`

它会把 Claude Code 的内建工具重新包装成 MCP 协议服务。

当前主能力：

- `ListTools`
- `CallTool`

需要注意：

- 当前实现重点是暴露内建工具
- 不应默认理解为“把所有已连接外部 MCP server 再次透传出去”

## 4. Remote、Bridge、Direct Connect、Assistant Viewer 的边界

这些能力都属于“远端会话相关”，但不是同一条实现链。

### 4.1 Remote session

关键入口：

- `src/remote/RemoteSessionManager.ts`
- `src/remote/SessionsWebSocket.ts`
- `src/remote/sdkMessageAdapter.ts`
- `src/hooks/useRemoteSession.ts`

职责：

- 通过 HTTP 发送用户消息
- 通过 WebSocket 订阅远端事件
- 处理远端权限请求与中断
- 把 SDK message 适配回本地 REPL 消息

### 4.1.1 SSH session

这条线和 remote/direct-connect 不同，它不是 WebSocket 连接管理，
而是“本地 UI + 远端工具执行”的另一种接入形态。

当前工作树里，REPL 侧真实存在的入口包括：

- `src/hooks/useSSHSession.ts`

它负责：

- 接收启动阶段创建好的 `SSHSession`
- 把 SSH 子进程消息适配回本地 REPL
- 转发远端 permission request
- 以与 `useDirectConnect()` 类似的接口接入 REPL

### 4.2 Bridge / remote-control

关键入口：

- `src/bridge/initReplBridge.ts`
- `src/bridge/replBridge.ts`
- `src/bridge/remoteBridgeCore.ts`
- `src/bridge/bridgeMessaging.ts`
- `src/hooks/useReplBridge.tsx`

职责：

- 把本地环境注册成 bridge 目标
- 接收 inbound message / control 请求
- 将本地执行结果同步回远端
- 协调 interrupt / model / thinking / permission 等控制信息

### 4.3 Direct connect

当前工作树里能确认的 direct connect 入口是：

- `src/server/createDirectConnectSession.ts`
- `src/server/directConnectManager.ts`
- `src/hooks/useDirectConnect.ts`

职责：

- 创建 direct-connect session
- 管理 direct-connect WebSocket
- 把会话接入 REPL

### 4.4 Assistant viewer

当前工作树里真实存在、且可以确认的 viewer 相关入口是：

- `src/hooks/useAssistantHistory.ts`
- `src/assistant/sessionHistory.ts`

这条线当前能明确确认的是：

- viewer 模式会分页读取远端 session events
- 历史数据通过 `/v1/sessions/{sessionId}/events` 这条链路进入本地 REPL

## 5. 远程主链路

### 5.1 Remote mode

```text
本地输入
  -> REPL.onSubmit()
  -> remote-safe command 过滤
  -> activeRemote.sendMessage()
  -> HTTP POST 到 remote session
  -> WebSocket 订阅远端事件
  -> 远端执行
  -> SDKMessage 回流
  -> sdkMessageAdapter
  -> useRemoteSession
  -> 本地 setMessages / setAppState
```

### 5.2 Bridge mode

```text
bridge 初始化
  -> useReplBridge
  -> initReplBridge
  -> 建立 bridge 会话
  -> 接收 inbound message / control
  -> 注入本地执行面
  -> 结果写回 bridge session
  -> 远端看到同步状态与输出
```

## 6. 当前工作树异常

下面这些远程相关文件**在源码中仍被引用**，但当前工作树缺失：

- `connectHeadless`
- `createSSHSession`
- `sessionDiscovery`
- `AssistantSessionChooser`

这意味着：

- `main.tsx` 中仍能看到这些动态导入
- 但它们不能作为当前可落地的阅读入口写入导航文档

## 7. Chrome / 浏览器相关桥接

当前 `cli.tsx` 仍保留浏览器相关 fast-path：

- `--claude-in-chrome-mcp`
- `--chrome-native-host`

它们说明 Claude Code 不只服务于终端本身，还包含浏览器侧接入适配。

## 8. 这一层的架构意义

MCP 与远程桥接层共同说明：

Claude Code 不是只在本地终端里工作的单体 CLI，而是一个可以：

- 接入外部能力
- 暴露自身能力
- 接受远端控制
- 支持多端会话协同

的代理运行平台。

## 9. 相关源码入口

- `src/services/mcp/client.ts`
- `src/services/mcp/config.ts`
- `src/services/mcp/auth.ts`
- `src/services/mcp/types.ts`
- `src/services/mcp/useManageMCPConnections.ts`
- `src/entrypoints/mcp.ts`
- `src/remote/RemoteSessionManager.ts`
- `src/bridge/initReplBridge.ts`
- `src/server/createDirectConnectSession.ts`
- `src/assistant/sessionHistory.ts`

## 10. 文档导航

- 上一篇：`05-命令-Skills-插件.md`
- 索引：`README.md`
- 下一篇：`07-接口与协议.md`
- 配套阅读：`09-关键时序图.md`、`15-常见问题定位手册.md`
