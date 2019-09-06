##### 内部类基础

* 将一个类定义在另一个类里面或者一个方法里面。这样的类称为内部类，广泛的内部类一般分为以下四种

###### 成员内部类

>成员内部类是最普通的内部类，它的定义为位于另一个类的内部。
>
>成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。 

* 例子

```java
package proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/27 14:40
 */
public class Person {
    
    //外部类变量
    private String name;
    //外部类静态成员变量
    private static int flag = 2018;
    
    //不过要注意的是，当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，
    // 即默认情况下访问的是成员内部类的成员。
    // 如果要访问外部类的同名成员，需要以下面的形式进行访问：
    //外部类.this.成员变量
    //外部类.this.成员方法
    private String gender = "man";
    
    
    public Person(String name) {
        this.name = name;
    }
    
    public void show() {
        //打印内部类变量
        
        System.out.println("在外部类中调用内部类");
        System.out.println("---------------------------------------");
        System.out.println("外部类属性：" + name);
        System.out.println("输出内部类属性: " + getTeacherInstance().gender);
        //在外部类中调用内部类方法
        new Teacher("math").hello();
        System.out.println("----------------------------------------");
    }
    
    //虽然成员内部类可以无条件地访问外部类的成员，而外部类想访问成员内部类的成员却不是这么随心所欲了。
    // 在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问：
    
    public Teacher getTeacherInstance() {
        return new Teacher("chinese");
    }
    
    
    class Teacher {//成员内部类
        
        private String description;
        
        private String gender = "woman";
        
        public Teacher(String description) {
            this.description = description;
        }
        
        public void hello() {
            System.out.println("在内部类中调用外部类");
            System.out.println("----------------------------");
            //外部类的静态变量
            System.out.println("外部类的静态变量: " + flag);
            //外部类的private 变量
            System.out.println("外部类的private 变量: " + name);
            
            //内部类的变量
            System.out.println("内部类的变量： " + description);
            
            //打印出外部类的gender属性
            System.out.println("外部类和内部类中存在相同属性 -- 输出外部类：" + Person.this.gender);
            
            //打印出内部类的gender属性
            System.out.println("外部类和内部类中存在相同属性 -- 输出内部类：" + gender);
            System.out.println("----------------------------");
            
            
        }
        
    }
    
    
}

```

* 输出结果

```java
在外部类中调用内部类
---------------------------------------
外部类属性：tch
输出内部类属性: woman
在内部类中调用外部类
----------------------------
外部类的静态变量: 2018
外部类的private 变量: tch
内部类的变量： math
外部类和内部类中存在相同属性 -- 输出外部类：man
外部类和内部类中存在相同属性 -- 输出内部类：woman
----------------------------
----------------------------------------
在内部类中调用外部类
----------------------------
外部类的静态变量: 2018
外部类的private 变量: tch
内部类的变量： InnerTest
外部类和内部类中存在相同属性 -- 输出外部类：man
外部类和内部类中存在相同属性 -- 输出内部类：woman
----------------------------
在内部类中调用外部类
----------------------------
外部类的静态变量: 2018
外部类的private 变量: tch
内部类的变量： chinese
外部类和内部类中存在相同属性 -- 输出外部类：man
外部类和内部类中存在相同属性 -- 输出内部类：woman
----------------------------
```

* 注意看内部类调用外部类，初始化了3个不同的内部类，每个内部类的description不同。

>内部类可以拥有和类一样的访问权限，比如上面的例子，如果成员内部类用private修饰，则只能在外部类的内部访问，如果用public修饰，则任何地方都能访问



###### 局部内部类

>定义在一个方法或者一个作用域里面的类。它和成员内部类的区别在于局部内部类的访问权限仅限于方法内或者该作用域内

```java
package proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/29 17:22
 */
public class Test1 {
    
    public int age;
    
    public void say(){
        System.out.println("类调用say方法");
        //定义一个内部类
        class innerTest{
            
            private String flag = "局部内部类输出";
            
            private void show(){
                System.out.println(flag);
            }
        }
        
        //调用局部内部类中方法：注意这里代码有先后顺序，要先有局部类定义，然后才能调用
        innerTest inner = new innerTest();
        inner.show();
        
    }
    
}

```

```java
class People{
    public People() {
         
    }
}
 
class Man{
    public Man(){
         
    }
     
    public People getWoman(){
        class Woman extends People{   //局部内部类
            int age =0;
        }
        return new Woman();
    }
}
```





调用方式

```java
 public static void main(String[] args) {
        
        Test1 t = new Test1();
        t.say();
        
    }


//结果为
类调用say方法
局部内部类输出
```

* 注意，局部内部类就像是方法里面的一个局部变量一样，是不能有`public,protect,private,static`修饰符的



###### 匿名内部类

>只能使用一次，没有名称，适用于只需要使用一次的类，创建匿名内部类时须继承一个已有的父类或实现一个接口，因为内部类没有名称，因此不存在构造方法。
>
>匿名内部类应该是我们平时写代码用的最多的，在编写事件监听的代码时使用匿名内部类不但方便，而且使代码更加容易维护

* 例子

本文中举了两个例子，一个是接口，另一个是类

```java
public interface MyInterface {
    
    void say();
}

public class Test1 {
    
    public String name;
    
    public void hello(){
        System.out.println(name + ":父类方法输出");
    }
    
}

```

调用

```java
public static void main(String[] args) {
        
    //相当于新建了一个接口的实现匿名类
        MyInterface i = new MyInterface() {
            @Override
            public void say() {
                System.out.println("匿名内部类输出");
            }
        };
        i.say();
        
        System.out.println("---------------------------------");
    
    //重写了类的某个方法匿名类
        Test1 t = new Test1() {
            public void hello() {
                System.out.println(name + "子类方法");
            }
        };
        t.name = "tch";
        t.hello();
        
    }

//结果

匿名内部类输出
---------------------------------
tch子类方法
```



* 注意，匿名内部类也是不能有访问修饰符和static修饰符的
* 匿名内部类时唯一一中没有构造器的类，正因为其没有构造器，所有使用范围非常有限，大部分匿名内部类用于接口回调
* 匿名内部类编译的时候系统自动起名为**类名\$1.class.**
* 一般来说，匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写

###### 静态内部类

* 静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。

* 静态内部类不需要依赖于外部类的，这点和类的静态成员属性有点类似，并且他不能使用外部类

  的非static成员变量或者方法

```java
package proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/29 17:22
 */
public class Test1 {
    
    private int a;
    
    private static String name = "test1";
    
    
    static  class innerTest1{
        
        public void show(){
            //无法打印出a这个属性
            System.out.println(name);
        }
    
    }
}

//调用方式
        Test1.innerTest1 innerTest1 = new Test1.innerTest1();
    
        innerTest1.show();


//输出结果
----------------------------
        test1
```



##### 深入理解内部类

==为什么成员内部类可以无条件访问外部类的成员？==

* 新建一个类`test.java`

```java
package proxy;

public class test{
    
    private String name;
    
    
    
    class innerTest {
        
        public void show() {
            System.out.println(name);
        }
        
    }
}
```

对代码进行编译`javac test.java`后会出现两个文件 `test$innerTest.class`,`test.class`

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package proxy;

class test$innerTest {
    test$innerTest(test var1) {
        this.this$0 = var1;
    }

    public void show() {
        System.out.println(test.access$000(this.this$0));
    }
}

```

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package proxy;

public class test {
    private String name;

    public test() {
    }

    class innerTest {
        innerTest() {
        }

        public void show() {
            System.out.println(test.this.name);
        }
    }
}

```

**javap -c 反编译查看机器码** 

对class文件进行反编译`javap -c test `和`javap -c test$innerTest`得到如下代码

```java
//Compiled from "test.java"
public class proxy.test {
  public proxy.test();
    Code:
       0: aload_0
       1: invokespecial #2                  // Method java/lang/Object."<init>":()V
       4: return

  static java.lang.String access$000(proxy.test);
    Code:
       0: aload_0
       1: getfield      #1                  // Field name:Ljava/lang/String;
       4: areturn
}

```



```java
//Compiled from "test.java"
class proxy.test$innerTest {
  final proxy.test this$0;

  proxy.test$innerTest(proxy.test);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Lproxy/test;
       5: aload_0
       6: invokespecial #2                  // Method java/lang/Object."<init>":()V
       9: return

  public void show();
    Code:
       0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: aload_0
       4: getfield      #1                  // Field this$0:Lproxy/test;
       7: invokestatic  #4                  // Method proxy/test.access$000:(Lproxy/test;)Ljava/lang/String;
      10: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      13: return
}

```

`javap -v`**可以查看class文件的其他附加信息** 

`javap -v test` 和`javap -v test$innerTest ` 

```java
E:\work\JavaProxy\src\proxy>javap -v test
警告: 二进制文件test包含proxy.test
Classfile /E:/work/JavaProxy/src/proxy/test.class
  Last modified 2018-11-30; size 394 bytes
  MD5 checksum c39804b5385dc637ef7d199698ca80b2
  Compiled from "test.java"
public class proxy.test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Fieldref           #3.#18         // proxy/test.name:Ljava/lang/String;
   #2 = Methodref          #4.#19         // java/lang/Object."<init>":()V
   #3 = Class              #20            // proxy/test
   #4 = Class              #21            // java/lang/Object
   #5 = Class              #22            // proxy/test$innerTest
   #6 = Utf8               innerTest
   #7 = Utf8               InnerClasses
   #8 = Utf8               name
   #9 = Utf8               Ljava/lang/String;
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               access$000
  #15 = Utf8               (Lproxy/test;)Ljava/lang/String;
  #16 = Utf8               SourceFile
  #17 = Utf8               test.java
  #18 = NameAndType        #8:#9          // name:Ljava/lang/String;
  #19 = NameAndType        #10:#11        // "<init>":()V
  #20 = Utf8               proxy/test
  #21 = Utf8               java/lang/Object
  #22 = Utf8               proxy/test$innerTest
{
  public proxy.test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #2                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  static java.lang.String access$000(proxy.test);
    descriptor: (Lproxy/test;)Ljava/lang/String;
    flags: ACC_STATIC, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #1                  // Field name:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 3: 0
}
SourceFile: "test.java"
InnerClasses:
     #6= #5 of #3; //innerTest=class proxy/test$innerTest of class proxy/test

```

编译``test$innerTest``

```java
E:\work\JavaProxy\src\proxy>javap -v test$innerTest
警告: 二进制文件test$innerTest包含proxy.test$innerTest
Classfile /E:/work/JavaProxy/src/proxy/test$innerTest.class
  Last modified 2018-11-30; size 574 bytes
  MD5 checksum e8573ae4ea77496478d960386d9a4d5c
  Compiled from "test.java"
class proxy.test$innerTest
  minor version: 0
  major version: 52
  flags: ACC_SUPER
Constant pool:
   #1 = Fieldref           #6.#18         // proxy/test$innerTest.this$0:Lproxy/test;
   #2 = Methodref          #7.#19         // java/lang/Object."<init>":()V
   #3 = Fieldref           #20.#21        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #22.#23        // proxy/test.access$000:(Lproxy/test;)Ljava/lang/String;
   #5 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #6 = Class              #26            // proxy/test$innerTest
   #7 = Class              #29            // java/lang/Object
   #8 = Utf8               this$0
   #9 = Utf8               Lproxy/test;
  #10 = Utf8               <init>
  #11 = Utf8               (Lproxy/test;)V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               show
  #15 = Utf8               ()V
  #16 = Utf8               SourceFile
  #17 = Utf8               test.java
  #18 = NameAndType        #8:#9          // this$0:Lproxy/test;
  #19 = NameAndType        #10:#15        // "<init>":()V
  #20 = Class              #30            // java/lang/System
  #21 = NameAndType        #31:#32        // out:Ljava/io/PrintStream;
  #22 = Class              #33            // proxy/test
  #23 = NameAndType        #34:#35        // access$000:(Lproxy/test;)Ljava/lang/String;
  #24 = Class              #36            // java/io/PrintStream
  #25 = NameAndType        #37:#38        // println:(Ljava/lang/String;)V
  #26 = Utf8               proxy/test$innerTest
  #27 = Utf8               innerTest
  #28 = Utf8               InnerClasses
  #29 = Utf8               java/lang/Object
  #30 = Utf8               java/lang/System
  #31 = Utf8               out
  #32 = Utf8               Ljava/io/PrintStream;
  #33 = Utf8               proxy/test
  #34 = Utf8               access$000
  #35 = Utf8               (Lproxy/test;)Ljava/lang/String;
  #36 = Utf8               java/io/PrintStream
  #37 = Utf8               println
  #38 = Utf8               (Ljava/lang/String;)V
{
  final proxy.test this$0;
    descriptor: Lproxy/test;
    flags: ACC_FINAL, ACC_SYNTHETIC

  proxy.test$innerTest(proxy.test);
    descriptor: (Lproxy/test;)V
    flags:
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #1                  // Field this$0:Lproxy/test;
         5: aload_0
         6: invokespecial #2                  // Method java/lang/Object."<init>":()V
         9: return
      LineNumberTable:
        line 9: 0

  public void show();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_0
         4: getfield      #1                  // Field this$0:Lproxy/test;
         7: invokestatic  #4                  // Method proxy/test.access$000:(Lproxy/test;)Ljava/lang/String;
        10: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: return
      LineNumberTable:
        line 12: 0
        line 13: 13
}
SourceFile: "test.java"
InnerClasses:
     #27= #6 of #22; //innerTest=class proxy/test$innerTest of class proxy/test

```



>
>在test\$innerTest的反编译文件中51行有 ==final proxy.test this$0==,这行式是一个指向外部类对象的指针，也就是说编译器默认为成员内部类添加了一个指向外部类对象的引用，那么引用是如何赋初值的呢？在55行中有==proxy.test$innerTest(proxy.test)==，即调用内部类的构造器，虽然内部构造器默认是无参构造器，但编译时默认添加了一个以==外部类对象的引用==为类型的参数，所以成员内部类中的==proxy.test（参数类型）==   this\$0（参数名称）便指向了外部类对象，因此可以在成员内部类中随意访问外部类的成员，从这里也间接说明了成员内部类依赖于外部类，如果没有创建外部类对象，则无法对this\$0引用进行赋值，也就无法创建成员内部类对象



##### 内部类的使用场景和好处

==为什么在java中需要内部类?==

>1. 每个内部类都能独立的继承一个接口的实现，所以无论外部类是否已经继承了某个(接口的)实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整，
>2. 方便将存在一定逻辑关系的类组织在一起，又可以对外界隐藏。
>3. 方便编写事件驱动程序
>4. 方便编写线程代码
>
>**内部类的存在使得java的多继承机制变得更加完善**



##### 常见的与内部类相关面试题

1. 请补全代码

```java
public class Test{
    public static void main(String[] args){
           // 初始化Bean1
           (1)
           bean1.I++;
           // 初始化Bean2
           (2)
           bean2.J++;
           //初始化Bean3
           (3)
           bean3.k++;
    }
    class Bean1{
           public int I = 0;
    }
 
    static class Bean2{
           public int J = 0;
    }
}
 
class Bean{
    class Bean3{
           public int k = 0;
    }
}
```



答案：

```java
Test test = new Test();    
Test.Bean1 bean1 = test.new Bean1();   


Test.Bean2 b2 = new Test.Bean2(); 


Bean bean = new Bean();     
Bean.Bean3 bean3 =  bean.new Bean3();

```



2. 程序输出结果为？

```java
public class Test {
    public static void main(String[] args)  {
        Outter outter = new Outter();
        outter.new Inner().print();
    }
}
 
 
class Outter
{
    private int a = 1;
    class Inner {
        private int a = 2;
        public void print() {
            int a = 3;
            System.out.println("局部变量：" + a);
            System.out.println("内部类变量：" + this.a);
            System.out.println("外部类变量：" + Outter.this.a);
        }
    }
}
```

答案

```java
3
2
1
```

