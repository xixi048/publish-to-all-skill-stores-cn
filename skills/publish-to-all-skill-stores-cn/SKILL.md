---
name: publish-to-all-skill-stores-cn
description: "把 Agent 技能上架到国内 Skill 商店：腾讯 SkillHub、讯飞 Astron、扣子 Coze、智谱 AgentMore、阿里百炼、字节 Find Skill、Cindy、WorkBuddy 技能中心、小红书、Kimi、美团等。一站式完成体检 → 打包 → 分渠道提交 → 回执留档：本地扫描危险调用与体积异常，产出 SKILL.md 在包根的 skillroot 包，按渠道给出可直接照做的提交配方（网页导入、上传 zip、GitHub PR、自建发现端点），并逐条记录各渠道回执与状态，驳回时给出具体改法。只依赖 Python 标准库，可离线运行。关键词：技能商店、上架、发布、推送、技能市场、国内渠道、skillhub、讯飞、Astron、扣子、Coze、百炼、AgentMore、腾讯、Cindy、WorkBuddy、审核、体检、打包、回执。"
license: MIT
metadata:
  author: xixi048
  version: "1.1.1"
  language: zh-CN
  scope: cn-only
---

# 国内技能商店上架（合规精简版）

## 本版的范围——先读这一段

本版**只覆盖国内 Skill 商店**，并且刻意不含两类内容：任何海外商店的推广、入口或注册指引；
任何把 GitHub 凭据转发给第三方镜像的推送方式。凭据只走官方地址 + `gh auth setup-git` 或 SSH，
连不上就停下报告，不绕道。

需要海外渠道时请用覆盖更全的完整版。两者不要混用，按下表选：

| 情况 | 做法 |
|---|---|
| 目标只有国内渠道 | 用**本版** |
| 需要海外渠道 | 用完整版 |
| 平台要求「不得含第三方商店推广」这类形式审查口径 | 用**本版** |
| 驳回理由是布局 / 许可证 / 危险调用 / 体积 | 技术问题，**改内容**，不要换版本 |
| 驳回理由是含海外商店推广或入口 | 换**本版**重提，并说明「已移除海外渠道条目」 |

## 渠道形态

「技能商店」不是一个东西，按**提交动作**分类，混淆会白费时间：

| 形态 | 提交动作 | 代表渠道 | 能否自动化 |
|---|---|---|---|
| 从 GitHub 导入 | 绑定账号 → 选仓库里的技能目录 | 腾讯 SkillHub、agentskill.sh 类 | ✅ 仓库就绪即可 |
| 上传 zip | 打包 → 传包 → 建草稿 | 讯飞 Astron、Cow Skill Hub | ⚠️ 需本人登录 |
| 宿主内置工具 | 调宿主的 `publish_skill` | Cindy SkillHub | ✅ 但 public 需审核 |
| 平台托管 | **把技能用平台方言重写** | 扣子 Coze、智谱 AgentMore、阿里百炼 | ❌ 是另一个开发项目 |
| GitHub PR | fork → 加文件 → PR | awesome 列表、社区 registry | ✅ 需 GitHub 写权限 |

只有前四类是「推送」，最后一类「平台托管」是「移植」。不要在移植上假装能一键推送。

**能 GitHub 导入就绝不本地上传。** 导入器读的是 `<owner>/<repo>/skills/<name>/`，
以后 push 就是更新版本，永远不会出现"商店版本落后于仓库"。只有平台不支持导入、
或包需要在仓库外裁剪时才用 zip。

国内渠道明细（含域名、登录要求、验证等级）见 [references/cn-stores.md](references/cn-stores.md)。

## 开工顺序

0. **先修仓库，再谈商店。** 仓库是所有下游的根。**默认私有**，但要**先问用户一句要不要开源**，
   不要自己定。私有仓库 = 抓取型目录站全部失效，只能走 zip 上传与 GitHub 导入。
1. **修仓库**（顺序错了全白干）
   - 技能放在 `skills/<name>/SKILL.md`——仓库根目录的 `{name}/SKILL.md` **不会被发现，也不报错**
   - 设 GitHub topics + 仓库 description（一句话说清"是什么 + 给谁用"）
   - 有 `LICENSE` 与可读 README（多数站直接拿 README 当详情页）
2. **体检**：`python scripts/preflight.py <repo>/skills/<name>` —— 报 ERROR 就别提交
3. **打包**：`python scripts/build_package.py <repo> skills/<name> --out dist`
   —— 上传型渠道要求 zip 根目录直接是 `SKILL.md`
4. **提交**：按渠道逐个走，缺凭据就明确告诉用户"这一步需要你登录"
5. **收尾**：每个渠道留回执（版本号 / 审核状态 / PR 链接）

## 硬性要求

**布局。** `skills/<name>/SKILL.md` 才被广泛索引。迁移后记得改 README 里所有相对路径。

**打包。** 上传型渠道和仓库布局天然冲突，必须单独打 skillroot 包。打包前删掉 `__pycache__`
（`.pyc` 会被原样打进 zip，很多平台的上传失败只回一句 "upload did not complete"）。

**中文区用中文。** 显示名、简介、分类、标签全部用中文，别把英文 description 粘过去。
SKILL.md 的 frontmatter 保持英文（它是给模型读的），本地化只发生在**商店的展示字段**上。

**含脚本的技能主动披露。** 审核会查 `eval` / `exec` / `subprocess` / 动态 import 与网络行为。
提前把「零依赖 / 无动态执行 / 写入目标由用户显式给出 / 每次写入前备份」写进提交备注，
比事后修改代码更有效。**索要补充材料时加说明，不要删功能**——安全能力就是卖点。

**推送凭据只给 GitHub 官方域名。** remote 必须是 `https://github.com/<owner>/<repo>` 或
`git@github.com:<owner>/<repo>`。连不上就**停止推送并如实报告**，不要把令牌塞进请求头转发给
任何第三方镜像——令牌一旦出去就脱离你的控制。细则见 [references/pitfalls.md](references/pitfalls.md)。

**定价。** 国内能收费的主要是腾讯 SkillPay（0 抽成，但只收依赖后端的 Pay Skill）与
扣子 Coze（约 70%）。纯本地脚本型技能通常收不了钱，别为了收费把架构改成必须联网。
见 [references/cn-paid-channels.md](references/cn-paid-channels.md)。

## MCP 侧（相邻赛道，交付物不同）

技能是「一份方法论」，MCP 是「一个可调用的服务」。同一个产品可以两边都发，
但**国内的 MCP 平台不是填个 URL 就完事**：

| 形态 | 平台做什么 | 你要交什么 |
|---|---|---|
| 元数据登记 | 只存一条记录 | 一个 URL |
| **平台托管（国内主流）** | **接管部署，代码跑在它的容器里** | 可容器化的完整代码 |

魔搭、百炼、火山都是第三类。走之前先问用户一句：**愿不愿意把服务代码托管给平台？**
计费、密钥、数据留在自己服务器上的服务，搬过去就是另一个产品。

三件事先做好再提交：端点能自己握手（`initialize` + `tools/list` + 一次真实调用）、
仓库里放一个可跑的自检脚本、服务名在包内与服务端声明一致。

**魔搭实测走通的是「自定义创建」，不是「从 GitHub 仓库快速创建」**——后者在境内会误报
「README 缺失」，别在这条路上改 README。表单里「托管类型」漏选会静默失败，
README 是 Monaco 编辑器、只能靠剪贴板粘贴写入。三个坑的细节见 cn-mcp.md。

**文案：不写会过期的固定数量。** 词条数、数据条数这类当前快照，词库一扩就对不上，
而它同时躺在 README、包描述、frontmatter 和几个平台后台里，改一次必然漏几处。
写能力不写数量：「一句话用感觉检索最匹配的视觉风格」。

**真正免登录的国内渠道只有两条**：GitHub 中文榜 PR（`yzfly/Awesome-MCP-ZH`，7.8K star，
`pull_request_creation_policy: all`）与自建发现端点。其余平台都要手机号/实名，必须用户本人登录。

逐站实测（入口、登录要求、卡点）、魔搭的完整创建路径与表单三坑、AIBase 的服务配置 JSON 格式，
见 [references/cn-mcp.md](references/cn-mcp.md)。文案类踩坑见
[references/pitfalls.md](references/pitfalls.md) 的「文案相关」。

## 提交配方

逐个渠道的动作、表单字段、导入步骤与踩坑见
[references/submission-recipes.md](references/submission-recipes.md)。

## 收尾标准

- 每个渠道都要有**回执**：提交 id、审核状态、版本号，缺一不可。没有回执等于没提交
- 提交后**不要反复轮询**审核状态；pending 就是 pending
- 交付时把「已提交 / 待登录 / 需移植 / 已排除」四类分开写，不要让用户以为全都上线了
