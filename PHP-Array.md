# PHP-Array

PHP 数组

### 1. 初始化

用 array() 来新建一个数组。它接受任意数量用逗号分隔的 键（key） => 值（value）对。PHP 5.4+，用 [] 替代 array()

- 键（key）可以是 整数integer 或 字符串string

    - PHP数组可以同时含有 integer 和 string 类型的键名，因为 PHP 实际并不区分索引数组和关联数组

    - 如果对给出的值没有指定键名，则取当前最大的整数索引值，而新的键名将是 该值+1 。如果指定的键名已经有了值，则该值会被覆盖

    - 包含有合法整型值的字符串会被转换为整型。例如键名 "8" 实际会被储存为 8。但是 "08" 则不会强制转换，因为其不是一个合法的十进制数值

    - 浮点数也会被转换为整型，意味着其小数部分会被舍去。例如键名 8.7 实际会被储存为 8

    - 布尔值也会被转换成整型。即键名 true 实际会被储存为 1 而键名 false 会被储存为 0

    - Null 会被转换为空字符串，即键名 null 实际会被储存为 ""

    - 数组和对象不能被用为键名。坚持这么做会导致警告：Illegal offset type

    - *如果在数组定义中多个单元都使用了同一个键名，则只使用了最后一个，之前的都被覆盖了*

- 值（value）可以是任意类型的值

- 最后一个数组单元之后的逗号可以省略。通常用于单行数组定义中，例如常用 array(1, 2) 而不是 array(1, 2, ) 。对多行数组定义通常保留最后一个逗号，这样要添加一个新单元时更方便

示例：
```
$arr = array(
    'foo' => 'bar', 
    'bar' => 'foo', 
    ...
);
$arr = [
    'foo' => 'bar', 
    'bar' => 'foo', 
    ...
];

// 混合使用 integer 和 string 类型的键名
$arr = array(
    "foo" => "bar",
    "bar" => "foo",
    100 => -100,
    -100 => 100,
);
var_dump($arr);
// 打印 array(4) { ["foo"]=> string(3) "bar" ["bar"]=> string(3) "foo" [100]=> int(-100) [-100]=> int(100) }

// 没有键名的索引数组
$arr = array("foo", "bar", "hello", "world");
var_dump($arr);
// 打印 array(4) { [0]=> string(3) "foo" [1]=> string(3) "bar" [2]=> string(5) "hallo" [3]=> string(5) "world" }

// 键名强制转换和覆盖值
$arr = array(
    1 => "a",
    "1" => "b",
    1.5 => "c",
    true => "d",
);
var_dump($arr); 
// 打印array(1) {[1]=>string(1) "d"}

// 自动赋予键名
$array = array(
    "a",
    "b",
    6 => "c",
    "d",
);
var_dump($array);
// 打印 array(4) { [0]=> string(1) "a" [1]=> string(1) "b" [6]=> string(1) "c" [7]=> string(1) "d" }
// 因为在 "d" 之前最大的整数键名是 6 ，所以它的键名为 7（6+1）
```

### 2. 访问

1. 数组单元可以通过 array[key] 语法或 array{key} 来访问

2. PHP 5.4+ 可以直接对函数或方法调用的结果进行数组解引用

示例：
```
function getArray()
{
    return array(1, 2, 3);
}
// PHP 5.4-
$tmp = getArray();
$arr = $tmp[1];
var_dump($arr);
// PHP 5.4+
$arr = getArray()[1];
var_dump($arr);
```

### 3. 用方括号的方法操作数组

1. 通过在方括号内指定键名来给数组赋值。也可以省略键名，只使用 []

    *如果方括号没有指定键名，则取当前最大整数索引值，新的键名将是该值加上 1（但是最小为 0）。如果当前还没有整数索引，则键名将为 0*

    *这里所使用的最大整数键名不一定当前就在数组中。它只要在上次数组重新生成索引后曾经存在过就行，详见示例*

2. 要修改某个值，通过其键名赋予其新值

3. 要删除某键值对，对其调用 unset() 函数


示例：
```
$arr = array(5 => 1, 12 => 2);
$arr[] = 56;
var_dump($arr);
// 打印 array(3) { [5]=> int(1) [12]=> int(2) [13]=> int(56) }

$arr["x"] = 42;
var_dump($arr);
// 打印 array(4) { [5]=> int(1) [12]=> int(2) [13]=> int(56) ["x"]=> int(42) }

unset($arr[5]);
var_dump($arr);
// 打印 array(3) { [12]=> int(2) [13]=> int(56) ["x"]=> int(42) }

// 删除数组
unset($arr);

// 3.1 最大整数索引值问题
// 新建一个数组
$arr = array(1, 2, 3, 4, 5);
var_dump($arr);
print '<br/>';

// 删除其中的所有元素，但保持数组本身不变:
foreach ($arr as $i => $value) {
    unset($arr[$i]);
}
var_dump($arr);
print '<br/>';

// 添加一个单元（注意新的键名是 5，而不是 0）
$arr[] = 6;
var_dump($arr);
print '<br/>';

// 重新索引：
$arr = array_values($arr);
$arr[] = 7;
var_dump($arr);

// 补充： array_values() 函数
// array_values(array) 函数返回一个包含给定数组array中所有键值的数组，但不保留键名。此数组使用数值键，从 0 开始并以 1 递增
```


