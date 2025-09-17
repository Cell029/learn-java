
# 1. 安装 VMware

>在安装前，如果有旧版本的 VMware 则需要先删除，包括以前的数据记录，然后再进行下载。

安装以后可以免费试用，可以去官网购买正版许可证，或者去网上找许可证。启动后的界面如图所示：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917133901.png)

如果 VMware 虚拟机运行报错，例如：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917133922.png)

这个是由于英特尔的虚拟化技术没有开启,，需要进入系统的 BIOS 界面开启英特尔的虚拟化技术，不过一般情况下都是开启的状态。

****
# 2. 创建虚拟机

Centos7 是比较常用的一个 Linux 发行版本，在国内的使用比例还是比较高的，所以首先要下载一个 Centos7 的 iso 文件，在后续会用到。

在 VMware 主页界面中点击 “创建新的虚拟机” 按钮：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134140.png)

然后会弹出一个窗口，选择典型并点击下一步：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134202.png)

然后选择 “稍后安装操作系统” 并点击下一步：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134448.png)

这里选择 Linux 和下载的对应版本的 CentOS：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134529.png)

然后就可以给虚拟机命名并自定义虚拟机将来保存的位置：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134641.png)

再次下一步，填写虚拟机磁盘大小，这里建议给大一点，否则将来不够用调整起来麻烦。而且这里设置大小并不是立刻占用这么多，而是设置一个上限：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134854.png)

继续下一步，然后选择虚拟机硬件设置：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917134923.png)

在弹出的窗口中设置虚拟机硬件，建议 CPU 给到 4 核，内存给到 8G：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135001.png)

再点击 “新 CD/DVD(IDE)”，选择 “使用 ISO 映像文件”，把之前下载好的 ISO 文件放进来：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135036.png)

配置完成后，点击关闭，回到上一页面，继续点击完成：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135212.png)

此时虚拟机就安装完成了：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135246.png)

****
# 3. 安装 CentOS 7

接下来，启动刚刚创建的虚拟机，开始安装 Centos7 系统，启动后需要选择安装菜单，将鼠标移入黑窗口中后，将无法再使用鼠标，需要按上下键选择菜单。选中 Install Centos 7 后按下回车：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135409.png)

然后会提示按下 enter 键继续：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135428.png)

过一会儿后，会进入语言选择菜单，这里可以使用鼠标选择。选择中文-简体中文，然后继续：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135444.png)

接下来，会进入安装配置页面：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135504.png)

鼠标向下滚动后，找到系统-安装位置配置，点击：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135528.png)

选择刚刚添加的磁盘，并点击完成：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135546.png)

然后回到配置页面，这次点击 “网络和主机名”：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135605.png)

在网络页面做下面的几件事情：

1. 自定义主机名，不要出现中文和特殊字符，建议用 localhost
2. 点击应用
3. 将网络连接打开
4. 点击配置，设置详细网络信息

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917135637.png)

记住上图中的网络详细信息，接下来的配置要参考：

![](images/VM%20虚拟机安装%20Linux%20CentOS/5bc42641ca3404bceb9b0860fae39141.png)

点击配置按钮后需要把网卡地址改为静态 IP，这样可以避免每次启动虚拟机 IP 都变化：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917140629.png)

最后，点击完成按钮。回到配置界面后，点击开始安装，此过程可以设置 root 密码，填写要使用的 root 密码，然后点击完成：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917140730.png)

等待安装完成后，点击重启：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917140822.png)

耐心等待一段时间，不要做任何操作，虚拟机即可启动完毕：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917140925.png)

输入登录的账号密码即可（root 账号以及刚刚设置的 root 的密码）：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917141000.png)

只要密码输入正确，就可以正常登录。此时可以用命令测试虚拟机网络是否畅通：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917141105.png)

****
# 4. 设置虚拟机快照

在虚拟机安装完成后，最好立刻设置一个快照，这样一旦将来虚拟机出现问题，可以快速恢复。先停止虚拟机，点击 VMware 顶部菜单中的暂停下拉选框，选择关闭客户机：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917141154.png)

接着，点击VMware菜单中的🔧按钮，然后在弹出的快照管理窗口中，点击拍摄快照，填写新的快照信息：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917141259.png)

快照拍摄完成了，而且可以在不同阶段拍摄多个不同快照作为备份，方便后期恢复数据。假如以后虚拟机文件受损，需要恢复到初始状态的话，可以选中要恢复的快照，点击转到即可。

****
# 5. SSH 客户端

在VMware界面中操作虚拟机非常不友好，所以一般推荐使用专门的SSH客户端。市面上常见的有：

- Xshell：个人免费，商业收费，之前爆出过有隐藏后门，不推荐。
- Finshell：基础功能免费，高级功能收费，基于Java，内存占用较高（在1个G左右），不推荐
- MobarXterm：基础功能免费、高级功能收费。开源、功能强大、内存占用低（只有10m左右），但是界面不太漂亮，推荐使用。

这里选择内存占用较低的 MobarXterm 作为 SSH 客户端，其官网地址：[MobaXterm free Xserver and tabbed SSH client for Windows](https://mobaxterm.mobatek.net/)

安装完成后界面如图所示：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917141758.png)

点击 session 按钮，进入会话管理，在弹出的 session 管理页面中，按照下图填写信息并保存：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142005.png)

点击 OK 后会提示你是第一次连接，询问是否信任连接的服务：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142036.png)

选择 accept 之后，会询问是否要记住密码，选择 yes：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142103.png)

接着就可以直接进入刚刚设置的 SSH 了，在界面的左侧就是 FTP 面板，展示的是虚拟机的文件系统，可以直接以拖曳的方式上传或下载文件：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142325.png)

首先建议设置一下默认编辑器，这样可以通过 MobarXterm 的 FTP 工具打开文件时会以指定的编辑器打开，方便修改。我这里配置的是vscode：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142221.png)

接着可以配置一下右键粘贴，MobarXterm默认左键选中即**复制**，但是需要手动配置右键点击为粘贴：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142459.png)

接下来还有一些配置：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142540.png)

分别是：

- 默认的登录用户
- ssh保持连接
- 取消连接成功后的欢迎 banner

大多数情况下没有 x-server 的需求，因此可以选择不要自启动：

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917142620.png)

****
# 6. 安装 Docker

```shell
# 1. 卸载旧版本（如果安装过）
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

# 2. 安装所需的依赖包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 3. 设置稳定的 Docker 仓库（使用阿里云镜像加速）
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 4. 更新 yum 软件包索引并安装 Docker CE（社区版）
sudo yum makecache fast
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 5. 启动 Docker 服务并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 6. 验证安装是否成功
sudo docker run hello-world
```

在虚拟机中安装的 Docker 没有可视化界面，操作起来不太方便，而 Portainer 是一个轻量级的管理 UI，可以轻松管理 Docker 主机或 Swarm 集群。

```shell
# 创建 Portainer 需要使用的数据卷
sudo docker volume create portainer_data

# 下载并运行 Portainer 容器
sudo docker run -d -p 9000:9000 --name portainer --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

- `-p 9000:9000`: 将容器的 9000 端口映射到虚拟机的 9000 端口。
- `-v /var/run/docker.sock...`: 这是最重要的部分，允许 Portainer 容器与 Docker 守护进程通信。
- `--restart always`: 容器总是重启，保证服务一直在线。

在本地浏览器输入虚拟机的 ip 访问 Portainer 界面：`http://192.168.149.101:9000`，首次访问需要创建管理员账号（输入用户名和密码），然后可以选择连接到的 Docker 环境。选择 "Local" （因为它通过 `docker.sock` 已经连接到了本地的 Docker 引擎）。

![](images/VM%20虚拟机安装%20Linux%20CentOS/file-20250917145309.png)

****
