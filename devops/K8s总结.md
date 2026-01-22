# 1、安装与使用
## 1.1、Docker安装
参见：[Ubuntu上Docker的安装](Docker总结.md#11Ubuntu系统)

## 1.2、Kubectl安装
- 方法一：通过snap（最简单、官方维护）
```shell
sudo snap install kubectl --classic
kubectl version --client
```

- 方法二：Google 的新 apt 仓库（注意：不再使用 kubernetes-xenial）
```shell
# 导入 key
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# 添加源（以 v1.29 稳定仓库为例，其他版本替换 v1.29）
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null

sudo apt-get update
sudo apt-get install -y kubectl
kubectl version --client
```

## 1.3、Minikube安装
```shell
# 下载最新二进制
curl -LO "https://storage.googleapis.com/minikube/releases/latest/minikube-linux-$(dpkg --print-architecture)"
# 安装到 /usr/local/bin
sudo install "minikube-linux-$(dpkg --print-architecture)" /usr/local/bin/minikube

# 验证
minikube version
```

## 1.4、启动本地K8s集群
使用Docker驱动（默认、最简单）
```shell
# 若上面给用户加了 docker 组，请重新登录终端以生效
minikube start --driver=docker

# 使用国内镜像优化拉取（以阿里云为例）
minikube start --driver=docker --image-mirror-country=cn \
  --registry-mirror=https://registry.docker-cn.com

# 指定自定义仓库镜像源（可替换为你的加速地址）
minikube start --driver=docker --docker-env=HTTP_PROXY=... --docker-env=HTTPS_PROXY=...
```

## 1.5、验证
```shell
# 查看节点
kubectl get nodes

# 部署一个示例服务
kubectl create deployment hello-minikube --image=registry.k8s.io/echoserver:1.10
kubectl expose deployment hello-minikube --type=NodePort --port=8080

# 获取服务地址（minikube 提供快捷命令）
minikube service hello-minikube --url
# 在浏览器或 curl 访问上面的 URL
```

## 1.6、常用操作
```shell
# 查看集群状态
minikube status

# 进入 minikube 节点（Docker 驱动时是一个容器）
minikube ssh

# 开启仪表盘
minikube dashboard

# 指定 Kubernetes 版本或资源大小
minikube start --driver=docker --kubernetes-version=v1.29.0 --cpus=2 --memory=4096

# 停止/删除集群
minikube stop
minikube delete
```

## 1.7、升级与卸载
```shell
# 升级 minikube
sudo minikube update

# 卸载
sudo rm -f /usr/local/bin/minikube
sudo snap remove --purge kubectl
sudo apt-get remove -y kubectl docker.io
```

