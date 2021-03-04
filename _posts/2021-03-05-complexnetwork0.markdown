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

查看开发文档，细品这么一句话:<span class='evidence'>_NetworkX uses a "dictionary of dictionaries of dictionaries" as the basic network data structure._</span> 对于无向图而言，以下有一个例子：

{% highlight html %}
>>> G = nx.Graph()
>>> G.add_edge('A', 'B')
>>> G.add_edge('B', 'C')
>>> print(G.adj)
{'A': {'B': {}}, 'B': {'A': {}, 'C': {}}, 'C': {'B': {}}}
{% endhighlight %}

## BA网络

BA模型指的是无标度网络模型，Albert-László Barabási 和Réka Albert为了解释幂律的产生机制所提出的模型。BA模型具有两个特性，其一是增长性，所谓增长性是指网络规模是在不断的增大的，在研究的网络当中，网络的节点是不断的增加的；其二就是优先连接机制，这个特性是指网络当中不断产生的新的节点更倾向于和那些连接度较大的节点相连接。
