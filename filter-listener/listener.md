# Listener 监听器

## **概念**

1. 事件ActionEvent
2. 事件源 Jbutton
3. 监听器ActionListener
4. 注册监听addActionListener

   监听器就是可以监听某一个事件在执行一个特定操作时，我们可以让其触发一个操作。 可以在满足特定条件的情况下执行一段操作。

   javaweb中的监听器，主要用于监听javaweb中常用对\(**request\(HttpServletRequest\), session\(HttpSession\), application\(ServletContext**\)的三种类型操作：

5. 对象的创建与销毁
6. 对象的属性变化
7. session绑定javaBean

**在javaweb中servlet规范中定义了三种技术 servlet、Listener 、Filter**

{% hint style="info" %}
servlet有初始化参数

Filter有初始化参数

Listener没有初始化参数,要使用，在开发中一般使用servletContext的初始化参数对应的监听器接口
{% endhint %}

#### 1.监听创建与销毁

HttpServletRequest

```text
    监听器SevletRequestListene可监听request对象的创建与销毁.
```

HttpSession

```text
     监听器HttpSessionListener可以监听session对象的创建与销毁.
```

ServletContext

```text
      监听器ServletContextListener可以监听application对象的创建与销毁。
```

#### 2.监听web对象的属性变化

HttpServletRequest属性变化

```text
      监听器ServletRequestAttributeListener监听request对象的属性变化
```

HttpSession属性变化

```text
      监听器HttpSessionAttributeListener监听session对象的属性变化
```

ServletContext属性变化

```text
       监听器ServletContextAttributeListener监听application对象的属性变化。
```

## web中监听器怎样使用

创建监听器步骤:

1. 创建一个类，去实现指定的监听器接口
2. .重写接口中方法。
3. 在web.xml文件中配置注册监听

```java
@WebListener//注解方式
public class MyServletContextListener implements ServletContextListener {

    public void contextDestroyed(ServletContextEvent sce) {

    }

    public void contextInitialized(ServletContextEvent sce) {
        // 这个方法执行了，就说明项目启动了.}
}
```

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">


    <!-- 注册监听ServletContext对象创建与销毁 -->
    <listener> 
        <listener-class>cn.itcast.web.listener.application.MyServletContextListener</listener-class> 
    </listener>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

### 三个对象何时创建何时销毁

1. application对象是服务器启动时创建，服务器关闭时销毁。
2. session对象 session对象创建:reqeust.getSession\(\);它是用于获session. 是否创建，分以下几种情况:

   ```text
    1.请求中如果没有jsessionid,那么就是创建session对象。

    2.如果请求头中有jsessionid值：

        1.如果在服务器端，有一个session的id值与其一样，不创建，直接使用。

        2.如果在服务器端，没有这个session的id值，那么会创建。
   ```

   session销毁:

   ```text
    1.默认超时 30分钟

    2.设置session超时时间

        setMaxInactiveInterval(int interval)

    3.invalidate()手动销毁.

    4.关闭服务器
   ```

3. request对象 请求发生，request对象创建，响应产生request对象销毁。

{% hint style="info" %}
session对象是否创建，看请求中所要的session与服务器端的session是否是同一个。
{% endhint %}

### 演示监听属性变化

演示监听session的属性变化

问题:在监听器中是否可以得到属性值?

![](../.gitbook/assets/image2.png) ![](../.gitbook/assets/image3.png)

{% hint style="success" %}
在java的监听机制中，是可以在监听器中获取事件源的我们在开发中，如果有到了事件触发机制，那么一般情况下，都可以使用方法的参数\(事件对象\)来获取想要的信息.
{% endhint %}

这些监听器在开发中有什么用?

在主流中应用比较少，但是可以完成一些性能监试操作。

### 监听器案例

功能：扫描session对象在指定时间内没有使用，人为销毁。

分析:

1. 怎样知道session多长时间没有使用？

   当前时间-最后使用时间（public long getLastAccessedTime\(\)）

2. 什么时候开始扫描，扫描多长时间?

   可以使用Timer完成

   ```java
    import java.util.Timer;
    import java.util.TimerTask;

    public class TimerDemo {

        public static void main(String[] args) {

            Timer t = new Timer();

            t.schedule(new TimerTask() {

                @Override
                public void run() {
                    System.out.println("hello timer");
                }
            }, 1000,2000);//延迟一秒，间隔两秒 
        }
    }
   ```

完成定时扫描session，如果超时没有使用，销毁案例：

1. 要将所有的session对象得到，保存到集合中。
2. 创建一个监听器 ServletContextListener,它**服务器启动**时，创建一个集合保存到ServletContext域。
3. 创建一个监听器 HttpSessionListener,当创建一个session时，就从ServletContext域中获取集合，将session对象储存到集合中。
4. 定时扫描

   ```java
    public void contextInitialized(ServletContextEvent sce) {
        // 这个方法执行了，就说明项目启动了.

        // 1.得到ServletContext对象
        ServletContext context = sce.getServletContext();

        // 2,将集合保存到context中.
        context.setAttribute("sessions", sessions);

        // 3.开始扫描
        Timer t = new Timer();

        t.schedule(new TimerTask() {

            @Override
            public void run() {
                // 判断session是否过期.----session如果10秒钟没有使用
                // 从集合中删除
                // 销毁session
                }
            }
        }, 1000, 3000);

    }
   ```

{% hint style="info" %}
```text
问题:
1. session超时，不能只销毁session，还要从集合中移除。

2. 我们的操作，它是多线程的，要考虑集合的同步问题。

    1. 集合需要是线程安全的。 

    2. 需要使用迭代器进行遍历。
```
{% endhint %}

## session绑定javaBean\(了解\)

1. HttpSessionBindingListener
2. HttpSessionActivationListener

这两个监听器特点;

1. 它们是由javaBean实现.
2. 它们不需要在web.xml文件中配置.
3. HttpSessionBindingListener

   这个监听器，可以让javaBean对象，感知它被绑定到session中或从session中移除。

4. HttpSessionActivationListener

   这个监听器，可以让javaBean感知，被钝化或活化。

{% hint style="info" %}
钝化---&gt;将session中的javaBean保存到文件中. 活化---&gt;从文件中将javaBean直接获取。
{% endhint %}

```text
需要创建一个配置文件context.xml

这个文件保存到META-INF目录下.
```

```java
<Context>

    <Manager className="org.apache.catalina.session.PersistentManager" maxIdleSwap="1">

        <Store className="org.apache.catalina.session.FileStore" directory="it315">

    </Manager>

</Context>
```

