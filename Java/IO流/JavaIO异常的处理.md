
## JDK7前处理

使用 `try...catch...finally` 代码块，处理异常部分。
```java
public class HandleException1 {
    public static void main(String[] args) {
        // 声明变量
        FileWriter fw = null;
        try {
            //创建流对象
            fw = new FileWriter("fw.txt");
            // 写出数据
            fw.write("程序员"); //程序员
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## JDK7的处理

还可以使用JDK7优化后的 `try-with-resource` 语句，该语句确保了每个资源在语句结束时关闭。所谓的资源（resource）是指在程序完成后，必须关闭的对象。

**格式：**
```java
try (创建流对象语句，如果多个,使用';'隔开) {
    // 读写数据
} catch (IOException e) {
    e.printStackTrace();
}
```

```java
public class HandleException2 {
    public static void main(String[] args) {
        // 创建流对象
        try ( FileWriter fw = new FileWriter("fw.txt"); ) {
            // 写出数据
            fw.write("程序员"); //程序员
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## JDK9的改进

JDK9中 `try-with-resource` 的改进，对于引入对象的方式，支持的更加简洁。被引入的对象，同样可以自动关闭，无需手动close。

```java
// 被final修饰的对象
final Resource resource1 = new Resource("resource1");

// 普通对象
Resource resource2 = new Resource("resource2");

// 引入方式：直接引入
try (resource1; resource2) {
    // 使用对象
}
```

```java
public class TryDemo {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        final  FileReader fr  = new FileReader("in.txt");
        FileWriter fw = new FileWriter("out.txt");
        // 引入到try中
        try (fr; fw) {
            // 定义变量
            int b;
            // 读取数据
            while ((b = fr.read())!=-1) {
                // 写出数据
                fw.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
