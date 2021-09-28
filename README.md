# 深度学习环境配置
## 1. 制作启动盘

1. ubuntu镜像下载

https://ubuntu.com/download/desktop

2. 软件制作

## 安装nvidia驱动

1. 查看显卡型号

```
lspci | grep -i nvidia
```

2. 进入nvidia官网下载驱动

https://www.nvidia.cn/geforce/drivers/

3. 卸载ubuntu自带驱动

```
sudo apt purge nvidia*
```

4. 禁用自带的nouveau nvidia驱动

```
sudo vim /etc/modprobe.d/blacklist.conf
```

在文件最后添加：

```
blacklist nouveau  
options nouveau modeset=0 
```

5. 更新

```
sudo update-initramfs -u
```

6. 重启电脑

```
sudo reboot
```

7. 重启后查看是否已经将自带的驱动屏蔽了，输入以下代码， 没有输出则代表屏蔽成功了

```
lsmod | grep nouveau
```

8. 安装gcc和make

```
sudo apt install gcc 
sudo apt install make
# 或者直接执行
sudo apt install gcc & make # 同时安装gcc和make，不用一条一条执行了，效果和上面两条命令相同
```

9. 开始安装

```
sudo chmod a+x NVIDIA-Linux-x86_64-450.80.02.run
sudo ./NVIDIA-Linux-x86_64-450.80.02.run -no-x-check -no-nouveau-check -no-opengl-files
# -no-x-check:安装时关闭X服务
# -no-nouveau-check: 安装时禁用nouveau
# -no-opengl-files:只安装驱动文件，不安装OpenGL文件
```

![image.png](http://www.mlzhilu.com/upload/2020/11/image-aab9f3d76fbe4c7293696e3dcb3c1b2b.png)

![image.png](http://www.mlzhilu.com/upload/2020/11/image-3629793f1cfa4ce096618e978b908577.png)

安装完成以后执行

```
nvidia-smi
```

# frp内网穿透

## 1. 客户端

a. 启动服务

```
vim frps/frpc.int
```

```vim
[common]
server_addr = 101.132.40.112 #服务器地址
server_port = 7001 # 服务器开放的端口
login_fail_exit = false # 加上这一句用于开机自启动后失败重新启动

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000 # 服务器通过端口号找到内网机器
```

b. 开机自启动

```
sudo vim /lib/systemd/system/frpc.service
```

填入一下内容：

```
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



