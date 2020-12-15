# 部署文档



## 架构说明

Capture 部署到网关服务器，或者web服务器，进行访问流量抓取

CopyServer部署到一台服务器作为复制节点，会接收Capture 抓取的流量

Sender部署到一台服务器作为流量发送节点，放大流量倍数发送到测试环境 

Senderpool是sender的升级版，支持定时、并发发送峰值流量



## 部署步骤

**以Centos7为例, 语言版本python2**

### 安装系统依赖

```bash
yum -y install flex.x86_64
yum -y install bison.x86_64
yum -y install byacc.x86_64
yum -y install python-devel.x86_64
yum -y install gcc unzip
```

### 安装lib依赖库

```bash
cd Meteor/lib
tar -xvf libpcap-1.7.4.tar.gz
cd libpcap-1.7.4/
./configure && make install
unzip pypcap-master.zip
cd pypcap-master/
python setup.py install
unzip dpkt-master.zip
cd dpkt-master/
python setup.py install

ln -s /usr/local/lib/libpcap.so.1  /usr/lib/
```



### 启动capture

#### 节点ip

192.168.1.1

#### 配置文件

```bash
[root@k8s-node2 capture]# cat capture.conf 
[filter]
# 过滤参数支持uri 和端口过滤
uri=
host=
[config]
# capture节点网卡
nc=ens33
# capture节点抓取端口
capture_port=80
# copyserver节点ip地址
copy_ip=192.168.1.2
# 端口默认65533
copy_port=65533
model=
[exclude]
# 不抓取的源ip
ip=


```

#### 启动命令

```bash
python start_capture.py
```



### 启动copyserve

##### 节点ip

192.168.1.2

##### 配置文件

```bash
[root@k8s-node2 copyserver]# cat copy.conf 
[filter]
# 支持url过滤参数
uri=
[config]
# 指定网卡
nc=ens33
# 端口默认65533
port=
# 默认使用本机IP地址发送udp广播，支持配置多个sendserver接收ip地址，为空就广播发送，不为空就是发送给指定ip
ip=

```

##### 启动命令

```bash
python start_copy.py
```



### 启动sendserve

##### 节点ip

192.168.1.3 （同一网段即可，只要能接收udp广播流量， 可以启动多个）

##### 配置文件

```bash
[root@k8s-node1 sender]# cat send.conf 
[filter]
# 过滤参数
uri=
[config]
# 网卡
nc=ens33
# 修改访问ip，这里配置测试环境地址即可把流量发送到测试环境
resent_ip=192.168.1.11
# 放大流量倍数， 默认5
count=5
# 发送端口， 默认65533
port=

```

##### 启动命令

```bash
python start_sender.py
```



**ps:流量不存储，直接转发**