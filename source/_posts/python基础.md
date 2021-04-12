---
title: python基础
date: 2021-04-12 10:36:41
categories: [python]
tags: [python,其他语言]
---

# 最简单的语法

## 1.输入与输出

``` python
name = input('please input you name') 
print('name,', name)
```

## 2.变量

``` python
10_000_000_000和10000000000表示的一样
'I\'m \"OK\"!' 符号转义 I'm "OK"!
'''...''' 表示多行字符串，java13也有这个功能了
变量本身类型不固定的语言称之为动态语言，与之对应的是静态语言。静态语言在定义变量时必须指定变量类型，如果赋值的时候类型不匹配，就会报错。例如Java是静态语言
print(11 // 3) 地板除法，向下取整数

-----------------------
py执行的推荐注释
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
-----------------------
```

## 3.格式化占位符

```python
'Hi, %s, you have $%d.' % ('Michael', 1000000) # 格式%s、%d
'Hi, Michael, you have $1000000.' # 被替换后
```

| 占位符 |   替换内容   |
| :----: | :----------: |
|   %d   |     整数     |
|   %f   |    浮点数    |
|   %s   |    字符串    |
|   %x   | 十六进制整数 |

**Demo**

``` python
# 格式
print('%2d-%02d' % (3, 1))
print('%.2f' % 3.1415926)
# 输出
3-01  补零（两位整数）
3.14  保留两位小数
--------------------------------------------------------------------------
# tip：有些时候，字符串里面的%是一个普通字符怎么办？这个时候就需要转义，用%%来表示一个%
'growth rate: %d %%' % 7
```

**format()** 方式替换

​	另一种格式化字符串的方法是使用字符串的`format()`方法，它会用传入的参数依次替换字符串内的占位符`{0}`、`{1}`……。

``` python
'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
'Hello, 小明, 成绩提升了 17.1%'
# format函数根据顺序进行替换
```

**f-string**

​	最后一种格式化字符串的方法是使用以`f`开头的字符串，称之为`f-string`，它和普通字符串不同之处在于，字符串如果包含`{xxx}`，就会以对应的变量替换

``` python
r = 2.5
s = 3.14 * r ** 2
print(f'The area of a circle with radius {r} is {s:.2f}')
# 输出
The area of a circle with radius 2.5 is 19.62
```

## 4.数组

常见的**list数组**

```python
# 获得数组的长度
classmates = ['Michael', 'Bob', 'Tracy']
length = len(classmates)
print(length)
# 按位置拿取数组
print(classmates[0])
# 追加元素，并输出最后一个
classmates.append('Adam')
print(classmates[-1])
# 按位置插入元素，之前的第0个变成第一个
classmates.insert(0, 'Jack')
print(classmates[-5])
---------------------
# 移出制定位置元素
one = classmates.pop(1)
# 指定位置的元素重新赋值
classmates[1] = 'Sarah'
print(classmates[1])
# list的嵌套
s= ['python', 'java', ['asp', 'php'], 'scheme']
print(len(s))
# ----------------------------------------------
p = ['asp', 'php']
s = ['python', 'java', p, 'scheme']
print(len(s))
# 要拿到'php'可以写p[1]或者s[2][1]，因此s可以看成是一个二维数组，类似的还有三维、四维……数组
```

**tuple**

``` python
classmates = ('Michael', 'Bob', 'Tracy')
t = (1, 2)
t = (1,) # ython在显示只有1个元素的tuple时，也会加一个逗号,，以免你误解成数学计算意义上的括号
# -------------------------------------------------------
t = ('a', 'b', ['A', 'B']) # 不可变是指向不可变，而不是内部指向以后，list内部的元素可以变
t[2][0] = 'X'
t[2][1] = 'Y'
```

tuple和list非常类似，但是tuple一旦初始化就不能修改，比如同样是列出同学的名字现在，classmates这个tuple不能变了，它也没有append()，insert()这样的方法。其他获取元素的方法和list是一样的，你可以正常地使用classmates[0]，classmates[-1]，但不能赋值成另外的元素。

## 5.条件判断

``` python
age = 20
if age >= 6:
    print('teenager')
elif age >= 18:
    print('adult')
else:
    print('kid')
```

## 6.循环

**for** 

``` python
sum = 0
for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
    sum = sum + x
print(sum)
```

**while**

``` python
# break 
n = 1
while n <= 100:
    if n > 10: # 当n = 11时，条件满足，执行break语句
        break # break语句会结束当前循环
    print(n)
    n = n + 1
print('END')
# continue 
n = 0
while n < 10:
    n = n + 1
    if n % 2 == 0: # 如果n是偶数，执行continue语句
        continue # continue语句会直接继续下一轮循环，后续的print()语句不会执行
    print(n)
```

## 7.集合

**dict**

类似于java的map，key-value的形式

``` python
d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
print(d['Michael'])
# 直接添加，新key-val，重复赋值，也是会给覆盖掉的
d['Jack'] = 90
d['Jack']
d['Jack'] = 88
d['Jack']
print(d['Jack'])
```

判断key存在与否

``` python
# 使用 in
'Thomas' in d
# dict提供的get()方法，如果key不存在，可以返回None，或者自己指定的value
d.get('Thomas')
d.get('Thomas', -1)
```

删除制定的元素

``` python
d.pop('Bob')
```

**set**

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。要创建一个set，需要提供一个**list**作为<span style="font-size:24px;color:red">输入集合</span>

``` python
s = set([1, 2, 3])
print(s)
-------------------
s = set([1, 1, 2, 2, 3, 3]) #会去重
-------------------
# 重复添加
s.add(4)
s.add(4)
# 删除元素
s.remove(4)
--------------------
# 进行数学上的交集并集运算
s1 = set([1, 2, 3])
s2 = set([2, 3, 4])
s3 = s1 & s2
s4 = s1 | s2
print(s3)
print(s4)
输出：{2, 3}
{1, 2, 3, 4}
----------------------
```

