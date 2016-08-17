---
title: PHP模板引擎的那些事
date: 2016-06-01 20:46:43
tags: PHP
---
### 模板引擎是什么
最早的时候，php是与html写在一起的，大概这也是它最早能够火起来的原因之一吧，这种写法应该早就已经废弃了，但其实1年多之前，前一个公司还有个实习的测试同学因为毕业设计找了个php项目让我帮忙看一下，这个项目就是混写出来的，看得我实在是相当的不习惯，而这个项目就是公司另一个php开发找来给她的……

我没有赶上早期php发展，不知道模板产生的原因是什么，但我觉得为了能把html与php代码分开应该就是主要原因之一，早期php没有MVC这类概念，讲究简单快速，为了能把V这一层独立出来所以有了模板引擎的出现。

其实模板引擎的作用很单纯，就是为了把V层独立出来给前端人员，他们能在不使用php代码的情况下写完所有V层，与C层只通过对应的变量名来对接，大多数模板实时就是自己定义了一套写法，然后在运行时将这套写法写出的模板“编译”（其实就是通过replace或正则来替换）成php代码然后执行。<!--more-->
### 当下模板引擎的作用
如今php发展已经算是比较成熟了，也诞生了各种各样的框架，MVC的理念也深入人心，所以其实大家现在还在使用模板引擎的原因就是因为习惯，或者与C层分开这样的观念，但其实有了MVC逻辑上是已经分开的，只是如果没有模板引擎语言上没有分开，大家觉得如果不用模板引擎那前端就要学习php了，但就不用那前端也是要学习模板层的语言的，都是有学习成本的，而且像smarty中的那些函数依然是用php写的。

其实php本身就是一种模板语言，模板引擎只是把自己定义成另一套模板语言而已，所换来的就是V层看起来稍微整洁一点。

那么使用模板引擎又会失去什么？最主要的肯定就是性能！我们把一个语言去转换成php，然后再次执行php，执行过程中会比之前多执行更多的代码，像smarty会执行几千个函数来进行，另外其在将代码“编译”到php时所使用的各种正则效率也相当低，更会使性能打折。
### V层要怎么写
前面提到了很多次smarty，因为这个引擎应该是大家使用的最多的了，这个也被官方认定为是官方的模板引擎，也有许多认为V层就是smarty，smarty就是V层。那这里我们先测一下原生php与smarty，写一个比较简单的混写的和通过smarty来渲染的例子，直接用smarty中带的demo就可以。
##### 环境
	CPU：Intel(R) Xeon(R) CPU E5-2620 0 @ 2.00GHz
	内存：128G
	系统：CentOS release 4.3
	PHP版本：5.6.7
	nginx版本：1.6.2

通过ab -n 10000 -c 100 的参数来跑几次，得出结果如下：

	方式：混写
	
	Concurrency Level:      100
	Time taken for tests:   0.949 seconds
	Complete requests:      10000
	Failed requests:        0
	Write errors:           0
	Total transferred:      8280000 bytes
	HTML transferred:       6670000 bytes
	Requests per second:    10538.56 [#/sec] (mean)
	Time per request:       9.489 [ms] (mean)
	Time per request:       0.095 [ms] (mean, across all concurrent requests)
	Transfer rate:          8521.42 [Kbytes/sec] received


	方式：smarty:
	Concurrency Level:      100
	Time taken for tests:   2.534 seconds
	Complete requests:      10000
	Failed requests:        0
	Write errors:           0
	Total transferred:      11170000 bytes
	HTML transferred:       9560000 bytes
	Requests per second:    3945.93 [#/sec] (mean)
	Time per request:       25.343 [ms] (mean)
	Time per request:       0.253 [ms] (mean, across all concurrent requests)
	Transfer rate:          4304.30 [Kbytes/sec] received
这里可以看到性能差距相当的夸张（不过这里提到一点我smarty中cache的打开和关闭并测下来没有什么区别，不知道什么原因），smarty的效率低的有点过分了，比如这篇[《HHVM at Baidu》](http://lamp.baidu.com/2014/11/04/hhvm-in-baidu/)文章中提到smarty的渲染占了业务性能的60%，当然我并不知道这个业务中有什么逻辑，但也可以证明smarty确实性能不怎么样。

既然smarty性能太差，那我们怎么写V层？
1. 写个简单的渲染库，直接extract到模板文件中，模板文件中也可以用php来处理一些简单逻辑，我觉得也挺好的啊
2. 其它模板引擎，其实不用模板引擎就是因为性能比较差，如果性能好，当然用起来也舒心，而且也不止smarty这一个吗，其它很多都比smarty快，这里推荐一个Blitz

### 高性能模板引擎Blitz
一句话，Blitz高性能的原因就是他是php的扩展，而且最近还看到它的[github](https://github.com/alexeyrybak/blitz)上更新了php7的版本，看来作者对这个还是很看重的，而且语法也比较简单，目前看来唯一一个不爽的地方就是set一个array后要遍历的话这个array的层数会比其它的多，具体直接看[官方文档](http://alexeyrybak.com/blitz/blitz_en.html#quick-geek)就好了，另外我也跑了一个一样的测试脚本，最后看下Blitz的速度吧。

	方式：blitz
	Concurrency Level:      100
	Time taken for tests:   0.991 seconds
	Complete requests:      10000
	Failed requests:        0
	Write errors:           0
	Total transferred:      8970897 bytes
	HTML transferred:       7360736 bytes
	Requests per second:    10090.19 [#/sec] (mean)
	Time per request:       9.911 [ms] (mean)
	Time per request:       0.099 [ms] (mean, across all concurrent requests)
	Transfer rate:          8839.65 [Kbytes/sec] received
