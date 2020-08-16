# 第一期作业
## 题目描述：
### 本地下载TiDB，TiKV，PD源代码，改写源代码并编译部署以下环境：
* 1. 1 TiDB
* 2. 1 PD
* 3. 3 TiKV

### 改写后
* 使得TiDB启动事务时， 会打印一个“hello transaction”的日志

## 资源：
  ### 公司虚拟机
   * linux version
   ```
   liuping@env-01.dev:/data/tidb [TESTING]$ uname -a
   Linux env-01 4.15.0-29-generic #31-Ubuntu SMP Tue Jul 17 15:39:52 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
   ```
   * cpu
   ```
   liuping@env-01.dev:/data/tidb [TESTING]$ cat /proc/cpuinfo | grep "processor" | wc -l
   96
   ```
   * mem
   ```
   liuping@env-01.dev:/data/tidb [TESTING]$ free -h
              total        used        free      shared  buff/cache   available
Mem:           187G         21G        128G         20M         38G        164G
Swap:          952M         15M        937M
```

## 源码准备
 ### fork 代码到个人repo
 ```
   https://github.com/wooxcoo-taotao/tidb
   https://github.com/wooxcoo-taotao/tikv
   https://github.com/wooxcoo-taotao/pd
 ```
  * 发现公司虚拟机 下github 超级慢
  * 代码拉到公司 git repo； 虚拟机从内部仓库
 
  * 代码拉到公司 git repo； 虚拟机从内部git git 
  * 代码拉到公司 git repo； 虚拟机从内部git clone 
  
## 编译环境准备
  * 网上搜索文章，参考：https://www.pianshen.com/article/7833964035/
  ### 基本环境
  * apt-get install  cmake， gcc, etc.
  
  * golang
  ```
  wget https://golang.org/dl/go1.13.12.linux-amd64.tar.gz
  tar -zxvf go1.13.12.linux-amd64.tar.gz -C /usr/local/
  
  export GOPATH=/data/tidb 
  export GOROOT=/usr/local/go
  export PATH=$PATH:$GOROOT/bin
  ```
  * rust
  ```
  curl https://sh.rustup.rs -sSf | sh -s
  rustup override set nightly-2018-01-12
  cargo +nightly-2018-01-12 install rustfmt-nightly --version 0.3.4 --force
  ```
  
 ## build & try
  * 先不修改代码， 尝试正常编译，搭建环境，试用一下
  * tidb 路径要求
  mkdir -p /data/tidb/src/github.com/pingcap
  
  ### build 没啥好说，一路执行
  ### 启动环境：
   * 资源有限， 全部启动在一台机器上， 
   
 #### .pd 
    ```
    ./bin/pd-server --data-dir=pd --log-file=pd.log &
    ```
 #### tikv
  * 要求启动三台tikv， 开始报错，地址重复， 存储位置重复，网上找资料，增加启动参数，分别指定端口，存贮位置后解决；
  * 参考：https://www.bookstack.cn/read/tidb-v3.0/reference-configuration-tikv-server-configuration.md
  ```
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv --log-file=tikv.log &
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv1 --log-file=tikv.log --addr=127.0.0.1:20162 
    ./bin/tikv-server --pd="127.0.0.1:2379" --data-dir=tikv2 --log-file=tikv.log --addr=127.0.0.1:20164
   ```
 #### tidb
   ```
   ./bin/tidb-server --store=tikv --path="127.0.0.1:2379" --log-file=tidb.log &
   ```
   
 ### mysql 链接试用
  ```
  liuping@env-01.dev:/data/tidb/src/github.com/pingcap/tidb [TESTING]$ mysql -h 127.0.0.1 -u root -P 4000
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.25-TiDB-cfe950d-dirty TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql>
mysql> 
mysql> select * runoob_transaction_test
    -> ;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use line 1 column 32 near "runoob_transaction_test"
mysql> show tables;
ERROR 1046 (3D000): No database selected
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

mysql>
mysql>
 ```
 
 ### 改源码重新编译
  * 起初毫无头绪，不知到看哪里
  * 参考一下源码简要：https://pingcap.com/blog-cn/tidb-source-code-reading-2/
  * 大概了解tidb sql 执行流程后，找到加log 最简单粗暴的地方， 是命令统计那里
    tidb/server/conn.go
      * dispatcher->addMetrics
      * logutil.Logge 日志工具需要试用 ctx， 很粗暴的给 addMetrics 函数加上 ctx 参数；
      * 对 go 的日志不了解， 先这么做了；这样不好
      
  change commit ：  https://github.com/wooxcoo-taotao/tidb/commit/bd427fe9413308f8db46b5521273ce613f41991e
 
 #### 重新编译，重新运行 tidb-server
   * mysql 客户端执行 开始事务：  begin
 ```
 mysql>
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

 ```
 #### tidb-server 日志中出现 “hello transaction”
    ```
    liuping@env-01.dev:/data/tidb [TESTING]$ tail -f tidb.log
     [2020/08/16 01:27:54.509 +08:00] [INFO] [gc_worker.go:1476] ["[gc worker] sent safe point to PD"] [uuid=5cfcbbaa8ec0002] ["safe point"=418778126558167040]

     [2020/08/16 01:28:28.982 +08:00] [INFO] [session.go:1502] ["NewTxn() inside a transaction auto commit"] [conn=2] [schemaVersion=24] [txnStartTS=418778279060439041]
     [2020/08/16 01:28:28.982 +08:00] [INFO] [conn.go:828] ["hello transaction"] [conn=2]
    ```
    
  # 作业完成
  
