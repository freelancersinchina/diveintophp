# 面试题目
```
1. 如何获取命令行参数？获取用户输入？
2. 如何在不用回车的情况下实时读取输入？
```

# 命令行程序开发

PHP的其中一个运行模式是CLI，所以我们可以利用PHP开发脚本，处理任务。

## 输入输出

作为程序，最重要的是输入输出功能。在CLI运行模式下，PHP可以通过特定的方法来获得输入输出能力。PHP提供了几个专门用于CLI模式的常量，`STDIN`,`STDOUT`,`STDERR`。

```php
<?php
// 获取输入句柄
$stdin = fopen("php://stdin",'r');

// 获取输出句柄
$stdout = fopen("php://stdout","w");

// 获取错误输出句柄
$stderr = fopen("php://stderr","w");

// 获取了上面的句柄之后，就可以像操作文件一样了。
// 获取输入的一行
$line = fgets($stdin);
echo $line;

// 输出一句话到标准输出上
fwrite($stdin,"Hello World");

?>
```

## 获取参数

PHP可以通过`$argc`和`$argv`来获取到参数个数和参数数组。`$argv`是一个参数数组，第0个是脚本文件的文件名，如果是直接运行的代码，那么第0个是`-`，这种情况下，如果要传入`-`开头的选项，需要先在这个参数前面加入 `--`，否则会出现错误。

以命令行方式运行的PHP脚本
```
#!/usr/bin/php
<?php
if ($argc != 2 || in_array($argv[1], array('--help', '-help', '-h', '-?'))) {
?>
This is a command line PHP script with one option.

  Usage:
  <?php echo $argv[0]; ?> <option>

  <option> can be some word you would like
  to print out. With the --help, -help, -h,
  or -? options, you can get this help.
<?php
} else {
    echo $argv[1];
}
?>

```

更多内容正在编辑中
