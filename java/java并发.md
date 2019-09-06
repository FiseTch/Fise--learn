### java并发

* 并发之前都是顺序编程-即程序中的所有事务在任意时刻都只能执行一个步骤
* 并发具有可论证的确定性，但是实际上具有不可确定性。 

>==并发解决的问题大体上分为“速度”和“设计可管理性”。==
>
>并发通常是提高运行在单处理器上的程序的性能
>
>事实上，从性能的角度看，如果没有任务会阻塞，那么在单处理器机器上使用并发就没有任何意义。



#### 线程的状态

---

>七个状态：
>
>1. 初始化状态 --  `start()`
>2. 就绪状态 -- `ready-to-run`
>3. 等待状态 -- `wait()`
>4. 终止状态 -- `dead`
>5. 阻塞状态 -- `blocked()`
>6. 运行状态 -- `run()`
>7. 挂起状态 -- `sleep()`

#### 创建线程的多种方式

----

* 继承`Runnable`接口

  ```java
  public class ThreadStatus extends Thread {
      @Override
      public void run() {
          System.out.println("任务启动"); 
      }
  }
  ```

* 实现`implements`接口--作为一个线程任务存在

  ```java
  public class ThreadStatus implements Runnable {
      @Override
      public void run() {
          System.out.println( "任务启动");
      }
  }
  
  ```

* 利用匿名内部类(只执行一次)

  ```java
  public class ThreadStatus {
    public static void main(String[] args)  {
          //创建线程并指定线程任务
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println("任务启动");
              }
          }).start();
      }
  }
  ```

  ```java
  public class ThreadStatus  {
      
      
      public static void main(String[] args)  {
          //创建线程并指定线程任务
   		 new Thread(){
              public void run(){
                  System.out.println("任务启动");
              }
          }.start(); 
      }
  }
  ```

  



#### volatile 轻量级锁，线程内可见



#### 线程安全性问题简单总结

* 出现线程安全性问题的条件

  >1. 在多线程的环境下
  >2. 必须有共享资源
  >3. 对共享资源进行非原子性操作

* 解决线程安全性问题的途径

  >1. synchronized（偏向锁，轻量级锁，重量级锁）
  >2. volatile
  >3. jdk提供的原子类
  >4. 使用Lock(共享锁，排他锁)

* 认识的相关==锁==

#### 常见的提高高并发下访问额度效率的手段

==高并发的瓶颈在哪方面？== 

>* 服务器网络带宽不够
>* web线程连接数不够
>* 数据库链接查询上不去

==解决思路==

>* 增加带宽，``DNS``域名解析分发多台服务器
>* 负载均衡，前置代理服务器`nginx`,`apache`等等
>* 数据库查询优化，读写分离，分表等等



高并发的解决方法有两种

> 一种是使用缓存，另一种是使用生成静态页面，还有就是从最基础的地方优化我们写代码减少不必要的资源浪费（
>
> 1. 不要频繁的new对象，对于在整个应用中只需要存在一个实例的类使用单例模式，对于String的连接操作，使用StringBuffer或者StringBuilder。对于utility类型的类通过静态方法来访问。
>
>   . 避免使用错误的方式，如`Exception`可以控制方法推出，但是``Exception``要保留`stacktrace`消耗性能，除非必要不要使用`instanceof`做条件判断，尽量使用比的条件判断方式，使用`JAVA	`中效率高的类，比如`ArrayList`比`Vector`性能好。
>
> ）







