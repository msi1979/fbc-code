# fbc-code
前端常用的通信技术
2017-06-22 陈为平 携程技术中心
作者简介
 
陈为平，携程市场部前端工程师，目前主要负责“携程运动”项目的大前端相关工作。

前段时间在忙开发携程运动项目和相应的微信小程序，其中和后端通信犹为频繁。get、post请求方法是很多前端童鞋使用最频繁的；websocket在11年盛行后方便了客户端和服务器之间传输，……and so on ，除了这些，还有很多我们不常使用的其他方式，但是在实际的业务场景中却真实需要。

本文总结了目前前端使用到的数据交换方式，阐述了业务场景中如何选择适合的方式进行数据交换( form ,xhr, fetch, SSE, webstock, postmessage， web workers等)，并列举了一些示例代码， 可能存在不足的地方，欢迎大家指正。

本文用到的源代码都放在Github上，点击下方阅读原文可直达。

关于HTTP协义基础可以参考阮一峰老师的《HTTP协议入门》一文。
前端经常使用的HTTP协议相关(1.0 / 1.1)

method
·        GET ( 对应 restful api 查询资源, 用于客户端从服务端取数据 )
·        POST(对应 restful api中的增加资源, 用于客户端传数据到服务端)
·        PUT (对应 restful api中的更新资源)
·        DELETE （ 对应 restful api中的删除资源 )
·        HEAD ( 可以用于http请求的时间什么，或者判断是否存在判断文件大小等)
·        OPTIONS （在前端中常用于 cors跨域验证）
·        TRACE * (我这边没有用到过，欢迎补充)
·        CONNECT * (我这边没有用到过，欢迎补充)

enctype
·        application/x-www-form-urlencoded (默认，正常的提交方式)
·        multipart/form-data(有上传文件时常用这种)
·        application/json (ajax常用这种格式)
·        text/xml
·        text/plain

enctype示例说明（ form , ajax, fetch 三种示例 )
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>enctype</title>
    <style>
        .box{border: 1px solid #ccc;padding:20px;}
        .out{background: #efefef; padding:10px 20px; margin-top: 20px;}
    </style>
    <script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.2.3.min.js"></script>
    
    <script>

    $(function(){

        $('#b1').on('click', function(){
            $.ajax({
                method: "POST",
                contentType:'application/x-www-form-urlencoded;charset=UTF-8',
                url: "form_action.php",
                data: {username: "John", password: "Boston" }
            }).done(function( msg ) {
                $('#msg1').html(msg);
            });
        });

        $('#f1').on('click', function(){
            fetch("form_action.php", {
                method: "POST",
                credentials: 'include', //带上cookie
                headers: {
                    "Content-Type": "application/x-www-form-urlencoded;charset=UTF-8"
                },
                body: "username=John&password=Boston"
            })
            .then(function(response){
                return response.text();
            })
            .then(function(msg) {
                $('#msg1').html(msg);

            }, function(e) {
                alert("Error submitting form!");
            });
        });


        $('#b2').on('click', function(){
            var formData = new FormData(document.querySelector("#data2"));
            $.ajax({
                method: "POST",
                processData:false, //无需让jquery正处理一下数据
                contentType:false, //已经是formData就默认为 multipart/form-data
                cache: false,
                url: "form_action.php",
                data: formData
            }).done(function( msg ) {
                $('#msg2').html(msg);
            });
        });

        $('#f2').on('click', function(){
            var formData = new FormData(document.querySelector("#data2"));
            fetch("form_action.php", {
                method: "POST",
                headers: {
                    "Content-Type": "multipart/form-data;charset=UTF-8"
                },
                body: formData
            })
            .then(function(response){
                return response.text();
            })
            .then(function(msg) {
                $('#msg2').html(msg);
            }, function(e) {
                alert("Error submitting form!");
            });
        });


        $('#b3').on('click', function(){
            $.ajax({
                method: "POST",
                contentType:'application/json;charset=UTF-8',
                url: "form_action.php",
                data: JSON.stringify({username: "John", password: "Boston" })
            }).done(function( msg ) {
                $('#msg3').html(msg);
            });
        });

        $('#f3').on('click', function(){
            var formData = new FormData(document.querySelector("#data2"));
            fetch("form_action.php", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json;charset=UTF-8"
                },
                body: JSON.stringify({username: "John", password: "Boston" })
            })
            .then(function(response){
                return response.text();
            })
            .then(function(msg) {
                $('#msg3').html(msg);
            }, function(e) {
                alert("Error submitting form!");
            });
        });

        $('#b4').on('click', function(){
            $.ajax({
                method: "POST",
                contentType:'text/plain;charset=UTF-8',
                processData:false, //无需让jquery正处理一下数据
                url: "form_action.php",
                data: "我是一个纯正的文本功能!\r\n我第二行"
            }).done(function( msg ) {
                $('#msg4').html(msg);
            });
        });

        $('#f4').on('click', function(){
            var formData = new FormData(document.querySelector("#data2"));
            fetch("form_action.php", {
                method: "POST",
                headers: {
                    "Content-Type": "text/plain;charset=UTF-8"
                },
                body: "我是一个纯正的文本功能!\r\n我第二行"
            })
            .then(function(response){
                return response.text();
            })
            .then(function(msg) {
                $('#msg4').html(msg);
            }, function(e) {
                alert("Error submitting form!");
            });
        });

        $('#b5').on('click', function(){
            $.ajax({
                method: "POST",
                contentType:'text/xml;charset=UTF-8',
                // processData:false, //无需让jquery正处理一下数据
                url: "form_action.php",
                data: "<doc><h1>我是标签</h1><p>我是内容</p></doc>"
            }).done(function( msg ) {
                $('#msg5').html(msg);
            });
        });

        $('#f5').on('click', function(){
            var formData = new FormData(document.querySelector("#data2"));
            fetch("form_action.php", {
                method: "POST",
                headers: {
                    "Content-Type": "text/xml;charset=UTF-8"
                },
                body: "<doc><max>我是XML标签</max><min>我是XML内容</min></doc>"
            })
            .then(function(response){
                return response.text();
            })
            .then(function(msg) {
                $('#msg5').html(msg);
            }, function(e) {
                alert("Error submitting form!");
            });
        });


    });
    </script>
</head>
<body>

<h1>enctype测试</h1>

<h2>表单提交: application/x-www-form-urlencoded</h2>
<div class="box">
    <form action="form_action.php" enctype="application/x-www-form-urlencoded" method="post">
        <p>用户: <input type="text" name="username" /></p>
        <p>密码: <input type="text" name="password" /></p>
        <input type="submit" value="提交" />
        <button type="button" id="b1">AJAX提交</button>
        <button type="button" id="f1">fetch提交</button>
    </form>
    <div id="msg1" class="out"></div>
</div>

<h2>multipart/form-data</h2>
<div class="box">
    <form id="data2" action="form_action.php" enctype="multipart/form-data" method="post">
        <p>用户: <input type="text" name="username" /></p>
        <p>密码: <input type="text" name="password" /></p>
        <p>文件: <input type="file" name="file" id="file1" /></p>
        <input type="submit" value="提交" />
        <button type="button" id="b2">AJAX提交</button>
        <button type="button" id="f2">fetch提交</button>
    </form>
    <div id="msg2" class="out"></div>
</div>

<h2>application/json</h2>
<div class="box">
    <form action="form_action.php" enctype="application/json" method="post">
        <p>用户: <input type="text" name="username" /></p>
        <p>密码: <input type="text" name="password" /></p>
        <input type="submit" value="提交" />
        <button type="button" id="b3">AJAX提交</button>
        <button type="button" id="f3">fetch提交</button>
    </form>
    <div id="msg3" class="out"></div>
</div>

<h2>text/plain</h2>
<div class="box">
    <form action="form_action.php" enctype="text/plain" method="post">
        <p>用户: <input type="text" name="username" /></p>
        <p>密码: <input type="text" name="password" /></p>
        <input type="submit" value="提交" />
        <button type="button" id="b4">AJAX提交</button>
        <button type="button" id="f4">fetch提交</button>
    </form>
    <div id="msg4" class="out"></div>
</div>

<h2>text/xml</h2>
<div class="box">
    <form action="form_action.php" enctype="text/xml" method="post">
        <p>用户: <input type="text" name="username" /></p>
        <p>密码: <input type="text" name="password" /></p>
        <input type="submit" value="提交" />
        <button type="button" id="b5">AJAX提交</button>
        <button type="button" id="f5">fetch提交</button>
    </form>
    <div id="msg5" class="out"></div>
</div>

</body>
</html>
服务端 form_action.php
<?php
echo '<pre>';

if($_POST){
    echo "<h1>POST</h1>";
    print_r($_POST);
    echo "<hr>";
}

if(file_get_contents("php://input")){
    echo "<h1>php://input</h1>";
    print_r(file_get_contents("php://input"));
    echo "<hr>";
}

if($_FILES){
    echo "<h1>file</h1>";
    print_r($_FILES);
    echo "<hr>";
}
* fetch api是基于Promise设计
* fetch 的一些例子 mdn/fetch-examples

服务器到客户端的推送 - Server-sent Events

这个是html5的一个新特性，主要用于服务器推送消息到客户端, 可以用于监控，通知，更新库存之类的应用场景, 在携程运动项目中我们主要应用于线上被预订后通知下发通知到场馆的操作界面上的即时改变状态。

图片来源于网络，侵删

优点： 基于http协义无需特别的改造，调试方便， 可以CORS跨域
server-send events 是服务端往客户端单向推送的，如果客户端需要上传消息可以使用 WebSocket

客户端代码
var source = new EventSource('http://localhost:7000/server');source.onmessage = function(e) {
    console.log('e', JSON.parse( e.data));
    document.getElementById('box').innerHTML += "SSE notification: " + e.data + '<br />';
};
服务端代码

<?php 
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');
//数据
$time = date('Y-m-d H:i:s');
$data = array(
    'id'=>1,
    'name'=>'中文',
    'time'=>$time
);
echo "data: ".json_encode($data)."\n\n";
flush();
?>
echo "event: ping\n"; // 增加 event可以多送多个事件
js使用 source.addEventListener('ping', function(){}, false); 来处理对应的事件

对于低版本的浏览器可以使用 eventsource polyfill

Yaffle/EventSource by yaffle
https://github.com/remy/polyfills/blob/master/EventSource.js by Remy Sharp
rwaldron/jquery.eventsource by Rick Waldron
amvtek/EventSource by AmvTek

客户端与服务器双向通信 WebSocket
特点

1. websocket 是个双向的通信。
2. 常用于应用于一些都需要双方交互的，实时性比较强的地方(如聊天,在线客服)
3. 数据传输量小
4. websocket 是个 持久化的连接

原理图

图片来源于网络. 侵删

这个的服务端是基于 nodejs实现的（不要问为什么不是php，因为 nodejs 简单些！)

server.js

var WebSocketServer = require('ws').Server;
var wss = new WebSocketServer({port: 2000});

wss.on('connection', function(ws) {
    ws.send('服务端发来一条消息');
    ws.on('message', function(message) {
        //转发一下客户端发过来的消息
        console.log('收到客户端来的消息: %s', message);
        ws.send('服务端收到来自客户端的消息:' + message);
    });
    ws.on('close', function(event) {
        console.log('客户端请求关闭',event);
    });
});
client.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>WebSocket 双向通信</title>
    <style>
      #boxwarp > div{
        border: 1px solid #ccc;
        padding:10px;
        margin:10px;
      }
    </style>

</head>
<body>
<button id="btn">发点什么</button>
<div id="boxwarp"></div>
<script>
var ws = new WebSocket("ws://127.0.0.1:2000/");
document.getElementById('btn').addEventListener('click', function() {
  ws.send('cancel_order');
});
function addbox(msg){
  var box = document.createElement('div');
      box.innerHTML = msg;
  document.getElementById('boxwarp').append(box);
}
ws.onopen = function() {
    var msg = 'ws已经联接';
    addbox(msg);
    ws.send(msg);
};
ws.onmessage = function (evt) {
    console.log('evt');
    addbox(evt.data);
};

ws.onclose = function() {
   console.log('close');
   addbox('服务端关闭了ws');
};
ws.onerror = function(err) {
   addbox(err);
};
</script>
</body>
</html>
说完了客户端与服客端之间的通信，现在我们来聊聊客户端之间的通信。


客户端与客户端页面之间的通信 postMessage

主要特点
1. window.postMessage() 方法可以安全地实现跨域通信
2.主要用于两个页面之间的消息传送
3. 可以使用iframe与window.open打开的页面进行通信.
特别的应用场景
我们的页面引用了其他的人页面，但我们不知道他们的页面高度，这时可以通过window.postMessages 从iframe 里面的页面来传到 当前页面.

语法

otherWindow.postMessage(message, targetOrigin, [transfer]);

示例代码
postmessage.html (入口)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>postmessage示例</title>
    <style>
        html,body{height: 100%;}
        *{padding: 0; margin:0;}
        .warp{ display: flex; }
        .warp > div,
        .warp > iframe{
            flex: 1;
            margin:10px;
        }
        iframe{
            height: 600px;
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>
<div class="warp">
    <div class="left">
        左边页面    </div>
    <div class="right">
        右边页面    </div></div><div class="warp">
    <div class="left warp">
        <iframe src="./post1.html" frameborder="0" id="post1" name="post1"></iframe>
    </div>
    <div class="right warp">
        <iframe src="./post2.html" frameborder="0" id="post2" name="post2"></iframe>
    </div>

<!-- window.frames[0].postMessage('getcolor','http://lslib.com'); -->
</div>

<div class="warp">
    <div class="left"><button id="postBtn1">向左边的(iframe)推送信息代码</button></div>
    <div class="right"><button id="postBtn2">向右边的(iframe)推送信息代码</button></div>
</div>

<script>
document.getElementById('postBtn1').addEventListener('click', function(){
    console.log('postBtn1');
    var date = new Date().toString();
    window.post1.postMessage(date,'*');});document.getElementById('postBtn2').addEventListener('click', function(){
    console.log('postBtn2');
    var date = new Date().toString();
    window.post2.postMessage(date,'*');});window.addEventListener('message',function(e){
    if(e.data){
        console.log(e.data);
        console.log(e);
        window.post1.postMessage(e.data,'*');
    }},false);
</script>
</body>
</html>
post1.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .sendbox{
            background: #efefef;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
<div class="sendbox">
    <button id="sendbox2">直接发送到右边iframe</button>
    左边的iframe
</div>
<div id="box2">
</div>
<script>
    document.getElementById('sendbox2').addEventListener('click', function(){
        window.parent.post2.postMessage('收到来自左边ifarme的消息' + +new Date(),'*');
    });
    function addbox(html){
        var item = document.createElement('div');
        item.innerHTML = html;
        document.getElementById('box2').append(item);
    }
    window.addEventListener('message',function(e){
        if(e.data){
            addbox(e.data);
        }
    },false);
</script>

</body>
</html>
post2.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .sendbox{
            background: #ccc;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
<div class="sendbox" style="text-align: right;">
    <button id="sendbox">中转到左边</button>
    <button id="sendbox2">直接到左边</button>
    右边的iframe
</div>
<div id="box"></div>

<script>
    document.getElementById('sendbox').addEventListener('click', function(){
        /*- 向父级页面传 -*/
        window.parent.postMessage('来自post2的消息' + +new Date(),'*');
    });
    document.getElementById('sendbox2').addEventListener('click', function(){
        window.parent.post1.postMessage('直接来自右边' + +new Date(),'*');
    });
    function addbox(html){
        var item = document.createElement('div');
        item.innerHTML = html;
        document.getElementById('box').append(item);
    }
    window.addEventListener('message',function(e){
        if(e.data){
            addbox(e.data);
        }
    },false);
</script>

</body>
</html>
Web Workers 进程通信（html5中的js的后台进程）

javascript设计上是一个单线，也就是说在执行js过程中只能执行一个任务, 其他的任务都在队列中等待运行。
如果我们执行大量计算的任务时，就会阻止浏览器执行js，导致浏览器假死。

html5的 web Workers 子进程 就是为了解决这种问题而设计的。把大量计算的任务当作类似ajax异步方式进入子进程计算，计算完了再通过 postmessage通知主进程计算结果。

图片来源于网络. 侵删
主线程代码(index.html)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .box-warp > div{
            border: 1px solid #ccc;
            margin:10px;
            padding:10px;
        }
    </style>
</head>
<body>
    <button id="btn">开启一个后台线程(点击外框中止线程)</button>
    <div class="box-warp" id="boxwarp"></div>

    <script>
    var id = 1;
    function init_works(){
        var warpid = 'box'+id;
        var box = document.createElement('div');
            box.id = warpid;
        document.getElementById('boxwarp').append(box);
        var worker = new Worker('./compute.js');
        //监听后台进程发过来的消息
        worker.onmessage= function (event) {
            // 把子线程返回的结果添加到 div 上
            document.getElementById(warpid).innerHTML += event.data+"<br/>";
        };
        //点击中止后端进程
        box.addEventListener('click', function(){
            worker.postMessage("oh, 我被干掉了" + warpid);
            var time = setTimeout(function(){
                worker.terminate();
                clearTimeout(time);
            },0);
        });
        //往后台线程发送消息
        worker.postMessage("hi, 我是" + warpid);
        id++;
    }

    document.getElementById('btn').addEventListener('click', function(){
        init_works();
    });
    </script>
</body>
</html>
后台进程代码( compute.js )

var i=0;
function timeX(){
    i++;
    postMessage(i);
    if(i>9){
        postMessage('no 我不想动了');
        close(); //中止线程
    }
    setTimeout(function(){
        timeX();
    },1000);
}

timeX();

//收到主线程的消息

onmessage = function (oEvent) {
  postMessage(oEvent.data);
};
上述代码简单的说明一下， 主进程与后台进程之间的互相通信。

（携程技术中心市场营销研发部武艺嫱，对本文亦有贡献）
https://mp.weixin.qq.com/s?__biz=MjM5MDI3MjA5MQ==&mid=2697266201&idx=2&sn=1b2ca738a21c6d899e82fa6fe769446b##
