# Bytesummer
#第四届字节跳动夏令营笔记

![image](https://user-images.githubusercontent.com/91240419/180639073-33fb3c74-3823-4bcc-b08f-7b7f0700877f.png)
消息队列一般用于解耦计算与存储
## SQL 查询优化器浅析
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
检查并绑定Database，Table，Column等原信息  
检查SQL合法性，比如max、min、avg的输入是数值  
AST->Logical Plan
#### 逻辑计划（Logical Plan） 
![image](https://user-images.githubusercontent.com/91240419/180640113-d27cf196-35bd-4cf9-99bc-b5eba258a2e6.png)
### 查询优化
SQL是声明式语言，用户只描述了做什么，没有告诉数据库怎么做  
作用：  
找到一个正确且代价最小的物理执行计划
#### 物理计划（Physical Plan）  
分为：  
Plan Fragment：执行计划子树 
最小化网络数据传输，把逻辑计划拆分成多个物理计划  
查询优化器需要感知数据分布，利用数据的物理分布（数据亲和性）  
增加Shuffle算子  
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
使用一个模型估算执行计划的代价，选择代价最小的执行计划  
执行计划的代价：所有算子的执行代价之和  
算子代价：CPU,内存，磁盘I/O，网络I/O等代价    
与算计的类型和输入数据的统计信息有关（输入输出的行数大小）  
#### 如何收集统计信息
![image](https://user-images.githubusercontent.com/91240419/180646730-a97b7709-cf6e-4d0f-840f-dd7fca0055f2.png)
-CBO的枚举执行计划  
动态规划和贪心算法  
-哈希连接（Hash Join）：将其中一个表的连接字段计算出一个哈希表，然后从另一个表中一次获取记录并计算哈希值，根据两个哈希值来匹配符合条件的记录。这种方式在数据量大且没有创建索引的情况下的性能可能更好。
-排序合并连接（Sort Merge Join）：首先将两个表中的数据基于连接字段分别进行排序，然后合并排序后的结果。这种方式通常用于没有创建索 引，并且数据已经排序的情况。


##流/批/OLAP 一体的 Flink 引擎介绍
### Flink分层架构
![image](https://user-images.githubusercontent.com/91240419/181015794-7692f649-d837-4654-afb0-d0c35d604383.png)
SDK 层：Flink's APIs Overview；  
执行引擎层（Runtime 层）：执行引擎层提供了统一的 DAG，用来描述数据处理的 Pipeline，不管是流还是批，都会转化为 DAG 图，调度层再把 DAG 转化成分布式环境下的 Task，Task 之间通过 Shuffle 传输数据；  
调度：Jobs and Scheduling；  
Task 生命周期：Task Lifecycle；  
Flink Failover 机制：Task Failure Recovery；  
Flink 反压概念及监控：Monitoring Back Pressure；  
Flink HA 机制：Flink HA Overview；  
状态存储层：负责存储算子的状态信息  
### Flink整体架构
![image](https://user-images.githubusercontent.com/91240419/181015949-dbd553b1-34ed-4706-806d-4deb01b1f484.png)
![image](https://user-images.githubusercontent.com/91240419/181016019-78c0da08-eafc-40a8-b2fe-3d317e14a7d6.png)
#### JobManager（JM）负责整个任务的协调工作，包括：调度 task、触发协调 Task 做 Checkpoint、协调容错恢复等，核心有下面三个组件：
Dispatcher: 接收作业，拉起 JobManager 来执行作业，并在 JobMaster 挂掉之后恢复作业；  
JobMaster: 管理一个 job 的整个生命周期，会向 ResourceManager 申请 slot，并将 task 调度到对应 TM 上；  
ResourceManager：负责 slot 资源的管理和调度，Task manager 拉起之后会向 RM 注册；  
TaskManager（TM）：负责执行一个 DataFlow Graph 的各个 task 以及 data streams 的 buffer 和数据交换。  
### --
Shuffle：在分布式计算中，用来连接上下游数据交互的过程叫做Shuffle  
![image](https://user-images.githubusercontent.com/91240419/181016635-fc3b9cc9-51c8-4e0d-a78a-ea95eda85d23.png)
#### Apache Flink 主要从以下几个模块来做流批一体：
SQL 层；  
DataStream API 层统一，批和流都可以使用 DataStream API 来开发；  
Scheduler 层架构统一，支持流批场景；  
Failover Recovery 层 架构统一，支持流批场景；  
Shuffle Service 层架构统一，流批场景选择不同的 Shuffle Service；  
#### 流批一体的 Scheduler 层
Scheduler 主要负责将作业的 DAG 转化为在分布式环境中可以执行的 Task；  
1.12 之前的 Flink 版本，Flink 支持两种调度模式：
EAGER（Streaming 场景）：申请一个作业所需要的全部资源，然后同时调度这个作业的全部 Task，所有的 Task 之间采取 Pipeline 的方式进行通信；  
LAZY（Batch 场景）：先调度上游，等待上游产生数据或结束后再调度下游，类似 Spark 的 Stage 执行模式。  
Pipeline Region Scheduler 机制：FLIP-119 Pipelined Region Scheduling - Apache Flink - Apache Software Foundation；  
#### 流批一体的 Shuffle Service 层（FLIP-31: Pluggable Shuffle Service - Apache Flink - Apache Software Foundation）
Shuffle：在分布式计算中，用来连接上下游数据交互的过程叫做 Shuffle。实际上，分布式计算中所有涉及到上下游衔接的过程，都可以理解为 Shuffle；  
#### Shuffle 分类：  
基于文件的 Pull Based Shuffle，比如 Spark 或 MR，它的特点是具有较高的容错性，适合较大规模的批处理作业，由于是基于文件的，它的容错性和稳定性会更好一些；  
基于 Pipeline 的 Push Based Shuffle，比如 Flink、Storm、Presto 等，它的特点是低延迟和高性能，但是因为 shuffle 数据没有存储下来，如果是 batch 任务的话，就需要进行重跑恢复；  
#### 流和批 Shuffle 之间的差异：
Shuffle 数据的生命周期：流作业的 Shuffle 数据与 Task 是绑定的，而批作业的 Shuffle 数据与 Task 是解耦的；  
Shuffle 数据存储介质：流作业的生命周期比较短、而且流作业为了实时性，Shuffle 通常存储在内存中，批作业因为数据量比较大以及容错的需求，一般会存储在磁盘里；  
Shuffle 的部署方式：流作业 Shuffle 服务和计算节点部署在一起，可以减少网络开销，从而减少 latency，而批作业则不同。  
Pluggable Shuffle Service：Flink 的目标是提供一套统一的 Shuffle 架构，既可以满足不同 Shuffle 在策略上的定制，同时还能避免在共性需求上进行重复开发  
#### Flink 流批一体总结
经过相应的改造和优化之后，Flink 在架构设计上，针对 DataStream 层、调度层、Shuffle Service 层，均完成了对流和批的支持。  
业务已经可以非常方便地使用 Flink 解决流和批场景的问题了。  
### 案例
![image](https://user-images.githubusercontent.com/91240419/181025563-c928a594-fb2e-4f11-be2e-8b797bf5603c.png)  
![image](https://user-images.githubusercontent.com/91240419/181025700-3466181c-5426-4f4f-8dde-0de6685b2630.png)  
![image](https://user-images.githubusercontent.com/91240419/181025827-2c07c4bc-c0e7-471b-84f2-b0118841f49e.png)


## Exactly Once 语义在 Flink 中的实现
动态表 ： 随时间不断变化的表，在任意时刻，可以像查询静态批处理表一样查询它们
#### 实时流的查询特点？
查询从不终止   
查询结果会不断更新，并且会产生一个新的动态表  
结果的动态表也可转换成输出的实时流    
#### 动态表到实时流的转换  
Append-only Stream: Append-only 流（只有 INSERT 消息）  
Retract Stream: Retract 流（同时包含 INSERT 消息和 DELETE 消息）  
### 三种语义
1.At-most-once:出现故障的时候，啥也不做。数据处理不保证任何语义，处理时延低。
2.At-least-once:保证每条数据均至少被处理一次，一条数据可能存在重复消费。
3.Exactly-once:最严格的处理语义，从输出结果来看，每条数据均被消费且仅消费一次，仿佛故障从未发生。

### 两阶段提交
![image](https://user-images.githubusercontent.com/91240419/181522513-9f668235-02e8-4f52-910d-b3affd34f871.png)

## 流式计算中的window机制
###回顾
动态表  
flink中的state和checkpoint的基本原理  
flink中的retract机制，以及算子如何产生和处理retract数据  
flink中如何实现exactly-once语义
### watermark
Watermark定义：当前系统认为的事件时间所在的真实时间。   
Watermark产生：一般是从数据的事件时间来产生，产生策略可以灵活多样，最常见的包括使用当前事件时间的时间减去一个固定的delay，来表示可以可以容忍多长时间的乱序。  
Watermark传递：这个类似于上节课中介绍的Checkpoint的制作过程，传递就类似于Checkpoint的barrier，上下游task之间有数据传输关系的，上游就会将watermark传递给下游；下游收到多个上游传递过来的watermark后，默认会取其中最小值来作为自身的watermark，同时它也会将自己watermark传递给它的下游。经过整个传递过程，最终系统中每一个计算单元就都会实时的知道自身当前的watermark是多少。
![image](https://user-images.githubusercontent.com/91240419/181904830-eca0bd40-0c71-4273-b010-a150a5efb69b.png)
#### 怎么观察一个任务中的watermark是多少，是否是正常的
一般通过Flink Web UI上的信息来观察当前任务的watermark情况  
这个问题是生产实践中最容易遇到的问题，大家在开发事件时间的窗口任务的时候，经常会忘记了设置watermark，或者数据太少，watermark没有及时的更新，导致窗口一直不能触发。
#### Per-partition / Per-subtask 生成watermark的优缺点
在Flink里早期都是per-subtask的方式进行watermark的生成，这种方式比较简单。但是如果每个source task如果有消费多个partition的情况的话，那多个partition之间的数据可能会因为消费的速度不同而最终导致数据的乱序程度增加。  
后期（上面图中）就逐步的变成了per-partition的方式来产生watermark，来避免上面的问题。
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

## spark的原理与实践
![image](https://user-images.githubusercontent.com/91240419/181902804-88504c25-b318-49f8-8324-38afd40f4718.png)
Spark应用在集群上运行时，包括了多个独立的进程，这些进程之间通过驱动程序（Driver Program）中的SparkContext对象进行协调，SparkContext对象能够与多种集群资源管理器（Cluster Manager）通信，一旦与集群资源管理器连接，Spark会为该应用在各个集群节点上申请执行器（Executor），用于执行计算任务和存储数据。Spark将应用程序代码发送给所申请到的执行器，SparkContext对象将分割出的任务（Task）发送给各个执行器去运行。
#### 需要注意的是  
1.每个Spark application都有其对应的多个executor进程。Executor进程在整个应用程序生命周期内，都保持运行状态，并以多线程方式执行任务。这样做的好处是，Executor进程可以隔离每个Spark应用。从调度角度来看，每个driver可以独立调度本应用程序的内部任务。从executor角度来看，不同Spark应用对应的任务将会在不同的JVM中运行。然而这样的架构也有缺点，多个Spark应用程序之间无法共享数据，除非把数据写到外部存储结构中。  
2.Spark对底层的集群管理器一无所知，只要Spark能够申请到executor进程，能与之通信即可。这种实现方式可以使Spark比较容易的在多种集群管理器上运行，例如Mesos、Yarn、Kubernetes。  
Driver Program在整个生命周期内必须监听并接受其对应的各个executor的连接请求，因此driver program必须能够被所有worker节点访问到。  
3.因为集群上的任务是由driver来调度的，driver应该和worker节点距离近一些，最好在同一个本地局域网中，如果需要远程对集群发起请求，最好还是在driver节点上启动RPC服务响应这些远程请求，同时把driver本身放在离集群Worker节点比较近的机器上。
### spark core
![image](https://user-images.githubusercontent.com/91240419/181902998-6ba9e609-1dc8-4577-a7cf-10a1dfd6b2bf.png)
#### RDD  
![image](https://user-images.githubusercontent.com/91240419/181905426-b643ddbf-e4e3-49fe-a01c-bf6786a7749f.png)
1.并行执行的分布式数据集，spark中数据处理模型   
2.分区（决定并行执行的数量）
3.前后依赖其他rdd
划分Stage的整体思路：从后往前推，遇到宽依赖就断开，划分为一个Stage。遇到窄依赖，就将这个RDD加入该Stage中，DAG最后一个阶段会为每个结果的Partition生成一个ResultTask。每个Stage里面的Task数量由最后一个RDD的Partition数量决定，其余的阶段会生成ShuffleMapTask。    
当RDD对象创建后，SparkContext会根据RDD对象构建DAG有向无环图，然后将Task提交给DAGScheduler。DAGScheduler根据ShuffleDependency将DAG划分为不同的Stage，为每个Stage生成TaskSet任务集合，并以TaskSet为单位提交给TaskScheduler。TaskScheduler根据调度算法(FIFO/FAIR)对多个TaskSet进行调度，并通过集群中的资源管理器(Standalone模式下是Master，Yarn模式下是ResourceManager)把Task调度(locality)到集群中Worker的Executor，Executor由SchedulerBackend提供。  
RDD算子  
1.transform算计：生成一个新的rdd 
2.action算子：触发job提交  
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
. SQL Parse： 将SparkSQL字符串或DataFrame解析为一个抽象语法树/AST，即Unresolved Logical Plan  
. Analysis：遍历整个AST，并对AST上的每个节点进行数据类型的绑定以及函数绑定，然后根据元数据信息Catalog对数据表中的字段进行解析。 利用Catalog信息将Unresolved Logical Plan解析成Analyzed Logical plan  
. Logical Optimization：该模块是Catalyst的核心，主要分为RBO和CBO两种优化策略，其中RBO是基于规则优化，CBO是基于代价优化。 利用一些规则将Analyzed Logical plan解析成Optimized Logic plan  
. Physical Planning: Logical plan是不能被spark执行的，这个过程是把Logic plan转换为多个Physical plans  
. CostModel: 主要根据过去的性能统计数据，选择最佳的物理执行计划(Selected Physical Plan)。  
. Code Generation: sql逻辑生成Java字节码
##### 影响SparkSQL性能两大技术：
1.Optimizer：执行计划的优化，目标是找出最优的执行计划  
2.Runtime：运行时优化，目标是在既定的执行计划下尽可能快的执行完毕。  

#### Catalyst优化
. Rule Based Optimizer(RBO): 基于规则优化，对语法树进行一次遍历，模式匹配能够满足特定规则的节点，再进行相应的等价转换。  
. Cost Based Optimizer(CBO): 基于代价优化，根据优化规则对关系表达式进行转换，生成多个执行计划，然后CBO会通过根据统计信息(Statistics)和代价模型(Cost Model)计算各种可能执行计划的代价，从中选用COST最低的执行方案，作为实际运行方案。CBO依赖数据库对象的统计信息，统计信息的准确与否会影响CBO做出最优的选择。
#### AQE
AQE对于整体的Spark SQL的执行过程做了相应的调整和优化，它最大的亮点是可以根据已经完成的计划结点真实且精确的执行统计结果来不停的反馈并重新优化剩下的执行计划。
##### AQE框架三种优化场景：  
. 动态合并shuffle分区（Dynamically coalescing shuffle partitions）  
. 动态调整Join策略（Dynamically switching join strategies）  
. 动态优化数据倾斜Join（Dynamically optimizing skew joins）
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
