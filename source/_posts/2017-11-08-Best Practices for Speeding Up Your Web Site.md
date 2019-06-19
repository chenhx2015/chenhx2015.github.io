---
title: Web站点加速最佳实践
date: 2017-11-08 11:29:07
tags: 前端性能
categories: frontend
---

&emsp;&emsp;在[高性能 Web 建设指南摘要](/2017/09/06/高性能Web建设指南摘要)总结了前端性能优化的 14 条原则，在高性能 Web 进阶指南中，也由一些建议。几年过去了，Yahoo 的 Exceptional Performance team 总结的规则也发展到 35 条。[原文地址](https://developer.yahoo.com/performance/rules.html)。规则分为七类：页面内容，服务器，cookie，css，javascript，图片和手机端。
&emsp;&emsp; 根据原文翻译整理了一下，记录了一些新的原则。感谢前端工具的发展，如 webpack，eslint，其中很多优化原则都可以让工具自动实施。其中原来的黄金规则可参看[高性能 Web 建设指南摘要](/2017/09/06/高性能Web建设指南摘要)

<!-- more -->

## 内容

### 减少 DOM 元素数量

&emsp;&emsp; 检查文档 dom 数量的简单办法，在浏览器控制台运行下一行代码:
document.getElementsByTagName('\*').length

### 尽量减少使用 iframe

iframe 帮助第三方内容嵌入，帮助脚本并行下载，但是使用它的代价很大，它会阻塞页面加载

### 不使用 404

一个无用的响应(例如 404 Not Found) 完全没有用处，只会降低用户体验。有些站点使用诸如"Did you mean X?"的 404 响应, 对用户体验有帮助但是浪费了服务器资源(例如数据库等). 如果链接是一个外部的脚本，而返回的响应是 404，尤其糟糕：首先, 脚本下载会阻塞并行下载，其次如果 404 响应的 body 是 JavaScript 代码，浏览器会去解析脚本。

## 服务器

### Flush the Buffer Early

When users request a page, it can take anywhere from 200 to 500ms for the backend server to stitch together the HTML page. During this time, the browser is idle as it waits for the data to arrive. In PHP you have the function flush(). It allows you to send your partially ready HTML response to the browser so that the browser can start fetching components while your backend is busy with the rest of the HTML page. The benefit is mainly seen on busy backends or light frontends.

A good place to consider flushing is right after the HEAD because the HTML for the head is usually easier to produce and it allows you to include any CSS and JavaScript files for the browser to start fetching in parallel while the backend is still processing.

Example:

      ... <!-- css, js -->
    </head>
    <?php flush(); ?>
    <body>
      ... <!-- content -->

### Use GET for AJAX Requests

The Yahoo! Mail team found that when using XMLHttpRequest, POST is implemented in the browsers as a two-step process: sending the headers first, then sending data. So it's best to use GET, which only takes one TCP packet to send (unless you have a lot of cookies). The maximum URL length in IE is 2K, so if you send more than 2K data you might not be able to use GET.

An interesting side affect is that POST without actually posting any data behaves like GET. Based on the HTTP specs, GET is meant for retrieving information, so it makes sense (semantically) to use GET when you're only requesting data, as opposed to sending data to be stored server-side.

### 避免 Image 元素的 src 属性为空

无论是 html 代码中的`<img src="">`或是通过 JavaScript

```
var img = new Image();
img.src = "";
```

都导致同样结果: 浏览器多发出一个 http 请求：

- Internet Explorer 会请求页面目录
- Safari 和 Chrome 会请求页面.
- Firefox 3 以及之前的版本如同 Safari 和 Chrome, 不过 3.5 不再发出请求.
- Opera 不会发出请求.

Why is this behavior bad?

Cripple your servers by sending a large amount of unexpected traffic, especially for pages that get millions of page views per day.
Waste server computing cycles generating a page that will never be viewed.
Possibly corrupt user data. If you are tracking state in the request, either by cookies or in another way, you have the possibility of destroying data. Even though the image request does not return an image, all of the headers are read and accepted by the browser, including all cookies. While the rest of the response is thrown away, the damage may already be done.

HTML5 增加了对 to the description of the tag's src attribute to instruct browsers not to make an additional request

The src attribute must be present, and must contain a valid URL referencing a non-interactive, optionally animated, image resource that is neither paged nor scripted. If the base URI of the element is the same as the document's address, then the src attribute's value must not be the empty string.
Hopefully, browsers will not have this problem in the future. Unfortunately, there is no such clause for `<script src="">` and `<link href="">` Maybe there is still time to make that adjustment to ensure browsers don't accidentally implement this behavior.
This rule was inspired by Yahoo!'s JavaScript guru Nicolas C. Zakas. For more information check out his article "Empty image src can destroy your site".

## Cookie

### 减少 Cookie 大小

### 静态内容使用 Cookie-free 域名

&emsp;&emsp; 浏览器请求静态图片等内容时候，Web 服务器一般并不使用 Cookie。因此，静态内容的主机名称使用 cookie-free 域名，浏览器就不会发送 cookie，可减少网络流量。

### 避免使用 CSS Filter

&emsp;&emsp; The IE-proprietary AlphaImageLoader filter aims to fix a problem with semi-transparent true color PNGs in IE versions < 7. The problem with this filter is that it blocks rendering and freezes the browser while the image is being downloaded. It also increases memory consumption and is applied per element, not per image, so the problem is multiplied.

&emsp;&emsp; The best approach is to avoid AlphaImageLoader completely and use gracefully degrading PNG8 instead, which are fine in IE. If you absolutely need AlphaImageLoader, use the underscore hack \_filter as to not penalize your IE7+ users.

### Choose <link> over @import

&emsp;&emsp; One of the previous best practices states that CSS should be at the top in order to allow for progressive rendering.

&emsp;&emsp; In IE @import behaves the same as using <link> at the bottom of the page, so it's best not to use it.

## Javascript

### Minimize DOM Access

&emsp;&emsp; Accessing DOM elements with JavaScript is slow so in order to have a more responsive page, you should:

Cache references to accessed elements
Update nodes "offline" and then add them to the tree
Avoid fixing layout with JavaScript
For more information check the YUI theatre's "High Performance Ajax Applications" by Julien Lecomte.

### Develop Smart Event Handlers

&emsp;&emsp; Sometimes pages feel less responsive because of too many event handlers attached to different elements of the DOM tree which are then executed too often. That's why using event delegation is a good approach. If you have 10 buttons inside a div, attach only one event handler to the div wrapper, instead of one handler for each button. Events bubble up so you'll be able to catch the event and figure out which button it originated from.

You also don't need to wait for the onload event in order to start doing something with the DOM tree. Often all you need is the element you want to access to be available in the tree. You don't have to wait for all images to be downloaded. DOMContentLoaded is the event you might consider using instead of onload, but until it's available in all browsers, you can use the YUI Event utility, which has an onAvailable method.

## 图片

### Optimize CSS Sprites

Arranging the images in the sprite horizontally as opposed to vertically usually results in a smaller file size.
Combining similar colors in a sprite helps you keep the color count low, ideally under 256 colors so to fit in a PNG8.
"Be mobile-friendly" and don't leave big gaps between the images in a sprite. This doesn't affect the file size as much but requires less memory for the user agent to decompress the image into a pixel map. 100x100 image is 10 thousand pixels, where 1000x1000 is 1 million pixels

### 不要在 HTML 中缩放图片

&emsp;&emsp; 例如

```
<img width="100" height="100" src="mycat.jpg" alt="My Cat" />
```

&emsp;&emsp; 图片 mycat.jpg 就应该是 100x100px，而不是一个 500x500px.

### favicon.ico

&emsp;&emsp;浏览器每次访问网站，都会自动请求，因此最好是在根目录创建一个 favicon.ico，同时尽可能缓存

- 尽量小于 1K.
- 设置 Expires 头

## 手机端

### 组件(译者注：指的是 html 页面，css，js，图片等等页面内容文件)的大小不要超过 25K

iPhone 不会缓冲大于 25K 的内容。 值得注意的是 25k 是指解压后的文件大小，单独使用 gzip 不起作用，因此减少文件的大小很重要。

详情可参看 Wayne Shea 和 Tenni Theurer 写的["Performance Research, Part 5: iPhone Cacheability - Making it Stick"](http://yuiblog.com/blog/2008/02/06/iphone-cacheability/).

### Pack Components into a Multipart Document

把内容组件打包到一个 multipart 文档，就像带附件的电子邮件, 有助于使用一次 HTTP 请求，获取多个内容组件。使用这个技巧前, 首先检查一下客户端的 user agent 是否支持(iPhone 不支持)。
