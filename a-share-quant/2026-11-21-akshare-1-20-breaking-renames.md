# AKShare 1.20.6x 连着几个不兼容改动：新浪日线接口改名、12 个接口改返回表格，升级前先查一遍旧接口名

> 仅供学习研究，不构成投资建议。

**一句话结论**：AKShare 在 11 月 14 日到 15 日连发了 1.20.63、1.20.64、1.20.65 三个带“不兼容”标记的版本：新浪的 6 个股票日线接口改名（旧名直接报错）、12 个接口的返回值统一改成 pandas.DataFrame、删掉了 3 个第三方客户端函数和 2 个死接口。再加上 1.20.44 到 1.20.46 那一轮改名，很多老脚本里常见的 `stock_zh_a_daily`、`stock_zh_a_minute`、`tool_trade_date_hist_sina` 在新版里都已经不存在了。用 AKShare 跑定时选股或回测的，升级前一定先查自己用了哪些旧名字，最稳的办法是在 requirements 里锁定版本。

## 要点

- **1.20.64：新浪日线接口改名并加 period 参数**。`stock_zh_a_daily_sina` 改为 `stock_zh_a_hist_sina`，同批共 6 个（A 股、B 股、CDR 等），旧名称不保留，调用会报 `AttributeError`。新增 `period` 参数，可选 daily（默认）、weekly、monthly，周线和月线由日线合成。注意参数顺序变了：`period` 排在 `symbol` 后面，按位置传 `start_date`、`end_date` 的代码要改，按关键字传参的只需改名。`period="daily"` 时结果和改名前一样。
- **1.20.65：全部接口都返回表格**。12 个原来返回字符串、列表、元组或字典的接口改成返回 DataFrame，接口名和参数不变。比较常用的是 `futures_main_contract_sina`：原来返回逗号连接的字符串，现在要自己 `",".join(df["合约"])`。另外几个接口在非交易日原来返回 None 或空字典，现在返回只有列名的空表；个别取数失败时改为抛异常。
- **北交所代码前缀修正**：工具函数 `stock_a_code_to_symbol` 原来把 4、8、92 开头的北交所代码加上 sz 前缀，现在加 bj 前缀。拿它拼新浪、腾讯行情代码的脚本，北交所部分的结果会变。
- **1.20.63：删掉的东西**。`pro_api`、`set_token`、`get_token` 是第三方付费期货数据服务的客户端，已移除；`set_token` 以前会把令牌明文存到用户主目录的 `tk.csv`，删除后这个文件不会自动清掉，用过的可以自己删。另外移除了两个上游已经没数据的接口：`fund_lcx_rank_em`、`index_detail_hist_adjust_cni`。
- **查旧名字去哪了**：1.20.58 起有 `ak.renamed_info("旧接口名")`，返回改名还是移除、新名字、在哪个版本改的。调用旧名字时报错信息里也会直接给出新名字。
- **东方财富接口不再说“访问无限制”**：数据字典里“建议切换为 stock_zh_a_hist_em（访问无限制）”的说法被删了，官方说明东方财富行情接口有访问频率限制；被限流时可以按同类接口对照表换成新浪的接口。

## 实测

在 Debian 13 上新建虚拟环境装了 akshare 1.20.65 实际跑过：

- 访问 `ak.stock_zh_a_daily_sina` 报 `AttributeError`，提示已在 1.20.64 更名为 `stock_zh_a_hist_sina`。
- `stock_zh_a_hist_sina(symbol="sh600000", period="weekly", ...)` 正常返回周线，最近一根是 2026-11-20。
- `futures_main_contract_sina` 返回的确实是 DataFrame。
- `stock_a_code_to_symbol("920001")` 返回 `bj920001`。
- 用 `ak.renamed_info` 查了几个老脚本里常见的名字，在 1.20.65 里都已经不存在：
  - `stock_zh_a_daily` → `stock_zh_a_hist_sina`（1.20.46 改名）
  - `stock_zh_a_minute` → `stock_zh_a_hist_min_sina`（1.20.46）
  - `stock_zh_index_daily` → `index_zh_daily_sina`（1.20.44）
  - `tool_trade_date_hist_sina` → `calendar_trade_date_hist_sina`（1.20.45）
  - 涨停池几个接口（`stock_zt_pool_em`、`stock_zt_pool_previous_em`、`stock_zt_pool_zbgc_em`）没改名。

## 适合谁

- 用 AKShare 跑盘中监控、每日选股、回测的人，尤其是没锁版本、会定期 `pip install -U` 的。
- 用新浪日线或周线数据、又担心东方财富接口被限流的人：新版新浪接口能直接出周线、月线。
- 交易北交所股票、用 `stock_a_code_to_symbol` 拼代码的人。

## 注意事项

- **别在盘中升级**：定时任务的环境一旦被升级，旧名字会直接报错，当天的监控就断了。先在单独的虚拟环境里装新版，把脚本跑一遍，再切过去。
- **锁版本**：在 requirements 里写死版本号（例如 `akshare==1.20.35`），升级当成一次有计划的改动来做。
- **按位置传参的要特别查**：`stock_zh_a_hist_sina` 的参数顺序变了，按位置传日期不一定报错，可能把日期当成别的参数。官方说 `period` 填错会抛异常，但最好还是全部改成按关键字传参。
- **两个接口官方没联网实测**：`index_code_id_map_em`、`futures_position_rank_other_dce` 在 1.20.65 只做了离线测试，用到的要自己验证。
- 本条只测了上面列出的几个接口，没有逐个跑完全部改动。

## 来源

- 社区：GitHub（akfamily/akshare）、PyPI
- 官方更新日志：https://akshare.akfamily.xyz/changelog.html
- 接口更名一览表：https://akshare.akfamily.xyz/changelog.html#changelog-rename
- PyPI：https://pypi.org/project/akshare
- 原文日期：1.20.63 发布于 2026-11-14，1.20.64、1.20.65 发布于 2026-11-15（PyPI 上传时间，UTC）
- 收录日期：2026-11-21
