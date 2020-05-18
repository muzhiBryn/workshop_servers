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

Then we're going to build a library. A...what? A ibrary. Except you can't read the books, just look at them. But that's fun too.

## Setup

Let's get all the tools we need. Head to the terminal.
1. Flask and Pymongo. Run
```console
pip install Flask
python -m pip install pymongo
```

2. If you read Flask tutorials, you'll see that a lot of them use virtual environments. We will not be doing that in this tutorial. The benefit of virtual environments is that it you can work on isolated projects with different dependencies at the same time. So, if you are already using a different version of flask, feel free to use virtual environments!
3. Get an API key for Google Books. We've done this a bunch of times, from YouTube to the ML tutorial. Go [here](https://console.developers.google.com/), in APIs and Services, search for Books. Enable the API. Then to to Credentials. Click on Create Credentials, select api key, restrict it to the Books API, then write it down somewhere.
4. Phew. Now we're ready.

## Step by Step

Much of the code we write here will look familiar to Express. Keep an eye out for the similarities, it'll help in remembering. We'll help you too, with this: :smiling_imp:

### File Structure
Create a directory to work in. Name it whatever you want. We're going to create two folders and two files.
1. Folder one is called static. This will hold CSS and other static files.
2. Folder two is called templates. This will hold our HTML. It's like views in Express :smiling_imp:
3. File one is called app.py. It's like server.js :smiling_imp:
4. File two is called db.py. Yep, you guessed it, it's our model and controller rolled into one. :smiling_imp:

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
bookshelf = db.bookshelf
 ```
</details>
The variable names could be literally anything. In our example, if you open up MongoDB Compass (or use the CLI), you'll find that there is now a database called database, and inside that, there is a collection called bookshelf.

And that's it for the model. We're not going to create a Schema. We're just going to go ahead and throw data into it.

#### Controller
We're still in db.py. We want the functions that will directly manipulate our database now. Can you think of any? Well, we're going to be searching for books with the Books API, and then we'll add those to our library. We'll also want to be able to kick books out of our library. Yep, add and kick functions. Or delete, if you want to be boring. Mongoose, I mean, pymongo, has methods called insert_one, and delete_one. :smiling_imp: They behave like pymongo, I mean, mongoose.

<details>
 <summary>Here, in case</summary>

 ```python
def add_book(book):
  bookshelf.insert_one(book)
```
```python
def delete_book(id):
    bookshelf.delete_one({'_id': id})
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
This is the fun part. Routing in Flask works like routing in express, except that you are required to return something to the view, even if you just perform an action like delete. For that case, there's a special empty return statement ('', 204) that translates to "all good. nothing to see here". In this tutorial we will construct four routes, only two of which will be accessible by the user.
1. Home page. This is where the search bar will be.
2. Book shelf, or library, or favourites.
3. add/<:id> :smiling_imp:
4. delete/<:id> :smiling_imp:

##### Home page
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

##### Bookshelf
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

##### add/<:id>
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

##### delete/<:id>
<details>
 <summary>try it first</summary>

 ```python
@app.route('/', methods=['GET', 'POST'])
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

### Finally

If all went well, it should look something like this:

![final](img/final.png)

Congrats! You deployed your first Flask app!

## Summary / What you Learned

* [ ] Deploy a web app using the Flask framework
* [ ] Using Mongodb to store data with the pymongo package
* [ ] Routing with an API in Flask

## Reflection

* [ ] Whats the difference between Django, Flask, and Ruby on Rails?
* [ ] When would you use each of these above frameworks?
