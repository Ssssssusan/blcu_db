## PHP 程序的运行流程
PHP代码是在服务器端被执行的。用户访问一个包含PHP代码的网页时，发送Request到服务器，其中包含网页的文件名。服务器收到Request后，找到文件名指向的文件，发现其中嵌有PHP代码，会调用PHP解释器处理该文件，然后将处理后的结果整理到Response，发送到客户端。PHP代码可以与服务器端的数据库或其他资源进行交互，或者根据用户的操作产生不同的页面。

因此，PHP脚本的触发是在服务器收到客户端的Request。收到一个Request后，服务器触发一个PHP脚本；处理完脚本后，返回结果到客户端，等待下一个Request。当收到下一个Request后，服务器触发另一个（或同一个）PHP脚本。两次PHP脚本的运行是相互独立的，第二次脚本的运行几乎不受前一次脚本运行的影响。

## 目前常用的服务器软件有哪些
1. IIS  
IIS是英文Internet Information Server的缩写，baidu译成中文就是"Internet信息服务"的意思。它是微zhi软公司主推的服务器，最新的dao版本是Windows2008里面包含的IIS 7，IIS与Window Server完全集成在一起，因而用户能够利用Windows Server和NTFS（NT File System，NT的文件系统）内置的安全特性，建立强大，灵活而安全的Internet和Intranet站点。

2. ApacheApache  
ApacheApache在世界上的排名是第一的，它可以运行在几乎所有广泛使用的计算机平台上。Apache源于NCSAhttpd服务器，经过多次修改，不仅简单、速度快、而且性能稳定，还可以用来做代理服务器。

3. Nginx  
Nginx不仅是一个小巧且高效的HTTP服务器，也可以做一个高效的负载均衡反向代理，通过它接受用户的请求并分发到多个Mongrel进程可以极大提高Rails应用的并发能力。

4. Zeus  
Zeus是一个运行于Unix下的非常优秀的Web Server，据说性能超过Apache，是效率最高的Web Server之一。

5. Sun  
Sun的Java系统Web服务器也就是以前的Sun ONE Web Server。主要出现在那些运行Sun的Solaris操作系统的关键任务级Web服务器上。它最新的版本号是6.1,可以支持x86版本Solaris，Red Hat Linux，HP-UX 11i， IBM AIX，甚至可以支持Windows，但它的大多数用户都选择了SPARC版本的Solaris操作系统。

## 如何将 PHP 与 Apache 建立关联
找到 Apache 文件夹下 httpd.conf，找到 #LoadModule xml2enc_module modules/mod_xml2enc.so一行添加 LoadModule php7_module D:/php7/php7apache2_4.dll（文件路径）

## 主目录下面的子目录和虚拟目录有何不同
- 子目录：根目录下的真实存在的文件夹
- 虚拟目录：文件夹以及其对应的访问权限，是通过将其他目录以映射的方式虚拟到该FTP服务器的主目录下形成的。