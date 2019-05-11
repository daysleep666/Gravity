---
layout: post
title:  "Cache-Control的理解"
date:   2018-11-23 11:00:00
categories: 基础知识
---

我先写了一段测试用的程序,用来观察在浏览器访问127.0.0.1:2234/hello/hello.html时，修改Cache-Control对StatusCode的影响。

```

package main

import (
	"github.com/labstack/echo"
)

func main() {
	e := echo.New()
	e.Use(func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c echo.Context) error {
			c.Response().Header().Set("Cache-Control", "max-age=0")
			return next(c)
		}
	})
	e.Static("/hello", "httpcachecontrol/view")
	e.File("/hello", "httpcachecontrol/view/*.html")
	err := e.Start(":2234")
	if err != nil {
		panic(err)
	}
}


```

hello.html中的内容

```

<!doctype html>
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="Cache-Control" content="no-cache" />
    
    <link rel="stylesheet" href="c.css" type="text/css" />
    <title></title>
</head>
<body>
    <p>Hello World...</p>
</body>
</html>

```

```

h1 {color:black; font-size:14px;}

```

----

将Response的Cache-Control设置为 max-age=0。

第一次访问结果:c.css，以下是在Chrome调试中看到的部分结果(我将跟Cache-Control无关的字段删掉了)

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 200 OK
      Response Header:
          Cache-Control: max-age=0
          Content-Length: 33
          Content-Type: text/css; charset=utf-8
          Date: Fri, 23 Nov 2018 11:12:09 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
```

可以看到响应的StatusCode是200。

刷新浏览器，我们看到了第二次访问结果。

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 304 Not Modified
      Response Header:
          Date: Fri, 23 Nov 2018 11:15:47 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
      Request Header:
          If-Modified-Since: Fri, 23 Nov 2018 09:32:11 GMT
```


可以看到响应对StatusCode变成了304。

通过RequestHeader的If-Modified-Since和Last-Modified: Fri, 23 Nov 2018 07:21:26 GMT是否相同来判断StatusCode是不是304.

Cache-Control: max-age=0的意思是，缓存时间0s过期，向服务器发起请求确认，该资源是否有修改。

而状态码304告诉客户端，虽然你的缓存过期了，但是文件并没有改变，你可以继续用你缓存里的东西。

第二次访问结果中的Response Header并没有Content-Length，表明没有数据从服务器传来。

------

将Response的Cache-Control设置为 max-age=20。

第一次访问结果:

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 200 OK
      Response Header:
          Cache-Control: max-age=20
          Content-Length: 33
          Content-Type: text/css; charset=utf-8
          Date: Fri, 23 Nov 2018 11:12:09 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
```

立刻刷新浏览器，我们看到了第二次访问结果。

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 200 OK(from memory cache)
      Response Header:
          Cache-Control: max-age=20
          Date: Fri, 23 Nov 2018 11:15:47 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
      Request Header:
          If-Modified-Since: Fri, 23 Nov 2018 09:32:11 GMT
```

Cache-Control: max-age=20，表明服务器在20秒内可以直接使用缓存里的资源。

二十秒后在刷新浏览器

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 304 Not Modified
      Response Header:
          Cache-Control: max-age=20
          Date: Fri, 23 Nov 2018 11:15:47 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
      Request Header:
          If-Modified-Since: Fri, 23 Nov 2018 09:32:11 GMT
```

因为缓存在20s秒后过期了，客户端重新向服务器发起请求，确认了该资源没有更改，继续使用缓存里的资源。

----

将Response的Cache-Control设置为 max-age=no-cache

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 304 Not Modified
      Response Header:
          Cache-Control: no-cache
          Date: Fri, 23 Nov 2018 11:15:47 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
      Request Header:
          If-Modified-Since: Fri, 23 Nov 2018 09:32:11 GMT
```

无论怎么刷新,结果都是这个，因为no-cache表示，强制客户端每次都需要向服务器发起请求，来确认资源是否有修改。

----

将Response的Cache-Control设置为 max-age=no-store

```
      General:
      Request URL: http://127.0.0.1:2234/hello/c.css
          Status Code: 200 OK
      Response Header:
          Cache-Control: no-store
          Content-Length: 33
          Content-Type: text/css; charset=utf-8
          Date: Fri, 23 Nov 2018 11:32:38 GMT
          Last-Modified: Fri, 23 Nov 2018 09:32:11 GMT
```

无论怎么刷新，结果都是200，no-store表示，客户端不能缓存该资源，每次都需要从服务器获取最新的资源。

----

为什么要在html中引入一个css资源来测试Cache-Control,而不是直接查看Request URL: http://127.0.0.1:2234/hello/hello.html的请求，
是因为无论怎么修改服务器的Cache—Control的值，Chrome中的RequestHeader的Cache—Control只会是max-age=0或者max-age=no-cache。对于
这个原因，我自己的理解是，如果访问的html也被允许缓存了，那如果服务器的html资源被修改了，客户端是无法知道这件事的，那如果缓存时间是很久很久
，那这个结果将会是灾难性的，或许哪天这个html没有了，客户端也不会知道这件事。所以Chrome强制必须向服务器发起请求，来看这个html是否有经过
修改。而对于引入的资源就可以安心的缓存了，如果对引入的资源进行修改，而本地进行了缓存，可以对它的引入链接进行修改，来强行使浏览器获取新的资源。