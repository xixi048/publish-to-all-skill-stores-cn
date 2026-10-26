# 踩坑清单

每条都是真实发生过的。按"症状 → 原因 → 正确做法"记，便于下次 10 秒内认出来。

## 收录相关

**提交了却永远搜不到**
原因：技能在仓库根目录 `{name}/SKILL.md`，不是 `skills/<name>/SKILL.md`。
导入器与爬虫只认后一种，而且**不报错**——它只是安静地不收录。
做法：迁移到 `skills/<name>/`，同步改 README 里所有相对路径，然后重新提交。
> 曾经有过一次反向结论（"`{name}/` 也命中扫描路径"），是错的。以
> `anthropics/skills` 的实际布局与 Agent Skills 开放规范为准：`skills/<name>/`。

**抓取型目录站一直收不到**
原因：多数抓取站只爬**公开**仓库，有的还要求 **≥2 stars** 才入库。0 star 的新仓库进不去。
做法：先在支持「从 GitHub 导入」的渠道拿到曝光，再等抓取；不要把"没被收录"当成接口问题反复重提。

## 提交接口相关

**重复提交被限流**
原因：不少平台对同一仓库设有**冷却窗口**（常见 1 小时内重复提交直接拒）。
做法：等窗口过去再提；备注里写清"上次提交针对旧路径"，让审核知道是改版不是刷量。

**上传被拒，但报错只给 "upload did not complete"**
原因：目录里有 `__pycache__/*.pyc` 之类的垃圾文件被原样打进 zip。
做法：打包/上传前 `find . -name __pycache__ -prune -exec rm -rf {} +`。

## 宿主 / 工具相关

**`publish_skill` 报 `INVALID_PARAMS`（cindy_helper）**
原因：带 `display_name` / `summary` / `tags` 时被判参数不合法；
**只传 `path` + `name` + `mode` + `visibility` 才能过**。
做法：先最简参数创建成功，再用 `mode: "update"` 补 `changelog`。

**`publish_skill` 报 `NAME_TAKEN`，而且无法自救**
原因：先做过一次 `visibility: "private"` 的测试上传，名称被自己占住了；
update **不能**改 visibility（"Updates preserve visibility, ownership and tags"）。
做法：**create 之前就想清楚 visibility**（要公开就一次到位）。
已占名时只能去 Skills 页删除那条私有记录，再重新 create。

**想删掉自己发错的 SkillHub 发布，但找不到接口**
原因：宿主 SkillHub 的工具只有 `search_skills` / `list_my_published_skills` / `publish_skill` /
`get_skill_publish_status` / `start_skill_learning`，**没有删除**。
做法：明确告诉用户到"技能"页手动删；不要承诺自己能删，也不要为绕开它去换名字重发
（会造成两条同名不同 slug 的重复记录）。

**以为已经在浏览器登录了，CLI 还是说 Not logged in**
原因：浏览器里的登录态和 CLI 的 token 是两套，登录式 CLI 各有自己的 device-code 流程。
做法：登录命令会打印一行带 `user_code` 的 URL，**直接把这个 URL 丢给浏览器打开**让用户确认，
然后把进程留在后台等（别 kill，kill 掉这次码就废了）。

## 仓库与 git 相关

**`git push` 报 `Password authentication is not supported`**
原因：全局 `url.<镜像前缀>.insteadof` 配置把 github.com 改写成第三方镜像，
而镜像地址没有可用凭据。
做法：**先把 remote 改回官方地址，再让 GitHub CLI 的凭据助手自己出面**——令牌只会发给 github.com：

```bash
git remote set-url origin https://github.com/<owner>/<repo>.git
gh auth setup-git            # 让 gh 成为 git 的凭据助手
git push -u origin main
```

⚠️ **绝不把令牌转发给镜像站。** 不要从凭据助手取出 GitHub 令牌、Base64 编码后塞进
`Authorization` 头再打第三方镜像地址——镜像是第三方，令牌一旦随请求出去就脱离你的控制，
还会被写进 shell history。
连不上就**停止推送并如实报告网络问题**，不要为了推送成功改用镜像；改走 SSH 这条正规路：

```bash
git remote set-url origin git@github.com:<owner>/<repo>.git
ssh -T -p 443 git@ssh.github.com    # 先验证 443 端口 SSH 通不通
git push -u origin main
```

**`preflight.py` 误报"缺少 LICENSE"**
原因：它用 `skill_dir.parent` 当仓库根；改成 `skills/<name>/` 布局后 parent 是 `skills/`。
做法：已修——向上找 `.git`，再退化为找 `LICENSE` / `README.md`。

**PR 打不开**
原因：目标仓库 `pull_request_creation_policy` 是 `collaborators_only`。
做法：先查这条策略，别写完文件才发现开不了 PR。

## 审核相关

**被驳回，理由是「包含海外/谷歌商店的推广与入口」**
原因：原版技能里确实带海外商店的条目、入口与注册指引。
做法：换**本仓库的裁剪版**（本版即为此而生），回复里说明"已移除海外渠道条目"。
如果驳回理由是布局、许可证、危险调用、体积，改**原版**重打包即可，**不要换版本**。

**审核索要补充材料**
做法：**加说明，不要删功能**。把"零依赖 / 无动态执行 / 写入前备份 / 目标由用户显式给出"
写成一段披露文字，比删掉脚本更容易过。

## 流程相关

- **先修仓库再提交。** 布局/许可证错的时候提交，等于把一次审核机会烧掉
- **每次提交都留回执**（提交 id / 版本号 / 审核状态）。没有回执的人工确认不算完成
- **别轮询审核状态。** pending 就是 pending
- 交付时必须把「已提交 / 待登录 / 需移植 / 已排除」分开写，
  否则用户会以为"全量推送"= 全部上线了
