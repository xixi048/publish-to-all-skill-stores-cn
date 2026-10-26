# 提交配方（国内渠道）

每个渠道的"提交动作"其实只有几种形状。先认清形状，再抄对应配方。

## 配方 0 · 把仓库改成"可被导入 / 可被抓取"的样子

```text
<repo>/
├── skills/
│   └── <name>/
│       ├── SKILL.md            # 必须在这个路径
│       ├── references/         # 可选
│       └── scripts/            # 可选
├── LICENSE                     # MIT / Apache-2.0
└── README.md                   # 含"怎么拿到这个技能"
```

补上 GitHub 元数据（影响抓取与搜索排序）：

```bash
gh api -X PUT repos/<owner>/<repo>/topics -f names[]=agent-skills -f names[]=skill-md
```

README 里写清安装方式。抓取站普遍直接拿 README 当详情页，可读性直接影响收录后的展示质量。

## 配方 1 · 腾讯 SkillHub（从 GitHub 导入，实测）

1. `发布 Skill` → 选「发布 Skill」（最快上架），**不要**选「发布 Pay Skill」（需支付改造）
2. 类型页切到 **「从 GitHub 导入」** 标签（默认是「本地上传」）
3. 绑定过 GitHub 后，它会自动列出你仓库里所有 `skills/<name>/`，勾选要发布的
4. 「下一步：确认信息」→ 核对 slug / 显示名 / 版本号 / 描述 → 点「发布」
5. 落在 `/dashboard`「我的 Skills」，状态 `安全审核中`

注意：点首页的「发布 Skill」按钮可能先跳 `/dashboard/verify`（实名+人脸）；
直接进 `/dashboard/publish` 可以绕开那个引导页。
**实名认证与绑定 GitHub 必须用户本人做，不要代填身份信息。**

## 配方 2 · 讯飞 Astron SkillHub（上传 ZIP，实测）

真注册中心，走「上传 ZIP → 建草稿 → 质量评测 → 发布」四步，产出正式版本号（形如 `v20261025.230752`）。

1. 登录 `passport.xfyun.cn`（微信扫码 / 手机号 / 密码）；未登录时发布页不可用
2. 正确入口是 **`/dashboard/publish`**；`/publish` 是 404
3. 上传 zip（`build_package.py` 打的 skillroot 包，≤5 MB 级别没问题），选**命名空间**（个人号只有 `Global (@global)`）
4. 「创建草稿并继续」→ 步骤 1 填**中文简介** → 保存 →「继续质量评测」
5. 步骤 2 用默认 Auto 快速评测（发布后异步跑，不阻断）→「继续发布」
6. 步骤 3 点「发布」→ 状态变 `ACTIVE · 审核中`，后台「我的技能」可见

坑：

- 命名空间下拉是异步渲染的，`click()` 之后要**另一次调用**再点 `[role=option]`，否则选不上、建草稿按钮无效
- 「继续质量评测 / 继续发布」按钮点完页面不跳转时，先点一次「保存」再点
- 简介字段是 `#candidate-summary`，React 受控组件，要用原生 value setter + `input` 事件写入

**被驳回时的处置**：如果驳回理由落在「包含海外/谷歌商店的推广与入口」，换交本仓库的裁剪版
（本版即为此而生）；如果理由是布局、许可证、危险调用、体积，改**原版**并重新打包即可，
不要换版本。

## 配方 3 · 上传型（Cow 类 / 表单型）

要求「zip 且 SKILL.md 在包根」的渠道，用：

```bash
python scripts/build_package.py <repo> skills/<name> --out dist
```

打包前删掉 `__pycache__`——`.pyc` 会被原样打进 zip，上传失败通常只回一句
"upload did not complete"，看不出是垃圾文件导致的。

## 配方 4 · GitHub PR 型

```bash
gh repo fork <owner>/<repo> --clone=false
# 本地加文件，按对方 CONTRIBUTING 的路径
gh pr create --repo <owner>/<repo> --head <me>:<branch> --title ... --body ...
```

细节：

- 先读 `CONTRIBUTING.md`，它决定的不是格式而是**能不能被合并**
- 先查 `pull_request_creation_policy`：`collaborators_only` 的仓库外部开不了 PR，直接跳过
- PR 正文按对方模板写，比长篇自我介绍有用

## 配方 5 · 把网页表单当 SPA 逆向（找不到接口时）

1. `curl` 提交页，grep 静态 `<form>` → 拿到 multipart 字段名
2. 试 `/openapi.json`、`/api/docs` → 拿到完整协议
3. 框架线索：SvelteKit 看 `__sveltekit_*` 与 `_app/immutable/entry/app.js` 里的
   `import("../nodes/N.*.js")` 路由映射；Next.js 看 `_next/static/chunks`
4. 在 chunk 里 grep `/api/`，打印每个命中位置前后 900 字符 → 请求体结构全暴露

挖到的接口立刻写进 `references/cn-stores.md`，下次直接用。

## 网络与推送（重要）

- remote 必须是官方地址：`https://github.com/<owner>/<repo>` 或 `git@github.com:<owner>/<repo>`
- 凭据交给 GitHub CLI 的凭据助手，**令牌只发给 github.com**：

```bash
git remote set-url origin https://github.com/<owner>/<repo>.git
gh auth setup-git
git push -u origin main
```

- 直连不通就改 SSH（443 端口）：`ssh -T -p 443 git@ssh.github.com` 先验通，再
  `git remote set-url origin git@github.com:<owner>/<repo>.git`
- **不要把令牌塞进 `Authorization` 头发给任何第三方镜像。** 连不上就**停止推送并如实报告**，
  不要为了推送成功绕道——令牌一旦转发出去就脱离你的控制，还会留在 shell history 里
- 提交商店时永远用**原始**仓库地址，镜像地址他人拉不到
- Git Bash 的 `/tmp` 与 Windows 的 `C:\tmp` 不是同一个目录；临时文件统一写到固定目录
