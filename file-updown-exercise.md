# 文件的上传下载练习

## 网盘系统

![](2020-03-01-15-47-27.png)

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



