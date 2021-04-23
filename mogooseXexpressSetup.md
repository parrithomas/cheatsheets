# Mongoose X Express Setup CheatSheet

## index.js

    // SETUP DEPENDENCIES
    const express = require('express');
    const app = express();
    const path = require('path');
    const mongoose = require('mongoose');
    // import model from ./models
    const ModelName = require('./models/modelFile')

    // SETUP MONGOOSE
    mongoose.connect('mongodb://localhost:27017/skateGround', { useNewUrlParser: true, useUnifiedTopology: true, useCreateIndex: true });

    // is mongoose on?
    const db = mongoose.connection;
    db.on('error', console.error.bind(console, "connection error:"));
    db.once('open', () => {
    console.log('DATABASE IS CONNECTED');
    })

    // SETUP EJS AND MIDDLEWARE
    app.set('views', path.join(__dirname, 'views'));
    app.set('view engine', 'ejs');

    // Parse DB data
    app.use(express.urlencoded({ extended: true }))

## Setup Model

### /models/modelName.js

    // SETUP MONGOOSE
    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;

    const mySchema = new Schema({
        title: String,
        price: String,
        description: String,
        location: String
    })

    module.exports = mogoose.model('ModelName', SkategroundSchema);

## Seed a database

### /seeds/seedData.js

    const seedData = [
        {
            name: 'Aubergine',
            price: 0.59,
            category: 'vegetable'
        },
        {
            name: 'Pineapple',
            price: 0.99,
            category: 'fruit'
        },
        // ...
    ]

    module.exports = seedData;

### /seeds/seeds.js

    const mongoose = require('mongoose');
    const Product = require('../models/product');

    mongoose.connect('mongodb://localhost:27017/breadShop', {     useNewUrlParser: true, useUnifiedTopology: true, useCreateIndex: true     });

    const db = mongoose.connection;
    db.on('error', console.error.bind(console, "connection error:"));
    db.once('open', () => {
        console.log('DATABASE IS CONNECTED');
    })

    const seedDB = async () => {
    await Product.deleteMany({});
    Product.insertMany(seedData)
        .then(data => {
            console.log(data)
        })
        .catch(err => {
            console.log(err)
        });
    }

    seedDB().then(() => {
        mongoose.connection.close()
    })

## CRUD API

### READ - Home route

_Gets all entires from mongoDB collection (`Product.find({})`), renders index.ejs, passes in all products_

    app.get('/', async (req, res) => {
        const products = await Product.find({});
        res.render('index', { products });
    })

### READ - Individual item route

    app.get('/product/:id', async (req, res) => {
    const product = await Product.findById(req.params.id);
    res.render('product', { product });
    })

- `req.params.id` to grab ID from the url/route
- pass this into `Product.findById()`. Returns the individual entry in the DB and we save to `product` variable.
- `res.render` our _product.ejs_ file and pass in the saved `product` variable

### CREATE

_NOTE: make sure `app.use(express.urlencoded({ extended: true }))` middleware is enable to parse the `req.body` in the `post` route_

    // CREATE - individual product page
    app.get('/new', (req, res) => {
        res.render('new');
    })

    // CREATE - post route
    app.post('/product', async (req, res) => {
        const product = await new Product(req.body);
        product.save();
        res.redirect(`/product/${product._id}`)
    })

The html page...

    <form action="/product" method="POST">
        <div>
            <label for="name">Name:</label>
            <input type="text" name="name" placeholder="Product name" />
        </div>
        <div>
            <label for="price">Price:</label>
            <input type="number" name="price" placeholder="Price" />
        </div>
        <div>
            <label for="category">category:</label>
            <input type="text" name="category" placeholder="Product category" />
        </div>
        <button type="submit">Add new product</button>
    </form>

_NOTE: the post route and the form action are the same location_

### UPDATE (patch and put)

`patch` to update one value // `put` to update all the values  
Be sure to have `method-override` installed and setup in the routes file

    const methodOverride = require('method-override');
    app.use(methodOverride('_method'))

    // UPDATE - edit form page
    app.get('/product/:id/edit', async (req, res) => {
        const product = await Product.findById(req.params.id);
        res.render('edit', { product });
    })

    // UPDATE - PUT route
    app.put('/product/:id', async (req, res) => {
        const { id } = req.params;
        const product = await Product.findByIdAndUpdate(id, req.body)
        res.redirect(`/product/${product._id}`);
    })

The HTML page...

    <form action="/product/<%= product._id %>?_method=PUT" method="POST">
        <div>
            <label for="name">Name:</label>
            <input type="text" name="name" value="<%=product.name%>" />
        </div>
        <div>
            <label for="price">Price:</label>
            <input type="number" name="price" value="<%=product.price%>" />
        </div>
        <div>
            <label for="category">category:</label>
            <input type="text" name="category" value="<%=product.category%>" />
        </div>
        <button type="submit">Edit product</button>
    </form>

### DELETE route

    app.delete('/product/:id', async (req, res) => {
        const { id } = req.params;
        await Product.findByIdAndDelete(id);
        res.redirect('/');
    })

The HTML

    <form action="/product/<%= product._id %>?_method=DELETE" method="post">
        <button type="submit">Delete</button>
    </form>

_In order to 'submit' the delete we have to wrap our button in a form so we have somewhere to set the `action` and the `method`_
