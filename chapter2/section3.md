# FastCGI协议详解及代码实现

FastCGI程序本身监听某个socket然后等待来自web服务器的连接，而不是像CGI程序是由web服务器 fork-exec，所以FastCGI本身是一个服务端程序，而web服务器对它来说则是客户端。

FastCGI程序和web服务器之间通过可靠的流式传输（Unix Domain Socket或TCP）来通信，相对于传统的CGI程序，有环境变量和标准输入输出，而FastCGI程序和web服务器之间则只有一条socket连接来传输数据，所以它把数据分成以下多种消息类型

由web服务器向FastCGI程序传输的消息类型有以下几种：



FCGI\_BEGIN\_REQUEST  表示一个请求的开始，



FCGI\_ABORT\_REQUEST  表示服务器希望终止一个请求



FCGI\_PARAMS     对应于CGI程序的环境变量，php $\_SERVER 数组中的数据绝大多数来自于此



FCGI\_STDIN    对应CGI程序的标准输入，FastCGI程序从此消息获取 http请求的POST数据



此外 FCGI\_DATA  和 FCGI\_GET\_VALUES 这里不做介绍。







由FastCGI程序返回给web服务器的消息类型有以下几种：



FCGI\_STDOUT    对应CGI程序的标准输出，web服务器会把此消息当作html返回给浏览器



FCGI\_STDERR    对应CGI程序的标准错误输出， web服务器会把此消息记录到错误日志中



FCGI\_END\_REQUEST  表示该请求处理完毕



FCGI\_UNKNOWN\_TYPE  FastCGI程序无法解析该消息类型



此外还有 FCGI\_GET\_VALUES\_RESULT 这里不做介绍







web服务器和FastCGI程序每传输一个消息，首先会传输一个8字节固定长度的消息头，这个消息头记录了随后要传输的这个消息的 类型，长度等等属性，消息头的结构体如下：





struct FCGI\_Header {

    unsigned char version;

    unsigned char type;

    unsigned char requestIdB1;

    unsigned char requestIdB0;

    unsigned char contentLengthB1;

    unsigned char contentLengthB0;

    unsigned char paddingLength;

    unsigned char reserved;

} ;



version  表示fastcgi协议版本

type  表示消息的类型，就是前面提到的多种消息类型之一，如 FCGI\_BEGIN\_REQUEST、FCGI\_PARAMS 等等



requestIdB1 和 requestIdB0 这两个字节组合来表示 requestId \(对于每个请求web服务器会分配一个requestId\)， requestIdB1 是requestId的高八位，requestIdB0是低八位，所以最终的 requestId = （requestIdB1 &lt;&lt; 8） + requestIdB0，因为是两个字节来表示，requestId最大取值为65535， 同理 contentLengthB1 和 contentLengthB0 共同来表示消息体的长度，对于超过65535的消息体，可以切割成多个消息体来传输



paddingLength   为了使消息8字节对齐，提高传输效率，可以在消息上添加一些字节数来达到消息对齐的目的，paddingLength 为添加的字节数，这些字节是无用数据，读出来可以直接丢弃。



reserved  保留字段，暂时无用







比如在传输 FCGI\_BEGIN\_REQUEST 消息之前，首先会传输一个消息头类似如下：



0x0000080001000101  粗体 08 对应 requestIdB0 ，粗体00 对应 requestIdB1 ，所以后续要传输的这个消息的长度是八字节，粗体01指代该消息类型为 FCGI\_BEGIN\_REQUEST







对于 FCGI\_BEGIN\_REQUEST 和  FCGI\_END\_REQUEST 消息类型，fastcgi协议分别定义了一个结构体如下，而对于其他类型的消息体，没有专门结构体与之对应，消息体就是普通的二进制数据。





struct FCGI\_BeginRequestBody {

    unsigned char roleB1;

    unsigned char roleB0;

    unsigned char flags;

    unsigned char reserved\[5\];

} ;

 

struct  FCGI\_EndRequestBody {

    unsigned char appStatusB3;

    unsigned char appStatusB2;

    unsigned char appStatusB1;

    unsigned char appStatusB0;

    unsigned char protocolStatus;

    unsigned char reserved\[3\];

};



从这个结构体可以知道 FCGI\_BEGIN\_REQUEST 和  FCGI\_END\_REQUEST 消息体的长度都是固定的8个字节。

FCGI\_BeginRequestBody 的 roleB1 和 roleB0 两个字节组合指代 web服务器希望FastCGI程序充当的角色，目前FastCGI协议仅定义了三种角色：





\#define FCGI\_RESPONDER  1

\#define FCGI\_AUTHORIZER 2

\#define FCGI\_FILTER     3



常见的FastCGI程序基本都是作为 FCGI\_RESPONDER （响应器角色），所以roleB1的值总是0， roleB0的值可取1~3三个值，但 常见都是1，其他两种角色这里不做讨论。



flags 是一个8位掩码， web服务器可以利用该掩码告知FastCGI程序在处理完一个请求后是否关闭socket连接 （最初协议的设计者可能还预留了该掩码的其他作用，只是目前只有这一个作用\)



flags & FCGI\_KEEP\_CONN  的值为1，则FastCGI程序请求结束不关闭连接，为0 则关键连接



其中 FCGI\_KEEP\_CONN 是一个宏，定义如下：





\#define FCGI\_KEEP\_CONN  1



以下是协议对于响应器角色的解释：



Responder FastCGI应用程序具有与CGI / 1.1程序相同的用途：它接收与HTTP请求相关的所有信息并生成HTTP响应。



以下解释CGI / 1.1的每个元素和响应者角色的消息类型的对应关系：



Responder应用程序通过FCGI\_PARAMS从Web服务器接收CGI / 1.1环境变量。

接下来，Responder应用程序通过FCGI\_STDIN从Web服务器接收CGI / 1.1 stdin数据。在接收流结束指示之前，应用程序最多从此流接收CONTENT\_LENGTH个字节。（仅当HTTP客户端无法提供时，应用程序才会收到少于CONTENT\_LENGTH的字节，例如，因为客户端崩溃了。）

响应者应用程序通过FCGI\_STDOUT发送CGI / 1.1 标准输出数据到Web服务器，通过FCGI\_STDERR发送CGI / 1.1 标准错误数据。应用程序并发地发送这些，而不是一个接一个地发送。应用程序必须等待读完FCGI\_PARAMS数据之后才可以写FCGI\_STDOUT和FCGI\_STDERR，但它不需要等待读完FCGI\_STDIN才可以写这两个流。

发送所有stdout和stderr数据后，Responder应用程序将发送FCGI\_END\_REQUEST记录。应用程序将protocolStatus组件设置为FCGI\_REQUEST\_COMPLETE，将appStatus组件设置为CGI程序通过退出系统调用返回的状态代码。

处理POST请求的响应方应比较FCGI\_STDIN上接收到的字节数和CONTENT\_LENGTH，如果两个数不相等，则中止请求。



FCGI\_EndRequestBody 中 appStatus是应用级别的状态码。每个角色记录其对appStatus的使用情况，不做深入讨论

所述的protocolStatus 是协议级别的状态码，可能的protocolStatus值是：





\#define FCGI\_REQUEST\_COMPLETE 0

\#define FCGI\_CANT\_MPX\_CONN    1

\#define FCGI\_OVERLOADED       2

\#define FCGI\_UNKNOWN\_ROLE     3



FCGI\_REQUEST\_COMPLETE：请求的正常结束，典型的应该都是该个值。

FCGI\_CANT\_MPX\_CONN：拒绝新的请求。当Web服务器通过一个连接将并发请求发送到每个连接一次处理一个请求的应用程序时，会发生这种情况。

FCGI\_OVERLOADED：拒绝新的请求。当应用程序耗尽某些资源（例如数据库连接）时会发生这种情况。

FCGI\_UNKNOWN\_ROLE：拒绝新的请求。当Web服务器指定了应用程序未知的角色时，会发生这种情况。







所以，当FastCGI程序从web服务器读取数据时，总是先读取一个8字节的消息头，然后得到消息的类型和长度信息，然后再读取消息体，一种消息过长可以切割成多个消息传输，当一个消息头里的 contentLength 为0（也即 contentLengthB1和contentLengthB0 的值都为0） 时，则表明这种消息传输完毕，然后我们可以把之前读到的这种类型的多个消息合并得到最终完整的消息。反之，当我们要从FastCGI程序向web服务器返回数据时，总是每发送一个8字节消息头，紧接发送一次消息体，循环往复，直到最后发送 FCGI\_END\_REQUEST类型的消息头 和消息体结束请求。







总结一下web服务器和FastCGi程序之间大概的消息发送流程：



1、web服务器向FastCGI程序发送一个 8 字节 type=FCGI\_BEGIN\_REQUEST的消息头和一个8字节  FCGI\_BeginRequestBody 结构的 消息体，标志一个新请求的开始



2、web服务器向FastCGI程序发送一个 8 字节 type=FCGI\_PARAMS 的消息头 和一个消息头中指定长度的FCGI\_PARAMS类型消息体



3、根据FCGI\_PARAMS消息的长度可能重复步骤 2 多次，最终发送一个 8 字节 type=FCGI\_PARAMS 并且 contentLengthB1 和 contentLengthB0 都为 0 的消息头 标志 FCGI\_PARAMS 消息发送结束



4、以和步骤2、3相同的方式 发送 FCGI\_STDIN 消息



5、FastCGI程序处理完请求后 以和步骤2、3相同的方式 发送 FCGI\_STDOUT消息 和 FCGI\_STDERR 消息返回给服务器



6、FastCGI程序 发送一个 type= FCGI\_END\_REQUEST  的消息头 和  一个8字节 FCGI\_EndRequestBody 结构的消息体，标志此次请求结束







FastCGI协议的完整规范请查看： https://fastcgi-archives.github.io/FastCGI\_Specification.html







下面用c 编写一个简单的实现fastcgi协议的demo，这个demo主要是用来向大家更直观的展示FastCGI协议，错误处理和内存泄漏检测都不到位，专业的协议实现可以看官方提供的 Fastcgi Developer‘s kit ： https://fastcgi-archives.github.io/FastCGI\_Developers\_Kit\_FastCGI.html ，其中封装的c库在 libfcgi 文件中。







完整代码托管在github上： https://github.com/zhyee/fastcgi-demo

