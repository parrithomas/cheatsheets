# CONTROLLERS

A way to keep code neat is to use the 'MVC' pattern: **Model-View-Controller**

Each section of code is kept in its own file. We have three folders:

- Models - For defining how our database is to be structured
- Views - For storing the files that output our data to the user
- Controllers - For writing our functions which we can then call in our routes files

## Set up

- Create a _/controllers_ folder
- In _/controllers_ create a file for each set of routes. ie. `users.js`, `reviews.js` ...
- In each controller file, require any models that are needed

      const User = require('../models/user');

Take the function code from each route and assign it to a `modules.export.nameOfRouteHere` eg:

    module.exports.logout = (req, res) => {
    req.logOut();
    req.flash('success', 'Logged out')
    res.redirect('/skategrounds')
    }

Now in our `routes/users.js`. First we import the controller file and assign it to a variable...

    const users = require('../controllers/users);

We can then pass in `users` to each of our routes and simple call any method we have exported. ie, we can pass `users.logout` to call the `logout` function defined above.
router.get('/logout', users.logout);

## `router.route`

We can further clean code by using `router.route('/path')` to group all CRUD verbs on the same path...

    router.route('/register')
        .get(users.newUserForm)
        .post(asyncWrapper(users.createNewUser))
