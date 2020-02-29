---
description: 关于设置encType = "multipart/form-data" 的效果
---

# enctype详解

## 关于设置encType = "multipart/form-data" 的效果

```text
<form action="${ pageContext.request.contextPath}/upload" method="post" encType = "multipart/form-data">
```

## 不设置enctype属性的情况

```text
<form action="${ pageContext.request.contextPath}/upload" method="post">
    <input type="text" name="content"><br>
    <input type="file"name="f"><br>
    <input type="submit" value="上传">
</form>
```

请求头中Content-Type：application/x-www-form-urlencoded（默认）

![&#x8BF7;&#x6C42;&#x6B63;&#x6587;](.gitbook/assets/2020-02-29-18-21-33.png)

## 设置enctype属性的情况

```text
<form action="${ pageContext.request.contextPath}/upload" method="post" encType = "multipart/form-data">
    <input type="text" name="content"><br>
    <input type="file"name="f"><br>
    <input type="submit" value="上传">
</form>
```

![&#x8BF7;&#x6C42;&#x5934;](.gitbook/assets/2020-02-29-18-25-21.png)

![&#x8BF7;&#x6C42;&#x6B63;&#x6587;](.gitbook/assets/2020-02-29-18-26-09.png)

