
## 题目：本地下载TiDB，TiKV，PD源代码，改写源代码并编译部署TiDB/PD/TiKV,在TiDB启动事务时， 会打印一个“hello transaction”的日志

## 环境准备
  * apt-get install  cmake.
  
  * golang
  
  * tidb 路径
  mkdir -p /root/data/tidb/src/github.com/pingcap
## 源码准备
tikv
 ```
  cd /root/data/tidb/src/github.com/pingcap
  git clone https://github.com/tikv/tikv.git
  cd tikv
  make
 ```
tidb
 ```
 cd /root/data/tidb/src/github.com/pingcap
 git clone https://github.com/pingcap/tidb.git
 cd tidb
 export GOPROXY=https://mirrors.aliyun.com/goproxy/
 make
 ```
 pd
 ```
 cd /root/data/tidb/src/github.com/pingcap
 git clone https://github.com/pingcap/pd.git
 cd pd
 export GOPROXY=https://mirrors.aliyun.com/goproxy/
 make
 ```
 ## 启动环境：
## pd 
    ```
    bin/pd-server -L debug -name first &
    ```
## tikv
  ```
    ./target/release/tikv-server -f tikv1/log.txt --data-dir ./tikv1 --pd-endpoints "127.0.0.1:2379" -A "127.0.0.1:20160" 2>&1 1>tikv1/log.txt &
    ./target/release/tikv-server -f tikv2/log.txt --data-dir ./tikv2 --pd-endpoints "127.0.0.1:2379" -A "127.0.0.1:20161" 2>&1 1>tikv2/log.txt & 
    ./target/release/tikv-server -f tikv3/log.txt --data-dir ./tikv3 --pd-endpoints "127.0.0.1:2379" -A "127.0.0.1:20162" 2>&1 1>tikv3/log.txt &
   ```
## tidb
   ```
   bin/tidb-server -host 127.0.0.1 -P 3306 -store tikv --path="127.0.0.1:2379" -L debug &
   ```
   
 ### mysql 链接试用
  ```
mysql> use test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
mysql> show tables;
+-------------------------+
| Tables_in_test          |
+-------------------------+
| runoob_transaction_test |
+-------------------------+
1 row in set (0.00 sec)

mysql> select * from runoob_transaction_test
    -> ;
+------+
| id   |
+------+
|    5 |
|    6 |
+------+
2 rows in set (0.01 sec)
 ```
 
## 修改源码重新编译
 在tidb/store/tikv package中的kv.go中，func (s *tikvStore) BeginWithStartTS(startTS uint64) (kv.Transaction, error)中加入code:
logutil.BgLogger().Info("hello transaction") 

## 重新编译，重新运行 tidb-server
 ```
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
 ```
## tidb-server 日志中出现 “hello transaction”
    ```
[2020/08/17 21:56:48.091 +08:00] [INFO] [gc_worker.go:1189] ["[gc worker] finish resolve locks with physical scan locks"] [uuid=5d01bcd8bf40006] [safePoint=418865429360672768] [takes=8.729752ms]
[2020/08/17 21:56:48.091 +08:00] [INFO] [gc_worker.go:1297] ["[gc worker] removing lock observers"] [uuid=5d01bcd8bf40006] [safePoint=418865429360672768]
[2020/08/17 21:56:50.942 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/17 21:56:51.019 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/17 21:56:51.021 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/17 21:56:51.023 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/17 21:56:53.942 +08:00] [INFO] [kv.go:291] ["hello transaction"]

    ```
  
