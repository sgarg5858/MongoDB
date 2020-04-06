# MongoDB #noSQL

************************************************************************************************************
INSERT DATA

//Create Database event the database doesn't exists it will switch to db hospital and db won't be created until we add data to db hospital

1. use hospital

//Here patients is collection and we will insert 3 documents here and make sure every id is different and if we don't give _id it mongodb will create one and will be unique

2.db.patients.insertMany(Copy patientsData.json here)

*************************************************************************************************************
UPDATE DATA

1.Update Age of One Patient using name of patient.

  db.patients.updateOne({"firstName": "Mike"}, {$set: {"age":22}})

2.Update Add new key admitted in all patients.

   db.patients.updateMany({}, {$set, {"admitted": true}})

**************************************************************************************************************

FIND DATA

1. findOne() 

  //It returns only one document topmost
    db.patients.findOne(filter,options).prett()
    
2. find()
  //It returns Cursor Object which has a lot of metadata using cursor we can iterate over data as data can be very large.
  
  db.patients.find().pretty()
  //we can use .toArray() or .forEach()  or many more operations 
  
3. find patients who are older than 30

  db.patients.find({age:{$gt:30}}).pretty() 

4.Find Patients who have history of cold

 db.patients.find({"history.disease": "cold"}).pretty()
 
 *****************************************************************************************************************
 
 Delete Patients who have col
 
 1.db.deleteMany({"history.disease": "cold"})
 
 
 *******************************************************************************************************************
 
DELETING DATABASE/COLLECTIONS
 
 1. use databaseName
 2. db.dropDatabase()
 
 Drop Collection
 
 1. db.mycollection.drop()


*************************************************************************************************************************
RELATIONS:
*************************************************************************************************************************
1.One To One:

In one to one it is preffered to use Embedded Documents Over References as there will be no duplicates

For E.g Each Patient will Have its own Disease Summary

i.  > db.patients.insertOne({name: "Mike", age: 22,diseaseSummary: {diseases: ["cold", "cough"]} })

*************************************************************************************************************************
2. One To Many:

In one to many we can use both Embedded or References if using Embedded Documents it is ensured that document size will always be less 
than 16 mb then it is suggested to use Embedded as it will save one request.
Otherwise Go For Reference!

Embedded:

Scenario: Comments one On Instagram Post!

use instagram

switched to db instagram

db.post.insertOne({user:"Disha Patani",comments:["Pretty","Hey There","Looking Great"]})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e81e3ed803b836b9740c109")
}

 db.post.find().pretty()
{
        "_id" : ObjectId("5e81e3ed803b836b9740c109"),
        "user" : "Disha Patani",
        "comments" : [
                "Pretty",
                "Hey There",
                "Looking Great"
        ]
}

Reference:

use cityData

switched to db cityData

db.cities.insertOne({cityName:"New York", Country:"USA"})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82ce782d55ad169b0290a4")
}

Use _id of City as CityId in Citizens

 db.citizens.insertMany([{name: "Sanjay",cityId: ObjectId("5e82ce782d55ad169b0290a4")},{ name: "Prim", cityId: ObjectId("5e82ce782d55ad169b0290a4")}])


To fetch citizens Use Find query with filter
db.citizens.find({cityId: ObjectId("5e82ce782d55ad169b0290a4")}).pretty()


*************************************************************************************************************

Many to Many:


Basic SQL approach would be to use 3 tables

1. db.dropDatabase()
{ "dropped" : "shop", "ok" : 1 }

2. use shop
switched to db shop

3. db.products.insertOne({price: 120, title:"Book"})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82d41a2d55ad169b0290a7")
}

4. db.customers.insertOne({name:"Sanjay", age:21})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82d4352d55ad169b0290a8")
}

5. db.orders.insertOne({custId: ObjectId("5e82d4352d55ad169b0290a8"), prodId:ObjectId("5e82d41a2d55ad169b0290a7")})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82d46f2d55ad169b0290a9")
}


Using MongoDB style USING 2 collections reference approach

1. db.products.insertOne({price: 120, title:"Book"})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82d41a2d55ad169b0290a7")
}


2. db.customers.insertOne({name:"Sanjay", age:21, orders:[{productId:ObjectId("5e82d41a2d55ad169b0290a7", quantity:2}] })
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e82d4352d55ad169b0290a8")
}


3. db.customers.find().pretty()
{
        "_id" : ObjectId("5e82d4352d55ad169b0290a8"),
        "name" : "Sanjay",
        "age" : 21,
        "orders" : [
                {
                        "productId" : ObjectId("5e82d41a2d55ad169b0290a7"),
                        "quantity" : 2
                }
        ]
}

If we use emdedded documents here then there is going to be a lot of duplication and if we have to update then we have to update in
all customers who bought that product

Embedded Way:

db.customers.insertOne({ name:"sanjay",orders: [{title: "Iphone", price:50000, quantity:2}]})

If update in the products doesn't affect already orders as customer will not increased amount after placing order,
we can also use embedded documents here.

When to Use Reference Aproach!

Suppose one book is written by multiple authors and one author may write multiple books

Many to Many:

db.books.insertOne({name:"Apache Kafka",authors:[{authorId:1,authorName:"ABC",age:21},{authorId:2,authorName:"BCD",age:22}]})


 db.authors.insertMany([{
...                         "authorId" : 1,
...                         "authorName" : "ABC",
...                         "age" : 21
...                 },
...                 {
...                         "authorId" : 2,
...                         "authorName" : "BCD",
...                         "age" : 22
...                 }
...         ])

If Author gets older then we have to do changes in both collections so it prefferd to use References here.

db.books.insertOne({name:"Apache Kafka",authorIds:[1,2]})

Use References!

*********************************************************************************************************************
LOOK-UP

Let's see how we can merge related Documents IN One Go instead of 2 steps

we will use look-up

we have one book in collection books and it has 2 authorIDs and in authors we have to authors

db.books.aggregate([ {$lookup: {from:"authors", localField:"authorIds", foreignField: "authorId", as:"creators"}} ]).pretty()

1.from => target collection
2.localField => what should be used to merge
3. foriegnField => against what localField should be checked.
4.creators:any name here merged data will come

output:

{
        "_id" : ObjectId("5e82e95b2d55ad169b0290ae"),
        "name" : "Apache Kafka",
        "authorIds" : [
                1,
                2
        ],
        "creators" : [
                {
                        "_id" : ObjectId("5e82e7c02d55ad169b0290ac"),
                        "authorId" : 1,
                        "authorName" : "ABC",
                        "age" : 21
                },
                {
                        "_id" : ObjectId("5e82e7c02d55ad169b0290ad"),
                        "authorId" : 2,
                        "authorName" : "BCD",
                        "age" : 22
                }
        ]
}
******************************************************************************************************************************

Data Validation Schema 

db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['text', 'creator', 'comments'],
      properties: {
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'string',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['username', 'comment'],
            properties: {
              username: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              string: {
                bsonType: 'string',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  }
});

It will createcollection with the given schema  

db.posts.insertOne({text: "Hello IG", creator:"Sanjay98", comments: [ {username:"yumsunyum",comment:"Welcome Babe"} ] })  

It will be inserted

db.posts.insertOne({text: "Hello IG", creator:123, comments: [ {username:"yumsunyum",comment:"Welcome Babe"} ] })  

It will Give ERROR with Message Document Failed Validation as creator is of number but required is string!


**********************************************************************************************************************
Ordered Insertions:

Suppose we are using custom Id and we are trying to insertMany and if any id exists before then it will stop the operation there
and won't try the next inserts 
If you want to try all inserts no matter what

db.mycollection.insertMany([],{ordered:false}) 

By Default it is True!



**************************************************************************************************************************
WriteConcern:

Check Journal For this: For Higher Security: Acknowledgement after wriiten to memory and journal 
Journal is backup for storage engine incase server fails to recover 
Journal contains operations that haven't done yet on disk but are done in memory By default it is false

db.mycollection.insertOne([],{writeConcern: {w: 1, j: true} }) 


***************************************************************************************************************************
Importing Data:

Stop Shell
mongoimport tvshows.json -d moviesData -c movies --jsonArray --drop

-d => DataBase
-c => Collection
-jsonArray => By Default it looks for only One Document So we have to specify For Multiple Documents
--drop if moviesData.movies exists before then it tells to drop and create new one.

****************************************************************************************************************************

Let's Dive Into Read Operations:

Operators,Filters:

1. Comparison Filters:
https://docs.mongodb.com/manual/reference/operator/query-comparison/

i) db.movies.find({runtime:{$eq:30}}).pretty()   runtime equal to 30

ii) db.movies.find({runtime:{$gt:30}}).pretty()  runtime >30

iii) db.movies.find({runtime:{$gte:30}}).pretty()  runtime >=30

iv) db.movies.find({runtime:{$lt:30}}).pretty()  runtime <30

v) db.movies.find({runtime:{$lte:30}}).pretty()  runtime <=30

vi) db.movies.find({runtime:{$ne:30}}).pretty()   runtime !=30

vii) db.movies.find({runtime:{$in: [30,42] }}).pretty()   runtime ==30 || runtime==42 matches any values in array to runtime field value

viii)  db.movies.find({runtime:{$nin: [30,42] }}).pretty()   runtime !=30 && runtime!=42   Doesn't match any values in array to runtime field value
When Embedded Documents:

db.movies.find({"rating.average" : {$eq:8}}).pretty()   rating equal to 30

When Arrays:
 
{genres:["Drama","Comedy"]}

when searching for that particular genres in array Other elements doesn't matter

db.movies.find({genres: "Drama"} ).pretty()   

When Exact Match:

db.movies.find({genres: ["Drama"] } ).pretty()  Make Array in Value MongoDB will look for Exact Array in Value


***************************************************************************************************************************

Logical Operators:

1. OR operator:

db.movies.find({$or: [{"rating.average": {$lt: 5} },{"rating.average": {$gt: 9.3 } } ]}).count()
  
For movies rating less than 5 and greater than 9.3

2.NOR Operator

db.movies.find({$nor: [{"rating.average": {$lt: 5} },{"rating.average": {$gt: 9.3 } } ]}).count()

For movies not less than 5 and not greater than 9.3

3. AND Operator:

db.movies.find({$and: [{"rating.average": {$gt:9} },{genres:"Drama"} ]}).pretty()

For movies having rating >9 and genre="Drama"

4. NOT Operator:

db.movies.find({runtime: {$not: {$eq: 60} } }).count()

Runtime not equal to 60

db.movies.find({runtime: {$ne: 60}  }).count()   can also be done using ne not equal


*****************************************************************************************************************

Element Operators:

1. $exist   2. $type

1. 

db.userData.insertMany([ {name: "Sanjay",phone:1234567890,age:21},{name:"sanyam",phone:"1234567890"}])


db.userData.find().pretty()
{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011104"),
        "name" : "Sanjay",
        "phone" : 1234567890,
        "age" : 21
}
{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011105"),
        "name" : "sanyam",
        "phone" : "1234567890"
}


Where Age Field Exists 2nd Doc doesn't have age field So it gets filter out


db.userData.find({age: {$exists: true}}).pretty()


{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011104"),
        "name" : "Sanjay",
        "phone" : 1234567890,
        "age" : 21
}


  For checking age !=null and name=Sanjay
 db.userData.find({$and: [{age: {$exists: true, $ne: null }} ,{name:"Sanjay"} ]}).pretty()
 
 Will Give same as Above
 
 2. $type
 
  db.userData.find({phone: {$type: ["string","number"]} } ).pretty()
  
  db.userData.find({phone: {$type: "string"} } ).pretty()
  
  
  **********************************************************************************************************************
  Evaluating Operators:
  
  $regex
  
  For Finding text Snippet Most Efficent way of doing this is to use Text Indexes to which we come later
  
  If text is not to large then we can use $regex
  
  db.movies.find({summary: {$regex: /musical/}}).count()
  
  It will look for musical word in summaries of all docs It's nice but not th efiicient approach


$expr

When we want to search documents based on some condition based on fields of document and fields will be of one document

db.pages.insertMany([ {total:4,want:8},{total:34,want:23},{total:6,want:3},{total:9,want:10}])

It will give all docs where total > want:

db.pages.find({$expr: {$gt: ["$total","$want"] } }).pretty()


More Complex:

If total>30 then total must be greater than want by 14.

 db.pages.find( {$expr: 
                  {$gt: 
                    [{$cond: 
                    {if: {$gte: ["$total",30]}, 
                    then: {$subtract: ["$total",14]}, 
                    else: "$total" }}
                    ,"$want"] 
                    }}).pretty()
 
 
 
 Array Queries:
 
 
 1. We have to find Users who have Hobby of Cooking
 {
        "_id" : 3,
        "hobbies" : [
                {
                        "name" : "Cooking",
                        "freq" : 4
                },
                {
                        "name" : "Reading",
                        "freq" : 7
                },
                {
                        "name" : "Hiking",
                        "freq" : 5
                }
        ]
}
 
 db.nycollection.find({"hobbies.name":"Cooking"}).prett();
 
 
 2. $size
 
 We have to find user who have exactly 3 hobbies
 
 db.mycollection.find(hobbies: {$size: 3}}.pretty()
 
 This will not work: as $size needs a number
 
 db.mycollection.find(hobbies: {$size: {$gt:2}}}.pretty()
 
 
3. $elemMatch

We have to find Users who Cooking with freq >=4

One Way is to use AND Operator

 
 db.mycollection.find({$and: [ {"hobbies.name:"Cooking"},{freq:{$gte:4}}} ] })
 
 But this will not work!
 
 As it can satisfy condition accross mutliple documents in Array
 but we need to check for cooking with freq>=4 
 
  db.mycollection.find({hobbies:{$elemMatch:{name:"Cooking",freq:{$gt:4}} }}).pretty()


Suppose we want all movie records where genre: action and drama

db.movies.find({genre:{$all:["thriller","action"]}}).pretty()

If we don't use $all then order we specify if use $all then order won't matter


***************************************************************************************************************************

Working With Cursor:

Cursor returns data in batches

When we run 
db.mycollection.find()  

query has ran on the server side and information is loaded into memory and is ready to delievered to us.

$it is available only in shell

In Drivers we have functions like .next()


const dataCursor = db.tvshows.find()
dataCursor.next()   //next will give you next element!


ForEach  :

dataCursor.forEach(doc => {printjson(doc)}) 

Sorting:

db.tvshows.find().sort({"rating.average":-1}) -1 for descending order and 1 for ascending order

and we can also use multiple criteria for sorting but order matters!

db.tvshows.find().sort({"rating.average":1,runtime:-1})

Skipping Elements:

db.tvshows.find().sort({"rating.average":1,runtime:-1}).skip(10).pretty()  

Limit: Related To Pagination

limit allows you to limit the no of retrieve elements

db.tvshows.find().sort({"rating.average":1,runtime:-1}).skip(10).limit(2).pretty()  

first we will find all then sort according to needs then we want to skip first 10 based on sorting and posing a limit of 2 per query


******************************************************************************************************************

PROJECTION:

db.tvshows.find({},{name:1, genres:1, runtime:1, rating:1, "schedule.time":1, _id:0}).pretty()  

It also works on Embedded Documents for that check schedule  and if we don't want _id we have to specify it

Projection In Arrays:


******************************************************************************************************************************

UPDATE OPERATIONS:

******************************************************************************************************************************
$set 

It just simply add/edits fields you specify by default it doesn't remove fields

1. db.userData.updateMany({},{$set:{hobbies:[ {name:"cooking",freq:4},{name:"reading",freq:7}]}}) 

If field doesn't exist it will create otherwise it will over ride the value we will see how to add new values to existing values

Updating multiple fields

2. db.userData.updateMany({},{$set:{age:20,phone:9041234556}})   


********************************************************************************************************
$inc

{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011104"),
        "name" : "Sanjay",
        "phone" : 9041234556,
        "age" : 20,
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4
                },
                {
                        "name" : "reading",
                        "freq" : 7
                }
        ]
}
{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011105"),
        "name" : "sanyam",
        "phone" : 9041234556,
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4
                },
                {
                        "name" : "reading",
                        "freq" : 7
                }
        ],
        "age" : 20
}

Suppose Sanyam got older by 1 year and we have to update his Age!

db.userData.updateMany({name:"sanyam"},{$inc:{age:1}})   

We can use multiple Operators:

db.userData.updateMany({name:"sanyam"},{$inc:{age:1},$set:{college:"UIET"}})   

For decrementing values use negative values!

Working On the SameField is not allowed..

$min

db.userData.updateMany({name:"sanyam"},{$min:{age:21}})

It will work if new value is less than old value

$max

db.userData.updateMany({name:"sanyam"},{$max:{age:25}})

It will modidy age if the new value is greater than old value

$mul multiply

db.userData.updateMany({name:"sanyam"},{$mul:{age:1.5}}) 

******************************************************************************************************************
Getting Rid Of Fields:

Suppose for sanjay we want to drop phone field:

 db.userData.updateMany({name:"Sanjay"},{$unset:{phone:" "}})  
 
 *****************************************************************************************************************
 
 Renaming Fields:
 
db.userData.updateMany({},{$rename:{age:"currAge"}}) 

**********************************************************************************************************************

Upsert:

db.userData.updateOne({name:"Prim"},{$set:{age:22,Hobbies:[{name:"Reading",freq:1}]}},{upsert:true})  

If it find document then it will override otherwise create new one


**********************************************************************************************************************
UPDATING MATCHED ARRAY ELEMENTS:

Adding new field to existing field:

Suppose we want to add new field to users who like cooking Like perDay freq say =2

db.userData.updateMany({hobbies :{$elemMatch: {name:"cooking", freq: {$gte:4}} } },{$set:{ "hobbies.$.perDayFreq":2}}) 

What this will do is find documents having hobby of cooking with freq>=4 and if we want to update the field using what we found data

here we are reffering to Cooking in hobbies then use hobbies.$.FieldName

This $ sign hobbies.$ is helpful we want to update specific element in array which you used for searching

We can also update other elements inside $set.

{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011105"),
        "name" : "sanyam",
        "phone" : 9041234556,
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4,
                        "perDayFreq" : 2  //check this 
                },
                {
                        "name" : "reading",
                        "freq" : 7
                }
        ],
        "college" : "UIET",
        "currAge" : 31.5,
        "year" : NaN
}

**************************************************************************************************************************
UPDATE ALL ARRAY ELEMENTS:

Suppose we search for all embedded documents in  hobbies who have frequency >=4 and we want to update all who matched

'But if we use above method it will only updated which embedded document matched first for each person and if there are multiple

matching documents in array it won't update them all

db.userData.updateMany({"hobbies.freq":{$gte:4}},{$set: {"hobbies.$.goodFreq": true} }) 

It will update First Matching Document



{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011105"),
        "name" : "sanyam",
        "phone" : 9041234556,
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4,
                        "perDayFreq" : 2,
                        "goodFreq" : true
                },
                {
                        "name" : "reading",
                        "freq" : 7
                }
        ],
        "college" : "UIET",
        "currAge" : 31.5,
        "year" : NaN
}

Check this won't update reading as it will only update first matching!

$ sign simply refers to first match!

$[] simply tells update all elements in Array no matter matched or not!

db.userData.updateMany({"hobbies.freq":{$gte:4}},{$set: {"hobbies.$[].daily": false} })  

{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011104"),
        "name" : "Sanjay",
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4,
                        "perDayFreq" : 2,
                        "goodFreq" : false,
                        "daily" : false
                },
                {
                        "name" : "reading",
                        "freq" : 7,
                        "goodFreq" : false,
                        "daily" : false
                }
        ],
        "currAge" : 20,
        "year" : NaN
}

*******************************************************************************************************************
UPDATING SELECTED FIELDS IN ARRAY!

db.userData.updateMany({"hobbies.freq":{$gte:5}},{$set: {"hobbies.$[el].enjoys":true}},{arrayFilters: [ {"el.freq": {$gt:5}} ] })  

el filters embedded document in array for applying changes


{
        "_id" : ObjectId("5e876b784105a1424b7bc60c"),
        "name" : "Prim",
        "age" : 21,
        "hobbies" : [
                {
                        "name" : "Reading",
                        "freq" : 1,
                        "daily" : false
                },
                {
                        "name" : "Reading",
                        "freq" : 1,
                        "daily" : false
                },
                {
                        "name" : "Cricket",
                        "freq" : 8,
                        "daily" : false,
                        "enjoys" : true
                }
        ]
}
**********************************************************************************************************************

Adding New Hobbies to existing Hobbies Array:

$push

db.userData.updateOne({name:"Sanjay"},{$push: {hobbies:{name:"Running",freq:7}}})   

{
        "_id" : ObjectId("5e847eecb5eb2cf2a5011104"),
        "name" : "Sanjay",
        "hobbies" : [
                {
                        "name" : "cooking",
                        "freq" : 4,
                        "perDayFreq" : 2,
                        "goodFreq" : false,
                        "daily" : false
                },
                {
                        "name" : "reading",
                        "freq" : 7,
                        "goodFreq" : false,
                        "daily" : false,
                        "enjoys" : true
                },
                {
                        "name" : "Running",
                        "freq" : 7
                }
        ],
        "currAge" : 20,
        "year" : NaN
}

db.userData.updateOne({name:"Sanjay"},{$push: {hobbies: {$each: [{name:"Coding",freq:7},{name:"Dancing",freq:4}],$sort:{freq:-1}}}})

For Adding multiples documents to Array and we can also sort  all the values !

***********************************************************************************************************************************

Deleting Values From Array!

$pull

db.userData.updateOne({name:"Sanjay"},{$pull: {hobbies:{name:"Running"}}})  

Sometimes you just want to remove last element from array

$pop

db.userData.updateOne({name:"Prim"},{$pop: {hobbies:1}})   1 for last -1 for first

**********************************************************************************************************************************
$addToSet

We can add values to Array using $addToSet instead of $push so what is difference

if you try to add same document push will allow it but $addToSet will not allow duplicate Values.

**********************************************************************************************************************************
INDEXES

https://docs.mongodb.com/manual/indexes/

Indexes support the efficient execution of queries in MongoDB. Without indexes, MongoDB must perform a collection scan, i.e. 

scan every document in a collection, to select those documents that match the query statement. If an appropriate index exists for a 

query, MongoDB can use the index to limit the number of documents it must inspect.

Indexes are special data structures [1] that store a small portion of the collection’s data set in an easy to traverse form. 

The index stores the value of a specific field or set of fields, ordered by the value of the field. 

The ordering of the index entries supports efficient equality matches and range-based query operations.

In addition, MongoDB can return sorted results by using the ordering in the index.

**********************************************************************************************************************************
Why Use Indexes?

Using Indexes can speed up find,delete and update queries!

By Default mongodb has no index!

Suppose you have to find a document with name:"Sanjay" then mongodb will go through entire collection which may contain thousands of

documents and it may take a while!

In indexes values are sorted by key and indexes contains references to full documents!

Don't Use too Many Indexes because Indexes speeds up find() queries but indexes doesn't come for free!

Because we have to update indexes for every inserts!

**********************************************************************************************************************************

Adding Single Field Indexes!

 db.contacts.explain("executionStats").find({"dob.age":{$gt:60}}).count() 
 
 We can use this command to check how mongoDB executed the query and how many documents it check and time it took.
 
 Let's add index!
 
 db.contacts.explain("executionStats").find({"dob.age":{$gt:60}}).count() 
 
 Without Index it took 5 milliseconds
 
 db.contacts.createIndex({"dob.age":1})    // 1 means ascending order -1 means descending order!
 
  
 Without Index it took 1 milliseconds
 
 
 Restrictions:
 
 when query returns large part of documents then using index can be slower as it will first go through indexes and then it will through
 
 a large number of documents.
 
 So if you have queries which returns majorly or all documents then index might not help you it you will do full scan and all documents

 will already be in memory indexing will just add another step and will lead to extra time.
 
 Indexes are helpful when you narrow down or need small number of documents.
 
 *********************************************************************************************************************
 
 Compund Indexes:     https://docs.mongodb.com/manual/core/index-compound/    limit of 32 fields
 
 The order of fields listed in a compound index has significance. For instance, 
 
 if a compound index consists of { userid: 1, score: -1 }, 
  
 the index sorts first by userid and then, within each userid value, sorts by score.

 Let's say we want to find male with age>=40
 
 Here we can consider using compund Index
 
 db.contacts.createIndex({"dob.age":1,gender:1})
 
 Obviously the order in which we write document matters how indexing will be done it won't create 2 indexes 
 
 It will create  one index like  30 male 30 male 30 female 31 male 31 female
 
 we can use index from left to right like alone age, age+gender but we can't use alone gender
 
 Basically if you want to use ith index then you can't use it without using 0-i-1 or create single field
 
 ********************************************************************************************************************

Indexes for Sorting!

db.contacts.explain("executionStats").find({"dob.age":35}).sort({gender:1})

It will use IXSCAN means indexing to sort!

If you are not using indexes and you have large amount of indexes and you sort you might timeout 32mb Limit

If indexing is not there then mongodb will fetch all documents in memory and do the sorting
 
 
db.mycollection.getIndexes()

It will list all indexes!

Unique indexes can help you to ensure no duplicates

db.contacts.createIndex({email:1},{unique:true})

If there are no duplicates then it will create index or create index before inserting values

*********************************************************************************************************************
Partial Indexes:

Let's say we are building an application where we want to how much people will get after they retire

You typically look for people older than 60.AGE or DOB using as index makes sense. But the problem is there are range of ages

which exists and are never queried.Index will be efficient but large index eats up size on disk.

We can actually create a partial index where you only add the values you regularly look at.

Let's create an index for age for only males as if most of our queries are for males! we can do the same for females

db.contacts.createIndex({"dob.age":1},{partialFilterExpression: {gender:"male"}})

If we mostly search for age>60

db.contacts.createIndex({"dob.age":1},{partialFilterExpression: {"dob.age":{$gt:60}}}) 

Creating an index with age for only males makes sense when we rarely serach for females !

************************************************************************************************************************
Unique Indexes

db.users.insertMany([{name:"Sanjay",email:"s@gmail.com"},{name:"Sanyam"}])

db.users.createIndex({email:1},{unique:true})  

created index of email in ascending order with unique values only if any document doesn't have this field it will treated as null

and multiple documents can't have null as only unique values are allowed.

db.users.insertMany([{name:"Prim",email:"p@gmail.com"},{name:"Aartik"}])

Here aartik doesn't have email so it will give error for duplicate key as index demands unique values only and it won't be added.

But sometimes we have to create index using field where field can be missing for some persons

I only want to add elements to my index when email exists

db.User.createIndex({email:1},{unique:true,partialFilterExpression:{email:{$exists:true}}})

*****************************************************************************************************************************

Query Diagonisis

db.myCollection.explain("executionStats").find({})

Suppose we habe hobbbies field which have value array of text;
If we create
