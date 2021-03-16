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

## Code

本次比赛的一个关键点是要发现所给mme数据集的基站坐标会有相同地点极其接近的，猜测这是因为题目所述的乒乓效应导致。所以通过在地图上画出各坐标点，将相邻坐标点归为一类是有效的处理手段。同时，需要标注每条信息的时间段，通过查阅山东博物馆的官网，发现其开馆时间为9：00--17：00，同时下午16：00开始只出不进。地点与时间的划分方法如下图所示：

{% highlight python %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

pd.set_option('display.max_columns',200)
pd.set_option('display.max_rows',200)

data_mme0 = pd.read_csv('./data_A/data_mme_A.csv')
data_mme1 = pd.read_csv('./data_B/data_mme_B.csv')
data_mme = pd.concat([data_mme0,data_mme1],axis=0)

data_mme['pos'] = data_mme.location_latitude.astype(str) + ',' +  data_mme.loaction_longitude.astype(str)

pos_class = {
    '36.65907,117.09141':0,
    '36.65657,117.08931':1,
    '36.65639,117.09141':2,
    '36.65657,117.08931000000001':1,
    '36.65907,117.09141000000001':0,
    '36.65818,117.08941000000002':3,
    '36.65749,117.09183':4,
    '36.65818,117.08937':3,
    '36.65972,117.08922':5,
    '36.65639,117.09141000000001':2,
    '36.656566,117.089345':1
}

data_mme['district'] = data_mme.pos.map(pos_class)
data_mme['hms'] = data_mme.process_time % 1000000

hms_class = {
    (0,7):0,
    (7,9):1,
    (9,11):2,
    (11,13):3,
    (13,15):4,
    (15,17):5,
    (17,19):6,
    (19,24):7    
}
data_mme['period'] = 0


for hms_t,ind in hms_class.items():
    col_name = 'hms_' + str(ind)
    data_mme[col_name] = 0
    left_limit = hms_t[0] * 10000
    right_limit = hms_t[1] * 10000
    data_mme.loc[(left_limit<=data_mme['hms']) & (data_mme['hms']<right_limit),'period'] = ind

{% endhighlight %}

处理data_mme的最终目的，是要合并一天中的所有信息，最终生成key值为['user_id','month_id','day_id']的数据。为了描绘每天的轨迹，可以以时间为纵坐标，基站分类（'district'属性）为横坐标画成二维图进行保留，然后使用PCA技术降维保留。代码如下：

{% highlight python %}
for time in range(8):
    for pos in range(6):
        col_name = 'pic_' + str(time) + str(pos)
        data_mme[col_name] = 0 
        data_mme.loc[(data_mme['period']==time) & (data_mme['district']==pos),col_name] = 1

pos_group = data_mme.groupby(['user_id','month_id','day_id'])
data_pic = pos_group.agg({
    'pic_00':'mean','pic_01':'mean','pic_02':'mean','pic_03':'mean',
    'pic_04':'mean','pic_05':'mean','pic_10':'mean','pic_11':'mean',
    'pic_12':'mean','pic_13':'mean','pic_14':'mean','pic_15':'mean',
    'pic_20':'mean','pic_21':'mean','pic_22':'mean','pic_23':'mean',
    'pic_24':'mean','pic_25':'mean','pic_30':'mean','pic_31':'mean',
    'pic_32':'mean','pic_33':'mean','pic_34':'mean','pic_35':'mean',
    'pic_40':'mean','pic_41':'mean','pic_42':'mean','pic_43':'mean',
    'pic_44':'mean','pic_45':'mean','pic_50':'mean','pic_51':'mean',
    'pic_52':'mean','pic_53':'mean','pic_54':'mean','pic_55':'mean',
    'pic_60':'mean','pic_61':'mean','pic_62':'mean','pic_63':'mean',
    'pic_64':'mean','pic_65':'mean','pic_70':'mean','pic_71':'mean',
    'pic_72':'mean','pic_73':'mean','pic_74':'mean','pic_75':'mean',
    })

data_pic.reset_index(inplace=True)

from sklearn.decomposition import PCA
pca = PCA(n_components=16)
pca.fit(data_pic.iloc[:,3:])
pic_pca = pca.transform(data_pic.iloc[:,3:])
data_pic_pca = pd.DataFrame(pic_pca,columns=['pca0','pca1','pca2','pca3','pca4','pca5','pca6','pca7','pca8','pca9','pca10','pca11','pca12','pca13','pca14','pca15'])
concat_pca = pd.concat([data_pic,data_pic_pca],axis=1)
concat_pca.to_csv('./data_B/picpca.csv',index=False) ## 保存下来

{% endhighlight %}

至此，最重要的一部分已经解决完了，接下来还需要构造一些常规的特征，代码如下：

{% highlight python %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

import warnings

import lightgbm as lgb
from  sklearn.model_selection import KFold,StratifiedKFold
from sklearn.metrics import roc_auc_score,f1_score,cohen_kappa_score
from sklearn.preprocessing import LabelEncoder

warnings.filterwarnings('ignore')
pd.set_option('display.max_columns', 200)
pd.set_option('display.max_seq_items', 200)

data_mme0 = pd.read_csv('./data_A/data_mme_A.csv')
data_user0 = pd.read_csv('./data_A/data_user_A.csv')
data_mme1 = pd.read_csv('./data_B/data_mme_B.csv')
data_user1 = pd.read_csv('./data_B/data_user_B.csv')

data_mme1['pos'] = data_mme1.location_latitude.astype(str) + ',' +  data_mme1.loaction_longitude.astype(str)

data_mme = pd.concat([data_mme0,data_mme1],axis=0)
data_user = pd.concat([data_user0,data_user1],axis=0)
data_label = pd.read_csv('./data_A/train_label.csv')
data_pre = pd.read_csv('./data_B/to_pred_B.csv')

data_user.drop_duplicates('user_id',inplace=True)

data_mme['count'] = 1
data_mme['pos'] = data_mme.location_latitude.astype(str) + ',' +  data_mme.loaction_longitude.astype(str)

pos_class = {
    '36.65907,117.09141':0,
    '36.65657,117.08931':1,
    '36.65639,117.09141':2,
    '36.65657,117.08931000000001':1,
    '36.65907,117.09141000000001':0,
    '36.65818,117.08941000000002':3,
    '36.65749,117.09183':4,
    '36.65818,117.08937':3,
    '36.65972,117.08922':5,
    '36.65639,117.09141000000001':2,
    '36.656566,117.089345':1
}

for pos,ind in pos_class.items():
    col_name = 'pos_' + str(ind)
    if col_name not in data_mme.columns:
        data_mme[col_name] = 0
    data_mme.loc[data_mme['pos']==pos,col_name] = 1
    

## 尽管题目中说乒乓效应有害，但是实际上是否发生了乒乓效应也是判断是否为游客的重要特征
data_mme['mus_0'] = 0
data_mme['mus_1'] = 0
data_mme.loc[data_mme['pos']=='36.65818,117.08941000000002','mus_0'] = 1
data_mme.loc[data_mme['pos']=='36.65818,117.08937','mus_1'] = 1

data_mme['grass_0'] = 0
data_mme['grass_1'] = 0
data_mme.loc[data_mme['pos']=='36.65639,117.09141','grass_0'] = 1
data_mme.loc[data_mme['pos']== '36.65639,117.09141000000001','grass_1'] = 1

data_mme['art_0'] = 0
data_mme['art_1'] = 0
data_mme.loc[data_mme['pos']=='36.65907,117.09141','art_0'] = 1
data_mme.loc[data_mme['pos']== '36.65907,117.09141000000001','art_1'] = 1

data_mme['shop_0'] = 0
data_mme['shop_1'] = 0
data_mme['shop_2'] = 0
data_mme.loc[data_mme['pos']=='36.65657,117.08931','shop_0'] = 1
data_mme.loc[data_mme['pos']=='36.656566,117.089345','shop_1'] = 1
data_mme.loc[data_mme['pos']=='36.65657,117.08931000000001','shop_2'] = 1

data_mme['district_num'] = data_mme.pos.map(pos_class)
data_mme['district_most'] = data_mme.pos.map(pos_class)

data_mme['hms'] = data_mme.process_time % 1000000
data_mme['hms_max'] = data_mme['hms']
data_mme['hms_min'] = data_mme['hms']

hms_class = {
    (0,7):0,
    (7,9):1,
    (9,11):2,
    (11,13):3,
    (13,15):4,
    (15,17):5,
    (17,19):6,
    (19,24):7    
}

data_mme['period'] = 0


for hms_t,ind in hms_class.items():
    col_name = 'hms_' + str(ind)
    data_mme[col_name] = 0
    left_limit = hms_t[0] * 10000
    right_limit = hms_t[1] * 10000
    data_mme.loc[(left_limit<=data_mme['hms']) & (data_mme['hms']<right_limit),col_name] = 1
    data_mme.loc[(left_limit<=data_mme['hms']) & (data_mme['hms']<right_limit),'period'] = ind
    
data_mme['period_num'] = data_mme['period']
data_mme['period_most'] = data_mme['period']

group = data_mme.groupby(['user_id','month_id','day_id'])

ripe_mme = group.agg({
    'count':'count',
    'pos_0':'mean',
    'pos_1':'mean', 
    'pos_2':'mean', 
    'pos_3':'mean', 
    'pos_4':'mean', 
    'pos_5':'mean',
    'mus_0':'mean',
    'mus_1':'mean',
    'shop_0':'mean',
    'shop_1':'mean',
    'shop_2':'mean',
    'grass_0':'mean',
    'grass_1':'mean',
    'art_0':'mean',
    'art_1':'mean',
    'district_num':'nunique',
    'district_most':lambda x: np.mean(x.mode()),
    'hms_0':'mean', 
    'hms_1':'mean', 
    'hms_2':'mean', 
    'hms_3':'mean', 
    'hms_4':'mean',  
    'hms_5':'mean',  
    'hms_6':'mean',  
    'hms_7':'mean', 
    'period_num':'nunique',
    'period_most':lambda x: np.mean(x.mode()),
    'hms_max':'max',
    'hms_min':'min'
          })

ripe_mme.reset_index(inplace=True)
ripe_mme.district_most = ripe_mme.district_most.astype(int)
ripe_mme.period_most = ripe_mme.period_most.astype(int)

ripe_mme['max_interval'] = ripe_mme['hms_max'] - ripe_mme['hms_min'] 
ripe_mme['average_interval'] = ripe_mme['max_interval'] / ripe_mme['count']

ripe_mme['mus_pingpong'] = 0
ripe_mme['shop_pingpong'] = 0
ripe_mme['grass_pingpong'] = 0
ripe_mme['art_pingpong'] = 0

ripe_mme.loc[(ripe_mme['mus_0']>0) & (ripe_mme['mus_1']>0),'mus_pingpong'] = 1
ripe_mme.loc[(ripe_mme['grass_0']>0) & (ripe_mme['grass_1']>0),'grass_pingpong'] = 1
ripe_mme.loc[(ripe_mme['art_0']>0) & (ripe_mme['art_1']>0),'art_pingpong'] = 1

ripe_mme.loc[((ripe_mme['shop_0']>0).astype(int) + (ripe_mme['shop_1']>0).astype(int) + (ripe_mme['shop_2']>0).astype(int))>=2,'shop_pingpong'] = 1

user_count = ripe_mme.user_id.value_counts()
ripe_mme['replicate'] = ripe_mme.user_id.map(user_count)

ripe_mme['more_like_worker'] = 0
ripe_mme.loc[ripe_mme['replicate']>2,'more_like_worker'] = 1

ripe_mme['min_day'] = ripe_mme.user_id.map(ripe_mme.groupby('user_id').agg({'day_id':'min'})['day_id'])
ripe_mme['max_day'] = ripe_mme.user_id.map(ripe_mme.groupby('user_id').agg({'day_id':'max'})['day_id'])

ripe_mme['delta_day'] = ripe_mme['max_day'] - ripe_mme['min_day']
ripe_mme['fore_day'] = ripe_mme['day_id'] - ripe_mme['min_day']
ripe_mme['behi_day'] = ripe_mme['max_day'] - ripe_mme['day_id']

ripe_mme['first_come'] = 1
ripe_mme.loc[ripe_mme['fore_day']>0,'first_come'] = 0

pos_cols = [ 'pos_0', 'pos_1', 'pos_2','pos_3', 'pos_4', 'pos_5']
ripe_mme['pos_std'] = ripe_mme[pos_cols].std(axis=1)

hms_cols = [  'hms_0', 'hms_1', 'hms_2', 'hms_3','hms_4', 'hms_5', 'hms_6', 'hms_7']
ripe_mme['hms_std'] = ripe_mme[hms_cols].std(axis=1)

picpca = pd.read_csv('./data_B/picpca.csv') ##加载之前做好的PCA向量用来描述轨迹

colls = ['user_id', 'month_id', 'day_id','pca0','pca1','pca2','pca3','pca4','pca5','pca6','pca7','pca8','pca9','pca10','pca11','pca12','pca13','pca14','pca15']
ripe_mme = pd.merge(ripe_mme,picpca,on=['user_id','month_id','day_id'],how='left')

{% endhighlight %}

接下来是对用户画像的处理，主要是分析其上月与这月的流量变化与居住地变化，当如如果一个用户是本地人，他去博物馆的概率也会下降。

{% highlight python %}

data_mix0 = pd.merge(data_label,data_user,how='left',on='user_id')
data_mix0 = pd.merge(data_mix0,ripe_mme,how='left',on=['user_id','month_id','day_id'])

data_mix1 = pd.merge(data_pre,data_user,how='left',on='user_id')
data_mix1 = pd.merge(data_mix1,ripe_mme,how='left',on=['user_id','month_id','day_id'])

col_group1 = ['x15', 'x16', 'x17','x18', 'x19','x20', 'x21', 'x22', 'x23', 'x24', 'x25', 'x26', 'x27', 'x28', 'x29']
col_group2 = ['x30','x31', 'x32', 'x33', 'x34', 'x35', 'x36', 'x37', 'x38', 'x39', 'x40','x41', 'x42', 'x43', 'x44']

data_mix0['app_dou0'] = data_mix0.loc[:,col_group1].sum(axis=1) / data_mix0['x7']
data_mix0['app_dou1'] = data_mix0.loc[:,col_group2].sum(axis=1) / data_mix0['x11']

data_mix1['app_dou0'] = data_mix1.loc[:,col_group1].sum(axis=1) / data_mix1['x7']
data_mix1['app_dou1'] = data_mix1.loc[:,col_group2].sum(axis=1) / data_mix1['x11']

district_class ={
    '历下区':10, 
    '历城区':9, 
    '市中区':8, 
    '天桥区':7, 
    '槐荫区':6, 
    '章丘区':5, 
    '长清区':4, 
    '济阳县':3, 
    '商河县':2, 
    '平阴县':1    
}
data_mix0['district_id0'] = 0
data_mix0.loc[data_mix0['x1']=='济南','district_id0'] = data_mix0.loc[data_mix0['x1']=='济南','x2'].map(district_class)
data_mix0['district_id1'] = 0
data_mix0.loc[data_mix0['x3']=='济南','district_id1'] = data_mix0.loc[data_mix0['x3']=='济南','x4'].map(district_class)

data_mix1['district_id0'] = 0
data_mix1.loc[data_mix1['x1']=='济南','district_id0'] = data_mix1.loc[data_mix1['x1']=='济南','x2'].map(district_class)
data_mix1['district_id1'] = 0
data_mix1.loc[data_mix1['x3']=='济南','district_id1'] = data_mix1.loc[data_mix1['x3']=='济南','x4'].map(district_class)

data_mix0['native'] = 0
data_mix0.loc[data_mix0['city_id']=='济南','native'] = 1
data_mix1['native'] = 0
data_mix1.loc[data_mix1['city_id']=='济南','native'] = 1


data_mix0['district0'] = data_mix0['x1'].astype(str) + data_mix0['x2'].astype(str)
data_mix0['district1'] = data_mix0['x3'].astype(str) + data_mix0['x4'].astype(str)
data_mix1['district0'] = data_mix1['x1'].astype(str) + data_mix1['x2'].astype(str)
data_mix1['district1'] = data_mix1['x3'].astype(str) + data_mix1['x4'].astype(str)

data_mix0['district_change'] = (data_mix0.district0 == data_mix0.district1).astype(int)
data_mix1['district_change'] = (data_mix1.district0 == data_mix1.district1).astype(int)

data_mix0['manyou_delta'] = data_mix0['x5'] - data_mix0['x6']
data_mix1['manyou_delta'] = data_mix1['x5'] - data_mix1['x6']

data_mix0['dou_rate0'] = data_mix0['x8'] / data_mix0['x7']
data_mix0['dou_rate1'] = data_mix0['x12'] / data_mix0['x11']
data_mix0['mou_rate0'] = data_mix0['x10'] / data_mix0['x9']
data_mix0['mou_rate1'] = data_mix0['x14'] / data_mix0['x13']

data_mix1['dou_rate0'] = data_mix1['x8'] / data_mix1['x7']
data_mix1['dou_rate1'] = data_mix1['x12'] / data_mix1['x11']
data_mix1['mou_rate0'] = data_mix1['x10'] / data_mix1['x9']
data_mix1['mou_rate1'] = data_mix1['x14'] / data_mix1['x13']

data_mix0['app_use0'] = 0
data_mix0.loc[data_mix0['app_dou0']>0,'app_use0'] = 1
data_mix0['app_use1'] = 0
data_mix0.loc[data_mix0['app_dou1']>0,'app_use1'] = 1

data_mix1['app_use0'] = 0
data_mix1.loc[data_mix1['app_dou0']>0,'app_use0'] = 1
data_mix1['app_use1'] = 0
data_mix1.loc[data_mix1['app_dou1']>0,'app_use1'] = 1


data_mix0['app_rate0'] = data_mix0['app_dou0'] / data_mix0['x7']
data_mix0['app_rate1'] = data_mix0['app_dou1'] / data_mix0['x11']

data_mix1['app_rate0'] = data_mix1['app_dou0'] / data_mix1['x7']
data_mix1['app_rate1'] = data_mix1['app_dou1'] / data_mix1['x11']

data_mix0['dou_class0'] = 0
data_mix0.loc[data_mix0['dou_rate0']>0,'dou_class0'] = 1
data_mix0.loc[data_mix0['dou_rate0']>0.01,'dou_class0'] = 2
data_mix0['dou_class1'] = 0
data_mix0.loc[data_mix0['dou_rate1']>0,'dou_class1'] = 1
data_mix0.loc[data_mix0['dou_rate1']>0.01,'dou_class1'] = 2

data_mix0['mou_class0'] = 0
data_mix0.loc[data_mix0['mou_rate0']>0,'mou_class0'] = 1
data_mix0.loc[data_mix0['mou_rate0']>0.01,'mou_class0'] = 2
data_mix0['mou_class1'] = 0
data_mix0.loc[data_mix0['mou_rate1']>0,'mou_class1'] = 1
data_mix0.loc[data_mix0['mou_rate1']>0.01,'mou_class1'] = 2

data_mix1['dou_class0'] = 0
data_mix1.loc[data_mix1['dou_rate0']>0,'dou_class0'] = 1
data_mix1.loc[data_mix1['dou_rate0']>0.01,'dou_class0'] = 2
data_mix1['dou_class1'] = 0
data_mix1.loc[data_mix1['dou_rate1']>0,'dou_class1'] = 1
data_mix1.loc[data_mix1['dou_rate1']>0.01,'dou_class1'] = 2

data_mix1['mou_class0'] = 0
data_mix1.loc[data_mix1['mou_rate0']>0,'mou_class0'] = 1
data_mix1.loc[data_mix1['mou_rate0']>0.01,'mou_class0'] = 2
data_mix1['mou_class1'] = 0
data_mix1.loc[data_mix1['mou_rate1']>0,'mou_class1'] = 1
data_mix1.loc[data_mix1['mou_rate1']>0.01,'mou_class1'] = 2

{% endhighlight %}

至此，数据终于处理完了，可以看一下现在data_mix0的columns有哪些：

{% highlight python %}
>> data_mix0.columns
Index(['user_id', 'month_id', 'day_id', 'label', 'city_id', 'x1', 'x2', 'x3',
       'x4', 'x5', 'x6', 'x7', 'x8', 'x9', 'x10', 'x11', 'x12', 'x13', 'x14',
       'x15', 'x16', 'x17', 'x18', 'x19', 'x20', 'x21', 'x22', 'x23', 'x24',
       'x25', 'x26', 'x27', 'x28', 'x29', 'x30', 'x31', 'x32', 'x33', 'x34',
       'x35', 'x36', 'x37', 'x38', 'x39', 'x40', 'x41', 'x42', 'x43', 'x44',
       'flag', 'count', 'pos_0', 'pos_1', 'pos_2', 'pos_3', 'pos_4', 'pos_5',
       'mus_0', 'mus_1', 'shop_0', 'shop_1', 'shop_2', 'grass_0', 'grass_1',
       'art_0', 'art_1', 'district_num', 'district_most', 'hms_0', 'hms_1',
       'hms_2', 'hms_3', 'hms_4', 'hms_5', 'hms_6', 'hms_7', 'period_num',
       'period_most', 'hms_max', 'hms_min', 'max_interval', 'average_interval',
       'mus_pingpong', 'shop_pingpong', 'grass_pingpong', 'art_pingpong',
       'replicate', 'more_like_worker', 'min_day', 'max_day', 'delta_day',
       'fore_day', 'behi_day', 'first_come', 'pos_std', 'hms_std', 'pic_00',
       'pic_01', 'pic_02', 'pic_03', 'pic_04', 'pic_05', 'pic_10', 'pic_11',
       'pic_12', 'pic_13', 'pic_14', 'pic_15', 'pic_20', 'pic_21', 'pic_22',
       'pic_23', 'pic_24', 'pic_25', 'pic_30', 'pic_31', 'pic_32', 'pic_33',
       'pic_34', 'pic_35', 'pic_40', 'pic_41', 'pic_42', 'pic_43', 'pic_44',
       'pic_45', 'pic_50', 'pic_51', 'pic_52', 'pic_53', 'pic_54', 'pic_55',
       'pic_60', 'pic_61', 'pic_62', 'pic_63', 'pic_64', 'pic_65', 'pic_70',
       'pic_71', 'pic_72', 'pic_73', 'pic_74', 'pic_75', 'pca0', 'pca1',
       'pca2', 'pca3', 'pca4', 'pca5', 'pca6', 'pca7', 'pca8', 'pca9', 'pca10',
       'pca11', 'pca12', 'pca13', 'pca14', 'pca15', 'app_dou0', 'app_dou1',
       'district_id0', 'district_id1', 'native', 'district0', 'district1',
       'district_change', 'manyou_delta', 'dou_rate0', 'dou_rate1',
       'mou_rate0', 'mou_rate1', 'app_use0', 'app_use1', 'app_rate0',
       'app_rate1', 'dou_class0', 'dou_class1', 'mou_class0', 'mou_class1'],
      dtype='object')

{% endhighlight %}

从这些列中选取特征。

{% highlight python %}
feat_cols = ['manyou_delta',
             'x5',
             'x6',
             'count',
             'pos_0', 
             'pos_1', 
             'pos_2', 
             'pos_3', 
             'pos_4', 
             'pos_5',
             'district_num',
             'district_most',
             'first_come',
             'hms_0', 
             'hms_1', 
             'hms_2', 
             'hms_3', 
             'hms_4',
             'hms_5',
             'hms_6',
             'hms_7',
             'period_num',
             'period_most',
             'max_interval',
             'average_interval',
             'mus_pingpong', 
             'shop_pingpong',
             'grass_pingpong', 
             'art_pingpong',
             'replicate',
#              'more_like_worker',
             'fore_day',
#              'behi_day', #该特征风险极高，可能导致overfit
             'pos_std', 
             'hms_std',
             'lda',
             'pca0',
             'pca1', 
             'pca2', 
             'pca3', 
             'pca4', 
             'pca5', 
             'pca6', 
             'pca7',
             'pca8', 
             'pca9',
             'pca10', 
             'pca11', 
             'pca12', 
             'pca13', 
             'pca14', 
             'pca15',
#               'pic_00',
#               'pic_01', 'pic_02', 'pic_03', 'pic_04', 'pic_05', 'pic_10', 'pic_11',
#               'pic_12', 'pic_13', 'pic_14', 'pic_15', 'pic_20', 'pic_21', 'pic_22',
#               'pic_23', 'pic_24', 'pic_25', 'pic_30', 'pic_31', 'pic_32', 'pic_33',
#               'pic_34', 'pic_35', 'pic_40', 'pic_41', 'pic_42', 'pic_43', 'pic_44',
#               'pic_45', 'pic_50', 'pic_51', 'pic_52', 'pic_53', 'pic_54', 'pic_55',
#               'pic_60', 'pic_61', 'pic_62', 'pic_63', 'pic_64', 'pic_65', 'pic_70',
#               'pic_71', 'pic_72', 'pic_73', 'pic_74', 'pic_75',
             'district_id0', 
             'district_id1',
             'native', 
             'district_change',
             'app_rate0',
             'app_rate1',
             'dou_rate0', 
             'dou_rate1',
             'mou_rate0', 
             'mou_rate1']

{% endhighlight %}

最后我选择了LGBM作为模型进行训练，采用K折交叉验证的方式。

{% highlight python %}
def eval_error(pred,train_set):
    labels = train_set.get_label()
    pred = pred.reshape((2,int(len(pred)/2))).T
    y_pred = pred.argmax(axis=1)
    score = f1_score(labels,y_pred)
    return 'f1_score',score,True

def model_train(df,test_,trainlabel,feature):
    '''
    @param df: 训练数据 DataFrame
    @param test_ : 测试数据 DataFrame
    @param trainlabel：训练标签 string  eg. 'label'
    @param feature ：所有训练特征 list  eg. ['feat1','feat2'...]
    @return sub_preds: 预测数据
    '''
    train_= df.copy()
    num_class = 2
    n_splits = 5
    oof_lgb = np.zeros([len(train_),num_class])
    folds = KFold(n_splits=n_splits, shuffle=True, random_state=2021)
    sub_preds = np.zeros([test_.shape[0],num_class])
    sub_preds1 = np.zeros([test_.shape[0],n_splits])
    use_cart = True
    cate_cols = []
    label = trainlabel
    pred = list(feature)
    params = {
        'learning_rate': 0.01,
        'boosting_type': 'gbdt',
        'objective':'multiclass',
        #'metric':'multi-error',
        'num_class':num_class,
        'num_leaves':100,
        'feature_fraction': 0.8,
        'bagging_fraction': 0.8,
        'bagging_freq': 5,
        'seed': 1,
        'bagging_seed': 1,
        'feature_fraction_seed': 5,
        'min_data_in_leaf': 10,
        'max_depth':-1,
        'nthread': 8,
        'verbose': 1,
#         'is_unbalanace':True,
#         'lambda_l1': 0.4,  
#         'lambda_l2': 0.5, 
        'device': 'cpu'
    }
    for n_fold, (train_idx, valid_idx) in enumerate(folds.split(train_[pred],train_[[label]]), start=1):
        print('the %s training start ...'%n_fold)
        
        temp_f1 = 0

        train_x, train_y = train_[pred].iloc[train_idx], train_[[label]].iloc[train_idx]
        valid_x, valid_y = train_[pred].iloc[valid_idx], train_[[label]].iloc[valid_idx]
        #x = train_['bid'].iloc[valid_idx]
        print(train_y.shape)
        if use_cart:
            dtrain = lgb.Dataset(train_x, label=train_y, categorical_feature=cate_cols)
            dvalid = lgb.Dataset(valid_x, label=valid_y, categorical_feature=cate_cols)
            #dvalid1 = lgb.Dataset(test_[pred], label=test_[['label']], categorical_feature=cate_cols)

        else:
            dtrain = lgb.Dataset(train_x, label= train_y)
            dvalid = lgb.Dataset(valid_x, label= valid_y)

        clf = lgb.train(
            params=params,
            train_set=dtrain,
            num_boost_round=1000,
            valid_sets=[dvalid],
           # early_stopping_rounds = 100,
            verbose_eval=100
           ,feval=eval_error
        )
        
        sub_preds += clf.predict(test_[pred].values,num_iteration=1500)/ folds.n_splits
        sub_preds1[:,n_fold-1] = clf.predict(test_[pred].values,num_iteration=400).argmax(axis=1)
        train_pred = clf.predict(valid_x,num_iteration=clf.best_iteration)
        y_pred = train_pred.argmax(axis=1)
        oof_lgb[valid_idx] = train_pred

        
    #print('MEAN AUC:',np.mean(auc))
    
    return sub_preds,oof_lgb,clf,sub_preds1

sub_preds,oof_lgb,clf,sub_preds1 = model_train(data_mix0,data_mix1,'label',feat_cols)
data_mix1['label'] = sub_preds.argmax(axis=1)
data_mix1[['id','label']].to_csv('./data_A/lgbm.csv',index=False)

{% endhighlight %}

该模型最终在B榜获得了F1值为0.954657的成绩。