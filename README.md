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
| [docs/03-obsidian-usage-guide.md](docs/03-obsidian-usage-guide.md) | Obsidian 使用方法（对照 demo 实战） |
| [docs/04-search-methods.md](docs/04-search-methods.md) | 搜索方式与信息渠道（采集方法论） |

## 问询功能（HTML 看板）

```bash
python3 scripts/ask.py <vault路径> "富瀚微的客户是谁" -o 看板.html
```

支持实体名或问题句式；输出自包含 HTML（基本信息表 + 关系网络 SVG + 关联卡片 + 来源），浏览器直接打开。关系边按类型着色：投资紫 / 供货蓝 / 竞争红 / 代工绿。

**看板模板**：[templates/board.html](templates/board.html) —— 样式/布局与数据分离，改模板即可换皮肤（占位符：TITLE/NAME/TYPE_LABEL/FETCHED/NEIGHBOR_COUNT/HERO_COLOR/FM_ROWS/SVG/REL_CARDS/BODY_SNIP/SRC/DATE）。自定义模板用 `-t 路径` 指定；不指定时自动用脚本同级 `templates/board.html`，文件不存在则用内置兜底样式。

## 可运行 Demo

[demo/](demo/) 是一个**已经跑通的 Obsidian 情报库案例**：以"上海富瀚微电子（300613）"和"珠海全志科技（300458）"为目标公司，完整演示：

- 投资方：联想控股入股富瀚微（9.89 亿/9.9%，现持股 15.6%）、君联资本早期投资
- 上下游：海康威视为富瀚微第一大客户（营收 60%+）；比亚迪为全志车载芯片第一大客户
- 竞争关系：两家同在端侧 AI 视频芯片赛道竞逐（含 mermaid 竞争格局图）

打开方式：Obsidian → 打开文件夹作为仓库 → 选择 `demo/` → 看「关系图谱」。

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
