# 条目9：与try-finally相比，首选try-with-resources

## 核心原则

**try-with-resources** 是Java 7引入的一个语言特性，用于自动管理资源的关闭操作。当处理必须关闭的资源时（如文件、数据库连接、网络连接等），应当优先使用try-with-resources而非try-finally，因为它能提供更简洁、更可靠的资源管理方式。

## 传统try-finally的问题

传统的try-finally方式存在以下问题：

```java
// 传统的try-finally方式
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close(); // 可能抛出异常，且会掩盖try块中的异常
    }
}
```

这种方式存在几个明显缺陷：
1. **异常掩盖问题**：如果try块和finally块都抛出异常，finally块中的异常会抑制try块中的异常，导致原始异常信息丢失
2. **代码冗长**：当需要关闭多个资源时，嵌套的try-finally结构会使代码难以阅读和维护
3. **容易忘记关闭资源**：开发者可能会忘记在finally块中关闭资源

## try-with-resources的优势

```java
// 使用try-with-resources的方式
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

**优势：**
1. **自动关闭资源**：无需显式调用close()方法
2. **异常处理更完善**：保留原始异常，将关闭资源时的异常作为被抑制的异常（suppressed exception）添加到原始异常
3. **代码简洁**：减少了样板代码，提高了可读性
4. **减少错误**：消除了忘记关闭资源的可能性

## 处理多个资源

try-with-resources可以优雅地处理多个资源：

```java
// 处理多个资源
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

而传统方式则需要嵌套多层try-finally：

```java
// 传统方式处理多个资源 - 代码冗长且容易出错
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

## 结合catch子句

try-with-resources也可以与catch和finally子句结合使用：

```java
// 结合catch子句的try-with-resources
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

## 自定义AutoCloseable资源

任何实现了`AutoCloseable`或`Closeable`接口的类都可以在try-with-resources中使用：

```java
// 自定义AutoCloseable资源
public class MyResource implements AutoCloseable {
    private final String name;
    
    public MyResource(String name) {
        this.name = name;
        System.out.println("Resource " + name + " created");
    }
    
    public void doSomething() {
        System.out.println("Resource " + name + " doing something");
    }
    
    @Override
    public void close() {
        System.out.println("Resource " + name + " closed");
    }
    
    public static void main(String[] args) {
        try (MyResource r1 = new MyResource("r1");
             MyResource r2 = new MyResource("r2")) {
            r1.doSomething();
            r2.doSomething();
            // 资源会按照创建的相反顺序关闭：先r2，后r1
        }
    }
}
```

## 异常抑制机制详解

Java 7的try-with-resources引入了异常抑制机制：

```java
// 演示异常抑制机制
public static void main(String[] args) {
    try {
        try (ExceptionResource resource = new ExceptionResource()) {
            throw new RuntimeException("Initial exception");
        }
    } catch (Exception e) {
        System.out.println("Main exception: " + e.getMessage());
        
        // 获取被抑制的异常
        Throwable[] suppressedExceptions = e.getSuppressed();
        System.out.println("Suppressed exceptions count: " + suppressedExceptions.length);
        for (Throwable suppressed : suppressedExceptions) {
            System.out.println("Suppressed: " + suppressed.getMessage());
        }
    }
}

static class ExceptionResource implements AutoCloseable {
    @Override
    public void close() {
        throw new RuntimeException("Exception during close()");
    }
}
```

输出结果：
```
Main exception: Initial exception
Suppressed exceptions count: 1
Suppressed: Exception during close()
```

## 使用场景

try-with-resources适用于所有需要关闭资源的场景，特别是：

1. **文件操作**：FileInputStream, FileOutputStream, FileReader, FileWriter等
2. **数据库操作**：Connection, Statement, ResultSet等
3. **网络操作**：Socket, ServerSocket等
4. **其他I/O资源**：InputStream, OutputStream, Reader, Writer的各种实现
5. **自定义资源**：任何实现了AutoCloseable接口的自定义类

## 注意事项

1. **资源关闭顺序**：资源会按照创建的相反顺序关闭
2. **Java版本要求**：需要Java 7或更高版本
3. **资源声明位置**：资源必须在try语句中声明和初始化
4. **final或effectively final**：在Java 9之前，try-with-resources中的资源变量必须是新创建的；Java 9后，可以使用final或effectively final的变量

Java 9增强版本示例：

```java
// Java 9增强版本 - 可以使用外部声明的资源
public static void main(String[] args) throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
    // reader是effectively final的
    try (reader) {
        // 使用reader
        String line = reader.readLine();
        System.out.println(line);
    }
}
```

## 总结

| 特性 | try-finally | try-with-resources |
|------|------------|-------------------|
| 代码简洁性 | 冗长，特别是处理多个资源时 | 简洁，易于阅读和维护 |
| 异常处理 | 可能掩盖原始异常 | 保留原始异常，支持异常抑制机制 |
| 资源管理 | 手动关闭，容易遗漏 | 自动关闭，更可靠 |
| 可读性 | 嵌套结构降低可读性 | 扁平结构提高可读性 |
| 维护性 | 较差 | 较好 |

**结论**：在处理必须关闭的资源时，应当始终优先使用try-with-resources而非try-finally。它不仅使代码更简洁、更清晰，还能更好地处理异常，提供更可靠的资源管理。

## 练习题

1. 编写一个使用try-with-resources同时处理文件读取和写入的程序。
2. 创建一个自定义的AutoCloseable资源，并在close()方法中抛出异常，然后在try块中也抛出异常，观察异常的处理情况。
3. 将一个使用嵌套try-finally结构的代码重构为使用try-with-resources的版本。

### 练习题答案

**练习1答案**：

```java
public static void copyFile(String source, String destination) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(source));
         BufferedWriter writer = new BufferedWriter(new FileWriter(destination))) {
        
        String line;
        while ((line = reader.readLine()) != null) {
            writer.write(line);
            writer.newLine();
        }
        System.out.println("File copied successfully");
    }
}
```

**练习2答案**：

```java
public static void main(String[] args) {
    try {
        try (CustomResource resource = new CustomResource()) {
            System.out.println("Using resource");
            throw new RuntimeException("Exception in try block");
        }
    } catch (Exception e) {
        System.out.println("Caught exception: " + e.getMessage());
        
        for (Throwable suppressed : e.getSuppressed()) {
            System.out.println("Suppressed: " + suppressed.getMessage());
        }
    }
}

static class CustomResource implements AutoCloseable {
    public CustomResource() {
        System.out.println("Resource created");
    }
    
    @Override
    public void close() {
        System.out.println("Closing resource");
        throw new RuntimeException("Exception in close method");
    }
}
```

**练习3答案**：

原始代码：
```java
public static String processFile(String path) throws IOException {
    BufferedReader br = null;
    BufferedWriter bw = null;
    try {
        br = new BufferedReader(new FileReader(path));
        bw = new BufferedWriter(new FileWriter(path + ".processed"));
        String line;
        StringBuilder result = new StringBuilder();
        while ((line = br.readLine()) != null) {
            String processed = line.toUpperCase();
            bw.write(processed);
            bw.newLine();
            result.append(processed).append("\n");
        }
        return result.toString();
    } finally {
        if (br != null) {
            try {
                br.close();
            } catch (IOException e) {
                // 处理关闭异常
            }
        }
        if (bw != null) {
            try {
                bw.close();
            } catch (IOException e) {
                // 处理关闭异常
            }
        }
    }
}
```

重构后：
```java
public static String processFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path));
         BufferedWriter bw = new BufferedWriter(new FileWriter(path + ".processed"))) {
        
        String line;
        StringBuilder result = new StringBuilder();
        while ((line = br.readLine()) != null) {
            String processed = line.toUpperCase();
            bw.write(processed);
            bw.newLine();
            result.append(processed).append("\n");
        }
        return result.toString();
    }
}
```
