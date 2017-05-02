# 面试题目
```
1. 谈一谈c10k问题。
```

上一篇我们熟悉了如何在PHP中使用socket，并且实现了简单的服务器，这一节我们将关注socket编程的性能问题，但是这性能是一个庞大的主题，我们这里先简单了解一下，更加详细的内容放在性能Circle节。

# C10k问题

socket是网络层的底层核心内容，也是TCP/IP以及UDP底层协议的实现通道。随着互联网信息时代的爆炸式发展，当代服务器的性能问题面临越来越大的挑战，著名的[C10K 问题](http://www.kegel.com/c10k.html)也随之出现。

> 谈一谈c10k问题

C10K问题具体说就是服务器如何处理10k个客户端的并发连接。到了web2.0之后，用户数增长快速，应用程序也变得更加复杂，从原来简单的表单提交，到现在的即时通讯和在线实时互动，而每一个用户都必须和服务器保持TCP连接才能进行实时的数据交互。这时候如何提高单个服务器的并发连接成为了非常重要的事情。如果不提高并发连接数，只能通过增加服务器数目来解决问题，这个成本对于Facebook这样的网站将是非常巨大的。

为了解决C10k问题，有很多解决方案被提出来。FreeBSD提出了kqueue，Linux退出了epoll，windows退出了IOCP。因为Linux是互联网企业中使用率最高的操作系统，Epoll就成为C10K killer、高并发、高性能、异步非阻塞这些技术的代名词了。为了理解这个问题，我们需要首先理解一些概念。

# 阻塞、同步、多路复用

1、阻塞/非阻塞：这两个概念是针对 IO 过程中进程的状态来说的，阻塞 IO 是指调用结果返回之前，当前线程会被挂起；相反，非阻塞指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回。

2、同步/异步：这两个概念是针对调用如果返回结果来说的，所谓同步，就是在发出一个功能调用时，在没有得到结果之前，该调用就不返回；相反，当一个异步过程调用发出后，调用者不能立刻得到结果，实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。

3、多路复用（IO/Multiplexing）：为了提高数据信息在网络通信线路中传输的效率，在一条物理通信线路上建立多条逻辑通信信道，同时传输若干路信号的技术就叫做多路复用技术。对于 Socket 来说，应该说能同时处理多个连接的模型都应该被称为多路复用，目前比较常用的有 select/poll/epoll/kqueue 这些 IO 模型（目前也有像 Apache 这种每个连接用单独的进程/线程来处理的 IO 模型，但是效率相对比较差，也很容易出问题，所以暂时不做介绍了）。在这些多路复用的模式中，异步阻塞/非阻塞模式的扩展性和性能最好。

# 一个简单的例子

之前我们编写的简单服务器只能同一时间为一个客户端服务，但是这样的方式是效率非常低。为了解决这个问题，PHP的socket模块提供了多路复用模型。

从前一节，我们知道我们有两种方法使用socket，分别是底层API和stream方式。对于两种方式，PHP分别提供了各自的多路复用函数，对应的是`socket_select`和`stream_select`。为了说明情况，我们这里就介绍`socket_select`。

下面这个例子是一个聊天室程序，所有连接到服务器的客户端能够发送消息，这个消息会同时发送到其他客户端。

```php
<?php

$port = 1337;
$host = "127.0.0.1";

// create a streaming socket, of type TCP/IP
$sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);

// set the option to reuse the port
socket_set_option($sock, SOL_SOCKET, SO_REUSEADDR, 1);

// "bind" the socket to the address to "localhost", on port $port
// so this means that all connections on this port are now our resposibility to send/recv data, disconnect, etc..
socket_bind($sock, $host, $port);

// start listen for connections
socket_listen($sock);

// create a list of all the clients that will be connected to us..
// add the listening socket to this list
$clients = array($sock);

while (true) {
    // create a copy, so $clients doesn't get modified by socket_select()
    $read = $clients;

    // get a list of all the clients that have data to be read from
    // if there are no clients with data, go to next iteration
    if (socket_select($read, $write = NULL, $except = NULL, 0) < 1)
        continue;

    // check if there is a client trying to connect
    if (in_array($sock, $read)) {
        // accept the client, and add him to the $clients array
        $clients[] = $newsock = socket_accept($sock);

        socket_getpeername($newsock, $ip);
        echo "New client connected: {$ip}\n";

        // send the client a welcome message
         socket_write($newsock, "no noobs, but ill make an exception :)\n".
         "There are ".(count($clients) - 1)." client(s) connected to the server\n");

        // remove the listening socket from the clients-with-data array
        $key = array_search($sock, $read);
        unset($read[$key]);
    }

    // loop through all the clients that have data to read from
    foreach ($read as $read_sock) {

        // read until newline or 1024 bytes
        // socket_read while show errors when the client is disconnected, so silence the error messages
        $data = @socket_read($read_sock, 1024, PHP_NORMAL_READ);

        // check if the client is disconnected
        if ($data === false) {
            // remove client for $clients array
            $key = array_search($read_sock, $clients);
            unset($clients[$key]);
            echo "client disconnected.\n";
            // continue to the next client to read from, if any
            continue;
        }

        // trim off the trailing/beginning white spaces
        $data = trim($data);

        // check if there is any data after trimming off the spaces
        if (!empty($data)) {

            // send this to all the clients in the $clients array (except the first one, which is a listening socket)
            foreach ($clients as $send_sock) {

                // if its the listening sock or the client that we got the message from, go to the next one in the list
                if ($send_sock == $sock || $send_sock == $read_sock)
                    continue;

                // write the message to the client -- add a newline character to the end of the message
                socket_write($send_sock, $data."\n");

            } // end of broadcast foreach

        }

    } // end of reading foreach
}

// close the listening socket
socket_close($sock);

?>

```
