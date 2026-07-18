<h1>
  <span class="prefix"></span>
  <span class="headline">Project 2 Build Guide</span>
</h1>

## About

This guide explains the main parts of a MEN stack application:

* **MongoDB** stores the application data.
* **Express** receives requests and sends responses.
* **Node.js** runs the application.
* **EJS** creates the pages shown to the user.
* **Mongoose** connects the Express application to MongoDB.

You will begin your project by cloning the Session Auth Template. The template already includes user authentication, sessions, middleware, and the basic Express application structure.

You will then add your own resources and features.

For example, Open House had a `Listing` resource. A different project may have resources such as:

* books
* recipes
* events
* products
* support tickets
* study groups

Throughout this guide, we will use the Open House `Listing` resource as our main example.


## Content

| Lesson                                                                          | Skills                                                                        |
| ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [Setup](./setup/README.md)                                                      | Cloning the MEN AUTH repo                                                     |
| [Static Files and the `public` Folder](./static-files/README.md)                | Set up your `public` folder, CSS, and images                                  |
| [Middleware](./middleware/README.md)                                            | Protecting routes with middleware                                             |
| [Create a Navigation Partial](./nav-partial/README.md)                          | Create a nav partial to reuse on all of your pages                            |
| [Adding a Feature](./adding-a-feature/README.md)                                | MEN Stack architecture includes a view, a route, and a controller             |
| [MODELS](./activity-create-a-restful-routing-chart/README.md)    | Apply what you’ve learned about RESTful routing by designing a routing chart  |
| [Views](./views/README.md)                                                      | Conventions for creating views                                                |
| [Routes](./routes/README.md)                                                    | Conventions for building routes                                               |

## References

📖 [Reference Materials](./references/README.md)


**Find a 👾 bug 👾 or have suggestions? [Let us know](https://ga-curriculum.github.io/universal-resources-internal/module-feedback.html)!**



# Part 3: Controllers

A **controller action** contains the code that handles a request.

In this course structure, the resource's router and controller actions are stored in the same file.

For Open House, the listing routes and actions were stored in:

```plaintext
controllers/listings.js
```

Create a controller file for each main resource:

```bash
mkdir -p controllers
touch controllers/listings.js
```

Examples:

```plaintext
controllers/books.js
controllers/events.js
controllers/recipes.js
controllers/products.js
```

---

## Basic controller setup

Start the controller file with:

```js
const express = require('express');
const router = express.Router();

const Listing = require('../models/listing.js');
```

End the file with:

```js
module.exports = router;
```

Import and mount it in `server.js`:

```js
const listingsController = require('./controllers/listings.js');

app.use('/listings', isSignedIn, listingsController);
```

---

## Main resource controller actions

| Controller action | Example model method          | Response                           | Useful notes                                     |
| ----------------- | ----------------------------- | ---------------------------------- | ------------------------------------------------ |
| `newListing`      | No database method needed     | Render `listings/new.ejs`          | Displays an empty form                           |
| `create`          | `Listing.create()`            | Redirect to `/listings`            | Add the signed-in user as the owner              |
| `index`           | `Listing.find()`              | Render `listings/index.ejs`        | Usually displays all resources                   |
| `show`            | `Listing.findById()`          | Render `listings/show.ejs`         | Use `populate()` when displaying referenced data |
| `edit`            | `Listing.findById()`          | Render `listings/edit.ejs`         | Check that the resource exists                   |
| `update`          | `Listing.findByIdAndUpdate()` | Redirect to the show or index page | Check ownership before updating                  |
| `deleteListing`   | `Listing.findByIdAndDelete()` | Redirect to `/listings`            | Check ownership before deleting                  |

---

## `newListing`

The new action does not need data from MongoDB.

```js
const newListing = (req, res) => {
  res.render('listings/new.ejs');
};
```

Route:

```js
router.get('/new', newListing);
```

---

## `create`

The create action receives form data through:

```js
req.body
```

Add the signed-in user as the owner before creating the listing:

```js
const create = async (req, res) => {
  try {
    req.body.owner = req.session.user._id;

    await Listing.create(req.body);

    res.redirect('/listings');
  } catch (error) {
    console.log(error);
    res.redirect('/listings/new');
  }
};
```

Route:

```js
router.post('/', create);
```

Mongoose method:

```js
Listing.create(req.body)
```

---

## `index`

The index action finds all listings:

```js
const index = async (req, res) => {
  try {
    const listings = await Listing.find({}).populate('owner');

    res.render('listings/index.ejs', {
      listings,
    });
  } catch (error) {
    console.log(error);
    res.redirect('/');
  }
};
```

Route:

```js
router.get('/', index);
```

Mongoose method:

```js
Listing.find({})
```

You may also write:

```js
Listing.find()
```

Both versions find all documents.

---

## `show`

The show action finds one listing by its ID:

```js
const show = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    ).populate('owner');

    res.render('listings/show.ejs', {
      listing,
    });
  } catch (error) {
    console.log(error);
    res.redirect('/listings');
  }
};
```

Route:

```js
router.get('/:listingId', show);
```

Mongoose method:

```js
Listing.findById(req.params.listingId)
```

When questions contain referenced authors, the query may also populate them:

```js
const listing = await Listing.findById(
  req.params.listingId
)
  .populate('owner')
  .populate('questions.author');
```

---

## `edit`

The edit action finds the current listing and sends it to the edit form:

```js
const edit = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    if (!listing.owner.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    res.render('listings/edit.ejs', {
      listing,
    });
  } catch (error) {
    console.log(error);
    res.redirect('/listings');
  }
};
```

Route:

```js
router.get('/:listingId/edit', edit);
```

The ownership check prevents another user from manually opening the edit page.

---

## `update`

Before updating, confirm that the signed-in user owns the listing.

```js
const update = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    if (!listing.owner.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    await Listing.findByIdAndUpdate(
      req.params.listingId,
      req.body,
      {
        runValidators: true,
      }
    );

    res.redirect(`/listings/${req.params.listingId}`);
  } catch (error) {
    console.log(error);
    res.redirect(`/listings/${req.params.listingId}/edit`);
  }
};
```

Route:

```js
router.put('/:listingId', update);
```

Mongoose method:

```js
Listing.findByIdAndUpdate(
  req.params.listingId,
  req.body
)
```

The `runValidators` option tells Mongoose to apply the model's validation rules during the update.

---

## `deleteListing`

Before deleting, confirm that the signed-in user owns the listing:

```js
const deleteListing = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    if (!listing.owner.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    await Listing.findByIdAndDelete(
      req.params.listingId
    );

    res.redirect('/listings');
  } catch (error) {
    console.log(error);
    res.redirect('/listings');
  }
};
```

Route:

```js
router.delete('/:listingId', deleteListing);
```

Mongoose method:

```js
Listing.findByIdAndDelete(req.params.listingId)
```

You could also delete the document after finding it:

```js
await listing.deleteOne();
```

Finding it first is useful because the controller needs to check the owner before deleting it.

---

## Connect the controller actions to the routes

At the bottom of `controllers/listings.js`, connect each route to its controller action:

```js
router.get('/new', newListing);
router.post('/', create);
router.get('/', index);
router.get('/:listingId/edit', edit);
router.get('/:listingId', show);
router.put('/:listingId', update);
router.delete('/:listingId', deleteListing);

module.exports = router;
```

---

## Other useful controllers

Some controller actions filter data instead of displaying every document.

For example, Open House displayed listings belonging to the signed-in user.

| Controller action | Query                                                      | Purpose                                    | View                       |
| ----------------- | ---------------------------------------------------------- | ------------------------------------------ | -------------------------- |
| `profile`         | `Listing.find({ owner: req.session.user._id })`            | Find resources created by the current user | `users/show.ejs`           |
| `profile`         | `Listing.find({ favoritedByUsers: req.session.user._id })` | Find resources saved by the current user   | `users/show.ejs`           |
| `userResources`   | `Listing.find({ owner: req.params.userId })`               | Find resources created by another user     | A public user profile view |

Example profile action:

```js
const profile = async (req, res) => {
  try {
    const myListings = await Listing.find({
      owner: req.session.user._id,
    }).populate('owner');

    const myFavoriteListings = await Listing.find({
      favoritedByUsers: req.session.user._id,
    }).populate('owner');

    res.render('users/show.ejs', {
      myListings,
      myFavoriteListings,
    });
  } catch (error) {
    console.log(error);
    res.redirect('/');
  }
};
```

The important part of the query is:

```js
{
  owner: req.session.user._id
}
```

This filters the results instead of returning every listing.

---

## Embedded resource controllers

Questions are stored inside a listing document.

A simplified embedded question may contain:

```js
{
  text: String,
  author: ObjectId
}
```

Embedded resources use the parent model.

There is no separate `Question` model method such as:

```js
Question.create()
```

Instead, the controller:

1. finds the parent listing
2. finds or changes the embedded question
3. saves the parent listing

| Controller action | CRUD action       | Main Mongoose pattern                               | Useful notes                                     |
| ----------------- | ----------------- | --------------------------------------------------- | ------------------------------------------------ |
| `createQuestion`  | Create            | `Listing.findById()`, `.push()`, `.save()`          | Add the signed-in user as the author             |
| `show`            | Read              | `Listing.findById()`                                | Questions are displayed on the listing show page |
| `editQuestion`    | Read one question | `listing.questions.id()`                            | Find the question by its embedded ID             |
| `updateQuestion`  | Update            | `listing.questions.id()`, `.save()`                 | Check that the signed-in user is the author      |
| `deleteQuestion`  | Delete            | `listing.questions.id()`, `.deleteOne()`, `.save()` | Check that the signed-in user is the author      |

---

## `createQuestion`

```js
const createQuestion = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    const questionData = {
      text: req.body.text,
      author: req.session.user._id,
    };

    listing.questions.push(questionData);

    await listing.save();

    res.redirect(`/listings/${req.params.listingId}`);
  } catch (error) {
    console.log(error);
    res.redirect(`/listings/${req.params.listingId}`);
  }
};
```

Route:

```js
router.post('/:listingId/questions', createQuestion);
```

---

## `editQuestion`

```js
const editQuestion = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    const question = listing.questions.id(
      req.params.questionId
    );

    if (!question.author.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    res.render('listings/questions/edit.ejs', {
      listing,
      question,
    });
  } catch (error) {
    console.log(error);
    res.redirect(`/listings/${req.params.listingId}`);
  }
};
```

Route:

```js
router.get(
  '/:listingId/questions/:questionId/edit',
  editQuestion
);
```

The embedded document helper:

```js
listing.questions.id(req.params.questionId)
```

searches the `questions` array for a question with the matching `_id`.

---

## `updateQuestion`

```js
const updateQuestion = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    const question = listing.questions.id(
      req.params.questionId
    );

    if (!question.author.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    question.text = req.body.text;

    await listing.save();

    res.redirect(`/listings/${req.params.listingId}`);
  } catch (error) {
    console.log(error);
    res.redirect(`/listings/${req.params.listingId}`);
  }
};
```

Route:

```js
router.put(
  '/:listingId/questions/:questionId',
  updateQuestion
);
```

Because the question is embedded, save the parent listing:

```js
await listing.save();
```

---

## `deleteQuestion`

```js
const deleteQuestion = async (req, res) => {
  try {
    const listing = await Listing.findById(
      req.params.listingId
    );

    const question = listing.questions.id(
      req.params.questionId
    );

    if (!question.author.equals(req.session.user._id)) {
      return res.send("You don't have permission to do that.");
    }

    question.deleteOne();

    await listing.save();

    res.redirect(`/listings/${req.params.listingId}`);
  } catch (error) {
    console.log(error);
    res.redirect(`/listings/${req.params.listingId}`);
  }
};
```

Route:

```js
router.delete(
  '/:listingId/questions/:questionId',
  deleteQuestion
);
```

Deleting an embedded question does not immediately update MongoDB.

The parent listing must be saved:

```js
question.deleteOne();
await listing.save();
```

---

# Feature Planning Checklist

Before building a feature, answer these questions:

### View

* Which page does the user need?
* Does the resource need its own folder inside `views`?
* Is this a new, index, show, or edit page?
* What variables must the controller pass to the view?
* Does the page need a form?
* What route should the form submit to?

### Route

* Which HTTP method should be used?
* What should the URL be?
* Does the route need a resource ID?
* Does it need both a parent ID and embedded resource ID?
* Does the user need to be signed in?
* Which controller action should run?

### Controller

* What should the controller action be called?
* Does it need data from `req.body`?
* Does it need an ID from `req.params`?
* Does it need the current user from `req.session.user`?
* Which Mongoose method should it use?
* Should it render a view or redirect?
* Does it need an ownership check?
* What should happen if an error occurs?

Build one complete feature at a time:

```plaintext
Plan the view
    ↓
Plan the route
    ↓
Write the controller action
    ↓
Connect the route and action
    ↓
Test the feature
    ↓
Commit the working code
```

Do not build every view first and every route later. Complete one user story from beginning to end before moving to the next feature.
