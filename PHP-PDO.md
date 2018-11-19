# PHP-PDO

PDO操作数据库

### 1. 连接

 > $dbh = new PDO($dsn, $user, $pass, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION)); 

解释：连接是通过创建PDO基类的实例而建立的。与MysqlI不同的是，它可以访问多种数据库

1. $dsn 必需。数据源，包括 $dbms（数据库管理系统）、$host、$post、$dbName

2. $user 必需。数据库用户名称

3. $pass 必需。数据库用户密码

4. PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION 此处为设置错误报告为主动抛出异常，还可以传递其他参数进行初始化设置，例如长连接等设置。此处的属性设置也可以再之后设置（部分属性之后设置将会无效，例如长连接）

5. return 返回值。连接数据成功后，返回一个PDO类的实例 $dbh 给脚本使用，此连接在 $dbh 的生存周期中保持活动。若想关闭连接，需要销毁 $dbh 以确保它的所有引用都被删除（销毁方法，赋 NULL 值给 $dbh）。若没有明确进行这种处理，则PHP会在脚本结束时自动关闭连接

*如果连接过程中出现任何错误，都将抛出一个 PDOException异常对象。如果我们不进行捕获异常处理，zend引擎将默认结束脚本并显示一个回溯跟踪，此回溯跟踪可能泄露完整的数据库连接细节，例如用户名、密码等信息。所以我们必需进行捕获处理，使用 catch语句 或 set_exception_handler() 函数捕获处理异常*

示例：
```
$dbms = 'mysql'; //数据库类型
$host = 'localhost'; //数据库主机名
$post = '3306'; //数据库主机名
$dbName = 'demo'; //使用的数据库
$user = 'root'; //数据库连接用户名
$pass = '123456'; //对应的密码
$dsn = "$dbms: host=$host;port=$post;dbname=$dbName";
try {
    // 初始化一个PDO实例$dbh
    $dbh = new PDO($dsn, $user, $pass);
    echo "连接成功<br/>";
    // 运行完成，需要销毁实例$dbh，关闭连接
    $dbh = null;
} catch (PDOException $e) { // 抛出PDOException异常对象$e
    // 异常处理
    die($e);
}
```

### 2. 事务

解释：
1. 事务的四大特性（ACID）：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）以及持久性（Durability）

2. 在一个事务中执行的任何操作，即使是分阶段执行的，也能保证安全地应用于数据库，并在提交时不会受到来自其他连接的干扰。事务操作也可以根据请求自动撤销（前提是还未提交事务）

3. 事务通常是通过把一批查询操作“积蓄”起来然后使之同时生效而实现的，大大地提高了效率

4. 如果需要一个事务，则必须用 PDO::beginTransaction() 方法来启动。如果底层驱动不支持事务，则抛出一个 PDOException异常。一旦开始了事务，可用 PDO::commit()（提交操作）或 PDO::rollBack()（回滚操作）来完成此次事务

*PDO仅在驱动层检查是否具有事务处理能力，如果某些运行时条件意味着事务不可用，且数据库服务接受请求去启动一个事务，PDO::beginTransaction() 将仍然返回 TRUE 而且没有错误（例如：在MySQL数据库的MyISAM数据表中使用事务，MyISAM数据表不支持事务，但是MySQL具有事务处理能力）*

示例：
```
try {
    $dbh = new PDO($dsn, $user, $pass);
} catch (PDOException $e) {
    die($e);
}
try {
    // 设置错误报告模式为ERRMODE_EXCEPTION，主动抛出Exception异常
    $dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    // 开启事务
    $dbh->beginTransaction();
    $dbh->exec("insert into users (`name`, `gender`) values ('Alice', '0')");
    $dbh->exec("insert into users (`name`, `gender`) values ('Joe', '1')");
    // 提交事务
    $dbh->commit();
} catch (Exception $e) {
    // 出错后进行回滚
    $dbh->rollBack();
    die($e);
}
```

### 3. 预处理

> bool PDOStatement::execute ([ array $input_parameters ] )

解释：通过在查询语句中使用占位符（? / :），提高效率，确保不会发生SQL注入

1. 执行预处理语句。如果预处理语句含有参数标记，执行时必须选择以下其中一种做法：

    + 调用PDOStatement::bindParam()绑定 PHP 变量到参数标记，此处不多做介绍

    + 传递一个只作为输入参数值的数组

2. $input_parameters 可选。不能绑定多个值到一个预处理语句参数。元素个数和预处理语句参数一样多
    *使用?占位符：array(0)*
    *使用:占位符：array(':name' => 'name')*
    *当元素个数超过预处理语句参数时，执行将会失败并抛出一个错误*

3. return 返回值。成功时返回TRUE，或者在失败时返回FALSE

示例：
```
try {
    $dbh = new PDO($dsn, $user, $pass);
} catch (PDOException $e) {
    die($e);
}
try {
    $dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    // 使用:占位符
    $gender = 0;
    $birthday = '2018-11-11';
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = :gender AND `birthday` = :birthday");
    if ($stmt->execute(array(':gender' => $gender, ':birthday' => $birthday))) {
        while ($row = $stmt->fetch()) {
            print_r($row);
        }
    }

    // 使用:占位符（bindParam()）
    $gender = 0;
    $birthday = '2018-11-11';
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = :gender AND `birthday` = :birthday");
    $stmt->bindParam(':gender', $gender);
    $stmt->bindParam(':birthday', $birthday);
    if ($stmt->execute()) {
        while ($row = $stmt->fetch()) {
            print_r($row);
        }
    }

    // 使用?占位符
    $gender = 0;
    $birthday = '2018-11-11';
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = ? AND `birthday` = ?");
    if ($stmt->execute(array($gender,$birthday))) {
        while ($row = $stmt->fetch()) {
            print_r($row);
        }
    }

    // 使用?占位符（bindParam()）
    $gender = 0;
    $birthday = '2018-11-11';
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = ? AND `birthday` = ?");
    // 从1开始，此处后面还可以补充两个参数（类型与长度）
    $stmt->bindParam(1, $gender);
    $stmt->bindParam(2, $birthday);
    if ($stmt->execute()) {
        while ($row = $stmt->fetch()) {
            print_r($row);
        }
    }
} catch (Exception $e) {
    die($e);
}
```

### 4. 其他常用方法

> public PDOStatement PDO::prepare ( string $statement [, array $driver_options = array() ] )

解释：为预处理准备待执行的SQL语句。SQL 语句可以包含零个或多个参数占位标记，格式是:name或?，当它执行时将用真实数据取代。在同一个 SQL 语句里，命名形式和问号形式不能同时使用，只能选择其中一种参数形式。用参数形式绑定用户输入的数据，不要直接字符串拼接到查询里

1. $statement 必需。SQL语句

2. $driver_options 可选。为返回的PDOStatement对象设置属性

3. return 返回值。返回PDOStatement对象，或者返回FALSE或抛出PDOException（取决于PDO::ATTR_ERRMODE）

> mixed PDOStatement::fetch ([ int $fetch_style [, int $cursor_orientation = PDO::FETCH_ORI_NEXT [, int $cursor_offset = 0 ]]] )
     
解释：从PDO::prepare返回的PDOStatement对象相关的结果集中获取下一行

1. $fetch_style 可选。控制下一行返回的格式，默认PDO::FETCH_BOTH，返回一个索引为结果集列名和以0开始的列号的数组

2. $cursor_orientation 可选。对于PDOStatement对象表示的可滚动游标，该值决定了哪一行将被返回给调用者。此值默认为PDO::FETCH_ORI_NEXT。要使用，必须在PDO::prepare() 预处理SQL语句时，设置PDO::ATTR_CURSOR属性为PDO::CURSOR_SCROLL

3. $cursor_offset 可选。若cursor_orientation参数设置为PDO::FETCH_ORI_ABS，此值指定结果集中想要获取行的绝对行号。若cursor_orientation参数设置为PDO::FETCH_ORI_REL，此值指定想要获取行相对于调用PDOStatement::fetch()前游标的位置

4. return 返回值。成功时返回的值依赖于提取类型，失败返回FALSE

> bool PDOStatement::bindParam ( mixed $parameter , mixed &$variable [, int $data_type = PDO::PARAM_STR [, int $length [, mixed $driver_options ]]] )

解释：绑定一个PHP变量到用作预处理SQL语句中对应占位符的位置。只能在PDOStatement::execute()调用时才能取其值

1. $parameter 必需。需替换的参数名，例如:name或?

2. &$variable 必需。绑定到SQL语句参数的PHP变量名

3. $data_type 可选。指定参数的类型

4. $length 可选。指定参数的长度

5. $driver_options 可选。

6. return 返回值。成功时返回TRUE，失败时返回FALSE 

*参数以只读的方式用来建立查询（区别于部分驱动能修改）*


---
STARTED 2018/11/12 ChenJian

MODIFIED 2018/11/19 ChenJian
