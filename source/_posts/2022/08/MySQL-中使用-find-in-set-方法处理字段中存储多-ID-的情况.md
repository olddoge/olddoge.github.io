---
title: MySQL 中使用 find_in_set 方法处理字段中存储多 ID 的情况
date: 2022-08-07 21:16:51
tags: [MySQL, 数据库]
categories: SQL
description: 如何使用 FIND_IN_SET 方法和 GROUP_CONCAT 更好的处理单字段存储多 id 的查询情况，提高查询效率
---

开发中经常会遇到用在某一张表的某个字段中存储另一张表`id`的情况，通常情况下，会采用其他语言先查询出结果，再对这个字段做拆分，接着用拆分的字段去另一张表做查询。

实际上，这种方法效率较低，`MySQL`中有一个现成的方法可以很好的处理这种情况。

# 场景

现在假设有下面 2 张表，一张`教师表（teachers）`，一张`学生表（students）`

- Teachers Table

|id|name|student_ids|
|:--:|:--:|:--:|
|1|老师A|1,2,3|
|2|老师B|4,5,6|
|3|老师C|2|
|4|老师D||

- Students Table

|id|name|
|:--:|:--:|
|1|学生A|
|2|学生B|
|3|学生C|
|4|学生D|
|5|学生E|
|6|学生F|

# FIND_IN_SET 函数

> 官方说明
> FIND_IN_SET(str,strlist) : str 要查询的字符串，strlist  需查询的字段，参数以 **","** 分隔，形式如 (1,2,6,8,10,22)；该函数的作用是查询字段(strlist)中是否包含(str)的结果，返回结果为null或记录。

```SQL
FIND_IN_SET('被关联的ID', '用英文逗号连接来存储ID集合的字段')
```

***只有以<span style="color:red">英文逗号分隔</span>的字段值，才能使用***

# 实际使用

现在我们要将`教师表`和`学生表`做关联，得到每一位教师底下所拥有的学生，期望的结果如下表：

- Results table

|id|teacher|student|
|:--:|:--:|:--:|
|1|老师A|学生A,学生B,学生C|
|2|老师B|学生D,学生E,学生F|
|3|老师C|学生B|
|4|老师D||

每一位教师的学生字段值，显示的是用逗号连接的学生名，原先比较笨的实现方法是，先查出所有教师的数据，然后拆分学生`id`，再用`id`去学生表查询出(好一点用`IN`查询)学生名字，最后组成结果。

现在可以用`FIND_IN_SET`和`GROUP_CONCAT`结合的方式做联表查询

具体语句如下：

```SQL

SELECT
    t.id,
    t.name AS teacher,
    GROUP_CONCAT(s.name) AS student
FROM
    teachers AS t
LEFT JOIN 
    students AS s ON FIND_IN_SET(s.id, t.student_ids)
GROUP BY
    t.id;   -- 这里一定要 group by 一下，不然会出现结果丢失

```

这样就能得到我们想要的结果了。

# 其他注意点

***GROUP_CONCAT() 方法有最大字符限制***