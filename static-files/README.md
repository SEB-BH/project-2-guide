<h1>
  <span class="headline">Build Guide</span>
  <span class="subhead">Static Files and the Public Folder</span>
</h1>


**Static files** are files that Express sends directly to the browser.

Common static files include:

* CSS files
* images
* client-side JavaScript
* icons
* fonts

These files should be stored inside a `public` folder.

## 1. Create the public folders

From the root of your project, run:

```bash
mkdir -p public/stylesheets
mkdir -p public/images
touch public/stylesheets/style.css
```

Your project should now contain:

```plaintext
public/
├── images/
└── stylesheets/
    └── style.css
```


## 2. Tell Express to use the public folder

Open `server.js`.

Add the static middleware with the other `app.use()` statements:

```js
app.use(express.static('public'))
```

```js
app.use(express.urlencoded({ extended: false })) 
app.use(methodOverride('_method'))
app.use(express.static('public'))  // add this line if it's not already there
```


## 3. Link the stylesheet

Add the stylesheet inside the `<head>` of each page:

✅ Correct:

```html
<link rel="stylesheet" href="/stylesheets/style.css">
```

Do not include `public` in the path.

❌ Incorrect:

```html
<link rel="stylesheet" href="/public/stylesheets/style.css">
```


## 4. Add a favicon (optional)

Visit [SVG Repo](https://www.svgrepo.com/) and search for an icon that matches your application (or make your own).

Rename the downloaded file to something clear, such as:

```text
app-icon.svg
```

**Create the favicon files**

Visit the [Favicon.io SVG Converter](https://favicon.io/svg-favicon/).

1. Upload your `app-icon.svg` file.
2. Preview the favicon.
3. Click **Download favicon package**.
4. Unzip the downloaded folder.

Place all the `favicon.ico` file directly inside the `public` folder:

```text
project-name/
├── public/
│   ├── images/
│   ├── stylesheets/
│   │   └── style.css
│   ├── favicon.ico
│   └── site.webmanifest
```

**Link the favicon files **

Add the following code inside the `<head>` of your EJS page:

```html
<link rel="icon" type="image/png" sizes="16x16" href="/favicon.ico"/>
```

Browsers often save old favicon files in their cache. If the previous icon still appears, perform a hard refresh or clear the browser cache.


## 5. Reminder: add images inside of the public folder

Place images inside:

```plaintext
public/images/
```

For example:

```plaintext
public/images/default-listing.jpg
```

Use the image in an EJS page:

```html
<img
  src="/images/default-listing.jpg"
  alt="Default property listing"
>
```

Again, do not include `public` in the image path.