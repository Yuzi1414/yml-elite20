# C5A 复盘（AAR）

> 复盘人：虞梦琳（yml）　日期：2026-11-03

---

## 一、做了什么（What）

C5A 是「GitHub 入门与仓库理解」挑战，我完成了 GitHub 操作闭环的前四级：

- **Level 1（Bronze）：** 使用已有的 GitHub 账号 `Yuzi1414`，创建了个人仓库 `yml-elite20`，并添加了包含名字/专业/一句话介绍的 `README.md`。
- **Level 2（Silver）：** Fork 了 `neolaf2/neoskills` 到 `Yuzi1414/neoskills`；Clone 到本地；通读 README、`skills/`、`agents/`、`pyproject.toml`、`src/` 分层，写出了《仓库分析》文档。
- **Level 3（Gold）：** 在 fork 上创建分支 `my-contribution`，新增 `docs/quickstart.zh-CN.md`（中文快速上手指南），commit 并 push 到 fork。
- **Level 4（Platinum）：** 在《仓库分析》中补充了架构分析（文件系统为真相源 + 本体图 + 适配器）、技能逐项分析、4 条改进建议、以及关于 Agent 技能设计的学习收获。

---

## 二、学了什么（Learn）

1. **工程素养优先于编程技巧。** C5A 不考写代码，考的是「能否读懂别人的仓库并清楚解释」。理解一个项目的分层（CLI / 核心 / 本体图 / 适配器）和设计意图，本身就是核心能力。
2. **一个「技能（skill）」的本质是「可发现、可治理、可移植的声明式指令」。** 从 neoskills 学到的三要素——可发现性（清晰的 name/description）、可治理性（来源/版本/依赖）、可移植性（adapter 翻译复用），直接指导我以后设计自己的技能。
3. **零拷贝符号链接的部署思想。** 用 symlink 而非复制来部署技能，改动一处全网生效、`unlink` 即可回滚，这是「可逆性」在工程里的漂亮体现。
4. **真实工程会遇到「非技术」问题。** 这次最大的拦路虎不是代码，而是本机 `git` 的全局代理指向了一个不可达的本地端口（`127.0.0.1:17897`），导致所有 GitHub 操作失败。

---

## 三、怎么验证的（Verify）

| 验证项 | 结果 |
|--------|------|
| Fork 成功 | `gh repo view Yuzi1414/neoskills` 可见，上游指向 `neolaf2/neoskills` |
| Clone 成功 | 本地 `.git` 目录存在，HEAD = 上游 `7c4dada` 提交 |
| 贡献分支 | `my-contribution` 已 push 到 fork，`gh api` 可见该分支 |
| 个人仓库 | `gh repo view Yuzi1414/yml-elite20` 返回公开仓库 URL |
| 分析文档准确 | 每个结论均来自真实文件阅读，并在文末列出「我实际做了哪些验证」 |

---

## 四、卡点与突破（Struggle & Breakthrough）

- **卡点：** `git clone`/`git push` 到 GitHub 全部失败，报「Failed to connect to github.com:443 over proxy 127.0.0.1:17897」。
- **定位：** 排查发现是 `git` 全局配置里残留了 `http.proxy` / `https.proxy`，指向一个已不存在的本地代理。
- **突破：** 用 `git -c http.proxy= -c https.proxy= <命令>` 临时清空代理，即可直连 GitHub。
- **额外收获：** clone 时 `git` 把进度写到了 stderr，PowerShell 把 stderr 当成「错误」显示，一度误判 clone 失败——实际验证 `.git` 目录后确认成功。这让我学会「看真实结果，别只看报错文字」。

---

## 五、改进方向（Improve）

1. **截图证据要尽早、边做边截。** 我在命令行完成了全部操作，但挑战要求的截图（账号/仓库/fork/clone/commit）需要本人打开浏览器补拍，如果边做边截会更从容。
2. **专业信息需提前准备。** README 需要「专业」字段，这是只有我自己知道的信息，提前准备好能让交付物一次到位。
3. **代理问题是环境陷阱。** 下次做任何涉及网络的操作前，先 `git config --list` 检查一遍代理配置，避免重复踩坑。

---

## 六、一句话总结

C5A 让我迈出了「把项目搬上 GitHub」的第一步，也让我明白：**读懂别人的代码、把学到的东西讲清楚，比写出自己的代码更难，也更重要。**
