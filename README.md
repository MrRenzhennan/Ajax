# ajax
AJAX即“Asynchronous Javascript And XML”（异步JavaScript和XML），是指一种创建交互式网页应用的网页开发技术。  
AJAX是一种用于创建快速动态网页的技术。通过在后台与服务器进行少量数据交换，AJAX 可以使网页实现异步更新。  
这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新,传统的网页（不使用 AJAX）如果需要更新内容，必须重载整个网页页面。
(引自百度百科)  
  
Ajax 技术的核心是 XMLHttpRequest 对象（简称 XHR），XHR 为向服务器发送请求和解析服务器响应提供了流畅的接口。  
能够以异步方式从服务器取得更多信息，意味着用户单击后，可以不必刷新页面也能取得新数据。  
也就是说，可以使用 XHR 对象取得新数据，然后再通过 DOM 将新数据插入到页面中。  
另外，虽然名字中包含 XML 的成分，但 Ajax通信与数据格式无关；这种技术就是无须刷新页面即可从服务器取得数据，但不一定是 XML 数据。
#原生XHR对象应用
## 创建XML HttpRequest对象
```js
// 兼容低版本ie浏览器的代码,对于现代浏览器可以省略部分代码
function createXHR(){
    if (typeof XMLHttpRequest != "undefined"){
        return new XMLHttpRequest();
    } else if (typeof ActiveXObject != "undefined"){
         if (typeof arguments.callee.activeXString != "string"){
            var versions = [ "MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
            "MSXML2.XMLHttp"], i, len;
            for (i=0,len=versions.length; i < len; i++){
                try {
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                    break;
                } catch (ex){
                    //跳过
                }
            }
        }
        return new ActiveXObject(arguments.callee.activeXString);
     } else {
            throw new Error("No XHR object available.");
    }
}

var xhr =  createXHR();
```
## open方法
使用 XHR 对象时，要调用的第一个方法是 open()，  
它接受 3 个参数：要发送的请求的类型（“get”、 “post”等） 、请求的 URL 和表示是否异步发送请求的布尔值。  
下面就是调用这个方法的例子:  
`xhr.open(“get”, “example.php”, true);`  
有关这行代码，需要说明两点：  
* 一是 URL相对于执行代码的当前页面（当然也可以使用绝对路径）；
* 二是调用 open()方法并不会真正发送请求，而只是启动一个请求以备发送。
## send方法
XMLHttpRequest.send() 方法用于发送 HTTP 请求。  
如果是异步请求（默认为异步请求），则此方法会在请求发送后立即返回；  
如果是同步请求，则此方法直到响应到达后才会返回。  
XMLHttpRequest.send()接受一个可选的参数，其作为请求主体；如果请求方法是 GET 或者 HEAD，则应将请求主体设置为 null。
```js
xhr.send(null);
// xhr.send('string');
// xhr.send(new Blob());
// xhr.send(new Int8Array());
// xhr.send({ form: 'data' });
// xhr.send(document);
```
## 发送GIT请求
GET 是最常见的请求类型，最常用于向服务器查询某些信息。  
必要时，可以将查询字符串参数追加到 URL 的末尾，以便将信息发送给服务器。  
对 XHR 而言，位于传入 open()方法的 URL 末尾的查询字符串必须经过正确的编码才行。
### 辅助编码函数
```js
function addURLParam(url, name, value){
    url += url.indexOf('?') == -1 ? '?' : '&';
    url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
    return url;
}
```
```js
//真正发送GET请求
var xhr = createXHR();
xhr.open("GET", "http://test.com/?name=lisi&age=10",true);
xhr.send(null);
```
##发送post请求
使用频率仅次于 GET 的是 POST 请求，通常用于向服务器发送应该被保存的数据。   
POST 请求应该把数据作为请求的主体提交，而 GET 请求传统上不是这样。   
POST 请求的主体可以包含非常多的数据，而且格式不限。
  
默认情况下，服务器对 POST 请求和提交 Web 表单的请求并不会一视同仁。  
因此，服务器端必须有程序来读取发送过来的原始数据，并从中解析出有用的部分。  
不过，我们可以使用 XHR 来模仿表单提交：  
首先将 Content-Type 头部信息设置为 application/x-www-form-urlencoded，也就是表单提交时的内容类型，其次是以适当的格式创建一个字符串。
```js
//真正发送POST请求
xhr.open("post", "postexample.php", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
var form = document.getElementById("user-info");
xhr.send(序列化的表单数据字符串); //格式"name=value&name=value&name=value"
```
## 接收响应
在收到响应后，响应的数据会自动填充 XHR 对象的属性，相关的属性简介如下。  
responseText：作为响应主体被返回的文本。  
responseXML：如果响应的内容类型是“text/xml”或“application/xml”，这个属性中将保存包含着响应数据的 XML DOM 文档。  
* status：响应的 HTTP 状态。  
* statusText： HTTP 状态的说明。  
  
多数情况下，我们还是要发送异步请求，才能让JavaScript 继续执行而不必等待响应。  
此时，可以检测 XHR 对象的 readyState 属性，该属性表示请求响应过程的当前活动阶段。  
这个属性可取的值如下。  
0. 未初始化。尚未调用 open()方法。
1. 启动。已经调用 open()方法，但尚未调用 send()方法。
2. 发送。已经调用 send()方法，但尚未接收到响应。
3. 接收。已经接收到部分响应数据。
4. 完成。已经接收到全部响应数据，而且已经可以在客户端使用了  
    只要 readyState 属性的值由一个值变成另一个值，都会触发一次 readystatechange 事件。
```js
参考代码
var xhr = createXHR();
xhr.onreadystatechange = function(){
    if (xhr.readyState == 4){
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
            alert(xhr.responseText);
        } else {
            alert("Request was unsuccessful: " + xhr.status);
        }
    }
};
xhr.open("get", "example.txt", true);
xhr.send(null);
```

#jQuery 中的ajax应用

#jQuery Ajax接口
jQuery 库支持完整的 Ajax操作。这里所包含的所有函数和方法用于从服务端加载数据，并且不会导致页面刷新。
## 底层接口
jQuery.ajax( [settings ] )  
一个以“{键:值}”组成的AJAX 请求设置。所有选项都是可选的  
返回: jqXHR  
说明: 执行一个异步的HTTP（Ajax）的请求  
### 常见选项
method (default: ‘GET’)  
Type: String  
HTTP 请求方法 (比如：“POST”, “GET ”, “PUT”)。 (添加版本: 1.9.0)。（如果你使用jQuery 1.9.0 之前的版本，你需要使用type选项。）
#### success  
##### Type: Function( Anything data, String textStatus, jqXHR jqXHR )
请求成功后的回调函数。这个函数传递3个参数：  
* 从服务器返回的数据，并根据dataType参数进行处理后的数据或dataFilter回调函数，如果指定的话；  
* 一个描述状态的字符串;  
* 还有 jqXHR（在jQuery 1.4.x前为XMLHttpRequest） 对象 。  
  在jQuery 1.5， 成功设置可以接受一个函数数组。每个函数将被依次调用。这是一个 Ajax Event
#### error
##### 类型: Function( jqXHR jqXHR, String textStatus, String errorThrown )
请求失败时调用此函数。有以下三个参数：  
* jqXHR (在 jQuery 1.4.x前为XMLHttpRequest) 对象、  
* 描述发生错误类型的一个字符串   
* 和 捕获的异常对象。  
如果发生了错误，错误信息（第二个参数）除了得到null之外，还可能是“timeout”, “error”, “abort” ，和 “parsererror”。
  
url (默认: 当前页面地址)  
类型: String  
发送请求的地址  
#### data
##### 类型: PlainObject 或 String 或 Array
发送到服务器的数据。它被转换成一个查询字符串,如果已经是一个字符串的话就不会转换。  
查询字符串将被追加到GET请求的URL后面。  
参见 processData 选项说明，以防止这种自动转换。对象必须为“{键:值}”格式。  
如果这个参数是一个数组，jQuery会按照traditional 参数的值， 将自动转化为一个同名的多值查询字符串(查看下面的说明)。  
如 {foo:[“bar1”, “bar2”]} 转换为 ‘&foo=bar1&foo=bar2’。
  
`ataType (default: Intelligent Guess (xml, json, script, or html))  
Type: String`
从服务器返回你期望的数据类型。   
如果没有指定，jQuery将尝试通过MIME类型的响应信息来智能判断  
（一个XML MIME类型就被识别为XML，在1.4中 JSON将生成一个JavaScript对象，在1.4中 script 将执行该脚本，其他任何类型会返回一个字符串）。
```js
参考代码：


1. 保存数据到服务器，成功时显示信息。
$.ajax({
  method: "POST",
  url: "some.php",
  data: { name: "John", location: "Boston" }
}).done(function( msg ) {
  alert( "Data Saved: " + msg );
});


2. 装入一个 HTML 网页最新版本。
$.ajax({
  url: "test.html",
  cache: false
}).done(function( html ) {
  $("#results").append(html);
}); 


3. 发送 XML 数据至服务器。设置 processData 选项为 false，防止自动转换数据格式。
var xmlDocument = [create xml document];
var xmlRequest = $.ajax({
  url: "page.php",
  processData: false,
  data: xmlDocument
});
xmlRequest.done(handleResponse);

4. 发送id作为数据发送到服务器， 保存一些数据到服务器上， 并通一旦它的完成知用户。  如果请求失败，则提醒用户。

var menuId = $("ul.nav").first().attr("id");
var request = $.ajax({
  url: "script.php",
  method: "POST",
  data: {id : menuId},
  dataType: "html"
});
 
request.done(function(msg) {
  $("#log").html( msg );
});
 
request.fail(function(jqXHR, textStatus) {
  alert( "Request failed: " + textStatus );
});


5. 载入并执行一个JavaScript文件.
$.ajax({
  method: "GET",
  url: "test.js",
  dataType: "script"
});

```
## 快捷接口
jQuery.get( url [, data ] [, success ] [, dataType ] )  
jQuery.get( [settings ] )  
settings  
类型:PlainObject  
一组用于配置Ajax请求的键/值对。除了url以外的所有选项属性都是可选的  
```js
$.get("test.php");

Example: 请求 test.php 页面 并且发送url参数(虽然仍然忽视返回的结果)。
$.get("test.php", { name: "John", time: "2pm" } );

Example: 传递数组形式data参数给服务器 (虽然仍然忽视返回的结果).
$.get("test.php", { 'choices[]': ["Jon", "Susan"]} );

Example: Alert 从 test.php请求的数据结果 (HTML 或者 XML,取决于返回的结果).
$.get("test.php", function(data){
alert("Data Loaded: " + data);
});


Example: Alert 从 test.cgi请求并且发送url参数的数据结果 (HTML 或者 XML,取决于返回的结果).
$.get("test.cgi", { name: "John", time: "2pm" },
   function(data){
     alert("Data Loaded: " + data);
   });

Example: 获取test.php的页面已返回的JSON格式的内容 (<?php echo json_encode(array("name"=>"John","time"=>"2pm")); ?>), 并且加到页面中.
$.get("test.php",
   function(data){
     $('body').append( "Name: " + data.name ) // John
              .append( "Time: " + data.time ); //  2pm
   }, "json");

```
jQuery.post( url [, data ] [, success ] [, dataType ] )  
jQuery.post( [settings ] )  
说明:使用一个HTTP POST 请求从服务器加载数据  
```js
Example: 请求 test.php 页面, 但是忽略返回结果
$.post("test.php");

Example: 请求 test.php 页面 并且发送url参数(虽然仍然忽视返回的结果)。
$.post("test.php", { name: "John", time: "2pm" } );


Example: 传递数组形式data参数给服务器 (虽然仍然忽视返回的结果)。
$.post("test.php", { 'choices[]': ["Jon", "Susan"] });

Example: 使用Ajax请求发送表单数据。
$.post("test.php", $("#testform").serialize());


Example: Alert 从 test.php请求的数据结果 (HTML 或者 XML,取决于返回的结果)。
$.post("test.php", function(data) {
  alert("Data Loaded: " + data);
});


Example: Alert 从 test.cgi请求并且发送url参数的数据结果 (HTML 或者 XML,取决于返回的结果)。
$.post("test.php", { name: "John", time: "2pm" },
  function(data) {
    alert("Data Loaded: " + data);
  });


Example: 得到test.php的内容，存储在一个 XMLHttpResponse 对象中并且运用 process() JavaScript函数。
$.post("test.php", { name: "John", time: "2pm" },
  function(data) {
    process(data);
  },
 "xml"
);


Example: Posts to the test.php page and gets contents which has been returned in json format (<?php echo json_encode(array("name"=>"John","time"=>"2pm")); ?>).
$.post("test.php", { "func": "getNameAndTime" },
  function(data){
    console.log(data.name); // John
    console.log(data.time); //  2pm
  }, "json");

```
