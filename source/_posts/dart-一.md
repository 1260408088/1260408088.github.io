---
title: dart(一)
date: 2019-08-13 10:47:10
categories: dart
tags: [dart,new language]
---

# Hello world

​	安装dart的环境就不赘述了，无脑安装就可以了，安装过程中好像需要梯子（vpn），我装的时候失败好多次，我的梯子不能用了，准备不装了的时候，莫名其妙的装好了。迷の操作。惯例，一门新的语言必须要传统一下。

``` dart
main(){
  print("hello world");
}
```

直接就输出了 “hello world”，没有那么多繁杂的语法。

<!--more-->

# 变量类型

## 定义变量

``` dart
main(){
  var age=24; // 使用var来定义变量，会自动的识别类型
  int Age=24; // 直接使用int来定义也可以
  age="25"; // （报错）类型检验自动识别类型以后，第一次什么类型之后就是什么类型了
  print(age);
}
```

## 定义常量

``` dart
main(){
  // 赋值一次后就不能改变了，称之为常量,（感觉我说的就是废话）
  final age=23;
  const Age=25;
  print(Age);
  print(age);
}
```

final与const的区别：final 可以开始不赋值 只能赋一次 ; 而final不仅有const的编译时常量的特性，最重要的它是运行时常量，并且final是惰性初始化，即在运行时第一次使用前才初始化。

``` dart
main(){
  final time=new DateTime.now();
  const Time=new DateTime.now(); // 报错
  print(time);
}
```

## 字符拼接

``` dart
main(){
  String name="kign";
  String age= "23";
  print(name +":"+ age);
  print("${name}+:+${age}");
}
```

## list的定义

``` dart
main() {
  List l1 = ["aaa", "bbb", "ccc"];
  print(l1[1]);
  List l2 = [
    {'age': 12},
    {"age": 34}
  ];
  print(l2[0]["age"]);
  var l3 = new List<String>();
  l3.add("value");
  l3.add("page");
  print(l3[1]);
}
```

## Map的定义

``` dart
main() {
 Map m1={"name":"张三","age":24,"work":["程序员","外卖员"]};
 print(m1["work"]);
 var p=new Map();
 p["name"]="李四";
 p["age"]=23;
 p["work"]=["程序员","外卖员"];
 print(p["age"]);
}
```

## 类型判断

``` dart
main() {
  var age = 25;
  if (age is int) { // is关键字
    print("int类型数据");
  } else {
    print("其它类型数据");
  }
}
```

## ？？运算符

​	判断当前是否为空，为空则为？？后的值，不为空则为当前赋值的值

``` dart
main() {
  var a;
  var b = a ?? 10;
  print(b);

  var c = 22;
  var d = c ?? 10;
  print(d);
}
```

输出：

``` dart
10
22
```

## ??=运算符

​	判断当前变量是否为空，为空则将运算符后的数值赋值给当变量

``` dart
main() {
  var a;
  a??=23;
  print(a);
    
  var b=10;
  b??=23;
  print(b);
}
```

输出：

``` dart
23
10
```

## 类型转换

``` dart
main() {
  var a="12"; 
  var b=double.parse(a);
  print(b);

  var c="13";
  print(c.toString());
}
```

# 集合循环

## list

``` dart
List myList=['香蕉','苹果','西瓜'];
// 最普通的循环
for(var i=0;i<myList.length;i++){
    print(myList[i]);
}
// forEach 
for(var item in myList){
    print(item);
}
// forEach 
myList.forEach((value){
    print("$value");
});
// A数组扩大二倍赋值给B数组
List myList=[1,3,4];
List newList=new List();
for(var i=0;i<myList.length;i++){
    newList.add(myList[i]*2);
}
print(newList);
// A数组扩大二倍给B数组
List myList=[1,3,4];      
var newList=myList.map((value){
    return value*2;
});
print(newList.toList());

```

## list内容的判断

``` dart
// 一
List myList=[1,3,4,5,7,8,9];
var newList=myList.where((value){ // 将数组中大于5的给予一个新数组
    return value>5;
});
print(newList.toList());
// 二
List myList=[1,3,4,5,7,8,9];
var f=myList.any((value){   //只要集合里面有满足条件的就返回true
    return value>5;
});
print(f);
// 三
 List myList=[1,3,4,5,7,8,9];
 var f=myList.every((value){   //每一个都满足条件返回true否则返回false
     return value>5;
 });
 print(f);
```

## Set的循环

``` dart
main() {
   var s=new Set();
   s.addAll([1,222,333]);
   s.forEach((value)=>print(value));
}
```

## Map的循环

``` dart
main() {
  Map person={
    "name":"张三",
    "age":20
  };
  person.forEach((key,value){
    print("$key---$value");
  });
}
```

# 方法定义

## 可选参数方法

``` dart
main() {
  String msg = printUserInfo("king");
  // String msg = printUserInfo("king",23);
  print(msg);
}
String printUserInfo(String username, [int age]) {
  //行参
  if (age != null) {
    return "姓名:$username---年龄:$age";
  }
  return "姓名:$username---年龄保密";
}

```

输出：

``` dart
姓名:king---年龄保密
// 姓名:king---年龄:23
```

## 默认参数方法

``` dart
main() {
  print(printUserInfo('张三'));
  print(printUserInfo('小李', '女'));
  print(printUserInfo('小李', '女', 30));
}
String printUserInfo(String username, [String sex = '男', int age]) {
  //行参
  if (age != null) {
    return "姓名:$username---性别:$sex--年龄:$age";
  }
  return "姓名:$username---性别:$sex--年龄保密";
}

```

输出：

``` dart
姓名:张三---性别:男--年龄保密
姓名:小李---性别:女--年龄保密
姓名:小李---性别:女--年龄:30
```

## 命名参数的方法

``` dart
main() {
  String printUserInfo(String username, {int age, String sex = '男'}) {
    //行参
    if (age != null) {
      return "姓名:$username---性别:$sex--年龄:$age";
    }
    return "姓名:$username---性别:$sex--年龄保密";
  }
  print(printUserInfo('张三', age: 20, sex: '未知'));
}
```

输出：

``` dart
姓名:张三---性别:未知--年龄:20
```

将方法当作参数

``` dart
main() {
  //方法
  fn1(){
    print('fn1');
  }
  //方法
  fn2(fn){
    fn();
  }
  //调用fn2这个方法 把fn1这个方法当做参数传入
  fn2(fn1);
}
```

输出：

``` dart
fn1
```

##  箭头函数

### 数组循环

``` dart
main(){
  List list=['苹果','香蕉','西瓜'];
  list.forEach((value)=>print(value));
  list.forEach((value)=>{
      print(value) // 不能写分号，只能写一行
});
}
```

### 数组计算

``` dart
main(){
  List list=[4,1,2,3,4];
  var newList=list.map((value)=>value>2?value*2:value);
  print(newList.toList());
}
```

## 闭包问题

### 闭包：

全局变量特点:    全局变量常驻内存、全局变量污染全局
局部变量的特点：  不常驻内存会被垃圾机制回收、不会污染全局  

想实现的功能：

1. 常驻内存        
2. 不污染全局   

  产生了闭包,闭包可以解决这个问题.....  

  闭包: 函数嵌套函数, 内部函数会调用外部函数的变量或参数, 变量或参数不会被系统回收(不会释放内存)

  闭包的写法： 函数嵌套函数，并return 里面的函数，这样就形成了闭包。



### 全局变量

``` dart
/*全局变量*/
var a=123;
void main() {
  print(a);
  fn() {
    a++;
    print(a);
  }
  fn();
  fn();
  fn();
}
```

输出：

``` dart
123
124
125
126
```

### 局部变量

``` dart
void main() {
  printInfo() {
    var myNum = 123;
    myNum++;
    print(myNum);
  }

  printInfo();
  printInfo();
  printInfo();
}
```

输出：

``` dart
124
124
124
```

### 闭包

``` dart
main() {
  fn() {
    var a = 123; /*不会污染全局   常驻内存*/
    return () {
      a++;
      print(a);
    };
  }

  var b = fn();
  b();
  b();
  b();
}
```

输出：

``` dart
124
125
126
```

## Dart的类

### 构造函数

​	dart的构造函数不像java那样可以重载，它有它自己的一套

``` dart
class Person{
  String name;
  int age;
  //默认构造函数的简写
  Person(this.name,this.age);

  Person.now(){
    print('我是命名构造函数');
  }

  Person.setInfo(String name,int age){
    this.name=name;
    this.age=age;
  }

  void printInfo(){
    print("${this.name}----${this.age}");
  }
}
```

### 类抽离成为模块

​	在此目录下的dart文件

``` dart
lib/Person.dart
```

引入所需要的类中

``` dart
import 'lib/Person.dart';

void main(){

  Person p1=new Person.setInfo('李四1',30);
  p1.printInfo(); 

}
```

### Dart中的私有方法和私有属性

​	私有方法与私有属性需要用_来修饰，但需要提取出来，否则无法达到私有的效果

``` dart
class Animal{
  String _name;   //私有属性
  int age; 
  //默认构造函数的简写
  Animal(this._name,this.age);

  void printInfo(){   
    print("${this._name}----${this.age}");
  }

  String getName(){ 
    return this._name;
  } 
  void _run(){
    print('这是一个私有方法');
  }

  execRun(){
    this._run();  //类里面方法的相互调用
  }
}
```

### get与set的使用

``` dart
 class Rect{
   num height;
   num width;
   Rect(this.height,this.width);
   get area{
     return this.height*this.width;
   }
 }

 void main(){
   Rect r=new Rect(10,2);
   print("面积:${r.area}");      //注意调用直接通过访问属性的方式访问area
 }

```

``` dart
class Rect{
  num height;
  num width;
  Rect(this.height,this.width);
  get area{
    return this.height*this.width;
  }
  set areaHeight(value){
    this.height=value;
  }
}
void main(){
  Rect r=new Rect(10,4);
  // print("面积:${r.area()}");   
  r.areaHeight=6;
  print(r.area);
}
```

## 连缀操作符

``` dart
main() {
  Person p1 = new Person('张三1', 20);

  p1
    ..name = "李四"
    ..age = 30
    ..printInfo();
}

class Person {
  String name;
  num age;

  Person(this.name, this.age);

  void printInfo() {
    print("${this.name}---${this.age}");
  }
}

```

输出：

``` dart
李四---30
```

## Dart的继承

​	其他未写的继承的特性与java区别不大

``` dart
class Person {
  String name;
  num age;
  Person(this.name,this.age);
  Person.xxx(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");
  }
}

class Web extends Person{
  String sex;
  Web(String name, num age,String sex) : super.xxx(name, age){
    this.sex=sex;
  }
  run(){
    print("${this.name}---${this.age}--${this.sex}");
  }

}

main(){
  Web w=new Web('张三', 12,"男");

  w.printInfo();

  w.run();
}

```

输出:

```dart
张三---12
张三---12--男
```

## Dart的抽象类与接口

​	dart中的抽象类大致上与java并无太大的区别，也就不赘述了。需要注意的是java与dart的向上转型，后子类的方法无法调用了，需要类型的转换。

​	dart是没有interface这个关键词的，接口的定义用abstract关键字修饰的。其他用法与java类似，也不赘述。

## dart的mixins属性

``` dart
/*
mixins的中文意思是混入，就是在类中混入其他功能。

在Dart中可以使用mixins实现类似多继承的功能


因为mixins使用的条件，随着Dart版本一直在变，这里讲的是Dart2.x中使用mixins的条件：

  1、作为mixins的类只能继承自Object，不能继承其他类
  2、作为mixins的类不能有构造函数
  3、一个类可以mixins多个mixins类
  4、mixins绝不是继承，也不是接口，而是一种全新的特性
*/

class Person{
  String name;
  num age;
  Person(this.name,this.age);
  printInfo(){
    print('${this.name}----${this.age}');
  }
  void run(){
    print("Person Run");
  }
}

class A {
  String info="this is A";
  void printA(){
    print("A");
  }
  void run(){
    print("A Run");
  }
}

class B {
  void printB(){
    print("B");
  }
  void run(){
    print("B Run");
  }
}

class C extends Person with B,A{
  C(String name, num age) : super(name, age);

}

void main(){
  var c=new C('张三',20);
  c.printInfo();
  c.run(); // 后面的继承会把前面的给覆盖掉
}
```

输出：

``` dart
张三----20
A Run
```

## 泛型接口

​	本质上和java的泛型接口大同小异

``` dart
/*
Dart中的泛型接口:

    实现数据缓存的功能：有文件缓存、和内存缓存。内存缓存和文件缓存按照接口约束实现。

    1、定义一个泛型接口 约束实现它的子类必须有getByKey(key) 和 setByKey(key,value)

    2、要求setByKey的时候的value的类型和实例化子类的时候指定的类型一致
*/

abstract class Cache<T>{
  getByKey(String key);
  void setByKey(String key, T value);
}

class FlieCache<T> implements Cache<T>{
  @override
  getByKey(String key) {
    return null;
  }

  @override
  void setByKey(String key, T value) {
    print("我是文件缓存 把key=${key}  value=${value}的数据写入到了文件中");
  }
}

class MemoryCache<T> implements Cache<T>{
  @override
  getByKey(String key) {
    return null;
  }

  @override
  void setByKey(String key, T value) {
    print("我是内存缓存 把key=${key}  value=${value} -写入到了内存中");
  }
}
void main(){

  // MemoryCache m=new MemoryCache<String>();
  //  m.setByKey('index', '首页数据');

  MemoryCache m=new MemoryCache<Map>();

  m.setByKey('index', {"name":"张三","age":20});
}
```

java的对照一下：

``` java
package com.king.Abstract;

import java.util.HashMap;
import java.util.Map;

/**
 * @Author: guoning
 * @Description: // TODO
 * @Date: 2019/8/16 17:08
 * @Version: 1.0
 */
interface Cache<T>{
    String getByKey(String key);
    void setByKey(String key,T value);
}

class FlieCache<T> implements Cache<T>{

    @Override
    public String getByKey(String key) {
        return null;
    }

    @Override
    public void setByKey(String key, T value) {
        System.out.println("我是内存缓存 把key="+key+"value="+value+" -写入到了内存中");
    }
}

class MemoryCache<T> implements Cache<T>{

    @Override
    public String getByKey(String key) {
        return null;
    }

    @Override
    public void setByKey(String key, T value) {
        System.out.println("我是内存缓存 把key="+key+"value="+value+" -写入到了内存中");
    }
}

public class Delete2{
    public static void main(String[] args) {
            // MemoryCache m=new MemoryCache<String>();
            //  m.setByKey('index', '首页数据');
            Map<String,Object> map=new HashMap();
            map.put("name","张三");
            map.put("age",20);
            MemoryCache m=new MemoryCache<Map>();
            m.setByKey("index",map);
    }
}
```

