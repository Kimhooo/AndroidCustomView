jetpack开发：

1.LiveData 一个数据持有类，能根据组件的生命周期进行销毁，不容易引起内存泄露

~~~
MutableLiveData PostValue setValue 区别：
postValue 切换到主线程后再setValue 可以在任意线程调用
setValue 只能在主线程调用 
~~~

2.liveData 能监听到数据的改变从而做出一些改变,观察者模式

3.ViewModel 

~~~
以注重生命周期的方式管理界面的数据，共享内存的方式管理数据，从而使得在旋转屏幕后数据仍然不变
~~~

4.DataBinding

~~~
数据的双向绑定：
<layout>
<data>
~~~

5.Navigation 

~~~
控制管理fragment 跳转 
1.navigation xml
2.activity 中 加上fragment 标签 需要加上 android.NavHostFragment
3.NavigationControl
~~~

