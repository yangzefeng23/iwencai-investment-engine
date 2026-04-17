# iwencai Skills 索引表

**项目：** iwencai-investment-engine  
**路径：** `.claude/skills/`  
**数据来源：** 同花顺问财 OpenAPI  
**认证：** 环境变量 `IWENCAI_API_KEY`（Bearer Token）

---

## 一、数据查询类（Query）

接口：`POST https://openapi.iwencai.com/v1/query2data`

| Skill ID | 中文名 | 核心查询能力 | 典型查询示例 |
|----------|--------|------------|------------|
| `hithink-market-query` | 行情数据查询 | 实时价格、涨跌幅、成交量/额、换手率、**主力资金净流入/流出**、大小单、MACD/KDJ/RSI/布林线、ETF行情、指数行情 | `"贵州茅台最新价 涨跌幅 主力净流入"` |
| `hithink-finance-query` | 财务数据查询 | 营业收入、净利润、毛利率、净利率、**ROE/ROA**、资产负债率、经营性/投资性/自由现金流、PE/PB/PS | `"宁德时代近三年ROE 负债率 净利润增速"` |
| `hithink-basicinfo-query` | 基本资料查询 | 股票上市日期/所属行业/主营业务、基金管理人/费率、期货合约到期日/交割方式、可转债评级/正股、债券票面利率 | `"比亚迪上市日期 所属行业 主营业务"` |
| `hithink-insresearch-query` | 机构研究与评级查询 | 券商研报评级（买入/增持/中性/卖出）、**业绩预测/EPS预测**、ESG评级、信用/主体评级、基金星级、**券商金股** | `"腾讯最新研报评级 目标价 机构数量"` |
| `hithink-macro-query` | 宏观数据查询 | GDP、CPI、PPI、**M2、PMI、社融、LPR**、汇率、工业增加值、固定资产投资、进出口 | `"2024年CPI同比 PPI同比数据"` |
| `hithink-zhishu-query` | 指数数据查询 | 上证/深证/创业板/沪深300/中证500/上证50/恒生/纳斯达克/道琼斯的点位、涨跌幅、成交量 | `"沪深300近一个月涨跌幅"` |
| `hithink-futures-query` | 期货期权数据查询 | 期货行情、**隐含/历史波动率**、库存/产量/销量、会员持仓排行、成交量排行、行权价/行权量 | `"沪铜期货最新行情 持仓量"` |

---

## 二、智能筛选类（Selector）

接口：`POST https://openapi.iwencai.com/v1/query2data`  
特性：空数据时自动放宽条件重试，最多 2 次

| Skill ID | 中文名 | 核心筛选维度 | 典型查询示例 |
|----------|--------|------------|------------|
| `hithink-astock-selector` | 问财选A股 | 行情指标（价格/涨跌幅/成交量）、**技术形态**（均线多头/突破新高/K线形态）、财务指标（PE/PB/利润）、行业概念 | `"今日涨跌幅超过5%的A股"` |
| `hithink-etf-selector` | 问财选ETF | 跟踪指数（沪深300/中证500等）、**资产规模/份额变化**、费率/跟踪误差、风格类型（成长/价值） | `"规模最大的沪深300ETF"` |
| `hithink-sector-selector` | 问财选板块 | **行业估值分位（PE/PB）**、主力资金净流入/北向资金、涨跌幅排名、板块类型（行业/概念/地域） | `"近一周主力净流入前五的行业板块"` |
| `hithink-hkstock-selector` | 问财选港股 | 行情、财务（PE/PB/营收/利润）、行业概念、**陆港通持股**（北向/南向） | `"港股科技股PE低于20的有哪些"` |
| `hithink-usstock-selector` | 问财选美股 | 行情、财务、行业概念、**业绩预测**（盈利增速）、研报评级（买入/增持/中性） | `"评级买入的美股科技公司"` |

---

## 三、资讯搜索类（Search）

接口：`POST https://openapi.iwencai.com/v1/comprehensive/search`  
固定参数：`app_id: "AIME_SKILL"`

| Skill ID | 中文名 | `channels` 值 | 数据来源与能力 | 典型查询示例 |
|----------|--------|--------------|--------------|------------|
| `news-search` | 新闻搜索 | `["news"]` | 官媒、主流财经媒体（证券时报/经济观察报等）、垂直行业网站、上市公司官网；覆盖**政策动态/行业趋势/企业进展** | `"宁德时代 最新动态"` |
| `announcement-search` | 公告搜索 | `["announcement"]` | A股/港股/基金/ETF公告；涵盖**定期报告、分红派息、回购增持、资产重组、业绩预告** | `"贵州茅台 最近公告"` |
| `report-search` | 研报搜索 | `["report"]` | 主流投研机构（券商/基金/独立研究机构）的深度研究报告；聚焦**分析逻辑/投资评级/目标价/风险提示** | `"比亚迪 深度研究报告"` |

---

## 四、分析场景 → Skill 速查

```
持仓复盘（每只标的）
  行情健康度    → hithink-market-query（价格、涨跌幅、主力资金）
  财务健康度    → hithink-finance-query（ROE、负债率、净利润同比）
  机构态度      → hithink-insresearch-query（评级、目标价）

买卖决策
  价格位置      → hithink-market-query（52周高低、近期涨跌）
  估值安全边际  → hithink-finance-query（PE/PB/PEG/自由现金流）
  财务质量趋势  → hithink-finance-query（收入/利润增速、ROE）
  机构共识      → hithink-insresearch-query（评级分布、EPS预测）
  近期公告      → announcement-search
  近期新闻      → news-search
  深度研报      → report-search（可选）

宏观分析
  宏观指标序列  → hithink-macro-query
  指数反应      → hithink-zhishu-query
  行业资金/估值 → hithink-sector-selector
  机构策略观点  → report-search
  事件新闻      → news-search

个股研究（全维度）
  基础信息      → hithink-basicinfo-query
  行情全貌      → hithink-market-query
  财务趋势      → hithink-finance-query × 2（利润表 + 资产负债表）
  机构评级预测  → hithink-insresearch-query
  近期公告      → announcement-search
  深度研报      → report-search
  近期新闻      → news-search
  行业背景      → hithink-sector-selector
```
