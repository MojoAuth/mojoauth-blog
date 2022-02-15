---
title: "Hashing a Password in NodeJs"
date: 2022-02-14T00:00:00+05:30
coverImage: "hashing-passwords-in-nodejs.jpg"
author: "Puneet Singh"
tags: ["Tutorial"]
description: "In this article, we will explore different hashing functions used to store a password in NodeJs."
---

We must not save users' passwords in the database's plain text in any application as it poses a severe security threat. We should always save a password in the hashed format. But not all hashing functions are suitable to store passwords. Popular hashing functions like sha1 and md5 are not good for passwords.

We will learn how to hash a password in NodeJs using better-suited functions such as `Bcrypt`, `PBKDF2`, and `Argon2`.

Pre-requisites:

- Basic understanding of HTML/JavaScript
- NodeJs installed in your system.
- Access to a test MongoDB Database.

## Password hashing using Bcrypt

Bcrypt function was designed by [Niels Provos] and [David Mazières] based on the [Blowfish cipher] in 1999.

Brcrypt contains a salt to protect against [rainbow table attacks](https://en.wikipedia.org/wiki/Rainbow_table). Also, it can make the hashing process slower by iteration count, so it remains resistant to brute-force attacks even with increasing computation power.

Let's create a node.js project to use in our application, create the project with the below file structure.

```
models
  --User.js
controllers
  --users.js
index.js
```

First, in `index.js` we used the `express` package to create a REST server and used the `mongoose `package to connect to a test database.

```JS
/* root index.js */

var express = require('express');
var mongoose = require('mongoose')
var cors = require('cors')

// Sample mongodb URI
const PORT = 3000;
const DB_URI = 'mongodb://127.0.0.1:27017/dbname';

// Create Database Connection
mongoose.connect(DB_URI);

var usersRouter = require('./controllers/users')

var app = express();
app.use(express.json());
app.use(cors())

app.use('/users/',usersRouter)

app.listen(PORT, function (err) {
  if (err) console.log(err);
  console.log("Server listening on PORT", PORT);
});

```

Then we will create a User Model to save the user data in the database. This User.js file will also contain two methods, `createHash` and `validatePassword`, which will contain the logic to generate password hash and validate it.

```JS
/* models/User.js */
const mongoose = require("mongoose");
const bcrypt = require("bcrypt");

// UserSchema
const UserSchema = mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
  },
  password_hash: {
    type: String,
    required: true,
  },
});


// Method to generate a hash from plain text
UserSchema.methods.createHash = async function (plainTextPassword) {

  // Hashing user's salt and password with 10 iterations,
  const saltRounds = 10;

  // First method to generate a salt and then create hash
  const salt = await bcrypt.genSalt(saltRounds);
  return await bcrypt.hash(plainTextPassword, salt);

  // Second mehtod - Or we can create salt and hash in a single method also
  // return await bcrypt.hash(plainTextPassword, saltRounds);
};

// Validating the candidate password with stored hash and hash function
UserSchema.methods.validatePassword = async function (candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password_hash);
};


module.exports.User = mongoose.model("User", UserSchema);

```

In `controllers/users.js` we will define two routes `/users/signup` and `/users/signin`.

```JS
/* controllers/users.js */

var express = require("express");
var router = express.Router();
var asyncHandler = require("express-async-handler");

const User = require("../models/User").User;

router.post(
  "/signup",
  asyncHandler(async (req, res) => {
    // Put some validation related to
    // email validation and strong password rules

    // Initialize newUser object with request data
    const newUser = new User({
      name: req.body.name,
      email: req.body.email
    });

    var hashedPassword = await newUser.createHash(req.body.password);
    newUser.password_hash = hashedPassword;

    // Save newUser object to database
    await newUser.save();

    return res.status(201).json({
      message: "User created successfully.",
    });
  })
);

router.post(
  "/signin",
  asyncHandler(async (req, res) => {
    // Find user with requested email
    let user = await User.findOne({ email: req.body.email });

    if (user === null) {
      return res.status(400).json({
        message: "User not found.",
      });
    } else {
      if (await user.validatePassword(req.body.password)) {
        return res.status(200).json({
          message: "User Successfully Logged In",
        });
      } else {
        return res.status(400).json({
          message: "Incorrect Password",
        });
      }
    }
  })
);

module.exports = router;

```

> Note: Above code is not production-ready and is only used to explain the password hashing functions syntax.

Once we are done with the above code, we need to install all the dependencies using npm and run the nodejs server using the below command.

```
node index.js
```

Once the server is running on our local machines, we can create a new user by submitting a post request on - `http://localhost:3000/users/signup` and getting a success message. It will save the password of the registered user in a hashed format.

```json
// Signup Post Request
{
  "name": "Ajay Sharma",
  "email": "ajaysharma@mail7.io",
  "password": "choosestrongpassword"
}
```

```json
// Response of signup API
{
  "message": "User successfully registered."
}
```

After creating the user, we can try the signin API - `http://localhost:3000/users/signin` and get the below response if we use the correct password.

```json
// Signin Post Request
{
  "email": "ajaysharma@mail7.io",
  "password": "choosestrongpassword"
}
```

```json
// Response of signin API
{
  "message": "User Successfully Logged In."
}
```

## Password hashing in NodeJs using PBKDF2

Password-Based Key Derivation Function 2 (PBKDF2) uses graphics processing units (GPUs) computation while creating the hash, it makes the [brute-force attack](https://en.wikipedia.org/wiki/Brute-force_attack) costly.

And we will also use a dynamic salt which will save our passwords from [Rainbow table attack](https://en.wikipedia.org/wiki/Rainbow_table)

To implement PBKDF2 function, we just need to update the `createHash` and `validatePassword` methods in `models/Users.js`. Here we are using the `pbkdf2` npm package.

```JS
/* models/Users.js */

//...
const pbkdf2 = require("pbkdf2");
const crypto = require("crypto");

// UserSchema
//...

// Method to create pbkdf2 hash from plain text
UserSchema.methods.createHash = async function (plainTextPassword) {

  // Generate a salt and then create hash
  const salt = crypto.randomBytes(16).toString("hex");
  const hashedPassword = pbkdf2
    .pbkdf2Sync(plainTextPassword, salt, 10, 32, "sha512")
    .toString("hex");

  // Saving both the dynamic salt and hash in the Database
  return [salt, hashedPassword].join("#");
};

// Validating the password with pbkdf2 hash
UserSchema.methods.validatePassword = async function (candidatePassword) {
  const hashedPassword = this.password_hash.split("#")[1];
  const salt = this.password_hash.split("#")[0];

  const hash = pbkdf2
    .pbkdf2Sync(candidatePassword, salt, 10, 32, "sha512")
    .toString("hex");

  if (hash === hashedPassword) {
    return true;
  }
  return false;
};

module.exports.User = mongoose.model("User", UserSchema);
```

## Password hashing in NodeJs using Argon2

Argon2 is the newest hashing algorithm out of the mentioned three. It emerged as the winner of the [Password Hashing Competition](https://en.wikipedia.org/wiki/Password_Hashing_Competition) in 2015.

It is the recommended first choice for passwords by [The Open Web Application Security Project](https://owasp.deteact.com/cheat/cheatsheets/Password_Storage_Cheat_Sheet.html) after that PBKDF2 and Bcrypt are the following choices.

We use the `argon2` npm package to create a password hash and validate it in the code below.

```JS
/* models/Users.js */
...
const argon2 = require("argon2");

// UserSchema
...

// Method to generate Hash from plain text  using argon2
UserSchema.methods.createHash = async function (plainTextPassword) {
    // return password hash
    return await argon2.hash(plainTextPassword);
};

// Method to validate the entered password using argon2
UserSchema.methods.validatePassword = async function (candidatePassword) {
  return await argon2.verify(this.password_hash, candidatePassword)
};

module.exports.User = mongoose.model("User", UserSchema);
```

# Conclusion

As per the [verizon Data Breach Investigations Report](https://www.verizon.com/business/resources/reports/dbir/), passwords are responsible for 81% of the data breaches. A strong hashing algorithm is recommended to save the passwords in the database. It can save you from a lot of trouble.

We at MojoAauth want to take application security one more step ahead. Give a try to our [Passwordless Authentication](https://mojoauth.com/blog/passwordless-is-future/) solution which provide a secure and hassle free authentication experience to your customer.
