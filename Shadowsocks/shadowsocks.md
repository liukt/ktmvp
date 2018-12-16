## 如何用shadowsocks-go搭建自己的VPN服务器
### - [****新手基本原理****](https://laucyun.com/5cce9d01b0a0210482d65f5bc040d83b.html)

![原理如下图](what-is-shadowsocks.png)
---
## Practice

### Install new OS
![install](pic01.png)

点击Reload后
弹出的页面会显示 **远程登陆端口**和**root用户密码**并且**发送到指定邮箱**

### 通过xshell远程登陆 my VPS
![Properties](xshell-Properties.png)

Host: [IP地址 or 域名(域名还是会解析成IP地址)]

Prot Number: 远程登陆端口号

##### 点击 OK 远程连接到服务器的指定端口后
输入用户名和密码登陆(也就是邮箱收到的user name :root 及root的密码 )

![user name](xshell-root.png)
![PassWd](xshell-Authentication.png)


- 更改密码
- root@debian:~# passwd
