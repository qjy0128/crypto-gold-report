# Changelog

All notable changes to this project will be documented in this file.

## [6.7.0] - 2026-04-15

### Changed - 继续补 BTC 高产扩展页并加固 Au(T+D) 抽取

- 为 `Au(T+D)` 增加按表头列顺序提取与合理性校验规则，避免把最高价/最低价误抓成 `今开盘`
- 新增 BTC 扩展源：`Cointelegraph Markets`、`Cointelegraph Bitcoin ETF`
- 明确 `Cointelegraph Markets` 只用于 BTC 价格波动、爆仓、风险偏好和技术位触发类条目
- 明确 `Cointelegraph Bitcoin ETF` 只用于现货 ETF 资金流、发行方动作和 ETF 驱动类条目
- 明确 `Cointelegraph Markets`、`Cointelegraph Bitcoin ETF` 标签页只能用于发现候选，是否入池必须以下钻正文页后的绝对时间为准
- 明确 `Kitco News` 列表页可直接读取 `<time dateTime>` 作为绝对时间
- 明确 `Investing.com` 分类页若返回 `403/Cloudflare`，本轮直接跳过，不做重复重试；仅对已拿到的具体文章页继续验证
- 上调 BTC 扩展源参考预算至 `0-10`

## [6.6.0] - 2026-04-15

### Changed - 加固 BOC / SHAU 抽取并补入宏观驱动池

- 为 `SHAU` 增加按列提取与合理性校验规则，避免把序号或报价轮次误抓成 `上海金基准价`
- 为 `中国银行外汇牌价` 增加按列提取、乱码行名兼容与合理性校验规则，降低 `中行折算价` 抓取失败率
- 新增宏观 / 分析补位源：`CNBC Economy`、`CNBC Markets`
- 明确宏观 / 驱动因子需要显式收集，不得用“综合判断”硬填；优先覆盖联储、美元、美债收益率、原油、地缘政治和风险偏好
- 上调宏观 / 分析补位源参考预算至 `0-5`

## [6.5.0] - 2026-04-15

### Changed - 继续补 BTC 风险侧与黄金宏观侧可用源

- 新增 BTC 扩展源：`Cointelegraph Regulation`、`crypto.news Regulation`
- 新增黄金扩展源：`CNBC Gold`
- 明确 `Cointelegraph Regulation`、`crypto.news Regulation` 只用于 BTC / ETF / 监管 / 交易所 / 安全事件直接相关条目，避免把泛 Web3 监管稿混入 BTC 报告
- 明确 `CNBC Gold` 主要用于补黄金偏空、美元/利率、央行动向和油价冲击类条目；若列表页缺少时分，必须在详情页提取 `article:published_time` 或 `datePublished`
- 更新去重优先级，将 `CNBC` 纳入主序列

## [6.4.0] - 2026-04-15

### Changed - 补充 BTC 风险/监管源并加入结构化补充源

- 新增 BTC 扩展源：`DL News Markets`、`DL News Regulation`
- 修正黄金扩展源入口：将无效的 `MINING.com/tag/gold` 更正为 `MINING.com/commodity/gold`
- 新增结构化补充源：`DL News ETF Tracker`、`Goldhub Data - Gold ETFs, holdings and flows`、`Goldhub Research - Gold ETF Flows`、`Goldhub Research / Data - Central Banks`
- 明确结构化补充源默认只用于驱动因子和证据补强，除非页面本身带明确绝对时间且落在 12 小时窗口内，否则不得进入 `利好 / 利空` 列表
- 更新调用预算与去重优先级，使 `DL News` 和结构化补充源的扫描逻辑与当前规则一致
- 将当前环境下不可稳定访问的源补入黑名单：`The Block`、`Blockworks`、`Reuters Commodities`、`MarketWatch`

## [6.3.0] - 2026-04-15

### Changed - 拆分 BTC / 黄金扩展池并补强黄金新闻侧

- 将新闻扩展池按市场拆分为 `BTC 扩展池`、`黄金扩展池`、`宏观 / 分析补位池`
- 新增黄金专向扩展源：`TradingEconomics Gold News`、`MINING.com Gold`
- 新增黄金分析补位源：`FXEmpire Gold`
- 明确新闻扫描顺序：先扫核心池，再按缺口分别扫对应市场扩展池，最后才允许使用宏观 / 分析补位池
- 明确黄金补位顺序：`事实型金市条目 > 金价驱动型宏观条目 > 技术 / 展望稿`
- 调整去重优先级和预算说明，使新增黄金源与分池逻辑一致

## [6.2.0] - 2026-04-15

### Changed - 扩充新闻源并移除单一工具硬依赖

- 将 skill 顶部和 frontmatter 从“仅 `mcp__web_reader__webReader`”改为“使用当前环境可用的网页搜索 / 抓取 / 阅读工具”
- 明确要求：如果指定工具不存在，必须改用等价网页工具继续执行，不能因为少一个 MCP 就卡死
- 新增扩展新闻源：`Cointelegraph`、`CoinDesk`、`CryptoSlate`、`crypto.news`、`World Gold Council Goldhub`
- 新增低优先级补位源：`Decrypt`、`BullionVault Gold News`
- 将新闻流程改为“核心池全扫，缺条再扫扩展池”，同时保留去重、排序和 `5 + 5` 输出目标
- 调用预算改为动态说明，不再假装能用固定总次数覆盖所有列表页和详情页

## [6.1.0] - 2026-04-15

### Changed - 新闻采集改为全源扫描与排序输出

- 将新闻流程从 `Tier A / Tier B` 改为每轮固定扫描 10 个新闻源，不再允许“够数即停”
- 新增“候选池 -> 去重聚类 -> 重要性排序 -> 入表”规则，避免同一事件重复出现在同一份报告里
- 明确每份报告的目标改为 `利好 5 条 + 利空 5 条`，不足时写 `暂无足够符合条件的条目`
- 明确新闻阶段预算为固定 10 次调用，总预算更新为最少 13 次、最多 21 次 `webReader` 调用

## [6.0.1] - 2026-04-15

### Fixed - 国内黄金源恢复为上金所官方路径

- 国内黄金主源切换为上海黄金交易所官方 `shanghaiAuAuto` 与 `yshqbg`
- `上海金基准价` 恢复为可验证输出，允许使用最近已发布交易日并强制附带交易日期
- `Au(T+D)` 恢复为可验证输出，使用上金所延时行情并按 `今开盘` 计算日内涨跌
- `sgenow.cn` 降级，不再作为主源
