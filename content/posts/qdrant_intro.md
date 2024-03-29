+++
title = "Qdrant 浅析"
date = 2023-09-29T20:26:00+08:00
tags = ["database", "Vector Database"]
draft = false
author = "lyraile"
+++

随着 AI 的火热发展，传统数据库或搜索引擎检索高维度信息时性能表现均不佳，伴随着这个问题向量数据库又焕发了第二春。 <br/>

本文介绍的 [Qdrant](https://qdrant.tech/) 是基于 Rust 所编写的一款向量数据库，在它的 [benchmarks](https://qdrant.tech/benchmarks/) 我们可以看到对比起 Elasticsearch 具有明显的性能优势。 <br/>

本文仅介绍 qdrant 相关概念，以及内部存储结构。涉及到 embedding 等相关概念请自行搜索 <br/>


## 模型 {#模型}


### Points {#points}

point 可以理解为关系型数据库中表记录中的一条，它由一个向量和一个可为空的 `Payload` 组成。伪代码如下： <br/>

{{< highlight rust >}}
enum PointIdType {
    U64(u64),
    UUID(uuid),
}

struct Point {
    id: PointIdType,
    vector: Vec<f32>,
    payload: Option<Payload>,
}  
{{< /highlight >}}


### Payload {#payload}

Qdrant的一个重要特性之一是能够在向量中存储附加信息。 Qdrant允许存储任何可以使用JSON表示的信息。 <br/>


### Collections {#collections}

Collections 可以理解为关系型数据库中的表，它的一行就是一个 point 。 <br/>


### 缩略图 {#缩略图}

{{< figure src="/ox-hugo/qdrant_architecture.png" caption="<span class=\"figure-number\">Figure 1: </span>qdrant结构图" >}} <br/>


## 流程 {#流程}


### Points upsert {#points-upsert}

{{< figure src="/ox-hugo/qdrant_points_upsert.png" caption="<span class=\"figure-number\">Figure 2: </span>客户端插入Point流程图" >}} <br/>

