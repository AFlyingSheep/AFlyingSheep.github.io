---
title: 数据预处理与特征工程
date: 2022-02-9 15:37:02
tags:
- AI
categories: 
- AI
description: 数据预处理与特征工程作为数据挖掘流程中不可或缺的一部分。本文详细介绍了数据处理流程，包括去量纲化、填补缺失值等方法，并用python来具体实现该流程。
top_img: 
cover: /image/shuju/cover.png
---

# 数据预处理与特征工程



## 课纲概述

- 数据挖掘五大流程

  1. 获取数据

  2. 数据预处理 - 使数据匹配与模型

  3. 特征工程 - 将原始数据转换为更能代表特征的数据

     - 目的：降低计算成本、提高模型上限。

     - 提取特征
     - 创造特征

  4. 建模

  5. 上线

## 数据预处理

### 数据的无量纲化

将不同规格的数据转换到统一规格。可以是线性的、非线性的。

#### 数据归一化

```python
from sklearn.preprocessing import MinMaxScaler
data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]

#不太熟悉numpy的小伙伴，能够判断data的结构吗？
#如果换成表是什么样子？
import pandas as pd
pd.DataFrame(data)

#实现归一化
scaler = MinMaxScaler() #实例化
scaler = scaler.fit(data) #fit,在这里本质是生成min(x)和max(x)
result = scaler.transform(data) #通过接口导出结果
result
"""
array([[0.  , 0.  ],
       [0.25, 0.25],
       [0.5 , 0.5 ],
       [1.  , 1.  ]])
"""

result_ = scaler.fit_transform(data) #训练和导出结果一步达成
result_
"""
array([[0.  , 0.  ],
       [0.25, 0.25],
       [0.5 , 0.5 ],
       [1.  , 1.  ]])
"""

scaler.inverse_transform(result) #将归一化后的结果逆转
"""
array([[ 5.  ,  5.  ],
       [ 6.25,  6.25],
       [ 7.5 ,  7.5 ],
       [10.  , 10.  ]])
"""

#使用MinMaxScaler的参数feature_range实现将数据归一化到[0,1]以外的范围中
data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]

scaler = MinMaxScaler(feature_range=[5,10]) #依然实例化
result = scaler.fit_transform(data) #fit_transform一步导出结果
result
"""
array([[ 5.  ,  5.  ],
       [ 6.25,  6.25],
       [ 7.5 ,  7.5 ],
       [10.  , 10.  ]])
"""

# 当X中的特征数量非常多的时候，fit会报错并表示：数据量太大了我计算不了，此时使用partial_fit作为训练接口
scaler = scaler.partial_fit(data)
```

#### 数据标准化

**数据标准化(Standardization，又称Z-score normalization)**：当数据(x)按均值(μ)中心化后，再按标准差(σ)缩放，数据就会服从为均值为0，方差为1的正态分布（即标准正态分布），公式如下：

![标准化](/image/shuju/biaozhunhua.png)

```python
from sklearn.preprocessing import StandardScaler
data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]


scaler = StandardScaler() #实例化
scaler.fit(data) #fit，本质是生成均值和方差


scaler.mean_ #查看均值的属性mean_
# array([-0.125,  9.   ])
scaler.var_ #查看方差的属性var_
# array([ 0.546875, 35.      ])


x_std = scaler.transform(data) #通过接口导出结果

x_std.mean() #导出的结果是一个数组，用mean()查看均值
# 0.0
x_std.std() #用std()查看方差
#1.0

scaler.fit_transform(data) #使用fit_transform(data)一步达成结果
"""
array([[-1.18321596, -1.18321596],
       [-0.50709255, -0.50709255],
       [ 0.16903085,  0.16903085],
       [ 1.52127766,  1.52127766]])
"""

scaler.inverse_transform(x_std) #使用inverse_transform逆转标准化
"""
array([[-1. ,  2. ],
       [-0.5,  6. ],
       [ 0. , 10. ],
       [ 1. , 18. ]])
"""

```



对于StandardScaler和MinMaxScaler来说，空值NaN会被当做是**缺失值**

- 在fit的时候会被忽略
- 在transform时保持NaN的状态显示

输入的X为特征矩阵，一般只接受**二维以上**数组，一维导入会报错。



#### 标准化和归一化的选择

- 大多数机器学习算法中，会选择StandardScaler进行特征缩放，因为MinMaxScaler对异常值非常敏感
- 在PCA，聚类，逻辑回归，支持向量机，神经网络这些算法中，StandardScaler往往是最好的选择

MinMaxScaler在不涉及距离度量、梯度、协方差计算以及数据需要被压缩到特定区间时使用广泛，比如**数字图像处理中量化像素强度**时，都会使用MinMaxScaler将数据压缩于[0,1]区间之中。

建议先试试看StandardScaler，效果不好换MinMaxScaler。

除了StandardScaler和MinMaxScaler之外，sklearn中也提供了各种其他缩放处理（中心化只需要一个pandas广播一下减去某个数就好了，因此sklearn不提供任何中心化功能）

- 在希望压缩数据，却不影响数据的稀疏性时（不影响矩阵中取值为0的个数时），我们会使用MaxAbsScaler（只压缩，不中心化）
- 在异常值多，噪声非常大时，我们可能会选用分位数来无量纲化，此时使用RobustScaler

![无量纲](/image/shuju/wulianggang.png)

### 缺失值处理

**机器学习和数据挖掘中所使用的数据，永远不可能是完美的**。很多特征，对于分析和建模来说意义非凡，但对于实际收集数据的人却不是如此，因此数据挖掘之中，常常会有重要的字段缺失值很多，但又不能舍弃字段的情况。因此，数据预处理中非常重要的一项就是**处理缺失值**。

```python
# 使用impute.SimpleImputer填补缺失值
class sklearn.impute.SimpleImputer (
	missing_values=nan, 
	strategy='mean', 
	fill_value=None, 
	verbose=0,
	copy=True
	)
```

- **missing_values** 
  - 告诉SimpleImputer缺失值是什么样的
- **strategy** 
  - 填补策略，默认采用均值
    - 输入"mean"使用均值填补（仅对数值型特征可用）
    - 输入"median"用中值填补（仅对数值型特征可用）
    - 输入"most_frequent"用众数填补（对数值型和字符型特征都可用）
    - 输入"constant"表示请参考参数"fill_value"中的值（对数值型和字符型特征都可用）

- **fill_value**
  - 当参数startegy为"constant"的时候可用，可输入字符串或数字表示要填充的值，常用0
- **copy**
  - 默认为True，将创建特征矩阵的副本，反之则会将缺失值填补到原本的特征矩阵中去

```python
import pandas as pd
#index_col=0是因为原数据中第1列本就是索引
data = pd.read_csv(r"..\datasets\Narrativedata.csv",index_col=0)
data.head()

# 查看数据集的总体信息
data.info()
# 由运行结果可知Age和Embarked有缺失值
"""
<class 'pandas.core.frame.DataFrame'>
Int64Index: 891 entries, 0 to 890
Data columns (total 4 columns):
 #   Column    Non-Null Count  Dtype  
---  ------    --------------  -----  
 0   Age       714 non-null    float64
 1   Sex       891 non-null    object 
 2   Embarked  889 non-null    object 
 3   Survived  891 non-null    object 
dtypes: float64(1), object(3)
memory usage: 34.8+ KB
"""

# 查看数据
Age = data.loc[:,"Age"].values.reshape(-1,1) #sklearn当中特征矩阵必须是二维
Age[:20]
"""
array([[22.],
       [38.],
       [26.],
       [35.],
       [35.],
       [nan],
       [54.],
       [ 2.],
       [27.],
       [14.]])
"""
Age = data.loc[:,"Age"].values.reshape(-1,1).shape
"""
(-1, 1)
"""

#填补年龄, 分别用均值、中位数、0填补
from sklearn.impute import SimpleImputer
imp_mean = SimpleImputer() #实例化,默认均值填补
imp_median = SimpleImputer(strategy="median") #用中位数填补
imp_0 = SimpleImputer(strategy="constant",fill_value=0) #用0填补

#fit_transform一步完成调取结果
imp_mean = imp_mean.fit_transform(Age) #均值填补
imp_median = imp_median.fit_transform(Age) #中值填补
imp_0 = imp_0.fit_transform(Age) # 使用0填补

imp_mean[:20] # 查看用均值填补后的前20条数据

imp_median[:10] # 查看用中值填补后的前20条数据

imp_0[:10] # 查看用0填补后的前20条数据

#在这里我们使用中位数填补Age
data.loc[:,"Age"] = imp_median
#data.info()

#使用众数填补Embarked
Embarked = data.loc[:,"Embarked"].values.reshape(-1,1)
imp_mode = SimpleImputer(strategy = "most_frequent")
data.loc[:,"Embarked"] = imp_mode.fit_transform(Embarked)

data.info() #
# 由结果可知填补已经完成了
"""
<class 'pandas.core.frame.DataFrame'>
Int64Index: 891 entries, 0 to 890
Data columns (total 4 columns):
 #   Column    Non-Null Count  Dtype  
---  ------    --------------  -----  
 0   Age       891 non-null    float64
 1   Sex       891 non-null    object 
 2   Embarked  891 non-null    object 
 3   Survived  891 non-null    object 
dtypes: float64(1), object(3)
memory usage: 34.8+ KB
"""

# data.head(20) #显示填补后的前20条数据
```

**我们同样可以使用Numpy和Pandas进行填补**

```python
import pandas as pd
data = pd.read_csv(r"..\datasets\Narrativedata.csv",index_col=0)
data.head()
data.loc[:,"Age"] = data.loc[:,"Age"].fillna(data.loc[:,"Age"].median())
#.fillna 在DataFrame里面直接进行填补
data.dropna(axis=0,inplace=True)

#.dropna(axis=0)删除所有有缺失值的行，.dropna(axis=1)删除所有有缺失值的列
#参数inplace，为True表示在原数据集上进行修改，为False表示生成一个复制对象，不修改原数据，默认False
```

### 处理分类型数据

在现实中，许多标签和特征在数据收集完毕的时候，都不是以数字来表现的：

- 付费方式可能包含 [“支付宝”，“现金”，“微信”]
- 学历的取值可以是 [“小学”，“初中”，“高中”，“大学”]

在这种情况下，为了让数据适应算法和库，我们必须将数据进行**编码**，也就是要**将文字型数据转换为数值型**。

#### preprocessing.LabelEncoder 

标签专用，将分类转换为分类数值

```python
from sklearn.preprocessing import LabelEncoder
y = data.iloc[:,-1] #要输入的是标签，不是特征矩阵，所以允许一维
```

```python
#进行编码
le = LabelEncoder() #实例化
le = le.fit(y) #导入数据
label = le.transform(y)   #transform接口调取结果
#label  #查看获取的结果label
#le.classes_ #属性.classes_查看标签中究竟有多少类别
"""
array(['No', 'Unknown', 'Yes'], dtype=object)
"""

#le.fit_transform(y) #也可以直接fit_transform一步到位,但是不能查看属性classes_

#le.inverse_transform(label) #使用inverse_transform可以逆转

data.iloc[:,-1] = label #让标签等于我们运行出来的结果
#data.head()
```

也可以一步完成

```python
from sklearn.preprocessing import LabelEncoder
data.iloc[:,-1] = LabelEncoder().fit_transform(data.iloc[:,-1])
```

#### preprocessing.OrdinalEncoder

特征专用，将分类特征转换为分类数值

```python
from sklearn.preprocessing import OrdinalEncoder
data_ = data.copy()
#data_.head()

#接口categories_对应LabelEncoder的接口classes_，一模一样的功能
#OrdinalEncoder().fit(data_.iloc[:,1:-1]).categories_

data_.iloc[:,1:-1] = OrdinalEncoder().fit_transform(data_.iloc[:,1:-1])
#data_.head()
```

#### preprocessing.OneHotEncoder 

独热编码，创建哑变量

我们刚才已经用OrdinalEncoder把分类变量Sex和Embarked都转换成数字对应的类别了。在舱门Embarked这一列中，我们使用 [0,1,2] 代表了三个不同的舱门，然而这种转换是正确的吗？

我们来思考三种不同性质的分类数据：

1. 舱门（S，C，Q）

   三种取值S，C，Q是相互独立的，彼此之间完全没有联系，表达的是 S≠C≠Q 的概念。这是**名义变量**

2. 学历（小学，初中，高中）

   三种取值不是完全独立的，我们可以明显看出，在性质上可以有高中>初中>小学这样的联系，学历有高低，但是学历取值之间却不是可以计算的，我们不能说小学 + 某个取值 = 初中。这是**有序变量**

3. 体重（>45kg，>90kg，>135kg）

   各个取值之间有联系，且是可以互相计算的，比如135kg - 45kg = 90kg，分类之间可以通过数学计算互相转换。这是**有距变量**。

然而在对特征进行编码的时候，这三种分类数据都会被我们转换为 [0,1,2]，这三个数字在算法看来，是连续且可以计算的，这三个数字相互不等，有大小，并且有着可以相加相乘的联系。所以算法会把舱门，学历这样的分类特征，都误会成是体重这样的分类特征。我们把分类转换成数字的时候，**忽略了数字中自带的数学性质**，所以给算法传达了一些不准确的信息，这会影响我们的建模。

OrdinalEncoder可以用来处理**有序变量**，但对于**名义变量**，我们只有使用**哑变量**的方式来处理，才能够尽量向算法传达最准确的信息：

三个取值是没有可计算性质的，是“有你就没有我”的不等概念。

因此我们需要使用独热编码，将性别与舱门都转换为哑变量。

```python
#data.head()
from sklearn.preprocessing import OneHotEncoder
X = data.iloc[:,1:-1]
enc = OneHotEncoder(categories='auto').fit(X)
result = enc.transform(X).toarray()
#result

#依然可以直接一步到位,但为了给大家展示模型属性,所以还是写成了三步
#OneHotEncoder(categories='auto').fit_transform(X).toarray()

#依然可以还原
#pd.DataFrame(enc.inverse_transform(result))

#返回每一个稀疏矩阵列的名字
enc.get_feature_names()

#result
#result.shape

#axis=1,表示跨行进行合并，也就是将量表左右相连，如果是axis=0，就是将量表上下相连
newdata = pd.concat([data,pd.DataFrame(result)],axis=1)
newdata.head()
newdata.drop(["Sex","Embarked"],axis=1,inplace=True)
newdata.columns = ["Age","Survived","Female","Male","Embarked_C","Embarked_Q","Embarked_S"]
newdata.head()
```

#### 总结

![zongjie](/image/shuju/zongjie.png)

#### 数据类型以及常用的统计量

![shuju](/image/shuju/shujuleixing.png)



### 连续性特征性特征

#### sklearn.preprocessing.Binarizer

根据阈值将数据二值化（将特征值设置为0或1），用于处理连续型变量。大于阈值的值映射为1，而小于或等于阈值的值映射为0。默认阈值为0时，特征中所有的正值都映射到1。二值化是对文本计数数据的常见操作，分析人员可以决定仅考虑某种现象的存在与否。它还可以用作考虑布尔随机变量的估计器的预处理步骤（例如，使用贝叶斯设置中的伯努利分布建模）。

```python
#将年龄二值化
data_2 = data.copy()
#data_2

from sklearn.preprocessing import Binarizer
X = data_2.iloc[:,0].values.reshape(-1,1) #类为特征专用，所以不能使用一维数组

transformer = Binarizer(threshold=30).fit_transform(X)
#transformer
```

#### preprocessing.KBinsDiscretize

这是将连续型变量划分为分类变量的类，能够将连续型变量排序后按顺序分箱后编码。

总共包含三个重要参数：

![KBins](/image/shuju/KBins.png)

```python
from sklearn.preprocessing import KBinsDiscretizer
X = data.iloc[:,0].values.reshape(-1,1)

est = KBinsDiscretizer(n_bins=3, encode='ordinal', strategy='uniform')
est.fit_transform(X)

#查看转换后分的箱：变成了一列中的三箱
set(est.fit_transform(X).ravel())

est = KBinsDiscretizer(n_bins=3, encode='onehot', strategy='uniform')
#查看转换后分的箱：变成了哑变量
est.fit_transform(X).toarray()
```

