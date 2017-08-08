---
title: k-近邻算法
date: 2017-08-08 01:27:18
tags:
	- "Machine Learning"
---
> ***注意：部分浏览器会拦截MathJax生成数学公式，如出现无法显示公式，请允许浏览器加载脚本***

## k-近邻算法

### 1. KNN算法思想
1. 计算已知数据集中的某个已知`类别`的点与当前点的距离；
2. 按照距离依次递增排序；
3. 选取与当前距离最小的k个点；
4. 确定前k个点，所属类别出现的频率；
5. 返回前k个点出现频率最高的类别作为当前点的预测类别；

距离公式:
$$d=\sqrt{\sum_j^n (x^j_i-x^j)^2} \tag{2-1}$$  
其中$x^j_i$表示第$i$个输入向量的第$j$个分量

下面是分类代码实现
```python
import numpy as np

def classify(in_x, data_set, labels, k):
    """
    进行分类
    :param in_x: 输入分量x
    :param data_set: 已知类别的数据集
    :param labels:  对应的数据集的label
    :param k: 迭代次数
    :return:
    """
    size = data_set.shape[0]  # 训练数据集的个数

    # np.title第二个参数是数组形式，
    # 其中第一个数字代表复制的次数，
    # 后面的数字代表在哪个维度上复制，
    # 如果只有一个值，则代表维度扩张复制

    # ======= 求距离 ========
    mat_dif = np.tile(in_x, (size, 1)) - data_set  # 2维， [[x1-x],[x2-x]]
    mat_pow = mat_dif ** 2
    dis = np.sqrt(mat_pow.sum(axis=1))  # axis是对哪一个维度求和运算, 无需开根号运算
    dis_arg = dis.argsort()
    # ======= 距离计算完成且有序 ===
    cls_count = {}
    for i in xrange(k):
        vote_label = labels[dis_arg[i]]  # 获得第i小的排序结果的分类
        cls_count[vote_label] = cls_count.get(vote_label, 0) + 1

    # 获取cls中值最大的key
    st = sorted(cls_count.iteritems(), key=lambda item: item[1], reverse=True)
    return st[0][0]
```

### 2. 数据归一化
数据存在数值上的差别，通常情况下需要进行归一化处理，即将数值映射到0~1或者-1~1之间的数值。

0~1归一化:   
$$val = \frac{val - min}{max-min} \tag{2-2}$$

-1~1归一化:  
$$val = \frac{val-avg}{max-min} \tag{2-3}$$

代码实现
```python
def auto_norm(data_set):
    """
    数据归一化
    :param data_set: 待归一化的数据集
    :return: 归一化后的数据集，每一列的取值范围，每一列的最小值
    """
    # 获取每一列最大值和最小值
    d_max, d_min = data_set.max(0), data_set.min(0)
    d_range = d_max - d_min

    mat_min = np.tile(d_min, (data_set.sharpe[0], 1))
    mat_max = np.tile(d_range, (data_set.sharpe[0], 1))

    mat_norm = (data_set - mat_min) / mat_max
    return mat_norm, d_range, d_min
```
