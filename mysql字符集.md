1.var_export和var_dump区别
    var_export 
    输出合法的php代码；可以传递第二个参数，从而返回结果；资源类型返回null
    var_dump 
    输出结构，展示值类型；资源类型输出resource；可以用控制输出函数捕获输出ob_get_contents()
2.MYSQL字符集
    character字符集就是 元字符 对应编码的映射表
    collation字符序，同一字符集间字符比较规则
    字符集可以对应多个字符序，一个字符序对应一个字符集
    字符序命名 _ci表示大小写不敏感 _cs表示大小写敏感 _bin表示按字符集编码比较
    
    字符转换过程
    1). MySQL Server收到请求时将请求数据从character_set_client转换为character_set_connection；
    2). 进行内部操作前将请求数据从character_set_connection转换为内部操作字符集，其确定方法如下：
      • 使用每个数据字段的CHARACTER SET设定值；
      • 若上述值不存在，则使用对应数据表的DEFAULT CHARACTER SET设定值(MySQL扩展，非SQL标准)；
      • 若上述值不存在，则使用对应数据库的DEFAULT CHARACTER SET设定值；
      • 若上述值不存在，则使用character_set_server设定值。
    3). 将操作结果从内部操作字符集转换为character_set_results。
    
    检测字符集问题的一些手段
      • SHOW CHARACTER SET;
      • SHOW COLLATION;
      • SHOW VARIABLES LIKE ‘character%’;
      • SHOW VARIABLES LIKE ‘collation%’;
      • SQL函数HEX、LENGTH、CHAR_LENGTH
      • SQL函数CHARSET、COLLATION
      
    其他注意事项
      • my.cnf中的default_character_set设置只影响mysql命令连接服务器时的连接字符集，不会对使用libmysqlclient库的应用程序产生任何作用！
      • 对字段进行的SQL函数操作通常都是以内部操作字符集进行的，不受连接字符集设置的影响。
      • SQL语句中的裸字符串会受到连接字符集或introducer设置的影响，对于比较之类的操作可能产生完全不同的结果，需要小心！
      • 使用mysql C API时，初始化数据库句柄后马上用mysql_options设定MYSQL_SET_CHARSET_NAME属性为utf8，这样就不用显式地用 SET NAMES语句指定连接字符集，且用mysql_ping重连断开的长连接时也会把连接字符集重置为utf8；
      • 对于mysql PHP API，一般页面级的PHP程序总运行时间较短，在连接到数据库以后显式用SET NAMES语句设置一次连接字符集即可；但当使用长连接时，请注意保持连接通畅并在断开重连后用SET NAMES语句显式重置连接字符集。
3.PHP执行流程
    SCANNING(LEXING)->PARSING->COMPILATION->EXECUTION
    LEXING将代码分解为一个个TOKENS
    PARSING将TOKEN转换为有意义的表达式
    COMPILATION将表达式编译为OPCODES
    EXECUTION顺序执行OPCODES，每次一条