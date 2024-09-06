
Heap (堆)，通过**new关键字创建的对象**都会使用堆内存

* 它是**线程共享**的，堆中对象都需要考虑线程安全的问题
* 有垃圾回收机制



## 堆内存溢出

`java.lang.OutofMemoryError ：java heap space` ：堆内存溢出

设置堆空间大小： `-Xmx2m`

```java
/**
 * 演示堆内存溢出 java.lang.OutOfMemoryError: Java heap space
 * -Xmx8m ，最大堆空间的jvm虚拟机参数，默认是4g
 */
public class main1 {
    public static void main(String[] args) {
        int i = 0;
        try {
            ArrayList<String> list = new ArrayList<>();// new 一个list 存入堆中
            String a = "hello";
            while (true) {
                list.add(a);// 不断地向list 中添加 a
                a = a + a;
                i++;
            }
        } catch (Throwable e) {// list 使用结束，被jc 垃圾回收
            e.printStackTrace();
            System.out.println(i);
        }
    }
}
```


## 堆内存诊断

### jps 工具

查看当前系统中有哪些 java 进程

![](Java/JVM/assets/JVM%E5%A0%86/101c5d61118ae0a615989488b1b890e4_MD5.png)



### jmap 工具

查看堆内存占用情况 `jmap - heap 进程id`

![](Java/JVM/assets/JVM%E5%A0%86/6722723fe7a8af2439bfa71a3a508858_MD5.png)



### jconsole 工具

图形界面的，多功能的监测工具，可以连续监测

![](Java/JVM/assets/JVM%E5%A0%86/6974636377f4242ada5ea22361463629_MD5.png)



### jvisualvm 工具

可视化的展示虚拟机的内容

![](Java/JVM/assets/JVM%E5%A0%86/69bd0280a06c0c8ec5d37384f7d780e8_MD5.png)

