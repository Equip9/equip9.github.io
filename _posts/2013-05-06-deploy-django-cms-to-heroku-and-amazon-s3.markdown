---
layout: post
title: "Deploy django-CMS to Heroku and Amazon S3"
date: 2013-05-06 23:13
comments: true
categories: 
---

Although I'm a Ruby on Rails lover today I'm going to write about how to deploy a fresh installation of [django-CMS](https://www.django-cms.org/en/) to [Heroku](https://www.heroku.com/) using [Amazon S3](http://aws.amazon.com/s3/) for storing static files.

This is a compilation from these sites:

* [Getting Started with Django on Heroku](https://devcenter.heroku.com/articles/django)
* [django cms Documentation. Introductory Tutorial](http://docs.django-cms.org/en/2.4.0/getting_started/tutorial.html#getting-help)

First of all we'll create our application directory ```my_app```

``` bash
$ mkdir my_app && cd my_app
```

Next, we'll create our Python's virtual environment using ```virtualenv``` and activate it

``` bash
$ virtualenv --no-site-packages venv 
$ source venv/bin/activate
```

Now we can install required Python packages like django itself, django-cms, gunicorn (web server) and some others with ```pip```

``` bash
$ pip install Django psycopg2 gunicorn dj-database-url django-cms PIL django-storage
```

We'll need the ```requirements.txt``` file to deploy to Heroku

``` bash
$ pip freeze > requirements.txt
```

And create our django project inside the current directory:

``` bash
$ django-admin.py startproject my_app .
```

Add these lines to the ```my_app/settings.py``` file

``` python
# -*- coding: utf-8 -*-
import os
gettext = lambda s: s
PROJECT_PATH = os.path.abspath(os.path.dirname(__file__))
```

and add these in your ```INSTALLED_APPS``` array inside ```my_app/settings.py```


``` python
INSTALLED_APPS = (
	# ...
	'cms',
	'mptt',
	'menus',
	'south',
	'sekizai',
	'cms.plugins.file',
	'cms.plugins.flash',
	'cms.plugins.googlemap',
	'cms.plugins.link',
	'cms.plugins.picture',
	'cms.plugins.snippet',
	'cms.plugins.teaser',
	'cms.plugins.text',
	'cms.plugins.video',
	'cms.plugins.twitter',
	'storages',
)
```

And add these to your ```MIDDLEWARE_CLASSES``` and ```TEMPLATE_CONTEXT_PROCESSORS``` arrays inside ```my_app/settings.py```

``` python
MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.middleware.doc.XViewMiddleware',
    'django.middleware.common.CommonMiddleware',
    'cms.middleware.page.CurrentPageMiddleware',
    'cms.middleware.user.CurrentUserMiddleware',
    'cms.middleware.toolbar.ToolbarMiddleware',
    'cms.middleware.language.LanguageCookieMiddleware',
)
```

``` python
TEMPLATE_CONTEXT_PROCESSORS = (
    'django.contrib.auth.context_processors.auth',
    'django.core.context_processors.i18n',
    'django.core.context_processors.request',
    'django.core.context_processors.media',
    'django.core.context_processors.static',
    'cms.context_processors.media',
    'sekizai.context_processors.sekizai',
)
```

Now we'll configure our templates. First, create the directory where we'll put our django templates

``` bash
$ mkdir templates
```

and create two empty files:

``` bash
$ touch templates/template_1.html
$ touch templates/template_2.html
```

in order two access these templates we have to put them in our ```settings.py``` file and set up available languages

``` python
TEMPLATE_DIRS = (
    # The docs say it should be absolute path: PROJECT_PATH is precisely one.
    # Life is wonderful!
    os.path.join(PROJECT_PATH, "templates"),
)

CMS_TEMPLATES = (
    ('template_1.html', 'Template One'),
    ('template_2.html', 'Template Two'),
)
```


``` python 
LANGUAGES = [
    ('en', 'English'),
]
```

Now let's configure our ```my_app/urls.py```. First, uncomment this line

``` python
admin.autodiscover()
```

And add these lines

``` python
urlpatterns = i18n_patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^', include('cms.urls')),
)

if settings.DEBUG:
    urlpatterns = patterns('',
    url(r'^media/(?P<path>.*)$', 'django.views.static.serve',
        {'document_root': settings.MEDIA_ROOT, 'show_indexes': True}),
    url(r'', include('django.contrib.staticfiles.urls')),
) + urlpatterns
```

Now let's create the base layout inside templates directory ```templates/base.html```

```html
{% raw  %}
{% load cms_tags sekizai_tags %}
<html>
  <head>
      {% render_block "css" %}
  </head>
  <body>
      {% cms_toolbar %}
      {% placeholder base_content %}
      {% block base_content %}{\% endblock %}
      {% render_block "js" %}
  </body>
</html>
{% endraw %}
```

And we can edit the ```templates/template_1.html``` template

``` html
{% raw  %}
{% extends "base.html" %}
{% load cms_tags %}

{% block base_content %}
  {% placeholder template_1_content %}
{% endblock %}
{% endraw %}
```

If you just did a fresh install of django projects run these commands to set up your database

``` bash
$ python manage.py syncdb --all
$ python manage.py migrate --fake
```

If you want to fire up a webserver to test your local set up you can run

``` bash
$ python manage.py runserver
```

Now you can open a browser window and go to [http://localhost:8000/](http://localhost:8000/)

In order to deploy to Heroku we'll have to setup the database in the ```my_app/settings.py```

``` python
# Parse database configuration from $DATABASE_URL
import dj_database_url
DATABASES['default'] =  dj_database_url.config()

# Honor the 'X-Forwarded-Proto' header for request.is_secure()
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

Add a ```Procfile``` file in your project root

``` bash
web: gunicorn my_app.wsgi: 
```

Finally add your Amazon credentials into ```settings.py```

``` python
# Amazon S3 credentials
AWS_ACCESS_KEY_ID       = os.environ['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY   = os.environ['AWS_SECRET_ACCESS_KEY']

# Amazon S3 URL
AWS_STORAGE_BUCKET_NAME = '<your-bucket-name>'
S3_URL = 'http://%s.s3.amazonaws.com/' % AWS_STORAGE_BUCKET_NAME

# Static files location
STATICFILES_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
STATIC_URL = S3_URL

# Default File storage
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
MEDIA_URL = S3_URL
```

If you need to use *Amazon S3* using *http* instead of *https* change this constant to ```False```

``` python
AWS_S3_SECURE_URLS = False
```

You may alse collect your static files and upload them to *Amazon S3*

``` bash
$ python manage.py collectstatic
```

Now you can push to Heroku and set up the database

``` bash
$ git push heroku master
$ heroku run python manage.py syncdb --all
$ heroku run python manage.py migrate --fake
```

After adding a new plugin or changing your database you'll have to execute

``` bash
$ heroku run python manage.py syncdb
$ heroku run python manage.py migrate
```