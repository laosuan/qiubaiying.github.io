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



refference:

- https://www.elastic.co/guide/cn/elasticsearch/guide/current/_monitoring_individual_nodes.html

- https://www.elastic.co/guide/en/elasticsearch/guide/current/_monitoring_individual_nodes.html

- [为什么要看ES源码]( http://www.54tianzhisheng.cn/2018/08/04/why-see-es-code/)

- [https://laijianfeng.org/2018/08/%E6%95%99%E4%BD%A0%E7%BC%96%E8%AF%91%E8%B0%83%E8%AF%95Elasticsearch-6-3-2%E6%BA%90%E7%A0%81/](https://laijianfeng.org/2018/08/教你编译调试Elasticsearch-6-3-2源码/)

