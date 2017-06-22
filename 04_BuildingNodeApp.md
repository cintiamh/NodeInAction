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
This means that if you don't listen to those errors, they'll crash your server.

### Preemptive error handling with `fs.stat`

```javascript
var server = http.createServer(function(req, res) {
  var url = parse(req.url);
  var path = join(root, url.pathname);
  fs.stat(path, function(err, stat) {
    if (err) {
      if ('ENOENT' == err.code) {
        res.statusCode = 404;
        res.end('Not found');
      } else {
        res.statusCode = 500;
        res.end('Internal Server Error');
      }
    } else {
      res.setHeader('Content-Length', stat.size);
      var stream = fs.createReadStream(path);
      stream.pipe(res);
      stream.on('error', function(err) {
        res.statusCode = 500;
        res.end('Internal Server Error');
      });
    }
  })
})
```

## Accepting user input from forms

### Handling submitted form fields

`Content-Type`:
* `application/x-www-form-urlencoded` - The default for HTML forms
* `multipart/form-data` - Used when the form contain files

To handle file uploads properly and accept the file's content, you need to set
the `enc-type` attribute to `multipart/form-data`, a MIME type suited for BLOBs
(binary large objects).

* use formidable.

Formidable's `progress` event emits the number of bytes received and bytes expected.

## Securing your application with https

Data sent using HTTPS is encrypted. (HTTP + TLS/SSL)

Self-signed certificate: just for development (using OpenSSL)

Generate a private key:
```
$ openssl genrsa 1024 > key.pem
```

Generate a certificate using the private key:
```
openssl req -x509 -new -key key.pem > key-cert.pem
```

Keys are usually kept in `~/.ssh`.
The following code will create a simple HTTPS server using your keys.

```
var https = require('https');
var fs = require('fs');

var options = {
  key: fs.readFileSync('./key.pem'),
  cert: fs.readFileSync('./key-cert.pem')
};

https.createServer(options, function(req, res) {
  res.writeHead(200);
  res.end('Hello World');
}).listen(3000);
```
