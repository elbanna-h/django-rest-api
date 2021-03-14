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









