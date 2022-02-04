# Querying

An empty query document (i.e., `{}`) matches everything in the collection. If find isn’t given a query document, it defaults to {}. For example, the following:
```js
> db.c.find()
```
matches every document in the collection c (and returns these documents in batches).

## Specifying Which Keys to Return
if you have a user collection and you are interested only in the "user name" and "email" keys, you could return just those keys with the following query:
```js
> db.users.find({}, {"username" : 1, "email" : 1})
{
"_id" : ObjectId("4ba0f0dfd22aa494fd523620"),
"username" : "joe",
"email" : "joe@example.com"
}
```

You can also use this second parameter to exclude specific key/value pairs from the results of a query.
```js
> db.users.find({}, {"username" : 1, "_id" : 0})
{
	"username" : "joe",
}
```

## Query Conditionals
`$lt`, `$lte`, `$gt`, and `$gte` are all comparison operators, corresponding to `<`, `<=`, `>`, and `>=`, respectively. They can be combined to look for a range of values. For example, to look for users who are between the ages of 18 and 30, we can do this:
```js
test> db.ages.insert([{"ID": 1},{"ID": 2},{"ID":3}])
test> db.ages.find()
{ "_id" : ObjectId("61f6adcfdeba048ac2c49d5c"), "ID" : 1 }
{ "_id" : ObjectId("61f6adcfdeba048ac2c49d5d"), "ID" : 2 }
{ "_id" : ObjectId("61f6adcfdeba048ac2c49d5e"), "ID" : 3 }
test> db.ages.find({"ID": {"$gte": 1, "$lte": 2}})
{ "_id" : ObjectId("61f6adcfdeba048ac2c49d5c"), "ID" : 1 }
{ "_id" : ObjectId("61f6adcfdeba048ac2c49d5d"), "ID" : 2 }
```

To find people who registered before January 1, 2007, we can do this:
```js
> start = new Date("01/01/2007")
> db.users.find({"registered" : {"$lt" : start}})
```

`$ne`, which stands for “not equal.” If you want to find all users who do not have the username “joe,” 
```js
> db.users.find({"username" : {"$ne" : "joe"}})
```

## OR Queries
```js
> db.raffle.find({"ticket_no" : {"$in" : [725, 542, 390]}})
// also
> db.users.find({"user_id" : {"$in" : [12345, "joe"]})
```
`$in` is very flexible and allows you to specify criteria of different types as well as values.
The opposite of `$in` is `$nin`, which returns documents that don’t match any of the criteria in the array. If we want to return all of the people who didn’t win anything in the raffle, we can query for them with this:
```js
> db.raffle.find({"ticket_no" : {"$nin" : [725, 542, 390]}})
```

`$or` takes an array of possible criteria.
```js
> db.raffle.find({"$or" : [{"ticket_no" : 725}, {"winner" : true}]})
```
`$or` can contain other conditionals. If, for example, we want to match any of the three
"ticket_no" values or the "winner" key, we can use this:
```js
> db.raffle.find({"$or" : [
		{"ticket_no" : {"$in" : [725, 542, 390]}},
		{"winner" : true}
		]
	})
```

## Not Operator: $not
`$not` is a metaconditional: it can be applied on top of any other criteria. As an example, let’s consider the modulus operator, `$mod`. `$mod` queries for keys whose values, when divided by the first value given, have a remainder of the second value:
```js
> db.users.find({"id_num" : {"$mod" : [5, 1]}})
```
The previous query returns users with "id_num"s of 1, 6, 11, 16, and so on. If we want,
instead, to return users with "id_num"s of 2, 3, 4, 5, 7, 8, 9, 10, 12, etc., we can use `$not`:
```js
> db.users.find({"id_num" : {"$not" : {"$mod" : [5, 1]}}})
```

## Conditional Semantics
```js
> db.users.find({"$and" : [{"x" : {"$lt" : 1}}, {"x" : 4}]})
> db.users.find({"x" : {"$lt" : 1, "$in" : [4]}})
```

## Type-Specific Queries

#### null
`null` behaves a bit strangely. It does match itself, so if we have a collection with the following documents:
```js
> db.c.find()
{"_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null }
{"_id" : ObjectId("4ba0f0dfd22aa494fd523622"), "y" : 1 }
{"_id" : ObjectId("4ba0f148d22aa494fd523623"), "y" : 2 }
```
We can query for documents whose "y" key is null in the expected way:
```js
> db.c.find({"y" : null})
{ "_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null }
```
However, null not only matches itself but also matches “does not exist.” Thus, querying for a key with the value null will return all documents lacking that key:
```js
> db.c.find({"z" : null})
{"_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null }
{"_id" : ObjectId("4ba0f0dfd22aa494fd523622"), "y" : 1 }
{"_id" : ObjectId("4ba0f148d22aa494fd523623"), "y" : 2 }
```
If we only want to find keys whose value is null, we can check that the key is null and exists using the `$exists` conditional:
```js
> db.c.find({"z" : {"$in" : [null], "$exists" : true}})
```

#### Regular Expressions
If we want to find all users with the name Joe or joe, we can use a regular expression to do case-insensitive matching:
```js
> db.users.find({"name" : /joe/i})		// joe Joe JOe JOE
> db.users.find({"name" : /joey?/i})	// joeya joeys joeyk
```
MongoDB uses the Perl Compatible Regular Expression (PCRE) library to match regular expressions;
Regular expressions can also match themselves. Very few people insert regular expressions into the database, but if you insert one, you can match it with itself:
```js
> db.foo.insert({"bar" : /baz/})
> db.foo.find({"bar" : /baz/})
{
"_id" : ObjectId("4b23c3ca7525f35f94b60a2d"),
"bar" : /baz/
}
```

#### Querying Arrays
If the array is a list of fruits, like this:
```js
> db.food.insert({"fruit" : ["apple", "banana", "peach"]})
```
the following query will successfully match the document.
```js
> db.food.find({"fruit" : "banana"})
```

#### $all
If you need to match arrays by more than one element we need to use `$all`.
```js
> db.food.insert({"_id" : 1, "fruit" : ["apple", "banana", "peach"]})
> db.food.insert({"_id" : 2, "fruit" : ["apple", "kumquat", "orange"]})
> db.food.insert({"_id" : 3, "fruit" : ["cherry", "banana", "apple"]})
```
Then we can find all documents with both "apple" and "banana" elements by querying with "$all":
```js
> db.food.find({"fruit" : {$all : ["apple", "banana"]}})
{"_id" : 1, "fruit" : ["apple", "banana", "peach"]}
{"_id" : 3, "fruit" : ["cherry", "banana", "apple"]}
```
You can also query by exact match using the entire array. However, exact match will not match a document if any elements are missing or superfluous.
```js
> db.food.find({"fruit" : ["apple", "banana", "peach"]})	// first doc
> db.food.find({"fruit" : ["apple", "banana"]})				// no result
> db.food.find({"fruit" : ["banana", "apple", "peach"]})	// no result
```

#### $size
A useful conditional for querying arrays is "$size", which allows you to query for arrays of a given size. Here’s an example:
```js
> db.food.find({"fruit" : {"$size" : 3}})
```

#### $slice
