# Enterprise Intel DB（企业情报数据库）

**AI 代理（Hermes）× Obsidian 构建的「实时更新」企业情报库** —— 做企业上下游供应商、投资人信息调查。

回答三个核心问题：

1. **谁供货** —— 某公司的供应商是谁
2. **供什么货** —— 供应商提供什么产品/物料
3. **谁投钱** —— 公司的投资人是谁、投了多少

节点 = 企业/人物/产品/投资人，连线 = 供货/投资/竞争关系。纯 markdown + YAML，不锁死在任何软件里。

---

## 功能清单

| 工具 | 作用 | 用法 |
|------|------|------|
| `scripts/ingest.py` | **写入端**：YAML 关系文件 → 批量生成 Obsidian 节点 + 双向 wikilink 关系 | `python3 scripts/ingest.py <vault> -f relations.yaml` |
| `scripts/ask.py` | **读取端**：问询 → 生成 HTML 看板（关系图 + 关联卡片 + 来源） | `python3 scripts/ask.py <vault> "富瀚微的客户是谁" -o 看板.html` |
| `scripts/panorama.py` | **全景图**：双实体关系全景（两家公司 + 全部上下游/投资/竞争网络） | `python3 scripts/panorama.py <vault> 富瀚微 全志科技 -o 全景.html` |
| `templates/board.html` | 看板皮肤（样式与数据分离，改模板即换肤） | ask.py 自动读取，或用 `-t 自定义.html` |
| `docs/` | 5 篇文档：协作环境 / 架构 / Obsidian 使用 / 搜索方法论 / 导入方案 | 见下方索引 |
| skill | `enterprise-intel-obsidian`（Hermes 全流程 + 坑） | 装入 Hermes 后自动触发 |

---

## 看效果（样本）

HTML 看板实例（富瀚微，24 个关联实体）：

- 样本 HTML：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian/blob/main/samples/看板样本-富瀚微.html
- 样本 PNG：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian/blob/main/samples/看板样本-富瀚微.png

---

## 快速上手（2 分钟）

### 方式 A：下载现成 Obsidian 库（推荐先看效果）

1. 下载 release 附件：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian/releases/download/v0.1.0/enterprise-intel-demo-vault.zip
2. 解压 → 得到 `enterprise-intel-demo/` 文件夹
3. Obsidian 桌面版 →「打开文件夹作为仓库」→ 选择该文件夹
4. 右侧打开「关系图谱」→ 立刻看到供应链/投资/竞争网络

### 方式 B：Clone 仓库（含全部代码与文档）

```bash
git clone https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian.git
cd enterprise-intel-obsidian
# 仓库里的 demo/ 本身就是可打开的 Obsidian 库
```

---

## Demo 内容（富瀚微 × 全志科技）

一个已经跑通的真实案例（2026-08-26 采集，信息均来自公开渠道、带来源链接）：

- **投资方**：联想控股 2020-09 通过旗下西藏东方企慧以 9.89 亿受让富瀚微 9.9% 股份，现持股 15.6%（第一大股东）；君联资本 2006 年起为富瀚微最早机构投资人
- **供货链**：海康威视为富瀚微第一大客户（营收 60%+）；比亚迪为全志科技车载芯片第一大客户（2024 采购超 12 亿）；全志供货海康旗下萤石网络
- **竞争关系**：富瀚微与全志科技同在端侧 AI 视频芯片赛道竞逐（直接竞争）；都进海康生态（间接竞争）；晶圆代工同为中芯国际/台积电/联电（供应链重叠）
- **图谱规模**：51 个节点（公司 24 / 人物 8 / 投资人 5 / 产品 3 / 事件 2 / 分析 2 / 索引 1 / 模板 5 + 日志），168 条关系边，实体核心关系全部双向

---

## 目录结构

```
├── docs/                  # 5 篇文档
│   ├── 01-obsidian-hermes-collaboration.md   # Obsidian × Hermes 协作方式与环境
│   ├── 02-企业情报数据库-架构与Proposal.md   # 架构 + 实施提案
│   ├── 03-obsidian-usage-guide.md            # Obsidian 使用方法（对照 demo 实战）
│   ├── 04-search-methods.md                  # 搜索方式与信息渠道（采集方法论）
│   └── 05-import-guide.md                    # 导入方案（并入自己的 Obsidian 库）
├── scripts/
│   ├── ask.py             # 问询 → HTML 看板
│   ├── ingest.py          # YAML → 节点 + 关系
│   └── import-helper.py   # 情报库合并进现有 vault（冲突自动处理）
├── templates/
│   └── board.html         # 看板模板（可自定义皮肤）
├── examples/
│   └── relations-demo.yaml  # ingest 输入示例
├── samples/
│   ├── 看板样本-富瀚微.html  # HTML 看板实例
│   └── 看板样本-富瀚微.png   # 看板渲染 PNG 实例
├── demo/                  # 可运行的 Obsidian 情报库案例
│   ├── 00-总目录.md
│   ├── 01-公司/ 02-人物/ 03-产品物料/ 04-投资人/
│   ├── 05-交易事件/ 06-产业链/ 07-变更日志/
│   ├── _模板/             # 笔记模板
│   ├── scripts/           # 工具（开箱即用）
│   └── templates/         # 看板模板
└── release/               # 打包好的 zip（不进 git，走 GitHub release）
```

---

## 日常使用流程

```
采集（Hermes 搜索/API） → 入库（ingest.py 生成节点+关系） → 问询（ask.py 出 HTML 看板）
        ↓                         ↓                            ↓
   docs/04 搜索方法论         Obsidian 图谱实时可视化        Obsidian 反链 + 图谱复核
```

1. **建库**：复制 `demo/_模板/`，按模板建节点；或直接用 `ingest.py -f relations.yaml` 批量生成
2. **更新**：改 YAML 加新实体/新关系 → 重跑 ingest（幂等，不覆盖已有正文）→ 变更记录写 `07-变更日志/`
3. **问询**：`ask.py` 生成看板，或直接在 Obsidian 里点开节点看反链

---

## 合规声明

- 只采集公开信息，遵守目标网站服务条款
- 不采集个人隐私（手机号、住址等）
- 每条关键信息带来源 URL + 采集时间，可追溯、可复核
- 数据是纯 markdown，可 git 版本管理、可导出、可审计

---

## 链接

- 仓库：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian
- Release（现成 Obsidian 库 zip）：https://github.com/zehaoli0324-cloud/enterprise-intel-obsidian/releases/tag/v0.1.0
- Hermes Agent（AI 代理本体）：https://hermes-agent.nousresearch.com/docs
