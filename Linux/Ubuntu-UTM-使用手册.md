官方文档：[https://docs.getutm.app/](https://docs.getutm.app/)

# 1、NAT模式下无法访问外网

在UTM中NAT模式就是设置为“Shared Network”，但是这种模式无法在系统中访问外网。
把网络模式设置为“Emulated VLAN”可以解决无法访问外网的问题。
不过，如果想要使用SSH从外部访问该系统，需要通过[端口转发](https://docs.getutm.app/settings-qemu/devices/network/port-forwarding/)的方式来实现。
主机端口 2222 --> 客户机(Ubuntu系统)端口 22
然后在Mac上通过命令 ssh -p 2222 @localhost 进入该系统，如下：
```shell
ssh -p 2222 yinxiao@localhost
```
如果还是无法SSH，查看是否客户机没有安装openssh-server，或者是否22端口被防火墙禁止，如下：
```shell
sudo ss -tnlp | grep ':22' #确认监听22端口 

#开通防火墙22端口
sudo ufw status
sudo ufw allow 22/tcp
```
还有一种可能的情况，主机端口转发是否生效，如下：
```shell
nc -vz 127.0.0.1 2222
```
