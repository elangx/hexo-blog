---
title: 添加php扩展方法
date: 2015-02-11 23:00:24
tags: PHP
---
* 找出php安装位置，可以用`phpinfo()`之类的输出一下，看看你装的php位置，比如我的电脑中有一个mac自带的和一个后来编译过的
* 在解压后的扩展中执行如下命令
```bash
$PHP_PATH/phpize
./configure --with-php-config=$PHP_PATH/php-config
make
sudo make install
```
<!--more-->
* 在你的php.ini中添加对应的extension
```bash
extension=xxx.so
```
* 之后重启php或服务器如果在`phpinfo()`中看到了就说明加载成功了
