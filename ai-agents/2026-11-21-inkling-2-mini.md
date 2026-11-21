# Inkling 2 Mini：Apache 2.0 开源、约 20GB 能在笔记本上跑的编程代理模型，同尺寸开源模型里编程榜第一

**一句话结论**：Thinking Machines Lab 11 月 20 日发布的 Inkling 2 Mini 是一个 MoE 模型，总参数 26.4B、每个 token 激活 3.4B，权重约 20GB，Apache 2.0 许可开放下载。它是“循环 Transformer”：中间 12 层每个 token 跑三遍，用更少的输出 token 推理得更深。官方给的成绩里，它在 100B 以下的开源模型中编程代理类测试领先（DeepSWE v1.1 48.2%），工具调用和联网搜索也强；多轮聊天和多约束指令不如 Qwen3.8-27B。想在本地或自己的 GPU 上跑一个编程代理模型的人值得一试。

## 要点

- **规格**：24 层、128 个专家中每个 token 选 8 个，中间 12 层共享权重循环三次，等效每 token 5.4B 激活参数。支持图片输入、文字输出，上下文最长 100 万 token。权重大部分是 NVFP4、最后四层 MXFP8，单个检查点约 20GB。
- **官方成绩（和 Gemma 4 31B、Qwen3.8-27B 等同尺寸开源模型比）**：
  - SWE-bench Verified（只用 bash）79.2%，Qwen3.8-27B 77.8%。
  - Terminal Bench v2.1 71.6%，略低于 Qwen3.8-27B 的 73.0%。
  - MCP Atlas 82.7%、BrowseComp 79.0%，工具调用和联网搜索两类领先。
  - 弱项：MultiChallenge（多轮对话）50.0%、ComplexConstraints（多约束指令）33.4%，都落后于 Qwen3.8-27B；事实知识量和同尺寸模型差不多。
- **硬件**：官方支持 1 台 NVIDIA DGX Spark、32GB 及以上内存的 Apple 芯片 Mac，其他 GPU 建议 32GB 以上显存。
- **能用的推理框架**：SGLang、vLLM、Hugging Face、Unsloth（含量化 GGUF）、llama.cpp（量化 GGUF）、Ollama 和 Exo（Apple 芯片）。也可以在官方的 Tinker 平台上通过 API 调用或微调。
- **社区实测（Hacker News 讨论，1030 分）**：
  - M1 Pro 32GB 跑 4-bit 量化约每秒 10 个 token，M1 Max 64GB 约每秒 20 个。
  - 有人在 M4 Pro 64GB 的 Mac mini 上跑 43 道数据分析题，成绩和 DeepSeek V4.1 Flash、Sonnet 5 同档。
  - 同一位用户遇到一次“停不下来”：找数据问题时连续跑了 3 个多小时、180 多次工具调用，最后手动打断。

## 适合谁

- 有 32GB 以上内存的 Mac 或大显存显卡，想在本地离线跑编程代理模型的人。
- 自己有 GPU 服务器、想给 OpenCode 这类支持 OpenAI 兼容接口的代理接一个便宜开源模型的人（用 vLLM 或 SGLang 起服务）。
- 做需要大量工具调用、联网搜索的代理的人：这几项是它的强项。

## 注意事项

- **成绩是官方自己挑的**：表格来自发布文和模型卡，各模型的评测设置不完全一样；社区真实体验还不多，以自己的任务为准。
- **给代理设上限**：社区已经有人遇到它在代理模式下长时间停不下来，接进自动化流程时要设最大步数和超时。
- **只有 CPU 的机器不太合适**：官方没有列纯 CPU 方案。量化 GGUF 可以用 llama.cpp 跑，但速度没人报过，别指望在普通 ARM VPS 上流畅用。
- **聊天机器人别急着换**：多轮对话和多约束指令是它的弱项，纯聊天用途不一定比现在的模型好。
- **许可**：Apache 2.0 可以商用，同时要遵守官方的可接受使用政策。
- 本条没有实际下载运行过，内容来自官方发布文、Hugging Face 模型卡和 Hacker News 讨论。

## 来源

- 社区：Hacker News（https://news.ycombinator.com/item?id=50581260 ，1030 分）
- 官方发布文：https://thinkingmachines.ai/news/inkling-2-mini/
- 模型卡与权重：https://huggingface.co/thinkingmachines/Inkling-2-Mini
- 原文日期：2026-11-20
- 收录日期：2026-11-21
