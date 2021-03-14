---
title: Git 学习之路
date: 2020-03-21 12:40:38
toc: true
tags: 
  - git
categories: git
---

learning-git：

- https://link.zhihu.com/?target=http%3A//www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/
- https://www.zhihu.com/search?type=content&q=git%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B

# Windows 下 git clone 慢的尝试解决方法

{% tip success %}
**重启大法?!** 
重启网络，重启机器
{% endtip %}

{% tip success %}
科学上网
{% endtip %}

{% tip warning faa-horizontal animated %}
STILL SLOW
{% endtip %}

{% tip error faa-horizontal animated %}
ERROR: PRC fail
{% endtip %}

<!-- more -->

![](prc-fail.png)

# [git使用太多内存](#git%e4%bd%bf%e7%94%a8%e5%a4%aa%e5%a4%9a%e5%86%85%e5%ad%98)

- git使用太多内存，需要先`git gc`

# [源代码过于庞大](#%e6%ba%90%e4%bb%a3%e7%a0%81%e8%bf%87%e4%ba%8e%e5%ba%9e%e5%a4%a7)

- http方式不行，可以用ssh的方式([首先要进行git ssh的配置](https://blog.csdn.net/lqlqlq007/article/details/78983879))
- 需要修改git的http.postBuffer，加大git传输字节，仅对http形式有效
  
  ```git bash
  % 查看当前的配置
  git config -l
  % 加大httpbuffer
  git config --global http.postBuffer 524288000
  git config --global http.sslVerify false
  ```

# [修改host文件](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99)

- 通过[查询ip地址](https://www.ipaddress.com/)
  - `github.global.ssl.fastly.net`
  - `github.com`
  - `assets-cdn.github.com`
- 位于`C:\Windows\System32\drivers\etc`目录下的`hosts`文件
- 按如下格式，在文件末尾写入
  
  ```
  151.101.185.194 global-ssl.fastly.net
  140.82.113.3  github.com
  ```

- 刷新系统dns缓存
  
  ```
  # 可在windows的cmd下
  ipconfig /flushdns
  ```

- ip经常会变，如发现速度又下降了，及时去更换ip

# [git设置和取消代理](#git%e8%ae%be%e7%bd%ae%e5%92%8c%e5%8f%96%e6%b6%88%e4%bb%a3%e7%90%86)

## [vpn的情况下](#vpn%e7%9a%84%e6%83%85%e5%86%b5%e4%b8%8b)

- 开全局代理的情况下，利用代理进行下载，对https有效，ssh无效
  ```git bash
  git config --global http.proxy http://127.0.0.1:自己的端口号
  git config --global https.proxy https://127.0.0.1:自己的端口号

  # 可以在计算机的设置->代理查看

  # 取消方式
  git config --global --unset http.proxy
  git config --global --unset https.proxy

  # 查看目前所有的配置
  git config --global list 
  ```
- 使用全局代理，clone国内仓库慢，改进：只对github进行代理，对国内的仓库不影响
  ```
  # 先取消全局代理，通过上面的取消方法

  git config --global http.https://github.com.proxy https://127.0.0.1:自己的端口号
  git config --global https.https://github.com.proxy https://127.0.0.1:自己的端口号

  # 取消
  git config --global --unset http.https://github.com.proxy
  git config --global --unset https.https://github.com.proxy
  ```

# [关于git其他问题](#关于git其他问题)

- [“fatal: HttpRequestException encountered.” Error with GitHub/Bitbucket Repositories due to dropping TLS-1.0 support](https://stackoverflow.com/questions/49067062/fatal-httprequestexception-encountered-error-with-github-bitbucket-repositor)

# [参考资料](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99)

- [git设置和取消代理](https://www.cnblogs.com/rockbean/p/12017010.html)
- [谜之问题](https://github.com/hexojs/hexo/issues/4172)
  
  ![](ERROR.png)

  可能是：短时间过多请求api造成的

  解决：
  `hexo\themes\next\scripts\events\index.js`

  将[作者修改的部分](https://github.com/theme-next/hexo-theme-next/commit/9b543ddacf21a59f2daa52b0ae64075aed62fca2)做注释

  ![](error-fix.png)