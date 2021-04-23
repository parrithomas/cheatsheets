# JOI Schema

Joi is an npm package that lets us run server-side validation

## Install JOI

    // CLI
    npm i joi

    // index.js (or equiv)
    const Joi = require('joi')

## Basic validation schema

    // definte the Schema
    const skategroundSchema = Joi.object({
        skateground: Joi.object({
            title: Joi.string().required(),
            price: Joi.number().required().min(0),
            location: Joi.string().required(),
            description: Joi.string().required(),
            image: Joi.string().required()
        }).required()
    });

    // Take the schema, run the .validate method and pass in the req.body
    // destructure the result and save the returned 'error' object
    const { error } = skategroundSchema.validate(req.body);


    // if there is an error, throw a new ExpressError and pass in our msg and an error status
    if (error) {
        // map over error.details array, save the .message for each element and join with a comma
        const msg = error.details.map(el => el.message).join(',');
        throw new ExpressError(msg, 400)
    }

## Setup Joi Validation as middleware

Create a middleware function. Here it is called `validateSkateground`

    const validateSkateground = ((req, res, next) => {
        const skategroundSchema = Joi.object({
            skateground: Joi.object({
                title: Joi.string().required(),
                price: Joi.number().required().min(0),
                location: Joi.string().required(),
                description: Joi.string().required(),
                image: Joi.string().required()
            }).required()
        });
        const { error } = skategroundSchema.validate(req.body);
        if (error) {
            const msg = error.details.map(el => el.message).join(',');
            throw new ExpressError(msg, 400)
        } else {
            next()
        }
    })

**NOTE - Be sure to pass `next()` or the middleware stack will end here!**

In the route that needs validation, pass the function as a middleware:

    app.post('/skategrounds', validateSkateground, asyncWrapper(async (req, res, next) => {
        const skateground = new Skateground(req.body.skateground)
        await skateground.save();
        res.redirect(`/skategrounds/${skateground._id}`)
    }))

Now the code won't run unless the middleware validation passes the Joi schema and goes to `next()`

## Saving schema in a separate file

We can save all our schema in a separate file and then call them into our middleware function

### schemas.js

    const Joi = require('joi');

    module.exports.skategroundSchema = Joi.object({
        skateground: Joi.object({
            title: Joi.string().required(),
            price: Joi.number().required().min(0),
            location: Joi.string().required(),
            description: Joi.string().required(),
            image: Joi.string().required()
        }).required()
    });

### index.js

    // require the schema. Destructuring means we can call many from the same file on one line
    const { skategroundSchema } = require('./schemas.js')

    //.......//

    // we can now call our scheme in our middleware
    const validateSkateground = ((req, res, next) => {
    const { error } = skategroundSchema.validate(req.body);
    if (error) {
        const msg = error.details.map(el => el.message).join(',');
        throw new ExpressError(msg, 400)
    } else {
        next()
    }

})
