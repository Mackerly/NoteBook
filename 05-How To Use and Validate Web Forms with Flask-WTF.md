# How To Use and Validate Web Forms with Flask-WTF
*The author selected the [Free and Open Source Fund](https://www.brightfunds.org/funds/foss-nonprofits) to receive a donation as part of the [Write for DOnations](https://do.co/w4do-cta) program.*

## Introduction

Web forms, such as text fields and text areas, give users the ability to send data to your application, whether that's a drop-down or a radio button that the application will use to perform an action, or to send large areas of text to be processed or displayed. For example, in a social media application, you might give users a box where they can add new content to their pages.

[Flask](http://flask.pocoo.org/) is a lightweight Python web framework that provides useful tools and features for creating web applications in the Python Language. To render and validate web forms in a safe and flexible way in Flask, you'll use [Flask-WTF](https://flask-wtf.readthedocs.io/en/1.0.x/), which is a Flask extension that helps you use the [WTForms](https://wtforms.readthedocs.io/en/3.0.x/) library in your Flask application.

[WTForms](https://wtforms.readthedocs.io/en/3.0.x/) is a Python library that provides flexible web form rendering. You can use it to render text fields, text areas, password fields, radio buttons, and others. WTForms also provides powerful data validation using different _validators_, which validate that the data the user submits meets certain criteria you define. For example, if you have a required field, you can ensure data the user submits is provided, or has a certain length.

WTForms also uses a CSRF token to provide protection from [CSRF attacks](https://owasp.org/www-community/attacks/csrf), which are attacks that allows the attacker to execute unwanted actions on a web application in which the user is authenticated. A successful CSRF attack can force the user to perform state-changing requests like transferring funds to the attacker's bank account in a banking application, changing the user's email address, and so forth. If the victim is an administrative account, CSRF can compromise the entire web application.

In this tutorial, you’ll build a small web application that demonstrates how to render and validate web forms using Flask-WTF. The application will have a page for displaying courses that are stored in a Python list, and the index page will have a form for entering the course title, its description, price, availability, and level (beginner, intermediate, or advanced).

## Prerequisites

* A local Python 3 programming environment. Follow the tutorial for your operating system in the [How To Install and Set Up a Local Programming Environment for Python 3](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-python-3) series. In this tutorial you’ll call the project directory `flask_app`.

* An understanding of basic Flask concepts, such as routes, view functions, and templates. If you are not familiar with Flask, check out [How to Create Your First Web Application Using Flask and Python](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-web-application-using-flask-and-python-3) and [How to Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

* An understanding of basic HTML concepts. You can review our [How To Build a Website with HTML](https://www.digitalocean.com/community/tutorial_series/how-to-build-a-website-with-html) tutorial series for background knowledge.

* (_optional_) An understanding of basic web form usage in Flask. See [How To Use Web Forms in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-web-forms-in-a-flask-application).

## Step 1 — Installing Flask and Flask-WTF
In this step, you'll install Flask and Flask-WTF, which also installs the WTForms library automatically.

With your virtual environment activated, use `pip` to install Flask and Flask-WTF:

```custom_prefix((env)sammy@localhost:$)
pip install Flask Flask-WTF
```

Once the installation is successfully finished, you'll see a line similar to the following at the end of the output:

```
[secondary_label Output]
Successfully installed Flask-2.0.2 Flask-WTF-1.0.0 Jinja2-3.0.3 MarkupSafe-2.0.1 WTForms-3.0.0 Werkzeug-2.0.2 click-8.0.3 itsdangerous-2.0.1
```

As you can see, the [WTForms](https://wtforms.readthedocs.io/en/3.0.x/) library was also installed as a dependency of the Flask-WTF package. The rest of the packages are Flask dependencies.

Now that you've installed the required Python packages, you'll set up a web form next.

## Step 2 — Setting up Forms
In this step, you'll set up a web form using fields and validators you'll import from the WTForms library.

You'll set up the following fields:

* **Title:** A text input field for the course title.
* **Description:** A text area field for the course description.
* **Price:** An integer field for the price of the course.
* **Level:** A radio field for the course level with three choices: Beginner, Intermediate, and Advanced.
* **Available:** A checkbox field that indicates whether the course is currently available.

First, open a new file called `forms.py` in your `flask_app` directory. This file will have the forms you'll need in your application:

```custom_prefix((env)sammy@localhost:$)
nano forms.py
```

This file will have a class that represents your web form. Add the following imports at the top:

```python
[label flask_app/forms.py]
from flask_wtf import FlaskForm
from wtforms import (StringField, TextAreaField, IntegerField, BooleanField,
                     RadioField)
from wtforms.validators import InputRequired, Length

```
To build a web form, you will create a subclass of the `FlaskForm` base class, which you import from the `flask_wtf` package. You also need to specify the fields you use in your form, which you will import from the `wtforms` package.

You import the following [fields](https://wtforms.readthedocs.io/en/3.0.x/fields/) from the WTForms library:

* [`StringField`](https://wtforms.readthedocs.io/en/3.0.x/fields/#wtforms.fields.StringField): A text input.
* [`TextAreaField`](https://wtforms.readthedocs.io/en/3.0.x/fields/#wtforms.fields.TextAreaField): A text area field.
* [`IntegerField`](https://wtforms.readthedocs.io/en/3.0.x/fields/#wtforms.fields.IntegerField): A field for integers.
* [`BooleanField`](https://wtforms.readthedocs.io/en/3.0.x/fields/#wtforms.fields.BooleanField): A checkbox field.
* [`RadioField`](https://wtforms.readthedocs.io/en/3.0.x/fields/#wtforms.fields.RadioField): A field for displaying a list of radio buttons for the user to choose from.

In the line `from wtforms.validators import InputRequired, Length`, you import validators to use on the fields to make sure the user submits valid data. [`InputRequired`](https://wtforms.readthedocs.io/en/3.0.x/validators/#wtforms.validators.InputRequired) is a validator you'll use to ensure the input is provided, and [`Length`](https://wtforms.readthedocs.io/en/3.0.x/validators/#wtforms.validators.Length) is for validating the length of a string to ensure it has a minimum number of characters, or that it doesn't exceed a certain length.

Next, add the following [class](https://www.digitalocean.com/community/tutorials/how-to-construct-classes-and-define-objects-in-python-3) after the `import` statements:

```python
[label flask_app/forms.py]

class CourseForm(FlaskForm):
    title = StringField('Title', validators=[InputRequired(),
                                             Length(min=10, max=100)])
    description = TextAreaField('Course Description',
                                validators=[InputRequired(),
                                            Length(max=200)])
    price = IntegerField('Price', validators=[InputRequired()])
    level = RadioField('Level',
                       choices=['Beginner', 'Intermediate', 'Advanced'],
                       validators=[InputRequired()])
    available = BooleanField('Available', default='checked')
```
Save and close the file.

In this `CourseForm` class, you inherit from the `FlaskForm` base class you imported earlier. You define a collection of form fields as class variables using the form fields you imported from the WTForms library. When you instantiate a field, the first argument is the field's label.

You define the validators for each field by passing a list of the validators you import from the `wtforms.validators` module. The title field, for example, has the string `'Title'` as a label, and two validators:

* `InputRequired`: To indicate that the field should not be empty.
* `Length`: Takes two arguments; `min` is set to `10` to make sure that the title is at least 10 characters long, and `max` is set to `100` to ensure it doesn't exceed 100 characters.

The description text area field has an `InputRequired` validator and a `Length` validator with the `max` parameter set to `200`, with no value for the `min` parameter, which means the only requirement is that it doesn't exceed 200 characters.

Similarly you define a required integer field for the price of the course called `price`.

The `level` field is a radio field with multiple choices. You define the choices in a Python list and pass it to the `choices` parameter. You also define the field as required using the `InputRequired` validator.

The `available` field is a check box field. You set a default `'checked'` value by passing it to the `default` parameter. This means the check box will be checked when adding new courses unless the user unchecks it, meaning courses are available by default.

For more on how to use the WTForms library, see the [Crash Course page](https://wtforms.readthedocs.io/en/3.0.x/crash_course/) on the WTForms documentation. See the [Fields page](https://wtforms.readthedocs.io/en/3.0.x/fields/) for more fields, and the [Validators page](https://wtforms.readthedocs.io/en/3.0.x/validators/) for more validators to validate form data.

You've configured your web form in a `forms.py` file. Next, you'll create a Flask application, import this form, and display its fields on the index page. You'll also display a list of courses on another page.

## Step 3 — Displaying the Web Form and Courses
In this step, you'll create a Flask application, display the web form you created in the previous step on the index page, and also create a list of courses and a page for displaying the courses on it.

With your programming environment activated and Flask installed, open a file called `app.py` for editing inside your `flask_app` directory:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

This file will import the necessary class and helpers from Flask, and the `CourseForm` from the `forms.py` file. You'll build a list of courses, then instantiate the form and pass it to a template file. Add the following code to `app.py`:

```python
[label flask_app/app.py]
from flask import Flask, render_template, redirect, url_for
from forms import CourseForm

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your secret key'


courses_list = [{
    'title': 'Python 101',
    'description': 'Learn Python basics',
    'price': 34,
    'available': True,
    'level': 'Beginner'
    }]


@app.route('/', methods=('GET', 'POST'))
def index():
    form = CourseForm()
    return render_template('index.html', form=form)
```
Save and close the file.

Here you import the following from Flask:
* The `Flask` class to create a Flask application instance.
* The `render_template()` function to render the index template.
* The `redirect()` function to redirect the user to the courses page once a new course is added.
* The `url_for()` function for building URLs.

First you import the `CourseForm()` class from the `forms.py` file, then you create a Flask application instance called `app`.

You set up a _secret key_ configuration for WTForms to use when generating a CSRF token to secure your web forms. The secret key should be a long random string. See Step 3 of [How To Use Web Forms in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-web-forms-in-a-flask-application#step-3-%E2%80%94-handling-form-requests) for more information on how to obtain a secret key.

Then you create a list of dictionaries called `courses_list`, which currently has one dictionary with a sample course titled `'Python 101'`. Here, you use a Python list as a data store for demonstration purposes. In a real world scenario, you'll use a database such as SQLite. See [How To Use an SQLite Database in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-an-sqlite-database-in-a-flask-application) to learn how to use a database to store your courses' data.

You create a `/` main route using the `app.route()` decorator on the `index()` view function. It accepts both `GET` and `POST` [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) in the `methods` parameter. GET methods are for retrieving data, and POST requests are for sending data to the server, through a web form for example. For more, see [How To Use Web Forms in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-web-forms-in-a-flask-application).

You instantiate the `CourseForm()` class that represents the web form and save the instance in a variable called `form`. You then return a call to the `render_template()` function, passing it a template file called `index.html` and the form instance.

To display the web form on the index page, you will first create a base template, which will have all the basic HTML code other templates will also use to avoid code repetition. Then you’ll create the `index.html` template file you rendered in your `index()` function. To learn more about templates, see [How to Use Templates in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-a-flask-application).

Create a `templates` folder in your `flask_app` directory where Flask searches for templates, then open a template file called `base.html`, which will be the base template for other templates:

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

This base template has all the HTML boilerplate you’ll need to reuse in your other templates. The `title` block will be replaced to set a title for each page, and the `content` block will be replaced with the content of each page. The navigation bar has two links, one for the index page where you use the `url_for()` helper function to link to the `index()` view function, and the other for an About page if you choose to include one in your application.

Save and close the file.

Next, open a template called `index.html`. This is the template you referenced in the `app.py` file:

```custom_prefix((env)sammy@localhost:$)
nano templates/index.html
```

This file will have the web form you passed to the `index.html` template via the `form` variable. Add the following code to it:

```html
[label flask_app/templates/index.html]
{% extends 'base.html' %}

{% block content %}
    <h1>{% block title %} Add a New Course {% endblock %}</h1>

    <form method="POST" action="/">
        {{ form.csrf_token }}
        <p>
            {{ form.title.label }}
            {{ form.title(size=20) }}
        </p>

        {% if form.title.errors %}
            <ul class="errors">
                {% for error in form.title.errors %}
                    <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <p>
            {{ form.description.label }}
        </p>
        {{ form.description(rows=10, cols=50) }}

        {% if form.description.errors %}
            <ul class="errors">
                {% for error in form.description.errors %}
                    <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <p>
            {{ form.price.label }}
            {{ form.price() }}
        </p>

        {% if form.price.errors %}
            <ul class="errors">
                {% for error in form.price.errors %}
                    <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <p>
            {{ form.available() }} {{ form.available.label }}
        </p>

        {% if form.available.errors %}
            <ul class="errors">
                {% for error in form.available.errors %}
                    <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <p>
            {{ form.level.label }}
            {{ form.level() }}
        </p>

        {% if form.level.errors %}
            <ul class="errors">
                {% for error in form.level.errors %}
                    <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <p>
            <input type="submit" value="Add">
        </p>
    </form>

{% endblock %}
```

Save and close the file.

You extend the base template, and set a title in an `<h1>` tag. Then you render the web form fields inside a `<form>` tag, setting its method to `POST` and the action to the `/` main route, which is the index page. You first render the CSRF token WTForms uses to protect your form from CSRF attacks using the line `{{ form.csrf_token }}`. This token gets sent to the server with the rest of the form data. Remember to always render this token to secure your forms.

You render each field using the syntax `form.field()` and you render its label using the syntax `form.field.label`. You can pass arguments to the field to control how it is displayed. For example, you set the size of the title input field in `{{ form.title(size=20) }}`, and you set the numbers of rows and columns for the description text area via the parameters `rows` and `cols` the same way you would do normally in HTML. You can use the same method to pass additional HTML attributes to a field such as the `class` attribute to set a CSS class.

You check for validation errors using the syntax `if form.field.errors`. If a field has errors, you loop through them with a `for` loop and display them in a list below the field.

While in your `flask_app` directory with your virtual environment activated, tell Flask about the application (`app.py` in this case) using the `FLASK_APP` environment variable. Then set the `FLASK_ENV` environment variable to `development` to run the application in development mode and get access to the debugger. For more information about the Flask debugger, see [How To Handle Errors in a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-handle-errors-in-a-flask-application). Use the following commands to do this (on Windows, use `set` instead of `export`):

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

You'll see the web form displayed on the index page:

![Index Page](https://assets.digitalocean.com/68100/28ijxDu.png)

Try to submit the form without filling in the title. You'll see an error message informing you that the title is required. Experiment with the form by submitting invalid data (such as a short title less than 10 characters long, or a description over 200 characters long) to see other error messages.

Filling the form with valid data does nothing so far because you don't have code that handles form submission. You'll add the code for that later.

For now, you need a page to display the courses you have in your list. Later, handling the web form data will add a new course to the list and redirect the user to the courses page to see the new course added to it.

Leave the development server running and open another terminal window.

Next, open `app.py` to add the courses route:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Add the following route at the end of the file:

```python
[label flask_app/app.py]
# ...

@app.route('/courses/')
def courses():
    return render_template('courses.html', courses_list=courses_list)
```

Save and close the file.

This route renders a template called `courses.html`, passing it the `courses_list` list.

Then create the `courses.html` template to display courses:
```custom_prefix((env)sammy@localhost:$)
nano templates/courses.html
```

Add the following code to it:

```html
[label flask_app/templates/courses.html]
{% extends 'base.html' %}

{% block content %}
    <h1>{% block title %} Courses {% endblock %}</h1>
    <hr>
    {% for course in courses_list %}
        <h2> {{ course['title'] }} </h2>
        <h4> {{ course['description'] }} </h4>
        <p> {{ course['price'] }}$ </p>
        <p><i>({{ course['level'] }})</i></p>
        <p>Availability:
            {% if course['available'] %}
                Available
            {% else %}
                Not Available
            {% endif %}</p>
        <hr>
    {% endfor %}
{% endblock %}
```
Save and close the file.

You set a title and loop through the items of the `courses_list` list. You display the title in an `<h2>` tag, the description in an `<h4>` tag, and the price and course level in a `<p>` tag.
You check whether the course is available using the condition `if course['available']`. You display the text "Available" if the course is available, and the text "Not Available" if it's not available.

Use your browser to go to the courses page:

```
http://127.0.0.1:5000/courses/
```

You'll see a page with one course displayed, because you only have one course in your course list so far:

![Courses Page](https://assets.digitalocean.com/68100/mx2PZOk.png)

Next, open `base.html` to add a link to the courses page in the navigation bar:

```custom_prefix((env)sammy@localhost:$)
nano templates/base.html
```

Edit it to look as follows:

```html
[label flask_app/templates/base.html]
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %} - FlaskApp</title>
    <style>
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
        <a href="{{ url_for('courses') }}">Courses</a>
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

Refresh the index page, and you'll see a new **Courses** link in the navigation bar.

You've created the pages you need for your application: An index page with a web form for adding new courses and a page for displaying the courses you have in your list.

To make the application functional, you need to handle the web form data when the user submits it by validating it and adding it to the courses list. You'll do this next.

## Step 4 — Accessing Form Data
In this step, you'll access data the user submits, validate it, and add it to the list of courses.

Open `app.py` to add code for handling the web form data inside the `index()` function:

```custom_prefix((env)sammy@localhost:$)
nano app.py
```

Edit the `index()` function to look as follows:

```python
[label flask_app/app.py]
# ...
@app.route('/', methods=('GET', 'POST'))
def index():
    form = CourseForm()
    if form.validate_on_submit():
        courses_list.append({'title': form.title.data,
                             'description': form.description.data,
                             'price': form.price.data,
                             'available': form.available.data,
                             'level': form.level.data
                             })
        return redirect(url_for('courses'))
    return render_template('index.html', form=form)
```
Save and close the file.
Here, you call the `validate_on_submit()` method on the `form` object, which checks that the request is a POST request, and runs the validators you configured for each field. If at least one validator returns an error, the condition will be `False`, and each error will be displayed below the field that caused it.

If the submitted form data is valid, the condition is `True`, and the code below the `if` statement will be executed. You build a course dictionary, and use the `append` method to add the new course to the `courses_list` list. You access the value of each field using the syntax `form.field.data`. After you add the new course dictionary to the courses list, you redirect the user to the `Courses` page.

With the development server running, visit the index page:

```
http://127.0.0.1:5000/
```

Fill the form with valid data and submit it. You'll be redirected to the `Courses` page, and you'll see the new course displayed on it.

## Conclusion
You made a Flask application that has a web form you built using the Flask-WTF extension and the WTForms library. The form has several types of fields to receive data from the user, validate it using special WTForms validators, and add it to a data store.

If you would like to read more about Flask, check out the other tutorials in the [How To Create Web Sites with Flask](https://www.digitalocean.com/community/tutorial_series/how-to-create-web-sites-with-flask) series.

