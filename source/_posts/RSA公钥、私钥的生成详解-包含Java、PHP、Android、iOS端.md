---
title: 'RSA公钥、私钥的生成详解,包含Java、PHP、Android、iOS端'
date: 2018-09-04 16:58:22
---

说明：
> Java和PHP为服务端，Android和iOS为客户端。
>Java和Android所用的公钥、私钥是同样的格式，私钥需要PKCS8格式，默认生成的私钥是PKCS1格式的
php私钥需要PKCS1格式的
iOS私钥需要.p12的文件格式，公钥需要.der格式的

>公钥作用：RSA加密 、验签
私钥作用：RSA解密、加签

以下为终端操作命令的详细步骤：

## 一、生成私钥文件
```
 openssl genrsa -out rsa_private_key.pem 2048
```

>  openssl:是一个自由的软件组织,专注做加密和解密的框架。
>  genrsa:指定了生成了算法使用RSA
>  -out:后面的参数表示生成的私钥key的文件名字
> 2048:表示的是生成key的长度,单位字节(bits)

此命令后会生成一个名字为rsa_private_key.pem、2048位、PKCS1格式的RSA私钥

<!--more-->

## 二、生成公钥文件

```
 openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```
此命令会从私钥中提取出来生成一个名字为rsa_public_key.pem的RSA公钥

### PHP
所需公钥、私钥已经生成，即rsa_public_key.pem、rsa_private_key.pem

### Java和Android
需要把私钥的格式从默认的PKCS1转换为PKCS8格式
```
openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt -out pkcs8_private_key.pem
```
此命令会会生成名字为pkcs8_private_key.pem的RSA私钥文件

Java和Android所需公钥、私钥已经生成，即rsa_public_key.pem、pkcs8_private_key.pem

可以到在线RSA验证网站上，验证公钥私钥是否成对。[在线RAS生成、转换工具](http://tool.chacuo.net/cryptrsakeyvalid)

### iOS
由rsa_private_key.pem生成csr -> 生成crt -> 生成der -> 生成p12

1、 创建证书请求
```
openssl req -new -key rsa_private_key.pem -out rsacert.csr
```
>拿着RSA私钥文件去数字证书颁发机构（即CA）申请一个数字证书。CA会给你一个新的文件cacert.pem,那才是你的数字证书。

>输入命令后会提示让输入国家、省份、地区、邮箱等信息，按要求填写或者不填都可以，最终会生成rsacert.csr 文件

2、生成证书并签名，设置有效期10年
```
openssl x509 -req -days 3650 -in rsacert.csr -signkey rsa_private_key.pem -out rsacert.crt
```
> 509是一种非常通用的证书格式

 将用上面生成的密钥rsa_private_key.pem和rsacert.csr证书请求文件生成一个数字证书rsacert.crt,这个就是公钥

3、转换格式：将pem格式文件转换成der格式  (公钥)
```
openssl x509 -outform der -in rsacert.crt -out public_key.der
```
> 在iOS开发中,公钥是不能使用base64编码的,上面的命令是将公钥的base64编码字符串转换成二进制数据

 此时生成的public_key.der就是iOS所需的RSA公钥

4、 导出 P12 文件
```
openssl pkcs12 -export -out private_key.p12 -inkey rsa_private_key.pem -in rsacert.crt

```
> 在iOS使用私钥不能直接使用，需要导出一个p12文件。此命令就是将私钥文件导出为p12文件

> 此时会提示让输入密码，可输入iOS私钥文件的密码，但是请注意：在项目中读取私钥文件时需要此时设置的密码。并且此密码是生成的p12私钥文件的密码，并不是rsa_private_key.pem原始私钥文件的密码

至此，iOS所需公钥、私钥生成完成，分别是public_key.der、private_key.p12

[iOS上RSA使用的demo](https://github.com/sunny110/RSADemo)
