+++
title = "摘录:分布式数据库学习路线"
date = 2018-12-27

[taxonomies]
tags = ["database", "excerpts", "distributed system"]
categories = ["Database System"]
+++

## 来源

摘录自知乎问题:[对数据库和分布式很感兴趣，学习路线是什么？](https://www.zhihu.com/question/62464757)

<!-- more -->

## [李晨曦的回答](https://www.zhihu.com/question/62464757/answer/202033688):

分布式的学习不是很了解，数据库有些经验可以按照如下步骤学习

###  1. 入门

接触过数据库的基础知识和应用之后，就可以开始进行内核级的学习了，先学习传统的关系型数据库，因为它是一切数据库的基础，包括现在比较火的内存数据库，分布式数据库等。

从`IV Database System Implementation`开始，把这本书吃透了，包括数据的物理存储结构，索引，查询引擎，缓冲池，SQL编译，查询优化，系统恢复，并发控制。然后实现上面这些理论知识，完成一个支持简单DML，DDL的数据库。一定要实现，只是学习理论，不动手实践是不可能深刻理解的，就像空中楼阁。

[《数据库系统实现（英文版）（第2版）》](http://rtbs24.com/?target=https%3A%2F%2Fytthn.com%2Fclick-IQL4686A-HFDQCIIE%3Fbt%3D25%26tl%3D1__%26url%3Dhttp%3A%2F%2Fitem.jd.com%2F10059733.html)

http://dsm.fudan.edu.cn/JSPWiki/attach/Material_db/database%20system%20complete%20book_2nd.pdf2

### 2. 深入理解

参考这个论文列表，把里面的论文都读一遍，了解关系型数据库历史，了解数据库的发展过程中走过的弯路，理解它为什么会变成现在这个样子

[rxin/db-readings](https://github.com/rxin/db-readings)

同时也要锻炼系统级编程的能力，深刻理解操作系统的工作原理，并行编程，熟悉一些硬件的工作原理，为以后打下良好的基础。可以读一些优秀的开源数据库的源码，如Peloton的[cmu-db/peloton](https://github.com/cmu-db/peloton)，只有几万行代码，PostgreSQL的，代码非常多，不过注释非常清晰

### 3. 进阶

看最近几年数据库相关的论文，各个方向的发展都全面了解一下，可以参考这两个课程提到的论文[Schedule - CMU 15-721 :: Advanced Database System (Spring 2017)](https://15721.courses.cs.cmu.edu/spring2017/schedule.html)，[Practical Course: Database Implementation](https://db.in.tum.de/teaching/ws1617/imlab/?lang=en)，找一些感觉兴趣的论文复现一下里面提到的方法。很多算法，数据结构都有一定的适用场景和局限性，要通过自己实现与实验来深刻体会。

### 4. 破茧成蝶

这才是读博最重要的阶段，通过之前的学习，应该对数据库的各个方面都非常了解了，而且也打下了良好的编程能力，可以选1,2个感兴趣的方向，看看有没有什么可以改进，突破的点，想出自己的idea，实现并与现有的方法进行对比，如果可以做的很好，就非常厉害了。

## [Ed Huang的回答](https://www.zhihu.com/question/62464757/answer/202312500):

深夜倒时差睡不着，上来写写

对传统数据库不太了解，毕竟不是科班出身。

对分布式系统学习还是有点心得，理论基础要打牢。

1. 从存储系统入手，Google 的老三篇入门，最好能顺手把 6.824 做了，不难，智商正常的本科生都能做完，另外推荐一本书 Distributed systems for fun and profit

2. 做完 6.824 后就可以从复制协议开始入手, Paxos 的几篇，Lamport 那篇有空膜拜一下好了，真正有价值的是 Paxos made live / Paxos made simple 那几篇，然后可以深入看看 Raft, 这个在 6.824 里面会用到.

3. 然后开始开非 Google 系的存储系统比如 Dynamo ，Haystack 啊什么的还有一些最终一致性的系统，比如 FB 在一些系统上的设计虽然没有 Google 那么 fancy，但是看看还是不错的，至少知道在 FB 的数量级下会遇到那些问题，如何用糙快猛的办法 workaround。。。 和一些分布式计算系统和流计算系统，比如  MR 就不说了，比如 Dremel 啊，Spark 啊，MillWheel 啊，Sawzall 啊

4. 把 SQL 优化器的一些基础知识学会咯，然后尽量用分布式系统的思想去思考。然后有点感觉了以后，可以看 F1 和 2017 的 Spanner 那两篇论文找找感觉，毕竟比较简单。然后就可以去找一些 OLAP 系统的论文看看了，HyPer 有一堆论文，Impala / Presto / Kudu 啊， AsterixDB 啊什么的，这个领域就多了去了。

5. 这时候就可以开始实践了，自己动手撸没啥意思，而且工程量巨大无比，我建议还是要和靠谱的团队一起工作，进步很快，比如暑假没事或者实验室老师不管的话可以来我们这边实实习啊，毕竟鄙司是分布式系统和数据库技术融合得很不错的公司，离你们学校又近，打个车 20 分钟就到了，我的邮箱是 huang at http://pingcap.com，不过建议还是你把邮件发给 shenli，毕竟我推荐没有 iPhone 拿，他是有的...有机会未来见。
