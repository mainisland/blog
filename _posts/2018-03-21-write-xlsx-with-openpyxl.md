---
layout: post
title: "Write xlsx with Openpyxl"
author:  wilmosfang
date: 2018-03-21 14:15:46
image: '/assets/img/'
excerpt: '使用 Openpyxl 来写 xlsx 文件'
main-class: python
color: '#265277'
tags:
 - python
 - openpyxl
 - pip
categories:
 - python
twitter_text: 'Write xlsx file with python Openpyxl'
introduction: 'the simple way to write xlsx file with python Openpyxl'
---




## 前言

**[Openpyxl][openpyxl]** 是一个用来读写 **`Excel 2010 xlsx/xlsm/xltx/xltm`** 文件的开源库

> A Python library to read/write Excel 2010 xlsx/xlsm files

它的诞生是为了解决 **Python** 没有原生的读取 **Office Open XML** 格式库的问题

**[Openpyxl][openpyxl]** 是基于 **PHPExcel** 开发出来的

这里演示一下如何傅用 **[Openpyxl][openpyxl]** 来修改和写入 **xlsx** 文件

> **Tip:** 当前的版本为 **openpyxl-2.5.1**

---

# 操作


## 环境

~~~
[root@56-201 ~]# hostnamectl 
   Static hostname: 56-201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: b278707b56304e11a4f30711cf56d76b
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@56-201 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0e:38:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81534sec preferred_lft 81534sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:bb:5d:54 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.201/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febb:5d54/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@56-201 ~]# 
~~~


## 表格

![openpyxl](/assets/img/openpyxl/openpyxl01.png)

![openpyxl](/assets/img/openpyxl/openpyxl02.png)

## 安装 python

这里我用新版本的 python

~~~
[root@56-201 ~]# yum install python36.x86_64
Loaded plugins: fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
c7-media                                                 | 3.6 kB     00:00     
centos-openshift-origin15                                | 2.9 kB     00:00     
epel/x86_64/metalink                                     | 7.0 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
openresty                                                | 2.9 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): epel/x86_64/updateinfo                              | 905 kB   00:07     
(2/2): epel/x86_64/primary_db                              | 6.3 MB   00:12     
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * c7-media: 
 * epel: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package python36.x86_64 0:3.6.3-7.el7 will be installed
--> Processing Dependency: python36-libs(x86-64) = 3.6.3-7.el7 for package: python36-3.6.3-7.el7.x86_64
--> Processing Dependency: libpython3.6m.so.1.0()(64bit) for package: python36-3.6.3-7.el7.x86_64
--> Running transaction check
---> Package python36-libs.x86_64 0:3.6.3-7.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch           Version                Repository    Size
================================================================================
Installing:
 python36              x86_64         3.6.3-7.el7            epel          64 k
Installing for dependencies:
 python36-libs         x86_64         3.6.3-7.el7            epel         9.1 M

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 9.1 M
Installed size: 40 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): python36-3.6.3-7.el7.x86_64.rpm                     |  64 kB   00:01     
(2/2): python36-libs-3.6.3-7.el7.x86_64.rpm                | 9.1 MB   00:15     
--------------------------------------------------------------------------------
Total                                              617 kB/s | 9.1 MB  00:15     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python36-libs-3.6.3-7.el7.x86_64                             1/2 
  Installing : python36-3.6.3-7.el7.x86_64                                  2/2 
  Verifying  : python36-3.6.3-7.el7.x86_64                                  1/2 
  Verifying  : python36-libs-3.6.3-7.el7.x86_64                             2/2 

Installed:
  python36.x86_64 0:3.6.3-7.el7                                                 

Dependency Installed:
  python36-libs.x86_64 0:3.6.3-7.el7                                            

Complete!
[root@56-201 ~]# 
~~~


## 安装 pip


~~~
[root@56-201 ~]# python3.6 get-pip.py 
Collecting pip
  Using cached pip-9.0.2-py2.py3-none-any.whl
Collecting setuptools
  Using cached setuptools-39.0.1-py2.py3-none-any.whl
Collecting wheel
  Using cached wheel-0.30.0-py2.py3-none-any.whl
Installing collected packages: pip, setuptools, wheel
Successfully installed pip-9.0.2 setuptools-39.0.1 wheel-0.30.0
[root@56-201 ~]# pip install ipython 
Collecting ipython
  Downloading ipython-6.2.1-py3-none-any.whl (745kB)
    100% |████████████████████████████████| 747kB 1.2MB/s 
Collecting pickleshare (from ipython)
  Using cached pickleshare-0.7.4-py2.py3-none-any.whl
Collecting pygments (from ipython)
  Downloading Pygments-2.2.0-py2.py3-none-any.whl (841kB)
    100% |████████████████████████████████| 849kB 1.1MB/s 
Collecting simplegeneric>0.8 (from ipython)
  Using cached simplegeneric-0.8.1.zip
Collecting pexpect; sys_platform != "win32" (from ipython)
  Downloading pexpect-4.4.0-py2.py3-none-any.whl (56kB)
    100% |████████████████████████████████| 61kB 5.0MB/s 
Collecting decorator (from ipython)
  Downloading decorator-4.2.1-py2.py3-none-any.whl
Requirement already satisfied: setuptools>=18.5 in /usr/lib/python3.6/site-packages (from ipython)
Collecting prompt-toolkit<2.0.0,>=1.0.4 (from ipython)
  Downloading prompt_toolkit-1.0.15-py3-none-any.whl (247kB)
    100% |████████████████████████████████| 256kB 1.6MB/s 
Collecting traitlets>=4.2 (from ipython)
  Using cached traitlets-4.3.2-py2.py3-none-any.whl
Collecting jedi>=0.10 (from ipython)
  Downloading jedi-0.11.1-py2.py3-none-any.whl (250kB)
    100% |████████████████████████████████| 256kB 2.0MB/s 
Collecting ptyprocess>=0.5 (from pexpect; sys_platform != "win32"->ipython)
  Downloading ptyprocess-0.5.2-py2.py3-none-any.whl
Collecting six>=1.9.0 (from prompt-toolkit<2.0.0,>=1.0.4->ipython)
  Downloading six-1.11.0-py2.py3-none-any.whl
Collecting wcwidth (from prompt-toolkit<2.0.0,>=1.0.4->ipython)
  Using cached wcwidth-0.1.7-py2.py3-none-any.whl
Collecting ipython-genutils (from traitlets>=4.2->ipython)
  Using cached ipython_genutils-0.2.0-py2.py3-none-any.whl
Collecting parso==0.1.1 (from jedi>=0.10->ipython)
  Downloading parso-0.1.1-py2.py3-none-any.whl (91kB)
    100% |████████████████████████████████| 92kB 74kB/s 
Building wheels for collected packages: simplegeneric
  Running setup.py bdist_wheel for simplegeneric ... done
  Stored in directory: /root/.cache/pip/wheels/7b/31/08/c85e74c84188cbec6a6827beec4d640f2bd78ae003dc1ec09d
Successfully built simplegeneric
Installing collected packages: pickleshare, pygments, simplegeneric, ptyprocess, pexpect, decorator, six, wcwidth, prompt-toolkit, ipython-genutils, traitlets, parso, jedi, ipython
Successfully installed decorator-4.2.1 ipython-6.2.1 ipython-genutils-0.2.0 jedi-0.11.1 parso-0.1.1 pexpect-4.4.0 pickleshare-0.7.4 prompt-toolkit-1.0.15 ptyprocess-0.5.2 pygments-2.2.0 simplegeneric-0.8.1 six-1.11.0 traitlets-4.3.2 wcwidth-0.1.7
[root@56-201 ~]# echo $?
0
[root@56-201 ~]# 
~~~


## 安装 openpyxl

~~~
[root@56-201 ~]# pip install openpyxl
Collecting openpyxl
Collecting et-xmlfile (from openpyxl)
  Using cached et_xmlfile-1.0.1.tar.gz
Collecting jdcal (from openpyxl)
  Using cached jdcal-1.3.tar.gz
Building wheels for collected packages: et-xmlfile, jdcal
  Running setup.py bdist_wheel for et-xmlfile ... done
  Stored in directory: /root/.cache/pip/wheels/99/f6/53/5e18f3ff4ce36c990fa90ebdf2b80cd9b44dc461f750a1a77c
  Running setup.py bdist_wheel for jdcal ... done
  Stored in directory: /root/.cache/pip/wheels/0f/63/92/19ac65ed64189de4d662f269d39dd08a887258842ad2f29549
Successfully built et-xmlfile jdcal
Installing collected packages: et-xmlfile, jdcal, openpyxl
Successfully installed et-xmlfile-1.0.1 jdcal-1.3 openpyxl-2.5.1
[root@56-201 ~]# 
~~~

## 导入模块

~~~
[root@56-201 ~]# ipython
Python 3.6.3 (default, Jan  4 2018, 16:40:53) 
Type 'copyright', 'credits' or 'license' for more information
IPython 6.2.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from openpyxl import load_workbook

In [2]:
~~~


### 读取内容

~~~
In [2]: wb = load_workbook(filename='/tmp/test.xlsx', read_only=True)

In [3]: ws=wb['test2']

In [4]: for row in ws.rows:
   ...:     for cell in row:
   ...:         print("%s "%cell.value,end='')
   ...:     print('')
   ...:     
e f g h 
1 2 3 4 
5 6 7 8 

In [5]: 
~~~

### 写入内容

~~~
In [1]: from openpyxl import Workbook

In [2]: wb = Workbook(write_only=True)

In [3]: ws = wb.create_sheet()

In [4]: ws.append([1,2,3,4,5])

In [5]: ws.append(['just','for','test'])

In [6]: ws.append(['hello','openpyxl'])

In [7]: ws.append(['1.2','2.3','2017/1/1'])

In [8]: wb.save('/tmp/fortest.xlsx')

In [9]: ls -l /tmp/fortest.xlsx
-rw-r--r-- 1 root root 4794 3月  21 23:15 /tmp/fortest.xlsx

In [10]:
~~~

保存退出后进行查看

![openpyxl](/assets/img/openpyxl/openpyxl03.png)


和前面写入的内容一致


### 读写单元格内容

~~~
In [1]: from openpyxl import load_workbook

In [2]: wb = load_workbook(filename='/tmp/test.xlsx')

In [3]: wb.sheetnames
Out[3]: ['test1', 'test2']

In [4]: ws=wb['test2']

In [5]: ws.title
Out[5]: 'test2'

In [6]: ws.title='new_title'

In [7]: ws.title
Out[7]: 'new_title'

In [8]: wb.sheetnames
Out[8]: ['test1', 'new_title']

In [9]: [x.value for x in ws[1]]
Out[9]: ['e', 'f', 'g', 'h']

In [10]: [x.value for x in ws[2]]
Out[10]: [1, 2, 3, 4]

In [11]: [x.value for x in ws[3]]
Out[11]: [5, 6, 7, 8]

In [12]: for r in range(1,10):
    ...:     for c in range(1,10):
    ...:         ws.cell(r,c).value=r*c
    ...:         

In [13]: for row in ws.rows:
    ...:     for cell in row:
    ...:         print("%4s "%cell.value,end='')
    ...:     print('')
    ...:     
   1    2    3    4    5    6    7    8    9 
   2    4    6    8   10   12   14   16   18 
   3    6    9   12   15   18   21   24   27 
   4    8   12   16   20   24   28   32   36 
   5   10   15   20   25   30   35   40   45 
   6   12   18   24   30   36   42   48   54 
   7   14   21   28   35   42   49   56   63 
   8   16   24   32   40   48   56   64   72 
   9   18   27   36   45   54   63   72   81 

In [14]: 

~~~



### 保存退出

~~~
In [15]: wb.save('/tmp/newtest.xlsx')

In [16]: quit
[root@56-201 ~]# 
~~~

拷贝到本地

~~~
wilmos@Nothing:~$ scp root@h201:/tmp/newtest.xlsx /tmp/
newtest.xlsx                                  100% 6963     6.8KB/s   00:00    
wilmos@Nothing:~$ 
~~~

可以看到内容如预期发生了变化(包括了sheetname)

![openpyxl](/assets/img/openpyxl/openpyxl04.png)

而另一张表的内容没有发生任何变化

![openpyxl](/assets/img/openpyxl/openpyxl05.png)



到此已经借用 Openpyxl 完成了对于 xlsx 的最基本的写操作

---

# 总结

结构如下

workbook 然后是 worksheet 然后是 cell

cell 有两个维度 row 和 column 

两个维度定位了一个 cell 后可以用 value 来取出其中的值进行读或写


* TOC
{:toc}


---



[openpyxl]:https://openpyxl.readthedocs.io/en/stable/

