# Storing Node Application Data

Types:

* In-memory and filesystem data storage
* Conventional relational database storage
* Nonrelational database storage

Factors:

* What is being stored
* How quickly data needs to be read and written to maintain adequate performance
* How much data exists
* How data needs to be queried
* How long and reliably the data needs to be stored

## Serverless data storage

### In-memory storage

In-memory storage uses variables to store data.
Reading and writing the data is fast, but it will be lost once the application re-starts.

### File-based storage

Often used to persist application configuration information.

Concurrency Issue: For multiuser applications database management systems are better.

## Relational database management systems

* RDBMSs allow complex information to be stored and easily queried.
* RDBMSs are used for high-end applications, such as content management, customer relationship management, and shopping carts.
* You need to know SQL.

### MySQL

To allow Node to talk to MySQL, use `node-mysql` module.

https://dev.mysql.com/doc/refman/5.7/en/tutorial.html

```
$ npm install mysql
```

timetrack_server.js
```javascript
var http = require('http');
var work = require('./lib/timetrack');
var mysql = require('mysql');

var db = mysql.createConnection({
  host: '127.0.0.1',
  user: 'myuser',
  password: 'mypassword',
  database: 'timetrack'
});

var server = http.createServer(function(req, res) {
  switch(req.method) {
    case 'POST':
      switch(req.url) {
        case '/':
          work.add(db, req, res);
          break;
        case '/archive':
          work.archive(db, req, res);
          break;
        case '/delete':
          work.delete(db, req, res);
          break;
      }
      break;
    case 'GET':
      switch(req.url) {
        case '/':
          work.show(db, res);
          break;
        case '/archived':
          work.showArchived(db, res);
      }
      break;
  }
});

db.query(
  "CREATE TABLE IF NOT EXISTS work ("
  + "id INT(10) NOT NULL AUTO_INCREMENT, "
  + "hours DECIMAL(5, 2) DEFAULT 0, "
  + "date DATE, "
  + "archived INT(1) DEFAULT 0, "
  + "description LONGTEXT, "
  + "PRIMARY KEY(id))",
  function(err) {
    if (err) throw err;
    console.log('Server started...');
    server.listen(3000, '127.0.0.1');
  }
);
```

lib/timetrack.js
```javascript
var qs = require('querystring');

exports.sendHtml = function(res, html) {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('Content-Length', Buffer.byteLength(html));
  res.end(html);
};

exports.parseReceivedData = function(req, cb) {
  var body = '';
  req.setEncoding('utf8');
  req.on('data', function(chunk) { body += chunk; });
  req.on('end', function() {
    var data = qs.parse(body);
    cb(data);
  });
};

exports.actionForm = function(id, path, label) {
  var html = '<form method="POST" action="' + path + '">' +
    '<input type="hidden" name="id" value="' + id + '" />' +
    '<input type="submit" value="' + label + '" />' +
    '</form>';
  return html;
}

exports.add = function(db, req, res) {
  exports.parseReceivedData(req, function(work) {
    db.query(
      "INSERT INTO work (hours, date, description) " +
      "VALUES (?, ?, ?)",
      [work.hours, work.date, work.description],
      function(err) {
        if (err) throw err;
        exports.show(db, res);
      }
    )
  })
}

exports.delete = function(db, req, res) {
  exports.parseReceivedData(req, function(work) {
    db.query(
      "DELETE FROM work WHERE id=?",
      [work.id],
      function(err) {
        if (err) throw err;
        exports.show(db, res);
      }
    )
  })
}

exports.archive = function(db, req, res) {
  exports.parseReceivedData(req, function(work) {
    db.query(
      "UPDATE work SET archived=1 WHERE id=?",
      [work.id],
      function(err) {
        if (err) throw err;
        exports.show(db, res);
      }
    )
  })
}

exports.show = function(db, res, showArchived) {
  var query = "SELECT * FROM work WHERE archived=? ORDER BY date DESC";
  var archiveValue = (showArchived) ? 1 : 0;
  db.query(
    query,
    [archiveValue],
    function(err, rows) {
      if (err) throw err;
      html = (showArchived) ?
        '' :
        '<a href="/archived">Archived Work</a><br/>';
      html += exports.workHitlistHtml(rows);
      html += exports.workFormHtml();
      exports.sendHtml(res, html);
    }
  )
}

exports.showArchived = function(db, res) {
  exports.show(db, res, true);
}

exports.workHitlistHtml = function(rows) {
  var html = '<table>';
  for (var i in rows) {
    html += '<tr>';
    html += '<td>' + rows[i].date + '<td>';
    html += '<td>' + rows[i].hours + '<td>';
    html += '<td>' + rows[i].description + '<td>';
    if (!rows[i].archived) {
      html += '<td>' + exports.workArchiveForm(rows[i].id) + '</td>';
    }
    html += '<td>' + exports.workDeleteForm(rows[i].id) + '</td>';
    html += '</tr>';
  }
  html += '</table>';
  return html;
}

exports.workFormHtml = function() {
  var html = '<form method="POST" action="/">' +
    '<p>Date (YYYY-MM-DD):<br/><input name="date" type="text"></p>' +
    '<p>Hours worked:<br/><input name="hours" type="text"></p>' +
    '<p>Description:<br/><textarea name="description"></textarea></p>' +
    '<input type="submit" value="Add">' +
    '</form>';
  return html;
}

exports.workArchiveForm = function(id) {
  return exports.actionForm(id, '/archive', 'Archive');
}

exports.workDeleteForm = function(id) {
  return exports.actionForm(id, '/delete', 'Delete');
}
```

Try it out:
```
$ node timetrack_server.js
```

Navigate to `http://127.0.0.1:3000`

### PostgreSQL

https://www.postgresql.org/docs/7.4/static/tutorial.html
https://github.com/brianc/node-Postgres

* supports recursive queries and many specialized data types.
* can use a variety of standard authentication methods.
* supports synchronous replication.

```
$ npm install pg
```

Connecting to PostgreSQL:
```javascript
var pg = require('pg');
var conString = "tcp://myuser:mypassword@localhost:5432/mydatabase";
var client = new pg.Client(conString);
client.connect();
```

Insert a row:
```javascript
client.query(
  'INSERT INTO users ' +
  "(name) VALUES ('Mike')"
);
```

Using placeholders:
```javascript
client.query(
  "INSERT INTO users " +
  "(name, age) VALUES ($1, $2)",
  ['Mike', 39]
);
```

Get the primary key value of a row after an insert:
```javascript
client.query(
  "INSERT INTO users " +
  "(name, age) VALUES ($1, $2) " +
  "RETURNING id",
  ['Mike', 39],
  function(err, result) {
    if (err) throw err;
    console.log('Insert ID is ' + result.rows[0].id);
  }
);
```

Selecting rows from a PostgreSQL database:
```javascript
var query = client.query(
  "SELECT * FROM users WHERE age > $1",
  [40]
);

query.on('row', function(row) {
  console.log(row.name);
});

// The end event is emitted after the last row is fetched.
query.on('end', function() {
  client.end();
})
```

## NOSQL databases
