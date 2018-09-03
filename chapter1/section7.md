# CGI数据输出

CGI程序如何将信息处理结果返回给客户端？这实际上是CGI格式化输出。



在CGI程序中的标准输出stdout是经过重定义了的，它并没有在服务器上产生任何的输出内容，而是被重定向到客户浏览器，这与它是由C，还是Perl或Python实现无关。



所以，我们可以用打印来实现客户端新的HTML页面的生成。比如，C的printf是向该进程的标准输出发送数据，Perl和Python用print向该进程的标准输出发送数据。



（1）   CGI标题



CGI的格式输出内容必须组织成标题/内容的形式。CGI标准规定了CGI程序可以使用



的三个HTTP标题。标题必须占据第一行输出！而且必须随后带有一个空行。



标题



描述



Content\_type   \(内容类型\)



设定随后输出数据所用的MIME类型



Location    \(地址\)



设定输出为另外一个文档（URL）



Status      \(状态\)



指定HTTP状态码



 



MIME：



向标准输出发送网页内容时要遵守MIME格式规则：



任意输出前面必须有一个用于定义MIME类型的输出内容（Content-type）行，而且随后还必须跟一个空行。如果遗漏了这一条，服务将会返回一个错误信息。（同样使用于其他标题）



例如Perl和Python：



print “Content-type:text/html\n\n”;   //输出HTML格式的数据



print “&lt;body&gt;welcome&lt;br&gt;”



print “&lt;/body&gt;”



C语言：



printf\( “Content-type:text/html\n\n”\);



printf\(“Welcome\n”\);



 



MIME类型以类型/子类型（type/subtype）的形式表示。



其中type表示一下几种典型文件格式的一种：



Text、Audio、Video、Image、Application、Mutipart、Message



Subtype则用来描述具体所用的数据格式。



Application/msword



微软的Word文件



Application/octet-stream



一种通用的二进制文件格式



Application/zip



Zip压缩文件



Application/pdf



Pdf文件



。。。。。。。。。。。。。。。。。。。。。。。。。。



。。。。。。。。。。。。。。。。。。。。。。。。。



 



Location：



使用Location标题，一个CGI可以使当前用户转而访问同一服务器上的另外一个程序，甚至可以访问另外一个URL，但服务器对他们的处理方式不一样。



使用Location的格式为：Location：Filename/URL，例如：



print “Location:/test.html\n\n”;



这与直接链接到test.html的效果是一样的。



 



print “Location:http://www.chinaunix.com/\n\n”



由于该URL并不指向当前服务器，用户浏览器并不会直接链接到指定的URL，而是给用户输出提示信息。



 



 



HTTP状态码：



       表示了请求的结果状态，是CGI程序通过服务器用来通知用户其请求是否成功执行的信息码，本文不做研究。

