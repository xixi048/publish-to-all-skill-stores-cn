# 国内渠道清单

> 时间基准：2026-10。生态变动快，**URL 先按本表试，失败就重新逆向一次**。
>
> 本表只收**国内渠道**与 **GitHub 正常开发引用**。海外商店条目、入口与注册指引已在本版移除。

验证等级：✅ 实际访问/读到官方文档 ｜ △ 二手信源 ｜ ⚪ 未验证，先别花时间

## 一、GitHub 本身（所有下游的根）

| 项 | 内容 |
|---|---|
| 技能路径 | `skills/<name>/SKILL.md`（仓库根目录的 `{name}/SKILL.md` 不会被发现，也不报错） |
| 必备 | `LICENSE`（MIT / Apache-2.0）、可读 README、仓库 description |
| 建议 | GitHub topics：`agent-skills` `skill-md` + 具体领域词 |
| 用途 | 抓取型目录站的数据源；支持「从 GitHub 导入」的平台直接读这里 |

## 二、支持「从 GitHub 导入」的渠道（首选）

| 渠道 | 入口 | 登录 | 备注 |
|---|---|---|---|
| **腾讯 SkillHub** | `https://skillhub.cn`（旧 `skillhub.tencent.com` 已 301） | 需登录 + 实名 | 走「发布 Skill」不要选「发布 Pay Skill」（后者需支付改造）；详见配方 |
| **讯飞 Astron SkillHub** | `https://skill.xfyun.cn` → `/dashboard/publish` | 讯飞账号 | `/publish` 是 404，正确入口在 dashboard 下；走「上传 ZIP → 草稿 → 质量评测 → 发布」 |
| **WorkBuddy 技能中心** | 宿主内 | 宿主账号 | 技能目录形态，读 `.workbuddy/skills/` 本地布局 |

## 三、宿主内置工具型

| 渠道 | 形态 | 注意 |
|---|---|---|
| **Cindy SkillHub** | 宿主 `publish_skill` 工具 | `create` 时必须**显式**指定 `visibility`；先 private 测试会把名字占住，之后 `update` 改不回来 |

## 四、上传 / 表单型

| 渠道 | 入口 | 前置 | 验证 |
|---|---|---|---|
| **讯飞 Astron** | `skill.xfyun.cn/dashboard/publish` | zip（SKILL.md 在包根）、中文简介 | ✅ 实测走通 |
| **小红书 Red Skill** | 平台内 | 仅支持 Markdown 等有限格式 | △ |
| **美团 虾345** | 平台内 | 跨平台聚合型，域名未核实 | △ |

## 五、平台托管型（**必须移植，不属于"推送"**）

| 平台 | 归属 | 收费 | 分成 | 备注 |
|---|---|---|---|---|
| **扣子 Coze 技能商店** | 字节 | ✅ 按次 / 月订阅 | 创作者约 **70%** | 国内实测唯一支持 Skill 交易；付费需入驻 `code.coze.cn/merchant-management` |
| **字节 Find Skill** | 字节/火山 | ✅ | 未公开 | 与扣子并列的另一条线 |
| **智谱 AgentMore Skills 广场** | 智谱 | 免费（零 Token 安装） | — | 广场形态，多为官方/合作技能 |
| **阿里 虾小宝（JVS Claw 内置）** | 阿里 | 免费但耗算力 | — | 内置市场，需走合作 |
| **阿里云百炼 Agent Store** | 阿里 | ✅ 打赏 + API 收费 + 分润 | 未公开 | 要求交付可调用服务 |
| **月之暗面 Kimi Claw** | 月之暗面 | △ | — | 二手信源，需移植为平台形态 |

**判断要不要移植**：技能的核心动作是在**用户本机**读写文件的（存档、构建产物、本地仓库），
平台托管型读不到用户磁盘，搬过去就是另一个产品；反过来，纯文本/纯推理类技能移植成本低。

## 六、GitHub PR 型（社区列表）

| 仓库 | 外部能否开 PR | 结论 |
|---|---|---|
| **VoltAgent/awesome-agent-skills** | ✅ `all` | 可开，加一行即可 |
| trailofbits/skills-curated | ✅ `all` | 可开，但它是安全团队的 vetted 目录，非安全类技能放进去不合适 |
| 其他社区 registry | 先查 `pull_request_creation_policy` | 是 `collaborators_only` 就直接跳过 |

> 教训：CONTRIBUTING 写得再热情，也要先看 `pull_request_creation_policy`。
> 一个 `gh api repos/<owner>/<repo>` 就能省掉一次白干。

## 七、自建发现端点（不依赖任何商店审核）

仓库开 GitHub Pages，把索引放在 `docs/.well-known/agent-skills/index.json`，任何支持该约定的
agent 都能直接发现你的技能源。

**坑（一定会踩）**：Pages 默认跑 Jekyll，而 Jekyll 会跳过以点开头的目录，`.well-known` 直接 404。
必须加一个空的 `docs/.nojekyll`。

索引就是一组 `{name, description, version, source, path, install}`，`source` 与 `install`
都指向 GitHub 仓库本身（唯一真源），**不要指向任何镜像或第三方地址**。

## 八、清单会变长

每轮开工前重新扫一遍，别拿上一轮当全集：

```bash
gh search repos --topic agent-skills --topic skill-md --sort updated --limit 40
```

扫到新站先确认三件事：**域名能打开**、**有提交动作**（表单/导入/zip/PR）、**不是假表单**
（判据：提交后给回执 id 或进审核队列；只弹 toast 说"成功"的一律不算）。
