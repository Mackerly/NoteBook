# How To Use Web Forms in a Flask Application
_The author selected the [Free and Open Source Fund](https://www.brightfunds.org/funds/foss-nonprofits) to receive a donation as part of the [Write for DOnations](https://do.co/w4do-cta) program._

## Introduction

Web forms, such as text fields and text areas, give users the ability to send data to your application to use it to perform an action, or to send larger areas of text to the application. For example, in a social media application, you might give users a box where they can add new content to their pages. Another example is a login page, where you would give the user a text field to enter their username and a password field to enter their password. The server (your Flask application in this case) uses the data the user submits and either signs them in if the data is valid, or responds with a message like `Invalid credentials!` to inform the user that the data they submitted is not correct.

[Flask](http://flask.pocoo.org/) is a lightweight Python web framework that provides useful tools and features for creating web applications in the Python Language. In this tutorial, you'll build a small web application that demonstrates how to use web forms. The application will have a page for displaying messages that are stored in a Python list, and a page for adding new messages. You'll also use [_message flashing_](https://flask.palletsprojects.com/en/2.0.x/patterns/flashing/) to inform users of an error when they submit invalid data.

## Prerequisites

* A local Python 3 programming environment, follow the tutorial for your distribution in [How To Install and Set Up a Local Programming Environment for Python 3](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-python-3) series. In this tutorial, we’ll call the project directory `flask_app`.

* An understanding of basic Flask concepts, such as routes, view functions, and templates. If you are not familiar with Flask, check out [How to Create Your First Web Application Using Flask and Python](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-web-application-using-flask-and-python-3) and [How to Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

* An understanding of basic HTML concepts. You can review our [How To Build a Website with HTML](https://www.digitalocean.com/community/tutorial_series/how-to-build-a-website-with-html) tutorial series for background knowledge.

## Step 1 — Displaying Messages
In this step, you'll create a Flask application with an index page for displaying messages that are stored in a list of Python dictionaries.

First open a new file called `app.py` for editing:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following code inside the `app.py` file to create a Flask server with a single route:

```python
[label flask_app/app.py]
from flask import Flask, render_template

app = Flask(__name__)

messages = [{'title': 'Message One',
             'content': 'Message One Content'},
            {'title': 'Message Two',
             'content': 'Message Two Content'}
            ]

@app.route('/')
def index():
    return render_template('index.html', messages=messages)
```

Save and close the file.

In this file, you first import the `Flask` class and the `render_template()` function from the `flask` package. You then use the `Flask` class to create a new _application instance_ called `app`, passing the special `__name__` variable, which is needed for Flask to set up some paths behind the scenes.  Rendering templates is covered in the tutorial [How To Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

You then create a global Python list called `messages`, which has Python dictionaries inside it. Each dictionary has two keys: `title` for the title of the message, and `content` for the message content. This is a simplified example of a data storage method; in a real-world scenario, you'd use a database that permanently saves the data and allows you to manipulate it more efficiently.

After creating the Python list, you use the `@app.route()` decorator to create a _view function_ called `index()`. In it, you return a call to the `render_template()` function, which indicates to Flask that the route should display an HTML template. You name this template `index.html` (you'll create it later), and you pass a variable called `messages` to it. This variable holds the `messages` list you previously declared as a value and makes it available to the HTML template. View functions are covered in the tutorial [How To Create Your First Web Application Using Flask and Python 3](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-web-application-using-flask-and-python-3).

Next, create a `templates` folder in your `flask_app` directory where Flask searches for templates, then open a template file called `base.html`, which will have code that other templates will inherit to avoid code repetition:

```custom_prefix((env)sammy@localhost:$)
mkdir templates
nano templates/base.html
```

Add the following code inside the `base.html` file to create the base template with a navbar and a content block:

```html
[label flask_app/templates/base.html]

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %} - FlaskApp</title>
    <style>
        .message {
            padding: 10px;
            margin: 5px;
            background-color: #f3f3f3
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

This base template has all the HTML boilerplate you’ll need to reuse in your other templates. The `title` block will be replaced to set a title for each page, and the `content` block will be replaced with the content of each page. The navigation bar has two links, one for the index page where you use the `url_for()` helper function to link to the `index()` view function, and the other for an About page if you choose to include one in your application.

Next, open a template called `index.html`. This is the template you referenced in the `app.py` file:

```custom_prefix((env)sammy@localhost:$)
nano templates/index.html
```

Add the following code to it:

```html
[label flask_app/templates/index.html]
{% extends 'base.html' %}

{% block content %}
    <h1>{% block title %} Messages {% endblock %}</h1>
    {% for message in messages %}
        <div class='message'>
            <h3>{{ message['title'] }}</h3>
            <p>{{ message['content'] }}</p>
        </div>
    {% endfor %}
{% endblock %}
```
Save and close the file.

In this code, you extend the `base.html` template and replace the contents of the `content` block. You use an `<h1>` heading that also serves as a title.

You use a [Jinja `for` loop](https://jinja.palletsprojects.com/en/3.0.x/templates/#for) in the line `{% for message in messages %}` to go through each message in the `messages` list. You use a `<div>` tag to contain the message's title and content. You display the title in an `<h3>` heading and the content in a `<p>` tag.

While in your `flask_app` directory with your virtual environment activated, tell Flask about the application (`app.py` in this case) using the `FLASK_APP` environment variable:

```custom_prefix((env)sammy@localhost:$)
export FLASK_APP=app
```

Then set the `FLASK_ENV` environment variable to `development` to run the application in development mode and get access to the debugger. For more information about the Flask debugger, see [How To Handle Errors in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-handle-errors-in-a-flask-application). Use the following commands to do this (on Windows, use  `set` instead of `export`):

```custom_prefix((env)sammy@localhost:$)
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

You'll see the messages in the `messages` list displayed on the index page:

![Index Page](https://assets.digitalocean.com/68050/hFm7YKr.png)

Now that you have set up your web application and displayed the messages, you'll need a way to allow users to add new messages to the index page. This is done through web forms, which you'll set up in the next step.

## Step 2 — Setting Up Forms
In this step, you will create a page in your application that allows users to add new messages into the list of messages via a web form.

Leave the development server running and open a new terminal window.

First, open your `app.py` file:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route to the end of the file:
```python
[label flask_app/app.py]
# ...

@app.route('/create/', methods=('GET', 'POST'))
def create():
    return render_template('create.html')
```

Save and close the file.

This `/create` route has the `methods` parameter with the tuple `('GET', 'POST')` to accept both `GET` and `POST` requests. `GET` and `POST` are [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). By default, only `GET` requests are accepted,  which are used to retrieve data, such as asking the server for an index page or an About page. `POST` requests are used to submit data to a specific route, which often changes the data on the server. 

In this example, you will ask for the `create` page using a `GET` request. The Create page will have a web form with input fields and a Submit button. When a user fills in the web form and clicks the Submit button, a `POST` request gets sent to the `/create` route. There you handle the request, validate the submitted data to ensure the user has not submitted an empty form, and add it to the `messages` list.

The `create()` view function currently does only one thing: render a template called `create.html` when it receives a regular GET request. You will now create this template, then edit the function to handle `POST` requests in the next step.

Open a new template file called `create.html`:

```custom_prefix((env)sammy@localhost:$)
nano templates/create.html
```

Add the following code to it:

```html
[label flask_app/templates/create.html]
{% extends 'base.html' %}

{% block content %}
    <h1>{% block title %} Add a New Message {% endblock %}</h1>
    <form method="post">
        <label for="title">Title</label>
        <br>
        <input type="text" name="title"
               placeholder="Message title"
               value="{{ request.form['title'] }}"></input>
        <br>

        <label for="content">Message Content</label>
        <br>
        <textarea name="content"
                  placeholder="Message content"
                  rows="15"
                  cols="60"
                  >{{ request.form['content'] }}</textarea>
        <br>
        <button type="submit">Submit</button>
    </form>
{% endblock %}

```
Save and close the file.

In this code, you extend the `base.html` template and replace the `content` block with an `<h1>` heading that serves as a title for the page. In the `<form>` tag, you set the `method` attribute to `post` so the form data gets sent to the server as a `POST` request. 

In the form, you have a text input field named `title`; this is the name you'll use on the application to access the title form data. You give the `<input>` tag a `value` of `{{ request.form['title'] }}`. This is useful to restore the data the user enters so it does not get lost when things go wrong. For example, if the user forgets to fill in the required `content` text area, a request gets sent to the server and an error message will come back as a response, but the data in the title will not be lost because it will be saved on the `request` global object, and can be accessed via `request.form['title']`.

After the title input field, you add a text area named `content` with the value `{{ request.form['content'] }}` for the same reasons mentioned previously.

Last, you have a Submit button at the end of the form.

Now, with the development server running, use your browser to navigate to the `/create` route:

```
http://127.0.0.1:5000/create
```

You will see an "Add a New Message" page with an input field for a message title, a text area for the message's content, and a Submit button.

![Add a new message](https://assets.digitalocean.com/68050/YxB8Khp.png)

This form submits a `POST` request to your `create()` view function. However, there is no code to handle a `POST` request in the function yet, so nothing happens after filling in the form and submitting it. In the next step, you'll handle the incoming `POST` request when a form is submitted. You'll check whether the submitted data is valid (not empty), and add the message title and content to the `messages` list.

## Step 3 — Handling Form Requests
In this step, you will handle form requests on the application side. You'll access the form data the user submits via the form you created in the previous step and add it to the list of messages. You'll also use [_message flashing_](https://flask.palletsprojects.com/en/2.0.x/patterns/flashing/) to inform users when they submit invalid data. The _flash message_ will only be shown once and will disappear on the next request (if you navigate to another page for example).

Open the `app.py` file for editing:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

First, you’ll import the following from the Flask framework:

* The global [`request`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.request) object to access incoming request data that will be submitted via the HTML form you built in the last step.
* The [`url_for()`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.url_for) function to generate URLs.
* The [`flash()`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.flash) function to flash a message when a request is processed (to inform the user that everything went well, or to inform them of an issue if the submitted data is not valid).
* The [`redirect()`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.redirect) function to redirect the client to a different location.

Add these imports to the first line in the file:

```python
[label flask_app/app.py]
from flask import Flask, render_template, request, url_for, flash, redirect

# ...
```

The `flash()` function stores flashed messages in the client’s browser session, which requires setting a _secret key_. This secret key is used to secure sessions, which allow Flask to remember information from one request to another, such as moving from the new message page to the index page. The user can access the information stored in the session, but cannot modify it unless they have the secret key, so you must never allow anyone to access your secret key. See [the Flask documentation for sessions](https://flask.palletsprojects.com/en/2.0.x/api/#sessions) for more information.

The secret key should be a long random string. You can generate a secret key using the `os` module with the [`os.urandom()`](https://docs.python.org/3.8/library/os.html#os.urandom) method, which returns a string of random bytes suitable for cryptographic use. To get a random string using it, open a new terminal and open the Python interactive shell using the following command:

```custom_prefix((env)sammy@localhost:$)
python
```

In the Python interactive shell, import the `os` module from the standard library and call the `os.urandom()` method as follows:

```custom_prefix(>>>)
import os
os.urandom(24).hex()
```

You'll get a string similar to the following:

```
[secondary_label Output]

'df0331cefc6c2b9a5d0208a726a5d1c0fd37324feba25506'
```

You can use the string you get as your secret key.

To set the secret key, add a `SECRET_KEY` configuration to your application via the `app.config` object. Add it directly following the `app` definition before defining the `messages` variable:

```python
[label flask_app/app.py]

# ...
app = Flask(__name__)
app.config['SECRET_KEY'] = 'your secret key'


messages = [{'title': 'Message One',
             'content': 'Message One Content'},
            {'title': 'Message Two',
             'content': 'Message Two Content'}
            ]
# ...
```

Next, modify the `create()` view function to look exactly as follows:

```python
[label flask_app/app.py]
# ...

@app.route('/create/', methods=('GET', 'POST'))
def create():
    if request.method == 'POST':
        title = request.form['title']
        content = request.form['content']

        if not title:
            flash('Title is required!')
        elif not content:
            flash('Content is required!')
        else:
            messages.append({'title': title, 'content': content})
            return redirect(url_for('index'))

    return render_template('create.html')
```

In the `if` statement you ensure that the code following it is only executed when the request is a `POST` request via the comparison `request.method == 'POST'`.

You then extract the submitted title and content from the `request.form` object that gives you access to the form data in the request. If the title is not provided, the condition `if not title` would be fulfilled. In that case, you display a message to the user informing them that the title is required using the `flash()` function. This adds the message to a _flashed messages_ list. You will later display these messages on the page as part of the `base.html` template.  Similarly, if the content is not provided, the condition `elif not content` will be fulfilled. If so, you add the `'Content is required!'`  message to the list of flashed messages. 

If the title and the content of the message are properly submitted, you use the line `messages.append({'title': title, 'content': content})` to add a new dictionary to the `messages` list, with the title and content the user provided. Then you use the `redirect()` function to redirect users to the index page. You use the `url_for()` function to link to the index page.

Save and close the file.

Now, navigate to the `/create` route using your web browser:

```
http://127.0.0.1:5000/create
```

Fill in the form with a title of your choice and some content. Once you submit the form, you will see the new message listed on the index page.

Lastly, you’ll display flashed messages and add a link for the "New Message" page to the navigation bar in the `base.html` template to have easy access to this new page. Open the base template file:

```custom_prefix((env)sammy@localhost:$)
nano templates/base.html
```

Edit the file by adding a new `<a>` tag after the FlaskApp link in the navigation bar inside the `<nav>` tag. Then add a new `for` loop directly above the `content` block to display the flashed messages below the navigation bar. These messages are available in the special `get_flashed_messages()` function Flask provides. Then add a class attribute called `alert` to each message and give it some CSS properties inside the `<style>` tag:

```html
[label flask_app/templates/base.html]
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %} - FlaskApp</title>
    <style>
        .message {
            padding: 10px;
            margin: 5px;
            background-color: #f3f3f3
        }
        nav a {
            color: #d64161;
            font-size: 3em;
            margin-left: 50px;
            text-decoration: none;
        }

        .alert {
            padding: 20px;
            margin: 5px;
            color: #970020;
            background-color: #ffd5de;
        }

    </style>
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">FlaskApp</a>
        <a href="{{ url_for('create') }}">Create</a>
        <a href="#">About</a>
    </nav>
    <hr>
    <div class="content">
        {% for message in get_flashed_messages() %}
            <div class="alert">{{ message }}</div>
        {% endfor %}
        {% block content %} {% endblock %}
    </div>
</body>
</html>
```

Save and close the file, and then reload `https://127.0.0.1:5000` in your browser. The navigation bar will now have a "Create" item that links to the `/create` route.

To see how flash messages work, go to the "Create" page, and click the Submit button without filling the two fields. You'll receive a message that looks like this:

![No title no content flash message](https://assets.digitalocean.com/68050/XoTXYvD.png)

Go back to the index page and you'll see that the flashed messages below the navigation bar disappear, even though they are displayed as part of the base template. If they weren't flashed messages, they would be displayed on the index page too, because it also inherits from the base template.

Try submitting the form with a title but no content. You'll see the message "Content is required!". Click the FlaskApp link in the navigation bar to go back to the index page, then click the Back button to come back to the Create page. You'll see that the message content is still there. This only works if you click the Back button, because it saves the previous request. Clicking the Create link in the navigation bar will send a new request, which clears the form, and as a result, the flashed message will disappear.

You now know how to receive user input, how to validate it, and how to add it to a data source.

[note]
**Note:**
The messages you add to the `messages` list will disappear whenever the server is stopped, because Python lists are only saved in memory, to save your messages permanently, you will need to use a database like SQLite. Check out [How To Use the sqlite3 Module in Python 3](https://www.digitalocean.com/community/tutorials/how-to-use-the-sqlite3-module-in-python-3) to learn how to use SQLite with Python.


## Conclusion

You created a Flask application where users can add messages to a list of messages displayed on the index page. You created a web form, handled the data the user submits via the form, and added it to your messages list. You also used flash messages to inform the user when they submit invalid data.

If you would like to read more about Flask, check out [the other tutorials in the Flask series](https://www.digitalocean.com/community/tutorial_series/how-to-create-web-sites-with-flask).

