 # 条目12：总是重写toString方法

## 核心原则

虽然重写`toString`方法不像重写`equals`和`hashCode`那样是强制性的，但提供一个良好的`toString`实现可以使你的类更易于使用、调试和维护。当对象被打印、连接到字符串或在断言失败时显示，都会自动调用`toString`方法。

## toString方法的通用约定 {id="tostring_1"}

1. **简洁性**：返回一个简洁、信息丰富、易于阅读的字符串表示。
2. **完整性**：字符串应包含对象中所有值得关注的信息。
3. **格式**：建议使用一种标准的、清晰的格式，最好在文档中说明。

## 默认toString的问题 {id="tostring_2"}

```java
// 默认toString的输出示例
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    // 没有重写toString方法！
}

// 使用默认toString的输出
PhoneNumber phone = new PhoneNumber(707, 867, 5309);
System.out.println(phone); // 输出类似：PhoneNumber@1b6d3586
```

默认的`toString`实现只提供类名和对象的哈希码，这对调试和日志记录几乎没有帮助。

## 正确实现toString的方法 {id="tostring_3"}

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
    public String toString() {
        return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNum);
    }
}
```

### 2. 包含所有重要信息的实现 {id="2_1"}

```java
public class User {
    private final String username;
    private final String email;
    private final boolean active;
    
    // 构造函数略
    
    @Override
    public String toString() {
        return String.format("User{username='%s', email='%s', active=%b}", 
                             username, email, active);
    }
}
```

### 3. 处理复杂对象 {id="3_1"}

```java
public class Department {
    private final String name;
    private final List<Employee> employees;
    
    // 构造函数略
    
    @Override
    public String toString() {
        return String.format("Department{name='%s', employeeCount=%d}", 
                             name, employees.size());
    }
}
```

## 格式化选项

### 1. 简单连接 {id="1_2"}

```java
@Override
public String toString() {
    return "Point{x=" + x + ", y=" + y + "}";
}
```

### 2. StringBuilder

```java
@Override
public String toString() {
    StringBuilder result = new StringBuilder("Customer{");
    result.append("id='").append(id).append('\'');
    result.append(", name='").append(name).append('\'');
    result.append(", email='").append(email).append('\'');
    result.append('}');
    return result.toString();
}
```

### 3. String.format或MessageFormat

```java
@Override
public String toString() {
    return String.format("Product[id=%s, name=%s, price=%.2f]", 
                         id, name, price);
}
```

### 4. StringJoiner (Java 8+)

```java
@Override
public String toString() {
    return new StringJoiner(", ", "Address[", "]")
            .add("street='" + street + "'")
            .add("city='" + city + "'")
            .add("zipCode='" + zipCode + "'")
            .toString();
}
```

## 特殊情况处理

### 1. 处理大型集合 {id="1_3"}

```java
public class Library {
    private final String name;
    private final List<Book> books;
    
    // 构造函数略
    
    @Override
    public String toString() {
        // 对于大型集合，只显示数量或前几个元素
        String bookSummary;
        if (books.size() <= 3) {
            bookSummary = books.toString();
        } else {
            bookSummary = books.subList(0, 3) + "... (and " + 
                          (books.size() - 3) + " more)";
        }
        return String.format("Library{name='%s', books=%s}", name, bookSummary);
    }
}
```

### 2. 处理循环引用

```java
public class Employee {
    private final String name;
    private Department department; // 可能形成循环引用
    
    // 构造函数略
    
    @Override
    public String toString() {
        return String.format("Employee{name='%s', department=%s}", 
                             name, 
                             department == null ? "null" : department.getName());
    }
}
```

### 3. 处理敏感信息

```java
public class CreditCard {
    private final String number;
    private final String cvv;
    private final String expiryDate;
    
    // 构造函数略
    
    @Override
    public String toString() {
        // 隐藏敏感信息
        return String.format("CreditCard{number='%s', expiryDate='%s'}", 
                             maskCardNumber(number), expiryDate);
    }
    
    private String maskCardNumber(String number) {
        if (number == null || number.length() < 4) {
            return "****";
        }
        return "****-****-****-" + number.substring(number.length() - 4);
    }
}
```

## 使用IDE生成toString

现代IDE可以自动生成toString方法：

```java
public class Product {
    private final String id;
    private final String name;
    private final double price;
    private final String category;
    
    // 构造函数略
    
    // IDE生成的toString方法
    @Override
    public String toString() {
        return "Product{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", category='" + category + '\'' +
                '}';
    }
}
```

## 使用Lombok简化

使用Lombok库可以通过注解自动生成toString方法：

```java
import lombok.ToString;

@ToString
public class Customer {
    private final String id;
    private final String name;
    private final String email;
    
    // 构造函数略
    // toString方法由Lombok自动生成
}
```

可以排除特定字段：

```java
import lombok.ToString;

@ToString(exclude = {"password", "creditCardNumber"})
public class User {
    private final String username;
    private final String password;      // 将被排除
    private final String email;
    private final String creditCardNumber; // 将被排除
    
    // 构造函数略
}
```

## 文档化toString的格式 {id="tostring_4"}

如果你的toString返回值有特定格式，应该在文档中说明：

```java
/**
 * 表示电话号码。
 * <p>
 * toString方法返回格式为"(xxx) xxx-xxxx"的字符串，
 * 其中x是数字。
 */
public class PhoneNumber {
    // 实现略
}
```

## 提供配套方法

如果toString提供了详细信息，考虑提供访问这些信息的方法：

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    // 构造函数略
    
    /**
     * 返回格式为"(xxx) xxx-xxxx"的电话号码字符串
     */
    @Override
    public String toString() {
        return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNum);
    }
    
    // 提供访问方法
    public int getAreaCode() { return areaCode; }
    public int getPrefix() { return prefix; }
    public int getLineNumber() { return lineNum; }
}
```

## 测试toString方法

```java
public class ToStringTest {
    public static void main(String[] args) {
        // 测试基本实现
        PhoneNumber phone = new PhoneNumber(707, 867, 5309);
        System.out.println("Phone: " + phone);
        
        // 测试在日志中的使用
        Logger logger = Logger.getLogger("test");
        logger.info("Processing phone: " + phone);
        
        // 测试在异常消息中的使用
        try {
            if (phone.getAreaCode() > 999) {
                throw new IllegalArgumentException("Invalid phone: " + phone);
            }
        } catch (Exception e) {
            System.err.println(e.getMessage());
        }
        
        // 测试在集合中的显示
        List<PhoneNumber> phones = Arrays.asList(
            new PhoneNumber(123, 456, 7890),
            new PhoneNumber(800, 555, 1212)
        );
        System.out.println("Phone list: " + phones);
    }
}
```

## 总结

1. **为什么重写toString**：
   - 提高可调试性和可用性
   - 使日志和错误消息更有意义
   - 简化开发和调试过程

2. **toString方法应该**：
   - 包含对象的所有重要信息
   - 格式清晰易读
   - 在文档中说明格式（如果有特定格式）
   - 考虑安全性，不显示敏感信息

3. **实现技巧**：
   - 使用String.format或StringBuilder提高效率
   - 处理循环引用以避免StackOverflowError
   - 对于大型集合，只显示摘要信息
   - 考虑使用IDE或Lombok自动生成

4. **最佳实践**：
   - 在开发早期就实现toString
   - 随着类的演变更新toString
   - 为toString显示的信息提供访问方法
   - 在单元测试中验证toString的输出

良好的toString实现虽然不是必需的，但它能显著提高代码的可维护性和开发效率，是一个值得养成的良好习惯。