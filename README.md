# redis-tool
Redis Cluster Daily Maintenance Tool/Redis集群日常运维工具

## 工具简介

**redis-tool**基于原生的redis-cli客户端工具来进行Redis集群的监控、配置、问题分析等运维管理，能够极大降低Redis cluster集群的运维成本。同时作为脚本化工具，下载即可用，即使对于Redis初学者，也能够快速掌握集群的运行状况，完成集群配置管理、性能问题排查，具备Redis集群的基本运维能力。

工具主要面向日常运维管理中的常见工作，提升运维效率，简化操作复杂度：

- **集群监控**：能够获取集群节点的键值数、内存使用量、CPU使用率、客户端连接数、请求响应时延等信息，以表格形式输出，并给出异常告警提示
- **配置管理**：可以按节点属性来统一查询或修改运行参数，并且支持对节点运行参数进行差异对比分析
- **问题分析**：具备慢命令日志查询、热点key分析、TOP命令跟踪、KEY前缀统计等问题分析能力

## 安装部署

redis-tool工具使用shell脚本实现，下载到具有redis-cli工具的主机上即可使用（通常可部署到redis服务主机上）。

```bash
# 1. 从github下载redis-tool工具
# 方法一：使用git下载
$ git clone https://github.com/iwhalecloud-platform/redis-tool.git
# 进入下载目录
$ cd redis-tool
# 方法二：使用wget下载
$ wget https://github.com/iwhalecloud-platform/redis-tool/archive/refs/heads/main.zip
# 解压工具包&修改目录名
$ unzip main.zip && mv redis-tool-main redis-tool
# 进入下载目录
$ cd redis-tool

# 2. 设置REDIS_HOME环境变量为redis-cli工具所在目录（如果PATH环境变量中包含该目录，则该步可省略）
$ echo "export REDIS_HOME=/path/to/redis-cli/" >> ~/.bashrc
$ source ~/.bashrc
```

## 使用帮助

直接运行**redis-tool**可查看到使用帮助：

```bash
$ ./redis-tool
Usage: redis-tool [OPTIONS] [command]
OPTIONS:
   -h <IP>:       Redis集群中某个节点IP (默认值: 127.0.0.1).
   -p <port>:     Redis集群中某个节点端口 (默认值: 6379).
   -i <nodefile>: Redis集群节点信息文件(适配codis等基于PROXY构建的集群模式).
                  文件格式(每行一个分片)：<slots1> <MasterAddr> [SlaveAddr] [SlaveAddr]
   -a <password>: Redis访问密码，也可使用'REDISCLI_AUTH'环境变量来传入.
   -c <count>:    1. TOP10命令统计滚动展示次数(默认无限次, 'moni'使用).
                  2. 从Redis节点中获取的慢命令记录数(默认值: 100, 'slowlog'使用).
                  3. 从Redis节点中随机获取的KEY名称数量(默认值: 100000, 'keys'使用).
   -d <delay>:    TOP10命令统计滚动展示等待间隔(单位: 秒, 默认值: 3, 'moni'使用).
   -t <time>:     实时命令监控跟踪运行时长(单位: 秒, 默认值: 10, 'trace'使用).
   -s:            只处理指定Redis节点('moni' & 'slowlog'使用).
   -l:            TOP10命令统计滚动展示包含命令执行平均处理耗时('moni'使用).
   -f <file>:     1. 基于监控的命令详情文件进行命令监控跟踪处理('trace'使用).
                  2. 基于指定的KEY名称列表文件进行前缀统计('keys'使用).
   -L <level>:    按KEY前缀统计层级(默认值: 3, 'keys'使用).
   -H:            基于监控的命令详情文件进行热点KEY访问分析('trace'使用).
   -C:            基于监控的命令详情文件按客户端IP进行命令统计('trace'使用).
   -r:            按裸输据形式输出集群节点监控指标,便于存储或上报至第三方监控系统 ('nodes'使用).
   -k <key>:      要查询或修改的Redis运行参数名('config'使用).
   -v <value>:    要修改的Redis运行参数值('config'使用).
   -M:            只访问集群中主节点的运行参数 ('config'使用).
   -S:            只访问集群中从节点的运行参数 ('config'使用).
   -w:            Redis节点运行参数修改后需要重写配置文件('config'使用).
command:
   nodes:   集群节点状态监控 (默认执行)
   keys:    按KEY前缀层级进行统计分析
   moni:    TOP10命令周期性滚动统计
   trace:   实时命令监控跟踪 & 热点KEY访问分析
   slowlog: 集群慢命令日志查询
   config:  查询/修改集群运行参数 & 集群运行参数差异检查
```



## 使用示例

<span style='color:red'>**注意事项**：</span>

1. 请提前设置<span style='color:red'>REDIS_HOME</span>环境变量为redis-cli工具所在目录（如果`PATH`环境变量中包含该目录，可省略）
2. 强烈建议使用<span style='color:red'>REDISCLI_AUTH</span>环境变量来传入Redis访问密码，提升安全性

### Redis集群监控

使用`nodes`子命令可以汇总集群各节点的主要指标信息，并按照主从关系进行级联展示（当未指定子命令时，默认执行Redis集群监控子命令），可以针对Redis Cluster、主从Redis、自定义集群(codis、ctgcache)等形式集群的监控。

#### Redis Cluster集群

使用`-h`和`-p`参数传入集群中任一节点的IP和PORT，即可获取整个集群节点的监控指标信息：

```bash
$ ./redis-tool -h 172.16.18.80 -p 6380 nodes
================================================== 2022-04-28 18:44:02 ==================================================
SERVER                   STATUS ROLE    VERSION  UPTIME(s) KEYS      USEMEM   %CPU   CLIENTS OPS    RTT(us)   SLOTS 
-------------------------------------------------------------------------------------------------------------------------
172.16.18.80:6379        OK     master  6.2.6    120542    55        21.56M   0.2    1       2      87        0-5460
 |-172.16.18.81:6380     OK     slave   6.2.6    120538    55        21.56M   0.2    2       0      7702     
172.16.18.81:6379        OK     master  6.2.6    120542    56        21.66M   12.7   2       1317   179       5461-10922
 |-172.16.18.82:6380     OK     slave   6.2.6    120538    56        21.56M   0.2    2       2      424      
172.16.18.82:6379        OK     master  6.2.6    120542    45        21.56M   0.3    1       0      151       10923-16383
 |-172.16.18.80:6380     OK     slave   6.2.6    120539    45        21.57M   0.2    2       0      84       
_________________________________________________________________________________________________________________________
SUM(master/all): NODE=3/6 KEY=156/312 MEM=64.8M/129.5M CLI=4/10 OPS=1319/1321 CPU=13.2/13.8
=========================================================================================================================
```

#### 主从Redis

同样使用`-h`和`-p`参数传入主从Redis中任一节点的IP和PORT，即可获取主从节点的监控指标信息：

```bash
$ redis-tool -h 172.16.18.80 -p 7770
================================================== 2022-10-12 12:26:54 ==================================================
SERVER                   STATUS ROLE    VERSION  UPTIME(s) KEYS      USEMEM   %CPU   CLIENTS OPS    RTT(us)   SLOTS 
-------------------------------------------------------------------------------------------------------------------------
172.16.18.80:7770        OK     master  6.2.6    251       3         1.87M    0.3    1       1      193       single-M
 |-172.16.18.80:7772     OK     slave   6.2.6    237       3         1.85M    0.3    2       0      140       single-S
 |-172.16.18.80:7771     OK     slave   6.2.6    243       3         1.85M    0.2    2       2      119       single-S
_________________________________________________________________________________________________________________________
SUM(master/all): NODE=1/3 KEY=3/9 MEM=1.9M/5.6M CLI=1/5 OPS=1/3 CPU=0.3/0.8
=========================================================================================================================
```

#### 自定义集群

使用`-i`参数传入自定义Redis集群的节点信息文件（适配codis等基于PROXY构建的集群模式）

文件格式（每行一个分片）：`<slots1> <MasterAddr> [SlaveAddr] [SlaveAddr]`

```bash
# 例如：自定义Redis集群中共有4个分片，每个分片包含1主1从节点 
$ cat cluster93.conf
1 172.16.84.94:19100 172.16.84.93:19100
2 172.16.84.94:19101 172.16.84.93:19101
3 172.16.84.93:19102 172.16.84.94:19102
4 172.16.84.93:19103 172.16.84.94:19103
$ redis-tool -i cluster93.conf
================================================== 2022-10-12 09:16:10 ==================================================
SERVER                   STATUS ROLE    VERSION  UPTIME(s) KEYS      USEMEM   %CPU   CLIENTS OPS    RTT(us)   SLOTS 
-------------------------------------------------------------------------------------------------------------------------
172.16.84.93:19102       OK     master  2.7.0    5270208   773329    578.64M  0.4    314     48     637       3
 |-172.16.84.94:19102    OK     slave   2.7.0    5270684   773337    572.38M  0.2    2       6      1312     
172.16.84.93:19103       OK     master  2.7.0    5270208   775536    577.49M  0.3    202     20     190       4
 |-172.16.84.94:19103    OK     slave   2.7.0    5270684   775544    573.72M  0.2    2       1      224      
172.16.84.94:19100       OK     master  2.7.0    5270699   774345    581.09M  1.3    396     28     301       1
 |-172.16.84.93:19100    OK     slave   2.7.0    5270223   774345    573.14M  0.2    2       1      1013     
172.16.84.94:19101       OK     master  2.7.0    5270699   777694    579.13M  0.3    175     11     409       2
 |-172.16.84.93:19101    OK     slave   2.7.0    5270223   777694    575.97M  0.2    2       0      243      
_________________________________________________________________________________________________________________________
SUM(master/all): NODE=4/8 KEY=3100904/6201824 MEM=2.3G/4.5G CLI=1087/1095 OPS=107/115 CPU=2.3/3.1
=========================================================================================================================
```

#### 输出结果说明

1. `SERVER`：Redis服务地址

2. `STATUS`：可用于判断节点基本状态

   - `OK`：节点正常运行中

   - `FAIL`：节点连接失败

   - `LOAD`：节点正在进行数据加载

   - `NOAUTH`：节点密码认证失败（需指定正确的访问密码）

3. `ROLE`：如果节点角色变更，可能是发生过故障切换，需要通过服务日志进一步核查

   - `master`：主节点
   - `slave`：从节点

4. `VERSION`：对于Redis集群，版本号应该 >=`3.0.0`

5. `UPTIME(s)`：节点已经运行的时长（单位：s），**可用于判断节点是否发生重启**

6. `KEYS`：节点中存在的键值数

7. `USEMEM`：节点使用的内存量，建议不要超过10G，如果过高应该扩大集群规模

8. `CLIENTS`：节点当前接入的客户端连接数

9. `%CPU`：节点当前的CPU使用率百分比，通常不应该超过70%

10. `OPS`：节点当前每秒处理的命令请求量

11. `RTT`：节点当前请求响应时延（单位：us），通常不应该超过1000

12. `SUM(master/all)`：按主节点和集群维度，汇总节点数、键值数、内存使用量、客户端连接数、OPS、%CPU等指标信息

**备注**：

- 主节点分布均匀，能够更好的发挥Redis性能，而且也有利于故障切换时的主从选举，所以对于主节点分布不均匀时，会给出告警信息
- 工具支持`-r`参数，用于将集群节点指标以裸输据输出，方便进行指标文件存储，或上报至第三方监控平台



### 慢命令分析

使用`slowlog`子命令可以汇总集群各节点的慢命令日志，按日志生成时间进行排序展示格式化后的命令详情：

```bash
$ redis-tool -p 6380 slowlog
2022-04-28 18:13:46 <INFO> [CLUSTER] get slowlog of cluster by the node 127.0.0.1:6380 (COUNT=100)
=========================================== 2022-04-28 18:13:47 ============================================
Redis                 ID     Time[Asc]       Duration(us) Client                Command
------------------------------------------------------------------------------------------------------------
172.16.18.80:6379     0      20220428181334  728          172.16.18.80:47026    [ "COMMAND" ]
172.16.18.81:6379     0      20220428181334  897          172.16.18.80:58702    [ "COMMAND" ]
172.16.18.82:6379     0      20220428181334  996          172.16.18.80:57426    [ "COMMAND" ]
172.16.18.80:6379     1      20220428181337  697          172.16.18.80:47044    [ "COMMAND" ]
172.16.18.81:6379     1      20220428181337  1033         172.16.18.80:58714    [ "COMMAND" ]
172.16.18.82:6379     1      20220428181337  1024         172.16.18.80:57444    [ "COMMAND" ]
172.16.18.80:6379     2      20220428181341  716          172.16.18.80:47068    [ "COMMAND" ]
172.16.18.81:6379     2      20220428181341  1320         172.16.18.80:58742    [ "COMMAND" ]
172.16.18.82:6379     2      20220428181341  767          172.16.18.80:57470    [ "COMMAND" ]
172.16.18.80:6379     3      20220428181344  705          172.16.18.80:47104    [ "COMMAND" ]
172.16.18.81:6379     3      20220428181344  886          172.16.18.80:58780    [ "COMMAND" ]
172.16.18.82:6379     3      20220428181344  834          172.16.18.80:57504    [ "COMMAND" ]
172.16.18.82:6379     4      20220428181344  111          172.16.18.80:57504    [ "info" "commandstats" ]
---------------------------[ 20220428:       count = 13 ]-------------------------------------------------
```

**结果说明**：

- 日志按时间顺序展示，当前页面展示最新的日志信息
- 对于存在慢命令日志的日期会进行日志数量汇总
- 工具支持`-c <count>`参数，来指定查询节点最新的慢命令记录数（默认值：`100`）

**慢命令日志主要原因**：

- **大key访问**：key值包含的元素数太大时对Redis服务性能影响较大，建议优化业务逻辑，进行大key拆分或使用替代命令（例如使用`HSCAN`代替`HGETALL`）
- **复杂命令**：`KEYS`、`FLUSHDB`、`HGETALL`等，生产环境通常禁止使用
- **主机内存不足**：当主机因为内存不足，而使用了磁盘交换区时，对于Redis服务性能影响很大，需要进行内存使用限制或主机资源扩容
- **主机CPU繁忙**：主机CPU配置较低，或者部署了其他高CPU占用软件，生产环境建议Redis主机独占部署
- **CPU使用限制**：通过cgroup限制了Redis进程的CPU使用率，导致访问性能下降，生产环境不建议进行限制

### 热点key分析

当发现某个Redis节点的`%CPU`或`OPS`指标相对于其他节点高很多时，通常是因为存在热点key访问，此时可以使用`trace`子命令实时监控该节点的命令请求，并分析KEY的请求百分比，从而发现热点key：

**备注**：实时命令监控统计实际使用的时Redis的`monitor`命令，因此建议在节点所在主机上执行，并且不要使用`-t`参数调整监控时间过长。

```bash
# 1. 实时监控统计节点每秒处理的命令，并记录命令详情至文件中[172.16.18.81-6379.mon]
#    默认实时跟踪10秒，可通过-t <time>参数进行修改
$ ./redis-tool -h 172.16.18.81 -p 6379 trace 
2022-04-28 19:07:42 <INFO> Analyze the command on seconds for the node 172.16.18.81:6379 ...
--------------------------------------------------------------------------------
19:07:42> "GET":1584
19:07:43> "GET":4927
19:07:44> "GET":3255
19:07:45> "AUTH":1 "COMMAND":1 "GET":5052 "INFO":3
19:07:46> "GET":2842
19:07:47> "GET":5699
19:07:48> "AUTH":1 "COMMAND":1 "GET":3598 "INFO":3
19:07:49> "GET":4543
19:07:50> "GET":3699
19:07:51> "AUTH":1 "COMMAND":1 "GET":5074 "INFO":3
19:07:52> "GET":4025
2022-04-28 19:07:52 <INFO> You can check the monitor file: [172.16.18.81-6379.mon], or analyze the file by -f again!
# 2. 基于监控的命令详情文件进行热点KEY访问分析
$ ./redis-tool -f 172.16.18.81-6379.mon -H trace
2022-04-28 19:08:06 <INFO> Analyze the monitor file 172.16.18.81-6379.mon for hot keys ...
==================================================  =======  =======
KEY                                                 COUNT    PERCENT
--------------------------------------------------  -------  -------
"commandstats"                                      3        0.01%
"cpu"                                               3        0.01%
NO-KEY                                              3        0.01%
"(redacted)"                                        3        0.01%
"stats"                                             3        0.01%
"key1"                                              44298    99.97%
```



热点key的存在通常需要从业务使用侧进行优化，**主要的处理策略**：

- 将每次业务访问的key进行拆分，避免总是访问同一个key
- 对需要频繁访问的key进行本地缓存，本地缓存数据可以通过定时策略进行更新
- 优化业务流程处理逻辑，减少无效的交互访问次数



### TOP命令跟踪

使用`moni`子命令能够滚动展示周期内TOP10命令执行的执行情况；当存在大量非应用直接调用的命令时（例如`PING`、`CLUSTER`），或者某个命令执行次数或平均处理耗时不符合预期，都可以作为下一步排查的方向。

**备注**：

- 默认汇总集群所有主节点的命令执行情况；当使用`-s`参数时，将只展示指定Redis节点的命令执行情况
- 每滚动输出20次命令统计后，会重新计算当前的TOP10命令进行展示

```bash
# 1. 每秒滚动展示集群TOP10命令
$ ./redis-tool -h 172.16.18.80 -p 6380 -d 1 moni
2022-04-28 18:46:11 <INFO> [CLUSTER] monitor with 3 masters from 172.16.18.80:6380 every 1 seconds
-----------------------TOP10 commands 2022-04-28 18:46:12----------------------- -----all------ -----------netio-----
get     info    replconf command auth    time    slowlog set     select  scan    TOTAL   %CPU   CONNS IN(KB)  OUT(KB)
4445    9       3        3       3       0       0       0       0       0       4463    17.5   7     100     118    
6115    9       4        3       3       0       0       0       0       0       6134    19.7   4     139     139    
4038    9       4        3       3       0       0       0       0       0       4057    16.0   7     91      113    
5647    9       4        3       3       0       0       0       0       0       5666    23.0   6     127     133    
6051    9       3        3       3       0       0       0       0       0       6069    22.0   6     137     137  
... ...
# 2. 每3秒滚动展示集群TOP10命令及平均处理耗时(*数值，对应的即为平均处理耗时，单位：us)
$ ./redis-tool -h 172.16.18.80 -p 6380 -c 5 -l moni
2022-04-28 18:47:17 <INFO> [CLUSTER] monitor with 3 masters from 172.16.18.80:6380 every 3 seconds
-----------------------TOP10 commands 2022-04-28 18:47:20----------------------- -----all------ -----------netio-----
get     replconf info    command auth    time    slowlog set     select  scan    TOTAL   %CPU   CONNS IN(KB)  OUT(KB)
14513   10       9       3       3       0       0       0       0       0       14538   21.2   10    327     237    
*1      *1       *32     *891    *7      *0      *0      *0      *0      *0      
16470   10       9       3       3       0       0       0       0       0       16495   22.8   11    371     259    
*1      *1       *33     *904    *30     *0      *0      *0      *0      *0      
11883   9        9       3       3       0       0       0       0       0       11907   16.9   16    268     206    
*1      *1       *25     *882    *8      *0      *0      *0      *0      *0      
15255   10       9       3       3       0       0       0       0       0       15280   19.2   11    343     245    
*1      *1       *28     *979    *9      *0      *0      *0      *0      *0      
7974    9        9       3       3       0       0       0       0       0       7998    13.2   9     180     160    
*1      *1       *37     *1020   *8      *0      *0      *0      *0      *0      

```

**结果说明**：

1. **TOP10 commands**：
   - 命令执行次数：周期内命令执行次数
   - 平均处理耗时：使用`-l`参数时，会增加一行显示命令平均处理耗时（以`*`开头）
2. **all**：
    - TOTAL：周期内执行的命令总数
    - %CPU：当前节点的CPU使用率汇总
3. **netio**：
   - CONNS：周期内新建客户端连接数
   - IN(KB)：周期内网络流入数据量汇总
   - OUT(KB)：周期内网络流出数据量汇总



### KEY前缀统计

使用`keys`子命令可以快速的抽取一定数量的KEY名称，对KEY按前缀统计键值数和占比信息后按前缀层级进行展示，其中第一层级会采用背景高亮，而键值占比达到一定阈值时会进行字体高亮区分：

```bash
# 1. 连接指定redis节点进行KEY前缀统计（默认前缀层级为3）
$ ./redis-tool -p 6380 keys
Time elapsed 0 (s): 52
=============================================
 KEY_PREFIX             | COUNT    | PERCENT
------------------------|----------|---------
 hash:                  | 5        | 9.62%
  |- hash.foo           | 5        | 9.62%
      |- hash.foo.00000 | 5        | 9.62%
 key0:                  | 4        | 7.69%
 set:                   | 4        | 7.69%
  |- set.aaa            | 4        | 7.69%
      |- set.aaa.bbb    | 4        | 7.69%
 str:                   | 39       | 75.00%
  |- str.abc            | 2        | 3.85%
      |- str.abc.efg    | 1        | 1.92%
      |- str.abc.lmn    | 1        | 1.92%
  |- str.xxx            | 37       | 71.15%
      |- str.xxx.0      | 5        | 9.62%
      |- str.xxx.00     | 32       | 61.54%
=============================================
# 2. 使用上面生成的键名称文件127.0.0.1-6380.keys进行分析，前缀层级为4
$ ./redis-tool -f 127.0.0.1-6380.keys -L 4 keys
==================================================
 KEY_PREFIX                  | COUNT    | PERCENT
-----------------------------|----------|---------
 hash:                       | 5        | 9.62%
  |- hash.foo                | 5        | 9.62%
      |- hash.foo.00000      | 5        | 9.62%
 key0:                       | 4        | 7.69%
 set:                        | 4        | 7.69%
  |- set.aaa                 | 4        | 7.69%
      |- set.aaa.bbb         | 4        | 7.69%
          |- set.aaa.bbb.ccc | 4        | 7.69%
 str:                        | 39       | 75.00%
  |- str.abc                 | 2        | 3.85%
      |- str.abc.efg         | 1        | 1.92%
          |- str.abc.efg.000 | 1        | 1.92%
      |- str.abc.lmn         | 1        | 1.92%
          |- str.abc.lmn.00  | 1        | 1.92%
  |- str.xxx                 | 37       | 71.15%
      |- str.xxx.0           | 5        | 9.62%
      |- str.xxx.00          | 32       | 61.54%
==================================================
```

**备注**：KEY前缀连接符统一使用`.`进行代替（这样也方便使用`grep`命令来对keys文件按前缀进行检索）

### 配置参数管理

使用`config`子命令可以方便的检查各节点的配置差异，以及按角色来统一查询/修改指定运行参数。

#### 检查节点配置差异

当未指定参数进行查询或修改时，默认会检查集群中各节点的配置差异，并且按各参数值数量(`UNIQ`)升序展示，其中存在不同值的参数行会高亮显示：

- CONFIG_NAME：配置参数名
- UNIQ：不同参数值数量，`1`表示各节点的参数值相同
- DIFF_VALUES：参数值详情，不同参数值以逗号（`,`）间隔

```bash
$ ./redis-tool -p 6380 config
2022-04-28 17:52:54 <INFO> Audit the config with diffrent value on all(count=6) nodes:
====================================================================================================
 CONFIG_NAME                     | UNIQ | DIFF_VALUES
----------------------------------------------------------------------------------------------------
 aclfile                         | 1    |                                                           
 acllog-max-len                  | 1    | 128                                                       
 acl-pubsub-default              | 1    | allchannels                                               
 ... ... 
 zset-max-ziplist-entries        | 1    | 128                                                       
 zset-max-ziplist-value          | 1    | 64                                                        
 dbfilename                      | 2    | .dump.rdb,dump.rdb                                        
 dir                             | 2    | /home/redis/data/6380,/tmp                                
 logfile                         | 2    | redis-6379.log,redis-6380.log                             
 pidfile                         | 2    | /home/redis/data//6379/redis-6379.pid,/home/redis/data/...
 port                            | 2    | 6379,6380                                                 
 slaveof                         | 4    | ,172.16.18.80 6379,172.16.18.81 6379,172.16.18.82 6379    
----------------------------------------------------------------------------------------------------
 CONFIG_NAME                     | UNIQ | DIFF_VALUES
====================================================================================================
```

 

#### 查询/修改指定参数

集群参数查询/修改范围包括三类：所有节点（默认），所有主节点（`-M`），所有从节点（`-S`）

```bash
# 1. 查询集群各节点的slowlog-log-slower-than参数值
$ ./redis-tool -p 6380 -k slowlog-log-slower-than config
2022-04-28 18:11:23 <INFO> GET the value of 'slowlog-log-slower-than' on all nodes:
================================================================================
 REDIS                 | slowlog-log-slower-than
--------------------------------------------------------------------------------
 172.16.18.80:6379     | "10000"
 172.16.18.80:6380     | "10000"
 172.16.18.81:6379     | "10000"
 172.16.18.81:6380     | "10000"
 172.16.18.82:6379     | "10000"
 172.16.18.82:6380     | "10000"
================================================================================
# 2. 修改集群主节点的slowlog-log-slower-than参数值为100
$ ./redis-tool -p 6380 -k slowlog-log-slower-than -v 100 -M config
2022-04-28 18:12:55 <INFO> SET the value of 'slowlog-log-slower-than' on master nodes:
2022-04-28 18:12:55 <INFO> [172.16.18.80:6379]: CONFIG SET slowlog-log-slower-than 100 : [OK]
2022-04-28 18:12:55 <INFO> [172.16.18.81:6379]: CONFIG SET slowlog-log-slower-than 100 : [OK]
2022-04-28 18:12:55 <INFO> [172.16.18.82:6379]: CONFIG SET slowlog-log-slower-than 100 : [OK]
2022-04-28 18:12:55 <INFO> Get the value of 'slowlog-log-slower-than' on master nodes after CONFIG SET:
================================================================================
 REDIS                 | slowlog-log-slower-than
--------------------------------------------------------------------------------
 172.16.18.80:6379     | "100"
 172.16.18.81:6379     | "100"
 172.16.18.82:6379     | "100"
================================================================================
```

**备注**：使用`-w`参数时，当Redis运行参数修改后，会重写Redis配置文件，这样节点重启后能够继续生效

**密码参数统一修改示例**：

```bash
# 示例一：无密码集群新增密码：redis
redis-tool -h 172.16.18.80 -p 6379 -a '' -k masterauth -v 'redis' config
redis-tool -h 172.16.18.80 -p 6379 -a '' -k requirepass -v 'redis' -w config
redis-tool -h 172.16.18.80 -p 6379 -a 'redis'

# 示例二：将集群密码redis修改为Red1s@2022
redis-tool -h 172.16.18.80 -p 6379 -a 'redis' -k masterauth -v 'Red1s@2022' config
redis-tool -h 172.16.18.80 -p 6379 -a 'redis' -k requirepass -v 'Red1s@2022' -w config
redis-tool -h 172.16.18.80 -p 6379 -a 'Red1s@2022'

# 示例三：去掉集群密码设置
redis-tool -h 172.16.18.80 -p 6379 -a 'Red1s@2022' -k masterauth -v '' config
redis-tool -h 172.16.18.80 -p 6379 -a 'Red1s@2022' -k requirepass -v '' -w config
redis-tool -h 172.16.18.80 -p 6379
```







