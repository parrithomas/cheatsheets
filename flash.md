# Flash

Use flash to display - or 'flash' - messages to a user on screen that disapear.

## Setup

Install `connect-flash`

    npm i connect-flash

Install `express-session`

    npm i express-session

Require `connect-flash` and `express-session`

    const flash = require('connect-flash');
    const session = require('express-session');

Setup as middleware...

    app.use(session({ secret: 'mysecrectpasssword', resave: true, saveUninitialized: true }))
    app.use(flash())

## Useage

All requests now have a `req.flash()` function  
It takes two parameters: a key and a function  
It is defined before the redirect...

    app.post('/farms', async (req, res) => {
        const farm = new Farm(req.body)
        await farm.save();
        req.flash('success', 'Successfully made a new farm!');
        res.redirect(`/farms`);
    })

We can pass `req.flash('success')` into the `res.render` of the route that the redirect points to.

The value passed in is whatever we defined as our key value.

Continuing this example withthe redirect to _/farms_...

    app.get('/farms', async (req, res) => {
        const farms = await Farm.find({});
        res.render('farms/index', { farms, messages: req.flash('success') })
    })

If we now use `<%= messages %> (or whatever we defined it as) in an ejs file it will display the value attached to our key.

# res.locals

We can pass 'messages' (as our example), to _every_ route by defining a middleware on the `res.locals` object.

    app.use((req, res, next) => {
        res.locals.messages = req.flash('success');
        next();
    })

- We have defined `res.locals.messages` (it can me .anything you want)
- Set it equal to `req.flash('success')`, as set on our route
- ALL our routes now pass in `messages` and we have access using `<%= messages %>`
