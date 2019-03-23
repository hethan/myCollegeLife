<font size = 3> 

[TOC]
**import pandas as pd导入pandas**  
# 5.1 pandas数据结构介绍
## 5.1.1 Series
**Series是一种一维的数组型对象，包含一个值序列（与NumPy中的类型相似），并且包含了数据标签，称为索引(index)。**  
```python
>>> obj = pd.Series([4,7,-5,3])
>>> obj
 0 4
 1 7
 2 -5
 3 3
 dtype int64
```
交互式环境中Series的字符串表示，索引在左边，值在右边。当不为数据指定索引时，默认生成的索引是0到N-1.可以分别通过values属性和index属性分别获得Series对象的值和索引：
```python
>>> obj.values
 array([4, 7, -5, 3])
>>> obj.index
 RangeIndex(start = 0, stop = 4, step = 1)
>>> obj2 = pd.Series(obj.values, index = ['a','b','c','d'])
>>> obj2['c']
 -5
```
可以认为Series是一个长度固定且有序的字典。  

如果已经有数据包含在Python字典中，可以使用字典生成一个Series:
```python
>>> sdata = {'Ohii':35000, 'Texas':71000, 'Oregon':16000, 'Utah':5000}
>>> obj = pd.Series(sdata)
>>> obj
 Ohio   35000
 Oregon 16000
 Texas  71000
 Utah    5000
```
当将字典传递给Series构造函数时，产生的Series的索引是排序好的字典键。**可以将字典键按照个人意愿顺序传递给构造函数，从而使生成的Series的索引顺序符合意愿**

```python
>>> data = {'a':0, 'b':1, 'c':2, 'd':3}
>>> obj = pd.Series(data, ['a','c','f'])
>>> obj
 a    0.0
 c    2.0
 f    NaN
 dtype: float64
```
因为'f'键不存在于data字典中，它的对应值则是NaN(not a number)，这是pandas中标记缺失值或NA值的方式。因为'b','d'并不在['a','c','f']中，因此被排除在外。

pandas中用isnull和notnull函数来检查缺失数据，**且isnull和notnull也是Series的实例方法。**
```python
>>> pd.isnull(obj)
 a False
 c False
 f True
```

Series对象自身和其索引都有name属性，这个特性与pandas其他重要功能集成在一起。
```python
>>> obj.name = 'values'
>>> obj.index.name = 'index'
```

Series的索引可以通过按位置赋值的方式进行修改
```python
>>> obj.index = ['c','a','d','f']
```

## 5.1.2 DataFrame
DataFrame表示的是矩阵的数据表，它包含已排序的列集合，每一列可以是不同的值类型（数值、字符串、布尔值等）。DataFrame既有行索引也有列索引，它可以被视为一个共享相同索引的Series的字典。

**虽然DataFrame是二维的，但可以利用分层索引在DataFrame中展现更高维度的数据。分层索引是pandas中一种更为高级的数据处理特性**  

```pyhton
>>> data = {'state':['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada', 'Nevada'],
            'Year':[2000, 2001, 2002, 2001, 2002, 2003],
            'pop':[1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}
>>> frame = pd.DataFrame(data, index = np.arange(1, 7, 1))
>>> frame
     state  Year  pop
 1    Ohio  2000  1.5
 2    Ohio  2001  1.7
 3    Ohio  2002  3.6
 4  Nevada  2001  2.4
 5  Nevada  2002  2.9
 6  Nevada  2003  3.2
```

**如果使用的是Jupyter notebook, pandas的DataFrame对象将会展示为一个对浏览器更为友好的HTML表格。**  

对于大型DataFrame，head方法将会只选出头部的五行（实例方法）

如果指定了列的顺序，DataFrame的列将会按照指定顺序排列(存在实例方法)。
```python
>>> pd.DataFrame(data, columns=['Year', 'state', 'pop'])
   Year   state  pop
 0  2000    Ohio  1.5
 1  2001    Ohio  1.7
 2  2002    Ohio  3.6
 3  2001  Nevada  2.4
 4  2002  Nevada  2.9
 5  2003  Nevada  3.2

```
如果传递的列不包含在字典中，将会在结果中出现缺失值。

DataFrame中的一列，可以按照字典型标记或属性那样检索为Series

行可以通过位置或特殊属性loc进行选取
```python
>>> frame.loc[3]
 year     Ohio
 state    2002
 pop       3.6
 Name: 3, dtype: object
```

列的引用是可以修改的，当将列表或数组赋值给一个列时，值的长度必须和DataFrame的长度相匹配
```python
>>> frmae['year'] = 2000
>>> frame
    state  year  pop
1    Ohio  2000  1.5
2    Ohio  2000  1.7
3    Ohio  2000  3.6
4  Nevada  2000  2.4
5  Nevada  2000  2.9
6  Nevada  2000  3.2
>>> frame['year'] = np.arange(6)+2000
>>> frame
    state  year  pop
1    Ohio  2000  1.5
2    Ohio  2001  1.7
3    Ohio  2002  3.6
4  Nevada  2003  2.4
5  Nevada  2004  2.9
6  Nevada  2005  3.2
```

**如果将Series赋值给一列是，Series的索引将会按照DataFrame的索引重新排列，并在空缺的地方填充缺失值。如果被赋值的列不存在，则会生成一个新的列。del关键字可以像在字典中那样对DataFrame删除列。**  
```python
>>> del frame['year']
```

从DataFrame中选取的列是数据的视图，而不是拷贝。因此，对Series的修改会映射到DataFrame中。如果需要复制，则应当显式地使用Series的copy方法。  

另一种常用的数据形式是包含字典的嵌套字典，如果嵌套字典被赋值给DataFrame，Pandas会将字典的键作为列，将内部字典的键作为索引。并且可以使用类似NumPy的语法对DataFrame进行转置（掉换行和列）
```python
>>> data = {'a':{1000:2.4, 1002:2.3}, 'b':{1002:1, 123:4.2}}
>>> frame1 = pd.DataFrame(data)
>>> frame1
        a    b
123   NaN  4.2
1000  2.4  NaN
1002  2.3  1.0
```
**内部字典的键被联合、排序后形成了结果的索引。如果已经显示指明索引的话，内部字典的键将并不会不被排序。**  

如果DataFrame构造函数传递的对象拥有name属性，则这些name属性也会被显示。和Series类似，DataFrame的values属性会将包含在DataFrame中的数据以二维ndarray的形式返回。

**如果DataFrame的列是不同的dtype，则values的dtype会自动选择适合所有列的类型**  

|            类型            |                             注释                             |
| :------------------------: | :----------------------------------------------------------: |
|         2D ndarray         |              数据的矩阵，行和列的标签是可选参数              |
| 数组、列表和元组构成的字典 |     每个序列成为DataFrame的一列，所有的序列必须长度相等      |
|   NumPy结构化/记录化数组   |                     与数组构成的字典一致                     |
|      Series构成的字典      | 每一个内部字典成为一列，每个Series的索引联合起来形成结果行索引，也可以显示地传递索引 |
|       字典构成的字典       |      每一个内部字典成为一列，键联合起来形成结果的行索引      |
|   字典或Series构成的列表   | 列表中的一个元素形成DataFrame的一行，字典键或Series索引联合起来形成DataFrame的一行，字典键或Series索引联合起来形成DataFrame的列标签 |
|    列表或元组构成的列表    |                    与2D ndarray的情况一致                    |
|       其他DataFrame        |         如果不显示传递索引，则会使用原DataFrame索引          |
|     NumPy MaskedArray      | 与2D ndarray的情况类似，但隐蔽值会在结果DataFrame中成为NA/缺失值 |
## 5.1.3 索引对象
pandas中的索引对象是用于存储轴标签和其他元数据的（例如轴名称或标签）。在构造Series或DataFrame时，所使用的任意数组或标签序列都可以在内部转换为索引对象。

**索引对象是不可变的，因此用户无法修改索引对象。不变性使得在多种数据结构中分享索引对象更为安全。**  

与Python集合不同，pandas索引对象可以包含重复标签，根据重复标签进行筛选，会选取所有重复标签对应的数据。

每个索引都有一些集合逻辑的方法和属性，这些方法和属性解决了关于它所包含的数据的其他常见问题。

**一些索引对象的方法和属性**  

|     方法     |                       描述                       |
| :----------: | :----------------------------------------------: |
|    append    | 将额外的索引对象粘贴到原索引后，产生一个新的索引 |
|  difference  |                计算两个索引的差集                |
| intersecion  |                计算两个索引的交集                |
|    union     |                计算两个索引的并集                |
|     isin     |    计算表示每一个值是否在传值容器中的布尔数组    |
|    delete    |        将位置i的元素删除，并产生新的索引         |
|     drop     |      根据参数删除指定索引值，并产生新的索引      |
|    insert    |         在位置i插入元素，并产生新的索引          |
| is_monotonic |            如果索引序列递增则返回True            |
|  is_unique   |            如果索引序列唯一则返回True            |
|    unique    |               计算索引的唯一值序列               |

# 5.2 基本功能
## 5.2.1 重建索引
## 5.2.2 轴向上删除条目
## 5.2.3 索引、选择与过滤
## 5.2.4 整数索引
## 5.2.5 算术和数据对齐
## 5.2.6 函数应用和映射
## 5.2.7 排序和排名
## 5.2.8 含有重复把标签的轴索引
# 5.3 描述性统计的概述和计算
## 5.3.1 相关性和协方差
## 5.3.2 唯一值、计数和成员属性