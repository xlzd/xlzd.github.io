---
title: 在多台机器间同步 Hexo 配置和数据
date: 2017-11-30 20:07:09
tags: hexo
---

把博客迁移到 hexo 后，开始考虑如何同步 hexo 的配置和数据，以便在多台电脑上无痛无缝切换。

解决思路：GitHub。

如果需要定制 theme，最好将对应的 theme fork 到自己的 GitHub 中，对 theme 的修改都同步到自己 fork 的仓库中去。方便管理自己对 theme 的修改。

首先在 hexo 根目录下初始化仓库，并将 themes 中对应的 theme 通过 git submodule 管理：

```bash
$ cd xlzd.github.io
$ git init
$ git checkout -b hexo-source
$ git add .
$ git commit -m "init hexo source"
$ git remote add origin git@github.com:xlzd/xlzd.github.io.git
$ git submodule add git@github.com:xlzd/hexo-theme-freemind.386.git themes/freemind.386
$ git push origin hexo-source
```

---

在另一台机器上：

```bash
$ git clone git@github.com:xlzd/xlzd.github.io.git
$ cd xlzd.github.io
$ git checkout hexo-source
$ git submodule init
$ git submodule update
$ npm install
$ hexo generate && hexo server
```
