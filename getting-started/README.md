# Getting Started

- A **document** is the basic unit of data for MongoDB and is roughly equivalent to a **row** in a relational database management system (but much more expressive). **document** is an ordered set of keys with associated values. **document** cab bit have duplicate keys.
- Similarly, a **collection** can be thought of as a table with a dynamic schema.
- A single instance of MongoDB can host multiple independent databases, each of which can have its own collections.
- Every document has a special key, `"_id"`, that is unique within a collection.
- MongoDB comes with a simple but powerful JavaScript shell, which is useful for the administration of MongoDB instances and data manipulation.
- MongoDB is type-sensitive and case-sensitive. For example, these documents are distinct:
	```json
	{"foo" : 3}
	{"foo" : "3"}
	```
	as are as these:
	```json
	{"foo" : 3}
	{"Foo" : 3}
	```
- Key/value pairs in documents are ordered: `{"x" : 1, "y" : 2}` is not the same as `{"y" : 2, "x" : 1}`.


## Collections
- Collections have dynamic schemas. This means that the documents within a single collection can have any number of different “shapes.”
- You should not create any collections that start with `system.`, a prefix reserved for internal collections. For example, the `system.users` collection contains the database’s users, and the `system.namespaces` collection contains information about all of the database’s collections.

## Databases
- A database has its own permissions, and each database is stored in separate files on disk. A good rule of thumb is to store all data for a single application in the same database. Separate databases are useful when storing data for
several application or users on the same MongoDB server.
- Database names are case-sensitive, even on non-case-sensitive filesystems and are limited to 64 bytes.
- One thing to remember about database names is that they will actually end up as files on your filesystem.

## Get started with MongoDB
- MongoDB is almost always run as a network server that clients can connect to and perform operations on. Start mongo shell like this
```bash
~ $ mongo --quiet
test>
```

When run with no arguments, `mongod` will use the default data directory, `/data/db/`. If the data directory does not already exist or is not writable, the server will fail to start. It is important to create the data directory (e.g., `mkdir -p /data/db/`) and to make sure your user has permission to write to the directory before starting MongoDB.

Check for status of `mongod` process as:
```bash
$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-01-29 20:09:18 IST; 2s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 12782 (mongod)
     Memory: 160.7M
        CPU: 822ms
     CGroup: /system.slice/mongod.service
             └─12782 /usr/bin/mongod --config /etc/mongod.conf
```

If it is not active then either start it or make it start on boot as:
```bash
$ sudo systemctl start mongod	# start the server
$ sudo systemctl enable mongod	# start on startup
```

By default MongoDB listens for socket connections on port **27017**. The server will fail to start if the port is not available the most common cause of this is another instance of MongoDB that is already running.

`mongod` also sets up a very basic HTTP server that listens on a port 1,000 higher than the main port, in this case `28017`. This means that you can get some administrative information about your database by opening a web browser and going to http://localhost:28017.

- The shell is a full-featured JavaScript interpreter, capable of running arbitrary JavaScript programs. We can also leverage all of the standard JavaScript libraries.
	```js
	> Math.sin(Math.PI / 2);
	1
	> new Date("2010/1/1");
	"Fri Jan 01 2010 00:00:00 GMT-0500 (EST)"
	> "Hello, World!".replace("World", "MongoDB");
	Hello, MongoDB!
	```
- To see the database db is currently assigned to, type in db and hit Enter:
	```js
	> db
	test
	```
- Change database like this:
	```js
	> use foobar
	switched to db foobar
	> db
	foobar
	```

## Basic Operations with the Shell

#### Create
Create a local variable called post that is a JavaScript object representing our document.
```js
> post = {"title" : "My Blog Post",
... "content" : "Here's my blog post.",
... "date" : new Date()}
{
"title" : "My Blog Post",
"content" : "Here's my blog post.",
"date" : ISODate("2012-08-24T21:12:09.982Z")
}
> db.blog.insert(post)
> db.blog.find()
{
"_id" : ObjectId("5037ee4a1084eb3ffeef7228"),
"title" : "My Blog Post",
"content" : "Here's my blog post.",
"date" : ISODate("2012-08-24T21:12:09.982Z")
}
```

#### Read
`find` and `findOne` can be used to query a collection. If we just want to see one document from a collection, we can use `findOne`:
```js
> db.blog.findOne()
{
"_id" : ObjectId("5037ee4a1084eb3ffeef7228"),
"title" : "My Blog Post",
"content" : "Here's my blog post.",
"date" : ISODate("2012-08-24T21:12:09.982Z")
}
```

#### Update
`update` takes (at least) two parameters: the first is the criteria to find which document to update, and the second is
the new document.
```js
> post.comments = []
[ ]
> db.blog.update({title : "My Blog Post"}, post)
> db.blog.find()
{
"_id" : ObjectId("5037ee4a1084eb3ffeef7228"),
"title" : "My Blog Post",
"content" : "Here's my blog post.",
"date" : ISODate("2012-08-24T21:12:09.982Z"),
"comments" : [ ]
}
```

#### Delete
`remove` permanently deletes documents from the database. Called with no parameters, it removes all documents from a collection. It can also take a document specifying criteria for removal.
```js
> db.blog.remove({title : "My Blog Post"})
```

## Data Types

- `null`: Null can be used to represent both a null value and a nonexistent field: `{"x" : null}`

- `boolean`: There is a boolean type, which can be used for the values true and false: `{"x" : true}`

- `number`: The shell defaults to using 64-bit floating point numbers. Thus, these numbers look
“normal” in the shell: `{"x" : 3.14}` or: `{"x" : 3}`. For integers, use the `NumberInt` or `NumberLong` classes, which represent 4-byte or 8-byte signed integers, respectively. `{"x" : NumberInt("3")}` and `{"x" : NumberLong("3")}` 

- `string`: Any string of UTF-8 characters can be represented using the string type: `{"x" : "foobar"}`

- `date`: Dates are stored as milliseconds since the epoch. The time zone is not stored: `{"x" : new Date()}`. A new `Date` object, always call `new Date(...)`, not just `Date(...)`. Calling the constructor as a function (that is, not including new) returns a string representation of the date, not an actual Date object. This is not MongoDB’s choice; it is how JavaScript works.

- `regular expression`:  Queries can use regular expressions using JavaScript’s regular expression syntax: `{"x" : /foobar/i}`

- `array`: Sets or lists of values can be represented as arrays: `{"x" : ["a", "b", "c"]}`

- `embedded document`: Documents can contain entire documents embedded as values in a parent document: `{"x" : {"foo" : "bar"}}`

- `object id`: An object id is a 12-byte ID for documents `{"x" : ObjectId()}`. In a single collection, every document
must have a unique value for `"_id"`, which ensures that every document in a collection can be uniquely identified. That is, if you had two collections, each one could have a document where the value for `"_id"` was 123. However, neither collection could contain more than one document with an `"_id"` of 123. MongoDB’s distributed nature is the main reason why it uses `ObjectIds` as opposed to something more traditional, like an autoincrementing primary key: it is difficult and time-consuming to synchronize autoincrementing primary keys across multiple servers. Because MongoDB was designed to be a distributed database, it was important to be able to generate unique identifiers in a sharded environment. `ObjectIds` use **12 bytes** of storage, which gives them a string representation that is 24 hexadecimal digits: 2 digits for each byte.

- `binary data`: Binary data is a string of arbitrary bytes. It cannot be manipulated from the shell. Binary data is the only way to save non-UTF-8 strings to the database.

- `code`: Queries and documents can also contain arbitrary JavaScript code: `{"x" : function() { /* ... */ }}`

## Connect to MongoDB Shell
To connect to a mongod on a different machine or port, specify the hostname, port, and database when starting the
shell:
```js
$ mongo some-host:30000/myDB
MongoDB shell version: 2.4.0
connecting to: some-host:30000/myDB
>
```
Sometimes it is handy to not connect to a mongod at all when starting the mongo shell. If you start the shell with `--nodb`, it will start up without attempting to connect to anything:
```
$ mongo --nodb
MongoDB shell version: 2.4.0
>
```
Once started, you can connect to a `mongod` at your leisure by running `new Mongo(hostname)`:
```js
> conn = new Mongo("some-host:30000")
connection to some-host:30000
> db = conn.getDB("myDB")
myDB
```

Some useful shells
```js
show dbs
show collections
show users
```
A good way of figuring out what a function is doing is to type it without the parentheses. This will print the JavaScript source code for the function.

## Running Scripts with the Shell
Other chapters have used the shell interactively, but you can also pass the shell JavaScript files to execute. Simply pass in your scripts at the command line:
```js
$ mongo script1.js script2.js script3.js
MongoDB shell version: 2.4.0
connecting to: test
I am script1.js
I am script2.js
I am script3.js
```
The mongo shell will execute each script listed and exit.

If you want to run a script using a connection to a non-default host/port mongod, specify the address first, then the script(s):
```bash
$ mongo --quiet server-1:30000/foo script1.js script2.js script3.js
```

You can also run scripts from within the interactive shell using the `load()` function:
```js
> load("script1.js")	// relative path to js script from the place where mongo shell started
I am script1.js
```
```js
// defineConnectTo.js
/**
* Connect to a database and set db.
*/
var connectTo = function(port, dbname) {
	if (!port) {
		port = 27017;
	}
	if (!dbname) {
		dbname = "test";
	}
	db = connect("localhost:"+port+"/"+dbname);
	return db;
};
```
Run this script in mongo shell
```js
> typeof connectTo
undefined
> load('defineConnectTo.js')
> typeof connectTo
function
```

## Creating a .mongorc.js

```bash
~ $ pwd
/home/himanshu
~ $ 
~ $ cat .mongorc.js 
print("The only difference in winner and loser is : persistence");

prompt = function() {
	if (typeof db == 'undefined') {
		return '(nodb)> ';
	}
	// Check the last db operation
	try {
		db.runCommand({getLastError:1});
	}
	catch (e) {
		print(e);
	}
	return db+"> ";
};
```

One of the most common uses for `.mongorc.js` is remove some of the more “dangerous” shell helpers. You
can override functions like `dropDatabase` or `deleteIndexes` with no-ops or undefine them altogether:
```js
var no = function() {
	print("Not on my watch.");
};
// Prevent dropping databases
db.dropDatabase = DB.prototype.dropDatabase = no;
// Prevent dropping collections
DBCollection.prototype.drop = no;
// Prevent dropping indexes
DBCollection.prototype.dropIndex = no;
```
Make sure that, if you change any database functions, you do so on both the db variable and the DB prototype (as shown in the example above). If you change only one, either the db variable won’t see the change or all new databases you use (when you run use anotherDB) won’t see your change.

You can disable loading your `.mongorc.js` by using the `--norc` option when starting the shell.

## Editing Complex Variables
Now, if you want to edit a variable, you can say "edit varname", for example:
```js
> EDITOR="/usr/bin/emacs"
> var wap = db.books.findOne({title: "War and Peace"})
> edit wap
```
Add `EDITOR="/path/to/editor"`; to your `.mongorc.js` file and you won’t have to worry about setting it again.


## Inconvenient Collection Names
Fetching a collection with the db.collectionName syntax almost always works, unless the collection name a reserved word or is an invalid JavaScript property name.
```js
> db.version
function () {
return this.serverBuildInfo().version;
}
```
To actually access the version collection, you must use the `getCollection` function:
```js
> db.getCollection("version");
test.version
```
This can also be used for collection names with characters that aren’t valid in JavaScript property names, such as `foo-bar-baz` and `123abc` (JavaScript property names can only contain letters, numbers, "$" and "_" and cannot start with a number).
In Java‐Script, `x.y` is identical to `x['y']`. This means that subcollections can be accessed using variables, not just literal names. You can use this technique to access awkwardly-named collections:
```js
> var name = "@#&!"
> db[name].find()
```
Attempting to query `db.@#&!` would be illegal, but `db[name]` would work.
