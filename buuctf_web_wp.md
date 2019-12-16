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
### 7.[GWCTF 2019]我有一个数据库
    phpmyadmin4.8.1远程文件包含漏洞（CVE-2018-12613）
    漏洞payload：http://yourip:port/phpmyadmin/index.php?target=db_sql.php%253f<你的文件路径>
### 8.[RoarCTF 2019]Simple Upload
    条件竞争漏洞，有待分析
### 9.[GWCTF 2019]枯燥的抽奖
    php的伪随机数漏洞
网页源代码：
```php
header("Content-Type: text/html;charset=utf-8");
session_start();
if(!isset($_SESSION['seed'])){
$_SESSION['seed']=rand(0,999999999);
}

mt_srand($_SESSION['seed']);
$str_long1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
$str_show = substr($str, 0, 10);
echo "<p id='p1'>".$str_show."</p>";
```
得到Gn0qGVf5hw为随机生成的前10位，后10位不可知道
可以使用php_mt_seed爆破，但是前提得让php_mt_seed识别这种类型的随机数，格式为数 + 数 + 最小值 + 最大值
```python
str1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
choose = "Gn0qGVf5hw"
flag = ""
for i in range(10):
    for j in range(len(str1)):
        if choose[i] == str1[j]:
            flag += str(j) + ' ' + str(j) + ' ' + '0' + ' ' + str(len(str1) - 1) + ' '
            break
print flag //42 42 0 61 13 13 0 61 26 26 0 61 16 16 0 61 42 42 0 61 57 57 0 61 5 5 0 61 31 31 0 61 7 7 0 61 22 22 0 61
```
    运行./php_mt_seed 42 42 0 61 13 13 0 61 26 26 0 61 16 16 0 61 42 42 0 61 57 57 0 61 5 5 0 61 31 31 0 61 7 7 0 61 22 22 0 61
    得到338669776
    还原20位的随机数得到flag
