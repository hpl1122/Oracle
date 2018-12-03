## 实验5报告文档

### 1.实验内容：
 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。
 通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
 我的用户为“LTT”。

### 2.实验过程：

- 创建一个包(Package)，包名是MyPack。

```sql
create or replace PACKAGE MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END MyPack;
```

- 在MyPack中创建一个函数SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销售额之和。   函数SaleAmount要求输入的参数是部门号，输出部门的销售金额。

```sql
create or replace PACKAGE BODY MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
    N NUMBER(20,2);
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;
```

- 在MyPack中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的ID,姓名，销售总金额。信息用   dbms_output包中的put或者put_line函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。

```sql
PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                             V.EMPLOYEE_ID||' '||v.NAME);
      END LOOP;
    END;
END MyPack;
```

- 由于订单只是按日期分区的，上述统计是全表搜索，因此统计速度会比较慢，如何提高统计的速度呢？

  1）扩大数据表空间到500M，用于存放本系统的数据；
  2）段盘区的初始大小为10K，增长大小为10K，增长幅度为1；
  3）用户临时空间增大40M；
  4）系统临时表空间和回滚段表空间增大40M，并且新建4个回滚段；
  5）需要经常联结查询，而且数据量又大的库存表、名录表、收发料表放在一簇内；
  6）提供定时备份，备份文件放在另外的机器上。
    

## 实验总结
   通过本次试验熟悉了循环语句的使用方法；
   学习了条件语句的使用方法，分支语句的使用方法；
   了解了PL/SQL语言结构，变量和常量的声明和使用方法；
   掌握了常用的PL/SQL函数，包，过程的用法。


