# neoskills 仓库分析

> 分析对象：`neolaf2/neoskills`（Fork 至 `Yuzi1414/neoskills`）
> 分析人：虞梦琳（yml）
> 日期：2026-11-03

---

## 一、这个仓库是做什么的？（一句话概括）

**neoskills 是一个「Homebrew 风格」的 AI 编程代理技能管理器**——它把分散在不同 AI 代理（Claude Code、OpenCode、OpenClaw）里的「技能（skill）」统一管理起来，让你能在一个地方浏览、搜索、同步、部署技能，并用一张「本体图（ontology graph）」看清技能之间的依赖关系。

打个比方：Homebrew 管理 macOS 的软件包，而 neoskills 管理 AI 代理的技能包。它不是某个具体的 AI 功能，而是一套**管理 AI 技能的基础设施（substrate）**。

---

## 二、仓库的目录结构

```
neoskills/
├── .claude-plugin/          # Claude Code 插件配置（把 neoskills 嵌入为 MCP 插件）
├── .github/workflows/       # CI/CD 自动化（测试、构建）
├── agents/                  # 4 个 AI 代理定义（scanner/importer/deployer/dedup）
├── commands/                # 自定义 slash 命令（ns.md）
├── docs/                    # 设计文档（ontology-design.md、plans/）
├── skills/                  # 仓库自带的技能（bank-status、skill-dedup）
├── src/neoskills/           # Python 源码（核心实现）
│   ├── adapters/            # 各代理适配器（claude / opencode / openclaw）
│   ├── cli/                 # 命令行接口（18 个命令模块）
│   ├── core/                # 核心逻辑（config、frontmatter、linker 等）
│   ├── meta/                # 元数据
│   ├── ontology/            # 本体图层（属性图：graph/loader/writer/engine…）
│   ├── plugin/              # MCP 插件
│   ├── runtime/             # 运行时集成（claude）
│   └── translators/         # 跨代理翻译
├── tests/                   # 单元测试 + 集成测试（共 172 个用例）
├── main.py                  # 入口
├── pyproject.toml           # 项目元数据与依赖（Python 3.13+，MIT）
├── uv.lock                  # 依赖锁文件
├── Dockerfile               # 容器化
├── CLAUDE.md                # Claude Code 的仓库级指令
├── TASKS.md                 # 任务清单
├── neoskills_v0_1.md ~ v0_3.md  # 版本演进文档（设计演进历史）
├── LICENSE                  # MIT 协议
└── README.md                # 项目说明
```

**结构特点：** 这是一个「分层清晰」的 Python 项目——`cli`（对外命令）、`core`（核心）、`ontology`（本体图）、`adapters`（跨代理适配）四层分离，符合工程上「关注点分离」的原则。

---

## 三、里面有哪些技能（skills）？各自做什么？

仓库 `skills/` 目录下自带两个技能：

### 1. bank-status（技能库状态查询）

- **作用：** 当用户询问「技能库现在有什么」「库存如何」时，读取 `~/.neoskills/registry.yaml` 和 `state.yaml`，展示技能总数、标签、最近更新时间、当前已嵌入（symlink）的技能、最近导入的 5 个技能及其来源。
- **输出格式：** 一个结构化摘要（Bank / Targets / Embedded / Recent）。
- **一句话：** 它是技能库的「仪表盘」。

### 2. skill-dedup（技能去重）

- **作用：** 扫描技能库和多个代理目标（Claude Code、OpenCode、插件），识别**重复/近似重复**的技能，并用 merge/remove/keep 三种策略解决。
- **核心分类：** 把重复分为三类——① 完全重复（相同 SHA256，可安全合并为 symlink）；② 发散副本（同 ID 但内容不同，推荐导入更丰富的一版）；③ 名称相似组（需人工判断）。
- **亮点：** 自带 `dedup_scan.py` 脚本，支持 `--json`、`--resolve`（自动解决）、`--dry-run`（预览），并自动备份被替换的文件。
- **一句话：** 它是技能库的「去重清理工」。

> 补充：`agents/` 目录下还有 4 个与技能管理配套的 AI 代理定义（skill-scanner 扫描发现、skill-importer 导入、skill-deployer 部署、skill-dedup 去重），它们和 `skills/` 目录里的技能是「工具箱」和「自动化操作员」的关系。

---

## 四、我觉得哪个技能最有意思？为什么？

**我认为 `skill-dedup`（技能去重）最有意思**，原因有三：

1. **它解决了一个真实且普遍的痛点。** 用过多个 AI 代理的人都会有这种体验：同一个技能在 Claude Code、OpenCode 里各存了一份，改了一个忘了另一个，久而久之版本混乱。skill-dedup 把这个「隐性混乱」变成「显性可治理」的问题——先扫描、再分类、再解决。

2. **它的「三分法」体现了工程判断力。** 它没有粗暴地「发现重复就删除」，而是区分「完全重复 / 发散副本 / 名称相似」，对不同情况给不同策略，甚至对不确定的「名称相似组」留给人工判断。这种「该自动自动、该人工人工」的边界感，正是好工具设计的标志。

3. **它自带安全机制。** 自动解决前有 `--dry-run` 预览，替换文件前会自动备份为 `.<name>.dedup-backup`。这让我联想到「可逆性」是工程里非常重要的原则——工具应该让用户敢用、敢试错。

通过这个技能，我理解了 neoskills 的核心理念：**让技能从「散落在各处的文件」变成「有来源、有关系、可治理的资产」**。

---

## 五、架构设计思路（Level 4 深度补充）

### 整体设计：一个「以文件系统为真相源」的技能银行

neoskills 最核心的设计决定是：**不用外部数据库，而是用文件系统 + YAML 侧文件（sidecar）作为唯一真相源**。技能就是一个目录，里面放 `SKILL.md`（定义）+ 可选的 `ontology.yaml`（图元数据）+ `scripts/`、`references/`、`assets/`。

这带来几个好处：

- **零依赖、易迁移**：把整个 `~/.neoskills/` 目录打包拷走，技能就完整迁移了。
- **天然版本可控**：技能目录本身就在 git 仓库里，版本历史天然存在。
- **渐进式增强**：技能可以「L0 只有 SKILL.md → L1 加元数据 → L2 加关系 → L3 加治理」，不用一开始就填全。

### 各模块如何协作

```
CLI（cli/） ──调用──▶ core/（config、linker、frontmatter）
                        │
                        ▼
              ontology/（loader 从文件系统读 → graph 内存属性图 → writer 写回 YAML）
                        │
                        ▼
              adapters/（把技能翻译/链接到 claude-code、opencode、openclaw 各自的技能目录）
```

- **ontology/ 是灵魂**：它把散落的技能文件读进内存，构建成一张「属性图」（节点=技能，边=requires/extends/composes/conflicts 等关系），支撑依赖分析、组合、拆分、版本管理、可视化（Mermaid/DOT/JSON）等高级能力。
- **adapters/ 是桥梁**：不同代理的技能目录路径和格式不同，adapter 负责「翻译」和「创建符号链接」，实现「一份源码，多代理共用」。

### 部署模型：零拷贝符号链接

技能部署不是复制文件，而是创建 symlink：`~/.claude/skills/kstar-loop → ~/.neoskills/taps/.../kstar-loop`。这样改动 tap 里的技能，所有链接它的代理立即生效，且 `unlink` 即可完全回滚。

---

## 六、改进建议（至少 3 条）

1. **`skills/` 目录内置技能太少（仅 2 个），与庞大的 `agents/` 和 `ontology/` 能力不成比例。** 建议内置更多示例技能（如 `skill-search`、`skill-publish`），让新用户 clone 后能立刻看到「技能银行」的实际价值。

2. **文档存在版本割裂。** 根目录有 `neoskills_v0_1.md ~ v0_3.md` 四份历史版本文档，而 `pyproject.toml` 已到 `0.4.2`，README 又提到了 `v0.4` 的 ontology 层——历史文档没有「已废弃」标记，新用户容易混淆。建议在旧文档开头加注「本版本已合并进 v0.4，见 README」的提示。

3. **`dedup_scan.py` 的分类阈值（`--threshold 0.80`）和相似度算法没有在文档里充分说明。** 用户不知道这个 0.8 是怎么算出来的（是名称编辑距离还是内容重叠度），难以判断结果是否可信。建议补充一段「相似度是如何计算的」说明。

4. **缺少「新手的一分钟上手指引」的端到端示例。** README 的 Quick Start 是命令罗列，但对「完全没用过的人」缺少一个「从 0 到装好第一个技能」的完整截图式教程。

---

## 七、学习收获（关于 Agent 技能设计）

从 neoskills 我学到的最重要一课是：**一个「技能（skill）」的本质，是一段「可被 AI 代理发现、理解并执行的声明式指令」**，它的价值取决于三件事：

1. **可发现性（discoverable）**——技能要有清晰的 `name` + `description`，让代理知道「什么时候该用我」。skill-dedup 的 description 写得很讲究，列举了大量触发场景和正反例。
2. **可治理性（governable）**——技能要有来源（provenance）、版本、生命周期状态、依赖关系，否则会退化成一堆没人敢动的文件。这是 ontology 层存在的意义。
3. **可移植性（portable）**——一份技能定义，通过 adapter 翻译，就能在不同代理间复用，不用为每个代理重写。

这让我明白：**写技能不是写一段 prompt，而是在设计一个有边界、有元数据、有关系、可治理的小型知识资产。** 这对我以后设计自己的技能（比如 C4 系列里做过的 skill-evaluator、wechat-publisher）有直接的指导意义。

---

## 附：我实际做了哪些验证

- ✅ 通过 `git clone` 克隆上游仓库到本地，浏览了真实目录结构
- ✅ 通过 `gh` 命令行将仓库 Fork 到我的账号 `Yuzi1414/neoskills`
- ✅ 逐个阅读了 `skills/` 下两个技能的 `SKILL.md` 全文
- ✅ 逐个阅读了 `agents/` 下 4 个 agent 定义的 frontmatter 与职责说明
- ✅ 阅读了 `pyproject.toml`（确认 Python 3.13+、MIT、依赖列表）
- ✅ 梳理了 `src/neoskills/` 的模块分层（cli/core/ontology/adapters）
- ✅ 确认测试规模（README 声明 172 个测试用例）

（以上结论均来自对仓库真实文件的阅读，而非凭印象臆测。）
