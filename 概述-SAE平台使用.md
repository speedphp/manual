新版sp框架直接支持在新浪云计算平台（SAE）上面运行。

> 不再需要像旧版一样区分SAE版本和非SAE版本的做法，目前SAE对许多PHP原生环境支持都比较好了。

当然，在SAE上面使用新版sp框架，还是得注意一下：

protected/config.php配置文件中，设置一下view的compile_dir属性，以便能使用SAE的文件系统进行模板编译。

    ....
    $domain = array(
        "speedweb.applinzi.com" => array( // SAE配置
            'view'  => array('compile_dir'=>SAE_TMP_PATH),
        ),
    );
    ....

**SAE的伪静态方法配置**

config.yaml是SAE的配置文件，以下是伪静态的实例：

    name: speedweb
    version: 3

    handle:
    - rewrite: if (!-d && !-f) goto "/index.php?%{QUERY_STRING}"
    - hostaccess: if (%{REQUEST_URI} ~ "/protected/") deny "all"
