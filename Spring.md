##### IOC  （Inversion of Controll）

* 控制反转
  * 创建对象的权利的反转
  * 把成员变量赋值的控制权，从代码中转移（反转）到了配置文件中
  * 好处：**解耦合**
  * 原理：工厂 + 反射
* DI （Dependency Injection）依赖注入
  * 把依赖的对象作为成员变量由Spring的配置文件进行赋值
  * 好处：**解耦合**

##### AOP  面向切面编程

* 附加功能

  * 特点：不属于业务，可有可无
  * 作用：
    * 控制事务
    * 测试性能
    * 打印日志（记录用户重要操作的流水账）

* 代理模式

  * 概念：通过代理类为原始类增加额外功能

  * 代理类的实质

    * 是原始类实现同一个接口
    * 依赖与原始类

  * 好处：避免原始类因为额外功能而频繁修改，从而是代码更加利于维护

  * 名字解释：

    * 原始类，目标类（target）：哪些只负责核心功能，没有加入额外功能的类，是纯净的service
    * 原始功能：原始方法：原始类中的方法，没有加入额外功能的方法
    * 额外功能：可有可无，附加的功能

  * 静态代理设计模式存在的问题

    * 代理类的数量过多，不利于项目的管理
    * 额外功能的代码冗余
    * 替换代理的额外功能太麻烦

  * 动态代理设计模式

    * 概念：通过代理类为原始类增加额外功能

    * 好处：利于原始类维护

    * 开发步骤

      ```java
      1、创建原始对象（service没有额外功能）
      2、创建额外功能 Advice（接口）
      	该接口有四个常用的实现接口
      	MethodBeforeAdvice
      	如果额外功能需要运行在原始方法之前
      	AfterReturnAdvice
      	如果额外功能需要运行在原始方法之后
      	MethodInterceptor
      	如果额外功能需要运行在原始方法之前，后
      	ThrowsAdvice
      	如果额外功能需要运行在原始方法抛异常时
      3、通过配置定义切入点 pointcut，切入点决定了额外功能加入的位置，加给		哪个方法   execution(* *(..))所有方法都加额外功能
          <aop:config>
              <aop:pointcut id="pc" expression="execution(* *(..))"/>
          </aop:config>
          
      4、组装，定义切面=切入点+通知
          <aop:config>
              <aop:pointcut id="pc" expression="execution(* *(..))"/>
              <aop:advisor pointcut-ref="pc" advice-ref="before" />
          </aop:config>
      最后：spring会根据上述的四步开发过程，在程序运行时动态创建代理类
      注意：可以通过原始类的id值 获得代理对象
      ```
  ```java
      
      ```xml
      切入点表达式  execution()
      	a) 方法切入点：
              额外功能嫁给哪些方法
              execution()
              切入点表达式  * *(..)  所有类的所有方法加入额外功能，返回值为任何类型
              方法的签名包括四个部分：修饰符，返回值类型，方法名，参数列表
              第一个 * 意思是修饰符和返回值类型 任意
              第二个 * 意思是方法名 任意
              (..) 意思是参数列表任意
              例如
              	i) 为login()方法加入额外功能  * login(..)
              	ii) login只有一个字符串参数的方法加入额外功能  * login(String)
                     如果参数类型是java.lang包下的类可以省略包名，非java.lang包的类必须写全限定名
              	   如果有多个参数，用逗号隔开
              	   如果是 * login(String,..)这种形式意思是第一个参数是String，其他不限定
          b) 类切入点    给某一个类的所有方法添加额外功能
              execution()  切入点表达式
             	例如  public boolean dynamicproxy.UserServiceImpl.save(String,String)
              	   修饰符返回值    包.类名.方法（参数）
        		为UserServiceImpl类添加额外功能  execution(* *.UserServiceImpl.*(..))
          c) 包切入点   execution(* com.wz.service.*.*(..))  
              这种写法只能切到本包下的类，不包括子包下的类
              如果要想让子包里类也可以切到，应该这么写  execution(* com.wz.service..*.*(..))
      ===============================================================================
      切入点函数
      	execution()
      	args()
      		作用：主要用于匹配函数中的参数作为切入点
      		需求：选择只有2个字符串参数的函数作为额外功能加入的地方
      			execution方法 ： execution(* *(String,String))
      			args方法		： args(String,String)
      	within()
      		作用：匹配包或者类作为切入点
      		需求：选择UserServiceImpl类作为切入点添加额外功能
      				execution方法 ： execution(* *.UserServiceImpl.*(..))
      				within方法 ： 	  within(*.UserServiceImpl)
      			选择某个包下的全部类，包括子包
      				within(service..*)
      	@annotation
      		作用：选择方法上面有符合要求的注解的那些方法作为切入点
      		只查找方法上面有没有 @annotation 函数需要的注解，如果有则加入额外功能，如果没有则不加入
          切入点函数的逻辑运算 and  or  not
          execution(* login(..)) or execution(* registe(..))
          login或register方法加入额外功能
         注意：同一种切入点函数，不能使用and运算符
          
          
  ```

* ##### Spring 配置文件
  
  * 位置随意
  * 名字随意
    * 约定俗成   ： applicationContext.xml
  
* ##### 核心API

  * ApplicationContext  工厂类（负责生产对象）
  * 特点
    * 1、接口
      * 实现类
        * ClassPathXmlApplicationContext    非web环境下，可以加载类路径下的xml文件
        * WebXmlApplicationContext   Web环境
        * FileSystemXmlApplicationContext    可以加载磁盘任意位置的xml文件，需要全限定名
        * AnnotationConfigApplicationContext   用于读取注解创建容器
    * 2、重量级资源
      * 其他重量级资源  SqlSessionFactory
      * 占用内存多，功能强，一个应用只应该创建一个，线程安全

##### Spring创建对象的实现原理

* 读取配置文件，获得class属性指定的全限定名
* 通过反射创建对应类型的对象，反射调用该类的无参构造方法，若该类没有无参构造，则会报NoSuchMethodException运行时异常

##### Spring注入的实现原理

* Spring创建对象后通过set/get方法对成员变量赋值

##### 为什么 Spring 工厂控制对象的创建次数？

* 目的：节省内存资源
* 不能够被用户公用
  * Connection	SqlSession	ShopCart	Action(成员变量不可以共享)
* 能被用户公用的：
  * Service	DAO	SqlSessionFactory	所有的工厂

##### 核心容器的两个接口引发出的问题

* ApplicationContext
  * 它在构建核心容器时，创建对象采取的策略是立即加载的方式，也就是说只要一读取完配置文件马上就创建配置文件中配置的对象   **适合单例对象使用**   开发中多采用这个接口
* BeanFactory
  * 它在构建核心容器时，创建对象采取的策略是采取延迟加载的方式，也就是说，什么时候根据 id 获取对象了，什么时候才真正去创建对象   **适合多例对象**

##### Spring工厂的高级特性

* Spring工厂创建对象的生命周期（ApplicationContext）

  * 对象创建的时机

    * 现用现创建（懒汉式）
    * **工厂启动时把工厂中的对象都创建出来（饿汉式）**。***具体是工厂启动时把单例bean全部创建，多例bean每次使用时创建***
    * **为什么Spring采用饿汉式**
      * 懒汉式思想在这里的体现不合理，我既然在配置文件配了，我肯定是要用对象的
      * 主要目的：**提高程序的运行效率**，节省类加载和创建对象的时间，以空间换时间

  * 对象销毁的时机

    * 调用工厂的 close() 方法之后，Spring会销毁所有创建好的对象
    * ApplicationContext接口中没有定义close()方法，它定义在了实现类中

  * bean对象的作用范围

    * bean 标签中的 scope 属性指定作用范围
    * 取值
    * singleton   单例 （默认）
      * prototype  多例
  * request     作用于web应用的请求范围
      * session    作用于web应用的会话范围
      * global-session   作用于集群环境的会话范围（全局会话范围），非集群环境下相当于session
  
* 生命周期方法
  
    * 1、初始化方法
      * 用户可以任意定义一个方法，为初始化方法，那么这个初始化方法会在对象创建之后，被Spring调用执行
    * 2、销毁方法
      * 用户可以任意定义一个方法，为销毁方法，Spring 会在这个对象销毁之前，调用这个方法
    
    ```java
    <bean id="" class="" init-method="" destroy-method=""></bean>
    ```
  
  * PropertiesPlaceHolder  配置文件参数化
  
    * 目的：把Spring 配置文件中，需要经常修改的字符串，从Spring的配置文件中，转移到一个小的配置文件中，即properties 文件中
    * 好处：利用维护
    * 1 、提供小的配置文件 .properties
    * 位置：随意    名字  随意
    
    ```java
    //jdbc.properties
    driverClassName = com.mysql.jdbc.Driver
    url = jdbc:mysql://localhost:3306/ems
    name = root
  password = root
    ```
    
    * 2 、 
  
  ```java
  //ApplicationContext.xml
  <context:property-placeholder location="jdbc.properties" />
  <bean id="conn" class="placeholder.ConnectionFactoryBean">
  	<property name="driverClassName" value="${driverClassName}">
      <property name="url" value="${url}">
      <property name="username" value="${name}">
      <property name="password" value="${password}">
  </bean>
  //在spring 3.x 中，小配置文件中的key 不要定义为 username
  ```
  
  * 自定义类型转换器 
  * BeanPostProcessor 后置处理Bean

##### AOP编程的实现原理

* 动态代理类  怎么创建出来的？
  * 原始JDK动态代理技术完成（基于接口）
  * cglib动态字节码增加技术完成（不基于接口，是继承）
* 为什么原始对象的ID值，获得的是代理对象？