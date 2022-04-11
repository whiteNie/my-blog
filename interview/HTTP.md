# HTTP协议学习笔记

 学习思路：个人认为学习一个东西，我们最先需要了解的是他的组成结构。就像建房子一样，我们需要知道我们***要建造的房子***会由哪些组成，然后再去逐一了解。简而言之就是观全局再究其细节。
---
## HTTP请求
>一个HTTP请求报文由请求行、请求头、请求体构成。
### HTTP请求结构
---
   #### 请求行
   + 作用及组成

            请求行的作用是声明请求方法，主机域名，资源路径，协议版本。
            请求行的组成：请求方法+空格+请求路径+空格+协议版本+\r\n
   + 请求行示例

            `请求行示例:POST 　/index.php　HTTP/1.1 　　 `
   + 请求行图解
            ![avatar](http://qiniuyun.whitenip.site/image/blog/http/%E8%AF%B7%E6%B1%82%E8%A1%8C%E7%BB%84%E6%88%90.png)
      从上面的内容可以看出HTTP请求的组成的第一部分请求行的组成是很简单的。但是我们还是需要清楚的知道请求方法、请求路径、协议版本。
            ![avatar](http://qiniuyun.whitenip.site/image/blog/http/%E8%AF%B7%E6%B1%82%E8%A1%8C%E7%BB%84%E6%88%90%E8%AF%A6%E7%BB%86%E4%BB%8B%E7%BB%8D.png)
      注：GET和POST的区别
            ![avatar](http://qiniuyun.whitenip.site/image/blog/http/GET%E5%92%8CPOST%E7%9A%84%E5%8C%BA%E5%88%AB.png)
      
      > 这里有一点需要说明:还需要理解URL的结构
---
   #### 请求头
   + 作用

      声明客户端、服务端部分报文信息。
   +  请求报文的信息有很多，以下为我在项目中随机抓取的Http数据包，并附上解释
      ![avatar](http://qiniuyun.whitenip.site/image/blog/http/%E8%AF%B7%E6%B1%82%E5%A4%B4%E4%BF%A1%E6%81%AF.png)
      
      ```
      请求头：声明客户端、服务端报文的部分信息
      Host:主机，接收请求的服务器地址，可以是ip+port，也可以是域名
      User-Agent:用户标识，如：Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36
      Accept:告诉服务器，客户端可接受的类型，可以是多个值，用逗号分隔，如：application/json, text/plain, */*
      Content-Type:请求体、响应体的类型，如：application/json
      Content-Encoding:请求体、响应体的编码格式,如：gzip、deflate
      Content-Length:请求体、响应体的长度
      Referer:标识请求引用于哪个地址，如：页面A(http://218.67.246.252:83/enterprise/front/inspection/specialcheck/specialrecord
      )跳转到页面B,则此时Referer对应的值为http://218.67.246.252:83/enterprise/front/inspection/specialcheck/specialrecord
      Accept-Encoding:告诉服务器，客户端可接受的数据压缩包编码格式，如：gzip、deflate
      Accept-Language:告诉服务器，客户端可接受的语言，如：zh-CN,zh;q=0.9
      Cookie:本地缓存
      Connection:指定与连接相关的属性，如：Keep-Alive
      注：当HTTP请求方法为GET请求时，是不会有请求体，按照HTTP规范，请求参数是以?开始连接，以&&分隔拼接在请求路径之后的。
      ```
   #### 请求体
   只有为非幂等性请求时才会存在,用来存放发送给服务器的数据信息。
   ![avatar](http://qiniuyun.whitenip.site/image/blog/http/%E8%AF%B7%E6%B1%82%E4%BD%93%E7%9A%84%E4%B8%89%E7%A7%8D%E5%BD%A2%E5%BC%8F.png)

   #### 报文示例
>POST请求报文示例1：
```
POST 　/index.php　HTTP/1.1 　　 请求行
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:10.0.2) Gecko/20100101 Firefox/10.0.2　　请求头
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,/;q=0.8
Accept-Language: zh-cn,zh;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://localhost/
Content-Length：25
Content-Type：application/x-www-form-urlencoded
　　空行
username=aa&password=1234　　请求数据
```
>GET请求报文示例2：
```
GET /enterprise/api/inspection/selfPlan?curPage=1&pageSize=10 HTTP/1.1
Host	218.67.246.252:83
Accept	application/json, text/plain, */*
Access-Token	22ef1c73883f48cbacbb72631f03e4f8
User-Agent	Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36
Referer	http://218.67.246.252:83/enterprise/front/inspection/selfcheck/checkplan
Accept-Encoding	gzip, deflate
Accept-Language	zh-CN,zh;q=0.9
Cookie	ENTERPRISE_SESSIONID=0AADD0D48DBA976295B17BA01A5F0C5B
Connection	keep-alive
```
---
## HTTP响应
>一个HTTP响应报文由状态行、响应头和响应体构成
### HTTP响应的组成结构
#### 状态行
+ 结构 

   协议版本+空格+状态码+空格+状态描述+\r\n

+ 示例

   HTTP/1.1 404 Not Found

>关于HTTP状态码的补充：分为以下五大类
---
状态码|状态码含义|示例
-:|:-:|:-
1xx|信息通知，如请求收到了或者正在进行处理|
2xx|成功，如接受或者知道了|200：请求成功。202请求被接受但还没处理
3xx|重定向，如要完成请求还需要进行进一步的行动|301：请求的资源已被永久的移动到新的位置。302请求的资源被临时移动到新的位置。
4xx|客户端错误|401：请求需要验证用户。403：不允许访问该地址。404：资源未找到。408：请求超时
5xx|服务端错误|500：服务器内部错误。502：Bad Gateway网关错误。
---
>***补充：304：告诉客户端使用缓存资源即可，因为服务器上的资源没有更新。 所以304意味着在获取资源的时候，仍旧会发起一次请求到服务器，服务器确认 If-None-Match 和资源对比，没有发生变化，随即返回客户端一个304响应，告诉客户端使用缓存的资源即可。***
#### 响应头
响应体的信息和请求头的信息类似，都是以键值的形式展示，用来声明客户端、服务端的部分报文信息。


#### 响应体