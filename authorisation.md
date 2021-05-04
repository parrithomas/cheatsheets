# Authorisation

Once a user is logged in (authenticated) was can **authorise** them to do certain things - or not.

## SETUP

First we need to create the association in the database schema...

    const SkategroundSchema = new Schema({
        title: { type: String, required: true },
        image: { type: String, required: true },
        price: { type: Number, required: true },
        description: { type: String, required: true },
        location: { type: String, required: true },
        reviews: [
            {
                type: Schema.Types.ObjectId,
                ref: 'Review'
            }
        ],
        author: {
            type: Schema.Types.ObjectId,
            ref: 'User'
        }
    })

Above, in our Skateground Schema we have added `author` and used `type: Schema.Types.ObjectId` with `ref: User`, to link the author to our User model.

We can need to save id of the logged in `User` to the `Author` when we create a new `Skateground`.

We do this by adding `skateground.author = req.user._id;` after we have created our `new Skateground`

_Remember: Passport has created a `user` model on `req`, from which we can pull the `._id`_

    router.post('/', loginCheck, validateSkateground, asyncWrapper(async (req, res, next) => {
        const skateground = new Skateground(req.body.skateground)
        skateground.author = req.user._id;
        await skateground.save();
        req.flash('success', '🛹 New Skateground created! 🛹');
        res.redirect(`/skategrounds/${skateground._id}`)

    }))

## Using the user info

We can now pass chain on another `.populate()` when pulling an entry from the database to have access to all the information from the author object.

    router.get('/:id', asyncWrapper(async (req, res, next) => {
        const skateground = await Skateground.findById(req.params.id).populate('reviews').populate('author')
        if (!skateground) {
            req.flash('error', `Couldn't find the Skateground you're looking for ¯\_(ツ)_/¯`);
            return res.redirect('/skategrounds');
        }
        res.render('skategrounds/show', { skateground })
    }))

## Authorising tasks with the user info

We have two pieces of information we can work with in our ejs:

- `currentUser`
  - defined in our global middleware (`res.locals.currentUser = req.user;`) it holds info/object of the current user
- `skateground.user`
  - passed in as an option on our route `{ skateground }`

With this we can see if one `equals` the other to check if something belongs to the user currently logged in...

    if (currentUser && skateground.author.equals(currentUser._id)){
        // code here runs if currentUser equals the author
    }

Above explained:

- `currentUser` returns `true` or `false` to check if the current user is logged in or not. If false the code doesn't run
- If it is true AND...
- The author of this page `.equals()` the `._id` attached to `currentUser` we run some code.
- We check with `&&` so the code doesn't break if no one is logged in
