# MS17-010

SMB Transaction子协议包括以下6种命令：

```
SMB_COM_TRANSACTION
SMB_COM_TRANSACTION_SECONDARY
SMB_COM_TRANSACTION2
SMB_COM_TRANSACTION2_SECONDARY
SMB_COM_NT_TRANSACT
SMB_COM_NT_TRANSACT_SECONDARY
```

SMB Transaction命令相关的三组子命令码

```
SMB_COM_TRANSACTION
    TRANS_MAILSLOT_WRITE 0x0001
    TRANS_SET_NMPIPE_STATE 0x0001
    TRANS_RAW_READ_NMPIPE 0x0036
SMB_COM_TRANSACTION2
    TRANS2_OPEN2 0x0000
    TRANS2_FIND_NEXT2 0x0002
    TRANS2_QUERY_FILE_INFORMATION 0x0007
SMB_COM_NT_TRANSACT
    NT_TRANSACT_CREATE 0x0001
    NT_TRANSACT_SET_SECURITY_DESC 0x0003
```

+ SMB_COM_TRANSACTION命令用于和邮槽、命名管道进行通信。
+ SMB_COM_TRANSACTION2命令用于打开或创建一个共享文件或文件夹，设置它们的扩展属性。
+ SMB_COM_NT_TRANSACT命令用于打开或创建一个文件或文件夹，并应用扩展属性EA或安全描述符SD。
+ SMB_COM_*TRANSACT*_SECONDARY的作用就是，当一个需要发送的transaction消息的实际长度超过SMB_Parameters中MaxBufferSize字段能够定义的最大长度时，客户端必须通过一个或多个SMB_COM_*TRANSACT*_SECONDARY命令来发送剩余的消息内容。SMB_COM_*TRANSACT*_SECONDARY必须保证和SMB_COM_*TRANSACT*命令具有相同的TID、UID、PID和MID。

Windows SMB transaction的实现细节。

TRANSACTION结构体和Transaction data buffer总是分配在同一个缓冲区，在内存中它们是相邻的：

```
+-----------------+--------------------------------------------+
|   TRANSACTION   |   transaction data buffer                  |
+-----------------+--------------------------------------------+
```

TRANSACTION和Transaction data buffer共同组成了的缓冲区称之为Transaction buffer。

在transaction data buffer中，InParameters、OutParameters、InData、OutData缓冲区有三种布局。

**第一种：**除TRANS_MAILSLOT_WRITE和SetupCount字段置为0的”TRANS“数据包外，其它SMB_COM_TRANSACTION数据包的内存布局如下所述，In*和 Out*缓冲区是重叠的。

```
+---------------+------------------------------------------------------+
|  TRANSACTION  |             transaction data buffer                  |
+---------------+------------------------------------------------------+
                | InSetup |   InParameters   |      InData       |     |
                +------------------------------------------------------+
                |  OutParameters  |            OutData                 |
                +------------------------------------------------------+
```

重叠的含义是指，InSetup、InParameters、InData所在内存位置，就是OutParameters、OutData所在内存位置。

**第二种：**除第一种情况外的其它SMB_COM_TRANSACTION数据包和所有SMB_COM_TRANSACTION2数据包的内存布局如下所述，所有缓冲区都不重叠。

```
+---------------+-------------------------------------------------------------------+
|  TRANSACTION  |                  transaction data buffer                          |
+---------------+-------------------------------------------------------------------+
                | InSetup | InParameters |   InData   |  OutParameters  |  OutData  |
                +-------------------------------------------------------------------+
```

**第三种：**SMB_COM_NT_TRANS数据包的内存布局如下所述，InParameters和OutParameters之间、InData和OutData之间的缓冲区都是重叠的。

```
+---------------+-----------------------------------------------------------+
|  TRANSACTION  |               transaction data buffer                     |
+---------------+-----------------------------------------------------------+
                | InSetup |      InParameters    |     InData      |        |
                +---------+----------------------+--------------------------+
                |         |  OutParameters  |    |        OutData           |
                +-----------------------------------------------------------+
```

## Windows SMB transaction 实现漏洞

### 1、Transaction InParameters和InData缓冲区未初始化为0

通过发送多个Parameter偏移为0且Data偏移为0的transaction请求（ParameterCount和DataCount无论偏移是多少都会增长），因此服务器会将未初始化的trans_parameter和trans_data作为后续处理函数的输入，如果可以找到一个将输入数据作为输出返回给客户端的transaction命令，就可以利用该命令泄漏未初始化的数据内容。

这个能够完美利用该bug的transaction子命令就是NT_TRANSACT_RENAME。

### 2、TRANS_PEEK_NMPIPE子命令始终期望MaxParameterCount为16

SrvAllocationTransaction函数用于申请transaction结构和transaction  data缓冲区。如果申请的transaction  data缓冲区长度大于0×10400字节，该函数就将指向transaction的指针设置为NULL，然后向客户端返回一个STATUS_INSUFF_SERVER_RESOURCES/0xC0000205错误码。

OutParameters和OutData的内存位置是相邻的，正常情况下OutData Pointer = OutParameters + 16。但如果故意设置MaxParameterCount的值大于16，OutData Pointer = OutParameters + MaxParameterCount，就有可能泄露未初始化的OutData缓冲区。

为了修复上面这个内存泄露漏洞，在调用SrvAllocationTransaction函数之前，MS17-010补丁会将TRANS_PEEK_NMPIPE命令的MaxParameterCount字段强制修改为16。这样就会导致，即使原本MaxParameterCount、MaxDataCount二者之和大于0×10400，如果MaxParameterCount被修改后，二者之和很没有超过0×10400，SrvAllocationTransaction函数就不会返回错误码，反而继续调用SrvPeekNamedPipe函数。

在实际环境中，可以构造TRANS_PEEK_NMPIPE命令请求数据包，保证MaxParameterCount、MaxDataCount满足二者之和大于0×10400，MaxDataCount+16小于0×10400。如果目标主机没有安装MS17-010补丁，则会返回一个0xC0000205错误码；如果目标主机已经修复了这个漏洞，则响应给客户端的内容就不是0xC0000205错误码

### 3、允许Transaction响应数据长度大于申请的缓冲区长度

SMB_COM_NT_CREATE_ANDX (0xa2)  打开文件，获取文件名，获得读取文件的 总长度。