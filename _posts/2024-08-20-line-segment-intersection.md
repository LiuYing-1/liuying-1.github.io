---
layout: post
title: Line Segment Intersection
date: 2024-08-20 08:00:00
description: Self-learning note for CG - Line Segment Intersection.
tags: cg
categories: study c-ai
related_posts: true
giscus_comments: false
thumbnail: assets/img/line_segment_intersection.png
---

#### 线段求交-专题图叠合

This post is the collection of the extracts from the book Computational Geometry 3^rd^ - Berg et. al., and it is collected in Chinese so that I can recall it quickly when I need it later on.

**Why** 地图不能帮助游客知道确切想要的信息。即使知道某个小镇的大致方位，也很难在地图上确定他的位置。

**So** 为了增加可读性，地图由很多个不同类型的 thematic map 堆叠组成。比如某一层存放公路，某一层存放河流。

每层 **map 数据类型可能都不一样**。比如道路是一组线段或曲线，而城市那层可能是一组点。

为了发挥地图的最大效用，比如用户可以先通过查看城市 map 从而知道大致方向，然后通过**叠合**只关注道路等 map 来获取到这个城市的路线，从而不被别的 map 所干扰。

<div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
       <img src="https://i.imgur.com/G8g8lCu.png" alt="image-20230919201623320" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 1. 加拿大西部的城市、河流、铁道线，以及叠合后的效果
</div>

在叠合这个过程中，不同 map 相交的位置就显得很有用。比如气候学家可以通过叠合 松林覆盖 map 和降雨量 map 筛选出符合这两个条件的子区域（可以理解为交集）。

##### 线段求交

我们提取出其中的数学表达形式。

**给定由线段组成的两个集合，计算出其中来自一个集合的所有线段与来自于另一个集合线段之间的交点。**

可是怎么才算**两条线段相交**呢？回归初始问题，任何一个专题图中的单元，比如道路图中的道路，河流图中的河流，都可以表示为一条（由依次相联的）**线段（组成的）链**，因此，**“道路与河流的交汇点”即为一条链的内部和另一条链的内部的交点**。此外，交点也有可能发生在链中线段的任何一个端点处，因此，**某条线段的端点正好落在另一条 线段上也是一个交点**。

**为了进一步简化，将两组线段的集合合到一起变成一个集合，然后考虑其中所有线段之间的交点。此外，尤为关注于来自同一个集合的各线段之间的交点。**

于是，问题就演变成了：

给定由平面上 $n$ 条闭线段构成的一个集合 $S$，报告出 $S$​ 中各线段之间的所有交点。

<div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
       <img src="https://i.imgur.com/rgc3BGf.png" alt="image-20230919201623320" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 2. 最坏情况下，所有线段都两两相交。
</div>

$O$ 代表 $\leq$ 表示在最坏情况下，算法的运行时间至多为，而 $\Omega$ 代表 $\geq$ 表示在最坏情况下，算法的运行时间至少为。

对于一般情况，可以依次检查每一对线段，看看是否相交：如果的确相交，就将其交点报告出来。可是将会需要 $O(n^2)$​ 时间。结合后续提到的扫描线算法。针对如图 2 的这种情况，不管什么算法，都至少需要 $\Omega(n^2)$​ 时间。

**在实际应用中，大多数线段要么根本不与其他线段相交，要么只与少数的线段相交。因此交点的数量远达不到平方量级。**

有没有其他更快的算法呢？比如，**“输出敏感”**的算法（**Output-sensitive algorithm**）：其运行时间不仅与输入中线段的数目有关，还与实际的交点数目有关。这种算法的运行时间对实际输出的大小很敏感。这个算法也可以成为是**“交点敏感的”**（**Intersection-sensitive**），因为输出的大小就是由**交点的数目**决定的。

**在找出所有交点的过程中，如何避免对所有的线段进行测试呢？**

<u>几何特性：只有相互靠近的线段才可能会相交；而相距甚远的线段则不可能相交。</u>

利用这一特性，可以得出一个解决**线段求交问题（line segment intersection problem）**的输出敏感的算法。

:star2::star2::star2: 设定，给定一个集合 $S:=\{s_1, s_2, ..., s_n\}$。避免对相距很远的线段进行求交。

首先排除一个简单的情况，如下图。

<div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
       <img src="https://i.imgur.com/Zx3HEL8.png" alt="image-20230919201623320" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 3. 通过投影，排除不可能相交的线段对
</div>

将一条线段在 $y$ 轴上进行正交投影，定义它的 $y$ 区间 （$y-interval$）。任何两条线段，只要其 $y-interval$ 没有重叠部分，它们在 $y$ 轴方向上相距很远，换句话说，它们一定不会相交。$\implies$ 只要对那些 $y-interval$ 上有重合的那些线段进行比对即可。

为了找出这样的线段对，用一条直线 $l$ ，**从一个高于所有线段的位置，从上而下的扫过整个平面**。在这条假想的直线扫过平面的过程中，跟踪记录所有与之相交的线段，以找出所需的所有线段对。

这类算法被称为**平面扫描算法（plane sweep algorithm）**，其中使用到的直线 $l$​ 被称为**扫描线（sweep line）**。与**当前扫描线相交的所有线段的集合**是这条**扫描线的状态（status）**。随着扫描线的向下推进，它的状态不断变化，不过并不是连续的。只有在某些特定位置，才需要对扫描线的状态进行更新。我们称这些位置为平面扫描算法的时间点（event point）。在这个算法中，这里的事件点（event point）就是各线段的端点，如下图。

![image-20240825163803049](https://i.imgur.com/4JGt9R6.png)
