title: dokcer基本操作(二)
tags: docker
categories: 容器
description: 'docker容器镜像的制作,以及相关操作'
date: 2018-07-19 13:23:19
---
#### docker镜像的制作
- **一般docker的镜像有两种制作方式**
	1. 通过Dockerfile和build制作镜像
	2. 通过docker commit制作镜像
- Dockerfile使用基本的基于DSL语法的指令来构建一个Docker镜像，之后使用docker builder命令基于该Dockerfile中的指令构建一个新的镜像。
- **Dockerfile常用的指令**
	1. FROM 设置镜像使用的的基础镜像
		- 语法: `FROM <image>[:<tag> | @<digest>]`
	2. MAINTAINER 设置镜像的作者
		- 语法: `MAINTAINER <name>`
	3. RUN 编译镜像时运行的脚本以及命令
		- 语法：`RUN <command>`;<br/>`RUN ["executable","param1","param2"]`
	4. CMD 设置容器的启动命令
		- 语法: `CMD ["executable","param1","param2"]`;<br/>`CMD ["param1","param2"]`;<br/>`CMD <command>`
	5. LABEL 设置镜像的标签
		- 语法: `LABEL <key>=<value> <key>=<value> …`
		- 镜像标签可以通过docker inspect查看 
	6. EXPOSE 设置镜像要暴露的端口
		- 语法: `EXPOSE <port> <port> …`
		- 提示：容器启动时，Docker Daemon会扫描镜像中暴露的端口，如果加入-P参数，Docker Daemon会把镜像中所有暴露端口导出，并为每个暴露端口分配一个随机的主机端口（暴露端口是容器监听端口，主机端口为外部访问容器的端口）
		- 注意：EXPOSE只设置暴露端口并不导出端口，只有启动容器时使用-P/-p才导出端口，这个时候才能通过外部访问容器提供的服务
	7. ENV 设置容器的环境变量
		- 语法: `ENV <key>=<value>…|<key> <value>`
		- 注意：环境变量在整个编译周期都有效，第一种方式可设置多个环境变量，第二种方式只设置一个环境变量 
		- 提示：通过${变量名}或者 $变量名使用变量，使用方式${变量名}时可以用${变量名:-default} ${变量名:+cover}设定默认值或者覆盖值 
	8. ADD 编译镜像时添加文件到镜像
		- 语法：`ADD <src>… <dest>|[“<src>”,… “<dest>”]`
		- 注意：当路径中有空格时，需要使用第二种方式, 当src为文件或目录时，Docker Daemon会从编译目录寻找这些文件或目录，而dest为镜像中的绝对路径或者相对于WORKDIR的路径
	9. COPY 编译镜像时复制文件到镜像
		- 语法：`COPY <src>… <dest>|[“<src>”,… “<dest>”]`
		- 提示：指令逻辑和ADD十分相似，同样Docker Daemon会从编译目录寻找文件或目录，dest为镜像中的绝对路径或者相对于WORKDIR的路径
	10. ENTRYPOINT 设置容器的入口程序
		- 语法：`ENTRYPOINT [“executable”,”param1”,”param2”]`, `ENTRYPOINT command param1 param2`（shell方式）
		- 提示：入口程序是容器启动时执行的程序，docker run中最后的命令将作为参数传递给入口程序, 入口程序有两种格式：exec、shell，其中shell使用/bin/sh -c运行入口程序，此时入口程序不能接收信号量, 当Dockerfile有多条ENTRYPOINT时只有最后的ENTRYPOINT指令生效, 如果使用脚本作为入口程序，需要保证脚本的最后一个程序能够接收信号量，可以在脚本最后使用exec或gosu启动传入脚本的命令 
		- 注意：通过shell方式启动入口程序时，会忽略CMD指令和docker run中的参数, 为了保证容器能够接受docker stop发送的信号量，需要通过exec启动程序；如果没有加入exec命令，则在启动容器时容器会出现两个进程，并且使用docker stop命令容器无法正常退出（无法接受SIGTERM信号），超时后docker stop发送SIGKILL，强制停止容器 
	11. VOLUME 设置容器的挂载卷
		- 语法：`VOLUME [“/data”]`, `VOLUME /data1 /data2`
		- 提示：启动容器时，Docker Daemon会新建挂载点，并用镜像中的数据初始化挂载点，可以将主机目录或数据卷容器挂载到这些挂载点
	12. USER 设置运行命令以及脚本时的用户
		- 语法：`USER <name>`
	13. WORKDIR 设置指令运行时的工作目录
		- 语法：WORKDIR <Path> 
		- 提示：如果工作目录不存在，则Docker Daemon会自动创建, Dockerfile中多个地方都可以调用WORKDIR，如果后面跟的是相对位置，则会跟在上条WORKDIR指定路径后（如WORKDIR /A   WORKDIR B   WORKDIR C，最终路径为/A/B/C）
	14. ARG 设置编译镜像时加入的参数
	15. ONBUILD	设置镜像的ONBUILD指令
		- 语法：`ONBUILD [INSTRUCTION]`
		- 提示：从该镜像生成子镜像，在子镜像的编译过程中，首先会执行父镜像中的ONBUILD指令，所有编译指令都可以成为钩子指令
	16. STOPSIGNAL 设置容器的退出信号量
- **docker build命令使用**
	- `--build-arg`，设置构建时的变量
	- `--no-cache`，默认false。设置该选项，将不使用Build Cache构建镜像
	- `--pull`，默认false。设置该选项，总是尝试pull镜像的最新版本
	- `--compress`，默认false。设置该选项，将使用gzip压缩构建的上下文
	- `--disable-content-trust`，默认true。设置该选项，将对镜像进行验证
	- `--file`, -f，Dockerfile的完整路径，默认值为‘PATH/Dockerfile’
	- `--isolation`，默认--isolation="default"，即Linux命名空间；其他还有process或hyperv
	- `--label`，为生成的镜像设置metadata
	- `--squash`，默认false。设置该选项，将新构建出的多个层压缩为一个新层，但是将无法在多个镜像之间共享新层；设置该选项，实际上是创建了新image，同时保留原有image。
	- `--tag`, -t，镜像的名字及tag，通常name:tag或者name格式；可以在一次构建中为一个镜像设置多个tag
	- `--network`，默认default。设置该选项，Set the networking mode for the RUN instructions during build
	- `--quiet`, -q ，默认false。设置该选项，Suppress the build output and print image ID on success
	- `--force-rm`，默认false。设置该选项，总是删除掉中间环节的容器
	- `--rm`，默认--rm=true，即整个构建过程成功后删除中间环节的容器
- **docker run命令使用**
	- Usage: `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`  
  	- `-d`, `--detach=false`         指定容器运行于前台还是后台，默认为false   
  	- `-i`, `--interactive=false`   打开STDIN，用于控制台交互  
  	- `-t`, `--tty=false`            分配tty设备，该可以支持终端登录，默认为false  
  	- `-u`, `--user=""`              指定容器的用户  
  	- `-a`, `--attach=[]`            登录容器（必须是以docker run -d启动的容器）
  	- `-w`, `--workdir=""`           指定容器的工作目录 
  	- `-c`, `--cpu-shares=0`        设置容器CPU权重，在CPU共享场景使用  
  	- `-e`, `--env=[]`               指定环境变量，容器中可以使用该环境变量  
  	- `-m`, `--memory=""`            指定容器的内存上限  
  	- `-P`, `--publish-all=false`    指定容器暴露的端口  
  	- `-p`, `--publish=[]`           指定容器暴露的端口 
  	- `-h`, `--hostname=""`          指定容器的主机名  
  	- `-v`, `--volume=[]`            给容器挂载存储卷，挂载到容器的某个目录  
  	- `--volumes-from=[]`          给容器挂载其他容器上的卷，挂载到容器的某个目录
  	- `--cap-add=[]`               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  	- `--cap-drop=[]`              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  	- `--cidfile=""`               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法  
  	- `--cpuset=""`                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU  
  	- `--device=[]`                添加主机设备给容器，相当于设备直通  
  	- `--dns=[]`                   指定容器的dns服务器  
  	- `--dns-search=[]`            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件  
  	- `--entrypoint=""`            覆盖image的入口点  
  	- `--env-file=[]`              指定环境变量文件，文件格式为每行一个环境变量  
  	- `--expose=[]`                指定容器暴露的端口，即修改镜像的暴露端口  
  	- `--link=[]`                  指定容器间的关联，使用其他容器的IP、env等信息  
  	- `--lxc-conf=[]`              指定容器的配置文件，只有在指定--exec-driver=lxc时使用  
  	- `--name=""`                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字  
  	- `--net="bridge"`             
		```
		容器网络设置:
        bridge 使用docker daemon指定的网桥     
        host 	//容器使用主机的网络  
        container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源  
        none 容器使用自己的网络（类似--net=bridge），但是不进行配置 
		```
    - `--privileged=false`         指定容器是否为特权容器，特权容器拥有所有的capabilities  
    - `--restart="no"`             ```指定容器停止后的重启策略:
				                no：容器退出时不重启  
				                on-failure：容器故障退出（返回值非零）时重启 
				                always：容器退出时总是重启  ```
    - `--rm=false`                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)  
    - `--sig-proxy=true`           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理  
- **示例**
```
# Dockerfile 示例
# version: 1.0.0
FROM ubuntu:latest
MAINTAINER "yaobq13@gmail.com"
# 修改apt-get源
# 备份默认的文件
RUN mv /etc/apt/sources.list ./source.list_bak
# 创建新的文件
RUN touch /etc/apt/sources.list
# 写入
RUN echo deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse >> /etc/apt/sources.list
# 更新apt-get工具
RUN apt-get update
# 下载安装nginx
RUN apt-get install -y nginx
# 安装vim
RUN apt-get install -y vim
# 写入"hello world"到文件index.html
RUN echo 'Hello World!!!' \
    >/usr/share/nginx/html/index.html
# 暴露80端口
EXPOSE 80
# ENTRYPOINT service nginx start
# 容器启动时要执行的命令,执行完成后容器会自动退出,因此使用`tail -f`命令
ENTRYPOINT nginx && tail -f /var/log/nginx/access.log
```
	- 使用Dockerfile制作镜像:`docker build -t "ubuntu_nginx:v1.0" -f "/root/static_web/Dfile" .`
	- 启动容器: `docker run -tid -p 8080:80 --name ubuntu_nginx_container ubuntu_nginx:v1.0 /bin/bash`
	- 进入容器内部: `docker exec -ti ubuntu_nginx_container /bin/bash`
	- 退出容器: `exit`
- **使用docker commit制作镜像**
	- 需要一个基础镜像,下载最新的Ubuntu的docker: `docker pull centos:7`
	- 镜像下载完毕后,启动容器 `docker run --name centos_test --privileged -t -i centos:7 /sbin/init`
	- 进入容器: `docker exec -it centos_test /bin/bash`
	- 在容器内安装vim和nginx: `yum install -y vim`,`yum install -y nginx`
	- 安装完毕后退出容器: `exit`
	- 停止容器的运行: `docker stop centos_test`
	- 将容器提交为新的镜像: `docker commit centos_test my_centos_nginx`

###### 容器的目录映射
- 在部署应用容器的时候，最好将容器的配置文件及数据目录跟宿主机目录做个映射关系！最好不要在容器内修改数据
- 启动tomcat容器, 将tomcat的配置文件和数据目录跟宿主机做个映射
	- `docker cp tomcat:/usr/local/tomcat7/webapps /mnt/`
	- `docker cp tomcat:/usr/local/tomcat7/conf /mnt/`
	- 关闭和删除容器,重新启动时,做目录映射关系
		- `docker stop tomcat`
		- `docker rm tomcat`
	- 重新启动容器
		- `docker run -t -i -d --name=tomcat -v /mnt/webapps:/usr/local/tomcat7/webapps -v /mnt/conf:/usr/local/tomcat7/conf -p 8888:8080 tomcat7 /bin/bash`
	- 进入容器启动tomcat进程或者
		- `docker exec tomcat /usr/local/tomcat7/bin/startup.sh`