---
title: GO语言印象
date: 2016-08-05 18:06:27
tags: GO
---
看了一段时间的GO，是时候来一波总结了

GO语言自从出生以来，一直带着google出品的光环，有人顶，有人踩。不过所有事物都会有争论，这也是正常的，具体怎么样，优点缺点来是要自己体会<!--more-->
### 简洁
简洁应该是我对GO的最大印象，语法简练不墨迹，跟它主要的对手C++相比那是相当明显的，定义不要类型，不要头文件，也不支持一堆特性，就让你踏踏实实写代码就行，在我看来现在冒出来的一堆JS的库就不简洁，这个也要想那个也想要最后写出来的代码看着乱七八糟，用两个东西写出来的简直就像两个语言

但是没有了一些特性当然也会有失去一些便利，GO中有指针，但不支持指针操作，不过好在有`slice`，也没有过模板类，不过我觉得提供的`map`,`slice`,`array`基本上可以解决模板问题，不用关心内存回收问题，也不耽误操作具体的自定义类型，有点像是PHP里的强大的数组的感觉

GO的简洁还是恰到好处的，像python那样连`{}`都没有的，我已经踩了好几个坑了
### 强制性
这个特点我觉得还是挺重要的，我不知道GO是不是第一个严格规定`{`必须写在行尾的语言，这个东西在我看来简直终结了两大门派的正直，不这么写你就不能通过，而且它也不是为了这样才让这么写，只是为了编译时能自动加`;`而已。抛开这些不说增加一些这样语法上的强制性我觉得倒是件好事。统一语法风格，所有人写出来的格式一样，这样对之后看代码的人会有非常好的体验

源码也有要求，不能东放一块，西放一块的，而且和你的package名对应，可以直接从github等上面拉下来直接编译就完了

全静态编译，结果出来就是一个可执行文件，不给选择，也不用纠结去链接别的package
### 工具链
GO语言的工具链很完整，甚至还有`gofmt`这样的东西，而且从github等上面拉代码直接`go get
xxxx`就可以，简直不要太方便，官方还给了一大堆的东西，服务到家
### 编译速度
我现在虽然不再写C了，但还是会记得当初编译的速度，而且现在随便编译个包也得等上一会儿，更不要说debug时候的感觉了，GO把编译速度提升上来绝对是一个非常正确的做法，只是这样相对于C/C++就会少偷懒很多
### 并发
GO确实是为并发而生，`goroutine`说起就起，不起几个都不好意思说自己写的是GO，而且还有`channel`这样比较好的通信解决方案，加上简单的语法，很轻松就可以写出一段来
### GC
这个是把C/C++程序员解放出来的一个东西，不用考虑内存泄露怎么办，什么时候释放，忘了释放怎么办，各种debug检查

不过之前看过几篇文章提到GC的效率，在高并发下GC用时过长会阻塞所有线程，当时好像是`1.4`时代？不知道现在的`1.6`会怎么样
## 总结
总之感觉GO还是值得一学的，起码对于我这样的PHP或者python来看，有速度、语法简单，还能写WEB，这3点就足够了
