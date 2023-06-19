# oracle数据库

## 一. 基本语法

### 1.表空间,创建用户,用户授权

创建表空间

```
create tablespace [name] //创建叫'name'的表空间

datafile '路径'//表空间的路径

size 100m//大小

autoextend on//空间不够时自动添加

next 10m//添加大小为10m
```

删除表空间

```
drop tablespace 'name';
```

---

创建用户:

```
create user [name]    identified [password] default [tablespace] //创建用户
```

注意:没有授权的用户是无法登陆的

---

授权:  

权限类型:

connect--只能连接,别的屁用没有

resource--开发者权限

dba--超级管理员权限

```
grant [上述权限] to  [name];
```

### 2. 创建表

创建表:

```
create table [name] (
		pid number(长度),
		pname varchar2(20)
)
```

### 3.修改表结构

修改表结构:

```
alter table [tablename] add age number(4);//增加一行
alter table [tablename] add (age number(4), psex number(4));//增加多行
```

```
alter table [tablename] modify age number(1);//修改一列
alter table [tablename] modify (age number(4), psex number(4));//修改多列
```

```
alter table [tablename] rename column age to sex;//修改列名
```

```
alter table [tablename] drop column age ;//删除一列
```

### 数据的增删改

增加,修改的基本语法和MySQL差不多  不写了,

---

删除语法

```
delete from  person;  //删除表中的所有数据
drop table  person;  //删除这个表
truncate table person;//先删除,再创建,效果等同于第一个,但是在数据量大的情况下,效率更高
```



