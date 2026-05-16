---
title: 我的第一篇博客文章
published: 2026-05-14
description: 这是我博客的第一篇文章。
image: ./cover.jpg
tags: [php，安全]
category: 网络安全
draft: false
---
# PHP

弱类型、函数命名混乱、文件包含、反序列化、危险函数

PHP 会进行**弱类型比较**。它会尝试将两边的值转换为相同的类型，然后再比较。

```php
// 松散比较 ==  vs  严格比较 ===
var_dump("123" == 123);   // true
var_dump("123" === 123);  // false
// 魔法哈希比较（CTF高频）
var_dump("0e12345" == "0e67890");   // true，因为都当作科学计数法0的多少次方
var_dump(md5('QNKCDZO') == md5('240610708')); // true！都是0e开头
// 类型转换陷阱
var_dump(in_array("abc", [0,1,2]));  // true！因为"abc"被转成0
```

- `==` 下的类型转换规则
- `0e...` 字符串相互相等
- `in_array` 默认是松散比较
- `strcmp` 函数传入数组返回NULL

在 PHP 中，数字 `0`、布尔值 `false`、空字符串 `""` 在进行松散比较 (`==`) 时都被视为相等。

```php
// 危险代码
if (strcmp($_POST['password'], $correct_password) == 0) {
    echo "登录成功！";
}
```

攻击者可以通过 POST 提交 `password[]=1`（将密码设为数组）。此时 `strcmp` 返回 `NULL`，而 `NULL == 0` 成立，导致**绕过验证直接登录**。

### **在 if 判断中的误用**

```php
if (strcmp($str1, $str2)) { 
    // 如果相等，返回 0 (false)，不执行 -> 符合预期
    // 如果不等，返回 -1 或 1 (都视为 true)，执行 -> 符合预期
}
if (strcmp($str1, $str2)) { // 错误！相等时返回 0，会被当作 false
    echo "它们相等"; // 永远不会输出
}

#可以改成
if (strcmp($str1, $str2) === 0) {
    echo "它们相等";
}
```

### 超全局变量函数

```php
$_GET
$_POST
$_COOKIE
$_SESSION
$_SERVER   #提供服务器和执行环境信息,包含请求头、IP地址、脚本路径等大量环境信息。
$_FILES    #处理通过 HTTP POST 上传的文件,包含文件名、类型、临时路径、大小及错误代码等信息。
$_REQUEST  #统一获取 GET、POST 和 COOKIE 数据.
```

永远不要相信这些变量里的任何数据。

### 文件包含

```php
include "pages/".$_GET['page'];  // 危险！
require $_GET['file'];
```

###  危险函数清单

|                             函数                             | 危害                             | 典型用法                                                     |
| :----------------------------------------------------------: | :------------------------------- | :----------------------------------------------------------- |
|                      `eval()` .assert()                      | 任意代码执行                     | `eval($_POST['cmd'])` eval（$_POST['cmd']）                  |
| `system()`、`exec()`、`shell_exec()` `,shell_exec（）`,passthru()，反引号（`） | 命令执行                         | `system("ping ".$_GET['ip'])` System（“ping ”.$_GET['ip']）  |
|                     `include`、`require`                     | 文件包含/LFI                     | `include($_GET['file'])` include（$_GET['file']）            |
|               `unserialize()` unserialize（）                | 反序列化漏洞                     | `unserialize($_COOKIE['data'])` unserialize（$_COOKIE['data']） |
|                       `preg_replace()`                       | `/e` 修饰符导致代码执行（PHP<7） | `preg_replace("/.*/e", $_GET['cmd'], "")` preg_replace（“/.*/e”， $_GET['cmd']， “”） |
|                     `create_function()`                      | 代码注入                         | `create_function('$a', $_GET['code'])` create_function（'$a'， $_GET[''code']） |
|                    `file_get_contents()`                     | SSRF/路径遍历                    | `file_get_contents($_GET['url'])` file_get_contents（$_GET['url']） |



###PHP反序列化

`__construct()`、`__destruct()`、`__wakeup()`、`__toString()` 等魔法方法

