# Oracle存储过程

## 1、创建和使用存储过程

用CREATE PROCEDURE命令建立存储过程和存储函数

语法：

​	create  [or	replace]	Procedure 过程名（参数列表）

​	AS

​	PLSQL子程序体；

例子：

​	create or replace procedure  sayHelloWorld

​	as	

​	begin

​		dbms_output.put_line('hello world');

​	end;

​	/

## 2、调用存储过程

### 2.1、exec	sayHelloWorld()

## 2.2、begin

​		sayHelloWorld()

​		end;

​		/





