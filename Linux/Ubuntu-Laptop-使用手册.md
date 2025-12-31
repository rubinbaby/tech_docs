# 1、Dell XPS L502x笔记本
## 1.1、Ubuntu版本兼容
因为该笔记本硬件和BIOS比较老旧（2012年），只能安装Ubuntu 20.04或以下版本，Ubuntu 22.04及以上系统只支持新版本BIOS（需要支持UEFI）。
官方驱动：[https://www.dell.com/support/product-details/zh-cn/product/xps-l502x/drivers](https://www.dell.com/support/product-details/zh-cn/product/xps-l502x/drivers)

## 1.2、独立显卡安装
### 1.2.1、确认显卡型号
```shell
lspci | grep -i nvidia
```
输出应包含 NVIDIA Corporation Device [GeForce GT 540M 的 PCI 地址]（如 01:00.0 VGA compatible controller: NVIDIA Corporation GF108M）

### 1.2.2、安装独立显卡驱动
XPS L502x 自带的 GT 525M/540M 属于 Fermi 架构，最佳兼容通常在旧版驱动。
在 Ubuntu 20.04 这类较旧版本更友好，因为Ubuntu 24.04 对 Fermi（390）支持已较弱甚至移除。
```shell
# 查看可用驱动
ubuntu-drivers devices

# 安装旧卡支持的专有驱动（Fermi 通常是 nvidia-driver-390）
sudo apt update && sudo apt install nvidia-driver-390

# 使用 PRIME 切换
sudo prime-select nvidia
# 重启后生效
# 核显/独显切换用 prime-select intel|nvidia

# 验证（重启后运行如下命令查看独显是否工作）
nvidia-smi
```

# 2、配置静态网络
``` shell
sudo cd /etc/netplan
sudo cat > 01-network-manager-all.yaml << 'EOF'
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlp3s0:
      dhcp4: no
      addresses:
        - 192.168.3.32/24
      routes: #网关
        - to: default
          via: 192.168.3.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      access-points:
        "你的WiFi SSID":
          password: "你的WiFi密码"      
  ethernets:
    enp6s0:
      dhcp4: false
      addresses: #IP地址
        - 192.168.3.33/24
      nameservers: #DNS服务器
        addresses:
          - 8.8.8.8
          - 8.8.4.4
EOF

netplan apply

# 为了让自定义的WiFi生效，需要断开之前连接的WiFi（不要让其自动连接），然后选择自定义WiFi（比如，名称为netplan-wlp3s0-zyx）并保持自动连接。
```
若只需一个默认出口，保留其中一个接口的 gateway（或 routes 默认路由），另一接口仅设地址与 DNS，避免策略路由复杂性。

