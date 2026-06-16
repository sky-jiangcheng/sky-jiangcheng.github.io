---
title: 解决一个Hexo deploy报错问题
date: 2023-06-14 00:00:00
categories:
  - Hexo
tags:
  - Hexo
  - Hexo deploy
description: 记录在 Termux 环境中部署 Hexo 博客到 GitHub Pages 时遇到的 SSH 认证问题及其排查解决过程。
---

## 解决一个Hexo deploy报错问题

部署hexo博客到github时报错如下：

```
~/jiangcheng-pages $ hexo d -g
INFO  Validating config
INFO  Start processing
WARN  ===============================================================
WARN  ========================= ATTENTION! ==========================
WARN  ===============================================================
WARN   NexT repository is moving here: https://github.com/theme-next
WARN  ===============================================================
WARN   It's rebase to v6.0.0 and future maintenance will resume there
WARN  ===============================================================
INFO  Files loaded in 803 ms
INFO  0 files generated in 55 ms
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
fatal: unable to access 'https://github.com/sky-jiangcheng/sky-jiangcheng.github.io/': The requested URL returned error: 403
FATAL Something's wrong.
```

按网上很多网友推荐的不断的重试hexo d命令并没有解决问题，猜想可能是ssh公钥不匹配的问题，然后比较了本地id_rsa.pub与github上个人配置的sshkey，发现名字一样，但是隐约感觉有什么不对，我的服务器是termux，之前只有使用ssh登陆termux上的记录，所以这个公钥是其他服务器上粘贴过来的，于是删除这个公钥并重新创建了一个秘钥对，解决了问题，命令如下：

```
~/jiangcheng-pages $ ls ~/.ssh/
authorized_keys  id_rsa.pub  known_hosts
~/jiangcheng-pages $ rm -f ~/.ssh/id_rsa.pub
~/jiangcheng-pages $ ssh-keygen -t rsa -C "jiangcheng1806@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/data/data/com.termux/files/home/.ssh/id_rsa):
```

创建好本地的id_rsa.pub之后配置到github个人ssh key配置，之后执行命令，问题解决：

```
~/jiangcheng-pages $ hexo d -g
INFO  Validating config
INFO  Start processing
INFO  Files loaded in 815 ms
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
On branch master
nothing to commit, working tree clean
```

## 问题根因分析

出现 403 错误的原因是 SSH 公钥不匹配：

1. Termux 环境中的 `id_rsa.pub` 是从其他服务器复制过来的
2. 本地私钥和 GitHub 上配置的公钥不是一对，导致认证失败
3. 重新生成密钥对并配置到 GitHub 后认证通过

## 经验总结

1. 每个设备应使用独立的 SSH 密钥对，不建议直接复制
2. 使用 `ssh -T git@github.com` 可以快速测试 SSH 认证是否正常
3. 建议使用 `git remote set-url origin git@github.com:user/repo.git` 将远程地址切换为 SSH 协议
