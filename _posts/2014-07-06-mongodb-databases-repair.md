---
title: 修复MongoDB Replica Set数据库
layout: post
tags:
    - mongodb
---


1 - 登入PRIMARY的机器检查replica sets里的其他SECONDARY成员的数据是否跟PRIMARY保持同步：

```
$ mongo primary_host:port
PRIMARY> rs.status()
// 为方便阅读，这里只列出了关键字段
{
    "set" : "BackupTest",
    "members" : [
        {
            "_id" : 0,
            "name" : "MacintoshdeMacBook-Air.local:30001",
            "stateStr" : "PRIMARY",
            "optime" : Timestamp(1407767556, 100),
            "optimeDate" : ISODate("2014-08-11T14:32:36Z")
        },
        {
            "_id" : 1,
            "name" : "MacintoshdeMacBook-Air.local:30002",
            "stateStr" : "SECONDARY",
            "optime" : Timestamp(1407767556, 100),
            "optimeDate" : ISODate("2014-08-11T14:32:36Z")
        },
        {
            "_id" : 2,
            "name" : "MacintoshdeMacBook-Air.local:30003",
            "stateStr" : "ARBITER"
        }
    ]
}
```
检查members数组里各个SECONDARY成员的*optime*是否与PRIMARY的*optime*保持一致， 接下来的操作将会丢失两者时间差的数据。

2. 将当前的PRIMARY切换为SECONDARY：

```
PRIMARY> rs.stepDown()  # 切换
2014-08-12T18:45:39.480+0800 DBClientCursor::init call() failed
2014-08-12T18:45:39.483+0800 Error: error doing query: failed at src/mongo/shell/query.js:81
2014-08-12T18:45:39.485+0800 trying reconnect to 127.0.0.1:30001 (127.0.0.1) failed
2014-08-12T18:45:39.486+0800 reconnect 127.0.0.1:30001 (127.0.0.1) ok
SECONDARY> rs.status()
// 为方便阅读，这里只列出了关键字段
{
    "set" : "BackupTest",
    "members" : [
        {
            "_id" : 0,
            "name" : "MacintoshdeMacBook-Air.local:30001",
            "stateStr" : "SECONDARY"    // PRIMARY -> SECONDARY
            "self" : true
        },
        {
            "_id" : 1,
            "name" : "MacintoshdeMacBook-Air.local:30002",
            "stateStr" : "PRIMARY"  // SECONDARY -> PRIMARY
        },
        {
            "_id" : 2,
            "name" : "MacintoshdeMacBook-Air.local:30003",
            "stateStr" : "ARBITER"
        }
    ]
}
```
检查是否切换成功（别的机器是否成为了PRIMARY， 当前机器是否成为了SECONDARY）

3. 结束当前的SECONDARY（旧的PRIMARY）进程：

```
SECONDARY> use admin
SECONDARY> db.shutdownServer()
```

检查各个使用到MongoDB的Client进程是否能正常查询和写入数据。


4. 更改原PRIMARY的配置文件，取消replica set模式并更改端口启动

```
$ sudo vi /etc/mongodb.conf
# 注释掉 replSet = yourReplicaName 和 journal = true
# 并更改 port 至原来不一样的值，例如 port = 30001 -> 30011
```

5. 以修复模式启动MongoDB：

```
$ sudo -umongodb mongod -f /etc/mongodb.conf --repair --repairpath /xxx/xxx # /xxx/xxx 必须是MongoDB dbpath的子目录
```
修复期间可以通过log文件查看修复进度。

6. 修复完后，重新以replica set模式启动：

```
$ sudo vi /etc/mongodb.conf
# 取消刚才replSet = yourReplicaName 和 journal = true 的注释
# 并把port改回原始值
$ sudo -umongodb numactl --interleave=all mongod -f /etc/mongodb.conf   #启动
$ mongo old_primary_host:port
SECONDARY> rs.status()
```

至此，数据库已修复完成， replica set恢复至正常状态。

7. 假如想将机器重新变成PRIMARY，只需要在现有的PRIMARY机器上重新执行步骤1-2。
如果Replica Sets里有多个SECONDARY，MongoDB会在满足条件的基础上随机选取一个变成PRIMARY。
如果想将特定的SECONDARY变成PRIMARY，可以在其他SECONDARY上执行```SECONDARY> rs.freeze(60) ``` 防止其在60秒内变成PRIMARY。

P.S:
    在修复完数据库，重新启动MongoDB时，有可能出现端口被占用的情况，可以添加系统预留端口，防止MongoDB的端口被随机分配。
    
```
$ sudo vim /etc/sysctl.conf
net.ipv4.ip_local_reserved_ports = 30001, 30003-30004
$ sudo sysctl -p
```




