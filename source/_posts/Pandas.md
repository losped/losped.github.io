---
title: Pandas
date: 2019-10-02 14:13:02
tags: Py
categories: Py
---


## 一. 数据填充

##### 1.apply()
apply()对多列数据计算处理
```
df14['Bulges数量'] = df14.apply(lambda row: row['Bulges数量'] if row['Bulges提报'] > 0 else 0, axis=1)
```
**关于axis**
简单的来记就是axis=0代表往跨行（down)，而axis=1代表跨列（across)
##### 2.applymap()
##### 3.map()
总的来说就是apply()是一种让函数作用于列或者行操作，applymap()是一种让函数作用于DataFrame每一个元素的操作，而map是一种让函数作用于Series每一个元素的操作**

##### 4.eval()
python数据科学生态环境的强大力量在Numpy和Pandas的基础之上，并通过直观的语法将基本操作转化为c语言：在Numpy里是向量化/广播运算，在pandas里是分组型的运算。虽然这些抽象功能可以简洁高效的解决很多问题，但是他们经常需要创建临时对象，这样会占用很大的计算时间和内存。
Pandas为了解决性能问题，引入了eval()和query()函数，他们可以让用户直接运行C语言速度的操作，不需要费力的配置中间数组，它们都依赖于Numexpr程序包

##### 5.agg()
多用于groupby处理后的列聚合
```
df1[COMMON_VALUE['SUMMARY_CLOUMNS1']] = df2.groupby(groupby_list).agg({'ID':'count', 'Score':np.mean,'Score1':np.mean,'Score2':np.mean,'Score3':np.mean,'Score4':np.mean,'Fail3':np.mean,'jug6_copy':np.mean,'SKUnum':np.mean,'A5h1':np.mean,'A5h2':np.mean,'jug1':'sum','jug1_copy':np.mean,'jug2':'sum','jug2_copy':np.mean,'jug3':'sum','jug3_copy':np.mean,'jug5':'sum','jug5_copy':np.mean,'jug4':'sum','jug4_copy':np.mean})
```

**DataFrame.eval()实现列间运算**

- 新增列
```
        df2.eval('Doritos或FF = Doritos + FF', inplace=True)
        df2.eval('Doritos或FF或Baked = Doritos + FF + Baked', inplace=True)
```

- 使用局部变量
```
         column_mean = df.mean(1)
         result = df.eval('A + @column_mean')
```

**DataFrame.query()**
query()方法和eval()方法一样是基于DataFrame列的计算代数式。对于过滤的操作，可以使用query()方法。

**pd.eval()支持的运算**
- 算术运算
```
         df1,df2,df3,df4,df5 = (pd.DataFrame(np.random.randint(0,1000,(100,3))),for i in range(5))
         result = pd.eval('-df1 * df2 / (df3 + df4) -df5')
```
- 比较运算
```
result = pd.eval('df1 < df2 <= df3 != df4')
```
- 位运算
```
result = pd.eval('(df1<0.5) & (df2<0.5) | (df3<df4)')
```
- 对象属性和索引
```
result = pd.eval('df2.T[0] + df3.iloc[1]')
```

## 二. 数据表合并

##### 1.merage()
pandas提供了一个类似于关系数据库的连接(join)操作的方法<Strong>merage</Strong>,可以根据一个或多个键将不同DataFrame中的行连接起来

```
#coding=utf-8
from pandas import Series,DataFrame,merge
import numpy as np
data=DataFrame([{"id":0,"name":'lxh',"age":20,"cp":'lm'},{"id":1,"name":'xiao',"age":40,"cp":'ly'},{"id":2,"name":'hua',"age":4,"cp":'yry'},{"id":3,"name":'be',"age":70,"cp":'old'}])
data1=DataFrame([{"id":100,"name":'lxh','cs':10},{"id":101,"name":'xiao','cs':40},{"id":102,"name":'hua2','cs':50}])
data2=DataFrame([{"id":0,"name":'lxh','cs':10},{"id":101,"name":'xiao','cs':40},{"id":102,"name":'hua2','cs':50}])

print "单个列名做为内链接的连接键\r\n",merge(data,data1,on="name",suffixes=('_a','_b'))
print "多列名做为内链接的连接键\r\n",merge(data,data2,on=("name","id"))
print '不指定on则以两个DataFrame的列名交集做为连接键\r\n',merge(data,data2) #这里使用了id与name

#使用右边的DataFrame的行索引做为连接键
##设置行索引名称
indexed_data1=data1.set_index("name")
print "使用右边的DataFrame的行索引做为连接键\r\n",merge(data,indexed_data1,left_on='name',right_index=True)


print '左外连接\r\n',merge(data,data1,on="name",how="left",suffixes=('_a','_b'))
print '左外连接1\r\n',merge(data1,data,on="name",how="left")
print '右外连接\r\n',merge(data,data1,on="name",how="right")
data3=DataFrame([{"mid":0,"mname":'lxh','cs':10},{"mid":101,"mname":'xiao','cs':40},{"mid":102,"mname":'hua2','cs':50}])

#当左右两个DataFrame的列名不同，当又想做为连接键时可以使用left_on与right_on来指定连接键
print "使用left_on与right_on来指定列名字不同的连接键\r\n",merge(data,data3,left_on=["name","id"],right_on=["mname","mid"])
```

##### 2.join()
提供了一个简便的方法用于将两个DataFrame中的不同的列索引合并成为一个DataFrame。
其中参数的意义与merge方法基本相同,**只是join方法默认为左外连接how=left**
```
#coding=utf-8
from pandas import Series,DataFrame,merge

data=DataFrame([{"id":0,"name":'lxh',"age":20,"cp":'lm'},{"id":1,"name":'xiao',"age":40,"cp":'ly'},{"id":2,"name":'hua',"age":4,"cp":'yry'},{"id":3,"name":'be',"age":70,"cp":'old'}],index=['a','b','c','d'])
data1=DataFrame([{"sex":0},{"sex":1},{"sex":2}],index=['a','b','e'])

print '使用默认的左连接\r\n',data.join(data1)  #这里可以看出自动屏蔽了data中没有的index=e 那一行的数据
print '使用右连接\r\n',data.join(data1,how="right") #这里出自动屏蔽了data1中没有index=c,d的那行数据；等价于data1.join(data)
print '使用内连接\r\n',data.join(data1,how='inner')
print '使用全外连接\r\n',data.join(data1,how='outer')
```

##### 3.concat()
concat方法相当于数据库中的全连接(UNION ALL),可以指定按某个轴进行连接,也可以指定连接的方式join(outer,inner 只有这两种)。
**与数据库不同的是concat不会去重，要达到去重的效果可以使用drop_duplicates方法**

```
#coding=utf-8
from pandas import Series,DataFrame,concat

df1 = DataFrame({'city': ['Chicago', 'San Francisco', 'New York City'], 'rank': range(1, 4)})
df2 = DataFrame({'city': ['Chicago', 'Boston', 'Los Angeles'], 'rank': [1, 4, 5]})
print '按轴进行内连接\r\n',concat([df1,df2],join="inner",axis=1)
print '进行外连接并指定keys(行索引)\r\n',concat([df1,df2],keys=['a','b']) #这里有重复的数据
print '去重后\r\n',concat([df1,df2],ignore_index=True).drop_duplicates()
```

## 三. 数据过滤

- 保留某些数据样本
```
   df = df[df['creativeID']<=10000]
```

- 删除某些数据样本
```
   df = df[(df['appID'].isin([278,382]))&(df['appPlatform'].isin([2]))]
```
