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
Absolutely. Here's a more detailed explanation:

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
