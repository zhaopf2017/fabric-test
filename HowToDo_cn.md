描述一下如何使用测试工具

一，搭建fabric-test的运行环境。

1，安装docker，安装curl，安装docker-compose

2，下载fabric-test源代码。

git clone https://github.com/hyperledger/fabric-test.git

3，fabric-test目录中有若干个关联项目，fabric,fabric-sdk-node,fabric-ca

fabric目录需要通过git的项目关联，下载下来相应的代码。

fabric-sdk-node也需要下载相关的代码。

4，执行pre_setup.sh，安装nodejs及nvm。

5，复制fabric-test/tools/PTE及NL到fabric-test/fabric-sdk-node/test目录。

6，在fabric-test/fabric-sdk-node中进行npm install，安装依赖包。

7，下面的操作主要都在

/opt/gopath/src/github.com/hyperledger/fabric-test/fabric-sdk-node/test/PTE目录下。



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

在fabric_network服务器启动fabric网络。

进入到/opt/gopath/src/github.com/hyperledger/fabric-test/fabric-sdk-node/test/PTE

cd CITest/CISFiles/scripts

./nt_driver.sh -n -c -p 

这个过程中会产生配置文件、证书文件、初始块文件、配置通道文件。这些文件会放置在

/opt/gopath/src/github.com/hyperledger/fabric-test/fabric/common/tools/cryptogen目录下。

在CITest目录中我们有许多测试案例可以操作。

需要注意的是，我们在fabric-network这台主机上，启动了fabric网络，可以完成创建通道、加入通道、安装链码、实例化链码的操作。




五，配置服务器之间的免密码登陆

ssh-copy-id user@fabric_server

登陆后，sudo到root用户，cat /home/user/.ssh/authorized_keys >>/root/.ssh/authorized_keys

同理，实现pte_controller到pte_driver1和pte_driver2的免密码登陆。因为要远程操作这两个节点。

六，部署PTE远程测试

我们登陆到pte-controller这台主机，然后切换到 /opt/gopath/src/github.com/hyperledger/fabric-test/fabric-sdk-node/test/PTE目录下

pte-ctlr.sh就是用来做远程测试的。

ctlrInputs目录下有一个PTECtlr.txt 配置文件，我们要修改的是这里

root@s78:/opt/gopath/src/github.com/hyperledger/fabric-test/fabric-sdk-node/test/PTE/ctlrInputs# vi PTECtlr.txt 

driver=ctlr root@pte-driver1	ctlrInputs/pteHost11-samplecc-q.sh
driver=ctlr root@pte-driver2	ctlrInputs/pteHost11-samplecc-i.sh

针对pteHost11-samplecc-q.sh的修改：

./test_driver.sh -t FAB-3989-4i-TLS -b $tStart &

将这里的FAB-3989-4i-TLS改为你要启动的那个测试用例编号







