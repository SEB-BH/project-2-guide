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
| [Models](./models/README.md)                                                    | Create your schemas                                                           |
| [Views](./views/README.md)                                                      | Conventions for creating views                                                |
| [Routes](./routes/README.md)                                                    | Conventions for building routes                                               |
| [Controllers](./controllers/README.md)                                          | Conventions for building routes                                               |

## References

📖 [Reference Materials](./references/README.md)


**Find a 👾 bug 👾 or have suggestions? [Let us know](https://ga-curriculum.github.io/universal-resources-internal/module-feedback.html)!**



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
