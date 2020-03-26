# 四、网络层
## 4.1 概述
- 转发  在路由器本地，将数据包从输入接口转移到输出接口的动作
- 路由选择 在整个网络范围，决定数据从源地址到目的地址所采取的路径
- 连接建立

## 4.2 虚电路和数据报网络
- 与传输层类似，网络层也能够在两台主机间提供无连接服务和连接服务
- 网络层仅提供连接服务的计算机网络被称为虚电路网络(Virtual-Circuit VC)，如ATM 帧中继
- 网络层仅提供无连接服务的计算机网络被称为数据报网络(datagram nerwork)，如因特网
- 迄今为止，所有主要的计算机网络不同时提供上述两种服务

  ### 4.2.1 虚电路
  虚电路的概念来源于电话界
  - 虚电路建立  网络层决定该虚电路所有分组所经过的一系列链路和路由器；并为该路径上所有链路决定一个VC号；最后，网络层在路径上的每一个路由器的转发表中增加一个表项
  - 数据传送  建立好虚电路后，分组沿着该虚电路流动
  - 虚电路拆除  一端的网络层通知另一端网络层结束虚电路，并更新路径上每台路由器中的转发表
  - 运输层的连接建立仅涉及两个端系统，网络中路由器对此完全不知情；网络层的连接建立涉及了两个端系统及它们之间路径上所有路由器
    
  ### 4.2.2 数据报网络
  - 路由器转发表是核心，转发表用分组目的地址的前缀与转发表中的表项进行匹配，如果有一个匹配项，则向其对应的链路接口转发分组；如果有多个匹配项，依据最长前缀匹配规则进行转发
  - 路由器转发表依据路由选择算法进行更新，通常1至5分钟更新一次
    
## 4.3 路由器工作原理
- 路由器由输入端口、交换结构、输出端口和路由选择处理器组成
- 路由器的输入端口、输出端口、交换结构实现转发功能，总是用硬件实现，称为路由器转发平面；路由选择处理器由软件实现，称为路由器控制平面
- 输入端口最重要的动作是在路由表中查找匹配项，还要进行物理层和链路层处理；检查分组的版本号、校验和寿命字段，并重写后两个字段；更新网络管理计数器

## 4.4 IP协议
- IPv4编址 CIDR 无类别域间路由选择 32bitIP地址被分为两部分 a.b.c.d/x，x指示了地址第1部分中的比特数
- DHCP 动态主机配置协议 
  + DHCP服务器发现 新到的主机找到与其交互的DHCP服务器
  + DHCP服务器提供 DHCP服务器向客户作出响应，向客户提供IP地址
  + DHCP请求 客户从一个或多个服务器中选择一个，并用请求报文进行回应
  + DHCP ACK 服务器对DHCP请求进行响应
- 网络地址转换 NAT 解决外网与内网通信的问题
- Question NAT如何妨碍P2P应用程序？NAT穿越？

## 4.5 路由选择算法
- 默认路由器 第一跳路由器
- 分类
  + 全局式路由选择算法 需要有关全局状态OSPF
  + 分散式路由选择算法 
  
  ### 4.5.1 链路状态路由选择算法
  - 需已知网络拓扑和各个链路费用，记D(v)是源节点到目的节点v的最低费用路径的费用，c(u,v)表示两个相邻节点的链路费用，p(v)表示源节点到目的节点v的最低费用路径中的前一节点（v的邻居）
  - 第一次迭代，计算源节点u到所有邻居节点的费用，非邻居节点的费用记为无穷大，得到D(v)，将费用最小的节点添加到已知节点集合中，并记录p(v) = u
  - 第k次迭代，记上一次迭代添加的节点为w，若存在w的邻居节点x，满足D(w)+c(w,x) < D(x)，意味着找到了源节点到节点x的一条更优路径，更新D(x)的值，并记录p(x) = w
  - 直到网络中所有节点都在已知节点集合中
  - 当链路状态路由选择算法终止时，对于任意节点，我们得到了从源节点到它的最低费用路径上它的前一节点，如此便可以构建出完整路径

## 复习题
- R1 网络层分组的名称是数据报（datagram）。路由器是根据网络层IP地址进行转发，交换机是根据分组的MAC地址
- R2 网络层最重要的两个功能是转发和路由选择。虚电路网络多了一个重要功能是虚电路建立
- R3 转发是将数据报从路由器的输入接口转移到输出接口；路由选择是决定从源地址到目的地址的路径
- R4 均使用转发表。
- R7 有了转发表影子副本，转发决策可以在输入端本地作出，无须调用中央路由处理器，避免了集中式处理的瓶颈
- R12 路由器每个接口有一个IP地址
- R13 11011111 00000001 00000011 00011011
- R15 8个接口；查询3张转发表
- R17 网络层根据IP头中的协议字段决定将数据交给哪个传输层协议
- R18 路由器充当DHCP服务器，为5台PC和路由器接口分配IP地址，同时该路由器使用NAT协议实现内网IP与公网IP转换

## 套接字编程实验
- `bytes(str.encode('utf-8'))` 将字符串转为字节流，可以使用下标访问每一个字节，类型是int型
- `struct.pack(fmt,v1,v2,...)` 按照给定格式，将数据封装成字节流
- `select.select()` 没整明白

```python
import struct
import os
import socket
import math
import time
import select

class Pinger:
    def __init__(self, target_host):
        self.target_host = target_host
        self.timeout = 2
    def checksum(self, str_data):
        count = math.floor(len(str_data)/2)
        cksum = 0
        for i in range(count):
            cksum = cksum + str_data[2*i] + str_data[2*i+1] * 256
            cksum = cksum & 0xffffffff
        if len(str_data) % 2 == 1:
            cksum = cksum + ord(str_data[-1])
            cksum = cksum & 0xffffffff
            
        cksum = (cksum>>16) + (cksum&0xffff)
        cksum = (cksum>>16) + (cksum&0xffff)
        answer = (~cksum) & 0xffff
        return answer
    
    def send_ping(self,sock,ID):
        cksum = 0
        header = struct.pack('bbHHh',8,0,cksum,ID,1)
        bytes_in_double = struct.calcsize('d')
        data = (192-bytes_in_double) * 'Q'
        data = struct.pack('d',time.time()) + bytes(data.encode('utf-8'))
        
        cksum = self.checksum(header+data)
        header = struct.pack('bbHHh',8,0,cksum,ID,1)
        packet = header+data
        sock.sendto(packet,(self.target_host,12345))
        print(packet)
        
    def receive_pong(self, sock, ID, timeout):
        """
        Receive ping from the socket.
        """
        time_remaining = timeout
        while True:
            start_time = time.time()
            readable = select.select([sock], [], [], time_remaining)
            time_spent = (time.time() - start_time)
            if readable[0] == []: # Timeout
                return

            time_received = time.time()
            recv_packet, addr = sock.recvfrom(1024)
            icmp_header = recv_packet[20:28]
            type, code, checksum, packet_ID, sequence = struct.unpack("bbHHh", icmp_header)
            if packet_ID == ID:
                bytes_In_double = struct.calcsize("d")
                time_sent = struct.unpack("d", recv_packet[28:28 + bytes_In_double])[0]
                return time_received - time_sent

            time_remaining = time_remaining - time_spent
            if time_remaining <= 0:
                return
        
    def ping_once(self):
        icmp = socket.getprotobyname('icmp')
        try:
            sock = socket.socket(socket.AF_INET,socket.SOCK_RAW,icmp)        
        except socket.error as e:
            print(e.errno)
            if e.errno == 1:
                e.msg += 'ICMP message can only be sent from root user processes'
                raise socket.error(e.msg)
            else:
                raise socket.error(e)
        except Exception as e:
            print("Exception: %s"%(e))
        else:
            ID = os.getpid() & 0xffff
            self.send_ping(sock, ID)
            #print('send')
            delay = self.receive_pong(sock,ID,self.timeout)
            sock.close()
            return delay
        
    
    def ping(self):
        """
        Run the ping process
        """
        for i in range(2):
            print ("Ping to %s..." % self.target_host,)
            try:
                delay  =  self.ping_once()
            except socket.gaierror as e:
                print ("Ping failed. (socket error: '%s')" % e[1])
                break

            if delay  ==  None:
                print ("Ping failed. (timeout within %ssec.)" % self.timeout)
            else:
                delay  =  delay * 1000
                print ("Get pong in %0.4fms" % delay)

if __name__ == '__main__':
    pinger = Pinger('192.168.2.113')
    pinger.ping()
```

# 三、运输层

网络层提供主机之间的逻辑通信，运输层为运行在不同主机上的进程之间提供了逻辑通信
- 多路分解 将运输层报文段中的数据交付到正确的套接字的工作
- 多路复用 从不同套接字中收集数据块，并为每个数据块封装头部信息从而生成报文段，再将报文段传递到网络层的工作
- UDP套接字由一个二元组标识，即目的IP地址和目的端口号
- TCP套接字由一个四元组标识，即源IP地址、源端口号、目的IP地址、目的端口号

## 3.3 UDP
- 报文段结构：源端口+目的端口+长度+校验和+报文。校验和与IP头校验一致
- UDP套接字（目的IP地址，目的端口号），若两个UDP报文段有不同源IP地址和/或源端口号，但有相同目的IP地址和目的端口号，则这两个报文段将通过相同的套接字被定向到相同的目的进程。

## 3.4 可靠数据传输原理

  ### 3.4.1 构造可靠传输协议
  可类比于两个人电话转述消息，接收方通过肯定确认和否定确认两种方式告知发送方继续或是重复
  - 校验和 检查数据分组是否出错
  - 序号   便于接收方确定自己收到的分组是否是一次重传（接收方无法确认发送方是否收到了自己的ACK分组）
  - 定时器 解决是否重传的问题
  - 确认   可靠传输的保证


  ### 3.4.2 流水线可靠传输
  流水线可靠传输的差错恢复有两种基本方法：回退N步（go-back-N）、选择重传（Selective Repeat）
  
  ### 3.4.3 回退N步（GBN）
  - 窗口长度N 流水线中未确认的分组数不能超过某个最大允许数N
  - 基序号base 最早的未确认分组的序号
  - 下一个序号nextseq  下一个待发送的序号
  
  - GBN发送方必须响应的三种事件
    + 上层的调用 检查发送窗口是否已满(nextseq-base == N)
    + 收到ACK GBN采用累积确认，意味着该分组之前的分组都已正确收到，更新base
    + 超时 发送方重传已发送但未被确认的分组
  - GBN接收方
    + 正确接收序号为n的分组，并且上次交付应用层的分组序号为n-1，则为分组n发送ACK
    + 否则，丢弃该分组（根据GBN重传规则，该分组会重新发送）
    
  
  ### 3.4.4 选择重传（SR）
  - GBN提高了信道利用率，但在差错率较高的信道中，会发生大量重传，导致严重性能问题
  - 接收方向发送方确认收到的每一个分组
  - 发送发仅重传没有确认的分组，避免大量重传
  
  - SR发送方的事件和动作
    + 上层的调用 与GBN一样，检查发送窗口是否已满
    + 超时 每个分组都有一个逻辑定时器，超时后只发送一个分组
    + 收到ACK 若该分组序号在窗口内，将其标记为已接受；若该分组序号等于send_base，则窗口基序号移动到最小的未确认分组序号
    
  - SR接收方的事件和动作
    + 正确接收序号在[rcv_base, rcv_base+N-1]内的分组 发送ACK到发送方，如果分组序号等于接收窗口基序号，将收到的若干连续分组向上交付，并更新接收窗口基序号
    + 正确接收序号在[rcv_base-N, rcv_base-1]内的分组 发送ACK到发送方（即使该分组已被正确接收过）
    + 其他 忽略该分组
    
  -  窗口长度必须小于或等于序号空间大小的一半，否则无法区分旧分组重传和新分组初次传输

## 3.5 TCP
  ### 3.5.2 TCP报文段结构
  - 首部： 源端口 目的端口 序号 确认号 首部长度 选项字段 6比特标志字段 接收窗口
    + 标志字段：URG|ACK|PSH|RST|SYN|FIN，ACK指示报文段中包含一个对成功接收报文段的确认；RST、SYN、FIN用于连接的建立和拆除；PSH、URG在事件中未使用
    + 接收窗口：用于流量控制
    + 序号：该报文段首字节的字节流编号，通常初始序号是随机的
    + 确认号： 主机正在等待的下一个字节序号。TCP采取累积确认（确认号之前的数据都已正确收到）
    
    
  ### 3.5.3 往返时间的估计与超时
  - 估计往返时间
    + 样本RTT（SampleRTT）: 某报文段发出到该报文段被确认收到之间的时间量
    + RTT均值（EstimatedRTT）： EstimatedRTT = 0.875 * EstimatedRTT + 0.125 * SampleRTT，最近的样本被赋予较大的权值
    + RTT偏差（DevRTT）：DevRTT = 0.75 * DevRTT + 0.25 * |SampleRTT - EstimatedRTT|
    + 对于重传的报文段，TCP不测量它的样本RTT，因为不能确定发生重传后接收到的ACK是对哪一次发送的报文段的ACK
  - 设置超时重传
    + TCP超时间隔（TimeoutInterval）应大于RTT均值，否则会造成不必要的重传；但又不应比RTT均值大太多，否则当报文段丢失时，TCP不能很快地重传该报文，导致数据传输时延大，因此超时间隔应设置为RTT均值加上一定余量
    + 当样本RTT波动较大时，这个余量应该大一些；当波动较小时，这个余量应该小一些
    + TimeoutInterval = EstimatedRTT + 4 * DevRTT
    
    
  ### 3.5.4 可靠数据传输 
  TCP的可靠数据传输服务依赖两项机制：超时重传、快速重传
  - 定时器：对于多个已发送但未被确认的报文段，TCP仅采用单一的重传定时器，TCP发送方在下面3种事件发生时对定时器的操作如下，
    + 发送报文段：若定时器未启动，启动定时器
    + 超时： 重传第一个未被确认的报文段，重启定时器
    + 收到ACK：更新发送基序号，若还有已发送但未被确认的报文段，重启定时器
  - 超时重传：
    + 当发生超时事件时，TCP重传第一个未被确认的报文段。
    + 当发生超时事件时，TCP将超时时间间隔设置为之前的2倍，而不是用EstimatedRTT和DevRTT推算的值（定时器在另外两个事件中启动时，超时事件间隔是由最近的EstimatedRTT和DevRTT推算得到），该机制提供了受限的拥塞控制。
  - 快速重传：
    + 冗余ACK： 当TCP接收方收到一个比期望序号大的报文段，它对已经接收到的最后一个按序字节数据进行重复确认（即产生一个冗余ACK）
    + 快速重传：一旦发送方收到3个冗余ACK，发送方立即重传接收方期望的报文段，而不再等待定时器过期。
    + $\color{red}{捎带确认与数据接收触发确认:}$ TCP是全双工通信协议，它原本应该全双工两个方向独立进行传输/确认，为了提高资源利用率，实施了$\color{red}{捎带确认}$ 的逻辑，也就是说，捎带确认只是一个优化，而不是与生俱来的。只有当数据接收触发的确认是冗余ACK时，才会快速重传，而连续发送大量数据时的捎带确认并不会被认为是冗余ACK。
  - TCP的差错恢复机制应被视作GBN和SR的混合体
  
  
  ### 3.5.5 流量控制
  当接收方的接收缓存有限，当发送方发送地太快、太多时，容易造成接收方的接收缓存溢出，因此接收方主动告知发送方自己还有多少可用的缓存空间。
  - 接收窗口： 可用的接收缓存，在TCP首部结构，TCP通联双方各自维护一个接收窗口
  - 发送方应确保已发送但未确认的数据量小于接收窗口的大小
  - 当接收方接收窗口为0时，发送方停止发送数据，接收方的上层进程会清空接收缓存，但接收方无法主动告知发送方已有新的接收空间。为解决该问题，TCP规范中要求，当接收方的接收窗口为0时，发送发继续发送只有一个字节数据的报文段，这些报文段会被确认，接收缓存被清理后，确认报文里将包含非0的接收窗口值。
  
  
  ### 3.5.6 TCP连接管理
  建立TCP连接
  - 第一步：客户端TCP向服务器端TCP发送一个特殊的TCP报文段。首部SYN被置1，客户端随机选择初始序号client_isn放置在序号字段
  - 第二步：服务器收到客户端的TCP SYN报文段，为该TCP连接分配缓存和变量，并向客户端发送允许连接的报文段。该报文段首部包含3个重要信息，SYN被置1、确认字段为clinet_isn+1、服务器随机选择自己的初始序号server_isn，该报文段被称为SYN ACK报文段
  - 第三步：客户端收到SYN ACK报文段后，给该TCP连接分配缓存和变量，同时客户向服务器发送确认报文段，将server_isn+1放置在TCP报文段的确认字段、SYN被置为0。
  
  
  关闭TCP连接
  参与TCP连接的两个进程中的任何一个都能终止该连接。
  - 进程A向进程B发送关闭TCP连接的报文段，其首部FIN置1，进程B收到后，向进程A回送确认报文段ACK，
  - 然后，进程B发送它自己的终止连接报文段，其FIN同样被置1；进程A收到后，同样向进程B发送确认报文段。
  
  
  ## 3.6 拥塞控制原理
  ### 3.6.1 拥塞的原因与代价
  - 当分组的到达速率接近链路的容量时，分组经历巨大的排队时延
  - 发送发必须执行重传以补偿因为缓存溢出而丢失的分组
  - 发送方在遇到大时延时所进行的不必要的重传会引起路由器利用其链路带宽来转发不必要的分组副本
  - 当一个分组沿一条路径被丢弃时，每个上游路由器用于转发该分组到丢弃该分组而使用的传输容量最终被浪费掉了
  
  
  ## 3.7 TCP拥塞控制
  - TCP发送方如何感知拥塞： 超时或收到3个冗余ACK，发送方就认为路径上出现了拥塞现象；
  - 拥塞窗口（cwnd）：对TCP发送方向网络中发送数据的速率进行限制，发送方未被确认的数据量不会超过拥塞窗口和接收窗口的最小值
  - 慢启动阈值（ssthresh）：慢启动阈值可理解为上一次发生拥塞时拥塞窗口值的一半，当拥塞窗口值到达或超过慢启动阈值时，应谨慎增加拥塞窗口值。
  
  
  TCP拥塞控制算法包含3个主要部分：慢启动、拥塞避免、快速恢复
  - 慢启动
    + TCP连接开始时，cwnd的值通常设置为一个MSS，ssthresh设置为64kb
    + 收到1个确认，cwnd = cwnd+MSS，dupACKcount = 0，这样可使得拥塞窗口以指数级增长；
    + 收到冗余ACK时，dupACKcount++;
    + 当发生超时时，令慢启动阈值ssthresh=cwnd/2，cwnd=MSS，dupACKcount = 0，重传丢失的报文段
    + 当dupACKcount==3，即收到3个连续冗余ACK时，令慢启动阈值ssthresh=cwnd/2，cwnd=ssthresh+3*MSS，并进入快速恢复状态；
    + 当cwnd > ssthresh时，进入拥塞避免状态
  - 拥塞避免
    + 一旦进入拥塞避免状态，cwnd的值约为上一次遇到拥塞时值得一半，TCP每经过一个往返时延RTT，只将cwnd的值增加1个MSS，而不是再翻倍。一种通用的做法是，收到一个ACK后，cwnd = cwnd+MSS*(MSS/cwnd)，dupACKcount = 0；
    + 收到冗余ACK时，dupACKcount++；
    + 发生超时时，令慢启动阈值ssthresh=cwnd/2，cwnd=MSS，dupACKcount = 0，重传丢失的报文段，并进入慢启动状态；
    + 当dupACKcount==3，即收到3个连续冗余ACK时，令慢启动阈值ssthresh=cwnd/2，cwnd=ssthresh+3*MSS，并进入快速恢复状态；
  - 快速恢复
    + 收到冗余ACK，cwnd=cwnd+MSS，继续发送新的报文段
    + 发生超时时，令慢启动阈值ssthresh=cwnd/2，cwnd=MSS，dupACKcount = 0，重传丢失的报文段，并进入慢启动状态；
    + 收到正确的ACK时，令cwnd=ssthresh，dupACKcount = 0，并进入拥塞避免状态
    + $\color{red}{在收到冗余ACK时，快速恢复状态与慢启动、拥塞避免两个状态的处理方式不同。}$ 快速恢复状态会增加拥塞窗口值并继续发送新的报文段，而另外两种状态则是等待正确的ACK，不再发送新的报文段
    
    
  ## 复习题
  - R4 开发者不希望其应用的性能受到TCP拥塞控制的影响，并且他们的应用程序并不需要可靠的传输
  - R5 防火墙阻止了很多UDP传输
  - R6 可以。应用层自己实现可靠传输
  - R7 使用相同的UDP套接字。接收主机根据源IP地址的不同进行区分
  - R8 Web服务采用TCP连接，TCP套接字由一个四元组标识，即源IP地址、源端口号、目的IP地址、目的端口号，所以不会通过C上的相同套接字。
  - R9 便于接收方确认收到的数据是新数据还是之前数据的重传。
  - R10 便于发送方判断是否需要重传。
  - R11 仍然需要定时器。便于发送方判断报文段是否丢失。
  - R14 a.错 b.错 c.对 d.错 e.对 f.错 g.错
  - R15 a.20 b.90
  - R16 3个报文段。
  - R17 R/2
  - R18 错。设置为发生拥塞时拥塞窗口值的一半。 
  
  
# 二、应用层
## 2.2 Web和HTTP
HTTP服务器并不存储关于客户的任何信息，所以我们说HTTP是一个无状态协议（stateless protocol）。
### 2.2.3 HTTP报文格式
1、请求报文
- HTTP请求报文的第一行叫做请求行，包含方法字段、URL字段和HTTP版本字段。
- HTTP请求报文第二行开始到空行为止叫做首部行。
- HTTP请求报文中首部行后面是实体体（entity body）
- 使用GET方法时，实体体（entity body）为空，使用POST方法时才使用实体体。当用户提交表单时，常常使用POST方法；
- 用表单生成的请求报文不是必须使用POST方法，也可以采用GET方法+扩展的URL；
- HEAD方法类似于GET方法，服务器收到HEAD方法的请求时，会用HTTP报文进行响应，但不返回对象，常常用来调试跟踪；
- PUT方法允许用户上传对象到指定的Web服务器的指定目录；
- DELETE方法允许用户删除Web服务器上的对象。

2、响应报文
- HTTP响应报文的第一行叫做状态行，包含协议版本字段、状态码、相应状态信息
- 状态码
  + 200 OK 请求成功
  + 301 请求的对象被永久转移了
  + 400 差错代码，该请求不能被服务器理解
  + 404 请求的文档不在服务器上
  + 505 服务器不支持请求报文使用的HTTP协议版本
- HTTP响应报文第二行开始到空行为止称为首部行，包含date,server,last-modified,content-length,content-type

### 2.2.4 用户与服务器的交互：cookie
HTTP是一个无状态协议，但服务器通常希望可以识别用户，因为服务器希望限制用户的访问或是希望将内容与用户身份联系起来，cookie技术用于实现上述目的。<br>
cookie技术有4个组件
- HTTP响应报文中有一个cookie首部行
- HTTP请求报文中有一个cookie首部行
- 用户端系统中保存一个cookie文件，并由用户浏览器管理
- web站点后端数据库

### 2.2.5 Web缓存
Web缓存器（Web catch）也叫代理服务器（proxy server），是能够代表初始Web服务器满足HTTP请求的网络实体。Web缓存器有自己的磁盘存储空间，存储着最近请求过的对象的副本。<br>
部署Web缓存器的两个原因：1、可以大大减少对客户请求的响应时间；2、大大减少一个机构到互联网的通信量。<br>
如何实时更新Web缓存器存储的对象？？？

### 2.2.6 条件GET方法
为解决Web缓存器中存储对象是否是陈旧的问题，可以采用条件GET方法。<br>
条件GET方法：1、请求报文使用GET方法；2、请求报文首部行中包含“If-Modified-Since”，后面接一个日期。<br>
服务器收到使用条件GET方法的请求报文后，判断被请求的对象在请求报文中给出的时间后是否发生过修改，若未修改，响应报文状态码为304 Not Modified，实体体为空。<br>
## 2.3 文件传输协议 FTP
- FTP使用两个并行的TCP连接来传输文件，一个是控制连接，一个是数据连接
- FTP在一个数据连接上仅仅准确传输一个文件，然后关闭该连接。在同一个回话期间，如果用户还需要传输另一个文件，FTP则打开另一个数据连接。

## 2.4 电子邮件
电子邮件系统有3个组成部分：用户代理，邮件服务器，简单邮件传输协议（SMTP）。
从发送方的用户代理开始，传输到发送方的邮件服务器，再传输到接收方的邮件服务器，最后再分发到接收方的邮箱中。
### 2.4.1 SMTP
限制所有邮件报文的体部分（不只是首部）只能采用简单的7比特ASCII表示
### 2.4.4 邮件访问协议
ALice给Bob发送邮件，经历如下过程：Alice的代理使用SMTP协议将邮件传输到Alice的邮件服务器，Alice的邮件服务器使用SMTP协议将邮件传输到Bob的邮件服务器，最后Bob使用POP3、IMAP或HTTP协议从他的邮件服务器获取邮件

## 2.5 DNS 因特网的目录服务
DNS（Domain Name System）域名系统，提供主机名到IP地址的转换
- DNS请求和回答报文使用UDP数据报经端口53发送；
- DNS是一个在因特网上的分布式数据库
  + 分布式、层次数据库：根DNS服务器、顶级域（Top-Level Domain TLD）DNS服务器和权威DNS服务器。在DNS层次体系之外，还存在一类重要的DNS服务器，称为本地DNS服务器；
  + DNS缓存 为改善时延性能并减少因特网上DNS报文数量，DNS采用了缓存技术；由于主机名与IP地址间的映射并不是永久的，DNS服务器一般在两天后丢弃缓存的信息。
- DNS记录 记录为一个四元组（Name, Value, Type, TTL）。忽略TTL，Name和Value的含义取决于Type类型：
  + Type=A，Name是主机名，Value是IP地址
  + Type=NS，Name是个域，Value是知道该域中主机IP地址的权威DNS服务器的主机名
  + Type=CNAME，Value是别名为Name的主机对应的规范主机名
  + Type=MX，Value是别名为Name的邮件服务器的规范主机名
