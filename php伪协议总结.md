# PHP伪协议总结
###1.php.ini里面的两个总要参数allow_url_fopen和allow_url_include
>allow_url_fopen:默认为ON，允许url里的封装协议访问文件
allow_url_include:默认为OFF,不允许包含url里的封装协议包含文件

    php总共支持以下协议
    file:// -访问本地文件系统
    http:// -访问HTTP(s)网站
    ftp:// -访问FTP(s) URLs
    php:// -访问各个输入/输出流
    zlib:// - 压缩流
    data:// - 数据
    glob:// - 查找匹配的文档路径模式
    phar:// - php归档
    ssh2:// - Secure Shell 2
    rar:// - RAR
    ogg:// - 音频流
    expect:// - 处理交互式的流
###2.php://filter
>最常使用的伪协议，一般用于任意文件的读取，有时候也可以用于getshell，在`双OFF`的情况下也可以使用。
php://filter是一种元封装器，用于数据流打开时筛选过滤应用。`这对于一体式(all-in-one)的文件函数非常有用。类似readfile(),file(),file_get_contents(),在数据流读取之前没有机会使用其他过滤器。`

    resource=<要过滤的数据流>这个参数是必须的。他指定了你要筛选过滤的数据流。
    
    read=<读链的筛选列表>这个参数可选。可以设定一个或多个过滤器名称，以管道符号|分隔
    
    write=<写链的筛选列表>这个参数可选。可以设定一个或多个过滤器名称，以管道符|分隔

    <; 两个链表的筛选列表>任何没有以read=或write=作为前缀的筛选器列表会视情况应用于读或者写链
    
    php://filter/[read/write]=string.[rot13/strip_tags/…..]/resource=xxx
    
    filter和string过滤器连用可以对字符串进行过滤。filter的read和write参数有不同的应用场景。read用于include()和file_get_contents(),write用于file_put_contents()中。
    
    php://filter/convert.base64-[encode/decode]/resource=xxx
    
    这是使用的过滤器是convert.base64-encode.它的作用就是读取upload.php的内容进行base64编码后输出。可以用于读取程序源代码经过base64编码后的数据
###3.php://input
        php://input可以访问请求的原始数据的只读流，将post请求的数据当作php代码执行。当传入的参数作为文件名打开时，
    可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。

>需要开启allow_url_include

    注：当enctype=”multipart/form-data”时，php://input是无效的。
    
    
    
###4.file://
    file://伪协议在双OFF的时候也可以使用，用于本地文件包含
    
    注：file://协议必须是绝对路径
    
    
###5.phar://
    PHP 归档，常常跟文件包含，文件上传结合着考察。说通俗点就是php解压缩包的一个函数，解压的压缩包与后缀无关。
    
    phar://test.[zip/jpg/png…]/file.txt
    
    其实可以将任意后缀名的文件(必须要有后缀名)，只要是zip格式压缩的，都可以进行解压，因此上面可以改为phar://test.test/file.txt也可以运行。
    
    
###6.zip:// bzip2:// zlib://
    在双OFF的时候也可以用
    
    zip://test.zip%23file.txt
    
    和phar://一样用于读取压缩文件，不过对于"zip://test.zip#file.txt"中的"#"要编码为"%23".因为url的#后的内容不会被传送
###7.data://text/plain;base64,base编码字符串

    很常用的数据流构造器，将读取后面base编码字符串后解码的数据作为数据流的输入