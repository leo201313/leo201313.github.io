---
layout: post
title: "三种常见复杂网络模型及python生成代码"
subtitle: "BA，WS，社区网络"
date: 2021-03-05
author: Krad
category: Complex Network
tags: introduction complexnetwork bigdata
finished: true
published: true
---

## 复杂网络

复杂网络（Complex Network），是指具有自组织、自相似、吸引子、小世界、无标度中部分或全部性质的网络。特征：小世界、集群即集聚程度的概念、幂律的度分布概念。本篇博文主要介绍三种常见网络，分别为BA，WS，社区网络，其中前两种为生成网络。同时本文将介绍如何使用python的networkx库进行网络生成或加载。

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

BA模型指的是无标度网络模型，Albert-László Barabási 和Réka Albert为了解释幂律的产生机制所提出的模型。BA模型具有两个特性，其一是增长性，所谓增长性是指网络规模是在不断的增大的，在研究的网络当中，网络的节点是不断的增加的；其二就是优先连接机制，这个特性是指网络当中不断产生的新的节点更倾向于和那些连接度较大的节点相连接。BA网络的生成代码如下：

{% highlight python %}
import networkx as nx
import matplotlib.pyplot as plt

ba = nx.barabasi_albert_graph(100, 1)

# if need drawing
ps = nx.spring_layout(ba)
nx.draw(ba, ps, with_labels = False, node_size = 50)
plt.show()
{% endhighlight %}

其中barabasi_albert_graph函数有两个参数，第一个参数指定一共生成多少个节点，第二个参数指定后续节点在加入时连接几个点。

## WS网络

小世界网络是一类特殊的复杂网络结构，在这种网络中大部分的节点彼此并不相连，但绝大部分节点之间经过少数几步就可到达。WS网络（Watts–Strogatz small-world graph）的生成代码如下：

{% highlight python %}
import networkx as nx
import matplotlib.pyplot as plt

ws = nx.random_graphs.watts_strogatz_graph(50, 10, 0.4)

# if need drawing
ps = nx.spring_layout(ws)
nx.draw(ws, ps, with_labels = False, node_size = 50)
plt.show()
{% endhighlight %}

random_graphs.watts_strogatz_graph有三个参数，分别为节点数$N$,平均度$K$（假定为偶数），和一个特殊的参数$\beta$，满足$0 \leq \beta \leq1$，且$N \gg K \gg lnN \gg 1$。该函数构造WS模型的算法如下：

1. 构建一个正则环点阵。该图有$N$个节点，每个节点和$K$个相邻的节点相连，其中每一侧有$K/2$个。即，若每个节点用$n_0,...,n_{N-1}$表示，当且仅当：<span>$0<\left | i-j \right |mod(N-1-\frac{K}{2})\leq \frac{K}{2}$</span>存在边$n_i,n_j$。

2. 对于每个节点$n_i=n_0,...,n{N-1}$，选出其与最右侧$K/2$相邻节点之间的边，即满足$i < j < i+K/2$的所有边$(n_i,n_{j mod N})$，以概率$\beta$将其重新连接。重新连接是把边$(n_i,n_{j mod N})$替换为边$n_i,n_k$，其中$k$以一致的随机性原则从所有可能的节点选出，并且避免出现自回路（即$k=i$的情况）和重复连接的情况。

## 社区网络(Network with Community Structure)

基于真实社交场景建立的复杂网络中的节点往往呈现出集群特性。例如，社会网络中总是存在熟人圈或朋友圈，其中每个成员都认识其他成员，将每一个熟人圈都可以看作为一个社区。在一个网络之中，通过社区内部的边的最短路径相对较少，而通过社区之间的边的最短路径的数目则相对较多，这也是真实社交网络与上述两种生成网络大不相同的地方。以下代码以_Facebook combined ego networks_数据集（[下载地址](https://snap.stanford.edu/data/egonets-Facebook.html)）为例：

{% highlight python %}
import networkx as nx
import matplotlib.pyplot as plt

G_fb = nx.read_edgelist("./data/facebook_combined.txt", create_using = nx.Graph(), nodetype=int)
print(nx.info(G_fb))

# if need drawing
ps = nx.spring_layout(G_fb)
nx.draw(G_fb, ps, with_labels = False, node_size = 5)
plt.show()
{% endhighlight %}

该网络画图可得到：

![csnpic1](https://res.cloudinary.com/dyd911kmh/image/upload/f_auto,q_auto:best/v1538167894/FB_mhwr8l.png)

我们还可以将网络可视化，使节点颜色随度数而变化，节点大小随中介度（Betweenness Centrality）而变化。执行此操作的代码是：

{% highlight python %}
ps = nx.spring_layout(G_fb)
betCent = nx.betweenness_centrality(G_fb, normalized=True, endpoints=True)
node_color = [20000.0 * G_fb.degree(v) for v in G_fb]
node_size =  [v * 10000 for v in betCent.values()]
plt.figure(figsize=(20,20))
nx.draw_networkx(G_fb, pos=ps, with_labels=False,
                 node_color=node_color,
                 node_size=node_size )
plt.axis('off')
{% endhighlight %}

效果图如下：

![csnpic2](https://res.cloudinary.com/dyd911kmh/image/upload/f_auto,q_auto:best/v1538167894/FB2_dxrzpc.png)

可以看到图中的各节点被分为了颜色不同的几大社区。

## 复杂网络数据聚集

这里推荐一个斯坦福大学大型网络数据集库<https://snap.stanford.edu/data/>。

