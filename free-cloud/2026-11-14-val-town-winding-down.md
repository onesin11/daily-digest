# Val Town 关停：免费跑脚本、定时任务的平台，2027 年 1 月 10 日全部停服

**一句话结论**：Val Town（有 $0 免费档、能在线写 JS/TS 小程序、跑定时任务、收发邮件、带 SQLite 和 Blob 存储的托管平台）2026 年 11 月 10 日宣布关停；12 月 10 日代码变只读，2027 年 1 月 10 日所有服务停止。还在用的人要在这之前把 val、数据和 `*.val.run` 链接迁走。

## 要点

- **时间表**（官方公告）：11 月 10 日起停止新注册和升级套餐；12 月 10 日进入只读（SQLite 和 Blob 存储不受只读影响）；2027 年 1 月 10 日其余服务全部停止；2027 年 2 月 1 日开源整个 Val Town。
- **钱的事**：当天起暂停扣费，现有订阅免费服务到停服；预付了年费的会部分退款。
- **关停原因**：创始人 Steve Krouse 说得很直白，在云计算这一行竞争不过别人。HN 评论补充了一层：Val Town 整个建在 Deno 运行时上，而 Val Town 文档写明 Deno 核心维护者 2027 年 10 月以后不再支持 Deno。
- **官方迁移指南**：给了 Cloudflare、Vercel、Netlify、Render、GitHub 五份迁移教程，还有一段现成的提示词，直接贴给 AI 代理让它帮你装 Val Town 的 MCP、挑出要搬的 val、选平台、出迁移方案。
- **搬去哪**：官方认为 Cloudflare 功能最对得上（网站、SQLite 用 D1、Blob 用 R2、定时任务、收发邮件、访问控制、AI 都有），Val Town 团队自己的博客和文档也搬去了 Cloudflare。定时任务也可以换 GitHub Actions、Vercel/Render 的 Cron；运行时建议换成 Node、Bun 或 Cloudflare Workers 这类平台自带的，别再迁到 Deno 上。
- **旧链接**：迁完、验证好、数据同步完以后，把原来的 `*.val.run` 地址重定向到新地址，外面的链接就不会断；用到 webhook 的要去对应服务商那里改回调地址。

## 适合谁

- 在 Val Town 上挂着小脚本、定时任务、webhook、邮件处理的人：现在就该开始搬。
- 在挑“免费跑点小程序”平台的人：可以当成一个提醒，小公司的免费托管平台随时可能关门。

## 注意事项

- **别拖到 12 月 10 日以后**：只读之后就改不了代码了，迁移时要边改边测会很麻烦。
- **SQLite 和 Blob 里的数据要单独导出**：迁的不只是代码。Val Town 的 SQLite 底层是 Turso，可以考虑直接迁到 Turso 或 Cloudflare D1。
- **免费平台的通用教训**：代码、数据、域名尽量放在自己能导出的地方；用自己的域名指向平台，而不是只用平台给的子域名，换平台时链接才不用全改。
- 2 月开源以后理论上可以自己部署，但到时能不能顺利跑起来还不知道，别把它当成迁移方案。

## 来源

- 社区：Hacker News（423 分）
- 官方公告：https://blog.val.town/winding-down
- 迁移指南：https://docs.val.town/migration
- 搬去哪：https://docs.val.town/migration/where-to-migrate
- 讨论：https://news.ycombinator.com/item?id=50443427
- 原帖日期：2026-11-10
- 收录日期：2026-11-14
