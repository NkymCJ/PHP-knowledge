# PHP-PDO

PDO操作数据库

### 1. 创建连接

连接是通过创建PDO基类的实例而建立的。与MysqlI不同的是，它可以访问多种数据库

 > $dbh = new PDO($dsn, $dbUser, $dbPass); 

1. $dsn 必需。数据源，包括$dbms（数据库管理系统）、$host、$post、$dbName
2. $user 必需。数据库用户名称
3. $pass 必需。数据库用户密码
4. 此处还可以传递其他参数进行初始化设置，例如长连接和PDO错误处理模式等设置，此处不做介绍
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

### x. 整合


### y. 总结
