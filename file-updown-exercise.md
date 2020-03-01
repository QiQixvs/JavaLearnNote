# 文件的上传下载练习

## 网盘系统

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

    看图
```

扩展:使用队列来优化递归操作.

```text
队列特点:先进先出.

在jdk中有一个接口Queue 它有一个实现类叫LinkedList它其时就是一个队列。

如果要使用队列，插入 offer  获取使用 poll


使用队列来优化递归操作:是可以解决目录层次过多问题。
    因为:递归操作可以理解成是纵向的遍历，如果目录层次比较多，在内存中存储的数据也多，会引起溢出。
    使用队列，它是横向遍历，一层一层遍历，可以解决目录层次比较多问题。
    因为使用队列，最多时候在内存中只存储了一层的信息。

最常用的就是树型结构。
```

