---
name: publish-to-all-skill-stores-cn
description: "把技能上架到国内 Skill 商店（腾讯 SkillHub、讯飞 Astron、扣子 Coze、智谱 AgentMore、阿里百炼、Cindy、WorkBuddy 技能中心等）。**仅在原版 publish-to-all-skill-stores 被平台驳回、且驳回理由是「包含谷歌 / 海外商店的推广与入口」时使用本版**——其余情况一律用原版。本版已移除全部海外商店条目、链接与注册项，只保留国内渠道与 GitHub 正常开发引用，并禁用任何向第三方镜像转发令牌的推送方式。关键词：国内技能商店、上架、推送、技能市场、skillhub、讯飞、Astron、扣子、Coze、百炼、AgentMore、审核驳回、合规版。"
license: MIT
metadata:
  author: xixi048
  version: "1.0.0"
  language: zh-CN
  scope: cn-only
---

# 国内技能商店上架（合规精简版）

## 本版的定位——先读这一段

**默认用原版 `publish-to-all-skill-stores`。** 国内大部分渠道（腾讯 SkillHub、扣子、智谱、阿里百炼、
Cindy、WorkBuddy）都能正常接受原版，原版渠道表更全、配方更细，没有理由自废武功。

**只有一种情况用本版**：原版已经提交、平台审核**明确驳回**，且驳回理由落在
「包含谷歌 / 海外应用商店的推广、入口或注册指引」这一类合规问题上。

判定与处置：

| 情况 | 做法 |
|---|---|
| 还没提交过 | 用**原版**。国内渠道基本都收 |
| 原版被驳回，理由是布局 / 许可证 / 危险调用 / 体积 | **改原版**，不换版。那是技术问题，不是合规问题 |
| 原版被驳回，理由是含谷歌 / 海外商店推广与入口 | 用**本版**重新提交，并在回复里说明「已移除海外渠道条目」 |
| 平台只做形式审查、要求"无第三方商店推广" | 用**本版** |

本版相对原版做了三件事：**移除了全部海外商店条目与链接**、**移除了全部向第三方镜像转发令牌的
推送方式**、**只保留国内渠道与 GitHub 正常开发引用**。

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

## 提交配方

逐个渠道的动作、表单字段、导入步骤与踩坑见
[references/submission-recipes.md](references/submission-recipes.md)。

## 收尾标准

- 每个渠道都要有**回执**：提交 id、审核状态、版本号，缺一不可。没有回执等于没提交
- 提交后**不要反复轮询**审核状态；pending 就是 pending
- 交付时把「已提交 / 待登录 / 需移植 / 已排除」四类分开写，不要让用户以为全都上线了
