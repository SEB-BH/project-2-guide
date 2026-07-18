<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Views</span>
</h1>

A **view** is an EJS file that creates HTML.  Each main resource should normally have its own folder inside `views`.  Open House had a `Listing` resource, so it used:

```plaintext
views/listings/
```

Create a folder for your resource.

Examples:

```plaintext
views/books/
views/events/
views/recipes/
views/products/
```

> 🧠 Use plural resource names for folders and routes.

## Main resource views

Open House used the following views for the `Listing` resource:

| View                       | Purpose                                 | Related action |
| -------------------------- | --------------------------------------- | -------------- |
| `views/listings/new.ejs`   | Display the form for creating a listing | New            |
| `views/listings/index.ejs` | Display all listings                    | Index          |
| `views/listings/show.ejs`  | Display one listing                     | Show           |
| `views/listings/edit.ejs`  | Display the form for editing a listing  | Edit           |

Not every controller action needs its own view.

For example:

* `create` normally redirects after creating data.
* `update` normally redirects after updating data.
* `deleteListing` normally redirects after deleting data.


## Other useful views

Some features do not follow the standard new, index, show, and edit pattern.

For example, Open House had a profile page that only displayed listings belonging to the signed-in user.

| View                   | Purpose                                           | Data needed          |
| ---------------------- | ------------------------------------------------- | -------------------- |
| `views/users/show.ejs` | Display the signed-in user's profile              | The current user     |
| `views/users/show.ejs` | Display resources owned by the signed-in user     | `myListings`         |
| `views/users/show.ejs` | Display resources favorited by the signed-in user | `myFavoriteListings` |

Example:

```ejs
<h2>My Listings</h2>

<% if (myListings.length === 0) { %>
  <p>You have not created any listings.</p>
<% } %>

<% myListings.forEach((listing) => { %>
  <article>
    <a href="/listings/<%= listing._id %>">
      <%= listing.streetAddress %>
    </a>
  </article>
<% }) %>
```

This same pattern can be used for:

* my books
* my events
* my recipes
* my support requests
* my saved resources
* resources created by a specific user


## Embedded resource views

An **embedded resource** belongs inside another resource.

**Create a nested view folder for embedded resources.**

In Open House, questions were embedded inside listings.

A question does not need its own main index page. It is displayed inside the listing's show page.

| CRUD action | View                                | What the view does                         |
| ----------- | ----------------------------------- | ------------------------------------------ |
| Create      | `views/listings/show.ejs`           | Displays the new question form             |
| Read        | `views/listings/show.ejs`           | Displays the listing's questions           |
| Update      | `views/listings/questions/edit.ejs` | Displays the edit question form            |
| Delete      | `views/listings/show.ejs`           | Displays a delete button beside a question |


### Create a question

Place the question form on the listing show page:

```ejs
<form
  action="/listings/<%= listing._id %>/questions"
  method="POST"
>
  <label for="text">Ask a question:</label>
  <textarea
    id="text"
    name="text"
    required
  ></textarea>

  <button type="submit">Submit Question</button>
</form>
```

### Read the questions

Display the questions on the same show page:

```ejs
<section>
  <h2>Questions</h2>

<!-- checks if any questions exist -->
  <% if (listing.questions.length === 0) { %>
    <p>No questions have been asked yet.</p>
  <% } %>

<!-- displays all the questions -->
  <% listing.questions.forEach((question) => { %>
    <article>
      <p><%= question.text %></p>
      <p>Asked by: <%= question.author.username %></p>

<!-- checks if the logged in user is the one who asked the question -->
      <% if (question.author._id.equals(user._id)) { %>
  
<!-- links to a separate view for editing the question -->
        <a href="/listings/<%= listing._id %>/questions/<%= question._id %>/edit" >
          Edit Question
        </a>

<!-- deletes the question -->
        <form action="/listings/<%= listing._id %>/questions/<%= question._id %>?_method=DELETE" method="POST">
          <button type="submit">Delete Question</button>
        </form>

        
      <% } %>
    </article>
  <% }) %>
</section>
```

### Update a question

Inside `views/listings/questions/edit.ejs`:

```ejs
<h1>Edit Question</h1>

<form
  action="/listings/<%= listing._id %>/questions/<%= question._id %>?_method=PUT"
  method="POST"
>
  <label for="text">Question:</label>
  <textarea
    id="text"
    name="text"
    required
  ><%= question.text %></textarea>

  <button type="submit">Save Changes</button>
</form>
```
