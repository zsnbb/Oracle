
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

## 创建orders表和order_drtails表

创建orders表的语句如下：
```sql
CREATE TABLE ORDERS
(
order_id NUMBER(10,0)NOT NULL,
customer_name VARCHAR2(40 BYTE)NOT NULL,
customer_tel VARCHAR2(40 BYTE)NOT NULL,
order_date DATE NOT NULL,
employee_id NUMBER(6,0) NOT NULL,
discount NUMBER(8,2)DEFAULT 0,
trade_receivable NUMBER(8,2)DEFAULT 0
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
TO_DATE(' 2016-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS,

PARTITION partition_before_2017 VALUES LESS THAN (
TO_DATE(' 2017-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS02,

PARTITION partition_before_2018 VALUES LESS THAN (
TO_DATE(' 2018-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS03
);
alter table orders add constraint order_details_fk1 primary key (order_id);
```

创建order_details表的语句如下：
```sql
CREATE TABLE order_details
(
id NUMBER(10,0)NOT NULL,
order_id NUMBER(10,0)NOT NULL,
product_id VARCHAR2(40 BYTE)NOT NULL,
product_num NUMBER(8,2) NOT NULL,
product_price NUMBER(8,2) NOT NULL,
CONSTRAINT order_details_fk1 FOREIGN KEY (order_id)
REFERENCES orders ( order_id )
ENABLE
)
TABLESPACE USERS
PCTFREE 10 
INITRANS 1
STORAGE( BUFFER_POOL DEFAULT )
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1);
```


## 插入数据

以下样例查看表空间的数据库文件，以及每个文件的磁盘占用情况。

触发器创建

```sql
create or replace trigger tr_IDADD
before insert on orders
for each row
begin
select seq_id.nextval into :new.order_id from dual;
end;


create sequence SEQ_ID
minvalue 1
maxvalue 99999
start with 1
increment by 1
cache 20
order;
```


```sql
create or replace trigger tr_DETAILS_IDADD
before insert on order_details
for each row
begin
select seql_id.nextval into :new.order_id from dual;
end;


create sequence SEQL_ID
minvalue 1
maxvalue 99999
start with 1
increment by 1
cache 20
order;
```
插入语句执行多次
```sql
!NSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('www', '182', to_date ( '2015-09-18 12:42:20' , 'YYYY-MM-DD HH24:MI:SS' ), 23, 343, 2);
INSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('eee', '155', to_date ( '2016-08-17 11:21:20' , 'YYYY-MM-DD HH24:MI:SS' ), 233, 322,4);
INSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('rrr', '182', to_date ( '2017-06-16 11:11:20' , 'YYYY-MM-DD HH24:MI:SS' ), 123, 2333,1);
insert into orders
select *
from orders;
```

```sql
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(123, 123, 123, 250);
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(234, 234, 234, 350);
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(345, 345, 345, 450);
insert into orders
select *
from order_details;
```
插入后查询结果脚本输出
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj1.png)
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj2.png)

## 分区查询
```sql
SELECT
    *
FROM orders partition (PARTITION_BEFORE_2018), order_details partition (PARTITION_BEFORE_2018);

select * from order_details ode join orders ods
	on ode.order_id=ods.order_id;
```
查询脚本
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj3.png)
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj4.png)

## 不分区查询
```sql

select * from orders, order_details where orders.order_id = order_details.order_id(+)；
```
查询脚本
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj5.png)
![](https://github.com/zsnbb/Oracle/blob/master/test3/zj6.png)


- 分区的作用：
> 分区功能能够将表、索引或索引组织表进一步细分为段，这些数据库对象的段叫做分区。每个分区有自己的名称，还可以选择自己的存储特性。

- 表分区的优点 

> 1、优化性能：可以按用户的需求制定部分查询，提高检索速度。   
> 2、容错性提高：如果表个别分区出错，表在其他分区的数据仍然可用；   
> 3、维护性上升：如果表个别分区出错，需要修复数据，只修复该分区即可；
