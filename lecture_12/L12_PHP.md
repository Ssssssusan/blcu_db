## PHP 内置数组 

- $_GET：预定义的 $_GET 变量用于收集来自 method="get" 的表单中的值。从带有 GET 方法的表单发送的信息，对任何人都是可见的（会显示在浏览器的地址栏），并且对发送信息的量也有限制。
- $_POST： 预定义的 $_POST 变量用于收集来自 method="post" 的表单中的值。从带有 POST 方法的表单发送的信息，对任何人都是不可见的（不会显示在浏览器的地址栏），并且对发送信息的量也没有限制。
- $_SERVER：变量由web服务器设定或者直接与当前脚本的执行环境相关联。
- $_SESSION：PHP session 变量用于存储关于用户会话（session）的信息，或者更改用户会话（session）的设置。Session 变量存储单一用户的信息，并且对于应用程序中的所有页面都是可用的。
- $_COOKIE：cookie 常用于识别用户。cookie 是一种服务器留在用户计算机上的小文件。每当同一台计算机通过浏览器请求页面时，这台计算机将会发送 cookie。通过 PHP，您能够创建并取回 cookie 的值。
- $_FILES：经由HTTP POST 文件上传而提交至脚本的变量。
- $_ENV: 执行环境提交至脚本的变量。
- $_REQUEST: 经由GET，POST 和 COOKIE机制提交至脚本的变量。
- $GLOBALS：包含一个引用指向每个当前脚本的全局范围内有效的变量。

## PHP 数据传递
- POST
- GET
- SESSION/COOKIE

### COOKIE 和 SESSION 的区别
COOKIE的数据信息存放在客户端浏览器上。SESSION 的数据信息存放在服务器上。安全性不同。

## PHP 内置数组获取服务器信息
- $_SERVER['REMOTE_ADDR'] //当前用户 IP 。
- $_SERVER['REMOTE_HOST'] //当前用户主机名
- $_SERVER['REQUEST_URI'] //URL
- $_SERVER['REMOTE_PORT'] //端口。
- $_SERVER['SERVER_NAME'] //服务器主机的名称。

