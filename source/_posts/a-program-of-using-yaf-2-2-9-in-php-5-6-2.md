---
title: YAF 2.2.9在php5.6.2中的一个问题记录
date: 2015-02-11 22:47:43
tags: PHP
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近在用yaf，在家里的电脑上编译安装，安装的是最近的稳定版2.2.9，php版本是5.6.2，在编译时会却提示了一个错误：
```bash
/Users/yangxian/tmp/yaf-2.2.9/yaf_config.c:89:9: error: use of undeclared identifier
'IS_CONSTANT_ARRAY' case IS_CONSTANT_ARRAY: {
```
<!--more-->
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;于是就自己在yaf_config.c中define了一个，编译通过后写了一个简单的实例，但不知道为什么，在输出页面时总加上一个乱码bom头之类的东西，但我的vim编辑的文件是没有bom头的，而且发现输出是在action之后，view文件之前的，可能是框架本身的问题，由于yaf是C的扩展，目前要调试就只能跟到源码中了……去yaf群里问还被个『大神』鄙视……非把chrome自己的排版问题说成我把html头写到了body里……无果后觉得可能是2.2.9和高版本php之间的问题或者我加了define完后出的问题。后在[yaf的issue](https://github.com/laruence/php-yaf/issues/144)中有人问过一个类似问题，鸟哥还是建议用yaf的2.3.3了，于是编译了2.3.3后问题就消失了……这里记录下吧
