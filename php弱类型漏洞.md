# php弱类型引发的漏洞


### 1.类型转换
    1.会先进行类型转换，再进行对比。
    
    2.会先比较类型，如果类型不同直接返回false
```php
<?php
$a = null;
$b = 'a';
$c = 0;

var_dump($a == $b); //false
var_dump($a === $b); //false

var_dump($a == $c); //true
var_dump($a === $c); //false

var_dump($b == $c); //true
var_dump($b === $c); //false
?>
```

### 注意：

`1.`当一个字符串被当作一个数值来取值，其结果和类型如下:如果该字符串没有包含`.``e``E`并且其数值值在整形的范围之内，该字符串被当作`int`来取值。其他所有情况下都被作为`float`来取值，该字符串的开始部分决定了它的值，如果该字符串以合法的数值开始，则使用该数值，否则其值为0。
`所有类型的不是以0x和0e开头的字符串在被强制转换类型成int时都会被转成int类型的0。`
`如果字符串是以数字开头的，则转成首数字的int类型`
```php
<?php
var_dump('admin' == 0); //true
var_dump('1admin' == 1); //true
var_dump('admin1' == 1); //false
var_dump('admin1' == 0); //true
?>
```
`2.`在进行比较运算时，如果遇到了`0e`这类似的字符串，php会将它解析为`科学计数法`。
```php
<?php
var_dump('0e1234' == '0e5678'); //true 待实测
?>
```
`3.`在进行比较运算时，如果遇到了`0x`这类字符串,php会将它解析为`科学计数法`。
```php
<?php
var_dump('0x1046a' == '66666'); //true php7.3版本实测为false
?>
```

### 2.is_number()
`1.`is_numeric在判断时候，如果攻击者把payload改成16进制`0x....`，`is_numeric`会先对十六进制做`类型判断`，十六进制被判断为数字类型为真，就进入了条件语句，如果再把这个代入`sql`语句进入`mysql`数据库，`mysql`数据库会对`hex`进行`解析`成字符串存入到数据库中，如果这个字段再被取出来二次利用，就可能造成二次注入漏洞。

### 3.md5()
```php
string md5(string $str [, bool $raw_output = false ]);
```
`1.`md5()需要是一个string类型的参数。但是当你传递一个`array`时，md5()不会报错，但是会无法正确地求出`array`的`md5值`，返回null，这样就会导致任意2个array的md5值都会想等。
```php
<?php
$a = [1, 2, 3];
$b = ['a', 'b', 'c';
var_dump(md5($a)); //NULL
var_dump(md5($a) == md5($b)); //true

$c = 240610708;
$d = 'QNKCDZO';
echo $c; //0e462.....全部为数字
echo $d; //0e830.....全部为数字
//如果0e后跟上中存在字符串则弱类型==判断不相等
//如：0e123456a == 0e123456a 为false
var_dump(md5($c) == md5($d));//true
?>
```

### 4.strcmp()
`1.`strcmp(string1, string2)用于比较括号内两个字符串string1和string2，当他们相等时候，返回0；string1的大于string2时，返回>0，小于时返回<0。在5.3及以后的php版本中，当strcmp()括号内是一个数组与字符串比较时，也会返回0。 `实测php7.3输入数组报错`
##### 注意：
```php
var_dump(false == 0); //true
```

### 5.in_array()
`1.`in_array(search,array,type): 如果给定的值 search 存在于数组 array 中则返回 true（类似于 == ）。如果第三个参数设置为 true，函数只有在元素存在于数组中且数据类型与给定值相同时才返回 true（类似于 === ）。如果没有在数组中找到参数，函数返回 false。
```php
<?php
$a = "1 and 1=1";
var_dump(in_array($a, array(0, 1, 2, 3))); //true
var_dump(in_array($a, array(0, 1, 2, 3), true)); //false
//在没有给定true时候，比较结果和 == 相同
?>
```

### 6.switch()
`1.`如果switch是数字类型的case的判断时，switch会将其中的参数转换为int类型。
```php
<?php
$string = '2iphone';
switch($string){
    case 1:
        echo 'this is 1';
        break;
    case 2:
        echo "this is 2";  //输出点！！！
        break;
    case "2iphone":
        echo "this is iphone";
        break;
}
?>
```


