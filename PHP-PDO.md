# PHP-PDO

PDO操作数据库

### 1. 连接

连接是通过创建PDO基类的实例而建立的。与MysqlI不同的是，它可以访问多种数据库

 > $dbh = new PDO($dsn, $user, $pass, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION)); 

1. $dsn 必需。数据源，包括$dbms（数据库管理系统）、$host、$post、$dbName
2. $user 必需。数据库用户名称
3. $pass 必需。数据库用户密码
4. PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION 此处为设置错误报告为主动抛出异常，还可以传递其他参数进行初始化设置，例如长连接等设置。此处的属性设置也可以再之后设置（部分属性之后设置将会无效，例如长连接）
5. return 返回值。连接数据成功后，返回一个PDO类的实例$dbh给脚本使用，此连接在$dbh的生存周期中保持活动。若想关闭连接，需要销毁$dbh以确保它的所有引用都被删除（销毁方法，赋NULL值给$dbh）。若没有明确进行这种处理，则PHP会在脚本结束时自动关闭连接

*如果连接过程中出现任何错误，都将抛出一个PDOException异常对象。如果我们不进行捕获异常处理，zend引擎将默认结束脚本并显示一个回溯跟踪，此回溯跟踪可能泄露完整的数据库连接细节，例如用户名、密码等信息。所以我们必需进行捕获处理，使用catch语句或set_exception_handler()方法捕获处理异常*

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

事务的四大特性（ACID）：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）以及持久性（Durability）

在一个事务中执行的任何操作，即使是分阶段执行的，也能保证安全地应用于数据库，并在提交时不会受到来自其他连接的干扰。事务操作也可以根据请求自动撤销（前提是还未提交事务）

事务通常是通过把一批查询操作“积蓄”起来然后使之同时生效而实现的，大大地提高了效率

如果需要一个事务，则必须用PDO::beginTransaction()方法来启动。如果底层驱动不支持事务，则抛出一个PDOException异常。一旦开始了事务，可用 PDO::commit()（提交操作）或PDO::rollBack()（回滚操作）来完成此次事务

*PDO仅在驱动层检查是否具有事务处理能力，如果某些运行时条件意味着事务不可用，且数据库服务接受请求去启动一个事务，PDO::beginTransaction()将仍然返回TRUE而且没有错误（例如：在MySQL数据库的MyISAM数据表中使用事务，MyISAM数据表不支持事务，但是MySQL具有事务处理能力）*

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

通过在查询语句中使用占位符（? / :），提高效率，确保不会发生SQL注入

 > bool PDOStatement::execute ([ array $input_parameters ] )

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
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = :gender AND `birthday` = :birthday");
    if ($stmt->execute(array(':gender' => 0, ':birthday' => '2018-11-11'))) {
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
    $stmt = $dbh->prepare("SELECT * FROM `users` WHERE `gender` = ? AND `birthday` = ?");
    if ($stmt->execute(array(0,'2018-11-11'))) {
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

 > mixed PDOStatement::fetch ([ int $fetch_style [, int $cursor_orientation = PDO::FETCH_ORI_NEXT [, int $cursor_offset = 0 ]]] )

 > bool PDOStatement::bindParam ( mixed $parameter , mixed &$variable [, int $data_type = PDO::PARAM_STR [, int $length [, mixed $driver_options ]]] )

### 5. 总结
好像没什么好总结的 -,-
