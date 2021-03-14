---
title: 阿里云服务器(centos7系统) 使用nginx+uwsgi 部署python+flask项目
date: 2020-06-21 21:05:05
toc: true
tags:
   - nginx
   - uwsgi
   - web server
cover: "https://w.wallhaven.cc/full/rd/wallhaven-rddgwm.jpg"
---

# Web服务器的搭建

## 阿里云ECS(centos7) + python flask + Nginx

- [uwsgi、wsgi和nginx的区别和关系](https://blog.csdn.net/CHENYAoo/article/details/83055108)
- [CentOS7 python2升级到python3的那些坑](https://blog.csdn.net/u011244708/article/details/82915006)
- [Centos7安装Python3的方法](https://www.cnblogs.com/FZfangzheng/p/7588944.html)
- [Linux/UNIX 上安装 MySQL](https://www.runoob.com/mysql/mysql-install.html)
  
  - 选择MySQL5.7版本 8.0版本一直报错
- [CentOS7安装MySql8不能启动的问题](https://my.oschina.net/u/2455518/blog/3039411)
- [unable-to-access-mysql-after-it-automatically-generated-a-temporary-password](https://stackoverflow.com/questions/33326065/unable-to-access-mysql-after-it-automatically-generated-a-temporary-password)
  - mysql -uroot -p : 123456
  - mysql 修改 root 密码
    - ALTER USER 'root'@'localhost' IDENTIFIED BY 'test4321';
    > ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
    - https://blog.csdn.net/vv19910825/article/details/82979563
    > ERROR 1819 (HY000): Your password does not satisfy the current policy 
    
    - https://blog.csdn.net/hello_world_qwp/article/details/79551789
  
- [在本地端使用浏览器打开该云实例的公网地址:端口](https://blog.csdn.net/qq_41090453/article/details/83451321)
- [nginx + uwsgi](https://www.jianshu.com/p/61d18009b657)

<!--more-->

## ip+端口号访问

### Method 1 - 访问需ip + 端口号访问

- web应用是由Flask内置的web服务托管

```
systemctl stop nginx
kill uwsgi的所有进程
systemctl stop iptables
systemctl stop firewalld

# vi app.py
app.run(host="0.0.0.0",port=5000)

# app.run一定要设成 0.0.0.0 才可以访问

# 以http://121.199.46.37:5000/ 访问 ip+端口号
```

### Method 2 - uwsgi - 访问无需端口号，直接以阿里云服务器公网访问

- pip install uwsgi
- [uwsgi官方文档](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html)
- `run.py`文件修改成`app.run()`
- 在项目中新建文件config.ini，`uwsgi`的配置`/Sfim/src/config.ini`，编写如下配置配置如下

```
[uwsgi]

# uwsgi 启动时所使用的地址与端口
socket = 127.0.0.1:5000

# 外网访问端口，如果直接用uWSGI外网，这里由于使用了Nginx，故注释掉
http= :80

# 指向网站目录
chdir = /root/test/

# python 启动程序文件
wsgi-file = app.py

# python 程序内用以启动的 application 变量名
# app 是 app.py 程序文件内的一个变量，这个变量的类型是 Flask的 application 类
callable = app

# 处理器数
processes = 4

# 线程数
threads = 2

#状态检测地址
stats = 127.0.0.1:9191

#daemonize=/var/log/uwsgi.log
```
- 启动：uwsgi --ini config.ini
- 使用uwsgi之后，修改app.py -> app.run()
- 一定要保证代码风格是严格按照官网的
  - 即app=Flask(__name__)要放在`if __name__ = "__main__"`外面
  - `if __name__ = "__main__"`里面只能放`app.run()`
  - 不然会报错
        
    ```
    Exception: Install 'email_validator' for email validation support.
    unable to load app 0 (mountpoint='') (callable not found or import error)
    --- no python application found, check your startup logs for errors ---
    ```

    - [解决方法1](https://segmentfault.com/q/1010000014488435)
    - [解决方法2](https://segmentfault.com/q/1010000014488435)

- uwsgi运行相关应用时要加上-d参数使其在后台运行，否则断开ssh连接后uwsgi就停止运行了，-d后要加上项目的uwsgi.log文件，没有新建一个即可。例 -d ~/uwsgi.log

### Method 3 -nginx - 访问无需端口号，直接以阿里云服务器公网访问

- 先关闭uwsgi
- /etc/nginx/nginx.conf / 日志/var/log/nginx/error.log  
  ```
    #只用nginx代理的配置
    server {
         listen          80;
         server_name     47.106.218.225;  # 阿里云公网ip
 
         location / {
            proxy_pass    http://127.0.0.1:5000;      # 本机:启动端口（此处端口与项目端口一致）      
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
     }
  ```

    ```
    python app.py
    nohup python app.py >> app.out &   将运行日志输出到app.out文件
    nohup python run.py >> run.out & 
    # nohup 加在一个命令的最前面，表示不挂断的运行命令
    # & 加在一个命令的最后面，表示这个命令放在后台执行
    # ps 和 jobs
    # 区别在于 jobs 只能查看当前终端后台执行的任务，换了终端就看不见了
    # ps命令适用于查看瞬时进程的动态，可以看到别的终端的任务
    # a: 显示所有程序  u: 以用户为主的格式来显示   x: 显示所有程序，不以终端机来区分
    # fg命令 将后台中的命令调至前台继续运行
    # Ctrl + z 命令 将一个正在前台执行的命令放到后台，并且处于暂停状态
    # bg命令 将一个在后台暂停的命令，变成在后台继续执行
    tail -f app.out
    tailf app.out
    ```

### Method 4 - nginx + uwsgi - 访问无需端口号，直接以阿里云服务器公网访问

```
> config.ini
#http = :80

> /etc/nginx/nginx.conf
server {
        #ssl_protocols TLSv1.2;
        #server_tokens off;
        listen       80 default_server;
        #listen       [::]:80 default_server;
        server_name  121.199.46.37; # 阿里云公网ip
        #root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
           #proxy_pass http://127.0.0.1:5000; # 本机:启动端口(此处端口与项目端口一致)
           #proxy_set_header Host $host;
           #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           include uwsgi_params;
           uwsgi_pass 127.0.0.1:5000;
           uwsgi_param UWSGI_PYHOME /usr/bin/python; # python位置
           uwsgi_param UWSGI_CHDIR /root/test; # 项目根目录
           uwsgi_param UWSGI_SCRIPT app:app;
        }
# 跑起来
systemctl start nginx

nginx -s reload
service nginx restart  # 重启nginx服务
service nginx start # 启动服务
service nginx stop # 停止服务 

运行uwsgi服务（通过执行启动脚本运行项目）
uwsgi --ini config.ini  （启动脚本存放位置）
# 后台运行
uwsgi --ini config.ini --daemonize uwsgi.out
```

- 错误提示
    ```
    [error] 2168#0: *19 connect() failed (111: Connection refused) while connecting to upstream
    ```
    [参考资料](https://segmentfault.com/q/1010000014488435)
    ```
    > config.ini
    [uwsgi]
    socket=127.0.0.1:8080
    plugins = python
    wsgi-file=test.py
    master=true
    processes=4
    threads=2
    callable=app
    stats=127.0.0.1:9191

    > nginx.conf
    server {
        listen 80;
        server_name 111.230.140.182;
        charset utf-8;
        client_max_body_size 75M;
        location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:8080;
            #uwsgi_param UWSGI_PYTHON /usr/bin/python;  //注释掉
            #uwsgi_param UWSGI_CHDIR /home/ubuntu/project/test;  //注释掉
            #uwsgi_param UWSGI_SCRIPT test:app ;  //注释掉
        }
    }
    ```



- 注意点：
  - 不同的web application要更换不同的端口，否则会出错，即使停掉了所有服务只启动一个服务也是如此

## nginx+uwsgi+flask的参考资料

| Name | Explanation |
| ---- | ----------- |
| Flask | 一个轻量级的Python Web框架 |
| Nginx | 一个非常稳定的Web服务器 |
| uwsgi | 一个全站式的托管服务，它实现了应用服务器（支持多种编程语言）、代理、进程管理器、监视器 |

- 使用Nginx服务器托管Flask应用的安装、配置过程

- Nginx是一个提供静态文件访问的web服务，然而，它不能直接执行托管Python应用程序，而uWSGI解决了这个问题？？？

- [可以看看理论部分-使用Flask+uwsgi+Nginx部署Flask正式环境](https://www.missshi.cn/api/view/blog/5b1511a213d85b1251000000)

    > 直接使用python run.py运行服务的方式只适合本地开发
    > 线上运行时要保证更高的性能和稳定性，需要使用uwsgi进行部署

    > 使用Nginx有如下一些优点：
    > 安全：不管什么请求都要经过代理服务器，避免了外部程序直接攻击web服务器
    > 负载均衡：根据请求情况和服务器负载情况，将请求分配给不同的web服务器，保证服务器性能
    > 提高web服务器的IO性能：对于一些静态文件，可以直接由反向代理处理，不经过web服务器

- [只看了概念解释-新手的Flask+uwsgi+Nginx+Ubuntu部署过程](https://www.jianshu.com/p/5b73444eb47d)

- [一篇就弄懂WSGI、uwsgi和uWSGI的区别](https://www.jianshu.com/p/48048554f989)

# 备用资料

- [最新阿里云申请免费SSL证书实现网站HTTPS化（图文教程一）](https://yq.aliyun.com/articles/637307)

> 通过在阿里云服务器ECS(系统为centos7)上搭建Web服务器，能够实现不仅仅是在本地，而是在互联网的访问我们的网站。使用uWSGI和Nginx部署flask项目，
> 其中uWSGI一个全站式的托管服务，它实现了应用服务器(支持多种编程语言)、代理、进程管理器、监视器，帮助实现WSGI协议、Http协议等，使得开发者不再需要关注网络通信的底层实现，而花更多的时间开发上层应用。
> WSGI(Web Server Gateway Interface)服务器网关接口是Python应用程序或框架和Web服务器之间的一种接口，已经被广泛接受。只要遵照这些协议，WSGI应用(Application)都可以在任何服务器(Server)上运行。通过利用uWSGI可以使我们的web应用得到更强的并发能力。
> 通过Nginx，一个非常稳定的Web服务器和反向代理服务器，能够进一步提高并发能力和访问效率。Nginx还有如下优点：安全，不管什么请求都要经过代理服务器，避免了外部程序直接攻击web服务器；负载均衡，根据请求情况和服务器负载情况，将请求分配给不同的web服务器，保证服务器性能；提高web服务器的IO性能，对于一些静态文件，可以直接由反向代理处理和缓存，不经过web服务器。
> 用户和本项目搭建的应用程序中的交互流程为：Nginx接受来自客户端的Http请求发送给uWSGI，uWSGI处理请求并将关键信息传递给web框架flask或者web应用等，应用返回Response经由uWSGI发送给Nginx，Nginx再发送给客户端。

解决问题的实际感受：了解阿里云云服务器ECS，发现一台暴露在网络的服务器经常受到威胁和攻击，从买了他们的产品开始，每天都会有警告邮件，要按照阿里云官方的解决方法修补漏洞和加固，深入了解了Web服务器的部署，学习到了Nginx和uWSGI的知识

<details>
<summary>代码细节</summary>

## sqlalchemy的用法

## flask蓝图

## 编写index.html界面

- index.html出现的位置home.py

## Windows跑代码

- 验证邮箱的时候，服务没有开启 www.sfim.tools 改为 192.168.0.101:8080
- models里面是和数据库有关的

## windows 安装 mysql 8.0.19

- [安装教程](https://www.jb51.net/article/179326.htm)
- [mysql front 报错](https://www.cnblogs.com/MrKeen/p/12575325.html)

</details>