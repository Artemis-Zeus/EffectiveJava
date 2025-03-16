# 条目3：利用私有构造器或枚举类型强化Singleton属性

## 核心原则

**单例模式(Singleton)**是一种确保类只有一个实例，并提供全局访问点的设计模式。在Java中，通过私有构造器和枚举类型可以有效地实现和强化单例属性，防止多实例创建，同时保证线程安全和序列化安全。

## 示例代码

### 1. 传统单例实现方式（不推荐）

```java
public class BadSingleton {
    // 静态变量保存唯一实例
    private static BadSingleton instance = new BadSingleton();
    
    // 公有构造器（问题所在）
    public BadSingleton() {
        // 初始化代码
    }
    
    // 提供全局访问点
    public static BadSingleton getInstance() {
        return instance;
    }
}

// 使用方式
BadSingleton singleton = BadSingleton.getInstance();
// 但也可以直接创建新实例，破坏单例属性
BadSingleton anotherInstance = new BadSingleton(); // 单例被破坏!
```

### 2. 使用私有构造器实现单例（基本方法）

```java
public class Singleton {
    // 静态变量保存唯一实例
    private static final Singleton INSTANCE = new Singleton();
    
    // 私有构造器防止外部实例化
    private Singleton() {
        // 防止通过反射创建实例
        if (INSTANCE != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
        // 初始化代码
    }
    
    // 提供全局访问点
    public static Singleton getInstance() {
        return INSTANCE;
    }
    
    // 业务方法
    public void doSomething() {
        System.out.println("Singleton is doing something");
    }
}
```

### 3. 懒加载单例（延迟初始化）

```java
public class LazySingleton {
    // 使用volatile确保多线程可见性
    private static volatile LazySingleton instance;
    
    private LazySingleton() {
        // 防止通过反射创建实例
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
    }
    
    // 双重检查锁定模式
    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

### 4. 静态内部类实现单例（推荐）

```java
public class StaticHolderSingleton {
    // 私有构造器
    private StaticHolderSingleton() {
    }
    
    // 静态内部类持有单例实例
    private static class Holder {
        private static final StaticHolderSingleton INSTANCE = new StaticHolderSingleton();
    }
    
    // 全局访问点
    public static StaticHolderSingleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### 5. 枚举类型实现单例（最佳方式）

```java
public enum EnumSingleton {
    INSTANCE;
    
    // 可以添加实例变量
    private String config;
    
    // 可以有初始化代码
    EnumSingleton() {
        config = "Default configuration";
    }
    
    // 业务方法
    public void doSomething() {
        System.out.println("EnumSingleton is doing something with config: " + config);
    }
    
    // 设置配置
    public void setConfig(String config) {
        this.config = config;
    }
    
    // 获取配置
    public String getConfig() {
        return config;
    }
}

// 使用方式
EnumSingleton singleton = EnumSingleton.INSTANCE;
singleton.doSomething();
```

### 6. 序列化安全的单例（非枚举方式）

```java
public class SerializableSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private static final SerializableSingleton INSTANCE = new SerializableSingleton();
    
    private SerializableSingleton() {
        if (INSTANCE != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
    }
    
    public static SerializableSingleton getInstance() {
        return INSTANCE;
    }
    
    // 防止反序列化创建新实例
    private Object readResolve() {
        // 返回单例实例而不是新创建的对象
        return INSTANCE;
    }
}
```

## 优缺点分析

### 私有构造器实现单例

**优点：**
1. **防止外部实例化**：通过私有构造器阻止外部直接创建实例
2. **控制实例数量**：确保只有一个实例存在
3. **灵活性**：可以根据需要轻松改变为创建有限数量的实例

**缺点：**
1. **反射攻击风险**：可以通过反射API强制访问私有构造器
2. **序列化问题**：反序列化时可能创建新实例，除非实现readResolve方法
3. **线程安全问题**：需要额外处理多线程环境下的安全问题

### 枚举类型实现单例

**优点：**
1. **简洁**：代码量最少，最易实现
2. **序列化安全**：Java保证枚举类型的序列化机制是安全的
3. **反射安全**：枚举构造器受到JVM特殊保护，防止反射攻击
4. **线程安全**：枚举创建是线程安全的

**缺点：**
1. **灵活性较低**：不能延迟初始化
2. **继承限制**：枚举不能继承其他类（但可以实现接口）

## 使用场景

### 1. 资源管理器

```java
public enum DatabaseManager {
    INSTANCE;
    
    private Connection connection;
    
    DatabaseManager() {
        try {
            // 初始化数据库连接
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "password");
        } catch (SQLException e) {
            throw new RuntimeException("Failed to initialize database connection", e);
        }
    }
    
    public Connection getConnection() {
        return connection;
    }
    
    public void executeQuery(String sql) {
        try (Statement stmt = connection.createStatement()) {
            ResultSet rs = stmt.executeQuery(sql);
            // 处理结果集
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    public void close() {
        try {
            if (connection != null && !connection.isClosed()) {
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

// 使用方式
DatabaseManager.INSTANCE.executeQuery("SELECT * FROM users");
```

### 2. 配置管理器

```java
public enum ConfigManager {
    INSTANCE;
    
    private final Properties properties = new Properties();
    
    ConfigManager() {
        try (InputStream input = getClass().getClassLoader().getResourceAsStream("config.properties")) {
            if (input != null) {
                properties.load(input);
            }
        } catch (IOException e) {
            throw new RuntimeException("Failed to load configuration", e);
        }
    }
    
    public String getProperty(String key) {
        return properties.getProperty(key);
    }
    
    public void setProperty(String key, String value) {
        properties.setProperty(key, value);
    }
    
    public void saveProperties() {
        try (OutputStream output = new FileOutputStream("config.properties")) {
            properties.store(output, "Configuration updated");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// 使用方式
String dbUrl = ConfigManager.INSTANCE.getProperty("database.url");
```

### 3. 线程池管理器

```java
public enum ThreadPoolManager {
    INSTANCE;
    
    private final ExecutorService executorService;
    
    ThreadPoolManager() {
        executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
    }
    
    public void execute(Runnable task) {
        executorService.execute(task);
    }
    
    public <T> Future<T> submit(Callable<T> task) {
        return executorService.submit(task);
    }
    
    public void shutdown() {
        executorService.shutdown();
        try {
            if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
                executorService.shutdownNow();
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

// 使用方式
ThreadPoolManager.INSTANCE.execute(() -> {
    System.out.println("Task running in thread: " + Thread.currentThread().getName());
});
```

## 最佳实践样例

### 1. 日志管理器（枚举单例）

```java
public enum LoggerManager {
    INSTANCE;
    
    private final Map<String, Logger> loggers = new ConcurrentHashMap<>();
    private LogLevel defaultLevel = LogLevel.INFO;
    
    public enum LogLevel {
        DEBUG, INFO, WARNING, ERROR
    }
    
    public Logger getLogger(String name) {
        return loggers.computeIfAbsent(name, this::createLogger);
    }
    
    private Logger createLogger(String name) {
        Logger logger = new Logger(name, defaultLevel);
        System.out.println("Created new logger: " + name);
        return logger;
    }
    
    public void setDefaultLogLevel(LogLevel level) {
        this.defaultLevel = level;
    }
    
    // 内部Logger类
    public static class Logger {
        private final String name;
        private LogLevel level;
        
        private Logger(String name, LogLevel level) {
            this.name = name;
            this.level = level;
        }
        
        public void setLevel(LogLevel level) {
            this.level = level;
        }
        
        public void debug(String message) {
            if (level.ordinal() <= LogLevel.DEBUG.ordinal()) {
                log(LogLevel.DEBUG, message);
            }
        }
        
        public void info(String message) {
            if (level.ordinal() <= LogLevel.INFO.ordinal()) {
                log(LogLevel.INFO, message);
            }
        }
        
        public void warning(String message) {
            if (level.ordinal() <= LogLevel.WARNING.ordinal()) {
                log(LogLevel.WARNING, message);
            }
        }
        
        public void error(String message) {
            if (level.ordinal() <= LogLevel.ERROR.ordinal()) {
                log(LogLevel.ERROR, message);
            }
        }
        
        private void log(LogLevel level, String message) {
            System.out.println(String.format("[%s] %s: %s", 
                level, name, message));
        }
    }
}

// 使用方式
LoggerManager.Logger logger = LoggerManager.INSTANCE.getLogger("MyApp");
logger.info("Application started");
logger.debug("Debug information");

// 修改日志级别
LoggerManager.INSTANCE.setDefaultLogLevel(LoggerManager.LogLevel.DEBUG);
```

### 2. 应用上下文（静态内部类单例）

```java
public class ApplicationContext {
    private final Map<String, Object> beans = new HashMap<>();
    
    // 私有构造器
    private ApplicationContext() {
        // 初始化应用上下文
        System.out.println("Initializing application context");
    }
    
    // 静态内部类持有单例
    private static class Holder {
        private static final ApplicationContext INSTANCE = new ApplicationContext();
    }
    
    // 全局访问点
    public static ApplicationContext getInstance() {
        return Holder.INSTANCE;
    }
    
    // 注册bean
    public void registerBean(String name, Object bean) {
        beans.put(name, bean);
    }
    
    // 获取bean
    @SuppressWarnings("unchecked")
    public <T> T getBean(String name) {
        Object bean = beans.get(name);
        if (bean == null) {
            throw new IllegalArgumentException("No bean found with name: " + name);
        }
        return (T) bean;
    }
    
    // 检查bean是否存在
    public boolean containsBean(String name) {
        return beans.containsKey(name);
    }
    
    // 获取所有bean名称
    public Set<String> getBeanNames() {
        return Collections.unmodifiableSet(beans.keySet());
    }
}

// 使用方式
ApplicationContext context = ApplicationContext.getInstance();
context.registerBean("userService", new UserService());
UserService service = context.getBean("userService");
```

### 3. 缓存管理器（双重检查锁定懒加载单例）

```java
public class CacheManager {
    private static volatile CacheManager instance;
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    private final long defaultExpirationTimeMs = 3600000; // 1小时
    
    private CacheManager() {
        // 防止反射攻击
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
    }
    
    public static CacheManager getInstance() {
        if (instance == null) {
            synchronized (CacheManager.class) {
                if (instance == null) {
                    instance = new CacheManager();
                }
            }
        }
        return instance;
    }
    
    // 缓存项，包含过期时间
    private static class CacheItem {
        private final Object value;
        private final long expirationTime;
        
        public CacheItem(Object value, long expirationTimeMs) {
            this.value = value;
            this.expirationTime = System.currentTimeMillis() + expirationTimeMs;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() > expirationTime;
        }
        
        public Object getValue() {
            return value;
        }
    }
    
    // 添加缓存项
    public void put(String key, Object value) {
        put(key, value, defaultExpirationTimeMs);
    }
    
    // 添加缓存项，指定过期时间
    public void put(String key, Object value, long expirationTimeMs) {
        cache.put(key, new CacheItem(value, expirationTimeMs));
    }
    
    // 获取缓存项
    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        CacheItem item = (CacheItem) cache.get(key);
        if (item == null) {
            return null;
        }
        
        if (item.isExpired()) {
            cache.remove(key);
            return null;
        }
        
        return (T) item.getValue();
    }
    
    // 移除缓存项
    public void remove(String key) {
        cache.remove(key);
    }
    
    // 清空缓存
    public void clear() {
        cache.clear();
    }
    
    // 清理过期缓存项
    public void cleanExpiredItems() {
        for (Iterator<Map.Entry<String, Object>> it = cache.entrySet().iterator(); it.hasNext();) {
            Map.Entry<String, Object> entry = it.next();
            CacheItem item = (CacheItem) entry.getValue();
            if (item.isExpired()) {
                it.remove();
            }
        }
    }
}

// 使用方式
CacheManager cache = CacheManager.getInstance();
cache.put("user:123", userObject);
User user = cache.get("user:123");
```

## 注意事项

1. **防止反射攻击**
   - 在私有构造器中检查实例是否已存在
   - 使用枚举类型实现单例可以完全防止反射攻击

2. **序列化安全**
   - 实现`readResolve()`方法确保反序列化时返回单例实例
   - 使用枚举类型实现单例可以自动保证序列化安全

3. **线程安全**
   - 静态初始化的单例是线程安全的
   - 懒加载单例需要使用双重检查锁定或静态内部类方式确保线程安全
   - 枚举类型实现的单例天然线程安全

4. **内存泄漏**
   - 单例对象持有的资源需要适当管理，避免内存泄漏
   - 考虑提供显式的清理方法

5. **测试难度**
   - 单例模式使得测试变得困难，因为无法轻易替换为mock对象
   - 考虑使用依赖注入框架或设计模式来提高可测试性

## 练习题

### 练习1：实现线程安全的单例模式
实现一个表示应用程序配置的单例类，要求：
1. 线程安全
2. 懒加载
3. 防止反射攻击
4. 序列化安全

**解答：**

```java
public class AppConfig implements Serializable {
    private static final long serialVersionUID = 1L;
    private static volatile AppConfig instance;
    
    private final Map<String, String> configMap = new HashMap<>();
    
    private AppConfig() {
        // 防止反射攻击
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
        
        // 加载配置
        configMap.put("app.name", "MySingletonApp");
        configMap.put("app.version", "1.0.0");
    }
    
    public static AppConfig getInstance() {
        if (instance == null) {
            synchronized (AppConfig.class) {
                if (instance == null) {
                    instance = new AppConfig();
                }
            }
        }
        return instance;
    }
    
    // 防止反序列化创建新实例
    private Object readResolve() {
        return instance;
    }
    
    public String getConfig(String key) {
        return configMap.get(key);
    }
    
    public void setConfig(String key, String value) {
        configMap.put(key, value);
    }
}
```

### 练习2：使用枚举实现功能完整的单例

实现一个使用枚举的单例模式，管理应用程序的主题设置。

**解答：**

```java
public enum ThemeManager {
    INSTANCE;
    
    public enum Theme {
        LIGHT("light", "#FFFFFF", "#000000"),
        DARK("dark", "#000000", "#FFFFFF"),
        BLUE("blue", "#0000FF", "#FFFFFF");
        
        private final String name;
        private final String backgroundColor;
        private final String textColor;
        
        Theme(String name, String backgroundColor, String textColor) {
            this.name = name;
            this.backgroundColor = backgroundColor;
            this.textColor = textColor;
        }
        
        public String getName() {
            return name;
        }
        
        public String getBackgroundColor() {
            return backgroundColor;
        }
        
        public String getTextColor() {
            return textColor;
        }
    }
    
    private Theme currentTheme = Theme.LIGHT;
    private final List<ThemeChangeListener> listeners = new ArrayList<>();
    
    public interface ThemeChangeListener {
        void onThemeChanged(Theme newTheme);
    }
    
    public void setTheme(Theme theme) {
        if (theme != currentTheme) {
            Theme oldTheme = currentTheme;
            currentTheme = theme;
            notifyListeners();
            System.out.println("Theme changed from " + oldTheme.getName() + " to " + theme.getName());
        }
    }
    
    public Theme getCurrentTheme() {
        return currentTheme;
    }
    
    public void addThemeChangeListener(ThemeChangeListener listener) {
        listeners.add(listener);
    }
    
    public void removeThemeChangeListener(ThemeChangeListener listener) {
        listeners.remove(listener);
    }
    
    private void notifyListeners() {
        for (ThemeChangeListener listener : listeners) {
            listener.onThemeChanged(currentTheme);
        }
    }
}

// 使用方式
ThemeManager.INSTANCE.setTheme(ThemeManager.Theme.DARK);
ThemeManager.Theme currentTheme = ThemeManager.INSTANCE.getCurrentTheme();

// 添加主题变化监听器
ThemeManager.INSTANCE.addThemeChangeListener(newTheme -> {
    System.out.println("Theme changed to: " + newTheme.getName());
    System.out.println("New background color: " + newTheme.getBackgroundColor());
});
```

## 补充知识点

### 1. 单例与依赖注入框架

在现代Java应用程序中，单例模式通常通过依赖注入(DI)框架来管理，如Spring：

```java
@Component
@Scope("singleton") // 默认就是单例，可以省略
public class UserService {
    // 服务实现
}
```

这种方式的优点：
- 由框架管理生命周期
- 更容易进行单元测试（可以注入mock对象）
- 更好的关注点分离

### 2. 单例与多实例环境

在分布式系统中，单例模式需要特别注意：
- 每个JVM实例都会有自己的"单例"
- 可能需要使用分布式锁或共享存储来实现真正的全局单例

### 3. 单例与内存模型

Java内存模型(JMM)对单例实现有重要影响：
- `volatile`关键字确保可见性和有序性
- 双重检查锁定模式在Java 5之前可能存在问题
- 静态内部类方式利用了类加载机制的线程安全特性

### 4. 单例与垃圾回收

单例对象通常在应用程序整个生命周期内存在，但在某些情况下可能被垃圾回收：
- 类加载器被卸载时
- 使用弱引用实现的"软单例"

### 5. 单例与函数式编程

在函数式编程范式中，单例模式通常被视为反模式，因为：
- 它引入了全局状态
- 降低了代码的可测试性
- 违反了引用透明性原则

在这种情况下，可以考虑使用依赖注入或上下文传递来替代单例。
