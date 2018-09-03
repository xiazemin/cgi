# 动态web技术

CGI 全称为Common Gateway Interface （通用网关接口），目的是能够让服务器能够方便的调用外部程序。CGI本身是一套协议和规范，原则上只要是拥有读写文件功能的编程语言都可以用来编写CGI程序，例如C,C++,Perl,Visual Basic,Shell等等，历史上用来编写CGI程序使用最广泛的是Perl语言，连PHP一开始也是用Perl编写的，估计也受这个传统的影响。服务器在认为这是一个CGI请求时，会调用相关CGI程序，并通过环境变量和标准输出将数据传送给CGI程序，CGI程序处理完数据，生成html，然后再通过标准输出将内容返回给服务器，服务器再将内容交给用户，CGI进程退出，在这个过程中，服务器的标准输出对应了CGI程序的标准输入，CGI程序的标准输出对应着服务器的标准输入，相当于利用两条管道建立了进程间的通信。

下面是用C编写的一个CGI小程序，向服务器返回数据只需要将数据写入标准输出即可，可见CGI程序的编写也是相当容易的：



cgi.c :





\#include &lt;stdio.h&gt;

 

int main\(\)

{

        char MimeType\[\]="text/html";

        fprintf\(stdout, "Content-type: %s\r\n\r\n", MimeType\);  //输出响应头，响应头之后要加两个"\r\n"

        fprintf\(stdout, "&lt;html&gt;&lt;head&gt;&lt;title&gt;CGI小程序&lt;/title&gt;&lt;/head&gt;\n"\);

        fprintf\(stdout, "&lt;body&gt;由C编写的CGI小程序&lt;/body&gt;&lt;/html&gt;\n"\);

 

        return 0;

}



由于Nginx不支持CGI（支持CGI的升级版FastCGI和SCGI），而Apache原生支持CGI，所以这里选用Apache来举例

首先要对Apache进行一定的配置，使之支持CGI程序,配置如下：







LoadModule cgi\_module modules/mod\_cgi.so  \#注意这项配置是否已经存在，已存在就不要重复配置

 

AddHandler cgi-script .cgi    \#设置cgi程序的扩展名，这里.cgi扩展名文件会被当作CGI来执行

 

\#设置cgi-bin的目录权限，假设 /var/www/html 为你的DocumentRoot

&lt;Directory "/var/www/html/cgi"&gt;

    AllowOverride None

    Options Indexes ExecCGI  \# ExecCGI 表示该目录允许执行CGI，如果没有加这个权限，即使是.cgi也没有权限执行

    Order allow,deny

    Allow from all

&lt;/Directory&gt;



重启Apache，把上面的C编写的CGI小程序用gcc编译成可执行文件cgi.out，并放到你配置的CGI目录，这里为 /var/www/html/cgi-bin







\[root@localhost c\]\# gcc cgi.c -o test.cgi 

\[root@localhost c\]\# mv test.cgi /var/www/html/cgi/





然后浏览器访问该文件 ，就可以看到输出结果了







从以上可以看出，cgi编程和普通的编程并没有太大的区别，cgi小程序也是可以直接运行的可执行文件，并且大多数编程语言都可以进行cgi编程，这或许也是cgi能够流行起来的原因之一，下面看用 php写一个CGI程序。



cgi.php :



\#!/usr/bin/env php

&lt;?php

// cgi.php

//由于php脚本不是可执行文件，这里用shell的方式来执行php脚本

 

fwrite\(STDOUT, "Content-type: text/html\r\n\r\n"\);

fwrite\(STDOUT, "&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;&lt;b&gt;PHP编写的CGI程序演示 ". date\("Y-m-d H:i:s"\) ."&lt;/b&gt;&lt;/body&gt;&lt;/html&gt;\n"\);



给cgi.php 添加可执行权限

\[root@localhost cgi\]\# chmod +x cgi.php 

\[root@localhost cgi\]\# ./cgi.php 

COntent-type: text/html

 

&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;&lt;b&gt;PHP编写的CGI程序演示 2017-06-23 15:41:00&lt;/b&gt;&lt;/body&gt;&lt;/html&gt;

 



配置Apache支持.php扩展名的CGI小程序

AddHandler cgi-script .cgi .php    \#设置cgi程序的扩展名，这里 .cgi 和 .php 扩展名文件会被当作CGI来执行

 重启Apache，访问该php文件：







可以看到CGI协议非常简单，但当前为止还没有涉及获取http请求的GET和POST参数，这就要说前面提到过的环境变量和CGI小程序的标准输入了，



下面修改前面的C和PHP小程序，使之支持读取GET和POST参数



cgi.c :





//cgi.c

\#include &lt;stdio.h&gt;

\#include &lt;stdlib.h&gt;

extern char \*\*environ;  //调用环境变量

int main\(\)

{

 

        char MimeType\[\]="text/html";

        fprintf\(stdout, "Content-type: %s\r\n\r\n", MimeType\);  //输出响应头，响应头之后要加两个"\r\n"

        fprintf\(stdout, "&lt;html&gt;&lt;head&gt;&lt;title&gt;CGI小程序&lt;/title&gt;&lt;/head&gt;&lt;body&gt;\n"\);

 

        char \*\*env;

 

        //循环输出环境变量

        for\(env = environ; \*env != NULL; env++\)

        {

                fprintf\(stdout, "%s&lt;br&gt;\n", \*env\);

        }

 

        char \*query\_string = getenv\("QUERY\_STRING"\); //获取环境变量QUERY\_STRING,也就是GET参数

 

        fprintf\(stdout, "---------------&lt;br&gt;query\_string:%s&lt;br&gt;-------------&lt;br&gt;", query\_string\);

 

        fprintf\(stdout, "&lt;/body&gt;&lt;/html&gt;\n"\);

 

        return 0;

}



再次编译之后浏览器访问：



http://127.0.0.1:8888/cgi/test.cgi?user=zyee&age=27



访问结果：



HTTP\_HOST=127.0.0.1:8888

HTTP\_CONNECTION=keep-alive

HTTP\_CACHE\_CONTROL=max-age=0

HTTP\_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,\*/\*;q=0.8

HTTP\_UPGRADE\_INSECURE\_REQUESTS=1

HTTP\_USER\_AGENT=Mozilla/5.0 \(Windows NT 10.0; WOW64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/50.0.2661.102 Safari/537.36

HTTP\_ACCEPT\_ENCODING=gzip, deflate, sdch

HTTP\_ACCEPT\_LANGUAGE=zh-CN,zh;q=0.8,en;q=0.6

HTTP\_COOKIE=UM\_distinctid=15b130cb46b156-0f197c7d0f440a-3e64430f-1fa400-15b130cb46d3f2; CNZZDATA1258531050=1910116482-1490671422-%7C1490671422; \_cnzz\_CV1258531050=isLogin%7CLogout%7C1493265278109; CNZZDATA5201073=cnzz\_eid%3D509423027-1490969081-http%253A%252F%252F127.0.0.1%253A801%252F%26ntime%3D1490969081; Hm\_lvt\_6bcd52f51e9b3dce32bec4a3997715ac=1496995397

PATH=/sbin:/usr/sbin:/bin:/usr/bin

SERVER\_SIGNATURE=

Apache/2.2.15 \(CentOS\) Server at 127.0.0.1 Port 8888

 

SERVER\_SOFTWARE=Apache/2.2.15 \(CentOS\)

SERVER\_NAME=127.0.0.1

SERVER\_ADDR=10.0.2.15

SERVER\_PORT=8888

REMOTE\_ADDR=10.0.2.2

DOCUMENT\_ROOT=/var/www/html

SERVER\_ADMIN=root@localhost

SCRIPT\_FILENAME=/var/www/html/cgi/test.cgi

REMOTE\_PORT=58373

GATEWAY\_INTERFACE=CGI/1.1

SERVER\_PROTOCOL=HTTP/1.1

REQUEST\_METHOD=GET

QUERY\_STRING=user=zyee&age=27

REQUEST\_URI=/cgi/test.cgi?user=zyee&age=27

SCRIPT\_NAME=/cgi/test.cgi

---------------

query\_string:user=zyee&age=27

-------------





可以看到环境变量中包含了很多有用信息， 包括当前的URL，GET参数，客户端IP地址，请求头等等信息。

下面用PHP脚本来演示获取POST参数



cgi.php :







\#!/usr/bin/env php

&lt;?php

// cgi.php

//由于php脚本不是可执行文件，这里用shell脚本的方式来让php脚本可执行

 

$post = fread\(STDIN, 1024\);  // post参数从标准输入读取

 

fwrite\(STDOUT, "COntent-type: text/html\r\n\r\n"\);

fwrite\(STDOUT, "&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;"\);

 

fwrite\(STDOUT, "\npost: $post\n"\);

 

fwrite\(STDOUT, "&lt;b&gt;PHP编写的CGI程序演示 ". date\("Y-m-d H:i:s"\) ."&lt;/b&gt;"\);

fwrite\(STDOUT, "&lt;/body&gt;&lt;/html&gt;\n"\);

~                                          





curl模拟POST请求



\[root@localhost cgi\]\# curl -d "name=zyee&age=27" "http://127.0.0.1/cgi/cgi.php"

&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;

post: name=zyee&age=27

&lt;b&gt;PHP编写的CGI程序演示 2017-06-23 17:25:26&lt;/b&gt;&lt;/body&gt;&lt;/html&gt;





上传文件类型的POST请求  \( Multipart/form-data \)：



\[root@localhost cgi\]\# curl -F "filename=@/var/www/html/cgi/test.cgi" "http://127.0.0.1/cgi/cgi.php"

&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;

post: ------------------------------1a7c7aacb288

Content-Disposition: form-data; name="filename"; filename="test.cgi"

Content-Type: application/octet-stream

 

 

&lt;b&gt;PHP编写的CGI程序演示 2017-06-23 19:32:08&lt;/b&gt;&lt;/body&gt;&lt;/html&gt;



CGI编写对于有编程基础的人来说是相当容易上手的，那为什么又会被历史所遗弃呢，这就不得不说到CGI的运行方式问题了，每当一个CGI请求过来时，服务器会fork一个子进程来执行相应的CGI程序，当请求结束时，该CGI进程也随之结束，这样不停fork进程的开销是非常大的，这是造成CGI程序效率低下的主要原因。我们可以让CGI程序睡眠一段时间来观察这个过程，比如修改以上程序如下：







\#include &lt;stdio.h&gt;

\#include &lt;unistd.h&gt;

 

 

int main\(\)

{

        sleep\(10\);  //睡眠10秒钟

 

        char MimeType\[\]="text/html";

        fprintf\(stdout, "Content-type: %s\r\n\r\n", MimeType\);  //输出文件类型

 

        fprintf\(stdout, "&lt;html&gt;&lt;head&gt;&lt;title&gt;CGI小程序&lt;/title&gt;&lt;/head&gt;\n"\);

        fprintf\(stdout, "&lt;body&gt;由C编写的CGI小程序&lt;/body&gt;&lt;/html&gt;"\);

 

        return 0;

}



然后请求该页面，观察服务器进程信息，可以看到httpd进程fork出的一个子进程在执行CGI程序，并且请求结束该子进程便会退出















正是这种缺陷，所以Apache之后又推出了CGI模块的升级版 mod\_cgid 模块，我们来看看官网对该模块的介绍：



Except for the optimizations and the additional ScriptSock directive noted below, mod\_cgid behaves similarly to mod\_cgi. See the mod\_cgi summary for additional details about Apache and CGI.



On certain unix operating systems, forking a process from a multi-threaded server is a very expensive operation because the new process will replicate all the threads of the parent process. In order to avoid incurring this expense on each CGI invocation, mod\_cgid creates an external daemon that is responsible for forking child processes to run CGI scripts. The main server communicates with this daemon using a unix domain socket.





大致意思： 



除了优化性能和增加了 ScriptSock指令外，mod\_cgid 和 mod\_cgi是非常类似的...



在当前的unix操作系统上，一个多线程的服务fork一个子进程代价是非常高昂的，因为fork出来的子进程会复制它父进程的所有线程。为了避免每次执行CGI程序都引起这个高代价的操作， mod\_cgid模块创建一个外部的守护进程来负责创建子进程执行CGI程序，server和这个守护进程间通过unix domain socket来通信。





