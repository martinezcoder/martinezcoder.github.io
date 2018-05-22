---
layout: post
title:  "Create a new Rails project with Docker compose"
date:   2018-05-22 18:40:00 +0200
categories: jekyll update
permalink: /rails-project-with-docker/
---

This guide shows you how to use **Docker Compose** to set up and run a **Rails/PostgreSQL** app. 

## Define the project

We need to define four files to start a complete Rails project with Docker:

* Dockerfile
* docker-compose.yml
* Gemfile
* Gemfile.lock

Let's start with the `Dockerfile`. This file defines an image with a Docker container with Ruby and its dependencies. The application code will be located in the `myapp` directory.

```bash
# Dockerfile

FROM ruby:2.5.0
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp
```

We will need an existent `Gemfile` and `Gemfile.lock`. The `Gemfile.lock` is just an empty file:

```sh
$ touch Gemfile.lock
```

About the `Gemfile` we just need to create a bootstrap one to load Rails. It will be overwriteen with the call to `rails new`.

```sh
# Gemfile

source 'https://rubygems.org'
gem 'rails', '5.1.5'
```

Finally, create a `docker-comspose.yml` file:

```bash
# docker-compose.yml

version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
    tty: true
    stdin_open: true
```

See that we have defined a couple of services: `web` and `db`, and that `web` depends on `db`. That means that the web knows the existence and have access to the database.

## Build the project

Having those files created, you just need to run next command:


```sh
docker-compose run web rails new . --force --database=postgresql
```

You will see that the content of `Gemfile` has changed after this last command. Remember that the `Dockerfile` moves this file into the image and we have now a different one. Because of this change, you need to build again:

```sh
docker-compose build
```

## Connect the database

The service of the databse is called `db` in the `docker-compose.yml`. We need to configure the `database.yml` in order to access to this service. To do so, we need to add the `host` and the `username` part of the file:

```sh
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
```

### Postgres port

By default, postgres opens the port `5432`. If you have a running local instance of _postgres_ this port will be already opened and you won't be able to run the Docker one. Given that case you have two options:

 * stop your local instance of postgres
 * change the port that uses your docker in you image. That could be done adding a `port` tag in the `db` configuration in `docker-compose.yml`.

## Try it!

Run migrations:

```
docker-compose run web bundle exec rails db:create db:migrate
```

This will run all the services:

```
docker-compose up
```

This will run an specific service and its dependencies:

```
docker-compose run --service-ports web
```

From here, you can work as always. Maybe it's timne to create a new model. Remeber that you have to use docker in any call:

```sh
docker-compose run web bundle exec rails g model User name:string
```


