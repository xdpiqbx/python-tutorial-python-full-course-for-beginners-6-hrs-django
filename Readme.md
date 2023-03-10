# [Django Project](https://www.djangoproject.com/download/)

## [Lesson](https://youtu.be/_uQrJ0TkZlc?t=17953)

---
django-admin

```text
[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    optimizemigration
    runserver
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
```

---

### Create project PyShop in current directory

`django-admin startproject pyshop .`

- `./pyshop/settings.py` - settings for app
- `./pyshop/urls.py` - routing
- `./pyshop/wsgi.py` - web server gateway interface
- `./manage.py` - Django's command-line utility for administrative tasks

---

### manage.py

`manage.py` same as `django-admin`

`python manage.py runserver` - starts on localhost:8000

`python manage.py startapp products` - it creates package products with basic files structure

---

### Mapping ursl 

`./products/views.py` - create views
```python
from django.http import HttpResponse
from django.shortcuts import render


def index(request):
    return HttpResponse("In Product")
```

create `./products/urls.py` - mapping URLs to views here
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index)
]
```

---

### Main routing, binding product to `pyshop`
add path too root application `pyshop` using `include` function because
`TypeError: view must be a callable or a list/tuple in the case of include().`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('products/', include('products.urls'))
]
```

---

### Create model for database

in file `./products/models.py`

```python
from django.db import models


class Product(models.Model):
    name = models.CharField(max_length=255)
    price = models.FloatField()
    stock = models.IntegerField()
    image_url = models.CharField(max_length=2083)
```

---

### Make migrations

`python manage.py makemigrations`

No changes detected - because root `pyshop` don't know anything about `products`

In `./pyshop/settings.py` in `INSTALLED_APPS` list add `products.apps.ProductsConfig`:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    # .....
    'products.apps.ProductsConfig'
]
```

again `python manage.py makemigrations`

```text
Migrations for 'products':
  products\migrations\0001_initial.py
    - Create model Product
```

---

### Apply migration

to apply migration `python manage.py migrate`

```text
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, products, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
.....
```

and if i repeat `python manage.py migrate`

```text
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, products, sessions
Running migrations:
  No migrations to apply.
```

---

### Admin panel

python manage.py createsuperuser

```text
Username (leave blank to use 'max'): admin
Email address: admin@mail.com
Password: admin (when type you do not see password)
Password (again): admin (when type you do not see password)
The password is too similar to the username.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

`http://127.0.0.1:8000/admin` - admin:admin

#### Administrate products

`./products/admin.py`

register your model Product for Admin section

```python
from django.contrib import admin
from .models import Product


admin.site.register(Product)
```

#### Customize product admin

`./products/admin.py`

```python
from django.contrib import admin
from .models import Product


class ProductAdmin(admin.ModelAdmin):
    list_display = ('name', 'price', 'stock')


admin.site.register(Product, ProductAdmin)
```

---

### View and Templates

create `./products/template`
create `./products/template/index.html`

```html
<h1>Products</h1>
<ul>
    {% for product in products %}
        <li>{{ product.name }} (${{ product.price }})</li>
    {% endfor %}
</ul>
```

- `{% ... %}` - template tag, for dynamic logic (for, if ...)
- `{{ ... }}` - for dynamically render values in html

#### Binding View and Template

in `./products/views.py`

```python
from django.http import HttpResponse
from django.shortcuts import render
from .models import Product


def index(request):
    products = Product.objects.all()
    # return HttpResponse("In Products")
    return render(
        request,
        'index.html',
        {'products': products}
    )


def new(request):
    return HttpResponse("New Product")
```

#### Short about some database methods

- `Product.objects.all()` - return all products from database
- `Product.objects.filter()` - return some products
- `Product.objects.get()` - to get single product
- `Product.objects.save()` - insert or updating existing product

#### Relocate base.html from ./products/templates to ./templates

- create `template` folder in root of project
- in root package `pyshop` `settings.py` add template to `TEMPLATES` list
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'templates')
        ],
        #  ........
    },
]
```

---

### Add Bootstrap

- in `base.html`
```html
<head><!-- add  Bootstrap--></head>
<body>
<!-- ... -->
    <div class="container">
        {% block content %}
        {% endblock %}
    </div>
<!-- ... -->
</body>
```

- in `./products/templates/index.html`
```html
{% extends 'base.html' %}
{% block content %}
    <h1>Products</h1>
    <div class="row">
        {% for product in products %}
          <img src="{{ product.image_url }}"
               alt="{{ product.name }}">
            <h5 class="card-title">{{ product.name }}</h5>
            <p class="card-text">${{ product.price }}</p>
        {% endfor %}
    </div>
{% endblock %}
```