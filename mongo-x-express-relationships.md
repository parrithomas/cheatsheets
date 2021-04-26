# Mongo x Express Relationships

## Cross populate a new database entry

Farm model includes `products` in schema...

    const farmSchema = new Schema({
        name: {
            type: String,
            required: [true, 'Farm must have a name'],
        },
        city: {
            type: String
        },
        email: {
            type: String,
            required: [true, 'Email required']
        },
        products: [
            {
                type: Schema.Types.ObjectId,
                ref: 'Product'
            }
        ]
    })

Products model includes `farm` in schema...

    const productSchema = new mongoose.Schema({
        name: {
            type: String,
            required: true,
        },
        price: {
            type: Number,
            required: true,
        },
        category: {
            type: String,
            enum: ['fruit', 'vegetable', 'dairy'],
            lowercase: true,
        },
        farm: {
            type: Schema.Types.ObjectId,
            ref: 'Farm'
        }
    })

## Create new product attached to a farm

In *index.js*

The form page route...


    app.get('/farms/:id/products/new', async (req, res) => {
        const { id } = req.params;
        const farm = await Farm.findById(id)
        res.render('products/new', { categories, farm });
    })

The POST route...

    app.post('/farms/:id/products', async (req, res) => {
        const { id } = req.params; // get the farm id
        const farm = await Farm.findById(id); // find the farm
        const newProduct = new Product(req.body); // save the form info as newProduct
        farm.products.push(newProduct); // push newProduct onto the farm 
        newProduct.farm = farm; // set the farm on the newProduct
        await newProduct.save(); // save the product
        await farm.save() // save the farm
        res.redirect(`/farms/${id}`)
    })

## Get many-to-one results and display

In *index.js*...
Use `.populate()` and pass in the linked field from the Schema to pull in the details of each item (rather than just having access to their IDs)

    app.get('/farms/:id', async (req, res) => {
        const { id } = req.params;
        const farm = await Farm.findById(id).populate('products');
        res.render('farms/show', { farm });
    })

On rendered ejs pages now have access to `farm.products.name` etc.

    <ul>
      <% for (let product of farm.products){%>
      <li>
        <a href="/products/product/<%=product._id%>"><%= product.name %></a> -
        £<%= product.price %>
      </li>
      <%} %>
    </ul>

## Deleting linked data
To delete linked data you need to add a `.post` middleware to the Schema  

In `models/farm.js`...  
Require the Product model so it can be used in the middleware

    const Product = require('./product');

Before the model is defined... 

    farmSchema.post('findOneAndDelete', async function (farm) {
        if (farm.products.length) {
            await Product.deleteMany({ _id: { $in: farm.products } })
        }
    })

The above explained...
- `.post` rather than `.pre` as we need to after we have deleted the farm and, as such, have access to the farm's ID
- whenever `findOneAndDelete` is run on this model, run the function
- create a new async function that takes a param. We've called it farm
- Check to see if our farm has products (ie. does the products array have a lenght?)
- if it does run `DeleteMany` on the Product model
- We now have to find ONLY the products with IDs related to this farm so we pass parameters: select all products where the id is IN `farm.products`

Our delete route is as normal, as the middleware defined in the schema will trigger and delete any products associated to the farm...

    app.delete('/farms/:id', async (req, res) => {
        const farm = await Farm.findByIdAndDelete(req.params.id);
        res.redirect('/farms')
    })