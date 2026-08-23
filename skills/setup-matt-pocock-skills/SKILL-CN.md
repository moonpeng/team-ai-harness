---
name: setup-matt-pocock-skills
description: 为工程技能配置本仓库——设置议题跟踪器、分诊标签词汇和领域文档布局。在首次使用其他工程技能之前运行一次。
disable-model-invocation: true
---

# 配置 Matt Pocock 的技能

搭建工程技能所依赖的仓库级配置：

- **议题跟踪器**——议题存放位置（默认 GitHub；也原生支持本地 Markdown）
- **分诊标签**——五个标准分诊角色所使用的字符串
- **领域文档**——`CONTEXT.md` 与 ADR 的存放位置，以及读取它们的使用规则

这是由提示驱动的技能，而不是确定性脚本。先探索并展示发现，与用户确认后再写入。

## 流程

### 1. 探索

检查当前仓库以了解起始状态。读取所有已有内容，不要假设：

- `git remote -v` 和 `.git/config`——这是 GitHub 仓库吗？具体是哪一个？
- 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`——是否存在？其中是否已有 `## Agent skills` 部分？
- 根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
- `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
- `docs/agents/`——本技能之前的输出是否已存在？
- `.scratch/`——是否表明已经采用本地 Markdown 议题跟踪器约定？
- 是否安装了 `triage` 技能？（与本技能同级的 `triage` 文件夹，或可用技能中的 `triage`。）这决定是否运行 B 节。
- Monorepo 信号——`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或包含自身 `src/` 的非空 `packages/*`。只有真正的大型多包仓库才具备这些信号；没有信号就表示单上下文，这适用于几乎所有仓库。

### 2. 展示发现并询问

总结已有和缺失的内容。然后按顺序逐节处理——每节只问一个问题，得到回答后再进入下一节。

每节先给出推荐答案，让用户可以用一个词接受。只有在选择确实会产生分支时才给出一行解释；探索已经确定答案时完全跳过该节（未安装 `triage` 时跳过 B 节；没有 monorepo 时跳过 C 节）。

**A 节——议题跟踪器。**

> 说明：“议题跟踪器”是本仓库中议题的存放位置。`to-tickets`、`triage` 和 `to-spec` 等技能会读写它——它们需要知道应调用 `gh issue create`、在 `.scratch/` 下写 Markdown 文件，还是遵循你描述的其他工作流。请选择本仓库实际跟踪工作的地方。

默认立场：这些技能为 GitHub 设计。如果 `git remote` 指向 GitHub，建议使用 GitHub；如果指向 GitLab（`gitlab.com` 或自托管主机），建议使用 GitLab。否则（或用户偏好其他方案），提供：

- **GitHub**——议题位于仓库的 GitHub Issues（使用 `gh` CLI）
- **GitLab**——议题位于仓库的 GitLab Issues（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 Markdown**——议题以文件形式位于仓库的 `.scratch/<feature>/` 下（适合个人项目或没有远程仓库的项目）
- **其他**（Jira、Linear 等）——请用户用一段话描述工作流；技能会将其记录为自由文本

将选择记录到 `docs/agents/issue-tracker.md`。GitHub 和 GitLab 模板包含“将 PR 作为请求入口”开关，默认**关闭**——保持关闭且不要主动询问；希望将外部 PR 纳入分诊队列的用户以后可自行修改文件中的开关。

**B 节——分诊标签词汇。** 如果未安装 `triage` 技能（探索阶段已经得知），则完全跳过本节——未安装的技能不需要标签。

如果已安装，只问一个问题：

> 是否保留默认分诊标签？（推荐：**是**）

默认值是五个标准角色，每个标签字符串都与角色名相同：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。回答“是”时原样写入。只有用户回答“否”时——通常因为跟踪器已有其他名称（例如用 `bug:triage` 表示 `needs-triage`）——才收集覆盖值，使 `triage` 使用现有标签而不创建重复项。

**C 节——领域文档。** 默认使用**单上下文**——仓库根目录一个 `CONTEXT.md` + `docs/adr/`。这适合几乎所有仓库，无需询问即可写入。

只有探索发现 monorepo 信号时才提供**多上下文**方案——根目录 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md`。然后确认用户需要哪种布局。

### 3. 确认并编辑

向用户展示以下内容的草稿：

- 要添加到 `CLAUDE.md` / `AGENTS.md` 中任一文件的 `## Agent skills` 区块（选择规则见第 4 步）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一项仅在安装了 `triage` 时）

写入前允许用户编辑。

### 4. 写入

**选择要编辑的文件：**

- 如果存在 `CLAUDE.md`，编辑它。
- 否则如果存在 `AGENTS.md`，编辑它。
- 如果两者都不存在，询问用户要创建哪一个——不要替用户选择。

当 `CLAUDE.md` 已存在时绝不要创建 `AGENTS.md`（反之亦然）——始终编辑已有文件。

如果所选文件已有 `## Agent skills` 区块，则原位更新其内容，不要追加重复区块。不要覆盖周边章节中的用户修改。

区块内容：

```markdown
## Agent skills

### Issue tracker

[用一行总结议题跟踪位置]。参见 `docs/agents/issue-tracker.md`。

### Triage labels

[用一行总结标签词汇]。参见 `docs/agents/triage-labels.md`。

### Domain docs

[用一行总结布局——“单上下文”或“多上下文”]。参见 `docs/agents/domain.md`。
```

只有安装了 `triage` 且运行了 B 节时，才包含 `### Triage labels` 子区块并写入 `docs/agents/triage-labels.md`。否则两者都省略。

然后以本技能目录中的种子模板为起点写入文档文件：

- [issue-tracker-github.md](./issue-tracker-github.md)——GitHub 议题跟踪器
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md)——GitLab 议题跟踪器
- [issue-tracker-local.md](./issue-tracker-local.md)——本地 Markdown 议题跟踪器
- [triage-labels.md](./triage-labels.md)——标签映射（仅在安装 `triage` 时）
- [domain.md](./domain.md)——领域文档读取规则 + 布局

对于“其他”议题跟踪器，根据用户描述从头编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告知用户配置已完成，以及哪些工程技能现在会读取这些文件。说明以后可以直接编辑 `docs/agents/*.md`——只有想切换议题跟踪器或从头重新配置时，才需要再次运行本技能。
