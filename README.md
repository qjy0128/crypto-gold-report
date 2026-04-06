# crypto-gold-report

每日加密货币与黄金市场资讯报告生成工具，基于 OpenClaw 平台运行。

## 功能

- 自动生成比特币（BTC）市场报告和黄金（XAU）市场报告
- **纯 webReader 枋取**：不依赖 WebSearch（该环境无 API Key），所有数据通过 `mcp__web_reader__webReader` 直接抓取
- 九步执行流程：价格获取 → 新闻网站抓取 → 资讯过滤 → 抋告生成 → 输出扫描
- 新闻来源：直接抓取 jin10.com、cls.cn、coindesk.com、kitco.com 等 6 个网站首页
- 自动去重（同一事件/相同链接/高度相似标题），按来源权威性排序
- 集成恐惧贪婪指数（Fear & Greed Index）
- 输出前自动扫描：来源合法性、时间精度、价格来源

## 核心规则

| 规则 | 说明 |
|------|------|
| 价格真实性 | 所有价格从 API/网页实时抓取，失败显示 N/A，绝不编造 |
| 资讯真实性 | 所有资讯来自实际抓取结果，禁止编造标题/来源/时间 |
| 来源真实性 | 禁止"综合判断""综合分析""市场共识"等非真实来源 |
| 备用源强制 | 主源失败必须尝试所有备用源，全部失败才标注 N/A |
| 时效性 | 仅保留 24 小时内资讯 |
| 去重 | 同一事件仅保留一条，按权威性优先 |

## 数据源

| 数据项 | 来源 |
|--------|------|
| BTC价格 | CoinGecko API → Binance API → CoinPaprika API → CoinGecko页面 → CoinMarketCap |
| 国际金价 | TradingEconomics |
| 国内金价 | 上海黄金交易所 (sge.com.cn) → 中国黄金集团 (chinagoldgroup.com) |
| 恐惧贪婪指数 | alternative.me API |
| 新闻资讯 | jin10.com → cls.cn → coindesk.com → sina财经 → kitco.com → cointelegraph.com |

## 文件结构

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 技能定义文件（触发条件、执行流程、报告格式、输出扫描） |
| `CHANGELOG.md` | 变更日志 |
| `README.md` | 本文件 |

## 风险提示
本工具生成的报告仅供参考，不构成任何投资建议。
