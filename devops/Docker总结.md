# 1、安装
## 1.1、Ubuntu系统
```shell
# 更新系统
sudo apt update
# 安装基础依赖（curl、conntrack、NetworkManager等）
sudo apt install -y curl apt-transport-https ca-certificates gnupg lsb-release conntrack

# 添加官方 GPG 与源（20.04/22.04/24.04 通用）
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker

# 允许当前用户管理 docker（避免每次用 sudo）
sudo usermod -aG docker "$USER"
newgrp docker

# 验证
docker version
docker info
```

## 1.2、Windows系统

## 1.3、MacOS系统


# 2、创建容器
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

# 3、持久化存储与网络
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

# 4、常用管理命令
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

# 5、常用问题与优化
## 5.1、镜像加速（国内网络建议）
```shell
## ocker 可配置 /etc/docker/daemon.json：
sudo tee /etc/docker/daemon.json >/dev/null <<'JSON'
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
JSON
sudo systemctl restart docker
```

## 5.2、容器时区与语言
```
# Docker：运行时传入环境变量或挂载宿主时区
docker run -d --name node2 -e TZ=Asia/Shanghai -v /etc/localtime:/etc/localtime:ro ubuntu:24.04 sleep infinity
```

## 5.3、在容器里运行系统服务
Docker 默认不跑 systemd，若确实需要，可用专用镜像或在容器里以 tini/supervisord 管理多进程。

## 5.3、安装SSH
```
docker exec -it node1 bash -c 'apt update && apt install -y openssh-server && service ssh start && netstat -lntp | grep ":22"'
```

## 5.4、创建新用户和密码
```
docker exec -it node1 bash -c 'adduser devuser && passwd devuser'
```

## 5.5、docker-compose命令不存在
现在推荐使用 Docker 的内置子命令 docker compose（空格），而不是旧版 docker-compose（带连字符）。
```
# 检查 Compose v2（内置子命令）
docker compose version
```

若以上命令成功，后续把所有命令中的 `docker-compose` 改为 `docker compose`

