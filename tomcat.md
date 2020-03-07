# Tomcat

## 1.配置虚拟主机

将estore项目配置虚拟主机，以顶级域名方式进行发布。在浏览器上直接输入www.estore.com就可以访问到我们的工程.

### 1.1 在tomcat的conf目录下的server.xml文件中配置

* 修改tomcat的端口 80.
* 配置虚拟主机

```java
<Engine name="Catalina" defaultHost="www.estore.com">
<Host name="www.estore.com" appBase="D:\java1110\workspace\estore" unpackWARs="true" autoDeploy="true">
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
    prefix="localhost_access_log." suffix=".txt"
    pattern="%h %l %u %t &quot;%r&quot; %s %b" />

<Context path="" docBase="D:\java1110\workspace\estore\WebRoot"/>
</Host>
```

### 1.2 在C:\Windows\System32\drivers\etc\hosts文件中配置

127.0.0.1  www.estore.com

注意: 在启动tomcat时，不需要将工程estore工程部署到tomcat.
