#### `java 8` 函数式编程

------

##### 引子 --- 行为就是数据

------


* 将行为作为数据传递

>怎样在一行代码里同时计算一个列表的和，最大值，最小值，平均值，元素个数，奇偶分组，指数，排序呢？
>
>答案是思维反转，将**行为作为数据传递**，实例代码如下

```java
package function;

import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.stream.Collectors;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/12/3 14:07
 */
public class function {
    
    public static <T, R> List<R> mutiGetResult(List<Function<List<T>, R>> functions, 
                                               List<T> list) {
        return functions.stream().map(f -> f.apply(list)).collect(Collectors.toList());
    }
    
    
    /**
     * 先简单解释一下语句的含义，stream()操作将集合转换成一个流，
     * filter()执行我们自定义的筛选处理，这里是通过lambda表达式筛选出所有偶数，
     * 最后我们通过collect()对结果进行封装处理，并通过Collectors.toList()指定其封装成为一个List集合返回。
     * @param args
     */
    public static void main(String[] args) {
        
        System.out.println(mutiGetResult(
                Arrays.asList(
                        //计算数组的长度，和，最小值，平均数，最大值
                        list -> list.stream().collect(Collectors.summarizingInt(x -> x)),
                    
                        //将数组内的数按从小到大的顺序排序并以数组的形式输出
                        list -> list.stream().sorted().collect(Collectors.toList()),
                    
                        //将数组内所有小于50的数按从小到大的顺序并以数组的形式输出
                        list -> list.stream()
                    .filter(x -> x < 50).sorted().collect(Collectors.toList()),
                    
                        //将数组内的所有数排序后开方并数组的形式输出
                        list -> list.stream().sorted()
                    .map(Math::sqrt).collect(Collectors.toList()),
                    
                        //将数组内的所有数排序后开方，然后对开方后的数作为2的幂计算后输出
                        //todo: 如何将计算出来的结果map进行排序
                        list -> list.stream().sorted().map(Math::sqrt)
                    .collect(Collectors.toMap(x -> x, y -> Math.pow(2, y))),
                    
                    //将数组内的所有数排序后开方，然后对开方后的数作为2的幂计算后输出
                        //将数组内的数按奇数偶数进行分组
                        list -> list.stream()
                    .collect(Collectors.groupingBy(x -> (x % 2 == 0 ? "even" : "odd")))
                ), Arrays.asList(64, 49, 25, 16, 9, 4, 1, 81, 36)
        ));
        
    }
    
}

```

··· 输出结果

```java
[
 IntSummaryStatistics{count=9, sum=285, min=1, average=31.666667, max=81},
 [1, 4, 9, 16, 25, 36, 49, 64, 81], 
    [1, 4, 9, 16, 25, 36, 49], 
 [1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], 
 {8.0=256.0, 4.0=16.0, 2.0=4.0, 1.0=2.0, 9.0=512.0, 5.0=32.0, 6.0=64.0, 3.0=8.0, 7.0=128.0}, {even=[64, 16, 4, 36], odd=[49, 25, 9, 1, 81]}
]
```



因为调用函数过程中`List<Function<List<T>, R>> functions`总共有6个`Function<List<T>, R>,`,因此会有6个输出结果。而且这6个输出结果包含在一个数组中



##### `java 8`函数框架解读

--------

**函数编程的最直接的表现，莫过于将函数作为数据传递，结合泛型推导能力，使代码表现能力获得飞一般的提升**,`java 8` 是如何支持函数编程的呢？主要有三个核心概念

1. 函数接口(Function)
2. 流(Stream)
3. 聚合器(Collector)



###### 函数接口

>关于函数接口，需要记住的就是两件事

* 函数接口就是**行为的抽象**
* 函数接口就是**数据转换器**

最直接的支持就是`java.util.Function`包，定义了四个最基础的函数接口：

1. `supplier`

>数据提供器，可以提供==T==类型对象
>
>无参的构造器，提供了`get`方法



2. `Funticon`

>数据转换器，可以提供==T==类型对象，返回一个==R==类型的对象
>
>单参数返回值的行为接口，提供了`apply`，`compose`,`andThen`，`identity`方法



3. `Consumer`

>数据消费器，接受一个==T==类型的对象，无返回值，通常用于根据==T==对象做些处理
>
>单参数无返回值的行为接口
>
>提供了`accept`,`andThen`方法



4. `Predicate`

>条件测试器，接受一个==T==类型的对象，返回布尔值，通常用于传递条件参数
>
>单参数布尔值的条件性接口
>
>提供了`test`(条件测试),`and-or-negate`(与或非)方法

----

>小贴士：其中,`compose`,`andThen`,`and`,`or`,`negate`用来组合函数接口而得到更强大的函数接口
>
>其他的函数接口都是通过这四个拓展而来:
>
>1. 在参数个数上拓展 : 比如接受双参数的，有`bi`前缀，比如`BiConsumer`,`BiFunction`;
>2. 在类型上拓展 : 比如接受原子类型参数的，有 `Int ,Double,Long,Function,Consumer,supplier,Predicate`
>3. 特殊常用的变形 ：比如`BinaryOperator`,是同类型的双参数`BiFunction`，二元操作符，`UnaryOperator`是`Function`一元操作符
>
>那么，这些函数接口可以接受那些值呢？
>
>* 类/对象的静态方法引用，实例方法引用。引用符号为双冒号：：
>* 类的构造器引用，比如Class：：new
>* lambda表达式

···使用这些接口可以快速精炼代码，需要练习和尝试



###### 流

----

>流(Stream)是`java8`对函数式编程的重要支撑，大部分函数式工具都围绕`Stream`展开

* `Stream`的接口

`Stream`主要有四类接口

1. 流到流之间的转换:比如`filter`(过滤)，`map(映射转换)`,`mapToInt|Long|Double(高维结构平铺)`,`flatMap`,`flatMaoTo[Int|Long|Double]`,`sorted(排序)`,`distinct(不重复值)`,`peek(执行某种操作，流不变，可用于调试)`,`limit(限制到指定元素数量)`,`skip(跳过若干元素)`
2. 流到终值的转换：比如`toArray(转为数组)`,`reduce(推导结果)`，`collect(聚合结果)`，`min(最小值)`，`max(最大值)`,`count(元素个数)`，`anyMatch(任一匹配)`，`allMatch(所有都匹配)`，`noneMatch(一个都不匹配)`，`findFirst(选择首元素)`，`findAny(任选一元素)`
3. 直接遍历：`forEach(不保序遍历，比如并行流)`，`forEachOrdered(保序遍历)`
4. 构造流：`empty(构造空流)`，`of(单个元素的流及多元素顺序流)`，`iterate(无限长度的有序顺序流)`，`generate(将数据提供器转换成无限非有序的顺序流)`，`concat(流的连接)`，`Builder(用于构造流的Builder对象)`

>除了`Stream`本身自带的生成`Stream`的方法，数组和容器及`StreamSuport`都有转换为流的方法。比如`Arrays.Stream,[List|Set|Collection|].[stream|parallelStream]，StreamSupport,[int|long|double|]Stream`
>
>流的类型主要有：`reference(对象流)，IntStream(int元素流)，LongStream(long元素流)，Double(double元素流)，定义在类StreamShape中`，主要将操作适配于类型系统

* **`collector`实现**

这里我们仅分析串行是怎么实现的，入口在类`java.util.stream.ReferencePipeline`的`collect`方法

```java
container = evaluate(ReduceOps.makeRef(collector));
return collector.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)
    ?(R)container : collector.finisher().apply(container);
```

这里的关键是ReduceOps.makeRef(collector)。点进去:

```java
public static <T, I> TerminalOp<T, I>
    makeRef(Collector<? super T, I, ?> collector) {
        Supplier<I> supplier = Objects.requireNonNull(collector).supplier();
        BiConsumer<I, ? super T> accumulator = collector.accumulator();
        BinaryOperator<I> combiner = collector.combiner();
        class ReducingSink extends Box<I>
                implements AccumulatingSink<T, I, ReducingSink> {
            @Override
            public void begin(long size) {
                state = supplier.get();
            }

            @Override
            public void accept(T t) {
                accumulator.accept(state, t);
            }

            @Override
            public void combine(ReducingSink other) {
                state = combiner.apply(state, other.state);
            }
        }
        return new ReduceOp<T, I, ReducingSink>(StreamShape.REFERENCE) {
            @Override
            public ReducingSink makeSink() {
                return new ReducingSink();
            }

            @Override
            public int getOpFlags() {
                return collector.characteristics().contains(Collector.Characteristics.UNORDERED)
                       ? StreamOpFlag.NOT_ORDERED
                       : 0;
            }
        };
    }


 private static abstract class Box<U> {
        U state;

        Box() {} // Avoid creation of special accessor

        public U get() {
            return state;
        }
    }

```

`box`是一个结果置的持有者; `ReducingSink`用`begin,accpet,combine`三个方法定义了要进行的计算，`ReducingSink`是有状态的流数据消费的计算抽象，阅读`Sink`接口文档可知，`ReduceOps.makeRef(collector)`

返回了一个封装了`Reduce`操作的`ReduceOps`对象，注意到，这里都是声明要执行的计算，而不涉及计算的实际过程，展示了==表达与执行分离==的思想，真正的计算过程启动在`ReferencePipeline.evaluate`方法里

```java
final <R> R evaluate(TerminalOp<E_OUT, R> terminalOp) {
        assert getOutputShape() == terminalOp.inputShape();
        if (linkedOrConsumed)
            throw new IllegalStateException(MSG_STREAM_LINKED);
        linkedOrConsumed = true;

        return isParallel()
               ? terminalOp.evaluateParallel(this, sourceSpliterator(terminalOp.getOpFlags()))
               : terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));
    }
```

使用`IDE`的`go to implmenttations`功能，跟进去，最终在`AbstracePipeLine`中定义了:

```java
@Override
    final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
        Objects.requireNonNull(wrappedSink);

        if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
            wrappedSink.begin(spliterator.getExactSizeIfKnown());
            spliterator.forEachRemaining(wrappedSink);
            wrappedSink.end();
        }
        else {
            copyIntoWithCancel(wrappedSink, spliterator);
        }
    }
```

`Spliterator`用来对流中的元素进行分区和遍历以及施加`Sink`指定操作，可以用于并发计算。`Spliterator`的1具体实现类定义在`Spliterators`的静态类和静态方法中。其中有

```java
    // 数组 Spliterators
     
    static final class ArraySpliterator<T> implements Spliterator<T>
    
    static final class IntArraySpliterator implements Spliterator.OfInt 
    
    static final class LongArraySpliterator implements Spliterator.OfLong 
    
    static final class DoubleArraySpliterator implements Spliterator.OfDouble 
    
    // 迭代 Spliterators
    
    static class IteratorSpliterator<T> implements Spliterator<T> 

    static final class IntIteratorSpliterator implements Spliterator.OfInt 

    static final class LongIteratorSpliterator implements Spliterator.OfLong 

    static final class DoubleIteratorSpliterator implements Spliterator.OfDouble 
    
    //抽象 Spliterators
    
    public static abstract class AbstractSpliterator<T> implements Spliterator<T> {
        
    private static abstract class EmptySpliterator<T, S extends Spliterator<T>, C> 
   
    public static abstract class AbstractIntSpliterator implements Spliterator.OfInt 
    
    public static abstract class AbstractLongSpliterator implements Spliterator.OfLong 

    public static abstract class AbstractDoubleSpliterator implements Spliterator.OfDouble 
    
```

每个具体类都实现了`trySpilt,forEachRemaining,tryAdvance,estimateSize,characteristics,getComparator.trySplit`用于拆分流，提供并发能力，`forEachRemaining,tryAdvance`用于遍历和消费流中的数据。下面展示了`iteratorSpliterator`的`forEachRemaining,tryAdvance`两个方法的实现，可以看到，作用是遍历元素并将操作施加于元素

```java

```

...省略一部分





* 总结

套路基本一样，关键点在于`accept`方法，`filter`只在满足条件时将值转给下一个`pipeline`，而`map`将计算的值传给下一个`pipeline.statelessOp`

`ReferencePipeline`负责将值在`pipeline`之间传递，交给`Sink`去计算



###### 聚合器

-----


>每一个流式计算的末尾总会有一个类似`collect(Collectors.toList())`的方法调用。`collect`是`Stream`的方法，而参数则是聚合器`Collector`。已有的聚合器定义在`Collectors`的静态方法里。那么这个聚合器是怎么实现的呢？

**大部分聚合器都是基于`reduce`操作实现的。**

···底层实现

------

* `reduce`，名曰推导，含有三个要素：初始值，`init`，二元操作符`BinaryOperator`，以及一个用于聚合结果的数据源S。

具体算法如下

>STEP1：初始化结果R = init
>
>STEP2：每次从S中抽出一个值v，通过二元操作符施加到R和V，产生一个新值赋给R  = `BinaryOperator(R,v)`， 重复STEP2,直到S中没有值可取为止
>
>STEP3:  返回结果

比如一个列表求和,Sum([1,2,3]),那么定义一个初始值0以及一个二元加法操作BO = a+b ,通过三步完成`Reduce`操作：

step1 ： R = 0 ；

step2 ： v = 1， R = 0+v =1;

step2 :    v =2 ,    R = 1+ v = 3;

step2 :     v =3 ,    R = 3 + v = 6；

//此处step2因为是重复取S中的值，因此step2应该是个for循环，根据数组的长度来确定执行多少次

step3 ： 返回结果源



···代码实现

----

>一个聚合器的实现，通常需要提供四要素
>
>1. 一个结果容器的初始值提供器 `supplier`
>2. 一个用于将每次二元操作的中间结果与结果容器的值进行操作并重新设置结果容器的累计器`accumulator`
>3. 一个用于对`Stream` 元素和中间结果进行操作的二元操作符`combiner`
>4. 一个用于对结果容器进行最终聚合的转换器`finisher`（可选）

`Collectors`文件中的`CollectorImpl`方法的实现充分体现了上述思想

```java
static class CollectorImpl<T, A, R> implements Collector<T, A, R> {
    	//初始值提供器supplier
        private final Supplier<A> supplier;
    	//结果容器的累计器accumulator
        private final BiConsumer<A, T> accumulator;
        //二元操作符combiner
    	private final BinaryOperator<A> combiner;
        //最终聚合的转换器finisher
    	private final Function<A, R> finisher;
        private final Set<Characteristics> characteristics;

        CollectorImpl(Supplier<A> supplier,
                      BiConsumer<A, T> accumulator,
                      BinaryOperator<A> combiner,
                      Function<A,R> finisher,
                      Set<Characteristics> characteristics) {
            this.supplier = supplier;
            this.accumulator = accumulator;
            this.combiner = combiner;
            this.finisher = finisher;
            this.characteristics = characteristics;
        }
      ...
}
```

* 列表类聚合器

列表类聚合器实现，基本是基于`reduce`操作完成的。看如下代码:

```java
public static <T>
    Collector<T, ?, List<T>> toList() {
        return new CollectorImpl<>((Supplier<List<T>>) ArrayList::new, List::add,
                                   (left, right) -> { left.addAll(right); return left; },
                                   CH_ID);
    }
```



首先使用`ArrayList :: new `创造一个空列表，然后`List::add`将`Stream`累积操作的中间结果加入到这个列表；第三个函数则将这两个列表元素进行合并成一个结果列表中，就是这么简单。集合聚合器`toSet()`,字符串连接器`joining()`,以及列表求和(summingXXX)，最大(maxBy),最小值(minBy)等都是这个套路



* 映射类聚合器

映射类聚合器基于`map`合并来完成，下面代码：

```java
 private static <K, V, M extends Map<K,V>>
    BinaryOperator<M> mapMerger(BinaryOperator<V> mergeFunction) {
        return (m1, m2) -> {
            for (Map.Entry<K,V> e : m2.entrySet())
                m1.merge(e.getKey(), e.getValue(), mergeFunction);
            return m1;
        };
    }
```



根据指定的值合并函数`mergeFunction`,返回`map`合并器，用来合并两个`map`里相同`key`的值，`mergeFunction`用来对两个`map`中相同`key`的值进行运算得到新的`value`值，如果`value`值为`null`，会移除相应的`key`。否则使用`value`值作为对应`key`的值。这个方法是私有的，主要为支撑`toMap,groupingBy`而生。

`toMap`的实现很简短，实际上就是将指定`Stream`的每个元素分别使用给定函数`keyMapper`，`valueMapper`进行映射得到`newKey,newValue`,从而形成新的`MapnewKey,newValue`,再使用`mapMerger(mergeFunction)`生成的`map`合并器将其合并到`mapSupplier(2)`。如果只传`keyMapper,valueMapper`，那么就只得到结果(1)·

```java
 public static <T, K, U, M extends Map<K, U>>
    Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction,
                                Supplier<M> mapSupplier) {
        BiConsumer<M, T> accumulator
                = (map, element) -> map.merge(keyMapper.apply(element),
                                              valueMapper.apply(element), mergeFunction);
        return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
    }
```



`toMap`的一个实例见如下代码

```java
public static void main(String[] args) {
    
   		//新建一个List
        List<Integer> list = Arrays.asList(1,2,3,4,5);
    	//创建初始化值容器
    	//将数组[1,2,3,4,5] 转成 map{1=1,2=4,3=9,4=16,5=25}
        Supplier<Map<Integer,Integer>> mapSupplier = () -> list.stream().collect(Collectors.toMap(x->x, y-> y * y));
    
       //toMap函数的四个入参分别为keyMapper，valueMapper，mergeFunction，mapSupplier
    	//含义为：根据指定的合并规则mergeFunction，即第三个参数，用来合并两个map中相同的key。此处相同的key为[1,2,3,4,5],根据规则算出新的value后，返回新的Map，此处即将[1=1,2=2,3=3,4=4,5=5]和				map{1=1,2=4,3=9,4=16,5=25}进行，
    //第一二个参数是对list本身进行的修改
        Map<Integer, Integer> mapValueAdd = list.stream().collect(Collectors.toMap(x->x,
                                                                                   y->y, (v1,v2) -> v1+v2, mapSupplier));
        System.out.println(mapValueAdd);
    }
```



··· 返回结果

```java
{1=2, 2=6, 3=12, 4=20, 5=30}
```



* 自定义聚合器

>实现一个含N个数的斐波那契序列List，由于`Reduce`每次都从流中取一个数，因此需要生产一个含N个数的`Stream`；可使用Arrays.asList(1,2,3).Stream(),亦可使用`IntStream.range(1,11)`,不过两者的`collector`方法是不一样的，这里我们取前者

**步骤**（构造四要素）

1. 可变的结果容器提供器`Supperlier <list> = () ->[0,1] `,这里不能使用Arrays.asList,因为该方法生成的列表是不可变的。
2. 累计器`BiConsumer<list,Integer> accumlator():`这里流的元素未用，仅仅用来使计算进行和终止，新的元素从结果容器中取最后两个相加后产生新的结果放放到结果容器中
3. 组合器`BinaryOperator<List> combiner()` 照葫芦画瓢，目前没看出这步是做什么用；直接`return null`，也是ok的，
4. 最终转换器`Function<List,List> finisher()` 在最终转换器中，移除初始设置的两个值0，1

```java
package com.szkingdom.controller.controllers;

import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/12/10 10:28
 */
public class test implements Collector<Integer, List<Integer>, List<Integer>> {
    
    //第二步执行，list中设置两个斐波那契数列的初始值
    public Supplier<List<Integer>> supplier() {
        return () -> {
            List<Integer> result = new ArrayList<>();
            result.add(0);
            result.add(1);
            return result;
        };
    }
    
    //第三步，根据传递过来的数组的长度，循环递增计算斐波那契
    @Override
    public BiConsumer<List<Integer>, Integer> accumulator() {
        return (res, num) -> {
            Integer next = res.get(res.size() - 1) + res.get(res.size() - 2);
            res.add(next);
        };
    }
    //第四步，目前作用还不知道，猜想是对数据进行格式的问题
    @Override
    public BinaryOperator<List<Integer>> combiner() {
        return null;
        //return (left, right) -> { left.addAll(right); return left; };
    }
    
    //将数组拿掉一开始赋予的初值(按位置删除)
    @Override
    public Function<List<Integer>, List<Integer>> finisher() {
        return res -> {
            res.remove(0);
            res.remove(1);
            return res;
        };
    }
    
    //第一步执行
    @Override
    public Set<Collector.Characteristics> characteristics() {
        return Collections.emptySet();
    }
    
    public static void main(String[] args) {
        List<Integer> fibo = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).stream().collect(new test());
        System.out.println(fibo);
    }
}


[1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```









