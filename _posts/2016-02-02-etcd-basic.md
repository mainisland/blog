---
layout: post
title: etcd 基础
author: wilmosfang
tags:   etcd
categories:   etcd
wc: 858 2417 40495
excerpt: 下载安装，服务运行，存取测试，单点集群，获取版本信息，关键字操作，统计信息等相关API
comments: true
---



# 前言



**[etcd][etcd]** 是一个分布式的，一致性键值存储，主要用于共享配置和服务发现

>etcd is a distributed, consistent key-value store for shared configuration and service discovery, with a focus on being:
>
> * Simple: curl able user-facing API (HTTP+JSON)
> * Secure: optional SSL client cert authentication
> * Fast: benchmarked 1000s of writes/s per instance
> * Reliable: properly distributed using Raft

它是使用 **Go** 开发的，**[Raft][raft]** 算法是其一致性保障的核心

> **Tip:** **[Raft][raft]** 的相关细节可以参考 **[剖析 etcd][coreos_analyse_etcd]**  和 **[raft动画][raft_move]**


这里简单分享一下 **[etcd][etcd]** 的基础 ，相关的详细内容可以参考 **[官方Git][etcd]**


> **Tip:** 当前的最新版本为 **etcd v2.2.4** 
>
> **Note:** The master branch may be in an unstable or even broken state during development. Please use **[releases][releases]** instead of the master branch in order to get stable binaries

---


# 概要


* TOC
{:toc}



---

## 下载安装

~~~
[root@docker etcd]# curl -L  https://github.com/coreos/etcd/releases/download/v2.2.4/etcd-v2.2.4-linux-amd64.tar.gz -o etcd-v2.2.4-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   606    0   606    0     0    328      0 --:--:--  0:00:01 --:--:--   328
  0     0    0     0    0     0      0      0 --:--:--  0:02:08 --:--:--     0curl: (7) Failed connect to github-cloud.s3.amazonaws.com:443; Connection timed out
[root@h104 ~]# wget https://github.com/coreos/etcd/releases/download/v2.2.4/etcd-v2.2.4-linux-amd64.tar.gz
--2016-02-01 22:34:11--  https://github.com/coreos/etcd/releases/download/v2.2.4/etcd-v2.2.4-linux-amd64.tar.gz
Resolving github.com (github.com)... 192.30.252.129
Connecting to github.com (github.com)|192.30.252.129|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-cloud.s3.amazonaws.com/releases/11225014/559a703a-ba00-11e5-8b3c-1aeb0a1fad55.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAISTNZFOVBIJMK3TQ%2F20160201%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20160201T143415Z&X-Amz-Expires=300&X-Amz-Signature=addebe637a795ab47739422d9963a2694c006fb29e402b288f14a5ac3690a5ca&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Detcd-v2.2.4-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2016-02-01 22:34:16--  https://github-cloud.s3.amazonaws.com/releases/11225014/559a703a-ba00-11e5-8b3c-1aeb0a1fad55.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAISTNZFOVBIJMK3TQ%2F20160201%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20160201T143415Z&X-Amz-Expires=300&X-Amz-Signature=addebe637a795ab47739422d9963a2694c006fb29e402b288f14a5ac3690a5ca&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Detcd-v2.2.4-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving github-cloud.s3.amazonaws.com (github-cloud.s3.amazonaws.com)... 54.231.19.152
Connecting to github-cloud.s3.amazonaws.com (github-cloud.s3.amazonaws.com)|54.231.19.152|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7567734 (7.2M) [application/octet-stream]
Saving to: ‘etcd-v2.2.4-linux-amd64.tar.gz’

 0% [                                                                                             ] 16,959      68.5B/s   in 4m 7s   

2016-02-01 22:39:24 (68.5 B/s) - Read error at byte 16959/7567734 (Connection reset by peer). [root@h104 ~]# 
[root@docker etcd]# ls
etcd-v2.2.4-linux-amd64.tar.gz
[root@docker etcd]# tar -zxvf etcd-v2.2.4-linux-amd64.tar.gz 
etcd-v2.2.4-linux-amd64/
etcd-v2.2.4-linux-amd64/Documentation/
etcd-v2.2.4-linux-amd64/Documentation/runtime-configuration.md
etcd-v2.2.4-linux-amd64/Documentation/admin_guide.md
etcd-v2.2.4-linux-amd64/Documentation/tuning.md
etcd-v2.2.4-linux-amd64/Documentation/glossary.md
etcd-v2.2.4-linux-amd64/Documentation/rfc/
etcd-v2.2.4-linux-amd64/Documentation/rfc/v3api.md
etcd-v2.2.4-linux-amd64/Documentation/rfc/v3api.proto
etcd-v2.2.4-linux-amd64/Documentation/discovery_protocol.md
etcd-v2.2.4-linux-amd64/Documentation/errorcode.md
etcd-v2.2.4-linux-amd64/Documentation/metrics.md
etcd-v2.2.4-linux-amd64/Documentation/security.md
etcd-v2.2.4-linux-amd64/Documentation/configuration.md
etcd-v2.2.4-linux-amd64/Documentation/docker_guide.md
etcd-v2.2.4-linux-amd64/Documentation/dev/
etcd-v2.2.4-linux-amd64/Documentation/dev/release.md
etcd-v2.2.4-linux-amd64/Documentation/auth_api.md
etcd-v2.2.4-linux-amd64/Documentation/backward_compatibility.md
etcd-v2.2.4-linux-amd64/Documentation/platforms/
etcd-v2.2.4-linux-amd64/Documentation/platforms/freebsd.md
etcd-v2.2.4-linux-amd64/Documentation/libraries-and-tools.md
etcd-v2.2.4-linux-amd64/Documentation/implementation-faq.md
etcd-v2.2.4-linux-amd64/Documentation/reporting_bugs.md
etcd-v2.2.4-linux-amd64/Documentation/upgrade_2_2.md
etcd-v2.2.4-linux-amd64/Documentation/internal-protocol-versioning.md
etcd-v2.2.4-linux-amd64/Documentation/upgrade_2_1.md
etcd-v2.2.4-linux-amd64/Documentation/faq.md
etcd-v2.2.4-linux-amd64/Documentation/runtime-reconf-design.md
etcd-v2.2.4-linux-amd64/Documentation/clustering.md
etcd-v2.2.4-linux-amd64/Documentation/proxy.md
etcd-v2.2.4-linux-amd64/Documentation/branch_management.md
etcd-v2.2.4-linux-amd64/Documentation/other_apis.md
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/etcd-2-2-0-rc-memory-benchmarks.md
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/etcd-3-demo-benchmarks.md
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/etcd-2-1-0-alpha-benchmarks.md
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/README.md
etcd-v2.2.4-linux-amd64/Documentation/benchmarks/etcd-2-2-0-rc-benchmarks.md
etcd-v2.2.4-linux-amd64/Documentation/api.md
etcd-v2.2.4-linux-amd64/Documentation/authentication.md
etcd-v2.2.4-linux-amd64/Documentation/04_to_2_snapshot_migration.md
etcd-v2.2.4-linux-amd64/Documentation/production-ready.md
etcd-v2.2.4-linux-amd64/README-etcdctl.md
etcd-v2.2.4-linux-amd64/etcdctl
etcd-v2.2.4-linux-amd64/etcd
etcd-v2.2.4-linux-amd64/README.md
[root@docker etcd]# ll 
total 7392
drwxrwxr-x 3 cc   cc        91 Jan 14 06:13 etcd-v2.2.4-linux-amd64
-rw-r--r-- 1 root root 7567734 Feb  1 22:36 etcd-v2.2.4-linux-amd64.tar.gz
[root@docker etcd]# cd etcd-v2.2.4-linux-amd64/
[root@docker etcd-v2.2.4-linux-amd64]# ls
Documentation  etcd  etcdctl  README-etcdctl.md  README.md
[root@docker etcd-v2.2.4-linux-amd64]# ll
total 28000
drwxrwxr-x 6 cc cc     4096 Jan 14 06:13 Documentation
-rwxr-xr-x 1 cc cc 15145664 Jan 14 06:13 etcd
-rwxr-xr-x 1 cc cc 13502144 Jan 14 06:13 etcdctl
-rw-rw-r-- 1 cc cc     5613 Jan 14 06:13 README-etcdctl.md
-rw-rw-r-- 1 cc cc     4684 Jan 14 06:13 README.md
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

> **Tip:** 上面的报错是由于地址被重定向了，到了一个被墙的服务器上，于是翻墙，改为手动下载，放到了目录底下


---

## 运行服务

~~~
[root@docker etcd-v2.2.4-linux-amd64]# ./etcd
2016-02-01 22:42:57.420693 I | etcdmain: etcd Version: 2.2.4
2016-02-01 22:42:57.420802 I | etcdmain: Git SHA: bdee27b
2016-02-01 22:42:57.420809 I | etcdmain: Go Version: go1.5.3
2016-02-01 22:42:57.420816 I | etcdmain: Go OS/Arch: linux/amd64
2016-02-01 22:42:57.420825 I | etcdmain: setting maximum number of CPUs to 2, total number of available CPUs is 2
2016-02-01 22:42:57.420833 W | etcdmain: no data-dir provided, using default data-dir ./default.etcd
2016-02-01 22:42:57.421532 I | etcdmain: listening for peers on http://localhost:2380
2016-02-01 22:42:57.421658 I | etcdmain: listening for peers on http://localhost:7001
2016-02-01 22:42:57.421754 I | etcdmain: listening for client requests on http://localhost:2379
2016-02-01 22:42:57.421860 I | etcdmain: listening for client requests on http://localhost:4001
2016-02-01 22:42:57.422302 I | etcdserver: name = default
2016-02-01 22:42:57.422318 I | etcdserver: data dir = default.etcd
2016-02-01 22:42:57.422326 I | etcdserver: member dir = default.etcd/member
2016-02-01 22:42:57.422333 I | etcdserver: heartbeat = 100ms
2016-02-01 22:42:57.422339 I | etcdserver: election = 1000ms
2016-02-01 22:42:57.422346 I | etcdserver: snapshot count = 10000
2016-02-01 22:42:57.422361 I | etcdserver: advertise client URLs = http://localhost:2379,http://localhost:4001
2016-02-01 22:42:57.422372 I | etcdserver: initial advertise peer URLs = http://localhost:2380,http://localhost:7001
2016-02-01 22:42:57.422388 I | etcdserver: initial cluster = default=http://localhost:2380,default=http://localhost:7001
2016-02-01 22:42:57.445177 I | etcdserver: starting member ce2a822cea30bfca in cluster 7e27652122e8b2ae
2016-02-01 22:42:57.445294 I | raft: ce2a822cea30bfca became follower at term 0
2016-02-01 22:42:57.445322 I | raft: newRaft ce2a822cea30bfca [peers: [], term: 0, commit: 0, applied: 0, lastindex: 0, lastterm: 0]
2016-02-01 22:42:57.445331 I | raft: ce2a822cea30bfca became follower at term 1
2016-02-01 22:42:57.446112 I | etcdserver: starting server... [version: 2.2.4, cluster version: to_be_decided]
2016-02-01 22:42:57.447573 E | etcdmain: failed to notify systemd for readiness: No socket
2016-02-01 22:42:57.447621 E | etcdmain: forgot to set Type=notify in systemd service file?
2016-02-01 22:42:57.447838 N | etcdserver: added local member ce2a822cea30bfca [http://localhost:2380 http://localhost:7001] to cluster 7e27652122e8b2ae
2016-02-01 22:42:57.847950 I | raft: ce2a822cea30bfca is starting a new election at term 1
2016-02-01 22:42:57.848346 I | raft: ce2a822cea30bfca became candidate at term 2
2016-02-01 22:42:57.848370 I | raft: ce2a822cea30bfca received vote from ce2a822cea30bfca at term 2
2016-02-01 22:42:57.848488 I | raft: ce2a822cea30bfca became leader at term 2
2016-02-01 22:42:57.848515 I | raft: raft.node: ce2a822cea30bfca elected leader ce2a822cea30bfca at term 2
2016-02-01 22:42:57.849127 I | etcdserver: setting up the initial cluster version to 2.2
2016-02-01 22:42:57.881583 N | etcdserver: set the initial cluster version to 2.2
2016-02-01 22:42:57.882146 I | etcdserver: published {Name:default ClientURLs:[http://localhost:2379 http://localhost:4001]} to cluster 7e27652122e8b2ae
...
...
...
~~~

前台运行，直接占用了当前终端


---

## 存取测试


~~~
[root@docker etcd-v2.2.4-linux-amd64]# ./etcdctl set keytest "hello world for etcd test"
hello world for etcd test
[root@docker etcd-v2.2.4-linux-amd64]# ./etcdctl get keytest
hello world for etcd test
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

---

## 单点运行

~~~
[root@docker etcd-v2.2.4-linux-amd64]# netstat  -ant | grep -E '(2379|2380)'
[root@docker etcd-v2.2.4-linux-amd64]# 
----------
[root@docker etcd-v2.2.4-linux-amd64]# ./etcd
2016-02-01 22:54:18.977681 I | etcdmain: etcd Version: 2.2.4
2016-02-01 22:54:18.977850 I | etcdmain: Git SHA: bdee27b
2016-02-01 22:54:18.977859 I | etcdmain: Go Version: go1.5.3
2016-02-01 22:54:18.977867 I | etcdmain: Go OS/Arch: linux/amd64
...
...
...
----------
[root@docker etcd-v2.2.4-linux-amd64]# netstat  -ant | grep -E '(2379|2380)'
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN     
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

etcd运行后会监听在本地的 **2379** 和 **2380** 上面

* 2379 用来与客户通讯
* 2380 用来进行服务间通讯


----

## 获取版本信息

~~~
[root@docker etcd-v2.2.4-linux-amd64]# curl -L http://127.0.0.1:2379/version
{"etcdserver":"2.2.4","etcdcluster":"2.2.0"}[root@docker etcd-v2.2.4-linux-amd64]# 
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

---

## 关键字空间操作

etcd主要在维护一个分层的关键字空间，关键字空间由关键字和目录构成，它们也被称作节点

etcd的绝大部分API都是在对这两类对象进行操作

### 给关键字赋值


~~~
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":8,"createdIndex":8}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":9,"createdIndex":9},"prevNode":{"key":"/message","value":"Hello world","modifiedIndex":8,"createdIndex":8}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":10,"createdIndex":10},"prevNode":{"key":"/message","value":"Hello world","modifiedIndex":9,"createdIndex":9}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":11,"createdIndex":11},"prevNode":{"key":"/message","value":"Hello world","modifiedIndex":10,"createdIndex":10}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":12,"createdIndex":12},"prevNode":{"key":"/message","value":"Hello world","modifiedIndex":11,"createdIndex":11}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/abc -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/abc","value":"Hello world","modifiedIndex":13,"createdIndex":13}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/abc -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/abc","value":"Hello world","modifiedIndex":14,"createdIndex":14},"prevNode":{"key":"/abc","value":"Hello world","modifiedIndex":13,"createdIndex":13}}
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/abc -XPUT -d value="abc"
{"action":"set","node":{"key":"/abc","value":"abc","modifiedIndex":15,"createdIndex":15},"prevNode":{"key":"/abc","value":"Hello world","modifiedIndex":14,"createdIndex":14}}
[root@docker etcd-v2.2.4-linux-amd64]#
~~~


Attribute | Comment
-------- | ---
action  | 事件类型
node.key | 关键字，etcd使用了文件系统的结构来管理关键字，都以 **`/`** 开头
node.value| 值
node.modifiedIndex | 在有修改操作时会递增
node.createdIndex|有创建操作时会递增


---

### 获取值

~~~
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"Hello world","modifiedIndex":12,"createdIndex":12}}
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~


---

### 修改值

~~~
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="hello etcd"
{"action":"set","node":{"key":"/message","value":"hello etcd","modifiedIndex":17,"createdIndex":17},"prevNode":{"key":"/message","value":"Hello world","modifiedIndex":12,"createdIndex":12}}
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

直接使用 **PUT** 就可以进行修改，并且后面会接上修改前的状态信息

---

### 删除键


~~~
[root@docker etcd-v2.2.4-linux-amd64]# curl http://127.0.0.1:2379/v2/keys/message -XDELETE
{"action":"delete","node":{"key":"/message","modifiedIndex":18,"createdIndex":17},"prevNode":{"key":"/message","value":"hello etcd","modifiedIndex":17,"createdIndex":17}}
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

> **Tip:** 留意到 **modifiedIndex** 有递增， **createdIndex** 却没有


---

### 设定key过期时间

~~~
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=6
Mon Feb  1 23:30:12 CST 2016
{"action":"set","node":{"key":"/foo","value":"bar","expiration":"2016-02-01T15:30:18.084936032Z","ttl":6,"modifiedIndex":21,"createdIndex":21}}
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo 
Mon Feb  1 23:30:17 CST 2016
{"action":"get","node":{"key":"/foo","value":"bar","expiration":"2016-02-01T15:30:18.084936032Z","ttl":1,"modifiedIndex":21,"createdIndex":21}}
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo 
Mon Feb  1 23:30:19 CST 2016
{"errorCode":100,"message":"Key not found","cause":"/foo","index":22}
[root@docker etcd-v2.2.4-linux-amd64]# 
~~~

多出来两个属性

Attribute | Comment
-------- | ---
node.expiration | 过期的时间点
node.ttl | 剩余的存活时间


> **Note:** 只有集群主节点才有能力让key过期，如果一个节点与集群失联，除非重新加入集群，否则它的key不会过期

---

### 解除过期

~~~
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=10
Mon Feb  1 23:41:21 CST 2016
{"action":"set","node":{"key":"/foo","value":"bar","expiration":"2016-02-01T15:41:31.103825075Z","ttl":10,"modifiedIndex":29,"createdIndex":29}}
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo 
Mon Feb  1 23:41:25 CST 2016
{"action":"get","node":{"key":"/foo","value":"bar","expiration":"2016-02-01T15:41:31.103825075Z","ttl":6,"modifiedIndex":29,"createdIndex":29}}
[root@docker etcd-v2.2.4-linux-amd64]# date; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl= -d prevExist=true
Mon Feb  1 23:41:30 CST 2016
{"action":"update","node":{"key":"/foo","value":"bar","modifiedIndex":30,"createdIndex":29},"prevNode":{"key":"/foo","value":"bar","expiration":"2016-02-01T15:41:31.103825075Z","ttl":1,"modifiedIndex":29,"createdIndex":29}}
[root@docker etcd-v2.2.4-linux-amd64]# date ; curl http://127.0.0.1:2379/v2/keys/foo 
Mon Feb  1 23:42:02 CST 2016
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":30,"createdIndex":29}}
[root@docker etcd-v2.2.4-linux-amd64]#
~~~

---

### 变更提醒

etcd可以实现变更提醒，如果要监控子层关键字的变更可以加上 **recursive=true**


我们打开一个终端，输入下面命令后，会hung住

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?wait=true
~~~

然后在另一个终端中输入

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=abc
{"action":"set","node":{"key":"/foo","value":"abc","modifiedIndex":33,"createdIndex":33},"prevNode":{"key":"/foo","value":"bar","modifiedIndex":31,"createdIndex":31}}
[root@docker ~]# 
~~~

于是第一个终端里会反馈出第二个终端里的结果，并退出hung状态

---

### 响应头

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo -vv
* About to connect() to 127.0.0.1 port 2379 (#0)
*   Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 2379 (#0)
> GET /v2/keys/foo HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 127.0.0.1:2379
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: application/json
< X-Etcd-Cluster-Id: 7e27652122e8b2ae
< X-Etcd-Index: 54
< X-Raft-Index: 11440
< X-Raft-Term: 6
< Date: Tue, 02 Feb 2016 02:49:21 GMT
< Content-Length: 90
< 
{"action":"get","node":{"key":"/foo","value":"abc","modifiedIndex":54,"createdIndex":54}}
* Connection #0 to host 127.0.0.1 left intact
[root@docker ~]# 
~~~

Item     | Comment
-------- | ---
X-Etcd-Cluster-Id| 集群ID
X-Etcd-Index| 当前的etcd index ,用来记录变更，类似于SCN,watch操作就是依此值判断，watch事件会发生在当前值之后
X-Raft-Index|为底层的raft协议标记序列，和 **X-Etcd-Index** 一样递增
X-Raft-Term|master选举发生的次数，发生一次递增一，如果递增速度太快，说明选举发生的太频繁，有必要进行timeout的调整


---

### 创建序列

因为是使用的etcd序列，所以确保了有序性

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job1
{"action":"create","node":{"key":"/queue/00000000000000000055","value":"Job1","modifiedIndex":55,"createdIndex":55}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job2
{"action":"create","node":{"key":"/queue/00000000000000000056","value":"Job2","modifiedIndex":56,"createdIndex":56}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job3
{"action":"create","node":{"key":"/queue/00000000000000000057","value":"Job3","modifiedIndex":57,"createdIndex":57}}
[root@docker ~]#
~~~

---

### 读取序列

~~~
[root@docker ~]# curl -s 'http://127.0.0.1:2379/v2/keys/queue?recursive=true&sorted=true'
{"action":"get","node":{"key":"/queue","dir":true,"nodes":[{"key":"/queue/00000000000000000055","value":"Job1","modifiedIndex":55,"createdIndex":55},{"key":"/queue/00000000000000000056","value":"Job2","modifiedIndex":56,"createdIndex":56},{"key":"/queue/00000000000000000057","value":"Job3","modifiedIndex":57,"createdIndex":57}],"modifiedIndex":55,"createdIndex":55}}
[root@docker ~]# 
~~~


---

### 设定目录过期时间


目录也可以像key一样设定过期

~~~
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true
1454394081
{"action":"set","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:21:31.631806205Z","ttl":10,"modifiedIndex":81,"createdIndex":81}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394086
{"action":"get","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:21:31.631806205Z","ttl":5,"modifiedIndex":81,"createdIndex":81}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394088
{"action":"get","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:21:31.631806205Z","ttl":3,"modifiedIndex":81,"createdIndex":81}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394094
{"errorCode":100,"message":"Key not found","cause":"/testdir","index":82}
[root@docker ~]# 
~~~

---

### 刷新目录过期时间

~~~
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true
1454394195
{"action":"set","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:25.962070512Z","ttl":10,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394200
{"action":"get","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:25.962070512Z","ttl":6,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true -d prevExist=true 
1454394202
{"action":"update","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:32.096351254Z","ttl":10,"modifiedIndex":86,"createdIndex":85},"prevNode":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:25.962070512Z","ttl":4,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394204
{"action":"get","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:32.096351254Z","ttl":8,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true -d prevExist=true 
1454394209
{"action":"update","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:39.280877021Z","ttl":10,"modifiedIndex":87,"createdIndex":85},"prevNode":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:32.096351254Z","ttl":3,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394211
{"action":"get","node":{"key":"/testdir","dir":true,"expiration":"2016-02-02T06:23:39.280877021Z","ttl":8,"modifiedIndex":85,"createdIndex":85}}
[root@docker ~]# date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir 
1454394356
{"errorCode":100,"message":"Key not found","cause":"/testdir","index":88}
[root@docker ~]# 
~~~




---


### 条件操作

**CompareAndSwap (CAS)**  其实就是条件操作，或是判断执行


先进行比较，然后根据比较结果来决定是否进行后面的操作，可以用于构建分布式锁机制(协调服务)

> **Note:**  **CompareAndSwap** 不能用于目录，如果测试用于目录，会获得 **102 "Not a file"** 的错误


Item     | Comment
-------- | ---
prevValue | 比较当前的值
prevIndex | 比较当前的modifiedIndex
prevExist | 检查key是否存在，如果是 **true** 就是一个更新操作; 如果是 **false** 就是一个创建操作


~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one
{"action":"set","node":{"key":"/foo","value":"one","modifiedIndex":89,"createdIndex":89},"prevNode":{"key":"/foo","value":"abc","moddex":54}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevExist=false -XPUT -d value=three
{"errorCode":105,"message":"Key already exists","cause":"/foo","index":89}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevValue=two -XPUT -d value=three
{"errorCode":101,"message":"Compare failed","cause":"[two != one]","index":89}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevValue=one -XPUT -d value=five
{"action":"compareAndSwap","node":{"key":"/foo","value":"five","modifiedIndex":90,"createdIndex":89},"prevNode":{"key":"/foo","value9,"createdIndex":89}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=99 -XPUT -d value=xxx
{"errorCode":101,"message":"Compare failed","cause":"[99 != 90]","index":90}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=90 -XPUT -d value=xxx
{"action":"compareAndSwap","node":{"key":"/foo","value":"xxx","modifiedIndex":91,"createdIndex":89},"prevNode":{"key":"/foo","value"0,"createdIndex":89}}
[root@docker ~]# 
~~~

---

### 条件删除


**Compare-and-Delete(CAD)** 就是判断删除

> **Note:**  **CompareAndDelete** 不能用于目录，如果测试用于目录，会获得 **102 "Not a file"** 的错误

Item     | Comment
-------- | ---
prevValue | 比较当前的值
prevIndex | 比较当前的modifiedIndex

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one
{"action":"set","node":{"key":"/foo","value":"one","modifiedIndex":94,"createdIndex":94},"prevNode":{"key":"/foo","value":"three","modifiedIndex":93,"createdIndex":89}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevValue=two -XDELETE
{"errorCode":101,"message":"Compare failed","cause":"[two != one]","index":94}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo
{"action":"get","node":{"key":"/foo","value":"one","modifiedIndex":94,"createdIndex":94}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=3 -XDELETE
{"errorCode":101,"message":"Compare failed","cause":"[3 != 94]","index":94}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevValue=one -XDELETE
{"action":"compareAndDelete","node":{"key":"/foo","modifiedIndex":95,"createdIndex":94},"prevNode":{"key":"/foo","value":"one","modifiedIndex":94,"createdIndex":94}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo
{"errorCode":100,"message":"Key not found","cause":"/foo","index":95}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one
{"action":"set","node":{"key":"/foo","value":"one","modifiedIndex":96,"createdIndex":96}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=96 -XDELETE
{"action":"compareAndDelete","node":{"key":"/foo","modifiedIndex":97,"createdIndex":96},"prevNode":{"key":"/foo","value":"one","modifiedIndex":96,"createdIndex":96}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo 
{"errorCode":100,"message":"Key not found","cause":"/foo","index":97}
[root@docker ~]#
~~~


---

### 创建目录

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d dir=true
{"action":"set","node":{"key":"/testdir","dir":true,"modifiedIndex":98,"createdIndex":98}}
[root@docker ~]# 
~~~


---

### 目录列举

在etcd中我们可以存储两种类型的对象：键和目录

* 键保存一个字符串
* 目录保存一批键或目录


使用 **recursive=true** ，可以递归显现子目录

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d dir=true
{"action":"set","node":{"key":"/testdir","dir":true,"modifiedIndex":98,"createdIndex":98}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir/abc -XPUT -d dir=true
{"action":"set","node":{"key":"/foo_dir/abc","dir":true,"modifiedIndex":99,"createdIndex":99}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/
{"action":"get","node":{"dir":true,"nodes":[{"key":"/testdir","dir":true,"modifiedIndex":98,"createdIndex":98},{"key":"/abc","value":"jkjkj","modifiedIndex":16,"createdIndex":16},{"key":"/queue","dir":true,"modifiedIndex":55,"createdIndex":55},{"key":"/dir","dir":true,"modifiedIndex":77,"createdIndex":77},{"key":"/foo_dir","dir":true,"modifiedIndex":99,"createdIndex":99},{"key":"/keytest","value":"hello world for etcd test","modifiedIndex":4,"createdIndex":4}]}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/?recursive=true
{"action":"get","node":{"dir":true,"nodes":[{"key":"/keytest","value":"hello world for etcd test","modifiedIndex":4,"createdIndex":4},{"key":"/testdir","dir":true,"modifiedIndex":98,"createdIndex":98},{"key":"/abc","value":"jkjkj","modifiedIndex":16,"createdIndex":16},{"key":"/queue","dir":true,"nodes":[{"key":"/queue/00000000000000000055","value":"Job1","modifiedIndex":55,"createdIndex":55},{"key":"/queue/00000000000000000056","value":"Job2","modifiedIndex":56,"createdIndex":56},{"key":"/queue/00000000000000000057","value":"Job3","modifiedIndex":57,"createdIndex":57}],"modifiedIndex":55,"createdIndex":55},{"key":"/dir","dir":true,"modifiedIndex":77,"createdIndex":77},{"key":"/foo_dir","dir":true,"nodes":[{"key":"/foo_dir/abc","dir":true,"modifiedIndex":99,"createdIndex":99}],"modifiedIndex":99,"createdIndex":99}]}}
[root@docker ~]# 
~~~

---

### 删除目录



如果目录不为空，会报错，加上 **recursive=true** 后就可以递归删除

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir/abc?dir=true  -XDELETE
{"action":"delete","node":{"key":"/foo_dir/abc","dir":true,"modifiedIndex":100,"createdIndex":99},"prevNode":{"key":"/foo_dir/abc","dir":true,"modifiedIndex":99,"createdIndex":99}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir/abc -XPUT -d value=uiuiuii
{"action":"set","node":{"key":"/foo_dir/abc","value":"uiuiuii","modifiedIndex":101,"createdIndex":101}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir?dir=true  -XDELETE
{"errorCode":108,"message":"Directory not empty","cause":"/foo_dir","index":101}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir?recursive=true  -XDELETE
{"action":"delete","node":{"key":"/foo_dir","dir":true,"modifiedIndex":102,"createdIndex":99},"prevNode":{"key":"/foo_dir","dir":true,"modifiedIndex":99,"createdIndex":99}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/foo_dir
{"errorCode":100,"message":"Key not found","cause":"/foo_dir","index":102}
[root@docker ~]# 
~~~

---


### 创建隐藏节点

我们可以通过添加 **\_** 的前缀来创建隐藏的键值对

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/_message -XPUT -d value="Hello hidden"
{"action":"set","node":{"key":"/_message","value":"Hello hidden","modifiedIndex":103,"createdIndex":103}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
{"action":"set","node":{"key":"/message","value":"Hello world","modifiedIndex":104,"createdIndex":104}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/
{"action":"get","node":{"dir":true,"nodes":[{"key":"/queue","dir":true,"modifiedIndex":55,"createdIndex":55},{"key":"/dir","dir":true,"modifiedIndex":77,"createdIndex":77},{"key":"/message","value":"Hello world","modifiedIndex":104,"createdIndex":104},{"key":"/keytest","value":"hello world for etcd test","modifiedIndex":4,"createdIndex":4},{"key":"/testdir","dir":true,"modifiedIndex":98,"createdIndex":98},{"key":"/abc","value":"jkjkj","modifiedIndex":16,"createdIndex":16}]}}
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/_message
{"action":"get","node":{"key":"/_message","value":"Hello hidden","modifiedIndex":103,"createdIndex":103}}
[root@docker ~]# 
~~~


---

### 使用文件内容作为值

可以使用etcd来存储小型的配置文件

JAON 文件，XML文件可以进行直接存储


~~~
[root@docker ~]# echo "Hello\nWorld" > testfile.txt
[root@docker ~]# curl http://127.0.0.1:2379/v2/keys/testfile -XPUT --data-urlencode value@testfile.txt
{"action":"set","node":{"key":"/testfile","value":"Hello\\nWorld\n","modifiedIndex":105,"createdIndex":105}}
[root@docker ~]# 
~~~

---

## 统计

etcd集群对延时，带宽，和运行时间都有统计，通过统计信息，可以了解集群内部的健康状态


### leader统计信息


主节点拥有整个集群的视野，主要关心两类信息

* 到每个节点的延时
* 成功和失败的Raft RPC请求


~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/stats/leader
{"leader":"ce2a822cea30bfca","followers":{}}[root@docker ~]# 
[root@docker ~]#  
~~~

我的环境中还没有followers，所以是空的


下面是官方文档中的示例

**`curl http://127.0.0.1:2379/v2/stats/leader`**


~~~
{
    "followers": {
        "6e3bd23ae5f1eae0": {
            "counts": {
                "fail": 0,
                "success": 745
            },
            "latency": {
                "average": 0.017039507382550306,
                "current": 0.000138,
                "maximum": 1.007649,
                "minimum": 0,
                "standardDeviation": 0.05289178277920594
            }
        },
        "a8266ecf031671f3": {
            "counts": {
                "fail": 0,
                "success": 735
            },
            "latency": {
                "average": 0.012124141496598642,
                "current": 0.000559,
                "maximum": 0.791547,
                "minimum": 0,
                "standardDeviation": 0.04187900156583733
            }
        }
    },
    "leader": "924e2e83e93f2560"
}
~~~

---

### 自身统计

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/stats/self
{"name":"default","id":"ce2a822cea30bfca","state":"StateLeader","startTime":"2016-02-02T10:16:08.734974815+08:00","leaderInfo":{"leader":"ce2a822cea30bfca","uptime":"7h10m25.17655575s","startTime":"2016-02-02T10:16:10.036154166+08:00"},"recvAppendRequestCnt":0,"sendAppendRequestCnt":0}[root@docker ~]# 
~~~


Attribute| Comment
-------- | ---
id|成员的唯一标识
leaderInfo.leader|当前主节点的id
leaderInfo.uptime|主节点的主持时间
name|这个成员的名字
recvAppendRequestCnt|这个节点接收到请求数
recvBandwidthRate| 这个节点的接收Bps(bytes per second)带宽速率 (follower only)
recvPkgRate|这个节点的rps(requests per second)请求速率 (follower only)
sendAppendRequestCnt| 这个节点的发送过的请求数
sendBandwidthRate| 这个节点的发送Bps(bytes per second)带宽速率 (leader only). 在单节点集群中这个属性没有定义.
sendPkgRate|这个节点的rps(requests per second)发送请求速率 (leader only).  在单节点集群中这个属性没有定义.
state| 当前的 leader 或 follower 角色
startTime|这个节点的启动时间



---

### 存储统计

存储统计提供了这个节点上的各种操作的统计信息


> **Note:** v2版本将信息保存在了内存中，节点重启后，信息会丢失

~~~
[root@docker ~]# curl http://127.0.0.1:2379/v2/stats/store
{"getsSuccess":73,"getsFail":13,"setsSuccess":65,"setsFail":8,"deleteSuccess":3,"deleteFail":2,"updateSuccess":9,"updateFail":4,"createSuccess":7,"createFail":1,"compareAndSwapSuccess":2,"compareAndSwapFail":2,"compareAndDeleteSuccess":2,"compareAndDeleteFail":2,"expireCount":17,"watchers":0}[root@docker ~]# 
[root@docker ~]# 
~~~


---

# 命令汇总

* **`curl -L  https://github.com/coreos/etcd/releases/download/v2.2.4/etcd-v2.2.4-linux-amd64.tar.gz -o etcd-v2.2.4-linux-amd64.tar.gz`**
* **`wget https://github.com/coreos/etcd/releases/download/v2.2.4/etcd-v2.2.4-linux-amd64.tar.gz`**
* **`tar -zxvf etcd-v2.2.4-linux-amd64.tar.gz`**
* **`./etcd`**
* **`./etcdctl set keytest "hello world for etcd test"`**
* **`./etcdctl get keytest`**
* **`./etcd`**
* **`netstat  -ant | grep -E '(2379|2380)'`**
* **`curl -L http://127.0.0.1:2379/version`**
* **`curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"`**
* **`curl http://127.0.0.1:2379/v2/keys/abc -XPUT -d value="Hello world"`**
* **`curl http://127.0.0.1:2379/v2/keys/abc -XPUT -d value="abc"`**
* **`curl http://127.0.0.1:2379/v2/keys/message`**
* **`curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="hello etcd"`**
* **`curl http://127.0.0.1:2379/v2/keys/message -XDELETE`**
* **`date ; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=6`**
* **`date ; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=10`**
* **`date; curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl= -d prevExist=true`**
* **`date ; curl http://127.0.0.1:2379/v2/keys/foo`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?wait=true`**
* **`curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=abc`**
* **`curl http://127.0.0.1:2379/v2/keys/foo -vv`**
* **`curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job1`**
* **`curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job2`**
* **`curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job3`**
* **`curl -s 'http://127.0.0.1:2379/v2/keys/queue?recursive=true&sorted=true'`**
* **`date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true`**
* **`date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d ttl=10 -d dir=true -d prevExist=true`**
* **`date +%s ;curl http://127.0.0.1:2379/v2/keys/testdir`**
* **`curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevExist=false -XPUT -d value=three`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevValue=two -XPUT -d value=three`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevValue=one -XPUT -d value=five`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=99 -XPUT -d value=xxx`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=90 -XPUT -d value=xxx`**
* **`curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevValue=two -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=3 -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevValue=one -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one`**
* **`curl http://127.0.0.1:2379/v2/keys/foo?prevIndex=96 -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo`**
* **`curl http://127.0.0.1:2379/v2/keys/testdir -XPUT -d dir=true`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir/abc -XPUT -d dir=true`**
* **`curl http://127.0.0.1:2379/v2/keys/`**
* **`curl http://127.0.0.1:2379/v2/keys/?recursive=true`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir/abc?dir=true  -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir/abc -XPUT -d value=uiuiuii`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir?dir=true  -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir?recursive=true  -XDELETE`**
* **`curl http://127.0.0.1:2379/v2/keys/foo_dir`**
* **`curl http://127.0.0.1:2379/v2/keys/_message -XPUT -d value="Hello hidden"`**
* **`curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"`**
* **`curl http://127.0.0.1:2379/v2/keys/`**
* **`curl http://127.0.0.1:2379/v2/keys/_message`**
* **`echo "Hello\nWorld" > testfile.txt`**
* **`curl http://127.0.0.1:2379/v2/keys/testfile -XPUT --data-urlencode value@testfile.txt`**
* **`curl http://127.0.0.1:2379/v2/stats/leader`**
* **`curl http://127.0.0.1:2379/v2/stats/self`**
* **`curl http://127.0.0.1:2379/v2/stats/store`**

---

[etcd]:https://github.com/coreos/etcd
[raft]:https://raft.github.io/
[coreos_analyse_etcd]:http://www.infoq.com/cn/articles/coreos-analyse-etcd
[raft_move]:http://thesecretlivesofdata.com/raft/
[releases]:https://github.com/coreos/etcd/releases/

