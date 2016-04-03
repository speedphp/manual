在日常的编程工作中，我们经常会使用到$_POST/$_GET来获取表单提交数据以及URL参数，而sp框架提供了arg()函数来简化了$_POST/$_GET的一些使用过程。

arg()是框架内置函数，可以有两个参数，第一个参数是将要获取的参数名称，为空则返回全部参数的数组。第二个参数是默认值，当需要获取的参数为空时，将返回该默认值。

> 新版已经使用arg()函数来代替原来的控制器spArgs方法，这样写起来更为方便些。

下面我们来看看留言本中的arg()使用：

留言表单：

    <form action="<{url c="main" a="write"}>" method=POST>
    <p>您的名字：<input type=text name='name' size=40></p>
    <p>留言标题：<input type=text name='title' size=40></p>
    <p>留言内容：</p>
    <p><textarea name=contents cols=60 rows=6></textarea></p>
    <p><input type=submit value=" 提交 "></p>
    </form> 

表单中我们将提交name，title和contents等参数。然后在程序当中：

    ...
    $name = arg("name", "jake"); // 可以获取到表单的name，第二个参数是当name没有值时返回的默认值
    $title = arg("title", "这里是默认标题"); // 可以获取到表单的title
    ...        
    
或者可以用：

    ...
    if( $name = arg("name") ){
    // name被提交

    }else{
    // 没有提交
    }
    ...     
    
同时，如果arg()没有输入参数，将返回全部的提交参数： 

    ...
    dump(arg()); // 该语句在开发中常用作调试用
    ...                                                 