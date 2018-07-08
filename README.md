<p align="center">
  <a href="http://mariadb.org/">
    <img src="https://mariadb.com/themes/custom/mariadb/logo.svg">
  </a>
</p>

# MariaDB Node.js connector

[![Linux Build](https://travis-ci.org/rusher/mariadb-connector-nodejs.svg?branch=master)](https://travis-ci.org/rusher/mariadb-connector-nodejs)
[![Windows status](https://ci.appveyor.com/api/projects/status/nuvvbkx82ixfhp12?svg=true)](https://ci.appveyor.com/project/rusher/mariadb-connector-nodejs)
[![License (LGPL version 2.1)](https://img.shields.io/badge/license-GNU%20LGPL%20version%202.1-green.svg?style=flat-square)](http://opensource.org/licenses/LGPL-2.1)
[![Coverage Status](https://coveralls.io/repos/github/rusher/mariadb-connector-nodejs/badge.svg)](https://coveralls.io/github/rusher/mariadb-connector-nodejs)

**Non-blocking MariaDB and MySQL client for Node.js.**

MariaDB and MySQL client, 100% JavaScript, compatible with Node.js 6+, with the Promise API.

## Why a New Client?

While there are existing MySQL clients that work with MariaDB, (such as the [mysql](https://www.npmjs.com/package/mysql) and [mysql2](https://www.npmjs.com/package/mysql2) clients), the MariaDB Node.js Connector offers new functionality, like [Insert Streaming](#insert-streaming) and [Pipelining](#pipelining) while making no compromises on performance.

### Insert Streaming 

Using a Readable stream in your application, you can stream `INSERT` statements to MariaDB through the Connector.

```javascript
    
    https.get('https://someContent', readableStream => {
        //readableStream implement Readable, driver will stream data to database 
        connection.query("INSERT INTO myTable VALUE (?)", [readableStream]);
    });
```
 
### Pipelining

With Piplining, the Connector sends commands without waiting for server results, preserving order.  For instance, consider the use of executing two `INSERT`  statements.

```
          │ ――――――――――――――――――――― send first insert ―――――――――――――> │ ┯ 
          │ ――――――――――――――――――――― send second insert ――――――――――――> │ │  processing first insert
          │                                                        │ │ 
Client    │ <―――――――――――――――――――― first insert result ―――――――――――― │ ▼  ┯
          │                                                        │    │ processing second insert
          │                                                        │    │
          │ <―――――――――――――――――――― second insert result ――――――――――― │    ▼ 
```

The Connector doesn't wait for query results before sending the next `INSERT` statement. Instead, it sends queries one after the other, avoidng much of the network latency.

For more information, see the [Pipelining](/documentation/piplining.md) documentation.


## Benchmarks

MariaDB provides benchmarks comparing the Connector with popular Node.js MySQL clients, including: 

* [`promise-mysql`](https://www.npmjs.com/package/promise-mysql) version 3.3.1 + [mysql](https://www.npmjs.com/package/mysql) version 2.15.0 
* [`mysql2`](https://www.npmjs.com/package/mysql2) version 1.5.3

```
promise-mysql  : 1,366 ops/sec ±1.42%
mysql2         : 1,469 ops/sec ±1.63%
mariadb        : 1,802 ops/sec ±1.19%
```

<img src="./documentation/misc/bench.png" width="559" height="209"/>

For more information, see the [Benchmarks](/documentation/benchmarks.md) page.

## Road Map 

The Connector remains in development.  Here's a list of features being developed for future releases:

* Pooling and `PoolCluster`
* MariaDB `ed25519` plugin authentication
* Query Timeouts
* Bulk Insertion, (that is, fast batch).


## Contributing 

If you would like to contribute to the MariaDB Node.js Connector, please follow the instructions given in the [Developers Guide.](/documentation/developers-guide.md)

To file an issue or follow the development, see [JIRA](https://jira.mariadb.org/projects/CONJS/issues/).


## Quick Start

The MariaDB Connector is available through the Node.js repositories.  You can install it using npm.

```
$ npm install mariadb
```

Using the ECMAScript, prior to 2017:

```js
const mariadb = require('mariadb');
mariadb.createConnection({host: 'mydb.com', user:'myUser'})
.then(conn => {

  conn.query("SELECT 1 as val")
	.then((rows) => {
	  console.log(rows); //[ {val: 1}, meta: ... ]
	  return conn.query("INSERT INTO myTable value (?, ?)", [1, "mariadb"]);
	})
	.then((res) => {
	  console.log(res); // { affectedRows: 1, insertId: 1, warningStatus: 0 }
	  conn.end();
	})
	.catch(err => {
	  //handle error
	  conn.end();
	})
	
}).catch(err => {
  //not connected
});
```

Using ECMAScript 2017:

```js
const mariadb = require('mariadb');
async function asyncFunction() {
  let conn;
  try {
	conn = await mariadb.createConnection({host: 'localhost', user: 'root'});

	const rows = await conn.query("SELECT 1 as val");
	console.log(rows); //[ {val: 1}, meta: ... ]
	const res = await conn.query("INSERT INTO myTable value (?, ?)", [1, "mariadb"]);
	console.log(res); // { affectedRows: 1, insertId: 1, warningStatus: 0 }

  } catch (err) {
	return Promise.reject(err);
  } finally {
	if (conn) return conn.end();
  }
  return Promise.resolve();
}
```

## Documentation

The MariaDB Connector can use different API's on the back-end: Promise and Callback.  The default API is Promise.  Callback is provided for compatibility with the `mysql` and `mysql2` API's.

Documentation provided on this page uses the Promise API.  If you would like to develop an application with the Callback API or have an existing application that you want to switch from the MySQL API's to the MariaDB Connector, see the [Callback API](/documentation/callback-api.md) documentation.

### Installation

As described in the quick start section above, you can install the MariaDB Connector using npm.

```
$ npm install mariadb
```


### API

**Creating a Connection:**

* [`createConnection(options) → Promise`](#createconnectionoptions--promise) : Creates a new connection.

**Connection API:** 

* [`query(sql[, values]) → Promise`](#querysql-values---promise): Executes a query.
* [`queryStream(sql[, values]) → Emitter`](#querystreamsql-values--emitter): Executes a query, returning an emitter object to stream rows.
* [`beginTransaction() → Promise`](#begintransaction--promise): Begins a transaction.
* [`commit() → Promise`](#commit--promise): Commits the current transaction, if any.
* [`rollback() → Promise`](#rollback--promise): Rolls back the current transaction, if any.
* [`changeUser(options) → Promise`](#changeuseroptions--promise): Changes the current connection user
* [`ping() → Promise`](#ping--promise): Sends a 1 byte packet to database to validate the connection.
* [`isValid() → boolean`](#isvalid--boolean): Checks that the connection is active without checking socket state.
* [`end() → Promise`](#end--promise): Gracefully closes the connection.
* [`destroy()`](#destroy): Forces the connection to close. 
* [`pause()`](#pause): Pauses the socket output.
* [`resume()`](#resume): Resumes the socket output.
* [`serverVersion()`](#serverversion): Retrieves the current server version.
* [`events`](#events): Subscribes to connection error events.

#### `createConnection(options) → Promise`

> * `options`: *JSON* [connection option documentation](#connection-options)
>
> Returns a promise that :
> * resolves with a [Connection](#connection-object) object,
> * raises an [Error](#error).

Creates a new connection.

**Example:**

```javascript
const mariadb = require('mariadb');
mariadb.createConnection({
      host: 'mydb.com', 
      user:'myUser'
    })
    .then(conn => {
      console.log("connected ! connection id is " + conn.threadId);
    })
    .catch(err => {
      console.log("not connected due to error: " + err);
    });
```

##### Connection options

Essential options list:

|option|description|type|default| 
|---:|---|:---:|:---:| 
| **`user`** | User to access database. |*string* | 
| **`password`** | User password. |*string* | 
| **`host`** | IP address or DNS of the database server. *Not used when using option `socketPath`*. |*string*| "localhost"|
| **`port`** | Database server port number. *Not used when using option `socketPath`*|*integer*| 3306|
| **`ssl`** | Enables TLS support. For more information, see the [`ssl` option](/documentation/connection-options.md#ssl) documentation. |*mixed*|
| **`database`** | Default database to use when establishing the connection. | *string* | 
| **`socketPath`** | Permits connections to the database through the Unix domain socket or named pipe. |  *string* | 
| **`compress`** | Compresses the exchange with the database through gzip.  This permits better performance when the database is not in the same location.  |*boolean*| false|
| **`connectTimeout`** | Sets the connection timeout in milliseconds. |*integer* | 10 000|
| **`socketTimeout`** | Sets the socket timeout in milliseconds after connection succeeds. A value of `0` disables the timeout. |*integer* | 0|
| **`rowsAsArray`** | Returns result-sets as arrays, rather than JSON. This is a faster way to get results. For more information, see Query. |*boolean* | false|

For more infomration, see the [Connection Options](/documentation/connection-options.md) documentation. 

#### Connecting to Local Databases 

When working with a local database, (that is, cases where MariaDB and your Node.js application run on the same host), you can connect to MariaDB through the Unix socket or Windows named pipe for better performance, rather than using the TCP/IP layer.

In order to set this up, you need to assign the connection a `socketPath` value.  When this is done, the Connector ignores the `host` and `port` options.

The specific socket path you need to set is defined by the 
[`socket`](https://mariadb.com/kb/en/library/server-system-variables/#socket) server system variable.  If you don't know it off hand, you can retrieve it from the server.

```sql
SHOW VARIABLES LIKE 'socket';
```

It defaults to `/tmp/mysql.sock` on Unix-like operating systems and `MySQL` on Windows.  Additionally, on Windows this feature only works when the server is started with the `--enable-named-pipe` option.

For instance, on Unix a connection might look like this:

```javascript
const mariadb = require('mariadb');
mariadb.createConnection({socketPath: '/tmp/mysql.sock'})
    .then(conn => { ... })
    .catch(err => { ... });
```

It has a similar syntax on Windows: 

```javascript
const mariadb = require('mariadb');
mariadb.createConnection({socketPath: '\\\\.\\pipe\\MySQL'})
    .then(conn => { ... })
    .catch(err => { ... });
```
 
#### `query(sql[, values])` -> `Promise`

> * `sql`: *string | JSON* SQL string or JSON object to supersede default connection options.  When using JSON object, object must have a "sql" key. For instance,
>
>   `{ dateStrings: true, sql: 'SELECT now()' }`
> * `values`: *array | object* Placeholder values. Usually an array, but in cases of only one placeholder, it can be given as is. 
>
> Returns a promise that :
> * resolves with a JSON object for update/insert/delete or a [result-set](#result-set-array) object for result-set.
> * rejects with an [Error](#error).


Sends a query to database and return result as a Promise.

For instnace, when using an SQL string:

```js
connection
  .query("SELECT NOW()")
  .then(rows => {
	console.log(rows); //[ { 'NOW()': 2018-07-02T17:06:38.000Z }, meta: [ ... ] ]
  })
  .catch(err => {
	//handle error
  });
```

Alternatively, you could use the JSON object:

```js
connection
   .query({dateStrings:true, sql:'SELECT NOW()'})
   .then(rows => {
	  console.log(rows); //[ { 'NOW()': '2018-07-02 19:06:38' }, meta: [ ... ] ]
	})
	.catch(...)
```

##### Placeholder

To prevent SQL Injection attacks, queries permit the use of question marks as placeholders.  The Connection escapes values according to their type.  Values can be of native JavaScript types, Buffers, Readables, objects with `toSQLString` methods, or objects that can be stringified, (that is, `JSON.stringfy`)

When streaming, objects that implement Readable are streamed automatically.  But, there are two server system variables that may interfere:

- [`net_read_timeout`](https://mariadb.com/kb/en/library/server-system-variables/#net_write_timeout): The server must receive queries before reaching this timeout, which defaults to 30 seconds.
- [`max_allowed_packet`](https://mariadb.com/kb/en/library/server-system-variables/#max_allowed_packet): This system variable defines the maximum amount of data the Connector can send to the server.

For instance,

```js
connection
  .query(
	 "INSERT INTO someTable VALUES (?, ?, ?)", 
	 [1,Buffer.from("c327a97374", "hex"),"mariadb"]
  )
  .then(...)
  .catch(...);
  //will send INSERT INTO someTable VALUES (1, _BINARY '.\'.st', 'mariadb')
```


In the case of streaming, 

```js
const https = require("https");
//3Mb page
https.get("https://node.green/#ES2018-features-Promise-prototype-finally-basic-support",
  readableStream => {
    connection.query("INSERT INTO StreamingContent (b) VALUE (?)", [readableStream]);
      .then(res => {
        //inserted
      })
      .catch(console.log);
  }
)
```

##### JSON Result-sets 

Queries return two different kinds of results, depending on the type of query you execute.  When you execute write statements, (such as `INSERT`, `DELETE` and `UPDATE`), the method returns a JSON object with the following properties:

* `affectedRows`: An integer listing the number of affected rows.
* `insertId`: An integer noting the auto-increment ID of the last row written to the table.
* `warningStatus`: An integer indicating whether the query ended with a warning.

```js
connection.query('CREATE TABLE animals (' +
                       'id MEDIUMINT NOT NULL AUTO_INCREMENT,' +
                       'name VARCHAR(30) NOT NULL,' +
                       'PRIMARY KEY (id))');
connection.query('INSERT INTO animals(name) value (?)', ['sea lions'])
    .then(res => {
      console.log(res); 
      //log : { affectedRows: 1, insertId: 1, warningStatus: 0 }
    })
    .catch(...);
```

##### Array Result-sets 

When the query executes a `SELECT` statement, the method returns the result-set as an array.  Each value in the array is a returned row as a JSON object.  Additionally, the method returns a special `meta` array that contains the [column metadata](#column-metadata) information. 

The rows default to JSON objects, but two other formats are also possible with the `nestTables` and `rowsAsArray` options.

```javascript
connection.query('select * from animals')
    .then(res => {
      console.log(res); 
      // [ 
      //    { id: 1, name: 'sea lions' }, 
      //    { id: 2, name: 'bird' }, 
      //    meta: [ ... ]
      // ]
    });
```

##### Query options

* [`namedPlaceholders`](#namedPlaceholders)
* [`typeCast`](#typeCast)
* [`rowsAsArray`](#rowsAsArray)
* [`nestTables`](#nestTables)
* [`dateStrings`](#dateStrings)
* [`supportBigNumbers`](#supportBigNumbers)
* [`bigNumberStrings`](#bigNumberStrings)

Those options can be set on query level, but are usually set at connection level, then will apply to all queries. 

###### `namedPlaceholders`

*boolean, default false*

While the recommended method is to use the question mark [placeholder](#placeholder), you can alternatively allow named placeholders by setting this query option.  Values given in the query must contain keys corresponding  to the placeholder names. 

```javascript
connection
  .query(
	{ namedPlaceholders: true, sql: "INSERT INTO someTable VALUES (:id, :img, :db)" },
	{ id: 1, img: Buffer.from("c327a97374", "hex"), db: "mariadb" }
  )
  .then(...)
  .catch(...);
```

#### rowsAsArray

*boolean, default false*

Using option rowsAsArray permit to have row format as an array (not as JSON).
This permit to save memory and avoid connector to have to parse [column metadata](#column-metadata) completely. It is fastest row format (by 5-10% with local database).


Default format : { id: 1, name: 'sea lions' }
with option "rowsAsArray" : [ 1, 'sea lions' ]

Example :
```javascript
connection.query({ rowsAsArray: true, sql: 'select * from animals' })
    .then(res => {
      console.log(res); 
      // [ 
      //    [ 1, 'sea lions' ], 
      //    [ 2, 'bird' ],
      //    meta: [...]
      // ]
    });
```

#### nestTables

*boolean / string, default false*

Some queries could return the columns with the **same** name. The standard JSON format won't permit having 2 keys with same name.
 
Setting the option nestTables to true, data will be group by table.
When using string parameter, json field name will be prefixed by table name + nestTable value. 

Example with boolean value:
```javascript
connection.query({nestTables:true, 
                sql:'select a.name, a.id, b.name from animals a, animals b where b.id=1'})
    .then(res => {
      console.log(res); 
      //[ 
      //  { 
      //     a: { name: 'sea lions', id: 1 }, 
      //     b: { name: 'sea lions' } 
      //  },
      //  { 
      //     a: { name: 'bird', id: 2 }, 
      //     b: { name: 'sea lions' } 
      //  },
      //  meta: [...]
      //]
    });
```

Example with string value:
```javascript
connection.query({nestTables: '_', 
                sql:'select a.name, a.id, b.name from animals a, animals b where b.id=1'})
    .then(res => {
      console.log(res); 
      //[ 
      //  { a_name: 'sea lions', a_id: 1, b_name: 'sea lions' }, 
      //  { a_name: 'bird', a_id: 2, b_name: 'sea lions' },
      //  meta: [...]
      //]
    });
```

#### dateStrings

*boolean, default: false*

Indicate if date must be retrieved as a string (not as a Date object)  

#### supportBigNumbers

*boolean, default: false*

If integer is not in ["safe"](documentation/connection-options.md#support-for-big-integer) range, the value will be return as a [Long](https://www.npmjs.com/package/long) object.


#### bigNumberStrings

*boolean, default: false*

if integer is not in ["safe"](documentation/connection-options.md#support-for-big-integer) range, the value will be return as a string

#### typeCast

*Experimental*

*function(column, next)*

If can cast type yourself if needed using option `typeCast`.

Example for casting all TINYINT(1) to boolean :
```javascript
const tinyToBoolean = (column, next) => {
  if (column.type == "TINY" && column.length === 1) {
    const val = column.int();
    return val === null ? null : val === 1;
  }
  return next();
};
connection.query({typeCast: tinyToBoolean, sql:"..."});
```

## Column metadata

* `collation`: object indicating column collation. Properties : index, name, encoding, maxlen. Example : 33, "UTF8_GENERAL_CI", "utf8", 3 
* `columnLength`: column maximum length if there is a limit, 0 if not (for blob).
* `type`: column type int value. see [values](/lib/const/field-type.js)
* `columnType`: column type string value. see [values](/lib/const/field-type.js). example "DECIMAL"
* `scale`: decimal part length.  
* `flags`: byte encoded flags. see [values](/lib/const/field-detail.js) 
* `db()`: database schema name (alias `schema()` exists)
* `table()`: table alias
* `orgTable()`: real table name
* `name()`: column alias
* `orgName()`: real column name

```js
    connection
      .query("SELECT 1, 'a'")
      .then(rows => {
        console.log(rows);
        // [ 
        //   { '1': 1, a: 'a' },
        //   meta: [ 
        //     { 
        //       collation: [Object],
        //       columnLength: 1,
        //       columnType: 8,
        //       scale: 0,
        //       type: 'LONGLONG',
        //       flags: 129,
        //       db: [Function],
        //       schema: [Function],
        //       table: [Function],
        //       orgTable: [Function],
        //       name: [Function],
        //       orgName: [Function] 
        //     },
        //     { 
        //       collation: [Object],
        //       columnLength: 4,
        //       columnType: 253,
        //       scale: 39,
        //       type: 'VAR_STRING',
        //       flags: 1,
        //       db: [Function],
        //       schema: [Function],
        //       table: [Function],
        //       orgTable: [Function],
        //       name: [Function],
        //       orgName: [Function] 
        //     } 
        //   ] 
        // ]
        assert.equal(rows.length, 1);
      })
```


## queryStream(sql[, values]) → Emitter

> * `sql`: *string | JSON* sql string value or JSON object to supersede default connections options.
>           if JSON object, must have a "sql" property
>           example : { dateStrings: true, sql: 'SELECT now()' }
> * `values`: *array | object* placeholder values. usually an array, but in case of only one placeholder, can be given as is. 
>
>Returns an Emitter object that emit different type of event:
>  * error : emit an [Error](#error) object if query failed. (No "end" event will be emit then).
>  * columns : emit when columns metadata from result-set are received (parameter is an array of [Metadata fields](#metadata-field)).
>  * data : emit each time a row is received (parameter is a row). 
>  * end : emit when query ended (no parameter). 

The function "query" return a promise returning all data at once. For huge result-set, that mean stored all data in memory. 
The function "queryStream" is very similar to query, but the event driven architecture will permit to handle a row one by one, to avoid overloading memory.   

If query time and result handle will take some amount of time, you may consider changing [@@net_read_timeout](https://mariadb.com/kb/en/library/server-system-variables/#net_read_timeout): 
Query must be received totally before this timeout (default to 60s).

example : 
```javascript
connection.queryStream("SELECT * FROM mysql.user")
      .on("error", err => {
        console.log(err); //if error
      })
      .on("columns", meta => {
        console.log(meta); // [ ...]
      })
      .on("data", row => {
        console.log(row);
      })
      .on("end", () => {
        //ended
      });
```

## `beginTransaction() → Promise`

>
>Returns a promise that :
>  * resolves (no argument)
>  * rejects with an [Error](#error).

Begin a new transaction.

## `commit() → Promise`

>
>Returns a promise that :
>  * resolves (no argument)
>  * rejects with an [Error](#error).

Commit current transaction, if there is any active.
(Driver does know current transaction state: if no transaction is active, no commands will be send to database) 


## `rollback() → Promise`

>
>Returns a promise that :
>  * resolves (no argument)
>  * rejects with an [Error](#error).

Rollback current transaction, if there is any active.
(Driver does know current transaction state: if no transaction is active, no commands will be send to database) 

Example : 
```javascript
conn.beginTransaction()
  .then(() => {
    conn.query("INSERT INTO testTransaction values ('test')");
    return conn.query("INSERT INTO testTransaction values ('test2')");
  })
  .then(() => {
    conn.commit();
  })
  .catch((err) => {
    conn.rollback();
  })
```
 
## `changeUser(options) → Promise`

> * `options`: *JSON*, subset of [connection option documentation](#connection-options) = database / charset / password / user
>
> Returns a promise that :
>   * resolves without result
>   * rejects with an [Error](#error).

This permit to resets the connection and re-authenticates with the given credentials. 
This is equivalent of creating a new connection with a new user, reusing open socket. 

Example : 
```javascript
conn.changeUser({user: 'changeUser', password: 'mypassword'})
   .then(() => {
      //connection user is now changed. 
   })
   .catch(err => {
      //error
   });
```

## `ping() → Promise`

>
>Returns a promise that :
>  * resolves (no argument)
>  * rejects with an [Error](#error).

Send to database a packet containing one byte to check that the connection is active.

Example : 
```javascript
conn.ping()
  .then(() => {
    //connection is valid
  })
  .catch(err => {
    //connection is closed
  })
```

## `isValid() → boolean`

> Returns a boolean

Indicates the state of the connection as the driver knows it.
Socket might be disconnected without connector still knowing it.

## `end() → Promise`

>Returns a promise that :
>  * resolves (no argument)
>  * rejects with an [Error](#error).

Gracefully end the connection. Connector will wait for current executing queries to finish, then close connection. 

Example : 
```javascript
conn.end()
  .then(() => {
    //connection has ended properly
  })
  .catch(err => {
    //connection was closed but not due of current end command
  })
```


## `destroy()`

Immediately close the connection. If there is a running query, it will be interrupted, and database will log this as an unexpected socket close. 

Example : 
```javascript
  conn.query(
      "select * from information_schema.columns as c1, " +
       "information_schema.tables, information_schema.tables as t2"
    )
    .then(rows => {
      //won't occur
    })
    .catch(err => {
      console.log(err);
      //Error: Connection destroyed, command was killed
      //    ...
      //  fatal: true,
      //  errno: 45004,
      //  sqlState: '08S01',
      //  code: 'ER_CMD_NOT_EXECUTED_DESTROYED' 
      done();
    });
  conn.destroy(); //will immediately close the connection, even if query above would have take a minute
```

## `pause()`

Pauses the reading of data.  

## `resume()`

Resume the pause. 


## `serverVersion()` 

> Returns a string 

Return current connected server version or throw an Error if not connected. 

Example : 
```javascript
  console.log(connection.serverVersion()); //10.2.14-MariaDB
```

## Error

On error, Promise return an [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) object with the following additional properties :
* fatal : *boolean* indicating if connection is still valid
* errno : error number. 
* sqlState : sql state code
* code : error code.

Example on console.log(error): 
```
{ Error: (conn=116, no: 1146, SQLState: 42S02) Table 'testn.falsetable' doesn't exist
  sql: INSERT INTO falseTable(t1, t2, t3, t4, t5) values (?, ?, ?, ?, ?)  - parameters:[1,0x01ff,'hh','01/01/2001 00:00:00.000',null]
      ...
      at Socket.Readable.push (_stream_readable.js:134:10)
      at TCP.onread (net.js:559:20)
    From event:
      at C:\mariadb-connector-nodejs\lib\connection.js:185:29
      at Connection.query (C:\mariadb-connector-nodejs\lib\connection.js:183:12)
      at Context.<anonymous> (C:\mariadb-connector-nodejs\test\integration\test-error.js:250:8)
    fatal: false,
    errno: 1146,
    sqlState: '42S02',
    code: 'ER_NO_SUCH_TABLE' } }
```

Errors contain error stack, query and parameter values (length limited to 1024 characters by default).
To get initial stack trace (the "From event ..."  part of previous example), connection option "trace" must be enable.  

See [error codes](https://mariadb.com/kb/en/library/mariadb-error-codes/) for error number and sql state signification.

## `events`

Connection object inherit from node.js [EventEmitter](https://nodejs.org/api/events.html), to emit 'error' event when connection close unexpectedly.

Example : 
```javascript
  const mariadb = require('mariadb');
  mariadb.createConnection({user: 'root', host: 'localhost', socketTimeout: 100})
  .then(conn => {
    conn.on('error', err => {
      //will be executed after 100ms due to inactivity, socket has closed. 
      console.log(err);
      //log : 
      //{ Error: (conn=6283, no: 45026, SQLState: 08S01) socket timeout
      //    ...
      //    at Socket.emit (events.js:208:7)
      //    at Socket._onTimeout (net.js:410:8)
      //    at ontimeout (timers.js:498:11)
      //    at tryOnTimeout (timers.js:323:5)
      //    at Timer.listOnTimeout (timers.js:290:5)
      //  fatal: true,
      //  errno: 45026,
      //  sqlState: '08S01',
      //  code: 'ER_SOCKET_TIMEOUT' }
    });
  })
  .catch(done);
```


