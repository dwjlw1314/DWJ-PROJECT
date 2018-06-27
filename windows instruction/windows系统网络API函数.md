计算机网络及Windowssocket网络编程相关函数说明

一、获取计算机的IP地址和名称：利用函数GetComputerName()
/*The GetComputerName function retrieves the NetBIOS name of the local computer.
This name is established at system startup, when the system reads it from the registry.*/

>BOOL GetComputerName( LPTSTR lpBuffer, LPDWORD lpnSize);

获取计算机名称:

    TCHAR ch[20];
    memset(ch,0,20*sizeof(TCHAR));
    DWORD dw = 20;
    GetComputerName(ch,&dw);
    AfxMessageBox((CString)ch);

利用WinSock库也可以获取计算机的名称，但是首先必须得初始化WinSock库。

gethostname函数：
/*This function returns the standard host name for the local machine.*/
>int gethostname(char FAR* name, int namelen);  <br>
没有错误则返回0，有错误则返回SOCKET_ERROR，通过WSAGetLastError获取指定的错误代码

    char name[20];
    memset(name,0,sizeof(char)*20);
    gethostname(name,20);

gethostbyname函数：从主机数据库中得到对应的主机的相应信息，如IP地址等。
/*This function retrieves host information corresponding to a host name from a host database.*/

name是主机名字，一般是由gethostname获得的名字  <br>
>struct hostent FAR* gethostbyname(const char FAR* name );

    空指针结尾的主机地址列表
    struct hostent {
      char FAR* h_name;//正规的主机名字
      char FAR* FAR* h_aliases;//一个以空指针结尾的可选主机名队列
      short h_addrtype;//返回地址类型
      short h_length;//每个地址的长度
      char FAR* FAR* h_addr_list;
    };

获取IP地址:

    char name[] = “www.sina.com.cn”;
    struct hostent *p_HostEnt;
    p_HostEnt = gethostbyname(name);

    if(p_HostEnt != NULL)
    {
    	WCHAR HostAddress[20];
    	wsprintf(HostAddress,_T("%d.%d.%d.%d"),(p_HostEnt->h_addr_list[0][0] & 0x00ff),(p_HostEnt->h_addr_list[0][1] & 0x00ff),
    	(p_HostEnt->h_addr_list[0][2] & 0x00ff),(p_HostEnt->h_addr_list[0][3] & 0x00ff));

        AfxMessageBox((CString)HostAddress);
    }

gethostbyaddr函数：从网络地址得到对应的主机。
/*This function retrieves the host information corresponding to a network address.*/  <br>
>struct hostent FAR* gethostbyaddr(const char FAR* addr,  int len,  int type);  <br>
addr为地址，len为地址的长度，type为地址类型，AF_INET。

根据地址来获取主机名字:

    char sin[] = "100.0.0.150";
    DWORD dw = inet_addr(sin);//网络字节序
    int len = strlen(sin);
    hostent* pHostEnt;
    pHostEnt = gethostbyaddr((LPSTR)(&dw),len,AF_INET);

二、获取计算机子网掩码

用函数GetAdaptersInfo可以获取本地计算机的网络信息，从而获得该计算机的子网掩码。
/*MSDN: This function retrieves adapter information for the local computer.*/
>DWORD GetAdaptersInfo(PIP_ADAPTER_INFO pAdapterInfo,  PULONG pOutBufLen);  <br>
pAdapterInfo为指向IP_ADAPTER_INFO，此结构体包含了本地计算机上的一个特定网络适配卡的信息：

```
typedef struct _IP_ADAPTER_INFO {
  struct _IP_ADAPTER_INFO* Next;
  DWORD ComboIndex;
  Char AdapterName[MAX_ADAPTER_NAME_LENGTH + 4];//网卡名
  char Description[MAX_ADAPTER_DESCRIPTION_LENGTH + 4];//对网卡的描述
  UINT AddressLength;//物理地址的长度
  BYTE Address[MAX_ADAPTER_ADDRESS_LENGTH];//物理地址
  DWORD Index;//网卡索引号
  UINT Type;//网卡类型
  UINT DhcpEnabled;//是否启用了DHCP动态IP分配
  PIP_ADDR_STRING CurrentIpAddress;//当前使用的IP
  IP_ADDR_STRING IpAddressList;//绑定到此网卡的IP地址链表
  IP_ADDR_STRING GatewayList;//网关地址链表
  IP_ADDR_STRING DhcpServer;//DHCP服务器地址
  BOOL HaveWins;//是否启用了WINS
  IP_ADDR_STRING PrimaryWinsServer;//主WINS地址
  IP_ADDR_STRING SecondaryWinsServer;//辅WINS地址
  time_t LeaseObtained;//当前DHCP租借获取的时间
  time_t LeaseExpires;//失效时间
} IP_ADAPTER_INFO, *PIP_ADAPTER_INFO;
```
需要包含头文件Iphlpapi.h，需要包含库文件Iphlpapi.lib

获取网卡的相关信息，还可以枚举网卡
```
IP_ADAPTER_INFO* pAdapterInfo;
pAdapterInfo = (IP_ADAPTER_INFO *) malloc( sizeof(IP_ADAPTER_INFO) );
ULONG ulOutBufLen = sizeof(IP_ADAPTER_INFO);
```
第一次调用GetAdaptersInfo获取适当的ulOutBufLen变量大小
```
DWORD res = GetAdaptersInfo(pAdapterInfo,&ulOutBufLen);
free (pAdapterInfo);
pAdapterInfo = (IP_ADAPTER_INFO *) malloc ( ulOutBufLen );
res = GetAdaptersInfo(pAdapterInfo,&ulOutBufLen);
```
三、获取计算机的DNS设置
采用函数GetNetWorkParams函数可以获得本地计算机的网络参数，从而获得计算机的DNS设置。
DWORD GetNetworkParams(  PFIXED_INFO pFixedInfo,   PULONG pOutBufLen);
MSDN: This function retrieves network parameters for the local computer.
参数pFixedInfo定义为：
```
typedef struct {
  char HostName [MAX_HOSTNAME_LEN + 4];//主机名字
  char DomainName [MAX_DOMAIN_NAME_LEN + 4]; //域名
  PIP_ADDR_STRING CurrentDnsServer; //IP地址的一个节点
  IP_ADDR_STRING DnsServerList; //服务器IP地址链表
  UINT NodeType; //节点类型
  char ScopeId [MAX_SCOPE_ID_LEN + 4]; //
  UINT EnableRouting; //
  UINT EnableProxy; //
  UINT EnableDns; //
} FIXED_INFO, *PFIXED_INFO;
```

获取DNS设置
```
FIXED_INFO *pFixedInfo;
ULONG       ulOutBufLen;
DWORD       dwRetVal;
pFixedInfo = (FIXED_INFO *) malloc( sizeof( FIXED_INFO ) );
ulOutBufLen = sizeof( FIXED_INFO );
if ( GetNetworkParams( pFixedInfo, &ulOutBufLen ) == ERROR_BUFFER_OVERFLOW )
{
	 free( pFixedInfo );
	 pFixedInfo = (FIXED_INFO *) malloc( ulOutBufLen );
}

dwRetVal = GetNetworkParams( pFixedInfo, &ulOutBufLen );
```
四、获取计算机安装的协议,可以通过函数WSAEnumProtocols获取安装在本地主机上可用的网络协议集  <br>
>int WSAEnumProtocols(  LPINT lpiProtocols,  LPWSAPROTOCOL_INFO lpProtocolBuffer,  ILPDWORD lpdwBufferLength);  <br>
MSDN：This function retrieves information about available transport protocols.
lpProtocolBuffer是结构WSAPROTOCOL填充的缓冲区，定义为：
```
typedef struct _WSAPROTOCOL_INFO {
	DWORD dwServiceFlags1;
	DWORD dwServiceFlags2;
	DWORD dwServiceFlags3;
	DWORD dwServiceFlags4;
	DWORD dwProviderFlags;
	GUID ProviderId;
	DWORD dwCatalogEntryId;
	WSAPROTOCOLCHAIN ProtocolChain;
	int iVersion;
	int iAddressFamily;
	int iMaxSockAddr;
	int iMinSockAddr;
	int iSocketType;
	int iProtocol;
	int iProtocolMaxOffset;
	int iNetworkByteOrder;
	int iSecurityScheme;
	DWORD dwMessageSize;
	DWORD dwProviderReserved;
	TCHAR szProtocol[WSAPROTOCOL_LEN+1];
} WSAPROTOCOL_INFO, *LPWSAPROTOCOL_INFO;
```

同时也可以调用函数getprotobyname获取对应于给定协议名的相关协议信息：  <br>
>struct PROTOENT* FAR getprotobyname(const char* name);  <br>
MSDN: The getprotobyname function retrieves the protocol information corresponding to a protocol name.
```
typedef struct protoent {
  char FAR* p_name;//正规协议名
  char FAR  FAR** p_aliases;//一个以空指针结尾的可选协议队列
  short p_proto;//主机字节顺序的协议号
} protoent;
```

还有一个函数根据协议号来获取协议的相关信息：  <br>
>struct PROTOENT* FAR getprotobynumber( int number );

五、获取计算机提供的服务  <br>
可以调用函数getservbyname来获取对应给定服务名和服务协议的相关服务信息。  <br>
>struct servent* FAR getservbyname( onst char* name,  const char* proto );   <br>
MSDN: The getservbyname function retrieves service information corresponding to a service name and protocol.
```
typedef struct servent {
  char FAR* s_name;//正规服务名
  char FAR  FAR** s_aliases; //一个以空指针结尾的可选协议队列
  short s_port;//端口号
  char FAR* s_proto;//连接该服务时所用到的协议名
} servent;
```

也可以调用函数getservbyport来获取对于给定端口号和协议名的相关服务信息： <br>
>struct servent* FAR getservbyport( int port, const char* proto );

六、获取计算机的所有网络资源  <br>
函数WNetOpenEnum开始一个网络资源或存在的网络连接枚举值，可以通过调用函数WNetEnumResource获取详细的网络资源。
```
DWORD WNetOpenEnum(
  DWORD dwScope,
  DWORD dwType,
  DWORD dwUsage,
  LPNETRESOURCE lpNetResource,
  LPHANDLE lphEnum
);
```

上述具体各个参数见MSDN，NETRESOURCE结构体定义如下：
```
typedef struct _NETRESOURCE {
  DWORD dwScope;
  DWORD dwType;
  DWORD dwDisplayType;
  DWORD dwUsage;
  LPTSTR lpLocalName;
  LPTSTR lpRemoteName;
  LPTSTR lpComment;
  LPTSTR lpProvider;
} NETRESOURCE;

WNetEnumResource：This function continues a network-resource enumeration started by the WNetOpenEnum function.

DWORD WNetEnumResource(
  HANDLE hEnum, //这个参数为WNetOpenEnum的最后一个参数
  LPDWORD lpcCount,
  LPVOID lpBuffer,
  LPDWORD lpBufferSize
);
```
七、修改本地网络设置
修改本地已经存在的地址解析协议表：SetIpNetEntry:   <br>
>DWORD SetIpNetEntry( PMIB_IPNETROW pArpEntry );

l 设置某一网络接口卡的管理状态，SetIfentry：  <br>
>DWORD SetIfEntry( PMIB_IFROW pIfRow );

l 为本地计算机特定的适配器添加地址，AddIpAddress:
```
DWORD AddIPAddress(
  IPAddr Address,
  IPMask IpMask,
  DWORD IfIndex,
  PULONG NTEContext,
  PULONG NTEInstance
);
```
l 删除IP地址，DeleteIpAddress:
>DWORD DeleteIPAddress( ULONG NTEContext );

八、获取计算机TCP/IP的所有信息
通过函数GetTcpStatics获取本地的TCP协议统计信息：
>DWORD GetTcpStatistics( PMIB_TCPSTATS pStats ); 参数pStats是指向MIB_TCPSTATS的指针，定义如下：
```
typedef struct _MIB_TCPSTATS {
  DWORD dwRtoAlgorithm;
  DWORD dwRtoMin;
  DWORD dwRtoMax;
  DWORD dwMaxConn;
  DWORD dwActiveOpens;
  DWORD dwPassiveOpens;
  DWORD dwAttemptFails;
  DWORD dwEstabResets;
  DWORD dwCurrEstab;
  DWORD dwInSegs;
  DWORD dwOutSegs;
  DWORD dwRetransSegs;
  DWORD dwInErrs;
  DWORD dwOutRsts;
  DWORD dwNumConns;
} MIB_TCPSTATS, *PMIB_TCPSTATS;
```

函数GetTcpTable获取TCP协议连接表：
>DWORD GetTcpTable(PMIB_TCPTABLE pTcpTable, PDWORD pdwSize, BOOL bOrder );


函数GetIpAddrTable获取网络接口卡与IP地址的映射表：
>DWORD GetIpAddrTable( PMIB_IPADDRTABLE pIpAddrTable,PULONG pdwSize,BOOL bOrder );

GetIpStatics获取当前IP统计信息：
>DWORD GetIpStatistics( PMIB_IPSTATS pStats );
```
typedef struct _MIB_IPSTATS {
  DWORD dwForwarding;
  DWORD dwDefaultTTL;
  DWORD dwInReceives;
  DWORD dwInHdrErrors;
  DWORD dwInAddrErrors;
  DWORD dwForwDatagrams;
  DWORD dwInUnknownProtos;
  DWORD dwInDiscards;
  DWORD dwInDelivers;
  DWORD dwOutRequests;
  DWORD dwRoutingDiscards;
  DWORD dwOutDiscards;
  DWORD dwOutNoRoutes;
  DWORD dwReasmTimeout;
  DWORD dwReasmReqds;
  DWORD dwReasmOks;
  DWORD dwReasmFails;
  DWORD dwFragOks;
  DWORD dwFragFails;
  DWORD dwFragCreates;
  DWORD dwNumIf;
  DWORD dwNumAddr;
  DWORD dwNumRoutes;
} MIB_IPSTATS, *PMIB_IPSTATS;
```
