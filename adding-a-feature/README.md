<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Adding a Feature</span>
</h1>


Most MEN stack features contain three main parts:

1. **View** — What the user sees.
2. **Route** — The URL and HTTP method used by the request.
3. **Controller action** — The code that handles the request.

The controller action often communicates with a fourth part:

4. **Model** — The Mongoose model used to read or change database data.

## Feature Planning Checklist

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
