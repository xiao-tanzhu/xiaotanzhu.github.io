---
layout: post
title: "千亿数据扛不住，三思后还是从MySQL迁走了……"
date: 2021-05-26
tags: ["MySQL", "MongoDB"]
categories: [数据库]
description: "千亿级数据从MySQL迁移至MongoDB的分析、执行、优化全过程"
---

作者介绍

> 杨亚洲，前滴滴出行专家工程师，现任OPPO文档数据库MongoDB负责人，负责数万亿级数据量文档数据库MongoDB内核研发、性能优化及运维工作，一直专注于分布式缓存、高性能服务端、数据库、中间件等相关研发。后续持续分享《MongoDB内核源码设计、性能优化、最佳运维实践》。

## 前言

线上某IOT核心业务集群之前采用MySQL作为主存储数据库，随着业务规模的不断增加，MySQL已无法满足海量数据存储需求，业务面临着容量痛点、成本痛点问题、数据不均衡问题等。

400亿该业务迁移MongoDB后，同样的数据节省了极大的内存、CPU、磁盘成本，同时完美解决了容量痛点、数据不均衡痛点，并且实现了一定的性能提升。

此外，迁移时候的MySQL数据为400亿，3个月后的现在对应MongoDB集群数据已增长到1000亿，如果以1000亿数据规模等比例计算成本，实际成本节省比例会更高。迁移MongoDB后，除了解决业务痛点问题，同时也促进了业务的快速迭代开发，业务不在关心数据库容量痛点、数据不均衡痛点、成本痛点等问题。

当前国内很多mongod文档资料、性能数据等还停留在早期的MMAP_V1存储引擎，实际上从MongoDB-3.x版本开始，MongoDB默认存储引擎已经采用高性能、高压缩比、更小锁粒度的wiredtiger存储引擎，因此其性能、成本等优势相比之前的MMAP_V1存储引擎更加明显。

## 一、业务迁移背景

该业务在迁移MongoDB前已有约400亿数据，申请了64套MySQL集群，由业务通过shardingjdbc做分库分表，提前拆分为64个库，每个库100张表。主从高可用选举通过依赖开源orchestrator组建，MySQL架构图如下图所示：

![](/images/0011.png)

**说明：**上图中红色代表磁盘告警，磁盘使用水位即将100%。如上图所示，业务一年多前一次性申请了64套MySQL集群，单个集群节点数一主三从，每个节点规格如下：

- cpu：4
- mem：16G
- 磁盘：500G
- 总节点数：64*4=256
- SSD服务器

该业务运行一年多时间后，总集群数据量达到了400亿，并以每月200亿速度增长，由于数据不均衡等原因，造成部分集群数据量大，持续性耗光磁盘问题。由于节点众多，越来越多的集群节点磁盘突破瓶颈，为了解决磁盘瓶颈，DBA不停的提升节点磁盘容量。业务和DBA都面临严重痛点，主要如下：

- 数据不均衡问题
- 节点容量问题
- 成本持续性增加
- DBA工作量剧增(部分磁盘提升不了需要迁移数据到新节点)，业务也提心吊胆

## 二、为何选择MongoDB-附十大核心优势总结

业务遇到瓶颈后，基于MongoDB在公司已有的影响力，业务开始调研MongoDB，通过和业务接触了解到，业务使用场景都是普通的增、删、改、查、排序等操作，同时查询条件都比较固定，用MongoDB完全没任何问题。

此外，MongoDB相比传统开源数据库拥有如下核心优索：

### 优势一：模式自由

MongoDB为schema-free结构，数据格式没有严格限制。业务数据结构比较固定，该功能业务不用，但是并不影响业务使用MongoDB存储结构化的数据。

### 优势二：天然高可用支持

MySQL高可用依赖第三方组件来实现高可用，MongoDB副本集内部多副本通过raft协议天然支持高可用，相比MySQL减少了对第三方组件的依赖。

### 优势三：分布式-解决分库分表及海量数据存储痛点

MongoDB是分布式数据库，完美解决MySQL分库分表及海量数据存储痛点，业务无需在使用数据库前评估需要提前拆多少个库多少个表，MongoDB对业务来说就是一个无限大的表(当前我司最大的表存储数千亿数据，查询性能无任何影响)。

此外，业务在早期的时候一般数据都比较少，可以只申请一个分片MongoDB集群。而如果采用MySQL，就和本次迁移的IOT业务一样，需要提前申请最大容量的集群，早期数据量少的时候严重浪费资源。

### 优势四：完善的数据均衡机制、不同分片策略、多种片建类型支持

- 关于balance：支持自动balance、手动balance、时间段任意配置balance.
- 关于分片策略：支持范围分片、hash分片，同时支持预分片。
- 关于片建类型：支持单自动片建、多字段片建

### 优势五：不同等级的数据一致性及安全性保证

MongoDB在设计上根据不同一致性等级需求，支持不同类型的`Read Concern`、`Write Concern`读写相关配置，客户端可以根据实际情况设置。此外，MongoDB内核设计拥有完善的rollback机制来保证数据安全性和一致性。

### 优势六：高并发、高性能

为了适应大规模高并发业务读写，MongoDB在线程模型设计、并发控制、高性能存储引擎等方面做了很多细致化优化。

### 优势七：wiredtiger高性能存储引擎设计

网上很多评论还停留在早期MMAPv1存储引擎，相比MMAPv1，wiredtiger引擎性能更好，压缩比更高，锁粒度更小，具体如下：

- WiredTiger提供了低延迟和高吞吐量
- 处理比内存大得多的数据，而不会降低性能或资源
- 系统故障后可快速恢复到最近一个checkpoint
- 支持PB级数据存储
- 多线程架构，尽力利用乐观锁并发控制算法减少锁操作
- 具有hot-caches能力
- 磁盘IO最大化利用，提升磁盘IO能力
- 其他

> 更多WT存储引擎设计细节可以参考：
> http://source.wiredtiger.com/3.2.1/architecture.html

### 优势八：成本节省-WT引擎高压缩比支持

MongoDB对数据的压缩支持snappy、zlib算法，在以往线上真实的数据空间大小与真实磁盘空间消耗进行对比，可以得出以下结论：

- MongoDB默认的snappy压缩算法压缩比约为2.2-4.5倍
- zlib压缩算法压缩比约为4.5-7.5倍(本次迁移采用zlib高压缩算法)

![](/images/0001.jpeg)

此外，以线上已有的从MySQL、Es迁移到MongoDB的真实业务磁盘消耗统计对比，同样的数据，存储在MongoDB、MySQL、Es的磁盘占比≈1：3.5：6。

后续会有数千亿hbase数据迁移MongoDB，到时候总结同样数据MongoDB和Hbase的磁盘消耗比。

### 优势九：天然N机房(不管同城还是异地)多活容灾支持

MongoDB天然高可用机制及代理标签自动识别转发功能的支持，可以通过节点不同机房部署来满足同城和异地N机房多活容灾需求，从而实现成本、性能、一致性的“三丰收”。

### 优势十：完善的客户端均衡访问策略

MongoDB客户端访问路由策略由客户端自己指定，该功能通过Read Preference实现，支持primary 、primaryPreferred 、secondary 、secondaryPreferred 、nearest 五种客户端均衡访问策略。

### 补充：分布式事务支持

MongoDB-4.2 版本开始已经支持分布式事务功能，当前对外文档版本已经迭代到 version-4.2.11，分布式事务功能也进一步增强。此外，从 MongoDB-4.4 版本产品规划路线图可以看出，MongoDB 官方将会持续投入开发查询能力和易用性增强功能，例如 union 多表联合查询、索引隐藏等。

> 更多MongoDB核心优势细节详见我分享的一篇文章，也欢迎各位参加讨论：
>
> mongodb源码分析、更多实践案例细节：  
> https://github.com/y123456yz/reading-and-annotate-mongodb-3.6
>
> 话题讨论 | MongoDB 拥有十大核心优势，为何国内知名度远不如 MySQL 高？  
> https://xie.infoq.cn/article/180d98535bfa0c3e71aff1662

## 三、MongoDB资源评估及部署架构

业务开始迁移MongoDB的时候，通过和业务对接梳理，该集群规模及业务需求总结如下：

- 已有数据量400亿左右
- 数据磁盘消耗总和30T左右
- 读写峰值流量4-5W/s左右，流量很小
- 同城两机房多活容灾
- 读写分离
- 每月预计增加200亿数据
- 满足几个月内1500亿新增数据需求

**说明：**数据规模和磁盘消耗按照单副本计算，例如MySQL 64个分片，256个副本，数据规模和磁盘消耗计算方式为：64个主节点数据量之和、64个分片主节点磁盘消耗之和。

### 1、MongoDB资源评估

分片数及存储节点套餐规格选定评估过程如下：

#### 内存评估

我司都是容器化部署，以往经验来看，MongoDB对内存消耗不高，历史百亿级以上MongoDB集群单个容器最大内存基本上都是64Gb，因此内存规格确定为64G。

#### 分片评估

业务流量峰值3-5W/s，考虑到可能后期有更大峰值流量，因此按照峰值10W/s写，5w/s读，也就是峰值15W/s评估，预计需要4个分片。

#### 磁盘评估

MySQL中已有数据400亿，磁盘消耗30T。按照以网线上迁移经验，MongoDB默认配置磁盘消耗约为mysql的1/3-1/5，400亿数据对应MongoDB磁盘消耗预计8T。考虑到1500亿数据，预计4个分片，按照每个分片400亿规模，预计每个分片磁盘消耗8T。

线上单台物理机10多T磁盘，几百G内存，几十个CPU，为了最大化利用服务器资源，我们需要预留一部分磁盘给其他容器使用。另外，因为容器组套餐化限制，最终确定确定单个节点磁盘在7T。预计使用7T的节点，4个分片存储约1500亿数据。

#### CPU规格评估

由于容器调度套餐化限制，因此CPU只能限定为16CPU(实际上用不了这么多CPU)。

#### mongos代理及config server规格评估

此外，由于分片集群还有mongos代理和config server复制集，因此还需要评估mongos代理和config server节点规格。由于config server只主要存储路由相关元数据，因此对磁盘、CUP、MEM消耗都很低；mongos代理只做路由转发只消耗CPU，因此对内存和磁盘消耗都不高。最终，为了最大化节省成本，我们决定让一个代理和一个config server复用同一个容器，容器规格如下：

8CPU/8G内存/50G磁盘，一个代理和一个config server节点复用同一个容器。

- 分片及存储节点规格总结：4分片/16CPU、64G内存、7T磁盘。
- mongos及config server规格总结：8CPU/8G内存/50G磁盘

![](/images/0002.jpeg)

### 2、集群部署架构

由于该业务所在城市只有两个机房，因此我们采用2+2+1(2mongod+2mongod+1arbiter模式)，在A机房部署2个mongod节点，B机房部署2个mongod节点，C机房部署一个最低规格的选举节点，如下图所示：

![](/images/0012.png)

**说明：**

- 每个机房代理部署2个mongos代理，保证业务访问代理高可用，任一代理挂掉，对应机房业务不受影响；
- 如果机房A挂掉，则机房B和机房C剩余2mongod+1arbiter，则会在B机房mongod中从新选举一个主节点。arbiter选举节点不消耗资源；
- 客户端配置nearest ，实现就近读，确保请求通过代理转发的时候，转发到最近网络时延节点，也就是同机房对应存储节点读取数据；
- 弊端：如果是异地机房，B机房和C机房写存在跨机房写场景。如果A、B、C为同城机房，则没用该弊端，同城机房时延可以忽略。

## 四、业务全量+增量迁移方式

![](/images/0003.jpeg)

> 迁移过程由业务自己完成，通过阿里开源的datax工具实现，该迁移工具的更多细节可以参考：
> https://github.com/alibaba/DataX

## 五、性能优化过程

该集群优化过程按照如下两个步骤优化：数据迁移开始前的提前预优化、迁移过程中瓶颈分析及优化、迁移完成后性能优化。

### 1、数据迁移开始前的提前预操作

和业务沟通确定，业务每条数据都携带有一个设备标识ssoid，同时业务查询更新等都是根据ssoid维度查询该设备下面的单条或者一批数据，因此片建选择ssoid。

#### 分片方式

为了充分散列数据到4个分片，因此选择hash分片方式，这样数据可以最大化散列，同时可以满足同一个ssoid数据落到同一个分片，保证查询效率。

#### 预分片

MongoDB如果分片片建为hashed分片，则可以提前做预分片，这样就可以保证数据写进来的时候比较均衡的写入多个分片。预分片的好处可以规避非预分片情况下的chunk迁移问题，最大化提升写入性能。
```javascript
sh.shardCollection("xxx.xxx", {ssoid:"hashed"}, false, { numInitialChunks: 8192} )
```

> 注意事项：切记提前对ssoid创建hashed索引，否则对后续分片扩容有影响。

#### 就近读

客户端增加nearest 配置，从离自己最近的节点读，保证了读的性能。

#### mongos代理配置

A机房业务只配置A机房的代理，B机房业务只配置B机房代理，同时带上nearest配置，最大化的实现本机房就近读，同时避免客户端跨机房访问代理。

#### 禁用`enableMajorityReadConcern`

禁用该功能后ReadConcern majority将会报错，ReadConcern majority功能主要是避免脏读，和业务沟通业务没该需求，因此可以直接关闭。

MongoDB默认使能了`enableMajorityReadConcern`，该功能开启对性能有一定影响，参考：

> MongoDB readConcern 原理解析  
> https://developer.aliyun.com/article/60553

> OPPO百万级高并发MongoDB集群性能数十倍提升优化实践  
> https://mongoing.com/archives/29934

#### 存储引擎cacheSize规格选择

单个容器规格：16CPU、64G内存、7T磁盘，考虑到全量迁移过程中对内存压力，内存碎片等压力会比较大，为了避免OOM，设置cacheSize=42G。

### 2、数据全量迁移过程中优化过程

![](/images/0013.png)

全量数据迁移过程中，迁移速度较块，内存脏数据较多，当脏数据比例达到一定比例后用户读写请求对应线程将会阻塞，用户线程也会去淘汰内存中的脏数据page，最终写性能下降明显。

wiredtiger存储引擎cache淘汰策略相关的几个配置如下:

![](/images/0014.png)

由于业务全量迁移数据是持续性的大流量写，而不是突发性的大流量写，因此eviction_target、eviction_trigger、eviction_dirty_target、eviction_dirty_trigger几个配置用处不大，这几个参数阀值只是在短时间突发流量情况下调整才有用。

但是，在持续性长时间大流量写的情况下，我们可以通过提高wiredtiger存储引擎后台线程数来解决脏数据比例过高引起的用户请求阻塞问题，淘汰脏数据的任务最终交由evict模块后台线程来完成。

全量大流量持续性写存储引擎优化如下：

```javascript
db.adminCommand( { setParameter : 1, "wiredTigerEngineRuntimeConfig" : "eviction=(threads_min=4, threads_max=20)"})
```

### 3、全量迁移完成后，业务流量读写优化

![](/images/0015.png)

前面章节我们提到，在容器资源评估的时候，我们最终确定选择单个容器套餐规格为如下：

> 16CPU、64G内存、7T磁盘。

全量迁移过程中为了避免OOM，预留了约1/3内存给MongoDB server层、操作系统开销等，当全量数据迁移完后，业务写流量相比全量迁移过程小了很多，峰值读写OPS约2-4W/s。

也就是说，前量迁移完成后，cache中脏数据比例几乎很少，基本上不会达到20%阀值，业务读流量相比之前多了很多(数据迁移过程中读流量走原MySQL集群)。为了提升读性能，因此做了如下性能调整(提前建好索引)：

- 节点cacheSize从之前的42G调整到55G，尽量多的缓存热点数据到内存，供业务读，最大化提升读性能；
- 每天凌晨低峰期做一次cache内存加速释放，避免OOM。

上面的内核优后后，业务测时延监控曲线变化，时延更加平稳，平均时延也有25%左右的性能优后，如下图所示：

![](/images/0016.png)

## 六、迁移前后，业务测时延统计对比：(MySQL vs MongoDB)

迁移前业务测时延监控曲线(平均时延7ms, 2月1日数据，此时mysql集群只有300亿数据)：

![](/images/0017.png)

迁移MongoDB后并且业务流量全部切到MongoDB后业务测时延监控曲线(平均6ms, 3月6日数据，此时MongoDB集群已有约500亿数据))

![](/images/0016.png)

总结：

- MySQL(300亿数据)时延：7ms
- MongoDB(500亿数据)时延：6ms

## 七、迁移成本收益对比

### 1、MySQL集群规格及存储数据最大量

![](/images/0019.png)

原mysql集群一共64套，每套集群4副本，每个副本容器规格：4CPU、16G mem、500G磁盘，总共可以存储400亿数据，这时候大部分节点已经开始磁盘90%水位告警，DBA对部分节点做了磁盘容量提升。

总结如下：

- 集群总套数：64
- 单套集群副本数：4
- 每个节点规格：4CPU、16G mem、500G磁盘
- 该64套集群最大存储数据量：400亿

### 2、MongoDB集群规格及存储数据最大量

![](/images/0020.png)

MongoDB从MySQL迁移过来后，数据量已从400亿增加到1000亿，并以每个月增加200亿数据。

MongoDB集群规格及存储数据量总结如下：

- 分片数：4
- 单分片副本数：4
- 每个节点规格：16CPU、64G mem、7T磁盘
- 四个分片存储数据量：当前已存1000亿，最大可存1500亿数据。

### 3、成本对比计算过程

**说明：**由于MySQL迁移MongoDB后，数据不再往MySQL中写入，流量切到MongoDB时候MySQL中大约存储有400亿数据，因此我们以这个时间点做为对比时间点。以400亿数据为基准，资源消耗对比如下表(每个分片只计算主节点资源消耗，因为MySQL和MongoDB都是4副本)：

![](/images/0004.jpeg)

由于MongoDB四个分片还有很多磁盘冗余，该四个分片相比400亿数据，还可以写1100亿数据。如果按照1500亿数据计算，如果还是按照MySQL之前套餐规格，则MySQL集群数需要再增加三倍，也就是总集群套数需要64*4=256套，资源占用对比如下：

![](/images/0005.jpeg)

### 4、收益总结(客观性对比)

从上面的内容可以看出，该业务迁移MongoDB后，除了解决了业务容量痛点、促进业务快速迭代开发、性能提升外，成本还节省了数倍。成本节省总结如下：

400亿维度计算(mysql和MongoDB都存储相同的400亿数据)：

- CPU和内存成本比例：4:1
- 磁盘成本比例：3.3:1

1500亿维度计算(假设mysql集群都采用之前规格等比例换算)：

- CPU和内存成本比例：16:1
- 磁盘成本比例：3.3:1

从上面的分析可以看出，数据量越大，按照等比例换算原则，MongoDB存储成本会更低，原因如下：

#### CPU/内存节省原因：

主要是因为MongoDB海量数据存储及高性能原因，索引建好后，单实例单表即使几百亿数据，读写也是ms级返回(注意：切记查询更新建好索引)。

此外，由于MongoDB分布式功能，对容量评估更加方便，就无需提前一次性申请很多套mysql，而是根据实际需要可以随时加分片。

#### 磁盘节省原因：

MongoDB存储引擎wiredtiger默认高压缩、高性能。

最后，鉴于客观性成本评价，CPU/内存成本部分可能会有争议，比如mysql内存和CPU是否申请的时候就申请过大。MongoDB对应CPU也同样存在该问题，例如申请的单个容器是16CPU，实际上真实只消耗了几个CPU。

但是，磁盘节省是实实在在的，是相同数据情况下mysql和MongoDB的真实磁盘消耗对比。

当前该集群总数据量已经达到近千亿，并以每个月200亿规模增加，单从容器计费层面上换算，1000亿数据按照等比例换算，预计节省成本10倍。

## 八、最后：千亿级中等规模MongoDB集群注意事项

MongoDB无需分库分表，单表可以无限大，但是单表随着数据量的增多会引起以下问题：
- 切记提前建好索引，否则影响查询更新性能(数据越多，无索引查询扫描会越慢)。
- 切记提前评估好业务需要那些索引，单节点单个表数百亿数据，加索引执行时间较长。
- 服务器异常情况下节点替换时间相比会更长。
- 切记数据备份不要采用mongodump/mongorestore方式，而是采用热备或者文件拷贝方式备份。
- 节点替换尽量从备份中拷贝数据加载方式恢复，而不是通过主从全量同步方式，全量同步过程较长。

## 九、未来挑战(该集群未来万亿级实时数据规模挑战)

随着时间推移，业务数据增长也会越来越多，单月数据量增长曲线预计会直线增加(当前每月数据量增加200亿左右)，预计未来2-3年该集群总数据量会达到万亿级，分片数也会达到20个分片左右，可能会遇到各自各样的问题。

但是，IOT业务数据存在明显的冷数问题，一年前的数据用户基本上不会访问，因此我们考虑做如下优后来满足性能、成本的进一步提升：

- 冷数据归档到低成本SATA盘
- 冷数据提升压缩比，最大化减少磁盘消耗

> 如何解决冷数据归档sata盘过程中的性能问题
> 冷热归档存储可以参考之前在dbaplus分享的另一篇文章：
> 1.《用最少人力玩转万亿级数据，我用的就是MongoDB！》
> 2.MongoDB源码分析、更多实践案例细节
> https://github.com/y123456yz/reading-and-annotate-mongodb-3.6

## 十、最后说明(业务场景)

本千亿级IOT业务使用场景总结如下：

- 业务数据读、更新、排序等都可以走索引，包括单字段索引、多字段索引、数组索引，所有查询和更新都能确定走具体的某个最优索引。
- 查询都是单表查询，不涉及多表联合查询。

数据库场景非常重要，脱离业务场景谈数据库优劣无任何意义。例如本文的业务场景，业务能确定需要建那些索引，同时所有的更新、查询、排序都可以对应具体的最优索引，因此该场景就非常适合MongoDB。

每种数据库都有其适合的业务场景，没有万能的数据库。此外，不能因为某种场景不适合而全盘否定没数据库，主流数据库都有其存在的意义，千万不能因为某种场景下的不合适而全盘否定某个数据库。
