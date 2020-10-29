---
title: Docker核心技术
date: 2020-10-21 15:03:02
tags:
---

# Docker
* Docker是基于Go语言实现的云端开源项目。
* Docker的主要目的是“Build， Ship and Run Any App Anywhere”，也就是通过对对应组件的封装、分发、部署、运行等生命周期的管理，使用户的App（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次封装，到处运行”


## Docker底层原理

* Docker架构图
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Docker/1.Docker%E7%BB%84%E6%88%90.png)

* Docker镜像
Docker镜像就是一个只读的模板。镜像可以用来创建Docker容器，一个镜像可以创建很多容器。

* Docker容器
Docker利用容器（Container）独立运行的一个或一组应用。容器是用镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
可以把容器看做是一个简易版的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。
容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的

* Docker仓库
仓库是集中存放镜像文件的场所。仓库和仓库注册服务器是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签。
仓库分为公开仓库和私有仓库两种形式

* 概括总结：
Docker本身是一个容器运行载体或称之为管理引擎。应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就似乎image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看作是容器的模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例

## Docker安装
[CentOS官方安装教程](https://docs.docker.com/engine/install/centos/)
1. 确定CentOS版本
``` 
cat /etc/redhat-release
```
2. 安装GCC
```
yum -y install gcc
yum -y install gcc-c++
```
3. 卸载旧版本
``` 
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```
4. 安装需要的软件包
``` 
sudo yum install -y yum-utils
```
5. 安装stable镜像仓库
``` 
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
6. 更新yum软件包索引
```
CentOS7
sudo yum makecache fast
CentOS8
```
7. 安装Docker CE
``` 
sudo yum -y install docker-ce
```
8. 启动Docker
```
systemctl start docker
```
9. 运行hello-world测试
``` 
docker run hello-world
```
10. 配置镜像加速（CentOS7）
```
创建级联目录 mkdir -p /etc/docker
创建配置文件 vim /etc/docker/daemon.json （与CentOS6有差异）
重启Docker systemctl restart docker.service
```
11. 使用Aliyun加速
``` 
在配置文件中替换镜像加速地址 /etc/docker/daemon.json
刷新缓存 systemctl daemon-reload
重启Docker systemctl restart docker
```

## CentOS7更换镜像源
1. 备份原来的yum源
```
sudo cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak 
```
2. 设置aliyun的yum源
``` 
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
```
3. 生成缓存
``` 
yum makecache
```

## 底层原理
Docker是一个Client-Serve结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接受命令管理运行在主机上的容器，容器是一个运行时环境
* 为什么Docker比VM快
    1. Docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化，运行在Docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上Docker将会在效率上有明显优势
    2. Docker利用的是宿主机的内核，而不需要Guest OS。因此，当创建一个容器时，Docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费事资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，返个新建过程是分钟级别的。而Docker由于直接利用宿主机的操作系统，则省略了返个过程，因此新建一个Docker容器只需要几秒钟

||Docker容器|虚拟机（VM）|
|:-|:-|:-|
|操作系统|与宿主机共享OS|宿主机OS上运行虚拟机OS|
|存储大小|镜像校，便于存储与传输|镜像庞大|
|运行性能|几乎无额外性能损失|操作系统额外的CPU、内存消耗|
|移植性|轻便、灵活、适应于Linux|笨重，与虚拟化技术耦合度高|
|硬件亲和性|面向软件开发者|面向硬件运维者|
|部署速度|快速，秒级|较慢，10s以上|


# Docker命令

## 帮助命令
* 查看Docker版本
```
docker version
```
* 查看主机信息
``` 
docker info
```
* 查看命令b
``` 
docker --help
```

## 基础指令
* 列出本地主机上的镜像
``` 
docker images
REPOSITORY：表示镜像的仓库源
TAG：镜像的标签
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小
可选参数:
    -a：列出本地所有的镜像（含中间映像层）
    -q：只显示镜像ID
    --dig：显示镜像的摘要信息
    --no-trunc：显示完整的镜像信息
```

* 搜索某个镜像名字
```
docker search
可选参数：
    --no-trunc：显示完整的镜像描述
    -s：列出收藏数不小于指定值的镜像
    --automated：只列出automated build类型的镜像
```
* 下载镜像文件
```
docker pull
例：docker pull tomcat 等价于 docker pull tomcat:latest
```
* 删除镜像文件
```
docker rmi 镜像文件名
删除全部
    - docker rmi -f $(docker images -qa)
```

## 容器命令
1. 下载镜像
有镜像才能创建容器，这是根本前提，以CentOS镜像为例
``` 
docker pull centos
```
2. 新建并启动容器
```
docker run [OPTIONS] IMAGE [COMMAND] [ARGS...]
常用OPTIONS：有些事一个减号
    --name="容器新名字"：为容器指定一个名称
    -d：后台运行容器，并返回容器ID，也即启动守护式容器
    -i：为容器重新分配一个伪输入终端，通常与-i同时使用
    -P：随机端口映射
    -p：指定端口映射，有以下四种格式
        ip：hostPort:containerPort
        ip::containerPort
        hostPort:containerPort
        containerPort
```
3. 列出当前所有正在运行的容器
```
docker ps[OPTIONS]
常用参数：
    -a：列出当前所有正在运行的容器+历史上运行过的
    -l：显示最近创建的容器
    -n：显示最近n个创建的容器
    -q：静默模式，只显示容器编号
    --no-trunc：不截断输出
```
4. 退出容器
``` 
exit 容器停止退出
ctrl + P + Q 容器不停止退出
```
5. 启动容器
``` 
docker start 容器ID或者容器名
```
6. 停止容器
```
docker stop 容器ID或者容器名称
docker kill 容器ID或者容器名称：强制停止容器
```
7. 删除已停止的容器
```
docker rm 容器ID
删除多个容器：docker rm -f $(docker ps -a -q)
             docker ps -a -q | xargs docker rm
```
8. 启动守护式容器
``` 
docker run -d
```
使用docker ps -a进行查看会发现容器已经退出
Docker容器后台运行，就必须有一个前台进程，容器运行的命令如果不是那些一直挂起的命令，就会自动退出
9. 查看容器日志
```
docker logs -f -t -tail 容器ID
常用参数
    -t：加入时间戳
    -f：跟随最新的日志打印
    --tail：显示最后多少条
```
10. 查看容器内运行的进程
``` 
docker top 容器ID
```
11. 查看容器内部细节
```
docker inspect 容器ID
```
12. 进入正在运行的容器并以命令行交互
``` 
docker exec -it 容器ID bashShell
重新进入docker attack 容器ID
区别：
    -attack：直接进入容器启动命令的终端，不会启动新的线程
    -exec：是在容器中打开新的终端，并且可以启动新的线程
```
13. 从容器内拷贝文件到主机上
``` 
docker cp 容器ID:容器内路径 目的主机路径
```

# Docker镜像
镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时的库，环境变量和配置文件
## UnionFS（联合文件系统）
Union文件系统是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
特点：一次同时加载多个文件系统，但从外面开起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

## Docker镜像加载原理
docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS
bootfs主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层时bootfs。这一层和典型的Linux/Unix系统是一样的，包含加载器和内核。当boot加载完成之后整个内核都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs

## Docker镜像Commit
```
docker commit：提交容器副本使之成为一个新的镜像
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
```

# 容器数据卷
* docker的理念：
将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的。容器之间希望有可能共享数据。Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。为了能够保存数据在docker中，可以使用卷

* 卷：
卷就是目录或文件，存在于一个或多个容器中，由dokcer挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性。
卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

* 特点：
    1. 数据卷可在容器之间共享或重用数据
    2. 卷中的更改可以直接生效
    3. 数据卷中的更改不会包含在镜像的更新中
    4. 数据卷的生命周期一直持续到没有容器使用它为止

## 添加数据卷
1. 命令添加
``` 
docker run -it -v/宿主机绝对路径目录:/容器内目录 镜像名
```

2. DockerFile添加
    1. 根目录下新建mydocker文件夹并进入
    2. 可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷
        * VOLUME["/dataVolumeContainer", “/dataVolumeContainer2”, “/dataVolumeContainer3”]
        * 出于可移植和分享的考虑，用-v 主机目录：容器目录这种方法不能够直接在DockerFile中实现
        * 由于宿主机目录是依赖于特定宿主机的，并不能够保证所有的宿主机上都存在这样的特定目录
    3. File构建
    4. build后生成镜像
    5. run容器

## 数据卷容器
命名的容器挂载数据卷，其他容器通过挂载这个（父容器）实现数据共享，挂载数据卷的容器，称之为数据卷容器
```
docker run -it --name dc02 --volumes -from dc01 centos
```

# DockerFile
* 手动编写一个DockerFile文件，必须要符合File的规范，有这个文件后，直接Docker Build命令执行，获得一个自定义的镜像。
* DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

## DockerFile体系结构
1. FROM
基础镜像，当前新镜像是基于哪个镜像的
2. MAINTAINER
镜像维护者的姓名和邮箱地址
3. RUN
容器构建时需要用到的命令
4. EXPOSE
当前容器对外暴露出的端口号
5. WORKING
指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
6. ENV
用来在构建镜像过程中设置环境变量
7. ADD
将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
8. COPY
类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到一层新的镜像内<目标路径>位置
```
COPY src dest
COPY ["src", "dest"]
```
9. VOLUME
容器数据卷，用于数据保存和持久化工作
10. CMD
指定一个容器启动时要运行的命令
DockerFile中可以由多个CMD指令，但只有最后一个生效CMD会被docker run之后的参数替换
11. ENTRYPOINT
指定一个容器启动时要运行的命令、
ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数
12. ONBUILD
当构建一个被继承的DockerFile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

# Docker常用安装
1. 在DockerHub上查找镜像
```
docker search 镜像名
```
2. 从DockHub上拉取镜像
``` 
docker pull 镜像名
```