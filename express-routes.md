# Express Routes

## Creating routes in a separate folder

#### Where: _/routes/dogs_

First set up express and define a variable for router

    const express = require('express');
    const router = express.Router();

Now create all your routes on this router variable

    router.get('/', (req, res) => {
        res.send('We are dogs')
    })
    router.post('/add', (req, res) => {
        res.send('create a new dog')
    })
    router.get('/:id', (req, res) => {
        res.send('i am a dog')
    })
    router.delete('/:id', (req, res) => {
        res.send('delete a dog')
    })
    router.get('/:id/edit', (req, res) => {
        res.send('Edit a dog')
    })

    // and so on

Then export the module for use elsewhere...

    module.exports = router;

## Importing and using the routes

#### Where: _index.js_ (or equivalent)

First import the module from _/routes/dogs_

    const dogRoutes = require('./routes/dogs')

Now `app.use` the imported module.
First parameter is the / url appended to each route.
Second paramter is the module we just imported.

    app.use('/dogs', dogRoutes)

The routes defined in _dogs.js_ can now visited at:

- _localhost/dogs_ (root route)
- _localhose/dogs/add_
- _localhose/dogs/dogId6789_
- _localhose/dogs/dogId6789/edit_

## Setup Middleware

Middleware can be setup on specific routes directly in the routes.js file using `router.use`.

#### Where: _/routes/admin.js_ (or any route-defining .js file)

    // setup express
    const express = require('express');
    const router = express.Router();

    // define middleware
    router.use((req, res, next) => {
        if (req.query.isAdmin) {
            next();
        }
        res.send('sorry not an admin')
    })

    // define routes
    router.get('/topsecret', (req, res) => {
        res.send('this is top secret');
    })

    router.get('/deleteall', (req, res) => {
        res.send('delete errrrything');
    })

    // export routes
    module.exports = router;
