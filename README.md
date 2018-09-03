# Introduction

https://datatracker.ietf.org/doc/rfc3875/

作为一个服务器，基本要求是能受理请求，提取信息并将消息分发给 CGI 解释器，再将解释器响应的消息包装后返回客户端。在这个过程中，除了和客户端 socket 之间的交互，还要牵扯到第三个实体 - 请求解释器。

![](/assets/cgi.png)

如上图所示，客户端负责封装请求和解析响应，服务器的主要职责是管理连接、数据转换、传输和分发客户端请求，而真正进行数据文档处理与数据库操作的就是请求解释器，这个解释器，在 PHP 中一般是 PHP-FPM，JAVA 中是 Servlet。

我们之前进行的处理多在客户端和服务器之间的通信，以及服务器的内部调整，这次更新的内容主要是后面两个实体之间的进程间通信。

进程间通信牵涉到三个方面，即方式和形式和内容。

方式指的是进程间通信的传输媒介，如 Nginx 中实现的 TCP 方式和 Unix Domain Socket，它们分别有跨机器和高效率的优点，还有我实现的服务器用了很 low 的popen方式。

而形式就是数据格式了，我认为它并无定式，只要服务器容易组织数据，解释器能方便地接收并解析，最好也能节约传输资源，提高传输效率。目前的解决方案有经典的 xml，轻巧易理解的 json 和谷歌高效率的 protobuf。它们各有优点，我选择了 json，主要是因为有CJson库的存在，数据在 C 中方便组织，而在PHP中，一个json\_decode\(\)方法就完成了数据解析。

至于应该传输哪些内容呢？CGI 描述了一套协议：

CGI

通用网关接口（Common Gateway Interface/CGI）是一种重要的互联网技术，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。CGI描述了服务器和请求处理程序之间传输数据的一种标准。

CGI 是服务器与解释器交互的接口，服务器负责受理请求，并将请求信息解释为一条条基本的请求信息（在文档中被称为“元数据”），传送给解释器来解释执行，而解释器响应文档和数据库操作信息。

之前看了一下 CGI 的 RFC 文档，总结了几个重要点，有兴趣的可以看下底部参考文献。常见规范（信息太多，只考虑 MUST 的情况）如下：

CGI请求

服务器根据 以 / 分隔的路径选择解释器；

如果有 AUTH 字段，需要先执行 AUTH，再执行解释器; 

服务器确认 CONTENT-LENGTH 表示的是数据解析出来的长度，如果附带信息体，则必须将长度字段传送到解释器；

如果有 CONTENT-TYPE 字段，服务器必须将其传给解释器；若无此字段，但有信息体，则服务器判断此类型或抛弃信息体；

服务器必须设置 QUERY\_STRING 字段，如果客户端没有设置，服务端要传一个空字符串“”

服务器必须设置 REMOTE\_ADDR，即客户端请求IP；

REQUEST\_METHOD 字段必须设置， GET POST 等，大小写敏感；

SCRIPT\_NAME 表示执行的解释器脚本名，必须设置；

SERVER\_NAME 和 SERVER\_PORT 代表着大小写敏感的服务器名和服务器受理时的TCP/IP端口；

SERVER\_PROTOCOL 字段指示着服务器与解释器协商的协议类型，不一定与客户端请求的SCHEMA 相同，如'\[[https://'可能为HTTP；\]\(https://'可能为HTTP；](https://'可能为HTTP；]%28https://'可能为HTTP；)\)

在 CONTENT-LENGTH 不为 NULL 时，服务器要提供信息体，此信息体要严格与长度相符，即使有更多的可读信息也不能多传；

服务器必须将数据压缩等编码解析出来；

CGI响应

CGI解释器必须响应 至少一行头 + 换行 + 响应内容；

解释器在响应文档时，必须要有 CONTENT-TYPE 头；

在客户端重定向时，解释器除了 client-redir-response=绝对url地址，不能再有其他返回，然后服务器返回一个 302 状态码；

解释器响应 三位数字状态码，具体配置可自行搜索；

服务器必须将所有解释器返回的数据响应给客户端，除非需要压缩等编码，服务器不能修改响应数据；

Nginx和PHP的CGI实现

介绍完了 CGI，我们来参考一下当前服务器 CGI 协议实现的成熟方案，这里挑选我熟悉的 Nginx 和 PHP。

在 Nginx 和 PHP 的配合中，Nginx 自然是服务器，而解释器是 PHP 的 SAPI。

SAPI

SAPI: Server abstraction API，指的是 PHP 具体应用的编程接口，它使得 PHP 可以和其他应用进行交互数据。

PHP 脚本要执行可以通过很多种方式，通过 Web 服务器，或者直接在命令行下，也可以嵌入在其他程序中。常见的 sapi 有apache2handler、fpm-fcgi、cli、cgi-fcgi，可以通过 PHP 函数php\_sapi\_name\(\)来查看当前 PHP 执行所使用的 sapi。

PHP5.3 之前使用的与服务器交互的 sapi 是cgi，它实现基本的 CGI 协议，由于它每次处理请求都要创建一个进程、初始化进程、处理请求、销毁进程，消耗过大，使得系统性能大大下降。

这时候便出现了 CGI 协议的升级版本 Fast-CGI。

PHP-FPM

快速通用网关接口（Fast Common Gateway Interface／FastCGI）是一种让交互程序与Web服务器通信的协议。FastCGI是早期通用网关接口（CGI）的增强版本。

Fast-CGI 提升效率主要靠将 CGI 解释器长驻内存重现，避免了进程反复加载的损耗。PHP 的 sapi cgi-fcgi实现了 Fast-CGI 协议，提升了 PHP 处理 Web 请求的效率。

那么我们常见的 php-fpm 是什么呢？它是一种进程管理器（PHP-FastCGI Process Manager），它负责管理实现 Fast-CGI 的那些进程（worker进程），它加载php.ini信息，初始化 worker 进程，并实现平滑重启和其他高级功能。

Nginx 将请求都交给 php-fpm，fpm 选择一个空闲工作进程来处理请求。

纠偏

这里总结一下几个名字，以防混淆：

sapi，是 PHP 与外部进程交互的接口；

CGI/Fast-CGI（大写）是一种协议；

本节中出现的 cgi（小写），是指 PHP 的 sapi，即实现 CGI 协议的一种接口。

php-fpm 是管理实现了Fast-CGI协议的进程的一个进程。

代码实现

介绍完了高端的Nginx服务器，说一下我的实现：

服务器解析 http 报文，实现 CGI 协议，将数据包装成 json 格式，通过 PHP 的cli sapi 发送至 PHP 进程，PHP 进程解析后响应 json 格式数据，服务器解析响应数据后包装成 http 响应报文发送给客户端。

http\_parser

首要任务是解析 http 报文，C 中没有很丰富字符串函数，我也没有封装过常用的函数库，所以只好临时自己实现了一个util\_http.c，这里介绍几个处理 http 报文时好用的字符串函数。

strtok\(char str\[\], const \*delimeter\)，将 delimeter 设置为 "\n"，分行处理 http 报文头正好适合。

sscanf\(const \*str, format, dest1\[,dest...\]\)，它从字符串中以特定格式读取字符串，读取时的分隔符是空格，用它来处理 http 请求行十分方便。

至于解析 http 报文头的键值对应，没想到好方法，只好使用字符遍历来判断。

cJSON

cJSON 是一个 C 实现的用以生成和解析 json 格式数据的函数库，在 GitHub 上可以轻松搜到，只用两个文件 cJSON.c和cJSON.h即可。

需要注意：C 作为强类型语言，往 json 内添加不同类型的数据要使用不同的方法，cJSON 支持 string, bool, number, cJSON object等类型。

这里简单地介绍一下生成和解析的一般方法；

生成：

cJSON \*root; // 声明cJSON格式数据

root = cJSON\_CreateObject\(\); // 创建一个cJSON对象

cJSON\_AddStringToObject\(root, "key", "value"\) // 往cJSON对象内添加键值对

char \*output = cJSON\_PrintUnformatted\(root\); // 生成json字符串

cJSON\_Delete\(root\); // 别忘记释放内存

解析：

cJSON \*json = cJSON\_Parse\(response\_json\);

value = cJSON\_GetObjectItem\(cJSON, "key"\);

当然，也可以声明 cJSON 类型的数据进行嵌套；

