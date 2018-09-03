# GET c程序示例

　　void main\(void\) {// 本程序将用户输入的数据打印出来 fprintf\(stdout,"content-type:text/plain\n\n"\); // 输出一个CGI标题，这行代码的意义后面会讲解 char \*pszMethod; pszMethod =getenv\("REQUEST\_METHOD"\); if\(strcmp\(pszMethod,"GET"\) == 0\) { //GET method //读取环境变量来获取数据 printf\("This is GETMETHOD!\n"\); printf\("SERVER\_NAME:%s\n",getenv\("SERVER\_NAME"\)\); printf\("REMOTE\_ADDR:%s\n",getenv\("REMOTE\_ADDR"\)\); fprintf\(stdout,"input data is:%s\n",getenv\("QUERY\_STRING"\)\); } else { // POST method //读取STDIN来获取数据 intiLength=atoi\(getenv\("CONTENT\_LENGTH"\)\); printf\("This is POSTMETHOD!\n"\); fprintf\(stdout,"input data is:\n"\); for\(int i=0;i&lt;iLength;i++\) { char cGet=fgetc\(stdin\); fputc\(cGet,stdout\); } } }

