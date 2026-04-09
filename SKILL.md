---
name: crypto-gold-report
description: 生成黄金和比特币市场资讯报告。仅使用 mcp__web_reader__webReader。
---

# 加密货币与黄金市场报告

> 工具：仅 `mcp__web_reader__webReader` | 输出：BTC + 黄金两份报告
> **语言：全部中文。英文/外文资讯标题必须翻译为中文**
> **翻译示例**：原标题 "Bitcoin climbs above $70,000" → 报告中写 "比特币攀升突破7万美元"（仅来源列保留英文如 CoinDesk）

---

## ⛔ 禁止

- 编造价格、资讯、来源
- "综合判断""综合分析""市场共识""综合研判"作为来源
- 保留英文/外文标题（来源名称如 CoinDesk 除外）

---

## 执行步骤

### 步骤 1：价格数据（4次 webReader 调用）

**BTC 价格**（第1个成功即停，优先选含完整数据的源）：
1. `https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT`
   → JSON: `lastPrice`（价格）、`priceChangePercent`（涨跌%）、`quoteVolume`（成交量）
2. `https://api.coinpaprika.com/v1/tickers/btc-bitcoin`
   → JSON: `quotes.USD.price`（价格）、`quotes.USD.percent_change_24h`（涨跌%）、`quotes.USD.volume_24h`（成交量）
3. `https://api.coincap.io/v2/assets/bitcoin`
   → JSON: `data.priceUsd`（价格）、`data.changePercent24Hr`（涨跌%）、`data.volumeUsd24Hr`（成交量）
4. `https://api.kraken.com/0/public/Ticker?pair=XBTUSD`
   → JSON: `result.XXBTZUSD.c[0]`（最新价）、`result.XXBTZUSD.v[1]`（24h成交量）、`result.XXBTZUSD.p[1]`（24h加权均价，可算涨跌%）
5. `https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd`
   → JSON: `bitcoin.usd`（仅价格，无涨跌幅和成交量）
6. `https://api.blockchain.info/stats`
   → JSON: `market_price_usd`（仅价格）
7. `https://api.bitfinex.com/v1/pubticker/btcusd`
   → JSON: `last_price`（价格）、`volume`（成交量）

**国际金价**（第1个成功即停）：
1. `https://data-asg.goldprice.org/dbXRates/USD` → JSON: `items[0].xauPrice`（价格）、`items[0].chgXau`（涨跌额）、`items[0].pcXau`（涨跌%）
2. `https://forex-data-feed.swissquote.com/public-quotes/bboquotes/instrument/XAU/USD` → JSON: `spreadProfilePrices[0].bid`（买入价）、`spreadProfilePrices[0].ask`（卖出价）
3. `https://tradingeconomics.com/commodity/gold` → HTML 页面提取 XAU/USD 价格和涨跌幅

**恐惧贪婪指数**：
`https://api.alternative.me/fng/` → JSON: `data[0].value`、`data[0].value_classification`

### 步骤 2：国内金价（1-2次调用）

主源：`https://www.chinagoldgroup.com/` → 查找"主营金属实时价格"表格，提取黄金T+D价格和上海金基准价

主源失败时，备用计算法：
1. 用步骤1获取的国际金价（美元/盎司）
2. 抓取 `https://www.boc.cn/sourcedb/whpj/` 获取美元兑人民币汇率
3. 估算价 = 国际金价 × 汇率 ÷ 31.1035（盎司转克）
4. 标注"（估算价，基于国际金价换算）"

两个源均失败才标注 N/A

### 步骤 3：新闻资讯（9次调用）

⚠️ 每个源调用后，若返回空内容/JS渲染页/无新闻条目 → 立即跳过，不重试

**中文综合财经**：
1. `https://www.jin10.com` → 金十数据，实时中文财经快讯 ✅
2. `https://www.cls.cn/telegraph` → 财联社电报，中文财经快讯
3. `https://www.yicai.com/news/` → 第一财经，中文财经新闻

**英文加密货币**：
4. `https://www.coindesk.com` → CoinDesk，英文加密货币新闻 ✅
5. `https://finance.yahoo.com/topic/crypto/` → Yahoo Finance加密货币，50+条聚合新闻 ✅

**英文宏观/黄金**：
6. `https://edition.cnn.com/business` → CNN Business，英文宏观经济新闻 ✅
7. `https://www.bbc.com/news/business` → BBC Business，英文宏观经济新闻 ✅
8. `https://www.investing.com/news/commodities-news` → Investing.com 商品/黄金新闻，含时间戳 ✅
9. `https://www.forexlive.com/tag/gold/` → ForexLive（现 investingLive）黄金专题新闻，含时间戳 ✅

**已确认不可用（不要尝试）**：wallstreetcn.com（JS）、cointelegraph.com（JS）、theblockbeats.info（仅行情）、kitco.com（1214错误）

**提取规则**：
- 标题：英文必须翻译为中文。示例：原标题"Bitcoin climbs above $70,000" → 填入表格时写"比特币攀升突破7万美元"
- 时间：仅保留12小时内（当前时间 - 发布时间 > 12小时则丢弃）
- 去重：同一事件保留一条，优先级 金十 > 财联社 > 第一财经 > CoinDesk > Yahoo > CNN > BBC > Investing.com > ForexLive
- 分类：BTC利好 / BTC利空 / 黄金利好 / 黄金利空（宏观资讯两边都用）
- 不足时标注"暂无"，不编造

### 步骤 4：填充并输出报告

按下方格式输出两份报告。每条资讯标注真实来源媒体名。利好/利空因素注明来源。

填充后执行扫描：
- 来源含"综合判断/综合分析/市场共识/综合研判" → 删除
- 标题含英文/外文 → 翻译为中文（逐条检查，不可保留原文。如 "Bitcoin climbs above $70,000" 必须改为 "比特币攀升突破7万美元"）
- 发布时间超过12小时 → 删除
- 价格来源列为空 → 补上数据源名

### 步骤 5：输出

输出两份完整报告。

---

## 格式A：BTC报告

```
📊 比特币市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：12小时内

1️⃣ 当前价格与市场情绪

| 指标 | 数值 | 来源 |
|------|------|------|
| BTC/USD | $XX,XXX | 来源名 |
| 24h 涨跌幅 | ±X.XX% 📈/📉 | 来源名 |
| 24h 成交量 | $XX.XB | 来源名 |
| 恐惧贪婪指数 | XX（分类） | alternative.me |

价格更新时间：{YYYY}-{MM}-{DD} {HH}:{MM} UTC

2️⃣ 利好资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |

3️⃣ 利空资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |

4️⃣ 趋势分析

宏观背景：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
| 美联储政策 | | |
| 美元指数 | | |
| 原油价格 | | |
| 地缘政治 | | |
| 美股表现 | | |

利好因素：
- ✅ （来源：XXX）

利空因素：
- ❌ （来源：XXX）

5️⃣ 趋势预测

| 周期 | 趋势 | 关键价位 | 核心依据 |
|------|------|----------|----------|
| 24小时 | | | |
| 3天 | | | |
| 7天 | | | |
| 1个月 | | | |

⚠️ 风险提示：本报告仅供参考，不构成投资建议。
```

## 格式B：黄金报告

```
📊 黄金市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：12小时内

1️⃣ 当前价格与市场情绪

| 市场 | 价格 | 涨跌 | 来源 |
|------|------|------|------|
| 国际金价 (XAU/USD) | X,XXX 美元/盎司 | ±X.XX% 📈/📉 | 来源名 |
| 上海金基准价 | XXX 元/克 | — | 来源名 |
| 黄金T+D | XXX 元/克 | ±X.XX% | 来源名 |

| 指标 | 数值 | 来源 |
|------|------|------|
| 恐惧贪婪指数 | XX（分类） | alternative.me |

价格更新时间：{YYYY}-{MM}-{DD} {HH}:{MM}

2️⃣ 利好资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |

3️⃣ 利空资讯

| # | 标题 | 来源 | 发布时间 | 链接 |
|---|------|------|----------|------|
| 1 | | | | |

4️⃣ 趋势分析

宏观背景：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
| 美联储政策 | | |
| 美元指数 | | |
| 原油价格 | | |
| 地缘政治 | | |
| 实际利率 | | |

利好因素：
- ✅ （来源：XXX）

利空因素：
- ❌ （来源：XXX）

5️⃣ 趋势预测

| 周期 | 趋势 | 关键价位 | 核心依据 |
|------|------|----------|----------|
| 24小时 | | | |
| 3天 | | | |
| 7天 | | | |
| 1个月 | | | |

⚠️ 风险提示：本报告仅供参考，不构成投资建议。
```