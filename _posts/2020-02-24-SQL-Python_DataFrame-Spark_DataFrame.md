# SQL-Python_DataFrame-Spark_DataFrame

就SQL，pandas dataframe，spark模型 之间进行比较。SQL十分灵活，语法简单，可以实现各种业务逻辑。如果只关注查询功能，可以按照不同的分类，进行一一对比。

但是sql虽然简单，但是逻辑语义未必清晰，例如，join的时候，同时取了两个表的值，这可能有以下几点：

1. 通过第二个表过滤第一个表。
2. 通过第一个表过滤第二个表。
3. 同时取表1和表2的属性。
4. 数据重复是需要小心的，有可能需要，也可能未必。



sql可以实现一次多个表的关联，一个sql可以处理非常多的逻辑。pandas一次不能关联多张表。这未必是坏事，因为这样炎症性更强。

sql何以在join上（hive除外）使用多种判断进行关联，例如大于，小于等。但是pandas不行。



## 字符串处理

在预处理过程中，split字符串，concat字符串是常用的技术。

### split

sql

```sql
select split(fullname, ' ')[0],split(fullname, ' ')[1];
```

pandas .str.split

```python
import pandas as pd
df = pd.DataFrame({'user':['Alina Li', 'Will Wang', 'Jessie Zhang']})
df[['first_name', 'last_name']] = df['user'].str.split(' ', expand=True)#expand will expand to multiple columns, 使用expand要非常小心，因为如果某一行列很多，就会造成结果有很多列
```



### replace

```sql
select replace(col, 'a', '');
```



pandas

```python
import pandas as pd
df = pd.DataFrame({'user':['Alina Li', 'Will Wang', 'Jessie Zhang']})
df['user'].str.replace(' ', ',')
```







## 信息融合

### 两表信息融合

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.merge.html

这也是ETL基础的一部分。

两个表都有主键（逻辑主键），两个表的信息进行融合。

t1:  id (key), cnt1,cnt2

t2: id(key), cnt3

取出所有来t1和t2的字段

#### Sql

```sql
--inner join
select t1.id, t1.cnt1, t1.cnt2, t2.cnt3
from t1
join t2 -- left join, right join, full join
on t1.id=t2.id;
--left join
select t1.id, t1.cnt1, t1.cnt2, t2.cnt3
from t1
left join t2
on t1.id=t2.id;
-- right join
select t2.id, t1.cnt1, t1.cnt2, t2.cnt3
from t1
right join t2
on t1.id=t2.id;
-- full join
select t1.id, t1.cnt1, t1.cnt2, t2.cnt3
from t1
full join t2
on t1.id=t2.id;

```



#### pandas dataframe
#### 数据准备

```python
import pandas as pd

df1 = pd.DataFrame({'id':['a','b', 'c'],'cnt1':[1,2,3], 'cnt2':[4,5,6]})
df2 = pd.DataFrame({'id':['b', 'c', 'd'],'cnt3':[7,8,9]})
```
| cnt1 | cnt2 |      |
| ---: | ---: | ---- |
|    a |    1 | 4    |
|    b |    2 | 5    |
|    c |    3 | 6    |

#### 使用index关联

可以直接使用index进行关联，前提是必须有index

```python
#set index, should remember inplace=True
df1.set_index('id', inplace=True)
df2.set_index('id', inplace=True)
df3 = df1.merge(df2, how='inner', left_index=True, right_index=True)
```

|   id | cnt1 | cnt2 | cnt3 |      |
| ---: | ---: | ---: | ---: | ---- |
|    0 |    b |    2 |    5 | 7    |
|    1 |    c |    3 |    6 | 8    |

#### 使用name关联

可以使用name进行关联， 名字可以是column的，也可以不是index的name

```python
#use name as column
#这样，不用非得set_index
df1.merge(df2, how='inner', on='id')
```

##### 左表主表

```python
df1.merge(df2, how='left', on='id')
```

|   id | cnt1 | cnt2 | cnt3 |      |
| ---: | ---: | ---: | ---: | ---- |
|    0 |    a |    1 |    4 | NaN  |
|    1 |    b |    2 |    5 | 7.0  |
|    2 |    c |    3 |    6 | 8.0  |

##### 右表主表

```python
df1.merge(df2, how='right', on='id')
```

|   id | cnt1 | cnt2 | cnt3 |      |
| ---: | ---: | ---: | ---: | ---- |
|    0 |    b |  2.0 |  5.0 | 7    |
|    1 |    c |  3.0 |  6.0 | 8    |
|    2 |    d |  NaN |  NaN | 9    |

##### full join

```python
df1.merge(df2, how='outer', on='id')
```

|   id | cnt1 | cnt2 | cnt3 |      |
| ---: | ---: | ---: | ---: | ---- |
|    0 |    a |  1.0 |  4.0 | NaN  |
|    1 |    b |  2.0 |  5.0 | 7.0  |
|    2 |    c |  3.0 |  6.0 | 8.0  |
|    3 |    d |  NaN |  NaN | 9.0  |

#### 字段选择

##### 列名相同问题

不同于sql， sql中直接使用as别名处理

```python
import pandas as pd
#下面两个表中，有列名相同
df1 = pd.DataFrame({'id':['a','b', 'c'],'cnt1':[1,2,3], 'cnt2':[4,5,6]})
df2 = pd.DataFrame({'id':['b', 'c', 'd'],'cnt1':[7,8,9]})
```


默认情况
```python
#没有管列名重复问题， 所以cnt1->cnt1_x,cnt1_y, 在两个表的重复字段上，同时添加了后缀
df1.merge(df2, how='inner', on='id')
```

|      |   id | cnt1_x | cnt2 | cnt1_y |
| ---: | ---: | -----: | ---: | -----: |
|    0 |    b |      2 |    5 |      7 |
|    1 |    c |      3 |    6 |      8 |

可以使用suffixes改变其中的一个, 但是想类似sql那样对列名进行alias，需要其他方法。例如在使用前就重命名。这种的好处是，

```python
#第一个表就没有后缀了，但是第二个表的重名字段有了后缀
df1.merge(df2, how='inner', on='id', suffixes=['', '_y'])
```

|      |   id | cnt1 | cnt2 | cnt1_y |
| ---: | ---: | ---: | ---: | -----: |
|    0 |    b |    2 |    5 |      7 |
|    1 |    c |    3 |    6 |      8 |



##### 选择部分字段

因为不是每个字段都需要，sql中，只要出现在SELECT中就好，这里需要在merge之前就选定字段。

使用sql

```sql
select t1.id, t1.cnt1 ,t2.cnt as cnt_y
from t1 join t2 on t1.id=t2.id;
```

相应的pandas.DataFrame做法：

```python
#事先选定，例如下面的，表1选 id, cnt1, 表2选id, cnt1
df1[['id', 'cnt1']].merge(df2[['id', 'cnt1']], how='inner', on='id', suffixes=['', '_y'])
```



## 聚合



聚合是统计的基础

### COUNT
##### row count

sql

```sql
--行数
SELECT COUNT(1)
FROM t;
```

pandas

```python
#这里不能用pd.count(), 如果你关心的是行数，就该用shape属性
import pandas as pd

df = pd.DataFrame({'c':['a', 'a', 'a', 'b', 'b', 'b'],'cnt':[1, 2, 3, 4, 5, 6]})
count_row = df.shape[0]  # gives number of row count
count_col = df.shape[1]  # gives number of col count
print(count_row)
print(count_col)
```



##### 统计字段值的数量

```python
import pandas as pd

df = pd.DataFrame({'c':['a', 'a', 'a', 'b', 'b', 'b'],'cnt':[1, 2, 3, 4, 5, 6]})
df.count() #统计所有字段的count， 返回Series
#c      6
#cnt    6
#dtype: int64

#如果只想知道其中一个，可以在之前或者之后选择字段
df['c'].count()#只计算想要的字段，性能更好
df.count()['c'] #算完后再取想要的
```



##### 按照字段分组COUNT
sql

```sql
SELECT c, COUNT(1)
FROM t
GROUP BY c;
```



pandas

```python
import pandas as pd
df = pd.DataFrame({'c':['a', 'a', 'a', 'b', 'b', 'b'],'cnt':[1, 2, 3, 4, 5, 6]})
df.groupby(['c']).count()
```



#####  pandas的value_counts(), 小甜点

SQL, 下面的sql只是针对两个字段的组合

```sql
SELECT a, b, COUNT(1)
FROM t
GROUP BY a, b;
```



pandas

```python
import pandas as pd
df = pd.DataFrame({'c':['a', 'a', 'a', 'b', 'b', 'b'], 'c0':['a0', 'a0', 'a1', 'b0', 'b1', 'b1'],'cnt':[1, 2, 3, 4, 5, 6]})
# 以下两种方式结果集相同，但是第一个保序，第二个不
df.groupby(['c', 'c0']).count() 
df.groupby(['c'])['c0'].value_counts()
```






### max, min

这些与count类似，可以计算某个字段的整个数据集max， min，也可以按照字段分组后，求每个分组的最大，最小等。

**SQL**

```python
select c, 
	count(1), 
  max(cnt) as max_cnt
  min(cnt) as min_cnt
from t;
```



**pandas**

```python
import pandas as pd
df = pd.DataFrame({'c':['a', 'a', 'a', 'b', 'b', 'b'],'cnt':[1, 2, 3, 4, 5, 6], 'cnt1':[2, 3, 4, 5, 6, 7]})
group = df[['c', 'cnt']].groupby(['c'])
df1 = group.max()
#对GroupBy对象，调用max方程后，返回的是一个DataFrame对象，index是groupby的columns,列是GroupBy的列
df1.index
#Index(['a', 'b'], dtype='object', name='c')
df2 = group.min()
#融合df1和df2
df1['min_cnt'] = group.min() #dataframe
```



### mean, median

sql

```sql
select c, 
	count(1), 
  avg(cnt) as mean_cnt
  min(cnt) as min_cnt
from t;
```



pandas

```python
df[['c', 'cnt']].groupby(['c']).mean()
df[['c', 'cnt']].groupby(['c']).median()
```



### 滚动加和

滚动加是比较常用的计算，例如月销售额，sql采用的方式比较直接。pandas提供了直接的方法-cum*, 

下面一张表，记录了用户的点击行为 

| 列名      | 描述           |
| --------- | -------------- |
| user      | 用户           |
| time      | 时间，精确到秒 |
| date_hour | 小时           |
| date      | 日期           |

#### 计算每天小时总的滚动点击次数

##### SQL， 关联-过滤-压缩

使用SQL的要点是，从逻辑上，量表按照最终要groupby的字段关联，对关联出的逻辑表，进一步过滤

例如下表：

| time                | cnt  |
| ------------------- | ---- |
| 2020-01-01 00:00:00 | 100  |
| 2020-01-01 01:00:00 | 50   |
| 2020-01-01 02:00:00 | 120  |

转换成下表
| time                |time| cnt  |
| ------------------- |----| ---- |
| 2020-01-01 00:00:00 |2020-01-01 00:00:00| 100  |
| 2020-01-01 01:00:00 |2020-01-01 00:00:00| 100   |
| 2020-01-01 01:00:00 |2020-01-01 01:00:00| 50  |
| 2020-01-01 02:00:00 |2020-01-01 00:00:00| 100  |
| 2020-01-01 02:00:00 |2020-01-01 01:00:00| 50  |
| 2020-01-01 02:00:00 |2020-01-01 02:00:00| 120  |
再按照第一列压缩（纵向）
| time                | cnt  |
| ------------------- | ---- |
| 2020-01-01 00:00:00 | 100  |
| 2020-01-01 01:00:00 | 150   |
| 2020-01-01 02:00:00 | 270  |

SQL code

```sql
select t1.date_hour, COUNT(1) AS cnt
from t t1
left join t t2
on t1.date=t2.date -- 关联
where t2.date_hour<=t1.date_hour -- 过滤
group by t1.date_hour; -- 压缩
```



##### 使用pandas， 先聚合到指定粒度，再调用cumsum， 要保证顺序

多种方法可以使用，也可以使用上面的**关联-过滤-压缩**法, 但是使用自带的**cum***方法更好

注意，pandas的 cumsum, cumcount, cummax, cummin等，是按照index进行滚动计算，因此index以及index的顺序非常重要。

如下创建一个DataFrame对象

```python
import pandas as pd
df = pd.DataFrame({'user':['a', 'a', 'a', 'b', 'b', 'b']
    ,'date_hour':['2020-01-01 01:00:00', '2020-01-01 00:00:00', '2020-01-01 02:00:00', '2020-01-01 01:00:00', '2020-01-01 02:00:00', '2020-01-01 03:00:00']
    ,'date':['2020-01-01', '2020-01-01', '2020-01-01', '2020-01-01', '2020-01-01', '2020-01-01']
    ,'cnt':[1, 2, 3, 4, 5, 6]})
```

|      | user |             date_hour |       date |  cnt |
| ---: | ---: | --------------------: | ---------: | ---: |
|    0 |    a | *2020-01-01 01:00:00* | 2020-01-01 |    1 |
|    1 |    a | *2020-01-01 00:00:00* | 2020-01-01 |    2 |
|    2 |    a |   2020-01-01 02:00:00 | 2020-01-01 |    3 |
|    3 |    b |   2020-01-01 01:00:00 | 2020-01-01 |    4 |
|    4 |    b |   2020-01-01 02:00:00 | 2020-01-01 |    5 |
|    5 |    b |   2020-01-01 03:00:00 | 2020-01-01 |    6 |

###### 错误做法

想要计算每天，每个小时的滚动加和，先得按照**date**分组，然后再组内按照index进行滚动加和，因此index的**顺序非常重要**。

```python
#首先，cumsum是按照index进行的，因此聚合的维度应当在其中
df.groupby('date').cumsum()
```

下面表格是按照默认index进行的叠加，即从0-5， 这不是想要的结果

|  cnt |      |
| --- | ---- |
|    0 | 1    |
|    1 | 3    |
|    2 | 6    |
|    3 | 10   |
|    4 | 15   |
|    5 | 21   |

```python
#设置date_hour为index
df.set_index(['date_hour'], inplace=True)
df.groupby('date').cumsum()
```

**这也不是想要的**

|  date_hour  |  cnt |
| ------------------| --- |
| 2020-01-01 01:00:00 |    1 |
| 2020-01-01 00:00:00 |    3 |
| 2020-01-01 02:00:00 |    6 |
| 2020-01-01 01:00:00 |   10 |
| 2020-01-01 02:00:00 |   15 |
| 2020-01-01 03:00:00 |   21 |

```python
#设置date_hour为index, sort_index
df.set_index(['date_hour'], inplace=True)
df.sort_index(inplace=True)
df.groupby('date').cumsum()
```
**依然不是想要的**，因为相同的index没有merge

|                 date_hour |  cnt    |
| ------------------ | ---- |
| 2020-01-01 00:00:00 |    2 |
| 2020-01-01 01:00:00 |    3 |
| 2020-01-01 01:00:00 |    7 |
| 2020-01-01 02:00:00 |   10 |
| 2020-01-01 02:00:00 |   15 |
| 2020-01-01 03:00:00 |   21 |

###### 正确做法

**先聚合到最终粒度，再用cumsum**这种cumsum, 需要先对要聚合的力度进行一次聚合

```python
df_group = df.groupby(['date', 'date_hour'])['cnt'].sum()#粒度需要先到聚合的力度
df_group.groupby(['date']).cumsum()#再通过cumsum计算

#date        date_hour          
#2020-01-01  2020-01-01 00:00:00     2
#            2020-01-01 01:00:00     7
#            2020-01-01 02:00:00    15
#            2020-01-01 03:00:00    21
```



再添加一个user维度也可以

```python
df_group = df.groupby(['user', 'date', 'date_hour'])['cnt'].sum()
df_group.groupby(['user','date']).cumsum()

#user  date        date_hour          
#a     2020-01-01  2020-01-01 00:00:00     2
#                  2020-01-01 01:00:00     3
#                  2020-01-01 02:00:00     6
#b     2020-01-01  2020-01-01 01:00:00     4
#                  2020-01-01 02:00:00     9
#                  2020-01-01 03:00:00    15
```





### pandas.GroupBy函数和sql对比

按照使用频率排序

|       | Desc       |      | Desc                                                  |
| ----- | ---------- | ---- | ----------------------------------------------------- |
| count | count(col) |      | Compute count of group, **excluding missing values**. |
|       |            |      |                                                       |








###   GroupBy的一下

在sql中，分组只是逻辑上的模型。但是在pandas和spark中分组是对象。

##### pandas





## 同表的前后两行差（时序）

### 相邻两行差

事件排序， 假设表中记录了user和事件的时间，计算时间差是个常用的需求

| user | Timestamp |
| ---- | --------- |
| a    | 1         |
| a    | 2         |
| b    | 1         |
| b    | 3         |

#### sql

```python
#spark
data = [('a', 1),('a', 2),('a', 2),('a', 3),('b', 6),('b',5),('b',8)]
rdd = spark.sparkContext.parallelize(data)
df = rdd.toDF(['user', 'timestamp'])
df.createOrReplaceTempView('t')
```

```sql
--使用windows function lag
SELECT user, timestamp- lag(timestamp) over (partition by user order by timestamp asc) as last_stamp 
FROM t;

-- 不使用lead, lag function
with cte as 
(SELECT user, timestamp, row_number() over (partition by user order by timestamp asc) as rk 
FROM t)
SELECT t1.user,  t1.timestamp, (t1.timestamp-t2.timestamp) as diff
FROM cte t1
LEFT JOIN cte t2
ON t1.user=t2.user AND t1.rk=t2.rk+1;
```



#### pandas diff

```python
import pandas as pd
df = pd.DataFrame({'name':['a', 'a', 'a', 'a', 'b', 'b', 'b'], 'timestamp':[1, 2, 2, 3, 6, 5, 8]})
df = df.sort_values(by=['name','timestamp'])#sort非常重要， 这是组内cum*, diff等的顺序
df['time_diff'] = df.groupby('name')['timestamp'].diff()
```
结果如下：

| name | timestamp | time_diff |      |
| ---: | --------: | --------: | ---- |
|    0 |         a |         1 | NaN  |
|    1 |         a |         2 | 1.0  |
|    2 |         a |         2 | 0.0  |
|    3 |         a |         3 | 1.0  |
|    5 |         b |         5 | NaN  |
|    4 |         b |         6 | 1.0  |
|    6 |         b |         8 | 2.0  |

*If you have a lot of data and you want to save some time (this can be about 10 times faster depends on your data size) you can skip the **groupby** and just do the **diff** after sorting the data and then deleting the first row of each person which is not relevant.*

```python
df = df.sort_values(by=['name','timestamp'])
df['time_diff'] = df['timestamp'].diff()
df.loc[df.name != df.name.shift(), 'time_diff'] = None
```





### shift, 同比，环比







## pivot & unpivot(stack & unstack ) aka 行列互转

| user | activity | time     |
| ---- | -------- | -------- |
| a    | show     | 20200101 |
| a    | click     | 20200101 |

### unpivot | unstack, 行转列, features engineer用的比较多

| user | show | click |
| ---- | ---- | ----- |
| a    | 1    | 0     |
| a    | 0    | 1     |

**SQL**, 横向列扩展-纵向压缩

```sql
select user
,if(activity='show', 1, 0) as show
,if(activity='click', 1, 0) as click
from t
```

**pandas**， unstack函数

```python
import pandas as pd
df = pd.DataFrame({'user':['a', 'a', 'b', 'b', 'c'], 'activity':['show', 'click', 'show', 'click', 'show']})
df1 = df.groupby(['user', 'activity'])['user'].count()
df2 = df1.unstack(fill_value=0)
```

| user | click | show |
| ---: | ----: | ---: |
|    a |     1 |    1 |
|    b |     1 |    1 |
|    c |     0 |    1 |





## 问题：

pandas的聚合怎么进行？











