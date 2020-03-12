---
description: 国际化(了解);拦截器(Interceptor)---重点;struts2文件上传与下载(次重点);ognl与valuestack
---

# Struts2框架-3

## 1. struts2中国际化

struts2中对国际化进行了封装，我们只需要根据其提供的API进行访问就可以。

问题1:在struts2中国际化时properties文件怎样定义？(怎样定义properties)

1.全局
需要通过一个常量来声明.
struts.custom.i18n.resources=testmessages,testmessages2

对于properties配置文件可以放置在任意位置

<constant name="struts.custom.i18n.resources" value="message"> 代表message.properties在src下
<constant name="struts.custom.i18n.resources" value="cn.itcast.i18n.resource.message"> 代表message.properties在cn.itcast.i18n.resource包下.
2.局部
1.针对于action类
位置:与action类在同一个包下.
名称:ActionClassName.properties.
这个配置文件只对当前action有效。
2.针对于package下所有action
位置:在指定的包下
名称:package.properties
3.jsp页面临时使用某一个properties文件.
<s:i18n name="cn.itcast.action.package"></s:i18n>


问题2:在struts2中国际化操作可以在哪些位置使用?(在哪此位置上使用)

1.action类中使用

2.配置文件中使用<validation.xml>

3.在jsp页面上使用


问题3:怎样在struts2中操作国际化?(怎样使用)
1.在action类中使用
前提:action类要继承ActionSupport类。

getText(String name)就可以获取配置文件中对应名称的值。

2.在validation.xml文件中使用

<message key="名称"/>

3.在jsp页面上使用

<s:text name="名称"> 如果没有使用<s:i18n name="">来指定，会从全局配置文件中获取。
如果要从某一个配置文件中获取，通过name属性来指定，  包名.配置文件名称 .
	--------------------------------------------------------
	在struts2中国际化配置文件中使用动态文本
		1.action中怎样使用
			msg=hello world  {0}
			this.getText("msg",new String[]{"tom"})
			
			结果就是 hello world tom
			
		2.jsp页面上怎样使用
			msg=hello world  {0}
			
			<s:i18n name="cn.itcast.action.I18nDemo1Action">
				<s:text name="msg">
					<s:param>张三</s:param>
				</s:text>
			</s:i18n>
			
			结果就是 hello world  张三.

## 2.拦截器(interceptor)

struts2拦截器使用的是AOP思想。
AOP的底层实现就是动态代理。
拦截器 采用 责任链 模式 
*  在责任链模式里,很多对象由每一个对象对其下家的引用而连接起来形成一条链。
*  责任链每一个节点，都可以继续调用下一个节点，也可以阻止流程继续执行

struts2中在struts-default.xml文件中声明了所有的拦截器。
而struts2框架默认使用的是defaultStack这个拦截器栈。
在这个拦截器栈中使用了18个拦截器。简单说，struts2框架
在默认情况下，加载了18个拦截器。		

### 2.1 struts2中怎样使用拦截器

问题:使用拦截器可以做什么？
可以通过使用拦截器进行控制action的访问。例如，权限操作。

怎样使用拦截器?
1.创建一个Interceptor  可以自定义一个类实现com.opensymphony.xwork2.interceptor.Interceptor
在这个接口中有三个方法  init  destory intercept， intercept方法是真正拦截的方法。

在intercept方法中如果要向下继续执行，通过其参数ActionInvocation调用它的invoke()方法就可以。			

2.声明一个Interceptor  
在struts-default.xml文件中
<interceptors>
<interceptor name="" class=""/>
</interceptors>
注意:我们要自己声明一个interceptor可以在struts.xml文件中声明。

3.在action中指定使用哪些拦截器.
<interceptor-ref name="my"/>

注意:只要显示声明使用了一个拦截器。那么默认的拦截器就不在加载。

### 2.2 分析拦截器原理

源代码执行流程:
1.在StrutsPrepareAndExecuteFilter中查找
在doFilter方法内有一句话 execute.executeAction (request, response, mapping) 执行Action操作.

2.在executeAction执行过程中会访问Dispatcher类中的serviceAction，在这个方法中会创建一个
ActionProxy proxy = config.getContainer().getInstance(ActionProxyFactory.class).createActionProxy(namespace, name, method, extraContext, true, false);
这就是我们的Action的代理对象

3.查看ActionInvocation，查看其实现类 DefaultActionInvocation.

在其invoke方法中
if (interceptors.hasNext()) {//判断是否有下一个拦截器.
final InterceptorMapping interceptor = interceptors.next(); //得到一个拦截器
String interceptorMsg = "interceptor: " + interceptor.getName();
UtilTimerStack.push(interceptorMsg);
try {
resultCode = interceptor.getInterceptor().intercept(DefaultActionInvocation.this); 
//调用得到的拦截器的拦截方法.将本类对象传递到了拦截器中。
}
finally {
UtilTimerStack.pop(interceptorMsg);
}
} 

通过源代码分析，发现在DefaultActionInvocation中就是通过递归完成所有的拦截调用操作.


关于interceptor与Filter区别:
1、拦截器是基于java反射机制的，而过滤器是基于函数回调的。
2、过滤器依赖于servlet容器，而拦截器不依赖于servlet容器。
3、拦截器只能对Action请求起作用，而过滤器则可以对几乎所有请求起作用。
4、拦截器可以访问Action上下文、值栈里的对象，而过滤器不能。
5、在Action的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化时被调用一次。

### 2.3 案例

权限控制:
1.login.jsp------>LoginAction------------->book.jsp
登录成功，将用户存储到session。

2.在book.jsp中提供crud链接。
每一个连接访问一个BookAction中一个方法。

要求:对于BookAction中的add,update,delete方法要求用户必须登录后才可以访问。search无要求。	

怎样解决只控制action中某些方法的拦截？
1.创建类不在实现Interceptor接口，而是继承其下的一个子类.MethodFilterInterceptor
不用在重写intercept方法，而是重写 doIntercept方法。

2.在struts.xml文件中声明
<interceptors>
<intercept name="" class="">
<param name="includeMethods">add,update,delete</param>
<param name="excludeMethods">search</param>
</intercept>
</interceptors>

## 3. struts2中文件上传与下载

### 3.1 上传

浏览器端:
1.method=post
2.<input type="file" name="xx">
3.encType="multipart/form-data";

服务器端:
commons-fileupload组件
1.DiskFileItemFactory
2.ServletFileUpload
3.FileItem

struts2中文件上传:
默认情况下struts2框架使用的就是commons-fileupload组件.
struts2它使用了一个interceptor帮助我们完成文件上传操作。
<interceptor name="fileUpload" class="org.apache.struts2.interceptor.FileUploadInterceptor"/>

在action中怎样处理文件上传?
页面上组件:<input type="file" name="upload">

在action中要有三个属性:
private File upload;
private String uploadContentType;
private String uploadFileName;

在execute方法中使用commons-io包下的FileUtils完成文件复制.			
FileUtils.copyFile(upload, new File("d:/upload",uploadFileName));


#### 关于struts2中文件上传细节

1.关于控制文件上传大小
在default.properties文件中定义了文件上传大小
struts.multipart.maxSize=2097152 上传文件默认的总大小 2m

2.在struts2中默认使用的是commons-fileupload进行文件上传。

* /# struts.multipart.parser=cos
* /# struts.multipart.parser=pell

struts.multipart.parser=jakarta

如果使用pell,cos进行文件上传，必须导入其jar包.

3.如果出现问题，需要配置input视图，在页面上可以通过<s:actionerror>展示错误信息.
问题:在页面上展示的信息，全是英文，要想展示中文，国际化

struts-messages.properties 文件里预定义 上传错误信息，通过覆盖对应key 显示中文信息
struts.messages.error.uploading=Error uploading: {0}
struts.messages.error.file.too.large=The file is to large to be uploaded: {0} "{1}" "{2}" {3}
struts.messages.error.content.type.not.allowed=Content-Type not allowed: {0} "{1}" "{2}" {3}
struts.messages.error.file.extension.not.allowed=File extension not allowed: {0} "{1}" "{2}" {3}

修改为
struts.messages.error.uploading=上传错误: {0}
struts.messages.error.file.too.large=上传文件太大: {0} "{1}" "{2}" {3}
struts.messages.error.content.type.not.allowed=上传文件的类型不允许: {0} "{1}" "{2}" {3}
struts.messages.error.file.extension.not.allowed=上传文件的后缀名不允许: {0} "{1}" "{2}" {3}

{0}:<input type=“file” name=“uploadImage”>中name属性的值
{1}:上传文件的真实名称
{2}:上传文件保存到临时目录的名称
{3}:上传文件的类型(对struts.messages.error.file.too.large是上传文件的大小)

4.关于多文件上传时的每个上传文件大小控制以及上传文件类型控制.

1.多文件上传
服务器端:
只需要将action属性声明成List集合或数组就可以。

private List<File> upload;
private List<String> uploadContentType;
private List<String> uploadFileName;

2.怎样控制每一个上传文件的大小以及上传文件的类型?
在fileupload拦截器中，通过其属性进行控制.

maximumSize---每一个上传文件大小
allowedTypes--允许上传文件的mimeType类型.
allowedExtensions--允许上传文件的后缀名.

<interceptor-ref name="defaultStack">
<param name="fileUpload.allowedExtensions">txt,mp3,doc</param>
</interceptor-ref>

### 3.2 下载

文件下载方式:
1.超连接
2.服务器编码，通过流向客户端写回。

1.通过response设置  response.setContentType(String mimetype);
2.通过response设置  response.setHeader("Content-disposition;filename=xxx");
3.通过response获取流，将要下载的信息写出。



struts2中文件下载：		
通过<result type="stream">完成。

<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>
在StreamResult类中有三个属性:
protected String contentType = "text/plain"; //用于设置下载文件的mimeType类型
protected String contentDisposition = "inline";//用于设置进行下载操作以及下载文件的名称
protected InputStream inputStream; //用于读取要下载的文件。

在action类中定义一个方法
public InputStream getInputStream() throws FileNotFoundException {
FileInputStream fis = new FileInputStream("d:/upload/" + filename);
return fis;
}

<result type="stream">
<param name="contentType">text/plain</param>
<param name="contentDisposition">attachment;filename=a.txt</param>
<param name="inputStream">${inputStream}</param> 会调用当前action中的getInputStream方法。
</result>


问题1:<a href="${pageContext.request.contextPath}/download?filename=捕获.png">捕获.png</a>下载报错
原因:超连接是get请求，并且下载的文件是中文名称，乱码。


问题2:下载捕获文件时，文件名称就是a.txt	,下载文件后缀名是png,而我们在配置文件中规定就是txt?			
<result type="stream">
<param name="contentType">${contentType}</param> <!-- 调用当前action中的getContentType()方法 -->
<param name="contentDisposition">attachment;filename=${downloadFileName}</param>
<param name="inputStream">${inputStream}</param><!-- 调用当前action中的getInputStream()方法 -->
</result>

在struts2中进行下载时，如果使用<result type="stream">它有缺陷，例如：下载点击后，取消下载，服务器端会产生异常。
在开发中，解决方案:可以下载一个struts2下载操作的插件，它解决了stream问题。

## 4. ognl与valueStack

问题:ognl是什么，它有什么用?
OGNL是Object-Graph Navigation Language的缩写，它是一种功能强大的表达式语言.
比el表达式功能强大。
struts2将ognl表达式语言，集成当sturts2框架中，做为它的默认表达式语言。

OGNL 提供五大类功能 
1、支持对象方法调用，如xxx.doSomeSpecial()； 
2、支持类静态的方法调用和值访问
3、访问OGNL上下文（OGNL context）和ActionContext； （重点 操作ValueStack值栈 ）
4、支持赋值操作和表达式串联
5、操作集合对象。

问题:valueStack是什么，它有什么用?
