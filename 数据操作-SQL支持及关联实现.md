本章将介绍sp框架数据SQL操作的使用方法，以及关联查询。

###一、SQL查询方法 query() 

**用法：** query($sql, $params = array())

**参数：**

- $sql参数是需要查询的SQL语句，SQL中输入参数值需要用类似“:foo”、“:bar”的绑定标识来代替。
- $params是参数绑定值列表，键是类似“:foo”、“:bar”的绑定标识，值是输入数据。

**返回值**

query()返回查询结果的二维数组，跟findAll()结果一样。

**举例：**

子查询

    $obj->query(
        "select tbl_student.* from tbl_student, (select student_id from tbl_score where examdate = ':examdate') sc where sc.student_id = tbl_student.student_id group by tbl_student.student_id", 
        array(
            ":examdate" => "2010-10-12"
        )
    );
    
    
UNION联合查询

    $obj->query(
        "(SELECT a FROM t1 WHERE a=:v1 AND B=:v2 ORDER BY a LIMIT 10) UNION (SELECT a FROM t2 WHERE a=:v3 AND B=:v4 ORDER BY a LIMIT 10)", 
        array(
            ":v1" => 10,
            ":v2" => 2,
            ":v3" => 20,
            ":v4" => 3,
        )
    );


JOIN连接查询


    $obj->query("SELECT t1.id,t2.id,t3.id FROM t1,( t2 LEFT JOIN t3 ON (t3.id=t1.id) ) WHERE t1.id=t2.id");

> 对比旧版的findSql()来说，新版的query()最大的不同是通过绑定参数来输入数据的，更加安全。

###二、SQL操作方法 execute()

**用法：** execute($sql, $params = array())

**参数：**

- $sql参数是需要查询的SQL语句，SQL中输入参数值需要用类似“:foo”、“:bar”的绑定标识来代替。
- $params是参数绑定值列表，键是类似“:foo”、“:bar”的绑定标识，值是输入数据。

请注意query()和execute()方法之间的区别：query()是SQL语句查找时使用的，而execute()是SQL语句更新/删除/新建的时候使用的。

> 通俗点说，query()中的SQL语句主要以“SELECT”为开头，而execute()中的SQL语句以“UPDATE/DELETE/CREATE”开头。

> 而且两者的返回值是最大的不同。

**返回值**

execute()返回影响行数，跟update()/delete()方法结果一样。

**举例**

增删改create，update，delete

    $obj->execute("INSERT INTO table_name (col1, col2) VALUES(:col1, col1*2)", array(
        ":col1" => 15,
    ));
    
    $obj->execute("UPDATE table_name SET col1=:col1, col2 = col1*2 WHERE col3 > :col3", array(
        ":col1" => 100,
        ":col3" => 100,
    ));
    
    $obj->execute("DELETE table_name where col1 > :col1 AND col2 < :col2", array(
        ":col1" => 30,
        ":col2" => 50,
    ));
    
    
建表，改变表


    $obj->execute("CREATE TABLE tbl_topic(tid int NOT NULL AUTO_INCREMENT,topic VARCHAR(200) NOT NULL,clicks BIGINT NOT NULL DEFAULT 0,PRIMARY KEY (tid)) DEFAULT CHARSET utf8");
    
    $obj->execute("ALTER TABLE t1 RENAME t2");
    
其他操作

    $obj->execute("REPLACE INTO mysql.user (Host,User,Password) VALUES('%', :username,PASSWORD(:password)) ", array(
        ":username" => "jake",
        ":password" => "123456"
    ));

###三、事务支持

支持SQL就能支持数据库事务，当然数据库类型需要是innoDB。

    $g = new Model("lib_guestbook");
    // 开启事务
    $g->execute("START TRANSACTION"); // 或者是$g->execute("BEGIN");
    // 这里是很多的插入或修改操作等，一般来说查询不需要用事务的。
    $result1 = $g->create(xxx);
    $result2 = $g->update(xxx);
    ...
    // 这里判断操作是否成功，然后回滚或提交事务
        if( false == $result1 || false == $result2 || ... ){ // create、update之类的返回false即是操作失败，也有可能是字段错误
        $g->execute("ROLLBACK");  // 出现问题，事务回滚
    }else{
        $g->execute("COMMIT");  // 没有问题，那么事务提交。
    }
    
以上就是事务的实现，不过一般情况下不需要使用到这些，只有在大并发或数据库管理的时候才需要用到，请谨慎。

> 开发者可以自行覆盖Model的方法来对事务进行封装，因为Model本身的绝大部分函数，对数据库的操作都是一条SQL的，所以对事务的封装没有很大的必要。

> 如果应用程序内，spModel的派生类内，有比较复杂的数据处理，那么将这个处理和事务封装到覆盖的方法里，这是更轻便的OOP做法。

###四、数据关联查询

跟旧版框架不一样的地方是，新版sp框架提倡数据库关联直接通过SQL来进行查询，新版代码也没有对关联进行封装。

理由：

- 封装关联操作，是通过所谓的ORM方式封装一些很古怪的函数和方法来实现。但是这样学习成本非常高。
- 封装关联后，不利于更细致的操作。
- 封装关联后，不能直观地看出查询的条件。在理解代码上存在比较大的困难。
- 封装关联性能方面会比较差，相对直接用SQL的话。

所以：

- 使用query()方法，直接使用SQL语句进行关联查询。
- 关联查询一般只是SQL的join语法，很容易理解，直接把代码功能写出来，也很直观。
- SQL语句是比较基础的web开发知识，普通开发者不用再学习一门ORM封装语言。
- 执行效率非常高。

> 其实换句话来说，SQL语句并非复杂到不敢面对的事情，过度把它封装起来是不是很多旧余。

