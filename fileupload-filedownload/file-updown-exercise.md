# 文件的上传下载练习

网盘系统

![](../.gitbook/assets/2020-03-01-15-47-27.png)

## 准备数据库

```text
    create database day22

    create table resources(
      id int primary key auto_increment,
      uuidname varchar(100) unique not null,
      realname varchar(40) not null,
      savepath varchar(100) not null,
      uploadtime timestamp ,
      description varchar(255)
    );
```

## 导入jar包

c3p0 c3p0-config.xml

dbutils mysql驱动

commons-fileupload-1.2.1.jar 文件上传

commons-io-1.4.jar 它是提供的io工具

jstl standard

beanutils collections logging

## 编码实现上传

* 在index.jsp页面添加上传链接

```java
<a href='${pageContext.request.contextPath}/upload.jsp'>上传</a>
```

* JavaBean

```java
private int id;
private String uuidname;
private String realname;
private String savepath;
private Timestamp uploadtime;
private String description;

...getter and setter...
```

* 创建upload.jsp页面

上传操作浏览器端三个注意事项:

1. method = post
2. encType = "multipart/form-data"
3. 要使用 &lt;input type="file" name='f'&gt;

```java
<form action="${pageContext.request.contextPath}/upload" method="post" enctype="multipart/form-data">
      <input type="file" name="f"><br>
  描述:<input type="text" name="description"><br>
      <input type="submit" value="提交">
</form>
```

* 创建UploadServlet
* 完成上传操作
* 将数据封装，存储到数据库。

```text
  1.上传操作
    commons-fileupload.jar
      DiskFileItemFactory ---确定缓存大小，临时文件位置
      ServletFileUpload  ---parseRequest方法
      FileItem ---isFormField 判断是否为上传组件
    中文乱码
      upload.setHeaderEncoding("utf-8");---文件名
      FileItem.getString("utf-8")--非上传组件中文内容

  2.生成UUID文件名和多级存储目录结构 mkdirs

  3.IOUtils.copy(item.getInputStream(), new FileOutputStream(new File(目录，uuid文件名)));

  4.将数据封装，存储到数据库.
    问题:怎样将数据封装到javaBean？
        手动创建一个Map<String,String[]>将数据封装到map集合，通过BeanUtils完成数据封装.
        map.put(jBea属性名,new String[]{对应数据});
        BeanUtils.populate(r,map);
```

* 在数据库添加信息

id 由数据库控制自动增加。

TIMESTAMP 时间戳，数据库自动生成。

```java
public void save(Resource r) throws SQLException {

    String sql = "insert into resources values(null,?,?,?,null,?)";
    QueryRunner runner = new QueryRunner(DataSourceUtils.getDataSource());

    runner.update(sql, r.getUuidname(), r.getRealname(), r.getSavepath(),
    r.getDescription());
}
```

## 编码实现下载

* index.jsp页面代码

```java
<a href="${pageContext.request.contextPath}/showDownload">下载</a>
```

* 创建ShowDownloadServlet

在这个servlet中，查看db,得到所有可以下载的信息.

```java
List<Resource> rs = service.findAll();
request.setAttribute("rs",rs);
`
```

* 创建一个download.jsp页面，展示所有可以下载的信息.

```java
<c:forEach items="${rs}" var="r">
    <tr>
        <td>${r.realname}</td>
        <td>${r.description }</td>
        <td><a href=''>下载</a></td>
    </tr>
</c:forEach>
```

在download.jsp，点击下载时，传递的是要下载文件的id。

```java
<a href='${pageContext.request.contextPath}/download?id=${r.id}'>下载</a>
```

* 创建一个DownloadServlet
* 根据request.getParamter("id") 查询数据库，得到要下载的文件的相关信息
* 下载操作

设置mimeType

```java
response.setContentType(getServletContext.getMimeType(String filename));
```

设置响应头，目的是永远是下载操作

```java
//下载时中文文件名
//filename根据user-agent encode
response.setHeader("content-disposition", "attachment;filename="+filename);
...
```

参考[根据浏览器编码文件名](file-download.md)

将下载的文件通过resposne.getOutputStream()流写回到浏览器端。

```java
byte[] b = FileUtils.readFileToByteArray(file); // 将指定文件读取到byte[]数组中.
response.getOutputStream().write(b);
```

