在许多应用程序内，尤其是“后台”类型的应用程序，很多时候都会用到frameset，也就是HTML的框架页面。

区分一下：

- PHP“框架”是php framework，泛指PHP的一种辅助应用程序，比如SpeedPHP框架
- HTML的“框架”是frameset，iframe等，泛指HTML的一个标签，主要用于在页面内显示别的网页。

本文主要讲述的是frameset，也就是HTML的页面框架的使用，还有一些常见问题的处理。

在HTML页面中，使用frameset其实相等于使用&lt;a&gt;标签来链接一个网页，只是该网页的显示是在当前页面之中。所以，在frameset的属性——网址（src），同样需要使用url()函数来进行网址的生成（生成URL）。

[例子下载](images/7.zip)

例子：

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>无标题文档</title>
    </head>

    <frameset rows="80,*" frameborder="no" border="0" framespacing="0">
    <frame src="<{url c="main" a="top"}>" name="topFrame" scrolling="No" noresize="noresize" id="topFrame" title="topFrame" />
    <frame src="<{url c="main" a="bottom"}>" name="mainFrame" id="mainFrame" title="mainFrame" />
    </frameset>
    <noframes><body>
    </body>
    </noframes></html>
    
这里是一个比较标准的frameset实例，页面被分为上下两个区域，加上本身页面一共是三个页面（HTML）
从例子可以看出，frameset的链接地址src，使用的仍然是url()来进行网址的生成。

protected/controller/MainController.php文件

    <?php
    class MainController extends BaseController {
        function actionIndex(){
            $this->display("main_index.html");
        }

        function actionTop(){
            $this->display("main_top.html");
        }
        
        function actionBottom(){
            $this->display("main_bottom.html");
        }
    }
    
这里MainController.php文件，通过三个动作actionIndex()，actionTop()，actionBottom()分别生成了三个页面，对应HTML页面中的三个页面。

从上面例子可以看出，其实frameset是多个页面的集合，所以从SpeedPHP的角度来看，就需要有多个action来一一对应多个页面。

常见问题：

1. frameset页面空白？ 可以检查一下页面编码（要统一编码），比如UTF8，需要检查页面的<meta>，PHP文件和模板文件的文件编码等等。

2. 要传递一个参数到某个frame？比如说上面例子中，我们需要传递一个ID到top模板中，那么就需要在url()构造的地址中继续传递：<{url c="main" a="top" id=1000}>，然后在MainController.php的function actionTop()内可以使用arg('id')接收并进行处理。

3. 页内框架（iframe）怎么样使用？ 和frameset一样，iframe也需要通过url()来构造iframe的src地址。比如：&lt;iframe src="<{url c="main" a="myhtml"}>"&gt;&lt;/iframe&gt;。

4. 在框架（frameset）内点击某个链接，希望是另一个框架（frame）改变并显示链接的页面，怎么做？ 每个frameset都有自己的name属性，比如上面的top部分的frame的name是name="topFrame"，所以，可以在链接&lt;a&gt;中的target属性中设置为topFrame，那么点击这个链接就会在top中打开了。&lt;a href="<{url c="main" a="othertop"}>" target="topFrame"&gt;点击这里&lt;/a&gt;