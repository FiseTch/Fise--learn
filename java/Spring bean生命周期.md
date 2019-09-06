### spring bean生命周期

1. 实例化
2. 传入参数（默认无参构造）--设置属性值
3. 实现==beanNameAware==接口的``setBeanName``方法,如果实现了接口，则可以通过``setBeanName``获取id号
4. 实现==BeanFactoryAware==接口返回``Bean``工厂对象
5. 实现==ApplicationAware==接口返回上下文信息
6. ``bean``与后置处理器关联，会调用``BeforeProcessor``方法,Aop编程
7. 如果实现了==initializationBean==，会调用``AfterPropertiesSet``
8. 可以在``bean``中定义自己的初始化方法
9. ``bean``与后置处理器关联,会调用``AfterProcessor``方法
10. ==destroy==，关闭数据连接，`socket`，文件流
11. 容器关闭
12. 可以通过实现==DisposableBean==接口来调用方法``Destroy``
13. 可以定义自己的销毁方法

### 小结

实际开发中，没有这么多具体的过程

1-->2-->6-->10-->9-->11

#### 通过beanFactory来获取bean对象，bean的声明周期是不是和通过上下文获取一样的？？？

是不一样的，通过工厂获取周期是有变化的。会简单些，具体如

1-->2-->3-->4-->7->8-->11-->12-->13





