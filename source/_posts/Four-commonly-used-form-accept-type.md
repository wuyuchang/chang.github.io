---
title: Four commonly used form accept type
date: 2016-12-20 11:06:46
tags: Node.js
---

The blog based on Node.js server.

In front-end, we have several types to send a request to server, most of the time, we use 3 commonly below:
1. application/json
2. application/x-www-form-urlencoded
3. multipart/form-data

## application/json
### GET
#### client
``` javascript
$.ajax({
  url: '/upload',
  method: 'GET',
  contentType: 'application/json',
  data: JSON.stringify({   // must turn to string, whether jQuery will turn to "x-www-form-urlcoded" format
    username: 'username',
    password: 'password'
  })
})
```
Check the request header in network panel in Chrome dev-tool, you will see the request URL turn to *http://domain/upload?{"test":"wuyuchang","passowrd":"chang"}*, it's because GET request always append data to the request URL.

``` header
Request URL:http://domain/upload?{"test":"username","passowrd":"password"}
Request Method:GET
Content-Type:application/json
...
```
#### server
In server side, you can read the header that client send throw req.headers, and URL throw req.url, then you just compile analysis the parameter after question mark.
``` javascript
http.createServer((req, res) => {
  const qs = require('querystring')
  let param = qs.unescape(req.url.split('?')[1])
  let json = JSON.parse(param)
  // ...
}).listen(8080)
```
### POST
#### Client
For client, you just change the request method from 'GET' to 'POST'.
#### Server
In server side, you can't get content that client send directly, it's send throw buffer, so you have to receive it by listen it throw req.on('data')
``` javascript
http.createServer((req, res) => {
  let chunk = ''
  req.on('data', data => {
    chunk += data
  })
  req.on('end', err => {
    let json = JSON.parse(chunk.toString())
    // ...
  })
}).listen(8080)
```
It's convenient to get JSON data throw this way.

## application/x-www-form-urlencoded
### GET
#### Client
Same with above, just change the contenType to 'application/x-www-form-urlencoded', the jQuery wouldn't add contentType automatic if you set method as 'GET'.
#### server
``` javascript
http.createServer((req, res) => {
  let json = require('url').parse(req.url, true).query
  // ...
}).listen(8080)
```
### POST
jQuery set contentType as 'application/x-www-form-urlencoded' if you send request throw 'POST' method.
#### server
``` javascript
http.createServer((req, res) => {
  let chunk = ''
  req.on('data', data => {
    chunk += data
  })
  req.on('end', err => {
    let qs = require('querystring')
    let json = qs.parse(chunk.toString())
    // ...
  })
}).listen(8080)
```

## multipart/form-data
*multipart/form-data* always used to send a file, of course, you also can send form without file.
### GET
It's same with below, the different is that if you send a file to server, the server can't accept it.

### POST
#### Client
``` html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="text" name="test" value="wuyuchang">
  <input type="password" name="pwd" value="chang">
  <input type="file" multiple="multiple" name="upload"><br>
  <input type="submit" value="Upload">
</form>
```
the request header is different
``` text
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryuyeyXEbnVwhZJcl6
Request Payload:
  ------WebKitFormBoundaryuyeyXEbnVwhZJcl6
  Content-Disposition: form-data; name="test"

  wuyuchang
  ------WebKitFormBoundaryuyeyXEbnVwhZJcl6
  Content-Disposition: form-data; name="pwd"

  chang
  ------WebKitFormBoundaryuyeyXEbnVwhZJcl6
  Content-Disposition: form-data; name="upload"; filename="boom.png"
  Content-Type: image/png
```

the value of boundary is random.
#### Server
In server side you can compile it, but it's complex, you can import a package name of 'formidable'.
