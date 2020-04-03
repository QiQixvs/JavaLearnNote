---
description: 网上商城用户模块注册和登录功能
---

# SSH综合案例-1

## 导入相应jar包

Struts2的jar包:

* struts2框架解压路径/apps/struts2-blank.war/WEB-INF/lib/*.jar
* struts2框架解压路径/lib/struts2-spring-plugin-2.3.15.3.jar
* struts2框架解压路径/lib/struts2-json-plugin-2.3.15.3.jar

Spring的jar包:

Spring开发基本jar包

* spring框架解压路径/lib/spring-beans-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-context-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-core-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-expression-3.2.0.RELEASE.jar
* spring框架依赖包解压路径/com.springsource.org.apache.commons.logging-1.1.1.jar
* spring框架依赖包解压路径/com.springsource.org.apache.log4j-1.2.15.jar

Spring的AOP开发(Aspectj)

* spring框架解压路径/lib/spring-aop-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-aspects-3.2.0.RELEASE.jar
* spring框架依赖包解压路径/com.springsource.org.aopalliance-1.0.0.jar
* spring框架依赖包解压路径/com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar

Spring的JDBC支持、事务管理、整合Hibernate

* spring框架解压路径/lib/spring-jdbc-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-tx-3.2.0.RELEASE.jar
* spring框架解压路径/lib/spring-orm-3.2.0.RELEASE.jar
* Spring整合web项目:
* spring框架解压路径/lib/spring-web-3.2.0.RELEASE.jar
* Spring整合Junit单元测试:
* spring框架解压路径/lib/spring-test-3.2.0.RELEASE.jar

Hibernate框架jar包:

* hibernate框架解压路径/hibernate3.jar
* hibernate框架解压路径/lib/required/*.jar
* hibernate框架解压路径/lib/jpr/*.jar
* hibernate框架整合log4j
* slf4j-log4j12-1.7.2.jar

* 数据库驱动包
* c3p0连接池jar包.

## 配置文件

不使用hibernate配置文件，将hibernate的信息配置到spring框架中.

* 日志配置文件
* jdbc连接参数配置文件
* web.xml中核心过滤器
* web.xml中监听器加载Spring配置文件
* Spring配置文件中配置连接池信息
* Spring配置文件中配置Hibernate的相关属性
* 配置事务管理器

## 编码实现

Action类交给Spring管理，且注意scope="prototype"，所以Struts2配置中Action标签的class属性写的是Spring配置中类的id

User.hbm.xml配置好后，在spring配置中引入Hibernate映射信息

UserService类增加事务管理注解@Transactional

* 首页显示 通过action跳转
* 注册页面显示 通过action跳转
* 注册功能 Regist.jsp----->UserAction(regist方法)----->UserService（设置state和激活码，传递user对象）--------->UserDao,注册成功跳转信息页面，显示ActionMessage----this.addActionMessage("注册成功，请去邮箱激活");
* 注册后台校验 UserAction-user_regist-validation.xml，校验后的INPUT视图配置 @InputConfig(resultName = "registInput")
* 发送激活邮件
* 在邮箱地址上点击链接------->UserAction(active方法)--------->UserService------->UserDao，根据激活码完成查找，找到该用户，修改用户的状态. 0 修改为1，update数据库.

* 登录页面 UserAction(loginPage方法)，通过action跳转
* 登录，登录页面------->UserAction(login方法)--------UserService---------UserDao，接收提交的用户名和密码.调用Service根据用户名和密码及用户的状态查询. 登录失败显示错误信息this.addActionError("登录失败");
* 登录后台校验 UserAction-user_login-validation.xml@InputConfig(resultName = "loginInput")
* 登录成功，将用户存到Session作用范围，ServletActionContext.getRequest().getSession().setAttribute("existUser",existUser);，重定向到首页的Action显示用户姓名，type="redirectAction"，&lt;s:property value="#session.existUser.name"/&gt;
* 退出 user_quit.action 获得用户的session，将session销毁，ServletActionContext.getRequest().getSession().invalidate(); 重定向到首页的Action.

* 注册页面Ajax校验

## 注册页面前台js校验

```java
function checkForm(){
		// 校验用户名:
		var username = document.getElementById("username").value;
		if(username == ''){
			alert("用户名不能为空!");
			return false;
		}
		// 校验密码:
		var password = document.getElementById("password").value;
		if(password == ''){
			alert("密码不能为空!");
			return false;
		}
		
		// 校验确认密码
		var repassword = document.getElementById("repassword").value;
		if(password != repassword){
			alert("两次密码不一致!");
			return false;
		}
	}
```

## 发送激活邮件

```java
public class MailUtils {
	public static void sendMail(String to,String code) throws Exception{
		Properties props = new Properties();
		props.setProperty("mail.smtp", "localhost");
		// 1.Session对象.连接(与邮箱服务器连接)
		Session session = Session.getInstance(props, new Authenticator() {

			@Override
			protected PasswordAuthentication getPasswordAuthentication() {
				return new PasswordAuthentication("service@shop.com", "111");
			}
			
		});
		
		// 2.构建邮件信息:
		Message message = new MimeMessage(session);
		// 发件人:
		message.setFrom(new InternetAddress("service@shop.com"));
		// 收件人:
		message.setRecipient(RecipientType.TO, new InternetAddress(to));
		// 设置标题
		message.setSubject("来自SHOP激活邮件");
		// 设置正文
		message.setContent("<h1>来自SHOP的官网激活邮件</h1><h3><a href='http://192.168.40.99:8080/shop/user_active.action?code="+code+"'>http://192.168.40.99:8080/shop/user_active.action?code="+code+"</a></h3>", "text/html;charset=UTF-8");
	
		// 3.发送对象
		Transport.send(message);
	}
}

```

在邮箱地址上点击链接------->UserAction(active方法)--------->UserService---------->UserDao

修改用户的状态. 0 修改为1.

## 注册页面的Ajax校验

```markdown
<script type="text/javascript">
	function checkUserName(){
		// 获得用户名的值:
		var username = document.getElementById("username").value;
		// 1.创建异步加载对象:
		var xhr = createXMLHttpRequest();
		// 2.设置监听
		xhr.onreadystatechange = function(){
			if(xhr.readyState == 4){
				if(xhr.status == 200){
					var data = xhr.responseText;
					document.getElementById("span1").innerHTML = data;
				}
			}
		}
		// 3.打开连接:
		xhr.open("GET","${pageContext.request.contextPath}/user_checkUserName.action?"+new Date().getTime()+"&username="+username,true);
		// 4.发送
		xhr.send(null);
	}

	function createXMLHttpRequest() {
		var xmlHttp;
		try { // Firefox, Opera 8.0+, Safari
			xmlHttp = new XMLHttpRequest();
		} catch (e) {
			try {// Internet Explorer
				xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
			} catch (e) {
				try {
					xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
				} catch (e) {
				}
			}
		}

		return xmlHttp;
	}

</script>
```

```java
/**
     * 前台:注册AJAX校验用户名.
     * @throws IOException
     */
    public String checkUserName() throws IOException {
        User existUser = userService.findByUserName(user.getUsername());
        HttpServletResponse response = ServletActionContext.getResponse();
        response.setContentType("text/html;charset=UTF-8");
        if(existUser == null){
            // 用户名可以使用的
            response.getWriter().print("<font color='green'>用户名可以使用</font>");
        }else{
            // 用户名已经存在
            response.getWriter().print("<font color='red'>用户名已经存在</font>");
        }
        return NONE;
    }
}
```

### jquery

待补充

## 生成验证码CheckImgAction

在struts2中配置action，类交由spring管理，注意scope

* 重写execute方法，不需要返回值。

* 主要注意两处代码要用ServletActionContext，存验证码，和回写图片。

```java
ServletActionContext.getRequest().getSession()
                .setAttribute("checkcode", sb.toString());

ImageIO.write(bufferedImage, "jpg", ServletActionContext.getResponse()
                .getOutputStream());
```

* 在jsp页面上增加点击图片刷新验证码功能

* 在login和registAction中验证验证码
