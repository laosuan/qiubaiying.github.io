---
layout:     post
title:      深入浅出elasticsearch1-运维
subtitle:   elasticsearch
date:       2020-07-30
author:     laosuan
header-img: elastic.png
catalog: true
tags:
    - linux
---

##  集群健康

### 集群状态总览

```
http://10.128.10.179:9200/_cluster/health

//返回如下
{
"cluster_name": "gfask-cluster",
"status": "yellow",
"timed_out": false,
"number_of_nodes": 1,
"number_of_data_nodes": 1,
"active_primary_shards": 160,
"active_shards": 160,
"relocating_shards": 0,
"initializing_shards": 0,
"unassigned_shards": 155, //是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本。
"delayed_unassigned_shards": 0,
"number_of_pending_tasks": 0,
"number_of_in_flight_fetch": 0
}
```

响应信息中最重要的一块就是 `status` 字段。状态可能是下列三个值之一：

- **`green`**

  所有的主分片和副本分片都已分配。你的集群是 100% 可用的。

- **`yellow`**

  所有的主分片已经分片了，但至少还有一个副本是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过，你的高可用性在某种程度上被弱化。如果 *更多的* 分片消失，你就会丢数据了。把 `yellow` 想象成一个需要及时调查的警告。

- **`red`**

  至少一个主分片（以及它的全部副本）都在缺失中。这意味着你在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常。

`green`/`yellow`/`red` 状态是一个概览你的集群并了解眼下正在发生什么的好办法。剩下来的指标给你列出来集群的状态概要：

- `number_of_nodes` 和 `number_of_data_nodes` 这个命名完全是自描述的。
- `active_primary_shards` 指出你集群中的主分片数量。这是涵盖了所有索引的汇总值。
- `active_shards` 是涵盖了所有索引的_所有_分片的汇总值，即包括副本分片。
- `relocating_shards` 显示当前正在从一个节点迁往其他节点的分片的数量。通常来说应该是 0，不过在 Elasticsearch 发现集群不太均衡时，该值会上涨。比如说：添加了一个新节点，或者下线了一个节点。
- `initializing_shards` 是刚刚创建的分片的个数。比如，当你刚创建第一个索引，分片都会短暂的处于 `initializing` 状态。这通常会是一个临时事件，分片不应该长期停留在 `initializing` 状态。你还可能在节点刚重启的时候看到 `initializing` 分片：当分片从磁盘上加载后，它们会从 `initializing` 状态开始。
- `unassigned_shards` 是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本。比如，一个有 5 分片和 1 副本的索引，在单节点集群上，就会有 5 个未分配副本分片。如果你的集群是 `red` 状态，也会长期保有未分配分片（因为缺少主分片）。



### 集群索引状态清单（用于定位具体故障索引）

```
http://10.128.10.179:9200/_cluster/health?level=indices
```

### 每个索引里每个分片的状态和位置

```
http://10.128.10.179:9200/_cluster/health?level=shards
```



##  监控单个节点

```
http://10.128.10.179:9200/_nodes/stats
```

此接口返回包含集群的每一个节点统计值，用来评估节点的性能瓶颈。

### 索引部分

索引(indices) 部分列出了这个节点上所有索引的聚合过的统计值：

- `docs`展示节点内存有多少文档，包括还没有从段里清除的已删除文档数量。
- `store` 部分显示节点耗用了多少物理存储。这个指标包括主分片和副本分片在内。如果限流时间很大，那可能表明你的磁盘限流设置得过低（在[段和合并](https://www.elastic.co/guide/cn/elasticsearch/guide/current/indexing-performance.html#segments-and-merging)里讨论过）。

```json
// 生产环境
indices":{
    "docs":{
      "count":1066173,
      "deleted":53914
    },
      "store":{
        "size_in_bytes":3429169540,
        "throttle_time_in_millis":792608
      },
// 测试环境 
"indices": {
    "docs": {
        "count": 62208,
        "deleted": 9415
    },
    "store": {
        "size_in_bytes": 154985290,
        "throttle_time_in_millis": 43044
    },
```

---

返回的统计值被归入以下部分：

- `get` 显示通过 ID 获取文档的接口相关的统计值。包括对单个文档的 `GET` 和 `HEAD` 请求。

- `search` 描述在活跃中的搜索（ `open_contexts` ）数量、查询的总数量、以及自节点启动以来在查询上消耗的总时间。用 `query_time_in_millis / query_total` 计算的比值，可以用来粗略的评价你的查询有多高效。比值越大，每个查询花费的时间越多，你应该要考虑调优了。

  fetch 统计值展示了查询处理的后一半流程（query-then-fetch 里的 *fetch* ）。如果 fetch 耗时比 query 还多，说明磁盘较慢，或者获取了太多文档，或者可能搜索请求设置了太大的分页（比如， `size: 10000` ）。

- `merges` 包括了 Lucene 段合并相关的信息。它会告诉你目前在运行几个合并，合并涉及的文档数量，正在合并的段的总大小，以及在合并操作上消耗的总时间。

  在你的集群写入压力很大时，合并统计值非常重要。合并要消耗大量的磁盘 I/O 和 CPU 资源。如果你的索引有大量的写入，同时又发现大量的合并数，一定要去阅读[索引性能技巧](https://www.elastic.co/guide/cn/elasticsearch/guide/current/indexing-performance.html)。

  注意：文档更新和删除也会导致大量的合并数，因为它们会产生最终需要被合并的段 *碎片* 。

```json
// 生产环境
 "search":{
   "open_contexts":0,
   "query_total":3601296,
   "query_time_in_millis":200795495,
   "query_current":0,
   "fetch_total":2845835,
   "fetch_time_in_millis":154850468,
   "fetch_current":0
 },
"merges":{
  "current":0,
  "current_docs":0,
  "current_size_in_bytes":0,
  "total":19188,
  "total_time_in_millis":2538745,
  "total_docs":14425820,
  "total_size_in_bytes":22678293953
},
// 测试环境 
"search": {
  "open_contexts": 0,
  "query_total": 265990,
  "query_time_in_millis": 1614815,
  "query_current": 0,
  "fetch_total": 148167,
  "fetch_time_in_millis": 2289133,
  "fetch_current": 0
},
"merges": {
  "current": 0,
  "current_docs": 0,
  "current_size_in_bytes": 0,
  "total": 877,
  "total_time_in_millis": 293886,
  "total_docs": 1974785,
  "total_size_in_bytes": 2398768002
},

```

在你的集群写入压力很大时，合并统计值非常重要。合并要消耗大量的磁盘 I/O 和 CPU 资源。如果你的索引有大量的写入，同时又发现大量的合并数，一定要去阅读[索引性能技巧](https://www.elastic.co/guide/cn/elasticsearch/guide/current/indexing-performance.html)。

注意：文档更新和删除也会导致大量的合并数，因为它们会产生最终需要被合并的段 *碎片* 。

---

`segments` 会展示这个节点目前正在服务中的 Lucene 段的数量。这是一个重要的数字。大多数索引会有大概 50–150 个段，哪怕它们存有 TB 级别的数十亿条文档。段数量过大表明合并出现了问题（比如，合并速度跟不上段的创建）。注意这个统计值是节点上所有索引的汇聚总数。记住这点。

`memory` 统计值展示了 Lucene 段自己用掉的内存大小。这里包括底层数据结构，比如倒排表，字典，和布隆过滤器等。太大的段数量会增加这些数据结构带来的开销，这个内存使用量就是一个方便用来衡量开销的度量值。

```json
 // 生产环境
 "segments":{
   "count":68,
   "memory_in_bytes":9045304,
   "index_writer_memory_in_bytes":0,
   "index_writer_max_memory_in_bytes":92160000,
   "version_map_memory_in_bytes":0,
   "fixed_bit_set_memory_in_bytes":0
 },
// 测试环境 
"segments": {
  "count": 35,
  "memory_in_bytes": 1237230,
  "index_writer_memory_in_bytes": 0,
  "index_writer_max_memory_in_bytes": 81920000,
  "version_map_memory_in_bytes": 0,
  "fixed_bit_set_memory_in_bytes": 0
}
```



### JVM 部分

`jvm` 部分包括了运行 Elasticsearch 的 JVM 进程一些很关键的信息。最重要的，它包括了垃圾回收的细节，这对你的 Elasticsearch 集群的稳定性有着重大影响。

> **垃圾回收入门**
>
> 在我们描述统计值之前，先上一门速成课程讲解垃圾回收以及它对 Elasticsearch 的影响是非常有用的。如果你对 JVM 的垃圾回收很熟悉，请跳过这段。
>
> Java 是一门 *垃圾回收* 语言，也就是说程序员不用手动管理内存分配和回收。程序员只管写代码，然后 Java 虚拟机（JVM）按需分配内存，然后在稍后不再需要的时候清理这部分内存。
>
> 当内存分配给一个 JVM 进程，它是分配到一个大块里，这个块叫做 *堆* 。JVM 把堆分成两组，用 *代* 来表示：
>
> - **新生代（或者伊甸园）**
>
>   新实例化的对象分配的空间。新生代空间通常都非常小，一般在 100 MB–500 MB。新生代也包含两个 *幸存* 空间。
>
> - **老生代**
>
>   较老的对象存储的空间。这些对象预计将长期留存并持续上很长一段时间。老生代通常比新生代大很多。Elasticsearch 节点可以给老生代用到 30 GB。
>
> 当一个对象实例化的时候，它被放在新生代里。当新生代空间满了，就会发生一次新生代垃圾回收（GC）。依然是"存活"状态的对象就被转移到一个幸存区内，而"死掉"的对象被移除。如果一个对象在多次新生代 GC 中都幸存了，它就会被"终身"置于老生代了。
>
> 类似的过程在老生代里同样发生：空间满的时候，发生一次垃圾回收，死掉的对象被移除。
>
> 不过，天下没有免费的午餐。新生代、老生代的垃圾回收都有一个阶段会“停止时间”。在这段时间里，JVM 字面意义上的停止了程序运行，以便跟踪对象图，收集死亡对象。在这个时间停止阶段，一切都不会发生。请求不被服务，ping 不被回应，分片不被分配。整个世界都真的停止了。
>
> 对于新生代，这不是什么大问题；那么小的空间意味着 GC 会很快执行完。但是老生代大很多，而这里面一个慢 GC 可能就意味着 1 秒乃至 15 秒的暂停——对于服务器软件来说这是不可接受的。
>
> JVM 的垃圾回收采用了 *非常* 精密的算法，在减少暂停方面做得很棒。而且 Elasticsearch 非常努力的变成对 *垃圾回收友好* 的程序，比如内部智能的重用对象，重用网络缓冲，以及默认启用 [Doc Values](https://www.elastic.co/guide/cn/elasticsearch/guide/current/docvalues.html) 功能。但最终，GC 的频率和时长依然是你需要去观察的指标。因为它是集群不稳定的头号嫌疑人。
>
> 一个经常发生长 GC 的集群就会因为内存不足而处于高负载压力下。这些长 GC 会导致节点短时间内从集群里掉线。这种不稳定会导致分片频繁重定位，因为 Elasticsearch 会尝试保持集群均衡，保证有足够的副本在线。这接着就导致网络流量和磁盘 I/O 的增加。而所有这些都是在你的集群努力服务于正常的索引和查询的同时发生的。
>
> 总而言之，长时间 GC 总是不好的，需要尽可能的减少。

---

因为垃圾回收对 Elasticsearch 是如此重要，你应该非常熟悉 `node-stats` API 里的这部分内容：

`jvm` 部分首先列出一些和 heap 内存使用有关的常见统计值。你可以看到有多少 heap 被使用了，多少被指派了（当前被分配给进程的），以及 heap 被允许分配的最大值。理想情况下，`heap_committed_in_bytes` 应该等于 `heap_max_in_bytes` 。如果指派的大小更小，JVM 最终会被迫调整 heap 大小——这是一个非常昂贵的操作。如果你的数字不相等，阅读 [堆内存:大小和交换](https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html) 学习如何正确的配置它。

`heap_used_percent` 指标是值得关注的一个数字。Elasticsearch 被配置为当 heap 达到 75% 的时候开始 GC。如果你的节点一直 >= 75%，你的节点正处于 *内存压力* 状态。这是个危险信号，不远的未来可能就有慢 GC 要出现了。

如果 heap 使用率一直 >=85%，你就麻烦了。Heap 在 90–95% 之间，则面临可怕的性能风险，此时最好的情况是长达 10–30s 的 GC，最差的情况就是内存溢出（OOM）异常。

GC 花费的时间也很重要。比如，在索引文档时，一系列垃圾生成了。这是很常见的情况，每时每刻都会导致 GC。这些 GC 绝大多数时候都很快，对节点影响很小：新生代一般就花一两毫秒，老生代花一百多毫秒。这些跟 10 秒级别的 GC 是很不一样的。

我们的最佳建议是定期收集 GC 计数和时长（或者使用 Marvel）然后观察 GC 频率。你也可以开启慢 GC 日志记录，在 [日志记录](https://www.elastic.co/guide/cn/elasticsearch/guide/current/logging.html) 小节已经讨论过。

// to do -- install marvel观察GC时长 https://www.elastic.co/cn/downloads/marvel

```json
//生产环境
 "jvm":{
   "timestamp":1596074408763,
   "uptime_in_millis":42632257232,
   "mem":{
     "heap_used_in_bytes":511948848,
     "heap_used_percent":49,
     "heap_committed_in_bytes":1038876672,
     "heap_max_in_bytes":1038876672,
     "non_heap_used_in_bytes":112445176,
     "non_heap_committed_in_bytes":114618368,
    
   "gc":{
     "collectors":{
       "young":{
         "collection_count":1496995,
         "collection_time_in_millis":6607399
       },
     "old":{
       "collection_count":532,
       "collection_time_in_millis":24089
    	 }

//测试环境
"jvm": {
  "timestamp": 1596162121893,
  "uptime_in_millis": 35233016120,
  "mem": {
    "heap_used_in_bytes": 436156360,
    "heap_used_percent": 41,
    "heap_committed_in_bytes": 1038876672,
    "heap_max_in_bytes": 1038876672,
    "non_heap_used_in_bytes": 104950336,
    "non_heap_committed_in_bytes": 107356160,

  }
  "gc": {
    "collectors": {
    "young": {
      "collection_count": 45333,
      "collection_time_in_millis": 363118
    },
    "old": {
      "collection_count": 248,
      "collection_time_in_millis": 21963
    }
    }
  },
},


```







refference:

- https://www.elastic.co/guide/cn/elasticsearch/guide/current/_monitoring_individual_nodes.html

- https://www.elastic.co/guide/en/elasticsearch/guide/current/_monitoring_individual_nodes.html

- [为什么要看ES源码]( http://www.54tianzhisheng.cn/2018/08/04/why-see-es-code/)

- [https://laijianfeng.org/2018/08/%E6%95%99%E4%BD%A0%E7%BC%96%E8%AF%91%E8%B0%83%E8%AF%95Elasticsearch-6-3-2%E6%BA%90%E7%A0%81/](https://laijianfeng.org/2018/08/教你编译调试Elasticsearch-6-3-2源码/)

