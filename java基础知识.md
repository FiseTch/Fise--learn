### java基型信息

#### java 反射

----

* 概念：运行时的类信息

* 需求：在编译时，编译器必须知道所有要通过``RTTI``来处理的类

* `java`包支持：``Class``类，``java.lang.Reflect``类库

  >`Reflect`类库包含`Filed`,`Method`及`Constructor`类（每个类都实现了`Member`接口）
  >
  >这些类型由JVM在运行时创建的，用以表示未知类对应的成员。
  >
  >因此我们可以用`Constructor`创建新的对象，用`get()`和`set()`方法读取和修改与`Field`对象关联的字段，用`invoke()`调用与`Method`对象关联的方法，此外还可以调用`getFields()`,`getMethods()`和`getConstrutors()`等便利的方法，已返回字段，方法以及构造器的对象的数组，这样匿名对象的类信息就能在运行时被完全确定下来，而在编译的时候不需要知道任何事情

* 通常你不需要直接使用反射工具，但是在你需要创建更加动态代码的时候会很有用，反射在``Java``中是支持其他特性的，例如对象序列化和``JavaBean``

####  java代理

----

* 静态代理

1. 创建服务类接口

```java

public interface BuyHouse {
    void buyHosue();
}
```

2. 实现服务接口

```java

public class BuyHouseImpl implements BuyHouse {

    @Override
    public void buyHosue() {
        System.out.println("我要买房");
    }
}
```

3. 创建代理类

```java

public class BuyHouseProxy implements BuyHouse {

    private BuyHouse buyHouse;

    public BuyHouseProxy(final BuyHouse buyHouse) {
        this.buyHouse = buyHouse;
    }

    @Override
    public void buyHosue() {
        System.out.println("买房前准备");
        buyHouse.buyHosue();
        System.out.println("买房后装修");

    }
}
```

4. 编写测试类

```java

public class ProxyTest {
    public static void main(String[] args) {
        BuyHouse buyHouse = new BuyHouseImpl();
        buyHouse.buyHosue();
        BuyHouseProxy buyHouseProxy = new BuyHouseProxy(buyHouse);
        buyHouseProxy.buyHosue();
    }
}
```

* 动态代理（所有实现都需要继承``InvocatinHandler``）

1. 编写动态处理器

```java

public class DynamicProxyHandler implements InvocationHandler {

    private Object object;

    public DynamicProxyHandler(final Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("买房前准备");
        Object result = method.invoke(object, args);
        System.out.println("买房后装修");
        return result;
    }
}
```

2. 编写测试类

```java

public class DynamicProxyTest {
    public static void main(String[] args) {
        BuyHouse buyHouse = new BuyHouseImpl();
        BuyHouse proxyBuyHouse = (BuyHouse) Proxy.newProxyInstance(BuyHouse.class.getClassLoader(), new Class[]{BuyHouse.class}, new DynamicProxyHandler(buyHouse));
        proxyBuyHouse.buyHosue();
    }
}
```



> 注意*Proxy.newProxyInstance()*方法接受三个参数：
>
>1. `ClassLoader loader`:指定当前目标对象使用的类加载器,获取加载器的方法是固定的
>2. `Class<?>[] interfaces`:指定目标对象实现的接口的类型,使用泛型方式确认类型
>3. `InvocationHandler`:指定动态处理器，执行目标对象的方法时,会触发事件处理器的方法 注意``Proxy.newProxyInstance()``方法接受三个参数：
>   1. `ClassLoader loader`:指定当前目标对象使用的类加载器,获取加载器的方法是固定的
>   2. `Class<?>[] interfaces`:指定目标对象实现的接口的类型,使用泛型方式确认类型
>   3. `InvocationHandler`:指定动态处理器，执行目标对象的方法时,会触发事件处理器的方法

* 总结

>相比静态代理，动态代理大大减少了我们的开发任务，同时减少了对业务接口的依赖，降低了耦合度，但是还是有一点点的遗憾之处，那就是它始终无法摆脱仅支持==`interface`代理==的桎梏，回想一下那些动态生成的代理类额度继承关系图，它们已经注定有一个共同的父类叫`Proxy`，`java`的继承机制注定了这些动态代理类们无法实现对`class`的动态代理，原因是多继承在`java`本质上就行不通，人们可以否认对代理的必要性，但是同样有一些理由，相信支持`class`动态代理会更美好，接口和类的划分，本就不是很明显，只是到了`java`中才变得如此的细化。

#### CGLIB代理

* `JDK`实现动态代理需要实现类通过接口定义业务方法，对于没有接口的类，如何实现动态代理呢，这需要``CGLIB``。``CGLIB``采用了非常底层的字节码技术，其原理是通过字节码技术为以一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，但因为采用的是继承，所以不能对`final`修饰的类进行代理，``JDK``动态代理与``CGLIB``动态代理均是实现``Spring AOP``的基础

1. 创建`CGLIB`代理类

```java

public class CglibProxy implements MethodInterceptor {
    private Object target;
    public Object getInstance(final Object target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("买房前准备");
        Object result = methodProxy.invoke(object, args);
        System.out.println("买房后装修");
        return result;
    }
}
```

2. 创建测试类

```java

public class CglibProxyTest {
    public static void main(String[] args){
        BuyHouse buyHouse = new BuyHouseImpl();
        CglibProxy cglibProxy = new CglibProxy();
        BuyHouseImpl buyHouseCglibProxy = (BuyHouseImpl) cglibProxy.getInstance(buyHouse);
        buyHouseCglibProxy.buyHosue();
    }
}
```

* 总结

>`CGLIB`创建的动态代理对象比``JDK``创建的动态代理对象的性能更高，但是`CGLIB`创建对象时所花费的时间却比`JDK`多的多，所以对于单例的对象，因为无需频繁创建对象，用`CGLIB`合适，反之使用``JDK``方式要更为合适一些。同时由于``CGLIB``采用动态创建子类的方法，对于``final``修饰的方法无法进行代理

#### 获取Class的三种方式

----

```java
//第一种方式 -- 通过对象获取
Person p = new Person();
Class clazz = p.getClass();

//第二种方式 -- 通过类名直接获取，简单安全高效
Class clazz = Person.class;

//第三种方式 -- 通过Class.forName()获取 -- 知道类的全路径
Class clazz = Class.forName("com.tch.Person");
```

==一个类在JVM只会有一个Class实例==

#### RTTI

---

*  RTTI (``run-Time Type Identification``)，通过运行时类型信息程序能够使用基类的指针或引用来检查这些指针或引用所指的对象的实际派生类型

#### Class类加载

----

* 所有类都是在对其第一次使用时，==动态==加载到``JVM``中，动态意味着分步加载

  >当程序创建第一个对类的静态成员的引用时，就会加载这个类。
  >
  >因此构造器也是类的静态方法，即使在构造器之前并么有使用`static`关键字，故此使用`new`操作符创建类的新对象也会被当作对类的静态成员的引用

* ==一旦某个类的Class对象被载入内存，它就被用来创建这个类的所有对象==

* 使用``Class.newInstance()``创建的类，必须带有==默认的构造器==

* 类加载和类初始化要区分

  

#### 类使用时需要三步准备工作

---

	>1. ==加载==，类加载器执行，从字节码中创建一个``Class``对象
	>2. ==链接==，在链接阶段件验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将解析这个类创建的对其它类的所有引用
	>3. ==初始化==，如果该类具有超类，则对其初始化，执行静态初始化器和静态初始化块



#### static{}静态代码块

----

* 加载顺序

  > ``static{}``(静态代码块)在程序加载中``static``是先于构造方法加载的，并且只会加载一次
  >
  > 代码块中只能使用``static``修饰的属性
  >
  > 
  >
  > ```flow
  > st=>start: 类内部函数加载顺序
  > op1=>operation: 首先执行父类静态的内容块
  > op2=>operation: 执行子类的静态内容块
  > op3=>operation: 执行父类的非静态代码块
  > op4=>operation: 执行父类的构造函数
  > op5=>operation: 执行子类的非静态代码块
  > op6=>operation: 执行子类的构造函数
  > e=>end: 加载结束
  > st->op1->op2->op3->op4->op5->op6->e
  > ```

* 程序中的``static{}``块大多数是为了加载``properties``文件信息，这个加载只会被加载一次

* 总结

  >总之一句话，静态代码块内容先执行，接着执行父类非静态代码块和构造方法，然后执行非静态代码块和构造函数，而且子类的构造方法，不管构造方法带不带参数，默认的都会先去寻找父类的不带参数的构造方法，如果父类没有不带参数的构造方法，那么子类必须用``supper``关键字来调用父类带参数的构造方法，否则编译不能通过

   

#### 基本类型的包装器类

* 获取基本类型的类``Class``，除了用`.class`外，还可以用`.TYPE`

  ```java
  boolean.class //等价于   Boolean.TYPE,但为了保持一致性，建议使用.class
  ```

#### java泛型

* 定义泛型方法的语法格式

  ```java
      /**
       * 
       * @param clazz 声明一个泛型
       * @param <T>  转入泛型的类型
       * @return
       */
      public <T> T getObject(Class<T> clazz){
          T t = clazz.newInstance();//创建一个泛型对象
          return t;
      }
  ```


#### java static final 区别

* `static`

1. 为什么要用`static`

>有一些频繁使用的东西，如果每次使用都``new``一下，那么这个开销会很高，如果使用``static``,一直放在内存中，那么用的时候就直接用，而不需要new一块空间初始化数据，那么`static`就是为了实现一个系统的缓存作用的，其生命周期直到应用程序退出结束。
>
>----
>
>这说明，`static`修饰的类成员，在程序运行过程中，只需要初始化一次即可，不会进行多次的初始化。

2. 用法

>* 用来修饰成员变量，将其变为类的成员，从而实现所有对象对于该成员的共享
>* 修饰成员方法，将其变为类方法，可以直接使用“类名.方法名”的方式调用，常用于工具类
>* 静态块用法，将多个类成员放在一起初始化，使得程序更加规整，其中理解对象的初始化过程非常关键
>* 静态导包用法，将类的方法直接导入到当前类中，从而直接使用“方法名”即可调用类方法

3. 与非静态的区别

>* `static`成员变量
>
>  1. 静态变量被所有对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被加载。
>  2. 非静态变量时对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本各不影响
>  3. `static`不允许用来修饰局部变量
>
>* `static`代码块
>
>  1. `static`块可以用来优化程序性能，是因为它的特性，只会在类加载的时候执行一次
>  2. 大多数时候会将一些只需要进行一次的初始化操作都放在`static`代码块中执行
>
>* `static`内部类
>
>  1. 非静态内部类中不可以声明静态成员
>  2. 静态内部类不能访问其外部类的非静态成员变量和方法
>  3. 在创建静态内部对象时，不需要其外部类的对象
>
>  



* `final`

  1. 用法

     >* 用来修饰数据，包括成员变量和局部变量，该变量只能被赋值一次且它的值无法被改变。对于成员变量来讲，我们必须在声明时或者构造方法中对它进行赋值
     >* 用来修饰方法参数，表示在变量的生存期中它的值不能被改变
     >* 修饰方法，表示该方法无法被重写
     >* 修饰类，表示类无法被继承
     >
     >

  2. `final`数据

     >* `final`关键字修饰的变量，只能进行一次赋值操作，并且在生存期内不能改变它的值
     >* 在编译器编译期间，对该数据进行替换甚至执行计算，可先声明后赋值
     >* `final`修饰的成员变量，我们有且只有两个地方可给它赋值，一个是声明该成员时赋值，另一个在构造方法中赋值，在这两个地方我们必须给它们赋初始值
     >* `final`在引用基本类型和引用类型时，存在差别，`final`修饰引用变量时，限定了引用变量的==引用== 不可改变，但是引用的对象的值是可以改变的

     

* `static final` 

  >==同时使用`static`和`final`修饰的成员在内存中只占据一段不能改变的存储空间==

  