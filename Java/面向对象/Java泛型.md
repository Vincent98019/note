
**泛型：** 又称参数化类型，是JDK5.0出现的新特性，用于解决数据类型的安全性问题。可以在类或方法中预支地使用未知的类型，例如`ArrayList <E>`，`<E>`表示一种指定的数据类型，叫做泛型。 `E` ，取自**Element（元素）** 的首字母。
在出现`E`的地方，使用一种引用数据类型将其替换即可，表示我们将存储哪种引用类型的元素。

一般在创建对象时，将未知的类型确定具体的类型。当没有指定泛型时，默认类型为`Object`类型。避免了类型强转的麻烦。

```java
ArrayList<String> arrList1 = new ArrayList<>();
ArrayList<Student> arrList2 = new ArrayList<>();
```

> 如果不使用泛型，不能对加入到集合中的数据类型进行约束。
并且在遍历的时候，需要进行类型转换，如果数据量大，对效率有较大的影响。

Java泛型可以保证如果程序在编译时没有发出净高，运行时就不会产生`ClassCastException`异常。同时，代码更加简洁、健壮。


## 泛型的声明
接口：

```java
interface 接口名<T> {}
```

类：

```java
class 类<K,V> {}
```

> 比如：List、ArrayList

说明：
① 其中 T， K， V 不代表值，而是表示类型。
② 任意字母都可以。T 一般表示 Type 的缩写。

## 泛型实例化

要在类名后面指定类型参数的值(类型)。

比如：

```java
List<String> list = new ArrayList<String>();
Map<String, Object> map = new HashMap<String, Object>();
```

## 泛型细节

> ① `interface 接口名<T> {}`中，T、E等只能是引用类型。

```java
List<Integer> list = new ArrayList<Integer>();	// √
// List<int> list = new ArrayList<int>();	// ×
```

> ② 在给泛型指定了具体类型后，可以传入该类型或者该类型的子类类型。

```java
class A{}
class B extends A {}
class C {}

public static void main(String[] args) {
	List<A> list = new ArrayList<A>();
	list.add(new A());	// √
	list.add(new B());	// √
	// list.add(new C());	// ×
}
```

> ③ 在编译时，编译器会进行类型推断，所以泛型可以简写。
> 右边的尖括号中的泛型类型可以省略。

```java
List<Integer> list = new ArrayList<>();
//                    这里的可以省略 ↑
//                    但是尖括号需要保留
```

> ④ 如果不指定泛型类型，则默认为Object类型

```java
// 这样写的话，泛型的类型就是Object类型
List list = new ArrayList();

// 等价于
List<Object> list = new ArrayList<>();
```

## 自定义泛型类

**语法格式：**

```java
class 类名 <T, R ...> {	// ...表示可以有多个泛型
	// 成员
}
```

**细节：**

> ① 普通成员可以使用泛型(属性、方法)
> ② 使用泛型的数组，不能初始化
> ③ 静态成员不能使用类的泛型
> ④ 泛型类的类型，是在创建对象时确定的(因为创建对象时，需要指定泛型的类型)
> ⑤ 如果在创建对象时没有指定泛型类型，默认为Object

**使用案例：**

```java
// `T, R, M`是泛型的标识符，一般是单个字母大写。
// 泛型标识符可以有多个。
class Tiger<T, R, M> {
	T ttt;
	R rrr;
	M mmm;
	
	T[] tArr;	// 不能初始化，不能确定类型，就无法在内存中开辟空间

	// static T t;	// 静态成员无法使用类的泛型
	// static void setT(T t){}	// 因为静态成员是和类相关的
	// 在类没有创建对象时就可以调用，而泛型是在创建对象时指定的
	// 所以静态成员无法使用类的泛型，否则JVM则无法完成初始化
	
	// 普通成员
   public void setM(M m) {
       this.mmm = m;
  }
    
   public M getM() {
       return mmm;
  }
}
```

**使用泛型类：**

```java
Tiger<String, Integer, Object> tiger = new Tiger<>();

// 没有指定，就默认指定为Object
Tiger tiger = new Tiger();
// 等价于
// Tiger<Object, Object, Object> tiger = new Tiger<>();
```

## 自定义泛型接口

**语法格式：**

```java
interface 接口名 <T, R ...> {	// ...表示可以有多个泛型
	// 成员
}
```

**细节：**

> ① 静态成员不能使用接口的泛型
> ② 泛型接口的类型，在**继承接口**或**实现接口**时指定
> ③ 如果没有指定泛型类型，默认为Object

**使用案例：**

```java
// 泛型标识符可以有多个。
interface IUsb<U, R> {

	// 普通的抽象方法，可以使用接口泛型
	R getR(U u);
	vodi hello(R r);
	
	// U name = "arbor";	// 在接口中，属性都是静态的，无法使用接口泛型
	
	// 在java 8中，可以在接口中使用默认方法
   default R method(U u) {
		// 方法体
	}
    
}
```

**使用泛型接口：**

```java
// 泛型接口需要在继承或者实现时指定泛型类型
interface IA extends IUsb<String, Double> {
	// ...
}

// 因为IA接口指定了泛型类型，AA实现类则不用指定了，泛型类型会直接被替换为指定的类型
class AA implements IA {
	// ...
}

// 泛型接口需要在继承或者实现时指定泛型类型
class BB implements IUsb<String, Integer> {
	// ...
}

// 如果不指定泛型类型，默认为Object
class CC implements IUsb {
	// ...
}

// 等价于
// class CC implements IUsb<Object, Object> {	// ...	}


// 也可以在继承或者实现接口时不指定泛型，直到创建对象时指定
class DD<U, R> implements IUsb<U, R> {
	// ...
}
```

## 自定义泛型方法

**语法格式：**

```java
修饰符 <T, R...> 返回类型 方法名(参数列表) {	// ...表示可以有多个泛型
	// 方法体
}
```

**细节：**

> ① 泛型方法可以定义在普通类中，也可以定义在泛型类中
> ② 当泛型方法被调用时，泛型类型会确定
> ③ `public void eat(E e){}`，因为修饰符后没有`<T, R...>`，所以，该方法不是泛型方法，而是使用了泛型。

**使用案例：**

```java
// 普通类
class Cat {
    
    // 泛型方法
    // <T, R> 就是泛型，只给该方法使用
    public <T, R> void fly(T t, R r) {
    }
    
}

// 泛型类
class Dog<T, R> {

	// 普通方法
	public void eat() {
	}
    
    // 泛型方法
    // <A, E> 就是泛型，只给该方法使用
    // 尽量保持不和类泛型一致，提高代码阅读
    public <A, E> void fly(A a, E e) {
    }

	// 泛型方法
	// 泛型方法既可以使用自定义的泛型，也可以使用类定义的泛型
	// B 使用了方法定义的泛型，R 使用了类定义的泛型
	public <B> void sleep(B b, R r){
	}

	// 使用了泛型的方法
	// 该方法不是泛型方法，是方法使用了类定义的泛型
	public void run(T t){
	}
    
}
```

**使用泛型方法：**

```java
Cat cat = new Cat();
cat.fly("宝马", 100);	// 在调用泛型方法时，编译器会确定传入参数的类型

Dog<String, Integer> dog = new Dog<>();
dog.sleep("晚上睡", "白天起");	// 此时，该方法的 B 和 R 泛型都是String类型
dog.run("跑");	// 因为已经指定了类泛型 T 的类型，所以这里的参数只能是String类型
```

## 泛型通配符和继承性

**泛型不具备继承性**，比如：
```java
// 这样写是错的，泛型没有继承性
List<Object> list = new ArrayList<String>();
```

**泛型的通配符：** 不知道使用什么类型来接收的时候，此时可以使用`?`，`?`表示未知通配符。

当使用泛型类或者接口时，传递的数据中，泛型类型不确定，可以通过通配符`<?>`表示。
但是一旦使用泛型的通配符后，只能使用Object类中的共性方法，集合中元素自身方法无法使用。

**语法格式：**

`<?>`：支持任意泛型类型
`<? extends A>`：支持A类以及A类的子类，规定了泛型的上限
`<? super A>`：支持A类以及A类的父类，规定了泛型的下限

**使用案例：**

比如：现已知Object类，String 类，Number类，Integer类，其中Number是Integer的父类

```java
class Test {
	public static void main(String[] args) {
	   Collection<Integer> list1 = new ArrayList<Integer>();
	   Collection<String> list2 = new ArrayList<String>();
	   Collection<Number> list3 = new ArrayList<Number>();
	   Collection<Object> list4 = new ArrayList<Object>();
	   
	   getElement(list1);
	   getElement(list2);//报错
	   getElement(list3);
	   getElement(list4);//报错
	 
	   getElement2(list1);//报错
	   getElement2(list2);//报错
	   getElement2(list3);
	   getElement2(list4);
	 
	}
	// 泛型的上限：此时的泛型?，必须是Number类型或者Number类型的子类
	public static void getElement(Collection<? extends Number> coll){}
	// 泛型的下限：此时的泛型?，必须是Number类型或者Number类型的父类
	public static void getElement2(Collection<? super Number> coll){}

}
```


