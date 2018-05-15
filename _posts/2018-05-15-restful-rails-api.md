---
layout: post
title:  "RESTful JSON API With Rails 5"
date:   2018-05-15 10:40:00 +0200
categories: jekyll update
permalink: /restful-rails-json-api/
---

The [Rails documentation](http://api.rubyonrails.org/) says that _Ruby on Rails is a web-application framework that includes everything needed to create database-backed web applications according to the Model-View-Controller (MVC) pattern_. However, several companies were using rails just to create a backend API, and that was translated into having tons of unnecessary code dependencies.  That's why since version 5, Rails started offering a way to create an **API-only** applications, which only provides the required things to have a super powerful API.

Find a more detailed explanation in the [API-only documentation](http://guides.rubyonrails.org/api_app.html).


This post is a self-guide to create a new API-only project. It's not a tutorial that resolves and exercise, but a guide to remember the steps to follow in the creation of an API from scratch.

## Creating an API-only rails project

To start creating an API-only Rails applications, ensure that your Rails version is equal or higher than 5.

```sh
> rails -v
Rails 5.2.0
```

Then create a new **API-only** application:

```sh
> rails new my-api --api -T
```

The `--api` part, tells Rails that we want an **API-only** application. The `-T` part is necessary if you want to avoid the use _Minitest_ as the default testing framework. I normally use `rspec`, so I would include it.

Arriving here, you have a _Rails API-only_ application with these properties:

* It still provides things out of the box. It provices a set of middleware with all the stuff we could need for an API.

* The `ApplicationController` inherit from `ActionController::API` instead of from `ActionController::Base`.

* As any other Rails application, it will provide `routing` resources and helpers, `caching`, `authentication`, etc.


## Project Setup

This would depend on personal preferences, but I normally setup every new project with this gems:
 
* [rspec-rails](https://github.com/rspec/rspec-rails) - I have nothing against _Minitest_, but I normally use Rspec testing framework. It is needed to run `rails generate rspec:install` to install it in a project.
* [shoulda_matchers](https://github.com/thoughtbot/shoulda-matchers) - Include more matchers to Rspec
* [factory_bot_rails](https://github.com/thoughtbot/factory_bot_rails) - Very userful tool to use fixtures.
* [faker](https://github.com/stympy/faker) - A library for generating fake data such as names, addresses, and phone numbers.
* [database_cleaner](https://github.com/DatabaseCleaner/database_cleaner) - We can ensure a clean environment before the execution of each test. It needs to include a piece of code in our `rails_helper.rb` file.

Find [here](https://github.com/martinezcoder/coding-challenge-backend/pull/1/commits/6793b23de244f021b30880fff8a6bb8bb23df66d) a commit adding all these dependencies.

## API Endpoints

Now that we have our project to start, it's the best moment o start thinking in the endpoints we will need. It will help us to have a clear idea of the models and the controllers that we will need.

Write down a draft of the endpoints.


## Model definition

We have a list of endpoints, so we know more or less the main models and its associations. 

- Create the database: models, associations, migrations
- **TDD**: add tests to check associations and presence validations
- Fix the tests adding the validations and associations to the models.
- Create factories for each model using `Faker` value. They will be useful in the next step.

At this point we have valid models and its associations and validations.

## Controllers and routes

 - Generate the required controllers with `rails g controller Name`.
 - Create `requests` tests. You can use [this one](https://github.com/martinezcoder/coding-challenge-backend/blob/test/spec/requests/zombies_spec.rb) as a guide.
 - The errors in the tests will guide you to add the required routes.


### Responses

I like controller responses with this structure:

```ruby
render json: object, status: status
```

Look at this [example](https://github.com/martinezcoder/coding-challenge-backend/blob/test/app/controllers/zombies_controller.rb).

### Exception handler

In the case that a searched record does not exist, Rails will through an error. Instead or raising the error, it would be necessary to rescue that error and notify the user with the problem.

We should rescue any kind of error on which we need to notify the API client. To do this, we can create a concern:

```ruby
# app/controllers/concerns/exception_handler.rb
module ExceptionHandler
  # provides the more graceful `included` method
  extend ActiveSupport::Concern

  included do
    rescue_from ActiveRecord::RecordNotFound do |e|
      render json: { message: e.message }, status: :not_found
    end

    rescue_from ActiveRecord::RecordInvalid do |e|
      render json: { message: e.message }, status: :unprocessable_entity)
    end
  end
end
```

And include it in the `ApplicationController`:

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  include ExceptionHandler
end
```

## Versioning

It is not necessary to add _versioning_ in your first version. But when the day comes in which you have to add a second version, you can afford it in different ways.

Have in mind that in any kind of solution you would have to 

* Add a route constraint to select the version
* Use different namespaces for the controllers, depending on the version.

There are different options to apply versioning in our project. The most common are:

* Using [different `routes` and `namespaces`](https://chriskottom.com/blog/2017/04/versioning-a-rails-api/). 
* Selecting the version from the headers (using a [route constraint](http://guides.rubyonrails.org/routing.html#advanced-constraints)). Follow this [documentation](https://scotch.io/tutorials/build-a-restful-json-api-with-rails-5-part-three#toc-versioning).

## Serializers


I have always used [Active Models Serializers](https://github.com/rails-api/active_model_serializers), but I recently discovered [Fast JSON API](https://github.com/Netflix/fast_jsonapi) from Netflix, and sounds promising. I will use that last one in my next project.

https://github.com/Netflix/fast_jsonapi


## Pagination

Any serious production API have to include pagination. [Will_paginate](https://github.com/mislav/will_paginate) or [kaminari](https://github.com/kaminari/kaminari) will do the trick.


## Authentication and authorization

Follow [this](https://scotch.io/tutorials/build-a-restful-json-api-with-rails-5-part-two) guide.

For **authorization**, the gem [Pundit](https://github.com/varvet/pundit) is great.


## Caching

Conditional **GET**s: Rails handles conditional GET (**ETag** and **Last-Modified**) processing request headers and returning the correct response headers and status code. All you need to do is use the [`stale?`](http://api.rubyonrails.org/v5.2.0/classes/ActionController/ConditionalGet.html#method-i-stale-3F) check in your controller, and Rails will handle all of the HTTP details for you.

