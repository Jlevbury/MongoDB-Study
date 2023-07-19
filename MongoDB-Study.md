## MongoDB Introduction & Key Definitions

**MongoDB** is a source-available, document-oriented database designed for ease of development and scaling. It's part of the NoSQL family of database systems. Instead of storing data in tables as is done in a "classical" relational database, MongoDB stores structured data as JSON-like documents with dynamic schemas (MongoDB calls this BSON), making the integration of data in certain types of applications easier and faster.

Key Definitions:
- **Document**: A record in MongoDB, similar to a row in SQL, containing key-value pairs.
- **Collection**: The equivalent of an SQL table. It holds documents.
- **Database**: A container for collections, similar to an SQL database.
- **Schema**: MongoDB is a schema-less DB, i.e., it doesn't require a fixed structure like SQL databases.
- **BSON**: Binary representation of JSON-like documents. MongoDB uses this format under the hood.

  ## What is BSON?
BSON, which stands for Binary JSON, is a binary-encoded serialization of JSON-like documents. BSON is designed to be efficient in space, but also in scan-speed. It was developed by MongoDB for use in the MongoDB database, but its use is not restricted to MongoDB.

Like JSON, BSON supports the embedding of objects and arrays within other objects and arrays. However, BSON has additional data types not included in JSON, such as Date and binary data types.

BSON can be compared to binary interchange formats, like Protocol Buffers. BSON is more "schema-less" than Protocol Buffers, which can give it an advantage in flexibility.

Let's break down some key features of BSON:

1. **Additional Data Types**: BSON introduces additional data types such as int, long, date, floating point, and decimal128. These data types don't exist in pure JSON but are commonly used in programming languages, making BSON a better fit for developers.

2. **Efficiency**: BSON data format provides a binary representation of data structures which makes it very efficient to traverse.

3. **Size**: In some cases, BSON will take up more space than JSON due to the length prefixes and explicit array indices.

4. **Compatibility**: BSON is compatible with JSON and can be parsed to JSON and vice versa.

5. **Traversal**: BSON includes some (but not all) types present in BSON, making type-specific query possible. This is not possible with JSON.

6. **Use in MongoDB**: MongoDB uses BSON to organize, store, and manipulate its data. The MongoDB query language is designed to work directly on the BSON format for efficiency.

In conclusion, BSON is not a replacement for JSON. Rather, it's a powerful tool designed to enable efficiency where JSON’s human readability and ease-of-use come at the cost of performance.

## MongoDB and JavaScript

### Step 1: Installation
First, make sure you have MongoDB installed on your machine. You can download it from the official MongoDB website.

For integrating MongoDB with a JavaScript/Node.js project, the most popular choice is Mongoose, an Object Data Modeling (ODM) library that provides a straightforward, schema-based solution to model your application data.

To install it, use npm:
```bash
npm install mongoose
```

### Step 2: Connecting to MongoDB
To use MongoDB in your JavaScript file, first require the mongoose module and then connect to your database.

```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/my_database', {useNewUrlParser: true, useUnifiedTopology: true})
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could not connect to MongoDB...', err));
```
Replace `my_database` with the name of your database.

### Step 3: Creating a Model
Models are constructors compiled from Schema definitions. An instance of a model is a document.

```javascript
const courseSchema = new mongoose.Schema({
    name: String,
    author: String,
    tags: [String],
    date: { type: Date, default: Date.now },
    isPublished: Boolean
});

const Course = mongoose.model('Course', courseSchema);
```
This schema can be used to create new courses.

### Step 4: Querying Documents
To fetch documents from the database, you use queries.

```javascript
async function getCourses() {
    const courses = await Course
        .find({ author: 'AuthorName', isPublished: true })
        .limit(10)
        .sort({ name: 1 })
        .select({ name: 1, tags: 1 });
    console.log(courses);
}
getCourses();
```
This function fetches all courses authored by 'AuthorName' and that are published, limited to the top 10.

### Step 5: Updating Documents
Updating a document can be done in two ways: Query First (find the document, modify its properties, and then save it) or Update First (directly update the document in the DB and optionally get the updated document).

```javascript
async function updateCourse(id) {
    const course = await Course.findById(id);
    if (!course) return;

    course.isPublished = true;
    course.author = 'Another Author';
    
    const result = await course.save();
    console.log(result);
}
updateCourse('id');
```
This function updates the author of the document with the given id and sets 'isPublished' to true.
## MongoDB and JavaScript with Docker

### Step 1: Installation

First, make sure you have Docker installed on your machine. You can download it from the official Docker website.

Pull the MongoDB Docker image and run it:

```bash
docker pull mongo
docker run -d -p 27017-27019:27017-27019 --name mongodb mongo
```

This will start a MongoDB container with the name "mongodb" and expose its ports 27017-27019 to your host machine.

### Step 2: Connecting to MongoDB

After MongoDB is running in Docker, you can connect your application to it in the same way as the previous example, just make sure you're pointing to the correct URL. If you're running your JavaScript application on the same host as the Docker container, it should look something like this:

```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/my_database', {useNewUrlParser: true, useUnifiedTopology: true})
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could not connect to MongoDB...', err));
```

### Step 3: Creating a Model, Step 4: Querying Documents, Step 5: Updating Documents

These steps will be the same as in the previous version.

## Best Practices for MongoDB with Docker

1. **Persist MongoDB data**: By default, the data within your MongoDB container will be lost once the container is removed. To persist the data, use Docker volumes.
   Example:
   ```bash
   docker run -d -p 27017-27019:27017-27019 -v mongodbdata:/data/db --name mongodb mongo
   ```
   This command creates a volume named "mongodbdata" and mounts it to /data/db in the container. Now your MongoDB data will be stored in the "mongodbdata" volume and persist even when the container is removed.

2. **Configure MongoDB using environment variables**: When running the MongoDB Docker container, you can use environment variables to configure MongoDB.
   Example:
   ```bash
   docker run -d -p 27017-27019:27017-27019 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb mongo
   ```
   This command will create a MongoDB container with a root account that has a username of "admin" and a password of "password".

3. **Use Docker Compose for ease of use**: Docker Compose allows you to define your entire application, including services, networks, and volumes, in a single YAML file. This can make managing your application much easier.

4. Follow the same best practices as the non-Docker version for schema design, indexing, document size, normalization, denormalization, and BSON types.



## Best Practices for MongoDB


1. **Design your Schema according to user requirements**: 

MongoDB is schema-less, meaning that the database doesn't require any predefined schema set up by an admin. Data can be inserted without a pre-configured schema. That's a powerful feature but can lead to messy and inconsistent data if not handled correctly. Therefore, when creating your collections, it's a good practice to plan ahead and define your schema based on user requirements and how you plan to retrieve your data.

Here's an example of how a schema could be defined in mongoose:

```javascript
const userSchema = new mongoose.Schema({
    name: String,
    email: String,
    password: String,
    createdAt: { type: Date, default: Date.now },
    isActive: Boolean
});

const User = mongoose.model('User', userSchema);
```

2. **Use Indexing to improve search speed**:

Just like in other databases, indexing significantly improves search speed within MongoDB. Indexes are special data structures that store a small portion of the data set’s data in an easy-to-traverse form. The index stores the value of a specific field or set of fields, sorted by the value of the field as specified in the index.

Here's an example of creating an index in MongoDB:

```javascript
userSchema.index({ name: 1 });
```

In this example, `{ name: 1 }` means that this index sorts `name` in ascending order.

3. **Avoid Large Documents**:

The maximum BSON document size in MongoDB is 16 megabytes. You should strive to keep documents as small as possible, and avoid designs that would cause documents to grow unbounded.

If you have a document with an array of items and you're constantly pushing items into the array, consider splitting the data into multiple documents to avoid reaching the size limit.

4. **Normalize if you need to use data in many places**:

In MongoDB, it's good to separate data that you'll use across multiple collections, which is contrary to how you'd handle data in a SQL database. This process is called normalization. 

For example, if you're keeping track of orders and each order belongs to a user, you'd reference the user ID in the order document, and then make another query to the user collection when you need more data about the user.

```javascript
const orderSchema = new mongoose.Schema({
    product: String,
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User'
    },
    createdAt: { type: Date, default: Date.now },
});
```

5. **Denormalize for speed**:

Sometimes, you need to optimize your reads by including all the data you need in one query, rather than making multiple queries. This process is called denormalization. 

Here's an example:

```javascript
const blogSchema = new mongoose.Schema({
    title: String,
    author: {
        name: String,
        email: String
    },
    tags: [String]
});
```

Here, we included the author's name and email directly in the blog post document. This way, we can retrieve a blog post and its author's information in one go.

6. **Use the right BSON types**:

BSON, or Binary JSON, extends the JSON model to provide additional data types, ordered fields, and to be efficient for encoding and decoding within different languages. MongoDB uses BSON to represent document structures.

Using the correct BSON type can save a lot of space. For example, if a field is going to store a number between 0 and 100, you can use the `int` BSON type, which uses 4 bytes, instead of `double` (the default for numbers in JavaScript), which uses 8 bytes. Here's how you'd specify the type in mongoose:

```javascript
const productSchema = new mongoose.Schema({
    name: String,
    price: {
        type: Number,
        set: v => Math.round(v), // round to nearest integer
        get: v => Math.round(v)
    }
});
```

Here, `price` will still be a JavaScript `Number`, but mongoose will cast it to a 32-bit integer when saving to MongoDB.

Remember that these are guidelines, not hard and fast rules. Always consider your specific use case and benchmark different solutions if performance is a concern.

## Commonly Used MongoDB Commands


- `find()`: This command is used to retrieve documents in a database. It can take multiple parameters such as query objects and projection. A projection determines which fields are included or excluded in the query result.

    Example:
    ```javascript
    collection.find({isActive: true}, {name: 1, email: 1});
    ```
    In this example, we are querying for all documents where `isActive` is `true`, and we are only interested in `name` and `email` fields.

- `insert()`: This is used to insert documents into a MongoDB collection. You can pass an object to insert one document, or an array of objects to insert multiple documents.

    Example:
    ```javascript
    collection.insert({name: "John", email: "john@example.com", isActive: true});
    ```

- `update()`: This is used to update an existing document. You need to provide a filter object to determine which documents to update, and an update object which specifies the changes.

    Example:
    ```javascript
    collection.update({name: "John"}, {$set: {isActive: false}});
    ```
    This example will set `isActive` to `false` for all documents where `name` is "John".

- `remove()`: This command is used to delete documents from a collection. You have to provide a filter object to determine which documents to remove.

    Example:
    ```javascript
    collection.remove({isActive: false});
    ```
    This command will delete all documents where `isActive` is `false`.

- `createCollection()`: As the name suggests, this command is used to create a new collection.

    Example:
    ```javascript
    db.createCollection("users");
    ```
    This command will create a new collection named "users".

- `drop()`: This command is used to delete a collection.

    Example:
    ```javascript
    db.users.drop();
    ```
    This command will delete the "users" collection.

- `limit()`: This is used in conjunction with `find()` to limit the number of documents that the query returns.

    Example:
    ```javascript
    collection.find().limit(5);
    ```
    This will return only the first 5 documents.

- `sort()`: This command is used to sort the output of a query. You can sort the results in ascending (1) or descending (-1) order.

    Example:
    ```javascript
    collection.find().sort({name: 1});
    ```
    This command will return the documents sorted by the `name` field in ascending order.

Remember, with Mongoose, all these commands are methods called on your models (like Course.find(), Course.update(), etc.)

## Update Operators


 ### Fields

| Name | Description |
| --- | --- |
| `$currentDate` | Sets the value of a field to current date, either as a Date or a Timestamp. |
| `$inc` | Increments the value of the field by the specified amount. |
| `$min` | Only updates the field if the specified value is less than the existing field value. |
| `$max` | Only updates the field if the specified value is greater than the existing field value. |
| `$mul` | Multiplies the value of the field by the specified amount. |
| `$rename` | Renames a field. |
| `$set` | Sets the value of a field in a document. |
| `$setOnInsert` | Sets the value of a field if an update results in an insert of a document. Has no effect on update operations that modify existing documents. |
| `$unset` | Removes the specified field from a document. |

### Array Operators

| Name | Description |
| --- | --- |
| `$` | Acts as a placeholder to update the first element that matches the query condition. |
| `$[]` | Acts as a placeholder to update all elements in an array for the documents that match the query condition. |
| `$[<identifier>]` | Acts as a placeholder to update all elements that match the arrayFilters condition for the documents that match the query condition. |
| `$addToSet` | Adds elements to an array only if they do not already exist in the set. |
| `$pop` | Removes the first or last item of an array. |
| `$pull` | Removes all array elements that match a specified query. |
| `$push` | Adds an item to an array. |
| `$pullAll` | Removes all matching values from an array. |

### Modifiers

| Name | Description |
| --- | --- |
| `$each` | Modifies the `$push` and `$addToSet` operators to append multiple items for array updates. |
| `$position` | Modifies the `$push` operator to specify the position in the array to add elements. |
| `$slice` | Modifies the `$push` operator to limit the size of updated arrays. |
| `$sort` | Modifies the `$push` operator to reorder documents stored in an array. |

### Bitwise

| Name | Description |
| --- | --- |
| `$bit` | Performs bitwise AND, OR, and XOR updates of integer values. |


## Query Selectors

### Comparison
For comparison of different BSON type values, see the specified BSON comparison order.

| Name | Description |
| --- | --- |
| `$eq` | Matches values that are equal to a specified value. |
| `$gt` | Matches values that are greater than a specified value. |
| `$gte` | Matches values that are greater than or equal to a specified value. |
| `$in` | Matches any of the values specified in an array. |
| `$lt` | Matches values that are less than a specified value. |
| `$lte` | Matches values that are less than or equal to a specified value. |
| `$ne` | Matches all values that are not equal to a specified value. |
| `$nin` | Matches none of the values specified in an array. |

### Logical

| Name | Description |
| --- | --- |
| `$and` | Joins query clauses with a logical AND returns all documents that match the conditions of both clauses. |
| `$not` | Inverts the effect of a query expression and returns documents that do not match the query expression. |
| `$nor` | Joins query clauses with a logical NOR returns all documents that fail to match both clauses. |
| `$or` | Joins query clauses with a logical OR returns all documents that match the conditions of either clause. |

### Element

| Name | Description |
| --- | --- |
| `$exists` | Matches documents that have the specified field. |
| `$type` | Selects documents if a field is of the specified type. |

### Evaluation

| Name | Description |
| --- | --- |
| `$expr` | Allows use of aggregation expressions within the query language. |
| `$jsonSchema` | Validate documents against the given JSON Schema. |
| `$mod` | Performs a modulo operation on the value of a field and selects documents with a specified result. |
| `$regex` | Selects documents where values match a specified regular expression. |
| `$text` | Performs text search. |
| `$where` | Matches documents that satisfy a JavaScript expression. |

### Geospatial

| Name | Description |
| --- | --- |
| `$geoIntersects` | Selects geometries that intersect with a GeoJSON geometry. The 2dsphere index supports `$geoIntersects`. |
| `$geoWithin` | Selects geometries within a bounding GeoJSON geometry. The 2dsphere and 2d indexes support `$geoWithin`. |
| `$near` | Returns geospatial objects in proximity to a point. Requires a geospatial index. The 2dsphere and 2d indexes support `$near`. |
| `$nearSphere` | Returns geospatial objects in proximity to a point on a sphere. Requires a geospatial index. The 2dsphere and 2d indexes support `$nearSphere`. |

### Array

| Name | Description |
| --- | --- |
| `$all` | Matches arrays that contain all elements specified in the query. |
| `$elemMatch` | Selects documents if element in the array field matches all the specified `$elemMatch` conditions. |
| `$size` | Selects documents if the array field is a specified size. |

### Bitwise

| Name | Description |
| --- | --- |
| `$bitsAllClear` | Matches numeric or binary values in which a set of bit positions all have a value of 0. |
| `$bitsAllSet` | Matches numeric or binary values in which a set of bit positions all have a value of 1. |
| `$bitsAnyClear` | Matches numeric or binary values in which any bit from a set of bit positions has a value of 0. |
| `$bitsAnySet` | Matches numeric or binary values in which any bit from a set of bit positions has a value of 1. |

## Projection Operators

| Name | Description |
| --- | --- |
| `$` | Projects the first element in an array that matches the query condition. |
| `$elemMatch` | Projects the first element in an array that matches the specified `$elemMatch` condition. |
| `$meta` | Projects the document's score assigned during `$text` operation. |
| `$slice` | Limits the number of elements projected from an array. Supports skip and limit slices. |

## Miscellaneous Operators

| Name | Description |
| --- | --- |
| `$comment` | Adds a comment to a query predicate. |
| `$rand` | Generates a random float between 0 and 1. |

## MongoDB Compass using the MongoSH (MongoDB Shell).
For this example, we will use a database named "exampleDb" and a collection named "exampleCollection". 

1. **Creating a new collection**: 

   ```javascript
   use exampleDb
   db.createCollection('exampleCollection')
   ```
   This command creates a new collection named 'exampleCollection' inside the 'exampleDb' database. Replace 'exampleDb' and 'exampleCollection' with your actual database and collection names.

2. **Inserting documents into the collection**:

   ```javascript
   db.exampleCollection.insertMany([
       { name: 'John Doe', email: 'john@example.com', isActive: true },
       { name: 'Jane Doe', email: 'jane@example.com', isActive: false }
   ])
   ```
   This command inserts multiple documents into the 'exampleCollection'. You can replace the sample data in the documents with your own.

3. **Finding documents in a collection**:

   ```javascript
   db.exampleCollection.find({ isActive: true })
   ```
   This command fetches all the documents in the 'exampleCollection' where 'isActive' is true. You can replace 'isActive: true' with your own criteria.

4. **Updating documents in a collection**:

   ```javascript
   db.exampleCollection.updateOne({ name: 'John Doe' }, { $set: { isActive: false } })
   ```
   This command updates the first document in the 'exampleCollection' where the name is 'John Doe', setting 'isActive' to false. Replace the filter and update parameters with your own.

5. **Adding a field to documents in a collection**:

   ```javascript
   db.exampleCollection.updateMany({}, { $set: { createdAt: new Date() } })
   ```
   This command adds a new 'createdAt' field with the current date and time to all documents in the 'exampleCollection'. You can replace 'createdAt' and 'new Date()' with your own field and value.

6. **Deleting a field from documents in a collection**:

   ```javascript
   db.exampleCollection.updateMany({}, { $unset: { createdAt: "" } })
   ```
   This command removes the 'createdAt' field from all documents in the 'exampleCollection'. Replace 'createdAt' with the field you want to remove.

7. **Removing documents from a collection**:

   ```javascript
   db.exampleCollection.deleteOne({ name: 'Jane Doe' })
   ```
   This command removes the first document in the 'exampleCollection' where the name is 'Jane Doe'. Replace the filter with your own.

8. **Sorting documents in a collection**:

   ```javascript
   db.exampleCollection.find().sort({ name: 1 })
   ```
   This command fetches documents from the 'exampleCollection' sorted in ascending order by the name. Replace 'name: 1' with your own field and sort order (1 for ascending, -1 for descending).

9. **Deleting a collection**:

   ```javascript
   db.exampleCollection.drop()
   ```
   This command drops the 'exampleCollection' from the 'exampleDb' database.

Remember, replace 'exampleDb', 'exampleCollection', and other placeholders with your actual database, collection names, and data. Also, make sure to select the correct database from the dropdown next to the "Open Shell" button before running these commands.


## CRUD Operations using mongoDB
The given code is an Express application that connects to a MongoDB database using Mongoose, a MongoDB object modeling tool designed to work in an asynchronous environment. It provides routes for creating a new department, getting all departments, finding the 'Wine' department, and deleting a department by name. Here's the code with added comments to further explain the operations:

```javascript
// Importing necessary modules
const express = require('express');
const db = require('./config/connection');
const { Department } = require('./models');

// Setting up the port for the server to listen on
const PORT = process.env.PORT || 3001;

// Creating an instance of an Express server
const app = express();

// Middleware to parse incoming request bodies
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Creates a new department
app.post('/new-department/:department', (req, res) => {
  // Creating a new department document based on the Department model
  // The department name is passed as a URL parameter
  const newDepartment = new Department({ name: req.params.department });
  
  // Saving the newly created document to the database
  newDepartment.save();
  
  // Sending a response based on whether the document was saved or not
  if (newDepartment) {
    res.status(201).json(newDepartment);  // Sends back the saved document
  } else {
    console.log('Uh Oh, something went wrong');
    res.status(500).json({ error: 'Something went wrong' }); // Sends an error response
  }
});

// Finds all departments
app.get('/all-departments', async (req, res) => {
  try {
    // Fetching all documents in the 'Department' collection
    const result = await Department.find({});
    res.status(200).json(result);  // Sends back the fetched documents
  } catch (err) {
    console.log('Uh Oh, something went wrong');
    res.status(500).json({ error: 'Something went wrong' }); // Sends an error response
  }
});

// Finds the first matching document
app.get('/find-wine-department', async (req, res) => {
  try {
    // Fetches the first document that has the 'name' field as 'Wine'
    const result = await Department.findOne({ name: 'Wine' });
    res.status(200).json(result); // Sends back the fetched document
  } catch (err) {
    console.log('Uh Oh, something went wrong');
    res.status(500).json({ error: 'Something went wrong' }); // Sends an error response
  }
});

// Finds first document matching parameter and deletes
// For demo, use 'Wine' as URL param
app.delete('/find-one-delete/:departmentName', async (req, res) => {
  try {
    // Deletes the first document that matches the 'name' field with the provided URL parameter
    const result = await Department.findOneAndDelete({ name: req.params.departmentName });
    res.status(200).json(result); // Sends back the deleted document
    console.log(`Deleted: ${result}`);
  } catch (err) {
    console.log('Uh Oh, something went wrong');
    res.status(500).json({ error: 'Something went wrong' }); // Sends an error response
  }
});

// Connection to MongoDB
db.once('open', () => {
  // Starts the server to begin listening on PORT after establishing a database connection
  app.listen(PORT, () => {
    console.log(`API server running on port ${PORT}!`);
  });
});
```
Each of these routes is doing one of the CRUD operations: Create, Read, Update, and Delete. This is a good demonstration of how MongoDB integrates with Express and Node.js. 

Please note that the actual `./config/connection` and `./models` files are not provided, so you'll need to replace them with your actual connection configuration and models.

## CRUD concepts with MongoDB in an express application

1. **Create (Insert)**

    In the `/new-department/:department` route, a new department document is being created and saved to the MongoDB collection specified in the Department model.

    ```javascript
    app.post('/new-department/:department', (req, res) => {
      const newDepartment = new Department({ name: req.params.department }); // Create a new document based on the Department model
      newDepartment.save(); // Save the new document to MongoDB
      ...
    });
    ```
    The `new` keyword is used to instantiate a new Department object. `req.params.department` accesses the URL parameter `:department`, passed into the route as the name of the new department. `newDepartment.save()` is the operation to save the newly created document to the database. The dot notation is used to call the `save()` method on the `newDepartment` object. 

2. **Read (Select)**

    The `/all-departments` route queries for all documents in the MongoDB collection specified in the Department model.

    ```javascript
    app.get('/all-departments', async (req, res) => {
      try {
        const result = await Department.find({}); // Find all documents in the Department collection
        ...
      } catch (err) { ... }
    });
    ```
    `Department.find({})` fetches all documents in the 'Department' collection. The `{}` is an empty filter that matches all documents in the collection. The `await` keyword is used to wait for the database operation to complete before moving on to the next line of code.

    The `/find-wine-department` route fetches the first document in the Department collection that has the 'name' field as 'Wine'.

    ```javascript
    app.get('/find-wine-department', async (req, res) => {
      try {
        const result = await Department.findOne({ name: 'Wine' }); // Find the first document with 'name' as 'Wine'
        ...
      } catch (err) { ... }
    });
    ```
    `Department.findOne({ name: 'Wine' })` fetches the first document in the 'Department' collection that matches the filter `{ name: 'Wine' }`.

3. **Update**

    The provided code does not include an example of an update operation, but if you were to create a route to update the name of a department, it might look like this:

    ```javascript
    app.put('/update-department/:currentName/:newName', async (req, res) => {
      try {
        const result = await Department.findOneAndUpdate(
          { name: req.params.currentName }, 
          { $set: { name: req.params.newName } },
          { new: true }
        );
        ...
      } catch (err) { ... }
    });
    ```
    `Department.findOneAndUpdate(filter, update, options)` updates the first document that matches the filter and returns the updated document. The `{ new: true }` option is used to return the document after the update has been applied.

4. **Delete**

    The `/find-one-delete/:departmentName` route deletes the first document that matches the filter `{ name: req.params.departmentName }`.

    ```javascript
    app.delete('/find-one-delete/:departmentName', async (req, res) => {
      try {
        const result = await Department.findOneAndDelete({ name: req.params.departmentName }); // Delete the first document with 'name' as the URL parameter
        ...
      } catch (err) { ... }
    });
    ```
    `Department.findOneAndDelete(filter)` deletes the first document that matches the filter and returns the deleted document.

In all of these routes, "dot notation" is used to call methods on the `Department` model (`Department.find()`, `Department.findOne()`, `Department.findOneAndDelete()`) and instances of the model (`newDepartment.save()`). These methods are provided by Mongoose to interact with MongoDB.

## Model-View-Controller (MVC) Express.js application using MongoDB and Mongoose:

1. **Connection**

   In the MVC architecture, the MongoDB connection is typically established in a separate configuration or database file. In this example, it's done with Mongoose, which is a MongoDB Object Data Modeling (ODM) library. 

   ```javascript
   const mongoose = require('mongoose');
   
   mongoose.connect('mongodb://127.0.0.1:27017/mygroceryDB');
   
   module.exports = mongoose.connection;
   ```
   Here, the `mongoose.connect` function is used to connect to the MongoDB instance running locally on port 27017 and to the 'mygroceryDB' database. The established connection is then exported for use in other parts of the application.

2. **Model Schemas**

   In an MVC architecture, model schemas define the structure of data and the operations that can be performed on this data. In this example, an express application is set up to query all items in the `Item` model.

   ```javascript
   const express = require('express');
   const db = require('./config/connection');
   const { Item } = require('./models');
   ...
   ```
   After setting up the express server and requiring the necessary modules, the route `/all-items` uses the `find` method on the `Item` model to retrieve all documents in the MongoDB `Item` collection. This route corresponds to the "read" operation in CRUD.

3. **Model Instance methods**

   Mongoose schemas also allow you to define custom methods that will be available on all instances of a model. These methods can be used to encapsulate complex operations that need to be performed on a document. 

   ```javascript
   const mongoose = require('mongoose');

   const departmentSchema = new mongoose.Schema({
     name: { type: String, required: true },
     totalStock: Number,
     lastAccessed: { type: Date, default: Date.now },
   });

   departmentSchema.methods.getDocumentInfo = function () {
     console.log(`This department has the name ${this.name} and a total stock of ${this.totalStock}`);
   };

   const Department = mongoose.model('Department', departmentSchema);

   const produce = new Department({ name: 'Produce', totalStock: 100 });
   const responseGetInstance = produce.get('totalStock', String);
   console.log(`The value of the totalStock for this document in string form is ${responseGetInstance}`);

   produce.getDocumentInfo();

   module.exports = Department;
   ```
   Here, a `departmentSchema` is defined with `name`, `totalStock`, and `lastAccessed` fields. The `getDocumentInfo` method is defined to log a message to the console with the name and total stock of the department. A `Department` model is then created from the schema.

   An instance of the `Department` model (a document), `produce`, is created with a `name` of 'Produce' and a `totalStock` of 100. The built-in `get` method is used to get the value of the `totalStock` field. The custom `getDocumentInfo` method is then called on the `produce` document.

   This model can now be imported and used in the controller to interact with the `Department` collection in MongoDB.

This is a broad overview of how these elements fit together in an MVC architecture. Actual implementation may vary based on the specific requirements and structure of the application. For instance, the controller will typically contain logic to handle various HTTP requests (GET, POST, PUT, DELETE) and use the models to interact with the database. The view, which is not discussed here, would handle the presentation of the data to the user.

## Subdocuments overview and implementation

Subdocuments are documents embedded in other documents in MongoDB. They are useful when data does not require a new collection and can exist in a logically hierarchical manner. In Mongoose, you can create subdocuments by nesting schemas.

How to approach this process:

1. **Define a new schema named `bookSchema` for the subdocument.**
   
   In Mongoose, subdocuments are created by embedding a schema in another schema. So, the first step is to define a new schema for the `bookSchema`.

   ```javascript
   const bookSchema = new mongoose.Schema({
     title: String,
     price: Number,
   });
   ```

2. **The `books` subdocument is nested in the parent document.**

   To embed the `bookSchema` in the `librarySchema`, you can add it as an array property in the parent schema. This allows multiple books to be associated with a single library.

   ```javascript
   const librarySchema = new mongoose.Schema({
     name: { type: String, required: true },
     books: [bookSchema],
     lastAccessed: { type: Date, default: Date.now },
   });
   ```

3. **Create a model named `Library`.**

   After defining the schemas, you can create a Mongoose model for the `librarySchema`. 

   ```javascript
   const Library = mongoose.model('Library', librarySchema);
   ```

4. **Create an array of three books using the `bookSchema`.**

   As books are defined as an array in the `librarySchema`, you can add new books when creating a new library document.

   ```javascript
   const newLibrary = new Library({
     name: 'My Library',
     books: [
       { title: 'Book 1', price: 10 },
       { title: 'Book 2', price: 15 },
       { title: 'Book 3', price: 20 },
     ],
   });
   ```

5. **Create a new instance of the `Library` model which includes the `books` subdocument.**

   To create a new instance of the `Library` model, you use the `new` keyword along with the `Library` constructor, passing an object with the required data. Since books is an array of `bookSchema`, you pass an array of book data.

   ```javascript
   newLibrary.save((error, library) => {
     if (error) {
       console.log(error);
     } else {
       console.log(library);
     }
   });
   ```

6. **Test the `GET` route in Insomnia and the subdocuments are nested in the parent document.**

   Finally, you would want to verify your implementation. If you have a GET route setup in Express to fetch library data, you could use Insomnia to send a GET request to that route. The returned library data should display the `books` array with the nested `bookSchema` data.

   A GET route might look like this:

   ```javascript
   app.get('/libraries/:id', async (req, res) => {
     try {
       const library = await Library.findById(req.params.id);
       res.status(200).json(library);
     } catch (err) {
       res.status(500).send({ message: 'Internal Server Error' });
     }
   });
   ```
   
That's the high-level overview of how you could implement and test subdocuments in Mongoose and MongoDB.

## Aggregates 
MongoDB provides an `aggregate` function as a way to process data records and return computed results. The `aggregate` function groups the data from multiple documents together, and can perform a variety of operations on this grouped data to return a single result. 

The `aggregate` function can execute a pipeline of operations. The pipeline provides stages for transforming the documents. These stages are executed in the order they are defined, and each stage takes the output documents of the previous stage as its input.

**Aggregation pipelines explained**
In MongoDB, an aggregation pipeline is a multi-stage process that transforms the documents into an aggregated result. Each stage of the pipeline transforms the documents as they pass through. The concept is similar to a Unix/Linux pipeline where the output of each process (stage) is the input of the next.

Here is a brief description of some of the most commonly used pipeline stages:

1. **`$match`:** This stage is often used at the beginning of the pipeline to filter documents. Only documents that match the condition(s) are passed to the next stage. It's similar to the `WHERE` clause in SQL.

2. **`$group`:** This stage groups documents by a specified expression. The `_id` field in the `$group` stage represents the group by field. You can also calculate accumulated fields for the grouped documents, such as sum, average, min, max, etc.

3. **`$sort`:** This stage sorts documents in a specified order. It's similar to the `ORDER BY` clause in SQL.

4. **`$project`:** This stage selectively includes, excludes, or reshapes the document fields. It's similar to the `SELECT` statement in SQL, but also allows renaming, computed fields, and nested documents.

5. **`$limit`:** This stage limits the number of documents passed to the next stage. It's similar to the `LIMIT` keyword in SQL.

6. **`$skip`:** This stage skips a number of documents from the beginning of the input. It's often used with `$limit` for pagination.

7. **`$unwind`:** This stage deconstructs an array field from the input documents, outputting one document for each element in the array. It's useful when you want to flatten an array and access its elements as separate documents.

8. **`$lookup`:** This stage performs a left outer join with another collection. It's similar to the `LEFT JOIN` in SQL and allows you to combine data from multiple collections.

9. **`$addFields` or `$set`:** These stages add new fields to documents. The `$addFields` stage adds new fields while retaining all existing fields. The `$set` stage either adds new fields or replaces the existing fields with the same name.

10. **`$out`:** This stage writes the resulting documents of the pipeline to a specified collection. This can be useful when you want to save the result of an aggregation for later use. The `$out` stage should be the last stage of the pipeline.

Understanding how each of these stages works will allow you to transform your data in virtually any way you want with MongoDB's aggregation framework. However, keep in mind that while it's powerful, the aggregation framework can also be resource-intensive. It's usually a good idea to perform as much data reduction as possible in the early stages (like `$match`) to reduce the amount of data that needs to be processed in later stages.


For example, if we have a collection of orders and we want to find the total quantity of orders for each product, we might use an aggregation like this:

```javascript
const mongoose = require('mongoose');
const Order = mongoose.model('Order'); // Assuming Order model is already defined

Order.aggregate([
  {
    $group: {
      _id: '$product', // group by the product field
      totalQuantity: { $sum: '$quantity' }, // sum the quantity field
    },
  },
])
  .then(result => console.log(result))
  .catch(err => console.error(err));
```

In the above example, we first group the documents by the `product` field. In the `$group` stage, we're telling MongoDB to sum the `quantity` field for each group of documents. The `_id` field in the `$group` stage is mandatory and is used to determine how to group the documents. In this case, we're grouping by `product`, but this could be any field in your documents or a combination of fields.

You would typically use `aggregate` function in your route handlers, after receiving a request that requires some sort of data processing. 

Here's an example of how you might use it in an Express route:

```javascript
const express = require('express');
const router = express.Router();
const Order = require('../models/order'); // Assuming Order model is in ../models/order.js

router.get('/product-summary', async (req, res) => {
  try {
    const productSummary = await Order.aggregate([
      {
        $group: {
          _id: '$product',
          totalQuantity: { $sum: '$quantity' },
        },
      },
    ]);
    res.status(200).json(productSummary);
  } catch (err) {
    console.error(err);
    res.status(500).json({ message: 'Internal Server Error' });
  }
});

module.exports = router;
```
## Targeting values in aggregate operations. 
In this example, when a GET request is made to the `/product-summary` route, the server calculates the total quantity for each product and sends back the result as a JSON response.
```javascript
app.get('/all-books', async (req, res) => {
  try {
    const result = await Book.find({});
    res.status(200).json(result);
  } catch (err) {
    res.status(500).send(err);
  }
});

app.get('/sum-price', async (req, res) => {
  try {
    const result = await Book
      .aggregate([
        {
          $group: {
            _id: null,
            sum_price: { $sum: '$price' },
            avg_price: { $avg: '$price' },
            max_price: { $max: '$price' },
            min_price: { $min: '$price' },
          },
        },
      ]);
    res.status(200).json(result);
  } catch (err) {
    res.status(500).send(err);
  }
});
```

The code above creates an Express server that connects to MongoDB through Mongoose. It has two routes:

1. `/all-books`: returns all books from the database.
2. `/sum-price`: returns an aggregate of all book prices in the database, including sum, average, max, and min price.

There is an issue that the `/sum-price` route is returning aggregate information for all books, including those that are not in stock. The desired behavior is to have this route return aggregate information only for books that are in stock.

To filter out the books that are not in stock, you should use the `$match` stage before `$group` stage in your aggregation pipeline. The `$match` stage filters the documents to pass only the documents that match the specified condition(s) to the next pipeline stage.

Assuming that each book document has an `inStock` field that is `true` if the book is in stock and `false` otherwise, you can modify your code like this:

```javascript
app.get('/sum-price', async (req, res) => {
  try {
    const result = await Book
      .aggregate([
        {
          $match: { inStock: true }, // add this stage
        },
        {
          $group: {
            _id: null,
            sum_price: { $sum: '$price' },
            avg_price: { $avg: '$price' },
            max_price: { $max: '$price' },
            min_price: { $min: '$price' },
          },
        },
      ]);
    res.status(200).json(result);
  } catch (err) {
    res.status(500).send(err);
  }
});
```

Now, only the documents where `inStock` is `true` will be passed to the `$group` stage, and the returned aggregate information will be calculated based on these documents only.

In the future, when you are working with MongoDB aggregation and you encounter an issue where the returned data is not what you expect, consider the following:

1. **Check your pipeline stages**: Make sure each stage in your pipeline is working as expected. Test each stage independently if necessary.
2. **Understand your data**: Be familiar with the structure of your data, what fields exist and what their values mean. This can help you determine what conditions to use in the `$match` stage.
3. **Handle errors properly**: In your current code, if an error happens during aggregation, the error object is directly sent as a response. It's generally a good idea to not expose the raw error objects to the client as they might contain sensitive information. Instead, you could log the error and send a custom error message to the client.
