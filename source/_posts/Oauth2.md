---
title: Oauth2 理解
tags: 
	- 概念
categories: 概念
date: 2021-3-30 00:42:13
---



# Oauth2 理解

## 1. 什么是Oauth2

​     Oauth是一套开放授权的标准/协议.  旨在让用户允许第三方应用去访问该用户存储在另外服务提供者上的用户信息,而不需要将用户名与密码提供给第三方应用. OAuth 2.0是OAuth协议的下一版本, 但不向后兼容OAuth 1.0. 

用一句话来讲:

​	Oauth解决的问题是使用*授权服务器*提供一个*访问凭据*给到*第三方应用*，让第三方应用可以在*不知道资源所有者*在*资源服务器上的账号和密码*的情况下，能获取到*资源所有者*在*资源服务器上的受保护资源*.

## 2. 应用场景

第三方应用授权登录: 当app或者网页接入一些第三方应用时,一般需要用户登录另一个合作平台, 比如QQ,微信,github的登录授权.

前后端分离应用: 前后端分离框架,前端请求后端数据,需要通过认证. 比如使用vue,react或h5开发的app.

## 3. 名词说明

1. Third-party application: 第三方应用程序, 比如打开知乎, 使用第三方QQ授权登录, 这时候知乎就是客户端.
2. HTTP service: HTTP服务提供商, 本文中简称"服务提供商", 即用于授权登录的qq.
3. client: 代表资源所有者及其授权进行受保护资源请求的应用程序. 术语“客户端”并不暗示任何特定的实现特征(例如, 应用程序是在服务器, 台式机还是其他设备上执行).
4. Resource Owner: 资源所有者,即登录用户.
5. Authorization server: 认证服务器, 即服务提供商专门用来处理认证的服务器.
6. Resource server: 资源服务器, 即服务提供商存放用户生成的资源的服务器. 

## 4. 运行过程

```bash
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
                  Figure 1: Abstract Protocol Flow
```



A.  客户端请求资源所有者授权. 该授权请求可以直接直接呈现给资源拥有者, 也可间接地通过授权服务器进行(例如弹出嵌套的第三方登录页面,跳转到授权服务器).

B.客户端收到授权许可. (用户点击授权,并提供一些能支持认证的信息).

C. 客户端通过向授权服务器进行认证并呈现用户赋予的权限来请求`access token`.

D. 授权服务器验证客户端并验证用户赋予的权限，如果有效，则颁发`access token`.

E. 客户端从资源服务器请求受保护资源，并通过呈现`access token`进行身份验证.

F. 资源服务器验证`access token`，如果有效，则为该请求提供服务.

## 5. 授权模式

- 授权码模式(authorization code)

- 简化模式(implicit)

- 密码模式(resource owner password credentials)

- 客户端模式(client credentials)

  ### 5.1 授权码模式

  

  