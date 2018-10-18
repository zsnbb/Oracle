# Oracle
# 实验一：分析SQL执行计划，执行SQL语句的优化指导

## 实验内容：
- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

## 教材中的查询语句

-查询1：

```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```
运行结果：
![运行结果](https://github.com/zsnbb/Oracle/blob/master/test1/zj.png)



- 查询2：
```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
运行结果：
![运行结果](https://github.com/zsnbb/Oracle/blob/master/test1/zj1.png)

优劣：我认为查询2优于查询一。两条语句都是查询所有部门及以下员工的总数与平均工资，第一条语句直接在匹配外键约束时，进行查询内容过滤，在结果返回之前起作用，后者则是在匹配出外键之后相当于新生成一张表，再从那张表里进行查询内容过滤，在结果返回之后起作用，所以查询2在查询大量数据时将会优于查询1，在查询时间以及效率比查询一快。

-SQL developer 优化指导：
在比较查询1与查询2的优劣后，选择查询2使用SQL developer进行优化指导，指导结果：
![运行结果](https://github.com/zsnbb/Oracle/blob/master/test1/zj2.png)

![运行结果](https://github.com/zsnbb/Oracle/blob/master/test1/zj3.png)

- 自定义简单查询：
查询所有工作里工资小于5000的人员信息，当前显示了第一页
查询思想为将查询语句视为一张表，查询这个表里所有的内容和rownum这个关键字
最后再限制rownum的范围

```sql
select t.*,rownum 
from (
select j.job_id,j.job_title 
from hr.jobs j 
where j.min_salary<5000 
order by j.job_id) t 
where rownum>0 and rownum<=10
```

运行结果：
![运行结果](https://github.com/zsnbb/Oracle/blob/master/test1/zj4.png)
