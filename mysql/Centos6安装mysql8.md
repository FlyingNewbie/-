

# Centos6安装mysql8

## 一、下载mysql8

下载地址：https://dev.mysql.com/downloads/mysql/

![image-20201217142101781](https://aliyun-picture.oss-cn-beijing.aliyuncs.com/image/image-20201217142101781.png)

## 二、解压

```
tar -xvf mysql-8.0.20-1.el6.x86_64.rpm-bundle.tar 
```

解压出来后入下：

```
mysql-community-server-8.0.20-1.el6.x86_64.rpm
mysql-community-libs-compat-8.0.20-1.el6.x86_64.rpm
mysql-community-client-8.0.20-1.el6.x86_64.rpm
mysql-community-common-8.0.20-1.el6.x86_64.rpm
mysql-community-test-8.0.20-1.el6.x86_64.rpm
mysql-community-devel-8.0.20-1.el6.x86_64.rpm
mysql-community-libs-8.0.20-1.el6.x86_64.rpm
```



## 三、安装

建议装之前先把之前的或自带的mysql相关包全部卸载，rpm -e --nodeps <包名>即可

安装顺序为：

```
rpm -ivh mysql-community-common-8.0.11-1.el6.x86_64.rpm \
mysql-community-libs-8.0.11-1.el6.x86_64.rpm \
mysql-community-libs-compat-8.0.11-1.el6.x86_64.rpm \
mysql-community-client-8.0.11-1.el6.x86_64.rpm \
mysql-community-server-8.0.11-1.el6.x86_64.rpm \
mysql-community-devel-8.0.11-1.el6.x86_64.rpm
```



如果报错：

```
warning: mysql-community-common-8.0.20-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
        pkgconfig(openssl) is needed by mysql-community-devel-8.0.20-1.el6.x86_64
```

原因

通过rpm安装时，会自动检查环境依赖，这是因为检查到依赖冲突导致的。

解决方法

通过 --force --nodeps 命令不检查依赖强制安装，--nodeps就是安装时不检查依赖关系，--force就是强制安装。





安装完毕查看mysql版本：

```
mysql -V
```

配置文件在  `/etc/my.cnf`  中



默认的datadir是在`/var/lib/mysql/`，可以通过修改my.cnf修改



## 四、启动

启动命令：

```
service mysqld start
```

显示如下即为成功：

![image-20201217142229979](https://aliyun-picture.oss-cn-beijing.aliyuncs.com/image/image-20201217142229979.png)



启动成功后，在 /var/log/mysqld.log 中包含登陆密码  ：

![image-20201217142247187](https://aliyun-picture.oss-cn-beijing.aliyuncs.com/image/image-20201217142247187.png)



可以使用如下命令修改密码：

```
alter user root@'localhost' identified by '123456';
```

密码设置的较简单会设置失败， 因为有复杂度校验，查看复杂度要求:

```
show variables like '%pass%';
```

![image-20201217142258221](https://aliyun-picture.oss-cn-beijing.aliyuncs.com/image/image-20201217142258221.png)

参看官方文档：https://dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html#sysvar_validate_password.policy



对校验进行如下修改后就可以设置为123456；

```
#密码长度要求
set global validate_password.length = 6;
#大小写混合数量
set global validate_password.mixed_case_count = 0;
#特殊字符
set global validate_password.special_char_count = 0; 
#密码复杂度等级
set global validate_password.policy = LOW;
```

上述命令执行在mysql重启后会失效，建议写在配置文件`/etc/my.cnf `中