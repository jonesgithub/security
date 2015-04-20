## 访问控制缺陷
主要是一个用户角色能访问该角色不能访问的资源。常见的有基于角色的访问控制，基于路径的访问控制。

### 漏洞分析
0x01 基于角色的访问控制

一般基于角色的访问控制缺陷就是用户权限跨界访问，一般都是逻辑错误。

0x02 基于路径的访问控制

一般Linux，windows路径目录的访问方式。
Linux：
```text
/etc/passwd
~/.ssh
./configure
./../../../etc/passwd
```
Windows：
```text
c:\boot.ini
.\test.txt
..\..\..\boot.ini
file:///c:\bbot.ini
c:\test~1.txt
```
这种漏洞，一般都是目录路径访问未得到严格限制，会造成跨站执行CGI脚本。更严重的CGI执行用户是root的权限，会让用户能查看系统文件的内容。
就像下面的错误代码：
```php
<?php
    //test.php
    $filename = $_GET['file'];
    $fd = fopen($filename,"r");
    $fcontent = fread($fd,filesize($filename));
    fclose($file);
    echo $fcontent;
?>
```
在正常的情况下访问`http://exmaple.com/test.php?file=test.txt`会打开读取当前php的工作目录下`test.txt`文件。
如果访问`http://exmaple.com/test.php?file=/etc/passwd`就会读取到系统的密码文件，如果系统是通过sshkey认证登录的，就可以直接拿到ssh的认证key直接登录服务器了。

### 危害
基于角色的访问控制的缺陷，安全系数高，角色权限分配不当或者整个角色权限系统设计有缺陷，会造成用户拥有的角色权限访问不应该能访问的数据。
基于路径的访问控制的缺陷，安全系数非常高,稍微不慎就会造成整个服务器沦陷。只要能查看系统文件就能做很多事情了。

### 解决方案
基于角色的访问控制的缺陷的解决方案，设计角色权限系统时严格的验证测试，确保权限系统不会跨界访问。
基于路径的访问控制的缺陷的解决方案，需要指定程序的工作目录路径，比如PHP在配置时指定php脚本所在目录为当前根目录，不允许夸目录操作文件。在操作一个文件时使用绝对路径，
尽量避免相对路径，对用户输入的文件名进行路径检测，进行转义。