---
title: openresty 入门
tags: 
	- java
	- openresty
categories: openresty
---

#  Openresty => Nginx + Lua

Nginx 是一个主进程配合多个工作进程的工作模式.每个进程由单个线程来处理多个连接.

通过将cpu内核绑定到工作进程上,来提升性能.



## 安装

### 预编译安装

参照: http://openresty.org/cn/installation.html

先在centos系统中添加openresty仓库.

- `yum install yum-utils`   管理repository及扩展包的工具 (主要是针对repository)
- `yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo`

然后安装软件包openresty.

- `yum install openresty`

### 源码编译安装

从官网中下载源码:

- http://openresty.org/cn/download.html

并解压 

- `tar -xzvf openresty-1.19.3.1.tar.gz`

进入解压目录并配置 

- `./configure`

  默认安装目录配置  --perfix=/usr/local/openresty

  可以通过--help查看更多可配置项

安装依赖

- `yum install gcc openssl-devel pcre-devel zlib-devel postgresql-devel`

编译安装

- `make && make install`

### 服务命令

启动 

- `service openresty start`

停止

- `service openresty stop`

重新加载配置文件

- `service openresty reload`

检查版本与配置文件是否正确(安装目录下nginx模块下)

- `Nginx -v`
- `Nginx -t`





## lua-nginx-module

### 创建配置文件lua.conf

```bash
server {
	listen 			8090;
	service_name	localhost;
}
location /lua {
	default_type	text/html;
	# 通过外部文件的方式引入脚本
	content_by_lua_file		lua_script/hello.lua; 
	# 通过代码块的方式引入脚本
	# content_by_lua_block {
	# 	ngx.say("<h1>hello world</h1>")
	# }
}
```

### 在nginx.conf中引用lua配置

`include lua.conf;`

### 创建外部lua脚本

`lua_script/hello.lua`

以openresty下载nginx模块为相对路径.

内容:

`ngx.say("<h1>hello world</h1>")` nginx返回内容.由于conf定义了类型为html.所以输出为html页面

### 获取nginx uri中的单一变量

```lua
ngx.say(ngx.var.arg_param)
#param为参数名 eg : localhost:8090?param=123
```

### 获取nginx uri中的全部变量

```lua
local  uri_args = ngx.req.get_uri_args()
for k,v in pairs(uri_args) do
	if type(v) == "table" then
		ngx.say(k," : ", table.concat(v,","),"<br/>")
	else
		ngx.say(k, " : ",v,"</br>")
	end
end
# 获取url中所有的变量,多键值对为table.
# eg: localhsot:8090?a=123&b=4&c=5&a=6
# 得到的结果为:
# a: 123,6
# b: 4
# c:5
```

### 获取nginx请求头信息

```lua
local headers = ngx.req.get_headers()
ngx.say("Hostr:", headers["Host"],"<br/>")
for k,v in pairs(uri_args) do
	if type(v) == "table" then
		ngx.say(k," : ", table.concat(v,","),"<br/>")
	else
		ngx.say(k, " : ",v,"</br>")
	end
end
```

### 获取post 请求参数

``` lua
# 如果请求体尚未被读取，请先调用该语句,强制本模块读取请求体
ngx.req.read_body() 

#返回请求体字符串。
#local body_data = ngx.req.get_body_data()  
# 返回一个请求体lua table
local post_args = ngx.req.get_post_args()  
for k, v in pairs(post_args) do  
    if type(v) == "table" then  
        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  
    else  
        ngx.say(k, ": ", v, "<br/>")  

    end  
end

```

### http 协议版本

`ngx.req.http_version()`

### 请求方法

`ngx.req.get_method()`

### 原始的请求头内容

`ngx.req.raw_header()`







