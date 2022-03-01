# frp内网穿透

作用：让内网机器能够通过外网访问，实现pycharm等IDE的远程调试功能。

需要准备：

- 一个轻量服务器，可以上阿里云或腾讯云申请

- 自己的电脑一台

- 内网机器一台

## 1. 申请服务器

[阿里云服务器](https://developer.aliyun.com/plan/grow-up)

[腾讯云服务器](https://console.cloud.tencent.com/lighthouse/instance/index)

申请完服务器以后，要给服务器开放一个端口用于服务器和内网机器建立连接，这里我开放的是7000端口，一般是在防火墙规则那里添加，不同的服务器不一样。

![](/Users/zhouhan/githubProject/deep-learning-environment-config/images/2022-03-01-18-30-59-image.png)

## 2. ssh连接服务器

可以用ssh软件链接申请好的服务器

## 3. 在服务器上配置frp

a. 下载frp

```bash
wget https://github.com/fatedier/frp/releases/download/v0.39.1/frp_0.39.1_linux_amd64.tar.gz
tar -zxvf frp_0.39.1_linux_amd64.tar.gz
```

b. 编辑配置文件

```bash
vim frps.ini
```

```bash
[common]
bind_port = 7000 # 前面服务器开放的端口
```

c. 运行frp

```bash
./frps -c ./frps.ini
```

d. 开启后台运行

```bash
nohup ./frps -c ./frps.ini &
```

## 4. 在内网机器（客户端）上配置frp

a. 启动服务

```bash
vim frps/frpc.ini
```

```bash
[common]
server_addr = 101.132.40.112 #服务器地址
server_port = 7000 # 服务器开放的端口
login_fail_exit = false # 加上这一句用于开机自启动后失败重新启动

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000 # 服务器通过端口号找到内网机器
```

b. 开机自启动

```bash
sudo vim /lib/systemd/system/frpc.service
```

填入以下内容：

```bash
[Unit]
Description=frpc
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/home/zhouhan/frps/frpc -c /home/zhouhan/frps/frpc.ini

[Install]
WantedBy=multi-user.target
```

c. 使用systemctl实现开机自启动

```bash
sudo systemctl enable frpc
sudo systemctl start frpc
```

e. 查看服务是否启动

```bash
sudo systemctl status frpc
```
