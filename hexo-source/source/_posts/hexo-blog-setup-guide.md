---
title: Hexo 博客搭建与 GitHub Pages 部署完整指南
date: 2026-05-28 11:00:00
categories:
  - Hexo
tags:
  - Hexo
  - 博客
  - 部署
description: 从零开始搭建 Hexo 博客，配置 Pure 主题，集成搜索功能，最终部署到 GitHub Pages 的完整流程。
---

## 前言

Hexo 是一个快速、简洁且高效的静态博客框架。配合 GitHub Pages 可以实现零成本的个人博客托管。本文记录了完整的搭建过程。

## 1. 环境准备

确保安装了 Node.js (推荐 v18+) 和 Git：

```bash
node --version
git --version
```

全局安装 Hexo CLI：

```bash
npm install -g hexo-cli
```

## 2. 初始化项目

```bash
hexo init my-blog
cd my-blog
npm install
```

初始化后的目录结构：

```
my-blog/
  _config.yml      # 站点配置文件
  package.json     # 依赖配置
  scaffolds/       # 模板文件夹
  source/          # 源文件目录
    _posts/        # 文章目录
  themes/          # 主题目录
```

## 3. 配置站点信息

编辑 `_config.yml`：

```yaml
title: My Tech Blog
subtitle: ''
description: '记录技术成长的点滴'
author: Your Name
language: zh-CN
timezone: 'Asia/Shanghai'
url: https://yourname.github.io
```

## 4. 安装 Pure 主题

```bash
git clone https://github.com/cofess/hexo-theme-pure.git themes/pure
```

修改 `_config.yml` 中的主题配置：

```yaml
theme: pure
```

安装搜索插件：

```bash
npm install hexo-generator-json-content --save
```

## 5. 创建文章

```bash
hexo new "我的第一篇博客"
```

Markdown 文件头部使用 Front-matter 定义元数据：

```markdown
---
title: 我的第一篇博客
date: 2026-05-28 10:00:00
categories:
  - 技术
tags:
  - Hexo
  - 博客
---
```

## 6. 本地预览

```bash
hexo server
# 访问 http://localhost:4000
```

## 7. 部署到 GitHub Pages

安装部署插件：

```bash
npm install hexo-deployer-git --save
```

配置部署信息：

```yaml
deploy:
  type: git
  repo: https://github.com/yourname/yourname.github.io
  branch: master
```

执行部署：

```bash
hexo clean && hexo generate && hexo deploy
```

简化命令：

```bash
hexo d -g
```

## 8. 常见问题

### 部署报错 403

通常是因为 SSH Key 配置不正确，检查本地 SSH Key 是否已添加到 GitHub：

```bash
ssh -T git@github.com
```

### 主题样式不生效

执行 `hexo clean` 清除缓存后重新生成：

```bash
hexo clean && hexo g
```

## 总结

Hexo + GitHub Pages 的组合适合个人博客和技术文档站点。Markdown 写作体验流畅，生成的是纯静态文件，访问速度快且无需维护服务器。Pure 主题简洁美观，支持搜索、分类、标签等功能。
