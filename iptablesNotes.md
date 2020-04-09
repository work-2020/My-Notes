# iptables学习笔记

## 一、概述

iptables/netfilter组成Linux平台下的包过滤防火墙，完成封包过滤、封包重定向和网络地址转换（NAT）等功能。

netfilter是防火墙的真正安全框架，位于内核空间，是Linux核心层内部的一个数据包管理模块，具备网络地址转换、数据包内容修改和数据包过滤功能；iptables是命令行工具，位于用户空间，用来操作netfilter框架。

iptables根据规则定义的方法处理数据包，规则主要包括源地址、目的地址、传输协议和服务类型（端口）等等，处理方法主要是放行（accept）、拒绝（reject）和丢弃（drop）等。配置iptables主要是添加、修改和删除这些规则。

对于一台终端的防火墙，数据包存在进入和流出，防火墙针对进入数据包和流出数据包制定不同的规则，可以通俗地理解input关卡和output关卡。又因为终端收到的数据包的目的地址可能并不是自己，而是其他终端，如果本机的内核支持转发（IP_FORWARD）时，可以将数据包转发给其他终端，这时防火墙又需要另外三个关卡路由前（prerouting）、转发（forward）、路由后（postrouting）。数据包经过每一个关卡时，必须逐一匹配每一个规则，因此关卡就是一串规则的集合，iptables称之为链，iptables有5个链：input链、output链、prerouting链、forward链、postrouting链。

iptables将功能相似的规则整合在一起形成“表”，不同功能的规则放在不同表中进行管理。iptables定义了4种表，每种表对应不同的功能。

+ filter表：负责过滤功能
+ nat表：网络地址转换功能
+ mangle表：拆解数据包，修改并重新封装
+ raw表：关闭nat表上启用的连接追踪机制

## 二、iptables详情

### iptables传输过程

1. 当一个数据包进入网卡时，它首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转送出去。
2. 如果数据包就是进入本机的，它就会沿着图向下移动，到达INPUT链。数据包到了INPUT链后，任何进程都会收到它。本机上运行的程序可以发送数据包，这些数据包会经过OUTPUT链，然后到达POSTROUTING链输出。 
3. 如果数据包是要转发出去的，且内核允许转发，数据包就会如图所示向右移动，经过FORWARD链，然后到达POSTROUTING链输出。

```flow
st=>start: 开始
data_in=>inputoutput: 数据包到来
data_out=>inputoutput: 数据包流出
prerouting=>operation: prerouting
input_chain=>operation: input
forward_chain=>operation: forward
output_chain=>operation: output
postrouting_chain=>operation: postrouting
localhost=>operation: 本机处理
cond=>condition: 目的地址是否为本机

data_in->prerouting->cond
cond(yes)->input_chain(right)->localhost
cond(no)->forward_chain->postrouting_chain->data_out
localhost(right)->output_chain->postrouting_chain->data_out
```

### iptables表与链

表（tables）提供特定的功能，iptables内置了4个表，即filter表、nat表、mangle表和raw表，分别用于实现包过滤，网络地址转换、包重构(修改)和数据跟踪处理。

 链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一  条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据  该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包。

**规则表**

1. filter表——三个链：INPUT、FORWARD、OUTPUT
   作用：过滤数据包 内核模块：iptables_filter.
2. Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT
   作用：用于网络地址转换（IP、端口） 内核模块：iptable_nat
3. Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD
   作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS内核模块：iptable_mangle(别看这个表这么麻烦，咱们设置策略时几乎都不会用到它)
4. Raw表——两个链：OUTPUT、PREROUTING
   作用：决定数据包是否被状态跟踪机制处理 内核模块：iptable_raw

**规则链**

1. INPUT——进来的数据包应用此规则链中的策略，mangle->nat->filter
2. OUTPUT——外出的数据包应用此规则链中的策略，raw->mangle->nat->filter
3. FORWARD——转发数据包时应用此规则链中的策略，mangle->filter
4. PREROUTING——对数据包作路由选择前应用此链中的规则，raw->mangle->nat
   （记住！所有的数据包进来的时侯都先由这个链处理）
5. POSTROUTING——对数据包作路由选择后应用此链中的规则，mangle->nat
   （所有的数据包出来的时侯都先由这个链处理）

**记忆方法**

1. 所有的链都可以修改数据包，都包含mangle表；
2. 对于本机和其他终端而言，过滤规则是不同，因此，prerouting和postrouting链中没有过滤规则，只在input、forward和output链中存在；

**规则表之间的优先级**

raw->mangle->nat->filter

## 三、查询iptables

1. `iptables -t filter -L`， `-t`指定要查询的表，`-L`查看-t选项对应的规则。
2. `iptables -L INPUT `，省略`-t`选项，默认是filter表，`INPUT`只查看INPUT链中filter规则。
3. `iptable -vL INPUT`，`-v`查看更多更详细的信息
   + pkts 对应规则匹配到的数据包的个数
   + target 对应规则的动作
   + in 数据包由哪个网卡流入
   + out 
   + source 规则对应的源地址
   + destination 规则对应的目的地址
4. `iptables --line-number -nvL INPUT`  列出序号

## 四、配置iptables

**参数选项**

```
-A 在指定链的添加（append）一条新的规则
-D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
-I 在指定链中插入（insert）一条新的规则，默认在第一行添加
-R 修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
-L 列出（list）指定链中所有的规则进行查看
-E 重命名用户定义的链，不改变链本身
-F 清空（flush）
-N 新建（new-chain）一条用户自己定义的规则链
-X 删除指定表中用户自定义的规则链（delete-chain）
-P 设置指定链的默认策略（policy）
-Z 将所有表的所有链的字节和数据包计数器清零
-n 使用数字形式（numeric）显示输出结果
-v 查看规则表详细信息（verbose）的信息
-V 查看版本(version)
-h 获取帮助（help）
-t 指定要操作的表，缺省表示filter表
-s 指定匹配条件中的源地址
-j 指定匹配规则对应的动作
```

**处理数据包的四种方式**

```
ACCEPT 允许数据包通过
DROP 直接丢弃数据包，不给任何回应信息
REJECT 拒绝数据包通过，必要时会给数据发送端一个响应的信息。
LOG 在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则
```

**保存与恢复规则**

使用命令iptables-save来保存规则，生成保存规则的文件 /etc/sysconfig/iptables

```
iptables-save > /etc/sysconfig/iptables
```

当计算机启动时，rc.d下的脚本将用命令iptables-restore调用这个文件，从而就自动恢复了规则。

**增加规则**

1. 拒绝进入防火墙的所有ICMP协议数据包，`-t`指定要操作的表是filter，缺省时表示filter，`-I`指定将规则插入到哪个链中，并且放置在第一条。

```
iptables -t filter -I INPUT -p icmp -j REJECT
```

2. 允许防火墙转发除ICMP协议以外的所有数据包，说明：使用“！”可以将条件取反。

```
iptables -A FORWARD -p ! icmp -j ACCEPT
```

3. 拒绝转发来自192.168.1.10主机的数据，允许转发来自192.168.0.0/24网段的数据，说明：注意要把拒绝的放在前面不然就不起作用了啊。

```
iptables -A FORWARD -s 192.168.1.11 -j REJECT 
iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT
```

4. 丢弃从外网接口（eth1）进入防火墙本机的源地址为私网地址的数据包

```
iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP 
iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP 
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

5. 只允许管理员从202.13.0.0/16网段使用SSH远程登录防火墙主机。这个用法比较适合对设备进行远程管理时使用，比如位于分公司中的SQL服务器需要被总公司的管理员管理时。

```
iptables -A INPUT -p tcp --dport 22 -s 202.13.0.0/16 -j ACCEPT 
iptables -A INPUT -p tcp --dport 22 -j DROP
```

6. 允许本机开放从TCP端口20-1024提供的应用服务。

```
iptables -A INPUT -p tcp --dport 20:1024 -j ACCEPT 
iptables -A OUTPUT -p tcp --sport 20:1024 -j ACCEPT
```

7. 允许转发来自192.168.0.0/24局域网段的DNS解析请求数据包。

```
iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT 
iptables -A FORWARD -d 192.168.0.0/24 -p udp --sport 53 -j ACCEPT
```

8. 禁止其他主机ping防火墙主机，但是允许从防火墙上ping其他主机

```
iptables -I INPUT -p icmp --icmp-type Echo-Request -j DROP 
iptables -I INPUT -p icmp --icmp-type Echo-Reply -j ACCEPT 
iptables -I INPUT -p icmp --icmp-type destination-Unreachable -j ACCEPT
```

9. 禁止转发来自MAC地址为00：0C：29：27：55：3F的和主机的数据包

```
iptables -A FORWARD -m mac --mac-source 00:0c:29:27:55:3F -j DROP
```

10. 允许防火墙本机对外开放TCP端口20、21、25、110以及被动模式FTP端口1250-1280

```
iptables -A INPUT -p tcp -m multiport --dport 20,21,25,110,1250:1280 -j ACCEPT
```

11. 禁止转发源IP地址为192.168.1.20-192.168.1.99的TCP数据包。

```
iptables -A FORWARD -p tcp -m iprange --src-range 192.168.1.20-192.168.1.99 -j DROP
```

12. 禁止转发与正常TCP连接无关的非—syn请求数据包。

```
iptables -A FORWARD -m state --state NEW -p tcp ! --syn -j DROP
```

13. 拒绝访问防火墙的新数据包，但允许响应连接或与已有连接相关的数据包

```
iptables -A INPUT -p tcp -m state --state NEW -j DROP 
iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
```

14. 只开放本机的web服务（80）、FTP(20、21、20450-20480)，放行外部主机发住服务器其它端口的应答数据包，将其他入站数据包均予以丢弃处理。

```
iptables -I INPUT -p tcp -m multiport --dport 20,21,80 -j ACCEPT 
iptables -I INPUT -p tcp --dport 20450:20480 -j ACCEPT 
iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT 
iptables -P INPUT DROP
```
15. 将 80 端口的封包转递到 8080端口 

```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080 
```

**删除规则**

```
iptables -t filter -D INPUT 1
```







