# DVWA 漏洞

## 一、文件上传漏洞

文件上传漏洞可以说是危害很大了，因为可以直接通过此漏洞getshell。想要成功利用文件上传漏洞至少需要满足三个条件：

+ 有效的上传点
+ 上传文件能够被访问到
+ 上传文件能够被解析执行

### 1.1 Low Level Security

没有任何限制，可以直接上传.php文件

### 1.2 Medium Level Security

只能上传jpg或png格式文件，限制文件类型的方式有：

+ 前端：利用js判断
+ 后端：
  - 判断文件名后缀
  - 判断MIME类型
  - 判断文件内容

确定限制文件类型的机制：

+ 通过禁用浏览器js，判断是否是在前端利用js限制文件类型
+ 利用burp拦截数据包，修改content-type: image/jpeg，再次尝试上传，可以成功



### 1.3 High Level Security



## 二、OS命令注入漏洞

DVWA提供了Command Execution功能，可以Ping，采用名录连接符进行命令注入。

### 2.1 Low Level Security

输入完全无限制，直接坠在ping命令后，可以通过命令连接符执行注入命令

```php
    command1 && command2   逻辑与，command1执行成功才会执行command2
    command1 | command2    管道，command1的输出作为command2的输入
    command1 ; command2   先执行command1后执行command2
    以上三种连接符在windows和linux环境下都支持。
```

### 2.2 Medium Level Security

`&&`、`;`无法成功注入，管道`|`仍然可以