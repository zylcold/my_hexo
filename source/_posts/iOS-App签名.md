---
layout: title
title: iOS App签名
date: 2019-05-27 01:05:18
tags:
---

众所周知iOS是一个封闭的平台，大多数iPhone的用户App获取的渠道只有AppStore。但是对于开发人员，Apple又不得不提供绕过AppStore来安装App的方法。

如何控制？其中Apple用到了数字签名[^数字签名]技术。

Apple会生成一对密钥，公钥A放在每台iOS设备里面，私钥A放在Apple后台。
对于开发者，Apple会提供两样东西Provisioning Profile(PP文件) 和 Certificate(签名证书)。

[^数字签名]: [数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

## PP文件

用来限制iOS设备上非AppStore下载的App的安装，被Apple私钥A签名。

### 流程

1. App安装前，iOS设备会首先使用公钥A验证PP文件是否合法
2. 验证AppID是否匹配
3. 通过允许设备的UUID列表来判断此设备是否可以安装
4. 验证Entitlements和App内的Entitlements文件是否匹配
5. 取出Certificates中的公钥列表，验证是否存在匹配
6. 验证通过，安装成功

<!-- more -->

### Xcode配置

Build Settings中**PROVISIONING_PROFILE_SPECIFIER**指定PP名或者UUID，一些特定的PP文件(企业发布)，还需要设置**DEVELOPMENT_TEAM**

### 存放位置

~/Library/MobileDevice/Provisioning Profile/下，以uuid为文件名，mobileprovision作为后缀名。

### 组成

1. App基本信息：通常包括AppName、 uuid、BundleIdentifier、Platform、CreationDate、ExpirationDate
2. 权限开关(Entitlements)：IAP权限、Keychain Sharing权限、Apple Pay等等
3. 签名证书列表(Certificates)：可用于签名的证书列表
4. 允许设备UUID列表(Provisioned Devices)：允许安装的设备
    
## 签名证书

Apple为开发者公钥签名的证书。同样被Apple私钥签名。

### 流程

1. 开发者本地生成一对密钥B
2. 公钥B上传给Apple，Apple使用私钥为其签名，生成证书
3. 应用打包时，开发者使用私钥B对其签名
4. 安装时，iOS设备使用公钥B对其验证

### Xcode配置

Build Settings中**CODE_SIGN_IDENTITY**指定证书名称

### 存放位置

位于KeyChain Access中的Certificates

### 组成

1. 基本信息：Code Signing Identity(证书标示)，签发者，过期时间
2. 主体信息：User ID、Common Name、Organizational Unit
3. 签发信息：Organization
4. 公钥信息：公钥内容
5. 证书摘要：sha-1
6. 私钥

## IPA文件的生成

包含Archive 和 Distribute两步。
### Archive

源代码的编译，链接，私钥签名。

### Distribute

将签名后的可执行文件，资源文件，PP文件，Entitlements打包成IPA文件

## 一些问题

### 证书的sha-1

由于证书可能出现重名问题，有一些时候需要指定证书的sha-1

### DEVELOPMENT_TEAM

证书中的Organizational Unit



## 引用
[iOS App 签名的原理](http://blog.cnbang.net/tech/3386/)



