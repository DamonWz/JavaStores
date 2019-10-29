#### Servlet

* 运行于服务器端的代码，是javaweb开发的基础，javaEE的编程规范之一，是控制层(Control)底层代码
* 通过servlet程序动态生成页面，实现动态页面效果

##### Servlet开发步骤

* 写类实现Servlet接口，或者继承HttpServlet，覆盖service方法
* web.xml中配置

##### Servlet的生命周期

* 同一个类型的Servlet对象在Servlet容器中以单例的形式存在，不能定义成员变量，否则高并发环境下操作数据不一致问题
* 创建
  * 1 、 **第一次访问使用**
  * 2 、 **Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的<load-on-startup>属性决定的**
  * 创建的详细过程
    * **Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法**
    * 在Servlet容器启动后：客户首次向Servlet发出请求，**Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service方法**
    * Servlet的类文件被更新后，重新创建Servlet
  * ---init---service---destory
* 销毁
  * 服务器关闭时销毁
  * 服务器重启

##### Servlet的执行顺序

* 服务器接收请求，从路径中截取出资源路径
  * /项目名/abc
* web.xml中查找对应<url-pattern>的资源/abc
* <servlet-mapping>标签中获取<servlet-name>值
* 在<servlet>标签中匹配相同的<servlet-name>
* 获取<servlet-class>值，也就是服务类的全限定名，通过反射创建对象
  * Class clazz = Class.forName("xxx");
  * Servlet s = (Servlet)clazz.newInstance();

* 调用s.service方法，执行服务功能

*  servlet的执行流程: 前台请求–过滤器–servlet–逻辑处理–response

