# 1、国内版本汇总

| 名称       | 链接                                                                                                                 |
| -------- | ------------------------------------------------------------------------------------------------------------------ |
| 优麒麟      | [https://www.ubuntukylin.com/index-cn.html](https://www.ubuntukylin.com/index-cn.html)                             |
| 统信UOS    | [https://www.chinauos.com/resource/download-professional](https://www.chinauos.com/resource/download-professional) |
| 银河麒麟     | [https://www.kylinos.cn/support/trial/download/](https://www.kylinos.cn/support/trial/download/)                   |
| 深度Deepin | [https://www.deepin.org/index/zh](https://www.deepin.org/index/zh)                                                 |

# 2、启动Linux实例
## 2.1、使用lxd
LXD 提供“系统容器”，看起来更像轻量级的虚拟机（有完整 init、可运行 systemd 服务）。

### 2.1.1、Ubuntu系统
1. 安装与初始化
```shell
# 用 snap 安装
sudo snap install lxd

# 将当前用户加入 lxd 组（用于不加 sudo 管理容器）
sudo usermod -aG lxd "$USER"
newgrp lxd  # 或者注销/重新登录一次

# 初始化 LXD（选择存储池与网络）
lxd init

# lxd init 交互建议：
# - Storage backend: zfs 或 btrfs（若不想额外配置，选 dir 也行但性能较弱）
# - Create a new storage pool: yes
# - Network bridge (e.g. lxdbr0): yes（容器用 NAT 上网）
# - Would you like stale cached images to be removed: yes
# - 来自中国大陆网络：建议设置镜像服务器为官方 simplestreams，或稍后用镜像加速
```

2. 创建多个实例
在虚拟机上搭建的Ubuntu系统中启动多个Ubuntu实例，需要选择支持container类型的image，所以这里选择container类型的image；相反，可以选择支持virtual-machine类型的image。
```shell
# 查看可用镜像（Ubuntu）
lxc image list ubuntu: | grep -i 'container' | grep -i "$(dpkg --print-architecture)" | grep -i '24.04'

# 启动第一个实例（系统容器），并设置资源
lxc launch ubuntu:24.04 node1 -c limits.cpu=1 -c limits.memory=1GB
lxc launch ubuntu:24.04 node2 -c limits.cpu=1 -c limits.memory=1GB

# 查看实例列表与状态
lxc list

# 进入容器交互（类似 ssh）
lxc exec node1 -- bash
```

3. 网络与文件共享
```shell
# 查看容器 IP
lxc list

# 宿主共享目录到容器（持久化挂载）
# 注：把 /data/myshare 替换为你实际目录
sudo mkdir -p /data/myshare
lxc config device add node1 myshare disk source=/data/myshare path=/mnt/share

# 端口映射（从宿主转发到容器）
# 把容器里的 22 端口映射到宿主 2222 端口
lxc config device add node1 sshproxy proxy listen=tcp:0.0.0.0:2222 connect=tcp:127.0.0.1:22
```

4. 常用管理命令
```shell
# 启停与重启
lxc stop node1
lxc start node1
lxc restart node1

# 限制资源（动态调整）
lxc config set node1 limits.cpu 2
lxc config set node1 limits.memory 3GB

# 修改静态IP地址
lxc config device override node1 eth0 ipv4.address=10.200.0.2
lxc restart node1

# 执行命令与复制文件
lxc exec node1 -- apt update
lxc file push ./localfile.txt node1/root/localfile.txt
lxc file pull node1/etc/os-release ./os-release-node1

# 快照与回滚
lxc snapshot node1 snap1
lxc restore node1 snap1

# 删除
lxc delete node2 --force
```

### 2.1.2、常见问题与优化
#### 1）镜像加速（大陆网络建议）
LXD 可在 lxd init 时选用官方 simplestreams

#### 2）容器时区与语言
在容器内设置
```shell
lxc exec node1 -- ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
lxc exec node1 -- dpkg-reconfigure tzdata
```

#### 3）在容器里运行系统服务
LXD 支持 systemd，能跑常驻服务更像 VM。

#### 4）安装SSH
```shell
lxc exec node1 -- bash -c 'apt update && apt install -y openssh-server && systemctl enable ssh && systemctl start ssh && ss -lntp | grep ":22"'
```

#### 5）创建新用户和密码
```shell
lxc exec node1 -- bash -c 'adduser devuser && passwd devuser'
```

#### 6）Failed to create local member network "lxdbr0"
问题：
Error: Failed to create local member network "lxdbr0" in project "default": Failed generating auto config: Failed to automatically find an unused IPv4 subnet, manual configuration required

原因分析：
这个错误是因为 lxd 在自动创建网桥 lxdbr0 时，找不到未被使用的 IPv4 子网，需要你手动指定一个不与现有网络冲突的网段。
- 当前主机上可能已存在 Docker 默认桥接网段 172.17.0.0/16，以及可能还有其他 172.16/172.18/172.19 等子网。
- lxd init 的 “auto” 会尝试挑选未占用的子网，但因冲突或路由占用而失败。

解决步骤（手动指定 IPv4/IPv6 子网）：
在交互式 lxd init 里，将 “What IPv4 address should be used?” 从 auto 改为一个不冲突的网段。例如：
- 如果没有 10.200.0.0/24 被占用，推荐用它。比如：10.200.0.1/24
- 也可以用 10.100.0.0/24、10.99.0.0/24 等任意 RFC1918 私有网段。
- What IPv6 address should be used? → none（若不需要 IPv6，先禁用，后续再启用也可）
这样 lxd 会创建 lxdbr0 网桥，容器会分到 10.200.0.x 的地址。

手动修复已创建网络：
如果 lxd 已创建了一个失败的网络条目，可以先删除再重新创建。
```shell
# 查看现有网络
lxc network list
# 删除失败网络（如果存在）
lxc network delete lxdbr0

# 手动创建指定子网的桥
lxc network create lxdbr0 ipv4.address=10.200.0.1/24 ipv4.nat=true ipv6.address=none
```

#### 7）The requested image couldn't be found for fingerprint
问题：
Error: Failed instance creation: Failed getting remote image info: Failed getting image: The requested image couldn't be found for fingerprint "ubuntu/24.04"

原因分析：
说明当前的镜像远端别名或 simplestreams 未正确配置，导致找不到images:ubuntu/24.04。

解决步骤：
```shell
# 列出已配置远端
lxc remote list

# 如果没有 images:，添加它（官方 community 镜像）
lxc remote add images https://images.linuxcontainers.org --protocol=simplestreams

# 如果没有 ubuntu:，添加 Ubuntu 官方 simplestreams
lxc remote add ubuntu https://cloud-images.ubuntu.com/releases --protocol=simplestreams

# 可选：还可以添加 daily（开发版镜像）
lxc remote add ubuntu-daily https://cloud-images.ubuntu.com/daily --protocol=simplestreams
```

然后，选择输出结果中第二列的fingerprint值作为实例启动image的参数值，比如：b292d60f6bae

## 2.2、使用Docker
Docker 更偏向应用容器，默认没有 systemd；如果你只是需要多环境跑命令/进程，非常高效。

### 2.2.1、Ubuntu系统
1. 安装与检查
```shell
# Ubuntu 官方安装（参考 Docker 官方文档）
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# 添加官方 GPG 与源（20.04/22.04/24.04 通用）
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 允许当前用户管理 docker（避免每次用 sudo）
sudo usermod -aG docker "$USER"
newgrp docker

# 验证
docker version
docker info
```

2. 创建多个容器
```shell
# 运行 ubuntu 容器并进入交互，自动清理退出的容器用 --rm
docker run -it --name node1 ubuntu:24.04 bash
# 在容器内：apt update && apt install -y 必要工具

# 后台运行多个容器
docker run -d --name node2 ubuntu:24.04 sleep infinity
docker run -d --name node3 ubuntu:24.04 sleep infinity

# 进入已有容器
docker exec -it node2 bash

# 查看容器
docker ps -a
```

3. 持久化存储与网络
```shell
# 创建数据卷并挂载
docker volume create vol_node2
docker run -d --name node2 \
  -v vol_node2:/data \
  ubuntu:24.04 sleep infinity

# 宿主目录绑定挂载（最常用）
mkdir -p ~/workdir
docker run -d --name node3 \
  -v ~/workdir:/workspace \
  ubuntu:24.04 sleep infinity

# 端口映射（容器里起服务）
docker run -d --name web1 -p 8080:80 ubuntu:24.04 sleep infinity
# 在 web1 里安装 nginx 后即可通过宿主 8080 访问

# 自定义桥接网络（容器间互通、隔离）
docker network create mynet
docker run -d --name node4 --network=mynet ubuntu:24.04 sleep infinity
docker run -d --name node5 --network=mynet ubuntu:24.04 sleep infinity
# 相互解析：容器 node4 可用主机名 node5 访问 node5
```

4. 常用管理命令
```shell
# 启停、重启、删除
docker stop node2 node3
docker start node2
docker restart node2
docker rm -f node3

# 复制文件
docker cp ./localfile.txt node2:/root/localfile.txt
docker cp node2:/etc/os-release ./os-release-node2

# 查看资源占用
docker stats
```

### 2.2.2、常用问题与优化
```shell
# 1）镜像加速（国内网络建议）
## ocker 可配置 /etc/docker/daemon.json：
sudo tee /etc/docker/daemon.json >/dev/null <<'JSON'
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
JSON
sudo systemctl restart docker

# 2）容器时区与语言
# Docker：运行时传入环境变量或挂载宿主时区
docker run -d --name node2 -e TZ=Asia/Shanghai -v /etc/localtime:/etc/localtime:ro ubuntu:24.04 sleep infinity

# 3) 在容器里运行系统服务
# Docker 默认不跑 systemd，若确实需要，可用专用镜像或在容器里以 tini/supervisord 管理多进程。

# 4）安装SSH
docker exec -it node1 bash -c 'apt update && apt install -y openssh-server && service ssh start && netstat -lntp | grep ":22"'

# 5)创建新用户和密码
docker exec -it node1 bash -c 'adduser devuser && passwd devuser'
```

