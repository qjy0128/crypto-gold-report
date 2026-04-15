# crypto-gold-report

面向 OpenClaw 的 BTC / 黄金市场报告 skill，使用当前环境可用的网页搜索、抓取、阅读工具获取实时价格、情绪和新闻。

## 本次校验结论（2026-04-15）

### 可用价格与情绪源

- BTC：Binance、Kraken、OKX、Bybit、Gate、CoinGecko、Blockchain.info
- 国际金价：Swissquote、Investing Gold、TradingEconomics、Kitco
- 国内金价：上海黄金交易所 `shanghaiAuAuto`、上海黄金交易所 `yshqbg`、中国银行外汇牌价
- 情绪：Alternative.me

### 可用新闻源

核心源：
- 金十、财联社、第一财经、FXStreet News、FXStreet Crypto、FXStreet Gold、Investing Crypto、Investing Commodities、investingLive Gold、Kitco News

扩展源：
- BTC 扩展：Cointelegraph Bitcoin、Cointelegraph Regulation、Cointelegraph Markets、Cointelegraph Bitcoin ETF、CoinDesk Crypto、CryptoSlate Bitcoin、crypto.news、crypto.news Regulation、DL News Markets、DL News Regulation
- 黄金扩展：World Gold Council Goldhub、TradingEconomics Gold News、MINING.com Gold/Commodity、CNBC Gold

宏观 / 分析补位源：
- CNBC Economy、CNBC Markets、BullionVault Gold News、FXEmpire Gold、Decrypt

结构化补充源：
- BTC：DL News ETF Tracker
- 黄金：Goldhub Data - Gold ETFs, holdings and flows；Goldhub Research - Gold ETF Flows；Goldhub Research / Data - Central Banks

### 已移出主路径的数据源

- CoinPaprika：可读但数据滞后
- CoinCap：当前抓取失败
- Bitfinex：当前抓取失败 / 限流
- GoldPrice.org：当前返回历史旧值
- 中国黄金集团主页：主营金属价格非当日
- sgenow.cn：当前同步滞后
- Yahoo Finance Crypto、CNN Business、BBC Business：当前抓取器返回异常状态
- The Block：当前环境 403
- Blockworks：当前环境重定向 / 403
- Reuters Commodities、MarketWatch：当前环境 401

## 当前执行策略

- 不再硬绑定 `mcp__web_reader__webReader`，执行 agent 只要能用当前环境可用的网页工具拿到真实来源即可
- 价格改为字段级降级，不要求整行同源
- 国际金价允许“价格”和“涨跌”分源，但来源必须拆开写清
- 国内黄金固定输出 `上海金基准价 + Au(T+D) + 国际金价折算人民币（估算）`
- `SHAU` 与 `BOC` 优先按表格列顺序提取，并做数值合理性校验，避免把序号、轮次或买入价误当成目标字段
- `Au(T+D)` 同样按表头列顺序提取：`最新价` 用第 2 列，`今开盘` 用第 5 列
- 新闻阶段先扫核心池，再按报告缺口分别扫 BTC 扩展池、黄金扩展池，最后才用宏观 / 分析补位池
- 结构化补充源只用于驱动因子或给新闻补强证据；除非页面本身带明确绝对时间且落在 12 小时窗口内，否则不进入 `利好 / 利空` 列表
- `Cointelegraph Regulation` 与 `crypto.news Regulation` 专门补 BTC 偏空、监管、交易所和安全事件缺口
- `Cointelegraph Markets` 与 `Cointelegraph Bitcoin ETF` 分别补 BTC 价格波动条目和 ETF 资金流条目
- `Cointelegraph Markets` 与 `Cointelegraph Bitcoin ETF` 的标签页时间不能直接信，必须下钻正文页确认绝对发布时间
- `CNBC Gold` 主要补黄金偏空、美元/利率、央行动向和油价冲击类条目；若列表页只有日期，必须下钻详情页补绝对时间
- `Kitco News` 列表页能直接给出 `time dateTime` 时，可以直接作为绝对时间使用
- `Investing Crypto/Commodities` 分类页若返回 `403`，当前轮次直接跳过，不做重复重试；只有拿到具体文章页时才继续验证
- `CNBC Economy` 与 `CNBC Markets` 用于显式收集联储、美元、美债、原油和风险偏好等宏观驱动因子；抓不到真实来源就留空，不做主观补全
- 先收集 12 小时内全部相关候选，再去重聚类，再按重要性排序
- 黄金补位优先顺序固定为：事实型金市条目 > 金价驱动型宏观条目 > 技术 / 展望稿
- BTC 报告与黄金报告都要求 `利好 5 条 + 利空 5 条`；不足时明确写 `暂无足够符合条件的条目`
- 宏观分析只保留实际抓到的 2-4 条，不硬填预设表格

## 文件

- `SKILL.md`：执行说明与报告模板
- `CHANGELOG.md`：变更记录
