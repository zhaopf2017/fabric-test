描述一下如何使用测试工具

一，搭建fabric-test的运行环境。

0，安装docker，安装curl，安装docker-compose
1，下载fabric-test源代码。
git clone https://github.com/hyperledger/fabric-test.git
2，fabric-test目录中有若干个关联项目，fabric,fabric-sdk-node,fabric-ca
fabric目录需要通过git的项目关联，下载下来相应的代码。
fabric-sdk-node也需要下载相关的代码。
3，执行pre_setup.sh，安装nodejs及nvm。

二，使用网络共享存储，存取配置文件
1，在ubuntu上安装nfs-common.
服务器端，安装nfs-server,nfs-server服务端
2，配置nfs-server
在服务器上创建
/opt/cryptogen,权限是nobody:nogroup
在nfs-server配置文件中输出这个目录
vi /etc/exports
加入如下内容：
/opt/cryptogen	*(rw,sync,no_subtree_check)
