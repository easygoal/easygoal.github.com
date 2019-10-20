---
layout: post
category: guideline
tagline: ""
keywords: server
tags: ["guideline","wow","魔兽世界"]
header:
lang: zh_CN 
date: 2019-10-06
title: Debian 9 下安装基于TrinityCore 的魔兽世界服务器
---

# 测试配置要求：
- CPU：INTEL，AMD系列至少四核或以上
- 内存： 最少4G起
- 硬盘： 至少60G剩余空间，推荐SSD硬盘

# 安装编译环境
```
$ sudo apt-get install build-essential git clang cmake make gcc g++ libmariadbclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev libboost-all-dev mariadb-server p7zip
$ sudo update-alternatives --install /usr/bin/cc cc /usr/bin/clang 100
$ sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang 100
```

# 安装测试版本3.3.5a（12340）
```
$ cd ~
$ git clone -b 3.3.5 git://github.com/TrinityCore/TrinityCore.git
$ cd TrinityCore
$ git pull origin 3.3.5
```
创建服务器路径，$USER是你当前用户名
```
$ mkdir -p /home/$USER/server
```

创建编译目录
```
$ cd ~/TrinityCore
$ mkdir build
$ cd build
```
开始编译，根据电脑配置，决定完成时间
```
$ cmake ../ -DCMAKE_INSTALL_PREFIX=/home/$USER/server
$ make -j4  
$ make install
```
下载Wrath of Litch King 3.3.5a (12340)客户端
先保证硬盘剩余空间超过40G以上，然后访问 [https://www.xspio.com/](https://www.xspio.com/) 去下载客户端，或者从 [http://wlkwow.com/viewtopic.php?f=2&t=9]() 获取。
电信宽带线路下载：
``wget http://222.186.15.202:12345/wowclient/wlkwowc.rar``
这将是一个漫长的过程，除非你家宽带够快。
下载完后解压到当前目录。

提取客户端数据
```
$ cd "wow客户端目录"   ##wow客户端目录就是你能看到wow.exe运行程序的目录
$ /home/$USER/server/bin/mapextractor
$ /home/$USER/server/bin/vmap4extractor
$ mkdir vmaps
$ /home/$USER/server/bin/vmap4assembler Buildings vmaps
$ /home/$USER/server/bin/mmaps_generator
$ mkdir -p /home/$USER/server/data
$ cp -r dbc maps vmaps mmaps /home/$USER/server/data
```
修改配置文件
```
$ cd /home/$USER/server/etc
$ cp authserver.conf.dist authserver.conf
$ cp worldserver.conf.dist worldserver.conf
```
worldserver.conf为游戏配置，authserver.conf为用户权限配置，worldserver.conf可以控制角色初始等级，金币，跨阵营组队等N多功能。

```
LoginDatabaseInfo = "127.0.0.1;3306;trinity;trinity;auth"  ## worldserver.conf / authserver.conf    
WorldDatabaseInfo = "127.0.0.1;3306;trinity;trinity;world" ## worldserver.conf    
CharacterDatabaseInfo = "127.0.0.1;3306;trinity;trinity;characters" ## worldserver.conf
DataDir = "/home/$USER/server/data" ## worldserver.conf
```

# 安装数据库
1. 数据库初始化
``$ sudo mysql_secure_installation ## 设置一下root密码，然后一路按 y 回车``
2. 安装wow数据库，查看[https://github.com/TrinityCore/TrinityCore/blob/3.3.5/sql/create/create_mysql.sql](https://github.com/TrinityCore/TrinityCore/blob/3.3.5/sql/create/create_mysql.sql)

```
GRANT USAGE ON * . * TO 'trinity'@'localhost' IDENTIFIED BY 'trinity' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 ;

CREATE DATABASE `world` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

CREATE DATABASE `characters` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

CREATE DATABASE `auth` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

GRANT ALL PRIVILEGES ON `world` . * TO 'trinity'@'localhost' WITH GRANT OPTION;

GRANT ALL PRIVILEGES ON `characters` . * TO 'trinity'@'localhost' WITH GRANT OPTION;

GRANT ALL PRIVILEGES ON `auth` . * TO 'trinity'@'localhost' WITH GRANT OPTION;
```
把里面的内容复制下来
```
$ nano wowmysql.sql   ##右键粘贴上面复制的内容
$ sudo mysql -uroot -p
mysql> sources /home/$USER/wowmysql.sql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root的密码' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
mysql> \q
```
3. 修改数据库配置，供局域网或者外网连接
打开文件``/etc/mysql/mariadb.conf.d/50-server.cnf``，注释掉bind-address项。
重启mariadb，``sudo systemctl restart mariadb``

## 下载数据文件
访问[https://github.com/TrinityCore/TrinityCore/releases]()，下载TDB 335.64下的文件，
```
$ cd ~
$ wget -c -t5 https://github.com/TrinityCore/TrinityCore/releases/download/TDB335.64/TDB_full_world_335.64_2018_02_19.7z
$ 7zr x TDB_full_world_335.64_2018_02_19.7z
$ mv TDB_full_world_335.64_2018_02_19.sql /home/$USER/server/
```
运行服务器
```
$ /home/$USER/server/bin/authserver &
$ /home/$USER/server/bin/worldserver
TC> account create 游戏账号 密码
TC> account set gmlevel 游戏账号 3 1  ## 升级为GM 3级
```
怎么退出我也不知道，我是ctrl + c，然后回车，等一下。

## 修改数据库IP，新服名字，版本号。
去网上寻找mysql客户端工具，如Navicat for MySQL。
```
$ sudo mysql -uroot -p
mysql> show databases;
mysql> use auth;
mysql> select * from realmlist;

+----+---------+---------------+--------------+-----------------+------+------+------+----------+----------------------+------------+-----------+
| id | name    | address       | localAddress | localSubnetMask | port | icon | flag | timezone | allowedSecurityLevel | population | gamebuild |
+----+---------+---------------+--------------+-----------------+------+------+------+----------+----------------------+------------+-----------+
|  1 | testwow | 192.168.1.200 | 127.0.0.1    | 255.255.255.0   | 8085 |    0 |    0 |        1 |                    0 |          0 |     12340 |
+----+---------+---------------+--------------+-----------------+------+------+------+----------+----------------------+------------+-----------+
1 row in set (0.00 sec)
```
其中，name是服务器名称，开新服就可以在表2添加 ，address就是你连接网络的IP，gamebuild就是游戏版本号，请与客户端版本号相同，不然你的服务器会显示离线。

# 启动服务器
```
$ /home/$USER/server/bin/authserver &
$ /home/$USER/server/bin/worldserver
```
上面这样启动有个弊端，会一直卡在TC状态，你可以安装pm2做进程守护，保护长久运行。

# 客户端登陆
```
echo y | rd /s "Cache"
echo SET realmlist "192.168.1.200" > Data\zhTW\realmlist.wtf
echo SET realmlist "192.168.1.200" > Data\enTW\realmlist.wtf
echo SET realmlist "192.168.1.200" > Data\zhCN\realmlist.wtf
echo SET realmlist "192.168.1.200" > Data\enCN\realmlist.wtf
echo SET realmlist "192.168.1.200" > Data\enUS\realmlist.wtf
echo SET realmlist "192.168.1.200" > realmlist.wtf
start Wow.exe
goto end
```
把上面的内容生成一个bat批处理文件，IP地址根据你服务器的改，放到你的游戏客户端目录，点击就可以了。
关于GM命令，网上大把，自助注册等功能，不再研究，本来就是自己测试着玩，过过80级时拿极品装备，极品坐骑的瘾。

