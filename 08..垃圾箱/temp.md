logincookie
<?php
header('Content-type: text/html; charset=utf-8');
if(($_POST['username']!=null) && ($_POST['password']!=null)){
    $userName=$_POST['username'];
    $password=$_POST['password'];
$conn=mysqli_connect('localhost','admin','admin123');
    if ($conn === false) {
     die("ERROR: Could not connect. " . mysqli_connect_error());
}

mysqli_select_db($conn,'login');
$sql="select * from user where username = '$userName'";
$res=mysqli_query($conn,$sql);
$row=mysqli_fetch_array($res);

if($row['username']!=$userName){
    echo '登陆不能';
    header('Location:fail.php');
}
else if($row['username']!=$userName||$row['password']!=$password) {
    echo '不能登陆';
    header('Location:fail.php');
}
elseif($row['username']==$userName&&$row['password'] ==$password) {    
    //如果密码验证通过，设置一个cookies，把用户名保存在客户端
    setcookie('username',$userName,time()+3600);//设置一个小时
    //最后跳转到登录后的欢迎页面
    echo '登陆成功';
    header('Location:welcome.php');//跳转到最后的欢迎页面
}


}
else {
	echo '登陆失败';
	header('Location:fail.php');//跳转到失败页面
}