title: Github上SSH key 的使用与管理
tags:
  - github
  - ssh
categories:
  - github
date: 2016-01-23 19:48:00
---
SSH 为 [Secure Shell](http://baike.baidu.com/view/2118359.htm) 的缩写，是相对[FTP](http://baike.baidu.com/view/369.htm)、POP和Telnet等明文传输数据来讲较为安全的一种协议。SSH传输的数据是经过压缩处理后的，传输速度快，从客户端来看，SSH提供两种级别的安全验证，**第一种级别（基于口令的安全验证）**，**第二种级别（基于密匙的安全验证）**。Github、Gitlab及Bitbuckut等代码托管平台都支持基于密匙的SSH来进行远程代码管理，下面以Github为例具体说下ssh key的创建与使用。

#### 1，SSH key的生成

abc@163.com 为Github的登录邮箱，通过以下命令即可创建一对公私钥 （公钥文件：~/.ssh/id_rsa.pub； 私钥文件：~/.ssh/id_rsa）:

``` 
ssh-keygen -t rsa -C "abc@163.com"
```

然后会提示本地ssh key的保存路径，如果是单个创建，回车即可报错默认/Users/用户名/.ssh目录下。

``` bash
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/pandora/.ssh/id_rsa):	
```

接下来会提示是否需要帐号密码，可以为空，也可以任意指定（首次连接ssh是则会提示输入此密码）。

``` objective-c
Enter passphrase (empty for no passphrase):
```

至此，ssh key创建完毕。接下来只需将生成的ssh key保存至github即可，查看ssh key命令：

``` 
cat ~/.ssh/id_rsa.pub
```

显示结果为：

``` objective-c
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCohNI1KuNzVP7UlclbueAp/2Gxhbm0romfChDaqvF3dlMS0SS1HH1HQivG7G2J+hXwhV+V11x3LRKfyIkZy0iq6cccn4+Yan3zdWI12CfhzuHuVOQ7I2nLeDDF/CwqGrY/81r9HQpMNsPfnAHsoAT44M0QcTQORlapJYKIfz4LBT0ZXtGMnm8UeNR3t3RUL0RUZrBjgaeZIuihZjsxfpT3awOsLeTFJDld4Nv2ldw3sADQry0gT912r1IVBvpdmJ8SmQWDvjMggldhzHJoVq3ACM5jK+MSeVAUe11B3WlHDXaUIbHNyRhM+PyQ1FRgckVhz4NwJwPYSWJ5Zalm3GFl abc@163.com
```

bcopy命令将生成的公钥拷贝至剪切板：

``` 
pbcopy < ~/.ssh/id_rsa.pub
```

最后，打开github，找到设置页，在SSH keys中添加即可。

#### 2，链接测试SSH key

运行ssh -T命令即可测试ssh key是否链接成功：

``` 
ssh -T git@github.com
```

如成功，则提示：Hi user_abc! You've successfully authenticated, but GitHub does not provide shell access. user_abc就是该邮箱在Github注册的用户名。

如果测试连接不成功，可使用`ssh -vT git@github.com`命令查看详细输出，便于跟踪问题，执行结果如下：

``` bash
OpenSSH_6.9p1, LibreSSL 2.1.8
debug1: Reading configuration data /Users/pandora/.ssh/config
debug1: /Users/pandora/.ssh/config line 2: Applying options for github.com
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug1: Connecting to github.com [192.30.252.129] port 22.
debug1: Connection established.
debug1: identity file /Users/pandora/.ssh/id_rsa_dama2716588 type 1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/pandora/.ssh/id_rsa_dama2716588-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.9
debug1: Remote protocol version 2.0, remote software version libssh-0.7.0
debug1: no match: libssh-0.7.0
debug1: Authenticating to github.com:22 as 'git'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client chacha20-poly1305@openssh.com <implicit> none
debug1: kex: client->server chacha20-poly1305@openssh.com <implicit> none
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: ssh-rsa SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8
debug1: Host 'github.com' is known and matches the RSA host key.
debug1: Found key in /Users/pandora/.ssh/known_hosts:1
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/pandora/.ssh/id_rsa_dama2716588
debug1: Server accepts key: pkalg ssh-rsa blen 279
debug1: Authentication succeeded (publickey).
Authenticated to github.com ([192.30.252.129]:22).
debug1: channel 0: new [client-session]
debug1: Entering interactive session.
debug1: Sending environment.
debug1: Sending env LC_CTYPE = UTF-8
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
Hi dama2716588! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3244, received 1776 bytes, in 2.0 seconds
Bytes per second: sent 1650.3, received 903.5
debug1: Exit status 1
```

#### 3，配置管理SSH key

当本地存储使用多个ssh key时，需要通过config文件（/Users/用户名/.ssh/config）来切换默认账户，ssh config文件常用配置如下：

``` bash
# Default github user(dama2716588@126.com)  默认配置，一般可以省略
Host github.com
Hostname github.com
User git
Identityfile ~/.ssh/id_rsa_dama2716588

# 2 user(dama2716588@163.com)
Host github.com
HostName github.com
User git
Identityfile ~/.ssh/id_rsa_pandorago

# 3 user(mayulong01@baidu.com)
gitlab.com 对应配置 
Host gitlab.com
HostName gitlab.com
User mayulong01
Identityfile ~/.ssh/id_rsa_gitlab_mayulong01
```

Host： "personal.github.com"是一个"别名"，可以随意命名, 像github-PERSONAL这样的命名也可以；

HostName：比如我工作的git仓储地址是ssh://g@gitlab.baidu.com/abc.git, 那么我的HostName就要填"baidu.com"；

IdentityFile： 所使用的公钥文件;

参考链接：

https://help.github.com/articles/generating-ssh-keys/