---
title: springboot项目混淆方案
date: 2019-07-02 16:28:54
tags:
    - springboot混淆
---

# springboot 项目混淆

proguard简单来说是为了防止反编译，更准确的说，进行业务代码的混淆，是使得代码易读性变差。

## 引入proguard-maven-plugin

```
<plugin>
        <groupId>com.github.wvengen</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.14</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>proguard</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <proguardVersion>6.0.3</proguardVersion>
          <injar>${project.build.finalName}.jar</injar>
          <!-- <injar>classes</injar> -->
          <outjar>${project.build.finalName}.jar</outjar>
          <!--
                    <outjar>${project.build.finalName}.jar</outjar>
          -->
          <obfuscate>true</obfuscate>
          <options>
            <!--# JDK目标版本1.8-->
            <option>-target 1.8</option>
            <!-- 不做收缩（删除注释、未被引用代码）-->
            <option>-dontshrink</option>
            <!-- 不做优化（变更代码实现逻辑）-->
            <option>-dontoptimize</option>
            <!--  ##对于类成员的命名的混淆采取唯一策略-->
            <option>-useuniqueclassmembernames</option>
            <!--## 混淆类名之后，对使用Class.forName('className')之类的地方进行相应替代-->
            <option>-adaptclassstrings</option>
            <!--混淆时不生成大小写混合的类名，默认是可以大小写混合-->
            <option>-dontusemixedcaseclassnames</option>
            <!--忽略警告-->
            <option>-ignorewarnings</option>
            <!-- This option will replace all strings in reflections method invocations with new class names.
                 For example, invokes Class.forName('className')-->
            <!-- <option>-adaptclassstrings</option> -->
            <!-- This option will save all original annotations and etc. Otherwise all we be removed from files.-->
            <!-- 不混淆所有特殊的类-->
            <option>-keepattributes Exceptions,InnerClasses,Signature,Deprecated,
              SourceFile,LineNumberTable,*Annotation*,EnclosingMethod
            </option>
            <!-- This option will save all original names in interfaces (without obfuscate).-->
            <!--
                         <option>-keepnames interface **</option>
            -->
            <!-- This option will save all original methods parameters in files defined in -keep sections,
                 otherwise all parameter names will be obfuscate.-->

            <!--保留参数名字-->
            <option>-keepparameternames</option>
            <!--保留主程序入口-->
            <!--
                        <option>-keep @org.springframework.boot.autoconfigure.SpringBootApplication class * {*;}</option>
            -->
            <!-- <option>-libraryjars **</option> -->
            <!-- This option will save all original class files (without obfuscate) but obfuscate all in domain package.-->
            <!--<option>-keep class !com.slm.proguard.example.spring.boot.domain.** { *; }</option>-->

<!--
            <option>-keep class !com.bmsoft.graph.** { *; }</option>
-->
            <option>-keep class com.bmsoft.graph.config.** { *; }</option>
            <option>-keep class com.bmsoft.graph.LinkGraphApplication { *; }</option>
            <option>-keep class com.bmsoft.graph.mapper.** { *; }</option>
<!--
            <option>-keep class com.bmsoft.graph.auth.filter.** { *; }</option>
-->
            <option>-keep class com.bmsoft.graph.aspect.** { *; }</option>
            <option>-keep class com.bmsoft.graph.domain.** { *; }</option>
            <option>-keep class com.bmsoft.graph.controller.** { *; }</option>
<!--
            <option>-keep interface * extends * { *; }</option>
-->
             <!--##保留枚举成员及方法-->
            <option> -keepclassmembers enum * { *; }</option>
            <option>-keepclassmembers class * {
              <!-- @org.springframework.beans.factory.annotation.Autowired *; -->
              @org.springframework.beans.factory.annotation.Autowired *;
              @org.springframework.beans.factory.annotation.Value *;
              }
            </option>
          </options>
          <libs>
            <!-- Include main JAVA library required.-->
            <lib>${java.home}/lib/rt.jar</lib>
            <lib>${java.home}/lib/jce.jar</lib>
            <!-- <lib>${java.home}/lib/spring-boot-starter-web-1.4.1.RELEASE.jar</lib> -->
          </libs>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>net.sf.proguard</groupId>
            <artifactId>proguard-base</artifactId>
            <version>6.0.3</version>
          </dependency>
        </dependencies>
      </plugin>
```

这里引用了com.github.wvengen的proguard-maven-plugin插件，使用的proguard-base版本是6.0.3
这里使用java8，因为libs那里照常配置rt.jar，jce.jar，如果是java9的话，则需要换成相应的模块。

另外指定proguard的阶段为package，springboot打包在repackage阶段

```
<plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <executions>
          <execution>
            <!-- <phase>none</phase> -->
            <goals>
              <goal>repackage</goal>
            </goals>
            <configuration>
              <mainClass>com.bmsoft.graph.LinkGraphApplication</mainClass>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

## bean命名重复异常

由于proguard混淆貌似不能指定混淆的类名在basePackages下面类名混淆后唯一，不同包名经常有a.class，b.class,c.class之类重复的类名，因此spring容器初始化bean的时候会报错。

异常信息如下
```
org.springframework.beans.factory.BeanDefinitionStoreException: Failed to parse configuration class [com.example.demo.MvcDemoApplication]; nested exception is org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name 'a' for bean class [com.example.demo.c.a] conflicts with existing, non-compatible bean definition of same name and class [com.example.demo.b.a]
    at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:181) ~[spring-context-4.3.10.RELEASE.jar!/:4.3.10.RELEASE]
```

需要在更改bean 命名
```
public class LinkGraphApplication {

  public static class CustomGenerator implements BeanNameGenerator {

    @Override
    public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
      return definition.getBeanClassName();
    }
  }

  public static void main(String[] args) {
    new SpringApplicationBuilder(LinkGraphApplication.class)
        .beanNameGenerator(new CustomGenerator())
        .run(args);
  }
}
```

## 执行maven命令,打成jar包

```
clean package -DskipTests
```

## 运行jar包

nohup java -jar  jar包名.jar -&
 
### 可能出现的问题
项目中引入了swagger 插件，需要在打包的时候，不进行bean的初始化配置

```
@Configuration
@ConditionalOnProperty(prefix = "swagger", value = {"enable"}, havingValue = "true")
@EnableSwagger2
public class SwaggerConfig {
   @Bean
  public Docket createRestApi() {
    ParameterBuilder tokenParam = new ParameterBuilder();
    List<Parameter> params = new ArrayList<Parameter>();
    tokenParam.name("Authorization").description("令牌标识").modelRef(new ModelRef("string"))
        .parameterType("header").required(false).defaultValue(TOKEN).build();
    params.add(tokenParam.build());
    return new Docket(DocumentationType.SWAGGER_2)
        .select()
        .apis(RequestHandlerSelectors.basePackage("com.bmsoft.graph.controller"))
        .paths(PathSelectors.any())
        .build()
        .globalOperationParameters(params)
        .apiInfo(apiInfo())
        .pathMapping("");
        .new ResponseMessageBuilder().code(403).message("Forbidden!!").build()));
  }


  private ApiInfo apiInfo() {
    return new ApiInfoBuilder()
        .title("图服务的 API")
        .description("图服务")
        .contact("图服务")
        .version("1.0.0")
        .build();
  }
}
```

## 反编译

Windows下，直接解压jar 包，可以查看jar包内的文件形式，Mac 可以使用 unarchiver 进行解压jar包，要查看字节码文件的话，可以直接把class文件放到idea进行查看

## 注意

-keep class 类/包.**  表示保留类名

-keepclassmembers class 类/包.**{ *;} 表示保留类下边的所有变量，均不混淆


参考链接 
https://blog.csdn.net/songluyi/article/details/79554928

https://www.jianshu.com/p/8f6c72def69d