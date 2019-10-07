---
title: flutter入门
date: 2019-08-02 10:25:12
categories: flutter
tags: [flutter,Dart]
---

# Hello Word

​    工作需要学习flutter，只知道这玩意比原生的还好快，还要美观，而且还能跨平台。但是需要学习一门新的语言，也挺麻烦的。先来个demo，熟悉一个。

<!--more-->

``` dart
import 'package:flutter/material.dart';
// 注释详细了一点
void main() => runApp(MyApp());  // 程序的入口，指向runApp类

class MyApp extends StatelessWidget {
  // This widget is the root of your application.根元素
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo', // 标题
      theme: ThemeData(
        primarySwatch: Colors.red, // 主题样式
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'), // home的界面
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
              width: 100.0, // 添加一个宽为100.0的图片(此处类型是double的)
            ),
            TextField( // 输入框
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
      floatingActionButton: FloatingActionButton( // 右下角的浮动按钮
        onPressed: _incrementCounter, // 按钮点击事件
        tooltip: 'Increment', // 长按弹出的字符
        child: Icon(Icons.add),
      ),
    );
  }
}

```

# flutter布局

​	flutter支持多重布局，每个布局先简单的介绍一下。

## 水平布局

``` dart
import 'package:flutter/material.dart';

void main() => runApp(
  new MaterialApp(
    title: '水平布局示例',
    home: new LayoutDemo(),
  ),
);

class LayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('水平布局示例'),
      ),
      body: new Row(
        children: <Widget>[
          new Expanded(
            child: new Text('我是第一个', textAlign: TextAlign.center),
          ),
          new Expanded(
            child: new Text('我是第二个', textAlign: TextAlign.center),
          ),
          new Expanded(
            child: new FittedBox(
              fit: BoxFit.contain, // otherwise the logo will be tiny
              child: const FlutterLogo(),
            ),
          ),
        ],
      ),
    );
  }
}

```

![水平布局](1.png)

## 垂直布局

``` dart
import 'package:flutter/material.dart';

void main() => runApp(
  new MaterialApp(
    title: '垂直布局示例',
    home: new LayoutDemo(),
  ),
);

class LayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('垂直布局示例'),
      ),
      body: new Column(
        children: <Widget>[
          new Text('大家好'),
          new Text('才是真的好'),
          new Expanded(
            child: new FittedBox(
              fit: BoxFit.contain,
              child: const FlutterLogo(),
            ),
          ),
        ],
      ),
    );
  }
}
```

![垂直布局](2.png)

## Container布局

​	许多布局会自由使用容器来使用padding分隔widget，或者添加边框（border）或边距（margin）。您可以通过将整个布局放入容器并更改其背景颜色或图片来更改设备的背景。

``` dart
import 'package:flutter/material.dart';

void main() => runApp(
  new MaterialApp(
    title: 'Container布局容器示例',
    home: new LayoutDemo(),
  ),
);

class LayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    Widget container = new Container(
      decoration: new BoxDecoration(
        color: Colors.black26,
      ),
      child: new Column(
        children: <Widget>[
          new Row(
            children: <Widget>[
              new Expanded(
                child: new Container(
                  width:150.0,
                  height: 150.0,
                  decoration: new BoxDecoration(
                    border: new Border.all(width: 10.0,color: Colors.black38),
                    borderRadius: const BorderRadius.all(const Radius.circular(8.0)),
                  ),
                  margin: const EdgeInsets.all(4.0),
                  child: new Image.asset('images/1.jpeg'),
                ),
              ),
              new Expanded(
                child: new Container(
                  width:150.0,
                  height: 150.0,
                  decoration: new BoxDecoration(
                    border: new Border.all(width: 10.0,color: Colors.black38),
                    borderRadius: const BorderRadius.all(const Radius.circular(8.0)),
                  ),
                  margin: const EdgeInsets.all(4.0),
                  child: new Image.asset('images/2.jpeg'),
                ),
              ),
            ],
          ),

          new Row(
            children: <Widget>[
              new Expanded(
                child: new Container(
                  width:150.0,
                  height: 150.0,
                  decoration: new BoxDecoration(
                    border: new Border.all(width: 10.0,color: Colors.black38),
                    borderRadius: const BorderRadius.all(const Radius.circular(8.0)),
                  ),
                  margin: const EdgeInsets.all(4.0),
                  child: new Image.asset('images/3.jpeg'),
                ),
              ),
              new Expanded(
                child: new Container(
                  width:150.0,
                  height: 150.0,
                  decoration: new BoxDecoration(
                    border: new Border.all(width: 10.0,color: Colors.black38),
                    borderRadius: const BorderRadius.all(const Radius.circular(8.0)),
                  ),
                  margin: const EdgeInsets.all(4.0),
                  child: new Image.asset('images/2.jpeg'),
                ),
              ),
            ],
          ),
        ],
      ),
    );
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('Container布局容器示例'),
      ),
      body:container,
    );
  }
}

```

![container布局](3.png)

demo中的图片需要在配置文件中配置

``` yaml
 assets:
    - images/1.jpeg
    - images/2.jpeg
    - images/3.jpeg
```

## GridView布局

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: 'GridView布局示例',
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    List<Container> _buildGridTitleList(int count) {
      return new List<Container>.generate(
          count,
              (int index) => new Container(
            child: new Image.asset('images/${index + 1}.jpeg'),
          ));
    }

    Widget buildGrid(){
      return new GridView.extent(
        maxCrossAxisExtent: 100.0, // 宽度为150像素的网格
        padding: const EdgeInsets.all(4.0),
        mainAxisSpacing: 4.0, // 上下的间距
        crossAxisSpacing: 8.0, // 左右的间距
        children: _buildGridTitleList(9),
      );
    }

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('GridView布局示例'),
      ),
      body: new Center(
        child: buildGrid(),
      ),
    );
  }
}

```

![GridView](4.png)

## ListView

### 静态列表

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: 'ListView布局示例',
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    List<Widget> list = <Widget>[ // 下一篇手势中，会有构造list的方法，dart的语法之后单独写
      new ListTile(
        title: new Text('广州市福田区中山路店',style: new TextStyle(fontWeight: FontWeight.w400,fontSize: 18.0),),
        subtitle: new Text('广州市福田区国际大厦8楼'),
        leading: new Icon(
          Icons.wifi,
          color: Colors.pinkAccent,
        ),
      ),
      new ListTile(
        title: new Text('广州市南山区北京路店',style: new TextStyle(fontWeight: FontWeight.w400,fontSize: 18.0),),
        subtitle: new Text('广州市南山区比克大厦18楼'),
        leading: new Icon(
          Icons.airplay,
          color: Colors.green,
        ),
      ),
      new ListTile(
        title: new Text('广州市福田区中山路店',style: new TextStyle(fontWeight: FontWeight.w400,fontSize: 18.0),),
        subtitle: new Text('广州市福田区国际大厦8楼'),
        leading: new Icon(
          Icons.wifi,
          color: Colors.pinkAccent,
        ),
      ),
      new ListTile(
        title: new Text('广州市福田区中山路店',style: new TextStyle(fontWeight: FontWeight.w400,fontSize: 18.0),),
        subtitle: new Text('广州市福田区国际大厦8楼'),
        leading: new Icon(
          Icons.title,
          color: Colors.deepPurple,
        ),
      ),
      new ListTile(
        title: new Text('北京市海淀区国际大厦',style: new TextStyle(fontWeight: FontWeight.w400,fontSize: 18.0),),
        subtitle: new Text('北京市海淀区国际大厦亢老师教育培训'),
        leading: new Icon(
          Icons.account_circle,
          color: Colors.lightBlueAccent,
        ),
      ),
    ];

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('ListView布局示例'),
      ),
      body: new Center(
        child: new ListView(
          children: list,
        ),
      ),
    );
  }
}

```

![地名瞎编的](5.png)

### 动态列表

``` dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
// TODO: implement build
    return MaterialApp(
        home: Scaffold(
      appBar: AppBar(title: Text('FlutterDemo')),
      body: HomeContent(),
    ));
  }
}

class HomeContent extends StatelessWidget {
  List list = new List();

  HomeContent({Key key}) : super(key: key) {
    for (var i = 0; i < 20; i++) {
      list.add("这是第${i}条数据");
    }
    print(list);
  }

  @override
  Widget build(BuildContext context) {
	// TODO: implement build
    return ListView.builder(
      itemCount: this.list.length,
      itemBuilder: (context, index) {
		// print(context);
        return ListTile(
          leading: Icon(Icons.phone),
          title: Text("${list[index]}"),
        );
      },
    );
  }
}

```



## stack

​	使用 Stack 在基本小部件(通常是图像)上排列小部件。 小部件可以完全或部分重叠基本小部件。

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: 'Stack层叠布局示例',
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    var stack = new Stack(
      alignment: const FractionalOffset(0.5, 0.5), // 文本在CircleAvatar所处的位置
      children: <Widget>[
        new CircleAvatar(
          backgroundImage: new AssetImage('images/1.jpeg'),
          radius: 100.0,
        ),
        new Container(
          decoration: new BoxDecoration(
            color: Colors.black38, // 盒子背景颜色
          ),
          child: new Text(
            'guoning',
            style: new TextStyle(
              fontSize: 22.0,
              fontWeight: FontWeight.bold,
              color: Colors.white,
            ),
          ),
        ),
      ],
    );

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('Stack层叠布局示例'),
      ),
      body: new Center(
        child: stack,
      ),
    );
  }
}

```

![stack](6.png)

## 层叠定位布局

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: '层叠定位布局示例',
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('层叠定位布局示例'),
      ),
      body: new Center(
        child: new Stack(
          children: <Widget>[
            new Image.network('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1539049245502&di=84f58b05165b05ab75587d1c6bc73e3f&imgtype=0&src=http%3A%2F%2Fpic.qiantucdn.com%2F58pic%2F25%2F99%2F58%2F58aa038a167e4_1024.jpg'),
            new Positioned( 
                bottom: 50.0, // 设置偏移量
                right: 50.0,
                child: new Text(
                  'guoning',
                  style: new TextStyle(
                    fontSize: 20.0,
                    fontWeight: FontWeight.bold,
                    fontFamily: 'serif',
                    color: Colors.pink,
                  ),
                )
            ),
          ],
        ),
      ),
    );
  }
}

```

![](7.png)

## 滚动布局

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: '滚动布局示例',
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new Scaffold(
      appBar: new AppBar(
        title: new Text('滚动布局示例'),
      ),
      body:new ListView(
        children: <Widget>[  // 同样此处的文字内容也可以改为图片的
          new Center(
            child: new Text(
              '\n九寨沟',
              style: new TextStyle(
                fontSize:30.0,
              ),
            ),
          ),
          new Center(
            child: new Text(
              '五花海风景点',
              style: new TextStyle(
                fontSize: 16.0,
              ),
            ),
          ),
          new Center(
            child: new Text( // 此处的''' '''用户包裹大段文字的
              '''
              九寨沟五花海 [1]  海拔2472米，水深5米，面积9万平方米，被誉为“九寨沟一绝”和“九寨精华”，在珍珠滩瀑布之上，熊猫湖的下部于日则沟孔雀河上游的尽头。五花海四周的山坡，入秋后便笼罩在一片绚丽的秋色中，色彩丰富，姿态万千。由于海底的钙华沉积和各种色泽艳丽的藻类，以及沉水植物的分布差异，使一湖之中形成了许多斑斓的色块，宝蓝、翠绿、橙黄、浅红，似无数块宝石镶嵌成的巨形佩饰，珠光宝气，雍容华贵。金秋时节，湖畔五彩缤纷的彩林倒映在湖面，与湖底的色彩混合成了一个异彩纷呈的彩色世界。其色彩十分丰富，甚至超出了画家的想象力。黄昏时分，火红的晚霞映入水中，湖水似金星飞溅，彩波粼粼，绮丽无比。从老虎嘴俯瞰它的全貌，俨然是一只羽毛丰满的开屏孔雀。长海流入五花海的水在经过石灰岩岩脉时，使水中带入了大量的石灰钙华物质。这些含有钙华物质的白色砂粒有很强的过滤作用，又像是热带珊瑚海中的沙子一样堆积着，连这里的藻类也因为受到了钙华物质的影响而变成白色。阳光一照，海子更为迷离恍惚，绚丽多姿，一片光怪陆离，使人进入了童话境地。
              有“九寨沟一绝”和“九寨精华”之誉的五花海，位于四川省九寨沟日则沟孔雀河上游的尽头。沿著幽林栈道，一路下坡而去，穿越幽林，不久便到达五花海。绕过五花海的西侧，有一段栈道是欣赏水光秋色的绝佳点。游人可在此驻卟。沿着栈道继续北行，到达五花海的北岸。一片空旷、平缓的山坡地很快就到了栈疲乏的终点。这里是五花海的出水口与孔雀河道的交接点，上建一座栈桥。栈桥南侧的湖面，水色斑斓，墨绿、宝蓝、翠黄的色块混杂交钽一光十色，似孔雀彩翅；栈桥北侧，河湾状如孔誉头颈，三株古树似顶花翎。因此从这里以下被称为孔雀河道。沿著孔雀河道的左岸北行约一百米，越过河道便上到环山公路。从这段公路俯视五花海，景色更加令人叫绝。沿环山公路往东南方向，就到了五花海东南侧的最高点，这里有一块巨大的石头，称为老虎石。站在老虎石上俯视，可以观察到五花海的全貌。
              五花海是九寨沟诸景点中最精彩一个。四周的山坡，入秋后便笼罩在一片绚丽的秋色中，色彩丰富，姿态万千，独霸九寨。五花海的彩叶大半集中在出水口附近的湖畔，一株株彩叶交织成锦，如火焰流金。含碳酸钙质的池水，与含不同叶绿素的水生群落，在阳光作用下，幻化出缤纷色彩，一团团、一块块，有湛蓝、有墨绿、有翠黄。岸上林丛，赤橙黄绿倒映池中，一片色彩斑斓，与水下沉木、植物相互点染，其美尤妙，故得名五花海。九寨人说：五花海是神池，它的水洒向哪儿，哪儿就花繁林茂，美丽富饶。
　　五花海的底部景观妙不可言，湖水一边是翠绿色的，一边是湖绿色的，湖底的枯树由于钙化，变成一丛丛灿烂的珊瑚，在阳光的照射下，五光十色，非常迷人
              ''',
              style: new TextStyle(
                fontSize: 14.0,
              ),
            ),
          ),
        ],
      ),
    );
  }
}

```

![](8.png)

## Card布局

``` dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter layout demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Flutter layout demo'),
        ),
        body: Center(child: _buildCard()),
      ),
    );
  }

  // #docregion Card
  Widget _buildCard() => SizedBox(
    height: 210,
    child: Card(
      child: Column(
        children: [
          ListTile(
            title: Text('1625 Main Street',
                style: TextStyle(fontWeight: FontWeight.w500)),
            subtitle: Text('My City, CA 99984'),
            leading: Icon(
              Icons.restaurant_menu,
              color: Colors.blue[500],
            ),
          ),
          Divider(), // 分割线
          ListTile(
            title: Text('(408) 555-1212',
                style: TextStyle(fontWeight: FontWeight.w500)),
            leading: Icon(
              Icons.contact_phone,
              color: Colors.blue[500],
            ),
          ),
          ListTile(
            title: Text('costa@example.com'),
            leading: Icon(
              Icons.contact_mail,
              color: Colors.blue[500],
            ),
          ),
        ],
      ),
    ),
  );
}
```
![](9.png)

​	于之前学习Android的时候，布局内容要更丰富一些，这布局只是一个浅显的入门，还要接着深入的。

