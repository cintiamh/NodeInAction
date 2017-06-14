# Building a multiroom chat application

* MIME types: https://en.wikipedia.org/wiki/MIME
* Websocket: https://en.wikipedia.org/wiki/WebSocket
  * bidirectional lightweight communication protocol to support real-time communication.
  * Socket.IO library (to support older browsers).

Node can handle simultaneously serving HTTP and WebSocket using a single TCP/IP port.

Application folder structure:

* lib - server-side logic
* public - client-side files
  * JavaScripts
  * stylesheets

Dependencies:
package.json - describes the functionality and dependencies of your application.

```javascript
{
  "name": "chatapp",
  "version": "0.0.1",
  "description": "A simple chat room",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Cintia Higashi",
  "license": "ISC",
  "dependencies": {
    "mime": "^1.3.6",
    "socket.io": "^2.0.3"
  }
}
```

Create the package.json file:
```
$ npm init
```

Install socket.io and mime packages:
```
$ npm i socket.io mime --save
```

After installing the node packages, you should have a node_modules folder created for you with the dependencies.

Socket.IO provides virtual channels out of the box.

Event emmiters are a handy design pattern for organizing asynchronous logic.
