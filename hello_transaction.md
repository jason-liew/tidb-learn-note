
操作环境在 centos7 

- 编译tikv，这个最麻烦耗时先来
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
 ```
- 编译tidb,pd
-- git clone https://github.com/pingcap/pd.git
-- git clone https://github.com/pingcap/tidb.git

-- 分别切到两者目录，make 即可。


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
