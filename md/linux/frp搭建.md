# 搭建frp内网穿透连接win10子系统

frp内网穿透是通过一个带有公网IP的服务器进行中转，对被控主机实现反向代理，用户通过访问frps（中转服务器）来实现对frpc（被控主机）的远程访问。

如今买了台matebook轻薄办公,之前的游戏本就一直闲置着.想想浪费.不如把它穿透了当个服务器用,性能非常好.

## 1. 前期准备

中转服务器: 一台阿里云服务器,(配置没啥要求,但带宽最好能好些)

被控主机: 一台连着家用宽带的闲置电脑(安装上win10 子系统Ubuntu并安装好了ssh服务)

## 2. 安装

### 2.1. 下载frp安装包

![下载frp安装包](frp搭建.assets\下载frp安装包.png)

我这里都是用的linux系统,所以直接下载了一个linux安装包.

分别传到`中转服务器`与`被控主机`的合适目录下,解压.

目录结构如图:

![image-20210407234735160](frp搭建.assets\frp目录结构.png)

其中`frpc.ini`是客户端(被控主机端)配置的参数,`frpc`是启动客户端的命令.

下面`frps.ini`是服务端(中转服务器)配置的参数,`frps`是启动服务端的命令.

它们都有带`_full`的后缀配置,这里包含了可配置的所有参数.需要添加配置是可以在里面寻找.

## 2.2 配置服务端参数并启动

### 2.2.1 更改配置文件

登录到阿里云服务器,更改frps.ini配置

```bash
[common]
# 服务器端端口
bind_port = 24841
# 客户端连接凭证
privilege_token = k3481
# 最大连接数
max_pool_count = 5                                                                                                           # 客户端映射的端口
vhost_http_port = 80                                                                                                         # 服务器看板的访问端口(可视化界面的访问接口)
dashboard_port = 24842
# 服务器看板账户(之后登入可视化界面的账户)
dashboard_user = admin
dashboard_pwd = amdin123  

```

### 2.2.2 启动服务

更改好后:eq保存退出并启动服务端服务

```bash
./frps -c frps.ini

2021/04/07 23:02:10 [I] [root.go:200] frps uses config file: frps.ini
2021/04/07 23:02:10 [I] [service.go:192] frps tcp listen on 0.0.0.0:24841
2021/04/07 23:02:10 [I] [service.go:235] http service listen on 0.0.0.0:80
2021/04/07 23:02:10 [I] [service.go:294] Dashboard listen on 0.0.0.0:24842
2021/04/07 23:02:10 [I] [root.go:209] frps started successfully
```

看到上述日志则表示服务已经正常启动.

### 2.2.3 阿里云控制面板中开启端口

这时候可以浏览器访问`ip:24842` (ip为服务器ip)来打开看板页面.

![](frp搭建.assets\看板1.png)

如果无法打开可以去阿里云控制面板开放相关的端口.

![image-20210408000338477](frp搭建.assets\开放阿里云端口1.png)



## 2.3 配置客户端参数并启动

 ### 2.3.1 更改配置文件

进入win10子系统,更改frpc.ini配置

```bash
[common]
# 服务器ip地址
server_addr = 45.94.175.69
# 服务器端端口
server_port = 24841
# 服务端设置的连接凭证
privilege_token = k3481

[ssh]
# 映射到服务端的端口
remote_port = 6051
#连接类型
type = tcp
# 客户端所在环境的ip
local_ip = 192.168.3.12
# 客户端ssh服务的端口
local_port = 22
```

### 2.3.2 启动服务

更改好后:eq保存退出并启动客户端服务

```bash
./frpc -c frpc.ini


2021/04/08 00:17:05 [I] [service.go:304] [ab0e6f44745eac11] login to server success, get run id [ab0e6f44745eac11], server udp port [0]
2021/04/08 00:17:05 [I] [proxy_manager.go:144] [ab0e6f44745eac11] proxy added: [ssh]
2021/04/08 00:17:05 [I] [control.go:180] [ab0e6f44745eac11] [ssh] start proxy success
```

看到上述日志则表示服务已经正常启动.

登录看板可以观察到ssh的映射记录已经在TCP目录下上线.

![image-20210408002318991](frp搭建.assets\ssh上线.png)

*同时保证子系统中的ssh服务开启,并对外可以访问. `service ssh start`*

### 2.3.3 阿里云控制面板中开启端口

需要再进入阿里云控制面板,将刚刚映射的ssh端口开启

![image-20210408002925080](frp搭建.assets\开端口2.png)

## 3. 连接

打开xshell,新建一个连接,主机为服务器ip,端口为ssh映射的端口.用户名密码为win10子系统的用户.这里只是通过跳板机连接到win10子系统中的ssh服务.

![image-20210408003244210](frp搭建.assets\连接验证1.png)

![image-20210408003827499](frp搭建.assets\连接验证2.png)

成功连接.\(^o^)/~



### 4 使用事项

配置中默认的超时时间为90秒.当ssh连接长时间不活动,会被服务主动关闭连接,不是网络波动的原因.







