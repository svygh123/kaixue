---
layout: post
title: SQLServer常用语句
categories: SQLServer
lastUpdated: 
---

## {{ page.title }}

{{ page.date | date: "%Y.%-m.%-d" }} | <a href="/archive#{{ page.categories }}">{{ page.categories}}</a>

1.
```
-- 给值加1位前缀0
update CM_Secondment set fPersonID='0'+fPersonID;
-- 给值去掉第1位
update CM_Secondment set fPersonID=SUBSTRING(fPersonID,2,LEN(fPersonID));


```


2.Sql的with as 的使用

```
with as语法
–针对一个别名
with tmp as (select * from tb_name)

–针对多个别名
with
   tmp as (select * from tb_name),
   tmp2 as (select * from tb_name2),
   tmp3 as (select * from tb_name3),

--相当于建了个e临时表  
with e as (select * from scott.emp e where e.empno=7499)  
select * from e;  
   
--相当于建了e、d临时表  
with  
     e as (select * from scott.emp),  
     d as (select * from scott.dept)  
select * from e, d where e.deptno = d.deptno;  


with  
    sql1 as (select to_char(a) s_name from test_tempa),  
    sql2 as (select to_char(b) s_name from test_tempb where not exists (select s_name from sql1 where rownum=1))  
select * from sql1  
union all  
select * from sql2  
union all  
select 'no records' from dual  
       where not exists (select s_name from sql1 where rownum=1)  
       and not exists (select s_name from sql2 where rownum=1); 
```

3.重新创建主键

```
create table abcd
(
  a char(10) not null,
  b char(10) not null primary key(a,b),
  c char(10) null,
  d char(10) null
)

 

一个存在的表 abcd ,主键为 a+b ,  现在想把 a+b+c 三列修改为主键

 

 1.   删除主键：

Declare @Pk varChar(100);

Select @Pk=Name from sysobjects where Parent_Obj=OBJECT_ID('abcd') and xtype='PK';

if @Pk is not null

begin

     exec('Alter table abcd Drop '+ @Pk)  --删除原主键

 end

 

2. 把所有主键设为不能为空

alter table abcd alter column c char(10) not null

  

3.  重建主键：

ALTER Table abcd ADD CONSTRAINT pk_abcd   PRIMARY KEY (a, b, c )
```

4.查询占位符

```
select * from 表名
where 联系电话 like '[1-3]%' --------查电话是1-3开头的人
--------like是像这些条件的语句，能用通配符：%、_、[]、[^]
--------意思分别代表：所有字符、一个字符、一位上可取值、一位上不可取值

select * from 表名
where 姓名 like '[e[]%'
or 姓名 like '%e]'
escape 'e' ------------显示以‘[’开头或以‘]’结尾的所有数据，中间有不显示
----------escape ''是指定通配符
```

5.连接表更新

```
For Oracle:
update ipop_worktype_loader wl set wl.status=0
where  exists (select 1 from ipop_worktype_lkp w where  wl.worktype_id = w.worktype_id and w.application_Id=10 and wl.status=1)
 
For MS SQL Server:
update  wl  set wl.status=0  from ipop_worktype_lkp w,ipop_worktype_loader wl
where wl.worktype_id = w.worktype_id and w.application_Id=10 and wl.status=1
```

6.sqlserver数据库中随机生成N位的随机数

```
select RIGHT(100000000 + CONVERT(bigint, ABS(CHECKSUM(NEWID()))), N);
```

7.sql 触发器 if条件判断

```
if integral>50 and integral<200
begin
update customer
set lev=1
end

else 
begin

end
```

8.SQLServer数据集合的交、并、差集运算

```
并集：select * from t1 union select * from t2

不过滤重复：select * from t1 union all   select * from t2

t1对t2的差集：select * from t1 except select * from t2 

t1对t2的交集：select * from t1 intersect select * from t2

```

8.创建触发器时，从更新的表中查询出数据值

```
create trigger [tig_update] on 表名
after update  
as 
declare @id int
begin
	if (update(主表列名))
		begin
			select @id=id from inserted
			update 子表名 set lasttime = GETDATE() where pid = @id
		end
end
```

9.在SQL中获取一个长字符串中某个字符串出现次数的实现方法

```
declare @a varchar(100)
set @a='X-BGS-2010-09-15-001'
select len(replace(@a,'-','--'))-len(@a)
```

10.sql server 实现lastIndexOf

```
len(@WapImage)+1-charindex('.',reverse(@WapImage)) 
```

11.判断某个字符串是否包含另一个字符串

```
一般有两个方法：

1.用like——select * from tablename where field1 like like ‘%key%’

这种方法可以查一个句子里面包含什么词儿啥的。

2.用charindex（）——charindex（字符，字符串）>0 –>包含
```

12.使用sqlcmd 命令执行sql 脚本

```
sqlcmd -S"127.0.0.1" -U"sa" -P"abcd1234!"  -i"fincc.sql" 
```

13.获取部门字符串

```
declare @s varchar(500)
declare @n varchar(500)
set @s='/ORG01.ogn/BzAVRGMxhhxBzsIw2DQ.dpt/sU4kgIg3aCkmHQcpqcX.pos/MltPzzQvty9s6J6XS0I@sU4kgIg3aCkmHQcpqcX.psm'
set @n=reverse(substring(reverse(@s), charindex('tpd.',reverse(@s)), LEN(@s)-charindex('tpd.',reverse(@s))))
select REVERSE(substring(REVERSE(@n), 5, charindex('/',REVERSE(@n))-5))
```

14.delete where exists使用

```
DELETE FROM suppliers
WHERE EXISTS
  ( select customers.name
     from customers
     where customers.customer_id = suppliers.supplier_id
     and customers.customer_name = 'IBM' );

这将在供应商表，其中有一个客户表的名称是IBM，CUSTOMER_ID是相同的supplier_id记录中删除所有记录。

15.查看sqlserver数据库当前死锁

```
--死锁检测
use master
Select * from sysprocesses where blocked<>0
--找到SPID  
exec sp_lock
--根据SPID找到OBJID
select object_name(85575343)
--根据OBJID找到表名

```

16.查询字段中包含字母的结果

```
select PATINDEX('%[A-Za-z]%', 'ads23432')>0
```

17.with语句使用

```
with aliasName as (select * from tableName)

如果有多个,则用逗号分隔,如

with alias1 as (select * from tableName1),
alias2 as (select * from tableName2)
```

18.SQL 中如何去掉decimal字段後面的0

先转为float再转为varchar

```
CONVERT(varchar(10), cast(a.qty as float))
```

19.Sql server存储过程中常见游标循环用法

```
DECLARE 
  @A1 VARCHAR(10),
  @A2 VARCHAR(10),
  @A3 INT
DECLARE YOUCURNAME CURSOR FOR SELECT A1,A2,A3 FROM YOUTABLENAME 
OPEN YOUCURNAME 
fetch next from youcurname into @a1,@a2,@a3 
while @@fetch_status<>-1
 begin 
   --您要执行的操作写在这里 
   fetch next from youcurname into @a1,@a2,@a3 
end 
close youcurname 
deallocate youcurname
```

20.sqlserver 行转列

```
SELECT 
    [UserName],
    SUM(CASE [Subject] WHEN '数学' THEN [Source] ELSE 0 END) AS '[数学]',
    SUM(CASE [Subject] WHEN '英语' THEN [Source] ELSE 0 END) AS '[英语]',
    SUM(CASE [Subject] WHEN '语文' THEN [Source] ELSE 0 END) AS '[语文]'     
FROM [TestRows2Columns]
GROUP BY [UserName]
```

21.使用临时表实现字符串合并处理的示例

```
--3.3.3 使用临时表实现字符串合并处理的示例
--处理的数据
CREATE TABLE tb(col1 varchar(10),col2 int)
INSERT tb SELECT 'a',1
UNION ALL SELECT 'a',2
UNION ALL SELECT 'b',1
UNION ALL SELECT 'b',2
UNION ALL SELECT 'b',3
 
--合并处理
SELECT col1,col2=CAST(col2 as varchar(100)) 
INTO #t FROM tb
ORDER BY col1,col2
DECLARE @col1 varchar(10),@col2 varchar(100)
UPDATE #t SET 
    @col2=CASE WHEN @col1=col1 THEN @col2+','+col2 ELSE col2 END,
    @col1=col1,
    col2=@col2
SELECT * FROM #t
/*--更新处理后的临时表
col1       col2
---------- -------------
a          1
a          1,2
b          1
b          1,2
b          1,2,3
--*/
--得到最终结果
SELECT col1,col2=MAX(col2) FROM #t GROUP BY col1
/*--结果
col1       col2
---------- -----------
a          1,2
b          1,2,3
--*/
--删除测试
DROP TABLE tb,#t
GO
```

22.字符串转日期时间

```
convert(datetime,'2017-12-12 00:00:01', 20)
```

23.SQL Server中取得某个月的天数

```
declare @dt datetime 
set @dt=getdate() 
select 32-Day(@dt+(32-Day(@dt)))
```

24.SQL Server中 求两个指定日期的相差天数

```
-- 求两个指定日期的相差天数,不考虑时、分、秒

declare @StartDateTime datetime

declare @EndDateTime datetime

declare @iDays int

select @StartDateTime = '2009-04-06'

select @EndDateTime = '2009-04-10 09:23:23'

select @iDays = datediff(day,convert(varchar(100),@StartDateTime,23),convert(varchar(100),@EndDateTime,23))

print @iDays
```

25.SQL server 除法运算

```
select 500 / (501) *100

结果为0，由于数值是INT类型，怎么才能得到小数点呢？

select 500 / (501 + 0.0) *100

99.800300
这样就有小数点了
```

26.sql表中如何获得最大时间的记录

```
SELECT  top 1 * FROM  table   WHERE   MaxTime= (select MAX(MaxTime) from  table) 
```

27.sqlserver以逗号分割的字符串拆分到临时表

```
alter FUNCTION [dbo].[func_split](@str nvarchar(4000),@separtor varchar(10))   
  returns @temp table([row] [int] IDENTITY(1,1) NOT NULL,valuess nvarchar(4000))   
  as    
begin  
   declare @i int  
   set  @str=rtrim(ltrim(@str))   
   set  @i=charindex(@separtor,@str)   
   while   @i>=1   
    begin  
     insert   @temp   values(left(@str,@i-1))   
     set  @str=substring(@str,@i+1,len(@str)-@i)   
     set  @i=charindex(@separtor,@str)   
    end  
    if @str<>''    
    insert @temp values(@str)   
    return    
end

select * from  dbo.func_split('1,2,3,4,5,7', ',')
```

28.SqlServer实现mysql中GROUP_CONCAT函数

```
-- 新建测试表
CREATE TABLE temp(id int, value varchar(10));
 
-- 插入测试数据
INSERT INTO temp VALUES(1, 'aa'); 
INSERT INTO temp VALUES(1, 'bb'); 
INSERT INTO temp VALUES(2, 'cc'); 
INSERT INTO temp VALUES(2, 'dd'); 
INSERT INTO temp VALUES(2, 'ff'); 
 
 
-- 查询，实现相同id的name以，分隔拼成字符串，id和value都为字段名称
  SELECT id, 
         [value] = stuff((
                 SELECT ',' + [value] 
                   FROM temp t 
                  WHERE t.id = temp.id 
                    FOR xml path('')) , 1 , 1 , '') 
    FROM temp 
GROUP BY id;

```

29.数据字典SQL

```
DECLARE @TableName varchar(20)
SET @TableName='MM_BNHead'
SELECT
        --(CASE when a.colorder=1 then d.name else '' end) AS 表名,
        a.colorder 字段序号,
        a.name 字段名,
        isnull(g.[value],'') AS 字段说明,        
        b.name 类型,
        '' as 创建人,
        --COLUMNPROPERTY(a.id,a.name,'PRECISION') as 长度,        
        case when b.name='decimal' then CONVERT(varchar(10),COLUMNPROPERTY(a.id,a.name,'PRECISION')) + ',' + CONVERT(varchar(10),isnull(COLUMNPROPERTY(a.id,a.name,'Scale'),0))
        else CONVERT(varchar(10),COLUMNPROPERTY(a.id,a.name,'PRECISION')) + '' end as 长度,
        --isnull(COLUMNPROPERTY(a.id,a.name,'Scale'),0) as 小数位数,
        (case when a.isnullable=1 then '' else 'Y' end) 是否必填
        /*,
        (case when COLUMNPROPERTY( a.id,a.name,'IsIdentity')=1 then '√'else '' end) 标识,
        (case when (SELECT count(*)
        FROM sysobjects
        WHERE (name in
                  (SELECT name
                FROM sysindexes
                WHERE (id = a.id) AND (indid in
                          (SELECT indid
                         FROM sysindexkeys
                         WHERE (id = a.id) AND (colid in
                                   (SELECT colid
                                  FROM syscolumns
                                  WHERE (id = a.id) AND (name = a.name))))))) AND
              (xtype = 'PK'))>0 then '√' else '' end) 主键,
       
       a.length 占用字节数,         
       isnull(e.text,'') 默认值       
		*/
FROM syscolumns  a
    LEFT JOIN systypes b on  a.xtype=b.xusertype
    inner join sysobjects d on a.id=d.id  and  d.xtype='U' and  d.name<>'dtproperties'
    left join syscomments e on a.cdefault=e.id
    left join sys.extended_properties g on a.id=g.major_id AND a.colid = g.minor_id
WHERE d.name=@TableName    --如果只查询指定表,加上此条件
order by a.id,a.colorder
```

**更新列表：**

*



**参考文章：**

* [Sql的with as 的使用][1]
* [Sql Server删除主键和重建主键][2]
* [sql server中sql语句][3]
* [Oracle和SQL Server连接表的更新操作Update SQL 语句][4]
* [sqlserver数据库中随机生成N位的随机数][5]
* [sql 触发器 if条件判断][6]
* [SQLServer触发器创建、删除、修改、查看][7]
* [SQLServer数据集合的交、并、差集运算][8]
* [SQLServer存储过程返回值总结][9]
* [使用存储过程并返回值与及返回值的获得方法][10]
* [在SQL中获取一个长字符串中某个字符串出现次数的实现方法][11]
* [sql server 实现lastIndexOf][12]
* [sql中判断某个字符串是否包含一个字符串][13]
* [sqlserver函数的使用][14]
* [使用sqlcmd 命令执行sql 脚本][15]
* [sql delete 语句用法介绍][16]
* [查看sqlserver数据库当前死锁][17]
* [数据库连接池到底应该设多大？这篇文章可能会颠覆你的认知][18]
* [SQL 中如何去掉decimal字段後面的0][19]
* [Sql server存储过程中常见游标循环用法][20]
* [sqlserver 行转列][21]
* [如何将SQL查询结果的列数据连接成一行（在SQL中用函数或存储过程实现）][22]
* [sql存储过程，语句拼接，使用游标][23]
* [SqlServer实现mysql中GROUP_CONCAT函数][24]

[1]: https://blog.csdn.net/turejackon/article/details/76607492
[2]: https://blog.csdn.net/bobwu/article/details/5715529
[3]: https://zhidao.baidu.com/question/257951371
[4]: http://www.cnblogs.com/ycxyyzw/archive/2012/03/09/2387668.html
[5]: https://blog.csdn.net/u013628196/article/details/44778099
[6]: http://www.cnblogs.com/activities/archive/2012/06/06/2537605.html
[7]: https://blog.csdn.net/fwj380891124/article/details/7016328
[8]: https://blog.csdn.net/jinjazz/article/details/4527863
[9]: https://www.cnblogs.com/soundcode/p/6810712.html
[10]: http://www.cnblogs.com/zm235/archive/2008/05/09/1189622.html
[11]: https://www.jb51.net/article/39813.htm
[12]: http://tanyongbing.iteye.com/blog/2071375
[13]: https://www.cnblogs.com/ahlx/p/5292200.html
[14]: https://jingyan.baidu.com/article/7908e85caa9511af481ad2b4.html##1
[15]: https://blog.csdn.net/zghnpdswyp/article/details/49635609
[16]: https://yq.aliyun.com/ziliao/41767
[17]: https://blog.csdn.net/saga_gallon/article/details/52465187
[18]: https://blog.csdn.net/pangjl1982/article/details/79295241
[19]: https://blog.csdn.net/donnie88888888/article/details/52439299
[20]: https://www.cnblogs.com/tuyile006/p/5319117.html
[21]: https://blog.csdn.net/zml_900417520/article/details/45641047
[22]: https://bbs.csdn.net/topics/250076021
[23]: https://blog.csdn.net/bing1051/article/details/16986175
[24]: https://blog.csdn.net/Cxy_357/article/details/82190771