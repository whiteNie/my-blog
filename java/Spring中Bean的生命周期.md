### 普通 Java 应用
>在传统的 Java 应用中，bean 的生命周期很简单，使用 Java 关键字 new 进行Bean 的实例化，然后该 Bean 就能够使用了。一旦 bean 不再被使用，则由 GC 自动进行垃圾回收。
### Spring 架构应用
>Spring 管理的 bean 的生命周期比传统的 Java 应用要复杂很多，因为 Spring 对 bean 的管理可扩展性非常强，我们可以在 bean 的生命周期中做很多事情，以下是 bean 的生命周期图：
![avatar](http://qiniuyun.whitenip.site/image/blog/spring/bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

#### Bean 的实例化

>容器通过获取 BeanDefinition 对象中的信息进行实例化。这一步只是简单的实例化，并未进行依赖注入。
1. 对于 BeanFactory 容器，当客户向容器请求一个尚未初始化的 bean 时或初始化 bean 的时候需要注入另一个尚未初始化的依赖时，容器就会调用 createBean 进行实例化。
2. 对于 ApplicationContext 容器，当容器启动结束后，便实例化所有的 bean 。
>实例化的对象被包装在 BeanWrapper 对象中，BeanWrapper 设置了对象属性的接口，避免了使用反射机制设置属性。
#### 设置对象属性（依赖注入）

>实例化后的对象被包装在 BeanWrapper 对象中，并且此时对象任然是一个原生的状态，并没有进行依赖注入。紧接着Spring会根据 BeanDefinition 中的信息进行依赖注入。并且通过 BeanWrapper 提供的设置属性的接口完成依赖注入。
#### 注入 Aware 接口（增强 Bean）

> 此时容器需要检测该对象是否实现了 xxxAware 接口，并将相关的 xxxAware 实例注入给 Bean。此时一个正确的对象已经被构造。
接口可以用于在初始化 bean 时获得 Spring 中的一些对象，如获取 Spring 上下文等。
#### BeanPostProcessor (Bean 的自定义处理)

>俗称增强处理器,BeanPostProcessor 接口提供了两个函数 postProcessBeforeInitialization 和 postProcessAfterInitialization 。见名知义，这两个函数分别在 Bean 初始化之前和初始化之后做操作。刚才我们提到的  BeanPostProcessor 并没有在初始化的时候做操作，Spring 提供了一个接口 InitializingBean 该接口提供了一个函数 afterPropertiesSet，该函数允许 bean 实例仅在设置了所有 bean 属性后才可能执行初始化，并在配置错误的情况下抛出异常。
Spring 给 Bean 的配置提供了 init-method 属性，该属性指定了在这一阶段需要执行的函数名
```
@Configuration
public class LifeCycleConfig {
    @Bean(initMethod = "start", destroyMethod = "destroy")
    public SpringLifeCycle create(){
        SpringLifeCycle springLifeCycle = new SpringLifeCycle() ;
        return springLifeCycle ;
    }
}
```
#### 销毁

>和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑
* 补充：依赖注入的几种方式
定一个Java类
```
public class Role { 
  private Long id;    
  private String roleName;    
  private String note;     
  public Role(String roleName, String note) {        
    this.roleName = roleName;        
    this.note = note;    
  }
 /******** setter and getter *******/
}
```
>一般而言，依赖注入分为三种方式：1：构造器注入 2：setter注入 3：接口注入
* 构造器注入
>构造器注入依赖为类的构造函数，构造函数又分为无参构造和有参构造两种。
```
<bean id="role1" class="com.ssm.chapter9.pojo.Role">    
    <constructor-arg index="0" value="总经理"/>    
    <constructor-arg index="1" value="公司管理者"/>
</bean>
```
* setter注入
>首先把构造方法声明为无参的，然后会用 setter 注入为其设置对应的值。
```
<bean id="role2" class="com.ssm.chapter9.pojo.Role">   
    <property name="roleName" value="高级工程师"/>    
    <property name="note" value="重要人员"/>
</bean
```
* 接口注入
>有些时候资源并非来自于自身系统，而是来自于外界，比如数据库连接资源完全可以在Tomcat下配置，然后通过JNDI的形式去获取它，这样数据库连接资源是属于开发工程外的资源，这个时候我们可以采用接口注入的形式来获取它



参考资料：
作者：Young Wang
链接：https://www.zhihu.com/question/38597960/answer/1063970966
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。