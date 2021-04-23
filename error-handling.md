# Error Handling

## Common Error statuses
- 400: Bad request
- 401: Unknown user with no permission / authorisation
- 403: Known user with no permission / authorisation
- 404: Can not find something

- 500: 

## Define error handling
We define our error handling with **four** parameters in our Middleware

    app.use((err, req, res, next) => {
        res.status(500).send('OH DEAR)
        next(err)
    })

- If we pass `next()` it hands off to the next middleware
- We have to pass `next(err)` to hand off to the next ERROR HANDLING middleware

## Throwing an error in our functions

We can `throw` an error in our middleware functions with the following syntax:  

    throw new Error(<error status>,<message>)

A full function might look like this:

    const checkPassword = (req, res, next) => {
        if (req.query.password === 'letmein') {
            next();
        } else {
            throw new Error(401, 'Wrong password')
        }
    };

## Define our own Error Class

We can extend the build in Error class and make our own.

In a _appError.js_ file:

    class AppError extends Error {
        constructor(status, message) {
            super();
            this.status = status;
            this.message = message;
        }
    }
    module.exports = AppError;

Broken down:
- Create a new class called `AppError` that extends the existing `Error` method
- set the constructor to take two parameters: `status` and `message`
- call `super()` to include the parent's (`Error`) existing methods and properties
- set `status` to equal what was passed as the `status` parameter
- set `message` to equal what was passed as the `message` parameter
- export the module so we can require, then use, it in other files

In use:

    const checkPassword = (req, res, next) => {
        if (req.query.password === 'letmein') {
                next();
            } else {
                throw new AppError(401, 'Wrong password')
            }
    }

We can now set our error handler like this:

    app.use((err, req, res, next) => {
        const { status = 500 } = err
        const { message = 'Something went wrong' } = err;
        res.status(status).send(message)
    })

## Handling Async Errors

In async code we have to pass our errors into the `next` function

    app.get('/product/:id', async (req, res, next) => {
        const product = await Product.findById(req.params.id);
        if (!product) {
            return next(new AppError(404, 'Product not found'));
        }
        res.render('product', { product });
    })

- Above we first pass the `next` param into our `app.get`.  
- We then get the product and assign it to a the variable `product`  
- We then check to see if we have a product, if not, we pass the `new - AppError()` into `next()`  
- We `return` next so it stops all other code from running... ie. passing to the next error handler in the stack
- If we didn't the `res.render()` would still try and run and throw further errors


### ASYNC Dealing with Mongoose errors
We can catch our own errors and those thrown by Mongoose issues (ie. required fields not being populated) with `try` `catch`

    app.get('/product/:id', async (req, res, next) => {
        try {
            const product = await Product.findById(req.params.id);
            if (!product) {
                throw new AppError(404, 'Product not found');
            }
            res.render('product', { product });
        } catch (err) {
            next(err);
        }
    })

*NOTE: because we are using catch, we can `throw` our `AppError` as it will be caught by the `catch`*

## Define an Async Utility

We can define a function that we can pass the entire callback to. It's complicated

First we define this function

    function wrapAsync(fn) {
        return function (req, res, next) {
            fn(req, res, next).catch(e => next(e));
        }
    }

We than pass our async function INTO `wrapAsync()`
   
    app.get('/product/:id/edit', wrapAsync(async (req, res, next) => {
        const product = await Product.findById(req.params.id);
        if (!product) {
            throw new AppError(404, 'Your mum')
        }
        res.render('edit', { product });
    }))

## Differentiating Mongoose Errors
All Mongoose errors have a `name` property. We can see the name by setting up this error handling middleware

    app.use((err, req, res, next) => {
        console.log(err.name);
        next(err);
    })

As such we can set up handling for types of error. eg. a validation error...

A function that takes in the `err` object and returns a new `AppError` with a custom message

    const handleValidationErr = err => {
        return new AppError(400, `Validation Failed... ${err.message}`)
    }

Our error handler checking that runs if our error name = 'ValidationError'

    app.use((err, req, res, next) => {
        if (err.name === 'ValidationError') err = handleValidationErr(err);
        next(err);
    })

Above line-by-line:
- Basic setup passing all four params
- If our error is 'Validation Error' then...
- err is equal to our function with the err passed in
- we then pass `err` (ie. the function, into `next()`)