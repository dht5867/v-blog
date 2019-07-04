---
title: springboot多数据源切换注解方式
date: 2019-07-03 19:53:54
tags:
    -多数据源
---

# springboot多数据源切换注解方式

## 配置连接数据库信息
```
spring.datasource.inner.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.inner.url=jdbc:mysql://
spring.datasource.inner.username=root
spring.datasource.inner.password=root 

spring.datasource.third.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.third.url=jdbc:mysql:
spring.datasource.third.username=root
spring.datasource.third.password=root 
```
## 数据源配置

```
@Configuration
public class DbDuridMysqlProperties {

  @Autowired
  private DataSourceProperties dataSourceProperties;

  @Bean(name = "innerDataSource", destroyMethod = "close", initMethod = "init")
  @ConfigurationProperties("spring.datasource.inner")
  public DataSource innerDataSource() throws SQLException {
    DruidDataSource druidDataSource = new DruidDataSource();
    setDataSourcePool(druidDataSource);
    return druidDataSource;
  }

  @Bean(name = "thirdDataSource", destroyMethod = "close", initMethod = "init")
  @ConfigurationProperties("spring.datasource.third")
  public DataSource thirdDataSource() throws SQLException {
    DruidDataSource druidDataSource = new DruidDataSource();
    setDataSourcePool(druidDataSource);
    return druidDataSource;
  }
}
```

## 动态数据源持有者
```
/**
 * 动态数据源持有者，负责利用ThreadLocal存取数据源名称
 */
public class DynamicDataSourceHolder {

  /**
   * 本地线程共享对象
   */
  private static final ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<>();

  public static void putDataSource(String name) {
    THREAD_LOCAL.set(name);
  }

  public static String getDataSource() {
    return THREAD_LOCAL.get();
  }

  public static void removeDataSource() {
    THREAD_LOCAL.remove();
  }
}
```

## 动态数据源实现类

```
/**
 * 动态数据源实现类
 */
@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {

  //数据源路由，此方用于产生要选取的数据源逻辑名称
  @Override
  protected Object determineCurrentLookupKey() {
    //从共享线程中获取数据源名称
    return DynamicDataSourceHolder.getDataSource();
  }
}
```

## 定于数据源注解,主要用于dao切换数据源
```
/**
 * 目标数据源注解，注解在方法上指定数据源的名称
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TargetDataSource {

  //此处接收的是数据源的名称
  String value();
}
```
## 数据源配置,加载配置的数据源，形成key - 数据源
```
package com.bmsoft.newborn.config.datasource;

import com.bmsoft.newborn.config.dds.DynamicDataSource;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;
import lombok.extern.slf4j.Slf4j;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

/**
 * 数据源配置
 */
@Configuration
@Slf4j
@MapperScan({"com.bmsoft.newborn.mapper.inner", "com.bmsoft.newborn.mapper.third"})
public class DataSourceConfig {


  @Autowired
  private DbDuridMysqlProperties duridMysqlProperties;


  /**
   * @Primary 该注解表示在同一个接口有多个实现类可以注入的时候，默认选择哪一个，而不是让@autowire注解报错
   * @Qualifier 根据名称进行注入，通常是在具有相同的多个类型的实例的一个注入（例如有多个DataSource类型的实例）
   */
  @Bean(name = "dataSource")
  @Primary
  public DynamicDataSource dataSource() throws SQLException {
    //按照目标数据源名称和目标数据源对象的映射存放在Map中
    //采用是想AbstractRoutingDataSource的对象包装多数据源
    DynamicDataSource dynamicDataSource = new DynamicDataSource();
    Map<Object, Object> targetDataSources = new HashMap<>();
    targetDataSources.put("inner", duridMysqlProperties.innerDataSource());
    targetDataSources.put("third", duridMysqlProperties.thirdDataSource());

    dynamicDataSource.setTargetDataSources(targetDataSources);
    //设置默认的数据源，当拿不到数据源时，使用此配置
    dynamicDataSource.setDefaultTargetDataSource(duridMysqlProperties.innerDataSource());
    return dynamicDataSource;
  }

  //开启事务
  @Bean
  public PlatformTransactionManager txManager() throws SQLException {
    return new DataSourceTransactionManager(dataSource());
  }


}
```

##  数据源AOP切面定义
```
package com.bmsoft.newborn.config.datasource;

import com.bmsoft.newborn.annotation.TargetDataSource;
import com.bmsoft.newborn.config.dds.DynamicDataSourceHolder;
import java.lang.reflect.Method;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

/**
 * 数据源AOP切面定义
 */
@Component
@Aspect
@Slf4j
@Order(4)
public class DataSourceAspect {

  @Autowired
  private Environment env;


  //切换放在mapper接口的方法上，所以这里要配置AOP切面的切入点
  @Pointcut("execution( * com.bmsoft.newborn.mapper.*.*.*(..))")
  public void dataSourcePointCut() {
  }

  @Before("dataSourcePointCut()")
  public void before(JoinPoint joinPoint) throws Exception {
    Object target = joinPoint.getTarget();
    /**
     * Signature 包含了方法名、申明类型以及地址等信息
     */
    //方法名称
    String method = joinPoint.getSignature().getName();
    //参数值
    Object[] paramValues = joinPoint.getArgs();
    Class<?>[] clazz = target.getClass().getInterfaces();
    Class<?>[] parameterTypes = ((MethodSignature) joinPoint.getSignature()).getMethod()
        .getParameterTypes();
    //获取参数类型
    try {
      //类上或者方法上存在注解
      Method m = clazz[0].getMethod(method, parameterTypes);
      Class<?> declaringClass = m.getDeclaringClass();
      if (declaringClass.isAnnotationPresent(TargetDataSource.class)) {
        TargetDataSource data = declaringClass.getAnnotation(TargetDataSource.class);
        changeDataSource(data);
      } else if (m.isAnnotationPresent(TargetDataSource.class)) {
        TargetDataSource data = m.getAnnotation(TargetDataSource.class);
        //可以根据参数获取当前数据源
        changeDataSource(data);
      }
    } catch (Exception e) {
      log.error("当前线程 " + Thread.currentThread().getName() + " 添加数据源失败", e);
    }
  }

  private void changeDataSource(TargetDataSource data) {
    //可以根据参数获取当前数据源
    String deafultSourceName = data.value();
    log.info(
            "使用数据源 " + Thread.currentThread().getName() + " 添加数据源到-- " + deafultSourceName
                    + " 本地线程变量");
    DynamicDataSourceHolder.putDataSource(deafultSourceName);
  }

  private void changeDataSource(Object[] paramValues, TargetDataSource data) {
    //可以根据参数获取当前数据源
    String deafultSourceName = data.value();
    if (paramValues != null && paramValues.length > 0) {
      log.info(
          "使用数据源 " + Thread.currentThread().getName() + " 添加数据源到-- " + deafultSourceName
              + " 本地线程变量");
      DynamicDataSourceHolder.putDataSource(deafultSourceName);
    }
  }


  //执行完切面后，将线程共享中的数据源名称清空
  @After("dataSourcePointCut()")
  public void after(JoinPoint joinPoint) {
    DynamicDataSourceHolder.removeDataSource();
  }


}
```
## springboot 关闭数据源自动配置，改为手动配置
```
package com.bmsoft.newborn;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class MicroNewbornServiceApplication {

  @Bean
  public BCryptPasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder();
  }

  public static void main(String[] args) {
    SpringApplication.run(MicroNewbornServiceApplication.class, args);
  }

}
```

## 在mapper接口上使用数据源注解到目标数据源
```
package com.bmsoft.newborn.mapper.inner;


import com.bmsoft.newborn.annotation.TargetDataSource;
import com.bmsoft.newborn.domain.inner.Dict;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@TargetDataSource("inner")
public interface DictMapper  {

  Dict selectById(@Param("code") String code);

  List<Dict> selectAllDict();

}
```
## 注意
1、同一个service中如果启动事务注解@Transaction，数据源切换将不起作用，需要改为分布式事务
2、需要注意配置默认的数据源，不然在非注解的情况下，查询SQL会报错
3、在aop 切换数据源时，可以在mapper中传入固定的参数，解析参数可以做切换数据源处理,利用反射获取主键方法的方法传参值进行切换
```

/**
 * 数据源AOP切面定义
 */
@Component
@Aspect
@Slf4j
@Order(4)
@PropertySource("classpath:datasource.properties")//这是多数据源属性文件路径
public class DataSourceAspect {

    @Autowired
    private Environment env;


    //切换放在mapper接口的方法上，所以这里要配置AOP切面的切入点
    @Pointcut("execution( * com.bmsoft.wfy.mapper.*.*.*(..))")
    public void dataSourcePointCut() {
    }

    @Before("dataSourcePointCut()")
    public void before(JoinPoint joinPoint) throws Exception {
        Object target = joinPoint.getTarget();
        /**
         * Signature 包含了方法名、申明类型以及地址等信息
         */
        String class_name = joinPoint.getTarget().getClass().getName();

        //方法名称
        String method = joinPoint.getSignature().getName();
        //参数值
        Object[] paramValues = joinPoint.getArgs();
        Class<?>[] clazz = target.getClass().getInterfaces();
        Class<?>[] parameterTypes = ((MethodSignature) joinPoint.getSignature()).getMethod().getParameterTypes();
        //获取参数类型
        try {
            Method m = clazz[0].getMethod(method, parameterTypes);
            //如果方法上存在切换数据源的注解，则根据注解内容进行数据源切换
            if (m != null && m.isAnnotationPresent(TargetDataSource.class)) {
                TargetDataSource data = m.getAnnotation(TargetDataSource.class);
                //可以根据参数获取当前数据源
                String deafultSourceName = data.value();
                if (paramValues != null && paramValues.length > 0) {
                    for (int i = 0; i < paramValues.length; i++) {
                        log.info("数据源方法" + method);
                        log.info("请求参数类型" + parameterTypes[i]);
                        if (parameterTypes[i] == Map.class) {
                            log.info("Map类型值" + paramValues[i]);
                            Map<String, Object> map = (HashMap) paramValues[i];
                            log.info("转换之后的Map类型值" + map);
                            //获取法院代码
                            Object courtNumber = map.get("courtNumber");
                            String court ="";
                            if(courtNumber!=null){
                                court =String.valueOf(courtNumber);
                            }
                            changeDataSource(deafultSourceName, courtNumber, court);

                        } else if (parameterTypes[i] == List.class) {
                            log.info("List类型值" + paramValues[i]);
                        } else {
                            log.info("对象类型值" + paramValues[i]);
                            final Map<String, String> stringStringMap = obj2Map(paramValues[i]);
                            log.info("对象转为map后的值" + stringStringMap);
                            Object courtNumber = stringStringMap.get("courtNumber");
                            String court ="";
                            if(courtNumber!=null){
                                court =String.valueOf(courtNumber);
                            }
                            changeDataSource(deafultSourceName, courtNumber, court);
                        }

                    }
                }
            } else {
                //TODO,默认MySQL数据源的处理
                //mapper 接口没有数据源注解,使用MySQL数据库
                log.info("使用默认mysql数据源");
            }
        } catch (Exception e) {
            log.error("当前线程 " + Thread.currentThread().getName() + " 添加数据源失败", e);
        }
    }

    //更换连接的数据源
    private void changeDataSource(String deafultSourceName, Object courtNumber, String court) {
        if (StringUtils.isNotBlank(court)&& !"".equals(court) &&!"null".equalsIgnoreCase(court)&&court.length()>=4) {
            court = court.substring(0, 4);
            //TODO,当前只有一个Sybase数据库,后期多个进行扩展
            if ( StringUtils.isNotBlank(deafultSourceName)&&deafultSourceName.indexOf("sybase") != -1) {
                String key = "datasource.sybase." + court;
                //使用 3200 3201 3202  
                String dataSourceName = env.getProperty(key);
                DynamicDataSourceHolder.putDataSource(dataSourceName);
                log.info("使用线程 " + Thread.currentThread().getName() + " sybase添加--案件 " + "法院代码---" + courtNumber + "--的数据源----" + dataSourceName + " 本地线程变量");

            } else if( StringUtils.isNotBlank(deafultSourceName)&&deafultSourceName.indexOf("mysql") != -1){
                //使用其他库的判断
                String key = "datasource.mysql." + court;
                //使用 3200 3201 3202  
                String dataSourceName = env.getProperty(key);
                DynamicDataSourceHolder.putDataSource(dataSourceName);
                log.info("使用线程 " + Thread.currentThread().getName() + "mysql 添加--执行 " + "法院代码---" + courtNumber + "--的数据源----" + dataSourceName + " 本地线程变量");

            }else if( StringUtils.isNotBlank(deafultSourceName)&&deafultSourceName.indexOf("oa") != -1){
                //使用其他库的判断
                //log.info("使用线程 " + Thread.currentThread().getName() + "sybaseoa 添加--执行 " + "法院代码---" + courtNumber + "--的数据源----" + dataSourceName + " 本地线程变量");
                log.info("使用默认江苏高院OA");

                String key = "datasource.oa.3200";
                //使用 3200 3201 3202 的办公oa
                String dataSourceName = env.getProperty(key);
                DynamicDataSourceHolder.putDataSource(dataSourceName);
                log.info("使用线程 " + Thread.currentThread().getName() + "sybaseoa 添加--执行 " + "法院代码---" + courtNumber + "--的数据源----" + dataSourceName + " 本地线程变量");
            }
        } else {
            //TODO,没有传法院代码的情况默认使用集中库
            //判断注解上的值,为集中库上的值
            if ("sybaseTtre".equalsIgnoreCase(deafultSourceName)) {
                DynamicDataSourceHolder.putDataSource(deafultSourceName);
                log.info("使用Sybase集中库线程 " + Thread.currentThread().getName() + " 添加数据源到-- " + deafultSourceName + " 本地线程变量");
            }else{
                //TODO 当事人测试使用,后期会更改
                DynamicDataSourceHolder.putDataSource("sybaseTtre");
                log.info("使用Sybase集中库线程 " + Thread.currentThread().getName() + " 添加数据源到-- " + deafultSourceName + " 本地线程变量");
                //TODO,目前不传法院代码,只有一个集中库的情况,后期进行扩展
            }
        }
    }

    private static Map<String, String> obj2Map(Object obj) {
        Map<String, String> map = new HashMap<String, String>();
        // System.out.println(obj.getClass());
        // 获取f对象对应类中的所有属性域
        Field[] fields = obj.getClass().getDeclaredFields();
        for (int i = 0, len = fields.length; i < len; i++) {
            String varName = fields[i].getName();
            //varName = varName.toLowerCase();//将key置为小写，默认为对象的属性
            try {
                // 获取原来的访问控制权限
                boolean accessFlag = fields[i].isAccessible();
                // 修改访问控制权限
                fields[i].setAccessible(true);
                // 获取在对象f中属性fields[i]对应的对象中的变量
                Object o = fields[i].get(obj);
                if (o != null) {
                    map.put(varName, o.toString());
                }
                // System.out.println("传入的对象中包含一个如下的变量：" + varName + " = " + o);
                // 恢复访问控制权限
                fields[i].setAccessible(accessFlag);
            } catch (IllegalArgumentException ex) {
                ex.printStackTrace();
            } catch (IllegalAccessException ex) {
                ex.printStackTrace();
            }
        }
        return map;
    }

    //执行完切面后，将线程共享中的数据源名称清空
    @After("dataSourcePointCut()")
    public void after(JoinPoint joinPoint) {
        DynamicDataSourceHolder.removeDataSource();
    }

    public static void main(String[] args) {
        String courtNumer = "320100";
        System.out.println(courtNumer.substring(0, 4));
    }
    
}

```