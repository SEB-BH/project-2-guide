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

## 2. `passUserToView`

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

## 3. `isSignedIn`

`isSignedIn` prevents signed-out users from accessing protected routes.

```js
const isSignedIn = (req, res, next) => {
    if (req.session.user) return next()
    res.redirect('/auth/sign-in')
}

module.exports = isSignedIn
```

```js
// server.js

const isSignedIn = require('./middleware/is-signed-in.js')
```

Protect the complete resource controller:

```js
app.get('/listings/new', isSignedIn, listingsController.newListings);
```

- This means accessing the new listing form requires a signed-in user.
- Any route that creates, updates, or deletes data should normally require a signed-in user.
- You may decide whether the index and show routes are public or protected.


## 4. Add a role with `isAdmin` (optional)

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

### Create `isAdmin`

```bash
touch middleware/is-admin.js`
```

```js
const isAdmin = (req, res, next) => {
    if (
        req.session.user &&
        req.session.user.role === 'admin'
    ) {
        return next()
    }

    res.redirect('/')

// if you create an error page you can render that instead of redirecting
    // res.render('error.ejs', {
    //     msg: 'You do not have permission to view this page.',
    // })
}

module.exports = isAdmin
```

### Import `isAdmin`

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

Use `isSignedIn` before `isAdmin`.

`isSignedIn` confirms that a session user exists. `isAdmin` then checks the user's role.