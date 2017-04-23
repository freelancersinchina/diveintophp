# 面试题目
```
1. 你知道PHP各个版本的区别么？都增加了什么内容？
2. 你了解PHP7么？增加了什么新特性？
```

# PHP版本的认识

由于PHP4对于OOP支持力度不够，已经被淘汰，所以这里讨论的是PHP5.x以及PHP7。

## PHP5.2
- 弃用__autoload()
- PDO,MySQLi
- 类型约束
- 增加了json_encode和json_decode

在5.2之前，通过`__autoload()`函数来自动加载，在5.2中这个函数不再建议，因为一个项目中仅能只有一个这样的`__autoload()`函数，用来代替的是`spl_autoload_register()`。

```php
spl_autoload_register(function($classname){
  require_once("{$classname}.php");
});
```

5.2也对JSON提供了支持，提供了两个函数`json_encode()`和`json_decode()`。

## PHP5.3
- 弃用register_global方法
- 弃用Magic Quotes(自动转移用户输入）
- 弃用Safe Mode
- 匿名函数
- ==命名空间==
- Heredoc和Nowdoc
- 后期静态绑定
- const 可以用在class外部，像define，但不能进行运算
- 三元运算符简写


### 魔术方法
新增了魔法方法:`__invoke()`,`__callStatic()`方法。

```php
<?php
class CallableClass
{
    public function __invoke($x)
    {
        var_dump($x);
    }
}
$obj = new CallableClass;
$obj(5);
var_dump(is_callable($obj));
?>
```

```php
<?php
class MethodTest
{
    public function __call($name, $arguments)
    {
        // Note: value of $name is case sensitive.
        echo "Calling object method '$name' "
             . implode(', ', $arguments). "\n";
    }

    /**  As of PHP 5.3.0  */
    public static function __callStatic($name, $arguments)
    {
        // Note: value of $name is case sensitive.
        echo "Calling static method '$name' "
             . implode(', ', $arguments). "\n";
    }
}

$obj = new MethodTest;

// 调用__call
$obj->runTest('in object context');

// 调用__callStatic
MethodTest::runTest('in static context');  
?>
```

### Heredoc和Nowdoc

Nowdoc
```php
$name = "MyName";
echo <<< TEXT
My name is "{$name}".
TEXT;
```

Heredoc
```php
$name = "MyName";
echo <<< 'TEXT'
My name is "{$name}".
TEXT;
```
Nowdoc 的行为像一个单引号字符串，不能在其中嵌入变量，和 Heredoc 唯一的区别就是，三个左尖括号后的标识符要以单引号括起来。

### 后期静态绑定
```php
class ATest {
    public function say()
    {
        echo 'Segmentfault';
    }
    public function callSelf()
    {
        self::say();
    }
    public function callStatic()
    {
        static::say();
    }
}

class BTest extends ATest {
    public function say()
    {
        echo 'PHP';
    }
}
$b = new BTest();
$b->say(); // output: php
$b->callSelf(); // output: segmentfault
$b->callStatic(); // output: php
```

## PHP5.4
- Short Open Tag自5.4起总是可用
- 数组简写
- trait
- **内置web服务器**

PHP5.4提供了一个web服务器，主要用来本地开发，不可用于线上产品环境。URI请求被发送到PHP所在的的工作目录（Working Directory）进行处理，除非你使用了-t参数来自定义不同的目录。

当你在命令行启动这个Web Server时，如果指定了一个PHP文件，则这个文件会作为一个“路由”脚本，意味着每次请求都会先执行这个脚本。如果这个脚本返回 FALSE ，那么直接返回请求的文件（例如请求静态文件不作任何处理）。否则会把输出返回到浏览器。

```bash
$ cd ~/public_html
$ php -S localhost:8000
```


## PHP5.5
- **yield关键词**
- list()可用于foreach

yield关键字用于当函数需要返回一个迭代器的时候, 逐个返回值。

```php
<?php
function gen_one_to_three() {
    for ($i = 1; $i <= 3; $i++) {
        //注意变量$i的值在不同的yield之间是保持传递的。
        yield $i;
    }
}
$generator = gen_one_to_three();
foreach ($generator as $value) {
    echo "$value\n";
}
?>
```

list可以用于foreach:
```php
<?php
$array = [
    [1, 2, 3],
    [4, 5, 6],
];
foreach ($array as list($a, $b, $c))
    echo "{$a} {$b} {$c}\n";
?>
```

## PHP5.6
- const支持计算
- **更好地可变函数参数用于代替func_get_args()**
- 命名空间增强，里面可以定义常量和方法

在PHP5.6之前，可变参数都是通过func_get_args(),func_num_args()来实现的。5.6之后可以通过`...`来实现。

使用`...`来导入参数，`$numbers`是一个array。
```php
<?php
function sum(...$numbers) {
    $acc = 0;
    foreach ($numbers as $n) {
        $acc += $n;
    }
    return $acc;
}

echo sum(1, 2, 3, 4);
?>
```

当有一个数组或者Traversable的对象传入到多个参数的函数时，可以用`...`来解压对象。
```php
<?php
function add($a, $b) {
    return $a + $b;
}

echo add(...[1, 2])."\n";

$a = [1, 2];
echo add(...$a);
?>
```
可以在`...`之前添加类型，约束传入参数都是同一种类型。

```php
<?php
function sumOfInts(int ...$numbers){
  return array_reduce(function($sum,$number){
      return $sum + $number;
  },$numbers);
}

echo sumOfInts(1,2,3,4,5);
echo sumOfInts(0,1,2,6,7);
?>
```


# PHP7
PHP7是最新版本的PHP，经过社区的努力，PHP性能得到了大幅度的提升，在相同条件下，性能是PHP5的一倍，甚至比HHVM性能还好。之前由于没有支持Windows上的64位数字和大文件，所以之前x64版本的PHP一直被认为是实验性的。现在PHP7全面支持64位，所以能够在Windows安全地运行PHP7了。除了这些特性，PHP7给语言增加了很多新的语法特性。

## 新运算符

### 1. ??运算符
```php
<?php
// 如果$_GET['user']不存在，则返回'nobody'
$username = $_GET['user']??'nobody';
// 等同于
$username = isset($_GET['user'])?$_GET['user']:'nobody';
// ??可以连起来使用，返回第一个定义的值
$username = $_GET['user']??$_POST['user']??'nobody';

?>
```

### 2. <=>比较运算符
比较两个表达式的值，三种关系: =返回0，<返回-1，>返回1.

```php
// Integers
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1
// Floats
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
// Strings
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
```

## 类型相关

### 1. 标量参数类型声明
现在支持字符串(string)、整型(int)、浮点数(float)、及布尔型(bool)参数声明，以前只支持类名、接口、数组及Callable。两种风格：强制转换模式（默认）与严格模式。

```php
function sumOfInts(int ...$ints){
  return array_sum($ints);
}
var_dump(sumOfInts(2,'3',4.1))
```

### 2. 返回类型声明

```php
<?php
function arraysSum(array ...$arrays):array{
  return array_map(function($array):int{
      return array_sum($array);
  },$arrays);
}
print_r(arraySum([1,2,3],[4,5,6],[7,8,9]))
?>
```

### 3. define支持定义数组类型的值
php 5.6已经支持CONST 语法定义数组类的常量，PHP7中支持define语法。

```php
<?php
define('ANIMALS', [
'dog',
'cat',
'bird'
]);
echo ANIMALS[1]; // outputs "cat"
```

### 4. 匿名类
```php
<?php
interface Logger {
   public function log(string $msg);
}
class Application {
   private $logger;
   public function getLogger(): Logger {
     return $this->logger;
   }
   public function setLogger(Logger $logger) {
     $this->logger = $logger;
   }
}
$app = new Application;
$app->setLogger(new class implements Logger {
     public function log(string $msg) {
        echo $msg;
     }
});
var_dump($app->getLogger());
```
