
<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Routes</span>
</h1>


A route combines:

* an HTTP method
* a URL
* a controller action

For example:

```js
app.get('/listings', isSignedIn, listingsController.index);
```

## Main resource routes

The Open House `Listing` resource used the following RESTful routes:

| HTTP method | Route                       | Purpose                       |
| ----------- | --------------------------- | ----------------------------- |
| `GET`       | `/listings/new`             | Display the new listing form  |
| `POST`      | `/listings`                 | Create a listing              |
| `GET`       | `/listings`                 | Display all listings          |
| `GET`       | `/listings/:listingId`      | Display one listing           |
| `GET`       | `/listings/:listingId/edit` | Display the edit listing form |
| `PUT`       | `/listings/:listingId`      | Update a listing              |
| `DELETE`    | `/listings/:listingId`      | Delete a listing              |


> 🚨 Place `/new` above `/:listingId`. Otherwise, Express may treat the word `new` as a listing ID.

## Route parameters

A colon creates a route parameter:

```plaintext
/listings/:listingId
```

If the user visits:

```plaintext
/listings/67f1254c91ab28c4387c1234
```

Express stores the ID in:

```js
req.params.listingId
```

Example:

```js
const listing = await Listing.findById(req.params.listingId);
```

Use clear parameter names:

```plaintext
:listingId
:bookId
:eventId
:recipeId
```


## Other useful routes


| HTTP method | Route                      | Purpose                                      |
| ----------- | -------------------------- | -------------------------------------------- |
| `GET`       | `/users/profile`           | Display the signed-in user's profile         |
| `GET`       | `/books/mine`              | Display books created by the signed-in user  |
| `GET`       | `/events/mine`             | Display events created by the signed-in user |
| `GET`       | `/recipes/saved`           | Display recipes saved by the signed-in user  |
| `GET`       | `/users/:userId/resources` | Display resources belonging to one user      |

Use the route that best matches the feature's user story.


## Embedded resource routes

Questions are embedded inside listings, so every question route includes the parent listing ID.

| CRUD action | HTTP method | Route                                             | Purpose                               |
| ----------- | ----------- | ------------------------------------------------- | ------------------------------------- |
| Create      | `POST`      | `/listings/:listingId/questions`                  | Add a question to a listing           |
| Read        | `GET`       | `/listings/:listingId`                            | Display the listing and its questions |
| Update form | `GET`       | `/listings/:listingId/questions/:questionId/edit` | Display the edit question form        |
| Update      | `PUT`       | `/listings/:listingId/questions/:questionId`      | Update a question                     |
| Delete      | `DELETE`    | `/listings/:listingId/questions/:questionId`      | Delete a question                     |

Many of the routes contains two IDs:

```js
req.params.listingId
req.params.questionId
```

The listing ID finds the parent document.

The question ID finds the embedded subdocument.
