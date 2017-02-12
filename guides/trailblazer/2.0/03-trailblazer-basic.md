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

In this third guide we are going to learn how to build a basic Blog app using TRB 2.

As mentioned before TRB uses different gems to looking after different layers of a rails app which means code organised and easy to read (if you know TRB).
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

gem "rake", "~> 10.0"
gem 'minitest'

gem 'formular', :github => "trailblazer/formular"
gem 'trailblazer-cells'
gem 'trailblazer-rails'
gem 'cells-rails'
```

## Folder Structure

Trailblazer offers you a new, more intuitive file layout:

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

The structure is stratight forward in terms of where to put the code and it must be followed in order to have TRB working properly.

## Overview Trailblazer Cells

Before starting with the code is better to have a quick overview of `Cells` in case you have never worked with it.

A cell is an object that represent a fragment of your UI. The scope of that fragment is up to you: it can embrace an entire page, a single comment container in a thread or just an avatar image link.

In other words: **A cell is an object that can render a template**.

Cells are faster than ActionView. While exposing a better performance, you step-wise encapsulate fragments into cell widgets and enforce interfaces.

{% callout %}
We suggest to have a look [here](http://trailblazer.to/gems/cells/getting-started.html) for more info.
{% endcallout %}

## CRUD for Post

We are back to chapter 02 of any rails tutorial where the CRUD procedure is illustrated, well, `routes` and `controllers` are excatly the same as any other rails application but obviously what we call, in particular, in the 'controllers' are different.

Here an example of what `BlogPostController` looks like with TRB:

{{ "app/controllers/blog_posts_controller.rb:postcontroller:../trailblazer-guides:operation-03" | tsnippet }} 

No worries we will explore every step of it.

### New && Create

Here the model for BlogPost and User:

{{ "app/models/blog_post.rb:model:../trailblazer-guides:operation-03" | tsnippet }}

{{ "app/models/user.rb:model:../trailblazer-guides:operation-03" | tsnippet }}

The `User` model has the property `signed_in` which is just a flag that we are going to use in order to have `current_user`(we should use an authorization gem for this though like [tyrant](https://github.com/apotonick/tyrant)).

In order to have a `New` post we have to implement few **SPTES** which is at the base of the railway concept in TRB:
- make sure that a User is signed in
- create a `BlogPost` model
- present a form that the User will use to add `Title` and `Body` 

Here the operation:

{{ "app/concepts/blog_post/operation/new.rb:newop:../trailblazer-guides:operation-03" | tsnippet }}

And the contract:

{{ "app/concepts/blog_post/contract/create.rb:contract:../trailblazer-guides:operation-03" | tsnippet }}

The `new` method in our controller will be:

{{ "app/controllers/blog_posts_controller.rb:new:../trailblazer-guides:operation-03" | tsnippet }}

Using `run TRB::Op` or `TRB::Op.()` will procedure the same result, the only different is that using `.()` is possible to [inject dependecy to the operation](http://trailblazer.to/guides/trailblazer/2.0/02-trailblazer-basics.html#dependencies) like for example `"current_user" => user`.

Calling an operation will always return a [Result Object](http://trailblazer.to/gems/operation/2.0/api.html#result-object) and in base on which steps are implemented this object will be populated so for example in this case it will have a `policy`, `model` and `form` object (it actually has much more but this is what we need now).
The pipe flow is nothing different than the `BlogPost::Create` operation shown in the second guide, the only different is that we only build the contract becuase we just need to present the form to the User.

We can quickly test the pipeflow trying to create a post without a user signed in:

{{ "test/concepts/blog_post/operation/operation_test.rb:policy:../trailblazer-guides:operation-03" | tsnippet }}

In the test we verify that if the user is not signed in the model will be nil and the policy object will return fasle which will move the pipeflow to the left and won't let the other steps to be executed.

Now that we have the data structure we need to actually show it to the User therefore we call `render BlogPost::Cell::New` and we pass the `Reform::Form` object to the cell class with `result["contract.default"]`.
We actually need to pass more information to the cell in order to have a better looking and smarter application, so we need to pass for exaple `current_user`, `flash` messages, the `layout` and all the information that the cell class needs to elaborate. This more information are basically the same for all the cell classes so we create a `render` method in `application_controller`:

{{ "app/controllers/application_controller.rb:render:../trailblazer-guides:operation-03" | tsnippet }}

In this guide we are skipping all the css/layout preparation and also we are going to pass `current_user` as a normal User without actually log in but the concept is there.
We use another trick to pass `current_user` in all the operations which is necessary when we actually run the app or we implement integration tests:

{{ "app/controllers/application_controller.rb:render:../trailblazer-guides:operation-03" | tsnippet }}

Here `BlogPost::Cell::New`: 

{{ "app/concepts/blog_post/cell/new.rb:newcell:../trailblazer-guides:operation-03" | tsnippet }}

Here the view:
```
.row.new
  h1 New Post

  = simple_form_for model do |f|
    .row
      .col-sm-12
        = f.input :title, placeholder: "Title", label: false
    .row
      .col-sm-12
        = f.input :subtitle, placeholder: "Title", label: false
    .row
      .col-sm-12
        = f.textarea :body, placeholder: "What do you wanna say?", label: false
    .row
      .col-sm-12
        = "by " + user_name
        = f.input :author, :value => user_name, :as => :hidden
        = f.input :user_id, :value => current_user.id, :as => :hidden
    .row
      .col-sm-12  
        = f.submit 'Create Post'
```

First in first in cell we need to have for each cell class a template in the view folder which must have the same name of the cell class so `cell/new.rb` --> `view/new.slim` otherwise you will have a `MissingTemplate Error`.

All the helpers are necessary in order to present correctly the different inputs for the form and the possible errors.
It's possible to implement any method where most of the logic should be instead of in the views, in this case `user_name` will pass either the email or the firstname to use as author in the post form (it's possible to create a User only with the email so this is used to make sure that we don't have `nil` as author).

Ones User clicks on `Create Post` the `create` method in `blog_posts_controller` will be called:

{{ "app/controllers/blog_posts_controller.rb:create:../trailblazer-guides:operation-03" | tsnippet }} 

This will run `BlogPost::Create` and in case of a success `result` a flash message might be shown and we would redirect to the `rooth_path`, otherwise the same `BlogPost::Cell::New` will be rendered because something went wrong in the `create` operation:

{{ "app/concepts/blog_post/operation/create.rb:createop:../trailblazer-guides:operation-03" | tsnippet }}

With the first step we introduce the concept of [Nested Operations](http://trailblazer.to/gems/operation/2.0/api.html#nested), this basically add all the `BlogPost::New` steps in the current operation, which means that with one step we check if a user is still signed in, create a new model and build the contract. In case all the steps in the Nested operation return true, `Contract::Validate` is finally called and here it's where most of the time *things can go wrong*. Basically if the validations implemented in the contract are not sutisfied the pipeflow will move to failure pipe and all the next steps won't be exucuted (if a `failure` step has not been implemented) and the operation will return failure result which means rendering the `Cell::New` and showing the errors.
Let's have a look to a test to understand this better:

{{ "test/concepts/blog_post/operation/operation_test.rb:validation:../trailblazer-guides:operation-03" | tsnippet }}

Keeping in mind that the first argument when we call, `.()`, an operation is **always** `params` we inject `"current_user"`to make sure that the nested `BlogPost::New` will return true so the contract can be validated which means elaborate `params`.
When the hash params is empty, the operation will return false and all the errors depending on the validations. Just to make it more clear there is another test where `body` is correct therefore the errors for `body` are not shown.

If the User is *kind* enough to fill in all the inputs as requested the operation can finally call `Contract::Persist` which basically copy the data from the `contract` to the `model` and then call `model.save` which means **we have just created a new post**.
We are not actually sure about this until we are tesing it:

{{ "test/concepts/blog_post/operation/operation_test.rb:create:../trailblazer-guides:operation-03" | tsnippet }}

Passing `current_user` and the hash `params` correctly we expect that the operation will execute successfully, that the title in the model is correct, that we have one `BlogPost` in the database and we should also test that a notification has been sent.

## Index && Show

Now that we have a Post to show we can have a look how to preset a Post lists and a single Post.

To present the list of the posts we use the action `index` in `blog_posts_controller`:

{{ "app/controllers/blog_posts_controller.rb:index:../trailblazer-guides:operation-03" | tsnippet }} 

Here the operation:

{{ "app/concepts/blog_post/operation/index.rb:indexop:../trailblazer-guides:operation-03" | tsnippet }}

Really simply we want to get all the `BlogPost` in the database and save it into the model so we can pass in the cell class (check the second argument of the controller `result["model"]`).
Here it's possible to filter the data to show for example if the post has a status you may want to pass in the model only the `BlogPosts` that have been approved by the administrator, so our `model!` step would be:

```
def model!(options, *)
  options["model"] = ::BlogPost.where("status like ?", "Approved").reverse_order
end
```

Here the cell class:

{{ "app/concepts/blog_post/cell/index.rb:index:../trailblazer-guides:operation-03 | tnippet"}}

and the view:

```
h6 = total
= cell("post/cell/item", collection: model)
```

The cell class is used just as a decoration, so in case there is no `BlogPost` the string `"No posts"` is shown.
Much more insteresting is the view, we bascially call another cell class `Item` in order to show the actual a sort of summary of the `BlogPosts` and because we pass a `collection` as second argument, in case model has size more than 1, this will create the list of `BlogPosts` that we want.

Here the `BlogPost::Cell::Item`:

{{ "app/concepts/blog_post/cell/item.rb:item:../trailblazer-guides:operation-03 | tnippet"}}

and the view:

.post-preview
  h2.post-title
    = title
  h3.post-subtitle
    = subtitle
  p.post-meta
    = "Posted by #{author} on #{time.strftime("%d %B %Y")}"
hr  

In this way each element of the list will have the structure above and using either the `Title` or the `Subtitle` we access to the actual `BlogPost`.
*In light of a really simple `Index` operation, we don't have any test for it.*

The `show` action is a little bit more exciting, here the controller:

{{ "app/controllers/blog_posts_controller.rb:show:../trailblazer-guides:operation-03" | tsnippet }} 

Here the operation:

{{ "app/concepts/blog_post/operation/show.rb:showop:../trailblazer-guides:operation-03" | tsnippet }}

`BlogPost::Show` needs the `id` as argument and we use `find_by` because in case the `BlogPost` is not found which it returns a `nil` model and the flow is moved in the failure pipe. In this way we can use a trick to show a "fake" `BlogPost` with title `Post not found!` and not having a breaken application, here how:
  
{{ "app/concepts/blog_post/operation/lib/error.rb:error:../trailblazer-guides:operation-03" | tsnippet }}

Let's have a loot to the test:

{{ "app/test/concepts/operation/blog_post/operation_test.rb:not_found:../trailblazer-guides:operation-03" | tsnippet }}

As shown here if you pass a not existing `id` the result of the `show` operation is still false but it shows a not found Post in order to correctly render the `Cell::Show` and to let the `User` undestands that something went wrong and keep using the web-application. 

## Edit && Update

Here we have the same concept in terms of presenting a form and actually validate and save the data.
Therefore `BlogPost::Edit` will preset a form building a contract and `BlogPost::Update` will nest the edit operation, validate and save the data.

The controller actions will be:

{{ "app/controllers/blog_post_controller.rb:edit-update:../trailblazer-guides:operation-03" | tsnippet }}

**We need to present and elaborate a form so we pass `result["contract.default"]` to the cell classes**

Here the `BlogPost::Contract::Edit`:

{{ "app/concepts/blog_post/contract/edit.rb:contract:../trailblazer-guides:operation-03" | tsnippet }}

The only different with `Contract::Create` is the `user_id` property, just because we decide to not modify it.
Using a different contract allows us to have different validations and also modify part of the model, for example in `User` you don't want to change the password everytime the User decides to change the address so you will have 2 different contracts in order to change the password and to change the user details.

Here the operation:

{{ "app/concepts/blog_post/operation/edit.rb:editop:../trailblazer-guides:operation-03" | tsnippet }}

Same as for the `BlogPost::Show` we use the same trick in case `BlogPost` is not found and we introduce the new feature [fail_fast](http://trailblazer.to/gems/operation/2.0/api.html#flow-control-fail-fast-option). Basically if the `failure` step that has `true` as `fail_fast` it's executed, the operation will stop the execution on that step. We need this in our case otherwise all the failure steps after `failure BlogPost::Error, fail_fast: true` will be executed.

After making sure that we have a real BlogPost we apply our `Policy`, so we check if first in first `current_user` is != nill and then we check if `current_user` is either the owner of the `BlogPost` or the admin. In case the `Policy` is not satisfied we raise a `NotAuthorizedError`, so we declare a class from `RunTimeError` and we rescue it when it is raised:

{{ "app/controllers/application_controller.rb:notauthorized:../trailblazer-guides:operation-03" | tsnippet }}

In the rescue we just show a Not Authorized flash message and redirect to the `rooth_path`.
The `NotAuthorizedError` is raised in `failure BlogPost::ThrowException`:

{{ "app/concepts/blog_post/lib/throw_exception.rb:exception:../trailblazer-guides:operation-03" | tsnippet }}

If everything is ok we finally build the contract and preset a form, here the cell class:

{{ "app/concepts/blog_post/cell/edit.rb:edit:../trailblazer-guides:operation-03" | tsnippet }}

and the view:

```
.row
  h1 Edit Post

  = simple_form_for model do |f|
    .row
      .col-sm-12
        = f.input :title, placeholder: "Title", label: false
    .row
      .col-sm-12
        = f.input :subtitle, placeholder: "Subtitle", label: false
    .row
      .col-sm-12
        = f.textarea :body, placeholder: "What do you wanna say?", label: false
    .row
      .col-sm-12
        = "by " + user_name
    .row
      .col-sm-6  
        = f.submit 'Save'
      .col-sm-4
        h6 = delete
      .col-sm-2
        h6 = back
```

The update operation below is basically the same of `BlogPost::Create`:

{{ "app/concepts/blog_post/operation/update.rb:updateop:../trailblazer-guides:operation-03" | tsnippet }}

