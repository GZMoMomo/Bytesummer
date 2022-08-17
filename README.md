# Bytesummer
# 第四届字节跳动夏令营笔记

![image](https://user-images.githubusercontent.com/91240419/180639073-33fb3c74-3823-4bcc-b08f-7b7f0700877f.png)
消息队列一般用于解耦计算与存储
## 1.SQL 查询优化器浅析
![image](https://user-images.githubusercontent.com/91240419/180648530-5a66e88f-67eb-4bc6-95db-875cbc8b1114.png)
## SQL的处理流程
![image](https://user-images.githubusercontent.com/91240419/180639390-84292810-2b7a-4a6a-b108-17c91933c9fa.png)
### Parser 语法分析器
#### 抽象语法树（Abstract Syntax Tree，AST）  
词法分析：拆分字符串，得到关键字、数值常量、字符串常量、运算符号等token  
语法分析：将token组成AST node，最终得到一个AST
![image](https://user-images.githubusercontent.com/91240419/180639680-dca51533-7780-4a65-87d3-f49f5db574ee.png)
### Analyzer 
作用：  
+ 检查并绑定Database，Table，Column等原信息  
+ 检查SQL合法性，比如max、min、avg的输入是数值  
AST->Logical Plan
#### 逻辑计划（Logical Plan） 
![image](https://user-images.githubusercontent.com/91240419/180640113-d27cf196-35bd-4cf9-99bc-b5eba258a2e6.png)
### 查询优化
SQL是声明式语言，用户只描述了做什么，没有告诉数据库怎么做  
作用：  
找到一个正确且代价最小的物理执行计划
#### 物理计划（Physical Plan）  
分为：  
+ Plan Fragment：执行计划子树 
+ 最小化网络数据传输，把逻辑计划拆分成多个物理计划  
+ 查询优化器需要感知数据分布，利用数据的物理分布（数据亲和性）  
+ 增加Shuffle算子  
![image](https://user-images.githubusercontent.com/91240419/180640483-a4ac9846-3f55-476c-b1d9-1e9b1e472dab.png)  
Executer  
单机并行：cache、pipeline、SIMD  
多机并行：一个fragment对应多个实例  

## 常见的查询优化器
### 查询优化器分类
![image](https://user-images.githubusercontent.com/91240419/180641624-b6c9872c-ee9c-4616-a516-2b4c82e9d45a.png)
### RBO
基于经验归纳得到的优化规则  
实现简单，优化速度快  
无法保证最优的执行计划  
-关系代数  
-优化内容：  
减少I/O，减少Network传输，减少CPU和内存的使用量  
#### -优化方法：
-列裁剪  
对于查询和算子，裁剪掉不需要的列，减少I/O和内存的占用
![image](https://user-images.githubusercontent.com/91240419/180642045-8f619fee-101c-40d6-a22c-e46a392c2558.png)
从上到下扫描需要过滤的条件  
-谓词下推  
![image](https://user-images.githubusercontent.com/91240419/180642661-26f68e22-499a-4057-96fd-40163ecb010d.png)
-传递闭包
![image](https://user-images.githubusercontent.com/91240419/180642949-0b2768e9-53de-475b-ad6d-56eec3cbf3ea.png)
-Runtime Filter
在执行时才能产生的过滤器（在已经过滤后的表中检阅数据，生成新的过滤规则用于需要join的另一张表）  
min-max：最大最小值范围过滤器  
in-list：当数据量小的时候，可以生成一个数据集过滤器  
bloom filter：创建一个bloom filter表，说明该数据在与不在，在扫描数据时先查询bloom filter，若不在其中则不需要该数据  
![image](https://user-images.githubusercontent.com/91240419/180643733-eedb6ef5-16cf-437f-9e83-4050f8d936ef.png)

### CBO
+ 使用一个模型估算执行计划的代价，选择代价最小的执行计划  
+ 执行计划的代价：所有算子的执行代价之和  
+ 算子代价：CPU,内存，磁盘I/O，网络I/O等代价    
+ 与算计的类型和输入数据的统计信息有关（输入输出的行数大小）  
#### 如何收集统计信息
![image](https://user-images.githubusercontent.com/91240419/180646730-a97b7709-cf6e-4d0f-840f-dd7fca0055f2.png)
-CBO的枚举执行计划  
- 动态规划和贪心算法  
- 哈希连接（Hash Join）：将其中一个表的连接字段计算出一个哈希表，然后从另一个表中一次获取记录并计算哈希值，根据两个哈希值来匹配符合条件的记录。这种方式在数据量大且没有创建索引的情况下的性能可能更好。
- 排序合并连接（Sort Merge Join）：首先将两个表中的数据基于连接字段分别进行排序，然后合并排序后的结果。这种方式通常用于没有创建索 引，并且数据已经排序的情况。


## 2.流/批/OLAP 一体的 Flink 引擎介绍
### Flink分层架构
![image](https://user-images.githubusercontent.com/91240419/181015794-7692f649-d837-4654-afb0-d0c35d604383.png)
- SDK 层：Flink's APIs Overview；  
- 执行引擎层（Runtime 层）：执行引擎层提供了统一的 DAG，用来描述数据处理的 Pipeline，不管是流还是批，都会转化为 DAG 图，调度层再把 DAG 转化成分布式环境下的 Task，Task 之间通过 Shuffle 传输数据；  
- 调度：Jobs and Scheduling；  
- Task 生命周期：Task Lifecycle；  
- Flink Failover 机制：Task Failure Recovery；  
- Flink 反压概念及监控：Monitoring Back Pressure；  
- Flink HA 机制：Flink HA Overview；  
- 状态存储层：负责存储算子的状态信息  
### Flink整体架构
![image](https://user-images.githubusercontent.com/91240419/181015949-dbd553b1-34ed-4706-806d-4deb01b1f484.png)
![image](https://user-images.githubusercontent.com/91240419/181016019-78c0da08-eafc-40a8-b2fe-3d317e14a7d6.png)
#### JobManager（JM）负责整个任务的协调工作，包括：调度 task、触发协调 Task 做 Checkpoint、协调容错恢复等，核心有下面三个组件：
- Dispatcher: 接收作业，拉起 JobManager 来执行作业，并在 JobMaster 挂掉之后恢复作业；  
- JobMaster: 管理一个 job 的整个生命周期，会向 ResourceManager 申请 slot，并将 task 调度到对应 TM 上；  
- ResourceManager：负责 slot 资源的管理和调度，Task manager 拉起之后会向 RM 注册；  
- TaskManager（TM）：负责执行一个 DataFlow Graph 的各个 task 以及 data streams 的 buffer 和数据交换。  
### --
Shuffle：在分布式计算中，用来连接上下游数据交互的过程叫做Shuffle  
![image](https://user-images.githubusercontent.com/91240419/181016635-fc3b9cc9-51c8-4e0d-a78a-ea95eda85d23.png)
#### Apache Flink 主要从以下几个模块来做流批一体：
SQL 层；  
- DataStream API 层统一，批和流都可以使用 DataStream API 来开发；  
- Scheduler 层架构统一，支持流批场景；  
- Failover Recovery 层 架构统一，支持流批场景；  
- Shuffle Service 层架构统一，流批场景选择不同的 Shuffle Service；  
#### 流批一体的 Scheduler 层
Scheduler 主要负责将作业的 DAG 转化为在分布式环境中可以执行的 Task；  
1.12 之前的 Flink 版本，Flink 支持两种调度模式：
- EAGER（Streaming 场景）：申请一个作业所需要的全部资源，然后同时调度这个作业的全部 Task，所有的 Task 之间采取 Pipeline 的方式进行通信；  
- LAZY（Batch 场景）：先调度上游，等待上游产生数据或结束后再调度下游，类似 Spark 的 Stage 执行模式。  
- Pipeline Region Scheduler 机制：FLIP-119 Pipelined Region Scheduling - Apache Flink - Apache Software Foundation；  
#### 流批一体的 Shuffle Service 层（FLIP-31: Pluggable Shuffle Service - Apache Flink - Apache Software Foundation）
Shuffle：在分布式计算中，用来连接上下游数据交互的过程叫做 Shuffle。实际上，分布式计算中所有涉及到上下游衔接的过程，都可以理解为 Shuffle；  
#### Shuffle 分类：  
- 基于文件的 Pull Based Shuffle，比如 Spark 或 MR，它的特点是具有较高的容错性，适合较大规模的批处理作业，由于是基于文件的，它的容错性和稳定性会更好一些；  
- 基于 Pipeline 的 Push Based Shuffle，比如 Flink、Storm、Presto 等，它的特点是低延迟和高性能，但是因为 shuffle 数据没有存储下来，如果是 batch 任务的话，就需要进行重跑恢复；  
#### 流和批 Shuffle 之间的差异：
- Shuffle 数据的生命周期：流作业的 Shuffle 数据与 Task 是绑定的，而批作业的 Shuffle 数据与 Task 是解耦的；  
- Shuffle 数据存储介质：流作业的生命周期比较短、而且流作业为了实时性，Shuffle 通常存储在内存中，批作业因为数据量比较大以及容错的需求，一般会存储在磁盘里；  
- Shuffle 的部署方式：流作业 Shuffle 服务和计算节点部署在一起，可以减少网络开销，从而减少 latency，而批作业则不同。  
- Pluggable Shuffle Service：Flink 的目标是提供一套统一的 Shuffle 架构，既可以满足不同 Shuffle 在策略上的定制，同时还能避免在共性需求上进行重复开发  
#### Flink 流批一体总结
经过相应的改造和优化之后，Flink 在架构设计上，针对 DataStream 层、调度层、Shuffle Service 层，均完成了对流和批的支持。  
业务已经可以非常方便地使用 Flink 解决流和批场景的问题了。  
### 案例
![image](https://user-images.githubusercontent.com/91240419/181025563-c928a594-fb2e-4f11-be2e-8b797bf5603c.png)  
![image](https://user-images.githubusercontent.com/91240419/181025700-3466181c-5426-4f4f-8dde-0de6685b2630.png)  
![image](https://user-images.githubusercontent.com/91240419/181025827-2c07c4bc-c0e7-471b-84f2-b0118841f49e.png)


## 3.Exactly Once 语义在 Flink 中的实现
动态表 ： 随时间不断变化的表，在任意时刻，可以像查询静态批处理表一样查询它们
#### 实时流的查询特点？
- 查询从不终止   
- 查询结果会不断更新，并且会产生一个新的动态表  
- 结果的动态表也可转换成输出的实时流    
#### 动态表到实时流的转换  
Append-only Stream: Append-only 流（只有 INSERT 消息）  
Retract Stream: Retract 流（同时包含 INSERT 消息和 DELETE 消息）  
### 三种语义
1. At-most-once:出现故障的时候，啥也不做。数据处理不保证任何语义，处理时延低。
2. At-least-once:保证每条数据均至少被处理一次，一条数据可能存在重复消费。
3. Exactly-once:最严格的处理语义，从输出结果来看，每条数据均被消费且仅消费一次，仿佛故障从未发生。

### 两阶段提交
![image](https://user-images.githubusercontent.com/91240419/181522513-9f668235-02e8-4f52-910d-b3affd34f871.png)

## 4.流式计算中的window机制
### 回顾
- 动态表  
- flink中的state和checkpoint的基本原理  
- flink中的retract机制，以及算子如何产生和处理retract数据  
- flink中如何实现exactly-once语义
### watermark
- Watermark定义：当前系统认为的事件时间所在的真实时间。   
- Watermark产生：一般是从数据的事件时间来产生，产生策略可以灵活多样，最常见的包括使用当前事件时间的时间减去一个固定的delay，来表示可以可以容忍多长时间的乱序。  
- Watermark传递：这个类似于上节课中介绍的Checkpoint的制作过程，传递就类似于Checkpoint的barrier，上下游task之间有数据传输关系的，上游就会将watermark传递给下游；下游收到多个上游传递过来的watermark后，默认会取其中最小值来作为自身的watermark，同时它也会将自己watermark传递给它的下游。经过整个传递过程，最终系统中每一个计算单元就都会实时的知道自身当前的watermark是多少。
![image](https://user-images.githubusercontent.com/91240419/181904830-eca0bd40-0c71-4273-b010-a150a5efb69b.png)
#### 怎么观察一个任务中的watermark是多少，是否是正常的
一般通过Flink Web UI上的信息来观察当前任务的watermark情况  
这个问题是生产实践中最容易遇到的问题，大家在开发事件时间的窗口任务的时候，经常会忘记了设置watermark，或者数据太少，watermark没有及时的更新，导致窗口一直不能触发。
#### Per-partition / Per-subtask 生成watermark的优缺点
- 在Flink里早期都是per-subtask的方式进行watermark的生成，这种方式比较简单。但是如果每个source task如果有消费多个partition的情况的话，那多个partition之间的数据可能会因为消费的速度不同而最终导致数据的乱序程度增加。  
- 后期（上面图中）就逐步的变成了per-partition的方式来产生watermark，来避免上面的问题。
#### 如果有部分partition/subtask会断流，应该如何处理
数据断流是很常见的问题，有时候是业务数据本身就有这种特点，比如白天有数据，晚上没有数据。在这种情况下，watermark默认是不会更新的，因为它要取上游subtask发来的watermark中的最小值。此时我们可以用一种IDLE状态来标记这种subtask，被标记为这种状态的subtask，我们在计算watermark的时候，可以把它先排除在外。这样就可以保证有部分partition断流的时候，watermark仍然可以继续更新。
#### 算子对于时间晚于watermark的数据的处理
对于迟到数据，不同的算子对于这种情况的处理可以有不同的实现（主要是根据算子本身的语义来决定的）    
比如window对于迟到的数据，默认就是丢弃；比如双流join，对于迟到数据，可以认为是无法与之前正常数据join上。  
### 处理时间窗口
![image](https://user-images.githubusercontent.com/91240419/181867225-06d48d7c-1107-4e98-9ab5-e4e8be087885.png)  
![image](https://user-images.githubusercontent.com/91240419/181867267-f287ec83-4367-463a-912f-8cc638f02cde.png)
![image](https://user-images.githubusercontent.com/91240419/181867260-dd737179-a289-4ecd-9186-5b06d2ec141c.png)
![image](https://user-images.githubusercontent.com/91240419/181869494-34a9b4ab-0fe7-4a71-85fa-a48f82a1adca.png)
![image](https://user-images.githubusercontent.com/91240419/181869890-0e733988-f52b-472c-90e4-16c7c8748f21.png)
![image](https://user-images.githubusercontent.com/91240419/181870178-b496b236-4f24-446d-b902-da6d071dee26.png)
![image](https://user-images.githubusercontent.com/91240419/181870117-aa38492a-342e-401b-ac24-2ed21b7dce66.png)

### window基本功能
#### 1.滚动窗口
这是最常见的窗口类型，就是根据数据的时间（可以是处理时间，也可以是事件时间）划分到它所属的窗口中windowStart = timestamp - timestamp % windowSize，这条数据所属的window就是[windowStart, windowStart + windowSize)  
在我们使用window的过程中，最容易产生的一个疑问是，window的划分是subtask级别的，还是key级别的。这里大家要记住，Flink 中的窗口划分是key级别的。 比如下方的图中，有三个key，那每个key的窗口都是单独的。所以整个图中，一种存在14个窗口。  
窗口的触发，是时间大于等于window end的时候，触发对应的window的输出（计算有可能提前就增量计算好了），目前的实现是给每个window都注册一个timer，通过处理时间或者事件时间的timer来触发window的输出。  
#### 2.滑动窗口 
了解了上面的TUMBLE窗口的基本原理后，HOP窗口就容易理解了。上面的TUMBLE窗口是每条数据只会落在一个窗口中。在HOP窗口中，每条数据是可能会属于多个窗口的（具体属于多少，取决于窗口定义的大小和滑动），比如下图中假设滑动是1h的话，那窗口大小就是2h，这种情况每条数据会属于两个窗口。除了这一点之外，其它的基本跟HOP窗口是类似的，比如也是key级别划分窗口，也是靠timer进行窗口触发输出。
#### 3.会话窗口  
会话窗口跟上面两种窗口区别比较大，上面两个窗口的划分，都是根据当前数据的时间就可以直接确定它所属的窗口。会话窗口的话，是一个动态merge的过程。一般会设置一个会话的最大的gap，比如10min。  
那某个key下面来第一条数据的时候，它的window就是 [event_time, event_time + gap)，当这个key后面来了另一条数据的时候，它会立即产生一个窗口，如果这个窗口跟之前的窗口有overlap的话，则会将两个窗口进行一个merge，变成一个更大的窗口，此时需要将之前定义的timer取消，再注册一个新的timer。
所以会话窗口要求所有的聚合函数都必须有实现merge。
![image](https://user-images.githubusercontent.com/91240419/181870464-6f27f014-1376-4e40-9434-bfaba5b6f3e0.png)
![image](https://user-images.githubusercontent.com/91240419/181870524-72759ce3-2a48-424c-8cf6-08cf01f4846d.png)
![image](https://user-images.githubusercontent.com/91240419/181870532-93738ea0-9f60-4f0c-a0de-e776a6d4c20a.png)
![image](https://user-images.githubusercontent.com/91240419/181870710-541e5a72-ecc5-4e6e-8077-7d3d77f43659.png)
![image](https://user-images.githubusercontent.com/91240419/181870840-ec04fe24-4573-4d2a-bd6f-ad65b6d6fbb2.png)
#### 迟到数据处理
根据上面说到的watermark原理，watermark驱动某个窗口触发输出之后，这个窗口如果后面又来了数据，那这种情况就属于是迟到的数据了。（注意，不是数据的时间晚于watermark就算是迟到，而是它所属的窗口已经被触发了，才算迟到）。  
对于迟到的数据，我们现在有两种处理方式：  
使用side output方式，把迟到的数据转变成一个单独的流，再由用户自己来决定如何处理这部分数据  
直接drop掉  
注意：side output只有在DataStream的窗口中才可以用，在SQL中目前还没有这种语义，所以暂时只有drop这一个策略。
#### 增量计算 VS 全量计算
这个问题也是使用窗口的时候最典型的问题之一。先定义一下：
##### 增量计算：每条数据到来后，直接参与计算（但是还不需要输出结果）
##### 全量计算：每条数据到来后，先放到一个buffer中，这个buffer会存储到状态里，直到窗口触发输出的时候，才把所有数据拿出来统一进行计算
在SQL里面，主要是窗口聚合，所以都是可以增量计算的，也就是每条数据来了之后都可以直接进行计算，而不用把数据都存储起来。举个例子，比如要做sum计算，那每来一条数据，就直接把新的数据加到之前的sum值上即可，这样我们就只需要存储一个sum值的状态，而不需要存储所有buffer的数据，状态量会小很多。  
DataStream里面要用增量计算的话，需要用reduce/aggregate等方法，就可以用到增量计算。如果用的是process接口，这种就属于是全量计算。
#### EMIT触发
上面讲到，正常的窗口都是窗口结束的时候才会进行输出，比如一个1天的窗口，只有到每天结束的时候，窗口的结果才会输出。这种情况下就失去了实时计算的意义了。  
那么EMIT触发就是在这种情况下，可以提前把窗口内容输出出来的一种机制。比如我们可以配置一个1天的窗口，每隔5s输出一次它的最新结果，那这样下游就可以更快的获取到窗口计算的结果了。  
这个功能只在SQL中，如果是在DataStream中需要完成类似的功能，需要自己定义一些trigger来做。  
上节课中，有讲到retract机制，这里需要提一下，这种emit的场景就是一个典型的retract的场景，发送的结果类似于+[1], -[1], +[2], -[2], +[4]这样子。这样才能保证window的输出的最终结果是符合语义的。
#### Window Offset
按照上面提到的，滚动窗口的计算方式是：windowStart = timestamp - timestamp % windowSize [windowStart, windowStart + windowSize)，这个时间戳是按照unix timestamp来算的。比如我们要用一个一周的窗口，想要的是从周一开始，到周日结束，但是按照上面这种方式计算出来的窗口的话，就是从周四开始的（因为1970年1月1日是周四）。  
那么window offset的功能就是可以在计算窗口的时候，可以让窗口有一个偏移。所以最终计算window的公式就变成了：windowStart = timestamp - (timestamp - offset + windowSize) % windowSize  
DataStream原生就是支持offset的，但是SQL里并不支持，字节内部版本扩展支持了SQL的window offset功能。
### window高级优化
#### Mini-batch  
一般来讲，Flink的状态比较大一些都推荐使用rocksdb statebackend，这种情况下，每次的状态访问就都需要做一次序列化和反序列化，这种开销还是挺大的。为了降低这种开销，我们可以通过降低状态访问频率的方式来解决，这就是mini-batch最主要解决的问题：即赞一小批数据再进行计算，这批数据每个key的state访问只有一次，这样在单个key的数据比较集中的情况下，对于状态访问可以有效的降低频率，最终提升性能。  
这个优化主要是适用于没有窗口的聚合场景，字节内部也扩展了window来支持mini-batch，在某些场景下的测试结果可以节省20-30%的CPU开销。  
mini-batch看似简单，实际上设计非常巧妙。假设用最简单的方式实现，那就是每个算子内部自己进行攒一个小的batch，这样的话，如果上下游串联的算子比较多，任务整体的延迟就不是很容易控制。所以真正的mini-batch实现，是复用了底层的watermark传输机制，通过watermark事件来作为mini-batch划分的依据，这样整个任务中不管串联的多少个算子，整个任务的延迟都是一样的，就是用户配置的delay时间。  
下面这张图展示的是普通的聚合算子的mini-batch原理，window的mini-batch原理是一样的。
![image](https://user-images.githubusercontent.com/91240419/181905011-9e5f92a3-4f31-4f48-a2d4-fbc3226f4d20.png)

#### 倾斜优化
##### Local-global  
local-global优化是分布式系统中典型的优化，主要是可以降低数据shuffle的量，同时也可以缓解数据的倾斜。  
所谓的local-global，就是将原本的聚合划分成两阶段，第一阶段先做一个local的聚合，这个阶段不需要数据shuffle，是直接跟在上游算子之后进行处理的；第二个阶段是要对第一个阶段的结果做一个merge（还记得上面说的session window的merge么，这里要求是一样的。如果存在没有实现merge的聚合函数，那么这个优化就不会生效）。  
如下图所示，比如是要对数据做一个sum，同样颜色的数据表示相同的group by的key，这样我们可以再local agg阶段对他们做一个预聚合；然后到了global阶段数据倾斜就消除了。  
![image](https://user-images.githubusercontent.com/91240419/181871063-7ca763cf-49f4-4f2d-997c-b4b313186c08.png)
做预聚合
![image](https://user-images.githubusercontent.com/91240419/181871268-882f15dd-c4f6-430b-8d26-616bcd899c3e.png)  
#### Distinct状态复用  
对于distinct的优化，一般批里面的引擎都是通过把它优化成aggregate的方式来处理，但是在流式window中，我们不能直接这样进行优化，要不然算子就变成会下发retract的数据了。所以在流式中，对于count distinct这种情况，我们是需要保存所有数据是否出现过这样子的一个映射。  
我们可以把相同字段的distinct计算用一个map的key来存储，在map的value中，用一个bit vector来实现就可以把各个状态复用到一起了。比如一个bigint有64位，可以表示同一个字段的64个filter，这样整体状态量就可以节省很多了。
#### 滑动窗口pane复用
滑动窗口如上面所述，一条数据可能会属于多个window。所以这种情况下同一个key下的window数量可能会比较多，比如3个小时的窗口，1小时的滑动的话，每条数据到来会直接对着3个窗口进行计算和更新。这样对于状态访问频率是比较高的，而且计算量也会增加很多。  
优化方法就是，将窗口的状态划分成更小粒度的pane，比如上面3小时窗口、1小时滑动的情况，可以把pane设置为1h，这样每来一条数据，我们就只更新这条数据对应的pane的结果就可以了。当窗口需要输出结果的时候，只需要将这个窗口对应的pane的结果merge起来就可以了。  

## 5.spark的原理与实践
![image](https://user-images.githubusercontent.com/91240419/181902804-88504c25-b318-49f8-8324-38afd40f4718.png)
Spark应用在集群上运行时，包括了多个独立的进程，这些进程之间通过驱动程序（Driver Program）中的SparkContext对象进行协调，SparkContext对象能够与多种集群资源管理器（Cluster Manager）通信，一旦与集群资源管理器连接，Spark会为该应用在各个集群节点上申请执行器（Executor），用于执行计算任务和存储数据。Spark将应用程序代码发送给所申请到的执行器，SparkContext对象将分割出的任务（Task）发送给各个执行器去运行。
#### 需要注意的是  
1. 每个Spark application都有其对应的多个executor进程。Executor进程在整个应用程序生命周期内，都保持运行状态，并以多线程方式执行任务。这样做的好处是，Executor进程可以隔离每个Spark应用。从调度角度来看，每个driver可以独立调度本应用程序的内部任务。从executor角度来看，不同Spark应用对应的任务将会在不同的JVM中运行。然而这样的架构也有缺点，多个Spark应用程序之间无法共享数据，除非把数据写到外部存储结构中。  
2. Spark对底层的集群管理器一无所知，只要Spark能够申请到executor进程，能与之通信即可。这种实现方式可以使Spark比较容易的在多种集群管理器上运行，例如Mesos、Yarn、Kubernetes。  
Driver Program在整个生命周期内必须监听并接受其对应的各个executor的连接请求，因此driver program必须能够被所有worker节点访问到。  
3. 因为集群上的任务是由driver来调度的，driver应该和worker节点距离近一些，最好在同一个本地局域网中，如果需要远程对集群发起请求，最好还是在driver节点上启动RPC服务响应这些远程请求，同时把driver本身放在离集群Worker节点比较近的机器上。
### spark core
![image](https://user-images.githubusercontent.com/91240419/181902998-6ba9e609-1dc8-4577-a7cf-10a1dfd6b2bf.png)
#### RDD  
![image](https://user-images.githubusercontent.com/91240419/181905426-b643ddbf-e4e3-49fe-a01c-bf6786a7749f.png)
1. 并行执行的分布式数据集，spark中数据处理模型   
2. 分区（决定并行执行的数量）
3. 前后依赖其他rdd
- 划分Stage的整体思路：从后往前推，遇到宽依赖就断开，划分为一个Stage。遇到窄依赖，就将这个RDD加入该Stage中，DAG最后一个阶段会为每个结果的Partition生成一个ResultTask。每个Stage里面的Task数量由最后一个RDD的Partition数量决定，其余的阶段会生成ShuffleMapTask。    
- 当RDD对象创建后，SparkContext会根据RDD对象构建DAG有向无环图，然后将Task提交给DAGScheduler。DAGScheduler根据ShuffleDependency将DAG划分为不同的Stage，为每个Stage生成TaskSet任务集合，并以TaskSet为单位提交给TaskScheduler。TaskScheduler根据调度算法(FIFO/FAIR)对多个TaskSet进行调度，并通过集群中的资源管理器(Standalone模式下是Master，Yarn模式下是ResourceManager)把Task调度(locality)到集群中Worker的Executor，Executor由SchedulerBackend提供。  
RDD算子  
1. transform算计：生成一个新的rdd 
2. action算子：触发job提交  
RDD依赖   
窄依赖：  
宽依赖：会产生shuffle    
![image](https://user-images.githubusercontent.com/91240419/181904336-0e432e3f-6ef8-4d11-bf22-aff7882b6c63.png)
#### 内存管理
![image](https://user-images.githubusercontent.com/91240419/181905455-2a641cd0-0383-4653-9117-1a111faa3fe9.png)
Spark 作为一个基于内存的分布式计算引擎，Spark采用统一内存管理机制。重点在于动态占用机制。  
设定基本的存储内存(Storage)和执行内存(Execution)区域，该设定确定了双方各自拥有的空间的范围，UnifiedMemoryManager统一管理Storage/Execution内存  
双方的空间都不足时，则存储到硬盘；若己方空间不足而对方空余时，可借用对方的空间  
当Storage空闲，Execution可以借用Storage的内存使用，可以减少spill等操作， Execution内存不能被Storage驱逐。Execution内存的空间被Storage内存占用后，可让对方将占用的部分转存到硬盘，然后"归还"借用的空间  
当Execution空闲，Storage可以借用Execution内存使用，当Execution需要内存时，可以驱逐被Storage借用的内存，可让对方将占用的部分转存到硬盘，然后"归还"借用的空间  
user memory存储用户自定义的数据结构或者spark内部元数据  
Reserverd memory：预留内存，防止OOM，  
堆内(On-Heap)内存/堆外(Off-Heap)内存：Executor 内运行的并发任务共享 JVM 堆内内存。为了进一步优化内存的使用以及提高 Shuffle 时排序的效率，Spark 可以直接操作系统堆外内存，存储经过序列化的二进制数据。减少不必要的内存开销，以及频繁的 GC 扫描和回收，提升了处理性能。
### SparkSQL
![image](https://user-images.githubusercontent.com/91240419/181915346-8635b2bb-4e7d-4900-9991-9ad6b4e2d17a.png)
##### SparkSQL执行过程
- SQL Parse： 将SparkSQL字符串或DataFrame解析为一个抽象语法树/AST，即Unresolved Logical Plan  
- Analysis：遍历整个AST，并对AST上的每个节点进行数据类型的绑定以及函数绑定，然后根据元数据信息Catalog对数据表中的字段进行解析。 利用Catalog信息将Unresolved Logical Plan解析成Analyzed Logical plan  
- Logical Optimization：该模块是Catalyst的核心，主要分为RBO和CBO两种优化策略，其中RBO是基于规则优化，CBO是基于代价优化。 利用一些规则将Analyzed Logical plan解析成Optimized Logic plan  
- Physical Planning: Logical plan是不能被spark执行的，这个过程是把Logic plan转换为多个Physical plans  
- CostModel: 主要根据过去的性能统计数据，选择最佳的物理执行计划(Selected Physical Plan)。  
- Code Generation: sql逻辑生成Java字节码
##### 影响SparkSQL性能两大技术：
1. Optimizer：执行计划的优化，目标是找出最优的执行计划  
2. Runtime：运行时优化，目标是在既定的执行计划下尽可能快的执行完毕。  

#### Catalyst优化
- Rule Based Optimizer(RBO): 基于规则优化，对语法树进行一次遍历，模式匹配能够满足特定规则的节点，再进行相应的等价转换。  
- Cost Based Optimizer(CBO): 基于代价优化，根据优化规则对关系表达式进行转换，生成多个执行计划，然后CBO会通过根据统计信息(Statistics)和代价模型(Cost Model)计算各种可能执行计划的代价，从中选用COST最低的执行方案，作为实际运行方案。CBO依赖数据库对象的统计信息，统计信息的准确与否会影响CBO做出最优的选择。
#### AQE
AQE对于整体的Spark SQL的执行过程做了相应的调整和优化，它最大的亮点是可以根据已经完成的计划结点真实且精确的执行统计结果来不停的反馈并重新优化剩下的执行计划。
##### AQE框架三种优化场景：  
- 动态合并shuffle分区（Dynamically coalescing shuffle partitions）  
- 动态调整Join策略（Dynamically switching join strategies）  
- 动态优化数据倾斜Join（Dynamically optimizing skew joins）
#### RuntimeFilter
实现在Catalyst中。动态获取Filter内容做相关优化，当我们将一张大表和一张小表等值连接时，我们可以从小表侧收集一些统计信息，并在执行join前将其用于大表的扫描，进行分区修剪或数据过滤。可以大大提高性能
##### Runtime优化分两类：
+ 全局优化：从提升全局资源利用率、消除数据倾斜、降低IO等角度做优化。包括AQE。  
+ 局部优化：提高某个task的执行效率，主要从提高CPU与内存利用率的角度进行优化。依赖Codegen技术。
####Codegen
从提高cpu的利用率的角度来进行runtime优化。
#### Expression级别
表达式常规递归求值语法树。需要做很多类型匹配、虚函数调用、对象创建等额外逻辑，这些overhead远超对表达式求值本身，为了消除这些overhead，Spark Codegen直接拼成求值表达式的java代码并进行即时编译
#### WholeStage级别
+ 传统的火山模型：SQL经过解析会生成一颗查询树，查询树的每个节点为Operator，火山模型把operator看成迭代器，每个迭代器提供一个next()接口。通过自顶向下的调用 next 接口，数据则自底向上的被拉取处理，火山模型的这种处理方式也称为拉取执行模型，每个Operator 只要关心自己的处理逻辑即可，耦合性低。  
+ 火山模型问题：数据以行为单位进行处理，不利于CPU cache 发挥作用；每处理一行需要调用多次next() 函数，而next()为虚函数调用。会有大量类型转换和虚函数调用。虚函数调用会导致CPU分支预测失败，从而导致严重的性能回退  
Spark WholestageCodegen：为了消除这些overhead，会为物理计划生成类型确定的java代码。并进行即时编译和执行。  
Codegen打破了Stage内部算子间的界限，拼出来跟原来的逻辑保持一致的裸的代码（通常是一个大循环）然后把拼成的代码编译成可执行文件。

## 6.大数据 Shuffle 原理与实践
![image](https://user-images.githubusercontent.com/91240419/182065115-96d7532b-38f3-469e-b1bf-13ae721843ad.png)
####为什么shuffle如此重要
- 数据shuffle表示了不同分区数据交换的过程，不同的shuffle策略性能差异较大。
- 目前在各个引擎中shuffle都是优化的重点，在spark框架中，shuffle是支撑spark进行大规模复杂数据处理的基石。  

### shuffle算子
####常见的触发shuffle的算子
##### repartition
- coalesce、repartition
##### ByKey
- groupByKey、reduceByKey、aggregateByKey、combineByKey、sortByKeysortBy
##### Join
- cogroup、join
### Shuffle Dependency
创建会产生shuffle的RDD时，RDD会创建Shuffle Dependency来描述Shuffle相关的信息
##### 构造函数
- A single key-value pair RDD, i.e. RDD[Product2[K, V]],
- Partitioner (available as partitioner property),
- Serializer,
- Optional key ordering (of Scala’s scala.math.Ordering type),
- Optional Aggregator,
- mapSideCombine flag which is disabled (i.e. false) by default.
### Partitioner
用来将record映射到具体的partition的方法  
经典实现：hashPartitioner  
##### 接口
- numberPartitions
- getPartition
### Aggregator
在map侧合并部分record的函数
##### 接口
- createCombiner：只有一个value的时候初始化的方法
- mergeValue：合并一个value到Aggregator中
- mergeCombiners：合并两个Aggregator

## 7.shuffle过程
![image](https://user-images.githubusercontent.com/91240419/182066400-436fb4b1-d6dc-4d50-8f1b-61f770cc1cb8.png)
![image](https://user-images.githubusercontent.com/91240419/182066458-f35ccf24-8911-4c62-af8a-2143f0e03a60.png)
#### HashShuffle
- 优点：不需要排序
- 缺点：打开，创建的文件过多
![image](https://user-images.githubusercontent.com/91240419/182066595-91cf0a99-0d66-4911-8691-e8c4be89ce53.png)
![image](https://user-images.githubusercontent.com/91240419/182066604-3db7cf1f-4503-4081-89f4-24e856a2c2c1.png)
#### SortShuffle
- 优点：打开的文件少、支持map-side combine
- 缺点：需要排序
#### TungstenSortShuffle
- 优点：更快的排序效率，更高的内存利用效率
- 缺点：不支持map-side combine
### shuffle触发流程
![image](https://user-images.githubusercontent.com/91240419/182067024-e9b34415-3970-4ef7-9fbb-fd753bec9870.png)
#### Register Shuffle
![image](https://user-images.githubusercontent.com/91240419/182067064-9f0d510c-ba1a-4ecf-b231-5a668117a012.png)
- 由action算子触发DAG Scheduler进行shuffle register
- Shuffle Register会根据不同的条件决定注册不同的ShuffleHandle
#### shuffle writer
![image](https://user-images.githubusercontent.com/91240419/182067195-18851419-a4d8-4d94-b320-c36914df0b07.png)
![image](https://user-images.githubusercontent.com/91240419/182067413-71c205de-11e1-4539-8cc2-116f0ede2f6f.png)
![image](https://user-images.githubusercontent.com/91240419/182067485-bbe753d5-ee7a-4fc6-bba5-6e307821fd31.png)
![image](https://user-images.githubusercontent.com/91240419/182067525-8a6ae044-80a9-43fa-bf87-71eda590edf5.png)
![image](https://user-images.githubusercontent.com/91240419/182067547-6c029304-5646-41c3-91d3-2815a1fa41a5.png)
### ShuffleReader网络请求流程
- 使用基于netty的网络通信框架  
- 位置信息记录在MapOutputTracker中
- 主要会发送两种类型的请求：OpenBlocks请求、Chunk请求或Stream请求
 ![image](https://user-images.githubusercontent.com/91240419/182068141-962e0655-69a4-4951-a514-7bb694364afe.png)
 使用netty作为网络框架提供网络服务，并接受reducetask的fetch请求  
首先发起openBlocks请求获得streamId，然后再处理stream或者chunk请求
### Reader实现-ShuffleBlockFetchIterator
![image](https://user-images.githubusercontent.com/91240419/182068272-a7813610-5328-4ced-9137-ae067641239c.png)
### Read实现-External Shuffle Service
ESS作为一个存在于每个节点上的agent为所有Shuffle Reader提供服务，从而优化了Spark作业的资源利用率，MapTask在运行结束后可以正常退出。
![image](https://user-images.githubusercontent.com/91240419/182068473-26725476-6c07-48e1-a826-6c313b386b34.png)
为了解决Executor为了服务数据的fetch请求导致无法退出问题，我们在每个节点上部署一个External Shuffle Service，这样产生数据的Executor在不需要继续处理任务时，可以随意退出。
### shuffle优化
#### 避免使用shuffle
```
//传统的join操作会导致shuffle操作。
//因为两个RDD中，相同的key都需要通过网络拉取到一个节点上，由一个task进行join操作。
val rdd3 = rdd1.join(rdd2)

//Broadcast+map的join操作，不会导致shuffle操作。
//使用Broadcast将一个数据量较小的RDD作为广播变量。
val rdd2Data = rdd2.collect()
val rdd2DataBroadcast = sc.broadcast(rdd2Data)

//在rdd1.map算子中，可以从rdd2DataBroadcast中，获取rdd2的所有数据。
//然后进行遍历，如果发现rdd2中某条数据的key与rdd1的当前数据的key是相同的，那么就判定可以进行join。
//此时就可以根据自己需要的方式，将rdd1当前数据与rdd2中可以连接的数据，拼接在一起（String或Tuple）。
val rdd3 = rdd1.map(rdd2DataBroadcast...)

//注意，以上操作，建议仅仅在rdd2的数据量比较少（比如几百M，或者一两G）的情况下使用。
//因为每个Executor的内存中，都会驻留一份rdd2的全量数据。
```
- 使用可以map-side预聚合的算子
- Shuffle 参数优化
```
spark.default.parallelism && spark.sql.shuffle.partitions
spark.hadoopRDD.ignoreEmptySplits
spark.hadoop.mapreduce.input.fileinputformat.split.minsize
spark.sql.file.maxPartitionBytes
spark.sql.adaptive.enabled && spark.sql.adaptive.shuffle.targetPostShuffleInputSize
spark.reducer.maxSizeInFlight
spark.reducer.maxReqsInFlight spark.reducer.maxBlocksInFlightPerAddress
```

#### 零拷贝-Zero Copy
![image](https://user-images.githubusercontent.com/91240419/182069064-c9a49ade-c129-4fc1-91b1-258b154c8773.png)
#### Netty 零拷贝
- 可堆外内存，避免 JVM 堆内存到堆外内存的数据拷贝。
- CompositeByteBuf 、 Unpooled.wrappedBuffer、 ByteBuf.slice ，可以合并、包装、切分数组，避免发生内存拷贝
- Netty 使用 FileRegion 实现文件传输，FileRegion 底层封装了 FileChannel#transferTo() 方法，可以将文件缓冲区的数据直接传输到目标 Channel，避免内核缓冲区和用户态缓冲区之间的数据拷贝

### Shuffle 倾斜优化
#### 解决倾斜方法举例
- 增大并发度
- AQE
 ![image](https://user-images.githubusercontent.com/91240419/182080256-3f19cd00-6121-42a0-91da-4388d9ca3294.png)
 ![image](https://user-images.githubusercontent.com/91240419/182080383-e3c9c938-ab99-42db-a5ba-e4cd0b67d342.png)

### shuffle过程问题
- 数据存储在本地磁盘，没有备份
- IO 并发：大量 RPC 请求（M*R）
- IO 吞吐：随机读、写放大（3X）
- GC 频繁，影响 NodeManager
![image](https://user-images.githubusercontent.com/91240419/182080481-54618a67-0a01-4312-b0c7-ec0bf08ca004.png

####  Magnet主要流程
![image](https://user-images.githubusercontent.com/91240419/182080600-3ca255df-7368-4358-9d64-279e295f271b.png)
![image](https://user-images.githubusercontent.com/91240419/182080696-922dbffd-8afc-4cf3-9da7-b0c489815f99.png)
![image](https://user-images.githubusercontent.com/91240419/182080621-86c62267-eb7a-4dbb-879f-8bd27e0be57a.png)
![image](https://user-images.githubusercontent.com/91240419/182080719-9c7809d1-d3ee-4ded-8da2-62418a572789.png)
主要为边写边push的模式，在原有的shuffle基础上尝试push聚合数据，但并不强制完成，读取时优先读取push聚合的结果，对于没有来得及完成聚合或者聚合失败的情况，则fallback到原模式。  
![image](https://user-images.githubusercontent.com/91240419/182080822-fbb257ca-a3ea-422c-8579-7cab16a721d2.png)
####  Cloud Shuffle Service架构
![image](https://user-images.githubusercontent.com/91240419/182080944-d9b4cd1e-5b67-4c12-b4f0-d0d9aad35d98.png)
- Zookeeper WorkerList [服务发现]
- CSS Worker [Partitions / Disk | Hdfs]
- Spark Driver [集成启动 CSS Master]
- CSS Master [Shuffle 规划 / 统计]
- CSS ShuffleClient [Write / Read]
- Spark Executor [Mapper + Reducer]
##### Cloud Shuffle Service 支持AQE
![image](https://user-images.githubusercontent.com/91240419/182081075-de2491be-9539-4f63-9d0f-50792c025edf.png)
- 在聚合文件时主动将文件切分为若干块，当触发AQE时，按照已经切分好的文件块进行拆分。

## 8.HDFS高可用与高扩展机制
### HDFS高可用架构
![image](https://user-images.githubusercontent.com/91240419/183241717-6cf57ff9-7f0d-4721-a695-3e7f54331645.png)
- Active NameNode：提供服务的 NameNode 主节点，生产 editlog。
- Standby NameNode：不提供服务，起备份作用的 NameNode 备节点，消费 editlog
- editlog：用户变更操作的记录，具有全局顺序，是 HDFS 的变更日志。
- ZooKeeper：开源的分布式协调组件，主要功能有节点注册、主节点选举、元数据存储。
- BookKeeper：开源的日志存储组件，存储 editlog
- ZKFC：和 ZK、NN 通信，进行 NN 探活和自动主备切换。
- HA Client：处理 StandbyException，在主备节点间挑选到提供服务的主节点。
 ![image](https://user-images.githubusercontent.com/91240419/183241732-f1bae5a4-217e-41a4-8c10-d9955dc3f163.png)
#### NameNode 状态持久化
- FSImage 文件：较大的状态记录文件，是某一时刻 NN 全部需要持久化的数据的记录。大小一般在 GB 级别。
- EditLog 文件：是某段时间发生的变更日志的存储文件。大小一般在 KB~MB 级别。
- checkpoint 机制：将旧的 FSImage 和 EditLog 合并生成新的 FSImage 的流程，在完成后旧的数据可以被清理以释放空间。
物理日志：存储了物理单元（一般是磁盘的 page）变更的日志形式。  
逻辑日志：存储了逻辑变更（例如 rename /a to /b）的日志形式。

#### HDFS 主备切换
- DataNode 心跳与块汇报需要同时向 active NN 和 standby NN 上报，让两者可以同时维护块信息。但只有 active NN 会下发 DN 的副本操作命令。
- content stale 状态：在发生主备切换后，新 active NN 会标记所有 DN 为 content stale 状态，代表该 DN 上的副本是不确定的，某些操作不能执行。直到一个 DN 完成一次全量块上报，新 active NN 才标记它退出了 content stale 状态。
例子，多余块的删除：NN 发现某个块的副本数过多，会挑选其中一个 DN 来删除数据。在主备切换后，新 active NN 不知道旧 active NN 挑选了哪个副本进行删除，就可能触发多个 DN 的副本删除，极端情况下导致数据丢失。content stale 状态的引入解决了这个问题。
- 脑裂问题：因为网络隔离、进程夯住（例如 Java GC）等原因，旧的 active NN 没有完成下主，新的 active NN 就已经上主，此时会存在双主。client 的请求发给两者都可能成功，但不能保证一致性（两个 NN 状态不再同步）和持久化（只能保留一个 NN 状态）。
- fence 机制：在新 active NN 上主并正式处理请求之前，先要确保旧 active NN 已经退出主节点的状态。一般做法是先用 RPC 状态检测，发现超时或失败则调用系统命令杀死旧 active NN 的进程。

#### 自动主备切换
- ZooKeeper 是广泛使用的选主组件，它通过 ZAB 协议保证了多个 ZK Server 的状态一致，提供了自身的强一致和高可用。
- ZooKeeper 的访问单位是 znode，并且可以确保 znode 创建的原子性和互斥性（CreateIfNotExist）。client 可以创建临时 znode，临时 znode 在 client 心跳过期后自动被删除。
- ZK 提供了 Watch 机制，允许多个 client 一起监听一个 znode 的状态变更，并在状态变化时收到一条消息回调（callback）。
- 基于临时 znode 和 Watch 机制，多个客户端可以完成自动的主选举。
- ZKFailoverController：一般和 NN 部署在一起的进程，负责定时查询 NN 存活和状态、进行 ZK 侧主备选举、执行调用 NN 接口执行集群的主备状态切换、执行 fence 等能力。
- Hadoop 将集群主备选举的能力和 NN 的服务放在了不同的进程中，而更先进的系统一般会内置在服务进程中。
#### 高可用日志系统 BookKeeper
- 高可靠：数据写入多个存储节点，数据写入就不会丢失。
- 高可用：日志存储本身是高可用的。因为日志流比文件系统本身的结构更为简单，日志系统高可用的实现也更为简单。
- 强一致：日志系统是追加写入的形式，Client 和日志系统的元数据可以明确目前已经成功的写入日志的序号（entry-id）。
- 可扩展：整个集群的读写能力可以随着添加存储节点 Bookie 而扩展。

### 数据高可用
![image](https://user-images.githubusercontent.com/91240419/183241842-9de69448-f00a-4a1d-a2b7-38843784b7a0.png)
- RAID 0 ：将数据分块后按条带化的形式分别存储在多个磁盘上，提供大容量、高性能。
- RAID 1：将数据副本存储在多个磁盘上，提供高可靠。
- RAID 3：在数据分块存储的基础上，将数据的校验码存储在独立的磁盘上，提供高可靠、高性能。
![image](https://user-images.githubusercontent.com/91240419/183241866-19124712-3fc0-4628-9b5a-b7afecb84cbe.png)
#### Erasure Coding 方案：将数据分段，通过特殊的编码方式存储额外的校验块，并条带化的组成块，存储在 DN 上。
- 条带化：原本块对应文件内连续的一大段数据。条带化后，连续的数据按条带（远小于整个块的单位）间隔交错的分布在不同的块中。
- Reed Solomon 算法：参考 Reed-solomon codes
- 成本更低：多副本方案需要冗余存储整个块，EC 方案需要冗余存储的数据一般更少。

### 元数据扩展性
#### 扩展性方案
- scale up：通过单机的 CPU、内存、磁盘、网卡能力的提升来提升系统服务能力，受到机器成本和物理定律的限制。
- scale out：通过让多台机器组成集群，共同对外提供服务来提升系统服务能力。一般也称为高扩展、水平扩展。
#### partition 方法
- 水平分区和垂直分区：水平分区指按 key 来将数据划分到不同的存储上；垂直分区指将一份数据的不同部分拆开存储，用 key 关联起来。partition 一般都水平分区，又称 shard。
- 常用于 KV 模型，通过 hash 或者分段的手段，将不同类型 key 的访问、存储能力分配到不同的服务器上，实现了 scale out。
- 重点：不同单元之间不能有关联和依赖，不然访问就难以在一个节点内完成。例如 MySQL 的分库分表方案，难以应对复杂跨库 join。
#### federation 架构
- 使得多个集群像一个集群一样提供服务的架构方法，提供了统一的服务视图，提高了服务的扩展性。
- 文件系统的目录树比 kv 模型更复杂，划分更困难。
- 邦联架构的难点一般在于跨多个集群的请求，例如 HDFS 的 rename 操作就可能跨多个集群。
#### blockpool
- 将文件系统分为文件层和块存储层，对于块存储层，DN 集群对不同的 NN 提供不同的标识符，称为 block pool。
- 解决了多个 NN 可能生成同一个 block id，DN 无法区分的问题。
#### viewfs
- 邦联架构的一种实现，通过客户端配置决定某个路径的访问要发送给哪个 NN 集群
- 缺点：客户端配置难以更新、本身配置方式存在设计（例如，只能在同一级目录区分；已经划分的子树不能再划分）。
#### NNProxy
- ByteDance 自研的 HDFS 代理层，于 2016 年开源，项目地址： github.com/bytedance/n…
- 主要提供了路由管理、RPC 转发，额外提供了鉴权、限流、查询缓存等能力。
- 开源社区有类似的方案 Router Based Federation，主要实现了路由管理和转发
#### 小文件问题
- HDFS 设计上是面向大文件的，小于一个 HDFS Block 的文件称为小文件。
- 元数据问题：多个小文件相对于一个大文件，使用了更多元数据服务的内存空间。
- 数据访问问题：多个小文件相对于一个大文件，I/O 更加的随机，无法顺序扫描磁盘。
- 计算任务启动慢：计算任务在启动时，一般会获得所有文件的地址来进行 MapReduce 的任务分配，小文件会使得这一流程变长。
- 典型的 MR 流程中，中间数据的文件数和数据量与 mapper*reducer 的数量成线性，而为了扩展性，一般 mapper 和 reducer 的数量和数据量成线性。于是，中间数据的文件数和数据量与原始的数据量成平方关系。
- 小文件合并任务：计算框架的数据访问模式确定，可以直接将小文件合并成大文件而任务读取不受影响。通过后台运行任务来合并小文件，可以有效缓解小文件问题。通过 MapReduce/Spark 框架，可以利用起大量的机器来进行小文件合并任务。
- Shuffle service：shuffle 流程的中间文件数是平方级的，shuffle service 将 shuffle 的中间数据存储在独立的服务上，通过聚合后再写成 HDFS 文件，可以有效地缓解中间数据的小文件问题。
![image](https://user-images.githubusercontent.com/91240419/183241975-4530e1a0-8f8d-4f20-8649-2af2652f4c6d.png)
### 数据扩展性
#### 长尾
- 二八定律：在任何一组东西中，最重要的只占其中一小部分，约 20%，其余 80% 尽管是多数，却是次要的。
- 长尾：占绝大多数的，重要性低的东西就被称为长尾。
#### 百分位延迟
将所有请求的响应速度从快到慢排序，取其中某百分位的请求的延迟时间。
例如 pct99 代表排在 99% 的请求的延迟。相对于平均值，能更好的衡量长尾的情况。
#### 尾部延迟放大
- 木桶原理：并行执行的任务的耗时取决于最慢的一个子任务。
- 尾部延迟放大：一个请求或任务需要访问多个数据节点，只要其中有一个慢，则整个请求或任务的响应就会变慢。
- 固定延迟阈值，访问的集群越大， 高于该延迟的请求占比越高。
- 固定延迟百分位，访问的集群越大，延迟越差。
#### 长尾问题
- 尾部延迟放大+集群规模变大，使得大集群中，尾部延迟对于整个服务的质量极为重要。
- 慢节点问题：网络不会直接断联，而是不能在预期的时间内返回。会导致最终请求不符合预期，而多副本机制无法直接应对这种问题。
- 高负载：单个节点处理的请求超过了其服务能力，会引发请求排队，导致响应速度慢。是常见的一个慢节点原因。
![image](https://user-images.githubusercontent.com/91240419/183241984-06aed100-22cc-4285-94c5-b124022e5e99.png)

### 数据可靠性
- 超大集群下，一定有部分机器是损坏的，来不及修理的。
- 随机的副本放置策略，所有的放置组合都会出现。而 DN 容量够大，足够
- 三副本，单个 DN 视角：容量一百万，机器数量一万。那么另外两个副本的排列组合有一亿种，容量比放置方案大约百分之一。
- 三副本，全局视角：一万台机器，每台一百万副本，损坏 1%（100 台）。根据排列组合原理，大约有 1009998/(1000099999998)(100000010000)=9704 个坏块
- callback 一下，叠加长尾问题。每个任务都要访问大量的块，只要一个块丢失就整个任务收到影响。导致任务层面的丢块频发，服务质量变差。
#### copyset
- 降低副本放置的组合数，降低副本丢失的发生概率。
- 修复速度：DN 机器故障时，只能从少量的一些其他 DN 上拷贝数据修复副本。
![image](https://user-images.githubusercontent.com/91240419/183242031-7eaa4497-f73a-4263-a845-289919befe6a.png)
### 负载均衡的意义
避免热点
- 机器热点会叠加长尾问题，少数的不均衡的热点会影响大量的任务。
成本：
- 数据越均衡，CPU、磁盘、网络的利用率越高，成本更低。
- 集群需要为数据腾挪预留的空间、带宽更少，降低了成本。
可靠性
- 全速运行的机器和空置的机器，以及一会全速运行一会空置的机器，可靠性表现都有不同。负载均衡可以降低机器故障的发生。
- 同一批机器容易一起故障，数据腾挪快，机器下线快，可以提升可靠性。
#### 负载均衡性影响因素：多个复杂因素共同影响负载均衡性
- 不同节点上的业务量的平衡
- 数据放置策略
- 数据搬迁工具的能力
- 系统环境
#### 集群的不均衡情况
- 节点容量不均：机器上的数据量不均衡。
- 原因可能是各种复杂情况导致，归根结底是混沌现象。

- 数据新旧不均：机器上的数据新旧不均匀。  

- 例如：新上线的机器，不做任何数据均衡的情况下，只会有新写入的数据。而一般新数据更容易被读取，更为「热」。  

- 访问类型不均：机器上的数据访问类型不均。  

- 例如：机器学习训练需要反复读取数据，小 I/O 更多。而大数据场景一般只扫描一次，大 I/O 为主。这两种模式的读写比不同，I/O pattern 不同，就来带访问冷热的不同。
- 异构机器：有的机器配置高、有的机器配置低，不考虑异构情况的话配置高的机器会闲置，配置低的机器会过热。

- 资源不均：机器上的访问请求吞吐、IOPS 不均衡，导致最终机器冷热不均、负载不均。一般由于容量不均、新旧不均、模式不均导致
##### 需要数据迁移的典型场景
- DN 上线：新上线的机器没有任何数据，而且只会有新数据写入。需要迁移其他 DN 的旧数据到新 DN 上，使得负载和数据冷热均衡。
- DN 下线：需要下线的机器，需要提前将数据迁移走再停止服务，避免数据丢失的风险。
- 机房间均衡：因为资源供应、新机房上线等外部条件，机房规划、业务分布等内部条件，不同机房的资源量和资源利用率都是不均衡的。需要结合供应和业务，全局性的进行资源均衡。

- 日常打散：作为日常任务运行，不断地从高负载、高容量的机器上搬迁数据到低负载、低容量的机器上，使得整个集群的负载均衡起来。
### 数据迁移工具
- 目的：将数据从一部分节点搬迁到另一部分节点。
- 要求：高吞吐、不能影响前台的服务。
- 带元数据迁移的迁移工具
- 痛点：涉及到元数据操作，需要停止用户的写入。
#### DistCopy 工具
通过 MapReduce 任务来并行迁移数据，需要拷贝数据和元数据。  
网络流量较大，速度较慢。
#### FastCopy 工具
- 基于 hardlink 和 blockpool 的原理
- 元数据直接在 NN 集群间拷贝，而数据则在 DN 上的不同 blockpool（对应到 NN 集群）进行 hardlink，不用数据复制。
迁移速度要大大优于 DistCopy。
### 数据迁移工具
#### Balancer 工具
代替 NN 向 DN 发起副本迁移的命令，批量执行副本迁移。
场景：大规模数据平衡、机器上下线。

## 9.深入浅出 HBase 实战
https://juejin.cn/post/7126813033602482190/#heading-0
## Parquet 和 ORC：高性能列式存储
### 一个大数据查询作业，可以简单的概括为以下几个步骤：
- 从存储层读取文件
- 计算层解析文件内容，运行各种计算算子
- 计算层输出结果，或者把结果写入存储层
### 行存 vs 列存
#### 数据格式层
- 数据格式层：定义了存储层文件内部的组织格式，计算引擎通过格式层的支持来读写文件
- 严格意义上，并不是一个独立的层级，而是运行在计算层的一个Library
![image](https://user-images.githubusercontent.com/91240419/184046171-3dadad0c-7189-4f4b-9dd2-8598503f4510.png)
#### OLTP vs OLAP
OLTP 和 OLAP 作为数据查询和分析领域两个典型的系统类型，具有不同的业务特征，适配不同的业务场景
![image](https://user-images.githubusercontent.com/91240419/184046367-a97745cb-f4c8-4dc7-8b83-054a1e0ef0dd.png)

#### 行式存储格式 (行存) 与 OLTP
- 每一行 (Row) 的数据在文件的数据空间里连续存放的
- 读取整行的效率比较高，一次顺序 IO 即可
- 在典型的 OLTP 型的分析和存储系统中应用广泛，例如：MySQL、Oracle、RocksDB 等
![image](https://user-images.githubusercontent.com/91240419/184046422-2e6d7b70-9e8b-4072-bf7d-b09d6ad92364.png)

#### 列式存储格式 (列存) 与 OLAP
- 每一列 (Column) 的数据在文件的数据空间里连续存放的
- 同列的数据类型一致，压缩编码的效率更好
- 在典型的 OLAP 型分析和存储系统中广泛应用，例如：
  - 大数据分析系统：Hive、Spark，数据湖分析
  - 数据仓库：ClickHouse，Greenplum，阿里云 MaxCompute
![image](https://user-images.githubusercontent.com/91240419/184046531-db7c6c20-2922-4630-b2c1-998f42569124.png)

### Parquet （列存）
#### Parquet 中的数据编码
- 在 Parquet 的 ColumnChunk 里，同一个 ColumnChunk 内部的数据都是同一个类型的，可以通过编码的方式更高效的存储
下面举例介绍常见的 Encoding：
- Run Length Encoding (RLE)：适用于列基数不大，重复值较多的场景，例如：Boolean、枚举、固定的选项等
- Bit-Pack Encoding: 对于 32位或者64位的整型数而言，并不需要完整的 4B 或者 8B 去存储，高位的零在存储时可以省略掉。适用于最大值非常明确的情况下。
  -一般配合 RLE 一起使用
- Dictionary Encoding：适用于列基数 (Column Cardinality) 不大的字符串类型数据存储；
  -构造字典表，用字典中的 Index 替换真实数据
  -替换后的数据可以使用 RLE + Bit-Pack 编码存储

#### Parquet 中的压缩方式
- Page 完成 Encoding 以后，进行压缩
- 支持多种压缩算法
  -snappy: 压缩速度快，压缩比不高，适用于热数据
  -gzip：压缩速度慢，压缩比高，适用于冷数据
  -zstd：新引入的压缩算法，压缩比和 gzip 差不多，而且压缩速度略低于 Snappy
  
#### 索引和排序 Index and Ordering
- 和传统的数据库相比，索引支持非常简陋
- 主要依赖 Min-Max Index 和 排序 来加速查找
- Page：记录 Column 的 min_value 和 max_value
- Footer 里的 Column Metadata 包含 ColumnChunk 的全部 Page 的 Min-Max Value
- 一般建议和排序配合使用效果最佳
- 一个 Parquet 文件只能定义一组 Sort Column，类似聚集索引概念
![image](https://user-images.githubusercontent.com/91240419/184046978-9d6f17d0-c84b-48f3-a67c-50779358693f.png)
典型的查找过程：
- 读取 Footer
- 根据 Column 过滤条件，查找 Min-Max Index 定位到 Page
- 根据 Page 的 Offset Index 定位具体的位置
- 读取 Page，获取行号
- 从其他 Column 读取剩下的数据

#### Bloom Filter 索引
- 适用场景
  -对于列基数比较大的场景，或者非排序列的过滤，Min-Max Index 很难发挥作用
- 引入 Bloom Filter 加速过滤匹配判定
- 每个 ColumnChunk 的头部保存 Bloom Filter 数据
- Footer 记录 Bloom Filter 的 page offset
![image](https://user-images.githubusercontent.com/91240419/184047091-8cbbca47-9e08-4aaa-9e8d-f07d99e17fc0.png)

#### 过滤下推 Predicate PushDown
- parquet-mr 库实现，实现高效的过滤机制
- 引擎侧传入 Filter Expression
- parquet-mr 转换成具体 Column 的条件匹配
- 查询 Footer 里的 Column Index，定位到具体的行号
- 返回有效的数据给引擎侧
优点：
- 在格式层过滤掉大多数不相关的数据
- 减少真实的读取数据量
![image](https://user-images.githubusercontent.com/91240419/184048208-d57fec9c-a446-45aa-9f63-25066676e984.png)

#### Parquet & Spark
作为最通用的 Spark 数据格式
主要实现在：ParquetFileFormat
- 支持向量化读：spark.sql.parquet.enableVectorizedReader
- 向量化读是主流大数据分析引擎的标准实践，可以极大的提升查询性能
- Spark 以 Batch 的方式从 Parquet 读取数据，下推的逻辑也会适配 Batch 的方式

### ORC （大数据分析领域使用最广的列存格式之一）
#### ACID 特性
- 支持 Hive Transactions 实现，目前只有 Hive 本身集成
- 类似 Delta Lake / Hudi / Iceberg
- 基于 Base + Delta + Compaction 的设计

#### 索引增强
- 支持 Clusterd Index，更快的主键查找
- 支持 Bitmap Index，更快的过滤

#### 其他优化
- 小列聚合，减少小 IO
  - 重排 ColumnChunk
 ![image](https://user-images.githubusercontent.com/91240419/184048439-4fcdffd8-dba3-4db8-98b9-76e37886b16d.png)
- 异步预取优化
  -在计算引擎处理已经读到的数据的时候，异步去预取下一批次数据
 ![image](https://user-images.githubusercontent.com/91240419/184048463-1ebd6b7c-69b4-4004-8e96-222db20719bd.png)

#### Parquet vs ORC 对比
- 从原理层面，最大的差别就是对于 NestedType 和复杂类型处理上
- Parquet 的算法上要复杂很多，带来的 CPU 的开销比 ORC 要略大
- ORC 的算法上相对加单，但是要读取更多的数据
- 因此，这个差异的对业务效果的影响，很难做一个定性的判定，更多的时候还是要取决于实际的业务场景

### 列存演进
#### 数仓中的列存
- 典型的数仓，例如 ClickHouse 的 MergeTree 引擎也是基于列存构建的
  - 默认情况下列按照 Column 拆分成单独的文件，也支持单个文件形式
![image](https://user-images.githubusercontent.com/91240419/184048545-9900e857-614d-4eef-8539-36d840364b70.png)
- 支持更加丰富的索引，例如 Bitmap Index、Reverted Index、Data Skipping Index、Secondary Index 等
- 湖仓一体的大趋势下，数仓和大数据数据湖技术和场景下趋于融合，大数据场景下的格式层会借鉴更多的数仓中的技术

#### 存储侧下推
- 更多的下推工作下沉到存储服务侧
- 越接近数据，下推过滤的效率越高
  -例如 AWS S3 Select 功能
![image](https://user-images.githubusercontent.com/91240419/184048601-6a92598d-5de8-40f1-9c76-3ca132a7157e.png)
挑战：
- 存储侧感知 Schema
- 计算生态的兼容和集成

#### Column Family 支持
- 背景：Hudi 数据湖场景下，支持部分列的快速更新
- 在 Parquet 格式里引入 Column Family 概念，把需要更新的列拆成独立的 Column Family
- 深度改造 Hudi 的 Update 和 Query 逻辑，根据 Column Family 选择覆盖对应的 Column Family
- Update 操作实际效果有 10+ 倍的提升
![image](https://user-images.githubusercontent.com/91240419/184048659-cfb681a2-7dbb-423b-92bd-4b9ac9ee2b6e.png)

## LSMT 存储引擎浅析
较早的数据库产品，如 MySQL，PostgresQL 默认均采用 B+Tree（B-Tree 变种）索引。较新的数据库产品，如 TiDB，CockroachDB，默认均采用 LSMT 存储引擎（RocksDB / Pebble）。  
LSMT 模型变得越来越流行。LSMT 模型广泛应用于目前的数据库系统，例如 Google BigTable，HBase，Canssandra，RocksDB 等，可以说是数据库存储子系统的基石之一。  

### LSMT 是如何工作的？
一言以蔽之，通过 Append-only Write + 择机 Compact 来维护索引树的结构。  
![image](https://user-images.githubusercontent.com/91240419/184476346-01851380-a048-4c20-a609-2c993c38539b.png)
数据先写入 MemTable，MemTable 是内存中的索引可以用 SkipList / B+Tree 等数据结构实现。当 MemTable 写到一定阈值后，冻结，成为 ImmemTable，任何修改只会作用于 MemTable，所以 ImmemTable 可以被转交给 Flush 线程进行写盘操作而不用担心并发问题。Flush 线程收到 ImmemTable ，在真正执行写盘前，会进一步从 ImmemTable 生成 SST(Sorted String Table)，其实也就是存储在硬盘上的索引，逻辑上和 ImmemTable 无异。  
新生成的 SST 会存放于 L0(Layer 0)，除了 L0 以外根据配置可以一直有 Ln。SST 每 Compact 一次，就会将 Compact 产物放入下一层。Compact 可以大致理解为 Merge Sort，就是将多个 SST 去掉无效和重复的条目并合并生成新的 SST 的过程。Compact 策略主要分为 Level 和 Tier 两种，会在课中进行更详细的描述。

### 为什么要采用 LSMT 模型
机械硬盘的读写依赖于磁盘的旋转和机械臂移动。工程上一般估计机械硬盘的点查（主要开销是 Seek 寻道）延迟是 1ms。即使每次点查都读 4KB（对于点查来说相当大了），也就只能输出约 4MB/s。  
反观顺序写，由于不需要寻道，磁头始终能处在工作状态，基本都能做到至少 100MB/s 写吞吐，是点查的 25 倍！

SSD 时代：    
顺序写和随机写的不对称性  
![image](https://user-images.githubusercontent.com/91240419/184476408-d61a1a60-4de4-4ba3-9291-3a6c5a786656.png)
SSD 是基于 NAND Flash 颗粒的构建的，称之为 DIE，DIE 上有多个 Plane，每个 Plane 能单独提供读写能力，Plane 包含多个 Block，Block 包含多个 Page。擦除的电路实现比较复杂，出于成本的考量，写入的最小单位是 Page，而擦除的最小单位是 Block。  
随着用户不断写入和删除，有可能出现有很多 Page 已经被删除了，逻辑上有可用空间，但是物理上 Block 还有别的有效 Page，无效 Page 无法回收。这样用户就写不进数据了。因此，SSD 主控必须执行 GC(Garbage Collection)，将有效的 Page 从要回收的 Block 中挑出来，写到另一个 Block 上，再整体回收旧 Block。因此如果用户长期都是随机写，大量 Block 都会处于一部分 Page 是有效，一部分 Page 是无效的状态，SSD 主控不得不频繁 GC。  
以经典服务器 SSD，Intel P4510 2TB 为例，根据官方 spec，随机写吞吐是 318MB/s，顺序写则高达 2000MB/s 是随机写的 6 倍多！  
简单总结一下，无论对于 HDD 还是 SSD，顺序写都是一个很好的特质，LSMT 符合这一点，B+Tree 则依赖原地更新，会导致随机写。    
  
传统数据库大致可以分为
- 计算层
- 存储层（存储引擎层）
介于二者之间还有一些界限比较模糊的组件，比如 Replication，MySQL 是用 bin log 独立于存储引擎，而对于一些 NoSQL 数据库（字节 Abase 1.0）来说，Replication 直接基于存储引擎的 WAL。
计算层主要负责 SQL 解析/ 查询优化 / 计划执行。我们重点关注存储层提供了什么能力。数据库著名的 ACID 特性，在 MySQL 中全部强依赖于存储引擎。
ACID 定义：
Atomicity
- 原子性依赖于存储引擎 WAL(Redo Log)
Consistency (Correctness)
- 一致性需要数据库整体来保证
Isolation
- 隔离性依赖于存储引擎提供 Snapshot（有时候会直接说 MVCC）能力。如果上层没有单独的事务引擎的话，也会由存储引擎提供事务能力。一般的是实现是 2PL（2 Phase Lock） + MVCC。2PL 可以简单理解为对所有需要修改的资源上锁。
Durability
- 持久性依赖于存储引擎确保在 Transaction Commit 后通过操作系统 fsync 之类的接口确保落盘了

### LSMT 与 B+Tree 的异同
先简单回顾下经典 B+Tree 写入流程
![image](https://user-images.githubusercontent.com/91240419/184476500-01510513-350d-47e4-978a-faed61749b86.png)
有一 Order 为 5 的 B+Tree，目前存有 (10, 20, 30, 40)，继续插入 15，节点大小到达分裂阈值 5，提取中位数 20 放入新的内部节点，比 20 大的 (30, 40) 移入新的叶节点。这个例子虽然简单，但是涉及了 B+Tree 最核心的两个变化，插入与分裂。  
在 B+Tree 中，数据插入是原地更新的，装有 (10, 20, 30, 40) 的节点在插入和分裂后，原节点覆写成 (10, 15)。此外，B+Tree 在发生不平衡或者节点容量到达阈值后，必须立即进行分裂来平衡。
反观 LSMT，数据的插入是追加的（Append-only），当树不平衡或者垃圾过多时，有专门 Compact 线程进行 Compact，可以称之为延迟（Lazy）的。  
思考一个问题，B+Tree 能不能把部分数据采用追加写，然后让后台线程去 Compact 维护树结构呢？或者 LSMT 能不能只有一层 L0，ImmemTable 给 Flush 线程之后，立马 Compact 呢？  
答案是都可以。前者的做法叫做 Fractal tree（分型树）应用在了 TokuDB 中。后者的做法在 OceanBase 或者类似对延迟有严格要求的在线数据库中得到了应用，因为 LSMT 层数越少，读取越快。
所以从高层次的数据结构角度来看，B+Tree 和 LSMT 并没有本质的不同，可以统一到一个模型里，根据 Workload 的不同互相转换。  

B+Tree 中内部节点指向其它节点的指针，被称之为 Fence Pointers。在 LSMT 也有，只不过是隐式表达的。B+Tree 直接通过 Fence Pointer 一层一层往下找，而 LSMT 是有一个中心的 Meta 信息记录所有 SST 文件的 Key 区间，通过区间大小关系，一层一层向下找。  
再看 LSMT 的 SST，其实和 B+Tree 的 Node 也没有本质差别，逻辑上就是一个可查询的有序块，统一模型中称之为 Run。B+Tree 为了支持随机修改，结构会比较松散和简单，LSMT 则因为不需要支持随机修改，利用压缩技术，结构可以更紧凑。  
更详细的统一模型描述，请同学们参见论文。尽管 LSMT 和 B+Tree 可以用一个模型描述，工程实践上我们还是用 LSMT 来表示一个 Append-only 和 Lazy Compact 的索引树，B+Tree 来表示一个 Inplace-Update 和 Instant Compact 的索引树。Append-only 和 Lazy Compact 这两个特性更符合现代计算机设备的特性。  
### LSMT 存储引擎的优势
- 相对于 B+Tree 的优势  
我们在前文已经阐述了 LSMT 与 B+Tree 的异同，在这里总结下 LSMT 的优势。
  - 顺序写模型对于 SSD 设备更友好
  - SST 不可修改的特性使得其能使用更加紧凑的数据排列和加上压缩
  - 后台延迟 Compact 能更好利用 CPU 多核处理能力，降低前台请求延迟
- 相对于 HashTable 的优势
  - LSMT 存储引擎是有序索引抽象，HashTable 是无序索引抽象。无序索引是有序索引的真子集。LSMT 相比于 HashTable 更加通用。HashTable 能处理点查请求，LSMT 也能，但 LSMT 能处理 TopK 请求，但 HashTable 就不行了。为了避免维护多套存储引擎，绝大多数数据库都直接采用一套有序的存储引擎而非针对点查和顺序读取分别维护两个引擎。
### LSMT 存储引擎的实现，以 RocksDB 为例
RocksDB 是一款十分流行的开源 LSMT 存储引擎，最早来自 Facebook（Meta），应用于 MyRocks，TiDB，在字节内部也有 Abase，ByteKV，ByteNDB，Bytable 等用户。因此接下来将会以 RocksDB 为例子介绍 LSMT 存储引擎的经典实现。
#### Write
为了确保操作的原子性，RocksDB 在真正执行修改之前会先将变更写入 WAL（Write Ahead Log），WAL 写成功则写入成功。因为即使这时候程序 crash，在重启阶段可以通过回放 WAL 来恢复或者继续之前的变更。操作只有成功和失败两种状态。  
RocksDB WAL 写入流程继承自 LevelDB。LevelDB 在 WAL 写入主要做的一个优化是多个写入者会选出一个 Leader，由这个 Leader 来一次性写入。这样的好处在于可以批量聚合请求，避免频繁提交小 IO。  
但很多业务其实不会要求每次 WAL 写入必须落盘，而是写到 Kernel 的 Page Cache 就可以，Kernel 自身是会聚合小 IO 再下刷的。这时候，批量提交的好处就在于降低了操作系统调度线程的开销。
批量提交时，Leader 可以同时唤醒其余 Writer。  
![image](https://user-images.githubusercontent.com/91240419/184476617-448e445f-7ffc-481d-9ea7-f3ff10b82ed8.png)
如果没有批量提交就只能链式唤醒了。
![image](https://user-images.githubusercontent.com/91240419/184476625-b5242bf3-bf67-41f8-b74e-c7739a7fa8b6.png)
写完 WAL 实际还要写 MemTable，这步相比于写 WAL 到 Page Cache 更耗时而且是可以完全并行化的。RocksDB 在 LevelDB 的基础上主要又添加了并发 MemTable 写入的优化，由最后一个完成 MemTable 写入的 Writer 执行收尾工作。完整 RocksDB 写入流程如下：  
为了方便更好表明哪些事件是同时发生的，相同时刻的事件的背景颜色是一样的。  
![image](https://user-images.githubusercontent.com/91240419/184476634-9b9c2e0e-dc0f-4a98-ab5a-bdf4fefa405a.png)
RocksDB 为了保证线性一致性，必须有一个 Leader 分配时间戳，每条修改记录都会带着分配到的时间戳，也必须有一个 Leader 推进当前可见的时间戳。目前的写入流程已经相当优化了。
#### Snapshot & SuperVision
RocksDB 的数据由 3 部分组成，MemTable / ImmemTable / SST。直接持有这三部分数据并且提供快照功能的组件叫做 SuperVersion。  
![image](https://user-images.githubusercontent.com/91240419/184476646-9cd14aaa-8513-4b00-8d79-440031b4a4a4.png)
RocksDB 的 MemTable 和 SST 的释放与删除都依赖于引用计数，SuperVersion 不释放，对应的 MemTable 和 SST 就不会释放。对于读取操作来说，只要拿着这个 SuperVersion，从 MemTable 开始一级一级向下，就能查询到记录。那么拿着 SuperVersion 不释放，等于是拿到了快照。  
如果所有读者开始操作前都给 SuperVersion 的计数加 1，读完后再减 1，那么这个原子引用计数器就会成为热点。CPU 在多核之间同步缓存是有开销的，核越多开销越大。一般工程上可以简单估计，核多了之后 CAS 同一个 cache line，性能不会超过 100W/s。为了让读操作更好的 scale，RocksDB 做了一个优化是 Thread Local SuperVersion Cache。每个读者都缓存一个 SuperVersion，读之前检查下 SuperVersion 是否过期，如果没有就直接用这个 SuperVersion，不需要再加减引用计数器。如果 SuperVersion 过期了，读者就必须刷新一遍 SuperVersion。为了避免某一个读者的 Thread Local 缓存持有一个 SuperVersion 太久导致资源无法回收，每当有新的 SuperVersion 生成时会标记所有读者缓存的 SuperVersion 失效。  
没有 Thread Local 缓存时，读取操作要频繁 Acquire 和 Release SuperVersion  
![image](https://user-images.githubusercontent.com/91240419/184476679-d05db543-910b-4124-b910-a169bf1b5448.png)
有 Thread Local 缓存时，读取只需要检查一下 SuperVersion 并标记缓存正在使用即可，可以看出多核之间的交互就仅剩检查 SuperVersion 缓存是否过期了。
![image](https://user-images.githubusercontent.com/91240419/184476683-3a2f17bc-e856-401d-8da0-32571949ae2d.png)
#### Get & BloomFilter
由于 LSMT 是延迟 Compact 的且 SST 尺寸（MB 级别）比 B+Tree Node （KB 级别）大得多。所以相对而言，LSMT 点查需要访问的数据块更多。为了加速点查，一般 LSMT 引擎都会在 SST 中嵌入 BloomFilter，例如 RocksDB 默认的 BlockBasedTable。BloomFilter 可以 100% 断言一个元素不在集合内，但只能大概率判定一个元素在集合内。  
RocksDB 的读取在大框架上和 B+ Tree 类似，就是层层向下。[1, 10] 表示这个索引块存储数据的区间在 1 - 10 之间。索引块可以是 MemTable / ImmemTable / SST，它们抽象上是一样的。查询 2，就是顺着标绿色的块往下。如果索引块是 SST，就先查询 BloomFilter，看数据是否有可能在这个 SST 中，有的话则进行进一步查询。  
![image](https://user-images.githubusercontent.com/91240419/184476694-4b045a98-7016-47e9-a40b-ce356ac76044.png)
除了 BloomFilter 外，BlockBasedTable 还有额外两个值得提的实现。一个是两层索引：
![image](https://user-images.githubusercontent.com/91240419/184476705-3db8bc31-74dd-4555-bce7-35692925d906.png)
浅黄部分是 DataBlock，绿色部分是 IndexBlock。DataBlock 记载实际数据，IndexBlock 索引 DataBlock。假如要查询 3，先从 IndexBlock 中找到 >= 3 的第一条记录是什么，发现是 4，对应的 value 是 data_block_0 的 offset，直接定位到 Data Block 0。然后可以在 Data Block 0 中进行搜索。  
另一个是前缀压缩，RocksDB 源代码中的注释已经写得很明白了。
#### Compact
Compact 在 LSMT 中是将 Key 区间有重叠或无效数据较多的 SST 进行合并，以此来加速读取或者回收空间。Compact 策略可以分为两大类。
- Level
![image](https://user-images.githubusercontent.com/91240419/184476728-c69f05a9-8b10-41c9-8948-6a22eb2baffa.png)
Level 策略直接来自于 LevelDB，也是 RocksDB 的默认策略。每一个层不允许有 SST 的 Key 区间重合。当用户写入的 SST 加入 L0 的时候会和 L0 里区间重叠的 SST 进行合并。当 L0 的总大小到达一定阈值时，又会从 L0 挑出 SST，推到 L1，和 L1 里 Key 区间重叠的 SST 进行合并。Ln 同理。  
由于在 LSMT 中，每下一层都会比上一层大 T 倍（可配置），那么假设用户的输入是均匀分布的，每次上下层的合并都一定是一个小 SST 和一个大 SST 进行 Compact。这个从算法的角度来说是低效的，增加了写放大，具体理论分析会在之后阐述，这里可以想象一下 Merge Sort。Merge Sort 要效率最高，就要每次 Merge 的时候，左右两边的数组都是一样大。  
实际上，RocksDB 和 LevelDB 都不是纯粹的 Level 策略，它们将 L0 作为例外，允许有 SST Key 区间重叠来降低写放大。  
- Tier
![image](https://user-images.githubusercontent.com/91240419/184476738-76a3daf6-1db0-40c2-8387-44c3a5b2d140.png)
Tier 策略允许 LSMT 每层有多个区间重合的 SST，当本层区间重合的 SST 到达上限或者本层大小到达阈值时，一次性选择多个 SST 合并推向下层。Tier 策略理论上 Compact 效率更高，因为参与 Compact 的 SST 大小预期都差不多大，更接近于完美的 Merge Sort。  
Tier 策略的问题在于每层的区间内重合的 SST 越多，那么读取的时候需要查询的 SST 就越多。Tier 策略是用读放大的增加换取了写放大的减小。  
#### Cloud-Native LSMT Storage Engine
RocksDB 是单机存储引擎，那么现在都说云原生，HBase 比 RocksDB 就更「云」一些，SST 直接存储于 HDFS 上，Meta 信息 RocksDB 自己管理维护于 Manifest 文件，HBase 放置于 ZK。二者在理论存储模型上都是 LSMT。
![image](https://user-images.githubusercontent.com/91240419/184476760-61cb4d7b-4a2e-4eab-90d5-7f00512acf09.png)
### Level
Write：每条记录抵达最底层需要经过 L 次 Compact，每次 Compact Ln 的一个小 SST 和 Ln+1 的一个大 SST。设小 SST 的大小为 1，那么大 SST 的大小则为 T，合并开销是 1+T，换言之将 1 单位的 Ln 的 SST 推到 Ln+1 要耗费 1+T 的 IO，单次 Compact 写放大为 T。每条记录的写入成本为 1/B 次最小单位 IO。三者相乘即得结果。
Point Lookup：对于每条 Key，最多有 L 个重叠的区间，每个区间都有 BloomFilter，失效率为e−MNe^{- \frac{M}{N} } e−NM​，只有当 BloomFilter 失效时才会访问下一层。因此二者相乘可得读取的开销。注意，这里不乘 1/B 的原因是写入可以批量提交，但是读取的时候必须对齐到最小读取单元尺寸。
### Tier
Write：每条记录抵达最底层前同样要经过 L 次 Compact，每次 Compact Ln 中 T 个相同尺寸的 SST 放到 Ln+1。设 SST 大小为 1，那么 T 个 SST Compact 的合并开销是 T，换言之将 T 单位的 Ln 的 SST 推到 Ln+1 要耗费 T 的 IO，单次 Compact 的写放大为 T / T = 1。每条记录的写入成本为 1/B 次最小单位 IO。三者相乘即得结果。
Point Lookup：对于每条 Key，有 L 层，每层最多有 T 个重叠区间的 SST，对于整个 SST 来说有 T *
L 个可能命中的 SST，乘上 BloomFilter 的失效率即可得结果。
#### 总结，Tier 策略降低了写放大，增加了读放大和空间放大，Level 策略增加了写放大，降低了读和空间放大。

### LSMT 引擎调优案例 
TerarkDB aka LavaKV 是字节跳动内部基于 RocksDB 深度定制优化的自研 LSMT 存储引擎，其中完全自研的 KV 分离功能，上线后取得了巨大的收益。  
KV 分离受启发于论文 WiscKey: Separating Keys from Values in SSD-conscious Storage，www.usenix.org/system/file… 较长的记录的 Value 单独存储，避免 Compact 过程中频繁挪动这些数据。做法虽然简单，但背后的原理却十分深刻。存储引擎其实存了两类数据，一类是索引，一类是用户输入的数据。对于索引来说，随着记录不断变更，需要维护索引的拓扑结构，因此要不断 Compact，但对于用户存储的数据来说，只要用户没删除，可以一直放着，放哪里不重要，能读就行，不需要经常跟着 Compact。只要 Value 足够长，更少 Compact 的收益就能覆盖 KV 分离后，额外维护映射关系的开销。  

收益说明：
- 平均 CPU 收益主要来自于，开启 KV 分离，减少写放大
- 容量收益主要来自于 schedule TTL GC，该功能可以根据 SST 的过期时间主动发起Compaction，而不需要被动的跟随 LSM-tree 形态调整回收空间


