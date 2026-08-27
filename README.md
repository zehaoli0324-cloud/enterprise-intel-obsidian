# Enterprise Intel DB · 企业情报数据库

**Agent 自动构建、证据可追溯、持续更新的深科技供应链情报系统。**

回答四个问题：

> 谁供货？供什么货？谁投钱？——以及：这件事什么时候变？

## 为什么有价值（不是又一个"AI 搜公司"）

| 普通爬虫/搜索 | 本系统 |
|--------------|--------|
| "富瀚微是海康的供应商" | 该关系**带来源 URL、置信度等级（一手官方/权威媒体/自媒体）、信息年份** |
| 公司→公司 一层图 | **公司→产品→组件→工厂** 产品级穿透（NVIDIA H200 → HBM3E → SK海力士 → CoWoS → 台积电） |
| 静态快照 | **事件流 + 产能追踪**（谁扩产、谁售罄、何时变化） |
| 无法验证对错 | **Eval 报告**：50 条 gold set 核对，Relation Precision 100%，短板清单追踪 |
| 锁死在数据库/UI | 纯 markdown+YAML，Obsidian 看图，机器层可导出，不锁死任何软件 |

核心是 **claim-evidence-event** 模型：每条关系不是"AI 找到的"，而是"有证据链支撑、有时间戳、可复核的"。

## 数据流水线

```
公开数据（东财 API/公告/年报/新闻）
        ↓ 搜索系统（多源验证 + 来源分级）
  结构化抽取（实体/关系/产品/事件）
        ↓ 关系串联系统（ingest.py 幂等入库）
  Obsidian 情报库（实体图 + 证据 + 时间）
        ↓
  ask.py 单实体看板 ｜ panorama 全景图 ｜ watchlist 事件流 ｜ evidence-audit 审计
```

## 快速上手（2 分钟）

**方式 A：看效果**
下载 release zip → Obsidian 打开 → 图谱视图：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian/releases/tag/v0.1.0

**方式 B：问它**
```bash
python3 scripts/ask.py demo "富瀚微的客户是谁" -o 看板.html   # 单实体情报卡
python3 scripts/ask.py demo "联想控股投资了谁" -o 看板.html
python3 scripts/panorama.py demo 富瀚微 全志科技 -o 全景.html  # 两家公司全景
python3 scripts/watchlist.py ai-compute --watch NVIDIA,SK海力士 -o watchlist.html  # 事件流
```

**方式 C：建自己的库**
```bash
python3 scripts/ingest.py <你的vault> -f examples/relations-demo.yaml   # YAML → 节点+关系
python3 scripts/evidence-audit.py <你的vault>                            # 证据置信度打标
python3 scripts/import-helper.py demo <你的vault> --merge                # 整库合并
```

## 两个库

| 库 | 模型 | 规模 | 看什么 |
|----|------|------|--------|
| [demo/](demo/) | v1：公司-公司关系图 | 159 节点 / 718 关系 / 0 断裂 | 半导体全链条（设计/代工/设备/封测）+ 股东全节点化 + 证据打标 |
| [ai-compute/](ai-compute/) | v2：产品级+事件+产能 | 47 节点 / 8 事件 / 2 产能追踪 | NVIDIA 供应链穿透 + Watchlist 事件流 + claim 内联 |

## 工具链

| 工具 | 系统 | 作用 |
|------|------|------|
| `scripts/ingest.py` | 关系串联 | YAML → 节点 + 双向 wikilink（幂等） |
| `scripts/ask.py` | 展示 | 问询 → HTML 看板（证据徽章） |
| `scripts/panorama.py` | 展示 | 双实体全景图 |
| `scripts/cross-entity-scan.py` | **入库后扫描**：新实体自动对比全库，发现竞争/共享客户/供应链关联 + Discovery Report | `python3 scripts/cross-entity-scan.py <vault> 寒武纪 --report` |
| `scripts/watchlist.py` | 事件流 | 变化摘要（Change Detection） |
| `scripts/evidence-audit.py` | 审计 | 置信度打标 + 审计报告 |
| `scripts/import-helper.py` | 导入 | 合并进个人 Obsidian 库（冲突自动处理） |

## 文档

| 文档 | 内容 |
|------|------|
| docs/01 | Obsidian × Hermes 协作方式与环境 |
| docs/02 | 架构与 Proposal |
| docs/03 | Obsidian 使用方法 |
| docs/04 | 搜索方式与信息渠道 |
| docs/05 | 导入方案 |
| docs/06 | **Eval v1**：50 条关系 gold set 核对 + 短板追踪 |
| docs/07 | **Schema V2**：7 对象模型 + 50 字段 + 产品封装 |

## 合规声明

只采集公开信息；遵守网站条款；不采个人隐私；每条关键信息带来源 URL + 采集时间，可审计可导出。
