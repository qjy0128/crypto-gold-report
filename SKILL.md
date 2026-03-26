---
name: crypto-gold-report
description: 生成黄金和比特币市场资讯晨间报告。当用户需要金融资讯、行情报告、市场分析时使用此技能。
---

# 加密货币与黄金市场报告

此技能用于生成每日的黄金和比特币市场资讯报告。

## 触发条件

- 用户请求黄金资讯/报告
- 用户请求比特币/BTC资讯/报告
- 用户请求市场晨间报告
- 用户请求行情分析

## 资讯获取方式

### 1. 比特币资讯搜索

使用 `batch_web_search` 搜索以下关键词：
- "Bitcoin price news March 2026"
- "BTC ETF inflows 2026"
- "Bitcoin price forecast March 2026"

使用 `extract_content_from_websites` 提取详细资讯：
- https://coinfomania.com/bitcoin-etf-inflows-march-2026/
- https://cryptonews.com/news/bitcoin-price-prediction-high-stakes-march-2026-120k-forecasts/
- https://tradingeconomics.com/commodity/gold

### 2. 黄金资讯搜索

使用 `batch_web_search` 搜索以下关键词：
- "gold price March 2026 XAU USD"
- "黄金价格走势 2026年3月"
- "gold ETF inflows March 2026"

使用 `extract_content_from_websites` 提取详细资讯：
- https://tradingeconomics.com/commodity/gold
- https://forex24.pro/gold-price-forecast/gold-forecast-and-xau-usd-analysis-for-march-6-2026/
- https://www.cngold.org/quote/
- https://cngoldprice.com/

## 报告格式

### 比特币报告格式

```
📊 比特币晨间报告 | YYYY年MM月DD日 周X

---

## 1️⃣ 当前价格
**BTC/USD**: $XX,XXX ~ $XX,XXX
**24小时涨跌幅**: ±X.XX% 📈/📉

---

## 2️⃣ 利好资讯 (10条)

1. **标题** - 描述
   - 来源: XXX | 链接

(共10条)

---

## 3️⃣ 利空资讯 (10条)

1. **标题** - 描述
   - 来源: XXX | 链接

(共10条)

---

## 4️⃣ 24小时趋势分析

**主要影响因素：**
- ✅ 因素1
- ✅ 因素2
- ❌ 因素3
- ❌ 因素4

**综合判断**：XXX

---

## 5️⃣ 趋势预测

| 周期 | 涨跌概率 | 关键价位 |
|------|----------|----------|
| **24小时** | 🔴 XX% 利空 / 🟢 XX% 利多 | 阻力$XX,XXX / 支撑$XX,XXX |
| **3天** | 🔴 XX% 利空 / 🟢 XX% 利多 | 阻力$XX,XXX / 支撑$XX,XXX |
| **7天** | 🔴 XX% 利空 / 🟢 XX% 利多 | 波动区间$XX,XXX-$XX,XXX |
| **1个月** | 🔴 XX% 利空 / 🟢 XX% 利多 | 长期底部确认中 |

⚠️ **风险提示**：本报告仅供参考，不构成投资建议。
```

### 黄金报告格式

```
📊 黄金市场报告 (YYYY年MM月DD日)

---

💰 当前价格

| 市场 | 价格 | 相比昨日 |
|------|------|----------|
| 国际金价 (伦敦金) | X,XXX 美元/盎司 | ±X.XX% ↓/↑ |
| 国内足金 (周大福) | X,XXX 元/克 | ±XX元/克 ↓/↑ |
| 中国黄金基础金价 | X,XXX 元/克 | ±XX元/克 ↓/↑ |
| 投资金条 | X,XXX 元/克 | ±XX元/克 ↓/↑ |

---

✅ 利好资讯 (10条)

| # | 标题 | 发布时间 | 来源 | 链接 |
|---|------|----------|------|------|
| 1 | XXX | YYYY-MM-DD | XXX | 链接 |

(共10条)

---

⚠️ 利空资讯 (10条)

| # | 标题 | 发布时间 | 来源 | 链接 |
|---|------|----------|------|------|
| 1 | XXX | YYYY-MM-DD | XXX | 链接 |

(共10条)

---

📈 趋势分析

最近24小时走势原因：
1. 原因1
2. 原因2
3. 原因3
4. 原因4

---

🔮 趋势预测

| 时间范围 | 趋势判断 | 概率 |
|----------|----------|------|
| 未来24小时 | 🔴/🟡/🟢 XX% XXX | |
| 未来3天 | 🔴/🟡/🟢 XX% XXX | |
| 未来7天 | 🔴/🟡/🟢 XX% XXX | |
| 未来1个月 | 🔴/🟡/🟢 XX% XXX | |

---

⚠️ 风险提示：本报告仅供参考，不构成投资建议。
```

## 输出要求

1. 严格按照上述格式输出报告
2. 资讯必须包含具体链接和来源
3. 价格数据必须包含具体数字和涨跌幅
4. 趋势预测必须有概率和关键价位
5. 利好/利空各至少10条
6. 黄金报告使用表格格式，比特币报告使用列表格式
