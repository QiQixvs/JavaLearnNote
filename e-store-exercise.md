---
description: web项目servlet应用综合练习
---

# 商城项目练习笔记

## UML用例图（系统需求分析）

![UML&#x7528;&#x4F8B;&#x56FE;](.gitbook/assets/2020-03-06-20-30-49.png)

## E-R图（数据库设计）

![E-R](.gitbook/assets/2020-03-06-20-26-43.png)

## 时序图

![sequence diagram](.gitbook/assets/2020-03-06-20-28-03.png)

## 记录

### InputStreamReader读取本地中文字符文件乱码

```java
public void init() throws ServletException {
    // 初始化阶段，读取WEB-INF目录下待选验证码文件new_words.txt
    // web工程中读取 文件，必须使用绝对磁盘路径
    String path = getServletContext().getRealPath("/WEB-INF/new_words.txt");
    try {
        //BufferedReader reader = new BufferedReader(new FileReader(path));
        //使用FileReader的父类InputStreamReader，指定charset
        BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(path),"utf-8"));
        String line;
        while ((line = reader.readLine()) != null) {
            words.add(line);
        }
        reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### refresh倒计时刷新将用户并重定向到另外一个地址

* 在servlet中实现

```java
response.setHeader("refresh","3;url=http://www.estore.com");
```

* 在jsp页面上实现

```java
<head>
<meta http-equiv="refresh" content="3;url=http://www.estore.com"/>
</head>
```

写法一：

```java
<script type="text/javascript">
    var x =3;
    function run() {
        var span = document.getElementById("timeid");
    span.innerHTML = x;
    x--;
    window.setTimeout("run()",1000);
}


</script>
<body onload="run()">
注册成功！您将在 <span id="timeid">3</span> 秒内跳转到首页
```

写法二：

```java
<script type="text/javascript">
    var interval;
    window.onload=function () {
        interval= window.setInterval("run()",1000);
    }
    function run() {
    var time =document.getElementById("timeid").innerHTML;
    if(time == 0){
        window.clearInterval(interval);
    return;
    }
    document.getElementById("timeid").innerHTML=(time-1);
}
</script>
```

### 全局编码过滤

* [Filter案例](filter-listener/filter-examples.md)

### Exception自定义异常

#### 1. 继承Exception

```java
public class RegistException extends Exception{}
...
```

```java
//在service中抛出自定义异常
}catch(SQLException e){
    throw new RegistException("注册失败")
}...
```

```java
//在servlet中catch自定义异常，获得异常信息。
}catch(RegistException){
    request.setAttribute("regist.msg",e.getMessage());
    ...
}
```

#### 2. 继承RuntimeException

```java
public class RegistException extends RuntimeException{}
```

在web.xml文件中配置全局异常处理，指向自定义错误页面。

```java
<error-page>
    <exception-type>cn.itcast.estore.exception.RegistException</exception-type>
    <location>/error/registerror.jsp</location>
</error-page>
```

### 一次性验证码

在所有操作前，通过requst获取请求中的验证码，与session中存储的验证码进行对比。 从session中获取完成后，马上删除。

```java
request.getSession().setAttribute("checkcode_session", word);
```

```java
String checkCode = request.getParameter("checkcode");

String _checkCode = (String) request.getSession().getAttribute(
    "checkcode_session");
request.getSession().removeAttribute("checkcode_session");//从session中删除。

if (!checkCode.equals(_checkCode)) {
    request.setAttribute("regist.message", "验证码不正确");
    request.getRequestDispatcher("/regist.jsp").forward(request,
        response);
    return;
}
```

