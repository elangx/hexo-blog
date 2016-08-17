---
title: PHP编译过程中出现virtual memory exhausted： Cannot allocate memory
date: 2015-09-11 11:06:53
tags: PHP
---
* 今天在机器上编译php5.6，make时出现一个virtual memory exhausted: Cannot allocate memory的问题，感觉我的云主机虽然比较low但也不至于如此吧，怎么说也有1G内存呢（虽然当时开着mysql和nginx），于是开始去网上找原因，最后在官网bug里发现这么一条[https://bugs.php.net/bug.php?id=48809](https://bugs.php.net/bug.php?id=48809)，里面提到编译可能需要600M-700M……
* 不过下面有人给出个方案，configure时加上--disable-fileinfo选项，具体原因还不知道，反正现在还不需要fileinfo，用到时再加扩展吧
