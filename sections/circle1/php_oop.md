# 面试题目
```
1. 什么是静态延迟绑定？
```
PHP支持面向对象开发，在这一篇中，我们将深入了解PHP这方面的支持。包括了以下的内容：

1. 抽象类和接口：设计和实现相分离
2. 延迟静态绑定：static和self
3. 错误处理：异常
4. 拦截器方法
5. 回调、匿名函数和闭包
6. 类函数和对象函数
7. 反射API

# 抽象类和接口

抽象类是抽象的类型，是相对于具体而言的，一般而言，具体类有直接的对象，而抽象类没有，它表达的是抽象概念。比如Shape就是一个抽象类，Square继承Shape类，表明一个具体类。

```php
abstract class Shape{
   abstract function draw();
}

class Square extends Shape{
  function draw(){
    echo "in Square::draw()";
  }
}

// var box = new Shape() ; 错误，抽象类不能被实例
var box = new Square();
box.draw();
```

接口是表示某一种能力，Drawable接口表示实现这个接口的类具有draw能力，Writable接口表示实现这个接口的类具有write能力等。

```php
interface Drawable{
   public function draw();
}

interface Writable{
  public function write();
}

class Painter implements Drawable,Writable{
  public function draw(){
    echo "Painter can draw";
  }

  public function write(){
    echo "Painter can write";
  }
}

var painter = new Painter();
painter.draw();
painter.write();
```

## 题目

在Yii2框架中，所有组件(Component)都是继承自Object，而Object是具有可配置(Configurable)，Widget是其中的一种组件。请使用面向对象的方法设计出相关类和接口。

```php
interface Configurable{ }

class Object implements Configurable{ }

class Component extends Object{ }

class Widget extends Component{ }
```

# 延迟静态绑定：static和self的区别

## 题目

修改下面的类，使得代码简洁。

```php
abstract class DomainObject{}

class User extends DomainObject{
  public static function create(){ return new User() ; }
}

class Document extends DomainObject{
  public static function create(){ return new Document(); }
}
```

```php
abstract class DomainObject{
  public static function create(){ /* 请在这里填写代码 */ }
}

class User extends DomainObject{ }
class Document extends DomainObject{ }

// 测试
var u = User::create();
if( u instanceof User )
    echo "You are right !";
else
    echo "Sorry ,the answer is not right ";

var doc = Document::create();
if( doc instanceof Document )
    echo "You are right !";
else
    echo "Sorry,the answer is not right ";
```

从上面的代码，可以看出self是对该类的引用，是解析上下文，而不是调用上下文。也就是说，self指的是self出现的那个类，所以当上面空白处填写的是`return new self(); `的时候，每次调用`User::create()`的时候，会调用`DomainObject::create()`，而这里的Self指向的是DomainObject，所以`User::create()`产生的是DomainObject实例，而不是User实例。

PHP5.3引入了延迟静态绑定概念，使用static关键字，作用类似于self，但是指的是被调用的类，而不是定义的类。所以空白处填写的是`return new static();`，每次调用`User::create()`的时候，都会讲static绑定到User（被调用的类），产生一个User实例。

# 异常处理

## 问题

填写下面代码

```php
try{
   if(!file_exists('/etc/conf'))
      throw new Exception("file does not exist");
   $conf = Conf::load('/etc/conf');  // May throw BadConfException
}
catch( BadConfException $e){

   /* 打印出消息字符串，错误代码，异常文建，异常行号 */
}
catch(Exception $e){
  /* 打印出消息字符串，错误代码，异常文建，异常行号 */
}

```
异常主要具有以下的方法:

- getMessage()
- getCode()
- getFile()
- getLine()

# 拦截器方法

PHP的拦截器方法

- __get($property)   访问未定义的属性时被调用
- __set($property,$value)  给未定义的属性赋值是被调用
- __isset($property)     给未定义的属性调用isset()时被调用
- __unset($property)     给未定义的属性调用unset()时被调用
- __call($method,$arg_array)  调用未定义的方法时被调用

###问题

设计两个类,PersonWriter和Person,Person类中有一个私有属性PersonWriter，公共属性名字。PersonWriter用于打印Person的信息。将对Person调用的不存在方法都转发到PersonWriter上。

```php
class PersonWriter{
  public writeName(Person $p){
    echo $p->getName();
  }
}

class Person{
 private $writer;
 private $name;
 public function __construct(PersonWriter $w,$name){
   $this->writer = $w;
   $this->name = $name;
 }

 public function getName(){
    return $this->name;
 }

 public function __call($method,$arg_arr){
   if(method_exists($this->writer,$method)){
     array_unshift($arg_arr,$this);
     return call_user_func_array(array($this->writer,$method),$arg_arr);
   }
 }
}

$writer = new PersonWriter();
$p = new Person($writer);
$p->writeName();
```

除了上面的拦截器，还有一个特殊的方法：__clone。__clone()是在使用clone关键字的时候在肤质得到的对象上被调用，采用了值复制。

```php
class Person{
  private $name;
  private $age;
  private $id;

 function __construct($name,$age){
  $this->name = $name;
  $this->age = $age;
 }

 function setId($id){
  $this->id = $id;
 }

 // clone调用这个__clone之前，值复制，得到内容一样的对象，然后通过__clone来控制接下来需要修改的内容
 function __clone(){
  $this->id = 0;
 }
}

$p1 = new Person('bbb',21);
$p2 = clone $p1;

var_dump($p1);
var_dump($p2);
```

# 回调、匿名函数和闭包

回调在PHP代码中被广泛使用，在各种框架中更是非常常见，正是利用了回调，所以框架开发者才能将具体业务处理和框架进行分离，使得框架具有很大的扩展性。比如在Yii2中，Action都具有`beforeAction`和`AfterAction`这些Hook，Hook的实现离不开回调函数。

## 问题

现在有一个产品(Product)，销售人员需要在销售之前和之后对产品进行一些处理，比如销售之前记录一下产品的ID，销售之后记录一下销售的时间等等。这些处理在之前是不知道的，随时可以进行修改。现在请设计一下这两个类。

```php
class Product{
  public $id;
  public $category;

 function __construct($category,$id){
   $this->id = $id;
   $this->category = $category;
 }
}

class ProcessSale{

  // 添加一下属性

  // 添加处理方法注册函数

  // 添加sale方法
}

$processor = new ProcessSale();
$processor->addBeforeSaleHandle(function ($product){
   echo "before sale:{$product->category},{$product->id}";
});
$processor->addAfterSaleHandle(function($product){
 echo "after sale:{$product->category},{$product->id}";
})

$processor->sale(new Product('Shoes',1));
$processor->sale(new Product('coffee',2));
```

一般我们都是设置一个数组，用来存放回调函数，通过is_callable()函数来确保传递进来的值都是能够被call_user_func()或者array_walk()等函数调用的。

回调有什么用？利用回调，你可以在运行时将于组件核心任务没有直接关系的功能插入到组件中。有了组件，你就赋予其他人在你不知道的上下文中扩展你的代码的权利。

回调函数的创建方法有很多。
```php
$logger = function($product){
  echo "logger";
}
```
```php
class Mailer{
  public doMail($product){
   echo "send Mail";
 }
}

// 以下是通过传入对象的方法
$processor->addAfterSaleHandle(array(new Mailer(),"doMail"));
```

# 类函数和对象函数

PHP提供了一系列强大的函数来检查类和对象。为什么需要这些东西呢？因为在PHP程序运行时你可能无法知道你正在使用的是哪一个类。比如以下的例子。

```php

// Task.php
namespace tasks;

class Task{
   function doSpeak(){
    echo "hello\n";
  }
}

// TaskRunner.php
$className = "Task";

require_once("tasks/{$className}.php");

$classname = "task\{$className}";

$myObj = new $classname();
$myObj->doSpeak();

```

这里面存在很多导致异常的地方，在实际开发过程中我们需要检查类是否存在、它是否具有某个将要使用的方法，以避免可能存在的危险。

class_exists()来查看某个类是否存在，get_class()来确定对象的类型。get_class_methods()来获得一个类的所有方法，get_class_vars()俩获得一个类的属性的关联数组,get_parent_class()来获得类的父类，如果没有返回false,is_subclass_of($subclassObj,$parentClassName)来返回是否为子类。call_user_func和call_user_func_array用来调用用户函数。

## 问题

修改上面的代码，增加代码的鲁棒性。

```php
// TaskRunner.php
$className = "Task";

if(file_exists("tasks/{$className}.php")
   require_once("tasks/{$className}.php");
else
  throw new Exception("file tasks/{${className}.php not found");

$classname = "task\{$className}";

if(class_exists($classname))
    $myObj = new $classname();
else
  throw new Exception("class tasks\{$className} not exist ");

if(methods_exists($myObj,"doSpeak"))
  $myObj->doSpeak();
else
  throw new Exception("class tasks\{$className} do not have method: doSpeak");

```

# 反射API

反射API和类函数对象函数功能类似，但是更加强大，能够获得更加多的信息。

反射API中的类有以下部分，每一个反射类都代表了某一个类型：

- Reflection
- ReflectionClass
- ReflectionMethod
- ReflectionParameter
- ReflectionProperty
- ReflectionFunction
- ReflectionExtension
- ReflectionException

想要某一个类的相关信息，可以通过
```php
$prod_class = new ReflectionClass('Product');
Reflection::export($prod_class');

// 检查方法,$methods是一个ReflectionMethod的数组
$methods = $prod_class->getMethods();

$method = $methods[0];

$path = $method->getFileName();
$lines = @file($path);  // content splited by line
$from = $method->getStartLine();
$to = $method->getEndLine();
$len = $to-$from +1;

// source of method
echo implode(array_slice($lines,$from-1,$len));

$params = $method->getParameters();

// suppose method has at least one params
$param = $params[0];

$name = $param->getName();
$class = $param->getClass();
$className = $class->getClassName();
$position = $param->getPosition();
```

## 例子

现在我们要创建一个类来动态调用Module对象，即该类可以自由加载第三方插件并且集成到现有的系统中，而不需要把第三方硬编码到现有代码中。要实现这个功能，我们首先可以创建一个抽象类或者接口Module，具有execute方法，强制每个子类都必须实现这个方法。系统可以通过`XML`或者数据库等方法来加载需要的Module对象，然后对每个Module执行execute()。

现在实现两个模块，一个PersonModule用来管理用户，另外一个FtpModule用来实现Ftp的功能。

// 应该由读者实现
```php
class Person{
  public $name;
  function __construct($name){
    $this->name = $name;
  }
}

/* 请在这里声明Module 接口 */
interface Module{
  public function execute();
}

/* 请在这里声明PersonModule和FtpModule */

class PersonModule implements Module{

  private $person;
  function execute(){
    // do something here
  }

  function setPerson(Person $person){
   $this->person = $person;
 }
}

class FtpModule implements Module{

  private $host;
  private $password;
  private $user;
  function execute(){
    // do something here
  }

  function setHost($host){
    $this->host = $host;
  }

  function setPassword($password){
   $this->password = $password;
  }

  function setUser(Person $user){
   $this->user = $user;
  }
}

```

有了模块之后，我们需要一个模块运行器来运行这些模块。模块运行器在运行模块之前需要利用反射API来确保这些模块和相应的调用方法都被正确的使用和调用。一般模块都是可以配置的，这些配置可以从数据库中获得，也可以写在配置文件中，我们这里简单地把配置写在ModuleRunner里面。

```php

class ModuleRunner{

   private $configData = array(
                  "PersonModule"=>array('person'=>'bob'),  
                  "FtpModule"=>array('host'=>'example.com',
                                     'password'=>'anon'));
   private $modules = array(); // store module instances;

   // init()，包含了模块的实例化，检查模块和方法合法性，运行模块的execute
   public function init(){
    $interface = new ReflectionClass('Module');

    foreach( $this->configData as $modulename=>$params){
      $module_class = new ReflectionClass($modulename);

      // 首先确保module是实现了Module接口
      if(! $module_class->isSubclassOf($interface) ){
        throw new Exception("unknown module type:{$modulename}");
      }

      $module = $module_class->newInstance();
      foreach( $module_class->getMethods() as $method){

        $this->handleMethod($module,$method,$params);
      }


    }

   private function handleMethod(Module $module,ReflectionMethod $method,$params){

     $name = $method->getName();
     $args = $method->getParameters();

     if( count($args) !=1|| substr($name,0,3) != "set")
       reurn false;

     $property = strtolower(substr($name,0,3));
     if(! isset($params[$property])) return false;
     $arg_class = $args[0]->getClass();
     if(empty($arg_class))
       $method->invoke($module,$params[$property]);
    else{
       $method->invoke($module,$arg_class->newInstance($params[$property]));
    }
   }
}
```
