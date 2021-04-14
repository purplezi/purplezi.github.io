---
title: hexo+github搭建博客 
date: 2020-03-21 12:37:28
toc: true
tags: 
  - blog
  - github
  - hexo
categories: Blogs
---

> Hexo是一个基于 Github Pages 的博客发布工具，支持Markdown格式，有众多优秀插件和主题
> github：https://github.com/hexojs/hexo
> 官网：http://hexo.io

<!-- more -->

# hexo 一些命令 & 用法

- 一般下载的主题的位置位于`hexo/themes`，切换主题的时候需要修改`hexo/_config.yml`中的`theme`
- 一般生成的文章的位置位于`hexo/source/_posts`
- 所有的命令都在`git bash`下进行，一些`hexo`的命令
  
  ```
  # hexo generate 生成静态文件 
  hexo g
  # hexo server 启动服务，在本地预览
  hexo s 
  # hexo deploy 部署网站
  # 上传到github，无需通过git add和git push，但需要配置
  hexo d
  # 新建文章
  hexo new "name"
  # 新建页面
  hexo new page "pageName" 
  ```
- 插入图片
  ```
  # hexo/_config.yml 
  post_asset_folder: true
  # hexo n 生成md博文时，/source/_posts文件夹内除了md文件还有一个同名的文件夹，不再赘述

  # 在hexo目录下，安装上传本地图片的插件，我装了会出错
  npm install hexo-asset-image --save
  # 卸载
  npm uninstall hexo-asset-image
  ```
- [The symbols count is undefined and reading time is NaN:aN](https://github.com/theme-next/hexo-symbols-count-time/issues/53)

# Github

- 部署到github上，通过hexo直接更新github仓库
  如果要通过`hexo d`直接上传，需要配置
  - `ssh key`
  - 修改`hexo/_config.yml`中有关`deploy`的部分
   
   ```yml
   deploy:
     type: git
     repository: git@github.com:purplezi/purplezi.github.io.git
     branch: master
   ```
  
  - 安装插件`npm install hexo-deployer-git --save`

# 其他主题

> 1. Fluid，基于 Hexo 的一款 Material Design 风格的主题
>    github：https://github.com/fluid-dev/hexo-theme-fluid
>    官网：https://hexo.fluid-dev.com/docs
>    配置指南：https://hexo.fluid-dev.com/docs
> 2. https://github.com/blinkfox/hexo-theme-matery
> 3. https://github.com/litten/hexo-theme-yilia 推荐用yilia-plus
> 4. https://www.zhihu.com/question/24422335?utm_source=qq&utm_medium=social&utm_oi=868617446326681600
> 5. http://ppoffice.github.io/hexo-theme-icarus
>    dark mode change: https://blog.sbx0.cn/2020/04/07/icarus-3-night-mode/

- 更换主题后出现文章的图片加载不出，找不到原因
  - 解决方法：对每篇文章进行微修改，就可以马上加载出来了
- 如果是配置gitee的Gitee Pages，不是Pro，则需要手动到Gitee Pages上更新

# 参考资料

- [hexo+Github超深度优化](https://io-oi.me/tech/hexo-next-optimization/)
- [在hexo中加入图片、音频、视频](http://chant00.com/2015/11/04/%E5%9C%A8hexo%E5%8D%9A%E5%AE%A2%E4%B8%AD%E6%8F%92%E5%85%A5%E5%9B%BE%E7%89%87%EF%BC%8C%E9%9F%B3%E4%B9%90%EF%BC%8C%E8%A7%86%E5%B1%8F%EF%BC%8C%E5%85%AC%E5%BC%8F/)

