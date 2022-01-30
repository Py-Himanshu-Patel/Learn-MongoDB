# Creating, Updating, and Deleting Documents

## Insert
```js
> db.foo.insert({"bar" : "baz"})
```
Batch inserts allow you to pass an array of documents to the database.
```js
test> db.blog.insertMany([{"_id" : 0}, {"_id" : 1}, {"_id" : 2}]);
{ "acknowledged" : true, "insertedIds" : [ 0, 1, 2 ] }
db.blog.find()
"_id" : 0 }
"_id" : 1 }
"_id" : 2 }
```
Current versions of MongoDB do not accept messages longer than 48 MB, so there is a limit to how much can be inserted in a single batch insert. If you attempt to insert more than 48 MB, many drivers will split up the batch insert into multiple 48 MB batch inserts.

If you are importing a batch and a document halfway through the batch fails to be inserted, the documents up to that document will be inserted and everything after that document will not:
```js
> db.foo.batchInsert([{"_id" : 0}, {"_id" : 1}, {"_id" : 1}, {"_id" : 2}])
```
Only the first two documents will be inserted, as the third will produce an error: you cannot insert two documents with the same "_id". If you want to ignore errors and make batchInsert attempt to insert the rest of the
batch, you can use the `continueOnError`. To see the BSON size (in bytes) of the document, run `Object.bsonsize(doc)` from the shell.

## Remove
```js
db.foo.remove()
```
This will remove all of the documents in the foo collection. This doesn’t actually remove the collection, and any meta information about it will still exist. 
Remove everyone from the mailing.list collection where the value for "opt-out" is true:
```js
> db.mailing.list.remove({"opt-out" : true})
```
Once data has been removed, it is gone forever. There is no way to undo the remove or recover deleted documents Removing documents is usually a fairly quick operation, but if you want to clear an entire collection, it is faster to `drop` it (and then recreate any indexes on the empty collection).
```js
db.foo.drop()
```

## Update
`update` takes two parameters: a query document, which locates documents to update, and a modifier document, which describes the changes to make to the documents found.
Updating a document is atomic: if two updates happen at the same time, whichever one reaches the server first will be applied, and then the next one will be applied. Thus, conflicting updates can safely be sent in rapid-fire succession without any documents being corrupted: the last update will “win.”

```js
{
"_id" : ObjectId("4b2b9f67a1f631733d917a7a"),
"name" : "joe",
"friends" : 32,
"enemies" : 2
}
> var joe = db.users.findOne({"name" : "joe"});
> joe.relationships = {"friends" : joe.friends, "enemies" : joe.enemies};
{
"friends" : 32,
"enemies" : 2
}
> joe.username = joe.name;
"joe"
> delete joe.friends;
true
> delete joe.enemies;
true
> delete joe.name;
true
> db.users.update({"name" : "joe"}, joe);
> db.users.findOne()
{
	"_id" : ObjectId("4b2b9f67a1f631733d917a7a"),
	"username" : "joe",
	"relationships" : {
	"friends" : 32,
	"enemies" : 2
	}
}
```
Consider the data given
```js
> db.people.find()
{"_id" : ObjectId("4b2b9f67a1f631733d917a7b"), "name" : "joe", "age" : 65},
{"_id" : ObjectId("4b2b9f67a1f631733d917a7c"), "name" : "joe", "age" : 20},
{"_id" : ObjectId("4b2b9f67a1f631733d917a7d"), "name" : "joe", "age" : 49},
```
Now, if it’s Joe #2’s birthday, we want to increment the value of his "age" key, so we might say this:
```js
> joe = db.people.findOne({"name" : "joe", "age" : 20});
{
"_id" : ObjectId("4b2b9f67a1f631733d917a7c"),
"name" : "joe",
"age" : 20
}
> joe.age++;
> db.people.update({"name" : "joe"}, joe);
E11001 duplicate key on update
```
When you call update, the database will look for a document matching `{"name" : "joe"}`. The first one it finds will be the 65-year-old Joe. It will attempt to replace that document with the one in the joe variable, but there’s already a document in this collection with the same "_id". Thus, the update will fail, because "_id" values must be unique. The best way to avoid this situation is to make sure that your update always specifies a unique document, perhaps by matching on a key like "_id". For the example above, this would be the correct update to use:
```js
> db.people.update({"_id" : ObjectId("4b2b9f67a1f631733d917a7c")}, joe)
```
Using "_id" for the criteria will also be faster than querying on random fields, as "_id" is indexed.

### Using Modifiers

```bash
{
"_id" : ObjectId("4b253b067525f35f94b60a31"),
"url" : "www.example.com",
"pageviews" : 52
}
```
#### $inc
`$inc` modifier to increment the value of the "pageviews" key. This also create a new key with the same name if the key do not exists. Also we can increment the mentioned key by any number and not just 1. This works only on values of type integer, long or double, it fails on other type. 
```js
> db.analytics.update({"url" : "www.example.com"},
... {"$inc" : {"pageviews" : 1}})
```
Now, if we do a find, we see that "pageviews" has increased by one:
```js
> db.analytics.find()
{
"_id" : ObjectId("4b253b067525f35f94b60a31"),
"url" : "www.example.com",
"pageviews" : 53
}
```
#### $set
`$set` sets the value of a field. If the field does not yet exist, it will be created.
```js
> db.users.findOne()
{
"_id" : ObjectId("4b253b067525f35f94b60a31"),
"name" : "joe",
"age" : 30,
"sex" : "male",
"location" : "Wisconsin"
}
> db.users.update({"_id" : ObjectId("4b253b067525f35f94b60a31")},
... {"$set" : {"favorite book" : "War and Peace"}})
> db.users.findOne()
{
"_id" : ObjectId("4b253b067525f35f94b60a31"),
"name" : "joe",
"age" : 30,
"sex" : "male",
"location" : "Wisconsin",
"favorite book" : "War and Peace"
}
```

`$set` can even change the type of the key it modifies. For instance, if user decides that he actually likes quite a few books, he can change the value of the “favorite book” key into an array:
```js
> db.users.update({"name" : "joe"},
		{"$set" : {
						"favorite book" : 
							["Cat's Cradle", "Foundation Trilogy", "Ender's Game"]
					}
		}
	)
```
If the user realizes that he actually doesn’t like reading, he can remove the key altogether with "$unset":
```js
> db.users.update({"name" : "joe"},
... {"$unset" : {"favorite book" : 1}})
```
We can also update the fields nested in document. 
```js
> db.blog.posts.findOne()
	{
		"_id" : ObjectId("4b253b067525f35f94b60a31"),
		"title" : "A Blog Post",
		"content" : "...",
		"author" : {
			"name" : "joe",
			"email" : "joe@example.com"
		}
	}
> db.blog.posts.update({"author.name" : "joe"},
{"$set" : {"author.name" : "joe schmoe"}})
> db.blog.posts.findOne()
{
	"_id" : ObjectId("4b253b067525f35f94b60a31"),
	"title" : "A Blog Post",
	"content" : "...",
	"author" : {
		"name" : "joe schmoe",
		"email" : "joe@example.com"
	}
}
```
#### $push
`$push` adds elements to end of an array if the array exists and creates a new array if it does not.
```js
> db.blog.posts.findOne()
	{
		"_id" : ObjectId("4b2d75476cc613d5ee930164"),
		"title" : "A blog post",
		"content" : "..."
	}
> db.blog.posts.update({"title" : "A blog post"},
	{"$push" : {"comments" :
					{
						"name" : "joe", 
						"email" : "joe@example.com",
						"content" : "nice post."
					}
				}
	})
> db.blog.posts.findOne()
	{
		"_id" : ObjectId("4b2d75476cc613d5ee930164"),
		"title" : "A blog post",
		"content" : "...",
		"comments" : [{
			"name" : "joe",
			"email" : "joe@example.com",
			"content" : "nice post."
		}]
	}
```
Now, if we want to add another comment, we can simply use `$push` again:
```js
> db.blog.posts.update({"title" : "A blog post"},
	{"$push" : {"comments" :
					{
						"name" : "bob", 
						"email" : "bob@example.com",
						"content" : "good post."}}})
> db.blog.posts.findOne()
	{
		"_id" : ObjectId("4b2d75476cc613d5ee930164"),
		"title" : "A blog post",
		"content" : "...",
		"comments" : [
			{
				"name" : "joe",
				"email" : "joe@example.com",
				"content" : "nice post."
			},
			{
				"name" : "bob",
				"email" : "bob@example.com",
				"content" : "good post."
			}
		]
	}
```
#### $each
This is the “simple” form of push, but you can use it for more complex array operations as well. You can push multiple values in one operation using the `$each` suboperator:
```js
> db.stock.ticker.update({"_id" : "GOOG"},
	{"$push" : {"hourly" : {"$each" : [562.776, 562.790, 559.123]}}})
> db.stock.ticker.find()
	{ "_id" : ObjectId("61f651e7deba048ac2c49d5b"), "Name" : "DB Timer", "hourly" : [ 1, 2, 3 ] }
```
#### $slice and $sort
If you only want the array to grow to a certain length, you can also use the `$slice` operator in conjunction with `$push` to prevent an array from growing beyond a certain size, effectively making a “top N” list of items:
```js
> db.movies.find({"genre" : "horror"},{
	"$push" : {"top10" : {
	"$each" : [
					{"name" : "Nightmare on Elm Street", "rating" : 6.6},
					{"name" : "Saw","rating" : 4.3}
				],
	"$slice" : -10,
	"$sort" : {"rating" : -1}}}
})
```
This example would limit the array to the last 10 elements pushed. Slices must always be negative numbers. If the array was smaller than 10 elements (after the push), all elements would be kept. If the array was larger than 10 elements, only the last 10 elements would be kept.
This will sort all of the objects in the array by their "rating" field and then keep the first 10. Note that you must include "$each"; you cannot just `$slice` or `$sort` an array with `$push`.

#### $addToSet : (using arrays as sets)
You might want to treat an array as a set, only adding values if they are not present. This can be done using a `$ne`  (not equal) in the query document.
```js
> db.papers.update({"authors cited" : {"$ne" : "Richie"}},
... {$push : {"authors cited" : "Richie"}})
```
The better way to do this is using `$addToSet`
```js
> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
{
	"_id" : ObjectId("4b2d75476cc613d5ee930164"),
	"username" : "joe",
	"emails" : [
		"joe@example.com",
		"joe@gmail.com",
		"joe@yahoo.com"
	]
}
```
When adding another address, you can use `$addToSet` to prevent duplicates:
```js
> db.users.update({"_id" : ObjectId("4b2d75476cc613d5ee930164")},
	{"$addToSet" : {"emails" : "joe@gmail.com"}})
> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
{
	"_id" : ObjectId("4b2d75476cc613d5ee930164"),
	"username" : "joe",
	"emails" : [
		"joe@example.com",
		"joe@gmail.com",
		"joe@yahoo.com",
	]
}
> db.users.update({"_id" : ObjectId("4b2d75476cc613d5ee930164")},
	{"$addToSet" : {"emails" : "joe@hotmail.com"}})
> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
{
	"_id" : ObjectId("4b2d75476cc613d5ee930164"),
	"username" : "joe",
	"emails" : [
		"joe@example.com",
		"joe@gmail.com",
		"joe@yahoo.com",
		"joe@hotmail.com"
	]
}
```

#### $pop and $pull
`{"$pop" : {"key" : 1}}` removes an element from the end of the array. `{"$pop" :{"key" : -1}}` removes it from the beginning.

Sometimes an element should be removed based on specific criteria, rather than its position in the array. `$pull` is used to remove elements of an array that match the given criteria.
```js
> db.lists.insert({"todo" : ["dishes", "laundry", "dry cleaning"]})
> db.lists.update({}, {"$pull" : {"todo" : "laundry"}})
> db.lists.find()
{
	"_id" : ObjectId("4b2d75476cc613d5ee930164"),
	"todo" : [
		"dishes",
		"dry cleaning"
	]
}
```
Pulling removes all matching documents, not just a single match. If you have an array that looks like [1, 1, 2, 1] and pull 1, you’ll end up with a single-element array, [2].

#### Positional array modifications
There are two ways to manipulate values in arrays: by **position** or by using the **position operator** (the `$` character). Arrays use 0-based indexing.
```js
> db.blog.posts.findOne()
{
	"_id" : ObjectId("4b329a216cc613d5ee930192"),
	"content" : "...",
	"comments" : [
		{
			"comment" : "good post",
			"author" : "John",
			"votes" : 0
		},
		{
			"comment" : "i thought it was too short",
			"author" : "Claire",
			"votes" : 3
		},
		{
			"comment" : "free watches",
			"author" : "Alice",
			"votes" : -1
		}
	]
}
```
If we want to increment the number of votes for the first comment, we can say the following:
```js
> db.blog.update({"post" : post_id},
... {"$inc" : {"comments.0.votes" : 1}})
```
For example, if we have a user named John who updates his name to Jim, we can replace it in the comments by using the positional operator:
```js
db.blog.update({"comments.author" : "John"},
... {"$set" : {"comments.$.author" : "Jim"}})
```
The positional operator updates only the first match. Thus, if John had left more than one comment, his name would be changed only for the first comment he left.

#### Modifier speed
Some modifiers are faster than others. `$inc` modifies a document in place: it does not have to change the size of a document, only a couple of bytes, so it is very efficient. On the other hand, array modifiers might change the size of a document and can be slow. (`$set` can modify documents in place if the size isn’t changing but otherwise is subject
to the same performance limitations as array operators.)

## Upserts
An upsert is a special type of update. If no document is found that matches the update criteria, a new document will be created by combining the criteria and updated documents. If a matching document is found, it will be updated normally.

The manual process of upsert is like
```js
// check if we have an entry for this page
blog = db.analytics.findOne({url : "/blog"})
// if we do, add one to the number of views and save
if (blog) {
	blog.pageviews++;
	db.analytics.save(blog);
}
// otherwise, create a new document for this page
else {
	db.analytics.save({url : "/blog", pageviews : 1})
}
```
are also subject to a race condition where more than one document can be inserted for a given URL.
We can eliminate the race condition and cut down on the amount of code by just sending an upsert (the third parameter to update specifies that this should be an upsert):
```js
db.analytics.update({"url" : "/blog"}, {"$inc" : {"pageviews" : 1}}, true)
```
This line does exactly what the previous code block does, except it’s faster and atomic!

#### $setOnInsert
Sometimes a field needs to be seeded when a document is created, but not changed on subsequent updates. This is what `$setOnInsert` is for. `$setOnInsert` is a modifier that only sets the value of a field when the document is being inserted. Thus, we could do something like this:
```js
> db.users.update({}, {"$setOnInsert" : {"createdAt" : new Date()}}, true)
> db.users.findOne()
	{
		"_id" : ObjectId("512b8aefae74c67969e404ca"),
		"createdAt" : ISODate("2013-02-25T16:01:50.742Z")
	}
```
If we run this update again, it will match the existing document, nothing will be inserted, and so the "createdAt" field will not be changed:
```js
> db.users.update({}, {"$setOnInsert" : {"createdAt" : new Date()}}, true)
> db.users.findOne()
	{
		"_id" : ObjectId("512b8aefae74c67969e404ca"),
		"createdAt" : ISODate("2013-02-25T16:01:50.742Z")
	}
```

#### The save shell helper
`save` is a shell function that lets you insert a document if it doesn’t exist and update it if it does. It takes one argument: a document. If the document contains an "_id" key, save will do an upsert. Otherwise, it will do an insert. save is really just a convenience function so that programmers can quickly modify documents in the shell:
```js
> var x = db.foo.findOne()
> x.num = 42
42
> db.foo.save(x)
```
Without save, the last line would have been a more cumbersome `db.foo.update({"_id" : x._id}, x)`.

## Updating Multiple Documents
Updates, by default, update only the first document found that matches the criteria. If there are more matching documents, they will remain unchanged. **To modify all of the documents matching the criteria, you can pass `true` as the fourth parameter to update**.

Give a gift to every user who has a birthday on a certain day.
```js
> db.users.update({"birthday" : "10/13/1978"},
... {"$set" : {"gift" : "Happy Birthday!"}}, false, true)
```

To see the number of documents updated by a multiupdate, you can run the `getLastError` database command (which you can think of as “return information about the last operation”). The "n" key will contain the number of documents affected by the update:
```js
> db.count.update({x : 1}, {$inc : {x : 1}}, false, true)
> db.runCommand({getLastError : 1})
	{
		"err" : null,
		"updatedExisting" : true,
		"n" : 5,
		"ok" : true
	}
```
`n` is 5, meaning that five documents were affected by the update. `updatedExisting` is `true`, meaning that the update modified existing documents.

## Returning Updated Documents
We need to find the job with the highest priority in the **READY** state, run the process function, and then update the status to **DONE**. We might try querying for the ready processes, sorting by priority, and updating the status of the highest-priority process to mark it is "RUNNING". Once we have processed it, we update the status to "DONE". This looks something like the following:
```js
var cursor = db.processes.find({"status" : "READY"});
ps = cursor.sort({"priority" : -1}).limit(1).next();
db.processes.update({"_id" : ps._id}, {"$set" : {"status" : "RUNNING"}});
do_something(ps);
db.processes.update({"_id" : ps._id}, {"$set" : {"status" : "DONE"}});
```
This algorithm isn’t great because it is subject to a race condition. Suppose we have two threads running.

`findAndModify` can return the item and update it in a single operation. In this case, it looks like the following:
```js
> ps = db.runCommand({"findAndModify" : "processes",
						"query" : {"status" : "READY"},
						"sort" : {"priority" : -1},
						"update" : {"$set" : {"status" : "RUNNING"}
					})
{
	"ok" : 1,
	"value" : {
		"_id" : ObjectId("4b3e7a18005cab32be6291f7"),
		"priority" : 1,
		"status" : "READY"
	}
}
```
**status** is still **READY** in the returned document as `findAndModify` defaults to returning the document in its pre-modified state.
```js
> db.processes.findOne({"_id" : ps.value._id})
{
	"_id" : ObjectId("4b3e7a18005cab32be6291f7"),
	"priority" : 1,
	"status" : "RUNNING"
}
```
`findAndModify` can have either an **update** key or a **remove** key. A **remove** key indicates that the matching document should be removed from the collection. For instance, if we wanted to simply remove the job instead of updating its status, we could run the following:
```js
ps = db.runCommand({"findAndModify" : "processes",
"query" : {"status" : "READY"},
"sort" : {"priority" : -1},
"remove" : true}).value
do_something(ps)
```

The `findAndModify` command has the following fields:
- `findAndModify`: A string, the collection name 
- `query`: A query document; the criteria with which to search for documents
- `sort`: Criteria by which to sort results (optional)
- `update`: A modifier document; the update to perform on the document found (either this or "remove" must be specified)
- `remove`: Boolean specifying whether the document should be removed (either this or "update" must be specified)
- `new`: Boolean specifying whether the document returned should be the updated document or the pre-update document, to which it defaults
- `fields`: The fields of the document to return (optional)
- `upsert`: Boolean specifying whether or not this should be an upsert, and which defaults to `false`

## Setting a Write Concern
Write concern is a client setting used to describe how safely a write should be stored before the application continues. By default, inserts, removes, and updates wait for a database response—did the write succeed or not? before continuing. Generally, clients will throw an exception (or whatever the language’s version of an exception is) on failure.
There are a number of options available to tune exactly what you want the application to wait for. The two basic write concerns are acknowledged or unacknowledged writes. Acknowledged writes are the default: you get a response that tells you whether or not the database successfully processed your write. Unacknowledged writes do not return any response, so you do not know if the write succeeded or not.
