# PHP-String

PHP 字符串

PHP字符串最大可以达到2GB

### 1. 初始化

1. 单引号

    对于单引号字符串而言，字符串中需要转义的特殊字符只有 反斜线(\\) 和 单引号(\') 本身，其余不转义。不替换内插的变量
    
    示例：
    ```
    print 'Hello World'; 
    // 打印 Hello World
    print '\''; 
    // 打印 '
    print '\\'; 
    // 打印 \
    print '\n'; 
    // 打印 \n，不转义除反斜线和单引号之外的转义序列
    $str = 0;
    print '$str'; 
    // 打印 $str，因为不替换内插变量
    ```

2. 双引号

    双引号字符串无法识别转义的单引号，不过可以识别内插的变量和以下表中的转义序列
    
    | 转义序列 | 字符 |
    | - | - |
    | \n | 换行 |
    | \r | 回车 |
    | \t | 水平制表符 |
    | \v | 垂直制表符（PHP5.2.5+） |
    | \e | Escape（PHP5.4+） |
    | \f | 换页（PHP5.2.5+） |
    | \\\\ | 反斜线 |
    | \\\$ | 美元符 |
    | \\\" | 双引号 |
    | \0到\777 | 八进制值 |
    | \x0到\xFF | 十六进制值 |
    
    示例：
    ```
    print "Hello World"; 
    // 打印 Hello World
    print "\'"; 
    // 打印 \'，无法识别转义的单引号
    print "\" \n \r \\ \$  \t \060 \061 \x30"; 
    // 打印 " \ $ 0 1 0
    $str = 0;
    print "$str"; 
    // 打印 0，因为内插变量$str

    // 含有变量的更复杂的示例
    class foo
    {
        public $foo;
        public $bar;
        public function __construct()
        {
            $this->foo = 'Foo';
            $this->bar = array('Bar1', 'Bar2', 'Bar3');
        }
    }
    $foo = new foo();
    print "$foo->foo"; // 打印 Foo
    print "{$foo->foo}"; // 打印 Foo
    print "$foo->bar[0]"; // 报错 NOTICE
    print "{$foo->bar[0]}"; // 打印 Bar1
    ```

3. heredoc格式

    - heredoc格式就像是双引号字符串，能识别所有变量内插和转义序列，不过不需要对双引号和单引号进行转义

    - heredoc格式以 <<< 和 自定义一个标识符（标识符后换行）开始，中间为内容，最后以前面自定义的标识符与分号(;)（特定情况不使用分号，例如包含字符串连接操作等）作为结束标志

    - 自定义的标识符只能包含字母、数字和下划线，并且必须以字母或下划线作为开头

    - 结束标识符除可能使用分号(;)外，不能包含其他字符，即结束标识符不能缩进，前后也不能有任何的空白或制表符

    - PHP 5.3+，可以在heredoc格式中用双引号来声明标识符

    - PHP 5.3+，可以用heredoc格式来初始化静态变量和类的属性和常量
    
    - 可以在函数参数中使用heredoc格式来传递数据

    示例：
    ```
    $str0 = '123456';
    // print <<<Token
    // 使用双引号声明标识符
    print <<<"Token" 
    Hello World
    // 打印 Hello World
    $str0
    // 打印 123456
    ' " \n \r \\ \$ \t \060 \061 \x30
    // 打印 ' " \ $ 0 1 0
    Token;

    // 特定情况结束标识符后面不应使用分号;
    $str0 = <<<"Token"
    Hello World
    Token
    . ' 123456'; 
    // 因为要进行字符串连接，所以结束标识符后面不能使用分号;，但是结束标识符还是必须独占一行
    print $str0;
    // 打印 Hello World 123456

    // 使用heredoc格式在函数参数中传递参数
    var_dump(array(<<<"EOD"
    Hello World
    EOD
    ));

    // 使用heredoc格式初始化静态变量和类的常量、属性
    function foo()
    {
        static $bar = <<<LABEL
    0
    LABEL;
    $bar++;
    print $bar;
    }
    class foo
    {
        const BAR = <<<"FOOBAR"
    Hello World
    FOOBAR;
        public $bar = <<<"FOOBAR"
    123456
    FOOBAR;
    }
    foo(); // 打印 1
    foo(); // 打印 2
    $fo = new foo();
    print $fo::BAR; // 打印 Hello World
    print $fo->bar; // 打印 123456

    // 含有变量的更复杂的示例
    class foo
    {
        public $foo;
        public $bar;
        public function __construct()
        {
            $this->foo = 'Foo';
            $this->bar = array('Bar1', 'Bar2', 'Bar3');
        }
    }
    $foo = new foo();
    print <<<"EOT"
    // 打印 Foo
    $foo->foo
    // 打印 Foo
    {$foo->foo}
    // 报错 NOTICE
    $foo->bar[0]
    // 打印 Bar1
    {$foo->bar[0]}
    EOT;
    ```

4. nowdoc格式（PHP 5.3+）
    
    - nowdoc格式与heredoc格式相似，但不进行解析操作
    
    - 声明标识符使用单引号''包裹

    示例：
    ```
    $str = <<<'EOD'
    Example of string
    spanning multiple lines
    using nowdoc syntax.
    EOD;
    print $str;
    ```

### 2. 变量解析

当字符串用 双引号("") 或 heredoc 结构定义时，其中的变量将会被解析

1. 简单语法

    - 当PHP解析器遇到一个 美元符号（$）时，它会去组合尽量多的标识以形成一个合法的变量名，但只适合简单的结构
    - 一个数组索引或一个对象属性也可被解析。数组索引要用方括号（]）来表示索引结束的边际，对象属性则是和上述的变量规则相同

    示例：
    ```
    $str = 'Hello World';
    print "$str"; // 打印 Hello World
    $arr = array(0,1,2,3,4,5,6);
    print $arr[0]; // 打印 0
    print $arr[6]; // 打印 6

    class obj {
        public $prop0 = 'Hello World';
        public $prop1 = array(0,1,2,3,4,5,6);
    }
    $obj = new obj();
    print "$obj->prop0"; // 打印 Hello World
    print "$obj->prop1[0]"; // 报错 NOTICE ，写法复杂需使用复杂语法
    ```

2. 复杂（花括号{}）语法

    - 使用 花括号{} 可以解析更复杂的表达式，且 $ 需紧挨着 { 。

    - 使用花括号相当于告诉PHP解析器这是一个变量，需要解析。其实最好就是使用这个语法，可以明确变量名的界线

    - PHP 5+，函数、方法、 静态类变量和类常量可以在 {$} 中使用

    - 只有在该字符串被定义的命名空间中才可以将其值作为变量名来访问

    - 只单一使用花括号 ({}) 无法处理从函数或方法的返回值或者类常量以及类静态变量的值
    
    示例：
    ```
    class obj
    {
        public $prop0 = 'Hello World';
        public $prop1 = array(0, 1, 2, 3, 4, 5, 6);
    }
    $obj = new obj();
    print "{$obj->prop1[0]}"; // 打印 0
    $arr = array(array(0, 1, 2, 3, 4, 5, 6), 1, 2, 3, 4, 5, 6);
    print "{$arr[0][6]}"; // 打印 6

    // 静态类变量和类常量在 {$} 中使用
    class obj {
        const str0 = 'str0';
        public static $str1 = 'str1';
    }
    $str0 = 'Hello World';
    $str1 = '123456';
    echo "{${obj::str0}}";
    echo "{${obj::$str1}}";
    ```

### 3. 常用的字符串操作函数

---
STARTED 2018/11/19 ChenJian

MODIFIED 2018/11/19 ChenJian