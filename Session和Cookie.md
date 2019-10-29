##### Cookie

* 客户端记录用户状态的技术（不安全，用户可以伪造Cookie）

* Cookie操作：

  * 创建

  ~~~java
  Cookie c = new Cookie(String name , String value)
  //设置Cookie存活时长
  c.setMaxAge(int n); 
  n  为正整数  Cookie存活的时长  单位  秒  
  n 为负整数  关闭浏览器销毁Cookie  -1【默认】
  n 为0     直接销毁  不创建
  //将Cookie对象返回到客户端
  response.addCookie(c);
  ~~~

  * 获取

  ~~~java
  //获取本网站发放的所有Cookie对象
  Cookie[] cs =request.getCookies();
  Cookie.getName();//获取当前Cookie对象的name值
  Cookie.getValue();//获取当前COokie对象的value值
  ~~~

##### Session

* 服务器端记录用户状态的技术
  * 每个用户（即浏览器）第一次访问服务器时，可以创建对应的session对象
  * 每个用户（即浏览器）后续访问都能找到同一个session对象（使用Cookie完成这个功能）
* 会话：同一个用户对应用程序的一次访问，一次会话

##### Session和Cookie的区别与联系

| 对象    | 信息量           | 保存时间                              | 应用范围 | 保存位置 |
| ------- | ---------------- | ------------------------------------- | -------- | -------- |
| session | 小量，简单的数据 | 用户活动时间+一段延迟时间（约20分钟） | 单个用户 | 服务器端 |
| cookie  | 小量，简单的数据 | 可以根据需要设定                      | 单个用户 | 客户端   |

##### Session的生命周期

* 创建

  * request.getSession()；
  * 通过session.isNew()；方法判断session是不是新创建的

* 销毁

  * 手动销毁
    * session.invalidate();
  * 自动销毁
    * 回话访问最后一次之后过30分钟，自行销毁
    * 存活时长可以在容器中配置，比如tomcat

  ~~~java
  //tomcat--conf--web.xml
  <session-config>
  	<session-timeout>30</session-timeout>
  </session-config>
  ~~~

  不要直接修改容器的配置，可以在项目web.xml中覆盖容器的配置

##### session的核心原理

* 在第一次使用session对象时，服务器会自动创建一个Cookie对象(“JSESSIONID”,”当前session的id值”)，Cookie对象会随着响应返回客户端存储在操作系统中，后续每次访问都会携带当前Cookie。服务器拿到Cookie对象之后就能获取Session对象的id值，从而找到session对象。

##### URL重写

* 原理：将该用户session的id信息重写到URL地址中，服务器能够解析重写后的URL获取session的ID。
* 作用：防止用户禁用Cookie
* **response.encodeRedirectURL(java.lang.String url)** 
  * 用于对sendRedirect方法后的url地址进行重写
* **response.encodeURL(java.lang.String url)**
  * 用于对表单action和超链接的url地址进行重写
* 该方法会自动判断客户端是否支持Cookie。如果客户端支持Cookie，会将URL原封不动的输出来。如果客户端不支持Cookie，则会将用户session的id重写到URL中。
* 注意：TOMCAT判断客户端浏览器是否支持Cookie的依据是请求中是否含有Cookie。尽管客户端可能会支持Cookie，但是由于第一次请求时不会携带任何Cookie（因为并无任何Cookie可以携带），URL地址重写后的地址中仍然会带有jsessionid。当第二次访问时服务器已经在浏览器中写入Cookie了，因此URL地址重写后的地址中就不会带有jsessionid了

##### Cookie禁用后session的使用问题

* 虽然session保存在服务器，对客户端是透明的，它的正常运行仍然需要客户端浏览器的支持。这是因为session需要使用Cookie作为识别标志。HTTP协议是无状态的，session不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为JSESSIONID的Cookie，它的值为该session的id（也就是HttpSession.getId()的返回值）。session依据该Cookie来识别是否为同一用户。该Cookie为服务器自动生成的，它的maxAge属性一般为-1，表示仅当前浏览器内有效，并且各浏览器窗口之间不共享，关闭浏览器就会失效。**因此同一台电脑的两个浏览器窗口访问服务器时，会生成两个不同的session。但是由浏览器窗口内的链接、脚本等打开的新窗口（也就是说不是双击桌面浏览器图标等打开的窗口）除外**。这类子窗口会共享父窗口的Cookie，因此会共享一个session。
* 注意：**新开的浏览器窗口会生成新的session，但子窗口除外。子窗口会公用父窗口的session。**例如，在链接上右击，在弹出的快捷菜单中选择“在新窗口中打开”时，子窗口便可以访问父窗口的session
* 如果客户端浏览器将Cookie功能禁用，或者不支持Cookie怎么办？例如，绝大多数的手机浏览器都不支持Cookie。Java Web提供了另一种解决方案：URL重写

##### 多个服务部署时 Session 的管理

