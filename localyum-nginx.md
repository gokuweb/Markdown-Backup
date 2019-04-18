---
title: CentOS 制作本地 Yum 库并安装 Nginx
categories:
  - 运维
tags:
  - yum
  - nginx
  - keepalived
abbrlink: 76e13aa5
date: 2019-04-09 06:29:22
---

&#8195;&#8195;内网环境不允许联网的时候，安装各种软件和库是一件非常麻烦的事，安装 A 提示依赖 B ，安装 B 提示依赖 C，甚至 `./configure` 、`make` 都无法正常执行，此时有一个本地 yum 源就非常方便。
<!-- more -->

## CentOS 镜像版本
先说说 CentOS 的几个镜像版本：
> 1. LiveDVD，一个光盘 CentOS 系统，该镜像无需安装就可进入 CentOS 系统，进入后也可以选择安装到虚拟机，或者直接退出。
> 2. bin-DVD1.iso，普通安装版，集成了很多常用软件，因为个人电脑刻录光盘的文件不能超过 4.7G，所以还有一个 DVD2 的镜像做扩充，不过不知道为什么 7.0 版本直接出了超过 7G 的 Everything。
> 3. bin-DVD2.iso，可以理解为 DVD1 的软件扩充包，基本都是 rpm 软件包。
> 4. Minimal版，精简版本，包含核心组件。
> 5. Everything ，最新的 7.0 多了这么个版本，顾名思义就是完整版，集成了很多软件，感觉是 DVD1 和 DVD2 的合体。如果不能联网的话推荐下载完整版，否则安装各种依赖非常麻烦。

## rpm 和 dpkg
### rpm 和 dpkg 是什么
Linux 系统基本上分两大类：
1.RedHat 系列：Redhat、Centos、Fedora等。
2.Debian 系列：Debian、Ubuntu等。

RedHat 系列：
> 1. 常见的安装包格式 rpm 包，安装 rpm 包的命令是 **“rpm -参数”**。rpm 相当于 windows 中 的安装文件，它会自动处理软件包之间的依赖关系；
> 2. 包管理工具 **yum**  ；
> 3. 支持 tar 包，只是一个压缩包。

Debian 系列 
> 1. 常见的安装包格式 deb 包,安装deb包的命令是 **“dpkg -参数”** ；
> 2. 包管理工具 **apt-get** ；
> 3. 支持 tar 包。

### rpm 和 dpkg 命令
rpm 与 dpkg 常用命令对比：

|       rpm        |      dpkg        |          说明          |
| ---------------- | ---------------- | ---------------------- |
| rpm -qa          | dpkg -l          | 列出所有已安装的软件包 |
| rpm -qf 文件名   | dpkg -S 文件名   | 查询一个文件属于哪个软件包 |
| rpm -ql 软件包名 | dpkg -L 软件包名 | 查询已安装软件包的路径和配置文件 |
| rpm -qpl xx.rpm  | dpkg -c xx.deb   | 查询软件包中包含的文件 |
| rpm -ivh xx.rpm  | dpkg -i xx.deb   | 安装，并显示详细安装信息和进度。<br>--node 忽略依赖包（双杠）。<br>--force 强制安装 |
| rpm -Uvh xx.rpm  | dpkg -i xx.deb   | 更新，dpkg 更新和安装一样 |
| rpm -e xx.rpm    | dpkg -r xx.deb   | 卸载 |

## apt 和 yum 
### apt 和 yum 是什么
&#8195;&#8195;rpm 与 dpkg 在安装或移除时只能提示某些依赖不满足，但无法自动安装对应的依赖软件，所以就有了前端软件包管理器 -- YUM（Yellow dog Updater, Modified）及 APT（Advanced Packaging Tool），包管理器最大的优点是可以 **自动处理各种软件包可能的依赖关系**，自动查找安装相关的依赖包。

### apt-get 和 yum 命令
**ap-get 与 yum 命令对比：**

|       apt-get        |    yum       |          说明          |
| ---------------- | ---------------- | ---------------------- |
| apt-get update  |  yum update  | apt-get update 只更新源<br>yum update 同时升级软件包和系统版本|
| apt-get upgrade     |  yum upgrade  | apt-get upgrade 只更新已安装的包<br>yum update 同时升级软件包和系统版本，yum 的 update 和 upgrade 一样 |
| apt-get install | yum install |  安装包<br> apt-get -f install 修复安装 "-f = ——fix-missing"|
| apt-get remove   |  yum remove  | 删除包<br>apt-get remove package -- purge 删除包，包括删除配置文件等|
| apt-cache search    | yum search  | 搜索匹配特定字符的包 |
| apt-cache show    |  yum info  | 获取指定包的相关信息，如说明、大小、版本等 |
| apt-file search    | yum provides  | 检索包内文件 |

### 配置文件
**apt 配置文件**
比如 Ubuntu 系统，联网环境我们装好系统后第一件事就是修改源，首先查看系统版本：
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.4 LTS
Release:	16.04
Codename:	xenial
```
我们需要 `xenial` 版本的源，`apt` 命令的配置文件是 `sources.list`，以中科大源为例：
```
$ vim /etc/apt/sources.list

deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
```

**yum 配置文件**
yum 的配置有两种方式：
> 1. /etc/yum.conf。
> 2. /etc/yum.repos.d/ 目录下的若干 repo 后缀文件，具体配置看下一章节。


## 制作 yum 本地库
### yum 库原理
&#8195;&#8195;在联网状态下在使用 `yum` 或者 `apt-get` 时候都是去指定的地址库下载安装对应的软件，但是内网状态不能联网的情况下，如果想用这些命令就需要使用本地 yum 库了，yum 库主要由三部分组成：
> 1. 软件仓库，就是一个包含各种 rpm 软件包的软件仓库（即 yum 源），这个仓库就是我们软件的来源，也就是镜像中的 `Packages/` 。
> 2. 软件仓库的数据库文件，也就是镜像中的 `repodata/`，这个目录是分析 RPM 软件后所产生的软件属性相依数据放置处，该目录下文件用于收集软件仓库中所有 rpm 包的头部信息（每个 rpm 包的包头信息包含了该包的描述、功能、提供的文件、依赖关系等信息)，其中文件 `repomd.xml` 是数据库的索引文件， md（metadata）表示元数据。
> 3. 配置文件，也就是本地 `/etc/yum.repos.d/` 下的若干 `*.repo` 文件。

&#8195;&#8195;配置文件中 `baseurl` 指向的是 `repodata` 父目录的路径，用 yum 进行更新或安装软件时，第一步是对 `repodata` 目录中的文件进行分析，首先要找的是 `repomd.xml` ，然后经过一系列的分析便可知道软件包的详细信息和依赖关系，最后从 `Packages/` 找到对应的软件安装和更新。

### 挂载系统镜像
&#8195;&#8195;挂载系统镜像主要是为了 **获取光盘中的软件仓库 `Packages/` 和软件仓库的数据库文件 `repodata/`** 。
1. 开启共享文件夹
&#8195;&#8195;需要先安装 VMware Tools，开启共享文件夹后把镜像文件拷贝进去，这样就能在虚拟机中直接访问，默认路径是 `/mnt/hgfs/` 。

2. 把镜像文件挂载到 `/media/CentOS` 目录：
```
$ sudo mkdir -p /media/CentOS
$ sudo mount -o loop /mnt/hgfs/CentOS-6.8-x86_64-bin-DVD1.iso /media/CentOS/
```

### 修改配置文件 
1. 备份配置文件。
&#8195;&#8195;在 `/etc/yum.repos.d/` 下面有很多 `repo` 后缀的文件，我们留一个就行了，先做一下备份：
```
$ cd /etc/yum.repos.d/
$ sudo rename .repo .repo.bak *.repo
$ sudo cp CentOS-Media.repo.bak CentOS-Media.repo
```

2. 修改配置文件
&#8195;&#8195;然后修改 `CentOS-Media.repo` 文件如下：
```
$ sudo vim CentOS-Media.repo

[c6-media] #容器名字
name=CentOS-$releasever - Media
baseurl=file:///media/CentOS  #rpm 软件包的路径
gpgcheck=1                    #是否开启gpg 校验，1开启
enabled=1                     #是否启用该源，0 为禁用
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

**说明**
> 1. [base]：第一行是容器的名字，中刮号一定要存在，名称可以随意取但是不能有两个相同的容器名称， 否则 yum 会不晓得该到哪里去找容器相关软件清单文件。
> 2. name：只是说明一下这个容器的意义而已，重要性不高。
> 3. mirrorlist=：列出这个容器可以使用的映射站台，可选项，我们这里没有使用该配置。
> 4. baseurl=：这个最重要，因为后面接的就是容器的实际地址。mirrorlist 是由 yum 程序自行去捉映射站台， baseurl 则是指定固定的一个容器地址，这个地址是 `Packages/` 和 `repodata/` 的父目录路径。
> 5. enable=1：就是让这个容器被启动。如果不想启动可以使用 enable=0 。
> 6. gpgcheck=1：还记得 RPM 的数码签章吗？这就是指定是否需要查阅 RPM 文件内的数码签章，用以确定 rpm 包来源的有效性和安全性，1 代表启用。
> 7. gpgkey=：就是数码签章的公钥档所在位置，使用默认值即可。

### 验证源
&#8195;&#8195;安装更新软件时候yum 会先下载容器的清单到本机的 `/var/cache/yum/` 下，因为缓存的关系，可能会出现无法安装或升级的问题，我们清理下缓存：
```
$ yum clean all

Loaded plugins: fastestmirror, refresh-packagekit, security
Cleaning repos: c6-media
Cleaning up Everything
Cannot remove rpmdb file /var/lib/yum/rpmdb-indexes/file-requires
Cannot remove rpmdb file /var/lib/yum/rpmdb-indexes/version
Cannot remove rpmdb file /var/lib/yum/rpmdb-indexes/conflicts
Cannot remove rpmdb file /var/lib/yum/rpmdb-indexes/pkgtups-checksums
Cleaning up list of fastest mirrors
```

然后查看软件库：
```
$ yum repolist

Loaded plugins: fastestmirror, refresh-packagekit, security
Determining fastest mirrors
c6-media                                                 | 4.0 kB     00:00 ... 
repo id                          repo name                                status
c6-media                         CentOS-6 - Media                         6,575
repolist: 6,575
```

### 扩展软件库
&#8195;&#8195;合并 `DVD1` 和 `DVD2` 镜像的 `Packages/`，现在我们已经使用 `CentOS-6.8-x86_64-bin-DVD1.iso` 镜像做了本地 yum 库，`DVD2` 镜像没有软件仓库的数据文件 `repodata` ，所以不可以直接用来做 yum 库，需要进行一些配置，流程如下：
**拷贝文件**
```
$ mkdir -p /media/CentOS
$ mount -o loop /mnt/hgfs/CentOS-6.8-x86_64-bin-DVD1.iso /mnt/dvd1/
$ mount -o loop /mnt/hgfs/CentOS-6.8-x86_64-bin-DVD2.iso /mnt/dvd2/
$ cp -av /mnt/dvd1 /mnt/dvd3
$ cp -v /mnt/dvd2/Packages/*.rpm /mnt/dvd3/Packages/
```

**合并 TRANS.TBL **
将 DVD2 中 TRANS.TBL 的信息追加到 DVD1 中 TRANS.TBL 后面, 并排序保存：
```
$ cat /mnt/dvd2/Packages/TRANS.TBL >> /mnt/dvd3/Packages/TRANS.TBL
$ mv /mnt/dvd3/Packages/{TRANS.TBL,TRANS.TBL.BAK}
$ sort /mnt/dvd3/Packages/TRANS.TBL.BAK > /mnt/dvd3/Packages/TRANS.TBL
$ rm -rf /mnt/dvd3/Packages/TRANS.TBL.BAK
```

**备份 YUM 配置文件**
```
$ cd /etc/yum.repos.d
$ rename .repo .repo.bak *.repo
```

**修改 YUM 配置文件**
```
$vim /etc/yum.repos.d/CentOS-Media.repo

[c6-media]
name=CentOS-\$releasever - Media
baseurl=file:///mnt/dvd3
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

**更新源**
```
$ yum clean all
$ yum repolist
```

**将 /mnt/dvd3/ 打包为 ISO**
```
$ mkisofs -l -J -L -r -V "CentOS-6.8-x86_64" -o /mnt/iso/CentOS-6.8-x86_64-DVD.iso /mnt/dvd3
```

## 安装 Nginx
Nginx 依赖 pcre、zlib 和 ssl。

### 源码安装 pcre

```
$ cd pcre-8.43
$ ./configure
$ make
$ make install
```

**问题：**
1. `./configure` 可能遇到的问题是 `C compiler cannot create executables`、` C compiler cc is not found` 等，安装一些必备的包就行了：
```
yum -y install gcc gcc-c++ autoconf automake make
```

2. `make` 可能遇到的问题是 `automake-1.14 is missing`，需要重新配置软件源码包：
```
$ autoreconf -ivf　
```

&#8195;&#8195;autoconf 是一个用于生成可以自动地配置软件源码包，用以适应多种 UNIX 类系统的 shell 脚本工具，其中 autoconf 需要用到 m4 ，便于生成脚本。如果有哪个源文件更新，autoreconf 会检测到并重新调用上面的工具来生成新的 Makefile。
这一步可能会提示 `can't exec libtoolize`，安装 libtool 后继续执行即可：
```
$ yum install -y libtool
$ autoreconf -ivf
$ make
```

### 源码安装 zlib
相关包安装好，依赖没问题的话就很容易了：
```
$ cd zlib-1.2.11
$ ./configure
$ make
$ make install
```

### 源码安装nginx

```
$ cd nginx-1.14.2
$ ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
$  make
$  make install
$ /usr/local/nginx/nginx -t  #检测配置文件
$ /usr/local/nginx/nginx   #启动 nginx
$ /usr/local/nginx/nginx -s  reload  #重新加载 nginx
$ /usr/local/nginx/nginx -s stop  #退出nignx
```

**问题：**
1. 只有本地 `127.0.0.1` 可以访问，其他电脑局域网电脑无法访问，查看一下防火墙的配置：
```
$ vim /etc/sysconfig/iptables

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```
&#8195;&#8195;第 11 和 12 行的意思是，在 INPUT 表和 FORWARD 表中拒绝所有其他不符合上述任何一条规则的数据包，可以理解为除了已经开放的端口，其他端口全部拒绝，在第 10 行后面加入如下规则：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```

重启防火墙后局域网内其他机器可以正常访问了：
```
$ service iptables restart  
```

2. 如果事后需要新增某个模块，比如要开启 HTTPS 需要 SSL 模块 ：
```
$ /usr/local/nginx/nginx -s stop
$ cd nginx-1.14.2
$ ./configure --with-http_ssl_module
$ make  #但是不要执行 make install，因为 make 是用来编译的，而 make install 是安装，不然整个 nginx 会重新覆盖的。
$ cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
$ cp objs/nginx /usr/local/nginx/sbin/nginx
$ /usr/local/nginx/sbin/nginx  #启动
```

### keepalived 
源码安装：
```
$ cd keepalived-2.0.15
$  ./configure --prefix=/usr/local/keepalived
$  make
$ make install
```

**问题：**
1. 提示 `OpenSSL is not properly installed on your system` ：
```
$ yum install -y openssl-devel
```

**配置**
```
#将keepalived命令软连接到/usr/bin下
$ ln -s /usr/local/keepalived/sbin/keepalived /usr/bin/keepalived 

#添加启动脚本且方便用service keepalived start/stop/restart 管理
$ cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/keepalived  

#添加执行权限
$ chmod 755 /etc/init.d/keepalived 

#开机启动
$ chkconfig keepalived on 

#默认情况下，keepalived 会读取 /etc/keepalived 下keepalived.conf 文件
#如果没有建立这个文件，keepalived也不会报错，但是会发现，所创建的关于keepalived的相关参数根本就没有生效。
$ mkdir /etc/keepalived
$ ln -s /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf
```

**负载均衡：**
```
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
	## keepalived 自带的邮件提醒需要开启 sendmail 服务。 建议用独立的监控或第三方 SMTP
	router_id liuyazhuang133  ## 标识本节点的字条串，通常为 hostname
} 

## keepalived 会定时执行脚本并对脚本执行的结果进行分析，动态调整 vrrp_instance 的优先级。如果脚本执行结果为 0，并且 weight 配置的值大于 0，则优先级相应的增加。如果脚本执行结果非 0，并且 weight配置的值小于 0，则优先级相应的减少。其他情况，维持原本配置的优先级，即配置文件中 priority 对应的值。
vrrp_script chk_nginx {
	script "/etc/keepalived/nginx_check.sh" ## 检测 nginx 状态的脚本路径
	interval 2 ## 检测时间间隔
	weight -20 ## 如果条件成立，权重-20
}

## 定义虚拟路由， VI_1 为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
	state MASTER ## 主节点为 MASTER， 对应的备份节点为 BACKUP
	interface eth0 ## 绑定虚拟 IP 的网络接口，与本机 IP 地址所在的网络接口相同， 我的是 eth0
	virtual_router_id 33 ## 虚拟路由的 ID 号， 两个节点设置必须一样， 可选 IP 最后一段使用, 相同的 VRID 为一个组，他将决定多播的 MAC 地址
	#mcast_src_ip 192.168.50.133 ## 本机 IP 地址
	priority 100 ## 节点优先级， 值范围 0-254， MASTER 要比 BACKUP 高
	nopreempt ## 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
	advert_int 1 ## 组播信息发送间隔，两个节点设置必须一样， 默认 1s
	## 设置验证信息，两个节点必须一致
	authentication {
		auth_type PASS
		auth_pass 1111 ## 真实生产，按需求对应该过来
	}
	## 将 track_script 块加入 instance 配置块
	track_script {
		chk_nginx ## 执行 Nginx 监控的服务
	} #
	# 虚拟 IP 池, 两个节点设置必须一样
	virtual_ipaddress {
		192.168.50.130 ## 虚拟 ip，可以定义多个
	}
}
```

本机两个 `vrrp_instance` 组的 `virtual_router_id` 值不能相同，但对应备用节点的此值必须相同。


## 参考

[CentOS 版本选择：DVD、Everything、LiveCD、Minimal、NetInstall](https://blog.csdn.net/frank1998819/article/details/84774176)
[dpkg、rpm 和 apt-get、yum 的区别及使用](https://blog.csdn.net/lu_embedded/article/details/51994466)
[dpkg常用参数](https://blog.csdn.net/Blaider/article/details/7688902)
[yum (rpm) 和 apt-get的对应关系](https://blog.csdn.net/cuipengchong/article/details/52484920)
[第二十三章、软件安装： RPM, SRPM 与 YUM 功能](http://cn.linux.vbird.org/linux_basic/0520rpm_and_srpm_4.php)
[yum本地源 baseurl repodata repomd.xml comps.xml(一)](https://blog.51cto.com/kpshare/274730)
[合并 CentOS 6.8 的两个ISO镜像](https://www.cnblogs.com/Sunzz/p/6915150.html)
[Nginx 安装配置](https://www.runoob.com/linux/nginx-install-setup.html)
[Keepalived之——Keepalived + Nginx 实现高可用 Web 负载均衡](https://blog.csdn.net/l1028386804/article/details/72801492)

