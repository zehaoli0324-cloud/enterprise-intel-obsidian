# Schema V2：从"公司笔记库"到"产品级情报数据产品"

> 版本：v2.0-draft · 2026-08-26
> 定位：把 Obsidian 情报库从"公司-公司关系图"升级为"产品级、可验证、可演进的数据产品"的 schema 设计。
> 配套现状：v1 已在 Obsidian 落地（157 实体、718 关系、Evidence 打标、Eval v1），本文定义 v2 目标结构。

---

## 1. 设计原则

**筛选数据的唯一标准：**
```
Commercial Data Value ≈ Scarcity × Freshness × Decision Relevance × Verifiability × Structural Reusability
```

**五层数据图（v2 目标）：**
```
Entity Graph（谁是谁）
    +
Product Graph（造什么、含什么）
    +
Supply Graph（谁供谁、供多少）
    +
Evidence Graph（为什么相信）
    +
Temporal Event Graph（什么时候变了）
```

**行业选择：窄而深**。只做 AI 芯片 + 半导体产业链（全球约 500-2000 家重要公司做深），不做综合企业库（企查查红海）。

---

## 2. 七类核心表

### 2.1 Entity（实体：公司/人/投资机构）

| 字段 | 类型 | 必要性 | 说明 |
|------|------|--------|------|
| entity_id | string | 必填 | 统一 ID（CN_xxx / HK_xxx / US_xxx） |
| entity_type | enum | 必填 | company / person / investor / org |
| name_zh / name_en | string | 必填 | 双语名（Entity Resolution 基础） |
| aliases | list | 选填 | 别名（"台积电"/"TSMC"/"台湾积体电路"） |
| legal_id | string | 选填 | 统一社会信用代码 / LEI |
| industry / sub_industry | string | 必填 | 行业 + 细分 |
| region | string | 选填 | 注册地/运营地 |
| status | enum | 必填 | active / ipo_planned / ipo / delisted / bankrupt |
| founded / listed | date | 选填 | 成立/上市时间 |
| source_url / evidence | 证据字段 | 必填 | 见 2.5 |

### 2.2 Product（产品/货品）

| 字段 | 类型 | 说明 |
|------|------|------|
| product_id | string | P_xxx |
| name | string | 产品名 |
| category | enum | SoC / CIS / 存储 / 设备 / 代工服务 / 封测服务 / 材料… |
| specs | string | 规格（制程/容量/带宽） |
| manufacturers | list[entity_id] | 生产商（可多家） |
| customers | list[entity_id] | 已证实客户（见 Verified Customer Graph） |
| components | list[product_id] | 内含组件（产品→零部件穿透） |
| replaced_by / replaces | list | 替代关系（REPLACEMENT_FOR） |

**v2 最有壁垒的部分**：Company → Product → Component → Material → Factory 的穿透链。例：
```
NVIDIA H200 → HBM3e → SK海力士 → CoWoS → 台积电 → ABF基板 → Ibiden/揖斐电
```

### 2.3 Factory（工厂/产能）

| 字段 | 类型 | 说明 |
|------|------|------|
| factory_id / name / location | - | 工厂 |
| operator | entity_id | 运营方 |
| process_nodes | list | 制程节点（28nm/14nm/7nm…） |
| capacity | int | 产能（万片/月） |
| utilization | float | 稼动率 |
| products | list[product_id] | 产出 |

### 2.4 Relation（关系：边）

| 字段 | 类型 | 说明 |
|------|------|------|
| from_id / to_id | entity_id | 两端 |
| relation_type | enum | SUPPLIES / CUSTOMER_OF / MANUFACTURED_BY / PACKAGED_BY / USES_PROCESS / INVESTED_IN / OWNS / COMPETES_WITH / STRATEGIC_PARTNER / REPLACEMENT_FOR |
| product_id | 选填 | 关系对应的货品 |
| start_date / end_date | date | 有效期（**temporal 核心**） |
| amount / share | 数值 | 金额/份额（有则填） |
| status | enum | active / superseded / pending |
| confidence | float | 0-1（由 Evidence 聚合） |

### 2.5 Claim + Evidence（主张 + 证据：本 schema 的灵魂）

```
CLAIM:
  富瀚微 → SUPPLIES → 海康威视
  product = ISP/SoC
  confidence = 0.93

EVIDENCE:
  E1 年报（2025）          → Level A
  E2 招股书                → Level A
  E3 行业媒体（2025-07）    → Level D
  LAST_VERIFIED: 2026-08-25
  CONTRADICTION: none
```

| Claim 字段 | 说明 |
|-----------|------|
| claim_id | CL_xxx |
| subject / predicate / object | 主-谓-宾 |
| confidence | 0-1（由证据加权） |
| status | VERIFIED / SUPPORTED / INFERRED / CONFLICTING / STALE / UNKNOWN |

| Evidence 字段 | 说明 |
|--------------|------|
| source_url / source_type | 来源 + 类型（年报/公告/招股书/媒体/讨论） |
| publication_date / retrieved_at | 发布/采集时间 |
| evidence_span | 引用原文片段 |
| direct / inferred | 直接证据 or 推断 |
| extractor | 抽取方式（人工/规则/LLM） |

**Evidence Level（商业验证强度）**：
```
Level A  客户官方采购/招标/公告
Level B  供应商年报披露
Level C  客户官网 case study
Level D  高可信行业媒体
Level E  传闻
```

### 2.6 Event（事件流：订阅制产品的核心）

| 字段 | 说明 |
|------|------|
| event_id / date | 事件 ID + 时间 |
| event_type | NEW_CUSTOMER / SUPPLIER_CHANGED / CAPACITY_EXPANSION / EXECUTIVE_LEFT / PRICE_CUT / NEW_PRODUCT / FUNDING / PATENT_CLUSTER_SURGE / TENDER / REGULATION |
| subject | 主体 entity_id |
| payload | 结构化详情（客户=Y、产品=Z、+30%…） |
| confidence / evidence | 同 2.5 |

**商业形态**：watchlist intelligence feed——"过去 24h 与你关注的公司相关的变化"（订阅制）。

### 2.7 存储演进（Obsidian 不是终点，是起点）

| 阶段 | 存储 | 触发条件 |
|------|------|---------|
| v1（现在） | Obsidian（markdown+YAML） | 已达成：157 实体、可读可审计 |
| v2a | + SQLite/DuckDB 镜像 | 实体 >5000 或需要复杂查询 |
| v2b | + 关系级 API | 外部消费方出现 |
| v3 | Postgres + 图查询 | 多用户/并发/历史版本查询 |

**原则**：Obsidian 永远保留为 human interface / analyst workspace；机器层做 canonical 数据；Obsidian 由机器层导出。

---

## 3. 半导体行业第一批字段清单（50 个）

### 3.1 值钱的字段（高 Scarcity × Decision Relevance）— 优先采

| # | 字段 | 所属表 | 为什么值钱 |
|---|------|--------|-----------|
| 1 | 前五大客户金额/占比 | Relation+Claim | 集中度=风险（海康占富瀚微 60%+） |
| 2 | 前五大供应商金额/占比 | Relation+Claim | 单一供应商依赖 |
| 3 | 关联交易金额及变化率 | Event | 兆易-长鑫 57 亿案例 |
| 4 | 产品-客户映射（哪款芯片卖给谁） | Product | 产品级穿透 |
| 5 | 晶圆代工来源（厂+制程节点） | Factory | 产能/国产化判断 |
| 6 | 产能/资本开支/扩产计划 | Factory+Event | 设备订单先行指标 |
| 7 | 大基金/国家队增减持 | Event | 政策信号 |
| 8 | 招标/中标金额 | Event | 订单硬证据 |
| 9 | 客户验证状态（设计导入/量产） | Claim | 前导指标 |
| 10 | 单一供应商风险评分 | 派生 | 直接可卖 |
| 11 | 出口管制/制裁影响 | Event | 政策风险 |
| 12 | 研发费用率/方向 | Entity | 技术路线判断 |
| 13 | 专利集群/技术节点 | Entity+Event | HBM/先进封装等布局 |
| 14 | 车规认证状态（AEC-Q100/ISO26262） | Product | 进入门槛 |
| 15 | 高管/核心团队变动 | Event | 公司治理信号 |

### 3.2 普通字段（有就用，不值得单独采）— 中优先级

| # | 字段 | 说明 |
|---|------|------|
| 16-25 | 注册资本/成立时间/注册地/员工数/营收/毛利率/净利率/市值/PE/股票代码 | 东财 API 一键可得 |
| 26 | 十大股东列表 | 已有 |
| 27-30 | 子公司/参股公司/分支机构/对外投资 | 工商数据，企查查红海 |

### 3.3 别浪费时间采的字段（低 Value 或已有替代）— 不做

| # | 字段 | 原因 |
|---|------|------|
| 31-35 | 法人姓名/联系电话/详细地址 | 低决策价值 + 隐私风险 |
| 36-40 | 官网新闻逐条归档 | 新闻摘要价值低，事件抽取才有价值 |
| 41-45 | 工商行政处罚/司法诉讼逐条 | 企查查/天眼查已覆盖，非差异化 |
| 46-50 | 通用 ESG 评分/信用评分 | 已有成熟供应商，不做 |

---

## 4. 与当前 Obsidian 实现的映射

| v2 表 | 现状 | 差距 |
|-------|------|------|
| Entity | ✅ 157 实体（company/person/investor） | 缺 aliases/entity_id 体系 |
| Product | ✅ 21 货品节点 | 缺 specs/components 穿透链 |
| Factory | ❌ 未建 | 新增大类（中芯/华虹/长鑫工厂+制程） |
| Relation | ✅ 718 边 + 置信度打标 | 缺 start/end_date、amount/share |
| Claim+Evidence | 🟡 Evidence 节点级 v1 | 缺关系级 evidence、Level A-E 分级 |
| Event | 🟡 2 个事件节点 | 缺事件流 schema + 监控 |
| 存储 | Obsidian | 按 2.7 演进 |

---

## 5. 结论

v2 的核心不是换存储，而是：
1. **产品级穿透**（Company→Product→Component→Factory）
2. **关系级证据**（Claim→Evidence[]→Confidence，Level A-E）
3. **时间维**（Relation 有效期 + Event 流）

这三件事做完，Obsidian 库就具备了"数据产品"的骨架；商业化形态 = 窄行业的 Verified Customer Graph + watchlist event feed。
