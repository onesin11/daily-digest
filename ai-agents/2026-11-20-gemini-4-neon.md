# Gemini 4 Neon 发布：编程代理榜第一的中档模型，API 只开付费档，没有免费额度

**一句话结论**：Google 11 月 18 日发布 Gemini 4 Neon，定位在旗舰 Carbon 和快速档 Hydrogen 之间，官方称在 Artificial Analysis 编程代理指数上排第一（58.6），每百万 token 输入 3 美元、输出 15 美元，上下文 100 万 token。开发者只能走 Gemini API **付费档**；Antigravity 和 AI Studio 里试用要 Google AI Pro / Ultra 订阅。想白嫖的话这次没有入口。

## 要点

- **价格**：输入 3 美元 / 百万 token，缓存命中输入 0.15 美元（便宜 95%），输出 15 美元 / 百万 token，上下文窗口 100 万 token。
- **编程相关成绩（官方数字）**：Artificial Analysis 编程代理指数 58.6，排第一；Terminal-Bench 4.1 65.2%；FrontierCode 2.0 46.2%（默认 high 档）。Artificial Analysis 综合智能指数在 xhigh 档得 61 分，总排名第三。
- **为长任务代理调过**：官方说它会少做重复读文件、边做边汇报进度，长会话里上下文压缩更稳。思考强度分四档，可以按任务调花费。
- **成本**：官方称默认 high 档每个任务比 Carbon 便宜 32%，比同级竞品便宜约 50% 到 80%。这是 Google 自己选的第三方榜单口径。
- **在哪能用**：Gemini API 付费档（AI Studio、Android Studio）、Gemini for business、Antigravity（付费 AI Pro / Ultra 用户）、Gemini app（Pro / Ultra 订阅）。安全研究专用的 "Neon for cyber" 只给受信任的防守方。
- **社区反响**：Hacker News 上 1,035 分、768 条评论（截至 11 月 20 日）。

## 适合谁

- 在 Gemini CLI 或其他代理里按 token 付费、想找比 Carbon 便宜但编程能力不掉太多的模型的人。
- 已经订了 Google AI Pro / Ultra、想在 Antigravity 里试新模型的人。

## 注意事项

- **没有免费档**：官方发布文只写了“Gemini API 付费档”。只用免费 key 的 Gemini CLI 或机器人暂时用不上，别急着改模型名。另外结合 11 月 16 日那条：新开的 Google Cloud 试用账号，300 美元赠金也不能拿来付 Gemini 的钱。
- **榜单数字是厂商挑的**：上面的成绩都来自 Google 发布文。11 月 18 日那条评测笔记说过，同一个模型换个代理壳子，花费能差好几倍，换之前最好用自己的真实任务比一次。
- **按 token 算的代理要盯步数**：输出 15 美元 / 百万 token 不算便宜，长任务代理的钱主要花在来回步数上，开 xhigh 档前先想清楚。
- 我没有付费 key，没实际调用过，内容来自官方发布文。

## 来源

- Google 官方博客：https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-neon/ （2026-11-18）
- Hacker News 讨论：https://news.ycombinator.com/item?id=50552424
- 收录日期：2026-11-20
