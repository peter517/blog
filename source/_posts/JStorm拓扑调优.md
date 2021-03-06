title: JStorm拓扑调优
tags: [Java,JStorm]
date: 2017-04-25 15:06:04
description: 如何确定spout和bolt的个数及上下游关系
---

写JStorm代码业务逻辑比较容易，关键是设计一个好的拓扑关系，能保证数据正常流动又不占用太多机器资源

# 评估spout和bolt的个数
## 日志增速
先查看整个拓扑输入源的日志增速，比如20G/min，特别注意处理高峰时期的日志增速，由于metric是以min为单位，后面的统计统一以min为计算单元
## spout读取能力
计算一个spout读取日志的能力，可以通过metrics来count每分钟读取数据大小，用日志每分钟增量除以spout读取速度，这样就可以计算出总共需要多少个spout，比如一个spout可以读取速度为300M/min,那边20G/min的日志就需要8个spout来处理（多预留一些空间）
## spout发送数据条数
spout发送数据量关系到下游的bolt需要多少个，可以通过查看emit的数量来估算，这里一般会比日志数量少，因为业务逻辑会过滤掉部分数据

## bolt处理能力
bolt处理能力可以通过执行一次bolt的execute函数的耗时来技术，比如0.5ms平均一次execute，这个bolt的处理能力是60 * 1000/0.5 = 120000条/min，如果上流数据是1200000条/min，那么需要15个bolt来处理数据

## bolt发送数据量
bolt发送数据量关系到下游的bolt需要多少个，可以通过查看emit的数量来估算，这里一般会比进来的数量少，业务逻辑会过滤掉部分数据

# 优化spout和bolt吞吐量
spout和bolt吞吐量表示处理数据的能力，IO操作对吞吐量影响很大，尽量在IO时进行批量IO

# 上下游关系
日志处理一般通过shuffle来向下游发送数据，可以把数据平衡发到每个节点

# Metrics数据监控
通过Metrics来统计count和time cost很有必要，方便问题定位和后续监控

# 窗口聚合
如果需要处理100条日志或者1分钟的日志，需要引入窗口的概念，聚合一个窗口的数据进行处理

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
