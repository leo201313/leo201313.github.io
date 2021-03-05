---
layout: post
title: "三种生成网络及代码"
subtitle: "BA，WS，GN网络生成"
date: 2021-03-05
author: Krad
category: Complex Network
tags: introduction complexnetwork bigdata
finished: false
published: true
---

## 复杂网络

复杂网络（Complex Network），是指具有自组织、自相似、吸引子、小世界、无标度中部分或全部性质的网络。特征：小世界、集群即集聚程度的概念、幂律的度分布概念。本篇博文主要介绍三种生成网络，分别为BA，WS，GN，并且介绍如何使用python的networkx库进行生成。

## Networkx库

查看开发文档，细品这么一句话:<span class='evidence'>_NetworkX uses a "dictionary of dictionaries of dictionaries" as the basic network data structure._</span> 意思是NetworkX里的Graph都是由三种相嵌的字典集构成。最外层的字典的key值是图中的点，次外层的key值是选中点的相邻点（存在连接关系），最里层的字典包含的是边的值（可以自定义很多边上的值，如颜色、权重等），对于无向图而言，以下有一个例子：

{% highlight python %}
G = nx.Graph()
G.add_edge('A', 'B')
G.add_edge('B', 'C')
print(G.adj)
{'A': {'B': {}}, 'B': {'A': {}, 'C': {}}, 'C': {'B': {}}}
G.add_edge(1, 2, color='red', weight=0.84, size=300)
print(G[1][2]['size'])
300
print(G.edges[1, 2]['color'])
red
{% endhighlight %}

采取三层字典的数据结构的好处如下：

* 通过指定外层两个key值（节点）删除边
* 对于"list"和"sets"都很友好
* G[u][v]直接返回边的属性值
* 支持 n in G ， 如果n节点在G中返回true值
* 支持 for n in G 遍历所有节点
* 支持 nbr in G[n] 遍历节点n的所有邻节点

## BA网络

BA模型指的是无标度网络模型，Albert-László Barabási 和Réka Albert为了解释幂律的产生机制所提出的模型。BA模型具有两个特性，其一是增长性，所谓增长性是指网络规模是在不断的增大的，在研究的网络当中，网络的节点是不断的增加的；其二就是优先连接机制，这个特性是指网络当中不断产生的新的节点更倾向于和那些连接度较大的节点相连接。
