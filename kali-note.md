---
title: 双系统 kali 下的折腾
categories:
  - 运维
tags:
  - kali
abbrlink: 6dae8fe5
date: 2019-02-19 13:28:27
---

&#8195;&#8195;本来只是想破解一下 wifi ，结果虚拟机无法识别网卡，最终装了 Win10+Kali 的双系统，Kali 跟 Ubuntu 还有些区别，期间遇到了 kali 黑屏无法启动的情况，这里整理下几乎是必备的几个软件： 进程管理 supervisor、代理 ss、浏览器代理 switchyomega、容器 docker、下载器 aria2 等。

<!-- more -->
&#8195;&#8195;破解 wifi 一般情况都需要 3070 和 8187 芯片的网卡，但是我只有一个 TPLINK WN725 的网卡，在虚拟机下无论安装 Linux 头文件，还是安装相应 [驱动程序](https://www.tp-link.com/us/download/TL-WN725N.html#Driver) 都没用，甚至 lsusb 压根就不识别这块 usb 网卡，索性我就直接物理机双系统了，过程中遇到的一些问题在此记录一下。

## Win+Linux双系统注意事项
### 简介
&#8195;&#8195;首先 Kali 是基于 Debian 测试版，很多资料搜不到的话可以参考 Debian，Debian 维护着至少三个发行版本：稳定版（stable）、测试版（testing）和不稳定版（unstable），稳定版多用于服务器，软件比较旧但非常稳定；测试版比较适合个人电脑使用，软件够新也比较稳定；想第一时间体验新功能的话可以选择不稳定版。[Kali’s Relationship With Debian](https://docs.kali.org/policy/kali-linux-relationship-with-debian)：
> The Kali Linux distribution is based on Debian Testing. Therefore, most of the Kali packages are imported, as-is, from the Debian repositories. In some cases, newer packages may be imported from Debian Unstable or Debian Experimental, either to improve user experience, or to incorporate needed bug fixes.

### Kali 安装步骤
&#8195;&#8195;默认已经装好了 Win，Linux 的具体安装过程可以参考附录文章文章：
> 1. 在 Win 磁盘管理中，选一个盘压缩 100G 出来；
> 2. 用 U 盘制作一个 Kali 启动盘;
> 3. 安装 Linux 过程中，**安装 GRUB 引导程序要选择手动输入**，填写 Linux 的路径，比如 `/dev/sda7`；
> 4. 安装完后进入 Win 启动 EasyBCD，设置开机引导。

### Kali 黑屏处理

&#8195;&#8195;某次重启后，选择 Linux 发现直接黑屏，只有一个光标闪啊闪，黑屏有两种情况，在 Grub 选了 Linux 之后黑屏和进入 Grub 前黑屏。
1. 在 Grub 之后黑屏，大部分是由 Nvidia显卡 和 nouveau 显卡冲突导致
> 在 Grub 界面按 `e` 进入编辑模式，在倒数第三行（linux开头的那行）结尾添加 ` nouveau.modeset=0`，然后 `ctrl+x` 保存并重启即可正常启动，然后修改写入 `/etc/default/grub` 文件就不必每次启动都编辑了。我印象里是因为运行了 `nvidia-xconfig` 导致黑屏，所以重新进入后删除了生成的 `/etc/X11/xorg.conf` 就好了，如果能用 `ctrl+alt+F1` 进入终端模式，直接删除也行。

2. 还没到 Grub 就黑屏，一般是引导程序坏了
> 这时候需要用 U 盘或者光盘启动，切换程序根目录后可以先查看 `/etc/X11/xorg.conf` 文件是否存在，存在的话删除之后看是否能够启动，依然不行就只好重装 Grub 引导了。进入的系统是 U 盘的系统，并非我们硬盘的系统，所以要切换到我们硬盘系统，首先挂载几个关键目录，然后安装 Grub 引导：
```
#查看硬盘及分区信息，找到我们 Linux 系统的具体路径
fdist -l 

#挂载 Linux 系统到 /mnt 目录
$ mount /dev/sda7 /mnt 

#挂载几个关键目录
$ mount --bind /dev /mnt/dev
$ mount --bind /dev/pts /mnt/dev/pts
$ mount --bind /proc /mnt/proc
$ mount --bind /sys /mnt/sys

# 改变程序执行的根目录
$ chroot /mnt

#安装grub，也可以--root-directory指定路径
$ grub-install /dev/sda
$ update-grub
$ exit

#解除挂载
$ umount /mnt/sys
$ umount /mnt/proc
$ umount /mnt/dev
$ umount /mnt

#重启
$ reboot
```

## 必备软件
### 国内源
&#8195;&#8195;为了方便以后更新系统升级软件，先改成国内的源：
```
vim /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
```

### 中文输入法 ibus
&#8195;&#8195;中文输入法必不可少：
```
$ apt-get install ibus ibus-pinyin
```
&#8195;&#8195;安装完后打开系统设置--Language--InputSources--Chinese(China)--Chinese(Pinyin) 就可以切换中文输入法了，但现在双击鼠标会删除选中的字符，设置一下：
```
$ ibus-setup
```
&#8195;&#8195;把这一项的勾去掉 `Embed preedit text in application window`。

### 代理 ShadowSocks
&#8195;&#8195;想要访问国外网站，还需要有一个代理：
```
$ pip install shadowsocks
$ mkdir -p /etc/shadowsocks
$ vim /etc/shadowsocks/shadowsocks.json

{
"server":"x.x.x.x",
"server_port":xxxx,
"local_port":1080,
"password":"xxxx",
"timeout":600,
"method":"aes-256-cfb"
}

$ sslocal -c /etc/shadowsocks/shadowsocks.json -d start #启动ss客户端

#测试
$ curl --socks5 127.0.0.1:1080 http://httpbin.org/ip
$ curl http://httpbin.org/ip
```

&#8195;&#8195;每次开机都启动一次 ss 客户端还是挺烦的，如果有很多个软件需要后台运行呢？我们还要监控进程状态，为了方便管理就要用到下面的 Supervisor，先贴一下配置信息：
```
$ vim /etc/supervisor/conf.d/shadowsocks.conf

[program:shadowsocks]
; directory = /home/leon/projects/usercenter ; 程序的启动目录
command = sslocal -c /etc/shadowsocks/shadowsocks.json ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = root          ; 用哪个用户启动
; redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
; stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
; stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
; stdout_logfile = /data/logs/usercenter_stdout.log
logfile = /var/log/shadowsocks.log
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

### 进程管理 Supervisor
&#8195;&#8195;日常使用中有很多守护进程在运行，我们要知道这些进程是否正常运行，更新配置后也需要重启等操作，Supervisor 就是一个进程管理工具，方便监控和管理后台进程：
```
$ apt-get install supervisor
$ vim /etc/supervisord.conf

; supervisor config file
...
...
[include]
files = /etc/supervisor/conf.d/*.conf

[inet_http_server]
port=127.0.0.1:9001
username=root
password=xxxx
```
&#8195;&#8195;配置文件中 `[include]` 部分说明我们需要管理的进程信息创建在 `/etc/supervisor/conf.d/` 下面就行，`[inet_http_server]` 部分是开启了 Web 页面，可以更直观的操作所管理进程的状态，修改了配置文件需要 `reload` 重启一下。下面是一些常用命令：
```
supervisord  #启动 supervisord，去几个默认目录找配置文件
/etc/init.d/supervisor start #启动supervisord
supervisord -c /etc/supervisor/supervisord.conf #启动supervisord，去指定目录找配置文件

supervisorctl status   #查看所有子进程的状态
supervisorctl stop xx  #关闭指定的子进程
supervisorctl start xx #开启指定的子进程
supervisorctl stop all #关闭所有子进程
supervisorctl update   # 更新新的配置到 supervisord
supervisorctl reload   #载入最新的配置文件，并按新的配置启动所有进程
```

### 下载器 Aria2
&#8195;&#8195;浏览器下载并不能满足我们的需求，Win 下离一定离不开迅雷，百度网盘，但其实我们并不想下载那么多软件，Aria2 是一款免费开源跨平台且不限速的多线程下载器，用来代替迅雷，百度网盘用一个插件代替，下面有提，先安装配置 Aria2：
```
$ apt-get install aria2 
$ mkdir /etc/aria2/
$ vim /etc/aria2/aria2.conf

## '#'开头为注释内容, 选项都有相应的注释说明, 根据需要修改 ##
## 被注释的选项填写的是默认值, 建议在需要修改时再取消注释  ##

## 文件保存相关 ##
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/root/Downloads
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
#disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
#file-allocation=none

# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数, 运行时可修改, 默认:5
#max-concurrent-downloads=5
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
#split=5
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0
# 单个任务上传速度限制, 默认:0
#max-upload-limit=0
# 禁用IPv6, 默认:false
#disable-ipv6=true
# 连接超时时间, 默认:60
#timeout=60
# 最大重试次数, 设置为0表示不限制重试次数, 默认:5
#max-tries=5
# 设置重试等待的秒数, 默认:0
#retry-wait=0

## 进度保存相关 ##

# 从会话文件中读取下载任务
input-file=/etc/aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/etc/aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
#save-session-interval=60

## RPC相关设置 ##

# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
#rpc-listen-port=6800
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
#rpc-secret=<TOKEN>
# 设置的RPC访问用户名, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-user=<USER>
# 设置的RPC访问密码, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-passwd=<PASSWD>
# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
#rpc-secure=true
# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件,
# 使用 PEM 格式时，您必须通过 --rpc-private-key 指定私钥
#rpc-certificate=/path/to/certificate.pem
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件
#rpc-private-key=/path/to/certificate.key

## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
#follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 单个种子最大连接数, 默认:55
#bt-max-peers=55
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=false
# 打开IPv6 DHT功能, PT需要禁用
#enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
#bt-enable-lpd=false
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=false
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true

```
&#8195;&#8195;需要注意的就是 `dir=`、`input-file=`、`save-session=` 的路径是否存在，按自己需求配置，然后用下面命令启动：
```
$ aria2c --conf-path=/etc/aria2/aria2.conf
```
&#8195;&#8195;类似之前的 ss 客户端，每次启动都要执行一条命令挺麻烦，还要看着后台进程是否异常退出，所以也用 Supervisor 来管理，配置如下：
```
$ vim /etc/supervisor/conf.d/aria2.conf
 
[program:aria2]
; directory = /home/leon/projects/usercenter ; 程序的启动目录
command = aria2c --conf-path=/etc/aria2/aria2.conf ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = root          ; 用哪个用户启动
; redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
; stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
; stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
; stdout_logfile = /data/logs/usercenter_stdout.log
logfile = /var/log/aria2.log
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```
&#8195;&#8195;Aria2 是一个命令行下的下载器，如果下载的文件较大，半天没反应还以为是卡死了，为了方便我们还是用一个 Web 界面来管理，YAAW 是一个开源项目，只要访问地址 `http://aria2c.com/` 即可进入 Web 界面,详细配置参考这里 [Aria2 & YAAW 使用说明](http://aria2c.com/usage.html)。

## 重要软件
### 容器 Docker
&#8195;&#8195;经常搭建环境的话，Docker 是非常好用的一个软件，按照官方文档：
```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
&#8195;&#8195;但是这里有个问题，Kali 去掉了 Debian 的 add-apt-repository，所以这一步会报错，一个办法是先安装该程序，另一个办法是手动导入PPA源：
```
echo 'deb [arch=amd64] https://download.docker.com/linux/debian stretch stable' > /etc/apt/sources.list.d/docker.list
```
&#8195;&#8195;如果官方源无法使用，还可以替换成国内的源：
```
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/debian \
   $(lsb_release -cs) \
   stable"
```

### 测试工具 BurpSuite
&#8195;&#8195;如果做 Web 安全测试，BurpSuite 也是必不可少的软件，其实 Kali 已经集成了这个软件，但是可能会出现 HTTPS 网站无法访问的情况，明明导入了证书，明明开启了代理，明明目标站点没用 HSTS ，明明表示一脸懵比，这一般是 JAVA 版本过高导致，`alternatives` 命令可以选择软件的版本。
```
$ java --version
$ sudo update-alternatives --config java
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
* 1            /usr/lib/jvm/java-10-openjdk-amd64/bin/java      1101      manual mode
  2            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
  3            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```
&#8195;&#8195;选择 10 版本的就行，重启 Burp 即可。

### 网易云
&#8195;&#8195;官方有 Linux版，虽然比较旧。安装完后有两个问题：
1. 点击图标无法启动，但是直接运行 `sudo netease-cloud-music` 可以启动，修改配置文件 `Exec` 的值如下：
```
$ vim /usr/share/applications/netease-cloud-music.desktop

Exec=sudo netease-cloud-music %U

```
&#8195;&#8195;如果想要在非 root 用户下（比如 zhangsan）点击图标启动，还要增加如下一行：
```
$ sudo vim /etc/sudoers

zhangsan ALL = NOPASSWD: /usr/bin/netease-cloud-music
```

2. 耳机没声音
&#8195;&#8195;这个问题很奇怪，外放没问题，插上耳机却没声音，按照网上的教程在终端运行 `pavucontrol` 做了设置（单击图标 PulaseAudio 也能进入设置项），但并没有效果，重启了一下却好了，然后第二天又不行了，不知道什么原因，随缘听吧。

### NodJs
&#8195;&#8195;NodJs 推荐用 `nvm` 安装，便于管理：
```
$ wget -qO- https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
$ nvm install stable
```
&#8195;&#8195;但是安装完发现 `nvm: command not found`，只要执行一下 `source .profile` 就行了，这样可以使修改后的环境变量立刻生效。

### 虚拟机
&#8195;&#8195;如果你想要做逆向，一个 Win 系统还是必备的，虚拟机比较主流的有 KVM、VMware、VirtualBox。安装方面没什么坑，但是如果你下了吾爱破解专用版的虚拟机，可能导入到 VirtualBox 之后没有网卡驱动，我尝试了各种办法最后放弃了 VirtualBox，猜测是 VMware 生成的虚拟机导入 VirtualBox 之后有兼容问题，用 VMware 导入后一切正常。

## 浏览器插件
### 自动代理 SwitchyOmega
&#8195;&#8195;有了 ss 客户端，我们还要设置浏览器代理才能使用，来回配置也是很麻烦，所以用一款开源的插件
 [SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega/releases) 来偷懒，支持 Chromium & Firefox，主要是 Rule List URL 填写对就行：
```
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

### 百度网盘 BaiduExporter
&#8195;&#8195;下载百度盘的大文件还需要使用百度盘客户端，百度盘有 Linux版？无论如何还是用插件  [BaiduExporter](https://github.com/acgotaku/BaiduExporter) 方便，安装完后下载页面会多出一个选项 **导出下载** ，单击后选择下面的 **ARIA2 RPC** ，就会直接调用 Aria2 下载，可以在 `http://aria2c.com/` 看到下载状态，需要注意的是要在网页 **登录百度盘帐号** 后，刷新一下才能正常下载。

### 其他
&#8195;&#8195;谷歌翻译插件也是必备，但没什么需要注意的。

## 参考
[Debian 发行版本](https://www.debian.org/releases/index.zh-cn.html)
[Kali Linux + Windows10双系统安装教程](https://www.cnblogs.com/superye/p/7288443.html)
[debian intel+nvidia不黑屏安装显卡驱动](https://blog.csdn.net/mhlwsk/article/details/51713530)
[Linux与Windows 10用grub引导教程](https://www.jianshu.com/p/5007e555ec12)
[双系统恢复linux的grub2系统引导](https://my.oschina.net/watcher/blog/376378)
[配置Linux客户端使用socks5代理上网](https://www.cnblogs.com/thatsit/p/6481820.html?utm_source=itdadao&utm_medium=referral)
[supervisor 安装、配置、常用命令](https://www.cnblogs.com/xueweihan/p/6195824.html)
[进程管理supervisor的简单说明](https://www.cnblogs.com/zhoujinyi/p/6073705.html）
[Get Docker CE for Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
[Installing Docker in Kali Linux 2018.1](https://medium.com/@calypso_bronte/installing-docker-in-kali-linux-2018-1-ef3a8ce3648)
[Docker —— 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/install/debian.html)
[gettin error code : SSL_ERROR_RX_RECORD_TOO_LONG](https://support.portswigger.net/customer/portal/questions/17434431-gettin-error-code-ssl-error-rx-record-too-long)
[Aria2 & YAAW 使用说明](http://aria2c.com/usage.html)

