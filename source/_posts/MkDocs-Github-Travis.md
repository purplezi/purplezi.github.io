---
title: MkDocs+Github+Travis 搭建博客
date: 2020-03-21 12:38:41
toc: true
tags: 
  - blog
  - MkDocs
  - github
  - Travis
categories: Blogs
---

最近是不是玩博客玩疯了

> 大概逻辑：通过 Travis CI 部署的 MkDocs 环境生成静态页面并且发布到 GitHub 上
> 能够通过https://xxx.github.io/<仓库名>访问
> MkDocs 官网
> 1. https://squidfunk.github.io/mkdocs-material/
> 2. https://www.mkdocs.org/
> 
> Travis 官网 https://travis-ci.org

<!-- more -->

# Github

- 建立仓库后，`git clone`仓库到本地

# MkDocs

## 使用简介 [最好看文档]

### 安装

```bash
# 查看python和pip版本
python --version
pip --version
# Installing and verifying MkDocs 
# Material requires MkDocs >= 1.0.0.
pip install mkdocs && mkdocs --version
# install material
pip install mkdocs-material
```

### 使用

```git
# 创建文档库
# xxx是目录名称 会生成docs文件夹和mkdocs.yml
mkdocs new xxx
# 启用服务
mkdocs serve
# 编译构建站点
mkdocs build 
```

**至于页面的样式，看文档更改 mkdocs.yml 文件**

## 部署到 Github

- 将克隆下来的仓库切到`master`主分支
- 将上述产生的`docs文件夹和mkdocs.yml`放入该git目录下
- 自动编译并发布至`Github gh-pages`分支
  ```git
  mkdocs gh-deploy --clean
  # 可以 mkdocs serve 看看是否能够正常显示
  ```
- 将本地的仓库同步远程
  ```git
  git add -A
  git push
  ```
- 通过`https:xxx.github.io/仓库名称`访问

# Travis 

- 持续集成服务 ~~看起来像是监控每一次的提交修改的情况~~
- [直接附上教程](https://flc.io/more/github-travis-mkdocs-document/)

# References

- [使用 MKdocs 搭建个人博客并发布在 Github Pages 上](https://www.jianshu.com/p/b07dc1fd4f9e)
- [基于 mkdocs-material 搭建个人静态博客](https://cloud.tencent.com/developer/article/1468110)