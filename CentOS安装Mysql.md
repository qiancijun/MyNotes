# 前置环境

1. 安装依赖包

    ```
    yum install -y  gcc gcc-c++ cmake ncurses ncurses-devel bison openssl-devel
    ```

2. 准备源码包

    CentOS7安装Mysql5.7需要Boost源码包

3. 卸载**mariadb**

    ```
    rpm -qa | grep -i mariadb
    rpm -e mariadb-libs-5.5.50-1.el7_2.x86_64
    # 如果上面那个没有效果，通过yum卸载
    yum remove  mariadb-libs-5.5.50-1.el7_2.x86_64
    ```

    



# 安装

1. 解压源码包到`/usr/local`目录下

    ``` 
    tar -zxvf mysql-boost-5.7.25.tar.gz -C /usr/local/
    ```

2. 编译安装

    ```
    cmake -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_BOOST=boost
    
    make && make install
    ```

    注意：如果安装过程中出现

    ```
    c++: internal compiler error: Killed (program cc1plus)
    ```

    该错误由虚拟机内存不足所引起，需要手动添加一个交换分区

    ```
    dd if=/dev/zero of=/swapfile bs=1k count=2048000 --获取要增加的2G的SWAP文件块
    mkswap /swapfile     -- 创建SWAP文件
    swapon /swapfile     -- 激活SWAP文件
    swapon -s            -- 查看SWAP信息是否正确
    echo "/var/swapfile swap swap defaults 0 0" >> /etc/fstab     -- 添加到fstab文件中让系统引导时自动启动
    ```

    编译成功后如果不想使用分区，可以选择删除

    ```
    swapoff /swapfile
    rm -fr /swapfile
    ```

    删除`CMakeCache.txt`后重新执行`CMake`

3. 修改配置文件`/etc/my.cnf`

    ```
    [mysqld]
    user=root
    port=3306
    datadir=
    basrdir=
    pid-file=
    ```

    

4. 初始化mysql

    ```
    mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql --datadir=/usr/local/data  --user=root
    ```

5. 配置服务

    ```
    cp /usr/local/mysql/support-file/mysql.server /etc/init.d/mysqld
    service start mysqld
    ```

6. 添加全局变量

    ```
    vim /etc/profile
    PATH=$PATH:路径
    export PATH
    
    source /etc/profile
    ```

7. 暴露3306端口

    ```
    firewall-cmd --zone=public --add-port=3306/tcp --permanent
    ```

    

# 用户权限

## 修改root密码

```
alter user 'root'@'localhost' identified by '密码'
```

## 设置远程连接

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '密码'；
flush privileges;
```

> 如果是阿里云需要在控制台暴露3306端口