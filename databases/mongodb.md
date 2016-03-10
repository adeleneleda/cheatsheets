# MongoDB

Open source document database

## I. Definitions

### A. Document

* Represent one record in MongoDB; consists of key-value pairs.
* Similar to JSON Objects
*  Values may include other documents, arrays, or arrays of documents


      {
        "_id" : ObjectId("54c955492b7c8eb21818bd09"),
          "student_number" : "2010-30010",
          "last_name" : "Dela Cruz",
          "first_name" : "Juan",
          "middle_name" : "Masipag",
          "address" : {
              "street" : "#35 Maharlika St.",
              "zipcode" : "30011",
              "city" : "Quezon City",
              "coord" : [ -73.9557413, 40.7720266 ]
          },
          "gwa" : "1.75",
          "program" : "Computer Science",
          "degree" : "Bachelor of Science"
      }


### B. Collection

Documents are stored in ``collections``. They are similar to tables but unlike a table, a collection does not require documents to have the same schema.

Documents stored in a collection has a unique identifier `_id` that acts as the primary key.

## II. Installation

### A. Mac OSx

  brew install mongodb

### B. Ubuntu

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

    echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

    sudo apt-get update

    sudo apt-get install -y mongodb-org

## III. Setup

### A. Running the Database

By default, mongod looks for your database at /data/db

     mongod

In case your database is at a different path, provide the --dbpath parameter
   
      mongod --dbpath .
   

### B. Running the Console

       mongo
     
By default, Mongo console would connect to localhost at port 27017. Make sure that mongod is running before you issue the `mongo` command.

### C. Switching database

    use mongo-cheatsheet
    # switched to db mongo-cheatsheet; would be created if non existent

## III. CRUD Operations

### A. Insertion

    db.students.insert(
      {
         "student_number" : "2010-30010",
         "last_name" : "Dela Cruz",
         "first_name" : "Juan",
         "middle_name" : "Masipag",
         "address" : {
           "street" : "#35 Maharlika St.",
           "zipcode" : "30011",
           "city" : "Quezon City",
           "coord" : [ -73.9557413, 40.7720266 ]
         },
         "gwa" : 1.75,
         "course" : "BS Computer Science"
      })
      
     # => WriteResult({ "nInserted" : 1 })
     
     # _id is automatically asssigned
     
     
### B. Read or Lookup

         # Find student with student_number 2010-30010
         db.students.find( { "student_number": "2010-30010" } )

         # Find student with zipcode (embedded attribute) 30011
         db.students.find( { "address.zipcode": "30011" } )
 
         # Find students with the gwa column greater than 1.25
         db.students.find( { "gwa": { $gt: 1.25 } } )

         # Find students with the gwa column less than 1.25 and course is BS Computer Science
         db.students.find( { "gwa": { $lt: 1.25 } , "course": "BS Computer Science"} )

         # Find students with the gwa column less than 1.25 or course is Computer Science
         db.students.find( { $or: [{ "gwa": { $lt: 1.25 } } , {"course": "BS Computer Science"}]})



### C. Update

#### i. Update Attribute/s

Updates first matching document with first_name: Juan

      db.students.update(
        { "first_name" : "Juan" },
        {
            $set: { "first_name": "Juana" }
        }
    )

The following code snippet updates the first matching document with first name Juan. And also sets the field ``lastModified`` to true (since it is non existent on the first run based on our schema, it will be created and set.)

     db.students.update(
         { "first_name" : "Juan" },
         {
             $set: { "first_name": "Juana" },
             $currentDate: { "lastModified": true }
         }
     )

#### ii. Update Embedded Fields


    db.students.update(
          { "first_name" : "Juana" },
          { $set: { "address.street": "#45 Maginhawa St.",
                    "address.city": "Quezon City" }}
    )

#### iii. Updating All Matching Documents

By default, ``update`` only updates the first matching document. To tell MongoDB to update all matching, we pass `` multi: true ``

    db.students.update(
      { "first_name" : "Juan" },
      { $set: { "address.street": "East 31st Street" } },
      { multi: true}
    )
  # WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
  
#### iv. Replace

     db.students.update(
       { "first_name" : "Juan" },
       {
           "first_name" : "Victoria",
           "address" : {
                   "street" : "Emerson Subdivision",
                   "city" : "Saog, Marilao"}
           }
     )

If you want to insert in case the data is non-existent, pass upsert: true as well to the update call.

     { upsert: true }


### D. Delete

#### i. Delete a document


      db.students.remove( { "first_name": "Juan" } )
      # removes all document
      
      db.students.remove( { "first_name": "Victoria" }, { justOne: true } )
      # removes only one of the matching document   
    
#### ii. Drop a Collection

      db.students.drop
      # => true


## IV. Query

In addition to simple lookup commands, you can also use aggregation:

     db.students.aggregate(
     [
       { $group: { "_id": "$address.city", "count": { $sum: 1 } } }
       ]
       );
    
Other available operators:

* sort
* project
* and many more...


## V. Data Import

### A. Import from JSON, CSV, TSV

To import dataset from a JSON, CSV, or TSV

      mongoimport --db mongo-cheatsheet --collection students --drop --file primer-dataset.json

Where:

* **Database name**: mongo-cheatsheet
* **Collection name**: students
* **Source File**: primer-dataset.json

By default, connects to localhost:27017. If you wish to connect to other ports, add flags: --host and --port

        mongoimport --db mongo-cheatsheet --collection students --drop --file primer-dataset.json --host 192.168.123.321 --port 27019


### B. Restore from Mongo Backup

To restore from a mongoDB backup:

        mongorestore --drop --db mongo-cheatsheet /path/to/your/dump

Where:

* **Database name**: mongo-cheatsheet
* **Dump path**: /path/to/your/dump

### C. Backup a Mongo Database

      mongodump --db mongo-cheatsheet

* **Database name**: mongo-cheatsheet