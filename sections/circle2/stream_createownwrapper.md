# 面试题目
```
1. 创建自己的stream包装器有什么好处？
```

上一篇介绍了基本的PHP Stream知识，这一篇我们将会了解如何创建一个自己定制的包装器，在真实情境下使用。比如创建一个`asw://`包装器，用来获取`Aamazon S3`上的资源。


> 为什么要实现自己的stream包装器？

包装器可以将对资源的操作封装成类似对文件资源的操作，这样用户就可以使用文件操作函数操作资源，而不需要了解具体的协议和实现。

# streamWrapper
PHP建议了一些接口，如果自己想要实现一个包装器的话，需要实现其中的一些，如果不是按照这些接口规则实现，则有可能发生不可预知的错误。

可以访问[这里](http://php.net/manual/en/class.streamwrapper.php)，查看哪些接口是你的包装器需要实现的。注意，这里的`streamWrapper`并不是真实的一个类，你的包装器不需要继承，而且这里枚举的方法并不需要全部实现，只需要实现自己需要的。比如类似操作文件的流是不需要实现`dir_closedir()`。

# 例子：var://包装器

这个包装器是用来读取和写入全局变量的，使用方法如下，可以看到使用方法就像是操作文件一样。
```php
$myvar = "";
$fp = fopen("var://myvar", "r+");
fwrite($fp, "line1\n");
fwrite($fp, "line2\n");
fwrite($fp, "line3\n");
rewind($fp);
while (!feof($fp)) {
    echo fgets($fp);
}
fclose($fp);
var_dump($myvar);
```

因为像文件一样操作，所以我们需要实现以下几个方法

- stream_open
- stream_close
- stream_eof
- stream_metadata
- stream_read
- stream_seek
- stream_tell
- stream_write

```php
<?php
class VariableStream{

  private $position;
  private $varname;
  public function stream_open($path,$mode,$options,&$opened_path){
    $url = parse_url($path);  // 解析url成一个索引数组，var://myvar  
    $this->varname = $url['host'];
    $this->position = 0;
 }

  public function stream_read($count){
    $ret = substr($GLOBALS[$this->varname],$this->position,$count);
    $this->position += strlen($ret);
    return $ret;
  }

  public function stream_write($data){
    $left = substr($GLOBALS[$this->varname],0,$this->poistion);
    $right = substr($GLOBALS[$this->varname],strlen($data)+$this->poisition);
    $GLOBALS[$this->varname] = $left.$data.$right;
    $this->position += strlen($data);
    return strlen($data);
  }

  public function stream_tell(){
    return $this->position;
  }

  public function stream_eof(){
    return $this->position >= strlen($GLOBALS[$this->varname]);
  }

  public function stream_seek($offset,$whence){
    switch($whence){
      case SEEK_SET:
         if($offset<strlen($GLOBALS[$this->varname]) && $offset>0){
            $this->position = $offset;
            return true;
         }
         else{
            return false;
         }
      case SEEK_CUR:
         if($offset>=0){
            $this->position += $offset;
            return true;
         }
         else{
            return false;
         }
      case SEEK_END:
         if(strlen($GLOBALS[$this->varname]) +$offset>=0){
           $this->position = strlen($GLOBALS[$this->varname]) +$offset;
           return true;
         }
         else{
           return false;
         }
      default:
           return false;

    }
  }

  public function stream_metadata($path,$option,$var){
     if($option == STREAM_META_TOUCH){
       $url = parse_url($path);
       $this->varname = $url['host'];
      if(!isset($GLOBALS[$this->varname])){
         $GLOBALS[$this->varname] = "";
      }
      return true;
     }
    return false;
  }
}
```

为了使得这个包装器能够使用，需要使用`stream_wrapper_register`来注册包装器。

```php
<?php
stream_wrapper_register("var","VariableStream") or die("Failed to register protocol");
```

# 参考资料
- [php.net上的Stream](http://php.net/manual/en/book.stream.php)
