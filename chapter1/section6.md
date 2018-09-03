# CGI程序实现步骤

1．从服务器获取数据

C语言实现代码：



\#include &lt;stdio.h&gt;



\#include &lt;stdlib.h&gt;



\#include &lt;string.h&gt;



 



int get\_inputs\(\)



{



int length;



char \*method;



char \*inputstring;



 



method = getenv\(“REQUEST\_METHOD”\); //将返回结果赋予指针



if\(method == NULL\)



    return 1;       //找不到环境变量REQUEST\_METHOD



if\(!strcmp\(method, ”POST”\)\)  // POST方法



{



    length = atoi\(getenv\(“CONTENT\_LENGTH”\)\); //结果是字符，需要转换



    if\(length != 0\)



    {



        inputstring = malloc\(sizeof\(char\)\*length + 1\) //必须申请缓存，因为stdin是不带缓存的。



        fread\(inputstring, sizeof\(char\), length, stdin\); //从标准输入读取一定数据



}



}



else if\(!strcmp\(method, “GET”\)\)



{



    Inputstring = getenv\(“QUERY\_STRING”\);   



    length = strlen\(inputstring\);



}



if\(length == 0\)



return 0;



}



Perl实现代码：



$method = $ENV{‘REQUEST\_METHOD’};



if\($method eq ‘POST’\)



{



    Read\(STDIN, $input, $ENV{‘CONTENT\_LENGTH’}\);



}



if\($method eq ‘GET’ \|\| $method eq ‘HEAD’\)



{



    $input = $ENV{‘QUERY\_STRING’};



}



if\($input eq “”\)



{



&print\_form;



exit;



}



       PYTHON代码实现



\#!/usr/local/bin/python



import cgi



def main\(\):



form = cgi.FieldStorage\(\)



 



Python代码实现更简单，cgi.FieldStorage\(\)返回一个字典，字典的每一个key就是变量名，key对应的值就是变量名的值，更本无需用户再去进行数据解码！



 



       获取环境变量的时候，如果先判断“REQUEST\_METHOD”是否存在，程序会更健壮，否则在某些情况下可能会造成程序崩溃。因为假若CGI程序不是由服务器调用的，那么环境变量集里就没有与CGI相关的环境变量（如REQUEST\_METHOD，REMOTE\_ADDR等）添加进来，也就是说“getenv\(“REQUEST\_METHOD”\)”将返回NULL！



2．URL编码

不管是POST还是GET方式，客户端浏览器发送给服务器的数据都不是原始的用户数据，而是经过URL编码的。此时，CGI的环境变量Content\_type将被设置，如Content\_type = application/x-www-form-urlencode就表示服务器收到的是经过URL编码的包含有HTML表单变量数据。



编码的基本规则是：



变量之间用“&”分开；



变量与其对应值用“=”连接；



空格用“+”代替；



保留的控制字符则用“%”连接对应的16禁止ASCII码代替；



某些具有特殊意义的字符也用“%”接对应的16进制ASCII码代替；



空格是非法字符；



任意不可打印的ASCII控制字符均为非法字符。



例如，假设3个HTML表单变量filename、e-mail和comments，它们的值对应分别为hello、mike@hotmail.com和I’ll bethere for you，则经过URL编码后应为：



 



filename=hello&e-mail=hello@hotmail.com&comments=I%27ll+be+there+for+you



 



 



所以，CGI程序从标准输入或环境变量中获取客户端数据后，还需要进行解码。解码的过程就是URL编码的逆变：根据“&”和“=”分离HTML表单变量，以及特殊字符的替换。



在解码方面，PYTHON代码实现是最理想的，cgi.FieldStorage\(\)函数在获取数据的同时就已自动进行代码转换了，无需程序员再进行额外的代码编写。Perl其次，因为在一个现成的Perl库：cgi-lib.pl中提供了ReadParse函数，用它来进行URL解码很简单：



require ‘cgi-lib.pl’;



&ReadParse\(\*input\);





