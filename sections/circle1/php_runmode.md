# 面试题目
```
1. PHP都有哪些运行模式？区别是什么？
2. PHP发行版都有TS(线程安全）和NTS(非线程安全），这两者的区别是什么？
```

# PHP运行模式

1. CGI（通用网关接口 / Common Gateway Interface）
2. FastCGI（常驻型CGI / Long-Live CGI）
3. CLI（命令行运行 / Command Line Interface）
4. Web模块模式（Apache等Web服务器运行的模式）
5. ~~ISAPI（Internet Server Application Program Interface）~~
备注：在PHP5.3以后，PHP不再有ISAPI模式

## CGI
>CGI全称是“通用网关接口”(Common Gateway Interface)， 它可以让一个客户端，从网页浏览器向执行在Web服务器上的程序请求数据。
>
>CGI描述了客户端和这个程序之间传输数据的一种标准。
>CGI的一个目的是要独立于任何语言的，所以CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等

CGI模式已经很老，由于性能低下，所以现在已经很少使用了，代替的是升级版的CGI，叫做FastCGI。

CGI性能低下主要是他的运行方式。每一个用户请求，都首先创建一个CGI子进程，然后处理请求，处理完成之后，这个子进程就会关闭。这种方式就是Fork-And-Excute模式。

创建进程是一种费资源的行为，而且每次创建进程之后，处理请求之前都需要重新解析一遍`php.ini`文件，初始化环境，这个过程是每次请求都要进行一次的，也会造成性能低下。


## FastCGI
>fast-cgi 是cgi的升级版本，FastCGI 像是一个常驻 (long-live) 型的 CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次 (这是 CGI 最为人诟病的 fork-and-execute 模式)。


### FastCGI工作原理

1. Web Server启动时载入FastCGI进程管理器【PHP的FastCGI进程管理器是PHP-FPM(php-FastCGI Process Manager)】（IIS ISAPI或Apache Module)
2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (可见多个php-cgi.exe或php-cig)并等待来自Web Server的连接；
3. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi
4. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 WebServer中）的下一个连接。
5. 在正常的CGI模式中，`php-cgi.exe`在此便退出了。在CGI模式中，你可以想象 CGI通常有多慢。每一个Web请求PHP都必须重新解析`php.ini`、重新载入全部dll扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接（Persistent database connection）可以工作。

备注：PHP的FastCGI进程管理器是PHP-FPM（PHP-FastCGI Process Manager）

### 优点

1. 从稳定性上看，FastCGI是以独立的进程池来运行CGI，单独一个进程死掉，系统可以很轻易的丢弃，然后重新分配新的进程来运行逻辑；
2. 从安全性上看，FastCGI支持分布式运算。FastCGI和宿主的Server完全独立，FastCGI怎么down也不会把Server搞垮；
3. 从性能上看，FastCGI把动态逻辑的处理从Server中分离出来，大负荷的IO处理还是留给宿主Server，这样宿主Server可以一心一意作IO，对于一个普通的动态网页来说, 逻辑处理可能只有一小部分，大量的是图片等静态。

### 不足

因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。

## CLI

>PHP-CLI是PHP Command Line Interface的简称，就是PHP在命令行运行的接口，区别于在Web服务器上运行的PHP环境（PHP-CGI，ISAPI等）。
也就是说，PHP不单可以写前台网页，它还可以用来写后台的程序。 PHP的CLI Shell脚本适用于所有的PHP优势，使创建要么支持脚本或系统甚至与GUI应用程序的服务端，在Windows和Linux下都是支持PHP-CLI模式的。

>我们在Linux下经常使用”php –m”查找PHP安装了那些扩展就是PHP命令行运行模式；

## 模块模式

模块模式是以`mod_php5`模块的形式集成，此时`mod_php5`模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（`mod_php5`）， PHP模块通过注册apache2的`ap_hook_post_config`挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载，这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号`HUP`或者`AP_SIG_GRACEFUL`通知服务器重新载入模块。但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 Apache中对动态链接库的处理是通过模块`mod_so`来完成的，因此`mod_so`模块不能被动态加载，它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

## ISAPI模式

ISAPI（Internet Server Application Program Interface）是微软提供的一套面向Internet服务的API接口，一个ISAPI的DLL，可以在被用户请求激活后长驻内存，等待用户的另一个请求，还可以在一个DLL里设置多个用户请求处理函数，此外，ISAPI的DLL应用程序和WWW服务器处于同一个进程中，效率要显著高于CGI。（由于微软的排他性，只能运行于windows环境）

PHP作为Apache模块，Apache服务器在系统启动后，预先生成多个进程副本驻留在内存中，一旦有请求出现，就立即使用这些空余的子进程进行处理，这样就不存在生成子进程造成的延迟了。这些服务器副本在处理完一次HTTP请求之后并不立即退出，而是停留在计算机中等待下次请求。对于客户浏览器的请求反应更快，性能较高。

# 流行的模式
目前在HTTPServer这块基本可以看到有三种stack比较流行：

1. Apache+mod_php5
2. lighttp+spawn-fcgi
3. nginx+PHP-FPM

三者后两者性能可能稍优，但是Apache由于有丰富的模块和功能，目前来说仍旧是老大。有人测试nginx+PHP-FPM在高并发情况下可能会达到Apache+mod_php5的5~10倍。现在nginx+PHP-FPM使用的人越来越多了。

# 线程安全和非线程安全版本的区别

php.net上提供了两种二进制安装文件，线程安全以及非线程安全。这两者的区别是什么？为什么会出现这两种版本？

不同服务器在处理请求的实现上不同，一个比较流行的方式是通过线程，也就是说，服务器会创建销毁一个线程来处理请求。Apache HTTP服务器支持多种模型，其中一种叫做worker MPM的模型就是利用线程来处理请求的。不过，它也支持另外一种并发模型，叫做prefork MPM，这种模型使用了进程，也就是说Apache会创建销毁一个进程来处理请求。

PHP本身其实是不对具体的http请求做出反应的，这些工作是交给服务器的。比如Apache是通过mod_php5这个模块来连接服务器和php的，一旦请求到来，服务器就会把请求转发给mod_php5，然后php做处理，返回结果，服务器接收到结果之后发回到客户端。

所以如果Apache使用的worker MPM模型（也就是使用线程）的话，PHP必须要处理多线程环境的问题，比如线程冲突，所以这时候必须要使用线程安全版本。

线程安全版本的性能要比非线程安全的低，Apache的`prefork MPM`以及Nginx的`FastCGI(php-fpm)`使用的都是进程，所以都可以使用非线程安全。
