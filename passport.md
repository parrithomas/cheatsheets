# Passport

## Installation and basic setup

Install passport

    npm i passport passport-local passport-local-mongoose

In our user model file...

    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;
    const passportLocalMongoose = require('passport-local-mongoose');

    const userSchema = new Schema({
        email: {
            type: String,
            required: true,
            unique: true
        },
    });

    userSchema.plugin(passportLocalMongoose);

    module.exports = mongoose.model('User', userSchema);

The above explained...

- We setup our userSchema but only defined `email`. Because...
- We then run `.plugin()` on our `userSchema` and pass in `passportLocalMongoose`. Which...
- Takes our schema and adds a username, id, password and salt for us.

## Configuration

In our _app.js_ (or equiv) first require `passport`, `passport-local` and the `User` model.

    const passport = require('passport');
    const LocalStrategy = require('passport-local');
    const User = require('./models/user');

Next setup the middleware. Setup AFTER `app.set(session())`...

    app.use(passport.initialize());
    app.use(passport.session());
    passport.use(new LocalStrategy(User.authenticate()));
    passport.serializeUser(User.serializeUser())
    passport.deserializeUser(User.deserializeUser())

`passport.use(new LocalStrategy(User.authenticate()));` explained...

- We are telling passport to use a new LocalStratey
- We are telling that to use our User model...
- and to use the `authenticate()` method... which has beed added automatically by `passportLocalMongoose`

`passport.serializeUser(User.serializeUser()) passport.serializeUser(User.deserializeUser())` explained...

- Telling passport how to 'store' and 'unstore' a user.

## New user setup

In _routes/users.js_

Setup our requires...

    const express = require('express'); // express
    const router = express.Router(); // express Router
    const User = require('../models/user'); // our User model
    const asyncWrapper = require('../utilities/asyncWrapper'); // our error wrapper
    const passport = require('passport') // passport

A GET route to a page with the registration form...

    router.get('/register', (req, res) => {
        res.render('users/register')
    })

A POST route to save the user...

    router.post('/register', asyncWrapper(async (req, res) => {
        try {
            const { username, email, password } = req.body;
            const user = new User({ username, email })
            const regUser = await User.register(user, password);
            req.flash('success', `Sign up successfull!`)
            res.redirect('/skategrounds');
        } catch (e) {
            req.flash('error', e.message);
            res.redirect('/register')
        }
    }))

The above explained...

- A try/catch so we can pass the user any error messages if an error is thrown. eg. 'User with that username already exists'
- pull the `username`, `email` and `password` from the `req.body`
- create a new `User` and pass in the `username` and `email`
- Use the `.register()` method build into passport on our `User` model and pass the `user` we just saved and the `password` saved from `req.body`
- Create a flash message to display with the redirect...
- Redirect to the chosen route
- If we catch an error, create a flash and pass in `e.message` to feedback to the user what went wrong.

## Logging in

To log a user in (ie. check their username and password against what is stored in the database) we use `passport`'s built-in middleware of `passport.authenticate()`.

Build a form that with field names of `username` and `password` and give it `post` method and an action that points to `/login` (or whever is appropriate)

Create a POST route in _routes/users.js_...

    router.post('/login', passport.authenticate('local', { failureFlash: true, failureRedirect: '/login' }), (async (req, res) => {
        req.flash('success', 'Sup')
        res.redirect('/skategrounds')
    }))

`passport.authenticate('local', { failureFlash: true, failureRedirect: '/login' })` explained...

- `local` - This is the `passport-strategy`. We need to set up separate routes for all login strategies we have installed (Google, Facebook, Twitter...)
- `failureFlash: true` - Flash an error message if login fails.
- `failureRedirect: '/login'` - Route to redirect to if the login fails

## Authentication Middleware

`passport` gives us a helper method to determine if a user is logged in or not.

`req.isAuthenticated()`

It either returns `true` or `false`.

With this we can restrict functionality of the site to only those who are logged in.

### Create a `checkLogin` middleware

In _utilites/middleware.js_

    module.exports.loginCheck = function (req, res, next) {
        if (!req.isAuthenticated()) {
            req.flash('error', 'You need to log in.')
            return res.redirect('/login');
        } next()
    }

Above explained...

- Create a function called `loginCheck`
- Pass the usual middleware params of `(req, res, next)`
- Check true/false value of `req.isAuthenticated()`
- If false, create a flash message and redirect
- Prepend with `module.exports.` to we can require it in other files

This can now be passed as middleware in any routes we want to protect.

In _/routes/skategrounds.js_  
Import the function...

    const { loginCheck } = require('../utilities/middleware')

In a route...

    router.get('/new', loginCheck, (req, res) => {
        res.render('skategrounds/new');
    })

## Logging out

Logging out is easy. We can just create a GET route that runs the method `req.logOut()`

    router.get('/logout', (req, res) => {
        req.logOut();
        req.flash('success', 'Logged out')
        res.redirect('/skategrounds')
    })

## Using `req.user` data

`passport` creates a `req.user` object for us.

It is going to be automatically filled in with the _deserialized_ information from the session.

If noone is signed in it returns `undefined`

### Setup a global reference to `req.user`

In _index.js_ (or _app.js_ or equiv) setup a global middleware with `res.locals`...

    app.use(req, res, next){
    res.locals.currentUser = req.user;
    }

We can now pass `<%= currentUser %>` in any .ejs file to access the info stored in `req.user`

### Show / hide login on nav bar

We can now use `currentUser` in `nav.ejs` to check if a user is logged in and display either Login or Logout...

    <% if(!currentUser){%>
        <a href="/login">Login</a>
    <% } else { %>
        <a href="/logout">Log out</a>
    <% } %>

## Log a user in automatically when registering

`Passport` puts a `.login()` method on the `req` object that, when run, will automatically log a user in.

As such, we can add it to our register route.

It is a call back function and must be written like this...

            req.login(newUser, err => {
            if (err) return next(err);
        });

The full register route looks like this...

    router.post('/register', asyncWrapper(async (req, res, next) => {
        try {
            const { username, email, password } = req.body;
            const user = new User({ username, email })
            const newUser = await User.register(user, password);
            req.login(newUser, err => {
                if (err) return next(err);
            });
            req.flash('success', `Nice one, ${req.user.username}, you're all signed up for SkateGrounds. Happy skating.`)
            res.redirect('/skategrounds');
        } catch (e) {
            req.flash('error', e.message);
            res.redirect('/register')
        }
    }))

## Using `req.originalUrl` to redirect users

When a user is asked to login we can keep track of where they were tyring to go, and redirect them there when they login.

We can access `req.originalUrl` on the request object.

We can save that to the session and then use that data.

Define the middleware...

    module.exports.loginCheck = function (req, res, next) {
        if (!req.isAuthenticated()) {
            req.session.returnTo = req.originalUrl
            req.flash('error', 'You need to log in.')
            return res.redirect('/login');
        } next()
    }

We can now use that session in the Login route to see where the user was trying to go.

    router.post('/login', passport.authenticate('local', { failureFlash: true, failureRedirect: '/login' }), (async (req, res) => {
        req.flash('success', `You're logged in, ${req.user.username}`)
        const redirectURL = req.session.returnTo || '/skategrounds'
        delete req.session.returnTo;
        res.redirect(redirectURL)
    }))

The above explained...

- first we use `passport` middleware to authenticate the user, as normal. If they are authenticated...
- Create the flash message
- Create a variable `redirectURL`...
- set it to what is stored in `req.session.returnTo` OR `'/skategrounds` (in case the user was just logging in via the login form).
- Delete `req.session.returnTo` as we don't need it any more
- Redirect to the path that is now saved into the `redirectURL` variable.
