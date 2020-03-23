# Kali安装
## 下载并安装OWASP Broken Web Apps

官方网站速度太慢，亲测迅雷链接http://sourceforge.mirrorservice.org/o/ow/owaspbwa/1.2/OWASP_Broken_Web_Apps_VM_1.2.7z 

OWASP IP地址是192.168.254.131

## 安装ibus拼音输入法

安装拼音输入法
```bash
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

# Kali侦察
## 使用nmap扫描识别服务

1.  查看服务器是否响应Ping

    ```
    nmap -sn 192.168.254.131
    ```
    
2.  查看开放的端口

    ```
    nmap 192.168.254.131
    ```

    调用nmap最简单的方式，先Ping目标服务器，如果响应了，nmap会向1000个TCP端口列表发送探针，观察端口是否响应并报告结果。

3.  查询服务器运行服务的版本

    ```  
    nmap -sV -O 192.168.254.131
    ```

    向第二个命令添加如下两个任务：
    
    - `-sV` 请求每个被发现的开放端口的标识，用来确定服务的版本
    - `-sV` 让namp尝试猜测操作系统的版本

4.  其他实用参数

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

