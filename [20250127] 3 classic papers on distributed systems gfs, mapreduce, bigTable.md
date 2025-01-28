# [20250127] 3 classic papers on distributed systems: *GFS*, *MapReduce*, *BigTable*

[TOC]

## Introduction

These three official Google papers mark the beginning of the big data era, GFS, MapReduce, BigTable:

- ***The Google File System***: 提出了 Google 文件系统架构，它是面向分布式数据密集型应用的、可伸缩的分布式文件系统。打破了传统文件系统的设计思路，针对大规模数据存储和处理需求，采用了中心服务器（Master）和数据块服务器（Chunk Server）的架构，将数据切分成固定大小的数据块存储在多个数据块服务器上，实现了数据的分布式存储和管理，为大规模数据存储提供了新的架构模式。
  - 影响：为大规模数据存储和处理提供了新的思路和解决方案，启发了后续 Hadoop 分布式文件系统 HDFS 等的设计。

- ***MapReduce: Simplified Data Processing on Large Clusters***: 提出了一种全新的分布式计算模型 MapReduce，将数据处理过程抽象为 Map 和 Reduce 两个简单的操作，用户只需实现这两个函数，就能轻松进行大规模数据的并行处理，这种简化的编程模型极大地降低了分布式计算的门槛。
  - 影响：极大地降低了大规模数据处理的难度，推动了大数据处理技术的发展，在数据挖掘、机器学习和商业智能等众多领域得到广泛应用，Hadoop MapReduce 就是受其启发而产生的开源实现

- ***Bigtable: A Distributed Storage System for Structured Data***: 提出了 BigTable 这种分布式结构化数据存储系统，它是一种稀疏的、分布式的、持久化存储的多维度排序映射表，能够高效存储和管理大规模的结构化数据，为结构化数据的分布式存储提供了新的解决方案。
  - 影响：为大规模结构化数据的存储和管理提供了有效的解决方案，对后来的分布式数据库如 HBase 等的发展产生了深远影响。


## The Google File System

### Background

- **Component failures are the norm**. The system is made up of a large number of cheap components and failures occur frequently. So it needs to have constant monitoring, error detection, fault tolerance, and automatic recovery.
- **Files are huge**. Traditional file systems’ design parameters such as I/O operations and block size no longer apply.
- **Files are mutated by appending new data**. Most files are for appending new data, with very little random writing. Caching data blocks is not very meaningful, so appending becomes the focus of performance optimization and atomicity guarantees
- **Co-designing the applications and the file system API**. Such as relaxing the consistency model and introducing atomic append operations.

### Assumes

System is build in the cheap components 

系统构建在易故障的廉价组件之上，需处理大量大文件，工作负载包含特定读写模式，要高效实现多客户端并发追加语义，高带宽比低延迟更重要。

### Interface

The system provide a familiar file system interface, and has snapshot and record append operation.

- Usual operations: create & delete, open & close, read & write
- **Snapshot**: creates a copy of a file or directory tree at low cast.
- **Record append**: guaranteeing the atomicity of each individual client's append.

### Architecture

The file system consists of a single master, multiple chunk-servers and multiple client. Files are divided into fixed-size chunks, and 

- A single **master**: Master maintains all file system metadata. Each chunk is identified by 64 bit chunk handle assigned by the master at the time of chunk creation. Master also controls system-wide activities.
- multiple **chunk-servers**: Chunk-servers store chunks on local disks as Linux files and read or write chunk data specified by a chunk handle and byte range. Each chunk is replicated on multiple chunk-servers. 
- multiple **client**: 

由一个主服务器、多个块服务器和多个客户端组成。文件被分割为固定大小的块存储在块服务器上，默认多副本存储；主服务器管理元数据；客户端直接与块服务器进行数据交互。

#### Single Master

单主服务器优势与操作流程: 单主服务器简化设计，能进行复杂的块放置和复制决策。客户端通过与主服务器交互获取块位置信息，缓存后直接与块服务器操作，减少主服务器参与。

#### Chunk Size

选择 64MB 的块大小，可减少客户端与主服务器交互、降低网络开销、缩小主服务器元数据存储量，但可能导致小文件热点问题。

#### Metadata

主服务器存储多种元数据，部分元数据通过操作日志持久化，采用内存存储元数据提高操作速度。

#### Consistency Model

采用宽松一致性模型，文件命名空间突变原子性，数据突变后文件区域状态依突变类型等而定，应用可通过特定技术适应。

### System Interaction

#### 租约与突变顺序

通过租约保证数据突变顺序，主服务器授予主副本租约，主副本为突变操作排序，所有副本按序执行。

#### 数据传输优化

数据传输与控制流分离，数据沿精心挑选的块服务器链线性推送，以充分利用网络带宽、避免瓶颈和降低延迟。

#### 原子记录追加

支持多客户端原子追加操作，应用广泛。主副本检查追加记录是否会使块超大小，决定操作方式，失败时客户端重试。

#### 快照实现

利用写时复制技术，主服务器在收到快照请求后，撤销相关块的租约，记录操作，复制元数据，新写操作触发块复制。

### 主服务器操作

#### 命名空间管理与锁机制

通过锁机制保证命名空间操作的序列化，支持并发突变，锁对象按需分配和删除，按特定顺序获取以防止死锁。

#### 副本放置策略

旨在最大化数据可靠性、可用性和网络带宽利用率，将副本分散在不同机架的服务器上。

#### 块的创建、重新复制与平衡

主服务器根据多种因素创建、重新复制和平衡块副本，确保系统稳定运行，优化资源利用。

#### 垃圾回收机制

文件和块的删除采用延迟垃圾回收，简化系统，提高可靠性，但可能影响存储资源的及时复用。

#### 陈旧副本检测

主服务器通过块版本号检测陈旧副本，确保客户端和块服务器访问最新数据，陈旧副本在垃圾回收时删除。

### 容错与诊断

#### 高可用性保障

通过快速恢复、块复制和主服务器复制，确保系统在组件故障时仍能保持高可用性。

#### 数据完整性保护

块服务器使用校验和检测数据损坏，读取时验证，损坏时从其他副本恢复，写操作时优化校验和计算。

#### 诊断工具助力

详细的诊断日志用于问题隔离、调试和性能分析，记录系统关键事件和 RPC 请求响应，对性能影响小。

### 性能测量

#### 微基准测试

在特定配置的集群上进行读写和记录追加测试，分析性能瓶颈，如网络栈影响写入速度、多客户端并发操作影响效率等。

#### 真实集群分析

研究两个真实集群的存储、元数据、读写速率、主服务器负载和恢复时间等指标，展示 GFS 在实际应用中的表现。

#### 工作负载剖析

分析两个集群的工作负载，发现读写操作大小呈双峰分布，记录追加操作使用频繁，生产系统对其依赖度高。

### 实践经验

#### 应用拓展与问题

GFS 从生产系统后端文件系统拓展到研发领域，在使用中遇到磁盘与 Linux 相关问题，如协议版本不兼容、fsync () 成本高、锁机制问题等。

#### 解决措施与收获

通过修改内核、调整使用方式等解决问题，同时得益于 Linux 代码的开源，便于深入理解和优化系统。

### 相关工作对比

与 AFS、xFS 等分布式文件系统相比，GFS 在数据分布、冗余方式、缓存机制、服务器架构等方面存在差异，各有优劣。

### 结论

GFS 成功满足 Google 的存储需求，在大规模数据处理方面表现卓越。其设计理念对类似数据处理任务具有重要参考价值，未来有望通过改进网络栈提升写入性能。

## MapReduce: Simplified Data Processing on Large Clusters

### 编程模型

MapReduce 的编程模型是一种用于处理和生成大规模数据集的计算模型，计算过程基于输入的键值对集合，产生一组输出键值对。用户只需编写 map 和 reduce 两个函数这两个函数来表达计算逻辑就能实现复杂的数据处理任务. 由 MapReduce 库负责处理并行化、数据分布、负载均衡 、故障处理和通信调度以及容错等复杂操作.

- **Map**: 将获取到的数据集进一步解析成<key,value>,通过Map函数计算生成中间结果，进过shuffle处理后作为reduce的输入. 把一组键值对映射成一组新的键值对，对输入数据进行处理和转换，例如将文本中的单词提取出来作为键，出现次数作为值。
- **Reduce**: reduce得到map输出的中间结果，合并计算将最终结果输出HDFS，其中List(v2)，指同-k2的value. 对 Map 阶段输出的具有相同键的键值对进行合并和处理，得到最终结果，如对单词的出现次数进行统计求和。

### 系统组成

- **JobTracker**：MapReduce 框架的主节点，负责管理和监控整个任务的执行过程，包括分配任务、监控任务执行情况、处理任务失败和重试等。
- **TaskTracker**：工作节点，接收 JobTracker 分配的任务，执行 Map 或 Reduce 任务，并向 JobTracker 汇报任务执行情况。
- **Mapper**：负责将输入数据映射为键值对，接收 JobTracker 分配的数据块，对每个数据块进行处理并输出键值对。
- **Reducer**：负责将 Mapper 输出的键值对按照键进行合并和处理，接收 JobTracker 分配的 Mapper 输出的键值对，对相同键的值进行合并处理，输出最终结果。
- **Combiner**：可选组件，用于在 Mapper 和 Reducer 之间进行局部合并处理，减少数据传输量，提高处理效率。
- **InputFormat 和 OutputFormat**：分别是输入格式组件和输出格式组件，负责将输入数据格式化为 MapReduce 框架可处理的格式，以及将输出结果格式化为指定格式。

## Bigtable: A Distributed Storage System for Structured Data

- **超大规模可扩展性**：能够处理数千台商用服务器上的 PB 级数据，可根据业务需求灵活扩展或收缩，轻松应对数据量的爆发式增长。
- **数据模型灵活**：是一个稀疏、分布式、持久的多维有序映射表。数据通过行键、列键和时间戳进行索引，每条数据都有唯一的行键，类似数据库主键，支持通过行键随机读写。列属于列族，无需预先定义具体列，每行数据可拥有不同列集，体现 “稀疏” 特性。还支持在列下存储多个版本数据，可按需求保留特定版本，方便数据追踪与恢复。
- **高性能与低延迟**：为不同延迟要求的应用，如从后台批量处理到实时数据服务，都能提供高性能解决方案，可快速响应用户请求，确保数据的高效读写。
- **高可用性和可靠性**：即使部分节点故障，也能保障整个系统稳定运行，通过数据冗余、故障检测与自动恢复机制等，确保数据不丢失、服务不中断。

### 架构与组件

- **客户端库**：为应用程序提供与 Bigtable 交互的接口，负责缓存元数据和数据位置信息等，以减少与服务器的交互次数，提高访问效率。
- **主服务器**：负责管理元数据，如分配数据分片到各个 tablet 服务器，监控 tablet 服务器状态，处理服务器的添加和删除，以及执行垃圾回收等任务。
- **Tablet 服务器**：实际存储和处理数据的节点，管理一组数据分片（Tablet），处理客户端的读写请求，当数据量增长超过一定阈值时，负责对 Tablet 进行拆分。
