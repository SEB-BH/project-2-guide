<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Create a Nav Partial</span>
</h1>


A **partial** is a reusable piece of EJS.

Instead of writing the navigation bar on every page, we can write it once and include it wherever it is needed.


## 1. Create the partial

Create a `partials` folder inside `views`:

```bash
mkdir -p views/partials
touch views/partials/nav.ejs
```

Your views folder may look like this:

```plaintext
views/
├── auth/
├── partials/
│   └── nav.ejs
└── index.ejs
```

## 2. Add navigation links

Open `views/partials/nav.ejs`.

```ejs
<nav>
  <a href="/">Home</a>

  <% if (user) { %>
    <a href="/listings">All Listings</a>
    <a href="/listings/new">Create a Listing</a>
    <a href="/users/profile">My Profile</a>
    <a href="/auth/sign-out">Sign Out</a>
  <% } else { %>
    <a href="/auth/sign-in">Sign In</a>
    <a href="/auth/sign-up">Sign Up</a>
  <% } %>
</nav>
```

The `user` variable is provided by the `passUserToView` middleware.

Use the sign-out route already included in your auth template. Do not create a second sign-out route.

## 3. Include the partial in a root-level view (home.ejs)

Inside a file such as `views/index.ejs`, use:

```ejs
<%- include('./partials/nav.ejs') %>
```

## 4. Include the partial in a nested view (any view inside of another folder like views/listings)

Inside a file such as `views/listings/index.ejs`, move up one folder before entering `partials`:

```ejs
<%- include('../partials/nav.ejs') %>
```

For example:

```ejs
  <%- include('../partials/nav.ejs') %>

  <main>
    <h1>All Listings</h1>
  </main>
</body>
</html>
```

Your public files and navigation partial are now ready.
