---
title: 安装Zencart Edify定制版
date: 2016-10-19 11:21:38
tags:
---

## 预备

平台Windows 7，所需软件如下：

* VirtualBox (VirtualBox-5.1.8-111374-Win.exe)
* CentOS 7 (CentOS-7-x86_64-Minimal.iso)
* Xftp
* Xshell (可选)
* Zencart Edify定制版安装包

### 在Vbox上安装CentOS 7 

具体安装步骤略过，主要注意点是网卡的配置。
一般默认网卡1的默认连接方式为**网络地址转换(NAT)**,不要管它；在网卡2选项卡页面钩选**启用网络连接(E)**，并在**连接方式(A)**下拉列表里面选择**仅主机(Host-Only)网络**。启动安装...

> 添加Host-Only网卡的目的是与主机(这里是Windows 7)通信，因为NAT网卡（网卡1）与主机处于不同网段，不能Ping通。想了解VBox不同网卡的作用，自行Google。

### 给CentOS 7 设置静态IP

进入网卡配置文件目录：

```bash
cd /etc/sysconfig/network-scripts/
```

编辑网卡2的配置文件(CentOS下网卡2默认配置文件为ifcfg-enp0s8)：

> 本文的命令都是在root用户下执行的，如果非root用户，请自行在命令前追加` sudo `进行提权。

```bash
vi ifcfg-enp0s8
```

填入以下内容：

```
	TYPE="Ethernet"
	BOOTPROTO="static" #这里修改为static
	IPADDR=192.168.56.99 #这里是要设置的IP，注意，Host-Only默认static IP区段为192.168.56.1 - 192.168.56.99，其他值可能出错，或者去VBox全局设置确认一下IP区间
	NETMASK=255.255.255.0
	NETWORK=192.168.56.255
	NM_CONTROLLED="no" #这里设置是否受NetManager控制
	IPV4_FAILURE_FATAL="no"
	NAME="enp0s8"
	UUID="e56f85b7-6253-4101-9bc9-c4e93ec10d0e" #这里填写生成的uuid
	DEVICE="enp0s8" #这里是要配置的网卡
	ONBOOT="yes" #是否开机启动
```

> 可以使用命令 ` uuidgen ifcfg-enp0s8 ` 生成UUID。

重启网络服务：

```
systemctl restart network.service
```

如果没报错误，可以查看一下本机IP：

```
ip addr
```

会显示类似如下信息：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:87:e5:3c brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 58173sec preferred_lft 58173sec
    inet6 fe80::a00:27ff:fe87:e53c/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:d0:ab:60 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.99/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed0:ab60/64 scope link 
       valid_lft forever preferred_lft forever
```

可以看到enp0s8（即我们的网卡2）的IP已经是上面设定的IP了，静态IP设定完毕！

### 自定义域名

为了方便开发，可以自定义一个域名指向本机IP（这里是Host-Only网卡的IP，这就是为什么非要把Host-Only网卡设为静态IP）。
编辑hosts文件：

```
vi /etc/hosts
```

添加一行：

```
192.168.56.99 www.devel.orz devel.orz
```

然后重启网络服务，就可以使用自定义域名www.devel.orz访问虚拟机了。

> 这个域名是随意定的，我开发常用这个，你可以自己选一个www.abc.orn之类的。

### 安装Apache

安装前先更新当前系统：

```bash
yum update -y
```

安装Apache：

```bash
yum install httpd -y
```

这样就安装完成了。
Apache是以httpd服务的形式存在的，要使服务开机或重启的时候自动运行，必须启用这个服务，使用命令：

```bash
systemctl enable httpd
```

这样就启用服务了。由于服务刚安装，系统也没有重启，服务可能没启动，我们手动启动服务：

```bash
systemctl start httpd
```

然后，检查httpd服务状态是否启用和启动了：

```bash
systemctl status httpd.service
```

显示类似如下信息：

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2016-10-20 10:08:21 CST; 1min 13s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1286 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1286 /usr/sbin/httpd -DFOREGROUND
           ├─1632 /usr/sbin/httpd -DFOREGROUND
           ├─1633 /usr/sbin/httpd -DFOREGROUND
           ├─1634 /usr/sbin/httpd -DFOREGROUND
           ├─1635 /usr/sbin/httpd -DFOREGROUND
           └─1636 /usr/sbin/httpd -DFOREGROUND
```

看到enabled和active就表明已经启用服务并正在运行了。
如果看到*AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using orz.localdomain.* 这样的信息，编辑*httpd.conf*：

```bash
vi /etc/httpd/conf/httpd.conf
```

定位到以 *#ServerName* 开头的行，替换为下面内容：

```
ServerName www.devel.orz:80 
```

域名与上面添加到hosts里面的相同。
再次查看httpd状态，提示AH00558不再出现。
若要停止或重启httpd服务，可分别使用如下指令：

```bash
systemctl stop httpd.service
systemctl restart httpd.service
```

这时，我们已经安装并启用了Apache，可以在主机（这里指Windows 7）浏览器打开链接 <http://192.168.56.99> 测试Apache是否可用。这时，浏览器并不能显示一个有效的页面。因为，我们的请求被防火墙拦截了。
下面，我们配置一下防火墙，使Web服务能够正常访问：

```
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

这时，再次访问 <http://192.168.56.99> ，应该显示Apache的默认页面。
由于后面的安装涉及文件操作，CentOS的SELinux服务会阻止我们对文件进行操作。由于是开发环境，我们就简单将其关闭。编辑SELinux配置文件：

```bash
vi /etc/sysconfig/selinux
```

找到下面的内容：

```
#SELINUX=enforcing
```

改为：

```
SELINUX=disabled
```

设置好了并不能即时生效，需要重启系统。要想即时生效，可执行命令：

```bash
setenforce 0
```

查看SELinux状态，可用：

```bash
getenforce
```

### 修改主机hosts

这一步骤是可选的，只是为了开发时显得更正规。Windows 7的hosts文件放在*C:\Windows\System32\drivers\etc\hosts*，以管理员权限打开，可用NotePad++。追加如下内容：

```
192.168.56.99 www.devel.orz devel.orz
```

这样在浏览器里直接访问<http://www.devel.orz>就可以访问刚配置的服务器了。

### 安装MySQL

由于MySQL默认不在Yum的repo源里，要使用yum安装必须先添加MySQL的源。

```bash
rpm -Uvh http://repo.mysql.com//mysql57-community-release-el7-9.noarch.rpm
```

执行以上命令，MySQL就添加到了Yum的源。添加的源里包含了所有当前可用的MySQL版本，默认MySQL5.7版。当前Zencart不能使用这个版本，需要更改默认版本为5.6版，编辑源：

```bash
vi /etc/yum.repos.d/mysql-community.repo
```

把*[mysql56-community]*标签下的enable值设为1，如下：

```
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

其他标签下的enable值设为0，这里只要修改*[mysql57-community]*标签的enable值就可以了。
开始安装：

```
yum install mysql-server -y
```

安装完后，进行一些中文用户必要的设置，修改my.cnf以支持UTF-8：

```
vi /etc/my.cnf
```

在 *[mysqld]* 标签下追加：

```
collation-server=utf8_general_ci
character-set-server=utf8
```
最好同时在 *[mysqld]* 标签下追加：

```
sql_mode=IGNORE_SPACE,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

不然可能网站后台不能登陆。重启mysql服务：

```
systemctl restart mysqld
```

这样以后创建的数据库，就默认支持UTF-8了。
下面进行一些安装后的初始化，运行指令：

```
mysql_secure_installation
```

根据提示，先设置root密码，后面的全选yes即可。

以root用户登陆mysql：

```
mysql -u root -p
```

输入刚才设定的密码，就进入了SQL命令行交互模式，依次输入以下SQL语句，创建后面安装需要的数据库：

```sql
create database zencartdb;
create user 'zencart'@'localhost' identified by 'zencart';
grant all privileges on zencartdb.* to 'zencart'@'localhost';
flush privileges;
exit;
```

这样就创建了数据库zencartdb及普通用户zencart。

### 安装PHP

安装PHP及ZenCart需要的组件：

```
sudo yum -y install php php-mysql php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-mcrypt curl zlib
```

安装 phpMyAdmin（可选）

```
yum -y install epel-release
yum -y install phpmyadmin
```

配置 *phpMyAdmin.conf* ：

```bash
vi /etc/httpd/conf.d/phpMyAdmin.conf
```

修改后的部分为：

```
<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     # Apache 2.4
     <RequireAny>
       Require all granted #原来两行删除，修改为这样
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>
```

重启Apache服务:

```bash
systemctl restart httpd.service
```

在浏览器打开 <http://www.devel.orz/phpmyadmin> 就可以看到phpMyAdmin的登陆页面了。

## Zencart安装

### 上传

使用工具将 *\Quick Start Demo Package\Zencart* 目录下的所有内容复制到CentOS的 */var/www/html/* 目录下。

### 设置配置文件及相关权限

依次执行如下指令：

```bash
cd /var/www/html/
mv admin/includes/dist-configure.php admin/includes/configure.php
mv includes/dist-configure.php includes/configure.php
chmod 777 admin/includes/configure.php
chmod 777 includes/configure.php
chmod 777 cache
chmod 777 includes/languages/english/html_includes
chmod 777 logs
chmod 777 media
chmod 777 pub
chmod 777 images/
chmod 777 admin/backups
chmod 777 admin/images/graphs
```

### 执行安装

打开链接 <http://www.devel.orz/zc_install> ，安装提示进行安装。

> 如果已经设定了相关的权限，还是碰到类似777文件夹无权限的问题，可能忘了关闭SELinux。运行 **setenforce 0**，继续安装。

### 安装后

由于安装是Edify定制版，安装后需要执行文件 *sql-patch.txt* 内的SQL语句。
现在网站应该能够运行，但是注册用户的时候要用到Google验证码，根据提示申请一下，域名填写 *www.devel.orz* ，把申请到的google_site_key及google_secret_key更新到数据库表 *edify* ，就可以正常注册了。