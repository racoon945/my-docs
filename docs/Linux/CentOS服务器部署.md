## 查看服务器上的启用端口和服务

### `netstat -tlnp`命令

既可以查看端口号占用又可以查服务启动情况 (  需要root 权限才能查所有 )

------

## MySQL 命令

> 安装离线MySQL遇到加载依赖报错
>
> root@bigdata-159:/usr/local/mysql# ./bin/mysqld -- defaults-file=/etc/my.cnf --initialize --user=mysql
> ./bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
>
> 解决方法：
>
> [root@example.com data]# yum install -y libaio  //安装后在初始化就OK了

### 一、 mysql 远程登陆命令

 `mysql -h 127.0.0.1 -u username -p password -P 3306`



### 二、 添加 mysql 数据库访问权限

 mysql 的数据库是默认没开启访问权限的，默认情况下只允许localhost和127.0.0.1访问

`grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码' `

```mysql
# 修改密码 mysql 5.7 可用
update mysql.user set authentication_string=PASSWORD('newPassword') where user='root' and host='localhost';

# 赋予访问权限
grant all privileges on *.* to root@"%" identified by 'abc';
flush privileges;

# 如果不知道自己的公网ip，百度一下ip,就可以看到了!，有时还会错,根据报错信息调整即可，不过好像公网Ip会被还是什么的，可以配成识别ip网段的就不会有问题了，
GRANT ALL ON *.* to root@'119.143.12.%' IDENTIFIED BY 'password';   %表示通配的意思

# 如配置完还是无法远程访问 ,请检查如下配置
# /etc/mysql/mysql.conf.d/mysqld.cnf 下面的 bind-address = 127.0.0.1 (把它注释掉)
```



### 三、 导出数据库

用mysqldump命令（注意mysql的安装路径，即此命令的路径）：

1、导出数据和表结构：
mysqldump -u用户名 -p密码 数据库名 > 数据库名.sql
`#/usr/local/mysql/bin/   mysqldump -uroot -p abc > abc.sql`
敲回车后会提示输入密码

2、只导出表结构
mysqldump -u用户名 -p密码 -d 数据库名 > 数据库名.sql
`#/usr/local/mysql/bin/   mysqldump -uroot -p -d abc > abc.sql`

注：/usr/local/mysql/bin/  --->  mysql的data目录



### 四、 导入数据库

1 、 首先建空数据库
`mysql>create database abc;`

2 、 导入数据库

方法一： 

（1）选择数据库
`mysql>use abc;`

（2）设置数据库编码
`mysql>set names utf8;`

（3）导入数据（注意sql文件的路径）
`mysql>source /home/abc/abc.sql;`

方法二： *(推荐方法一 , 如搭建`MySQL`时没有指定数据库默认的编码为`utf-8`易造成数据库中文乱码 )*

mysql -u用户名 -p密码 数据库名 < 数据库名.sql
` mysql -uroot -p abc < abc.sql`



### 五、 `too much connection` 解决办法

```mysql
# 修改最大连接数

set persist max_connections = 200;  # 可永久有效

set GLOBAL max_connections = 200; # 5.7版本使用 , 重启后失效
```

[mysql8.0 解决办法, 点击我](https://blog.csdn.net/YoFog/article/details/82022687)

> ## MySQL 修改最大连接数限制
>
> **一、查看最大连接数**
>
> ```
> mysql> show variables like 'max_connections';
> ```
>
> **二、修改最大连接数**
> vim 编辑 /etc/mysql/mysql.conf.d/mysqld.cnf （Ubuntu）
> vim 编辑 /etc/my.cnf （CentOS）
>
> **编辑以下参数**
>
> ```
> max_connections = 10000
> ```
>
> **重启 MySQL 服务，再查看连接数是否修改**
>
> ```
> service mysql restart
> ```
>
> 或（上下两条命令都可用）
>
> ```
> systemctl restart mysql
> ```
>
> **三、临时修改 MySQL 最大连接数，重启失效**
>
> ```
> msyql > set global max_connections=1000;
> ```

### 六 、查看表结构

> 推荐使用 (Terminal 中结尾使用`\G` 优化展示效果)
>
> ```mysql
> # 查看完整建表语句
> show create table @tableName;
> ```
>
> ```mysql
> # 按表格展示字段详细信息 带注释 Comment
> show full fields from @tableName;
> ```

####  DESCRIBE：以表格的形式展示表结构

`DESCRIBE/DESC` 语句会以表格的形式来展示表的字段信息，包括字段名、字段数据类型、是否为主键、是否有默认值等，语法格式如下：

`DESCRIBE <表名>;`  或简写成：`DESC <表名>;`

```mysql
mysql> DESCRIBE tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```



#### SHOW CREATE TABLE：以SQL语句的形式展示表结构

`SHOW CREATE TABLE` 命令会以 `SQL` 语句的形式来展示表信息。和 `DESCRIBE` 相比，`SHOW CREATE TABLE` 展示的内容更加丰富，它可以查看表的存储引擎和字符编码；另外，你还可以通过\g或者\G参数来控制展示格式。

```mysql
mysql> SHOW CREATE TABLE tb_emp1\G
*************************** 1. row ***************************
       Table: tb_emp1
Create Table: CREATE TABLE `tb_emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.03 sec)
```



---



## Linux 命令

> 1. `mv` 即可以用来**移动文件** , 又可以用来**重命名文件**
>
> 2. nohup = No HangUP   后台运行 jar 包并将标准错误`stderr`和标准输出`stdout`合并后添加到`log.txt`文件末尾
>
>    If standard input is a terminal, redirect it from /dev/null.
>    If standard output is a terminal, append output to 'nohup.out' if possible,
>    '$HOME/nohup.out' otherwise.
>    If standard error is a terminal, redirect it to standard output.
>    To save output to FILE, use 'nohup COMMAND > FILE'.
>
>    `nohup java -jar xxx.jar >> log.txt 2>&1 &`
>
> 3. `alias ll='ls -l --color=tty'` 设置命令别名
>
>    用法：alias [-p] [name[=value] ... ]    注意‘=’和字符串之间不能包含空格
>
>    显示当前设置的别名：
>
>    ```
>    alias l.='ls -d .* --color=tty'
>    alias ll='ls -l --color=tty'
>    alias ls='ls --color=tty'
>    alias vi='vim'
>    alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
>    ```
> ```
>    
>    4. `chown -R racoon:racoon ./mydata`  给当前目录下的`/mydata` 目录及其下所有文件添加`racoon:racoon`用户组权限
> ```

###  Linux 常用网络测试命令

#### Ping

ping + IP(域名)

`ping 192.168.1.175`  查看 ip 192.168.1.175 是否能连通

#### Telnet

Telnet ip 端口    

`telnet 192.168.1.175 8080 `  查看 ip 192.168.1.175 的端口号8080 是否能连通 

即网址 192.168.1.175:8080

>  Escape character is ‘^]’这个意思是ctrl键+]这个键，输入 `ctrl +] `可以进入telnet交互命令行
>  `crtl + ]  `后 输入 `quit` 即可退出

### Linux 常用查看内存命令

#### Free

以MB显示内存使用状态：free -m
```shell script
# Mem 行(第二行)是内存的使用情况。
# Swap 行(第三行)是交换空间的使用情况。
# total 列显示系统总的可用物理内存和交换空间大小。
# used 列显示已经被使用的物理内存和交换空间。
# free 列显示还有多少物理内存和交换空间可用使用。
# shared 列显示被共享使用的物理内存大小。
# buff/cache 列显示被 buffer 和 cache 使用的物理内存大小。
# available 列显示还可以被应用程序使用的物理内存大小。

# 20200314 使用free -h输出结果会更友好
➜  mall git:(master) ✗ free -m
              total        used        free      shared  buff/cache   available
Mem:          31942       12318        1348        9341       18275        9889
Swap:          7999          83        7916
➜  mall git:(master) ✗ free -h
              total        used        free      shared  buff/cache   available
Mem:            31G         12G        1.1G        9.1G         17G        9.6G
Swap:          7.8G         83M        7.7G
➜  mall git:(master) ✗ 

```

#### du 

> Summarize disk usage of the set of FILEs, recursively for directories.

```shell
#   -h, --human-readable  print sizes in human readable format (e.g., 1K 234M 2G)
#   -d, --max-depth=N     print the total for a directory (or file, with --all) only if it is N or fewer levels below the command line argument;  --max-depth=0 is the same as --summarize
[root@localhost .ssh]# du -hd 0 ~/
29M     /root/
[root@localhost .ssh]# du -hd 1 ~/
0       /root/.cache
23M     /root/.npm
6.0M    /root/.oh-my-zsh
4.0K    /root/.ssh
56K     /root/.config
29M     /root/

# du 命令中 -x 跳过其他文件系统  --max-depth参数设置为1,可以统计出根目录下第一级目录中所有文件大小
# sort命令中 -k 指明具体按照哪一列进行排序 -n 参数表示只对数值进行排序 -r 参数表示反向排序
# 指定第一列并按照数据大小做反序排序
du -x --max-depth=1/|sort -k1 -nr
```

#### df 

> Show information about the file system on which each FILE resides,
> or all file systems by default.

```shell
# -a	显示所有文件系统信息，包括系统特有的 /proc、/sysfs 等文件系统；
# -m	以 MB 为单位显示容量；
# -k	以 KB 为单位显示容量，默认以 KB 为单位；
# -h	使用人们习惯的 KB、MB 或 GB 等单位自行显示容量；
# -T	显示该分区的文件系统名称；
# -i	不用硬盘容量显示，而是以含有 inode 的数量来显示。

[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /
/dev/hdc3             4.8G  139M  4.4G   4% /home
/dev/hdc1              99M   11M   83M  12% /boot
tmpfs                 363M     0  363M   0% /dev/shm

[root@localhost ~]# df -h /etc
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /

# 注意，使用 -a 选项，会将很多特殊的文件系统显示出来，这些文件系统包含的大多是系统数据，存在于内存中，不会占用硬盘空间，因此你会看到，它们所占据的硬盘总容量为 0。
[root@localhost ~]# df -aT
Filesystem    Type 1K-blocks    Used Available Use% Mounted on
/dev/hdc2     ext3   9920624 3823112   5585444  41% /
proc          proc         0       0         0   -  /proc
sysfs        sysfs         0       0         0   -  /sys
devpts      devpts         0       0         0   -  /dev/pts
/dev/hdc3     ext3   4956316  141376   4559108   4% /home
/dev/hdc1     ext3    101086   11126     84741  12% /boot
tmpfs        tmpfs    371332       0    371332   0% /dev/shm
none   binfmt_misc         0       0         0   -  /proc/sys/fs/binfmt_misc
sunrpc  rpc_pipefs         0       0         0   -  /var/lib/nfs/rpc_pipefs

```

#### JVM CPU高负载的排查办法

今天线上一个java进程cpu负载100%。按以下步骤查出原因。

   1. 执行`top -c`命令，找到cpu最高的进程的id

   2. 执行`top -H -p pid`，这个命令就能显示刚刚找到的进程的所有线程的资源消耗情况。找到CPU负载高的线程pid 8627, 把这个数字转换成16进制，21B3（10进制转16进制，用linux命令: printf %x 8627）。

   3. 执行`jstack -l pid`，拿到进程的线程dump文件。这个命令会打出这个进程的所有线程的运行堆栈。

   4. 用记事本打开这个文件，搜索“21B3”，就是搜一下16进制显示的线程id。搜到后，下面的堆栈就是这个线程打出来的。排查问题从这里深入。



---



## 服务器之间传输文件

使用 scp 这个远程传输命令，只要是 Linux 系统，登录 ssh 客户端（比如 putty）即可使用。

### 1、获取远程服务器上的文件

```sh
# scp -P <port> <username>@<ip>:</sourcePath> </targetPath>
scp -P 2223 root@10.23.185.16:/root/test.tar.gz /home/test.tar.gz
```

命令中的大写P 为端口参数，2223 表示ssh的端口，如果是 22 的话，可以不需要该参数，如果是其他端口，必须填写。
root@10.23.185.16 表示使用root用户登录远程服务器10.23.185.16
:/root/test.tar.gz 表示远程服务器上的文件及路径
最后面的/home/test.tar.gz 表示保存在本地上的路径和文件名。
执行命令后，正常的话会有一个提问，输入 yes 回车，然后需要输入远程服务器的 root 密码，回车即可。

### 2、获取远程服务器上的目录

```sh
scp -P 2223 -r root@10.23.185.16:/root/dirname/ /home/dirname/
```

注意：如果是目录，需要添加一个 -r 参数

### 3、将本地文件上传到服务器上

```sh
scp -P 2223 /home/test.tar.gz root@10.23.185.16:/root/test.tar.gz
```

### 4、将本地目录上传到服务器上

```sh
scp -P 2223 -r /home/dirname/ root@10.23.185.16:/root/dirname/
```

---



## 文件解压

### 一、ZIP 工具的使用

```bash
# 安装支持ZIP的工具
yum install -y unzip zip

# 解压zip文件
unzip 文件名.zip

# 压缩一个zip文件
zip 文件名.zip  # 文件夹名称或文件名称 

# 使用 zip 命令压缩目录，需要使用“-r”选项，例如：
#建立测试目录
[root@localhost ~]# mkdir dir1
#压缩目录
[root@localhost ~]# zip -r dir1.zip dir1
adding: dir1/(stored 0%)
#压缩文件生成
[root@localhost ~]# ls -dl dir1.zip
-rw-r--r-- 1 root root 160 6月 1716:22 dir1.zip

```

网络上下载到linux源码包主要是tar.gz和tar.bz2压缩格式的，有一部分是zip

### 五、解压tar.gz

`tar -zxvf xx.tar.gz`

### 六、解压tar.bz2

`tar -jxvf xx.tar.bz2`

### 七、7-zip 工具的使用

```bash
# 安装
yum install -y p7zip

# 7z a <压缩后文件名> <被压缩文件名>    被压缩文件参数放最后
# 这条命令是将所有.jpg的文件压缩成一个7z包
7za a yasuobao.7z *.jpg 

# 将yajiu.7z中的所有文件解压出来，x是解压到压缩包命名的目录下
7za x yasuobao.7z
```

> 使用7zip的命令是7za。
> 安装完成后的使用方法：
> 7za {a|d|l|e|u|x} 压缩包文件名 {文件列表或目录，可选}
>
> a  向压缩包里添加文件或创建压缩包，如向001.7z添加001.jpg，执行：7za a 001.7z 001.jpg；将001目录打包执行：7za a 001.7z 001；
> d  从压缩里删除文件，如将001.7z里的001.jpg删除，执行：7za d 001.7z 001.jpg
> l  列出压缩包里的文件，如列出001.7z里的文件，执行：7za l 001.7z
> e  解压到当前目录，目录结构会被破坏，如001.rar内有如下目录及文件123/456/789.html，
> 执行：7za e 001.rar，目录123和456及文件789.html都会存放在当前目录下。
> x  以完整路径解压。
>
> zip文件解压中文文件乱码问题，由于zip文件中没有声明其编码，所以在Linux上使用unzip解压以默认编码解压，中文文件名会出现乱码。
>
> 
>
> 7za x phpMyAdmin-3.3.8.1-all-languages.7z -r -o./
>
> 参数含义：
>
> x 代表解压缩文件，并且是按原始目录树解压（还有个参数 e 也是解压缩文件，但其会将所有文件都解压到根下，而不是自己原有的文件夹下）
>
> phpMyAdmin-3.3.8.1-all-languages.7z 是压缩文件，这里我用phpadmin做测试。这里默认使用当前目录下的phpMyAdmin-3.3.8.1-all-languages.7z
>
> -r 表示递归解压缩所有的子文件夹
>
> -o 是指定解压到的目录，**-o后是没有空格的，直接接目录。这一点需要注意。**

---



## nginx 常用操作

```sh
# 首先切换到nginx安装根目录 , 再进行下面的操作
cd {NGINX_HOME} 
# 启动 nginx
./sbin/nginx
# 停止 Nginx
./sbin/nginx -s stop
./sbin/nginx -s quit
# 修改nginx配置
vi conf/nginx.conf 
# 检查nginx配置修改有无语法错误
./sbin/nginx -t 
# 配置重载 , 无需重启 
./sbin/nginx -s reload # (无响应状态提示)
service nginx reload   # (推荐 , 有响应状态提示)
```

[test](../Nginx/Nginx 常用命令教程.md)

---

## 查看服务状态

```shell
➜  桌面 systemctl status redis                
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since 一 2020-06-15 08:32:34 CST; 1 months 7 days ago
 Main PID: 2567 (redis-server)
    Tasks: 3
   Memory: 960.0K
   CGroup: /system.slice/redis.service
           └─2567 /usr/bin/redis-server *:6379

6月 15 08:32:34 w26-260284 systemd[1]: Starting Redis persistent key-value ....
6月 15 08:32:34 w26-260284 systemd[1]: Started Redis persistent key-value d....
Hint: Some lines were ellipsized, use -l to show in full.

# /etc/systemd/system/redis.service.d 为启动脚本 , 里面有redis服务启动配置 , 可按需修改
# 如下可见配置文件在 /etc/redis.conf 

➜  redis cat /usr/lib/systemd/system/redis.service                           
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target

```
> 杀掉守护进程使用`ps -aux|grep mysql`命令找出`pid` 再`kill -9 ${pid}`
---



## k8s 操作

#### 重启应用

找到项目`yml`文件目录(如`~/deploy/yml`)使用如下命令

```sh
# 删除正在运行的容器
kubectl delete -f sale-screen-svc.yaml
# 先启用最新的容器 , 启动成功后再替换原有容器
kubectl apply -f sale-screen-svc.yaml
```

