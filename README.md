

# 树莓派做 DNS 服务器以及网关，实现自由上 Google

首先声明，本教程的实验环境为树莓派，理论上只要是 Linux 系统就可以，可能细节步骤有所差别。教程仅限于交流学习，禁止用于非法用途。



## 服务器配置

首先，我们想要上 Google，需要有一个可以上 Google 的服务器，我们就可以通过服务器中转来访问 Google YouTube 等网站的请求。



如果你已经有服务器了并配置好 v2ray 软件，可以忽略下面的链接。如果你还没有一个能访问 Google 的服务器，这里器推荐 Vultr，本人测试访问 Google YouTube 等国外主流网站的速度很快，价格也比较合理。

[Vultr 注册以及创建服务器安装配置 V2ray](./vultr.md)



## 树莓派配置



### 安装系统

树莓派我们选择了比较轻量稳定的 [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/) 系统，具体安装教程如下

[树莓派安装 Raspbian Stretch Lite 系统](./raspberry_pi_install.md)



### 环境配置

树莓派系统中默认是不支持 ssh 的，中文显示也乱码，我们需要配置一下。首先，将树莓派连接屏幕开机并联网（建议直接插网线，因为后面做 DNS 服务器，长期开机，网线比较稳定）。默认的用户名是 `pi`，默认密码是 `raspberry`（注意输入密码的时候屏幕是没有显示的，输完直接按回车就可以）



因为默认的密码不太安全，所以我们首先修改下默认密码，输入 `sudo passwd pi` 会提示你输入新密码，输入后会提示你再次输入，然后就会设置成功（后面提示要密码的地方输入你新设置的密码就可以了）

![10](./pic/raspberrypi/10.png)

#### 开启 ssh

使用命令：`sudo raspi-config`，下方向键选择 `Interfacing Options`

![1](./pic/raspberrypi/1.png)

选择 SSH![2](./pic/raspberrypi/2.png)

开启 SSH

![3](./pic/raspberrypi/3.png)

按 tab 键盘选择 Finsh，完成设置

![4](./pic/raspberrypi/4.png)



##### 支持中文显示

树莓派默认不支持中文，需要我们配置一下编码，执行 `sudo raspi-config`，选择 Localisation Options

![5](./pic/raspberrypi/5.png)

选择 Change Locale

![6](./pic/raspberrypi/6.png)

上下移动到 `en_US.UTF-8 UTF-8` , `zh_CN.GBK GBK`, `zh_CN.UTF-8 UTF-8` 按空格勾选（有 `*` 号代表选中），然后按 Tab 键选择 OK

![7](./pic/raspberrypi/7.png)

![8](./pic/raspberrypi/8.png)



默认系统语言我们选择 `en_US.UTF-8`

![9](./pic/raspberrypi/9.png)

按 tab 键盘选择 Finsh，完成设置

![4](./pic/raspberrypi/4.png)



安装中文库，在命令行输入：

```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install ttf-wqy-zenhei
```

现在我们的树莓派环境已经配置好了，输入 `sudo reboot`重启生效



