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
