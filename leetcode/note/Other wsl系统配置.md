---
title: wsl系统配置
date: 2023-11-21 15:51:32
tags: [日常笔记]
---

# 将 wsl 配置为局域网服务器

## 校园网内的另一台设备访问 wsl

wsl 和 win 同为运行在 hyper-v 上的虚拟机，win 直接连接实际网卡，连接学校校园网。

wsl 默认采用 NAT 模式，这意味着 wsl 和 win 上都有一块虚拟网卡，配合虚拟的路由器和交换机，再次组成一个内网。由于 win 和 wsl 都有着虚拟网卡，因此他们都有着相应的内网 ip，同时虚拟的路由器（网关）也有着一个网关 ip（路由器内网端口的内网 ip），虚拟路由器的外网端口连接着实际网卡，其 ip 等于 win 的校园网 ip。

wsl 和 win 是可以相互 ping 通的（因为在同一个局域网内），但是校园网内的另一台设备（比如 mac），由于 NAT 的限制是无法主动访问 wsl 的，但是 wsl 可以主动访问 mac。

有两种方法可以解决这个问题，第一种方法是更改 wsl 为桥接模式，桥接模式可以使得 wsl 直接连接物理网卡并拥有校园网内网 ip，另一种方法是设置 win 的端口转发，将 wsl 的一些需要用到的端口暴露出来。

什么是端口转发呢？我们知道在 NAT 中，内网某台机器的 TCP 报文段到达 NAT 网关（路由器）之后，其目标 IP 和目标端口不变，源 IP 换成路由器外网端口的 IP，源端口换成 NAT 分配的一个随机端口，同时 NAT 会记录这个更换的映射。在 NAT 网关收到返回的 TCP 报文段时，会根据之前的映射关系，将目标 IP 换成内网相应机器的内网 IP，端口换成记录的内网机器相应端口，然后通过局域网转发给内网机器。这也就是为什么内网机器可以主动访问外网机器，而外网机器不能主动访问内网机器（因为到达 NAT 后没有相应的 NAT 映射记录）。而端口转发就是自己创建一条 NAT 映射记录，核心在于 NAT 网关某个端口映射内网的某个 IP 和端口。

win 中 NAT 映射（端口转发）可以通过以下命令完成：

- 增加转发规则，以 80 端口为例

  ```shell
  netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=wsl虚拟网卡ip
  ```

- 查看所有已转发的端口
  ```shell
  netsh interface portproxy show all
  ```
- 重置转发规则（删除所有规则）
  ```shell
  netsh interface portproxy reset
  ```

我们可以通过上述命令开放一些端口（比如 ssh 的 22，mysql 的 3306，redis 的 6379）

## 配置 ssh

查看服务器是否安装了 ssh-server

```shell
ps -ef|grep sshd
#root      2859     1  020:29 ?        00:00:00 /usr/sbin/sshd -D
```

如果没有，安装

```shell
apt install openssh-server
```

允许密码登录，允许 root 登录，重启 ssh

```shell
# vim /etc/ssh/sshd_config

PasswordAuthentication yes
PermitRootLogin yes
service sshd restart
```

为服务器 root 用户设置密码

```shell
sudo passwd root
```

win 配置端口转发

```shell
netsh interface portproxy add v4tov4 listenport=22 listenaddress=0.0.0.0 connectport=22 connectaddress=wsl虚拟网卡ip
```

接下来就可以直接用密码登录服务器 root 用户了

```shell
# win
ssh root@wsl内网ip
# mac
ssh root@win校园网ip
```

## 安装 mysql

安装 mysql 有两种方式，一种是通过 apt 软件管理器，另一种是手动，我们通过 apt 安装 mysql 8.0

首先需要更换 apt 镜像源，默认镜像源是国外的很慢，当然也可以设置 apt 代理解决。

使用阿里云镜像源：https://developer.aliyun.com/mirror/ubuntu

```shell
vim /etc/apt/sources.list # 删掉所有内容，然后粘贴镜像源
apt update # 更新一下
```

安装 mysql

```shell
apt install mysql-server
```

查看 mysql 状态

```shell
service mysql status
```

启动 mysql

```shell
service mysql start
```

此时 wsl 已经可以连接 mysql 了，但是 win 还无法连接

修改账号权限为%

```shell
use mysql;
select host, user, authentication_string, plugin from user;
update user set host = '%' where user = 'root';
GRANT ALL ON *.* TO 'root'@'%';
flush privileges;
```

修改 mysql 配置文件 my.cnf 允许外部 ip 访问

```shell
vim /etc/mysql/my.cnf
# 添加如下
[mysqld]
bind-address = 服务器本地ip
# 重启mysql服务
service mysql restart
```

为 root 用户设置密码

SHA2 认证改为密码认证（mysql 8.0 默认 SHA2 认证 5.7 默认密码认证）

```shell
update user set plugin='mysql_native_password' where user='root';
```

修改操作系统防火墙和云服务器安全组，放行 3306 端口

现在 win 可以连接 mysql 了，但是校园网内其他机器还不可以

win 配置端口转发

```shell
netsh interface portproxy add v4tov4 listenport=3306 listenaddress=0.0.0.0 connectport=3306 connectaddress=wsl虚拟网卡ip
```

现在校园网内其他机器也可以连接了

## 安装 redis

apt 安装 redis

```shell
apt install redis-server
```

修改 redis 配置

```shell
vim /etc/redis/redis.conf
# 注释掉下面这一行，开放远程访问
bind 127.0.0.1
# 解开下面一行注释，设置redis密码
requirepass yourpassword
```

重启 redis

```shell
/etc/init.d/redis-server restart
```

win 配置端口转发

```shell
netsh interface portproxy add v4tov4 listenport=6379 listenaddress=0.0.0.0 connectport=6379 connectaddress=wsl虚拟网卡ip
```
