# Node programming fundamentals

## Organizing and reusing node functionality

Node modules bundle up code for reuse, but they don't alter global scope.

Node modules allow you to select what functions are variables from the included file are exposed to the application.

### Creating modules

It can be a file or a folder, default: main file is called `index.js`.

* if a module is a directory, the file in the module directory that will be evaluated must be named `index.js`, unless specified in `package.json`.

```javascript
{
  "main": "./currency.js"
}
```

## Asynchronous programming techniques

Response:
* callbacks: one-off responses
* event listeners: callbacks associated with events.

```javascript
var EventEmiter = require('event').EventEmiter;
var channel = new EventEmiter();
channel.on('join', function() {
  console.log('Welcome!');
});

channel.emit('join');
```

The concept of sequencing groups of asynchronous tasks is called flow control by the Node community.

There are two types of flow control:
* serial
* parallel

### Serial

```
Start => Task 1 => Task 2 => Task 3 => Continue when task 3 is complete.
```

### Parallel

```
  |--------- Start ---------|
  |            |            |
  v            v            v
Task         Task         Task
  |            |            |
  |            v            |
  |--> Continue when all <--|
         tasks complete
```

Node's module system is based on CommonJS module specification (http://www.commonjs.org/specs/modules/1.0/).
