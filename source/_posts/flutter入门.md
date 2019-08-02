---
title: flutter入门
date: 2019-08-02 10:25:12
categories: flutter
tags: [flutter,Dart]
---

# Hello Word

​    工作需要学习flutter，只知道这玩意比原生的还好快，还要美观，而且还能跨平台。但是需要学习一门新的语言，也挺麻烦的。先来个demo，熟悉一个。

``` dart
import 'package:flutter/material.dart';
// 注释详细了一点
void main() => runApp(MyApp());  // 程序的入口，指向runApp类

class MyApp extends StatelessWidget {
  // This widget is the root of your application.根元素
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.red,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class NewRoute extends StatelessWidget { // 页面的跳转
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("New route"),
      ),
      body: Center(
        child: Text("This is new route"),
      ),
    );
  }
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
            FlatButton(
              child: Text("open new route"),
              textColor: Colors.blue,
              onPressed: () {
                //导航到新路由
                Navigator.push( context,
                    new MaterialPageRoute(builder: (context) {
                      return new NewRoute();
                    }));
              },
            ),
            Image.asset("images/avatar.png",
              width: 100.0,
            ),
            TextField(
              autofocus: true,
              decoration: InputDecoration(
                  labelText: "用户名",
                  hintText: "用户名或邮箱",
                  prefixIcon: Icon(Icons.person)
              ),
            ),
            TextField(
              decoration: InputDecoration(
                  labelText: "密码",
                  hintText: "您的登录密码",
                  prefixIcon: Icon(Icons.lock)
              ),
              obscureText: true,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}

```

