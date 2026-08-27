# Commercial Validation Proposal V1（落地版）

> 生成：2026-08-26 · 依据外部评审 + 本 repo 现状整理的下一阶段 roadmap
> 核心原则：**不堆 UI/节点，围绕 Detection / Evidence / Impact / Evaluation / User Value 建设**

---

## 1. 项目定位（叙事升级）

从"AI Agent × Obsidian 企业情报数据库"升级为：

> **Evidence-Grounded Continuous Enterprise Intelligence System**
>
> Continuously detects commercially significant changes in deep-tech supply chains, grounds each claim in traceable evidence, and propagates events through company–product–supplier networks to explain not only **what changed**, but **why it matters**.

产品价值链：
```
Public Information → Continuous Monitoring → Evidence Extraction → Claim/Event Detection
→ Temporal State Update → Impact Propagation → Decision Intelligence → Watchlist/Weekly/API
```

## 2. 五大能力缺口与状态

| Gap | 目标 | 当前状态 | 差距 |
|-----|------|---------|------|
| 1. Continuous Monitoring | 无人值守持续运行，14 天 run log | ✅ cron 已注册（每天 08:00，local 交付，写 07-监控日志/） | 等 2026-08-28 首跑，累计 run log |
| 2. Eval V2 | 从 Precision 到 Coverage（Recall/latency/noise） | 🟡 docs/08-eval-v2.md：Event Recall 67%、回溯 237-908 天 | Historical Replay 轻量版 + live 数据 |
| 3. Event → Impact | 从"发生了什么"到"这意味着什么" | ❌ 未实现 | Impact Graph v1（1st/2nd order） |
| 4. Significance Scoring | 降 false alert（目标 <10-20%） | ❌ 未实现 | rule-based 5 权重 + Eval 校准 |
| 5. User Validation | 10 outreach → 3 watchlist → 1 paid | ❌ 未启动 | docs/09 行动包待发 |

## 3. Eval V2 指标（目标值，非声称值）

| 指标 | 目标 | 测量 |
|------|------|------|
| Event Recall | ≥70%（窄域） | Known Event Set 20-50 条 |
| Evidence Coverage | ≥90% | 当前 98%（带源率） |
| Detection Latency | <24h | Historical Replay（回顾标注）+ live（cron 记录） |
| False Alert Rate | <20% | cron 运行 ≥2 周后统计 |

## 4. Impact Propagation 设计（v1 只做 1st/2nd order）

事件 schema 增加 impact 字段：
```yaml
impact:
  first_order:
    - entity: NVIDIA
      direction: positive
      mechanism: packaging_capacity
      confidence: 0.85        # SUPPORTED_IMPACT
  second_order:
    - entity: server_ODM
      direction: positive
      confidence: 0.72        # INFERRED_IMPACT
```

**四层分离（事实 vs 分析）**：
```
FACT                VERIFIED 级，来源可追溯
SUPPORTED_IMPACT    ≥0.7-0.85，多源/机理明确
INFERRED_IMPACT     0.5-0.7，单源/类推
SPECULATIVE_IMPACT  <0.5，明确标注"推测"
```

## 5. Event Significance Score（rule-based v1）

```
S = w1·Novelty + w2·CommercialImpact + w3·SupplyCriticality + w4·EvidenceStrength + w5·WatchlistRelevance
```

| 事件 | Score 示例 |
|------|-----------|
| 新增关键客户 | 0.91 → Alert |
| HBM 大规模扩产 | 0.88 → Alert |
| CEO 普通采访 | 0.23 → Store only |
| 官网文案更新 | 0.08 → Store only |

阈值：S≥0.75 Alert / 0.50-0.75 Weekly / <0.50 Store only。权重用 Eval 校准（跑 2 周后按 false alert 调）。

## 6. 用户验证（轻量版，压缩自 7 天计划）

Funnel：10 outreach → 5 reply → 3 watchlist → 2 repeated → 1 paid

信号分级：Weak（"不错"）→ Medium（"再加寒武纪"）→ Strong（"下周继续盯"）→ Commercial（"多少钱？"）→ Validation（付款）→ Strong Validation（续费）

Offer 统一：免费一周 watchlist（用户给 5-20 家公司，我们给每日变化+证据+置信度+为什么重要）。

## 7. 三个产品实验（不提前选型）

- Experiment A：Watchlist（盯公司）→ 验证 monitoring WTP
- Experiment B：Weekly Intelligence（产业链最重要变化）→ 验证 filtering WTP
- Experiment C：Deep Dive（如"梳理昇腾 910C 供应链"）→ 验证 research-as-service WTP

## 8. Commercial Gate V1（通过标准）

```
Technical: 7 天无人值守监控 ✓ / 事件检测 ✓ / claim diff ✓ / watchlist ✓ / impact v1 ✓
Eval:      ≥20 历史事件，Recall≥70%，Coverage≥90%，latency<24h，false alert<20%
User:      10 outreach，≥3 real watchlists，≥2 repeated usage
Commercial: ≥1 paid user 或 ≥1 paid custom research 或 ≥2 explicit WTP signals
最终：MRR > 0
```

## 9. 明确暂停（除非直接服务上述目标）

❌ UI 美化 / ❌ 更多 graph / ❌ 为节点数扩库 / ❌ SaaS 登录支付 / ❌ 手机 App / ❌ Neo4j / ❌ Vector DB / ❌ 大规模爬虫 infra / ❌ 复杂 multi-agent

## 10. 执行状态追踪

- [x] 2026-08-26：cron 公告监控注册（每天 08:00，local，写监控日志）
- [x] 2026-08-26：Eval v2 首版（docs/08，Event Recall 67%）
- [x] 2026-08-26：用户验证行动包（docs/09）
- [ ] 2026-08-28：cron 首跑 → run log #1
- [ ] Historical Replay：20 条已知事件回顾标注
- [ ] Impact v1：CoWoS 扩产事件做 1st/2nd order 示例
- [ ] 发 10 条用户验证消息
- [ ] 2 周后：false alert 统计 + significance 权重校准
