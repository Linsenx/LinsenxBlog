---
title: CORS-跨域资源共享详解
date: 2020-04-06 17:17:56
tags: [CORS]
thumbnail: http://image.linsenx.com/blog/2020-04-06-fantasy-2543658_1920.jpg?imageMogr2/format/webp/blur/1x0/quality/75%7Cimageslim
---

## 1.介绍

跨域资源共享定义了一种方式，使浏览器和服务器之间能互相确认是否能够安全的使用跨域请求。

以下场景遵循跨域资源共享标准：

- 由 `XMLHttpRequest` 或 `Fetch` 发起的跨域 HTTP 请求

- Web 字体（CSS中通过 `@font-face` 使用跨域字体资源）

- 使用 `drawImage` 将 imgage/video 绘制到 canvas 中

- WebGL 纹理加载

  <!--more-->

## 2.工作原理

CORS需要浏览器和服务器的共同支持：

1、定义了一组 HTTP 响应头字段，允许服务器声明哪些源站通过浏览器有权访问哪些资源：

- Access-Control-Allow-Origin
- Access-Control-Allow-Method
- Access-Control-Allow-Headers
- Access-Control-Allow-Credentials
- Access-Control-Expose-Headers：控制 XMLHttpRequest.getResponseHeader(headerName) 能访问到哪些响应头

除了上述响应头，还有 Access-Control-Max-Age 响应头表示预检『预检请求』的缓存时间（单位为秒）。

2、定义了一组 HTTP 请求头字段，要求浏览器在发送对服务器可能产生副作用的请求前，必须先使用 OPTIONS 方法发起『预检请求』，服务器允许之后，再发起『主请求』：

- Access-Control-Request-Method
- Access-Control-Request-Header

## 3.CORS请求的流程

将 CORS 请求分为两类：『简单请求』和『非简单请求』。

某些请求不会触发预检请求，称为『简单请求』，满足以下两大条件：

1、请求方法是以下三种方法之一：

- HEAD
- GET
- POST

2、HTTP的头信息不超出以下几种字段：

- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Tpye: 只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain

简单请求的流程：

1. 浏览器在 HTTP 请求头中添加一个 `Origin` 字段。
2. 若服务器认定该源站不在许可范围内，服务器会返回正常的 HTTP 响应，但是响应头中不包含  `Access-Control-Allow-Origin` 字段。
3. 浏览器根据 HTTP 响应头中是否包含 `Access-Control-Allow-Origin` 字段，若不包含则证明跨域请求失败，浏览器抛出相应的错误信息。

以下是非简单请求的请求流程：

![](http://image.linsenx.com/blog/2020-04-06-Untitled.png)

1. 在发起真正产生作用的请求前，浏览器发送『预检请求』并在 HTTP 请求头中添加 `Origin` 字段；并添加 `Access-Control-Request-Method` 描述主请求的方法；以及添加 `Access-Control-Request-Header` 描述用到的额外请求头字段。
2. 若服务器认定该源站不在许可范围内，服务器会返回正常的 HTTP 响应，但是响应头中不包含`Access-Control-Allow-Origin` 、`Access-Control-Allow-Method` 等字段。
3. 浏览器根据 HTTP 响应头中是否包含 `Access-Control-Allow-Origin` 、`Access-Control-Allow-Method` 等字段，若不包含则证明『预检请求』失败，浏览器抛出相应的错误信息。若『预检请求』成功，则发送『主请求』。

## 4.crossorigin属性

`<img>`、`<script>`、`<link>`、`<audio>`、`<video>` 元素具有 crossorigin 属性。

若不指定 crossorigin 属性，则完全不使用 CORS，但是会在部分能力上受到限制。

**但也存在例外，模块脚本（`<script type="module">`）无论是否指定 crossorigin 属性都会强制使用 CORS。**

#### crossorigin的取值及作用：

若指定了 crossorigin 属性，获取跨域资源时会使用 CORS，并可以控制获取跨域资源时是否携带 cookie 、HTTP Basic authentication 等凭据信息。

- anonymous: 同源时才携带 cookie
- use-credentials: 无论是否跨域，总是携带 cookie**（此时跨域资源返回的 `Access-Control-Allow-Origin` 不能是 `*` ，否则会抛出错误）**
- "": 等同 anonymous

**在跨站允许跨域的情况下，cookie 的携带也受到 SameSite 属性的限制。**

指定 crossorigin 属性除了控制是否发送 cookie 外，对以下元素还有额外的作用：

- [img](https://html.spec.whatwg.org/multipage/embedded-content.html#attr-img-crossorigin)：控制跨域图像资源是否能被用于 `canvas`。
- [script](https://html.spec.whatwg.org/multipage/scripting.html#attr-script-crossorigin)：对于经典脚本，可以控制跨域时是否暴露错误信息。

## 5.XHR和Fetch发起跨域请求

使用 `XMLHttpRequest` 发起跨域 HTTP 请求时，如果想要携带 cookie 必须指定 `withCrendential = true` 。

使用 `Fetch` 发起跨域 HTTP 请求时，想要携带 cookie 则必须指定 `credentials` 属性，`credentials` 有以下几种取值：

- omit: 从不发送cookies
- same-origin: 同源时才携带 cookie
- include: 无论是否跨域，总是携带 cookie**（此时跨域资源返回的 `Access-Control-Allow-Origin` 不能是 `*` ，否则会抛出错误）**

## 6.跨域脚本错误

对经典脚本来说，若不指定 crossorigin 属性则完全不使用 CORS，这也是 JSONP 实现的原理。

但是在不使用 CORS 的情况下，跨域脚本中的错误信息不会暴露给 `window.onerror` 。

**而模块脚本强制使用了 CORS。也就是说，跨域的模块脚本必须返回带有有效的 `Access-Control-Allow-Origin` 响应头。**

下面是一个使用跨域经典脚本但是没有指定 crossorigin 属性的例子：

```html
<!doctype html>
<html>
<head>
  <title>example.com/test</title>
</head>
<body>
  <script src="http://another-domain.com/app.js"></script>
  <script>
  window.onerror = function (message, url, line, column, error) {
    console.log(message, url, line, column, error);
  }
  foo(); // call function declared in app.js
  </script>
</body>
</html>
```

```javascript
// another-domain.com/app.js
function foo() {
  bar(); // ReferenceError: bar is not a function
}
```

运行该HTML后，通过 window.onerror 中的 console 将输出以下信息：

```javascript
"Script error.", "", 0, 0, undefined
```

这并不是真正的错误信息，出于安全考虑，浏览器有意的隐藏了没有使用 CORS 的跨域脚本的错误信息，这是为了避免无意间把潜在的敏感信息暴露给了它无法控制的 onerror 回调。

## 7.跨域脚本如何暴露错误信息

#### 方法一

一、给 script 标签添加 `crossorigin="anonymous"`

```html
<script src="http://another-domain.com/app.js" crossorigin="anonymous"></script>
```

二、服务器给跨域脚本添加响应头

```javascript
Access-Control-Allow-Origin: http://another-domain.com
```

#### 方法二

上面这种解决方案需要浏览器和服务器的配合，但在某些情况下，我们没有办法去调整服务器返回的响应头，在这个情况下，还有另外一种方案：使用 try/catch。

回到刚才的例子，这次我们使用了 try/catch 。

    <script src="http://another-domain.com/app.js"></script>
    <script>
    try {
      foo(); // call function declared in app.js
    } catch (e) {
      console.log(e);
    }
    </script>
    
    // another-domain.com/app.js
    function foo() {
      bar(); // ReferenceError: bar is not a function
    }

运行该HTML后，将输出以下 console 信息：

    => ReferenceError: bar is not defined
         at foo (http://another-domain.com/b.js:2:3)
         at http://example.com/test/:15:3

该 console 来自于 try/catch，可以看到输出了完整的错误信息。因此我们可以使用在调用跨域脚本中的函数时可以通过包裹一层 try/catch 来捕获完整的错误信息。

## 8.Canvas 使用跨域图像

若使用跨域图像资源时不指定 crossorigin 属性，任可以使用 `drawImage` 将图像绘制到 canvas。但是 canvas 会“被污染”，在“被污染”的 canvas 使用以下方法将抛出错误：

- 在 [canvas](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas) ` 的上下文上调用` [getImageData()](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/getImageData)
- 在 [canvas](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas) ` 上调用  ` [toBlob()](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toBlob)
- 在 [canvas ](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas)` 上调用 ` [toDataURL()](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)

解决方法同第六节的方法一：

一、给 img 标签添加 `crossorigin="anonymous"`

二、服务器给跨域图像添加响应头 `Access-Control-Allow-Origin: http://another-domain.com`



### Reference:

- [https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)
- [https://html.spec.whatwg.org/multipage/embedded-content.html#attr-img-crossorigin](https://html.spec.whatwg.org/multipage/embedded-content.html#attr-img-crossorigin)
- [https://html.spec.whatwg.org/multipage/scripting.html#attr-script-crossorigin](https://html.spec.whatwg.org/multipage/scripting.html#attr-script-crossorigin)
- [https://stackoverflow.com/questions/39652618/classic-scripts-v-s-module-scripts-in-javascript](https://stackoverflow.com/questions/39652618/classic-scripts-v-s-module-scripts-in-javascript)
- https://blog.sentry.io/2016/05/17/what-is-script-error

