# 面试
```
```

filter在数据被读取或者被写入的时候会对数据进行处理。对于一个流，应用在上面的filter是没有限制的。我们可以使用PHP內建的filter，也可以自己创建自定义filter，然后利用PHP提供的`stream_filter_register`注册使用。

获取现在可以使用的filter，我们可以利用函数`stream_get_filters()`，会返回一个可用filter组成的数组。

# 使用filters

PHP有一些內建的filter，比如`string.toupper`,`string.tolower`和`string.strip_tags`。另外一些扩展也会提供一些自己的filter，比如mcrypt扩展就提供了`mcrypt.*`和`mdecrypt.*`。

对于一个现有的流，我们可以通过`stream_filter_append()`来增加filter。

```php
$h = fopen('somefile.txt','r');
stream_filter_append($h,'convert.base64-encode');
fpassthru($h);
fclose($h);
```
或者使用另外一种

```php
$filter = 'convert.base64-encode';
$file = 'somefile.txt';
$h = fopen("php://filter/read={$filter}/resource={$file}",'r');
fpassthru($h);
fclose($h);
```

# 创建自己的filter

我们可以利用PHP提供的`php_user_filter`来创建自己的filter，我们需要做的是继承这个类，然后对数据做处理，返回就可以了。


下面我们以一个Markdown filter来作为例子讲解一下如何创建自己的filter。这个filter是用来将markdown文本转换为html。

这里面最重要的是一个方法`filter()`，这个方法比stream调用，同时会收到四个参数：

- `$in`：指向装有数据的buckets
- `$out`：指向将会装有被转换数据的buckets
- `$consumed`：是一个引用，这个参数在处理完成之后必须增加被转换数据的长度
- `$closing`：如果stream已经到了最后，将会结束的时候，这个参数回事TRUE

另外两个比较重要的方法是`onClose()`和`onCreate()`，是在关闭和创建流的时候被调用。这两个方法在初始化资源的方面很有用，比如创建其他流或者数据buffer。

我们这个filter需要使用临时数据存储，我们可以使用`php://temp`这个流，所以我们需要在`onCreate`的时候创建这个流，并且在`onClose`的时候关闭这个流。这个filter中使用了其他人实现的[markdown解析工具](https://github.com/michelf/php-markdown)。

```php
<?php
namespace MarkdownFilter;

use \Michelf\MarkdownExtra as MarkdownExtra;

class MarkdownFilter extends \php_user_filter
{

    private $bufferHandle = '';

    public function filter($in, $out, &$consumed, $closing)
    {
        $data = '';

        while ($bucket = stream_bucket_make_writeable($in)) {
            $data .= $bucket->data;
            $consumed += $bucket->datalen;
        }

        $buck = stream_bucket_new($this->bufferHandle, '');

        if (false === $buck) {
            return PSFS_ERR_FATAL;
        }

        $parser = new MarkdownExtra;
        $html = $parser->transform($data);
        $buck->data = $html;

        stream_bucket_append($out, $buck);
        return PSFS_PASS_ON;
    }

    public function onCreate()
    {
        $this->bufferHandle = @fopen('php://temp', 'w+');
        if (false !== $this->bufferHandle) {
            return true;
        }
        return false;
    }

    public function onClose()
    {
        @fclose($this->bufferHandle);
    }
}
```


在filter()中，首先我们将内容都收集到`$data`中。在第一个循环中，我们使用了`stream_bucket_make_writeable()`来收集数据，每一个bucket都有`data`和`datalen`两个属性。

当所有的数据收集完成，我们需要新的空的bucket来将转换后的数据添加到输出流中。我们使用`stream_bucket_create()`来完成这个功能，如果这个操作失败就返回`PSFS_ERR_FATAL`，这个会触发一个`filter error`。因为这个方法需要一个资源指针，所以我们在`onCreate`的时候就创建了一个临时流。

将转换之后的数据添加到到bucket之后，我们需要将这个bucket添加到输出流中去，也就是`stream_bucket_append`，然后返回一个`PSFS_PASS_ON`，表示处理成功。

```php
    // Require the MarkdownFilter or autoload

    // Register the filter
    stream_filter_register("markdown", "\MarkdownFilter\MarkdownFilter")
        or die("Failed to register filter Markdown");

    // Apply the filter
    $content = file_get_contents(
        'php://filter/read=markdown/resource=file:///path/to/somefile.md'
    );

    // Check for success...
    if (false === $content) {
        echo "Unable to read from source\n";
        exit(1);
    }

    // ...and enjoy the results
    echo $content, "\n";
```

这样我们就完成了一个filter，这个filter应用在读取数据的时候。同样的我们可以创建一个写数据时候处理数据的流，创建方法类似。

# 总结

使用自己创建的流包装器和filter可以将数据的处理封装成自己的协议，这样将会简化代码，提高了代码的易读性。
