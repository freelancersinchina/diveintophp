# 面试题目
```
1. 如何安全的处理密码？
2. 面对提交上来的`html`、`javascript`代码如何处理？如何输出用户的html和js代码
```
# 密码安全

用户的账号和密码都会存储在数据库中，如何正确的存储密码显得非常重要。一般我们都会先将密码进行哈希，然后再存入数据库中，这样即使数据库被破解了，黑客也无法看到密码到底是什么。

密码进行哈希时，应该单独进行加盐处理，加盐值指的是在哈希之前先加入随机子串，以防范字典破解或者彩虹碰撞（一个利用保存的通用哈希后密码数据库来逆向推出密码）。

为了增加破解的难度，一般会加入随机盐值。如果一直使用同一个SALT，那么一旦有人破解一个密码，加入的SALT相当于没有作用，如果采用对每个密码使用不同的随机SALT，那么需要将SALT和密码一起存储，以便获取密码，这并不太方便。

针对密码哈希这个问题，PHP中有专门用来哈希密码的函数：password_hash()。

## 使用password_hash来哈希密码

password_hash 函数在 PHP 5.5 时被引入。 此函数现在使用的是目前 PHP 所支持的最强大的加密算法 BCrypt 。 当然，此函数未来会支持更多的加密算法。 password_compat 库的出现是为了提供对 PHP >= 5.3.7 版本的支持。这个密码哈希扩展会创建在计算上复杂的安全的密码哈希值，包括在幕后生成和处理随机的 SALT。加进去的随机子串通过加密算法自动保存着，成为哈希的一部分。password_verify() 会把随机子串从中提取，所以你不必使用另一个数据库来记录这些随机子串。

在下面例子中，我们哈希一个字符串，然后和新的哈希值对比。因为我们使用的两个字符串是不同的（’secret-password’ 与 ‘bad-password’），所以登录失败。

```php
<?php
$passwordHash = password_hash('secret-password', PASSWORD_DEFAULT);

if (password_verify('bad-password', $passwordHash)) {
    // Correct Password
} else {
    // Wrong password
}
?>
```

所以在密码存储时，可以尽量使用password_hash来哈希密码，使用password_verify()来检查密码正确性。

# 数据安全
永远不要信任外部输入。请在使用外部输入前进行过滤和验证。filter_var() 和 filter_input() 函数可以过滤文本并对格式进行校验（例如 email 地址）。

数据可以根据不同的目的进行不同的 过滤 。比如，当原始的外部输入被传入到了 HTML 页面的输出当中，它可以在你的站点上执行 HTML 和 JavaScript 脚本！这属于跨站脚本攻击（XSS），是一种很有杀伤力的攻击方式。一种避免 XSS 攻击的方法是在输出到页面前对所有用户生成的数据进行清理，使用 strip_tags() 函数来去除 HTML 标签或者使用 htmlentities() 或是 htmlspecialchars() 函数来对特殊字符分别进行转义从而得到各自的 HTML 实体。

另一个例子是传入能够在命令行中执行的选项。这是非常危险的（同时也是一个不好的做法），但是你可以使用自带的 escapeshellarg() 函数来过滤执行命令的参数。


## 数据过滤
数据过滤有两种，一种是数据有效性验证，另外一种是数据清洗。

### Validating（验证）
- 验证用户输入的有效性
- 设置严格的格式规则（不如URL和Email验证）
- 如果成功则返回预期类型的数据，否则返回FALSE

验证是来确保外部输入的是你所想要的内容。比如，你也许需要在处理注册申请时验证 email 地址、手机号码或者年龄等信息的有效性。

### Sanitizing（清洗）
- 用于允许或者禁止字符串中指定的字符
- 无数据格式规则
- 始终返回字符串

数据清洗是指删除（或者转义）外部输入中的非法和不安全的字符。例如，你需要在将外部输入包含在 HTML 中或者插入到原始的 SQL 请求之前对它进行过滤。当你使用 PDO 中的限制参数功能时，它会自动为你完成过滤的工作。

有些时候你可能需要允许一些安全的 HTML 标签输入进来并被包含在输出的 HTML 页面中，但这实现起来并不容易。尽管有一些像 HTML Purifier 的白名单类库为了这个原因而出现，实际上更多的人通过使用其他更加严格的格式限制方式例如使用 Markdown 或 BBCode 来避免出现问题。

## PHP中的Filter模块

PHP中提供了一个数据过滤模块[Filter](http://php.net/manual/zh/book.filter.php)，里面有很多过滤器函数。过滤器函数需要一个过滤器来声明过滤方法，PHP有三种过滤器：Validating、Sanitizing和自定义。

### 过滤函数
- filter_has_ver - 检测是否存在指定类型的变量
- filter_id - 返回某个特定名字的过滤器的相关联的id
- filter_input_array -获取一系列外部变来干，并且可以通过过滤器处理他们
- filter_input - 通过名称获取特定的外部变量，并且通过过滤器处理
- filter_list - 返回所支持的过滤器列表
- filter_var_array -获取多个变量并且过滤他们
- filter_var - 使用特定的过滤器过滤一个变量

具体的使用方法可以参考官网上的[使用说明](http://php.net/manual/zh/ref.filter.php)。

下面是几个例子，简单说明使用方法。

#### 使用filter_var验证email的合法性
```php
<?php
$email_a = 'joe@example.com';
$email_b = 'bogus';

if (filter_var($email_a, FILTER_VALIDATE_EMAIL)) {
    echo "This ($email_a) email address is considered valid.\n";
}
if (filter_var($email_b, FILTER_VALIDATE_EMAIL)) {
    echo "This ($email_b) email address is considered valid.\n";
} else {
    echo "This ($email_b) email address is considered invalid.\n";
}
?>
```
#### 使用filter_var传入options来设置自定义条件
每一个过滤函数都可以传入一个options参数，来设置自定义条件。具体的使用参考[官网说明](http://php.net/manual/zh/function.filter-var.php)
```php
<?php
// 使用options的filter函数，传入的options参考一下例子
$options = array(
    'options' => array(
        'default' => 3, // 如果失败了，返回默认3
        // 其他的选项
        'min_range' => 0
    ),
    'flags' => FILTER_FLAG_ALLOW_OCTAL,
);
$var = filter_var('0755', FILTER_VALIDATE_INT, $options);

// 如果过滤器只支持flags选项，那么可以直接传入flag
$var = filter_var('oops', FILTER_VALIDATE_BOOLEAN, FILTER_NULL_ON_FAILURE);

// 如果过滤器只支持flags选项，也可以传入一个option
$var = filter_var('oops', FILTER_VALIDATE_BOOLEAN,
                  array('flags' => FILTER_NULL_ON_FAILURE));

?>
```

#### 在filter中使用自定义验证函数
filter函数都支持使用自己的验证函数，下面是使用自定义验证函数的例子。

```php
// 设置自定义的验证函数，如果非法返回false,合法就返回一个值（可以传入值，也可以是经过清洗过后的值）
function foo($value)
{
    // 期待的格式: Surname, GivenNames
    if (strpos($value, ", ") === false) return false;
    list($surname, $givennames) = explode(", ", $value, 2);
    $empty = (empty($surname) || empty($givennames));
    $notstrings = (!is_string($surname) || !is_string($givennames));
    if ($empty || $notstrings) {
        return false;
    } else {
        return $value;
    }
}
$var = filter_var('Doe, Jane Sue', FILTER_CALLBACK, array('options' => 'foo'));
?>
```

以上的例子都使用了Validate，Sanitizing用法类似，不过返回的值是经过清洗过的数据或者是false。

```php
<?php
$a = 'joe@example.org';
$b = 'bogus - at - example dot org';
$c = '(bogus@example.org)';

$sanitized_a = filter_var($a, FILTER_SANITIZE_EMAIL);
if (filter_var($sanitized_a, FILTER_VALIDATE_EMAIL)) {
    echo "This (a) sanitized email address is considered valid.\n";
}

$sanitized_b = filter_var($b, FILTER_SANITIZE_EMAIL);
if (filter_var($sanitized_b, FILTER_VALIDATE_EMAIL)) {
    echo "This sanitized email address is considered valid.";
} else {
    echo "This (b) sanitized email address is considered invalid.\n";
}

$sanitized_c = filter_var($c, FILTER_SANITIZE_EMAIL);
if (filter_var($sanitized_c, FILTER_VALIDATE_EMAIL)) {
    echo "This (c) sanitized email address is considered valid.\n";
    echo "Before: $c\n";
    echo "After:  $sanitized_c\n";    
}
?>
```
