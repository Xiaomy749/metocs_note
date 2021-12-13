---
title: Java内存区域（三）内存区域异常实战
date: 2021-12-13 21:58:00
tags:
- Java
- Jvm
categories:
- [Java,Jvm]

---

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

​	

