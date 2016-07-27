---
layout: post
title: 在生产环境中查出占用大量CPU的Java线程
tags:
- Linux
- CPU
- JVM
- Java
categories: Linux
description: 在生产环境中查出占用大量CPU的线程
---
生产环境中，有的时候会发现某一个Java进程占用了大量CPU，在测试环境又很难重现。这时候就需要在线进行保护现场和Debug。

### 定位大量占用CPU的进程
执行`top`命令，然后按`P`按照CPU使用率排序
```
top - 16:08:03 up 54 days, 20:22,  1 user,  load average: 0.67, 1.00, 1.01
Tasks: 477 total,   2 running, 475 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.3 us,  1.3 sy,  0.3 ni, 96.5 id,  0.4 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem:  32898100 total, 29648584 used,  3249516 free,   474052 buffers
KiB Swap: 33505276 total,  4258368 used, 29246908 free.  8192840 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                  
36730 jenkins   20   0 13.068g 1.185g   8228 S  11.6  3.8   3673:10 java                                                                                                                     
35378 root      20   0  110784  34768   3748 S   5.0  0.1 249:51.07 gunicorn                                                                                                                 
39103 root      39  19    7980   3172   1524 S   3.3  0.0   0:13.06 apps.plugin                                                                                                              
 1329 mongodb   20   0 28.476g 122940  55292 S   1.7  0.4 286:41.33 mongod                                                                                                                   
 2073 redis     20   0  495752 181572   2024 S   1.7  0.6 213:49.77 redis-server                                                                                                             
 5053 root      20   0  110956  15216   2376 S   1.7  0.0 228:35.26 gunicorn                                                                                                                 
14444 root      20   0  110784  34720   3636 S   1.7  0.1 122:03.66 gunicorn                                                                                                                 
20403 ubuntu    20   0   82616   2644   2564 S   1.7  0.0  11:49.45 zabbix_agentd                                                                                                            
35032 root      20   0  110528  34944   4088 S   1.7  0.1 134:09.61 gunicorn                                                                                                                 
40879 ubuntu    20   0  103576   3356   2400 S   1.7  0.0   0:00.01 sshd                                                                                                                     
45221 root      20   0 8979028 921240  15540 S   1.7  2.8   4:38.85 java                                                                                                                     
45666 root      20   0 8704800 748200  17084 S   1.7  2.3   3:53.45 java                                                                                                                     
    1 root      20   0   33732   3892   2448 S   0.0  0.0   1:49.23 init                                                                                                                     
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.66 kthreadd                                                                                                                 
    3 root      20   0       0      0      0 S   0.0  0.0   2:18.79 
```
可以看到，Jenkins占用了较大的CPU资源，进程ID为**36730**

### 找到占用资源最多的线程

执行以下命令显示**36730**的所有线程ID：
```bash
ubuntu@linasvr:~$ ps -mp 36730 -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID     TIME
jenkins   0.2   -    - -         -      -     - 03:18:59
jenkins   0.0  19    - futex_    -      - 36730 00:00:00
jenkins   0.0  19    - futex_    -      - 32117 00:00:01
jenkins   0.0  19    - futex_    -      - 32118 00:00:01
jenkins   0.0  19    - futex_    -      - 32119 00:00:01
jenkins   0.0  19    - futex_    -      - 32120 00:00:01
jenkins   0.0  19    - futex_    -      - 32121 00:00:01
jenkins   0.0  19    - futex_    -      - 32122 00:00:01
jenkins   0.0  19    - futex_    -      - 32123 00:00:01
jenkins   0.0  19    - futex_    -      - 32124 00:00:01
jenkins   0.0  19    - futex_    -      - 32125 00:00:01
jenkins   0.0  19    - futex_    -      - 32126 00:00:01
jenkins   0.0  19    - futex_    -      - 32127 00:00:01
jenkins   0.0  19    - futex_    -      - 32128 00:00:07
jenkins   0.0  19    - futex_    -      - 32129 00:00:00
jenkins   0.0  19    - futex_    -      - 32131 00:00:00
jenkins   0.0  19    - futex_    -      - 32132 00:00:00
...
```
注：这里只是演示命令，其实没有异常的线程。

### 查看某线程正在做什么

假设我们要查看**43255**线程正在执行什么代码，首先需要将**43255**转换为16进制表示：
```
ubuntu@linasvr:~$ printf "%x\n" 43255
a8f7
```
然后可以使用**jstack**查看**36730**中的**a8f7**线程在执行什么：
```bash
sudo -u jenkins -H jstack 36730 |grep a8f7 -A 30
```
输出以下内容：
```
"RemoteInvocationHandler [#1]" #125 daemon prio=5 os_prio=0 tid=0x00007fe2ac017000 nid=0xa8f7 in Object.wait() [0x00007fe34e64e000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x00000005c3c49870> (a java.lang.ref.ReferenceQueue$Lock)
        at hudson.remoting.RemoteInvocationHandler$Unexporter.run(RemoteInvocationHandler.java:415)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at hudson.remoting.AtmostOneThreadExecutor$Worker.run(AtmostOneThreadExecutor.java:110)
        at java.lang.Thread.run(Thread.java:745)

"Thread-12" #118 daemon prio=5 os_prio=0 tid=0x00007fe2ac020000 nid=0xa89c runnable [0x00007fe34e44c000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:170)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at com.trilead.ssh2.crypto.cipher.CipherInputStream.fill_buffer(CipherInputStream.java:41)
        at com.trilead.ssh2.crypto.cipher.CipherInputStream.internal_read(CipherInputStream.java:52)
        at com.trilead.ssh2.crypto.cipher.CipherInputStream.getBlock(CipherInputStream.java:79)
        at com.trilead.ssh2.crypto.cipher.CipherInputStream.read(CipherInputStream.java:108)
        at com.trilead.ssh2.transport.TransportConnection.receiveMessage(TransportConnection.java:232)
        at com.trilead.ssh2.transport.TransportManager.receiveLoop(TransportManager.java:693)
        at com.trilead.ssh2.transport.TransportManager$1.run(TransportManager.java:489)
        at java.lang.Thread.run(Thread.java:745)

"Scheduler-248609774" #92 prio=5 os_prio=0 tid=0x00007fe2fc007000 nid=0x91d5 waiting on condition [0x00007fe34e24a000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000005c002e830> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
```
可以看到`nid=0xa8f7`线程的调用栈。如果有问题的话一目了然。

### jstack的权限问题
如果执行jstack发现以下异常：
```
ubuntu@linasvr:~$ sudo jstack 38275
38275: Unable to open socket file: target process not responding or HotSpot VM not loaded
The -F option can be used when the target process is not responding
```
或者：
```
ubuntu@linasvr:~$ sudo jstack 36730
36730: well-known file is not secure
```
那么八成是权限的问题。我们可以用`sudo -u`命令使用某特定用户执行jstack。比如以上的例子，我们使用jenkins用户来执行`jstack`命令：
```bash
sudo -u jenkins -H jstack 36730
```

