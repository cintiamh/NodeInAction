# Express

http://expressjs.com/

## Generating the application skeleton

Express doesn't force application structure on the developer.

The `express` executable script bundled with Express can set up an application skeleton for you.

### Installing the Express executable

Install Express globally:
```
$ npm install express-generator -g
$ express -h
```

### Generating the application

```
$ express --view=ejs photoApp
$ cd photoApp
$ npm install
$ npm install ejs --save
```

### Configuring Express and your application

Express has a minimalistic environment-driven configuration system, consisting of five methods, all driven by the NODE_ENV environment variable.

* `app.configure()`
* `app.set()`
* `app.get()`
* `app.enable()`
* `app.disable()`

#### Setting environment variables

```
$ NODE_ENV=production node app
```

These environment variables will be available in your application on the `process.env` object.

### Environment-based configuration
