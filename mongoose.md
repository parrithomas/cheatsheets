# Mongoose Cheatsheet

## Setup Mongoose

    // SETUP MONGOOSE
    mongoose.connect('mongodb://localhost:27017/skateGround', { useNewUrlParser: true, useUnifiedTopology: true, useCreateIndex: true });

## Check Mongoose is running

    // Is mongoose on?
    const db = mongoose.connection;
    db.on('error', console.error.bind(console, "connection error:"));
    db.once('open', () => {
        console.log('DATABASE IS CONNECTED');
    })

## Create new Schema

    const Schema = mongoose.Schema;

    const mySchema = new Schema({
        title: String,
        price: String,
        description: String,
        location: String
    })

## Create Model

    const ModelName = mongoose.model('ModelName', mySchema);

_Note: Model name is initial-cap and singular. MongoDB will convert collection name to plural. eg. 'Plant' becomes 'Plants' and 'Person' becomes 'People'_

## Create and save document

     const myVar = new ModelName({ title: 'lorem', price: 0,  description: 'ipsum', location:  'dolar' })
     myVar.save();

## MODEL METHODS

`find(criteria, [fields], [options], [callback])`: find document; callback has error and documents arguments
`count(criteria, [callback]))`: return a count; callback has error and count arguments
`findById(id, [fields], [options], [callback])`: return a single document by ID; callback has error and document arguments
`findByIdAndUpdate(id, [update], [options], [callback])`: executes MongoDB's findAndModify to update by ID
`findByIdAndRemove(id, [options], [callback])`: executes MongoDB's findAndModify to remove
`findOne(criteria, [fields], [options], [callback])`: return a single document; callback has error and document arguments
`findOneAndUpdate([criteria], [update], [options], [callback])`: executes MongoDB's findAndModify to update
`findOneAndRemove(id, [update], [options], [callback])`: executes MongoDB's findAndModify to remove
`update(criteria, update, [options], [callback])`: update documents; callback has error, and count arguments
`create(doc(s), [callback])`: create document object and save it to database; callback has error and doc(s) arguments
`remove(criteria, [callback])`: remove documents; callback has error argument

## DOCUMENT METHODS

`save([callback])`: save the document; callback has error, doc and count arguments
`set(path, val, [type], [options])`: set value on the doc's property
`get(path, [type])`: get the value
`isModified([path])`: check if the property has been modified
`populate([path], [callback])`: populate reference
`toJSON(options)`: get JSON from document
`validate(callback)`: validate the document`
