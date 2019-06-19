## 1、“.equals()” should not be used to test the values of “Atomic” classes
>`AtomicInteger`, and `AtomicLong` extend `Number`, but they're distinct from `Integer` and `Long` and should be handled differently. `AtomicInteger` and `AtomicLong`are designed to support lock-free, thread-safe programming on single variables. As such, an `AtomicInteger` will only ever be "equal" to itself. Instead, you should`.get()` the value and make comparisons on it.

### Noncompliant Code Example
```java
AtomicInteger aInt1 = new AtomicInteger(0);
AtomicInteger aInt2 = new AtomicInteger(0);
if (aInt1.equals(aInt2)) { ... } // Noncompliant
```
### Compliant Solution
```java
AtomicInteger aInt1 = new AtomicInteger(0);
AtomicInteger aInt2 = new AtomicInteger(0);
if (aInt1.get() == aInt2.get()) { ... }
```
### 一句话说明
- `AtomicInteger`、`AtomicLong`、`AtomicDouble`等类不能使用equals来判断是否相等

### 解释
`Atomic`类相等与否的判断不能使用`equals()`方法,`Atomic`的作用为了在多线程情况下保持单个变量的线程安全性。`Atomic`变量永远只会和自身相等，如果要判断`Atomic`变量是否相等，应该使用`get()`方法获取值之后再进行判断。
- AtomicInteger VS Integer
`Integer`类重写了`Object`的`equals()`方法，根据值的大小判断是否相等：
```java
/**
 * Compares this object to the specified object.  The result is
 * {@code true} if and only if the argument is not
 * {@code null} and is an {@code Integer} object that
 * contains the same {@code int} value as this object.
 *
 * @param   obj   the object to compare with.
 * @return  {@code true} if the objects are the same;
 *          {@code false} otherwise.
 */
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```
而`AtomicInteger`没有重写`equals()`方法，当使用`equals()`时，实际上使用的是`Object`的`equals()`方法，最终判断的是对象的地址是否相等。
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
`Object`的`equals()`描述：只有当x和y指向相同的引用时,`equals()`才返回`true`。
>for any non-null reference values {@code x} and {@code y}, this method returns {@code true} if and only if {@code x} and {@code y} refer to the same object

### 例子
```java
@Test
public void test1() {
    AtomicInteger aInt1 = new AtomicInteger(0);
    AtomicInteger aInt2 = new AtomicInteger(0);
    //正例
    if (aInt1.get() == aInt2.get()) {
    }
    System.out.println(aInt1.get() == aInt2.get()); //true
    //反例
    if (aInt1.equals(aInt2)) {
    }
    System.out.println(aInt1.equals(aInt2)); //false
}
```









## 2、"=+" should not be used instead of "+="
The use of operators pairs ( `=+`, `=-` or `=!` ) where the reversed, single operator was meant (`+=`, `-=` or `!=`) will compile and run, but not produce the expected results.
This rule raises an issue when `=+`, `=-`, or `=!` is used without any spacing between the two operators and when there is at least one whitespace character after.
### Noncompliant Code Example
```java
int target = -5;
int num = 3;
target =- num;  // Noncompliant; target = -3\. Is that really what's meant?
target =+ num; // Noncompliant; target = 3</pre>
```
### Compliant Solution
```java
int target = -5;
int num = 3;
target =- num;  // Noncompliant; target = -3\. Is that really what's meant?
target =+ num; // Noncompliant; target = 3
```
### 一句话说明
- 必须正确写成+=，绝对不能错误地写成=+，意义完全不一样

### 例子
```java
@Test
public void test2() {
    int target = -5;
    int num = 3;
    //正例
    target = -num;
    target += num;
    System.out.println(target); // 0
    //反例
    int target2 = -5;
    int num2 = 3;
    target2 = -num2;
    target2 = +num2;
    System.out.println(target2); //3
}
```





## 3、"@Controller" classes that use "@SessionAttributes" must call "setComplete" on their "SessionStatus" objects
> A Spring `@Controller` that uses `@SessionAttributes` is designed to handle a stateful / multi-post form. Such `@Controller`s use the specified `@SessionAttributes`to store data on the server between requests. That data should be cleaned up when the session is over, but unless `setComplete()` is called on the `SessionStatus`object from a `@RequestMapping` method, neither Spring nor the JVM will know it's time to do that. Note that the `SessionStatus` object must be passed to that method as a parameter.
This applies to all the atomic, seeming-primitive wrapper classes: `AtomicInteger`, `AtomicLong`, and `AtomicBoolean`.

### Noncompliant Code Example
```java
@Controller
@SessionAttributes("hello")  // Noncompliant; this doesn't get cleaned up
public class HelloWorld {

  @RequestMapping("/greet", method = GET)
  public String greet(String greetee) {

    return "Hello " + greetee;
  }
}
```
### Compliant Solution
```java
@Controller
@SessionAttributes("hello")
public class HelloWorld {

  @RequestMapping("/greet", method = GET)
  public String greet(String greetee) {

    return "Hello " + greetee;
  }

  @RequestMapping("/goodbye", method = POST)
  public String goodbye(SessionStatus status) {
    //...
    status.setComplete();
  }

}
```
### 一句话说明
- 使用SessionAttributes注解的类中，必须调用setComplete方法完成session的清除，否则会永久留在session，从而产生隐患
### 说明
- [https://www.cnblogs.com/daxin/p/SessionAttributes.html](https://www.cnblogs.com/daxin/p/SessionAttributes.html "SpringMVC @SessionAttributes 使用详解以及源码分析")
- sessionattributes注解，可以将属性存储到session中
- 需要手动调用接口清除session的对象，因为不会自动清空

### 例子






## 4、"@NonNull" values should not be set to null
>Fields, parameters and return values marked `@NotNull`, `@NonNull`, or `@Nonnull` are assumed to have non-null values and are not typically null-checked before use. Therefore setting one of these values to `null`, or failing to set such a class field in a constructor, could cause `NullPointerException`s at runtime.

### Noncompliant Code Example
```java
public class MainClass {

  @Nonnull
  private String primary;
  private String secondary;

  public MainClass(String color) {
    if (color != null) {
      secondary = null;
    }
    primary = color;  // Noncompliant; "primary" is Nonnull but could be set to null here
  }

  public MainClass() { // Noncompliant; "primary" Nonnull" but is not initialized
  }

  @Nonnull
  public String indirectMix() {
    String mix = null;
    return mix;  // Noncompliant; return value is Nonnull, but null is returned.}}
  }
```
### 一句话说明
- 标有`@NotNull`注解的字段、参数、返回值，不应该出现空值的情况，否则会有可能抛运行时异常





## 5、"@SpringBootApplication" and "@ComponentScan" should not be used in the default package

>`@ComponentScan` is used to determine which Spring Beans are available in the application context. The packages to scan can be configured thanks to the `basePackageClasses` or `basePackages` (or its alias `value`) parameters. If neither parameter is configured, `@ComponentScan` will consider only the package of the class annotated with it. When `@ComponentScan` is used on a class belonging to the default package, the entire classpath will be scanned.

This will slow-down the start-up of the application and it is likely the application will fail to start with an `BeanDefinitionStoreException` because you ended up scanning the Spring Framework package itself.

This rule raises an issue when:
- `@ComponentScan`, `@SpringBootApplication` and `@ServletComponentScan` are used on a class belonging to the default package
- `@ComponentScan` is explicitly configured with the default package

### Noncompliant Code Example
```java
import org.springframework.boot.SpringApplication;

@SpringBootApplication // Noncompliant; RootBootApp is declared in the default package
public class RootBootApp {
...
}
@ComponentScan("")
public class Application {
...
}
```
### Compliant Solution
```java
package hello;

import org.springframework.boot.SpringApplication;

@SpringBootApplication // Compliant; RootBootApp belongs to the "hello" package
public class RootBootApp {
...
}
```
### 一句话说明
- 标有`@SpringBootApplication`或`@ComponentScan`注解的类不允许放在默认包下，必须放在自定义包名下，否则会扫描全部文件




## 6、BigDecimal(double)" should not be used
Because of floating point imprecision, you're unlikely to get the value you expect from the `BigDecimal(double)` constructor.
From [the JavaDocs](http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html#BigDecimal(double)):
>The results of this constructor can be somewhat unpredictable. One might assume that writing new BigDecimal(0.1) in Java creates a BigDecimal which is exactly equal to 0.1 (an unscaled value of 1, with a scale of 1), but it is actually equal to 0.1000000000000000055511151231257827021181583404541015625\. This is because 0.1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the value that is being passed in to the constructor is not exactly equal to 0.1, appearances notwithstanding.

### Noncompliant Code Example
```java
double d = 1.1;

BigDecimal bd1 = new BigDecimal(d); // Noncompliant; see comment above
BigDecimal bd2 = new BigDecimal(1.1); // Noncompliant; same result
```
### Compliant Solution
```java
double d = 1.1;

BigDecimal bd1 = BigDecimal.valueOf(d);
BigDecimal bd2 = new BigDecimal("1.1"); // using String constructor will result in precise value
```
### See

*   [CERT, NUM10-J.](https://www.securecoding.cert.org/confluence/x/NQAVAg) - Do not construct BigDecimal objects from floating-point literals

### 一句话说明
- 因为浮点的不精确,可能使用BigDecimal(double)得不到期望的值
- 不要使用`BigDecimal(double)`去构造一个`BigDecimal`对象，因为`double`类型在计算机表示方法中并不精确，因此，`BigDecimal(double)`构造出来的对象很可能不是预期的大小，若一定要使用`double`类型去构造一个`BigDecimal`对象，请使用`BigDecimal.valueOf`方法，该方法先将`double`转换为`String`，再通过`String`构造`BigDecimal`对象，通常更建议使用`public BigDecimal(String val)`构造方法。






## 7、"compareTo" results should not be checked for specific values

### 级别
- 次要

>While most `compareTo` methods return -1, 0, or 1, some do not, and testing the result of a `compareTo` against a specific value other than 0 could result in false negatives.

### Noncompliant Code Example
```java
if (myClass.compareTo(arg) == -1) {  // Noncompliant
  // ...
}```

### Compliant Solution
```java
if (myClass.compareTo(arg) < 0) {
  // ...
}```

### 一句话说明
- 调用compareTo方法的返回值建议别和指定值进行比较，最好和`==0`、`>0`、`<0`进行比较，compareTo绝大多数情况是返回-1、0、1，但也存在返回不是这三个数





## 8、"compareTo" should not be overloaded

#### 级别
- 主要

>When implementing the `Comparable<T>.compareTo` method, the parameter's type has to match the type used in the `Comparable` declaration. When a different type is used this creates an overload instead of an override, which is unlikely to be the intent.
>
This rule raises an issue when the parameter of the `compareTo` method of a class implementing `Comparable<T>` is not same as the one used in the `Comparable`declaration.

### Noncompliant Code Example
```java
public class Foo {
  static class Bar implements Comparable<Bar> {
    public int compareTo(Bar rhs) {
      return -1;
    }
  }

  static class FooBar extends Bar {
    public int compareTo(FooBar rhs) {  // Noncompliant: Parameter should be of type Bar
      return 0;
    }
  }
}
```

### Compliant Solution
```java
public class Foo {
  static class Bar implements Comparable<Bar> {
    public int compareTo(Bar rhs) {
      return -1;
    }
  }

  static class FooBar extends Bar {
    public int compareTo(Bar rhs) {
      return 0;
    }
  }
}
```

### 一句话说明
- compareTo方法不允许被重载






## 9、"compareTo" should not return "Integer.MIN_VALUE"

- 主要

>It is the sign, rather than the magnitude of the value returned from `compareTo` that matters. Returning `Integer.MIN_VALUE` does _not_ convey a higher degree of inequality, and doing so can cause errors because the return value of `compareTo` is sometimes inversed, with the expectation that negative values become positive. However, inversing `Integer.MIN_VALUE` yields `Integer.MIN_VALUE` rather than `Integer.MAX_VALUE`.

### Noncompliant Code Example
```java
public int compareTo(MyClass) {
  if (condition) {
    return Integer.MIN_VALUE;  // Noncompliant
  }
```

### Compliant Solution
```java
public int compareTo(MyClass) {
  if (condition) {
    return -1;
  }
```

### 一句话说明
- compareTo只代表一个不等标识，不代表不等的程度，应返回-1,0,1标识即可
，`Integer.MIN_VALUE`有可能会反转，负值变成正值。




## 10、"DefaultMessageListenerContainer" instances should not drop messages during restarts

- 主要

>`DefaultMessageListenerContainer` is implemented as a JMS poller. While the Spring container is shutting itself down, as each in-progress JMS `Consumer.receive()`call completes, any non-`null` return value will be a JMS message that the DMLC will _discard_ due to the shutdown in progress. That will result in the received message never being processed.

>To prevent message loss during restart operations, set `acceptMessagesWhileStopping` to `true` so that such messages will be processed before shut down.

### Noncompliant Code Example
```java
<bean id="listenerContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">  <!-- Noncompliant -->
   <property name="connectionFactory" ref="connFactory" />
   <property name="destination" ref="dest" />
   <property name="messageListener" ref="serviceAdapter" />
   <property name="autoStartup" value="true" />
   <property name="concurrentConsumers" value="10" />
   <property name="maxConcurrentConsumers" value="10" />
   <property name="clientId" value="myClientID" />
</bean>
```

### Compliant Solution
```java
<bean id="listenerContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
   <property name="connectionFactory" ref="connFactory" />
   <property name="destination" ref="dest" />
   <property name="messageListener" ref="serviceAdapter" />
   <property name="autoStartup" value="true" />
   <property name="concurrentConsumers" value="10" />
   <property name="maxConcurrentConsumers" value="10" />
   <property name="clientId" value="myClientID" />
   <property name="acceptMessagesWhileStopping" value="true" />
</bean>
```

### 一句话说明
- 在重启之前需要处理完所有已接受的消息，`acceptMessagesWhileStopping`设置为true，就不会因重启导致已接受的消息未处理完




## 11、"Double.longBitsToDouble" should not be used for "int"

- 主要

>`Double.longBitsToDouble` expects a 64-bit, `long` argument. Pass it a smaller value, such as an `int` and the mathematical conversion into a `double` simply won't work as anticipated because the layout of the bits will be interpreted incorrectly, as if a child were trying to use an adult's gloves.

### Noncompliant Code Example
```java
int i = 42;
double d = Double.longBitsToDouble(i);  // Noncompliant
```

### Compliant Solution
```java

```

### 一句话说明
- Double.longBitsToDouble需要64位的长参数。传递一个更小的值，例如int和将数学转换为double，这将不会像预期的那样工作，因为位的布局将被错误地解释，就像孩子试图使用大人的手套一样。







## 12、"equals" method overrides should accept "Object" parameters

- 主要

>"equals" as a method name should be used exclusively to override `Object.equals(Object)` to prevent any confusion.

>It is tempting to overload the method to take a specific class instead of `Object` as parameter, to save the class comparison check. However, this will not work as expected when that is the only override.

### Noncompliant Code Example
```java
class MyClass {
  private int foo = 1;

  public boolean equals(MyClass o) {  // Noncompliant; does not override Object.equals(Object)
    return o != null && o.foo == this.foo;
  }

  public static void main(String[] args) {
    MyClass o1 = new MyClass();
    Object o2 = new MyClass();
    System.out.println(o1.equals(o2));  // Prints "false" because o2 an Object not a MyClass
  }
}

class MyClass2 {
  public boolean equals(MyClass2 o) {  // Ignored; `boolean equals(Object)` also present
    //..
  }

  public boolean equals(Object o) {
    //...
  }
}
```

### Compliant Solution
```java
class MyClass {
  private int foo = 1;

  @Override
  public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
      return false;
    }

    MyClass other = (MyClass)o;
    return this.foo == other.foo;
  }

  /* ... */
}

class MyClass2 {
  public boolean equals(MyClass2 o) {
    //..
  }

  public boolean equals(Object o) {
    //...
  }
}
```

### 一句话说明
- equals作为方法名应该仅用于重写Object.equals(Object)来避免混乱.

### 例子
```java
@Test
    public void test12() {
        MyClass o1 = new MyClass();
        Object o2 = new MyClass();
        System.out.println(o1.equals(o2));  //打印false

        Object o3 = new MyClass();
        Object o4 = new MyClass();
        System.out.println(o3.equals(o4));  // 打印false，Prints "false" because o2 an Object not a MyClass

        MyClass o5 = new MyClass();
        MyClass o6 = new MyClass();
        System.out.println(o5.equals(o6));  //打印true

        Object o7 = new MyClass();
        MyClass o8 = new MyClass();
        System.out.println(o7.equals(o8));  //打印false

        MyClass2 o9 = new MyClass2();
        Object o10 = new MyClass2();
        System.out.println(o9.equals(o10));  //打印true
    }

    class MyClass {
        private int foo = 1;

        public boolean equals(MyClass o) {  // Noncompliant; does not override Object.equals(Object)
            return o != null && o.foo == this.foo; //不会执行，没有重载Object类的equals方法
        }
    }

    class MyClass2 {
        private int foo = 1;

        //会执行，即使没有@Override注解，方法签名跟Object一致即可
        public boolean equals(Object obj) {
            MyClass2 otherClass = (MyClass2) obj;
            return otherClass != null && otherClass.foo == this.foo;
        }
    }
```





## 13、"equals" methods should be symmetric and work for subclasses


- 次要

>A key facet of the `equals` contract is that if `a.equals(b)` then `b.equals(a)`, i.e. that the relationship is symmetric.

>Using `instanceof` breaks the contract when there are subclasses, because while the child is an `instanceof` the parent, the parent is not an `instanceof` the child. For instance, assume that `Raspberry extends Fruit` and adds some fields (requiring a new implementation of `equals`):

>等号契约的一个关键方面是，如果A = (b)那么b = (A)，即关系是对称的。
当存在子类时，使用instanceof会破坏契约，因为虽然子类是父类的instanceof，但父类不是子类的instanceof。例如，假设Raspberry扩展了Fruit并添加了一些字段(需要一个新的equals实现):

```java
Fruit fruit = new Fruit();
Raspberry raspberry = new Raspberry();

if (raspberry instanceof Fruit) { ... } // true
if (fruit instanceof Raspberry) { ... } // false
```
>If similar `instanceof` checks were used in the classes' `equals` methods, the symmetry principle would be broken:
>如果在类的equals方法中使用类似的`instanceof`检查，对称性原则就会被打破:

```java
raspberry.equals(fruit); // false
fruit.equals(raspberry); //true
```
>Additionally, non `final` classes shouldn't use a hardcoded class name in the `equals` method because doing so breaks the method for subclasses. Instead, make the comparison dynamic.

>此外，非`final`类不应该在`equals`方法中使用硬编码的类名，因为这样做会破坏子类的方法。相反，要让比较是动态的。

------------

>Further, comparing to an unrelated class type breaks the contract for that unrelated type, because while `thisClass.equals(unrelatedClass)` can return true, `unrelatedClass.equals(thisClass)` will not.

>此外，与不相关的类类型进行比较会破坏该不相关类型的契约，因为虽然`thisClass.equals(unrelatedClass)`可以返回`true`，但`unrelatedClass.equals(thisClass)`不会返回`true`。

### Noncompliant Code Example
```java
public class Fruit extends Food {
  private Season ripe;

  public boolean equals(Object obj) {
    if (obj == this) {
      return true;
    }
    if (obj == null) {
      return false;
    }
    if (Fruit.class == obj.getClass()) { // Noncompliant; broken for child classes
      return ripe.equals(((Fruit)obj).getRipe());
    }
    if (obj instanceof Fruit ) {  // Noncompliant; broken for child classes
      return ripe.equals(((Fruit)obj).getRipe());
    }
    else if (obj instanceof Season) { // Noncompliant; symmetry broken for Season class
      // ...
    }
    //...
```

### Compliant Solution
```java
public class Fruit extends Food {
  private Season ripe;

  public boolean equals(Object obj) {
    if (obj == this) {
      return true;
    }
    if (obj == null) {
      return false;
    }
    if (this.getClass() == obj.getClass()) {
      return ripe.equals(((Fruit)obj).getRipe());
    }
    return false;
}
```
### See

*   [CERT, MET08-J.](https://www.securecoding.cert.org/confluence/x/zIUbAQ) - Preserve the equality contract when overriding the equals() method

### 一句话说明
-  equals应是对等并且在有子类参与时能正常工作

### 例子
```java
@Test
    public void test13() {
        Fruit fruit = new Fruit();
        Raspberry raspberry = new Raspberry();//覆盆莓水果

        System.out.println(raspberry instanceof Fruit);// true
        System.out.println(fruit instanceof Raspberry);// false
        System.out.println(raspberry.equals(fruit));// false
        System.out.println(fruit.equals(raspberry));// false
    }

    class Fruit {
        private int water;
    }

    class Raspberry extends Fruit {
        private String otherAttr;
    }
```






## 14、"equals(Object obj)" and "hashCode()" should be overridden in pairs


- 次要

According to the Java Language Specification, there is a contract between `equals(Object)` and `hashCode()`:

> If two objects are equal according to the `equals(Object)` method, then calling the `hashCode` method on each of the two objects must produce the same integer result.

> It is not required that if two objects are unequal according to the `equals(java.lang.Object)` method, then calling the `hashCode` method on each of the two objects must produce distinct integer results.

> However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hashtables.

In order to comply with this contract, those methods should be either both inherited, or both overridden.

### Noncompliant Code Example
```java
class MyClass {    // Noncompliant - should also override "hashCode()"

  @Override
  public boolean equals(Object obj) {
    /* ... */
  }

}
```

### Compliant Solution
```java
class MyClass {    // Compliant

  @Override
  public boolean equals(Object obj) {
    /* ... */
  }

  @Override
  public int hashCode() {
    /* ... */
  }

}
```
### See

*   [MITRE, CWE-581](http://cwe.mitre.org/data/definitions/581.html) - Object Model Violation: Just One of Equals and Hashcode Defined
*   [CERT, MET09-J.](https://www.securecoding.cert.org/confluence/x/EYYbAQ) - Classes that define an equals() method must also define a hashCode() method

### 一句话说明
- equals和hashCode要么一起重载，要么都不重载

### 说明
#### 为什么要重载equal方法？
- Object的equal方法默认是两个对象引用的比较。意思是指向同一内存地址才相等，否则不相等；如果你现在需要利用对象里面的值来判断是否相等，则要重载equal方法。
- 在我们的业务系统中判断对象时有时候需要的不是一种严格意义上的相等，而是一种业务上的对象相等。在这种情况下，原生的equals方法就不能满足我们的需求了
#### 为什么重载hashCode方法？
- 一般的地方不需要重载hashCode，只有当类需要放在HashTable、HashMap、HashSet等等hash结构的集合时才会重载hashCode，那么为什么要重载hashCode呢？就HashMap来说，好比HashMap就是一个大内存块，里面有很多小内存块，小内存块里面是一系列的对象，可以利用hashCode来查找小内存块hashCode%size(小内存块数量)，所以当equal相等时，hashCode必须相等，而且如果是object对象，必须重载hashCode和equal方法。
#### 为什么equals()相等，hashCode就一定要相等，而hashCode相等，却不要求equals相等?
- 因为按照hashCode来访问小内存块，所以hashCode必须相等。
- HashMap获取一个对象是比较key的hashCode相等和equal为true。
之所以hashCode相等，却可以equal不等，就比如ObjectA和ObjectB他们都有属性name，那么hashCode都以name计算，所以hashCode一样，但是两个对象属于不同类型，所以equal为false。
#### 为什么需要hashCode?
- 通过hashCode可以很快的查到小内存块。
- 通过hashCode比较比equal方法快，当get时先比较hashCode，如果hashCode不同，直接返回false。

#### Object.hashCode的通用约定（摘自《Effective Java》第45页）
- 在一个应用程序执行期间，如果一个对象的equals方法做比较所用到的信息没有被修改的话，那么，对该对象调用hashCode方法多次，它必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，这个整数可以不同，即这个应用程序这次执行返回的整数与下一次执行返回的整数可以不一致。
- 如果两个对象根据equals(Object)方法是相等的，那么调用这两个对象中任一个对象的hashCode方法必须产生同样的整数结果。
- 如果两个对象根据equals(Object)方法是不相等的，那么调用这两个对象中任一个对象的hashCode方法，不要求必须产生不同的整数结果。然而，程序员应该意识到这样的事实，对于不相等的对象产生截然不同的整数结果，有可能提高散列表（hash table）的性能。

如果只重写了equals方法而没有重写hashCode方法的话，则会违反约定的第二条：相等的对象必须具有相等的散列码（hashCode）
     同时对于HashSet和HashMap这些基于散列值（hash）实现的类。HashMap的底层处理机制是以数组的方法保存放入的数据的(Node<K,V>[] table)，其中的关键是数组下标的处理。数组的下标是根据传入的元素hashCode方法的返回值再和特定的值异或决定的。如果该数组位置上已经有放入的值了，且传入的键值相等则不处理，若不相等则覆盖原来的值，如果数组位置没有条目，则插入，并加入到相应的链表中。检查键是否存在也是根据hashCode值来确定的。所以如果不重写hashCode的话，可能导致HashSet、HashMap不能正常的运作、

原文：https://blog.csdn.net/zknxx/article/details/53862572 
### 例子




## 15、"equals(Object obj)" should test argument type


- 次要

> Because the `equals` method takes a generic `Object` as a parameter, any type of object may be passed to it. The method should not assume it will only be used to test objects of its class type. It must instead check the parameter's type.

>因为equals方法接受一个泛型对象作为参数，所以任何类型的对象都可以传递给它。该方法不应该假定它只用于测试其类类型的对象。它必须检查参数的类型。

### Noncompliant Code Example
```java
public boolean equals(Object obj) {
  MyClass mc = (MyClass)obj;  // Noncompliant
  // ...
}
```

### Compliant Solution
```java
public boolean equals(Object obj) {
  if (obj == null)
    return false;

  if (this.getClass() != obj.getClass())
    return false;

  MyClass mc = (MyClass)obj;
  // ...
}
```
### 一句话说明
- 重写`equals`方法，必须对入参是否是当前类的实例进行判断，因为入参是`Object`类型，允许传任何类型，否则容易出现类型转换异常





## 16、"Externalizable" classes should have no-arguments constructors

- 主要

> An Externalizable class is one which handles its own Serialization and deserialization. During deserialization, the first step in the process is a default instantiation using the class' no-argument constructor. Therefore an Externalizable class without a no-arg constructor cannot be deserialized.

> 外部化类是一个处理自己的序列化和反序列化的类。在反序列化期间，流程中的第一步是使用类的无参数构造函数进行默认实例化。因此，没有no-arg构造函数的Externalizable类不能反序列化。

### Noncompliant Code Example
```java
public class Tomato implements Externalizable {  // Noncompliant; no no-arg constructor

  public Tomato (String color, int weight) { ... }
}
```

### Compliant Solution
```java
public class Tomato implements Externalizable {

  public Tomato() { ... }
  public Tomato (String color, int weight) { ... }
}
```
### 一句话说明
- 实现Externalizable接口的类必须拥有默认构造方法

### 说明
- Serializable序列化时不会调用默认的构造器，而Externalizable序列化时会调用默认构造器的
- Serializable：一个对象想要被序列化，那么它的类就要实现 此接口，这个对象的所有属性（包括private属性、包括其引用的对象）都可以被序列化和反序列化来保存、传递。
- Externalizable：他是Serializable接口的子类，有时我们不希望序列化那么多，可以使用这个接口，这个接口的writeExternal()和readExternal()方法可以指定序列化哪些属性。



## 17、"getClass" should not be used for synchronization

- 主要

> getClass should not be used for synchronization in non-final classes because child classes will synchronize on a different object than the parent or each other, allowing multiple threads into the code block at once, despite the synchronized keyword.

> getClass不应该用于非最终类的同步，因为子类将在不同于父类或彼此的对象上同步，从而允许多个线程同时进入代码块，尽管synchronized关键字是这样的。

------------
>Instead, hard code the name of the class on which to synchronize or make the class final.

### Noncompliant Code Example
```java
public class MyClass {
  public void doSomethingSynchronized(){
    synchronized (this.getClass()) {  // Noncompliant
      // ...
    }
  }
```

### Compliant Solution
```java
public class MyClass {
  public void doSomethingSynchronized(){
    synchronized (MyClass.class) {
      // ...
    }
  }
```
### 一句话说明
- `synchronization`不能用`getClass()`，只能用硬编码的形式，如`ClassName.class`形式




## 18、"hashCode" and "toString" should not be called on array instances

- 主要

> While hashCode and toString are available on arrays, they are largely useless. hashCode returns the array's "identity hash code", and toString returns nearly the same value. Neither method's output actually reflects the array's contents. Instead, you should pass the array to the relevant static Arrays method.

> 虽然hashCode和toString可用于数组，但它们在很大程度上是无用的。hashCode返回数组的“标识哈希码”，toString返回几乎相同的值。这两个方法的输出实际上都没有反映数组的内容。相反，应该将数组传递给相关的静态数组方法。

### Noncompliant Code Example
```java
public static void main( String[] args )
{
    String argStr = args.toString(); // Noncompliant
    int argHash = args.hashCode(); // Noncompliant
}
```

### Compliant Solution
```java
public static void main( String[] args )
{
    String argStr = Arrays.toString(args);
    int argHash = Arrays.hashCode(args);
}
```
### 一句话说明
- 数组`Array`不能直接使用`toString()`和`hashCode()`方法，而是先使用`Arrays.toString(args)`转成`String`类型再使用`toString()`和`hashCode()`
### 例子
```java
    @Test
    public void test18() {
        String [] fruit = {"apple","banana","orange"};

        //反例
        String fruitToString = fruit.toString();//
        int fruitHashCode = fruit.hashCode();//
        System.out.println(fruitToString+"\t"+fruitHashCode);  //[Ljava.lang.String;@72967906    1922464006

        //正例
        String fruitToString2 = Arrays.toString(fruit); //
        int fruitHashCode2 = Arrays.hashCode(fruit); //
        System.out.println(fruitToString2+"\t"+fruitHashCode2);//[apple, banana, orange]    -2139403102
    }
```





## 19、"InterruptedException" should not be ignored

- 主要

> InterruptedExceptions should never be ignored in the code, and simply logging the exception counts in this case as "ignoring". 
The throwing of the InterruptedException clears the interrupted state of the Thread, so if the exception is not handled properly the fact that the thread was interrupted will be lost. 
Instead, InterruptedExceptions should either be rethrown - immediately or after cleaning up the method's state - or the thread should be re-interrupted by calling Thread.interrupt() even if this is supposed to be a single-threaded application. 
Any other course of action risks delaying thread shutdown and loses the information that the thread was interrupted - probably without finishing its task.

> 在代码中不应该忽略interruptedexception，在本例中只记录异常计数为“忽略”。抛出InterruptedException会清除线程的中断状态，因此如果没有正确处理异常，那么线程被中断的事实将会丢失。相反，interruptedexception应该被重新抛出(立即抛出或在清理方法的状态之后抛出)，或者通过调用thread .interrupt()来重新中断线程，即使这应该是一个单线程应用程序。任何其他的行动都有可能拖延时间

------------

>Similarly, the ThreadDeath exception should also be propagated. According to its JavaDoc:
>类似地，ThreadDeath异常也应该传播。根据它的JavaDoc

>If ThreadDeath is caught by a method, it is important that it be rethrown so that the thread actually dies.
>如果ThreadDeath被一个方法捕获，那么必须重新抛出它，这样线程才会死。
### Noncompliant Code Example
```java
public void run () {
  try {
    while (true) {
      // do stuff
    }
  }catch (InterruptedException e) { // Noncompliant; logging is not enough
    LOGGER.log(Level.WARN, "Interrupted!", e);
  }
}
```

### Compliant Solution
```java
public void run () {
  try {
    while (true) {
      // do stuff
    }
  }catch (InterruptedException e) {
    LOGGER.log(Level.WARN, "Interrupted!", e);
    // Restore interrupted state...
    Thread.currentThread().interrupt();
  }
}
```
### 一句话说明
- 必须处理InterruptedException异常，绝对不能忽略它，而不仅仅打印错误日志

### 原因分析
[处理InterruptedException](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html "处理InterruptedException")





## 20、"iterator" should not return "this"

- 主要

>There are two classes in the Java standard library that deal with iterations: `Iterable<T>` and `Iterator<T>`. An `Iterable<T>` represents a data structure that can be the target of the "for-each loop" statement, and an `Iterator<T>` represents the state of an ongoing traversal. An `Iterable<T>` is generally expected to support multiple traversals.
Java标准库中有两个类处理迭代:`Iterable<T>`和`Iterator<T>`。`Iterable<T>`表示可以作为“for-each loop”语句目标的数据结构，`Iterator<T>`表示正在进行的遍历的状态。`Iterable<T>`通常支持多次遍历。

>An Iterator<T> that also implements Iterable<t> by returning itself as its iterator() will not support multiple traversals since its state will be carried over.
Iterator<T>也实现了Iterable< T>，方法是将自身作为Iterator()返回，因为它的状态将被传递，所以不支持多次遍历。

>This rule raises an issue when the iterator() method of a class implementing both Iterable<T> and Iterator<t> returns this.
当实现Iterable<T>和iterator <T>的类的iterator()方法返回这个值时，这个规则会引发一个问题。

### Noncompliant Code Example
```java
class FooIterator implements Iterator<Foo>, Iterable<Foo> {
  private Foo[] seq;
  private int idx = 0;

  public boolean hasNext() {
    return idx < seq.length;
  }

  public Foo next() {
    return seq[idx++];
  }

  public Iterator<Foo> iterator() {
    return this; // Noncompliant
  }
  // ...
}
```

### Compliant Solution
```java
class FooSequence implements Iterable<Foo> {
  private Foo[] seq;

  public Iterator<Foo> iterator() {
    return new Iterator<Foo>() {
      private int idx = 0;

      public boolean hasNext() {
        return idx < seq.length;
      }

      public Foo next() {
        return seq[idx++];
      }
    };
  }
  // ...
}
```
### 一句话说明
- 构造iterator对象，不能直接共享同一个，需要重新生成来遍历

## 技术说明
- [iterator接口源码分析（ArrayList中的实现）](https://blog.csdn.net/HBL6016/article/details/80890823 "iterator接口源码分析（ArrayList中的实现）")
- [java的Iterator源码浅析](https://www.cnblogs.com/expiator/p/6125141.html "java的Iterator源码浅析")




## 21、"Iterator.hasNext()" should not call "Iterator.next()"

- 主要

>Calling `Iterator.hasNext()` is not supposed to have any side effects, and therefore should not change the state of the iterator. `Iterator.next()` advances the iterator by one item. So calling it inside `Iterator.hasNext()`, breaks the `hasNext()` contract, and will lead to unexpected behavior in production.
调用`iterator. hasnext()`不应该有任何副作用，因此不应该改变iterator的状态。next()将迭代器向前推进一项。因此，在Iterator.hasNext()中调用它，会破坏hasNext()契约，并导致生产中出现意外行为。

### Noncompliant Code Example
```java
public class FibonacciIterator implements Iterator<Integer>{
...
@Override
public boolean hasNext() {
  if(next() != null) {
    return true;
  }
  return false;
}
...
}
```
### 一句话说明
- `Iterator.hasNext()`方法里面不允许调用`hasNext()`方法




## 22、"Iterator.next()" methods should throw "NoSuchElementException"

- 次要

>By contract, any implementation of the `java.util.Iterator.next()` method should throw a `NoSuchElementException` exception when the iteration has no more elements. Any other behavior when the iteration is done could lead to unexpected behavior for users of this `Iterator`.
根据约定，当迭代没有更多元素时，java.util.Iterator.next()方法的任何实现都应该抛出NoSuchElementException异常。迭代完成时的任何其他行为都可能导致此`Iterator`用户的意外行为。

#### Noncompliant Code Example
```java
public class MyIterator implements Iterator<String>{
  ...
  public String next(){
    if(!hasNext()){
      return null;
    }
    ...
  }
}
```

#### Compliant Solution
```java
public class MyIterator implements Iterator<String>{
  ...
  public String next(){
    if(!hasNext()){
      throw new NoSuchElementException();
    }
    ...
  }
}
```
#### 一句话说明
- 实现`Iterator.next()`方法时，如果遇到没有更多元素时，需要抛出`NoSuchElementException`异常





## 23、"notifyAll" should be used

- 主要

>`notify` and `notifyAll` both wake up sleeping threads, but `notify` only rouses one, while `notifyAll` rouses all of them. Since `notify` might not wake up the right thread, `notifyAll` should be used instead.


### Noncompliant Code Example
```java
class MyThread extends Thread{

  @Override
  public void run(){
    synchronized(this){
      // ...
      notify();  // Noncompliant
    }
  }
}
```

### Compliant Solution
```java
class MyThread extends Thread{

  @Override
  public void run(){
    synchronized(this){
      // ...
      notifyAll();
    }
  }
}
```
### See

-  [CERT, THI02-J.](https://www.securecoding.cert.org/confluence/x/OoAlAQ) - Notify all waiting threads rather than a single thread
### 一句话说明
- 在线程中尽可能地使用`notifyAll()`，而不是`notify()`




## 24、"null" should not be used with "Optional"

- 主要

>The concept of `Optional` is that it will be used when `null` could cause errors. In a way, it replaces `null`, and when `Optional` is in use, there should never be a question of returning or receiving `null` from a call.
`Optional`的概念是，当`null`可能导致错误时将使用它。在某种程度上，当使用Optional时，它替换了null，应该不会有返回或接收调用null的问题。

### Noncompliant Code Example
```java
public void doSomething () {
  Optional<String> optional = getOptional();
  if (optional != null) {  // Noncompliant
    // do something with optional...
  }
}

@Nullable // Noncompliant
public Optional<String> getOptional() {
  // ...
  return null;  // Noncompliant
}
```

### Compliant Solution
```java
public void doSomething () {
  Optional<String> optional = getOptional();
  optional.ifPresent(
    // do something with optional...
  );
}

public Optional<String> getOptional() {
  // ...
  return Optional.empty();
}
```
### 一句话说明
- 把判空包装起来使用而不直接使用!=null
- Optional不是对null关键字的一种替代，而是对于null判定提供了一种更加优雅的实现。

### 技术说明
- [ JDK8新特性Optional类](https://blog.csdn.net/u012129558/article/details/78654751 " JDK8新特性Optional类")




## 25、"PreparedStatement" and "ResultSet" methods should be called with valid indices

- 阻断

>The parameters in a `PreparedStatement` are numbered from 1, not 0, so using any "set" method of a `PreparedStatement` with a number less than 1 is a bug, as is using an index higher than the number of parameters. Similarly, `ResultSet` indices also start at 1, rather than 0
PreparedStatement中的参数是从1开始编号的，而不是0，所以使用数值小于1的PreparedStatement的任何“set”方法都是错误的，就像使用高于参数数量的索引一样。类似地，ResultSet索引也从1开始，而不是0

### Noncompliant Code Example
```java
PreparedStatement ps = con.prepareStatement("SELECT fname, lname FROM employees where hireDate > ? and salary < ?");
ps.setDate(0, date);  // Noncompliant
ps.setDouble(3, salary);  // Noncompliant

ResultSet rs = ps.executeQuery();
while (rs.next()) {
  String fname = rs.getString(0);  // Noncompliant
  // ...
}
```

### Compliant Solution
```java
PreparedStatement ps = con.prepareStatement("SELECT fname, lname FROM employees where hireDate > ? and salary < ?");
ps.setDate(1, date);
ps.setDouble(2, salary);

ResultSet rs = ps.executeQuery();
while (rs.next()) {
  String fname = rs.getString(1);
  // ...
}
```
### 一句话说明
-  设置`PreparedStatement`参数 和 读取`ResultSet`数据游标都是从1开始，而不是0，否则报异常





## 26、"Random" objects should be reused

- 严重

>Creating a new `Random` object each time a random value is needed is inefficient and may produce numbers which are not random depending on the JDK. For better efficiency and randomness, create a single `Random`, then store, and reuse it.
每次需要一个随机值时创建一个新的随机对象是低效的，并且可能会产生非随机的数字，这取决于JDK。为了提高效率和随机性，创建一个随机变量，然后存储并重用它。

>The `Random()` constructor tries to set the seed with a distinct value every time. However there is no guarantee that the seed will be random or even uniformly distributed. Some JDK will use the current time as seed, which makes the generated numbers not random at all.
Random()构造函数每次都尝试用不同的值设置种子。然而，并不能保证种子是随机的，甚至是均匀分布的。一些JDK将使用当前时间作为种子，这使得生成的数字完全不是随机的。

>This rule finds cases where a new `Random` is created each time a method is invoked and assigned to a local random variable.
此规则发现在每次调用方法并将其分配给局部随机变量时都会创建一个新随机变量的情况。

### Noncompliant Code Example
```java
public void doSomethingCommon() {
  Random rand = new Random();  // Noncompliant; new instance created with each invocation
  int rValue = rand.nextInt();
  //...
```

### Compliant Solution
```java
private Random rand = SecureRandom.getInstanceStrong();  // SecureRandom is preferred to Random

public void doSomethingCommon() {
  int rValue = this.rand.nextInt();
  //...
```
### Exceptions
>A class which uses a Random in its constructor or in a static main function and nowhere else will be ignored by this rule.
在构造函数或静态主函数中使用随机变量的类，其他任何地方都不会被此规则忽略。

### 一句话说明
-  `Random`对象实例需要重复使用，而不应该在每个方法里面每次调用都实例化一次






## 27、"read" and "readLine" return values should be used
- 主要

>When a method is called that returns data read from some data source, that data should be stored rather than thrown away. Any other course of action is surely a bug.
当调用返回从某个数据源读取的数据的方法时，应该存储而不是丢弃该数据。任何其他的做法都肯定是错误的。

------------

>This rule raises an issue when the return value of any of the following is ignored or merely null-checked: `BufferedReader.readLine()`, `Reader.read()`, and these methods in any child classes.
当忽略或仅检查空值时，此规则会引发一个问题:BufferedReader.readLine()、Reader.read()和任何子类中的这些方法

### Noncompliant Code Example
```java
public void doSomethingWithFile(String fileName) {
  BufferedReader buffReader = null;
  try {
    buffReader = new BufferedReader(new FileReader(fileName));
    while (buffReader.readLine() != null) { // Noncompliant
      // ...
    }
  } catch (IOException e) {
    // ...
  }
}
```

### Compliant Solution
```java
public void doSomethingWithFile(String fileName) {
  BufferedReader buffReader = null;
  try {
    buffReader = new BufferedReader(new FileReader(fileName));
    String line = null;
    while ((line = buffReader.readLine()) != null) {
      // ...
    }
  } catch (IOException e) {
    // ...
  }
}
```
### 一句话说明
- 调用`read()`或`readLine()`方法时，需要存储和使用这方法的返回值，而不是丢弃




## 28、"runFinalizersOnExit" should not be called

- 主要

>Running finalizers on JVM exit is disabled by default. It can be enabled with `System.runFinalizersOnExit` and `Runtime.runFinalizersOnExit`, but both methods are deprecated because they are are inherently unsafe.
默认情况下，在JVM退出上运行终结器是禁用的。它可以在System.runFinalizersOnExit和Runtime.runFinalizersOnExit中启用，但是这两个方法都是不推荐的，因为它们本质上是不安全的。

> It may result in finalizers being called on live objects while other threads are concurrently manipulating those objects, resulting in erratic behavior or deadlock.
If you really want to be execute something when the virtual machine begins its shutdown sequence, you should attach a shutdown hook.
它可能导致在活动对象上调用finalizers，而其他线程正在同时操作这些对象，从而导致不稳定的行为或死锁。
如果您真的想在虚拟机开始关闭序列时执行某些操作，您应该附加一个关闭钩子。

### Noncompliant Code Example
```java
public static void main(String [] args) {
  ...
  System.runFinalizersOnExit(true);  // Noncompliant
  ...
}

protected void finalize(){
  doSomething();
}
```

### Compliant Solution
```java
public static void main(String [] args) {
  Runtime.addShutdownHook(new Runnable() {
    public void run(){
      doSomething();
    }
  });
  //...
```
### See
-   [CERT, MET12-J.](https://www.securecoding.cert.org/confluence/x/H4cbAQ) - Do not use finalizers

### 一句话说明
- 不应该直接使用`runFinalizersOnExit`方法




## 29、"ScheduledThreadPoolExecutor" should not have 0 core threads

- 严重

>java.util.concurrent.ScheduledThreadPoolExecutor's pool is sized with corePoolSize, so setting corePoolSize to zero means the executor will have no threads and run nothing.
This rule detects instances where corePoolSize is set to zero, via either its setter or the object constructor.
>java.util.concurrent.ScheduledThreadPoolExecutor的池大小与corePoolSize相同，因此将corePoolSize设置为零意味着executor将没有线程，也不运行任何东西。

------------
>This rule detects instances where corePoolSize is set to zero, via either its setter or the object constructor
此规则通过其setter或对象构造函数检测corePoolSize设置为零的实例

### Noncompliant Code Example
```java
public void do(){

  ScheduledThreadPoolExecutor stpe1 = new ScheduledThreadPoolExecutor(0); // Noncompliant

  ScheduledThreadPoolExecutor stpe2 = new ScheduledThreadPoolExecutor(POOL_SIZE);
  stpe2.setCorePoolSize(0);  // Noncompliant
```

### 一句话说明
- 不允许设置ScheduledThreadPoolExecutor的线程池大小为0





## 30、"Serializable" inner classes of non-serializable classes should be "static"

- 严重

>Serializing a non-`static` inner class will result in an attempt at serializing the outer class as well. If the outer class is not serializable, then serialization will fail, resulting in a runtime error.
序列化非静态内部类也会导致序列化外部类的尝试。如果外部类不可序列化，则序列化将失败，从而导致运行时错误。


>Making the inner class `static` (i.e. "nested") avoids this problem, therefore inner classes should be `static` if possible. However, you should be aware that there are semantic differences between an inner class and a nested one:
使内部类静态(即“嵌套”)避免了这个问题，因此内部类应该是静态的(如果可能的话)。但是，您应该注意到内部类和嵌套类之间存在语义差异:
- an inner class can only be instantiated within the context of an instance of the outer class.
内部类只能在外部类的实例上下文中实例化。
- a nested (`static`) class can be instantiated independently of the outer class.
嵌套(静态)类可以独立于外部类实例化。

### Noncompliant Code Example
```java
public class Pomegranate {
  // ...

  public class Seed implements Serializable {  // Noncompliant; serialization will fail
    // ...
  }
}
```
### Compliant Solution
```java
public class Pomegranate {
  // ...

  public static class Seed implements Serializable {
    // ...
  }
}
```
### See
- [CERT SER05-J.](https://www.securecoding.cert.org/confluence/x/O4CpAQ) - Do not serialize instances of inner classes

### 一句话说明
-  序列化非静态内部类将导致尝试序列化外部类,如果外部类不是序列化类,会产生运行时异常，内部类静态化会避免这种情况





## 31、"SingleConnectionFactory" instances should be set to "reconnectOnException"

- 主要

>Use of a Spring SingleConnectionFactory without enabling the reconnectOnException setting will prevent automatic connection recovery when the connection goes bad.
在不启用reconnectOnException设置的情况下使用Spring SingleConnectionFactory将防止在连接出错时自动恢复连接。

>That's because the reconnectOnException property defaults to false. As a result, even if the code that uses this connection factory (Spring's DefaultMessageListenerContainer or your own code) has reconnect logic, that code won't work because the SingleConnectionFactory will act like a single-connection pool by preventing connection close calls from actually closing anything. As a result, subsequent factory create operations will just hand back the original broken Connection.
这是因为reconnectOnException属性默认为false。因此,即使使用此连接工厂的代码(Spring的DefaultMessageListenerContainer或您自己的代码)连接的逻辑,这些代码不会奏效,因为SingleConnectionFactory将像一个单个连接池通过阻止连接关闭调用实际上关闭任何东西。因此，后续的工厂创建操作将只返回原始断开的连接。
### Noncompliant Code Example
```java
<bean id="singleCF" class="org.springframework.jms.connection.SingleConnectionFactory">  <!-- Noncompliant -->
   <constructor-arg ref="dummyConnectionFactory" />
 </bean>
```
### Compliant Solution
```java
<bean id="singleCF" class="org.springframework.jms.connection.SingleConnectionFactory" p:reconnectOnException="true">
   <constructor-arg ref="dummyConnectionFactory" />
 </bean>
```
or
```java
<bean id="singleCF" class="org.springframework.jms.connection.SingleConnectionFactory">
   <constructor-arg ref="dummyConnectionFactory" />
   <property name="reconnectOnException"><value>true</value></property>
 </bean>
```
### 一句话说明
- 使用Spring SingleConnectionFactory而不启用reconnectOnException设置当连接恶化将阻止自动连接恢复





## 32、"StringBuilder" and "StringBuffer" should not be instantiated with a character

- 主要

>Instantiating a `StringBuilder` or a `StringBuffer` with a character is misleading because most Java developers would expect the character to be the initial value of the `StringBuffer`.
用字符实例化StringBuilder或StringBuffer是误导人的，因为大多数Java开发人员都希望字符是StringBuffer的初始值。

------------

>What actually happens is that the int representation of the character is used to determine the initial size of the `StringBuffer`.
实际上，字符的int表示形式用于确定StringBuffer的初始大小。

### Noncompliant Code Example
```java
StringBuffer foo = new StringBuffer('x');   //equivalent to StringBuffer foo = new StringBuffer(120);
```
### Compliant Solution
```java
StringBuffer foo = new StringBuffer("x");
```
### 一句话说明
- 不允许使用字符构造`StringBuilder` and `StringBuffer`对象，使用字符会构造会把它当容量对待

### 技术说明
- [ASCII对照表](http://tool.oschina.net/commons?type=4 "ASCII对照表")
- new StringBuffer('x')跟new StringBuffer(120)是一个意思，因为在ASCII当中'x'对应的就是120这个值

### 例子
```java
    @Test
    public void test32() {
        System.out.println(new StringBuffer('x')); //打印空
        System.out.println(new StringBuffer("x")); //打印x
        System.out.println(new StringBuilder('x'));//打印空
        System.out.println(new StringBuilder("x"));//打印x
    }
```




## 33、"super.finalize()" should be called at the end of "Object.finalize()" implementations

- 严重

>Overriding the `Object.finalize()` method must be done with caution to dispose some system resources.
必须谨慎地覆盖Object.finalize()方法，以处理一些系统资源。

------------

>Calling the `super.finalize()` at the end of this method implementation is highly recommended in case parent implementations must also dispose some system resources.
强烈建议在方法实现的末尾调用super.finalize()，以防父实现也必须配置一些系统资源。

### Noncompliant Code Example
```java
protected void finalize() {   // Noncompliant; no call to super.finalize();
  releaseSomeResources();
}

protected void finalize() {
  super.finalize();  // Noncompliant; this call should come last
  releaseSomeResources();
}
```
### Compliant Solution
```java
protected void finalize() {
  releaseSomeResources();
  super.finalize();
}
```
### See

-   [MITRE, CWE-568](http://cwe.mitre.org/data/definitions/568.html) - finalize() Method Without super.finalize()
-   [CERT, MET12-J.](https://www.securecoding.cert.org/confluence/x/H4cbAQ) - Do not use finalizers

### 一句话说明
- super.finalize()必须在子类重载finalize方法的最后来调用

### 技术说明
- 参考：[不要使用 finalize()](https://blog.csdn.net/weixin_37891479/article/details/81016130 "不要使用 finalize()")




## 34、"Thread.run()" should not be called directly

- 主要

>The purpose of the `Thread.run()` method is to execute code in a separate, dedicated thread. Calling this method directly doesn't make sense because it causes its code to be executed in the current thread.
run()方法的目的是在单独的专用线程中执行代码。直接调用这个方法没有意义，因为它会导致其代码在当前线程中执行。

------------

>To get the expected behavior, call the `Thread.start()` method instead.
要获得预期的行为，可以调用Thread.start()方法。

### Noncompliant Code Example
```java
Thread myThread = new Thread(runnable);
myThread.run(); // Noncompliant
```
### Compliant Solution
```java
Thread myThread = new Thread(runnable);
myThread.start(); // Compliant
```
### 一句话说明
- 不允许直接调用`Thread.run()`方法

### See
-   [MITRE, CWE-572](http://cwe.mitre.org/data/definitions/572.html) - Call to Thread run() instead of start()
-   [CERT THI00-J.](https://www.securecoding.cert.org/confluence/x/KQAiAg) - Do not invoke Thread.run()





## 35、"toArray" should be passed an array of the proper type

- 次要

>Given no arguments, the `Collections.toArray` method returns an `Object []`, which will cause a `ClassCastException` at runtime if you try to cast it to an array of the proper class. Instead, pass an array of the correct type in to the call.
如果没有参数，集合。toArray方法返回一个对象[]，如果您试图将其转换为适当类的数组，则会在运行时引发ClassCastException。相反，将正确类型的数组传递给调用。

### Noncompliant Code Example
```java
public String [] getStringArray(List<String> strings) {
  return (String []) strings.toArray();  // Noncompliant; ClassCastException thrown
}
```
### Compliant Solution
```java
public String [] getStringArray(List<String> strings) {
  return strings.toArray(new String[0]);
}
```
### 一句话说明
- 调用toArray()方法，需要传对应的参数，否则强制转换有可能出现类型转换失败。应该显示调用，而不是强制转换




## 36、"toString()" and "clone()" methods should not return null

- 主要

>Calling `toString()` or `clone()` on an object should always return a string or an object. Returning `null` instead contravenes the method's implicit contract.
在对象上调用`toString()`或`clone()`应该总是返回字符串或对象。返回`null`反而违反了方法的隐式契约

### Noncompliant Code Example
```java
public String toString () {
  if (this.collection.isEmpty()) {
    return null; // Noncompliant
  } else {
    // ...
 {code}
```
### Compliant Solution
```java
public String toString () {
  if (this.collection.isEmpty()) {
    return "";
  } else {
    // ...
```
### See

-   [MITRE CWE-476](http://cwe.mitre.org/data/definitions/476.html) - NULL Pointer Dereference
-   [CERT, EXP01-J.](https://www.securecoding.cert.org/confluence/x/ZwDOAQ) - Do not use a null in a case where an object is required
### 一句话说明
- 重载`toString()`或`clone()`方法，必须返回字符串或者对象，而不能返回`null`






## 37、"volatile" variables should not be used with compound operators

- 主要

>Using compound operators as well as increments and decrements (and toggling, in the case of `boolean`s) on primitive fields are not atomic operations. That is, they don't happen in a single step. For instance, when a `volatile` primitive field is incremented or decremented you run the risk of data loss if threads interleave in the steps of the update. Instead, use a guaranteed-atomic class such as `AtomicInteger`, or synchronize the access.
在基本字段上使用复合运算符以及递增和递减(在布尔值的情况下，还可以切换)不是原子操作。也就是说，它们不是一步完成的。例如，当易失性原语字段增加或减少时，如果线程在更新的步骤中交错，则会有数据丢失的风险。相反，使用一个有保证的原子类，比如AtomicInteger，或者同步访问。


### Noncompliant Code Example
```java
private volatile int count = 0;
private volatile boolean boo = false;

public void incrementCount() {
  count++;  // Noncompliant
}

public void toggleBoo(){
  boo = !boo;  // Noncompliant
}
```
### Compliant Solution
```java
private AtomicInteger count = 0;
private boolean boo = false;

public void incrementCount() {
  count.incrementAndGet();
}

public synchronized void toggleBoo() {
  boo = !boo;
}
```
### See
-   [CERT, VNA02-J.](https://www.securecoding.cert.org/confluence/x/RIFJAg) - Ensure that compound operations on shared variables are atomic

### 一句话说明
- volatile声明的变量不能用于复合运算
- 被volatile修饰的变量对所有线程总数立即可见的，对volatile变量的所有写操作总是能立刻反应到其他线程中，但是对于volatile变量运算操作在多线程环境并不保证安全性

### 技术说明
- [全面理解Java内存模型(JMM)及volatile关键字](https://blog.csdn.net/javazejian/article/details/72772461 "全面理解Java内存模型(JMM)及volatile关键字")





## 38、"wait" should not be called when multiple locks are held

- 阻断

>When two locks are held simultaneously, a `wait` call only releases one of them. The other will be held until some other thread requests a lock on the awaited object. If no unrelated code tries to lock on that object, then all other threads will be locked out, resulting in a deadlock.
当同时持有两个锁时，等待调用只释放其中一个锁。另一个线程将被持有，直到其他线程请求锁定所等待的对象。如果没有不相关的代码试图锁定该对象，那么所有其他线程都将被锁定，从而导致死锁。


### Noncompliant Code Example
```java
synchronized (this.mon1) {  // threadB can't enter this block to request this.mon2 lock & release threadA
    synchronized (this.mon2) {
        this.mon2.wait();  // Noncompliant; threadA is stuck here holding lock on this.mon1
    }
}
```
### 一句话说明
- 当持有多个锁时，不应调用`wait`方法




## 39、"wait", "notify" and "notifyAll" should only be called when a lock is obviously held on an object

- 主要

>By contract, the method `Object.wait(...)`, `Object.notify()` and `Object.notifyAll()` should be called by a thread that is the owner of the object's monitor. If this is not the case an `IllegalMonitorStateException` exception is thrown. This rule reinforces this constraint by making it mandatory to call one of these methods only inside a `synchronized` method or statement.
根据约定，方法`object.wait(…)`、`object.notify()`和
`object.notifyall()`应该由对象监视器的所有者的线程调用。如果不是这种情况，则抛出IllegalMonitorStateException异常。此规则通过强制只在同步方法或语句中调用其中一个方法来加强此约束。


### Noncompliant Code Example
```java
private void removeElement() {
  while (!suitableCondition()){
    obj.wait();
  }
  ... // Perform removal
}
```
or
```java
private void removeElement() {
  while (!suitableCondition()){
    wait();
  }
  ... // Perform removal
}
```

### Compliant Solution
```java
private void removeElement() {
  synchronized(obj) {
    while (!suitableCondition()){
      obj.wait();
    }
    ... // Perform removal
  }
}
```
or
```java
private synchronized void removeElement() {
  while (!suitableCondition()){
    wait();
  }
  ... // Perform removal
}
```

### 一句话说明
-  先要获得对象锁才能进行上述操作






## 40、"wait(...)" should be used instead of "Thread.sleep(...)" when a lock is held

- 阻断

>If `Thread.sleep(...)` is called when the current thread holds a lock, it could lead to performance and scalability issues, or even worse to deadlocks because the execution of the thread holding the lock is frozen. It's better to call `wait(...)` on the monitor object to temporarily release the lock and allow other threads to run.
如果在当前线程持有锁时调用`thread.sleep(…)`，可能会导致性能和可伸缩性问题，甚至会导致死锁，因为持有锁的线程的执行被冻结。最好调用`monitor`对象上的`wait(…)`来临时释放锁并允许其他线程运行。


#### Noncompliant Code Example
```java
public void doSomething(){
  synchronized(monitor) {
    while(notReady()){
      Thread.sleep(200);
    }
    process();
  }
  ...
}
```

#### Compliant Solution
```java
public void doSomething(){
  synchronized(monitor) {
    while(notReady()){
      monitor.wait(200);
    }
    process();
  }
  ...
}
```
#### See
-   [CERT, LCK09-J.](https://www.securecoding.cert.org/confluence/x/FgG7AQ) - Do not perform operations that can block while holding a lock

#### 一句话说明
- 当锁被持有时，应该使用`wait(...)`而不是`Thread.slee(...)`
- 当持有锁的当前线程调用`Thread.sleep(...)`可能导致性能和扩展性问题，甚至死锁因为持有锁的当前线程已冻结.合适的做法是锁对象`wait()`释放锁让其它线程进来运行.





## 41、A "for" loop update clause should move the counter in the right direction

- 主要

>A `for` loop with a counter that moves in the wrong direction is not an infinite loop. Because of wraparound, the loop will eventually reach its stop condition, but in doing so, it will run many, many more times than anticipated, potentially causing unexpected behavior.
带有计数器向错误方向移动的for循环不是一个无限循环。由于循环的结束，循环最终将达到停止状态，但在此过程中，它将运行比预期多得多的次数，这可能会导致意外行为。


### Noncompliant Code Example
```java
public void doSomething(String [] strings) {
  for (int i = 0; i < strings.length; i--) { // Noncompliant;
    String string = strings[i];  // ArrayIndexOutOfBoundsException when i reaches -1
    //...
  }
```

### Compliant Solution
```java
public void doSomething(String [] strings) {
  for (int i = 0; i < strings.length; i++) {
    String string = strings[i];
    //...
  }
```
### See
-   [CERT, MSC54-J.](https://www.securecoding.cert.org/confluence/x/zYEzAg) - Avoid inadvertent wrapping of loop counters

### 一句话说明
- for循环i++不能写成i--





## 42、All branches in a conditional structure should not have exactly the same implementation

- 主要

>Having all branches in a `switch` or `if` chain with the same implementation is an error. Either a copy-paste error was made and something different should be executed, or there shouldn't be a `switch`/`if` chain at all.
在`switch` or `if`链中具有相同实现的所有分支是错误的。要么复制粘贴错误，应该执行一些不同的操作，要么根本不应该有开关/if链。


### Noncompliant Code Example
```java
if (b == 0) {  // Noncompliant
  doOneMoreThing();
} else {
  doOneMoreThing();
}

int b = a > 12 ? 4 : 4;  // Noncompliant

switch (i) {  // Noncompliant
  case 1:
    doSomething();
    break;
  case 2:
    doSomething();
    break;
  case 3:
    doSomething();
    break;
  default:
    doSomething();
}
```

### Exceptions
This rule does not apply to if chains without else-s, or to switch-es without default clauses.
此规则不适用于没有else-s的if链，也不适用于没有默认子句的switch-es。
```java
if(b == 0) {    //no issue, this could have been done on purpose to make the code more readable
  doSomething();
} else if(b == 1) {
  doSomething();
}
```
### See
-   [CERT, MSC54-J.](https://www.securecoding.cert.org/confluence/x/zYEzAg) - Avoid inadvertent wrapping of loop counters

### 一句话说明
- 分支中不应该有相同的实现





## 43、Blocks should be synchronized on "private final" fields

- 主要

>Synchronizing on a class field synchronizes not on the field itself, but on the object assigned to it. So synchronizing on a non-`final` field makes it possible for the field's value to change while a thread is in a block synchronized on the old value. That would allow a second thread, synchronized on the new value, to enter the block at the same time.
在类字段上同步不是在字段本身上同步，而是在分配给它的对象上同步。因此，在非final字段上同步可以使字段的值在线程处于与旧值同步的块中时发生更改。这将允许第二个线程(在新值上同步)同时进入块。

------------

>The story is very similar for synchronizing on parameters; two different threads running the method in parallel could pass two different object instances in to the method as parameters, completely undermining the synchronization.
在参数同步方面，情况非常相似;并行运行方法的两个不同线程可以将两个不同的对象实例作为参数传递给方法，这完全破坏了同步。

### Noncompliant Code Example
```java
private String color = "red";

private void doSomething(){
  synchronized(color) {  // Noncompliant; lock is actually on object instance "red" referred to by the color variable
    //...
    color = "green"; // other threads now allowed into this block
    // ...
  }
  synchronized(new Object()) { // Noncompliant this is a no-op.
     // ...
  }
}
```

### Compliant Solution
```java
private String color = "red";
private final Object lockObj = new Object();

private void doSomething(){
  synchronized(lockObj) {
    //...
    color = "green";
    // ...
  }
}
```
### See

-   [MITRE, CWE-412](http://cwe.mitre.org/data/definitions/412.html) - Unrestricted Externally Accessible Lock
-   [MITRE, CWE-413](http://cwe.mitre.org/data/definitions/413) - Improper Resource Locking
-   [CERT, LCK00-J.](https://www.securecoding.cert.org/confluence/x/6IEzAg) - Use private final lock objects to synchronize classes that may interact with untrusted code

### 一句话说明
- 块应该在“私有final”字段上同步






## 44、Boxing and unboxing should not be immediately reversed

- 次要

>Boxing is the process of putting a primitive value into an analogous object, such as creating an `Integer` to hold an `int` value. Unboxing is the process of retrieving the primitive value from such an object.
装箱是将原始值放入类似对象的过程，例如创建一个整数来保存int值。解装箱是从这样一个对象中检索原始值的过程。

------------

>Since the original value is unchanged during boxing and unboxing, there's no point in doing either when not needed. This also applies to autoboxing and auto-unboxing (when Java implicitly handles the primitive/object transition for you).
由于初始值在装箱和拆箱过程中是不变的，所以在不需要装箱和拆箱时两者都不做是没有意义的。这也适用于自动装箱和自动拆箱(当Java隐式地为您处理原语/对象转换时)。

### Noncompliant Code Example
```java
public void examineInt(int a) {
  //...
}

public void examineInteger(Integer a) {
  // ...
}

public void func() {
  int i = 0;
  Integer iger1 = Integer.valueOf(0);
  double d = 1.0;

  int dIntValue = new Double(d).intValue(); // Noncompliant

  examineInt(new Integer(i).intValue()); // Noncompliant; explicit box/unbox
  examineInt(Integer.valueOf(i));  // Noncompliant; boxed int will be auto-unboxed

  examineInteger(i); // Compliant; value is boxed but not then unboxed
  examineInteger(iger1.intValue()); // Noncompliant; unboxed int will be autoboxed

  Integer iger2 = new Integer(iger1); // Noncompliant; unnecessary unboxing, value can be reused
}
```

### Compliant Solution
```java
public void examineInt(int a) {
  //...
}

public void examineInteger(Integer a) {
  // ...
}

public void func() {
  int i = 0;
  Integer iger1 = Integer.valueOf(0);
  double d = 1.0;

  int dIntValue = (int) d;

  examineInt(i);

  examineInteger(i);
  examineInteger(iger1);
}
```
### 一句话说明
- 装箱和拆箱不应立即反转







## 45、Child class methods named for parent class methods should be overrides

- 主要

>When a method in a child class has the same signature as a method in a parent class, it is assumed to be an override.
当子类中的方法与父类中的方法具有相同的签名时，假定它是覆盖的

However, that's not the case when:
- the parent class method is `static` and the child class method is not.
父类方法是静态的，子类方法不是。
- the arguments or return types of the child method are in different packages than those of the parent method.
子方法的参数或返回类型位于不同于父方法的包中。
- the parent class method is `private`.
父类方法是私有的。

------------

>Typically, these things are done unintentionally; the private parent class method is overlooked, the `static` keyword in the parent declaration is overlooked, or the wrong class is imported in the child. But if the intent is truly for the child class method to be different, then the method should be renamed to prevent confusion.
通常，这些事情都是无意中发生的;将忽略私有父类方法，忽略父声明中的静态关键字，或者将错误的类导入子类。但是，如果真正的意图是让子类方法不同，那么应该重新命名该方法，以防止混淆。

### Noncompliant Code Example
```java
// Parent.java
import computer.Pear;
public class Parent {

  public void doSomething(Pear p) {
    //,,,
  }

  public static void doSomethingElse() {
    //...
  }
}

// Child.java
import fruit.Pear;
public class Child extends Parent {

  public void doSomething(Pear p) {  // Noncompliant; this is not an override
    // ...
  }


  public void doSomethingElse() {  // Noncompliant; parent method is static
    //...
  }
}
```

### Compliant Solution
```java
// Parent.java
import computer.Pear;
public class Parent {

  public void doSomething(Pear p) {
    //,,,
  }

  public static void doSomethingElse() {
    //...
  }
}

// Child.java
import computer.Pear;  // import corrected
public class Child extends Parent {

  public void doSomething(Pear p) {  // true override (see import)
    //,,,
  }

  public static void doSomethingElse() {
    //...
  }
}
```
### 一句话说明
- 应该重写以父类方法命名的子类方法





## 46、Classes extending java.lang.Thread should override the "run" method

- 主要

According to the Java API documentation:
> There are two ways to create a new thread of execution. One is to declare a class to be a subclass of Thread. This subclass should override the run method of class Thread. An instance of the subclass can then be allocated and started...
有两种方法可以创建新的执行线程。一种方法是将一个类声明为Thread的子类。这个子类应该覆盖类Thread的run方法。然后可以分配并启动子类的实例…

> The other way to create a thread is to declare a class that implements the Runnable interface. That class then implements the run method. An instance of the class can then be allocated, passed as an argument when creating Thread, and started.
创建线程的另一种方法是声明一个实现Runnable接口的类。然后该类实现run方法。然后可以分配该类的实例，在创建Thread时作为参数传递，并启动它。

By definition, extending the Thread class without overriding the `run` method doesn't make sense, and implies that the contract of the `Thread` class is not well understood.
根据定义，在不覆盖run方法的情况下扩展Thread类是没有意义的，这意味着没有很好地理解Thread类的契约。

### Noncompliant Code Example
```java
public class MyRunner extends Thread { // Noncompliant; run method not overridden

  public void doSometing() {...}
}
```

### Exceptions
If `run()` is not overridden in a class extending `Thread`, it means that starting the thread will actually call `Thread.run()`. However, `Thread.run()` does nothing if it has not been fed with a target `Runnable`. The rule consequently ignore classes extending `Thread` if they are calling, in their constructors, the `super(...)` constructor with a proper `Runnable` target.
如果在类扩展线程中没有覆盖run()，这意味着启动线程将实际调用Thread.run()。但是，如果没有提供目标Runnable, Thread.run()将不执行任何操作。因此，如果类在其构造函数中调用具有适当Runnable目标的super(…)构造函数，则规则将忽略扩展线程的类。
```java
class MyThread extends Thread { // Compliant - calling super constructor with a Runnable
  MyThread(Runnable target) {
    super(target); // calling super constructor with a Runnable, which will be used for when Thread.run() is executed
    // ...
  }
}
```
### 一句话说明
- 类扩展`java.lang.Thread`线程应该覆盖`run`方法






## 47、Classes should not be compared by name

- 主要

>There is no requirement that class names be unique, only that they be unique within a package. Therefore trying to determine an object's type based on its class name is an exercise fraught with danger. One of those dangers is that a malicious user will send objects of the same name as the trusted class and thereby gain trusted access.
没有要求类名是惟一的，只要求它们在包中是惟一的。因此，试图根据对象的类名确定对象的类型是一项充满危险的练习。其中一个危险是恶意用户将发送与可信类同名的对象，从而获得可信访问权。

------------

>Instead, the `instanceof` operator or the `Class.isAssignableFrom()` method should be used to check the object's underlying type.
相反，应该使用instanceof操作符或Class.isAssignableFrom()方法来检查对象的底层类型。

### Noncompliant Code Example
```java
package computer;
class Pear extends Laptop { ... }

package food;
class Pear extends Fruit { ... }

class Store {

  public boolean hasSellByDate(Object item) {
    if ("Pear".equals(item.getClass().getSimpleName())) {  // Noncompliant
      return true;  // Results in throwing away week-old computers
    }
    return false;
  }

  public boolean isList(Class<T> valueClass) {
    if (List.class.getName().equals(valueClass.getName())) {  // Noncompliant
      return true;
    }
    return false;
  }
}
```
### Compliant Solution
```java
class Store {

  public boolean hasSellByDate(Object item) {
    if (item instanceof food.Pear) {
      return true;
    }
    return false;
  }

  public boolean isList(Class<T> valueClass) {
    if (valueClass.isAssignableFrom(List.class)) {
      return true;
    }
    return false;
  }
}
```
### See
-   [MITRE, CWE-486](http://cwe.mitre.org/data/definitions/486.html) - Comparison of Classes by Name
-   [CERT, OBJ09-J.](https://www.securecoding.cert.org/confluence/x/LAFlAQ) - Compare classes and not class names
### 一句话说明
- 不要用类名称比较类是否相同，而用i`nstanceof`或者`Class.isAssignableFrom()`进行底动类型比较

### 摘自CERT
### 例子1
```java
// 错误的写法
// Determine whether object auth has required/expected class object
 if (auth.getClass().getName().equals(
      "com.application.auth.DefaultAuthenticationHandler")) {
   // ...
}
```
```java
//正确的写法
// Determine whether object auth has required/expected class name
 if (auth.getClass() == com.application.auth.DefaultAuthenticationHandler.class) {
   // ...
}
```

### 例子2
```java
// 错误的写法
// Determine whether objects x and y have the same class name
if (x.getClass().getName().equals(y.getClass().getName())) {
  // Objects have the same class
}
```
```java
//正确的写法
// Determine whether objects x and y have the same class
if (x.getClass() == y.getClass()) {
  // Objects have the same class
}
```





### 48、Classes that don't define "hashCode()" should not be used in hashes

- 主要

>Because `Object` implements `hashCode`, any Java class can be put into a hash structure. However, classes that define `equals(Object)` but not `hashCode()` aren't truly hash-able because instances that are equivalent according to the `equals` method can return different hashes.
因为Object实现了hashCode，所以任何Java类都可以放入散列结构中。但是，定义equals(Object)而不是hashCode()的类实际上是不可哈希的，因为根据equals方法等效的实例可以返回不同的哈希值。

### Noncompliant Code Example
```java
public class Student {  // no hashCode() method; not hash-able
  // ...

  public boolean equals(Object o) {
    // ...
  }
}

public class School {
  private Map<Student, Integer> studentBody = // okay so far
          new HashTable<Student, Integer>(); // Noncompliant

  // ...
```
### Compliant Solution
```java
public class Student {  // has hashCode() method; hash-able
  // ...

  public boolean equals(Object o) {
    // ...
  }
  public int hashCode() {
    // ...
  }
}

public class School {
  private Map<Student, Integer> studentBody = new HashTable<Student, Integer>();

  // ...
```
### 一句话说明
- 没有定义HashCode()方法的类不能在hash上使用





## 49、Collections should not be passed as arguments to their own methods

- 主要

>Passing a collection as an argument to the collection's own method is either an error - some other argument was intended - or simply nonsensical code.
将集合作为参数传递给集合自己的方法要么是一个错误(有意使用其他参数)，要么仅仅是无意义的代码。

>Further, because some methods require that the argument remain unmodified during the execution, passing a collection to itself can result in undefined behavior.
此外，由于一些方法要求参数在执行期间保持未修改状态，因此将集合传递给自己可能会导致未定义的行为。

### Noncompliant Code Example
```java
List <Object> objs = new ArrayList<Object>();
objs.add("Hello");

objs.add(objs); // Noncompliant; StackOverflowException if objs.hashCode() called
objs.addAll(objs); // Noncompliant; behavior undefined
objs.containsAll(objs); // Noncompliant; always true
objs.removeAll(objs); // Noncompliant; confusing. Use clear() instead
objs.retainAll(objs); // Noncompliant; NOOP
```
### 一句话说明
- 集合不应该作为参数传递给它们自己的方法






## 50、Conditionally executed blocks should be reachable

- 主要

>Conditional expressions which are always `true` or `false` can lead to dead code. Such code is always buggy and should never be used in production.
总是为真或为假的条件表达式可能导致死代码。这样的代码总是有bug，不应该在生产中使用。

### Noncompliant Code Example
```java
a = false;
if (a) { // Noncompliant
  doSomething(); // never executed
}

if (!a || b) { // Noncompliant; "!a" is always "true", "b" is never evaluated
  doSomething();
} else {
  doSomethingElse(); // never executed
}
```
### Exceptions
This rule will not raise an issue in either of these cases:
- When the condition is a single `final boolean`
```java
final boolean debug = false;
//...
if (debug) {
  // Print something
}
```
- When the condition is literally `true` or `false`.
```java
if (true) {
  // do something
}
```
In these cases it is obvious the code is as intended.
### See
*   MISRA C:2004, 13.7 - Boolean operations whose results are invariant shall not be permitted.
*   MISRA C:2012, 14.3 - Controlling expressions shall not be invariant
*   [MITRE, CWE-570](http://cwe.mitre.org/data/definitions/570.html) - Expression is Always False
*   [MITRE, CWE-571](http://cwe.mitre.org/data/definitions/571) - Expression is Always True
*   [CERT, MSC12-C.](https://www.securecoding.cert.org/confluence/x/NYA5) - Detect and remove code that has no effect or is never executed

### 一句话说明
- 应该可以访问有条件执行的块






## 51、Constructor injection should be used instead of field injection

- 主要

>Field injection seems like a tidy way to get your classes what they need to do their jobs, but it's really a `NullPointerException` waiting to happen unless all your class constructors are `private`. That's because any class instances that are constructed by callers, rather than instantiated by a Dependency Injection framework compliant with the JSR-330 (Spring, Guice, ...), won't have the ability to perform the field injection.
字段注入似乎是一种让类完成它们的工作所需的整洁方法，但它实际上是一个等待发生的NullPointerException，除非所有的类构造函数都是私有的。这是因为任何由调用者构造的类实例，而不是由符合JSR-330 (Spring, Guice，…)的依赖注入框架实例化的类实例，都不能执行字段注入。

------------

>Instead `@Inject` should be moved to the constructor and the fields required as constructor parameters.
相反，@Inject应该被移动到构造函数和作为构造函数参数所需的字段中。

------------

>This rule raises an issue when classes with non-`private` constructors (including the default constructor) use field injection.
当使用非私有构造函数(包括默认构造函数)的类使用字段注入时，此规则会引发一个问题。

### Noncompliant Code Example
```java
class MyComponent {  // Anyone can call the default constructor

  @Inject MyCollaborator collaborator;  // Noncompliant

  public void myBusinessMethod() {
    collaborator.doSomething();  // this will fail in classes new-ed by a caller
  }
}
```
### Compliant Solution
```java
class MyComponent {

  private final MyCollaborator collaborator;

  @Inject
  public MyComponent(MyCollaborator collaborator) {
    Assert.notNull(collaborator, "MyCollaborator must not be null!");
    this.collaborator = collaborator;
  }

  public void myBusinessMethod() {
    collaborator.doSomething();
  }
}
```
### 一句话说明
- 构造器注入应该替代属性注入(非Spring framework)因为任何非Spring framework实例化而是通过构造器实例化的实例不能注入属性,这样公有的构造器实化化后可能产生NullPointerException，除非所有的构造器都是私有的







## 52、Consumed Stream pipelines should not be reused

- 主要

>Stream operations are divided into intermediate and terminal operations, and are combined to form stream pipelines. After the terminal operation is performed, the stream pipeline is considered consumed, and cannot be used again. Such a reuse will yield unexpected results.
流操作分为中间操作和终端操作，并结合起来形成流管道。执行终端操作后，将认为流管道已被消耗，不能再次使用。这样的重用将产生意想不到的结果。

### Noncompliant Code Example
```java
Stream<Widget> pipeline = widgets.stream().filter(b -> b.getColor() == RED);
int sum1 = pipeline.sum();
int sum2 = pipeline.mapToInt(b -> b.getWeight()).sum(); // Noncompliant
```
### See
- [Stream Operations](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps)

### 一句话说明
- 流不允许被重用

### 例子





## 53、Custom serialization method signatures should meet requirements

- 主要

>Writers of `Serializable` classes can choose to let Java's automatic mechanisms handle serialization and deserialization, or they can choose to handle it themselves by implementing specific methods. However, if the signatures of those methods are not exactly what is expected, they will be ignored and the default serialization mechanisms will kick back in.
可序列化类的编写器可以选择让Java的自动机制处理序列化和反序列化，也可以选择通过实现特定的方法自行处理。但是，如果这些方法的签名不完全符合预期，那么它们将被忽略，默认的序列化机制将重新启动。

### Noncompliant Code Example
```java
public class Watermelon implements Serializable {
  // ...
  void writeObject(java.io.ObjectOutputStream out)// Noncompliant; not private
        throws IOException
  {...}

  private void readObject(java.io.ObjectInputStream in)
  {...}

  public void readObjectNoData()  // Noncompliant; not private
  {...}

  static Object readResolve() throws ObjectStreamException  // Noncompliant; this method may have any access modifier, may not be static

  Watermelon writeReplace() throws ObjectStreamException // Noncompliant; this method may have any access modifier, but must return Object
  {...}
}
```
### Compliant Solution
```java
public class Watermelon implements Serializable {
  // ...
  private void writeObject(java.io.ObjectOutputStream out)
        throws IOException
  {...}

  private void readObject(java.io.ObjectInputStream in)
        throws IOException, ClassNotFoundException
  {...}

  private void readObjectNoData()
        throws ObjectStreamException
  {...}

  protected Object readResolve() throws ObjectStreamException
  {...}

  private Object writeReplace() throws ObjectStreamException
  {...}
```
### See

- [CERT, SER01-J.](https://www.securecoding.cert.org/confluence/x/4gAMAg) - Do not deviate from the proper signatures of serialization methods

#### 一句话说明
- 自定义类序列化方法签名应该合法






## 54、Dependencies should not have "system" scope

- 主要

>`system` dependencies are sought at a specific, specified path. This drastically reduces portability because if you deploy your artifact in an environment that's not configured just like yours is, your code won't work.

### Noncompliant Code Example
```java
<dependency>
  <groupId>javax.sql</groupId>
  <artifactId>jdbc-stdext</artifactId>
  <version>2.0</version>
  <scope>system</scope>  <!-- Noncompliant -->
  <systemPath>/usr/bin/lib/rt.jar</systemPath>  <!-- remove this -->
</dependency>
```

### 一句话说明
- 不允许有`system`范围的依赖





## 55、Dissimilar primitive wrappers should not be used with the ternary operator without explicit casting

- 主要

>If wrapped primitive values (e.g. `Integers` and `Floats`) are used in a ternary operator (e.g. `a?b:c`), both values will be unboxed and coerced to a common type, potentially leading to unexpected results. To avoid this, add an explicit cast to a compatible type.
如果在三元运算符(例如a?b:c)中使用包装的原始值(例如整数和浮点数)，那么这两个值都将被取消装箱并强制转换为公共类型，这可能会导致意想不到的结果。要避免这种情况，请向兼容类型添加显式强制转换。

### Noncompliant Code Example
```java
Integer i = 123456789;
Float f = 1.0f;
Number n = condition ? i : f;  // Noncompliant; i is coerced to float. n = 1.23456792E8
```
### Compliant Solution
```java
Integer i = 123456789;
Float f = 1.0f;
Number n = condition ? (Number) i : f;  // n = 123456789
```
### 一句话说明
- 在没有显式强制转换的情况下，不应该将不同的基本包装器与三元运算符一起使用

### 例子
```java
    @Test
    public void test54() {

        Integer i = 123456789;
        Float f = 1.0f;
        Number n = true ? i : f;  // Noncompliant; i is coerced to float. n = 1.23456792E8

        System.out.println(n);// 输出1.23456792E8， 即1.23456792*10的8次方
        System.out.println(n.longValue());//输出123456792
    }
```




## 56、Double Brace Initialization should not be used

- 主要

>Because Double Brace Initialization (DBI) creates an anonymous class with a reference to the instance of the owning object, its use can lead to memory leaks if the anonymous inner class is returned and held by other objects. Even when there's no leak, DBI is so obscure that it's bound to confuse most maintainers.
因为双花括号初始化(DBI)创建了一个匿名类，该类引用拥有对象的实例，如果匿名内部类被其他对象返回并持有，那么使用它可能导致内存泄漏。即使在没有泄漏的情况下，DBI也是非常模糊的，这一定会使大多数维护人员感到困惑。

>For collections, use `Arrays.asList` instead, or explicitly add each item directly to the collection.
对于集合，使用数组。或者显式地将每个项直接添加到集合中。

### Noncompliant Code Example
```java
Map source = new HashMap(){{ // Noncompliant
    put("firstName", "John");
    put("lastName", "Smith");
}};
```
### Compliant Solution
```java
Map source = new HashMap();
// ...
source.put("firstName", "John");
source.put("lastName", "Smith");
// ...
```

### 一句话说明
- 不要使用双花括号初始化

### 例子





## 57、Double-checked locking should not be used

- 阻断

>Double-checked locking is the practice of checking a lazy-initialized object's state both before and after a `synchronized` block is entered to determine whether or not to initialize the object.
双重检查锁定是在输入同步块之前和之后检查延迟初始化对象的状态，以确定是否初始化该对象。

>It does not work reliably in a platform-independent manner without additional synchronization for mutable instances of anything other than `float` or `int`. Using double-checked locking for the lazy initialization of any other type of primitive or mutable object risks a second thread using an uninitialized or partially initialized member while the first thread is still creating it, and crashing the program.
它不可靠地工作在一个独立于平台的方式没有额外的同步可变的实例不是浮动或int。使用延迟初始化的双重检查锁定任何其他类型的原始或可变对象风险第二个线程使用未初始化或部分初始化成员第一个线程仍然是创建它时,程序崩溃。

>There are multiple ways to fix this. The simplest one is to simply not use double checked locking at all, and synchronize the whole method instead. With early versions of the JVM, synchronizing the whole method was generally advised against for performance reasons. But `synchronized` performance has improved a lot in newer JVMs, so this is now a preferred solution. If you prefer to avoid using `synchronized` altogether, you can use an inner `static class` to hold the reference instead. Inner static classes are guaranteed to load lazily.
有多种方法可以解决这个问题。最简单的方法就是根本不使用双重检查锁定，而是同步整个方法。对于早期版本的JVM，出于性能原因，通常不建议同步整个方法。但是在较新的jvm中，同步性能已经有了很大的改进，所以现在这是一个首选的解决方案。如果希望完全避免使用synchronized，可以使用内部静态类来保存引用。内部静态类保证延迟加载。

### Noncompliant Code Example
```java
@NotThreadSafe
public class DoubleCheckedLocking {
    private static Resource resource;

    public static Resource getInstance() {
        if (resource == null) {
            synchronized (DoubleCheckedLocking.class) {
                if (resource == null)
                    resource = new Resource();
            }
        }
        return resource;
    }

    static class Resource {

    }
}
```
### Compliant Solution
```java
@ThreadSafe
public class SafeLazyInitialization {
    private static Resource resource;

    public synchronized static Resource getInstance() {
        if (resource == null)
            resource = new Resource();
        return resource;
    }

    static class Resource {
    }
}
```
With inner static holder:
```java
@ThreadSafe
public class ResourceFactory {
    private static class ResourceHolder {
        public static Resource resource = new Resource(); // This will be lazily initialised
    }

    public static Resource getResource() {
        return ResourceFactory.ResourceHolder.resource;
    }

    static class Resource {
    }
}
```
Using "volatile":
```java
class ResourceFactory {
  private volatile Resource resource;

  public Resource getResource() {
    Resource localResource = resource;
    if (localResource == null) {
      synchronized (this) {
        localResource = resource;
        if (localResource == null) {
          resource = localResource = new Resource();
        }
      }
    }
    return localResource;
  }

  static class Resource {
  }
}
```
### See

-   [The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)
-   [CERT, LCK10-J.](https://www.securecoding.cert.org/confluence/x/IgAZAg) - Use a correct form of the double-checked locking idiom
-   [MITRE, CWE-609](http://cwe.mitre.org/data/definitions/609.html) - Double-checked locking
-   [JLS 12.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.4) - Initialization of Classes and Interfaces
-   Wikipedia: [Double-checked locking](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)

### 一句话说明
- 不要使用双检锁来初始化实例

### 详细说明
- [并发：双重检查锁定（Double-Checked Locking）与延迟初始化（Lazy Initialization）](https://blog.csdn.net/en_joker/article/details/84761611 "并发：双重检查锁定（Double-Checked Locking）与延迟初始化（Lazy Initialization）")







## 58、Exception should not be created without being thrown

- 主要

>Creating a new `Throwable` without actually throwing it is useless and is probably due to a mistake.
创建一个新的可扔的东西而不实际地扔它是没有用的，可能是由于一个错误。

### Noncompliant Code Example
```java
if (x < 0)
  new IllegalArgumentException("x must be nonnegative");
```
### Compliant Solution
```java
if (x < 0)
  throw new IllegalArgumentException("x must be nonnegative");
```
### 一句话说明
- 抛出异常必须写上throw关键字






## 59、Expressions used in "assert" should not produce side effects

- 主要

>Since `assert` statements aren't executed by default (they must be enabled with JVM flags) developers should never rely on their execution the evaluation of any logic required for correct program function.

### Noncompliant Code Example
```java
assert myList.remove(myList.get(0));  // Noncompliant
```
### Compliant Solution
```java
boolean removed = myList.remove(myList.get(0));
assert removed;
```
### See
-   [CERT, EXP06-J.](https://www.securecoding.cert.org/confluence/x/vwG7AQ) - Expressions used in assertions must not produce side effects

### 一句话说明
- “断言”中使用的表达式不应该产生副作用
- assert表达式不要产生负影响，不要改变数据状态





## 60、Files opened in append mode should not be used with ObjectOutputStream

- 阻断

>`ObjectOutputStream`s are used with serialization, and the first thing an `ObjectOutputStream` writes is the serialization stream header. This header should appear once per file, at the beginning. Pass a file opened in append mode into an `ObjectOutputStream` constructor and the serialization stream header will be added to the end of the file before your object is then also appended.
`ObjectOutputStream`用于序列化，`ObjectOutputStream`写的第一件事是序列化流头。这个头文件应该在每个文件的开头出现一次。将以追加模式打开的文件传递到`ObjectOutputStream`构造函数中，序列化流头将被添加到文件的末尾，然后才追加对象。

>When you're trying to read your object(s) back from the file, only the first one will be read successfully, and a `StreamCorruptedException` will be thrown after that.
当您试图从文件中读取对象时，只有第一个对象将被成功读取，然后将抛出一个stream腐蚀异常。

### Noncompliant Code Example
```java
FileOutputStream fos = new FileOutputStream (fileName , true);  // fos opened in append mode
ObjectOutputStream out = new ObjectOutputStream(fos);  // Noncompliant
```
### Compliant Solution
```java
FileOutputStream fos = new FileOutputStream (fileName);
ObjectOutputStream out = new ObjectOutputStream(fos);
```
### 一句话说明
- 在append模式下打开的文件不应该与ObjectOutputStream一起使用






## 61、Getters and setters should access the expected fields

- 主要

>Getters and setters provide a way to enforce encapsulation by providing `public` methods that give controlled access to `private` fields. However in classes with multiple fields it is not unusual that cut and paste is used to quickly create the needed getters and setters, which can result in the wrong field being accessed by a getter or setter.
getter和setter提供了一种方法，通过提供提供对私有字段的受控访问的公共方法来强制封装。然而，在具有多个字段的类中，使用剪切和粘贴来快速创建所需的getter和setter并不少见，这可能导致getter或setter访问了错误的字段。

This rule raises an issue in any of these cases:

- A setter does not update the field with the corresponding name.
- A getter does not access the field with the corresponding name.

### Noncompliant Code Example
```java
class A {
  private int x;
  private int y;

  public void setX(int val) { // Noncompliant: field 'x' is not updated
    this.y = val;
  }

  public int getY() { // Noncompliant: field 'y' is not used in the return value
    return this.x;
  }
}
```
### Compliant Solution
```java
class A {
  private int x;
  private int y;

  public void setX(int val) {
    this.x = val;
  }

  public int getY() {
    return this.y;
  }
}
```
### 一句话说明
- getter和setter应该访问预期的字段




## 62、Getters and setters should be synchronized in pairs

- 主要

>When one part of a getter/setter pair is `synchronized` the other part should be too. Failure to synchronize both sides of a pair may result in inconsistent behavior at runtime as callers access an inconsistent method state.
当getter/setter对的一部分同步时，另一部分也应该同步。当调用方访问不一致的方法状态时，未能同步对的两边可能会在运行时导致不一致的行为。

>This rule raises an issue when either the method or the contents of one method in a getter/setter pair are synchrnoized but the other is not.
当getter/setter对中的方法或其中一个方法的内容同步而另一个方法没有同步时，该规则会引发问题。

>A common misconception is that shared references to immutable objects are immediately visible across multiple threads as soon as they are updated. For example, a developer can mistakenly believe that a class containing fields that refer only to immutable objects is itself immutable and consequently thread-safe.
一个常见的误解是，对不可变对象的共享引用一旦更新，就会立即在多个线程之间可见。例如，开发人员可能错误地认为，包含仅引用不可变对象的字段的类本身是不可变的，因此是线程安全的。

### Noncompliant Code Example
```java
public class Person {
  String name;
  int age;

  public synchronized void setName(String name) {
    this.name = name;
  }

  public String getName() {  // Noncompliant
    return this.name;
  }

  public void setAge(int age) {  // Noncompliant
    this.age = age;
  }

  public int getAge() {
    synchronized (this) {
      return this.age;
    }
  }
}
```
### Compliant Solution
```java
public class Person {
  String name;
  int age;

  public synchronized void setName(String name) {
    this.name = name;
  }

  public synchronized String getName() {
    return this.name;
  }

  public void setAge(int age) {
    synchronized (this) {
      this.age = age;
   }
  }

  public int getAge() {
    synchronized (this) {
      return this.age;
    }
  }
}
```

### Noncompliant Code Example
```java
// Immutable Helper
public final class Helper {
  private final int n;
 
  public Helper(int n) {
    this.n = n;
  }
  // ...
}


final class Foo {
  private Helper helper;
 
  public Helper getHelper() {
    return helper;
  }
 
  public void setHelper(int num) {
    helper = new Helper(num);
  }
}
```
### Compliant Solution
```java
//解决方案一：synchronized方式
final class Foo {
  private Helper helper;
 
  public synchronized Helper getHelper() {
    return helper;
  }
 
  public synchronized void setHelper(int num) {
    helper = new Helper(num);
  }
}

//解决方案二：volatile方式
final class Foo {
  private volatile Helper helper;
 
  public Helper getHelper() {
    return helper;
  }
 
  public void setHelper(int num) {
    helper = new Helper(num);
  }
}

//解决方案三：AtomicReference方式
final class Foo {
  private final AtomicReference<Helper> helperRef =
      new AtomicReference<Helper>();
 
  public Helper getHelper() {
    return helperRef.get();
  }
 
  public void setHelper(int num) {
    helperRef.set(new Helper(num));
  }
}
```

### See

-  [CERT, VNA01-J.](https://www.securecoding.cert.org/confluence/x/I4BoAg) - Ensure visibility of shared references to immutable objects

### 一句话说明
- getter和setter应该成对同步





## 63、Hibernate should not update database schemas

- 严重

>The use of any value but `"validate"` for `hibernate.hbm2ddl.auto` may cause the database schema used by your application to be changed, dropped, or cleaned of all data. In short, the use of this property is risky, and should only be used in production with the `"validate"` option, if it is used at all.
使用hibernate.hbm2ddl中除了“validate”之外的任何值。auto可能会导致应用程序使用的数据库模式被更改、删除或清除所有数据。简而言之，使用此属性是有风险的，并且只有在使用“validate”选项时才应该在生产中使用。

### Noncompliant Code Example
```java
<session-factory>
  <property name="hibernate.hbm2ddl.auto">update</property>  <!-- Noncompliant -->
</session-factory>
```
### Compliant Solution
```java
<session-factory>
  <property name="hibernate.hbm2ddl.auto">validate</property>  <!-- Compliant -->
</session-factory>
```
or
```java
<session-factory>
  <!-- Property deleted -->
</session-factory>
```
### 一句话说明
- Hibernate不应该设置成更新数据库模式





## 64、Identical expressions should not be used on both sides of a binary operator

- 主要

>Using the same value on either side of a binary operator is almost always a mistake. In the case of logical operators, it is either a copy/paste error and therefore a bug, or it is simply wasted code, and should be simplified. In the case of bitwise operators and most binary mathematical operators, having the same value on both sides of an operator yields predictable results, and should be simplified.
在二进制操作符的两边使用相同的值几乎总是一个错误。对于逻辑操作符，它要么是复制/粘贴错误，因此是一个bug，要么只是浪费代码，应该加以简化。在位操作符和大多数二进制数学操作符的情况下，在操作符的两边都有相同的值会产生可预测的结果，应该进行简化。

>Code that has no effect or is never executed (that is, dead or unreachable code) is typically the result of a coding error and can cause [unexpected behavior](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-unexpectedbehavior). Such code is usually optimized out of a program during compilation. However, to improve readability and ensure that logic errors are resolved, it should be identified, understood, and eliminated.
没有效果或从未执行的代码(即死代码或不可访问代码)通常是编码错误的结果，并可能导致意外行为。这样的代码通常在编译过程中从程序中优化出来。然而，为了提高可读性并确保逻辑错误得到解决，应该识别、理解和消除逻辑错误。

### Noncompliant Code Example

```java
if ( a == a ) { // always true
  doZ();
}
if ( a != a ) { // always false
  doY();
}
if ( a == b && a == b ) { // if the first one is true, the second one is too
  doX();
}
if ( a == b || a == b ) { // if the first one is true, the second one is too
  doW();
}

int j = 5 / 5; //always 1
int k = 5 - 5; //always 0

c.equals(c); //always true
```
### Exceptions
-   This rule ignores `*`, `+`, and `=`.
-   The specific case of testing a floating point value against itself is a valid test for `NaN` and is therefore ignored.
-   Similarly, left-shifting 1 onto 1 is common in the construction of bit masks, and is ignored.

```java
float f;
if(f != f) { //test for NaN value
  System.out.println("f is NaN");
}

int i = 1 << 1; // Compliant
int j = a << a; // Noncompliant
```
### See

-   [CERT, MSC12-C.](https://www.securecoding.cert.org/confluence/x/NYA5) - Detect and remove code that has no effect or is never executed
-   [S1656](http://10.50.8.99:9000/coding_rules#rule_key=squid%3AS1656) - Implements a check on `=`.

### 一句话说明
- 不应该在二进制运算符的两边使用相同的表达式





## 65、Inappropriate "Collection" calls should not be made

- 主要

>The `java.util.Collection` API has methods that accept `Object` parameters such as `Collection.remove(Object o)`, and `Collection.contains(Object o)`. When the actual type of the object provided to these methods is not consistent with the type declared on the `Collection` instantiation, those methods will always return `false` or `null`. This is most likely unintended and hides a design problem.
`java.util.Collection`API具有接受对象参数(如Collection)的方法。删除(对象o)和集合。当提供给这些方法的对象的实际类型与集合实例化时声明的类型不一致时，这些方法总是返回false或null。这很可能是无意的，并隐藏了一个设计问题。

This rule raises an issue when the type of the argument of the following APIs is unrelated to the type used for the `Collection` declaration:

- `Collection.remove(Object o)`

- `Collection.contains(Object o)`

- `List.indexOf(Object o)`

- `List.lastIndexOf(Object o)`

- `Map.containsKey(Object key)`

- `Map.containsValue(Object value)`

- `Map.get(Object key)`

- `Map.getOrDefault(Object key, V defaultValue)`

- `Map.remove(Object key)`

- `Map.remove(Object key, Object value)`


>The interfaces of the Java Collections Framework [[JCF 2014](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-JCF14)] use generically typed, parameterized methods, such as `add(E e)` and `put(K key, V value)`, to insert objects into the collection or map, but they have other methods, such as `contains()`, `remove()`, and `get()`, that accept an argument of type `Object` rather than a parameterized type. Consequently, these methods accept an object of _any_ type. The collections framework interfaces were designed in this manner to maximize backwards compatibility, but this design can also lead to coding errors. Programmers must ensure that arguments passed to methods such as `Map<K,V>` `get()`, `Collection<E>`  `contains()`, and `remove()` have the same type as the parameterized type of the corresponding class instance.
接口的Java集合框架[JCF 2014]使用泛型参数化方法,如`add(E e)`和`put(K key, V value)`,将对象插入到集合或地图,但是他们有其他的方法,如`contains()`、 `remove()`,`get()`,接受一个参数类型的对象,而不是一个参数化的类型。因此，这些方法接受任何类型的对象。集合框架接口的设计方式是为了最大限度地向后兼容，但是这种设计也可能导致编码错误。程序员必须确保传递给`Map<K,V>`、`get()`、 `Collection<E>`  `contains()`和`remove()`等方法的参数具有与对应类实例的参数化类型相同的类型。

### Noncompliant Code Example
```java
public class S2175 {

  public static void main(String[] args) {
    String foo = "42";
    Map<Integer, Object> map = new HashMap<>();
    map.remove(foo); // Noncompliant; will return 'null' for sure because 'map' is handling only Integer keys

    // ...

    List<String> list = new ArrayList<String>();
    Integer integer = Integer.valueOf(1);
    if (list.contains(integer)) { // Noncompliant; always false.
      list.remove(integer); // Noncompliant; list.add(integer) doesn't compile, so this will always return 'false'
    }
  }

}
```
### See
-   [CERT, EXP04-J.](https://www.securecoding.cert.org/confluence/x/QwFlAQ) - Do not pass arguments to certain Java Collections Framework methods that are a different type than the collection parameter type

### 一句话说明
- 不要调用集合方法入参和集合定义类型不一致的方法




## 66、Inappropriate regular expressions should not be used

- 主要

>Regular expressions are powerful but tricky, and even those long used to using them can make mistakes.
正则表达式功能强大，但很棘手，即使是那些长期使用正则表达式的人也可能会犯错误。

>The following should not be used as regular expressions:
下列内容不应用作正则表达式:
-  `.` - matches any single character. Used in `replaceAll`, it matches _everything_
`.`- 匹配任何单个字符。在`replaceAll`中使用，它匹配所有内容
-  `|` - normally used as an option delimiter. Used stand-alone, it matches the space between characters
`|` - 通常用作选项分隔符。使用独立，它匹配字符之间的空间
-  `File.separator` - matches the platform-specific file path delimiter. On Windows, this will be taken as an escape character
`File.separator` - 匹配特定于平台的文件路径分隔符。在Windows上，这将作为转义字符

### Noncompliant Code Example
```java
String str = "/File|Name.txt";

String clean = str.replaceAll(".",""); // Noncompliant; probably meant to remove only dot chars, but returns an empty string
String clean2 = str.replaceAll("|","_"); // Noncompliant; yields _/_F_i_l_e_|_N_a_m_e_._t_x_t_
String clean3 = str.replaceAll(File.separator,""); // Noncompliant; exception on Windows

String clean4 = str.replaceFirst(".",""); // Noncompliant;
String clean5 = str.replaceFirst("|","_"); // Noncompliant;
String clean6 = str.replaceFirst(File.separator,""); // Noncompliant;
```
### 一句话说明
- 不应该使用不正确的正则表达式




## 67、InputSteam.read() implementation should not return a signed byte

- 主要

>According to the Java documentation, any implementation of the `InputSteam.read()` method is supposed to read the next byte of data from the input stream. The value byte must be an `int` in the range 0 to 255\. If no byte is available because the end of the stream has been reached, the value -1 is returned.
根据Java文档，`InputSteam.read()`方法的任何实现都应该从输入流中读取下一个字节的数据。值字节必须是0到255范围内的整数。如果由于到达流的末尾而没有可用字节，则返回值-1。

>But in Java, the `byte` primitive data type is an 8-bit signed two's complement integer. It has a minimum value of -128 and a maximum value of 127\. So by contract, the implementation of an `InputSteam.read()` method should never directly return a `byte` primitive data type. A conversion into an unsigned byte must be done before by applying a bitmask.
但是在Java中，字节基元数据类型是一个8位带符号2的补数。它的最小值是-128，最大值是127。因此，根据约定，`InputSteam.read()`方法的实现不应该直接返回字节基元数据类型。在应用位掩码之前，必须先将其转换为无符号字节。

### Noncompliant Code Example
```java
@Override
public int read() throws IOException {
  if (pos == buffer.length()) {
    return -1;
  }
  return buffer.getByte(pos++); // Noncompliant, a signed byte value is returned
}
```
### Compliant Solution
```java
@Override
public int read() throws IOException {
  if (pos == buffer.length()) {
    return -1;
  }
  return buffer.getByte(pos++) & 0xFF; // The 0xFF bitmask is applied
}
```
### 一句话说明
- read()实现不应该返回带符号的字节

### 原理说明
- [Java中为何与0xff进行与运算](https://blog.csdn.net/birdflyto206/article/details/50234685 "Java中为何与0xff进行与运算")





## 68、Intermediate Stream methods should not be left unused

- 主要

>There are two types of stream operations: intermediate operations, which return another stream, and terminal operations, which return something other than a stream. Intermediate operations are lazy, meaning they aren't actually executed until and unless a terminal stream operation is performed on their results. Consequently if the result of an intermediate stream operation is not fed to a terminal operation, it serves no purpose, which is almost certainly an error.
流操作有两种类型:中间操作(返回另一个流)和终端操作(返回流以外的内容)。中间操作是惰性的，这意味着它们实际上不会执行，除非对它们的结果执行终端流操作。因此，如果中间流操作的结果没有提供给终端操作，那么它就没有任何作用，这几乎肯定是一个错误。

### Noncompliant Code Example
```java
widgets.stream().filter(b -> b.getColor() == RED); // Noncompliant
```
### Compliant Solution
```java
int sum = widgets.stream()
                      .filter(b -> b.getColor() == RED)
                      .mapToInt(b -> b.getWeight())
                      .sum();
Stream<Widget> pipeline = widgets.stream()
                                 .filter(b -> b.getColor() == GREEN)
                                 .mapToInt(b -> b.getWeight());
sum = pipeline.sum();
```
### See

- [Stream Operations](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps)

### 一句话说明
- 使用流必须要用到终端操作方法，而不能仅仅使用中间操作就完事了





## 69、Ints and longs should not be shifted by zero or more than their number of bits-1

- 次要

>Since an `int` is a 32-bit variable, shifting by more than +/-31 is confusing at best and an error at worst. When the runtime shifts 32-bit integers, it uses the lowest 5 bits of the shift count operand. In other words, shifting an `int` by 32 is the same as shifting it by 0, and shifting it by 33 is the same as shifting it by 1.
因为int是一个32位的变量，所以移位超过+/-31最容易让人混淆，最坏也会导致错误。当运行时移位32位整数时，它使用移位计数操作数的最低5位。换句话说，移动一个int数32和移动它0是一样的，移动它33和移动它1是一样的。

>Similarly, when shifting 64-bit integers, the runtime uses the lowest 6 bits of the shift count operand and shifting `long` by 64 is the same as shifting it by 0, and shifting it by 65 is the same as shifting it by 1.
类似地，当移动64位整数时，运行时使用移位计数操作数的最低6位，将long移动64位与将其移动0位相同，将其移动65位与将其移动1位相同。

### Noncompliant Code Example
```java
public int shift(int a) {
  int x = a >> 32; // Noncompliant
  return a << 48;  // Noncompliant
}
```
### Compliant Solution
```java
public int shift(int a) {
  int x = a >> 31;
  return a << 16;
}
```
### Exceptions
This rule doesn't raise an issue when the shift by zero is obviously for cosmetic reasons:
当0的偏移明显是出于表面原因时，这个规则不会引起任何问题:
- When the value shifted is a literal.
当值移动为文字时。
- When there is a similar shift at the same position on line before or after. E.g.:
当在同一位置发生类似的位移时，可在前后进行。例如:

```java
bytes[loc+0] = (byte)(value >> 8);
bytes[loc+1] = (byte)(value >> 0);
```
### 一句话说明
- 整数和整数的移位不应大于或等于零





## 70、Invalid "Date" values should not be used

- 主要

>Whether the valid value ranges for `Date` fields start with 0 or 1 varies by field. For instance, month starts at 0, and day of month starts at 1\. Enter a date value that goes past the end of the valid range, and the date will roll without error or exception. For instance, enter 12 for month, and you'll get January of the following year.
日期字段的有效值范围从0开始还是从1开始随字段的不同而不同。例如，month从0开始，day of month从1开始。输入一个日期值，该值超过有效范围的末尾，日期将滚动，没有错误或异常。例如，每个月输入12，就会得到下一年的1月。

>This rule checks for bad values used in conjunction with `java.util.Date`, `java.sql.Date`, and `java.util.Calendar`. Specifically, values outside of the valid ranges:
此规则检查与`java.util.Date`, `java.sql.Date`, 和 `java.util.Calendar`结合。具体来说，有效范围之外的值:

| Field | Valid |
| ------------ | ------------ |
| month | 0-11 |
| date (day) | 0-31 |
| hour | 0-23 |
| minute | 0-60 |
| second | 0-61 |

>Note that this rule does not check for invalid leap years, leap seconds (second = 61), or invalid uses of the 31st day of the month.
注意，此规则不检查无效的闰年、闰秒(秒= 61)或每月31日的无效使用。

### Noncompliant Code Example
```java
Date d = new Date();
d.setDate(25);
d.setYear(2014);
d.setMonth(12);  // Noncompliant; rolls d into the next year

Calendar c = new GregorianCalendar(2014, 12, 25);  // Noncompliant
if (c.get(Calendar.MONTH) == 12) {  // Noncompliant; invalid comparison
  // ...
}
```
### Compliant Solution

```java
Date d = new Date();
d.setDate(25);
d.setYear(2014);
d.setMonth(11);

Calendar c = new Gregorian Calendar(2014, 11, 25);
if (c.get(Calendar.MONTH) == 11) {
  // ...
}
```
### 一句话说明
- 不要使用无效的“日期”值




## 71、Jump statements should not occur in "finally" blocks

- 严重

>Writers of `Serializable` classes can choose to let Java's automatic mechanisms handle serialization and deserialization, or they can choose to handle it themselves by implementing specific methods. However, if the signatures of those methods are not exactly what is expected, they will be ignored and the default serialization mechanisms will kick back in.
可序列化类的编写器可以选择让Java的自动机制处理序列化和反序列化，也可以选择通过实现特定的方法自行处理。但是，如果这些方法的签名不完全符合预期，那么它们将被忽略，默认的序列化机制将重新启动。

### Noncompliant Code Example
```java
public static void main(String[] args) {
  try {
    doSomethingWhichThrowsException();
    System.out.println("OK");   // incorrect "OK" message is printed
  } catch (RuntimeException e) {
    System.out.println("ERROR");  // this message is not shown
  }
}

public static void doSomethingWhichThrowsException() {
  try {
    throw new RuntimeException();
  } finally {
    for (int i = 0; i < 10; i ++) {
      //...
      if (q == i) {
        break; // ignored
      }
    }

    /* ... */
    return;      // Noncompliant - prevents the RuntimeException from being propagated
  }
}
```
### Compliant Solution
```java
public static void main(String[] args) {
  try {
    doSomethingWhichThrowsException();
    System.out.println("OK");
  } catch (RuntimeException e) {
    System.out.println("ERROR");  // "ERROR" is printed as expected
  }
}

public static void doSomethingWhichThrowsException() {
  try {
    throw new RuntimeException();
  } finally {
    for (int i = 0; i < 10; i ++) {
      //...
      if (q == i) {
        break; // ignored
      }
    }

    /* ... */
  }
}
```
### See
- [MITRE, CWE-584](http://cwe.mitre.org/data/definitions/584.html) - Return Inside Finally Block
- [CERT, ERR04-J.](https://www.securecoding.cert.org/confluence/x/mIEbAQ) - Do not complete abruptly from a finally block

### 一句话说明
- 跳转语句不应该出现在“finally”块中

### 例子
```java
    @Test
    public void test71() {
        try {
            doSomethingWhichThrowsException1();
            System.out.println("OK");   // 打印OK，因为有return语句
        } catch (RuntimeException e) {
            System.out.println("ERROR");  // 不会打印ERROR
        }
        try {
            doSomethingWhichThrowsException2();
            System.out.println("OK");   // 不会打印OK
        } catch (RuntimeException e) {
            System.out.println("ERROR");  // 打印ERROR
        }
    }
    public static void doSomethingWhichThrowsException1() {
        try {
            throw new RuntimeException();
        } finally {
            return;
        }
    }
    public static void doSomethingWhichThrowsException2() {
        try {
            throw new RuntimeException();
        } finally {

        }
    }
```






## 72、Locks should be released

- 主要

>If a lock is acquired and released within a method, then it must be released along all execution paths of that method.
如果在一个方法中获取并释放一个锁，那么它必须沿着该方法的所有执行路径释放。

>Failing to do so will expose the conditional locking logic to the method's callers and hence be deadlock-prone.
如果不这样做，就会将条件锁定逻辑暴露给方法的调用者，从而导致死锁。

### Noncompliant Code Example
```java
public class MyClass {
  private Lock lock = new Lock();

  public void doSomething() {
    lock.lock(); // Noncompliant
    if (isInitialized()) {
      // ...
      lock.unlock();
    }
  }
}
```
### Compliant Solution
```java
public class MyClass {
  private Lock lock = new Lock();

  public void doSomething() {
    if (isInitialized()) {
      lock.lock();
      // ...
      lock.unlock();
    }
  }
}
```
### See
-  [MITRE, CWE-459](https://cwe.mitre.org/data/definitions/459.html) - Incomplete Cleanup

### 一句话说明
- 保证锁能够被释放掉





## 73、Loop conditions should be true at least once

- 主要

>If a `for` loop's condition is false before the first loop iteration, the loop will never be executed. Such loops are almost always bugs, particularly when the initial value and stop conditions are hard-coded.
如果for循环的条件在第一次循环迭代之前为false，则循环将永远不会执行。这样的循环几乎总是错误的，特别是当初始值和停止条件被硬编码时。

### Noncompliant Code Example
```java
for (int i = 10; i < 10; i++) {  // Noncompliant
  // ...
```

### 一句话说明
- 循环条件应该至少执行一次





## 74、Loops should not be infinite

- 阻断

>An infinite loop is one that will never end while the program is running, i.e., you have to kill the program to get out of the loop. Whether it is by meeting the loop's end condition or via a `break`, every loop should have an end condition.
无限循环是指程序运行时永远不会结束的循环，即，你必须杀死程序才能跳出循环。无论是通过满足循环的结束条件还是通过中断，每个循环都应该有一个结束条件。

>An infinite loop with an empty body consumes CPU cycles but does nothing. Optimizing compilers and just-in-time systems (JITs) are permitted to (perhaps unexpectedly) remove such a loop. Consequently, programs must not include infinite loops with empty bodies.
一个空体的无限循环消耗CPU周期，但什么也不做。优化编译器和即时系统(jit)允许(可能出乎意料地)删除这样的循环。因此，程序不能包含无限循环的空体。

### Noncompliant Code Example
```java
for (;;) {  // Noncompliant; end condition omitted
  // ...
}

int j;
while (true) { // Noncompliant; end condition omitted
  j++;
}

int k;
boolean b = true;
while (b) { // Noncompliant; b never written to in loop
  k++;
}
```
### Compliant Solution
```java
public class Watermelon implements Serializable {
  // ...
  private void writeObject(java.io.ObjectOutputStream out)
        throws IOException
  {...}

  private void readObject(java.io.ObjectInputStream in)
        throws IOException, ClassNotFoundException
  {...}

  private void readObjectNoData()
        throws ObjectStreamException
  {...}

  protected Object readResolve() throws ObjectStreamException
  {...}

  private Object writeReplace() throws ObjectStreamException
  {...}
```
### See
-  [CERT, MSC01-J.](https://www.securecoding.cert.org/confluence/x/PYHfAw) - Do not use an empty infinite loop

### 一句话说明
-  循环不应该是无限的

### 例子
### Noncompliant Code Example
>This noncompliant code example implements an idle task that continuously executes a loop without executing any instructions within the loop. An optimizing compiler or JIT could remove the `while` loop in this example.
这个不兼容的代码示例实现了一个空闲任务，该任务连续执行一个循环，而不执行循环中的任何指令。优化编译器或JIT可以删除本例中的while循环。

```java
public int nop() {
  while (true) {}
}
```

### Compliant Solution
>This compliant solution avoids use of a meaningless infinite loop by invoking `Thread.sleep()` within the `while` loop. The loop body contains semantically meaningful operations and consequently cannot be optimized away.
这个兼容的解决方案通过在while循环中调用Thread.sleep()来避免使用无意义的无限循环。循环体包含语义上有意义的操作，因此不能进行优化。

```java
// 解决方案一：Thread.sleep()
public final int DURATION=10000; // In milliseconds
 
public void nop() throws InterruptedException {
  while (true) {
    // Useful operations
    Thread.sleep(DURATION);
  }
}
```
> This compliant solution invokes `Thread.yield()`, which causes the thread running this method to consistently defer to other threads:
这个兼容的解决方案调用thread .yield()，这会导致运行这个方法的线程一致地遵从其他线程

```java
// 解决方案二：Thread.yield()
public void nop() {
  while (true) {
    Thread.yield();
  }
}
```





## 75、Loops with at most one iteration should be refactored

- 主要

>A loop with at most one iteration is equivalent to the use of an `if` statement to conditionally execute one piece of code. No developer expects to find such a use of a loop statement. If the initial intention of the author was really to conditionally execute one piece of code, an `if` statement should be used instead.
最多一次迭代的循环相当于使用if语句有条件地执行一段代码。没有开发人员希望找到循环语句的这种用法。如果作者的最初意图确实是有条件地执行一段代码，那么应该使用If语句。

>At worst that was not the initial intention of the author and so the body of the loop should be fixed to use the nested `return`, `break` or `throw` statements in a more appropriate way.
在最坏的情况下，这并不是作者的最初意图，因此应该修复循环体，以便以更合适的方式使用嵌套的return、break或throw语句。

### Noncompliant Code Example
```java
for (int i = 0; i < 10; i++) { // noncompliant, loop only executes once
  printf("i is %d", i);
  break;
}
...
for (int i = 0; i < 10; i++) { // noncompliant, loop only executes once
  if(i == x) {
    break;
  } else {
    printf("i is %d", i);
    return;
  }
}
```
### Compliant Solution
```java
for (int i = 0; i < 10; i++) {
  printf("i is %d", i);
}
...
for (int i = 0; i < 10; i++) {
  if(i == x) {
    break;
  } else {
    printf("i is %d", i);
  }
}
```

### 一句话说明
- for只有1次循环的需要重构





## 76、Map values should not be replaced unconditionally

- 主要

>It is highly suspicious when a value is saved for a key or index and then unconditionally overwritten. Such replacements are likely in error.
当一个值被保存为键或索引，然后被无条件地覆盖时，这是非常可疑的。这样的替换可能是错误的。

### Noncompliant Code Example
```java
letters.put("a", "Apple");
letters.put("a", "Boy");  // Noncompliant

towns[i] = "London";
towns[i] = "Chicago";  // Noncompliant
```
### 一句话说明
- 不应该没任何条件发生地更新Map的值




## 77、Math operands should be cast before assignment

- 次要

>When arithmetic is performed on integers, the result will always be an integer. You can assign that result to a `long`, `double`, or `float` with automatic type conversion, but having started as an `int` or `long`, the result will likely not be what you expect.
当对整数执行算术时，结果总是整数。您可以使用自动类型转换将该结果赋值给long、double或float，但是在以int或long开始之后，结果可能与您所期望的不同。

>For instance, if the result of `int` division is assigned to a floating-point variable, precision will have been lost before the assignment. Likewise, if the result of multiplication is assigned to a `long`, it may have already overflowed before the assignment.
例如，如果将int除法的结果分配给浮点变量，那么在分配之前就会丢失精度。同样，如果将乘法的结果赋值给long，那么在赋值之前它可能已经溢出了。

>In either case, the result will not be what was expected. Instead, at least one operand should be cast or promoted to the final type before the operation takes place.
在这两种情况下，结果都不会像预期的那样。相反，在操作发生之前，至少应该将一个操作数强制转换或提升为最终类型。

### Noncompliant Code Example
```java
float twoThirds = 2/3; // Noncompliant; int division. Yields 0.0
long millisInYear = 1_000*3_600*24*365; // Noncompliant; int multiplication. Yields 1471228928
long bigNum = Integer.MAX_VALUE + 2; // Noncompliant. Yields -2147483647
long bigNegNum =  Integer.MIN_VALUE-1; //Noncompliant, gives a positive result instead of a negative one.
Date myDate = new Date(seconds * 1_000); //Noncompliant, won't produce the expected result if seconds > 2_147_483
...
public long compute(int factor){
  return factor * 10_000;  //Noncompliant, won't produce the expected result if factor > 214_748
}

public float compute2(long factor){
  return factor / 123;  //Noncompliant, will be rounded to closest long integer
}

short a = 533;
int b = 6789;
long c = 4664382371590123456L;
 
float d = a / 7;    // d is 76.0 (truncated)
double e = b / 30;  // e is 226.0 (truncated)
double f = c * 2;   // f is -9.1179793305293046E18
                    // because of integer overflow

int a = 60070;
int b = 57750;
 
double value = Math.ceil(a/b);

```
### Compliant Solution
```java
float twoThirds = 2f/3; // 2 promoted to float. Yields 0.6666667
long millisInYear = 1_000L*3_600*24*365; // 1000 promoted to long. Yields 31_536_000_000
long bigNum = Integer.MAX_VALUE + 2L; // 2 promoted to long. Yields 2_147_483_649
long bigNegNum =  Integer.MIN_VALUE-1L; // Yields -2_147_483_649
Date myDate = new Date(seconds * 1_000L);
...
public long compute(int factor){
  return factor * 10_000L;
}

public float compute2(long factor){
  return factor / 123f;
}

short a = 533;
int b = 6789;
long c = 4664382371590123456L;
 
float d = a / 7.0f;       // d is 76.14286
double e = b / 30.;       // e is 226.3
double f = (double)c * 2; // f is 9.328764743180247E18

int a = 60070;
int b = 57750;
 
double value = Math.ceil(a/((double) b));
```
or
```java
float twoThirds = (float)2/3; // 2 cast to float
long millisInYear = (long)1_000*3_600*24*365; // 1_000 cast to long
long bigNum = (long)Integer.MAX_VALUE + 2;
long bigNegNum =  (long)Integer.MIN_VALUE-1;
Date myDate = new Date((long)seconds * 1_000);
...
public long compute(long factor){
  return factor * 10_000;
}

public float compute2(float factor){
  return factor / 123;
}
```
### See
-   MISRA C++:2008, 5-0-8 - An explicit integral or floating-point conversion shall not increase the size of the underlying type of a cvalue expression.
-   [MITRE, CWE-190](http://cwe.mitre.org/data/definitions/190) - Integer Overflow or Wraparound
-   [CERT, NUM50-J.](https://www.securecoding.cert.org/confluence/x/woIyAQ) - Convert integers to floating point for floating-point operations
-   [CERT, INT18-C.](https://www.securecoding.cert.org/confluence/x/AxE) - Evaluate integer expressions in a larger size before comparing or assigning to that size
-   [SANS Top 25](https://www.sans.org/top25-software-errors/#cat2) - Risky Resource Management

### 一句话说明
- 数字操作在操作或赋值前要转换，避免截断或者溢出

### 例子
```java
    @Test
    public void test77() {

        //案例1
        float twoThirds = 2 / 3; // 错误
        System.out.println(twoThirds); //打印0.0

        twoThirds = 2f / 3; // 正确
        System.out.println(twoThirds); //打印0.6666667

        twoThirds = (float) 2 / 3; // 正确
        System.out.println(twoThirds); //打印0.6666667


        //案例2
        long millisInYear = 1_000 * 3_600 * 24 * 365; // 错误
        System.out.println(millisInYear); //打印1471228928

        millisInYear = 1_000L * 3_600 * 24 * 365; // 正确
        System.out.println(millisInYear); //打印31536000000

        millisInYear = (long) 1_000 * 3_600 * 24 * 365; // 正确
        System.out.println(millisInYear); //打印31536000000

        //案例3
        long bigNum = Integer.MAX_VALUE + 2; //错误
        System.out.println(bigNum); //打印-2147483647

        bigNum = Integer.MAX_VALUE + 2L; // 正确
        System.out.println(bigNum); //打印2147483649

        bigNum = (long) Integer.MAX_VALUE + 2; //正确
        System.out.println(bigNum); //打印2147483649

        //案例4
        long bigNegNum = Integer.MIN_VALUE - 1; //错误
        System.out.println(bigNegNum); //打印2147483647

        bigNegNum = Integer.MIN_VALUE - 1L;//正确
        System.out.println(bigNegNum); //打印-2147483649

        bigNegNum = (long) Integer.MIN_VALUE - 1;//正确
        System.out.println(bigNegNum); //打印-2147483649

        //案例5
        int seconds = 1560482167;
        Date myDate = new Date(seconds * 1_000); //错误写法
        System.out.println(myDate.toString()); //打印Sat Jan 17 15:23:58 CST 1970

        myDate = new Date(seconds * 1_000L);//正确
        System.out.println(myDate.toString()); //打印Fri Jun 14 11:16:07 CST 2019

        myDate = new Date((long) seconds * 1_000);//正确
        System.out.println(myDate.toString()); //打印Fri Jun 14 11:16:07 CST 2019


        System.out.println(compute(20000)); //打印200000000，入参小于2147480结果是对的
        System.out.println(compute(2147480));//打印-36480，入参大于2147480结果是错的

        System.out.println(compute2(10000)); //打印81.0， 自动四舍五入了

        System.out.println(compute3(20000)); //打印200000000，正确
        System.out.println(compute3(2147480));//打印21474800000，正确

        System.out.println(compute4(10000)); //打印81.30081

        System.out.println(compute5(20000)); //打印200000000
        System.out.println(compute5(2147480));//打印21474800000

        System.out.println(compute6(10000)); //打印81.30081
    }


    public long compute(int factor) {
        return factor * 10_000;  //错误
    }

    public float compute2(long factor) {
        return factor / 123;  //错误
    }

    public long compute3(int factor) {
        return factor * 10_000L;
    }

    public float compute4(long factor) {
        return factor / 123f;
    }

    public long compute5(long factor) {
        return factor * 10_000;
    }

    public float compute6(float factor) {
        return factor / 123;
    }
```




## 78、Method parameters, caught exceptions and foreach variables' initial values should not be ignored

- 次要

>While it is technically correct to assign to parameters from within method bodies, doing so before the parameter value is read is likely a bug. Instead, initial values of parameters, caught exceptions, and `foreach` parameters should be, if not treated as `final`, then at least read before reassignment.
虽然从技术上讲，从方法体内部分配参数是正确的，但是在读取参数值之前这样做很可能是一个错误。相反，参数的初始值、捕获的异常和`foreach`参数应该是，如果不作为`final`处理，那么至少在重新分配之前读取。

### Noncompliant Code Example
```java
public void doTheThing(String str, int i, List<String> strings) {
  str = Integer.toString(i); // Noncompliant

  for (String s : strings) {
    s = "hello world"; // Noncompliant
  }
}
```
### See
- MISRA C:2012, 17.8 - A function parameter should not be modified

### 一句话说明
- 不要忽略方法参数、捕捉异常、for循环里参数的初始化




## 79、Methods "wait(...)", "notify()" and "notifyAll()" should not be called on Thread instances

- 阻断

>The methods `wait(...)`, `notify()` and `notifyAll()` are available on a `Thread` instance, but only because all classes in Java extend `Object` and therefore automatically inherit those methods. But there are two very good reasons for not calling them on a `Thread`:
方法`wait(...)`, `notify()` 和 `notifyAll()` 在线程实例中可用，但这只是因为Java中的所有类都扩展对象，因此会自动继承这些方法。但是有两个很好的理由不使用线程调用它们:
-   Internally, the JVM relies on these methods to change the state of the Thread (`BLOCKED`, `WAITING`, ...), so calling them will corrupt the behavior of the JVM.
在内部，JVM依赖这些方法来更改线程的状态(阻塞、等待、……)，因此调用它们将破坏JVM的行为。
-   It is not clear (perhaps even to the original coder) what is really expected. For instance, it is waiting for the execution of the Thread to suspended, or is it the acquisition of the object monitor that is waited for?
目前还不清楚(甚至对最初的编码人员也不清楚)真正的期望是什么。例如，它正在等待线程的执行被挂起，还是正在等待对象监视器的获取?

### Noncompliant Code Example
```java
Thread myThread = new Thread(new RunnableJob());
...
myThread.wait(2000);
```

### 一句话说明
- 不应该在线程实例上调用`wait(...)`, `notify()` 和 `notifyAll()`

### 解释说明
[为什么wait(), notify(), notifyAll()必须要在synchronized方法/块](https://www.jianshu.com/p/20abb4047d78 "为什么wait(), notify(), notifyAll()必须要在synchronized方法/块")




## 80、Methods should not be named "tostring", "hashcode" or "equal"

- 主要

>Naming a method `tostring`, `hashcode()` or `equal` is either:
-   A bug in the form of a typo. Overriding `toString`, `Object.hashCode()` (note the camelCasing) or `Object.equals` (note the 's' on the end) was meant, and the application does not behave as expected.
打字错误的一种错误。重写`toString`, `Object.hashCode()`(注意驼峰式大小写)或`Object.equals`(注意末尾的“s”)的意思是，应用程序的行为不像预期的那样。
-   Done on purpose. The name however will confuse every other developer, who may not notice the naming difference, or who will think it is a bug.
故意做的。但是，这个名称会使其他开发人员感到困惑，他们可能没有注意到命名的差异，或者认为这是一个bug。

In both cases, the method should be renamed.
在这两种情况下，都应该重命名方法。

### Noncompliant Code Example
```java
public int hashcode() { /* ... */ }  // Noncompliant

public String tostring() { /* ... */ } // Noncompliant

public boolean equal(Object obj) { /* ... */ }  // Noncompliant
```
### Compliant Solution
```java
@Override
public int hashCode() { /* ... */ }

@Override
public String toString() { /* ... */ }

@Override
public boolean equals(Object obj) { /* ... */ }
```
### 一句话说明
- 定义的方法名不能和`tostring`, `hashcode()` or `equal` 一样，如果要重写方法必须加上`@Override`注解





## 81、Methods should not call same-class methods with incompatible "@Transactional" values

- 阻断

>When using Spring proxies, calling a method in the same class (e.g. `this.aMethod()`) with an incompatible `@Transactional` requirement will result in runtime exceptions because Spring only "sees" the caller and makes no provisions for properly invoking the callee.
当使用Spring代理时，使用不兼容的@Transactional需求调用同一个类中的方法(例如this.aMethod())将导致运行时异常，因为Spring只“看到”调用者，而没有为正确调用被调用者做出任何规定。

Therefore, certain calls should never be made within the same class:
因此，某些调用不应该在同一个类中执行:

| From | To |
| ------------ | ------------ |
| non-`@Transactional` | MANDATORY, NESTED, REQUIRED, REQUIRES_NEW |
| MANDATORY | NESTED, NEVER, NOT_SUPPORTED, REQUIRES_NEW |
| NESTED | NESTED, NEVER, NOT_SUPPORTED, REQUIRES_NEW |
| NEVER | MANDATORY, NESTED, REQUIRED, REQUIRES_NEW |
| NOT_SUPPORTED | MANDATORY, NESTED, REQUIRED, REQUIRES_NEW |
| REQUIRED or `@Transactional` | NESTED, NEVER, NOT_SUPPORTED, REQUIRES_NEW |
| REQUIRES_NEW | NESTED, NEVER, NOT_SUPPORTED, REQUIRES_NEW |
| SUPPORTS | MANDATORY, NESTED, NEVER, NOT_SUPPORTED, REQUIRED, REQUIRES_NEW |


### Noncompliant Code Example
```java
@Override
public void doTheThing() {
  // ...
  actuallyDoTheThing();  // Noncompliant
}

@Override
@Transactional
public void actuallyDoTheThing() {
  // ...
}
```
### Compliant Solution
```java
 class Test81 {
        public void doTheThing() {
            actuallyDoTheThing();  // 在同一个类里，调用有事务的方法，actuallyDoTheThing事务不会生效
        }

        @Transactional
        public void actuallyDoTheThing() {
            // ...
        }
    }

    class Test81Right {
        public void doTheThing2() {
            //使用代理对象调用事务方法开始事务
            Test81Right test81Right = (Test81Right) AopContext.currentProxy();
            test81Right.actuallyDoTheThing2();
        }

        `@Transactional`注解的方法，注解不会生效
        public void actuallyDoTheThing2() {
            // ...
        }
    }
```
### 一句话说明
- 同一个方法里不可以直接调用带有`@Transactional`注解的方法，注解不会生效

### 技术解释
[踩坑! spring事务,非事务方法与事务方法执行相互调用](https://blog.csdn.net/m0_38027656/article/details/84190949 "踩坑! spring事务,非事务方法与事务方法执行相互调用")





## 82、Min and max used in combination should not always return the same value

- 主要

>When using `Math.min()` and `Math.max()` together for bounds checking, it's important to feed the right operands to each method. `Math.min()` should be used with the **upper** end of the range being checked, and `Math.max()` should be used with the **lower** end of the range. Get it backwards, and the result will always be the same end of the range.
当同时使用`Math.min()`和`Math.max()`进行边界检查时，重要的是为每个方法提供正确的操作数。应该使用`Math.min()`检查范围的上端，使用`Math.max()`检查范围的下端。把它倒回去，结果总是相同的。

### Noncompliant Code Example
```java
 private static final int UPPER = 20;
  private static final int LOWER = 0;

  public int doRangeCheck(int num) {    // Let's say num = 12
    int result = Math.min(LOWER, num);  // result = 0
    return Math.max(UPPER, result);     // Noncompliant; result is now 20: even though 12 was in the range
  }
```
### Compliant Solution
Swapping method min() and max() invocations without changing parameters.
```java
  private static final int UPPER = 20;
  private static final int LOWER = 0;

  public int doRangeCheck(int num) {    // Let's say num = 12
    int result = Math.max(LOWER, num);  // result = 12
    return Math.min(UPPER, result);     // Compliant; result is still 12
  }
```
or swapping bounds UPPER and LOWER used as parameters without changing the invoked methods.
```java
  private static final int UPPER = 20;
  private static final int LOWER = 0;

  public int doRangeCheck(int num) {    // Let's say num = 12
    int result = Math.min(UPPER, num);  // result = 12
    return Math.max(LOWER, result);     // Compliant; result is still 12
  }
```
### 一句话说明
- 组合使用的Min和max不应该总是返回相同的值

### 技术解释
- 在[a,b]区间返回内，判断x是否在区间内，通过x跟b比较，是否x<=b并且x是否>=aj即可







## 83、Neither "Math.abs" nor negation should be used on numbers that could be "MIN_VALUE"

- 次要

>It is possible for a call to `hashCode` to return `Integer.MIN_VALUE`. Take the absolute value of such a hashcode and you'll still have a negative number. Since your code is likely to assume that it's a positive value instead, your results will be unreliable.
调用`hashCode`可以返回`Integer.MIN_VALUE`。取这样一个`hashcode`的绝对值，仍然会得到一个负数。因为您的代码可能会假设它是一个正值，所以您的结果将是不可靠的。

>Similarly, `Integer.MIN_VALUE` could be returned from `Random.nextInt()` or any object's `compareTo` method, and `Long.MIN_VALUE` could be returned from `Random.nextLong()`. Calling `Math.abs` on values returned from these methods is similarly ill-advised.
同样,`Integer.MIN_VALUE`可以从`Random.nextInt()`或任何对象的compareTo方法返回，并且`Long.MIN_VALUE` 可以从`Random.nextLong()`返回。调用 `Math.abs`这些方法返回的值也是不明智的。

### Noncompliant Code Example
```java
public void doSomething(String str) {
  if (Math.abs(str.hashCode()) > 0) { // Noncompliant
    // ...
  }
}
```
### Compliant Solution
```java
public void doSomething(String str) {
  if (str.hashCode() != 0) {
    // ...
  }
}
```
### 一句话说明
- 不要对数值类型的MIN_VALUE值或返回值为此值进行Math.abs与取反操作，因为不会起作用。

### 例子
```java
    @Test
    public void test83() {
        String str = "sonar";
        System.out.println(str.hashCode());//109620547
        System.out.println(Math.abs(str.hashCode()));//109620547

        //最小整数取绝对值，不会出现大于0的值，2147483648超过了Integer最大值
        System.out.println(Math.abs(Integer.MIN_VALUE));//-2147483648
        System.out.println(Math.abs(Integer.MAX_VALUE));//2147483647
        System.out.println(Math.abs(Long.MIN_VALUE));//-9223372036854775808
        System.out.println(Math.abs(Long.MAX_VALUE));//9223372036854775807
    }
```





## 84、Non-primitive fields should not be "volatile"

- 次要

>Marking an array `volatile` means that the array itself will always be read fresh and never thread cached, but the items _in_ the array will not be. Similarly, marking a mutable object field `volatile` means the object _reference_ is `volatile` but the object itself is not, and other threads may not see updates to the object state.
标记数组易失性意味着数组本身将始终被读取为新数据，而不会被线程缓存，但是数组中的项不会被缓存。类似地，标记可变对象字段volatile意味着对象引用是volatile，而对象本身不是，其他线程可能看不到对象状态的更新。

This can be salvaged with arrays by using the relevant AtomicArray class, such as `AtomicIntegerArray`, instead. For mutable objects, the `volatile` should be removed, and some other method should be used to ensure thread-safety, such as synchronization, or ThreadLocal storage.。
这可以通过使用相关的AtomicArray类(例如AtomicIntegerArray)与数组一起回收。对于可变对象，应该删除volatile，并使用其他一些方法来确保线程安全，比如同步或线程本地存储。

This safe publication guarantee applies only to primitive fields and object references. Programmers commonly use imprecise terminology and speak about "member objects." For the purposes of this visibility guarantee, the actual member is the object reference; the objects referred to (aka _referents_) by volatile object references are beyond the scope of the safe publication guarantee. Consequently, declaring an object reference to be volatile is insufficient to guarantee that changes to the members of the referent are published to other threads. A thread may fail to observe a recent write from another thread to a member field of such an object referent.
这种安全发布保证只适用于基本字段和对象引用。程序员通常使用不精确的术语并谈论“成员对象”。为了保证可见性，实际成员是对象引用;易失性对象引用引用的对象(也称为引用对象)超出了安全发布保证的范围。因此，将对象引用声明为volatile不足以保证对referent成员的更改发布到其他线程。线程可能无法观察到最近从另一个线程写到此类引用对象的成员字段。

Furthermore, when the referent is mutable and lacks thread safety, other threads might see a partially constructed object or an object in a (temporarily) inconsistent state [[Goetz 2007](https://wiki.sei.cmu.edu/confluence/display/java/Rec.+AA.+References#Rec.AA.References-Goetz07)]. However, when the referent is [immutable](https://www.securecoding.cert.org/confluence/display/jg/BB.+Definitions#BBDefinitions-immutable), declaring the reference volatile suffices to guarantee safe publication of the members of the referent. Programmers cannot use the `volatile` keyword to guarantee safe publication of mutable objects. Use of the `volatile` keyword can only guarantee safe publication of primitive fields, object references, or fields of immutable object referents.
此外，当referent是可变的且缺乏线程安全性时，其他线程可能会看到部分构造的对象或处于(临时)不一致状态的对象[Goetz 2007]。但是，当referent是不可变的时，声明引用volatile就足够保证referent成员的安全发布。程序员不能使用volatile关键字来保证可变对象的安全发布。使用volatile关键字只能保证基本字段、对象引用或不可变对象引用的字段的安全发布。

Confusing a volatile object with the volatility of its member objects is a similar error to the one described in [OBJ50-J. Never confuse the immutability of a reference with that of the referenced object](https://wiki.sei.cmu.edu/confluence/display/java/OBJ50-J.+Never+confuse+the+immutability+of+a+reference+with+that+of+the+referenced+object).
混淆易失性对象及其成员对象的易失性与OBJ50-J中描述的错误类似。永远不要将引用的不变性与引用对象的不变性混淆。

### Noncompliant Code Example
```java
private volatile int [] vInts;  // Noncompliant
private volatile MyObj myObj;  // Noncompliant

final class Foo {
  private volatile int[] arr = new int[20];// Noncompliant
 
  public int getFirst() {
    return arr[0];
  }
 
  public void setFirst(int n) {
    arr[0] = n;
  }
 
  // ...
}


final class Foo {
  private volatile Map<String, String> map;// Noncompliant
  
  public Foo() {
    map = new HashMap<String, String>();
    // Load some useful values into map
  }
  
  public String get(String s) {
    return map.get(s);
  }
  
  public void put(String key, String value) {
    // Validate the values before inserting
    if (!value.matches("[\\w]*")) {
      throw new IllegalArgumentException();
    }
    map.put(key, value);
  }
}

final class DateHandler {
  private static volatile DateFormat format =
    DateFormat.getDateInstance(DateFormat.MEDIUM);// Noncompliant
 
  public static java.util.Date parse(String str)
      throws ParseException {
    return format.parse(str);
  }
}

```
### Compliant Solution
```java
private AtomicIntegerArray vInts;
private MyObj myObj;

//解决方案一：AtomicIntegerArray
final class Foo {
  private final AtomicIntegerArray atomicArray =
    new AtomicIntegerArray(20);
 
  public int getFirst() {
    return atomicArray.get(0);
  }
 
  public void setFirst(int n) {
    atomicArray.set(0, 10);
  }
 
  // ...
}

//解决方案二：Synchronization
final class Foo {
  private int[] arr = new int[20];
 
  public synchronized int getFirst() {
    return arr[0];
  }
 
  public synchronized void setFirst(int n) {
    arr[0] = n;
  }
}

final class Foo {
  private final Map<String, String> map;
  public Foo() {
    map = new HashMap<String, String>();
    // Load some useful values into map
  }
  public synchronized String get(String s) {
    return map.get(s);
  }
  public synchronized void put(String key, String value) {
    // Validate the values before inserting
    if (!value.matches("[\\w]*")) {
      throw new IllegalArgumentException();
    }
    map.put(key, value);
  }
}

//解决方案一：Instance per Call
final class DateHandler {
  public static java.util.Date parse(String str)
      throws ParseException {
    return DateFormat.getDateInstance(
      DateFormat.MEDIUM).parse(str);
  }
}

//解决方案二：Synchronization
final class DateHandler {
  private static DateFormat format =
    DateFormat.getDateInstance(DateFormat.MEDIUM);
 
  public static java.util.Date parse(String str)
      throws ParseException {
    synchronized (format) {
      return format.parse(str);
    }
  }
}

//解决方案三：ThreadLocal Storage
final class DateHandler {
  private static final ThreadLocal<DateFormat> format =
    new ThreadLocal<DateFormat>() {
    @Override protected DateFormat initialValue() {
      return DateFormat.getDateInstance(DateFormat.MEDIUM);
    }
  };
  // ...
}

```
### See
-   [CERT, CON50-J.](https://www.securecoding.cert.org/confluence/x/twD1AQ) - Do not assume that declaring a reference volatile guarantees safe publication of the members of the referenced object

### 一句话说明
- 非基本字段不应该是“易变的”

### 技术分析
- [就是要你懂 Java 中 volatile 关键字实现原理](https://mp.weixin.qq.com/s/keaORK_ePM3x2GLcrFZZHw? "就是要你懂 Java 中 volatile 关键字实现原理")





## 85、Non-public methods should not be "@Transactional"

- 主要

>Marking a non-public method `@Transactional` is both useless and misleading because Spring doesn't "see" non-`public` methods, and so makes no provision for their proper invocation. Nor does Spring make provision for the methods invoked by the method it called.
标记非公共方法@Transactional是无用的，而且会误导人，因为Spring没有“看到”非公共方法，因此没有为它们的正确调用提供任何准备。Spring也没有为它调用的方法调用的方法提供条件。

>Therefore marking a `private` method, for instance, `@Transactional` can only result in a runtime error or exception if the method is actually written to be `@Transactional`.
因此，标记一个私有方法，例如，@Transactional只能在方法实际被写为@Transactional时导致运行时错误或异常。

### Noncompliant Code Example
```java
@Transactional  // Noncompliant
private void doTheThing(ArgClass arg) {
  // ...
}
```
### 一句话说明
- 非public方法不要注解Transactional,调用时spring 会抛出异常

### 技术分析
- When you use proxies, you should apply the `@Transactional` annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. If you need to annotate non-public methods, consider using AspectJ (described later).
当使用代理时，应该只将`@Transactional`注释应用于具有公共可见性的方法。如果使用`@Transactional`注释注释受保护的、私有的或包可见的方法，则不会引发错误，但是注释的方法不显示配置的事务设置。如果需要注释非公共方法，请考虑使用`AspectJ`(稍后将进行描述)。

- [官方解释](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/transaction.html "官方解释")







## 86、Non-serializable classes should not be written

- 主要

>Nothing in a non-serializable class will be written out to file, and attempting to serialize such a class will result in an exception being thrown. Only a class that `implements Serializable` or one that extends such a class can successfully be serialized (or de-serialized).
不可序列化类中的任何内容都不会写入文件，并且试图序列化此类将导致抛出异常。只有实现Serializable的类或扩展该类的类才能成功地序列化(或反序列化)。

### Noncompliant Code Example
```java
public class Vegetable {  // neither implements Serializable nor extends a class that does
  //...
}

public class Menu {
  public void meal() throws IOException {
    Vegetable veg;
    //...
    FileOutputStream fout = new FileOutputStream(veg.getName());
    ObjectOutputStream oos = new ObjectOutputStream(fout);
    oos.writeObject(veg);  // Noncompliant. Nothing will be written
  }
}
```
### Compliant Solution
```java
public class Vegetable implements Serializable {  // can now be serialized
  //...
}

public class Menu {
  public void meal() throws IOException {
    Vegetable veg;
    //...
    FileOutputStream fout = new FileOutputStream(veg.getName());
    ObjectOutputStream oos = new ObjectOutputStream(fout);
    oos.writeObject(veg);
  }
}
```
### 一句话说明
- 执行写操作的类要序列化，否则会抛出异常




## 87、Non-serializable objects should not be stored in "HttpSession" objects

- 主要

>If you have no intention of writting an `HttpSession` object to file, then storing non-`serializable` objects in it may not seem like a big deal. But whether or not you explicitly serialize the session, it may be written to disk anyway, as the server manages its memory use in a process called "passivation". Further, some servers automatically write their active sessions out to file at shutdown & deserialize any such sessions at startup.
如果您不打算将HttpSession对象写入文件，那么在其中存储不可序列化的对象似乎没什么大不了的。但是无论您是否显式地序列化会话，它都可能被写入磁盘，因为服务器在一个称为“挂起”的进程中管理其内存使用。此外，一些服务器在关闭时自动将其活动会话写入文件，并在启动时反序列化任何此类会话。

>The point is, that even though `HttpSession` does not `extend Serializable`, you must nonetheless assume that it will be serialized, and understand that if you've stored non-serializable objects in the session, errors will result.
关键是，即使HttpSession没有扩展Serializable，您也必须假定它将被序列化，并且要理解，如果您在会话中存储了不可序列化的对象，将会导致错误。

### Noncompliant Code Example
```java
public class Address {
  //...
}

//...
HttpSession session = request.getSession();
session.setAttribute("address", new Address());  // Noncompliant; Address isn't serializable
```
### See

- [MITRE, CWE-579](http://cwe.mitre.org/data/definitions/579.html) - J2EE Bad Practices: Non-serializable Object Stored in Session

### 一句话说明
-  HttpSession要保存序列化的对象，没有序列化的对象保存会报异常






## 88、Non-thread-safe fields should not be static

- 主要

>Not all classes in the standard Java library were written to be thread-safe. Using them in a multi-threaded manner is highly likely to cause data problems or exceptions at runtime.
并不是标准Java库中的所有类都被编写为线程安全的。以多线程方式使用它们很可能在运行时导致数据问题或异常。

>This rule raises an issue when an instance of `Calendar`, `DateFormat`, `javax.xml.xpath.XPath`, or `javax.xml.validation.SchemaFactory` is marked `static`.
当`Calendar`, `DateFormat`, `javax.xml.xpath.XPath`的实例出现问题时，此规则会引发问题，或者`javax.xml.validation.SchemaFactory`被标记为静态。
### Noncompliant Code Example
```java
public class MyClass {
  private static SimpleDateFormat format = new SimpleDateFormat("HH-mm-ss");  // Noncompliant
  private static Calendar calendar = Calendar.getInstance();  // Noncompliant
```
### Compliant Solution
```java
public class MyClass {
  private SimpleDateFormat format = new SimpleDateFormat("HH-mm-ss");
  private Calendar calendar = Calendar.getInstance();
```
### 一句话说明
- 非线程安全字段不应该是静态的

### 技术分析
- [java中的线程安全（Threadsafe）及其四种策略](https://blog.csdn.net/qq_38969070/article/details/80767370 "java中的线程安全（Threadsafe）及其四种策略")






## 89、Null pointers should not be dereferenced

- 主要

>A reference to `null` should never be dereferenced/accessed. Doing so will cause a `NullPointerException` to be thrown. At best, such an exception will cause abrupt program termination. At worst, it could expose debugging information that would be useful to an attacker, or it could allow an attacker to bypass security measures.
对null的引用永远不应该取消引用/访问。这样做将导致抛出NullPointerException。在最好的情况下，这样的异常将导致程序突然终止。在最坏的情况下，它可能会暴露对攻击者有用的调试信息，或者允许攻击者绕过安全措施。

>Note that when they are present, this rule takes advantage of `@CheckForNull` and `@Nonnull` annotations defined in [JSR-305](https://jcp.org/en/jsr/detail?id=305) to understand which values are and are not nullable except when `@Nonnull` is used on the parameter to `equals`, which by contract should always work with null.
注意，当它们出现时，这个规则利用JSR-305中定义的@CheckForNull和@Nonnull注释来理解哪些值是可空的，哪些值不是可空的，除非在参数to equals上使用@Nonnull，根据合约，这两个值应该始终使用null。

### Noncompliant Code Example
```java
@CheckForNull
String getName(){...}

public boolean isNameEmpty() {
  return getName().length() == 0; // Noncompliant; the result of getName() could be null, but isn't null-checked
}
Connection conn = null;
Statement stmt = null;
try{
  conn = DriverManager.getConnection(DB_URL,USER,PASS);
  stmt = conn.createStatement();
  // ...

}catch(Exception e){
  e.printStackTrace();
}finally{
  stmt.close();   // Noncompliant; stmt could be null if an exception was thrown in the try{} block
  conn.close();  // Noncompliant; conn could be null if an exception was thrown
}
private void merge(@Nonnull Color firstColor, @Nonnull Color secondColor){...}

public  void append(@CheckForNull Color color) {
    merge(currentColor, color);  // Noncompliant; color should be null-checked because merge(...) doesn't accept nullable parameters
}
void paint(Color color) {
  if(color == null) {
    System.out.println("Unable to apply color " + color.toString());  // Noncompliant; NullPointerException will be thrown
    return;
  }
  ... 
}
```
### See
-   [MITRE, CWE-476](http://cwe.mitre.org/data/definitions/476.html) - NULL Pointer Dereference
-   [CERT, EXP34-C.](https://www.securecoding.cert.org/confluence/x/PAw) - Do not dereference null pointers
-   [CERT, EXP01-J.](https://www.securecoding.cert.org/confluence/x/ZwDOAQ) - Do not use a null in a case where an object is required

### 一句话说明
- 不应该引用空指针






## 90、Optional value should only be accessed after calling isPresent()

- 主要

>`Optional` value can hold either a value or not. The value held in the `Optional` can be accessed using the `get()` method, but it will throw a
`NoSuchElementException` if there is no value present. To avoid the exception, calling the `isPresent()` method should always be done before any call to `get()`.
`Optional` 可以保存值，也可以不保存值。`Optional`方法中包含的值可以使用get()方法访问，但是如果没有值，它会抛出一个`NoSuchElementException`异常。为了避免异常，应该总是在调用get()之前调用isPresent()方法。

>Alternatively, note that other methods such as `orElse(...)`, `orElseGet(...)` or `orElseThrow(...)` can be used to specify what to do with an empty `Optional`.
另外，请注意其他方法，如`orElse(…)`、`orElseGet(…)`或`orElseThrow(…)`，可用于指定如何处理空的可选方法。

### Noncompliant Code Example
```java
Optional<String> value = this.getOptionalValue();
// ...
String stringValue = value.get(); // Noncompliant
```
### Compliant Solution
```java
Optional<String> value = this.getOptionalValue();
// ...
if (value.isPresent()) {
  String stringValue = value.get();
}
```
or
```java
Optional<String> value = this.getOptionalValue();
// ...
String stringValue = value.orElse("default");
```
### See
- [MITRE, CWE-476](https://cwe.mitre.org/data/definitions/476.html) - NULL Pointer Dereference

### 一句话说明
- 调用Optional类的get方法前先调用isPresent方法检查是否存储






## 53、Overrides should match their parent class methods in synchronization

- 主要

>When `@Overrides` of `synchronized` methods are not themselves `synchronized`, the result can be improper synchronization as callers rely on the thread-safety promised by the parent class.
当同步方法的@Overrides本身不同步时，结果可能是不正确的同步，因为调用者依赖于父类承诺的线程安全性。

### Noncompliant Code Example
```java
public class Parent {

  synchronized void foo() {
    //...
  }
}

public class Child extends Parent {

 @Override
  public foo () {  // Noncompliant
    // ...
    super.foo();
  }
}
```
### Compliant Solution
```java
public class Parent {

  synchronized void foo() {
    //...
  }
}

public class Child extends Parent {

  @Override
  synchronized foo () {
    // ...
    super.foo();
  }
}
```
### See
-   [CERT, TSM00-J](https://www.securecoding.cert.org/confluence/x/XgAZAg) - Do not override thread-safe methods with methods that are not thread-safe

### 一句话说明
- 重写应该与同步中的父类方法匹配





## 92、Custom serialization method signatures should meet requirements

- 阻断

>Because `printf`-style format strings are interpreted at runtime, rather than validated by the Java compiler, they can contain errors that lead to unexpected behavior or runtime errors. This rule statically validates the good behavior of `printf`-style formats when calling the `format(...)` methods of `java.util.Formatter`, `java.lang.String`, `java.io.PrintStream`, `MessageFormat`, and `java.io.PrintWriter` classes and the `printf(...)` methods of `java.io.PrintStream` or `java.io.PrintWriter` classes.
因为printf样式的格式字符串是在运行时解释的，而不是由Java编译器验证的，所以它们可能包含导致意外行为或运行时错误的错误。当调用 `java.util.Formatter`, `java.lang.String`, `java.io.PrintStream`, `MessageFormat`, 和 `java.io.PrintWriter`的`format(...)`方法和调用`java.io.PrintStream` 、 `java.io.PrintWriter`的`printf(...)` 方法时 ，该规则静态地验证printf样式格式的良好行为。


### Noncompliant Code Example
```java
String.format("The value of my integer is %d", "Hello World");  // Noncompliant; an 'int' is expected rather than a String
String.format("Duke's Birthday year is %tX", c);  //Noncompliant; X is not a supported time conversion character
String.format("Display %0$d and then %d", 1);   //Noncompliant; arguments are numbered starting from 1
String.format("Not enough arguments %d and %d", 1);  //Noncompliant; the second argument is missing
String.format("%< is equals to %d", 2);   //Noncompliant; the argument index '<' refers to the previous format specifier but there isn't one

MessageFormat.format("Result {1}.", value); // Noncompliant; Not enough arguments. (first element is {0})
MessageFormat.format("Result {{0}.", value); // Noncompliant; Unbalanced number of curly brace (single curly braces should be escaped)
MessageFormat.format("Result ' {0}", value); // Noncompliant; Unbalanced number of quotes (single quote must be escaped)

java.util.logging.Logger logger;
logger.log(java.util.logging.Level.SEVERE, "Result {1}!", 14); // Noncompliant {{Not enough arguments.}}

org.slf4j.Logger slf4jLog;
org.slf4j.Marker marker;

slf4jLog.debug(marker, "message {}"); // Noncompliant {{Not enough arguments.}}
```
### Compliant Solution
```java
String.format("The value of my integer is %d", 3);
String.format("Duke's Birthday year is %tY", c);
String.format("Display %1$d and then %d", 1);
String.format("Not enough arguments %d and %d", 1, 2);
String.format("%d is equals to %<", 2);

MessageFormat.format("Result {0}.", value);
MessageFormat.format("Result {0} & {1}.", value, value);
MessageFormat.format("Result {0}.", myObject);

java.util.logging.Logger logger;
logger.log(java.util.logging.Level.SEVERE, "Result {1}!", 14, 2); // Noncompliant {{Not enough arguments.}}

org.slf4j.Logger slf4jLog;
org.slf4j.Marker marker;

slf4jLog.debug(marker, "message {}", 1);
```
### See
-  [CERT, FIO47-C.](https://www.securecoding.cert.org/confluence/x/wQA1) - Use valid format strings

### 一句话说明
- 因为Printf风格格式化是在运行期解读，而不是在编译期检验,会存在风险






## 93、Raw byte values should not be used in bitwise operations in combination with shifts

- 主要

>When reading bytes in order to build other primitive values such as `int`s or `long`s, the `byte` values are automatically promoted, but that promotion can have unexpected results.
当读取字节以构建其他原始值(如int或long)时，字节值会自动提升，但这种提升可能会产生意想不到的结果。

>For instance, the binary representation of the integer 640 is `0b0000_0010_1000_0000`, which can also be written with the array of (unsigned) bytes `[2, 128]`. However, since Java uses two's complement, the representation of the integer in signed bytes will be `[2, -128]` (because the `byte` `0b1000_0000` is promoted to the `int``0b1111_1111_1111_1111_1111_1111_1000_0000`). Consequently, trying to reconstruct the initial integer by shifting and adding the values of the bytes without taking care of the sign will not produce the expected result.
例如，整数640的二进制表示形式是0b0000_0010_1000_0000，也可以用(无符号)字节数组[2,128]编写。但是，由于Java使用2的补码，整数的带符号字节表示形式将是[2，-128](因为字节0b1000_0000被提升为int 0b1111_1111_1111_1111_1111_1111_1000_0000)。因此，试图通过移位和添加字节的值来重构初始整数，而不考虑符号，将不会产生预期的结果。

>To prevent such accidental value conversion, use bitwise and (`&`) to combine the `byte` value with `0xff` (255) and turn all the higher bits back off.
为了防止这种意外的值转换，使用bit - wise和(&)将字节值与0xff(255)组合起来，并关闭所有较高的位。

>This rule raises an issue any time a `byte` value is used as an operand without `& 0xff`, when combined with shifts.
每当将字节值用作没有& 0xff的操作数时，当与移位结合使用时，该规则都会引发一个问题。

### Noncompliant Code Example
```java
 int intFromBuffer() {
    int result = 0;
    for (int i = 0; i < 4; i++) {
      result = (result << 8) | readByte(); // Noncompliant
    }
    return result;
  }
```
### Compliant Solution
```java
 int intFromBuffer() {
    int result = 0;
    for (int i = 0; i < 4; i++) {
      result = (result << 8) | (readByte() & 0xff);
    }
    return result;
  }
```
### See
-   [CERT, NUM52-J.](https://www.securecoding.cert.org/confluence/x/SAHEAw) - Be aware of numeric promotion behavior

### 一句话说明
- 原始字节值不应参与位运算





## 94、Reflection should not be used to check non-runtime annotations

- 主要

>The writer of an annotation can set one of three retention policies for it:

-   `RetentionPolicy.SOURCE` - these annotations are dropped during compilation, E.G. `@Override`, `@SuppressWarnings`.
这些注解在编译期间被删除
-   `RetentionPolicy.CLASS` - these annotations are present in a compiled class but not loaded into the JVM at runtime. This is the default.
这些注解出现在编译后的类中，但不会运行时加载到JVM中。这是默认值。
-   `RetentionPolicy.RUNTIME` - these annotations are present in the class file and loaded into the JVM.
这些注解出现在类文件中，并加载到JVM中

>Only annotations that have been given a `RUNTIME` retention policy will be available to reflection. Testing for annotations with any other retention policy is simply an error, since the test will always return false.
只有已被赋予运行时保留策略的注解才可用于反射。使用任何其他保留策略测试注释都只是一个错误，因为测试总是返回false。

>This rule checks that reflection is not used to detect annotations that do not have `RUNTIME` retention.
此规则检查反射是否用于检测没有运行时保留的注解。

### Noncompliant Code Example
```java
Method m = String.class.getMethod("getBytes", new Class[] {int.class,
int.class, byte[].class, int.class});
if (m.isAnnotationPresent(Override.class)) {  // Noncompliant; test will always return false, even when @Override is present in the code
```
### 一句话说明
- 反射不应该用于检查非运行时注解






## 95、Related "if/else if" statements should not have the same condition

- 主要

>A chain of `if`/`else if` statements is evaluated from top to bottom. At most, only one branch will be executed: the first one with a condition that evaluates to `true`.
从上到下计算if/else if语句链。最多只执行一个分支:第一个条件为true的分支。

>Therefore, duplicating a condition automatically leads to dead code. Usually, this is due to a copy/paste error. At best, it's simply dead code and at worst, it's a bug that is likely to induce further bugs as the code is maintained, and obviously it could lead to unexpected behavior.
因此，复制条件会自动导致死代码。通常，这是由于复制/粘贴错误造成的。在最好的情况下，它只是死代码，在最坏的情况下，它是一个bug，随着代码的维护，它可能会引发更多的bug，并且很明显，它可能会导致意想不到的行为。

### Noncompliant Code Example
```java
if (param == 1)
  openWindow();
else if (param == 2)
  closeWindow();
else if (param == 1)  // Noncompliant
  moveWindowToTheBackground();
}
```
### Compliant Solution
```java
if (param == 1)
  openWindow();
else if (param == 2)
  closeWindow();
else if (param == 3)
  moveWindowToTheBackground();
}
```
### See
-  [CERT, MSC12-C.](https://www.securecoding.cert.org/confluence/x/NYA5) - Detect and remove code that has no effect or is never executed

### 一句话说明
- if/else if中不应该有相同的条件






## 96、Resources should be closed

- 阻断

>Connections, streams, files, and other classes that implement the `Closeable` interface or its super-interface, `AutoCloseable`, needs to be closed after use. Further, that `close` call must be made in a `finally` block otherwise an exception could keep the call from being made. Preferably, when class implements `AutoCloseable`, resource should be created using "try-with-resources" pattern and will be closed automatically.
连接、流、文件和其他已实现 `Closeable`接口或其父接口`AutoCloseable`的类在使用后需要关闭。此外，必须在`finally`块中执行`close`调用，否则异常可能会阻止调用的执行。最好，当类实现 `AutoCloseable`时，应该使用“try-with-resources”模式创建资源，并自动关闭资源。

>Failure to properly close resources will result in a resource leak which could bring first the application and then perhaps the box it's on to their knees.
如果不能正确地关闭资源，就会导致资源泄漏，这可能会首先导致应用程序，然后可能会导致应用程序崩溃。

### Noncompliant Code Example
```java
private void readTheFile() throws IOException {
  Path path = Paths.get(this.fileName);
  BufferedReader reader = Files.newBufferedReader(path, this.charset);
  // ...
  reader.close();  // Noncompliant
  // ...
  Files.lines("input.txt").forEach(System.out::println); // Noncompliant: The stream needs to be closed
}

private void doSomething() {
  OutputStream stream = null;
  try {
    for (String property : propertyList) {
      stream = new FileOutputStream("myfile.txt");  // Noncompliant
      // ...
    }
  } catch (Exception e) {
    // ...
  } finally {
    stream.close();  // Multiple streams were opened. Only the last is closed.
  }
}
```
### Compliant Solution
```java
private void readTheFile(String fileName) throws IOException {
    Path path = Paths.get(fileName);
    try (BufferedReader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
      reader.readLine();
      // ...
    }
    // ..
    try (Stream<String> input = Files.lines("input.txt"))  {
      input.forEach(System.out::println);
    }
}

private void doSomething() {
  OutputStream stream = null;
  try {
    stream = new FileOutputStream("myfile.txt");
    for (String property : propertyList) {
      // ...
    }
  } catch (Exception e) {
    // ...
  } finally {
    stream.close();
  }
}
```
### Exceptions
>Instances of the following classes are ignored by this rule because `close` has no effect:
以下类的实例将被此规则忽略，因为close没有效果

*   `java.io.ByteArrayOutputStream`
*   `java.io.ByteArrayInputStream`
*   `java.io.CharArrayReader`
*   `java.io.CharArrayWriter`
*   `java.io.StringReader`
*   `java.io.StringWriter`

>Java 7 introduced the `try-with-resources` statement, which implicitly closes `Closeables`. All resources opened in a `try-with-resources` statement are ignored by this rule.
Java 7引入了`try-with-resources`语句，它隐式地关闭关闭项。在`try-with-resources`语句中打开的所有资源都将被此规则忽略。

```java
try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {
  //...
}
catch ( ... ) {
  //...
}
```
### See
*   [MITRE, CWE-459](http://cwe.mitre.org/data/definitions/459.html) - Incomplete Cleanup
*   [CERT, FIO04-J.](https://www.securecoding.cert.org/confluence/x/9gFqAQ) - Release resources when they are no longer needed
*   [CERT, FIO42-C.](https://www.securecoding.cert.org/confluence/x/GAGQBw) - Close files when they are no longer needed
*   [Try With Resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)

### 一句话说明
- 打开的资源应该关闭并且放到finally块中进行关闭






## 97、Return values from functions without side effects should not be ignored

- 主要

>When the call to a function doesn't have any side effects, what is the point of making the call if the results are ignored? In such case, either the function call is useless and should be dropped or the source code doesn't behave as expected.
当调用一个函数没有任何副作用时，如果忽略了结果，那么调用这个函数又有什么意义呢?在这种情况下，要么函数调用无用，应该删除，要么源代码的行为不像预期的那样。

To prevent generating any false-positives, this rule triggers an issue only on the following predefined list of immutable classes in the Java API :
为了防止产生任何假阳性，这个规则只在Java API中预定义的不可变类列表上触发一个问题:

*   `java.lang.String`
*   `java.lang.Boolean`
*   `java.lang.Integer`
*   `java.lang.Double`
*   `java.lang.Float`
*   `java.lang.Byte`
*   `java.lang.Character`
*   `java.lang.Short`
*   `java.lang.StackTraceElement`
*   `java.time.DayOfWeek`
*   `java.time.Duration`
*   `java.time.Instant`
*   `java.time.LocalDate`
*   `java.time.LocalDateTime`
*   `java.time.LocalTime`
*   `java.time.Month`
*   `java.time.MonthDay`
*   `java.time.OffsetDateTime`
*   `java.time.OffsetTime`
*   `java.time.Period`
*   `java.time.Year`
*   `java.time.YearMonth`
*   `java.time.ZonedDateTime`
*   `java.math.BigInteger`
*   `java.math.BigDecimal`
*   `java.util.Optional`

>Methods can return values to communicate failure or success or to update local objects or fields. Security risks can arise when method return values are ignored or when the invoking method fails to take suitable action. Consequently, programs must not ignore method return values.
方法可以返回值来通信失败或成功，或更新本地对象或字段。当方法返回值被忽略或调用方法未能采取适当的操作时，可能会出现安全风险。因此，程序不能忽略方法返回值。

>When getter methods are named after an action, a programmer could fail to realize that a return value is expected. For example, the only purpose of the `ProcessBuilder.redirectErrorStream()` method is to report via return value whether the process builder successfully merged standard error and standard output. The method that actually performs redirection of the error stream is the overloaded single-argument method `ProcessBuilder.redirectErrorStream(boolean)`.
当getter方法以某个操作命名时，程序员可能无法意识到期望返回值。例如，ProcessBuilder.redirectErrorStream()方法的唯一目的是通过返回值报告process builder是否成功地合并了标准错误和标准输出。实际执行错误流重定向的方法是重载的单参数方法ProcessBuilder.redirectErrorStream(boolean)。

### Noncompliant Code Example
```java
public void handle(String command){
  command.toLowerCase(); // Noncompliant; result of method thrown away
  ...
}
public void deleteFile(){
  File someFile = new File("someFileName.txt");
  // Do something with someFile
  someFile.delete();
}
public class Replace {
  public static void main(String[] args) {
    String original = "insecure";
    original.replace('i', '9');
    System.out.println(original);
  }
}
```
### Compliant Solution
```java
public void handle(String command){
  String formattedCommand = command.toLowerCase();
  ...
}
public void deleteFile(){
  File someFile = new File("someFileName.txt");
  // Do something with someFile
  if (!someFile.delete()) {
    // Handle failure to delete the file
  }
}
public class Replace {
  public static void main(String[] args) {
    String original = "insecure";
    original = original.replace('i', '9');
    System.out.println(original);
  }
}
```
### Exceptions
>This rule will not raise an issue when both these conditions are met:
当这两个条件都满足时，这条规则不会产生问题:

*   The method call is in a `try` block with an associated `catch` clause.
方法调用位于try块中，并带有关联的catch子句
*   The method name starts with "parse", "format", "decode" or "valueOf" or the method is `String.getBytes(Charset)`.
方法名以“parse”、“format”、“decode”或“valueOf”开头，或者方法是String.getBytes(Charset)。

```java
private boolean textIsInteger(String textToCheck) {

    try {
        Integer.parseInt(textToCheck, 10); // OK
        return true;
    } catch (NumberFormatException ignored) {
        return false;
    }
}
```
### See
*   [CERT, EXP00-J.](https://www.securecoding.cert.org/confluence/x/9gEqAQ) - Do not ignore values returned by methods

### 一句话说明
- 操作对函数返回值没有影响的应该忽略






## 98、Servlets should not have mutable instance fields

- 主要

>By contract, a servlet container creates one instance of each servlet and then a dedicated thread is attached to each new incoming HTTP request to process the request. So all threads share the servlet instances and by extension their instance fields. To prevent any misunderstanding and unexpected behavior at runtime, all servlet fields should then be either `static` and/or `final`, or simply removed.
根据约定，servlet容器创建每个servlet的一个实例，然后将一个专用线程附加到每个新的传入HTTP请求上，以处理请求。因此，所有线程共享servlet实例，并扩展它们的实例字段。为了防止在运行时出现任何误解和意外行为，所有servlet字段都应该是静态的和/或final的，或者干脆删除。

>With Struts 1.X, the same constraint exists on `org.apache.struts.action.Action`.
使用Struts 1.x， org.apache.struts.action.Action上也存在相同的约束。

>Java servlets often must store information associated with each client that connects to them. Using member fields in the `javax.servlet.http.HttpServlet` to store information specific to individual clients is a common, simple practice. However, doing so is a mistake for the following reasons:
Java servlet通常必须存储与连接到它们的每个客户机相关联的信息。使用javax.servlet.http中的成员字段。HttpServlet存储特定于单个客户端的信息是一种常见的、简单的实践。然而，这样做是错误的，原因如下
*   In any Java servlet container, such as [Apache Tomcat](http://tomcat.apache.org/), `HttpServlet` is a singleton class (see [MSC07-J. Prevent multiple instantiations of singleton objects](https://wiki.sei.cmu.edu/confluence/display/java/MSC07-J.+Prevent+multiple+instantiations+of+singleton+objects) for information related to singleton classes). Therefore, there can be only one instance of member variables, even if they are not declared static.
在任何Java servlet容器中，比如Apache Tomcat, HttpServlet都是一个单例类(参见MSC07-J)。防止单例对象的多个实例化，以获取与单例类相关的信息)。因此，成员变量只能有一个实例，即使它们不是声明为静态的。
*   A servlet container is permitted to invoke the servlet from multiple threads. Consequently, accessing fields in the servlet can lead to [data races](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary).
允许servlet容器从多个线程调用servlet。因此，访问servlet中的字段可能导致数据竞争。
*   If two clients initiate sessions with the servlet, the servlet can leak information from one client to the other client.
如果两个客户机使用servlet发起会话，servlet可以将信息从一个客户机泄漏到另一个客户机。

>Java servlets provide an `HttpSession` class for storing session-specific data, which is encoded in each web request. Use of this class prevents both cross-session information leakage and data races.
Java servlet提供一个HttpSession类来存储特定于会话的数据，这些数据在每个web请求中编码。使用该类可以防止跨会话信息泄漏和数据竞争。

### Noncompliant Code Example
```java
public class MyServlet extends HttpServlet {
  private String userName;  //As this field is shared by all users, it's obvious that this piece of information should be managed differently
  ...
}
```
or
```java
public class MyAction extends Action {
  private String userName;  //Same reason
  ...
}
```
```java
public class SampleServlet extends HttpServlet {
 
  private String lastAddr = "nobody@nowhere.com";
 
  public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    response.setContentType("text/html");
    PrintWriter out = response.getWriter();
    out.println("<html>");
 
    String emailAddr = request.getParameter("emailAddr");
 
    if (emailAddr != null) {
      out.println("Email Address:");
      out.println(sanitize(emailAddr));
      out.println("<br>Previous Address:");
      out.println(sanitize(lastAddr));
    };
 
    out.println("<p>");
    out.print("<form action=\"");
    out.print("SampleServlet\" ");
    out.println("method=POST>");
    out.println("Parameter:");
    out.println("<input type=text size=20 name=emailAddr>");
    out.println("<br>");
    out.println("<input type=submit>");
    out.println("</form>");
 
    lastAddr = emailAddr;
  }
 
  public void doPost(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    doGet(request, response);
  }
 
  // Filter the specified message string for characters
  // that are sensitive in HTML.
  public static String sanitize(String message) {
    // ...
  }
}
```
```java
public class SampleServlet extends HttpServlet {
  
  private static String lastAddr = "nobody@nowhere.com";
  private static final Object lastAddrLock = new Object();
 
  public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    response.setContentType("text/html");
    PrintWriter out = response.getWriter();
    out.println("<html>");
  
    String emailAddr = request.getParameter("emailAddr");
  
    if (emailAddr != null) {
      out.println("Email Address::");
      out.println(sanitize(emailAddr));
      synchronized (lock) {
        out.println("<br>Previous Email Address::");
        out.println(sanitize(lastAddr));
      }
    };
  
    out.println("<p>");
    out.print("<form action=\"");
    out.print("SampleServlet\" ");
    out.println("method=POST>");
    out.println("Parameter:");
    out.println("<input type=text size=20 name=emailAddr>");
    out.println("<br>");
    out.println("<input type=submit>");
    out.println("</form>");
  
    synchronized (lock) {
      lastAddr = emailAddr;
    }
  }
  
  public void doPost(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    doGet(request, response);
  }
 
  // Filter the specified message string for characters
  // that are sensitive in HTML.
  public static String sanitize(String message) {
    // ...
  }
}
```
### Compliant Solution
```java
public class SampleServlet extends HttpServlet {
 
  public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    response.setContentType("text/html");
    PrintWriter out = response.getWriter();
    out.println("<html>");
 
    String emailAddr = request.getParameter("emailAddr");
    HttpSession session = request.getSession();
    Object attr = session.getAttribute("lastAddr");
    String lastAddr = (attr == null) ? "null" : attr.toString();
 
    if (emailAddr != null) {
      out.println("Email Address::");
      out.println(sanitize(emailAddr));
      out.println("<br>Previous Email Address::");
      out.println(sanitize(lastAddr));
    };
 
    out.println("<p>");
    out.print("<form action=\"");
    out.print("SampleServlet\" ");
    out.println("method=POST>");
    out.println("Parameter:");
    out.println("<input type=text size=20 name=emailAddr>");
    out.println("<br>");
    out.println("<input type=submit>");
    out.println("</form>");
 
    session.setAttribute("lastAddr", emailAddr);
  }
 
  public void doPost(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
    doGet(request, response);
  }
 
  // Filter the specified message string for characters
  // that are sensitive in HTML.
  public static String sanitize(String message) {
    // ...
  }
}
```
### See
*   [CERT, MSC11-J.](https://www.securecoding.cert.org/confluence/x/EYBUC) - Do not let session information leak within a servlet

### 一句话说明
- servlet容器对每一个servlet创建一个实例导致实例变量共享产生问题，struts1.x 也是单例

### 技术分析
[Servlet 单例多线程](https://www.cnblogs.com/yjhrem/articles/3160864.html "Servlet 单例多线程")







## 99、Silly equality checks should not be made

- 主要

>Comparisons of dissimilar types will always return false. The comparison and all its dependent code can simply be removed. This includes:
不同类型的比较总是返回false。可以简单地删除比较及其所有相关代码。这包括:

*   comparing an object with null
将对象与null进行比较
*   comparing an object with an unrelated primitive (E.G. a string with an int)
将对象与不相关的基础数据类型(例如带int的字符串)进行比较
*   comparing unrelated classes
比较不相关的类
*   comparing an unrelated `class` and `interface`
比较不相关的类和接口
*   comparing unrelated `interface` types
比较不相关的接口类型
*   comparing an array to a non-array
比较数组和非数组
*   comparing two arrays
比较两个数组

Specifically in the case of arrays, since arrays don't override `Object.equals()`, calling `equals` on two arrays is the same as comparing their addresses. This means that `array1.equals(array2)` is equivalent to `array1==array2`.
特别是在数组的情况下，因为数组不覆盖Object.equals()，所以在两个数组上调用equals与比较它们的地址是一样的。这意味着array1.equals(array2)等价于array1==array2。

However, some developers might expect `Array.equals(Object obj)` to do more than a simple memory address comparison, comparing for instance the size and content of the two arrays. Instead, the `==` operator or `Arrays.equals(array1, array2)` should always be used with arrays.
然而，一些开发人员可能期望数组`Array.equals(Object obj)`做的不仅仅是简单的内存地址比较，例如比较两个数组的大小和内容。而是==操作符或`Arrays.equals(array1, array2)`应该始终与数组一起使用。


>In Java, arrays are objects and support object methods such as `Object.equals()`. However, arrays do not support any methods besides those provided by `Object`. Consequently, using `Object.equals()` on any array compares only array _references_, not their _contents_. Programmers who wish to compare the contents of two arrays must use the static two-argument `Arrays.equals()` method. This method considers two arrays equivalent if both arrays contain the same number of elements, and all corresponding pairs of elements in the two arrays are equivalent, according to `Object.equals()`. In other words, two arrays are equal if they contain equivalent elements in the same order. To test for reference equality, use the reference equality operators, `== `and `!=`.  
在Java中，数组是对象，并且支持对象方法，比如object .equals()。但是，数组不支持对象提供的方法之外的任何方法。因此，在任何数组上使用Object.equals()只比较数组引用，而不比较它们的内容。希望比较两个数组内容的程序员必须使用静态双参数数组.equals()方法。根据Object.equals()，如果两个数组包含相同数量的元素，并且两个数组中所有对应的元素对都是相等的，则该方法认为两个数组是等价的。换句话说，如果两个数组包含相同顺序的等价元素，那么它们就是相等的。要测试引用是否相等，请使用引用相等运算符==和!=。

>Because the effect of using `Object.equals()` to compare two arrays is often misconstrued as content equality, and because a better alternative exists in the use of reference equality operators, the use of the `Object.equals()` method to compare two arrays is disallowed.
由于使用Object.equals()来比较两个数组的效果常常被误解为内容相等，而且由于在使用引用相等操作符时存在更好的替代方法，因此不允许使用Object.equals()方法来比较两个数组。

### Noncompliant Code Example
```java
interface KitchenTool { ... };
interface Plant {...}

public class Spatula implements KitchenTool { ... }
public class Tree implements Plant { ...}
//...

Spatula spatula = new Spatula();
KitchenTool tool = spatula;
KitchenTool [] tools = {tool};

Tree tree = new Tree();
Plant plant = tree;
Tree [] trees = {tree};


if (spatula.equals(tree)) { // Noncompliant; unrelated classes
  // ...
}
else if (spatula.equals(plant)) { // Noncompliant; unrelated class and interface
  // ...
}
else if (tool.equals(plant)) { // Noncompliant; unrelated interfaces
  // ...
}
else if (tool.equals(tools)) { // Noncompliant; array & non-array
  // ...
}
else if (trees.equals(tools)) {  // Noncompliant; incompatible arrays
  // ...
}
else if (tree.equals(null)) {  // Noncompliant
  // ...
}
```
```java
int[] arr1 = new int[20]; // Initialized to 0
int[] arr2 = new int[20]; // Initialized to 0
System.out.println(arr1.equals(arr2)); // Prints false
```

### Compliant Solution
```java
int[] arr1 = new int[20]; // Initialized to 0
int[] arr2 = new int[20]; // Initialized to 0
System.out.println(Arrays.equals(arr1, arr2)); // Prints true
```
```java
int[] arr1 = new int[20]; // Initialized to 0
int[] arr2 = new int[20]; // Initialized to 0
System.out.println(arr1 == arr2); // Prints false
```

### See
*   [CERT, EXP02-J.](https://www.securecoding.cert.org/confluence/x/IQAlAg) - Do not use the Object.equals() method to compare two arrays

### 一句话说明
- 不应该做愚蠢的相等检查






## 100、Strings and Boxed types should be compared using "equals()"

- 主要

>It's almost always a mistake to compare two instances of `java.lang.String` or boxed types like `java.lang.Integer` using reference equality `==` or `!=`, because it is not comparing actual value but locations in memory.


The _values_ of boxed primitives cannot be directly compared using the `==` and `!=` operators because these operators compare object references rather than object values. Programmers can find this behavior surprising because autoboxing [memoizes](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-memoization), or caches, the values of some primitive variables. Consequently, reference comparisons and value comparisons produce identical results for the subset of values that are memoized.
不能使用==和!=操作符直接比较装箱原语的值，因为这些操作符比较的是对象引用，而不是对象值。程序员会发现这种行为令人惊讶，因为自动装箱会记忆或缓存一些基本变量的值。因此，引用比较和值比较对于记忆的值子集产生相同的结果。

Autoboxing automatically wraps a value of a primitive type with the corresponding wrapper object. _The Java Language Specification_ (JLS), [§5.1.7, "Boxing Conversion"](http://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.7)[ [](http://java.sun.com/docs/books/jls/third_edition/html/conversions.html#5.1.7)[JLS 2015](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-JLS05)[], explains which primitive values are memoized during autoboxing:](http://java.sun.com/docs/books/jls/third_edition/html/conversions.html#5.1.7)
自动装箱用相应的包装器对象自动包装基本类型的值。Java语言规范(JLS)，§5.1.7，“装箱转换”[JLS 2015]，解释了哪些原始值在自动装箱过程中被记忆:

> If the value `p` being boxed is `true`, `false`, a `byte`, a `char` in the range `\u0000` to `\u007f`, or an `int` or `short` number between `-128` and `127`, then let `r1` and `r2` be the results of any two boxing conversions of `p`. It is always the case that `r1 == r2`.
如果该值p盒装是真的,假的,一个字节,一个字符范围内\ \ u007f u0000, int或短数在-128年至127年之间,然后让r1和r2的任意两个拳击转换结果p。总是这样,r1, r2 = =。


Use of the `==` and `!=` operators for comparing the values of fully memoized boxed primitive types is permitted.
允许使用==和!=操作符来比较完全记忆的盒装原始类型的值。

Use of the `==` and `!=` operators for comparing the _values_ of boxed primitive types that are not fully memoized is permitted only when the range of values represented is guaranteed to be within the ranges specified by the JLS to be fully memoized.
只有在保证所表示的值的范围在要完全记忆的JLS指定的范围内时，才允许使用==和!=操作符来比较未完全记忆的装箱原始类型的值。

Use of the `==` and `!=` operators for comparing the _values_ of boxed primitive types is not allowed in all other cases.
在所有其他情况下都不允许使用==和!=操作符来比较装箱的基本类型的值。

Note that Java Virtual Machine (JVM) implementations are allowed, but not required, to memoize additional values [[JLS 2015](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-JLS05)]:
注意，允许但不要求Java虚拟机(JVM)实现存储额外的值[JLS 2015]:

> Less memory-limited implementations could, for example, cache all characters and shorts, as well as integers and longs in the range of −32K to +32K. ([§5.1.7](http://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.7))
例如，内存限制较少的实现可以缓存所有字符和short，以及范围在- 32K到+32K之间的整数和long。(§5.1.7)

Code that depends on implementation-defined behavior is nonportable. It is permissible to depend on implementation-specific ranges of memoized values provided that all targeted implementations support these greater ranges.
依赖于实现定义行为的代码是不可移植的。如果所有目标实现都支持这些较大的范围，则允许依赖于特定于实现的记忆值范围。

Java Collections contain only objects; they cannot contain primitive types. Further, the type parameters of all Java generics must be object types rather than primitive types. That is, attempting to declare an `ArrayList<int>` (which, presumably, would contain values of type `int`) fails at compile time because type `int` is not an object type. The appropriate declaration would be `ArrayList<Integer>`, which makes use of the wrapper classes and autoboxing.
Java集合只包含对象;它们不能包含原始类型。此外，所有Java泛型的类型参数必须是对象类型，而不是基本类型。也就是说，试图声明ArrayList<int>(它可能包含int类型的值)在编译时失败，因为int类型不是对象类型。适当的声明应该是ArrayList<Integer>，它使用包装器类和自动装箱。

This noncompliant code example attempts to count the number of indices in arrays `list1` and `list2` that have equivalent values. Recall that class `Integer` is required to [memoize](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-memoization) only those integer values in the range −128 to 127; it might return a nonunique object for any value outside that range. Consequently, when comparing autoboxed integer values outside that range, the `==` operator might return false and the example could deceptively output 0.
这个不兼容的代码示例试图计算数组list1和list2中具有相同值的索引的数量。回想一下，类Integer只需要记忆范围在- 128到127之间的那些整数值;对于该范围之外的任何值，它都可能返回一个非惟一对象。因此，当比较超出该范围的自装箱整数值时，==操作符可能返回false，而示例可能欺骗性地输出0。

### Noncompliant Code Example
```java
String firstName = getFirstName(); // String overrides equals
String lastName = getLastName();

if (firstName == lastName) { ... }; // Non-compliant; false even if the strings have the same value
```
```java
import java.util.Comparator;
static Comparator<Integer> cmp = new Comparator<Integer>() {
  public int compare(Integer i, Integer j) {
    return i < j ? -1 : (i == j ? 0 : 1);
  }
};
```
```java
public class Wrapper {
  public static void main(String[] args) {
    Integer i1 = 100;
    Integer i2 = 100;
    Integer i3 = 1000;
    Integer i4 = 1000;
    System.out.println(i1 == i2);
    System.out.println(i1 != i2);
    System.out.println(i3 == i4);
    System.out.println(i3 != i4);
  }
}
```
```java
public class Wrapper {
  public static void main(String[] args) {
    // Create an array list of integers, where each element
    // is greater than 127
    ArrayList<Integer> list1 = new ArrayList<Integer>();
    for (int i = 0; i < 10; i++) {
      list1.add(i + 1000);
    }
 
    // Create another array list of integers, where each element
    // has the same value as the first list
    ArrayList<Integer> list2 = new ArrayList<Integer>();
    for (int i = 0; i < 10; i++) {
      list2.add(i + 1000);
    }
 
    // Count matching values
    int counter = 0;
    for (int i = 0; i < 10; i++) {
      if (list1.get(i) == list2.get(i)) {  // Uses '=='
        counter++;
      }
    }
 
    // Print the counter: 0 in this example
    System.out.println(counter);
  }
 
}
```
```java
public void exampleEqualOperator(){
  Boolean b1 = new Boolean("true");
  Boolean b2 = new Boolean("true");
 
  if (b1 == b2) {    // Never equal
    System.out.println("Never printed");
  }
}
```
### Compliant Solution
```java
String firstName = getFirstName();
String lastName = getLastName();

if (firstName != null && firstName.equals(lastName)) { ... };
```
```java
import java.util.Comparator;
static Comparator<Integer> cmp = new Comparator<Integer>() {
  public int compare(Integer i, Integer j) {
    return i < j ? -1 : (i > j ? 1 : 0) ;
  }
};
```
```java
public class Wrapper {
  public static void main(String[] args) {
    Integer i1 = 100;
    Integer i2 = 100;
    Integer i3 = 1000;
    Integer i4 = 1000;
    System.out.println(i1.equals(i2));
    System.out.println(!i1.equals(i2));
    System.out.println(i3.equals(i4));
    System.out.println(!i3.equals(i4));
  }
}
```
```java
public class Wrapper {
  public static void main(String[] args) {
    // Create an array list of integers
    ArrayList<Integer> list1 = new ArrayList<Integer>();
 
    for (int i = 0; i < 10; i++) {
      list1.add(i + 1000);
    }
 
    // Create another array list of integers, where each element
    // has the same value as the first one
    ArrayList<Integer> list2 = new ArrayList<Integer>();
    for (int i = 0; i < 10; i++) {
      list2.add(i + 1000);
    }
  
    // Count matching values
    int counter = 0;
    for (int i = 0; i < 10; i++) {
      if (list1.get(i).equals(list2.get(i))) {  // Uses 'equals()'
        counter++;
      }
    }
  
    // Print the counter: 10 in this example
    System.out.println(counter);
  }
}
```
```java
public void exampleEqualOperator(){
  Boolean b1 = true;
  Boolean b2 = true;
     
  if (b1 == b2) {   // Always equal
    System.out.println("Always printed");
  }
  
  b1 = Boolean.TRUE;
  if (b1 == b2) {   // Always equal
    System.out.println("Always printed");
  }
}
```
### See
*   [MITRE, CWE-595](http://cwe.mitre.org/data/definitions/595.html) - Comparison of Object References Instead of Object Contents
*   [MITRE, CWE-597](http://cwe.mitre.org/data/definitions/597.html) - Use of Wrong Operator in String Comparison
*   [CERT, EXP03-J.](https://www.securecoding.cert.org/confluence/x/wwD1AQ) - Do not use the equality operators when comparing values of boxed primitives
*   [CERT, EXP50-J.](https://www.securecoding.cert.org/confluence/x/8AEqAQ) - Do not confuse abstract object equality with reference equality

### 一句话说明
- 字符串和装箱类型应该使用“equals()”进行比较








## 101、Synchronization should not be based on Strings or boxed primitives

- 主要

>Objects which are pooled and potentially reused should not be used for synchronization. If they are, it can cause unrelated threads to deadlock with unhelpful stacktraces. Specifically, `String` literals, and boxed primitives such as Integers should not be used as lock objects because they are pooled and reused. The story is even worse for `Boolean` objects, because there are only two instances of `Boolean`, `Boolean.TRUE` and `Boolean.FALSE` and every class that uses a Boolean will be referring to one of the two.
被池化和可能被重用的对象不应用于同步。如果是，则会导致不相关的线程死锁，并使用无用的堆栈跟踪。具体来说，字符串文本和整数等装箱原语不应该用作锁对象，因为它们是池化和重用的。布尔对象的情况更糟，因为布尔对象只有两个实例。真正的和布尔。FALSE，每个使用布尔值的类都将引用这两个类中的一个。

>Misuse of synchronization primitives is a common source of concurrency issues. Synchronizing on objects that may be reused can result in deadlock and nondeterministic behavior. Consequently, programs must never synchronize on objects that may be reused.
误用同步原语是并发性问题的常见来源。对可重用的对象进行同步可能会导致死锁和不确定性行为。因此，程序永远不能同步可能重用的对象。

### Noncompliant Code Example
```java
private static final Boolean bLock = Boolean.FALSE;
private static final Integer iLock = Integer.valueOf(0);
private static final String sLock = "LOCK";

public void doSomething() {

  synchronized(bLock) {  // Noncompliant
    // ...
  }
  synchronized(iLock) {  // Noncompliant
    // ...
  }
  synchronized(sLock) {  // Noncompliant
    // ...
  }
```
>The `Boolean` type is unsuitable for locking purposes because it allows only two values: true and false. Boolean literals containing the same value share unique instances of the `Boolean` class in the Java Virtual Machine (JVM). In this example, `initialized` refers to the instance corresponding to the value `Boolean.FALSE`. If any other code were to inadvertently synchronize on a `Boolean` literal with this value, the lock instance would be reused and the system could become unresponsive or could deadlock.
布尔类型不适合用于锁定，因为它只允许两个值:true和false。包含相同值的布尔文字在Java虚拟机(JVM)中共享布尔类的惟一实例。在本例中，initialized引用与值Boolean.FALSE对应的实例。如果任何其他代码无意中在布尔值上与此值同步，那么lock实例将被重用，系统可能变得无响应或死锁。

```java
private final Boolean initialized = Boolean.FALSE;
public void doSomething() {
  synchronized (initialized) {
    // ...
  }
}
```
>Boxed types may use the same instance for a range of integer values; consequently, they suffer from the same reuse problem as `Boolean` constants. The wrapper object are reused when the value can be represented as a byte; JVM implementations are also permitted to reuse wrapper objects for larger ranges of values. While use of the intrinsic lock associated with the boxed `Integer` wrapper object is insecure; instances of the `Integer` object constructed using the `new` operator (`new Integer(value)`) are unique and not reused. In general, locks on any data type that contains a boxed value are insecure.
装箱类型可以对整数值范围使用相同的实例;因此，它们与布尔常量面临相同的重用问题。当值可以表示为字节时，包装器对象被重用;JVM实现还允许为更大范围的值重用包装器对象。而使用与已装箱整数包装器对象关联的内部锁是不安全的;使用new操作符(new Integer(value))构造的Integer对象的实例是惟一的，不能重用。通常，任何包含已装箱值的数据类型上的锁都是不安全的。

```java
private int count = 0;
private final Integer Lock = count; // Boxed primitive Lock is shared
public void doSomething() {
  synchronized (Lock) {
    count++;
    // ...
  }
}
```
```java
private final String lock = new String("LOCK").intern();
 
public void doSomething() {
  synchronized (lock) {
    // ...
  }
}
```
```java
// This bug was found in jetty-6.1.3 BoundedThreadPool
private final String lock = "LOCK";
 
public void doSomething() {
  synchronized (lock) {
    // ...
  }
}
```
### Compliant Solution
```java
private static final Object lock1 = new Object();
private static final Object lock2 = new Object();
private static final Object lock3 = new Object();

public void doSomething() {

  synchronized(lock1) {
    // ...
  }
  synchronized(lock2) {
    // ...
  }
  synchronized(lock3) {
    // ...
  }
```
>When explicitly constructed, an `Integer` object has a unique reference and its own intrinsic lock that is distinct not only from other `Integer` objects, but also from boxed integers that have the same value. While this is an acceptable solution, it can cause maintenance problems because developers can incorrectly assume that boxed integers are also appropriate lock objects. A more appropriate solution is to synchronize on a private final lock object as described in the final compliant solution for this rule.
当显式构造整数对象时，它有一个惟一的引用和它自己的内部锁，它不仅与其他整数对象不同，而且与具有相同值的装箱整数也不同。虽然这是一个可接受的解决方案，但它可能会导致维护问题，因为开发人员可能错误地假定装箱的整数也是适当的锁对象。更合适的解决方案是对私有final lock对象进行同步，如该规则的最终兼容解决方案中所述。

```java
private int count = 0;
private final Integer Lock = new Integer(count);
 
public void doSomething() {
  synchronized (Lock) {
    count++;
    // ...
  }
}
```
>A `String` instance differs from a `String` literal. The instance has a unique reference and its own intrinsic lock that is distinct from other `String` object instances or literals. Nevertheless, a better approach is to synchronize on a private final lock object, as shown in the following compliant solution.
字符串实例与字符串文字不同。实例有一个惟一的引用和它自己的固有锁，这与其他字符串对象实例或字面值不同。不过，更好的方法是在私有的final lock对象上同步，如下面的兼容解决方案所示。

```java
private final String lock = new String("LOCK");
 
public void doSomething() {
  synchronized (lock) {
    // ...
  }
}
```
```java
private final Object lock = new Object();
 
public void doSomething() {
  synchronized (lock) {
    // ...
  }
}
```
### See
*   [CERT, LCK01-J.](https://www.securecoding.cert.org/confluence/x/rQGeAQ) - Do not synchronize on objects that may be reused

### 一句话说明
- 字符串和封箱类不应该被用作锁定对象，因为它们被合并和重用。





## 102、The non-serializable super class of a "Serializable" class should have a no-argument constructor

- 次要

>When a `Serializable` object has a non-serializable ancestor in its inheritance chain, object deserialization (re-instantiating the object from file) starts at the first non-serializable class, and proceeds down the chain, adding the properties of each subsequent child class, until the final object has been instantiated.
当一个可序列化的对象有non-serializable祖先继承链,反序列化对象(对象池从文件)开始在第一个non-serializable类,和资金链,增加每个随后的子类的属性,直到最后一个对象被实例化。

>In order to create the non-serializable ancestor, its no-argument constructor is called. Therefore the non-serializable ancestor of a `Serializable` class must have a no-arg constructor. Otherwise the class is `Serializable` but not deserializable.
为了创建不可序列化的祖先，调用其无参数构造函数。因此，可序列化类的不可序列化祖先必须有一个无arg构造函数。否则，该类是可序列化的，但不可反序列化。

### Noncompliant Code Example
```java
public class Fruit {
  private Season ripe;

  public Fruit (Season ripe) {...}
  public void setRipe(Season ripe) {...}
  public Season getRipe() {...}
}

public class Raspberry extends Fruit
        implements Serializable {  // Noncompliant; nonserializable ancestor doesn't have no-arg constructor
  private static final long serialVersionUID = 1;

  private String variety;

  public Raspberry(Season ripe, String variety) { ...}
  public void setVariety(String variety) {...}
  public String getVarity() {...}
}
```
### Compliant Solution
```java
public class Fruit {
  private Season ripe;

  public Fruit () {...};  // Compliant; no-arg constructor added to ancestor
  public Fruit (Season ripe) {...}
  public void setRipe(Season ripe) {...}
  public Season getRipe() {...}
}

public class Raspberry extends Fruit
        implements Serializable {
  private static final long serialVersionUID = 1;

  private String variety;

  public Raspberry(Season ripe, String variety) {...}
  public void setVariety(String variety) {...}
  public String getVarity() {...}
}
```
### 一句话说明
- “可序列化”类的不可序列化超类应该具有无参数构造函数






## 103、The Object.finalize() method should not be called

- 主要

>According to the official javadoc documentation, this `Object.finalize()` is called by the garbage collector on an object when garbage collection determines that there are no more references to the object. Calling this method explicitly breaks this contract and so is misleading.
根据官方javadoc文档，当垃圾收集确定不再引用对象时，对象上的垃圾收集器将调用`Object.finalize()`。显式地调用此方法会破坏此契约，因此具有误导性。

### Noncompliant Code Example
```java
public void dispose() throws Throwable {
  this.finalize();                       // Noncompliant
}
```
### Compliant Solution
```java
public class Watermelon implements Serializable {
  // ...
  private void writeObject(java.io.ObjectOutputStream out)
        throws IOException
  {...}

  private void readObject(java.io.ObjectInputStream in)
        throws IOException, ClassNotFoundException
  {...}

  private void readObjectNoData()
        throws ObjectStreamException
  {...}

  protected Object readResolve() throws ObjectStreamException
  {...}

  private Object writeReplace() throws ObjectStreamException
  {...}
```
### See
*   [MITRE, CWE-586](http://cwe.mitre.org/data/definitions/586.html) - Explicit Call to Finalize()
*   [CERT, MET12-J.](https://www.securecoding.cert.org/confluence/x/H4cbAQ) - Do not use finalizers

### 一句话说明
- 不应该调用Object.finalize()方法






## 104、The signature of "finalize()" should match that of "Object.finalize()"

- 严重

>`Object.finalize()` is called by the Garbage Collector at some point after the object becomes unreferenced.
finalize()由垃圾收集器在对象未被引用之后的某个时候调用。

In general, overloading `Object.finalize()` is a bad idea because:
通常，重载Object.finalize()不是一个好主意，因为:
*   The overload may not be called by the Garbage Collector.
垃圾收集器可能不会调用重载。
*   Users are not expected to call `Object.finalize()` and will get confused.
不希望用户调用Object.finalize()，这会让他们感到困惑。

But beyond that it's a terrible idea to name a method "finalize" if it doesn't actually override `Object.finalize()`.
但是除此之外，如果一个方法实际上没有覆盖Object.finalize()，那么将它命名为“finalize”是一个糟糕的主意。

### Noncompliant Code Example
```java
public int finalize(int someParameter) {        // Noncompliant
  /* ... */
}
```
### Compliant Solution
```java
public int someBetterName(int someParameter) {  // Compliant
  /* ... */
}
```
### 一句话说明
- 不要重写Object.finalize()






## 105、The value returned from a stream read should be checked

- 次要

>You cannot assume that any given stream reading call will fill the `byte[]` passed in to the method. Instead, you must check the value returned by the read method to see how many bytes were read. Fail to do so, and you introduce bug that is both harmful and difficult to reproduce.
您不能假定任何给定的流读取调用都将填充传入方法的byte[]。相反，您必须检查read方法返回的值，以查看读取了多少字节。如果做不到这一点，就会引入有害且难以繁殖的bug。

>Similarly, you cannot assume that `InputStream.skip` will actually skip the requested number of bytes, but must check the value returned from the method.
类似地，您不能假定`InputStream.skip`实际上会跳过请求的字节数，但必须检查方法返回的值。

>This rule raises an issue when an `InputStream.read` method that accepts a `byte[]` is called, but the return value is not checked, and when the return value of `InputStream.skip` is not checked. The rule also applies to `InputStream` child classes.
这个规则在`InputStream`时引发了一个问题。调用read方法，该方法接受一个byte[]，但不检查返回值，当InputStream返回值时。未选中跳过。该规则也适用于`InputStream`子类。

### Noncompliant Code Example
```java
public void doSomething(String fileName) {
  try {
    InputStream is = new InputStream(file);
    byte [] buffer = new byte[1000];
    is.read(buffer);  // Noncompliant
    // ...
  } catch (IOException e) { ... }
}
```
### Compliant Solution
```java
public void doSomething(String fileName) {
  try {
    InputStream is = new InputStream(file);
    byte [] buffer = new byte[1000];
    int count = 0;
    while (count = is.read(buffer) > 0) {
      // ...
    }
  } catch (IOException e) { ... }
}
```
### See
*   [CERT, FIO10-J.](https://www.securecoding.cert.org/confluence/x/XACSAQ) - Ensure the array is filled when using read() to fill an array

### 一句话说明
- 从流中读取的值应先检查再操作






## 106、Unary prefix operators should not be repeated

- 主要

>The needless repetition of an operator is usually a typo. There is no reason to write `!!!i` when `!i` will do.
操作员不必要的重复通常是打字错误。没有理由写!!我什么时候来，我会的。

>On the other hand, the repetition of increment and decrement operators may have been done on purpose, but doing so obfuscates the meaning, and should be simplified.
另一方面，递增和递减运算符的重复可能是故意的，但是这样做会混淆意思，应该简化。

>This rule raises an issue for sequences of: `!`, `~`, `-`, and `+`.

### Noncompliant Code Example
```java
int i = 1;

int j = - - -i;  // Noncompliant; just use -i
int k = ~~~i;    // Noncompliant; same as i
int m = + +i;    // Noncompliant; operators are useless here

boolean b = false;
boolean c = !!!b;   // Noncompliant
```
### Compliant Solution
```java
int i =  1;

int j = -i;
int k = ~i;
int m =  i;

boolean b = false;
boolean c = !b;
```
### Exceptions
Overflow handling for GWT compilation using ~~ is ignored.
使用~~进行GWT编译的溢出处理将被忽略。

### 一句话说明
- 不应重复使用一元前缀操作符





## 107、Value-based classes should not be used for locking

- 主要

According to the documentation,

> A program may produce unpredictable results if it attempts to distinguish two references to equal values of a value-based class, whether directly via reference equality or indirectly via an appeal to synchronization...
如果程序试图区分对基于值的类的相等值的两个引用，无论是直接通过引用相等还是间接通过请求同步，都可能产生不可预测的结果。

This is because value-based classes are intended to be wrappers for value types, which will be primitive-like collections of data (similar to `struct`s in other languages) that will come in future versions of Java.
这是因为基于值的类旨在成为值类型的包装器，这些值类型将是将来Java版本中类似于原始数据集合(类似于其他语言中的结构)的数据集合。

Instances of a value-based class ...
基于值的类的实例…
> *   do not have accessible constructors, but are instead instantiated through factory methods which make no committment as to the identity of returned instances;
没有可访问的构造函数，而是通过工厂方法实例化，这些工厂方法不指定返回实例的标识;

Which means that you can't be sure you're the only one trying to lock on any given instance of a value-based class, opening your code up to contention and deadlock issues.
这意味着您不能确定您是惟一一个试图锁定基于值的类的任何给定实例的人，这会导致您的代码出现争用和死锁问题。

Under Java 8 breaking this rule may not actually break your code, but there are no guarantees of the behavior beyond that.
在Java 8中，违反这条规则实际上可能不会破坏您的代码，但除此之外没有任何行为的保证。

This rule raises an issue when a known value-based class is used for synchronization. That includes all the classes in the `java.time` package except `Clock`; the date classes for alternate calendars, `HijrahDate`, `JapaneseDate`, `MinguoDate`, `ThaiBuddhistDate`; and the optional classes: `Optional`, `OptionalDouble`, `OptionalLong`, `OptionalInt`.
当使用已知的基于值的类进行同步时，此规则会引发问题。这包括java中的所有类。除时钟外的时间包;交替日历的日期类，HijrahDate, JapaneseDate, MinguoDate, ThaiBuddhistDate;以及可选类:optional, OptionalDouble, OptionalLong, OptionalInt。

**Note** that this rule is automatically disabled when the project's `sonar.java.source` is lower than `8`.
注意，当项目的sonar.java时，将自动禁用此规则。`sonar.java.source`小于8。

### Noncompliant Code Example
```java
Optional<Foo> fOpt = doSomething();
synchronized (fOpt) {  // Noncompliant
  // ...
}
```
### See
*   [Value-based classes](http://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html)

### 一句话说明
- 基于值的类不应该用于锁定







## 108、Values should not be uselessly incremented

- 主要

>A value that is incremented or decremented and then not stored is at best wasted code and at worst a bug.
递增或递减而不存储的值充其量是浪费代码，最坏的情况是错误。

### Noncompliant Code Example
```java
public int pickNumber() {
  int i = 0;
  int j = 0;

  i = i++; // Noncompliant; i is still zero

  return j++; // Noncompliant; 0 returned
}
```
### Compliant Solution
```java
public int pickNumber() {
  int i = 0;
  int j = 0;

  i++;
  return ++j;
}
```
### See

- [CERT, SER01-J.](https://www.securecoding.cert.org/confluence/x/4gAMAg) - Do not deviate from the proper signatures of serialization methods

### 一句话说明
- 自定义类序列化方法签名应该合法






## 109、Variables should not be self-assigned

- 主要

>There is no reason to re-assign a variable to itself. Either this statement is redundant and should be removed, or the re-assignment is a mistake and some other value or variable was intended for the assignment instead.
没有理由将变量重新分配给它自己。要么这个语句是冗余的，应该删除，要么重赋是一个错误，而其他一些值或变量本来是用于赋值的。

### Noncompliant Code Example
```java
public void setName(String name) {
  name = name;
}
```
### Compliant Solution
```java
public void setName(String name) {
  this.name = name;
}
```
### See
*   [CERT, MSC12-C.](https://www.securecoding.cert.org/confluence/x/NYA5) - Detect and remove code that has no effect or is never executed

### 一句话说明
- 变量不应该自赋值







## 110、Week Year ("YYYY") should not be used for date formatting

- 主要

>Few developers are aware of the difference between `Y` for "Week year" and `y` for Year when formatting and parsing a date with `SimpleDateFormat`. That's likely because for most dates, Week year and Year are the same, so testing at any time other than the first or last week of the year will yield the same value for both `y` and `Y`. But in the last week of December and the first week of January, you may get unexpected results.
在使用`SimpleDateFormat`格式化和解析日期时，很少有开发人员知道“Week year”的Y和year的Y之间的区别。这可能是因为对于大多数日期,星期,每年都是一样的,所以测试以外的任何时候第一或最后一周年将产生相同的值的`y`和`Y`。但是在上周12月和1月的第一个星期,你可能会得到意想不到的结果。

According to the [Javadoc](http://docs.oracle.com/javase/8/docs/api/java/util/GregorianCalendar.html#week_year):

> A week year is in sync with a WEEK_OF_YEAR cycle. All weeks between the first and last weeks (inclusive) have the same week year value. Therefore, the first and last days of a week year may have different calendar year values.
一个week year与一个WEEK_OF_YEAR循环同步。第一周和最后一周(包括)之间的所有周都具有相同的周年值。因此，一周一年的第一天和最后一天可能具有不同的日历年值。

> For example, January 1, 1998 is a Thursday. If getFirstDayOfWeek() is MONDAY and getMinimalDaysInFirstWeek() is 4 (ISO 8601 standard compatible setting), then week 1 of 1998 starts on December 29, 1997, and ends on January 4, 1998\. The week year is 1998 for the last three days of calendar year 1997\. If, however, getFirstDayOfWeek() is SUNDAY, then week 1 of 1998 starts on January 4, 1998, and ends on January 10, 1998; the first three days of 1998 then are part of week 53 of 1997 and their week year is 1997.
例如，1998年1月1日是星期四。如果getFirstDayOfWeek()是星期一，getMinimalDaysInFirstWeek()是4 (ISO 8601标准兼容设置)，那么1998年的第1周从1997年12月29日开始，到1998年1月4日结束。1997年日历年的最后三天是1998年。但是，如果getFirstDayOfWeek()是星期天，则1998年的第1周从1998年1月4日开始，到1998年1月10日结束;1998年的头三天是1997年第53周的一部分，他们的周是1997年。


### Noncompliant Code Example
```java
Date date = new SimpleDateFormat("yyyy/MM/dd").parse("2015/12/31");
String result = new SimpleDateFormat("YYYY/MM/dd").format(date);   //Noncompliant; yields '2016/12/31'
```
### Compliant Solution
```java
Date date = new SimpleDateFormat("yyyy/MM/dd").parse("2015/12/31");
String result = new SimpleDateFormat("yyyy/MM/dd").format(date);   //Yields '2015/12/31' as expected
```
### Exceptions
```java
Date date = new SimpleDateFormat("yyyy/MM/dd").parse("2015/12/31");
String result = new SimpleDateFormat("YYYY-ww").format(date);  //compliant, 'Week year' is used along with 'Week of year'. result = '2016-01'
```
### See

- [CERT, SER01-J.](https://www.securecoding.cert.org/confluence/x/4gAMAg) - Do not deviate from the proper signatures of serialization methods

### 一句话说明
- 日期格式不允许使用年(“YYYY”)





## 111、Zero should not be a possible denominator

- 严重

>If the denominator to a division or modulo operation is zero it would result in a fatal error.
如果除法或模数运算的分母为零，则会导致致命的错误。

### Noncompliant Code Example
```java
void test_divide() {
  int z = 0;
  if (unknown()) {
    // ..
    z = 3;
  } else {
    // ..
  }
  z = 1 / z; // Noncompliant, possible division by zero
}
```
### Compliant Solution
```java
void test_divide() {
  int z = 0;
  if (unknown()) {
    // ..
    z = 3;
  } else {
    // ..
    z = 1;
  }
  z = 1 / z;
}
```
### See
*   [MITRE, CWE-369](https://cwe.mitre.org/data/definitions/369.html) - Divide by zero
*   [CERT, NUM02-J.](https://www.securecoding.cert.org/confluence/x/KAGyAw) - Ensure that division and remainder operations do not result in divide-by-zero errors
*   [CERT, INT33-C.](https://www.securecoding.cert.org/confluence/x/cAI) - Ensure that division and remainder operations do not result in divide-by-zero errors

### 一句话说明
- 零不应该是一个可能的分母






































































