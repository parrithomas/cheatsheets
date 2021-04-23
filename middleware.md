# Middleware

## Intro
- Middleware are the building blocks of everything in Express
- Express Middleware are functions that run during the request/response 'lifecycle' 
- Each middleware has access to the *request* and the *response* objects
- The 'flow' looks a little like this:
  - `request` -> `middleware` -> `response`
- Middleware can be the end of the line by sending a method like `res.send()`
- Middleware can also CHAIN together, by calling a method like `next()`

## What can middleware do?
- Execute any code
- Make changes to the req and res objects
- End the req-res cycle
- Call the next middleware function in the stack

## Syntax
### `app.use(<code goes here>)`
Run code on every single API request

### `next()`
Our middleware function takes three parameters
 - request
 - response
 - next

If we don't pass and then call next, our middleware stack with STOP.  
We call `next()` like so...

    app.use((req, res, next) => {
        console.log(req.method.toUpperCase(), req.path);
        next();
    })

## Middleware on a specific route
    app.use('/dogs', (req, res, next) => {
        console.log('Dogs are swell');
        next();
    })

## Setup a 404 route
We can setup a middleware AT THE END OF OUR ROUTES to set up a 404. It will only trigger if none of the routes defined before it sent a response

    // ^ all our other routes...
    app.use((req, res)=>res.status(404).send('404 Page not found));

## Middleware called INSIDE a route

A function to check for a password passed as a query string...


    const verifyPassword = ((req, res, next) => {
        if (req.query.password) {
            if (req.query.password === 'letmein') {
                next();
            } else {
                res.send(`That is the WRONG password`)
            }
        } else {
            res.send('You need a password to access this page')
        }
    })

We can now call the function as middleware in our route


    app.get('/secret', verifyPassword, (req, res) => {
        res.send('Welcome to the underground page of secrets')
    })