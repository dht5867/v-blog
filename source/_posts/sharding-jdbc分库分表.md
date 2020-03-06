---
title: shardingjdbc分库分表
date: 2019-07-04 14:07:32
tags:
    -多数据源
    -分库分表
---

## 理论基础
  数据切分根据其切分类型，可以分为两种方式
- 垂直（纵向）切分：垂直分库就是根据业务耦合性，将关联度低的不同表存储在不同的数据库。

- 水平（横向）切分：水平切分分为库内分表和分库分表，是根据表内数据内在的逻辑关系，将同一个表按不同的条件分散到多个数据库或多个表中，每个表中只包含一部分数据，从而使得单个表的数据量变小，达到分布式的效果。

### 垂直分表
垂直分表是基于数据库中的"列"进行，某个表字段较多，可以新建一张扩展表，将不经常用或字段长度较大的字段拆分出去到扩展表中。在字段很多的情况下（例如一个大表有100多个字段），通过"大表拆小表"，更便于开发与维护，也能避免跨页问题，MySQL底层是通过数据页存储的，一条记录占用空间过大会导致跨页，造成额外的性能开销

 
垂直切分优点：
解决业务系统层面的耦合，业务清晰；
与微服务的治理类似，也能对不同业务的数据进行分级管理、维护、监控、扩展等；
高并发场景下，垂直切分一定程度的提升IO、数据库连接数、单机硬件资源的瓶颈。
缺点：
部分表无法join，只能通过接口聚合方式解决，提升了开发的复杂度；
分布式事务处理复杂；
依然存在单表数据量过大的问题（需要水平切分）

### 水平切分
水平切分后同一张表会出现在多个数据库/表中，每个库/表的内容不同，一般按照时间区间或ID区间来切分。

### 分库分表带来的问题
  事务一致性问题
  分布式事务
  最终一致性
  跨节点关联查询 join 问题
  跨节点分页、排序、函数问题
  全局主键避重问题
  数据迁移、扩容问题

## 实践

### MySQL脚本
```
test_msg1 数据库1
-- ----------------------------
-- Table structure for t_order_0
-- ----------------------------
DROP TABLE IF EXISTS `t_order_0`;
CREATE TABLE `t_order_0` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` varchar(32) DEFAULT NULL COMMENT '顺序编号',
  `user_id` varchar(32) DEFAULT NULL COMMENT '用户编号',
  `userName` varchar(32) DEFAULT NULL COMMENT '用户名',
  `passWord` varchar(32) DEFAULT NULL COMMENT '密码',
  `user_sex` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=30 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Table structure for t_order_1
-- ----------------------------
DROP TABLE IF EXISTS `t_order_1`;
CREATE TABLE `t_order_1` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` varchar(32) DEFAULT NULL COMMENT '顺序编号',
  `user_id` varchar(32) DEFAULT NULL COMMENT '用户编号',
  `userName` varchar(32) DEFAULT NULL COMMENT '用户名',
  `passWord` varchar(32) DEFAULT NULL COMMENT '密码',
  `user_sex` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8mb4;
```
```
test_msg2 数据库2
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_order_0
-- ----------------------------
DROP TABLE IF EXISTS `t_order_0`;
CREATE TABLE `t_order_0` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` varchar(32) DEFAULT NULL COMMENT '顺序编号',
  `user_id` varchar(32) DEFAULT NULL COMMENT '用户编号',
  `userName` varchar(32) DEFAULT NULL COMMENT '用户名',
  `passWord` varchar(32) DEFAULT NULL COMMENT '密码',
  `user_sex` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Table structure for t_order_1
-- ----------------------------
DROP TABLE IF EXISTS `t_order_1`;
CREATE TABLE `t_order_1` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` varchar(32) DEFAULT NULL COMMENT '顺序编号',
  `user_id` varchar(32) DEFAULT NULL COMMENT '用户编号',
  `userName` varchar(32) DEFAULT NULL COMMENT '用户名',
  `passWord` varchar(32) DEFAULT NULL COMMENT '密码',
  `user_sex` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=30 DEFAULT CHARSET=utf8mb4;
```
### 引入pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.bmsoft</groupId>
  <artifactId>share-jdbc</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>share-jdbc</name>
  <description>Demo project for Spring Boot</description>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <exclusions>
        <exclusion>
          <groupId>com.zaxxer</groupId>
          <artifactId>HikariCP</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.1.1</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <optional>true</optional>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
        </exclusion>
      </exclusions>
      <version>1.1.10</version>
    </dependency>
    <!--sharding-jdbc -->
    <dependency>
      <groupId>com.dangdang</groupId>
      <artifactId>sharding-jdbc-core</artifactId>
      <version>1.5.4</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

### 数据源配置
```
#datasource
spring.devtools.remote.restart.enabled=false

#data source1
spring.datasource.test1.driverClassName=com.mysql.jdbc.Driver
spring.datasource.test1.url=jdbc:mysql://127.0.0.1:3306/test_msg1
spring.datasource.test1.username=root
spring.datasource.test1.password=root

#data source2
spring.datasource.test2.driverClassName=com.mysql.jdbc.Driver
spring.datasource.test2.url=jdbc:mysql://127.0.0.1:3306/test_msg2
spring.datasource.test2.username=root
spring.datasource.test2.password=root
```
### 分库策略的简单实现

```

import com.dangdang.ddframe.rdb.sharding.api.ShardingValue;
import com.dangdang.ddframe.rdb.sharding.api.strategy.database.SingleKeyDatabaseShardingAlgorithm;
import com.google.common.collect.Range;
import java.util.Collection;
import java.util.LinkedHashSet;

/**
 * 分库策略的简单实现
 */
public class ModuloDatabaseShardingAlgorithm implements SingleKeyDatabaseShardingAlgorithm<Long> {

  @Override
  public String doEqualSharding(Collection<String> databaseNames,
      ShardingValue<Long> shardingValue) {
    for (String each : databaseNames) {
      if (each.endsWith(Long.parseLong(shardingValue.getValue().toString()) % 2 + "")) {
        return each;
      }
    }
    throw new IllegalArgumentException();
  }

  @Override
  public Collection<String> doInSharding(Collection<String> databaseNames,
      ShardingValue<Long> shardingValue) {
    Collection<String> result = new LinkedHashSet<>(databaseNames.size());
    for (Long value : shardingValue.getValues()) {
      for (String tableName : databaseNames) {
        if (tableName.endsWith(value % 2 + "")) {
          result.add(tableName);
        }
      }
    }
    return result;
  }

  @Override
  public Collection<String> doBetweenSharding(Collection<String> databaseNames,
      ShardingValue<Long> shardingValue) {
    Collection<String> result = new LinkedHashSet<>(databaseNames.size());
    Range<Long> range = (Range<Long>) shardingValue.getValueRange();
    for (Long i = range.lowerEndpoint(); i <= range.upperEndpoint(); i++) {
      for (String each : databaseNames) {
        if (each.endsWith(i % 2 + "")) {
          result.add(each);
        }
      }
    }
    return result;
  }
}

```

### 分表的策略实现

``` 
import com.dangdang.ddframe.rdb.sharding.api.ShardingValue;
import com.dangdang.ddframe.rdb.sharding.api.strategy.table.SingleKeyTableShardingAlgorithm;
import com.google.common.collect.Range;
import java.util.Collection;
import java.util.LinkedHashSet;

/**
 * 分表策略的基本实现
 */
public class ModuloTableShardingAlgorithm implements SingleKeyTableShardingAlgorithm<Long> {

  @Override
  public String doEqualSharding(Collection<String> tableNames, ShardingValue<Long> shardingValue) {
    for (String each : tableNames) {
     /* if (each.endsWith(shardingValue.getValue() % 2 + "")) {
        return each;
      }*/
      if (each.endsWith(shardingValue.getValue() + "")) {
        return each;
      }
    }
    throw new IllegalArgumentException();
  }

  @Override
  public Collection<String> doInSharding(Collection<String> tableNames,
      ShardingValue<Long> shardingValue) {
    Collection<String> result = new LinkedHashSet<>(tableNames.size());
    for (Long value : shardingValue.getValues()) {
      for (String tableName : tableNames) {
        /*if (tableName.endsWith(value % 2 + "")) {
          result.add(tableName);
        }*/
        if (tableName.endsWith(value + "")) {
          result.add(tableName);
        }
      }
    }
    return result;
  }

  @Override
  public Collection<String> doBetweenSharding(Collection<String> tableNames,
      ShardingValue<Long> shardingValue) {
    Collection<String> result = new LinkedHashSet<>(tableNames.size());
    Range<Long> range = (Range<Long>) shardingValue.getValueRange();
    for (Long i = range.lowerEndpoint(); i <= range.upperEndpoint(); i++) {
      for (String each : tableNames) {
       /* if (each.endsWith(i % 2 + "")) {
          result.add(each);
        }*/
        if (each.endsWith(i + "")) {
          result.add(each);
        }

      }
    }
    return result;
  }
}

```
### 数据源初始化、绑定分库分表策略

```
/**
 * 分库分表最主要有几个配置： 有多少个数据源 每张表的逻辑表名和所有物理表名 用什么列进行分库以及分库算法 用什么列进行分表以及分表算法 分为两个库：test_msg1 , test_msg2，
 * 每个库都包含两个表: t_order_0 , t_order_1 使用user_id作为分库列； 使用order_id作为分表列；
 */
@Configuration
@MapperScan(basePackages = "com.bmsoft.sharejdbc.mapper", sqlSessionTemplateRef = "test1SqlSessionTemplate")
public class DataSourceConfig {

  /**
   * 配置数据源0，数据源的名称最好要有一定的规则，方便配置分库的计算规则
   */
  @Bean(name = "dataSource0")
  @ConfigurationProperties(prefix = "spring.datasource.test1")
  public DataSource dataSource0() {
    return new DruidDataSource();
  }

  /**
   * 配置数据源1，数据源的名称最好要有一定的规则，方便配置分库的计算规则
   */
  @Bean(name = "dataSource1")
  @ConfigurationProperties(prefix = "spring.datasource.test2")
  public DataSource dataSource1() {
    return new DruidDataSource();
  }

  /**
   * 配置数据源规则，即将多个数据源交给sharding-jdbc管理，并且可以设置默认的数据源， 当表没有配置分库规则时会使用默认的数据源
   */
  @Bean
  public DataSourceRule dataSourceRule(
      @Qualifier("dataSource0") DataSource dataSource0  , @Qualifier("dataSource1") DataSource dataSource1) {
    //设置分库映射
    Map<String, DataSource> dataSourceMap = new HashMap<>();
    dataSourceMap.put("dataSource0", dataSource0);
    dataSourceMap.put("dataSource1", dataSource1);
    //设置默认库，两个库以上时必须设置默认库。默认库的数据源名称必须是dataSourceMap的key之一
    return new DataSourceRule(dataSourceMap, "dataSource0");
  }

  /**
   * 配置数据源策略和表策略，具体策略需要自己实现
   */
  @Bean
  public ShardingRule shardingRule(DataSourceRule dataSourceRule) {
    //具体分库分表策略
    TableRule orderTableRule = TableRule.builder("t_order")
        .actualTables(Arrays.asList("t_order_1", "t_order_2"))
        .tableShardingStrategy(
            new TableShardingStrategy("order_id", new ModuloTableShardingAlgorithm()))
        .dataSourceRule(dataSourceRule)
        .build();

    //绑定表策略，在查询时会使用主表策略计算路由的数据源，因此需要约定绑定表策略的表的规则需要一致，可以一定程度提高效率
    List<BindingTableRule> bindingTableRules = new ArrayList<BindingTableRule>();
    bindingTableRules.add(new BindingTableRule(Arrays.asList(orderTableRule)));
    return ShardingRule.builder()
        .dataSourceRule(dataSourceRule)
        .tableRules(Arrays.asList(orderTableRule))
        .bindingTableRules(bindingTableRules)
        .databaseShardingStrategy(
            new DatabaseShardingStrategy("user_id", new ModuloDatabaseShardingAlgorithm()))
        .tableShardingStrategy(
            new TableShardingStrategy("order_id", new ModuloTableShardingAlgorithm()))
        .build();
  }

  /**
   * 创建sharding-jdbc的数据源DataSource，MybatisAutoConfiguration会使用此数据源
   */
  @Bean(name = "dataSource")
  public DataSource shardingDataSource(ShardingRule shardingRule) throws SQLException {
    return ShardingDataSourceFactory.createDataSource(shardingRule);
  }

  /**
   * 需要手动配置事务管理器
   */
  @Bean
  public DataSourceTransactionManager transactitonManager(
      @Qualifier("dataSource") DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
  }

  @Bean(name = "test1SqlSessionFactory")
  @Primary
  public SqlSessionFactory testSqlSessionFactory(@Qualifier("dataSource") DataSource dataSource)
      throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(dataSource);
    bean.setMapperLocations(
        new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/*.xml"));
    return bean.getObject();
  }

  @Bean(name = "test1SqlSessionTemplate")
  @Primary
  public SqlSessionTemplate testSqlSessionTemplate(
      @Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory);
  }
}
```
mapper 接口

```
public interface User1Mapper {

  List<UserEntity> getAll(UserEntity userEntity);

  void insert(UserEntity user);
}
```
xml 

```
<select id="getAll" resultMap="BaseResultMap"  parameterType="com.bmsoft.sharejdbc.entity.UserEntity" >
    SELECT
    <include refid="Base_Column_List" />
    FROM t_order
<where>
  <if test="user_id!=null">
    user_id = #{user_id}
  </if>
  <if test="order_id!=null">
    and order_id = #{order_id}
  </if>
</where>
  </select>

  <insert id="insert" parameterType="com.bmsoft.sharejdbc.entity.UserEntity" >
    INSERT INTO
    t_order
    (order_id,user_id,userName,passWord,user_sex)
    VALUES
    (#{order_id},#{user_id},#{userName}, #{passWord}, #{userSex})
  </insert>
  ```

User1Service
```
private User1Mapper user1Mapper;

  public List<UserEntity> getUsers(UserEntity userEntity) {
    List<UserEntity> users = user1Mapper.getAll(userEntity);
    return users;
  }

  //    @Transactional(value="test1TransactionManager",rollbackFor = Exception.class,timeout=36000)
  // 说明针对Exception异常也进行回滚，如果不标注，则Spring 默认只有抛出 RuntimeException才会回滚事务
  public void updateTransactional(UserEntity user) {
    try {
      user1Mapper.insert(user);
      log.error(String.valueOf(user));
    } catch (Exception e) {
      log.error("find exception!");
      // 事物方法中，如果使用trycatch捕获异常后，需要将异常抛出，否则事物不回滚。
      throw e;
    }

  }
  ```

  说明，
  1、查询所有的会把不同库和不同表的数据一起查询组合，新增时，会根据user_id分库，order_id 分表
  2、分库分表，需要定义主键保持全库唯一，这边没有处理

### 启动项配置
  ```
  @SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
@EnableTransactionManagement(proxyTargetClass = true)   //开启事物管理功能
public class ShareJdbcApplication {

  public static void main(String[] args) {
    SpringApplication.run(ShareJdbcApplication.class, args);
  }
}

```

