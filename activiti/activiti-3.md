# Activiti-3

## 配置支持注解

在 Spring 的配置文件 applicationContex.xml 文件中添加组件扫描、支持注解、事务的注解驱动

```markdown
<!-- 组件扫描 -->

<context:component-scan base-package="activitiex"/>

<!-- 引入spring提供的注解解析器 -->

<context:annotation-config/>

<!-- 支持事务注解 -->

<tx:annotation-driven transaction-manager="transactionManager"/>
```

Spring 的框架中提供了与@Component 注解等效的三个注解:

- @Repository 用于对 DAO 实现类进行标注
- @Service 用于对 Service 实现类进行标注
- @Controller 用于对 Controller 实现类进行标注

Bean 的属性注入

对象属性:

- @Autowired: 自动装配，默认使用类型注入.
- @Qualifier\("userDao"\) --- 按名称进行注入.

```java
@Autowired
@Qualifier("userDao")
private UserDao userDao;

//等价于
@Resource(name="userDao")
private UserDao userDao;
```

需要注入 SessionFactory 类

```java
@Repository
public class TemplateDaoImpl extends HibernateDaoSupport implements ITemplateDao {
	@Resource
	public void setSF(SessionFactory sessionFactory) {
		super.setSessionFactory(sessionFactory);
	}
```

Action 类注解 prototype

```java
@Controller
@Scope("prototype")
public class TemplateAction extends ActionSupport implements ModelDriven<Template>{}
```

## 配置环境

spring-3.2.0，Struts2-2.3.7，hibernate3，activiti5.13，tomcat7，jdk1.6

spring3.2 版本对 jdk1.8 支持报错，tomcat9 必须要高版本 jdk，activiti 框架新版本建的表低版本不能用...

eclipse 使用 javax.servlet-api 时需要手动引用 tomcat 下 lib 中对应的 jar 包。

## 配置文件

spring 配置文件 ApplicationContenxt.xml

```markdown
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/context 
						http://www.springframework.org/schema/context/spring-context-2.5.xsd
						http://www.springframework.org/schema/tx 
						http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

<!-- 加载jdbc属性文件 -->

<context:property-placeholder location="classpath:jdbc.properties"/>

<!-- 配置数据源 -->
<bean id="ds" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="**"/>
    <property name="user" value="**"/>
    <property name="password" value="****"/>
</bean>

<!-- 配置本地会话工厂bean，用于创建一个SessionFactory对象 -->
<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource" ref="ds"/>
    <!-- 注入hibernate相关属性值 -->
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
            <prop key="hibernate.hbm2ddl.auto">update</prop>
            <prop key="hibernate.show_sql">true</prop>
            <prop key="hibernate.format_sql">true</prop>
        </props>
    </property>
    <!-- 注入hbm映射文件 -->
    <property name="mappingDirectoryLocations">
        <list>
            <value>classpath:activitiex/domain</value>
        </list>
    </property>
</bean>

<!-- 事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>

<!-- 配置一个spring提供的对象，用于创建一个流程引擎配置对象 -->
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="databaseSchemaUpdate" value="true"/>
    <!-- 必须注入数据源对象 -->
    <property name="dataSource" ref="ds"/>
</bean>

<!-- 创建流程引擎对象 -->
<bean id="pe" class="org.activiti.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration"/>
</bean>

<bean id="repService" factory-bean="pe" factory-method="getRepositoryService"></bean>

<!-- 组件扫描 -->

<context:component-scan base-package="activitiex"/>

<!-- 引入spring提供的注解解析器 -->

<context:annotation-config/>

<!-- 支持事务注解 -->

<tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

struts.xml

```markdown
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
	<constant name="struts.devMode" value="true" />
    <!--配置临时文件目录-->
	<constant name="struts.multipart.saveDir" value="c:/myweb/dataTem"/>
	<!-- 修改主题样式 -->
	<constant name="struts.ui.theme" value="simple"/>
	<!-- 注册国际化文件 -->
	<constant name="struts.custom.i18n.resources" value="messages" />
	<package name="default" namespace="/" extends="struts-default">
		...
	</package>
</struts>
```

web.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	id="WebApp_ID" version="2.5">

<!-- 指定spring配置文件位置 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
<welcome-file-list>
<welcome-file>/index.jsp</welcome-file>
</welcome-file-list>
</web-app>
```

## 审批流程管理

在流程定义服务实现类中注入流程引擎对象，不需要额外的 Dao

```java
public class ProcessDefinitionServiceImpl implements IProcessDefinitionService {
	@Resource
	ProcessEngine processEngine;
}
```

### 从 zip 文件部署流程定义

- 在 Action 中提供属性接收上传时的临时文件 private File resource 提供 set/get 方法，属性名与表单页面的 input 输入框名称一致
- 在 Service 中完成部署

```java
public void deploy(File resource){
    DeploymentBuilder deploymentBuilder = processEngine.getRepositoryService().createDeployment();
    ZipInputStream zipInputStream = null;
    try {
        zipInputStream = new ZipInputStream(new FileInputStream(resource));
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    deploymentBuilder.addZipInputStream(zipInputStream);
    deploymentBuilder.deploy();
	}
```

### jsp 页面弹出删除确认框

```mairkdown
<a onClick="return window.confirm('确定删除当前记录？')" href="">删除</a>
```

- 删除流程定义时，注意级联删除
- 根据 key 删除流程定义时，需要先根据 key 查找到所有版本的 pd，然后根据 id 删除每一个。

### 显示流程图

显示流程图，本质是下载 png 文件，需要用到输入流。

jsp 页面 ModalDialog 弹窗显示

```markdown
<a href="javascript: window.showModalDialog('${pageContext.request.contextPath }/processDefinitionAction_showPng.action?pdId=${id}','','dialogHeight:500px')">查看流程图</a>
```

```java
public String showPng() {
    InputStream in = processDefinitionService.findPngStream(pdId);
    ActionContext.getContext().put("pngStream", in);
    return "showPng";
}
```

在 struts.xml 中配置文件下载结果

```markdown
<action name="processDefinitionAction_*" class="processDefinitionAction" method="{1}">
    <result name="showPng" type="stream">
        <param name="contentType">image/png</param>
        <param name="inputName">pngStream</param>
    </result>
</action>
```

## 表单模板管理

### 设计表单模板实体

id,name,pdKey,docFilePath

### 保存临时文件工具类

- 工具类静态方法
- 用 UUID 保证文件名的唯一不重复
- 使用 FIleUtils 工具类保存文件

```java
public class UploadFileUtils {

	public static String copy(File resource) {
		Date date = new Date();
		SimpleDateFormat sdf =new SimpleDateFormat("/yyy/MM/ddd/");
		String dateStr = sdf.format(date);

		File dateDir = new File("C:\\JlernDoc\\uploadFiles"+dateStr);
		if(!dateDir.exists()) {
			dateDir.mkdirs();
		}

		File target = new File(dateDir.getPath()+File.separator+UUID.randomUUID().toString());

		try {
			FileUtils.copyFile(resource,target);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return target.getPath();
	}
}
```

### 添加表单模板

- 需要准备流程定义列表数据，用于填充下拉框
- 在 Action 中提供属性接收上传时的临时文件 private File resource 提供 set/get 方法，属性名与表单页面的 input 输入框名称一致

```java
public String add() {
    String filePath = UploadFileUtils.copy(resource);
    model.setDocFilePath(filePath);
    templateService.save(model);
    return "toList";
}
```

### 删除表单模板

删除模板对象是，如果存在对应的 doc 文件，需要删除

```java
public void deleteById(Integer id) {
    Template template = templateDao.findById(id);
    String docFilePath= template.getDocFilePath();
    File file = new File(docFilePath);
    if(file.exists()) {
        file.delete();
    }
    templateDao.delete(template);
}
```

### 修改表单模板

```java
public String edit() {
    Template template = templateService.findById(model.getId());
    template.setName(model.getName());
    template.setPdKey(model.getPdKey());
    if(resource != null) {
        //删除原文件
        String docFilePath = template.getDocFilePath();
        File file = new File(docFilePath);
        if(file.exists()) {
            file.delete();
        }
        //保存新文件
        String newFilePath = UploadFileUtils.copy(resource);
        template.setDocFilePath(newFilePath);
    }
    templateService.update(template);
    return "toList";
}
```

### 下载表单模板

```java
public String download() throws FileNotFoundException {
    Template template = templateService.findById(model.getId());
    String docFilePath = template.getDocFilePath();
    InputStream docStream = new FileInputStream(new File(docFilePath));
    ActionContext.getContext().put("docStream", docStream);
    String filename= template.getName()+".doc";
    ActionContext.getContext().put("filename", filename);
    return "download";
}
```

动态确定下载文件的名称

````markdown
<result name="download" type="stream">
    <param name="inputName">docStream</param>
    <param name="contentDisposition">attachment;filename="${filename}"</param>
</result>```markdown
````
