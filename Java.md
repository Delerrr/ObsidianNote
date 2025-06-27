## 模块化系统
带来的访问控制变化：
![[Pasted image 20250621231404.png]]
语法：
![[Pasted image 20250621231559.png]]
语法解读：
- `[open] module <module>`: 声明一个模块，模块名称应全局唯一，不可重复。加上 `open` 关键词表示模块内的所有包都允许通过 Java 反射访问，模块声明体内不再允许使用 `opens` 语句。
- `requires [transitive] <module>`: 声明模块依赖，一次只能声明一个依赖，如果依赖多个模块，需要多次声明。加上 `transitive` 关键词表示传递依赖，比如模块 A 依赖模块 B，模块 B 传递依赖模块 C，那么模块 A 就会自动依赖模块 C，类似于 Maven。
- `exports <package> [to <module1>[, <module2>...]]`: 导出模块内的包（允许直接 `import` 使用），一次导出一个包，如果需要导出多个包，需要多次声明。如果需要定向导出，可以使用 `to` 关键词，后面加上模块列表（逗号分隔）。
- `opens <package> [to <module>[, <module2>...]]`: 开放模块内的包（允许通过 Java 反射访问），一次开放一个包，如果需要开放多个包，需要多次声明。如果需要定向开放，可以使用 `to` 关键词，后面加上模块列表（逗号分隔）。
- `provides <interface | abstract class> with <class1>[, <class2> ...]`: 声明模块提供的 Java SPI 服务，一次可以声明多个服务实现类（逗号分隔）。
- `uses <interface | abstract class>`: 声明模块依赖的 Java SPI 服务，加上之后模块内的代码就可以通过 `ServiceLoader.load(Class)` 一次性加载所声明的 SPI 服务的所有实现类。

##  接口中的private、static、default

在 Java 8 及以上版本中，接口引入了新的方法类型：`default` 方法、`static` 方法和 `private` 方法。这些方法让接口的使用更加灵活。

Java 8 引入的`default` 方法用于提供接口方法的默认实现，可以在实现类中被覆盖。这样就可以在不修改实现类的情况下向现有接口添加新功能，从而增强接口的扩展性和向后兼容性。

```java
public interface MyInterface {
    default void defaultMethod() {
        System.out.println("This is a default method.");
    }
}
```

Java 8 引入的`static` 方法无法在实现类中被覆盖，只能通过接口名直接调用（ `MyInterface.staticMethod()`），类似于类中的静态方法。`static` 方法通常用于定义一些通用的、与接口相关的工具方法，一般很少用。

```java
public interface MyInterface {
    static void staticMethod() {
        System.out.println("This is a static method in the interface.");
    }
}
```

Java 9 允许在接口中使用 `private` 方法。`private`方法可以用于在接口内部共享代码，不对外暴露。

```java
public interface MyInterface {
    // default 方法
    default void defaultMethod() {
        commonMethod();
    }

    // static 方法
    static void staticMethod() {
        commonMethod();
    }

    // 私有静态方法，可以被 static 和 default 方法调用
    private static void commonMethod() {
        System.out.println("This is a private method used internally.");
    }

      // 实例私有方法，只能被 default 方法调用。
    private void instanceCommonMethod() {
        System.out.println("This is a private instance method used internally.");
    }
}
```

## 一些`Unchecked Exception`

`RuntimeException` 及其子类都统称为非受检查异常，常见的有（建议记下来，日常开发中会经常用到）：

- `NullPointerException`(空指针错误)
- `IllegalArgumentException`(参数错误比如方法入参类型错误)
- `NumberFormatException`（字符串转换为数字格式错误，`IllegalArgumentException`的子类）
- `ArrayIndexOutOfBoundsException`（数组越界错误）
- `ClassCastException`（类型转换错误）
- `ArithmeticException`（算术错误）
- `SecurityException` （安全错误比如权限不够）
- `UnsupportedOperationException`(不支持的操作错误比如重复创建同一用户)
- ……

Collection类图

![[Pasted image 20250627204128.png]]

## StringBuffer和StringBuilder, HashMap和Hashtable
## StringBuffer和StringBuilder

![[Pasted image 20250627211742.png]]
StringBuffer线程安全（所有公共方法（如 append、insert 等）都使用 synchronized 关键字进行同步），StringBuilder线程不安全但速度快一些
## HashMap和Hashtable

1. ### 两者**父类**不同
    
    - HashMap是继承自AbstractMap类
        
    - Hashtable是继承自Dictionary类
        
    - 都实现了同时实现了map、Cloneable（可复制）、Serializable（可序列化）这三个接口

2. ### **安全性**不同
    
    - HashMap 在多线程并发的环境下，线程是不安全的,效率远远高于Hashtable
        
    - Hashtable在多线程并发的环境下，线程是安全的，它的每个方法上都有synchronized 关键字
    
3. ### 对**null的支持**不同
    
    - HashMap允许键和值都为null，
        
    - Hashtable不允许键或值为null。
        
    - 如果尝试将null作为键或值存入Hashtable，将会抛出**NullPointerException (空指针异常)**。

4. ### **初始容量大小和每次扩充容量大小**不同
    
    - Hashtable 的初始长度是 11，之后每次扩充容量变为之前的2n+1（n 为上一次的长度）
        
    - HashMap 的初始长度为 16，之后每次扩充变为原来的两倍（位与）

5. ### **对外提供的接口**不同

- Hashtable比HashMap多提供了elments() 和contains() 两个方法。
	
- elements() 返回此Hashtable中的value的枚举
	
- contains(V) 判断该Hashtable是否包含传入的value。作用与hashtable的containsValue()一致

## Hashtable不允许有 null 键和null值,否则会抛出NullPointerException

原因：

1.  null键：null作为键，无法使用hashcode和equals方法，所以不好

2.  null值：会出现二义性：
	1. 情形一：首先通过contains(key)返回true，在get之前，另一个线程删除了value，于是get到了null，这时就无法判断是原本就是null还是因为不存在而返回的null
	2. 情形二：线程A看到get(key) == null并认为键不存在，尝试插入新值，但线程B可能同时将key的值设置为null，导致逻辑混乱

## Function使用技巧

在Java中，使用`@FunctionalInterface`注解来标识一个接口为函数式接口。函数式接口是指只包含一个抽象方法的接口。

1. andThen和compose
    
      `Function` 接口的 `andThen()` 方法可以将多个函数连接在一起形成一个链式调用。这种方式可以使代码更加清晰和简洁。
    
    ```Java
    import java.util.function.Function;
    
    public class Example {
        public static void main(String[] args) {
            Function<Integer, Integer> add = x -> x + 2;
            Function<Integer, Integer> multiply = x -> x * 3;
    
            // 先加2再乘3
            Function<Integer, Integer> addAndMultiply = add.andThen(multiply);
            System.out.println(addAndMultiply.apply(5)); // 输出为21
        }
    }
    ```
    
      `Function` 接口的 `compose()` 方法可以将多个函数组合起来形成一个新的函数。这种方式可以在不改变原有函数的情况下创建新的函数。
    
    ```Java
    import java.util.function.Function;
    
    public class Example {
        public static void main(String[] args) {
            Function<Integer, Integer> add = x -> x + 2;
            Function<Integer, Integer> multiply = x -> x * 3;
    
            // 先乘3再加2
            Function<Integer, Integer> multiplyAndAdd = add.compose(multiply);
            System.out.println(multiplyAndAdd.apply(5)); // 输出为17
        }
    }
    ```
    
**compose** 和 **andThen** 分别对应数学中的函数组合（f ∘ g）和顺序执行的两种不同思维方式：

- compose：表示“先执行 g，再将结果传给 f”，即 f(g(x))。这符合数学中函数组合的习惯（从右到左）。
- andThen：表示“先执行 f，再将结果传给 g”，即 g(f(x))。这更符合程序员直觉的“先做这个，再做那个”的顺序。

## BigDecimal 等值比较问题
![[Pasted image 20250627222335.png]]