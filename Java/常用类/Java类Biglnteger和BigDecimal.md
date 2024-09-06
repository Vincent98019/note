如果基本的整数和浮点数精度不能够满足需求， 那么可以使用`java.math`包中的两个很有用的类：`Biglnteger`和`BigDecimal`。这两个类可以处理包含任意长度数字序列的数值。

- `Biglnteger`类实现了任意精度的整数运算
- `BigDecimal`实现了任意精度的浮点数运算

**valueOf方法可以将普通的数值转换为大数值：**

```java
Biglnteger a = Biglnteger.valueOf(100);
```

不能使用算术运算符（如：`+`和`*`) 处理大数值，要使用大数值类中的`add`和`multiply`方法：

```java
Biglnteger c = a.add(b); // c = a + b
Biglnteger d = c.multiply(b.add(Biglnteger.valueOf(2))); // d = c * (b + 2)
```

## 常用方法

### Biglnteger类

`add(Biglnteger other)`：返回这个大整数和另一个大整数 other 的和。
`subtract(Biglnteger other)`：返回这个大整数和另一个大整数 other 的差。
`multipiy(Biginteger other)`：返回这个大整数和另一个大整数 other 的积。
`divide(Biglnteger other)`：返回这个大整数和另一个大整数 other 的商。
`mod(Biglnteger other)`：返回这个大整数和另一个大整数 other 的余数。

`compareTo(Biglnteger other)`：如果这个大整数与另一个大整数 other 相等，返回 0；如果这个大整数小于另一个大整数 other, 返回负数；否则，返回正数。

`valueOf(long x)`：返回值等于 x 的大整数。

### BigDecimal类

`add(BigDecimal other)`：返回这个大实数与另一个大实数 other 的和。
`subtract(BigDecimal other)`：返回这个大实数与另一个大实数 other 的差。
`multipiy(BigDecimal other)`：返回这个大实数与另一个大实数 other 的积。
`divide(BigDecimal other, RoundingMode mode)`：返回这个大实数与另一个大实数 other 的商。要想计算商，必须给出舍入方式（rounding mode)。
> `RoundingMode.HALF_UP`是在学校中学习的四舍五入方式(数值 0 到 4 舍去，数值 5 到 9 进位）。它适用于常规的计算。
> `RoundingMode.ROUND_CEILING`：保留分子的精度。
> 如果不加的话，得到的结果可能是无限循环小数，会抛出异常。

`int compareTo(BigDecimal other)`：如果这个大实数与另一个大实数相等，返回 0;如果这个大实数小于另一个大实数，返回负数；否则，返回正数。

`valueOf(long x)`：返回值等于 x 的大实数。
`valueOf(long x, int scale)`：返回值为`x / 10scale`的一个大实数。