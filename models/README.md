<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Models</span>
</h1>

A **model** describes the data that your application stores in MongoDB.

For example, Open House stores listings. Each listing contains information such as:

* a street address
* a city
* a price
* a size
* an owner
* users who favorited the listing
* questions asked about the listing

Before creating views, routes, and controller actions, plan what information your resource needs to store.


## 1. Create the model file

The Session Auth Template already contains a `models` folder and a `User` model.

Create a model file for your main resource.

Examples:

```plaintext
models/book.js
models/event.js
models/recipe.js
models/support-ticket.js
```

## 2. Create a basic schema

Here is the Listing schema from open house:

```js
const mongoose = require('mongoose')

const listingSchema = new mongoose.Schema({
    streetAddress: {
        type: String,
        required: true,
        trim: true,
    },
    city: {
        type: String,
        required: true,
        trim: true,
    },
    price: {
        type: Number,
        required: true,
        min: 0,
    },
    size: {
        type: Number,
        required: true,
        min: 0,
    },
}, { timestamps: true })

const Listing = mongoose.model('Listing', listingSchema)

module.exports = Listing
```

> 💡 Use a singular, capitalized name when creating the model: `Listing`, `Book`, `Recipe`, or `Event`.

### Common schema data types

| Data type                        | Example use                         |
| -------------------------------- | ----------------------------------- |
| `String`                         | Names, titles, descriptions, cities |
| `Number`                         | Prices, quantities, ratings         |
| `Boolean`                        | Available, completed, published     |
| `Date`                           | Appointment dates, deadlines        |
| `mongoose.Schema.Types.ObjectId` | References to other documents       |
| Array                            | Tags, users, embedded comments      |


### Common schema options

Schema options help validate and clean your data.

| Option            | Purpose                                 |
| ----------------- | --------------------------------------- |
| `required: true`  | Requires a value                        |
| `default`         | Provides a value when none is submitted |
| `enum`            | Allows only specific values             |
| `min`             | Sets the smallest allowed number        |
| `max`             | Sets the largest allowed number         |
| `minlength`       | Sets the shortest allowed string        |
| `maxlength`       | Sets the longest allowed string         |
| `trim: true`      | Removes extra spaces from a string      |
| `lowercase: true` | Changes a string to lowercase           |
| `unique: true`    | Creates a unique database index         |


## 3. Reference the resource owner

Most Project 2 resources should record which user created them.

Add an `owner` field:

```js
owner: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
}
```

`owner` is the `_id` of a user.

The `create` controller should add the signed-in user's ID:

```js
listingData.owner = req.session.user._id

await Listing.create(req.body)
```

The owner should come from the signed-in session.


## 4. Embed related data _(optional)_

An **embedded document** is stored directly inside its parent document.

Open House embeds questions inside listings.

```js
const mongoose = require('mongoose')

const questionSchema = new mongoose.Schema(
    {
        text: {
            type: String,
            required: true,
            trim: true,
        },
        author: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'User',
            required: true,
        },
    },
    {
        timestamps: true,
    }
)

const listingSchema = new mongoose.Schema(
    {
        streetAddress: {
            type: String,
            required: true,
            trim: true,
        },
        city: {
            type: String,
            required: true,
            trim: true,
        },
        price: {
            type: Number,
            required: true,
            min: 0,
        },
        size: {
            type: Number,
            required: true,
            min: 0,
        },
        owner: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'User',
            required: true,
        },
        favoritedByUsers: [
            {
                type: mongoose.Schema.Types.ObjectId,
                ref: 'User',
            },
        ],
        questions: [questionSchema],
    },
    {
        timestamps: true,
    }
)

const Listing = mongoose.model('Listing', listingSchema)

module.exports = Listing
```

To populate both the owner and question authors:

```js
const listing = await Listing
    .findById(req.params.listingId)
    .populate('owner')
    .populate('questions.author')
```

### References compared with embedded documents

| Use a reference when...                            | Use an embedded document when...                     |
| -------------------------------------------------- | ---------------------------------------------------- |
| The related data needs its own main page           | The related data belongs to one parent               |
| The related data can exist independently           | The related data should not exist without its parent |
| Many documents may share the same related document | The related data is usually viewed with its parent   |
| The related data may become large                  | The related data should remain relatively small      |

Examples of references:

* a listing's owner
* an event's organizer
* users attending an event
* a book's borrower

Examples of embedded documents:

* questions inside a listing
* comments inside a post
* updates inside a support ticket
* notes inside a project task

> 🧠 Embedding is not automatically better or easier. Decide whether the related data needs to exist separately from its parent.


## 5. Add roles to the User model _(optional)_

A **role** describes a category of user and the permissions that category receives.

Your application might use two roles:

```plaintext
user
admin
```

Open `models/user.js` and add a `role` field to the user schema:

```js
role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
}
```

The default role should be:

```js
'user'
```

Do not include a role selection in the public sign-up form.

Create or assign admins through a controlled process, such as:

* updating the user in MongoDB Atlas

When a user signs in, make sure the role is included in the session:

```js
req.session.user = {
    _id: foundUser._id,
    username: foundUser.username,
    role: foundUser.role,
}
```
