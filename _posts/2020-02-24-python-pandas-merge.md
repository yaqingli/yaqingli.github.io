# Python-pandas-merge

## 使用场景

1. 数据查询
2. 数据关联



## 数据关联

在sql中，使用关键词"join"对两个数据集合进行关联。

分为下列一些情景

1. 两个表有相同的键，关联也按照相同的建，例如都是id，或者都是日期。需要将这两个具有**相同粒度**的表进行融合，具有两个集合的属性（表字段）。（**常用**）
2. 一个表的属性是另一个表的主键（逻辑主键也行）。事实-维度表，就类似这种关系。（**常用**）
3. 两个表关联字段都不是主键，这种会造成多对多的关系，应当小心这种情景。(**不常用**)



**建议**： 一般数据集最好有主键，而pandas dataframe可以设置index，当使用index的时候关联变得比较方便.



### dataframe关联的方式

#### 使用index

假设需要关联的字段是index字段

```python
import pandas as pd

df1 = pd.DataFrame({'cnt1':[1,2,3], 'cnt2':[4,5,6]}, index=['a','b', 'c'])
df2 = pd.DataFrame({'cnt3':[7,8,9]}, index=['b', 'c', 'd'])
df1['cnt3'] = df2
```

结果如下，可以看出，这是一个left join，而且按照index进行关联。

**优点**：简单

**缺点**：只能是left join

|      | cnt1 | cnt2 | cnt3 |
| ---: | ---: | ---: | ---: |
|    a |    1 |    4 |  NaN |
|    b |    2 |    5 |  7.0 |
|    c |    3 |    6 |  8.0 |



#### 使用merge(类似sql-join)

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.merge.html

这个类似sql中的join

DataFrame.merge(self, right, how='inner', on=None, left_on=None, right_on=None, left_index=False, right_index=False, sort=False, suffixes=('_x', '_y'), copy=True, indicator=False, validate=None)

**作用**：Merge **DataFrame** or **named Series objects** with a database-style join.

**注意**： The join is done on **columns** or **indexes**. If joining columns on columns, the DataFrame indexes *will be ignored*. Otherwise if joining indexes on indexes or indexes on a column or columns, the index will be passed on.

merge

|        | index     | Column        |
| ------ | --------- | ------------- |
| index  | index保持 | index保持     |
| Column | index保持 | **index忽略** |

？？如果想要保持怎么办？

right: 右边的表

**how**: 如何连接，**{‘left’, ‘right’, ‘outer’, ‘inner’},**

**on**: 在哪个字段上， Column or index level names to join on. These must be found in both DataFrames. If on is None and not merging on indexes then this defaults to the intersection of the columns in both DataFrames.

**left_on**：**label or list, or array-like**. Column or index level names to join on in the left DataFrame. Can also be an array or list of arrays of the length of the left DataFrame. These arrays are treated as if they are columns.

**right_on**: same as left_on

**left_index**：**bool, default False**. Use the index from the left DataFrame as the join key(s). If it is a MultiIndex, the number of keys in the other DataFrame (either the index or a number of columns) must match the number of levels.

**right_index**: same as right_on

**sort**: **bool**, 是否需要按照关联字段排序

**suffixes**: **tuple**

**copy** : **bool**

**indicator**: **bool**, 告诉你左表和右表，关联的时候，哪边有数据。还是sql灵活。

```python
import pandas as pd

df1 = pd.DataFrame({'cnt1':[1,2,3], 'cnt2':[4,5,6]}, index=['a','b', 'c'])
df2 = pd.DataFrame({'cnt3':[7,8,9]}, index=['b', 'c', 'd'])
df1.merge(df2, how='left',indicator=True)

```

| cnt1 | cnt2 | cnt3 | _merge |           |
| ---: | ---: | ---: | -----: | --------- |
|    0 |    1 |    4 |    NaN | left_only |
|    1 |    2 |    5 |    7.0 | both      |
|    2 |    3 |    6 |    8.0 | both      |



**validate** :**str**，验证是否符合下列的规则。这个比sql强，sql无法直接给出这样的答案。



If specified, checks if merge is of specified type.

- “one_to_one” or “1:1”: check if merge keys are unique in both left and right datasets.
- “one_to_many” or “1:m”: check if merge keys are unique in left dataset.
- “many_to_one” or “m:1”: check if merge keys are unique in right dataset.
- “many_to_many” or “m:m”: allowed, but does not result in checks.



