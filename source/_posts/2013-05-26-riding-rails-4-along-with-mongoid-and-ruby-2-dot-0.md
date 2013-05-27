---
layout: post
title: "Riding Rails 4 along with Mongoid and Ruby 2.0"
date: 2013-05-26 01:38
comments: true
author: Neil Rosenstech
categories:
- Ruby on Rails
- Mongoid
---

You're not without knowing that Rails 4 is about to kick in in a few days (weeks?), are you? In a recent hobby project of mine: [6LOCK](https://github.com/Raindal/6lock) (still in construction),
I tested out MongoDB, together with Mongoid (the ORM that comes in replacement of ActiveRecord for querying the MongoDB database) and other pretty cool stuff. As I had a great time using it
I wanted to give it a try using the all new and shiny Rails 4 and Ruby 2.

If you have a basic understanding of how the framework works, you have already been following some sort of "Getting Started with Ruby on Rails" tutorial (plenty of them out there) 
and now you want to try MongoDB and see how it feels, this tutorial is made for you.

Note: I am not going to list the new features of Rails 4, if you want to know more, check out this great post: 
[digging-into-rails-4](http://net.tutsplus.com/tutorials/ruby/digging-into-rails-4/)

All the resources and documentation stuff are at the end of the post, be sure to check them out, you'll probably learn very interesting stuff and subtleties.

I ran into some compatibility issues when writing this tutorial (nothing to be worried about) but gems versions may now have evolved.  
FYI: Ubuntu 12.10, rails 4.0.0.rc1, mongoid (github master, something between 3.1.4 and ?) and bson_ext 1.8.6

Application github repo: [https://github.com/Raindal/rails4_mongoid](https://github.com/Raindal/rails4_mongoid)

Finally, I am not going to give you any advice on why you should or should not use MongoDB, lots of topics are listing pros and cons compared to traditional relational databases such as
MySQL. To give you some leads you could dig into, MongoDB brings to the table things like dynamic fields, embedding, map/reduce, indexing and so on.

But enough talking, let's dive in!

### Install MongoDB

> [Install MongoDB on Ubuntu](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/)

First you'll need to import the 10gen public GPG key (MongoDB is mainly maintained by 10gen), which is used to ensure package consistency and authenticity

    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10

Create a /etc/apt/sources.list.d/10gen.list file using the following command

    echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/10gen.list

Now you can update your repository

    sudo apt-get update

And then install the package

    sudo apt-get install mongodb-10gen

To check that it works, just issue a `mongo` in your shell and you should see something like

    MongoDB shell version: x.x.x
    connecting to: test
    >

### Set up RVM

> [Why you should use RVM](http://net.tutsplus.com/tutorials/why-you-should-use-rvm/)

Now we are going to set up RVM to use Ruby 2.0.0. We only have to install the Ruby version we want and use it.

    rvm install 2.0.0
    rvm use 2.0.0

And finally we are going to create a gemset for our application, it will contain all the gems we need

    rvm gemset create rails4_mongoid

### Set up the application

First we need to install the Rails 4 gem.

    gem install rails --version 4.0.0.rc1 --no-ri --no-rdoc

Now we can create the application and cd inside

    rails new rails4_mongoid
    cd rails4_mongoid/

For convinience, we are creating a .rvmrc file at the root of our application. The file will tell RVM to use the right version of Ruby with the right gemset when we cd inside the directory.  
Just use your favorite text editor

    nano .rvmrc

And paste this inside

``` ruby .rvmrc
rvm use 2.0.0@rails4_mongoid --create
```

This basically tells RVM to use Ruby 2.0.0 with the gemset rails4_mongoid and that it should create the gemset if it doesn't already exist.

To make sure everything works as expected, you can go back and then cd again in the directory.

    cd ..
    cd rails4_mongoid

You should see something like this

    Using /home/neil/.rvm/gems/ruby-2.0.0-p0 with gemset rails4_mongoid

Now, we have to edit the Gemfile to install the gems we want.  
First, remove sqlite: delete these lines

``` ruby Gemfile
# Use sqlite3 as the database for Active Record
gem 'sqlite3'
```
Now we can add mongoid and bson_ext

``` ruby Gemfile
gem 'mongoid', github: 'mongoid/mongoid'
gem 'bson_ext', '~> 1.8.6'
```

At the time I'm writing these lines, [mongoid 3.1.4 relies on ActiveModel 3.2](http://rubygems.org/gems/mongoid/versions/3.1.4) but we are using ActiveModel 4.0.0-rc1, so we need to use directly
the github repo that supports it.  
bson_ext is a C extension which basically makes things faster with MongoDB.

Now, just run a

    bundle update

If you run into errors of some kind (mine was related to bson_ext), try to update your gem version with

    rvm rubygems latest --verify-downloads 1

And `bundle update` again.

### Configure Mongoid

Still in the root of the application, run the following command

    rails g mongoid:config

This will generate a **mongoid.yml** file under **config/**.  
We can edit the file to suit our needs.

Find the line that says `# consistency: :eventual` and change it for

``` yaml config/mongoid.yml
consistency: :strong
```

`:eventual` will send reads to secondaries, `:strong` sends everything to master.

As we are not using ActiveRecord, we need to get rid of it.  
First in **config/application.rb** comment out the line

``` ruby config/application.rb
# require 'rails/all'
```

And add these lines right underneath the first one

``` ruby config/application.rb
require "action_controller/railtie"
require "action_mailer/railtie"
# require "active_resource/railtie"
require "sprockets/railtie"
```

The 3rd line is commented out because normally you would need it but Rails 4 has extracted the ActiveResource gem and now you need to specify its name in the Gemfile if you want to use it. But here we won't
use it.

There are some more ActiveRecord settings we have to get rid of out there.  
In **config/environments/development.rb**, comment out this line

``` ruby config/environments/development.rb
# config.active_record.migration_error = :page_load
```

To check that everything is working like a charm, run a `rails s` and then go check that the application is running on [http://localhost:3000](http://localhost:3000).

Now you should see the usual **Welcome aboard** message.

### Now we're talking!

Let's write some code, finally! But first we will have to generate our first model. For that we are going to use the builtin *scaffold* command in order to generate our model, controller, 
views and everything else at the same time.

> In a real project, unless you're sure you're going to use all the generated files, I personnaly don't recommend using the scaffold command. You could quickly end up with a messy application.
I would rather recommend using each command one by one: `rails g model xxx`, `rails g controller xxx` etc.

For this example I'll be using the usual Post > Comment stuff which is actually a perfect example for demonstrating the embedding capabilities of MongoDB.

Let's generate our Post first, it will have 3 fields: a **title**, a **body** and it can be **starred**.

    rails g scaffold post title:string body:string starred:boolean

> [Mongoid document field types](http://mongoid.org/en/mongoid/docs/documents.html#fields)

This generated all the files we wanted.  
Let's first edit the model to add the **created_at** and **updated_at** fields that are not present by default.  
We can also add an index on the **starred** field because we will often be querying on it with a `where` clause for example (not in this application actually).

Your Post model should look like so

``` ruby app/models/post.rb
class Post
  include Mongoid::Document
  include Mongoid::Timestamps

  field :title, type: String
  field :body, type: String
  field :starred, type: Boolean

  index({ starred: 1 })
end
```

Now we have a command to enforce indexes on the database

    rake db:mongoid:create_indexes

This will set an index on the *starred* field of the *post* collection.

> Quick tip: use `rake -T` to see a list a available tasks

> Wait... we didn't create the database, did we? No we didn't, the database is automatically created.

For some reason, Rails 4 uses the `update()` method by default to update an object in the corresponding controller action which does not behave as one could expect (and basically does not update
the document). This may be due to mongoid... Anyway, let's use `update_attributes()` instead.

In the recently created controller, change the *update* action to look like this one

``` ruby app/controllers/posts_controller.rb
# PATCH/PUT /posts/1
# PATCH/PUT /posts/1.json
def update
  respond_to do |format|
    if @post.update_attributes(post_params)
      format.html { redirect_to @post, notice: 'Post was successfully updated.' }
      format.json { head :no_content }
    else
      format.html { render action: 'edit' }
      format.json { render json: @post.errors, status: :unprocessable_entity }
    end
  end
end
```

We actually do not need all the json stuff because we are not writing some sort of API but the `scaffold` command generates it for us.

Now if you navigate to [http://localhost:3000/posts](http://localhost:3000/posts) you should be able to play with the posts. : )  
Go ahead, create/update/destroy a few posts to see if everything is working as expected.

## Embedding documents

> A collection can be compared to a table.  
A document can be compared to a row.

Here I've decided to show you how to embed documents in each other because this is one of the features that you will probably wind up using the most.  
In MongoDB you can associate two documents with "foreign keys" although the concept does not really exists. So you can still use the usual stuff: `has_many`, `belongs_to` and so on.  
**But**, there is no *join* is MongoDB, loading one document and its associated document will therefore execute two queries.

However you can set another type of relation: you can embed one document into another. Loading the parent document will also load all the children documents at the same time.  
**But**, it will always instanciate all of the objects retrieved even if you do not need the children, plus, each document has a size limit: 16MB, if you have too many child
documents, the parent document's size could exceed this limit.

> Foreign key = 2 queries  
Embedding = loading many objects all the time + parent document size limited

> [MongoDB limits](http://docs.mongodb.org/manual/reference/limits/)

We suppose that when we load a post, we also want all of its associated comments and that there will not be thousands of comments.  
So embedding seems to be a good choice.

Here is a design example of a MongoDB database with users that have posts and posts that have comments.

**users collection**
    [
      {
        "_id": "5063114bd386d8fadbd6b004",
        "name": "John Snow"
      }
    ]

**posts collection**  
    [
      {
        "_id": "6563521bd386d6dadbd6b002",
        "title": "Why King's Landing should belong to Daenerys",
        "user_id": "5063114bd386d8fadbd6b004",
        "comments": [
          {
            "_id": "4586521bd638d6dadbd7b003"
            "body": "I totally agree with you, great post!"
          },
          {
            "_id": "8526521bd654d6dadbd7b001"
            "body": "I'm so sad since Drogo died..."
          }
        ]
      }
    ]

This being said, let's get started, shall we?

    rails g scaffold comment body:string

Now that our Comment model is generated, we can edit it to reflect this

``` ruby app/models/comment.rb
class Comment
  include Mongoid::Document
  include Mongoid::Timestamps

  embedded_in :post, inverse_of: :comments

  field :body, type: String
end
```

This just tells MongoDB that comments are to be embedded in the corresponding post.  
The `inverse_of` option is required in order to tell Mongoid what the comment should be embedded through.

We can edit the Post model to reflect that association and add the `embeds_many :comments` (line 5)

``` ruby app/models/post.rb
class Post
  include Mongoid::Document
  include Mongoid::Timestamps

  embeds_many :comments

  field :title, type: String
  field :body, type: String
  field :starred, type: Boolean

  index({ starred: 1 })
end
```

Now we need to edit the **routes** file to make posts and comments nested resources.

``` ruby config/routes.rb
resources :posts do
  resources :comments
end
```

This creates urls like */posts/:post_id/comments*.

Now we have to change our **comments_controller.rb** and all our comments's views to reflect these nested resources: any comment we will be using now depends on a post.  
Note: this part is not related to MongoDB or Mongoid. We would have gone through the same steps anyway with nested resources.

> [Nested resources](http://guides.rubyonrails.org/routing.html#nested-resources)

``` ruby app/controllers/comments_controller.rb
class CommentsController < ApplicationController
  before_action :load_post
  before_action :set_comment, only: [:show, :edit, :update, :destroy]

  # GET /comments
  # GET /comments.json
  def index
    @comments = @post.comments
  end

  # GET /comments/1
  # GET /comments/1.json
  def show
  end

  # GET /comments/new
  def new
    @comment = @post.comments.build
  end

  # GET /comments/1/edit
  def edit
  end

  # POST /comments
  # POST /comments.json
  def create
    @comment = @post.comments.build(comment_params)

    respond_to do |format|
      if @comment.save
        format.html { redirect_to [@post, @comment], notice: 'Comment was successfully created.' }
        format.json { render action: 'show', status: :created, location: @comment }
      else
        format.html { render action: 'new' }
        format.json { render json: @comment.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /comments/1
  # PATCH/PUT /comments/1.json
  def update
    respond_to do |format|
      if @comment.update_attributes(comment_params)
        format.html { redirect_to [@post, @comment], notice: 'Comment was successfully updated.' }
        format.json { head :no_content }
      else
        format.html { render action: 'edit' }
        format.json { render json: @comment.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /comments/1
  # DELETE /comments/1.json
  def destroy
    @comment.destroy
    respond_to do |format|
      format.html { redirect_to post_comments_url(@post) }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_comment
      @comment = @post.comments.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def comment_params
      params.require(:comment).permit(:body)
    end

    def load_post
      @post = Post.find(params[:post_id])
    end
end
```

What changed? Well, now we use the post passed in params to load, create, redirect comments.  
See the **load_post** method, this is the root of everything else. We load a post using the :post_id that is sent and then we use this post everywhere.

> Quick tip: issue a `rake routes` to see the routes you can use in the application

Now we need to change these routes in all the view files.
Here I'll show you only the lines that have changed.

``` erb app/views/comments/index.html.erb
<td><%= link_to 'Show', post_comment_path(@post, comment) %></td>
<td><%= link_to 'Edit', edit_post_comment_path(@post, comment) %></td>
<td><%= link_to 'Destroy', [@post, comment], method: :delete, data: { confirm: 'Are you sure?' } %></td>

...

<%= link_to 'New Comment', new_post_comment_path(@post) %>
```

``` erb app/views/comments/_form.html.erb
<%= form_for([@post, @comment]) do |f| %>

...
```

``` erb app/views/comments/new.html.erb
...

<%= link_to 'Back', post_comments_path(@post) %>
```

``` erb app/views/comments/show.html.erb
...

<%= link_to 'Edit', edit_post_comment_path(@post, @comment) %> |
<%= link_to 'Back', post_comments_path(@post) %>
```

``` erb app/views/comments/edit.html.erb
...

<%= link_to 'Show', post_comment_path(@post, @comment) %> |
<%= link_to 'Back', post_comments_path(@post) %>
```

Now return in your running application and create a new post.  
Click on the *show* link to view the post you just created and just append *comments* to the url, like so: http://localhost:3000/posts/:post_id/comments.

That's it! You should now be able to create comments and manage them with all the basic usual actions.

I hope this little introduction to Rails 4 with Mongoid helped you.

See you around!

### Resources

+ [Why you should use RVM](http://net.tutsplus.com/tutorials/why-you-should-use-rvm/)
+ [Digging into Rails 4](http://net.tutsplus.com/tutorials/ruby/digging-into-rails-4/)
+ [Install MongoDB on Ubuntu](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/)
+ [MongoDB limits](http://docs.mongodb.org/manual/reference/limits/)
+ [Mongoid installation](http://mongoid.org/en/mongoid/docs/installation.html)
+ [Mongoid document field types](http://mongoid.org/en/mongoid/docs/documents.html#fields)
+ [Indexing with Mongoid](http://mongoid.org/en/mongoid/docs/indexing.html)
+ If you don't know what are the new "concerns" folders in Rails 4: [Put chubby models on a diet with concerns](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns)
+ [Nested resources](http://guides.rubyonrails.org/routing.html#nested-resources)