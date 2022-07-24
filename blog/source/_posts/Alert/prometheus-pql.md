---
title: Prometheus | PromQL
date: 2022-07-05 20:16:49
categories: 
    - 监控告警
tags: 
    - Prometheus
    - PromQL
---

## 0. 什么是PromQL
> PromQL（Prometheus Query Language）是 Prometheus 内置的数据查询语言，它能实现对事件序列数据的查询、聚合、逻辑运算等。它并且被广泛应用在 Prometheus 的日常应用当中，包括对数据查询、可视化、告警处理当中。简单地说，PromQL 广泛存在于以 Prometheus 为核心的监控体系中。所以需要用到数据筛选的地方，就会用到 PromQL。例如：监控指标的设置、报警指标的设置等等。

## 1. 基础用法
当我们直接使用监控指标名称查询时，可以查询该指标下的所有时间序列。
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220705202637.png)
可以看到我们查询出了所有指标名称为 `prometheus_http_requests_total` 的数据。
PromQL 支持户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。

### 1.1 完全匹配
PromQL 支持使用 = 和 != 两种完全匹配模式。
* 等于。通过使用 `label=value` 可以选择那些标签满足表达式定义的时间序列。
* 不等于。通过使用 `label!=value` 则可以根据标签匹配排除时间序列。

例如我们上面查询出了所有指标名称为 `prometheus_http_requests_total` 的数据。这时候我们希望只查看错误的请求，即过滤掉所有 code 标签不是 200 的数据。

那么我们的 PromQL 表达式可以修改为：
```
prometheus_http_requests_total{code!="200"}
```

### 1.2 正则匹配
PromQL 还可以使用正则表达式作为匹配条件，并且可以使用多个匹配条件。
* 正向匹配。使用 `label=~regx` 表示选择那些标签符合正则表达式定义的时间序列。
* 反向匹配。使用 `label!~regx` 进行排除。

例如我想查询指标 `prometheus_http_requests_total` 中，所有 handler 标签以 /api/v1 开头的记录。

那么我的表达式为：
```
prometheus_http_requests_total{handler=~"/api/v1/.*"}
```

### 1.3 范围查询
我们上面直接通过类似 `prometheus_http_requests_total` 表达式查询时间序列时，同一个指标同一标签只会返回一条数据。这样的表达式我们称之为`瞬间向量表达式`，返回的结果称之为`瞬间向量`

而如果我们想查询一段时间范围内的样本数据，那么我们就需要用到`区间向量表达式`，其查询出来的结果称之为`区间向量`。

时间范围通过时间范围选择器 `[]` 进行定义。例如，通过以下表达式可以选择最近5分钟内的所有样本数据：
```
prometheus_http_requests_total{}[5m]
```
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220705204103.png)
通过查询结果可以看到，此时我们查询出了所有的样本数据，而不再是一个样本数据的统计值。

PromQL的时间范围选择器支持其它时间单位: 
* s - 秒
* m - 分
* h - 时
* d - 天
* w - 周
* y - 年

### 1.4 时间偏移
在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准。如果我们想查询 5 分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为 `offset`
```
# 查询 5 分钟前的最新数据
http_request_total{} offset 5m

# 往前移动 1 天，查询 1 天前的数据
http_request_total{}[1d] offset 1d
```

### 1.5 聚合查询
一般情况下，我们通过 PromQL 查询到的数据都是很多的。PromQL 提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列。

以我们的 `prometheus_http_requests_total` 指标为例，不加任何条件我们查询到的数据为：
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220705204914.png)

* 第一个表达式，计算一共有几条数据: 
```
count(prometheus_http_requests_total)
# 查询结果为8，代表总共有8条数据
```

* 第二个表达式，计算所有数据的 value 总和:
```
sum(prometheus_http_requests_total)
# 查询结果为307，代表所有数据的value之和为307
```

## 2. 常用函数
scalar(v instance-vector)是我们第一个见过的函数， 他将瞬时向量转化成标量。
> 标量可以用于运算和可视化。例如和瞬时向量可以和标量进行加减乘除运算。只有一个时间序列的瞬时变量可以通过函数scalar()转化为标量。有时候在监控中，我们只需要知道当前的系统是不是健康的，当前磁盘空间是多少，只是后就可以用scalar()得到标量并在可视化工具中展示。

abs(v instant-vector)可以将瞬时变量中的值转变为绝对值。

increase(v range-vector) 求区间向量的增长量（最新值减最旧值）并返回一个瞬时向量。