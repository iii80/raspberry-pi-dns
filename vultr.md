### 注册 Vultr

首先，我们需要在 [Vultr](https://www.vultr.com/?ref=8038470) 注册一个账号，填上邮箱和密码，就可以创建账号了



![img0](./pic/server/0.png)

注意，你的密码如果设置比较简单，会跳转到如下页面，需要按照要求重新创建密码。

![img1](./pic/server/1.png)



接下来，需要我们进行邮箱验证，点击个人账户，发送验证邮件，接下来登录自己的邮箱查看邮件，点击验证链接就可以了![img2](./pic/server/2.png)



如果验证成功，首先需要给账户里充值，才可以创建服务器，点击账单，这里我们先充值 10 美元（Vultr 一直会有活动，新用户一次性充值 10 美元会送 50 美元）

![img3](./pic/server/3.png)



### 创建服务器

接下来我们开始创建服务器，点击当前页面的 "加号"

![img4](./pic/server/4.png)



来选择服务器的机房以及容量等信息

![img5](./pic/server/5.png)

点击 `Deploy Now` 后应该会跳到系统安装界面：

![img6](./pic/server/6.png)

点击 Manage，可以查看系统的一些信息

![img7](./pic/server/7.png)

这里的 `IP` 和 `密码` 一会要用到



### 远程连接服务器

首先，我们需要有连接服务器的软件，Windows 下推荐使用 Xshell，链接如下：

```
链接：https://pan.baidu.com/s/1FAS9bQaDC6lVzYXNiikATg 
提取码：1bvq 
```

打开 Xshell 软件界面如下，我们通过 ssh 方式连接的服务器，命令如下 `ssh root@149.38.25.212`，其中 `149.38.25.212` 是上面你服务的 `IP`

![0](./pic/ssh/0.png)

输入回车后，会出现如下界面，点击接受并保存：

![1](./pic/ssh/1.png)

接下来会让我们输入密码，我们输入上面的服务器的密码并确定：

![2](./pic/ssh/2.png)

出现如下界面，说明我们已经成功连接到服务器：

![3](./pic/ssh/3.png)



### 安装 V2ray

V2ray 是一个小巧灵活的代理软件，鉴于 ShadowSocks 已经越来越容易被封，所以我们用 V2ray 实现代理功能。首先执行 `bash <(curl -s -L https://git.io/v2ray.sh)` 来安装

![0](./pic/v2ray/0.png)

回车后选择 1，安装：

![1](./pic/v2ray/1.png)

通信协议选择 3，WebSocket。注意，你的选项可能和我不一样，要选择 WebSocket 对应的数字：

![2](./pic/v2ray/2.png)

下面的一直按回车就行：

![3](./pic/v2ray/3.png)

注意你的 v2ray 端口可能和我不一样

![4](./pic/v2ray/4.png)

最后如果出现如下界面，说明安装完成，要保存下面信息，后面配置 V2ray 客户端时候要用

![5](./pic/v2ray/5.png)