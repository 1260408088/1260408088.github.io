---
title: python学习笔记
date: 2020-02-28 16:36:16
categories: [python]
tags: [python,其他语言]
---

# python笔记

## python的字符串

​	1. 字符串类型有独特的索引机制，可以正着数也可以倒着数；

![zifusuoyin](zifusuoyin.PNG)

<!--more-->

2. 从字符串中取单个字符串或者字串的方式分别为:

``` python
#单个字符串,拿去最后一个字符
TempStr[-1] 
#拿取字符子串,从第一个到第三个字符（但是不包括第三个）
TempStr[1:3]
#从头拿到最后一个但是不包括最后一个
TempStr[0:-1] 
```

![](zifusuoyin2.PNG)

​	字符串中的转义符

![](zhuanyiofu.PNG)

3. python中表示字符串的方法有很多，使用单引号、双引号、三引号都可以的；字符串中有单引号，外层就是用双引号，字符串中有使用双引号的，外层就包裹单引号，如果字符串中有双引号也有单引号、那就使用三引号进行包裹;


``` python
a='''一切都像是,刚睡醒一样'''
print("朱自清春的第一句{}".format(a))
```

4.nuicode编码切换

![](nuicode.PNG)

5. 常见的字符串处理方法

   ![](method1.PNG)

![](method2.PNG)

![](method3.PNG)

6. 槽的格式的控制

   ![](caogeshi.PNG)

具体使用

![](demo.PNG)

7. 字符串进度条

   ``` python
   import time
   scale = 50
   print("执行开始".center(scale//2,"-")) # 求商然后取整，位于字符的中间位置了
   start = time.perf_counter()
   for i in range(scale+1):
       a="*"*i;
       b="."*(scale-i)
       c=(i/scale)*100
       dur=time.perf_counter()-start
       print("\r{:^3.0f}%[{}{}]{:.2f}s".format(c,a,b,dur),end='') # \r移动光标到最开始的位置
       time.sleep(0.1)
   print("\n"+"执行结束".center(scale//2,'-'))
   ```

   输出结果：

   ``` python
   -----------执行开始----------
   100%[**************************************************]5.05s
   -----------执行结束----------
   ```

   8.国际与国内BMI指数计算

   ``` python
   height, weight = eval(input("请输入身高(米)和体重(公斤)[使用逗号隔开]："))
   bmi = weight / pow(height, 2)
   print("BMI 数值为：{:.2f}".format(bmi))
   who, nat = "", ""
   if bmi < 18.5:
       who, nat = "偏瘦", "偏瘦"
   elif 18.5 <= bmi < 24:
       who, nat = "正常", "正常"
   elif 24 <= bmi < 25:
       who, nat = "正常", "偏胖"
   elif 25 <= bmi < 28:
       who, nat = "偏胖", "偏胖"
   elif 28 <= bmi < 30:
       who, nat = "偏胖", "肥胖"
   else:
       who, nat = "肥胖", "肥胖"
   print("BMI指标为：国际{},国内{}".format(who,nat))
   ```

   ## python常用库

   1. random随机数

   计算机中不会真正的产生随机数。ptyhon中的随机数会设置一个seed值，seed值设置相同出现的随机数也就相同，未设置seed的值，会默认为当前时间。

   ``` python
   import random
   random.seed(10)
   print(random.random())
   random.seed(10)
   print(random.random())
   ```

   两次输出结果一样都是 0.5714025946899135

![](random1.PNG)

![](random2.PNG)

![](random3.PNG)

圆周率的计算（蒙特卡洛方式计算）：

``` python
from random import random
from time import perf_counter
Darts =10000*10000
hits =0.0
start =perf_counter()
for i in range(1,Darts+1):
    x,y=random(),random()
    dist=pow(x**2 +y**2,0.5)
    if dist <=1.0:
        hits=hits+1
pi=4*(hits/Darts)
print("圆周率为：{}".format(pi))
print("运行时间{:.5f}s".format(perf_counter()-start))
```

## python变量

python中在函数中有局部变量和全局变量相同时，要在函数中使用全局变量，需要使用保留字 <span style="color:red">global</span>

``` python
import turtle


def drawLine(draw):
    drawGap()
    turtle.pendown() if (draw) else turtle.penup()
    turtle.fd(40)
    drawGap()
    turtle.right(90)


def drawDigit(digit):
    drawLine(True) if (digit) in [2, 3, 4, 5, 6, 8, 9] else drawLine(False)
    drawLine(True) if (digit) in [0, 1, 3, 4, 5, 6, 7, 8, 9] else drawLine(False)
    drawLine(True) if (digit) in [0, 2, 3, 5, 6, 8] else drawLine(False)
    drawLine(True) if (digit) in [0, 2, 6, 8] else drawLine(False)
    turtle.left(90)
    drawLine(True) if (digit) in [0, 4, 5, 6, 8, 9] else drawLine(False)
    drawLine(True) if (digit) in [0, 2, 3, 5, 6, 7, 8, 9] else drawLine(False)
    drawLine(True) if (digit) in [0, 1, 2, 3, 4, 7, 8, 9] else drawLine(False)
    turtle.left(180)
    turtle.penup()
    turtle.fd(20)


def drawDate(date):
    for i in date:
        drawDigit(eval(i))


def main():
    turtle.setup(800, 350, 200, 200)
    turtle.penup()
    turtle.fd(-300)
    turtle.pensize(5)
    drawDate("20200221")
    turtle.hideturtle()
    turtle.done()


main()
# 增加小间隔更像数码管
def drawGap():
    turtle.penup()
    turtle.fd(5)

```

![](date1.PNG)

## python的第三方库的安装

​	使用pip进行安装，有时候会有pip在cmd下不是命令的报错，需要将python的安装目录下的Script文件夹加入到Path的环境变量之中即可。

``` shell
pip install pyinstaller
```

pyinstaller这个第三方库可以将python的程序打包为直接可执行的程序，比如exe之类的。打包命令为：

``` python
pyinstaller -F <文件名.py>
```

相关参数如下：

![](install.PNG)

加图标的打包命令：

``` shell
pyinstaller -i <图标名称.ico> -F <文件名.py>
```

## python递归

基本上都一样，这记一个科恩曲线，表示一下：

``` python
import turtle
def koch(size,n):
    if n==0:
        turtle.fd(size)
    else:
        for angle in [0,60,-120,60]:
            turtle.left(angle)
            koch(size/3,n-1)
def main():
    turtle.setup(600,600)
    turtle.penup()
    turtle.goto(-200,100)
    turtle.pendown()
    turtle.pensize(2)
    level=5
    koch(400,level)
    turtle.right(120)
    koch(400,level)
    turtle.right(120)
    koch(400, level)
    turtle.hideturtle()
    turtle.done()
main()
```

![](xuehua.PNG)

## python集合

python的集合没有重复的元素，使用{}或者set()建立集合

``` python
A={"python",123,("python",123)}
print(A) #{'python', ('python', 123), 123}
B=set("pypy123") 
print(B) # {'2', '3', '1', 'p', 'y'}
```

集合的操作符

![](jiaochabingbu.PNG)

```  返回True+
S <= T 或者 S < T 返回True/False，判断S和T的子集关系
S >= T 或者 S > T 返回True/False, 判断S和T的包含关系
```

![](zengqiangcaozuofu.PNG)

集合的操作方法

![](jihe1.PNG)

![](jihe2.PNG)

数组去重：

``` python
ls = ["p","p","y","y",123]
s = set(ls) # 利用集合无重复的特点
it = list(s) # 将集合转换回数组
```

## python序列

``` python
ls = ["123",123,"lsss"]
```

元组类型的方法：

![](xulie1.PNG)

![](xuelie2.PNG)

## python元组类型

元组类型一旦被创建就不可以被修改了，可以用圆括号表示也可以不用，python的函数中可以返回两个值

``` python
def func():
    return 1,2
```

在内部会被解析成为一个元组类型返回出去;因为元组的不可改变的特性，可以将序列改变为元组起到保护数据的作用:

``` python
lt = tuole(ls) # 将序列变为元组
ls = list(lt)  # 将序列变为元组 
```

## python 字典类型

字典类型是映射的体现，字典是键值对的集合，键值对之间无序，采用大括号{}和dict()创建，键值对用冒号：表示

``` pyth
{<键1>:<值1>,<键2>:<值2>}
```

