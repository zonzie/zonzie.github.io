---
title: docker基本操作
date: 2018-04-16 15:59:39
tags: docker
categories: 容器
description: Docker是一个客户端-服务器（C/S）架构程序。Docker客户端只需要向Docker服务器或者守护进程发出请求，服务器或者守护进程将完成所有工作并返回结果。Docker提供了一个命令行工具Docker以及一整套RESTful API。你可以在同一台宿主机上运行Docker守护进程和客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程。
---

#### docker安装
- centos7以下版本安装太麻烦,这里只说centos7安装docker的过程
- Docker官方建议在Ubuntu中安装，因为Docker是基于Ubuntu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的。在很多版本的CentOS中是不支持更新最新的一些补丁包的。
- docker安装
    1. 通过yum安装 `yum -y install docker`
    2. 可以通过 `docker -v` 查看当前的docker版本
    3. 安装最新版的docker `curl -sSL https://get.daocloud.io/docker | sh`
    4. 开机启动：`systemctl enable docker.service/docker`
    5. 启动docker：`systemctl start docker.service`
    6. 停止docker：`systemctl stop docker.service`
    7. 重启docker：`systemctl restart docker.service`
    8. 查看docker状态：`systemctl status docker.service`
    9. 查看docker概要信息：`docker info`
    10. 查看docker帮助文档：`docker --help`

#### docker镜像文件
- Docker镜像是由文件系统叠加而成（是一种文件的存储形式）。最底端是一个文件引导系统，即bootfs，这很像典型的Linux/Unix的引导文件系统。Docker用户几乎永远不会和引导系统有什么交互。实际上，当一个容器启动后，它将会被移动到内存中，而引导文件系统则会被卸载，以留出更多的内存供磁盘镜像使用。Docker容器启动是需要的一些文件，而这些文件就可以称为Docker镜像。
- 列出docker下所有的镜像 `docker images` 这些镜像都是存储在Docker宿主机的/var/lib/docker目录下
    - REPOSITORY：镜像所在的仓库名称
    - TAG：镜像标签（version）
    - IMAGE ID：镜像ID
    - CREATED：镜像的创建日期（不是获取该镜像的日期）
    - SIZE：镜像大小
- 镜像TAG 
    - 为了区分同一个仓库下的不同镜像，Docker提供了一种称为标签（Tag）的功能。每个镜像在列出来时都带有一个标签，例如12.10、12、04等等。每个标签对组成特定镜像的一些镜像层进行标记（比如，标签12.04就是对所有Ubuntu12.04镜像层的标记）。这种机制使得同一个仓库中可以存储多个镜像。
    - 我们在运行同一个仓库中的不同镜像时，可以通过在仓库名后面加上一个冒号和标签名来指定该仓库中的某一具体的镜像，例如`docker run --name custom_container_name –i –t docker.io/ubunto:12.04 /bin/bash`， 表明从镜像Ubuntu:12.04启动一个容器，而这个镜像的操作系统就是Ubuntu:12.04。在构建容器时指定仓库的标签也是一个好习惯。
- 拉取镜像 
    - 国内访问docker hub 速度太慢,需要配置镜像加速
    - 使用ustc的镜像
        - 编辑该文件 `vim /etc/docker/daemon.json` ,如果该文件不存在就手动创建
        - 在文件中输入以下内容 
        ```
        {
            "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
        }
        ```
        - 然后重启docker `systemctl restart docker.service`
        - 通过 `docker pull` 下载镜像
            - `docker pull centos:7`
            - `docker pull ubuntu`
    - DaoCloud也提供了docker加速器，但是跟ustc不同，需要用户注册后才能使用，并且每月限制流量10GB。linux上使用比较简单，一条脚本命令搞定。
        1. 执行该命令：`curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://{your_id}.m.daocloud.io`
        2. 实际上面的脚本执行后，改的是/usr/lib/systemd/system/docker.service文件，加了个--registry-mirror参数。如果不执行上面的脚本命令，可以如下直接修改这个文件也可.<br/>
        ![image](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/daocloud.png)
        3. 然后重启配置 `systemctl daemon-reload`
        4. 重启docker服务 `systemctl restart docker`
    - 其他的镜像加速服务 阿里云|网易蜂巢,配置过程与daocloud类似

#### 启动镜像
- 启动命令 `docker run -it docker.io/centos`
- 创建容器常用的参数说明: 
    - 创建容器的命令: `docker run`
    - `-i`: 表示以"交互模式"运行容器
    - `-t`: 表示容器启动后会进入其命令行. 加入`-it`两个参数后容器启动后就能登陆进去.即分配一个伪终端
    - `--name`: 表示创建的容器名称
    - `-v`: 表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个`-v`做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
    - `-d`: 在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。 
    - `-p`：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射
- 启动之前拉取的centos7镜像并且进入容器 `docker run -it centos:7`
- 创建一个守护式的容器 `docker run -d -t ubuntu:latest /bin/bash`
- 登陆容器的方式: 
    - `docker attach container_name 或者 container_id /bin/bash` （exit，容器退出）
    - `docker exec –it container_name 或者 container_id /bin/bash`（exit，容器正常运行）
    - 查看所有的容器(启动过的): `docker ps -a`
    - 查看最后一次运行的容器: `docker ps -I`
    - 查看正在运行的容器: `docker ps`
    - 停止正在运行的容器: `docker stop $CONTAINER_NAME/ID`
    - 启动已经运行过的容器: `docker start $CONTAINEr_NAME/ID`
    - 进入守护式容器内部: `docker exec -t -i $CONTAINER_NAME /bin/bash`
    - 深入容器内部: `docker inspech $CONTAINER_NAME`
    - 删除所有的容器: ``` docker rm `docker ps -a -q` ```
    - 删除指定的容器: `docker rm $CONTAINER_ID`
    - 查看运行的容器日志: `docker logs $CONTAINER_ID`



 
