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
![]()

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