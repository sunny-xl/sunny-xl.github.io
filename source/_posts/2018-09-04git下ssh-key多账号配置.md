---
title: git下ssh-key多账号配置
date: 2018-09-04 16:59:13
---

公司项目从svn中迁移到了gitlab，而自己的一些项目放在github上。这样就导致要配置不同的ssh-key对应不同的环境。废话不多说，直接上步骤：

### 1、生成gitlab上账号对应的ssh-key
```
ssh-keygen -t rsa -C "yourmail@xxx.com" -b 4096 -f ~/.ssh/gitlab_id_rsa
```
> 在~/.ssh/目录下会生成gitlab_id_rsa和gitlab_id_rsa.pub私钥和公钥。我们将gitlab_id_rsa.pub中的内容粘帖到公司gitlab服务器的SSH-key的配置中。


### 2、生成github上账号对应的ssh-key
```
ssh-keygen -t rsa -C "yourmail@xxx.com" -b 4096 -f ~/.ssh/github_id_rsa
```
> 在~/.ssh/目录会生成github_id_rsa和github_id_rsa.pub私钥和公钥。我们将github_id_rsa.pub中的内容粘帖到公司github服务器的SSH-key的配置中。

<!--more-->

### 3、添加私钥
```
ssh-add ~/.ssh/gitlab_id_rsa $ ssh-add ~/.ssh/github_id_rsa
```

>出现 `Identity added: /Users/uhope/.ssh/github_id_rsa (/Users/uhope/.ssh/github_id_rsa)`就表示成功了

```
# 可以通过 ssh-add -l 来确认私钥列表，看是否2个私钥添加成功
ssh-add -l
# 可以通过 ssh-add -D 来清空私钥列表
ssh-add -D
```
### 4、修改配置文件
在~/.ssh/目录下新建一个config文件
```
touch config
```
在config中添加如下内容：
```
# gitlab
Host gitlab.com
   HostName gitlab.com
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/gitlab_id_rsa
# github
Host github.com
   HostName github.com
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/github_id_rsa

# 配置文件参数
# Host : 自定义，方便记录是那个网站的ssh-key
# HostName : 对应网站的域名，如github.com，我们公司自己配置的gitlab内部地址：10.0.0.10
# IdentityFile : 指明上面User对应的identityFile路径
```

### 5、目录结构
```
config			github_id_rsa.pub	gitlab_id_rsa.pub
github_id_rsa		gitlab_id_rsa
```

### 6、测试是否配置成功
```
ssh -T git@github.com
```
输出：`The authenticity of host 'github.com (192.30.255.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? `
直接输入 yes, 然后输出：`Hi sunny110! You've successfully authenticated, but GitHub does not provide shell access.` 就表示成功的连上github了。

同理，测试gitlab, ssh -T git@10.0.0.10,输入“yes”,最终输出“Welcome to GitLab, xxx!”表示配置成功

### 7、最终的目录结构
```
config			github_id_rsa.pub	gitlab_id_rsa.pub
github_id_rsa		gitlab_id_rsa		known_hosts
```
至此，就可以尽情享受使用git的快感了！
