 # 条目13：谨慎重写clone方法

## 核心原则

`Cloneable` 接口和 `Object.clone()` 方法的设计存在缺陷，在实现对象拷贝时应当谨慎使用。在大多数情况下，应该考虑使用复制构造函数或复制工厂方法作为替代方案。如果确实需要实现 `Cloneable` 接口，请务必正确地重写 `clone` 方法。

## Cloneable接口的特殊性 {id="cloneable_1"}

`Cloneable` 接口不同于普通接口，它不声明任何方法，而是决定了 `Object` 类中受保护的 `clone` 方法的行为：

1. 如果一个类实现了 `Cloneable`，`Object.clone()` 方法将返回该对象的逐字段拷贝。
2. 如果一个类没有实现 `Cloneable`，调用 `clone()` 方法将抛出 `CloneNotSupportedException`。

这种接口用法违反了接口的一般目的（定义类可以实现的方法类型），是一种不应效仿的设计。

## Object.clone方法的通用约定 {id="clone_1"}

1. **创建并返回对象的拷贝**：`x.clone() != x` 应该为 `true`。
2. **等价性**：`x.clone().getClass() == x.getClass()` 应该为 `true`。
3. **相等性**：`x.clone().equals(x)` 通常为 `true`，但不是必需的。
4. **独立性**：克隆对象的修改不应影响原始对象，反之亦然。

## 基本实现方式

### 1. 简单不可变类 {id="1_1"}

对于简单的不可变类，通常不需要实现 `Cloneable`：

```java
public final class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    // 不实现Cloneable，而是提供复制构造函数
    public PhoneNumber(PhoneNumber phone) {
        this(phone.areaCode, phone.prefix, phone.lineNum);
    }
    
    // 或者提供静态工厂方法
    public static PhoneNumber valueOf(PhoneNumber phone) {
        return new PhoneNumber(phone.areaCode, phone.prefix, phone.lineNum);
    }
}
```

### 2. 简单可变类的clone实现 {id="2_1"}

```java
public class Point implements Cloneable {
    private int x;
    private int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public Point clone() {
        try {
            return (Point) super.clone();
        } catch (CloneNotSupportedException e) {
            // 不应该发生，因为我们实现了Cloneable
            throw new AssertionError();
        }
    }
    
    // getter和setter方法略
}
```

## 处理包含引用的类

### 1. 浅拷贝问题 {id="1_2"}

```java
// 错误示例：只进行浅拷贝
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    
    // 构造函数和其他方法略
    
    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // 问题：result和原始对象共享elements数组
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 2. 正确实现深拷贝 {id="2_2"}

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    
    // 构造函数和其他方法略
    
    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // 深拷贝数组
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 3. 处理复杂对象图 {id="3_1"}

```java
public class HashTable implements Cloneable {
    private Entry[] buckets;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        // 递归复制链表
        Entry deepCopy() {
            return new Entry(key, value,
                    next == null ? null : next.deepCopy());
        }
    }
    
    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            
            // 深拷贝每个桶中的链表
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 4. 避免递归导致的栈溢出 {id="4_1"}

```java
public class HashTable implements Cloneable {
    private Entry[] buckets;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        // 构造函数略
        
        // 使用迭代而非递归复制链表
        Entry deepCopy() {
            Entry result = new Entry(key, value, null);
            Entry p = result;
            Entry n = next;
            while (n != null) {
                p.next = new Entry(n.key, n.value, null);
                p = p.next;
                n = n.next;
            }
            return result;
        }
    }
    
    // clone方法同上
}
```

## 特殊情况处理

### 1. 处理final字段 {id="1_3"}

```java
public class WithFinalField implements Cloneable {
    private final int id; // final字段
    private String name;  // 非final字段
    
    public WithFinalField(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    @Override
    public WithFinalField clone() {
        try {
            WithFinalField result = (WithFinalField) super.clone();
            // 无法修改final字段id，它将保持与原对象相同的值
            // 只能修改非final字段
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 2. 在子类中重写clone {id="2_3"}

```java
public class Shape implements Cloneable {
    private int x, y;
    
    // 构造函数略
    
    @Override
    public Shape clone() {
        try {
            return (Shape) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Circle extends Shape {
    private double radius;
    
    // 构造函数略
    
    @Override
    public Circle clone() {
        Circle result = (Circle) super.clone();
        // 处理Circle特有的字段
        return result;
    }
}
```

### 3. 线程安全考虑 {id="3_2"}

```java
public class ThreadSafeClass implements Cloneable {
    private final Object lock = new Object();
    private int[] data;
    
    // 构造函数略
    
    @Override
    public ThreadSafeClass clone() {
        try {
            ThreadSafeClass result = (ThreadSafeClass) super.clone();
            // 创建新的锁对象，避免与原对象共享
            result.lock = new Object();
            // 深拷贝数据
            synchronized (lock) {
                result.data = data.clone();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

## 替代方案

### 1. 复制构造函数 {id="1_4"}

```java
public class Rectangle {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    // 复制构造函数
    public Rectangle(Rectangle rect) {
        this.width = rect.width;
        this.height = rect.height;
    }
}
```

### 2. 复制工厂方法 {id="2_4"}

```java
public class Complex {
    private final double re;
    private final double im;
    
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    
    // 静态工厂方法
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    
    // 复制工厂方法
    public static Complex valueOf(Complex c) {
        return new Complex(c.re, c.im);
    }
}
```

### 3. 使用序列化 {id="3_3"}

```java
public class DeepCopy {
    // 使用序列化进行深拷贝
    public static <T extends Serializable> T deepCopy(T obj) {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(obj);
            oos.flush();
            
            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);
            return (T) ois.readObject();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 常见错误和陷阱

### 1. 忘记调用super.clone() {id="1_5"}

```java
// 错误示例：没有调用super.clone()
public class WrongClone implements Cloneable {
    private int value;
    
    @Override
    public WrongClone clone() {
        // 错误：直接创建新对象而不是调用super.clone()
        return new WrongClone();
    }
}
```

### 2. 忘记处理CloneNotSupportedException {id="2_5"}

```java
// 错误示例：没有正确处理异常
public class BadExceptionHandling implements Cloneable {
    private int value;
    
    @Override
    public BadExceptionHandling clone() {
        // 错误：将检查异常传播给调用者
        return (BadExceptionHandling) super.clone();
        // 应该捕获并处理CloneNotSupportedException
    }
}
```

### 3. 忘记转换返回类型 {id="3_4"}

```java
// 错误示例：没有转换返回类型
public class WrongReturnType implements Cloneable {
    private int value;
    
    @Override
    public Object clone() { // 应该返回WrongReturnType
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 4. 忘记实现Cloneable接口 {id="4_2"}

```java
// 错误示例：没有实现Cloneable接口
public class ForgotCloneable {
    private int value;
    
    @Override
    public ForgotCloneable clone() {
        try {
            // 将抛出CloneNotSupportedException
            return (ForgotCloneable) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

## 测试clone方法

```java
public class CloneTest {
    public static void main(String[] args) {
        // 测试基本克隆
        Point original = new Point(10, 20);
        Point clone = original.clone();
        
        System.out.println("Original: " + original);
        System.out.println("Clone: " + clone);
        System.out.println("original == clone: " + (original == clone));
        System.out.println("original.equals(clone): " + original.equals(clone));
        
        // 测试修改后的独立性
        clone.setX(30);
        System.out.println("After modification:");
        System.out.println("Original: " + original);
        System.out.println("Clone: " + clone);
        
        // 测试深拷贝
        Stack originalStack = new Stack();
        originalStack.push("test");
        Stack clonedStack = originalStack.clone();
        
        System.out.println("Original stack: " + originalStack);
        System.out.println("Cloned stack: " + clonedStack);
        
        // 修改克隆后的栈
        clonedStack.push("another");
        System.out.println("After modification:");
        System.out.println("Original stack: " + originalStack);
        System.out.println("Cloned stack: " + clonedStack);
    }
}
```

## 总结

1. **谨慎使用clone**：
   - `Cloneable` 接口和 `clone` 方法设计存在缺陷
   - 在大多数情况下，复制构造函数或复制工厂是更好的选择
   - 如果类已经实现了 `Cloneable`，新的子类可能别无选择，必须实现 `clone`

2. **正确实现clone的步骤**：
   - 实现 `Cloneable` 接口
   - 重写 `clone` 方法，访问修饰符为 `public`
   - 方法返回类型为类本身（协变返回类型）
   - 首先调用 `super.clone()`
   - 修复任何需要深拷贝的字段

3. **深拷贝注意事项**：
   - 所有包含引用的可变字段都需要深拷贝
   - 注意避免递归导致的栈溢出
   - 处理特殊情况，如 `final` 字段

4. **替代方案**：
   - 复制构造函数：`public Foo(Foo foo) { ... }`
   - 复制工厂方法：`public static Foo newInstance(Foo foo) { ... }`
   - 这些方案更简单、更安全、更灵活

5. **最佳实践**：
   - 对于新类，应避免实现 `Cloneable` 接口
   - 考虑使用复制构造函数或复制工厂方法
   - 如果必须实现 `clone`，确保正确处理深拷贝
   - 在文档中明确说明克隆的行为

总之，`clone` 方法应当谨慎使用，在大多数情况下，有更好的替代方案来实现对象拷贝。