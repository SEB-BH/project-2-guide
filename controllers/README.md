<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Controller Cheat Sheet</span>
</h1>

Controller actions handle requests.

They usually:

1. read information from `req`
2. use a Mongoose model
3. render a view or redirect

For Open House, listing controller actions are stored in:

```plaintext
controllers/listings.js
```

Create a controller file for each main resource

Examples:

```plaintext
controllers/books.js
controllers/events.js
controllers/recipes.js
controllers/products.js
```

## Basic setup

Import the model at the top:

```js
const Listing = require('../models/listing')
```

The model gives the application methods for working with a resource:

```js
Listing.create()
Listing.find()
Listing.findById()
Listing.findByIdAndUpdate()
Listing.findByIdAndDelete()
```

Export each action at the bottom as you write each function:

```js
module.exports = {
    showNewForm,
    create,
    ...
}
```

Import the controller into `server.js`:

```js
const listingsController = require('./controllers/listings')
```

Connect an action to a route:

```js
app.get('/listings/new', isSignedIn, listingsController.showNewForm)
```

## Main controller actions

| Controller action | Example model method          | Response                           | Useful notes                                     |
| ----------------- | ----------------------------- | ---------------------------------- | ------------------------------------------------ |
| `newListing`      | No database method needed     | Render `listings/new.ejs`          | Displays an empty form                           |
| `create`          | `Listing.create()`            | Redirect to `/listings`            | Add the signed-in user as the owner              |
| `index`           | `Listing.find()`              | Render `listings/index.ejs`        | Usually displays all resources                   |
| `show`            | `Listing.findById()`          | Render `listings/show.ejs`         | Use `populate()` when displaying referenced data |
| `edit`            | `Listing.findById()`          | Render `listings/edit.ejs`         | Check that the resource exists                   |
| `update`          | `Listing.findByIdAndUpdate()` | Redirect to the show or index page | Check ownership before updating                  |
| `deleteListing`   | `Listing.findByIdAndDelete()` | Redirect to `/listings`            | Check ownership before deleting                  |
| `favorite`        | `$push`                       | Redirect to the show or index page | Add the user to favorites                        |
| `unfavorite`      | `$pull`                       | Redirect to the show or index page | Remove the user from favorites                   |


## Useful filtered queries

Find resources owned by the signed-in user:

```js
const myListings = await Listing.find({
    owner: req.session.user._id
})
```

Find resources favorited by the signed-in user:

```js
const favoriteListings = await Listing.find({
    favoritedByUsers: req.session.user._id
})
```

## Embedded question (or comments, reviews, etc.) actions

Questions are stored inside a listing.

The usual pattern is:

```plaintext
Find the listing
        ↓
Find or change the question
        ↓
Save the listing
```

| Action          | Main pattern                    |
| --------------- | ------------------------------- |
| Create question | `.push()` and `.save()`         |
| Find question   | `.questions.id()`               |
| Update question | Change a property and `.save()` |
| Delete question | `.deleteOne()` and `.save()`    |

### Create

```js
const createQuestion = async (req, res) => {
// find the listing
    const foundListing = await Listing.findById(
        req.params.listingId
    )

// get the question data from the form
    const questionData = {
        text: req.body.text,
        author: req.session.user._id
    }

// add the question data to the found listing
    foundListing.questions.push(questionData)

// save the listing
    await foundListing.save()

    res.redirect(
        `/listings/${req.params.listingId}`
    )
}
```

### Find one embedded question

```js
// use the found listing to then find a specific question to update or delete
const foundQuestion =
    foundListing.questions.id(
        req.params.questionId
    )
```

### Update

```js
// update the question
foundQuestion.text = req.body.text

// save the listing (which also saves the question)
await foundListing.save()
```

### Delete

```js
// delete the question
foundQuestion.deleteOne()

// save the listing with the question now deleted
await foundListing.save()
```

Embedded questions do not use a separate `Question` model. Save the parent listing after making changes.