

# Kali安装

## 下载并安装OWASP Broken Web Apps

官方网站速度太慢，亲测迅雷链接http://sourceforge.mirrorservice.org/o/ow/owaspbwa/1.2/OWASP_Broken_Web_Apps_VM_1.2.7z 

OWASP IP地址是192.168.254.131

## 安装ibus拼音输入法

安装拼音输入法
```
apt-get install ibus ibus-pinyin
#打开ibus Preferences
ibus-setup
```
添加拼音输入法：ibus Preferences->Input Method->Add->Chinese->Pinyin

设置输入法切换快捷键：ibus Preferences->General->Keyboard Shortcuts

## 安装gnome桌面

1.  gnome桌面安装

    `sudo apt-get install gnome-core kali-defaults kali-root-login desktop-base`

2.  GNOME 3 安装

    `sudo apt-get install gnome-session-flashback`

3.  GNOME Shell 扩展

    `sudo apt-get install gnome-tweak-tool`

4.  需要重启切换gnome桌面，然后去https://extensions.gnome.org下载插件

5. 然后输入gnome-tweaks可以各种桌面设置

## 安装Chrome
1.  下载安装包
    `wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb`
2.  安装
    `dpkg -i google-chrome-stable_current_amd64.deb`
3.  可能会出现依赖问题，执行
    `apt-get -f install`
4.  运行依赖包安装命令
    `apt-get install google-chrome-stable`
    再运行第二个安装命令
5.  此时在终端启动`google-chrome`，提示root下无法运行
6.  编辑/opt/google/chrome/google-chrome文件
7.  找到`exec -a "$0" "$HERE/chrome" "$@"`，修改为`exec -a "$0" "$HERE/chrome" "$@" --user-data-dir --no-sandbox`
8.  再次运行google-chrome即可

## 设置虚拟机共享文件夹
1.	虚拟机 -> 设置 -> 选项 -> 共享文件夹 ->在主机设置共享文件集
2.	新建mount-shared-folders文件`sudo vim /usr/local/sbin/mount-shared-folders`
3.	输入如下内容
```
	#! /bin/bash
	vmware-hgfsclient | while read folder; do
    vmwpath="/mnt/hgfs/${folder}"
    echo "[i] Mounting ${folder}    (${vmwpath})" 
    mkdir -p "${vmwpath}"
    umount -f "${vmwpath}" 2>/dev/null
    vmhgfs-fuse -o allow_other -o auto_unmount ".host:/${folder}" "${vmwpath}"
	done
	sleep 2s
```

4.	获取执行权限 `sudo chmod +x /usr/local/sbin/mount-shared-folders`

5.	执行脚本 `sudo sh /usr/local/sbin/mount-shared-folders`
6.	若虚拟机中看不到主机中的共享文件时，重新执行第5条指令

## Kali中使用Typora
Typora是一款及其优雅的markdown编辑器，比retext强多了。
### 在线安装
Typora官网提供了Linux在线安装教程，十分复杂，更难过的是，我自己在线安装失败，幸运的是，Typora官网提供了Linux二进制软件包，可以离线使用。
### 离线使用
1.	使用迅雷下载，浏览器下载是龟速
2.	解压出来，随便放在某个目录下，可以看到文件夹里的Typora文件，将该文件所在的目录添加到环境变量PATH中。用vim在文件/etc/profile文件中增加变量，该变量将会对Linux下所有用户有效，并且是“永久的”。要让刚才的修改马上生效，需要执行以下代码 `source /etc/profile`
3.	由于Typora在root权限下无法直接运行，需加上参数--no-sandbox，`Typora --no-sandbox`

## 新版Kali没有owasp-zap工具
1.	运行`apt-get update` 更新本地可用于升级本地软件的软件源列表
2.	运行`apt-get upgrade` 更新本地全部软件包
3.	运行`apt-get install zaproxy`下载安装owasp-zap，打开工具栏就能看到了

## 安装Burp Suite Professional
1.	下载Burp Suite Professional，下载链接 https://pan.baidu.com/s/1adX8-aTXVLltwdxGMINayw 提取码mhui，我放在了Downloads/目录下
2.	解压 `unzip Burp_Suite_Pro_v1.7.37_Loader_Keygen.zip`，出现burp-loader-keygen.jar，burpsuite_pro_v1.7.37.jar 
3.	查看自己java版本 `java -version`
```bash
root@kali:~/Downloads# java -version
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-8u212-b01-1-b01)
OpenJDK 64-Bit Server VM (build 25.212-b01, mixed mode)
```
4.	启动注册机`java -jar burp-loader-keygen.jar`，注册机启动起来，将注册机窗口向拉伸，可以看到按钮`RUN`，点击该按钮应该会启动Burp Suite Professional，若无反应，需降低java版本。
5.	降低java版本，输入2
```bash
update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
* 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```
6.	重新执行第4条命令，启动Burp Suite Professional，点击accept，复制注册机中的license到Burp Suite Professional中，点击next-> manual activation->copy requests
7.	复制到注册机activation request中，注册机activation response出现了内容，复制到Burp Suite Professional `paste response`中，再点击next，可以看到注册成功。
8.	以后使用Burp Suite Professional的话，执行`java -jar burp-loader-keygen.jar`启动注册机，点击`run`即可启动Burp Suite Professional。

# Kali侦察

## 使用nmap扫描识别服务
1. 查看服务器是否响应Ping

	```
   nmap -sn 192.168.254.131
	```

2. 查看开放的端口

   ```
   nmap 192.168.254.131
   ```

   调用nmap最简单的方式，先Ping目标服务器，如果响应了，nmap会向1000个TCP端口列表发送探针，观察端口是否响应并报告结果。

3. 查询服务器运行服务的版本

   ```  
   nmap -sV -O 192.168.254.131
   ```
   向第二个命令添加如下两个任务：

   - `-sV` 请求每个被发现的开放端口的标识，用来确定服务的版本
   - `-O` 让namp尝试猜测操作系统的版本

4. 其他实用参数

   - `-sT` 使用SYN扫描类型，即完全连接的扫描，速度更慢，并会在服务器日志中留下记录。
   - `-Pn` 告诉nmap跳过ping测试扫描指定目标
   - `-v` 开启详细模式
   - `-p n1,n2,...` 告诉nmap测试指定的端口
   - `--script=script_name` 与参数`-p`结合使用，告诉nmap在目标的开放端口上运行的脚本，用来进行漏洞检测、扫描、识别等等。

## 识别Web应用防火墙
1.  使用脚本测试防火墙

    ```
    nmap -p 80,443 --script=http-waf-detect 192.168.254.131
    ```

2.  另一个namp脚本，可以帮助我们更加精确地识别防火墙设备

    ```
    nmap -p 80,443 --script=http-waf-fingerprint 192.168.254.131
    ```

3.  使用Kali自带的wafw00f

    ```
    wafw00f 192.168.254.131
    ```

## 查看源代码
1.  教程中使用firebug，现在已不再支持

2.  firefox-> F12->Inspector->修改页面元素，script里的内容修改后无反应

## 获取修改Cookie
1.  教程中使用OWASP-Mantra 浏览器以及它的工具cookies manager +

2.  作为替代，在firefox中添加cookie quick manager

3.  在搜索框中输入192.168.254.131，可以看到3条内容

4.  选择PHPSESSID，勾选httponly，告诉浏览器，该cookie值不能被脚本访问

## 利用robots.txt查看站点的隐藏链接
1.  浏览192.168.254.131/vicnum/

2.  向URL后添加robots.txt，显示如下内容

    User-agent: *
    
    Disallow: /jotto/
    
    Disallow: /cgi-bin/

    这个文件告诉搜索引擎，jotto和cgi-bin的首页不允许被任何搜索引擎（User Agent）收录

## 使用dirbuster发现站点的文件和文件夹
1.  访问Application | Web Applications | Web Crawlers | dirbuster

2.  设置站点url为192.168.254.131

3.  选择list based brute force(基于列表爆破)，并选择一个字典文件

4.  取消Be recursive (递归)

5.  开始

6.  结果中的响应代码：
    - 200 文件存在并能够读取
    - 301 到给定url的重定向
    - 401 需要权限来访问该文件
    - 403 请求有效但服务器拒绝
    - 404 文件不存在

## 使用cewl分析站点使用的单词列表，可以用来尝试口令破解

## 使用John the Ripper生成字典



# 爬虫和蜘蛛

作为每个 Web  渗透测试中侦查阶段的一部分，我们需要浏览器每个包含在网页中的链接，并跟踪它展示的每个文件。有一些工具能够帮助我们自动和以及加速完成这个任务，它们叫做 Web  爬虫或蜘蛛。这些工具通过跟随所有到外部文件的链接和引用，有的时候会填充表单并将它们发送到服务器，保存所有请求和响应来浏览网页，从而提供给我们离线分析它们的机会。

## 一、 使用Wget为离线分析

1. 语法 `wget [OPTION] ... [URL]...`

   + -r` 递归下载每个页面的所有链接

   - `-P` 允许设置保存下载内容的位置，默认是当前目录
   - `-l` 递归下载时，规定wget遍历的深度
   - `-k` 文件下载后，wget修改所有链接，使其指向本地文件，使得可以在本地浏览这些页面
   - `-p` 使得wget下载所需的所有图像，即使位于其他站点
   - `-w` 这个选项让 Wget 在两次下载之间等待指定的描述。当服务器中存在防止自动浏览的机制时，这会非常有用

2. 步骤

   ```bash
   mkdir bodgeit_offline
   wget -r -P bodgeit_offline/ http://192.168.2.167/bodgeit/
   ```

## 二、使用HTTrack为离线分析下载页面
它允许你从互联网下载 WWW 站点到本地目录中，递归构建所有目录、从服务器获得 HTML、图像，和其它文件到你的计算机中。

### 准备

在kali中安装HTTrack

```bash
apt-get update
apt-get install httrack
```

### 操作步骤

1. 创建目录存储下载的站点

   ```bash
   mkdir bodgeit_httrack
   cd bodgeit_httrack
   ```

2. 下载站点

   `httrack http://192.168.2.167/bodgeit/`

   需要注意的是最后的`/`非常重要，缺少的话，HTTrack返回404错误。

### 工作原理

HTTrack 创建站点的完整静态副本，这意味着所有动态内容，例如用户输入的响应，都不会有效。在我们下载站点的文件夹中，我们可以看到下列文件和目录：

+   以服务器名称或地址命名的目录，包含所有下载的文件。
+   `cookies.txt`文件，包含用于下载站点的 cookie 信息。
+   `hts-cache`目录包含由爬虫检测到的文件列表，这是 httrack 所处理的文件列表。
+   `hts-log.txt `文件包含错误、警告和其它在爬取或下载站点期间的信息
+   ` index.html`文件重定向到副本的原始主页，它位于名称为服务器的目录中。
+   `-rN` 将爬取链接的深度设置为N
+   `-%eN` 设置外部链接的深度

## 三、使用ZAP蜘蛛

wget、httrack只是下载了站点完整的静态副本，但并没有服务器的请求与响应，zap蜘蛛可以获得上述信息。

### 步骤

1. 启动zap，设置浏览器代理，浏览http://192.168.2.167/bodgeit/
2. 在sites标签页，打开http://192.168.2.167/文件夹，右键GET：bodgeit
3. 选择attack -> spider，保留默认选项，点击start scan
4. 结果出现在底部面板，可以查看每一个页面的请求和响应

### 原理

ZAP 的蜘蛛跟随它找到的每个链接,获取每个页面的请求和响应。此外,蜘蛛会跟随表单响应、重定向和包含在 robots.txt 和 sitemap.xml 文件中的URL。

## 四、Burp Suite爬取站点

Burp是应用最广泛的渗透测试的工具，Kali默认安装社区版本，目前Burp Suite Community 2.0移除了spider功能，Burp Suite Professional的安装教程见上文所述。

# 第四章 漏洞发现

## 4.1 使用hackbar简化参数分析

在测试 Web 应用时，我们需要和浏览器的地址栏交互，添加或修改参数，以及修改 URL。一些服务器的相应会包含重定向，刷新以及参数修改。所有这些改动都会使对相同变量尝试不同值的操作非常费时间。Hackbar可以简化上述操作。

### 准备

最新版firefox安装的hackbar是收费版，网上有安装免费版的教程。

### 步骤

1. 访问DVWA并登录，默认用户名/密码是admin/admin

2. 选择SQL Injection，在User ID框中输入不同数值，可以看到服务器的不同响应

3. 使用hackbar简化上述工作，f12调出hackbar（不同版本调出的方式可能不同）

   ![](./pictures/f12_hackbar.png)
   
4.	点击loadurl导入url框中的地址，修改参数后，点击execute，可以看到服务器的响应。在测试很多输入的表单或者服务器会重定向其他页面时，非常有用。

### 原理

Hackbar 是带有一些实用特性的第二个地址栏，比如不受 URL 重定向影响，并且允许我们修改 POST参数。此外，Hackbar 可用于向我们的请求中添加 SQL 注入或跨站脚本代码段,以及哈希、加密和编码我们的输入。

## 4.2  使用Tamper Data插件拦截修改请求

有时候,应用拥有客户端的输入校验机制，它们通过 JavaScript,隐藏表单或者 POST 数据，并不能直接在地址栏中了解或看到。为了测试这些以及其它类型的变量，我们需要拦截浏览器发送的请求并且在它们到达服务器之前修改它们。Tamper Data可以完成上述工作。

### 步骤

1. 打开浏览器Mantra，打开`Tools | application auditing | tamper data`，[owaspbwa][owaspbwa]/dvwa，可以看到tamper data中出现了我们请求的会话
![tamper data](./pictures/tamper_data.png)
2.	为了拦截请求，点击`start tamper`，在浏览器中随机输入用户名/密码，点击登录
3. 在确认框中取消勾选`ocntinue tampering` 并点击`tamper`，会出现`tamper popup`窗口，可以修改请求头、POST参数，再点击OK，`tamper data`将修改后的请求包发送给服务器。

### 原理
Tamper Data 会在请求离开浏览器之前捕获请求，并提供给我们时间来修改它包含的任何变量。但是，它也有一些限制,例如不能编辑 URL 或 GET 参数。

## 4.3 使用ZAP查看和修改请求

相比于Tamper Data，ZAP不仅可以修改参数，也可以修改方法（GET改为POST）

### 准备
启动ZAP并配置浏览器代理

### 步骤
1.	访问[owaspbwa][]/mutillidae，并点击 `OWASP 2013 | A1 Injection(SQL) | SQLi-Extract Data | User Info(SQL)`
2.	输入用户名/密码时，浏览器会检测是否存在非法字符
3.	使用ZAP拦截请求（点击`break on all requests`，菜单栏中绿色圆球），在浏览器中输入用户名/密码
4.	回到ZAP中，输入合法的用户名/口令，提交后，在ZAP | Break面板中可以看到拦截的包，修改url中的`username` 和 `password`的值，点击`unbreak on all requests`和播放按钮，可以在ZAP | History 面板中看到刚发出去的包，同时浏览器也可以看到服务器的响应。

### 原理
ZAP可以拦截所有的request包，并允许我们对其进行修改

## 4.4 使用Burp Suite查看和修改请求

使用Burp拦截和修改请求

### 步骤
1.	配置浏览器代理，并运行`java -jar burp-loader-keygen.jar`，打开burp professional
2.	浏览[owaspbwa][]/mutillidae，并点击 `OWASP 2013 | A1 Injection(SQL) | SQLi-Extract Data | User Info(SQL)`，保持 burp | proxy | intercept is off（拦截数据包）
3.	填写合法用户名/口令，并提交，burp会拦截该数据包，修改数据包后点击 forward 或 intercept is on，重新发送数据包。可以在浏览器中看到服务器的响应。

### 更多
可以使用burp repeator来测试不同的输入值，在brup | render 中可以看到服务器的响应。

[owaspbwa]: http://192.168.2.167

## 4.5 识别跨站脚本（XSS）漏洞

本节学习反射型XSS，简单地说就是用户的输入会被反射到响应中，成为响应中HTML的一部分，如果客户端没有检查用户输入且服务端没有对用户输入进行合理编码，则可以输入用于HTML代码中的字符，从何被浏览器解释为源代码。

### 步骤

1. 登录[owaspbwa][]/DVWA并访问反射型XSS
2. 在输入框中输入任何字符，都会成为响应HTML源代码的一部分
3. 输入一个名称，后面带上简单的脚本代码`Bob<script>alert('XSS')</script>`，页面会执行脚本并弹框。

### 原理

为了发现 XSS 漏洞,我们需要遵循以下原则：

+ 我们在输入框中输入的，准确来说是被发送的文本，用于形成在页面中展示的信息，这是反射型漏
  洞。
+ 特殊的字符没有编码或转义。
+ 源代码表明，我们的输入被集成到某个位置，变成了 HTML 代码的一部分，并且会被浏览
  器解释。

### 更多

除了反射型XSS，还有一种存储型XSS。存储型XSS在提交后浏览器不一定会有反应，但输入会存储在服务器中，会在用户每次访问数据时执行。

## 4.6 基于错误的SQL注入识别

攻击者向表单输入或请求中的其它参数注入 SQL 命令，应用发送修改后的请求，使服务器执行恶意的SQL 语句。

### 步骤

1. 登录[owaspbwa][]/DVWA，访问SQL Injection
2. 测试输入`1`、`1'`、`1''`，观察不同的响应，可以使用hackbar来测试。
3. 执行SQL攻击 `' or  '1'='1`，可以获取所有数据库中的注册用户。

### 原理

根据不同的测试结果的响应，特别是错误信息，可以猜测服务端代码拼装了一个请求`"SELECT * FROM users WHERE id='input'"`

构造SQL注入代码的关键是引号闭合。

## 4.7 识别SQL盲注

SQL盲注：浏览器不能显示任何可以引导我们的错误信息或提示，只能告诉我们是与否，通过不断的提问，获得数据库的信息。

### 步骤

1. 登录[owaspbwa][]/DVWA，访问SQL Injection(Blind)

2. 输入`1`、`1'`、`1''`，服务器只响应正确的语句，可以知道这里存在SQL盲注入。

3. 猜测数据库名长度 `1' and length(database()) =4 #`

4. 猜测数据库名字

   ```sql
   1' and ascii(substr(database(),1,1))=100#        //d
   1' and ascii(substr(database(),2,1))=118#        //v
   1' and ascii(substr(database(),3,1))=119#        //w
   1' and ascii(substr(database(),4,1))=97#         //a
   ```
### 原理

在基于错误的 SQL 注入中，我们使用由服务器发送的错误来识别查询类型、表和列的名称。当我们视图利用盲注时，我们需要通过问问题来得到信息。

## 4.8 识别Cookie中的漏洞

Cookie 是服务器发送给客户端的小型数据片段，它储存于用户的浏览器中。它们包含有关于特定 Web 应用用户的信息。服务器使用Cookie能够同时识别由不同客户端产生的不同请求。

### 步骤

1. 访问[owspabwa][]/mutillidae，打开cookie manager
2. 访问OWASP 2013 | A2-Broken Authentication and Session Management | Authentication Bypass | via cookie
3. 在cookie manager中可以看到获取的cookie，在detail中，httponly没有开启，意味着该cookie值可以被XSS脚本获取。

## 4.9 使用SSLScan获取SSL和TLS信息

SSLScan，它是 Kali Linux 所包含的工具，通过建立到服务器的安全通信来分析服务器的配置文件(从客户端的角度)。

```
sslscan 192.168.2.167
```

SSLScan 通过创建多个到 HTTPS 的链接来工作,并尝试不同的加密方式和客户端配置来测试它接受什么。

## 4.10 查找文件包含漏洞

文件包含漏洞的产生原因是在通过PHP的函数引入文件时，由于传入的文件名没有经过合理的校验，从而操作了预想之外的文件，就可能导致意外的文件泄露甚至恶意的代码注入。

### 步骤

1. 登录DVWA访问File Inclusion，此时URL为`https://192.168.2.167/dvwa/vulnerabilities/fi/?page=include.php`，将include.php替换为index.php，浏览器显示空白页面，证明这里存在本地文件包含漏洞，只是在当前目录下未找到index.php。
2. 查看网页源码，可以看到很多文件路径，

![fileinclude](/root/Documents/GitHub/My-Notes/pictures/fileinclude.png)

修改网页url=`https://192.168.2.167/dvwa/vulnerabilities/fi/page=../../setup.php`，可以访问setup页面。此外，可以看到很多文件路径最后是`.`，这表示访问该目录下的默认文件，访问<https://192.168.2.167/dvwa/vulnerabilities/fi/?page=../../vulnerabilities/csrf/index.php>即可。

3. 除了本地文件包含漏洞，这里还存在远程文件包含漏洞，访问<https://192.168.2.167/dvwa/vulnerabilities/fi/?page=https://192.168.2.167/mutillidae/>

## 4.11 识别POODLE漏洞

POODLE 攻击是将 TLS 通信降级为 SSLv3 并强制使用易于被攻破的加密算法(CBC)

Nmap使用脚本ssl-poodle可以检测目标服务器是否存在poodle漏洞

### 步骤

1. 下载ssl-poodle.nse，<http://nmap.org/nsedoc/scripts/ssl-poodle.html>
2. 将上述文件复制到/usr/share/nmap/scripts/中
3. 运行`nmap --scripts ssl-poodle -sV -p 443 192.168.2.167`

















