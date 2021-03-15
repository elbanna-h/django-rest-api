# django-rest-api
Django Rest API source code



to clear everything in docker

```
$ docker system prune --all
```



```
$ git clone https://github.com/elbanna-h/django-rest-api.git
```



```
add .idea/ to .getignore file
```



```
$ cd django-rest-api
$ touch Dockerfile
```



https://hub.docker.com/

search for python -> 3.9.2-alpine



in Dockerfile

```dockerfile
FROM python:3.9.2-alpine
MAINTAINER Hany Elbanna

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /requirements.txt
RUN pip install -r /requirements.txt

RUN mkdir /app
WORKDIR /app
COPY ./app /app

RUN adduser -D user
USER user

```



```
$ touch requirements.txt
```



in requirements.txt

```
Django>=3.1.7,<3.2.0
djangorestframework>=3.12.2,<3.13.0
flake8>=3.8.4,<3.9.0
```



```
$ mkdir app

```



```
$ docker build .

```



```
$ touch docker-compose.yml

```



in docker-compose.yml

```yaml
version: "3"

services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
```



command + option + l

to reformat in pycharm



```
$ docker-compose build

```



```
$ docker-compose run app sh -c "django-admin.py startproject app ."

```



```
$ git add .
$ git commit -m  "Setup docker and Django project"

```



travis-ci.org -> sign in with github account -> + -> enable django-rest-api

```
$ touch .travis.yml
```



in .travis.yml

```yaml
language: python
python:
  - "3.9"

services:
  - docker

before_script: pip install docker-compose

script:
  - docker-compose run app sh -c "python manage.py test && flake8"
```



command + option + l

to reformat in pycharm



```
$ cd app
$ touch .flake8
```



in .flake8

```
[flake8]
exclude =
	migrations,
	__pycache__,
	manage.py,
	settings.py
```



```
$ cd ..
$ git add .
$ git commit -m "Added flake8 and Travis-CI configuration."

$ git push origin
```



```
$ docker-compose run app sh -c "python manage.py startapp core"
$ rm app/core/tests.py  # will replace it with testes folder
$ rm app/core/views.py  # no need for views in core app
$ mkdir app/core/tests
$ touch app/core/tests/__init__.py
```



```
add:
'core',
to installed apps in settings.py
```



```
$ touch app/core/tests/test_models.py
```



in test_models.py

```python
from django.contrib.auth import get_user_model
from django.test import TestCase


class ModelTests(TestCase):
    def test_create_user_with_email_successful(self):
        """Test creating a new user with an email is successful"""
        email = 'test@hany.ws'
        password = 'TestPass123'
        user = get_user_model().objects.create_user(
            email=email,
            password=password
        )

        self.assertEqual(user.email, email)
        self.assertTrue(user.check_password(password))

```



```
$ docker-compose run app sh -c "python manage.py test"
```



in models.py

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, \
    PermissionsMixin
from django.db import models


class UserManager(BaseUserManager):

    def create_user(self, email, password=None, **extra_fields):
        """Creates and saves a new user"""
        if not email:
            raise ValueError('Users must have an email address')
        user = self.model(email=self.normalize_email(email), **extra_fields)
        user.set_password(password)
        user.save(using=self._db)

        return user

    def create_superuser(self, email, password):
        """Creates and saves a new super user"""
        user = self.create_user(email, password)
        user.is_staff = True
        user.is_superuser = True
        user.save(using=self._db)

        return user


class User(AbstractBaseUser, PermissionsMixin):
    """Custom user model that supports using email instead of username"""
    email = models.EmailField(max_length=255, unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = UserManager()

    USERNAME_FIELD = 'email'

```



in settings.py -> in the last add:

```
AUTH_USER_MODEL = 'core.User'
```



```
$ docker-compose run app sh -c "python manage.py makemigrations core"
```

```
$ docker-compose run app sh -c "python manage.py test && flake8"
```



```
$ git add .
$ git commit -m "Added custom user model."

$ git push origin
```



create test_admin.py in tests folder:

```python
from django.contrib.auth import get_user_model
from django.test import TestCase, Client
from django.urls import reverse


class AdminSiteTests(TestCase):

    def setUp(self):
        """setUp method user as a setup helper for other testing methods"""
        # self in self.client to create instance variable accessible in other functions
        self.client = Client()

        self.admin_user = get_user_model().objects.create_superuser(
            email='admin@hany.ws',
            password='AdminPass123'
        )

        self.client.force_login(self.admin_user)

        self.user = get_user_model().objects.create_user(
            email='test@hany.ws',
            password='TestPass123',
            name='Test user full name'
        )

    def test_users_listed(self):
        """Test that users are listed on user page"""
        url = reverse('admin:core_user_changelist')
        res = self.client.get(url)

        self.assertContains(res, self.user.name)
        self.assertContains(res, self.user.email)

    def test_user_change_page(self):
        """Test that the user edit page works"""
        url = reverse('admin:core_user_change', args=[self.user.id])
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)

    def test_create_user_page(self):
        """Test that the create user page works"""
        url = reverse('admin:core_user_add')
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)

```



admin.py

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext as _

from core import models


class UserAdmin(BaseUserAdmin):
    ordering = ['id']
    list_display = ['email', 'name']
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (_('Personal Info'), {'fields': ('name',)}),
        (
            _('Permissions'),
            {'fields': ('is_active', 'is_staff', 'is_superuser')}
        ),
        (_('Important dates'), {'fields': ('last_login',)})
    )
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'password1', 'password2')
        }),
    )


admin.site.register(models.User, UserAdmin)
```



```
$ docker-compose run app sh -c "python manage.py test && flake8"
```



```
$ git add .
$ git commit -m "updated admin to support custom user model."

$ git push origin

```

