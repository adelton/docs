---
title: Getting Started
layout: framework_docs
order: 1
redirect_from: /docs/getting-started/django/
subnav_glob: docs/django/getting-started/*.html.*
objective: Build and deploy a very basic Django app to Fly.io. This guide is the fastest way to try using Fly.io, so if you're short on time start here.
related_pages:
  - /docs/django/getting-started/existing
  - /docs/flyctl/
  - /docs/reference/configuration/
  - /docs/postgres/
---

<div>
  <img src="/static/images/django-intro.webp" srcset="/static/images/django-intro@2x.webp 2x" alt="">
</div>

In this guide we recreate and deploy [this Django application](https://github.com/fly-apps/hello-django) to demonstrate how quickly Django apps can be deployed to Fly.io!

## Initial Local Setup

Make sure that [Python](https://www.python.org/) is already installed on your computer along with a way to create virtual environments.

This allows you to run your project locally, and test that it works, before deploying it to Fly.io.

> We recommend the latest [supported versions](https://devguide.python.org/versions/#supported-versions) of Python.

Create a folder for your project. Here we'll call it `hello-django`. Enter the folder with `cd`:

```cmd
mkdir hello-django && cd hello-django
```

### Virtual Environment

For this guide, we use [venv](https://docs.python.org/3/library/venv.html#module-venv) but any of the other popular choices such as [Poetry](https://python-poetry.org/), [Pipenv](https://github.com/pypa/pipenv), or [pyenv](https://github.com/pyenv/pyenv) work too.

```shell
# Unix/macOS
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $
```
```shell
# Windows
$ python -m venv .venv
$ .venv\Scripts\activate
(.venv) $
```

<section class="callout">From this point on, the commands won't be displayed with `(.venv) $` but we assume you have your Python virtual environment activated.</section>

### Install Django

With your virtual environment **activated**, install the [latest](https://www.djangoproject.com/download/#supported-versions) version of Django using [pip](https://pip.pypa.io/en/stable/):

```cmd
python -m pip install Django
```

### Create a Django Project

Inside the `hello-django` folder, create a Django project named `hello_django`:

```cmd
django-admin startproject hello_django .
```

> Don't forget the `.` at the end. It's crucial because it tells the script to create the Django project directory structure in the current directory, our folder `hello-django`.

Note that by convention, we name Django projects using `snake_case`: words written in lowercase with spaces replaced by underscore (`_`). Hyphens (`-`) are not valid identifiers and you might get this error message:
```terminal
'hello-django' is not a valid project name. Please make sure the name is a valid identifier.
```

### Create a Django App

Now create a new app called `hello`:

```cmd
python manage.py startapp hello
```

Add the new `hello` app to the `INSTALLED_APPS` configuration in the [`hello_django/settings.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/settings.py) file:

```python
# hello_django/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'hello',  # <-- Updated!
]
```

By this point, our project structure should look like this:

```terminal
hello-django/
|-- .venv
|-- hello/
|   |-- migrations/
|   |   |-- __init__.py
|   |-- __init__.py
|   |-- admin.py
|   |-- apps.py
|   |-- models.py
|   |-- tests.py
|   |-- views.py
|-- hello_django/
|   |-- __init__.py
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   |-- wsgi.py
|-- manage.py
```

### Create and Map a URL to a View

Now let's configure a basic view that returns the text, `Hello, Fly!` by updating the [`hello/views.py`](https://github.com/fly-apps/hello-django/blob/main/hello/views.py) file:

```python
# hello/views.py
from django.http import HttpResponse


def hello(request):
    return HttpResponse('Hello, Fly!')
```

Inside the `hello/` app folder, create a new file called [`urls.py`](https://github.com/fly-apps/hello-django/blob/main/hello/urls.py) for our app-level URL configuration:

```python
# hello/urls.py
from django.urls import path

from .views import hello

urlpatterns = [
    path('', hello, name='hello'),
]
```

And update the existing [`hello_django/urls.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/urls.py) file for project-level URL configuration:

```python
# hello_django/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('hello.urls'))  # <-- Updated!
]
```

### Run migrations

As part of Django's core functionality, some existing apps are included by default to provide you with out-of-the-box features. Some of those apps require their own database tables.

To initialize the local database and set up those tables, run the `migrate` command:

```cmd
python manage.py migrate
```

### Start the Web Server

Now `runserver` to start up Django's local web server:

```cmd
python manage.py runserver
```

If you open `http://127.0.0.1:8000/` in your web browser it now displays the text `Hello, Fly!`.

## Django Deployment Checklist

By default, Django is configured for local development. The [How to Deploy Django](https://docs.djangoproject.com/en/stable/howto/deployment/) and [Django Deployment Checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/) guide list the various steps required for a secure deployment.

> You can also find a complete guide [Deploying Django to Production](https://fly.io/django-beats/deploying-django-to-production/) in our [Django Beats Blog](https://fly.io/django-beats/).

However, for demonstration purposes, we can take some shortcuts.

First, in the [`hello_django/settings.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/settings.py) file update the `ALLOWED_HOSTS` configuration to accept all hosts:

```python
# hello_django/settings.py
ALLOWED_HOSTS = ["*"]  # <-- Updated!
```

Second, install [Gunicorn](https://gunicorn.org/) as our [production server](https://docs.djangoproject.com/en/dev/ref/django-admin/#runserver):

```cmd
python -m pip install gunicorn
```

Third, create a [`requirements.txt`](https://github.com/fly-apps/hello-django/blob/main/requirements.txt) file listing all the packages in the current Python virtual environment:

```cmd
pip freeze > requirements.txt
```

That's it! We're ready to deploy on Fly.io.

## flyctl

Fly.io has its own command-line utility for managing apps, [flyctl](https://fly.io/docs/hands-on/install-flyctl/). If not already installed, follow the instructions on the [installation guide](https://fly.io/docs/hands-on/install-flyctl/) and [log in to Fly.io](https://fly.io/docs/getting-started/log-in-to-fly/).


## Configure and Deploy your Fly App

To configure and launch the app, run the `fly launch` command and follow the wizard. You can set a name for the app and choose your primary region. You can also choose to launch and attach a Postgresql database and/or a Redis database though we are not using either in this example.

```cmd
fly launch
```
```output
Creating app in ../flyio//hello-django
Scanning source code
Detected a Django app
? Choose an app name (leave blank to generate one): hello-django
automatically selected personal organization: Fly.io
? Choose a region for deployment: Amsterdam, Netherlands (ams)
Created app hello-django in organization personal
Set secrets on hello-django: SECRET_KEY
Wrote config file fly.toml
? Would you like to set up a Postgresql database now? No
? Would you like to set up an Upstash Redis database now? No

[INFO] Python 3.10.11 was detected. 'python:3.10-slim-buster' image will be set in the Dockerfile.

Validating ../flyio/hello-django/fly.toml
Platform: machines
✓ Configuration is valid
Your app is ready! Deploy with `flyctl deploy`
```

This creates two new files in the project that are automatically configured: a [`Dockerfile`](https://github.com/fly-apps/hello-django/blob/main/Dockerfile) and [`fly.toml`](https://github.com/fly-apps/hello-django/blob/main/fly.toml) file to configure applications for deployment.


To deploy the application use the following command:

```cmd
fly deploy
```

This will take a few seconds as it uploads your application, verifies the app configuration, builds the image, and then monitors to ensure it starts successfully. Once complete visit your app with the following command:

```cmd
fly apps open
```

YAY! You are up and running! Wasn't that easy?

## Recap

We started with an empty directory and in a matter of minutes had a running Django application deployed to the web. A few things to note:

  * Your application is running on a Virtual Machine that was created based on the `Dockerfile` image.
  * The `fly.toml` file controls your app configuration and can be modified as needed.
  * `fly dashboard` can be used to monitor and adjust your application. Pretty much anything you can do from the browser window you can also do from the command line using [`fly` commands](/docs/flyctl/). Try `fly help` to see what you can do.

Now that you have seen how to deploy a simple Django application, it is time to move on to [Existing Django Apps](/docs/django/getting-started/existing/) that feature static files and a PostgreSQL database.
