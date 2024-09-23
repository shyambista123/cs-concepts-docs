---
layout: default
title: Django Quickstart
---

### Django Quickstart

#### 1. Install Python

First, ensure you have Python installed. You can check your version with:

```bash
python --version
```

If you don’t have Python, download and install it from [python.org](https://www.python.org/).

#### 2. Create a Virtual Environment

Navigate to your desired project directory and create a virtual environment:

```bash
mkdir myprojectdir
cd myprojectdir
python -m venv myenv
```

Activate the virtual environment:

- On Windows:

  ```bash
  myenv\Scripts\activate
  ```

- On macOS/Linux:

  ```bash
  source myenv/bin/activate
  ```

#### 3. Install Django

With your virtual environment activated, install Django using pip:

```bash
pip install django
```

#### 4. Create a New Project

To create a new Django project, use the `django-admin` command:

```bash
django-admin startproject myproject
```

This creates a new directory named `myproject` with the following structure:

```
myproject/
    manage.py
    myproject/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

#### 5. Run the Development Server

Navigate to your project directory:

```bash
cd myproject
```

Run the development server:

```bash
python manage.py runserver
```

You should see output indicating the server is running, usually at `http://127.0.0.1:8000/`. Open this URL in your web browser to see the Django welcome page.

#### 6. Create a New App

Django projects are made up of apps. To create an app, run:

```bash
python manage.py startapp myapp
```

This creates a new directory called `myapp` with the following structure:

```
myapp/
    migrations/
    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    views.py
```

#### 7. Define a Model

In `myapp/models.py`, define a simple model. For example:

```python
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()

    def __str__(self):
        return self.name
```

#### 8. Register the Model with Admin

To manage this model in the Django admin interface, register it in `myapp/admin.py`:

```python
from django.contrib import admin
from .models import Item

admin.site.register(Item)
```

#### 9. Configure the App

Add your app to the project's settings. In `myproject/settings.py`, find the `INSTALLED_APPS` list and add your app:

```python
INSTALLED_APPS = [
    # other apps
    'myapp',
]
```

#### 10. Create the Database

Run the following commands to create the database and apply migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### 11. Create an Admin User

Create a superuser to access the admin panel:

```bash
python manage.py createsuperuser
```

Follow the prompts to set up your admin user.

#### 12. Access the Admin Interface

Start the server again if it's not running:

```bash
python manage.py runserver
```

Visit `http://127.0.0.1:8000/admin/` and log in with the superuser credentials. You should see your `Item` model listed.

#### 13. Create a View

In `myapp/views.py`, create a simple view:

```python
from django.shortcuts import render
from .models import Item

def item_list(request):
    items = Item.objects.all()
    return render(request, 'item_list.html', {'items': items})
```

#### 14. Set Up URLs

In `myapp/urls.py`, set up a URL for your view:

```python
from django.urls import path
from .views import item_list

urlpatterns = [
    path('', item_list, name='item_list'),
]
```

Then, include your app’s URLs in the project’s URLs. In `myproject/urls.py`, add:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

#### 15. Create a Template

Create a directory named `templates` in your `myapp` folder and create a file named `item_list.html` inside it:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Item List</title>
</head>
<body>
    <h1>Item List</h1>
    <ul>
        {% for item in items %}
            <li>{{ item.name }}: {{ item.description }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

#### 16. Test Your App

Now, navigate to `http://127.0.0.1:8000/` in your browser to see the list of items. If you add items through the admin interface, they should appear here.

### Conclusion

You've created a simple Django application with a virtual environment, model, admin interface, view, and template. From here, you can explore more features like forms, authentication, and deploying your application.

For more detailed documentation, visit the official Django documentation at [djangoproject.com](https://www.djangoproject.com/). Happy coding!