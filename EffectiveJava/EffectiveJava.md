
<h1>目录</h1>

[TOC]


# 第一张、创建和销毁对象

## 1.静态工厂代替构造器

### 提供一个公有静态方法（工厂方法），返回类的实例
```java
public static Boolean valueOf(boolean b) {
    return b?Boolean.TRUE : Boolean.FALSE;
}
```
### 与构造器相比优点与缺点<br>
    1. 有名称,用`BigInteger.probablePrime`代替`new BigInteger`更加清楚
    2. 不必每次都创建一个新对象。在不可变类等情况中可以反复利用一个对象。在第一点中说明了这项技术：从来不创建对象。
    3. 可以返回原返回类型任何子类对象。
    4. 返回的对象的类可以随每次调用而变化，取决于静态工厂方法的参数。
    5. 利于接口化开发，例如开发时双发定义了一个Service接口，使用该服务的程序员只需要使用Service.getService()即获得服务，服务内部如何实现不需要关心。
### 惯用名称
    1. from
    2. of
    3. valueOf
    4. instance/getInstance
    5. create/newInstance
    6. getType
    7. newType
    8. type例如Collections.list()

## 2.多个构造器参数考虑使用构建器

### 示例
```java
return new Resp.Builder.code("200").build;
```

## 3.私有构造器或枚举类型强化Sinslgeton属性

### 示例
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        // 防止使用反射创建第二个实例
        if(INSTANCE != null) {
            throw new Exception();
        }
    }
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```
### 实现Singleton三种方法
    1. 使用私有构造器
    2. 公有成员是个静态工厂方法如1中的代码
    3. 声明一个包含单个元素的枚举类型,如果必须拓展一个超类而不是enum时则不宜使用
    ```java
    public enum Elvis {
        INSTANCE;
        ...
    }
    ```


## 4.通过私有构造器强化不可实例化的能力
### 示例
当设计工具类时使用私有构造器防止被实例化
### 为什么不使用抽象类
可以被继承然后被实例化，这样甚至会误导用户这个类是专门为了继承设计的

## 5.优先考虑依赖注入来引用资源
### 示例
```java
new SpellChecker(dictionary);
```

## 6.避免创建不必要对象
### 示例
1. 使用`Sring s = "bulublu";`代替`String s = new String("bulublu");`
2. 正则表达式中抽象出Pattern实例，例如`s.matches("[\\w]")`中抽象出Pattern`Pattern pattern = Pattern.compile("[\\w]");`多次复用
3. 尽量使用基本类型可以减少创建对象数量

    
## 7.消除过期的对象引用

### 数组中的对象 存在过期引用
垃圾回收机制不仅不会处理这个对象而且也不会处理这个对象引用的其他对象。<br>将数组引用清空即可`elements[i] = null`<br>一般来说，只要是类自己管理内存就应该警惕内存泄漏问题
### 出现在缓存中
推荐使用WeakHashMap作为缓存，这个实现是基于弱引用的
> 弱引用： 当GC开始，看见弱引用的对象就回收<br>不要使用基础类型作为Key

### 监听器和其他回调

## 8.避免使用终结方法和清除方法
### 缺点
    1. 不能保证被及时执行。
    2. 在终结过程中抛出未被捕获的异常则不终结
    3. 有严重的性能损失
    4. 存在严重的安全问题。为了避免非final类遭受攻击，应当便携一个空的final修饰的finalize方法禁止子类继承
### 替代方法
> 让类实现AutoCloseable，要求实例不再需要的时候调用close方法,一般利用try-with-resource方法确保终止


## 9. try-with-resource优于try-finally

### 参考
[《更优雅地关闭资源 - try-with-resource及其异常抑制》](https://www.cnblogs.com/itZhy/p/7636615.html)
<br>JDK7以后使用此种写法用来自动关闭实现了AutoCloseable接口的外部资源类

### 与其他方式比较
此种方法只出现一种异常，因为第二种异常被抑制了。使用`getSupressed()`可以调用

### 示例
```java
void copy(String src, String dst) throws IOException {
    try(InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst);) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
}
```


# 第二章、对于所有对象都通用的方法

## 10.覆盖equals遵守通用规定 **可以使用AutoValue生成10、11**

### 单例模式下不需要覆盖equals方法详见第一条

### 约定内容 针对任意非null的x

1. 自反性 `x.equals(x)` 返回true
2. 对称性 `x.equals(y)`当且仅当`y.equals(x)`都为true时才成立<br>例如写一个忽视大小写的类，与String类型做比较时返回true但是当String与这个类型做比较则返回false就违反了规定
3. 传递性 若`x.equals(y)`为true，`y.equals(z)`为true，则`x.equals(z)`也为true
4. 一致性 对象信息没被修改情况下多次调用返回相同结果
5. `x.equals(null)`必须为false


### 如何实现高质量equals方法
1. 使用==操作符检查 参数是否为这个对象的引用
2. 使用instanceof检查参数是否为正确类型
3. 参数转为正确类型
4. 对成员变量挨个检查
> 对象引用域递归地使用equals方法<br>float域使用Float.compare(float, float)<br>double域使用Double.compare(double, double)<br>==对比Float.equals和Double.equals方法减少自动装箱提升性能==<br>数组域可以使用Arrays.equals方法<br>不同比较顺序可能会影响性能，最佳性能应是将最可能不一致地域放在最前面

### 告诫
    1. 覆盖equals总要覆盖hashcode
    2. 不要企图让equals过于智能
    3. 不要将equals声明地object对象换为其他对象

## 11.覆盖equals时总要覆盖hashcode **可以使用AutoValue生成10、11**

### 每个覆盖equals方法的类中，必须覆盖hashcode方法

> OBJECT规范：<br>1.对象的equals方法中所比较信息没有被修改时，对一个对象多次调用hashcode返回的方法都必须时同一个值；不同程序中可以不一致<br>2.两个对象通过equals方法比较为相等则hashcode必须产生同样的整数结果<br>3.如果equals不相等，不一定要求hashcode必须产生不同结果。==不相等对象产生不同结果，可以提高散列表性能，即为不相等的对象产生不相等的散列码==<br>==总结：当equals相等时hashcode必须相等，反之hashcode相同 equals不一定相等==

### 举例 
当违反了object规范第二条时
> PhoneNumber时一个没有覆盖hashcode的类<br>Map<PhoneNumber, String> map = new HashMap<>();<br>m.put(new PhoneNumber(1,1,1), "zed");<br>当使用m.get(new PhoneNumber(1,1,1));时返回null。==因为这过程中get产生了一个对象，put又产生了一个对象，两个对象的hashcode并不相同（违反了第二条规定）hashmap的put方法会把第一个对象放在一个散列桶中，get方法却根据第二个对象在另一个散列桶中寻找，因此返回null。即使侥幸放在了一个散列桶中，由于hashmap的优化将每项相关联的散列码缓存起来，如果散列码不匹配则不再去检验对象的等同性==<br>如果hashcode总是相同则都放在同一个散列桶中，使散列表退化为链表，极大降低性能，甚至可能导致无法正常工作


### 简单的实现方法
1. 声明一个int变量并命名为result，将它初始化为第一个关键域的散列码c，如2.a中计算
2. 对象中剩下的每一个关键域f都完成以下步骤
    1. 为该域计算int类型的散列码c：
        * 如果该域是基本类型，则计算Type.hashCode(f)，type指的是装箱类型
        * 如果是一个对象引用，并且递归的调用equals进行比较，则递归地调用hashcode。如果需要复杂的比较，则为这个域计算一个“范式”，然后针对范式调用hashcode。如果域的值null，则返0
        * 如果该域是一个数组，则要把每一个元素当作单独的域进行处理。然后根据步骤2.b中将这些列码组合起来。如果数组中没有重要元素，可以使用一个常量但最好不要用0.如果数组中所有素都很重要，可以使用Arrays.hashCode方法
    2. result = 31 * result + c ==通用31， 31 * i == （i << 5）- i==
3. 返回result

### 示例
```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = result << 5 - result + Short.hashCode(prefix);
    result = result << 5 - result + Short.hashCode(lineNum);
    return result;
    
    // 与上述质量相当，但是运行速度更慢一些
    //return Objects.hahs(lineNum, prefix, areaCode);
}
```
### 如果一个类是不可变的，并且计算散列码开销较大时，考虑将散列码存储在对象内部
```java
private int hashCode;
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = result << 5 - result + Short.hashCode(prefix);
    result = result << 5 - result + Short.hashCode(lineNum);
    return result;
    
    // 与上述质量相当，但是运行速度更慢一些
    //return Objects.hahs(lineNum, prefix, areaCode);
}
```
### 试图排除关键域来提高性能，也许会导致散列表慢到根本无法使用

## 12.始终要覆盖toString

## 13.谨慎的覆盖clone
### 约定
> x.clone() != x //true <br>x.clone().getClass() == x.getClass() //true <br>x.clone().equals(x) //true 

### 浅克隆和深克隆
clone方法就是另一个构造器；必须保证它不会伤害原始的对象，确保约束条件<br>例如如果类中有引用对象如数组则需要递归调用clone，否则会导致克隆对象和原始对象引用同一数组，一起变化。==浅克隆与深克隆区别==
```java
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch(CloneNotSupportesException e) {
        throw new AssertionError();
    }
}
```

## 14.考虑实现Comparable接口

### 值得实现的类的特征
1. 值类
2. 具有明显的内在排序关系，比如按字母顺序、按数值顺序或者按年代顺序

### 通用约定
当对象小于、大于或者等于指定对象时分别返回一个负整数、0或者正整数。<br>==无法比较情况则抛出classCastException异常==
1. sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2. 关系可传递 (x.compareTo(y) > 0 && y.compareTo(z) > 0) == true 则 x.compareTo(z) > 0
3. x.compareTo(y) == 0 则所有的z都必须满足 sgn(x.compareTo(z)) == sgn(y.compareTo(z))
4. 强烈建议 x.compareTo(y) == 0 时 x.equals(y) ==true<br>==如果违反这个条件则应注明，内在排序功能与equals不一致==

### 其他情况
BigDecimal违反了第四条建议，在这个类的equals方法上，比较了大小和精度；在compareTo方法上只比较了大小因此会出现以下情况
```java
BigDecimal b1 = new BigDecimal(1.2);
BigDecimal b2 = new BigDeciaml(1.2000);
b1.equals(b2);// false
b1.compareTo(b2); // 0

// 一般来讲不建议使用new BigDeciaml(float) 而是使用 new BigDecimal(String) 或者是 BigDecimal.valueOf(float)
BigDecimal b3 = new BigDecimal("1.2");
BigDecimal b4 = new BigDecimal("1.2000");
b3.equals(b4); // true
b3.compareTo(b4); // 0
```
### 如果一个类没有实现此接口，可以显式使用Comparator来替代，或者使用已有的比较器。
### 可以使用java8中Comparator接口配置的一组比较器构造方法，但是需要牺牲一定的性能。
```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
.thenComparingInt(pn -> pn.prefix)
.thenComparingInt(pn -> pn.lineNum);
```
### 关于基本值类的实例
```java
// 错误，会导致整数溢出
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}

// 以下两种方法二选一
// 基于compare方法
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integet.compare(o1.hashCode(), o2.hashCode());
    }
}

// 基于构造器
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());

```


# 第三章、类和接口

## 15. 使类和成员的可访问性最小化
### 其他文献中的说明
《java开发手册1.5》中，在一个项目进行优化解耦的时候，如果是私有方法想删除就可以删除，但是如果是一个public方法，删除后还是有点担心，尽量让这些类和成员在可限制范围内活动
### 优点
在系统完成后，可以确定哪些组件影响系统性能，进一步优化不会影响其他组件
### 原则
1. 对于顶层类和接口，只有公有和包级私有两种，如果缺省公有则是包级私有。==如果是包级私有则后续对其进行改写也不会影响客户端，如果是公有的则需要永远支持它==，如果只是某一个类使用到了则建议使用私有嵌套类
2. 四种访问级别中私有和包级私有属于类的实现之一。受保护和公有则是导出的api需要一直支持。
3. 包含公有可变域的类通常不是线程安全的。

### 示例：暴露成员变量中的数组会是客户端可以修改数组内容，安全漏洞一个常见根源：
```java
// 这种方式虽然final修饰了数组，但是只是让数组的引用地址不可变，内容仍然是可变的
public static final Thing[] = {...};

// 如果使用get/set返回数组引用也会导致这个问题

// 正确方式
// 1.增加一个不可变列表
private static final Thing[] PRIVATE_VALUES = { ... };

// 2.使数组变成私有，并添加一个公有方法返回数组的克隆
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing values() {
    return PRIVATE_VALUES.clone();
}
```
### ==java9开始==新增了两种隐式访问级别


### 16.在公有类而非公有域中使用访问方法
1. 不可直接暴露域，需要提供访问方法


## 17.使可变性最小化
### 以String类为代表jdk中的不可变类，不容易出错，更加安全
> BigInteger<br>BigDecimal<br>基本数据类型的包装类

### 5条规则使类变为不可变
1. 不要提供任何会修改对象状态的方法比如get/set
2. 保证类不会被扩展。为防止子类化一般用final进行声明
3. 声明的域都是final修饰的
4. 声明的域都是私有的。
5. 确保任何可变组件的互斥访问，==如果具有指向可变对象的域，则应确保客户端永远无法获得这些对象的引用来保证不可变==

### 不可变对象本质上是线程安全的，可以自由共享。
应当鼓励客户端利用这一特点，尽可能重用现有的实例
```java
// Complex 复数类为例
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```
这种方法可以进一步扩展，建立工厂类，把频繁请求的实例缓存起来，当再次请求时则不需要创建新的实例==nosql缓存思想== 。提供了无偿的失败的原子性，状态永远不变因此不存在临时不一致的问题。

### ==缺点==：每个不同的值都需要一个单独的对象
flipBit方法创建了一个新的BigInteger实例，只与原来的有一位的不同。如果原长有上百万位，经过数次操作最后就只剩最后的结果一个对象了，此时性能问题也暴露了出来。<br>BigSet与BigInteger不同的是，提供了一个方法，允许在固定时间内改变此实例单个位的状态。


### 不可变类的其他设计方案
不使用final修饰，将构造方法都私有化或者包级私有，然后提供公有的静态工厂。此方法在后续开发版本中提供了改善性能的可能性==在工厂中添加缓存==
### 如果实现了Serializable接口，需要提供显式的readObject或者readResolve方法，或者使用ObjectOutputStream.writeUnshared/ObjectInputStream.readUnshared。详见第88条

## 18.复合优先于继承

### 打破封装性，当超类的实现更新以后，==子类依赖于超类的实现==功能也会变化。
```java
public class InstrumentHashSet<E> extends HashSet<E> {

    private int addCount = 0
    @Override
    public boolean addAll(Collection<? extends E> c) {
        // 加了一次
        addCount += c.size();
        // 父类方法又调用一次增加
        return super.addAll();
    }
    
    // 返回6 
    public int getCount() {
        return addCount;
    }
}
```
### 复合的原理：不依赖于超类的实现
创建一个新类转发类，包含一个私有域指向现有类
```java
public class ForwardingSet<E> implements Set<e> {
    // 指向现有类
    private final Set<E> set;
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        return c.addAll();
    }
}

public class InstrumentSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll();
    }
    
    public int getCount() {
        return addCount;
    }
}
```

## 19.


# 第五章、泛型

## 26.不要使用原生态类型
**已经投入使用**
 
 
## 27. 消除非受检的警告

### 如果无法消除警告但是可以证明代码是类型安全的，可以使用`@SupressWarning`注解消除警告。==应当注意尽量缩小范围==
### 使用后需要加一条注释，为什么这么使用。

## 28. 列表优于数组
### 大多数情况下列表都优于数组使用


## 29. 优先考虑泛型


## 30. 优先考虑泛型方法

## 31.利用有限制通配符来提升API灵活性
### <? extends TYPE> -> 有限制通配符
### PESCS -> producer extends & conumser super


## 32. 

## 33.
 
# 第六章、枚举和注解

## 34.用enum代替int常量

### 什么是枚举类型
枚举类型是指由一组固定的常量组成合法值的类型

### int枚举的缺点
1. int时编译时常量，如果与int枚举常量关联的值发生了变化，客户端必须重新编译；否则会造成不准确。
2. 很难将int枚举常量转换成可打印的字符串，打印出来是个数字没什么用

### string枚举的缺点
1. 硬编码放到客户端代码中，一旦书写错误会在运行时报错。
2. 性能问题，依赖于字符串

### 如何将数据与枚举常量关联  
需要声明一个域然后编写一个带有数据并将数据保存在域中的构造器

### 如果枚举类型被删除了会怎么样？
- 重新编译  
报错
- 不重新编译  
抛出异常

### 对枚举类型，如何绑定实例与方法？
1. 使用switch，但是会导致一个问题，如果添加一个实例但是忘记添加switch语句则会进入default
2. 使用抽象方法，每个实例实现自己的方法


### 枚举类型可以覆盖toString，如何实现一个stringToEnum
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(
toMap(Object::toString, e -> e));
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

### 在枚举类型中定义普适性方法
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND)
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    PayrollDay() {
        this.payType = PayType.WEEKDAY;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MIN_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        abstract int overtimePay(int minsWorked, int payRate);
        private static int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
### 何时使用枚举？
- 每当需要一组固定变量，并且在编译时就知道其成员的时候
- 当然，不一定要一直保持不变

## 35.使用实例域替代序数
枚举天生就有一个int值，这就是序数。如果需要使用数，考虑添加域而非使用序数。
```
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3)
    ...
    
    private final int numberOfMusicians;
    Ensemble(int size) {
        this.numberOfMusicians = size;
    }
    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

## 36.使用EnumSet代替位域

### 什么是位域？ 
用OR位运算将几个常量合并到一个集合中，称为位域`text.applyStyles(A | B);`
### 替代方法
虽然平常使用枚举，但是在传递多组常量集时仍然倾向用位域。==替代方法：使用java.util.EnumSet==

### 例子
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    public void applyStyles(Set<Style> styles) { ... }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

## 37.用EnumMap代替序数索引

### EnumMap改写后的代码
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for(Plant.LiftCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for(Plant p : garden) 
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```


### 使用lambda进行改写
```java
System.out.println(garden.stream().collect(Arrays.groupingBy(p -> p.lifeCycle)));

System.out.println(garden.stream().collect(Arrays.groupingBy(p -> p.lifeCycle), () -> new EnumMap<>(LifeCycle.class), Arrays.toSet()));
```

### 使用EnumMap代替序数数组
- 如果表示的关系是多维的，使用EnumMap<..., EnumMap<...>>

### 尽量不使用Enum的oridinal方法

## 38.用接口模拟可扩展的枚举

## 39.注解优于命名模式

### 什么是命名模式？
- 定义  
表明有些程序元素需要通过某种工具或者框架进行特殊处理
- 缺点   
    - 容易拼错，拼错了一般就不会执行了
    - 无法确保他们只用于相应的程序元素上
    - 没有提供将参数值与程序元素关联起来的好方法  

### 使用注解进行替代
- 什么是元注解？  
注解声明类型中的注解
- `@Retention(RetentionPolicy.RUNTIME)`  
表明注解在程序运行时RUNTIME也应该存在
- `@Target(ElementType.METHOD)`  
表明只有在方法声明上才是合法的，其他不合法

### 注解不会影响被注解的代码的语义

### 多值注解
- 声明为数组
- `@Repeatable`元注解对注解的生命进行注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Exception> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}

@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void testMethod() { ... }
```
- ==生成包含一个注解类型的合成注解== 


## 40.坚持使用Override注解

### 示例
- 不使用注解覆盖Object.equals方法可能会造成重写


## 41.用标记接口定义类型

### 什么是标记接口
- 不包含方法声明的接口
- 只是致命了某一个类实现了某种属性的接口
- 举例`RandomAccess`,`Serializable`

### 标记接口和标记注解的区别
- 标机接口定义的类型由标记类的实例实现
- 标记注解没有定义这样的类型

### 标记接口的优点1：更加精确地锁定
- 

### 标记注解的优点：是更大注解机制的一部分
- 在那些支持注解作为编程元素之一的框架中同样具有一致性

### 什么时候用标记注解，什么时候用标记接口？
- 标记适用于任何元素应该使用注解
- 使用标记接口可以提供==类型检查==

# 第七章、Lambda和Stream

## 42.Lambda优先于匿名类

### 什么是匿名类
- 带个单个抽象方法的接口
```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s1) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

### 什么是lambda
- 类似于匿名类的函数，但是更加简洁
```java
Collections.sort(words, 
(s1, s2) -> Integer.compare(s1.length(), s2.length()));

// 简化1
Collections.sort(words, comparingInt(String::length));

// 简化2 java8之后
words.sort(comparingInt(String::length));

```
- 没有参数类型和返回值类型  
编译器通过==类型推导==的过程

### 删除所有lambda参数，除非它们存在能够使程序更加清晰
- 当编译器产生错误信息无法推导类型时就需要指定

### 使用lambda可以让不能使用函数对象的地方也可以使用
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
        },
    MINUS("-"){ 
        public double apply(double x, double y) { return x - y; },
    }
    ...
    };
    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }
    @Overrride
    public String toString() {
        return symbol;
    }
    public abstract double apply(double x, double y);
}

// lambda version
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINS ("-", (x, y) -> x - y),
    ...
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    @Override
    public String toString() { return symbol; }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}

```
- 使用了DoubleBinaryOperator接口

### 什么情况不要使用lambda
- 如果一个计算本身不是自描述的
- 超出几行
    - 一行最理想
    - 三行是合理最大极限
- 很长则难以阅读

### lambda限制
- lambda只限于接口
- 如果是抽象类的实例则需要匿名类
- lambda无法获得自身引用
    - ==this指外围实例== 
    - 匿名类中this指匿名类实例

### 尽可能不要序列化一个lambda


## 43.方法引用优先于lambda
### 什么是方法引用
- 比lambda更加简洁
```
map.merge(key, 1, (count, incr) -> count + incr);

// 改进
map.merge(key, 1, Integer::sum);
```

### 如果lambda过长
- 使用方法引用
- 从lambda中提取代码，放到一个新的方法中，用该方法的一个引用代替lambda

### 一个使用lambda而非方法引用的反例
```
// 方法引用
service.execute(GoshThisClassNameIsHumongous::action);

// lambda
service.execute(() -> action());
// Q:这种写法中如何指定action的类？
```

### 什么时候用lambda，什么时候用方法引用
- 什么简单就写什么


## 44.坚持用标准的函数接口
### 如果准备函数接口能够满足需求，应该优先考虑而不是专门再构建一个新的函数接口

### 基础接口
一共有43个接口，记住以下6个可以推断出其他的  
1. Operaotr代表结果与参数一致的函数  
2. Predicate接口代表带有一个参数并返回布尔型的函数
3. Function接口代表参数与返回值类型不一致
4. Supplier代表没有参数并且返回一个值。==提供==
5. Consumer代表带有一个参数但不返回任何值。==消费==

> T = input  
R = result

接口|函数签名|范例|lambda用法|备注
-|-|-|-|-
UnaryOperator<T>|T apply(T t) |String::toLowerCase|(T) -> T|单参数，返回类型相同
BinaryOperator<T>|T apply(T t1, T t2)|BigInteger::add|(T, T) -> U|多参数
Predicate<T>|boolean test(T t)|Collection::isEmpty|(T) -> boolean|返回类型布尔
Function<T,R>|R apply(T t)|Arrays:asList|(T) -> R|返回类型不同
Supplier<T>|T get()|Instance::now|() -> T|返回新的实例
Consumer<T>|void accept(T t)|System.out::println|(T) -> void|单个参数不返回结果


### 变种接口
- 分别作用于基础类型int、long、double
- 基础接口名称前面加基本类型 `e.g. IntPredicate` ==Function不同==LongFunction<int[]>表示入参是一个long，返回一个int数组
- Function的另外9个变种
    - 用于入参和结果类型为基本类型：  
    SrcToResult `e.g. LongToIntFunction`(6种，int、long、double)
    - 如果基本类型转对象  
    SrcToResult `e.g. DoubleToObjFunction`(3种)
- 接受两个参数的接口
- BooleanSupplier


### 标准函数接口只支持基本类型
**千万不要用包装类型的基础函数接口来代替基本函数接口**，破坏了“基本类型优于装箱类型”原则。==装箱基本类型进行批量操作会导致性能问题==

### 什么时候应该自己编写接口？
- 接受三个参数的predicate接口
- 抛出受检异常的接口
- 其他情况：结构相同的标准函数接口可用，但还是应该自己编写函数接口
- ==示例==   
以Comparator<T>为例：与ToIntBiFunction<T, T>接口在结构上一致，但是必须自己编写接口
    1. 每当API被使用，名称提供了良好的文档信息
    2. Comparator接口对于如何构成一个有效实例有严格条件限制，实现此接口相当于承诺遵守契约
    3. 接口提供了大量好用的缺省方法，可以对比较器进行合并和转换
- ==通用规则==
    1. 通用，受益于描述性名称
    2. 具有与其关联的严格契约
    3. ==受益于定制的缺省方法==

### 始终使用@FunctionalInterface注解对自己编写的函数接口进行标注
1. 告诉其他人，这个是针对lambda设计的
2. 不会编译，除非只有一个抽象方法
3. 避免其他人对它增加抽象方法


## 45.谨慎使用stream

### 什么是stream
- 简化串行或者并行的大批量操作
- 提供了两个关键的抽象对象
    - Stream(流) 代表数据元素有限或者无限的顺序
    - Stream pipeline(流管道) 代表这些元素的一个多级运算
    - ==两者关系==  
    一个stream pipeline中包含一个源stream，接着是0个或者多个中间操作和一个终止操作  
==所有的中间操作都是将一个stream转化为另一个stream==

### stream pipeline
- **如果没有终止操作stream pipeline是静止的**，直到调用终止操作才会开始计算并且不需要的数据元素永远不会被计算
- 默认情况下顺序运行，如果想要并发执行，只需要在该pipeline的任何stream上调用parallel方法即可

### 什么时候应该使用Stream
- 滥用stream会让程序代码难懂难维护
- 最好避免用Stream来处理char值
```
"Hello world!".chars().forEach(System.out::println);
// 输出一串数字

"Hello world!".chars().forEach(System.out.println((char) x));
// 输出hello world！
```
- 重构现有代码来使用Stream，并且只在必要的时候才在新代码中使用
- ==示例，只能用代码块而不能用函数对象完成==
    - 代码块中可以读取或者修改范围内任意局部变量，lambda只能读取final值，并且不能修改任何local变量
    - 代码块中可以从外围方法中return、break或者continue外围循环或者抛出该方法声明要抛出的任何受检异常，lambda无法完成
- ==Stream的优势场景==
    - 统一转换元素的序列
    - 过滤元素的序列
    - 利用单个操作（添加、连接或者计算最小值等）合并元素的顺序
    - 将元素的序列存放到一个集合中，比如根据公共属性进行分组
    - 搜索满足某些条件的元素序列
- 一个两者皆可的示例
```java
// 迭代
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values()) 
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}

// stream
private staic List<Card> newDeck() {
    return Stream.of(Suit.values()) // 开启流
        .flatMap(suit ->  // 映射
            Stream.of(Rank.values())
                .map(rank -> new Card(suit, rank))) // rank和suit组成一个card
        .collect(toList()); // 形成list
}
```

## 46.优先选择Stream中无副作用的函数
### 一个反例
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String) words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- 改变外部状态
- `forEach`操作的任务不只展示由Stream执行的计算结果

### forEach操作应该只用于Stream计算的结果，而不是执行计算
- 改良
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCaser, counting()));
}
```
- 显示迭代不适合并行
- 用forEach将计算结果存入之前已经存在的集合

### Stream中收集结果
- Collectors.toSert();
- Collectors.toList();
- Collectors.toCollection(collectionFactory);


### 一个例子
```
Map<Artist, Album> topHits = albums.collect(
    // (keymapper, valueMapper, mergeFunction)
    // 也就是第一个是键映射，第二个是值映射，第三个是解决多个V对应一个K
    toMap(Album::artist, a -> a, maxBy(comparing(Album:sales))));
```
- 关于如何使用comparing，可以参考附录

### 为了正确使用Stream，必须了解收集器
- 收集器工厂  
> toList  
toSert  
toMap  
groupingBy  
joining  返回拼接完的字符串
```java
// 定义人名数组
final String[] names = {"Zebe", "Hebe", "Mary", "July", "David"};
Stream<String> stream1 = Stream.of(names);
Stream<String> stream2 = Stream.of(names);
Stream<String> stream3 = Stream.of(names);
// 拼接成 [x, y, z] 形式
String result1 = stream1.collect(Collectors.joining(", ", "[", "]"));
// 拼接成 x | y | z 形式
String result2 = stream2.collect(Collectors.joining(" | ", "", ""));
// 拼接成 x -> y -> z] 形式
String result3 = stream3.collect(Collectors.joining(" -> ", "", ""));
```

## 47.Stream要优先用Collection作为返回类型？

### 对于公共的、返回序列的方法，collection或者适当子类型通常是最佳的返回类型
- 如果返回的序列足够小、容易存储可以返回ArrayList或者HashSet
- ==如果是个巨大的序列，不要在内存中保存，作为集合返回即可==


## 48.谨慎使用Stream并行

### 使用并行stream有可能无法提升性能
- 源头来自Stream.iterate
    - 用于创建无限流 
- 使用了中间操作的limit
    - 和Stream.iterate联用
```java
Stream.iterate(0, n -> n+2)
    .limit(5)
    .forEach(System.out::println);
```

### 可以使用Stream并行获得性能提升的情况
- ArrayList、HashMap、HashSet、ConcurrentHashMap实例，数组、int范围和long范围
- ==特点==
    - 可以被精确、轻松分成任意大小的子范围  
    使用分割迭代器(spliterator)
    - 顺序处理时，提供优异的引用局部性  
    序列化的元素引用一起保存在内存中。如果没有的话则是引用访问到的对象在内存中可能不是一个挨着一个的。没有这个特性，线程会出现闲置，等待数据从内存转移到处理器的缓存，**最佳引用局部性的数据结构：数组**


### 并行Stream是一个很严格的优化
- 适当条件下添加parallel调用，可以利用处理器更多的内核
- 但是情况较少，需要在优化前后进行测试，确保不是反优化

# 第八章、方法

## 49.检查参数有效性

### Objects.requireNonNull
- api
    - `this.str = Objects.requireNonNull(str, "<error msg>");`
    - 可以进行判空并且抛出自定义NPE信息
![Objects.requireNonNull](https://note.youdao.com/yws/res/10548/BCA0760739274905A1BCE86EE929974C)

### 断言

- 如果没有起作用，本质上不会产生开销
- 需要对JVM进行设置`-ea`开启断言，开启后不符合会抛出`AssertionError`

## 50.必要时进行保护性拷贝

### 一个例子
- 声明start不能在end之后
- 但是可以对date进行修改
```java
// source
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "after" + end);
        }
        this.start = start;
        this.end = end;
    }
    ....
}

// 修改end的时间，导致这个类不可用
Date start = new Date();
Date end = new Dater();
Period p = new Period(start, end);
end.setYear(78);

// 保护性拷贝
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if(this.start.compaterTo(this.end) > 0) {
        throw new IllegalArgumentException(start + "after" + end);
    }
}
```

### 应该在哪进行保护性拷贝
1. 构造方法中
    - 验证参数有效性之前
    - 有效性检查针对拷贝之后的对象，而不是原始对象
2. 访问方法中
    - setter中进行修改也会改变类的内部参数
3. ==不只针对不可变类==，如果允许客户提供的对象进入内部的数据结构中，需要考虑是否可以忍受对象进入数据结构之后发生变化，如果不行则需要保护性拷贝
    - 由客户提供的对象引用作为Set元素或者Map的key，就必须要保证不可变
4. 对于长度非零的数组，总是可变的。返回客户端应该返回保护性拷贝
    - 或者返回数组的不可变视图（参考第15条） 

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);

// 防御第二种修改
    public Date start()
        return new Date(start.getTime());
    }
    
    public Date end() {
        return new Date(end.getTime());
    }
```

### 使用clone()还是不使用
- 对于构造方法中，参数类型可以被不信任方子类化的参数，不要使用clone进行拷贝
    - Date类不是final的，所以不能保证clone方法一定返回Date类 
- 对于访问方法可以使用clone，因为内部的参数类型一定是Date类型的


## 51.谨慎设计方法签名
本条目是若干API设计技巧总结

### 谨慎的选择方法的名称
- 易于理解

### 不要过于追求提供便利的方法
- 只有一项操作经常被用到的时候，才考虑为他提供快捷方式

### 避免过长的参数列表
1. 参数限制
    - 目标是不超过4个参数   
    - 相同类型的长参数列表会让API用户记不住参数顺序
2. 缩短的技巧
    - 把一个方法分解成多个方法，每个方法只需要接收子集
    - 创建辅助类，用来保存参数的分组（见24条）比如创建类表示纸牌的点数和花色接受两个参数
    - 从对象构建到方法调用都采用builder模式

### 参数类型优先使用接口而不是类
- 比如应该使用Map作为参数类型而不是HashMap

### 对于Boolean参数，优先使用两个元素的枚举类型
- `public enum TemperatureScale { FAHRENHEIT, CELSIUS }`


## 52.慎用重载

### 要调用哪个重载方法是在编译时才做出决定的
```
public Class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    
    public static String classify(List<?> list) {
        return "List";
    }
    
    public static String classify(Collection<?> c) {
        return "Collection";
    }
    
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteget>(),
            new HashMap<String, String>().values()
        };
        for(Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

### 对于重载方法选择是静态的，覆盖方法选择是动态的
- 选择覆盖方法正确版本是运行时进行的，选择的依据是被调用方法所在对象的运行时类型
    - 通常是接口定义对象，实现类来new出来，所以会根据运行时的类型调用方法
    - 当一个子类包含的方法声明与其祖先类中的方法声明有同样的签名，方法就被覆盖了


### 使用重载的一些建议
- 永远不要导出两个具有相同参数数量的重载方法
- 可以起不同的名字，不使用重载机制


## 53.慎用可变参数

### 什么是可变参数
```java
static int sum(int... args) {
    int sum = 0;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
```

### 改进：参数列表至少有一个
```java
static int sum(int firstArg, int... args) {
    int sum = firstArg;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
```

### 避免使用可变参数
- 每次调用可变参数方法都会导致一次数组分配和初始化
- 重载多个方法，分别是1,2,3个参数和可变参数


## 54.返回零长度的数组或者集合，而不是null

### 优点
- 客户端不需要额外写代码去处理null
    - 忘记处理会NPE 

## 55.谨慎返回optional

### java8之前无法返回任何值情况下
- 返回null
    - 客户端需要额外编写代码来处理null 
- 抛出异常
    - 需要捕捉整个堆栈轨迹，开销非常大 

### 什么是Optional
- 一个不可变容器
- 可以存放单个非null的T引用或者什么也没有
- 最多只可以存放一个元素
- 与null对比
```
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("empty collection");
    }
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return result;
}

// optional
public static <E extends Comparable<E>> E max(Optinal<E> c) {
    if (c.isEmpty()) {
        return Optional.empty();
        
    }
    E result = null;
    for (E e : c) 
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    return Optional.of(result);
}
```

### Optional缺省值
- 如果返回Optional客户端必须做出选择，该方法==不能返回值==时的做法
```java
String lastWordInLexicon = max(words).orElse("no words");

String lastWordInLexicon = max(words).orElse(TemperTantrumException::new);
```
- 如果可以证明非空，就可以不指定
    - 判断错了就会抛出==NoSuchElementException== 
- ==有时候缺省值的开销非常高==，除非十分必要，否则还是希望避开
    - 替代方法：orElseGet/orElseCompute/isPresent
```
streamOfOptionals
.filter(Optional::isPresent)
.map(Optional::get)
```

### 什么情况下不该使用Optional
- 容器类型包括集合、映射、stream、数组和optional
- 无法返回结果并且返回结果客户端必须特殊处理时返回Optional
    - ==需要成本== 
- 必须分配和初始化的对象，从optional中读取需要额外开销
    - 不适合高性能场景 
- 不要返回基本包装类型的Optional    
    - 替代方法：OptionalInt、OptionalLong、OptionalDouble 
    - boolean、byte、cha、short、float除外

## 56.为所有导出的API元素编写文档注释

### 工具：javadoc
- 个人认为意义不大
    - 使用参考书第56条

# 第九章、通用编程

## 57.将局部变量的作用域最小化
- 与15条类似
    - 增强可读性
    - 降低出错性

### 对于局部变量：只在第一次使用的地方进行声明
- 过早声明导致
    - 可读性变低
    - 作用域被扩展

### 局部变量应该包含一个初始化表达式
- 如果没有那就应该推迟声明
- ==例外==与try-catch有关
    - 变量初始化有可能抛出异常
    - 初始化应该try-catch内部进行
    - 如果外部需要使用则应该在try前面声明

### 循环终止之后不再需要循环变量的内容，for优于while
- 例子
```java
Iterator<Element> i = c.iterator();
while(i.hasNext()) {
    ...
}

Iterator<Element> i2 = cc.iterator();
while(i.hasNext()) { // bug，如果使用for根本不会通过编译
    ...
}
```
- for循环有更高可读性
```java
for (int i = 0, n =expensiveComputation(); i < n; i++) {
    ...''
}
```

### 将局部变量作用域最小化
- 使方法小而集中

## 58.for-each优于传统for循环

### 无法使用for-each的情况
- 解耦过滤
    - 需要遍历集合删除选定元素，需要显式迭代器
    - java8中的Collection的removeId可以避免显式遍历
- 转换
    - 需要取代部分元素（赋值）需要使用==列表迭代器或者数组索引== 
- 平行迭代
    - 需要并行迭代多个集合，使用迭代器或者数组索引可以多个集合同步进行


## 59.了解和使用类库

### 生成一个0和某个上界之间的随机整数
```java
static Random rnd = new Random();
static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```
- 缺点1.如果n一个比较小的n的乘方，一段时间后随机序列会重复
- 缺点2.不是2的乘方，有些数会比其他数出现更频繁
- 缺点3.极少数情况下会出现很难复现的失败
- ==改良==，`Random.nextInt(int)`

### Java7开始使用ThreadLocalRandom生成随机数
- 比上述改良情况更快

### 程序员必须熟悉的包
- java.lang
- java.util
- java.io
- collections framework
- stream
- java.util.concurrent


## 60.如果需要精确的答案，请避免使用float和double

### float、double不适合货币计算
- 会有精度丢失

### 使用BigDecimal、int或者long计算货币
- 使用long计算货币需要以分为单位


## 61.基本类型优先于装箱基本类型

### 基本类型和装箱基本类型的区别
1. 基本类型只有值，装箱类型具有相同的值和不同的同一性？（没懂）
2. 基本类型只有函数值，

<center>

基本类型|包装类型
-|-
只有值|具有和它们值不同的同一性？没懂
只有函数值|除了与基本类型对应的函数值之外还有null
更少的时间和空间|

</center>


### 一个Integer比较器
- 当相等时结果出错
    - 使用`><`正常拆箱，但是使用`==`时就是比较两个引用对象地址
```java
Comparator<Integer> naturalOrder = (i, j) -> (i - j) ? -1 : (i == j ? 0 :1)
```

### 包装类型可能导致NPE
```java
public class TestClass {
    static Integer i;
    public static void main(String[] args) {
        if (i == 42) {
            System.out.println("42");
        }
    }
}
```
- 这个程序会NPE
- 如果使用int就不会


### 拆装装箱可能导致性能问题
```
public static void main(String[] args) {
    Long sum = 0L;
    for (Long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(i);
}
```

## 62.如果其他类型更合适，尽量避免使用字符串

### 字符串不适合做的事情
- 不适合代替枚举
- 不适合代替聚合类型
- 不适合代替能力表(capabilites)
    - 可能发生在线程中
    - 标准做法
```java
    public final class ThreadLocal<T> {
        public ThreadLocal();
        public void set(T value);
        public T get();
    }
 ```

## 63.了解字符串连接的性能

### 大规模场景中
- 拼接n个字符串重复使用'+'需要n的平方级时间

### 更好的方案-StringBuilder
```java
public String statement() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }
    return b.toString();
}
```

## 64.通过接口引用对象

### 使用接口来实例化可以更加灵活
```java
Set<Son> sonSet = new LinkedHashSet<>();

// 可能业务数据结构改了
Set<Son> sonSet = new HashSet<>();
```
- 实现改了但是程序仍然可以正常运行
- 为什么要改变实现？
    - 第二个实现比第一个实现有更好的性能 

### 如果没有合适的接口，完全可以用类而不是接口来引用对象
- 值类
- 不是基于接口而是基于类的框架，抽象类
- 需要用到有接口中没有的方法

### 如果没有合适的接口，用类层次结构中提供了必要功能的最小具体类来引用


## 65.接口优先于反射

### 反射是什么
- java.lang.reflect包中提供，通过程序来访问任意类任何方法

### 反射的缺点
- ==损失了编译时类型检查==
- 执行反射需要的代码非常笨拙和冗长
- 性能损失

### 反射的优点
- 部分用到的类是编译时不可用的，可以通过反射来创建
- 一个例子，侵入式操作，检查是否符合set规范
```java
public static void main(String[] args) {
    Class<? extends Set<String>> c1 = null;
    try {
        c1 = (Class<? extends Set<String>>)Class.forName(arg[0]);
    
    } catch(ClassNotFoundException e) {
        fatalError("class not found");
    }
    
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = c1.getDeclaredConstructor();
    } catch(NoSuchMethodException e) {
        fatalError("no parameterless constructor");
    }
    
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch(IllegalAccessException e) {
        fatalError("Constructor not access");
    } catch(InstantiationException e) {
        fatalError("Class not initialable");
    } catch(InvocationTargetException e) {
        fatalError("Constructor threw" + e.getCause());
    } catch(ClassCastException e) {
        fatalError("class doesn't implement set");
    }
    
    s.addAll(Arrays.asList(args)).subList(1, args.length);
}
```

## 66.谨慎地使用本地方法

### 什么是本地方法
- JNI - JAVVA NATIVE INTERFACE，本地使用C、C++编写的方法
    - 提供了访问特定于平台机制的能力，比如注册表
    - 访问本地遗留代码库的能力

## 67.谨慎地进行优化代码

### 不要为了性能牺牲合理的结构
- 要编写好的程序而不是快的程序
    - 好的程序是把设计决策集中到单个模块，改变单个决策不影响系统其他部分 

### 努力避免限制性能的设计决策
- 最难更改的组件是指定了模块之间交互关系以及与外界交互关系的组件
    - API、交互层协议、永久数据格式 

### 考虑API设计决策的性能后果
- 公有类型成为可变的会导致大量不必要保护性拷贝（见50条）
- 大量使用继承会导致子类和超类束缚在一起，限制性能
- API使用实现类型而不是接口导致只能束缚在一个具体的实现上

## 68.遵守普遍接受的命名惯例

### 部分自己不知道的命名规则
- 可以被实例的用一个名词或者名词短语
- 不可以被实例的用复数比如Collectors

# 第十章、异常

## 69.只针对异常情况才使用异常

### 异常应该只用于异常情况下，不应该用于正常的控制流
- 控制流可以使用for-each等来结束

## 70.对可恢复的情况使用受检异常，编程错误使用运行时异常

```
graph LR
throwable-->受检异常EXCEPTION
throwable-->运行时异常RUNTIMEEXCEPTION
throwable-->error
```

### 异常怎么用
- 期望调用者可以恢复，使用受检异常
- 用运行时异常表明编程错误
- 按照惯例，错误被JVM保留下来使用，表明资源不足等情况

## 71.避免不必要地使用受检异常

### 非受检和受检的区别
- 受检表示程序中必须处理的异常 
    - FileNotFound
    - IOException
- 非受检
    - NPE
    - ArrayIndexsOutOfBounds

### 受检异常的缺点
- 需要在程序中额外进行处理（try-catch）
- ==抛出受检异常的方法不能在stream中使用==


## 72.优先使用标准的异常

### 为什么要重用标准异常
- API更易于学习和使用，与习惯用法一致
- 可读性更好
- 异常类越少，内存占用越少，装载时更快

### 常见异常说明

<center>

类名|说明
-|-
IllegalArgumentException|参数不合适
IllegalStateExpcetion|接受对象状态使调用非法
NPE|
IndexOutOfBoundsExpcetion|
ConcurrentModificationExcpetion|设计用于单线程的对象被并发修改了<br>不可靠，不可能可靠地侦测到并发修改
UnsupportesOperationException|对象不支持请求的操作

</center>

### 准确重用异常
- 一副纸牌对象为例，传递的参数大于剩余张数 
    -  IllegalArgumentException  
    参数太大
    - IllegalStateExpcetion  
    纸牌对象纸牌太少
    - 没有可用参数抛出2，否则抛出1

## 73.抛出与抽象对应的异常

### 更高层的实现应该捕获低层异常，抛出高层异常
- 比如如果出现了`FileNotFounException`在业务代码中，应该转译为自定义异常
```
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch(NoSuchElementExceptioon) {
        throw new IndexOutOfBoundsException("Index" + index);
    }
}
```

### 如果有可能理应尽量避免使用
- 确保低层方法能够成功执行

## 74.每个方法抛出的所有异常都要建立文档
- 使用javadoc为方法抛出的异常进行说明

## 75.在细节消息中包含失败-捕获信息
- 在message中打印细节信息，为解决这个异常服务
- 有的异常可能很难复现

## 76.努力保持失败的原子性

### 什么是失败的原子性
- 失败的方法调用应该使对象保持在被调用之前的状态

### 如何保持
1. 使用不可变对象
2. 执行操作之前检查参数
3. 在对象的一份临时拷贝上操作
    - 编写一段恢复代码来拦截操作中发生的失败 


## 77.不要忽略异常

### 空的catch块会使异常达不到应有的目的
- 如果选择忽略需要写一条注释为什么忽略
- 变量名从`e`变为`ignored`

# 第十一章、并发

## 78.同步访问共享的可变数据

### 什么是同步
- 阻止一个线程看到对象处于不一致的的状态
- 保证进入同步方法或者代码块的线程可以看到同一个锁保护的之前的所有的修改结果


### 同步的必要性
- 未能同步共享可变数据会造成程序活性失败、安全性失败
    - 可能是间歇的
    - 也可能和时间相关
```java
public static boolean stop = false;

public static void main(String[] args) {
    Thread background = new Thread(() -> {
        int i = 0;
        while (!stop)
            i++;
    });
    background.start();
    TimeUnit.SECONDS.sleep(1);
    stop = true;
}

// 并不会停止，因为JVM把代码优化为
if (!stop)
    while(true)
        i++;
```
- 同步优化之后的代码
```java
public static boolean stop;

private static synchronized void request() {
    stop = true;
}

private static synchronized void stopReq() {
    return stop;
}

public static void main(String[] args) {
    Thread background = new Thread(() -> {
            int i = 0;
            while (!stopReq)
                i++;
    });
    background.start();
    TimeUnit.SECONDS.sleep(1);
    request();
}
```
- 读写方法必须都进行同步，否则无法保证结果
- 这些方法的同步只是为了通信，而不是互斥访问
    - 可以使用volatile代替锁
    - 保证线程读取该域的时候都是看到最近刚刚被写入的值
    - 禁止指令重排

### volatile并不总是好用
- 只确保了变量在线程之间可见
- 但是没有确保原子性
```java
private static volatile int nextNumber = 0;
public static int getNumber() {
    return next++;
}
```
- 这段程序中，增量操作符（++）不是原子的
    - 读取值
    - 写回一个值
    - 第二个线程在第一个线程读取旧值和写回新值期间读取这个域就会造成==安全性失败==
- 修正方法
    - 使用synchronized修饰方法，保证不会交叉存取
    - 最好使用AtomicLong类，是java.util.concurrent.atomic的组成部分，这个包为单个变量上进行==免锁定、线程安全==的编程提供了基本类型
```java
private static final AtomicLong next = new AtomicLong();

public static long getNumber() {
    return next.getAndIncrement();
}
```

### 高效不可变、安全发布
- 一个线程修改后发布到其他线程上
    - 其他线程没有进一步的同步也可以读取
    - 只要没有再被修改
- 保存在静态域中，作为类初始化的一部分


## 79.避免过度同步

### 过度同步的危害
- 性能降低
- 死锁
- 不确定行为

### 同步方法或者代码块中使用可靠的方法
- 不要调用设计成要被覆盖的方法
- 这些方法不确定，可能导致异常、死锁
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }
    private final List<SetObserver<E>> observers = new ArrayList<>();
    
    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SerObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element)
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added) 
            notifyElementAdded(element);
        return added;
    }
    
    @Override
    ppublic boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (Element e : c) 
            reuslt |= add(element);
        return result;
    }
    
    @FunctionalInterface
    public interface SetObserver<E> {
        void added(ObserverableSet<E>, E element);
    }
    
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver((s, e) -> System.out.println(e));
        for (int i = 0; i < 100; i++) 
            set.add(i);
        
    }
    
    // 替换
    // 导致ConcurrentModificationException
    // 因为只锁了增加删除的行为，但是对主题的内部列表没有锁
    // 当这里在删除的时候，主题内部正在遍历，所以导致异常
    set.addObserver(new SetObserver<>() {
        public void added(Observable<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23)
                e.removeObserver(this);
        }
    });
    
    
    // 错误，死锁
    // 主线程的addObserver获得锁
    // 后台线程请求锁来做removeObserver动作，但是锁在主线程手上
    // 主线程等待后台线程进行完毕，后台线程等待主线程释放锁
    // 因为代码中锁的对象是observers，所以多个函数只有一个能进行
    set.addObserver(new SetObserver<>()) {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService service = Executors.newSingleThreadExecutor();
            
            try {
                service.submit(() -> s.removeOberser(this)).get();
                
            } catch (ExecutionException | InterruptedException e) {
                throw new AssertionError(e);
            } finally {
                service.shutdown();
            }
            }
        }
    }
    
    // 修改，使用克隆的observers来通知订阅者
    private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized(observers) {
            snapshot = new ArrayList<>(observers);
        }
        for (SetObserver<E> observer : snapshot) 
            observer.added(this, element);
    }
}
```

### 使用jdk实现的并发集合
- `CopyOnWriteArrayList`
    - 是`ArrayList`的一个变种
    - 通过重新拷贝底层数组实现所有写操作
    - 内部数组永远不改动
    - 如果大量使用性能会受影响，但是在==观察者模式==中很好，因为它们几乎不改动并且经常遍历

### 实现同步的两种方式
- 省略所有同步，想要并发使用需要客户端从外部同步
    - java.util中的集合（除了vector和hashtable）
- 内部同步
    - java.util.concurrent中的集合

### 不要在同步区域内部调用外来方法
- 避免死锁和数据破坏

## 80.executor、task和stream优于线程

### 利用executorService完成更多工作
- `invokeAny`或者`invokeAll`
    - 等待一个任务集合中任何任务或者所有任务完成 
- `awaitTermination`
    - 等待executor service 优雅完成终止
- `ExecutorCompletionService`
    - 任务完成时逐个抽取结果
- `ScheduledThreadPoolExecutor`
    - 调度某个特殊时间段定时运行或者阶段性地运行的任务

## 81.并发工具优先于wait和notify

### java.util.concurrent高级工具分类
- Executor Framework
- 并发集合 Concurrent Collection
- 同步器 Synchronizer


### 并发集合
- 为标准集合接口提供了高性能的并发实现
    - 这些实现在内部管理同步
    - 不可能排除并发活动
    - 锁定也没有什么作用，只会让程序变慢
- 并发集合导致同步的集合大多被废弃
    - 应当优先使用concurrentHashMap而非synchronizedMap
- 大多数的ExecutorService实现都使用了一个BlockingQueue
    - 一直等待，知道操作成功 

### 同步器
- 使线程能够等待另一个线程的对象，允许它们的协调动作
- 常用同步器
    - CountDownLatch
    - Semaphore
- 较不常用的同步器
    - CyclicBarrier
    - Exchanger
- 功能最强大的同步器
    - Phaser 

### CountDownLatch倒计数锁存器
- 一次性的障碍
- 允许一个或者多个线程等待一个或者多个其他线程来做某事
- 唯一构造器带有一个int类型参数
    - 允许所有在等待的线程被处理之前，必须在锁存器上调用countDown方法的次数 
```
public static long time(Executor executor, int concurrency,
    Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);
        
        for (int i =0; i< concurrency; i++) {
            executor.execute(() -> {
                // 启动指定个数
                ready.countDown();
                try {
                    // 等待其他线程全部启动完成，主线程会控制这个变量一起启动
                    start.await();
                    action.run();
                } catch(InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 完成就-1
                    done.countDown();
                }
            });
        }
        // 等待线程启动完全
        ready.await();
        long startNanos = System.nanoTime();
        // 启动线程池中所有线程一起进行
        start.countDown();
        // 等待完成
        done.await();
        return System.nanoTime() - startNanos;
    }
```
- `await()`当前线程挂起，等待计数完成就会自动继续
- `countDown()` 计数器-1

### 如果需要维护wait和notify应遵从这些建议
- wait必须在同步区域被调用
    - 标准模式如下
```java
synchronized(obj) {
    while (<condition>) 
        obj.wait();
}
```
- 优先使用notifyAll而不是notify
    - 如果处于等待状态的所有线程都在等待同一个条件
    - 每次只有一个线程可以从这个条件被唤醒
    - 选择notify

## 82.线程安全性的文档化

### 线程安全性级别
1. 不可变的
    - 类的实例不可变，所以不需要外部同步
2. 无条件的线程安全
    - 实例可变，但是这个类有足够的内部同步无须外部同步
    - 比如AtomicLong和ConcurrentHashMap
3. 有条件的线程安全
    - 有些方法为了安全并发使用需要外部同步
    - 比如Collections.synchronized包装返回的集合，它们的迭代器要求外部同步
4. 非线程安全
    - 实例可变
    - 客户端需要自己实现外部同步
    - 比如ArrayList和HashMap
5. 线程对立
    - 不能安全地被多个线程使用，即使使用了外部同步
    - ==根源在于==没有同步修改静态数据

### 关于类内部使用lock
- 应用使用一个私有锁对象来代替同步方法
    - 锁不可以在这个类之外被访问
    - ==始终应该声明为final==
- 可以使用普通的监控锁（自己定义）
- 也可以使用java.util.concurrent.locks包中的锁

## 83.慎用延迟初始化

### 什么是延迟初始化
- 指延迟到需要域的时候才初始化
- 如果永远不需要则永远不初始化
- 是一种优化
- 但也可以用来破坏类中的有害循环和实例初始化

### 什么情况下应该使用
- 不使用的情况
    - 可能是反向优化
    - 虽然降低了类加载的开销，但是增加了访问被延迟初始化域的开销
- 如果这个域的开销很高，就值得延迟初始化
- 要确定这点，唯一方法就是测量类在用和不用延迟初始化的性能差别

### 多线程场景下
- 大多数情况下正常初始化优先于延迟初始化
- 如果利用延迟初始化来破坏初始化的循环，需要同步访问方法
```java
private FileType filed;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```
- 如果出于性能考虑需要对静态域使用延迟初始化，需要使用lazy initialization holder class模式
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```
- getField方法不需要同步，并且只执行一句语句
- 出于性能考虑需要对实例域使用延迟初始化，需要使用双重检验模式

## 84.不要依赖于线程调度器
- 依赖于线程调度器来达到正确性或者性能要求的程序，很可能是不可移植的

### 限制线程数量
- 可运行线程平均数量最好不超过处理器的数量
    - 可运行线程平均数量不等于线程总量。
    - 在等待的线程并不是可运行的
- 让每个线程做些有意义的工作
    - 如果线程没有在做有意义的工作就不该运行 
- 任务划分应该适当的小
    - 不应该太小，否则分配的开销也会影响到性能 

### 线程调度的建议
- 不应该一致处于忙-等的状态
    - 反复检查共享对象对处理器有很大负担
- 如果一个程序因为某些线程无法获得足够cpu
    - 不要通过条用Thread.yield修正程序
    - 这种做法是不可移植的
    - 最好可以重新构造应用程序，减少并发运行的线程数量
- 线程优先级是java平台上最不可移植的特征
    - 不必要，不可移植
    - 找到底层原因之前可能再次出现问题

# 第十二章、序列化

## 85.其他方法优先于Java序列化

### 避免序列化攻击
- 最好方式：永远不要反序列化任何东西
- 其他机制完成对象和字节序列之间的转化
    - 跨平台的结构化数据表示法
    - JSON 基于文本 可读
    - Protocol Buffers 基于二进制
- 如果不可避免需要反序列化
    - 永远不要反序列化不被信任的数据 
- 无法避免又无法保证被反序列化的数据安全性
    - 使用对象反序列化过滤(object deserialization filtering)，目前已经被移植到早期版本(java.io.ObjectInputFilter)
    - 可以操作类的粒度，允许接受或者拒绝某些类

### 新系统一定要用JSON或者protobuf来代替
- 不要编写可序列化的类，如果必须这么多必须加倍小心试验


## 86.谨慎地实现Serializable接口

### 让一个类可以被序列化
- 实现Serializable接口
- 直接开销非常低，但是长期开销非常高

### 实现Serializable的代价
1. 一旦发布，大大降低了改变这个类的实现的灵活性
    - 接受这种默认表示法，类中的私有和包级私有都会导出成为API一部分
    - 使类的演变收到限制，序列化版本（serial version uid）
2. 增加出现bug和安全漏洞的可能性
    - 反序列化是一个隐藏的构造器
    - 容易使对象的约束关系遭到破坏，遭到非法访问
3. 随着类发型新版本，相关测试负担也会增加
    - 需要检查新版本中序列化一个实例，在旧版本中反序列化

### 哪些情况不要实现Serializable接口
- 为了继承而设计的类应该尽可能少去实现
    - 为了继承而设计的类中实现了这个接口的Throwable和Component类 
    - 但是很少使用
- 用户的接口也应该尽可能少实现
- 内部类不应该实现

## 87.考虑使用自定义的序列化形式

### 接受默认的序列化形式需要深入考虑
- 需要从灵活性、性能和正确性多个角度进行考虑
- 一般只有自定义的形式和默认形式基本相同才接受默认的序列化形式

### 接受默认的序列化还需要其他的辅助手段
- 必须提供一个readObject方法保证约束关系和安全性

### 默认序列化的缺点
1. 这个类导出的API永远束缚在类的内部表示法上
2. 消耗过多空间
    - 实现细节也会记录在序列化中
    - 但这是不必要的
3. 消耗过多时间
    - 序列化逻辑不了解对象图的拓扑关系
    - 所以必须经过图遍历
4. 引起堆栈溢出
    - 因为需要进行递归遍历 

### 序列化规范
- `transient`注解标记的域不会被序列化
- 必须调用defaultReadObject和defaultWriteObject
    - 否则会序列化时忽略新增加的域
    - 反序列化失败并抛出`StreamCorruptedException`
- 大多数实例域或者所有实例域，都应该被标注`transient`
    - 除非确信它的值是对象逻辑状态一部分
- 如果在读取整个对象状态的任何其他方法上强制任何同步，则必须在对象序列化上也强制这种同步
    - 使用`synchronized`关键字
- 无论哪种序列化，都必须声明一个序列化版本号UID
    - 并且不能修改，否则破坏现有的已被序列化实例的兼容性
    - 修改了会抛出`InvalidClassExcpetion`

## 88.保护性地编写readObject方法
- readObject本质就是一个以字节流为参数的构造方法

### 例子
- 见50条Date类使用保护性拷贝完成约束
- 保护性readObject需要在`defaultReadObject`之后对参数进行校验，不能让参数破坏约束
- 客户端不应该拥有对象引用，所以需要对包含对象引用的域做保护性拷贝

### 编写更加健壮的readObject方法建议
1. 对象引用域必须保持为私有的类，要保护性拷贝这些域中的每个对象。
2. 检查约束条件
3. 反序列化之后必须验证的化使用ObjectInputValidation接口
4. 不要条用类中任何可覆盖的方法

## 89.对于实例控制，枚举类型优先于redResolve

### 保证反序列化仍然可以得到单例属性的方法
- 方法忽略了被序列化的对象，只返回类初始化时候创建的特殊实例
- 否则会创建垃圾对象
```java
private Object readResolve() {
    return INSTANCE;
}
```
- **如果依赖readResolve进行实例控制，带有对象引用类型的所有实例域必须声明为transient**
    - 如果单例包含一个非瞬时的对象印象域，这个域的内容可以在readsolve方法运行之前就被反序列化
    - 盗用者类可以引用这个单例

### 修饰readResolve为final
- 不适用任何子类

## 90.考虑用序列化代理代替序列化实例

### 什么是序列化代理
1. 为可序列化的类设计一个私有的静态嵌套类，精确表示外围类的实例的逻辑状态
2. 有一个构造器，参数就是外围类
3. 构造器只从参数复制数据，所以不需要一致性检查或者保护性拷贝
4. 实现Serializable接口

### 示例
```java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private fianl Date end;
    
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
    
    private static final long serialVersionUID = 1L;
}

// 通过代理获得对象
private Object writeReplace() {
    return new SerializationProxy(this);
}

// 添加如下readObject方法可以抵御攻击者伪造代理
private void readObject(ObjectInputstram stream) throws InvalidObjectException {
    throw new InvalidObjectException("proxy required");
}
```

### 







# 附录
## 静态导包
- 例子
```java
import static java.util.stream.Collectors.*;
import static java.lang.System.*;
```
- 与不同包管理资源背道而驰,基于便捷考虑
- 如果导入的静态方法有冲突则需要重新考虑


## 程序
### supplier官方说明
``` 
/**
 * Represents a supplier of results.
 *
 * <p>There is no requirement that a new or distinct result be returned each
 * time the supplier is invoked.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #get()}.
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {
 
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

### BinaryOperator的自定义
```
  BinaryOperator<Integer> add = (x, y) -> x + y + 1;
  // output 4
    System.out.println(add.apply(1, 2));
```

### Comparator.comparing()使用
```
// 1.使用已经定义好的比较器
list.sort(Comparator.comparing(Album::sales));

// 2.使用自定义的
list.sort(Comparator.comparing(
    Album::artist, (x, y) -> {
    // 会提示没有0 排序而言我觉得无所谓
        return x.length > y.length ? 1 : -1;
    }
));

// 综上所述 comparing用法：
// 1.Comparator.comparing(keyExtractor)
// 2.Comparator.comparing(keyExtractor, keyComparator)
```

### Collectors.groupingBy()用法
```
// 1.直接分组
// 对于基本类型使用Function.identity()
// 不常用
Collector.groupingBy(Album::getSales);

// 2.加上分组之后产生的
// 也就是Map<主键, 产生什么>
// 第二个参数必须是Collector类型
// 最常用
Collector.groupingBy(Album::getSales, Collector.counting());

// 3.在2的基础上加上对产生的map做具体类型限制
Collectors.groupingBy(Album::getSales, HashMap::new, Collector.counting());
```

### System.currentTimeMillis还是System.nanoTime
- `System.currentTimeMillis`是1970年1月1日至今的时间毫秒
- `System.nanoTime`是基于虚拟机的时间，纳秒为单位
    - 由于基于虚拟机时间，所以只有同一程序内才有意义 
- 都是本地方法，但是1可以更改本机事件实现变化
- 1可以和date等互相转换，2不行
- 2更适合用于高精度计算，比如计算程序运行时间等













