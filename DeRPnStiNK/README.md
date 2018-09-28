### 环境搭建
下载虚拟机包文件VMware或VirtualBox打开，配置网络模式为NAT模式。  
>下载链接:[https://download.vulnhub.com/derpnstink/VulnHub2018_DeRPnStiNK.ova ](https://download.vulnhub.com/derpnstink/VulnHub2018_DeRPnStiNK.ova)
hash:949e2f8a7d63fabdc55c675c95efe022    
### 开搞
#### flag1
首先```nmap -sP 192.168.14.1/24```一波找一下靶机地址
![nmap -sP 192.168.14.1/24](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_1.png)    
访问```http://192.168.14.148```查看源码
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_2.png)
112行处找到flag1:```flag1(52E37291AEDF6A46D7D0BB8A6312F4F9F1AA4975C248C3F0E008CBA09D6E9166)```
#### flag2
进行目录扫描
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_3.png)    
访问```/weblog/```目录url跳转至无法访问页面```http://derpnstink.local/weblog/```    
在首页源码中发现的文件```/webnotes/info.txt```内容如下
><-- @stinky, make sure to update your hosts file with local dns so the new derpnstink blog can be reached before it goes live -->    

将```192.168.14.148  derpnstink.local```写入hosts文件后再次访问```http://derpnstink.local/weblog/```  
该目录为一个WordPress博客地址，后台地址为```http://derpnstink.local/weblog/wp-admin/```
尝试账号密码```admin/admin```成功进入后台
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_4.png)   
利用wpscan扫描WordPress ```wpscan http://derpnstink.local/weblog/```
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_5.png)    
下载poc并使用```python 34514.py -t http://derpnstink.local/weblog/ -u admin -p admin -f [your_shell].php```
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_6.png)    
利用shell读取```wp-config.php```获取到数据库账户密码```root/mysql```
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_7.png)    
登录phpmyadmin在```wp-posts```表中拿到flag2:```flag2(a7d355b26bda6bf1196ccffead0b2cf2b81f0a9de5b4876b44407f1dc07e51e6)``` 
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_8.png)    
#### flag3
wp-users表中得到hash值```$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41```利用john破解得到```wedgie57```   
利用webshell读取```/etc/password```获得用户名:```stinky```   
测试发现该用户无法使用ssh用户名口令方式登录，登录ftp服务器成功   
在```ftp://192.168.14.148/files/network-logs/derpissues.txt```中发现类似聊天记录```(WTF?谁家聊天记录这么保存的...)```文件  
>12:06 mrderp: hey i cant login to wordpress anymore. Can you look into it?   
12:07 stinky: yeah. did you need a password reset?   
12:07 mrderp: I think i accidently deleted my account   
12:07 mrderp: i just need to logon once to make a change   
12:07 stinky: im gonna packet capture so we can figure out whats going on   
12:07 mrderp: that seems a bit overkill, but wtv   
12:08 stinky: commence the sniffer!!!!   
12:08 mrderp: -_-   
12:10 stinky: fine derp, i think i fixed it for you though. cany you try to login?   
12:11 mrderp: awesome it works!   
12:12 stinky: we really are the best sysadmins #team
12:13 mrderp: i guess we are...   
12:15 mrderp: alright I made the changes, feel free to decomission my account   
12:20 stinky: done! yay

```/files/ssh/ssh/ssh/ssh/ssh/ssh/ssh/key.txt``` 中拿到ssh登录key文件，登录方式为:```ssh -i key.txt stinky@192.168.14.148``` 
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_9.png)   
Desktop目录下获得flag3:```flag3(07f62b021771d3cf67e2e1faf18769cc5e5c119ad7d4d1847a11e11d6d5a7ecb)``` 
#### flag4
在Documents目录下得到一个数据包
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_10.png)   
应该就是聊天记录中的那个数据，分析这个数据包在某POST请求中获得密码```derpderpderpderpderpderpderp```
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_11.png)    
利用这个密码su切换用户mrderp，在Desktop目录下拿到一个邮件来往记录，记录中提到sudo配置文件```sudoers``` ,执行sudo
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_12.png)
```(ALL) /home/mrderp/binaries/derpy*``` 即mrderp用户只能在binaries目录下执行derpy*文件时使用sudo，创建binaries目录，在derpy.sh中写入```/bin/bash```,sudo执行。
![image](https://raw.githubusercontent.com/trialv/vulnhub_test/master/DeRPnStiNK/png_13.png)   
在```/root/Desktop/``` 下拿到flag4:```flag4(49dca65f362fee401292ed7ada96f96295eab1e589c52e4e66bf4aedda715fdd)```
