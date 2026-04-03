# Crypto Gold Report Skill

OpenClaw 定时任务技能，自动生成黄金和比特币市场每日晨间报告。

## 功能

- **比特币晨间报告**: 当前价格、24h涨跌幅、恐惧贪婪指数、利好/利空资讯（各5条）、趋势分析与预测
- **黄金市场报告**: 国内外金价、恐惧贪婪指数、利好/利空资讯、趋势分析与预测

## 运行方式

通过 **OpenClaw cron 定时任务** 触发，非 Claude Code 直接调用。

## 核心工具

| 工具 | 用途 |
|------|------|
| `WebSearch` | 搜索资讯和行情数据 |
| `mcp__web_reader__webReader` | 提取网页详细内容 |

## 资讯源

| 来源 | 覆盖 | 语言 |
|------|------|------|
| 金十数据 (jin10.com) | BTC + 黄金 | 中文 |
| 新浪财经 (finance.sina.com.cn) | BTC + 黄金 | 中文 |
| CoinDesk (coindesk.com) | BTC | 英文 |
| CoinTelegraph (cointelegraph.com) | BTC | 英文 |
| Kitco (kitco.com) | 黄金 | 英文 |
| TradingEconomics (tradingeconomics.com) | 黄金 | 英文 |
| Alternative.me | 恐惧贪婪指数 | 英文 |

## 关键规则

- **时间窗口**: 仅保留12小时内发布的资讯，超时直接丢弃
- **去重**: 同一事件按来源权威性优先保留一条
- **趋势依据**: 必须引用具体资讯来源，不得凭空推测
- **降级策略**: 资讯不足时逐轮扩大搜索范围，不放宽时间窗口

## 文件说明

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 技能定义文件（触发条件、搜索流程、报告格式、输出规则） |
| `CHANGELOG.md` | 变更日志 |
| `README.md` | 本文件 |
