# Web安全之CSRF和XSS

Web安全问题中，CRSF和XSS是最常见的攻击方式，本文将从概念、原理、举例、预防方式和两种方式对比总结这两种安全问题。

## Cookie概念

Cookie是一个很小的文本文件，是浏览器储存在用户的机器上的。Cookie是纯文本，没有可执行代码。储存一些服务器需要的信息，每次请求站点，会发送相应的cookie，这些cookie可以用来辨别用户身份信息等作用

采取key=value的键值对对形式存储数据，key是唯一的。

Domain:域名，限制那些域名下可以使用

Path:路径，只有这个路径前缀的才可以用

Expires:过期时间

Http（HttpOnly):只有浏览器请求时，才会在请求头中带着，javaScript无法读写

Secure:非Https请求是不带

SameSite:用于定义cookie如何跨域请求

## CSRF概念

CSRF（Cross-site request forgery）：跨站请求伪造，是一种对网站的恶意利用，尽管听起来跟XSS跨站脚本攻击有点相似，但事实上CSRF与XSS差别很大，XSS利用的是站点内的信任用户，而CSRF则是通过伪装来自受信任用户的请求来利用受信任的网站。你可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义向第三方网站发送恶意请求。 CRSF能做的事情包括利用你的身份发邮件、发短信、进行交易转账等，甚至盗取你的账号。

## CSRF攻击原理

![](https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=310409039,637637879&fm=26&gp=0.jpg)

看图识意

1.用户C浏览并登陆受信任的站点A

2.登陆验证通过后，站点A返回给浏览器已经登陆的cookie，cookie 信息会在浏览器中保留一段时间

3.完成上一步之后，用户没有登出站点A，也没有清除浏览器的cookie，此时访问不安全站点B

4.这时站点B让用户访问站点A，并向站点A发出恶意请求，这个请求就会带上浏览器保留的站点A的cookie

5.站点A根据请求所带的cookie，判断此请求为用户C所发送的。

因此，站点A会根据用户C的全乡来处理恶意站点B所发起的请求，而这个请求可能以用户C的省份发送邮件、消息以及转账支付等操作，这样恶意站点B就达到了伪造用户C的请求站点A的目的。

受害者只要登陆受信任站点A，并在本地生成cookie,再不登出站点A以及清除站点A的cookie的情况下，访问恶意站点B，攻击者就可以完成CSRF攻击。

## 抽象举例

小明在一个理发店办了一张vip卡，下次去理发店理发的时候，一刷卡看到写的小明，就说是vip的会员小明啊，今天剪什么发型，可是有一天突然卡丢了被别人捡到了，别人拿着卡也去理发店，理发店人比较多，可能不知道每个人长什么样子，一刷卡就说你是vip小明啊，然后就可以花你理发卡的钱来理发。

## 代码举例

假设A网站以GET请求来发起支付请求，转账链接为`http://xxx.com/pay.do?accountId=123456&money=100` ，accountId为转账账户，参数money为转账金额，而B网站中有一个图片，地址并不是正常的图片地址，写的是`<img src="http://xxx.com/pay.do?accountId=1234567&money=100"/>`。这时候你登录网站A之后，没有点击登出，这时访问了B，你会发现的账号少了100块。

分析：你登录A网站的时候，浏览器保留了A网站的cookie，而当访问B的时候，img标签发起了一个http请求来获取图片资源，发起的请求是http://xxx.com/pay.do?accountId=1234567&money=100 ,并会带上A的cookie信息，结果A收到请求后，会认为是你自己的发挥的请求，你的账号就会少100。

## CRSF防御

**1、尽量使用POST，限制GET**  
GET接口太容易被拿来做CSRF攻击，看上面示例就知道，只要构造一个img标签，而img标签又是不能过滤的数据。接口最好限制为POST使用，GET则无效，降低攻击风险。  
当然POST并不是万无一失，攻击者只要构造一个form就可以，但需要在第三方页面做，这样就增加暴露的可能性。

**2、将cookie设置为HttpOnly**  
CRSF攻击很大程度上是利用了浏览器的cookie，为了防止站内的XSS漏洞盗取cookie,需要在cookie中设置“HttpOnly”属性，这样通过程序（如JavaScript脚本、Applet等）就无法读取到cookie信息，避免了攻击者伪造cookie的情况出现。  
在Java的Servlet的API中设置cookie为HttpOnly的代码如下：  
`response.setHeader( "Set-Cookie", "cookiename=cookievalue;HttpOnly");`

**3、增加token**  
CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于cookie中，因此攻击者可以在不知道用户验证信息的情况下直接利用用户的cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：**在请求中放入攻击者所不能伪造的信息，并且该信总不存在于cookie之中**。鉴于此，系统开发人员可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务端进行token校验，如果请求中没有token或者token内容不正确，则认为是CSRF攻击而拒绝该请求。  
假设请求通过POST方式提交，则可以在相应的表单中增加一个隐藏域：  
`<input type="hidden" name="_toicen" value="tokenvalue"/>`  
token的值通过服务端生成，表单提交后token的值通过POST请求与参数一同带到服务端，每次会话可以使用相同的token，会话过期，则token失效，攻击者因无法获取到token，也就无法伪造请求。  
在session中添加token的实现代码：

```csharp
HttpSession session = request.getSession();
Object token = session.getAttribute("_token");
if(token == null I I "".equals(token)) {
    session.setAttribute("_token", UUID.randomUUIDO .toString());
}
```

**4、通过Referer识别**  
根据HTTP协议，在HTTP头中有一个字段叫Referer，它记录了该HTTP请求的来源地址。在通常情况下，访问一个安全受限的页面的请求都来自于同一个网站。比如某银行的转账是通过用户访问`http://www.xxx.com/transfer.do`页面完成的，用户必须先登录`www.xxx.com`，然后通过单击页面上的提交按钮来触发转账事件。当用户提交请求时，该转账请求的Referer值就会是  
提交按钮所在页面的URL（本例为[www.xxx](https://link.jianshu.com/?t=http%3A%2F%2Fwww.xxx). com/[transfer.do](https://link.jianshu.com/?t=http%3A%2F%2Ftransfer.do)）。如果攻击者要对银行网站实施CSRF攻击，他只能在其他网站构造请求，当用户通过其他网站发送请求到银行时，该请求的Referer的值是其他网站的地址，而不是银行转账页面的地址。因此，要防御CSRF攻击，银行网站只需要对于每一个转账请求验证其Referer值即可，如果是以`www.xx.om`域名开头的地址，则说明该请求是来自银行网站自己的请求，是合法的；如果Referer是其他网站，就有可能是CSRF攻击，则拒绝该请求。  
取得HTTP请求Referer：  
`String referer = request.getHeader("Referer");`

## XSS概念

XSS(Cross Site Script)，中译是跨站脚本攻击；其原本缩写是 CSS，但为了和层叠样式表(Cascading Style Sheet)有所区分，因而在安全领域叫做 XSS。XSS 攻击是指攻击者在网站上注入恶意的客户端代码，通过恶意脚本对客户端网页进行篡改，从而在用户浏览网页时，对用户浏览器进行控制或者获取用户隐私数据的一种攻击方式。

攻击者对客户端网页注入的恶意脚本一般包括 JavaScript，有时也会包含 HTML 和 Flash。有很多种方式进行 XSS 攻击，但它们的共同点为：将一些隐私数据像 cookie、session 发送给攻击者，将受害者重定向到一个由攻击者控制的网站，在受害者的机器上进行一些恶意操作。

XSS攻击可以分为3类：反射型（非持久型）、存储型（持久型）、基于DOM。

### 反射性

反射型 XSS 只是简单地把用户输入的数据 “反射” 给浏览器，这种攻击方式往往需要攻击者诱使用户点击一个恶意链接，或者提交一个表单，或者进入一个恶意网站时，注入脚本进入被攻击者的网站。

我们在前端页面放置一个`<a href="localhost:8001/?q=111&p=222"/>`，恶意链接的地址指向了 localhost:8001/?q=111&p=222。然后，我再启一个简单的 Node 服务处理恶意链接的请求：

```
    const http =require('http');
    functionhandleReequest(req, res) {
    res.setHeader('Access-Control-Allow-Origin','*');
    res.writeHead(200,{'ContentType':'text/html;charset=UTF-});
    res.write('<script>alert("反射型 XSS 攻击")</script>');
    res.end();
    }
    const server =new http.Server();
    server.listen(8001,'127.0.0.1');
    server.on('request', handleReequest);
```

当用户点击恶意链接时，页面跳转到攻击者预先准备的页面，会发现在攻击者的页面执行了 js脚本，这样就产生了反射型 XSS 攻击。攻击者可以注入任意的恶意脚本进行攻击，可能注入恶作剧脚本，或者注入能获取用户隐私数据(如cookie)的脚本，这取决于攻击者的目的。

### 存储型

存储型 XSS 会把用户输入的数据 “存储” 在服务器端，当浏览器请求数据时，脚本从服务器上传回并执行。这种 XSS 攻击具有很强的稳定性。

比较常见的一个场景是攻击者在社区或论坛上写下一篇包含恶意 JavaScript 代码的文章或评论，文章或评论发表后，所有访问该文章或评论的用户，都会在他们的浏览器中执行这段恶意的 JavaScript 代码。

```
<script> alert("XSS 攻击")</script>//输入的恶意代码
```



### 基于DOM

基于 DOM 的 XSS 攻击是指通过恶意脚本修改页面的 DOM 结构，是纯粹发生在客户端的攻击。

```
<h2>XSS: </h2>
<input type="text" id="input">
<button id="btn">Submit</button>
<div id="div"></div>
<script>
const input = document.getElementById('input');
const btn = document.getElementById('btn');
const div = document.getElementById('div');
let val; 
input.addEventListener('change', (e) => {
val = e.target.value;
}, false);
btn.addEventListener('click', () => {
div.innerHTML = `<a href=${val}>testLink</a>`
}, false);
</script>
```

点击 Submit 按钮后，会在当前页面插入一个链接，其地址为用户的输入内容。如果用户在输入时构造了如下内容：

```javascript
onclick=alert(/xss/)
```

用户提交之后，页面代码就变成了：

```html
<a href onlick="alert(/xss/)">testLink</a>
```

此时，用户点击生成的链接，就会执行对应的脚本

## 举例

1.在网页 input 或者 textarea 中输入 <script>alert('xss')</script>或者其他脚本

2.直接使用 URL 参数攻击  
`https://www.baidu.com?jarttoTest=<script>alert(document.cookie)</script>`

## XSS攻击的防范

现在主流的浏览器内置了防范 XSS 的措施，例如 CSP。但对于开发者来说，也应该寻找可靠的解决方案来防止 XSS 攻击。

### HttpOnly 防止劫取 Cookie

将重要的cookie标记为httponly，这样的话当浏览器向Web服务器发起请求的时就会带上cookie字段，但是在js脚本中却不能访问这个cookie，这样就避免了XSS攻击利用JavaScript的document.cookie获取cookie

#### 输入过滤

不要相信用户的任何输入。 对于用户的任何输入要进行检查、过滤和转义。建立可信任的字符和 HTML 标签白名单，对于不在白名单之列的字符或者标签进行过滤或编码。

在 XSS 防御中，输入检查一般是检查用户输入的数据中是否包含 <，> 等特殊字符，如果存在，则对特殊字符进行过滤或编码，这种方式也称为 XSS Filter。

避免 XSS 的方法之一主要是将用户输入的内容进行过滤。对所有用户提交内容进行可靠的输入验证，包括对 URL、查询关键字、POST数据等，仅接受指定长度范围内、采用适当格式、采用所预期的字符的内容提交，对其他的一律过滤。(客户端和服务器都要)

#### 输出转义

用户的输入会存在问题，服务端的输出也会存在问题。一般来说，除富文本的输出外，在变量输出到 HTML 页面时，可以使用编码或转义的方式来防御 XSS 攻击。例如利用 sanitize-html 对输出内容进行有规则的过滤之后再输出到页面中。

1.往 HTML 标签之间插入不可信数据的时候，首先要做的就是对不可信数据进行 HTML Entity 编码[HTML 字符实体](http://www.w3school.com.cn/html/html_entities.asp)

```
function htmlEncodeByRegExp (str){
 var s = "";
 if(str.length == 0) return "";
 s = str.replace(/&/g,"&");
 s = s.replace(/</g,"<");
 s = s.replace(/>/g,">");
 s = s.replace(/ /g," ");
 s = s.replace(/\'/g,"'");
 s = s.replace(/\"/g,""");
 return s;
 }
var tmpStr="<p>123</p>";
 var html=htmlEncodeByRegExp (tmpStr)
console.log(html) //<p>123</p>
document.querySelector(".content").innerHTML=html; //<p>123</p>
```

当然，富文本还要更麻烦一些，因为要保留一部分标签和属性，要不然全变纯文本了，就不富了。这种情况一般通过黑名单进行过滤，或者白名单放行。即只允许一部分指定的标签和属性，其它的全部转义掉。

2.将用户数据输出到html 标签的属性时，必须经过标签属性的转义。注意：不包含href, src, style和事件处理函数属性（比如onmouseover）

编码：除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 &#xHH; （以&#x开头，HH则是指该字符对应的十六进制数字，分号作为结束符）

3.对动态生成的JavaScript代码，这包括脚本部分以及HTML标签的事件处理属性（Event Handler，如onmouseover, onload）等进行Javascript编码

```
var JavaScriptEncode = function(str){
     
    var hex=new Array('0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f');
        
    function changeTo16Hex(charCode){
        return "\\x" + charCode.charCodeAt(0).toString(16);
    }
    
    function encodeCharx(original) {
        
        var found = true;
        var thecharchar = original.charAt(0);
        var thechar = original.charCodeAt(0);
        switch(thecharchar) {
            case '\n': return "\\n"; break; //newline
            case '\r': return "\\r"; break; //Carriage return
            case '\'': return "\\'"; break;
            case '"': return "\\\""; break;
            case '\&': return "\\&"; break;
            case '\\': return "\\\\"; break;
            case '\t': return "\\t"; break;
            case '\b': return "\\b"; break;
            case '\f': return "\\f"; break;
            case '/': return "\\x2F"; break;
            case '<': return "\\x3C"; break;
            case '>': return "\\x3E"; break;
            default:
                found=false;
                break;
        }
        if(!found){
            if(thechar > 47 && thechar < 58){ //数字
                return original;
            }
            
            if(thechar > 64 && thechar < 91){ //大写字母
                return original;
            }

            if(thechar > 96 && thechar < 123){ //小写字母
                return original;
            }        
            
            if(thechar>127) { //大于127用unicode
                var c = thechar;
                var a4 = c%16;
                c = Math.floor(c/16); 
                var a3 = c%16;
                c = Math.floor(c/16);
                var a2 = c%16;
                c = Math.floor(c/16);
                var a1 = c%16;
                return "\\u"+hex[a1]+hex[a2]+hex[a3]+hex[a4]+"";        
            }
            else {
                return changeTo16Hex(original);
            }
            
        }
    }     
    var preescape = str;
    var escaped = "";
    var i=0;
    for(i=0; i < preescape.length; i++){
        escaped = escaped + encodeCharx(preescape.charAt(i));
    }
    return escaped;
}
```

4.将不可信数据插入到HTML URL里时，对这些数据进行URL编码

编码:除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 %HH （以 % 开头，HH则是指该字符对应的十六进制数字）

```
对URI使用encodeURI()
对参数使用encodeURIComponent()
```

对URI使用encodeURI()  
对参数使用encodeURIComponent()

## 总结

现代web开发框架如vue.js、react.js等，在设计的时候就考虑了XSS攻击对html插值进行了更进一步的抽象、过滤和转义，我们只要熟练正确地使用他们，就可以在大部分情况下避免XSS攻击.此篇借鉴比较多哈，总结了下，放出借鉴链接

链接1:[https://www.jianshu.com/p/67408d73c66d](https://www.jianshu.com/p/67408d73c66d)

链接2:[https://www.cnblogs.com/yangsg/p/10621496.html](https://www.cnblogs.com/yangsg/p/10621496.html)
