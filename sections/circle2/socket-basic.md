# 面试题目
```
1. 在PHP中，如何获知哪个端口空闲，可以监听？
```

# 使用方式

PHP中使用socket有两种方法：

1. stream方式，
2. 底层API调用

网络上的教程一般都使用了底层API，这些API和C中的使用方法非常相似。而stream方式是使用流的方式使用socket，所以可以用非常熟悉的文件操作来操作socket资源，使用方式上更加方便。

## 底层API调用

在PHP中，最基本最常用的socket使用方式是使用PHP提供的类似于C API的接口。下面是一般的调用过程。

1. 使用`socket_create`创建一个socket资源
2. 使用`socket_set_block`设置socket为阻塞模式
3. 使用`socket_bind`来绑定端口
4. 使用`socket_listen`来监听事件
5. 进入一个循环
	1. 使用`socket_accept`来接受客户端连接，失败的时候返回为FALSE，成功返回一个连接资源
	2. 使用`socket_read`来读取连接资源，获得客户端发送过来的信息
	3. 使用`socket_write`往连接资源中写入数据，就是返回到客户端的信息
6. 使用`socket_close`来关闭socket

下面是一个简单的`echo`服务器，用来演示如何使用底层API来进行socket编程。

```php
<?php 

// server.php
set_time_limit(0);
// 设置主机和端口
$host = "127.0.0.1";
$port = 1337;
// 创建一个tcp流
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP) 
    or die("socket_create() failed:" . socket_strerror(socket_last_error()));
 
// 设置阻塞模式
socket_set_block($socket) 
    or die("socket_set_block() failed:" . socket_strerror(socket_last_error()));  
 
// 绑定到端口
socket_bind($socket, $host, $port) 
    or die("socket_bind() failed:" . socket_strerror(socket_last_error()));
 
// 开始监听
socket_listen($socket, 4) 
    or die("socket_listen() failed:" . socket_strerror(socket_last_error()));
 
echo "Binding the socket on $host:$port ... \n";
 
while (true) {
 
    // 接收连接请求并调用一个子连接Socket来处理客户端和服务器间的信息
    if (($msgsock = socket_accept($socket)) < 0) {
        echo "socket_accept() failed:" . socket_strerror(socket_last_error());
    }else{
        // 读数据
        $out = '';
        while($buf = socket_read($msgsock,8192)){
            $out .= $buf;
        }
 
        // 写数据
        $in = $out;
        socket_write($msgsock, $in, strlen($in));
    }
    // 结束通信
    socket_close($msgsock);
}
socket_close($socket);
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

### 相关资料

- [php.net上的socket api参考](http://php.net/manual/zh/book.sockets.php)


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

### 相关资料
- [PHP Socket Programming, done the Right Way](https://www.christophh.net/2012/07/24/php-socket-programming/)

# 相关问题
> 在PHP中，如何获知哪个端口空闲，可以监听？

`socket_bind`函数第三个参数端口，如果传入0，表示随机使用一个空闲的端口。`socket_bind ($socket, $bind_address, 0)`。然后我们可以通过`socket_getsockname($socket, $socket_address, $socket_port)`可以获得到底监听了哪一个空闲端口，这个端口数值填充在`$socket_port`中。