# 条目10：覆盖equals时请遵守通用约定

## 代码示例

### 1. 错误的equals实现（不推荐）

```java
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    // 违反对称性的equals实现
    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Point) {
            Point other = (Point) obj;
            return this.x == other.x && this.y == other.y;
        }
        return false;
    }
}
```

### 2. 正确的equals实现（推荐）

```java
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Point)) {
            return false;
        }
        Point other = (Point) obj;
        return this.x == other.x && this.y == other.y;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

## 实现高质量equals方法的步骤

### 1. 使用==检查参数是否为对象的引用

```java
public class Employee {
    private final String id;
    private final String name;
    private final Department department;
    
    public Employee(String id, String name, Department department) {
        this.id = id;
        this.name = name;
        this.department = department;
    }
    
    @Override
    public boolean equals(Object obj) {
        // 性能优化：如果是同一个对象引用，直接返回true
        if (this == obj) {
            return true;
        }
        
        // 其他检查...
        return false;
    }
}
```

### 2. 使用instanceof检查参数类型

```java
public class Book {
    private final String isbn;
    private final String title;
    private final String author;
    
    public Book(String isbn, String title, String author) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        // 使用instanceof检查类型
        if (!(obj instanceof Book)) {
            return false;
        }
        
        // 其他比较...
        return false;
    }
}
```

### 3. 转换参数并比较关键域 {id="3_1"}

```java
public final class CreditCard {
    private final String number;
    private final String cardHolder;
    private final LocalDate expiryDate;
    
    public CreditCard(String number, String cardHolder, LocalDate expiryDate) {
        this.number = number;
        this.cardHolder = cardHolder;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof CreditCard)) {
            return false;
        }
        
        // 转换参数
        CreditCard other = (CreditCard) obj;
        
        // 比较关键域
        return Objects.equals(number, other.number) &&
               Objects.equals(cardHolder, other.cardHolder) &&
               Objects.equals(expiryDate, other.expiryDate);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(number, cardHolder, expiryDate);
    }
}
```

## 特殊情况处理

### 1. 处理null值

```java
public class OptionalField {
    private final String requiredField;
    private final String optionalField; // 可以为null
    
    public OptionalField(String requiredField, String optionalField) {
        this.requiredField = Objects.requireNonNull(requiredField);
        this.optionalField = optionalField;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof OptionalField)) {
            return false;
        }
        
        OptionalField other = (OptionalField) obj;
        return Objects.equals(requiredField, other.requiredField) &&
               Objects.equals(optionalField, other.optionalField);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(requiredField, optionalField);
    }
}
```

### 2. 处理数组字段

```java
public class ArrayContainer {
    private final int[] numbers;
    private final String[] strings;
    
    public ArrayContainer(int[] numbers, String[] strings) {
        this.numbers = numbers.clone();
        this.strings = strings.clone();
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof ArrayContainer)) {
            return false;
        }
        
        ArrayContainer other = (ArrayContainer) obj;
        return Arrays.equals(numbers, other.numbers) &&
               Arrays.equals(strings, other.strings);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(Arrays.hashCode(numbers), Arrays.hashCode(strings));
    }
}
```

### 3. 处理浮点数字段

```java
public class FloatingPoint {
    private final float floatField;
    private final double doubleField;
    
    public FloatingPoint(float floatField, double doubleField) {
        this.floatField = floatField;
        this.doubleField = doubleField;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof FloatingPoint)) {
            return false;
        }
        
        FloatingPoint other = (FloatingPoint) obj;
        return Float.compare(floatField, other.floatField) == 0 &&
               Double.compare(doubleField, other.doubleField) == 0;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(Float.floatToIntBits(floatField), 
                          Double.doubleToLongBits(doubleField));
    }
}
```

## 注意事项

1. **始终覆盖hashCode**
   - 当覆盖equals时，必须同时覆盖hashCode
   - 确保相等的对象具有相同的哈希码

2. **保持对称性**
   - equals必须是对称的：如果x.equals(y)返回true，那么y.equals(x)也必须返回true

3. **保持传递性**
   - 如果x.equals(y)返回true且y.equals(z)返回true，那么x.equals(z)必须返回true

4. **保持一致性**
   - 多次调用equals应该返回相同的结果，除非对象被修改

5. **不要将equals依赖于不可靠的资源**
   - equals的结果不应依赖于可能改变的外部状态

## 最佳实践

### 1. 使用IDE生成equals和hashCode

```java
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(name, person.name) &&
                Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }
}
```

### 2. 使用Builder模式的类的equals实现

```java
public class Address {
    private final String street;
    private final String city;
    private final String country;
    private final String postalCode;
    
    public static class Builder {
        private String street;
        private String city;
        private String country;
        private String postalCode;
        
        public Builder street(String street) {
            this.street = street;
            return this;
        }
        
        public Builder city(String city) {
            this.city = city;
            return this;
        }
        
        public Builder country(String country) {
            this.country = country;
            return this;
        }
        
        public Builder postalCode(String postalCode) {
            this.postalCode = postalCode;
            return this;
        }
        
        public Address build() {
            return new Address(this);
        }
    }
    
    private Address(Builder builder) {
        this.street = builder.street;
        this.city = builder.city;
        this.country = builder.country;
        this.postalCode = builder.postalCode;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Address)) {
            return false;
        }
        Address other = (Address) obj;
        return Objects.equals(street, other.street) &&
               Objects.equals(city, other.city) &&
               Objects.equals(country, other.country) &&
               Objects.equals(postalCode, other.postalCode);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(street, city, country, postalCode);
    }
}
```

## 总结

覆盖equals方法时，必须遵守以下原则：

1. **遵守通用约定**
   - 自反性
   - 对称性
   - 传递性
   - 一致性
   - 非空性

2. **正确实现步骤**
   - 使用==检查引用相等性
   - 使用instanceof检查类型
   - 转换参数类型
   - 比较关键域

3. **注意事项**
   - 始终同时覆盖hashCode
   - 考虑特殊字段的处理（null、数组、浮点数）
   - 保持方法的简单性和清晰性

4. **最佳实践**
   - 使用Objects.equals比较对象引用
   - 使用Arrays.equals比较数组
   - 使用Float.compare和Double.compare比较浮点数
   - 考虑使用IDE生成equals和hashCode实现

通过遵守这些原则和最佳实践，可以确保equals方法的正确性和可靠性，避免在使用集合类等场景中出现意外行为。