# BUUCTF做题总结
### 1.[RoarCTF 2019]Easy Calc
```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```
### 2.[极客大挑战 2019]BuyFlag
```php
<?php
if (isset($_POST['password'])) {
	$password = $_POST['password'];
	if (is_numeric($password)) {
		echo "password can't be number</br>";
	}elseif ($password == 404) {
		echo "Password Right!</br>";
	}
}
```
    wp：使用password=404%20可以绕过is_numeric的检测，并且404%20在和404比较时候，由于使用了弱类型，所以404%20会被转成404。
    而如果money不够就返回you have not enough money,loser~
    money够就返回Nember lenth is too long
    所以这个地方也需要进行绕过，估计采用的是strcmp函数所以传入数组直接绕过。
    
### 3.[极客大挑战 2019]Secret File
    wp：找秘密web页面，一个跳转，抓一下就有了
    include使用php://filter协议直接读取flag.php就行了。
### 4.[De1CTF 2019]ShellShellShell
    留坑，知识点巨多
### 5.[安洵杯 2019]easy_web
    通过img包含文件得到源码。不过这个base64decode2次就太坑了- -还要转一次16进制。
    得到源码后发现是md5碰撞，进而得到flag
    知识点：sort可以读取，还有ca\t 也可以读取!
### 6.[RCTF 2019]Nextphp
    php7.4新型序列号漏洞