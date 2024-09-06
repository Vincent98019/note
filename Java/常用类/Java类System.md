`java.lang.System`类中提供了大量的静态方法，可以获取与系统相关的信息或系统级操作。

## 常用方法

`exit(int status)`：退出当前程序。一般传入0，表示正常退出程序。

`arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`：将数组中指定的数据拷贝到另一个数组中。

> 数组的拷贝动作是系统级的，性能很高。System.arraycopy方法具有5个参数。

| 参数名称 | 参数类型 | 参数含义 |
|:----|:----|:----|
|src | Object | 源数组 |
| srcPos | int | 源数组索引起始位置 |
| dest | Object | 目标数组 |
| destPos | int | 目标数组索引起始位置 |
| length | int | 复制元素个数 |

```java
public static void main(String[] args) {
    int[] src = new int[]{1,2,3,4,5};
    int[] dest = new int[]{6,7,8,9,10};
    System.arraycopy( src, 0, dest, 0, 3);
    /*代码运行后：两个数组中的元素发生了变化
     src数组元素[1,2,3,4,5]
     dest数组元素[1,2,3,9,10]
    */
}
```

`currentTimeMillis()`：返回以毫秒为单位的当前时间。

```java
public static void main(String[] args) {
   	//获取当前时间毫秒值
    System.out.println(System.currentTimeMillis()); // 1516090531144
}
```

`gc()`：运行垃圾回收机制。