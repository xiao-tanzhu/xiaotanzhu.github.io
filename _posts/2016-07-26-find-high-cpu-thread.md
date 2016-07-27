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
38275 root      20   0 13.068g 1.185g   8228 S  11.6  3.8   3673:10 java                                                                                                                     
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
    3 root      20   0       0      0      0 S   0.0  0.0   2:18.79 ksoftirqd/0                                                                                                              
    5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                                                                                                             
    7 root      20   0       0      0      0 S   0.0  0.0 318:24.44 rcu_sched                                                                                                                
    8 root      20   0       0      0      0 S   0.0  0.0  69:11.07 rcuos/0                                                                                                                  
    9 root      20   0       0      0      0 S   0.0  0.0  59:25.95 rcuos/1                                                                                                                  
   10 root      20   0       0      0      0 S   0.0  0.0 113:30.15 rcuos/2                                                                                                                  
   11 root      20   0       0      0      0 S   0.0  0.0 119:32.66 rcuos/3                                                                                                                  
   12 root      20   0       0      0      0 S   0.0  0.0 118:16.90 rcuos/4                                                                                                                  
   13 root      20   0       0      0      0 S   0.0  0.0 117:20.28 rcuos/5                                                                                                                  
   14 root      20   0       0      0      0 S   0.0  0.0  29:46.64 rcuos/6
```

