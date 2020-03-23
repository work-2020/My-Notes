# Linux学习
- `who am i` 查看用户名
- `sudo adduser lilei` 添加用户lilei
- `sudo useradd lilei` 只创建用户lilei，需要用`passwd lilei`设置用户密码
- `su -l lilei` 切换用户lilei
- 用户组： 一组用户的集合，它们共享一些资源和权限，同时拥有私有资源
  + `groups shiyanlou` 查看用户的用户组，返回`shiyanlou:shiyanlou`，冒号前面表示用户，冒号后面表示用户组
  + 创建用户时，若未指定用户组，默认创建一个与用户名相同的用户组
  + `sudo usermod -G sudo lilei` 将用户lilei添加到sudo用户组
- `sudo deluser lilei --remove-home`删除用户
- `ls` 查看文件
  + `-l` 查看文件详细信息
  + `-A` 查看隐藏文件
- `sudo chown shiyanlou iphone6` 修改文件iphone6的所有者为shiyanlou
- 文件权限 drwxr-xr-x：d表示文件类型为目录，前面rwx表示拥有者权限，中间r-x表示用户组权限，后面r-x表示其他用户权限
- `chmod 600 iphone6` 将文件iphone6权限修改为-rw-------

## Linux目录结构及文件基本操作
- `cd ..` 返回上一级目录
- `cd ~` 切换到当前用户的home目录，等价于`cd /home/<用户名>`
- `cd -` 切换到上一次所在目录
- `pwd` 获取当前路径
- 绝对路径 以根目录`/`为起点，以所要到的目录为终点，表现形式`/usr/local/bin`
- 相对路径 以当前目录`.`为起点，以所要到的目录为终点，可以省略`.`
- $\color{red}{touch}$
  
  + 主要作用是更改已有文件的时间戳（如：最近访问时间，最近修改时间），在不加任何参数的情况下，只指定文件名，可以创建一个空白文件（不会覆盖已有同名文件）`touch test`
- $\color{red}{mkdir}$
  + 创建一个空目录，同时也可以指定创建目录的文件权限属性 `mkdir mydir`创建名为mydir的目录
  + 使用$\color{red}{-p}$参数，同时创建父目录（如果父目录不存在） `mkdir father/son/grandson`
- $\color{red}{cp}$
  + 复制一个文件到指定目录 `cp test father/son/grandson`
  + 成功复制文件夹需使用$\color{red}{-r}$参数，表示递归复制
  ```bash
  mkdir family
  cp -r father family
  ```
- $\color{red}{rm}$
  + 删除文件 `rm test`
  + 删除目录 `rm -r family`
- $\color{red}{mv}$
  + 移动文件（剪切） `mv 源文件 目的目录`
  + 重命名文件 `mv 源文件名 新文件名`
- $\color{red}{cat}$
  + 查看文件
  + $\color{red}{-n}$ 显示行号
- $\color{red}{more}$
  
  + 更加专业地阅读文件内容，默认显示第一屏内容，使用$\color{red}{Enter}$键向下滚动一行，使用$\color{red}{Space}$键向下滚动一屏，使用$\color{red}{h}$显示帮助，使用$\color{red}{q}$退出
- $\color{red}{less}$
  + less 的用法比起 more 、tail更加的有弹性。在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 $\color{red}{pageup}$ $\color{red}{pagedown}$ 等按键的功能来往前往后翻看文件，更容易用来查看一个文件的内容 
  + 空格键 滚动一行
  + 回车键 滚动一页
  + g 跳到文件头  
  + G 跳到文件尾部
- $\color{red}{file}$
  
  + 查看文件类型
- $\color{red}{cut}$
  + 语法 `cut [-args] [file]`
  +  cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。如果不指定 File 参数，cut 命令将读取标准输入。必须指定 -b、-c 或 -f 标志之一。
  + -d ：自定义分隔符，默认为制表符
  + -f ：与-d一起使用，指定显示哪个区域
  + -c : 以字符为单位进行分割 
  ```
    # 前五个（包含第五个）
    cut /etc/passwd -c -5
    # 前五个之后的（包含第五个）
    cut /etc/passwd -c 5-
    # 第五个
    cut /etc/passwd -c 5
    # 2到5之间的（包含第五个）
    cut /etc/passwd -c 2-5
  ```
- $\color{red}{grep}$
  + grep命令是很强大的，也是相当常用的一个命令，它结合正则表达式可以实现很复杂却很高效的匹配和查找
  + `grep [命令选项] 用于匹配的表达式 [文件]`
  + `-c` 统计以模式匹配的数目
  + -r 参数表示递归搜索子目录中的文件
  + -n 表示打印匹配项行号
  + -I 表示忽略二进制文件
  + `i` 忽略大小写
  + `-v` 反选，输出不匹配行的内容
  + `-A n` 除了匹配行之外，还列出后面的n行
  + `-B n` 除了匹配行之外，还列出前面的n行
  + `-E` 使用扩展正则表达式 `echo 'zero\nzo\nzoo' | grep -E 'zo{1}'`
- $\color{red}{wc}$
  
  + wc 命令用于统计并输出一个文件中行、单词和字节的数目
- $\color{red}{sort}$
  + 将输入按照一定方式排序，然后再输出,它支持的排序有按字典排序,数字排序，按月份排序，随机排序，反转排序，指定特定字段进行排序等等
  + -n 按照数值进行排序
  + -r 降序排列
  + -k 按照指定的列进行排序
  + -t 指定分隔符，默认是空格
- $\color{red}{uniq}$
  + 过滤或者输出连续重复行
  + -c 在每列旁边显示该行重复出现的次数
  + -d 仅显示重复出现的行列
  + -u 仅显示出现一次的行列
- $\color{red}{tr}$
  + 删除一段文本信息中的某些文字或者对其转换
  + -d 删除指定的字符
  + -s 删除连续重复出现的指定字符
  + `tr '[a-z]' '[A-Z]'`
- $\color{red}{join}$
  + 将两个文件指定栏位同样地行连接起来，类似sql语句中的join操作
  + 语法 `join [option] file1 file2`默认使用空格作为分隔符，根据两份文件第1栏的内容进行连接
  + -t 指定分隔符
  + -i 忽略大小写
  + -1 指定第1份文件用哪一栏内容进行对比
  + -2 指定第2份文件用哪一栏内容进行对比
- $\color{red}{paste}$
  + 直接将多个文件的每一行连接起来，默认使用tab隔开
  + -d 指定分隔符
  + -s 每个文件的内容单独放在1行
  
## 变量
- 环境变量
- 命令的查找路径与顺序
- 搜索文件
  + whereis 搜索命令的路径
  + locate 搜索文件或目录，locate命令要比find -name快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db。新增的文件无法locate，使用updatedb
  + which 使用which确定是否安装了某个软件，因为它只从PATH环境变量指定的目录搜索命令
  + find 不但可以通过文件类型、文件名进行查找而且可以根据文件的属性（如文件的时间戳，文件的权限等）进行搜索， find 的第一个参数是要搜索的地方。find [path] [option] [action]
  
  
## 文件打包与压缩
- $\color{red}{zip}$
  + `zip -r -q -o outputfile.zip input` 将input（可以是文件或文件夹）打包压缩到文件outputfile.zip，-r 参数表示递归打包包含子目录的全部内容，-q 参数表示为安静模式，即不向屏幕输出信息，-o，表示输出文件，需在其后紧跟打包输出文件名
  + 设置压缩级别<br>
    `zip -r -9 -q -o outputfile.zip input` 9 表示体积最小但耗时最久<br>
    `zip -r -1 -q -o outputfile.zip input` 1表示最快压缩但体积大
  + -x 排除待压缩文件夹中的某些文件
    `zip -r -9 -q -o shiyanlou_9.zip /home/shiyanlou/Desktop -x ~/*.zip`
  + -e 创建加密压缩文件<br>
    `zip -r -e -o outputfile.zip input`
  + -l 将LF转变为CR LF。在 Windows 为 CR+LF（Carriage-Return+Line-Feed：回车加换行），而在 Linux/Unix 上为 LF（换行），所以如果在不加处理的情况下，在 Linux 上编辑的文本，在 Windows 系统上打开可能看起来是没有换行的。<br>
    `zip -r -l -o outputfile.zip input`
  
- $\color{red}{unzip}$
  + `unzip outputfile.zip`
  + `unzip -q outputfile.zip -d dir` 使用安静模式，将文件解压到指定目录
  + -l 显示压缩文件中的文件列表
  
- $\color{red}{tar}$
  + `tar -P -cf output.tar input`-P 保留绝对路径符，-c 表示创建一个 tar 包文件，-f 用于指定创建的文件名，注意文件名必须紧跟在 -f 参数之后
  + `tar -xf output.tar -C dir` 解包一个文件（-x 参数）到指定路径的已存在目录（-C 参数）
  + `tar -tf output.tar`只查看不解包文件 -t 参数
  + `tar -czf output.tar.gz input` 使用 gzip 来压缩文件
  + `tar -xzf output.tar.gz -C dir` 使用 gzip 解压文件到dir文件夹中
  
## 文件系统操作与磁盘管理
- $\color{red}{df}$
  + `df` 查看磁盘容量
  + `df -h` 以更易读的方式显示磁盘容量
- $\color{red}{du}$
  + `du` 查看当前目录（及子目录）容量，或者在后面指定待查看的目录
  + `du -h` 以更易读的方式显示当前目录容量
  + `du -d 1` 指定查看目录的深度
  + `du -a` 同时显示目录中文件的大小
  + `du -s` 仅显示目录的总大小，不显示其子目录的信息
  
## Linux帮助命令
- 内建命令 实际上是 shell 程序的一部分，其中包含的是一些比较简单的 Linux 系统命令，这些命令是写在bash源码的builtins里面的，由 shell 程序识别并在 shell 程序内部完成运行，通常在 Linux 系统加载运行时 shell 就被加载并驻留在系统内存中。
- 外部命令 Linux 系统中的实用程序部分，因为实用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调入内存。
- `type cmd`查看命令cmd是内建命令还是外部命令
- `help`命令只能用来显示内建命令的帮助信息
- `cmd --help` 外部命令基本都有参数 --help ，可以用来显示帮助信息
- `man cmd`

## Linux任务计划crontab
### crontab使用
- $\color{red}{crontab}$
  + 通过crontab命令，可以在固定的间隔时间执行
  + `sudo cron －f &` 启动crontab
  + `crontab -e` 添加计划任务，第一次启动会让选择编辑器，选择第二个vim
  + `*/1 * * * * touch /home/shiyanlou/此处为dollar字符(date +\%Y\%m\%d\%H\%M\%S)` 该任务是每分钟我们会在/home/shiyanlou 目录下创建一个以当前的年月日时分秒为名字的空白文件
  ```
  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR     sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name command to be executed
  ```
  + `crontab -l` 查看计划任务
  + `sudo apt-get install -y rsyslog`
  + `sudo service rsyslog start`
  + `sudo tail -f /var/log/syslog` 查看到执行任务命令之后在日志中的信息反馈

### crontab深入
- 每个用户使用 crontab -e 添加计划任务，都会在 /var/spool/cron/crontabs 中添加一个该用户自己的任务文档

## 命令执行顺序控制与管制
### 命令执行顺序控制
- 顺序执行多条命令 多条命令之间使用分号隔开
- 选择执行多条命令
  + 环境变量dollar?指示上一次命令执行的返回结果，成功为0，否则为1
  + `&&` 如果上一次命令执行结果为0则执行后面的命令，否则跳过；


## new

  + `||` 如果上一次命令执行结果为1则执行后面的命令，否则跳过；
  + `which cowsay>/dev/null && echo "exist" || echo "not exist"`

### 管道
- 管道是一种通信机制，通常用于进程间的通信（也可通过 socket 进行网络通信），它表现出来的形式就是将前面每一个进程的输出(stdout)直接作为下一个进程的输入(stdin)

## 数据流重定向
### 简单的重定向
- `echo 'hello world' > file1` 将echo的输出重定向到文件file1
- `cat`默认使用标准输入作为命令的输入，标准输出作为命令的输出；`cat > file2 << EOF`用heredoc的方式进行输入，将cat的输出重定向到文件file2，heredoc是以>>EOF作为输入开始，以EOF作为输入结束
  ``` bash
  mkdir Documents
  cat > Documents/test.c <<EOF
  #include <stdio.h>

  int main()
  {
      printf("hello world\n");
      return 0;
  }

  EOF
  ```

### 标准错误重定向
- 标准输出和标准错误都被指向屏幕，所以我们可以看到一个命令的结果包含标准输出和标准错误
- `cat file1 > somefile`当file1不存在时，屏幕会显示`cat: file1: 没有那个文件或目录`
- 使用文件描述符，可以将标准错误重定向到标准输出，再将标准输出重定向到文件，注意要将重定向到文件写在前面`cat file1 > somefile 2>&1`

### 永久重定向
- exec是以新的进程去代替原来的进程，但进程的PID保持不变。因此，可以这样认为，exec系统调用并没有创建新的进程，只是替换了原来进程上下文的内容。原进程的代码段，数据段，堆栈段被新的进程所代替。
```bash
#开启一个子shell
zsh
#使用exec替换当前进程的重定向，将标准输出重定向到文件
exec 1>somefile
ls
#进程结束函数
exit
```

### 创建输出文件描述符
- 在 Shell 中有 9 个文件描述符。上面我们使用了也是它默认提供的 0,1,2 号文件描述符。另外我们还可以使用 3-8 的文件描述符，只是它们默认没有打开而已。
- 查看当前shell进程中打开的文件描述符`ls -Al /dev/fd/`
- 使用exec创建新的文件描述符
  ```bash
  zsh
  exec 3>somefile
  echo 'test' >&3
  cat somefile
  exit
  ```
- 关闭文件描述符 `exec 3>&-`

### 屏蔽命令的输出
在 Linux 中有一个被称为“黑洞”的设备文件,所有导入它的数据都将被“吞噬”。在类 UNIX 系统中，/dev/null，或称空设备，是一个特殊的设备文件，它通常被用于丢弃不需要的输出流，或作为用于输入流的空文件，这些操作通常由重定向完成。

利用/dev/null屏蔽命令的输出 `cat somefile 1>/dev/null 2>&1`



## Linux 下软件安装

### apt包管理工具

1. 执行安装操作时，apt-get工具会在本地的一个数据库中搜索关于待安装软件的相关信息，并根据这些信息在相关服务器上下载软件安装

2. `sudo apt-get update` 保持本地软件包列表时最新的，有时需手动执行这个操作，特别是更换了软件源后。这个表里会有软件依赖信息的记录

3. `apt-get`包含的常用工具

   | 工具         | 说明                                                         |
   | ------------ | ------------------------------------------------------------ |
   | install      | 后面加上软件名，用于安装一个软件包                           |
   | update       | 从软件源镜像服务器上下载/更新用于更新本地软件源的软件包列表  |
   | upgrade      | 升级本地可更新的全部软件包，但存在依赖问题时将不会升级，通常会在更新之前执行一次`update` |
   | dist-upgrade | 解决依赖关系并升级(存在一定危险性)                           |
   | remove       | 移除已安装的软件包，包括与被移除软件包有依赖关系的软件包，但不包含软件包的配置文件 |
   | autoremove   | 移除之前被其他软件包依赖，但现在不再被使用的软件包           |
   | purge        | 与 remove 相同，但会完全移除软件包，包含其配置文件           |
   | clean        | 移除下载到本地的已经安装的软件包，默认保存在/var/cache/apt/archives/ |

   下面列出一些`apt-get`常用参数

   | 参数          | 说明                                                         |
   | :------------ | ------------------------------------------------------------ |
   | `-y`          | 自动回应是否安装软件包的选项，在一些自动化安装脚本中使用这个参数将十分有用 |
   | `-f`          | 修复损坏的依赖关系                                           |
   | `-d`          | 只下载不安装                                                 |
   | `-q`          | 静默安装方式，指定多个`q`或者`-q=#`,#表示数字，用于设定静默级别，这在你不想要在安装软件包时屏幕输出过多时很有用 |
   | `-d`          | 只下载不安装                                                 |
   | `--reinstall` | 重新安装软件包                                               |

4. 软件搜索

   `sudo apt-cache search softname`

### dpkg

dpkg从本地磁盘安装deb软件包，我们经常可以在网络上见到以`deb`形式打包的软件包，就需要使用`dpkg`命令来安装。

1. 常用参数介绍

   | 参数 | 说明                                              |
   | ---- | ------------------------------------------------- |
   | `-i` | 安装指定 deb 包                                   |
   | `-R` | 后面加上目录名，用于安装该目录下的所有 deb 安装包 |
   | `-r` | emove，移除某个已安装的软件包                     |
   | `-I` | 显示`deb`包文件的信息                             |
   | `-s` | 显示已安装软件的信息                              |
   | `-L` | 显示已安装软件包的目录信息                        |




## Linux进程的概念

### top工具的使用

1. 第一行

   | 内容                           | 解释                                |
   | ------------------------------ | ----------------------------------- |
   | top                            | 表示当前程序的名称                  |
   | 11:05:18                       | 当前系统的时间                      |
   | up 8 days,17:12                | 表示该机器已启动了多长时间          |
   | 1 user                         | 表示当前系统中只有一个用户          |
   | load average：0.29，0.20，0.25 | 分别对应1、5、15分钟内cpu的平均负载 |

2. 第二行

   | 内容            | 解释                |
   | --------------- | ------------------- |
   | Tasks: 26 total | 进程总数            |
   | 1 running       | 1个正在运行的进程数 |
   | 25 sleeping     | 25个睡眠的进程数    |
   | 0 stopped       | 停止的进程数        |
   | 0 zombie        | 僵尸进程数          |

3. 第三行

   | 内容           | 解释                      |
   | -------------- | ------------------------- |
   | cpu(s): 1.0%us | 用户空间进程占用CPU百分比 |
   | 1.0% sy        | 内核空间运行占用CPU百分比 |
   | 97.0%id        | 空闲CPU百分比             |

4. 第四行

   | 内容    | 解释                 |
   | ------- | -------------------- |
   | total   | 物理内存总量         |
   | used    | 使用的物理内存总量   |
   | free    | 空闲内存总量         |
   | buffers | 用作内核缓存的内存量 |

5. 第五行

   | 内容   | 解释             |
   | ------ | ---------------- |
   | total  | 交换区总量       |
   | used   | 使用的交换区总量 |
   | free   | 空闲交换区总量   |
   | cached | 缓冲的交换区总量 |

### 进程管理



## Linux日志系统

### 常见日志

Linux大部分发行版都内置syslog系统日志，常见的日志一般存放在/var/log中

| 日志名称           | 记录信息                                                     |
| ------------------ | ------------------------------------------------------------ |
| alternatives.log   | 系统的一些更新替代信息记录                                   |
| apport.log         | 应用程序崩溃信息记录                                         |
| apt/history.log    | 使用 apt-get 安装卸载软件的信息记录                          |
| apt/term.log       | 使用 apt-get 时的具体操作，如 package 的下载、打开等         |
| auth.log           | 登录认证的信息记录                                           |
| boot.log           | 系统启动时的程序服务的日志信息                               |
| btmp               | 错误的信息记录                                               |
| Consolekit/history | 控制台的信息记录                                             |
| dist-upgrade       | dist-upgrade 这种更新方式的信息记录                          |
| dmesg              | 启动时，显示屏幕上内核缓冲信息,与硬件有关的信息              |
| dpkg.log           | dpkg 命令管理包的日志。                                      |
| faillog            | 用户登录失败详细信息记录                                     |
| fontconfig.log     | 与字体配置有关的信息记录                                     |
| kern.log           | 内核产生的信息记录，在自己修改内核时有很大帮助               |
| lastlog            | 用户的最近信息记录                                           |
| wtmp               | 登录信息的记录。wtmp 可以找出谁正在进入系统，谁使用命令显示这个文件或信息等 |
| syslog             | 系统信息记录                                                 |

 wtmp，lastlog这两个日志并不是 ASCII 文件而是被编码成了二进制文件，查看的方法是使用 last 与 lastlog 工具来提取其中的信息。