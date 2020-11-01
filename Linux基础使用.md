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

# 组管理和权限管理
* 在Linux中的每个用户必须属于一个组，不能独立于组外。

## 文件/目录所有者
一般未见的创建者，谁创建了该文件，就自然成为该文件的所有者
1. 查看文件的所有者
```
ls -ahl
```

2. 修改文件的所有者
```
chown 用户名 文件名
```

3. 组的创建
```
gropuadd 组名
```

4. 修改文件所在组
```
chgrp 组名 文件名
```

5. 改变用户所在组
```
usermod -g 组名 用户名
usermod -d 目录名 用户名 改变该用户登录的初始目录
```

## 权限
* `ls -l`显示的内容如下
-rwxrw-r-- 1 root root 
1. 第0位确定文件的了类型（d,-,l,c,b）： -普通文件；d目录；l软链接；c字符设备【键盘，鼠标】；b块文件【硬盘】
2. 第1-3位确定所有者（该文件的所有者）拥有该文件的权限。 --User
3. 第4-6位确定所属组（同用户组的）拥有该文件的权限。 --Group
4. 第7-9位确定其他用户拥有该文件的权限。 --Other

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E6%9D%83%E9%99%90-%E6%96%87%E4%BB%B6%E7%9A%84%E4%BF%A1%E6%81%AF.png)

* rwx权限作用到文件：
    1. [r]：代表可读，可以读取查看
    2. [w]：代表可写，可以修改，但是不代表可以删除文件，删除文件的前提条件是对该文件所在的目录有写权限，才能删除该文件
    3. [x]：代表可执行
* rwx权限作用到目录：
    1. [r]：代表可读，可以读取，ls查看目录内容
    2. [w]：代表可写，可以修改，目录内创建和删除，重命名目录
    3. [x]：代表可执行，可以进入该目录

## 权限管理
1. chmod
修改文件或者目录的权限：u:所有者；g:所有组；o:其他人；a:所有人
```
chmod u=rwx,g=rx,o=x 文件目录名
chmod o+w 文件目录名：添加权限
chmod a-x 文件目录名：移出权限
```

2. chown
修改文件所有者
```
chown newonwer file：改变文件的所有者
chown newowner:newgroup file：改变用户的所有者和所有组
-R：如果是目录，则使其下所有子文件夹或目录递归生效
```

## 任务调度
任务调度是指系统在某个时间执行特定的命令或程序
* 任务调度分类：
    1. 系统工作：有些重要的工作必须周而复始的执行，如病毒扫描
    2. 各别用户工作，个别用户可能希望执行某些程序，比如对mysql数据的备份

1. crontab
```
crontab [选项]
常用参数：
    -e：编辑crontab定时任务
    -l：查询crontab任务
    -r：删除当前用户所有的crontab任务
```
例：设置任务调度文件：/etc/crontab，设置个人任务调度。执行`crontab -e`，接着输入任务到调度文件`* /1 * * * * * ls -l /etc/ > /tmp/to.txt`意思说每小时的每分钟执行`ls -l /etc/ > /tmp/to.txt`
* 五个占位符的说明
    1. 第一个`*`：一小时当中的第几分钟，范围0~59
    2. 第二个`*`：一天当中的第几小时，范围0~23
    3. 第三个`*`：一个月当中的第几天，范围1~31
    4. 第四个`*`：一年当中的第几月，范围1~12
    5. 第五个`*`：一周当中的星期几，范围0~7（0和7都代表星期日）
* 特殊符号的说明:
    1. *：代表任意时间。比如第一个`*`就代表一小时中每分钟都执行一次
    2. ,：代表不连续的时间，比如`0 8,12,16 * * *`，就代表在每天的8点0分，12点0分，16点0分都执行一次
    3. -：代表连续的时间范围。比如`0 5 * * 1-6`，代表在周一到周六的凌晨5点0fen执行命令
    4. */n：代表每隔多久执行一次。比如`*/10 * * * *`，代表每隔10分钟就执行一遍命令

## 磁盘分区、挂载
* 分区的方式
    1. mbr分区：
        - 最多支持四个主分区
        - 系统只能安装在主分区
        - 扩展分区要占一个主分区
        - mbr最大只支持2TB，但拥有最好的兼容性
    2. gpt分区
        - 支持无限多个主分区（但操作系统可能限制）
        - 最大支持18EB的大容量（1EB = 1024PB， 1PB = 1024TB）
        - windows7 64为以后支持gpt
* Linux分区
    1. Linux无论有几个分区，分给哪一目录使用，它归根结底就只有一个根目录，一个独立且唯一的文件结构，Linux中每分区都是用来组成整个文件系统的一部分
    2. Linux采用了一种”载入“的处理方式，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来，这时要载入的一个分区将使它的存储空间在一个目录下获得
* 硬盘
    1. Linux硬盘分为IDE硬盘和SCSI硬盘，目前基本上是SCSI硬盘
    2. 对于IDE硬盘，驱动器标识符为“hdx~”，其中“hd”表明分区所在设备的类型，这里是指IDE硬盘了。“x”为盘号（a为基本盘，b为基本从属盘，c为辅助主盘，d为辅助从属盘），“~”代表分区，前四个分区用数字1到4表示，它们是主分区或扩展分区，从5开始就是逻辑分区。例：`hda3`表示为第一个IDE硬盘上的第三个主分区或扩展分区，`hdb2`表示为第二个IDE硬盘上的饿第二个主分区或扩展分区
    3. 对于SCSI硬盘则标识为“sdx~”，SCSI硬盘是用“sd”来表示分区所在设备的类型的，其余则和IDE硬盘的表示方法一样

1. lsblk
查看系统的饿分区和挂载情况
```
lsblk -f
```
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E7%A3%81%E7%9B%98-%E7%A3%81%E7%9B%98%E5%88%86%E5%8C%BA.png)

### 挂载硬盘
给Linux系统新增加一个硬盘，并且挂载到/home/newDisk下
* 步骤:
    1. 虚拟机添加硬盘
    2. 分区 fdisk /dev/sdb
    3. 格式化 mkfs -t ext4 /dev/sdb1
    4. 挂载 先创建目录 /home/newDisk， 挂载 mount /dev/sdb1 /home/newDisk
    5. 设置可以自动挂载 vim /etc/fstab；输入/dev/sdb1     /home/newDisk      ext4      defaults    0 0(其中ext4是分区类型)

### 磁盘查询
1. df
查询系统整体磁盘使用情况
```
df -h
```

2. du
查询指定目录的磁盘占用情况
```
du -h /目录：查询指定目录的磁盘占用情况，默认为当前目录
常用参数：
    -s：指定目录占用大小汇总
    -h：带计量单位
    -a：含文件
    -max-depth=1：子目录深度
    -c：列出明细的同时，增加汇总值
```

### 磁盘命令举例
1. 统计`/home`文件夹下文件的个数
```
ls -l /home | grep "^-" | wc -l
```

2. 统计`/home`文件夹下目录的个数
```
ls -l /home | grep "^d" | wc -l
```

3. 统计`/home`文件夹下文件的个数。包括子文件夹里的
```
ls -lR /home | grep "^-" | wc -l
```

4. 统计文件夹下目录的个数，包括子文件夹
```
ls -lR /home | grep "^d" | wc -l
```

## 网络配置

### Linux网络环境配置
1. 自动获取
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E7%BD%91%E7%BB%9C-%E8%87%AA%E5%8A%A8%E9%93%BE%E6%8E%A5.png)
Linux启动后会自动获取IP，每次自动获取的IP地址可能不一样

2. 指定ip
直接修改配置文件来指定IP，并可以连接到外网，编辑`/etc/sysconfig/network-scripts/ifcfg-eth0`（不同版本的网卡配置不一样）
将BOOTPROTO改成static
ONBOOT=yes

3. 重启网络服务
```
service network restart
```

## 进程管理
1. ps
ps命令是用来查看目前系统中，有哪些正在执行，以及它们执行的情况，可以不加任何参数
* ps显示的信息选项：
    1. PID：进程识别号
    2. TTY：终端机号
    3. TIME：此进程占用CPU的总计时间
    4. CMD：正在执行的命令或进程名
```
ps -a：显示当前终端的所有进程信息
ps -u：以用户的格式显示进程信息
ps -x：显示后台进程运行的参数
```
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86-%E6%9F%A5%E7%9C%8B%E8%BF%9B%E7%A8%8B.png)

```
ps -ef：以全格式显示当前所有的进程
    -e：显示所有进程
    -f：全格式
ps -ef | grep xxx
其中：
    UID：用户ID
    PID：进程ID
    PPID：父进程ID
    C：CPU用于计算执行优先级的因子。数值越大表明进程是CPU密集型运算，执行优先级会降低；数值越小，表明进程是I/O密集型运算，执行优先级会提高
    STIME：进程启动的时间
    TTY：完整的终端名称
    TIME：CPU时间
    CMD：启动进程所用的命令和参数
```

2. kill和killall
若某个进程执行一般需要停止时，或是已经消耗了很大的系统资源的时候，此时可以考虑停止该进程，使用`kill`命令来完成停止进程
```
kill [选项] 进程号：通过进程号杀死进程
killall 进程名称：通过进程名称来杀死进程，也支持通配符，这在系统因负载过大而变得很慢时很有用
常用参数：
    -9：强迫进程立即停止
```
例：踢掉某个非法登录用户
```
ps -aux | grep sshd：查看用户登录的进程
kill 进程号
```

3. pstree
查看进程树
```
pstree [选项]：可以更加直观的来查看进程信息
常用参数：
    -p：显示进程的PID
    -u：显示进程的所属用户
```

## 服务管理
服务本质就是进程，但是是运行在后台，通常都会监听某个端口，等待其他程序的请求，因此又称为守护进程

1. service
```
service 服务名 start | stop | restart | reload | status
systemctl（CentOS7以后）
```
例：查看当前防火墙的状态，关闭防火前，重启防火墙
```
seervice iptables status：查看防火墙状态
seervice iptables stop：关闭防火墙
seervice iptables start：开启防火墙

CentOS7：
systemctl status firewalld
systemctl stop firewalld
systemctl start firewalld
```

2. chkconfig
通过chkconfig命令可以给每个运行级别设置自启动/关闭
```
chkconfig --list：查看服务
chkconfig 服务名 --list
chkconfig --leevl 5 服务名 on/off
```

## 动态监控进程
```
top [选项]
常用参数：
    -d 秒数：指定top命令每隔几秒更新，默认3秒
    -i：使top不显示任何闲置或者僵死进程
    -p：通过指定监控进程ID来仅仅监控某个进程的状态
交互指令:
    P：以CPU使用率排序，默认项
    M：以内存使用率排序
    N：以PID排序
    q：退出top
监视特定用户：
    u：然后输入用户名即可
终止指定的进程
    k：再输入要结束的进程ID号
```
top与ps命令很相似，它们都用来显示正在执行的进程。top与ps最大的不同之处在于，top在执行一段时间可以更新正在运行的进程
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/Linux/Linux%E5%8A%A8%E6%80%81%E7%9B%91%E6%8E%A7%E8%BF%9B%E7%A8%8B.png)

## 监控网络状态
```
netstat [选项]
netstat -anp
常用参数：
    -an：按一定顺序排列输出
    -p：显示哪个进程在调用
```
例：查看服务名为`sshd`的服务的信息
```
netstat -anp | grep sshd
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