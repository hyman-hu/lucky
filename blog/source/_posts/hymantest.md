title: hymantest
author: John Doe
tags:
  - test
categories:
  - test
date: 2018-06-11 13:25:00
---
##springboot学习笔记01
######致敬程序员DD大神
- 学习自：<a href="http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/">Spring Boot基础教程<a/>
####Spring Boot属性配置文件详解
- 自定义属性与加载
	- 通过@Value("${属性名}")注解来加载对应的配置属性
	- `application.properties`中配置：com.hyman.blog.name=hyman
	- Bean里面引用：`@Value("${com.hyman.blog.name}")
    private String name;`
	- 参数间的引用：`com.hyman.blog.desc=${com.hyman.blog.name}正在努力写《${com.hyman.blog.title}》`
	- 使用随机数
	- ![](https://i.imgur.com/QMgJtaf.gif)
- 通过命令设置属性
	- `java -jar xxx.jar --server.port=8888`等价于`在application.properties中添加属性server.port=8888` 
	- 通过命令行来修改属性值固然提供了不错的便利性，但是通过命令行就能更改应用运行的参数，那岂不是很不安全？
		- 是的，所以Spring Boot也贴心的提供了屏蔽命令行访问属性的设置，只需要这句设置就能屏蔽：`SpringApplication.setAddCommandLineProperties(false)。`
- 多环境配置
	- 在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：
		- `application-dev.properties`：开发环境
		- `application-test.properties`：测试环境
		- `application-prod.properties`：生产环境 
	- 至于哪个具体的配置文件会被加载，需要在application.properties文件中通过spring.profiles.active属性来设置，其值对应{profile}值。
		- 如：spring.profiles.active=test就会加载application-test.properties配置文件内容
	- 可以如下总结多环境的配置思路：
		- application.properties中配置通用内容，并设置
		- spring.profiles.active=dev，以开发环境为默认配置
		- application-{profile}.properties中配置各个环境不同的内容
		- 通过命令行方式去激活不同环境的配置  

----------
####配置绑定2.0全解析
- 配置文件绑定
	- 简单类型：
		- 下面的4种配置方式都是等价的：
			- properties格式：
				- `spring.jpa.databaseplatform=mysql`
				- spring.jpa.database-platform=mysql
				- spring.jpa.databasePlatform=mysql
				- spring.JPA.database_platform=mysql

			- yaml格式：
				- `spring:jpa:databaseplatform: mysql` 
				- database-platform: mysql
    			- databasePlatform: mysql
    			- database_platform: mysql
	-  属性的读取
		- 如果我们要在Spring应用程序的environment中读取属性的时候，每个属性的唯一名称符合如下规则：
			- 通过.分离各个元素
			- 最后一个.将前缀与属性名称分开
			- 必须是字母（a-z）和数字(0-9)
			- 必须是小写字母
			- 用连字符-来分隔单词
			- 唯一允许的其他字符是[和]，用于List的索引
			- 不能以数字开头
		- `this.environment.containsProperty("spring.jpa.database-platform")`
		- ***注意***：使用@Value获取配置内容的时候也需要这样的特点

----------
####新增时间ApplicationStartedEvent
- 在2.0版本所有的事件按执行的先后顺序如下
	- ApplicationStartingEvent
	- ApplicationEnvironmentPreparedEvent
	- ApplicationPreparedEvent
	- ApplicationStartedEvent <= 新增的事件
	- ApplicationReadyEvent
	- ApplicationFailedEvent
- `public class ApplicationStartedEventListener implements ApplicationListener<ApplicationStartedEvent>`
	- 第一步：我们可以编写ApplicationPreparedEvent、ApplicationStartedEvent以及ApplicationReadyEvent三个事件的监听器，然后在这三个事件触发的时候打印一些日志来观察它们各自的切入点，比如：
	- 第二步：在/src/main/resources/目录下新建：META-INF/spring.factories配置文件，通过配置org.springframework.context.ApplicationListener来加载上面我们编写的监听器。
- 官方文档对ApplicationStartedEvent和ApplicationReadyEvent的解释： 
	- An ApplicationStartedEvent is sent after the context has been refreshed but before any application and command-line runners have been called.An ApplicationReadyEvent is sent after any application and command-line runners have been called. It indicates that the application is ready to service requests


----------

####Spring Boot构建RESTful API与单元测试
- @Controller：修饰class，用来创建处理http请求的对象
- @RestController：Spring4之后加入的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接用@RestController替代@Controller就不需要再配置@ResponseBody，默认返回json格式。
- @RequestMapping：配置url映射
```
 @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @ModelAttribute User user) {
        // 处理"/users/{id}"的PUT请求，用来更新User信息
```

----------
####Spring Boot开发Web应用
- 静态资源访问
	- 在我们开发Web应用的时候，需要引用大量的js、css、图片等静态资源。
- 默认配置
	- Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：
		- /static
		- /public
		- /resources
		- /META-INF/resources 
- 渲染Web页面
	- 在之前的示例中，我们都是通过@RestController来处理请求，所以返回的内容为json对象。那么如果需要渲染html页面的时候，我们一般用@Controller
- 模板引擎  
	- 在动态HTML实现上Spring Boot依然可以完美胜任，并且提供了多种模板引擎的默认配置支持，所以在推荐的模板引擎下，我们可以很快的上手开发动态网站。
	- Spring Boot提供了默认配置的模板引擎主要有以下几种：
		- Thymeleaf
		- FreeMarker
		- Velocity
		- Groovy
		- Mustache
	-  Spring Boot建议使用这些模板引擎，避免使用JSP，若一定要使用JSP将无法实现Spring Boot的多种特性。
	-  如硬要使用JSP页面，也是可以的。需要参考 <a href="https://github.com/spring-projects/spring-boot/tree/v1.3.2.RELEASE/spring-boot-samples/spring-boot-sample-web-jsp">SpringBoot对JSP的支持</a>
- Thymeleaf模板引擎
	- Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。它是一个开源的Java库，基于Apache License 2.0许可，由Daniel Fernández创建，该作者还是Java加密库Jasypt的作者。
	- Thymeleaf提供了一个用于整合Spring MVC的可选模块，在应用开发中，你可以使用Thymeleaf来完全代替JSP或其他模板引擎，如Velocity、FreeMarker等。
	- Thymeleaf的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。
	- 你可以使用它创建经过验证的XML与HTML模板。相对于编写逻辑或代码，开发者只需将标签属性添加到模板中即可。
	- eg:`th th:text="#{msgs.headers.name}">Name</td>` 
	- 在Spring Boot中使用Thymeleaf，只需要引入下面依赖，并在默认的模板路径src/main/resources/templates下编写模板文件即可完成。
- Thymeleaf的默认参数配置
	- 如有需要修改默认配置的时候，只需复制下面要修改的属性到application.properties中，并修改成需要的值，如修改模板文件的扩展名，修改默认的模板路径等。
	- ![](https://i.imgur.com/0iCVcRs.gif) 
- FreeMarker模板引擎
	- `FreeMarker模板引擎<h1>${host}</h1>`
- Velocity模板引擎
	- `Velocity模板<h1>${host}</h1>`


----------
####Spring Boot中使用Swagger2构建强大的RESTful API文档
- Swagger2也提供了强大的页面测试功能来调试每个RESTful API。
- 使用Swagger我们可以将后端写的接口，可视化的给前端测试调用
- 步骤：
	- 1.通过@Configuration注解，让Spring来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2。
	- 2.通过createRestApi函数创建Docket的Bean之后，apiInfo()用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。select()函数返回一个ApiSelectorBuilder实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被@ApiIgnore指定的请求）。
	- 3.添加文档内容：
		- 我们通过@ApiOperation注解来给API增加说明、通过@ApiImplicitParams、@ApiImplicitParam注解来给参数增加说明。
		- `@ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")`
		- `@ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")`
	- 4.启动Spring Boot程序，访问：http://localhost:8080/swagger-ui.html
- ***注意：***  Swagger任需继续深入学习


----------

####Spring Boot中Web应用的统一异常处理
- Spring Boot提供了一个默认的映射：/error，当处理中抛出异常之后，会转到该请求中处理，并且该请求有一个全局的错误页面用来展示异常内容。
- 统一异常处理步骤：
	- 1.创建全局异常处理类：通过使用***@ControllerAdvice***定义统一的异常处理类，而不是在每个Controller中逐个定义。
	- 2.***@ExceptionHandler***用来定义函数针对的异常类型，最后将Exception对象和请求URL映射到error.html中
	- 3.实现error.html页面展示：在templates目录下创建error.html，将请求的URL和Exception对象的message输出。
- 有多种不同的Exception。然后在@ControllerAdvice类中，根据抛出的具体Exception类型匹配@ExceptionHandler中配置的异常类型来匹配错误映射和处理。 


----------
####Spring Boot使用JDBCTemplate访问数据库
- 使用JdbcTemplate操作数据库：
	- Spring的JdbcTemplate是自动配置的，你可以直接使用@Autowired来注入到你自己的bean中来使用。
		- 定义包含有插入、删除、查询的抽象接口UserService
		- 通过JdbcTemplate实现UserService中定义的数据访问操作


----------

####Spring Boot使用Spring-date-JPA
- spring.jpa.properties.hibernate.hbm2ddl.auto是hibernate的配置属性，其主要作用是：自动创建、更新、验证数据库表结构。该参数的几种配置如下：
	- create：每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。
	- create-drop：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
	- update：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
	- validate：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。


----------
####Spring Boot多数据源配置与使用
- 多数据源配置
	- 创建一个Spring配置类，定义两个DataSource用来读取application.properties中的不同配置。
	- 如下例子中，主数据源配置为spring.datasource.primary开头的配置，第二数据源配置为spring.datasource.secondary开头的配置。
	- ![](https://i.imgur.com/UEGXbfY.gif)
	- ![](https://i.imgur.com/crN3mKY.gif)
- JdbcTemplate支持
	- 创建JdbcTemplate的时候分别注入名为primaryDataSource和secondaryDataSource的数据源来区分不同的JdbcTemplate。
	- ![](https://i.imgur.com/4UPyYnY.gif) 
- JPA支持 
	- 主数据源的实体和数据访问对象位于：com.didispace.domain.p，
	- 次数据源的实体和数据访问接口位于：com.didispace.domain.s。
	-  ![](https://i.imgur.com/RQgFo04.gif)
	-  ![](https://i.imgur.com/g1e2WuX.gif)