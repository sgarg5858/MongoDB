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
