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

在LTT用户中创建分区表使用帐号LTT创建表order_details，按订单日期分区:
```sql
CREATE TABLE LTT.order_details 
(
id NUMBER(10, 0) NOT NULL 
, order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL 
, product_num NUMBER(8, 2) NOT NULL 
, product_price NUMBER(8, 2) NOT NULL 
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
REFERENCES orders  (  order_id   )
ENABLE 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1)
(
PARTITION PARTITION_BEFORE_2016 
NOLOGGING 
TABLESPACE USERS 
) 
NOCOMPRESS NO INMEMORY, 
PARTITION PARTITION_BEFORE_2017 
NOLOGGING 
TABLESPACE USERS02
) 
NOCOMPRESS NO INMEMORY  
);
```

插入数据的语句:
```sql
INSERT INTO LTT.orders (order_id,customer_name ,customer_tel ,order_date ,employee_id,discount)
     SELECT 'test1','test1','test1'...,20 FROM DUAL
     UNION ALL SELECT 'test2','test2','test2'...,30 FROM DUAL
     UNION ALL SELECT 'test3','test3','test3'...,40 FROM DUAL
```
##  实验总结：
在实验过程中,以ltt的用户身份进入数据库，使用命令 ls -lh 可以查看3个表空间：

 USERS,USERS02,USERS03，创建两张表：订单表(orders)与订单详表(order_details)。
 
 使用该帐号创建分区表orders和order_details，按订单日期分区。
 
 分区的优点：增强可用性：如果表的一个分区由于系统故障而不能使用，表的其余好的分区仍然可以使用； 
 减少关闭时间：如果系统故障只影响表的一部分分区那么只有这部分分区需要修复，故能比整
 个大表修复花的时间更少；  维护轻松：如果需要重建表，独立管理每个分区比管理单个大表
 要轻松得多；  均衡I / O :      可以把表的不同分区分配到不同的磁盘来平衡I / O 改善
 性能； 改善性能：对大表的查询、增加、修改等操作可以分解到表的不同分区来并行执行，可
 使运行速度      更快； 分区对用户透明，最终用户感觉不到分区的存在。


