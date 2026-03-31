# 03. REPL 与状态管理

## 1. REPL 挂载链路

交互式模式的主要挂载链路是：

- `src/replLauncher.tsx`
- `src/components/App.tsx`
- `src/screens/REPL.tsx`

分工如下：

- `replLauncher.tsx`
  动态装配 `App` 与 `REPL`。

- `components/App.tsx`
  负责顶层 Provider 组合。

- `screens/REPL.tsx`
  负责真实会话屏幕、输入协作、消息回写和远程变体接入。

## 2. 终端 UI 基础设施

终端 UI 运行时主要由这些文件构成：

- `src/ink/root.ts`
- `src/ink/components/App.tsx`
- `src/ink/parse-keypress.ts`
- `src/ink/termio/*`

主要职责：

- 原始输入与按键序列解析
- 鼠标、焦点、终端响应处理
- Raw mode 生命周期管理
- 渲染上下文与终端状态维护

## 3. REPL 的核心职责

`src/screens/REPL.tsx` 是交互式运行时的协调中心。

它负责：

- 维护消息列表和 loading / stream 状态
- 接收 `PromptInput` 提交
- 管理 queued commands、permission dialogs、modal state
- 构造 `ToolUseContext`
- 调用 `query()`
- 把模型事件、工具结果、状态变化回写到 UI

当前代码里，REPL 还会接入多种会话变体：

- `useRemoteSession`
- `useDirectConnect`
- `useSSHSession`
- `useAssistantHistory`
- `useReplBridge`

其中：

- `useSSHSession` 这条线在 REPL 侧存在
- 但其上游 `ssh` 会话创建实现文件在当前工作树缺失

## 4. Prompt 提交流程

### 4.1 本地主链路

```text
用户输入
  -> ink 输入解析
  -> PromptInput / useTextInput
  -> REPL.onSubmit()
  -> handlePromptSubmit()
  -> processUserInput()
  -> onQuery()
  -> query()
  -> setMessages / setAppState
  -> UI 重绘
```

### 4.2 `onSubmit` 的分流职责

`REPL.onSubmit()` 并不是单纯把输入全部转给 `handlePromptSubmit()`。

它还会在前面处理：

- `local-jsx` 命令
- immediate 命令
- speculation accept
- remote / direct connect / ssh 等远程发送路径

因此 `handlePromptSubmit()` 是**默认本地执行路径**的统一入口，不是所有输入来源的唯一入口。

### 4.3 输入旁路

除了键盘输入，当前 REPL 还有几条重要旁路：

- bridge inbound message
  由 `useReplBridge()` 注入 queued command / prompt

- remote session event stream
  由 `useRemoteSession()` 直接更新消息与状态

- direct connect event stream
  由 `useDirectConnect()` 驱动

- assistant viewer 历史分页
  由 `useAssistantHistory()` 从 `/v1/sessions/{id}/events` 拉取历史

## 5. 状态模型

项目采用“两层状态”设计。

### 5.1 进程级全局状态

文件：

- `src/bootstrap/state.ts`

职责：

- 保存跨模块共享的进程级状态
- 提供 session、cwd、成本、模型、计量等全局变量

这层更像运行时注册表。

### 5.2 UI 应用状态

文件：

- `src/state/AppStateStore.ts`
- `src/state/store.ts`
- `src/state/onChangeAppState.ts`
- `src/state/AppState.tsx`

职责：

- 维护交互式会话可响应状态
- 为 React 组件提供读写能力
- 把状态变化同步到权限、配置、环境等副作用

当前 `AppState` 中高频且重要的状态域包括：

- `settings`
- `toolPermissionContext`
- `mcp.clients / mcp.tools / mcp.commands / mcp.resources`
- `plugins`
- `tasks`
- `todos`
- `notifications`
- `thinkingEnabled`
- `remoteConnectionStatus`
- `remoteBackgroundTaskCount`
- `replBridge*`

## 6. Store 机制

`src/state/store.ts` 提供极简 store：

- `getState()`
- `setState(updater)`
- `subscribe(listener)`

这不是 Redux / Zustand，而是仓库内自定义的轻量实现。

## 7. AppState 副作用同步

`src/state/onChangeAppState.ts` 是状态副作用同步点。

当前可确认的职责包括：

- 权限模式同步
- settings 写回
- 认证缓存相关清理
- 环境变量重新应用

所以 AppState 不只是 UI 展示状态，它会直接影响真实运行行为。

## 8. 当前工作树异常

assistant viewer 与 ssh 相关链路在 REPL 层有两类现象需要分开看：

- 当前可确认实现：
  - `src/hooks/useAssistantHistory.ts`
  - `src/assistant/sessionHistory.ts`
  - `src/hooks/useSSHSession.ts`

- 当前仍有源码引用、但实现缺失：
  - `sessionDiscovery`
  - `createSSHSession`

## 9. 相关源码入口

- `src/replLauncher.tsx`
- `src/components/App.tsx`
- `src/screens/REPL.tsx`
- `src/components/PromptInput/PromptInput.tsx`
- `src/hooks/useTextInput.ts`
- `src/state/AppStateStore.ts`
- `src/state/store.ts`
- `src/state/onChangeAppState.ts`
- `src/bootstrap/state.ts`

## 10. 文档导航

- 上一篇：`02-启动与运行时.md`
- 索引：`README.md`
- 下一篇：`04-查询执行与工具系统.md`
- 配套阅读：`09-关键时序图.md`、`15-常见问题定位手册.md`
