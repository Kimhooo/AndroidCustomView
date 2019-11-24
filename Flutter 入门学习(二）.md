# 1. Flutter引用第三方包并使用

- Flutter 搜索第三方包和插件的地址是：https://pub.dev/packages

- 引入第三方包的位置是 pubspec.yaml (类似android的gradle文件)

  ![image-20191123094529019](/Users/cailei/Library/Application Support/typora-user-images/image-20191123094529019.png)

# 2. Flutter statelessWidget基础组件的使用

~~~java


import 'package:flutter/material.dart';

class PluginTest extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    TextStyle textStyle=TextStyle(fontSize: 19
    ,color: Colors.red);
    return MaterialApp(
      title: "我是路飞哈哈哈",
      //标题主题
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text("statelessWidget与基础组件")),
        body: Container(
          decoration: BoxDecoration(color: Colors.white),
          alignment: Alignment.center,
          child: Column(
            children: <Widget>[
              Text("I am Test",style: textStyle,),
              Icon(Icons.android,size: 50,color: Colors.red,),
              //关闭按钮
              CloseButton(),
              //返回按钮
              BackButton(),
              Chip(
                //头像
                avatar: Icon(Icons.people),
                label: Text("StatelessWidget 小部件"),

              ),
              Divider(
                color: Colors.red,
                height: 20, //容器的高度，不是线的高度，设置不了线的高度
                indent: 10, //左侧间距

              ),
              Card(
                //带有圆角，阴影，边框等效果的卡片 类似于android 中的CardView
                color: Colors.blue,
                elevation: 5, //卡片的阴影
                margin: EdgeInsets.all(10), //上下左右是10的边距
                child: Container(
                  padding: EdgeInsets.all(10),
                  child: Text("I am A card",style: textStyle,),

                ),
              ),
              AlertDialog(
                title: Text("盘他"),
                content: Text("你个糟老头子坏得很"),
              )
            ],
          ),
        ),
      ),
    );
  }
}
~~~

# 3. Flutter StatefulWidget 基础组件的使用

~~~java
import 'package:flutter/material.dart';

class StatefulGroupTest extends StatefulWidget {
  @override
  _StatefulGroupTestState createState() => _StatefulGroupTestState();
}

class _StatefulGroupTestState extends State<StatefulGroupTest> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "statefulWidget与基础组件",
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text("statefulWidget与基础组件"),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {},
          child: Text("点我"),
        ),
        bottomNavigationBar: BottomNavigationBar(
            currentIndex: _currentIndex,
            onTap: (index) {
              setState(() {
                _currentIndex = index;
              });
            },
            items: [
              BottomNavigationBarItem(
                  icon: Icon(
                    Icons.home,
                    color: Colors.grey,
                  ),
                  activeIcon: Icon(Icons.home, color: Colors.blue),
                  title: Text("首页")),
              BottomNavigationBarItem(
                  icon: Icon(
                    Icons.list,
                    color: Colors.grey,
                  ),
                  activeIcon: Icon(Icons.home, color: Colors.blue),
                  title: Text("列表"))
            ]),
        body: _currentIndex == 0
            ? RefreshIndicator(
                child:
                ListView(children:[Text("路飞"),Text("乔巴"),Text("娜美")]),
                onRefresh: _refreshCallBack,
              )
            : Text("列表界面"),
      ),
    );
  }

  Future<Null> _refreshCallBack() async {
    await Future.delayed(Duration(milliseconds: 2000));
    return null;
  }
}

~~~

# 4. Flutter 布局开发

![image-20191124133059986](/Users/cailei/Library/Application Support/typora-user-images/image-20191124133059986.png)

