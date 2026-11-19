# DeepSeek API 调整计费：模型自己联网搜索、读网页，工具调用不再单独收钱，只按 token 算

**一句话结论**：从北京时间 2026 年 11 月 18 日 12:00 起，调用 DeepSeek 模型时由服务端触发的联网搜索和网页抓取，这次工具调用本身不再收费；搜回来的内容会拼进上下文，按模型的 token 用量计费。单独调用的 Search / Fetch 接口没变，还是每 1,000 次成功请求 5 元。想给代理或机器人加“会上网查资料”的能力又不想另配搜索服务，现在用 `deepseek-flash` 加服务端联网工具最省事。

## 要点

- **哪些算“服务端搜索”**：官方价格页列了三种入口：
  - 联网工具（`deepseek_web_search` 联网搜索、`deepseek_read_url` 阅读链接）。
  - Anthropic 格式接口里的 `web_search` 工具。
  - Responses API 里的 `web_search` 工具。
- **怎么开**：在请求的 `tools` 里声明 `deepseek_web_search` 和/或 `deepseek_read_url`，模型自己判断要不要搜，搜索和抓取都在服务端完成，客户端不用写任何工具代码。Messages（Anthropic 格式，地址 `https://api.deepseek.com/anthropic`）、Responses、Chat Completions 三种协议都支持，官方要求用 `deepseek-flash` 模型。
  - 关掉思考：普通联网搜索，回答带引用。
  - 打开思考：“深度思考搜索”，模型会在多步推理里自己搜、打开网页、页内查找。
  - 只要读链接、不要搜索：只声明 `deepseek_read_url`。
- **现在的 token 价格**（`deepseek-flash`，每百万 token）：输入缓存未命中空闲时段 1 元、高峰 2 元；输出空闲 4 元、高峰 8 元；缓存命中空闲 0.02 元。高峰时段是北京时间工作日 9:00–12:00、14:00–18:00，其余时间（含周末和法定节假日全天）半价。
- **单独的 Search / Fetch 接口**：给自己写程序、做 Agent 用的原始 JSON 接口，`POST https://api.deepseek.com/v1/tools/search` 和 `/v1/tools/fetch`，用 DeepSeek 的 API key 鉴权。Search 一次最多返回 50 条，可以要正文（`need_content`），支持 `site:`、`lang:`、`after:`、`before:` 等语法；Fetch 把单个网页转成 Markdown。这两个照旧按次收费，每 1,000 次成功请求 5 元，不在这次免费范围内。

## 适合谁

- 在 Telegram / QQ 机器人里接了 DeepSeek，想让它能回答“今天发生了什么”这类问题的人：声明一个工具就行，不用另外接搜索 API。
- 用 Anthropic 兼容地址把 Claude Code 一类代理接到 DeepSeek 的人：按官方说法，Anthropic 格式的 `web_search` 也算服务端搜索，只按 token 计费。具体某个代理发出的请求是不是走这条路，要自己看一下账单确认。
- 已经用 Parallel 搜索加 Jina Reader 读正文的：那套是自己控制搜什么、读哪篇；DeepSeek 这个是交给模型自己决定，适合聊天问答，不适合需要固定流程、可复现的报告任务。

## 注意事项

- **不是“完全免费”**：工具调用不收钱了，但搜到的网页内容会塞进上下文，按输入 token 算钱。深度思考搜索会多步搜、多次读网页，上下文可能涨得很快，实际花费要看账单。
- **只说了这些入口**：价格页只列了上面三种服务端搜索方式。单独调用 Search / Fetch 接口仍按次收费，别搞混。
- **网页抓取不一定成功**：官方文档写明 Fetch 的结果受目标站点的可用性和访问策略影响，被站点拦截时会返回 404。
- **对外提供服务要开安全审核**：官方建议面向公众的服务打开模型安全参数，并自己做内容审核、大模型备案和 AI 标识。
- 我没有 DeepSeek API key，这次没实际调用过，内容都来自官方文档。

## 来源

- 官方价格页（含 11 月 18 日计费调整说明）：https://api-docs.deepseek.com/zh-cn/quick_start/pricing
- 联网工具文档：https://api-docs.deepseek.com/zh-cn/guides/web_tools
- Search 与 Fetch 接口文档：https://api-docs.deepseek.com/zh-cn/guides/search
- 更新日志（11 月 13 日发布应用 API 全家桶）：https://api-docs.deepseek.com/zh-cn/updates
- 媒体报道：凤凰网科技 https://tech.ifeng.com/c/8y9yGjlRPjR （2026-11-18）
- 生效时间：2026-11-18 12:00（北京时间）
- 收录日期：2026-11-19
