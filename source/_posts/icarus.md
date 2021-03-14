---
title: icarus-一个hexo主题美化过程
date: 2021-01-15 18:43:21
tags:
- hexo
- blog
cover: false
hide: true
---

- 添加背景
  
  - `hexo\themes\icarus\source\css`
    
    背景图片位于`hexo\themes\icarus\source\img`
    ```styl
    body
    background: url(/img/gallery/3.jpg) no-repeat;
    background-attachment: fixed;
    background-size: cover;
    ```
- 调整背景透明度

  - `themes\icarus\source\js\animation.js`

- [在hexo博客中加入豆瓣读书功能](https://www.jianshu.com/p/2c2726da64a3?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)