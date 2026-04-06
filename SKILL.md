---
name: crypto-gold-report
description: 生成黄金和比特币市场资讯报告。通过 OpenClaw cron 定时任务触发，使用 WebSearch 搜索资讯 + mcp__web_reader__webReader 提取详情。
---

# 加密货币与黄金市场报告

> **平台**: OpenClaw | **触发**: cron / 用户请求 | **工具**: `WebSearch` + `mcp__web_reader__webReader`

---

## 输出格式（最终报告必须符合此格式，不符合=报告作废）

报告分两种，**必须全部输出**，不可合并：

### 格式A：比特币市场报告

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

宏观背景：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
 美联储政策 | | |
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

报告生成时间：{YYYY}-{MM}-{DD} {HH}:{MM} | 数据时效：12小时内

1️⃣ 当前价格与市场情绪

| 市场 | 价格 | 涨跌 | 来源 |
|------|------|------|------|
| 国际金价 (XAU/USD) | X,XXX 美元/盎司 | ±X.XX% 📈/📉 | 来源名 |
| 国内足金 (周大福) | XXX 元/克 | ±XX元/克 | 来源名 |
| 中国黄金基础金价 | XXX 元/克 | ±XX元/克 | 来源名 |
| 投资金条 | XXX 元/克 | ±XX元/克 | 来源名 |

| 指标 | 数值 | 来源 |
|------|------|------|
| 恐惧贪婪指数 | XX（分类） | alternative.me |

价格更新时间：{YYYY}-{MM}-{DD} {HH}:{MM}

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

宏观背景：
| 因素 | 当前状况 | 来源 |
|------|----------|------|
 美联储政策 | | |
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

## 执行步骤（必须按顺序执行，每步完成前不得进入下一步）

### 步骤 1

使用 `mcp__web_reader__webReader` 依次抓取以下 URL，直到成功：

1. `https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_vol=true&include_24hr_change=true`
   - 解析 JSON：价格=`bitcoin.usd`，24h涨跌=`bitcoin.usd_24h_change`，成交量=`bitcoin.usd_24h_vol`
2. 若失败 → `https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT`
   - 解析 JSON：价格=`lastPrice`，涨跌=`priceChangePercent`，成交量=`quoteVolume`
3. 若失败 → `https://api.coinpaprika.com/v1/tickers/btc-bitcoin`
   - 解析 JSON：价格=`quotes.USD.price`，涨跌=`quotes.USD.percent_change_24h`，成交量=`quotes.USD.volume_24h`
4. 若失败 → `https://www.coingecko.com/en/coins/bitcoin`
   - 从 HTML 提取价格
5. 若失败 → `https://coinmarketcap.com/currencies/bitcoin/`
   - 从 HTML 提取价格
6. 若失败 → `WebSearch` 搜索 `bitcoin price USD today`，从摘要提取价格

**记录**：BTC价格、24h涨跌%、成交量、来源名称。全部失败则 BTC价格=NA（数据获取失败，已尝试6个数据源）。

### 步骤 2

使用 `mcp__web_reader__webReader` 抓取 `https://tradingeconomics.com/commodity/gold`，提取 XAU/USD 价格和涨跌幅。

若失败 → `WebSearch` 搜索 `XAU USD price today`。

记录：国际金价、涨跌%、来源名称。

### 步骤 3

使用 `mcp__web_reader__webReader` 抓取 `https://api.alternative.me/fng/`。
- 解析 JSON 中的 `data[0].value`（数值）和 `data[0].value_classification`（分类）
- 若失败 → `WebSearch` 搜索 `crypto fear and greed index today`

记录：指数数值、分类、来源=alternative.me。

### 步骤 4

使用 `mcp__web_reader__webReader` 抓取 `https://www.cngold.org/quote/golds/`。
提取：周大福足金价格、中国黄金基础金价、投资金条价格（单位：元/克）。

**如果 cngold.org 返回 404、空白、超时、或无有效数据，你不得直接写 N/A。你必须立即执行以下 3 个 WebSearch：**

第1个搜索：`WebSearch` 搜索 `周大福 今日金价 元/克`
第2个搜索：`WebSearch` 搜索 `中国黄金 基础金价 今日`
第3个搜索：`WebSearch` 搜索 `今日金店金价 元/克`

**3个搜索必须全部执行完毕后，才能记录国内金价结果。如果3个搜索都没有有效结果，才标注 N/A。**

记录：国内金价数据及其来源。

### 步骤 5（资讯搜索）

使用 `WebSearch` 依次执行以下搜索（共8组，每组必须执行）：

BTC相关：
1. `Bitcoin price news today {今天日期}`
2. `BTC ETF flows latest`
3. `比特币 价格 今日`
4. `BTC 行情 分析`

黄金相关：
5. `gold price today XAU USD {今天日期}`
6. `gold market news today`
7. `黄金价格 今日`
8. `金价 走势 最新`

### 步骤 6（资讯提取）

对步骤5的每条搜索结果，使用 `mcp__web_reader__webReader` 抓取详情页面。

对每条资讯提取：
- 标题（原文）
- 来源（真实媒体名称，如：金十数据、CoinDesk、财联社）
- 发布时间（必须是 YYYY-MM-DD HH:MM 格式，**仅有日期无时间的丢弃**）
- 链接

**时间过滤**：只保留报告生成时间前12小时内的资讯。

**去重**：同一事件只保留一条，按金十数据 > 新浪财经 > CoinDesk > 其他 的优先级保留。

### 步骤 7（补充搜索 — 必须执行满 3 轮）

数一数当前获取到的资讯数量：

- 利好资讯：___ 条
- 利空资讯：___ 条

**如果利好 ≥ 5 且利空 ≥ 5 → 跳到步骤 8**

**如果任一 < 5 → 必须执行以下 3 轮搜索，不可只执行 1 轮就放弃：**

**第 1 轮搜索（必须执行）：**
```
WebSearch: "Bitcoin rally" / "Bitcoin crash"
WebSearch: "gold surges" / "gold drops"
WebSearch: "比特币 突破" / "比特币 暴跌"
WebSearch: "黄金 突破" / "黄金 回调"
```
执行后回到步骤6提取详情。数一数：利好 ___ 条，利空 ___ 条。
如果 ≥ 5 → 跳到步骤 8。

**第 2 轮搜索（第1轮后仍不足5条时必须执行）：**
```
WebSearch: "site:jin10.com 比特币" / "site:jin10.com 黄金"
WebSearch: "site:cls.cn 黄金" / "site:cls.cn 比特币"
WebSearch: "比特币 利好 利空 今日"
WebSearch: "黄金 利好 利空 今日"
```
执行后回到步骤6提取详情。数一数：利好 ___ 条，利空 ___ 条。
如果 ≥ 5 → 跳到步骤 8。

**第 3 轮搜索（第2轮后仍不足5条时必须执行）：**
```
mcp__web_reader__webReader 抓取 https://www.jin10.com 首页获取最新快讯
mcp__web_reader__webReader 抓取 https://www.cls.cn 首页获取最新快讯
WebSearch: "加密货币 最新消息 今日"
WebSearch: "贵金属 最新消息 今日"
```
执行后回到步骤6提取详情。

**3 轮全部执行完后仍不足 → 在报告中标注实际数量。绝不编造。**

### 步骤 8（填充报告）

将步骤1-7获取的数据填入上方"格式A"（BTC报告）和"格式B"（黄金报告）模板。

填入时注意：
- 所有"来源"必须是真实媒体名称（禁止：综合判断、综合分析、市场共识、综合研判）
- 每条资讯的发布时间必须是 YYYY-MM-DD HH:MM 格式
- 利好/利空因素列表中的每条都要注明来源
- 宏观背景表格每行的来源列都要填写真实名称

### 步骤 9（来源合法性扫描）

在输出报告之前，对报告全文执行以下扫描：

搜索以下关键词，如果找到就将包含该关键词的行删除或替换：
- `综合判断` → 删除该行或替换来源
- `综合分析` → 同上
- `市场共识` → 同上
- `综合研判` → 同上
- `多方信息` → 同上
- `公开信息` → 同上
- `行业共识` → 同上

搜索以下关键词，如果找到则将"来源：XXX"替换为真实来源：
- `来源：`后面不是真实媒体名的

### 步骤 10（时间精度扫描）

对资讯列表中的每一条，检查发布时间列：
- 如果是 `YYYY-MM-DD` 格式（没有 HH:MM）→ **删除这条**
- 如果时间不完整 → **删除这条**

### 步骤 11（数量确认）

确认利好/利空资讯各有至少5条：
- 如果任一 < 5 条 → **返回步骤7继续补充搜索**，不得直接发布
- 如果任一不足5条但已执行3轮搜索 → 标注实际数量后发布

### 步骤 12（价格来源标注）

检查每条价格旁边的来源列：
- 如果是空的 → **填入实际使用的数据源名称**
- 如果是N/A → 确认已在步骤1-4中尝试了所有备用源

### 步骤 13（输出）

输出步骤8的两个完整报告（格式A和格式B）。在此之前的任何中间输出都不是正式报告。

---

## 数据源优先级

| 数据项 | 优先级1 | 优先级2 | 优先级3 |
|--------|---------|---------|---------|
| BTC价格 | CoinGecko API JSON | Binance API | CoinPaprika |
| BTC价格 | CoinGecko页面 | CoinMarketCap | WebSearch搜索 |
| 国际金价 | TradingEconomics | WebSearch | - |
| 国内金价 | cngold.org | WebSearch周大福 | WebSearch中国黄金 |
| 恐惧贪婪 | alternative.me API JSON | alternative.me页面 | WebSearch |