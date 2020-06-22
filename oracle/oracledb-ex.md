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

第三题 统计每年入职的员工个数

```text

```

## rownum

伪列 行号

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

SQL>  select *
  2   from 	(select rownum r,e1.*
  3  	 from (select * from emp order by sal) e1
  4   	 where rownum <=8
  5  	)
  6   where r >=5;

         R      EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO
---------- ---------- ---------- --------- ---------- -------------- ---------- ---------- ----------
         5       7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30
         6       7934 MILLER     CLERK           7782 23-1月 -82           1300                    10
         7       7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30
         8       7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30

SQL> --临时表
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

SQL>
SQL> create global temporary table test2
  2  (tid number,
  3   tname varchar2(20))
  4  on commit reserve rows;
on commit reserve rows
          *
第 4 行出现错误:
ORA-00901: 无效 CREATE 命令


SQL> create global temporary table test2
  2  (tid number,
  3   tname varchar2(20))
  4  on commit preserve rows;

表已创建。

SQL> insert into test2 values(1,'Tom');

已创建 1 行。

SQL> commit;

提交完成。

SQL> select * from test2;

       TID TNAME
---------- --------------------
         1 Tom

SQL> exit
SQL> host cls

SQL> --第二题 找出员工表中薪水大于本部门平均薪水的员工
SQL> select e.empno,e.ename,e.sal,d.avgsal
  2  from emp e, (select deptno,avg(sal) avgsal from emp group by deptno) d
  3  where e.deptno=d.deptno and e.sal> d.avgsal;

     EMPNO ENAME             SAL     AVGSAL
---------- ---------- ---------- ----------
      7698 BLAKE            2850 1566.66667
      7499 ALLEN            1600 1566.66667
      7902 FORD             3000       2175
      7788 SCOTT            3000       2175
      7566 JONES            2975       2175
      7839 KING             5000 2916.66667

已选择 6 行。

SQL> --相关子查询：将主查询的值作为参数传递给子查询
SQL> select empno,ename,sal,(select avg(sal) from emp where deptno=e.deptno) avgsal
  2  from emp e
  3  where sal > (select avg(sal) from emp where deptno=e.deptno);

     EMPNO ENAME             SAL     AVGSAL
---------- ---------- ---------- ----------
      7499 ALLEN            1600 1566.66667
      7566 JONES            2975       2175
      7698 BLAKE            2850 1566.66667
      7788 SCOTT            3000       2175
      7839 KING             5000 2916.66667
      7902 FORD             3000       2175

已选择 6 行。

SQL> host cls

SQL> --第三题 统计每年入职的员工个数
SQL> select hiredate from emp;

HIREDATE
--------------
17-12月-80
20-2月 -81
22-2月 -81
02-4月 -81
28-9月 -81
01-5月 -81
09-6月 -81
19-4月 -87
17-11月-81
08-9月 -81
23-5月 -87

HIREDATE
--------------
03-12月-81
03-12月-81
23-1月 -82

已选择 14 行。

SQL> /*
SQL> select count(*) Total,
SQL>
SQL>        sum(if 是81年 then +1 else +0) "1981",
SQL> from emp;
SQL>
SQL> HIREDATE       count81:=0
SQL> --------------------------
SQL> 17-12月-80           0
SQL> 20-2月 -81           1
SQL> 22-2月 -81           1
SQL> 02-4月 -81           1
SQL> 28-9月 -81           1
SQL> 01-5月 -81           1
SQL> 09-6月 -81           1
SQL> 19-4月 -87           0
SQL> 17-11月-81           1
SQL> 08-9月 -81           1
SQL> 23-5月 -87           0
SQL> 03-12月-81           1
SQL> 03-12月-81           1
SQL> 23-1月 -82           0
SQL> -----------------------------
SQL>                     10
SQL> */
SQL> host cls

SQL> --行转列
SQL> -- wm_concat(字符串)  组函数
SQL> select deptno,wm_concat(ename) nameslist
  2  from emp
  3  group by deptno;

    DEPTNO
----------
NAMESLIST
--------------------------------------------------------------------------------
        10
CLARK,KING,MILLER

        20
SMITH,FORD,ADAMS,SCOTT,JONES

        30
ALLEN,BLAKE,MARTIN,TURNER,JAMES,WARD


SQL> col NAMESLIST for a60
SQL> select deptno,wm_concat(ename) nameslist
  2  from emp
  3  group by deptno;

    DEPTNO NAMESLIST
---------- ------------------------------------------------------------
        10 CLARK,KING,MILLER
        20 SMITH,FORD,ADAMS,SCOTT,JONES
        30 ALLEN,BLAKE,MARTIN,TURNER,JAMES,WARD

SQL> spool off
```
