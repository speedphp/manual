本章我们会讲解伪静态的配置，还有URL构造的方法等相关内容。

> 新版的sp框架，伪静态功能是内置支持的。

###一、特色

在不到80行代码里面，实现了功能强大的php伪静态路由功能。（包括伪静态路由和url产生）
精简的代码带来非常高的执行效率，对比旧版的UrlRewrite扩展速度上有三倍的提升。

> 当然，旧版框架对比其他大型PHP框架的伪静态已经非常轻量级和快速了。

- 支持moduels多模块。
- 支持URL和参数定制。
- 支持http开头的域名适配，对跨站构造URL有良好的支持。
- 支持泛域名适配，如 *.example.com 的适配。
- 规则更简单，更直观，更容易配置了。
- 通过url()函数即可构造URL地址。

###二、服务器配置

一般初学者使用伪静态时，首先遇到的问题是产生“404找不到页面”的情况。这情况通常都是因为没有正确配置服务器的原因。

接下来我们介绍一下较为常见的服务器的伪静态配置方法。

**Apache**

在默认情况下，sp框架在程序根目录已经自带.htaccess文件，该文件在Apache服务器下面是自动加载并且已经完成配置的。

根目录的.htaccess文件内容是：

    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . index.php [L]
    </IfModule>
    
注意，有时候我们会发现Apache服务器即使已经有了.htaccess文件，但是还是会发生404的情况，这样就需要检查一下Apache本身的配置是否开启“文件配置”的选项了。

1. 找到httpd.conf文件，一般windows在apache/conf/httpd.conf目录里面，linux在/etc/httpd/conf/httpd.conf。
2. 打开并且搜索到您的web根目录配置。如：

        <Directory "/var/www/html">
        　　AllowOverride None 
        </Directory>
        
    改为

        <Directory "/var/www/html">
        　　AllowOverride All 
        </Directory>
    主要是修改AllowOverride的值，AllowOverride意思是：是否能通过.htaccess文件配置来覆盖httpd.conf的配置。

3. 检查以下语句前面有没有#号，有的话去掉#。如果无法找到该行配置，则在httpd.conf文件最后增加。

       LoadModule rewrite_module modules/mod_rewrite.so 
   
4. 重启Apache即可。

**Nginx**

Nginx现在非常流行，大多数时候已经替代了Apache的地位。



**新浪SAE平台**

config.yaml是SAE的配置文件，以下是伪静态的实例：

    name: speedweb
    version: 3

    handle:
    - rewrite: if (!-d && !-f) goto "/index.php?%{QUERY_STRING}"
    - hostaccess: if (%{REQUEST_URI} ~ "/protected/") deny "all"

###三、框架伪静态配置

sp框架的伪静态配置，在protected/config.php文件里面，大概是这样：

    $config = array(
        'rewrite' => array(
            'admin/index.html' => 'admin/main/index',
            'admin/<c>_<a>.html'    => 'admin/<c>/<a>', 
            '<c>/<a>'          => '<c>/<a>',
            '/'                => 'main/index',
        ),
    );
    
这里rewrite数组，就是我们的伪静态配置，可以看到，左边key值是URL地址，而右边是指向的modules/controller/action（当然没有modules也行）。

如果需要关闭rewrite，可以设置'rewrite' => null,那么全部URL都会变成原来的“**/index.php?m=模块&c=控制器&a=方法&参数名=参数值**”的形式。

我们来了解几个规则，非常简单：

1. &lt;m&gt;&lt;c&gt;&lt;a&gt;分别指代modules，controller，action。
2. 其他的<单词>，都是_GET的参数名称。
3. 越是明确指向的URL配置，越要放前面。比如说'admin/index.html'，要放在'admin/&lt;c&gt;_&lt;a&gt;.html'的前面，因为'admin/index.html'是明确的。而'&lt;c&gt;/&lt;a&gt;'和'/'就被放到最后了。

我们来看看各种配置对应表：

类型 | 配置 | 实现URL | 指向控制器/方法（或 模块/控制器/方法） | GET参数
---|---|---|---|---
固定URL | 'index.html' => 'main/index' | /index.html | main/index | -
固定URL | '123.json' => 'view/number' | /123.json | view/number | -
固定URL | 'admin/index.html' => 'admin/main/index' | /admin/index.html | admin/main/index | -
使用&lt;c&gt;，&lt;a&gt; | '&lt;c&gt;_&lt;a&gt;.html' => '&lt;c&gt;/&lt;a&gt;' | /控制器_方法.html | 控制器/方法 | -
使用&lt;m&gt;，&lt;c&gt;，&lt;a&gt; | '&lt;m&gt;/&lt;c&gt;_&lt;a&gt;.html' => '&lt;m&gt;/&lt;c&gt;/&lt;a&gt;' | /模块/控制器_方法.html | 模块/控制器/方法 | -
使用&lt;m&gt;，&lt;c&gt;，&lt;a&gt;，固定部分URL | 'user-&lt;a&gt;.html' => 'user/&lt;a&gt;' | /user-方法.html | user/方法 | -
使用参数 | 'blog-&lt;id&gt;.do' => 'user/blog' | /blog-9527.do | user/blog | $_GET["id"] = 9527
使用参数 | 'admin/user/&lt;username&gt;' => 'admin/user/detail' | /admin/user/jake | admin/user/detail | $_GET["username"] = "jake"
使用参数 | 'u/&lt;uid&gt;/album.html' =&gt; 'user/album', | /u/123/album.html | user/album | $_GET["uid"] = 123
使用参数 | 'page-&lt;username&gt;/&lt;tid&gt;' =&gt; 'page/view', | /page-jake/10086 | page/view | $_GET["username"] = 'jake'，$_GET["tid"] = 10086
泛域名 | 'http://&lt;username&gt;.speedphp.com/' =&gt; 'main/index' | http://jake.speedphp.com | main/index | $_GET["username"] = 'jake'
泛域名 | 'http://&lt;shopname&gt;.shop.speedphp.com/article-&lt;id&gt;.html' =&gt; 'article/show' | http://ak47.shop.speedphp.com/article-520.html | article/show | $_GET["shopname"] = 'ak47'，$_GET["id"] = 520

###四、URL地址函数

当配置好上述的规则后，我们可以通过url()函数，来生成URL地址。

url()函数有三个参数：$c, $a, $param

1. $c参数是控制器名称，对应到protected/controller目录下面的controller类，如果用到modules模块开发，那么参数值是“模块名/控制器名”。对应该模块目录下的controller类。
2. $a参数是方法名称。对应controller类里面的，带action前缀的方法。
3. $param是参数数组，键是参数名称，值是参数值。

**在没有伪静态的时候**：

> 不设置伪静态就是配置：'rewrite' => null

类型 | 控制器内使用url()函数 | 模板内使用 | 显示结果
---|---|---|---
进入控制器/方法 | url("view", "index"); | <{url c="view" a="index"}> | /index.php?c=view&a=index
进入模块/控制器/方法 | url("admin/view", "index"); | <{url c="admin/view" a="index"}> | /index.php?m=admin&c=view&a=index
带参数| url("view", "index", array("page" => 100)); | <{url c="view" a="index" page="100"}> | /index.php?c=view&a=index&page=100
带参数| url("admin/view", "index", array("page" => 100, "sort" => "desc")); | <{url c="admin/view" a="index" page="100" sort="desc"}> | /index.php?m=admin&c=view&a=index&page=100&sort=desc

**在开启伪静态之后**：

开启伪静态之后，会根据rewrite配置而定URL的生成方式。

配置 | 控制器内使用url()函数 | 模板内使用 | 显示结果 | 指向控制器/方法（或 模块/控制器/方法） | GET参数
---|---|---|---|---
'index.html' => 'main/index' | url("main", "index"); | <{url c="main" a="index"}> | /index.html | main/index | -
'123.json' => 'view/number' | url("view", "number"); | <{url c="view" a="number"}> | /123.json | view/number | -
'admin/index.html' => 'admin/main/index' | url("admin/main", "index"); | <{url c="admin/main" a="index"}> | /admin/index.html | admin/main/index | -
'&lt;c&gt;_&lt;a&gt;.html' => '&lt;c&gt;/&lt;a&gt;' | url("mycontrol", "myaction"); | <{url c="mycontrol" a="myaction"}> |  /mycontrol_myaction.html | mycontrol/myaction | -
'&lt;m&gt;/&lt;c&gt;_&lt;a&gt;.html' => '&lt;m&gt;/&lt;c&gt;/&lt;a&gt;' | url("mod/mycontrol", "myaction"); | <{url c="mod/mycontrol" a="myaction"}> |  /mod/mycontrol_myaction.html | mod/mycontrol/myaction | -
'user-&lt;a&gt;.html' => 'user/&lt;a&gt;'| url("user", "myaction"); | <{url c="user" a="myaction"}> | /user-myaction.html | user/myaction | -
'blog-&lt;id&gt;.do' => 'user/blog'| url("user", "blog", array("id"=>9527)); | <{url c="user" a="blog" id="9527"}>  | /blog-9527.do | user/blog | $_GET["id"] = 9527
'blog-&lt;id&gt;.do' => 'user/blog'| url("user", "blog", array("page"=>2, "id"=>9527)); | <{url c="user" a="blog" page="2" id="9527"}>  | /blog-9527.do**?page=2** | user/blog | $_GET["id"] = 9527, **$_GET["page"] = 2**
'admin/user/&lt;username&gt;' => 'admin/user/detail'| url("admin/user", "detail", array("username"=>"jake")); | <{url c="admin/user" a="detail" username="jake"}> | /admin/user/jake | admin/user/detail | $_GET["username"] = "jake"
'u/&lt;uid&gt;/album.html' =&gt; 'user/album', | url("user", "album", array("uid"=>123)); | <{url c="user" a="album" uid="123"}>  |/u/123/album.html | user/album | $_GET["uid"] = 123
'u/&lt;uid&gt;/album.html' =&gt; 'user/album', | url("user", "album", array("uid"=>123, "sort"=>"2")); | <{url c="user" a="album" uid="123" sort="2"}>  |/u/123/album.html?sort=2 | user/album | $_GET["uid"] = 123, $_GET["sort"] = 2
'page-&lt;username&gt;/&lt;tid&gt;' =&gt; 'page/view', | url("page", "view", array("username"=>"jake", "tid"=>"10086")); | <{url c="user" a="album" username="jake" tid="10086"}>| /page-jake/10086 | page/view | $_GET["username"] = 'jake'，$_GET["tid"] = 10086
'http://&lt;username&gt;.speedphp.com/' =&gt; 'main/index' | url("main", "index", array("username"=>"jake")); | <{url c="main" a="index" username="jake"}> | http://jake.speedphp.com | main/index | $_GET["username"] = 'jake'
'http://&lt;shopname&gt;.shop.speedphp.com/article-&lt;id&gt;.html' =&gt;'article/show'| url("article", "showindex", array("shopname"=>"ak47", "id"=>520)); | <{url c="article" a="show" shopname="ak47" id="520"}> | http://ak47.shop.speedphp.com/article-520.html | article/show | $_GET["shopname"] = 'ak47'，$_GET["id"] = 520
'http://&lt;shopname&gt;.shop.speedphp.com/article-&lt;id&gt;.html' =&gt;'article/show'| url("article", "showindex", array("shopname"=>"ak47", "id"=>520, "page"=>2)); | <{url c="article" a="show" shopname="ak47" id="520" page="2"}> | http://ak47.shop.speedphp.com/article-520.html?page=2 | article/show | $_GET["shopname"] = 'ak47'，$_GET["id"] = 520, $_GET["page"] = 2

这里注意两个问题：

1. 比配置多出来的参数，那么会以?参数名=参数值的方式跟在URL后面。上表应该可以看得出来。
2. **如果发现取得的URL不是想要的，可以调整一下rewrite配置规则的前后顺序，多几次调整就会正确，并且找到规律。**