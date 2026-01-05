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
# 切到 prime-select nvidia 后，默认整个系统使用独显渲染。

# 验证（重启后运行如下命令查看独显是否工作）
nvidia-smi
```

在Ubuntu18上安装独显：
```shell
# 安装核心依赖
sudo apt install xserver-xorg-core xserver-xorg-video-intel

# 安装旧卡支持的专有驱动（Fermi 通常是 nvidia-driver-390）
# sudo apt install nvidia-driver-390 bumblebee-nvidia primus
sudo apt install nvidia-driver-390 nvidia-prime
sudo usermod -aG bumblebee $USER

# 切换会话与登录管理器（老卡更稳的组合），可换 LightDM 更稳
sudo apt install lightdm
sudo dpkg-reconfigure lightdm   # 选择 lightdm 为默认
sudo reboot
```

即使系统版本改为Ubuntu18，还是发生卡死现象，所以把会话固定到Xorg、用核显为主、回退到GA内核并检查驱动与扩展，能解决大多数卡死。
```shell
# 1. 强制使用 Xorg，会话以核显为主
# 切核显（PRIME）
sudo apt update
sudo apt install -y nvidia-prime
sudo prime-select intel

# 清理强制独显的Xorg配置
sudo rm -f /etc/X11/xorg.conf /etc/X11/xorg.conf.d/90-nvidia.conf

# 重启到LightDM + Xorg
sudo reboot

# 验证当前为 Xorg
echo $XDG_SESSION_TYPE   # 期望 x11

# 2. 固定到 GA 5.4 内核栈（避免HWE与390冲突）
sudo apt update
sudo apt install -y linux-generic
sudo reboot

##### 以下方法未验证

# 3. 进 BIOS/UEFI 关闭 Secure Boot；否则未签名的 390 模块可能被拒绝加载

# 4. 切换 LightDM 的 greeter，减少桌面层问题（可选）
sudo apt install -y slick-greeter
sudo sed -i 's/^#*greeter-session.*/greeter-session=slick-greeter/' /etc/lightdm/lightdm.conf
sudo systemctl restart lightdm

# 5. 禁用可能导致冻结的 GNOME Shell 扩展与自启项（用户态）
mv ~/.local/share/gnome-shell/extensions ~/.local/share/gnome-shell/extensions.bak
mv ~/.config/autostart ~/.config/autostart.bak

# 6. 显卡相关内核参数与模块设置（针对核显与旧NVIDIA）
# 禁用Intel面板自刷新，降低冻结概率
echo 'GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i915.enable_psr=0"' | sudo tee /etc/default/grub > /dev/null
sudo update-grub

# 打开NVIDIA DRM模式（在需要独显时更稳）
echo 'options nvidia-drm modeset=1' | sudo tee /etc/modprobe.d/nvidia-drm.conf
sudo update-initramfs -u
sudo reboot

# 7. 按需启用独显渲染（保留核显显示）
# 示例验证（独显渲染器应显示NVIDIA）
DRI_PRIME=1 glxinfo | grep "OpenGL renderer"

# 8. 仍卡死时的兜底与定位
# 尝试临时禁用专有驱动，换开源栈（稳定但性能低）
sudo apt purge 'nvidia-*'
sudo apt install -y xserver-xorg-video-nouveau
sudo update-initramfs -u
sudo reboot

# 收集冻结前日志（从上一次启动的内核日志）
journalctl -b -1 -k | grep -Ei 'nvidia|nouveau|i915|hang|Xid|NVRM'

# 9. 外接显示器下用全局独显（仅当外屏接到独显HDMI/DP）
sudo prime-select nvidia
sudo reboot
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

