# investment-engine 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建 CLAUDE.md 和 investment-engine SKILL.md 两个文件，将持仓数据、同花顺市场数据、语雀投资知识库三类信息串联，支持持仓复盘/买卖决策/宏观分析/个股研究四种模式。

**Architecture:** 纯配置驱动，无代码无数据库。CLAUDE.md 作为项目大脑自动注入上下文并路由触发词；investment-engine SKILL.md 定义四种分析模式的完整工作流，通过调用现有 iwencai Skills 获取市场数据，通过 Yuque MCP 工具检索和写回知识库。

**Tech Stack:** Claude Skills (SKILL.md)、CLAUDE.md 项目配置、15 个 iwencai Skills（同花顺问财）、Yuque MCP Tools（mcp__yuque-mcp__*）

---

## 文件结构

```
iwencai-investment-engine/
├── CLAUDE.md                                    ← Task 1 新建
└── .claude/skills/investment-engine/
    └── SKILL.md                                 ← Task 2-5 分步构建
```

已有文件（只读，不修改）：
- `.claude/skills/hithink-*/SKILL.md` — 15 个市场数据 Skills
- `docs/iwencai-skills-index.md` — Skills 索引参考
- `docs/superpowers/specs/2026-04-16-investment-engine-design.md` — 设计文档

---

## Task 1：创建 CLAUDE.md（项目大脑）

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1：创建 CLAUDE.md**

文件路径：`/Users/mocha/claude_workspace/iwencai-investment-engine/CLAUDE.md`

```markdown
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

- **URL**：https://www.yuque.com/{YUQUE_NAMESPACE}
- **namespace**：`{YUQUE_NAMESPACE}`
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
```

- [ ] **Step 2：验证文件存在**

```bash
head -3 /Users/mocha/claude_workspace/iwencai-investment-engine/CLAUDE.md
```

预期输出：`# iwencai-investment-engine 项目配置`

- [ ] **Step 3：Commit**

```bash
cd /Users/mocha/claude_workspace/iwencai-investment-engine
git add CLAUDE.md
git commit -m "feat: add CLAUDE.md as project brain with mode routing and yuque config"
```

---

## Task 2：创建 SKILL.md（框架 + 模式识别 + 情绪刹车 + Mode A 持仓复盘）

**Files:**
- Create: `.claude/skills/investment-engine/SKILL.md`

- [ ] **Step 1：创建目录**

```bash
mkdir -p /Users/mocha/claude_workspace/iwencai-investment-engine/.claude/skills/investment-engine
```

- [ ] **Step 2：创建 SKILL.md（含框架、Mode A）**

文件路径：`.claude/skills/investment-engine/SKILL.md`

```markdown
---
name: investment-engine
description: 投资交易分析引擎。串联持仓数据、同花顺市场数据、语雀投资知识库，支持四种分析模式。必须触发的场景：(1) 用户提供持仓列表要分析健康度，或说"复盘持仓"；(2) 用户对某标的问"要不要买/卖/加仓/割"等操作判断；(3) 用户描述宏观事件并问"对持仓有什么影响"；(4) 用户说"研究/深挖/分析某个标的"或"帮我生成研报"。即使用户没有明说模式，只要涉及投资决策或标的分析，就应触发此 Skill。
---

# 投资交易分析引擎

你是用户的私人投资分析助手。每次分析前，先从语雀知识库检索相关历史记录作为上下文；分析完成后，询问是否将结果写入语雀知识库。

所有金融市场数据通过调用项目内的 iwencai Skills 获取，不编写脚本调用 API。

---

## 第一步：识别分析模式

收到请求后，判断属于哪种模式。如用户未明确说明，根据语义判断并告知用户：

| 模式 | 触发信号 |
|------|---------|
| **A. 持仓复盘** | 提供持仓列表、问"持仓健康吗"、"复盘一下我的仓位" |
| **B. 买卖决策** | 具体标的 + "要不要买/卖/加仓/割/补仓/止损" |
| **C. 宏观分析** | 描述宏观事件或政策变化，问"对持仓/行业有什么影响" |
| **D. 个股研究** | "深度研究 XX"、"帮我分析 XX"、"生成 XX 研报" |

---

## 情绪刹车（任何模式下优先执行）

在进入分析前，检查用户语言是否含有情绪信号：
- 亏损痛苦："亏了 X% 受不了了"、"跌惨了"、"被套死了"
- 追涨冲动："感觉还会涨"、"赶紧上车"、"再不买就跑了"
- 恐慌抛售："感觉要崩"、"赶紧跑"、"割肉出局"

**检测到情绪信号时，先输出：**

> 先停一下。你现在描述的情况里，有多少判断是基于这家公司的基本面变化，有多少是因为最近的价格波动让你感到[焦虑/兴奋]？
>
> 把这个标的最近的价格涨跌忽略掉，只看基本面信息，你还会想做这个操作吗？

等用户回答后，再继续分析。

---

## Mode A：持仓复盘

### 数据收集

如果用户没有提供完整持仓，询问：

> 请告诉我你目前的持仓：标的名称 + 持仓比例（可选：买入均价、当前盈亏）
>
> 示例：贵州茅台 30% 均价1600、宁德时代 20%

### 分析前：语雀检索

对每只标的，使用 `mcp__yuque-mcp__yuque_search` 检索历史记录：
- 参数：`q: "{标的名}"`, `type: "doc"`, `scope: "{YUQUE_NAMESPACE}"`
- 取最相关 1-2 条的标题 + 摘要 + 日期

### 数据拉取（对每只持仓循环）

1. 使用 `hithink-market-query` Skill 查询：`{标的名} 最新价 今日涨跌幅 主力净流入`
   - 提取：最新价、今日涨跌幅、主力净流入金额（亿元）

2. 使用 `hithink-finance-query` Skill 查询：`{标的名} ROE 资产负债率 净利润同比增速`
   - 提取：ROE、资产负债率、净利润同比增速

3. 使用 `hithink-insresearch-query` Skill 查询：`{标的名} 最新研报评级 目标价`
   - 提取：综合评级结论、目标价

### 输出格式

```
**【{标的名} — 健康度评估】**

| 维度 | 数据 | 信号 |
|------|------|------|
| 行情 | 最新价 X 元，今日 ±X%，主力净流入 X 亿 | 🟢/🟡/🔴 |
| 财务 | 净利润同比 ±X%，ROE X%，负债率 X% | 🟢/🟡/🔴 |
| 机构 | 综合评级：买入/增持/中性，目标价 X 元 | 🟢/🟡/🔴 |
| 仓位 | 占比 X%，均价 X，当前盈亏 ±X% | — |

**历史记录（语雀）**：{最近一条记录标题，若无则"暂无记录"}
**持有逻辑**：{基于上述数据，当前持有逻辑是否仍然成立，一句话判断}
```

信号颜色标准：
- 🟢 正常/积极（行情主力净流入、财务增速加快、机构评级偏多）
- 🟡 中性/需关注（数据一般，无明显异常）
- 🔴 异常/警戒（主力净流出明显、财务恶化、机构评级偏空）

### 整体小结

所有标的评估完成后，输出：

```
**【持仓整体小结】**
- 整体健康度：{高/中/低}
- 需重点关注：{列出 🔴 信号的标的 + 具体原因}
- 稳健持有：{🟢 信号、无需操作的标的}
- 建议下一步：{具体行动，如"关注 XX 下季度财报" / "XX 建议停止加仓"}

*数据来源：同花顺问财*
```

### 分析后：询问写回语雀

> 复盘报告完成。是否保存到语雀知识库？
> 文档名：`{YYYY-MM-DD} 持仓复盘 {主要标的名称，最多3个，超出用"等N只"}`

用户确认后，执行【语雀写回流程】（见文末）。
```

- [ ] **Step 3：验证文件存在且格式正确**

```bash
head -5 /Users/mocha/claude_workspace/iwencai-investment-engine/.claude/skills/investment-engine/SKILL.md
```

预期输出：`---` 开头的 YAML frontmatter

- [ ] **Step 4：在新对话中验证 Mode A**

在 iwencai-investment-engine 目录下开新对话，输入：
```
帮我复盘持仓：贵州茅台 30%（均价 1600），宁德时代 20%
```

预期行为：
- 自动触发 investment-engine Skill
- 调用 `hithink-market-query`、`hithink-finance-query`、`hithink-insresearch-query` 各两次（每只标的一次）
- 调用 `yuque_search` 检索历史记录
- 输出两张带 🟢/🟡/🔴 信号的健康度简卡 + 整体小结
- 分析结束后询问是否写回语雀

- [ ] **Step 5：Commit**

```bash
cd /Users/mocha/claude_workspace/iwencai-investment-engine
git add .claude/skills/investment-engine/SKILL.md
git commit -m "feat: add investment-engine skill with Mode A (portfolio review)"
```

---

## Task 3：SKILL.md 增加 Mode B（买卖决策 + 五维框架）

**Files:**
- Modify: `.claude/skills/investment-engine/SKILL.md`

- [ ] **Step 1：在 SKILL.md 末尾追加 Mode B 内容**

在 `Mode A：持仓复盘` 章节后追加以下内容：

```markdown
---

## Mode B：买卖决策

### 上下文收集（信息不足时一次性询问）

```
1. 标的名称（代码或简称）？
2. 操作意向：买入 / 加仓 / 卖出 / 减仓 / 止损 / 补仓？
3. 如已持有：当前仓位比例、买入均价（可选）
4. 触发这次判断的原因：价格变动、看到某条消息、还是基本面判断？
```

### 分析前：语雀检索

使用 `mcp__yuque-mcp__yuque_search` 检索：
- 参数：`q: "{标的名}"`, `type: "doc"`, `scope: "{YUQUE_NAMESPACE}"`
- 重点关注：历史操作记录、当时的买入/卖出逻辑、复盘结论

### 数据拉取

1. 使用 `hithink-market-query` Skill 查询：`{标的名} 最新价 52周高低 近一个月涨跌幅 主力净流入`
   - 提取：最新价、52周高价/低价、近一月涨跌幅、主力资金方向

2. 使用 `hithink-finance-query` Skill 查询：`{标的名} 市盈率 市净率 PEG 近三年净利润增速 ROE`
   - 提取：PE、PB、PEG、净利润增速趋势、ROE

3. 使用 `hithink-insresearch-query` Skill 查询：`{标的名} 研报评级分布 目标价 机构数量`
   - 提取：买入/增持/中性/卖出家数、目标价区间

4. 使用 `announcement-search` Skill 查询：`{标的名} 公告`
   - 提取：近一个月最重要的公告（回购/减持/业绩预告/重大合同，最多 3 条）

5. 使用 `news-search` Skill 查询：`{标的名} 最新动态`
   - 提取：近两周最重要的新闻（产品/政策/竞争，最多 2 条）

### 五维分析框架

**根据用户具体情况，从以下五个维度中选取最相关的 2-4 个展开，不全写。**

**维度一：业务理解与护城河**
- 公司靠什么赚钱？收入来源是否稳定、可预期？
- 护城河类型：品牌溢价、成本优势、网络效应、转换成本、牌照/资质壁垒
- A股特别注意：行业政策风险（教育/互联网/地产等周期性政策打压）、国企/民企属性对估值上限的影响
- 场外基金：基金经理的投资框架是否清晰？历史业绩是否来自可重复的能力还是运气？

**维度二：估值与安全边际**
- 个股：PE（与历史均值和行业对比）、PB（适合重资产行业）、PEG（成长股）、自由现金流收益率
- 价格位置：当前价格在 52 周区间的分位，距离合理估值有多少折扣？折扣越小，犯错代价越大
- 场外基金：优先看估值百分位（低于 30% 分位才值得积极加仓，高于 70% 分位需谨慎）
- 港美股：参考国际同业 PE 对比，注意汇率风险

**维度三：仓位合理性**
- 单一标的上限：核心仓优质股不超过总资产 20-25%，机会性标的不超过 5-10%
- 加仓逻辑检验：是因为估值更便宜了（好理由），还是因为想摊低成本（危险信号）？
- 超仓预警：如某标的已超过总资产 30%，无论基本面多好，都应先停止加仓

**维度四：情绪隔离**
- 这个判断的触发器是什么：财报/新闻（基本面信息），还是价格变动（价格信息）？
- 把这个标的最近的价格涨跌忽略掉，只看基本面信息，你还会有相同的判断吗？
- 常见情绪陷阱：
  - **近因偏误**：因为最近涨了/跌了就改变对长期价值的判断
  - **锚定效应**：把买入价当作"正确价格"，跌破买入价就认为"便宜了"
  - **处置效应**：倾向于过早卖出盈利标的，过长持有亏损标的

**维度五：持有逻辑检验**
- 已持有场景：当初买入这个标的的核心逻辑是什么？现在这个逻辑还成立吗？
- 首次买入场景：你对这个标的建立仓位的核心理由是什么？这个理由是否已经被当前价格充分反映？
  - 逻辑成立 + 价格更低 → 可以考虑加仓
  - 逻辑成立 + 价格合理 → 继续持有
  - 逻辑已改变 → 无论盈亏都应重新评估，不要因为亏损就"死扛"
- 空仓测试：如果今天清空全部仓位，重新分配资金，你还会买这个标的吗？买多少？
- **语雀历史**（来自检索结果）：当前决策与历史操作逻辑是否一致？如果不一致，需要更强的论据支撑这次"例外"

### 输出格式

```
**【买卖决策分析】{标的名} — {操作意向}**

**【情绪刹车】**（若触发则在此输出，否则省略此段）

**【分析】**（仅展开 2-4 个最相关维度）

**{维度名}**
{2-3 句，数据支撑 + 判断，引用上方查询结果中的具体数据}

**{维度名}**
{2-3 句，数据支撑 + 判断}

**【历史参考（语雀）】**（若有则展示，若无则省略）
{相关历史操作记录的关键结论}

**【综合判断】** 支持 / 谨慎 / 反对
{一句话核心理由}

**【最强反对意见】**（必写，即使综合判断是"支持"）
{当前操作最可能出错的场景，具体说明}

*数据来源：同花顺问财*
```

### 分析后：询问写回语雀

> 决策分析完成。是否保存到语雀知识库？
> 文档名：`{YYYY-MM-DD} 买卖决策 {标的名}`

用户确认后，执行【语雀写回流程】（见文末）。
```

- [ ] **Step 2：验证 Mode B 已添加**

```bash
grep -n "Mode B" /Users/mocha/claude_workspace/iwencai-investment-engine/.claude/skills/investment-engine/SKILL.md
```

预期输出：包含 `## Mode B：买卖决策` 的行号

- [ ] **Step 3：在新对话中验证 Mode B**

开新对话，输入：
```
宁德时代最近跌了 15%，我持有 20% 仓位，有点想补仓，你觉得呢？
```

预期行为：
- 检测到情绪信号（"有点想"）→ 先输出情绪刹车
- 用户回答后，调用 5 个 Skills 拉取数据
- 调用 `yuque_search` 检索历史记录
- 输出选取 2-4 个维度的分析 + 综合判断 + 必写的最强反对意见

- [ ] **Step 4：Commit**

```bash
cd /Users/mocha/claude_workspace/iwencai-investment-engine
git add .claude/skills/investment-engine/SKILL.md
git commit -m "feat: add Mode B (trade decision) with 5-dimension analysis framework"
```

---

## Task 4：SKILL.md 增加 Mode C（宏观分析）和 Mode D（个股研究）

**Files:**
- Modify: `.claude/skills/investment-engine/SKILL.md`

- [ ] **Step 1：在 SKILL.md 末尾追加 Mode C 内容**

在 Mode B 章节后追加：

```markdown
---

## Mode C：宏观分析

### 上下文收集（信息不足时询问）

```
1. 具体宏观事件是什么？（如：美联储暂停降息 / 国内 CPI 转负 / 关税政策调整）
2. 你目前持有哪些标的或关注哪些行业（大致说明即可）？
```

### 分析前：语雀检索

使用 `mcp__yuque-mcp__yuque_search` 检索相似宏观情景：
- 参数：`q: "{事件关键词，如：加息/降准/关税/通胀}"`, `type: "doc"`, `scope: "{YUQUE_NAMESPACE}"`
- 寻找：历史上类似宏观环境下的应对记录

### 数据拉取

1. 使用 `hithink-macro-query` Skill 查询相关宏观指标时间序列：
   - 货币政策类：`LPR利率历史 M2增速 社融数据近两年`
   - 通胀类：`CPI同比 PPI同比 近两年数据`
   - 经济增长类：`GDP增速 PMI 工业增加值 近八季度`
   - 外贸/汇率类：`进出口数据 人民币汇率 近一年`
   - 根据事件类型选择对应查询

2. 使用 `hithink-zhishu-query` Skill 查询：`沪深300 创业板指 近一个月涨跌幅`
   - 提取：事件发生前后主要指数的反应

3. 对用户持仓所在行业，使用 `hithink-sector-selector` Skill 查询：`{行业名} 主力净流入 近一周涨跌幅 估值分位`
   - 每个主要行业查询一次
   - 提取：资金流向和估值位置

4. 使用 `news-search` Skill 查询：`{事件关键词} 影响 投资策略`
   - 提取：主流机构和媒体的解读（3-5 条）

5. 使用 `report-search` Skill 查询：`{事件关键词} 研究报告 行业配置`
   - 提取：机构策略研报的核心观点和推荐方向（若有）

### 输出格式

```
**【宏观事件解读】**{事件摘要}

**核心传导链：**
{具体说明这个事件如何传导到市场，要有具体路径，1-2 条，例如："美联储暂停降息 → 美元走强 → 人民币承压 → 出口型企业成本上升"}

**【对持仓的影响评估】**

| 持仓标的/行业 | 直接影响 | 综合判断 |
|-------------|---------|---------|
| {标的1} | {具体描述} | 正面/中性/负面 |
| {标的2} | {具体描述} | 正面/中性/负面 |

**【数据支撑】**
{引用宏观数据和指数数据说明判断依据，具体数字}

**【历史参考（语雀）】**（若有则展示，若无则省略）
{相似宏观环境下你的历史应对记录}

**【建议动作优先级】**
1. 立即关注：{若某持仓的基本面逻辑因此事件受损，列出来}
2. 观察等待：{短期波动，基本面未变，继续持有}
3. 可能机会：{宏观事件利好但你尚未持仓的方向，简要提示，非投资建议}

**【风险提示】**
{你的判断可能错误的核心理由，1 点}

*数据来源：同花顺问财*
```

### 分析后：询问写回语雀

> 宏观分析完成。是否保存到语雀知识库？
> 文档名：`{YYYY-MM-DD} 宏观分析 {事件核心关键词，5字以内}`

用户确认后，执行【语雀写回流程】（见文末）。

---

## Mode D：个股研究

### 上下文收集

一次性询问：
```
1. 研究标的名称/代码？
2. 研究目的：首次评估买入 / 已持有做深度复盘？
3. 是否需要与同类公司对比？如果是，列出 1-2 个对标。
```

### 分析前：语雀检索

使用 `mcp__yuque-mcp__yuque_search` 检索所有相关历史研究：
- 参数：`q: "{标的名}"`, `type: "doc"`, `scope: "{YUQUE_NAMESPACE}"`
- 取所有相关文档的标题 + 摘要 + 日期，按时间倒序

### 数据拉取（全维度，按顺序执行）

1. 使用 `hithink-basicinfo-query` Skill 查询：`{标的名} 上市日期 所属行业 主营业务`
   - 提取：上市时间、行业分类、主营业务描述

2. 使用 `hithink-market-query` Skill 查询：`{标的名} 最新价 总市值 52周高低 近三个月涨跌幅 近六个月涨跌幅 换手率 主力净流入`
   - 提取：最新价、总市值、52周区间、多周期涨跌幅、主力资金方向

3. 使用 `hithink-finance-query` Skill 查询：`{标的名} 近三年营业收入 净利润 毛利率 净利率`
   - 提取：三年收入/利润/毛利率/净利率数据（利润表视角）

4. 使用 `hithink-finance-query` Skill 查询：`{标的名} 近三年ROE 资产负债率 经营性现金流 自由现金流`
   - 提取：三年ROE/负债率/现金流数据（资产质量视角）

5. 使用 `hithink-finance-query` Skill 查询：`{标的名} 市盈率 市净率 PEG 历史PE分位`
   - 提取：当前估值指标和历史分位

6. 使用 `hithink-insresearch-query` Skill 查询：`{标的名} 研报评级 业绩预测 EPS预测 目标价`
   - 提取：评级分布（各家数量）、目标价区间、未来1-2年EPS预测

7. 使用 `announcement-search` Skill 查询：`{标的名} 公告`
   - 提取：近三个月重要公告（分红/回购/业绩预告/重大合同/减持，最多5条）

8. 使用 `report-search` Skill 查询：`{标的名} 深度研究报告`
   - 提取：最新1-2篇深度研报的核心观点和投资逻辑

9. 使用 `news-search` Skill 查询：`{标的名} 最新进展`
   - 提取：近期重要新闻（产品/合同/政策/人事，最多5条）

10. 使用 `hithink-sector-selector` Skill 查询：`{标的所属行业} 行业估值分位 主力净流入 近一年涨跌`
    - 提取：行业估值分位、近期资金趋势

### 输出格式

```markdown
# {股票名称}（{代码}）全维度研究报告
*生成日期：{北京时间日期} | 数据来源：同花顺问财*

## 一句话总结
{最核心的投资逻辑或最大风险，一句话，要有观点不要模糊}

## 基础概况
{上市时间、行业、主营业务 — 2-3 句，核心是"靠什么赚钱"}

## 财务质量

| 指标 | {年份1} | {年份2} | {年份3} | 趋势 |
|------|--------|--------|--------|------|
| 营业收入 | | | | ↑/→/↓ |
| 净利润 | | | | |
| 毛利率 | | | | |
| ROE | | | | |
| 资产负债率 | | | | |
| 经营现金流 | | | | |

{关键财务特征解读：2-4 句，指出最重要的亮点和隐忧，数据支撑}

## 估值定位
- 当前 PE：X 倍（历史均值 X，行业均值 X）
- 历史 PE 分位：X%（<30% 偏低估，>70% 偏高估）
- PEG：X（<1 通常认为成长合理，成长股参考）
- 综合判断：{低估/合理/高估} — {一句话理由}

## 机构观点
- 评级分布：买入 X 家 / 增持 X 家 / 中性 X 家 / 卖出 X 家
- 目标价区间：X-X 元（较当前溢价/折价 X%）
- EPS 预测：{年份} X 元 / {年份} X 元
- 研报核心逻辑：{最新深度研报的 1-2 个核心投资观点}

## 近期重要动态
{公告 + 新闻整合，时间倒序，5 条以内，每条一句话，只写真正重要的}

## 行业背景
行业估值分位：X% | 近期主力：净流入/净流出 X 亿
{行业景气度判断：1-2 句，结合宏观背景}

## 历史研究记录（语雀）
{若有：按时间倒序列出，每条格式为"[日期] {关键结论}"}
{若无：「暂无历史记录，本次为首次研究」}

## 综合结论

**投资价值**：高 / 中 / 低

**核心逻辑**（若看好，每条须有数据支撑）：
1. {正面论据1}
2. {正面论据2}

**核心风险**（必写，即使看好）：
1. {风险1}
2. {风险2}

**建议持仓策略**：
{核心仓（≤20%）/ 卫星仓（5-10%）/ 观察位（≤5%）} — {一句话理由}
```

### 分析后：询问写回语雀

> 研究报告完成。是否保存到语雀知识库？
> 文档名：`{YYYY-MM-DD} 个股研究 {标的名}`

用户确认后，执行【语雀写回流程】（见文末）。
```

- [ ] **Step 2：验证 Mode C 和 Mode D 已添加**

```bash
grep -n "## Mode [CD]" /Users/mocha/claude_workspace/iwencai-investment-engine/.claude/skills/investment-engine/SKILL.md
```

预期输出：Mode C 和 Mode D 对应的行号

- [ ] **Step 3：验证 Mode C**

开新对话，输入：
```
今天看到美联储暂停降息的消息，我持有新能源和消费板块，会有什么影响？
```

预期行为：
- 触发 Mode C
- 调用 `hithink-macro-query`（利率数据）、`hithink-zhishu-query`（指数反应）、`hithink-sector-selector`（两个行业各一次）、`news-search`、`report-search`
- 输出传导链 + 持仓影响评估表 + 建议优先级

- [ ] **Step 4：验证 Mode D**

开新对话，输入：
```
帮我深度研究一下比亚迪
```

预期行为：
- 触发 Mode D
- 按顺序调用 10 个 Skills（basicinfo → market → finance×3 → insresearch → announcement → report → news → sector）
- 调用 `yuque_search` 检索历史记录
- 输出含财务表格、估值分位、综合结论的完整研究报告
- 分析结束后询问是否写回语雀

- [ ] **Step 5：Commit**

```bash
cd /Users/mocha/claude_workspace/iwencai-investment-engine
git add .claude/skills/investment-engine/SKILL.md
git commit -m "feat: add Mode C (macro analysis) and Mode D (stock research)"
```

---

## Task 5：SKILL.md 增加语雀写回流程

**Files:**
- Modify: `.claude/skills/investment-engine/SKILL.md`

- [ ] **Step 1：在 SKILL.md 末尾追加语雀写回流程**

在所有 Mode 章节后追加：

```markdown
---

## 语雀写回流程

当用户确认保存分析结果时，按以下步骤执行：

### Step 1：检查目录结构

调用 `mcp__yuque-mcp__yuque_get_toc`：
- 参数：`namespace = "{YUQUE_NAMESPACE}"`
- 从返回结果中获取 `book_id`（整数 ID）
- 检查 TOC 中是否存在「投资分析资产库」group 节点，以及当前模式对应的子 group 节点

### Step 2：按需创建缺失的目录节点

若「投资分析资产库」不存在，调用 `mcp__yuque-mcp__yuque_update_toc` 创建：
```
新建 group 节点，title: "投资分析资产库"
```

若对应模式子目录不存在，调用 `mcp__yuque-mcp__yuque_update_toc` 在「投资分析资产库」下创建：
```
持仓复盘 → title: "持仓复盘"
买卖决策 → title: "买卖决策"
宏观分析 → title: "宏观分析"
个股研究 → title: "个股研究"
```

### Step 3：新建文档

调用 `mcp__yuque-mcp__yuque_create_doc`：
```
book_id: {Step 1 获取的 book_id}
title:   "{YYYY-MM-DD} {模式} {关键词}"
body:    {本次分析的完整 Markdown 输出内容}
format:  "markdown"
```

文档标题关键词规则：
- 持仓复盘：列出主要标的名称（最多 3 个，超出用「等X只」），如 `2026-04-17 持仓复盘 茅台+宁德+比亚迪`
- 买卖决策：标的名称，如 `2026-04-17 买卖决策 宁德时代`
- 宏观分析：事件核心关键词（5字以内），如 `2026-04-17 宏观分析 美联储暂停降息`
- 个股研究：标的名称，如 `2026-04-17 个股研究 比亚迪`

### Step 4：挂载到正确目录

调用 `mcp__yuque-mcp__yuque_update_toc` 将新文档节点移动到对应模式子目录下。

### Step 5：确认结果

- **成功**：展示文档 URL（从 create_doc 响应中获取）
- **失败**：输出完整 Markdown 内容供用户手动复制，说明失败原因，不静默失败

### 时间规范

文档标题中的日期使用**北京时间**（UTC+8）。
MCP 接口若需要 UTC 参数，调用前自动换算，对用户完全透明。
```

- [ ] **Step 2：验证语雀流程已添加**

```bash
grep -n "语雀写回流程" /Users/mocha/claude_workspace/iwencai-investment-engine/.claude/skills/investment-engine/SKILL.md
```

预期输出：包含「语雀写回流程」的行号

- [ ] **Step 3：验证语雀写回**

完成任意一次分析（例如 Mode A 持仓复盘），当出现询问写回提示时，回答「是」，验证：
1. `yuque_get_toc` 成功获取 invest-kb 的 book_id
2. 若「投资分析资产库」目录不存在，自动创建
3. `yuque_create_doc` 成功创建文档
4. 返回可访问的语雀 URL

- [ ] **Step 4：Commit**

```bash
cd /Users/mocha/claude_workspace/iwencai-investment-engine
git add .claude/skills/investment-engine/SKILL.md
git commit -m "feat: add yuque write-back flow to investment-engine skill"
```

---

## Task 6：端到端验证

**目标：确认四种模式 + 语雀集成均正常工作。**

- [ ] **Test 1：项目上下文加载验证**

在 `iwencai-investment-engine` 目录下开新对话，输入：
```
你好
```

预期：CLAUDE.md 自动加载，Claude 知道这是投资分析引擎项目，知道语雀知识库地址和 Skills 列表。

- [ ] **Test 2：持仓复盘（Mode A）完整流程**

输入：
```
帮我复盘持仓：贵州茅台 30%（均价 1600），宁德时代 20%（均价 200），招商银行 10%
```

通过标准：
- 三只股票各调用 3 个 Skills（market / finance / insresearch）
- 各调用一次 yuque_search 检索历史
- 输出三张健康度简卡 + 整体小结
- 结束后询问是否写回语雀

- [ ] **Test 3：买卖决策（Mode B）情绪刹车验证**

输入：
```
宁德时代最近跌了很多，感觉跌不动了，是不是可以加仓了？我持有 20%
```

通过标准：
- 检测到情绪信号（"感觉跌不动了"）→ 先输出情绪刹车问句
- 用户回答后拉取 5 类数据
- 输出选取 2-4 个维度的分析 + 必写的最强反对意见

- [ ] **Test 4：宏观分析（Mode C）**

输入：
```
央行今天宣布降准 0.5 个百分点，对我持有的银行股和消费股有什么影响？
```

通过标准：
- 调用 hithink-macro-query（社融/M2）、hithink-zhishu-query（指数）、hithink-sector-selector（银行、消费板块各一次）
- 输出传导链 + 逐持仓影响评估表

- [ ] **Test 5：个股研究（Mode D）**

输入：
```
帮我全面研究一下腾讯控股
```

通过标准：
- 按顺序调用 10 个 Skills
- 输出含财务表格、估值分位、机构观点、综合结论的完整报告
- 结束后询问是否写回语雀

- [ ] **Test 6：语雀写回验证**

完成 Test 5 后确认写入，登录语雀 https://www.yuque.com/{YUQUE_NAMESPACE} 验证：
- `投资分析资产库/个股研究/` 目录存在
- 文档 `{今日日期} 个股研究 腾讯控股` 已创建，内容完整

---

## Self-Review

**Spec Coverage：**
- [x] 持仓数据手动输入 → Task 2 Mode A 中有收集提示
- [x] 市场数据通过 iwencai Skills → 所有 Mode 均通过 Skill 名称指定，无 Python 脚本
- [x] 语雀历史检索（分析前） → 每个 Mode 均有「分析前：语雀检索」步骤
- [x] 持仓复盘 Mode A → Task 2 完整实现
- [x] 买卖决策 Mode B + 五维框架 → Task 3 完整实现
- [x] 宏观分析 Mode C → Task 4 完整实现
- [x] 个股研究 Mode D → Task 4 完整实现（10步全维度）
- [x] 语雀写回（按日期新建文档，按模式归档） → Task 5 完整实现
- [x] 情绪刹车（任何模式优先） → Task 2 SKILL.md 框架部分实现
- [x] 北京时间规范 → Task 1 CLAUDE.md + Task 5 语雀流程均有说明
- [x] 数据来源标注 → 每种 Mode 输出格式末尾均有标注

**Placeholder Scan：** 无 TBD/TODO，所有输出模板含完整结构，Skill 名称全部指定，Yuque 工具调用参数完整。

**一致性检查：**
- `yuque_search` 参数在 Mode A/B/C/D 中保持一致：`scope: "{YUQUE_NAMESPACE}"`
- 写回文档标题格式在 Task 5 中统一定义，各 Mode 询问写回时引用一致
- 信号颜色（🟢/🟡/🔴）只在 Mode A 使用，其他 Mode 使用文字判断，与 spec 一致
