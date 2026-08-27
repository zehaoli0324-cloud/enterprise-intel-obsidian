# Schema V2：从 Obsidian 情报库升级为"可售卖的数据资产"

> 版本：v2.1-draft · 2026-08-26
> 定位：定义"谁是谁、和谁发生什么关系、凭什么可信、何时发生/有无变化"四层数据模型。
> 配套：v1 已在 Obsidian 落地（159 实体、718 关系、Evidence 打标、Eval v1）；V2 压测子库为 AI Compute Supply Chain。

---

## 0. 核心原则

**每一条持续采集的数据必须回答三个问题 + 一个时间维度：**
> 它是谁？它和谁发生了什么关系？这件事凭什么可信？——以及：什么时候发生、后来有没有变化？

**数据价值筛选公式：**
```
Commercial Data Value ≈ Scarcity × Freshness × Decision Relevance × Verifiability × Structural Reusability
```

**护城河（长期积累的四样东西）：**
```
1. Historical claims   2. Evidence provenance   3. Temporal changes   4. Human/agent corrections
```

---

## 1. 七个核心对象

```
ENTITY   公司/机构/人物
PRODUCT  产品/组件（H100 / HBM3E / CoWoS）
FACILITY 工厂/产线（Fab、封装厂、数据中心）
RELATION canonical 关系（本体，不含真伪判断）
CLAIM    带状态和置信度的主张（系统灵魂）
EVIDENCE 支撑 claim 的证据（含双分数）
EVENT    时间戳事件（持续更新真正生产的东西）
```

**关键：RELATION 不是事实本身。** 真实性由 CLAIM 控制：
```text
RELATION: 富瀚微 SUPPLIES 海康威视        （canonical）
CLAIM:    subject=富瀚微 predicate=SUPPLIES object=海康威视
          claim_status=supported confidence=0.92 valid_from=2023-01
EVIDENCE: 年报(2025) + 招股书 + 行业媒体
```

---

## 2. ENTITY

```yaml
entity_id: company_cn_0000142     # 永远用 ID，不用名字
type: company                     # company / person / investor / org
canonical_name: 富瀚微
aliases: [上海富瀚微电子股份有限公司, Fullhan, Fullhan Microelectronics]
ticker: 300613.SZ
country: CN
region: Shanghai
industry_primary: semiconductor
industry_secondary: [video_soc, isp, automotive_chip]
```

**实体归一（Entity Resolution）红线**：台积电 / TSMC / Taiwan Semiconductor / 台湾积体电路 / 2330.TW / TSM 是同一实体。必须 canonical_entity_id + aliases，禁止用名字当 ID。

## 3. PRODUCT（最可能最值钱的一层）

```yaml
product_id: product_000031
name: H200
company_id: nvidia
category: AI Accelerator
generation: Hopper
launch_date: 2023-11
status: production
process_node: 4N
memory: {type: HBM3E, capacity: 141GB}
packaging: {technology: CoWoS}
target_market: [datacenter, AI_training, AI_inference]
```

**Supply Graph 穿透链（从 Company→Company 升级为）：**
```text
Company → makes → Product → uses → Component → made_by → Company
NVIDIA → H200 → HBM3E → SK海力士 → CoWoS → 台积电
```

## 4. FACILITY（工厂独立）

```yaml
facility_id: fab_tsmc_coWoS_tn
owner: TSMC
type: advanced_packaging_fab
location: {country: TW, city: Tainan}
process_capability: [CoWoS-S, CoWoS-L]
estimated_capacity: {value: xxx, unit: wafers_per_month}
status: active
```

设施类型：wafer_fab / packaging_plant / OSAT / material_plant / server_factory / data_center / R&D_center。
**为什么值钱**：风险不是"TSMC 出问题"，而是"TSMC 某个 Fab 某条产线出问题"。

## 5. RELATION（canonical，不含真伪）

```yaml
relation_id: R000142
subject: fullhan
predicate: SUPPLIES    # SUPPLIES/CUSTOMER_OF/MANUFACTURED_BY/PACKAGED_BY/USES_PROCESS/INVESTED_IN/OWNS/COMPETES_WITH/STRATEGIC_PARTNER/REPLACEMENT_FOR
object: hikvision
product: [ISP, video_soc]
```

## 6. CLAIM（系统灵魂）

```yaml
claim_id: C00019420
subject: fullhan
predicate: SUPPLIES
object: hikvision
qualifiers: {product: video surveillance chips}
valid_from: 2023-01
valid_to: null
claim_status: supported
confidence: 0.92
last_verified_at: 2026-08-25
```

**claim_status 八态：**
```
SUPPORTED / VERIFIED / INFERRED / DISPUTED / STALE / SUPERSEDED / REJECTED / UNKNOWN
```

**系统回答方式**（从"富瀚微是海康供应商"升级为）：
> "目前有三项独立证据支持该关系，最近一次验证 2026-08-25，置信度 0.92。"

## 7. EVIDENCE（数据护城河）

```yaml
evidence_id: E00088319
claim_id: C00019420
source_type: annual_report        # annual_report/prospectus/announcement/media/discussion
source_title: 2025 Annual Report
publisher: Fullhan
publication_date: 2026-04-21
retrieved_at: 2026-08-27
source_url: xxx
evidence_text: "第一大客户..."
evidence_type: direct
supports_claim: true
source_quality: 0.95     # 来源可信度（Reuters 高 ≠ 证据强）
evidence_strength: 0.96  # 该条证据对 claim 的支撑强度（独立分数！）
```

**双分数分离**：`source_quality`（来源权威度）与 `evidence_strength`（证据对结论的支撑强度）必须分开——
Reuters 是高质量来源，但一篇"疑似进入供应链"的报道证据强度只有 0.55。

**Evidence Level（商业验证强度）：**
```
Level A  客户官方采购/招标/公告
Level B  供应商年报披露
Level C  客户官网 case study
Level D  高可信行业媒体
Level E  传闻
```

## 8. EVENT（持续更新真正生产的东西）

Agent 不"更新数据库"，而是**每天生产 New Events**：

```yaml
event_id: EVT202608270031
type: NEW_CUSTOMER        # NEW_CUSTOMER/SUPPLIER_CHANGED/CAPACITY_EXPANSION/EXECUTIVE_LEFT/PRICE_CUT/NEW_PRODUCT/FUNDING/PATENT_CLUSTER_SURGE/TENDER/REGULATION
entity: {supplier: company_a, customer: company_b}
product: product_x
event_date: 2026-08-26
detected_at: 2026-08-27
confidence: 0.88
evidence: [E19291, E19294]
impact: {commercial: high, supply_chain: medium}
```

**数据流：**
```text
New Evidence → Event detection → Claim update → Relation status change
Web/PDF/API/公告/年报 → Source doc → Evidence extraction → Claim → Entity resolution → Event → Relation/State
```

## 9. 第一批 50 字段（按商业价值分组）

### A. 公司基础识别 8 个（只为 entity resolution，不是产品核心）
canonical_name / aliases / ticker / country / headquarters / company_type / industry_primary / industry_tags
**不要花人力采**：注册资本、法人身份、工商变更流水（企查查战场）。

### B. 产品与技术 10 个（第一组高价值）
product_name / product_category / product_generation / product_status / launch_date / technology_node / core_technology / target_application / key_specification / **replacement_products**
> replacement_products 极值钱："谁可以替代谁"直接服务采购决策。

### C. 供应链 12 个（重点积累）
supplier / customer / supplied_product / supplied_component / supply_chain_tier / supply_status / supply_start_date / supply_end_date / **estimated_share** / **estimated_revenue_exposure** / **single_source_dependency** / **alternative_supplier**
> 后四个最贵：不只是"谁供货"，而是"这条供应关系有多重要"。

### D. 制造能力 8 个（2026 半导体尤其值钱）
facility / facility_location / manufacturing_process / capacity / capacity_unit / utilization / capacity_expansion / expected_online_date
> "谁有什么产能、什么时候释放、被谁预订"本身就是情报（HBM/先进封装/先进制程 capacity 是 2026 瓶颈）。

### E. 商业验证 6 个（强烈建议建立状态链）
design_win / customer_validation / qualification_status / shipment_status / shipment_volume / commercialization_stage

**commercialization 状态链**（区分被混用的词）：
```
rumor → sample → testing → qualified → design win → small-volume shipment → mass production
```
"导入/合作/进入供应链/拿到订单/量产"根本不是一回事，结构化区分极有价值。

### F. 证据与时间 6 个（与普通爬虫库拉开差距）
source_type / source_quality / evidence_strength / claim_confidence / valid_time / **last_verified_at**
> 商业数据库最可怕的问题不是"错"，而是"曾经对，现在已经错了"。

### 隐藏字段（内部保存，不展示）
```
extraction_method: manual / rule_based / LLM_extracted / LLM_inferred / cross_source_inferred
review_status: unreviewed / agent_reviewed / human_reviewed
```

## 10. 禁止采集清单（比采什么更重要）

* 企业简介、CEO 简历全文、新闻摘要、企业文化、ESG 空泛描述
* 官网产品宣传文案、无法证明的"合作伙伴"、模糊市场份额描述
* 普通工商字段、全量专利、全量新闻

> 每进入一条数据前问：**它有没有改变我们对一家公司/产品/供应关系/未来事件的判断？** 没有，就不存。
> 数据库不应成为"AI 自动生成的互联网摘要垃圾场"。

## 11. 来源分级（Source Hierarchy）

| Tier | 类型 | 用途 |
|------|------|------|
| Tier 1 一级证据 | 交易所公告/年报/招股书/公司公告/政府文件/采购公告/招投标/监管文件 | 直接写 VERIFIED |
| Tier 2 强二级 | Reuters/Bloomberg/Nikkei/FT/专业行业媒体/机构研究 | 写 SUPPORTED |
| Tier 3 线索源 | 公众号/雪球/论坛/LinkedIn/招聘信息/传闻 | 只做 lead：→ 搜索佐证 → promote |

**Tier 3 不可直接写 VERIFIED relation，必须 promote 流程。**

## 12. 产品封装（四种，按可卖性排序）

1. **Company Intelligence Card**：单公司供应链卡（产品/客户/供应商/产能依赖/替代/事件/证据）——单份报告出售
2. **Industry Map**：产业链图谱（GPU→HBM→CoWoS→基板→PCB→光模块→服务器→液冷），每个节点可点入——VC/PE/咨询/券商
3. **Watchlist（最容易的付费 MVP）**：用户选 20 家公司，每日/每周推送"真正改变商业判断的事件"（Change Detection，不是新闻汇总）
4. **API / Data Feed**：`GET /company/nvidia/suppliers` 返回含 confidence/valid_from/last_verified/evidence 的结构化数据——长期最值钱，可接 CRM/采购/投资模型/内部 Agent

## 13. 第一阶段战场：AI Compute Supply Chain

**只做一个窄而深的行业**：约 100-200 公司、300-500 产品/组件、100 工厂、2000-5000 claims、5000-10000 evidence。

重点环节：AI accelerator / HBM / advanced packaging / foundry / substrate / PCB / optical module / server ODM / power / liquid cooling。

理由：变化快、信息碎片化、company-product-supplier 关系密集、capacity 极重要、投资者与产业方都关心、2026 年 HBM 与先进封装仍是瓶颈。

## 14. Obsidian 在 V2 的定位

> **数据库负责 truth，Obsidian 负责 cognition。**

每个 company 节点自动 materialize（不真正存所有数据，而是渲染）：
```markdown
# NVIDIA
## Products        [[H200]] [[B200]] [[GB200]]
## Critical suppliers  [[TSMC]] [[SK Hynix]]
## Manufacturing dependencies  ...
## Recent Events   2026-08-27 ... / 2026-08-21 ...
## Claims needing review  ⚠ C001928
## Conflicting evidence  ⚠ C008218
## Last refreshed  2026-08-27
```

## 15. 存储演进

| 阶段 | 存储 | 触发条件 |
|------|------|---------|
| v1 | Obsidian（markdown+YAML） | 已达成（159 实体） |
| v2a | + SQLite/DuckDB 镜像 | 实体 >5000 或复杂查询需求 |
| v2b | + 关系级 API | 外部消费方出现 |
| v3 | Postgres + 图查询 | 多用户/历史版本查询 |

## 16. V2 度量指标（不再看节点数）

```
verified claims 数          ← 经过验证的主张
独立 evidence 数            ← 支撑 claims 的证据条数
有效更新事件数              ← 每月/周新增 Events
stale-claim 检出率          ← 过期主张被识别/更新的比例
```
