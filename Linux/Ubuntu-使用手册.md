# 1、下载
## 1.1、桌面版
Intel/AMD版本：[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

ARM版本：[https://cdimage.ubuntu.com/releases/](https://cdimage.ubuntu.com/releases/)

## 1.2、服务器版
[https://ubuntu.com/download/server](https://ubuntu.com/download/server)

## 1.3、Multipass
[https://cloud.tencent.com.cn/developer/article/1982247](https://cloud.tencent.com.cn/developer/article/1982247)

[https://cloud.tencent.com.cn/developer/article/2078993](https://cloud.tencent.com.cn/developer/article/2078993)

# 2、制作U盘启动盘
## 2.1、下载U盘启动软件
[rufus 启动盘软件下载](https://rufus.ie/en/#download)

教程：[how to create a bootable usb stick](https://documentation.ubuntu.com/desktop/en/latest/how-to/create-a-bootable-usb-stick)

## 2.2、载入系统镜像
![Pasted image 20251225104246.png](../images/Pasted%20image%2020251225104246.png)

# 3、安装
## 3.1、安装步骤
[https://cloud.tencent.com.cn/developer/article/2383208](https://cloud.tencent.com.cn/developer/article/2383208)

## 3.2、替换软件源
```shell
sudo cd /etc/apt
# 1. 对于Ubuntu22.04及以下系统
sudo cp sources.list sources.list.backup.xxx
sudo cat > sources.list << 'EOF'
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu focal-security main restricted
deb http://security.ubuntu.com/ubuntu focal-security universe
deb http://security.ubuntu.com/ubuntu focal-security multiverse
EOF

## 对于Ubuntu18，需要替换版本名字
sed -i 's/focal/bionic/g' sources.list

# 2. 对于Ubuntu24.04及以上系统
sudo cd sources.list.d
sudo cp ubuntu.sources ubuntu.sources.backup.xxx
sudo sed -i 's@http://ports.ubuntu.com@http://mirrors.aliyun.com@g' ubuntu.sources
sudo echo "" >> ubuntu.sources
sudo sed -i 's@http://ports.ubuntu.com@http://mirrors.tuna.tsinghua.edu.cn@g' ubuntu.sources

sudo apt update
```

## 3.3、安装ssh等基础包
打开Ubuntu的终端，输入：
```shell
sudo apt-get install openssh-server net-tools curl ffmpeg
# 安装htop监控工具
sudo apt install htop
# 安装rar文件解压缩工具
apt install unrar
```
安装完毕后ssh默认已启动。可以使用下述命令查看是否有进程在22端口上监听，即是否已启动：
```shell
netstat -nat | grep 22
```

## 3.4、安装搜狗输入法
安装指南： [https://shurufa.sogou.com/linux/guide](https://shurufa.sogou.com/linux/guide)

```shell
# 安装fcitx
sudo apt install fcitx-bin
sudo apt-get install fcitx-table

# 设置fcitx开机自启动
sudo cp /usr/share/applications/fcitx.desktop /etc/xdg/autostart/

# 下载搜狗输入法安装包
# https://shurufa.sogou.com/linux

# 安装下载的搜狗输入法
cd ~/Downloads/
sudo dpkg -i sogoupinyin_*.deb

# 安装依赖 (不安装的话，无法输入中文)
sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2 ibgsettings-qt1

# 重启电脑
```

系统重启，添加搜狗输入法：

![Pasted image 20251225104817.png](../images/Pasted%20image%2020251225104817.png)

## 3.5、安装第三方软件
### 3.5.1、使用内置命令安装
```shell
# 安装WPS办公软件
# 下载官方deb软件包 https://www.wps.cn/product/wpslinux
sudo dpkg -i wps-office_*.deb

# 安装微信
# 下载官方deb软件包 https://linux.weixin.qq.com/en
sudo dpkg -i WeChatLinux_*.deb

# 安装VLC播放器
sudo apt install vlc

# 安装腾讯会议
# 下载官方deb软件包 https://meeting.tencent.com/download/
sudo dpkg -i TencentMeeting_*.deb

# 安装Microsoft Edge浏览器
# 下载官方deb软件包 https://www.microsoft.com/zh-cn/edge/download
sudo dpkg -i microsoft-edge-*.deb

# 安装Snipaste截图软件
# 下载官方AppImage软件包 https://dl.snipaste.com/linux
# 赋予执行权限
chmod +x Snipaste-*.AppImage
# 移动AppImage文件到固定目录
mkdir -p ~/.local/bin
mv ~/Downloads/Snipaste-*.AppImage ~/.local/bin/
# 下载图标
mkdir -p ~/.local/share/icons
mv ~/Downloads/snipaste.svg ~/.local/share/icons/
# 创建.desktop桌面快捷方式文件
cat > ~/.local/share/applications/snipaste.desktop << 'EOF'
[Desktop Entry]
Name=Snipaste
Exec=/home/xxxxxx/.local/bin/Snipaste-xxx.AppImage
Icon=/home/xxxxxx/.local/share/icons/snipaste.svg
Type=Application
Categories=Utility;
Terminal=false
EOF
# 设置开机自启
# 打开“启动应用程序”（Startup Applications），点击“添加”：
# 名称：Snipaste
# 命令：/home/xxxxxx/.local/bin/Snipaste-xxx.AppImage
# 注释：可选
# 对于老电脑，可以使用flameshot来代替 https://flameshot.org/
sudo apt install flameshot

# 安装obsidian软件
# 下载官方deb软件包 https://obsidian.md/download
sudo dpkg -i obsidian_*.deb
# 对于老电脑，可以安装mark-text https://github.com/marktext/marktext
sudo snap install marktext

# 安装Slack软件
# 下载官方deb软件包 https://slack.com/downloads/linux
sudo dpkg -i slack-desktop-*.deb

# 安装Zoom软件
# 下载官方deb软件包 https://zoom.us/download
sudo dpkg -i zoom_*.deb

# 安装GIMP修图软件
# 下载官方AppImage软件包 https://www.gimp.org/downloads
# 赋予执行权限
chmod +x GIMP-*.AppImage
# 移动AppImage文件到固定目录
mkdir -p ~/.local/bin
mv ~/Downloads/GIMP-*.AppImage ~/.local/bin/
# 下载图标
mkdir -p ~/.local/share/icons
mv ~/Downloads/gimp.png ~/.local/share/icons/
# 创建.desktop桌面快捷方式文件
cat > ~/.local/share/applications/gimp.desktop << 'EOF'
[Desktop Entry]
Name=GIMP
Exec=/home/xxxxxx/.local/bin/GIMP-xxx.AppImage
Icon=/home/xxxxxx/.local/share/icons/gimp.png
Type=Application
Categories=Utility;
Terminal=false
EOF
```

### 3.5.2、使用Flatpak安装软件
#### 3.5.2.1、配置
- Ubuntu 18.04/20.04：apt安装flatpak后即可使用；首次需添加Flathub。
- Ubuntu 22.04/24.04：系统自带支持更完善，按相同步骤即可。
- 更老版本（16.04等）：仍可用，但需要启用PPA或手动安装，维护度较低。

文档：[https://flathub.org/en/setup/Ubuntu](https://flathub.org/en/setup/Ubuntu)

```shell
# 1. 安装flatpak
sudo apt install flatpak

# 2. 安装GNOME Software Flatpak插件
sudo apt install gnome-software-plugin-flatpak

# 3. 添加Flathub repository
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

#### 3.5.2.2、软件安装更新
```shell
# 安装Obsidian
flatpak install flathub md.obsidian.Obsidian

# 安装GIMP
flatpak install flathub org.gimp.GIMP

# 安装WPS office
flatpak install flathub cn.wps.wps_365

# 安装微信
flatpak install flathub com.tencent.WeChat

# 安装VLC播放器
flatpak install flathub org.videolan.VLC

# 安装Flameshot截图软件
flatpak install flathub org.flameshot.Flameshot

# 安装Typora MD软件
flatpak install flathub io.typora.Typora

# 安装Microsoft VScode
flatpak install flathub com.visualstudio.code

# 更新
flatpak update
```

#### 3.5.2.3、软件卸载
```shell
# 1.卸载应用本体并删除其数据
## 用户安装（默认大多数应用是用户级）：
flatpak list --user
flatpak uninstall --user --delete-data <应用ID>
## 系统安装（需要管理员权限）：
flatpak list --system
sudo flatpak uninstall --system --delete-data <应用ID>

# 2. 清理未使用的运行时与扩展
## 用户级未用依赖：
flatpak uninstall --user --unused
## 系统级未用依赖：
sudo flatpak uninstall --system --unused

# 3. 清理缓存与残留（可选，更彻底）
rm -rf ~/.cache/flatpak
## 若曾做过手动覆盖，重置或删除对应文件
flatpak override --reset <应用ID>  # 重置权限覆盖
rm -rf ~/.config/flatpak/overrides  # 删除所有覆盖（谨慎）

# 4. 移除不再需要的源（仅在确认不用该仓库时）
flatpak remotes
flatpak remote-delete <远程名>     # 用户级
sudo flatpak remote-delete <远程名> # 系统级

# 5. 一次性“清仓”示例（谨慎执行）
## 删除所有用户级应用与数据：
flatpak uninstall --user --all --delete-data
flatpak uninstall --user --unused
## 删除所有系统级应用与数据：
sudo flatpak uninstall --system --all --delete-data
sudo flatpak uninstall --system --unused
```

### 3.5.3、使用Snap安装软件
```shell
# 安装Gitkraken GIT GUI工具
sudo snap info gitkraken
sudo snap install gitkraken --classic

# 删除
sudo snap remove --purge <应用名>
```

# 4、常见问题
## 4.1、error while loading shared libraries: libatomic.so.1
问题：error while loading shared libraries: libatomic.so.1: cannot open shared object file: No such file or directory
解决：
- 安装库并刷新链接缓存
```shell
sudo apt update
sudo apt install libatomic1
sudo ldconfig
```

- 验证是否已安装
```shell
ldconfig -p | grep libatomic.so.1
```

## 4.2、error while loading shared libraries: libz.so
问题：error while loading shared libraries: libz.so: cannot open shared object file: No such file or directory
解决：
缺少 zlib 的共享库。安装 zlib1g 与 zlib1g-dev 即可。
- 安装库并刷新链接缓存
```shell
sudo apt update
sudo apt install zlib1g zlib1g-dev
sudo ldconfig
```

- 验证是否已安装
```shell
ldconfig -p | grep libz.so
```

## 4.3、dlopen(): error loading libfuse.so.2
问题：dlopen(): error loading libfuse.so.2     AppImages require FUSE to run.
解决：
缺少 libfuse.so.2 导致的；安装 FUSE 库即可解决。
- 对于Ubuntu24.04及以上
```shell
sudo apt update

# 默认主推 fuse3
sudo apt install fuse3
# 但仍可装兼容的 v2 运行库：
sudo apt install libfuse2

sudo ldconfig
```

- 对于Ubuntu 20.04和22.04
```shell
# 安装 FUSE v2 运行库与工具
sudo apt update
sudo apt install libfuse2 fuse
sudo ldconfig
```
