# 1、介绍
Ansible是一款基于Python开发的开源自动化工具，主要用于配置管理、应用部署、任务自动化和持续交付。它由RedHat公司维护，采用无代理架构（无需在目标主机上安装客户端），通过SSH进行通信，简化了大规模系统的管理。
## 1.1、特点
1. 无代理架构（部署简单）
	a. 直接通过SSH或WinRM管理目标主机，无需安装额外的客户端
2. 基于模块化设计
	a. Ansible本身仅提供框架，真正的批量操作由模块实现。支持的模块成千上万，可解决大部分场景 。就像一个工具箱，根据不同场景，选择最合适自己的工具（模块）。
3. 声明式YAML语法
	a. 使用Playbook定义自动化任务，直观易读，方便完成复杂任务
4. 跨平台支持
	a. 各平台设备都可被Ansible管理
5. 可扩展
	a. 支持API及自定义模块，可通过Python进行扩展
6. 幂等性
	a. 重复执行任务不会导致系统状态异常，确保操作一致性

# 2、安装
## 2.1、安装pipx
文档：
- [https://pipx.pypa.io/stable/#install-pipx](https://pipx.pypa.io/stable/#install-pipx)

```shell
sudo apt update
sudo apt install pipx
pipx ensurepath
sudo pipx ensurepath --global # optional to allow pipx actions with --global argument
```

## 2.2、安装和更新Ansible包
文档：[https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#installation-guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#installation-guide)

```shell
# 安装
pipx install --include-deps ansible

# 更新
pipx upgrade --include-injected ansible
```

# 3、操作
## 3.1、创建多个Linux实例
参见：[创建和启动Linux实例](../Linux/Linux-使用手册.md#2启动Linux实例)

多个实例服务器不需要安装 Ansible，但需要 Python 来运行由 Ansible 生成的 Python 代码。还需要一个能通过 SSH 连接到该服务器并带有交互式 POSIX shell 的用户账号。

## 3.2、常用命令
### 3.2.1、Ansible配置文件
文档：
- [https://docs.ansible.com/projects/ansible/latest/cli/ansible-config.html](https://docs.ansible.com/projects/ansible/latest/cli/ansible-config.html)
- [https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html](https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html)

```shell
ansible-config init --disabled > ansible.cfg #显示默认设置项
ansible-config init --disabled -t all > ansible.cfg #显示所有设置项（包含可用插件）
```

### 3.2.2、Ansible主机清单
文档：
- [https://docs.ansible.com/projects/ansible/latest/cli/ansible-inventory.html](https://docs.ansible.com/projects/ansible/latest/cli/ansible-inventory.html)

```
# 查看层级结构
ansible-inventory --graph

# 验证inventory
ansible-inventory --list -i <inventory>
```

### 3.2.3、Ansible命令运行
```
# 多个服务器上执行模块
ansible <group> -m <module> -i <inventory>
```

## 3.3、常用模块
文档：
- [https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin)

### 3.3.1、file模块
```shell

```

### 3.3.2、


需要深入了解：
Ansible YAML 配置文件
