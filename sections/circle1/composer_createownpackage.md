# 面试题目
```
1. 如何创建一个package?
```

有了`Composer`之后，我们就可以自由使用其他人发布的扩展包。现在我们来创建一个自己的`Composer`包，同时发布到`Packagist.org`上，这样其他人就可以使用我们的这个扩展包了。

# 前期准备
- github账号
- packagist.org账号

我们创建的这个包是一个获取梁咏琪(Gigi)音乐的工具包，基于网易云音乐。由于composer已经有了网易云音乐API的扩展包(metowolf/meting)，所以我们可以直接在这个扩展包之上构建我们自己的包。

# 流程
## 1. 在github上创建gigi-music项目
## 2. 克隆到本地
## 3. 初始化

克隆下来之后的`gigi-music`文件夹就是你的包的root目录。composer有一个命令可以帮助我们初始化。在项目的根目录下
```bash
# 在gigi-music目录下
> composer init
```

这个命令会提示你填写一些信息，其中Package name是你将要在packagist.org上注册的包名字，其他人通过这个来获取你的包。如果你的packagist.org的用户名是abc的话，可以填写`abc/gigi-music`。

![composerinit](/assets/circle1/composer_init.png)

初始化之后，root目录下会出现`composer.json`，我们需要设置一下这个包的自动载入映射。

```json
{
    "name": "freelancersinchina/gigi-music",
    "description": "a tool used to get gigi's music",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "freelancersinchina",
            "email": "openclazz@163.com"
        }
    ],
    "require": {},
    "autoload":{
      "psr-4":{
         "Gigi\\":"./src"
       }
    }
}
```
这里比较重要的是`autoload`的配置，这里的意思是使用`psr-4`规范，并且将`Gigi`命名空间映射到`./src`文件夹。

我们这里使用了`Gigi`命名空间。按照`autoload`配置，我们需要再`root`目录下创建`src`目录，作为所有源文件的文件夹。

## 4. 安装依赖项

我们需要`metowolf/meting`这个包，所以我们可以通过下面命令安装。
```
> composer require metowolf/meting
```

如何在我们这个包里面使用这些扩展包呢？其实我们可以直接使用，不需要任何配置。

## 5. 编码

###### src/GigiMusic.php

```php
<?php
namespace Gigi;
class GigiMusic{
	const GigiMusicType_Random = 0;
	const GigiMusicType_Seq = 1;
	private $provider;
	public function __construct($type){
		switch($type){
		case self::GigiMusicType_Random:
			$this->provider = new GigiRandom();
			break;
		case self::GigiMusicType_Sequence:
		default:
			$this->provider = new GigiSequence();
			break;
		}
	}
	public function getFirst( $num=1 ){
		return $this->provider->getFirst($num);
	}
}
```

###### src/GigiRandom.php

这里我们使用了`metowolf\meting`包。
```php
<?php
namespace Gigi;
use \Metowolf\Meting;
class GigiRandom extends MusicProvider{
	public function getFirst($num){
		$data = json_decode($this->api->search($this->singerName),true);

		shuffle($data);
		return $this->processData(array_slice($data,0,$num));
	}
}
```

###### src/GigiSequence
```php
<?php
namespace Gigi;
class GigiSequence extends MusicProvider{
	public function getFirst($num){
		$data = json_decode($this->api->search($this->singerName),true);
		return $this->processData(array_slice($data,0,$num));
	}
}
```

###### src/MusicProvider.php
```php
<?php
namespace Gigi;
use \Metowolf\Meting;
abstract class MusicProvider{
	public $singerName = "梁咏琪";
	protected $api;
	public function __construct(){
		$this->api = (new Meting('netease'))->format(true);
	}
	abstract public function getFirst($num);
	protected function processData($data){
		$self = $this;
		return array_map(function($item) use ($self){
			$item['pic_url'] = json_decode($self->api->pic($item['pic_id']),true)['url'];
			$item['url'] = json_decode($self->api->url($item['url_id']),true)['url'];
			return $item;
		},$data);
	}
}
```

## 6. 提交代码

我们需要提交的文件主要是`composer.json`以及`src`下面的源文件，而`vendor`和`composer.lock`不需要提交。

提交完成之后，我们需要将代码`push`到`github`之上。

## 7. 设置packagist.org，创建新的包，并且同步

访问[https://packagist.org/packages/submit](https://packagist.org/packages/submit)。这个页面会让你填写包对应的github的地址，将github上的地址提交，检查一下,如果没有问题，就代表你在packagist上的包发布成功了。

这里有四个颜色的按钮对这个包进行操作：

- Abandon：点击这个后提示使用者自己不再维护了
- Delete：删除此项目
- Update：从git更新到packagist上
- Edit：修改git项目的地址

每次github上代码更新了，需要在packagist上点击一下update完成同步。

因为`https://packagist.org`在国内是被墙的，所以一般都是使用的国内的镜像源。国内源和packageist同步需要时间，现在大概是一分钟同步一次。也就是说，当你在packagist上update之后，在国内镜像源上需要一分钟之后才能更新。所以如果出现没有找到包或者包没有更新，那么再等等1分钟试试看。

## 8. 安装使用gigi-music包

我们可以按照一般包的安装方法安装自己的包。
```
> composer require freelancersinchina/gigi-music:dev-master
```

因为我们现在是开发版本，所以如果没有添加标签的话，很有可能不能安装，因为低于最小稳定版。

为了正式发布，我们需要到github上，项目release，添加一个标签，比如`v0.1.0`，然后提交。接着在packagist上update一下项目。packagist会自动检出到release版本。

```
require_once("./vendor/autoload.php");
use Gigi;

// get a random list of songs
$songlist = new  GigiMusic(GigiMusic::GigiMusicType_Random);

$count = 10;

// get detail infos
$songs = $songlist->getFirst($count);

print_r($songs);
```

这样子我们就安装使用了我们自己的composer包了。

# 如何让packagist自动同步更新github

每一次更新代码都需要自己手动update packagist，这样比较麻烦。幸好github提供了自动更新服务。

1. 你需要到github上对==这个项目==进行设置——Settings里Add Service。
2. 选择左侧的Webhooks & service，然后点击右侧Add Service按钮，然后选择下拉选项中的Packagist，这时需要你输入github的密码继续操作。
3. 接着你需要填入以下3条信息，Token信息全部可以从https://packagist.org/profile/获取——Show API Token。User填你在packagist的用户名。Token填Show API Token之后的key：****************。Domain填https://packagist.org。

# 如何安装一个本地的package

修改composer.json文件，添加repositories属性。
```
{
    "repositories": [
        {
            "type": "path",
            "url": "../../packages/my-package"
        }
    ]
}
```
修改完后我们就可以在本地执行`composer require`,composer会从本地文件中做个软连接到vendor目录下。 用这种方式可以非常方便的==测试==的你的package,不需要等到发布到packagist上才测试。

# 相关链接
- [什么叫做`psr-4`](https://laravel-china.org/topics/2081/psr-specification-psr-4-automatic-loading-specification)
- [composer使用文档](http://docs.phpcomposer.com)
