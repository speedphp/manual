##框架中session／cookie的使用

**session**

session的作用主要在保存会话信息，在访问者浏览网站的期间，对访问者相关的信息进行记录，当浏览器关闭后，会话结束，session数据也就消失了。

PHP框架中session是一个数组，可以通过$_SESSION['key'] = $value的方式对session赋值。如：

    $_SESSION['myname'] = 'Helllo';
    echo $_SESSION['myname'];
    
使用session的前提，是程序已经打开session_start()。

新版sp框架中，session_start()默认没有开启，如果需要开启，请把session_start()放在init()或者入口文件内。

使用session过程中，会遇到输出header与session冲突的问题。通常会提示：
Warning: session_start() [function.session-start]: Cannot send session cache limiter - headers already sent (output started at xxx.php:29) in xxx

引起该问题的原因，是在session_start()之前，存在输出的header或字符。这里给以下几个建议：

- 全局的header输出，要放在入口文件全局定义的位置，不要放入口文件的开头。当然，如果是单个header输出，如文件下载，则可以直接放到某个控制器/方法中。
- 检查文件是否带BOM或其他输出字符，BOM可以通过编辑器来检查，而输出字符，则可以通过在浏览器中另存源文件下来，用记事本打开，看到乱码或空格就证明有输出字符。这些字符可以通过把源文件放到记事本中另存为来消除。
- 可以在入口文件全局定义位置中，加入ob_start();来避免出错。

> session通常还有许多设置，如将session存放到数据库中等等。详情请见php.net文档。
这些session的设置，均可以放到入口文件全局定义位置来设置。

session存放的位置，是在服务器的系统临时中，所以一般而言，session文件中的内容不会被访问者浏览到。

**PHP框架中cookie的使用**

cookie在程序中使用通常作为“保持登录”或是跟踪访问者操作等。和session不同，cookie存放的位置，是在访问者的浏览器缓存中。

cookie的使用有几个要素：过期时间、路径、域。cookie可以通过setcookie函数设置：

setcookie ( COOKIE名字, COOKIE值, 过期时间, 路径, 域名 );

- 过期时间：默认是会话时间长度，和session相同。
- 路径：默认是“/”，设置在当前域名下COOKIE生效的路径。
- 域：域名，默认是当前网站域名。可以设置成 
- “.speedphp.com”来使得整个网站（包括二级域名）都可以读取该COOKIE。


    $value = '这里是设置的值';
    setcookie("TestCookie", $value);  // 该cookie的过期时间是会话时间
    setcookie("TestCookie", $value, time()+3600);   // 该cookie的过期时间是1小时，当前时间time() 加 3600秒（1小时）
    setcookie("TestCookie", $value, time()+3600, "/bbs/", ".speedphp.com");// 该cookie的过期时间是1小时，只在speedphp.com及speedphp.com二级域名的bbs目录下使用。
    
cookie值可以用$_COOKIE数组来获取。

    echo $_COOKIE['TestCookie'];
    
**cookie测试问题**

由于COOKIE是先发送到访问者浏览器中，然后再被浏览器发送到服务器给PHP程序。所以在调试COOKIE的时候，要刷新一下页面才能看到刚才setcookie的值。也就是

    // 第一个页面
    // 假设COOKIE的TestCookie没有值
    setcookie("TestCookie", "我是值");
    echo $_COOKIE['TestCookie']; // 不会输出值，因为浏览器没有发送COOKIE上来。
    // 第二个页面
    echo $_COOKIE['TestCookie']; // 输出“我是值”，因为PHP已经读取到浏览器发送的值。