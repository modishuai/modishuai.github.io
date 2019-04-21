---
title: MyBatis核心组件作用域和生命周期
date: 2019-04-21 22:21:19
categories:
- MyBatis
tags:
- Java
- MyBatis
---

# MyBatis 核心组件

- SqlSessionFactoryBuilder（构建器）使用配置和 Java 代码创建 SqlSessionFactory，之后就不再需要它了，因此他作为方法作用域（局部方法变量）。

- SqlSessionFactory（工厂接口）创建成功后长期驻留在应用运行期间，因此作为应用作用域。依靠它来生成 SqlSession。

- SqlSession（会话）SqlSession 实例不是线程安全的，因此不能不共享，所以它的作用域应是请求或方法作用域。可以用来发送 SQL 执行返回结果，也可以用来获取 Mapper 的接口。

- Sql Mapper（映射器）用来发送 SQL 执行并返回结果。由 Java 接口和  XML 文件（或注解）构成，需要给出相对应的 SQL 和映射规则。

![MyBatis核心组件](MyBatis核心组件作用域和生命周期/MyBatis核心组件.jpg)

# SqlSessionFactoryBuilder

SqlSessionFactoryBuilder 基于构建器模式，主要目的是创建 SqlFactory，创建之后就功臣身退，不需要长期存在，所以作为方法作用域。

# SqlSessionFactory

SqlSessionFactory 的生命周期应伴随 Mybatis 应用运行存在，所以一经创建就要长期保存它，直到你不在运行 Mybatis 应用。SqlSessionFactory 可以看做是一个数据库连接池，占着数据库连接资源，如果创建多个 SqlSessionFactory  就存在多个数据库连接池，不利于对数据库资源控制。因此 SqlSessionFactory 可作为一个单例，在应用中被共享。

MyBatis 提供 SqlSessionFactoryBuilder 构建器根据 XML 配置或 Java 代码来生成 SqlSessionFactory。SqlSessionFactory 有两个实现类：DefaultSqlSessionFactory 和 SqlSessionManager（使用在多线程环境）

![SqlSessionFactory](MyBatis核心组件作用域和生命周期/SqlSessionFactory生成.jpg)

```java
public class SqlSessionFactoryUtils {

	private final static Class<SqlSessionFactoryUtils> LOCK = SqlSessionFactoryUtils.class;
	private static SqlSessionFactory sqlSessionFactory = null;
	private SqlSessionFactoryUtils() {}
	/**
	 * 通过 xml 配置文件构建 SqlSessionFactory
	 * @return
	 */
	public static SqlSessionFactory getSqlSessionFactoryForXml() {
		synchronized (LOCK) {
			if(sqlSessionFactory!=null) {
				return sqlSessionFactory;
			}
			String resource = "mybatis-config.xml";
			InputStream inputStream;
			try {
				inputStream = Resources.getResourceAsStream(resource);
				sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
			}catch(IOException e) {
				e.printStackTrace();
				return null;
			}
			return sqlSessionFactory;
		}
	}
	
	/**
	 * 通过代码构建 SqlSessionFactory 
	 * @return
	 */
	public static SqlSessionFactory getSqlSessionFactoryForCode() {
		synchronized (LOCK) {
			PooledDataSource dataSource = new PooledDataSource();
			dataSource.setDriver("com.mysql.jdbc.Driver");
			dataSource.setUsername("root");
			dataSource.setPassword("P@ssw0rd");
			dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/blogs?serverTimezone=GMT");
			dataSource.setDefaultAutoCommit(false);
			TransactionFactory transactionFactory = new JdbcTransactionFactory();
			Environment environment = new Environment("development", transactionFactory, dataSource);
			Configuration configuration = new Configuration(environment);
			configuration.getTypeAliasRegistry().registerAlias("article", Article.class);
			configuration.addMapper(ArticleMappper.class);
			SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
			return sqlSessionFactory;
		}
	}
	public static SqlSession openSqlSession() {
		if (sqlSessionFactory == null) {
			getSqlSessionFactoryForXml();
		}
		return sqlSessionFactory.openSession();
	}
}

```

# SqlSession

 每个线程都应该有自己的 SqlSession 实例， SqlSession 实例不是线程安全的，因此不能被共享，它的作用域是基于请求或方法作用域。注意：不能将 SqlSession 实例引用放到一个类的静态域中或一个类的实例变量，也不可以将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。如果使用 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。即收到 HTTP 请求就打开一个 SqlSession，得到响应就将其关闭。

主要用途如下：

- 获取 Mapper 接口

- 发送 SQL 给数据库

- 控制数据库事务

# Mapper

Mapper 接口的实例是从 SqlSession 中获得，因此它的最大作用域和 SqlSession 是相同的，由一个接口和一个相对应的 XML 文件或注解组成。配置内容包括：

- 描述映射规则

- 提供 SQL 语句，并可以配置 SQL 参数类型，返回类型、缓存刷新等信息；

- 配置缓存

- 提供动态 SQL

## XML 实现映射器

```haml
<mappers>
    <mapper resource="mds/learn/mapper/ArticleMapper.xml" />
</mappers>
```

```haml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="mds.learn.mapper.ArticleMappper">
      <select id="getArticle" parameterType="long" resultType="article">
          select id,title,author from m_article where id=#{id}
      </select>
  </mapper>
```

## 注解实现映射器

```java
	@Select("select id,title,author from m_article where id=#{id}")
	public Article getArticle(Long id);
```

```haml
<mappers>
	<mapper class="mds.learn.mapper.ArticleMappper2" />
</mappers>
```

或使用 Configuration 实例对象注册该接口

```java
configuration.addMapper(ArticleMappper2.class);
```
