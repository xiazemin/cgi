# 环境变量

  对于CGI程序来说，它继承了系统的环境变量。CGI环境变量在CGI程序启动时初始化，在结束时销毁。



       当一个CGI程序不是被HTTP服务器调用时，它的环境变量几乎是系统环境变量的复制。



当这个CGI程序被HTTP服务器调用时，它的环境变量就会多了以下关于HTTP服务器、客户端、CGI传输过程等项目。



 



与请求相关的环境变量



REQUEST\_METHOD



服务器与CGI程序之间的信息传输方式



QUERY\_STRING



采用GET时所传输的信息



CONTENT\_LENGTH



STDIO中的有效信息长度



CONTENT\_TYPE



指示所传来的信息的MIME类型



CONTENT\_FILE



使用Windows HTTPd/WinCGI标准时，用来传送数据的文件名



PATH\_INFO



路径信息



PATH\_TRANSLATED



CGI程序的完整路径名



SCRIPT\_NAME



所调用的CGI程序的名字



与服务器相关的环境变量



GATEWAY\_INTERFACE



服务器所实现的CGI版本



SERVER\_NAME



服务器的IP或名字



SERVER\_PORT



主机的端口号



SERVER\_SOFTWARE



调用CGI程序的HTTP服务器的名称和版本号



与客户端相关的环境变量



REMOTE\_ADDR



客户机的主机名



REMOTE\_HOST



客户机的IP地址



ACCEPT



例出能被次请求接受的应答方式



ACCEPT\_ENCODING



列出客户机支持的编码方式



ACCEPT\_LANGUAGE



表明客户机可接受语言的ISO代码



AUTORIZATION



表明被证实了的用户



FORM



列出客户机的EMAIL地址



IF\_MODIFIED\_SINGCE



当用get方式请求并且只有当文档比指定日期更早时才返回数据



PRAGMA



设定将来要用到的服务器代理



REFFERER



指出连接到当前文档的文档的URL



USER\_AGENT



客户端浏览器的信息



       CONTENT\_TYPE:如application/x-www-form-urlencoded，表示数据来自HTML表单，并且经过了URL编码。



ACCEPT:客户机所支持的MIME类型清单，内容如：”image/gif,image/jpeg”



REQUEST\_METHOD：它的值一般包括两种:POST和GET，但我们写CGI程序时，最后还要考虑其他的情况。



1．POST方法

如果采用POST方法，那么客户端来的用户数据将存放在CGI进程的标准输入中，同时将用户数据的长度赋予环境变量中的CONTENT\_LENGTH。客户端用POST方式发送数据有一个相应的MIME类型（通用Internet邮件扩充服务：Multi-purpose Internet Mail Extensions）。目前，MIME类型一般是：application/x-wwww-form-urlencoded，该类型表示数据来自HTML表单。该类型记录在环境变量CONTENT\_TYPE中，CGI程序应该检查该变量的值。



2．GET方法

在该方法下，CGI程序无法直接从服务器的标准输入中获取数据，因为服务器把它从标



准输入接收到得数据编码到环境变量QUERY\_STRING（或PATH\_INFO）。



GET与POST的区别：采用GET方法提交HTML表单数据的时候，客户机将把这些数



据附加到由ACTION标记命名的URL的末尾，用一个包括把经过URL编码后的信息与CGI程序的名字分开：http://www.mycorp.com/hello.html？name=hgq$id=1，QUERY\_STRING的值为name=hgq&id=1



有些程序员不愿意采用GET方法，因为在他们看来，把动态信息附加在URL的末尾有



违URL的出发点：URL作为一种标准用语，一般是用作网络资源的唯一定位标示。



 



环境变量是一个保存用户信息的内存区。当客户端的用户通过浏览器发出CGI请求时，服务器就寻找本地的相应CGI程序并执行它。在执行CGI程序的同时，服务器把该用户的信息保存到环境变量里。接下来，CGI程序的执行流程是这样的：查询与该CGI程序进程相应的环境变量：第一步是request\_method，如果是POST，就从环境变量的len，然后到该进程相应的标准输入取出len长的数据。如果是GET，则用户数据就在环境变量的QUERY\_STRING里。



3．POST与GET的区别

       以 GET 方式接收的数据是有长度限制，而用 POST 方式接收的数据是没有长度限制的。并且，以 GET 方式发送数据，可以通过URL 的形式来发送，但 POST方式发送的数据必须要通过 Form 才到发送

