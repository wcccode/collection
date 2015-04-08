#apache

##apache 作为文件服务器
1.配置 Alias  /webpath  D:/filepath/<br/>
2.配置访问权限<br/>
<Directory /><br/>
    Options Indexes FollowSymLinks Includes ExecCGI<br/>
    AllowOverride All<br/>
    Order deny,allow<br/>
    Allow from all<br/>
</Directory><br/>
3.文件名称包含%或者#会找不到文件，解决方法是把url进行encode<br/>
如果需要在URL中用到，需要将这些特殊字符换成相应的十六进制的值  <br/> 
+    %20   
/    %2F   
?    %3F   
%    %25   
#    %23   
&    %26

4.url中斜杠和反斜杠问题 http://www.leakon.com/archives/865
