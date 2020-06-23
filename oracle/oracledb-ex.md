# oracle-练习

##

第一题：找出员工表中工资最高的前三名

```text
SQL> select ROWNUM, EMPNO, ENAME, SAL from (select * from emp order by sal desc)
  2  where ROWNUM <4;
```

第二题 找到员工表中薪水大于本部门平均新书的员工

```text
SQL> select e.empno, e.ename, e.sal, d.avgsal
  2  from emp e, (select AVG(sal) avgsal, deptno from emp group by deptno) d
  3  where e.deptno = d.deptno and e.sal>d.avgsal;
```

第三题 统计每年入职的员工个数(不使用子查询)

```text
SQL> select count(*) Total,
  2  sum(decode(to_char(hiredate,'yyyy'),'1980',1,0)) "1980",
  3  sum(decode(to_char(hiredate,'yyyy'),'1981',1,0)) "1981",
  4  sum(decode(to_char(hiredate,'yyyy'),'1982',1,0)) "1982",
  5  sum(decode(to_char(hiredate,'yyyy'),'1983',1,0)) "1983"
  6  from emp;
     TOTAL       1980       1981       1982       1983
---------- ---------- ---------- ---------- ----------
        14          1         10          1          0
```

## rownum

伪列，行号，从 1 开始

1. rownum 永远按照默认的顺序生成,维持排序前的原行号
2. rownum 只能使用< <=;不能使用> >=

```text
SQL> select rownum,empno,ename,sal from emp;

    ROWNUM      EMPNO ENAME             SAL
---------- ---------- ---------- ----------
         1       7369 SMITH             800
         2       7499 ALLEN            1600
...
SQL> select rownum,empno,ename,sal
  2  from emp
  3  where rownum<=3
  4  order by sal desc;

    ROWNUM      EMPNO ENAME             SAL
---------- ---------- ---------- ----------
         2       7499 ALLEN            1600
         3       7521 WARD             1250
         1       7369 SMITH             800

SQL> select rownum,empno,ename,sal
  2  from (select * from emp order by sal desc)
  3  where rownum<=3;

    ROWNUM      EMPNO ENAME             SAL
---------- ---------- ---------- ----------
         1       7839 KING             5000
         2       7788 SCOTT            3000
         3       7902 FORD             3000
===================================================

SQL> -- 2. rownum只能使用< <=;不能使用> >=
SQL> select rownum,empno,ename,sal from emp where rownum>=5 and rownum<=8;

未选定行

SQL> select rownum,empno,ename,sal from emp where rownum>=5;

未选定行

SQL> select rownum,empno,ename,sal from emp where rownum<=3;

    ROWNUM      EMPNO ENAME             SAL
---------- ---------- ---------- ----------
         1       7369 SMITH            800
         2       7499 ALLEN            1600
         3       7521 WARD             1250
=====================================================

查5-8之间
SQL>  select *
  2   from 	(select rownum r,e1.*
  3  	 from (select * from emp order by sal) e1
  4   	 where rownum <=8
  5  	)
  6   where r >=5;

```

## 临时表

```text
SQL> --create global temporary table ******
SQL> --特点：当事务或者会话结束的时候，表中的数据自动删除
SQL> create global temporary table test1
  2  (tid number,
  3   tname varchar2(20))
  4  on commit delete rows;

表已创建。

SQL> insert into test1 values(1,'Tom');

已创建 1 行。

SQL> select * from test1;

       TID TNAME
---------- --------------------
         1 Tom

SQL> commit;

提交完成。

SQL> select * from test1;

未选定行

SQL> create global temporary table test2
  2  (tid number,
  3   tname varchar2(20))
  4  on commit reserve rows;
on commit preserve rows

表已创建。

SQL> insert into test2 values(1,'Tom');

已创建 1 行。

SQL> commit;

提交完成。

SQL> select * from test2;

       TID TNAME
---------- --------------------
         1 Tom
```

## 行转列

wm_concat(字符串) 组函数

```text
SQL> select deptno,wm_concat(ename) nameslist
  2  from emp
  3  group by deptno;

    DEPTNO    NAMESLIST
---------    ----------------------------------
        10    CLARK,KING,MILLER
        20    SMITH,FORD,ADAMS,SCOTT,JONES
        30    ALLEN,BLAKE,MARTIN,TURNER,JAMES,WARD
```
