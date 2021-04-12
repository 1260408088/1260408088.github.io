---
title: flutter页面的跳转
date: 2019-08-06 15:34:52
categories: flutter
tags: [flutter,dart]
---

# Flutter简单的页面跳转

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: 'Stack层叠布局示例',
    home: new FirstScreen(),
  ));
}

class FirstScreen extends StatelessWidget{

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('导航页面'),
      ),
      body: new Center(
        child: new RaisedButton(child: new Text('查看商品详情'), 
            onPressed: (){ // 以压栈的形式，将页面压入
              Navigator.push(context, new MaterialPageRoute(builder: (context)=> new SecondScreen())
              );
        }),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    return new Scaffold(appBar: new AppBar(
      title: new Text("这是一个金铲铲"),
    ),
    body: new Center(
      child: new RaisedButton(
          onPressed: (){ // 返回页面的时候会将栈顶元素pop出来
            Navigator.pop(context);
          },
        child: new Text('返回页面'),
        ),
      ),
    );
  }
}

```

![第一个页面](1.PNG)

![第二个页面](2.PNG)

# 页面跳转并发送数据

``` dart
import 'package:flutter/material.dart';

class Product {
  final String title;
  final String description;

  Product(this.title, this.description);
}

void main() {
  runApp(new MaterialApp(
    title: '传递数据示例',
    home: new ProductList(
      products:
      new List.generate(20, (i) => new Product('商品 $i', '这是一个商品详情 $i')),
    ),
  ));
}

class ProductList extends StatelessWidget {
  final List<Product> products;

  ProductList({Key key, @required this.products}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("商品列表"),
      ),
      body: new ListView.builder(
          itemCount: products.length,
          itemBuilder: (context, index) {
            return new ListTile(
              title: new Text(products[index].title),
              onTap: () {
                Navigator.push(
                  context,
                  new MaterialPageRoute(
                      builder: (context) =>
                      new ProductDetail(product: products[index])),
                );
              },
            );
          }),
    );
  }
}

class ProductDetail extends StatelessWidget {
  final Product product;

  ProductDetail({Key key, @required this.product}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("${product.title}"),
      ),
      body: new Padding(
        padding: new EdgeInsets.all(16.0),
        child: new Text('${product.description}'),
      ),
    );
  }
}

```

![](3.PNG)

![](4.PNG)

# 带数据返回的页面跳转

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: '跳转页面返回数据',
    home: new FirstPage(),
  ));
}

class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("跳转页面返回数据"),
      ),
      body: new Center(child: new RouteButton(),),
    );
  }
}

class RouteButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new RaisedButton(
      onPressed: (){
        _navigateToSecondPage(context);
      },
      child: new Text('选择一个选项，任意选项'),
    );

  }

  _navigateToSecondPage(BuildContext context) async {

    final result = await Navigator.push(
      context,
      new MaterialPageRoute(builder: (context) => new SecondPage()),
    );

    Scaffold.of(context).showSnackBar(new SnackBar(content: new Text("$result")));

  }
}


class SecondPage extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("选择一条数据"),
      ),
      body: new Center(
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Padding(
              padding: const EdgeInsets.all(8.0),
              child: new RaisedButton(
                onPressed: (){
                  Navigator.pop(context,'hi，您好');
                },
                child: new Text('hi，您好'),
              ),
            ),
            new Padding(
              padding: const EdgeInsets.all(8.0),
              child: new RaisedButton(
                onPressed: (){
                  Navigator.pop(context,'hi flutter');
                },
                child: new Text('hi flutter'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

