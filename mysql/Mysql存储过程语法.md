# Mysql存储过程语法

## 一、创建语法

CREATE    PROCEDURE     存储过程名（参数列表）

​	BEGIN

​		存储过程体（一组合法的sql语句）

​	END



#### **注意：**

##### 1、参数列表包含三个部分

​	参数模式      参数名      参数类型

​	IN  			test           varchar(20)

参数模式：

IN:该参数可以作为输入，也就是该参数需要调用方传入值

OUT：该参数作为输出，也就是可以作为返回值

INOUT：该参数既可以作为输入也可以作为输出

##### 2、如果存储体仅仅只有一句话，那么BEGIN和END可以省略

存储过程体中每条sql语句的结尾要求必须加分号；

存储过程的结尾可以使用	DELIMITER重新设置

语法：

##### DELIMITER  结束标记

案例：

DELIMITER  $

##### 3、声明变量

DECLARE 变量名   数据类型  

案例：

DECLARE   result   varchar(20)   default    '';//声明变量并初始化

##### 4、给变量赋值

selct   count(*)   into result   from   student;



## 二、调用语法

CALL   存储过程名（实参列表）





## 三、删除存储过程

语法

DROP   PROCEDURE   存储过程名





## 四、查看存储过程信息

show    create   procedure   存储过程名  ；



# 函数创建

## 一、创建语法

CREATE     FUNCTION     函数名（参数列表）  RETURNS    返回类型

BEGIN

函数体

END

##### 注意：

 1、参数列表包含两部分

参数名	参数类型

2、函数体：肯定会有return语句，如果没有会报错

如果return语句没有放在函数体的最后也不报错，但不建议

3、函数体仅有一句就可以省略BEGIN和END

4、使用DELIMITER语句设置结束标记



## 二、调用函数

SELECT   函数名（参数列表）





## 三、查看函数

SHOW   CREATE   FUNCTION  函数名



## 四、删除函数

DROP    FUNCTION   函数名





# 流程控制结构

##### 顺序结构：

程序从上往下依次执行

##### 分支结构：

程序从两条或多条路径中选择一条去执行

##### 循环结构:

程序在满足一定条件的基础上,重复执行一段代码

### 一、分支结构

##### 1.if函数

功能:实现简单的双分支
语法
if(表达式1,表达式2,表达式3)
执行顺序:
如果表达式1成立,则if函数返回表达式2的值,否则返回表达式3的值
应用:任何地方

##### 2.case结构

情况1:类似于java中的 switch语句,一般用于实现等值判断
语法:
​	CASE	变量|表达式|字段
​	WHEN	要判断的值	THEN	返回的值1或语句1;
​	WHEN	要判断的值	THEN	返回的值2或语句2;

​	...

​	ELSE	要返回的值n或语句n;

​	END	CASE;
情况2:类似于java中的金重IF语句,一般用于实现区间判断
语法:
​	CASE
​	WHEN	要判断的条件1	THEN	返回的值1或语句1
​	WHEN	要判断的条件2	THFN	返回的值2或语句2

​	...

​	ELSE	要返回的值n或语句n;
​	END CASE;

特点:
①
可以作为表达式,嵌套在其他语句中使用,可以放在任何地方, BEGIN END中或 BEGIN END的外面
可以作为独立的语句去使用,只能放在 BEGIN END中
②
如果WHEN中的值满足或条件成立,则执行对应的THEN后面的语句,并且结束CASE
如果都不满足,则执行ELSE中的语句或值
③ELSE可以省略,如果ELSE省略了,并且所有WHEN条件都不满足,则返回NULL



##### 3.IF结构

功能：实现多重分支

语法：

if	条件1	then	语句1；

elseif	条件2	then	语句2；

...

【else	语句n;】

end if;

应用场合：应用在begin	end中



### 二、循环结构

分类：

​	while、loop、repeat

循环控制：

iterate类似与	continue，继续，结束本次循环，继续下一次

leave	类似与	break	跳出，结束当前所在的循环



##### 1.while

语法：

【标签：】while	循环条件	do

​			循环体；

end while【标签】；



##### 2、loop

语法：

【标签：】	loop

​			循环体；

end	loop	【标签】；

可以用来模拟简单的死循环；



##### 3、repeat

语法：

【标签：】repeat

​			循环体；

​		until	结束循环的条件

end	repeat	【标签】；

