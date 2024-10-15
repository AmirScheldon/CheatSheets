# MongoDB
1. instead of TABLE it has COLLECTION
2. instead of ROW and COLUMN it has DOCUMENT
3. it is an schema-less DB.


> Data is stored in BSON (binary on the server side) formats in document databases. you write data as JSON and mongodb drivers convaert it into BSON data. BSON data are more efficient in storagge

# import Data

**--jsonArray**:
> This option tells mongoimport to treat the input as an array of JSON documents, rather than individual documents separated by newlines.

**--drop**:
> When you want to replace the entire collection with fresh data.

```
mongoimport FileName -d DataBaseName -c CollectionName --jsonArray --drop
```

# Cursor
> cursor is a pointer that references the documents of the collection returned by the find() method. example: You have a query that meets 1000 docs. You have a website where you only display 10 items at a time. There is no need to load all 1000 results that matches your query right at the start, instead, only fetch first 10, display them on the screen and then fetch next 10 when user navigates to the next page.


> it reutrns the same doc each time

`db.movies.find().next()`


> it iterate over docs
```
const cursor = db.movies.find()
cursor.next()
```


# CRUD
1. **Create**
> NOTE: to use insertMany you should insert data in list form:\

`insertMany([data], options)`
```
insertOne(data, options)
insertMany(data, options)
```

**Ordered Insert**:
> Each element you insert is processed independently, but if one fails, it cancels the entire insert operation. However, it does not roll back the elements that have already been inserted. This is the default behavior of MongoDB.

overrirde behaviour:
```
db.CollectionName.insertMany([{}], {ordered: false})
```
> you still encounter erros but MongoDb will continue to process the operation. if it find non-duplicate data, it will insert data into the collection.  

**WriteConcern**:
1. **w**:

    w: 1 => The write operation is acknowledged by the primary node only (default).

    w: 0 => No acknowledgment is required. The operation is sent to the server, and the client does not wait for a response.

2. **j (Journaling)**:
>Data Durability:\
>  Journaling ensures that even if the server crashes, the write operations can be recovered. When `j: true` is specified, MongoDB guarantees that the write operation has been written to the journal file on disk before acknowledging the write to the client.\
>Crash Recovery:\
>   If the server crashes after the write operation is acknowledged but before the data is fully written to the database's data files, MongoDB can use the journal file to replay and recover the write operations. This helps prevent data loss in case of unexpected failures.

    j: true => Ensures that the write operation is written to the journal on the primary node before acknowledgment.

    j: false => The operation does not wait for journaling, which might be faster but less durable.

3. **wtimeout**:
>This option specifies the maximum time (in milliseconds) that the server should wait for the write concern to be met. If the time limit is exceeded, a timeout error is returned.

```
db.collectionName.insertOne({}, {writeConcern: {w: 1, j: 0, wtimeout: 200}})
```

**Atomicity**:
> MongoDB guarantees that the document will be completely stored. If there is any error, the data will be rolled back, including any embedded documents.

2. **Read**

Two OPERATORS:
>greater than\
`find({key: {$gt: number}} , options)`

>lower than\
`find({key: {$lt: number}} , options)`

> find func gives you a cursor(first 20 objects) of data, not all data. To get all data use:
1. `db.collectionName.find().toArray()`
2. `db.collectionName.find().forEach(func in js)`

> find all matchs data\
`
find(filter, options)
`

> find only first match data\
`
findOne(fiter, options)
`

> To find an array that includes Drama\
`
db.movies.find({genres: 'Drama'})
`

> To find an array that just have Drama\
`
db.movies.find({genres: ['Drama']})
`


Operators
1. **Query selectors:**
  * *Comparison*
    * $ne => not equal
    * $eq
    * $lt
    * $lte
    * $gt
    * $gte => greater than or equal
    * $in 
      > this is not a range. runtime must be 30 or 40\
      `
      db.movies.find({runtime: {$in: [30, 40]}})
      `
    * $nin
    
  * *Evaluation*
    * $regex
    > Provides regular expression capabilities for pattern matching within strings in queries. It is particularly useful when the exact field value is unknown which allows for flexible and efficient searches within collections.\
    `
    db.movies.findOne({summary: {$regex: /musical/}})
    `
    * $expr
    > it return where the Volume(shows by `'$FieldName'`) field is greater than the Target field.\
    `
    db.sales.find({$expr: {$gt: ['$volume', '$target']}})
    `
    > $cond and if statement
    ```
    db.sales.find({$expr: {$gt: [{$cond: {if: {$gt: ['$volume', 90]}, then: {$subtract: ['$volume', 5]}, else: '$volume'}}, '$target']}}) 
    ```
  * *logical*
    * $and\
    `
    db.movies.find({$and : [{'rating.average': {$gt : 8}}, {'rating.average': {$lt : 9}}]})
    `
    > Note: MongoDB uses the AND operator between fields by default.\
    ```
    db.movies.find({{'rating.average': {$gt : 8}}, {'rating.average': {$lt : 9}}})
    ```

    > Note: It does not use the AND operator between the same fields, as in JSON, the second key overrides the first one."
    ```
    db.movies.find({genres : 'Drama'}, {genres : 'Horror'}).count()
    output: 153

    db.movies.find({$and : [{genres : 'Drama'}, {genres : 'Horror'}]}).count()
    output: 17
    ```
    
    * $not
    * $nor\
    `
     db.movies.find({$nor : [{'rating.average': {$eq : 5.6}}, {'rating.average': {$eq : 9}}]})
    `
    * $or\
    `
     db.movies.find({$or : [{'rating.average': {$eq : 5.6}}, {'rating.average': {$eq : 9}}]})
    `
  * *Array*
    querying from embeded document.\
    ```
    {hobbies:[{title: 'work', frequency: 6}]}
    db.users.find({'hobbies.title': 'work'})
    ```
    * $size => retrieve the exact size.
    `db.users.find({hobbies: {$size: 3}})`

    * $all => retrieve all elements are exists in the field. does not care about order.\
    `db.movies.find({genres: {$all: ['Crime', 'Drama']}})`

    * $elemMatch => matches documents that contain an array field with at least one element that matches all the specified query criteria.\
    `db.users.find({hobbies: {$elemMatch: {title: 'football', frequency: {$gte: 3}}}})`

  * *Elements*
    * $exists
    `
    db.users.find({age : {$exists : true}})
    `

    * $type
    `
    db.users.find({phone: {$type: ['string', 'number']}})
    `
  * Comments
  * Geospatial

2. **Projection operators:**
  * $
  > shows only the element that you wanted to retrieve in filter part.\
  `db.movies.find({genres: 'Drama'}, {'genres.$': 1})`

  * $elementMatch
  > you can take out some data from fetched data that you want to show it to user.\
  `db.movies.find({genres: 'Drama'}, {genres: {$elemMatch: {$eq: 'Horror'}}})`

  > show fields which are same to filter in all properties you defined.\

  it finds users who have a hobby titled "football" with a frequency of 3 or more.
  `db.users.find({hobbies: {$elemMatch: {title: 'football', frequency: {$gte: 3}}}})` 

  * $meta
  * $slice
  > shows amount element that you want to show.\
  `db.movies.find({genres: 'Drama'}, {genres: {$slice: 2})`

  * $slice: [NumberOfPassedData, NumberOfShowingData]
  `db.movies.find({genres: 'Drama'}, {genres: {$slice: [1, 2]}})`

  original data =>genres: [ 'Drama', 'Action', 'Crime' ]\
  showed data => ['Action', 'Crime' ]
  


# Pagination

**sort()**
> sort doc that showed by cursor. `'FieldName': 1` means ascending and `'FieldName': -1` means descending.

`db.movies.find().sort({'FieldName': 1})`

you can chain sorting Fields.

`db.movies.find().sort({'FieldName': 1, 'FieldNameOne': -1})`

**skip()**
> skip 100 docs

`db.movies.find().sort().skip(100)`

**limit()**
> limit the amount of elements that the cursor should retrieve at a time.

`db.movies.find().sort().skip(100).limit(10)`

> does the order of this funcs matter? No. you can put them on whatever order you want to and MongoDB will put it on right order(1.sort 2.skip 3.limit)

**Projection**
> A way to filter data in MongoDb. it means just gives you part of data which you needed.

`db.collectionName.find({}, {name: 1, _id: 0})`\
* first curly brucket means you need all data.
* second curly brucket means which part of data you need. `1` means you need this part of data and `0` means you do not need this. `_id` default is `1`.


3. **Update**

> NOTE: to update a document you must use:\
`
db.collectionName.updateOne(filter, {$set: data}, options)
`
```
update(filter, data, options)
updateOne(filter, data, options)
updateMany(filter, data, options)
replaceOne(filter, data, options)
```

update vs updateOne and updateMany:\
    1. update and updateMany both update all matching elements.\
    2. in updateOne and updateMany you must use $set method but it is not necessary in update.\
    3. update override document.


Operators

  * $set
  > overrides or add some data to document\

  `db.users.updateMany({'hobbies.title': 'football'},{$set:{isfootball: true}})`

  * $inc
  > increase and decrease numeric fields, `NameField: +` means increase, `NameField: -` decrease.

  `db.users.updateOne({name: 'mohammad'}, {$inc: {age: 2}}, {$set: {isFootball: false}})`

  > Note: You can use multiple update option in one command but rhe must effect on diffrent field

  * $min
  > it set new value if only the new value is lower than the previous one.\
  `db.users.updateOne({name: 'mohammad'}, {$min: {age: 25}})`

  * $max
  > it set new value if only the new value is greater than the previous one.\
  `db.users.updateOne({name: 'mohammad'}, {$max: {age: 28}})`

  * $mul
  > it multiple a field to a value.\
  ` db.users.updateOne({name: 'mohammad'}, {$mul: {age: 1.1}})`

  * $unset
  > drops a field/fields.\
  `db.users.updateMany({isfootball: true}, {$unset: {phone: ''}})`

  * $rename
  `db.users.updateMany({}, {$rename: {isfootball: 'isFootball'}})`

  * $upsert
  > It is a combination of update and insert. We want to update a field, but we are not sure if the field exists or not. If it does not exist, insert the data; if it does exist, update the data.\
  `db.users.updateMany({name: 'sozan'}, {$set: {hobbies: [{title: 'art', frequency: 3}], age: 21, isFootball: false}}, {upsert: true})` 

  * $ and $elemMatch
  > `$` is a placeholder which you can use it in a path to a field to quickly acces to match element.it gives you first matching elements, not all element.\

  in the users collection where there is a hobby titled "football" with a frequency greater than or equal to 3. It sets a new field, IsHighFrequency, to true for that specific hobby in the hobbies array.

  `db.users.updateMany({hobbies: {$elemMatch: {title: 'football', frequency: {$gte: 3}}}}, {$set: {'hobbies.$.IsHighFrequency': true}})`

  * $[]
  > it means update all element and for each element upadate named field.\

  it decreases the frequency of all hobbies for all users who are older than 30 by 1. It utilizes the $[] array filter to update all elements in the hobbies array.
  `db.users.updateMany({age: {$gt: 30}}, {$inc: {'hobbies.$[].frequency': -1}})`

  => $[<identifier>]

  it sets a new field, IsGood, to true for all elements in the hobbies array where the frequency is greater than or equal to 3. This is done using arrayFilters to filter specific elements of the array that match the condition.

  `db.users.updateMany({'hobbies.frequency': {$gte: 3}}, {$set: {'hobbies.$[el].IsGood': true}}, {arrayFilters: [{'el.frequency': {$gte: 3}}]})`

  * $push
  > it pushes elements to array without changing previouse datas. you can add non-uniques value by using this operator.

  It adds a new hobby to the hobbies array for a user whose name is "sozan". The $push operator is used to append an element to an array field in a MongoDB document.

  `db.users.updateOne({name: 'sozan'}, {$push: {hobbies: {title: 'shoping', frequency: 4}}})`

  => $push and $each and $sort

   it adds multiple new hobbies to the hobbies array for a user named "sozan" and sorts the array in descending order based on the frequency field after adding the new elements.

  `db.users.updateOne({name: 'sozan'}, {$push: {hobbies: {$each: [{title: 'walking', frequency: 1}, {title: 'biking', frequency: 3}], $sort: {frequency: -1}}}})`

  * $addToSet
  > it adds only unique values to array.\
  `db.users.updateOne({name: 'sozan'}, {$addToSet: {hobbies: {title: 'walking', frequency: 1}}})`

  * $pull
  > it removes element from an array.\
  `db.users.updateOne({name: 'sozan'}, {$pull: {hobbies: {title: 'biking'}}})`

  * $pop
  > it removes element from end or begining of an array. `FieldName: -1` means begining of array, `FieldName: 1` vice versa.\
  `db.users.updateOne({name: 'sozan'}, {$pop: {hobbies: 1}})`

4. **Delete**
```
deleteOne(filter, options)
db.users.deleteOne({name: 'sozan'})


deleteMany(filter, options)
db.users.deleteMany({isFootball: true, 'hobbies.title': 'football'})

=> it emptys the collection.
db.users.deleteMany({})

=> it drops/delete the collection
db.users.drop()

=> it drops entire database.
db.dropDatabase() 
```

# Indexes
> The indexes are ordered by the value of the field specified in the index. Indexes are special data structures. Without indexing, MongoDB must scan every document in a collection to retrieve the matching documents and leading to slower query performance. However, indexing leads to faster find (retrieval) operations, but it also makes the insert process slower.\
> Note: if you have queries that regulary return the majority or all of your documents, index would not really help you there. it might be slow down the execution(bcs there is an extra step: fetching data from index pointer).\
> it helps you to sort large amount of data, bcz in large amount of data `sort()` leads to Timeout. to solve this problem you can use indexing and `sort()` at the same time.


to get all indexes:
> every collection has a default index which is `_id` index.

`db.persons.getIndexes()`

before indexing:\
`db.persons.explain('executionStats').find({'dob.age': {$gt: 60}})`

* it has `stage: COLLSCAN` in executionStages.stage. it scans entire Collection to find data.
* executionTimeMillis late more than indexed search.

indexing: 
> `fieldName: 1` means ascending order and `fieldName: -1` means descending order
`db.persons.createIndex({'dob.age': 1})`

after indxing: \
`db.persons.explain('executionStats').find({'dob.age': {$gt: 60}})`

* it has two stages, first,`executionStages.inputStage.stage: 'IXSCAN' ` it scans only in ordered indexs. second, `executionStages.stage: 'FETCH' ` now it fetchs correct data from collection.

drop index:
`db.persons.dropIndex({'dob.age': 1})`

* **Compound indexing**
> use multiple indexes at the same time.\
`db.persons.createIndex({'dob.age':  1, gender: 1})`

HINT: MongoDB can only use a compound index if the query includes a condition on the first field of the index. In your index, the first field is `dob.age`. If you query with only `{'gender': 'female'}`, MongoDB cannot use the index efficiently because the query doesn't provide a value for `dob.age`

* **Unique index**
> We get a warning if we want to create the index or we already had it in colletion and we try to add a document with a value that already exist, we would get an error during the insert operation.\
`db.persons.createIndex({email: 1}, {unique: true})` 

* **Partial Indexing**
> Partial indexes only index the documents in a collection that meet a specified filter expression. By indexing a subset of the documents in a collection, partial indexes have lower storage requirements and reduced performance costs for index creation and maintenance.\
`db.persons.createIndex({'dob.age': 1}, {partialFilterExpression: {gender: 'female'}})`

> To retrieve all documents and ensure the query uses the partial index efficiently, you must include both the fields defined in the index and the `partialFilterExpression` in your query. Specifically, for the partial index created with the filter expression `{ gender: 'female' }` on the `dob.age` field, the query must include both `dob.age` and gender conditions. \
`db.persons.find({'dob.age': {$gt: 40}, gender: 'female'})`


* **Partial Indexing & Unique index**
> The partial index ensures that only documents where the email field exists are indexed. This is particularly useful when there are documents without an email field, and you don't want to enforce uniqueness on `null` or `undefined` values.\
`db.users.createIndex({email: 1}, {unique: true, partialFilterExpression: {email:{$exists: true}}})`

* **TTL indexing**
> by using `{expireAfterSeconds: 10}` you set a expire time for documents .\
> Note: If there is already a document in the collection and you set expireAfterSeconds, it won't work. To establish expireAfterSeconds, you must insert a new document. \
`db.sessions.createIndex({createdAt: 1}, {expireAfterSeconds: 10})`


* **Multikey indexing**
> multikey indexes are a special type of index that allows you to index fields that contain arrays. When a field in a document is an array, MongoDB creates index entries for each element in the array, allowing efficient querying on array values.

Indexing arrays:
  > Multikey indexes are created automatically when you create an index on a field that contains an array. For each document, MongoDB creates a separate index entry for every element in the array.\
  > For example, if you have a collection of documents representing products, and each product document contains a field tags that is an array of strings (e.g., ["electronics", "gadgets", "sale"]), MongoDB will index each string in the tags array separately.

* **Text indexing**
> This feature allows users to perform full-text searches on text fields in a collection efficiently. It supports text search queries, enabling users to search for specific words or phrases within text fields.

1. Tokenization

Once you create a text index, MongoDB starts by tokenizing the text fields. Tokenization involves breaking down strings into smaller pieces (tokens), typically words or phrases, that will be used for indexing and search purposes. For example, the sentence "MongoDB is a NoSQL database" would be split into tokens like ["MongoDB", "is", "a", "NoSQL", "database"].

2. Stemming

After tokenization, MongoDB applies stemming (a linguistic process). Stemming reduces words to their root form. For instance, "running" and "runner" may both be stemmed to "run". This allows the search engine to return relevant results regardless of the specific form of the word used in the document.

3. Normalization

MongoDB normalizes the tokens by converting them to lowercase. This allows for case-insensitive searches. For example, "MongoDB" and "mongodb" are considered the same for search purposes.

4. Stop Words Removal

Common words (like "is", "the", "and", etc.) are removed during the indexing process, as these words generally do not provide significant meaning for search results. These words are called stop words, and MongoDB maintains a list of them for removal.

5. Inverted Index Construction

MongoDB then creates an inverted index, which is a structure used to map terms to the documents in which they appear. Instead of storing the text itself, MongoDB stores each term (word or token) along with references to the documents containing that term.


create index:\
`db.users.createIndex({description: 'text'})`

find:\
`db.users.find({$text:{$search: 'new'}})`

sorting:\
> expression is used to retrieve the relevance score of each document based on the search query. This score reflects how well each document matches the search terms, allowing you to rank documents by relevance.

`db.users.find({$text:{$search: 'new'}}, {score: {$meta: 'textScore'}})`

excluding:\

It searches for documents in the users collection that contain the word "new" but exclude any documents that also contain the word "description".

`db.users.find({$text:{$search: 'new -description'}})`

* $default_language & $weight:\
> the weights option within a text index allows you to assign different importance levels (or weights) to the fields included in the index. This affects the relevance score when performing a text search, influencing how much a match in one field contributes to the ranking of the document compared to matches in another field.\
> Higher weights make matches in the corresponding field contribute more to the document’s relevance score.Lower weights make matches in the corresponding field contribute less to the document’s relevance score.If no weight is specified for a field, MongoDB assigns a default weight of 1.

before weight:\
```
db.users.find({$text:{$search: 'new'}}, {score: {$meta: 'textScore'}})

[
  {
    _id: ObjectId('66e81c0b03c25d21af2710bc'),
    name: 'New Amir',
    description: 'second not new description',
    score: 0.6666666666666666
  },
  {
    _id: ObjectId('66e2b7126f1e5e09442710bc'),
    name: 'Amir',
    hobbies: [ 'sport', 'reading book', 'programming' ],
    address: { street: 'main', alley: 'second' },
    description: 'this is a new description which i just wrote it',
    score: 0.625
  }
]
``` 
create index:\
`db.users.createIndex({ description: 'text', name: 'text'},{$default_language: 'English', $weight: {description: 9, name: 1}})`

after weight:\
```
db.users.find({$text:{$search: 'new'}}, {score: {$meta: 'textScore'}})

[
  {
    _id: ObjectId('66e81c0b03c25d21af2710bc'),
    name: 'New Amir',
    description: 'second not new description',
    score: 1.4166666666666665
  },
  {
    _id: ObjectId('66e2b7126f1e5e09442710bc'),
    name: 'Amir',
    hobbies: [ 'sport', 'reading book', 'programming' ],
    address: { street: 'main', alley: 'second' },
    description: 'this is a new description which i just wrote it',
    score: 0.625
  }
]
```

## Foreground indexing vs Background indexing

1. Foreground Index Creation

When you create an index in the foreground, MongoDB locks the collection while building the index. This means that all read and write operations are blocked for the duration of the index creation process, which can affect database performance and availability, especially in large collections.

*Behavior*: The collection is locked, preventing all operations (inserts, updates, queries, and deletes) until the index is fully built.

*Speed*: Foreground index creation is faster because it has exclusive access to the collection.

*Impact*: Suitable for smaller collections or during maintenance windows when downtime is acceptable.


2. Background Index Creation

When you create an index in the background, MongoDB builds the index without blocking read and write operations on the collection. This means that the database remains available, but the index creation process takes longer because it must handle ongoing operations in parallel.

*Behavior*: MongoDB allows read and write operations while the index is being built, so the collection remains available.

*Speed*: Background index creation is slower compared to foreground creation because MongoDB must manage concurrent operations while building the index.

*Impact*: Ideal for large collections and in production environments where uptime and availability are critical, as it minimizes the impact on ongoing operations.

  `db.collection.createIndex({ field: 1 }, { background: true })`


# Geopatial data
> MongoDB supports the GeoJSON object types.

To specify GeoJSON data, use an embedded document with:

    a field named type that specifies the GeoJSON object type, and

    a field named coordinates that specifies the object's coordinates.

`<field>: { type: <GeoJSON type> , coordinates: <coordinates> }`


If specifying latitude and longitude coordinates, list the longitude first, and then latitude.

    Valid longitude values are between -180 and 180, both inclusive.

    Valid latitude values are between -90 and 90, both inclusive.

Find near palces to a certain location:
```
First create index for that field:
db.place.createIndex({location: '2dsphere'})

Then write query:

db.place.find({location: {$near: {$geometry: {coordinates: [51.1115598, 35.6918046], $maxDistance: 450, $minDinstance: 10}}}})
```

To determine if a polygon contains a point:
```
define polygon:
const p1 = [51.1113311, 35.6910677]
const p2 = [51.1152458, 35.6898856]
const p3 = [51.1144719, 35.6867969]
const p4 = [51.1105277, 35.6892128]

use find method to find polygons inside places:
Note: To draw a polygon, start from a point and return to the starting point to complete the shape
db.place.find({location:{$geoWithin: {$geometry: {type: 'Polygon', coordinates: [[p1, p2, p3, p4, p1]]}}}})
```

To determine if a place is in a Polygon or not:
```
define polygon:
const p1 = [51.1113311, 35.6910677]
const p2 = [51.1152458, 35.6898856]
const p3 = [51.1144719, 35.6867969]
const p4 = [51.1105277, 35.6892128]


db.areas.insertOne({name: 'university', area: {type: 'Polygon', coordinates: [[p1, p2, p3, p4, p1]]}})

db.areas.createIndex({area: '2dsphere'})

db.areas.find({area: {$geoIntersects: {$geometry: {type: 'Point', coordinates: [51.112576302503285, 35.68933136835019]}}}})
```

To determine near place of a point:

This MongoDB query searches the place collection for documents where the location field contains coordinates that fall within a circular area, centered at [51.11096184525407, 35.690118043813776] (longitude, latitude). The search area is a 1-kilometer radius on the Earth's surface, calculated using spherical geometry. The $geoWithin operator ensures that only locations within this circular region are returned, and the radius is specified in radians (1 / 6378.1), where 6378.1 is the approximate radius of the Earth in kilometers.

`db.place.find({location: {$geoWithin: {$centerSphere: [[51.11096184525407, 35.690118043813776], 1 / 6378.1]}}})`

# Embedded DocumentsBN
> Nested Document

* Up to 100 level of Nesting allows on MongoDb.
* Max. 16mb document.

# Arrays
> YOu can hold Array of embedded documents. Arrays can hold ANY data.

# DataTypes
1. **text**
2. **boolean**
3. **Number**
    1. **integer(32 bit)**
    2. **Long(64 bit)**
    3. **Double(64 bits)**
    4. **High precision Double(128 bits)**
4. **Timestamp** 
> typically generate by mongodb. it has unique value for each document.
5. **ObjectId**
> generate by mongodb. it has unique value for each document.
6. **ISOdate("2018-08-08")**
7. **embedded document**
> {"a": {}}
8. **Array**
> {"a": []}

# Relations

**Options:**
1. Nested Document
```
{
    "username": 'max',
    "age": 29,
    "address":{
        "street": "second street',
        "city": "new york" 
    }
}
```

2. References
```
Customer collection
{
    "username": 'max',
    "favbooks": ['id1']
}

Book collection
{
    "_id1": 'id1',
    name: 'lord of rings'
}
```
**Types:**
1. One-to-One:
> it is better to use Nested Document.

    1. reference
    ```
    use shop
    db.product.insertOne({name: "book", price: 12.99, detail: "detail_id_1"}
    db.detail.insertOne({_id: 'detail_id_1', description: "second handed"})
    var bkid = db.product.findOne().detail
    db.detail.find({_id: bkid})
    ```

    2. Nested document
    ```
    db.product.insertOne({name: "pc", price: 1245.5, detail: ['core 17', 'rtx 3050']})
    db.product.findOne({name: 'pc'}).detail
    ```
2. One-To-Many:
    1. reference
    ```
    db.questions.insertOne({creator: 'Amir',
    questions: 'what is this?',
    answers: [ 'q1a1', 'q1a2']})

    db.answers.insertMany([{_id: 'q1a1', text: 'book' }, {_id: 'q1a2', text: 'pc'}])
    var qu =  db.questions.findOne().answers[1]
    db.answers.findOne({_id: qu})
    ```
    2. Nested Document
    ```
    db.products.insertOne({title: 'book'})
    db.customers.insertOne({name: 'amir', age: 21, orders: { title: 'book', quantity: 3 }})
    ```

    3. Linked collections
    ```
    db.customers.insertOne({name: 'amir', age: 21})
    db.products.insertOne({title: 'book'})
    db.orders.insertOne({customer_id: ObjectId('66bdd7a00d4c678380228fb5'), product_id: ObjectId('66bdd7a00d4c678380228fb5')})
    ```

# Methods

1. aggregate and $lookup
> merge two collections that related in one document in one step.

```
collectionName.aggregate([
    {$lookup: {
        from: "fieldName", => from which other collection do you want to relate document.
        localField: "fieldName", => where can the references to other collection can be founded. 
        foreignField: "fieldName", => which field are you relating to in target collection
        as: "fieldName" => merge under this field
        }
    }
])
```
2.createCollection
> it is a lazy method that create a empty collection and set validatiors
> it has a validationAction field that the default is "error'
```
db.createCollection('posts', {
    validator:{
    $jsonSchema:{
        bsonType: "object", 
        required: ['title', 'text', 'user', 'comments'], 
    properties:{
        title:{
            bsonType: 'string',
            description: 'must be a string and is required'
        },
        text:{
            bsonType: 'string',
            description: 'must be a string and is required' 
        },
        user:{
            bsonType: 'objectId',
            description: 'must be a objectId and is required' 
        },
        comments:{
            bsonType: 'array',
            description: 'must be a array and is required',
            items:{
                bsonType: 'object',
                properties:{
                    text:{
                        bsonType: 'string',
                        description: 'must be a string and is required'
                    },
                    author:{
                        bsonType: 'objectId',
                        description: 'must be a objectId and is required'
                    }
                }
            }
        }
    }
}}})
```
3. runCommand
> add new setting validation as db admin
> it has a validationAction field that the default is "warn". 
```
db.runCommand({
  collMod: 'posts',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  },
  validationAction: 'warn'
})
```

# Commands

> show databases\
` show dbs `

> switch between dbs. even if there is not such a db\
`use databasename`

> query a collection\
`db.collectionName.Query()`

> manually insert ID\
`db.collectionName.insertOne({_id:"dafdafdfdb"})`

> replace data\
`db.collectionName.replaceOne(filter, data, options)`


# Aggregation framework

> The aggregation framework in MongoDB is a powerful tool used for data processing and transformation within the database. It allows you to process data and perform complex queries on collections, such as filtering, grouping, and transforming data, as well as aggregating results across multiple documents. This framework is highly efficient for handling large datasets and is a key feature of MongoDB.

>The core component of the aggregation framework is the pipeline, which consists of multiple stages that transform the data. Each stage processes the documents and passes the results to the next stage in the pipeline, allowing you to chain operations together. Common aggregation stages include:

  1. $match: Filters documents to pass only those that meet specified conditions.\
    `db.person.aggregate({$match: {gender: 'female'}})`

  2. $group: Groups documents by a specified field and applies aggregation expressions like sum, average, min, max, etc.\
    `db.person.aggregate([{$match: {gender: 'female'}}, {$group: {_id: {city:'$location.city'}, total: {$sum: 1}}}])`

    adding together arrays.\
    `db.friends.aggregate({$group: {_id: '$age', allHobbies: {$push: '$hobbies'}}})`

  3. $project: Reshapes documents by including, excluding, or adding new fields.\
    `db.person.aggregate({$project: {_id: 0 , 'dob.age': 1}})`

    * filtering
    `db.friends.aggregate({$project: {scores: {$filter: {input: '$examScores', as: 'sc', cond: {$gt: ['$$sc.score', 50]}}}}})`

    ```
        db.person.aggregate([
      {
        $project: {
          _id: 0,
          fullName: {
            $concat: [
              { $toUpper: { $substrCP: ['$name.first', 0, 1] } }, // First letter of first name in uppercase
              {
                $substrCP: [
                  '$name.first', // Rest of the first name starting from index 1
                  1,
                  { $subtract: [ { $strLenCP: '$name.first' }, 1 ] }
                ]
              },
              ' ', // Space between first and last names
              { $toUpper: { $substrCP: ['$name.last', 0, 1] } }, // First letter of last name in uppercase
              {
                $substrCP: [
                  '$name.last', // Rest of the last name starting from index 1
                  1,
                  { $subtract: [ { $strLenCP: '$name.last' }, 1 ] }
                ]
              }
            ]
          }
        }
      }
    ]);
    ```

    to use Geopatial data and convert data types
    ```
    db.person.aggregate([{
      $project:{
        _id: 0, 
        email: 1,
        birthDate: {$toDate: '$dob.date'},
        age: '$dob.age', 
        location:{
            type: 'Point', 
            coordinates: [{
              $convert:{
                input:'$location.coordinates.longitude', 
                to: 'double', 
                onNull: 0, 
                onError: 0
              }
            },
              {
                $convert:{
                input:'$location.coordinates.latitude', 
                to: 'double', 
                onNull: 0, 
                onError: 0
                }
              } 
            ]
        }
      }},
      {$group: {_id: {birthYear: {$isoWeekYear: '$birthDate'}}, peopleCounter: {$sum: 1}}},
      {$sort: {peopleCounter: -1}}
    ])
    ```

  4. $sort: Sorts documents by specified fields.\
    `db.person.aggregate({$project: {_id: 0 , 'dob.age': 1}}, {$sort: {'dob.age': -1}})`

  5. $limit: Limits the number of documents passed along the pipeline.
  6. $skip: Skips a specified number of documents.
  7. $lookup: Performs joins with another collection.
  8. $unwind: Deconstructs arrays to output a document for each element in the array.\
    `db.friends.aggregate({$unwind: '$hobbies'})`


      you can combine $unwind and $group together:
      > it returns a single level array that might be filled be cuplicate values.

      `db.friends.aggregate([{$unwind: '$hobbies'}, {$group: {_id: '$age', allHobbies: {$push: '$hobbies'}}}])`

      > to avoid duplicate data use $addToSet instead of $push 
    
    9. $bucker: Categorizes incoming documents into groups, called buckets, based on a specified expression and bucket boundaries and outputs a document per each bucket.
    `db.persons.aggregate([{$bucket: {groupBy: `$dob.age`, boundaries: [40, 50, 60, 70], default: 'other', output: {count: {$sum: 1 }, average_age: {$avg$avg: '$dob.age'}}}}])`

    10. $out: Takes the documents returned by the aggregation pipeline and writes them to a specified collection. The `$out` stage must be the last stage in the pipeline.

  a query which retrieve highest score of a student.
  ```
  db.friends.aggregate([{$unwind: `$examScores`}, {$project: {id: 1, name: 1, score: '$examScores.score'}}, {$sort: {score: -1}}, {$group: {_id: '$_id', name: {$first: '$name'}, maxScore: {$max: '$score'}}}, {$sort: {maxScore: -1}}])
  ```

# Security

> Authenticate: identifies valid users of the database.\
> Authorization: identifies what these users actually can do in this databse.\
> User: A user is someone who uses a MongoDB server, not just someone who uses your app on a MongoDB server.


**Built-in Rules**

1. DataBase User:
    * read
    * readWrite

2. Database Admin:
    * dbAdmin
    * userAdmin
    * dbOwner

3. All Database Roles:
    * readAnyDatabase
    * readWriteAnyDatabase
    * userAdminAnyDatabase
    * dbAdminAnyDatabase

4. Cluster Admin:

multiple mongodb server working together are a cluster.

    * clusterManager
    * clusterMonitor
    * hostManager
    * clusterAdmin

5. Backup/ Restore
    * backup
    * restore

6. Superuser
    * dbOwner (admin)
    * userAdmin (admin)
    * userAdminAnyDatabase
    * root


Create and Authenticate users:vc

*create*
```
use admin
db.createUser({user: 'username', pwd: 'password', roles: ['userAdminAnyDatabase',...]})
```

*authenticate*
```
use admin
db.auth('username', 'password')
```

*get user*
``
db.getUser('username')
``

*update a user*
``
db.updateUser('username', {roles: ['readWrite', {role: 'readWrite', db: 'blog'}]})
``


# Capped collection
> Capped collections are fixed-size collections that support high-throughput operations that insert and retrieve documents based on insertion order.


*Fixed Size*: The size parameter specifies that the capped collection can only store up to 1000 bytes of data. Once this limit is reached, older documents will be overwritten by newer ones.\
*Max Documents*: The max parameter limits the collection to 3 documents. When a fourth document is inserted, the oldest document will be removed automatically, making capped collections behave like circular buffers.\
*Capped*: Setting capped: true ensures that the collection maintains the defined maximum size and document count. Capped collections are highly performant as they do not support document deletion or resizing.\

``
db.createCollection('capped', {max: 3, size: 1000, capped: true})
``

# Replica Set

A MongoDB replica set consists of several key components and concepts, each playing an important role in providing data redundancy, high availability, and fault tolerance. Let’s break it down part by part:
1. Primary Node

    * The primary is the main node where all write operations occur. It handles client requests for writes (insert, update, delete) and ensures data consistency across the replica set.
    * The primary replicates data changes to the secondary nodes.
    * If the primary fails, one of the secondary nodes is automatically elected as the new primary, ensuring continuous availability.

2. Secondary Nodes

    * Secondary nodes replicate data from the primary node to maintain an identical dataset.
    * These nodes can optionally handle read operations if the client is configured for read preference, reducing load on the primary and increasing read scalability.

# Sharding

Sharding is a method for distributing data across multiple machines.\
Horizontal Scaling: involves dividing the system dataset and load over multiple servers, adding additional servers to increase capacity as required.
While the overall speed or capacity of a single machine may not be high, each machine handles a subset of the overall workload, potentially providing better efficiency than a single high-speed high-capacity server.
Expanding the capacity of the deployment only requires adding additional servers as needed, which can be a lower overall cost than high-end hardware for a single machine.\
MongoDB uses the shard key to distribute the collection's documents across shards. The shard key consists of a field or multiple fields in the documents.

# Transaction
MongoDB transactions provide the means to execute multiple operations as a single logical unit of work. This ensures that either all operations within the transaction are completed successfully, or none of them take effect, maintaining data consistency and integrity.

1. *Transaction Begin*: A transaction starts with a `startTransaction` command. All subsequent operations within this session are considered part of the transaction until explicitly committed or aborted.
2. *Performing Operations*: During the transaction, you can perform various CRUD operations as well as other administrative actions.
3. *Transaction Commit*: To commit the transaction, you call the `commitTransaction` command. MongoDB ensures that all changes made within the transaction are persisted to the database and are visible to other transactions only after the commit is successful.
4. *Transaction Abort*: If an issue arises and you need to undo the changes made within a transaction, you can call the `abortTransaction` command. This discards all changes and releases any locks held by the transaction.