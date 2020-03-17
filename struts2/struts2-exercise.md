# Struts2-练习笔记

## 目标功能

1. 登陆
2. 添加用户 （简历上传）
3. 组合条件 员工信息列表查询
4. 员工信息详情查看（简历下载）
5. 员工信息删除
6. 员工信息编辑
7. 异常处理
8. 登录校验

## 数据库设计

### Oracle和MySQL 作为应用数据库区别

mysql存在数据库概念，在企业开发中，针对一个项目创建一个单独数据库，创建单独用户， 为用户授予数据库权限 ，
oracle 一个数据库就是一个服务，在这个库中可以存在很多用户，每个用户有单独表空间 ，针对一个项目，只需要创建一个用户

### 创建用户与授权

为项目数据库创建单独用户，并为用户授予数据库权限,在c3p0配置中使用这个用户的信息连接数据库。

* [MySQL数据库操作-DCL](../mysql-ddl.md)

#### 用户表

```text
CREATE TABLE S_User(
    userID INT  NOT NULL AUTO_INCREMENT, #主键ID
    userName VARCHAR(50)   NULL,  #用户姓名
    logonName VARCHAR(50)   NULL, #登录名
    logonPwd VARCHAR(50)  NULL,   #密码#
    sex VARCHAR(10)  NULL,        #性别（例如：男，女）
    birthday VARCHAR(50) NULL,    #出生日期
    education VARCHAR(20)  NULL,  #学历（例如：研究生、本科、专科、高中）
    telephone VARCHAR(50)  NULL,  #电话 
    interest VARCHAR(20)  NULL,   #兴趣爱好（例如：体育、旅游、逛街）
    path VARCHAR(500)  NULL,      #上传路径（path路径）
    filename VARCHAR(100)  NULL,  #上传文件名称（文件名）
    remark VARCHAR(500)  NULL,    #备注
    PRIMARY KEY (userID)
);

初始化数据：默认用户名和密码是admin
INSERT INTO s_user (userID,userName,logonName,logonPwd) VALUES (1,'超级管理员','admin','admin');
```

## 搭建开发环境

## 功能实现

### 1.登录操作

#### 1. 使用struts2提供的表单标签来改造页面

```MARKDOWN
<form>-------------------<s:form>
<input type="text">------<s:textfield>
<input type="password">---<s:password>
<input type="submit">-----<s:submit>
<input type="reset">------<s:reset>
```

1.改造form

```MARKDOWN
<s:form id="loginAction_home" name="form1" action="user_login" namespace="/" target="_parent" method="post">
</s:form>
```

与form标签中action属性不同，这里的action写的是struts2.xml文件中对应的action的那么属性。

2.改造登录名

```MARKDOWN
<s:textfield name="logonName" value="" id="logonName" cssClass="text" cssStyle="width: 160px;"/>
```

3.改造登录密码

```MARKDOWN
<s:password  name="logonPwd" id="logonPwd" cssClass="text" cssStyle="width: 160px;"/>
```

{% hint style="info" %}
密码框默认不回显示.需要设置属性showPassword="true"
{% endhint %}

4.提交

```MARKDOWN
<s:submit name="submit" value="登录" cssClass="buttoninput"/>
```

5.重置

```MARKDOWN
<s:reset name="reset" value="取消" cssClass="buttoninput"/>
```

注意: struts2中的表单标签，有默认的主题xhtml. 如果不想要添加任何修饰，只需要将主题修改为simple.

##### 怎样设置表单主题

1.全局----在struts.xml文件中配置一个常量

```MARKDOWN
<constant name="struts.ui.theme" value="simple"></constant>
```

2.局部----针对于某一个form.

```MARKDOWN
<s:form theme="simple">
```

3.局部----可以给任意的表单组件去指定theme属性值。

#### 2. 需要使用xml配置方式对数据进行校验

用户名 非空，3-12位
密码  非空 


1.在UserAction所在包下创建一个UserAction-validation.xml
2.在xml文件中添加dtd约束
<!DOCTYPE validators PUBLIC
"-//Apache Struts//XWork Validator 1.0.3//EN"
"http://struts.apache.org/dtds/xwork-validator-1.0.3.dtd">
3.对属性进行校验
<field name="logonName">
<field-validator type="requiredstring">
<message>用户名不能为空</message>
</field-validator>

<field-validator type="stringlength">
<param name="maxLength">12</param>
<param name="minLength">3</param>
<message>用户名长度必须在${minLength}到${maxLength}之间</message>
</field-validator>
</field>
<field name="logonPwd">
<field-validator type="requiredstring">
<message>密码不能为空</message>
</field-validator>
</field>
在页面上通过<s:fielderror>


3.登录成功，将用户存储到session，在页面上显示用户。
top.jsp   ${user.userName } 显示当前登陆用户

### 查询功能--查询全部

登录成功后，跳转到home.jsp页面。home.jsp页面使用了frameset布局。
它的左边(left.jsp)页面使用了dtree,来展示。有一个连接用户管理，
当点击它时，打开了list.jsp页面。

问题:点击用户管理，应该直接打开list.jsp页面？
当点击用户管理时，应该访问UserAction中的list方法，查询出所有的List<User>
在跳转到list.jsp页面，展示出所有用户。


实现:
1.修改left.jsp页面上的用户管理的连接。
原来:d.add(3,2,'用户管理','${pageContext.request.contextPath}/user/list.jsp','','mainFrame');
修改后
d.add(3,2,'用户管理','${pageContext.request.contextPath}/user_list','','mainFrame');

2.在list方法中调用service,dao完成查询操作
得到一个List<User> users;

我们将List<User> users声明成成员变量，提供get/set方法，
这样，集合就会自动的压入到valueStack中.

问题:
1.返回SUCCESS问题   因为我们的配置是使用了通配符，一个配置对应多个请求。
这时，有可能有多种情况都返回SUCCESS,所以我们修改.
登录成功返回  login_success   查询所有成功  list_success.
2.关于查询操作时，校验的配置文件会执行。
原因:我们之前配置文件名称叫  UserAction-validation.xml，它会对UserAction下所有方法校验。
解决:将其修改为   UserAction-user_login-validation.xml 这就只会对user_login进行校验。

3.查询成功跳转到了 list.jsp，在页面上展示信息.

### 添加员工

1.对add.jsp页面上html标签修改----struts2的表单标签
1.性别
原标签
<input type="radio" name="sex" id="sex男" value="男"/><label for="sex男">男</label>
<input type="radio" name="sex" id="sex女" value="女"/><label for="sex女">女</label>
struts2标签:
<s:radio list="{'男','女'}" name="sex" id="sex" value="%{'男'}"/>

2.学历
原标签:
<select name="education" id="education">
<option value=""
selected="selected"
>--选择学历--</option>
<option value="博士">博士</option>
<option value="硕士">硕士</option>
<option value="研究生">研究生</option>
<option value="本科">本科</option>
<option value="专科">专科</option>
<option value="高中">高中</option>
</select>
struts2标签
<s:select list="{'博士','硕士','研究生','本科','专科','高中'}" name="education" id="education" headerKey="" headerValue="--选择学历--"></s:select>
3.兴趣爱好
<s:checkboxlist list="{'看电影','旅游','健身','购物','睡觉'}" name="interest"/>

4.上传
<s:file name="upload" size="30" value="" id="userAction_save_do_upload"/>

5.文本域
<s:textarea name="remark" cols="30" rows="3" id="userAction_save_do_remark" cssStyle="WIDTH: 96%"/>


2.添加数据的校验
在UserAction类所在包下创建一个 UserAction-user_add-validation.xml


3.完成添加操作(上传)	

问题:怎样将要添加的信息在action中获取到?

在UserAction类中有一个  private User user=new User();
我们又声明了
private File upload;
private String uploadContentType;
private String uploadFileName;

添加的用户信息，除了上传文件的信息，其它的都封装到了user对象中。
而上传文件信息在三个属性上封装。


对于我们添加用户还需要有下列信息:
userID----->自动增长
path------->人为指定。

简历不允许被浏览器端直接访问。
d:/upload下.
上传简历，保存时的重名问题.
d:/upload/随机名.
filename=真实名

对于我们上面操作，因为多个action在同一个配置中(使用了通配符).
多个请求操作时，可能都需要跳转到input视图。但是它们跳转的页面
不一样，怎样处理?

可以 通过 @InputConfig注解，改为校验失败后 跳转视图 

### 条件查询

1.在list.jsp页面修改查询组件

是否上传简历
<s:select list="#{'1':'有','2':'无'}" name="isUpload" id="isUpload" headerKey="0" headerValue="--请选择--"></s:select>


2.添加校验

3.完成条件查询操作	
问题:是否上传简历，怎样在action中获取？

需要在User中添加一个属性  String isUpload

在dao中怎样根据条件查询?
1.sql语句生成

2.参数怎样传递?
创建一个List<Object>,在每一次判断时，直接将参数添加到集合中，
最后将集合转换成Object[]，做为参数传递到query方法中。

String sql = "select * from s_user where 1=1 ";
List<Object> params=new ArrayList<Object>();
String username = user.getUserName();
if (username != null && username.trim().length() > 0) {
sql += " and userName like ?";
params.add("%"+username+"%");
}
String sex = user.getSex();
if (sex != null && sex.trim().length() > 0) {
sql += " and sex=?";
params.add(sex);
}
String education = user.getEducation();
if (education != null && education.trim().length() > 0) {
sql += " and education=?";
params.add(education);
}

String isupload = user.getIsUpload();
if ("1".equals(isupload)) {
sql += " and filename is not null";
} else if ("2".equals(isupload)) {
sql += " and filename is null";
}

QueryRunner runner = new QueryRunner(DataSourceUtils.getDataSource());

return runner.query(sql, new BeanListHandler<User>(User.class),params.toArray());

### 5. 员工删除

在list.jsp页面上有删除链接，我们只需要将当前用户的id传递到服务器端
在服务器端根据id删除用户信息。删除完成，在查询一次。	

1.修改list.jsp页面上删除连接
原标签:
<a href="${pageContext.request.contextPath}/user/list.jsp?userID=15">
<img src="${pageContext.request.contextPath}/images/i_del.gif" width="16" height="16" border="0" style="CURSOR: hand">
</a>
使用struts2标签修改
第一种方式
<s:a href="路径">
第二种方式
<s:a action="" namespace="">
<s:param name="" value="">
</s:a>
第三种方式:
<s:url>标签来定义一个路径
<s:a href="url">来导入url值。

<s:url namespace="/" action="user_del" var="delUrl">
<s:param name="id" value="%{#u.userID}"/>
</s:url>

<s:a href="%{#delUrl}">
<img src="${pageContext.request.contextPath}/images/i_del.gif" width="16" height="16" border="0" style="CURSOR: hand">
</s:a>	


2.完成删除操作

问题:如果用户有简历，删除用户时，也要将简历删除。
1.先查询出用户。
2.判断user.getPath()!=null
new File(user.getPath()).delete();

### 6.员工详细信息查看

在list.jsp页面，有查看客户详细信息连接。
<s:url namespace="/" action="user_findById" var="findByIdUrl">
<s:param name="userID" value="%{#u.userID}"/>
</s:url>
<s:a href="%{#findByIdUrl}">
<img src="${pageContext.request.contextPath}/images/button_view.gif" border="0" style="CURSOR: hand">
</s:a>

查询出用户信息(user),需要在view.jsp页面展示 

在页面上展示时，我们不能使用valueStack栈顶的user对象，而要
使用压入的action的getModel方法，重新得到user对象去获取信息。

### 7. 员工简历下载

在view.jsp页面上展示的员工信息包含了简历，它是一个连接，点击它，实现员工简历下载。

在<result name="download_success" type="stream">
<param name="contentType">${contentType}</param>
<param name="contentDisposition">attachment;filename=${downloadFilename}</param>
<param name="inputStream">${inputStream}</param>
</result>

### 8. 员工信息修改

1.查询
<a href="${pageContext.request.contextPath}/user/edit.jsp?userID=15">
<img src="${pageContext.request.contextPath}/images/i_edit.gif" border="0" style="CURSOR: hand">
</a>
<s:url namespace="/" action="user_updateForFind" var="editUrl">
<s:param name="userID" value="%{#u.userID}"/>
</s:url>

<s:a href="%{#editUrl}">
<img src="${pageContext.request.contextPath}/images/i_edit.gif" border="0" style="CURSOR: hand">
</s:a>

查询出user对象，跳转到edit.jsp页面，展示用户信息。	

2.修改
修改就是一个上传操作:
问题:在修改时，关于用户简历的处理?
1.原来没有  修改也没有。---不管
2.原来没有  修改有了    ----处理
3.原来有    修改没有了 -----不管
4.原来有    修改也有.  -----处理(将旧的删除)

修改前必须先查询出用户。

### 9. 登陆校验拦截器

功能:用户只有登录成功后，才可以进行操作.

做一个Interceptor.判断用户是否登录了。（登录标志session中有user）

自定义Interceptor步骤:
1.创建一个类，实现Interceptor接口
2.重写方法完成功能
3.在struts.xml文件注册
4.在action中引入

### 10.struts2 提供的异常处理

对于action中的操作，出现问题，直接抛出自定义异常。
在struts.xml文件中
<global-exception-mappings>
<exception-mapping result="login"
exception="cn.itcast.user.exception.FindByIdException"></exception-mapping>
</global-exception-mappings>
这就可以让特定的异常，跳转到自定的页面。


原理:
struts2,默认加载的18个拦截器的第一个是exception这个拦截器，它没有做任何操作，
直接放行，，只是它将 invocation.invoke()操作使用try-catch进行了处理。

其它的拦截器，或是action只要向外抛出异常，exception拦截器就会将其捕获。
