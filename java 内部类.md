##### 内部类基础

1. 成员内部类
2. ==局部内部类== 

>在类方法里面定义一个内部类，

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





3. 匿名内部类

>只能使用一次，没有名称，适用于只需要使用一次的类，创建匿名内部类时须继承一个已有的父类或实现一个接口，因为内部类没有名称，因此不存在构造方法。

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









##### 深入理解内部类



##### 内部类的使用场景和好处



##### 常见的与内部类相关面试题