---
type: analysis
tags:
  - type/analysis
---

# V2 度量报告（AI Compute 压测库）

> 生成：2026-08-26 · 指标定义见 docs/07-schema-v2.md §16

## 关键指标

| 指标 | 数值 | 说明 |
|------|------|------|
| 实体数 | 40 节点 | 公司 15 + 产品 11 + 设施 5 + 事件 4 + 索引/模板/审计 |
| verified claims 数 | 92 条关系 | 关系行 = claim 的 Obsidian 表达 |
| 带来源 claims | 90/92（98%） | 行内来源 URL 引用率（自动继承补全后） |
| 独立 evidence 数 | 13 个 URL | 不同来源链接去重 |
| 有效更新事件数 | 4 | 2024-03 Blackwell / 2025-03 HBM3E / 2025 CoWoS 扩产 / 2026-01 沪电 |
| stale/待核实 claims | 3 | 昇腾代工来源、CoWoS 产能数字等（报道口径） |

## 说明

- 本库为 Schema V2 压测（AI Compute Supply Chain 子集 40 节点），验证 claim-evidence-event 模型在 Obsidian 的落地
- claim 在关系行内联表达：`（claim: SUPPORTED, 0.85）（来源: URL）`
- 待核实项如实标注，不伪装 VERIFIED
- 数据为公开渠道采集（2026-08-26），部分为报道口径（置信度 <0.8）

## 与主库（enterprise-intel-demo）的关系

- 主库：v1 模型（159 节点半导体全链条）
- 本库：v2 模型压测（AI 芯片窄链条，产品/设施/事件层更完整）
- 后续：v2 模型验证后，主库按需迁移（docs/07-schema-v2.md §15 触发条件）
