# iwencai-investment-engine 项目配置

## 项目定位

投资交易分析引擎：串联三类信息孤岛——持仓数据（每次对话手动输入）、同花顺市场数据（iwencai Skills）、个人投资知识（语雀 invest-kb），辅助做出更一致的投资决策。

## 分析模式路由

当用户发起以下类型请求时，使用 `investment-engine` Skill 响应：

| 模式 | 触发关键词 |
|------|-----------|
| 持仓复盘 | 复盘、持仓情况、仓位健康、看看我的仓、帮我看看持仓 |
| 买卖决策 | 买、卖、加仓、减仓、清仓、止损、要不要、该不该、能不能买、值不值得 |
| 宏观分析 | 宏观、政策、消息影响、利好/利空、大盘怎么看、这个消息 |
| 个股研究 | 研究、深挖、分析、生成研报、全面了解、帮我看看这只股 |

即使用户没有明说模式，只要涉及投资决策或标的分析，就应触发 investment-engine Skill。

## 语雀知识库配置

- **URL**：https://www.yuque.com/mocha-9hjjn/invest-kb
- **namespace**：`mocha-9hjjn/invest-kb`
- **归档目录**：`投资分析资产库/{模式子目录}`
  - 持仓复盘 → `投资分析资产库/持仓复盘/`
  - 买卖决策 → `投资分析资产库/买卖决策/`
  - 宏观分析 → `投资分析资产库/宏观分析/`
  - 个股研究 → `投资分析资产库/个股研究/`
- **时间规范**：所有时间统一使用北京时间（UTC+8）

## iwencai Skills 速查

> 完整索引见 `docs/iwencai-skills-index.md`

| 用途 | Skill |
|------|-------|
| 行情/涨跌幅/主力资金/技术指标 | `hithink-market-query` |
| 财务/营收/利润/ROE/估值PE/PB | `hithink-finance-query` |
| 基本资料/上市日期/主营业务 | `hithink-basicinfo-query` |
| 机构评级/EPS预测/目标价 | `hithink-insresearch-query` |
| 宏观数据/GDP/CPI/PMI/利率 | `hithink-macro-query` |
| 指数行情/上证/沪深300/恒生 | `hithink-zhishu-query` |
| 板块资金流向/行业估值分位 | `hithink-sector-selector` |
| A股多条件筛选 | `hithink-astock-selector` |
| 财经新闻/政策动态 | `news-search` |
| 公司公告/年报/回购/分红 | `announcement-search` |
| 投研机构深度研报 | `report-search` |
| ETF筛选 | `hithink-etf-selector` |
| 港股/美股筛选 | `hithink-hkstock-selector` / `hithink-usstock-selector` |
| 期货期权行情 | `hithink-futures-query` |

## 数据来源声明

所有市场数据分析必须标注：**数据来源：同花顺问财**
