# 概要

<!--type=misc-->

一个由Node编写的回应'Hello World'的 [web server](http.html) 的例子：

    var http = require('http');

    http.createServer(function (request, response) {
      response.writeHead(200, {'Content-Type': 'text/plain'});
      response.end('Hello World\n');
    }).listen(8124);

    console.log('Server running at http://127.0.0.1:8124/');

把代码写入名为 `example.js` 的文件并通过node程序来执行它来启动这个web服务。

译注：代码中该服务运行在 `http://127.0.0.1:8124/` ，可通过浏览器访问该地址获得运行结果。

    > node example.js
    Server running at http://127.0.0.1:8124/

所有在文档中的例子都可以被类似地运行。