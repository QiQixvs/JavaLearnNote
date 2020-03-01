# 文件的下载

文件下载的方式

1. 超链接下载
2. 服务器端通过流下载\(服务器端编程\)

## 超链接下载

download1.jsp

```text
<a href='${pageContext.request.contextPath}/upload/a.bmp'>a.bmp</a><br>
<a href='${pageContext.request.contextPath}/upload/a.doc'>a.doc</a><br>
<a href='${pageContext.request.contextPath}/upload/a.txt'>a.txt</a><br>
<a href='${pageContext.request.contextPath}/upload/tk.mp3'>tk.mp3</a><br>
```

如果文件可以直接被浏览器解析，那么会在浏览器中直接打开不能被浏览器直接解析，就是下载操作。直接打开的要想下载 ，右键另存为。

超连接下载，要求下载的资源，必须是可以直接被浏览器直接访问的。客户端访问服务器静态资源文件时，静态资源文件是通过缺省Servlet返回的，在tomcat配置文件conf/web.xml 找到 --- org.apache.catalina.servlets.DefaultServlet

## 在服务器端编程完成下载

### 创建download.jsp

```text
<a href='${pageContext.request.contextPath}/download?filename=a.bmp'>a.bmp</a><br>
<a href='${pageContext.request.contextPath}/download?filename=a.doc'>a.doc</a><br>
<a href='${pageContext.request.contextPath}/download?filename=a.txt'>a.txt</a><br>
<a href='${pageContext.request.contextPath}/download?filename=tk.mp3'>tk.mp3</a><br>
`
```

### 创建DownloadServlet

```java
@WebServlet(name="DownloadServlet",urlPatterns={"/download"})
...
// 1.得到要下载的文件名称
String filename = request.getParameter("filename");

//2.判断文件是否存在
File file = new File("d:/upload/" + filename);
if (file.exists()){
    //3.进行下载
}
```

### 进行下载

**原理:** 就是通过response获取一个输出流，将要下载的文件内容写回到浏览器端就可以了.

```java
FileInputStream fis = new FileInputStream(file); // 读取要下载文件的内容
OutputStream os = response.getOutputStream(); // 将要下载的文件内容通过输出流写回到浏览器端.
int len = -1;
byte[] b = new byte[1024 * 100];

while ((len = fis.read(b)) != -1) {
   os.write(b, 0, len);
   os.flush();
}
os.close();
fis.close();
```

{% hint style="danger" %}
不同类型的文件，写回到浏览器端解析方式不同。
{% endhint %}

注意: 要想通过编程的方式，实现文件下载，在进行读写操作之前，

* 要设置mimetype类型

```java
resposne.setContextType(String mimeType);
```

问题:怎样可以得到要下载文件的mimeType类型?

```java
ServletContext.getMimeType(String filename);
```

如果设置了mimeType,浏览器能解析的就直接展示了，不能解析的，直接下载.

* 设置一个响应头，设置后的效果，就是无论返回的是否可以被浏览器解析，就是下载。

```java
response.setHeader("content-disposition","attachment;filename=下载文件名称");
```

### 服务器端编程下载

1. 将下载的文件通过resposne.getOutputStream\(\)流写回到浏览器端。
2. 设置mimeType  response.setContentType\(getServletContext.getMimeType\(String filename\)\);
3. 设置响应头，目的是永远是下载操作

   response.setHeader\("content-disposition","attachment;filename=下载文件名称"\);

## 文件下载时的乱码问题

### 下载时中文名称资源查找不到

原因: 超链接默认get请求，中文参数传递到服务器端是乱码

```text
<a href='${pageContext.request.contextPath}/download?filename=天空.mp3'>天空.mp3</a>
```

在服务器端:

```java
String filename = request.getParameter("filename");

new String(filename.getBytes("iso8859-1"),"utf-8");
```

### 下载文件显示时的中文乱码问题

```java
response.setHeader("content-disposition", "attachment;filename="+filename);
```

* Edge浏览器，关键字Edge。使用utf-8对文件名编码。URLEncoder.encode\(fileName,“UTF8”\);
* Firefox 可以使用Firefox区分, 用ISO编码的中文输出。 new String\(fileName.getBytes\(“UTF-8”\), “ISO8859-1”\);
* chrome 用ISO编码的中文输出。 new String\(fileName.getBytes\(\), “ISO8859-1”\)

怎样判断浏览器?

```java
String agent = request.getHeader("user-agent");
```

**Edge**：Mozilla/5.0 \(Windows NT 10.0; Win64; x64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362

**Firefox**：Mozilla/5.0 \(Windows NT 10.0; Win64; x64; rv:73.0\) Gecko/20100101 Firefox/73.0

**Chrome**：Mozilla/5.0 \(Windows NT 10.0; Win64; x64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/80.0.3987.122 Safari/537.36

```java
if (agent.contains("Edge")) {
// Edge浏览器
filename = URLEncoder.encode(filename, "utf-8");

} else{
//其他浏览器
filename = new String(fileName.getBytes(“UTF-8”), “ISO8859-1”);

}
response.setHeader("content-disposition", "attachment;filename="+filename);
 ...
```
