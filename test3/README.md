# 实验3：创建分区表

## 实验目的：

掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 实验内容：
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用**你自己的账号创建本实验的表**，表创建在上述3个分区，自定义分区策略。
- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。

## 实验过程

使用帐号LTT创建表orders：

```sql
SQL>CREATE TABLE LTT.orders
(
order_id VARCHAR2(40 BYTE)NOT NULL
,customer_name VARCHAR2(40 BYTE)NOT NULL
,customer_tel VARCHAR2(40 BYTE)NOT NULL
,order_date DATE NOT NULL
,employee_id NUMBER(6,0)NOT NULL 
,discount NUMBER(8,2)DEFAULT 0 
,trade_receivable NUMBER(8,2)DEFAULT 0 
)
LOGGING
TABLESPACE "USERS"
PCTFREE 10
INITRANS 1
STORAGE
( 
INITIAL 65536 
NEXT 1048576 
MINEXTENTS 1 
MAXEXTENTS 2147483645 
BUFFER_POOL DEFAULT
);
```

创建order_details表的部分语句如下：
```sql
SQL>CREATE TABLE LTT.order_details
(
, order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL 
, product_num NUMBER(8, 2) NOT NULL 
, product_price NUMBER(8, 2) NOT NULL 
)
LOGGING
TABLESPACE "USERS"
PCTFREE 10
INITRANS 1
STORAGE
( 
 INITIAL 65536 
 NEXT 1048576 
 MINEXTENTS 1 
 MAXEXTENTS 2147483645 
 BUFFER_POOL DEFAULT
);
```


在用户中创建分区表orders，按订单日期分区：

```sql
SQL>CREATE TABLE LTT.orders 
(
 order_id NUMBER(10,0)NOT NULL 
 ,customer_name VARCHAR2(40 BYTE)NOT NULL 
 ,customer_tel VARCHAR2(40 BYTE)NOT NULL 
 ,order_date DATE NOT NULL 
 ,employee_id NUMBER(6,0)NOT NULL 
 ,discount NUMBER(8,2)DEFAULT 0 
 ,trade_receivable NUMBER(8,2)DEFAULT 0 
)
TABLESPACE USERS 
PCTFREE 10 
INITRANS 1 
STORAGE 
( 
 BUFFER_POOL DEFAULT 
)
PARTITION BY RANGE (order_date)
(
 PARTITION partition_before_2016 VALUES LESS THAN (
  TO_DATE('01-01-2016', 'MM-DD-YYYY'))TABLESPACE USERS,
  PARTITION partition_before_2017 VALUES LESS THAN (
  TO_DATE('01-01-2017', 'MM-DD-YYYY'))TABLESPACE USERS02 
);
```




