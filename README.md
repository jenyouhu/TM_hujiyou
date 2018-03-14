# BlockChain-Tendermint
 
这是一个基于Tendermint 共识机制的区块链项目，记录了基于tendermin核心，如何打造自己的区块链项目。
 
非常详细：

构建基于tendermind区块链应用（概念部分）

一、 Tendermint 是什么
Tendermint（以下简称 TM），如果有玩家听说过以太坊（Ethereum），这个什么坊有个分支 Ethermint 就是基于 TM 开发的，反正PO主只懂撸代码不懂炒币，不是很了解这到底是个什么工坊。
好了，先来了解一下 TM 的原理，因为实在没什么可以可视化的 UI 让玩家们一目了然。TM 主要包含两部分：
1.	Tendermint Core：区块链共识引擎。它负责两件事情：节点之间的数据同步有序传输，拜占庭共识机制的实现。
2.	ABCI：区块链应用接口。它被设计成一组有接口规范的协议，目的是可以使用多种语言实现区块链应用逻辑。

构建基于tendermind区块链应用（操作部分）

一、安装git ，可以直接在lunix上用git命令，从github网站下载github上面的项目（CVS版本控制工具）
	yum -y install git
    yum -y instatll mercurial
二、安装golang语言（编程语言）
  1、 由于tendermint 是用golang 语言编写，所以需要安装go语言以及相关编译工具。
  2、 下载 go1.9.3.linux-amd64.tar.gz 按照包，直接解压到你在/etc/profile设置的GOROOT目录 (千万别用yum install golang安装，因为默认的版本是 1.8.3 后续一堆的坑) 
三、设置go语言的相关环境变量
   编辑 /etc/profile 文件，加入如下变量
	export GOROOT=/home/go/go
	export GOBIN=$GOROOT/bin
	export GOPKG=$GOROOT/pkg/tool/linux_amd64
	export GOARCH=amd64
	export GOOS=linux
	export GOPATH=/Golang
	export PATH=$PATH:$GOBIN:$GOPKG:$GOPATH/bin

四、安装glide工具 （go语言用glide 管理依赖库）。 
go get github.com/Masterminds/glide （安装glide，go语言使用glide管理依赖库）
  A、编译 cd github.com/Masterminds/glide 
     make build  
     go build -o glide -ldflags "-X main.version=v0.11.0" glide.go
五、安装tendermint 。 
go get github.com/tendermint/tendermint/cmd/tendermint（安装tendermint源码）
(这个实际比较长，请耐心等待，当然也可以直接从 www.tendermint.com官方网站下载）
在安装 tendermint 源码时，由于无法访问 google , golang.org等网站，导致无法下载
相关库，会报错：例如package golang.org/x/sys/unix: unrecognized import path "golang.org/x/sys/unix" (https fetch: Get https://golang.org/
x/sys/unix?go-get=1: dial tcp 216.239.37.1:443: i/o timeout) 暂时不管。

六、要编译源码，必须按照下面三个工具
  1、 go get github.com/mitchellh/gox   (安装gox 跨平台编译器)
  2、 go get -u github.com/tcnksm/ghr （安装ghr ）
  3、 go get -u gopkg.in/alecthomas/gometalinter.v2 （安装gometalinter.v2 ）

$ glide install
七、执行 gilde install 

在执行 glide install 时，由于无法访问google相关网站，所以先执行下面的步骤，
目的是从 github网站下载与 google.golang.org，golang.org网站对应的关联代码。
1、git clone https://github.com/golang/sys.git       （golang.org/x/sys）
2、git clone https://github.com/golang/text.git        （golang.org/x/text）
3、git clone https://github.com/golang/net.git         （golang.org/x/net）
4、git clone https://gihub.com/golang/crypto.git        (golang.org/x/crypto)
5、git clone https://github.com/golang/tools.git        (golang.org/x/tools)
6、git clone https://github.com/grpc/grpc-go.git      （.google.golang.org/grpc）
7、git clone https://github.com/google/go-genproto.git（google.golang.org/genproto）
cd  $GOPATH/src/github.com/tendermint/tendermint
在执行前，必须先修改 glide.lock 文件，将里面 google,golang.org网站名称及路径 替换成github网站上对应的文件路径，例如：
将  google.golang.org/grpc 替换成 github.com/grpc/grpc-go 
然后，执行 glide install  ,最后一行显示 Replacing existing vendor dependencies 表示执行成功。

八、编译 tendermint 代码
   运行 make ；这个地方坑太大了。。。。各种的错误，让你摸不到头脑，到网上去找，发现坑更多了，没办法，还是自力更生，花费了一番功夫才发现，其中最大的问题出在：/tendermint/tendermint/rpc/grpc/types.pb.go 文件里面，该文件是由第三方软件protobuf自动生成的，其中import了一个库文件 context。但关键这该死的 context 库有“双重身份”，在标准库里面有，在/golang.org/x/net/context"也有，更变态的是，在types.pb.go文件中竟然同时引用了 这两个“context”包里面的方法；为了区别他们的不同，我不得不 加入如下行 ：
import  cotx "golang.org/x/net/context" ，其中别名取成 cotx 也是没办法，为了区分标准库里面的 context包。

             折腾了2天，终于 make all 成功，可执行文件存放在 $GOPATH/src/github.com/tendermint/tendermint/build 下面： 
执行tendermint version ，截止到我发稿是0.16.0版。
 

九、编译基于tendermint的区块链接口 abci，提供了很多语言接口，包括
 （ C++，JavaScript，Java 和 Erlang），本包里面我集成了 C语言接口在
  $GOPATH/src/github.com/tednermint/c-abci目录下。
  编译该接口时，需要在：/etc/ld.so.conf.d/local.conf 文件中增加如下两行：  
$GOPATH/src/github.com/tendermint/c-abci/lib
$GOPATH/src/github.com/tendermint/c-abci/type/protobuf-c
  刷新编辑文件：ldconfig后，直接运行 make 编译，可执行文件存放在
 ~c-abci/demo/bin/c-dummy。下面主要重点说其他语言的abci接口编译
1） 安装必要工具如下：
    确保你的Centos 机器上安装了 autoconf、automake、libtool、curle、make
    g++,unzip ，如果没有 yum install xxxx就好。
   
2） 下载protobuf源码包，解压，编译安装
    地址：https://github.com/google/protobuf/releases
    选择Source code (tar.gz)下载
    tar -zxvf protobuf-3.1.0.tar.gz -C /usr/local/
cd protobuf-3.1.0/
# 如果使用的不是源码，而是release版本 (已经包含gmock和configure脚本)，可以略过这一步
./autogen.sh
# 指定安装路径
./configure --prefix=/xxxxxxxx/protobuf
#编译
make   (需要很长时间，耐心等待 )
# 测试，这一步很耗时间
make check （需要很长时间，耐心等待 ）
make install
# refresh shared library cache.
ldconfig
注意make check这一步会花费比较多的时间

3） 添加环境变量
vi  /etc/profile  加入
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GOPATH/src/github.com/gogo/protobuf/lib
export LIBRARY_PATH=$LIBRARY_PATH:$GOPATH/src/github.com/gogo/protobuf/lib
export PATH=$PATH:$GOPATH/src/github.com/gogo/protobuf/bin
4） 编译查看版本protoc --version
 
5） 由于abci接口用了 gogo 里面的protobuf
  mkdir  gogo_protobuf ，cd gogo_protobuf 
 下载：git clone  https://github.com/gogo/protobuf.git 
 将Makefile 文件中的 下面这行注释掉，因为golang.org网站无法访问，
如果碰到无法访问的类似网站，一律到 github.com网站上去找，替代golang.org网站对应的文件（例如 go get golang.org/x/tools/cmd/benchcmp  需要注释掉）
 编译：make  成功。
    
十 编译 tendermint 下的 abci 
   1)  cd $GOPATH/ github.com/tendermint/abci 
   2)  运行 make  成功后，在当前目录下生产 abci-cli 
   3） 运行 abci-cli version 
 

  至此，断断续续，我已经折腾了 差不多 2天，重要恭喜你成功安装 tendermint 了，
  并且 是源码安装，源码安装的好处就是，当你成为大牛后，你就可以修改源代码。

十一  测试 tendermint , abci-cli是否成功
1） 运行官方例子，
官方文档提供了两个栗子，已经集成在了刚才的安装里：
dummy：一个简单的键值存储区块链应用，使用起来有点像 Redis 或 ElasticSearch。
counter：一个简单的计数器区块链应用，写入区块的数字必须递增，否则将不被区块接受。
这里我们简单介绍一下 dummy 这个栗子，首先启动这个区块链应用，命令行输入：
abci-cli dummy  （显示如下，表示成功）
 
其实上面启动，是作为服务端启动的，其IP地址 和 端口的配置在如下文件中：


然后启动 tendermint ，命令行输入：（作为节点端启动---也就是客户端 ）
tendermint init  （下面的显示表示以前启动过,可以用 tendermint unsafe_reset_all启动，清除数据，并且重新初始化私人秘钥 ）
 
tendermint node  （显示如下表示成功）
 

顺利的话，可以看到 abci 和 tendermint 两个程序连通（Connect）了，并且 tendermint 会像心跳一样每一秒提交一个空区块
tendermint 初始化启动成功后，在 /root/.tendermint 目录下有两个目录 data和config ， 其中 data目录下存放数据， config 存放配置文件：
genesis.json ：    公钥文件
priv_validator.json  私钥文件
addrbook.json      节点地址文件
config.toml         tendermint的配置文件（例如IP地址、端口、协议、数据库类型等）

十二  往区块链里面写实质内容 
 1、命令行输入：
 curl -s 'localhost:46657/broadcast_tx_commit?tx="jenyouhu"'
 curl -s 'localhost:46657/broadcast_tx_commit?tx="name=hujiyou"'
我们的区块链里就有了一个记录为“jenyouhu”的区块和另一个记录为“name=hu ji you”的区块; 
2) 接下来，我们查询一下是否OK 
   如果需要对区块链内容进行查询，命令行输入：
curl -s 'localhost:46657/abci_query?data="name"'
你会发现在 tendermint 节点上显示：
I[03-13|13:06:00.212] ABCIQuery                                    module=rpc path= data=6E616D65 result="log:\"exists\" index:-1 key:\"name\" value:\"hujiyou\" "  表示查询成功了。

curl 是 Linux 常用工具，我们也可以通过网页浏览器直接输入 localhost:46657/abci_query?data="name"


十三  构建我们自己的区块链

十四 tendermint管理发布工具MintNet 安装介绍
  1） MintNet源码安装方式
      下载源码：git clone http://github.com/mininet/mininet
      安装：cd mintnet/util  
            ./install.sh  -a   (全部安装）
     安装报错： MintNet 不支持 CentOS ，于是漫长的流程走下去：
a. yum -y install openssh-server
b, chkconfig sshd on
c, service sshd start
– Disable SELinux to get OVSDB to stasrt
sudo setenforce Permissive
– Modify sudoers secure_path to add /usr/local/bin so the ‘controller’
 which be found. 设置环境变量 : /etc/profile  
secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin

D，修改 install.sh 文件如下：
修改mininet/util/install.sh：
***ADD the following before the line ‘test -e /etc/fedora-release && DIST=”Fedora”‘. Somewhere around line 47.  May differ.
增加如下内容 
test -e /etc/centos-release && DIST="CentOS"
if [ "$DIST" = "CentOS" ]; then
    install='sudo yum -y install'
    remove='sudo yum -y erase'
    pkginst='sudo rpm -ivh'
    # Prereqs for this script
    if ! which lsb_release &> /dev/null; then
        $install redhat-lsb-core
    fi
fi
***修改如下
if ! echo $DIST | egrep 'Ubuntu|Debian|Fedora|CentOS'; then
    echo "Install.sh currently only supports Ubuntu, Debian and Fedora."
    exit 1
fi
  ./install.sh  -a  执行安装 mintNet 成功
但在运行 mn 是，部分报错，那是因为需要安装 openvswitch . 见下面的步骤。

E，还需要安装 openvswitch 
1, 安装依赖包     yum -y install make gcc openssl-devel autoconf automake rpm-build redhat-rpm-config  
yum -y install python-devel openssl-devel kernel-devel kernel-debug-devel libtool wget  
2.预处理：
[plain] view plain copy
1.	mkdir -p ~/rpmbuild/SOURCES  
2.	wget http://openvswitch.org/releases/openvswitch-2.5.0.tar.gz  
3.	cp openvswitch-2.5.0.tar.gz ~/rpmbuild/SOURCES/  
4.	tar xfz openvswitch-2.5.0.tar.gz  
5.	sed 's/openvswitch-kmod, //g' openvswitch-2.5.0/rhel/openvswitch.spec > openvswitch-2.5.0/rhel/openvswitch_no_kmod.spec  

3.构建RPM包：

[plain] view plain copy
1.	rpmbuild -bb --nocheck ~/rpmbuild/SOURCES /openvswitch-2.5.0/rhel/openvswitch_no_kmod.spec  

4.安装：

[plain] view plain copy
1.	yum localinstall ~/rpmbuild/RPMS/x86_64/openvswitch-2.5.0-1.x86_64.rpm  

5.启动相关服务：
[plain] view plain copy
1.	systemctl start openvswitch.service  
2.	

最后： 运行 mn --test pingall  成功
 
十五 tendermint管理发布工具MintNet 使用介绍
待续

 
