# 1、下载

## 1.1、Dell XPS L502x笔记本
因为该笔记本硬件和BIOS比较老旧（2012年），只能安装Ubuntu 20.04或以下版本，Ubuntu 22.04及以上系统只支持新版本BIOS（支持UEFI）。
官方驱动：[https://www.dell.com/support/product-details/zh-cn/product/xps-l502x/drivers](https://www.dell.com/support/product-details/zh-cn/product/xps-l502x/drivers)

## 1.2、配置静态网络
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
      routes: #网关
        - to: default
          via: 192.168.3.1
      nameservers: #DNS服务器
        addresses:
          - 8.8.8.8
          - 8.8.4.4
EOF
```
若只需一个默认出口，保留其中一个接口的 gateway（或 routes 默认路由），另一接口仅设地址与 DNS，避免策略路由复杂性。