```java
处理流程
    1、发起请求到前端控制器 DispatcherServlet
    2、前端控制器请求 HandlerMapping 查找 Handler（根据XML配置，注解进行查找）
    3、处理器映射器 HandlerMapping 向前端控制器返回 Handler
    4、前端控制器调用处理器适配器去执行Handler
    5、处理器适配器执行Handler
    6、Handler 执行完给适配器返回 ModelAndView
    7、处理器适配器向前端控制器返回 ModelAndView (SpringMVC的一个底层对象，包括Model和View)
    8、前端控制器请求视图解析器根据逻辑视图名进行视图解析成真正的视图(jsp)
    9、视图解析器向前端控制器返回 View
    10、前端控制器进行视图渲染，视图渲染将模型数据(在ModelAndView中)填充到request域
    11、前端控制器向用户响应结果

```

```java
组件
    1、前端控制器 DispatcherServlet   接收请求，响应结果，相当于转发器
    2、处理器映射器 HandlerMapping   根据请求的url 查找Handler
    3、处理器适配器 HandlerAdapter  按照特定规则（HandlerAdapter要求的规则）去执行Handler
    注意：编写Handler 时按照 HandlerAdapter的要求去做，这样适配器才可以执行
    4、视图解析器 ViewResolver  进行视图渲染，根据逻辑视图名解析成真正的视图View
    5、视图View   View是一个接口 ，实现类支持不同的View类型，jsp,pdf等等
      
    
```

```java
//前端适配器
书写<url-pattern>的两种方式
一、  *.do    访问以 .do 结尾可由 DisparcherServlet 进行解析
二、  /       所有访问的url都由前端处理器进行解析，但是对于静态资源的解析需要配置不让前端处理器解析，使用此种方式可以实现RESTful风格的 url
三、/*    这样 配置是不可以的，使用这种配置，最终要转发到一个jsp页面，仍然会由前端处理器解析jsp地址，不能根据jsp页面找到Handler，会报错
```

![1571400648729](C:\Users\85261\AppData\Roaming\Typora\typora-user-images\1571400648729.png)





























