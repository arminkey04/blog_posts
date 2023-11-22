---
title: Linux用户和用户组管理
date: 2023-11-22 21:05:03
categories:
- 学习
tags:
- 运维
- linux
---
## 为什么需要了解用户和组

服务器要添加多账户的作用

- 针对不同用户分配不同的权限，不同权限可以限制用户可以访问到的系统资源
- 提高系统的安全性
- 帮助系统管理员对使用系统的用户进行跟踪

## 用户和组的关系
理论上Linux系统中的每个用户在创建时都应该有一个对应的用户组，这个组就称之为用户的主组。同时，有些情况下，某个用户需要临时使用某个组的权限，那这个组就称之为这个用户的附属组或附加组。

> 主组只能拥有一个，但是附属组或附加组可以同时拥有多个 => 亲爹，干爹（多个）

## 用户组操作

用户组的操作无疑三件事：用户组的添加、用户组的修改以及用户组的删除操作

- 组：group

- 添加：add

- 修改：mod

- 删除：del

### 用户组的添加

基本语法：

```powershell
# groupadd [选项]  用户组的组名称
选项说明：
-g ：代表用户组的组ID编号，自定义组必须从1000开始，不能重复
```

案例：在系统中添加一个hr的用户组

```powershell
# groupadd hr
```

案例：在系统中添加一个test的用户组并指定定编号1100

```powershell
# groupadd -g 1100 test
```

#### 问题：我们刚才创建的hr以及test用户组到底添加到哪里了？

答：默认情况下，我们添加的用户组都会放在一个系统文件中，文件位置在`/etc/group`
```powershell
# tail -3 /etc/group
root:x:0:
mememe:x:1000:
```

### /etc/group文件解析

由以上命令的执行结果可知，在`/etc/group`文件中，其一共拥有三个冒号，共四列。每列含义：

```powershell
第一列：用户组的组名称
第二列：用户组的组密码，使用一个x占位符
第三列：用户组的组ID编号，1-999代表系统用户组的组编号，1000以后的代表自定义组的组编号
CentOS6 => 1-499,500...
CentOS7 => 1-999,1000...
第四列：用户组内的用户信息（如果一个用户的附属组或附加组为这个组名，则显示在此位置）
```

### 用户组的修改

基本语法：

```powershell
# groupmod [选项 选项的值] 原来组的组名称
选项说明：
-g ：gid缩写，设置一个自定义的用户组ID数字，1000以后
-n ：name缩写，设置新的用户组的名称
```

案例：把hr用户组更名为szhr

```powershell
# groupmod -n szhr hr
```

案例：把test用户组的组编号由1100更改为1003

```powershell
# groupmod -g 1003 test
```

案例：把mememe组的组名称更改为admin且用户组的组编号更改为1004

```powershell
# groupmod -g 1004 -n admin mememe
```

### 用户组的删除

基本语法：

```powershell
# groupdel 用户组名称
```

案例：使用groupdel删除test用户组

```powershell
# groupdel test
```

## 4、用户操作

- 用户：user

- 添加：add

- 修改：mod

- 删除：del

### 用户的添加

基本语法：

```powershell
# useradd [选项 选项的值] 用户名称
选项说明：
-g ：代表添加用户时指定用户所属组的主组，唯一的组信息（重要）
-s ：代表指定用户可以使用的Shell类型，默认为/bin/bash（拥有大部分权限）还可以是/sbin/nologin，代表账号创建成功，但是不能用于登录操作系统。
/bin/bash => 给人使用的（运维工程师）
/sbin/nologin => 给软件使用的

-G ：代表添加用户时指定用户所属组的附属组或附加组，可以指定多个，用逗号隔开即可
-u ：代表添加用户时指定的用户ID编号，CentOS6从500开始，CentOS7中从1000开始
-c ：代表用户的备注信息，cqw:123456:(cqw的账号)
-d ：代表用户的家目录，默认为/home/用户名称。可以使用-d进行更改
-n ：取消建立以用户名称为名的群组（了解）
```

案例：在系统中创建一个linuxuser账号

```powershell
# useradd linuxuser
```

#### 问题：我们并没有为linuxuser账号指定所属的主组，可以成功创建账号么？

答：可以，因为在创建账号时，如果没有明确指定用户所属的主组，默认情况下，系统会自动在用户组中创建一个与用户linuxuser同名的用户组，这个组就是这个用户的主组。

#### 问题：刚才创建的linuxuser账号能不能用于登录操作系统

答：不行，因为Linux的登录账号必须要求有密码，如果一个账号没有密码是无法登录操作系统的。



案例：在系统中创建一个账号zhangsan，指定用户所属的主组为mememe

```powershell
第一步：查询一下mememe的组ID编号
# tail -5 /etc/group
mememe:x:1000:
第二步：根据组的编号添加用户
# useradd -g 1000 zhangsan
```

案例：在系统中创建一个账号lisi，指定主组为mememe，此用户只能被软件所使用，不能用于登录操作系统

```powershell
# useradd -g 1000 -s /sbin/nologin lisi
```

案例：在系统中创建一个wangwu，指定主组为wangwu，附属组为mememe

```powershell
# useradd -G 1000 wangwu
```



### 用户信息查询

基本语法：

```powershell
# id 用户名称
```

主要功能：查询某个指定的用户信息

案例：查询linuxuser用户的信息

```powershell
# id linuxuser
uid=1002(linuxuser) gid=1005(linuxuser) groups=1005(linuxuser)
uid：用户编号
gid：用户所属的主组的编号
groups：用户的主组以及附属组信息，第一个是主组，后面的都是附属组或附加组信息
```

