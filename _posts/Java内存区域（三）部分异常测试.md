---
title: Java内存区域（三）部分异常测试
date: 2021-12-13 21:58:00
tags:
- Java
- Jvm
categories:
- [Java,Jvm]

---

本章目的：

1. 验证各个内存存储区域的内容
2. 简单演示各个区域发生的各种异常

------

####  Java堆溢出

Java堆用于存储对象实例，因此我们只需不断的创建对象，并且确保这些对象不被GC回收，那么随着对象数量的增加，总容量达到堆最大内存限制后就会产生内存溢出异常。

通过参数堆Jvm进行限制：

- 设置Java堆大小为20m 不可扩展 （将堆的最小值  `-Xms` 参数与最大值 `-Xmx` 设置为一样的数值即可避免自动扩展 ）
  - -Xms20m  -Xmx20m
- 通过参数 `-XX:+HeapDumpOnOutOfMemoryError`可以在出现内存溢出异常时Dump出内存堆转储快照。

![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112132308710.png)

图 3-1	idea设置Jvm参数



```java
public class Test {

    static class OomObject{
    }
    public static void main(String[] args) {
        List<OomObject> list = new ArrayList<>();
        while (true){
            list.add(new OomObject());
        }
    }
}
```

代码 3-1	Java堆内存溢出异常测试	

![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112132314663.png)

图3-2	运行结果

------

#### 虚拟机栈与本地方法栈溢出

对于HotSpot并不区分虚拟机栈与本地方法栈，因此 `-Xoss` 参数（设置本地方法栈大小）虽然存在，但实际上并没有任何效果。栈容量只能由 `-Xss` 参数设置

关于虚拟机栈的异常有两种

1. 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出 `StackOverflowError` 异常。
2. 如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出 `OutOfMemoryError` 异常。 HotSpot 不支持扩展所以创建线程申请内存时因无法获得足够内存才会出现 `OutOfMemoryError` 异常。

##### 测试`StackOverflowError` 异常 （请求栈深度方式）

**1.配置`jvm`参数**

使用 `-Xss` 参数减少栈内存容量，win64位下Java 11 栈容量不能低于180k ，设置内存容量为180k  `-Xss180k`

![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112162153310.png)

图3-3	设置Jvm 参数

![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112162154004.png)

图3-4	修改Jvm 参数

**2.测试代码**

```java
public class Test {

    private int stackLength = 1;

    public void stackLeak(){
        stackLength++;							// 统计栈深度
        stackLeak();							// 递归调用方法压栈
    }
    
    public static void main(String[] args) {
        Test test = new Test();
        try {
            test.stackLeak();
        }catch (Throwable e){
            System.out.println("stack length: "+ test.stackLength );
            throw e;
        }
    }
}
```



![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112181938410.png)

图3-4 `StackOverflowError` 异常测试结果

##### 测试`StackOverflowError` 异常 （本地变量表方式）

**测试代码 ** **`jvm` 设置与上面测试完全相同**

```java
public class Test {

    private int stackLength = 1;

    public void stackLeak(){
        stackLength++;
        long long1,long2,long3,long4,long5,long6,long7,long8,long9,long10,long11,long12,long13,long14,long15,
        long16,long17,long18,long19,long20,long21,long22,long23,long24,long25,long26,long27,long28,long29,long30,
        long31,long32,long33,long34,long35,long36,long37,long38,long39,long40,long41,long42,long43,long44,long45,
        long46,long47,long48,long49,long50,long51,long52,long53,long54,long55,long56,long57,long58,long59,long60,
        long61,long62,long63,long64,long65,long66,long67,long68,long69,long70,long71,long72,long73,long74,long75,
        long76,long77,long78,long79,long80,long81,long82,long83,long84,long85,long86,long87,long88,long89,long90,
        long91,long92,long93,long94,long95,long96,long97,long98,long99,long100;
        
        stackLeak();
        
        long1= long2= long3= long4= long5= long6= long7= long8= long9= long10= long11= long12= long13= long14= long15=
        long16= long17= long18= long19= long20= long21= long22= long23= long24= long25= long26= long27= long28=
        long29= long30= long31= long32= long33= long34= long35= long36= long37= long38= long39= long40= long41=
        long42= long43= long44= long45= long46= long47= long48= long49= long50= long51= long52= long53=
        long54= long55= long56= long57= long58= long59= long60= long61= long62= long63= long64=
        long65= long66= long67= long68= long69= long70= long71= long72= long73= long74=
        long75= long76= long77= long78= long79= long80= long81= long82= long83=
        long84= long85= long86= long87= long88= long89= long90= long91=
        long92= long93= long94= long95= long96= long97= long98= long99= long100= 0;
    }

    public static void main(String[] args) {
        Test test = new Test();
        try {
            test.stackLeak();
        }catch (Throwable e){
            System.out.println("stack length: "+ test.stackLength );
            throw e;
        }
    }
}
```



![](https://cdn.jsdelivr.net/gh/Xiaomy749/metocs_pic/202112182028776.png)

图3-5 `StackOverflowError` 异常测试结果

对比 图3-4 与图3-5 可发现 stack length 长度明显减小

**因此：无论由于栈帧太大还是虚拟机栈容量太小，当新栈帧内存无法分配的时候HotSpot虚拟机都将抛出 `StackOverflowError` 异常**



##### 测试 `OutOfMemoryError` 异常 （通过创建线程的方式）

以下32位系统之进行测试，有单个进程最大内存限制为2G

因虚拟机栈与本地方法栈分配到的内存是有限的，因此为每个线程分配到的栈内存越大，可以建立的线程数量就越少，建立线程时就越容易把剩余内存耗尽。

**测试代码 **

**注意**  以下代码执行风险很高

```java
public class Test {

    private  int stackLength = 1;

    private  void testTread(){  // 使线程的执行不会结束
        while (true){
        }
    }
    
    public  void createThread() {
        try {
            while (true){
                stackLength++;
                Thread thread = new Thread(()->{
                    testTread();
                });
                thread.start();
            }
        }catch (Throwable e){
            System.out.println("stack length: "+ stackLength);
            throw e;
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.createThread();
    }
}
```

------

#### 方法区和运行时常量池溢出测试

