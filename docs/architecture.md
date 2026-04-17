# iwencai-investment-engine 技术架构文档

**版本：** 1.0  
**日期：** 2026-04-17  
**维护人：** MoCha.

---

## 一、项目定位

投资交易分析引擎，串联三类原本独立的信息孤岛：

| 信息孤岛 | 来源 | 接入方式 |
|---------|------|---------|
| 持仓数据 | 用户各券商 App | 每次对话手动输入 |
| 市场数据 | 同花顺问财 | 通过 iwencai Skills 调用 |
| 个人投资知识 | 语雀 invest-kb | 通过 yuque MCP 检索与写回 |

**核心设计约束：**
- **轻量优先**：不引入独立数据库或持续部署服务，全部能力通过 Claude Code 配置文件实现
- **效果优先**：宁可流程简单，也要分析质量高
- **可持续沉淀**：每次分析都有机会反哺语雀知识库，形成历史经验正循环
- **数据获取约束**：所有金融市场数据统一通过项目内 iwencai Skills 调用，不编写脚本直接请求 API

---

## 二、整体架构

### 2.1 系统分层

```
┌─────────────────────────────────────────────────────────────────┐
│                          用户对话层                               │
│              Claude Code CLI / Web / IDE Extension               │
└──────────────────────────────┬──────────────────────────────────┘
                               │ 触发
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                         路由与编排层                               │
│  CLAUDE.md（项目大脑）                                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  模式路由规则  →  investment-engine Skill（SKILL.md）    │    │
│  │  情绪刹车机制  →  四种分析模式（A/B/C/D）               │    │
│  │  五维分析框架  →  输出格式规范                           │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────┬───────────────────────────────────────┬──────────────┘
           │                                       │
           ▼                                       ▼
┌──────────────────────┐              ┌────────────────────────────┐
│   市场数据层          │              │      知识存储层              │
│   iwencai Skills      │              │  语雀 MCP（yuque-mcp）      │
│   ─────────────────  │              │  ─────────────────────── │
│   同花顺问财 OpenAPI  │              │  {YUQUE_NAMESPACE}     │
│   Bearer Token 认证  │              │  分析前检索 + 分析后写回    │
└──────────────────────┘              └────────────────────────────┘
```

### 2.2 目录结构

```
iwencai-investment-engine/
├── CLAUDE.md                              # 项目大脑：路由、配置、Skills速查
├── docs/
│   ├── architecture.md                   # 本文档
│   ├── iwencai-skills-index.md           # 15 个 Skills 完整索引
│   ├── 需求.md                           # 原始需求文档
│   ├── iwencai-investment-engine白皮书.md # 白皮书
│   └── superpowers/
│       ├── specs/
│       │   └── 2026-04-16-investment-engine-design.md  # 设计文档
│       └── plans/
│           └── 2026-04-17-investment-engine.md         # 实施计划
└── .claude/
    └── skills/
        ├── investment-engine/
        │   ├── SKILL.md                           # 分析引擎主逻辑（核心文件）
        │   ├── references/
        │   │   ├── yuque-retrieval-rules.md       # 语雀检索规范（✅/❌过滤规则）
        │   │   ├── mode-b-five-dimensions.md      # Mode B 五维分析框架
        │   │   ├── mode-c-data-pipeline.md        # Mode C 宏观数据拉取管道
        │   │   ├── mode-d-data-pipeline.md        # Mode D 全维度数据拉取管道（10步）
        │   │   └── yuque-writeback-flow.md        # 语雀写回5步流程
        │   └── assets/
        │       └── mode-d-report-template.md      # Mode D 研究报告输出模板
        ├── hithink-market-query/         # 行情数据
        │   ├── SKILL.md
        │   ├── references/api.md
        │   └── scripts/cli.py
        ├── hithink-finance-query/        # 财务数据
        ├── hithink-basicinfo-query/      # 基本资料
        ├── hithink-insresearch-query/    # 机构研究
        ├── hithink-macro-query/          # 宏观数据
        ├── hithink-zhishu-query/         # 指数行情
        ├── hithink-astock-selector/      # A股筛选
        ├── hithink-etf-selector/         # ETF筛选
        ├── hithink-sector-selector/      # 板块筛选
        ├── hithink-hkstock-selector/     # 港股筛选
        ├── hithink-usstock-selector/     # 美股筛选
        ├── hithink-futures-query/        # 期货期权
        ├── news-search/                  # 财经新闻
        │   ├── SKILL.md
        │   ├── references/api.md
        │   └── scripts/
        ├── announcement-search/          # 公告搜索
        └── report-search/                # 研报搜索
```

---

## 三、核心组件说明

### 3.1 CLAUDE.md — 项目大脑

项目级配置文件，由 Claude Code 在每次对话时自动加载。承担以下职责：

1. **声明项目定位**：三类信息孤岛的融合目标
2. **分析模式路由**：定义触发关键词 → 激活 `investment-engine` Skill 的映射规则
3. **语雀知识库配置**：namespace、归档目录结构、时间规范（北京时间 UTC+8）
4. **iwencai Skills 速查索引**：15 个 Skills 按用途分组的快速参考
5. **数据来源声明**：强制所有市场数据分析标注「数据来源：同花顺问财」

### 3.2 investment-engine SKILL.md — 分析引擎

系统的核心编排逻辑，被 investment-engine Skill 加载执行。包含：

- **模式识别逻辑**：4 种分析模式的语义判断规则
- **情绪刹车机制**：全局前置，检测追涨/恐慌/FOMO信号后先暂停分析
- **数据编排策略**：每种模式下 Skills 的调用顺序与组合
- **五维分析框架**（Mode B 专用）：业务理解、估值、仓位、情绪隔离、持有逻辑
- **语雀集成规范**：检索参数、写回流程、错误处理
- **输出格式规范**：各模式的标准化 Markdown 模板

---

## 四、iwencai Skills 体系

### 4.1 API 端点

| 类型 | 端点 | 用途 |
|------|------|------|
| 数据查询 | `POST https://openapi.iwencai.com/v1/query2data` | 结构化金融数据 |
| 资讯搜索 | `POST https://openapi.iwencai.com/v1/comprehensive/search` | 新闻/公告/研报 |

**认证：** 所有请求头携带 `Authorization: Bearer {IWENCAI_API_KEY}`，密钥通过环境变量注入。

### 4.2 Skills 分类

#### 数据查询类（7 个）— 接口：`/v1/query2data`

| Skill ID | 核心能力 | 主要字段 |
|----------|---------|---------|
| `hithink-market-query` | 实时价格、涨跌幅、成交量/额、换手率、**主力资金净流入**、技术指标（MACD/KDJ/RSI/布林线） | 最新价、涨跌幅、主力净流入、52周高低 |
| `hithink-finance-query` | 营收、净利润、毛利率、ROE/ROA、**自由现金流**、PE/PB/PEG/PS | 近三年财务数据、估值倍数 |
| `hithink-basicinfo-query` | 上市日期、行业分类、主营业务、基金费率、期货合约到期 | 基础资料字段 |
| `hithink-insresearch-query` | 券商研报评级（买入/增持/中性/卖出）、**EPS预测**、目标价、基金星级 | 评级分布、目标价区间 |
| `hithink-macro-query` | GDP、CPI/PPI、**M2/PMI/社融/LPR**、汇率、进出口 | 宏观指标时间序列 |
| `hithink-zhishu-query` | 上证/深证/创业板/沪深300/中证500/恒生/纳斯达克点位、涨跌幅 | 指数行情 |
| `hithink-futures-query` | 期货行情、**隐含/历史波动率**、库存/产量/销量、会员持仓排行 | 期货衍生品数据 |

#### 智能筛选类（5 个）— 接口：`/v1/query2data`，空数据时自动放宽条件重试（最多 2 次）

| Skill ID | 核心筛选维度 |
|----------|------------|
| `hithink-astock-selector` | 行情指标、技术形态（均线/突破/K线）、财务指标、行业概念 |
| `hithink-etf-selector` | 跟踪指数、资产规模/份额变化、费率/跟踪误差、风格类型 |
| `hithink-sector-selector` | **行业PE/PB估值分位**、主力资金净流入/北向资金、涨跌幅排名 |
| `hithink-hkstock-selector` | 行情/财务/行业概念、陆港通持股（北向/南向） |
| `hithink-usstock-selector` | 行情/财务/行业概念、业绩预测（盈利增速）、研报评级 |

#### 资讯搜索类（3 个）— 接口：`/v1/comprehensive/search`，固定参数 `app_id: "AIME_SKILL"`

| Skill ID | `channels` 值 | 数据来源 |
|----------|--------------|---------|
| `news-search` | `["news"]` | 官媒、主流财经媒体（证券时报/经济观察报等）、垂直行业网站、上市公司官网 |
| `announcement-search` | `["announcement"]` | A股/港股/基金/ETF公告：定期报告、分红、回购增持、资产重组、业绩预告 |
| `report-search` | `["report"]` | 主流投研机构（券商/基金/独立研究机构）深度研究报告 |

### 4.3 Skills 调用规范

每个 Skill 目录结构统一：
```
{skill-name}/
├── SKILL.md          # 技能说明（由 Claude Skill 工具加载执行）
├── references/
│   └── api.md        # 接口文档（部分 Skill 含此文件）
└── scripts/
    └── cli.py        # CLI 入口（可独立命令行调用）
```

**调用约束：**
- investment-engine Skill 加载后，通过 `Bash` 工具执行 Python 代码调用 API
- 不允许绕过 Skill 直接在 investment-engine 层编写 API 调用脚本
- API Key 统一通过环境变量 `IWENCAI_API_KEY` 注入，不硬编码

---

## 五、四种分析模式

### 5.1 模式路由

```
用户输入
    │
    ▼
CLAUDE.md 关键词匹配
    │
    ├── 持仓相关词 ──────────────────────────────→ Mode A（持仓复盘）
    │   (复盘/持仓情况/仓位健康/看看我的仓)
    │
    ├── 买卖操作词 ──────────────────────────────→ Mode B（买卖决策）
    │   (买/卖/加仓/减仓/止损/要不要/该不该)
    │
    ├── 宏观事件词 ──────────────────────────────→ Mode C（宏观分析）
    │   (宏观/政策/消息影响/利好利空/大盘)
    │
    └── 个股研究词 ──────────────────────────────→ Mode D（个股研究）
        (研究/深挖/分析/生成研报/全面了解)
```

情绪刹车在任何模式前优先执行，检测到追涨/恐慌/FOMO信号时暂停分析，等待用户理性确认。

### 5.2 Mode A — 持仓复盘

**数据流：**
```
用户输入持仓列表
    ↓
对每只标的并行/循环：
    ├── hithink-market-query  →  价格、涨跌幅、主力资金
    ├── hithink-finance-query →  ROE、负债率、净利润同比
    ├── hithink-insresearch-query → 机构评级、目标价
    └── yuque_search          →  该标的历史记录
    ↓
输出健康度评估表（🟢🟡🔴 三色信号）
    ↓
持仓整体小结
```

**信号颜色标准：**
- 🟢 主力净流入 > 0 且财务增速为正 且机构以买入/增持为主
- 🟡 资金方向不明 或 财务增速趋缓 或 机构评级分歧
- 🔴 主力净流出 > 2亿 或 净利润同比下滑 > 20% 或 机构以中性/卖出为主
- ⚪ 数据无法获取（不影响其他维度判断）

### 5.3 Mode B — 买卖决策

**数据流：**
```
收集上下文（标的/操作意向/触发原因）
    ↓
情绪刹车检测
    ↓
yuque_search  →  历史操作记录（最关键，提供决策一致性参考）
    ↓
hithink-market-query     →  价格位置（52周区间分位）
hithink-finance-query    →  估值安全边际（PE/PB/PEG）
hithink-insresearch-query →  机构共识
announcement-search      →  近期重大公告
news-search              →  近期催化剂/利空事件
    ↓
五维分析框架（选取 2-4 个最相关维度）
    ↓
综合判断（支持/谨慎/反对）+ 必写【最强反对意见】
```

**五维分析框架：**

| 维度 | 核心问题 | 适用场景 |
|------|---------|---------|
| 维度一：业务理解与护城河 | 公司靠什么赚钱？护城河是什么？ | 首次研究未知标的 |
| 维度二：估值与安全边际 | 当前价格在历史估值什么分位？ | 任何买入决策 |
| 维度三：仓位合理性 | 这个标的占总资产比例是否过高？ | 加仓/首次建仓 |
| 维度四：情绪隔离 | 判断触发器是基本面还是价格信号？ | 检测到情绪信号时 |
| 维度五：持有逻辑检验 | 当初买入逻辑现在还成立吗？ | 已持有标的的操作 |

### 5.4 Mode C — 宏观分析

**数据流：**
```
确认宏观事件 + 用户持仓/关注行业
    ↓
yuque_search          →  历史相似宏观环境下的应对记录
    ↓
hithink-macro-query   →  相关指标时间序列（GDP/CPI/PMI/LPR等）
hithink-zhishu-query  →  主要指数近期反应
hithink-sector-selector → 持仓行业资金流向 + 估值分位（最多3个行业）
news-search           →  主流机构对事件的解读
report-search         →  相关策略研报核心观点
    ↓
输出核心传导链 + 持仓影响评估表 + 建议动作优先级
```

### 5.5 Mode D — 个股研究

最全面的分析模式，10步数据编排：

| 步骤 | Skill | 查询维度 |
|------|-------|---------|
| 1 | `hithink-basicinfo-query` | 上市日期、行业、主营业务 |
| 2 | `hithink-market-query` | 最新价、市值、52周区间、多周期涨跌幅、主力资金 |
| 3 | `hithink-finance-query` | 近三年营收/净利润/毛利率/净利率（利润表） |
| 4 | `hithink-finance-query` | 近三年ROE/负债率/经营现金流/自由现金流（资产负债表） |
| 5 | `hithink-finance-query` | 当前PE/PB/PEG、历史PE分位（估值） |
| 6 | `hithink-insresearch-query` | 评级分布、目标价区间、EPS预测 |
| 7 | `announcement-search` | 近三个月重要公告（分红/回购/业绩预告/重大合同） |
| 8 | `report-search` | 最新深度研报核心观点和投资逻辑 |
| 9 | `news-search` | 近期重要新闻动态 |
| 10 | `hithink-sector-selector` | 行业估值分位、近期主力资金净流向 |
| + | `yuque_search` | invest-kb 历史研究记录（所有相关文档） |

**数据缺失处理：** 任何步骤无法获取数据时，在报告对应位置标注「暂无数据」，不中断整体流程。

---

## 六、语雀集成规范

### 6.1 知识库结构

```
{YUQUE_NAMESPACE}（语雀知识库）
└── {YUQUE_ARCHIVE_DIR}/     ← 首次写入时若不存在则自动新建（默认：投资分析资产库）
    ├── 持仓复盘/
    ├── 买卖决策/
    ├── 宏观分析/
    └── 个股研究/
```

### 6.2 分析前检索（前置步骤）

每种模式在正式分析前统一执行：

```
工具：mcp__yuque-mcp__yuque_search
参数：
  q:     "{标的名 或 宏观关键词}"
  type:  "doc"
  scope: "{YUQUE_NAMESPACE}"（全局搜索，非限定路径）

结果处理：
  有结果 → 提取历史分析结论和操作逻辑（见 6.2.1 规范），注入分析上下文
  无结果 → 标注「知识库暂无相关记录，本次为首次分析」，继续分析
```

### 6.2.1 检索内容过滤规范

对检索结果必须严格区分「可用信息」和「忽略信息」：

**✅ 应提取的内容**
- 历史分析结论（如「曾认为估值过高」「当时买入逻辑是 XX」）
- 历史操作决策与复盘（买入/卖出的理由、判断失误的教训）
- 宏观/行业观点（类似宏观环境下的历史应对策略）
- 投资方法论、心法、原则性内容

**❌ 必须忽略的内容（时效性强，极易过时）**
- 持仓数量、持仓比例、买入均价、当前盈亏数字
- 「目标仓位 X 万」「计划买入 X 股」等操作计划
- portfolio_analysis / portfolio_allocation / 投资日记中的具体仓位数据

**原因：持仓情况变化频繁，历史仓位数据随时可能已不反映真实情况，直接引用会导致分析基于错误前提。**

### 6.2.2 持仓数据唯一来源

**涉及个人持仓的任何分析（Mode A 持仓复盘、Mode B 已持有场景），持仓事实（持有数量/比例/成本价）只接受用户在本次对话中实时输入，不得从语雀历史文档中读取。**

- Mode C 宏观分析：只需用户提供持仓的标的名称和行业方向，不需要数量和成本价
- Mode D 个股研究「已持有」场景：需用户提供当前仓位比例和成本价，用于持仓策略建议

### 6.3 分析后写回流程

分析完成后询问用户是否保存，用户确认后执行 5 步写回：

```
Step 1：yuque_get_toc(repo_id="{YUQUE_NAMESPACE}")
        → 检查 TOC 中「{YUQUE_ARCHIVE_DIR}」group 节点及模式子目录是否存在
        → 记录各节点的 uuid（Step 2/4 作为 target_uuid 使用）
        → 注意：yuque_create_doc 直接用 YUQUE_NAMESPACE 作 repo_id，无需单独数字 book_id

Step 2：按需 yuque_update_toc
        → 若「{YUQUE_ARCHIVE_DIR}」不存在则新建 group 节点
        → 若对应模式子目录不存在则在其下新建子 group 节点

Step 3：yuque_create_doc(repo_id="{YUQUE_NAMESPACE}", title, body, format="markdown")
        → 创建文档，body 为本次分析完整 Markdown 内容

Step 4：yuque_update_toc（prependNode / appendNode）
        → 将新文档节点挂载到对应模式子目录下（用 Step 1 记录的 uuid 定位目录）

Step 5：确认反馈
        → 成功：展示文档 URL（从 create_doc 响应获取）
        → 失败：输出完整 Markdown 供手动复制，说明失败原因，不静默失败
```

### 6.4 文档命名规范

| 模式 | 文档名格式 | 示例 |
|------|----------|------|
| 持仓复盘 | `{YYYY-MM-DD} 持仓复盘 {标的1+标的2+...（最多3个）}` | `2026-04-17 持仓复盘 茅台+宁德+比亚迪` |
| 买卖决策 | `{YYYY-MM-DD} 买卖决策 {标的名}` | `2026-04-17 买卖决策 宁德时代` |
| 宏观分析 | `{YYYY-MM-DD} 宏观分析 {关键词5字内}` | `2026-04-17 宏观分析 美联储降息` |
| 个股研究 | `{YYYY-MM-DD} 个股研究 {标的名}` | `2026-04-17 个股研究 比亚迪` |

### 6.5 时间规范

所有时间统一使用**北京时间（UTC+8）**展示：
- 文档标题日期
- 检索结果中的时间字段展示
- MCP 接口若需传入 UTC 参数，调用前自动换算（北京时间 -8 小时），对用户透明

---

## 七、数据流总览

```
用户输入
    │
    ▼
① CLAUDE.md 路由规则匹配 → 激活 investment-engine Skill
    │
    ▼
② 情绪刹车检测（全局前置）
    │ 检测到情绪信号 → 暂停，等用户理性确认后继续
    │
    ▼
③ 语雀知识库检索（mcp__yuque-mcp__yuque_search）
    │ → 注入历史经验到分析上下文
    │
    ▼
④ 根据模式并发/串行调用 iwencai Skills
    │
    ├── 数据查询类 → /v1/query2data（返回结构化数据表格）
    └── 资讯搜索类 → /v1/comprehensive/search（返回新闻/公告/研报列表）
    │
    ▼
⑤ 综合数据 + 历史经验 → 输出结构化分析报告（Markdown）
    │
    ▼
⑥ 询问是否保存 → 用户确认 → 语雀写回流程（5步）
```

---

## 八、环境配置

### 8.1 必要环境变量

所有环境变量通过项目根目录的 **`.env`** 文件管理（已 gitignore），参考 `.env.example` 填写。

| 变量名 | 用途 | 获取方式 |
|--------|------|---------|
| `IWENCAI_API_KEY` | 同花顺问财 OpenAPI 认证（Bearer Token）| 向同花顺申请 |
| `YUQUE_NAMESPACE` | 语雀知识库命名空间（格式：用户名/知识库slug）| 从语雀知识库 URL 获取 |
| `YUQUE_KB_URL` | 语雀知识库完整访问 URL | 同上 |
| `YUQUE_ARCHIVE_DIR` | 分析结果在语雀中的归档根目录名 | 默认填写「投资分析资产库」，可自定义 |

初始化方式：
```bash
cp .env.example .env   # 复制模板
# 然后编辑 .env 填入真实值
```

### 8.2 必要 MCP Server

| MCP | 用途 |
|-----|------|
| `yuque-mcp` | 语雀知识库读写（yuque_search / yuque_create_doc / yuque_get_toc / yuque_update_toc 等） |

### 8.3 Claude Code Skills 配置

所有 Skills 存放于 `.claude/skills/` 目录，Claude Code 自动发现并可通过 `Skill` 工具调用。

---

## 九、设计边界（不在范围内）

| 功能 | 说明 |
|------|------|
| 持仓数据持久化 | 每次对话手动输入，不存储到数据库 |
| 实时价格提醒 | 不做持续运行的价格监控服务 |
| 自动触发分析 | 不做定时任务，全部由用户主动发起 |
| 直接调用 API 脚本 | 统一通过 iwencai Skills 调用，不在 investment-engine 层写脚本 |
| 期货/期权专项分析 | 可通过 `hithink-futures-query` 后续扩展 |
| 港股/美股专项研究 | 可通过 `hithink-hkstock-selector` / `hithink-usstock-selector` 后续扩展 |

---

## 十、扩展指引

### 新增 Skill

1. 在 `.claude/skills/` 下新建目录，结构参考现有 Skill
2. 创建 `SKILL.md`（包含 `name`/`description` frontmatter）
3. 在 `docs/iwencai-skills-index.md` 中补充索引条目
4. 在 `CLAUDE.md` 的速查表中添加映射关系
5. 在 `investment-engine/SKILL.md` 对应模式的数据编排中补充调用步骤

### 新增分析模式

1. 在 `CLAUDE.md` 的「分析模式路由」表中添加触发词
2. 在 `investment-engine/SKILL.md` 中添加新模式的完整工作流（数据编排 + 输出格式）
3. 在语雀「投资分析资产库」下为新模式创建对应子目录

### 数据来源声明

所有分析报告末尾必须标注：

```
*数据来源：同花顺问财*
```
