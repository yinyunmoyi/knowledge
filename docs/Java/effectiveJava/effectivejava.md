## effective java（一）：创建和销毁对象

### 优先考虑静态工厂方法创建对象

它是一个返回类的实例的静态方法：

~~~java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.False;
}
~~~

使用静态工厂方法建立对象和使用构造方法的对比如下：

|       |            静态工厂方法            |      构造方法      |
| :---: | :--------------------------: | :------------: |
|  命名   |         可以用方法名表达更多信息         |     方法名固定      |
|  性能   |    可以把对象缓存好然后返回出来，类似享元模式     |   每次调用必创建新对象   |
| 返回类型  |      可以适用多态，具体返回什么类不确定       |     返回类型固定     |
| 返回类型2 |       返回的类型可以根据参数不通变化        |     返回类型固定     |
| 返回类型3 |      返回的实际类型可以在编写阶段不存在       | 返回的类型在编写阶段必须存在 |
|  ？？   | 类如果不含公有或protected的构造器就不能被子类化 |                |
|  易用性  |         难以在API文档中发现它         |  发现很简单因为方法名固定  |

静态方法的惯用名称：from（转换）、of（聚合）、valueOf、getInstance、create/newInstance（每次都要创建一个新对象）、getType1/newType1（返回一个实现类为type1的对象）

### 多参数时的选择：构造器模式

当我们需要使用 很多参数构造一个对象时，通常的选择有以下两种：

1、重叠构造器模式：在内部封装多个构造器互相调用，如提供一个简单的构造器只有一个必要的参数，最后调用其中最复杂的一个构造器，问题在于复杂的构造器难以使用，很容易搞错参数位置，且难以阅读。

2、JavaBeans模式：将对象new出来之后用setter来设置它的属性，这样使用即简单，可读性也好，但是问题在于除了对象使用者以外，其他调用者不知道何时这个对象才算构造完全，才能够投入使用，很容易去使用一个处于不一致状态的对象。

合适的做法是使用构造器模式：

~~~java
NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
~~~

当处理更多的参数时，Builder模式是更好的选择，大致意思是入参构造参数只有一个builder对象，根据这个对象来生成新对象。

### 合适的单例设计

设计一个单例有以下几种方式：

1、公有静态成员变量：

~~~java
public static final Class1 INSTANCE = new Class1();
~~~

2、静态工厂方法：

~~~java
private static final Class1 INSTANCE = new Class1();
public Class1 getInstance() {return INSTANCE};
~~~

3、包含一个元素的枚举类型

使用单例模式的时候需要注意，有可能其他客户端会利用反射调用私有的构造方法，此时可以在私有构造里抛出异常禁止构造。

如果想要把单例类变成可序列化的，实现Serializable，且一定要重新readResolve方法，把实例域都标记为transient的，否则每次序列化时会重复创建一个新的实例。

三种方法的对比如下：

|        |   公有静态成员变量   |         静态工厂方法          |         包含一个元素的枚举类型          |
| :----: | :----------: | :---------------------: | :--------------------------: |
|  易维护   | 要修改成非单例要改API |        容易修改为非单例         |           容易修改为非单例           |
|  扩展性   |      固定      |       可以写一个带泛型的工厂       |              固定              |
|  扩展性   |     不能提供     | 可以通过方法引用来提供单例（Supplier） |             不能提供             |
| 安全和序列化 |   需要人为提供措施   |        需要人为提供措施         | 免疫反射攻击、天然提供序列化机制（枚举只序列化name） |
|  扩展性   |     可以继承     |          可以继承           |           不能继承自某类            |

### 优先依赖注入来引入资源

静态工具类和单例类因为依赖某些资源，将这些资源设计成一个final成员变量进行默认初始化，这样的做法是不合适的，原因是这些资源很难固定不变，一旦遇到资源变化的情况这种设计就难以变更了，正确的做法是将这些资源作为构造方法的参数传入，这就是依赖注入模式。

使用依赖注入模式可以实现同一个底层资源传入不同的业务对象，它们具有不可变性。

依赖注入的一种变体是在构造方法中传入一个资源工厂，在构造方法中利用资源工厂来构造成员变量。

当需要注入的变量过多时，会影响项目的灵活性和可测试性，解决办法是用依赖注入框架，如spring

### 避免重复创建对象

一般来说最好能重用单个对象而不是每次都创建新的对象，需要注意的细节点：

1、有静态工厂方法的情况下不要使用构造器，如：

~~~java
Boolean.valueOf(String)优于new Boolean(String)
~~~

2、缓存一些昂贵的对象，如正则的Pattern、数据库的连接对象

3、如果想安全的使用一个不可变的对象，可以考虑使用视图，或者适配器，创建多个视图都是源于同一个源对象

4、避免无意识的自动装箱和拆箱，这会在一定程度上影响性能

避免重复创建对象是重要的，但是不要为了避免重复创建对象牺牲可读性造成过度设计，现代的JVM对于小对象的创建和回收动作都是很廉价的。

不要轻易的维护缓存对象池，这会增加代码的复杂度，除非创建它的代价真的很昂贵

### 消除过期的对象引用

对于java这种垃圾自动回收的语言，容易在不经意间触发内存泄露，如用数组实现一个栈，当栈增长时指针前移，栈弹出后指针后移，指针后移之后程序使用完毕栈弹出的对象，该对象也不会被垃圾回收器回收，原因是栈中还在引用它，解决办法是当弹栈之后对于以后用不到的位置赋值为null。

类似这种类自己管理内存的情况，应该警惕内存泄露问题。

弱引用的意思是如果没有其他额外引用，则该对象在下一轮就会被垃圾回收期回收，比如使用WeakHashMap

### 避免使用终结方法finalizer和cleaner

使用终结方法的问题：

1、它们不保证对象会被及时清除，不保证清除优先级、不保证最后一定会被清除

2、终结方法会阻止正常的垃圾回收，导致有严重的性能损失

3、有安全问题，可能会引发终结方法攻击

使用它们的场景：

1、使用资源时忘记调用close方法时，可以用它们当做安全网。虽然不保证及时执行，但总比没有好

2、本地对等体（非java的对象）的回收

### try-with-resources 优先于 try-finally

使用try finally的主要劣势在于：在finally中调用close抛出异常时，会抹除第一个异常，给定位带来很大困难，但是使用try-with-resources就不会，在try块中初始化的部分必须实现AutoCloseable接口：

~~~java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
}
~~~

此时即使readLine和close方法（不可见的）都抛出了异常，两个都会打印在堆栈轨迹中，后一个异常会标记为禁止状态。而且这种方式后面也可以捕捉异常，捕捉到异常后处理错误后的逻辑：

~~~java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
} catch (IOException) {
    return defaultVal;
}
~~~

## effective java（二）：Object类的方法

### 覆盖equals方法

当一个类需要定义一种逻辑相等的概念时，就需要重写equals方法

重写它的时候要遵守以下几个约定：

1、自反性：x.equals(x)必须返回true

2、对称性：对非空的x和y，当y.equals(x)返回true时，x.equals(y)也要返回true

3、传递性：x和y相等，y和z相等，x和z也要相等

4、一致性：属性不改变的情况下多次调用的结果一致

5、非null的值和null比较，结果是false

重写equals的常规步骤：

1、使用==检查对象是否是自己

2、使用instanceof来检查入参是否是正确的类型

3、把入参进行类型转换

4、每个逻辑相等的部分都要分别比较

注意当float和double比较时要用Float.compare方法，double比较时要用Double.compare方法

数组比较可以用Arrays.equals方法，包含null值的比较可以用Objects.equals方法

重写equals的时候一定要注意入参是Object，否则就会重写错方法

### 重写equals时一定要重写hashCode方法

重写hashCode遵循的原则是：如果两个对象调用equals是相等的，执行hashCode也必须要相等；如果两个对象调用equals是不相等的，那么执行hashCode不相等的几率最高，散列表性能越好

当自定义一个hashCode方法时，如果不太注重性能，可以直接使用Objects类的散列函数：

~~~java
return Objects.hash(lineNum, prefix, areaCode);
~~~

如果要自己写hashCode方法，首先要确定对象中的关键域，这些关键域是决定对象equals的返回结果的域，注意要排除一些衍生域（可以由其他关键域计算出来的那些），然后对每一个关键域都确定它的散列码：

1、如果是基本类型就用对应类型的hashCode方法，如Integer.hashCode(val)

2、如果是对象就调用对象的hashCode方法

3、如果是数组则对每一个元素都当做单独的域来处理，如果数组中没有重要的元素可以用一个非0变量来代替

对于计算出的每个散列码用下列迭代式来计算：（用31是因为它是奇数，乘法溢出不会丢失信息；且31 * i = (i <<5) - i，优化器会做这样的优化，至于使用素数的好处并不明显）

~~~java
int result = 31 * result + c;
~~~

例如下列的hashCode方法：

~~~java
int result = Short.hashCode(areaCode);
result = 31 * result + Short.hashCode(prefix);
result = 31 * result + Short.hashCode(lineNum);
return result;
~~~

当计算一个对象的hashCode代价很昂贵时，可以把散列码当做成员变量缓存在内部，用延迟初始化的方式来加载：

~~~java
private int hashCode;

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        ...
        hashCode = result;
    }
    return result;
}
~~~

### 谨慎覆盖clone方法

实现Cloneable接口表明对象允许克隆，但是并不强制要求重新clone方法，实现该接口后，在类内部调用clone方法就会返回该对象的逐域拷贝（Object的clone方法是受保护的），如果没有实现该接口，调用clone就会抛出CloneNotSupportException

一般来说，如果每个域包含一个基本类型的值，或者包含的对象都是不可变的，此时类重写clone方法只需要简单的执行父类的clone方法即可：

~~~java
@Override
public PhoneNumber clone() {
    return (PhoneNumber) super.clone();
}
~~~

clone方法在执行过程中，不能在方法内部调用其他可以被覆盖的方法。

当包含对象域的时候，在写clone方法时就应该先调用父类的clone方法，然后再去深拷贝那些可变的对象。

对于数组来说最好的办法是使用clone方法，除此之外，其他更复杂的对象的克隆更推荐采用拷贝构造器（入参和返回值类型相同的构造器）或者拷贝工厂（入参和返回值类型相同的静态方法）

### Comparable接口

当一个数据类内部有明显的内在排序关系时，就可以考虑实现该接口

在判断时注意不要直接使用两个数相减，相减是有可能会出现溢出的，此时应该用一个静态方法compare：

~~~java
return Integer.compare(o1, o2);
~~~

## effective java（三）：类和接口

### 使类和成员的可访问性最小化

关于可访问性，几个禁止的点：

1、公有类的实例域不能是公有的，静态域可以有公有的常量

2、公有的常量如果是数组或者对象，那就有被外部类修改的风险

一种方法是将公有变量调整为公有的不可变对象：

~~~java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
~~~

还有一种方法是公有方法返回其拷贝：

~~~java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
~~~

### 公有类不应该直接暴露数据域

有些只表示数据的类没有get和set方法：

~~~java
class Point {
    public double x;
    public double y;
}
~~~

这种类的数据域是直接可以访问的，对于包私有的或者私有的嵌套类，这样定义是可以的，但是对于公有类来说，不应该直接暴露数据域，如果暴露数据域，意味着直接访问其成员变量的代码将散步在各处，将来如果想改变其内部的表示法，将会连锁修改所有引用它的地方，给维护带来很大困难

### 谨慎使用继承，对继承做好文档说明

父类的文档必须精确地描述覆盖每个方法带来的影响，对于每个public和protected的方法和构造器都必须指明它调用了哪些可覆盖（public或protected）的方法，是以什么顺序调用的，每个调用结果是怎么影响后续处理的。

如果方法调用了可覆盖的方法，就应该在方法注释上加上@implSpec，也就是实现要求。例如remove方法的实现要求就应该提醒：由集合的iterator方法返回的迭代器没有实现remove方法，该实现会抛出异常。

好的API文档应该说明它做了什么工作，而不是描述它如何做到的，而上述的说明破坏了这个准则，这就是继承破坏了封装性带来的后果。

继承过程中需要注意的点：

构造器不能调用可被覆盖的方法。因为超类的构造器在子类的构造器前运行，子类覆盖的方法可能在子类构造器调用前被调用，这可能会使程序失败。同样的道理，clone方法（子类clone前被调用）和readObject方法（反序列化前被执行）也是一样。

做好继承是很难的，父类发布后，文档建成后就必须要一直遵守。所以非必要时要禁止其他类继承本类，可以用之前的final和私有化构造器来实现。

### 接口优于抽象类

接口是允许多个实现的最佳途径，是定义mixin(混合类型)的理想选择，mixin的意思就是类可以实现某个类型，表面它提供某种能力，如Comparable代表提供了对比的能力，它允许任选的功能，可以混合到类型的主要功能中。

接口使用起来比抽象类更加简单，它不会破坏类本身的体系，可以很容易的实现和扩展。如果用抽象类来代替接口的功能，将使得所有的子类都被迫实现这些功能。

包装类结合接口可以很安全的增强类的功能。

接口配合骨架实现类可以完成强大的功能，骨架实现类就是一个提供基本功能实现的抽象类。无论是骨架实现类还是接口的默认方法，都需要提供文档进一步说明。

### 谨慎添加默认方法

接口中的默认方法最好是在一开始就设计好的，如果在开发的过程中想要给接口添加一个默认方法，就意味着实现接口的类都可以调用它，可能会产生一些错误，因为不是所有类都能一定适应这个新方法。如子类中对元素状态进行了定义，但是接口中的默认方法对此就一无所知。所以要谨慎设计接口，尽量不要给现有的接口添加默认方法。

### 使用常量：不要使用常量接口

不包含任何方法，只包含静态final域的接口被称为常量接口，实现这些接口可能会让其他类使用这些不必要的常量，未来某个类不需要这些常量的时候，也不能将它取消掉。

使用常量可以定义一个final类，也可以把常用的常量添加到某个类中，如Integer中的MIN_VALUE，还可以用枚举。

使用常量时可以使用静态导入，这样就可以避免类名修饰常量名。

### 4种内部类

4种内部类：静态成员类、非静态成员类、匿名类和局部类

静态成员类相当于一个单独的类，它只是被声明在某个类的内部，它的常见用法是作为公有的辅助类，只有和外部类一起使用才有意义，如Map.Entry、Calculator.Operation

非静态成员类和静态成员类的区别：

非静态成员类内部隐含外部类的实例，相当于关联了一个外部类this的引用，当非静态成员类的实例创建出来的时候，它和外围实例的关联关系也随之被创建，这种关联关系会消耗非静态成员类的空间，增加构造的时间开销。这个引用很隐蔽，可能导致外围类不会被回收，导致内存泄漏。

如果成员类不要求访问外围实例，就要始终用静态成员类。

匿名类是一个只有声明的时候实例化的类，出现在非静态的环境中，它可以访问外围实例。无法声明一个匿名类实现多个接口或者拓展一个类。除了从超类型中继承之外，不能调用任何成员。它应该尽可能简短，否则会影响可读性。

局部类使用较少。属于方法内部的类，需要只在一个地方创建实例时就使用匿名类，否则就用局部类。

### 不要在一个文件中定义多个顶层类

java语法允许在一个源文件中定义多个顶级类，但是不要这么做，因为当多个文件中含有同名的顶级类时，在编译命令（javac）中会让靠前的类会覆盖靠后的类，程序的行为受到编译顺序的影响，难以接受。

解决办法就是将顶级类放入独立的源文件中。

## effective java（四）：泛型

略

## effective java（五）：枚举和注解

### 枚举的基本使用规则

枚举天生就是不可变的，因此所有的域都应该是final的

枚举有一个静态的values方法，它可以按照定义顺序返回所有枚举值的数组。

当枚举可以被外部包使用时，它应该是公有的；若只会用在枚举类型的定义类或者所在包中，此时它应该是包私有或者私有的。

当每个枚举值想实现不同的方法逻辑时，可以把枚举类中的方法定义为抽象的，让每个枚举值来实现它（不推荐用switch的方法来实现，因为switch拓展枚举值必须对这部分逻辑同步修改，如果忘记会比较麻烦）：

~~~java
public enum Operation {
    PLUS("+") {
        public double apply(double x,double y){return x+y;}
    },
    MINUS("-") {
        public double apply(double x,double y){return x-y;}
    },
    TIMES("*") {
        public double apply(double x,double y){return x*y;}
    },
    DIVIDE("/") {
        public double apply(double x,double y){return x/y;}
    };
    
    private final String symbol;
    
    Operation(String symbol) {this.symbol = symbol};
    
    public abstract double apply(double x, double y);
}
~~~

当多个枚举值采用一种方法逻辑，另外一些采用其他的方法逻辑时，考虑在枚举内部定义一个枚举（也不推荐用switch）：

~~~java
enum PayrollDay{
    MONDAY,TUESDAY,WEDNSDAY,THURSDAY,FRIDAY,
    SATURDAY(PayType.WEEKEND),SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {this.payType = payType}
    PayrollDay() {this(PayType.WEEKDAY);}
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                xxx
            }
        },
        WEEKEND{
            int overtimePay(int minutesWorked, int payRate) {
                xxx
            }
        }
        
        abstract int overtimePay(int minutesWorked, int payRate);
        
        int pay(int minutesWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
~~~

枚举中的switch适合用来转换枚举值类型，若业务逻辑需要依赖一个外部的枚举类，这个枚举类对你而言是不可控制的，此时应该自定义一个枚举，在枚举的方法中传入一个外部枚举类值，然后用switch根据不同的枚举值来返回枚举类型。

所有的枚举都有一个ordinal方法，用来返回该枚举在类型中的数字位置，但是最好不要使用它，它是专门为EnumSet和EnumMap这种结构准备的，因为枚举在类型中的位置非常难以维护，容易弄错。

### EnumMap

略

### 用接口模拟可扩展的枚举

可以利用接口来扩展枚举：

~~~java
public interface Operation{
    double apply(double x, double y);
}

public enum Operation implements Operation{
    PLUS("+") {
        public double apply(double x,double y){return x+y;}
    },
    MINUS("-") {
        public double apply(double x,double y){return x-y;}
    },
    TIMES("*") {
        public double apply(double x,double y){return x*y;}
    },
    DIVIDE("/") {
        public double apply(double x,double y){return x/y;}
    };
    
    private final String symbol;
    
    Operation(String symbol) {this.symbol = symbol};
    
    public abstract double apply(double x, double y);
}
~~~

### 注解的原理及使用

略

### 坚持使用Override注解

### 标记接口和标记注解

略

## effective java（六）：Lambda和Stream

### Lambda基本使用原则

java8中，有一种特殊的接口是函数式接口：它是带有单个抽象方法的接口，java8允许lambda表达式创建这些接口的实例。

lambda允许在大多数情况下省略参数的类型，除非程序无法自动推导，写lambda时应该尽可能省略这些参数类型，以突显其简洁，除非它们的存在能使程序变得更清晰。

lambda的存在可以简化好多程序，比如上面的一个枚举使用案例：

~~~java
public interface Operation{
    double apply(double x, double y);
}

public enum Operation implements Operation{
    PLUS("+") {
        public double apply(double x,double y){return x+y;}
    },
    MINUS("-") {
        public double apply(double x,double y){return x-y;}
    },
    TIMES("*") {
        public double apply(double x,double y){return x*y;}
    },
    DIVIDE("/") {
        public double apply(double x,double y){return x/y;}
    };
    
    private final String symbol;
    
    Operation(String symbol) {this.symbol = symbol};
    
    public abstract double apply(double x, double y);
}
~~~

如果用lambda表达式重构后：

~~~java
public enum Operation {
    PLUS("+", (x,y) -> x + y),
    MINUS("-", (x,y) -> x - y),
    TIMES("*", (x,y) -> x * y),
    DIVIDE("/", (x,y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
~~~

其中DoubleBinaryOperator是java的function包中预定义的函数式接口，它接受两个double参数，返回一个double参数。

lambda表达式的一大劣势是它没有方法名，也没有文档，如果一个lambda计算本身过于复杂，就会对程序的可读性造成严重损害。

lambda和匿名类相比，必须使用匿名类的场景：

1、创建抽象类的实例

2、创建带多个抽象方法的实例

3、lambda无法获得对自身的引用，在lambda中this是外层实例，而在匿名类中this是值匿名类的实例。如果需要获取自身引用并操作，则必须用匿名类

匿名类和lambda一样，都不能实现序列化和反序列化。

### 方法引用的使用场景

方法引用在大多数时候都比lambda表达式更简洁，如：

~~~
map.merge(key, 1, (count, incr) -> count + incr);

用方法引用改写：
map.merge(key, 1, Integer::sum);
~~~

如果lambda表达式过于复杂，可以将逻辑抽出独立成一个方法，然后使用方法引用。

不过也有方法引用显得更繁琐的场景，那就是调用本类的方法时，此时使用lambda表达式更简洁，如果用方法引用则还要加上类名：

~~~java
service.execute(() -> action());
~~~

### 坚持使用标准的函数接口

java.util.Function中有43个接口，他们都是标准的函数接口，应该尽可能使用这些接口，不应该自己定义函数式接口，因为标准函数式接口都支持基本类型，不要用带包装类型的自定义函数式接口替代它，否则会导致严重的性能问题。

坚持使用@FunctionalInterface来标注自定义的函数式接口。

### 谨慎使用Stream

stream通常是lazy的，直到调用终止操作才会开始计算，一个没有终止操作的stream是没有意义的。stream可以使程序变得简洁清晰，但是若使用不当则会使程序变得混乱且难以维护。

简洁清晰使用stream的建议是仔细命名lambda的参数，而且在lambda中多用方法去封装，而不是用一些复杂的流API。

stream只支持三种基本类型：int、long和double，最好避免利用stream来处理char值，字符串的chars方法返回的stream中的元素不是char值，而是int值。

迭代和使用stream对比：

|                       |             普通迭代              |             stream              |
| :-------------------: | :---------------------------: | :-----------------------------: |
|         局部变量          |           可以修改或者读取            |       只能读取final变量，且不能修改变量       |
|          调度           | 在外围方法中可以return/continue/break |           无法控制外围方法的流程           |
| 流处理（包括转换/过滤/合并/分组/搜索） |              不擅长              |               擅长                |
|     访问流程中处理过的某个元素     |              擅长               | 不擅长（要找到流处理中处理前的元素，需要建立包含新旧对象的对） |

###优先选择Stream中无副作用的函数

略

### Stream优先用Collection作为返回类型

略

### 谨慎使用Stream并行

Stream的并行不仅可能会降低效率，还可能把程序卡死、计算错误，任何的并行都要严格的测试后再使用

## effective java（七）：方法

### 检查参数的有效性

方法要做的尽可能通用，但是很多情况下参数是有限制的，此时有必要对方法入参进行检查，检查失败抛出对应的异常。要警惕一些非法参数的场景下，程序还能正常运行的情况，此时遇到问题就很难定位了

相关的检查方法：

~~~java
Objects.requireNonNull()   若为空则抛出空指针
java9中增加：
Objects.checkFromIndexSize();
Objects.FromToIndex();
Objects.checkIndex();
~~~

### 必要时进行保护性拷贝

当一个类中有固定的约束条件时，就要防止外部程序破坏这个条件。而类中使用对象，则很容易无意识的破坏这种保护条件，比如有个类Period中有两个成员变量start和end，这两个都是Date类型的，Period类要求内部的start<end，Period类在构造方法中限制这个条件：

~~~java
public Period(Date start, Date end) {
    if (start.compateTo(end) > 0) {
        throw new IllegalArgumentException(start + "after" + end);
    }
    this.start = start;
    this.end = end;
}
~~~

以下代码就会破坏这个类的内部条件：

~~~java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78)
~~~

修复这个问题的方法有两个，要么用不可变类Instant或LocalDateTime、ZonedDateTime代替Date（Date已经过时了，因为它是可变的，但是日期本身应该是不可变的概念），要么对构造器的每个可变参数进行保护性拷贝：

~~~java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (start.compateTo(end) > 0) {
        throw new IllegalArgumentException(start + "after" + end);
    }
}
~~~

这样就能保证类中的成员变量是完全独立的对象，对外部对象的修改影响不到类的规则，此外还有一点要注意，那就是有效性检查是针对拷贝后的对象的，这是为了防止从检查到拷贝这段时间内程序对外部对象的修改，此时外部对象的修改还是有可能影响到类的行为，这就是TOCTOU攻击。

保护性拷贝不能调用外部对象的clone方法，因为Date是非final的，恶意的子类完全有可能重写一个危险的clone方法。但是在Period内部对成员变量调用clone方法是安全的，因为保护性拷贝已经保证了成员变量是可靠的类，它的初始化过程掌握在Period类中。

除了针对构造器的攻击，还可以通过get方法攻击，将成员变量get出来并直接修改它的值：

~~~java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
~~~

为了抵御这种攻击，要对get方法进行改造，让它返回内部变量的保护性拷贝：

~~~java
public Date end() {
    return new Date(end.getTime());
}
~~~

这样即使修改外部变量，类内部的规则也不会被破坏。

保护性拷贝思想就是对于外部传来的对象和即将返回给外部的对象进行的一种防护手段，但是保护性拷贝不可避免的会降低性能，当客户端和类互相信任时，就可以不做保护性拷贝。还有一种方法，就是使用不可变对象来作为类的成员变量。

### 一些设计方法的建议

1、避免过长的参数列表。四个参数或者更少的参数比较便于理解和使用。相同类型的参数格外有害，因为使用它的用户很难记住它的顺序，即使顺序搞错方法依然可以正确调用。

减少参数列表的方法主要有三：

（1）将方法分解成多个方法，把参数拆分开，一部分在一个方法处理，一部分在另一个方法处理，多封装一些基础功能的方法，然后更复杂的方法通过这些方法的组合来实现。当一种功能需要构造一个很长参数的方法时，推荐采用把方法的功能拆开，然后供外部调用，比如用集合的subList和indexOf方法，可以找到子集合的第一个和最后一个索引，无需构造一个更多参数的方法

（2）创建辅助类来保存参数的分组，这些辅助类常常是静态成员类（有多个类用到时可以独立出来），比如各类context

（3）用Builder模式来链式调用，然后调用execute方法

2、方法的参数类型优选接口，不要选择具体类，扩展性更好

3、方法的boolean参数，优先使用两个元素的枚举类型，它更易于阅读和编写，而且可扩展。

### 慎用重载

当定义了两个同名方法，一个入参是List，一个入参是Collection，此时到底最后调用哪个方法是编译时确定的，也就是当入参的类型是Collection时（无论它真正的类型是Set还是List），就会选择调用参数类型是Collection的方法，这是很反常理的。

当子类重写了父类的同名方法，此时会自动选择子类实现的方法，这就是运行时决定方法调用。

故为了避免重载时方法预期结果不一致的问题，应该尽量避免使用重载，重载时应该尽量避免参数个数一样，还可以用给方法起不一样的名字避开重载。

使用多个构造器时就会不可避免的使用重载，此时应该尽量使用工厂方法模式

### 慎用可变参数

可变参数可以接受0个或者多个指定类型的参数，可变参数会创建一个数组，数组的大小就是参数的数量，所以可以总是用数组的数量来检查入参：

~~~java
if (args.length == 0) {
    throw new RunTimeExcetpion();
}
~~~

为了避免运行时出现异常，可以设置一个默认值，然后后面设置一个可变参数：

~~~java
static int min(int defaultVal, int... args)
~~~

在重试性能的情况下使用可变参数要小心，因为每次调用可变参数方法都涉及一次数组分配和初始化，有一个常见的性能优化是将常用的几个参数重载为固定参数个数的方法，然后最后用一个可变参数方法来兜底：

~~~java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
~~~

### 方法尽量不要返回null

方法返回null相当于把空指针的风险处理丢给调用代码的客户端，会埋下风险。

如果每次为了避免返回null而new一个空集合或数组会损害性能，可以考虑返回Collections.emptyList或者Collections.emptyMap，或者把空数组提前建好，然后作为一个成员变量返回：

~~~java
private static final int[] ARRAY = new int[0];
~~~

### 谨慎返回Optional

有一种替代返回null的方案是返回Optional，它可以存放一个对象，也可以是一个空内容。把原来返回空的地方设置为返回Optional.empty()即可，永远不要通过返回Optional的方法返回null，这会完全违背它的本意。

调用端拿到返回值Optional后，可以选择获取一个默认值：

~~~java
result.orElse("default");
~~~

也可以选择抛出一个异常：

~~~java
result.orElseThrow(异常工厂);
~~~

也可以提前判断它是否有值，若有直接取出值，若调用get时里面是空则会抛出NoSuchElementException。

有很多流中包含Optional：Stream<Optional<T\>>，可以用下列方式取出其中的元素：

~~~java
stream.filter(Optional::isPresent).map(Optional::get);

java9
stream.flatMap(Optional::stream);
~~~

使用Optional的场景是：无法返回结果，且当无法返回结果时客户端必须特殊处理时，就要返回Optional<T\>，尤其是涉及返回基本类型时，此时Optional的泛型类型是OptionalInt、OptionalLong、OptionalDouble，也可以返回包装类型Boolean、Byte等

不要用Optional作为映射键和值、作为集合或者数组中的元素

注意Optional的性能开销，它有两级包装，在注重性能的场景不要使用它，应该直接返回null或者抛出异常。

## effective java（八）：通用编程

一些通用的建议：

* 局部变量应该在它定义的位置初始化，并且不能和使用的地方离得太远
* for循环优先于while循环，它更简洁，内部定义的变量作用域更小，不容易犯错
* 增强的for语句强于普通for，它更简洁；除非遍历操作涉及删除集合元素（可以用集合的remove方法，也可以用Collection的removeIf方法）、修改集合元素、更复杂的下标处理。
* 随机数生成：0-1使用new Random().nextInt()、生成0-n使用Random.nextInt(n)、更快的产生随机数ThreadLocalRandom、对于并行程序使用SplittableRandom、
* 如果想计算精确的结果，不要使用float和double，而是使用int、BigDecimal、long。BigDecimal还可以控制进舍的精细策略
* 基本类型优先于装箱类型。装箱的值要用equals来比较，不用用==，尤其是不要自己实现装箱类型的Comparator，而是要使用Comparator.naturalOrder()。要注意装箱的值若未初始化是null，直接用自动拆箱会抛出空指针；频繁的装箱和拆箱会有性能问题
* 不要在循环中进行字符串拼接，要用StringBuilder，会有性能问题，为连接n个字符串而重复的使用+，需要n的平方级的时间

## effective java（九）：异常

一些关于异常的建议：

* 尽量重用已经封装好的异常：IllegalArgumentException（参数非法）、IllegalStateException（对象状态非法）、NullPointerException（空值异常）、IndexOutOfBoundsException（下标越界）、ConcurrentModificationException（并发修改异常）、UnsupportedOperationException（不支持的操作）
* 抛出与抽象层次对应的异常。高层的实现应该捕获底层抛上来的异常，然后进行封装，抛出更高层次的异常，以解释高层抛出的异常，这就是异常转译。可以使用initCause方法设置底层的异常，以便更高层来用getCause获取底层的原因，然后进行处理。

## effective java（十）：并发

略

## effective java（十一）：序列化

* 在新的系统中，不要尝试使用java原生的序列化机制，java原生的序列化机制有很多安全问题。如果要兼容旧系统的话，那就不要反序列化不被信任的数据，要使用java的对象反序列化过滤机制ObjectInputFilter，通过白名单来防范攻击
* 谨慎实现Serializable接口，它可能带来严重的安全问题，而且前期类的组件会序列化在磁盘中，后续版本都要带着这些信息，要不断考虑新旧版本是否能序列化兼容的问题、父类或者接口实现该接口，则所有子类都需要考虑该问题；不要让内部类实现Serializable接口，因为内部类持有外围作用域的信息，这些信息没有明确的规定，会出现异常的结果
* 多考虑自定义的序列化 形式，配合readObject、writeObject方法使用，默认序列化不灵活，想改变类的结构需要始终考虑和旧版本的兼容、和自定义序列化比默认的消耗时间和空间都更大。
* 用transient代表不会在默认序列化中序列化该变量，它不会再称为类状态的一部分
* 不管选择哪种序列化形式，都要显示的给可序列化的类一个UID，如果未提供，则会随机生成一个UID，若序列化时两边的UID不同，则会引发InvalidClassException失败，会有兼容性问题，且随机生成会消耗一点性能。若要为类生成一个新的版本，新版本和现有的类不兼容，则可以手动修改UID

自定义序列化的部分：略