
操作环境在 centos7 

### 编译各组件
编译tikv，这个最麻烦耗时先来
主要步骤如下：
 ```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh 
yum install git -y
yum group install "Development Tools"
wget https://github.com/Kitware/CMake/releases/download/v3.14.5/cmake-3.14.5-Linux-x86_64.tar.gz
tar -zxvf cmake-3.14.5-Linux-x86_64.tar.gz 
git clone  https://github.com/tikv/tikv.git

export CMAKE_HOME=/root/cmake-3.14.5-Linux-x86_64
export PATH=$PATH:/usr/local/go/bin:$HOME/.cargo/bin:$CMAKE_HOME/bin

make build
```

- 安装golang
```
 wget https://golang.google.cn/dl/go1.15.linux-amd64.tar.gz
 tar -zxvf go1.15.linux-amd64.tar.gz
 cp -r go /usr/local/
 ```
- 编译tidb,pd
```
 git clone https://github.com/pingcap/pd.git
 git clone https://github.com/pingcap/tidb.git
```
 分别切到两者目录，make 即可。

### 启动集群
```
./bin/pd-server --name=pd1 --data-dir=pd1 --client-urls="http://127.0.0.1:2379" --peer-urls="http://127.0.0.1:2380"  --initial-cluster="pd1=http://127.0.0.1:2380" --log-file=pd1.log
```
```
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:20160" --data-dir=tikv1 --log-file=tikv1.log
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:20161" --data-dir=tikv2 --log-file=tikv2.log
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:20162" --data-dir=tikv3 --log-file=tikv3.log
```
```
./bin/tidb-server --store=tikv --path="127.0.0.1:2379"
```
开始时不知怎么弄的，tikv报错说id重复了,pd里已经有个20160 id是1了,不能在用20160了,就换了20170.结果tikv是跑起来了，但是tidb报错说连不上20160，不知为什么
pd里有个20160还删不掉，虽然状态是down但是tidb还是连他而且只连他连不上就挂。
最后把pd的datadir清空了,tikv 又报连错pd了，又把tikv的datadir也清空了，就都好了。

## 另外pd,tikv的启动方法都很好找，这个tidb 的启动方法则相当隐蔽，一直纳闷怎么通信的和其他俩，文档都是tiup，ansible这些高级方法相当曲折，看了代码也没上看出来那个配置项是配ip端口啥的，还好看了docker部署方法，原来是path 这个参数，这个神似文件路径的名字竟然是 tikv server那里的 pd-endpoints,真是人不貌相,变量不能望文生义!

### 修改tidb 使其开始事物的时候打印 Hello Transaction.
有两个地方可以选择，加一句 print()即可。

-- 修改session/txn.go init 方法.

```
8/16 22:49:26.749 +08:00] [INFO] [domain.go:377] ["full load and reset schema validator"]
Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction![2020/08/16 22:49:26.756 +08:00] [INFO] [server.go:235] ["server is running MySQL protocol"] [addr=0.0.0.0:4000]
[2020/08/16 22:49:26.756 +08:00] [INFO] [http_status.go:82] ["for status and metrics report"] ["listening on addr"=0.0.0.0:10080]
[2020/08/16 22:49:26.759 +08:00] [INFO] [domain.go:1094] ["init stats info time"] ["take time"=2.681006ms]
Hello Transaction!
```



-- 修改session/txn.go getTxnFuture 方法

```
[2020/08/16 22:54:32.381 +08:00] [INFO] [domain.go:377] ["full load and reset schema validator"]
Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction!Hello Transaction![2020/08/16 22:54:32.389 +08:00] [INFO] [server.go:235] ["server is running MySQL protocol"] [addr=0.0.0.0:4000]
[2020/08/16 22:54:32.389 +08:00] [INFO] [http_status.go:82] ["for status and metrics report"] ["listening on addr"=0.0.0.0:10080]
Hello Transaction![2020/08/16 22:54:32.391 +08:00] [INFO] [domain.go:1094] ["init stats info time"] ["take time"=2.437822ms]
Hello Transaction!
```
