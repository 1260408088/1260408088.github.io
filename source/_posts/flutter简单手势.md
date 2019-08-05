---
title: flutter简单手势
date: 2019-08-05 17:26:06
categories: flutter
tags: [flutter,手势]
---

# 简单的手势

## 轻触tip

<!--more-->

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: '按下处理Demo',
    home: new MyApp(),
  ));
}

class MyButton extends StatelessWidget{
  @override
  Widget build(BuildContext context) {

    return new GestureDetector(
      onTap: (){  // 回调函数
        final snackBar = new SnackBar(content: new Text("按下"),);  // 底部弹出的提示，类似于toast
        Scaffold.of(context).showSnackBar(snackBar);
      },
      child: new Container(
        padding: new EdgeInsets.all(12.0),
        decoration: new BoxDecoration(
          color: Theme.of(context).buttonColor,
          borderRadius: new BorderRadius.circular(10.0),
        ),
        child: new Text('测试按钮'),
      ),
    );
  }
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new Scaffold(
        appBar: new AppBar(
          title: new Text('按下处理Demo'),
        ),
        body:new Center(child: new MyButton(),)
    );
  }
}

```

![](1.PNG)

## 滑动删除

​	结合listview使用滑动手势删除。

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: '滑动删除示例',
    home: new MyApp(),
  ));
}


class MyApp extends StatelessWidget {

  List<String> items = new List<String>.generate(30, (i) => "列表项 ${i + 1}");

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('滑动删除示例'),
      ),
      body: new ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) { // 有点懵逼，这语言有点委婉
          final item = items[index];
          return new Dismissible(
              key: new Key(item),
              onDismissed: (direction) {
                items.removeAt(index);
                Scaffold.of(context).showSnackBar(
                    new SnackBar(content: new Text("$item 被删除了")));
              },
              child: new ListTile(title: new Text('$item'),)
          );
        },
      ),
    );
  }
}
```

![](2.PNG)

