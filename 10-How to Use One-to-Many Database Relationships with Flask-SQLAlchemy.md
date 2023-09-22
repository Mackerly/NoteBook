# How to Use One-to-Many Database Relationships with Flask-SQLAlchemy
*The author selected the [Free and Open Source Fund](https://www.brightfunds.org/funds/foss-nonprofits) to receive a donation as part of the [Write for DOnations](https://do.co/w4do-cta) program.*

## Introduction

[Flask](https://palletsprojects.com/p/flask/) is a lightweight Python web framework that provides useful tools and features for creating web applications in the Python Language. [SQLAlchemy](https://www.sqlalchemy.org/) is an SQL toolkit that provides efficient and high-performing database access for relational databases. It provides ways to interact with several database engines such as SQLite, MySQL, and PostgreSQL. It gives you access to the database's SQL functionalities. And it also gives you an Object Relational Mapper (ORM), which allows you to make queries and handle data using simple Python objects and methods. [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/) is a Flask extension that makes using SQLAlchemy with Flask easier, providing you tools and methods to interact with your database in your Flask applications through SQLAlchemy.

A _one-to-many database relationship_ is a relationship between two database tables where a record in one table can reference several records in another table. For example, in a blogging application, a table for storing posts can have a one-to-many relationship with a table for storing comments. Each post can reference many comments, and each comment references a single post; therefore, one post has a relationship with many comments. The post table is a _parent table_, while the comments table is a _child table_ — a record in the parent table can reference many records in the child table. This relationship is important to enable access to related data in each table.

In this tutorial, you’ll build a small blogging system that demonstrates how to build one-to-many relationships using the Flask-SQLAlchemy extension. You'll create a relationship between posts and comments, where each blog post can have several comments.

## Prerequisites

* A local Python 3 programming environment. Follow the tutorial for your distribution in [How To Install and Set Up a Local Programming Environment for Python 3](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-python-3) series. In this tutorial we’ll call our project directory `flask_app`.

* An understanding of basic Flask concepts, such as routes, view functions, and templates. If you are not familiar with Flask, check out [How to Create Your First Web Application Using Flask and Python](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-web-application-using-flask-and-python-3) and [How to Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

* An understanding of basic HTML concepts. You can review our [How To Build a Website with HTML](https://www.digitalocean.com/community/tutorial_series/how-to-build-a-website-with-html) tutorial series for background knowledge.

## Step 1 — Installing Flask and Flask-SQLAlchemy
In this step, you'll install the necessary packages for your application.

With your virtual environment activated, use `pip` to install Flask and Flask-SQLAlchemy:

```custom_prefix((env)sammy@localhost:$)
pip install Flask Flask-SQLAlchemy
```

Once the installation is successfully finished, you’ll see a line similar to the following at the end of the output:

```
[secondary_label Output]
Successfully installed Flask-2.1.1 Flask-SQLAlchemy-2.5.1 Jinja2-3.1.1 MarkupSafe-2.1.1 SQLAlchemy-1.4.35 Werkzeug-2.1.1 click-8.1.2 greenlet-1.1.2 itsdangerous-2.1.2
```

With the required Python packages installed, you'll set up the database next.

## Step 2 — Setting up the Database and Models
In this step, you'll set up your database, and create SQLAlchemy _database models_ — Python classes that represent your database tables. You'll create a model for your blog posts and a model for comments. You'll initiate the database, create a table for posts, and add a table for comments based on the models you'll declare. You'll also insert a few posts and comments into your database.

### Setting up The Database Connection

Open a file called `app.py` in your `flask_app` directory. This file will have code for setting up the database and your Flask routes:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

This file will connect to an SQLite database called `database.db`, and will have two classes: A class called `Post` that represents your database posts table, and a `Comment` class representing the comments table. This file will also contain your Flask routes. Add the following `import` statements at the top of `app.py`:

```python
[label flask_app/app.py]
import os
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy

```

Here, you import the [`os` module](https://docs.python.org/3/library/os.html), which gives you access to miscellaneous operating system interfaces. You'll use it to construct a file path for your `database.db` database file.

From the `flask` package, you then import the necessary helpers you need for your application: the `Flask` class to create a Flask application instance, the `render_template()` function to render templates, the `request` object to handle requests, the `url_for()` function to construct URLs for routes, and the `redirect()` function for redirecting users. For more information on routes and templates, see [How To Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

You then import the `SQLAlchemy` class from the Flask-SQLAlchemy extension, which gives you access to all the functions and classes from SQLAlchemy, in addition to helpers, and functionality that integrates Flask with SQLAlchemy. You'll use it to create a database object that connects to your Flask application, allowing you to create and manipulate tables using Python classes, objects, and functions without needing to use the SQL language.

Below the imports, you'll set up a database file path, instantiate your Flask application, and configure and connect your application with SQLAlchemy. Add the following code:

```python
[label flask_app/app.py]

basedir = os.path.abspath(os.path.dirname(__file__))

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] =\
           'sqlite:///' + os.path.join(basedir, 'database.db')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False


db = SQLAlchemy(app)
```

Here, you construct a path for your SQLite database file. You first define a base directory as the current directory. You use the [`os.path.abspath()`](https://docs.python.org/3.8/library/os.path.html#os.path.abspath) function to get the absolute path of the current file's directory. The special `__file__` variable holds the pathname of the current `app.py` file. You store the absolute path of the base directory in a variable called `basedir`.

You then create a Flask application instance called `app`, which you use to configure two Flask-SQLAlchemy [configuration keys](https://flask-sqlalchemy.palletsprojects.com/en/2.x/config/):

* `SQLALCHEMY_DATABASE_URI`: The database URI to specify the database you want to establish a connection with. In this case, the URI follows the format `sqlite:///path/to/database.db`. You use the [`os.path.join()`](https://docs.python.org/3.8/library/os.path.html#os.path.join) function to intelligently join the base directory you constructed and stored in the `basedir` variable, and the `database.db` file name. This will connect to a `database.db` database file in your `flask_app` directory. The file will be created once you initiate the database.

* `SQLALCHEMY_TRACK_MODIFICATIONS`: A configuration to enable or disable tracking modifications of objects. You set it to `False` to disable tracking and use less memory. For more, see [the configuration page](https://flask-sqlalchemy.palletsprojects.com/en/2.x/config/) in the Flask-SQLAlchemy documentation.

[note]
**Note:** If you want to use another database engine such as PostgreSQL or MySQL, you'll need to use the proper URI.

For PostgreSQL, use the following format:
```
postgresql://username:password@host:port/database_name
```

For MySQL:
```
mysql://username:password@host:port/database_name
```

For more, see the [SQLAlchemy documentation for engine configuration](https://docs.sqlalchemy.org/en/14/core/engines.html).


After configuring SQLAlchemy by setting a database URI and disabling tracking, you create a database object using the `SQLAlchemy` class, passing the application instance to connect your Flask application with SQLAlchemy. You store your database object in a variable called `db`. You'll use this `db` object to interact with your database.

### Declaring The Tables

With the database connection established and the database object created, you'll use the database object to create a database table for posts and one for comments. Tables are represented by a _model_ — a Python class that inherits from a base class Flask-SQLAlchemy provides through the `db` database instance you created earlier. To define the posts and comments tables as models, add the following two classes to your `app.py` file:

```python
[label flask_app/app.py]

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    content = db.Column(db.Text)
    comments = db.relationship('Comment', backref='post')

    def __repr__(self):
        return f'<Post "{self.title}">'


class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text)
    post_id = db.Column(db.Integer, db.ForeignKey('post.id'))

    def __repr__(self):
        return f'<Comment "{self.content[:20]}...">'
```

Here, you create a `Post` model and a `Comment` model, which inherit from the `db.Model` class.

The `Post` model represents the post table. You use the [`db.Column`](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) class to define its columns. The first argument represents the column type, and additional arguments represent the column configuration.

You define the following columns for the `Post` model:

* `id`: The post ID. You define it as an integer with `db.Integer`. `primary_key=True` defines this column as a _primary key_, which will assign it a unique value by the database for each entry (that is, each post).
* `title`: The post's title. A string with a maximum length of 100 characters.
* `content`: The post's content. `db.Text` indicates the column holds long texts.

The `comments` class attribute defines a One-to-Many relationship between the `Post` model and the `Comment` model. You use the `db.relationship()` method, passing it the name of the comments model (`Comment` in this case). You use the `backref` parameter to add a back reference that behaves like a column to the `Comment` model. This way, you can access the post the comment was posted on using a `post` attribute. For example, if you have a comment object in a variable called `comment`, you will be able to access the post the comment belongs to using `comment.post`. You'll see an example demonstrating this later.

See the [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/14/core/type_basics.html) for column types other than the types you used in the preceding code block.

The special [`__repr__`](https://docs.python.org/3/reference/datamodel.html#object.__repr__) function allows you to give each object a string representation to recognize it for debugging purposes.

The `Comment` model represents the comment table. You define the following columns for it:

* `id`: The comment ID. You define it as an integer with `db.Integer`. `primary_key=True` defines this column as a _primary key_, which will assign it a unique value by the database for each entry (that is, each comment).
* `content`: The comment's content. `db.Text` indicates the column holds long texts.
* `post_id`: An integer _foreign key_ you construct using the `db.ForeignKey()` class, which is a key that links a table with another, using that table's primary key. This links a comment to a post using the primary key of the post, which is its ID. Here, the `post` table is a _parent table_, which indicates that each post has many comments. The `comment` table is a _child table_. Each comment is related to a parent post using the post's ID. Therefore, each comment has a `post_id` column that can be used to access the post the comment was posted on.

The special [`__repr__`](https://docs.python.org/3/reference/datamodel.html#object.__repr__) function in the `Comment` model shows the first 20 characters of the comment's content to give a comment object a short string representation.

The `app.py` file will now look as follows:

```python
[label flask_app/app.py]
import os
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] =\
           'sqlite:///' + os.path.join(basedir, 'database.db')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False


db = SQLAlchemy(app)


class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    content = db.Column(db.Text)
    comments = db.relationship('Comment', backref='post')

    def __repr__(self):
        return f'<Post "{self.title}">'


class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text)
    post_id = db.Column(db.Integer, db.ForeignKey('post.id'))

    def __repr__(self):
        return f'<Comment "{self.content[:20]}...">'
```

Save and close `app.py`.

### Creating the Database

Now that you've set the database connection and the post and comment models, you’ll use the Flask shell to create your database and your post and comment tables based on the models you declared.

With your virtual environment activated, set the `app.py` file as your Flask application using the `FLASK_APP` environment variable:

```custom_prefix((env)sammy@localhost:$)
export FLASK_APP=app
```
Then open the Flask shell using the following command in your `flask_app` directory:

```custom_prefix((env)sammy@localhost:$)
flask shell
```

A Python interactive shell will be opened. This special shell runs commands in the context of your Flask application, so that the Flask-SQLAlchemy functions you'll call are connected to your application.

Import the database object and the post and comment models, and then run the `db.create_all()` function to create the tables that are associated with your models:

```custom_prefix(>>>)
from app import db, Post, Comment
db.create_all()
```

Leave the shell running, open another terminal window and navigate to your `flask_app` directory. You will now see a new file called `database.db` in `flask_app`.

[note]
**Note:** The `db.create_all()` function does not recreate or update a table if it already exists. For example, if you modify your model by adding a new column, and run the `db.create_all()` function, the change you make to the model will not be applied to the table if the table already exists in the database. The solution is to delete all existing database tables with the `db.drop_all()` function and then recreate them with the `db.create_all()` function like so:

```custom_prefix(>>>)
db.drop_all()
db.create_all()
```

This will apply the modifications you make to your models, but will also delete all the existing data in the database. To update the database structure and preserve existing data, you'll need to use [_schema migration_](https://en.wikipedia.org/wiki/Schema_migration), which allows you to modify your tables and preserve data. You can use the [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/index.html) extension to perform SQLAlchemy schema migrations through the Flask command-line interface.


If you receive an error, make sure your database URI and your model declaration are correct.

### Populating the Tables

After creating the database and the post and comment tables, you'll create a file in your `flask_app` directory to add some posts and comments to your database.

Open a new file called `init_db.py`:

```custom_prefix((env)sammy@localhost:$)
nano init_db.py
```

Add the following code to it. This file will create three post objects and four comment objects, and add them to the database:

```python
[label flask_app/init_db.py]
from app import db, Post, Comment

post1 = Post(title='Post The First', content='Content for the first post')
post2 = Post(title='Post The Second', content='Content for the Second post')
post3 = Post(title='Post The Third', content='Content for the third post')

comment1 = Comment(content='Comment for the first post', post=post1)
comment2 = Comment(content='Comment for the second post', post=post2)
comment3 = Comment(content='Another comment for the second post', post_id=2)
comment4 = Comment(content='Another comment for the first post', post_id=1)


db.session.add_all([post1, post2, post3])
db.session.add_all([comment1, comment2, comment3, comment4])

db.session.commit()
```

Save and close the file.

Here, you import the database object, the `Post` model, and the `Comment` model from the `app.py` file.

You create a few post objects using the `Post` model, passing the post's title to the `title` parameter and the post's content to the `content` parameter.

You then create a few comment objects, passing the comment's content. You have two methods you can use to associate a comment with the post it belongs to. You can pass the post object to the `post` parameter as demonstrated in the `comment1` and `comment2` objects. And you can also pass the post ID to the `post_id` parameter, as demonstrated in the `comment3` and `comment4` objects. So you can just pass the integer ID of the post if you don't have the post object in your code.

After defining the post and comment objects, you use the `db.session.add_all()` to add all post and comment objects to the database session, which manages transactions. Then you use the `db.session.commit()` method to commit the transaction and apply the changes to the database. For more on SQLAlchemy database sessions, see step 2 of the [How to Use Flask-SQLAlchemy to Interact with Databases in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-flask-sqlalchemy-to-interact-with-databases-in-a-flask-application#step-2-setting-up-the-database-and-model) tutorial.

Run the `init_db.py` file to execute the code and add the data to the database:

```custom_prefix((env)sammy@localhost:$)
python init_db.py
```

To take a look at the data you added to your database, open the flask shell to query all posts and display their titles and the content of each post's comments:

```custom_prefix((env)sammy@localhost:$)
flask shell
```

Run the following code. This queries all posts and displays each post title and the comments of each post below it:

```python
from app import Post

posts = Post.query.all()

for post in posts:
    print(f'## {post.title}')
    for comment in post.comments:
            print(f'> {comment.content}')
    print('----')
```

Here, you import the `Post` model from the `app.py` file. You query all the posts that exist in the database using the `all()` method on the `query` attribute, and save the result in a variable called `posts`. Then you use a `for` loop to go through each item in the `posts` variable. You print the title and then use another `for` loop to go through each comment belonging to the post. You access the post's comments using `post.comments`. You print the comment's content and then print the string `'----'` to separate between posts.

You'll get the following output:

```
[secondary_label Output]

## Post The First
> Comment for the first post
> Another comment for the first post
----
## Post The Second
> Comment for the second post
> Another comment for the second post
----
## Post The Third
----
```

As you can see, you can access the data of each post and the comments of each post with very little code.

Now exit the shell:

```custom_prefix(>>>)
exit()
```

At this point, you have several posts and comments in your database. Next, you'll create a Flask route for the index page and display all of the posts in your database on it.

## Step 3 — Displaying All Posts
In this step, you'll create a route and a template to display all the posts in the database on the index page.

Open your `app.py` file to add a route for the index page to it:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route at the end of the file:

```python
[label flask_app/app.py]

# ...

@app.route('/')
def index():
    posts = Post.query.all()
    return render_template('index.html', posts=posts)
```

Save and close the file.

Here, you create an `index()` view function using the `app.route()` decorator. In this function, you query the database and get all the posts like you did in the previous step. You store the query result in a variable called `posts` and then you pass it to a `index.html` template file you render using the `render_template()` helper function.

Before you create the `index.html` template file on which you'll display the existing posts in the database, you'll first create a base template, which will have all the basic HTML code other templates will also use to avoid code repetition. Then you’ll create the `index.html` template file you rendered in your `index()` function. To learn more about templates, see [How to Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

Create a `templates` directory, then open a new template called `base.html`:

```custom_prefix((env)sammy@localhost:$)
mkdir templates
nano templates/base.html
```

Add the following code inside the `base.html` file:

```html
[label flask_app/templates/base.html]

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %} - FlaskApp</title>
    <style>
        .title {
            margin: 5px;
        }

        .content {
            margin: 5px;
            width: 100%;
            display: flex;
            flex-direction: row;
            flex-wrap: wrap;
        }

        .comment {
            padding: 10px;
            margin: 10px;
            background-color: #fff;
        }

        .post {
            flex: 20%;
            padding: 10px;
            margin: 5px;
            background-color: #f3f3f3;
            inline-size: 100%;
        }

        .title a {
            color: #00a36f;
            text-decoration: none;
        }

        nav a {
            color: #d64161;
            font-size: 3em;
            margin-left: 50px;
            text-decoration: none;
        }

    </style>
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">FlaskApp</a>
        <a href="#">Comments</a>
        <a href="#">About</a>
    </nav>
    <hr>
    <div class="content">
        {% block content %} {% endblock %}
    </div>
</body>
</html>
```

Save and close the file.

This base template has all the HTML boilerplate you’ll need to reuse in your other templates. The `title` block will be replaced to set a title for each page, and the `content` block will be replaced with the content of each page. The navigation bar has three links: one for the index page, which links to the `index()` view function using the `url_for()` helper function, one for a **Comments** page, and one for an **About** page if you choose to add one to your application. You'll edit this file later after you add a page for displaying all the latest comments to make the **Comments** link functional.

Next, open a new `index.html` template file. This is the template you referenced in the `app.py` file:

```custom_prefix((env)sammy@localhost:$)
nano templates/index.html
```

Add the following code to it:

```html
[label flask_app/templates/index.html]
{% extends 'base.html' %}

{% block content %}
    <span class="title"><h1>{% block title %} Posts {% endblock %}</h1></span>
    <div class="content">
        {% for post in posts %}
            <div class="post">
                <p><b>#{{ post.id }}</b></p>
                <b>
                    <p class="title">
                        <a href="#">
                            {{ post.title }}
                        </a>
                    </p>
                </b>
                <div class="content">
                    <p>{{ post.content }}</p>
                </div>
                <hr>
            </div>
        {% endfor %}
    </div>
{% endblock %}
```

Save and close the file.

Here, you extend the base template and replace the contents of the content block. You use an `<h1>` heading that also serves as a title. You use a [Jinja `for` loop](https://jinja.palletsprojects.com/en/3.0.x/templates/#for) in the line `{% for post in posts %}` to go through each post in the `posts` variable that you passed from the `index()` view function to this template. You display the post ID, its title, and the post content. The post title will later link to a page that displays the individual post and its comments.

While in your `flask_app` directory with your virtual environment activated, tell Flask about the application (`app.py` in this case) using the `FLASK_APP` environment variable. Then set the `FLASK_ENV` environment variable to `development` to run the application in development mode and get access to the debugger. For more information about the Flask debugger, see [How To Handle Errors in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-handle-errors-in-a-flask-application). Use the following commands to do this:

```custom_prefix((env)sammy@localhost:$)
export FLASK_APP=app
export FLASK_ENV=development
```

Next, run the application:

```custom_prefix((env)sammy@localhost:$)
flask run
```

With the development server running, visit the following URL using your browser:

```
http://127.0.0.1:5000/
```

You'll see the posts you added to the database in a page similar to the following:

![Index Page](https://assets.digitalocean.com/articles/68194/ibgcWby.png)

You've displayed the posts you have in your database on the index page. Next, you'll create a route for a post page, where you will display the details of each post and its comments below it.

## Step 4 — Displaying a Single Post and its Comments

In this step, you'll create a route and a template to display the details of each post on a dedicated page, and the post's comments below it.

By the end of this step, the URL `http://127.0.0.1:5000/1` will be a page that displays the first post (because it has the ID `1`) and its comments. The URL `http://127.0.0.1:5000/ID` will display the post with the associated `ID` number, if it exists.

Leave the development server running and open a new terminal window.

Open `app.py` for modification:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route at the end of the file:

```python
[label flask_app/app.py]
# ...

@app.route('/<int:post_id>/')
def post(post_id):
    post = Post.query.get_or_404(post_id)
    return render_template('post.html', post=post)
```

Save and close the file.

Here, you use the route `'/<int:post_id>/'`, with `int:` being a [converter](https://flask.palletsprojects.com/en/2.0.x/quickstart/#variable-rules) that converts the default string in the URL into an integer. `post_id` is the URL variable that will determine the post you'll display on the page.

The ID is passed from the URL to the `post()` view function through the `post_id` parameter. Inside the function, you query the post table and retrieve a post by its ID using the `get_or_404()` method. This will save the post data in the `post` variable if it exists, and respond with a `404 Not Found` HTTP error if no post with the given ID exists in the database.

You render a template called `post.html` and pass it the post you retrieved.

Open this new `post.html` template file:

```custom_prefix((env)sammy@localhost:$)
nano templates/post.html
```

Type the following code in it. This will be similar to the `index.html` template, except that it will only display a single post:

```html
[label flask_app/templates/post.html]
{% extends 'base.html' %}

{% block content %}
    <span class="title"><h1>{% block title %} {{ post.title }}  {% endblock %}</h1></span>
    <div class="content">
            <div class="post">
                <p><b>#{{ post.id }}</b></p>
                <b>
                    <p class="title">{{ post.title }}</p>
                </b>
                <div class="content">
                    <p>{{ post.content }}</p>
                </div>
                <hr>
                <h3>Comments</h3>
                {% for comment in post.comments %}
                    <div class="comment">
                        <p>#{{ comment.id }}</p>
                        <p>{{ comment.content }}</p>
                    </div>
                {% endfor %}
            </div>
    </div>
{% endblock %}
```

Save and close the file.

Here, you extend the base template, set the post title as a page title, display the post ID, post title, and the post content. Then, you go through the post comments available via `post.comments`. You display the comment ID, and the contents of the comment.


Use your browser to navigate to the URL for the second post:

```
http://127.0.0.1:5000/2/
```

You'll see a page similar to the following:

![Single Post Page](https://assets.digitalocean.com/articles/68194/Rc0s0NM.png)

Next, edit `index.html` to make the title of the post link to the individual post:

```custom_prefix((env)sammy@localhost:$)
nano templates/index.html
```

Edit the value of the `href` attribute of the post title's link inside the `for` loop:

```html
[label flask_app/templates/index.html]

...

{% for post in posts %}
    <div class="post">
        <p><b>#{{ post.id }}</b></p>
        <b>
            <p class="title">
                <a href="{{ url_for('post', post_id=post.id)}}">
                {{ post.title }}
                </a>
            </p>
        </b>
        <div class="content">
            <p>{{ post.content }}</p>
        </div>
        <hr>
    </div>
{% endfor %}
```

Save and close the file.

Navigate to your index page or refresh it:

```
http://127.0.0.1:5000/
```

Click on each of the post titles on the index page. You'll now see that each post links to the proper post page.

You've now created a page for displaying individual posts. Next, you'll add a web form to the post page to allow users to add new comments.

## Step 5 — Adding New Comments
In this step, you'll edit the `/<int:post_id>/` route and its `post()` view function, which handles displaying an individual post. You'll add a web form below each post to allow users to add comments to that post, then you'll handle the comment submission and add it to the database.

First, open the `post.html` template file to add a web form consisting of a text area for the comment's content, and an **Add Comment** submit button.

```custom_prefix((env)sammy@localhost:$)
nano templates/post.html
```

Edit the file by adding a form below the `Comments` H3 heading, and directly above the `for` loop:

```html
[label flask_app/templates/post.html]

<hr>
<h3>Comments</h3>
<form method="post">
    <p>
        <textarea name="content"
                    placeholder="Comment"
                    cols="60"
                    rows="5"></textarea>
    </p>

    <p>
        <button type="submit">Add comment</button>
    </p>
</form>
{% for comment in post.comments %}
```

Save and close the file.

Here, you add a `<form>` tag with the attribute `method` set to `post` to indicate that the form will submit a POST request.

You have a text area for the comment's content, and a submit button.

With the development server running, use your browser to navigate to a post:

```
http://127.0.0.1:5000/2/
```

You'll see a page similar to the following:

![Single Post Page with Comment Form](https://assets.digitalocean.com/articles/68194/UJ9b7ki.png)

This form sends a POST request to the `post()` view function, but because there is no code to handle the form submission, the form currently does not work.

Next, you will add code to the `post()` view function to handle the form submission and add the new comment to the database. Open `app.py` to handle the POST request the user submits:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Edit the `/<int:post_id>/` route and its `post()` view function to look as follows:

```python
[label flask_app/app.py]

@app.route('/<int:post_id>/', methods=('GET', 'POST'))
def post(post_id):
    post = Post.query.get_or_404(post_id)
    if request.method == 'POST':
        comment = Comment(content=request.form['content'], post=post)
        db.session.add(comment)
        db.session.commit()
        return redirect(url_for('post', post_id=post.id))

    return render_template('post.html', post=post)
```
Save and close the file.

You allow both GET and POST requests using the `methods` parameter. GET requests are used to retrieve data from the server. POST requests are used to post data to a specific route. By default, only GET requests are allowed.

Inside the `if request.method == 'POST'` condition, you handle the POST request the user will submit via the form. You create a comment object using the `Comment` model, passing it the content of the submitted comment which you extract from the `request.form` object. You specify the post the comment belongs to using the `post` parameter, passing it the `post` object you retrieved using the post ID, with the `get_or_404()` method.

You add the comment object you constructed to the database session, commit the transaction, and redirect to the post page.

Now refresh the post page on your browser, write a comment, and submit it. You'll see your new comment below the post.

You now have a web form that allows users to add comments to a post. For more on web forms, see [How To Use Web Forms in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-web-forms-in-a-flask-application). For a more advanced and more secure method of managing web forms, see [How To Use and Validate Web Forms with Flask-WTF](https://www.digitalocean.com/community/tutorials/how-to-use-and-validate-web-forms-with-flask-wtf). Next, you'll add a page that displays all the comments in the database and the posts they were posted on.

## Step 6 — Displaying All Comments
In this step, you'll add a **Comments** page where you will display all the comments in the database, ordering them by displaying the newest comments first. Each comment will have the title and link of the post the comment was posted on.

Open `app.py`:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route to the end of the file. This fetches all the comments in the database, ordered by the latest first. It then passes them to a template file called `comments.html`, which you'll create later:

```python
[label flask_app/app.py]
# ...

@app.route('/comments/')
def comments():
    comments = Comment.query.order_by(Comment.id.desc()).all()
    return render_template('comments.html', comments=comments)
```
Save and close the file.

You use the `order_by()` method on the `query` attribute to fetch all the comments in a specific order. In this case you use the `desc()` method on the `Comment.id` column to fetch comments in descending order, with the latest comments being first. Then you use the `all()` method to get the result and save it to a variable called `comments`.

You render a template called `comments.html`, passing it the `comments` object which contains all comments ordered by the latest first.

Open this new `comments.html` template file:

```custom_prefix((env)sammy@localhost:$)
nano templates/comments.html
```

Type the following code inside it. This will display the comments and link to the post they belong to:

```html
[label flask_app/templates/comments.html]
{% extends 'base.html' %}

{% block content %}
    <span class="title"><h1>{% block title %} Latest Comments {% endblock %}</h1></span>
    <div class="content">
                {% for comment in comments %}
                    <div class="comment">
                        <i>
                            (#{{ comment.id }})
                            <p>{{ comment.content }}</p>
                        </i>
                        <p class="title">
                        On <a href="{{ url_for('post',
                                                post_id=comment.post.id) }}">
                                {{ comment.post.title }}
                              </a>
                        </p>
                    </div>
                {% endfor %}
            </div>
    </div>
{% endblock %}
```

Save and close the file.

Here, you extend the base template, set a title, and go through the comments using a `for` loop. You display the comment's ID, its content, and a link to the post it belongs to. You access the post data via `comment.post`.

Use your browser to navigate to the comments page:

```
http://127.0.0.1:5000/comments/
```

You’ll see a page similar to the following:

![Comments Page](https://assets.digitalocean.com/articles/68194/JUIOGKO.png)

Now edit the `base.html` template to make the **Comments** navbar link point to this Comments page:

```custom_prefix((env)sammy@localhost:$)
nano templates/base.html
```

Edit the navigation bar to look as follows:

```html
[label flask_app/templates/base.html]
    <nav>
        <a href="{{ url_for('index') }}">FlaskApp</a>
        <a href="{{ url_for('comments') }}">Comments</a>
        <a href="#">About</a>
    </nav>
```
Save and close the file.

Refresh your comments page, and you'll see that the **Comments** navbar link works.

You now have a page that displays all the comments in the database. Next, you'll add a button below each comment on the post page to allow users to delete it.

## Step 7 — Deleting Comments

In this step, you'll add a **Delete Comment** button below each comment to allow users to delete unwanted comments.

First, you'll add a new `/comments/ID/delete` route that accepts POST requests. The view function will receive the ID of the comment you want to delete, fetch it from the database, delete it, and the redirect to post page the deleted comment was on.

Open `app.py`:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route to the end of the file.

```python
[label flask_app/app.py]
# ...

@app.post('/comments/<int:comment_id>/delete')
def delete_comment(comment_id):
    comment = Comment.query.get_or_404(comment_id)
    post_id = comment.post.id
    db.session.delete(comment)
    db.session.commit()
    return redirect(url_for('post', post_id=post_id))
```
Save and close the file.

Here, instead of using the usual `app.route` decorator, you use the [`app.post`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Flask.post) decorator introduced in [Flask version 2.0.0](https://flask.palletsprojects.com/en/2.0.x/changes/#version-2-0-0), which added shortcuts for common HTTP methods. For example, `@app.post("/login")` is a shortcut for `@app.route("/login", methods=["POST"])`. This means that this view function only accepts POST requests, and navigating to the `/comments/ID/delete` route on your browser will return a `405 Method Not Allowed` error, because web browsers default to GET requests. To delete a comment, the user clicks on a button that sends a POST request to this route.

This `delete_comment()` view function receives the ID of the comment to be deleted via the `comment_id` URL variable. You use the `get_or_404()` method to get a comment and save it in a `comment` variable, or respond with a `404 Not Found` in case the comment doesn't exist. You save the post ID of the post the comment belongs to in a `post_id` variable, which you'll use to redirect to the post after deleting the comment.

You use the `delete()` method on the database session in the line `db.session.delete(comment)`, passing it the comment object. This sets up the session to delete the comment whenever the transaction is committed. Because you don't need to perform any other modifications, you directly commit the transaction using `db.session.commit()`. Lastly, you redirect the user to the post the now-deleted comment was posted on.

Next, edit the `post.html` template to add a **Delete Comment** button below each comment:

```custom_prefix((env)sammy@localhost:$)
nano templates/post.html
```

Edit the `for` loop by adding a new `<form>` tag directly below the comment content:

```html
[label flask_app/templates/post.html]
    {% for comment in post.comments %}
        <div class="comment">
            <p>#{{ comment.id }}</p>
            <p>{{ comment.content }}</p>
            <form method="POST"
                action="{{ url_for('delete_comment',
                                    comment_id=comment.id) }}">
                <input type="submit" value="Delete Comment"
                    onclick="return confirm('Are you sure you want to delete this entry?')">
            </form>
        </div>
    {% endfor %}
```

Save and close the file.

Here, you have a web form that submits a POST request to the `delete_comment()` view function. You pass `comment.id` as an argument for the `comment_id` parameter to specify the comment that will be deleted. You use the [`confirm()` method](https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm) function available in web browsers to display a confirmation message before submitting the request.

Now navigate to a post page on your browser:

```
http://127.0.0.1:5000/2/
```

You'll see a **Delete Comment** button below each comment. Click on it, and confirm the deletion. You'll see that the comment has been deleted.

You now have a way of deleting comments from the database.

## Conclusion

You built a small blogging system that demonstrates how to manage one-to-many relationships using the Flask-SQLAlchemy extension. You learned how to connect a parent table with a child table, associate a child object with its parent and add it to the database, and how to access child data from a parent entry and vise versa.

If you would like to read more about Flask, check out the other tutorials in the [How To Build Web Applications with Flask](https://www.digitalocean.com/community/tutorial_series/how-to-create-web-sites-with-flask) series.


