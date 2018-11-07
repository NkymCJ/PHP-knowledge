# PHP-error

PHP错误处理机制

### 1. 常见的错误报告级别

| 值 | 常量 | 描述 |
| - | - | - |
| 1 | E_ERROR | 致命的运行时错误。这类错误一般是不可恢复的情况，导致脚本终止不再继续运行 |
| 2 | E_WARNING | 运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行 |
| 4 | E_PARSE  | 编译时语法解析错误。解析错误仅仅由分析器产生 |
| 8 | E_NOTICE | 运行时通知。表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知 |
| 256 | E_USER_ERROR | 用户产生的错误信息。类似 E_ERROR, 但是是由用户自己在代码中使用PHP函数 trigger_error()来产生的 |
| 512 | E_USER_WARNING | 用户产生的警告信息。类似 E_WARNING, 但是是由用户自己在代码中使用PHP函数 trigger_error()来产生的 |
| 1024 | E_USER_NOTICE | 用户产生的通知信息。类似 E_NOTICE, 但是是由用户自己在代码中使用PHP函数 trigger_error()来产生的 |
| 2048 | E_STRICT | 启用 PHP 对代码的修改建议，以确保代码具有最佳的互操作性和向前兼容性 |
| 8191 | E_ALL（在 PHP 6.0，E_STRICT 是 E_ALL 的一部分） | E_STRICT出外的所有错误和警告信息（在 PHP 6.0之后，E_STRICT 也是 E_ALL 的一部分） |

### 2. 设置错误报告的级别

 > int error_reporting ([ int $level ])

1. $level 可选。错误报告的级别。未设置$level时，默认当前的错误报告级别
2. return 返回值。返回旧的错误报告级别。当未设置$level时，返回当前的错误报告级别

示例:
```
// 关闭所有PHP错误报告
error_reporting(0);
// 报告所有 PHP 错误
error_reporting(-1);
// 报告所有 PHP 错误
error_reporting(E_ALL);
// 报告除E_NOTICE外的其他所有错误
error_reporting(E_ALL ^ E_NOTICE);
// 报告属于其中的错误
error_reporting(E_ERROR | E_WARNING | E_PARSE | E_NOTICE);
```

### 3. 产生用户级别的错误

 > bool trigger_error ( string $error_msg [, int $error_type = E_USER_NOTICE ] )

1. $error_msg 必需。指定错误的信息。长度限制在1024个字节，超出截断
2. $error_type 可选。指定产生的错误的级别。仅对E_USER系列错误级别有效，默认是E_USER_NOTICE
3. return 返回值。如果指定了错误的 error_type 会返回 FALSE ，正确则返回 TRUE

### 4. 基本的错误处理（少用）

 > die(status) / exit(status) 

1. status 必需。规定在退出脚本之前写入的消息或状态号。如果 status 是字符串，则该函数会在退出前输出字符串。如果 status 是整数，这个值会被用作退出状态。退出状态的值在 0 至 254 之间。退出状态 255 由 PHP 保留，不会被使用。状态 0 用于成功地终止程序

### 5. 创建自定义错误处理器

 > bool error_function ( int $error_level, string $error_message [, string $error_file [, int $error_line [, array $error_context ] ] ] ) {}

1. $error_level	 必需。包含错误的级别。
2. $error_message	必需。包含错误的消息。
3. $error_file	可选。包含发生错误的文件名。
4. $error_line	可选。包含错误发生的行号。
5. $error_context	可选。包含错误触发处作用域内所有变量的数组。*PHP 7.2.0后弃用*
6. return 返回值。若返回FALSE，还会执行PHP标准错误处理程序

### 6. 内建的错误处理程序

 > mixed set_error_handler ( callback $error_handler [, int $error_types = E_ALL | E_STRICT ] );

1. $error_handler 必需。自定义的错误处理器，见2
2. $error_types 可选。用于屏蔽$error_handler的触发，如果没有设置，则默认是E_ALL | E_STRICT，即$error_handler会在每个错误发生时调用，且error_reporting()失效。
3. return 返回值。如果之前有定义过错误处理程序，则返回该程序名称的 string；如果是内置的错误处理程序，则返回 NULL。 如果你指定了一个无效的回调函数，同样会返回 NULL。 如果之前的错误处理程序是一个类的方法，此函数会返回一个带类和方法名的索引数组(indexed array)。

*在需要时使用 die()结束脚本。因为如果错误处理程序返回了，脚本将会继续执行发生错误的后一行*

*以下级别的错误不能由用户定义的函数来处理： E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，和在调用 set_error_handler()函数所在文件中产生的大多数 E_STRICT*

*如果错误发生在脚本执行之前（比如文件上传时），将不会调用自定义的错误处理程序因为错误尚未在那时注册*