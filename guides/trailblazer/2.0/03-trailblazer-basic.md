---
layout: operation2
title: 03- Trailblazer Basics
gems:
  - ["trailblazer", "trailblazer/trailblazer", "2.0"]
description: "Having discussed the operation mechanics, we now explore the world of Trailblazer macros, how to leverage contracts and validations, and focus on strong tests for the business logic."
imageurl: http://trailblazer.to/images/summary/guide-02.png
---

{% row %}
Now that you know and curios about Trailblazer, we are going to learn how to use it in a rails application.
{% endrow %}

## The Challenge

In this third guide we are going to learn how to build a basic Rails app using TRB and in particular the basic CRUD in order to create a Post for a Blog.

As mentioned before TRB is using different gem to looking after different layers of a rails app which means code organised and easy to read (if you know TRB).
Furthermore TRB allows implementing a new feature without reviewing all classes in your project.

{% callout %}
Code for this session is [here](https://github.com/trailblazer/guides/tree/operation-03).
{% endcallout %}

## Gemfile
In this third guide we are goin to use `slim` and [formular](https://github.com/trailblazer/formular) for views, [Cell](http://trailblazer.to/gems/cells/) in order to have a much easier and more organised UI and `minitest` so our `gemfile` is a bit different than the guide 2:

```
source 'https://rubygems.org'

gem "trailblazer", ">= 2.0.3"
gem "activerecord", "= 4.2.6"
gem "sqlite3"
gem "dry-validation"

group :test do
  gem "memory_test_fix"
  gem "minitest-rails-capybara"
  gem "minitest-line"
  gem "minitest-bang"
end

gem 'formular', :github => "trailblazer/formular"
gem 'trailblazer-cells'
gem 'trailblazer-rails'
gem 'cells-rails'
```

## Folder Structure

Trailblazer offers you a new, more intuitive file layout so how project will look like that:

```
app
├── concepts
│   ├── comment
│   │   ├── cell
│   │   │   ├── show.rb
│   │   │   ├── index.rb
│   │   ├── contract
│   │   │   ├── create.rb
│   │   │   ├── update.rb
│   │   ├── operation
│   │   │   ├── create.rb
│   │   │   ├── update.rb
│   │   ├── view
│   │   │   ├── show.haml
│   │   │   ├── index.rb
│   │   │   ├── comment.css.sass
├── controllers
│   ├── application_controller.rb
│   ├── blog_post_controller.rb
├── models
│   ├── blog_post.rb
│   ├── user.rb

```
Instead of grouping by technology, classes and views are structured by *concept*, and then by *technology*. A concept can relate to a model, or can be a completely abstract concern such as invoicing.

The structure is stratight forward in terms of where to put my code and it must be followed in order to have TRB working properly.

## Overview Trailblazer Cells
Before starting with the code is better to have an overview of `Cells` in case you have never worked with it.

A cell is an object that represent a fragment of your UI. The scope of that fragment is up to you: it can embrace an entire page, a single comment container in a thread or just an avatar image link.

In other words: **A cell is an object that can render a template**.

Cells are faster than ActionView. While exposing a better performance, you step-wise encapsulate fragments into cell widgets and enforce interfaces.

{% callout %}
We suggest to have a look [here](http://trailblazer.to/gems/cells/getting-started.html) for more info.
{% endcallout %}

## CRUD for Post

In the chapter 02 of any rails tutorial the CRUD procedure is shown and TRB hasn't changed anything about it. `Routes` and `controllers` are excatly the same as any other rails application but obviously what we call, in particular, in the 'controllers' are different and most of the time cleaner and more elegant than many Rails applications.

Here an example of what `BlogPostController` looks like with TRB:

{{ "app/controllers/blog_posts_controller.rb:postcontroller:../trailblazer-guides:operation-03" | tsnippet }} 

No worries we will explore every step of it.

### New && Create

As per the second guide we need to use a model for BlogPost and User which are in `app/models/` and looks like:

{{ "app/models/blog_post.rb:model:../trailblazer-guides:operation-03" | tsnippet }}

{{ "app/models/user_post.rb:model:../trailblazer-guides:operation-03" | tsnippet }}

The `User` model is just a `signed_in?` flag that we are going to use in order to allow creating the Post.

In order to have a `New` post we need follow few **SPTES** which is at the base of the railway concept in TRB:
1- make sure that a User is signed in
2- create a `BlogPost` model
3- present a form that the User will use to add `Title` and `Body` 

Here the code for the operation:

{{ "app/concepts/blog_post/operation/new.rb:newop:../trailblazer-guides:operation-03" | tsnippet }}

Here the code for the contract:

{{ "app/concepts/blog_post/contract/create.rb:contract:../trailblazer-guides:operation-03" | tsnippet }}

The `new` method in our controller will be:

{{ "app/controllers/blog_posts_controller.rb:new:../trailblazer-guides:operation-03" | tsnippet }}

Using `run TRB::Op` or `TRB::Op.()` (call) will procedure the same results, the only different is that is possible to pass argument to `.()` like for example `"current_user" => User` or more interesting [inject dependecy to the operation](http://trailblazer.to/guides/trailblazer/2.0/02-trailblazer-basics.html#dependencies).

Calling an operation will always return a [Result Object](http://trailblazer.to/gems/operation/2.0/api.html#result-object) and in base on which steps are implemented this object will be populated so for example in this case it will have a `Policy`, `Model` and `Form` objects (it actually has much more but this is what we need now).
The pipe flow is nothing different than the `BlogPost::Create` operation shown in the second guide, as long as all steps return true the only different is that we only build the contract becuase we just need to present the form to the User.

Now that we have data structure we need to actually show it to the User therefore we call `render BlogPost::Cell::New` and we pass the `Reform::Form` object to the cell class with `result["contract.default"]`.
We actually need to pass more information to the cell in order to have better looking and smarter application, so we need to pass for exaple `current_user`, the `layout` and all the information that the cell class needs to elaborate. This more information are basically the same for all the cell class so we create a `render` method in `application_controller`:

{{ "app/controllers/application_controller.rb:render:../trailblazer-guides:operation-03" | tsnippet }}

In this example we are skipping all the css/layout preparation and also we are going to pass `current_user` as a normal User without actually log in but the concept is there.

Here `BlogPost::Cell::New`: 

{{ "app/concepts/blog_post/cell/new.rb:newcell:../trailblazer-guides:operation-03" | tsnippet }}

Here the view:
```
.row.new
  h1 New Post

  = trb_form_for(model, "/posts", method: :post, builder: :bootstrap4, id: :new_post) do |f|
    .row
      .col-sm-12
        = f.input :title, placeholder: "Title"
    .row
      .col-sm-12
        = f.textarea :body, placeholder: "What do you wanna say?"
    .row
      .col-sm-12
        = "by " + user_name
        = f.input :author, value: user_name, type: "hidden"
        = f.input :user_id, value: current_user.id, type: "hidden"
    .row
      .col-sm-12  
          = f.submit(value: "Create Post")
```

The first rule of cells gem is that for each cell class we need a template in the view folder which must have the same name so `cell/new.rb` --> `view/new.slim` otherwise you will have a `MissingTemplate Error`.

First in first we need to include all the helpers in order to present the different inputs for the form and the possible errors.

Then we can implement any method to pass data to the view, in this case `user_name` will pass either the email or the firstname of the User as author of the post.

Ones User clicks on `Create Post` the `create` method in `blog_posts_controller` will be called:

{{ "app/controllers/blog_posts_controller.rb:create:../trailblazer-guides:operation-03" | tsnippet }} 


