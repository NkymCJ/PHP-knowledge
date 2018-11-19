# PHP-Boolean

PHP 布尔值

### 1. 初始化
    
1. 要指定一个布尔值，使用常量TRUE或FALSE。不区分大小写

    示例：
    ```
    $foo = true; // 设置 $foo 为 TRUE
    $foo = TRUE; // 设置 $foo 为 TRUE
    $foo = True; // 设置 $foo 为 TRUE
    ...
    ```

2. 通常运算符所返回的boolean值结果会被传递给控制流程

    示例：
    ```
    $str0 = '123456';
    if ($str0 == '123456') {
        print 'Hello World';
    }
    $is_true = true;
    if ($is_true) {
        print 'Hello World';
    }
    ```

### 2. 类型转换

1. 用 (bool) 或者 (boolean) 来强制转换，不过一般不需要强制转换，因为当运算符，函数或者流程控制结构需要一个 boolean 参数时，该值会被自动转换

2. 当以下值转换时，被认为是FALSE
    - 布尔值FALSE本身
    - 整型值0
    - 浮点型值0.0
    - 空字符串，以及字符串"0"
    - 不包括任何元素的数组
    - 特殊类型NULL（包括尚未赋值的变量）
    - 从空标记生成的SimpleXML对象
3. 除2.2外，所有其它值都被认为是TRUE（包括任何资源和NAN）

    *-1和其它非零值（不论正负）一样，被认为是TRUE*

        示例：
        ```
        var_dump((bool) FALSE); // 打印 bool(false)
        var_dump((bool) 0); // 打印 bool(false)
        var_dump((bool) 0.0); // 打印 bool(false)
        var_dump((bool) ""); // 打印 bool(false)
        var_dump((bool) "0"); // 打印 bool(false)
        var_dump((bool) array()); // 打印 bool(false)
        var_dump((bool) null); // 打印 bool(false)
        var_dump((bool) $str); // 报错 NOTICE 打印 bool(false)
        var_dump((bool) 1); // 打印 bool(true)
        var_dump((bool) -2); // 打印 bool(true)
        var_dump((bool) "foo"); // 打印 bool(true)
        var_dump((bool) 2.3e5); // 打印 bool(true)
        var_dump((bool) array(12)); // 打印 bool(true)
        ```