# Team AI Harness

本仓库用于维护团队统一的 AI Harness 规范与配置。Agent 在本仓库或引用本仓库开展工作时，必须先阅读并遵守本文档及其引用的规则。

## Repository Structure

- `rules/`：按主题拆分的团队规则。
- `mcp/`：团队共享的 MCP Server 配置。

新增规范时，应放入对应目录，并同步更新本文档中的规则索引。不要在多个文件中重复定义同一规则。

## Required Rules

### CodeGraph

遵守 [`rules/codegraph.md`](rules/codegraph.md)。分析仓库结构、依赖关系、调用链或变更影响时，优先使用 CodeGraph：

```text
codegraph explore "<question>"
```

CodeGraph 已配置为 MCP Server，配置来源为 [`mcp/codegraph.json`](mcp/codegraph.json)。

### Git Commits

遵守 [`rules/git.md`](rules/git.md)。所有提交信息必须使用以下 Conventional Commits 格式：

```text
<type>(<scope>): <subject>
```

允许的 `type`：`feat`、`fix`、`refactor`、`perf`、`test`、`docs`、`build`、`ci`、`chore`。

AI 参与提交时，提交信息必须包含：

```text
Co-authored-by: AI Assistant
```

## Maintenance

- 新增或修改规则时，保持规则内容明确、可执行且无冲突。
- 主题规则统一维护在 `rules/` 中；本文档只提供入口、索引和关键约束摘要。
- MCP 配置统一维护在 `mcp/` 中，并使用有效的 JSON。
- 若本文档摘要与被引用的规则文件不一致，以 `rules/` 下的规则文件为准，并及时修正本文档。
