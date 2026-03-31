# 05. 命令、Skills 与插件

## 1. 命令系统

`src/commands.ts` 负责聚合**本地命令源**。

从当前代码看，`commands.ts` 内部会聚合这些来源：

- 内建 commands
- 本地 skills
- legacy `commands/` 风格 skill
- bundled skills
- built-in plugin skills
- marketplace / session plugin commands 与 skills
- workflow commands
- 动态发现的 dynamic skills

需要额外区分：

- MCP prompts 映射出的 prompt commands
- MCP skill 资源映射出的 skill commands

这两类 **不在 `commands.ts` 内部加载**。
它们会先进入 `AppState.mcp.commands`，再由 `main.tsx` 与
`REPL.tsx` 在运行时合并进最终命令视图。

## 2. Command 契约

统一命令类型定义在：

- `src/types/command.ts`

当前主要命令类型有：

- `prompt`
- `local`
- `local-jsx`

另外还有几组高频能力字段值得记住：

- `availability`
- `disableNonInteractive`
- `supportsNonInteractive`
- `isSensitive`
- `kind`

高频字段包括：

- `type`
- `name`
- `description`
- `source`
- `immediate`
- `allowedTools`
- `context`
- `agent`
- `effort`
- `hooks`
- `skillRoot`
- `paths`
- `loadedFrom`
- `userInvocable`

## 3. Prompt 提交阶段如何处理命令

命令处理主要发生在：

- `src/screens/REPL.tsx`
- `src/utils/handlePromptSubmit.ts`

当前主逻辑可以概括为：

- 识别 `/command`
- 判断是否 `immediate`
- `local-jsx` 命令可直接渲染本地 UI
- 其余命令走 prompt / queue / query 的统一链路

## 4. Skills 体系

核心文件：

- `src/skills/loadSkillsDir.ts`
- `src/tools/SkillTool/SkillTool.ts`

从当前实现看，skills 的来源包括：

- 本地 `skills/`
- legacy `commands/`
- bundled skills
- plugin skills
- MCP skill 资源

Skill 运行方式分两类：

- `inline`
  在当前会话直接展开

- `fork`
  作为子代理执行

Skill 还能携带这些附加信息：

- `hooks`
- `paths`
- `allowedTools`
- `model`
- `agent`

## 5. 插件体系

当前插件系统的核心文件包括：

- `src/plugins/builtinPlugins.ts`
- `src/utils/plugins/pluginLoader.ts`
- `src/utils/plugins/loadPluginCommands.ts`

### 5.1 插件来源

当前代码里可确认的插件来源至少有三类：

- built-in plugins
- marketplace / 已安装 plugins
- `--plugin-dir` 注入的 session-only plugins

### 5.2 插件不只提供 commands / skills

从 `LoadedPlugin` 与 loader 实现看，插件当前还能提供：

- commands
- agents
- skills
- hooks
- output styles
- MCP servers
- LSP servers
- plugin settings

这点很重要：plugin 已经不是“命令扩展器”，而是更广义的运行时扩展面。

### 5.3 built-in plugin 的边界

`src/plugins/builtinPlugins.ts` 里注册的 built-in plugin 和 bundled skills 不同。

它们的特征是：

- 出现在 `/plugin` UI 中
- 可被启用 / 禁用
- 可提供 `skills`、`hooks`、`mcpServers`

## 6. 三者关系

命令、skills、plugins 不是三套互相隔离的系统，而是分两层装配：

```text
commands.ts 内部
内建 commands
  + 本地 skills
  + legacy commands-as-skills
  + bundled skills
  + workflow commands
  + plugin commands
  + plugin skills
  =
本地命令集

运行时额外合并
本地命令集
  + built-in/plugin commands
  + AppState.mcp.commands
  =
最终命令视图
```

要额外区分两点：

- MCP prompt command 会进入命令视图
- `SkillTool` 只会把 skill 型命令纳入 skill 执行面，不会把普通 MCP prompt 当成 skill 跑

## 7. 这一层的意义

这层的核心价值是统一交互入口与扩展面：

- 新交互可以走 command
- 新工作流可以走 skill
- 新运行时能力可以走 plugin
- 外部系统可以经由 MCP 进入命令视图

所以这一层本质上是**用户交互面的统一扩展总线**。

## 8. 相关源码入口

- `src/commands.ts`
- `src/types/command.ts`
- `src/utils/handlePromptSubmit.ts`
- `src/skills/loadSkillsDir.ts`
- `src/tools/SkillTool/SkillTool.ts`
- `src/plugins/builtinPlugins.ts`
- `src/utils/plugins/loadPluginCommands.ts`
- `src/utils/plugins/pluginLoader.ts`

## 9. 文档导航

- 上一篇：`04-查询执行与工具系统.md`
- 索引：`README.md`
- 下一篇：`06-MCP与远程桥接.md`
- 配套阅读：`12-阅读路线-按角色.md`、`15-常见问题定位手册.md`
