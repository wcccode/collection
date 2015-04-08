#apache

##apache 作为文件服务器
1.配置 Alias  /webpath  D:/filepath/
2.配置访问权限
<Directory />
	  Options Indexes FollowSymLinks Includes ExecCGI
    AllowOverride All
    Order deny,allow
    Allow from all
</Directory>
3.文件名称包含%或者#会找不到文件，解决方法是把url进行encode
如果需要在URL中用到，需要将这些特殊字符换成相应的十六进制的值   
+   %20   
/   %2F   
?   %3F   
%   %25   
#   %23   
&   %26
