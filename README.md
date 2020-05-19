# CS52 Workshops:  ALTERNATIVE SERVERS

Do you remember this other language from CS1? What was it called again. Ah yes, Python?

![yes gif](https://media.giphy.com/media/Pd8Bf06Sas4yQ/giphy.gif)

Are you ready to leave semicolons and curly braces behind?

![it's about time](https://media.giphy.com/media/jVStxzak9yk2Q/giphy.gif)

## Overview

In this tutorial we're going to use a few things:
1. The Python requests library. Just as cool as axios.
2. MongoDB (YES!). We'll be using pymongo. It's like mongoose, but for Python.
3. Oh and, a little framework called Flask

Then we're going to build a library. A...what? A library. Except you can't read the books, just look at them. But that's fun too.

## Setup

Let's get all the tools we need. Head to the terminal.
1. Get Flask and Pymongo. Run
```console
pip install Flask
python -m pip install pymongo
```
pip comes with python, so if you have python, you have pip. We will be using python 3.7 and the latest version of pip. However, if you have python 3.6 you should have no problem.

2. If you read Flask tutorials, you'll see that a lot of them use virtual environments. We will not be doing that in this tutorial. The benefit of virtual environments is that it you can work on isolated projects with different dependencies at the same time. So, if you are already using a different version of flask, feel free to use virtual environments!
3. Get an API key for Google Books. We've done this a bunch of times, from YouTube to the ML tutorial. Go [here](https://console.developers.google.com/), in APIs and Services, search for Books. Enable the API. Then to to Credentials. Click on Create Credentials, select api key, restrict it to the Books API, then write it down somewhere.
4. Phew. Now we're ready.

## Step by Step

Much of the code we write here will look familiar to Express. Keep an eye out for the similarities, it'll help in remembering. We'll help you too, with this: :smiling_imp:

### File Structure
Create a directory to work in. Name it whatever you want. In there we're going to create two directories and two files.
1. The first file is called app.py. It's like server.js :smiling_imp:
2. The second file is called db.py. Yep, you guessed it, it's our model and controller rolled into one. :smiling_imp:
3. The first directory is called templates. This will hold our HTML. It's like views in Express :smiling_imp:
4. The second directory is called static. This will hold CSS and other static files.

### db.py
Let's start in db.py. If you would like to start with app.py, scroll on ahead, but it can be grounding to know what your database is doing. First we're going to need something from pymongo.
```python
from pymongo import MongoClient
```
Now we're going to create a client and connect it to a local database. Create a MongoClient instance with the url to your local database. If CS1 has been a while, here's how to do it:
```python
client = MongoClient('mongodb://localhost:27017')
```
Next, we want to create a database, and then a collection. Use the dot notation to do these, e.g db = client.database. Ah, we gave that one away.
<details>
 <summary>Here's the second one. But try it first!</summary>

 ```python
favourites = db.favourites
 ```
</details>
The variable names could be literally anything. In our example, if you open up MongoDB Compass (or use the CLI), you'll find that there is now a database called database, and inside that, there is a collection called bookshelf.

And that's it for the model. We're not going to create a Schema. We're just going to go ahead and throw data into it.

#### Controller
We're still in db.py. We want the functions that will directly manipulate our database now. Can you think of any? Well, we're going to be searching for books with the Books API, and then we'll add those to our library. We'll also want to be able to kick books out of our library. Yep, add and kick functions. Or delete, if you want to be boring. Mongoose, I mean, pymongo, has methods called insert_one, and delete_one. :smiling_imp: They behave like pymongo, I mean, mongoose.

Note: delete is fairly straightforward (one line), but add has a little trick to it. In our database, we will not be using Mongo-generated IDs, and will instead be using the ids that come with the Google objects for two reasons:
1. We already know they are unique
2. We want the ids to be the same across both our database and in the database of books we're querying.

In addition, since clicking a book adds it, and we can click on the same book twice and cause Mongo DB to yell at us because we're trying to add an id that already exists, so first check that the id doesn't already exist. Do that using pymongo's count_documents.

<details>
 <summary>Here, in case</summary>

 ```python
def add_book(book):
  if favourites.count_documents({ '_id': book['id'] }, limit = 1) == 0:
        book['_id'] = book['id']
        favourites.insert_one(book)
```
```python
def delete_book(id):
    favourites.delete_one({'_id': id})
 ```
</details>

### app.py
Ah, finally. Let's write some real python. Import all the necessaries! Go ahead, a bunch of import statements. Import everything! No actually just these:
```python
from flask import Flask, render_template, request, redirect, url_for
import db
import requests
import json
```
Now some flask
```python
app = Flask(__name__)
```
Now, there's so many ways to build our library, but we're going to work like this: We'll write a function that takes a search term and returns a list of objects about books whose titles match that name. Confused? Play around with the Books API and see what kind of data it returns. This function is not hard. You will use requests Here's how you use requests:
```python
requests.get(url, params=params)
```
The params parameter is optional. If you intend to use it, you will need to define a python dictionary (javascript object) that holds the parameter name and its value. In your url, you will also need to include this query "?q=title". Don't forget about the case where the entered title has spaces. Convert those spaces to '+'. Lots of ways, we'll do a basic, slightly inefficient one in our example.
<details>
 <summary>try it first</summary>

 ```python
def search_title(name):
    name = '+'.join(name.split(' '))
    r = requests.get(ROOT_URL + '?q=' + name + '&key=' + API_KEY, params=params)
    return r.json()['items']
 ```
</details>

We also want a default search term. So go ahead and call your function on your favourite book title. Assign the results to some variable.

#### routing :smiling_imp:
This is the fun part. Routing in Flask works like routing in express, except that you are required to return something to the view, even if you just perform an action like delete. For that case, there's a special empty return statement ('', 204) that translates to "all good. nothing to see here". 

We're going to use the `@app.route()` decorator to bind a URL to a function which will return what we want to display. The first argument passed to `route()` will be the url string we are targeting. By default the route will only respond to `GET` requests, but that can be changed by providing the `methods` argument to the route() decorator. After the decorator we will define the function that will return whatever we want to render. For example:  

```python
@app.route('/hello', methods=['GET', 'POST'])
def hello_world():
   return ‘hello world’
 ```
 In this tutorial we will construct four routes, only two of which will be accessible by the user.
1. Home page. This is where the search bar will be.
2. Book shelf, or library, or favourites.
3. add/<:id> :smiling_imp:
4. delete/<:id> :smiling_imp:

##### Home page

We want to respond to `GET` and `POST` requests. Our function will return the `render_template()` function which will in turn render our template `index.html` (which we will create later). In case of a `POST` request, we also want to retrieve the contents of the `search-bar` and pass that to our `render_template()` function so we render the appropriate books.

<details>
 <summary>try it first</summary>

 ```python
@app.route('/', methods=['GET', 'POST'])
def home_page():
    global results
    if request.method == 'POST' and request.form['search-bar']:
        results = search_title(request.form['search-bar'])
    return render_template('index.html', results=results)
 ```
</details>

##### Favourites
Here want to respond to `GET` requests on the URL `/favourites` and render our `favs.html` template with all our favourite books.
<details>
 <summary>try it first</summary>

 ```python
@app.route('/favourites')
def favourites():
    all = db.get_all()
    return render_template('favs.html', all=all)
 ```
</details>

##### add/<:id>
Here want to respond to `POST` requests and add the book with the given id to our database. We don't actually want to render anything so we just return the special empty return statement that we mentioned before.
<details>
 <summary>try it first</summary>

 ```python
@app.route('/add/<string:id>', methods=['POST'])
def add(id):
    if request.method == 'POST':
        book = list(filter(lambda b: b["id"] == id, results))[0]
        db.add_book(book)
    return ('', 204)
 ```
</details>

##### delete/<:id>
Here want to respond to `DELETE` requests and delete the book with the corresponding id from our database. Again, we don't render anything.

<details>
 <summary>try it first</summary>

 ```python
@app.route('/delete/<string:id>', methods=['DELETE'])
def delete(id):
    if request.method == 'DELETE':
        db.delete_book(id)
    return ('', 204)
 ```
</details>

At the end of your app.py, add this:
```python
if __name__ == '__main__':
    app.run()
 ```

### Templates

Alright, now that we have our Flask framework setup, let's get back to some good ol' HTML :page_facing_up:. 

If you haven't already, make a `templates/` directory. We are going to break our HTML up so also add a `partials/` directory inside of `templates/`.

#### Partials

In our `templates/partials/` directory, make a `templates/partials/head.html` and `templates/partials/nav.html` file. 
We are going to essentially use the same `head` and `nav` elements from SA7 but if you forget what they look like here they are:

<details>
 <summary>head.html</summary>

```html
<head>
    <title>Hot Cocoa</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.98.2/css/materialize.min.css">
    <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.98.2/js/materialize.min.js"></script>
    <link href="/static/style.css" rel="stylesheet" type="text/css"/>
</head>
```
</details>

<details>
<summary>nav.html</summary>

```html
<nav>
    <div>
      <a href="/" class="brand-logo">Home</a>
      <ul id="nav-mobile" class="right">
        <li><a href="/favourites">Favourites</a></li>    
      </ul>
    </div>
  </nav>
```
</details>

Now you are all done with your partials!

### Home Page

Now lets move back up to our `templates/` directory. Make a `templates/index.html` file and be sure to include the `partials/head.html` and `partials/nav.html` files that we just made.

<details>
<summary>If you forget how to include those elements</summary>
 
```html
<!DOCTYPE html>
<html>
    {% include 'partials/head.html' %}
    <body>
        {% include 'partials/nav.html' %}
    </body>
</html>
```
</details>

#### Body

We have a `nav` element but our page is looking a little empty... Let's fix that by adding in a name for our library and some books!

Lets put a `h1` element with the name of your library right below the `nav` element.
Below this header, add a `div` with `class="shelf"`. We are going to use `Jinja` to use `python` syntax to loop through and get the approriate books. If you think you are a `Jinja` wizard give it a shot, but otherwise here it is:

<details>
<summary>Your bookshelf</summary>

```html
<div class="shelf">
    {% for book in results %}
    {% if 'imageLinks' in book.volumeInfo %}
    <div class="book">
        <div class="options">
            <p data-id="{{book.id}}" class="delete"><i class="material-icons">delete</i></p>
            <p data-id="{{book.id}}" class="add"><i class="material-icons">add_circle</i></p>
        </div>
        <img src={{ book.volumeInfo.imageLinks.thumbnail }}>
    </div>
    {% endif %}
    {% endfor %}
</div>
```
</details>


But wait, we are missing a way to look for books! (What kind of librarian would let this happen...)

##### Search-bar

Under the `h1` element, add a `div` this element will contain a form that sends a post request to `/` that will call the flask function that we made. Put an `input` element and a submit `button` in that form and you are good to go!

<details>
<summary>Those elements might look like this</summary>

```html
<h1>Hot Cocoa</h1>
<div>
    <form action="/" method="POST">
        <input type="text" name="search-bar" placeholder="book title">
        <button type="submit">Search</button>
    </form>
</div>
```
</details>

##### Script

We also want to keep track of our favorite books or books that we might want to read in the future! Lets add a `script` element below the divs that we just made.

This script will contain two functions one to add a book to our favorites list and one to delete it:


<details>
<summary>Script element</summary>
 
```html
<script>
   $('.add').click(function(event) {
     var id=$(event.currentTarget).data('id');
       $.ajax({
           type: "POST",
           url: "/add/"+id,
       }).done(function() {location.reload()});
   });
   $('.delete').click(function(event) {
     var id=$(event.currentTarget).data('id');
       $.ajax({
           type: "DELETE",
           url: "/delete/"+id,
       }).done(function() {location.reload()});
   });
</script>
 ```
</details

Now you've finished your index.html!

<details>
<summary>Your templates/index.html should look like this!</summary>

```html
<!DOCTYPE html>
<html>
    {% include 'partials/head.html' %}
    <body>
        {% include 'partials/nav.html' %}
        <h1>Hot Cocoa</h1>
        <div>
            <form action="/" method="POST">
                <input type="text" name="search-bar" placeholder="book title">
                <button type="submit">Search</button>
            </form>
        </div>
        <div class="shelf">
            {% for book in results %}
            {% if 'imageLinks' in book.volumeInfo %}
            <div class="book">
                <div class="options">
                    <p data-id="{{book.id}}" class="delete"><i class="material-icons">delete</i></p>
                    <p data-id="{{book.id}}" class="add"><i class="material-icons">add_circle</i></p>
                </div>
                <img src={{ book.volumeInfo.imageLinks.thumbnail }}>
            </div>
            {% endif %}
            {% endfor %}
        </div>
        <script>
            $('.add').click(function(event) {
              var id=$(event.currentTarget).data('id');
                $.ajax({
                    type: "POST",
                    url: "/add/"+id,
                }).done(function() {location.reload()});
            });
            $('.delete').click(function(event) {
              var id=$(event.currentTarget).data('id');
                $.ajax({
                    type: "DELETE",
                    url: "/delete/"+id,
                }).done(function() {location.reload()});
            });
        </script>
    </body>
</html>
```
</details>

### Favorites Page

Sweet, now that you have the home page laid out, its time to make a page to display your favorite books!
Create a new html file, `templates/favs.html`.

#### Body

The `head` and `nav` parts of this page will be the same as the `index.html` so you can start there.

<details>
<summary>If you already forgot...</summary>

```html
<!DOCTYPE html>
<html>
    <head>
        {% include 'partials/head.html' %}
    </head>
    <body>
        {% include 'partials/nav.html' %}
    </body>
</html>
```
</details>

In our body, below the nav, add a header introducing your favorites.

Below that, we are going to use `Jinja` again to loop through and show all of your favorited books.

If you are not super comfortable with `Jinja` yet, we will give you some code:

<details>
<summary>Your body will look like this</summary>

```html
    <body>
        {% include 'partials/nav.html' %}
        <h1>Welcome to your favourites</h1>
        <div class="shelf">
            {% for book in all %}
            <div class="book">
                <div class="options">
                    <p data-id="{{book.id}}" class="delete"><i class="material-icons">delete</i></p>
                </div>
                <img src={{ book.volumeInfo.imageLinks.thumbnail }}>
            </div>
            {% endfor %}
        </div>
    </body>
```
</details>

Essentially, the `Jinja` control flow statements `{% for book in all %}` and `{% endfor %}` let us use `python` syntax to run a `for` loop to load multiple elements.

:boom: Cool, right? Yeah I agree!

#### Script

Now that we can display our favorite books, let's add a script that will allow us to remove books from that list as well.

Below your `<body>` add a `<script>` element. 
We are going to use Ajax to call our python function that deletes elements from our database by using the route established we established in our `app.py`.

<details>
<summary>Your script should look like this!</summary>

```html
    <script>
        $('.delete').click(function(event) {
          var id=$(event.currentTarget).data('id');
            $.ajax({
                type: "DELETE",
                url: "/delete/"+id,
            }).done(function() {location.reload()});
        });
    </script>
```
</details>

Finally:

<details>
<summary>Your favs.html should look like this</summary>

```html
<!DOCTYPE html>
<html>
    <head>
        {% include 'partials/head.html' %}
    </head>
    <body>
        {% include 'partials/nav.html' %}
        <h1>Welcome to your favourites</h1>
        <div class="shelf">
            {% for book in all %}
            <div class="book">
                <div class="options">
                    <p data-id="{{book.id}}" class="delete"><i class="material-icons">delete</i></p>
                </div>
                <img src={{ book.volumeInfo.imageLinks.thumbnail }}>
            </div>
            {% endfor %}
        </div>
    </body>
    <script>
        $('.delete').click(function(event) {
          var id=$(event.currentTarget).data('id');
            $.ajax({
                type: "DELETE",
                url: "/delete/"+id,
            }).done(function() {location.reload()});
        });
    </script>
</html>
```
</details>

### Static

Now, make a 'static/' directory. In that directory, add a 'static/style.css' file and add the styling below into there. 

<details>
 <summary>For brevity's sake, here is some styling:</summary>

```css
body {
    background-color: rgb(26, 19, 0);
    color: rgb(255, 255, 255);
    margin: 0 10px;
}
nav, button {
    background-color: rgb(105, 6, 13);
    border-width: 0;
    border-radius: 8px;
    padding: 7px;
}
h1 {
    font-size: 25px;
}
i {
    cursor: pointer;
}
.shelf {
    display: flex;
    flex-flow: row wrap;
}
.book {
    padding: 15px;
}
.options {
    display: flex;
}
```
</details>

### Deployment
If you're curious about how to deploy your new Flask app, you can easily deploy it with heroku as well. You'll need a few things:
1. A Procfile that just contains this: web: gunicorn app:app
2. A requirements.txt file. Generate this with this command:
```console
pip freeze > requirements.txt
```
3. In db.py, modify your client to look like this:
```python
uri = os.environ.get('MONGODB_URI', 'mongodb://localhost')
client = MongoClient(uri)
```
4. After deploying (and adding mlab as a resource), go to your config vars and add this to the end of your MONGODB_URI config var (?authSource={database_name}&retryWrites=false)
5. Finally, run it with `Python app.py`.

All done!

:checkered_flag: If all went well, your page should look something like this:

![final](img/final.png)

Congrats! You deployed your first Flask app!

## Summary / What you Learned

* [x] Deploy a web app using the Flask framework
* [x] Using Mongodb to store data with the pymongo package
* [x] Routing with an API in Flask

## Reflection

* [ ] Whats the difference between Django, Flask, and Ruby on Rails?
* [ ] When might you use each of those frameworks?
