### 1.攻防世界：ics-05:
    得到url http://111.198.29.45:52252/index.php?page=index
    这个page为文件包含漏洞，使用php://filter协议可以看到index.php的源码
    源码中存在preg_match($pat, $rep, $str);
    $pat为正则表达式字段
    $rep为被替换字段
    $str为待处理字段
    漏洞：当$pat的正则为/xxxx/e 以e结尾的，将执行$rep的字符串！
    payload：pat=/123/e&rep=phpinfo()&str=123将执行phpinfo
### 2.攻防世界：ics-06:
    爆破id，id=2333时得到flag
    有点坑= =