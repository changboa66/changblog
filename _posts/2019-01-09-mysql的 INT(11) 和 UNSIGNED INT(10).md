---
layout: mypost
title: mysql的 INT(11) 和 UNSIGNED INT(10)
categories: [database mysql]
---

在MySQL中，如果我们创建一个字段dataType的INT，没有指定任何长度/值，那么它自动变为int(11)，如果我们设置属性UNSIGNED或UNSIGNED ZEROFILL，那么它变成int(10)

答案：
![mysql-number-type](mysql-number-type.png)
int值可以是-2147483648，所以最小数是11位的（包含‘-’），因此默认显示宽度为11
unsigned int不允许负数，因此默认情况下它只需要显示宽度10位

当int字段类型设置为 **无符号** 且 **填充零（UNSIGNED ZEROFILL）** 时，当数值位数未达到设置的显示宽度时，会在数值前面补充零直到满足设定的显示宽度，为什么会有无符号的限制呢，是因为ZEROFILL属性会隐式地将数值转为无符号型，因此不能存储负的数值。

![sql-and-java-type](sql-and-java-type.png)
