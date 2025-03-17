 # 条目14：考虑实现Comparable接口

## 核心原则

`Comparable` 接口提供了对象的自然排序功能。实现 `Comparable` 接口的类表明其实例具有内在的排序关系。通过实现 `Comparable` 接口，可以让你的类与依赖排序的众多泛型算法和集合实现无缝协作。

## Comparable接口的定义 {id="comparable_1"}

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

`compareTo` 方法的通用约定：

1. **对称性**：`sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`，其中 sgn 是符号函数。
2. **传递性**：如果 `x.compareTo(y) > 0` 且 `y.compareTo(z) > 0`，则 `x.compareTo(z) > 0`。
3. **一致性**：如果 `x.compareTo(y) == 0`，则对于任何 z，`sgn(x.compareTo(z)) == sgn(y.compareTo(z))`。
4. **建议但非必需**：`(x.compareTo(y) == 0) == (x.equals(y))`。

## 基本实现方式

### 1. 简单类型的比较 {id="1_1"}

```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public int compareTo(PhoneNumber pn) {
        // 按照重要性顺序比较字段
        int result = Integer.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Integer.compare(prefix, pn.prefix);
            if (result == 0) {
                result = Integer.compare(lineNum, pn.lineNum);
            }
        }
        return result;
    }
}
```

### 2. 使用比较器构造方法 (Java 8+) {id="2_1"}

```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public int compareTo(PhoneNumber pn) {
        return Comparator.comparingInt((PhoneNumber p) -> p.areaCode)
                .thenComparingInt(p -> p.prefix)
                .thenComparingInt(p -> p.lineNum)
                .compare(this, pn);
    }
}
```

## 处理特殊情况

### 1. 处理浮点值 {id="1_2"}

```java
public class Point implements Comparable<Point> {
    private final double x;
    private final double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public int compareTo(Point p) {
        // 使用Double.compare避免浮点减法的危险
        int result = Double.compare(x, p.x);
        if (result == 0) {
            result = Double.compare(y, p.y);
        }
        return result;
    }
}
```

### 2. 处理可能为null的字段 {id="2_2"}

```java
public class Person implements Comparable<Person> {
    private final String lastName;
    private final String firstName;
    private final String middleName; // 可能为null
    
    // 构造函数略
    
    @Override
    public int compareTo(Person p) {
        int result = lastName.compareTo(p.lastName);
        if (result == 0) {
            result = firstName.compareTo(p.firstName);
            if (result == 0) {
                // 处理可能为null的字段
                if (middleName == null && p.middleName == null) {
                    return 0;
                } else if (middleName == null) {
                    return -1;
                } else if (p.middleName == null) {
                    return 1;
                } else {
                    return middleName.compareTo(p.middleName);
                }
            }
        }
        return result;
    }
}
```

### 3. 使用Comparator.nullsFirst/nullsLast (Java 8+) {id="3_1"}

```java
public class Person implements Comparable<Person> {
    private final String lastName;
    private final String firstName;
    private final String middleName; // 可能为null
    
    // 构造函数略
    
    @Override
    public int compareTo(Person p) {
        return Comparator.comparing((Person person) -> person.lastName)
                .thenComparing(person -> person.firstName)
                .thenComparing(person -> person.middleName, Comparator.nullsFirst(String::compareTo))
                .compare(this, p);
    }
}
```

## 避免常见错误

### 1. 避免使用减法 {id="1_3"}

```java
// 错误示例：使用减法可能导致整数溢出
public class BadComparable implements Comparable<BadComparable> {
    private final int value;
    
    public BadComparable(int value) {
        this.value = value;
    }
    
    @Override
    public int compareTo(BadComparable bc) {
        // 危险：可能导致整数溢出
        return value - bc.value;
    }
}

// 正确示例：使用静态比较方法
public class GoodComparable implements Comparable<GoodComparable> {
    private final int value;
    
    public GoodComparable(int value) {
        this.value = value;
    }
    
    @Override
    public int compareTo(GoodComparable gc) {
        // 安全：使用静态比较方法
        return Integer.compare(value, gc.value);
    }
}
```

### 2. 保持与equals一致 {id="2_3"}

```java
// 错误示例：compareTo与equals不一致
public class InconsistentComparable implements Comparable<InconsistentComparable> {
    private final int id;
    private final String name;
    
    // 构造函数略
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof InconsistentComparable)) return false;
        InconsistentComparable that = (InconsistentComparable) o;
        return id == that.id && Objects.equals(name, that.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
    
    @Override
    public int compareTo(InconsistentComparable ic) {
        // 只比较id，忽略name，与equals不一致
        return Integer.compare(id, ic.id);
    }
}
```

### 3. 正确处理继承关系 {id="3_2"}

```java
public class Animal implements Comparable<Animal> {
    private final String species;
    
    public Animal(String species) {
        this.species = species;
    }
    
    @Override
    public int compareTo(Animal a) {
        return species.compareTo(a.species);
    }
}

// 错误示例：违反里氏替换原则
public class Dog extends Animal implements Comparable<Dog> {
    private final String breed;
    
    public Dog(String breed) {
        super("Dog");
        this.breed = breed;
    }
    
    // 错误：重新声明了Comparable<Dog>
    @Override
    public int compareTo(Dog d) {
        return breed.compareTo(d.breed);
    }
}
```

## 高级用法

### 1. 使用Comparator作为比较策略 {id="1_4"}

```java
public class Student {
    private final String name;
    private final double gpa;
    private final int graduationYear;
    
    // 构造函数略
    
    // 不同的比较策略
    public static final Comparator<Student> BY_NAME = 
            Comparator.comparing(s -> s.name);
    
    public static final Comparator<Student> BY_GPA = 
            Comparator.comparingDouble(s -> s.gpa).reversed(); // 降序
    
    public static final Comparator<Student> BY_GRADUATION_YEAR = 
            Comparator.comparingInt(s -> s.graduationYear);
    
    // 组合比较器
    public static final Comparator<Student> BY_GRADUATION_THEN_GPA = 
            BY_GRADUATION_YEAR.thenComparing(BY_GPA);
}
```

### 2. 实现Comparable并提供额外比较器 {id="2_4"}

```java
public class Book implements Comparable<Book> {
    private final String title;
    private final String author;
    private final int publicationYear;
    
    // 构造函数略
    
    // 默认按标题排序
    @Override
    public int compareTo(Book b) {
        return title.compareTo(b.title);
    }
    
    // 额外的比较策略
    public static final Comparator<Book> BY_AUTHOR = 
            Comparator.comparing(b -> b.author);
    
    public static final Comparator<Book> BY_PUBLICATION_YEAR = 
            Comparator.comparingInt(b -> b.publicationYear);
}
```

### 3. 使用比较链 (Java 8+) {id="3_3"}

```java
public class Employee implements Comparable<Employee> {
    private final int id;
    private final String name;
    private final Department department;
    private final double salary;
    
    // 构造函数略
    
    @Override
    public int compareTo(Employee e) {
        return Comparator
                .<Employee, Department>comparing(emp -> emp.department, 
                        Comparator.comparing(Department::getName))
                .thenComparing(Employee::getSalary, Comparator.reverseOrder())
                .thenComparing(Employee::getName)
                .thenComparingInt(Employee::getId)
                .compare(this, e);
    }
}
```

## 使用Comparable的集合和算法 {id="comparable_2"}

### 1. 排序集合 {id="1_5"}

```java
public class ComparableDemo {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "apple", "pear", "orange");
        Collections.sort(words); // 使用自然排序
        System.out.println(words); // [apple, banana, orange, pear]
        
        // 使用TreeSet自动排序
        Set<String> wordSet = new TreeSet<>(words);
        System.out.println(wordSet); // [apple, banana, orange, pear]
    }
}
```

### 2. 查找和范围操作 {id="2_5"}

```java
public class RangeOperations {
    public static void main(String[] args) {
        NavigableSet<Integer> set = new TreeSet<>();
        for (int i = 0; i < 100; i += 10) {
            set.add(i);
        }
        
        // 范围查找
        System.out.println(set.floor(25));    // 返回20
        System.out.println(set.ceiling(25));  // 返回30
        System.out.println(set.higher(50));   // 返回60
        System.out.println(set.lower(50));    // 返回40
        
        // 范围视图
        System.out.println(set.headSet(50));  // [0, 10, 20, 30, 40]
        System.out.println(set.tailSet(50));  // [50, 60, 70, 80, 90]
        System.out.println(set.subSet(20, 60)); // [20, 30, 40, 50]
    }
}
```

### 3. 极值操作 {id="3_4"}

```java
public class ExtremumOperations {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(5, 2, 9, 1, 7, 3);
        
        // 查找最小值和最大值
        System.out.println(Collections.min(numbers)); // 1
        System.out.println(Collections.max(numbers)); // 9
        
        // 使用自定义比较器
        Comparator<Integer> reverseOrder = Comparator.reverseOrder();
        System.out.println(Collections.min(numbers, reverseOrder)); // 9
        System.out.println(Collections.max(numbers, reverseOrder)); // 1
    }
}
```

## 与其他接口的关系

### 1. Comparable与equals {id="1_6"}

```java
public class ComparableVsEquals {
    public static void main(String[] args) {
        BigDecimal bd1 = new BigDecimal("1.0");
        BigDecimal bd2 = new BigDecimal("1.00");
        
        // BigDecimal的equals要求精确匹配，包括小数位数
        System.out.println("bd1.equals(bd2): " + bd1.equals(bd2)); // false
        
        // 但compareTo只比较数值
        System.out.println("bd1.compareTo(bd2): " + bd1.compareTo(bd2)); // 0
        
        // 这会导致在不同集合中的行为不同
        Set<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(bd1);
        hashSet.add(bd2);
        System.out.println("HashSet size: " + hashSet.size()); // 2
        
        Set<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(bd1);
        treeSet.add(bd2);
        System.out.println("TreeSet size: " + treeSet.size()); // 1
    }
}
```

### 2. Comparable与Comparator {id="2_6"}

```java
public class ComparableVsComparator {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "apple", "PEAR", "Orange");
        
        // 使用自然排序（区分大小写）
        Collections.sort(words);
        System.out.println("Natural order: " + words); 
        // [PEAR, Orange, apple, banana]
        
        // 使用自定义比较器（不区分大小写）
        Collections.sort(words, String.CASE_INSENSITIVE_ORDER);
        System.out.println("Case insensitive: " + words); 
        // [apple, banana, Orange, PEAR]
        
        // 使用自定义比较器（按长度）
        Collections.sort(words, Comparator.comparing(String::length));
        System.out.println("By length: " + words); 
        // [PEAR, apple, Orange, banana]
    }
}
```

## 测试Comparable实现

```java
public class ComparableTest {
    public static void main(String[] args) {
        // 创建测试对象
        PhoneNumber pn1 = new PhoneNumber(707, 867, 5309);
        PhoneNumber pn2 = new PhoneNumber(707, 867, 5309);
        PhoneNumber pn3 = new PhoneNumber(707, 867, 5310);
        PhoneNumber pn4 = new PhoneNumber(415, 555, 1212);
        
        // 测试相等性
        System.out.println("pn1.compareTo(pn2): " + pn1.compareTo(pn2)); // 应为0
        
        // 测试顺序
        System.out.println("pn1.compareTo(pn3): " + pn1.compareTo(pn3)); // 应为负数
        System.out.println("pn3.compareTo(pn1): " + pn3.compareTo(pn1)); // 应为正数
        
        // 测试传递性
        System.out.println("pn1.compareTo(pn4): " + pn1.compareTo(pn4)); // 应为正数
        System.out.println("pn4.compareTo(pn3): " + pn4.compareTo(pn3)); // 应为负数
        
        // 测试与equals的一致性
        System.out.println("pn1.equals(pn2): " + pn1.equals(pn2));
        System.out.println("(pn1.compareTo(pn2) == 0) == pn1.equals(pn2): " 
                + ((pn1.compareTo(pn2) == 0) == pn1.equals(pn2)));
    }
}
```

## 总结

1. **为什么实现Comparable**：
   - 使类的实例能够自动排序
   - 与Java集合框架无缝协作
   - 使用各种通用算法（如搜索和极值计算）
   - 使类能够用作有序集合的键（如TreeSet、TreeMap）

2. **实现Comparable的关键点**：
   - 遵守compareTo方法的通用约定
   - 确保排序与equals一致（强烈建议）
   - 使用静态比较方法而非减法
   - 考虑使用比较器构造方法（Java 8+）

3. **实现技巧**：
   - 按照重要性顺序比较字段
   - 对于引用类型，处理null值
   - 对于数值类型，使用静态比较方法
   - 对于浮点类型，使用Double.compare或Float.compare

4. **最佳实践**：
   - 为值类实现Comparable接口
   - 考虑提供多个比较策略
   - 使用Comparator.comparing方法构建复杂比较器
   - 测试compareTo方法的所有约定

实现Comparable接口是一种强大的方式，可以使你的类与Java平台的许多通用算法和集合实现无缝协作。通过遵循本文档中的最佳实践，你可以确保你的类具有一致、可靠的排序行为。