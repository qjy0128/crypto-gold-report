---
name: crypto-gold-report
description: 生成黄金和比特币市场资讯报告。通过 OpenClaw cron 定时任务触发。仅使用 mcp__web_reader__webReader 抓取数据。
---

# 加密货币与黄金市场报告

> **平台**: OpenClaw | **触发**: cron / 用户请求
> **唯一工具**: `mcp__web_reader__webReader`（本技能不使用 WebSearch）
> **报告数量**: 每次生成两份独立报告（BTC + 黄金）

---

## ⛔ 禁止事项

- 禁止编造价格、资讯、来源
- 禁止使用"综合判断""综合分析""市场共识""综合研判"作为来源
- 禁止使用 WebSearch（本环境无 API Key，无法使用）

---

## 输出格式

### 格式A：比特币市场报告

```
📊 比特币市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：24小时内

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

### 格式B：黄金市场报告

```
📊 黄金市场报告 | {YYYY}年{MM}月{DD}日 周{W}

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：24小时内

1️⃣ 当前价格与市场情绪

| 市场 | 价格 | 涨跌 | 来源 |
|------|------|------|------|
| 国际金价 (XAU/USD) | X,XXX 美元/盎司 | ±X.XX% 📈/📉 | 来源名 |
| 上海金基准价 | XXX 元/克 | — | 上海黄金交易所 |
| 黄金T+D | XXX 元/克 | ±X.XX% | 中国黄金集团 |

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

---

## 执行步骤

### 步骤 1：BTC 价格

使用 `mcp__web_reader__webReader` 依次尝试：

1. `https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_vol=true&include_24hr_change=true`
   → JSON：`bitcoin.usd`（价格）、`bitcoin.usd_24h_change`（涨跌%）、`bitcoin.usd_24h_vol`（成交量）
2. 若失败 → `https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT`
   → JSON：`lastPrice`、`priceChangePercent`、`quoteVolume`
3. 若失败 → `https://api.coinpaprika.com/v1/tickers/btc-bitcoin`
   → JSON：`quotes.USD.price`、`quotes.USD.percent_change_24h`、`quotes.USD.volume_24h`
4. 若失败 → `https://www.coingecko.com/en/coins/bitcoin`
5. 若失败 → `https://coinmarketcap.com/currencies/bitcoin/`

### 步骤 2：国际金价

`mcp__web_reader__webReader` → `https://tradingeconomics.com/commodity/gold` → 提取 XAU/USD 价格和涨跌幅

### 步骤 3：恐惧贪婪指数

`mcp__web_reader__webReader` → `https://api.alternative.me/fng/`
→ JSON：`data[0].value`（数值）、`data[0].value_classification`（分类）

### 步骤 4：国内金价

依次尝试以下数据源：

1. `mcp__web_reader__webReader` → `https://www.sge.com.cn/`
   → 上海黄金交易所，提取上海金早盘/午盘基准价（元/克）
2. `mcp__web_reader__webReader` → `https://www.chinagoldgroup.com/`
   → 中国黄金集团，提取黄金T+D价格（元/克）
3. 若 sge.com.cn 失败 → `mcp__web_reader__webReader` → `https://www.sge.com.cn/sjzx/mrhq/`
   → 上海金交所每日行情页面

记录：基准价、T+D价格及其来源。

### 步骤 5：新闻资讯（全部通过 webReader 抓取）

**以下 6 个网站依次用 `mcp__web_reader__webReader` 抓取，每个网站通常包含 10+ 条最新资讯：**

1. `mcp__web_reader__webReader` → `https://www.jin10.com`
   → 金十数据首页，财经快讯实时更新，每条都有精确时间戳（HH:MM）
2. `mcp__web_reader__webReader` → `https://www.cls.cn`
   → 财联社首页，中国最权威的财经快讯平台，每条都有精确时间戳
3. `mcp__web_reader__webReader` → `https://www.coindesk.com`
   → CoinDesk 首页，全球最大加密货币媒体
4. `mcp__web_reader__webReader` → `https://finance.sina.com.cn/nmetal/`
   → 新浪财经贵金属板块，黄金相关资讯
5. `mcp__web_reader__webReader` → `https://www.kitco.com/news/`
   → Kitco 黄金新闻，国际权威贵金属资讯
6. `mcp__web_reader__webReader` → `https://cointelegraph.com`
   → CoinTelegraph 首页，加密货币资讯

> 这 6 次抓取是获取资讯的唯一方式。不使用 WebSearch。每个网站首页都包含大量最新文章/快讯。

### 步骤 6：资讯提取与过滤

从步骤 5 的 6 个网站内容中提取资讯。

**提取每条资讯：**
- 标题（原文）
- 来源（网站名：金十数据、财联社、CoinDesk、新浪财经、Kitco、CoinTelegraph）
- 发布时间（优先使用带 HH:MM 的时间；如果只有日期，也可以保留）
- 链接（使用对应网站域名）

**过滤：**
- 发布时间超过 24 小时 → 丢弃
- 来源为"综合判断"等虚假来源 → 丢弃

**分类：**
- BTC 相关 vs 黄金相关（宏观资讯两个报告都能用）
- 利好 vs 利空

**去重：**同一事件只保留一条，按 金十数据 > 财联社 > CoinDesk > 其他 优先。

### 步骤 7：填充报告

将数据填入格式A（BTC报告）和格式B（黄金报告）。

**填入规则：**
- "来源"列必须是真实媒体名称
- 禁止"综合判断""综合分析""市场共识""综合研判""行业共识"
- 利好/利空因素每条注明来源
- 宏观背景表格每行填写真实来源

### 步骤 8：输出前扫描

**来源扫描**：搜索"综合判断""综合分析""市场共识""综合研判""行业共识" → 发现则删除该行或替换

**时间扫描**：资讯列表中仅有日期无时分的条目 → 删除

**价格来源扫描**：价格旁来源列为空 → 填入实际数据源名称

### 步骤 9：输出

输出两份完整报告（格式A + 格式B）。
