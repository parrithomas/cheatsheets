# Authentication

## Authentication vs Authorisation

- **Authentication** is veryifying WHO a particular user is
- **Authorisation** is verifying WHAT a particular user is _authorised_ to do

## Authenticating users
We don't store passwords as text -- run them through a hashing function and store THAT in the database.

### Cryptographic hashing functions

Characteristics of a cyrptographic hashing function:
- A ONE-WAY function that produces a result that can NOT be reversed
- A small change in input will yield a large change in the output
- The same input always produces the same output
- Almost impossible that two different inputs will produce the same output
- Password hash functions are deliberately slow

### Salts - The extra safeguard

- There are only a few hashing functions appropriate for encrypting passwords. eg. bcrypt  
- Salts are an extra step taken to further safeguard a password when using a hashing function.
- They are a random value added to the password before it is hashed.
- This way two users with the same password will create a different hashed value.

## Bcrypt

Bcrypt is a password hashing function that can be used with Javascript.

Install bcrypt...

    npm i bcrypt

Require bcrypt...

    const bcrypt = require('bcrypt');

To hash a password we create a function that takes a param (ie. the password string)  

    const hashPassword = async (pw) => {
        const salt = await bcrypt.genSalt(12)
        const hash = await bcrypt.hash(pw, salt)
    }

Above explained...
- We first create a hash using `bcrypt.genSalt()` and pass in 12 as the number of tries.
- We then hash the password using `bcrypt.hash()` and pass in the password and the salt

We can now compare a password...

    const login = async (pw, hashPw) => {
        const result = await bcrypt.compare(pw, hashPw);
        if (result) {
            console.log('logged you in')
        } else {
            console.log('try again')
        }
    }

    login('monkey', '5sf456sdf1dsf45s6f1321f456f')

Above explained...

- Create a function that takes a password and the hashed password to compare it to
- Use `bcrypt.compare()` and pass in the password and the hashed password. Save it to variable 'result'.
- If they DO NOT match result is falsy.
- As such we can run an if truthy/falsy if statement to work with the result.

Another option to hash and salt a password in one line...

    const hashPassword = async (pw) => {
        const hashedPw = await bcrypt.hash(pw, 12)
    }


## Setup auth routes

Make sure the following are installed, setup and required:
- bcrypt
- ejs
- express
- mongoose
- nodemon
- express-session

## Create a user

Create a model... 

    const mongoose = require('mongoose');
    const schema = mongoose.Schema;

    const userSchema = new schema({
        username: {
            type: String,
            required: [true, 'Username can not be blank']
        },
        password: {
            type: String,
            required: [true, 'Username can not be blank']
        }

    })

    const User = mongoose.model('User', userSchema)

    module.exports = User

Create a form with a post method...

    <form action="/register" method="post">
      <div>
        <label for="username">Username</label>
        <input type="text" name="username" id="username" placeholder="Username" />
      </div>
      <div>
        <label for="username">Password:</label>
        <input type="password" name="password" id="password" />
      </div>
      <button type="submit">Sign up</button>
    </form>

Create a get route to the form...

    app.get('/register', (req, res) => {
        res.render('register');
    })


Create a post route that passes the supplied password through the hashing function and saves a new user to the database...

    app.post('/register', async (req, res) => {
        const { username, password } = req.body;
        const hashed = await bcrypt.hash(password, 12);
        const user = new User({ username: username, password: hashed });
        await user.save();
        req.session.userId = User._id
        res.redirect('/');
    })

## How to create a login

Create a login form with a POST method...

Create a POST route...

    app.post('/login', async (req, res) => {
        const { username, password } = req.body;
        const user = await User.findOne({ username });
        const validPw = await bcrypt.compare(password, user.password)
        if (validPw) {
            res.redirect('/')
        } else {
            res.send('try again')
        }
    })

Above explained:
- save the username and password from the submitted form
- find and save the user from the db with `User.findOne()` passing in the saved username. This works as all usernames have to be unique.
- Validate the password with `bcrypt.compare()` passing in the `password` saved from the form and the `user.password` pulled from the database.
- If we have a valid password do one thing, if we don't do another.

## Staying logged in

We can save a user's login state to a session id.

The above modified to include `req.session` to save the user id to the session...

    app.post('/login', async (req, res) => {
        const { username, password } = req.body;
        const user = await User.findOne({ username });
        const validPw = await bcrypt.compare(password, user.password)
        if (validPw) {
            req.session.userId = user._id;
            res.redirect('/secret')
        } else {
            res.send('try again')
        }
    })

Above, the inclusion of `req.session.userId = user._id;` is all we need.

We can now use the ID saved to `req.session.userId` to create conditionals. The below page only renders is the user is logged in. ie. if `req.session.userId` equated to true...

    app.get('/secret', (req, res) => {
        if (!req.session.userId) {
            res.redirect('/login')
        } else {
            res.render('secret')
        }
        res.send('You can only see me if you are logged in')
    })

## Logging out

Logging a user out is as simple as setting `req.session.userId` to null.

    app.post('/logout', (req, res)=>{
        req.session.userId = null;
        res.redirect('/login');
    })

## Middleware to check if logged in

We can write a middleware to check login state on any route...

    const loginCheck = (req, res, next) => {
        if (!req.session.userId) {
            return res.redirect('/login')
        } next();
    }

And then pass that into our route as a middleware...
    app.get('/secret', loginCheck, (req, res) => {
        res.render('secret')
    })


## Refactoring to Model Methods

### Hashing a new user's password

We can hash a new user's password using a middleware on the schema using `.pre()` to run a function _before_ we call `.save()`.

In our _user.js_ file...

    userSchema.pre('save', async function(next){
        if (!this.isModified('password')) return next();
        this.password = await bcrypt.hash(this.password, 12);
        next()
    })

The above explained...
- create a function that runs before (`pre`) any time we call the `save()` method on `userSchema`
- pass in `next` as a parameter so we can call in. THIS IS MIDDLEWARE. WE **HAVE** TO CALL `next()` OR IT STALLS.
- Check if the field `password` has been modified. If is hasn't (ie. returns false), return `next()` and continue with the save. If it returns true (ie. it is new or has been changed)...
- Run `bcrypt.hash`:
  - pass in `this.password` (ie. the password our schema is about to save)
  - set hash number to 12
- save `this.password` = to our new value
- call `next()` (ie. continue to run the `save()` method)

On our user registration route we no longer need to hash the supplied password...

    app.post('/register', async (req, res) => {
        const { username, password } = req.body;
        const user = new User({ username, password });
        await user.save();
        req.session.userId = User._id
        res.redirect('/');
    })



### Validating the user's password

We can take the logic that checks the supplied password against the password stored on our user model, and save it as a method ON our user model.

In our _user.js_ we need to require bcrypt...

    const bcrypt = require('bcrypt');

Now create a static method on the schema...

    userSchema.statics.findAndValidate = async function (username, password) {
        const foundUser = await this.findOne({ username });
        const isValid = await bcrypt.compare(password, foundUser.password);
        return isValid ? foundUser : false;
    }

The above explained...
- Create a statics async function on the `userSchema` called `findAndValidate`
- The function takes two parameters: `username` and `password`
- Run `findOne()` on `this` (ie, our _User_) and pass in supplied username param. Save to variable `foundUser`
- Run `bcrypt.compare()` to compare the supplied password against the saved password.
  - pass in the password supplied in the function parameter
  - pass in `foundUser.password`
  - save the result to `isValid`
- `return` ternary operator... if `isValid` is true return `foundUser`, else return `false`

We can now run this function on any route where we want to check the supplied password...

    app.post('/login', async (req, res) => {
        const { username, password } = req.body;
        const foundUser = await User.findAndValidate(username, password);
        if (foundUser) {
            req.session.userId = foundUser._id;
            res.redirect('/secret')
        } else {
            res.send('try again')
        }
    })

Above explained...
- Grab the username and password from `req.body`
- run `User.findAndValidate()` and pass in the username and password we just fetched.
- The method will either return `foundUser` or `false`
- We can now run our usual if statement using foundUser if it exists or redirect if it doesn't.