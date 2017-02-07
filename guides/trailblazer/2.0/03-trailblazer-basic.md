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

## CRUD for Post

In the chapter 02 of any rails tutorial the CRUD procedure is shown and TRB hasn't changed anything about it. `Routes` and `controllers` are excatly the same as any other rails application but obviously what we call, in particular, in the 'controllers' are different and most of the time cleaner and more elegant than many Rails applications.

Here an example of what `BlogPostController` looks like with TRB:

{{ "app/controllers/blog_posts_controller.rb:postcontroller:../trailblazer-guides:operation-03" | tsnippet }} 

No worries we will explore every step of it.

### New

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
The pipe flow is nothing different than the `BlogPost::Create` operation shown in the second guide, the only different is that we only build the contract becuase we just need to present form to the User.

[Render](http://trailblazer.to/gems/cells/api.html#render) means calling the `Trailblazer::Cell` that will elaborate the data from the `TRB::Op` and present the view files.

Here the cell class: 









