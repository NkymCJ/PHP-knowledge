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

 > int error_reporting ( [ int $level ] )

1. $level 可选。错误报告的级别。未设置$level时，默认当前的错误报告级别
2. return 返回值。返回旧的错误报告级别。当未设置$level时，返回当前的错误报告级别

示例:
```
// 关闭所有PHP错误报告
error_reporting(0);
// echo $a;
// 报告所有 PHP 错误
error_reporting(-1);
// echo $a;
// 报告所有 PHP 错误
error_reporting(E_ALL);
// echo $a;
// 报告除E_NOTICE外的其他所有错误
error_reporting(E_ALL ^ E_NOTICE);
// echo $a;
// 报告除E_NOTICE外的其他所有错误
error_reporting(E_ALL & ~ E_NOTICE);
// echo $a;
// 报告除E_NOTICE和E_USER_WARNING外的其他所有错误
error_reporting(E_ALL & ~ (E_NOTICE | E_USER_WARNING);
// echo $a;
// trigger_error('E_USER_WARNING',E_USER_WARNING);
// 报告属于其中的错误
error_reporting(E_ERROR | E_WARNING | E_PARSE | E_NOTICE);
// echo $a;
```

### 3. 产生用户级别的错误

 > bool trigger_error ( string $error_msg [, int $error_type = E_USER_NOTICE ] )

别名：user_error();

1. $error_msg 必需。指定错误的信息。长度限制在1024个字节，超出截断
2. $error_type 可选。指定产生的错误的级别。仅对E_USER系列（如E_USER_NOTICE、E_USER_WARNING、E_USER_ERROR等）错误级别有效，默认是E_USER_NOTICE
3. return 返回值。如果指定了错误的 error_type 会返回 FALSE ，正确则返回 TRUE

示例：
```
// 触发用户级别的警告
trigger_error('E_USER_WARNING',E_USER_WARNING);
// 触发用户级别的注意
trigger_error('E_USER_NOTICE',E_USER_NOTICE);
```

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
6. return 返回值。*若返回FALSE，还会执行PHP标准错误处理程序*

### 6. 内建的错误处理程序

 > mixed set_error_handler ( callback $error_handler [, int $error_types = E_ALL | E_STRICT ] );

1. $error_handler 必需。自定义的错误处理器，见2。此外，它还可以接受类的方法，但必须以数组的形式传递，数组的第一项为“类名”，第二个项为“方法名”

    示例：
    ```
    // 使用自定义的错误处理器名
    function customError()
    {
        echo 'customError';
    }
    set_error_handler('customError');
    echo $test;
    // 使用类的方法
    class App{
        public static function customError() {
            echo 'customError';
        }
    }
    set_error_handler(array("App","customError"));
    ```

2. $error_types 可选。用于屏蔽$error_handler的触发，如果没有设置，则默认是E_ALL | E_STRICT，即$error_handler会在每个错误发生时调用。$error_types中指定的级别会绕过 PHP 标准错误处理程序，其中，会导致error_reporting()失效(除非$error_handler返回FALSE)。

    示例：
    ```
    error_reporting(E_ALL); // 设置显示所有级别的错误
    function customError()
    {
        echo 'customError';
        // return false; 
    }
    set_error_handler('customError',E_NOTICE);
    echo $test; // 触发E_NOTICE级别错误

    // 分析：产生的E_NOTICE级别错误经由set_error_handler()处理，绕过了PHP标准错误处理程序，所以只会打印“customError”。若在$error_handler中返回FALSE，则还可以继续执行PHP标准错误处理程序
    ```

3. return 返回值。如果之前有定义过错误处理程序，则返回该程序名称的 string；如果是内置的错误处理程序，则返回 NULL。 如果你指定了一个无效的回调函数，同样会返回 NULL。 如果之前的错误处理程序是一个类的方法，此函数会返回一个带类和方法名的索引数组(indexed array)。


*在需要时使用 die()结束脚本。因为如果错误处理程序返回了，脚本将会继续执行发生错误的后程序*

*存在缺点，以下级别的错误不能由用户定义的函数来处理： E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，和在调用 set_error_handler()函数所在文件中产生的大多数 E_STRICT。说简单点，就是只能捕获WARNING、NOTICE级别的错误*

*如果错误发生在脚本执行之前（比如文件上传、编译时），将不会调用自定义的错误处理程序因为错误尚未在那时注册，下面介绍的register_shutdown_function()也将如此。当遇到这种情况时如何处理，参照现如今PHP框架的做法，函数处理集中放置在一个主文件中，其他需要编译的文件通过INCLUDE的方式引入*

### 7. 获取程序最后的错误

 > array error_get_last()

1. return 返回值。返回了一个关联数组，描述了最后错误的信息，以该错误的 "type"、 "message"、"file" 和 "line" 为数组的键。 如果错误由 PHP 内置函数导致的，"message"会以该函数名开头。 如果没有错误则返回 NULL

示例：
```
var_dump(error_get_last()); // 若程序没有错误，返回NULL 

echo $a;
echo $b; 
var_dump(error_get_last()); // error_get_last()返回这行的信息
```

### 8. PHP结束时执行的函数

 > void register_shutdown_function ( callable $callback [, mixed $parameter [, mixed $... ]] )

1. $callback 必需。中止回调的函数，它会在脚本执行完成或者 exit() 后被调用

2. $parameter 可选。设置传入中止回调函数的参数

*此函数无返回值*

*此函数可以多次调用，按顺序执行。但如果其中有一个中止函数存在exit()，那就真的是程序所有处理完全结束了，不会再调其他的中止回调函数*

示例
```
function shutdown0()
{
    echo "success0</br>";
    exit(); // 若加入此函数，不会再显示success1
}
function shutdown1()
{
    echo "success1</br>";
}
register_shutdown_function('shutdown0');
register_shutdown_function('shutdown1');
```

### 9. 整合

> 捕获所有的错误并进行处理：set_error_handler()与register_shutdown_function()结合使用

示例：
```
error_reporting(0); // 设置错误报告级别为所有
function customError1()
{
    if ($error = error_get_last()) {
        echo 'customError1';
    }
}
function customError0()
{
    echo 'customError0';
    // return false;
}
set_error_handler('customError0');
register_shutdown_function('customError1');

require 'error1.php';

// 分析：首先，如果是E_NOTICE或E_WARNING级别的错误由set_error_handler()处理就好了，调用customError0()，随后因为customError0()函数没有返回FALSE，所以不再执行PHP标准错误处理程序，程序结束。接着，如果E_NOTICE或E_WARNING级别之上的错误，则因为会导致程序中止，执行register_shutdown_function()函数，调用customError1()，随后程序真正结束。通过这样子的设计，就可以捕获并处理所有的错误级别了。
```

### 10. 总结

在实际开发过程中，需要通过参数的错误级别，不同错误级别进行不同处理，例如部分错误级别需输出成日志文件。