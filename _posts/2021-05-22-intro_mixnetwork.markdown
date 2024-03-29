---
layout: post
title: "复杂网络级联效应仿真平台介绍"
subtitle: "MixNetwork Introduction"
date: 2021-05-22
author: Krad
category: Complex Network
tags: code complexnetwork bigdata
finished: true
published: true
---

## Welcome to the MixNetwork!

最近在做毕业设计，题目为建立一个复杂网络级联攻击模型。在查阅了各方面文献后，发现python编写的复杂网络平台存在缺失，所以本人在networkx库的基础上进行拓展，建立了本复杂网络级联攻击仿真平台，called MixNetwork！

MixNetwork仿真平台的基本功能由三种模型构成，即恢复模型、代价模型与级联模型。对于任意的复杂网络数据集，在被MixNetwork加载后首先将建立上述三种模型作为仿真平台的底层支持。MixNetwork支持用户自定义三种模型的相关参数实现不同复杂网络的差异化处理。MixNetwork在加载了复杂网络数据集，实现了底层功能初始化后将创建并初始化攻击模型。不同的攻击模型尽管攻击方式与算法不同，但都是通过仿真平台提供的基本功能返回局部信息并与仿真平台交互完成攻击。仿真平台支持多种不同的攻击模型在其上进行仿真，为了加速仿真的效率，扩大仿真规模，仿真平台建立了缓存优化功能，以及限制攻击模型利用全局信息，将攻击算法被限制为只能利用局部信息的方式。仿真平台支持两种启动与操作模式，一种是图形化的操作模式，一种是命令行的操作模式，这两种操作模式可以满足用户大部分情况的需求。图形化的操作模式可以直观的反映不同攻击模型在不同的复杂网络中的攻击效果。命令行的操作模式因为省去了图形构建，可以更加快速地进行仿真，得出最终的试验结果。MixNetwork的功能构建如下图所示。
![pic1](../img/pysimuComplexNetwork/framework.png)

本项目的核心部分在于MixNetwork类，该类基于networkx库的Graph类实现网络中边的保存，子类Nodes完成对网络中节点属性的保存，以及子类Attacker对于网络中攻击模型属性的保存。其代码结构如下图。

![pic1](../img/pysimuComplexNetwork/codeframework.png)

MixNetwork的目标是**高效**地仿真复杂网络节点在不同攻击模型攻击的情况下的级联变化。主要的加速方式是采用缓存优化机制，MixNetwork中有一个pool，该pool会记忆可能在将来被级联影响的nodes，在需要循环扫描时不用直接扫描全局节点。同时攻击模型的一切行动都无法使用全局信息，只能利用局部信息。这种设定也更加符合真是网络下的攻击场景。

![pic1](../img/pysimuComplexNetwork/pool.png)


MixNetwork在读取了网络数据后会自动为每一条边分配合理的权重值，同时计算每个节点的初始负载值。MixNetwork会根据每个节点的初始负载值分配每个节点最大负载值，其计算方式是通过将初始负载乘以一大于1的系数加上一正态分布值。 在MixNetwork中节点有三种状态，分别为正常（状态值为1，下同），超载（2）以及被击破（0）。当节点被击破或者超载时，会将其上的**初始**负载值（L）重新分配给一定范围内的**正常**节点，如果没有符合的节点则不会重新分配，具体的分配方式为设定参数α（α<1），距离待重新分配节点一个标度的节点们根据自身的**实时度数**均分α倍L，而距离待重新分配节点两个标度的节点根据自身的实时度数均分α平方倍的L，以此类推。

MixNetwork现阶段只支持对节点负载的重新分配与级联故障。规定当节点的负载超出最大负载值时，节点将会被切换至超载状态，无法行使正常功能，也即级联故障。该种方式的级联设定适合电网与路由器网络场景的设置，但不适用于航班，地铁线路等网络场景的级联仿真。

对于每个节点而言，拥有一个生命值（health），生命值在初始时都设为100，当生命值降低为50时节点即被击破。节点拥有不同的攻击代价以及在被击破后的恢复能力。攻击代价原则上应该与节点的重要性相挂钩，即越重要的节点攻击代价越大。攻击者需要花费超过攻击代价的攻击能力才能损害到节点的生命值。同时被击破的节点由于具有恢复能力，所以需要攻击者持续进行攻击压制才能够防止已被击破的节点重新恢复工作能力。MixNetwork默认的节点攻击代价值与节点初始度数相关，节点恢复能力与节点的局部聚合度相关。

其他更多个性化定制请阅读具体代码的各项参数。

## Display
MixNetwork提供给用户两种启动并操作仿真平台的方式，分别为图形界面的方式与命令行的方式。图形界面的启动方式可以让用户值观感受攻击模型的每一步攻击对于网络的影响，而命令行启动方式可以省去图形处理完成更大规模的复杂网络的快速仿真。GUI启动模式展示如下：
![pic1](../img/pysimuComplexNetwork/pic1.png)
![pic1](../img/pysimuComplexNetwork/pic2.png)

## How to use
Just do as follow:

    python3 CNplatform/pynetsimu/run_gui.py        ----> to use MixNetwork with GUI

    python3 CNplatform/pynetsimu/run_normal.py     ----> to use MixNetwork much faster but without GUI

## Requirements

Python Version: 3.7

Required Python Libraries: networkx 2.2, matplotlib 3.0.2, and tk 8.6.8.

## Update Panel
* v2.3 增加了能够利用级联效应的攻击模型OC，优化了GUI界面的启动方式
* v2.2 新增了NND与NNDRS的攻击方式
* v2.1 新增了两种可以压制节点恢复的攻击方法，新增了两种大规模数据集的支持，优化了代码结构
* v2.0 优化了攻击方法的数据结构，编写了不要GUI的运行方式，实现了计算单次攻击消耗来判断攻击是否达到稳态，修复了各种bug
* v1.3 更新了节点恢复模型的曲线设定，设置被级联故障的节点也能够自我恢复，同时节点恢复不再采用级联的恢复方式，更加符合实际需求
* v1.2 更新了节点恢复模型，现在的节点恢复模型是通过恢复曲线进行计算的，同时已有的两种攻击模型也根据新的恢复模型进行了优化
* v1.1 优化了RD攻击模型的局部位置移动算法；增加了随机攻击模型RANDOM；增加了叙述攻击模型攻击效果的两种方式；使用紫色节点绘制上一次攻击的节点，方便区分已经被攻击的节点。
* v1.0 项目开源






