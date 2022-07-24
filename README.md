# Bytesummer
#第四届字节跳动夏令营笔记

## SQL 查询优化器浅析
![image](https://user-images.githubusercontent.com/91240419/180639073-33fb3c74-3823-4bcc-b08f-7b7f0700877f.png)
消息队列一般用于解耦计算与存储
## SQL的处理流程
![image](https://user-images.githubusercontent.com/91240419/180639390-84292810-2b7a-4a6a-b108-17c91933c9fa.png)
### Parser
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

