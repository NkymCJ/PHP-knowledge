# PHP-Function

PHP 函数

### 1. 用户自定义函数

1. 函数名以字母或下划线开头，后面跟字母、数字或下划线

2. 函数无需在调用之前被定义，除非以下两种情况
    
    - 当一个函数是有条件被定义时，必须在调用函数之前定义

        示例：
        ```
        $makefoo = true;
        // foo(); // 报错 致命错误，函数未定义
        if ($makefoo) { // 函数是否定义是有条件的
            function foo()
            {
                echo "Hello world";
            }
        }
        foo(); // 现在可以安全调用
        ```

    - 函数中的函数

        示例：
        ```
        function foo()
        {
            function bar()
            {
                echo "Hello World";
            }
        }
        // bar(); // 报错 致命错误，因为此时它未定义
        foo(); // 执行foo()函数，定义bar()函数
        bar(); // 现在可以执行bar()函数
        ```

3. PHP中的所有函数和类都具有全局作用域，可以定义在一个函数之内而在之外调用，反之亦然

4. PHP不支持函数重载，也不可能取消定义或者重定义已经声明的函数

    示例：
    ```
    function abc(){
        echo 'Hello World';
    }
    function abc(){
        echo '123456';
    }
    // 报错 致命错误，不能重复声明
    abc();
    ```

5. PHP函数名是大小写无关的，不过在调用函数的时候，应使用其在定义时相同的形式

    示例：
    ```
    function aBc(){
        echo 'Hello World';
    }
    AbC(); // 打印 Hello World
    ```

6. PHP中可以调用递归函数

    示例：
    ```
    function recursion($a)
    {
        if ($a <= 10) {
            echo "$a ";
            recursion($a + 1);
        }
    }
    recursion(5); // 打印 5678910
    ```
### 2. 函数参数

1. 普通形式，值传递

    *默认情况下，函数参数使用值传递（即使在函数内部改变参数的值，也不会改变函数外部的值）*

    示例：
    ```
    function print_arg($arg)
    {
        print $arg;
    }
    ```

2. 引用传递参数

    *在参数前面加上符号（&），允许函数修改它的参数值*

    示例：
    ```
    // 这种情况下，在函数中做的修改并不会影响外面的值
    $str = 'Hello World';
    function print_arg($arg)
    {
        $arg = $arg . '123456';
        print $arg;
    }
    print_arg($str); // 打印 Hello World123456
    print $str; // 打印 Hello World

    // 在函数参数前加上符号（&）
    $str = 'Hello World';
    function print_arg(&$arg)
    {
        $arg = $arg . '123456';
        print $arg;
    }
    print_arg($str); // 打印 Hello World123456
    print $str; // 打印 Hello World123456
    ```

3. 默认参数的值

    默认值必须是常量表达式

    *函数有参数时，若使用时没有传递参数又没有设置默认参数时，报致命错误*

    *使用默认参数时，任何默认参数必须放在任何非默认参数的右侧，否则报错 WARNING*

    *PHP 5+，传引用的参数也可以有默认值*

    示例：
    ```
    // 使用标量类型作为默认参数
    function makecoffee($type = "cappuccino")
    {
        return "Making a cup of $type.\n";
    }
    echo makecoffee(); // 不传，使用默认值 "cappuccino" 代替 $type
    echo makecoffee(null); // 传NULL， 使用 "" 代替 $type
    echo makecoffee("espresso"); // 使用 "espresso" 代替 $type

    // 使用非标量类型作为默认参数
    function makecoffee($types = array("cappuccino"), $coffeeMaker = NULL)
    {
        $device = is_null($coffeeMaker) ? "hands" : $coffeeMaker;
        return "Making a cup of ".join(", ", $types)." with $device.\n";
    }
    echo makecoffee();
    echo makecoffee(array("cappuccino", "lavazza"), "teapot");

    // 默认参数在非默认参数的右侧
    function print_arg($arg0,$arg1 = '123456')
    {
        print $arg0.$arg1;
    }
    print_arg('Hello World'); // 打印 Hello World123456
    ```

4. 参数类型声明

    常见的类型声明：class、interface、self、array、callable、bool（PHP 7+）、float（PHP 7+）、int（PHP 7+）、string（PHP 7+）

    PHP 7+，若调用时，指定的参数类型不对，将抛出 TypeError 异常（PHP 5 可恢复的致命错误）

    *默认情况下，PHP会先强迫错误类型的值转为函数期望的标量类型；严格模式下，只有与类型声明完全相符才会被接受*

    示例：
    ```
    // 正常情况
    function arg(int $e){
        var_dump($e); // 打印 int(123)
    }
    arg(123);

    // 发生了转换
    function arg(int $e){
        var_dump($e); // 打印 int(123)
    }
    arg('123');

    // 发生了转换
    function arg(int $e){
        var_dump($e); // 报错 TypeError 异常
    }
    arg([123]);
    ```

5. 可变数量的参数列表

    PHP 5.6+，使用 ... 语法
    
    - ...$e 访问参数

        示例：
        ```
        function sum(...$e)
        {
            $res = 0;
            foreach ($e as $x) {
                $res += $x;
            }
            return $res;
        }
        echo sum(3,2,1); // 打印 6
        ```

    - ...[x,y,z] 提供参数

        示例：
        ```
        function add($a, $b)
        {
            return $a + $b;
        }
        print add(...[1, 2]); // 打印 3

        $a = [1, 2];
        print add(...$a); // 打印 3
        ```

    - *可以在 ... 之前指定正常位置参数。在这种情况下，只有与位置参数不匹配的尾随参数会被添加到由 ... 生成的数组中*

        示例：
        ```
        function sum($e0,...$e)
        {
            $res = 0;
            foreach ($e as $x) {
                $res += $x;
            }
            print $e0;
            return $res;
        }
        echo sum(3,2,1); // 打印 33
        ```

    - *可以在 ... 之前添加类型声明，如果有进行声明，那么 ... 数组中的元素都必须符合此类型*

        示例：
        ```
        function sum(int ...$e)
        {
            $res = 0;
            foreach ($e as $x) {
                $res += $x;
            }
            return $res;
        }
        echo sum([0, 1],0,1); // 报错 TypeError异常
        echo sum(0,1); // 打印 3
        ```

---
STARTED 2018/11/19 ChenJian

MODIFIED 2018/11/19 ChenJian