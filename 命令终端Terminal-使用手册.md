# 1、终端版本汇总

| 版本           | 官方链接                                                                                                           | 支持系统              | 区域  |
| ------------ | -------------------------------------------------------------------------------------------------------------- | ----------------- | --- |
| iTerm2       | [https://iterm2.com/index.html](https://iterm2.com/index.html)                                                 | Mac               |     |
| Tabby        | [https://tabby.sh/](https://tabby.sh/)                                                                         | Mac、Linux、Windows |     |
| Terminator   | [https://gnome-terminator.org/](https://gnome-terminator.org/)                                                 | Linux             |     |
| WindTerm     | [https://kingtoolbox.github.io/](https://kingtoolbox.github.io/)                                               | Mac、Linux、Windows |     |
| SecureCRT    | [https://www.vandyke.com/products/securecrt/index.html](https://www.vandyke.com/products/securecrt/index.html) | Mac、Linux、Windows |     |
| ZOC Terminal | [https://www.emtec.com/zoc/index.html](https://www.emtec.com/zoc/index.html)                                   | Mac、Windows       |     |
| Nex Terminal | [https://www.icons6.top/](https://www.icons6.top/)                                                             | Mac               |     |
| Termius      | [https://termius.com/index.html](https://termius.com/index.html)                                               | Mac、Linux、Windows |     |
| HexHub       | [https://www.hexhub.cn/](https://www.hexhub.cn/)                                                               | Mac、Linux、Windows | 国内  |
| FinalShell   | [https://www.hostbuf.com/](https://www.hostbuf.com/)                                                           | Mac、Linux、Windows | 国内  |
| kitty        | [https://sw.kovidgoyal.net/kitty/](https://sw.kovidgoyal.net/kitty/)                                           | Mac、Linux         |     |
| WezTerm      | [https://wezterm.org/](https://wezterm.org/)                                                                   | Mac、Linux、Windows |     |
| electerm     | [https://electerm.html5beta.com/](https://electerm.html5beta.com/)                                             | Mac、Linux、Windows | 国内  |
| XShell       | [https://www.netsarang.com/en/free-for-home-school/](https://www.netsarang.com/en/free-for-home-school/)       | Windows           |     |
| Mobaxterm    | [https://mobaxterm.mobatek.net/](https://mobaxterm.mobatek.net/)                                               | Windows           |     |
| Royal TSX    | [https://www.royalapps.com/ts/mac/features](https://www.royalapps.com/ts/mac/features)                         | Mac、Windows       |     |

# 2、终端支持框架
## 2.1、ohmyzsh安装
ohmyzsh github: [https://github.com/ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)

themes github: [https://github.com/ohmyzsh/ohmyzsh/wiki/Themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)

### 2.1.1、Ubuntu下安装
Mac系统上默认安装了Zsh。
```shell
sudo apt install zsh curl wget git vim -y
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
#修改终端登录默认shell为zsh
chsh -s $(which zsh)

#使用国内镜像
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh

#重启zsh
exec zsh
```
安装好之后, 把 Zsh 设置为默认的shell
```shell
chsh -s /bin/zsh
```

### 2.1.2、agnoster
主题
字体：[Meslo LG M Regular for Powerline.ttf](https://github.com/powerline/fonts/blob/master/Meslo%20Slashed/Meslo%20LG%20M%20Regular%20for%20Powerline.ttf)

agnosterzak-ohmyzsh-theme github: [https://github.com/zakaziko99/agnosterzak-ohmyzsh-theme](https://github.com/zakaziko99/agnosterzak-ohmyzsh-theme)

agnosterzak.zsh-theme: [http://raw.github.com/zakaziko99/agnosterzak-ohmyzsh-theme/master/agnosterzak.zsh-theme](http://raw.github.com/zakaziko99/agnosterzak-ohmyzsh-theme/master/agnosterzak.zsh-theme)

WSL Ubuntu InputMono font: [https://input.djr.com/download/](https://input.djr.com/download/)

```shell
cd $ZSH_CUSTOM/themes
wget http://raw.github.com/zakaziko99/agnosterzak-ohmyzsh-theme/master/agnosterzak.zsh-theme
sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="agnosterzak"/g' ~/.zshrc
exec zsh
```

### 2.1.3、自动提示命令
```shell
cd ~/.oh-my-zsh/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions
sed -i 's/git)/zsh-autosuggestions git)/g' ~/.zshrc
#让~/.zshrc配置生效，执行如下命令（或者重启item2也行）
source ~/.zshrc
```

### 2.1.4、快速提示并补全目录
```shell
mkdir -p ~/.oh-my-zsh/plugins/incr
wget http://mimosa-pudica.net/src/incr-0.2.zsh -O ~/.oh-my-zsh/plugins/incr/incr.plugin.zsh
```

### 2.1.5、SSH Terminal主题

[https://iterm2colorschemes.com](https://iterm2colorschemes.com)

国外：[https://github.com/rubinbaby/ssh-terminal-themes](https://github.com/rubinbaby/ssh-terminal-themes)

国内：[https://gitee.com/rubinbaby/ssh-terminal-themes](https://gitee.com/rubinbaby/ssh-terminal-themes)

### 2.1.6、自定义终端主题颜色
```text
Ansi 0 Color: #1D2027
Ansi 1 Color: #8D0003
Ansi 10 Color: #7AC100
Ansi 11 Color: #E5E512
Ansi 12 Color: #006EFE
Ansi 13 Color: #BC3FBC
Ansi 14 Color: #19E062
Ansi 15 Color: #FFFFFF
Ansi 2 Color: #3A7D00
Ansi 3 Color: #E5E512
Ansi 4 Color: #092CCD
Ansi 5 Color: #BC3FBC
Ansi 6 Color: #229256
Ansi 7 Color: #FFFFFF
Ansi 8 Color: #394047
Ansi 9 Color: #EB0009
Background Color: #FFFFFF
Bold Color: #EB0009
Cursor Color: #006EFE
Cursor Text Color: #1D2027
Foreground Color: #1D2027
Selected Text Color: #4D5658
Selection Color: #B3B1AA
```

Linux Terminator配置文件：
```shell
# 国内
echo "$(wget -O- https://gitee.com/rubinbaby/ssh-terminal-themes/raw/master/terminator/yinxiao-light.config)" > ~/.config/terminator/config
# 国外
echo "$(wget -O- https://raw.githubusercontent.com/rubinbaby/ssh-terminal-themes/refs/heads/main/terminator/yinxiao-light.config)" > ~/.config/terminator/config
```

## 2.2、Ranger
官方文档：[https://github.com/ranger/ranger](https://github.com/ranger/ranger)

## 2.3、Homebrew
官方文档：[https://brew.sh/](https://brew.sh/)

中文镜像：[https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)

Ubuntu上安装文档：[https://zhuanlan.zhihu.com/p/721712759](https://zhuanlan.zhihu.com/p/721712759)

# 3、kitty终端
## 3.1、主题包
[https://github.com/dexpota/kitty-themes](https://github.com/dexpota/kitty-themes)

# 4、常见问题
## 4.1、zsh: corrupt history file
```shell
# 备份当前历史文件
cp ~/.zsh_history ~/.zsh_history_backup

# 创建一个新的历史文件
strings ~/.zsh_history_backup > ~/.zsh_history
# 其中，strings命令尝试从备份文件中提取可读的字符串，并写入新的历史文件中
```
