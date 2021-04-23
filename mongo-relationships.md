# Mongo Relationships

## One-to-few

Use nested arrays in the schema to create multiple values for one element...

    const userSchema = new mongoose.Schema({
        first: String,
        last: String,
        addresses: [
            {
                _id: { id: false }, // if we don't want each address to have its own id
                street: String,
                city: String,
                country: String,
                postcode: String,

            }
        ]
    })

    const User = mongoose.model('User', userSchema);

Above, the 'addresses' is an array of objects. Each object has a schema.  
Addresses can now accept multiple entires  
We can `.push()` on to the addresses array when creating a new user

    const makeUser = async () => {
        const u = new User({
            first: 'Garry',
            last: 'Trotter'
        })
        u.addresses.push({
            street: '45 Sausage Close',
            city: 'Manchester',
            country: 'UK',
            postcode: 'B4NG3R'
        })
        const res = await u.save()
        console.log(res)
    }

From here we can just keep adding (pushing) addresses to the same user

    const addAddress = async (id) => {
        const user = await User.findById(id);
        user.addresses.push({
            street: '2nd Home avenue',
            city: 'New York',
            country: 'USA',
            postcode: 'NYC44'
        })
        const res = await user.save()
        console.log(res);
    }

    addAddress('608169e4c1020b496e21dc59')

## One-to-many

Store infomation somewhere else and then reference that information

Shorthand `mongoose.Schema` to make things easier...

    const Schema = mongoose.Schema

Define the child Model...

    const productSchema = new Schema({
        name: String,
        price: Number,
        season: {
            type: String,
            enum: ['Spring', 'Summer', 'Autumn', 'Winter']
        }
    })

    const Product = mongoose.model('Product', productSchema);

Create some children...

    Product.insertMany([
        { name: 'Goddess Melon', price: 4.99, season: 'Summer' },
        { name: 'Hierloom tomatoes', price: 1.49, season: 'Summer' },
        { name: 'Golden Parsnips', price: 2.25, season: 'Winter' },
        { name: 'Hispi Cabbage', price: 3.45, season: 'Autumn' },
    ])

Define the Parent...

    const farmSchema = new Schema({
        name: String,
        city: String,
        products: [{ type: Schema.Types.ObjectId, ref: 'Product' }]
    })

    // products is pulling our child objects. It set to an type of ObjectId and we are referencing the 'Product' model

Make a new Parent and include a child...

    const makeFarm = async () => {
        const farm = new Farm({ name: 'Full Belly Farms', city: 'London' }) //  make a parent
        const parsnip = await Product.findOne({ name: 'Golden Parsnips' }) // find a child
        farm.products.push(parsnip) // add child to parent
        await farm.save(); // save
        console.log(farm); // print to console
    }
    makeFarm() // run the function

Add child to existing parent...

    const addProduct = async () => {
        const farm = await Farm.findOne({ name: 'Full Belly Farms' }) // find parent
        const toms = await Product.findOne({ name: 'Hierloom tomatoes' }) // find child
        farm.products.push(toms); // add child to parent
        await farm.save() // save parent
    }
    addProduct() // run function

## 'POPULATE' from ObjectIDs

Models have a `.populate()` method.  
It uses the `ref` option we passed when creating our parent Model:

    const farmSchema = new Schema({
        name: String,
        city: String,
        products: [{ type: Schema.Types.ObjectId, ref: 'Product' }]
    })

If we pass the schema key (ie. 'products') into `.populate()` then it will return the full object model, not just the object ID(s)...

    Farm.findOne({ name: 'Full Belly Farms' })
        .populate('products')
        .then(farm => console.log(farm))

Which returns...

    Mongo connected
    {
    products: [
        {
        _id: 60817ae69892fd49b2e318bb,
        name: 'Golden Parsnips',
        price: 2.25,
        season: 'Winter',
        __v: 0
        },
        {
        _id: 60817ae69892fd49b2e318ba,
        name: 'Hierloom tomatoes',
        price: 1.49,
        season: 'Summer',
        __v: 0
        }
    ],
    _id: 60817f0fd955a349ebe1be9c,
    name: 'Full Belly Farms',
    city: 'London',
    __v: 1
    }

Rather than...

    Farm.findOne({ name: 'Full Belly Farms' })
        .then(farm => console.log(farm))

Which returns...

    Mongo connected
    {
    products: [ 60817ae69892fd49b2e318bb, 60817ae69892fd49b2e318ba ],
    _id: 60817f0fd955a349ebe1be9c,
    name: 'Full Belly Farms',
    city: 'London',
    __v: 1
    }

## One to THOUSANDS AND MORE!

Using Twitter as an example, one user might have tens of thousands of tweets  
As such, in our database, we save the parent to the child  
ie. each tweet in the DB has one user, rather than each one user having thousands of tweets

Define the parent schema....

    const userSchema = new Schema({
        username: String,
        age: Number
    })

Definte the child schema...

    const tweetSchema = new Schema({
        text: String,
        likes: Number,
        reweets: Number,
        user: [{ type: Schema.Types.ObjectId, ref: 'User' }]
    })

Create the Models...

    const User = mongoose.model('User', userSchema);
    const Tweet = mongoose.model('Tweet', tweetSchema);

Create and save a user and some tweets...

    const makeTweets = async () => {
        const user = new User({ username: 'neo4life', age: '20' }); // create a user
        const tweet1 = new Tweet({ text: 'Do I take the red pill or blue pill?', likes: 0, retweets: 5 }); // create tweet 1
        const tweet2 = new Tweet({ text: 'Slo-mo is my jam!', likes: 10, retweets: 2 }); // create tweet 2
        tweet1.user = user; // add user to tweet1
        tweet2.user = user; // add user to tweet2
        await user.save();  // save user to db
        await tweet1.save(); // save tweet1 to db
        await tweet2.save(); // save tweet2 to db
    }
    makeTweets() // run the function

In our database both tweets have a user with the same ObjectId

    db.tweets.find().pretty()

    // returns...

        {
            "_id" : ObjectId("60818d0e0c50ec4aaf4dd153"),
            "user" : [
                    ObjectId("60818d0e0c50ec4aaf4dd152")
            ],
            "text" : "Do I take the red pill or blue pill?",
            "likes" : 0,
            "__v" : 0
    }
    {
            "_id" : ObjectId("60818d0e0c50ec4aaf4dd154"),
            "user" : [
                    ObjectId("60818d0e0c50ec4aaf4dd152")
            ],
            "text" : "Slo-mo is my jam!",
            "likes" : 10,
            "__v" : 0
    }

Find a tweet and see the user info with `.populate()`...

    const findTweet = async () => {
        const tweet = await Tweet.findOne({}).populate('user', 'username') // find the first tweet and populate 'user' and take just the 'username' value from it
        console.log(tweet); // log to console
    }
    findTweet();

Which returns...

    {
    user: [
        {
        _id: 60818eefbcce674b51021417,
        username: 'neo4life',
        age: 20,
        __v: 0
        }
    ],
    _id: 60818eefbcce674b51021418,
    text: 'Do I take the red pill or blue pill?',
    likes: 0,
    __v: 0
    }

## Mongo Schema Design

Sometimes it makes sense to duplicate (or 'denormalize') some data on both sides of a relationship.

Rules of thumb:

1. Favour embedding unless there is a compelling reason not to
2. Needing to access an object on its own is a compelling reason not to embed it
3. Avoid arrays that grow without limit. More than a couple hundred is too many: add the reference on the childside.
4. Structure your data in the way your app want to access it.

### Denormalisation example

In a Tasks app we have a Users collection and a Tasks collection.  
Each User can be assigned multiple Tasks.  
We want a page to display a User and their Tasks.  
We also a page to display all Tasks with their assigned Users.  
In this case it makes sense to save the Task to the User, and the User to the Task, so we can reference in either direction.
