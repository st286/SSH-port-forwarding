## [Using ssh tunnel as socks5 proxy](https://www.metahackers.pro/ssh-tunnel-as-socks5-proxy/)


---
# [SSH Academy](https://www.ssh.com/academy/ssh)

Setup socks5 tunel.
```
   ssh -NT -D 1080 usr@host
```
---
# SSH端口转发

---

## Contents

* [1. 实战 SSH 端口转发](#一实战-ssh-端口转发)
* [2. 动态端口转发：安装带有 SSH 的 SOCKS 服务器](#二动态端口转发安装带有-ssh-的-socks-服务器)
* [3. SSH的三种端口转发](#三SSH的三种端口转发)
* [4. SSH隧道：内网穿透实战](#四SSH隧道内网穿透实战)

---

## 一、实战 SSH 端口转发
from BM Developer
[原文](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html)


## 二、动态端口转发：安装带有 SSH 的 SOCKS 服务器

当我们谈论使用 SSH 进行动态端口转发时，我们说的是将 SSH 服务器转换为 SOCKS 服务器。那么什么是 SOCKS 服务器？

你知道 Web 代理是用来做什么的吗？答案可能是肯定的，因为很多公司都在使用它。它是一个直接连接到互联网的系统，允许没有互联网访问的内部网客户端让其浏览器通过代理来（尽管也有透明代理）浏览网页。Web 代理除了允许输出到 Internet 之外，还可以缓存页面、图像等。已经由某客户端下载的资源，另一个客户端不必再下载它们。此外，它还可以过滤内容并监视用户的活动。当然了，它的基本功能是转发 HTTP 和 HTTPS 流量。

一个 SOCKS 服务器提供的服务类似于公司内部网络提供的代理服务器服务，但不限于 HTTP/HTTPS，它还允许转发任何 TCP/IP 流量（SOCKS 5 也支持 UDP）。

例如，假设我们希望在一个没有直接连接到互联网的内部网上通过 Thunderbird 使用 POP3 、 ICMP 和 SMTP 的邮件服务。如果我们只有一个 web 代理可以用，我们可以使用的唯一的简单方式是使用某个 webmail（也可以使用 Thunderbird 的 Webmail 扩展）。我们还可以通过 HTTP 隧道)来起到代理的用途。但最简单的方式是在网络中设置一个 SOCKS 服务器，它可以让我们使用 POP3、ICMP 和 SMTP，而不会造成任何的不便。

虽然有很多软件可以配置非常专业的 SOCKS 服务器，但用 OpenSSH 设置一个只需要简单的一条命令：
```shell
 Clientessh $ ssh -D 1080 user@servidorssh
```
或者我们可以改进一下：
```shell
 Clientessh $ ssh -fN -D 0.0.0.0:1080 user@servidorssh
```
其中：

选项 -D 类似于选项为 -L 和 -R 的静态端口转发。像那些一样，我们可以让客户端只监听本地请求或从其他节点到达的请求，具体取决于我们将请求关联到哪个地址：
```shell
 -D [bind_address:] port
```
在静态端口转发中可以看到，我们使用选项 -R 进行反向端口转发，而动态转发是不可能的。我们只能在 SSH 客户端创建 SOCKS 服务器，而不能在 SSH 服务器端创建。

1080 是 SOCKS 服务器的典型端口，正如 8080 是 Web 代理服务器的典型端口一样。

选项 -N 防止实际启动远程 shell 交互式会话。当我们只用 ssh 来建立隧道时很有用。

选项 -f 会使 ssh 停留在后台并将其与当前 shell 分离，以便使该进程成为守护进程。如果没有选项 -N（或不指定命令），则不起作用，否则交互式 shell 将与后台进程不兼容。

**原文**
https://www.imooc.com/article/43700


## 三、SSH的三种端口转发

最近工作中经常需要ssh登录到某台跳板机，再连接受限网络环境中的某台服务器。以前经常用SSH端口转发这一功能，但周围的同事好像对这个并不清楚，这里记录一下以备其它同事询问。

### SSH一共提供了3种端口转发，分别是本地转发（-L参数）、远程转发（-R参数）和动态转发（-D参数)

接下来我就一一介绍这几种不同的转发方式的使用。

### 一些术语和约定

既然提到转发，就应该明白：这是三台主机之间要合作干的事。不然为何不两台主机直连，而要通过第三者转发？

本地主机：形式为IP或域名，你当前正在使用的这台机器；

远程主机：形式与本地主机一样。这里的远程并不是指实际的距离有多远，准确地说是另一台；

### 本地转发

本地转发，顾名思义就是把本地主机端口通过待登录主机端口转发到远程主机端口上去。

本地转发通过参数 -L 指定，格式：-L [本地主机:]本地主机端口:远程网络主机:远程网络主机端口。加上ssh待登录主机，这里就有了三台主机。

举例：
```shell
 ssh -L 0.0.0.0:50000:host2:80 user@host1
```
这条命令将host2的80端口映射到本地的50000端口，前提是待登录主机host1上可以正常连接到host2的80端口。

**畅想一下这个功能的作用：**

* 因为本地的mysql更顺手，想用本地的mysql客户端命令连接受限网络环境的mysql服务端。
* 本地安装了开发工具，想用这个开发工具连接受限网络环境中某个服务的远程调试端口。


### 远程转发

远程转发是指把登录主机所在网络中某个端口通过本地主机端口转发到远程主机上。

远程转发通过参数 -R 指定，格式：-R [登录主机:]登录主机端口:本地网络主机:本地网络主机端口。

举例：
```shell
 ssh -R 0.0.0.0:8080:host2:80 user@host1
```
这条命令将host2的80端口映射到待登录主机host1的8080端口，前提是本地主机可以正常连接host2的80端口。

**畅想一下这个功能的作用：**

* 本地网络中有一个http代理，通过这个代理可以上外网，因此通过这条命令将这个http代理映射到待登录主机的某个端口，这样受限网络环境中所有其它服务器即可使用这个http代理上外网了。
* 在本机开发了一个web应用，想拿给别人测试，但现在你却处在内网，外网是无法直接访问内网的主机的，怎么办！？很多人可能会说，找台有公网IP的主机，重新部署一下就行了。这样可行，但太麻烦。然而自从你了解了ssh的远程转发之后，一切都变得简单了。只需在本地主机上执行一下上面例子的命令即可实现外网访问内网的web应用。

**注意：**

* sshd_config里要打开AllowTcpForwarding选项，否则-R远程端口转发会失败。
* 默认转发到远程主机上的端口绑定的是127.0.0.1，如要绑定0.0.0.0需要打开sshd_config里的GatewayPorts选项。这个选项如果由于权限没法打开也有办法，可配合ssh -L将端口绑定到0.0.0.0，聪明的你应该能想到办法，呵呵。

### 动态转发

相对于本地转发和远程转发的单一端口转发模式而言，动态转发有点更加强劲的端口转发功能，即是无需固定指定被访问目标主机的端口号。这个端口号需要在本地通过协议指定，该协议就是简单、安全、实用的 SOCKS 协议。

动态转发通过参数 -D 指定，格式：-D [本地主机:]本地主机端口。相对于前两个来说，动态转发无需再指定远程主机及其端口。它们由通过 SOCKS协议 连接到本地主机端口的那个主机。

举例：
```shell
ssh -D 50000 user@host1。这条命令创建了一个SOCKS代理，所以通过该SOCKS代理发出的数据包将经过host1转发出去。
```
怎么使用？

* 用firefox浏览器，在浏览器里设置使用socks5代理127.0.0.1:50000，然后浏览器就可以访问host1所在网络内的任何IP了。
* 如果是普通命令行应用，使用proxychains-ng，参考命令如下：
  
```shell
    brew install proxychains-ng
    vim /usr/local/etc/proxychains.conf # 在ProxyList配置段下添加配置 "socks5 	127.0.0.1 50000"
    proxychains-ng wget http://host2 # 在其它命令行前添加proxychains-ng即可
```

如果是ssh，则用以下命令使用socks5代理：

```shell
   ssh -o ProxyCommand='/usr/bin/nc -X 5 -x 127.0.0.1:5000 %h %p' user@host2
```

**畅想一下这个功能的作用：**

* 想访问受限网络环境中的多种服务
* FQ
* ……

### SSH Over HTTP Tunnel

有一些HTTP代理服务器支持HTTP Tunnel，目前HTTP Tunnel的主要作用是辅助代理HTTPS请求，详情参见这里。今天在工作中竟发现有同事通过HTTP Tunnel连接ssh过去，猛然想起来HTTP Tunnel的原理里并没有限制连接的目标服务一定是HTTP或HTTPS服务，貌似只要是基于TCP的服务都可以。macOS下ssh走CONNECT tunnel稍微麻烦一点，参考命令如一下：

```shell
    brew install corkscrew
    ssh -o ProxyCommand='/usr/local/bin/corkscrew http_proxy_host http_proxy_port %h %p' user@host2
```


**原文**
https://jeremyxu2010.github.io/2018/12/ssh的三种端口转发/


## 四、SSH隧道：内网穿透实战

### SSH支持的端口转发模式

正文开始前先用5分钟过一遍ssh支持的端口转发模式，具体使用场景在下一节详述。太长不看的可直接跳到下一节。

#### 1.”动态“端口转发（SOCKS代理）：
```shell
 ssh -D 1080 JumpHost  # D is for Dynamic
```

区别于下面要讲的其他端口转发模式，-D是建立在TCP/IP应用层的动态端口转发。这条命令相当于监听本地1080端口作为SOCKS5代理服务器，所有到该端口的请求都会被代理（转发）到JumpHost，就好像请求是从JumpHost发出一样。由于是标准代理协议，只要是支持SOCKS代理的程序，都能从中受益，访问原先本机无法访问而JumpHost可以访问的网络资源，不限协议（HTTP/SSH/FTP, TCP/UDP），不限端口。

#### 2.本地端口转发

```shell
  ssh -L 2222:localhost:22 JumpHost  # L is for Local
```

这条命令的作用是，绑定本机2222端口，当有到2222端口的连接时，该连接会经由安全通道(secure channel)转发到JumpHost，由JumpHost建立一个到localhost（也就是JumpHost自己） 22端口的连接。

如果上述命令执行成功，新开一个终端，执行```ssh -p 2222 localhost```，登录的其实是JumpHost。

所以-L是一个建立在传输层的，端口到端口的转发模式，当然远程主机不仅限于localhost。

后面的stdio转发和ProxyJump可以看做是本地端口转发的升级版和便利版（参见OpenSSH netcat mode）

#### 3.远程端口转发

```shell
ssh -R 8080:localhost:80 JumpHost  # R is for Remote
```

顾名思义，远程转发就是在ssh连接成功后，绑定目标主机的指定端口，并转发到本地网络的某主机和端口：和本地转发相比，转发的方向正好反过来。
假如在本机80端口有一个HTTP服务器，上述命令执行成功后，JumpHost的用户就可以通过请求http://localhost:8080来访问本机的HTTP服务了。

**stdio转发（netcat模式）与ProxyJump**

```shell
ssh -W localhost:23 JumpHost
```

netcat模式可谓ssh的杀手特性：通过-W参数开启到目标网络某主机和端口的stdio转发，可以看做是组合了netcat(nc)和ssh -L。上述命令相当于将本机的标准输入输出连接到了JumpHost的telnet端口上，就像在JumpHost上执行telnet localhost一样，而且并不需要在本机运行telnet！

既然是直接转发stdio，用来做ssh跳板再方便不过（可以看做不用执行两遍ssh命令就直接跳到了目标主机），所以在ProxyJump面世前（OpenSSH 7.3），ssh -W常被用于构建主机到主机的透明隧道代理，而ProxyJump其实就是基于stdio转发做的简化，专门用于链式的SSH跳板。

### 使用场景

#### 建立代理

假设你在局域网A，HostB在局域网B，JumpHost有双网卡可以同时连接到局域网A和B。此时的你想要访问HostB上的web服务，便可以通过如下命令建立代理：
```shell
 ssh -D '[::]:1080' JumpHost
```

这样，浏览器设置代理socks5://localhost:1080后，就可以直接访问http://HostB了。

当然，还可以通过这个代理ssh登录到HostB:
```shell
ssh -oProxyCommand="nc -X 5 -x localhost:1080 %h %p" HostB
```
其中, nc需要BSD版（Ubuntu和OS X默认就是BSD版本），``-X 5``指定代理协议为SOCKS5，``-x``指定了代理地址，``%h %p``用于ProxyCommand中指代代理目的地（HostB）和目的端口。更多代理用法参见lainme姐的通过代理连接SSH和通过代理使用GIT。

**ssh -D也是最基本的翻墙手段之一**

### 通过公网主机穿透两个内网

好，现在进入一种更复杂的情况：你（HostA）和目标主机（HostB）分属不同的内网，从外界都无法直接连通。不过好在这两个内网都可以访问公网（JumpHost），你考虑通过一台公网机器建立两个内网之间的隧道。

于是在目标网络，你吩咐现场人员帮你连通公网主机：
```shell
# Host in LAN-B
ssh -qTfNn -R 2222:localhost:22 JumpHost
```
``-qTfNn``用于告知ssh连接成功后就转到后台运行，具体含义见下一节解释。

现在，你只需要同样登录到跳板机JumpHost，就可以通过2222端口登录HostB了：
```shell
# in JumpHost, login HostB
ssh -p 2222 localhost
```
**更进一步**

如果我们将2222绑定为公网端口，甚至都不用登录跳板机，从而直接穿透到HostB：
```shell
ssh -qTfNn -R '[::]:2222:localhost:22' JumpHost
```
（因为要绑定公网端口，请确保在JumpHost的/etc/ssh/sshd_config里，配置了GatewayPorts yes，否则SSH Server只能绑定回环地址端口。）

在HostA上执行:
```shell
ssh -p 2222 JumpHost # Login to HostB
```
这样还有一个好处，作为管理员可以直接禁用跳板机的shell权限，使他作为纯粹的隧道主机存在（见“安全性”一节）。

当然还有粗暴的方式，通过组合ssh -D和ssh -R打开Socks5代理：
```shell
# Host in LAN-B
ssh -qTfNn -D :1080 localhost  &&  \
ssh -qTfNn -R '[::]:12345:localhost:1080' JumpHost
```
上述命令在HostB创建了SOCKS代理，并且映射到了公网JumpHost的12345端口，整个内网对我们而言已经一览无余，ssh登录更是手到擒来：
```shell
  # Host in LAN-A
  ssh -oProxyCommand="nc -X 5 -x JumpHost:12345 %h %p" localhost
```
**限制访问**

然而，直接在公网主机上暴露穿透到内网的端口非常不安全。为提高安全性，我们把远程转发限制到回环地址， 这样就限制了只有有权限登录JumpHost的人才能穿透到局域网B。首先在HostB上设定远程转发：
```shell
# Host in LAN-B
ssh -qTfNn -R 2222:localhost:22 JumpHost
```

在HostA执行：
```shell
# Host in LAN-A
# 通过ProxyJump跳板登录到目标主机，即使跳板机用户不能分配tty也没关系
ssh -J JumpHost -p 2222 localhost
```
（如果要限制socks5代理的使用，道理也一样，不过是加一层由本机端口到跳板机socks5端口的本地转发而已）

如果OpenSSH版本<7.3, 需要用stdio转发(ssh -W)代替-J，该命令会先登录JumpHost，继而转发本机stdio到JumpHost，所以接下来的ssh登录操作如同是在JumpHost完成一样：
```shell
ssh -oProxyCommand="ssh -W %h:%p JumpHost" -p 2222 localhost
```

### 通常意义的”跳板“

通常意义的”跳板“，指的是连接发起端A，经由跳板机B->C->D，连接到目标主机E的过程。连接和数据流都是单向的，比起上述情况反而简单了许多。这里不再赘述，只举两个简单的例子说明。更多示例参见OpenSSH/Cookbook/Proxies and Jump Hosts
```shell
ssh -L 1080:localhost:9999 JumpHost -t ssh -D 9999 HostB
```
这条命令会在登录JumpHost时，建立本机1080端口到JumpHost 9999端口的转发，同时在JumpHost上执行ssh登录HostB，同时监听9999端口动态转发到HostB。于是，所有到本机1080端口的连接，都被代理到了远程的HostB上去。
```shell
ssh -J user1@Host1:22,user2@Host2:2222 user3@Host3
```
这条命令就是经由Host1, Host2，ssh登录到Host3的过程（需ssh版本高于7.3）。

### Tips

#### ssh执行为后台任务

``ssh -qTfNn``用于建立纯端口转发用途的ssh连接，参数具体含义如下：

``-q``: quiet模式，忽视大部分的警告和诊断信息（比如端口转发时的各种连接错误）
``-T``: 禁用tty分配(pseudo-terminal allocation)
``-f``: 登录成功后即转为后台任务执行
``-N``: 不执行远程命令（专门做端口转发）
``-n``: 重定向stdin为/dev/null，用于配合-f后台任务

#### 安全性

* 建议为端口转发建立专门的账户，使用随机密码（当然使用私钥登录更好），并且禁掉其执行命令的权限。最简单的方式为
```shell
  # add user tunnel-user for ssh port forwarding
  sudo useradd -m tunnel-user
  # generate 10 random passwords with 16 length
  pwgen -sy1 16 10
  # pick one password and set it to tunnel-user
  sudo passwd tunnel-user
  # disable shell for tunnel-user
  sudo chsh -s /bin/false tunnel-user
```
  更多可参考Ask Ubuntu

* 避免在公网直接暴露动态代理转发，很危险。 尽量远程端口转发到目标主机的ssh端口。这样需要远程接入的人可以自行ssh登录或打开本地Socks代理。

#### 保持连接

客户端设置（``~/.ssh/config``）：
```shell
Host *
     ServerAliveInterval 180
```
每180秒向SSH Server发送心跳包，默认累积三次超时即认为失去连接。

服务器端同样可以设置心跳（``/etc/ssh/sshd_config``），作用同理：

``ClientAliveInterval 180``

#### Windows 客户端

以PuTTY为例，假如这台Windows主机在内网，我们要借助公网主机的远程端口转发建立隧道：

1. 和往常一样，在Session菜单输入公网主机的IP和SSH端口
2. 在SSH菜单里勾选Don't start a shell or command at all，以建立一个纯隧道连接（不需要登录shell）
3. 展开SSH菜单，进入Tunnels子菜单：
    1. 勾选Remote ports do the same (SSH-2 only)，使远程端口监听公网连接。
    2. 输入具体端口配置，比如Source port（也就是远程主机要监听的端口）填写22222，Destination填写HostIP:22，其中HostIP为内网中SSH服务器的IP。
    3. 选择Remote, Auto，表示建立远程端口转发。
    4. 点击Add添加配置

点击Open登录公网主机即可建立隧道。

**原文**
https://cherrot.com/tech/2017/01/08/ssh-tunneling-practice.html


---

