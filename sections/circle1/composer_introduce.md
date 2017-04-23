# 面试题目
```
1. composer安装依赖包的流程？
```
Composer是PHP用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。这个功能就像是NodeJS中的npm，Python中的pip。


# 安装

安装Composer，只需要在控制台运行一下命令：
```
curl -sS https://getcomposer.org/installer | php
```

运行完成之后，在当前目录下会出现一个`composer.phar`文件，这是一个二进制文件，是PHP的归档文件。检查Composer是否正常工作，通过PHP来执行这个phar文件，这个命令将会返回一个可执行的命令列表。

```
php composer.phar
```

现在`composer.phar`文件在当前目录，为了让在任何地方都能够使用到composer命令，你只需要将`composer.phar`放到任何一处地方，然后将这个地方放入到Path环境变量里面去。比较常见的地方是`/usr/local/bin/`文件夹。

在当前目录下运行：
```
> mv composer.phar /usr/local/bin  
> ln -s /usr/local/bin/composer.phar /usr/local/bin/composer
```

这样的话，在任何地方都只需要运行`composer`就可以了

# 几个命令

- composer install

  如有`composer.lock`文件，每一个包都按照里面==锁定的版本号==进行安装，否则按照`composer.json`里面的安装最新版本的扩展包和依赖。

- composer update

  从`composer.json`安装最新的扩展包和依赖。
- composer require packagename

  安装扩展包，可以指定版本，并且会将新安装包的名字和版本号添加到`composer.json`里面。比如下面是安装`yiisoft`，并且指定了版本号。
```bash
composer require yiisoft ~2.*
```

# 使用扩展包

composer会将依赖扩展包安装在项目根目录下的`vendor`文件夹下，每一个文件夹就是一个扩展包。`composer`提供了一个自动加载文件`vendor/autoload.php`，用来自动加载扩展包。

```php
require 'vendor/autoload.php';
```

这样你就可以自由地使用第三方代码了。例如，如果你的项目依赖monolog，你就可以像这样开始使用这个类库，并且他们将被自动加载进来。

```php
$log = new Monolog\Logger('name');
$log->pushHandler(new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING));

$log->addWarning('Foo');
```

# 流程

下面是几个日常生产中composer常用的使用流程。

## 流程一：新项目创建
 1. 创建`composer.json`，并且添加依赖的扩展包
 2. 运行`composer install`，composer会根据`composer.json`里面的申明安装依赖项，并且生成`composer.lock`，里面是每个依赖项的安装时的具体版本号。
 3. 将`composer.json`和`composer.lock`添加到版本控制器中。

## 流程二：项目协作者安装项目
 1. 克隆项目之后，在根目录下直接运行`composer install`，composer会从`composer.lock`中安装==指定版本==的扩展包和依赖项。

```
这个流程适用于生产环境的代码部署
```

## 流程三：为项目添加新扩展包
 1. 开发过程中，发现需要新的扩展包，通过`composer require packagename`添加。
 2. 提交更新后的`composer.json`和`composer.lock`到代码版本控制器中。



# 几个问题

1. **composer.json和composer.lock的区别是什么？**

 `composer.json`是声明依赖项，有时候依赖项的版本号是模糊的，比如`1.0.*`、`*`，所以这里带来了不确定性，有时候依赖的扩展包更新，接口发生了变化，那么在没有`composer.lock`文件的情况下，就会安装最新的版本，极有可能会产生错误。

  `composer.lock`中记录了每一个安装的依赖扩展包的具体版本号，在运行`composer install`的时候，首先就会查看`composer.lock`是否存在，如果存在，就按照里面指定的具体版本号安装，否则就按照`composer.json`来安装最新版本，并且创建锁文件。

  如果你更新了依赖项，`composer.json`和`composer.lock`都会将相应的依赖项更新到最新的版本。

2. **我从哪里可以搜索到我需要的依赖包?**

  [packagist](https://packagist.org) 是 Composer 的主要资源库，你可以搜索浏览自己想要的任何依赖项。 一个 Composer 的库基本上是一个包的源：记录了可以得到包的地方。Packagist 的目标是成为大家使用库资源的中央存储平台。这意味着你可以 require 那里的任何包。

  任何支持 Composer 的开源项目应该发布自己的包在 packagist 上。虽然并不一定要发布在 packagist 上来使用 Composer，但它使我们的编程生活更加轻松。


# 相关资源
- [phpcomposer.com](http://docs.phpcomposer.com)
- [Packagist](https://packagist.org)
