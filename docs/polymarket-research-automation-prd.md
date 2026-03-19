# Polymarket Research & Automation PRD

## 1. Document Info

- Product Name: Sakura Polymarket Research & Automation System
- Version: v0.1
- Status: Draft
- Date: 2026-03-19
- Owner: hzqsns
- Document Purpose: 定义一个面向 Polymarket 的研究、扫描、提醒、模拟交易与自动化执行系统，作为仓库后续实现蓝图。

## 2. Background

Polymarket 是一个高度信息驱动、规则驱动、流动性分层明显的预测市场。传统“凭感觉下注”的方式难以复制，也难以规模化。相对更可持续的收益来源通常来自：

- 规则理解优势：提前识别标题表述与实际结算口径之间的差异
- 信息处理优势：更快地将公开信息转化为概率判断
- 结构性机会：相关市场之间的定价不一致
- 高概率 carry：把“高确定性、短久期”的事件当作类债券机会处理
- 做市收益：点差、maker rebate、流动性奖励
- 领域专精优势：在狭窄赛道中建立长期可复用的判断力
- 持仓收益：部分市场可能存在 holding reward

因此，本产品不将“自动下单”作为第一目标，而是先构建一个可以持续发现 edge、验证 edge、管理风险的研究与执行系统。

## 3. Vision

建立一个可迭代的 Polymarket 研究与自动化平台，帮助用户完成以下闭环：

1. 从公开市场中自动发现值得研究的机会
2. 对机会进行规则、数据、新闻和结构层面的分析
3. 生成可执行的交易候选与风险说明
4. 先通过 paper trading 验证，再逐步进入受控实盘
5. 用统一的数据和复盘体系持续优化策略

## 4. Product Goals

### 4.1 Core Goals

- 自动扫描 Polymarket 市场，持续发现 alpha 候选
- 将“研究”与“交易执行”拆层，先验证再实盘
- 尽量降低对 LLM 的依赖，把 token 成本控制在合理范围
- 提供清晰的风险控制与 kill switch
- 让系统既可手动研究使用，也可逐步自动化

### 4.2 Success Metrics

- 每日可稳定扫描全部目标市场并产出候选列表
- 候选列表具备可解释性，能够说明机会来源
- paper trading 阶段可输出成交、滑点、PnL、回撤和命中率
- 实盘阶段具备单市场、单事件、总账户级风控
- 月度 token 成本和基础设施成本可被明确追踪

### 4.3 Non-Goals

- 不追求首版即全自动高频 bot
- 不以“跟单”为核心产品逻辑
- 不依赖无法验证的社交媒体喊单作为主要信号
- 不尝试绕过地域限制或平台规则

## 5. Target Users

### 5.1 Primary User

- 独立研究者 / 交易者
- 有基础编程能力，愿意长期积累研究框架
- 更关注“持续 edge”而不是一次性暴利故事

### 5.2 Secondary User

- 想搭建 Polymarket 量化研究基础设施的人
- 想把手动研究流程逐步自动化的人

## 6. Problem Statement

当前 Polymarket 的机会分散在不同层面，但缺少统一系统去处理：

- 市场数量多，手动筛选成本高
- 很多机会不是裸价格，而是规则、结构、流动性或时序问题
- 社交媒体上的收益案例信息噪声很大，难以验证
- 实盘执行和研究流程耦合，容易在未验证前就承担真实损失
- 如果无节制地用 LLM 全量分析，token 成本会快速上升

## 7. Product Principles

- Research First: 先研究后执行
- Human Override: 任何自动化都应允许人工介入
- Cheap by Default: 默认优先使用规则和代码，不默认调用 LLM
- Explainability: 每个候选信号必须能解释来源
- Risk Before Return: 风控优先于收益最大化
- Compliance Aware: 尊重平台与地区合规约束

## 8. Strategy Map

本系统将支持以下机会类型，但分阶段上线。

### 8.1 Rule Edge

定义：
通过解析 market question、resolution rules、resolution source、时间边界与特殊条款，识别市场定价对规则理解不足的情况。

示例：

- 标题看似直观，但实际结算依赖具体时间点
- 市场参与者忽略 source priority
- 对“before/after/by/on”类时间口径理解不一致

自动化程度：

- 中等
- 需要规则抓取 + 文本解析 + 人工复核

### 8.2 Information Edge

定义：
更快地把公开新闻、政策、事件进展与市场价格变化关联起来。

示例：

- 重大新闻发布后，某些市场反应迟缓
- 关联市场之间反应速度不一致

自动化程度：

- 中高
- 可由新闻抓取、事件时间线、价格异动监控支持

### 8.3 Structural Edge

定义：
由市场结构、结果空间或相关事件之间关系产生的机会。

示例：

- YES + NO 定价异常
- 相关市场之间概率和不合理
- negRisk 事件中的组合定价机会

自动化程度：

- 高
- 适合作为 scanner 的第一优先级

### 8.4 Market Making Edge

定义：
通过双边挂单、控制库存、获取 spread、maker rebate 与流动性奖励获利。

自动化程度：

- 高
- 但对风控与执行质量要求高

### 8.5 Holding Reward Edge

定义：
在 eligible events 中持有仓位以赚取平台奖励，同时管理价格风险。

自动化程度：

- 中
- 更像 carry strategy，需要监控奖励变化和仓位波动

### 8.6 High-Probability Carry Edge

定义：
将临近结算、确定性极高、剩余收益较小但周转较快的市场视为“类债券”机会，通过频繁复投获取年化收益。

示例：

- 价格已经达到 0.95 以上，但结算窗口短、规则明确、外部状态基本确定
- 事件剩余不确定性极低，但市场价格仍未完全收敛到 1

自动化程度：

- 中高
- 适合由规则筛选 + 风险白名单 + 人工复核共同支持

关键风险：

- 最大风险不是收益低，而是把“看起来确定”误判成“真正确定”
- 一次黑天鹅可能吞噬多次成功的微利

### 8.7 Domain Specialization Edge

定义：
在单一赛道中长期积累数据、语境和经验，形成高于平均水平的判断力。

示例：

- 只研究美国选举类市场
- 只研究 crypto 事件类市场
- 只研究 mention / speech / debate 类短时市场

自动化程度：

- 中
- 更偏“研究工作流增强”，不是单纯算法扫描

产品意义：

- 这类 edge 不一定来自复杂算法，而可能来自更好的标签体系、历史案例库和研究 checklist

### 8.8 Speed / Event-Reaction Edge

定义：
在公开信息出现后、市场尚未完全消化前，利用秒级到分钟级窗口完成反应和执行。

示例：

- 演讲、直播、政策发布、临场事件触发 mention 市场快速定价
- 重要新闻源、官方账号、直播字幕与盘口之间的时差

自动化程度：

- 高
- 但对低延迟基础设施和执行质量要求极高

产品边界：

- 该方向可以作为长期研究方向，但不纳入 MVP
- 首版只做监控与提醒，不做高频自动实盘

## 8A. Strategy Evaluation Framework

系统需要用统一标准评估策略，而不是只看单次收益。

### 8A.1 Evaluation Dimensions

- Absolute PnL: 总利润是否足够有意义
- Risk-Adjusted Return: 是否考虑回撤、波动和盈亏比
- Reproducibility: 是否可规则化、可重复执行
- Consistency: 是否跨周期稳定，而不是一次性命中
- Scalability: 资金扩大后是否仍能执行

### 8A.2 Exclusion Rules

以下类型不应被视为可复制策略：

- 依赖内幕信息
- 依赖市场操纵或争议性结算漏洞
- 单次超高仓位赌博式押注
- 无法解释的黑箱信号

### 8A.3 Product Requirement

系统对每个策略或 signal family 应保留：

- strategy_type
- target_market_universe
- expected_holding_period
- required_latency
- expected_capacity
- core_risks
- reproducibility_score
- confidence_score

## 9. Product Scope

系统分为五个产品层。

### 9.1 Layer A: Market Intelligence

目标：
完成市场采集、标准化、存储、指标计算和候选排序。

能力：

- 拉取 markets、events、orderbook、trades、positions 数据
- 标准化市场元信息
- 计算 spread、mid、成交量、隐含概率变化
- 标记市场类别、时间窗口、流动性状态
- 识别结构性机会
- 为市场和事件打主题标签，支持领域专精研究
- 输出“高概率 carry 候选”与“速度交易观察列表”

输出：

- 市场总览看板
- 候选机会列表
- 每日研究清单

### 9.2 Layer B: Research Copilot

目标：
让系统辅助做规则、新闻、上下文研究，但不直接下单。

能力：

- 抽取 resolution 相关字段
- 汇总相关新闻和时间线
- 生成研究摘要
- 给候选打“值得人工复核”的优先级
- 生成“伪确定性风险”说明，避免把高概率事件误判成无风险机会
- 为特定领域维护历史案例和研究 checklist

说明：

- 首版中 LLM 只用于候选市场，不用于全量市场
- 需要保留 prompt、输入摘要、输出结论，便于复盘

### 9.3 Layer C: Alerting

目标：
当市场满足条件时，自动发提醒，不直接交易。

能力：

- 价格异动提醒
- spread 异常提醒
- 临近结算提醒
- 规则高风险提醒
- rebate / holding reward 候选提醒

通知渠道：

- Telegram
- 飞书
- 邮件
- 本地日志 / dashboard

### 9.4 Layer D: Paper Trading

目标：
在不承担真实资金风险的情况下验证策略。

能力：

- 模拟挂单 / 吃单
- 模拟成交概率
- 计算滑点、费用、持仓、PnL、回撤
- 支持策略参数回放
- 支持不同策略类别的持有周期建模，例如秒级、小时级、日级

输出：

- 策略收益曲线
- 交易记录
- 策略对比报告

### 9.5 Layer E: Live Trading

目标：
在严格风控下接入真实交易。

能力：

- 策略执行引擎
- 下单、撤单、改单
- 仓位与风险限制
- kill switch
- 异常告警

限制：

- 首版只支持少量白名单策略
- 默认从低频、低市场数、低资金规模开始

## 10. Functional Requirements

### 10.1 Data Ingestion

系统必须支持：

- 从 Polymarket 公共接口拉取市场和事件数据
- 从 CLOB 读取 orderbook 和实时价格
- 持久化市场快照、成交数据、候选信号和研究结论
- 定义统一市场 ID 和事件 ID 关联关系

### 10.2 Opportunity Scanner

系统必须支持以下扫描器：

- Structural Scanner
- Rule Risk Scanner
- High-Probability Carry Scanner
- Domain Focus Scanner
- Speed Watchlist Scanner
- Spread / Liquidity Scanner
- Rebate Candidate Scanner
- Holding Reward Candidate Scanner
- Event Deadline Scanner

每个 scanner 需要输出：

- signal_id
- signal_type
- market_id / event_id
- detection_time
- severity / confidence
- explanation
- raw metrics

对于需要进入交易核心评估的候选，scanner 还应支持生成：

- session_context_key
- prior_eligible
- signal_family
- sentinel_eligible

### 10.3 Research Workflow

系统必须支持：

- 对候选市场生成研究任务
- 保存研究摘要、来源和结论
- 区分事实、假设、观点
- 标记是否建议人工复核
- 对高概率市场标记“真确定性 / 伪确定性”判断
- 支持按赛道沉淀历史案例、失败案例和研究模板

### 10.4 Alerting

系统必须支持：

- 基于阈值触发提醒
- 去重与冷却时间
- 高优先级信号重复提醒
- 失败重试

### 10.5 Paper Trading

系统必须支持：

- 模拟 taker 执行
- 模拟 maker 挂单与成交
- 支持费用模型
- 支持资金占用与库存约束
- 支持 session prior、signal voting、sentinel dampening 的逐层回放
- 支持逐笔记录“方向判断正确但 sizing 错误”与“方向错误但 defense 减损成功”

### 10.6 Live Execution

系统必须支持：

- 手动批准模式
- 半自动模式
- 全自动模式

首版默认：

- 只开放手动批准模式
- 自动模式需显式开关与白名单策略
- 速度交易类策略默认禁用真实自动执行

对于接入三层交易核心的策略，系统必须区分：

- direction engine：负责决定 YES / NO 偏向
- sizing engine：负责决定 full size、reduced size 或 skip

### 10.7 Portfolio Construction

系统应支持基础组合管理，而不是把每个市场当成孤立交易。

最低要求：

- 标记头寸相关性和事件簇
- 跟踪短期 / 中期 / 长期仓位分布
- 预留现金仓位用于新机会
- 限制单一事件或高度相关事件的总敞口

建议的初始风险原则：

- 同时持有 5 到 12 个低相关头寸
- 保留 20% 到 40% 现金或等价流动性
- 单笔交易风险敞口不超过总资金的 5% 到 10%
- 单一事件或高度相关事件群总敞口不超过 40%

## 11. Non-Functional Requirements

### 11.1 Reliability

- 数据采集失败可重试
- 网络中断不会导致系统静默失效
- 下单异常必须可见
- 不同策略类型支持不同延迟等级，不默认以高频为目标

### 11.2 Auditability

- 所有信号、决策、下单请求、响应、撤单都必须记录
- 研究结论与最终交易结果可追溯

### 11.3 Cost Control

- 默认不开启全量 LLM 分析
- 记录每次模型调用的 token 使用量和成本
- 支持按策略、按日期查看 token 成本

### 11.4 Security

- API key、私钥、passphrase 采用环境变量或密钥管理
- 私钥不进入日志
- 实盘环境和研究环境隔离

## 12. Token Cost Strategy

### 12.1 Principle

LLM 不是默认数据引擎，而是“候选分析器”。

### 12.2 Cost Tiers

- Tier 0: 纯规则扫描，不调用 LLM
- Tier 1: 只对高优先级候选做 LLM 摘要
- Tier 2: 对少数复杂规则市场做深度研究
- Tier 3: 事件驱动时临时提高调用频率

### 12.3 Budget Targets

- Research-only 阶段：目标 0 到 10 美元 / 月
- Scanner + Copilot 阶段：目标 5 到 30 美元 / 月
- High-frequency research 阶段：超过预算需显式告警

### 12.4 Optimization Rules

- 优先缓存市场元信息和规则摘要
- 相同市场不重复全文分析
- 同类市场复用模板 prompt
- 先规则筛选，后 LLM 精筛

## 13. Risk Management

### 13.1 Trading Risks

- 规则理解错误
- 将高概率事件误判为无风险事件
- 市场流动性不足
- 滑点和吃单成本
- 假信号与过拟合
- 自动执行误触发
- 领域外交易导致错误自信
- 速度交易类策略的延迟劣势

### 13.2 Operational Risks

- API 限流
- 数据延迟
- WebSocket 中断
- 时钟不同步
- 日志缺失导致无法复盘

### 13.3 Compliance Risks

- 地域限制
- 平台使用规则变化
- 市场类型费用或奖励政策调整

### 13.4 Required Controls

- 每日总亏损阈值
- 单市场最大仓位
- 单事件最大敞口
- 最大同时挂单数
- kill switch
- 只在白名单策略上启用自动实盘
- 高概率 carry 策略必须有“伪确定性”排除规则
- 速度交易策略仅在专门环境中评估，不进入默认实盘白名单

## 14. Proposed System Architecture

### 14.1 Core Modules

- `collector`
  - 拉取市场、事件、orderbook、成交和持仓数据
- `normalizer`
  - 清洗并标准化数据结构
- `signal-engine`
  - 运行各类 scanner，产出 signal
- `research-engine`
  - 对候选市场做规则和信息分析
- `alert-engine`
  - 管理通知与去重
- `paper-engine`
  - 模拟交易和回测
- `execution-engine`
  - 实盘下单与风控
- `risk-engine`
  - 管理仓位、敞口、止损和 kill switch
- `storage`
  - 保存市场数据、信号、研究、成交和系统日志
- `dashboard`
  - 展示机会、研究结论、策略表现和风险状态

### 14.1A Trading Core Three-Layer Model

在交易核心内部，建议参考外部实战研究中的“三层结构”，但仅作为 `strategy-engine` 的内部设计，不替代整个产品架构。

#### Layer 1: Session Memory / Prior Layer

职责：

- 在任何实时信号触发前，为当前 session 生成一个初始方向偏置
- 从历史已完成 session 中寻找“与当前 session 相似”的样本
- 计算 YES / NO 的历史胜率，并将其转换为 prior

典型输入：

- 最近 30+ 已完成 session
- 当前 session 的 regime、波动、时间分桶、盘口轮廓
- 外部参考价格偏离
- 历史 session outcome

说明：

- 这层输出的是 directional prior，而不是最终交易决策
- 如果历史上“长得像现在”的 session 中 YES 胜率更高，系统可以在开局就轻度偏向 YES
- 这层的价值不在预测全部未来，而在于给实时投票一个合理起点

输出：

- prior_side
- prior_yes_win_rate
- prior_confidence
- session_similarity_bucket

#### Layer 2: Real-Time Signal Voting Layer

职责：

- 每秒或固定频率运行 8 到 12 条独立规则
- 每条规则分别对 YES / NO 投票，并给出置信度
- 通过加权投票聚合为最终方向和实时置信度

示例：

- reference price 偏离 -> YES / NO vote
- Binance momentum -> YES / NO vote
- CVD / order flow -> YES / NO vote
- market microstructure rule -> YES / NO vote
- carry / rule / structural 条件 -> YES / NO vote

输出：

- entry_candidate
- side
- vote_breakdown
- weighted_direction
- weighted_confidence
- required_latency

说明：

- 这层决定方向，但不直接决定最终仓位大小
- 每条规则都必须可解释，能回溯其 vote、方向和分数
- 首版优先支持规则型 signal，而不是黑箱模型

#### Layer 3: Defense Dampening / Sentinel Layer

职责：

- 不负责挑方向，而是决定应不应该下、下多大
- 对 signal layer 的方向结果进行风险缩放、放大或直接跳过
- 在下单前完成主要的 sizing 决策，在下单后继续负责防守和减损

能力：

- cvd agreement check
- oracle / PTB spread assessment
- time-left risk assessment
- session choppiness / reversal count assessment
- entry profit margin assessment
- composite risk score
- position dampening or skip
- partial fill recovery
- breakeven recovery
- inventory rebalance
- profit lock
- time-based exit
- emergency cancel / kill switch

说明：

- 这层的核心思想是：signal picks direction, sentinel picks size
- 一个 65% 置信度的方向信号，不等于 65% 仓位
- 如果 CVD 不同意、session 很 choppy、利润空间太薄，这层应显著 dampen，甚至直接 skip
- 如果五类风险因子都健康，这层也可以在上限约束内放大仓位
- 我们的系统应将每个新的防守规则记录为“由哪类失败样本触发新增”

输出：

- composite_risk_score
- dampener
- final_position_size
- skip_reason

#### Product Decision

- 采纳三层模型，作为 strategy-engine 内部标准
- 不把它上升为整个系统唯一架构
- 研究、提醒、回测、成本和审计模块仍然保持独立
- 对三层的具体职责做严格切分：
  - `Memory` 负责 prior
  - `Signals` 负责 direction
  - `Sentinel` 负责 sizing
- defense alpha 优先级高于 offense complexity，先把 sizing 与 skip 做对，再谈更多信号

### 14.2 Suggested Tech Stack

- Application Runtime:
  - Python 为主，必要时补 Node.js 工具
- Backend API:
  - FastAPI
- Data Store:
  - SQLite + Parquet 用于早期单机开发
  - PostgreSQL 用于长期演进
- Data Processing:
  - Polars / Pandas
- Market Data:
  - REST + WebSocket
- Task Scheduling:
  - cron / launchd 起步
  - 后续可演进到 Celery / Redis
- Dashboard:
  - Phase 1: Streamlit 或轻量内部 dashboard
  - Phase 2: Next.js 正式管理后台
- Deployment:
  - 本地开发 + Mac mini 常驻任务
  - 如需远程常驻，可迁移到 VPS

### 14.3 Tech Stack Decision Rationale

当前推荐的技术方案是有意“克制”的，原因如下：

- Python 最适合当前阶段
  - 数据抓取、特征计算、回测、策略实验和自动化脚本都更顺手
- FastAPI 适合作为统一服务层
  - 可以同时服务 scanner、research、dashboard 和后续手动审批接口
- SQLite + Parquet 是 Phase 1 最合理的起点
  - 简单、稳定、可迁移，足够支撑研究和 paper trading
- PostgreSQL 适合在策略稳定后再引入
  - 尤其适合多表关联、审计日志和并发访问
- Redis 不是首日必需
  - 只有在实时缓存、事件队列、提醒去重复杂度提升后才值得加入
- 不建议一开始就重投入前端
  - 先把数据流、prior、voting、sentinel 跑通，比先做复杂管理界面更重要

### 14.4 Storage Policy: External SSD First

本项目要求默认将所有核心存储放在外置 SSD，而不是 Mac mini 内置盘。

当前工作目录已经位于外置盘：

- `/Volumes/Sakura SSD GM7 M2 2TB Media/Project/Sakura-Polymarket`

强制要求：

- 代码仓库放在外置 SSD
- SQLite / PostgreSQL 数据文件放在外置 SSD
- Parquet / CSV / 导出报告放在外置 SSD
- 日志、回测结果、缓存、临时研究产物放在外置 SSD
- 如后续需要模型缓存或附件，也优先放在外置 SSD

建议目录约定：

```text
/Volumes/Sakura SSD GM7 M2 2TB Media/Project/Sakura-Polymarket/
├── data/
│   ├── raw/
│   ├── curated/
│   ├── parquet/
│   └── sqlite/
├── runtime/
│   ├── logs/
│   ├── cache/
│   ├── tmp/
│   └── exports/
├── reports/
└── docs/
```

环境变量建议：

- `POLY_ROOT=/Volumes/Sakura SSD GM7 M2 2TB Media/Project/Sakura-Polymarket`
- `POLY_DATA_DIR=$POLY_ROOT/data`
- `POLY_RUNTIME_DIR=$POLY_ROOT/runtime`
- `POLY_DB_PATH=$POLY_DATA_DIR/sqlite/polymarket.db`

### 14.5 Storage Implementation Constraints

为了避免数据偷偷写回系统盘，实施时应遵守以下约束：

- Phase 1 默认不用 Docker Desktop 作为主运行方式
  - Docker Desktop 的默认数据目录通常不在项目盘里，容易落到系统盘
- 优先使用本地 Python 进程、`uv` 虚拟环境、cron / launchd
- 如果未来引入 PostgreSQL，必须显式指定 data directory 到外置 SSD
- 如果未来引入 Redis，必须显式指定持久化目录到外置 SSD
- 所有日志路径、导出路径、临时目录都必须通过环境变量集中配置
- 禁止在代码里写死 `~/Library`, `/var`, `/tmp` 作为长期持久化目录

### 14.6 Dashboard Recommendation

系统最终应该有 Web 管理页面，但分阶段更合理：

- Phase 1:
  - 使用 Streamlit 或轻量内部 dashboard
  - 目标是查看市场、信号、prior、votes、sentinel 结果和 paper trading 报告
- Phase 2:
  - 使用 Next.js + FastAPI
  - 目标是提供正式管理界面、策略配置、手动审批、风险面板和审计视图

结论：

- `web 页面应该有`
- `但不应该先于核心交易与研究链路`
- `先轻量，后正式`

## 15. Data Model

首版建议至少定义以下实体：

- `markets`
- `events`
- `orderbook_snapshots`
- `trades`
- `signals`
- `research_notes`
- `alerts`
- `paper_orders`
- `paper_fills`
- `live_orders`
- `positions`
- `strategy_runs`
- `token_usage`
- `session_priors`
- `signal_votes`
- `sentinel_scores`
- `market_state_features`
- `strategy_guards`
- `execution_incidents`

## 16. User Flows

### 16.1 Research Flow

1. 系统定时抓取市场数据
2. scanner 发现异常市场
3. 系统生成候选研究任务
4. 研究模块输出摘要和风险说明
5. 用户决定忽略、继续观察或下单

### 16.2 Alert Flow

1. 市场满足触发条件
2. 系统生成信号
3. alert-engine 去重并发送提醒
4. 用户点击查看市场详情和研究摘要

### 16.3 Paper Trading Flow

1. 选择策略与参数
2. 选择市场池
3. memory layer 从历史 session 生成 prior
4. signal layer 逐条规则投票并聚合方向
5. sentinel layer 计算 risk score、dampener 和最终仓位
6. execution simulator 模拟成交与持仓变化
7. defense / recovery 处理成交后路径
8. 生成策略报告

### 16.4 Live Trading Flow

1. memory layer 更新当前 session prior
2. signal layer 实时投票并输出方向
3. sentinel layer 计算最终 sizing 或 skip
4. risk-engine 检查额度与全局风控
5. execution-engine 下单
6. defense / recovery 处理部分成交、反转或 profit lock
7. 订单状态监控与人工干预

## 17. Rollout Plan

### Phase 1: Research Foundation

目标：

- 完成市场采集和基础 scanner
- 建立本地数据存储
- 输出结构性机会清单
- 输出高概率 carry 候选和领域标签

交付：

- 市场抓取脚本
- 本地数据库
- scanner CLI
- 每日研究报告

### Phase 2: Research Copilot + Alerts

目标：

- 对候选市场生成可读研究摘要
- 接入提醒通道
- 建立“伪确定性识别”和赛道研究模板

交付：

- research pipeline
- alert bot
- token 成本统计

### Phase 3: Paper Trading

目标：

- 验证至少 2 到 3 个策略
- 输出收益、回撤、滑点数据
- 验证三层结构中 defense layer 的有效性

交付：

- 模拟成交引擎
- 策略参数配置
- 回测与复盘报告
- 基础组合管理和仓位分层报告
- guard / recovery effectiveness report

### Phase 4: Controlled Live Trading

目标：

- 小规模白名单实盘
- 所有决策可审计

交付：

- execution-engine
- risk controls
- kill switch

## 18. MVP Definition

MVP 不包含全自动实盘，定义如下：

- 能抓取 Polymarket 市场与基础盘口数据
- 能识别至少三类信号：
  - 结构性机会
  - 高概率 carry 候选
  - 流动性 / spread 机会
  - 临近结算 / 规则风险机会
- 能生成候选列表和解释
- 能发送提醒
- 能记录研究结论
- 不包含速度交易类自动执行

## 19. Open Questions

- 首版主要关注哪些市场分类：政治、加密、体育还是全量？
- 是否优先做 scanner CLI，还是先做 dashboard？
- 首版提醒通道优先级：Telegram、飞书还是邮件？
- paper trading 的成交模型精度要求到什么程度？
- 是否需要在仓库中同步维护研究模板与 checklist？
- 是否把“高概率 carry”作为首个 paper trading 策略，而不是先做做市？
- 首个领域专精赛道选择政治、crypto 还是 mention 市场？

## 20. Initial Implementation Recommendation

建议的第一批落地顺序：

1. 建立 `data/`, `scripts/`, `docs/`, `reports/` 基础目录
2. 实现市场抓取和本地存储
3. 实现 structural scanner
4. 实现 alert pipeline
5. 再引入 research copilot

原因：

- 这条路径最省 token
- 最容易验证价值
- 最快形成可以复用的数据资产
- 能在不碰实盘前积累足够上下文

## 21. Reference Assumptions

以下前提基于 2026-03-19 可见的官方公开资料，后续实现前需要再次核验：

- Polymarket 提供公共市场与数据接口
- CLOB 支持认证后下单和订单管理
- maker rebate / liquidity rewards 机制存在，但细则可能变化
- holding rewards 当前适用于部分 eligible events，且年化与范围可调整
- 部分市场存在 fee-enabled 规则，尤其是 crypto 等新市场

## 22A. External Research Notes

除官方资料外，本文档还吸收了一篇围绕 Polymarket 盈利模型的外部研究文章中的方法框架，用于补充“策略分类、评估标准、仓位管理和组合建议”。其中最有价值的补充包括：

- 不只看套利，也要覆盖高概率 carry、领域专精和速度交易
- 策略评估必须同时看收益、稳定性、可复制性和容量
- 系统设计要区分“适合自动化的策略”和“只适合研究辅助的策略”
- 高概率策略的核心不是找 95% 的票，而是识别伪确定性

另一个值得采纳的外部方法，是将交易核心拆为“状态/记忆层、信号层、防守/恢复层”。这一点并非来自 Polymarket 官方，而是来自交易者实战经验的工程抽象，尤其适合短周期和执行误差较多的策略。

说明：

- 这部分属于外部研究框架，不是 Polymarket 官方规则
- 其中涉及收益数字和个案表现的内容应视为待验证样本，而不是系统设计的事实前提

## 23. Summary

这不是一个“单一交易策略”的 PRD，而是一个“研究到执行”的系统 PRD。它的核心目标不是追逐社交媒体上的暴利故事，而是建立一个：

- 能持续发现机会
- 能解释机会
- 能控制成本
- 能验证策略
- 能逐步实盘

的长期研究与自动化框架。
