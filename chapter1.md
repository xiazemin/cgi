# CGI接口原理及实现

1.CGI定义：



　　CGI\(CommonGateway Interface\)是HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。



　　2.CGI功能：



　　绝大多数的CGI程序被用来解释处理来自表单的输入信息，并在服务器产生相应的处理，或将相应的信息反馈给浏览器。CGI程序使网页具有交互功能。



　　3.CGI运行环境：



　　CGI程序在UNIX操作系统上CERN或NCSA格式的服务器上运行。 在其它操作系统（如：windows NT及windows95等）的服务器上 也广泛地使用CGI程序，同时它也适用于各种类型机器。



　　4.CGI处理步骤：



　　⑴通过Internet把用户请求送到服务器。



　　⑵服务器接收用户请求并交给CGI程序处理。



　　⑶CGI程序把处理结果传送给服务器。



　　⑷服务器把结果送回到用户。



　　5.CGI服务器配置：



　　在许多服务器cgi-bin是仅能够放置CGI脚本的目录。



　　791x212



　　在Windows平台上将C或C++写好的程序的Debug或Release版本的.exe程序拷贝到cgi-bin的目录下（如上图所示），将.exe改为.cgi也可同样运行，如下2个图。



　　813x184



　　817x165



　　       cgi-bin目录是存放CGI脚本的地方。这些脚本使WWW服务器和浏览器能运行外部程序，而无需启动另一个程序。它是运行在Web服务器上的一个程序，并由来自于浏览者的输入触发。



　　CGI程序不是放在服务器上就能顺利运行，如果要想使其在服务器上顺利的运行并准确的处理用户的请求，则须对所使用的服务器进行必要的设置。



　　       配置：根据所使用的服务器类型以及它的设置把CGI程序放在某一特定的目录中或使其带有特定的扩展名。



　　Apache网络服务器配置在/var/www/cgi-bin里（如下图所示笔者电脑的目录位置）。C++编译的可执行文件可以转换成扩展名为.cgi的文件。



　　更改初始配置的的方法：



　　&lt;Directory"/var/www/cgi-bin"&gt;



　　AllowOverride None



　　Options ExecCGI



　　Order allow,deny



　　Allow from all



　　&lt;/Directory&gt;



　　&lt;Directory"/var/www/cgi-bin"&gt;



　　Options All



　　&lt;/Directory&gt;



　　636x298



　　6.CGI接口标准包括标准输入、环境变量、标准输出三部分。



 	

　　介绍



　　1.标准输入



　　CGI程序像其他可执行程序一样,可通过标准输入\(stdin\)从Web服务器得到输入信息,如Form中的数据,这就是所谓的向CGI程序传递数据的POST方法。这意味着在操作系统命令行状态可执行CGI程序,对CGI程序进行调试。POST方法是常用的方法。



　　2.环境变量



　　操作系统提供了许多环境变量,它们定义了程序的执行环境,应用程序可以存取它们。Web服务器和CGI接口又另外设置了自己的一些环境变量,用来向CGI程序传递一些重要的参数。CGI的GET方法还通过环境变量QUERY-STRING向CGI程序传递Form中的数据。



　　3.标准输出



　　CGI程序通过标准输出\(stdout\)将输出信息传送给Web服务器。传送给Web服务器的信息可以用各种格式,通常是以纯文本或者HTML文本的形式,这样我们就可以在命令行状态调试CGI程序,并且得到它们的输出。



　　7.环境变量



　　环境变量是文本串\(名字/值对\),可以被OSShell或其他程序设置 ,也可以被其他程序访问。它们是Web服务器传递数据给CGI程序的简单手段,之所以称为环境变量是因为它们是全局变量,任何程序都可以存取它们。



　　下面是CGI程序设计中常常要用到的一些环境变量。



　　环境变量         



　　意义



　　SERVER\_NAME



　　CGI脚本运行时的主机名和IP地址.



　　SERVER\_SOFTWARE



　　你的服务器的类型如： CERN/3.0 或 NCSA/1.3.



　　GATEWAY\_INTERFACE



　　运行的CGI版本. 对于UNIX服务器, 这是CGI/1.1.



　　SERVER\_PROTOCOL



　　服务器运行的HTTP协议. 这里当是HTTP/1.0.



　　SERVER\_PORT



　　服务器运行的TCP口，通常Web服务器是80.



　　REQUEST\_METHOD



　　POST 或 GET, 取决于你的表单是怎样递交的.



　　HTTP\_ACCEPT 



　　浏览器能直接接收的Content-types, 可以有HTTP Accept header定义.



　　HTTP\_USER\_AGENT



　　递交表单的浏览器的名称、版本 和其他平台性的附加信息。



　　HTTP\_REFERER



　　递交表单的文本的 URL，不是所有的浏览器都发出这个信息，不要依赖它



　　PATH\_INFO



　　附加的路径信息, 由浏览器通过GET方法发出.



　　PATH\_TRANSLATED



　　在PATH\_INFO中系统规定的路径信息.



　　SCRIPT\_NAME



　　指向这个CGI脚本的路径, 是在URL中显示的\(如, /cgi-bin/thescript\).



　　QUERY\_STRING



　　脚本参数或者表单输入项\(如果是用GET递交\). QUERY\_STRING 包含URL中问号后面的参数.



　　REMOTE\_HOST



　　递交脚本的主机名，这个值不能被设置.



　　REMOTE\_ADDR



　　递交脚本的主机IP地址.



　　REMOTE\_USER



　　递交脚本的用户名. 如果服务器的authentication被激活，这个值可以设置。



　　REMOTE\_IDENT



　　如果Web服务器是在ident \(一种确认用户连接你的协议\)运行, 递交表单的系统也在运行ident, 这个变量就含有ident返回值.



　　CONTENT\_TYPE



　　如果表单是用POST递交, 这个值将是 application/x-www-form-urlencoded. 在上载文件的表单中, content-type 是个 multipart/form-data.



　　CONTENT\_LENGTH



　　对于用POST递交的表单, 标准输入口的字节数.



　　REQUEST-METHOD:指的是当Web服务器传递数据给CGI程序时所采用的方法,分为GET和POST两种方法。



　　:GET方法仅通过环境变量\(如QUERY-STRING\)传递数据给CGI程序,而POST方法通过环境变量和标准输入传递数据给CGI程序,因此POST方法可较方便地传递较多的数据给CGI程序。



 	 	

　　问题



　　GET方法



　　通过在URL中嵌入的形式传递参数。对CGI程序而言，在GET method中传递的参数要通过化境变量“QUERY-STRING”来接收。



　　1）  参数的内容作为URL信息，用户可以看到；



　　2）  有大小的限制。



　　POST方法



　　CGI程序从标准输入接收参数。与GET方法不同的是，参数的内容从URL信息中不能获得，对于大小也没有限制。



　　与GET方法问题1\),2\)完全相反。



　　CONTENT-LENGTH:传递给CGI程序的数据字符数\(字节数\)。



　　在C语言程序中,要访向环境变量,可使用getenv\(\)库函数。例如:

       if \(getenv \(″CONTENT-LENGTH″\)\)n=atoi\(getenv\(″CONTENT-LENGTH″\)\);

　　请注意程序中最好调用两次getenv\(\):第一次检查是否存在该环境变量,第二次再使用该环境变量。这是因为函数getenv\(\)在给定的环境变量名不存在时,返回一个NULL\(空\)指针,如果你不首先检查而直接引用它,当该环境变量不存在时会引起CGI程序崩溃。



　　8. CGI的工作原理



　　CGI是一个WEB服务器提供信息服务的标准接口，通过这样一个接口，WEB服务器能够执行程序，并将程序输出的信息返回给浏览器。因为在WEB网上的数据都是静态的，通过CGI程序能够动态的处理浏览者的请求，如保存用户输入的信息，根据用户信息返回相关的资料等等。当客户端发送一个CGI请求给WEB服务器后，WEB服务器将根据CGI程序的类型决定数据向CGI程序的传送方式，一般来讲是通过标准输入/输出流和环境变量来与CGI程序间传递数据。



　　497x283



　　CGI输入输出原理



　　CGI的输入/输出方法：CGI程序通过标准输入（STDIN）和标准输出（STDOUT）来进行输入输出，STDIN和STDOUT是两个预先定义好的文件指针。你可以利用文件读写函数来对其进行操纵。



　　此外CGI程序还通过环境变量来得到输入，只不过环境变量中提供的是一些常用的信息，并且通常不包括用户在WEB页面中输入的信息（除使用下面讲的GET方法时，通过检查环境变量QUERY\_STRING来得到输入数据），而STDIN通常用来传递用户输入的信息。



　　在输入时所使用的POST/GET方法：在WEB页面向CGI发送数据时通常采用两种方法：GET/POST，GET方法将数据附加在URL后发送，如：/cgi/a\_cgi\_test.exe your\_data，CGI程序通过检查环境变量QUERY\_STRING来得到输入数据。

