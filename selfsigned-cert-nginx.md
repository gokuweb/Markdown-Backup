---
title: 自签名证书 + Nginx 实现 HTTP 升级 HTTPS
categories:
  - 运维
tags:
  - ssl
  - nginx
  - ReverseProxy
abbrlink: d95eae05
date: 2019-04-12 01:47:03
---
&#8195;&#8195;很多内网平台在开发时候并没有考虑 HTTPS，想要从 HTTP 升级到 HTTPS 一般有两种方法：一种是更新所有相关前后端代码；另一种是基本不改变代码，用 Nginx 把 HTTPS 反向代理到 HTTP 接口。第一种方法工作量有点大，本文介绍下第二种方法，当然首先要做的是生成自签名证书。
<!-- more -->

## 准备工作
&#8195;&#8195;**内网** 环境受影响的是 **软件安装升级** 和 **证书申请**，所以需要 ** [制作本地 yum](https://www.gokuweb.com/operation/76e13aa5.html)** 和 **自签名证书**，自签名证书稍后讲，制作本地 yum 库可参考链接内容，** [HTTPS 和 CA 基础知识](https://www.gokuweb.com/websec/873e23ee.html)** 查看这里 。
> 1. 制作本地 yum 库，为了自动处理软件之间的依赖关系，一般都会用 yum 等软件包管理器来安装更新软件，默认 yum 是从互联网获取各种软件包的，内网环境就需要制作本地 yum 库。 
> 2. 自签名证书，证书是否安全依赖于 CA 机构的认证，虽然有很多免费证书可以申请，但是内网 IP 是无法申请的，而且内网地址也没必要花钱做 CA 认证，这就需要我们制作自签名证书。

## 自签名证书过程
&#8195;&#8195;比如在内网的测试环境中，为了验证 HTTPS 下的一些问题，我们没必要申请昂贵的证书，这个时候就可以自签名一个证书，自签名证书步骤如下：
1. 创建根证书（Root CA）。
> 1. 生成私钥（.key），私钥只能自己拥有；
> 2. 生成证书请求文件（.csr），实际上就是把自身一些信息（国家、机构、域名、邮箱等）用第一步的私钥加密。
> 3. 自签名证书（.crt），用第一步的私钥给第二步的证书请求文件签名，根证书肯定是自签名的，CA 机构给自己发的证书。

2. 创建中间证书（Intermediate CA），用根证书给中间证书签名，中间证书再给用户签名，提高安全性。
> 1. 生成私钥（.key），要保证私钥只有自己拥有；
> 2. 生成证书请求文件（.csr），实际上就是把自身一些信息（国家、机构、域名、邮箱等）用自己的私钥加密产生的文件。
> 3. 根证书签名（.crt），用根证书的私钥给中间证书的证书请求文件签名，签名之后意味着 CA 机构信任中间证书。

3. 创建用户证书，用中间证书给用户签名，最终形成 **用户证书-->中间证书-->根证书** 的证书链，而浏览器信任根证书发布机构。这里的用户一般指服务提供商，如百度、阿里、腾讯等，他们的证书需要 CA 机构的签名，当然在他们看来我们也是用户，但是绝大部分环境都是单向 https 认证，我们不需要提供证书。
> 1. 生成私钥（.key），要保证私钥只有用户自己拥有；
> 2. 生成证书请求文件（.csr），由用户的私钥和用户自身的信息（国家、机构、域名、邮箱等）生成。其中用户的公钥和用户的信息是明文保存在证书请求文件中，而用户私钥的作用是对用户公钥及用户自身信息做签名，私钥不包含在证书请求中；
> 3. 中间证书签名（.crt），用中间证书的私钥给用户证书的证书请求文件签名，签名之后意味着中间证书信任用户证书。

## 自签名证书步骤
&#8195;&#8195;先一步一步来，掌握原理后可以 **<font color="Red">两条命令生成自签名证书</font>** ，[【传送门】](#easiest) 在这里。想仔细了解的话，先看看创建证书过程中用到的 `openssl` 命令：

| 命令   | 功能 | 备注 |
| ------ | ---------------------------------------- | ----------- | -------- |
| req    | 生成证书请求文件和自签名证书 | 创建和处理 PKCS#10 格式的证书 |
| x509   | X.509 证书管理。<br>显示证书信息、转换证书格式、签名证书请求及改变证书信任设置 | 证书工具 |
| ca     | 签发证书请求和生成CRL，维护一个已签发证书状态的文本数据库 | 证书中心 |
| verify | X.509证书验证 | 证书验证 |

&#8195;&#8195;在 CentOS 系统下 `openssl` 配置文件的路径在 `/etc/pki/tls/openssl.cnf`，`req` 和 `ca` 需要读取配置文件里面的很多设置，但 `x509` 不读取配置文件，一切配置都由 x509 自行提供。`req` 下有个 `[-x509]` 参数，`[x509]` 下也有一个`[-req]` 参数，不是一个东西，不要混淆。

### 创建根证书
**1. 生成私钥**
```
$ openssl genrsa -out ca.key 2048
```
`genrsa` 有很多参数，常用的有：
> 1. `[-out]`，指定生成文件名；
> 2. `[-des|-des3|-idea]`，采用什么加密算法来加密我们的密钥，使用其中任一参数，在生成密钥的过程都会要求输入密码，缺省情况无需输入密码，我们这里就没有加密；
> 3. `[numbits]`，指明产生参数的长度。必须是最后一个参数。我们这里是 2048，缺省则产生 512bit 长的参数（不过我看配置文件默认好像也是2048？）。

**2. 生成证书请求文件**
&#8195;&#8195;这个过程会要求输入很多信息如国家、城市、组织信息等，其中 `Common Name (eg, your name or your server's hostname)` 是 **<font color="Red">必填项</font>** ，可以是域名或者 IP，其他都可以回车跳过，但是这样的话在签名证书时候会报错，下一章详述，不过自签名证书不影响。
```
$ openssl req -new -key ca.key -out ca.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:     
State or Province Name (full name) []:       
Locality Name (eg, city) [Default City]:     
Organization Name (eg, company) [Default Company Ltd]:          
Organizational Unit Name (eg, section) []:                     
Common Name (eg, your name or your server's hostname) []:myca    
Email Address []: 
An optional company name []:
```
> 1. `[-new]`，生成证书请求文件，它会提示用户关于一些字段的值。可以通过 `openssl.cnf` 来查看默认会询问那些字段，最大/小长度限制等，如果 `[-key]` 没有指定，则会按照默认配置文件生成新的私钥>；
> 2. `[-key]`，指定已有的密钥文件；
> 3. `[-out]`，指定生成文件名。

&#8195;&#8195; **<font color="Red">Common Name 是必填项，如果是根证书 Common Name 可以写公司名、字母缩写等，我上面用的 myca，但如果是用于服务器的证书，这里必须用域名或者 IP</font>** 。


**3. 生成自签名证书**
```
$ openssl req -x509 -days 3650 -key ca.key -in ca.csr -out ca.crt
```
或者
```
$ openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt
Signature ok
subject=/C=XX/L=Default City/0=Default Company Ltd/CN=myca
```
> 1. `[-x509]`，`req` 中的 `[-x509]` 表示生成一个自签名的证书，而不是一个证书请求；
> 2. `[-req]`，`x509` 中的 `[-req]` 表示 in 后面的输入文件为证书请求文件，默认是证书文件；
> 3. `[-signkey]`，用于提供自签名时的私钥文件。

**4. 查看证书信息**

**查看证书请求（csr）信息**
```
$ openssl req -in ca.csr -text -verify -noout
```

**查看证书（crt）信息**
```
$ openssl x509 -in ca.crt -text -noout
```
&#8195;&#8195;`x509` 用于查看证书（crt）的信息，`req` 用于查看证书请求（csr）的信息，当然证书和证书请求并不是靠后缀来区分的。
> 1. `[-text]`，以 text 格式输出证书请求；
> 2. `[-noout]`，不输出证书请求中的编码版本信息；
> 3. `[-verify]`，校验请求文件的签名信息。

证书信息包含：
> 1. 数字签名（Signature Algorithm）
> 2. 有效时间（Validity）
> 3. 主体（Issuer）
> 4. 公钥（Public Key）
> 5. X509v3 扩展，openssl config 中配置了 v3_ca，所以会生成此项

### 创建用户证书
**1. 生成私钥**
```
$ openssl genrsa -out server.key 2048 
```

**2. 生成证书请求**
```
$ openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:     
State or Province Name (full name) []:       
Locality Name (eg, city) [Default City]:     
Organization Name (eg, company) [Default Company Ltd]:          
Organizational Unit Name (eg, section) []:                     
Common Name (eg, your name or your server's hostname) []:www.myweb.com 
Email Address []: 
An optional company name []:
```

&#8195;&#8195;其中 `Common Name (eg, your name or your server's hostname)` 是 **<font color="Red">必填项</font>** ，要填写自己的域名或者 IP，其他都可以回车跳过。

**问题：**
&#8195;&#8195;如果 `Common Name` 和自己的域名或者 IP 不符，会报错 **<font color="Red">NET::ERR_CERT_INVALID</font>** ，但如果是用于根证书，`Common Name` 甚至可以写公司名、字母缩写等字符串。


**3. 生成证书（根证书对用户的证书请求签名，最终生成用户证书）**
```
$ openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```
&#8195;&#8195;生成私钥和证书请求这两步，和根证书的操作基本一样，第三步根证书是用 `[-x509]` 自签名，而这里是用根证书给用户的证书请求签名，用了 `[ca]` 命令：
> 1. `[-cert]`，指定 CA 证书；
> 2. `[-keyfile]`，指定私钥。

**问题 1：**
报错 <font color="Red">The mandatory stateOrProvinceName field was missing</font>。
&#8195;&#8195;这是因为在生成证书请求 `server.csr` 时候只填写了 `Common Name` 这一项，而配置文件 `openssl.cnf`中配置如下：
```
# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

# For the 'anything' policy
[ policy_anything ]
countryName             = optional
stateOrProvinceName     = optional 
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
```
&#8195;&#8195;`match` 意味着要匹配才行，而我们都没填所以会报错，解决办法就是使用参数 `[-policy]` ，从配置文件中可以看到 `policy_anything` 其实就是把 `match` 变成了 `optional`，命令如下：
```
$ openssl ca -policy policy_anything -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```

**问题 2：**
报错 <font color="Red">Using configuration from /etc/pki/tls/openssl.cnf <br> I am unable to access the /etc/pki/CA/newcerts directory <br> /etc/pki/CA/index.txt: No such file or directory
unable to open '/etc/pki/CA/index.txt'</font>
&#8195;&#8195;有可能是权限问题，也有可能是缺少必要文件，证书生成过程中主要会用到如下文件：
> 1. /etc/pki/tls/openssl.cnf
> 2. /etc/pki/CA/newcerts
> 3. /etc/pki/CA/index.txt
> 4. /etc/pki/CA/serial

**openssl ca 方式解决**
&#8195;&#8195;`openssl.cnf` 是配置文件，`index.txt` 和 `serial` 分别是证书索引数据库文件和证书序列号文件，创建这些文件即可：
```
$ touch CA/index.txt
$ touch CA/serial
$ echo "01" > CA/serial
```

现在继续生成证书：
```
$ openssl ca -policy policy_anything -in server.csr \
 -out server.crt -cert ca.crt -keyfile ca.key \
 -config myopenssl.cnf
```

<B>openssl x509 方式解决</B>
也可以使用 `x509` 工具生成证书，因为它默认不使用 `openssl.cnf` 所以没有上面那么多问题：
```
$ openssl x509 -req -in server.csr -CA ca.crt \
 -CAkey ca.key -out server.crt -CAcreateserial
```
> 1. `[-req]`，使用该参数时，`[-in]` 后面将要使用证书请求文件，不指定该参数时，x509 工具默认以证书文件做为输入；
> 2. `[-CA]`，指定根证书；
> 3. `[-CAkey]`，指定根证书私钥；
> 5. `[-CAcreateserial]`，如果 CA 使用的序列号文件不存在将自动创建。

**问题 3：**
报错 <font color="Red">failed to update database TXT_DB error number 2</font>。
&#8195;&#8195;`CommonName` 冲突造成的，应该是目前签名的证书与另一个证书的 `CommonName` 重复造成，删除 `index.txt` 文件即可。

**4. 查看、校验证书**

**查看证书请求（csr）信息**
```
$ openssl req -in server.crt -text -verify -noout
```

**查看证书（crt）信息**
```
$ openssl x509 -in server.crt -text -noout
```

**5. 验证证书有效性**

&#8195;&#8195;verify 命令对证书的有效性进行验证，verify 指令会沿着证书链一直向上验证，直到一个自签名的 CA：
```
$ openssl verify -CAfile ca.crt server.crt
server.crt: OK
```
&#8195;&#8195;`[-CAfile]`，指定 CA 的证书文件，这个文件里可能不只包含一个证书。如果需要对证书链进行验证，指定的文件中应包含所有的证书，需要把证书链中的证书都包含，证书文件都是文本文件，简单地使用 `cat` 命令就可以进行连接。

**6. 生成 pem 格式证书（可选）**
&#8195;&#8195;有时需要用到 pem 格式的证书，可以用以下方式合并证书文件（crt）和私钥文件（key）来生成： 
```
$cat server.crt server.key > server.pem
```

&#8195;&#8195;现在把生成的私钥 `server.key` 和证书 `server.crt` 配置到 nginx 就已经可以使用了，但是还有些问题，比如最直观的的，Chrome 浏览器会提醒“您的连接不是私密连接”，然后显示一个红色叉号。

### Chrome 下的问题
&#8195;&#8195;现在证书是生成了，但在 Chrome 下还有一些问题如下：
![https-warning2](/imgs/https-warning2.png)

在 F12--Security 下是这样：
![https-warning1](/imgs/https-warning1.png)

这三个警告的原因分别是：
> 1. 网站的证书链中存在 SHA-1 签名算法，改为 SHA256 即可。
> 2. 网站证书的 Subject Alternative Nam 扩展中没有域名或者 IP，Chrome 是根据证书中的 SAN 来判断访问的域名和证书中的域名是否一致，所以图 1 中提示“此服务器无法证明其所在网域是xxx”，添加 SAN 扩展并指定正确的域名或者 IP 即可。 
> 3. 网站证书不可信，浏览器导入根证书即可。

### 解决办法
**关于第一个问题 SH1 签名算法**
&#8195;&#8195;因为浏览器验证的是服务器证书，虽然第一个警告提示证书链中存在在 SHA-1 签名算法，但我推测与根证书自签名的算法无关，应该指的是中间证书和服务器证书的签名算法，因为我查看了本机“受信任的根证书机构”，很多证书也是用的 SHA-1 签名算法，这个用 `[ca]` 的 `[-md]`参数指定签名算法为 sha256 即可。

**关于第三个问题证书不可信**
&#8195;&#8195;证书不可信，导入根证书即可，其实自签名的根证书可以直接用作服务端证书，下一章我们会说，这样最明显的缺点就是你又上线了一个不同域名的网站，那还要重新生成一个自签名证书，重新导入一次。但是根证书只用于签名的话，不管多少域名多少网站，只要导入根证书，用这个根证书签名的网站证书就都是可信的，不需要一次次导入。

**关于第二个问题 SAN 扩展**
&#8195;&#8195;重点说一下，我们在生成证书请求时候填过一个 `Common Name`，以前 Chrome 是根据证书的 CN（Common Name）来匹配域名或 IP，但是现在改为用 SAN（Subject Alternative Name）扩展来匹配域名或 IP 了（但服务器证书的 Common Name 依然需要和自己的域名或者 IP 匹配），所以 **<font color="Red">需要在签名时候指定 SAN 扩展</font>** ，其实生成证书请求和生成证书这两个阶段都能选择添加 SAN 扩展，我分别尝试仅在证书请求阶段或者签名阶段添加 SAN 扩展，签名阶段添加才会有效。

**openssl ca 方式解决**
<span id="san-extension">添加 SAN 扩展需要修改配置文件<span>：
1. 取消 `/etc/pki/tls/openssl.cnf` 中 `[req]` 节点下这句注释 `req_extensions= v3_req`，如果不想使用默认配置文件，也可以`openssl.cnf` 是我们在第二节中生成的配置文件，也可以直接在 `openssl.cnf` 进行配置；

2. 在 `[v3_req ]` 节点下新增属性 `subjectAltName = @alt_names`；

3. 新增节点：
```
[ alt_names ]
DNS.1 = localhost
DNS.1 = www.abc.com
IP.1 = 192.168.1.1
```

4. 用如下命令签名证书：
```
$ openssl ca -days 3650 -md sha256 -in server.csr \
 -out server.crt -cert ca.crt -keyfile ca.key -policy \
 policy_anything -config myopenssl.cnf -extensions v3_req
```
> 1. `[-md]`，指定签名算法；
> 2. `[-config]`，指定配置文件；
> 3. `[-extensions]`，指定证书扩展。


**openssl x509 方式解决**


```
$ openssl x509 -req -sha256 -days 3650 -in server.csr \
 -CA ca.crt -CAkey ca.key -out server.crt -CAcreateserial \
 -extfile openssl.cnf -extensions v3_req
```

根证书导入浏览器后，上面问题就全部解决了。

### <span id="custom-config">自定义配置文件（可选）</span>
&#8195;&#8195;CentOS 下 `openssl` 默认使用的配置文件是 `/etc/pki/tls/openssl.cnf`，不过生成证书的过程中不仅仅依赖这一个文件，自定义配置文件还需要生成其他一些必备的文件：
```
$ mkdir -p CA/newcerts
$ cp /etc/pki/tls/openssl.cnf myopenssl.cnf
$ touch CA/index.txt
$ touch CA/serial
$ echo "01" > CA/serial
```
&#8195;&#8195;还需要修改 `/etc/pki/tls/openssl.cnf` 中 `[ CA_default ]` 下的 `dir` 为 `dir = ./CA`，这样就把根路径指定到了当前目录下的 `CA` 文件夹，现在我们可以使用自己的配置文件来生成证书：
```
$ openssl ca -policy policy_anything -in server.csr \
 -out server.crt -cert ca.crt -keyfile ca.key \
 -config myopenssl.cnf
```

## 自签名证书总结
### <span id="openssl-config">修改配置文件</span>
`sudo vim /etc/pki/tls/openssl.cnf`，确保以下信息存在：
```
...
...
[ req ]
default_bits    = 2048
...
req_extensions= v3_req 

[ v3_req ]
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.1 = www.abc.com
IP.1 = 192.168.1.1

...
...
```
主要是针对 SAN 扩展的修改，可以参考配合上一章内容，点 [传送门](#san-extension)。

### 根证书
```
# 生成根证书私钥
$ openssl genrsa -out ca.key 2048

# 生成根证书请求文件
$ openssl req -new -key ca.key -out ca.csr

# 生成根证书（自签名）
$ openssl req -x509 -days 3650 -key ca.key -in ca.csr -out ca.crt

# 生成根证书（效果等同上一句）
$ openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt

# 查看证书请求文件
$ openssl req -in ca.csr -text -verify -noout

# 查看证书
$ openssl x509 -in ca.crt -text -noout
```

### 用户证书
```
# 生成服务端私钥
$ openssl genrsa -out server.key 2048

# 生成服务端证书请求文件
$ openssl req -new -key server.key -out server.csr

# 生成服务端证书（根证书给服务端证书签名）
$ openssl ca -days 3650 -md sha256 -in server.csr \
 -out server.crt -cert ca.crt -keyfile ca.key -policy \
 policy_anything -config openssl.cnf -extensions v3_req

# 生成服务端证书（效果等同上一句）
$ openssl x509 -req -sha256 -days 3650 -in server.csr \
 -CA ca.crt -CAkey ca.key -out server.crt -CAcreateserial \
 -extfile openssl.cnf -extensions v3_req

# 查看服务端证书请求信息
$ openssl req -in server.crt -text -verify -noout

# 查看服务端证书信息
$ openssl x509 -in server.crt -text -noout
```

## <span id="easiest">懒人版自签名证书</span>
&#8195;&#8195;以上我们都是先生成私钥 `ca.key` 和根证书 `ca.crt`，再用 `ca.key` 和 `ca.crt` 给用户证书请求 `server.csr` 签名，最简单的方法是直接使用自签名的根证书 `ca.crt`，当然名字无所谓，我们下面还用 `server.crt`。

### 一句话生成证书

1. **<font color="Red">需要先配置好 openssl.cnf 文件</font>**，其实就是配置一下 SAN 扩展，配置方法点 [传送门](#openssl-config)，也可以自定义配置文件，详细点 [传送门](#custom-config)。 

2. 生成证书
```
$ openssl req -x509 -nodes -sha256 -newkey \
 rsa:2048 -keyout server.key -out server.crt \
 -config openssl.cnf -extensions v3_req
```
> 1. `[-nodes]`，如果生成了私钥，则不加密，该参数可以避免每次重启 nginx 都要输一次密码。
> 2. `[-sha256]`，指定签名算法；
> 3. `[-newkey]`，建立一个新的证书请求和一个新的 private key，后面需要指定秘钥位数比如 `rsa:1024`，生成的密钥名称由 `[-keyout]` 指定；
> 4. `[-keyout]`，指定生成秘钥的名称；
> 5. `[-config]`，指定配置文件；
> 6. `[-extensions]`，指定扩展。

### 两句话生成证书
&#8195;&#8195; X509 工具优点是 **<font color="Red">不依赖配置文件 openssl.cnf，但仍然需要先提供 SAN 扩展信息</font>** 。X509 工具无法自己生成证书请求文件，所以需要先用 `openssl req` 生成证书请求，虽然没用 `genrsa` 但还是生成了私钥，可能是 `openssl req` 使用了默认配置：
```
$ openssl req -nodes -new -keyout server.key -out server.csr
$ openssl x509 -req -sha256 -days 3650 -in server.csr \
 -signkey server.key -out server.crt \
 -extfile <(printf "subjectAltName=DNS:localhost,IP:192.168.1.1")
```
> 1. `[-nodes]`， 密钥文件不加密。不加此参数会要求 “Enter PEM pass phrase：” ，可能这种方式生成的 key 和 `openssl genrsa -des3` 差不多；
> 2. `[-signkey]`，用于提供自签名时的私钥文件；
> 3. `[-extfile]`，指定签名时包含要添加到证书中的扩展项的文件，<font color="Red">自签名时候可以这样使用，根证书签名时候还是要用配置文件</font>，直接这样用报错 `x509 error loading the config file /dev/fd/63`，原因未知。

<B>注意：</B>
&#8195;&#8195;上面是 X509 工具生成自签名证书的命令，如果是通过 X509 工具用根证书给用户证书签名，是不可以直接在命令中指定 SAN 扩展的，会报错 <font color="Red">x509 error loading the config file /dev/fd/63</font>，原因未知，但是可以用下面方法来签名：
1. 配置 SAN 扩展，新建配置文件 `v3.ext` 如下：
```
$ vim v3.ext

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.1.1 
DNS.1 = localhost
#DNS.2 = www.myweb.com

```

2. 生成证书
```
$ openssl x509 -req -sha256 -days 3650 -in server.csr \
 -CA ca.crt -CAkey ca.key -out server.crt \
 -extfile v3_ext -CAcreateserial
```

## Nginx 配置
&#8195;&#8195;静态资源的配置很简单，Nginx 上开启 ssl 配置后就可以了，但是动态资源要和后端做交互，在发送 ajax 请求或者要获取 http 资源的时候，就会造成 https 请求无法请求 http 后端的接口，有两种解决方法：
> 1. 将后端的项目添加 https 支持，所有前端的 http 请求都要改成 https ，但这样工作量较大。
> 2. 后端接口依然是 http 请求，但是客户端到 Nginx 用 https，请求后端接口时候，用 Nginx 反向代理到 http 接口上。

**nginx.conf**
```
...
...

upstream myweb { 
      server 192.168.1.11:7000; 
      server 192.168.1.12:8000; 
      keepalived 18;
}
server {
    listen 80;  
    server_name www.xxx.com; # 域名，或者 IP 比如 192.168.2.33
    rewrite ^/(.*) https://$server_name$request_uri? permanent; # 80端口全部重定向到 https
    #if ($scheme = http){
    #    return 301 https://$host$request_uri;
    #}
}

server {
    listen 443;
    server_name www.xxx.com;  # 域名，或者 IP 比如 192.168.2.33
    
    ssl on;
    ssl_certificate   myca/server.crt;
    ssl_certificate_key  myca/server.key;
    
    ssl_session_timeout 5m;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    
    
    location / {
        root  /var/www/html;   # 静态资源路径地址
        index index.html index.htm;  
    }
    location /api {
        proxy_pass http://myweb;
	proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_http_version 1.1;
    }
}
...
...
```

nginx 命令：
```
$ /usr/local/nginx/sbin/nginx -t # 检查配置文件
$ /usr/local/nginx/sbin/nginx -s reload # 重新加载
```

## 引申
### 什么是x509证书
X509 其实就是证书的一种规范，严格说与文件扩展名也没有直接关系，详细参考[维基百科 x.509](https://zh.wikipedia.org/wiki/X.509)：
> &#8195;&#8195;X.509 是密码学里公钥证书的格式标准。 X.509 证书己应用在包括 TLS/SSL 在内的众多 Intenet 协议里，同时它也用在很多非在线应用场景里，比如电子签名服务。X.509 证书里含有公钥、身份信息（比如网络主机名，组织的名称或个体名称等）和签名信息（可以是证书签发机构 CA 的签名，也可以是自签名）。对于一份经由可信的证书签发机构签名或者可以通过其它方式验证的证书，证书的拥有者就可以用证书及相应的私钥来创建安全的通信，对文档进行数字签名.
> &#8195;&#8195;另外除了证书本身功能，X.509 还附带了证书吊销列表和用于从最终对证书进行签名的证书签发机构直到最终可信点为止的证书合法性验证算法。

x509 证书一般会用到三种类型的文件：key、csr、crt。
`key` 是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密。
`csr` 是证书签名请求文件，用于提交给证书颁发机构（CA）对证书签名。
`crt` 是证书文件，由证书颁发机构（CA）签名或者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息，签署人用自己的 key 给你签署的凭证。

### 证书格式和编码方式
**证书格式：**
> x509 ，这种证书只有公钥，不包含私钥。
> pcks#7 ，这种主要是用于签名或者加密。
> pcks#12 ，这种含有私钥，同时也含有公钥，但是有口令保护。

**编码方式：**
> .pem 后缀的证书都是 base64 编码。
> .der 后缀的证书都是二进制格式。

&#8195;&#8195;x509 是数字证书的规范。PKCS#7 和 PKCS#12 是两种封装形式。
&#8195;&#8195;P7 一般是把证书分成两个文件，一个公钥一个私钥，有 PEM 和 DER 两种编码方式。**PEM 是纯文本的 base64 编码**，P7 一般是分发公钥用，看到的就是一串可见字符串，扩展名经常是 `.crt`、`.cer`、`.key` 等。**DER 是二进制编码**。
&#8195;&#8195;P12是把证书压成一个文件，`.pfx` 。主要是考虑分发证书，私钥是要绝对保密的，不能随便以文本方式散播。所以 P7 格式不适合分发。`.pfx` 中可以加密码保护，所以相对安全些。
&#8195;&#8195;在实践中要中，用户证书都是放在 USB Key 中分发，服务器证书经常还是以文件方式分发。服务器证书和用户证书都是X509证书，就是里面的属性有区别。

**格式转换**
1. crt + key 转 pfx【pfx是证书安装包，方便在电脑上直接双击按向导安装证书】
```
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx
```

2. pfx 转化为 pem【curl 需要 pem 格式文件】
```
$ openssl pkcs12 -in client.pfx -nodes -out client.pem
```
3. crt + key转 p12【apache 的 cxf 客户端支持 jks 和 p12 证书】
```
$ openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
```

4. crt 转 jks【jks 支持存放信任证书，而 pkcs12 不支持，所以 tomcat 环境下配置 ca 只能使用 jks 才能保证 ca 密钥不泄露】
```
$ keytool -import -v -trustcacerts -storepass defaultpwd -keypass defaultpwd -file ca.crt -keystore ca_only.jks
```

### 部分参考图片
生成了 pem 格式的根证书，自签名证书，所以无需生成证书请求文件：
![root-ca](/imgs/root-ca.png)

生成了客户端证书请求文件：
![clint-csr](/imgs/client-csr.png)

生成了客户端证书，使用了默认的配置文件 `openssl.cnf`：
![client-crt](/imgs/client-crt.png)


## 参考
[OpenSSL中文手册之命令行详解（未完待续）](https://blog.csdn.net/liao20081228/article/details/77159039)
[ 手册 OpenSSL 之 req 命令](http://blog.chinaunix.net/uid-20054105-id-1979624.html)
[OpenSSL简介](https://blog.csdn.net/naioonai/article/details/80984032)

[细说 CA 和证书](https://www.barretlee.com/blog/2016/04/24/detail-about-ca-and-certs/)
[CA认证](https://www.cnblogs.com/lzcys8868/p/6281932.html)
[openssl x509(签署和自签署)](https://www.cnblogs.com/f-ck-need-u/p/6090885.html)
[openssl TXT_DB error number 2 failed to update database](https://blog.csdn.net/avilifans/article/details/38415053)
[chrome 是不是不支持内网 ip 的 ssl 证书了？](https://www.v2ex.com/t/376834)
[笔记：OpenSSL 生成「自签名」证书遇到的 missing_subjectAltName 问题](https://moxo.io/blog/2017/08/01/problem-missing-subjectaltname-while-makeing-self-signed-cert/)
[openssl生成SSL证书的流程](https://blog.csdn.net/liuchunming033/article/details/48470575)
[总结之：CentOS6.5下openssl加密解密及CA自签颁发证书详解](https://blog.51cto.com/tanxw/1379417)


[使用nginx作为反向代理解决前后端分离时前端https,后端http造成访问无法被加载](https://blog.csdn.net/qq_37105358/article/details/80854559)
[全站 HTTPS 没你想象的那么简单](https://www.cnblogs.com/mafly/p/allhttps.html)
[Stack Overflow 的 HTTPS 化：漫漫长路的终点](https://juejin.im/post/594735e861ff4b006cf6a2e3)

[x509和pkcs12以及pkcs7的关系](https://bbs.csdn.net/topics/190044123)
[FILIPPO.IO](https://blog.filippo.io/mkcert-valid-https-certificates-for-localhost/)

