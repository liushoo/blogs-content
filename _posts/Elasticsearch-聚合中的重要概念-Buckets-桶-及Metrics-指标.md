---
title: '[Elasticsearch] 聚合中的重要概念 - Buckets(桶)及Metrics(指标)'
date: 2015-01-04 10:03:00
categories: [译文, Elasticsearch]
tags: [Elasticsearch, 聚合]
---

本章翻译自Elasticsearch官方指南的[Aggregations-High-level Concepts](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/aggs-high-level.html)一章。

# 高层概念(High-Level Concepts)

和查询DSL一样，聚合(Aggregations)也拥有一种可组合(Composable)的语法：独立的功能单元可以被混合在一起来满足你的需求。这意味着需要学习的基本概念虽然不多，但是它们的组合方式是几近无穷的。

为了掌握聚合，你只需要了解两个主要概念：

**Buckets(桶)：**

满足某个条件的文档集合。

**Metrics(指标)：**

为某个桶中的文档计算得到的统计信息。

<!-- More -->

就是这样！每个聚合只是简单地由一个或者多个桶，零个或者多个指标组合而成。可以将它粗略地转换为SQL：

```sql
SELECT COUNT(color) 
FROM table
GROUP BY color
```

以上的COUNT(color)就相当于一个指标。GROUP BY color则相当于一个桶。

桶和SQL中的组(Grouping)拥有相似的概念，而指标则与COUNT()，SUM()，MAX()等相似。

让我们仔细看看这些概念。

## 桶(Buckets)

一个桶就是满足特定条件的一个文档集合：

- 一名员工要么属于男性桶，或者女性桶。
- 城市Albany属于New York州这个桶。
- 日期2014-10-28属于十月份这个桶。

随着聚合被执行，每份文档中的值会被计算来决定它们是否匹配了桶的条件。如果匹配成功，那么该文档会被置入该桶中，同时聚合会继续执行。

桶也能够嵌套在其它桶中，能让你完成层次或者条件划分这些需求。比如，Cincinnati可以被放置在Ohio州这个桶中，而整个Ohio州则能够被放置在美国这个桶中。

ES中有很多类型的桶，让你可以将文档通过多种方式进行划分(按小时，按最流行的词条，按年龄区间，按地理位置，以及更多)。但是从根本上，它们都根据相同的原理运作：按照条件对文档进行划分。

## 指标(Metrics)

桶能够让我们对文档进行有意义的划分，但是最终我们还是需要对每个桶中的文档进行某种指标计算。分桶是达到最终目的的手段：提供了对文档进行划分的方法，从而让你能够计算需要的指标。

多数指标仅仅是简单的数学运算(比如，min，mean，max以及sum)，它们使用文档中的值进行计算。在实际应用中，指标能够让你计算例如平均薪资，最高出售价格，或者百分之95的查询延迟。

## 将两者结合起来

一个聚合就是一些桶和指标的组合。一个聚合可以只有一个桶，或者一个指标，或者每样一个。在桶中甚至可以有多个嵌套的桶。比如，我们可以将文档按照其所属国家进行分桶，然后对每个桶计算其平均薪资(一个指标)。

因为桶是可以嵌套的，我们能够实现一个更加复杂的聚合操作：

1. 将文档按照国家进行分桶。(桶)
2. 然后将每个国家的桶再按照性别分桶。(桶)
3. 然后将每个性别的桶按照年龄区间进行分桶。(桶)
4. 最后，为每个年龄区间计算平均薪资。(指标)

此时，就能够得到每个<国家，性别，年龄>组合的平均薪资信息了。它可以通过一个请求，一次数据遍历来完成！


