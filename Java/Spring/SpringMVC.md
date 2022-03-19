# 内部原理
工作流程如图:  
1.  当接收到用户的请求时，请求进入到DispatcherServlet进行处理
2.  DispatcherServlet根据请求的URL从HandlerMapping获取映射的处理请求的Bean，也就是Controller
3.  通过HandlerAdapter调用Controller（应该是适配处理了数据转换），Controller内部处理业务
4.  Controller返回ModelAndView（Model相当于请求响应参数，View是指定要跳转的视图）
5.  ViewResolver根据返回的指定的View（可能没有）进行跳转
![](./Picture/SpringMVC_Workflow.jfif)  

# 配置
## XML配置
#### 映射Servlet
在项目web.xml中将DispatcherServlet注册并映射到根
```xml
<servlet>
        <servlet-name>YOUR_NAME</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:你的SpringBean配置文件</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>YOUR_NAME</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
#### 配置组件
在SpringBean配置文件中将HandlerMapper、HandlerAdapter和ViewResolver选择并注册为Bean
```xml
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <property name="prefix" value="指定视图名时(希望省略)的前缀，通常以斜杠结尾"/>
        <property name="suffix" value="后缀，通常是.jsp"/>
    </bean>
```
#### 映射
在SpringBean配置文件中将Controller类注册为Bean，id为映射，如：
```xml
<bean id="/map" class="soda.main.MyController"/>
```
则通过xxxx.xx:xx/map访问该控制器
## 注解配置(无SpringBoot的带XML配置方式)
*   web.xml中的内容保持不变，因为没有办法在其他的地方指定让请求先走DispatcherServlet
*   上面指定的SpringBean配置文件中：
    *   开启Bean注解扫描
    ```xml
    <context:component-scan base-package=""/>
    ```
    *   让SpringMVC能够通过如@ResquestMapping等注解映射
    ```xml
    <mvc:annotation-driven/>
    ```
    *   让SpringMVC忽略不处理静态资源，如.css  
        (还是说只是启用上面XML配置那样的默认Mapper？)
    ```xml
    <mvc:default-servlet-handler/>
    ```
    *   如果不使用模板引擎或手动指定跳转的试图路径(见下方)，可以不配置ViewResolver

# 使用
*需要注意的是，Spring会根据你方法返回值或参数的不同自动给你适配不同的行为或填充数据*
## 控制器类
#### 定义控制器类
*   类加上注解@RestController
    *   相当于@Controller + @ResponseBody
    *   @ResponseBody表示这个方法返回的字符串直接写到响应内容
*  **过时的** 实现org.springframework.web.servlet.mvc.Controller方法
    *   控制器类需要实现接口org.springframework.web.servlet.mvc.Controller，还需要Bean声明注解  
    *   一个类只能实现一个方法
*   新版本能够不实现该接口
    *   对类使用注解@Controller等（Bean）
    *   默认方法不加@ResponseBody那么返回的字符串为View名称
#### 操作
*   数据处理
    *   返回值的ModelAndView将Model（给View的数据）和View（指定跳转的视图）返回给DispatcherServlet（经Adapter） 
    *   ModelAndView通过.addObject()等方法添加参数（类似基本Servlet向请求或响应中setAttribute），通过.setViewName()指定跳转的视图
    *   如果需要Model则在方法参数列表声明(org.springframework.ui.)Model变量  
        返回字符串的方法会将Model转发
    *   参数列表的ModelMap  
        与Model或ModelAndView相比就是个HashMap和简化出常用方法的区别  
*   转发与重定向  
    若没有在XML或配置类中配置ViewResolver，实际上能够在return视图的名称时直接指定，也可以使用前缀指定是转发还是重定向：  
    ```java
    //转发
    return "/your-path/page.jsp";
    return "forward:/sbv/sss.jsp";
    //重定向
    return "redirect:/aaaa.jsp";
    ```
    *   转发是指带着请求的参数到下一个地方去
    *   重定向是客户端浏览器自己进行跳转的行为（相当于用户自己重新访问新页面  
        重定向无法访问WEB-INF
*   乱码解决
    *   过滤器（解决接收请求的乱码）  
        直接在web.xml配置使用SpringMVC的过滤器
        ```xml
        <filter>
            <filter-name>YOURNAME</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>utf-8</param-value>
            </init-param> 
            //强制编码可能需要       
            <init-param>  
                <param-name>forceEncoding</param-name>  
                <param-value>true</param-value>  
            </init-param> 
        </filter>
        <filter-mapping>
            <filter-name>YOURNAME</filter-name>
            <url-pattern>/</url-pattern>
            //考虑/*让其他资源也支持
        </filter-mapping>
        ```
    *   MessageConverter（解决@Response返回字符串乱码）
        *   直接在映射注解配置参数produces  
            如produces="application/json;charset=UTF-8"  
            事实证明，只设置为Json则不需要后面的编码charset，因为Json不会被SpringMVC内部转换
        *   在```<mvc:annotation-driven>```中配置converter  
            ```xml
            <mvc:annotation-driven>
                <mvc:message-converters register-defaults="true">
                    <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                        <property name="defaultCharset" value="UTF-8"/>
                        <property name="writeAcceptCharset" value="false"/>
                    </bean>
                    <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                        <property name="objectMapper">
                            <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                                <property name="failOnEmptyBeans" value="false"/>
                            </bean>
                        </property>
                    </bean>
                </mvc:message-converters>
            </mvc:annotation-driven>
            <!--这样也行-->
            <mvc:annotation-driven>
                <mvc:message-converters>
                    <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                        <property name="defaultCharset" value="utf-8"/>
                        <property name="writeAcceptCharset" value="false"/>
                    </bean>
                </mvc:message-converters>
            </mvc:annotation-driven>
            ```
*   请求和响应数据(HttpServletXxx)  
    当方法参数有HttpServletRequest或HttpServletResponse时会自动注入，像上面Model  
    当然也可以使用@Autowired注解成员变量，虽然@Controller是单例但会自动识别当前请求和响应
#### 非Restful(.../method?param=value&...)
*   获取参数
    ```java
    @RequestMapping("/echo")
    public ModelAndView handleRequest(String msg) throws Exception {}
    ```
    中，当使用".../echo?msg=xxx"访问时能正确获得msg参数
*   上一种情况中，当url中参数名和方法参数表变量名不一致时，可在方法参数前加@RequesParam("url中的名称")来指定
*   获取类对象  
    *客户端发送的请求是一个个零散的值，共同组成一个类时*  
    要接收的类为POJO类(简单属性+GetSetter)  
    *   什么都不写，方法参数名直接写类（自动匹配类的成员名称  
    *   
#### Restful(.../method/value/value2/...)
*   方法可在参数列表使用@PathVariable获取url参数
    *   @RequestMapping中需要注明参数名像@RequstMapping("/map/{varA}/{varB}")  
        然后对参数表中的参数注解如@PathVariable("varA") String bbb
    *   如果RequestMapping中的参数名称和方法参数表得到相同可省略@PathVariable的参数  
    *   @RequestMapping中可以指定method为GET/PUT/POST/DELETE
    *   方法的指定也可以通过将@RequestMapping改为@GetMapping/@PostMapping等实现

## 文件上传下载
依赖于commons-io和commons-fileupload
#### 将组件注册Bean
```xml
<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="maxUploadSize" value="1024000000"/>
    <property name="maxInMemorySize" value="1024000"/>
</bean>
```
#### 参数
控制器的方法的参数为CommonsMultipartFile类型