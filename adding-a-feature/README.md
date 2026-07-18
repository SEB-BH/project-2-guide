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

A request usually follows this flow:

```plaintext
User action
    ↓
Route
    ↓
Controller action
    ↓
Mongoose model
    ↓
Controller response
    ↓
EJS view or redirect
```

For example, when a user opens the new listing form:

```plaintext
User visits /listings/new
    ↓
GET /listings/new
    ↓
newListing controller action
    ↓
Render views/listings/new.ejs
```

When a user submits the form:

```plaintext
User submits the form
    ↓
POST /listings
    ↓
create controller action
    ↓
Listing.create()
    ↓
Redirect to /listings
```
