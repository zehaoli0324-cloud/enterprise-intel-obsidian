# Enterprise Intel DB（企业情报数据库）

企业上下游供应商与投资人信息调查的「实时更新数据库」，以 Obsidian × Hermes 方式构建。

## 这是什么

一个可部署、可扩展的企业情报收集系统：

- **资料搜索**：自动抓取公开信息（工商、财报、招投标、融资、新闻）
- **数据库管理**：Obsidian 纯文本库 —— 节点 = 企业/人物/产品/投资人，连线 = 关系
- **关键信息抽取**：LLM 从非结构化文本抽"谁供货、供什么货、谁投钱"，带来源可追溯

## 文档

| 文档 | 内容 |
|------|------|
| [docs/01-obsidian-hermes-collaboration.md](docs/01-obsidian-hermes-collaboration.md) | Obsidian × Hermes 协作方式与环境（部署学习指南） |
| [docs/02-企业情报数据库-架构与Proposal.md](docs/02-企业情报数据库-架构与Proposal.md) | 企业情报库架构 + 实施提案 |

## 快速开始（3 步）

1. 装 Obsidian，把任意空文件夹打开为 vault（或直接 `git clone` 本仓库作为 vault 骨架）
2. 部署 Hermes Agent（https://hermes-agent.nousresearch.com/docs），设置 `OBSIDIAN_VAULT_PATH` 指向 vault
3. 复制 `_模板/` 下的笔记模板，让 Hermes 按模板采集第一家公司

## 目录结构

```
├── docs/                  # 架构文档
│   ├── 01-obsidian-hermes-collaboration.md
│   └── 02-企业情报数据库-架构与Proposal.md
├── _模板/                 # 笔记模板（Company / Person / Product / Investor / Deal）
├── 00-总目录.md           # 索引页
├── 01-公司/
├── 02-人物/
├── 03-产品物料/
├── 04-投资人/
├── 05-交易事件/
├── 06-产业链/
└── 07-变更日志/
```

## 合规声明

只采集公开信息；遵守目标网站服务条款；不采集个人隐私；所有关键信息带来源 URL。
