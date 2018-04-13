描述一下如何使用测试工具

一，搭建fabric-test的运行环境。

0，安装docker，安装curl，安装docker-compose

1，下载fabric-test源代码。

git clone https://github.com/hyperledger/fabric-test.git

2，fabric-test目录中有若干个关联项目，fabric,fabric-sdk-node,fabric-ca

fabric目录需要通过git的项目关联，下载下来相应的代码。
fabric-sdk-node也需要下载相关的代码。

3，执行pre_setup.sh，安装nodejs及nvm。


二，测试系统部署
PTE测试：性能测试
172.16.10.120     fabric_network 运行一个fabric网络（可以networkLanch.sh定制网络拓扑）
172.16.10.78      pte_driver1
172.16.10.76      pte_dirver2
172.16.10.12      pte_ctrl  远程控制主机

这四台主机都需要安装fabric-test。



三，使用网络共享存储，存取配置文件

1，在ubuntu上安装nfs-common.
服务器端，安装nfs-server,nfs-server服务端

2，配置nfs-server
在服务器fabric_network上创建
/opt/cryptogen,权限是nobody:nogroup
在nfs-server配置文件中输出这个目录
vi /etc/exports
加入如下内容：
/opt/cryptogen	*(rw,sync,no_subtree_check)

3，加载nfs服务器上的资源到本地
cd /opt/gopath/src/github.com/hyperledger/fabric-test/fabric/common/tools/
将原来的cryptogen重新命名
新建cryptogen目录
mount -t nfs fabric_network:/opt/cryptogen cryptogen

四，fabric网络启动
在fabric服务器上（172.16.10.120）


五，配置服务器之间的免密码登陆
ssh-copy-id user@fabric_server

