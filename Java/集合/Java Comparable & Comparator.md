
**`Comparable`**：强行对实现它的每个类的对象进行整体排序。
> 这种排序被称为类的自然排序，类的compareTo方法被称为它的自然比较方法。只能在类中实现compareTo()一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**`Comparator`**：强行对某个对象进行整体排序。
> 可以将Comparator 传递给sort方法（如Collections.sort或 Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用Comparator来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象collection提供排序。


如果在使用的时候，想要独立的定义规则去使用 可以采用`Collections.sort(List list,Comparetor<T> c)`方式，自己定义规则：

```java
Collections.sort(list, new Comparator<Student>() {
   @Override
   public int compare(Student o1, Student o2) {
       return o2.getAge()-o1.getAge();//以学生的年龄降序
  }
});
```

如果想要规则更多一些，可以参考下面代码：

```java
Collections.sort(list, new Comparator<Student>() {
           @Override
           public int compare(Student o1, Student o2) {
               // 年龄降序
               int result = o2.getAge()-o1.getAge();//年龄降序
               if(result==0){//第一个规则判断完了 下一个规则 姓名的首字母 升序
                   result = o1.getName().charAt(0)-o2.getName().charAt(0);
              }
               return result;
          }
      });
```