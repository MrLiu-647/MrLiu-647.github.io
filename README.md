# MrLiuQ's Blog

基于 [Hexo](https://hexo.io/) 与 [hexo-theme-reimu](https://github.com/MrLiu-647/hexo-theme-reimu) 的博客源码。

## 本地开发

```bash
nvm use 22
npm install
npx hexo server
```

打开 http://localhost:4000/ 预览。

## 写文章

```bash
npx hexo new "文章标题"
```

文章位于 `source/_posts/`，常用 front matter：

```yaml
---
title: 文章标题
date: 2026-10-17 20:00:00
categories:
  - iOS
tags:
  - Swift
banner: /images/banner.webp
cover: false
sticky: false
---
```

## 目录结构

```text
source/_posts/    文章
source/_data/     头像、封面等站点数据
source/about/     关于页
source/friend/    友链页
source/assets/    文章引用的静态资源
themes/reimu/     主题（git submodule）
_config.yml       站点配置
_config.reimu.yml 主题个性化配置
```

## 部署

推送到 `master` 后，GitHub Actions 会自动构建并发布到 GitHub Pages。

仓库需要在 **Settings → Pages → Source** 中选择 **GitHub Actions**。

## 自定义域名

把域名写入 `source/CNAME`，并在域名服务商处配置 CNAME 记录指向 `mrliu-647.github.io`。

## 更新主题

```bash
git submodule update --remote themes/reimu
```

## 备份

迁移前的 Jekyll 站点保存在分支 `jekyll-backup` 和 tag `pre-hexo-2026-10-17` 中。
