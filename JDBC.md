# JDBC

### 概述

JDBC（Java DataBase Connectivity，Java 数据库连接）是一种用于执行 SQL 语句的 Java API，可以为多种关系型数据库提供统一访问，是由一组用 Java 语言编写的类和接口组成的。

JDBC 是 Java 官方提供的一套规范（接口），用于帮助开发人员快速实现不同关系型数据库的连接



***



### 功能类

#### DriverManager

DriverManager：驱动管理对象

* 注册驱动：

  * 注册给定的驱动：`public static void registerDriver(Driver driver)`

  * 代码实现语法：`Class.forName("com.mysql.jdbc.Driver)`

  * com.mysql.jdbc.Driver 中存在静态代码块

    ```java
    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    ```

  * 不需要通过 DriverManager 调用静态方法 registerDriver，因为 Driver 类被使用，则自动执行静态代码块完成注册驱动

  * jar 包中 META-INF 目录下存在一个 java.sql.Driver 配置文件，文件中指定了 com.mysql.jdbc.Driver

* 获取数据库连接并返回连接对象：

  方法：`public static Connection getConnection(String url, String user, String password)`

  * url：指定连接的路径，语法为 `jdbc:mysql://ip地址(域名):端口号/数据库名称`
  * user：用户名
  * password：密码



***



#### Connection

Connection：数据库连接对象

- 获取执行者对象
  - 获取普通执行者对象：`Statement createStatement()`
  - 获取预编译执行者对象：`PreparedStatement prepareStatement(String sql)`
- 管理事务
  - 开启事务：`setAutoCommit(boolean autoCommit)`，false 开启事务，true 自动提交模式（默认）
  - 提交事务：`void commit()`
  - 回滚事务：`void rollback()`
- 释放资源
  - 释放此 Connection 对象的数据库和 JDBC 资源：`void close()`



***



#### Statement

Statement：执行 sql 语句的对象

- 执行 DML 语句：`int executeUpdate(String sql)`
  - 返回值 int：返回影响的行数
  - 参数 sql：可以执行 insert、update、delete 语句
- 执行 DQL 语句：`ResultSet executeQuery(String sql)`
  - 返回值 ResultSet：封装查询的结果
  - 参数 sql：可以执行 select 语句
- 释放资源
  - 释放此 Statement 对象的数据库和 JDBC 资源：`void close()`



***



#### ResultSet

ResultSet：结果集对象，ResultSet 对象维护了一个游标，指向当前的数据行，初始在第一行

- 判断结果集中是否有数据：`boolean next()`
  - 有数据返回 true，并将索引**向下移动一行**
  - 没有数据返回 false
- 获取结果集中**当前行**的数据：`XXX getXxx("列名")`
  - XXX 代表数据类型（要获取某列数据，这一列的数据类型）
  - 例如：String getString("name");   int getInt("age");
- 释放资源
  - 释放 ResultSet 对象的数据库和 JDBC 资源：`void close()`



***



#### 代码实现

数据准备

```mysql
-- 创建db14数据库
CREATE DATABASE db14;

-- 使用db14数据库
USE db14;

-- 创建student表
CREATE TABLE student(
	sid INT PRIMARY KEY AUTO_INCREMENT,	-- 学生id
	NAME VARCHAR(20),					-- 学生姓名
	age INT,							-- 学生年龄
	birthday DATE,						-- 学生生日
);

-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,'1999-09-23'),(NULL,'李四',24,'1998-08-10'),
(NULL,'王五',25,'1996-06-06'),(NULL,'赵六',26,'1994-10-20');
```

JDBC 连接代码：

```java
public class JDBCDemo01 {
    public static void main(String[] args) throws Exception{
        //1.导入jar包
        //2.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //3.获取连接
        Connection con = DriverManager.getConnection("jdbc:mysql://192.168.2.184:3306/db2","root","123456");

        //4.获取执行者对象
        Statement stat = con.createStatement();

        //5.执行sql语句，并且接收结果
        String sql = "SELECT * FROM user";
        ResultSet rs = stat.executeQuery(sql);

        //6.处理结果
        while(rs.next()) {
            System.out.println(rs.getInt("id") + "\t" + rs.getString("name"));
        }

        //7.释放资源
        con.close();
        stat.close();
        con.close();
    }
}

```







***



### 注入攻击

#### 攻击演示

SQL 注入攻击演示

* 在登录界面，输入一个错误的用户名或密码，也可以登录成功 

  ![](https://seazean.oss-cn-beijing.aliyuncs.com/img/DB/SQL注入攻击演示.png)

* 原理：我们在密码处输入的所有内容，都应该认为是密码的组成，但是 Statement 对象在执行 SQL 语句时，将一部分内容当做查询条件来执行

  ```mysql
  SELECT * FROM user WHERE loginname='aaa' AND password='aaa' OR '1'='1';
  ```




***



#### 攻击解决

PreparedStatement：预编译 sql 语句的执行者对象，继承 `PreparedStatement extends Statement`

* 在执行 sql 语句之前，将 sql 语句进行提前编译，**明确 sql 语句的格式**，剩余的内容都会认为是参数
* sql 语句中的参数使用 ? 作为**占位符**

为 ? 占位符赋值的方法：`setXxx(int parameterIndex, xxx data)`

- 参数1：? 的位置编号（编号从 1 开始）

- 参数2：? 的实际参数

  ```java
  String sql = "SELECT * FROM user WHERE loginname=? AND password=?";
  pst = con.prepareStatement(sql);
  pst.setString(1,loginName);
  pst.setString(2,password);
  ```

执行 sql 语句的方法

- 执行 insert、update、delete 语句：`int executeUpdate()`
- 执行 select 语句：`ResultSet executeQuery()`





****



### 连接池

#### 概念

数据库连接背景：数据库连接是一种关键的、有限的、昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。

数据库连接池：**数据库连接池负责分配、管理和释放数据库连接**，它允许应用程序**重复使用**一个现有的数据库连接，而不是再重新建立一个，这项技术能明显提高对数据库操作的性能。

数据库连接池原理

![](https://seazean.oss-cn-beijing.aliyuncs.com/img/DB/数据库连接池原理图解.png)







***





#### 归还连接

使用动态代理的方式来改进

自定义数据库连接池类：

```java
public class MyDataSource implements DataSource {
    //1.准备一个容器。用于保存多个数据库连接对象
    private static List<Connection> pool = Collections.synchronizedList(new ArrayList<>());

    //2.定义静态代码块,获取多个连接对象保存到容器中
    static{
        for(int i = 1; i <= 10; i++) {
            Connection con = JDBCUtils.getConnection();
            pool.add(con);
        }
    }
    //3.提供一个获取连接池大小的方法
    public int getSize() {
        return pool.size();
    }

   	//动态代理方式
    @Override
    public Connection getConnection() throws SQLException {
        if(pool.size() > 0) {
            Connection con = pool.remove(0);

            Connection proxyCon = (Connection) Proxy.newProxyInstance(
                con.getClass().getClassLoader(), new Class[]{Connection.class}, 
                new InvocationHandler() {
                /*
                    执行Connection实现类连接对象所有的方法都会经过invoke
                    如果是close方法，归还连接
                    如果不是，直接执行连接对象原有的功能即可
                 */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    if(method.getName().equals("close")) {
                        //归还连接
                        pool.add(con);
                        return null;
                    }else {
                        return method.invoke(con,args);
                    }
                }
            });
            return proxyCon;
        }else {
            throw new RuntimeException("连接数量已用尽");
        }
    }
}
```



***



#### 开源项目

##### C3P0

使用 C3P0 连接池：

* 配置文件名称：c3p0-config.xml，必须放在 src 目录下

  ```xml
  <c3p0-config>
    <!-- 使用默认的配置读取连接池对象 -->
    <default-config>
    	<!--  连接参数 -->
      <property name="driverClass">com.mysql.jdbc.Driver</property>
      <property name="jdbcUrl">jdbc:mysql://192.168.2.184:3306/db14</property>
      <property name="user">root</property>
      <property name="password">123456</property>
      
      <!-- 连接池参数 -->
      <!--初始化数量-->
      <property name="initialPoolSize">5</property>
      <!--最大连接数量-->
      <property name="maxPoolSize">10</property>
      <!--超时时间 3000ms-->
      <property name="checkoutTimeout">3000</property>
    </default-config>
  
    <named-config name="otherc3p0"> 
      <!--  连接参数 -->
      <!-- 连接池参数 -->
    </named-config>
  </c3p0-config>
  ```

* 代码演示

  ```java
  public class C3P0Test1 {
      public static void main(String[] args) throws Exception{
          //1.创建c3p0的数据库连接池对象
          DataSource dataSource = new ComboPooledDataSource();
  
          //2.通过连接池对象获取数据库连接
          Connection con = dataSource.getConnection();
  
          //3.执行操作
          String sql = "SELECT * FROM student";
          PreparedStatement pst = con.prepareStatement(sql);
  
          //4.执行sql语句，接收结果集
          ResultSet rs = pst.executeQuery();
  
          //5.处理结果集
          while(rs.next()) {
              System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
          }
  
          //6.释放资源
          rs.close();   pst.close();   con.close();
      }
  }
  ```




***



##### Druid

Druid 连接池：

* 配置文件：druid.properties，必须放在 src 目录下

  ```properties
  driverClassName=com.mysql.jdbc.Driver
  url=jdbc:mysql://192.168.2.184:3306/db14
  username=root
  password=123456
  initialSize=5
  maxActive=10
  maxWait=3000
  ```

* 代码演示

  ```java
  public class DruidTest1 {
      public static void main(String[] args) throws Exception{
          //获取配置文件的流对象
          InputStream is = DruidTest1.class.getClassLoader().getResourceAsStream("druid.properties");
  
          //1.通过Properties集合，加载配置文件
          Properties prop = new Properties();
          prop.load(is);
  
          //2.通过Druid连接池工厂类获取数据库连接池对象
          DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);
  
          //3.通过连接池对象获取数据库连接进行使用
          Connection con = dataSource.getConnection();
          
  		//4.执行sql语句，接收结果集
          String sql = "SELECT * FROM student";
          PreparedStatement pst = con.prepareStatement(sql);
          ResultSet rs = pst.executeQuery();
  
          //5.处理结果集
          while(rs.next()) {
              System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
          }
  
          //6.释放资源
          rs.close();   pst.close();   con.close();
      }
  }
  
  ```