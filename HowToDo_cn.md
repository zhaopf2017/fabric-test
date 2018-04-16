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


pte-driver1上，你需要修改的是PTE/CITest/CISCFiles/config-chan*-TLS.json （两个文件），将其中的localhost指向fabric-network所在的地址。



七，核心文件说明

1，networkLanch.sh

该文件位于fabric-test/tools/NL目录中，可以启动或者停止fabric网络。

./networkLauncher.sh [opt] [value]
 
    -a: network action [up|down], default=up
 
    -x: number of ca, default=0
    
    -d: ledger database type, default=goleveldb
    
    -f: profile string, default=test
    
    -h: hash type, default=SHA2
    
    -k: number of kafka, default=0
    
    -z: number of zookeepers, default=0
    
    -n: number of channels, default=1
    
    -o: number of orderers, default=1
    
    -p: number of peers per organization, default=1
    
    -r: number of organizations, default=1
    
    -s: security type, default=256
    
    -t: ledger orderer service type [solo|kafka], default=solo
    
    -w: host ip, default=0.0.0.0
    
    -l: core logging level [CRITICAL|ERROR|WARNING|NOTICE|INFO|DEBUG], default=ERROR
    
    -q: orderer logging level [CRITICAL|ERROR|WARNING|NOTICE|INFO|DEBUG], default=ERROR
    
    -c: batch timeout, default=2s
    
    -B: batch size, default=10
    
    -F: local MSP base directory, default=/opt/gopath/src/github.com/hyperledger/fabric-test/fabric/common/tools/cryptogen/
    
    -G: src MSP base directory, default=/opt/hyperledger/fabric/msp/crypto-config
    
    -S: TLS enablement [enabled|disabled], default=disabled
    
    -C: company name, default=example.com 
    
    在CITest的测试用例中默认的网络参数是:
   
   ./networkLauncher.sh -o 3 -x 2 -r 2 -p 2 -k 4 -z 3 -n 2 -t kafka -f test -w localhost -S enabled -l INFO -B 500
   
   也就是三个orderer，两个ca，两个组织机构，每个组织默认peer个数为2，kafka为4个，zookeeper为3个，共识模式为kafka,区块默认高度500.
   
   2，test_driver.sh
   
   存放在PTE/CITest/scripts目录下，这个文件是远程调用的核心。接收到参数是测试用例的编号及起始的时间戳。

    ./test_driver.sh [opt] [values]"
   
       -e: install sdk packages, default=no"
   
       -n: create network, default=no"
   
       -m: directory where test_nl.sh, preconfig, chaincode to be used to create network, default=scripts"
   
       -p: preconfigure creation/join channels, default=no"
   
       -s: synchup peer ledgers, recommended when network brought up, default=no"
   
       -c: chaincode to be installed and instantiated [all|chaincode], default=no"
   
       -t [value1 value2 value3 ...]: test cases to be executed"
   
       -b [value]: test cases starting time"
       
   我们需要在fabric—network上将这个网络起来，而不用制定-t参数
   
   在PTE引擎主机上，不要创建网络，不要对通道进行操作。
   
   3,pte_dirver.sh
   在PTE目录下，主要根据传入参数，去做一些操作。会被pte_mgr掉用。
   
   4,SCFiles目录下的config文件
   在PTE的SCFiles目录下有，在CITest/CISCFiles也有，都是用指向fabric网络的配置信息。


 
 


