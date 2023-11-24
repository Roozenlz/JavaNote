# Jedis

### 基本使用

Jedis 用于 Java 语言连接 Redis 服务，并提供对应的操作 API

* jar 包导入

  下载地址：https://mvnrepository.com/artifact/redis.clients/jedis

  基于 maven：

  ```xml
  <dependency>
  	<groupId>redis.clients</groupId>
  	<artifactId>jedis</artifactId>
  	<version>2.9.0</version>
  </dependency>
  ```

* 客户端连接 Redis：API 文档 http://xetorthio.github.io/jedis/

  连接 redis：`Jedis jedis = new Jedis("192.168.0.185", 6379)`
  
  操作 redis：`jedis.set("name", "seazean");  jedis.get("name")`
  
  关闭 redis：`jedis.close()`

代码实现：

```java
public class JedisTest {
    public static void main(String[] args) {
        //1.获取连接对象
        Jedis jedis = new Jedis("192.168.2.185",6379);
        //2.执行操作
        jedis.set("age","39");
        String hello = jedis.get("hello");
        System.out.println(hello);
        jedis.lpush("list1","a","b","c","d");
        List<String> list1 = jedis.lrange("list1", 0, -1);
        for (String s:list1 ) {
            System.out.println(s);
        }
        jedis.sadd("set1","abc","abc","def","poi","cba");
        Long len = jedis.scard("set1");
        System.out.println(len);
        //3.关闭连接
        jedis.close();
    }
}
```



***



### 工具类

连接池对象：

* JedisPool：Jedis 提供的连接池技术
* poolConfig：连接池配置对象 
* host：Redis 服务地址
* port：Redis 服务端口号

JedisPool 的构造器如下：

```java
public JedisPool(GenericObjectPoolConfig poolConfig, String host, int port) {
	this(poolConfig, host, port, 2000, (String)null, 0, (String)null);
}
```

* 创建配置文件 redis.properties

  ```properties
  redis.maxTotal=50
  redis.maxIdel=10
  redis.host=192.168.2.185
  redis.port=6379
  ```

* 工具类：

  ```java
  public class JedisUtils {
      private static int maxTotal;
      private static int maxIdel;
      private static String host;
      private static int port;
      private static JedisPoolConfig jpc;
      private static JedisPool jp;
  
      static {
          ResourceBundle bundle = ResourceBundle.getBundle("redis");
          //最大连接数
          maxTotal = Integer.parseInt(bundle.getString("redis.maxTotal"));
          //活动连接数
          maxIdel = Integer.parseInt(bundle.getString("redis.maxIdel"));
          host = bundle.getString("redis.host");
          port = Integer.parseInt(bundle.getString("redis.port"));
  
          //Jedis连接配置
          jpc = new JedisPoolConfig();
          jpc.setMaxTotal(maxTotal);
          jpc.setMaxIdle(maxIdel);
          //连接池对象
          jp = new JedisPool(jpc, host, port);
      }
  
      //对外访问接口，提供jedis连接对象，连接从连接池获取
      public static Jedis getJedis() {
          return jp.getResource();
      }
  }
  ```