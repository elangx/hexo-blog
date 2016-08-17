---
title: PHP中使用xhprof
date: 2016-05-26 19:45:55
tags: PHP
---
### xhprof安装
xhprof的安装与一般php扩展一样，这里直接给出xhprof的地址：[https://github.com/phacility/xhprof](https://github.com/phacility/xhprof)，其中extension目录中的就是它的php扩展
### xhprof使用
安装成功后我们就可以使用相应的函数了，比如`xhprof_enable()`,`xhprof_disable()`这类的，我们得到的包里有一个example目录，是一个使用事例：
<!--more-->
```php
<?php
function bar($x) {
  if ($x > 0) {
    bar($x - 1);
  }
}
function foo() {
  for ($idx = 0; $idx < 5; $idx++) {
    bar($idx);
    $x = strlen("abc");
  }
}
// start profiling
xhprof_enable();
// run program
foo();
// stop profiler
$xhprof_data = xhprof_disable();
// display raw xhprof data for the profiler run
print_r($xhprof_data);
$XHPROF_ROOT = realpath(dirname(__FILE__) .'/..');
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_lib.php";
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_runs.php";
// save raw data for this profiler run using default
// implementation of iXHProfRuns.
$xhprof_runs = new XHProfRuns_Default();
// save the run under a namespace "xhprof_foo"
$run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");
echo "---------------\n".
     "Assuming you have set up the http based UI for \n".
     "XHProf at some address, you can view run at \n".
     "http://<xhprof-ui-address>/index.php?run=$run_id&source=xhprof_foo\n".
     "---------------\n";
```

从这里可以看到xhprof的使用过程，其中`xhprof_enable`与`xhprof_disable`之间的就是要监测的执行代码，这里会监测的就是foo()及foo()内部的bar()函数，并且xhprof_disable会返回数据结果，之后的代码就是将这些数据保存下来，因为我们还需要进一步分析。

这里用`XHProfRuns_Default->save_run()`方法保存数据，它的目录可以在`php.ini`中`xhprof.output_dir`指定（在`xhprof_runs.php`里可以看到，这也是xhprof中唯一一个可设置的参数），这个例子后面还会输出一个`http://<xhprof-ui-address>/index.php?run=$run_id&source=xhprof_foo\n`，这个就是它另外包含的一个简单的可以展示数据的web应用，在目录xhprof_html中，通过访问这个应用可以看到这次请求的具体数据。
### 注入WEB应用
虽然使用xhprof只需要几行代码，但每次还要手动添加还是太麻烦，不过好在有别的方法可以实现通用，我们把需要添加的代码写到一个文件中，通过nginx或php的配置可以实现：
- 在`php.ini`中添加`auto_prepend_file=/path/inject.php`，特点是所有的应用都会自动添加xhprof
- 在`nginx.conf`的应用配置中添加`fastcgi_param PHP_VALUE "auto_prepend_file=/path/inject.php"`;，特点是可以分应用添加
### 更好的数据展示结果--xhrpfo.io
原生所使用的展示方式稍微简陋些，我们也可以自己来操作数据的保存和展示，但这东西肯定有别人已经做好的轮子，我们就不用再造了，这里说的是[xhprof.io](https://github.com/gajus/xhprof.io)，它是把我们的每次的数据保存到数据库中的，所以我们先要新建一个数据库。之后把包里的`setup/database.sql`导入到数据库中，其实看下`xhprof.io`这其实就是一个WEB应用，我们首先把`xhprof/includes/config.inc.smaple.php`这个文件`cp`到`xhprof/includes/config.inc.php`：
```php
<?php
return array(
	'url_base' => 'https://dev.anuary.com/8d50658e-f8e0-5832-9e82-6b9e8aa940ac/',
	'url_static' => null, // When undefined, it defaults to $config['url_base'] . 'public/'. This should be absolute URL.
	'pdo' => new PDO('mysql:dbname=your_database_name;host=localhost;charset=utf8', 'username', 'password'),
    'enable' => function() {
        //enable for 1/100 probability
        //return rand(0, 100) === 42;

        //enable always
        return true;
    }
);
```

把里面的pdo对应的参数改成自己的数据库配置，另外url_base修改为这个应用的访问url，比如`http://localhost:8080/xhpro.io/`，url_static不需要添加，enable那个函数代表需要检测的条件，比如 `return $_GET['xhprof'] === '1'` 或例子中的 `return rand(0, 100) === 42` 这样的百分之一的监测率（线上为了性能，不应该全部监测）。

接下来，类似前面的方法，把`inc/inject.php`注入到你们应用中，这个地方有个坑，这个文件中的 `xhprof_enable(XHPROF_FLAGS_MEMORY | XHPROF_FLAGS_CPU)`这个函数在我的php5.6中会使nginx报502的错，google下发现这是[xhprof的bug](https://www.drupal.org/node/2126573)，修改为`xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_MEMORY | XHPROF_FLAGS_CPU)`后问题解决，之后我们访问xhprof.io后就可以看到我们访问过程中的数据了
![](http://7u2kbh.com1.z0.glb.clouddn.com/20160526201726.png)
