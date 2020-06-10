---
title: Cookie, Session是如何保持登录状态的
date: 2020-06-10 08:36:22
comments: true
categories: 
  - ['技术','HTTP']
tags:
  - HTTP
---
首先，http是一个无状态的协议，它自生不对请求和响应之间的通信状态进行保存

服务器并不会对每个人做特殊处理，服务器只对请求负责，并不对请求的人负责。

cookie 和 session 都是帮助http进行状态管理的一种手段

### cookie 对比 session
1. 保存位置
cookie保存在用户的客户端，session保存在服务端
2. 生命周期
cookie由用户指定或使用默认的过期时间，在这段时间cookie都保存在客户端
session属于一次对话，如果会话关闭，浏览器关闭，服务器启动都会导致session的清除。
3. 数据类型
cookie是一堆字符串， session是某种object
4. 安全性
cookie一般只保存一些用户的行为习惯，一些密码都是经过加密的，即使泄漏也无关紧要
session则保存用户相关的内容

Last-Modified 该资源最终修改的时间

- cookie 
  存储在用户本地终端的数据，用来存储用户信息
  参数：
  expires/Max-age(过期时间):  cookie的有效期(若不指定则默认浏览器关闭为止)
  path：设置cookie在哪些路径范围内有效
  secure：为true只能在https获得
  HTTPOnly：使得js脚本无法获得Cookie，主要目的是为了防止跨站脚本攻击(XSS).
  这样使用js的document.cookie无法读取附加过后的Cookie的内容
  1. 删除cookie
    cookie一旦从服务器端发送至客户端，服务端就无法删除Cookie了，但是可以通过覆盖一起的cookie，修改过期时间为过去的时间达到删除cookie的方法
- session
  当服务器在为某个客户端请求创建一个session的时候，服务器会首先检索客户端的请求是否已包含一个session标识session.id
  如果检索出来，服务器会使用这次的session识别出来的用户，否则就会创建一个新的session并生成一个session.id字段。
问题&Q ：
  如何把session.id传递给服务器呢？    -》 url重写

回答&A :
  1. 作为url的附加路径
    'http://..../xxx;jsessionid=abcdefjijeoijoifjioe'
  2. 作为url的查询字段
    'http://..../xxx?jsessionid=abcdefjijeoijoifjioe'


### cookie和localStorage，sessionStorage的区别
1. 存储大小
cookie的数据大小不能超过4k
虽然localStorage, sessionStorage也有存储大小的限制，但是比cookie大的多
2. 有效时间
localStorage: 数据永久存储，浏览器关闭不会删除数据，除非主动删除数据
sessionStorage: 一次会话的数据，在浏览器关闭后会被删除
cookie：在其设置的有效期之内一直有效 expires/Max-age(过期时间)
3. sessionStorage
  1. 会话级别的存储
  2. 临时性的，页面打开有，页面关闭没有
  3. 数据不共享
  4. 通过a标签来跳出一个页面，则sessionStorage共享
4. localStorage
  1. 永久性的存储
  2. 持久化的数据存储
  3. 可以数据共享
  4. 不能跨域
5. cookie
  1. 可以设置的cookie的有效期max-age，在有效期内一直有效
  2. 在同源内且符合path规则的路径中都有效
  3. 如果设置的max-age为0，则表示删除该cookie
  4. 如果设置的max-age为负数，则表示该cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该cookie即失效。

### csrf(跨站请求伪造)
1. 原理
  用户在某个重要网站登入期间，即cookie还未过期时间，又不小心进入某个攻击网站，攻击网站会获取那个重要网站的cookie，然后在cookie未过期的时间，攻击者利用cookie登入到那个重要网站，这样被会误以为是原用户进入的，造成财产损失。
2. 防御
  1. 检查Referer字段
  在HTTP检查Referer字段，通常应该为同一域名下的请求, 如果不在同一域名下，服务器能够识别为恶意访问

  2. 添加token验证

  开发者可以在开发时在HTTP请求中添加一个token中间验证，并在服务端设置一个拦截器来验证这个token，如果没有或者验证失败，都会被认为是csrf攻击而拒绝请求

### xss(跨站脚本攻击)
1. 原理
  利用js脚本来动态获取用户的cookie信息
  监听用户行为，比如输入账号密码后直接发送到黑客服务器。
  修改 DOM 伪造登录表单。
  在页面中生成浮窗广告。
2. 防御
  1. 无论是前端还是服务端，都要对用户的输入进行转码和过滤
  2. 很多的Xss攻击都是窃取cookie的值，在设置了HttpOnly后，这样就无法获得cookie值
  3. 利用CSP(浏览器的安全策略)
    例如知乎跳转到其他页面时会提示用户保护财产安全等
问题&Q: 对Cookie了解多少

Cookie是为了解决http协议缺陷产出的，http协议是一个无状态的协议，它每次请求的独立，不相关的，默认不保留状态信息，而cookie就是设计来保存客户端的状态信息的。
在相同域名下发送请求时，都会携带cookie，服务器拿到cookie进行解析，拿到客户端的状态

缺陷：
  1. 容量缺陷。cookie的最大容量为4kb，存储的信息没有localStorage和sessionStorage大，它俩都为5MB
  2. 性能缺陷。在域名内的地址内发送请求，都会发送完整的cookie，这样随着请求增多的时候，非常消耗性能
  3. 安全缺陷。cookie在未设置HTTPOnly属性的时候，cookie能由js动态获取，用户信息会丢失

- localStorage
  存储方式：是存储本地，永久性储存，不主动删除的话不会消失。
  容量：容量很大，有5MB。
  安全：因为是存储在本地，默认不参与服务端的通信，这样就会避免cookie的安全性问题

- sessionStorage
  存储方式：是会话级别的存储，在会话期间存储，会话结束后被删除
  容量：容量很大，为5MB
  安全：相比于cookie还是更加安全
