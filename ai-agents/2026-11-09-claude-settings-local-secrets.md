# Claude Code 的“不再询问”会把命令里的密码存进项目，再被一起推上 GitHub

**一句话结论**：在 Claude Code 里对一条带密码的命令点“Yes, and don't ask again”，这条命令会原样写进项目里的 `.claude/settings.local.json`；这个文件没进仓库的 `.gitignore`，很容易被顺手提交。Apiiro 在公开仓库里找到了 1,122 份含真实密钥的这类文件。

## 要点

- **怎么漏的**：批准命令时选“不再询问”，Claude Code 把整条命令（包括里面的密码、token）存进 `.claude/settings.local.json`。它只把这个文件加进了开发者的全局 git ignore，没加进仓库自己的 `.gitignore`，换台机器、换个 ignore 配置就会被提交。
- **规模**：GitHub 上公开的 104,185 份 `.claude/settings.local.json` 里，1,122 份含真实密钥；算上 git 历史、共享的 `settings.json` 和对话记录，一共 2,539 个真实密钥，分布在 1,994 个公开仓库。类型有数据库密码、云厂商 key、API token、npm token、SSH root 登录等。
- **代理自己也会提交**：含真实密钥的 1,105 个仓库里，有 539 个是由 Claude 署名参与的提交改动了这个文件。
- **扫描器容易漏**：gitleaks、TruffleHog 这类通用扫描器主要按密钥格式匹配，夹在命令行参数里的普通密码经常认不出来。
- **不止 Claude**：共享的 `.claude/settings.json`、代理的对话记录和记忆文件、交接/部署笔记、VS Code Copilot 的自动批准设置里也发现了密钥。`.cursor/`、`.gemini/` 这类目录同样应当当成可能含密钥的地方。
- **官方回应**：Claude Code 负责人 Boris Cherny 已确认问题，表示团队在研究密钥清洗和基础的密钥扫描。

## 怎么自查和处理

1. 在每个用过 Claude Code 的项目里看这个文件有没有被跟踪：`git ls-files | grep -E '\.claude/settings(\.local)?\.json'`。
2. 被跟踪了就取消跟踪并加进仓库的 `.gitignore`：`git rm --cached .claude/settings.local.json`，再在 `.gitignore` 里加一行 `.claude/settings.local.json`。
3. 翻一遍文件里的 `permissions.allow` 规则，删掉任何带密码、token、带账号的连接串的条目，即使从没提交过（文件以明文 JSON 留在本机，恶意软件也能读）。
4. 只要某个凭据曾经出现在这个文件里、又进过公开仓库或历史，就直接换掉（轮换），光删文件不够。
5. 以后别把密钥直接写进命令，改用环境变量引用；批准时尽量批准命令模式（例如 `Bash(npm run:*)`），不要批准写死参数的整条命令。
6. 推送前的扫描把 `.claude/`、`.cursor/`、`.gemini/` 和代理记忆文件也算进去，并加一条针对命令行里明文密码的 grep 规则，别只靠 gitleaks。

## 适合谁

用 Claude Code、Codex、Gemini CLI、OpenCode 这类代理在真实项目里跑命令、尤其是会让代理碰数据库、服务器和云账号的人；同时维护公开仓库的人更要查一遍。

## 注意事项

- 数字来自 Apiiro 自己的研究，没有第三方复核。
- 私有仓库同样有风险：仓库以后转公开、或者被别人 fork/拷走，历史里的密钥会一起带出去。
- 清理 git 历史（filter-repo 之类）只能减少继续扩散，已经公开过的凭据仍然要轮换。

## 来源

- 社区：Hacker News（606 分，409 条评论）
- 原文：https://apiiro.com/blog/coding-agents-leaking-secrets-github-dont-ask-again
- 讨论：https://news.ycombinator.com/item?id=50407882
- 原帖日期：2026-11-07（Hacker News 讨论日期）
- 收录日期：2026-11-09
