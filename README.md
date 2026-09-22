# Team AI Harness

本仓库是团队的 **TeamAI 共享资源仓库**（team repo），统一存放团队的 skills、rules、docs、env、MCP 等 AI Harness 配置，由 [`teamai-cli`](https://github.com/Tencent/teamai-cli) 分发到每位成员本地的 AI 工具（Claude Code、Codex、Cursor、Qoder 等）。

工作方式：**push → 创建分支 + MR → reviewer 审批合并 → 成员 pull 自动同步**。

## 快速开始

```bash
npm install -g teamai-cli          # 安装 CLI
teamai --version                   # 确认版本

# 成员：在自己的项目目录下初始化（资源装到 <cwd>/.teamai）
cd /path/to/your-project
teamai init https://github.com/moonpeng/team-ai-harness.git

# 或者用户级初始化（资源装到 ~/，对所有项目生效）
teamai init https://github.com/moonpeng/team-ai-harness.git --scope user
```

`init` 会完成 OAuth 登录、克隆团队仓库、注册成员、注入 hooks。之后**每次开启 AI 会话时 SessionStart hook 会自动 `teamai pull`**，无需手动同步。

初始化后可用 `teamai doctor` 自检配置，`teamai status` 查看本地与团队仓库的差异。

## 日常命令

| 命令 | 用途 |
|------|------|
| `teamai pull` | 拉取团队资源并注入本地 AI 工具（`--force` 强制全量同步） |
| `teamai push` | 把本地资源推到新分支并创建 MR（`--all` 跳过确认） |
| `teamai status` | 本地 vs 团队仓库的差异与资源数量 |
| `teamai list <type>` | 列出资源，type 取 `skills\|rules\|docs\|env\|agents\|hooks\|mcp` |
| `teamai skill show <name>` | 查看某个 skill 的来源、贡献者、已安装的 agent |
| `teamai skill exclude add <name>` | 个人跳过某个 skill 的同步，不影响团队仓库 |
| `teamai remove <type> <name>` | 删除资源并创建 MR |
| `teamai doctor` | 诊断配置问题 |
| `teamai uninstall` | 移除本机所有 teamai 资源和 hooks |

## 仓库结构与资源格式

| 资源 | 路径 | 说明 |
|------|------|------|
| Skills | `skills/<name>/SKILL.md` | 可按角色命名空间分层，如 `skills/hai_dev/<name>/` |
| Rules | `rules/**/*.md` | 团队规则，注入各 Agent；可按主题分子目录，如 `rules/java/` |
| Docs | `docs/` | 基础文档，渐进式披露，默认不全量加载 |
| Env | `env/env.yaml` | 团队环境变量与开关 |
| MCP | `mcp/mcp.yaml` | 团队 MCP server |
| Agents | `agents/<name>.yaml` | 子 agent 定义 |
| Hooks | `hooks/hooks.yaml` | 团队级 hooks |
| Culture | `culture.md` | 团队使命与协作准则，注入 CLAUDE.md / AGENTS.md |
| 配置 | `teamai.yaml` | 仓库地址、provider、reviewers、sharing 策略 |
| 成员 | `members/<username>.yaml` | 由 `teamai init` 自动注册 |

新增 skills 用 `teamai push` 从本地 AI 工具目录收集；rules、docs、mcp、hooks、culture 则**直接编辑仓库文件后提交 MR**。

MCP 只认 `mcp/mcp.yaml`，散落的 `.json` 不会被分发：

```yaml
# 正确：mcp/mcp.yaml
servers:
  - name: codegraph
    description: 代码结构图谱
    transport: stdio          # stdio | http | sse
    command: codegraph
    args: ["serve", "--mcp"]
```

```jsonc
// 错误：mcp/codegraph.json —— teamai 不读取该文件，成员 pull 后拿不到这个 server
{ "mcpServers": { "codegraph": { "command": "codegraph" } } }
```

## 环境变量

```bash
teamai env add API_BASE https://api.internal -d "内网网关"   # 写入 env/env.yaml
teamai push                                                  # 发布给团队
teamai pull && source ~/.zshrc                               # 成员侧生效
teamai env list                                              # 查看（默认打码）
teamai env list --reveal                                     # 明文
teamai env remove API_BASE
```

`teamai pull` 会生成 `~/.teamai/env.sh`（`export KEY='value'`），并在 `teamai.yaml` 的 `sharing.env.injectShellProfile: true` 时，把一行 `source` 幂等注入 `~/.zshrc`（标记块 `# [teamai:env:start]` … `# [teamai:env:end]`，请勿手改）。不想动 shell profile 就设 `false`，或用 `sharing.env.shellProfilePath` 指定其他文件。

> `env/env.yaml` 是**明文入库**的，只放通用变量和团队开关，**不要放密钥**；个人密钥写进本机 `.env`（已被 `.gitignore` 忽略）。

## 知识沉淀

```bash
teamai recall enable            # 开启团队知识自动检索（默认关闭）
teamai recall "port conflict"   # 手动检索团队知识库
teamai contribute               # 把本次 session 的经验分享到团队仓库
teamai session save --push      # 记录脱敏的会话摘要
teamai digest                   # 生成团队周报
teamai dashboard                # 启动 Web 看板（实时会话 + 趋势）
```

Session 结束时若检测到摩擦信号（打断、拒绝工具调用、反复重试），CLI 会提示运行 `/teamai-share-learnings` 沉淀经验。

## 提交规范

完整规则见 [`rules/git-workflow.md`](rules/git-workflow.md)（《AI Coding Git 提交规范 v1.0》），要点：

- 格式 `<type>(<scope>): <subject>`，header ≤72 字符，subject 祈使句现在时、小写开头、句末无句号。
- `type` 只能取 `feat` `fix` `refactor` `perf` `style` `docs` `test` `build` `ci` `chore` `revert`，禁止自创（如 `update` / `wip`）。
- `scope` 用模块语义名（如 `order`），**不是目录路径**；优先复用 `git log` 里已有的 scope，跨多模块则省略。
- body 写 Why 不写 What，每行 ≤100 字符；`fix` 类必填（现象 + 根因）。
- AI 参与时用 `Assisted-by` + `Model` 两个 trailer；**禁止用 `Co-authored-by` 标注 AI**——该字段表示真实人类作者，混用会污染贡献者统计与 `CODEOWNERS` 判定。`Model` 必须是工具实际暴露的模型标识，取不到就写 `unknown`，不得编造。
- 暂存只允许 `git add <明确文件>`，禁止 `git add -A` / `git add .`（易纳入密钥与大文件）。
- `push`、`--amend`、`reset --hard`、`rebase -i`、改 `git config` 等属受限操作，需用户逐次显式授权，一次授权不代表永久授权。

```text
# 正确
build(common): 建立公共组件聚合模块与版本管理

以 core、all 和 bom 分离基础能力、统一依赖入口及版本管理，统一 Java 8 构建配置。

Assisted-by: Qoder
Model: unknown
```

```text
# 错误：无 type/scope，且误用 Co-authored-by 标注 AI
新增 ignore 文件

Co-authored-by: AI Assistant
```

## 更多信息

完整用法见官方文档 [usage-guide.zh-CN.md](https://github.com/Tencent/teamai-cli/blob/main/docs/usage-guide.zh-CN.md)。角色（`teamai roles`）、项目（`teamai projects`）、标签（`teamai tags`）、跨团队订阅（`teamai source`）等分发策略按需启用。
