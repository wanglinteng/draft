# 应用场景
实验室有一台Taitan xp * 4深度学习服务器，为了最大化利用资源，每个人都能使用而且互不干扰，所以需要一种服务器共享，更确切的说是GPU资源共享服务。

考虑过Docker，LXD两种方案，Docker是一种应用级容器，LXD是一种系统级容器，由于LXD更贴合我们的应用场景，所以我们采用了LXD技术方案。

# 应用效果
每个人分配主机的一个端口、帐号、密码，通过主机-容器端口映射，可以像操作一台独立的ubuntu一样，操作容器，同时所有人共享4块GPU。

例如：主机地址 201.23.56.89 端口号 2011 帐号 ubuntu 密码 000000
通过主机的2011端口映射至容器的22端口，通过ssh远程即可访问容器，普通用户无主机操作权限。

# 安装ubuntu16.04
制作光盘镜像或者USB镜像，修改BIOS启动项进行安装，安装时仅设置`/分区`和`swap分区`即可。

# 换源
备份源文件 `sudo cp /etc/apt/sources.list /etc/apt/sources_init.list`

由于机房无法连接校外IPV4地址，故采用中科大源，通过IPV6通信
```shell
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse

deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
```

# 卸载预置软件
```shell
sudo apt-get remove libreoffice-common
sudo rm -f /usr/share/applications/com.canonical.launcher.amazon.desktop
sudo rm -f /usr/share/applications/ubuntu-amazon-default.desktop
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan
gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca
webbrowser-app gnome-sudoku landscape-client-ui-install transmission-common
```

# 安装Nvidia驱动
下载地址 `https://www.nvidia.cn/Download/index.aspx?lang=cn`, 这里安装的是418.43版本。
为了避免循环登陆问题，安装步骤如下：
```shell
sudo apt-get remove --purge nvidia* # 卸载可能存在的nvidia驱动
sudo vim /etc/modprobe.d/blacklist-nouveau.conf # 禁用nouveau

# 打开文件后，在尾部添加
blacklist nouveau
options nouveau modeset=0

# Ctrl + Alt + F1 进入文本模式
sudo service lightdm stop
sudo chmod a+x ./xxxx.sh
sudo ./xxxx.sh
```
安装成功后在`Terminal`中输入`nvidia-smi` ,如果安装成功将会输出
```shell
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.43       Driver Version: 418.43       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  TITAN Xp            Off  | 00000000:3B:00.0 Off |                  N/A |
| 18%   19C    P0    58W / 250W |      0MiB / 12196MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  TITAN Xp            Off  | 00000000:5E:00.0 Off |                  N/A |
| 17%   18C    P0    58W / 250W |      0MiB / 12196MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  TITAN Xp            Off  | 00000000:B1:00.0 Off |                  N/A |
| 17%   23C    P0    58W / 250W |      0MiB / 12196MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  TITAN Xp            Off  | 00000000:D9:00.0 Off |                  N/A |
| 15%   19C    P0    59W / 250W |      0MiB / 12196MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```

下载cuda，地址为`https://developer.nvidia.com/cuda-downloads?target_os=Linux`,这里选择的是CUDA10.1 ，下载时选用`.run`文件，安装时不安装驱动。
```shell
sudo chmod a+x ./xxxx.run
sudo ./xxxx.run
```
安装成功后输入`nvcc -V`会输出
```shell
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Fri_Feb__8_19:08:17_PST_2019
Cuda compilation tools, release 10.1, V10.1.105
```
最后启动图形界面 `sudo service lightdm start`

# 安装CUDNN
CUDNN下载地址是`https://developer.nvidia.com/cudnn`，nvidia会要求用户先注册，登陆同意协议以后，选择的是`Download cuDNN v7.5.0 (Feb 25, 2019), for CUDA 10.1`，官方给了一份安装说明`https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html`

```shell
1.Install the runtime library, for example:
sudo dpkg -i libcudnn7_7.0.3.11-1+cuda9.0_amd64.deb

2.Install the developer library, for example:
sudo dpkg -i libcudnn7-devel_7.0.3.11-1+cuda9.0_amd64.deb

3.Install the code samples and the cuDNN Library User Guide, for example:
sudo dpkg -i libcudnn7-doc_7.0.3.11-1+cuda9.0_amd64.deb
```
# 安装 NVIDIA Container Runtime
官方的github有安装教程，NVIDIA Container Runtime是LXD能够利用GPU资源很重要的一部分。
github地址：`https://github.com/NVIDIA/nvidia-container-runtime/`
```shell
# Setup a rootfs based on Ubuntu 16.04
cd $(mktemp -d) && mkdir rootfs
curl -sS http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04-core-amd64.tar.gz | tar --exclude 'dev/*' -C rootfs -xz

# Create an OCI runtime spec
nvidia-container-runtime spec
sed -i 's;"sh";"nvidia-smi";' config.json
sed -i 's;\("TERM=xterm"\);\1, "NVIDIA_VISIBLE_DEVICES=0";' config.json

# Run the container
# sudo nvidia-container-runtime run nvidia_smi
sudo apt install libnvidia-container-dev libnvidia-container-tools nvidia-container-runtime
```
安装成功后在`Terminal`中输入`nvidia-container-cli info`会输出
```shell
NVRM version:   418.43
CUDA version:   10.1

Device Index:   0
Device Minor:   0
Model:          TITAN Xp
Brand:          GeForce
GPU UUID:       GPU-0914e0f4-0488-3310-b6ff-49cd7a08e0ae
Bus Location:   00000000:3b:00.0
Architecture:   6.1

Device Index:   1
Device Minor:   1
Model:          TITAN Xp
Brand:          GeForce
GPU UUID:       GPU-c70d7632-0dd2-4b2b-d56e-b007f4810bc4
Bus Location:   00000000:5e:00.0
Architecture:   6.1

Device Index:   2
Device Minor:   2
Model:          TITAN Xp
Brand:          GeForce
GPU UUID:       GPU-235bbc2a-ab85-c586-308e-ae34199a63d5
Bus Location:   00000000:b1:00.0
Architecture:   6.1

Device Index:   3
Device Minor:   3
Model:          TITAN Xp
Brand:          GeForce
GPU UUID:       GPU-fde9d7e7-2455-5f86-52f5-06085a82c1a0
Bus Location:   00000000:d9:00.0
Architecture:   6.1
```

# 安装 LXD
因为`snap`中LXD为最新版，旧版可能不兼容最新的`nvidia`驱动，所以首先安装`snap`
```shell
sudo apt install snap
sudo snap install lxd # 这里安装的是目前最新的3.11版本
```
# LXD基础配置
```shell
sudo lxd init # 一路回车
sudo lxd init --auto --network-address=0.0.0.0
sudo lxd init --auto --storage-backend=dir
sudo lxc profile set default nvidia.runtime true # 配置LXD支持GPU操作
```

# 配置模板镜像

查询远程模板镜像`sudo lxc remote list`

镜像下载到本地`sudo lxc image copy images:ubuntu/16.04 local: --alias ubuntu/16.04 --copy-alias --public`

创建容器`sudo lxc init ubuntu/16.04 dl-1`

发布镜像 `sudo lxc publish dl-1 --alias dl-template --public`

容器启动 `sudo lxc start dl-1`

容器停止 `sudo lxc stop dl-1`

容器重启 `sudo lxc restart dl-1`

# 端口映射
dl-1:容器名称
8985：主机端口
```shell
lxc config device add dl-1 sshproxy proxy listen=tcp:0.0.0.0:8985 connect=tcp:localhost:22
```
配置完毕后，即可通过主机端口映射直接登陆dl-1容器`ssh ubuntu@主机地址 -p 8985`，ubuntu为容器内ubuntu16.04系统用户名。
# 分配容器脚本

# 开机自启动脚本
首先修复`nvidia-container-cli info` 重启报错问题，然后启动容器
```shell
#!/bin/bash
### BEGIN INIT INFO
# Provides:          land.sh
# Required-start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the svnd.sh daemon
# Description:       starts svnd.sh using start-stop-daemon
### END INIT INFO

# change root
echo "123456"|sudo -S

# output log
time=$(date "+%Y-%m-%d %T")
echo "#@#@" ${time} "@#@#" >>  /opt/gpus-start-log.txt

# fix nvidia-container-cli info restart problem
nvidia-container-cli -k -d /dev/tty list >> /opt/gpus-start-log.txt

# start containers
containers="dl-1 dl-2 dl-3 dl-4"
for container in ${containers[@]}
do
    echo "##@@" ${container} "@@##" >> /opt/gpus-start-log.txt
    lxc restart ${container} >> /opt/gpus-start-log.txt
done

# end signal
echo "#@#@END@#@#" >> /opt/gpus-start-log.txt

```
移动到自启动目录`mv ./gpus-server-init.sh /etc/init.d/`

修改权限`sudo chmod a+x ./gpus-server-init.sh `

添加启动项 `sudo update-rc.d gpus-server-init.sh defaults 95`

# 遇到问题
1.服务器重启后，`nvidia-container-cli info`报错，查了很多资料，最后找到`https://github.com/NVIDIA/libnvidia-container/issues/3`

输入`sudo nvidia-container-cli -k -d /dev/tty list`即可解决。

2.多次安装LXD,在`sudo init lxd`时`zpool`已经存在。

修改名称或者安装ZFS删除`default pool`

```shell
sudo apt install zfsutils-linux # 安装ZFS
sudo zpool list
sudo zpool destroy default # 销毁前要停止lxc容器服务
```

# 参考连接
> https://blog.csdn.net/dangruchujian/article/details/79338760

https://blog.csdn.net/weixin_42749767/article/details/83720831

https://blog.csdn.net/weixin_42749767/article/details/84592247
