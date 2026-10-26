# 讯飞 Astron SkillHub · 重新提交物料

> 平台：`https://skill.xfyun.cn/publish`（SPA，需讯飞账号；接口 401，无法脚本化）
> 提交包：`dist/publish-to-all-skill-stores-cn-v1.0.1-skillroot.zip`（SKILL.md 在包根）
> 上一轮驳回原因：包内含海外 / 谷歌商店的推广与入口 + 用第三方镜像转发 GitHub 令牌。本版两条都已移除。

---

## 1. 表单字段（直接复制）

**技能名称（中文）**

```
国内技能商店上架助手
```

**技能标识 / slug**

```
publish-to-all-skill-stores-cn
```

**版本**

```
1.0.1
```

**一句话简介（≤ 60 字）**

```
把 Agent 技能体检、打包并提交到国内各 Skill 商店，逐条记录回执与审核状态。
```

**详细介绍**

```
面向国内 Skill 商店的一站式上架助手，覆盖腾讯 SkillHub、讯飞 Astron、扣子 Coze、智谱 AgentMore、
阿里百炼、字节 Find Skill、Cindy、WorkBuddy 技能中心、小红书、Kimi、美团等渠道。

流程分四步：
1. 体检 —— 本地扫描技能目录，检查布局是否符合 skills/<name>/SKILL.md、许可证是否齐全、
   是否含动态执行类危险调用、包体是否异常，报 ERROR 就停止提交。
2. 打包 —— 产出 SKILL.md 位于包根的 skillroot zip，适配所有上传型渠道。
3. 提交 —— 按渠道给出可直接照做的配方：网页导入、上传 zip、GitHub PR、自建发现端点。
4. 留档 —— 逐条记录每个渠道的回执与状态；被驳回时按驳回理由给出具体改法。

技术特性：Python 3.8+，仅使用标准库，无第三方依赖，可离线运行。
唯一的网络访问是读取用户指定的公开页面，不发送任何凭据给第三方。

许可证：MIT。
```

**分类**

```
开发工具 / DevOps
```

**标签**

```
技能商店, 上架, 发布, 技能市场, skillhub, 审核, 打包, 体检, 回执, 国内渠道
```

**开源地址**

```
https://github.com/xixi048/publish-to-all-skill-stores-cn
```

> 若该仓库尚未推送，此处留空或填个人主页；上传型渠道不校验此项。

---

## 2. 提交四步（网页）

1. 打开 `https://skill.xfyun.cn/publish`，讯飞账号登录。
2. 新建技能 → 填上表字段 → 上传 `dist/publish-to-all-skill-stores-cn-v1.0.1-skillroot.zip`。
3. 等待质量评测（平台自动跑，通常几分钟）；评测不过会给出具体扣分项。
4. 评测通过后点发布。

---

## 3. 本次驳回的两条，本版怎么处理的

| 驳回意见 | 本版处理 |
|---|---|
| 用 `gh auth token` 取令牌、Base64 后塞 `Authorization` 头向第三方镜像推送 | **整段删除**。改为 `gh auth setup-git` 或 SSH over 443 走官方地址；连接失败即停止推送并如实报告。文档里明确写「不得把令牌转发给镜像站」，并说明会留在 shell history |
| 包内含境外商店的发布指引、入口与注册项 | **全部移除**。渠道表只剩国内渠道 + GitHub 正常开发引用（仓库本身、PR 型列表、自建发现端点）。相关脚本一并删除，避免残留 |

复审时可以主动说明这两点，比等对方问更省一轮。
