# IDEA踩坑记录

## Tomcat 部署时 war exploded的问题

war exploded模式是直接把文件夹、jsp页面 、classes等等移到Tomcat 部署文件夹里面，进行加载部署。

![](../.gitbook/assets/image7.png) ![](../.gitbook/assets/capture.png)

用于获取上下文环境绝对路径的代码：

```text
String contextPath = request.getSession().getServletContext().getRealPath();
```

war exploded模式最终没有部署在Tomcat的位置，而是项目本身所在位置的指定目录下。

## Struts2 struts.xml extends="struts-default"报错

```text
<struts>
    <package name="default" namespace="/" extends="struts-default">
        <action name="hello" class="test.HelloAction" method="say">
            <result name="good">/hello.jsp</result>
        </action>
    </package>
</struts>
```

错误原因：新建项目后手动新建lib目录导致struts-default引用错误

解决方法：

File --&gt; Project Structure --&gt; Facets --&gt; Struts2

![struts-default](../.gitbook/assets/2020-03-09-15-34-09.png)
