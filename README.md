# express-postgresql-join-insert-api 🚀

<p>This project demonstrates a Node.js Express REST API that performs atomic database operations using PostgreSQL transactions. 
The API inserts data into two related tables (employe and staff) using a single transaction to ensure data consistency. Input validation is handled using express-validator.</P>



## 📂 Project Structure
```js
Express-postgresql-insert-api
│
├── db.js # PostgreSQL connection pool
├── insert.js # Express server & API routes
├── package.json
└── README.md
```
## ⚙️ Configure the PostgreSQL connection pool (`db.js`)
```js
const { Pool } = require('pg');
const pool = new Pool({
   user: 'your_username',
   host: 'localhost',
   database: 'your_database_name',
   password: 'YOUR_PASSWORD',
   port: 5432,                     // Default PostgreSQL port

  });
  
  module.exports = pool;
```
## Create Your Express Server
Create a file: insert.js
```js
// insert.js
const express = require('express');
const { body, validationResult } = require('express-validator');
const pool = require('./db');
const bodyParser = require('body-parser');
const app = express();
const PORT = 2000;

app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', '*'); // Or specify your origin
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.setHeader('Access-Control-Allow-Credentials', true); // If needed
  next();
});

// Middleware to parse JSON
app.use(bodyParser.json());

app.get('/',(req, res) => {
    res.send(`<h1>EXPRESS JS API</h1>`);
});
//---------------------------------with bodyParser--------------------------------------
// Define a route to query the database
app.post('/employe-staff', async (req, res) => {
  const client = await pool.connect();

  try {
    const { name, mobile, age, city } = req.body;

     // Start a database transaction temporary(काम शुरू)
    await client.query('BEGIN');

    //  Employe insert
    const empResult = await client.query('INSERT INTO employe(name, mobile) VALUES($1,$2) RETURNING id',[name, mobile]);

    const empId = empResult.rows[0].id;

    //  Staff insert (same id)
    await client.query('INSERT INTO staff(id, age, city) VALUES($1,$2,$3)',[empId, age, city]);
  
    // Save all changes permanently and end the transaction(काम पक्का)
    await client.query('COMMIT');

    res.status(201).json({
      message: 'Employe & Staff inserted successfully',
      id: empId
    });

  } catch (err) {
    await client.query('ROLLBACK');    // Release the client back to the pool
    console.error(err);
    res.status(500).send('Transaction failed');
  } finally {
    client.release();
  }
});

 //------------------------------with validation-----------------------------
// Define a route to query the database
app.post('/employe-staffs',

  // ✅ Validation rules
  [
    body('name').notEmpty().withMessage('name is required'),
    body('mobile').notEmpty().withMessage('mobile is required').isLength({ min: 10, max: 10 }).withMessage('mobile must be 10 digits'),
    body('age').notEmpty().withMessage('age is required').isInt({ min: 18 }).withMessage('age must be 18 or above'),
    body('city').notEmpty().withMessage('city is required')
  ],

  async (req, res) => {
    //  validation error handle
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        status: 'validation_error',
        errors: errors.array()
      });
    }
    // Obtain a dedicated client for running a transaction(single connection)
    const client = await pool.connect();

    try {
      const { name, mobile, age, city } = req.body;
      
      // Start a database transaction temporary(काम शुरू)
      await client.query('BEGIN');

      //  Insert into employe
      const empResult = await client.query('INSERT INTO employe(name, mobile) VALUES ($1, $2) RETURNING id',[name, mobile]);
        const empId = empResult.rows[0].id;

      //  Insert into staff
      await client.query('INSERT INTO staff(id, age, city) VALUES ($1, $2, $3)',[empId, age, city]);

       // Save all changes permanently and end the transaction(काम पक्का)
      await client.query('COMMIT');

      res.status(201).json({
        status: 'success',
        message: 'Employe & Staff inserted successfully',
        id: empId
      });

    } catch (err) {
      await client.query('ROLLBACK');   // Roll back the transaction if any error occurs( काम cancel)
      console.error(err);
      res.status(500).json({
        status: 'error',
        message: 'Transaction failed'
      });
    } finally {
      client.release();   // Release the client back to the pool
    }
  }
);
 // Start the server
  app.listen(PORT,() => {
    console.log(`Server running on http://localhost:${PORT}`);
  });  
  ```
## 🚀 API Endpoints
### 🔹 Test API
```js
GET /
```
#### Response:
```js
<h1>EXPRESS JS API</h1>
```
### 🔹 Add Employee (Without Validation)
- POST /employe-staffs

#### Request Body:
```js

  {
    "name": "Ramu",
    "mobile": "9876543210",
    "age": "21",
    "city": "Raipur"
}

```
#### Response:
```js
{
  "message": "Employe & Staff inserted successfully",
  "id": 14
}
```
### 🔹 Add Employee (With Validation)
- POST /employe-staffs

#### Request Body:
```js

  {
    "name": "Ramu",
    "age": "21",
    "city": "Raipur"
}
```
### Validation Rules:
- mobile must not be empty
- name must not be empty
- Error Response Example:
```js
{
  "errors": [
    {
      "msg": "mobile is required",
      "param": "mobile",
      "location": "body"
    }
  ]
}  
 ```
  ## 🎯 Learning Outcomes
- Handling REST APIs using Express.js
- Working with PostgreSQL using Node.js
- Performing database transactions (BEGIN, COMMIT, ROLLBACK)
- Inserting data into multiple related tables safely
- Using express-validator for request validation
- Managing database connections with connection pooling
- Building real-world backend API structure
  
## 👨‍💻 Author

- Kumlesh Kurre
- Backend Developer
- Skills: Express.js | Node.js | PostgreSQL | REST APIs
 
## ⭐ Support
If you like this project, please ⭐ star the repository to support my work!
  
  
  
  
  
  
