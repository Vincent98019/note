`java.util.Arrays`此类包含用来操作数组的各种方法，比如排序和搜索等。其所有方法均为静态方法，调用起来非常简单。

## 常用方法

`toString(Object[] arr)`：返回数组的字符串形式。

```java
Arrays.toString(arr);
```


`sort(...)`：排序，分为自然排序和定制排序两种。因为数组是引用类型，所以使用该方法排序后会直接影响原数组的元素顺序。

```java
Integer[] arr = {1, -7, 0, 85, 62};
// 自然排序
Array.sort(arr);
// 定制排序，第二个数组传入一个比较器
Array.sort(arr, new Comparator<Integer>() {
	@Override
	public int compare(Integer o1, Integer o2) {
		return o2 - o1;
	}
});
```

> sort方法定制排序时，会走到TimSort类的BinarySory方法，然后调用传入的比较器的compare方法，该方法的返回值来决定排序的顺序

`binarySearch(Objectp[] arr, Object obj)`：通过二分搜索法进行查找，要求传入的数组有序并且是升序。
> 如果数组不存在该元素或者数组无序，返回负数。
> 如果数组中不存在，返回的是应该在的位置+1的相反数，就是一个负数。

`copyOf(Object[] arr, int a)`：返回一个新数组，将arr数组中的的前a个元素复制到新的数组中，如果传入的a大于arr数组的索引，大于的部分的元素会设置为默认值。

> 如果a传入负数会报异常。
> 该方法的底层使用的是System.arraycopy()方法。

`fill(Objectp[] arr, Object obj)`：数组元素填充，将arr数组中的元素全部填充为obj。

`equals(Objectp[] arr, Objectp[] arr2)`：比较两个数组内容是否完全一致，完全一样返回true。

`asList(...)`：返回一个List对象，将一组值转换为List。

> 编译类型是List接口，运行类型是Arrays类中的静态内部类ArrayList类。
