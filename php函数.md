# php函数
### 1.chdir()
改变目录 如：
```php
<?php
echo getcwd(); // test
chdir('image');
echo getcwd(); // test/image
```
### 2.pathinfo()
以数组形式返回文件信息 如：
```php
<?php
print_r(pathinfo('/Users/swagger/test.php' [, 这里有一个可选参数，是返回数组中的参数例如dirname])); 
```
    返回：Array(
        [dirname] => /Users/swagger
        ['basename'] => test.php
        ['extension'] => txt
    )
### 3.basename()
显示文件名 如：
```php
<?php
$path = "/Users/swagger/test.php";
echo basename($path);  //输出 test.php
echo basename($path, '.php'); // 输出test
```