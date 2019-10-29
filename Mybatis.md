### Mybatis

* 数据持久层框架。DAO层框架。对数据进行CRUD操作。内部封装了jdbc。

* JDBC：访问操作数据库的底层技术  规范之一。

* 通过xml或注解的方式，将要执行的各种Statement配置起来，并通过java对象和Statement中SQL的动态参数进行映射最终生成SQL语句，最终由mybatis框架执行SQL并将结果映射成Java对象并返回

* mybatis框架通过对象的**无参构造创建对象**，如果实体类中没有无参构造，会报出一下错误

  ```java
  ### Cause: org.apache.ibatis.reflection.ReflectionException: Error instantiating class entity.Msg with invalid types () or values (). Cause: java.lang.NoSuchMethodException: entity.Msg.<init>()
  ```

* JDBC编程问题：

  ​		1、编码冗余  6步骤

  ​		2、手工处理ORM

  ​		3、没有对查询相关优化(缓存 Cache)
  
  ​		4、底层没有用连接池，操作数据库频繁，消耗资源

##### Mybatis中的标签有哪些

* | 功能                                       | 标签        |
  | ------------------------------------------ | ----------- |
  | 定义sql 语句                               | insert      |
  |                                            | delete      |
  |                                            | update      |
  |                                            | select      |
  | 配置java对象属性与查询结果集中列名对应关系 | resultMap   |
  | id , result                                | resultType  |
  | 动态SQL                                    | foreach     |
  |                                            | if          |
  |                                            | choose      |
  | 格式化输出                                 | where       |
  |                                            | trim        |
  |                                            | set         |
  | 配置关联关系                               | association |
  |                                            | collection  |
  | 定义常量，公用SQL片段 include              | sql         |

##### 怎么配置Mybatis

* ~~~java
  <?xml version="1.0" encoding="UTF-8"?>
  
  <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  <!-- 注册属性文件 添加属性文件properties。 这个文件主要是可以让我们可以快速地修改数据库用户名，密码，切换数据源等-->
   <properties resource="jdbc.properties" />
   <!-- 配置mybatis运行环境 -->
    <environments default="development">
    <!-- 配置开发环境 -->
      <environment id="development">
      <!-- jdbc事务管理器 -->
        <transactionManager type="JDBC"/>
        <!-- 数据源
         UNPOOLED 不适用连接池 即 每次请求都会为其创建一个DB连接，适用完毕后，会马上将连接关闭
         POOLED 数据库连接池来维护连接
         JNDI 数据源可以定义到应用的外部，通过JDNI容器来获取数据库连接
         -->
        <dataSource type="POOLED">
          <property name="driver" value="${jdbc.driverClass}"/>
          <property name="url" value="${jdbc.url}"/>
          <property name="username" value="${jdbc.username}"/>
          <property name="password" value="${jdbc.password}"/>
        </dataSource>
      </environment>
      <!-- 可以多个环境切换。配置上线环境 -->
      <environment id="online">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
          <property name="driver" value="${jdbc.driverClass}"/>
          <property name="url" value="${jdbc.url}"/>
          <property name="username" value="${jdbc.username}"/>
          <property name="password" value="${jdbc.password}"/>
        </dataSource>
      </environment>
    </environments>
    <!-- 映射器 -->
    <mappers>
   	 <!-- 注册映射文件 -->
      	<mapper resource="mapper.xml"/>
     		<mapper resource="mapper2.xml"/>
    </mappers>
  </configuration>
  ~~~

##### Mybatis如何防止sql注入

~~~java
//看下面两个sql
<select id="selectByNameAndPassword" parameterType="java.util.Map" resultMap="BaseResultMap">
select id, username, password, role
from user
where username = #{username,jdbcType=VARCHAR}
and password = #{password,jdbcType=VARCHAR}
</select>
==============================================================
<select id="selectByNameAndPassword" parameterType="java.util.Map" resultMap="BaseResultMap">
select id, username, password, role
from user
where username = ${username,jdbcType=VARCHAR}
and password = ${password,jdbcType=VARCHAR}
</select>
~~~

* mybatis中的 # 与 $ 的区别

  * #将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号.
    如：where username=#{username}，如果传入的值是111,那么解析成sql时的值为where username="111", 如果传入的值是id，则解析成的sql为where username="id".
  * $将传入的数据直接显示生成在sql中.
    如：where username=${username}，如果传入的值是111,那么解析成sql时的值为where username=111；
    如果传入的值是;drop table user;，则解析成的sql为：select id, username, password, role from user where username=;drop table user;  这样会直接将user表删除
  * #方式能够很大程度防止sql注入，$方式无法防止Sql注入
  * 但是有时候${}还是很必要的,比如传入tableName,或者fieldName比如Order By id||time 就需要${}传入 #{}就无法搞定
  * 一般能用#的就别用$，若不得不使用“${xxx}”这样的参数，要手工地做好过滤工作，来防止sql注入攻击.
  * 在MyBatis中，“${xxx}”这样格式的参数会直接参与SQL编译，从而不能避免注入攻击。但涉及到动态表名和列名时，只能使用“${xxx}”这样的参数格式。所以，这样的参数需要我们在代码中手工进行处理来防止注入.

* 什么是SQL注入    SQL.md查看

* mybatis框架是如何防止SQL注入的

  * [MyBatis](https://mybatis.github.io/mybatis-3/)框架作为一款半自动化的持久层框架，其SQL语句都要我们自己手动编写，这个时候当然需要防止SQL注入。其实，MyBatis的SQL是一个具有“**输入+输出**”的功能，类似于函数的结构，参考上面的两个例子。其中，parameterType表示了输入的参数类型，resultType表示了输出的参数类型。回应上文，如果我们想防止SQL注入，理所当然地要在输入参数上下功夫。上面代码中使用#的即输入参数在SQL中拼接的部分，传入参数后，打印出执行的SQL语句，会看到SQL是这样的：

    ~~~java
    select id, username, password, role from user where username=? and password=?
    ~~~

    不管输入什么参数，打印出的SQL都是这样的。这是因为MyBatis启用了预编译功能，在SQL执行前，会先将上面的SQL发送给数据库进行编译；执行时，直接使用编译好的SQL，替换占位符“?”就可以了。因为SQL注入只能对编译过程起作用，所以这样的方式就很好地避免了SQL注入的问题。

  * 底层实现原理：MyBatis是如何做到SQL预编译的呢？其实在框架底层，是JDBC中的PreparedStatement类在起作用，PreparedStatement是我们很熟悉的Statement接口的子接口，它的实现类对象包含了编译好的SQL语句。这种“准备好”的方式不仅能提高安全性，而且在多次执行同一个SQL时，能够提高效率。原因是SQL已编译好，再次执行时无需再编译。

    ~~~java
    //安全的，预编译了的
    Connection conn = getConn();//获得连接
    String sql = "select id, username, password, role from user where id=?"; //执行sql前会预编译号该条语句
    PreparedStatement pstmt = conn.prepareStatement(sql); 
    pstmt.setString(1, id); 
    ResultSet rs=pstmt.executeUpdate(); 
    =========================================================
    //不安全的，没进行预编译
    private String getNameByUserId(String userId) {
        Connection conn = getConn();//获得连接
        String sql = "select id,username,password,role from user where id=" + id;
        //当id参数为"3;drop table user;"时，执行的sql语句如下:
        //select id,username,password,role from user where id=3; drop table user;  
        PreparedStatement pstmt =  conn.prepareStatement(sql);
        ResultSet rs=pstmt.executeUpdate();
    }
    ~~~

* 结论 

  * ~~~java
    #{} ： 相当于JDBC中的PreparedStatement
    ${} : 是输出变量的值
    ~~~

  * 简单说，#{}是经过预编译的，是安全的；${}是未经过预编译的，仅仅是取变量的值，是非安全的，存在SQL注入。如果我们order by语句后用了${}，那么不做任何处理的时候是存在SQL注入危险的。你说怎么防止，那我只能悲惨的告诉你，你得手动处理过滤一下输入的内容。如判断一下输入的参数的长度是否正常（注入语句一般很长），更精确的过滤则可以查询一下输入的参数是否在预期的参数集合中。

  * ${}是Properties文件中的变量占位符，它可以用于标签属性值和sql内部，属于静态文本替换，比如${driver}会被静态替换为com.MySQL.jdbc.Driver。#{}是sql的参数占位符，Mybatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0,parameterValue)，**#{item.name}的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。**

##### Mybatis怎么返回自动创建的id

* 

##### mybatis mapper接口方法重载问题

* 结论：可以有条件的进行重载。
* 为什么会有这个问题？：mybatis里面将接口里面的方法名称和配置文件里面的id属性进行唯一配对，在同一个命名空间下只能有一个id，那么所有函数名称相同的重载函数都会被绑定到一个id上，所以，如果要实现函数的重载，必须让一个SQL语句去适应多个函数的参数，如果是单纯的重载是肯定不行的（重载函数的定义就是参数相关），但是得益于mybatis的多种传参方式和隐性的分页功能，可以在接口里面进行函数重载，但是还是需要将所有的重载函数适配到同一个id的SQL上面去，仍然有很大的局限性，并不是可以随意的进行重载。

##### Mybatis工作的原理是什么

##### 使用Mybatis插入一条数据前怎么获取主键

##### Mabtis怎么做分页，怎么做批量插入

##### Mybatis的优点和缺点

##### mybatis 异常  Executor  was closed   原因：

* SQLSession 不能声明为成员变量。定义为成员变量在类中共享，如果当前线程使用完后关闭SQLSession，下一个线程会获取到一个关闭的连接对象，，导致这个问题。

##### Mapper文件

~~~java
/*namespace命名空间，作用就是对sql进行分类化管理，理解sql隔离
	注意：使用mapper代理方法开发，namespace有特殊重要的作用
*/
<mapper namespace="test">
	/* 	在映射文件中配置很多sql语句
		通过select执行数据库查询
		id：标识映射文件中的SQL
		将SQL语句封装到mappedStatement对象中，所以id称为statement的id
	*/
	<select id="" resultType=""></select>
~~~

**使用注解开发mybatis，如果项目资源路径下存在对应的XXXMapper.xml文件,那么此时不管我们有没有在mybatis-config.xml中使用这个mapper文件，程序都会报错**

##### mybatis框架和JDBC的区别

* JDBC是Java提供的一个操作数据库的API，MyBatis是一个支持普通SQL查询，存储过程和高级映射的优秀持久层框架。 MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及对结果集的检索封装。 MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数 据库中的记录