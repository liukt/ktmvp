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

### 登陆成功
![Debian](xshell-debian.png)

---
- 更改密码
```
root@debian:~# passwd
```
- update更新


```
root@debian:~# apt-get update
```
- update  Kernel and pre-Loading TCP BBR
- 更新内核并预加载TCP BBR算法模块,完成后重启。


```
root@debian:~# wget --no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && chmod a+x BBR.sh && bash BBR.sh -f
Download New Kernel
	linux-image-4.12.9-041209-generic_4.12.9-041209.201708242344_i386.deb
Install New Kernel
	linux-image-4.12.9-041209-generic_4.12.9-041209.201708242344_i386.deb
Uninstall Old Kernel
	linux-image-3.16.0-4-686-pae
Uninstall Old Kernel
	linux-image-686-pae
pre-Loading TCP BBR ...
Finish! 
It will reboot now...
```
###### reboot重启后连接会断开，需要重新连接
