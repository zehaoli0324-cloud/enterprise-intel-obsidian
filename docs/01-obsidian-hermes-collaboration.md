# Obsidian × Hermes 协作：方式与环境（部署学习指南）

> 版本：v1.0 · 2026-08-26
> 适用对象：想自己部署一套「AI 代理 + Obsidian 知识库」协作系统的人。
> 本文档基于一套真实运行的环境总结，可直接照搬或裁剪。

---

## 1. 一句话概括

**Hermes（AI 代理）和 Obsidian（知识管理软件）共享同一个 markdown 文件夹作为"数据库"。**

- Hermes 负责：采集资料、整理信息、写笔记、定时更新、批量处理
- Obsidian 负责：人类阅读、检索、关系图谱、表格查询、可视化
- 两者**不通过 API 对接**，而是通过"**文件即数据库**"的方式协作

Obsidian 的 vault（仓库）就是一个普通文件夹，里面全是 `.md` 纯文本文件。任何能读写文件的程序都可以参与进来——AI 代理只是其中一个"生产者"。

```
┌──────────────────┐         ┌──────────────────────┐
│   Hermes (AI)    │         │     Obsidian (人)     │
│  采集/写入/定时   │         │   阅读/图谱/查询      │
└────────┬─────────┘         └──────────┬───────────┘
         │                              │
         │    同一个 markdown 文件夹     │
         ▼                              ▼
    ┌───────────────────────────────────────┐
    │        Vault（纯 .md + YAML）          │
    │  笔记文件 · frontmatter · wikilinks    │
    └───────────────────────────────────────┘
```

---

## 2. 环境（实测配置）

| 组件 | 角色 | 运行位置 |
|------|------|---------|
| Hermes Agent | AI 代理（读写 vault、搜索、定时任务） | WSL2 Ubuntu（也支持 Linux/Mac/Windows 原生） |
| Obsidian | 知识库前端（图谱、双链、查询） | Windows 桌面版（macOS/Linux 均可） |
| Vault | 纯 markdown 文件夹 | 任意路径，例如 Windows D 盘 `D:\MyVault`（WSL 视角 `/mnt/d/MyVault`） |
| 可选：Feishu/Telegram gateway | 消息入口，聊天记录自动入 vault | 同 Hermes 所在机器 |

关键点：

1. **Vault 放在 Windows 侧，Hermes 跑在 WSL 里**，通过 `/mnt/d/...` 挂载路径访问。两边共享同一文件系统，无需同步。
2. Obsidian 不需要装任何特殊插件就能配合（核心插件 graph / backlink / properties / templates 已够用）。可选的社区插件（Dataview、Bases）用于进阶查询。
3. Hermes 的配置文件 `~/.hermes/` 里可以设置环境变量 `OBSIDIAN_VAULT_PATH` 指向 vault，之后所有知识库操作都走这个路径。

---

## 3. 协作机制（核心）

### 3.1 文件系统直连（最主要通道）

Hermes 用文件工具直接操作 vault 里的 `.md` 文件：

- `read_file` —— 读笔记
- `write_file` —— 创建/覆盖笔记
- `patch` —— 局部修改
- `search_files` —— 全文检索（比 Obsidian 自带搜索更快，支持正则）

不需要插件、不需要 API key、不需要端口监听。Obsidian 检测到文件变化会自动刷新。

### 3.2 结构化约定（让机器可写、人可读）

| 机制 | 语法 | 作用 |
|------|------|------|
| YAML frontmatter | 文件开头的 `---` 块 | 存结构化字段（日期、分类、来源、金额……），可被查询 |
| [[wikilinks]] | `[[笔记名]]` | 笔记之间的关系 → Obsidian 图谱里的"边" |
| tags | `#标签` 或 frontmatter 里的 `tags:` | 分类维度，图谱可分组染色 |
| mermaid | ` ```mermaid ` 代码块 | 流程图、组织架构图、产业链图 |

**约定即接口。** Hermes 写笔记时遵循固定的 frontmatter 字段和 wikilink 命名，Obsidian 端就能得到干净可查的图谱。

### 3.3 Obsidian URI 控制（让 AI 打开人的界面）

Hermes 可以从命令行触发 Obsidian 打开指定笔记或图谱：

```
obsidian://open?vault=VaultName&file=笔记路径
obsidian://graph?vault=VaultName&searchquery=关键词
```

Windows 下通过 PowerShell 调用（注意 `&` 要用单引号包住，否则会被当成命令分隔符）：

```powershell
powershell.exe -Command "Start-Process 'obsidian://open?vault=MyVault&file=总目录'"
```

### 3.4 对话记录持久化（配合消息网关，可选）

Hermes 跑在消息平台上时，每个群/私聊的讨论结论会自动存成 vault 里的一篇笔记（以 chat_id 命名），避免聊天上下文被压缩丢失。检索时直接搜 vault。

### 3.5 定时任务（让数据库"实时更新"）

Hermes 的 cron 可以定时执行任务：每天早上去抓指定网站、检查网页变更、把新信息写入 vault。这就是"实时更新数据库"的引擎。

---

## 4. 三层知识分工（避免混乱）

| 层 | 放什么 | 例子 |
|----|--------|------|
| Hermes memory | 少量长期事实：用户偏好、路径、环境 | "vault 在 /mnt/d/MyVault" |
| Obsidian vault | 内容主体：文献、项目、对话、调研 | 论文笔记、实验记录、公司档案 |
| Hermes skills | 可复用流程（怎么做事） | "如何把网页存成笔记" |

原则：**长内容进 Obsidian，短事实进 memory，流程进 skill。**

---

## 5. 从零部署步骤

1. **装 Obsidian**：官网下载桌面版 → 打开时选"打开文件夹作为仓库"，指向一个空文件夹（例如 `D:\MyVault`）。
2. **装 WSL2 + Hermes Agent**（Linux 环境里跑 AI 代理），参考 Hermes 官方文档（https://hermes-agent.nousresearch.com/docs）。
3. **告诉 Hermes vault 路径**：设置环境变量 `OBSIDIAN_VAULT_PATH=/mnt/d/MyVault`（WSL 路径）。
4. **建总目录**：让 Hermes 读一遍 vault，创建 `总目录.md` 索引页。
5. **建模板**：在 vault 里建 `_模板/` 文件夹，定义笔记的统一格式（frontmatter 字段）。
6. **开始协作**：让 Hermes 写第一篇笔记，Obsidian 里刷新就能看到；加个 `[[wikilink]]`，图谱里就出现连线。
7. **（进阶）装 Dataview/Bases 插件**：用 SQL 风格查询把 frontmatter 字段渲染成表格。
8. **（进阶）配置 cron**：定时任务自动抓取/更新，实现"实时更新"。

---

## 6. 常见坑（实测踩过）

1. **路径含空格**：vault 路径常含空格（`Obsidian Vault`），shell 命令要加引号；文件工具传绝对路径最稳。
2. **wikilink 重名破坏图谱**：两个文件夹里有同名笔记时，裸 `[[名字]]` 无法解析，图谱不连线。用全路径 `[[文件夹/名字]]`。
3. **wikilink 别放 frontmatter 和代码块里**：图谱不会渲染它们。
4. **文件名避免 `&`**：`CUT&Tag` 这类名字在 Windows 批处理里有歧义，用 `-` 代替（`CUT-Tag`）。
5. **大量文件复制用 shell `cp`**：Python 的 `shutil.copy2` 在 `/mnt` 挂载盘上极慢。
6. **改文件名要全局更新引用**：重命名笔记后，所有指向它的 wikilink 都会断，需要批量替换。

---

## 7. 为什么这套方案适合"企业情报数据库"

- 企业、人物、产品、投资人天然是**节点**，供货/投资/持股天然是**边** → Obsidian 图谱就是产业链可视化
- 每条信息带**来源 URL + 抓取时间** → 可追溯、防 AI 幻觉
- Hermes cron 定时采集 → **实时更新**
- 纯 markdown → 可 git 版本管理、可导出、可迁移、不锁死在软件里

具体的架构与 proposal 见：《02-企业情报数据库-架构与Proposal》。
