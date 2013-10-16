---
layout: post
title: "Ruby on Rails 4 and Batman.js - Another Getting Started Tutorial"
date: 2013-10-06 22:08
comments: true
author: Neil Rosenstech
categories:
- ruby on rails
- batmanjs
---

I’ve never been using JS frameworks before, this is why I wanted to give it a shot. Many options here: AngularJS, Ember.js, Backbone.js, etc, makes the choice pretty hard
when you don’t know the subtleties of each. As a Rails developer, I chose Batman.js because of its “Rails orientation” and because I know that [Shopify](http://www.shopify.ca/) folks here in Canada
are working hard to make things easy for us Rails developers (yes, they are developing Batman.js).

<!-- more -->

Throughout this tutorial I’m going to show you how I quickly built a simple post/comments application with Ruby on Rails 4 and Batman.js.

At the time I’m writing these lines, our friends of Gotham City are updating the [documentation](http://batmanjs.org/docs/index.html), so be sure to check it out: a lot of helpful, valuable resources.

You can find the application github repo here: <https://github.com/Raindal/batman_js_blog>.

I’m eager to better myself with the framework so any comment will be highly appreciated.

Let’s dive in!

## On the server side

Here I’m assuming that you already installed Rails 4. We’re first going to create a new application called **batman_js_blog**.
Let’s skip the documentation part with the following line.

    rails new batman_js_blog --no-ri --no-rdoc

And just to be clear, `cd` into the application.

    cd batman_js_blog

> Tired of using `cd` everytime? Check out [my previous post](http://requiremind.com/why-i-love-my-development-environment) and start using ZSH now!

### The models

The models are pretty simple, the posts will have a **title** and a **content**. The comments will have a **content** only and a reference to a post (**post_id**).

    rails g model post title:string content:text
    rails g model comment content:text post:references

Let’s get the associations right: a post *has many* comments, therefore, a comment *belongs to* a post. Let’s create some validations for future usage
(see *Errors Handling* at the end).

``` ruby app/models/post.rb
class Post < ActiveRecord::Base
  has_many :comments

  validates :title,   presence: true
  validates :content, presence: true
end
```

Using `references` when generating the Comment model should already get this part right with the `belongs_to`:

``` ruby app/models/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :post
end
```

Run the migrations to create both tables, as usual.

    rake db:migrate

We’re going to create some seeds to populate the database with fake data to work with. Let’s create 20 posts with 5 comments each.

``` ruby db/seeds.rb
20.times do |i|
  post = Post.create(title: "Post #{i}", content: "Some awesome content")
  5.times { |j| post.comments.create(content: "Comment n°#{j}") }
end
```

Run the seeds.

    rake db:seed

Here we are using nested routes (nested resources) to reflect the associations we created earlier.  
This will enable routes like /posts/:post_id/comments/:id.

``` ruby config/routes.rb
resources :posts do
  resources :comments
end
```

### The controllers

First you can create a **posts_controller.rb** file in your controllers folder.

As we only need to serve **json**, we’re going to use the handy `respond_to` method at the top of our controller. Then, in every action we only have to use `respond_with`,
that will basically call `to_json` on our resources.

When displaying a specific post, we want to display its comments as well, so we need to return them with the post, this is why we use `include: :comments` in the show action.

We do not need the `new` and `edit` actions, because Batman.js will take care of showing the form for us (it can because those actions do not require an interaction
with the database).

The rest of the controller is pretty much straightforward if you know your way around with Rails (which I assume).

``` ruby app/controllers/posts_controller.rb
class PostsController < ApplicationController
  respond_to :json

  def index
    respond_with Post.all
  end

  def show
    respond_with Post.find(params[:id]), include: :comments
  end

  def create
    respond_with Post.create(post_params)
  end

  def update
    post = Post.find(params[:id])
    post.update_attributes(post_params)
    respond_with post
  end

  def destroy
    respond_with Post.find(params[:id]).destroy
  end

  private

    def post_params
      params.require(:post).permit(:title, :content)
    end
end
```

The comments controller is rather simple too. We’re not going to list all the comments, neither are we going to show one comment on its own,
we’re not allowing the edition of a comment either. That means we don’t need the index, show, and update actions.

The only thing we want to do is display related comments on every post's show view and a small form for adding a comment (plus a link for destroying each post).  
As I stated before, the comments are returned with every post thanks to `include: :comments` so we don't have to take care of this. The only thing left to do is enable
comments creation and destruction.

You can create a **comments_controller.rb** file:

``` ruby app/controllers/comments_controller.rb
class CommentsController < ApplicationController
  respond_to :json

  def create
    post = Post.find(params[:post_id])
    comment = post.comments.build(comment_params)
    comment.save
    respond_with(post, comment)
  end

  def destroy
    comment = Comment.find(params[:id])
    respond_with(comment.post, comment.destroy)
  end

  private

    def comment_params
      params.require(:comment).permit(:content)
    end
end
```

You can now run a server with `rails s` and navigate to <http://localhost:3000/posts.json> to check that you do have a json posts list displayed.
You can check that each post is returned with its associated comments by navigating to <http://localhost:3000/posts/1.json>.

## On the client side

Now we can really crack into the subject. First of all, let’s add the gem to our gemfile and run `bundle` to install it.

``` ruby Gemfile
source 'https://rubygems.org'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '4.0.0'

gem 'batman-rails'

...
```

    bundle

We can now generate the skeleton of our Batman.js app, it’s going to reside in **app/assets/batman**.

    rails g batman:app

This created the following folders and files:

    create  app/controllers/batman_controller.rb
    create  app/views/layouts/batman.html.erb
    insert  config/routes.rb
    create  app/assets/batman/batman_js_blog.js.coffee
    create  app/assets/batman/models
    create  app/assets/batman/models/.gitkeep
    create  app/assets/batman/views
    create  app/assets/batman/views/.gitkeep
    create  app/assets/batman/controllers
    create  app/assets/batman/controllers/.gitkeep
    create  app/assets/batman/html
    create  app/assets/batman/html/.gitkeep
    create  app/assets/batman/lib
    create  app/assets/batman/lib/.gitkeep
    create  app/assets/batman/html/main
    create  app/assets/batman/controllers/application_controller.js.coffee
    create  app/assets/batman/controllers/main_controller.js.coffee
    create  app/assets/batman/html/main/index.html
    create  app/assets/batman/views/main/main_index_view.js.coffee
    prepend  app/assets/batman/batman_js_blog.js.coffee
    prepend  app/assets/batman/batman_js_blog.js.coffee
    prepend  app/assets/batman/batman_js_blog.js.coffee
    prepend  app/assets/batman/batman_js_blog.js.coffee

You can navigate to <http://localhost:3000> to see Batman.js home page.

### Sneak peek

If you open the **index view**, you're probably going to see something like this somewhere in the file:

``` html app/assets/batman/html/main/index.html
<div><label>First Name:</label><input type="text" data-bind="firstName"></div>
<div><label>Last Name:</label><input type="text" data-bind="lastName"></div>
<div data-showif="hasName">Hello, my name is <span data-bind="fullName"></span></div>
<div><button data-event-click="resetName">Reset name</button></div>
```

Ok, now you can finally catch a glimpse of Batman.js... And I'm going to go through each element here.  
Every `data` property you see here is Batman related and as we speak about a JS framework, each of these properties will apply live all the time.

`data-bind`: binds the element's inner html to the given accessor's value. For instance the first input will display the value returned by `firstName`.  
`data-showif`: shows the element depending on the value of `hasName` which is a boolean.  
`data-event-click`: triggers the given method upon click.

Now take a look at the controller.

``` coffeescript app/assets/batman/controllers/main_controller.js.coffee
class BatmanJsBlog.MainController extends BatmanJsBlog.ApplicationController
  routingKey: 'main'

  index: (params) ->
    @set 'firstName', 'Bruce'
    @set 'lastName', 'Wayne'

  @accessor 'fullName', ->
    "#{@get('firstName')} #{@get('lastName')}"
```

You can actually see that the initial values are set in the `index` action using `@set('var', 'value')`.  
You can also see that the `fullName` accessor that is used in a `data-bind` is defined here and returns the values of the `firstName` and `lastName` variables combined
with `@get('var')`.

Now let's see `hasName` and `resetName`. These two reside in the following file.

``` coffeescript app/assets/batman/views/main/main_index_view.js.coffee
class BatmanJsBlog.MainIndexView extends Batman.View
  resetName: ->
    @controller.set('firstName', '')
    @controller.set('lastName', '')

  @accessor 'hasName', ->
    @controller.get('fullName').length > 1
```

This code is specific to the index view. `hasName` checks that `fullName` contains at least one character and `resetName` resets tha values of `firstName` and `lastName`.


### The models

We can go and create folders and files for the post resource.

    rails g batman:scaffold post

The last command is a bit overkill for what we want to do but pretty handy for starting the project.

Take a look at the main batman file and change the `@root` to `posts#index`.

``` coffeescript app/assets/batman/batman_js_blog.js.coffee
Batman.config.pathToHTML = '/assets/html'

class BatmanJsBlog extends Batman.App

  # This was automatically generated by the scaffold command
  @resources 'posts'

  # Change 'main#index' to this, this is the landing page of the application
  # We want to first display a list of all the posts
  @root 'posts#index'

(global ? window).BatmanJsBlog = BatmanJsBlog
```

The Post model should look like so:

``` coffeescript app/assets/batman/models/post.js.coffee
class BatmanJsBlog.Post extends Batman.Model
  @resourceName: 'posts'
  @storageKey: 'posts'

  # We are using Batman.js with Ruby on Rails...
  @persist Batman.RailsStorage

  # Use @encode to tell batman.js which properties Rails will send back with its JSON.
  @encode 'title', 'content'
  @encodeTimestamps()
```

Let's generate the Comment model.

    rails g batman:model comment

``` coffeescript app/assets/batman/models/comment.js.coffee
class BatmanJsBlog.Comment extends Batman.Model
  @resourceName: 'comments'
  @storageKey: 'comments'

  @persist Batman.RailsStorage

  # Looks familiar...
  @belongsTo 'post'

  # Pretty much straightforward but we have to specify it
  @urlNestsUnder 'post'

  @encode 'content', 'post_id'
  @encodeTimestamps()
```

We can now add the association on the Post model side.

``` coffeescript app/assets/batman/models/post.js.coffee
class BatmanJsBlog.Post extends Batman.Model
  @resourceName: 'posts'
  @storageKey: 'posts'

  @persist Batman.RailsStorage

  # Here is the other part of the relation
  @hasMany 'comments'

  @encode 'title', 'content'
  @encodeTimestamps()
```

We can go back to the main file and set the **nested routes** for posts and comments, which also looks like the syntax you find in a traditional Rails application.

``` coffeescript app/assets/batman/batman_js_blog.js.coffee
...

class BatmanJsBlog extends Batman.App

  @resources 'posts', ->
    @resources 'comments'

...
```

### The controller

Let's go edit the **posts controller** and create an `index` action to list all our posts.  

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
class BatmanJsBlog.PostsController extends BatmanJsBlog.ApplicationController
  routingKey: 'posts'

  index: (params) ->
    @set('posts', BatmanJsBlog.Post.get('all'))
```

Here we are setting a `posts` variable when going through the `index` action that contains all our posts.  
`BatmanJsBlog.Post.get('all')` is a query to our Rails API we created earlier.

> At any time, just open your browser dev tools to watch requests sent by Batman.js.

Now we can modify our **index** view.

``` html app/assets/batman/html/posts/index.html
<span data-bind="'post' | pluralize posts.length"></span>

<ul>
  <li data-foreach-post="posts">
    <a data-bind="post.title" data-route="post"></a> |
    <a data-event-click="destroy">Destroy</a>
  </li>
</ul>
```
The first line sets a binding to `pluralize` the word "post" depending on the number of posts being retrieved: `posts.length`.  
Now look at this line `<li data-foreach-post="posts">`. Here `data-foreach` will iterate through `posts` that have been set earlier in the controller and yield a `li`
tag for each of these posts. In every `li` block, a variable is used to hold the post object: `post` as in `data-foreach-post`.  
`<a data-bind="post.title" data-route="post"></a>`: this is a link, again, the html value is set with `data-bind` and will be the post's title attribute.  
`data-route` is used to set the `href` value of the link. Here `data-route="post"` means we are linking to the `show` action corresponding to the current `post`.  
We also display a "destroy" link. In Batman.js, the `destroy` action is not routable so we have to use an event instead. The event name will be `destroy` and it will
be triggered on `click` on the link, therefore we have `data-event-click="destroy"`.

If you navigate to <http://localhost:3000> you should see the posts list.

Let's add our **show** action to the posts controller.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

show: (params) ->
  BatmanJsBlog.Post.find params.id, @errorHandler (post) =>
    @set('post', post)

...
```

Again `BatmanJsBlog.Post.find params.id` uses the given id in the request parameters to query our API. The result is stored in a `post` variable and set as a controller
attribute as usual with `@set`.

Now we can write the **show** view.

``` html app/assets/batman/html/posts/show.html
<h1 data-bind="post.title"></h1>

<p data-bind="post.content"></p>

<a data-route="routes.posts[post].edit">Edit</a> |
<a data-event-click="destroy">Destroy</a>

<ul>
  <li data-foreach-comment="post.comments">
    <p data-bind="comment.content"></p>
  </li>
</ul>
```

Everything should look familiar now. We are using multiple data bindings to display our post's attributes. `routes.posts[post].edit` is used to route to the **edit** action
of a particular post.  
Here we can also display the comments as they are returned in the JSON too: remember, earlier in this tutorial we included them in the API on the posts' **show** action with
`include: :comments`.

Go back to your application and click on one post: you should see the post's show view.

Now that we have seen the `destroy` event twice, we can handle it in the controller.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

destroy: (node, event, context) ->
  post = if context.get('post') then context.get('post') else @post
  post.destroy (err) =>
    if err
      throw err unless err instanceof Batman.ErrorsSet
    else
      @redirect '/posts'

...
```

The first line instantiates our post.  
As this is an event and not a route we have 3 arguments: `node, event, context`.  
`node` represents the html node.  
`event` represents the event that happenned.  
`context` can help you access objects defined within a certain scope: here our node.

We linked to the **destroy** event at 2 different places: in the show where there is only one post defined (`@post`) and in the index where we use a loop to display multiple posts.
Depending which event was triggered, we have to fetch our post differently.  
If `context.get('post')` exists then it means we are on the index view and we clicked "destroy" for one of our posts. This post is therefore available using context like this.  
If it doesn't exist, then we are on the show view and `@post` is defined because it was defined in the **show** action with `@set('post', post)`.

Then, if everything goes well, we redirect to the **index** view in both cases.

Go ahead and try out these new "destroy" links: on the index and on the show views.

Now we should add some links on our layout to better the navigation.

``` html app/views/layouts/batman.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>Batman Js Blog</title>
  <%= stylesheet_link_tag    "application", :media => "all" %>
  <%= javascript_include_tag "batman_js_blog" %>
  <%= csrf_meta_tags %>
</head>
<body>

<a data-route="routes.posts">Posts</a> |
<a data-route="routes.posts.new">New Post</a>

<div data-yield="main"></div>

<script type="text/javascript">
  BatmanJsBlog.run();
</script>

</body>
</html>
```

We are adding 2 links under the `body`.  
The first one routes to the posts list: the **index** action of our **posts controller**.  
The second one routes to the **new** action of our **posts controller** that we are now going to write.

The **new** action is pretty much straightforward too.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

new: (params) ->
  @set('post', new BatmanJsBlog.Post)

...
```

We just create a new post, not saved yet.

Let's create a **form** on the new view now.

``` html app/assets/batman/html/posts/new.html
<form data-formfor-post="post" data-event-submit="create">
  <div>
    <label>Title:</label>
    <input data-bind="post.title" />
  </div>

  <div>
    <label>Content:</label>
    <textarea data-bind="post.content"></textarea>
  </div>

  <input name="commit" type="submit" value="Submit">
</form>
```

As with `data-foreach`, `data-formfor` uses an existing variable and sets another one to be used within the context, both named "post".  
The form will just trigger an event, here the event is **create**, which corresponds to our **create** action (event) not yet defined.  
We don't need any argument for this action.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

create: ->
  @post.save (err, post) =>
    if err
      throw err unless err instanceof Batman.ErrorsSet
    else
      @redirect post

...
```

`@post` represents the post we created earlier in the **new** action.  
We can save the post and redirect to the corresponding **show** action.

Now, go back to your browser and try creating a new post.

The **edit** action now is going to look quite like the **show** action because it only fetches a post with its id.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

show: (params) ->
  BatmanJsBlog.Post.find params.id, @errorHandler (post) =>
    @set('post', post)

edit: (params) ->
  BatmanJsBlog.Post.find params.id, @errorHandler (post) =>
    @set('post', post)

...
```

Let's refactor and write a `fetchPost` method.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
class BatmanJsBlog.PostsController extends BatmanJsBlog.ApplicationController
  routingKey: 'posts'

  @beforeAction 'fetchPost', only: ['show', 'edit']

  ...

  show: (params) ->

  edit: (params) ->

  ...

  fetchPost: (params) ->
    BatmanJsBlog.Post.find params.id, @errorHandler (post) =>
      @set('post', post)
```

Here, the `@beforeAction` works exactly like Rails' one does. It executes `fetchPost` before the **show** and **edit** actions.

Again the **edit** view looks like the **new** view.

``` html app/assets/batman/html/posts/edit.html
<form data-formfor-post="post" data-event-submit="update">
  <div>
    <label>Title:</label>
    <input data-bind="post.title" />
  </div>

  <div>
    <label>Content:</label>
    <textarea data-bind="post.content"></textarea>
  </div>

  <input name="commit" type="submit" value="Submit">
</form>
```

The only difference is the event triggered: here it is `update`.

Let's refactor both our **edit** and **new** views using a partial.

``` html app/assets/batman/html/posts/edit.html
<form data-formfor-post="post" data-event-submit="update">
  <div data-partial="posts/_form"></div>
</form>
```

``` html app/assets/batman/html/posts/new.html
<form data-formfor-post="post" data-event-submit="create">
  <div data-partial="posts/_form"></div>
</form>
```

I think you get it here without explanations... Just create the **_form** partial.


``` html app/assets/batman/html/posts/_form.html
<div>
  <label>Title:</label>
  <input data-bind="post.title" />
</div>

<div>
  <label>Content:</label>
  <textarea data-bind="post.content"></textarea>
</div>

<input name="commit" type="submit" value="Submit">
```

Finally, the **update** action looks exactly like the **create** action.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

update: ->
  @post.save (err, post) =>
    if err
      throw err unless err instanceof Batman.ErrorsSet
    else
      @redirect post

...
```

You should be able to edit your posts thanks to the previously created link in the show view now.

### The comments

Let's add a link to destroy our comments on the posts' show view.

``` html app/assets/batman/html/posts/show.html
...

<ul>
  <li data-foreach-comment="post.comments">
    <p data-bind="comment.content"></p>
    <a data-event-click="destroyComment">Destroy</a>
  </li>
</ul>
```

The event triggered by the link will be `destroyComment` as we are still in the posts controller and the `destroy` action (event) is already used to destroy a post. It would
probably be messy to use the same event.

Let's write this new event.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
destroyComment: (node, event, context) ->
  comment = context.get('comment')
  comment.destroy (err) =>
    if err
      throw err unless err instanceof Batman.ErrorsSet
    else
      @post.get('comments').remove comment
      @redirect '/posts/' + @post.get('id')
```

Here we get the `comment` using the context.  
If everything goes well, we also remove the comment from the post's comments list so that the comment disappears (on the html page).  
And then we redirect on the post's **show** view i.e. where we were when clicking the destroy link.

Go ahead and try it.

Now let's add the possibility to create a new comment on every post show view.

``` html app/assets/batman/html/posts/show.html
...

<form data-formfor-comment="comment" data-event-submit="createComment">
  <div>
    <label>Content:</label>
    <textarea data-bind="comment.content"></textarea>
  </div>

  <input name="commit" type="submit" value="Submit">
</form>
```

Then just get the controller right.

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
...

show: (params) ->
  # Initializing a new comment with the post_id given in params to display
  # a corresponding form
  @set('comment', new BatmanJsBlog.Comment(post_id: params.id))

...

createComment: ->
  @comment.save =>
    # If everything goes well, we add the new comment to the current post's comments list so that it appears on the (html) page
    @post.get('comments').add @comment
    @redirect '/posts/' + @post.get('id')

...
```

You should now be able to create new comments. ; )

### Errors handling

We can now add some information when validations fail for posts.

You can add this at the top of the **new** and **edit** views.

``` html
<div data-partial="posts/_errors"></div>
```

And create the corresponding partial.

``` html app/assets/batman/html/posts/_errors.html
<div data-showif="post.errors.length">
  <p>
    <span data-bind="'error' | pluralize post.errors.length"></span>
    prevented this post from being created:
  </p>
  <ul>
    <li data-bind="error.message" data-foreach-error="post.errors"></li>
  </ul>
</div>
```

The first line means the `div` is displayed only if there are errors included within the post.  
Take a look at the `li` tag, and see how we took advantage of `data-bind` combined with `data-foreach` here.

If validations fail, Rails sends back the object with errors included in the JSON. But we could also take advantages of Batman.js capabilities and add some validations on
the client side.

``` coffeescript app/assets/batman/models/post.js.coffee
...

@validate "title",   presence: true
@validate "content", presence: true
```

Now Batman.js knows how to validate our post objects and does not need to send a request to Rails and wait for a reply before showing the errors: it can display them right away.

You can try creating an empty post and see what happens. : )

As a recap for this last part, your **posts_controller** should look like this:

``` coffeescript app/assets/batman/controllers/posts_controller.js.coffee
class BatmanJsBlog.PostsController extends BatmanJsBlog.ApplicationController
  routingKey: 'posts'

  @beforeAction 'fetchPost', only: ['show', 'edit']

  index: (params) ->
    @set('posts', BatmanJsBlog.Post.get('all'))

  show: (params) ->
    @set('comment', new BatmanJsBlog.Comment(post_id: params.id))

  edit: (params) ->

  new: (params) ->
    @set('post', new BatmanJsBlog.Post)

  create: ->
    @post.save (err, post) =>
      if err
        throw err unless err instanceof Batman.ErrorsSet
      else
        @redirect post

  update: ->
    @post.save (err, post) =>
      if err
        throw err unless err instanceof Batman.ErrorsSet
      else
        @redirect post

  destroy: (node, event, context) ->
    post = if context.get('post') then context.get('post') else @post
    post.destroy (err) =>
      if err
        throw err unless err instanceof Batman.ErrorsSet
      else
        @redirect '/posts'

  createComment: ->
    @comment.save =>
      @post.get('comments').add @comment
      @redirect '/posts/' + @post.get('id')

  destroyComment: (node, event, context) ->
    comment = context.get('comment')
    comment.destroy (err) =>
      if err
        throw err unless err instanceof Batman.ErrorsSet
      else
        @post.get('comments').remove comment
        @redirect '/posts/' + @post.get('id')

  fetchPost: (params) ->
    BatmanJsBlog.Post.find params.id, @errorHandler (post) =>
      @set('post', post)
```

Hope it all helps, talk to you later! ; )