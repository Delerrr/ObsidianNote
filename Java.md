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

## Collection类图

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

**interrupt、 interrupted、isInterrupted 区别**

- interrupt：用于中断线程，调用该方法的线程的状态为将被置为”中断”状态，设置中断标识为true
    
- Thread.interrupted()：查看线程的中断状态，并且清除中断信号，中断状态会被清零 静态方法
    
- isInterrupted：查询线程的中断状态，且不会改变中断状态标识

## Switch新特性

使用switch时，如果遗漏了break，就会造成严重的逻辑错误，而且不易在源代码中发现错误。从Java 12开始，switch语句升级为更简洁的表达式语法，使用类似模式匹配（Pattern Matching）的方法，保证只有一种路径会被执行，并且不需要[break语句](https://so.csdn.net/so/search?q=break%E8%AF%AD%E5%8F%A5&spm=1001.2101.3001.7020)：

```java
public class Main {
    public static void main(String[] args) {
        String fruit = "apple";
        switch (fruit) {
        case "apple" -> System.out.println("Selected apple");
        case "pear" -> System.out.println("Selected pear");
        case "mango" -> {
            System.out.println("Selected mango");
            System.out.println("Good choice!");
        }
        default -> System.out.println("No fruit selected");
        }
    }
}

```

注意新语法使用->，如果有多条语句，需要用{}括起来。不要写break语句，因为新语法只会执行匹配的语句，没有穿透效应。

很多时候，我们还可能用switch语句给某个变量赋值。例如：

```java
int opt;
switch (fruit) {
case "apple":
    opt = 1;
    break;
case "pear":
case "mango":
    opt = 2;
    break;
default:
    opt = 0;
    break;
}
```

使用新的switch语法，不但不需要break，还可以直接返回值。把上面的代码改写如下：

```java
public class Main {
    public static void main(String[] args) {
        String fruit = "apple";
        int opt = switch (fruit) {
            case "apple" -> 1;
            case "pear", "mango" -> 2;
            default -> 0;
        }; // 注意赋值语句要以;结束
        System.out.println("opt = " + opt);
    }
}

```

这样可以获得更简洁的代码。

**yield**

大多数时候，在switch表达式内部，我们会返回简单的值。

但是，如果需要复杂的语句，我们也可以写很多语句，放到{…}里，然后，用yield返回一个值作为switch语句的返回值：

```java
public class Main {
    public static void main(String[] args) {
        String fruit = "orange";
        int opt = switch (fruit) {
            case "apple" -> 1;
            case "pear", "mango" -> 2;
            default -> {
                int code = fruit.hashCode();
                yield code; // switch语句返回值
            }
        };
        System.out.println("opt = " + opt);
    }
}

```

## volatile 禁止指令重排序的例子

```java
public class ReorderExample {  
	private int x = 0;  
	private volatile boolean ready = false;  
	
	public void writer() {  x = 42; // 语句 1  
		ready = true; // 语句 2  
	} 
	 
	public void reader() {  
		if (ready) { // 语句 3  
			System.out.println(x); // 语句 4  
		}  
	}  
}
```
- **没有 volatile 的情况**：
    - 编译器或处理器可能将 writer() 中的语句 1 和语句 2 重排序，即先执行 ready = true，再执行 x = 42。
    - 如果线程 A 执行 writer()，线程 B 执行 reader()，可能出现以下情况：
        - 线程 B 看到 ready = true（语句 3），但 x 仍是 0（因为语句 1 还未执行）。
        - 结果：线程 B 打印 x = 0，而不是预期的 42。
- **使用 volatile 的情况**：
    - volatile boolean ready 确保 ready = true（语句 2）不会被重排序到 x = 42（语句 1）之前。
    - 同时，volatile 保证写操作的 happens-before 关系，线程 B 在看到 ready = true 时，必定能看到 x = 42。
    - 结果：线程 B 要么不执行（因为 ready = false），要么打印 x = 42

## StampedLock：支持乐观读锁


```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            stamp = stampedLock.readLock(); // 获取一个悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

```

和`ReadWriteLock`相比，写入的加锁是完全一样的，不同的是读取。注意到首先我们通过`tryOptimisticRead()`获取一个乐观读锁，并返回版本号。接着进行读取，读取完成后，我们通过`validate()`去验证版本号，如果在读取过程中没有写入，版本号不变，验证成功，我们就可以放心地继续后续操作。如果在读取过程中有写入，版本号会发生变化，验证将失败。在失败的时候，我们再通过获取悲观读锁再次读取。由于写入的概率不高，程序在绝大部分情况下可以通过乐观读锁获取数据，极少数情况下使用悲观读锁获取数据。

可见，`StampedLock`把读锁细分为乐观读和悲观读，能进一步提升并发效率。但这也是有代价的：一是代码更加复杂，二是`StampedLock`是不可重入锁，不能在一个线程中反复获取同一个锁。

`StampedLock`还提供了更复杂的将悲观读锁升级为写锁的功能，它主要使用在if-then-update的场景：即先读，如果读的数据满足条件，就返回，如果读的数据不满足条件，再尝试写。
