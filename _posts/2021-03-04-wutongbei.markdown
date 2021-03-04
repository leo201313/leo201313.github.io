---
layout: post
title: "大数据竞赛：景区游客精准识别"
subtitle: "“梧桐杯”中国移动大数据创新大赛-智慧生活赛道"
date: 2021-03-04
author: Krad
category: Big Data
tags: competition datascience bigdata
finished: false
published: true
---

## 任务描述

移动通信系统中，如果在一定区域里两基站信号强度剧烈变化，手机就会在两个基站间来回切换，产生所谓的"乒乓效应"，所以直接通过位置计算会有较大偏差，借助算法进行轨迹数据清洗并结合客户360度画像优化模型，可以满足大型景区游客统计需求。所以本赛题的具体任务是对景区里的游客进行精准识别。

大赛地址<https://js.dclab.run/v2/cmptDetail.html?id=466>

## 数据介绍

初赛训练集为4天位置样本数据，及样本内所有用户画像,测试集A为2天的样本数据，测试集B也为2天的样本数据，其中训练集和测试集A的游客数据供选手下载，测试集B数据暂时不开放下载，在初赛最后阶段才会开放下载。选手在本地进行算法调试，在比赛页面提交结果。

1）位置数据样本，获取自基站信令数据，坐标点为基站具体坐标，如下：

![datapic1](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/3cff8ed4-5980-4d24-9d6a-03919ef867dc.jpeg)

2）用户画像数据，如下：

![datapic2](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/a3175b79-24ff-480c-8dd7-834d3083047a.jpeg)

![datapic3](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/5ac2100a-7227-4b06-87eb-cfb6d824f262.jpeg)

3）游客数据样本获取自景区门票销售数据，如下：

![datapic4](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/f3b4a9c7-32f2-4c94-ae77-8267fe374bd9.jpeg)

4）景区边界数据，如下：

![datapic5](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/1b5f2e99-e145-4a2e-8e76-61dd24f16ce1.png)

> 博物馆坐标：
<br>左上角：117.08837146047,36.659502058203
<br>右上角：117.09066108437,36.659516028328
<br>左下角：117.08829565590,36.657882069574
<br>右下角：117.09065223369,36.657899003953

## 评测方法

以模型F值进行评判

计算公式：

![deduce1](https://pu-datacastle.obs.cn-north-1.myhuaweicloud.com/pkbigdata/master.other.img/ae4b650a-788b-4a09-bc4a-ffb76189e1f1.jpeg)

## 赛题分析

首先分析位置数据样本。直接对process_time信令产生时间进行分析，分析信令是否在周末产生，以及信令是否在工作时段产生。对于基站的经纬度数据，首先应标注是否在景区范围内，同时应分化出更小的区块进行分类。