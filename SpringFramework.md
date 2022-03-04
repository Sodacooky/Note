# IOC
*   XML方式
    *   定义Bean
		<bean id="指定名称" class="类名全路径com.xxx.xx" scope="singl/proto"/>
	*   注入参数
		*   构造函数注入
            如果不使用此种方法，bean必须有无参数构造函数  
            ```xml
            <constructor-arg name="参数名" value="值"/>
            ```
            若要注入（引用类型）类，则  
            ```xml
            <constructor-arg name="参数名" ref="其他bean的id"/>  
            ```
            value/ref的区别下同  
		*   setter注入
            ```xml
			<property name="属性名（setter名转换，遵守Bean规则）" value=""/>
            ```
		*   注入数组或容器
			在上述二者内部创建子级，如  
            ```xml
			<property name="">
				<list或者array或者map或者set>
					<value>值</value>
					<ref bean=""/>
				</list,array,map,set>
			</property>
            ```
			也可以使用util，将util注册为bean后使用ref或外部bean注入
            ```xml
			<util:list id=""><value/><ref/></util:list>
            ```
	*   自动注入、装配
		```<bean>```中的autowire=""  
		byType通过setter的类型自动寻找对应类型的bean装配  
		byName通过setter的名称自动匹配对应的bean的id  
	*   "切面"
		```<bean>```可以设置init-method和destroy-method，后者目前发现只在单例模式生效  
		通过实现BeanPostProcessor接口并注册为Bean，可以为文件内的Bean创建后处理器  
	*   工厂Bean
		通过实现FactoryBean并实现方法，getBean时通过该工厂Bean获得产品（不同类型）  
	*   执行顺序
		类的构造函数 -> setter注入 -> (aware ->) BeanPostProcessor.Before -> init-method
		-> BeanPostProcessor.After -> |生命周期结束| -> destroy-method  
	*   配置文件引入
        ```xml
		<context:property-placeholder location="目录"/>
        ```
		可用classpath:前缀指定类目录  
		使用${属性名}替换(即EL表达式)  
*   注解方式
	*   启用方式
		*   配置文件中
            ```xml
            <context:component-scan base-package="要搜索注解的包目录根部"/>
            ```
			* use-default-filters指是否扫描所有
			* 可创建子级
                ```xml
                <context:include-filter type="anntation" expression="">
                ```
                或exclude  
				其中expression为要包含或排除的注解类型
				org.springframework.stereotype.Component/Controller/….
		*   定义配置类  
            定义空类，给予注解@Configuration并给予@ComponentScan(basePackages={"包目录根",,,})  
			将ClassPathXmlApplicationContext替换为AnnotationConfigApplicationContext(该配置空类.class)
	*   定义Bean
		*   在类前加上注解@Component, @Service, @Controller, @Repository  
		四者功能相同，日后可能不同，并且用于约定区分不同类型的Bean并且用于过滤器过滤
		*   使用@Scope设定作用域（小写字符串
    *   自动装配
		*   在setter方法前添加注解@Autowired即按setter参数类型自动装配
		*   当有多个匹配项（参数类型为接口有多个实现）时可用@Qualifier指定要用的Bean的ID
		*   简单类型可用@Value注入
	*   配置文件引入
		@PropertySource("classpath:也可以不带前缀")

# AOP
*   基本概念
	*   连接点：类中可以被增强（代理）的方法
	*   切入点：实际上被增强的连接点
    *   通知：增强部分的代码
		*   前置通知、后置通知：方法执行前或后的增强
		*   环绕通知：前和后都增强
		*   异常通知：出现异常时处理
		*   最终通知：无论如何都会最后执行的（try后的finally
	*   切面：把通知应用到切入点的过程（织入、执行
*   基于XML
*   基于注解
	*   启用方式
		*   配置文件  
			开启扫描
            ```xml
            <context:component-scan base-package="要搜索注解的包目录根部"/>
            ```
            自动代理 
            ```xml
            <aop:aspectj-autoproxy/>
            ```
        *   