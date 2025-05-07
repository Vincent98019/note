Lombok 是一个用于简化 Java 代码的库，主要通过注解自动生成样板代码，减少冗余代码的编写量。以下是一些常用 Lombok 注解的作用：

### 1. `@Getter` / `@Setter` vs 手写 Getter / Setter

`@Getter` 和 `@Setter` 让你不用手写 `getXxx()` 和 `setXxx()` 方法，比手写 getter/setter 更简洁。

```java
@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

等价于：

```java
public class User {
    private String name;
    private int age;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

### 2. `@ToString` vs 手写 `toString()` 方法

`@ToString` 自动生成 `toString()` 方法，比自己手写 `toString()` 更方便。

```java
@ToString
public class User {
    private String name;
    private int age;
}
```

等价于：

```java
public class User {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "User(name=" + name + ", age=" + age + ")";
    }
}
```

### 3. `@EqualsAndHashCode` vs 手写 `equals()` 和 `hashCode()`

`@EqualsAndHashCode` 生成 `equals()` 和 `hashCode()` 方法，比自己写更省力，避免遗漏字段。

```java
@EqualsAndHashCode
public class User {
    private String name;
    private int age;
}
```

等价于：

```java
public class User {
    private String name;
    private int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

### 4. `@NoArgsConstructor` vs 手写无参构造

`@NoArgsConstructor` 生成无参构造方法，比手写无参构造更快捷。

```java
@NoArgsConstructor
public class User {
    private String name;
    private int age;
}
```

等价于：

```java
public class User {
    private String name;
    private int age;

    public User() {}
}
```

### 5. `@AllArgsConstructor` vs 手写全参构造

`@AllArgsConstructor` 生成全参数构造方法，比手写更方便。

```java
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

等价于：

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### 6. `@Data` vs 组合多个注解

`@Data` 其实是 `@Getter`、`@Setter`、`@ToString`、`@EqualsAndHashCode` 和 `@RequiredArgsConstructor` 的综合体，比单独加多个注解更高效。

```java
@Data
public class User {
    private String name;
    private int age;
}
```

等价于多个注解的组合。

### 7. `@Builder` vs 传统构造器

`@Builder` 提供链式调用的方式创建对象，比传统构造方法更优雅。

```java
@Builder
public class User {
    private String name;
    private int age;
}
```

调用方式：

```java
User user = User.builder()
               .name("Alice")
               .age(25)
               .build();
```

等价于手写 Builder 模式。

---

**总结：Lombok 比手写代码更简洁、更高效、更易维护！**