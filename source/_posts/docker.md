---
title: docker常用命令
date: 2022-05-29 10:40:10
tags:
- docker
categories: 
- docker
---
## docker简介
docker是一个开源的容器引擎，可以将开发者的应用以及依赖包打包到轻量级、可移植的容器中，从而部署到Linux系统中，可以实现虚拟化操作。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低
## docker的优缺点
### 优点
``` text
1.更快的进行项目交付与部署
2.易于迁移与扩展
3.CPU/内存资源的开销少
4.环境隔离
不同的容器之间环境互不依赖，因此可以部署在同一台机器上
```
### 缺点
``` text
1.无法在32bit的linux/Windows/unix环境下使用。
2.对于磁盘的管理比较有限
``` 
## docker使用场景
``` text
1.web应用的打包与部署，例如flask环境、redis、mongdb等环境
2.自动化测试与持续集成、发布
3.任何不依赖于硬件（或可以将依赖虚拟化）的可独立的环境打包，例如安卓不同项目、不同版本的编译
```
## 容器生命周期管理命令
### run
创建一个新容器：可选参数：
* --name：设置容器的名称
* -d: 容器以后台模式运行
* -i: 交互式方式操作
* -t: 分配一个伪终端
* -p: 设置将容器的端口映射到主机的端口
* -v: 设置将主机的路径映射到容器的对应路径，当映射完成后，主机路径内容改变会影响到容器对于路径的内容改变
* /bin/bash: 在创建玩容器后直接进入docker的命令行终端

``` bash
# 使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。  
docker run --name mynginx -d nginx:latest  

# 使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。  
docker run -p 80:80 -v /data:/data -d nginx:latest  

# 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。  
docker run -it nginx:latest /bin/bash  
```
### start/stop/restart
* docker start : 启动一个或多个已经被停止的容器。
* docker stop : 停止一个运行中的容器。
* docker restart : 重启容器。
``` bash
# 启动已被停止的容器mynginx  
docker start mynginx

# 停止运行中的容器mynginx  
docker stop mynginx

# 重启容器mynginx  
docker restart mynginx 
```
### kill
杀掉一个运行中的容器。可选参数：
* -s : 发送什么信号到容器，默认 KILL
``` bash
# 根据容器名字杀掉容器  
docker kill tomcat7  

# 根据容器ID杀掉容器  
docker kill 65d4a94f7a39  
```
### rm
删除一个或多个容器
``` bash
# 强制删除容器 db01、db02：  
docker rm -f db01 db02  

# 删除容器 nginx01, 并删除容器挂载的数据卷：  
docker rm -v nginx01  

# 删除所有已经停止的容器：  
docker rm $(docker ps -a -q)
```
### create
创建一个新的容器但不启动它。
``` bash
# 使用docker镜像nginx:latest创建一个容器,并将容器命名为mynginx  
docker create --name mynginx nginx:latest     
```
### exec
在运行的容器中执行命令;也可以进入容器，可选参数
* -d : 分离模式: 在后台运行
* -i : 即使没有附加也保持STDIN 打开
* -t : 分配一个伪终端
``` bash
# 在容器 mynginx 中以交互模式执行容器内 /root/nginx.sh 脚本  
docker exec -it mynginx /bin/sh /root/nginx.sh  

# 在容器 mynginx 中开启一个交互模式的终端
docker exec -i -t  mynginx /bin/bash  

# 也可以通过 docker ps -a 命令查看已经在运行的容器，然后使用容器 ID 进入容器。  
docker ps -a   
docker exec -it 9df70f9a0714 /bin/bash  
```
### attach
当容器以后台模式运行时，需要进入容器，除了上述的exec，还有attach，但exec与attach有一个区别，当使用exec进入容器后推出，容器不会停止，但attach在退出后容器会停止，所以建议使用exec
``` bash
# 进入容器id为1e560fca3906的容器
docker attach 1e560fca3906 
```
### pause/unpause
* docker pause :暂停容器中所有的进程。
* docker unpause :恢复容器中所有的进程。
``` bash
# 暂停数据库容器db01提供服务。  
docker pause db01  

# 恢复数据库容器 db01 提供服务  
docker unpause db0  
```
## 容器操作命令
### ps
列出容器，可选参数：
* -a : 显示所有的容器，包括未运行的。
* -f : 根据条件过滤显示的内容。
* –format : 指定返回值的模板文件。
* -l : 显示最近创建的容器。
* -n : 列出最近创建的n个容器。
* –no-trunc : 不截断输出。
* -q : 静默模式，只显示容器编号。
* -s : 显示总的文件大小。
``` bash
# 列出所有在运行的容器信息。  
docker ps  

# 列出最近创建的5个容器信息。  
docker ps -n 5  

# 列出所有创建的容器ID。  
docker ps -a -q  
```
``` text
补充说明：容器的7种状态：created（已创建）、restarting（重启中）、running（运行中）、removing（迁移中）、paused（暂停）、exited（停止）、dead（死亡）。
```
## 未完，持续更新中