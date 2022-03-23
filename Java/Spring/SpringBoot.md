# 原理

## 依赖
*   Maven包含了SpringBoot的各种依赖  
    用户只需要引入启动器([spring-boot-starter-xxx](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters))，版本被依赖管理（父项目）  
    当然，不能这样引入的、偏僻的包也可以用老方法引入

## 自动配置 @SpringBootApplication
#### @SpringBootApplication
该注解将类注册为Bean并标注为SpringBoot应用（启动类）  
#### 其内main方法内的SpringApplication.run(启动类.class,args)将该应用启动  
1.  推断程序是否为Web项目，设置监听器，设置main方法定义类（设置主类）
2.  获取配置参数(核心配置文件、banner.txt等)
3.  配置上下文、装配Bean等
#### @SpringBootConfiguration  
SpringBoot配置，是@Configuration修饰的@Component的组件
#### @EnableAutoConfiguration  
自动配置，将启动类下的所有包的东西导入    
*   @AutoConfigurationPackage  
    通过@Import({Registrar.class})，  
    将@SpringBootApplication注解的主程序类所在包及其子包下的“组件”扫描注册到Spring容器。
*   @Import({AutoConfigurationImportSelector.class})  
    查找classpath中所有jar包的META-INF/spring.factories进行加载，选择性地将配置信息交给Spring容器进行创建。
#### 总而言之
@SpringBootApplication注解让Spring在启动时自动将maven导入了的starter（各种模块）加载，也把启动类目录之下的所有东西加载。

---

# 配置

## 创建工程
*   官网Spring Initilizer
*   IDEA内置Spring Initilizer
*   需要勾选组件SpringWeb（内嵌Tomcat等）  
    通常还需要commons-io与commons-fileupload包  

## application.properties/yml
*SpringBoot的核心配置文件*
#### 常用配置
*   server.port=容器端口号  
    或yml
    ```yml
    server:
        port: 端口号
    ```
#### YAML语法 *一定记得空格*
*   普通KV
    ```yml
    key: value
    ```
*   对象
    ```yml
    object:
        key: false
        kkey: 2022/01/01
    ```
    ```yml
    object: {key: 1, kkey: 14514}
    ```
*   数组
    ```yml
    #行内
    object: [阿黄, 阿龙]
    #多行
    object:
      - value
      - vvalue
    ```
#### YAML能直接给实体类赋值（注入）
*类似于XML配置文件注入或@Value*
1.  在配置文件中按上述类的语法写好
2.  要注入的类加上注解@ConfiguratrionProperties(prefix="配置文件中的对象名")
*   ~~加载指定配置文件~~  
    @PropertySource("classpath:xxx")  
    @Value("${key}")

## 更换banner
resources目录下的banner.txt

---