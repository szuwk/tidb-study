
## 题目：本地下载TiDB，TiKV，PD源代码，改写源代码并编译部署TiDB/PD/TiKV,在TiDB启动事务时， 会打印一个“hello transaction”的日志

## 源码准备
 ```
   https://github.com/wooxcoo-taotao/tidb
   https://github.com/wooxcoo-taotao/tikv
   https://github.com/wooxcoo-taotao/pd
 ```
## 环境
  * apt-get install  cmake， gcc, etc.
  
  * golang
  ```
  wget https://golang.org/dl/go1.13.12.linux-amd64.tar.gz
  tar -zxvf go1.13.12.linux-amd64.tar.gz -C /usr/local/
  
  export GOPATH=/data/tidb 
  export GOROOT=/usr/local/go
  export PATH=$PATH:$GOROOT/bin
  ```
  
  * tidb 路径
  mkdir -p /data/tidb/src/github.com/pingcap
  
## 启动环境：
   
## .pd 
    ```
    ./bin/pd-server --data-dir=pd --log-file=pd.log &
    ```
## tikv
  ```
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv --log-file=tikv.log &
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv1 --log-file=tikv.log --addr=127.0.0.1:20162 
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv2 --log-file=tikv.log --addr=127.0.0.1:20164
   ```
## tidb
   ```
   ./bin/tidb-server --store=tikv --path="127.0.0.1:2379" --log-file=tidb.log &
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
  * tidb/server/conn.go      
## 重新编译，重新运行 tidb-server
 ```
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

 ```
## tidb-server 日志中出现 “hello transaction”
    ```
   wk@env-01.dev:/data/tidb [TESTING]$ tail -f tidb.log
     [2020/08/16 01:27:54.509 +08:00] [INFO] [gc_worker.go:1476] ["[gc worker] sent safe point to PD"] [uuid=5cfcbbaa8ec0002] ["safe point"=418778126558167040]
     [2020/08/16 01:28:28.982 +08:00] [INFO] [session.go:1502] ["NewTxn() inside a transaction auto commit"] [conn=2] [schemaVersion=24] [txnStartTS=418778279060439041]
     [2020/08/16 01:28:28.982 +08:00] [INFO] [conn.go:828] ["hello transaction"] [conn=2]
    ```
  
