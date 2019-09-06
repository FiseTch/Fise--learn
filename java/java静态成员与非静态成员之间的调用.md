#### `java`静态成员与非静态成员之间的调用

##### 类内部间方法之间的调用

* 非静态方法调用

  1. 非静态调用非静态方法

  ```java
      public static void main(String[] args) throws ClassNotFoundException {
         
          test test = new test();
          test.testFunc();
      }
      
      
      public void getTest(String message){
          System.out.println(message);
      }
      
      
      public  void testFunc(){
          System.out.println("调用testFunc");
          getTest("调用getTest");
      }
  }
  ```

  ```htm
         执行结果为：
         调用testFunc
         调用getTest
  ```

  2. 非静态方法调用静态方法

  ```java
      public static void main(String[] args) throws ClassNotFoundException {
         
          test test = new test();
          test.testFunc();
      }
      
      
      public static void getTest(String message){
          System.out.println(message);
      }
      
      
      public  void testFunc(){
          System.out.println("调用testFunc");
          getTest("调用getTest");
      }
  }
  ```

  ```htm
         执行结果为：
         调用testFunc
         调用getTest
  ```

  >***非静态方法中可以直接根据函数名调用静态和非静态方法，***




* 静态方法调用

  1. 静态方法调用非静态方法

      ```java
      public  void getTest(String message){
          System.out.println(message);
      }
      
      
      public static void testFunc(){
          System.out.println("调用testFunc");
          test.getTest("调用getTest");
      }
  
  --------------------------------------------------------------------
      public static void main(String[] args) throws ClassNotFoundException {
  
          test test = new test();
          test.testFunc();
      }
      
      
      public  void getTest(String message){
          System.out.println(message);
      }
      
      
      public static void testFunc(){
          System.out.println("调用testFunc");
          test t = new test();
          t.getTest("调用getTest");
      }
      ```

  

  ```htm
    编译报错，非静态方法无法调用静态方法
    
    -------------------------------------------------
    调用testFunc
    调用getTest
  ```

  

  2. 静态方法调用静态方法

  ```java
      
      public static void main(String[] args) throws ClassNotFoundException {
         
          test test = new test();
          test.testFunc();
      }
      
      
      public static void getTest(String message){
          System.out.println(message);
      }
      
      
      public static void testFunc(){
          System.out.println("调用testFunc");
          getTest("调用getTest");
      }
  ```

  ```htm
     执行结果
     调用testFunc
     调用getTest
  ```

  > 静态方法无法直接调用非静态方法



##### 在主函数方法或者其他类中调用

* 主函数中调用静态和非静态函数

   ```java
    
    
    public static void main(String[] args) throws ClassNotFoundException {
       
        test test = new test();
        test.getTest("ceshi");//非静态类必须实例化对象才可以调用
        
        //静态类可以实例化调用也可以直接函数名调用
        test.testFunc();
        testFunc();
        
        
    }
    
    
    public  void getTest(String message){
        System.out.println(message);
    }
    
    
    public static void testFunc(){
        System.out.println("调用testFunc");
    }
   ```





* 类外部调用静态函数和非静态函数

```java
package com.szkingdom.controller.controllers;

/**
 * @author Tongch
 * @version 1.0
 * @time 2019/3/26 9:37
 */
public class test1 {
    
    public void myTest1(){
        System.out.println("myTest1");
    }
    
    public static void myStaticTest1(){
        System.out.println("myStaticTest1");
    }
}

---------------------------------------------------------------
package com.szkingdom.controller.controllers;

/**
 * @author Tongch
 * @version 1.0
 * @time 2019/3/20 14:27
 */
public class test {
    
    public static void main(String[] args) {

        test1 test = new test1();
        test.myTest1();//其他类的非静态方法调用必须实例化
        
        
        //其他类的静态方法可以实例化调用也可以类名.函数名调用
        test.myStaticTest1();
        test1.myStaticTest1();
    }
    
    
    public void getTest(String message) {
        System.out.println(message);
    }
    
    
    public static void testFunc() {
        System.out.println("调用testFunc");
    }
}

```



##### 总结

***类的静态方法属于类的本身，在类加载的时候会分配内存，可以通过类名去访问：非静态方法和变量属于类的对象，所以只有在类的对象生成时（创建类的实例时）才会分配内存，然后通过类的对象（实例）去访问。在一个类的静态成员中去访问非静态成员会出错是因为在类的非静态成员不存在的时候类的静态成员就已经存在了，访问一个内存中不存在的东西自然会出错***



