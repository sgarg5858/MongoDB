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

Scenario: Comments one On Instagram Post!

> use instagram
switched to db instagram
> db.post.insertOne({user:"Disha Patani",comments:["Pretty","Hey There","Looking Great"]})
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
*************************************************************************************************************
