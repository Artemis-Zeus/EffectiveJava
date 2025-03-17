# 条目11：重写equals方法时总是重写hashCode方法

## 核心原则

如果重写了`equals`方法，就必须同时重写`hashCode`方法。否则，将违反`Object.hashCode`方法的通用约定，导致该类无法与基于哈希的集合（如`HashMap`、`HashSet`和`Hashtable`）一起正常工作。

## hashCode方法的通用约定 {id="hashcode_1"}

1. **一致性**：在应用程序执行期间，如果对象的equals方法比较的信息没有被修改，则对同一个对象多次调用hashCode方法必须返回相同的整数。
2. **相等性**：如果两个对象根据equals方法比较是相等的，那么这两个对象的hashCode方法必须返回相同的整数。
3. **不要求不相等对象返回不同哈希码**：不相等的对象可以返回相同的哈希码，但为了提高哈希表性能，应该尽量避免。

## 违反约定的后果

```java
// 错误示例：只重写equals而不重写hashCode
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return areaCode == pn.areaCode && 
               prefix == pn.prefix && 
               lineNum == pn.lineNum;
    }
    
    // 没有重写hashCode方法！
}
```

使用上述类的问题：

```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(707, 867, 5309), "Jenny");

// 这将返回null，尽管我们刚刚添加了一个"相等"的键
String name = map.get(new PhoneNumber(707, 867, 5309));
```

## 正确实现hashCode的方法

### 1. 基本实现方式 {id="1_1"}

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return areaCode == pn.areaCode && 
               prefix == pn.prefix && 
               lineNum == pn.lineNum;
    }
    
    @Override
    public int hashCode() {
        int result = 17; // 选择一个非零的初始值
        result = 31 * result + areaCode; // 31是一个奇素数，有良好的哈希分布特性
        result = 31 * result + prefix;
        result = 31 * result + lineNum;
        return result;
    }
}
```

### 2. 使用Objects.hash方法

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    // 构造函数和equals方法同上
    
    @Override
    public int hashCode() {
        return Objects.hash(areaCode, prefix, lineNum);
    }
}
```

### 3. 处理特殊类型字段

```java
public class ComplexObject {
    private final String name;
    private final int[] values;
    private final float price;
    private final Object reference;
    
    // 构造函数和equals方法略
    
    @Override
    public int hashCode() {
        int result = 17;
        // String类型
        result = 31 * result + (name == null ? 0 : name.hashCode());
        // 数组类型
        result = 31 * result + Arrays.hashCode(values);
        // float类型
        result = 31 * result + Float.floatToIntBits(price);
        // 引用类型
        result = 31 * result + (reference == null ? 0 : reference.hashCode());
        return result;
    }
}
```

## 性能优化技巧

### 1. 延迟初始化哈希码 {id="1_2"}

```java
public class LazyHashCode {
    private final String field1;
    private final String field2;
    
    // 缓存哈希码
    private int hashCode;
    
    // 构造函数和equals方法略
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) { // 首次计算
            result = 17;
            result = 31 * result + (field1 == null ? 0 : field1.hashCode());
            result = 31 * result + (field2 == null ? 0 : field2.hashCode());
            hashCode = result; // 缓存结果
        }
        return result;
    }
}
```

### 2. 不可变对象的哈希码缓存 {id="2_1"}

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;
    
    // 在构造时计算哈希码
    private final int hashCode;
    
    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
        // 在构造时计算并缓存哈希码
        int result = 17;
        result = 31 * result + x;
        result = 31 * result + y;
        this.hashCode = result;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ImmutablePoint)) return false;
        ImmutablePoint p = (ImmutablePoint) o;
        return x == p.x && y == p.y;
    }
    
    @Override
    public int hashCode() {
        return hashCode; // 直接返回缓存的哈希码
    }
}
```

## 常见错误和陷阱

### 1. 包含不必要的字段

```java
// 错误示例：包含不参与equals比较的字段
public class User {
    private final String username; // 用于equals比较
    private final String password; // 用于equals比较
    private long lastLoginTime;    // 不用于equals比较
    
    // 构造函数略
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(username, user.username) && 
               Objects.equals(password, user.password);
        // lastLoginTime不参与equals比较
    }
    
    @Override
    public int hashCode() {
        // 错误：包含了lastLoginTime
        return Objects.hash(username, password, lastLoginTime);
    }
}
```

### 2. 正确的实现

```java
public class User {
    private final String username;
    private final String password;
    private long lastLoginTime;
    
    // 构造函数略
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(username, user.username) && 
               Objects.equals(password, user.password);
    }
    
    @Override
    public int hashCode() {
        // 正确：只包含参与equals比较的字段
        return Objects.hash(username, password);
    }
}
```

## 使用IDE生成hashCode和equals

现代IDE（如IntelliJ IDEA、Eclipse）可以自动生成符合规范的equals和hashCode方法：

```java
public class Product {
    private final String id;
    private final String name;
    private final double price;
    private final String category;
    
    // 构造函数略
    
    // IDE生成的equals方法
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Product product = (Product) o;
        return Double.compare(product.price, price) == 0 &&
                Objects.equals(id, product.id) &&
                Objects.equals(name, product.name) &&
                Objects.equals(category, product.category);
    }
    
    // IDE生成的hashCode方法
    @Override
    public int hashCode() {
        return Objects.hash(id, name, price, category);
    }
}
```

## 使用Lombok简化

使用Lombok库可以通过注解自动生成equals和hashCode方法：

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class Customer {
    private final String id;
    private final String name;
    private final String email;
    
    // 构造函数略
    // equals和hashCode方法由Lombok自动生成
}
```

## 测试equals和hashCode

```java
public class EqualsHashCodeTest {
    public static void main(String[] args) {
        // 创建两个相等的对象
        PhoneNumber phone1 = new PhoneNumber(707, 867, 5309);
        PhoneNumber phone2 = new PhoneNumber(707, 867, 5309);
        
        // 测试equals的对称性
        System.out.println("phone1.equals(phone2): " + phone1.equals(phone2));
        System.out.println("phone2.equals(phone1): " + phone2.equals(phone1));
        
        // 测试hashCode的一致性
        System.out.println("phone1.hashCode(): " + phone1.hashCode());
        System.out.println("phone2.hashCode(): " + phone2.hashCode());
        
        // 测试在哈希集合中的行为
        Map<PhoneNumber, String> map = new HashMap<>();
        map.put(phone1, "Jenny");
        System.out.println("map.get(phone2): " + map.get(phone2));
        
        Set<PhoneNumber> set = new HashSet<>();
        set.add(phone1);
        System.out.println("set.contains(phone2): " + set.contains(phone2));
    }
}
```

## 总结

1. **重写equals必须重写hashCode**：这是Java对象约定的基本要求。

2. **hashCode方法必须满足的条件**：
    - 对于相等的对象必须产生相同的哈希码
    - 哈希码计算应该只包含equals方法中使用的字段
    - 哈希码计算应该高效且分布均匀

3. **实现技巧**：
    - 使用31作为乘法因子
    - 考虑使用Objects.hash简化实现
    - 对于不可变对象，考虑缓存哈希码
    - 对于频繁使用的可变对象，考虑延迟初始化

4. **工具支持**：
    - 使用IDE自动生成
    - 使用Lombok等库通过注解生成

遵循这些原则，可以确保你的类能够正确地与Java集合框架协同工作，特别是基于哈希的集合类。
