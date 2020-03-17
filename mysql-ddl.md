# MySQL数据库操作-DCL

## 1. 创建用户

语法：
CREATE USER 用户名@地址 IDENTIFIED BY '密码';
CREATE USER user1@localhost IDENTIFIED BY ‘123’;
CREATE USER user2@’%’ IDENTIFIED BY ‘123’;

## 2.给用户授权

　　语法：
GRANT 权限1, … , 权限n ON 数据库.* TO 用户名@IP
GRANT CREATE,ALTER,DROP,INSERT,UPDATE,DELETE,SELECT ON mydb1.* TO user1@localhost;
GRANT ALL ON mydb1.* TO user2@’%’;

## 3. 撤销授权

语法：
REVOKE权限1, … , 权限n ON 数据库.* FORM 用户名
REVOKE CREATE,ALTER,DROP ON mydb1.* FROM user1@localhost;

## 4.查看用户权限

语法：
SHOW GRANTS FOR 用户名
SHOW GRANTS FOR user1@localhost;

## 5. 删除用户

语法：
DROP USER 用户名
DROP USER user1@localhost;

## 6.　修改用户密码

语法：
Use mysql;
UPDATE USER SET PASSWORD=PASSWORD(‘密码’) WHERE User=’用户名’;
FLUSH PRIVILEGES;
UPDATE USER SET PASSWORD=PASSWORD('1234') WHERE User='user2';
FLUSH PRIVILEGES;
