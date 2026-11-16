# Chrome DevTools MCP 1.13：Google 官方的“给代理用的 Chrome”，Claude Code、Codex 一条命令接上，Linux 服务器上实测能跑

**一句话结论**：`chrome-devtools-mcp` 是 Chrome DevTools 团队做的 MCP 服务器加命令行工具，让 Claude Code、Codex、Gemini CLI 这些代理直接开一个真 Chrome 去打开网页、点按钮、截图、看网络请求和控制台报错、跑性能分析。11 月 11 日发了 1.13.0，11 月 16 日上了 GitHub 每日趋势榜，约 5.7 万颗星。要让代理“动手操作网页”或者调前端，它是现在最省事的官方方案；只是查资料的话，用搜索 API 更划算。

## 要点

- **怎么接**：
  - Claude Code：`claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest`；想连带官方技能（skills）一起装，就在 Claude Code 里用 `/plugin marketplace add ChromeDevTools/chrome-devtools-mcp`，再 `/plugin install chrome-devtools-mcp@chrome-devtools-plugins`。
  - Codex：`codex mcp add chrome-devtools -- npx chrome-devtools-mcp@latest`。
  - Gemini CLI：`gemini mcp add -s user chrome-devtools npx chrome-devtools-mcp@latest`，或者装成扩展（带技能）。
  - 其他支持 MCP 的客户端，配置里写 `npx -y chrome-devtools-mcp@latest` 就行。
- **不用 MCP 也能用**：同一个包里带了 `chrome-devtools` 命令行，`start` 启动后台进程，然后 `navigate_page`、`take_snapshot`、`click`、`take_screenshot`、`list_network_requests` 一条条敲，适合写成 shell 脚本，也可以让代理直接调命令行。
- **它给代理看的是“页面快照”**：`take_snapshot` 输出的是带编号的可访问性树（每个链接、按钮一行，带 uid），代理按 uid 点击，比靠截图找坐标稳；要按坐标点得另开 `--experimentalVision`。
- **1.13.0 新增**（2026-11-11）：日志断点（logpoints，不改代码就能在某行打印变量）；按页面单独模拟网络请求（network mocks），还能拿真实响应再改；配置文件带 UTF-8 BOM 也能读了。
- **服务器上常用的参数**：`--headless`（无界面）、`--isolated`（用临时的浏览器资料目录，用完自动删）、`--userDataDir`（固定资料目录，保留登录状态）、`--browserUrl` / `--wsEndpoint`（连一个已经开着的 Chrome）、`--proxyServer`（走代理）、`--blockedUrlPattern`（禁止访问某些网址）、`--slim`（只留基础工具，省上下文）。
- **实测（2026-11-16，Ubuntu 26.04 x86_64 服务器，无图形界面）**：用 `npx @puppeteer/browsers install chrome@stable` 装了 Chrome for Testing 157，补装 libnss3、libnspr4 等系统库后，`chrome-devtools start --headless --isolated --executablePath <chrome 路径> --chromeArg=--no-sandbox` 能起来，版本 1.13.0。打开 example.com、拿到页面快照、按 uid 点“Learn more”跳到 iana.org、列出网络请求、截图都成功，`stop` 之后没有残留的 Chrome 进程。

## 适合谁

- 想让 Claude Code / Codex 自己打开网页验证改动、看控制台报错、抓接口的人。
- 在服务器上跑代理、需要偶尔操作网页（点按钮、填表、截图）的人。
- 只是查资料、读网页正文的：用 Exa、Parallel 这类搜索 API 加 Jina Reader 更快更省，不必开浏览器。

## 注意事项

- **默认会上报使用统计**：Google 默认收集工具调用成功率、耗时、环境信息。不想上报就加 `--no-usage-statistics`，或者设环境变量 `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS=1`。性能分析默认还会把网址发给 Google 的 CrUX 接口拿真实用户数据，用 `--no-performance-crux` 关掉。
- **代理能看到浏览器里的一切**：官方明说 MCP 客户端可以查看、修改浏览器里的任何数据。别用你日常登录了各种账号的浏览器资料目录，给它单独一个 `--userDataDir`，或者用 `--isolated`。
- **服务器上要自己装 Chrome 和系统库**：只装 npm 包不够，没有 Chrome 时会启动失败；官方只保证 Google Chrome 和 Chrome for Testing，其他 Chromium 不保证。MCP 服务器默认不是无头模式，没有图形界面的机器要加 `--headless`（命令行的 `start` 默认就是无头）。实测时加了 `--no-sandbox`，没试不加的情况。
- **吃内存**：实测时各个 Chrome 进程的常驻内存加起来约 1.2 GB（这个算法会重复计入共享内存，偏高）。1 GB 内存的小机器上别指望长期开着。
- **不是反爬工具**：它就是一个普通的自动化 Chrome，遇到人机验证、严格风控照样会被拦。实测时 Hacker News 就因为请求太频繁直接返回了 503。
- `@latest` 每次启动都会拉最新版，想稳定就钉死版本号；用了官方技能的，升级包之后记得同步升级技能，否则技能里写的工具参数可能对不上。

## 来源

- 社区：GitHub Trending（2026-11-16 每日趋势榜，约 5.7 万星）
- 项目主页：https://github.com/ChromeDevTools/chrome-devtools-mcp
- 1.13.0 发布说明：https://github.com/ChromeDevTools/chrome-devtools-mcp/releases/tag/chrome-devtools-mcp-v1.13.0
- 各客户端配置：https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/client-configurations.md
- 全部参数：https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/configuration.md
- 发布日期：2026-11-11
- 收录日期：2026-11-16
