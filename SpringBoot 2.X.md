# SpringBoot 2.X

## 1、第一个spingboot程序

### 1.1 springboot可以使用springInitialnizer来启动，官网创建或使用idea创建。

![image-20201109183652888](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109183652888.png)

### 1.2 原理初探

**POM.xml**

- spring-boot-dependencies: 核心依赖存在于父工程中，版本号管理也存在于父工程中，只需要引入依赖即可，不需要指定版本。

**启动器**

- 启动器即spring boot的启动场景，引入某一starter时，自动引入相关环境。

  ```java
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

  需要时找到对应的环境即可。

**主程序**

- @SpringBootApplication标注这是springboot应用。

```java
@SpringBootApplication
public class SpringboottestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringboottestApplication.class, args);
    }
}
```

- ​	注解，springbootapplication注解层级如下

  ```
  @SpringBootConfiguration       springboot的配置注解
  	@Configuration             spring的配置类
  		@Component			   spring的一个组件
  		
  @EnableAutoConfiguration			自动配置包
  	@AutoConfigurationPackage       自动配置包
  		@Import({Registrar.class})  自动配置包的注册
  	@Import({AutoConfigurationImportSelector.class})    自动配置的导入选择
  
  ```

  AutoConfigurationImportSelector中获取已经配置了的配置

  ```java
  protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
      List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
      Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
      return configurations;
  }
  ```

加载文件如下。

​	![image-20201109190913869](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109190913869.png)

spring.factories文件中配置了autoconfiguration的内容。

``` java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
```

``` java
Properties properties = PropertiesLoaderUtils.loadProperties(resource);
```

将所有资源封装在静态类中。

结论：springboot所有要自动配置都是在启动时扫描并加载。spring.factories所有的配置类都在这里，但不一定生效，需要通过@ConditionOn等注解判断指定资源是否存在，如果存在，再加载自动装配。

![image-20201109201227497](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109201227497.png)

**SpringApplication.run(args).**

该过程分两部分，一部分是SpringApplication的初始化，一部分是run方法。

主要做了以下事：

1、判断应用是普通应用还是WEB应用；

2、查找并加载所有可用初始化器，设置到initializers中；

3、找出所有的应用程序监听器，设置到listeners中；

4、推断并设置mian方法的定义类，找到运行的主类。

### 1.3 配置文件

- yaml（yml) 基本语法：

  属性名： 值。

  对象：

  student: {age: 12,name: haha}

  数组：

  pets:

  ​	-dog

  ​	-cat

  pets: [cat,dog,pig] 行内写法。

- 给实体类赋值

  ![image-20201109205443122](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109205443122.png)

  - 使用@ConfigurationProperties(prefix="person")加在实体上即可自动注入。

  ![image-20201109210501131](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109210501131.png)

  ​	使用YML时还可以注入函数。

  ​	使用构造器注入，需要加上无参及有参构造器。

  - properties也可以注入：

    使用@PropertiesSources导入。

  对比：

  ![image-20201109211020122](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109211020122.png)

  数据校验：

  ``` java
  @Data
  @ConfigurationProperties(prefix = "dog")
  @Validated
  public class Dog {
      @Email
      private String name;
      private Integer age;
  }
  ```

  当name不符合要求时会报错。

  JSR303校验：包含大量注解形式的格式验证。

  ![image-20201109212254665](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109212254665.png)

- 配置文件生效优先级

  1、项目config目录下

  2、项目路径下

  3、resources/config目录下

  4、resources目录下

- 配置文件切换

  properties ： 

  建立application-dev.properties\application-pro.properties等文件。

  使用**sping.profiles.active=dev**进行切换即可。

  yml:

  可以在同一个文件中，使用---分割。

  ```java
  server:
    port: 8080
  spring:
    profiles:
      active: test
  ---
  server:
    port: 8081
  spring:
    profiles: dev
  ---
  server:
    port: 8082
  spring:
    profiles: test
  ```

  哪些内容可以配置

  ![image-20201109215200174](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109215200174.png)

![image-20201109220321160](C:\Users\renlo\AppData\Roaming\Typora\typora-user-images\image-20201109220321160.png)

debug: true可以看到更多输出。

## 2、WEB开发

### 2.1 静态资源位置

- ​	webjars: 可以以jar包的形式导入静态资源文件。

- ```
  {"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"}   /**
  ```

  多个路径均可以。优先级 resources>static>public

  一旦在配置文件中进行了静态资源自定义，以上文件夹就失效了。

  spring.mvc.....location。

### 2.2 首页定制

​	一般将首页放在static下。

​	首页图标：

​		Spring Boot在配置的静态内容位置和类路径的根（按此顺序）中查找`favicon.ico`。如果存在这样的文件，它将自动用作应用程序的favicon。

### 2.3 模板引擎（thymeleaf）

