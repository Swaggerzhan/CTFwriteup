# php反序列化漏洞总结
### 1.一般的反序列化漏洞分析方式
    1.寻找 unserialize()函数的参数是否有我们的可控点。
    2.寻找我们的反序列化的目标，重点寻找存在wakeup()或者destruct()魔法函数的类
    3.一层层的研究该类在魔法方法中使用的属性和属性的调用方法，看看是否有可控的属性能实现在当前调用的过程中触发的
### 2.利用phar://扩展php反序列化的攻击面
php反序列化攻击的必要条件：
>1.首先我们必须要有unserailize()
>2.unserailize()函数的参数必须可以控制
>如果不存在这两个条件，根本不可能实现反序列化攻击，但是phar://可以实现

    phar文件包在生成是会以序列化形式储存用户自定义的meta-data！
    phar格式为: xxx<?php xxx;__HALT_COMPILER();?>
    前面内容不限，但必须以__HALT_COMPILER();?>来结尾，这部分的目的就是让phar扩展识别着是一个标准的phar文件。
    
    
例如：
```php
<?php
    class TestObject {
    }
    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```
受影响的函数：
`fileatime`	 `filectime`	`file_exists`	`file_get_contents`
`file_put_contents`	`file`	`filegroup`	`fopen`
`fileinode`	`filemtime`	`fileowner`	`fikeperms`
`is_dir`	`is_executable`	`is_file`	`is_link`
`is_readable`	`is_writable`	`is_writeable`	`parse_ini_file`
`copy`	`unlink`	`stat`	`readfile`
