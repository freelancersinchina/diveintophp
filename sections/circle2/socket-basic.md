# 面试题目
```
1. 
2. 
```

# 使用方式

PHP中使用socket有两种方法：

1. stream方式，
2. 底层API调用

## 底层API调用

在PHP中，最基本最常用的socket使用方式是使用PHP提供的类似于C API的接口。下面是一般的调用过程。

TODO

## stream方式
顾名思义，以stream的方式使用socket，就是把socket看做类似文件资源，使用`fopen()`打开，然后如同使用文件资源一样操作socket。PHP中提供了这种方式的API：

- `stream_socket_server()`
- `stream_socket_client()`
- `stream_socket_accept()`
- `stream_socket_shutdown()`
- `stream_socket_sendto()`
- `stream_socket_recvfrom()`

下面是使用stream方式编写的一个`echo`服务器，作为演示，该服务器的功能是返回客户端发送过来的字符串，类似于PHP中的`echo`函数。

```php
<?php
// echo server

// 使用stream_socket_server创建服务器，如果创建失败，$errno和$errorMessage会被填充错误信息
$server = stream_socket_server("tcp://127.0.0.1:1337", $errno, $errorMessage);

if ($server === false) {
    throw new UnexpectedValueException("Could not bind to socket: $errorMessage");
}

for (;;) {

	// 创建一个连接
    $client = @stream_socket_accept($server);

    if ($client) {
    
    	 // 将客户端发送过来的信息发送到客户端
        stream_copy_to_stream($client, $client);
        fclose($client);
    }
}
?>
```

我们在一个终端中运行服务器。

```bash
% php server.php
```

在另外一个终端中运行测试服务器。

```bash
% echo "Hello World" | nc 127.0.0.1 1337
```

测试结果应该是客户端收到"Hello World"字符串，并且打印出来。

 

# 服务器

# 客户端