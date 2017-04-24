# 面试题目
```
1. 不使用curl，如何读取一个网页？
2. 如何读取请求 payload里面的数据？
```

`Streams`对于文件、网络、数据压缩以及其他操作提供了一系列公共的方法，为操作这些资源提供了简便方法。最常见的`Streams`就是我们通过`fopen()`获取的文件操作句柄。

PHP中有多种Stream类型，每一种都具有特定的代码来解决特定的协议和编码。PHP提供了几个内建包装器，同时我们也很容易地创建自己的包装器，甚至我们可以利用context和filter来修改或者增强现有包装器的功能。

# stream基础
一个流是通过 `schema://target`来表示的，schema表示包装器的名字，而target依据每种包装器的语法有不同的内容。

默认的包装器是file://，所以意味着我们每次使用文件系统的时候，我们都使用了包装器。`readfile('/path/to/somefile.txt')`也可以写成`readfile('file:///path/to/somefile.txt')`。如果我们使用`readfile('http://google.com')`，我们就告诉PHP使用`HTPP`流的包装器。

PHP提供了几个內建的包装器、协议和过滤器，想要知道机器上装了哪些，我们可以使用以下的方法：

```php
<?php
print_r(stream_get_transports());
print_r(stream_get_wrappers());
print_r(stream_get_filters());
```

另外，我们可以自己创建或者使用其他创建好的包装器，比如`Amazon S3`，`MS Excel`等。

## PHP的包装器

- php://stdin,php://stdout,php://stderr
- php://input
- php://temp,php://memory
- php://filter


PHP通过包装器来操作I/O流。使用`php://stdin`，`php://stdout`，`php://stderr`来操作最基本的输入输出资源。

可以使用`php://input`来获取`POST`请求中的body的原始数据，因为有时候客户端会把POST请求的数据放在POST的payloads里，这时候使用`$_POST[]`是没法取到这些数据，这时候需要使用`php://input`。下面是一个例子：

```bash
> curl -d "Hello World" -d "foo=bar&name=John" http://localhost/dev/streams/php_input.php
```

这时候，我们使用`print_r($_POST)`取到的数据是
```php
Array
(
    [foo] => bar
    [name] => John
)
```

这里面没有获取到第一个数据，因为这个数据是放在payloads里面的。使用`php://input`来代替获取数据，`readfile("php://input")`，我们就能够得到

```bash
Hello World&foo=bar&name=John
```

PHP5.1引入了`php://memory`和`php://temp`两个包装器，用来读去和写入临时的数据。就像命名提示的一样，一个是利用内存，另外一个是利用临时文件。

还有`php://filter`，是一个元包装器，使用`readfile()`，`file_get_contens()`/`stream_get_contens()`打开流的时候用来应用一些`filter`。

```php
<?php
// 写入编码过的数据
file_put_contents("php://filter/write=string.rot13/resource=file:///path/to/somefile.txt","Hello World");

// 读取数据，并且解编码
readfile("php://filter/read=string.toupper|string.rot13/resource=http://www.google.com");
```

## Stream的上下文

Stream的上下文(context)是指包含了特定包装器选项的数组，可以用来指定包装器的行为。context最常用的地方是修改HTTP包装器的行为，下面是不使用cURL的情况下进行的网络操作。

```php
<?php
$opts = array(
  'http'=>array(
    'method'=>"POST",
    'header'=> "Auth: SecretAuthTokenrn" .
        "Content-type: application/x-www-form-urlencodedrn" .
              "Content-length: " . strlen("Hello World"),
    'content' => 'Hello World'
  )
);
$default = stream_context_get_default($opts);
readfile('http://localhost/dev/streams/php_input.php');
```

首先我们定义了一个option数组，这个数组里面可以包含各种包装器的选项，使用索引来进行区别。然后使用`stream_context_get_default()`来返回一个默认的上下文，`readfile`就会使用这个默认的上下文。

在这个例子中，content是在请求的body中，所以服务器通过`php://input`来获取所有的内容。另外我们可以通过`apache_request_headers()`来获取关于头的信息，会得到一下的信息：

```bash
Array
(
    [Host] => localhost
    [Auth] => SecretAuthToken
    [Content-type] => application/x-www-form-urlencoded
    [Content-length] => 11
)
```

上面的方法`stream_context_get_default()`会修改掉默认的上下文，对于其他包装器也会产生影响，所以最好的方法是我们创建一个代替的上下文，和默认上下文分离开来。

```php
<?php
$opts = array(
  'http'=>array(
    'method'=>"POST",
    'header'=> "Auth: SecretAuthTokenrn" .
        "Content-type: application/x-www-form-urlencodedrn" .
              "Content-length: " . strlen("Hello World"),
    'content' => 'Hello World'
  )
);
$alternative = stream_context_create($opts);
readfile('http://localhost/dev/streams/php_input.php', false, $alternative);
```

# 接下来

这篇中我们学习了Stream的基础知识，接下来我们将会学习到如何创建一个包装器以及如何创造一个filter，这些知识将会将强我们对流的控制，实现我们个性化的格式和编码。
