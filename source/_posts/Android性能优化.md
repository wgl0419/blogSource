---
title: Android性能优化
date: 2018-01-25 16:00:25
tags: android
---

# 布局优化

1. ui渲染机制

android中，系统通过VSYNC信号触发对ui的渲染、重绘，间隔是16ms，即每秒60帧。
如果系统每次渲染的时间都保持在16ms之内，那么我们看见的ui界面是非常流畅的，这需要讲所有程序的逻辑都保证在16ms之内，如果不能在16ms内完成绘制，就会造成丢帧现象。
导致16ms*n一直显示同一帧，产生卡顿的现象


2. 避免Overdraw

过度绘制是指给布局绘制了重叠的背景，过度绘制会浪费很多的cpu、gpu资源。使用GPU OVERDRAW可以检查

3. 优化布局层级

android中，系统对view进行测量、布局和绘制时，都是通过对view树的遍历来进行操作的，如果一个view树太高，就会严重影响测量、布局和绘制的速度，因此优化布局的第一个方法就是降低view树的高度，树的高度不宜超过10层。

使用RelativeLayout可以有效降低嵌套。

4. 避免嵌套过多无用布局

嵌套的布局会让view树的高度越来越高，因此在布局时，需要根据自身布局的特点来选择不同的layout组件，从而避免通过某一种layout组件来实现功能时的局限性，从而造成嵌套过多的情况发生。

使用<include>可以复用标签
使用<viewstub>可以实现view的延迟加载。viewstub只有在显示时才会去渲染整个布局，而view.gone，在初始化布局树的时候就已经添加在布局树上面了。


5. Hierarcy Viewer

使用Hierarcy可以很容易的看到布局结构，找到冗杂的布局进行优化。

# 内存优化

1. 什么是内存

寄存器：速度最快的存储场所，因为寄存器位于处理器内部，在程序中无法控制
栈：存放基本类型的数据和对象的引用，但对象本身不存放在栈中，而是存放在堆中
堆：存放由new创建的对象和数组，在堆中分配的内存，由java虚拟机的自动垃圾回收装置回收（gc）
静态存储区域：静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据，java专门划分了一个静态存储区来管理一些特殊的数据变量，如静态的数据变量。
常量池：jvm虚拟机必须为每个被装载的类型维护一个常量池，常量池就是该类型所用到常量的一个有序集合，包括直接常量和对其他类型、字段和方法的符号引用。

```
	ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
	int heapSize = manager.getLargeMemoryClass();
```

使用如上代码可以获取heap的大小。

2. 获取android系统内存信息

Process state: 在android k 以上设置中开启该功能，进行内存的监视。

也可以使用 

```
	adb shell dumpsys procstats
```

Meminfo: 

```
	adb shell dumpsys meminfo
```

3. 内存回收

使用System.gc()可以建议系统进行gc，但系统是否采纳就不一定，有些对象因为算法的原因，不在回收，就会造成内存泄漏

4. 内存优化实例

bitmap优化：bitmap是造成内存占用过高，甚至是oom的最大威胁。可以使用适当分辨率和大小的图片，及时使用bitmap.recycle()回收内存，使用图片缓存（内存缓存或硬盘缓存）来减少该问题的发生。

代码优化：
1. 对常量使用static修饰符
2. 使用静态方法，静态方法会比普通方法提高15%的访问速度
3. 减少不必要的成员变量
4. 减少不必要的对象，使用基础类型会比使用对象更加节省资源，同时更应避免频繁创建短作用域的变量。
5. 尽量不用使用枚举，少用迭代器
6. 对Cursor, Receiver, Sensor, File等对象，要非常注意对他们的创建，回收与注册，解注册。
7. 避免使用IOC框架，注解和反射会带来性能的下降
8. 使用RenderScript, OpenGL来进行非常复杂的绘图操作
9. 使用SurfaceView来替代view进行大量、频繁的绘图操作
10. 尽量使用视图缓存，而不是每次都执行inflate（）方法。