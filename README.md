# 深度学习环境配置

## 安装系统

## 1. 制作启动盘

1. ubuntu镜像下载

https://ubuntu.com/download/desktop

2. 软件制作

使用软件制作启动盘

## 2.安装nvidia驱动

1. 查看显卡型号

```bash
lspci | grep -i nvidia
```

2. 进入nvidia官网下载驱动

https://www.nvidia.cn/geforce/drivers/

3. 卸载ubuntu自带驱动

```bash
sudo apt purge nvidia*
```

4. 禁用自带的nouveau nvidia驱动

```bash
sudo vim /etc/modprobe.d/blacklist.conf
```

在文件最后添加：

```bash
blacklist nouveau  
options nouveau modeset=0 
```

5. 更新

```bash
sudo update-initramfs -u
```

6. 重启电脑

```
sudo reboot
```

7. 重启后查看是否已经将自带的驱动屏蔽了，输入以下代码， 没有输出则代表屏蔽成功了

```bash
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

```bash
vim frps/frpc.int
```

```bash
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

d.查看服务是否启动

```bash
sudo systemctl status frpc
```



# Conda

## conda换源

```bash
vim ~/.condarc
#清华源
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
ssl_verify: true
#中科大源
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
ssl_verify: true
#上交大源
channels:
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/conda-forge/
ssl_verify: true
```



# pip

## pip换源

- 安装

  ```
  pip install pqi
  ```

- 列举所有支持的pip源

  ```
  pqi ls
  ```

- 换源

  ```
  pqi use <name>
  ```

- 添加新的pip源

  ```
  pqi add ustc https://mirrors.ustc.edu.cn/pypi/web/simple
  ```

- 移除pip源

  ```
  pqi remove pypi
  ```

  

## tensorflow 部署

### 1. 安装英伟达驱动

```
wget https://www.nvidia.cn/geforce/drivers/
```

### 2. 查看tensorflow需要的cuda版本

```
https://www.tensorflow.org/install/source#gpu
```

```
sh 安装包
```

```
export PATH=/usr/local/cuda-11.1/bin:$PATH  
export LD_LIBRARY_PATH=/usr/local/cuda11.1/lib64:$LD_LIBRARY_PATH
```

```
source ~/.bashrc
```

### 3. 安装cuda（cudatoolkit）

nvidia-smi 和 nvcc的区别：nvidia-smi是管理和监控gpu的工具，nvcc是cuda的编译器。nvidia-smi 显示的cuda版本：driver-api对应的cuda版本，nvcc -v 显示的是runtime-api。nvcc是与CUDA Toolkit一起安装的CUDA compiler-driver tool，它只知道它自身构建时的CUDA runtime版本，并不知道安装了什么版本的GPU driver，甚至不知道是否安装了GPU driver。

结论：通常，**driver api的版本能向下兼容runtime api的版本**，即 **nvidia-smi 显示的版本大于nvcc --version 的版本通常不会出现大问题。**

所以，保证nvidia-smi的版本号 >= nvcc -V的版本号即可

#### 3.1 查看能够安装的cuda最高版本

```
nvidia-smi
```

#### 3.2 安装cudatoolkit

```
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda_11.2.0_460.27.04_linux.run
sudo sh cuda_11.2.0_460.27.04_linux.run
```

### 4. 下载对应的cudnn版本

```
https://developer.nvidia.com/rdp/cudnn-archive
```

解压，拷贝到对应路径（之前设置过环境变量的路径）

```
tar -xzvf 文件名
sudo cp cuda/include/cudnn.h /usr/local/cuda/include 
sudo cp cuda/lib64/* /usr/local/cuda/lib64
```

```
nvcc -V 
```

### 5. pip 或者 conda安装tensorflow

