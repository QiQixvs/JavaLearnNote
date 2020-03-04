# servlet

## servlet3.0特性（了解）

在servlet3.0 中可以使用注解来替代我们配置文件，所以可以没有web.xml文件。

问题：怎样知道我们当前使用的是哪个版本?

在web.xml文件中有一个属性version=""它就可以标识当前是哪个版本.

* 版本对应关系

|Servlet|JavaEE|Tomcat|JDK|
|:-----|:-----|:-----|:-----|
|servlet2.5|javaee5.0|tomcat 5.x tomcat6|jdk1.5|
|servlet3.0|javaee6.0|tomcat7.0|jdk1.6|

关于servlet3.0特性:

### 1. 注解配置文

* @WebServlet("/hello") 用于配置servlet
* @WebFilter("/*")      用于配置Filter
* @WebListener          用于配置Listener

关于这些注解细节:
以 @WebServlet("/hello") 为例,相当于value=“/hello”

{% hint style="danger" %}
属性urlpatterns与values它们都是描述访问当前servlet的路径，但它们不能一起出现，只能使用一个.
{% endhint %}

web.xml

```java
<servlet>
    <servlet-name></servlet-name>   String name() default "";
    <servllet-class></servlet-class>
    <init-param>       WebInitParam[] initParams() default {};
        <param-name>
        <param-value>
    </init-param>
    <load-on-startup>     int loadOnStartup() default -1;
</servlet>

<servlet-mapping>
    <servlet-name></servlet-name>
    <url-pattern></url-pattern>
    String[] urlPatterns() default {};
    或
    String[] value() default {};
</servlet-mapping>
```

#### 在servlet中怎样获取初始化参数

ServletConfig对象获取

```java

@WebServlet(urlPatterns = { "/hello", "/h1" },initParams={@WebInitParam(name="username",value="tom"),@WebInitParam(name="encode",value="utf-8")})
public class MyServlet extends HttpServlet {

    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp)throws ServletException, IOException {

        // 获取初始化参数
        ServletConfig config = this.getServletConfig();
        String username = config.getInitParameter("username");

    }
...
```

在web.xml文件中的属性 metadata-complete,可以取值为true, false,
如果为false,代表servlet3.0中的注解可以使用，如果为true,代表不可以使用注解。

### 2. 文件上传

浏览器端:

1. method=post
2. encType="multipart/form-data"
3. 使用&lt;input type="file" name="f"&gt;

服务器端:

servlet3.0完成。

1.要在servlet上添加注解@MultipartConfig  
表示Servlet接收multipart/form-data 请求
2.在servlet中要想得到上传信息，通过request对象获取一个Part对象。
Part part=request.getPart();

part.write(String filename);

问题:
1.关于上传文件中文名称乱码问题
因为上传是post请求，直接使用post乱码解决方案就可以  request.setCharacterEncoding("utf-8");
2.关于获取上传文件名称 
通过Part获取一个header
String cd = part.getHeader("Content-Disposition");
在这个header中包含了上传文件名称，直接截取出来就可以。							
String filename = cd.substring(cd.lastIndexOf("\\") + 1,cd.length() - 1);
3.如果多文件上传怎样处理?
request.getParts();

### 3. 异步处理

本质就是在服务器端开启一个线程，来完成其它的操作。

1.必须在注解添加一项
@WebServlet(value = "/reg", asyncSupported = true)
asyncSupported=true,代表当前servlet支持异步操作.

2.需要一个异步 上下文对象，通过这个对象，可以获取request,response对象.

AsyncContext context = req.startAsync();	

还可以对异步上下文进行监听，在它的监听器方法中有一个onComplete,可以用于判断结束。

## Servlet4.0

(...)