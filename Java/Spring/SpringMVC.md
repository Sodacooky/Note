# 内部原理
工作流程如图:  （非Restful）
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
    *   仍然需要指定ViewResolver，除非不使用模板引擎

# 使用
## 非Restful
#### 控制器类
*   控制器类需要实现接口org.springframework.web.servlet.mvc.Controller，使用注解开发还需要Bean声明注解  
*   返回值的ModelAndView将Model（给View的数据）和View（指定跳转的视图）返回给DispatcherServlet（经Adapter）  
*   ModelAndView通过.addObject()等方法添加参数（类似基本Servlet向请求或响应中setAttribute），通过.setViewName()指定跳转的视图
