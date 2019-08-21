---
title: flutter零碎
date: 2019-08-21 18:06:29
categories: flutter
tags: [flutter,flutter小技巧]
---

# 沉浸式状态栏

``` dart
import 'package:flutter/material.dart';
/**************************主 要 引 入************************/
import 'dart:io';
import 'package:flutter/services.dart';
/************************************************************/
void main() {
  runApp(new MaterialApp(
    title: '滑动删除示例',
    debugShowCheckedModeBanner: false, // 去掉调试的标签
    home: new MyApp(),
  ));
}


class MyApp extends StatelessWidget {

  List<String> items = new List<String>.generate(30, (i) => "列表项 ${i + 1}");

  @override
  Widget build(BuildContext context) {
/*************************** 主 要 代 码 *********************/
    if (Platform.isAndroid) {
      SystemUiOverlayStyle systemUiOverlayStyle =
      SystemUiOverlayStyle(statusBarColor: Colors.transparent);
      SystemChrome.setSystemUIOverlayStyle(systemUiOverlayStyle);
    }
/************************************************************/
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

![沉浸式状态栏效果](1.PNG)