

# 树莓派做 DNS 服务器以及网关，实现自由上 Google

首先声明，本教程的实验环境为树莓派，理论上只要是 Linux 系统就可以，可能细节步骤有所差别。教程仅限于交流学习，禁止用于非法用途。



## 服务器配置

首先，我们想要上 Google，需要有一个可以上 Google 的服务器，我们就可以通过服务器中转来访问 Google YouTube 等网站的请求。



如果你已经有服务器了并配置好 v2ray 软件，可以忽略下面的链接。如果你还没有一个能访问 Google 的服务器，这里器推荐 Vultr，本人测试访问 Google YouTube 等国外主流网站的速度很快，价格也比较合理。

[Vultr 注册以及创建服务器安装配置 V2ray](./vultr.md)



## 树莓派配置



### 安装系统

树莓派系统我们选择了比较轻量稳定的 [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/) 系统，具体安装教程如下

[树莓派安装 Raspbian Stretch Lite 系统](./raspberry_pi_install.md)



### 环境配置

树莓派系统中默认是不支持 SSH 的，中文显示也乱码，我们需要配置一下。首先，将树莓派连接屏幕开机并联网（建议直接插网线，因为后面做 DNS 服务器需要长期开机，网线比较稳定）。默认的用户名是 `pi`，默认密码是 `raspberry`（注意输入密码的时候屏幕是没有显示的，输完直接按回车就可以）



因为默认的密码不太安全，所以我们首先修改下默认密码，输入 `sudo passwd pi` 会提示你输入新密码，输入后会提示你再次输入，然后就会设置成功（后面提示要密码的地方输入你新设置的密码就可以了）

![10](./pic/raspberrypi/10.png)

#### 开启 ssh

使用命令：`sudo raspi-config`，上下方向键选择 `Interfacing Options`

![1](./pic/raspberrypi/1.png)

选择 SSH![2](./pic/raspberrypi/2.png)

开启 SSH

![3](./pic/raspberrypi/3.png)

按 tab 键盘选择 Finish，完成设置

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

按 tab 键盘选择 Finish，完成设置

![4](./pic/raspberrypi/4.png)



安装中文库，在命令行输入：

```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install ttf-wqy-zenhei
```

现在我们的树莓派环境已经配置好了，输入 `sudo reboot`重启生效



### 安装依赖

```
sudo apt-get install ssh curl git vim dnsmasq tmux ipset ttf-wqy-zenhei dnsutils lsof proxychains -y
```

接下来安装 V2ray 客户端，执行

```
wget https://install.direct/go.sh && sudo bash go.sh
```



### 配置 dnsmasq

我们使用 dnsmasq 来做 DNS 解析服务，dnsmasq 适用于个人用户或少于50台主机的网络，对于个人或者家庭使用够了

首先备份一下旧的配置文件

```
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
```

编辑 `/etc/dnsmasq.conf` 填充为以下内容：

```
no-resolv # 不使用 /etc/resolv.conf 做 dns 解析
cache-size=5000
server=127.0.0.1  
server=114.114.114.114 # 国内默认使用 114.114.114.114 来进行 dns 解析
conf-dir=/etc/dnsmasq.d
```

>  注：编辑文件方法请自行查阅文档，例如 vim



### 配置 ipset 

我们使用 ipset 将需不能直接访问的网址的 `IP` 加到集合里，通过服务器访问。

```
git clone https://github.com/cokebar/gfwlist2dnsmasq.git
cd gfwlist2dnsmasq/
bash gfwlist2dnsmasq.sh -o gfwlist.conf -s gfwlist -p 5353 # -o 为输出文件， -s 为设置 ipset 集合名，-p 为 dns 端口
sudo cp gfwlist.conf /etc/dnsmasq.d/
```

我们来看一下生成的文件：  

```
server=/030buy.com/127.0.0.1#5353
ipset=/030buy.com/gfwlist
server=/0rz.tw/127.0.0.1#5353
ipset=/0rz.tw/gfwlist
server=/1000giri.net/127.0.0.1#5353
ipset=/1000giri.net/gfwlist
...
```

- `server=..../127.0.0.1#5353`  意思是碰到这个域名，交给本机 5353 端口来进行 DNS 解析处理
- `ipset=/.../gfwlist` 表示将这个域名解析出来的 ip 加到名字为 gfwlist 的 ipset 集合中 



这样，我们很容易想到透明代理方案： 

- 首先监听 5353 端口来将这些域名解析成真正的 ip。然后将解析出来的 ip 加到名字为 gfwlist 的集合中。
- 假设我们 V2ray 代理程序监听 2000 端口，作用是把各种类型的请求发给服务器，让服务器完成真正的请求。那么我们可以写一条规则，如果访问的 ip 在 gfwlist 中，那么直接将该请求转发到 2000 端口。



上面我们已经为常用的国外网站配置了 ipset，那么我们重启一下 dnsmasq 服务使其生效

```
sudo systemctl enable dnsmasq.service
sudo systemctl restart dnsmasq.service
sudo systemctl status dnsmasq.service # 如果状态正常，再继续进行下面步骤(按 q 退出)
```

![11](./pic/raspberrypi/11.png)



### V2ray 客户端配置



将 [config.json](./conf/config.json) 文件拷贝到 `/etc/v2ray/config.json`，要注意需要修改配置文件中的服务器 ip，id 以及 port 参数，主要参数如下：

![12](./pic/raspberrypi/12.png)