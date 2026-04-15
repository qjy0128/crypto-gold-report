---
name: crypto-gold-report
description: 生成比特币与黄金市场报告。使用当前环境可用的网页搜索、抓取、阅读工具获取实时价格、情绪和新闻，输出两份中文报告。
---

# 加密货币与黄金市场报告
> 工具：使用当前环境里可用的网页搜索 / 打开 / 抓取 / 阅读工具
> 优先：能直接返回正文、时间戳、链接的工具优先；如果 `mcp__web_reader__webReader` 存在可以优先用，但不是硬依赖
> 输出：BTC + 黄金两份中文报告
> 原则：字段级抓取；抓不到就写 `N/A`；英文标题必须翻译成中文；新闻先全量收集，再去重，再排序

## 工具策略

- 不要把执行绑定到单一 MCP 或单一函数名
- 如果当前环境没有 `mcp__web_reader__webReader`，改用等价工具继续执行，例如网页搜索、浏览器打开、页面抓取、正文读取、搜索结果点击
- 只要最终能拿到真实来源、真实标题、真实时间、真实链接，就可以使用当前 agent 最熟悉、最稳定的网页工具组合
- 不要因为某个指定工具缺失就停止任务

## 禁止

- 编造价格、资讯、时间、来源
- 使用“综合判断”“综合分析”“市场共识”“综合研判”作为来源
- 用陈旧数据冒充实时数据
- 用估算价冒充官方盘面价
- 同一事件在同一份报告里重复出现
- 在标题列保留英文原文（来源名除外）

## 已验证源状态（2026-04-15）

**价格 / 情绪主源：**

- BTC：Binance、Kraken、OKX、Bybit、Gate、CoinGecko、Blockchain.info
- 国际金价：Swissquote、Investing Gold、TradingEconomics、Kitco
- 国内金价：上海黄金交易所 `shanghaiAuAuto`、上海黄金交易所 `yshqbg`、中国银行外汇牌价
- 情绪：Alternative.me

**新闻核心源：**

- 金十
- 财联社电报
- 第一财经
- FXStreet News
- FXStreet Crypto
- FXStreet Gold
- Investing Crypto
- Investing Commodities
- investingLive Gold
- Kitco News

**新闻扩展源（按市场拆分，可纳入候选池）：**

- BTC 扩展：Cointelegraph Bitcoin、Cointelegraph Regulation、CoinDesk Crypto、CryptoSlate Bitcoin、crypto.news、crypto.news Regulation、DL News Markets、DL News Regulation
- 黄金扩展：World Gold Council Goldhub、TradingEconomics Gold News、MINING.com Gold/Commodity、CNBC Gold

**低优先级补位源：**

- Decrypt：可读，但列表页和部分文章页时间粒度偏弱，只有拿到明确时分时才可入池
- BullionVault Gold News：可读，但更新频率偏低，更适合 24 小时窗口或黄金宏观补位
- FXEmpire Gold：可读且有明确时间戳，但以分析/技术稿为主，只能在事实型条目不足时补位

**结构化补充源（默认不计入 12 小时新闻位）：**

- BTC：DL News ETF Tracker
- 黄金：Goldhub Data - Gold ETFs, holdings and flows；Goldhub Research - Gold ETF Flows（月度）；Goldhub Research - Central Bank Gold Reserves Survey / Goldhub Data Central Banks

**不要再尝试：**

- `https://api.coinpaprika.com/v1/tickers/btc-bitcoin`：当前可读，但数据滞后
- `https://api.coincap.io/v2/assets/bitcoin`：当前抓取失败
- `https://api.bitfinex.com/v1/pubticker/btcusd`：当前抓取失败 / 限流
- `https://data-asg.goldprice.org/dbXRates/USD`：当前返回历史旧值，不作实时源
- `https://www.chinagoldgroup.com/`：主营金属价格非当日，不作实时源
- `https://www.sgenow.cn/`：当前同步滞后，不作主源
- `https://finance.yahoo.com/topic/crypto/`
- `https://edition.cnn.com/business`
- `https://www.bbc.com/news/business`
- `https://www.theblock.co/`：当前环境 403
- `https://blockworks.co/`：当前环境重定向 / 403，不作稳定源
- `https://www.reuters.com/markets/commodities/`：当前环境 401
- `https://www.marketwatch.com/`：当前环境 401

## 调用预算

- 不要写死“总共只调几次”，实际调用量取决于环境工具能力、列表页可读性、详情页数量和去重冲突
- 参考预算只统计“源页 / 列表页”抓取，不统计详情页展开：
  - 价格 / 情绪：最少 3 次，最多 11 次
  - 新闻核心源：固定 10 次
  - BTC 扩展源：0-10 次，只在 BTC 报告去重后仍然缺条时继续扩
  - 黄金扩展源：0-4 次，只在黄金报告去重后仍然缺条时继续扩
  - 宏观 / 分析补位源：0-5 次，只在对应报告一侧仍然缺条时继续扩
  - 结构化补充源：0-4 次，只在驱动因子或资金流证据不足时使用
- 详情页点击 / 打开次数是动态的，通常为“候选条目数 - 能在列表页直接确认的条目数”
- 新闻阶段禁止“够数即停”。只要当前列出的源还没扫完，就不能提前结束

## 执行步骤

### 步骤 1：价格与情绪（字段级抓取）

- 先抓价格和情绪，再抓新闻
- 某个字段拿到可用值后，立即停止为该字段继续尝试备用源
- 字段可以分别降级，不要要求整行必须同源

#### 1A. BTC（字段级抓取）

**完整源，按顺序尝试：**

1. `https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT`
   - `lastPrice`：价格
   - `priceChangePercent`：24h 涨跌幅
   - `quoteVolume`：24h 成交额
   - `closeTime`：更新时间
2. `https://api.kraken.com/0/public/Ticker?pair=XBTUSD`
   - `result.XXBTZUSD.c[0]`：最新价
   - `result.XXBTZUSD.o`：开盘价，可计算涨跌幅
   - `result.XXBTZUSD.v[1]`：24h 成交量
3. `https://www.okx.com/api/v5/market/ticker?instId=BTC-USDT`
   - `data[0].last`：价格
   - `data[0].open24h`：24h 开盘价，可计算涨跌幅
   - `data[0].volCcy24h`：24h 成交额
   - `data[0].ts`：更新时间
4. `https://api.bybit.com/v5/market/tickers?category=spot&symbol=BTCUSDT`
   - `result.list[0].lastPrice`：价格
   - `result.list[0].price24hPcnt`：24h 涨跌幅
   - `result.list[0].turnover24h`：24h 成交额
5. `https://api.gateio.ws/api/v4/spot/tickers?currency_pair=BTC_USDT`
   - `[0].last`：价格
   - `[0].change_percentage`：24h 涨跌幅
   - `[0].quote_volume`：24h 成交额

**仅价格备用：**

6. `https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd`
7. `https://api.blockchain.info/stats`

**字段规则：**

- `BTC/USD`、`24h 涨跌幅`、`24h 成交额`可以来自不同源，但每一行都要在来源列写清具体来源
- 某字段所有可用源都失败时，该字段写 `N/A`
- 仅价格源不能拿来补涨跌幅或成交额
- 有明确时间戳时，`价格更新时间`用源时间；没有时间戳时，写“抓取时刻 {HH:MM}”

#### 1B. 国际金价（字段级抓取）

**价格优先级：**

1. `https://forex-data-feed.swissquote.com/public-quotes/bboquotes/instrument/XAU/USD`
   - `spreadProfilePrices[0].bid`、`ask`
   - 价格可用 `(bid + ask) / 2`
   - `ts`：更新时间
2. `https://www.investing.com/commodities/gold`
   - 页面顶部提取 Gold 当前价格
3. `https://tradingeconomics.com/commodity/gold`
   - 页面提取当前价格
4. `https://www.kitco.com/news`
   - 仅作最后兜底，需页面中出现明确金价数字才可用

**涨跌优先级：**

1. `https://www.investing.com/commodities/gold`
2. `https://tradingeconomics.com/commodity/gold`
3. `https://www.kitco.com/news`

**字段规则：**

- 国际金价一行允许多源合成，来源列写成 `Swissquote（价格） + Investing.com（涨跌）`
- 价格抓到但涨跌抓不到时，涨跌列写 `—`
- `价格更新时间`优先用 Swissquote 时间戳；否则写“抓取时刻 {HH:MM}”

#### 1C. 国内金价

**上海金基准价（官方）：**

1. `https://www.sge.com.cn/sjzx/shanghaiAuAuto?start_date={最近已发布交易日}&end_date={最近已发布交易日}`
   - 取 `SHAU 早盘` 的 `价格/PRC` 作为 `上海金基准价`
   - 若无 `早盘`，用该页最早一条 `SHAU` 记录
   - 必须同时记录 `交易日期/Trade Date`
   - 优先按表格列顺序提取，不要用“行内第一串数字”这种脆弱规则
   - 去掉 HTML 标签后，`SHAU 早盘` 常见列顺序为 `序号 | 交易日期 | 品种 | 场次 | 报价轮次 | 价格/PRC | 涨跌 | 成交量 | 成交额`
   - `上海金基准价` 取该行 `价格/PRC` 对应列，通常是该行第 6 个数据单元；忽略 `序号` 与 `报价轮次`
   - 做合理性校验：若抽到 `1`、`2` 这类轮次值，或数值明显不在人民币金价合理区间（例如 `800-2000 元/克`），视为抓错列并重新定位
   - 回退顺序：先查今天；若页面显示“系统无查询的数据”，再依次回退前 1、2、3、4、5 个自然日，取第一个有 `SHAU` 数据的日期

**Au(T+D) 延时行情（官方）：**

2. `https://www.sge.com.cn/sjzx/yshqbg`
   - 取 `Au(T+D)` 行的 `最新价`
   - `今开盘` 可用于计算日内涨跌幅：`(最新价 - 今开盘) / 今开盘`
   - 必须同时记录页面日期 `上海黄金交易所{YYYY年MM月DD日}延时行情`
   - 优先按表头列顺序提取，不要把 `最高价` 或 `最低价` 误当成 `今开盘`
   - 当前页面常见表头顺序为 `合约 | 最新价 | 最高价 | 最低价 | 今开盘`
   - `Au(T+D)` 行取第 2 列作为 `最新价`，第 5 列作为 `今开盘`
   - 做合理性校验：`最新价` 与 `今开盘` 通常都应落在人民币金价合理区间（例如 `800-2000 元/克`）；若抽到异常值，视为列错位并重新定位

**人民币折算金价（估算，不是官方盘面价）：**

3. 先用国际金价
4. 再抓 `https://www.boc.cn/sourcedb/whpj/`
   - 使用“美元”行的 `中行折算价`
   - 优先把 `<tr>` 去标签后按单元格顺序提取；美元行常见顺序为 `货币名称 | 现汇买入价 | 现钞买入价 | 现汇卖出价 | 现钞卖出价 | 中行折算价 | 发布时间`
   - 行名优先匹配 `美元`；若当前工具把页面错误解码为乱码，也允许匹配常见乱码别名如 `ç¾...`
   - `中行折算价` 取该行第 6 列数值，不要误取买入/卖出价
   - 做合理性校验：美元 `中行折算价` 通常落在 `500-900`；若拿到异常值，视为抓错列并重新定位
5. 估算价 = 国际金价 × (`中行折算价` / 100) / 31.1035
6. 来源列写 `国际金价源 + 中国银行（估算）`

**规则：**

- 国内黄金优先走上金所官方页，不再用聚合站替代
- `上海金基准价`可以使用最近已发布交易日的官方值，但必须在正文或来源中带上交易日期
- `Au(T+D)`使用上金所当日延时行情；如果只有价格没有今开盘，涨跌列写 `—`
- `上海金基准价`、`Au(T+D)`、`折算金价`分开输出，不要互相冒充
- 只有上金所页面不可读时，才允许对应字段写 `N/A`

#### 1D. 恐惧贪婪指数

- `https://api.alternative.me/fng/`
- 取 `data[0].value` 与 `data[0].value_classification`
- 失败时写 `N/A`

### 步骤 2：新闻与宏观因子（全量收集）

#### 2A. 核心新闻池：每轮都要扫

1. `https://www.jin10.com`
2. `https://www.cls.cn/telegraph`
3. `https://www.yicai.com/news/`
4. `https://www.fxstreet.com/news`
5. `https://www.fxstreet.com/cryptocurrencies/news`
6. `https://www.fxstreet.com/markets/commodities/metals/gold`
7. `https://www.investing.com/news/cryptocurrency-news`
8. `https://www.investing.com/news/commodities-news`
9. `https://www.forexlive.com/tag/gold/`（若跳转则落到 `https://investinglive.com/Tag/gold/`）
10. `https://www.kitco.com/news`

#### 2B. BTC 扩展池：BTC 报告去重后仍不足时继续扩

11. `https://cointelegraph.com/tags/bitcoin`
12. `https://cointelegraph.com/tags/regulation`
13. `https://cointelegraph.com/tags/markets`
14. `https://cointelegraph.com/tags/bitcoin-etf`
15. `https://www.coindesk.com/tag/crypto/`
16. `https://cryptoslate.com/news/bitcoin/`
17. `https://crypto.news/news/`
18. `https://crypto.news/regulation/`
19. `https://www.dlnews.com/articles/markets/`
20. `https://www.dlnews.com/articles/regulation/`

#### 2C. 黄金扩展池：黄金报告去重后仍不足时继续扩

21. `https://www.gold.org/goldhub/gold-focus/2026/04`
22. `https://tradingeconomics.com/commodity/gold/news`
23. `https://www.mining.com/commodity/gold/`
24. `https://www.cnbc.com/gold/`

#### 2D. 宏观 / 分析补位池：对应报告仍然缺条时最后再扩

25. `https://www.cnbc.com/economy/`
26. `https://www.cnbc.com/markets/`
27. `https://www.bullionvault.com/gold-news`（只有能确认时分，或你主动放宽到 24 小时时才入池）
28. `https://www.fxempire.com/commodities/gold`
29. `https://decrypt.co/news`（只有能确认时分时才入池）

#### 2E. 结构化补充源：资金流 / ETF / 央行持仓 / 慢变量

30. `https://www.dlnews.com/etf-tracker/`
31. `https://www.gold.org/goldhub/data/gold-etfs-holdings-and-flows`
32. `https://www.gold.org/goldhub/research/gold-etfs-holdings-and-flows/{YYYY/MM}`
33. `https://www.gold.org/goldhub/data#central-banks`
34. `https://www.gold.org/goldhub/research/central-banks`

#### 2F. 单源提取规则

- 核心池全部都要检查，不允许因为“已经够 5 条”就提前停
- BTC 报告缺条时，先扫 BTC 扩展池；黄金报告缺条时，先扫黄金扩展池；不要把两边的扩展池混着扫
- 对应市场扩展池用尽后仍有缺口，再进入宏观 / 分析补位池
- 宏观 / 分析补位池只能在核心池和对应市场扩展池都扫完后使用
- 宏观 / 驱动因子不是“顺手总结”，而是要显式收集的候选池；每轮都要为每份报告至少收集 `2-4` 条带来源的宏观候选
- 宏观候选优先覆盖：美联储 / 降息预期、美元、美国国债收益率、原油、地缘政治、风险偏好 / 美股、ETF / 央行流向
- 结构化补充源默认用于“宏观 / 驱动因素”或给新闻条目补强证据，不直接顶替 12 小时新闻位
- 结构化补充源只有在页面本身给出明确绝对时间且落在 12 小时窗口内时，才允许进入利好 / 利空列表
- `Cointelegraph Regulation`、`crypto.news Regulation` 只收录与 BTC / ETF / 监管 / 交易所 / 安全事件直接相关的条目，纯山寨币或泛 Web3 监管稿不入池
- `Cointelegraph Markets` 优先补 BTC 价格波动、爆仓、风险偏好和技术位触发类条目；只收录与 BTC 直接相关的市场稿
- `Cointelegraph Bitcoin ETF` 优先补现货 ETF 资金流、发行方动作和 ETF 驱动类条目
- `Cointelegraph Markets`、`Cointelegraph Bitcoin ETF` 的标签页相对时间可能与正文页绝对时间不一致；这两类页只用于发现候选，是否入池必须以下钻正文页后的绝对时间为准
- `CNBC Gold` 优先用来补黄金偏空、美元/利率、央行动向和油价冲击类条目；若列表页没有时分，必须下钻详情页读取 `article:published_time` 或 `datePublished`
- `CNBC Economy` 优先补联储、通胀、就业、油价、地缘政治冲击；`CNBC Markets` 优先补美债收益率、美元、跨资产风险偏好和市场联动
- `Kitco News` 列表页通常自带 `<time dateTime>`；若列表页已有绝对时间，允许直接入候选池，不必为时间单独下钻
- `Investing.com` 的分类页在当前环境下可能直接返回 `403`；若发生 `403/Cloudflare`，本轮将该源标记为不可用并继续，不要反复重试
- 当 `Investing.com` 分类页不可用但你已通过其他入口拿到具体文章 URL 时，仍可对文章页做单篇验证并按绝对时间入池
- 宏观因子若没有抓到真实来源，就不要硬填，也不要用“综合判断”补位
- 月度、季度、年度、调查类数据只能作为慢变量或驱动因子，不能伪装成“今天的新闻”
- 每个源都要尽量提取过去 12 小时内的全部相关候选条目，不只抓 1 条
- 每条候选至少保留：中文标题、来源、绝对发布时间、链接、适用市场（BTC / 黄金 / 宏观）、初步方向（利好 / 利空 / 中性）
- 标题全部翻译成中文
- `5 minutes ago`、`10 hours ago` 这类相对时间要换算成绝对时间
- 只有日期没有时分的资讯直接丢弃；但如果详情页或结构化元数据能补出时分，则可以保留
- 返回空内容、错误页、只有导航广告、没有新闻列表的页面，视为该源本轮不可用，继续检查剩余源
- 宏观资讯可以同时用于 BTC 和黄金两份报告，但在每份报告内部仍然要先去重
- 黄金扩展池优先补这几类利好：央行购金、ETF 流入、实物需求走强、美元走弱 / 实际利率回落、矿端或运输端供给扰动、地缘避险升温
- 列出来的源是“验证过、推荐优先扫的候选池”，不是封闭白名单；如果当前环境还能稳定访问其他主流、可读、带明确时间戳的财经或行业源，也允许补充

**BTC 相关候选：**

- 直接提到 Bitcoin、BTC、ETF、加密监管、交易所、链上资金、机构配置、矿企、稳定币、风险偏好
- 明显影响 BTC 的宏观项：美元指数、美债收益率、纳指 / 美股风险偏好、地缘政治、流动性、ETF 资金流、监管与安全事件

**黄金相关候选：**

- 直接提到 Gold、XAU、bullion、央行购金、实物金、期货金、上海金、Au(T+D)
- 明显影响黄金的宏观项：美元指数、美债收益率、原油、地缘政治、避险情绪、通胀预期、联储政策、ETF 流向、矿端供给

#### 2G. 去重聚类规则

- 同一事件在同一份报告里只保留 1 条
- 认定为同一事件的条件：
  - 主体相同：例如“美联储 / 特朗普 / 高盛 / 伊朗 / ETF 发行方 / 上金所”
  - 动作相同：例如“申请 / 批准 / 讲话 / 停火 / 制裁 / 爆仓 / 创新高 / 回落”
  - 核心数字或结论相同：例如同一个价格突破位、同一个收益率变化、同一个政策表态
  - 发布时间相差不大，通常在 6 小时内
- 去重优先级：
  - `金十 > 财联社 > FXStreet > Cointelegraph > CoinDesk > CNBC > DL News > CryptoSlate > crypto.news > Kitco > Investing.com > TradingEconomics > MINING.com > World Gold Council > investingLive > 第一财经 > FXEmpire > Decrypt > BullionVault`
  - 若第一财经是独家或带独立数据，可提升优先级
- 同一事件若多源重复，只保留优先级最高、信息最完整、时间最明确的一条
- 同一事件不能同时出现在同一份报告的“利好”和“利空”里，必须选择主导影响方向

#### 2H. 重要性排序与保留数量

- 对去重后的候选按以下顺序排序：
  1. 市场影响强弱：政策、监管、ETF、央行 / 联储、地缘政治、收益率 / 美元急变，优先级最高
  2. 与资产的直接相关度：直接讲 BTC / 黄金，高于泛宏观评论
  3. 信息具体度：有明确数字、时间、对象、价格位、政策动作的，优先于空泛评论
  4. 新鲜度：同等条件下，越近越优先
  5. 交叉验证：被多个源重复报道的核心事件，优先于单源观点文
- 黄金报告补位时，先保留事实型金市条目，再保留金价驱动型宏观条目，最后才允许技术 / 展望型稿件补位
- 评论型、观点型、复盘型内容，只能在事实型条目不足时补位
- 每份报告目标：
  - 利好 5 条
  - 利空 5 条
  - 宏观 / 驱动因素 2-4 条
- 如果核心池、对应市场扩展池、宏观 / 分析补位池和可用的结构化补充源都扫完、完成去重后，某一侧仍不足 5 条，空位写 `暂无足够符合条件的条目`
- 不允许用超过 12 小时的旧闻补数，除非你明确把窗口改成 24 小时并在正文中说明

### 步骤 3：填充与扫描

- 每一个价格字段都必须有来源；没有就写 `N/A`
- 利好 / 利空表必须先完成去重，再按重要性排序，再填进 1-5 行
- 超过 12 小时的资讯必须删除
- 标题列不能保留英文原文
- 来源列不能出现“综合判断 / 综合分析 / 市场共识 / 综合研判”
- 宏观表格只保留实际抓到的 2-4 条，不要硬填固定行
- 如果国际金价一行用了多源，来源列必须写清“谁提供价格、谁提供涨跌”

### 步骤 4：输出

输出两份完整报告。

---

## 格式 A：BTC 报告

```markdown
📳 比特币市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：12小时内

1️⃣ 当前价格与市场情绪
| 指标 | 数值 | 来源 |
|------|------|------|
| BTC/USD | $XX,XXX / N/A | 来源名 |
| 24h 涨跌幅 | ±X.XX% / N/A | 来源名 |
| 24h 成交额 | $XX.XB / N/A | 来源名 |
| 恐惧贪婪指数 | XX（分类） / N/A | alternative.me |

价格更新时间：{源时间戳或抓取时刻}

2️⃣ 利好资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

3️⃣ 利空资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

4️⃣ 趋势分析

宏观/驱动因素（只填实际抓到的 2-4 条）：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
| | | |

利好因素：
- （来源：XXX）

利空因素：
- （来源：XXX）

5️⃣ 趋势预测

| 周期 | 趋势 | 关键价位 | 核心依据 |
|------|------|----------|----------|
| 24小时 | | | |
| 3天 | | | |
| 7天 | | | |
| 1个月 | | | |

⚠️ 风险提示：本报告仅供参考，不构成投资建议。
```

## 格式 B：黄金报告

```markdown
📳 黄金市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：12小时内

1️⃣ 当前价格与市场情绪
| 市场 | 价格 | 涨跌 | 来源 |
|------|------|------|------|
| 国际金价 (XAU/USD) | X,XXX 美元/盎司 / N/A | ±X.XX% / — | 来源名 |
| 上海金基准价 | XXX 元/克 / N/A | — | 上金所上海金基准行情（交易日期：YYYY-MM-DD） |
| Au(T+D) | XXX 元/克 / N/A | ±X.XX% / — | 上金所延时行情（页面日期：YYYY-MM-DD） |
| 国际金价折算人民币（估算） | XXX 元/克 / N/A | — | 国际金价源 + 中国银行（估算） |

| 指标 | 数值 | 来源 |
|------|------|------|
| 恐惧贪婪指数 | XX（分类） / N/A | alternative.me |

价格更新时间：{源时间戳或抓取时刻}

2️⃣ 利好资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

3️⃣ 利空资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

4️⃣ 趋势分析

宏观/驱动因素（只填实际抓到的 2-4 条）：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
| | | |

利好因素：
- （来源：XXX）

利空因素：
- （来源：XXX）

5️⃣ 趋势预测

| 周期 | 趋势 | 关键价位 | 核心依据 |
|------|------|----------|----------|
| 24小时 | | | |
| 3天 | | | |
| 7天 | | | |
| 1个月 | | | |

⚠️ 风险提示：本报告仅供参考，不构成投资建议。
```
