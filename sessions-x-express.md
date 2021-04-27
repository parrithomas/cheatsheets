# Sessions in Express

## What are sessions

It is not practical or safe to store large amounts of data in cookies.

Sessions are a server-side data store. Instead of storing data using cookies, we send the browser a cookie that can be used to retrieve the data.

### Example - shopping website

- The server saves all the items stored in a user's shopping cart
- The server sends the _session ID_ associated with that cart as a cookie to the brower.
- No cart info is stored locally in the brower.
- The next time the browser goes to the site, it can send the key stored into the cookie to the server to unlock the shopping cart data

## How to implement sessions in Express

Install `express-session` package with nmp

    `npm install express-sessions`

Setup in index.js...

    const session = require('express-session');

    // clear deprications errors with sessionOptions
    const sessionOptions = { resave: false, saveUninitialized: false }
    // fire up express session and pass a secret
    app.use(session({ secret: 'thisisnotagoodsecret', sessionOptions }));

We can now add anything we want to the property `res.session`.
How to create a simple view count...

    app.get('/pageviews', (req, res) => {
        if (req.session.count) {
            req.session.count += 1;
        } else {
            req.session.count = 1;
        }
        res.send(`You have viewed this page ${req.session.count} times`);
    })

The cookie is sent to the browser every time the we load the page
Everytime we send the cookie it looks a the session and finds .count
Then we use JS to add one to the value of .count

### Example with a VERY basic template for saving a logged in user's name

Save the value passed into a query string of 'username' into a variable  
Set `req.session.username` to equal the username value  
Redirect to `/greet`

    app.get('/register', (req, res) => {
        const { username = 'anon' } = req.query;
        req.session.username = username;
        res.redirect('/greet');
    })

Pass in this as a url...

    http://localhost:3000/register?username=Skywalker

Destructure the username out of `req.session` and save it to a variable.  
Use the variable to pass the value back into a string tamplate literal

    app.get('/greet', (req, res) => {
        const { username } = req.session;
        res.send(`Hello, ${username}. How are you today?`)
    })

This value is now saved so that even if the user closes the browser, they will see their name the next time they visit the url.
