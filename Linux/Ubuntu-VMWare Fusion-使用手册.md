# 1、VMnet1和VMnet8区别

VMnet1对应仅主机模式。
VMnet8对应NAT模式。
```shell
# Mac上查看网络配置内容
cat /Library/Preferences/VMware\ Fusion/networking
`
VERSION=1,0
answer VNET_1_DHCP yes
answer VNET_1_DHCP_CFG_HASH 1BC5B9FF92CD054B65940F87A6CC55417E84CE82
answer VNET_1_HOSTONLY_NETMASK 255.255.255.0
answer VNET_1_HOSTONLY_SUBNET 192.168.101.0
answer VNET_1_HOSTONLY_UUID F59B07C6-BD33-4131-A16B-A4C58941131D
answer VNET_1_VIRTUAL_ADAPTER yes
answer VNET_8_DHCP yes
answer VNET_8_DHCP_CFG_HASH 4B2B934409E2785A21A88EEFFF4C4D8664B5E4C3
answer VNET_8_HOSTONLY_NETMASK 255.255.255.0
answer VNET_8_HOSTONLY_SUBNET 172.16.160.0
answer VNET_8_HOSTONLY_UUID E8BDEC05-6627-4E82-812C-8431F9FF0448
answer VNET_8_NAT yes
answer VNET_8_VIRTUAL_ADAPTER yes
add_bridge_mapping en0 2
`
```

# 2、NAT模式下无法访问外网

## 2.1、查看Mac终端虚拟机NAT网络
```shell
# Mac上查看NAT配置内容
cat /Library/Preferences/VMware\ Fusion/vmnet8/nat.conf
`
# VMware NAT configuration file
# Manual editing of this file is not recommended. Using UI is preferred.

[host]

# Use MacOS network virtualization API
useMacosVmnetVirtApi = 1

# NAT gateway address
ip = 172.16.160.2
netmask = 255.255.255.0

# VMnet device if not specified on command line
device = vmnet8
`
```

这里指定了NAT网关的地址为172.16.160.2，子网掩码为255.255.255.0。
## 2.2、查看Ubuntu的网络配置
```shell
sudo cd /etc/netplan
sudo cat 50-cloud-init.yaml
`
network:
  version: 2
  ethernets:
    ens160:
      dhcp4: false
      addresses: #IP地址
        - 172.16.160.248/24
      routes: #网关
        - to: default
          via: 172.16.160.2
      nameservers: #DNS服务器
        addresses:
          - 8.8.8.8
          - 8.8.4.4
`
```

这里可以得到Ubuntu的网关地址，查看是否和Mac虚拟机NAT网络提供的网关地址一致，如果不一致，替换之，因为网关应该是同一个IP地址。
## 2.3、更新网络配置
```shell
sudo netplan apply
```

[https://cloud.tencent.com.cn/developer/article/2439086](https://cloud.tencent.com.cn/developer/article/2439086)

# 3、分辨率自适应
## 3.1、使用open-vm-tools
VMware官方推荐使用‌open-vm-tools‌替代旧版VMware Tools：
```shell
sudo apt update
sudo apt install open-vm-tools-desktop # 安装桌面版工具
sudo reboot # 重启生效
```

## 3.2、使用挂载光盘
1. 虚拟机菜单 → `虚拟机` → `安装 VMware Tools`
2. 在 Ubuntu 桌面挂载光盘，打开终端执行：
```shell
# 安装 VMware Tools
sudo mount /dev/cdrom /mnt  # 挂载光盘
cd /mnt
## 如果是tar文件就先解压缩
sudo tar -xzf VMwareTools-*.tar.gz -C /tmp
cd /tmp/vmware-tools-distrib

sudo ./vmware-install.pl  # 按提示安装
sudo reboot  # 重启后分辨率自动适配
```

3. 重启虚拟机后分辨率自动适配（若需手动调整，跳转到方法二）。

