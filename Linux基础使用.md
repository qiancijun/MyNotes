---
title: Linux基础使用
date: 2020-10-29 23:01:44
tags:
---
# Linux

## Linux目录结构
* Linux的文件系统是采用级层式的树状目录结构，在此结构中的最上层是根目录“/"，然后在此目录下再创建其他的目录。
>在Linux世界里，一切皆是文件

# Vi和Vim
所有的Linux系统都会内建Vi文本编辑器

## Vi的三种模式
1. 一遍模式
以Vim打开一个文档就直接进入一般模式了。在这个模式中可以通过上下左右按键来移动光标，可以通过删除整行，删除字符来处理文档，也可以使用复制粘贴

2. 插入模式
按下i,I,o,O,a,A,r,R任何一个字母之后才会进入插入模式，一般按i

3. 命令行模式
在这个模式中，提供相关指令完成保存、退出、显示行号等操作

## 部分快捷键
1. 拷贝
``` 
拷贝当前行： yy
拷贝当前行向下的5行： 5yy
粘贴： p
```
2. 删除
```
删除当前行： dd
删除当前行向下的5行： 5dd
```
3. 在文件中查找某个单词
```
命令行模式下： /关键词
回车进行查找，输入n跳转到下一个，输入N跳转到上一个
```
4. 设置行号
```
命令行模式下
设置行号： :set nu
取消行号： :set nonu
```
5. 跳转到文档头或尾
```
跳转到最末行： G
跳转到最首行： gg
```
6. 撤销
```
在一个文件中输入“hello”，然后撤销这个动作： u
```
7. 跳转到指定行
```
先显示行号： :set nu
输入20，在按下shift + g
```

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/vi-vim-cheat-sheet-sch1.jpg)

# Linux常用指令

## 开机与重启
1. shutdoown
```
shutdown -h now 立刻进行关机
shutdown -h 1 一分钟后关机
shutdown -r now 立刻进行重启 
```
2. halt 
立刻关机
3. reboot
立刻进行重启
4. sync
把内存的数据同步到磁盘
>不管是重启系统还是关闭系统，首先要运行sync命令，把内存中的数据写入磁盘中

## 用户登录与注销
1. 登录时尽量少用root账号登录，因为它是系统管理员，最大的权限，避免操作失误。可以用普通用户登录，登陆后再用“su -用户名”命令来切换成系统管理员身份
2. 在终端输入logout即可注销用户
>logout注销指令在图形运行级别无效，在运行级别3下有效、

## 用户管理
用户组和家目录则是Linux设计者为了方便大家使用从而创建的概念，每一个用户都必须归属于一个组，一个用户可以归属于多个组。家目录则是用户登录时会来到目录/home下所对应的自己的家目录。
1. 添加用户的基本语法
```
useradd [选项] 用户名
把用户创建在指定的家目录下
useradd -d/[后面写你所指定的目录即可]
```
2. 给用户指定或者修改密码
```
passwd 用户名
```
3. 删除用户
```
userdel 用户名
删除用户并且保留家目录：userdel 用户名
删除用户和家目录：userdel -r 用户名 
```
>在实际开发中，一般保留家目录

4. 查询用户信息
```
id 用户名
```
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86-%E6%9F%A5%E8%AF%A2%E7%94%A8%E6%88%B7.png)

5. 切换用户
在操作Linux中，如果当前用户的权限不够，可以通过su-指令，切换到高权限用户，比如root
```
su - 切换用户名
```
>从权限高的用户切换到权限低的用户不需要密码，反之需要
当需要返回到原来用户时，使用exit指令

## 用户组
类似于角色，系统可以对有共性的多个用户进行统一的管理
1. 新增组
```
groupadd 组名
增加用户时直接加上组名
useradd -g 用户组 用户名
修改用户的组
usermod -g 用户组 用户名
```

2. 删除组
```
groupdel 组名
```

## 用户和组的配置文件
1. /etc/passwd 文件
用户（user）的配置文件，记录用户的各种信息
每行的含义：用户名:口令:用户标识号:注释性描述:主目录:登录Shell
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

2. /etc/shadow 文件
口令的配置文件
每行的含义：登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志

3. /etc/group 文件
组（group）的配置文件，记录Linux包含的组的信息
每行含义：组名:口令:组标识号:组内用户列表
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86-%E7%BB%84%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

## 运行级别
Linux为系统设置了6个运行级别
0：关机
1：单用户（找回丢失密码）
2：多用户无网络服务
3：多用户有网络（运用最多）
4：保留级别
5：图形界面
6：重启
系统的运行级别配置文件：`/etc/inittab` (CentOS6)
* 切换到指定运行级别的指令
```
init 运行级别（保留的4不要写）
```
>进入到单用户模式，然后修改root密码，即可找回丢失的root密码。进入单用户模式，root不需要密码就可以登录

## 帮助指令
当我们对某个指令不熟悉的时候，可以使用Linux提供的帮助指令
```
man [命令或配置文件]
help 指令：获得shell内置命令的帮助信息
```

## 文件目录指令
1. pwd
显示当前工作目录的绝对路径
2. ls
``` 
ls [选项] [目录或文件]
常用参数：
    -a：显示当前目录所有的文件和目录，包括隐藏的
    -l：以列表的方式显示信息 （可以简写为ll）
```
3. cd
切换到指定目录
``` 
cd [选项]
常用参数
    cd ~：回到自己的家目录
    cd ..：回到当前目录的上一级目录
```
4. mkdir
用于创建目录
```
mkdir [选项] 要创建的目录
常用参数：
    -p：创建多级目录
```
5. rmdir
删除空目录
```
rmdir [选项] 要删除的空目录
```
>rmdir删除的是空目录，如果目录下有内容是无法删除的，如果需要删除非空目录，需要使用rm -rf 要删除的目录

6. touch
创建空文件
```
touch 文件名称
```

7. cp
拷贝文件到指定目录
```
cp [选项] source dest
常用参数
    -r：递归复制整个文件夹
```

6. rm
移出文件或目录
```
rm [选项] 要删除的文件或目录
常用参数：
    -r：递归删除整个文件夹
    -f：强制删除不提示
```

7. mv
移动文件与目录或重命名
```
mv oldNameFile newNameFile：重命名
mv /temp/movefile /targetFolder：移动文件
```

8. cat
查看文件内容
``` 
cat [选项] 要查看的文件
常用参数
    -m：显示行号
```
>cat只能浏览文件，而不能修改文件，为了浏览方便，一般会带上 管道命令| more

9. more
more指令是一个基于Vi编辑器的文本过滤器，它以全屏幕的方式按页显示文本文件的内容。more指令中内置了若干快捷键
```
more 要查看的文件
快捷键：
    space：向下翻一页
    enter：向下翻一行
    q：立刻离开more，不再显示该文件内容
    Ctrl+F：向下滚动一屏
    Ctrl+B：返回上一屏
    =：输出当前行的行号
    :f：输出文件名和当前行的行号
```

10. less
less指令用来分屏查看文件内容，它的功能与more类似，但是比more指令更加强大，支持各种显示终端/less指令在显示文件内容时，并不是一次将整个文件加载之后才显示，而是根据显示需要加载内容，对于显示大型文件具有较高的效率
```
less 要查看的文件
快捷键：
    space：向下翻动一页
    [pagedown]：向下翻动一页
    [pageup]：向上翻动一页
    /字串：向下搜索[字串]的功能，n：向下查找，N：向上查找
    ?字串：向上搜索[字串]的功能，n：向下查找，N：向上查找
    q：离开less
```

11. 重定向与追加
`>`输出重定向：会将原来文件的内容副高
`>>`追加：不会覆盖，而是追加到文件的尾部
```
ls -l > 文件：列表的内容写入文件中（覆盖写）
ls -al >> 文件：列表的内容追加到文件的末尾
cat 文件1 > 文件2 将文件1的内容覆盖到文件2
echo "内容" >> 文件
```

12. echo
输出内容到控制台
```
echo [选项] [输出内容]
```

13. head
用于显示文件的开头部分，默认情况下head指令显示文件的前10行内容
```
head 文件：查看文件前10行内容
head -n 5 文件：查看文件前5行内容
```

14. tail
用于输出文件中尾部的内容，默认情况下tail指令显示文件的后10行内容
```
tail 文件：查看文件后10行内容
tail -n 5 文件：查看文件后5行内容
tail -f 文件：实时追踪该文件的所有更新
```

15. ln
软链接指令，也叫符号链接，类似于windows里的快捷方式，主要存放了链接其他文件的路径
```
ln -s [源文件或目录][软链接名]：给文件创建一个软链接
rm -rf linkToRoot：删除软链接（不要带斜杠）
```
* 例：在`/home`目录下创建一个软链接`linkToRoot`，链接到`/root`目录
```
ln -s /root linkToRoot 
```
>当使用pwd指令查看目录时，仍然看到的是软链接所在目录

16. history
查看已经执行过历史命令，也可以执行历史指令
```
history：查看已经执行过历史命令
!第n条指令：执行历史编号为n的指令
```

## 时间日期类
1. date
显示当前日期
```
date：显示当前时间
data +%Y：显示当前年份
date +%m：显示当前月份
date +%d：显示当前是哪一天
date "+%Y-%m-%d"：显示年月日
date -s 字符串时间：设置日期
```

2. cal
查看日历
```
cal [选项]：不加选项，显示本月日历
```

## 搜索查找类
1. find
find指令将从指定目录向下递归的遍历其各个子目录，将满足条件的文件或者目录显示在终端
```
find [搜索范围] [选项]
参数说明：
    -name<查询方式>：按照指定的饿文件名查找模式查找文件
    -user<用户名>：查找属于指定用户名所有文件
    -size<文件大小>：按照指定的文件大小查找文件 +n 大于 -n 小于 n 等于
```
例：根据名称查找`/home`目录下的`hello.txt`文件
```
find /home -name hello.txt
```

2. locate
locate指令可以快速定位文件路径。locate指令利用实现建立的系统中所有文件名称及路径的locate数据库实现快速定位给定文件。locate指令无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新locate时刻
``` 
locate 搜索文件
```
>由于locate指令基于数据库进行查询，所以第一次运行前，必须使用updatedb指令创建locate数据库

3. grep和管道符号 |
grep过滤查找，管道符"|"，表示将前一个命令的处理结果输出传递给后面的命令处理
```
grep [选项] 查找内容 源文件
常用参数：
    -n：显示匹配行及行号
    -i：忽略字母大小写  
```
例：在`hello.txt`文件中，查找`yes`所在行，并显示行号
```
cat hello.txt | grep -n yes
```

## 压缩和解压类
1. gzip/gunzip
gzip用于压缩文件，gunzip用于解压文件
```
gzip 文件：压缩文件，只能将文件压缩为*.gz文件
gunzip 文件.gz：解压缩文件
```
>当使用gzip对文件进行压缩后，不会保留原来的文件

2. zip/unzip
zip用于压缩文件，unzip用于解压，这个在项目打包发布中常用
```
zip [选项] XXX.zip 将要压缩的内容：压缩文件和目录
unzip [选项] XXX.zip：解压缩文件
常用参数：
    -r：递归压缩，即压缩目录（zip）
    -d<目录>：指定解压后文件的存放目录（unzip）
```

3. tar
tar是打包指令，最后打包后的文件是.tar.gz
```
tar [选项] XXX.tar.gz 打包的内容：打包目录，压缩后的文件格式.tar.gz
参数说明：
    -c：产生.tar打包文件
    -v：显示详细信息
    -f：指定压缩后的文件名
    -z：打包同时压缩
    -x：解包.tar文件
```
例：压缩多个文件，将`/home/a1.txt`和`/home/a2.txt`压缩成`a.tar.gz`
```
tar -zcvf a.tar.gz a1.txt a2.txt
```
例：将`a.tar.gz`解压到当前目录
```
tar -zxvf a.tar.gz
```
例：将`a.tar.gz`解压到`/opt/tmp`下
```
tar -zxvf a.tar.gz -C /opt/tmp
要确保/opt/tmp目录存在
```


# CentOS
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

## CentOS 7装Gnome图形界面
1. 输入下面的命令来安装Gnome包
```
yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
```

2. 用下面的命令修改启动模式
```
systemctl set-default graphical.target
```

3. 重启
```
reboot
```