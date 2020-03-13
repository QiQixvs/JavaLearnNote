---
description: 国际化(了解);拦截器(Interceptor)---重点;struts2文件上传与下载(次重点);ognl与valuestack
---

# Struts2框架-3

## 1. struts2中国际化

struts2中对国际化进行了封装，我们只需要根据其提供的API进行访问就可以。

### 1.1 怎样定义properties

#### 1. 全局

需要通过一个常量来声明.

```markdown
struts.custom.i18n.resources=testmessages,testmessages2
```

在struts.xml中配置常量：

```markdown
<constant name="struts.custom.i18n.resources" value="message">
代表message.properties在src下 此处value就是配置文件的基名。

<constant name="struts.custom.i18n.resources" value="i18n.resource.message">
代表message.properties在i18n.resource包下。
```

properties 配置文件可以放置在任意位置

#### 2. 局部

##### a. 针对于action类

位置: 与action类在同一个包下.

名称: ActionClassName.properties.

这个配置文件只对当前action有效。

##### b. 针对于package下所有action

位置: 在指定的包下

名称: package.properties

##### C. jsp页面临时使用某一个properties文件

```MARKDOWN
<s:i18n name="包名.文件名"></s:i18n>
```

### 1.2 在哪些位置使用

1. action类中使用

2. 配置文件中使用 &LT;validation.xml&GT;

3. 在jsp页面上使用

### 1.3 怎样操作

#### 1. 在action类中使用

前提: action类要**继承ActionSupport类**。

```JAVA
this.getText(String name);
//就可以获取配置文件中对应名称的值。
```

#### 2. 在validation.xml文件中使用

校验器的错误信息配置

```JAVA
<message key="properties文件中的名称"/>
```

#### 3. 在jsp页面上使用

```JAVA
<s:i18n name="包名.文件名">
    <s:text name="名称"/>;
</s:i18n>
```

如果没有使用&LT;s:i18n name=""&GT;来指定，会从全局配置文件中获取。

如果要从某一个配置文件中获取，通过name属性来指定， 包名.配置文件名称

### 1.4 在struts2中国际化配置文件中使用动态文本

#### 1. action中怎样使用

```JAVA
properties文件中：
msg=hello world  {0}

Action中：
this.getText("msg",new String[]{"tom"})
```

结果就是 hello world tom

#### 2. jsp页面上怎样使用

```JAVA
properties文件中：
msg=hello world  {0}

JSP页面上：
<s:i18n name="cn.itcast.action.I18nDemo1Action">
    <s:text name="msg">
        <s:param>张三</s:param>
    </s:text>
</s:i18n>
```

结果就是 hello world  张三.

## 2. 拦截器(interceptor)

struts2拦截器使用的是AOP思想(面向切面编程）。AOP的底层实现就是动态代理。

拦截器 采用 **责任链** 模式

* 在责任链模式里,很多对象由每一个对象对其下家的引用而连接起来形成一条链。
* 责任链每一个节点，都可以继续调用下一个节点，也可以阻止流程继续执行。

struts2中在struts-default.xml文件中声明了所有的拦截器, 而struts2框架默认使用的是**defaultStack**这个拦截器栈。

在这个拦截器栈中使用了18个拦截器。简单说，struts2框架在默认情况下，加载了18个拦截器。

### 2.1 怎样使用

可以通过使用拦截器进行控制action的访问。例如，权限操作。

怎样使用拦截器?

#### 1. 创建一个Interceptor

可以自定义一个类实现 com.opensymphony.xwork2.interceptor.Interceptor

在这个接口中有三个方法  init  destory intercept， intercept方法是真正拦截的方法。

在intercept方法中如果要向下继续执行，通过其参数ActionInvocation调用它的invoke()方法就可以。

#### 2. 声明一个Interceptor

在struts-default.xml文件中

```markdown
<interceptors>
    <interceptor name="" class=""/>
</interceptors>
```

注意:我们要自己声明一个interceptor在struts.xml文件的&lt;package&gt;标签下用同样格式声明。

interceptor配置写在action配置之前。

#### 3. 在action中指定使用哪些拦截器

在struts.xml文件的&lt;action&gt;标签下

```markdown
<interceptor-ref name="my"/>
```

{% hint style="info" %}
只要显示声明使用了一个拦截器。那么默认的拦截器就不再加载，若需要使用必须再引用。
{% endhint %}

### 2.2 分析拦截器原理

源代码执行流程:

#### 1. 在StrutsPrepareAndExecuteFilter中查找

在doFilter方法内有一句话 execute.executeAction (request, response, mapping) 执行Action操作.

#### 2. 在executeAction执行过程中会访问Dispatcher类中的serviceAction

在这个方法中会创建一个

ActionProxy proxy = config.getContainer().getInstance(ActionProxyFactory.class).createActionProxy(namespace, name, method, extraContext, true, false);

这就是我们的Action的代理对象

#### 3. 查看ActionInvocation，查看其实现类 DefaultActionInvocation

在其invoke方法中

```java
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
```

通过源代码分析，发现在DefaultActionInvocation中就是通过递归完成所有的拦截调用操作.

#### 关于interceptor与Filter区别

1. 拦截器是基于java反射机制的，而过滤器是基于函数回调的。
2. 过滤器依赖于servlet容器，而拦截器不依赖于servlet容器。
3. 拦截器只能对Action请求起作用，而过滤器则可以对几乎所有请求起作用。
4. 拦截器可以访问Action上下文、值栈里的对象，而过滤器不能。
5. 在Action的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化时被调用一次。

### 2.3 权限控制案例

#### 1. login.jsp------>LoginAction（登录成功，将用户存储到session）------->book.jsp

```java
public class LoginAction extends ActionSupport implements ModelDriven<User> {
    private User user = new User();
    @Override
    public User getModel() {
        return user;
    }
    @Override
    public String execute() throws Exception {
        System.out.println(user);
        if(登录成功){
            ServletActionContext.getRequest().getSession().setAttribute("user",user);
            return SUCCESS;
        }else{
            this.addActionError("用户名或密码错误");  
            //如果执行Action对应的服务出问题报错，对比addFieldError用法
            //在jsp页面上显示<s:actionerror/>
            return INPUT;
        }
    }
}
```

#### 2. 在book.jsp中提供crud链接，每一个连接访问一个BookAction中一个方法

```markdown
 <a href="${pageContext.request.contextPath}/book_add">book add</a><br>
 <a href="${pageContext.request.contextPath}/book_update">book update</a><br>
 <a href="${pageContext.request.contextPath}/book_delete">book delete</a><br>
 <a href="${pageContext.request.contextPath}/book_search">book search</a>
```

创建类不在实现Interceptor接口，而是继承其下的一个子类.MethodFilterInterceptor
不用在重写intercept方法，而是重写 doIntercept方法。

```java
public class BookInterceptor extends MethodFilterInterceptor {

    @Override
    protected String doIntercept(ActionInvocation actionInvocation) throws Exception {
        User user = (User) ServletActionContext.getRequest().getSession().getAttribute("user");
        if(user == null){
            //通过ActionInvocation的getAction的方法可以获得被拦截的Action对象
            BookAction action = (BookAction) actionInvocation.getAction();
            action.addActionError("没有权限，先登录");
            return Action.LOGIN;
        }
        return actionInvocation.invoke();
    }
}
```

要求: 对于BookAction中的add,update,delete方法要求用户必须登录后才可以访问。search无要求
怎样解决只控制action中某些方法的拦截？

```markdown
<struts>
    <package name="default" namespace="/" extends="struts-default">
        <interceptors>
            <interceptor name="bookInterceptor" class="intercept.BookInterceptor">
                <!--通过拦截器参数控制对某些方法的拦截-->
                <param name="includeMethods">add,delete,update</param>
                <param name="excludeMethods">search</param>
            </interceptor>
            <interceptor-stack name="myStack"><!---组成自定义拦截器栈-->
                <interceptor-ref name="bookInterceptor"/>
                <interceptor-ref name="defaultStack"/>
            </interceptor-stack>
        </interceptors>
        <global-results> <!--配置全局通用视图-->
            <result name="login">/login.jsp</result>
        </global-results>
        <action name="login" class="action.LoginAction">
            <result name="input">/login.jsp</result><!---登录失败-->
            <result>/book.jsp</result><!---登录成功-->
        </action>
        <action name="book_*" class="action.BookAction" method="{1}"><!--对action名称用通配符-->
            <interceptor-ref name="myStack"/><!--引用拦截器--->
        </action>
    </package>
</struts>
```

## 3. struts2中文件上传与下载

### 3.1 上传

浏览器端:

1. method=post
2. &lt;input type="file" name="xx"&gt;
3. encType="multipart/form-data";

服务器端: commons-fileupload组件

struts2中文件上传:
默认情况下struts2框架使用的就是commons-fileupload组件.

struts2它使用了一个interceptor帮助我们完成文件上传操作。

```markdown
<interceptor name="fileUpload" class="org.apache.struts2.interceptor.FileUploadInterceptor"/>
```

#### 在action中怎样处理文件上传

页面上组件:

```markdown
<input type="file" name="upload">
```

在action类中中要有三个属性,提供get/set方法

```java
private File upload; //必须要和页面上的name相同
private String uploadContentType; //页面上的组件名+ContentType
private String uploadFileName; //页面上的组件名+FileName
```

{% hint style="danger" %}
注意大小写
{% endhint %}

在execute方法中使用commons-io包下的FileUtils完成文件复制.

```java
FileUtils.copyFile(upload, new File("d:/upload",uploadFileName));
```

#### 关于struts2中文件上传细节

##### 1. 关于控制文件上传大小

在default.properties文件中定义了文件上传大小

struts.multipart.maxSize=2097152 上传文件默认的总大小 2m

##### 2. 在struts2中默认使用的是commons-fileupload进行文件上传

```text
 # struts.multipart.parser=cos
 # struts.multipart.parser=pell

 struts.multipart.parser=jakarta
```

如果使用pell,cos进行文件上传，必须导入其jar包.

##### 3. 配置input视图，在页面上可以通过<s:actionerror>展示错误信息

问题:在页面上展示的信息，全是英文，要想展示中文，国际化

struts-messages.properties 文件里预定义上传错误信息。

在UploadAction.properties文件中通过覆盖对应key 显示中文信息。

```MARKDOWN
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
```

##### 4. 关于多文件上传时的每个上传文件大小控制以及上传文件类型控制

1.多文件上传

服务器端:只需要将action属性声明成List集合或数组就可以。

```java
private List<File> upload;
private List<String> uploadContentType;
private List<String> uploadFileName;
```

2.怎样控制每一个上传文件的大小以及上传文件的类型?

在fileupload拦截器中，通过其属性进行控制.

```markdown
maximumSize---每一个上传文件大小
allowedTypes--允许上传文件的mimeType类型.
allowedExtensions--允许上传文件的后缀名.

<interceptor-ref name="defaultStack">
<param name="fileUpload.allowedExtensions">txt,mp3,doc</param>
</interceptor-ref>
```

### 3.2 下载

文件下载方式:

1. 超链接
2. 服务器编码，通过流向客户端写回。

struts2中文件下载：

通过&lt;result type="stream"&gt;完成。

```java
<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>
```

在StreamResult类中有三个属性需要在result配置时设置

```java
protected String contentType = "text/plain"; //用于设置下载文件的mimeType类型
protected String contentDisposition = "inline";//用于设置进行下载操作以及下载文件的名称
protected InputStream inputStream; //用于读取要下载的文件。
```

```markdown
<result type="stream">
    <param name="contentType">text/plain</param>
    <param name="contentDisposition">attachment;filename=a.txt</param>
    <param name="inputStream">${inputStream}</param> 会调用当前action中的getInputStream方法。ognl表达式
</result>
```

在action类中定义getInputStream方法

```java
public class DownloadAction extends ActionSupport{
    private String filename;
    //...get/set方法..
    public InputStream getInputStream() throws FileNotFoundException {
        FileInputStream fis = new FileInputStream("d:/upload/" + filename);
        return fis;
    }
    //execute方法。。。
}
```

* &lt;a href="${pageContext.request.contextPath}/download?filename=捕获.png"&gt;捕获.png&lt;/a&gt;下载报错

原因: 超链接是get请求，并且下载的文件是中文名称，乱码。

```java
filename = new String(filename.getBytes("ios8859-1"),"utf-8");
```

* ognl表达式配置文件类型和文件名

```markdown
<result type="stream">
    <param name="contentType">${contentType}</param> <!-- 调用当前action中的getContentType()方法 -->
    <param name="contentDisposition">attachment;filename=${downloadFileName}</param>
    <param name="inputStream">${inputStream}</param><!-- 调用当前action中的getInputStream()方法 -->
</result>
```

action类中获取文件类型

```java
public String getContentType(){
    String mimeType = ServletActionContext.getServletContext().getMimeType(filename);
    return mimeType;
}
```

action类中获取downloadFileName，中文乱码问题需要判断浏览器 参考* [文件的下载](fileupload-filedownload/file-download.md)

## 4. ognl与valueStack介绍

### 4.1 ognl

OGNL是Object-Graph Navigation Language的缩写，它是一种功能强大的表达式语言.

struts2将ognl表达式语言，集成当sturts2框架中，做为它的默认表达式语言。

OGNL 提供五大类功能

1. 支持对象方法调用，如xxx.doSomeSpecial()；
2. 支持类静态的方法调用和值访问
3. 访问OGNL上下文（OGNL context）和ActionContext； （重点 操作ValueStack值栈 ）
4. 支持赋值操作和表达式串联
5. 操作集合对象。

* 在jsp 结合 struts2 标签库 使用<s:property value="ognl表达式" />执行 ognl表达式
* 调用 实例方法 ：对象.方法()  --- <s:property value="'hello,world'.length()"/>
* 调用 静态方法 ： @[类全名（包括包路径）]@[方法名]  --- <s:property value="@java.lang.String@format('您好,%s','小明')"/>
* 使用 静态方法调用 必须 设置 struts.ognl.allowStaticMethodAccess=true

OgnlContext对象是一个Map集合，非根中的数据需要使用#获取，根中的数据不需要。

### 4.2 ValueStack值栈

ValueStack 是 struts2 提供一个接口，是一个容器，作用就是将action相关的数据以及web相关的对象携带到页面上。在页面上通过ognl表达式将ValueStack中数据获取出来。
