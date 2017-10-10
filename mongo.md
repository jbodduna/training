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
## find documents
- find documents in a range, greater than/less than
--  $gte, $gt, $lte, $lt, $not, $in
```
> db.foo.find({_id:{$gte:2, $lte:4 }},{name:1, things:2})
```
- matching elements in an array
-- {$in: ['one','two']}, {$notin: ['one','two']}
- dot notation, $exists
```
> db.animals.find({"info.canFly": true}).pretty()
```
- finding objects in a documents
-- dotNotation, 
### query criteira
### field selection
- use projects, with 0 and 1; 0 excludes the fild
### cursor operations
- cursor retrieves batch of documents
- sort does the sorting on server side
- use limit to limit the result set
- skip to skip specified number of documents, used for pagination
```
> var cursor = db.foo.find()
> cursor.size()
4
> cursor.hasNext();
true
> cursor.forEach(function(d){print(d.name)})
> db.foo.find({},{name:1}).sort({name:1})
> db.foo.find({},{name:1}).sort({name:1}).limit(1)
> db.foo.find({},{name:1}).sort({name:1}).skip(1).limit(1)
```
## indexing 
- indexing strategy 
-- regular (b-tree) - single or multiple field
-- geo - proximity or geo based; find elements near 
-- text index - parsing text fields and querying against text fields
-- hashed index - used in context of sharding so that keys/shards are more evently distributed
-- TTL index - date/time field on the document as an expiration period

- creating index
-- which field and what order, options - unique, sparse, TTL, language
-- listing indexes/check if an index exists
-- use explain to check if an index is used
-- indexes can be created to sub documents
-- sparse index will only create index for the document which have that field; when using sparse index need to make sure indexing strategy and querying match
-- compound index with two or more fields; order of fields and sort order determines if the index is used or not
-- deadweight - possible to create an index on field that does not exist on any document. mongo does not keep track of all the field names in a document
-- by default index creation happens in the foreground meaning it blocks all read access to the collection
-- by setting the background: true, index is created in the background; read and write operations all allowed. but this takes longer time
-- compound indexes can has upto 32 fields, max index name cannot be greater than 128 characters; to overcome the issue set the name of the index instead of letting mongo create one

```
> db.system.indexes.find({ns: 'test.foo'},{key: 1})
> db.foo.find({},{name:1}).sort({name:1}).skip(1).limit(1).explain()
> db.foo.ensureIndex({name:1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
> db.foo.find({name: 'obj 3'}).explain()
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.foo",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$eq" : "obj 3"
                        }
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "name" : 1
                                },
                                "indexName" : "name_1",
                                "isMultiKey" : false,
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 1,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[\"obj 3\", \"obj 3\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "LHPGSH12",
                "port" : 27017,
                "version" : "3.2.0",
                "gitVersion" : "45d947729a0315accb6d4f15a6b06be6d9c19fe7"
        },
        "ok" : 1        
}
> db.foo.dropIndex("name_1")
{ "nIndexesWas" : 2, "ok" : 1 }

> db.foo.ensureIndex({name:1}, {background: true})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```
