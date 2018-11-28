#### java动态代理

* 满足代理模式的三个必要条件

>1. 两个角色 ：执行者，被代理对象
>2. 注重过程，必须要做，被代理对象没时间做或者不想做，不专业
>3. 执行者必须拿到被代理对象的引用



##### 利用java自带类实现代理动态代理

1. 新建一个接口

```java
package proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/27 14:40
 */
public interface Person {
    
    void showInfo();
}

```

2. 创建需要被代理的实际类

```java
package proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/27 14:42
 */
public class Tong implements Person{
    
    private String name;
    
    private String age;
    
    
    public Tong(String name,String age){
        this.name = name;
        this.age = age;
    }
    
   
    
    @Override
    public void showInfo() {
        System.out.println("Tong  --- start");
        System.out.println(this.getClass());
        System.out.println("My name is " + this.name + ", I'm " + this.age + " year's old");
    }
}

```

3. 创建类实现`InvocationHandler`接口 ,并在`invoke`方法中执行被代理对象的`target`方法，在代理过程中，我们可以在对象方法前加入其他处理，这也是`spring aop` 实现的主要原理

```java
package proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/27 15:01
 */
public class TongInvoke implements InvocationHandler {
    
    
    private Object object;
    
    
    public TongInvoke(Object object) {
        
        this.object = object;
    }
    
    /**
     * proxy:代表动态代理对象
     * method：代表正在执行的方法
     * args：代表调用目标方法时传入的实参
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理执行" + method.getName());
        System.out.println(this.getClass().getSimpleName() + "方法执行");
        System.out.println(proxy.getClass());
        //需要有一个被代理类的对象
        Object invoke = method.invoke(object, args);
        return invoke;
    }
}

```

4. 测试类

```java
package proxy;

import java.lang.reflect.Proxy;

/**
 * @author Tongch
 * @version 1.0
 * @time 2018/11/27 14:47
 */
public class TestProxy {
    public static void main(String[] args) {
        
        Person t = new Tong("tong","18");
        
        //创建一个动态
        TongInvoke tong = new TongInvoke(t);
    
        
        //返回代理对象，代理对象会自动调用invoke方法
        Person p = (Person) Proxy.newProxyInstance(Person.class.getClassLoader(),new Class<?>[]{Person.class}, tong);
    
        
        p.showInfo();
    }
}

```

5. 运行结果

```properties
代理执行showInfo
TongInvoke方法执行
class com.sun.proxy.$Proxy0
Tong  --- start
class proxy.Tong
My name is :tong, I'm 18year's old
```

