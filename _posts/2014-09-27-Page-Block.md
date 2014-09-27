---
layout: post
title: Page Block
category: F2E
date: 2014-09-27
---
页面block划分

1. 场景分析
一个网站整体布局以及风格一般会保持一致，整个网站各个页面有很多共同的地方，例如整站导航，友情链接等等。我们会看到很多类似下面的页面：

 ![](/image/page-block.png)

如上图我们会发现整个网站中绝大多数页面都会有上面的Header，Left 和footer 这几个部分。这些相同部分无论是前端展示还是后台数据支持都几乎是一致的。如果按照传统开发方式我们开发可能会这样：

（现在有a,b,c三个页面，这三个页面都有上图中left这一部分）首先前端我们可以选择直接code复制或是独立left部分的code在各个要用的地方引用。后台根据request获取数据返回这样的操作同样也是和前端类似的处理方式。

	a页面前端：
	
	<div>header page</div>
	
	。。。。。。
	
	。。。。。。
	
	<div>left page</div>
	
	。。。。。。
	
	。。。。。。
	
	<div>footer page</div>
	
	a页面后台：
	
	Aaction{
	
	getDataForHeader(...);
	
	。。。。。。
	
	。。。。。。
	
	getDataForLeft(...);
	
	。。。。。。
	
	。。。。。。
	
	getDataForFooter(...);
	
	。。。。。。
	
	。。。。。。
	
	getDataForASelf(...);
	
	}

b,c页面的开发与上面雷同，<div>left page</div>，getDataForLeft(...)这样的代码会在每个页面前后台都会出现。这样我们就会发现left这个区块，其实它是一个内聚型的独立部分（展现+数据），它不需要依赖于所在页面的其它部分，这里我称之为block。

对于这样的场景我们为了提高开发效率提高代码复用性，block我们应该将其与其他部分独立，如将这个页面分为Header，Left，Footer，Main这么几个block，每个block负责自己的展示以及数据的获取最后输出最终的展示html代码，这样他们就完全独立于所在页面，在其他有用到header，left，footer的页面直接在调用对应的block。另外页面分为很多块，其中有些是很少发生变化有些是经常更新的，其实在每次页面request的中，其实是有很多没有必要的后台处理。当这个页面PV量达到一定量时，这样的开销会变得很可观。这时我们可以将这样很少发生变化的block最后输出的html存放在缓存中（如memcache）,从而减少后台处理开销提高网站性能。

实现技术选择：springMVC + freemarker.

2. 技术介绍

FreeMarker
What:
![](/image/FreeMarker.jpg)


Template engine：一个基于Java的模板引擎。
可应用于任何文本类型输出(更多的是用于web应用)
开源项目

Why：

使用template engine来代替JSP, 设计将变得简单，语法更简单，出错信息更易读，工具也更用户化。
template优势（也即jsp的不足）

1. 使前台脱离Java代码
2. 减少工作量，template更加简洁
3. 出错信息更加精准
 JSP常有一些让人头疼的出错信息。因为页面首先被转换成为一个servlet然后才进行编译。即使有好的 JSP 工具可以相对增加找到出错位置的可能性，那也是不是一件容易的事儿。
 由于template engine可以在template文件中直接产生而没有任何戏剧性的向代码转化，所以可以非常容易地给出适当的出错报告。
4. 不依赖编译器
JSP需要一个置放在webserver中的编译器。但这样的编译器并不能在所有平台上顺利工作(用 C++写成的) 也不利于建立纯Java 的web服务器。
5. 减少空间的浪费
JSP消耗了额外的内存和硬盘空间。对服务器上每30K的JSP文件，必须要有相应的大于30K的类文件产生。实际上使得硬盘空间加倍。考虑到JSP文件随时可以很容易地通过 ＜%@ include＞包含一个大的数据文件。同时，每一个JSP的类文件数据必须加载到服务器的内存中，这意味着服务器的内存必须永远地将整个JSP文档树保存下去。对template engines由于没有产生第二个文件，所以节省了空间。Template engines还为程序员提供对templates

在内存中进行缓存的完全控制。

6. 利于实现页面block划分，实现页面的静态化。

spring MVC
what：

Spring MVC框架是有一个MVC框架，通过实现Model-View-Controller模式来很好地将数据、业务与展现进行分离。Spring MVC的设计是围绕DispatcherServlet展开的，DispatcherServlet负责将请求派发到特定的handler。通过可配置的handler mappings、view resolution、locale以及theme resolution来处理请求并且转到对应的视图。

Why：

学习难度小于, 小于Struts2.
易于开发性能优秀的程序。
灵活性强，强大扩展性。

3. 实例

requirement

	功能：实现用户登录显示用户信息
	要求：使用freemarker替代jsp,作为view层组件
	页面划分为Header，Left，Footer，Main三个block，block间相互独立
	模板文件存放在数据库
	具备较高的扩展性
	易维护
	实现技术：springMVC，freemarker, mybatis

impletement

程序结构图：
 ![](/image/page-block-impl.png)

建立如下工程目录：

![](/image/page-block-project.png)

web.xml：(配置springMVC)

	<context-param>
	
	<param-name>contextConfigLocation</param-name>
	
	<param-value>classpath:*Context.xml</param-value>
	
	</context-param>
	
	<listener>
	
	<display-name>contextLoaderListener</display-name>
	
	<listener-class>org.springframework.web.context.ContextLoaderListener
	
	</listener-class>
	
	</listener>
	
	<listener>
	
	<listener-class>org.springframework.web.context.request.
	
	RequestContextListener
	
	</listener-class>
	
	</listener>
	
	<servlet>
	
	<servlet-name>springmvc</servlet-name>
	
	<servlet-class>org.springframework.web.servlet.DispatcherServlet
	
	</servlet-class>
	
	<init-param>
	
	<description>mvc.xml</description>
	
	<param-name>contextConfigLocation</param-name>
	
	<param-value>classpath:test-mvc.xml</param-value>
	
	</init-param>
	
	<load-on-startup>1</load-on-startup>
	
	</servlet>
	
	<servlet-mapping>
	
	<servlet-name>springmvc</servlet-name>
	
	<url-pattern>/</url-pattern>
	
	</servlet-mapping>
	
	test-mvc.xml：
	<mvc:annotation-driven/>
	
	<!-- 静态的资源文件不要spring的过滤 -->
	
	<mvc:resources mapping="/img/**" location="/img/" />
	
	<context:component-scan base-package="com.**.controller" />
	
	<bean id="viewResolver"
	
	class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewRes olver">
	
	<property name="cache" value="true" />
	
	<property name="prefix" value="" />
	
	<property name="suffix" value=".ftl" />
	
	<property name="contentType" value="text/html;charset=UTF-8" />
	
	<property name="requestContextAttribute" value="request" />
	
	<!-- 使用这些宏，必须设置FreeMarkerViewResolver的exposeMacroHelpers属性为true：
	
	在所有需要使用<@spring.bind>和<@spring.bindEscaped>的FreeMarker模板的
	
	顶部增加以下一行：<#import "/spring.ftl" as spring />
	
	这一行会在模板中导入Spring的FreeMarker宏。
	
	-->
	
	<property name="exposeSpringMacroHelpers" value="true" />
	
	<!-- 将请求和会话属性作为变量暴露给FreeMarker模板使用 -->
	
	<property name="exposeRequestAttributes" value="true" />
	
	<property name="exposeSessionAttributes" value="true" />
	
	</bean>
	
	在resource中的applicationContext.xml
	中配置freemarker和页面所需要的各个block，具体配置如下：
	<!-- freemarker 配置 -->
	
	<bean id="freemarkerConfig"
	
	class="org.springframework.web.servlet.view.freemarker.
	
	FreeMarkerConfigurer">
	
	<property name="templateLoaderPath" value="/WEB-INF/" />
	
	<property name="freemarkerSettings">
	
	<props>
	
	<prop key="template_update_delay">0</prop>
	
	<prop key="default_encoding">UTF-8</prop>
	
	<prop key="number_format">0.##########</prop>
	
	<prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>
	
	<prop key="classic_compatible">true</prop>
	
	<prop key="template_exception_handler">ignore</prop>
	
	</props>
	
	</property>
	
	</bean>
	
	<!-- 配置各个block,每个block包含对应的ftl模板文件在数据库对应的名字
	
	和freemarkerConfig对象以及其他所需属性 -->
	
	<beanid="baseBlack"abstract="true">
	
	<property name="freemarkerConfig" ref="freemarkerConfig"/>
	
	</bean>
	
	<bean id="blockManager" class="com.au.common.BlockManager"
	
	factory-method="getInstance" />
	
	<bean id="headBlock" class="com.au.cms.block.HeadBlock" parent="baseBlack">
	
	<property name="template" value="headBlock"></property>
	
	</bean>
	
	<bean id="middleBlock" class="com.au.cms.block.MiddleBlock" parent="baseBlack">
	
	<property name="template" value="middleBlock"></property>
	
	<property name="userDao" ref="userDao"/>
	
	</bean>
	
	<bean id="footBlock" class="com.au.cms.block.FootBlock" parent="baseBlack">
	
	<property name="template" value="footBlock"></property>
	
	</bean>

主要类实现

Controller类：

		@Autowired
		
		private BlockManager blockManager;
		
		@RequestMapping(value = "welcome", method=RequestMethod.GET)
		
		public ModelAndView getFirstPage(HttpServletRequest request) {
		
		ModelAndView mvc = new ModelAndView("layout/welcome");
		
		return mvc;
		
		}

BlockManager实现一个自定函数用于调用对应的Block并包含show方法如下：

	publicclass BlockManager implements TemplateMethodModel {
	
	@SuppressWarnings("rawtypes")
	
	@Override
	
	public Object exec(List args) throws TemplateModelException {
	
	String blockId = args.get(0).toString();
	
	ApplicationContext context = ApplicationContextUtil.getApplicationContext();
	
	HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
	
	.getRequest();
	
	if (context.containsBean(blockId)) {
	
	Block block = (Block) context.getBean(blockId);
	
	return block.execute(request);
	
	}
	
	return"";
	
	}
	
	}
	
	AbstractBlock（impletement Block 接口）：
	
	protected String template;
	
	protected FreeMarkerConfigurationFactory freemarkerConfig;
	
	publicvoid setTemplate(String template) {
	
	this.template = template;
	
	}
	
	publicvoid setFreemarkerConfig(FreeMarkerConfigurationFactory freemarkerConfig) {
	
	this.freemarkerConfig = freemarkerConfig;
	
	}
	
	abstractprotected Map<String, Object> preExecute(HttpServletRequest request);
	
	@Override
	
	public String execute(HttpServletRequest request) {
	
	Map<String, Object> map = preExecute(request);
	
	String templateByUser = (String)map.get("BLOCK_TEMPLATE_FILE");
	
	String content = "";
	
	if (templateByUser != null && !templateByUser.trim().equals("")) {
	
	content = TemplateUtil.process(map, freemarkerConfig, templateByUser);
	
	}
	
	if (templateByUser != null && templateByUser.trim().equals("")) {
	
	return"";
	
	}
	
	if (templateByUser == null) {
	
	content = TemplateUtil.process(map, freemarkerConfig, template);
	
	}
	
	return postExecute(content);
	
	}
	
	protected String postExecute(String content) {
	
	return content;
	
	}
	
	MiddleBlock（继承AbstractBlock，实现preExcute方法，获取数据）：
	
	@Override
	
	protected Map<String, Object> preExecute(HttpServletRequest request) {
	
	String name = request.getParameter("name");
	
	User user = userDao.getUserByName(name);
	
	Map<String, Object> map = new HashMap<String, Object>();
	
	map.put("user", user);
	
	return map;
	
	}
	
	TemplateDataMerge（从数据库取出模板文件merge数据）
	
	Configuration conf = ((FreeMarkerConfigurer) freemarkerConfig).getConfiguration();
	
	FtlSourceDao ftlSourceDao = (FtlSourceDao) ApplicationContextUtil.getApplicationContext().getBean("ftlSourceDao");
	
	FtlSource ftl = ftlSourceDao.getFtlSourceByName(ftlName);
	
	Reader reader = new StringReader(ftl.getFtlSource());
	
	Template template = new Template(ftlName, reader, conf, "UTF-8");
	
	writer = new StringWriter();
	
	template.process(map, writer);
	
	String rs = writer.toString();

至此程序的主要逻辑已经清楚，其他代码如model,service,dao等就不一一附上。如需要可联系我们

4.测试
访问项目结果如下

 ![](/image/page-block-res.png)

完。。。



