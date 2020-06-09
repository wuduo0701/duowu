---
title: 能不能说下前端缓存？
date: 2020-06-09 23:06:58
categories: 
  - ['技术','前端性能']
  - ['技术','HTTP']
tags:
  - HTTP
  - 缓存
  - 前端性能
---
浏览器的缓存机制：强缓存和协商缓存

### 强缓存

浏览器的缓存作用分为两种情况，一种是需要发送HTTP请求的，一种是不需要发送的。
1. 不需要发送HTTP请求的
首先是检查强缓存，这个阶段是不需要发送HTTP请求的。
问题&Q: 如何检查强缓存呢?
回答&A: 通过检查相应的字段。
  + 在HTTP/1.0中，检查的是Expires
  + 在HTTP/1.1中，检查的是Cache-control

#### Expires  过期时间
存在于服务器中的返回头中，告诉浏览器在这个过期时间端内可以直接从缓存获取数据
```js
  Expires: Wed, 22 Nov 2019 08:41:00 GMT
```

缺点：服务器的时间和浏览器的时间可能不一致，那返回的时间就是不准确的了。因此在HTTP/1.1被抛弃了

#### Cache-Control
这个用来控制的不是具体的时间点，而是采用过期时长来控制缓存，对应的缓存应该是max-age
```js
  Cache-Control:max-age=3600
```
public:  浏览器经过到达源服务器的中间任意缓存服务器都可以缓存
private： 这种情况就是只有浏览器能缓存了，中间的代理服务器不能缓存。
no-cache: 跳过当前的强缓存，发送HTTP请求，即直接进入协商缓存阶段。
no-store：非常粗暴，不进行任何形式的缓存。
s-maxage：这和max-age长得比较像，但是区别在于s-maxage是针对代理服务器的缓存时间。
must-revalidate: 是缓存就会有过期的时候，加上这个字段一旦缓存过期，就必须回到源服务器验证。

❗ 注意：值得注意的是，当Expires和Cache-Control同时存在的时候，Cache-Control会优先考虑。


当缓存时间都超时了，这时候就需要到协商缓存。

### 协商缓存

强缓存失效后，浏览器在请求头中携带 **缓存tag** 相应的缓存服务器发送请求，由服务器根据这个tag，来决定是否使用缓存，这就是协商缓存。

协商缓存分为两种：Last-Modified 和 ETag

#### Last-Modified
即资源的最终的修改时间。浏览器第一次给服务器发送请求后，服务器会在请求投中加上这个字段。

了解这个还需要了解另外一个字段：**If-Modified-Since**
```js
If-Modified-Since:  Thu, 15 Apr 2004 00:00:00 GMT
```
表示是在2004.4.15更新后的时间

此时如果 **Last-Modified** 字段为
```js
Last-Modified:  Sun, 29 Aug 2004 00:00:00 GMT
```
表示资源的最后的修改时间为：2004.8.29
- 总结：
如果**If-Modified-Since**的时间小于**Last-Modified**的时间，表示是时候更新了
否则直接返回304状态码， 告诉浏览器直接用缓存

#### ETag
将资源(文件)以字符串的形式给文件生成唯一标识，当资源发生改变时，这个值就会改变。

在此之前还需要了解另外一个字段: **If-None-Match**

浏览器在收到ETag的值时，会在下次请求时会将这个值作为If-None-Match这个字段的内容，然后会与服务器上的ETag值进行对比。
- 如果不一样时，说明资源更新了，需要返回新的资源，然后需要发送新的http请求获取最新资源
- 一样时，返回状态码304，告诉浏览器直接用缓存

### Last-Modified 与 ETag 对比
1. 在精度上。
ETag优于Last-Modified.因为ETag时按照内容给资源上标识，因此能准确感知资源的变化。Last-Modified在一些特殊的情况并不能准确感知资源变化。
  - 编辑了资源文件，但是文件内容并没有更改，这样也会造成缓存失效。
  - Last-Modified 能够感知的单位时间是秒，如果文件在 1 秒内改变了多次，那么这时候的 Last-Modified 并没有体现出修改了。

2. 在性能上。
Last-Modified优于ETag，也很好理解，Last-Modified仅仅只是记录一个时间点，而Etag需要根据文件的具体内容生成哈希值。

❗ 注意：如果两种方式都支持的话，服务器会优先考虑ETag。


### 缓存位置
当Last-Modified 或者 ETag返回304状态时，我们需要从缓存中拿去资源，可是缓存的资源放在哪呢？
浏览器中的缓存位置一共有四种，按优先级从高到低排列分别是：

- Service Worker
- Memory Cache
- Disk Cache
- Push Cache

#### Service Worker
Service Worker是PWA的重要实现机制。
- 原理
让 JS 运行在主线程之外，由于它脱离了浏览器的窗体，因此无法直接访问DOM
- 用法
官方的React框架的时候会生成一个serveice-worker.js。他在离线缓存、消息推送和网络代理等起到了很大的作用

#### Memory Cache 和 Disk Cache
即内存缓存和磁盘缓存
效率上Memory Cache比Disk Cache快。存储容量和存储时长Disk Cache更有优势。

#### Push Cache
即推送缓存，这是浏览器缓存的最后一道防线。它是 HTTP/2 中的内容，虽然现在应用的并不广泛，但随着 HTTP/2 的推广，它的应用越来越广泛。


### 总结：
浏览器的缓存机制：首先会对Cache-cache或者Expires字段进行检查(这里Expires就不进行考虑了，因为他是Http1.0，已经被淘汰)，判断有无强缓存，如果强缓存可以用，就直接使用，否则就进入协商缓存阶段。
在协商缓存中，浏览器会发送http请求，服务器会通过请求头中的If-Modified-Since与Last-Modified字段或者If-None-Match与ETag字段进行比较，判断资源是否更新。
  - 若资源更新，则返回新的资源和状态码200
  - 否则，返回状态304，告诉浏览器直接从缓存中获取。