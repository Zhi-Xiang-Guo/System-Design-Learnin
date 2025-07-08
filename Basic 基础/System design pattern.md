# The approach

## 「Functional Requirementsand」

Function：确定功能点



## Non-Functional Requirementsand

和面试官达成性能等问题一致（用户量、推导 QPS、具体难点以及技术选型



<details class="lake-collapse"><summary id="ubb7d5bf0"><span class="ne-text">推导例子</span></summary><p id="uad02c79c" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(44, 44, 54)">1 thousand = 1,000 = 10^3 、 1Millon = 1 , 000,000 = 10^6、</span></p><p id="ud7434f13" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(44, 44, 54)">数量级 8Bit ~ Byte ~ 10^-3 kb ~ 10^-3 Mb ~ 10^-3GB</span></p><p id="ue5992bd0" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(44, 44, 54)">数据压缩 ~ CPU 的取舍</span></p><p id="udb93eeb3" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">基础数据类型大小</span></p><p id="u2085b256" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">例子des：</span><span class="ne-text" style="color: rgb(44, 44, 54)">1KB ~ 500words （2B 一字符）</span></p><p id="uc71bc1b4" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"></p><p id="u50960233" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">DAU</span></p><p id="u8307e657" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">QPS TPS</span></p><p id="ud6858244" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">体量：</span></p><p id="ubea7a530" class="ne-p" style="margin: 0; padding: 0; min-height: 24px; text-indent: 2em"><span class="ne-text">用户 100W</span></p><p id="uf31d8995" class="ne-p" style="margin: 0; padding: 0; min-height: 24px; text-indent: 2em"><span class="ne-text">Read avg: 100W * 10 个帖 / day  24*3600 (100k sec -&gt;10K QPS</span></p><p id="u5a43b464" class="ne-p" style="margin: 0; padding: 0; min-height: 24px; text-indent: 2em"><span class="ne-text">Read pead (3~5 times 预估 50K QPS</span></p><p id="ubfc96a3c" class="ne-p" style="margin: 0; padding: 0; min-height: 24px; margin-left: 2em"><span class="ne-text">单机 MySQL 配置在  为 read:1-5k （注重数量级即可)-&gt;（复制 主从 or 分片</span></p></details>



Scale

High available

Scaling capacity

数据分区

功能分区

增加副本



分片：

添加删除分片问题（一致性 hash

跨分片查询

scatter and gather （分区时 get useid 的文章 先拉 all 再 聚合）

latency

cache (缓存 post）

大小预估（注重数量级)

缓存 3 天的热门 post

太小 20%-80%流量 存 20% 【一个服务器 100G 也是随便的吧 看机器

miss

更快（预构件的 timeline 、

缓存 post_id user_id 即可减少 join 效率还是很高的 存储没问题

看 qps ->

写入会放大 -> hot spot



我们的设计就是往三高上靠即可：**高性能、高可靠、高可用。**

我总结的这套思考方式将回答总结成3+1。3代表按照接入层、逻辑层、存储层的顺序递进思考；1代表需要完成的核心业务功能。

我将通用的这部分划分为以下几个层级。

**1.接入层**：负责流量的分发，如果面试官提到了百万QPS，不用想，我们肯定要在这里跟他吹一下你所了解的负载均衡工具，比如nginx、lvs、f5等。这和海量数据题利用了相同的思路就是分治。

**2.逻辑层**：在这里我们要完成核心业务功能。需要考虑引入什么中间件、什么缓存架构等。这一部分负责处理拆分后的流量。常见的思路有MQ异步解耦、本地-Redis-数据库多级缓存、CDN加速等。

**3.数据存储层**：我们需要考虑百万级别QPS的存储问题，这里一定是分布式的数据库，可以考虑分库分表、对于对象存储可以考虑元信息和内容信息的分离、冷热数据分离、备份等。

**4.核心业务功能**：给出设计方案并进行设计方案的对比



DB：

1. NoSQL
2. REDB
3. In-memory

| **指标**     | **Redis**      | **MySQL**     | **适用场景**                   |
| ------------ | -------------- | ------------- | ------------------------------ |
| **读吞吐量** | 10万+ QPS      | 1万~2万 QPS   | 高频读（缓存、会话管理）       |
| **写吞吐量** | 5万+ QPS       | 500~2000 QPS  | 低频写（持久化存储、事务操作） |
| **延迟**     | 微秒级         | 毫秒级        | 实时性要求高的场景             |
| **数据安全** | 依赖持久化配置 | ACID 强一致性 | 财务、交易类业务               |

## 

| **维度**   | **技术/策略**      | **说明**                                                     |
| ---------- | ------------------ | ------------------------------------------------------------ |
| **高性能** | （多级）Redis 缓存 | 通过内存缓存热点数据，减少数据库访问，提升响应速度。         |
|            | 异步解耦           | 通过消息队列（如Kafka）解耦服务，提升系统扩展性和响应速度（同时涉及性能与可用性）。 |
|            | 分片扩展           | 将数据分散到多个节点（如数据库分库分表），提高系统吞吐量和并行处理能力。 |
|            | 读写分离           | 主库写、从库读，分摊数据库负载，提升查询性能。               |
| **高可用** | 多活部署           | 跨机房/地域部署服务，流量可切换，避免单点故障（如异地多活架构）。 |
|            | 服务容灾           | 故障自动转移（如Kubernetes容器重启、数据库主从切换）。       |
|            | 限流熔断           | 通过限流（如令牌桶）和熔断（如Hystrix）保护系统不被突发流量击垮。 |
|            | 灰度发布           | 逐步发布新版本，监控无异常后全量上线，减少故障影响范围。     |
| **高可靠** | 消息幂等           | 确保消息重复消费时结果一致（如唯一ID+去重表）。              |
|            | 分布式事务         | 保证跨服务数据一致性（如TCC、Saga、Seata）。                 |
|            | 数据多副本         | 通过主从复制或分布式共识算法（如Raft）实现数据冗余，防止丢失。 |
|            | 数据校验           | 校验数据完整性（如CRC校验、区块链哈希链），防止脏数据或篡改。 |
| **其他**   |                    |                                                              |



三高目标（性能、可用、可靠）  

三层结构（接入层、逻辑层、存储层）  

三手段（分布式、缓存、解耦）  

一核心（业务驱动）

1. 高性能：快 —— 缓存 + 异步 + 分布式
2. 高可用：稳 —— 多副本 + 自动切换 + 容灾限流
3. 高可靠：准 —— 幂等 + 事务 + 数据持久 + 审计







## Design Core Entities

## Design API or Interface

## Data Flow -> High-level Design

Layers

Clinet Server DB





高并发：RT TPS QPS 吞吐

压测看数据，

# Tinder 

Miss Match：定时捞

redis保原子



合并一条Swipe 



md user1 user2 分区不在同一机器 原子性问题



一致性：

saga



# Tiwtter 推荐 feed 流

worker-poll

push 模型【写入 post 时，get followers 广播】

优点：

速度快 已经预构建

缺点

follwers 多，eg 明星，数量级放大 hot spot

缩小范围：只对活跃用户（预构建）

pull 拉取【登入 再找好友 再获取 post

缺点

延迟高

若 follow 太多 ，查询 合并 排序所需时间多

优点

登入才消耗 ，

无法得知关注的某人发了新的帖子（notification 做不了）



混合模型

小用户 

push 直接读取

大 V 

pull mode 、while followers logging to pull 大 V's post



# 设计一个百万级TPS的红包系统

**1.接入层（网络负载层）**：搭建多个数据中心，并通过负载均衡工具比如nginx、lvs、f5（处理量分别是万级、十万级、百万级）对流量进行分散，不同数据中心只承载一部分流量。如果需要用到redis(这种级别的服务几乎强依赖于redis)，拆分后的流量按照单机redis能处理8wtps的能力组建redis集群。

**2.逻辑层**：将红包拆分，并使用redis的list进行保存（key为红包id，list是拆分的金额），抢红包就是使用lua脚本pop这个list，并将结果存到用户维度的队列（key为uid+红包id，红包金额）。定时任务定时拉取用户队列进行批量入库操作，并发送mq通知结果。

**3.存储层**：mysql集群按照uid维度进行分库，存储用户已经抢到的红包列表。近期数据存MySQL，历史数据迁移至HBase，结合时间分表（如按月分表）控制单表规模

**4.核心业务功能：**

比如设计红包雨这个功能核心功能就是红包拆分，我们通过积累知道有一些俩种方式。

1. 二倍均值法：剩余红包金额为M，剩余人数为N，每次抢到的金额=随机区间（0，M/N*2）。可以通过证明每个人抢到的金额期望上是相等的。是公平的算法。
2. 线段分割法：将红包金额看作有一定长度的线段，N个用户抢红包就是确定N-1个分割点，用户对应的区间长度就是抢到的金额。

优缺点分析：二倍均值法实现简单，空间复杂度低，但是用户间抢到的金额不会有太大差距，因此给用户的惊喜较少。线段分割法实现复杂，但是金额差距大，娱乐性更强。