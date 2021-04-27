# Cookies in Express

http://expressjs.com/en/5x/api.html#res.cookie

## Sending cookies

Use `res.cookie(name, value, {options})` on a chosen route

    app.get('/setname', (req, res) => {
        res.cookie('name', 'Stevie');
        res.cookie('animal', 'kittens!!!!!!')
        res.send('You got a cookie!')
    })

## Using cookies

Express needs `cookie-parser` installed to be able to work with cookies.

    npm i cookie-parser

Require it

    const cookieParser = require('cookieParser')

Use it

    app.use(cookieParser());

Cookie values are now available via `req.cookies`

    app.get('/greet', (req, res) => {
        const { name = 'Buddy', animal = 'one' } = req.cookies;
        res.send(`Hello ${name}. You are a very cute ${animal}`);
    })

The above destructures the `name` and `animal` cookies. If they don't exist defauls are set (Buddy and one).

Cookie values are then passed into a string template literal.

## Signing cookies

Signing cookies is a way to verify if a cookie has been tampered with.

Set a cookie signature...

    app.use(cookieParser('thisismysecretcode'))

Pass a `signed: true` option when setting the cookie

    app.get('/getsignedcookie', (req, res) => {
        res.cookie('fruit', 'banana', { signed: true })
        res.send('Fruit is great')
    })

To get the values from signed cookies with `req.signedCookies'...

    app.get('/verifyfruit', (req, res) => {
        const {fruit} = req.signedCookies;
    })

# HMAC
**"Hash-based message authentication code"**

The goal is to take a cookie (or anything that as been digitally signed) 