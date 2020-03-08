# 权限管理系统

## 1. 系统需求

### 1.1 权限管理

* 创建权限
* 查看权限

### 1.2 角色管理

* 创建角色
* 查看角色
* 为角色授予权限

### 1.3 用户管理

* 增加用户
* 用户登陆
* 为用户授予角色

## 2. 需求分析

1. 系统功能和权限 ----- 一对一的关系
2. 角色和权限 ------- 多对多的关系
3. 用户和角色 ---- 多对多关系

## 3. 系统设计

### 3.1 数据库设计

根据实体和属性 设计实体表

一个实体一般情况下对应一张表

```markdown
create table users(
   id int primary key auto_increment,
   username varchar(20) unique not null,
   password varchar(20) not null,
   nickname varchar(20) not null
);

create table roles(
   id int primary key auto_increment,
   name varchar(20) unique not null,
   description varchar(200)
);

create table privileges(
   id int primary key auto_increment,
   name varchar(20) unique not null,
   fnpath varchar(100) unique not null,
   description varchar(200)
);
```

再根据关系 建立表之间联系

* 一对多：在多的一方添加一的一方主键作为外键
* 多对多：必须创建第三张关系表，保存两方主键作为外键
* 一对一：在任何一方 添加另一方主键作为外键

```markdown
create table roleprivilege(
   role_id int not null,
   privilege_id int not null,
   foreign key(role_id) references roles(id),
   foreign key(privilege_id) references privileges(id),
   primary key(role_id,privilege_id)//联合主键
);

create table userrole(
  user_id int not null,
  role_id int not null,
  foreign key(user_id) references users(id),
  foreign key(role_id) references roles(id),
  primary key(user_id,role_id)
);
```

{% hint style="info" %}
联合主键，都相同才算重复。
{% endhint %}

### 3.2 web工程环境搭建

建立工程包结构

导入jar包  BeanUtils 、DBUtils、C3P0、MySQL驱动 、JSTL

实体类 --- 每个表对应一个实体类

## 4. 系统功能实现

### 实现系统功能  ----- 权限管理

1) 权限添加 --- insert 操作
2) 权限查看 --- select 操作

### 实现系统功能 ---- 角色管理

1) 角色添加 ---- insert操作
2) 角色查看 ---- select操作

### 实现系统功能 --- 用户管理

1) 创建用户 ---- insert 操作
2) 查看用户 ---- select操作
alter table users modify password varchar(40) not null;

### 实现系统功能 --- 将权限授予角色、将角色授予用户

在角色权限表 和 用户角色表中，设置联合主键，目的防止对用户和角色 重复授权

1) 在角色查看列表中，添加为角色授权 操作
先根据角色查询对应权限，查询权限查出权限的名称！ ----------------- select * from roleprivilege,privileges where roleprivilege.privilege_id = privileges.id ;
 select * from roleprivilege,privileges where roleprivilege.privilege_id = privileges.id and role_id = ?;

2) 将角色授予用户
在用户列表中，添加为用户授予角色链接
查询用户角色时，同时查询出角色名称
select * from userrole,roles where userrole.role_id = roles.id and userrole.user_id = ?; 

### 实现系统功能 --- 用户登录，为用户添加权限控制

1) 用户登录
在登陆后，用户信息保存Session中，通过Session中用户信息，控制菜单的显示 
insert into users values(null,'admin',md5('admin'),'超级管理员'); ------- 添加内置账户，账户具有所有权限，控制权限系统
编写系统菜单
如果未登录 ---- 登陆
如果已经登陆（普通用户） ---- 功能菜单
如果已经登陆（超级管理员） ---- 权限管理菜单

2) 为用户的访问添加权限控制
获得访问者请求资源路径
根据路径查询需要权限 ----------- select * from privileges where fnpath = ?	
获得登陆用户信息，根据用户查找具有哪些权限 ? ---- 先查询userrole 获得当前用户角色，再查询roleprivilege获得角色权限 ，再查询privileges
select privileges.* from userrole,roleprivilege,privileges where userrole.role_id =roleprivilege.role_id and roleprivilege.privilege_id = privileges.id and userrole.user_id = ? ;

