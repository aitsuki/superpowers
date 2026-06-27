# Powerset 中文个人版

这是基于 [obra/superpowers](https://github.com/obra/superpowers) 的个人 fork，现已改名为 `powerset`。插件标识改为 `powerset`，避免和 Codex/Claude 内置或官方 marketplace 中的 `superpowers` 插件混淆。

原版 Superpowers 是一套偏强约束的软件开发工作流：自动触发 brainstorming、TDD、worktree、子代理驱动开发和多轮审查。这个 fork 的目标相反：保留有用的 skill，但把它们改成明确、克制、按需启用的工具箱。

## 这个 fork 改了什么

### 1. 禁止隐式调用 skill

原版会在会话开始时注入 `using-superpowers`，并要求 agent 在几乎任何任务前检查和调用相关 skill。

这个 fork 改成：

- 不因为“可能适用”就调用 skill
- 不在普通提问、读代码、小改动前自动进入 workflow
- 只有用户明确说“使用 brainstorming”“使用 Powerset workflow”“用某个 skill”时才启用

普通请求仍按普通工程任务处理。

### 2. 不默认使用子代理驱动开发

原版在执行计划时推荐 `subagent-driven-development`，每个任务都派发实现子代理、审查子代理，最后再做整体验收。

这个 fork 改成：

- 默认推荐 `Inline Execution`
- 子代理驱动只在用户明确选择时使用
- `subagent-driven-development` 不再被描述为默认或推荐路径

这样更适合 Codex 上对速度、token 成本和可控性更敏感的工作流。

### 3. 不默认创建 git worktree

原版会在功能开发或执行计划前倾向创建隔离 worktree。

这个 fork 改成：

- 默认在当前 checkout 工作
- 只有用户明确要求 worktree 时才创建或检查 worktree
- 避免 Codex 在 `.worktrees` 目录里执行命令时路径认知混乱

### 4. TDD 不再强制默认启用

原版 `test-driven-development` 是强纪律 skill，会要求先写失败测试再写代码。

这个 fork 改成：

- 只有用户明确要求 TDD 时才启用
- 或者某份显式选择的 Powerset 计划明确要求 TDD 时才启用
- 普通实现任务遵循项目现有测试习惯和用户偏好

## 推荐使用方式

直接告诉 agent 你想用哪个能力。

示例：

```text
使用 brainstorming 帮我梳理这个需求。
```

```text
使用 powerset:writing-plans，把这份规格文档拆成实现计划。
```

```text
按 Inline Execution 执行这个 plan，不要使用子代理，不要创建 worktree。
```

```text
这次使用 subagent-driven-development 执行计划。
```

```text
使用 systematic-debugging 分析这个 bug。
```

如果你只是说：

```text
帮我修一下这个 bug。
```

agent 应该直接按普通调试任务处理，而不是自动进入 Powerset workflow。

## 主要 skill

### 需求和计划

- `brainstorming`：通过对话澄清需求、方案和边界
- `writing-plans`：把规格文档拆成可执行计划
- `executing-plans`：在当前会话内按计划执行任务

### 实现和审查

- `subagent-driven-development`：显式选择后，用子代理逐任务实现和审查
- `requesting-code-review`：请求 Powerset 风格代码审查
- `receiving-code-review`：处理代码审查反馈
- `verification-before-completion`：完成前做验证
- `finishing-a-development-branch`：完成分支后的合并/PR/保留/丢弃选择

### 调试和测试

- `systematic-debugging`：系统化定位 bug 根因
- `test-driven-development`：显式启用时执行严格 TDD

### 工作区和元能力

- `using-git-worktrees`：显式要求时创建或检查 git worktree
- `dispatching-parallel-agents`：显式要求时并行派发子代理
- `writing-skills`：编写或修改 Powerset skill
- `using-powerset`：会话启动时注入的 skill 使用规则

## Codex 插件说明

Codex 插件入口在：

- `.codex-plugin/plugin.json`
- `hooks/hooks-codex.json`
- `hooks/session-start-codex`
- `skills/`

`hooks/session-start-codex` 会把 `using-powerset` 注入上下文，注入内容是中文个人版的核心规则：Powerset 是显式 opt-in 工具箱，不自动调用其他 skill。

Codex 插件 manifest 中的名称是：

```text
powerset
```

如果通过个人 marketplace 安装，优先使用这个名字，不要再用官方的 `superpowers` 标识。

## Claude marketplace

配套 marketplace 仓库：

```text
https://github.com/aitsuki/powerset-marketplace
```

安装 marketplace：

```text
/plugin marketplace add aitsuki/powerset-marketplace
```

安装插件：

```text
/plugin install powerset@powerset-marketplace
```

## 本地验证

检查 diff 中是否有空白问题：

```bash
git diff --check
```

如果你改了 shell hook，请注意保持 LF 换行。`.gitattributes` 已经固定：

- `.gitattributes`
- `*.sh`
- `hooks/session-start`
- `hooks/session-start-codex`
- `*.cmd`

## 和上游的关系

这个 fork 是个人工作流调整，不适合作为上游 PR：

- 它有意关闭原版的自动触发行为
- 它有意降低 TDD、worktree、子代理审查的默认强度
- 它面向个人使用偏好，而不是 Superpowers core 的通用设计

如果需要向上游贡献，请阅读 [CLAUDE.md](CLAUDE.md) 和 `.github/PULL_REQUEST_TEMPLATE.md`，并遵循上游贡献规则。

## 许可证

MIT License，见 [LICENSE](LICENSE)。
