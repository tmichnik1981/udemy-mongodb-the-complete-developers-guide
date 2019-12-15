# Mongodb the complete developers guide

#### Introduction

##### What ist Mongo?

- database
- Mongo server
  - Databases
    - Collections
      - Documents (JSON objects, schemaless)

##### BSON Data Structure

- No schema
- No/Few Relations

##### MongoDB Ecosystem

- Database
- Self-Managed / Enterprise
- CloudManager/OpsManager
- Atlas (Cloud)
- Mobile
- Compass

- Bi Connectors

- MongoDB Charts

- Stitch - Serverless Query API
- Serverless functions
- Database triggers
- Real-Time Sync

##### Installing MongoDB (Linux)

1. [MongoDB-community](https://www.mongodb.com/download-center/community)

2. Download and extract

3. Move to the directory you want

4. Got into the mongodb directory and create: **data/db** and **log** folders

5. Go to home directory and edit   .bashrc.

6. Set-up PATH pointing to mongo bin directory `export PATH=$PATH:/opt/mongodb-4.0.4/bin` 

7. Run `mongod --dbpath <SOME_PATH>/data/db`

   ```bash
    mongod --dbpath /opt/mongodb-4.0.4/data/db
   ```

8. Keep the window open unless you want to stop mongo server.

9. In a new window type `mongo` which will open a console and connect to the running server.

#####  First steps and commands

- List of databases `show dbs`
- Create(if not exists)/connect to the db `use shop`
- Inserting one document

```bash
> db.products.insertOne({name: "A Book", price: 12.99})
# one document has been inserted
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c02ba236e50e06628e43f88")
}
# adding one document with an embedded another one
> db.products.insertOne({name: "A Computer", price: 1229.99, description: "A high quality computer.", details: {cpu: "Intel i7 8770", memory: 32}})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c02cec56e50e06628e43f8a")
}

```

- Finding all

```shell
> db.products.find()
{ "_id" : ObjectId("5c02ba236e50e06628e43f88"), "name" : "A Book", "price" : 12.99 }
# print pretty
> db.products.find().pretty()
{
	"_id" : ObjectId("5c02ba236e50e06628e43f88"),
	"name" : "A Book",
	"price" : 12.99
}

```

#### The Basics & CRUD Operations

##### Creating Databases &, Collections

- Running with custom  port `mongod --port xxx`.
- List dbs `show dbs`.
- Switching to the db `use flights`.
- Inserting one document to **flightData** collection

```shell
db.flightData.insertOne({
     "departureAirport": "MUC",
     "arrivalAirport": "SFO",
     "aircraft": "Airbus A380",
     "distance": 12000,
     "intercontinental": true
   })
# result
{
	"acknowledged" : true,
	# auto created ID
	"insertedId" : ObjectId("5c05a0a0833226838546f881")
}
```

##### JSON v BSON

- Mongo stores BSON but we work with JSON.
- Inserting  a document with custom ID

```shell
db.flightData.insertOne({"departureAirport": "TXL", "arrivalAirport": "LHR", _id: "txl-lhr-1"})

```

##### CRUD Operations and MongoDB

- Create
  - `insertOne(data, options)`
  - `insertMany(data, options)`
- Read
  - `find(filter, options)`
  - `findOne(filter, options)`
- Update
  - updateOne(filter, data, options)
  - updateMany(filter, data, options)
  - replaceOne(filter, data, options)
- Delete
  - deleteOne(filter, options)
  - deleteMany(filter, options)

```shell
db.flightData.deleteOne({departureAirport: "TXL"})

# delete one
# where  _id =  ObjectId("5c05a032833226838546f87f")
db.flightData.deleteOne({"_id": ObjectId("5c05a032833226838546f87f")}

# updating one document 
# where distance = 12000
# set (updates field or insert)  marker = "delete"
db.flightData.updateOne({distance: 12000}, {$set:{marker: "delete"}})

# updating all ducuments 
db.flightData.updateMany({}, {$set:{marker: "toDelete"}})

# delete many
# where marker: "toDelete"
db.flightData.deleteMany({marker: "toDelete"})

# insert many
 db.flightData.insertMany([
 {
     "departureAirport": "MUC",
     "arrivalAirport": "SFO",
     "aircraft": "Airbus A380",
     "distance": 12000,
     "intercontinental": true
   },
   {
     "departureAirport": "LHR",
     "arrivalAirport": "TXL",
     "aircraft": "Airbus A320",
     "distance": 950,
     "intercontinental": false
   }
])
```

##### More about finding

```shell
# find all 
# where intercontinental is true
db.flightData.find({intercontinental: true}).pretty()

# find all 
# where distance  > 10000
db.flightData.find({distance: {$gt: 10000}}).pretty()
```

##### udpate() vs updateMany()

```shell
# update object 
# where _id: 5c0d2e3d988dca298fac58e3
# set delayed: true
db.flightData.updateOne({_id: ObjectId("5c0d2e3d988dca298fac58e3")}, {$set: {delayed: true}})
# update all documents which met criteria
db.flightData.updateMany({_id: ObjectId("5c0d2e3d988dca298fac58e3")}, {$set: {delayed: false}})

# update will set delayed: false and remove other fields
db.flightData.updateMany({_id: ObjectId("5c0d2e3d988dca298fac58e3")}, {{delayed: false}})

# replaces fields of one object with field of another one
# id remains the same
db.flightData.replaceOne({_id: ObjectId("5c0d2e3d988dca298fac58e3")},  {
     "departureAirport": "MUC",
     "arrivalAirport": "SFO",
     "aircraft": "Airbus A380",
     "distance": 12000,
     "intercontinental": true
   })

```

##### Understand find()

```shell
# returns a cursor
# only 20 elements in a console
db.passengers.find()

# exhaust the cursor and returns all elements
db.passengers.find().toArray()

# print json for each document
db.passengers.find().forEach((passengerData) => {printjson(passengerData)})
```

##### Projection

```shell
# limiting the columns we want to return
# 1 - include, 0 - exclude
db.passengers.find({}, {name: 1}).pretty()
 
db.passengers.find({}, {name: 1, _id: 0}).pretty()
```

##### Embedded Documents

- 100 levels of nesting
- Max 16mb/document
- Documents may contain Arrays

```shell
# update with a nested document
db.flightData.updateMany({}, {$set: {status: {description: "on-time", lastUpdated: "1 hour ago"}}})

# results:
{
	"_id" : ObjectId("5c0ebb7a8fab4d6590b1c18b"),
	"departureAirport" : "LHR",
	"arrivalAirport" : "TXL",
	"aircraft" : "Airbus A320",
	"distance" : 950,
	"intercontinental" : false,
	"status" : {
		"description" : "on-time",
		"lastUpdated" : "1 hour ago"
	}
}

# another update

db.flightData.updateMany({}, {$set: {status: {description: "on-time", lastUpdated: "1 hour ago", details: {responsible: "Max Schwarzmueller"}}}})

```

##### Arrays

- Numbers, string or other documents

```shell
# inserting an array of hobbies
db.passengers.updateOne({name: "Albert Twostone"}, {$set: {hobbies: ["sports", ,"cooking"]}})

# results

{
	"_id" : ObjectId("5c0ebbfc8fab4d6590b1c19f"),
	"name" : "Albert Twostone",
	"age" : 68,
	"hobbies" : [
		"sports",
		undefined,
		"cooking"
	]
}
```

##### Accessing Structured Data

```shell
# Find a passenger with name:"Albert Twostone" and return hobbies
db.passengers.findOne({name:"Albert Twostone"}).hobbies

# Finding passengers containing hobby: sports on a list
db.passengers.find({hobbies:"sports"}

# Quering with an embedded document's fields
db.flightData.find({"status.description": "on-time"}).pretty()

db.flightData.find({"status.details.responsible": "Max Schwarzmueller"}).pretty()
})

```

##### Assignment 1

```shell
# creating a new db
use hospital

# inserting patients data

db.patients.insertMany([
...     {
...         "firstName": "Max",
...         "lastName": "Schwarzmueller",
...         "age": 29,
...         "history": [
...             {"disease": "cold", "treatment": "To stay at home, to drink a lot water, to take Gripex"}
...         ]    
...     },
...     {
...         "firstName": "John",
...         "lastName": "Smith",
...         "age": 29,
...         "history": [
...             {"disease": "headache", "treatment": "To take the painkiller."}
...         ]    
...     },
...     {
...         "firstName": "Jan",
...         "lastName": "Kowalski",
...         "age": 36,
...         "history": [
...             {"disease": "broke leg", "treatment": "To splint a broken leg."}
...         ]    
...     

# upadating 1 patient
db.patients.updateMany({lastName: "Smith"},{$set: {firstName: "Albert", age: 37, history: [{disease:"fatige",treatment:"To take a day off"}]}})

# find all patients older than 30
db.patients.find({age :{$gt: 30}}).pretty()

# delete patients with cold as a disease
db.patients.deleteMany({"history.disease": "cold" })
```

##### Materials

- https://docs.mongodb.com/ecosystem/drivers/
- https://docs.mongodb.com/manual/tutorial/getting-started/

##### Schemas & Relations: How to Structure Documents

- Mongo is schemaless
- One collection may contain completely different structures 

##### Data Types

- [BSON types](https://docs.mongodb.com/manual/reference/bson-types/)
- [Limits](https://docs.mongodb.com/manual/reference/limits/)

- Text - "Max"
- Boolean  - true/false
- Number
  - Integer (int32)
  - NumberLong (int64) (integers and decimal number)
  - NumberDecimal - 12.99, high precision
  - **NumberInt** creates a **int32** value => `NumberInt(55)`
  - **NumberLong** creates a **int64** value => `NumberLong(7489729384792)`
  - Typing `insertOne({a: 1}`) will add **normal double** into the database as   shell is based on JS which only knows float.
- ObjectId - ObjectId("sdwdf") special object generated by Mongo which contains not only a random string but also a temporal component.
- ISODate - ("2018-09-09")
- Timestamp - Timestamp(11421532)
- Embedded Document
- Array - list of strings, numbers, documents etc

```shell
use companyDate

db.companies.insertOne({name: "Fresh Apples Inc", isStartup: true, employees: 33, funding: 12345678901234567890, details: {ceo: "Mark Super"}, tags :[{title: "super"}, {title: "perfect"}], foundingDate: new Date(), insertedAt: new Timestamp()})

db.companies.find().pretty()
{
	"_id" : ObjectId("5c29151cb356e7b2b73ebf16"),
	"name" : "Fresh Apples Inc",
	"isStartup" : true,
	"employees" : 33,
	"funding" : 12345678901234567000,
	"details" : {
		"ceo" : "Mark Super"
	},
	"tags" : [
		{
			"title" : "super"
		},
		{
			"title" : "perfect"
		}
	],
	"foundingDate" : ISODate("2018-12-30T18:57:32.023Z"),
	"insertedAt" : Timestamp(1546196252, 1)
}

```

##### Data Schemas & Data Modelling

- Which Data does my App need or generate?
  - User Information, Product Information, Orders,...
  - Defines the Fields you'll need (and how they relate).
- Where do I need my Data?
  - Welcome Page, Products List Page, Orders Page.
  - Defines your required collections + field groupings.
- Which kind of Data or Information do I want to display?
  - Welcome Page: Product Names; Products Page: ...
  - Defines which queries you'll need.
- How often do I fetch my data?
  - For every page reload.
  - Defines whether you should optimize for easy fetching.
- How often do I write or change my data?
  - Orders => Often, Product Data => Ralely.
  - Defines whether you should optimize for easy writing.

##### Relations - Options

- Nested / Embedded Documents
- References

```json
# Customers
{
    userName: 'max',
    favBooks: ['id1', 'id2']
}

# Books
{
    _id: 'id1',
    name: 'Lord of the Rings 1'
}
```

##### One to One Relations - Embedded

```shell
db.patients.insertOne({name: "Max", age: 29, diseaseSummary: {diseases: ["cold", "broken leg"]}})

db.patients.findOne()
{
	"_id" : ObjectId("5c2b96932c8f7d345faeb04d"),
	"name" : "Max",
	"age" : 29,
	"diseaseSummary" : {
		"diseases" : [
			"cold",
			"broken leg"
		]
	}
}


```

##### One to One Relations - Using References

- Sometimes in analytical apps it is better to use references for 1on1 relations.

```shell
db.persons.insertOne({name: "Max", age: 29, salary: 3000})

db.cars.insertOne({model: "BMW", price: 40000, owner: ObjectId("5c2b987b2c8f7d345faeb04e")})

db.cars.findOne()
{
	"_id" : ObjectId("5c2b9e182c8f7d345faeb04f"),
	"model" : "BMW",
	"price" : 40000,
	"owner" : ObjectId("5c2b987b2c8f7d345faeb04e")
}
```

##### One to Many - Embedded

- ie. posts and comments

```shell
db.questionThreads.insertOne({creator: "Max", question: "How does that wort?", answers:[{text: "Like that."}, {text: "Thanks!"}]})

db.questionThreads.findOne()
{
	"_id" : ObjectId("5c2ba8d02c8f7d345faeb050"),
	"creator" : "Max",
	"question" : "How does that wort?",
	"answers" : [
		{
			"text" : "Like that."
		},
		{
			"text" : "Thanks!"
		}
	]
}

```

##### One to Many - Using References 

- References or embedded documents?
  - Document size limit 16 MB.
  - Fetching too much data. 

```shell
use cityData

db.cities.insertOne({name: "New York City", coordinates: {lat:21, lng:55}})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c322b76bd70a4a8fde016f1")
}

dn.citizens.insertMany([
{name: "Max Schwarzmueller", cityId: ObjectId("5c322b76bd70a4a8fde016f1")}, 
{name: "Manuel Lorenz", cityId: ObjectId("5c322b76bd70a4a8fde016f1")}
])

db.citizens.find().pretty()
{
	"_id" : ObjectId("5c322dc5bd70a4a8fde016f2"),
	"name" : "Max Schwarzmueller",
	"cityId" : ObjectId("5c322b76bd70a4a8fde016f1")
}
{
	"_id" : ObjectId("5c322dc5bd70a4a8fde016f3"),
	"name" : "Manuel Lorenz",
	"cityId" : ObjectId("5c322b76bd70a4a8fde016f1")
}

```



##### Many To Many - Embedded

- References or embedded documents?
  - Are duplicates fine?
  - Think about changes. Do we need to change each product (including the embedded one) or only one from 'products' collection,

```shell
use shop

db.products.insertOne({title: "A Book", price: 12.9})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c32122a6c7ec66420a55727")
}

db.customers.insertOne({name: "Max", age: 29})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c3212606c7ec66420a55728")
}

# customers will hold an array to orders with the references to products
db.customers.updateOne({},{$set: {orders:[{productId: ObjectId("5c32122a6c7ec66420a55727"), quantity: 2}]}})

# another aproach with an embedded product
db.customers.updateOne({},{$set: {orders:[{title: "A Book", price: 12.9}]}})
```

##### Many To Many - Using References 

- Low probability of changes in related documents => use embedded documents  
- High probability of changes in related documents => use references  

```shell
use bookRegistry

db.books.insertOne({name: "My favorite Book", authors: [{name: "Max Schwarz", age:29}, {name: "Manuel Lor", age: 30}]})

# authors as embedded documents
db.books.find().pretty()
{
	"_id" : ObjectId("5c323015bd70a4a8fde016f4"),
	"name" : "My favorite Book",
	"authors" : [
		{
			"name" : "Max Schwarz",
			"age" : 29
		},
		{
			"name" : "Manuel Lor",
			"age" : 30
		}
}

db.authors.insertMany([{name: "Max Schwarz", age: 29, address: {street: "Main"}}, {name: "Manuel Lor", age: 30, address: {street: "Tree"}}])

db.authors.find().pretty()
{
	"_id" : ObjectId("5c32333abd70a4a8fde016f5"),
	"name" : "Max Schwarz",
	"age" : 29,
	"address" : {
		"street" : "Main"
	}
}
{
	"_id" : ObjectId("5c32333abd70a4a8fde016f6"),
	"name" : "Manuel Lor",
	"age" : 30,
	"address" : {
		"street" : "Tree"
	}
}

# It is better idea to store authors as references

db.books.updateOne({},{$set: {authors: [ObjectId("5c32333abd70a4a8fde016f5"),ObjectId("5c32333abd70a4a8fde016f6")]}})

db.books.find().pretty()
{
	"_id" : ObjectId("5c323015bd70a4a8fde016f4"),
	"name" : "My favorite Book",
	"authors" : [
		ObjectId("5c32333abd70a4a8fde016f5"),
		ObjectId("5c32333abd70a4a8fde016f6")
	]
}

```

##### Relations - Options

- Nested / Embedded Documents
  - Group data together logically.
  - Great for data that belongs together and is not really overlapping with other data.
  - Avoid super-deep nesting (100 + levels) or extremely long arrays (16mb size limit per documents).
- References
  - Split data across collections.
  - Great for related but shared data as well as for data is used in relations and standalone.
  - Allows you to overcome nesting and size limits (by creating new documents).

##### Joining with "lookUp"

```
db.books.aggregate([{$lookup: {from: "authors", localField: "authors", foreignField: "_id", as: "creators"}}]).pretty()
{
	"_id" : ObjectId("5c323015bd70a4a8fde016f4"),
	"name" : "My favorite Book",
	"authors" : [
		ObjectId("5c32333abd70a4a8fde016f5"),
		ObjectId("5c32333abd70a4a8fde016f6")
	],
	"creators" : [
		{
			"_id" : ObjectId("5c32333abd70a4a8fde016f5"),
			"name" : "Max Schwarz",
			"age" : 29,
			"address" : {
				"street" : "Main"
			}
		},
		{
			"_id" : ObjectId("5c32333abd70a4a8fde016f6"),
			"name" : "Manuel Lor",
			"age" : 30,
			"address" : {
				"street" : "Tree"
			}
		}
	]
}


```

##### Exercise project: A Blog

- 2 collections
- User
  - _id
  - name
  - age
  - email
- Post
  - _id
  - title
  - text
  - tags: []
  - comments: [{}. {}]
- Comment
  - _id
  - text

```bash
use blog

db.users.insertMany([{name: "Max Schwarzmueller", age: 29, email: "max@test.com"}, {name: "Manuel Lorenz", age: 30, email: "manu@test.com"}])

# result
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5d0cda846069ef1256a097ab"),
		ObjectId("5d0cda846069ef1256a097ac")
	]
}

db.users.find().pretty()
{
	"_id" : ObjectId("5d0cda846069ef1256a097ab"),
	"name" : "Max Schwarzmueller",
	"age" : 29,
	"email" : "max@test.com"
}
{
	"_id" : ObjectId("5d0cda846069ef1256a097ac"),
	"name" : "Manuel Lorenz",
	"age" : 30,
	"email" : "manu@test.com"
}

db.posts.insertOne({title: "My first Post!", text: "This is my first post, I hope you like it!", tags: ["new", "tech"], creator: ObjectId("5d0cda846069ef1256a097ac"), comments: [{text: "I like this post!", author: ObjectId("5d0cda846069ef1256a097ab")}]})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5d0cdd05bc391f8c6178cb70")
}

db.posts.findOne()
{
	"_id" : ObjectId("5d0cdd05bc391f8c6178cb70"),
	"title" : "My first Post!",
	"text" : "This is my first post, I hope you like it!",
	"tags" : [
		"new",
		"tech"
	],
	"creator" : ObjectId("5d0cda846069ef1256a097ac"),
	"comments" : [
		{
			"text" : "I like this post!",
			"author" : ObjectId("5d0cda846069ef1256a097ab")
		}
	]
}

```

##### Schema Validation

- validationLevel
  - Which documents get validated?
  - strict - All inserts & updates
  - moderate - All inserts, updates for previously correct documents
- validationAction
  - What happens if validation fails?
  - error - Throw error and deny insert/update
  - warn - Log warning but proceed
- Creating a collection with a **validator**

```bash
# validation.js

db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'], # required fields
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
                bsonType: 'objectId', # id type
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  }
});
```

##### Change the validation action

```bash
# validation-2.js

db.runCommand({collMod: "posts", validator: {....}})
```

##### Useful Resources & Links

- The MongoDB Limits: <https://docs.mongodb.com/manual/reference/limits/>
- The MongoDB Data Types: <https://docs.mongodb.com/manual/reference/bson-types/>
- More on Schema Validation: <https://docs.mongodb.com/manual/core/schema-validation/>

#### Exploring The Shell & the Server 

##### mongod

```bash
mongod --port 270 # changing port
mongod --help # help
mongod --logpath arg # where logs are stored
mongod --dbpath arg  # directory where your daa are stored 

# mongod --dbpath /opt/mongodb-4.0.4/data/db --logpath /opt/mongodb- 4.0.4/log/mongo.log

mongod --repair # run repair on all dbs 
mongod --directoryperdb # each database will be stored in a separate directory
mongod --storageEngine arg # what storage engine to use - defaults  to wiredTiger

mongod --fork # fork server process. Runs server in the backgroud (Linux, MacOS).

```

##### Using **config** file

- Create file mongo.cfg with your configuration
- Run mongo pointing to that file

~~~bash
# mongod.cfg

mongod -f [ --config ] PATH_TO_mongo.cfg
~~~

##### Mongo shell

```bash
mongo --help
mongo --nodb # don't connect to mongod on startup

mongo  --port arg # port to connect to
mongo  --host arg # server to connect to

mongo -u [ --username ] arg # username for authentication
mongo -p [ --password ] arg # password for authentication

```

- Run `mongo`

```bash
help

db.help()         # help on db methods
db.mycoll.help()  # help on collection methods
sh.help()         # sharding helpers
rs.help()         # replica set helpers
help admin        # administrative help
help connect      # connecting to a db help
help keys         # key shortcuts
help misc         # misc things to know
help mr           # mapreduce

show dbs          # show database names
show collections  # show collections in current database
show users        # show users in current database
show profile      # show most recent system.profile entries with time >= 1ms


use blog
db.help()  # displays db.x commands

db.posts.help() # displays all commands that can be done on 'posts' collection


```

##### Useful Resources & Links

- More Details about Config Files: <https://docs.mongodb.com/manual/reference/configuration-options/>
- More Details about the Shell (`mongo`) Options: <https://docs.mongodb.com/manual/reference/program/mongo/>
- More Details about the Server (`mongod`) Options: <https://docs.mongodb.com/manual/reference/program/mongod/>

#### MongoDB Compass

- Graphical client for mongoDB

##### Useful Resources & Links

- The MongoDB Compass Docs: <https://docs.mongodb.com/compass/master/install/>
- Only available in the Enterprise Edition => Exploring Schemas in Compass: <https://docs.mongodb.com/compass/master/schema/>

#### Create operations

##### Insert methods

```bash
use contactData

db.persons.insertOne({name:"Max", age:30, hobbies: ["Sports", "Cooking"]})

db.persons.insertOne({name:"Manuel", age:31, hobbies: ["Cars", "Cooking"]})

db.persons.insertMany([{name: "Anna", age: 29, hobbies: ["Sports", "Yoga"]}])

db.persons.insertMany([{name: "Maria", age: 31},{name: "Chris", age: 25}])

db.persons.insert({name:"Phil", age:35})

db.persons.insert([{name:"Sandeep", age:28}, {name:"Hans", age:38}])
```

##### Working with Ordered Inserts

```bash
db.hobbies.insertMany([{_id: "sports", name: "Sports"}, {_id: "cooking", name: "Cooking"}, {_id: "cars", name: "Cars"}]) # own id

db.hobbies.insertMany([{_id: "yoga", name: "Yoga"}, {_id: "cooking", name: "Cooking"}, {_id: "cars", name: "Cars"}]) # this will cause a duplicate id exception
# however thanks to  'Ordered Inserts' feature, 'yoga' is insterted but the rest is cancelled
```

- Turning off  Ordered Inserts -`{ordered: false}`

```
db.hobbies.insertMany([{_id: "yoga", name: "Yoga"}, {_id: "cooking", name: "Cooking"}, {_id: "hiking", name: "Hiking"}], {ordered: false})
```

##### WriteConcern

~~~json
# w - how many instances need to be aknowleged
# j - journal (backup TODO list)
{w: 1, j: undefined} 
# j : true success only when insert is aknowleged and saved to the journal
{w: 1, j: true} 
{w: 1, wtimeout: 200,  j: true} 
~~~

##### WriteConcern in practice

```bash
db.persons.insertOne({name: "Chrissy", age: 41}, {writeConcern: {w: 0}}) # do not wait for response. You do not know if it reached the server


db.persons.insertOne({name: "Alex", age: 36}, {writeConcern: {w: 1}}) # wait for server's confirmation

db.persons.insertOne({name: "Michael", age: 51}, {writeConcern: {w: 1, j: false}}) 

db.persons.insertOne({name: "Michaela", age: 51}, {writeConcern: {w: 1, j: true}}) # wait for journal

db.persons.insertOne({name: "Aliya", age: 22}, {writeConcern: {w: 1, j: true, wtimeout: 1}})
```

##### Atomicity

- Atomic operations  on document level

Assigments

```bash
db.companies.insertOne({name: "Polsoft", stock_value: 10000}) 
db.companies.insertMany([{name: "Polbud", stock_value: 90000}, {name: "Polmos", stock_value: 87000}]) 

db.companies.insertMany([{_id: "polex", name: "Polex", stock_value: 90000}, {_id: "polmix", name: "Polmix", stock_value: 87000}]) 
# ordered : false
db.companies.insertMany([{_id: "polex", name: "Polex", stock_value: 90000}, {_id: "polmax", name: "Polmax", stock_value: 87000}], {ordered: false})

db.companies.insertOne({name: "Polox", stock_value: 1000}, {writeConcern: {w: 1, j: false}}) 

db.companies.insertOne({name: "Polox2", stock_value: 1200}, {writeConcern: {w: 1, j: true}}) 
```

##### Importing Data

```bash
mongoimport tv-shows.json -d movieData -c movies --jsonArray --drop
```

##### Useful Resources & Links

- insertOne(): <https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/>
- insertMany(): <https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/>
- Atomicity: <https://docs.mongodb.com/manual/core/write-operations-atomicity/#atomicity>
- Write Concern: <https://docs.mongodb.com/manual/reference/write-concern/>
- Using mongoimport: <https://docs.mongodb.com/manual/reference/program/mongoimport/index.html>

#### Reading Documents with Operators

##### Methods, Filters & Operators

```bash
# Access current db . Access this collection . Apply this Method ( Equality/Single Value Filter )

db.myCollection.find({age: 32})


# Access current db . Access this collection . Apply this Method ( Range Filter )

db.myCollection.find({age: { $gt:30 } })
```

##### Operators

- Query Selectors
  - Query Selectors - to locate data ie. `$eq`
  - Projection Operators - to modify data presentation ie. `$`

- Update - to modify + add additional data ie. `$inc`
  - Fields
  - Arrays
- Aggregation
  - Pipeline Stages
  - Pipeline Operators

##### Query Selectors & Projection Operators

- Query Selectors
  - Comparison
  - Evaluation
  - Logical
  - Array
  - Element
  - Comments
  - Geospatial

- Projection Operators
  - $
  - $elemMatch
  - $meta
  - $slice

##### findOne() and find()

```bash
use movieData

db.movies.findOne() # always returns only first element

db.movies.find().pretty() # returns cursor

db.movies.find({name: "The Last Ship"}).pretty()
```

##### Comparison Operators

~~~bash
db.movies.find({runtime: {$eq: 60}})

db.movies.find({runtime: {$ne: 60}})

db.movies.find({runtime: {$gt: 60}})
db.movies.find({runtime: {$gte: 60}})

db.movies.find({runtime: {$lt: 30}})
db.movies.find({runtime: {$lte: 30}})
~~~

##### Querying Embedded Fields & Arrays

```bash
db.movies.find({"rating.average": {$gt:7}}).pretty()

db.movies.find({genres: "Drama"}).pretty() # there is Drama in genres array

db.movies.find({genres: ["Drama"]}).pretty() # there is an array quals to ["Drama"] array

```

##### in & nin

```bash
# set of values

# in
db.movies.find({runtime: {$in: [30, 42]}}).pretty()

# not in
db.movies.find({runtime: {$nin: [30, 42]}}).pretty()
```

##### Logical operators

```bash
# OR

db.movies.find({$or:[{"rating.average": {$lt: 5}}, {"rating.average": {$gt: 9.3}}  ]}).pretty()

# NOR

db.movies.find({$nor:[{"rating.average": {$lt: 5}}, {"rating.average": {$gt: 9.3}}  ]}).pretty()

# AND 1 

db.movies.find({$and:[{"rating.average": {$gt: 9}}, {genres: "Drama" }]}).pretty()

# AND 2
db.movies.find({"rating.average": {$gt: 9}, {genres: "Drama"}).pretty()

# NOT

db.movies.find({runtime: {$not:{$eq: 60}}}).count()

db.movies.find({runtime: {$ne: 60}}).count()

```

##### Element operators

```bash
use user

db.users.insertMany([{name: "Max", hobbies: [{title: "Sports", frequency: 3}, {title: "Cooking", frequency: 6}], phone: 0131782734}, 
{name: "Manuel", hobbies: [{title: "Cooking", frequency: 5}, {title: "Cars", frequency: 2}], phone: "012177972", age: 30}])

db.users.find().pretty()

# EXISTS
db.users.find({age: {$exists: true}}).pretty()
db.users.find({age: {$exists: true, $gt: 30}}).pretty()
db.users.find({age: {$exists: true, $gte: 30}}).pretty()

db.users.insertOne({name: "Anna", hobbies: [{title: "Sports", frequency: 2}, {title: "Yoga", frequency: 3}], phone: "80811987291", age: null})

# field 'age' exists and is not null
db.users.find({age: {$exists: true, $ne: null}}).pretty()

# $type

db.users.find({phone: {$type: "double"}}).pretty()
db.users.find({phone: {$type: ["double", "string"]}}).pretty()

# regex

use movieData

db.movies.find({summary: "musical"}).pretty()

db.movies.find({summary: {$regex: /musical/}}).pretty()

# $expr

use financialData

db.sales.insertMany([{volume: 100, target: 120}, {volume: 89, target: 80}, {volume: 200, target: 177}])

# $ sign says, to use the value of that field 
db.sales.find().pretty()
db.sales.find({$expr: {$gt: ["$volume", "$target"]}}).pretty()

db.sales.find({$expr: {$gt: [{$cond: {if: {$gte: ["$volume", 190]}, then: {$subtract: ["$volume", 10]}, else: "$volume"}}, "$target"]}}).pretty()

```

##### Assignment 3

- Import boxoffice.json to **boxOffice** db, collection: **movieStars**
- movies with a rating > 9.2 and a runtime < 100 min
- movies with genre == "drama" or "action"
- movies where visitors exceeded expectedVisitors

```bash
db.movieStars.find().pretty()

db.movieStars.find({$and:[{"meta.rating": {$gt: 9.2}}, {"meta.runtime": {$lt: 100} }]}).pretty()

db.movieStars.find({genre: {$in: ["drama","action"]}}).pretty()

db.movieStars.find({$expr: {$gt: ["$visitors", "$expectedVisitors"]}}).pretty()
```

##### More about Querying Arrays

```bash
use user
	
#	"hobbies" : [
#		{
#			"title" : "Sports",
#			"frequency" : 3
#		},
#		{
#			"title" : "Cooking",
#			"frequency" : 6
#		}
#	],

# searching through embedded documents
db.users.find({"hobbies.title": "Sports"}).pretty()


# $size
db.users.insertOne({name: "Chris", hobbies: ["Sports", "Cooking", "Hiking"]})
# size of an Array. Only 
db.users.find({hobbies: {$size: 3}}).pretty()

# $all

use boxOffice
db.movieStars.find().pretty()

# both elements must exist in an array
db.movieStars.find({genre: {$all: ["action", "thriller"]}}).pretty()

# $elemMatch

use user
# checking an embedded element
db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}).pretty()
```

##### Assignment 4

- Import boxoffice-extended.json to **boxOffice** db, collection: **exmoviestars**

- Find 2 movies with exactly two genres
- Find all movie which aired in 2018
- Find all movies which have ratings greater than 8 but lower than 10 

```bash
mongoimport boxoffice-extended.json -d boxOffice -c exmoviestars --jsonArray --drop

db.exmoviestars.find().pretty()

# Find 2 movies with exactly two genres
db.exmoviestars.find({genre: {$size: 2}}).pretty()

# Find all movie which aired in 2018
db.exmoviestars.find({"meta.aired": 2018}).pretty()

# Find all movies which have ratings greater than 8 but lower than 10 
db.exmoviestars.find({ratings: {$elemMatch: {$gt: 8, $lt: 10}}}).pretty()

```

##### Understanding Cursor

```bash
use movieData

db.movies.find().count()

const dataCursor = db.movies.find()
# next element in the cursot
dataCursor.next()
# iterate through all
dataCursor.forEach(doc => {printjson(doc)})

dataCursor.hasNext()

# sorting: -1 - desc, 1 - asc

db.movies.find().sort({"rating.average": -1}).pretty()

db.movies.find().sort({"rating.average": 1, runtime: -1}).pretty()

# skipping and limiting cursor results

db.movies.find().sort({"rating.average": 1, runtime: -1}).skip(10).pretty()

db.movies.find().sort({"rating.average": 1, runtime: -1}).skip(10).limit(10).pretty()

# using projections
# 1 - return the field 

db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1}).pretty()

db.movies.find({}, {name: 1, genres: 1, runtime: 1, "rating.average": 1, "schedule.time": 1}).pretty()

# projection in arrays
# displa only first elem from genres array

db.movies.find({genres: "Drama"}, {"genres.$": 1}).pretty()

db.movies.find({genres: {$all: ["Drama", "Horror"]}}, {"genres.$": 1}).pretty()

# display only values wihich match {$eq: "Horror"}
db.movies.find({"rating.average": {$gt: 9}}, {genres: {$elemMatch:{$eq: "Horror"}}}).pretty()

# $slice
# display only 2 first genres
db.movies.find({"rating.average": {$gt: 9}}, {genres: {$slice: 2}, name:1}).pretty()
# display only 2  genres, skipp the first 
db.movies.find({"rating.average": {$gt: 9}}, {genres: {$slice: [1,2]}, name:1}).pretty()
```

##### Useful Resources & Links

- More on find(): <https://docs.mongodb.com/manual/reference/method/db.collection.find/>
- More on Cursors: <https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/>
- Query Operator Reference: <https://docs.mongodb.com/manual/reference/operator/query/>

#### Update operations

```bash
use user

mongoimport users.json -d user -c users --jsonArray --drop


# $set - adds or updates fields
db.users.updateOne({_id: ObjectId("5d2e0b8b99fb995191677079")}, {$set: {hobbies: [{title: "Sports", frequency: 5}, {title: "Cooking", frequency: 3}, {title: "Hiking", frequency: 1}]}})

db.users.updateMany({"hobbies.title": "Sports"}, {$set: {isSporty:true}})

# update multiple fields

db.users.updateOne({_id: ObjectId("5d2e0b8b99fb995191677079")}, {$set: {age: 40, phone: 483924792}})

# incrementing & decrementing
# inc by 2
db.users.updateOne({name: "Manuel"}, {$inc: {age: 2}})

# decr by 1
db.users.updateOne({name: "Manuel"}, {$inc: {age: -1}})

db.users.updateOne({name: "Manuel"}, {$inc: {age: 1}, $set: {isSporty: false}})

# $min, $max, $mul
# change is applied when oldAge > newAge
db.users.updateOne({name: "Chris"}, {$min: {age: 38}})
# change is applied when oldAge < newAge
db.users.updateOne({name: "Chris"}, {$max: {age: 32}})
# age is multipied by 1.1
db.users.updateOne({name: "Chris"}, {$mul: {age: 1.1}})

# removing fields
db.users.updateMany({isSporty: true}, {$unset: {phone: ""}})
# renaming fields
db.users.updateMany({}, {$rename: {age: "totalAge"}})

# upsert() - if the document does not exist, it will be created
db.users.updateOne({name: "Maria"}, {$set: {age: 38, hobbies: [{title: "Good food", frequency: 3}], isSporty: true}}, {upsert: true})
```

##### Assignment 5

- Create a new collection ("sports") and upsert two new documents into it (with these fields "title", "requiresTeam")
- Update all documents which do require a team by adding a new field with the minimum amount of players required
- Update all documents which do require a team by increasing the number of required players by 10

```bash
use sport

db.sports.updateMany({}, {$set: {title: "Football", requiresTeam: true}}, {upsert: true})
db.sports.updateMany({title: "Running"}, {$set: {requiresTeam: false}}, {upsert: true})
db.sports.updateMany({title: "Soccer"}, {$set: {requiresTeam: true}}, {upsert: true})

db.sports.updateMany({requiresTeam: true}, {$set: {minPlayers: 11}})

db.sports.updateMany({requiresTeam: true}, {$inc: {minPlayers: 10}})
```

##### Updating all Array elements

```bash
use user 

db.users.find({$and: [{"hobbies.title": "Sports"}, {"hobbies.frequency": {$gte: 3}}]}).pretty()

db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency:{$gte: 3} }}}).pretty()

db.users.updateMany({hobbies: {$elemMatch: {title: "Sports", frequency:{$gte: 3}}}}, {$set: {"hobbies.$.highFrequency": true }})

# updates all elements in an array
db.users.updateMany({totalAge: {$gt: 30}}, {$inc: {"hobbies.$[].frequency": -1}})

#  finding and updating Specific fields
db.users.updateMany({"hobbies.frequency": {$gt:2}}, {$set: {"hobbies.$[el].goodFrequency": true}}, {arrayFilters: [{"el.frequency": {$gt: 2}}]})

# adding elements to arrays

db.users.updateOne({name: "Maria"}, {$push: {hobbies: {title: "Sports", frequency: 2}}})

db.users.updateOne({name: "Maria"}, {$push: {hobbies: {$each: [{title: "Good Wine", frequency: 1}, {title: "Hiking", fequency: 2}], $sort: {frequency: -1}}}})

# removing elements from Arrays

db.users.updateOne({name: "Maria"}, {$pull: {hobbies: {title: "Hiking"}}})

# removing first element
db.users.updateOne({name: "Chris"}, {$pop: {hobbies: 1}})

# $addToSet - adds uniques values only
db.users.updateOne({name: "Maria"}, {$addToSet: {hobbies: {title: "Hiking", frequency: 2}}})

```

##### Useful Resources & Links

- Official Document Updating Docs: <https://docs.mongodb.com/manual/tutorial/update-documents/>

#### Delete operations

```bash
use user

db.users.deleteOne({name: "Chris"})

db.users.deleteMany({totalAge: {$gt: 30}, isSporty: true})

db.users.deleteMany({totalAge: {$exists: false}, isSporty: true})

# deleting all entries
db.users.deleteMany({})

# dropping the collection
dn.users.drop()
```

##### Useful Resources & Links

- Official Document Deletion Docs: <https://docs.mongodb.com/manual/tutorial/remove-documents/>

#### Working with indexes

- Speeds up queries
- Index exists additionally to the collection
- Collscan vs Ixscan
- Do not overuse indexes! Slows down inserts!
- Indexes may slow down queries when returning all records or vast majority. Indexes enforce 2 steps:  1 index scan and 2 returning real values. 
- Default index is created on _id

##### Adding a Single Field Index

 ```bash
mongoimport persons.json -d contactData -c contacts --jsonArray

use contactData

db.contacts.find().pretty()

db.contacts.find({"dob.age":{$gt: 60}}).pretty()

# explain how mongo executes a query
db.contacts.explain().find({"dob.age": {$gt: 60}})

	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "contactData.contacts",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"dob.age" : {
				"$gt" : 60
			}
		},
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"dob.age" : {
					"$gt" : 60
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]

# more details
db.contacts.explain("executionStats").find({"dob.age": {$gt: 60}})

# creating an index
# 1 - ascending, -1 - descending
db.contacts.createIndex({"dob.age": 1})

# return indexes
db.contacts.getIndexes()


 ```

##### Creating Compound Indexes

```bash
# index on 2 fields 
db.contacts.createIndex({"dob.age": 1, gender: 1})

db.contacts.explain().find({"dob.age": 35, gender: "male"})
```

##### Sorting

- Mongo will utilize order of an index
- 32 MB in memory threashold for sort (without index)

##### Configuring index

~~~bash
db.contacts.createIndex({email: 1, unique:true})
~~~

##### Partial indexes

- For data queried rarly
- Index is small

~~~bash
# index for dob.age but only works when gender : male

db.contacts.createIndex({"dob.age": 1},{partialFilterExpression: {gender: "male"}})

db.contacts.explain().find({"dob.age": {$gt: 60}, gender: "male"})

db.users.insertMany([{name: "Max", email: "max@test.com"}, {name: "Manu"}])

db.users.createIndex({email:1})

db.users.dropIndex({email:1})

db.users.createIndex({email:1}, {unique: true})

# dup key error as there already exist an email null
db.users.insertOne({name: "Anna"})

ror collection: contactData.users index: email_1 dup key: { : null } :
WriteError({
	"index" : 0,
	"code" : 11000,
	"errmsg" : "E11000 duplicate key error collection: contactData.users index: email_1 dup key: { : null }",
	"op" : {
		"_id" : ObjectId("5d374db76e9ffa01af91cfaf"),
		"name" : "Anna"
	}
})
db.users.dropIndex({email:1})

# unique index but only for existing email values. Nulls are ignored.
db.users.createIndex({email:1}, {unique: true, partialFilterExpression: {email: {$exists: true}}})

# TTL index

db.sessions.insertOne({data: "dksafkjel", createdAt: new Date()})

# after 10 sec records will be deleted
db.sessions.createIndex({createdAt: 1}, {expireAfterSeconds: 10})

db.sessions.find().pretty()

~~~

##### Query Diagnosis & Query Planning

- explain()
  - "queryPlanner" - Show Summary for Executed Query + Winning Plan
  - "executionStats" - Show Detailed Summary for Executed Query + Winning Plan + Possibly Rejected Plans
  - "allPlansExecution" - Show Detailed Summary for Executed Query + Winning Plan + Winning Plan Decision Process
- Observe
  - Miliseconds process Time
  - IXSCAN usually beats COLLSCAN
  - \# of Keys (in index) examined should be close to \# of Documents Examined  (or 0 doc)
  - \# of Documents Examined  should be close to # of Documents Returned (or 0)

```bash
# covered queries

db.customers.insertMany([{name: "Max", age: 29, salary: 3000}, {name: "Manu", age: 30, salary: 4000}])

db.customers.createIndex({name: 1})

db.customers.explain("executionStats").find({name:"Max"})

# covered query as we return only name which is actually in an index
db.customers.explain("executionStats").find({name:"Max"}, {_id: 0, name:1})

```

##### Rejecting a Plan

```bash
db.customers.createIndex({age:1, name: 1})

db.customers.explain().find({name:"Max",age: 30})

	"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"age" : 1,
					"name" : 1
				},
				"indexName" : "age_1_name_1",

	"rejectedPlans" : [
			{
				"stage" : "FETCH",
				"filter" : {
					"age" : {
						"$eq" : 30
					}
				},
				"inputStage" : {
					"stage" : "IXSCAN",
					"keyPattern" : {
						"name" : 1
					},
					"indexName" : "name_1",

# all plans
db.customers.explain("executionStats").find({name:"Max",age: 30})
```

- Winning Plans
  - Choose the fastest
  - cache winning plan for query
  - cache is cleaned after
    - 1000 writes
    - Index is Rebuilt
    - Other Indexes are Added or Removed
    - MongoDB server is restarted

##### Using Multi-key Indexes

- Multi-key indexes are bigger than normal because extracts values from an array and puts them into an index
- Use carefully as they can affect performance
- Compound indexes with multi-key index are possible but only one element of an index  can be created on an  array 

```bash
use contactData

db.contacts.drop()

db.contacts.insertOne({name: "Max", hobbies: ["Cooking", "Sports"], addresses: [{street: "Main Street"}, {street: "Second Street"}]})

db.contacts.find().pretty()

# index on an array creates a multi-key index
db.contacts.createIndex({hobbies:1})

db.contacts.explain("executionStats").find({hobbies: "Sports"})
db.contacts.createIndex({addresses:1})

db.contacts.explain("executionStats").find({"addresses.street": "Main Street"})
# COLLSCAN as index holds this time documents

	"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"addresses.street" : {
					"$eq" : "Main Street"
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
		
db.contacts.explain("executionStats").find({addresses: {street: "Main Street"}})

	"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 1,
				"executionTimeMillisEstimate" : 0,
				"works" : 2,
				"advanced" : 1,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"addresses" : 1
				},


db.contacts.createIndex({"addresses.street":1})

db.contacts.explain("executionStats").find({"addresses.street": "Main Street"})
	"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 1,
				"executionTimeMillisEstimate" : 0,
				"works" : 2,
				"advanced" : 1,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"addresses.street" : 1
				},

```

##### Text indexes

- Mongo can index big text. It extracts keywords and removes stop words. 
- Only one text index per collection

```bash
use contactData

db.products.insertMany([{title: "A Book", description: "This is an awesome book about a young artist!"}, {title: "Red T-Shirt", description: "This T-Shirt is red and it's pretty awesome!"}])

db.products.find().pretty()

# special type: text
db.products.createIndex({description: "text"})
# special keywords  $text, $search for searching  the indexed text
db.products.find({$text: {$search:"awesome"}})
# lookinf for a frase
db.products.find({$text: {$search:"\"red book\""}})

# Text Indexes and Sorting

# special field score will hold calculated matching factor and can be used for sorting

db.products.find({$text: {$search:"awesome t-shirt"}}, {score: {$meta: "textScore"}}).pretty()


db.products.find({$text: {$search:"awesome t-shirt"}}, {score: {$meta: "textScore"}}).sort({score:{$meta: "textScore"}}).pretty()

# creating combined text indexes
db.products.getIndexes()

db.products.dropIndex("description_text")

db.products.createIndex({title: "text", description: "text"})

db.products.insertOne({title: "A Ship", description: "Floats perfectly"})

db.products.find({$text: {$search:"ship"}})

# excluding words
# records with  t-shirt will be excluded
db.products.find({$text: {$search:"awesome -t-shirt"}})

```

##### Setting the default Language & Using Weights

```bash
use contactData
db.products.getIndexes()

db.products.dropIndex("title_text_description_text")

# weights change score calculation
db.products.createIndex({title: "text", description: "text"}, {default_language: "english", weights:{title: 1, description: 10}})

db.products.find({$text: {$search:"", $language: "german"}})
db.products.find({$text: {$search:"", $caseSensitive: "true"}})
db.products.find({$text: {$search:"red"}}, {score:{$meta: "textScore"}}).pretty()
```

##### Building Indexes

- Foreground
  - Collection is locked during index creation
  - Faster
- Background
  - Collection  is accessible during index creation
  - Slower 

```bash
mongo credit-rating.js

mongo
use credit
show collections

db.ratings.findOne()

# blocks the collection
db.ratings.createIndex({age:1})

db.ratings.explain("executionStats").find({age: {$gt: 80}})
db.ratings.dropIndex({age:1})
# not blocking
db.ratings.createIndex({age:1}, {background: true})
```

##### Useful Resources & Links

- More on partialFilterExpressions: <https://docs.mongodb.com/manual/core/index-partial/>
- Supported default_languages: <https://docs.mongodb.com/manual/reference/text-search-languages/#text-search-languages>
- How to use different languages in the same index: <https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/#create-a-text-index-for-a-collection-in-multiple-languages>

#### Working with Geospatial Data

##### Adding GeoJSON Data

```bash
use awesomeplaces

# db.<COLLECTION>.insertOne({name: "<LOCATION_NAME>", location: {type: "<LOC_TYPE>", coordinates: [<LONG>, <LAT>]}})

db.places.insertOne({name: "California Academy of Sciences", location: {type: "Point", coordinates: [-122.4724356, 37.7672544]}})

db.places.findOne()
```

##### Geo Queries, Indexes

```bash
use awesomeplaces

db.places.find({location: {$near: {$geometry: {type: "Point", coordinates: [-122.471114, 37.771104]}}}})

Error: error: {
	"ok" : 0,
	"errmsg" : "error processing query: ns=awesomeplaces.placesTree: GEONEAR  field=location maxdist=1.79769e+308 isNearSphere=0\nSort: {}\nProj: {}\n planner returned error: unable to find index for $geoNear query",
	"code" : 2,
	"codeName" : "BadValue"
}
# 2dsphere index
db.places.createIndex({location: "2dsphere"})

db.places.find({location: {$near: {$geometry: {type: "Point", coordinates: [-122.471114, 37.771104]}}}})

db.places.find({location: {$near: {$geometry: {type: "Point", coordinates: [-122.471114, 37.771104]},$maxDistance: 500, $minDistance: 10}}}).pretty()

db.places.insertOne({name: "Conservatory of Flowers", location: {type: "Point", coordinates: [-122.4615748, 37.7701756]}})

db.places.insertOne({name: "Golden Gate Tennis Park", location: {type: "Point", coordinates: [-122.4593702, 37.7705046]}})

db.places.insertOne({name: "Nopa", location: {type: "Point", coordinates: [-122.4389058, 37.7747415]}})

# Find places inside a certain area
const p1 = [-122.4547, 37.77473]
const p2 = [-122.45303, 37.76641]
const p3 = [-122.51026, 37.76411]
const p4 = [-122.51088, 37.77131]
db.places.find({location: {$geoWithin: {$geometry: {type: "Polygon", coordinates: [[p1, p2, p3, p4, p1]] }}}}).pretty()

# Find out if a user is inside a Specific Area

db.areas.insertOne({name: "Golden Gate Park", area: {type: "Polygon", coordinates: [[p1, p2, p3, p4, p1]]}})

db.areas.findOne()

db.areas.createIndex({area: "2dsphere"})

db.areas.find({area: {$geoIntersects: {$geometry: {type: "Point",coordinates: [-122.49089, 37.76992]}}}})

db.areas.find({area: {$geoIntersects: {$geometry: {type: "Point",coordinates: [-122.48446, 37.77776]}}}})

# Find places within a certain Radius 
db.places.find().pretty()

# updating California Academy of Sciences
db.places.updateOne({_id: ObjectId("5d3f42b9b66cd42c40dcd32d")}, {$set: {location: {type: "Point", coordinates: [-122.46636, 37.77014]}}})

db.places.find({location: {$geoWithin: {$centerSphere: [[-122.46203, 37.77286], 1 / 6378.1]}}}).pretty()

```

##### Assignment 6 - Geospatial Data

- Pick 3 Points on Google Maps and store them in a collection
- Pick a point and find the nearest points within a min and max distance
- Pick an area and see which points  (that are stored in your collection) it contains
- Store at least one area in a different collection
- Pick a point and find out which areas in your collection contain that point

```bash
db.places.insertOne({name: "Beergarden", location: {type: "Point", coordinates: [11.59228, 48.15203]}})

db.places.insertOne({name: "Oktoberfest", location: {type: "Point", coordinates: [11.54965, 48.13203]}})

db.places.insertOne({name: "My old Place", location: {type: "Point", coordinates: [11.56934, 48.15105]}})

const myLocation = [11.59475, 48.14235]

db.places.createIndex({loc: "2dsphere"})


db.places.find({location: {$near: {$geometry: {type: "Point", coordinates: myLocation}}}}).pretty()

db.places.find({location: {$near: {$geometry: {type: "Point", coordinates: myLocation}, $minDistance: 1000, $maxDistance:2000}}}).pretty()

const p1 = [11.6097, 48.14522]
const p2 = [11.57142, 48.15416]
const p3 = [11.6, 48.15954]

const polygonArea = [[p1, p2, p3, p1]]
const polygonObject = {type: "Polygon", coordinates: polygonArea} 

db.places.find({location: {$geoWithin: {$geometry: polygonObject }}}).pretty()

db.areas.insertOne({name: "Free Time Well-Being Area", a: polygonObject })

db.areas.find({a: {$geoIntersects: {$geometry: {type: "Point",coordinates: [11.59228, 48.15203]}}}})

db.areas.createIndex({a: "2dsphere"})

```



##### Useful Resources & Links

- Official Geospatial Docs: <https://docs.mongodb.com/manual/geospatial-queries/>
- Geospatial Query Operators: <https://docs.mongodb.com/manual/reference/operator/query-geospatial/>

#### Aggregation Framework

```bash
mongoimport persons.json -d analytics -c persons --jsonArray

use analytics
# match stage
db.persons.aggregate([{$match: {gender:"female"}}]).pretty()

# group stage
## _id - by which fields we want to group
## group accumulates data
## state, totalPersons new fields only on the report
db.persons.aggregate([
    {$match: {gender: 'female'}},
    {$group: {_id: { state: "$location.state" }, totalPersons: { $sum: 1 } }}
]).pretty()

# sort stage
## each stahe can operate only on results from previous stage 

db.persons.aggregate([
    { $match: {gender: 'female'}},
    { $group: {_id: { state: "$location.state" }, totalPersons: { $sum: 1 } }},
    { $sort: { totalPersons: -1 }}
]).pretty()
```

##### Assignment 7

- Find  persons older than 50
- Group them by gender. How many persons you have per gender and average age they have ($avg).
- Order output by total by genders

````bash
use analytics

db.persons.aggregate([{ $match: {"dob.age": {$gt: 50}},
{ $group: {_id: { gender: $gender }, age: { $avg: 1 } }},
{ $sort: { totalPersons: -1 }}
]).pretty()

````

