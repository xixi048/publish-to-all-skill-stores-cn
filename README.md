# publish-to-all-skill-stores-cn

> 国内技能商店上架 · 合规裁剪版。**只在原版被驳回、且理由是「含海外/谷歌商店推广与入口」时使用。**

## 什么时候用这个版本

| 情况 | 用哪个 |
|---|---|
| 还没提交过 | **原版** [publish-to-all-skill-stores](https://github.com/xixi048/publish-to-all-skill-stores)。国内大部分渠道都能直接提交原版 |
| 原版被驳回，理由是布局 / 许可证 / 危险调用 / 体积 | **改原版**，不换版本 |
| 原版被驳回，理由是含海外 / 谷歌商店的推广与入口 | **本版** |
| 平台做形式审查，要求"无第三方商店推广" | **本版** |

本版相对原版只做减法：移除全部海外商店条目与链接、移除所有向第三方镜像转发令牌的推送方式、
只保留国内渠道与 GitHub 正常开发引用。

## 内容

```
skills/publish-to-all-skill-stores-cn/
├── SKILL.md                        什么时候用本版 + 提交顺序 + 硬性要求
├── references/
│   ├── cn-stores.md                国内渠道清单（域名 / 登录 / 验证等级）
│   ├── submission-recipes.md       GitHub 导入、讯飞四步、上传型、PR 型、SPA 逆向
│   ├── cn-paid-channels.md         国内付费渠道与分成
│   └── pitfalls.md                 踩坑：症状 → 原因 → 做法
└── scripts/
    ├── preflight.py                提交前体检（布局 / 许可证 / 危险调用 / 网络）
    └── build_package.py            打包成各渠道要的形态（skillroot zip）
```

## 用法

```bash
# 1. 体检 —— 报 ERROR 就别提交
python skills/publish-to-all-skill-stores-cn/scripts/preflight.py <repo>/skills/<name>

# 2. 打包 —— 上传型渠道要 zip 且 SKILL.md 在包根
python skills/publish-to-all-skill-stores-cn/scripts/build_package.py <repo> skills/<name> --out dist
```

## 要求

Python 3.8+，只用标准库。除你指定的公开端点外不发起网络请求。

## License

MIT — see [LICENSE](LICENSE).
