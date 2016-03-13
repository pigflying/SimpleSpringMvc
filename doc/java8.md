#1.简介
随着<a href="http://www.oracle.com/technetwork/java/javase/8train-relnotes-latest-2153846.html">Java 8 release</a>的发布，对于Java的编译，工具，JVM等都引入了很多新的特性。所以应该尽快用代码来说明这些变化.

这里简单的分为下列模块：

- 语言
- 编译
- 文档
- 工具
- JVM

#2.Java新特性

> java 8可以说是一个很重要的release版本，甚至有些特性几乎是每一个java程序员所期盼的。这一节，基本覆盖了大多数特性。

###2.1 lambdas表达式与函数接口

获取lambdas是整个Java8最值得期待的一个特性。函数可以作为一个参数传递，或者说是代码当做数据一样。有过<a href="http://www.javacodegeeks.com/2014/03/functional-programming-with-java-8-lambda-expressions-monads.html">函数式</a>编程经历的或许更明白。例如JVM平台上的Groovy,<a href="http://www.javacodegeeks.com/tutorials/scala-tutorials/">Scala</a>(正在用scala开发项目，很多特性蛮不错的！)语言，本身就具有lambdas，java开发者只能使用匿名类。

lamdbas的设计开发花费了很多时间和社区努力，但总的来说带来了很多益处，使Java变得更简洁紧凑。简单的来讲，lambdas表达式就是一组参数和方法的集合，``->``就是分隔符，例如：

```
Arrays.asList( "a", "b", "d" ).forEach( e -> System.out.println( e ) );
```

需要注意下参数```e```，是编译不过的，需要说明e的参数类型。例如：

```
Arrays.asList( "a", "b", "d" ).forEach( ( String e ) -> System.out.println( e ) );
```

lambdas的内部还是比较复杂的，或许会给函数加上大括号，所以可以像函数一样使用。例如：

```
Arrays.asList( "a", "b", "d" ).forEach( e -> {
    System.out.print( e );
    System.out.print( e );
} );
```

lambdas可以引用类的成员变量和局部变量(如果不是final类型，会隐士的转换成final) 例如下面两个写法是一样的：

```
String separator = ",";
Arrays.asList( "a", "b", "d" ).forEach( 
    ( String e ) -> System.out.print( e + separator ) );
```
和
```
final String separator = ",";
Arrays.asList( "a", "b", "d" ).forEach( 
    ( String e ) -> System.out.print( e + separator ) );
```

lambdas可以有返回值，编译器可以自动推断返回值的类型。如果表达式是一行的话，return语句也是不需要的。下面两种写法一样：

A:
```
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) );
```

B:
```
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> {
    int result = e1.compareTo( e2 );
    return result;
} );
```

语言的设计者花了很多心思在如何使用现有的框架很好的兼容lambdas,所以就有<a href="http://www.javacodegeeks.com/2013/03/introduction-to-functional-interfaces-a-concept-recreated-in-java-8.html">函数接口</a>：接口只有一个方法，这个方法会自动的隐士转换成lambda表达式。java.lang.Runnable 和java.util.concurrent.Callable就是两个很典型的例子。实际上函数接口也是很脆弱的，如果有人加入了另一个方法，那么就不再是函数接口，而且会编译失败。为了解决这个问题，隐士的将这个接口声明称函数式，java 8增加了自动注入@FunctionInterface. 让我看一个简单的定义：

```
@FunctionInterface
public interface Functional {
	void method();
}
```

还有一个点也是很重要的：接口中default修饰的方法是不会打断函数式接口的。可以如下定义：

```
@FunctionalInterface
public interface FunctionalDefaultMethods {
	void method();
	
	default void defaultMethod() {
	}
}
```

更多类容，请参考：<a href="http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html">官方文档</a>

## 2.2 接口的默认(default字段修饰)和静态方法

java 8延续了接口的两个基本概念：默认和静态方法。<a href="http://www.javacodegeeks.com/2014/04/java-8-default-methods-what-can-and-can-not-do.html">静态方法</a>使得接口很像特性(traits，意思应该与scala中的traits，go,php中也有类似的概念)，但是还是有点不同。它允许给已经存在的接口添加新方法，而不需要重写使用了老的接口的代码。

抽象方法需要实现，而静态方法不需要实现--是静态方法与抽象方法最大的区别。接口必须提供静态方法的实现，接口的实现类会自动继承（需要的话，可以重写）。简单例子：

```
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or 
    // may not implement (override) them.
    default String notRequired() { 
        return "Default implementation"; 
    }        
}
        
private static class DefaultableImpl implements Defaulable {
}
    
private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```

java 8还提供了一个有趣的实现，接口能够声明(或者实现)静态方法。


```
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}
```

下面一个例子说明了默认方法和静态方法区别：

```
public static void main( String[] args ) {
    Defaulable defaulable = DefaulableFactory.create( DefaultableImpl::new );
    System.out.println( defaulable.notRequired() );
	    
    defaulable = DefaulableFactory.create( OverridableImpl::new );
    System.out.println( defaulable.notRequired() );
}
```

输出：

```
Default implementation
Overridden implementation
```

JVM上，默认方法很高效，方法调用的二进制指令也同样支持。默认方法允许已经存在的java接口进行扩展而不需要重构。很好的的例子是java.util.Connection接口中增加了strea(),parallelStream(),forEach(),removeIf()………………

虽说很强大，但是需要注意：使用默认方法时还是需要慎重，容易导致结构复杂和编译错误，尤其是在复杂的系统中。更多详情，请关注<a href="http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html">官方文档</a>

##2.3 方法引用

方法引用提供了一个很有用的语法--直接关联到已经存在的方法或者java类的结构或者实例。
结合着lambdas，方法引用似的java看上去很简明高效，不再那么死板。

接下来，以Car类为例，让我们看看四种方法引用的不同之处：

```
public static class Car {
    public static Car create( final Supplier< Car > supplier ) {
        return supplier.get();
    }              
        
    public static void collide( final Car car ) {
        System.out.println( "Collided " + car.toString() );
    }
        
    public void follow( final Car another ) {
        System.out.println( "Following the " + another.toString() );
    }
        
    public void repair() {   
        System.out.println( "Repaired " + this.toString() );
    }
}
```

第一种的方法调用是构造器的引用，使用了语法Class::new. 注意：构造函数是无参的。

```
final Car car = Car.create( Car::new );
final List< Car > cars = Arrays.asList( car );
```

第二种方法的调用是一个类的静态方法，使用了语法：Class::static_method.请注意这个方法接收一个Car类型的参数。

```
cars.forEach(Car::collide);
```

第三种方法的调用是一个实例的任意方法，语法：Class::method.注意这个方法并没有接收参数。

```
cars.forEach(Car::repair)
```

最后一种，特定类型实例的方法，语法：instance::method. 可以接收任何Car的实例。

```
final Car police = Car.create(Car::new);
cars.forEach(police::follow);
```
运行结果(实际的Car对象可能不相同)：

```
Collided com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
Repaired com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
Following the com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
```

获取更多实例或关于方法引用的详情，参考：<a href="http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html">官方文档</a>

##2.4 重复注解

从Java5注解被引入开始，已经变得很流行，也被广泛使用。使用上的一个限制是同一个注解在同一个地方不能够被重复。java打断了这种限制，引入了重复注解。他允许相同的注解用一个地方可以被重复使用。重复的注解应该先@Repeatable自己。实际上，并不是一个语法的变更，更多的是编译器的小技巧，实际是保持不变的。看一个实例具体下：

```
package com.javacodegeeks.java8.repeatable.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class RepeatingAnnotations {
    @Target( ElementType.TYPE )
    @Retention( RetentionPolicy.RUNTIME )
    public @interface Filters {
        Filter[] value();
    }
    
    @Target( ElementType.TYPE )
    @Retention( RetentionPolicy.RUNTIME )
    @Repeatable( Filters.class )
    public @interface Filter {
        String value();
    };
    
    @Filter( "filter1" )
    @Filter( "filter2" )
    public interface Filterable {        
    }
    
    public static void main(String[] args) {
        for( Filter filter: Filterable.class.getAnnotationsByType( Filter.class ) ) {
            System.out.println( filter.value() );
        }
    }
}

```

 class Filter就被@Repeatable注解了。然而，Filters仅仅是Filter注解的一个持有者，而且java编译器尽力对开发者隐藏这些细节。Filterable接口自然就有了Filter注解的两次定义。还有反射api提供了新方法getAnnotationsByType()来返回重复的注解。
 
```
filter1
filter2
```

更多详情，参考：<a href="http://docs.oracle.com/javase/tutorial/java/annotations/repeating.html">官方文档</a>

##2.5 更好的类型推断

java 8中提供了更好的类型推断。很多情况下，有编译器完成隐士类型的推断。例如：

```
package com.javacodegeeks.java8.type.inference;

public class Value< T > {
    public static< T > T defaultValue() { 
        return null; 
    }
    
    public T getOrDefault( T value, T defaultValue ) {
        return ( value != null ) ? value : defaultValue;
    }
}
```

```
package com.javacodegeeks.java8.type.inference;

public class TypeInference {
    public static void main(String[] args) {
        final Value< String > value = new Value<>();
        value.getOrDefault( "22", Value.defaultValue() );
    }
}
```

Value.defaultValue()的返回类型就不用提供了，编译器可以推断出来。然后java7中必须下成：Value.<String>defaultValue().

##2.6 扩展注解的支持

java 8支持了使用注解的上下文。现在，基本可以注解任何东西：变量，泛型，超累，实现接口，甚至方法的异常声明。下面是例子：

```
package com.javacodegeeks.java8.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collection;

public class Annotations {
    @Retention( RetentionPolicy.RUNTIME )
    @Target( { ElementType.TYPE_USE, ElementType.TYPE_PARAMETER } )
    public @interface NonEmpty {		
    }
		
    public static class Holder< @NonEmpty T > extends @NonEmpty Object {
        public void method() throws @NonEmpty Exception {			
        }
    }
		
    @SuppressWarnings( "unused" )
    public static void main(String[] args) {
        final Holder< String > holder = new @NonEmpty Holder< String >();		
        @NonEmpty Collection< @NonEmpty String > strings = new ArrayList<>();		
    }
}

```

ElementType.TYPE_USE和ElementType.TYPE_PARAMETER是两个新增的元素类型，用来描述注解上下文。为了识别新的注解类型在java中，Annotation Processing API也做出了小的调整。

#3 java 库中的新特性

java 8中添加了许多新类和拓展了很多存在的类，对时间，函数编程，现代并发等做更好的支持。

##3.1 Optional

NullPointerException在java应用错误中最常见的。很久之前，著名的<a href="http://code.google.com/p/guava-libraries/">Google Guava</a>项目介绍了Optionals来作为NullPointerExceptions，null check的一个解决方案,鼓励程序员编写简洁的代码。受 Google Guava的启发，Optional现在出现了在java 8库中。

Optional仅仅是一个容器：只能持有T类型的Value或者null。它提供了许多有用的方法，例如隐士的null check。关注<a href="http://docs.oracle.com/javase/8/docs/api/">java 8官方文档</a>了解更多详情

通过两个简单的列子看下Optional的使用，可为null的值，不可为null的值。

```
Optional< String > fullName = Optional.ofNullable( null );
System.out.println( "Full Name is set? " + fullName.isPresent() );        
System.out.println( "Full Name: " + fullName.orElseGet( () -> "[none]" ) ); 
System.out.println( fullName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
```

isPresent():如果Optional有非null值返回true，否则false。
orElseGet():如果Optional为null则通过回调提供产生默认值的函数。
Map(): 转换Optional的value,并且返回新的Optional对象。
orElse():与orElseGet()很相似，但是它接受默认值而不是函数。

输出：

```
Full Name is set? false
Full Name: [none]
Hey Stranger!
```
另外一个例子：

```
Optional< String > firstName = Optional.of( "Tom" );
System.out.println( "First Name is set? " + firstName.isPresent() );        
System.out.println( "First Name: " + firstName.orElseGet( () -> "[none]" ) ); 
System.out.println( firstName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
System.out.println();
```

输出：

```
First Name is set? true
First Name: Tom
Hey Tom!
```

更多细节 参考：<a href="http://docs.oracle.com/javase/8/docs/api/java/util/Optional.html">官方文档</a>

##4.2 流

新增<a href="http://www.javacodegeeks.com/2014/05/the-effects-of-programming-with-java-8-streams-on-algorithm-performance.html">流式API</a>介绍了java中真正的函数式变成。为了让java开发者写出更高效，简介的代码，这也算是到目前为止最大的java库变化。 流API使得集合操作更简单(当然不仅限于集合)。让我们从简单的Task类开始吧。

```
public class Streams  {
    private enum Status {
        OPEN, CLOSED
    };
    
    private static final class Task {
        private final Status status;
        private final Integer points;

        Task( final Status status, final Integer points ) {
            this.status = status;
            this.points = points;
        }
        
        public Integer getPoints() {
            return points;
        }
        
        public Status getStatus() {
            return status;
        }
        
        @Override
        public String toString() {
            return String.format( "[%s, %d]", status, points );
        }
    }
}
```

```
final Collection< Task > tasks = Arrays.asList(
    new Task( Status.OPEN, 5 ),
    new Task( Status.OPEN, 13 ),
    new Task( Status.CLOSED, 8 ) 
);
```

Task包含了一个point,能够Open或者Closed.第二段代码是一个简单的Task集合。
如果要统计集合中有多少个Task是Open的，很容易想到foreach操作。但是在java8中应该想到流--一个序列元素支持聚合和并行操作。例如：

```
// Calculate total points of all active tasks using sum()
final long totalPointsOfOpenTasks = tasks
    .stream()
    .filter( task -> task.getStatus() == Status.OPEN )
    .mapToInt( Task::getPoints )
    .sum();
        
System.out.println( "Total points: " + totalPointsOfOpenTasks );
```

输出：

```
Total points: 18
```

两个处理步骤需要注意下。第一，把集合转换成的流，以及过滤掉了Closed状态的Task.第二步，通过mapToInt操作，调用Task::getPoints将Tasks转换成了，Integer流。最后求Sum.

在看下一个例子之前，需要注意流(<a href="http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps">更多细节</a>)操作被分为了中间（例：filter）和终端（例：sum）.

中间操作返回一个新的流。比较懒，每次都是产生一个新流，而不是在原有的上面真正的处理（例：filter）,不过最终操作的还是初始流所包含的元素。

终端操作，例如foreach或者sum, 可能产生一个结果或者副作用。执行了终端操作，就认为流管道是被消费国的，不再被使用。在大多数情况下，终端操作是最终目的，完成原始数据的处理。

另一个点是：流是不支持并行操作的。来看个例子：

```
// Calculate total points of all tasks
final double totalPoints = tasks
   .stream()
   .parallel()
   .map( task -> task.getPoints() ) // or map( Task::getPoints ) 
   .reduce( 0, Integer::sum );
    
System.out.println( "Total points (all tasks): " + totalPoints );
```
输出：

```
Total points (all tasks): 26.0
```

和上面的例子很相似，区别是我们想让他并行计算，使用了reduce方法。

通常我们需需要分组一个集合的数据通过一个特定的分隔条件。流可以很简单的帮我们做这个事情。例：

```
// Group tasks by their status
final Map< Status, List< Task > > map = tasks
    .stream()
    .collect( Collectors.groupingBy( Task::getStatus ) );
System.out.println( map );
```

输出

```
{CLOSED=[[CLOSED, 8]], OPEN=[[OPEN, 5], [OPEN, 13]]}
```

接下来，让我们计算下每一个task在整个集合中所占的权重。

```
// Calculate the weight of each tasks (as percent of total points) 
final Collection< String > result = tasks
    .stream()                                        // Stream< String >
    .mapToInt( Task::getPoints )                     // IntStream
    .asLongStream()                                  // LongStream
    .mapToDouble( points -> points / totalPoints )   // DoubleStream
    .boxed()                                         // Stream< Double >
    .mapToLong( weigth -> ( long )( weigth * 100 ) ) // LongStream
    .mapToObj( percentage -> percentage + "%" )      // Stream< String> 
    .collect( Collectors.toList() );                 // List< String > 
        
System.out.println( result );
```

输出：

```
[19%, 50%, 30%]
```

最后，流不仅支持java集合操作，典型的I/O操作例如:按行读取文件就是一个很好的例子。

```
final Path path = new File( filename ).toPath();
try( Stream< String > lines = Files.lines( path, StandardCharsets.UTF_8 ) ) {
    lines.onClose( () -> System.out.println("Done!") ).forEach( System.out::println );
}
```

在流上调用onClose方法，会返回一个与stream相等的额外增加的close Handler.
当close()方法被调用时，会触发handler.

流API与<a href="#lambdas_and_functional">Lambdas</a> and <a href="#_Method_References">方法引用</a>和<a href="#_Interface’s_Default_and">接口默认，静态方法Methods</a> 是java 8对于现代范式在软件开发上的最大支持。了解更多细节请关注<a href="http://docs.oracle.com/javase/tutorial/collections/streams/index.html">官方文档</a>

##4.3 日期时间API(JSR310)

java 8提供了日期和时间管理的<a href="https://jcp.org/en/jsr/detail?id=310">API (JSR 310)</a>。日期和时间操作对于java开发者来说很郁闷。即标准的java.util.Date之后的java.util.Calendar也并没有很好的解决这个问题（使得更糟糕了）。

<a href="http://www.joda.org/joda-time/">Joda-Time</a>就应运而生: 是java时间日期API很好的替代品。Java 8的 <a href="https://jcp.org/en/jsr/detail?id=310">Date-Time API (JSR 310)</a>吸取了 <a href="http://www.joda.org/joda-time/">Joda-Time</a>优点. 新的<strong>java.time</strong> 包含了<a href="http://www.javacodegeeks.com/2014/03/whats-new-in-java-8-date-api.html">所有关于日期，时间，时区，瞬时，时长和时钟操作的类</a>. api的设计着重的考虑了不变性：任何需要的修改，都会返回新的类（从java.util.Calendar吸取了教训）。

让我们通过例子来看看关键类的变更。Clock提供了访问瞬时时间，日期，时区时间等。Clock可以代替System.currentTimeMillis()和TimeZone.getDefault().

```
// Get the system clock as UTC offset 
final Clock clock = Clock.systemUTC();
System.out.println( clock.instant() );
System.out.println( clock.millis() );
```

输出：

```
2014-04-12T15:19:29.282Z
1397315969360
```

其他的新类：LocaleDate 和 LocaleTime.LocaleDate只有不支持时区的日期在ISO-8601日历系统中。LocaleTime也一样。都可以通过Clock类创建。

```
// Get the local date and local time
final LocalDate date = LocalDate.now();
final LocalDate dateFromClock = LocalDate.now( clock );
        
System.out.println( date );
System.out.println( dateFromClock );
        
// Get the local date and local time
final LocalTime time = LocalTime.now();
final LocalTime timeFromClock = LocalTime.now( clock );
        
System.out.println( time );
System.out.println( timeFromClock );
```

输出：

```
2014-04-12
2014-04-12
11:25:54.568
15:25:54.568
```

LocaleDateTime兼有LocaleDate和LocalTime作用，拥有一个日期时间，但是在 ISO-8601日历系统中也米有时区。一个<a href="http://www.javacodegeeks.com/2014/04/java-8-date-time-api-tutorial-localdatetime.html">简单的例子</a>：

```
// Get the local date/time
final LocalDateTime datetime = LocalDateTime.now();
final LocalDateTime datetimeFromClock = LocalDateTime.now( clock );
        
System.out.println( datetime );
System.out.println( datetimeFromClock );

输出:

2014-04-12T11:37:52.309
2014-04-12T15:37:52.309
```

如果需要特定时区的时间，ZonedDateTime很有帮助。他有时区时间在ISO-8601日历系统中。下面两个例子说明了不同：

```
// Get the zoned date/time
final ZonedDateTime zonedDatetime = ZonedDateTime.now();
final ZonedDateTime zonedDatetimeFromClock = ZonedDateTime.now( clock );
final ZonedDateTime zonedDatetimeFromZone = ZonedDateTime.now( ZoneId.of( "America/Los_Angeles" ) );
        
System.out.println( zonedDatetime );
System.out.println( zonedDatetimeFromClock );
System.out.println( zonedDatetimeFromZone );

输出：
2014-04-12T11:47:01.017-04:00[America/New_York]
2014-04-12T15:47:01.017Z
2014-04-12T08:47:01.017-07:00[America/Los_Angeles]
```

最后让我们看下Duration: 纳秒/秒的时间单位数量。是的时期计算很简单。例：

```
// Get duration between two dates
final LocalDateTime from = LocalDateTime.of( 2014, Month.APRIL, 16, 0, 0, 0 );
final LocalDateTime to = LocalDateTime.of( 2015, Month.APRIL, 16, 23, 59, 59 );

final Duration duration = Duration.between( from, to );
System.out.println( "Duration in days: " + duration.toDays() );
System.out.println( "Duration in hours: " + duration.toHours() );

```

所有印象深刻的java 8时间日期API都是非常积极的，因为他的基础是 (<a href="http://www.joda.org/joda-time/">Joda-Time</a>).一部分原因是它听到了java开发者的声音。了解更多细节，请关注：<a href="http://docs.oracle.com/javase/tutorial/datetime/index.html">官方文档</a>

#4.4 Nashorn javaScript引擎

<a href="http://www.javacodegeeks.com/2014/02/java-8-compiling-lambda-expressions-in-the-new-nashorn-js-engine.html">Java 8 引进了新的Nashorn JavaScript引擎</a>：允许开发和运行javaScript应用在JVM上。JavaScipt只是javax.script.ScriptEngine的另一个实现，允许同样的规则：java与javaScript交互。例子：

```
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName( "JavaScript" );
        
System.out.println( engine.getClass().getName() );
System.out.println( "Result:" + engine.eval( "function f() { return 1; }; f() + 1;" ) );

输出：

jdk.nashorn.api.scripting.NashornScriptEngine
Result: 2
```
#4.5 Base64

对于<a href="http://www.javacodegeeks.com/2014/04/base64-in-java-8-its-not-too-late-to-join-in-the-fun.html">Base64 encoding</a>的支持也写进了java 8的标准库。下面是一个非常简单的应用：

```
package com.javacodegeeks.java8.base64;

import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64s {
    public static void main(String[] args) {
        final String text = "Base64 finally in Java 8!";
        
        final String encoded = Base64
            .getEncoder()
            .encodeToString( text.getBytes( StandardCharsets.UTF_8 ) );
        System.out.println( encoded );
        
        final String decoded = new String( 
            Base64.getDecoder().decode( encoded ),
            StandardCharsets.UTF_8 );
        System.out.println( decoded );
    }
}

输出：

QmFzZTY0IGZpbmFsbHkgaW4gSmF2YSA4IQ==
Base64 finally in Java 8!
```
Base64也提供了URL友好的encoder/decoder，MIME友好的encoder/decoder。相关class:Base64.getUrlEncoder() / Base64.getUrlDecoder(), Base64.getMimeEncoder() / Base64.getMimeDecoder()).

##4.6 并行数组

java 8增加了许多并行数组的操作。可以说最主要的一个方法是parallelSort:在多核机器中可以加快排序速度。下面例子说明了用法：

```
package com.javacodegeeks.java8.parallel.arrays;

import java.util.Arrays;
import java.util.concurrent.ThreadLocalRandom;

public class ParallelArrays {
    public static void main( String[] args ) {
        long[] arrayOfLong = new long [ 20000 ];		
		
        Arrays.parallelSetAll( arrayOfLong, 
            index -> ThreadLocalRandom.current().nextInt( 1000000 ) );
        Arrays.stream( arrayOfLong ).limit( 10 ).forEach( 
            i -> System.out.print( i + " " ) );
        System.out.println();
		
        Arrays.parallelSort( arrayOfLong );		
        Arrays.stream( arrayOfLong ).limit( 10 ).forEach( 
            i -> System.out.print( i + " " ) );
        System.out.println();
    }
}

输出：

Unsorted: 591217 891976 443951 424479 766825 351964 242997 642839 119108 552378 
Sorted: 39 220 263 268 325 607 655 678 723 793
```

生成了20000个随机数，然后使用了parallelSort()进行排序。然后输出排序前后是个数字。

##4.7 并发

java.util.concurrent.ConcurrentHashMap类型新增了一些方法来支持基于流的操作和lambda表达式。同样，新方法也在Java.utilconcurrent.ForkJoinPool中出现，来支持普通的池(获取免费的关于java<a href="http://academy.javacodegeeks.com/course/java-concurrency-essentials/">多线程的课程</a>)

新的java.util.concurrent.locks.StampedLock类提供了基于锁的三种模式，用来控制读写权限（被认为是比现在流行的java.util.concurrent.locks.ReadWriteLock好）。

java.utilconcurrent.atomic新增了：

- DoubleAccumulator
- DoubleAdder
- LongAccumulator
- LongAdder

# 新的java工具

java 8新增了一系列命令行工具。

## Nashorn engine:jje

jjs是一个基于Nashorn引擎的命令。接受一系列JavaScript文件作为参数来运行。

例如func.js:

```
function f() { 
     return 1; 
}; 

print( f() + 1 );

输出：
jjs:
jjs func.js
```

更多细节请关注<a href="http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jjs.html">官方文档</a>

##5.2 类依赖分析：jdeps

jdeps是一个真是好用的命令行工具。展示了java类文件的文件依赖，包依赖。它接受。class文件，目录，或者jar文件作为输入。默认，jdeps输出依赖到console中。

可以看下流行的<a href="http://projects.spring.io/spring-framework/">Spring Framework</a>的依赖。为了是例子短小。我们仅仅分析org.springframework.core-3.0.5.RELEASE.jar。


```
jdeps org.springframework.core-3.0.5.RELEASE.jar

下面是输出的一部分（依赖以包分组，如果没有就显示not found）：

org.springframework.core-3.0.5.RELEASE.jar -> C:\Program Files\Java\jdk1.8.0\jre\lib\rt.jar
   org.springframework.core (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.io                                            
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.ref                                      
      -> java.lang.reflect                                  
      -> java.util                                          
      -> java.util.concurrent                               
      -> org.apache.commons.logging                         not found
      -> org.springframework.asm                            not found
      -> org.springframework.asm.commons                    not found
   org.springframework.core.annotation (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.reflect                                  
      -> java.util
```

更多细节，请参考<a href="http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html">官方文档</a>

#6 Jvm的新特性

PermGen space<a href="http://www.javacodegeeks.com/2013/02/java-8-from-permgen-to-metaspace.html">已经被Metaspace</a> (<a href="http://openjdk.java.net/jeps/122">JEP 122</a>)所替代. JVM可选项 -XX:PermSize和 XX:MaxPermSize 被 XX:MetaSpaceSize 和 XX:MaxMetaspaceSize替代。

#7 相关资源

- What’s New in JDK 8: <a href="http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html">http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html</a>
- The Java Tutorials: <a href="http://docs.oracle.com/javase/tutorial/">http://docs.oracle.com/javase/tutorial/</a>
- WildFly 8, JDK 8, NetBeans 8, Java EE 7: <a href="http://blog.arungupta.me/2014/03/wildfly8-jdk8-netbeans8-javaee7-excellent-combo-enterprise-java/">http://blog.arungupta.me/2014/03/wildfly8-jdk8-netbeans8-javaee7-excellent-combo-enterprise-java/</a>
- Java 8 Tutorial: <a href="http://winterbe.com/posts/2014/03/16/java-8-tutorial/">http://winterbe.com/posts/2014/03/16/java-8-tutorial/</a>
- JDK 8 Command-line Static Dependency Checker: <a href="http://marxsoftware.blogspot.ca/2014/03/jdeps.html">http://marxsoftware.blogspot.ca/2014/03/jdeps.html</a>
- The Illuminating Javadoc of JDK 8: <a href="http://marxsoftware.blogspot.ca/2014/03/illuminating-javadoc-of-jdk-8.html">http://marxsoftware.blogspot.ca/2014/03/illuminating-javadoc-of-jdk-8.html</a>
- The Dark Side of Java 8: <a href="http://blog.jooq.org/2014/04/04/java-8-friday-the-dark-side-of-java-8/">http://blog.jooq.org/2014/04/04/java-8-friday-the-dark-side-of-java-8/</a>
- Installing Java™ 8 Support in Eclipse Kepler SR2: <a href="http://www.eclipse.org/downloads/java8/">http://www.eclipse.org/downloads/java8/</a>
- Java 8: <a href="http://www.baeldung.com/java8">http://www.baeldung.com/java8</a>
- Oracle Nashorn. A Next-Generation JavaScript Engine for the JVM: <a href="http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html">http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html</a>

来源：https://www.javacodegeeks.com/2014/05/java-8-features-tutorial.html#streams
