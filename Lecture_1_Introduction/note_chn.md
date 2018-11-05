# 6.824 2018 第一讲: 介绍

## 6.824: 分布式系统开发

### 什么是分布式系统
  * 多电脑协作
  * 大型网站的数据存储，映射归约（MapReduce），点对点的文件共享（peer-to-peer sharing）
  * 很多重要的架构都是分布式的

### 为什么要使用分布式
  * 组织管理物理层面独立的多个个体
  * 通过隔离保证安全性
  * 通过冗余保证容错性
  * 通过并行的CPU计算、内存IO、磁盘IO、网络IO达到线性提升系统处理能力

### 然而:
  * 复杂性: 多个并发的任务
  * 必须处理部分失败
  * 需要通过很巧妙的手段最大化系统性能

### 为什么要参加这门课？
  * 有趣 -- 困难的问题，强大的解决方案
  * 现实中有很多使用场景 -- 大型网站的崛起驱动了分布式系统
  * 活跃的研究领域 -- 获得了很多进展，但还有很多悬而未决的问题
  * 动手实践 -- 你将会在实验课里搭建分布式系统

## 课程结构

### http://pdos.csail.mit.edu/6.824

### 相关教职人员:
  * Malte Schwarzkopf, lecturer
  * Robert Morris, lecturer
  * Deepti Raghavan, TA
  * Edward Park, TA
  * Erik Nguyen, TA
  * Anish Athalye, TA

### 课程资料:
  * 讲义
  * 阅读材料
  * 两个考试
  * 实验课
  * 期末实验课（可选）
  * 助教QA时间
  * 可以获得通告和实验帮助的piazza

#### 讲义:
  * 总体思想，论文讨论，实验课

#### 阅读材料:
  * 研究论文，有经典的，也有最新的
  * 这些论文阐述了关键思想和一些重要细节
  * 很多讲义都以这些论文为中心
  * 请在上课前认真阅读这些论文
  * 我们为每篇论文都准备了一个简短的问题
  * 你必须针对每篇论文提出一个问题并发给我们
  * 在讲义前一晚的凌晨前提交QA问题

#### 考试:
  * 其中考试（随堂考试）
  * 期末考试（最后一周）

#### 实验目标:
  * 加深对某些重要技术的理解
  * 获得分布式编程的经验
  * 第一个实验截止时间是第二个周五
  * 第一个实验后基本每周一个实验

##### 实验1: 映射归约（MapReduce）
##### 实验2: 使用Raft算法实现可容错的复制集
##### 实验3: 可容错的键值存储
##### 实验4: 分片的键值存储
##### 最后一次实验（可选）
  * 分成2或3人的小组进行
  * 可以用来代替实验4
  * 你提出一个想法并与我们一起实现
  * 编码，简短的介绍，在最后一天演示

##### 实验分数取决于你通过的测试用例数量，我们给你提供测试用例来判断你的实验是否做的足够好。注意：假如大部分的提交都成功，偶尔会失败，那么大概率当你线上运行的时候会失败

##### Debug会花费很多时间
  * 尽早开始
  * 在助教QA时间寻求帮助
  * 在Piazza上提问

## 主题

### 这是一门关于架构的课程
  * 是对具体应用不可见的分布式系统的抽象层
  * 有三大类抽象：
    - 存储
    - 通信
    - 计算
  * [ 示意图：用户，应用服务器，存储服务器 ]

### 以下几个主题会反复出现

### 主题: 实现（Implementation）
  * 远程调用（RPC）, 线程（threads）, 并发控制（concurrency control）

### 主题: 性能（Performance）
  * 理想状态：可伸缩的吞吐量
    - N个服务器 -> N倍总吞吐量，N台机器并行CPU计算、磁盘IO、网络IO
    - 处理更大的负载仅需要购买更多的机器
  * 伸缩的难度会随着N增大变得越来越难:
    - 负载分配容易不平衡
    - 处理系统里不可并发的部分也变得困难，例如初始化、交互
    - 系统共享资源的瓶颈，例如网络
  * 并不是所有性能问题都可以通过分布式解决
    - 例如，减少单个用户请求的响应时间更多地需要优化代码而不是靠堆砌更多的机器

### 主题: 容错（fault tolerance）
  * 数以千计的服务器组成的复杂的网络意味着总有节点会发生异常
  * 我们不希望应用方感知到这些异常
  * 通常我们希望:
    - 可用性 -- 无论失败是否发生应用都能取得进展
    - 持久性 -- 当失败被修复时应用能恢复工作
  * 大体思路：复制集
    - 如果一个服务器宕机，应用还可以切换到其他复制集继续工作

### 主题: 一致性
  * 通用性架构的行为定义应该合理
    - 例如，“Get(k)应该返回最近一次Put(k,v)的值”
  * 实现合理的行为并不容易！
    - 很难确保每个复制集都一模一样
    - 应用方很可能在多步更新操作中的任何一步失败
    - 服务器可能在很尴尬的时候宕机，例如，执行完成但还没返回结果的时候
    - 网络不畅可能会让你误以为正常运作的机器宕机了，“脑分裂”的风险
  * 一致性和性能是互相矛盾的
    - 一致性需要通过通信来实现，例如，获得最近一次Put()的结果
    - 强一致性通常意味着系统响应很迟钝
    - 高性能通常意味着弱一致性
  * 人们在这个领域已经总结出很多设计要点

## 案例分析: MapReduce

### 让我们以MapReduce(MR)为学习案例，MR是6.824的一个很好的阐述，也是实验1的重点

### MapReduce概况
  * 场景：涉及数以GB计的数据耗时数以小时计的计算任务
    - 例如，分析爬虫获得的网页的图结构
    - 只适用于1000+服务器节点
    - 通常不是分布式系统专家设计的
    - 分布式的开发室极其痛苦的，例如，处理失败
  * 总体目标：让非专家级的程序员也可以轻易地将大量数据分发到多台服务器高效地处理
  * 程序员定义Map和Reduce函数，顺序的代码，一般比较简单
  * MR将函数运行在1000+机器节点，处理大量输入并隐藏分布式的细节

### MapReduce的抽象视图
  * 输入被分为M个文件
  * [ 示意图: maps函数生成键值对形式的行数据， reduces函数消费列数据 ]
        
        Input1 -> Map -> a,1 b,1 c,1
        Input2 -> Map ->     b,1
        Input3 -> Map -> a,1     c,1
                          |   |   |
                          |   |   -> Reduce -> c,2
                          |   -----> Reduce -> b,2
                          ---------> Reduce -> a,2
  
  * MR为每个输入文件调用Map()函数，生成中间数据集<k2,v2>，每次Map()函数调用都是一个“任务”
  * MR收集中间数据集里某个给定k2对应的所有v2，并把他们作为参数传递给Reduce函数
  * 最后的输出是Reduce()函数产生的数据集<k3,v3>，保存在R个输出文件里
  * [ 示意图: MapReduce API ]
        
        map(k1, v1) -> list(k2, v2)
        reduce(k2, list(v2)) -> list(k2, v3)

### 例子: 词频统计
  * 输入是成千上万个文本文件
  
        Map(k, v)
          split v into words
          for each word w
            emit(w, "1")
        Reduce(k, v)
          emit(len(v))

### MapReduce隐藏了繁琐的细节:
  * 在各个服务器上启动程序
  * 追踪各个任务的完成进度tracking which tasks are done
  * 数据流动
  * 从失败中恢复

### MapReduce拥有很好的扩展性:
  * N台服务器可以提供N倍的吞吐量。
    - 假设M，R >= N (考虑很多输入文件，很多Map函数的输出键的场景)，Map()函数可以并行运行，因为他们不需要交互。Reduce()函数也一样。
    - 唯一的交互是让数据在Map()函数和Reduce()函数间随机洗牌。
  * 所以你可以通过添加服务器来获取更大的吞吐量。
    - 相比针对每个应用场景优化效率的秉性方法，服务器要廉价得多。

### 可能限制性能的因素有哪些？
  * 我们在乎这些因素，因为这些因素正是需要优化的地方。
  * CPU? 内存? 磁盘? 网络?
  * 2004年，“网络带宽”成为了MapReduce的作者们提升性能的瓶颈
    - [ 示意图: 服务器集群, 网络交换机树状图 ]
    
          Map->Reduce随机洗牌过中所有的数据都通过网络传输
          论文里交换机根节点：100 到 200 G bit/s
          1800个服务器节点，所以每台服务器分到的带宽是 55 M bit/s
          这个带宽太小了，远远小于当时的磁盘IO (~50-100 MB/s) 和 RAM 的速度
  
  * 于是他们关注的是最小化数据在网络内的流动(今时今日数据中心的网络速度已经远远超过当时的水平)

### 更多细节 (论文中 图 1):
  * master节点：分配任务给worker节点，记住中产出的位置
  * M个Map任务，R个Reduce任务
  * 输入存储在GFS(Google File System)，每个输入文件都有3个副本
  * 所有服务器上都运行着GFS和MR workers
  * 输入任务远大于worker数量
  * master给每个worker分配一个Map任务，当该Map任务完成再分配一个新的Map任务
  * Map worker将产生的keys根据哈希值分成R个部分,存储在本地磁盘
  * 问题: 针对这个场景应该如何设计数据结构？
  * 在所有的Map任务都完成前不执行Reduce任务
  * master告诉Reduce worker获取Map worker生成的中间数据
  * Reduce workers将最终结果输出到GFS (每个Reduce任务一个输出文件)

### 有哪些针对网络带宽不足而优化的设计？
  * Map的输入是从本地GFS副本中读取的，而不是通过网络
  * 中间数据仅在网络中传输了一次
    - Map worker将中间结果输出到本地磁盘，而不是GFS中
  * 中间数据被分到对应很多key的文件中
    - 问题: 为什么不将Map函数执行过程中的输出实时通过TCP传输到Reduce worker处理？

### 如何做到负载均衡？
  * 对扩展性来说至关重要 -- 让N-1个节点等待1个节点是很低效的。
  * 但是某些任务很可能比别的任务耗时要长
  * [ 示意图: 将长度不确定的任务分配给各个节点 ]
  * 解决方案: 让任务数量远大于节点数量
    - Master将新任务分配给完成任务的worker
    - 于是没有任务能主宰完成时间
    - 于是处理任务快的节点会比慢的节点处理更多的任务，并跟慢的节点几乎同时结束

What about fault tolerance?
  I.e. what if a server crashes during a MR job?
  Hiding failures is a huge part of ease of programming!
  Q: Why not re-start the whole job from the beginning?
  MR re-runs just the failed Map()s and Reduce()s.
    MR requires them to be pure functions:
      they don't keep state across calls,
      they don't read or write files other than expected MR inputs/outputs,
      there's no hidden communication among tasks.
    So re-execution yields the same output.
  The requirement for pure functions is a major limitation of
    MR compared to other parallel programming schemes.
    But it's critical to MR's simplicity.

Details of worker crash recovery:
  * Map worker crashes:
    master sees worker no longer responds to pings
    crashed worker's intermediate Map output is lost
      but is likely needed by every Reduce task!
    master re-runs, spreads tasks over other GFS replicas of input.
    some Reduce workers may already have read failed worker's intermediate data.
      here we depend on functional and deterministic Map()!
    master need not re-run Map if Reduces have fetched all intermediate data
      though then a Reduce crash would then force re-execution of failed Map
  * Reduce worker crashes.
    finshed tasks are OK -- stored in GFS, with replicas.
    master re-starts worker's unfinished tasks on other workers.
  * Reduce worker crashes in the middle of writing its output.
    GFS has atomic rename that prevents output from being visible until complete.
    so it's safe for the master to re-run the Reduce tasks somewhere else.

Other failures/problems:
  * What if the master gives two workers the same Map() task?
    perhaps the master incorrectly thinks one worker died.
    it will tell Reduce workers about only one of them.
  * What if the master gives two workers the same Reduce() task?
    they will both try to write the same output file on GFS!
    atomic GFS rename prevents mixing; one complete file will be visible.
  * What if a single worker is very slow -- a "straggler"?
    perhaps due to flakey hardware.
    master starts a second copy of last few tasks.
  * What if a worker computes incorrect output, due to broken h/w or s/w?
    too bad! MR assumes "fail-stop" CPUs and software.
  * What if the master crashes?
    recover from check-point, or give up on job

For what applications *doesn't* MapReduce work well?
  Not everything fits the map/shuffle/reduce pattern.
  Small data, since overheads are high. E.g. not web site back-end.
  Small updates to big data, e.g. add a few documents to a big index
  Unpredictable reads (neither Map nor Reduce can choose input)
  Multiple shuffles, e.g. page-rank (can use multiple MR but not very efficient)
  More flexible systems allow these, but more complex model.

How might a real-world web company use MapReduce?
  "CatBook", a new company running a social network for cats; needs to:
  1) build a search index, so people can find other peoples' cats
  2) analyze popularity of different cats, to decide advertising value
  3) detect dogs and remove their profiles
  Can use MapReduce for all these purposes!
  - run large batch jobs over all profiles every night
  1) build inverted index: map(profile text) -> (word, cat_id)
                           reduce(word, list(cat_id) -> list(word, list(cat_id))
  2) count profile visits: map(web logs) -> (cat_id, "1")
                           reduce(cat_id, list("1")) -> list(cat_id, count)
  3) filter profiles: map(profile image) -> img analysis -> (cat_id, "dog!")
                      reduce(cat_id, list("dog!")) -> list(cat_id)

Conclusion
  MapReduce single-handedly made big cluster computation popular.
  - Not the most efficient or flexible.
  + Scales well.
  + Easy to program -- failures and data movement are hidden.
  These were good trade-offs in practice.
  We'll see some more advanced successors later in the course.
  Have fun with the lab!