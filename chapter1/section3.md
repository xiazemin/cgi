# section3

而POST方法则会将数据送入CGI程序的STDIN输入流。在表单（FORM）中的各个变量都会成为name=value的形式向WEB服务器发送，多个数据间用&分隔，如：name=value&name2=value2。其中名字（name，name2）是Form中定义的INPUT、SELECT或TEXTAREA等标置\(Tag\)名字,值是用户输入或选择的标置值。



　　如上面说讲，在CGI程序输出时必须先输出一个CGI标题，标题共有以下三类：



　　·      Location: 标题，指明输出另一个文档的URL，例如 fprintf\(stdout,"Location: \n\n"\);



　　·      Content-Type: 标题，指明发送的数据的MIME类型，例如 fprintf\(stdout,"Content-Type:text/html\n\n"\);



　　·      Status: 标题，指明HTTP状态码，例如 fprintf\(stdout,"Status: 200\n\n"\);



　　注意每种标题后都必须跟一个换行和一个空行。



　　MIME类型以类型/子类型的形式来表示，下面是一些常用的类型/子类型的组合：



　　·      Text/plain 普通文本类型



　　·      Text/html HTML格式的文本类型



　　·      Audio/basic 八位声音文件格式，后缀为.au



　　·      Video/mpeg MPEG文件格式



　　·      Video/quicktime QuickTime文件格式



　　·      Image/gif GIF图形文件



　　·      Image/jpeg JPEG图形文件



　　·      Image/x-xbitmap X bitmap图形文件，后缀为.xbm



　　有了上面的知识我们就可以写出一些CGI程序，首先需要对输入数据进行分析，方法为：每当找到字符=,标志着一个Form变量名字的结束;每当找到字符& ,标志着一个Form变量值的结束。请注意输入数据的最后一个变量的值不以&结束。这样我们可以将输入数据分解为一组一组的值。



　　但随后会发现CGI的输入并不规则，例如有时会出现类似下面格式的输入字符号串：filename=hello&cmd=world+I%27，这是因为浏览器对一些上传的特殊字符进行了编码，所以在将数据分解开后需要进行解码，



　　     解码规则为：



　　1\)+: 将+转换成空格符;



     2\) %xx: 用其十六进制ASCII码值表示的特殊字符（%作为为转意符）。根据值xx将其转换成相应的ASCII字符。对Form变量名和变量值都要进行这种转换。

