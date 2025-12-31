官方文档：[https://iterm2.com/documentation.html](https://iterm2.com/documentation.html)

# 1、clone session config
实现类似于secureCRT一样的session clone功能
操作步骤
在~目录下的.ssh文件夹中创建一个config文件
文件内容输入：
```text
host *
ControlMaster auto
ControlPath ~/.ssh/master-%r@%h:%p
```
这样的话，当登录第一次登录跳板机器的时候，和往常一样，但是第二次登录同样的跳板机的时候，就不需要输入token了。
这样可以在一定程度上减少输入token的次数。
在~/.ssh/目录下可以看到master-的sock文件。它记录了你目前登录到的机器，这样的话，你登录同样的机器就会重用同一个链接了。
config文件详细配置说明：[http://linux.die.net/man/5/ssh_config](http://linux.die.net/man/5/ssh_config)

# 2、Shell Integration
官方文档： [https://iterm2.com/documentation-shell-integration.html](https://iterm2.com/documentation-shell-integration.html)

## 2.1、菜单项安装
安装步骤：
1. iTerm2 -->安装Shell集成（Install Shell Integration）
2. 询问你是否安装「Utilities」。Utilities 是一些 Shell 脚本，提供诸如内联图像显示、文件传输、界面自定义等功能。
3. 选择下载安装程序并运行，或进行无网络安装。 
	1) 下载并运行的选项，会输入以下命令：
		```shell
		curl -L https://iterm2.com/shell_integration/zsh \
		-o ~/.iterm2_shell_integration.zsh
		```
	2) 无网络安装，会运行一系列命令来确定你的 Shell、安装文件，并修改你的点文件以在登录时自动加载 Shell 集成。

## 2.2、手动安装
为了让iTerm2能够通过ssh到目标Linux服务器来显示其当前工作目录、主机名等数据，需要在Mac iTerm2和目标服务器上分别安装和启用shell integration。
```shell
# Zsh安装
curl -L https://iterm2.com/shell_integration/zsh \
-o ~/.iterm2_shell_integration.zsh

# Bash安装
curl -L https://iterm2.com/shell_integration/bash \
-o ~/.iterm2_shell_integration.bash
```
然后，需要确保登录时加载该脚本，将以下内容添加到~/.zshrc的末尾：
```shell
source ~/.iterm2_shell_integration.zsh
```

## 2.3、自动加载
iTerm2 --> 设置（Settings）-->资料库（Profiles）-->通用（General）-->命令部分（Command Section），然后勾选**自动加载Shell集成（Load shell integration automatically）**

# 3、启用自定义Status Bar Component
前提条件：需要启用前面提到的Shell integration。
官方文档：[https://iterm2.com/documentation-status-bar.html](https://iterm2.com/documentation-status-bar.html)

## 3.1、使用新建自定义Python脚本来显示
### 3.1.1、安装并启用Python运行环境
- 菜单Scripts --> Manage --> 安装/启用Python（按提示授权”辅助功能/自动化“）
- 安装后建议重启iTerm2
### 3.1.2、创建自定义脚本
官方教程：[https://iterm2.com/python-api/tutorial/index.html](https://iterm2.com/python-api/tutorial/index.html)

菜单Scripts --> Manage --> 新建Python脚本
比如，文件名：remote_status.py  （获取当前session的CPU等信息）
```python
#!/usr/bin/env python3
import iterm2
from subprocess import check_output

def get_shell_script(profile=None):
    command = None if profile is None or profile.command == "" or profile.command is None else profile.command.strip()
    if command and (command.startswith("ssh") or command.startswith("expect")):
        shell_script = command.split()[-1]
    else:
        shell_script = "localhost"
    return shell_script

def get_ssh_info(profile=None):
    shell_script = get_shell_script(profile)
    if shell_script is None:
        return None    
    try:
        if shell_script == "localhost":
            username = check_output(r"whoami", shell=True, universal_newlines=True)
            ip = "localhost"
            return f"{username.strip()}@{ip.strip()}"
        username = check_output(r"cat {script}|grep -i 'set user'|awk '{{print $3}}'".format(script=shell_script), shell=True, universal_newlines=True)
        ip = check_output(r"cat {script}|grep -i 'set host'|awk '{{print $3}}'".format(script=shell_script), shell=True, universal_newlines=True)
        return f"{username.strip()}@{ip.strip()}"
    except Exception:
        return None

def get_monitoring_usages(profile=None):
    ssh_info = get_ssh_info(profile)
    if ssh_info is None:
        return "CPU: N/A"

    MEM_CMD = "LC_ALL=C free | awk \"/Mem:/ {printf(\\\"%.1f%%\\n\\\",\$3/\$2*100)}\""
    mem_cmd = r"ssh {ssh_info} '{mem_cmd}'".format(ssh_info=ssh_info, mem_cmd=MEM_CMD)
    DISK_CMD = "df -P | awk \"NR>1{printf \\\"%-20s: %6.2f%% used\\n\\\",\$6,(100-\$4/\$2*100)}\"|grep -iE \"/(\\s+):\"|awk \"{print \$3}\""
    disk_cmd = r"ssh {ssh_info} '{disk_cmd}'".format(ssh_info=ssh_info, disk_cmd=DISK_CMD)
    UPTIME_CMD = "uptime|awk -F\"up\" \"{print \$2}\"|awk -F\"load \" \"{print \$1}\"|awk \"{s=\$0; sub(/,[^,]*$/, \\\"\\\", s); sub(/,[^,]*$/, \\\"\\\", s); print s}\""
    uptime_cmd = r"ssh {ssh_info} '{uptime_cmd}'".format(ssh_info=ssh_info, uptime_cmd=UPTIME_CMD)
    CPU_CMD = "iostat|awk \"/id/{p=1; next} p{print; p=0}\"|awk \"{printf(\\\"%.1f%%\\n\\\", (100-\$6))}\""
    cpu_cmd = r"ssh {ssh_info} '{cpu_cmd}'".format(ssh_info=ssh_info, cpu_cmd=CPU_CMD)
    try:
        mem_output = check_output(mem_cmd, shell=True, universal_newlines=True)
        disk_output = check_output(disk_cmd, shell=True, universal_newlines=True)
        uptime_output = check_output(uptime_cmd, shell=True, universal_newlines=True)
        cpu_output = check_output(cpu_cmd, shell=True, universal_newlines=True)
        output = f"CPU: {cpu_output.strip()} | Memory: {mem_output.strip()} | Disk: {disk_output.strip()} | Uptime: {uptime_output.strip()}"
        return f"{output.strip()}"
    except Exception:
        return "CPU: N/A"
    
async def main(connection):

    component = iterm2.StatusBarComponent(
        identifier="remote-status",
        short_description="Remote Status",
        detailed_description="Show lightweight CPU, memory, disk and uptime info from the focused session",
        knobs=[],               # 可添加自定义开关/刷新频率等
        exemplar="CPU | Memory | Disk | Uptime",
        update_cadence=10,       # 刷新间隔，稍大以避免频繁执行命令
    )
    
    @iterm2.StatusBarRPC
    async def show_remote_status(knobs):
        app = await iterm2.async_get_app(connection)
        window = app.current_terminal_window
        if window is None or window.current_tab is None or window.current_tab.current_session is None:
            return "—"
        session = window.current_tab.current_session
        prof = await session.async_get_profile() if hasattr(session, "async_get_profile") else None
        # 主动读取会话变量（可能在本地或远端 SSH 会话中）
        host = await session.async_get_variable("session.hostname") or ""
        pwd  = await session.async_get_variable("session.path") or ""
        monitor_results = get_monitoring_usages(prof if prof else None)
        print("Fetched variables:", host, pwd, monitor_results)
        parts = []
        if monitor_results:  parts.append(monitor_results)
        return " | ".join(parts) if parts else "—"
    
    await component.async_register(connection, show_remote_status)

iterm2.run_forever(main)
```

赋权：
```shell
chmod +x ~/Library/Application\ Support/iTerm2/Scripts/AutoLaunch/remote_status.py
```

在启动iTerm2的时候，如果想要让脚本自动执行，需要把该脚本文件放入`~/Library/Application\ Support/iTerm2/Scripts/AutoLaunch/`。
重启 iTerm2，或 Scripts → Manage → 重新加载脚本，确认无报错。
具体代码中Python对象和方法使用参考 **#3.1.4、Python API**

### 3.1.3、添加自定义component
在 Profile 的 Status Bar 选择手动创建的component（3.2中创建的）
- Preferences → Profiles → 选你的 Profile → 勾选“Status bar enabled”→ Configure Status Bar…
- 添加组件（比如，Remote Status，之前3.2中手动创建的component) 进入Active Components中

### 3.1.4、Python API
官方文档：[https://iterm2.com/python-api/](https://iterm2.com/python-api/)

## 3.2、使用Interpolated String来显示
官方文档：[https://iterm2.com/documentation-scripting-fundamentals.html](https://iterm2.com/documentation-scripting-fundamentals.html)

### 3.2.1、创建用户自定义变量
```shell
cat > ~/.iterm2_remote_status.zsh << 'EOF'
# 每 5 秒更新一次系统指标
function _update_iterm2_metrics() {
  # CPU指标
  cpu=$(iostat | awk '/id/{p=1; next} p{print; p=0}' | awk '{printf("CPU: %.1f%%\n", (100-$6))}')
  iterm2_set_user_var iTermCpu ${cpu}

  # 内存使用指标
  if [ "$(uname -s)" = "Linux" ]; then
    mem=$(LC_ALL=C free | awk '/Mem:/ {printf("Memory: %.1f%%\n", $3/$2*100)}')
    iterm2_set_user_var iTermMem ${mem}
  fi

  # 磁盘使用指标
  disk=$(df -P | awk 'NR>1 && $1 !~ /^(map|devfs)$/ {printf "%-20s: %6.2f%% used\n",$6,(100-$4/$2*100)}'|grep -iE "/(\s+):"|awk '{print "Disk: "$3}')
  iterm2_set_user_var iTermDisk ${disk}

  # 系统已启动时间指标
  uptime=$(uptime | awk -F"up" '{print $2}' | awk -F"load " '{print $1}' | awk '{s=$0; sub(/,[^,]*$/, "", s); sub(/,[^,]*$/, "", s); print "Uptime: "s}')
  iterm2_set_user_var iTermUptime ${uptime}
}

# 定时器：避免每条命令都跑
typeset -g LAST_ITERM2_METRICS_TS=0
function precmd() {
  local now=$(date +%s)
  #echo "test" >> /home/yinxiao/test.txt
  if (( now - LAST_ITERM_CPU_TS >= 5 )); then
    _update_iterm2_metrics
    LAST_ITERM2_METRICS_TS=$now
  fi
}
EOF

chmod +x ~/.iterm2_remote_status.zsh

echo "test -e \"\${HOME}/.iterm2_remote_status.zsh\" && source \"\${HOME}/.iterm2_remote_status.zsh\" || true" >> ~/.zshrc
```

### 3.2.2、添加Interpolated Strings

- Preferences → Profiles → 选你的 Profile → Configure Status Bar…
- 添加Interpolated Strings
- 输入\\(expression)来显示自定义变量的内容

## 3.3、自动切换profile
前提：必须在所有服务器上安装Shell Integration，同时为各自服务器配置对应的profile
官方文档： [https://iterm2.com/documentation-automatic-profile-switching.html](https://iterm2.com/documentation-automatic-profile-switching.html)

### 3.3.1、自动连接
在本地.ssh/目录下创建配置文件，比如：192.168.3.32.sh
```shell
cat ~/.ssh/192.168.3.32.sh << 'EOF'
#!/usr/bin/expect -f
set user [user_name]
set host [remote_link]
set port 22
set password [password]
set timeout -1		# 设置超时时间为无限


spawn ssh -p $port $user@$host			# 使用spawn命令启动SSH会话
expect "$user@$host's password:"		# 等待匹配 "*assword:*" 字符串
send "$password\r"					    # 发送密码并添加回车
interact							    # 进入交互模式
EOF
```

然后，打开iTerm2 → Settings → Profiles → add添加一个Profile name
在Command部分，在下拉列表中选择`Command`，并填写配置文件的位置：**expect ~/.ssh/192.168.3.32.sh**
这样就可以通过点击菜单栏中的Profiles，选择刚才配置的Profile就可以实现自动登录远程服务器。

# 4、Q&A
## 4.1、prompt_segment:10: character not in range for SecureCRT

