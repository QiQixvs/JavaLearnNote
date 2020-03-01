# 文件下载扩展: 使用队列来优化递归操作

使用队列来优化递归操作: 可以解决目录层次过多问题

递归操作可以理解成是纵向的遍历，如果目录层次比较多，在内存中存储的数据也多，会引起溢出。
使用队列，它是横向遍历，一层一层遍历，可以解决目录层次比较多问题。使用队列，最多时候在内存中只存储了一层的信息。

## 使用递归来下载指定目录下所有文件

```java
<%!//声明一个方法
    public void getFile(File file) {
        if (file.isDirectory()) {//是目录
            File[] fs = file.listFiles();

            for (int i = 0; i < fs.length; i++) {
                getFile(fs[i]); //递归调用
            }

        } else if (file.isFile()) {//是文件
            ...
}
}%>

<%
String path = "D:\\java1110\\workspace\\day22_2\\WebRoot\\upload";
File uploadDirectory = new File(path);
getFile(uploadDirectory);
%>
```

## 使用队列来下载指定目录下所有文件

队列特点: 先进先出

在jdk中有一个接口**Queue**，它有一个实现类叫**LinkedList**，它其实就是一个队列。

使用队列

* 插入 offer  
* 获取使用 poll

```java
<%
String path = "D:\\java1110\\workspace\\day22_2\\WebRoot\\upload";
File uploadDirectory = new File(path);

//创建一个队列
Queue<File> queue = new LinkedList<File>();
queue.offer(uploadDirectory);

while (!queue.isEmpty()) { //如果队列不为空
    File f = queue.poll(); //从队列中获取一个File

    if(f.isDirectory()){
    //是目录,将目录下所有文件遍历出来，存储到队列中
        File[] fs = f.listFiles();

        for (int i = 0; i < fs.length; i++) {
            queue.offer(fs[i]);
        }

    }else{
        //获得文件所在绝对路径
        String absolutePath=(f.getAbsolutePath());
        String p = absolutePath.substring(absolutePath.lastIndexOf("\\upload"));

        out.println("<a href='/day22_2"+p+"'>"+f.getName()+"</a><br>");
    }
}
%>
```

最常用的就是树型结构。
