##### MVC编程思想

* 核心：人为的将程序划分为3个层次。分别为M、V、C。
  * M（Model 模型层）模拟现实生活中存在的事物。
  * V（View 视图层）  可视化的图形界面 用来给用户展示数据
  * C（Controller 控制层 控制器） 控制整体程序的流程

* 现有程序中使用MVC

​		V： HTML +JSP

​		C:  Servlet

​		M: Service+DAO+Entity

* 核心对象
  * StrutsPrepareAndExecuteFilter 对象，该对象有web服务器创建
  * action对象，由Struts框架创建

* 好处：

​		1、解耦合  降低代码之间的耦合度

​		2、便于维护

​		客户端------>Web容器----(web.xml)------>struts2过滤器-----(struts.xml)---->Action(Controler)----->客户端

##### struts2 开发步骤

* 导入核心jar包 struts-core     xwork-core

* 配置文件 struts.xml

* web.xml中配置核心过滤器

  ~~~java
  <web-app>
   	<filter>
   		<filter-name>struts</filter-name>
   		<filter-			 class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
   	</filter>
   	<filter-mapping>
   		<filter-name>struts</filter-name>
   		<url-pattern>/*</url-pattern>
   	</filter-mapping>
  </web-app>
  ~~~

* 写类 implement Action  覆盖execute方法 或者继承 ActionSupport 
* struts.xml配置访问路径

##### struts2和servlet的区别和联系

* servlet存在的问题
  * 是单例的 多线程。线程不安全 不要创建成员变量
  * 接受数据的问题
    * 代码冗余，反复调用getParameter();
    * 需要手工转换数据类型
  * 跳转问题
    * 跳转方式写死在了源码中
* struts2
  * struts2中Action是多例的，多线程
  * Struts2是基于对servlet的封装，开发效率更高，代码更简洁
  * struts2能够通过默认的拦截器StrutsPrepareAndExecuteFilter ，解析请求路径，自动获取package的namespace+action的name+拼接的请求参数，而且能够自动转换成对应的数据类型。 前提要求是在对应的action类中 定义对应的属性 而且属性名要与对应的请求参数的key保持一致，并提供set，get方法。主要是set方法
  * struts2在转发和重定向的时候，URL是写在struts2.xml中，在后期维护的时候更直观、方便
  * Struts2是一个基于MVC设计模式的Web应用框架，它本质上相当于一个servlet，在MVC设计模式中，Struts2作为控制器(Controller)来建立模型与视图的数据交互
  * Strtus2不是Servlet,是使用了Filter过滤器来作为控制器，使用了拦截器，组成拦截器栈，是对Filter的改善，封装，简化，类之间的耦合性小，代码冗余少，更加灵活
* filter
  * 它使用户可以改变一个request和修改一个response. Filter 不是一个servlet，它不能产生一个response，它能够在一个request到达servlet之前预处理request，也可以在response离开servlet时处理response.换种说法，filter其实是一个“servlet chaining“（servlet 链）
  * 使用Filter作为过滤器，拦截客户端请求，代码有冗余，而且代码之间耦合性太高

##### 引用默认拦截器栈

* 作用：存放多个Action中的共有冗余代码，提高开发效率，提升可维护性。

* 开发步骤：

  * 写类实现Interceptor

  ~~~java
  class 类名  implements Interceptor{
      @Override
      public String intercept(ActionInvocation ai) throws Exception {
          if(检查条件){
              ai.invoke();//放行
              return Action.NONE;
          }else{
              return Action.ERROR;
        }
      }   
}
  ~~~

  * 配置Struts文件
  
    * ​	声明拦截器
  
    ~~~java
  <package name="包名" extends="struts-default">
    	<interceptors>
    		<interceptor class="类名" name="myintercptor"></interceptor>
    		<interceptor-stack name="myStack">
  			//引用默认拦截器
    			<interceptor-ref name="defaultStack"></interceptor-ref>
    			//引用自定义拦截器
    			<interceptor-ref name="myintercptor"></interceptor-ref>
    		</interceptor-stack>
    	</interceptors>
    </package>
    ~~~
    
    * 方法拦截器
    
      ~~~java
      <interceptor class="类名" name="myintercptor">
      	//拦截哪些方法（标签内部填的是方法返回值）
      	<param name="includeMethods">selectByCateId</param>
      	//不拦截哪些方法（标签内部填的是方法返回值）
      	<param name="excludeMethods">selectByCateId</param>
      </interceptor>
      ~~~
    
      * 定义全局跳转
    
        ~~~java
        <global-results>
        	<result name="ERROR"type="跳转方式">跳转目的地</result>
        </global-results>
        ~~~
    
        

##### struts2成员变量接收参数

​		注意：提供和传递数据时key同名的成员变量

​				为成员变量提供get set方法

* 接收零散数据（八种基本类型+String+Date）

* 接收对象类型数据

  ​			Action：提供一个对象类型的成员变量

  ​			页面:为成员变量的属性赋值传递数据key  

  ​			<input name="user.id">

  ​			<input name="user.name">

  ​					.....

* 接收数组或者集合类型（同零散变量的接收）

  ​			提供同名的成员变量，提供setget方法即可

  ​			注意 成员变量的类型 是数组或者集合

##### 优点：

（1）  实现了MVC模式，层次结构清晰，使程序员只需关注业务逻辑的实现。

（2）  丰富的标签库，大大提高了开发的效率。
（3） Struts2提供丰富的拦截器实现。
（4） 通过配置文件，就可以掌握整个系统各个部分之间的关系。
（5） 异常处理机制，只需在配置文件中配置异常的映射，即可对异常做相应的处理。

（6） Struts2的可扩展性高。Struts2的核心jar包中由一个struts-default.xml文件，在该文件中设置了一些默认的bean,resultType类型，默认拦截器栈等，所有这些默认设置，用户都可以利用配置文件更改，可以更改为自己开发的bean，resulttype等。因此用户开发了插件的话只要很简单的配置就可以很容易的和Struts2框架融合，这实现了框架对插件的可插拔的特性。
    （7） 面向切面编程的思想在Strut2中也有了很好的体现。最重要的体现就是拦截器的使用，拦截器就是一个一个的小功能单位，用户可以将这些拦截器合并成一个大的拦截器，这个合成的拦截器就像单独的拦截器一样，只要将它配置到一个、Action中就可以。

##### 缺点：

 	（1） Struts2中Action中取得从jsp中传过来的参数时还是有点麻烦。可以为Struts2的Action中的属性配置上Getter和Setter方法，通过默认拦截器，就可以将请求参数设置到这些属性中。如果用这种方式，当请求参数很多时，Action类就会被这些表单属性弄的很臃肿，让人感觉会很乱。还有Action中的属性不但可以用来获得请求参数还可以输出到Jsp中，这样就会更乱。假设从JSP1中获得了参数money=100000，但是这个Action还要输出到JSP2中，但是输出的格式却不同，money=100,000，这样这个Action中的money中的值就变了。
   （2） 校验还是感觉比较繁琐，感觉太烦乱，也太细化了，如果校验出错的只能给用户提示一些信息。如果有多个字段，每个字段出错时返回到不同的画面，这个功能在Strut2框架下借助框架提供的校验逻辑就不容易实现。
   （3） 安全性有待提高。Struts2曝出2个高危安全漏洞，一个是使用缩写的导航参数前缀时的远程代码执行漏洞，另一个是使用缩写的重定向参数前缀时的开放式重定向漏洞。这些漏洞可使黑客取得网站服务器的“最高权限”，从而使企业服务器变成黑客手中的“肉鸡”

##### Struts2中的跳转

* Action到jsp之间的跳转
  forward：默认形式

​				<result name=""  type="dispatcher"/></result>

​		redirect：

​				<result name=””  type=”redirect”/></result>

* Action到Action之间的跳转

​		forward：

​				<result name=””  type=”chain”/>目标action</result>

​		redirect：

​				<result name=""  type="redirectAction"/>目标action</result>

* 数据的传递：

​		工具类：ServletActionContext  

​		获取request和response对象

​		ServletActionContext.getRequest()

​		ServletActionContext.getResponse()

​		ServletActionContext.getRequest().getSession()

##### DMI(Dynamic Method Invoke) 动态方法调用

* 作用：解决Action的冗余

* 概念：在一个Action中书写多个服务方法,从而减少Action的数量，提高程序可维护性

* 开发方式：
  * 写类 class xxx extends ActionSupport
  * 服务方法（多个） 声明要和execute一致

  ​			public String 随意(){   //服务内容   }

##### 使用Struts2 框架的图片上传步骤

* 使用成员变量获得上传的路径和文件名（以及MIME类型）

* 获得上传的文件
* 通过Struts2 提供的FileUtils类的fileCopy();方法进行拷贝