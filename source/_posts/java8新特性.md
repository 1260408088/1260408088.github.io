---
title: java8新特性
date: 2020-05-28 20:34:07
categories: java
tags: [新特新,lambda]
---

​	java8的新特性lambda表达式出来好久了，虽然了解过但是从来没有正式用到工作中。这么好的东西，不拿到工作中装装逼，这不是白瞎了吗？本次只关注如何使用，要探究所以然的话，要等下次

废话少说开整！！！！！！

<!--more-->

最先看到这个lambda表达式，是开启线程中用到的。java8以前的开启方式：

``` java
Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                 System.out.println("test");
            }
        });
t1.start();
```

java8中使用lambda表达式，简洁了许多：

``` java
 Thread t2 = new Thread(() -> {
            System.out.println("这是一个新的线程");
        });
t2.start();
```

## 1.Stream的各式使用

``` java
// 生成10个随机数
public static void main(String[] args) {
	Stream.generate(Math::random).limit(10).forEach(System.out::println);    
}
```

``` java
// 计算从0到9的和
Integer reduce = Stream.iterate(0, s -> s + 1).limit(10).reduce(0, (b, c) -> (b + c));
```

``` java
// 输出0-9的10个数字
Stream.iterate(0, s -> s + 1).limit(10).forEach(System.out::println);
```

``` java
// 对数组中的某一个属性排序 new("guo",17,false) 升序
List<User> list = collect();
        list.stream().sorted(Comparator.comparing(User::getAge)).forEach(user -> System.out.println(user.getName()));
// 降序
list.stream().sorted(Comparator.comparing(User::getAge).reversed()).forEach(user -> System.out.println(user.getName()));
```

``` java
// 排序只出现岁数最大的 用到了limit
list.stream().sorted(Comparator.comparing(User::getAge).reversed()).limit(1).forEach(user -> System.out.println(user.getName()));
```

``` java
// 使用max求出最大值
User user = list.stream().max(Comparator.comparing(User::getAge)).get(); System.out.println(user.getAge());
```

``` java
// 使用filter，获得年龄大于15岁的
list.stream().filter(user -> user.getAge()>15).forEach(user -> System.out.println(user.getName()));
```

## 2.collect的各种收集

``` java
// 将list中的姓名，取出来放入一个list中
list.stream().map(user -> user.getName()).collect(Collectors.toList()).forEach(s -> System.out.println(s));
```

``` java
// 将list中的姓名取出放入一个set中
Set<String> collect = list.stream().map(user -> user.getName()).collect(Collectors.toSet());
```

``` java
// 将list中的user的name取出来，组成一个map name为key，value为user
Map<String, User> map = collect().stream().collect(Collectors.toMap(User::getName, Function.identity()));
```

``` java
// 将list中的user的name为key，age为value
Map<String, Integer> collect = list.stream().collect(toMap(User::getName, User::getAge));
```

## 3.字符串的拼接

``` java
// 将user中的name都拼接到一块
String s = list.stream().map(user -> user.getName()).reduce((name1, name2) -> (name1+name2)).get();
// 亦可以这么写（上面看着较为简洁，都lambda了，当然是怎么简洁怎么来了）
String s = list.stream().map(user -> user.getName()).reduce((name1, name2) -> {
            return name1 + name2;
        }).get();
```

## 4.分组操作

``` java
// 简单的分组操作，根据name进行分组的操作（name为key）
Map<String, List<User>> collect = list.stream().collect(groupingBy(User::getName));
```

``` java
// 升级版本的分组（多字段的分组）
Map<String, List<User>> collect = list.stream().collect(groupingBy(user -> feachGroup(user)));
// 需要添加一个方法进行多字段的处理
 private static String feachGroup(User user){
        return user.getName()+user.getAge();
}
```

``` java
// 根据name进行分组数量的统计
Map<String, Long> collect = list.stream().collect(groupingBy(User::getName, counting()));
```

## 5.map中foreach循环

``` java
Map<String, User> map = collect().stream().collect(Collectors.toMap(User::getName, Function.identity()));
        map.forEach((name,user)-> System.out.println(name+user.getAge())); 
```

## 6.去重复

``` java
// list中的user对象去重复，需要重写user对象的equals与hashCode方法(方法在下面)
list.stream().distinct().forEach(user-> System.out.println(user.getName()));
```

``` java
@Override
public boolean equals(Object obj) {
    if (obj == null) {
        return false;
    }
    final User user = (User) obj;
    if (this == user) {
        return true;
    } else {
        return (this.name.equals(user.getName()) && this.isTrue == user.isTrue() && this.age == user.getAge());
    }
}

@Override
public int hashCode() {
    int hashno = 7;
    hashno = 13 * hashno + (name == null ? 0 : name.hashCode());
    return hashno;
}
```

## 7.根据Boolean的状态进行分组

``` java
Map<Boolean, List<User>> collect = list.stream().collect(partitioningBy(User::isTrue));
```

## 8.map的缓存读取

之前将数据放入到map中，要判断存在与否然后才能决定放值还是取值，现在可以换一种方式了

``` java
Map<String, String> cathchmap = new HashMap<>();
cathchmap.put("king","16");
cathchmap.put("sala","16");
cathchmap.put("loulou","16");
/***********未测试，应该可以的**************/
private void computeabsent(Map<String, String> cathchmap){
    cathchmap.computeIfAbsent("ning",this::redfromDb);
}

private String redfromDb(String s) {
    return "14";
}
```

## 9.并行流

有机会接着补充







