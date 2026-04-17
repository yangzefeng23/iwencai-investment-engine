# iwencai-investment-engine

> 私人投资交易分析引擎。串联持仓数据、同花顺市场数据、语雀投资知识库，辅助做出更一致的投资决策。

---

## 项目简介

投资者日常分析面临三类信息孤岛：

| 信息孤岛 | 来源 | 接入方式 |
|---------|------|---------|
| 持仓数据 | 各券商 App | 每次对话手动输入 |
| 市场数据 | 同花顺问财 | 通过 15 个 iwencai Skills 调用 |
| 个人投资知识 | 语雀知识库 | 通过 yuque MCP 检索与写回 |

本项目通过 Claude Code 配置文件（零服务器、零数据库）将三者串联，提供四种标准化分析模式，并在每次分析后将洞见写回语雀，形成持续沉淀的投资知识闭环。

## 功能特性

- **四种分析模式**：持仓复盘（A）/ 买卖决策（B）/ 宏观分析（C）/ 个股研究（D）
- **情绪刹车机制**：全局前置检测追涨/恐慌/FOMO 信号，等理性确认后再继续分析
- **五维分析框架**：业务理解、估值安全边际、仓位合理性、情绪隔离、持有逻辑检验
- **15 个 iwencai Skills**：覆盖行情/财务/宏观/指数/机构研究/公告/研报等全品类数据
- **语雀知识闭环**：分析前自动检索历史经验，分析后用户确认写回，持续沉淀

## 快速开始

### 前置条件

- [Claude Code](https://claude.ai/code) CLI
- 同花顺问财 OpenAPI 密钥（`IWENCAI_API_KEY`）
- 语雀账号 + `yuque-mcp` MCP Server

### 初始化配置

```bash
# 1. 克隆项目
git clone <repo-url>
cd iwencai-investment-engine

# 2. 复制配置模板
cp .env.example .env

# 3. 编辑 .env，填入你的配置
#    IWENCAI_API_KEY=your_iwencai_api_key
#    YUQUE_NAMESPACE=your_username/your_book_slug
#    YUQUE_KB_URL=https://www.yuque.com/your_username/your_book_slug
#    YUQUE_ARCHIVE_DIR=投资分析资产库
```

### 配置 MCP Server

在 Claude Code 的 MCP 配置中添加 `yuque-mcp`：

```json
{
  "yuque-mcp": {
    "command": "npx",
    "args": ["yuque-mcp"],
    "env": {
      "YUQUE_TOKEN": "your_yuque_token"
    }
  }
}
```

### 使用示例

在 Claude Code 中打开此项目目录，直接用自然语言触发：

```
# 持仓复盘
帮我看看我的持仓情况

# 买卖决策
光迅科技值不值得持有？

# 宏观分析
美联储暂停降息对我的仓位有什么影响？

# 个股研究
深度研究宁德时代
```

## 目录结构

```
iwencai-investment-engine/
├── CLAUDE.md                              # 项目大脑：路由规则、语雀配置、Skills速查
├── .env.example                           # 环境变量模板（复制为 .env 后填入真实值）
├── .gitignore
├── index.html                             # 项目介绍网站（GitHub Pages 入口）
├── docs/
│   ├── architecture.md                    # 技术架构文档
│   ├── architecture-diagram.html          # 架构设计图（可视化）
│   ├── DESIGN.md                          # 设计系统参考（Notion 风格）
│   ├── iwencai-skills-index.md            # 15 个 Skills 完整索引
│   ├── iwencai-investment-engine白皮书.md  # 项目白皮书
│   └── 需求.md                            # 原始需求文档
└── .claude/
    └── skills/
        ├── investment-engine/             # 分析引擎主体
        │   ├── SKILL.md                   # 核心逻辑（319行）
        │   ├── references/                # 详细规范（5个文件）
        │   └── assets/                    # 输出模板（1个文件）
        ├── hithink-market-query/          # 行情数据
        ├── hithink-finance-query/         # 财务数据
        ├── hithink-basicinfo-query/       # 基本资料
        ├── hithink-insresearch-query/     # 机构研究
        ├── hithink-macro-query/           # 宏观数据
        ├── hithink-zhishu-query/          # 指数行情
        ├── hithink-futures-query/         # 期货期权
        ├── hithink-astock-selector/       # A股筛选
        ├── hithink-etf-selector/          # ETF筛选
        ├── hithink-sector-selector/       # 板块筛选
        ├── hithink-hkstock-selector/      # 港股筛选
        ├── hithink-usstock-selector/      # 美股筛选
        ├── news-search/                   # 财经新闻
        ├── announcement-search/           # 公告搜索
        └── report-search/                 # 研报搜索
```

## 分析模式

| 模式 | 触发关键词 | 核心能力 |
|------|-----------|---------|
| **A · 持仓复盘** | 复盘、持仓情况、仓位健康 | 三色信号（🟢🟡🔴）健康度评估 |
| **B · 买卖决策** | 买、卖、加仓、要不要、该不该 | 五维框架分析 + 必写最强反对意见 |
| **C · 宏观分析** | 宏观、政策、利好/利空、大盘 | 传导链分析 + 持仓影响评估 |
| **D · 个股研究** | 研究、深挖、分析、生成研报 | 10步全维度数据编排 + 标准化报告 |

## iwencai Skills 体系

15 个 Skills 基于同花顺问财 OpenAPI，统一使用 `IWENCAI_API_KEY` Bearer Token 认证：

- **数据查询（7）**：`hithink-market-query` / `hithink-finance-query` / `hithink-basicinfo-query` / `hithink-insresearch-query` / `hithink-macro-query` / `hithink-zhishu-query` / `hithink-futures-query`
- **智能筛选（5）**：`hithink-astock-selector` / `hithink-etf-selector` / `hithink-sector-selector` / `hithink-hkstock-selector` / `hithink-usstock-selector`
- **资讯搜索（3）**：`news-search` / `announcement-search` / `report-search`

## 设计原则

- **轻量优先**：无独立服务器，无数据库，全部能力通过 Claude Code 配置文件实现
- **数据约束**：所有金融市场数据统一通过 iwencai Skills 获取，不在分析引擎层编写 API 脚本
- **持仓来源**：持仓数量/成本价只接受用户本次对话实时输入，不从语雀历史文档读取（防止过时数据污染分析）
- **写回门槛**：语雀写入必须用户主动确认，确保知识库质量

## 数据来源声明

所有市场数据均来源于**同花顺问财**（[iwencai.com](https://www.iwencai.com)）。所有分析报告末尾均标注数据来源。

---

*数据来源：同花顺问财*
