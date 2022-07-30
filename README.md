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
### Flink整体架构
![image](https://user-images.githubusercontent.com/91240419/181015949-dbd553b1-34ed-4706-806d-4deb01b1f484.png)
![image](https://user-images.githubusercontent.com/91240419/181016019-78c0da08-eafc-40a8-b2fe-3d317e14a7d6.png)
### --
Shuffle：在分布式计算中，用来连接上下游数据交互的过程叫做Shuffle  
![image](https://user-images.githubusercontent.com/91240419/181016635-fc3b9cc9-51c8-4e0d-a78a-ea95eda85d23.png)
### 案例
![image](https://user-images.githubusercontent.com/91240419/181025563-c928a594-fb2e-4f11-be2e-8b797bf5603c.png)  
![image](https://user-images.githubusercontent.com/91240419/181025700-3466181c-5426-4f4f-8dde-0de6685b2630.png)  
![image](https://user-images.githubusercontent.com/91240419/181025827-2c07c4bc-c0e7-471b-84f2-b0118841f49e.png)


## Exactly Once 语义在 Flink 中的实现
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
### 处理时间窗口
![image](https://user-images.githubusercontent.com/91240419/181867225-06d48d7c-1107-4e98-9ab5-e4e8be087885.png)  
![image](https://user-images.githubusercontent.com/91240419/181867267-f287ec83-4367-463a-912f-8cc638f02cde.png)
![image](https://user-images.githubusercontent.com/91240419/181867260-dd737179-a289-4ecd-9186-5b06d2ec141c.png)
![image](https://user-images.githubusercontent.com/91240419/181869494-34a9b4ab-0fe7-4a71-85fa-a48f82a1adca.png)
![image](https://user-images.githubusercontent.com/91240419/181869890-0e733988-f52b-472c-90e4-16c7c8748f21.png)
![image](https://user-images.githubusercontent.com/91240419/181870178-b496b236-4f24-446d-b902-da6d071dee26.png)
![image](https://user-images.githubusercontent.com/91240419/181870117-aa38492a-342e-401b-ac24-2ed21b7dce66.png)

### window基本功能
1.滚动窗口 
2.滑动窗口  
3.会话窗口  
![image](https://user-images.githubusercontent.com/91240419/181870464-6f27f014-1376-4e40-9434-bfaba5b6f3e0.png)
![image](https://user-images.githubusercontent.com/91240419/181870524-72759ce3-2a48-424c-8cf6-08cf01f4846d.png)
![image](https://user-images.githubusercontent.com/91240419/181870532-93738ea0-9f60-4f0c-a0de-e776a6d4c20a.png)





