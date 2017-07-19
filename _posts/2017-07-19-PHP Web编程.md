---
title: PHP Web编程
date: 2017-07-19
categories:
- PHP
---
>PHP是开发Web应用的首选语言之一，
本文将介绍：
1. 用PHP处理HTML表单中的数据
2. 验证表单数据
3. 文件上传处理步骤和方法
4. 用PHP设计一个简单的网站

### PHP处理HTML表单中的数据
--------------
#### 访问获取HTML表单数据：
两个PHP预定义变量：$_GET 和 $_POST。

- $_GET：由HTTP的GET方法传递的表单数据组成。
表单元素的名称就是数组的“索引”。
例如：某个表单中，有一个文本输出框，叫“user_name”。
在PHP中，可以通过$_GET['user_name']获取文本框中用户输入的值。
- $_POST：由HTTP的POST方法获取的表单数据。
其他一样。

```HTML
//HTML:
<html>
<head>
<title>HTML</title>
</head>

<body>
<form name="form1" method="POST" action="1.php">
输入姓名:<input name="user_name" type="text">
<br/><br/>
选择性别:<input name="gender" type="radio" value="male">男
<input name="gender" type="radio" value="female">女
<br/><br/>
兴趣与爱好:
<input name="hobby[]" type="checkbox" value="reading">阅读
<input name="hobby[]" type="checkbox" value="travel">旅游
<input name="hobby[]" type="checkbox" value="sport">运动
<input name="hobby[]" type="checkbox" value="internet">上网
<br/><br/>
选择职业:
<select name="occup">
<option value="engineer">工程师</option>
<option value="teacher">教师</option>
<option value="doctor">医生</option>
<option value="other">其他</option>
</select>
<br/><br/>
<input type="submit" value="提交数据">
</form>

</body>
</html>

//1.php:获取表单数据
<?php
//通过$_POST全局变量，获取文本框user_name的值，并赋值给变量$user_name
$user_name = $_POST['user_name'];
$gender = $_POST['gender'];
$hobby = $_POST['hobby'][0].",".$_POST['hobby'][1].",".$_POST['hobby'][2].",".$_POST['hobby'][3];
$prof = $_POST['occup'];

echo "用户名:".$user_name."<br/>";
echo "性别:".$gender."<br/>";
echo "爱好:".$hobby."<br/>";
echo "职业:".$prof."<br/>";
?>
```

### 验证表单数据
----------
#### PHP中的session
PHP通过session记录用户的有关信息，以提供用户以此身份向服务端发送请求时，服务器端能够根据session做出正确的判断，区分不同用户的请求。
使用session就是通过注册一些session全局变量，在不同页面的程序中使用这些变量。这样就可以通过session完成用户身份验证、程序状态和页面之间的数据传递等功能。
示例代码：
```example
//2.php
<?php
session_start();      //使用session前必须调用该函数

$_SESSION['user'] = 'MrLiu';
$_SESSION['explain']='MrLiu的session变量';
echo '这个页面已经通过session保存了一些变量';
echo '<br/><a href="2.php">进入2.php</a>查看这些变量值';
?>

//3.php
<?php
session_start();
echo $_SESSION['user']."<br/>";
echo $_SESSION['explain']."<br/>";
echo '<a href="3.php">返回3.php</a>';
?>
```
注：上述传递session变量功能的实现，基于客户端浏览器支持cookie的。cookie是由服务器端产生的并且保存在客户端的一些文件，里面存放了一些用户信息和数据。PHP中的session机制是通过cookie实现的。所以，如果浏览器不支持cookie，那么上述示例程序无法看到预期效果。

### PHP中的文件上传
----------
通常，文件上传使用的是HTTP的POST方式，首先需要定义HTML表单的enctype属性为"multipart/form-data"。
```enctype
<form enctype="multipart/form-data" action="somefile.php" method="POST">
```
上传页面：
```HTMLform
//HTML文件：
<html>
<head>
<title>支持文件上传的HTML表单</title>
</head>
<body>
<form enctype="multipart/form-data" action="4.php" method="POST">
上传此文件：<input name="myfile" type="file" />
<input type="submit" value="提交上传" />
</form>
</body>
</html>
```
在PHP程序中，使用全局变量$_FILES处理文件上传。$_FILES是一个数组，包含了要上传的文件信息。
文件会被默认上传到默认路径，如果需要上传到指定路径:
```func
move_uoloaded_file (filename,destination);
```
处理文件的php文件：
```PHP
//4.php
<?php
$upload_path = $_SERVER['DOCUMENT_ROOT']."/upload/";
$dest_file = $upload_path.basename($_FILES['myfile']['name']);

if(move_uploaded_file($_FILES['myfile']['tmp_name'],$dest_file)){
          echo "文件已上传至服务器根目录的upload目录下";
}else{
          echo "上传错误".$_FILES['myfile']['error'];
}
?>
```
### 用PHP开发一个简单的网站：
一个简单的数据录入系统。
```HTMLUSER
//simpleTest.html：
<html>
<head>
<title>17-7-18-用户登录</title>
<style>
.tbl{font-size:10px ; width:30% ; text-align:right ; background-color:#abcdef ; }
</style>

<script language="JavaScript">
//用来检查用户输入是否为空的函数
function check_name(){
      //判断表单login中，名为user_name的input元素的值是否为空
      if(login.user_name.value==""){
              alert("请输入用户名！");
              return false;
      }
}
</script>
</head>

<body>
//注意：这里要添加onsubmit属性，通过它，当submit按钮被单击时，JavaScript函数check_name被调用
<form name="login" action="5.php" method="POST" onsubmit="return check_name()">
<table border="0" width="200px" align="center">
        <tr>
               <td class="tbl">用户名：</td>
               <td><input type="text" name="user_name"></td>
        </tr>
        <tr>
                <td><input type="submit" value="登入系统"></td>
        </tr>
</table>
</form>

</body>
</html>


<!-------------------------------------------------------->
//5.php：
<?php
session_start();      //开始session

$user = $_POST['user_name'];    //获取用户名
if(!empty($user)){
  $_SESSION['user'] = $user;
  $welcome = "您好，".$user."!请录入以下信息后提交。<br/>";
}

$gender = $_POST['gender'];
$age = $_POST['age'];
$blood = $_POST['blood_type'];

//如果当前用户提交了数据，则输出这些数据
if(!empty($gender)&&!empty($age)&&!empty($blood)){
  echo "姓名：".$user."<br/>";
  echo "性别：".$gender."<br/>";
  echo "年龄：".$age."<br/>";
  echo "血型：".$blood."<br/>";
}else{
  ?>
  <html>
  <head>
    <title>5.php 用户信息录入</title>
  </head>

  <body>
    <?php
        echo $welcome;    //显示一条欢迎语句
    ?>
    <form name="info" action="" method="POST">
      <table border="0">
        <tr>
          <td>性别：</td>
          <td><input name="gender" type="radio" value="男">男
            <input name="gender" type="radio" value="女">女
          </td>
        </tr>
        <tr>
          <td>年龄：</td>
          <td><input name="age" type="input" size="3"></td>
        </tr>
        <tr>
          <td>血型：</td>
          <td>
            <select name = "blood_type">
              <option value="A">A型</option>
              <option value="B">B型</option>
              <option value="O">O型</option>
              <option value="AB">AB型</option>
              <option value="其他">其他血型</option>
            </select>
          </td>
        </tr>
        <tr><td><input type="submit" value="提交"></td></tr>
      </table>
    </form>
  </body>
  </html>
  <?php
}
?>
```
