# 1. Android 中控件与Flutter中控件的对应替换关系

![image-20191115231053587](https://tva1.sinaimg.cn/large/006y8mN6gy1g99a5zfqm4j31bw0g0q8o.jpg)

# 2. 搭建Flutter环境

搭建Flutter环境参考官方中文网https://flutterchina.club/setup-macos/

# 3. Dart基础-数字类型

![image-20191117174029006](https://tva1.sinaimg.cn/large/006y8mN6gy1g99a63mcftj30zw0f676u.jpg)

# 4. Dart字符串类型

![image-20191117175007563](https://tva1.sinaimg.cn/large/006y8mN6gy1g99a66ugo2j30wk0d8go4.jpg)

# 5. Dart list(集合)的基本使用

```dart
void _listType() {
    //这里声明的集合和java不太一样，java需要new出来一个集合对象，Dart只需要一个大括号
    List list1=[1,2,3,"我是海贼王"];
    print(list1);
    List<int> list2;
    //这么写是会报错的，因为list2里面已经加上了泛型限制，所以list2里面只能放int类型的数字
    //list2=list1;
    List list3=[];
    //声明一个定长的list
    List list4=new List(5);
    list3.add("list3");
    list3.addAll(list1);
    print("list3:$list3");
    //generate 函数 生成一个list 第一个参数是list元素个数 第二个是一个迭代器 （index)=>index *index
    //
    List list5=List.generate(3, (a)=>a *a);
    print(list5);

    //集合的遍历
    //1.for 循环遍历
    for(int i=0;i<list5.length;i++){
      print(list5[i]);
    }
    for(var o in list5){
      print(o);
    }
    list5.forEach((va){
      print(va);
    });
  }
```

# 6. Dart Map 的基本用法

```java
void _mapType(){
    //初始化集合用中括号[] 初始化键值对的map集合用大括号{}
    Map names={"xiaoming":"小明","xiaohong":"小红"};
    print(names);
    //第二种集合初始化方式
    Map ages={};
    ages["xiaocai"]=16;
    ages["xiaolei"]=18;
    print(ages);

    //map的遍历方式 foreach
    ages.forEach((key,value){
      print("$key,$value");
    });
    //第二种遍历方式
    Map ages2=ages.map((key,value){
      return MapEntry(value,key);
    });
    print(ages2);

    for(var key in ages.keys){
      //调用ages[key] 只能加大括号才能进行字符串拼接
      print("$key,${(ages[key])}");
    }
  }
```

# 7. Dart var,object,dynamic 三者的区别

~~~java
_tips(){
    //dynamic 动态数据类型 使用它编译器不能提醒你错误 因为它是运行时赋值的
    dynamic x="xiaocai";
    print(x.runtimeType);
    print(x);
    //可以被重新赋值为不同的数字类型
    x=123;
    print(x.runtimeType);
    print(x);

    //var 关键字 不关心它的数据类型是啥 不能被赋值为不同的数字类型
    var a="xiaocai";
    print(a.runtimeType);
    print(a);

    //object
    Object o="123";
    print(o.runtimeType);
    print(o);
    // o.foo(); //调用这个会被编译器报错，它只能调动object 中的方法
  }
~~~

# 8. Dart 的面向对象

~~~java
class Person {
  String _name;
  String _age;

  Person(this._age, this._name);
}

class Student extends Person {
  String _school; //通过下划线代表私有变量
  String city;
  String country;
  String name;

  /// city为可选参数 country为默认参数
  Student(this._school, String age, String name,
      {this.city, this.country = "china"})
      :
        //初始化列表：除了调用父类构造器，在子类方法体之前你也可以实例化变量， 不同的变量用逗号分开
        name="$country.$city",
        //如果父类没有默认的构造方法，则需要在初始化列表中调用父类的构造方法
        super(age, name);

  //命名构造方法
  Student.cover(Student stu):super(stu.name,stu._age);

  ///命名工厂构造方法
  factory Student.stu(Student stu){
    return Student(stu._age,stu.name,stu._school)
  }

}

class Logger{
  static Logger _cache;


  //工厂构造方法 类似与单例模式
  factory Logger(){
    if(_cache==null){
      _cache=Logger._internal();
    }
    return _cache;
  }
  //类的私有构造方法
  Logger._internal();

  void log(String msg){
    print(msg);
  }
}

~~~

