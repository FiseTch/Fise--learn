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

··· 使用这些接口可以快速精炼代码，需要练习和尝试



###### 流







###### 聚合器

-----


>每一个流式计算的末尾总会有一个类似`collect(Collectors.toList())`的方法调用。`collect`是`Stream`的方法，而参数则是聚合器`Collector`。已有的聚合器定义在`Collectors`的静态方法里。那么这个聚合器是怎么实现的呢？

**大部分聚合器都是基于`reduce`操作实现的。**

··· 底层实现

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



··· 代码实现

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

1. 







