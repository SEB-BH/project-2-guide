<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Middleware for resource routes</span>
</h1>

**Middleware** is code that runs between receiving a request and running the final controller action.

Middleware can:

* check whether a user is signed in
* make the signed-in user available to EJS
* check whether a user has a specific role
* validate request data
* stop an unauthorized request


## 1. Create the middleware folder

From the root of your project, run:

```bash
mkdir -p middleware
touch middleware/is-signed-in.js
touch middleware/pass-user-to-view.js
```

Your project should now contain:

```plaintext
middleware/
├── is-signed-in.js
└── pass-user-to-view.js
```

### `passUserToView`

`passUserToView` makes the signed-in user available inside EJS views.

```
const passUserToView = (req, res, next) => {
    if (req.session.user) {
        res.locals.user = req.session.user
    } else {
        res.locals.user = null
    }
    next()
}

module.exports = passUserToView
```


You should import it into your `server.js` file and use it below your other middleware
```js
const passUserToView = require('./middleware/pass-user-to-view.js')

app.use(morgan('dev'))
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: true,
    store: MongoStore.create({
      mongoUrl: process.env.MONGODB_URI
    }),
}))
app.use(passUserToView) // use your custom middleware here
```

The middleware creates:

```js
res.locals.user
```

This allows EJS files to use:

```ejs
<% if (user) { %>
  <p>Welcome, <%= user.username %>.</p>
<% } %>
```

> 💡 Mount `passUserToView` before any controllers that need access to the `user` variable.

### `isSignedIn`

`isSignedIn` prevents signed-out users from accessing protected routes.

```js
const isSignedIn = (req, res, next) => {
    if (req.session.user) return next()
    res.redirect('/auth/sign-in')
}

module.exports = isSignedIn
```

```js
const isSignedIn = require('./middleware/is-signed-in.js')
```

Protect the complete resource controller:

```js
app.get('/listings', isSignedIn, listingsController.newListings);
```

This means accessing the new listing form requires a signed-in user.



---

## 1. Create the middleware folder

From the root of your project, run:

```bash
mkdir -p middleware
touch middleware/is-signed-in.js
touch middleware/pass-user-to-view.js
touch middleware/is-admin.js
```

Your project should now contain:

```plaintext
middleware/
├── is-admin.js
├── is-signed-in.js
└── pass-user-to-view.js
```

---

## 2. Create `passUserToView`

`passUserToView` makes the signed-in user available inside EJS views.

Open `middleware/pass-user-to-view.js`:

```js
const passUserToView = (req, res, next) => {
    if (req.session.user) {
        res.locals.user = req.session.user
    } else {
        res.locals.user = null
    }

    next()
}

module.exports = passUserToView
```

The middleware creates:

```js
res.locals.user
```

This allows EJS files to use the `user` variable:

```ejs
<% if (user) { %>
  <p>Welcome, <%= user.username %>.</p>
<% } %>
```

You can also use the user's role:

```ejs
<% if (user && user.role === 'admin') { %>
  <a href="/admin">Admin Dashboard</a>
<% } %>
```

---

## 3. Use `passUserToView` in `server.js`

Import the middleware:

```js
const passUserToView = require(
    './middleware/pass-user-to-view.js'
)
```

Use it after the session middleware and before your routes:

```js
app.use(morgan('dev'))

app.use(
    session({
        secret: process.env.SESSION_SECRET,
        resave: false,
        saveUninitialized: false,
        store: MongoStore.create({
            mongoUrl: process.env.MONGODB_URI,
        }),
    })
)

app.use(passUserToView)
```

The order matters because `passUserToView` reads:

```js
req.session.user
```

The session middleware must run first.

> 💡 Mount `passUserToView` before any routes or controllers whose EJS views need access to the `user` variable.

---

## 4. Create `isSignedIn`

`isSignedIn` prevents signed-out users from accessing protected routes.

Open `middleware/is-signed-in.js`:

```js
const isSignedIn = (req, res, next) => {
    if (req.session.user) {
        return next()
    }

    res.redirect('/auth/sign-in')
}

module.exports = isSignedIn
```

When a signed-in user makes the request:

```js
return next()
```

passes the request to the next middleware or controller action.

When a signed-out user makes the request:

```js
res.redirect('/auth/sign-in')
```

stops the request and sends the user to the sign-in page.

---

## 5. Import `isSignedIn`

In `server.js`:

```js
const isSignedIn = require(
    './middleware/is-signed-in.js'
)
```

Protect the new listing form:

```js
app.get(
    '/listings/new',
    isSignedIn,
    listingsController.newListing
)
```

This route follows this order:

```plaintext
GET /listings/new
        ↓
isSignedIn
        ↓
newListing
```

If the user is not signed in, `newListing` does not run.

---

## 6. Protect resource routes

Any route that creates, updates, or deletes data should normally require a signed-in user.

```js
app.get(
    '/listings/new',
    isSignedIn,
    listingsController.newListing
)

app.post(
    '/listings',
    isSignedIn,
    listingsController.create
)

app.get(
    '/listings/:listingId/edit',
    isSignedIn,
    listingsController.edit
)

app.put(
    '/listings/:listingId',
    isSignedIn,
    listingsController.update
)

app.delete(
    '/listings/:listingId',
    isSignedIn,
    listingsController.deleteListing
)
```

You may decide whether the index and show routes are public or protected.

Public examples:

```js
app.get(
    '/listings',
    listingsController.index
)

app.get(
    '/listings/:listingId',
    listingsController.show
)
```

Protected examples:

```js
app.get(
    '/listings',
    isSignedIn,
    listingsController.index
)

app.get(
    '/listings/:listingId',
    isSignedIn,
    listingsController.show
)
```

Choose the option that matches your project's user stories.

---

## 7. Add a role with `isAdmin`

Some applications need actions that are only available to administrators.

For example, an admin may be allowed to:

* view an admin dashboard
* manage all users
* remove inappropriate content
* update application categories
* view system reports

The user's role should be stored in the User model:

```js
role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
}
```

It must also be available in the session:

```js
req.session.user.role
```

---

## 8. Create `isAdmin`

Open `middleware/is-admin.js`:

```js
const isAdmin = (req, res, next) => {
    if (
        req.session.user &&
        req.session.user.role === 'admin'
    ) {
        return next()
    }

    res.status(403).render('error.ejs', {
        msg: 'You do not have permission to view this page.',
    })
}

module.exports = isAdmin
```

An HTTP status of `403` means:

```plaintext
Forbidden
```

The server understood the request, but the current user does not have permission to complete it.

---

## 9. Import `isAdmin`

In `server.js`:

```js
const isAdmin = require('./middleware/is-admin.js')
```

Use both middleware functions on an admin route:

```js
app.get(
    '/admin',
    isSignedIn,
    isAdmin,
    adminController.index
)
```

The middleware runs from left to right:

```plaintext
GET /admin
    ↓
isSignedIn
    ↓
isAdmin
    ↓
adminController.index
```

Use `isSignedIn` before `isAdmin`.

`isSignedIn` confirms that a session user exists. `isAdmin` then checks the user's role.

---

## 10. Protect admin actions

An admin may be allowed to delete any listing:

```js
app.delete(
    '/admin/listings/:listingId',
    isSignedIn,
    isAdmin,
    adminController.deleteListing
)
```

An admin may be allowed to view all users:

```js
app.get(
    '/admin/users',
    isSignedIn,
    isAdmin,
    adminController.usersIndex
)
```

An admin may be allowed to update another user's role:

```js
app.put(
    '/admin/users/:userId/role',
    isSignedIn,
    isAdmin,
    adminController.updateUserRole
)
```

Do not protect an admin route only by hiding its link in EJS.

This is not enough:

```ejs
<% if (user.role === 'admin') { %>
  <a href="/admin">Admin Dashboard</a>
<% } %>
```

Hiding the link improves the interface, but a user can manually enter the URL or send the request.

The server route must also use:

```js
isAdmin
```

---

## 11. Roles compared with ownership

A **role check** and an **ownership check** answer different questions.

| Check           | Question                                                  |
| --------------- | --------------------------------------------------------- |
| `isSignedIn`    | Is there a signed-in user?                                |
| `isAdmin`       | Does the signed-in user have the admin role?              |
| Ownership check | Does this specific resource belong to the signed-in user? |

A normal user may edit their own listing but not another user's listing.

That check belongs inside the controller:

```js
const edit = async (req, res) => {
    const foundListing = await Listing.findById(
        req.params.listingId
    )

    if (
        !foundListing.owner.equals(
            req.session.user._id
        )
    ) {
        return res.status(403).render('error.ejs', {
            msg: 'You do not have permission to edit this listing.',
        })
    }

    res.render('listings/edit.ejs', {
        listing: foundListing,
    })
}
```

An application may allow either the owner or an admin to perform an action:

```js
const isOwner = foundListing.owner.equals(
    req.session.user._id
)

const userIsAdmin =
    req.session.user.role === 'admin'

if (!isOwner && !userIsAdmin) {
    return res.status(403).render('error.ejs', {
        msg: 'You do not have permission to do that.',
    })
}
```

This means:

```plaintext
Owner
  OR
Admin
  ↓
Action is allowed
```

---

## 12. Middleware planning table

| Route type              | Recommended protection            |
| ----------------------- | --------------------------------- |
| Public landing page     | No authentication middleware      |
| Public resource index   | No authentication middleware      |
| Create form             | `isSignedIn`                      |
| Create action           | `isSignedIn`                      |
| Edit form               | `isSignedIn` plus ownership check |
| Update action           | `isSignedIn` plus ownership check |
| Delete action           | `isSignedIn` plus ownership check |
| User profile            | `isSignedIn`                      |
| Admin dashboard         | `isSignedIn`, then `isAdmin`      |
| Admin management action | `isSignedIn`, then `isAdmin`      |

Middleware protects the route before the controller runs.

Ownership checks usually happen inside the controller after the resource has been found.

---

## Middleware checklist

Before creating a route, ask:

* Does the user need to be signed in?
* Does the route need access to `res.locals.user`?
* Is the route only for admins?
* Does the controller need to verify resource ownership?
* Should admins be allowed to bypass the ownership check?
* What should happen when permission is denied?
* Is the middleware mounted before the controller action?
* Is `passUserToView` mounted after the session middleware?
