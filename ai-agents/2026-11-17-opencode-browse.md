# OpenCode Browse：让 Claude Code、Codex 直接用你日常的 Chrome，带着你的登录状态干活，遇到登录、两步验证就把标签页交还给你

**一句话结论**：OpenCode 团队（Anomaly）出的 `browse` 是一个本地 MCP 服务器加一个 Chromium 扩展，代理借用的是你**本人正在用的浏览器**：已登录的网站、装好的扩展都能直接用，不用另配账号或 key。11 月 16 日发了 v0.1.0，11 月 17 日上了 Hacker News 首页（约 266 分、150 多条评论）。它解决的是“代理要操作需要登录的网站”这个痛点；代价是代理能碰到你真实的登录状态，安全边界要自己想清楚。

## 要点

- **它是什么**：两部分。本机跑一个 `browse` 程序（MCP 服务器 + 中继），浏览器里装一个扩展。代理通过 MCP 发指令，中继转给扩展，扩展用浏览器自带的调试接口（`chrome.debugger`）在标签页里执行。支持 Chrome、Edge、Brave、Arc、Vivaldi、Opera、Chromium 等。MIT 开源，仓库约 790 星。
- **怎么装**：先在 Chrome 应用商店装扩展，它会带你走完剩下的步骤；也可以直接 `curl -fsSL https://opencode.ai/browse/install | bash`，或 `npx @opencode/browse install`。`browse install` 会自动把 MCP 写进本机找到的代理配置，支持 OpenCode、Claude Code、Codex、Gemini CLI、Qoder CLI、Hermes Agent 等三十多个客户端；指定代理用 `browse install -a claude codex`。装完要重启代理才能加载。
- **代理能做什么**：在后台开标签页（自动收进折叠的标签组，不打扰你）、读页面结构、点击、输入、选择、上传文件、截图和截图对比、抓网络请求和控制台报错、录屏、模拟设备。OpenCode 拿到的是一个个独立工具；其他客户端拿到 `execute`、`status`、`tabs` 三个，`execute` 里写类似 Playwright 的 JavaScript 去调用全部能力。所有功能也能当命令行用（`browse navigate`、`browse snapshot`、`browse click`）。
- **交接（handoff）**：碰到登录、两步验证、付款，代理调用 `handoff`，页面上弹出一张小卡片，你自己操作完点 **Continue**，代理接着干。也可以点工具栏按钮把你已经开着的某个标签页“借”给代理，再点一次收回。
- **代理跑在服务器上也能用**：代理在 VPS、浏览器在你自己电脑上时，在服务器上 `browse install --remote`，扩展里选“连接另一台电脑上的代理”填地址和配对码。官方明说这条连接**不加密**，要走 Tailscale 或 WireGuard，别直接暴露在公网。
- **限制代理乱跑**：开标签页时可以设 `allowedOrigins`（只许访问指定网站）和 `readOnly`（禁止点击、输入、跳转）。默认还拒绝清 Cookie、关浏览器、改网站权限、打开浏览器内部页面和本地文件。

## 适合谁

- 想让 Claude Code、Codex、OpenCode 帮你在**需要登录的网站**上干活（后台、工单、邮箱网页版）的人，不想再给代理单独配一套登录。
- 平时在自己电脑上开着浏览器、代理跑在服务器上，又能接受走 Tailscale 组网的人。
- 只是查资料、读网页正文的：用搜索 API 加 Jina Reader 更快更省；只需要一个干净的自动化浏览器的，用 Chrome DevTools MCP（见 [2026-11-16 那条](2026-11-16-chrome-devtools-mcp.md)）更合适，它不碰你的日常登录。

## 注意事项

- **代理拿到的是你真实的登录状态**：官方自己的安全说明写得很直白，`allowedOrigins` 和 `readOnly` 是“护栏，不是数据边界”：页面自己发出的请求不受限制，`eval` 仍能读到页面能读的一切。网页内容、邮件内容都可能夹带指令去诱导代理。不想让代理看到的页面，就别在这个浏览器里让它打开。最稳妥是给它单独建一个浏览器用户资料，只登录需要的网站。
- **脱敏不完全**：`fill` 只对密码框和验证码框隐藏输入值；抓包会按名字和格式遮掉认证头、Cookie 等，但放在普通字段名里的密钥照样会露出来；截图和页面文字原样返回给模型。
- **默认自动更新**：中继启动时和之后每小时检查一次新版并自动安装。想自己控制版本，就在扩展设置里改成只提醒或关闭，或设环境变量 `OPENCODE_BROWSER_AUTO_UPDATE=notify`（或 `off`）。
- **不是过人机验证的工具**：它模拟人的鼠标轨迹和打字节奏，让代理不容易被发现，但官方说“难发现不等于发现不了”，浏览器顶部仍会显示“正在调试”的提示条。遇到 Cloudflare 或验证码，它只会报告并交给你，不会去破解。
- **社区评价有分歧**：Hacker News 上有人批评 OpenCode 团队发新项目太快、安全上太随意；也有人认为安全本来就该靠外层沙箱，而不是在代理里不停弹确认。项目刚到 v0.1.0，先在不重要的浏览器资料里试用比较稳。
- 这条是照官方 README、产品页和 Hacker News 讨论整理的，没有实际装过：它需要一个装了扩展的真实浏览器配合。

## 来源

- 社区：Hacker News（2026-11-17 首页，约 266 分）：https://news.ycombinator.com/item?id=50523459
- 产品页：https://opencode.ai/product/browse
- 源码与 README（含安全说明）：https://github.com/anomalyco/browse
- 版本：v0.1.0，发布于 2026-11-16
