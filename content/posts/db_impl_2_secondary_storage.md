+++
title = "(二) 辅助储存管理"
date = 2019-05-02T21:17:00+08:00
tags = ["database"]
draft = false
author = "lyraile"
+++

## 1. 前言 {#前言}

数据库系统总会涉及辅助存储器(能够存储大量需要长期保存的数据设备)。所以研究辅助存储器的结构会利于我们分析SQL执行计划等数据库操作。 <br/>


## 2. 存储器层次 {#存储器层次}

{{< figure src="/ox-hugo/storage_structure.png" caption="<span class=\"figure-number\">Figure 1: </span>存储器结构图" >}} <br/>

高速缓存(Cache)
: 是用于减少 `CPU` 访问内存所需平均时间的部件。 `CPU` 访问高速缓存的数据只需几纳秒。 <br/>

主存储器(Main Memory)
: 数据从内存转移到 `CPU` 或 `Cache` 的速度在 `10 ~ 100` ns范围内。 <br/>

辅助存储器(Secondary Storage)
: 典型的辅助存储器是 `磁盘` 与 `固态硬盘` 。 `磁盘` 和 `主存` 间传送一个字节的时间在 =10=ms 左右。 <br/>

第三级存储器(Tertiary Storage)
: 多硬盘可以使计算机存储量非常大,但是有的数据库的数据量比单台甚至多台机器的硬盘容量大得多,为了适应这样的需求, `第三级存储器` 被开发出来,用于保存海量的数据。它的特点在于与 `辅助存储器` 相比,其读/写时间要长得多,但是容量比硬盘大得多。读取数据需要花费几秒至几分钟,但是可以达到 `PB` 级别的容量。(业务中不常见, 譬如 `光盘(DVD)` , `磁带` ) <br/>

| 存储器    | 级别  | 速度     |
|--------|-----|--------|
| 高速缓存  | 纳秒级 | 10ns内   |
| 主存储器  | 纳秒级 | 10~100ns |
| 辅助存储器(磁盘) | 毫秒级 | 10ms内   |
| 第三级存储器 | 秒/分钟级 | 几秒/几分钟 |


## 2.1 存储器层次间传递数据 {#存储器层次间传递数据}

正常情况下,数据会在相邻层间进行传输。在 `主存` 与 `辅助存储器` 之间,由于访问指定数据或查找指定位置以存储数据均会消耗大量时间。所以当需要数据时,每一层的访问都会被组织起来以便于和其下层传送大量数据。 <br/>

对于理解 `数据库` 特别重要的是 `磁盘` 被分为了 `磁盘块` 。每块大小是 `4~64` kb,整个块被从一个 `缓冲区` (例如MySQL中的buffer pool)的区域中移进移出。因此,加速数据库的一个关键技术就是 `组织数据`,使得当某一个磁盘块中有数据被访问时,大概率该块上的其他数据也需要被访问。 <br/>


## 3. 磁盘 {#磁盘}

辅助存储器的使用是数据库系统的特性之一,而辅助管理器是几乎基于磁盘/固态硬盘的。早期的数据库大部分是几乎磁盘来设计的。所以研读磁盘操作是理解数据库的有效途径之一。 <br/>


### 3.1 磁盘结构 {#磁盘结构}

磁盘驱动器大致分为两个重要的移动部件: <br/>

-   磁盘组合(disk assembly) <br/>
    -   磁盘组合由一个或多个盘片(platter)组成,它们围绕着一根中心主轴旋转。盘片的上下表面均被涂抹了一薄层磁性材料,二进制位被存储在这些磁性材料上。 <br/>
-   磁头组合(head assembly) <br/>
    -   磁头组合承载着磁头。每个盘面都有一个磁头,它极其贴近地悬浮在盘面上,但是绝对不与盘面接触(接触就会发生 `磁头损毁` ,盘面也被破坏)。磁头能读出经过它下面的盘面的磁方向,也能改变其磁方向,以便在磁盘上写信息。每个磁头被固定在一个磁头臂上,所有盘面的磁头随着磁头臂一同移进移出。 <br/>

<!--listend-->

{{< highlight text >}}
graph LR;
platter[盘面] --> track1[磁道]
platter[盘面] --> track2[磁道]
track1 --> sector1[扇区]
track1 --> sector2[扇区]
track2 --> sector3[扇区]
track2 --> sector4[扇区]
{{< /highlight >}}

| 结构 | 释义                                                   |
|----|------------------------------------------------------|
| 磁道 | 单个盘面上面的同心圆                                   |
| 扇区 | 磁道上被间隙分隔的片段。扇区是不可分割的单位,倘若一部分磁化层被损坏,那么包含这部分的整个扇区也不能再被使用 |
| 柱面 | 由盘面上半径相同的磁道组成                             |
| 间隙 | 扇区见的间隔,大约占整个磁道的10%,用于帮助标识扇区的起点儿 |
| 块 | 磁盘和主存间的传输数据的逻辑单元,由一个或多个扇区组成  |

{{< figure src="/ox-hugo/hdd_structure.png" caption="<span class=\"figure-number\">Figure 2: </span>典型磁盘结构" >}} <br/>

{{< figure src="/ox-hugo/platter_structure.png" caption="<span class=\"figure-number\">Figure 3: </span>盘面顶视图" >}} <br/>

PS: <br/>
此图有错误,每个磁道的扇区数通常是不同的,靠外圈磁道的扇区数比靠内圈的扇区数多。(越外圈的周长越大,此处涉及一个很久之前的(玄学)调优技巧,装系统分盘时根据不同软件的使用频率分盘,譬如文档分到E,F盘之类的) <br/>

{{< figure src="/ox-hugo/Cylinder_Head_Sector.png" caption="<span class=\"figure-number\">Figure 4: </span>磁盘结构" >}} <br/>

| 原文     | 译文 |
|--------|----|
| Track    | 磁道 |
| Cylinder | 柱面 |
| Sector   | 扇区 |
| Headers  | 磁头 |
| Platters | 盘片 |

> 每个盘片对应两个盘面,相对应也有2个磁头 <br/>


#### 样例 {#样例}

假设现有一块磁盘,它具有以下特效: <br/>

-   8个圆盘,16个盘面 <br/>
-   每个盘面有 \\(2^{16}=65536\\) 个磁道 <br/>
-   每个磁道(平均)有 \\(2^{8}=256\\) 个扇区 <br/>
-   每个扇区有 \\(2^{12}=4096\\) 个字节 <br/>

整个磁盘的容量是: <br/>

\\[16 \times 65536 \times 256 \times 4096 = 2^{4} \times 2^{16} \times 2^{8} \times 2^{12} = 2^{40}Bytes = 1TB\\] <br/>

一个磁道的(平均)容量是: <br/>

\\[2^{8} \times 2^{12} = 2^{20}Bytes = 1048576Bytes = 1MB\\] <br/>

一个柱面的(平均)容量是 \\(16MB\\) <br/>

假设一个块的容量是 \\(2^{14}=16KB\\) ,那么一个块使用 \\(2^{14} \div 2^{12} = 4\\) 个连续扇区,一个磁道上(平均)有 \\(256 \div 4 = 64\\) 个块,一个柱面上(平均)有 \\(16 \times 256 \div 4 = 1024\\) 个块。 <br/>


### 3.2 磁盘控制器 {#磁盘控制器}

一个或多个磁盘驱动器被一个=磁盘控制器=所控制,=磁盘控制器=是一个小处理器,可以完成以下功能 <br/>

-   控制移动磁头组合的机械马达,将磁头定位到一个特定的半径位置,使得某一柱面任何磁道都可以被读写。 <br/>
-   从磁头所在的柱面的扇区中选择一个扇区。控制器也负责识别何时旋转主轴已经到达了所要求的扇区(该扇区的起点移动到了磁头下面)。 <br/>
-   将从所要求的扇区读取的二进制位传送到计算机的主存储器。 <br/>
-   可能的话,将一整条或更多磁道缓存与磁盘控制器的内存中,期待该磁道的许多扇区能够被很快读取,从而避免对磁盘的额外访问。 <br/>

{{< figure src="/ox-hugo/simple_os.png" caption="<span class=\"figure-number\">Figure 5: </span>简单计算机系统示意" >}} <br/>

处理器经由数据总线与主存储器和磁盘驱动器通信。一个磁盘控制器可以控制多个磁盘。 <br/>


### 3.3 磁盘存取特性 {#磁盘存取特性}

存取(读或写)一个磁盘需要3步,每一步都有相关的延迟。 <br/>

1.  磁盘控制器将磁头组合定位在磁盘块所在磁道的柱面上。此步骤所消耗时间为 `寻道时间(seek time)` <br/>
2.  磁盘控制器等待访问块的第一个扇区旋转到磁头下。此步骤所消耗时间为 `旋转延迟(rotational latency)` <br/>
3.  当磁盘控制器读取或写数据时,数据所在扇区和扇区间的空隙会经过磁头。此步骤所消耗时间为 `传输时间(transfer time)` <br/>

> 寻道时间、旋转延迟和传输时间总和称为 `磁盘的延迟(latency)` <br/>

| 延迟类型(根据不同类型的磁盘不同速度) | 最大  | 最小 | 备注                                                                                |
|---------------------|-----|----|-----------------------------------------------------------------------------------|
| 寻道时间            | 约11ms | 约1ms | `寻道时间` 取决于磁头到它要访问位置的距离,如果磁头已经位于所需柱面上,则寻道时间为0,但是需要 `1ms` 的时间来启动磁头,约 `10ms` 的时间移过所有磁道 |
| 旋转延迟            | 约10ms | 0ms  | 磁盘旋转一次大约需要 `10ms`                                                         |
| 传输时间            | 毫秒级下 | 毫秒级下 | 磁道上通常有许多块,所以传输速度在毫秒级下                                           |

平均延迟: <br/>
\\[平均寻道时间(5.5ms) + 平均旋转延迟(5ms) + 平均传输时间(0.5ms) = 11ms\\] <br/>

最大延迟: <br/>
\\[最大寻道时间(11ms) + 最大旋转延迟(10ms) + 最大传输时间(1ms) = 22ms\\] <br/>


#### 样例(7200转的磁盘) {#样例7200转的磁盘}

接上个例子中的磁盘,该磁盘的一些计时特效: <br/>

-   磁盘以 `7200转/min` 的速度旋转,即 `8.33ms` 内旋转一周。 <br/>
-   在柱面之间移动磁头组合从启动到停止花费 `1ms` ,每移动 `4000` 个柱面另加 `1ms` 。这样,磁头在 `1.00025ms` 内移动一个磁道,从最内圈移动到最外圈,移动 `65536` 个磁道的距离大约用时 `17.38ms` 。 <br/>
-   一个磁道中扇区间的空隙大约占用 `10%` 的空间。间隙总占 \\(36\degree\\) 圆弧,扇区总占 \\(324\degree\\) 圆弧 <br/>

下面让我们计算读一个 `16384` 字节的块的最小、最大和平均时间。 <br/>


#### 最大时间 {#最大时间}

寻道延迟: <br/>
磁头在最外圈,要读取最内圈的数据(或者相反)。上面已给出最大的寻道时间为 `17.38ms` <br/>

旋转延迟: <br/>
最坏情况下,所需块的起点刚从磁头越过(还在这个扇区,只是起点刚刚越过)。所以我们必须从起点开始读,上面已给出最大的旋转延迟为 `8.33ms` <br/>

传输时间: <br/>
上述已给出一个扇区最大容量为 `4096` 个字节,所以该块占用$16384&divide;4096=4$个扇区。所以占用的总弧度为\\(36 \times (3 \div 256) + 324 \times (4 \div 256) \approx 5.48 \degree\\)。所以最大传输时间为\\((5.48 \div 360 ) \times 8.33 \approx 0.13ms\\) <br/>

> 最大磁盘延迟 = 最大寻道延迟 + 最大旋转延迟 + 传输时间 = 17.38 + 8.33 + 0.13 = 25.84ms <br/>


#### 平均时间 {#平均时间}

\\(平均磁盘延迟 = 平均寻道延迟 + 平均旋转延迟 + 传输时间 = 17.38 \div 2 + 8.33 \div 2 + 0.13 = 8.69 + 4.17 + 0.13 = 12.99ms\\) <br/>
(这个数值还是不准确的,磁头起始位置一般在中心位置,寻道距离是远远小于 \\(\frac{1}{2}\\) 的,不过这个值是近似值)

