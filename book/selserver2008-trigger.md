SQL Server 2008 R2 触发器调用python
======
# SQL Server 触发器
# 创建触发器
```
reconfigure 
go 
exec sp_configure 'show advanced options',1
reconfigure
go
exec sp_configure 'xp_cmdshell',1
go
reconfigure
go
use test
go
if exists(select name from sysobjects where name='trig_python1' )
Drop trigger trig_python1
go
create Trigger trig_python1
on test_table 
after insert,update,delete
as 
begin
declare @res char(2000)
select @res=convert(char(100),id)+title+content from Inserted 
select @sql = 'C:/python3-32/python E:1.py ' +@res
exec xp_cmdshell @sql
select * from inserted 
end
go

```
```
# 语法
# 创建触发器
CREATE TRIGGER  Trigger_Name --触发器名，在一个数据库中触发器名是唯一的。 
  
   ON  Table_Name | View_Name --触发器所在的表或者视图。 
  
   AFTER(FOR)|Instead Of  INSERT,DELETE,UPDATE --定义成AFTER或Instead Of类型的触发器。 
    
   --AFTER跟FOR相同，不可在视图上定义AFTER触发器 
    
   -- 后面是触发器被触发的条件，最少有一个，可以邮多个。如果有多个用逗号分开，顺序无要求。  www.2cto.com   
  
AS --触发器要执行的操作 
  
BEGIN 
  
   --BEGIN跟END组成一个代码块，可以写也可以不写，如果触发器中执行的SQL语句比较复杂，用BEGIN和END会让代码更加整齐，更容易理解。 
  
END 
GO --GO就代表结操作完毕 

# 启用xp_cmdshell

```
reconfigure  
go 
exec sp_configure 'show advanced options',1
reconfigure
go
exec sp_configure 'xp_cmdshell',1
go
reconfigure
go
```
```
# 拼接多列数据
```
select @res=convert(char(100),id)+title+content from Inserted 
```
# 将查询结果传值给python程序

```
select @sql = 'C:/python3-32/python E:1.py ' +@res
exec xp_cmdshell @sql
```
```
# 注意
触发器将会生成临时表，要得到当前插入的数据应该查询临时表

1，After触发器只能用于数据表不能用于视图；Instead Of触发器两者皆可，设置为With Check Option的视图也不允许建立Instead Of触发器。两种触发器都不可以建立在临时表上。 
2，一个数据表可以有多个触发器，但是一个触发器只能对应一个表。 
3，在同一个数据表中，对每个操作（如Insert、Update、Delete）而言可以建立许多个After触发器，而Instead Of触发器针对每个操作只有建立一个。 
4，如果针对某个操作即设置了After触发器又设置了Instead Of触发器，那么Instead of触发器一定会激活，而After触发器就不一定会激活。 
  
5，不同的SQL语句，可以触发同一个触发器，如Insert和Update语句都可以激活同一个触发器。 
6，触发器名在所在的数据库里必须是唯一的。由于触发器是建立中数据表或视图中的，所以有很多人都以为只要是在不同的数据表中，触发器的名称就可以相同，其实触发器的全名（Server.Database.Owner.TriggerName）是必须 唯一的，这与触发器在哪个数据表或视图无关。 
7，关键字AFTER可以用For来代取，它们的意思都是一样的，代表只有在数据表的操作都已正确完成后才会激活的触发器。 
# 参考
[SQL Server 触发器学习总结](http://www.voidcn.com/article/p-pszpyqnh-rd.html)

[sql server 2008如何打开和禁用xp_cmdshell](https://blog.csdn.net/wiser/article/details/23434607)

[初学sql server 2008之触发器](https://www.cnblogs.com/koeltp/archive/2012/04/07/2436364.html)
