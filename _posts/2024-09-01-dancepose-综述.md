---
title: 'DancePose初步研究'
date: 2012-08-14
permalink: /posts/2012/08/blog-post-8/
tags:
  - cool posts
  - category1
  - category2
---

# DancePose初步研究

## 入门——单一动作识别

Link：https://www.bilibili.com/video/BV1dL4y1h7Q6?p=2&vd_source=465dc960d84cfa3c588da2936bbf8fe7

Tag： `计算机视觉——关键点检测`，`回归`，`目标跟踪`，`重识别`

**研究问题**

* 单人-->多人：top-down(检测每个人区域，对每人进行关键点检测), bottom-up（检测每个关键点，聚类组装人）

* 追踪问题：

* DancePose:


**人体姿态估计应用场景**


**Baseline算法**

`Openpose`

    CMU蜂巢摄像采集

`Deepcut`

`AlphaPose`

`SimpleBasline`

`HRNet`

`BlazePose`

    实时边缘计算，准确度下降
    追踪思路
    技术=Heatmaps（输出灰度图--33关键点出现概率,3offset*33=一共99个offset）+regression


## google mediaipe
33个关键点
17mscoco


### 同济子豪兄-AI健身动作计数 

视频看作一组有顺序的图片，每张图进行二分类（标注1类别的置信度）

识别到（置信度>某值）时，计数+1

流程：
（这个流程图是按类的定义顺序写的，其实不对。应该按调用的顺序写）
![picture 2](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240527-164751489.png)  
（第二张稍微准确一些）

![picture 3](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240527-165805231.png)  


主要算法：图片特征提取和二分类

- 二分类图片-->pose embedding转化为数值数据(csv)

    实现：收集25张站起图片，28张蹲下图片，进行特征提取（提取33个点的xyz坐标）

- poseclassifier姿势分类器，采用传统数学算法，衡量最相似两张图片pose33个关键点的mean distance，完成分类。


所有类：
![picture 1](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240527-164740512.png)  

------



keyframes-->actions-->moves

我的问题，需要识别舞蹈动作而不是单一计数，所以不能根据一个动作做出判断，而需要根据多个动作。根据之前问chatGPT的，这应该是一个sequence  Action recognition

**不同点**

1. 动作的复杂性和范围：

    姿态估计与动作分类：通常关注单一动作或简单动作的分类，如深蹲、跳跃等。这涉及到从单个动作中提取关键帧或关键特征。

    序列动作识别：关注的是一系列动作或整个活动序列的识别，如一段完整的舞蹈或一系列体操动作。这需要理解动作之间的时序关系和连续性。
2. 时间依赖性：

    姿态估计与动作分类：虽然有时也考虑时间信息，但主要关注的是静态图像中的姿态或短时序列。

    序列动作识别：强调对长时间序列的理解，需要处理和分析时间上的依赖和变化，往往使用RNN或LSTM等模型来捕捉时间序列中的动态特征。
3. 技术实现：

    姿态估计与动作分类：可能更侧重于图像处理和特定关键点的检测。

    序列动作识别：更侧重于序列建模和长期依赖关系的处理。
    如果要检测一个完整舞蹈动作



**如果目标是检测和识别一个完整的舞蹈动作，以下步骤可以指导实现：**

- 数据准备：收集包含不同舞蹈的视频数据，并进行适当的标注，标注应包括不同舞蹈的类型以及可能的舞蹈动作序列。

- 特征提取：使用深度学习模型（如基于CNN的姿态估计网络）来提取视频中的关键帧和关键点信息。

- 序列模型训练：采用RNN、LSTM或Transformer等模型来处理时间序列数据，捕捉舞蹈中动作的时序关系和动态变化。

- 模型训练与评估：在标注的舞蹈视频上训练模型，使用交叉验证等方法来评估模型的性能，确保模型能够准确识别不同的舞蹈类型和动作。

- 部署与实时处理：将训练好的模型部署到实际应用中，实现对实时视频流的舞蹈动作识别。


## 进阶——序列动作识别

### 舞蹈视频Data
----


1. **AIST DanceDB**

    下载链接：
    https://aistdancedb.ongaaccel.jp/database_download/

    下载方式：
    - 下载zip file
    - 从url下载（首选！将需要的是video url筛选出来，然后只加载要用的一部分）url下载方式链接里有
    https://aistdancedb.ongaaccel.jp/getting_the_database/






### Paper&Code
-----
### **舞蹈领域识别** *DANCE-MOTION GENRE CLASSIFICATION*
- 数据加载
    - 下载并处理csv
    - 下载csv中筛选出的视频url
    - 分为trainingset validation set和test set

```
4.1 Experimental Conditions
    
    Data--Advanced Dance 210 videos (10genres* 3dancers * 7choreographies)

    training set—126+validation set—14 (10genres * 2dancers * 7choreo)

    test set—70 (10genres * 1remaining dancer * 7choreo)
```
按照Paper的说法，要从Advanced Dance中筛选出符合条件的210个videos，所以要从这个带有metadata标签（![picture 6](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-101544944.png)  
）的refined_10M_sFM_all.csv中筛选符合条件的视频url

Excel里一查，发现符合paper里front camera only（用c01摄像机）的正好210个
![picture 7](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-102750616.png)  
这就好办了

```python
# 安装必要的库
!pip install pandas
!pip install requests

# 导入库
import pandas as pd
import requests

# 加载CSV文件
csv_url = "https://aistdancedb.ongaaccel.jp/data/video_refined/10M/refined_10M_sFM_all.csv"
df = pd.read_csv(csv_url)

# 根据CAMERA=c01筛选数据
filtered_df = df[df['CAMERA'] == 'c01']

# 打印筛选后的数据以检查
print(filtered_df.head())
# 检查一共筛选出多少数据
print(f"Total number of filtered videos: {len(filtered_df)}")
```
![picture 8](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-103624514.png)  

下载文件之前，先看看默认下载地址
![picture 11](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-104345055.png)  
![picture 13](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-104434593.png)  
新建一个文件专门放dance videos

连接google drive云盘，挂载到某目录（不然下次再打开就没了）
开始下载视频
https://gongenbo.github.io/2019/06/26/Google-Colab%E5%8A%A0%E8%BD%BD%E6%95%B0%E6%8D%AE%E9%9B%86%E7%9A%84%E5%B8%B8%E7%94%A8%E6%96%B9%E5%BC%8F/

哈哈好慢大概1分钟下载7个，那210大概要30min，好了我可以去干别的了。

- 特征提取

1. Extract per frame

```
4.2 Methods
    In the first step of
    motion-feature extraction, we use the OpenPose library to estimate the dancer’s skeleton (body pose and motion) in all video frames (60 frames per second).

    Pose:
        21 joint angles * 2(θx and θy--the sine and cosine of the angles)=42-dimensional feature vector
        (缺失值用0表示)
    Motion:

```


我去openpose太难下了
要不用mediapipe先试试吧。也不行，我明天再来

我去，竟然可以了。10秒（60 frames per second）的视频用了1分47秒。
1个视频50秒来算，最多要10分钟。
210个视频，最多2100分钟=35h！
![picture 14](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240531-184332164.png)  
但今天也算有进展了，看来这个除了时间长，实现不难。

126维也拼好了

2. Aggregate to per unit
```
    126-dim to a 256-dim:
    calculates the mean vector and
    standard deviation vector (126 dimensions each) among body motions from the nstart-th to nend-th video frames of every unit and concatenates those vectors into the 252-dimensional unit-level vector of the k-th unit v˜(i)(k).
```
okey dokey！20帧拼一下，拼出来30个252维向量

2. Aggregate to per window
```
    In the third step, each method calculates a window-level
    feature vector. We use five different window lengths to
    aggregate unit-level feature vectors into a window-level one
    to see how many video frames are necessary to identify a
    dance genre. The window-level feature vector is obtained by
    concatenating all unit-level vectors v˜
    (i)
    (k) within a window
    of 2, 4, 8, 16, and 32 units, respectively. The window is then
    shifted by 1 unit to obtain the next window-level feature
    vector. We thus obtain five different window-level feature
    vectors.

```
easy!
![picture 15](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240601-095521405.png)  
30-4+1=27对的
- 模型baseline
好了我先去复习一下邱锡鹏循环神经网络和lstm部分。（不懂原理直接抄代码没啥意义的，要用实践印证理论，理论指导实践）




### Paper&Code
-----
### **舞蹈动作生成**

paper 

code
https://github.com/google-research/mint

trainer.py




好有趣都是舞步
![picture 5](../../.assets_IMG/dancepose-%E7%BB%BC%E8%BF%B0/IMG_20240528-151649964.png)  



