- mongo internal storage format is bson http://bson-spec.org
- every document need to have an id; mongo shell usually creates an id if one does not exist; not the mongo database but the shell which does that
- id can be of any type except array
- maximum size of a document is 64MB
- list objectid
- ObjectId contains timestamp
```
> ObjectId()
ObjectId("59dbc2bbde88115410295f61")
> ObjectId().getTimestamp();
ISODate("2017-10-09T18:51:43Z")  
```
## save and insert
- can you add 2 documents with the same id
-- using save operation will overwrite the previous document with the new one
-- using insert operation you will get 'duplicate key error'
```
> db.foo.insert({_id: 1, name: "object 4"})
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 11000,
                "errmsg" : "E11000 duplicate key error collection: test.foo index: _id_ dup key: { : 1.0 }"
        }
})
```
- pretty - prints documents in pretty format
```
> db.foo.find().pretty()
```
## update
- update command is atomic within a document,  2 client cannot update the same document at the same time ex: db.collection.update(query, update, options);
-- options : one, many, upsert (insert new if one does not exist); default is one
--  update an update only the required fields without touching other fields; ex: one client wants to increment value of a field, while other client wants to add a new field
--  set/unset can be used to update command to add/remove field
```
> db.foo.update({_id:4},{$inc:{x:1}})
> db.foo.update({_id:4},{$set:{newField:"adding new field"}})
> db.foo.update({_id:4},{$unset:{newField:''}})
```
## rename
- rename a field
## Array operators push/pull
- push add elements to an array
- addToSet adds only if the element does not exists
```
> db.foo.update({_id:1}, {$push: {things: 'one'}})
> db.foo.update({_id:1}, {$push: {things: 'two'}})
> db.foo.update({_id: 1}, {$addToSet: {things: 'four'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.foo.update({_id: 1}, {$addToSet: {things: 'four'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
> db.foo.update({_id: 1}, {$pop: {things: 1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```
## find and modify
- update only single document
