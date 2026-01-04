# 1、下载
## 1.1、桌面版
[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)
## 1.2、服务器版
[https://ubuntu.com/download/server](https://ubuntu.com/download/server)
## 1.3、Multipass
[https://cloud.tencent.com.cn/developer/article/1982247](https://cloud.tencent.com.cn/developer/article/1982247)

[https://cloud.tencent.com.cn/developer/article/2078993](https://cloud.tencent.com.cn/developer/article/2078993)

# 2、制作U盘启动盘
## 2.1、下载U盘启动软件
[https://rufus.ie/en/#download](https://rufus.ie/en/#download)

教程：[https://documentation.ubuntu.com/desktop/en/latest/how-to/create-a-bootable-usb-stick](https://documentation.ubuntu.com/desktop/en/latest/how-to/create-a-bootable-usb-stick)
## 2.2、载入系统镜像
![Pasted image 20251225104246.png](../images/Pasted%20image%2020251225104246.png)

# 3、安装
## 3.1、安装步骤
[https://cloud.tencent.com.cn/developer/article/2383208](https://cloud.tencent.com.cn/developer/article/2383208)

## 3.2、替换软件源
```shell
sudo cd /etc/apt
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

# 对于Ubuntu18，需要替换版本名字
sed -i 's/focal/bionic/g' sources.list

sudo apt update
```

## 3.3、安装ssh等基础包
打开Ubuntu的终端，输入：
```shell
sudo apt-get install openssh-server net-tools curl sysstat ffmpeg
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

# 安装obsidian软件
# 下载官方deb软件包 https://obsidian.md/download
sudo dpkg -i obsidian_*.deb

# 安装Slack软件
# 下载官方deb软件包 https://slack.com/downloads/linux
sudo dpkg -i slack-desktop-*.deb

# 安装Zoom软件
# 下载官方deb软件包 https://zoom.us/download
sudo dpkg -i zoom_*.deb

# 安装htop监控工具
sudo apt install htop

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

# 安装rar文件解压缩工具
apt install unrar
```

