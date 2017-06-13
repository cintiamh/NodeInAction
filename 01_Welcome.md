# Welcome to Node.js

## Build on JavaScript

"Write once, run anywhere"

* Event-driven and asynchronous - Browser and server side with Node.
* DIRT (data-intensive real-time) applications

Node implements some browser common objects:
* setTimeout
* console.log

Also the core includes some file I/O
* HTTP, TLS, HTTPS
* filesystem (POSIX)
* Datagram (UDP)
* NET (TCP)

Node is a PLATFORM for JavaScript applications.

* Common use case for Node: Building servers.

```javascript
var http = require('http');
http.createServer(function(req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(3000);
console.log('Server running at http://localhost:3000/');
```

### Streaming data

* Stream: data distributed over time. (chunk by chunk)

```javascript
var stream = fs.createReadStream('./resource.json');
stream.on('data', function(chunk) {
  console.log(chunk);
})
stream.on('end', function() {
  console.log('finished');
})
```

A chunk can vary on size depending on the type of data.

```javascript
var http =  require('http');
var fs = require('fs');
http.createServer(function(req, res) {
  res.writeHead(200, { 'Content-Type': 'image/png' });
  fs.createReadStream('./image.png').pipe(res);
}).listen(3000);
console.log('Server running at http://localhost:3000/')
```

The data is read in from the file (fs.createReadStream) and is sent out (.pipe) to the client (res) as it comes in.
