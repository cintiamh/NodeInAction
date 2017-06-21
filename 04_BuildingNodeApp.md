# Building Node Web Applications

* fs - filesystem module.

## HTTP server fundamentals

Node has a relatively low-level API.

### How Node presents incoming HTTP requests to developers

```javascript
var http = require('http');

// create a HTTP server:
http.createServer(function(req, res) {
  // handle request
});
```

You need to end the response using `res.end()`. If you fail to end the response, the request will hang until the client times out or it will just remain open.

### A basic HTTP server that responds with "Hello World!"

```javascript
var http = require('http');
var server = http.createServer(function(req, res) {
  // Writes response data to the socket
  res.write('Hello World');
  // End response.
  res.end();
});
// bind to a port so you can listen for incoming requests.
server.listen(3000);
```

You can combine `res.write()` and `res.end()` for small responses.
```javascript
res.end('Hello World');
```

### Reading request headers and setting response headers

```javascript
var body = 'Hello World';
res.setHeader('Content-Length', body.length);
res.setHeader('Content-Type', 'text/plain');
res.end(body);
```

After the first part of the response body is written, Node will flush the HTTP headers that have been set.

### Setting the status code of an HTTP response

You can set `res.statusCode` property anytime before `res.write()` or `res.end()`;

```javascript
var url = 'http://www.google.com';
var body = '<p>Redirecting to <a href="' + url + '">' + url + '</a></p>';
res.setHeader('Location', url);
res.setHeader('Content-Length', body.length);
res.setHeader('Content-Type', 'text/html');
res.statusCode = 302;
res.end(body);
```

## Building a RESTful Web Service

REST - representational state transfer - GET, POST, PUT, and DELETE.

```javascript
var http = require('http');
var url = require('url');
var items = [];

var server = http.createServer(function(req, res) {
  switch(req.method) {
      case 'POST':
        var item = "";
        req.setEncoding('utf8');
        req.on('data', function(chunk) {
          item += chunk;
        });
        req.on('end', function() {
          items.push(item);
          res.end('OK\n');
        });
        break;
  }
});
```

## Serving Static files

```javascript
var http = require('http');
var parse = require('url').parse;
var join = require('path').join;
var fs = require('fs');

var root = __dirname;

var server = http.createServer(function(req, res) {
  var url = parse(req.url);
  var path = join(root, url.pathname);
  var stream = fs.createReadStream(path);
  stream.on('data', function(chunk) {
    res.write(chunk);
  });
  stream.on('end', function() {
    res.end();
  });
});

server.listen(3000);
```

`fs.createReadStream()` works, but Node also provides `Stream#pipe()`.

https://github.com/substack/stream-handbook

```javascript
var server = http.createServer(function(req, res) {
  var url = parse(req.url);
  var path = join(root, url.pathname);
  var stream = fs.createReadStream(path);
  // res.end() called internally by stream.pipe().
  stream.pipe(res);
}
```

### Handling server errors

By default, `error` events will be thrown when no listeners are present.
